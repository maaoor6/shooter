# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A self-contained browser shooter game — a single HTML file (`index.html`) with no build step, no dependencies, and no package manager. Open directly in a browser.

```bash
open index.html
```

## Coordinate Systems — Critical to Understand

Three coordinate spaces coexist; mixing them up causes bugs:

| Space | Range | Used for |
|---|---|---|
| **Game** | `W=320 × H=240` | All game logic: positions, collision, AI, spawning |
| **Canvas** | `CW=640 × CH=480` | HUD, overlays, virtual controls drawing |
| **CSS** | `window.innerWidth × innerHeight` | Canvas CSS size, touch input |

**Conversions:**
- CSS → Canvas logical: `css / _cssScale`
- Canvas → Game: divide by 2 (because `render()` does `ctx.scale(2,2)` inside `ctx.save()`)
- Touch CSS → Game: `(touch.clientX) / _scale / 2`
- Canvas logical Y can exceed `CH` in portrait mode — this is the controls zone

**Portrait mode** (`_isPortrait`): game fills full screen width; `_vch = window.innerHeight / _cssScale` gives the total logical canvas height. Controls are drawn at `y > CH` in this extended space.

## Render Pipeline (in order each frame)

1. `ctx.setTransform(dpr * _cssScale, ...)` — normalize to CW/CH logical space
2. `ctx.save(); ctx.scale(2,2)` — enter W/H game space
3. Draw: background → aim line → bullets → particles → pickups → enemies → freeze overlays → player → crosshair
4. `ctx.restore()` — back to CW/CH
5. `drawCRT()` → `drawHUD()` — overlays in CW/CH coords
6. `drawVirtualControls()` — uses full `_vch` height (extends into portrait controls zone)
7. State screens (start / level-complete / game-over / victory)

## Key Classes

| Class | Notable fields |
|---|---|
| `InputHandler` | `joystick` (left, movement), `rightJoy` (right, aim+fire), `_scale`, `_vch`, `_portrait`. `isFiring` getter = right joystick deflected > 15%. |
| `Player` | `ammo{}`, `weapon`, `angle`. On mobile: `angle` comes from `rightJoy.angle`; `input.mouse` is updated to project 55 units in aim direction so crosshair tracks correctly. |
| `Enemy` (base) | `frozen`, `slowFactor` — used by freeze weapon. `isBoss=false` by default. |
| `Bullet` | `bouncing` (plasma, uses `vx/vy`), `freezing`, `flame` (range-limited). Non-bouncing bullets use `angle + speed`. |
| `LevelManager` | `spawnQueue` entries: `{type, delay}`. Boss levels set `isBossLevel=true`. |
| `Game` | Owns `_dpr`, `_cssScale`, `_isPortrait`, `_vch`. Global `game` variable is referenced by `LevelManager` for audio. |

## Weapons (8 total, keys 1–8)

`pistol` (∞), `shotgun`, `smg`, `rocket` (explosive+splash), `laser` (piercing), `plasma` (bounces off walls, uses vx/vy), `freeze` (slows enemies 30% for 2s, not on bosses), `flame` (range-limited, 3-shot spread).

## Level Structure (15 levels)

Boss levels: **3** (Plague Witch → `weapon_freeze`), **6** (Iron Sentinel → `weapon_plasma`), **8** (Tank Commander → `weapon_rocket`), **11** (Storm Hydra → `weapon_shotgun`), **13** (Necro Titan → `weapon_rocket`), **15** (Void Overlord → `weapon_laser`).

Bonus rounds: **4** (25s), **12** (20s). Boss queue entries use `delay:2.5` after mini-wave.

All boss `update()` signatures: `(dt, player, bullets, enemies)`.

## Mobile Touch Architecture

- **`JOY_R = 0.075`** — joystick max-throw as fraction of `CW`; used in both input math and draw rendering to keep them in sync.
- **Left half** of screen → movement joystick → translates to WASD keys.
- **Right half** → aim joystick → sets `rightJoy.angle` + fires when magnitude > 0.15.
- **Weapon buttons** at bottom: `_checkWeaponButtons(canvasX, canvasY)` — Y position is `CH + ctrlH*0.72` in portrait, `CH-36` in landscape.
- `drawVirtualControls()` reads `g._isPortrait` and `g._vch` to position joysticks and weapon strip dynamically.

## Scaling / HiDPI

`_setupScale()` sets `canvas.width = window.innerWidth * devicePixelRatio`. The `render()` applies `ctx.setTransform(dpr * cssScale, ...)` at the top of every frame so all downstream drawing stays in CW/CH logical space regardless of DPR.

## Persistence

`localStorage` key `retro_assault_hi` — high score only.
