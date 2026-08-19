# Build Spec — 12 Balls Weighing Puzzle Sandbox

## Purpose

A single-page web sandbox to **play with** the classic 12-balls balance puzzle. It is a manipulation/experimentation tool, **not a solver**. It never reveals the answer, never validates the player's deduction, and has no win/lose state.

The puzzle itself (for context only — do NOT encode a solution): there are 12 identical-looking balls; 11 weigh the same and 1 is different (either heavier or lighter). Using a symmetric balance scale, the player has 3 weighings to work out which ball is the odd one and whether it's heavy or light. This app just lets them run weighings and take notes.

## Non-goals / hard constraints

- **Do not reveal or hint** the odd ball, its direction (heavy/light), or any solution — not in the UI, not in user-visible code comments, not in `console.log`. The hidden state must never be rendered to the DOM.
- **No solver, no answer-checking, no success/failure screen.** After 3 weighings the player just starts over.
- **Single self-contained `index.html`** — inline CSS + JS, no frameworks, no build step, no backend.
- Target: **GitHub Pages** (static). File must be named `index.html` so it serves at the site root.
- Desktop is the priority (HTML5 drag-and-drop is mouse-based); mobile-friendliness is a bonus, not required.

## Balls

- 12 balls, labeled **A–L** (twelve letters, A through L).
- Weights are integers: every normal ball = **2**. The odd ball = **1** (lighter) or **3** (heavier).
- On each new game, randomize both: which ball is odd (`oddIndex`, 0–11) and its weight (`oddWeight`, 1 or 3).
- `weightOf(letter)`: `index = letter.charCodeAt(0) - 65; return index === oddIndex ? oddWeight : 2`.
- Hidden state = just `oddIndex` and `oddWeight`. Keep both out of the DOM.

## Layout

Full-viewport, two regions side by side:

```
+--------------------------------------------------+-----------------+
|  HEADER: title | weighings x/3 | Measure | Reset |  MOVE LOG (top) |
+--------------------------------------------------+                 |
|  SCALE:  [ left pan ]   ( symbol )   [ right pan ]|                 |
+--------------------------------------------------+-----------------+
|  NEUTRAL ZONE: 16-slot grid                      |  NOTES textarea |
|                                                  |  (fills to      |
|                                                  |   bottom)       |
+--------------------------------------------------+-----------------+
```

- **Left/main column:** header (title, weighing counter `x/3`, Measure button, Start over button), then the scale (two pans with a center symbol), then the neutral-zone grid.
- **Right sidebar (full height):** move log at the top, notes textarea filling the remaining height down to the bottom.

## Neutral zone (the 16-slot grid)

- Fixed grid of **16 cells**; each cell holds **0 or 1 ball**.
- At game start, balls A–L fill cells **0–11 in order**; cells 12–15 are empty.
- **Positional, not a list.** Gaps persist — nothing reflows, auto-sorts, or closes gaps. This is intentional: the player leaves empty cells between groups to stay organized (e.g. "known equal" vs "suspects").
- **Drop on an occupied cell → swap:** the two balls trade places (the occupant moves to the dragged ball's previous location). Drop on an empty cell → just place it.
- Positions are frozen during play; a ball only moves when the player drags it.
- On reset, repopulate A–L into cells 0–11.

## Pans (left and right)

- Two simple **ordered** drop zones (not slotted). Dropping a ball appends it to that pan.
- Any number of balls may sit on a pan, but see the Measure gate below.
- Render as two areas side by side (can be as simple as two lines/rows) with a large symbol between them.

## Drag and drop

- Plain HTML5 DnD. Drop targets: the 16 grid cells + the 2 pans.
- A ball can move freely between any zones (grid↔pan, pan↔pan, grid↔grid).
- Grid-cell occupied → swap (as above). Pan drop → append.
- Helper model: `locate(letter)` finds current zone+index; `removeFrom(loc)`; `placeAt(loc, letter)`.

## Measuring

- **Measure is enabled only when** `leftCount === rightCount && leftCount > 0`. Otherwise disable it and show a hint like: `Pans need equal, non-empty counts`.
- On Measure: sum each pan's weights and compare. Display `left [symbol] right` in the center:
  - `leftSum > rightSum` → `>`
  - `leftSum < rightSum` → `<`
  - equal → `=`
- Increment the weighing counter. After the **3rd** weighing, **lock** the Measure button (no more weighings until reset).
- No success/failure screen.

> Note: equal counts guarantee `=` means "the odd ball is not currently on the scale," and any tilt means the odd ball is on the heavier/lighter side. This is why the equal-count gate matters — don't allow lopsided weighings.

## Move log (top of right sidebar)

- One line per weighing, exact format: pan contents space-separated with the symbol between, e.g.
  - `A B C > D E F`
  - `A B C = D E F`
- **Newest first.** Clears on reset.

## Notes (right sidebar, below the log, fills to bottom)

- A `<textarea>` persisted to **localStorage** (suggested key `12balls-notes`).
- Load its value on page load; save on every input.
- **Does not clear on reset or new game** — only when the player edits it. Persists across reloads.
- Per-visitor / per-browser (not shared between people).

## Reset / Start over

Re-randomize `oddIndex` and `oddWeight`; restore grid to A–L in cells 0–11; empty both pans; clear the log; set counter to 0; unlock Measure; reset the center symbol to a neutral state. **Do not touch the notes.**

## Suggested state model

```js
oddIndex        // 0–11, hidden
oddWeight       // 1 or 3, hidden (normal = 2)
grid            // array length 16 of letter | null
leftPan         // array of letters, ordered
rightPan        // array of letters, ordered
measurements    // 0–3
locked          // boolean, true after 3 weighings
log             // array of strings, newest first
// notes lives in localStorage, not in game state
```

## Build order (suggested)

1. Static layout: header, scale, 16-cell grid seeded A–L, right sidebar with log + notes.
2. Drag-and-drop across the 16 cells and 2 pans (with swap-on-occupied for cells).
3. Measure logic: equal-count gate, sum + compare, center symbol, counter, lock after 3.
4. Move log rendering + reset behavior.
5. Notes textarea wired to localStorage.
