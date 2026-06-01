# ADR 0001: Technical baseline

Status: Accepted
Date: 2026-06-01
Decision owner: Project owner

## Context

Pocket Grove is a ~1-day MVP mobile-AR game (see [game design](../../product/game-design.md)). The
Unity project was already scaffolded with an AR base before gameplay work began. This ADR records the
baseline that is **already in place and accepted**, so future sessions don't re-decide it. Versions
here are the source of truth; cross-check against `Packages/manifest.json` and
`ProjectSettings/ProjectVersion.txt`.

## Decision

- **Engine:** Unity **6.3 LTS** (`6000.3.16f1`) — current Unity 6 LTS (supported to Dec 2027). Use this
  exact version.
- **Render pipeline:** **URP** (Universal). Mobile and PC renderer/asset variants exist under
  `Assets/Settings/`. Mobile build uses the mobile URP asset.
- **AR stack:** **AR Foundation 6.5** + **ARCore XR Plugin 6.5**, target **Android**. Manager-based
  architecture on a single **XR Origin** (not the deprecated `ARSessionOrigin`).
- **Input:** new **Input System** (1.19); `EnhancedTouch` for taps. No legacy `Input`.
- **Editor automation:** **MCP for Unity** (CoplayDev) for GameObject/component/scene/material/Test
  Runner/Console operations.
- **Editor AR testing:** **XR Simulation** (ships with AR Foundation) — plane detection and placement
  are exercised in Editor Play mode without a device.
- **Dependency injection:** **manual DI** (composition root + constructor injection + serialized refs +
  factories/pools). **No container** for MVP. VContainer is the only sanctioned container if one is
  later justified — via its own ADR (Zenject/Extenject are rejected as abandoned).
- **Scenes:** a **single** `Assets/_Project/Scenes/Gameplay.unity` for the MVP (the only build scene).
- **Pooling:** `UnityEngine.Pool.ObjectPool<T>` for motes and repeated VFX.
- **3D content:** authored in **Blender**, imported as `.fbx`/`.glb`.

## Consequences

- Easier: no DI-container learning curve or reflection cost; one scene to reason about; in-Editor AR
  iteration via XR Simulation keeps the loop fast without a phone.
- Constrained: gameplay must stay testable as plain C# (manual DI demands explicit wiring — a feature,
  not a bug); adopting a DI container or splitting scenes later requires a new ADR.
- **Android build note:** disable **Auto Graphics API** and set Graphics APIs explicitly (ARCore 6.x
  supports Vulkan and OpenGLES3; keep OpenGLES3 as a fallback if ARCore Requirement = Optional);
  minimum SDK **API 24+**; confirm via **Project Validation**. The old "force OpenGLES3 to avoid a black
  camera" rule is obsolete — the real fix is disabling Auto Graphics API and validating.

## Follow-Up

- Revisit DI if the service graph grows non-trivially beyond the MVP (would be ADR 0002).
- Revisit single-scene if additive loading becomes worthwhile.
- Record the chosen anchor strategy (free anchor via `TryAddAnchorAsync` vs plane attachment) once
  placement is implemented and tested on device.
