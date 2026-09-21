# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No dependencies, no build step, no tests, no linter. The README and UI text are in Spanish; keep user-facing strings in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder (e.g. `python3 -m http.server`) and visit `http://localhost:8000`.

## Architecture

Three files, all wired together by DOM ids — `game.js` looks up elements in `index.html` by id at load time (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`), so renaming an id in the HTML breaks the script. The canvas sizes in `index.html` (300×600 board, 120×120 preview) must match `COLS`/`ROWS`/`BLOCK` in `game.js` (10×20 at 30px).

`game.js` is a single script using module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, ...) declared once and reset by `init()`, which is also the restart handler.

- **Pieces/board**: `board` is a `ROWS×COLS` grid of ints; `0` is empty, `1–7` index into both `PIECES` (shape matrices) and `COLORS`. The piece type number *is* the cell value, so the two tables must stay index-aligned (index 0 is `null`).
- **Game loop**: `loop(ts)` is a `requestAnimationFrame` loop that accumulates time into `dropAccum` and gravity-drops when it exceeds `dropInterval`, then calls `draw()`. Pause and game over stop it via `cancelAnimationFrame(animId)`; unpausing restarts it. `init()` cancels any prior frame before starting a new one.
- **Piece lifecycle**: `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current` and triggers `endGame()` if the new piece collides immediately.
- **Input**: a single `keydown` listener handles movement, rotation (with simple wall kicks `[0,-1,1,-2,2]` in `tryRotate`), soft drop, hard drop, and pause (`P`). All collision checks go through `collide(shape, ox, oy)`, which allows `ny < 0` (above the board).
- **Scoring/levels**: `LINE_SCORES × level` per clear; soft drop +1/cell, hard drop +2/cell; level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
