# Porpezite

An aggressive performance resource pack for **Minecraft Bedrock Edition** (1.20+), in **three builds** with the same four modes.
It removes particles, freezes and strips animations and, depending on the build, hides sky / weather clutter, lowers the first-person view and limits the visible distance with fog.
It changes no gameplay, needs no behavior pack and works on any server.

Version 2.2.0 | Manifest format 2 | License: see `LICENSE.txt`

> Made by the author (Kijanggorreng) with the help of Claude, an AI assistant made by Anthropic.
> Not an official Minecraft product. Not approved by or associated with Mojang or Microsoft.

## Pick a build

| Build | File | What it is | Pick it if |
|---|---|---|---|
| **Normal** | `Porpezite_Normal.mcpack` | Everything: fog view limit (4 / 2 / 3x3 / 1 chunk) plus the view-clarity extras (low shield and arm, hidden overlays, ...) | you want the full package |
| **No Fog** | `Porpezite_NoFog.mcpack` | Normal without any fog or view limit. Vanilla fog stays | you want the cuts but keep the normal view distance |
| **Pure** | `Porpezite_Pure.mcpack` | No Fog minus every change that cannot save work even in theory | you want only changes with a real mechanism |

The unpacked source of the **Normal** build is this repository. The `.mcpack` files of all three builds are attached to the GitHub Releases.
Do not activate two builds at the same time.

## Install

1. Download a `.mcpack` from the Releases page and open it. Minecraft imports it automatically.
2. Go to **Settings > Global Resources**, select **Porpezite** and press **Activate**.
3. Open the pack's settings (gear icon) and move the slider to choose a mode.

## Modes

Each mode includes everything from the modes above it.

| Mode | Normal | No Fog | Pure |
|---|---|---|---|
| **LITE** | Fog view 4 chunks (64 blocks). Zero particles, frozen low fire, hidden clouds / pumpkin overlay / XP orbs, lower right arm, low shield, no walking hand-bob | Same, without fog | Zero particles, frozen low fire |
| **MAX** | Fog view 2 chunks (32 blocks). Frozen block animations, no shadows, no enchant glint, no rain / snow, no block-crack overlay, static end crystal, no mob eating / breeding / head tracking | Same, without fog | Frozen block animations, static end crystal, no mob eating / breeding / head tracking |
| **ULTRA** | Fog view 3x3 chunks (16 blocks). No sun, moon, vignette or powder-snow overlay, minimal mob animation (small leg swing for quadrupeds) | Same, without fog | Minimal mob animation |
| **ULTRA MAX** | Fog view your own chunk only (8 blocks). Every non-gameplay mob animation stripped, lowest shield and arm | Same, without fog | Every non-gameplay mob animation stripped |

## What actually saves work, and how much

These are **rough expectations, not measurements**. Results depend on hardware, scene and server. Gains are usually largest on weak CPUs, phones and busy scenes.

| Change | Builds | Mechanism | Rough impact |
|---|---|---|---|
| Particles removed (about 160 vanilla particle ids spawn nothing) | all | less CPU simulation and GPU drawing | **High** in busy scenes (explosions, fights, farms, many players). Near zero in an empty world. Usually the biggest pack-side gain |
| Mob animations cut (eating / breeding / head tracking, then minimal motion, then stripped) | all | fewer animation expressions evaluated per mob per frame | **Low to medium** with many mobs on a weak CPU. Small otherwise |
| Block and fire animations frozen on one frame | all | fewer texture-atlas updates per frame | **Low** (a few percent at most). Slightly more on weak or mobile GPUs |
| Static end crystal | all (MAX and up) | one constant pose instead of an animated one | **Very low**, only visible with many crystals |

### Visual only (no saving, even in theory)

Normal and No Fog keep these because they make the game clearer. **Pure removes them.**

| Change | Why it does not save work |
|---|---|
| Fog view limit (Normal only) | Fog only changes colors. Chunks behind it are still loaded and drawn |
| Transparent clouds, pumpkin overlay, XP orbs, sun, moon, vignette, powder-snow overlay, block-crack overlay | A transparent texture is still drawn, an XP orb is still simulated |
| No entity shadows, no enchant glint, no rain / snow | The shadow quad, the glint pass and the weather quads are still drawn, only with an invisible texture |
| Transparent particle atlases (safety net) | Only hides particles that were missed. They still cost the same |
| Lower right arm, low shield, no walking hand-bob | Moves things on screen. Draw cost is identical |
| Short flame shape of the fire | Cosmetic. The fire is frozen in every build because freezing is what saves work |

For a **real** saving of the visual side, use the game settings (clouds off, weather and particles settings) instead of transparent textures.

## Settings that matter more than any pack

Lowest **render distance** and **simulation distance** you accept, smooth lighting off, fancy graphics / fancy leaves off, VSync off, unlimited max frame rate.
On a server, the server decides its own distances.

## How to measure it yourself

Same world, same spot, same view, about 60 seconds. Compare **average FPS and 1% lows** with the pack off and on each mode, using an external overlay
(for example Xbox Game Bar on Windows or MSI Afterburner with RTSS). Repeat in a busy scene (mobs, redstone, fights) because that is where the pack helps most.

## What a resource pack cannot do

- Force render distance, simulation distance or VSync.
- Stop chunks behind fog from being loaded or drawn, cull caves, or limit dropped items.
- Remove piston, hopper or chest animations (engine-side block-entity rendering). Barrels have no animation.
- Show an FPS counter with CPU, GPU or memory load. JSON UI and Molang have no data source for it.
- Remove particles that the engine hard-codes on the terrain atlas (for example some block-break effects).

## Known limitations and unverified parts

- Checked with automated static validation only (JSON, manifest, references, per-mode contents, Molang syntax). **Verify in the game.**
- Shield and right-arm positions (Normal and No Fog) are calculated, not tuned from screenshots. The axes of first-person attachables are easy to get wrong, so the offsets may need adjusting.
- The end crystal pose assumes certain bone names. If they differ, the crystal simply stays in its base pose.
- Mob animation overrides live in separate `pz_*.animation.json` files. If the game does not override by identifier, the result is no effect, not a broken mob.
- Texture paths of the visual-only overlays are best effort. A wrong path is harmless.
- Mob head tracking is removed only for generic mobs. Humanoid mobs share animations with the player and are left alone.

## How it works (short)

- **Particles:** each vanilla particle id is redefined with an emitter that spawns nothing.
- **Fog (Normal):** vanilla fog ids cannot be redefined, so `biomes_client.json` maps every biome to a custom fog id. Each mode folder gives that id its own distance.
- **Fire:** `fire_0/1` and `soul_fire_0/1` are replaced with short single-frame textures and the flipbook entries are frozen on frame 0, so the frame count always matches.
- **Subpacks:** the mode slider selects a subpack folder. A file in the folder replaces the root file with the same path.
- Details for developers and AI assistants, including the exact file differences between builds, are in [`CLAUDE.md`](CLAUDE.md).

## Layout of this repository (the Normal build)

```
manifest.json, pack_icon.png, LICENSE.txt, README.md, CLAUDE.md, AGENTS.md, CHANGELOG.md
biomes_client.json, fogs/                 fallback no-fog definition
particles/                                about 160 zeroed particles
animations/                               shield and first-person arm
textures/                                 hidden overlays, low fire, fire flipbook
subpacks/lite | max | ultra | ultramax/   per-mode fog, flipbooks, overlays, mob animation, crystal
```

## Reuse and license

You may learn from the techniques and write your own version. Do not copy the files verbatim, and do not reuse the name "Porpezite" or the logo. See `LICENSE.txt`.

## Provenance

Particle, animation, fog and flipbook identifiers are functional game names. Normal and No Fog include modified excerpts of vanilla first-person arm and shield animation data (Mojang game data). Pure does not.
No textures, models or sounds are taken from Minecraft or from other packs. The logo was supplied by the author.
Reference material used while building: the Bedrock Wiki (vanilla particle and fog id lists) and Microsoft's Minecraft Creator documentation.
