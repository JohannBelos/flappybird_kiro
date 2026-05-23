# Requirements Document

## Introduction

Flappy Kiro is a browser-based arcade game inspired by the classic Flappy Bird format. The player controls Ghosty, a ghost character, navigating through a series of wall pairs with gaps at random heights. Ghosty moves automatically to the right and falls due to gravity; the player taps the spacebar to make Ghosty ascend. Each wall pair successfully passed awards one point. Colliding with a wall or the ground ends the game. The game features a Halloween ambient theme and uses existing assets: a ghost sprite (`assets/ghosty.png`), a jump sound (`assets/jump.wav`), and a game-over sound (`assets/game_over.wav`).

---

## Glossary

- **Game**: The Flappy Kiro browser-based arcade application.
- **Ghosty**: The ghost character sprite controlled by the player, rendered using `assets/ghosty.png`.
- **Player**: The human user interacting with the Game via keyboard input.
- **Wall**: A vertical rectangular obstacle rendered on the canvas. Walls come in pairs (top and bottom) with a gap between them.
- **Wall_Pair**: A set of two Walls (one top, one bottom) sharing the same horizontal position, with a fixed-size gap at a random vertical position.
- **Gap**: The open vertical space between the top and bottom Wall of a Wall_Pair through which Ghosty must pass.
- **Ground**: The bottom boundary of the game canvas, rendered as a solid strip 20 pixels tall at the bottom of the canvas.
- **Score**: An integer counter representing the number of Wall_Pairs Ghosty has successfully passed through.
- **Gravity**: The constant downward acceleration applied to Ghosty each game tick.
- **Jump**: The upward velocity impulse applied to Ghosty when the Player presses the spacebar.
- **Collision**: The event that occurs when Ghosty's bounding box overlaps with a Wall or the Ground.
- **Game_Loop**: The recurring update cycle that advances physics, moves objects, checks collisions, and redraws the canvas.
- **Renderer**: The component responsible for drawing all visual elements onto the HTML5 canvas.
- **Physics_Engine**: The component responsible for applying Gravity and Jump velocity to Ghosty's position each tick.
- **Collision_Detector**: The component responsible for detecting Collisions between Ghosty and Walls or the Ground.
- **Score_Tracker**: The component responsible for incrementing and displaying the Score.
- **Audio_Manager**: The component responsible for loading and playing sound effects.
- **Wall_Spawner**: The component responsible for generating Wall_Pairs at regular horizontal intervals with random Gap positions.
- **Start_Screen**: The initial screen displayed before gameplay begins.
- **Game_Over_Screen**: The screen displayed after a Collision ends gameplay.

---

## Requirements

### Requirement 1: Game Initialization and Start Screen

**User Story:** As a Player, I want to see a start screen when I open the game, so that I know how to begin playing.

#### Acceptance Criteria

1. WHEN the Game is loaded in a browser, THE Renderer SHALL display the Start_Screen with the game title "Flappy Kiro", the Ghosty sprite, and an instruction to press the spacebar to start.
2. WHILE the Start_Screen is displayed, THE Game_Loop SHALL remain paused and Ghosty SHALL NOT respond to spacebar input until the Player initiates the game.
3. WHEN the Player presses the spacebar on the Start_Screen, THE Game SHALL transition to the active gameplay state — defined as Ghosty being visible and responsive to input with the Game_Loop running — and begin the Game_Loop.
4. IF the `assets/ghosty.png` sprite fails to load, THEN THE Renderer SHALL display a placeholder rectangle of 40×40 pixels in place of the Ghosty sprite on the Start_Screen so that the screen remains fully rendered.
5. WHEN the Player clicks the mouse on the Start_Screen, THE Game SHALL transition to the active gameplay state in the same manner as a spacebar press.

---

### Requirement 2: Ghosty Movement and Physics

**User Story:** As a Player, I want Ghosty to fall automatically and rise when I press spacebar, so that I can navigate through the walls.

#### Acceptance Criteria

1. WHILE the Game_Loop is active, THE Physics_Engine SHALL apply a constant downward acceleration of 0.5 pixels per tick squared to Ghosty's vertical velocity each tick.
2. WHILE the Game_Loop is active, THE Physics_Engine SHALL update Ghosty's vertical position by adding the current vertical velocity to Ghosty's y-coordinate each tick.
3. WHILE the Game_Loop is active, THE Physics_Engine SHALL keep Ghosty's horizontal screen position fixed; wall scrolling SHALL simulate forward movement.
4. WHEN the Player presses the spacebar while the Game_Loop is active and the game is not in a Game_Over state, THE Physics_Engine SHALL set Ghosty's vertical velocity to -8 pixels per tick, replacing any current velocity.
5. WHEN the Player presses the spacebar while the Game_Loop is active and the game is not in a Game_Over state, THE Audio_Manager SHALL play the `assets/jump.wav` sound effect.
6. WHILE the Game_Loop is active, THE Physics_Engine SHALL clamp Ghosty's vertical velocity to a minimum of -8 pixels per tick (upward) and a maximum of +12 pixels per tick (downward).
7. WHEN a new game session starts, THE Physics_Engine SHALL initialize Ghosty's vertical velocity to 0 pixels per tick before the first Game_Loop tick executes.

---

### Requirement 3: Wall Generation and Scrolling

**User Story:** As a Player, I want walls to appear continuously as I fly, so that the game presents an ongoing challenge.

#### Acceptance Criteria

1. WHILE the Game_Loop is active, THE Wall_Spawner SHALL generate a new Wall_Pair every 90 game ticks with the Wall_Pair's left edge positioned at the canvas width x-coordinate.
2. THE Wall_Spawner SHALL position the Gap of each Wall_Pair at a random vertical offset such that the Gap top is between 60 pixels and (canvas height − Gap size − 60) pixels from the top of the canvas.
3. THE Wall_Spawner SHALL set the Gap height to 150 pixels and each Wall's width to 50 pixels for every Wall_Pair.
4. WHILE the Game_Loop is active, THE Renderer SHALL scroll all Wall_Pairs leftward at 3 pixels per tick.
5. WHEN a Wall_Pair's right edge x-coordinate becomes less than 0, THE Game SHALL remove that Wall_Pair from the active object list.
6. WHEN the Game_Loop starts a new session, THE Wall_Spawner SHALL generate the first Wall_Pair immediately on tick 1 rather than waiting 90 ticks.

---

### Requirement 4: Collision Detection and Game Over

**User Story:** As a Player, I want the game to end when Ghosty hits a wall or the ground, so that the game has meaningful consequences for mistakes.

#### Acceptance Criteria

1. WHILE the Game_Loop is active, THE Collision_Detector SHALL check Ghosty's bounding box — inset 2 pixels from each edge of the sprite — against every active Wall_Pair's top and bottom Wall bounding boxes each tick.
2. WHILE the Game_Loop is active, THE Collision_Detector SHALL check whether Ghosty's bottom edge has reached or exceeded the Ground (bottom of the canvas) each tick.
3. WHEN a Collision is detected, THE Game SHALL stop the Game_Loop within the same tick and SHALL NOT trigger further collision checks for that session.
4. WHEN a Collision is detected, THE Audio_Manager SHALL play the `assets/game_over.wav` sound effect.
5. WHEN a Collision is detected, THE Renderer SHALL display the Game_Over_Screen showing the final Score and an instruction to press the spacebar to restart within the same tick that the Game_Loop stops.
6. WHEN the Player presses the spacebar on the Game_Over_Screen, THE Game SHALL reset all state — including Score to 0, Ghosty's position and velocity to initial values, and all active Wall_Pairs removed — and return to the Start_Screen.

---

### Requirement 5: Scoring

**User Story:** As a Player, I want to earn a point for each wall pair I pass through, so that I can track my progress and compete with myself.

#### Acceptance Criteria

1. WHILE the Game_Loop is active, THE Score_Tracker SHALL increment the Score by 1 each time Ghosty's right edge crosses past the right edge of a Wall_Pair for the first time.
2. WHILE the Game_Loop is active, THE Renderer SHALL display the current Score as an integer horizontally centered within the top 15% of the canvas height.
3. THE Score_Tracker SHALL ensure each Wall_Pair contributes at most 1 point to the Score, regardless of Ghosty's position relative to that Wall_Pair after passing.
4. WHEN a new game session starts, THE Score_Tracker SHALL initialize the Score to 0 before the first Game_Loop tick executes.
5. WHEN a new game session starts after a previous session ended, THE Score_Tracker SHALL reset the Score to 0 before the first Game_Loop tick of the new session executes.

---

### Requirement 6: Halloween Ambient Theme and Visual Presentation

**User Story:** As a Player, I want the game to have a Halloween atmosphere, so that the experience feels thematic and immersive.

#### Acceptance Criteria

1. THE Renderer SHALL render the game canvas background with the exact color `#1a0a2e` (deep purple-black) on every frame to establish the Halloween night atmosphere.
2. THE Renderer SHALL render Ghosty using the `assets/ghosty.png` sprite at a display size of 40×40 pixels.
3. THE Renderer SHALL render all Wall_Pair rectangles with the exact fill color `#4a0e6e` (dark purple).
4. THE Renderer SHALL render the Ground as a solid strip 20 pixels tall at the bottom of the canvas with the exact fill color `#2d1b00` (dark brown).
5. THE Renderer SHALL display all Score text in the exact color `#ffffff` at a font size of at least 24 pixels, and all instructional text in the exact color `#ffffff` at a font size of at least 16 pixels.
6. THE Renderer SHALL render the game canvas at a fixed resolution of 480 pixels wide by 640 pixels tall.

---

### Requirement 7: Audio Management

**User Story:** As a Player, I want sound effects to play at the right moments, so that the game feels responsive and engaging.

#### Acceptance Criteria

1. WHEN the Game is loaded, THE Audio_Manager SHALL preload `assets/jump.wav` and `assets/game_over.wav` so that playback begins within 50 milliseconds of the triggering event.
2. WHEN a spacebar press triggers the jump action while `assets/jump.wav` is still playing, THE Audio_Manager SHALL reset the playback position of `assets/jump.wav` to the beginning and resume playback immediately.
3. IF an audio file fails to load, THEN THE Audio_Manager SHALL log an error message to the browser console and all subsequent play calls for that file SHALL be silently ignored so that game operation continues without audio.
4. WHEN a Collision is detected and `assets/game_over.wav` is already playing, THE Audio_Manager SHALL restart `assets/game_over.wav` from the beginning.

---

### Requirement 8: Responsive Canvas and Browser Compatibility

**User Story:** As a Player, I want the game to run in a standard desktop browser without installation, so that I can play it easily.

#### Acceptance Criteria

1. THE Game SHALL render using an HTML5 `<canvas>` element with a fixed resolution of 480×640 pixels, fully visible within a desktop viewport of at least 800×600 pixels without scrolling.
2. THE Game SHALL be playable in Chrome 120+, Firefox 120+, and Edge 120+ without requiring browser plugins or extensions.
3. IF the browser does not support the HTML5 Canvas API, THEN THE Game SHALL display a text message reading "Your browser does not support HTML5 Canvas. Please upgrade to a modern browser." centered in the page body.
