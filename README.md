# Pocket Grove

## What this is

Pocket Grove is a small **mobile-AR** game built in Unity. The player places a tiny magical grove on
a real flat surface, then taps floating light **motes** that match the seed's current color to fill a
bloom meter before a short timer ends. Cozy, readable, low-stress, one round at a time. *(Working
title — re-check name availability before any public release.)*

## Current status

**Baseline only.** Unity + URP + AR Foundation/ARCore + Input System + XR Simulation are configured,
with one `Gameplay` scene and **no gameplay code yet**. Latest detailed state:
[docs/handoff/current-status.md](docs/handoff/current-status.md).

## Requirements

- **Unity 6.3 LTS** (`6000.3.16f1`) — use this exact version.
- Unity modules: **Android Build Support** (incl. OpenJDK + Android SDK & NDK).
- Target device: Android with **ARCore** support (**Android 7.0 / API 24+**).
- Editor testing without a phone: **XR Simulation** (ships with AR Foundation, no extra install).
- Optional: **MCP for Unity** server (editor automation), **Blender** (3D content → `.fbx`/`.glb`).

## Quick start

1. Open this folder in Unity 6.3 LTS via Unity Hub; let it restore packages on first open.
2. Open `Assets/_Project/Scenes/Gameplay.unity`.
3. **Editor test:** Project Settings → XR Plug-in Management → enable **XR Simulation** for the
   Editor/Standalone target, then press **Play** to exercise plane detection and placement.
4. **Device build (human):** switch platform to Android, then Build & Run.

## Build and test

See [docs/development/build-and-test.md](docs/development/build-and-test.md) — the single source of
truth for commands.

## Documentation

Start at [docs/INDEX.md](docs/INDEX.md).

## AI-assisted development

Claude Code must read [CLAUDE.md](CLAUDE.md) and
[docs/handoff/current-status.md](docs/handoff/current-status.md) before editing. Hard rules (no git
writes, no secret reads) are enforced in `.claude/settings.json`; C# architecture rules auto-load from
`.claude/rules/`.
