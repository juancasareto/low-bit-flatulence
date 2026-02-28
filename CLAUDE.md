# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Fuego Gástrico** (Low-bit Flatulence) — a single-file browser game. A butt cannon fires a poop projectile at an angel toilet floating in the air. The player charges power by holding SPACE or the FUEGO button and adjusts the angle.

The entire game lives in one file: `index.html`.

## Running the game

No build step. Open `index.html` directly in a browser, or serve it with any static server:

```sh
npx serve -p 5500 .   # then open http://localhost:5500
# or
python3 -m http.server 5500
```

The `.claude/launch.json` is configured for `npx serve` on port 5500 (use `preview_start` with name `fuego-gastrico`).

## Architecture

`index.html` is self-contained: HTML + CSS + JS + one inline base64 PNG (the toilet sprite, ~line 178). There are no external dependencies, build tools, or separate files.

### Game loop

Two animation paths share the same `draw()` function:

- **`idleDraw()`** — runs while `animating = false` (player is aiming). Updates the power bar while charging and redraws the scene each frame.
- **`animate(ts)`** — runs while `animating = true` (projectile in flight). Calls `update(dt)` then `draw()` each frame.

`animating` is a module-scoped `let` (not on `window`). All game state lives in the `state` object (also module-scoped `let`), which means `eval`-based debugging cannot access them directly — use wrapper functions via `window.functionName()` instead. Note: `preview_eval` runs in an isolated context; `let` variables are invisible there but `function` declarations ARE accessible as `window.fnName()`.

### State machine

`state.fase` drives the game flow:
- `'apuntando'` → player can charge and fire
- `'disparando'` → projectile in flight
- `'resultado'` → showing outcome before next shot

### Physics

`index.html` lines ~278–315. Key constants:
- `CV_W = 1400`, `CV_H = 580`, `GROUND_Y = 460`, `CANNON_X = 60`
- `G = 9.8` (gravity)
- `MAX_V0 = sqrt(alcance * G)` — velocity for full range at 45°
- Actual `v0 = power * MAX_V0 * 1.5` (1.5× boost ensures all targets are reachable at any angle)
- `alcance` (total range) is randomized: 10 000–50 000 m
- `distObj` (target distance) is randomized: 5 000 – `alcance` m

### Projectile trail

`state.proj.trail` stores the last 28 positions. `drawTrail()` renders them; this function **must** exist — if removed, `draw()` throws a `ReferenceError` that silently kills the animation loop.

### Audio

Procedural via Web Audio API (no assets). `getAC()` lazily creates the `AudioContext`. Functions: `soundFire`, `soundHit`, `soundMiss`, `soundVictory`, `soundDefeat`.

### Mobile

Viewport meta and a `@media (max-height: 500px)` query handle landscape phones. The canvas uses `max-height: 55vh` on mobile so all UI elements fit without scrolling. Touch events (`touchstart`/`touchend`/`touchcancel`) are attached to the fire button alongside the mouse events.

## Skills

A `/develop-web-game` skill is available in `.claude/skills/develop-web-game/`. It provides a Playwright-based testing loop with screenshot capture, `window.render_game_to_text` state inspection, and `window.advanceTime(ms)` for deterministic frame stepping. The Playwright client script lives at `~/.codex/skills/develop-web-game/scripts/web_game_playwright_client.js`.
