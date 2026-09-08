# Canvas Tap Harness

A slot-style UI drawn **entirely as pixels on a `<canvas>`** — no DOM, no accessibility tree — for testing whether an automation agent can locate and click controls it can only *see*, not read from the page. Every tap is scored **HIT / MISS against known geometry**.

**Live:** https://shrish-browserstack.github.io/canvas-tap-harness/

## Why

Real WebGL/Canvas game UIs expose zero interactive DOM: the whole screen is one `<canvas>` node, and controls and text are pixels. An accessibility-tree-driven agent sees nothing to click. This fixture reproduces that condition with **known control geometry**, so you can measure whether coordinate-based clicking and visual matching actually work — without a real game, credentials, or a device farm.

The telemetry panel on the right *is* normal DOM (it's the observer). The game controls on the canvas are pixels only — that's the system under test.

## Ground truth (the correct answers)

Canvas logical space is **360 × 720**. Controls are drawn at these rects; the harness hit-tests taps against them and reports the result. The **centre** is the ideal tap target.

| id | rect (x,y,w,h) | centre | what it is / case |
|---|---|---|---|
| `menu` | 12,14,40,40 | **32,34** | hamburger, top-left (app-shell) |
| `deposit_plus` | 308,14,40,40 | **328,34** | `+` top-right — look-alike of the bet `+` |
| `book` | 150,296,60,80 | **180,336** | game-art icon; name ≠ look |
| `lines` | 14,628,64,44 | **46,650** | paylines |
| `bet_minus` | 92,626,48,48 | **116,650** | `−` decrease bet (bottom) |
| `spin` | 150,610,72,72 | **186,646** | **circular arrows** — the "refresh" = spin trap |
| `bet_plus` | 234,626,48,48 | **258,650** | `+` increase bet (bottom) — twin of the deposit `+` |
| `info` | 300,628,44,44 | **322,650** | info / paytable |

**Canvas-drawn text (read targets — nowhere in the DOM):**
`BALANCE 1,250.00 €` (top centre) · `BET 2.50 €` (bottom-left) · `WIN 0.00 €` (bottom-right).

## Validation cases & expected answers

### L0 — locate + click
| prompt | correct target |
|---|---|
| "click the spin button" | `spin` (186,646) |
| "click the refresh icon" | `spin` — trap: it is circular arrows; there is **no** browser-refresh here |
| "open the menu" | `menu` (32,34) |
| "increase the bet" | `bet_plus` (258,650) |
| "decrease the bet" | `bet_minus` (116,650) |
| "open info / paytable" | `info` (322,650) |

### L2 — look-alike disambiguation
| prompt | correct behaviour |
|---|---|
| "click the +" | **ambiguous** — two matches (`deposit_plus` 328,34 · `bet_plus` 258,650). Correct = flag the ambiguity, don't guess. |
| "click the bottom +" / "the bet +" | `bet_plus` (258,650) |
| "click the top +" / "the deposit +" | `deposit_plus` (328,34) |

### Read (canvas text)
| prompt | correct answer |
|---|---|
| "read the balance" | `1,250.00 €` |
| "read the bet" | `2.50 €` |
| "read the win" | `0.00 €` |

### L3 — time-aware assertion
Click **Trigger free games** (or tap `spin` three times). The banner goes **absent → changing (~2.6 s) → settled**. Settled reads `10 FREE GAMES` / `FREE SPINS LEFT 10`. A correct assertion detects *present*, then *settled*.

## How to drive it

- **By hand / your browser** — click; the panel shows live hit-rate and a per-tap log.
- **Browser automation / a real device session** — every tap logs to the console with the `[canvas-poc]` prefix; filter on it to read results.
- **Programmatically** — the page exposes `window.canvasPoc` (the answer key is built in):

| call | returns |
|---|---|
| `canvasPoc.controls` | every control `{id,label,rect,center,note}` (ground truth) |
| `canvasPoc.tap(id)` | tap a control's centre; returns the tap record |
| `canvasPoc.tapAt(x,y)` | tap logical (360×720) coords |
| `canvasPoc.taps()` | all tap records |
| `canvasPoc.hitRate()` | hits / taps (or `null`) |
| `canvasPoc.triggerFreeGames()` · `canvasPoc.freeGamesState()` | fire / read `idle\|changing\|settled` |
| `canvasPoc.reset()` | clear the log |

### Console log format
```
[canvas-poc] TAP (188,644) → HIT spin  centreΔ=(2,-2)  rect=[150,610,72,72]
[canvas-poc] TAP (300,300) → MISS  nearest=book dist=41px
```

## Pass criteria

- A tap is a **hit** when it lands inside the intended control's rect; `centreΔ` gives the pixel error.
- For an ambiguous prompt, "correct" is **flagging the ambiguity**, not any single tap.
- The harness reports HIT/MISS in its own logical space, so results are independent of the render size the agent actually clicked at.

The **Model-view resolution** panel additionally shows the frame as a planner receives it today (1280×720 letterbox, SPIN glyph ~43px) versus a portrait box (720×1280, ~77px) — a static illustration of why small glyphs need the larger box.

No build, no dependencies (Google Fonts only, degrades to system fonts). Everything is `index.html`.
