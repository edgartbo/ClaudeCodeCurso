# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in vanilla HTML5 Canvas + ES6 JavaScript. No build tools, no bundler, no package manager, no dependencies, no tests.

## Running

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build, lint, or test command — the entire game logic lives in a single file loaded directly by the browser.

## Architecture

Everything runs in `game.js` (~420 lines), loaded by `index.html` into a single `<canvas id="canvas">` (800×600, fixed size). Structure, top to bottom:

- **Input**: raw keydown/keyup listeners fill two global maps, `keys` (held state) and `justPressed`/`pressed()` (edge-triggered, used for shoot/restart so holding Space doesn't repeat-fire faster than the cooldown).
- **Entity classes** (`Bullet`, `Asteroid`, `Ship`, `Particle`): each owns its own `update(dt)` and `draw()`. There is no shared base class or entity-component system — new entity types are expected to follow the same `update`/`draw`/`dead` pattern.
- **Toroidal space**: all movement wraps position through `wrap(v, max)` on both axes (`W`/`H` = 800/600), so the play field has no edges.
- **Global mutable game state**: `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` (`'playing' | 'dead' | 'gameover'`) are module-level `let`s reassigned by `initGame()`/`nextLevel()`, not encapsulated in a class or store.
- **Asteroid sizing**: size is an integer 3 (large) → 1 (small); `RADII`, `SPEEDS`, `POINTS` arrays are indexed by size. `Asteroid.split()` produces two asteroids one size smaller (size 1 splits into nothing).
- **Game loop**: a single `requestAnimationFrame` loop (`loop(ts)`) computes `dt` in seconds (clamped to 0.05 max to avoid huge jumps on tab-switch) and calls `update(dt)` then `draw()` each frame. All physics/timers are dt-scaled, not frame-scaled.
- **Collision detection**: brute-force O(n·m) distance checks (`dist()`, circle-radius based) for bullet-vs-asteroid and ship-vs-asteroid; no spatial partitioning.
- **State transitions**: `killShip()` and `nextLevel()` mutate `state`/`lives`/`level` directly and are called from inside `update()`. `state === 'gameover'` and `'dead'` short-circuit `update()` into a reduced branch (particles/asteroids only keep animating; `'gameover'` waits for Space to call `initGame()`).

Note: the README (Spanish) advertises power-ups and a "shooting star" asteroid variant as features — these are not present in the current `game.js`; treat the README as aspirational/outdated on those points rather than a spec.
