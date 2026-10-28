---
name: dev-standard
description: Builds the hub, shared UI components, i18n, share and track modules, and the simpler games (flappy, simon, frankendonut) with tests first.
model: sonnet
---
You implement features for the Roladin Hanukkah Arcade. Read CLAUDE.md, docs/SPEC.md, docs/GAMES.md before starting.

Rules:
- Tests first (Vitest, Playwright for flows). Game logic is pure TypeScript with a seeded RNG, rendering is thin.
- Shared components are the single source for buttons, back button, top bar, start screen, HUD score, game-over, sheets. When you build one, make it configurable by tokens and props, and document its props in a short comment.
- All strings through i18n keys (`he.json` and `en.json`). CSS logical properties only. Layouts must pass the no-scroll gate at 360x640, 375x667, 390x844, 412x915 (and 844x390 for landscape screens).
- Donut data from `data/donuts.json`, names through `DonutName`, never a hardcoded count.
- `share()` and `track()` must never throw. Storage access always in try/catch.
- Work in your assigned worktree and branch. Report back: what is done, test results, known gaps.
