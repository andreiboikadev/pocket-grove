# Asset Ledger

Last verified: 2026-06-01

Every third-party asset used in a build must have a row here. **Nothing enters a showable build
without a ledger row and a confirmed license.** Re-verify the license on the page at download time and,
if the distribution workflow requires it, include the license text in the repo. Record AI-generated
assets too (tool, prompt summary, date, rights note).

## Status

**Nothing imported yet.** The rows below are the GDD's recommended **CC0** set, marked **PLANNED**.
Fill in Download date / Local path / Modifications when each is actually imported, and change status.

| Asset | Use | Source | Author | License | Download date | Local path | Modifications | Status |
|---|---|---|---|---|---|---|---|---|
| Kenney Nature Kit | Grove props: trees, rocks, foliage | https://kenney.nl/assets/nature-kit | Kenney | CC0 | — | — | — | PLANNED |
| Kenney Particle Pack | VFX sprites: sparkle, glow, trails, burst | https://kenney.nl/assets/particle-pack | Kenney | CC0 | — | — | — | PLANNED |
| Kenney UI Audio | UI clicks/toggles | https://kenney.nl/assets/ui-audio | Kenney | CC0 | — | — | — | PLANNED |
| Magic Spell SFX | Pickup, seed pulse, bloom | https://opengameart.org/content/magic-spell-sfx | JaggedStone | CC0 | — | — | — | PLANNED |
| Kenney Music Jingles | Victory / timeout / result stingers | https://www.kenney.nl/assets/music-jingles | Kenney | CC0 | — | — | — | OPTIONAL |
| 80 CC0 RPG SFX | Backup item/spell sounds | https://opengameart.org/content/80-cc0-rpg-sfx | rubberduck | CC0 | — | — | — | OPTIONAL |
| Calm Loop | Quiet background music | https://opengameart.org/content/calm-loop | wipics | CC0 | — | — | — | OPTIONAL |
| Poly Haven assets | Higher-detail rock/plant/texture swaps | https://polyhaven.com/ (license: https://polyhaven.com/license) | Poly Haven | CC0 | — | — | — | OPTIONAL |

## Notes

- **Recommended MVP set:** Kenney Nature Kit (props) + Kenney Particle Pack (VFX) + Kenney UI Audio
  (UI) + Magic Spell SFX (pickup/bloom). Skip music until the core loop feels good.
- Prefer one low-poly visual family for consistency; keep unique materials and texture sizes low.
- Poly Haven assets can be high-detail — check polygon counts and texture sizes before using in mobile
  AR, or skip for MVP.
- The seed and motes can be primitives + emissive materials; don't block the prototype on perfect assets.
