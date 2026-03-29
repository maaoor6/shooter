# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a collection of self-contained browser games — each game is a single HTML file with no external dependencies, no build step, and no package manager. Open any `.html` file directly in a browser to run it.

## Running the Games

```bash
open shooter.html
open tictactoe.html
```

## Architecture

All games follow the same pattern:

- **Single HTML file** — inline CSS + inline JavaScript, zero external dependencies
- **HTML5 Canvas** for game rendering (shooter), or DOM elements (tictactoe)
- **Vanilla ES6** — classes, arrow functions, `const`/`let`, no frameworks

### shooter.html

Top-down 2D retro shooter. Key architecture:

- **Logical resolution**: 320×240 game coordinates, rendered to a 640×480 canvas via `ctx.scale(2, 2)` in `render()`. All game logic uses W=320, H=240.
- **Mouse coordinates** must be divided by both the CSS scale factor (`input._scale`) and by 2 (for the canvas→game coordinate conversion).
- **Game loop**: fixed-timestep accumulator (`STEP = 1000/60`), decoupled from rendering via `requestAnimationFrame`.
- **Render layers**: game world (inside `ctx.scale(2,2)`) → CRT overlay → HUD → state screens. HUD and overlays use `CANVAS_W`/`CANVAS_H` (640×480) coordinates directly.
- **Classes**: `InputHandler`, `Player`, `GruntEnemy` / `TankEnemy` / `FastEnemy` / `ShooterEnemy` (all extend `Enemy`), `Bullet`, `Particle`, `LevelManager`, `RetroAudio`, `Game`.
- **`game`** is a global variable referenced by `LevelManager` for audio calls.
- **Audio**: procedural Web Audio API beeps, created on first user interaction to satisfy browser autoplay policy.
- **Persistence**: `localStorage` key `retro_assault_hi` for high score only.
