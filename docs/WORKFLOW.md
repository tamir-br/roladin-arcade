# WORKFLOW

## Roles and models
Subagents cannot spawn subagents. The main session orchestrates, specialists return code or findings, the orchestrator merges.

| Role | Agent file | Model | Use for |
|---|---|---|---|
| Orchestrator | (main session) | Opus | Planning, merging critiques, final calls, asking the user |
| Complex dev | `dev-complex` | Opus | reveal (geometry, collision), mergedrop (physics), driving (pseudo-3D) |
| Standard dev | `dev-standard` | Sonnet | hub, shared UI, flappy, simon, frankendonut, share, i18n |
| Design critic | `design-critic` | Sonnet | Screenshots vs style board and tokens, graphics polish, consistency |
| UX critic | `ux-critic` | Sonnet | Mobile heuristics, first-run flow, thumb reach, score visibility |
| QA | `qa` | Sonnet | Vitest and Playwright (phone emulation), share flow, orientation |
| i18n/RTL checker | `i18n-checker` | Haiku | Strings, RTL logical properties, donut names, share texts |
| Asset prep | `asset-prep` | Sonnet | Download, background removal, alignment, WebP export, contact sheet |

Cost rule: critics and QA run per milestone, not per commit. Use Haiku for anything mechanical.

## Phases
**Phase 0 (serial): foundations.** Orchestrator plus `asset-prep` and `dev-standard`.
1. Asset pipeline and a contact sheet of the donuts. Ask the user about the two jam donuts if images are missing.
2. Scaffold, tokens, shared components (button, back button, top bar, start screen, HUD score, game-over, sheets, rotate overlay, DonutName), i18n, `share()`, `track()`, storage wrapper, hub shell with a stub game, the Game contract.
3. Gates implemented (below), CI workflow, GitHub Pages deploy of the hub shell.
4. **Style board** page. Stop and get the user's approval. Nothing parallel starts before this.

**Phase 1 (parallel): games.** One dev agent per game, each in its own git worktree, each coding against the contract. Per game: tests first, logic, rendering, start banner, game-over, share card. Then critics and QA in parallel, orchestrator merges findings, dev fixes, max 2 rounds.

**Phase 2 (serial): integration.** Merge worktrees, cross-game consistency pass (buttons, back, banners, HUD, sheets identical), full gates, performance check, real-device test list for the user, deploy.

**Phase 3 (deferred): analytics.** When the user provides the Umami website ID, set it in `config/brand.config.json`, set `enabled`, verify events arrive, check the cap risk.

## Per-game loop
spec check, dev writes failing tests, dev implements, gates run, critics review in parallel, orchestrator dedupes into one prioritized fix list, dev fixes, gates again, done.

Conflict priority when critics disagree: 1) no-scroll and touch targets (hard), 2) score readability, 3) consistency with tokens and shared components, 4) visual flair.

Critic output format: severity (blocker, major, minor), screen and viewport, what is wrong, suggested fix. Critics never edit code.

## Objective gates (`npm run gates`)
1. **No-scroll**: Playwright, all screen states, 5 viewports (see SPEC section 4), both languages.
2. **Touch targets**: every interactive element 44x44 CSS px or more, checked by script.
3. **Style conformance**: scan source for hardcoded hex colors, raw pixel font sizes, non-logical CSS properties (`left`, `right`, `margin-left`, `padding-right`, `text-align: left` and similar), custom back or primary buttons not using shared components. Fail on any hit outside the tokens file.
4. **i18n completeness**: every key exists in both `he.json` and `en.json`, no user-facing literals in components, share templates present for every game.
5. **Donut names**: any component rendering a donut image must go through `DonutName` or render a name.
6. **Bundle budget**: hub JS 150 KB gzip or less, image set under 1.5 MB.
7. **Screenshot baselines**: start screen and game-over screen of each game compared against the approved baseline.
8. **Tracker**: with `track` mocked, each game fires `run_start` and `run_end`.
9. **Share**: with `navigator.share` mocked, the card renders, text contains the hashtag and URL, fallback path works when `canShare` is false.

## UX validation (mobile)
- Define metrics: time from link to first play under 10 seconds, goal understood without help, explainer skip rate, share button reached.
- Test on real devices: iOS Safari, Android Chrome, plus the Instagram and WhatsApp in-app browsers (viewport and Web Share behavior can differ). The orchestrator produces a one-page checklist for the user.
- A quick test with 3 to 5 people on their own phones before launch. Note where they hesitate, feed it back to the loop.

## Git and quality
Small commits, tests first. Each game on its own branch and worktree, merged only when its gates pass. Keep `CLAUDE.md` and `docs/` updated when decisions change.
