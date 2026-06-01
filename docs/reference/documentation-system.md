# Claude Code Unity Documentation System

> **Historical blueprint — already implemented.** This is the original design rationale for this
> repo's documentation system. The **live, authoritative** setup is the actual `CLAUDE.md`, `.claude/`,
> and `docs/` in this repo (start at [`../INDEX.md`](../INDEX.md)). Read this only for the "why";
> where a template here differs from the real files, **the real files win**. Imperative phrasing below
> ("the next chat must…") is historical, not a live to-do.

Status: reference — the documentation system described here is already implemented in this repo; kept for rationale. Live map: `../INDEX.md`  
Last verified: 2026-05-31  
Purpose: guide the next chat that will configure documentation inside the Unity project  
Target project type: small Unity AR game built with AI-assisted sessions and PR-style review  
Language policy: all project Markdown files created from this guide must be written in English.

## 1. How To Use This File

Originally this was an instruction package for the chat that set up the docs. That setup is done — this file is kept only as rationale; the live system is the repo's actual `CLAUDE.md`, `.claude/`, and `docs/`.

The next chat must:

1. Read this file before editing the Unity project.
2. Verify the actual Unity repository root before creating files.
3. Stop and ask if another agent is actively working in that Unity project.
4. Create or update the documentation system inside the Unity repository only after the project root is confirmed.
5. Keep generated docs small, linked, and maintainable.

The expected result is a repository-local documentation system that lets human developers and Claude Code sessions work through repeated sessions without losing context or turning the repo into a pile of stale notes.

## 2. Design Goals

The documentation system must optimize for:

- Fast onboarding for a new human or AI session.
- Clear handoff between coding sessions.
- Pull-request-style development even when one person is working alone.
- Low duplication between docs.
- Clear source of truth for game design, architecture, setup, tests, and current status.
- Claude Code instructions that are concise enough to load reliably.
- Hard safety controls for actions that must be blocked, instead of trusting soft prompts.
- Practical Unity version-control rules.
- A small first implementation that can be expanded only when needed.

The system must not optimize for:

- A giant documentation website before the prototype exists.
- A huge `CLAUDE.md` that burns context every session.
- Duplicating the same rules into `CLAUDE.md`, `AGENTS.md`, README, and random docs.
- Keeping important decisions only in chat history.
- Hiding build/test commands in a previous conversation.
- Creating many empty folders and placeholder documents.
- Treating AI memory as a substitute for repository documentation.

## 3. Source-Backed Principles

Use these principles when setting up the project docs:

1. **Docs as code.** Documentation should be plain text in the repository, reviewed with code, and updated in the same workflow as code changes.
2. **README as front door.** The README should explain what the project is, why it matters, how to start, and where to go next. It should not become the whole manual.
3. **Claude Code memory is soft guidance.** `CLAUDE.md` gives Claude persistent instructions, but it does not enforce behavior. For hard safety boundaries, use permissions or hooks.
4. **Keep `CLAUDE.md` concise.** Claude Code docs recommend concise, specific, well-structured instructions and a target under about 200 lines per `CLAUDE.md`.
5. **Use `.claude/rules/` for modular guidance.** Topic rules and path-scoped rules reduce noise and keep the main file short.
6. **Use skills for procedures.** If a repeated process grows beyond a short rule, create a Claude Code skill so it loads only when relevant.
7. **Use hooks or permissions for hard rules.** If something must never happen, block it with `permissions.deny` or a `PreToolUse` hook.
8. **Separate documentation types.** Use Diataxis thinking: tutorials, how-to guides, reference, and explanation are different user needs.
9. **Record decisions as ADRs.** Architecture decisions should be short, dated, and versioned in Markdown.
10. **Keep changelogs human-readable.** Do not dump raw git logs into `CHANGELOG.md`.
11. **Use structured commits.** Conventional Commits make changes easier for humans and tools to scan.
12. **Respect Unity version-control rules.** Track `Assets`, `Packages`, `ProjectSettings`, and `.meta` files. Ignore generated local folders such as `Library` and `UserSettings`.

## 4. Target Repository Documentation Structure

This is the target structure inside the Unity repository root. It is not a command to create every file immediately.

```text
README.md
CLAUDE.md
AGENTS.md                         optional, for Codex or other non-Claude agents
CHANGELOG.md                      optional until there are notable changes
CONTRIBUTING.md                   optional for solo work
.github/
  pull_request_template.md        recommended when using GitHub PRs
.claude/
  settings.json
  rules/
    documentation.md              optional; create when Markdown rules do not fit CLAUDE.md
    architecture.md               optional; create only as a thin pointer to guardrails
    unity-code.md                 optional; path-scoped C# rules if needed
  skills/
    session-handoff/              later, only if repeated handoff work needs a skill
      SKILL.md
    pre-pr-review/                later, only if repeated PR review work needs a skill
      SKILL.md
docs/
  INDEX.md
  product/
    game-design.md
    scope.md                      optional if scope does not fit game-design.md
  architecture/
    overview.md                   optional if guardrails already explain the shape
    implementation-guardrails.md
    adr/
      README.md                   create when first ADR is added
      0000-template.md            create when first ADR is added
  development/
    setup.md                      create if setup is more than README can hold
    build-and-test.md
    unity-workflow.md             optional
    ai-workflow.md                optional if CLAUDE.md is enough
  quality/
    test-strategy.md              optional until tests exist
    checklists.md                 optional
    performance-budget.md         optional until profiling matters
  assets/
    asset-ledger.md
  handoff/
    current-status.md
```

Do not create optional files as empty stubs. Create a file only when it has useful content, or when the file itself is a required checklist such as `asset-ledger.md` or `current-status.md`.

## 5. Bootstrap Minimum

For the first setup pass, create only the files that are needed to make the next coding session safe and understandable.

Core bootstrap files:

- `README.md`
- `CLAUDE.md`
- `.claude/settings.json`
- `docs/INDEX.md`
- `docs/product/game-design.md`
- `docs/architecture/implementation-guardrails.md`
- `docs/development/build-and-test.md`
- `docs/handoff/current-status.md`
- `docs/assets/asset-ledger.md`

Create these if they are immediately useful:

- `docs/development/setup.md` if setup needs more than a short README section.
- `.github/pull_request_template.md` if GitHub PRs will be used, or if a local PR checklist is useful.
- `.claude/rules/documentation.md` if Markdown rules would make `CLAUDE.md` too long.
- `.claude/rules/architecture.md` only as a short pointer to `docs/architecture/implementation-guardrails.md`.
- `docs/development/ai-workflow.md` if the AI workflow needs more detail than `CLAUDE.md`.

Create `AGENTS.md` only if the repository will also be used by Codex or another tool that reads it automatically.

Create Claude skills only after the basic docs exist and there is a repeated procedure worth extracting. Do not create skills during the first setup pass just because this guide mentions them.

## 6. Source Of Truth Rules

Every important fact must have one home:

| Fact type | Source of truth |
|---|---|
| Project summary and quick start | `README.md` |
| Claude Code session rules | `CLAUDE.md` |
| Codex or cross-agent entry rules | `AGENTS.md`, if used |
| Documentation map | `docs/INDEX.md` |
| Game rules and UX | `docs/product/game-design.md` |
| MVP, cuts, and non-goals | `docs/product/scope.md` |
| Architecture shape | `docs/architecture/overview.md` |
| Engineering guardrails | `docs/architecture/implementation-guardrails.md` |
| Architecture decisions | `docs/architecture/adr/` |
| Setup steps | `docs/development/setup.md` |
| Build and test commands | `docs/development/build-and-test.md` |
| Unity workflow | `docs/development/unity-workflow.md` |
| AI workflow | `docs/development/ai-workflow.md` |
| Test strategy | `docs/quality/test-strategy.md` |
| Performance targets | `docs/quality/performance-budget.md` |
| Asset sources and licenses | `docs/assets/asset-ledger.md` |
| Current active state | `docs/handoff/current-status.md` |
| Release notes | `CHANGELOG.md` |

If the same command or rule appears in more than one place, pick one source of truth and replace the duplicates with links.

## 7. `CLAUDE.md` Policy

`CLAUDE.md` is the primary Claude Code entry point. Claude reads it at the start of sessions, so it must be concise and useful every time.

Rules:

- Keep it under about 200 lines.
- Use direct commands and constraints.
- Put details in linked docs or `.claude/rules/`.
- Do not paste the full GDD into it.
- Do not paste the full architecture guide into it.
- Do not include long chat history.
- Do not include personal machine secrets or local paths unless they are intentionally project-wide.
- If a rule is hard safety, back it with `.claude/settings.json` or hooks.

Recommended `CLAUDE.md`:

````markdown
# Claude Code Instructions

## Project Snapshot

Pocket Grove is a small Unity AR prototype. The player places a tiny grove in AR and completes short rounds by collecting target motes.

## Start Every Session

1. Read `docs/INDEX.md`.
2. Read `docs/handoff/current-status.md`.
3. Read the docs relevant to the task.
4. Run `git status --short` before editing.
5. State what you are about to change before making broad edits.

## Must-Read Docs

- `docs/product/game-design.md`
- `docs/architecture/implementation-guardrails.md`
- `docs/development/build-and-test.md`
- `docs/development/ai-workflow.md`
- `docs/handoff/current-status.md`

## Repository Rules

- Keep project code and assets under `Assets/_Project/` unless an existing convention says otherwise.
- Track Unity `.meta` files.
- Do not edit generated `Library/`, `Temp/`, `Obj/`, or local build output.
- Do not add production dependencies without documenting why.
- Update docs in the same change when behavior, setup, architecture, tests, assets, or workflow changes.

## Architecture Rules

- Follow `docs/architecture/implementation-guardrails.md`.
- Prefer manual DI/composition root unless an ADR accepts a DI container.
- Do not introduce global service locators.
- Do not use scene-wide searches in runtime gameplay hot paths.
- Keep gameplay rules testable outside AR device code.

## Verification

Use `docs/development/build-and-test.md` as the source of truth.
Never claim a test passed unless it was run in this session or exact evidence is cited.

## Git Policy

- Read-only git is allowed: `git status`, `git diff`, `git log`.
- Do not run `git add`, `git commit`, `git push`, or destructive git commands unless the human explicitly asks.
- Propose Conventional Commit messages for human review.

## End Every Session

Update `docs/handoff/current-status.md` with outcome, files changed, checks run, blockers, and next actions.
````

## 8. `AGENTS.md` Policy

Use `AGENTS.md` only if the project will be opened by Codex or other agents that discover it automatically.

Avoid maintaining two independent agent manuals.

Recommended options:

### Option A: Claude-first project

Use `CLAUDE.md` as the main agent instruction file. Create a short `AGENTS.md` only for non-Claude agents:

````markdown
# AGENTS.md

This repository is Claude Code first. Read `CLAUDE.md` and the docs it links before editing.

Required start files:

- `CLAUDE.md`
- `docs/INDEX.md`
- `docs/handoff/current-status.md`
- `docs/product/game-design.md`
- `docs/architecture/implementation-guardrails.md`

Follow the same documentation, testing, and git policies described there.
````

### Option B: Shared cross-agent contract

If both Claude Code and Codex are used often, keep `AGENTS.md` as a short cross-agent contract and import it from `CLAUDE.md` with `@AGENTS.md`. Then put Claude-specific details in `CLAUDE.md` below the import.

Do not use this option if it causes duplication or confusion.

## 9. `.claude/rules/` Policy

Use `.claude/rules/` when instructions are too detailed for `CLAUDE.md` but still need to load regularly.

Possible files:

```text
.claude/rules/documentation.md
.claude/rules/architecture.md
.claude/rules/unity-code.md
```

Use unconditional rules for project-wide constraints. Use path-scoped rules for specific file types.

Do not mirror whole docs into `.claude/rules/`. A rule file should either add a short operational rule or point Claude to the real source of truth.

Example path-scoped Unity C# rule:

````markdown
---
paths:
  - "Assets/**/*.cs"
---

# Unity C# Rules

- Keep gameplay rules testable outside MonoBehaviours where practical.
- Do not use `GameObject.Find` or `FindObjectOfType` in runtime gameplay code.
- Use serialized references, composition root wiring, factories, or pools instead of hidden globals.
- Add or update tests when changing pure gameplay logic.
````

Example Markdown documentation rule:

````markdown
---
paths:
  - "*.md"
  - "docs/**/*.md"
---

# Documentation Rules

- Write project docs in English.
- Use relative links for repository files.
- Add `Last verified: YYYY-MM-DD` to setup, build, platform, and workflow docs.
- Do not duplicate commands. Link to `docs/development/build-and-test.md`.
- Do not add a new doc unless it is linked from `docs/INDEX.md`.
````

## 10. `.claude/settings.json` Policy

Use settings for hard safety boundaries and shared Claude Code configuration.

Recommended initial file:

````json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "deny": [
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git push *)",
      "Bash(git reset *)",
      "Bash(git checkout -- *)",
      "Bash(git restore *)",
      "Bash(git clean *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  }
}
````

Notes:

- Permission rules are enforced by Claude Code, not by the model.
- Deny rules take precedence over ask and allow rules.
- For complex command validation, use a `PreToolUse` hook instead of fragile argument matching.
- Do not enable bypass-style modes in shared project settings.
- Keep personal settings in `.claude/settings.local.json`, not in committed settings.
- The git deny rules above assume the human reviews, stages, commits, and pushes. If the project owner explicitly wants Claude Code to stage or open PRs, adjust this policy deliberately instead of copying it blindly.
- If the project uses Claude Code's PowerShell tool on Windows, verify the active tool name in `/permissions` and add matching deny rules there. Do not guess tool names in committed settings.

## 11. Hooks Policy

Use hooks only for actions that must be deterministic.

Do not add hooks in the first setup pass unless there is a real enforcement need. For this project, `permissions.deny` plus clear `CLAUDE.md` rules are enough to start.

Good hook use cases:

- Block dangerous commands that are hard to express with simple permission rules.
- Run a Markdown lint/check script after docs edits.
- Run a formatter after specific code edits.
- Scan the working tree at session stop for docs that must be updated.

Avoid hooks for:

- vague reminders,
- large hidden automation,
- commands that slow every turn,
- network calls without clear need,
- anything the developer cannot inspect.

If adding hooks, document them in `docs/development/ai-workflow.md`.

## 12. Skills Policy

Use Claude Code skills for repeatable procedures that should not live in `CLAUDE.md`.

Possible project skills:

```text
.claude/skills/session-handoff/SKILL.md
.claude/skills/pre-pr-review/SKILL.md
.claude/skills/unity-device-test/SKILL.md
```

Create a skill when:

- the same multi-step instruction gets pasted more than twice,
- a section of `CLAUDE.md` becomes a procedure,
- a task needs supporting templates or examples,
- a check should be invoked explicitly with `/skill-name`.

Do not create skills for one-line rules.
Do not create skills before the team has seen the same procedure repeat in real work.

Recommended `session-handoff` skill:

````markdown
---
name: session-handoff
description: Update project handoff after a coding or documentation session. Use before stopping work or when the user asks for a handoff.
disable-model-invocation: true
---

# Session Handoff

Update `docs/handoff/current-status.md`.

Include:

- date,
- branch or workspace context,
- objective,
- outcome,
- files changed,
- commands/checks run,
- exact test status,
- decisions made,
- blockers,
- next three actions.

Do not paste full logs.
Do not claim tests passed unless they were run in this session or exact evidence is cited.
````

Recommended `pre-pr-review` skill:

````markdown
---
name: pre-pr-review
description: Run a pre-PR self-review for scoped Unity changes. Use before proposing a commit or pull request.
disable-model-invocation: true
---

# Pre-PR Review

1. Read `docs/development/build-and-test.md`.
2. Check `git status --short`.
3. Review the diff.
4. Verify docs were updated or explicitly not needed.
5. Run applicable tests or state why they cannot be run.
6. Check for restricted Unity APIs if runtime code changed.
7. Summarize risks and propose a Conventional Commit message.
````

## 13. Subagents Policy

Project subagents are optional. Add them only after the base workflow is stable.

Use subagents for focused tasks such as:

- code review,
- docs audit,
- architecture risk review,
- test plan review.

Rules:

- Each subagent should have one focused purpose.
- Limit tool access.
- Check project subagents into version control only when they are useful to the team.
- Do not use subagents to bypass the main documentation and handoff process.

## 14. Core Documentation Files

### 14.1 `README.md`

The README is the front door.

It should include:

- project name,
- one-paragraph description,
- current status,
- Unity version,
- target platform,
- required tools,
- quick start,
- build/test pointer,
- documentation map pointer,
- contribution/AI workflow pointer.

It should not include:

- full game design,
- long architecture rules,
- full setup troubleshooting,
- chat history.

Template:

````markdown
# Pocket Grove

## What This Is

One paragraph.

## Current Status

Short status and latest handoff link.

## Requirements

- Unity version:
- Target platform:
- Required modules:

## Quick Start

See `docs/development/setup.md`.

## Build and Test

See `docs/development/build-and-test.md`.

## Documentation

Start with `docs/INDEX.md`.

## AI-Assisted Development

Claude Code must read `CLAUDE.md` and `docs/handoff/current-status.md` before editing.
````

### 14.2 `docs/INDEX.md`

The docs index is the map and freshness table.

Template:

````markdown
# Documentation Index

Last verified: YYYY-MM-DD

| Document | Purpose | Update when |
|---|---|---|
| `product/game-design.md` | Game rules, UX, content, MVP | Gameplay or UX changes |
| `architecture/implementation-guardrails.md` | Engineering rules | Architecture or code quality rules change |
| `development/setup.md` | First setup | Unity version, packages, platform setup change |
| `development/build-and-test.md` | Exact build/test commands | Commands or verification process changes |
| `handoff/current-status.md` | Current active state | End of every session |
| `assets/asset-ledger.md` | Third-party assets and licenses | Any external asset is added, removed, or modified |

## Start Here

1. `../README.md`
2. `handoff/current-status.md`
3. `product/game-design.md`
4. `architecture/implementation-guardrails.md`
5. `development/build-and-test.md`
````

Rule: no new doc file may be added unless it is linked from `docs/INDEX.md`.

### 14.3 `docs/product/game-design.md`

This is the playable design source of truth.

It should include:

- high concept,
- player goal,
- target session length,
- core loop,
- menus,
- controls,
- win/loss rules,
- scoring,
- feedback,
- content list,
- MVP cut line,
- non-goals,
- acceptance criteria.

It should not include implementation architecture except where game states or events affect design clarity.

### 14.4 `docs/product/scope.md`

This controls ambition.

Sections:

- MVP,
- demo build,
- stretch goals,
- explicit non-goals,
- accepted shortcuts,
- what "showable" means.

If a feature is not in this file, it is not part of the current plan.

### 14.5 `docs/architecture/overview.md`

This explains the system shape.

Include:

- module boundaries,
- dependency direction,
- state flow,
- scene/prefab ownership,
- data ownership,
- test boundaries.

Do not list every class unless that is truly needed.

### 14.6 `docs/architecture/implementation-guardrails.md`

This is the engineering contract.

It should include:

- dependency rules,
- DI/container policy,
- singleton policy,
- Unity API restrictions,
- object pooling rules,
- event policy,
- testing requirements,
- performance requirements,
- code review checklist.

For this project, adapt the existing guardrails document instead of rewriting from scratch.

### 14.7 `docs/architecture/adr/`

ADRs prevent every new session from rediscovering decisions.

Create an ADR for:

- Unity version baseline,
- AR stack,
- DI approach,
- scene structure,
- package additions,
- persistence approach,
- asset loading strategy,
- build pipeline,
- major guardrail deviations.

ADR template:

````markdown
# ADR 0000: Title

Status: Proposed  
Date: YYYY-MM-DD  
Decision owner: Project owner or developer

## Context

What problem requires a decision?

## Options Considered

- Option A
- Option B
- Option C

## Decision

What was chosen?

## Consequences

What becomes easier, harder, riskier, or more constrained?

## Follow-Up

What must be checked later?
````

Rules:

- One decision per ADR.
- Use status: `Proposed`, `Accepted`, `Deprecated`, or `Superseded`.
- Do not rewrite accepted ADRs except for typo or link fixes.
- Supersede changed decisions with a new ADR.

### 14.8 `docs/development/setup.md`

This describes clean checkout setup.

Include:

- Unity version,
- Unity Hub modules,
- Android/iOS setup,
- required SDK/NDK/JDK details if relevant,
- package restore expectations,
- first open steps,
- common setup problems,
- what should be ignored by version control.

Use exact versions where possible.

### 14.9 `docs/development/build-and-test.md`

This is the command source of truth.

Include:

- how to run Edit Mode tests,
- how to run Play Mode tests,
- how to build,
- how to run on device,
- what manual Unity steps are required,
- what counts as success,
- what cannot be automated yet.

Every command must be copy-pasteable from the Unity repository root.

### 14.10 `docs/development/unity-workflow.md`

Include Unity-specific habits:

- asset folder conventions,
- prefab workflow,
- scene workflow,
- package addition policy,
- `.meta` file handling,
- moving/renaming assets in Unity,
- Git LFS policy if large files appear,
- Unity MCP policy if used.

### 14.11 `docs/development/ai-workflow.md`

This explains how AI sessions should work.

Include:

- start protocol,
- edit protocol,
- verification protocol,
- docs update protocol,
- handoff protocol,
- how to propose commits,
- how to handle uncertainty,
- when to create ADRs,
- what to do when another agent is active.

### 14.12 `docs/quality/test-strategy.md`

Separate:

- pure C# gameplay tests,
- Unity Edit Mode tests,
- Play Mode smoke tests,
- manual AR device tests,
- performance checks,
- demo acceptance tests.

### 14.13 `docs/quality/performance-budget.md`

Include:

- target devices,
- target FPS,
- max active spawned objects,
- VFX limits,
- GC allocation goal,
- profiling steps,
- downgrade strategy when performance fails.

### 14.14 `docs/assets/asset-ledger.md`

Every third-party asset must be recorded.

Template:

````markdown
# Asset Ledger

Last verified: YYYY-MM-DD

| Asset | Source URL | Author | License | Download date | Local path | Modifications | Notes |
|---|---|---|---|---|---|---|---|
````

Rules:

- Do not import unclear-license assets into a showable build.
- Record AI-generated assets with tool, prompt summary, date, and rights note.
- Record modifications.
- Keep local paths accurate.

### 14.15 `docs/handoff/current-status.md`

This is the active memory bridge between sessions.

Template:

````markdown
# Current Status

Last updated: YYYY-MM-DD  
Updated by: human or AI session name  
Branch/context: branch name or local workspace state

## Current Objective

One or two sentences.

## Status

What is done, partially done, and not started.

## Files Changed Recently

- `path/to/file`

## Checks Run

- `command`: result
- Manual Unity check: result
- Device check: result or not run with reason

## Decisions Made

- Decision summary, with ADR link if applicable.

## Blockers

- Blocker, owner, next action.

## Next Actions

1. Next concrete task.
2. Next concrete task.
3. Next concrete task.

## Notes for Next Session

Short warnings only. No long logs.
````

Rules:

- Update at the end of every coding or documentation session.
- Keep it current, not historical.
- Do not paste full logs.
- Do not claim tests passed unless they were run or exact evidence exists.
- Use absolute dates.

## 15. Pull Request Workflow

Use PR discipline even for solo development.

Every logical change must answer:

- What changed?
- Why?
- How was it verified?
- What docs changed?
- What risks remain?
- What is the proposed Conventional Commit message?

Recommended `.github/pull_request_template.md`:

````markdown
## Summary

What changed and why?

## Scope

- [ ] Gameplay
- [ ] AR
- [ ] UI
- [ ] Audio/VFX
- [ ] Persistence
- [ ] Architecture
- [ ] Assets
- [ ] Build/CI
- [ ] Documentation

## Verification

- [ ] Edit Mode tests run
- [ ] Play Mode tests run
- [ ] Unity compile checked
- [ ] Device test run
- [ ] Screenshot/video attached for UI or AR changes

Commands/results:

```text
paste concise results here
```

## Documentation

- [ ] README updated or confirmed unchanged
- [ ] CLAUDE.md updated or confirmed unchanged
- [ ] Game design updated or confirmed unchanged
- [ ] Architecture docs or ADR updated or confirmed unchanged
- [ ] Build/test docs updated or confirmed unchanged
- [ ] Asset ledger updated or confirmed unchanged
- [ ] Changelog updated or confirmed unchanged
- [ ] Handoff status updated

## Risks

Known risks, limitations, and follow-up tasks.
````

## 16. Commit And Changelog Policy

Use Conventional Commits:

```text
feat(ar): add plane placement preview
fix(ui): prevent pause button during tracking loss
docs(architecture): record manual DI decision
test(gameplay): cover bloom progress completion
```

Recommended types:

- `feat`
- `fix`
- `docs`
- `test`
- `refactor`
- `perf`
- `build`
- `chore`
- `ci`
- `style`

Use `CHANGELOG.md` for notable changes, not raw commits.

Recommended changelog skeleton:

````markdown
# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Added

### Changed

### Fixed

### Removed
````

Update the changelog when a change affects:

- playable behavior,
- demo readiness,
- build process,
- public workflow,
- major architecture,
- imported assets,
- performance constraints.

## 17. Unity Version-Control Rules

Document and enforce these rules:

Track:

- `Assets/`
- `Packages/`
- `ProjectSettings/`
- `.meta` files
- project documentation
- `.github/` templates
- `.claude/` shared configuration after review

Ignore:

- `Library/`
- `Temp/`
- `Obj/`
- `Logs/`
- `UserSettings/`
- local build outputs unless intentionally versioned
- IDE-generated files unless the team explicitly wants them

Unity asset rules:

- Use Visible Meta Files or the current Unity equivalent.
- Move or rename assets in the Unity Editor when possible.
- If moving assets outside Unity, move the paired `.meta` files too.
- Do not commit orphan `.meta` files.
- Do not create empty folders just to create structure.
- Keep first-party project assets separate from third-party imports.

## 18. AI Session Protocol

### Start Every Session

The next chat must:

1. Confirm the repository root.
2. Read `CLAUDE.md`.
3. Read `docs/INDEX.md`.
4. Read `docs/handoff/current-status.md`.
5. Read the task-relevant docs.
6. Run read-only `git status --short`.
7. State the intended changes and files.
8. Ask before editing if another agent is active or the workspace appears unexpectedly dirty.

### During The Session

The chat must:

- keep changes scoped,
- avoid unrelated refactors,
- update docs with behavior/setup/architecture/test/asset changes,
- create ADRs for significant decisions,
- avoid duplicate facts,
- run the documented checks that apply,
- communicate blockers early.

### End Every Session

The chat must:

1. Update `docs/handoff/current-status.md`.
2. State tests/checks run.
3. State tests/checks not run and why.
4. Summarize changed files.
5. List remaining blockers.
6. Propose next actions.
7. Propose a Conventional Commit message if the human controls commits.

## 19. Documentation Maintenance Cadence

Use this cadence:

| Event | Required documentation action |
|---|---|
| End of every AI session | Update `docs/handoff/current-status.md` |
| Gameplay rule change | Update `docs/product/game-design.md` |
| Scope change | Update `docs/product/scope.md` |
| Architecture decision | Add or supersede an ADR |
| Code-quality rule change | Update implementation guardrails or `.claude/rules/` |
| Build/test command change | Update `docs/development/build-and-test.md` |
| Unity/package/platform setup change | Update `docs/development/setup.md` |
| New external asset | Update `docs/assets/asset-ledger.md` |
| Demo/release-visible change | Update `CHANGELOG.md` |
| New doc file | Link it from `docs/INDEX.md` |

## 20. Bootstrap Order For The Next Chat

The next chat should configure the Unity repository in this order:

1. Confirm no other agent is actively editing the Unity project.
2. Confirm the Unity repository root.
3. Create the core bootstrap docs only.
4. Create `CLAUDE.md`.
5. Create `.claude/settings.json` with safe deny rules if the human controls git writes.
6. Create `docs/INDEX.md`.
7. Adapt the existing GDD into `docs/product/game-design.md`.
8. Adapt the existing implementation guardrails into `docs/architecture/implementation-guardrails.md`.
9. Create `docs/development/build-and-test.md`.
10. Create `docs/handoff/current-status.md`.
11. Create `docs/assets/asset-ledger.md`.
12. Add `docs/development/setup.md` if setup does not fit cleanly in README.
13. Add `.github/pull_request_template.md` if GitHub PRs or PR-style local review will be used.
14. Add `.claude/rules/` files only if they reduce `CLAUDE.md` bloat.
15. Create ADR 0001 only if architecture choices are already known.
16. Run a docs audit before feature coding.

Large feature implementation may begin once the core bootstrap docs exist and the next session can safely answer: what is the game, what are the engineering guardrails, how do I build/test, what is the current state, and what must I not do. Optional docs, skills, subagents, and hooks can wait.

## 21. Red Flags

Stop and fix the docs system if:

- `CLAUDE.md` becomes the whole project manual.
- `CLAUDE.md`, `AGENTS.md`, and docs disagree.
- A future chat relies on chat history instead of `docs/handoff/current-status.md`.
- Build commands are duplicated and differ.
- An architecture decision exists only in chat.
- New assets have no license record.
- PRs change behavior but do not update docs.
- The changelog becomes a raw commit dump.
- Session notes become a long diary instead of current state.
- Tests are claimed without commands or evidence.
- A new doc is not linked from `docs/INDEX.md`.
- `TODO` appears without owner, date, and next action.
- Hard safety rules are written only as soft instructions.

## 22. Acceptance Criteria

The documentation system is acceptable when:

- A new developer can start from `README.md` and `docs/INDEX.md`.
- Claude Code has a concise `CLAUDE.md` and modular `.claude/rules/`.
- Dangerous actions are blocked by permissions or hooks where needed.
- The current state is recoverable from `docs/handoff/current-status.md`.
- Game design and architecture have clear source files.
- Build and test steps are exact enough to run.
- Important decisions are captured as ADRs.
- Assets have source and license records.
- PRs cannot silently skip documentation impact.
- The docs are small enough to maintain during a short Unity prototype.

## 23. Practical Self-Audit

This guide was checked for practical consistency:

- It does not require a documentation website.
- It does not require dozens of empty files on day one.
- It separates core bootstrap files from optional later files.
- It makes Claude Code setup explicit.
- It separates soft model guidance from hard permissions/hooks.
- It supports Codex/other agents without letting `AGENTS.md` conflict with `CLAUDE.md`.
- It gives handoff rules that survive chat resets.
- It uses ADRs only for meaningful decisions.
- It includes Unity-specific version-control safeguards.
- It makes documentation updates part of PR discipline.
- It keeps the next chat free to choose the technical Unity implementation.

## 24. References

- Claude Code memory and `CLAUDE.md`: https://code.claude.com/docs/en/memory
- Claude Code settings: https://code.claude.com/docs/en/settings
- Claude Code permissions: https://code.claude.com/docs/en/permissions
- Claude Code hooks: https://code.claude.com/docs/en/hooks-guide
- Claude Code skills: https://code.claude.com/docs/en/skills
- Claude Code subagents: https://code.claude.com/docs/en/sub-agents
- OpenAI Codex `AGENTS.md`: https://developers.openai.com/codex/guides/agents-md
- Write the Docs, Docs as Code: https://www.writethedocs.org/guide/docs-as-code/
- GitHub README docs: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes
- GitHub contributing guidelines: https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/setting-guidelines-for-repository-contributors
- GitHub pull request templates: https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository
- Diataxis documentation framework: https://diataxis.fr/
- Markdown Architectural Decision Records: https://adr.github.io/madr/
- Keep a Changelog: https://keepachangelog.com/en/1.1.0/
- Conventional Commits: https://www.conventionalcommits.org/en/v1.0.0/
- Unity external version control: https://docs.unity.cn/2021.2/Documentation/Manual/ExternalVersionControlSystemSupport.html
- Unity ignore files: https://docs.unity.com/en-us/unity-version-control/ignore-files
- Unity project organization: https://unity.com/how-to/organizing-your-project
