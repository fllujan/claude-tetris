# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris. No build process, no package manager, no dependencies — three files total (`index.html`, `style.css`, `game.js`). Everything runs directly in the browser via Canvas 2D.

## Running

There is no build/lint/test tooling in this repo. To run the game:

```bash
open index.html                 # macOS, or just open the file in a browser
python3 -m http.server 8000     # or serve locally, then visit http://localhost:8000
```

Changes to `game.js`/`style.css`/`index.html` take effect on browser refresh — no compilation step.

## Architecture

All game logic lives in `game.js` (single file, no modules). Key pieces:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or an integer `1–7` indexing into `COLORS`/`PIECES` for the piece type that occupies it.
- **Pieces**: `PIECES` defines the 7 tetrominoes as square matrices. `rotateCW` rotates a shape via transpose + row-reverse. `tryRotate` applies `rotateCW` then attempts wall kicks at offsets `[0, -1, 1, -2, 2]`, keeping the first that doesn't collide.
- **Collision**: `collide(shape, ox, oy)` is the single source of truth for validity — used by movement, rotation, ghost-piece projection, and spawn (game-over check).
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulating elapsed time in `dropAccum` and advancing the piece one row once `dropAccum >= dropInterval`. Locking, line clearing, and spawning of the next piece all happen through `lockPiece()`.
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; `level` increments every 10 lines cleared; `dropInterval = max(100, 1000 - (level-1)*90)` controls fall speed.
- **Rendering**: `draw()` clears and redraws the grid, locked board, ghost piece (`ghostY()` projects the current piece straight down, drawn at `globalAlpha = 0.2`), and the active piece — every frame, no diffing.
- **State**: module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, etc.) hold all mutable game state; `init()` resets them and is also wired to the restart button.

Tunable constants are at the top of `game.js` (`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`). If `COLS`/`ROWS`/`BLOCK` change, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).
