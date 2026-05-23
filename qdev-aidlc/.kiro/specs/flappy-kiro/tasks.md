# Implementation Plan: Flappy Kiro

## Overview

Implement the complete Flappy Kiro game as a single `index.html` file using vanilla JavaScript and the HTML5 Canvas 2D API. The implementation follows the module-pattern architecture defined in the design document, building each subsystem incrementally and wiring them together in the final steps. Property-based tests use **fast-check** loaded via CDN.

---

## Tasks

- [~] 1. Project scaffold — `index.html` skeleton, canvas setup, browser fallback
  - Create `index.html` with `<!DOCTYPE html>`, `<meta charset="UTF-8">`, and `<title>Flappy Kiro</title>`
  - Add a `<style>` block that centers the canvas on a dark body background
  - Add `<canvas id="gameCanvas" width="480" height="640">` with the fallback text: "Your browser does not support HTML5 Canvas. Please upgrade to a modern browser."
  - Add an empty `<script>` block with numbered section comments (1–11) matching the design's inline structure
  - _Requirements: 8.1, 8.2, 8.3_

- [x] 2. Constants and `GameState`
  - [x] 2.1 Define all named constants at the top of the `<script>` block
    - `CANVAS_W = 480`, `CANVAS_H = 640`, `GROUND_H = 20`
    - Physics: `GRAVITY = 0.5`, `JUMP_VY = -8`, `VY_MIN = -8`, `VY_MAX = 12`
    - Wall: `SPAWN_INTERVAL = 90`, `WALL_WIDTH = 50`, `GAP_HEIGHT = 150`, `SCROLL_SPEED = 3`, `GAP_MIN_TOP = 60`, `GAP_MAX_TOP = 430`
    - Colors: `BG_COLOR = '#1a0a2e'`, `WALL_COLOR = '#4a0e6e'`, `GROUND_COLOR = '#2d1b00'`, `TEXT_COLOR = '#ffffff'`
    - _Requirements: 2.1, 2.4, 2.6, 3.2, 3.3, 3.4, 6.1, 6.3, 6.4_
  - [x] 2.2 Define the `GameState` object with all fields from the design
    - `state: 'start'`, `tick: 0`, `score: 0`, `walls: []`
    - `ghosty` sub-object: `x: 80`, `y: 260`, `vy: 0`, `width: 40`, `height: 40`, `spriteLoaded: false`, `spriteEl: null`
    - _Requirements: 1.1, 2.3, 2.7, 5.4_

- [x] 3. `Audio_Manager`
  - [x] 3.1 Implement `Audio_Manager.init()`
    - Create `HTMLAudioElement` for `assets/jump.wav` and `assets/game_over.wav`
    - Store both in `Audio_Manager.sounds`
    - Attach `onerror` handlers that log to console and set the entry to `null`
    - _Requirements: 7.1, 7.3_
  - [x] 3.2 Implement `Audio_Manager.play(name)`
    - Guard: if `sounds[name]` is `null` or `undefined`, return silently
    - Reset `currentTime = 0` then call `.play()`
    - _Requirements: 7.2, 7.4_

- [x] 4. `Physics_Engine`
  - [x] 4.1 Implement `Physics_Engine.tick(state)`
    - Add `GRAVITY` to `state.ghosty.vy`
    - Clamp `vy` to `[VY_MIN, VY_MAX]`
    - Add `vy` to `state.ghosty.y`
    - _Requirements: 2.1, 2.2, 2.6_
  - [ ]* 4.2 Write property test for `Physics_Engine.tick` — gravity accumulation (Property 1)
    - **Property 1: Gravity accumulates velocity correctly**
    - For random `vy` in [-8, 12], after one tick `vy_new === clamp(vy + 0.5, -8, 12)`
    - Generator: `fc.float({ min: -8, max: 12 })`
    - **Validates: Requirements 2.1, 2.6**
    - Tag comment: `// Feature: flappy-kiro, Property 1: Gravity accumulates velocity correctly`
  - [ ]* 4.3 Write property test for `Physics_Engine.tick` — position update (Property 2)
    - **Property 2: Position updates by velocity each tick**
    - For random `(y, vy)`, after one tick `y_new === y + vy` (velocity applied before clamping position)
    - Generator: `fc.tuple(fc.float(), fc.float({ min: -8, max: 12 }))`
    - **Validates: Requirements 2.2**
    - Tag comment: `// Feature: flappy-kiro, Property 2: Position updates by velocity each tick`
  - [x] 4.4 Implement `Physics_Engine.applyJump(state)`
    - Set `state.ghosty.vy = JUMP_VY` unconditionally
    - _Requirements: 2.4_
  - [ ]* 4.5 Write property test for `Physics_Engine.applyJump` — jump velocity (Property 3)
    - **Property 3: Jump always sets velocity to -8**
    - For any current `vy`, after `applyJump` the velocity shall equal exactly -8
    - Generator: `fc.float({ min: -8, max: 12 })`
    - **Validates: Requirements 2.4**
    - Tag comment: `// Feature: flappy-kiro, Property 3: Jump always sets velocity to -8`
  - [ ]* 4.6 Write property test for velocity clamping invariant (Property 4)
    - **Property 4: Velocity is always clamped within bounds**
    - For any sequence of tick/jump actions, `vy` always remains in [-8, 12]
    - Generator: `fc.array(fc.oneof(fc.constant('tick'), fc.constant('jump')), { minLength: 1, maxLength: 200 })`
    - **Validates: Requirements 2.6**
    - Tag comment: `// Feature: flappy-kiro, Property 4: Velocity is always clamped within bounds`
  - [x] 4.7 Implement `Physics_Engine.reset(state)`
    - Reset `state.ghosty.y = 260`, `state.ghosty.vy = 0`
    - _Requirements: 2.7, 4.6_

- [x] 5. `Wall_Spawner`
  - [x] 5.1 Implement `Wall_Spawner.spawnWall(state)`
    - Generate a random `gapTop` in `[GAP_MIN_TOP, GAP_MAX_TOP]` using `Math.random()`
    - Push a new `WallPair` object `{ x: CANVAS_W, gapTop, gapHeight: GAP_HEIGHT, wallWidth: WALL_WIDTH, scored: false }` onto `state.walls`
    - _Requirements: 3.1, 3.2, 3.3_
  - [ ]* 5.2 Write property test for gap position bounds (Property 5)
    - **Property 5: Wall gap position is always within valid bounds**
    - For any generated `WallPair`, `gapTop` shall be in [60, 430]
    - Generator: `fc.integer({ min: 0, max: 999999 })` used as a seed to drive `Math.random` via a seeded helper
    - **Validates: Requirements 3.2**
    - Tag comment: `// Feature: flappy-kiro, Property 5: Wall gap position is always within valid bounds`
  - [ ]* 5.3 Write property test for fixed wall dimensions (Property 6)
    - **Property 6: Every generated wall pair has correct fixed dimensions**
    - For any generated `WallPair`, `gapHeight === 150` and `wallWidth === 50`
    - Generator: same as P5
    - **Validates: Requirements 3.3**
    - Tag comment: `// Feature: flappy-kiro, Property 6: Every generated wall pair has correct fixed dimensions`
  - [x] 5.4 Implement `Wall_Spawner.tick(state)`
    - Increment `state.tick`
    - Spawn on tick 1 and whenever `(state.tick - 1) % SPAWN_INTERVAL === 0`
    - Scroll all walls: `wall.x -= SCROLL_SPEED`
    - Cull walls where `wall.x + wall.wallWidth < 0`
    - _Requirements: 3.1, 3.4, 3.5, 3.6_
  - [ ]* 5.5 Write property test for wall scrolling (Property 7)
    - **Property 7: Walls scroll left by exactly 3 pixels per tick**
    - For random wall `x`, after one tick `x_new === x - 3`
    - Generator: `fc.integer({ min: 0, max: 1000 })`
    - **Validates: Requirements 3.4**
    - Tag comment: `// Feature: flappy-kiro, Property 7: Walls scroll left by exactly 3 pixels per tick`
  - [ ]* 5.6 Write property test for off-screen wall culling (Property 8)
    - **Property 8: Off-screen walls are always removed**
    - After any tick, no `WallPair` shall have `wall.x + wall.wallWidth < 0`
    - Generator: `fc.array(fc.integer({ min: -200, max: 600 }))` as initial x-positions
    - **Validates: Requirements 3.5**
    - Tag comment: `// Feature: flappy-kiro, Property 8: Off-screen walls are always removed`
  - [x] 5.7 Implement `Wall_Spawner.reset(state)`
    - Clear `state.walls = []` and reset the internal tick counter to 0
    - _Requirements: 4.6_

- [x] 6. Checkpoint — core physics and wall subsystems
  - Ensure all tests pass, ask the user if questions arise.

- [x] 7. `Collision_Detector`
  - [x] 7.1 Implement `Collision_Detector._aabb(a, b)`
    - Pure function: returns `true` iff rectangles `a` and `b` overlap on both axes
    - `a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y`
    - _Requirements: 4.1_
  - [ ]* 7.2 Write property test for AABB correctness (Property 9)
    - **Property 9: AABB collision detection is correct for all positions**
    - For random rect pairs, `_aabb` returns `true` iff the rectangles geometrically intersect
    - Generator: `fc.tuple(fc.record({ x: fc.integer(), y: fc.integer(), w: fc.nat({ max: 200 }), h: fc.nat({ max: 200 }) }), fc.record({ x: fc.integer(), y: fc.integer(), w: fc.nat({ max: 200 }), h: fc.nat({ max: 200 }) }))`
    - **Validates: Requirements 4.1**
    - Tag comment: `// Feature: flappy-kiro, Property 9: AABB collision detection is correct for all positions`
  - [x] 7.3 Implement `Collision_Detector._ghostyRect(ghosty)` and `Collision_Detector.check(state)`
    - `_ghostyRect`: return `{ x: g.x+2, y: g.y+2, w: g.width-4, h: g.height-4 }`
    - `check`: test inset rect vs top and bottom wall rects for every `WallPair`
    - Ground check: `ghosty.y + ghosty.height - 2 >= CANVAS_H - GROUND_H`
    - On hit: set `state.state = 'gameover'`, call `Audio_Manager.play('game_over')`
    - _Requirements: 4.1, 4.2, 4.3, 4.4_
  - [ ]* 7.4 Write property test for ground collision threshold (Property 10)
    - **Property 10: Ground collision is detected at the correct threshold**
    - For random `ghosty.y`, collision detected iff `y + 38 >= 620`
    - Generator: `fc.integer({ min: -100, max: 700 })`
    - **Validates: Requirements 4.2**
    - Tag comment: `// Feature: flappy-kiro, Property 10: Ground collision is detected at the correct threshold`

- [x] 8. `Score_Tracker`
  - [x] 8.1 Implement `Score_Tracker.check(state)`
    - For each wall where `!wall.scored`: if `ghosty.x + ghosty.width > wall.x + wall.wallWidth`, increment `state.score` and set `wall.scored = true`
    - _Requirements: 5.1, 5.3_
  - [ ]* 8.2 Write property test for one-point-per-wall invariant (Property 11)
    - **Property 11: Each wall pair contributes at most one point**
    - For any wall pair that Ghosty has passed, calling `check` N more times shall not increment score again
    - Generator: `fc.integer({ min: 1, max: 50 })` (number of extra ticks after crossing)
    - **Validates: Requirements 5.1, 5.3**
    - Tag comment: `// Feature: flappy-kiro, Property 11: Each wall pair contributes at most one point`
  - [x] 8.3 Implement `Score_Tracker.reset(state)`
    - Set `state.score = 0`
    - _Requirements: 5.4, 5.5_

- [x] 9. `Renderer`
  - [x] 9.1 Implement `Renderer.init(canvasEl)` and private drawing helpers
    - Store canvas reference and acquire `getContext('2d')`
    - `_drawBackground()`: `fillRect` with `BG_COLOR` covering full canvas
    - `_drawGround()`: `fillRect` with `GROUND_COLOR` at `y = CANVAS_H - GROUND_H`, height `GROUND_H`
    - `_drawWalls(walls)`: for each `WallPair` draw top wall and bottom wall rects with `WALL_COLOR`
    - _Requirements: 6.1, 6.3, 6.4_
  - [ ]* 9.2 Write property test for wall fill color (Property 13)
    - **Property 13: Wall rendering always uses the correct fill color**
    - For any `WallPair`, after `_drawWalls` the canvas `fillStyle` shall equal `'#4a0e6e'`
    - Use a mock canvas context that records `fillStyle` assignments
    - Generator: `fc.record({ x: fc.integer({ min: 0, max: 480 }), gapTop: fc.integer({ min: 60, max: 430 }) })`
    - **Validates: Requirements 6.3**
    - Tag comment: `// Feature: flappy-kiro, Property 13: Wall rendering always uses the correct fill color`
  - [x] 9.3 Implement `Renderer._drawGhosty(ghosty)`
    - If `ghosty.spriteLoaded`: `drawImage(ghosty.spriteEl, ghosty.x, ghosty.y, 40, 40)`
    - Else: draw a 40×40 filled placeholder rectangle at `(ghosty.x, ghosty.y)`
    - _Requirements: 1.4, 6.2_
  - [x] 9.4 Implement `Renderer._drawScore(score)`, `_drawStartScreen()`, and `_drawGameOverScreen(score)`
    - `_drawScore`: centered text, `TEXT_COLOR`, font size ≥ 24 px, positioned in top 15% of canvas
    - `_drawStartScreen`: title "Flappy Kiro", Ghosty sprite/placeholder, spacebar instruction
    - `_drawGameOverScreen`: "Game Over" heading, final score, spacebar-to-restart instruction
    - _Requirements: 1.1, 4.5, 5.2, 6.5_
  - [ ]* 9.5 Write property test for score text color and font size (Property 14)
    - **Property 14: Score text always uses correct color and minimum font size**
    - For any score value, `fillStyle === '#ffffff'` and parsed font size ≥ 24
    - Use a mock canvas context that records `fillStyle` and `font` assignments
    - Generator: `fc.integer({ min: 0, max: 9999 })`
    - **Validates: Requirements 6.5**
    - Tag comment: `// Feature: flappy-kiro, Property 14: Score text always uses correct color and minimum font size`
  - [x] 9.6 Implement `Renderer.draw(state)` master dispatch
    - Always call `_drawBackground()`, `_drawGround()`
    - If `state === 'start'`: call `_drawStartScreen()`
    - If `state === 'playing'`: call `_drawWalls`, `_drawGhosty`, `_drawScore`
    - If `state === 'gameover'`: call `_drawWalls`, `_drawGhosty`, `_drawGameOverScreen`
    - _Requirements: 1.1, 4.5, 5.2, 6.1_

- [x] 10. `Game_Loop`
  - [x] 10.1 Implement `Game_Loop._tick()` with the correct subsystem tick order
    - If `state === 'playing'`: call `Physics_Engine.tick`, `Wall_Spawner.tick`, `Collision_Detector.check`, `Score_Tracker.check`
    - Always call `Renderer.draw(GameState)`
    - Schedule next frame: `Game_Loop._rafId = requestAnimationFrame(Game_Loop._tick.bind(Game_Loop))`
    - _Requirements: 2.1, 2.2, 3.1, 4.1, 4.3, 5.1_
  - [x] 10.2 Implement `Game_Loop.start()`, `Game_Loop.stop()`, and `Game_Loop.reset()`
    - `start()`: call `requestAnimationFrame` to begin the tick cycle
    - `stop()`: call `cancelAnimationFrame(Game_Loop._rafId)`
    - `reset()`: call `Physics_Engine.reset`, `Wall_Spawner.reset`, `Score_Tracker.reset`, set `GameState.state = 'start'`, set `GameState.tick = 0`
    - _Requirements: 4.3, 4.6, 5.4, 5.5_
  - [ ]* 10.3 Write property test for full game reset (Property 12)
    - **Property 12: Game reset restores all state to initial values**
    - For any game state (arbitrary score, walls, vy, y, state string), after `Game_Loop.reset()` verify: `score === 0`, `vy === 0`, `walls.length === 0`, `state === 'start'`
    - Generator: `fc.record({ score: fc.nat({ max: 9999 }), vy: fc.float({ min: -8, max: 12 }), y: fc.float({ min: 0, max: 640 }), wallCount: fc.nat({ max: 10 }) })`
    - **Validates: Requirements 4.6, 5.4, 5.5, 2.7**
    - Tag comment: `// Feature: flappy-kiro, Property 12: Game reset restores all state to initial values`

- [x] 11. Input handlers
  - [x] 11.1 Implement `keydown` event listener on `document`
    - If `key === ' '` (spacebar):
      - `state === 'start'` → `GameState.state = 'playing'`, `Game_Loop.start()`
      - `state === 'playing'` → `Physics_Engine.applyJump(GameState)`, `Audio_Manager.play('jump')`
      - `state === 'gameover'` → `Game_Loop.reset()`, `Game_Loop.start()`
    - _Requirements: 1.3, 2.4, 2.5, 4.6_
  - [x] 11.2 Implement `click` event listener on the canvas element
    - Same branching logic as the spacebar handler
    - _Requirements: 1.5_

- [x] 12. Boot sequence and sprite preload
  - [x] 12.1 Implement the boot sequence at the bottom of the `<script>` block
    - Create `ghosty.spriteEl = new Image()`, set `src = 'assets/ghosty.png'`
    - `onload`: set `ghosty.spriteLoaded = true`
    - `onerror`: leave `spriteLoaded = false` (placeholder path)
    - Call `Audio_Manager.init()`
    - Call `Renderer.init(document.getElementById('gameCanvas'))`
    - Call `Game_Loop.start()` to begin the rAF loop (renders start screen immediately)
    - _Requirements: 1.1, 1.4, 7.1_

- [x] 13. Checkpoint — full integration
  - Ensure all tests pass, ask the user if questions arise.

- [x] 14. Integration and smoke verification
  - [x] 14.1 Write smoke tests: canvas element exists with correct dimensions
    - Assert `canvas.width === 480` and `canvas.height === 640`
    - Assert both `Audio_Manager.sounds.jump` and `Audio_Manager.sounds.game_over` are created after `init()`
    - _Requirements: 6.6, 7.1, 8.1_
  - [ ]* 14.2 Write integration test: game runs 200 ticks without throwing
    - Simulate 200 calls to `Game_Loop._tick()` with `GameState.state = 'playing'`
    - Assert no exceptions are thrown and `GameState.tick === 200`
    - _Requirements: 2.1, 3.1, 4.1, 5.1_
  - [ ]* 14.3 Write integration test: score increments after Ghosty passes a wall
    - Place a `WallPair` at `x = 60` (right edge at 110) with `scored = false`
    - Set `ghosty.x = 80`, `ghosty.width = 40` (right edge at 120 > 110)
    - Call `Score_Tracker.check(GameState)` and assert `GameState.score === 1` and `wall.scored === true`
    - _Requirements: 5.1, 5.3_

---

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Property-based tests require **fast-check** loaded via CDN: `<script src="https://cdn.jsdelivr.net/npm/fast-check/lib/bundle/fast-check.min.js"></script>`
- Each property test must include the tag comment `// Feature: flappy-kiro, Property N: <text>` and run a minimum of 100 iterations
- All subsystems communicate exclusively through the shared `GameState` object — no direct cross-component calls except through `Game_Loop._tick()`
- The `scored` flag on `WallPair` is the idempotency guard for scoring (Property 11)
- The `spriteLoaded` flag on `ghosty` is the guard for the sprite/placeholder rendering branch (Requirement 1.4)
- Checkpoints at tasks 6 and 13 are good moments to open `index.html` in a browser and verify visually

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["2.1", "2.2"] },
    { "id": 1, "tasks": ["3.1", "4.1", "4.4", "4.7", "5.1", "5.4", "5.7", "8.3"] },
    { "id": 2, "tasks": ["3.2", "4.2", "4.3", "4.5", "4.6", "5.2", "5.3", "5.5", "5.6", "7.1", "8.1"] },
    { "id": 3, "tasks": ["7.2", "7.3", "8.2", "9.1"] },
    { "id": 4, "tasks": ["7.4", "9.2", "9.3", "9.4"] },
    { "id": 5, "tasks": ["9.5", "9.6", "10.1"] },
    { "id": 6, "tasks": ["10.2", "10.3"] },
    { "id": 7, "tasks": ["11.1", "11.2"] },
    { "id": 8, "tasks": ["12.1"] },
    { "id": 9, "tasks": ["14.1", "14.2", "14.3"] }
  ]
}
```
