# Pocket Grove — Claude Code instructions

## Project snapshot
Unity **6.3 LTS** (`6000.3.16f1`), **URP** (Universal 3D). Building the GDD's **Pocket Grove**: a
cozy mobile-AR collection game — place a tiny grove on a real surface, tap motes that match the
seed's color, fill the bloom before the timer ends. Target: **Android / ARCore** via **AR Foundation
6.5**. New **Input System**. Editor automation via the **MCP for Unity** server. 3D content authored
in **Blender**, imported as `.fbx`/`.glb`. Aim: a playable MVP in ~1 focused day.

- Git repo root = this folder; it holds `Assets/ Packages/ ProjectSettings/`. Run Claude Code here.
- **All project docs live in-repo under `docs/` — the single source of truth** (see below). Design and
  architecture were consolidated here from earlier external files; do not rely on anything outside the repo.

## Start every session
1. Read `docs/INDEX.md` and `docs/handoff/current-status.md`.
2. Read the docs relevant to the task (game design, guardrails, build & test).
3. Run `git status --short` before editing; state what you're about to change before broad edits.
4. If the working tree looks unexpectedly dirty or another agent may be active, ask before editing.

## Must-read docs (source of truth — don't re-derive design/architecture from chat history)
- `docs/product/game-design.md` — what to build (loop, rules, scoring, screens, MVP cut line).
- `docs/architecture/implementation-guardrails.md` — how to build it; decisions in `docs/architecture/adr/`.
- `docs/development/build-and-test.md` — exact build / test / verify commands (the only command source).
- `docs/handoff/current-status.md` — where we are right now.

## Architecture must-knows (full detail auto-loads from `.claude/rules/unity-csharp.md` when editing C#)
Path-scoped rules load when you *read* a matching file, so keep these in mind even before opening code:
- Thin MonoBehaviour **adapters** over testable **plain-C# gameplay rules**. Manual DI via **one
  composition root**; **no new singletons** (expected custom singletons = 0).
- Forbidden in runtime: `GameObject.Find`/`FindObjectOfType`, per-frame `GetComponent`/LINQ,
  `SendMessage`, `Resources.Load`, `Instantiate`/`Destroy` of motes mid-round — pool via `ObjectPool<T>`.
- AR Foundation **6.5**: anchor via `ARAnchorManager.TryAddAnchorAsync`; **disable `ARPlaneManager`
  after placement**; taps via Input System `EnhancedTouch`. Verify any AR API against the installed package.
- **VContainer** is the only sanctioned DI container, and only with an ADR — no Zenject/Extenject.

## Division of labor
- **Assistant:** edit C#/text files directly; Unity editor ops via **MCP** (GameObjects, components,
  scenes, materials, Test Runner, reading the Console); add package deps by editing
  `Packages/manifest.json`. **Read-only git only** (`git status`, `git diff`, `git log`, `git show`).
- **Human:** GUI toggles unreliable via MCP (platform switch, parts of Player Settings, URP renderer
  feature, **Graphics API selection**), device build/run, and **all git commits**.

## Hard rule — git (also enforced in `.claude/settings.json` for Bash *and* PowerShell)
**Never** run `git add`, `git commit`, `git push` (or `reset`/`restore`/`checkout`/`switch`/`clean`/
`rebase`/`merge`). After each logical unit of work: stop, summarize, and **propose a commit message**.
The human reviews the diff and commits.

## Commit messages — Conventional Commits (always propose one, ready to paste)
```
<type>(<scope>): <imperative summary, ~50 chars, hard max 72>

<optional body — what & why, as bullets; omit if obvious>
```
- **Types:** `feat` `fix` `chore` `build` `docs` `refactor` `test` `perf` `style` `ci`
- **Scopes (typical here):** `ar` `mcp` `scene` `ui` `core` `gameplay` `deps` `build` `docs`
- **Language:** English summary. Add a body only for non-obvious *why*.
- **Examples:**
  - `chore: initial project baseline (Unity 6.3 LTS, URP)`
  - `build(deps): add AR Foundation 6.5 + ARCore XR plugin`
  - `feat(ar): add AR Session and XR Origin to the Gameplay scene`
  - `fix(ar): disable Auto Graphics API and set APIs explicitly to fix black camera`

## Branching — GitFlow-lite
- `dev` — default integration branch. `feature/<slug>`, `fix/<slug>` for focused work, merged into `dev`.
- No `main`/`release/*`/`hotfix/*` until we actually ship. When a change warrants its own branch, the
  assistant proposes the branch name (the human creates and switches it).

## Conventions (apply as needed, not "just in case")
- New **Input System** (not legacy `Input`); **ScriptableObjects** for config; **Prefabs** for
  instantiated objects.
- Keep our code/content under `Assets/_Project/...`, separate from imported packages.
- Introduce a new abstraction only on its **second** real use. (C# style detail: `.claude/rules/unity-csharp.md`.)

## "Ready for review" =
Compiles with no Console errors (checked via MCP), Project Validation passes, applicable EditMode/
PlayMode tests pass, and — where applicable — XR Simulation (Editor) or a device check works.

## End every session
Update `docs/handoff/current-status.md`: outcome, files changed, checks run + results, decisions
(link an ADR if one was made), blockers, next 3 actions. Never claim a test passed unless it ran this
session.
