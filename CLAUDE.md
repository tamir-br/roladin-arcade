# Roladin Hanukkah Arcade: master brief

You are the **orchestrator** (main session). Read this file first, then `docs/SPEC.md`, `docs/GAMES.md`, `docs/WORKFLOW.md`.

## What we are building
A mobile-first set of 6 short browser games using Roladin's Hanukkah sufganiya (donut) side-view images, entered in a likely Roladin Hanukkah competition. One static site on GitHub Pages. One hub page is the only link people share. Every game ends in a shareable result card and a prefilled post text with the brand hashtag.

Site title: **ROLADIN HANUKKAH ARCADE** (English). Everything else is Hebrew by default with an English toggle. All layouts are RTL in both languages.

## Non-negotiables (hard gates, see docs/WORKFLOW.md)
1. **No-scroll screens.** Every screen state fits the viewport on every phone size. `100dvh`, `overflow: hidden`. Content scales to fit, it never scrolls.
2. **Mobile first.** Portrait 390x844 is the design size. Worst case 360x640. Driving is the only landscape game. Touch targets 44px or more.
3. **One design system.** All games use the shared tokens and shared components (buttons, back button, top bar, start banner, HUD score, game-over, sheets). No hardcoded colors, sizes, or custom back buttons in a game.
4. **RTL + i18n from day one.** CSS logical properties only. No hardcoded user-facing strings: use `he.json` / `en.json` keys. Gameplay input is not mirrored (swipe and drag follow the physical screen).
5. **Donut names.** Whenever a whole donut is shown, show its catalogue name from `data/donuts.json`, through the shared `DonutName` component.
6. **TDD.** Failing test first. Game logic is pure TypeScript in `src/games/<id>/logic/`, rendering is a thin layer. Seeded RNG so logic is deterministic in tests.
7. **Professional graphics.** Polished flat-vector arcade look (see docs/SPEC.md, Visual direction). Donuts get one shared sticker treatment.
8. **No server.** Static site only. No user data stored. Scores live in localStorage (always inside try/catch).

## Stack
TypeScript, Vite, Vitest, Playwright (Chromium and WebKit with phone emulation). Hub and UI are DOM + CSS. Canvas games use Phaser 3 loaded lazily per game (Matter physics for Merge Drop) unless the Phase 0 size spike says a lighter custom canvas loop is better. Hash routing (`#/reveal`), Vite `base` set for GitHub Pages. Deploy with a GitHub Actions workflow.

## How you work
- Subagents cannot spawn subagents. **You are the only orchestrator.** Specialists are defined in `.claude/agents/`. They return findings or code; you relay and merge.
- Phase 0 is serial (foundations). After the design requirements are approved by the user, game builds run **in parallel**, one dev agent per game in its own git worktree.
- Critics (design, UX, i18n/RTL) and QA review in parallel, you merge their findings into one fix list, the dev fixes. Maximum 2 rounds per milestone, then ship or ask the user.
- **Model routing**: Opus for orchestration and the hard games, Sonnet for standard dev, critics, QA and asset work, Haiku for repetitive checks. Details in docs/WORKFLOW.md.
- Run the objective gates before any critic round. Critics are for judgment, gates are for facts.
- Ask the user questions when a decision is theirs (brand rules, taste calls). Batch questions, do not interrupt for small calls.

## Open items (keep as config, do not hardcode)
- Competition rules: hashtag, brand wording, logo and image usage. Everything lives in `config/brand.config.json`. Current values are placeholders.
- Image rights: confirm Roladin permission before public launch.
- **Deferred, keep in plan: analytics via Umami Cloud (Hobby plan).** Build the `track()` wrapper now (no-op while `analytics.websiteId` is null). The user will create the Umami account and provide the website ID later. Event list is in docs/SPEC.md.
- Two jam donuts (classic strawberry, dulce de leche) may lack a clean single side-view image. See `data/donuts.json` `imageStatus`. All games must work with N donuts read from the data file, never a hardcoded 11.

## Commands you should create in Phase 0
`npm run dev`, `npm test`, `npm run test:e2e`, `npm run gates` (no-scroll, style conformance, i18n completeness, bundle budget), `npm run build`, `npm run assets` (image pipeline).
