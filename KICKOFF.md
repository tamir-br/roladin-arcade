# Kickoff (for Tamir)

## One-time setup
1. Create a public GitHub repo named `roladin-arcade` under `tamir-br`. Enable Pages with "GitHub Actions" as the source.
2. Copy the contents of this kit into the repo root (CLAUDE.md, docs/, .claude/, data/, config/).
3. Open the repo in Claude Code on your own machine (terminal or desktop app). Use Opus for the main session.
4. For testing on your phone, run the dev server with `--host` and open it over the same Wi-Fi.

## Paste this as the first message
```
Read CLAUDE.md, then docs/SPEC.md, docs/GAMES.md and docs/WORKFLOW.md. You are the orchestrator.

Start Phase 0 only:
1. Image task first. Fetch https://www.roladin.co.il/%D7%9E%D7%AA%D7%97%D7%9D-%D7%94%D7%97%D7%92%D7%99%D7%9D/%D7%97%D7%A0%D7%95%D7%9B%D7%94/ , extract the donut image URLs, download them into assets/raw/, and map them to data/donuts.json by filename and page order (hints are already in the file). The first page image is a hero group shot, not a single donut. If downloads are blocked, tell me and I will save the files by hand. Then run background removal, normalize canvas size and baseline, export WebP to public/donuts/. Show me a contact sheet of all results.
2. Scaffold the project, shared design tokens, shared components, i18n, share() and track() modules, the hub shell with a stub game, and the gates.
3. Build a style board page (palette, fonts, buttons, banner, HUD score, one donut with the sticker treatment) and stop for my approval before any game work.

Ask me any questions you need in one batch.
```

## After you approve the style board
Tell the orchestrator "design approved, start Phase 1". It builds the games in parallel worktrees.

## Later
- When you have the competition rules, edit `config/brand.config.json` (hashtag, brand wording).
- When you create the Umami Cloud account (Hobby plan), put the website ID in `config/brand.config.json` under `analytics.websiteId` and set `enabled` to true.
