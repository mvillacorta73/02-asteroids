# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Asteroids clone in plain HTML5 Canvas. No dependencies, bundler, package.json, tests, or linter. The UI and comments are in Spanish; keep that convention. The README mentions power-ups and a "shooting star" asteroid, but neither exists in the code yet.

## Running

Open `index.html` directly in a browser, or serve it with `npx serve .` (http://localhost:3000). Reload the page to see changes.

## Architecture

All game logic lives in `game.js`, loaded by `index.html` as a classic script (`'use strict'`, no modules). The file is organized in banner-commented sections, in this order:

- **Input**: `keys` holds held state by `e.code`. `justPressed` holds edge-triggered presses, and `pressed(code)` consumes one (it resets the flag on read). Use `pressed()` for one-shot actions like shooting and restarting, and `keys[...]` for held controls.
- **Entities** (`Bullet`, `Asteroid`, `Ship`, `Particle`): each has `update(dt)`, `draw()`, and a `dead` flag. Dead entities are removed by `filter` in the main `update`. Entities draw to the global `ctx`.
- **Game state**: module-level globals (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`) and a `state` string: `'playing'` | `'dead'` (respawn countdown via `deadTimer`) | `'gameover'`. `update(dt)` branches on `state` first, and `draw()` renders everything and overlays on gameover.
- **Loop**: `requestAnimationFrame`, with `dt` in seconds clamped to 0.05.

Key conventions:
- The world is toroidal: positions are wrapped with `wrap(v, W|H)`. Canvas size is the constants `W = 800` and `H = 600`, which must match the `<canvas>` attributes in `index.html`.
- Asteroid sizes are 1 to 3, and the parallel arrays `RADII`, `SPEEDS`, and `POINTS` are indexed by size (index 0 is unused). `Asteroid.split()` returns two children of `size - 1`.
- Collisions are circle-based using `dist()` and each entity's `radius`. Ship vs. asteroid uses `a.radius * 0.82` and is skipped while `ship.invincible > 0`.
- Clearing all asteroids calls `nextLevel()`, which spawns `3 + level` large asteroids outside a safe radius around the center.
