# GAMES

Common contract. Every game module exports:
```ts
interface Game {
  id: GameId;
  init(root: HTMLElement, ctx: GameContext): Promise<void>; // ctx: donuts, i18n, tokens, track, share, storage
  start(opts: { donutId?: string }): void;
  pause(): void;
  resume(): void;
  destroy(): void;                 // free canvas, listeners, audio
  // emits: 'run_start', 'score', 'run_end' with a Result object
}
```
Game logic in `src/games/<id>/logic/` is pure TypeScript with a seeded RNG, fully unit tested. Rendering is a thin layer. Every game uses the shared top bar, start screen, HUD score and game-over screen.

All games are portrait except Driving. Orientation reasons are in the right column.

| id | Hebrew name | Orientation | Why |
|---|---|---|---|
| reveal | סוגר שטחים | portrait | Near-square play field, HUD above, thumbs below |
| frankendonut | סופגנייה מעורבבת | portrait | Three stacked strips, swipe each |
| flappy | סופגנייה מעופפת | portrait | Tall field shows more upcoming gaps, one-thumb tap |
| simon | סיימון סופגניות | portrait | 2x2 grid with large targets |
| mergedrop | סופגניות מהשמיים | portrait | Tall container, more stacking room, drop-line fairness |
| driving | משלוח סופגניות | landscape | Pseudo-3D road needs a wide horizon |

Donut name rule: whenever a whole donut is shown (picker, start, game over, billboards, merge tiers, Simon recap), show its catalogue name through `DonutName`.

## reveal: סוגר שטחים
- Concept: a glaze layer hides a donut. Clear 90% of the area to reveal it. Score is time (lower is better).
- Control: Jezzball style. A swipe inside the field sends a cut: swipe axis sets the cut orientation, start point sets the position. Bouncing balls roam. A cut finished without a ball touching its growing wall locks that area and clears it. If a ball hits a growing wall, the wall is destroyed and a life is lost (3 lives). Fallback if testers fail the swipe control: tap to cut plus a large orientation toggle button. The UX critic and a quick test with real people decide.
- Donut choice: pick or Random on the start screen. The hidden donut is that donut. Best time stored per donut.
- Ramp: each new level adds a ball. Within a level, ball speed rises as the cleared percentage grows, so the last 10% is hardest. Cap speed.
- HUD: time (big), cleared-% bar, lives.
- Win: 90% cleared, donut revealed with its name. Share: `חשפתי את {donut} ב-{seconds} שניות!`
- Hard game: Opus dev (geometry and collision).

## frankendonut: סופגנייה מעורבבת
- Concept: a donut made of 3 horizontal strips (top, middle, base). Swipe a strip left or right to swap it for the same strip from another donut. No fail state, no score.
- Starts as one whole random donut (all strips from the same image). Shuffle randomizes all 3 strips. Reset returns to a whole random donut. Share the creation.
- Name: generated Hebrew name from the three `short` words in `data/donuts.json`, shown with the three source donut names in small text.
- Apply the overlap rule (10% bleed, fixed layer order) so no seams appear.
- Combinations: N cubed.
- A "run" for tracking = first interaction after entering the game.
- Standard dev (Sonnet), DOM or small canvas, no Phaser.

## flappy: סופגנייה מעופפת
- Concept: tap to flap, the chosen donut flies through gaps between Hanukkah candles (top and bottom). Score is meters.
- Donut choice: pick or Random on the start screen. Best distance stored per donut.
- Ramp: speed up every 5 points, gap narrows slightly, gap height varies more. Cap speed and minimum gap.
- Share: `טסתי {meters} מטרים עם {donut}!`
- Standard dev (Sonnet).

## simon: סיימון סופגניות
- Concept: donut tiles light up in a sequence, repeat it. Sequence grows by exactly one per round. **Tile count is fixed**, default 4 (2x2), drawn at random from the full donut set each run. Keep tile count as one config value.
- Each tile has its own tone. Each tile shows its donut name in small text.
- No donut choice. Score is rounds reached. Game-over recap shows the donuts in the sequence with names.
- Share: `זכרתי {n} סופגניות ברצף!`
- Standard dev (Sonnet), DOM based.

## mergedrop: סופגניות מהשמיים
- Concept: donuts fall from the sky into a container. Two of the same type touching merge into the next tier. Container overflow ends the run. The tiers are the donut set, ordered by `mergeTier` in `data/donuts.json`. Tier count = N.
- No donut choice. Show a small tier ladder with names. Merge pop-ups show the new donut name.
- Matter physics. Score from merges. Share: `מיזגתי עד {topDonut} וצברתי {score} נקודות!`
- Hard game: Opus dev (physics and tuning).

## driving: משלוח סופגניות
- Concept: OutRun-style pseudo-3D road, landscape. The chosen donut rides on the car roof as cargo. Auto-accelerate, hold the left or right half of the screen to steer, dodge traffic and oil spills. Checkpoints add time. Score is kilometers.
- Billboards along the road show the Roladin logo next to a large donut, cycling through all donuts, with names.
- Donut choice: pick or Random on the start screen.
- 90s sunset look (gradient sky, palms, synth music). Avoid any Sega or OutRun names, cars or Ferrari look.
- Rotate overlay in portrait, plus "play in portrait anyway" fallback.
- Share: `נסעתי {km} ק״מ עם {donut}!`
- Hard game, last in build order, Opus dev. Cut it first if the schedule is tight.

## Build order
Phase 1 parallel: reveal, frankendonut, flappy, simon, mergedrop. Driving starts only when the others pass their gates.
