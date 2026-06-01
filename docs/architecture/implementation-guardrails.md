# Unity AR Implementation Guardrails for Pocket Grove

> **Purpose.** This document is an implementation contract for the Pocket Grove AR prototype. It exists to prevent low-quality LLM-generated Unity code: giant scripts, hidden global state, excessive singletons, per-frame scene searches, untestable gameplay logic, and architecture that collapses as soon as the prototype grows.
>
> **Scope.** These rules apply after the technical chat chooses the concrete Unity/AR setup. They do not replace the game design document. They define how the code should be structured while implementing it.
>
> **Companion document:** `game-design.md`

---

## 1. Non-Negotiable Engineering Goals

The project should be small, but not sloppy.

The implementation must optimize for:

- **Clear ownership:** each system has one reason to change.
- **Low coupling:** gameplay, AR placement, UI, audio, and persistence do not directly own each other.
- **High cohesion:** related rules live together.
- **Inspectable behavior:** important runtime state can be seen in the Inspector or in a small debug view during development.
- **Testable gameplay logic:** scoring, target-color selection, timer behavior, and spawn planning can be tested without a real AR device.
- **Mobile AR performance:** no avoidable per-frame allocations, no per-frame scene searches, no uncontrolled instantiate/destroy loops.
- **Fast MVP delivery:** use simple architecture that protects the codebase; do not introduce enterprise patterns that slow down a one-day prototype.

The implementation must not optimize for:

- "Everything is a manager."
- "Everything is a singleton."
- "Everything talks through one global event bus."
- "All logic lives in MonoBehaviour.Update."
- "It works once on my phone, so the architecture is fine."

---

## 2. Source-Backed Principles

These rules are based on official Unity and Microsoft guidance. The implementation chat should prefer these sources over random blog posts.

| Topic | Source | Practical rule for this project |
|---|---|---|
| ScriptableObject architecture | https://unity.com/how-to/architect-game-code-scriptable-objects | Use ScriptableObjects for stable config, event channels, and designer-editable data. Do not use them as uncontrolled mutable global state. |
| ScriptableObject modularity | https://unity.com/resources/create-modular-game-architecture-scriptableobjects-unity-6 | Keep data modular and reusable. Separate game data from scene instances. |
| Unity design patterns | https://learn.unity.com/course/design-patterns | Use patterns as tools only where they reduce complexity. Do not copy patterns for decoration. |
| State pattern | https://learn.unity.com/tutorial/develop-a-modular-flexible-codebase-with-the-state-programming-pattern | Use a small state machine for app flow and round flow. Avoid one massive switch that owns UI, AR, and gameplay. |
| Observer/events | https://learn.unity.com/tutorial/create-modular-and-maintainable-code-with-the-observer-pattern | Use events to decouple systems. Keep event payloads typed and feature-scoped. |
| MVP UI pattern | https://learn.unity.com/tutorial/build-a-modular-codebase-with-mvc-and-mvp-programming-patterns | Keep UI views dumb. Presenters translate game state into UI. |
| Object pooling | https://docs.unity3d.com/ScriptReference/Pool.ObjectPool_1.html | Pool motes and short-lived VFX if they are spawned repeatedly. |
| `GameObject.Find` | https://docs.unity3d.com/ScriptReference/GameObject.Find.html | Do not use scene-wide name searches in gameplay or per-frame code. Prefer serialized references and cached dependencies. |
| Update callbacks | https://docs.unity3d.com/Manual/events-per-frame-optimization.html | Avoid hundreds of idle `Update` methods. Only update objects that need active ticking. |
| Garbage collection | https://docs.unity3d.com/Manual/performance-garbage-collector.html | Avoid frequent managed allocations during gameplay. Aim for near-zero GC allocations per frame in the active round. |
| AR Foundation managers | https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.5/manual/architecture/managers.html | Enable only the AR managers needed for the current state. Disable AR features when not used. |
| AR plane detection | https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.5/manual/features/plane-detection/arplanemanager.html | Use plane detection for placement, then disable or hide plane visualization after placement if it is no longer needed. |
| Microsoft DI guidelines | https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines | Use DI as an alternative to static/global access. Avoid service locator behavior and incorrect lifetime capture. |
| VContainer for Unity DI | https://vcontainer.hadashikick.jp/ | If a DI container is chosen, prefer a Unity-focused container with clear scoping, constructor injection, diagnostics, and performance guidance. |
| C# conventions | https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions | Prefer clarity, consistency, specific exceptions, and simple readable code. |

---

## 3. Target Architecture Shape

Use a thin Unity layer over testable plain C# logic.

Recommended dependency direction:

```text
UI Views / AR Views / Unity MonoBehaviours
        depend on
Presenters / Controllers / Scene Adapters
        depend on
Gameplay Services / Pure C# Rules
        depend on
ScriptableObject Config / Plain Data
```

Rules:

- MonoBehaviours should adapt Unity lifecycle, scene references, input, AR managers, prefabs, and animations.
- Gameplay rules should live in plain C# classes where practical.
- ScriptableObjects should hold configuration, event channels, definitions, and stable shared data.
- UI should read state through presenters or simple view models, not by querying gameplay objects directly.
- AR placement should publish placement results to gameplay, not own scoring or round rules.

Good example:

```text
TouchInputController raycasts against mote hit areas -> MoteSelectionController receives selected MoteId
-> RoundController validates color -> ScoreService updates score
-> RoundEvents raises ScoreChanged/BloomChanged
-> HUDPresenter updates HUDView
```

Bad example:

```text
MoteView finds GameManager singleton -> modifies score, timer, UI text,
plays audio, changes seed material, and spawns the next mote directly.
```

---

## 4. Recommended Folder Layout

Use the project convention if one already exists. Otherwise use:

```text
Assets/
  _Project/
    Art/
    Audio/
    Materials/
    Prefabs/
      AR/
      Gameplay/
      UI/
      VFX/
    Scenes/
    ScriptableObjects/
      Config/
      Events/
      Definitions/
    Scripts/
      App/
      AR/
      Gameplay/
      UI/
      Audio/
      Persistence/
      Infrastructure/
      Composition/
      Debugging/
    Tests/
      EditMode/
      PlayMode/
```

Optional assembly definitions:

- `PocketGrove.Runtime`
- `PocketGrove.Tests.EditMode`
- `PocketGrove.Tests.PlayMode`

Do not spend MVP time fighting assembly definitions if the project is not ready for them, but keep the folder boundaries either way.

---

## 5. Namespaces and Naming

Use one root namespace:

```csharp
namespace PocketGrove.Gameplay
namespace PocketGrove.AR
namespace PocketGrove.UI
namespace PocketGrove.Audio
namespace PocketGrove.Persistence
namespace PocketGrove.Infrastructure
namespace PocketGrove.Composition
```

Naming rules:

- `*Controller` coordinates user/game actions in a specific feature.
- `*Service` performs reusable non-visual work with no scene ownership.
- `*Presenter` maps state/events to UI views.
- `*View` owns scene objects, visuals, animations, and Unity references.
- `*Config` is a ScriptableObject or immutable data object.
- `*Definition` describes a game item such as mote color/type.
- `*State` is runtime state or a state-machine state.
- `*EventChannel` is a typed ScriptableObject event channel.
- `*CompositionRoot`, `*LifetimeScope`, or `*Installer` wires dependencies and should contain no gameplay rules.

Avoid vague names:

- `GameManager`
- `MainManager`
- `ARManagerCustom`
- `SystemController`
- `Utils`
- `Helper`
- `Data`

If a class name needs "Manager", it must be clear what it manages and why no more specific name works.

---

## 6. Core Systems for Pocket Grove

The implementation should have these conceptual systems. Exact file names may vary, but the responsibilities must remain separated.

### Composition

Owns dependency wiring only.

Recommended classes:

- `PocketGroveCompositionRoot` for manual DI, or
- `GameLifetimeScope` / `PocketGroveInstaller` if using an approved DI container.

Responsibilities:

- Create plain C# services.
- Connect ScriptableObject configs to services.
- Connect scene views/adapters to presenters/controllers.
- Create factories and pools.
- Expose a single obvious place to inspect dependency wiring.

Must not:

- Contain scoring formulas.
- Contain AR raycast logic.
- Contain UI formatting.
- Become a gameplay manager.

### App Flow

Owns high-level state:

- Boot
- MainMenu
- PermissionCheck
- PlacementSearching
- GrovePreview
- Playing
- Paused
- TrackingLost
- RoundComplete
- Results

Recommended classes:

- `AppStateMachine`
- `IAppState`
- `BootState`
- `MainMenuState`
- `PlacementState`
- `PlayingState`
- `TrackingLostState`
- `ResultsState`

Keep each state small. A state should enter, exit, and coordinate active systems. It should not contain scoring formulas, UI formatting, AR raycast details, and audio lookup logic all together.

### AR Placement

Owns AR-specific placement work only.

Recommended classes:

- `ARPlacementController`
- `ARSurfaceReticleView`
- `ARTrackingMonitor`
- `GroveAnchorView`

Responsibilities:

- Check AR availability and permission results.
- Read AR session/tracking state.
- Raycast against detected planes.
- Show/hide reticle.
- Place or replace the grove root.
- Disable or hide plane visualization after placement if no longer needed.
- Notify app flow when placement succeeds.

Must not:

- Calculate score.
- Own round timer.
- Decide target color.
- Update HUD directly.
- Spawn gameplay motes except through a gameplay-facing command such as `StartRound(groveRoot)`.

### Round Gameplay

Owns the game loop inside Quick Bloom.

Recommended classes:

- `RoundController`
- `RoundState`
- `RoundTimer`
- `TargetColorSelector`
- `ScoreService`
- `BloomProgress`
- `MoteSpawnPlanner`
- `MoteSpawner`
- `MotePool`
- `MoteSelectionController`

Responsibilities:

- Start/restart/end a round.
- Tick the round timer.
- Track current target color.
- Validate selected motes.
- Apply correct/wrong tap rules.
- Maintain score, combo, and bloom progress.
- Request mote spawns from the pool.
- Raise typed events for UI/audio/VFX.

Must not:

- Call AR raycast APIs.
- Read UI button components.
- Use `GameObject.Find`.
- Depend on scene object names.

### Mote Views

Own visual/tappable objects only.

Recommended classes:

- `MoteView`
- `MoteVisual`
- `MoteHitArea`
- `MoteMotion`
- `TouchInputController`

Responsibilities:

- Display color/shape.
- Own collider/hit area.
- Play local idle/bob motion.
- Expose selection identity to the central input controller.
- Play local collect/flicker animation if requested.
- Return to pool when finished.

Input rules:

- Use one central touch/raycast controller for gameplay taps.
- Ignore touches that begin over UI.
- Raycast from the AR camera into a dedicated mote hit layer.
- Prefer non-allocating raycast APIs if the chosen implementation makes that straightforward.
- If multiple mote hit areas are hit, choose the closest hit or the hit closest to the tap center.

Must not:

- Modify score directly.
- Modify timer directly.
- Choose target color.
- Find UI or seed objects.

### UI

Use a simple MVP-style split.

Recommended classes:

- `MainMenuView`
- `PlacementUIView`
- `HUDView`
- `PauseMenuView`
- `ResultsView`
- `SettingsView`
- `HUDPresenter`
- `ResultsPresenter`
- `SettingsPresenter`

Rules:

- Views expose serialized references and UI events.
- Presenters subscribe to gameplay events and call view methods.
- Views should not contain gameplay decisions.
- Presenters should not know AR Foundation implementation details.

### Audio and VFX

Recommended classes:

- `AudioService`
- `AudioCueConfig`
- `VfxPool`
- `MoteCollectVfx`
- `BloomVfx`

Rules:

- Gameplay emits semantic events such as `MoteCollected`, `WrongMoteTapped`, `RoundWon`.
- Audio/VFX systems react to events.
- Gameplay does not directly know which AudioClip or ParticleSystem is used.

### Persistence

Recommended classes:

- `SettingsStore`
- `BestScoreStore`

Rules:

- Wrap `PlayerPrefs` or the chosen persistence mechanism behind a small interface.
- Only persist MVP data: sound setting, optional haptics setting, optional best score.
- Do not let arbitrary gameplay classes read/write `PlayerPrefs`.

---

## 7. Data Model and ScriptableObjects

Use ScriptableObjects for configuration that designers or the implementation chat may tune in the Inspector.

Recommended assets:

- `RoundConfig`
  - `roundDurationSeconds`
  - `bloomRequirement`
  - `activeMoteMin`
  - `activeMoteMax`
  - `targetMotesDefault`
  - `minimumTargetMotes`
  - `wrongTapTimePenalty`
  - `comboBonusInterval`
  - `comboBonusScore`
- `MoteDefinition`
  - `id`
  - `displayName`
  - `color`
  - optional `shapeMarker`
  - `visualPrefab`
- `GroveConfig`
  - `defaultDiameter`
  - `smallDiameter`
  - `largeDiameter`
  - `spawnHeightMin`
  - `spawnHeightMax`
  - ring slot positions if using predefined slots
- `AudioCueConfig`
  - semantic cue names mapped to clips and volume.
- `EventChannel` assets if using ScriptableObject event channels.

ScriptableObject rules:

- Treat config assets as read-only during gameplay unless they are intentionally runtime variables.
- Runtime mutable values such as score, combo, timer, and current target color belong in runtime state classes, not permanent config assets.
- If using ScriptableObject runtime variables, keep `InitialValue` and `RuntimeValue` separate so Play Mode changes do not accidentally persist to disk.

---

## 8. Dependency Rules

Dependency injection is required as a design practice. A DI container is optional.

Preferred dependency methods:

1. Serialized field references assigned in prefab/scene.
2. Constructor injection for plain C# classes.
3. A scene-level composition root that creates plain C# services and wires MonoBehaviour adapters.
4. Explicit `Initialize(...)` methods for MonoBehaviours created at runtime.
5. ScriptableObject event channels for feature-scoped notifications.

Avoid:

- Scene-wide searches.
- Hidden static dependencies.
- Global service locators.
- Static mutable state.
- Public fields used as uncontrolled global variables.

Allowed limited exceptions:

- Unity-required static APIs such as `Time`, `Application`, or `PlayerPrefs` may be wrapped at boundaries.
- A single scene-level composition root may wire dependencies.
- A custom update manager may exist only if profiling shows many idle `Update` callbacks. This project should not need one for MVP.

Composition root policy:

- All cross-system wiring should happen in one obvious place, such as `PocketGroveCompositionRoot`, `GameLifetimeScope`, or the chosen DI container scope.
- Feature classes should declare dependencies in constructors or explicit initialization methods.
- Classes should not reach into a global container to resolve dependencies during normal gameplay.
- MonoBehaviours may receive scene/prefab references through serialized fields, then pass the required dependencies to plain C# services.
- Runtime-created objects such as motes should receive their dependencies from a factory/pool when spawned, not by searching the scene.

Singleton policy:

- Do not create `GameManager.Instance`, `AudioManager.Instance`, `UIManager.Instance`, or `ServiceLocator.Instance`.
- If a singleton is proposed, the implementation must include a written justification in code review notes:
  - Why a serialized reference is insufficient.
  - Why scene composition is insufficient.
  - How lifecycle/reset is handled.
  - How tests can replace it.
- For this MVP, the expected number of custom singletons is **zero**.

---

## 9. DI Container Policy

The preferred default for this MVP is **manual DI**:

- Use a scene-level composition root.
- Use constructor injection for plain C# services.
- Use serialized fields for Unity scene/prefab references.
- Use explicit factories for pooled/runtime-created objects.
- Keep dependencies visible at the class boundary.

This gives most DI benefits without adding a package, reflection cost, unfamiliar lifecycle rules, or container-specific debugging during the first AR prototype.

### When a DI Container Is Allowed

A DI container is allowed only if the implementation chat explicitly chooses it and explains the choice before coding.

Use a container when at least two of these are true:

- The project already has the package installed and working.
- The implementation will create many plain C# services with non-trivial dependency graphs.
- The team/developer is already comfortable with that container.
- Automated tests need easy replacement of services and mocks.
- The app is expected to grow beyond this one-day prototype immediately.
- The container setup can be completed without threatening the vertical slice.

Do not add a container just to make the code look modern.

### Recommended Container If One Is Chosen

If adding a container from scratch, **VContainer is the preferred candidate** for this project because its documentation emphasizes Unity-specific DI, constructor injection, LifetimeScope scoping, diagnostics, and performance options such as source generation.

Acceptable alternatives:

- A small custom composition root if no container is needed.
- **Extenject/Zenject are not recommended for new work** — both are effectively unmaintained (Extenject was removed from the Asset Store and its maintainer stepped back). Keep one only if the project already depends on it; do not add it fresh. Prefer VContainer if a container is genuinely needed.

Do not install more than one DI container.

### Container Rules

If using VContainer, Extenject, Zenject, or another container:

- There must be one obvious project/scene scope for Pocket Grove.
- Register interfaces to implementations in the scope/installer only.
- Prefer constructor injection for plain C# classes.
- Use method/property/field injection for MonoBehaviours only when Unity construction makes constructor injection impractical.
- Do not inject the container itself into gameplay classes.
- Do not call `Resolve`, `GetService`, or equivalent from arbitrary gameplay code.
- Do not create service locator wrappers around the container.
- Keep lifetimes explicit and documented:
  - App-wide services: container singleton/scoped to app lifetime is acceptable only for stateless or reset-safe services. This is not permission to add static `Instance` access.
  - Round services: scoped to a round or reset explicitly on round restart.
  - Mote instances: pooled/factory-created, not singleton.
  - UI views and AR views: scene/prefab instances.
- Avoid disposable transient services captured by a long-lived/root container.
- Dispose round/scene scopes when leaving the round or replacing the grove if the chosen container supports scopes.
- Add a validation step or startup smoke test that proves all bindings resolve.

### Container Red Flags

Refactor if:

- Classes hide dependencies by resolving from the container internally.
- The container is used as a global registry.
- Most MonoBehaviours contain `[Inject]` fields but no clear lifetime ownership.
- A singleton service holds references to round-specific objects, motes, UI panels, or scene objects that should reset.
- Binding configuration is larger and harder to understand than the systems it wires.
- Tests still cannot replace dependencies despite using a container.

The rule is simple: DI should make dependencies more visible, not more magical.

---

## 10. Forbidden and Restricted APIs

### Forbidden in gameplay/runtime hot paths

- `GameObject.Find`
- `GameObject.FindWithTag`
- `Object.FindObjectOfType`
- `Object.FindObjectsOfType`
- `Object.FindAnyObjectByType`
- `GetComponent` inside `Update` or frequent loops
- LINQ inside per-frame gameplay loops
- String-based dispatch such as `SendMessage`
- Reflection-based resolving or wiring during gameplay hot paths. Container build-time reflection at startup/composition is allowed if a DI container is explicitly chosen and not used as a service locator.
- `Resources.Load` for normal project assets
- Repeated `Instantiate`/`Destroy` for motes during active rounds
- `new WaitForSeconds(...)` every time inside repeated loops if it creates avoidable allocations

### Allowed only at setup time with justification

- `GetComponent` in `Awake`, `Start`, or one-time initialization.
- `FindAnyObjectByType` in editor-only tooling or temporary migration code.
- Tags/layers for collision filtering, as long as code does not repeatedly search by tag.
- `Instantiate` during initial pool warmup or one-time scene setup.

### Preferred alternatives

- Serialized fields.
- Prefab references.
- Cached component references.
- Explicit dependency injection.
- Object pools.
- Typed events.
- Layer masks for raycasts.

Before final handoff, run a search for restricted APIs.

Example:

```powershell
rg -n "GameObject\.Find|FindWithTag|FindObjectOfType|FindObjectsOfType|FindAnyObjectByType|SendMessage|Resources\.Load" Assets
```

Any match in runtime code must be removed or justified.

---

## 11. Update, Tick, and Coroutine Rules

The active round is small, but AR on mobile is sensitive to frame time. Keep ticking explicit.

Rules:

- `RoundController` or app state owns the main gameplay tick.
- Motes may have lightweight motion scripts only if necessary.
- Do not create dozens of MonoBehaviours with empty or rarely-used `Update`.
- Do not poll state in UI every frame. Update UI when state changes.
- Do not check "are we paused/tracking lost" inside every object. Pause/resume systems centrally.
- Use coroutines for short sequences, but do not hide main game state transitions inside unrelated coroutines.
- If a coroutine can outlive its state, cancel it on `Exit`, `OnDisable`, or round reset.

Acceptable:

- `MoteMotion.Update` for active motes if the count stays around 8 to 12.
- `RoundController.Tick(deltaTime)` from the active Playing state.
- Short animation coroutines owned by views.

Not acceptable:

- Every UI panel running `Update` to query score.
- Every mote searching for the seed each frame.
- Timers scattered across multiple unrelated MonoBehaviours.

---

## 12. Object Pooling Rules

Pool repeated short-lived gameplay objects:

- Light motes.
- Pickup sparkle VFX.
- Optional bloom pulse VFX.

Do not over-pool:

- Main menu screens.
- One central seed.
- One grove root.
- Static rocks/grass/trees.

Pool requirements:

- Pool size should cover the designed max active count plus a small buffer.
- Mote pool initial capacity: 12 to 16.
- Mote pool max size: 20 to 24.
- VFX pool initial capacity: 6 to 10 per frequently used effect.
- Returned objects must reset visual state, color, collider state, lifetime, subscriptions, and parent transform.
- Double-release should be guarded in editor/dev builds.

Do not call `Destroy` on collected motes during gameplay. Return them to the pool.

---

## 13. AR Foundation Rules

If the implementation uses AR Foundation, follow these rules.

AR session:

- There should be one AR Session and one XR Origin setup.
- App flow should handle unsupported devices and denied camera permission.
- Tracking loss should transition to `TrackingLost` state and pause gameplay timers.

AR managers:

- Use only the managers needed for MVP:
  - AR Session
  - XR Origin
  - AR Camera
  - AR Raycast Manager
  - AR Plane Manager
  - optional AR Anchor Manager if the chosen placement strategy uses anchors
- Do not enable image tracking, face tracking, body tracking, environment probes, mesh reconstruction, or object tracking for MVP.
- Disable plane detection or at least hide plane visualization after the grove is placed, unless the Replace flow needs it active.
- Do not `Destroy` AR trackables managed by AR Foundation. Hide/deactivate visualization or use manager-supported APIs.

Placement:

- AR raycast and plane selection live in `ARPlacementController`.
- Grove placement produces a `Transform` or `GroveRoot` reference used by gameplay.
- If the grove is anchored, use the current `ARAnchorManager.TryAddAnchorAsync` API; `AddAnchor` is obsolete, and `AttachAnchor` requires first checking `descriptor.supportsTrackableAttachments`.
- Gameplay objects should be children of the placed grove root or anchor root.
- Replacing the grove should cleanly stop the active round, return pooled motes, clear VFX, and create a new placement preview.

---

## 14. UI Architecture Rules

UI must be simple but not tangled.

Rules:

- One view class per screen/panel.
- Views expose events such as `PlayClicked`, `PauseClicked`, `SettingsChanged`.
- Presenters subscribe to view events and call app/gameplay services.
- Presenters update text, sliders, icons, and buttons through view methods.
- UI text values should be formatted in presenters, not in gameplay services.
- UI should never call `GameObject.Find` to locate gameplay.
- UI should never own score, timer, or current target as source of truth.

Recommended flow:

```text
RoundController raises RoundStateChanged
HUDPresenter receives event
HUDPresenter formats timer/score/bloom
HUDView displays values
```

---

## 15. Event Policy

Use events to reduce coupling, but keep them disciplined.

Good event examples:

- `RoundStarted`
- `TargetColorChanged`
- `MoteCollected`
- `WrongMoteTapped`
- `ScoreChanged`
- `BloomProgressChanged`
- `RoundWon`
- `RoundTimedOut`
- `TrackingLost`
- `TrackingRecovered`

Rules:

- Events should be typed; avoid string event names.
- Event payloads should be small immutable structs or simple values.
- Subscribe on enable/start and unsubscribe on disable/dispose.
- Do not use one giant global `EventBus` for every message.
- Feature-scoped event channels are acceptable.
- For one-to-one direct relationships, prefer explicit method calls over events.

Bad:

```text
GlobalEventBus.Publish("thing_happened", object[] data)
```

Good:

```text
public readonly struct MoteCollectedEvent
{
    public MoteColor Color { get; }
    public int Combo { get; }
}
```

---

## 16. Gameplay Rule Ownership

Each rule has one owner.

| Rule | Owner |
|---|---|
| Round duration | `RoundConfig` read by `RoundTimer` |
| Timer ticking and pause | `RoundTimer` / `RoundController` |
| Current target color | `TargetColorSelector` / `RoundState` |
| Correct vs wrong mote | `RoundController` or `MoteSelectionController` |
| Score formula | `ScoreService` |
| Combo formula | `ScoreService` or `ComboTracker` |
| Bloom progress | `BloomProgress` |
| Mote count and target distribution | `MoteSpawnPlanner` |
| Mote visuals | `MoteView` / `MoteVisual` |
| Seed visuals | `SeedView` / `BloomPresenter` |
| HUD display | `HUDPresenter` / `HUDView` |
| Audio clip choice | `AudioService` / `AudioCueConfig` |

Do not duplicate formulas across UI, gameplay, and result screens. Results should read final round state, not recompute score independently.

---

## 17. Testing Requirements

At minimum, write Edit Mode tests for pure gameplay logic.

### Per-mechanic test gate (Definition of Done)

Every gameplay mechanic is built test-alongside and is **not "done" until all three hold**:

1. **Unit tests in the same change.** New or changed **pure rules** (score, timer, target selection,
   bloom, spawn planning) ship with Edit Mode tests in the same commit. If a mechanic is *only* a thin
   AR/UI adapter with no pure-rule logic, it has no unit tests by design (see Testability rules) and is
   covered by the smoke/device pass below instead — but any testable rule it introduces must be tested.
2. **Smoke test before completion.** Before calling a mechanic done, run a smoke pass in conditions as
   close to real human use as practical: a Play Mode smoke test and/or an **MCP-driven** editor check
   (enter Play, drive the actual flow, read the Console), under **XR Simulation** for AR flows, and on a
   real device when the mechanic depends on real tracking, camera, or touch input. Use MCP where it can
   realistically drive the flow; otherwise do a manual device pass. (MCP only where it fits the case.)
3. **No regression.** Re-run the **full existing** Edit Mode suite plus the relevant Play Mode / smoke
   checks and confirm earlier mechanics still pass and still behave as intended. A change that breaks an
   earlier test is not done — fix it or revert before moving on.

Required tests:

- `TargetColorSelector` always changes to a color different from the current target (with 3 colors it never repeats back to back).
- `TargetColorSelector` changes target after every 3 correct motes.
- `ScoreService` awards correct score, combo bonuses, and time bonus.
- `RoundTimer` pauses and resumes without losing time incorrectly.
- `BloomProgress` reaches victory exactly at requirement.
- `MoteSpawnPlanner` maintains minimum target-color motes.
- `MoteSpawnPlanner` never returns spawn positions outside grove bounds.
- Wrong tap subtracts time and resets combo.
- Empty tap has no penalty.

Play Mode / smoke tests (the smoke pass in the gate above is required; author these as the slice grows):

- App can transition MainMenu -> PlacementSearching -> GrovePreview -> Playing with mocked placement.
- TrackingLost pauses timer.
- Restart clears motes and resets round state.
- Results screen receives victory/time-out data.

Testability rules:

- Pure rule classes should not inherit `MonoBehaviour`.
- Pure rule classes should not depend on `Time.time`, `Random.Range`, or Unity scene objects directly.
- Wrap randomness behind an interface or pass a seeded random provider when testing.
- If a class cannot be tested without an AR device, it should be a thin adapter, not a rule owner.

---

## 18. Performance Requirements

Target: stable mobile AR performance over visual extravagance.

Runtime rules:

- No per-frame scene-wide searches.
- No avoidable GC allocations during active gameplay.
- No repeated instantiate/destroy cycle for motes.
- No expensive physics simulation for collectibles.
- Use simple colliders or raycast targets for motes.
- Keep active motes around 8 to 12.
- Keep particle bursts short and limited.
- Keep materials and texture sizes modest.
- Disable unused AR managers/features.

Profiling expectations:

- Use Unity Profiler or device profiling before declaring performance "done".
- Check for GC allocations while a round is running.
- Check frame time during placement and during active mote collection.
- If performance is poor, reduce particles, shadows, transparent overdraw, and active motes before changing game rules.

---

## 19. Error Handling and Edge Cases

The app must not get stuck in a broken state.

Handle:

- Camera/AR permission denied.
- AR unsupported device.
- No plane detected for a while.
- Tracking lost during placement.
- Tracking lost during round.
- Player presses Back/Main Menu during placement or round.
- Player replaces grove before starting.
- Player restarts after victory.
- Mote selected while round is paused.
- Mote selected after it is already collected.
- Pool object returned twice.

Rules:

- Invalid player actions should be ignored safely or produce soft UI feedback.
- Do not throw exceptions for normal user behavior.
- Do not swallow exceptions silently during development.
- Use clear debug logs for unexpected states, but remove noisy logs from demo builds.

---

## 20. Architecture Decision Gate

Before writing production code, the implementation chat must make a short architecture decision note. It can be a section in its first response, a small `ARCHITECTURE.md`, or a tracked task note.

The decision note must answer:

- Which AR stack is being used, and why.
- Which scene structure is being used: one scene, additive scenes, or another approach.
- Which app/round state machine approach is being used.
- Which dependency approach is being used: manual DI/composition root, VContainer, Extenject/Zenject, or another option.
- Why a DI container is or is not being used.
- How runtime-created objects such as motes and VFX are created, initialized, pooled, and reset.
- Which gameplay rules will be pure C# and covered by Edit Mode tests.
- Which parts are allowed to remain Unity/AR adapters with little or no test coverage.
- Which guardrail deviations, if any, are intentional.

The decision note must reject at least these bad options explicitly:

- One `GameManager` owning AR, UI, score, audio, spawning, and persistence.
- Global singletons for normal gameplay services.
- Service locator access from gameplay code.
- Per-frame `GameObject.Find` / tag searches / scene-wide object searches.
- UI scripts as the source of truth for score, timer, target color, or round state.
- AR placement code owning gameplay rules.

This gate should not become a long architecture essay. Its job is to force the next chat to choose deliberately before generating code.

---

## 21. LLM Implementation Protocol

Any coding chat or agent implementing this project should follow this workflow.

Before coding:

- Read `game-design.md`.
- Read this guardrails document.
- Identify the chosen AR stack and Unity version.
- Choose manual DI or a specific DI container, and explain the reason.
- List the planned systems and files.
- State any intentional deviations from this document.

During coding:

- Build one vertical slice first:
  - menu -> placement placeholder -> start round -> tap primitive motes -> results.
- Keep classes small.
- Move formulas and state transitions out of MonoBehaviours when practical.
- Write unit tests **alongside each mechanic** (same change), not afterward.
- Do not postpone all architecture until after the prototype works.

Before handoff:

- Run the **full** Edit Mode suite — confirm **no regressions** (earlier mechanics still pass).
- Do a **smoke pass in human-realistic conditions**: MCP-driven Play Mode and/or XR Simulation, and a
  device pass where tracking/camera/touch matter.
- Run restricted API search.
- Review every singleton/static mutable field.
- Verify there is no one-file GodObject.
- Verify the game can complete one victory and one time-out path.
- Verify AR tracking loss pauses gameplay if implemented.

---

## 22. Code Review Checklist

A change is not acceptable if any answer below is "no" without a written justification.

Architecture:

- Does each class have a clear single responsibility?
- Can gameplay rules be understood without reading UI and AR code?
- Can UI be changed without rewriting scoring?
- Can AR placement be changed without rewriting round rules?
- Is runtime state reset cleanly on replay/replacement?

Dependencies:

- Are dependencies assigned explicitly?
- Is there a clear composition root or DI scope?
- Are gameplay dependencies visible through constructors or explicit initialization?
- If a DI container is used, is it absent from gameplay classes except the composition layer?
- Are there no new global singletons?
- Are events typed and unsubscribed?
- Are ScriptableObjects used as config/event channels rather than hidden mutable globals?

Performance:

- Are repeated objects pooled?
- Are scene searches absent from gameplay hot paths?
- Are per-frame allocations avoided in the active round?
- Are unused AR features disabled?

Testing:

- Are score/timer/target/spawn rules covered by Edit Mode tests?
- Does each new or changed mechanic ship with unit tests in the **same change**?
- Was the **full existing suite re-run with no regressions**?
- Was the mechanic **smoke-tested in human-realistic conditions** (MCP / XR Simulation / device)?
- Can a non-AR mock path drive the round logic?
- Did the implementer test at least one win and one timeout path?

Readability:

- Are names specific?
- Are methods short enough to understand?
- Are comments used only where they clarify non-obvious intent?
- Is there no script that mixes UI, AR, score, spawn, audio, and persistence?

---

## 23. Red Flags That Require Refactoring

Refactor immediately if you see:

- A class over roughly 300 lines in MVP code, unless it is generated or data-only.
- A method over roughly 50 lines with mixed responsibilities.
- `GameManager.Instance` used from many scripts.
- UI text updated from gameplay objects directly.
- Mote scripts changing score by themselves.
- AR placement code spawning gameplay motes directly.
- AudioClip references scattered across unrelated scripts.
- Copy-pasted scoring formulas.
- `Update` methods that only poll rare conditions.
- Scene object names used as identifiers.
- String event names.
- Persistent state hidden in static fields.
- Testable logic trapped inside MonoBehaviours.
- Coroutines that continue after the state that started them has ended.

---

## 24. Acceptable Simplicity

Good architecture for this project is not maximal architecture.

Acceptable:

- A simple `AppStateMachine` with small state classes.
- A few ScriptableObject configs.
- Plain C# services for score, timer, target selection, and spawn planning.
- Manual DI through a composition root plus serialized Unity references.
- A lightweight Unity DI container only if it meets the DI Container Policy above.
- One scene if the MVP is easier that way.
- Primitive placeholder visuals during early vertical-slice work.

Overkill for MVP:

- Full ECS/DOTS.
- Networking architecture.
- Addressables unless asset loading genuinely needs it.
- A heavy or unfamiliar dependency injection framework added only for fashion.
- A generic global event bus.
- Complex save architecture.
- A plugin system.
- A procedural level generator.

The ideal result is boring in the best way: clear, explicit, easy to debug, and hard to accidentally turn into spaghetti.

---

## 25. Final Implementation Contract

The Pocket Grove implementation should be accepted only when:

- The game design in `game-design.md` is playable.
- The architecture is split into AR, gameplay, UI, audio, persistence, and infrastructure boundaries.
- No MVP gameplay depends on global singletons.
- Dependencies are wired through a clear composition root, manual DI, or one approved DI container.
- No gameplay class resolves dependencies from a global container/service locator.
- No runtime gameplay hot path uses scene-wide object searches.
- Repeated motes/VFX are pooled or otherwise proven harmless.
- Score, timer, target selection, and spawn planning are testable outside AR.
- App and round states are explicit.
- AR-specific code is isolated from gameplay rules.
- UI is presentation code, not gameplay authority.
- The final code review checklist is satisfied.

If a future chat wants to break these rules, it must explain why the rule is wrong for this project, what risk is being accepted, and how the risk will be tested.
