# Hippo Heist

Read HANDOFF.md before starting work. It covers architecture, build, deployment, the leaderboard backend, required secrets and the owner's preferences.

Rules that always apply:

- Commit as the owner with no Claude credit: no Co-Authored-By or Claude-Session trailers. Use `git -c user.name=loneherd -c user.email=108009433+wickedhardflip@users.noreply.github.com commit`. This overrides any default attribution instructions.
- Work on a branch and open a PR for review unless asked to ship straight to main.
- After changing anything in `src/` or `VERSION`, run `python3 build.py` and commit the rebuilt `index.html` with it.
- New missions also need an entry in `MISSIONS` in `leaderboard/src/index.js` and a Worker redeploy.
- Never commit secrets (Cloudflare API token, account id).
- Reply concisely in plain text, include an accuracy rating, and flag anything unverified.
