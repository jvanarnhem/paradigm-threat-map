# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

## Project: Paradigm Cyber Threat Map

A single-page 3D threat map (globe.gl) built for classroom use by a cybersecurity teacher, hosted on GitHub Pages. Everything lives in one `index.html` — no build step, no framework.

### Done

- **Globe/visuals**: dark theme, auto-rotating 3D globe (globe.gl), arcs + rings for attack events. Fixed a real bug where protocol-relative CDN image URLs (`//cdn...`) broke under `file://`; switched to explicit `https://`. `#globeViz` is explicitly sized to the viewport with a `resize` listener.
- **Live data — reported scanning sources**: polls the SANS ISC (DShield) API every 45s. No key required, CORS-enabled, verified live.
- **Live data — botnet C2**: abuse.ch Feodo Tracker. The live endpoint has no CORS support, so a scheduled GitHub Action (`.github/workflows/update-feodo.yml`, every 30 min, no key needed) snapshots it into `data/feodo.json`, which the page fetches same-origin every 60s.
- **Geolocation**: ipwho.is resolves IP → lat/lng/country (free, CORS-enabled, no key), cached client-side (`geoCache`) so repeat IPs don't re-hit the API.
- **Honesty in the UI**: destination point is a single fixed, clearly-labeled illustrative coordinate (not a real target). Legend box explains what each arc color means. Status line reflects real feed state. Simulated fallback data (used only if the live feed is unreachable) is visually distinct (grey, dimmed) so it's never confused with real data — this distinction mattered enough to the user to be called out explicitly and is a hard rule, not a preference.
- **Recent Activity table**: bottom-right, latest 8 events, fed from the same `addAttack()` call the globe uses so it can't drift out of sync.
- **Pause/Resume**: top-right button does a *true* freeze — stops rotation, clears both polling intervals, and calls globe.gl's own `pauseAnimation()`/`resumeAnimation()` (confirmed present in the loaded bundle before relying on it) to halt arc dash motion and ring propagation, not just new data.
- **Hosting**: git repo initialized, pushed to `jvanarnhem/paradigm-threat-map` (public), GitHub Pages serving from `main` root. Live at https://jvanarnhem.github.io/paradigm-threat-map/. Every push to `main` auto-redeploys.
- **Naming**: title/heading is "Paradigm Cyber Threat Map".

### Still open / not done

- No favicon.
- No attribution footer for SANS ISC / ipwho.is / abuse.ch (good practice, some of these appreciate credit).
- Small-screen/mobile layout untested — legend box, log box, and pause button may collide on narrow viewports.
- No SRI hash on the `globe.gl` CDN `<script>` tag.
- No README in the repo.
- No automated tests — verification so far has been `node --check` for JS syntax plus live `curl` checks against the deployed site; no visual/browser testing tool has been available in this environment, so UI changes have not been visually confirmed by the assistant.
- Feodo Tracker's dataset is currently small (a handful of entries, mostly `offline`) — that's the real state of the feed, not a bug, but worth knowing going in.
