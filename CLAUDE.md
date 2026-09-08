# CLAUDE.md — Canvas Tap Harness

Guidance for a Claude Code session **validating an automation agent** against this fixture.
This file is the **checker's reference and answer key** — do **not** feed it to the agent under
test. That agent must locate every control from the rendered canvas alone.

## What this repo is

One static page (`index.html`) that draws a slot-style UI **entirely on a `<canvas>`** — no DOM
controls, no accessibility nodes. It tests coordinate-based clicking + visual matching against
**known geometry**. Live: https://shrish-browserstack.github.io/canvas-tap-harness/ — no build,
no dependencies (Google Fonts only). The DOM telemetry panel is the observer; the canvas controls
are the system under test.

## How to run a validation pass

1. Point the agent under test at the live URL (or serve `index.html` locally).
2. Give it one prompt from `README.md` → *Validation cases*.
3. Read the result, most rigorous first:
   - `window.canvasPoc.taps()` — structured records (`hit`, `target`, `dx`, `dy`).
   - console lines prefixed `[canvas-poc]` (`HIT <id> centreΔ=…` / `MISS nearest=… dist=…`).
   - the on-page panel (hit-rate + log).
4. Compare against the answer key below and record hit / miss + pixel error.

## Answer key (ground truth · 360 × 720 logical space)

`menu` 32,34 · `deposit_plus` 328,34 · `book` 180,336 · `lines` 46,650 · `bet_minus` 116,650 ·
**`spin` 186,646** · `bet_plus` 258,650 · `info` 322,650.
Read targets (canvas text): balance `1,250.00 €` · bet `2.50 €` · win `0.00 €`.

## Pass / fail

- **Click:** pass if the tap HITs the intended `id`. `spin` must be reached from *both* "spin"
  and "refresh icon" (lexicon trap — circular arrows; no browser-refresh exists).
- **Disambiguation:** "click the +" has two valid matches (`deposit_plus`, `bet_plus`) → pass = the
  agent **flags ambiguity / asks**; fail = it guesses one. "bottom +" → `bet_plus`, "top +" →
  `deposit_plus`.
- **Read:** pass if the value equals the read target.
- **L3:** trigger free games → pass if `canvasPoc.freeGamesState()` reaches `settled` and the
  banner reads "10 FREE GAMES".

## Guardrails

- Controls are pixels: a valid solution finds them from a **screenshot**, not the DOM/a11y tree.
  If the agent reads the DOM to locate a control, that is a fail for the perception test (the panel
  is DOM, but the controls under test are not).
- Report each result as: prompt → agent's tap coords → HIT/MISS + `centreΔ`; then hit-rate over the
  full case set.
- This is a synthetic fixture. Keep it free of real product names, tickets, or credentials.
