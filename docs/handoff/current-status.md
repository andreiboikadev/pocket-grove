# Current Status

Last updated: 2026-06-01
Updated by: Claude Code (docs consolidation session)
Branch/context: local workspace (not yet committed — human reviews and commits)

## Current objective

All project documentation now lives in-repo under `docs/` and is the single source of truth; the
external concept docs were consolidated into the repo. Next: begin the gameplay vertical slice.

## Status

- **Done:** AR baseline already configured (Unity 6.3 LTS, URP, AR Foundation 6.5 + ARCore, Input
  System, XR Simulation, single `Gameplay` scene). Docs/rules bootstrap created this session —
  corrected hard rules (`.claude/settings.json`, dual-shell git deny), path-scoped `.claude/rules/`
  (C# architecture + docs hygiene), merged `CLAUDE.md`, `README.md`, `docs/INDEX.md`,
  `game-design.md`, `implementation-guardrails.md`, ADR 0000/0001, `build-and-test.md`,
  `asset-ledger.md`, this file.
- **Consolidated (this session):** the 4 external concept docs moved in-repo — full GDD →
  `docs/product/game-design.md`, full guardrails → `docs/architecture/implementation-guardrails.md`,
  Claude-Code rules guide + documentation-system guide → `docs/reference/`. In-repo docs are now the
  single source of truth; the parent-folder copies are superseded.
- **Not started:** all gameplay code. No scripts under `Assets/_Project/Scripts/` yet. No
  ScriptableObject configs, prefabs, or tests yet. No DI container (manual DI by ADR 0001).

## Files changed recently

- `docs/product/game-design.md`, `docs/architecture/implementation-guardrails.md` — replaced with the
  full finalized versions (consolidated from external).
- `docs/reference/claude-code-rules.md`, `docs/reference/documentation-system.md` — new (consolidated).
- `docs/INDEX.md`, `CLAUDE.md` — updated for in-repo single source of truth.
- Earlier this session: `.claude/settings.json`, `.claude/rules/*`, `README.md`, ADRs, build-and-test, asset-ledger.

## Checks run

- Docs-only session: no compile/tests required. None run.
- Unity Editor not opened this session; no MCP operations performed.

## Decisions made

- Technical baseline recorded in [ADR 0001](../architecture/adr/0001-tech-baseline.md): Unity 6.3 LTS,
  URP, AR Foundation 6.5 + ARCore (Android), new Input System, MCP automation, **manual DI**, single
  scene, XR Simulation for Editor testing, pooling via `ObjectPool<T>`.
- Documentation kept to a **lean bootstrap**; deferred docs are listed in `docs/INDEX.md`.
- **In-repo docs are the single source of truth.** External parent-folder concept docs are superseded
  and safe to delete; nothing in the repo depends on them.

## Blockers

- None.

## Next actions

1. **Pure rules + tests first.** Add `MoteColor`, a `RoundConfig` ScriptableObject, and plain-C#
   `TargetColorSelector` / `ScoreService` / `RoundTimer` / `BloomProgress` / `MoteSpawnPlanner` under
   `Assets/_Project/Scripts/Gameplay`, with EditMode tests (guardrails §13).
2. **Placement vertical slice** in the `Gameplay` scene: `ARPlacementController` (raycast →
   `TryAddAnchorAsync`), reticle, place/replace; verify under **XR Simulation**.
3. **Composition root** wiring menu → placement → start round → tap primitive motes → results, using
   serialized refs + manual DI; pool primitive motes.

## Notes for next session

Read `CLAUDE.md` + this file first. All docs are in-repo now — do not rely on the parent folder's
external `.md` files (superseded; the human may delete them). Hard rules block all git writes (Bash and
PowerShell) — propose a commit message, don't commit. Verify any AR API against the installed AR
Foundation 6.5 package before relying on it.
