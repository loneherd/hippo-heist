# Hippo Heist: project handoff

This file is the single source of truth for continuing work on Hippo Heist, written 2026-11-17 when the project moved off its original Claude account. Everything needed to keep building is in this repository. Nothing in the old Claude project (chat history, project memory, the claude.ai artifact) is required.

Player-facing docs are in [README.md](README.md). Release notes are in [CHANGELOG.md](CHANGELOG.md).

## Current state

- Version 2.1 (the `VERSION` file). Live at https://loneherd.github.io/hippo-heist/
- The repository now lives at `loneherd/hippo-heist`. It used to be `wickedhardflip/hippo-heist`, and GitHub redirects the old name for git and the API, but the old Pages address (https://wickedhardflip.github.io/hippo-heist/) now returns 404. Older notes and commits mention the old address.
- Four 3D missions, touch controls, and an online leaderboard, all merged to `main` on 2026-10-17 (PRs 2 to 5).
- The original 2D game is playable at `/classic/`.
- All feature branches are merged. `main` is the only branch that matters.

## Owner's working preferences

- Commits must not credit Claude. No `Co-Authored-By: Claude` trailer, no `Claude-Session` trailer, and author/committer set to the owner, not `Claude <noreply@anthropic.com>`. The owner had Claude removed from the repo's contributor list once already. Commit with:

      git -c user.name=wickedhardflip -c user.email=108009433+wickedhardflip@users.noreply.github.com commit -m "..."

- Open a pull request for review rather than pushing straight to `main`, unless asked to ship directly.
- Keep communication concise and in plain text. Give an accuracy rating, flag assumptions, and say plainly when something was not verified (for example, the live site could not be opened).
- Ask before anything that cannot be undone (force pushes, deleting branches or tags, production changes to the leaderboard).

## Architecture

A single-page Three.js (r128, from cdnjs) game written in plain JavaScript. No bundler and no npm dependencies for the game. Sound is synthesized live with the Web Audio API, so there are no audio files. Textures and geometry are generated in code, so the game has no image assets either.

`build.py` concatenates the sources into one IIFE in this order, so later files can use globals from earlier ones:

| File | Role |
|------|------|
| `src/head.html` | Page markup and CSS: menus, HUD layers, fonts, the `__VERSION__` placeholder |
| `src/core.js` | Utilities (`$`, `rand`, `pick`), renderer, post-processing, geometry builders, collision, particles, procedural textures (`TEX`) |
| `src/audio.js` | `AU`: synthesized, spatialized sound and music with convolution reverb |
| `src/levels.js` | `WORLD` size, `DIFF` difficulty table, enemy tools, `POWER` power-ups, `LEVELS` (the four missions) |
| `src/world.js` | Terrain, water, sky, lighting, props, vegetation, village houses and lamps |
| `src/actors.js` | Hippo viewmodel and body, leopard, captive rigs, farmers, police, dogs, villagers |
| `src/game.js` | `G` game state, player movement, AI, objectives, killcam, cutscenes |
| `src/touch.js` | On-screen controls. Feeds `TKEYS` (read by `act()`) and `TSTICK` (read by `updatePlayer`) |
| `src/leaderboard.js` | Leaderboard UI and API client. `LB_API` holds the Worker URL |
| `src/hud.js` | HUD, overlay canvas, menus, input and key binding, main loop. Exposes `window.__hh` for testing |

Settings and key bindings are saved in the player's browser (localStorage).

### Adding a mission

1. Add an entry to `LEVELS` in `src/levels.js` (id, op, place, clock, seed, briefing, `need`, `cap`, `power`, waves).
2. Add any new terrain or props in `src/world.js` and enemies in `src/actors.js`.
3. Add the mission id and its `need` to `MISSIONS` in `leaderboard/src/index.js` and redeploy the Worker. Scores for a mission the Worker does not list are rejected.
4. Update README.md (missions table) and CHANGELOG.md, bump `VERSION`, rebuild.

## Build

    python3 build.py

This writes:

- `index.html`: the page GitHub Pages serves (committed). Includes Open Graph tags pointing at `preview.png`.
- `game.html`: the same game without the outer page wrapper (git-ignored). This is the format used for the old claude.ai artifact copy.
- `test.html`: loads a local `three.min.js` instead of the CDN (git-ignored), for offline and headless testing.

Always rebuild and commit `index.html` together with any change in `src/` or `VERSION`. Running the build on an unchanged tree produces no diff.

## Deployment

GitHub Pages, "Deploy from a branch", `main`, root folder. The built-in `pages-build-deployment` Action publishes every push to `main`, usually within a minute or two. There is no custom workflow in `.github/`. Check the Actions tab to confirm a deploy. Browsers cache the page, so a hard refresh may be needed to see a new version.

## Leaderboard backend

Code is in `leaderboard/` (see [leaderboard/README.md](leaderboard/README.md) for endpoints, cheat resistance and local testing).

- Cloudflare Worker `hippo-heist-leaderboard`, live at https://hippo-heist-leaderboard.bri-wag.workers.dev
- Cloudflare D1 database `hippo-heist-leaderboard` (its id is in `wrangler.toml`; that id is not a secret). Tables come from `schema.sql` and already exist.
- The Worker only accepts requests from the origins in `ALLOWED_ORIGINS` in `wrangler.toml` (`https://loneherd.github.io` and the old `https://wickedhardflip.github.io`). If the Pages address changes again, update this list and redeploy the Worker, or the leaderboard stops working on the live site. Any other copy of the game, including the claude.ai artifact, cannot post scores.
- For testing, `?lb=<url>` on the game URL points it at a different Worker.

Secrets and config that are not in the repo and must be provided again in any new environment:

| Name | Where it lives | Needed for |
|------|----------------|------------|
| `CLOUDFLARE_API_TOKEN` | Create in the Cloudflare dashboard (My Profile > API Tokens). Permissions: Account > Workers Scripts: Edit, Account > D1: Edit. Previously stored as an environment secret in the old Claude cloud environment, which goes away with the account. | `npx wrangler deploy`, D1 queries |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare dashboard, Workers & Pages overview (right sidebar). | `npx wrangler deploy` |
| `IP_SALT` (optional) | Worker secret, set with `npx wrangler secret put IP_SALT`. If unset, the Worker uses a public default string to hash IP addresses. | Privacy of stored IP hashes |

Never commit a token. A security check on 2026-10-21 confirmed no API token exists anywhere in the repo history or the live site.

Deploy:

    cd leaderboard
    CLOUDFLARE_ACCOUNT_ID=<id> CLOUDFLARE_API_TOKEN=<token> npx wrangler deploy

Or run `npx wrangler login` on a local machine instead of using a token. The same Cloudflare account also has an unrelated Worker (`little-darkness-8b7c`); leave it alone.

## Testing

There are no automated test files in the repo. Past work was tested headless:

1. `npm pack three@0.128.0`, extract it, and copy `build/three.min.js` next to `test.html` (cdnjs may be blocked in sandboxes).
2. `python3 build.py`, then open `test.html` in Playwright Chromium (SwiftShader works). Use `waitUntil: 'domcontentloaded'`.
3. Drive the game through `window.__hh` (for example `__hh.startMission(...)`, `__hh.step(600)`, `__hh.G`, `__hh.farmers`).

CORS errors for the leaderboard Worker in a local or `file://` run are expected.

## Missions

| # | Id | Operation | Location | Food needed | Notes |
|---|----|-----------|----------|-------------|-------|
| 1 | 1 | Chain Breaker | Lower Valley Plantation, sunrise | 16 bananas | Leopard chained in a fenced pen |
| 2 | 2 | Hard Cargo | Cooper Packing Yard, dusk | 18 bananas | Cage on a flatbed truck |
| 3 | 3 | Black Rain | Monsoon Ridge Sawmill, midnight storm | 20 bananas | Cage hoisted under the gantry, guard dogs |
| 4 | 4 | Last Stop | Millbrook Junction village, night | 20 cuts of meat | Circus train with a departure clock, police, villagers, houses to hide in. Carry cap 4 |

Story note: missions 1 to 3 establish that the leopard eats bananas. Mission 4 explains the switch to meat because the circus bought every banana in the county for its elephants. Keep new story text consistent with that.

## History

- 2025-12-05: original 2D arcade game, later given mobile joystick controls.
- 2026-02-06: Open Graph tags and logo social preview.
- 2026-10-16: 2.0, full 3D rewrite (PR 1). Three missions, seven power-ups, four difficulties, rebindable keys, burst sprint, 5-banana carry cap, killcam. 2D game moved to `/classic/`. Claude-authored commits were rewritten to the owner. Tag `v2.0.0`.
- 2026-10-17: 2.1. Touch controls (PR 2), Mission 4 Last Stop (PR 3), online leaderboard (PR 4), Mission 4 banana story fix (PR 5).
- 2026-10-21: leaderboard key exposure check, nothing exposed.
- 2026-11-17: repo found moved to `loneherd/hippo-heist` (new Pages address). Site URLs and Worker origins updated. This handoff file, `CLAUDE.md`, and early dev screenshots in `docs/screenshots/dev/`.

## Branches and tags

- `main`: current game. Keep it deployable.
- Tags `v1-original`, `v1.0-2d-original`, `v1.0.0` and branch `legacy-2d-original`: the original 2D game.
- Tag `v2.0.0`: the first 3D release.
- `new-version`, `mobile-touch-controls`, `leaderboard`, `village-mission`, `mission4-bananas`: merged feature branches, safe to delete if the owner wants.
- `rebuild`, `3d-rewrite`: older experiments. Leave them alone.
- A separate private repo `hippo-heist-3d` (under the same owner) exists. It is not the Pages site and is not used.

## Known issues and open ideas

- None of the missions have been hand-playtested at full frame rate on a real GPU. Earlier tests were headless with still screenshots.
- Mission 4 police balance may be too harsh. In a headless test six officers killed an idle hippo in about 6 seconds.
- Farmers chase about 13% faster than the original 3D build. Walking no longer outruns them but a burst does. This was a balance call that the owner has not confirmed.
- Leaderboard scores are computed in the browser and the Worker has no auth, so scripted fake scores are possible outside a browser (the Origin check can be spoofed). Rate limits (20 posts per hour per IP) contain it. Bad rows can be deleted from the D1 console.
- On 2026-11-17 the deployed Worker still only allowed `https://wickedhardflip.github.io`, so the live game at loneherd.github.io could not load or post scores. The fix is in `wrangler.toml` and takes effect only after a Worker redeploy. Check with:

      curl -s -X POST -H 'Origin: https://loneherd.github.io' -H 'content-type: application/json' -d '{"mission":1,"difficulty":"x"}' https://hippo-heist-leaderboard.bri-wag.workers.dev/run

  `bad difficulty` means the origin is allowed. `origin not allowed` means the redeploy is still needed.
- `IP_SALT` may not be set as a Worker secret, in which case IP hashes use a public default.

## Things that did not carry over from the old Claude account

- The claude.ai artifact copy of the game (https://claude.ai/artifact/VoDPMb2BUKyZaRrwtab5tX). It was an October 16 build of 2.0, older than this repo, and it is owned by the old account. Rebuild `game.html` and publish a new artifact if one is wanted.
- Claude project chat history and project memory. Everything useful from them is summarized in this file.
- The old cloud environment and its `CLOUDFLARE_API_TOKEN` secret. Create a new token and add it to the new environment.
- The Cloudflare connector on claude.ai. Reconnect it in the new account if wanted; it can query D1 but cannot deploy Workers.
- The Claude GitHub App link to this repo. Connect GitHub and install the Claude app on this repo from the new account so it can push and open PRs.
