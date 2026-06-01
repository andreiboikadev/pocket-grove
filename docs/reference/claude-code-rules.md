# Claude Code Rules — A Practical Guide for Effective Project Work

> **Who this is for.** A chat/agent (Claude/Opus) or a developer who needs to understand what
> "rules" are in Claude Code, the types that exist, and how to compose and use them so the model
> works correctly and consistently on a project. All facts here are verified against the official
> Claude Code documentation (links at the end).

---

## 1. The one mental model to keep: SOFT vs HARD

Everything below is either a **soft rule** or a **hard rule**. Getting this distinction right is 90% of using rules well.

| Type | What it is | Enforced by | Guarantee |
|---|---|---|---|
| **SOFT** (guidance) | Instructions Claude reads and tries to follow | The model's judgment | High adherence if written well, but **no guarantee** |
| **HARD** (enforcement) | Constraints applied by the Claude Code client itself | The harness (not the model) | **Guaranteed** — applies regardless of what the model decides |

Official wording:
- CLAUDE.md / memory / rules are "**context, not enforced configuration**."
- "**Permission rules are enforced by Claude Code, not by the model.**"
- To block an action no matter what, "**use a PreToolUse hook**."

**Rule of thumb:** behavior, conventions, "how we work" → SOFT. "This must never happen" / "this must always run" → HARD.

---

## 2. The mechanisms (the actual "rules")

| # | Mechanism | Soft/Hard | Who writes | Committed to repo? | Use for |
|---|---|---|---|---|---|
| A | `CLAUDE.md` | Soft | You | Yes (project) | Always-on project facts, conventions, workflow |
| B | `.claude/rules/*.md` | Soft | You | Yes | Modular / path-scoped instructions |
| C | Auto memory | Soft | Claude | No (machine-local) | Things Claude learns from your corrections |
| D | `settings.json` `permissions` | **Hard** | You | Yes (`.claude/settings.json`) | Allow/ask/**deny** specific tools & commands |
| E | Hooks (`PreToolUse`, …) | **Hard** | You | Yes | Deterministic block/automation at lifecycle events |

### A. `CLAUDE.md` — the primary instruction file (SOFT)
Plain markdown that Claude loads **at the start of every session**. Locations, in load order (broad → specific; all are **concatenated**, not overridden):

| Scope | Location | Shared with |
|---|---|---|
| Managed policy | OS-specific managed path | Whole org (cannot be excluded) |
| User | `~/.claude/CLAUDE.md` | Just you, all projects |
| **Project** | `./CLAUDE.md` or `./.claude/CLAUDE.md` | **Team, via source control** |
| Local | `./CLAUDE.local.md` | Just you, this project (gitignore it) |

- Subdirectory `CLAUDE.md` files load **on demand** when Claude reads files in that subtree.
- Imports: `@path/to/file` (relative or absolute, max **4 hops**). Imported files still load at launch and consume context.
- `CLAUDE.local.md` is **not deprecated** — it's the standard place for private, gitignored, per-project notes.
- `AGENTS.md`: Claude reads `CLAUDE.md`, not `AGENTS.md`. If you have one, put `@AGENTS.md` at the top of `CLAUDE.md`.
- Create with `/init`; view/edit loaded files with `/memory`.

### B. `.claude/rules/*.md` — modular, optionally path-scoped (SOFT)
For larger projects, split instructions into topic files in `.claude/rules/`:
```
.claude/
├── CLAUDE.md            # main, always-on
└── rules/
    ├── code-style.md
    ├── testing.md
    └── api.md
```
- Files **without** a `paths` field load at launch (same priority as `.claude/CLAUDE.md`).
- Files **with** a `paths` field load only when Claude touches matching files — saves context:
  ```markdown
  ---
  paths:
    - "Assets/**/*.cs"
    - "src/**/*.{ts,tsx}"
  ---
  # Rules that apply only to those files
  ```
- `~/.claude/rules/` holds personal, all-project rules (lower priority than project rules).

### C. Auto memory — Claude's own notes (SOFT)
Claude writes learnings to `~/.claude/projects/<project>/memory/MEMORY.md` (first 200 lines / 25 KB loaded each session). It's **machine-local, not shared via git**. Toggle/inspect with `/memory`. Good for discovered build commands and preferences; not a substitute for explicit CLAUDE.md rules.

### D. `permissions` in `settings.json` — HARD, enforced
Three lists, evaluated **deny → ask → allow** (first match wins; a deny at any scope can't be overridden):
```json
{
  "permissions": {
    "allow": ["Bash(npm run *)"],
    "ask":   [],
    "deny":  ["Bash(git push *)"]
  }
}
```
- Pattern syntax: `Tool` or `Tool(specifier)`. `Bash(git commit *)` matches `git commit` + anything. The trailing `:*` is **equivalent** to a trailing ` *` (so `Bash(git commit:*)` == `Bash(git commit *)`); `:*` is only recognized at the end of a pattern.
- Claude Code is shell-operator-aware: a `Bash(safe *)` rule does **not** authorize `safe && other` (each subcommand must match).
- **Caveat (from the docs): "Bash permission patterns that try to constrain command *arguments* are fragile."** Prefix blocks like "no git commit" are reliable; argument-level filtering (e.g., "curl only to one domain") is not — use a hook for those.

Settings file locations:
| Scope | File | Committed? |
|---|---|---|
| Project (shared) | `.claude/settings.json` | **Yes** |
| Local (personal) | `.claude/settings.local.json` | No (auto-gitignored) |
| User | `~/.claude/settings.json` | No |
| Managed | OS managed path | Admin-deployed, wins over all |

### E. Hooks — HARD, deterministic
Shell commands that run at lifecycle events (`PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `SessionStart`, `SessionEnd`, …) for things that must "always happen rather than relying on the LLM."
- A `PreToolUse` hook **denies a call** by returning `exit 0` with a JSON `permissionDecision: "deny"` (the documented form), or by `exit 2` (which hard-stops the call before permission rules are even evaluated). Either way the block overrides allow rules.
- Configured in `settings.json` (`hooks` block); scripts live in `.claude/hooks/`. Committed with the repo.
- Use for: ironclad command blocking (beyond fragile arg-patterns), auto-format on edit, run validation after edits.

> Related (not "rules" but adjacent): **Skills** (`.claude/skills/`) for on-demand procedures that load only when relevant; **Subagents** for isolated tasks. Use skills for multi-step procedures that shouldn't sit in context all the time.

---

## 3. How to compose rules well

Straight from the docs + practice:

1. **Keep `CLAUDE.md` under ~200 lines.** Longer files consume context and *reduce* adherence. Split with `.claude/rules/` or imports.
2. **Be specific and verifiable.** "Use 2-space indentation" > "format code properly". "Run `npm test` before marking done" > "test your changes".
3. **Structure it.** Markdown headers + bullets. Claude scans structure like a reader.
4. **No contradictions.** If two rules conflict, the model picks one arbitrarily. Prune periodically.
5. **Put each thing in the right mechanism:**
   - Always-true project fact / convention → `CLAUDE.md`
   - Applies only to certain files → path-scoped `.claude/rules/`
   - A repeatable multi-step procedure → a **skill** (loads on demand)
   - "Must never happen" → `permissions.deny` (and a hook if it must be ironclad)
   - "Must always run at point X" → a **hook**
6. **Add a rule when you correct the same thing twice** — that's the signal it belongs in CLAUDE.md, not in chat.
7. **Don't bloat.** Every line is context spent every session. Delete stale rules.

---

## 4. How to use them for effective work on a project

- **Commit the shared ones** so every machine and every new chat inherits them automatically:
  `./CLAUDE.md`, `./.claude/rules/*`, `./.claude/settings.json`, `./.claude/hooks/`.
- **Keep personal/machine-specific ones out of git:** `CLAUDE.local.md`, `.claude/settings.local.json`.
- **Verify loading:** run `/memory` — if a file isn't listed, Claude can't see it.
- **Tighten over time:** start minimal (CLAUDE.md + one or two hard rules), grow as real friction appears. This is the same "no over-engineering" principle as good code.
- **When an instruction isn't being followed:** make it more specific, check for conflicts, and if it *must* hold, promote it from SOFT (CLAUDE.md) to HARD (permission/hook).

---

## 5. Worked example — project with MCP + "assistant never commits"

This is the common setup for an automated project where the assistant does the work but **you review the diff and commit yourself**.

**`./CLAUDE.md`** (committed) — soft rules:
```markdown
# Project: <name>

## What this is
<1–3 lines: engine/stack, target platform, what we're building>

## Division of labor
- You (assistant): edit source/text files directly; do editor operations via the MCP server;
  add package deps by editing the manifest. Read-only git (`git status`, `git diff`) is fine.
- Me (human): platform/IDE GUI toggles, runs on device, and ALL git commits.

## Hard rule
- Never run `git commit`, `git push`, or `git add`. After a logical unit of work, stop,
  summarize the change, and propose a commit message. I review the diff and commit.

## Conventions
- <language/style: e.g., 2-space indent, PascalCase types, one public type per file>
- Before "ready for review": code compiles with no console errors; tests pass.
```

**`./.claude/settings.json`** (committed) — hard rule that backs up the soft one:
```json
{
  "permissions": {
    "deny": [
      "Bash(git commit *)",
      "Bash(git push *)"
    ]
  }
}
```
*(Add `"Bash(git add *)"` too if the assistant shouldn't even stage. These deny prefixes are enforced by the client; the assistant literally cannot run them. Your own terminal git is unaffected — only the agent's Bash tool is.)*

**Optional — ironclad block via hook** (`./.claude/settings.json` + `./.claude/hooks/block-commit.sh`):
```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash",
        "hooks": [ { "type": "command", "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-commit.sh" } ] }
    ]
  }
}
```
```bash
#!/bin/bash
# PreToolUse hook. The tool call arrives as JSON on stdin; block git commit/push.
command=$(jq -r '.tool_input.command // empty')
if echo "$command" | grep -Eq '\bgit\b.*\b(commit|push)\b'; then
  jq -n '{ hookSpecificOutput: {
    hookEventName: "PreToolUse",
    permissionDecision: "deny",
    permissionDecisionReason: "Commits and pushes are reserved for the human; blocked by project policy."
  } }'
fi
exit 0   # exit 0 + the JSON above = block; exit 0 with no output = allow
```
Use the hook only if you want a guarantee that survives unusual command phrasings; for most projects the `deny` rules + CLAUDE.md are enough.

**Why this matches a multi-machine / multi-chat setup:** all three files are committed, so anyone who clones the repo (any machine, any new chat) gets the same rules with zero extra setup.

---

## 6. Pitfalls

- **Treating CLAUDE.md as enforcement.** It's guidance. If it MUST hold, use a permission/hook.
- **Over-long CLAUDE.md.** >200 lines hurts adherence. Move detail to rules/skills/imports.
- **Argument-level Bash deny patterns.** Fragile (docs say so). Use a hook for argument logic.
- **Vague instructions.** "Be careful with X" does little. State the concrete do/don't.
- **Conflicting rules across files.** The model picks arbitrarily — keep them consistent.
- **Putting personal config in committed files.** Use `CLAUDE.local.md` / `settings.local.json`.

---

## 7. Cheat sheet

| Goal | Use |
|---|---|
| Persistent project context & conventions | `CLAUDE.md` (committed) |
| Instructions only for certain files | `.claude/rules/*.md` with `paths:` |
| On-demand multi-step procedure | a **skill** |
| Block a command (e.g., commits) | `permissions.deny` (+ hook for ironclad) |
| Always run something at an event | a **hook** |
| Personal, not shared | `CLAUDE.local.md`, `settings.local.json` |
| See what's loaded | `/memory` |
| Generate a starting CLAUDE.md | `/init` |

---

## 8. Sources (official Claude Code docs)

- Memory / CLAUDE.md / `.claude/rules/` — https://code.claude.com/docs/en/memory
- Settings & file locations — https://code.claude.com/docs/en/settings
- Permissions (rule syntax, deny, enforcement, Bash caveats) — https://code.claude.com/docs/en/permissions
- Hooks (events, PreToolUse, exit code 2) — https://code.claude.com/docs/en/hooks-guide
- Skills (on-demand procedures) — https://code.claude.com/docs/en/skills
