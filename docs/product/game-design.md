# AR Game Concept GDD: Pocket Grove

> **Document purpose.** This is a game design brief for a small first AR game. It describes the desired player experience, game loop, menus, content, assets, and scope limits. It intentionally does not choose the final AR framework, engine package versions, or low-level implementation details. Another technical chat should use this document to pick the simplest reliable implementation path.
>
> **Implementation companion.** In Unity, also follow the implementation guardrails:
> [`../architecture/implementation-guardrails.md`](../architecture/implementation-guardrails.md). The
> architectural intent: small explicit state flow, isolated AR/platform adapters, testable gameplay
> rules, no hidden global state, and no runtime scene-wide searches.

---

## 1. High-Level Concept

**Working title:** Pocket Grove

**Naming note:** Pocket Grove is a working title, not a locked shipping name. Before public release, run a fresh app-store, web, and trademark search. Renaming should be cheap because the design does not depend on this exact title.

**One-sentence pitch:** The player places a tiny magical grove on a real table, leans or moves around it with the phone, catches floating light motes, and feeds them into a central seed before the round timer ends.

**Genre:** Cozy AR arcade collection game.

**Target experience:** A beautiful, readable, low-stress first AR app that feels like a small living object on the player's desk.

**Estimated build target:** 1 focused day for a functional playable prototype, assuming the AR base project, build pipeline, and device testing are already working. A showable demo may need one extra polish pass if AR setup, device testing, or asset import takes longer than expected.

**Primary platform assumption:** Mobile AR in portrait orientation. The design should also work in landscape if the technical implementation prefers it, but portrait is the default.

**Session length:** A round is capped at 90 seconds (the timer hard cap). The "60 to 120 seconds" feel refers to a whole session (placement, round, and results), not the round itself.

**Core fantasy:** "I grew a tiny glowing garden in my room."

---

## 2. Why This Is a Good First AR Game

Pocket Grove is designed around a single anchored tabletop scene. The player does not need large-room movement, GPS, multiplayer, world persistence, real-world object recognition, physics puzzles, networking, or complex character animation.

The AR-specific fun comes from:

- Finding a real flat surface.
- Placing a miniature world on it.
- Moving the phone around the grove to spot motes from different angles.
- Seeing particles, glow, and tiny props grounded in the real environment.

The simple technical shape is:

- One AR placement anchor.
- One compact play area.
- Tap-based interaction.
- Lightweight spawned collectibles.
- No AI navigation.
- No real-world occlusion requirement.
- No advanced physics requirement.

---

## 3. Design Pillars

1. **Tiny and tactile**
   - Everything should feel like a miniature desk toy.
   - The play area should be small enough to fit on a table, floor tile, or clear desk space.

2. **Readable in AR**
   - Important game objects use clear silhouettes, glow, and color.
   - The player should not need to interpret dense UI while moving the phone.

3. **Fast success**
   - The player should understand and make visible bloom progress within the first 20 seconds of a round.
   - A first win should be reachable within the first full round.
   - Failure is gentle: if the timer ends, the grove still partially blooms.

4. **One-day scope**
   - The MVP must be playable with primitives plus a small number of imported props.
   - Any feature requiring persistent world mapping, multiplayer, complex animation, hand tracking, or procedural level generation is out of scope.

5. **Looks better than it is complicated**
   - Low-poly nature props, emissive motes, soft particles, and good sound feedback should carry the presentation.

---

## 4. Player-Facing Summary

The player opens the app, scans a flat surface, and places a small circular grove. A central seed sits in the middle. Colored light motes float around the grove. The seed asks for one color at a time. The player taps matching motes while viewing the grove through the phone camera. Correct motes fly into the seed and fill a bloom meter. Wrong motes break the combo and cost a little time. When the bloom meter fills, the seed blooms into a glowing tree or flower, particles burst upward, and the results screen shows the score.

---

## 5. Core Game Loop

1. Start round.
2. Seed shows the current requested mote color.
3. Motes spawn around the grove.
4. Player physically moves or rotates the phone to find motes.
5. Player taps matching motes.
6. Correct mote:
   - Plays pickup sound.
   - Flies to seed.
   - Adds bloom progress.
   - Adds score and combo.
7. Wrong mote:
   - Plays soft error sound.
   - Resets combo.
   - Subtracts 2 seconds from timer.
8. When enough correct motes are collected:
   - Seed blooms.
   - Round ends with victory.
9. If timer reaches zero:
   - Round ends with partial bloom result.
10. Results screen offers replay.

---

## 6. MVP Feature Set

The MVP should include only these features:

- App launch into a simple main menu.
- AR permission flow handled gracefully.
- Plane/surface scanning state.
- Tap-to-place grove on a detected surface.
- One placed grove per round.
- One playable mode: **Quick Bloom**.
- 90-second timer.
- Three mote colors.
- One current target color shown by the seed and HUD.
- Tap interaction for motes.
- Correct/wrong feedback.
- Score, combo, and bloom meter.
- Victory and time-out result states.
- Basic settings: sound on/off.
- Haptics on/off only if the chosen engine/platform support makes it trivial.
- Credits screen with asset sources and licenses.

---

## 7. Explicit Non-Goals

Do not include these in the first version:

- Multiplayer.
- Account login.
- Cloud saves.
- GPS or location-based gameplay.
- Procedural outdoor exploration.
- Real-world semantic understanding.
- Real-world object recognition.
- Advanced occlusion as a required feature.
- Physics-heavy puzzles.
- Character pathfinding.
- Inventory systems.
- Crafting.
- Daily quests.
- Ads or monetization.
- In-app purchases.
- Large progression trees.
- A story campaign.
- More than one gameplay mode in MVP.

These can be considered after the prototype works, but they should not influence the initial architecture unless the technical chat decides they are essentially free.

---

## 8. AR Play Space

**Placement surface:** A flat horizontal surface such as a desk, table, or floor.

**Default physical size:** About 0.55 meters diameter.

**Small size option:** About 0.35 meters diameter for cramped desks.

**Large size option:** About 0.75 meters diameter only if tracking and framing feel stable.

**Placement behavior:**

- During scanning, show a simple reticle on detected surfaces.
- Player taps **Place** or taps the reticle to create the grove.
- After placement, the grove should stay fixed in world space.
- **Replace** returns to placement. From **Grove Preview** (before Start) it simply re-places the grove with no round state. From the **Pause menu** (mid-round) it abandons the current round — returning motes to the pool, clearing VFX, and resetting score, timer, bloom, and combo — then re-enters placement. From **Results** it starts a fresh placement and a new round.
- The player should not need to walk around a room; leaning, rotating, or moving around a desk should be enough.

**AR comfort constraints:**

- Keep all active gameplay objects within the chosen grove radius.
- Do not spawn required targets behind the player or far outside the placed area.
- Avoid forcing rapid phone movement.
- No flashing full-screen effects.
- If tracking is lost, pause the timer and show a calm tracking recovery overlay.

---

## 9. Main Objects

### Central Seed

The central seed is the main objective object.

**States:**

- Dormant: closed seed, faint glow.
- Hungry: seed displays requested mote color.
- Feeding: correct mote flies into seed; seed pulses.
- Almost bloomed: more petals/leaves visible, stronger glow.
- Bloomed: final flower/tree state with particles.

**MVP visual implementation:** Can be built from a low-poly plant/tree prop plus a simple emissive orb, or from primitives if no suitable model is ready.

### Light Motes

Light motes are small floating collectibles.

**Colors:**

- Sun mote: warm yellow.
- Leaf mote: fresh green.
- Moon mote: cool blue.

**Behavior:**

- Spawn around the grove at low heights.
- Drift slowly in short loops.
- Bob up and down.
- Face the camera or remain simple glowing spheres.
- Despawn after being collected or after a 12 to 20 second lifetime.
- Never let natural despawn remove the last visible target-color mote.

**Visibility rules:**

- Each mote should have an emissive center.
- Each mote should have a small particle trail or sparkle.
- Active target-color motes should be slightly brighter than distractors.
- The tap collider or hit area should be larger than the visible mote, roughly 1.5x to 2x visual size.
- Motes should be at least 4 to 6 cm wide in world scale, then tuned on device.

### Grove Base

The grove base grounds the scene.

**Shape:** Circular or rounded low-poly island, not a floating rectangular board.

**Props:**

- 1 central seed/plant.
- 3 to 6 small rocks.
- 3 to 8 grass/foliage clumps.
- 1 to 3 small trees or bushes.
- Optional small path stones.

**Performance rule:** The base should look rich from asset placement, not from high geometry density.

---

## 10. Controls

### Required Controls

- **Tap detected surface:** Place grove.
- **Tap mote:** Attempt to collect mote.
- **Tap Play Again:** Restart from results.
- **Tap Replace:** Return to placement (behavior differs by screen — see AR Play Space, Placement behavior).
- **Tap Pause:** Pause round.

### Optional Controls

- **Drag before round:** Rotate the grove.
- *(Pinch-to-scale is deferred to a stretch goal. In MVP the grove size is set only by the Small/Medium/Large toggle on the Grove Preview screen.)*
- **Tap seed after victory:** Trigger one extra bloom sparkle.

Optional controls should only be added if they do not delay the MVP.

---

## 11. Round Rules

**Mode name:** Quick Bloom

**Round duration:** 90 seconds.

**Bloom requirement:** 15 correct motes. Bloom progress is the same counter as "correct motes" — a single value that reaches 15 on victory. (It only diverges from the correct-mote count if the Shade Puffs stretch goal is added.)

**Target colors:** Sun, Leaf, Moon.

**Target color sequence:**

- Start with a random color.
- After every 3 correct motes, choose the next target color (by global correct count: at 3, 6, 9, and 12; the round ends at 15).
- On each change, choose a color different from the current target. With only three colors this guarantees a color never repeats back to back, so no extra anti-repeat rule is needed.

**Spawn rules:**

- Keep 8 to 12 motes active at once. Fill to about 10 at round start; when the active count drops below 8, refill toward 10 to 12 with a small stagger (about 0.2 to 0.4 seconds apart) so motes do not all pop in at once. Never exceed 12.
- Keep about 4 active motes matching the current target color, and never fewer than 3.
- After a target color change, at least one matching mote should be easy to notice from the player's current viewing side.
- When the target color changes, immediately spawn or recolor motes so the target-motes rule (about 4, never fewer than 3) remains true.
- Distractor motes use the other two colors.
- Spawn motes at varied angles around the grove, preferably using simple predefined ring slots rather than pure random positions.
- Spawn height should stay between 0.08 and 0.35 meters above the grove base.
- Avoid spawning motes inside dense props where they cannot be tapped.

**Correct tap:**

- +10 score.
- +1 bloom progress.
- +1 combo.
- Correct mote animates into the seed.
- At every 5-combo milestone, award +50 bonus points and play a slightly brighter pulse.
- Combo persists across target color changes; it resets only on a wrong tap.

**Wrong tap:**

- -2 seconds. The timer never drops below 0; if this penalty would pass 0, the round ends in time-out.
- Combo resets to 0.
- Wrong mote briefly flickers and remains in play.
- No harsh sound.

**Missed tap on empty space:**

- No penalty in MVP.

**Tap reliability rule:**

- Mote taps are evaluated only during the Playing state. During the victory and time-out transitions, and while paused or tracking is lost, taps on motes are ignored.
- During gameplay, mote taps should take priority over grove props and decorative objects.
- If several mote hit areas overlap, collect the mote closest to the tap center.
- Decorative props should not block tapping a visible mote unless the mote is intentionally hidden, which the MVP should avoid.

**Victory:**

- Triggered when bloom progress reaches 15.
- If the 15th correct mote and the timer reaching 0 land on the same frame, victory takes priority over time-out.
- Stop timer.
- Play bloom animation.
- Show results after 2 seconds.

**Time-out:**

- Triggered when timer reaches 0.
- Stop spawning.
- Seed shows partial bloom based on progress.
- Show results after 1 second.

---

## 12. Scoring

**Base score:**

- Correct mote: 10 points.
- Combo bonus at every 5-combo milestone: 50 points.
- Victory time bonus: remaining seconds x 2.

**Star rating:**

- 0 stars: 0 to 4 correct motes.
- 1 star: 5 to 9 correct motes.
- 2 stars: 10 to 14 correct motes.
- 3 stars: Bloom complete.

**Best score:**

- Best score is the highest final score (a number). Store it locally on device if simple.
- If local storage is not ready on day one, omit best score rather than adding complexity.

---

## 13. Difficulty and Balance

The first version should have one difficulty only.

Initial tuning targets:

- Average first-time player should complete the bloom in 60 to 85 seconds.
- Wrong taps should be noticeable but not punishing.
- Mote movement should be slow enough for casual tapping.
- Spawn rate should prioritize readability over challenge.

If the game feels too easy:

- Reduce active target-color motes from 4 to 3.
- Increase distractor count slightly.
- Shorten round duration to 75 seconds.

If the game feels too hard:

- Increase active target-color motes to 5.
- Remove the wrong-tap time penalty.
- Make target motes brighter.

---

## 14. User Flow

### First Launch

1. Splash/title appears briefly.
2. Main menu appears.
3. Player taps **Play**.
4. AR permission prompt appears if needed.
5. Safety note appears once:
   - Keep the play area clear.
   - Stay aware of surroundings.
6. Placement screen begins.

### Normal Launch

1. Main menu.
2. Play.
3. Placement.
4. Round.
5. Results.
6. Play again or return to menu.

---

## 15. Screens and Menus

### Splash Screen

**Purpose:** Fast brand moment, not a loading-heavy intro.

**Content:**

- Title: **Pocket Grove**
- Small subtitle: **An AR bloom game**
- Optional simple animated mote.

**Duration:** 1 to 2 seconds or skip instantly when loading is complete.

### Main Menu

**Layout:**

- Live camera background can be shown only if AR session is already ready; otherwise use a simple calm background.
- Title at top.
- Primary button: **Play**
- Secondary buttons: **How To**, **Settings**, **Credits**

**No marketing copy needed.** The menu should feel like a game menu, not a landing page.

### How To

Three compact panels or steps:

1. Place the grove on a flat surface.
2. Tap motes that match the seed color.
3. Fill the bloom before time runs out.

Keep this screen visual and short.

### Settings

Required:

- Sound: On/Off.

Optional:

- Haptics: On/Off, only if supported and trivial to implement.
- Color assist: On/Off. When enabled, target motes also use shape markers. If Color Assist is not implemented, omit this setting entirely.
- (Grove size is chosen on the Grove Preview screen, not in Settings.)

### Credits

Show:

- Asset name.
- Creator/source.
- License.
- Link.

Keep the credits accessible from the main menu.

### AR Placement Screen

**State 1: Searching**

- Show camera view.
- Show subtle scanning guide.
- UI text: **Find a flat surface**
- Disable Place until a surface is detected.
- If no surface is detected after about 20 seconds, show a short hint: **Move slowly and point at a textured flat surface**
- Keep a **Back** button available so the player is never trapped in scanning.

**State 2: Surface Found**

- Show reticle on surface.
- UI text: **Tap to place the grove**
- Buttons:
  - **Place**
  - **Back**

**State 3: Grove Preview**

- Grove appears on selected surface.
- Buttons:
  - **Start**
  - **Replace**
  - Size toggle: **Small / Medium / Large** (the only way to set grove size in MVP)

### Gameplay HUD

HUD should be minimal and readable:

- Top left: timer.
- Top center: current target color icon.
- Top right: score.
- Bottom center: bloom meter.
- Bottom right: pause button.

Avoid covering the central grove. HUD must remain usable over bright or dark camera backgrounds, so use contrast panels, outlines, or shadowed text.

### Pause Menu

Buttons:

- Resume.
- Restart.
- Replace Grove.
- Settings.
- Main Menu.

Timer and spawning must stop while paused. **Restart** (here) and **Play Again** (on Results) both reset the round — score, timer, bloom, and combo — on the same grove; only **Replace Grove** returns to placement.

### Tracking Lost Overlay

If tracking becomes unstable:

- Pause gameplay timer.
- Dim game UI.
- Show: **Hold still and point back at the grove**
- Resume automatically when tracking returns.

Do not count tracking loss as failure.

### Results Screen

Show:

- Result title:
  - **Full Bloom!** for victory.
  - **Still Growing** for time-out.
- Star rating.
- Correct motes collected.
- Score.
- Best score if available.
- Buttons:
  - **Play Again**
  - **Replace Grove**
  - **Main Menu**

---

## 16. Visual Direction

**Style:** Low-poly cozy miniature garden with clean silhouettes, soft emissive magic, and readable colors.

**Color palette:**

- Foliage greens.
- Warm yellow motes.
- Cool blue motes.
- Small coral/pink bloom accents.
- Neutral stone/earth base.

Avoid making the entire game one hue. The scene should read as a green garden with warm and cool magical accents.

**Lighting:**

- Use simple, mobile-friendly lighting.
- Prefer baked/static-looking materials and emissive motes.
- Dynamic shadows are optional and should be disabled if performance suffers.
- Add a soft circular shadow or contact decal under the grove if easy.

**Scale feel:**

- The grove should look like a tiny object placed into the real world.
- Props should be slightly exaggerated and toy-like.
- Motes should be large enough to tap comfortably on phone screens.

**VFX:**

- Correct mote pickup: short sparkle and trail into seed.
- Wrong tap: small color flicker, no explosion.
- Combo bonus: seed pulse ring.
- Victory: upward particle burst and gentle bloom glow.

Keep particles small in count. The charm should come from timing, color, and sound, not particle density.

---

## 17. Audio Direction

**Audio tone:** Soft, bright, magical, not loud or arcade-harsh.

Required sounds:

- UI tap.
- Place grove.
- Correct mote pickup.
- Wrong mote tap.
- Combo bonus.
- Bloom victory.
- Time-out result.

Optional:

- Quiet ambient loop during gameplay.
- Slight seed hum when target color changes.

Volume recommendations:

- UI: clear but soft.
- Pickup: frequent, so keep it short.
- Music/ambience: low and optional.
- Victory: satisfying but not long.

---

## 18. Suggested Free Assets

All listed assets were selected because they are free and suitable for a small prototype. Links and license labels were rechecked on 2026-05-31. The implementation chat should still verify final downloaded files and include license text in the project if required by the chosen distribution workflow.

| Asset | Use | Source | License | Notes |
|---|---|---|---|---|
| Kenney Nature Kit | Grove base props: trees, rocks, foliage, terrain-like pieces | https://kenney.nl/assets/nature-kit | Creative Commons CC0 | Primary 3D visual pack. The source page lists 330 files and CC0 license. Good fit for low-poly mobile AR. |
| Kenney Particle Pack | Sparkles, glow textures, pickup trails, bloom burst sprites | https://kenney.nl/assets/particle-pack | Creative Commons CC0 | Source page lists 80 VFX files, 512 x 512 tile size, CC0 license. |
| Kenney UI Audio | Menu clicks, toggles, soft UI feedback | https://kenney.nl/assets/ui-audio | Creative Commons CC0 | Source page lists 50 audio files and CC0 license. |
| Kenney Music Jingles | Short victory/time-out/result stingers | https://www.kenney.nl/assets/music-jingles | Creative Commons CC0 | Source page lists 85 audio files and CC0 license. Use only a few. |
| Magic Spell SFX by JaggedStone | Correct pickup, seed pulse, bloom magical effects | https://opengameart.org/content/magic-spell-sfx | CC0 | OpenGameArt page lists CC0 and several small OGG files. |
| 80 CC0 RPG SFX by rubberduck | Optional item/gem/wood/stone/spell sounds | https://opengameart.org/content/80-cc0-rpg-sfx | CC0 | Useful backup sound set. Use selectively to avoid audio style mismatch. |
| Calm Loop by wipics | Optional quiet background music | https://opengameart.org/content/calm-loop | CC0 | Small MP3 loop. Only include if it feels calm in AR and does not annoy on repeat. |
| Poly Haven assets | Optional replacement rocks/plants/textures | https://polyhaven.com/ and https://polyhaven.com/license | CC0 | Use sparingly. Poly Haven assets can be higher detail, so optimize or avoid for MVP if performance is uncertain. |

### Recommended MVP Asset Choices

Use this minimal set first:

- Kenney Nature Kit for all 3D grove props.
- Kenney Particle Pack for VFX sprites.
- Kenney UI Audio for interface sounds.
- Magic Spell SFX for pickup and bloom sounds.

Skip optional music until the core loop feels good.

---

## 19. Asset Integration Notes for the Technical Chat

These are design-facing requirements, not a required implementation recipe:

- Prefer low-poly props from one visual family for consistency.
- Keep total unique materials low.
- Keep visible prop count modest.
- Use simple emissive materials for motes.
- The seed can be made from primitives if a suitable plant model is not immediately available.
- If importing Poly Haven assets, check polygon counts and texture sizes before using them in mobile AR.
- Do not block the prototype on perfect asset selection. The game should be playable with primitive motes and a simple low-poly garden.

---

## 20. MVP Implementation Acceptance Criteria

The prototype is acceptable when:

- The app opens to a menu.
- Player can enter AR placement.
- A flat surface can be detected on a real device.
- Player can place one grove.
- Player can start a 90-second round.
- Motes spawn around the grove.
- Player can tap motes.
- Correct and wrong taps behave differently.
- Bloom progress can reach victory.
- Timer can reach time-out.
- Results screen appears.
- Sound can be turned off.
- The app remains responsive during normal play.
- The grove and motes are visually framed well enough to understand the game.

---

## 21. Practical MVP Cut Line

If time gets tight, protect the playable loop first. The project should cut or simplify features in this order.

**Must keep:**

- AR placement on a flat surface.
- One anchored grove.
- Target color shown by seed and HUD.
- Tappable motes with forgiving hit areas.
- Correct tap, wrong tap, timer, bloom progress, and results.

**Can be simplified without breaking the game:**

- Seed bloom animation can be a scale-up, material color change, and particle burst.
- Mote flight into the seed can be a simple lerp or tween.
- How To can be one overlay instead of a separate polished screen.
- Credits can be a plain scrollable text screen.
- The grove can use a small fixed prop layout instead of procedural decoration.

**Cut first if needed:**

- Splash screen.
- Haptics.
- Best score.
- Grove size toggle.
- Ambient music.
- Extra post-victory sparkle interaction.

This cut line keeps the result playable and demoable instead of spreading effort across polish before the core loop works.

---

## 22. Practical State Machine

The implementation can be simple if it follows a small set of states:

1. **Boot**
   - Load settings and required assets.
2. **MainMenu**
   - Show Play, How To, Settings, Credits.
3. **PermissionCheck**
   - Request AR/camera permission if needed.
   - If permission is denied, or the device does not support AR (no ARCore), show a calm blocking message with Back to Menu. Never dead-end.
4. **PlacementSearching**
   - Show camera, plane detection guide, and reticle when possible.
5. **GrovePreview**
   - Grove is placed but round has not started.
   - Player can Start or Replace.
6. **Playing**
   - Timer runs, motes spawn, taps are evaluated.
7. **Paused**
   - Timer and spawning stop.
8. **TrackingLost**
   - Timer and spawning stop until tracking recovers.
9. **RoundComplete**
   - Victory or time-out feedback plays briefly.
10. **Results**
   - Score and replay options are shown.

This avoids ambiguous transitions and makes the app easier to debug.

---

## 23. Practical Playability Checks

Before calling the prototype fun, run these checks with a real phone:

- A new player can understand what to tap within 10 seconds of the round starting.
- The player gets at least one correct tap in the first 15 seconds.
- A normal first-time player can complete the bloom without developer coaching.
- Wrong motes are visible enough to create choice, but not so numerous that the screen feels noisy.
- The player does not need to physically walk around the room.
- The game still works if the player stays seated and only leans or rotates the phone.
- The timer creates light urgency, not panic.
- The bloom result feels like a reward, even if the score system is simple.

If these checks fail, tune mote size, spawn positions, target brightness, and timer length before adding new features.

---

## 24. Demo Build Definition

The game is ready to show to other people when it meets this stricter demo bar:

- It has been tested on a real AR-capable phone, not only in editor simulation.
- The player can go from launch to a placed grove without developer help.
- The grove contains dressed art, not only primitive placeholders.
- Motes are large enough to tap reliably with one thumb.
- At least one victory round and one time-out round have been tested.
- Tracking loss pauses the round instead of creating an unfair failure.
- There is no visible debug UI, editor-only text, or placeholder file names.
- Audio feedback exists for placement, correct taps, wrong taps, and victory.
- The credits screen lists every imported third-party asset used in the build.
- The full demo can be explained to a new player in one sentence: "Place the grove and tap lights that match the seed color."

If any of these fail, the build may still be a useful prototype, but it is not yet a clean demo.

---

## 25. Performance Guardrails

The game should prioritize stable AR tracking and frame rate over visual complexity.

Recommended guardrails:

- Keep active motes at 8 to 12.
- Keep particle systems short-lived.
- Limit simultaneous particle bursts.
- Avoid real-time shadows if the device struggles.
- Avoid transparent overdraw filling the screen.
- Avoid high-resolution textures for tiny props.
- Avoid continuous expensive physics checks.
- Use simple tap hit detection against mote objects.
- Do not spawn objects outside the compact grove radius.

Performance should be tested on the weakest available target device, not only in the editor.

---

## 26. Accessibility and Comfort

MVP accessibility and comfort requirements:

- No penalty for empty-space taps.
- Wrong-tap penalty is mild.
- Timer pauses during tracking loss.
- Settings include sound off.
- Important UI uses large enough text and strong contrast.

Recommended accessibility improvement if time allows:

- Color assist mode: target motes have simple shape markers as well as colors.

Comfort requirements:

- No sudden full-screen flashes.
- No required fast spinning.
- No required walking backward.
- No horror or jump-scare elements.
- No gameplay that encourages ignoring real surroundings.

---

## 27. Tutorial Copy

Keep copy short. Suggested English UI text:

- **Find a flat surface**
- **Tap to place the grove**
- **Match the seed color**
- **Tap glowing motes**
- **Fill the bloom before time runs out**
- **Tracking lost. Point back at the grove.**
- **Full Bloom!**
- **Still Growing**

Do not add long tutorial paragraphs in the game UI.

---

## 28. Data and Persistence

MVP persistence:

- Sound setting.
- Haptics setting if implemented.
- Best score if trivial.
- A "first launch seen" flag, so the one-time safety note (see User Flow) shows only on first run.

Do not add:

- User profiles.
- Cloud sync.
- Analytics.
- Remote config.
- Inventory.
- Unlock trees.

---

## 29. Stretch Goals

Only consider these after the MVP is playable on a device:

1. **Color Assist Shapes**
   - Sun motes are circles.
   - Leaf motes are diamonds.
   - Moon motes are crescents or rings.

2. **Endless Calm Mode**
   - No timer.
   - Motes keep spawning.
   - Player blooms the grove at their own pace.

3. **Shade Puffs**
   - Rare dark puffs drift toward the seed.
   - Tapping one clears it.
   - If ignored, it reduces bloom progress by 1.
   - This adds tension, but it should not be in the first MVP if time is tight.

4. **Photo Moment**
   - After victory, hide HUD and let player take a screenshot with the bloomed grove.

5. **Alternate Grove Themes**
   - Stone circle.
   - Mushroom patch.
   - Tiny pond.

Do not implement stretch goals before the base round is fun and stable.

---

## 30. Risks and Mitigations

| Risk | Why it matters | Mitigation |
|---|---|---|
| AR placement feels unreliable | First impression depends on it | Keep placement UI simple, pause during tracking loss, test on device early. |
| Motes are hard to tap | AR camera movement makes precision harder | Make motes larger than realistic scale, use forgiving hit areas. |
| Visuals look empty | A plain AR prototype can feel underwhelming | Use low-poly props, glow, particles, and sound from the start. |
| Performance drops | Mobile AR is sensitive to rendering cost | Use few props, few particles, low-res textures, and simple lighting. |
| Scope grows | AR novelty invites too many ideas | Keep only Quick Bloom in MVP; put all extras in Stretch Goals. |
| Colors are not accessible | Color matching can exclude some players | Add shape markers as a simple stretch or toggle if time allows. |

---

## 31. Development Order Recommendation

Suggested order for the technical chat:

1. Create simple menu flow.
2. Implement AR placement with a primitive placeholder grove.
3. Spawn tappable primitive motes around the anchor.
4. Implement target color, scoring, timer, and bloom meter.
5. Add correct/wrong feedback.
6. Add result screen.
7. Import Kenney Nature Kit props and dress the grove.
8. Add particles.
9. Add sounds.
10. Test on real device and tune sizes/timing.

This order ensures the game becomes playable before art polish.

---

## 32. Final Sanity Check

This design has been checked for common first-AR-project problems:

- **No oversized feature set:** The MVP has one mode, one anchor, one round structure, and one interaction type.
- **No dependency on advanced AR features:** Plane detection and tap interaction are enough.
- **No contradiction between cozy tone and scoring:** Score and timer add replay value, but failure remains gentle.
- **No contradiction between beauty and performance:** The visual plan relies on low-poly props, glow, and short particles rather than heavy simulation.
- **No required paid assets:** The recommended asset set is free and CC0.
- **No unclear win condition:** Collect 15 correct motes before the timer ends.
- **No unclear player action:** Place grove, start round, tap matching motes.
- **No hidden campaign scope:** Progression, unlocks, and extra modes are explicitly out of MVP.
- **No technical overcommitment:** The document describes desired behavior while leaving engine/framework choices to the implementation chat.

The final intended MVP is a compact, polished AR toy-game: place a tiny grove, catch matching lights, bloom the seed, replay for a better score.
