# Build & Test

Last verified: 2026-06-01

The **single source of truth for commands**. Don't duplicate these elsewhere — link here. All commands
assume the repo root (this folder) as the working directory and Unity **6.3 LTS (`6000.3.16f1`)**.

## Who does what

- **Assistant (Claude Code):** edits C#/assets, drives the Editor via the **MCP for Unity** server
  (create/modify GameObjects, components, scenes, materials; run the Test Runner; read the Console),
  and edits `Packages/manifest.json` to add packages. Read-only git only.
- **Human:** platform switch, parts of Player Settings, URP renderer feature, **Graphics API
  selection**, device build & run, and all git commits. These are unreliable or impossible via MCP.

## Compile / sanity check (every change)

Via MCP: trigger a domain reload / script compile and **read the Console** — "ready for review" means
**no compile errors and no Console errors**, and **Project Validation** passes (Project Settings →
XR Plug-in Management → Project Validation; fix any ARCore/URP issues it flags).

## Run tests

Pure gameplay logic is **EditMode** (no Play mode needed). Prefer running via the **MCP Test Runner**
inside the open Editor. Headless CLI alternative (close the Editor or use a second Unity instance):

```
<UnityEditor>\Unity.exe -runTests -batchmode -projectPath . ^
  -testPlatform EditMode -testResults .\TestResults-EditMode.xml
```

- `<UnityEditor>` = the 6000.3.16f1 editor (e.g. `C:\Program Files\Unity\Hub\Editor\6000.3.16f1\Editor`).
- `-testPlatform PlayMode` for Play mode tests. **Run a fresh Unity process per PlayMode run** (a known
  CLI quirk prevents running PlayMode tests twice in one invocation).
- Results are NUnit XML at the path given. Never claim a test passed without running it this session or
  citing exact evidence.

## Test AR in the Editor (no phone) — XR Simulation

AR Foundation ships **XR Simulation**, which fakes plane detection and device movement in Play mode:

1. Project Settings → **XR Plug-in Management** → Editor/Standalone tab → enable **XR Simulation**
   (ARCore stays enabled for the Android tab only).
2. Press **Play**. Use the Simulation environment to exercise plane detection, the reticle, and
   placement. Simulated planes are detected on axis-aligned surfaces in the sample environments.

This is the **default fast loop** for placement/round logic. Device testing validates the rest.

## Build & run on device (human step)

1. Switch platform to **Android** (`File → Build Settings → Android → Switch Platform`).
2. Player Settings: **disable Auto Graphics API**, set APIs explicitly (Vulkan and/or OpenGLES3; keep
   **OpenGLES3** as fallback if ARCore Requirement = **Optional**); minimum API **24+**; ensure camera
   permission is requested at runtime.
3. `Build And Run` to an ARCore-capable device. The only build scene is
   `Assets/_Project/Scenes/Gameplay.unity`.

## Restricted-API check (before handoff)

Search first-party runtime code for forbidden APIs (see
[guardrails §7](../architecture/implementation-guardrails.md)). Use the Grep tool, or `rg`:

```
rg -n "GameObject\.Find|FindWithTag|FindObjectOfType|FindObjectsOfType|FindAnyObjectByType|SendMessage|Resources\.Load" Assets/_Project/Scripts
```

Any match in runtime code must be removed or justified in review.

## Per-mechanic gate: regression + smoke (see [guardrails §17](../architecture/implementation-guardrails.md))

Before a mechanic is "done":
- **Regression:** run the **full** EditMode suite (command above) — all green; confirm earlier mechanics
  still pass. For PlayMode, a fresh Unity process per run.
- **Smoke (human-realistic):** drive the actual flow and read the Console — via the **MCP** Test Runner /
  Play controls under **XR Simulation** for AR flows, and a **device** pass where tracking, camera, or
  touch matter. Use MCP only where it can realistically drive the case.

## What "done" means / what can't be automated yet

- **Mechanic done:** new pure rules have EditMode tests; the **full suite re-runs with no regression**;
  a **smoke pass** in human-realistic conditions passed (MCP / XR Simulation, device where needed); no
  Console errors.
- **AR slice done in Editor:** placement works under XR Simulation.
- **Demo-ready:** validated on a real ARCore phone (see the demo bar in the game-design doc) — this
  cannot be automated; it's a human device test.
