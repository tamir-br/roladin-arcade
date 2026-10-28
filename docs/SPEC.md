# SPEC: Roladin Hanukkah Arcade

## 1. Product
A hub page with 6 games. People arrive from a shared post, pick a game, play 1 to 3 minutes, and share their result with a card image and prefilled text. The hub URL is the only link we share.

Audience: Israeli phone users, many arriving from Instagram, WhatsApp, Facebook in-app browsers. Default language Hebrew, English toggle in settings.

## 2. Hub screen (portrait, RTL, no scroll)
```
[gear]  top corner (RTL: right side)
ROLADIN HANUKKAH ARCADE   (English title, sunset sky, sun, perspective grid)
2 x 3 grid of game cards (art, Hebrew name, best score, (i) button)
footer: "TamirB ©" small text, tap opens the contact sheet
```
- Cards: tap opens the game's start screen. The (i) button opens a bottom sheet with a one-line goal and the control hint.
- Deep link: `/?game=<id>` pulses that card once. Hub is at `/`, games at `#/<id>`.
- There is no global donut selection. Each game offers donut choice on its own start screen where relevant (see docs/GAMES.md).
- Contact sheet (from the footer): "All rights reserved TamirB", Instagram https://www.instagram.com/tamir_br/ , Homepage https://tamir-br.github.io/ , one line that Roladin names, logos and sufganiya images belong to Roladin, one line "anonymous usage statistics" (only once analytics is enabled). Links open in a new tab with `rel="noopener"`.
- Settings sheet (hub gear, also inside pause): music on/off, SFX on/off, language Hebrew/English. Persisted in localStorage (try/catch).
- Hebrew by default except: the site title, "TamirB", and the rights line.

## 3. Shared screens and components (build once, Phase 0)
All games use these. A game that re-implements one fails the style gate.

**Top bar** (fixed, same in every game, 3 buttons maximum):
- Back to arcade: top corner. In RTL it is top right, arrow pointing right. Asks "leave game?" only if a run is in progress.
- Pause: opposite top corner. Also triggers when the tab loses focus. Pause sheet: Resume, Restart, Settings, How to play, Back.
- Sound toggle next to pause.

**Start screen** (same layout in every game): themed background, banner above the Start button with the game name, a one-line goal, one control hint with icon, best score, and where relevant a donut chip showing the selected donut (tap to change, includes a Random option). The donut name is always displayed.

**First-run explainer**: shown once, on the first game ever opened. 2 to 3 spotlight callouts on the shared UI (score, pause, back). Tap anywhere to advance, **Skip** always visible. Each later new game shows one skippable callout about its own control only. Stored in localStorage, if storage fails it shows again.

**HUD score**: fixed position, top center, largest text on screen, outlined and shadowed so it reads over any background, pulses on increment, small best-score line under it. Reveal shows time plus a cleared-% bar.

**Game-over screen** (identical structure everywhere): result big, the donut with its name, **Share** (primary), Play again, "Try <next game>", Back to arcade.

**Landscape handling**: only Driving is landscape. Detect with `matchMedia('(orientation: landscape)')`. In a portrait phone, pause and show a full-screen "rotate your phone" overlay with a looping rotation animation. `screen.orientation.lock()` is unreliable (fullscreen Android Chrome only, not iOS Safari), so the prompt is the reliable path. Provide a "play in portrait anyway" fallback. Design the landscape HUD for a 390px short side with safe-area insets.

## 4. No-scroll rule
- Root `100dvh` (with `100vh` fallback), `overflow: hidden`, `overscroll-behavior: none`, `touch-action` set per surface, `env(safe-area-inset-*)` padding.
- Size with `clamp()`, flex and grid, never fixed pixel heights. Art and cards shrink on short screens.
- Sheets and modals have fixed-size bodies and short text.
- Cap text scaling in tokens. Test one large-font case.
- Gate: Playwright visits every screen state (hub, each start, play, pause, game over, each sheet, both languages) at 360x640, 375x667, 390x844, 412x915 and 844x390 (Driving). Fail if `document.documentElement.scrollHeight > innerHeight` or `scrollWidth > innerWidth`.

## 5. Visual direction
**Sunset 90s arcade, professional polish.** Flat-vector, layered silhouettes (sun, hills, palms, grid floor), crisp SVG UI. Pixel font only for numerals and English labels, not pixel-art sprites, because the donuts are photographic.

Starting tokens (the style board refines them, the user approves):
- Sky gradient: deep purple `#1B0B3B` to magenta `#5B1F6E` to coral `#E8456B`; sun `#FFB43C`; grid `#FF4FD8`; ink `#FFF6E6`; panel `#2A1250` at 85%; gold accent `#FFC94A`. Roladin brand color is sampled from the logo file and added as `--brand`.
- Type: Hebrew chunky display font (Rubik 800/900 or Secular One), Press Start 2P for numerals and English. Self-host fonts (OFL). Press Start 2P has no Hebrew.
- Spacing, radii, shadows, one easing curve, three durations. All in one tokens file.
- Components: primary button, secondary button, icon button, banner, HUD score, top bar, sheet, modal, donut chip, DonutName, rotate overlay.
- **Donut sticker treatment**, identical everywhere: thin white outline, soft ground shadow, subtle rim highlight. This is what makes photos and vectors feel like one world.
- Motion: button squash-and-bounce press, score pulse, screen transitions. Matching chiptune-style SFX via WebAudio, started after the first tap (iOS rule).
- Polish budget goes first to hub, start screen, HUD, game over, since every player sees them.

## 6. Donuts and names
`data/donuts.json` is the single source: Hebrew catalogue title (exact spelling from the catalogue, including "TIRAMISU" and "APPLE CINAMON"), provisional English name, a short Hebrew word for the Frankendonut name generator, image path, merge tier, and `imageStatus`.
- Games read donut count N from the data file. Never hardcode a number.
- Images: side views. Pipeline in Phase 0: download, remove background, normalize canvas size, align baseline, export WebP at modest size (cellular users). Keep originals in `assets/raw/` (git-ignored if large).
- The first page image on the catalogue is a hero group shot, not a donut. The two jam donuts may not have clean single images, flag them and ask the user. If they stay missing, N = 9 and everything still works.
- **Overlap rule** for any game that composes slices or tiles (Frankendonut): draw with about 10% bleed past each cell edge and a fixed layer order so silhouettes of narrow and wide donuts blend, no hard cut seams.

## 7. Sharing
One shared `share({ game, donutId, resultValues })` module.
- Builds a canvas card (1080x1350, 4:5) with: the donut sticker, donut name, the big result, game name, hashtag, short URL. Text is baked into the image because some targets drop prefilled text.
- `navigator.share({ files, text, url })` when `navigator.canShare({ files })` is true (HTTPS, user gesture). Fallbacks: download image, copy link, WhatsApp link (`https://wa.me/?text=`).
- URL: `https://tamir-br.github.io/roladin-arcade/?game=<nextGameId>&utm_source=share&utm_campaign=hanukkah2026`. It opens the hub and highlights the suggested game.
- Common text structure in every game: **I did X + result + try another game + link + hashtag.** The "try" game rotates, never the current game. Hashtag comes from `config/brand.config.json`.
- Hebrew templates (English equivalents in `en.json`):
  - Reveal: `חשפתי את {donut} ב-{seconds} שניות! נסו גם את {nextGame} ב-{url} {hashtag}`
  - Frankendonut: `יצרתי את {creationName}! נסו גם את {nextGame} ב-{url} {hashtag}`
  - Flappy: `טסתי {meters} מטרים עם {donut}! נסו גם את {nextGame} ב-{url} {hashtag}`
  - Simon: `זכרתי {n} סופגניות ברצף! נסו גם את {nextGame} ב-{url} {hashtag}`
  - Merge Drop: `מיזגתי עד {topDonut} וצברתי {score} נקודות! נסו גם את {nextGame} ב-{url} {hashtag}`
  - Driving: `נסעתי {km} ק״מ עם {donut}! נסו גם את {nextGame} ב-{url} {hashtag}`
- No per-creation link preview (static site). The attached image carries the content.

## 8. Analytics (deferred, keep in plan)
Plan: **Umami Cloud, Hobby plan** (free tier; limits to verify on umami.is/pricing before launch, a third party listed 100K events per month, one website, 6 months retention). The user creates the account and provides the website ID later.
- Phase 0: build `track(name, props)` in the shared module. It no-ops while `analytics.websiteId` is null or `enabled` is false, never throws, and does not send from dev builds.
- Events: pageview (automatic), `run_start {game, donut}`, `run_end {game, donut, score}`, `share {game, method}`, `lang_set {language}`.
- If the free event cap is a risk, drop `run_end` first.
- Dashboard stays private (no public share link unless the user decides to give Roladin a read-only view).
- Optional later: a script that pulls the Umami API and builds a private HTML report (runs per day per game).
- e2e tests assert that each game fires `run_start` and `run_end` (with `track` mocked).

## 9. Hosting
GitHub Pages under `tamir-br`, repo `roladin-arcade`, URL `https://tamir-br.github.io/roladin-arcade/`. Vite `base: '/roladin-arcade/'`, hash routing, GitHub Actions deploy. HTTPS is required for Web Share. Verify the Roladin image-use permission before public launch.

## 10. Performance budget (starting values, orchestrator may adjust with measured data)
- Hub initial JS 150 KB gzip or less. Each game is a lazy chunk. Heavy engine code loads only when a canvas game opens.
- All images WebP. Total donut image set under 1.5 MB.
- 60 fps target on a mid-range Android phone. No layout thrash during play.

## 11. Difficulty ramps (all capped so games stay beatable)
Flappy: speed up every 5 points, gap narrows, gap height varies more. Reveal: more balls per level, ball speed rises with cleared %. Simon: sequence grows by one per round, tile count fixed. Merge Drop: spawn tier range widens slowly. Driving: speed and traffic density rise with distance. Tune so most Flappy runs end between 10 and 30 seconds, so the share number feels worth posting.
