# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A self-contained browser shooter game — a single HTML file with no external dependencies, no build step, and no package manager. Open `index.html` directly in a browser to run it.

## Running the Game

```bash
open index.html
```

## Architecture

**Single HTML file** — inline CSS + inline JavaScript, zero external dependencies. Vanilla ES6 classes, HTML5 Canvas rendering.

### Coordinate Systems

Two coordinate spaces coexist — this is the most important thing to understand:

- **Game space**: `W=320, H=240` — all game logic (positions, collision, enemy AI) uses these coordinates.
- **Canvas space**: `CW=640, CH=480` — the actual canvas resolution. `render()` calls `ctx.scale(2,2)` before drawing game world, then `ctx.restore()` for HUD/overlays.
- **CSS space**: the canvas is scaled further by `input._scale` to fit the browser window.
- Mouse and touch coordinates must be divided by `_scale` (CSS→canvas) and then by `2` (canvas→game) to get game coordinates.

### Game Loop

Fixed-timestep accumulator (`STEP = 1000/60`), decoupled from rendering via `requestAnimationFrame`. `update()` receives `dt` in seconds.

### Render Layers (in order)

1. `ctx.scale(2,2)` → background, bullets, particles, pickups, enemies, player, crosshair
2. `ctx.restore()` → CRT scanline overlay
3. HUD (`drawHUD`) — drawn in CW/CH (640×480) coords
4. Virtual controls (`drawVirtualControls`) — drawn in CW/CH coords
5. State overlays (start screen, level complete, game over, victory)

### Classes

| Class | Purpose |
|---|---|
| `InputHandler` | Keyboard, mouse, and touch (joystick + right-side aim/fire). `isFiring` getter for held touch. `pendingWeaponSwitch` for touch weapon buttons. |
| `Player` | Movement, shooting, ammo tracking, weapon switching (keys 1–8). |
| `Enemy` (base) | Chase-player AI, `frozen`/`slowFactor` for freeze weapon effect. |
| `GruntEnemy`, `TankEnemy`, `FastEnemy`, `ShooterEnemy` | Extend `Enemy`. |
| `Boss1`–`Boss6` | 3-phase bosses; update signature is `(dt, player, bullets, enemies)`. |
| `Bullet` | Supports `explosive`, `piercing`, `bouncing` (plasma), `freezing`, `flame` (range-limited). Bouncing bullets use `vx`/`vy` instead of angle. |
| `LevelManager` | Spawn queue with per-entry delays. Boss levels: 3, 6, 8, 11, 13, 15. Bonus rounds: 4, 12. |
| `RetroAudio` | Procedural Web Audio API; initialized on first user interaction. |
| `Game` | Owns all state arrays (`bullets`, `enemies`, `particles`, `pickups`). Global `game` variable used by `LevelManager` for audio calls. |

### Weapons (8 total)

`pistol` (infinite), `shotgun`, `smg`, `rocket` (explosive), `laser` (piercing), `plasma` (bouncing, high dmg), `freeze` (slows enemies to 30% for 2s), `flame` (short range, 3-shot spread). Keys 1–8.

### Mobile Touch Controls

- Left 1/3 of screen → virtual joystick (translates to WASD keys)
- Right 2/3 → aim and auto-fire (held touch = continuous fire)
- Bottom strip → 8 weapon buttons (`_checkWeaponButtons` in `InputHandler`)
- Joystick and weapon buttons drawn by `drawVirtualControls(ctx, game)`

### Persistence

`localStorage` key `retro_assault_hi` — high score only.
