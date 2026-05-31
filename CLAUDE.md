# Pocket Grove — AR test project (Unity + MCP)

## What this is
Unity **6.3 LTS** (`6000.3.16f1`), **URP** (Universal 3D). Target: **Android / ARCore** AR app —
a camera + plane-tracking baseline, extended per spec. Editor automation via the **MCP for Unity**
server. 3D content authored in **Blender**, imported as `.fbx`/`.glb`.

## Repo layout & where to run
- **Git repo root = this folder** (`Pocket Grove/`); it holds `Assets/ Packages/ ProjectSettings/`.
- **Run Claude Code from this folder** so `CLAUDE.md` and `.claude/settings.json` load and apply.
- Planning docs (`AR-Unity-MCP-Setup-Plan.md`, `Claude-Code-Rules-Guide.md`) live one level up in
  `MyArProj/` and are **not** part of this repo.

## Division of labor
- **Assistant:** edit C#/text files directly; do Unity editor ops via **MCP** (GameObjects,
  components, scenes, materials, Test Runner, reading the Console); add package deps by editing
  `Packages/manifest.json`. **Read-only git only** (`git status`, `git diff`, `git log`).
- **Human:** GUI toggles unreliable via MCP (platform switch, parts of Player Settings, URP
  renderer feature), device build/run, and **all git commits**.

## Hard rule — git
- **Never** run `git commit`, `git push`, or `git add` (also denied in `.claude/settings.json`).
- After each logical unit of work: stop, summarize, and **propose a commit message** (format below).
  Human reviews the diff and commits.

## Commit messages — Conventional Commits (always propose one, ready to paste)
Format:
```
<type>(<scope>): <imperative summary, max 72 chars>

<optional body — what & why, as bullets>
```
- **Types:** `feat` `fix` `chore` `build` `docs` `refactor` `test` `perf` `style` `ci`
- **Scopes (typical here):** `ar` `mcp` `scene` `ui` `core` `deps` `build`
- **Language:** English summary.
- **Examples:**
  - `chore: initial project baseline (Unity 6.3 LTS, URP)`
  - `build(deps): add AR Foundation 6.x + ARCore XR plugin`
  - `feat(ar): add AR Session and XR Origin to the main scene`
  - `fix(ar): keep OpenGLES3 in Graphics APIs to avoid black camera`

## Branching — GitFlow-lite
- `dev` — default integration branch.
- `feature/<slug>`, `fix/<slug>` for focused work; merge back into `dev`.
- No `main`/`release/*`/`hotfix/*` until we actually ship (keep it simple).
- When a change warrants its own branch, the assistant proposes the branch name too.

## Conventions (apply as needed, not "just in case")
- C#: `PascalCase` types/methods, `camelCase` fields; one public type per file (= filename).
- Input System (not legacy `Input`); ScriptableObjects for config; Prefabs for instantiated objects.
- Keep our code/content under `Assets/_Project/...`, separate from imported packages.
- Introduce a new abstraction only on its **second** real use.

## "Ready for review" =
Compiles with no Console errors (checked via MCP), Project Validation passes, and — where
applicable — play-mode/build works.
