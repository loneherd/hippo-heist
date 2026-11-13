# Hippo Heist v2 Rebuild — Tasks

> Finished items live in `TASKS_HISTORY.md`. Do not read it at session start; search it only when asked about something you cannot find here.

Started 2026-09-28. Process: brainstorm → spec → mockup (approve) → plan → build. Nothing on `main` changes until Brian approves the merge.

## Decisions (locked 2026-09-28)
- Direction: **polished 2D** (2.5D hybrid saved for a later version)
- Art: ~~vibrant cartoon vector~~ **REOPENED 2026-09-29**: Brian wants the graphics to be the top priority: professional, a distinct style, not too cartoonish, NO balloon/inflated look, textured water + ground, an organic fresh map (not boxy). Bloomtown: A Different Story is a reference for mood (not to copy). Brian offered to generate images with Gemini or install other tools.
- Tech: **Phaser 3 + Vite**, deploy to **GitHub Pages**, PWA (home-screen install, offline)
- Audio: **free CC0 assets (Kenney/OpenGameArt) + Web Audio synth effects**
- Mobile: **landscape**, floating joystick left, action/ability buttons right, rotate prompt in portrait
- Scope: **expand significantly**: hippo abilities, power-ups, more enemy types, progression/unlocks
- Rollback: tag current `main` as `v1-original`; all work on a `rebuild` branch

## Assumptions (confirmed 2026-09-29)
- Core loop kept: steal bananas → feed leopard → avoid farmers → water = safe zone
- Local save only (no accounts/leaderboards)
- Same repo (`loneherd/hippo-heist`, public)

## Phasing (confirmed 2026-09-29)
1. Mockup: clickable art/feel prototype (title, world map, one level, HUD, mobile controls, sound sample)
2. Vertical slice: World 1 Farm, ~4 levels, dash+splash, vision-cone farmers, 2 power-ups, 3-star scoring
3. Content: Worlds 2 (River Delta) + 3 (Jungle), dogs, scarecrows, tractor boss, hats/skins (~12 levels total)

## Checklist
- [ ] **← START HERE:** Write Plan B (`docs/superpowers/plans/`, use writing-plans): obstacles/cover + Mud Coat → L3 The Barnyard; Splash + lanterns/dusk + Banana Magnet → L4 Harvest Dusk; music (CC0); iPhone test (verify the iOS audio unlock on touch end); deferred minors (unknown levelId crash, Farmer stuckT while alert, webkitAudioContext); deploy with v1 at `/classic/`
- [ ] **Deploy requirement (Brian 9/29):** keep v1 playable after v2 launches, e.g. v1 at `/classic/`, v2 at root, with a "Play the original" link. (`v1-original` tag = exact copy.)
- [ ] Implementation plan (writing-plans) → Brian picks execution method
- [ ] Build vertical slice
