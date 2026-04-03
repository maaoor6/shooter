# RETRO ASSAULT

A fast-paced retro shooter game that runs entirely in the browser — no install, no build step, no dependencies.

## Play

Open `index.html` in any modern browser:

```bash
open index.html
```

Or play on GitHub Pages.

## Tech Stack

- **Vanilla JavaScript (ES6)** — zero frameworks
- **HTML5 Canvas** — all rendering via 2D context
- **Web Audio API** — procedural SFX and 4-channel chiptune BGM
- **Firebase Realtime Database** — global leaderboard sync (ES module script)
- **localStorage** — persists high score, mute setting, and local leaderboard

## Features

- 15 levels with 6 unique boss fights
- 8 weapons: Pistol, Shotgun, SMG, Rocket, Laser, Plasma, Freeze, Flame — **infinite ammo**
- Global leaderboard synced via Firebase (top 10 worldwide)
- Local leaderboard stored in browser (top 10 per device)
- Screen shake, hit flash, particle effects, score pop-ups
- Boss slow-mo death cinematic (0.15× speed for 2 seconds)
- 6 achievements with toast notifications (persisted)
- Chiptune background music with mute toggle (🔊 button, top-right)
- "NEW RECORD!" banner on high score
- Enemy repulsion physics — enemies no longer stick to the player
- Full mobile support: dual virtual joysticks, aim assist, full-screen display
- Retina/HiDPI canvas — crisp on all screens

## Controls

| Input | Action |
|---|---|
| WASD / Arrow keys | Move |
| Mouse | Aim |
| Left click / hold | Fire |
| 1–8 | Switch weapon |
| Mouse wheel | Cycle weapons |
| R | Restart (Game Over screen) |
| L | View leaderboard (Start screen) |
| Left joystick (mobile) | Move |
| Right joystick (mobile) | Aim + fire |

## Weapon System

| # | Weapon | Type |
|---|---|---|
| 1 | Pistol | Standard, infinite ammo |
| 2 | Shotgun | 5-pellet spread |
| 3 | SMG | Fast fire rate |
| 4 | Rocket | Explosive splash damage |
| 5 | Laser | Piercing beam |
| 6 | Plasma | Bounces off walls |
| 7 | Freeze | Slows enemies |
| 8 | Flame | Short-range, 3-shot spread |

## Global Leaderboard (Firebase)

Scores are saved to Firebase Realtime Database when a player qualifies for the top 10.
The leaderboard screen shows two columns: **LOCAL** (this device) and **GLOBAL** (worldwide).

Firebase config keys for web apps are **not secrets** — they identify the project publicly.
Security is enforced via Firebase Security Rules (read/write rules on the database).

## Version Control

Git & GitHub
