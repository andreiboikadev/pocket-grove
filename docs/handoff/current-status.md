# Current Status

Last updated: 2026-06-01
Updated by: Claude Code (docs consolidation + testing-gate session)
Branch/context: local workspace (not yet committed — human reviews and commits)

## Current objective

All project documentation lives in-repo under `docs/` (single source of truth); external concept docs
were consolidated in and then deleted from the parent folder. A per-mechanic testing/regression gate is
now defined and mirrored across the rule layers. Next: begin the gameplay vertical slice.

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
- **Testing gate (this session):** per-mechanic Definition of Done — unit tests in the same change
  (pure rules), a smoke pass before done (MCP / XR Simulation / device, MCP where it fits), and a
  no-regression re-run of the full suite. Defined in guardrails §17/§21/§22 and mirrored into
  `CLAUDE.md` ("Ready for review"), `.claude/rules/unity-csharp.md`, and `build-and-test.md`.
- **Not started:** all gameplay code. No scripts under `Assets/_Project/Scripts/` yet. No
  ScriptableObject configs, prefabs, or tests yet. No DI container (manual DI by ADR 0001).

## Files changed recently

- `docs/product/game-design.md`, `docs/architecture/implementation-guardrails.md` — replaced with the
  full finalized versions (consolidated from external).
- `docs/reference/claude-code-rules.md`, `docs/reference/documentation-system.md` — new (consolidated).
- `docs/INDEX.md`, `CLAUDE.md` — updated for in-repo single source of truth.
- Test gate: `docs/architecture/implementation-guardrails.md` (§17/§21/§22) + mirrors in `CLAUDE.md`,
  `.claude/rules/unity-csharp.md`, `docs/development/build-and-test.md`.
- Earlier this session: `.claude/settings.json`, `.claude/rules/*`, `README.md`, ADRs, build-and-test, asset-ledger.

## Checks run

- Docs-only session: no compile/tests required. None run.
- Unity Editor not opened this session; no MCP operations performed.

## Decisions made

- Technical baseline recorded in [ADR 0001](../architecture/adr/0001-tech-baseline.md): Unity 6.3 LTS,
  URP, AR Foundation 6.5 + ARCore (Android), new Input System, MCP automation, **manual DI**, single
  scene, XR Simulation for Editor testing, pooling via `ObjectPool<T>`.
- Documentation kept to a **lean bootstrap**; deferred docs are listed in `docs/INDEX.md`.
- **In-repo docs are the single source of truth.** The external parent-folder concept docs were
  consolidated in and then **deleted** by the owner; nothing in the repo depends on them.
- **Per-mechanic testing/regression gate is policy** (guardrails §17), mirrored into the always-on and
  auto-loaded rule layers so it holds regardless of which doc a session opens.

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

Read `CLAUDE.md` + this file first. All docs are in-repo (the external parent-folder `.md` files were
deleted). Every mechanic follows the test gate (guardrails §17): unit tests in-change, smoke before
done, no regression. Hard rules block all git writes (Bash and PowerShell) — propose a commit message,
don't commit. Verify any AR API against the installed AR Foundation 6.5 package before relying on it.
