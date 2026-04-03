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

| Key | Contents |
|---|---|
| `retro_assault_hi` | High score (integer) |
| `retro_assault_lb` | Local top-10 leaderboard JSON array |
| `retro_achievements` | JSON array of unlocked achievement IDs |
| `retro_mute` | Mute state boolean |

## Firebase Global Leaderboard

Architecture: **two separate script tags** in `index.html`.

1. **Main `<script>`** (classic JS, no `type`) — all game code, runs synchronously.
2. **`<script type="module">`** (ES module, at end of `<body>`) — Firebase SDK + two window globals:
   - `window._fbSaveScore(name, score)` — sanitizes input, pushes to `/leaderboard` in Firebase
   - `window._fbFetchTop10()` — returns Promise resolving to sorted top-10 array

**Game integration:**
- `_initGame()` calls `window._fbFetchTop10()` → stores result in `this._globalLB`
- `_saveToLeaderboard(name)` calls `window._fbSaveScore()` after local save
- `drawLeaderboard(ctx, g)` renders two columns: LOCAL (`g._leaderboard`) and GLOBAL (`g._globalLB`)

**Security:** Firebase web config is public by design. Access control is via Firebase Security Rules, not API key secrecy. Input is sanitized (`replace(/[<>"'&]/g, '')`) before storage. Canvas rendering uses `ctx.fillText()` — never `innerHTML` — so XSS via leaderboard names is impossible.

## InputHandler — Mouse Wheel

`canvas.addEventListener('wheel', ...)` with `passive:false` (to allow `preventDefault`).
Cycles `pendingWeaponSwitch` through `Object.keys(WEAPONS)` with wrap-around.
`pendingWeaponSwitch` is consumed in `Player.update()`.

## Enemy Repulsion Physics

`Enemy.update()` runs a separation check after movement:
- Computes distance to player
- If `dist < (enemy.w + 10) * 0.5` → applies outward force proportional to overlap
- Skipped for bosses (`isBoss=true`) to preserve boss fight feel
- Prevents the "sticky enemy" bug where FastEnemies lock onto the player hitbox
