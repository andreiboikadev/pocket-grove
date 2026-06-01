---
paths:
  - "Assets/**/*.cs"
---

# Unity C# Rules — Pocket Grove

Auto-loads when you touch C#. This is the operational checklist; full rationale and the code-review
checklist live in [implementation-guardrails.md](../../docs/architecture/implementation-guardrails.md).

## Architecture in one line

Thin Unity layer over testable plain-C# rules. MonoBehaviours are **adapters** (Unity lifecycle,
serialized scene refs, input, AR managers, prefabs, animation). Gameplay rules — score, timer,
target-color selection, bloom progress, spawn planning — are **plain C# classes**: no `MonoBehaviour`,
no `UnityEngine` statics inside them.

## Forbidden in runtime / gameplay code (needs a written justification to break)

- `GameObject.Find`, `FindWithTag`, `FindObjectOfType`, `FindObjectsOfType`, `FindAnyObjectByType`
- `GetComponent` inside `Update` or per-frame loops; LINQ inside per-frame gameplay loops
- Any new singleton or service locator (`GameManager.Instance`, `AudioManager.Instance`, …)
- `SendMessage` / string-based dispatch; string event names
- `Resources.Load` for normal assets; `Instantiate`/`Destroy` for motes during a round (pool instead)
- Avoidable managed allocations during an active round (reuse buffers, e.g. one `List<ARRaycastHit>`)

## Allowed at setup time only

`GetComponent` in `Awake`/`Start`; `Instantiate` during pool warmup or one-time scene setup;
tags/layers for collision filtering (not repeated tag *searches*); `FindAnyObjectByType` only in
editor-only tooling.

## Dependency wiring — manual DI is the MVP default (no container)

- One composition root wires everything; constructor-inject plain C# services.
- MonoBehaviours receive scene/prefab refs via `[SerializeField]`, then pass deps to services.
- Runtime objects (motes, VFX) get deps from a factory/pool on spawn — never by searching the scene.
- **VContainer is the only sanctioned container** if one is ever justified (Zenject/Extenject are
  abandoned). Do not add it without an ADR. Expected custom singletons for this MVP: **zero**.

## Pooling

Pool motes and repeated VFX via `UnityEngine.Pool.ObjectPool<T>`. On release, reset visual/color/
collider/lifetime/subscriptions/parent; guard double-release in dev builds. Do **not** pool the single
seed, the grove root, or static props.

## Events

Typed events only; small immutable payloads (`readonly struct`). Subscribe on enable, unsubscribe on
disable/dispose. Use feature-scoped channels, not one global `EventBus`. For a one-to-one relationship,
prefer a direct method call over an event.

## AR — AR Foundation 6.5 (verify against the installed package before relying on an API)

- Managers sit on the **XR Origin** GameObject (except AR Session). Enable only what the current state
  needs; **disable `ARPlaneManager` after placement** (official perf guidance).
- Anchor with `ARAnchorManager.TryAddAnchorAsync` (async). `AddAnchor` is obsolete; use `AttachAnchor`
  only after checking `descriptor.supportsTrackableAttachments`.
- Plane hits: `ARRaycastManager.Raycast(screenPoint, hits, TrackableType.PlaneWithinPolygon)`.
  Mote hits: `Physics.Raycast` against colliders on a dedicated mote layer.
- Touch input: Input System `EnhancedTouch` (call `EnhancedTouchSupport.Enable()` first). Keep
  `ARInputManager` — it provides the device pose, it is not a touch API.

## Naming

`*Controller` (coordinates a feature) · `*Service` (reusable, no scene ownership) · `*Presenter`
(state → view) · `*View` (owns scene objects/visuals) · `*Config`/`*Definition` (ScriptableObject
data) · `*State` · `*EventChannel` · `*CompositionRoot`/`*Installer` (wiring only, no rules).
Avoid `GameManager`, `Manager`, `Utils`, `Helper`, `Data`. One public type per file (= filename);
PascalCase types/methods, camelCase fields.

## Tests

Pure rule classes don't inherit `MonoBehaviour` and don't read `Time`/`Random`/scene objects directly
(inject a clock and a seeded random). Add or extend EditMode tests when you change score, timer,
target-selection, bloom, or spawn logic.

## Size limits — refactor when crossed

A class over ~300 lines, or a method over ~50 lines with mixed responsibilities. No single script
mixes UI + AR + score + spawn + audio + persistence.
