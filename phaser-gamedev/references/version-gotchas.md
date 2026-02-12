# Version Gotchas, Camera, Audio, and Scene Transitions

Supplementary reference for Phaser 3 covering version-specific breaking changes, the camera system, audio pitfalls, and advanced scene transitions. Updated for **Phaser 3.90 "Tsugumi" (May 2025)**.

## Version Compatibility

| Version | Name | Date | Key Theme |
|---------|------|------|-----------|
| 3.90 | Tsugumi | May 2025 | Rounded rectangles, collision categories per-body, circle-polygon fix |
| 3.88 | Minami | Feb 2025 | `forceEven` RenderTexture, circle separation fix, Matter crash fix |
| 3.87 | Hanabi | Nov 2024 | Animation frame duration changes, TweenData updates |
| 3.86 | Aoi | Oct 2024 | Tilemap pixel rounding, font loader support |
| 3.85 | Itsuki | Sep 2024 | Stability release |
| 3.80 | Nino | Feb 2024 | Major update baseline |

---

## Breaking Changes and Gotchas by Version

### v3.90 (May 2025)

**`ImageCollections` default changed from `null` to `undefined`**
```javascript
// BEFORE (broken in v3.90):
if (tileset.image === null) { /* handle missing */ }

// AFTER (correct):
if (tileset.image == null) { /* handles both null and undefined */ }
// Or more explicit:
if (tileset.image === undefined) { /* handle missing */ }
```

**Per-body `collisionCategory` and `collisionMask` in Arcade Physics groups**

Previously collision categories only worked at the group level. Now individual bodies within a physics group can have their own unique collision categories:

```javascript
const CATEGORY = { PLAYER: 1, ENEMY: 2, BULLET: 4, WALL: 8 };

// Each body in a group can have unique categories
enemies.children.iterate(enemy => {
  if (enemy.type === 'flying') {
    enemy.body.collisionCategory = CATEGORY.ENEMY;
    enemy.body.collisionMask = CATEGORY.PLAYER | CATEGORY.BULLET;
  } else {
    enemy.body.collisionCategory = CATEGORY.ENEMY;
    enemy.body.collisionMask = CATEGORY.PLAYER | CATEGORY.BULLET | CATEGORY.WALL;
  }
});
```

**Immovable circle-polygon collision fixed** — immovable circle objects no longer move when pushed by polygons. If you had workarounds for this, remove them.

**`createFromTiles` fixed for multiple tilesets** — now correctly handles sprite sheets when multiple tilesets are present.

**Chrome 134+ Text rendering bug** — destroying a right-to-left `Text` object prevented the next left-to-right `Text` from rendering. Fixed with explicit `canvas.dir = 'ltr'` default. If you're on an older Phaser version, add `canvas.dir = 'ltr'` manually after creating text.

**`AnimationFrame` duration** — per-frame duration now works correctly (was broken in v3.87, fixed in v3.90).

### v3.88 (Feb 2025)

**`DynamicTexture` / `RenderTexture` `forceEven: true` by default**

This is the most impactful breaking change. Odd-sized render textures silently round up to the nearest even dimension:

```javascript
// This now creates a 102x102 texture, not 101x101
const rt = this.make.renderTexture({ width: 101, height: 101 });

// To preserve exact odd sizes:
const rt = this.make.renderTexture({
  width: 101,
  height: 101,
  forceEven: false  // Opt out of rounding
});
```

**Arcade Physics circle separation logic fixed** — both bodies now move correctly in circle-to-circle and circle-to-rectangle collisions. Previously only one body would separate. Remove any manual separation workarounds.

**Matter.js tab-switch crash fixed** — `Matter.World.update` no longer hangs when returning to a long-dormant tab (large delta values). Runner config values are properly passed through now.

**iOS 17.5.1+ / iOS 18+ audio fix** — audio no longer stops when losing/gaining focus in Safari. The Web Audio Sound Manager now listens for the `visibilitychange` event and suspends/resumes the AudioContext automatically.

**Audio unlock improvement** — both `mousedown` and `mouseup` events now used for Web Audio unlock (previously only `click`). Audio unlocks earlier on mouse-based devices.

**Hex tilemap dimensions fixed** — `widthInPixels` and `heightInPixels` now correctly account for hexagonal tile overlap. If you were manually calculating hex map dimensions, you can remove that workaround.

**`letterSpacing` added to Text** — new property for character spacing. Interacts correctly with word wrap:

```javascript
const text = this.add.text(100, 100, 'Hello World', {
  fontSize: '24px',
  letterSpacing: 2,    // 2px extra between characters
  wordWrap: { width: 200 }
});
```

**`setTransform` Canvas 2D fallback** — uses legacy 6-argument form for older browsers (Chromium Embedded Framework). No action needed unless you're calling `setTransform` directly.

### v3.87 (Nov 2024)

**Animation frame duration behavior changed** — `TweenData.update` applied eased values instead of final start/end values on completion. This was fixed in v3.88 patch. If on v3.87, be aware that tween completion values may be slightly off.

**`BaseTweenData.duration` minimum clamped** — duration can never be zero or negative (minimum 0.01ms). Previously this caused NaN errors.

### v3.86 (Oct 2024)

**Tilemap pixel rounding** — fixes for tilemap rendering in Phaser Editor. If you saw sub-pixel artifacts in tilemap rendering, this version resolves them.

---

## Camera System

### Following a Target

```javascript
// Basic follow
this.cameras.main.startFollow(player);

// Smooth follow with lerp (0 = no follow, 1 = instant)
this.cameras.main.startFollow(player, true, 0.08, 0.08);

// Round pixel positions (critical for pixel art)
this.cameras.main.setRoundPixels(true);

// Stop following
this.cameras.main.stopFollow();
```

### Dead Zones

The dead zone defines a rectangular region where the target can move without the camera scrolling. The camera only scrolls when the target exits this region.

```javascript
// Set dead zone (centered on camera viewport)
this.cameras.main.setDeadzone(200, 100);

// Access the dead zone rectangle for game logic
const dz = this.cameras.main.deadzone;
if (Phaser.Geom.Rectangle.Contains(dz, enemy.x, enemy.y)) {
  // Enemy is in the camera's dead zone area
}

// Reset dead zone
this.cameras.main.setDeadzone();  // No arguments = remove
```

**Gotcha**: The dead zone rectangle is re-centered on the camera mid-point every frame. Don't cache its position for multi-frame logic.

### Camera Bounds

```javascript
// Constrain camera to world/map bounds
this.cameras.main.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

// Match physics world bounds
this.physics.world.setBounds(0, 0, map.widthInPixels, map.heightInPixels);
this.cameras.main.setBounds(0, 0, map.widthInPixels, map.heightInPixels);
```

### Scroll Factor

Controls how much camera movement affects a game object's rendered position.

```javascript
gameObject.setScrollFactor(1);      // Moves with camera (default)
gameObject.setScrollFactor(0);      // Fixed to screen (HUD elements)
gameObject.setScrollFactor(0.5);    // Parallax (half-speed)
gameObject.setScrollFactor(1, 0.5); // Different X and Y factors
```

**CRITICAL GOTCHA**: Scroll factor values other than 1 are **NOT** used in physics collision calculations. Physics bodies always collide based on their world position, regardless of scroll factor. This means:
- A HUD element with `setScrollFactor(0)` will collide at its world position, not its screen position
- Parallax backgrounds with physics bodies will have mismatched visual and collision positions
- Don't add physics to objects with non-1 scroll factors unless you account for this

### Camera Effects

```javascript
// Fade in/out
this.cameras.main.fadeIn(1000);    // Fade from black over 1s
this.cameras.main.fadeOut(500);    // Fade to black over 0.5s
this.cameras.main.fade(1000, 0, 0, 0, false, (camera, progress) => {
  if (progress === 1) this.scene.start('NextScene');
});

// Flash (white flash for damage, etc.)
this.cameras.main.flash(250, 255, 255, 255);

// Shake (for impacts, explosions)
this.cameras.main.shake(200, 0.01);  // duration, intensity

// Pan to position
this.cameras.main.pan(targetX, targetY, 2000, 'Power2');

// Zoom
this.cameras.main.setZoom(2);       // 2x zoom
this.cameras.main.zoomTo(1.5, 1000); // Animate zoom over 1s

// Chain effects with callbacks
this.cameras.main.pan(500, 300, 2000, 'Sine.easeInOut', false, (cam, progress) => {
  if (progress === 1) {
    cam.shake(200, 0.01);
  }
});
```

### Multiple Cameras

```javascript
// Create secondary camera (e.g., minimap)
const minimap = this.cameras.add(10, 10, 200, 200);
minimap.setZoom(0.2);
minimap.setScroll(0, 0);
minimap.setBackgroundColor(0x002244);

// Camera ignores certain objects
minimap.ignore(uiLayer);
this.cameras.main.ignore(minimapBorder);
```

---

## Audio Gotchas

### Browser Autoplay Policy

All modern browsers require a user gesture (click, tap, keypress) before audio can play. Phaser handles this with an audio unlock mechanism, but you must be aware of the flow:

```javascript
// Audio will NOT play until user interacts with the page
// Phaser queues audio and plays after unlock

// Safe pattern: play audio in response to user input
this.input.on('pointerdown', () => {
  this.sound.play('bgm', { loop: true });
});

// Or check if audio context is unlocked
if (this.sound.locked) {
  this.sound.once('unlocked', () => {
    this.sound.play('bgm', { loop: true });
  });
} else {
  this.sound.play('bgm', { loop: true });
}
```

### iOS Safari Issues

- **iOS 17.5.1+ / iOS 18+**: Audio stops when tab loses/gains focus. Fixed in Phaser v3.88+ (auto suspend/resume of AudioContext on `visibilitychange`). If on older Phaser:

```javascript
// Manual fix for older Phaser versions on iOS
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    this.sound.pauseAll();
  } else {
    this.sound.resumeAll();
  }
});
```

### Firefox Spatial Audio

Firefox doesn't implement `positionX`, `positionY`, `positionZ` on AudioListener. Fixed in Phaser v3.90 with a fallback. On older versions, the `follow` feature of `WebAudioSound` won't work on Firefox.

### Audio Format Support

```javascript
// Provide multiple formats for cross-browser support
this.load.audio('bgm', [
  'assets/audio/music.ogg',   // Chrome, Firefox
  'assets/audio/music.m4a',   // Safari, iOS
  'assets/audio/music.mp3'    // Fallback
]);

// Base64 audio now works (fixed in v3.90)
this.load.audio('sfx', 'data:audio/mp3;base64,...');
```

### Common Audio Mistakes

| Mistake | Consequence | Fix |
|---------|-------------|-----|
| Playing audio in `create()` without user gesture | Silent, no error | Play on first user interaction |
| Only providing .mp3 format | May not work in all browsers | Provide .ogg + .m4a + .mp3 |
| Not handling audio lock | Audio never plays on mobile | Check `sound.locked` and listen for `unlocked` event |
| Large uncompressed audio files | Long load times, memory waste | Compress audio, use streaming for music |
| Creating new audio objects instead of reusing | Memory leaks | Use `sound.add()` once, then `.play()` / `.stop()` |

---

## Advanced Scene Transitions

### Animated Transitions

```javascript
this.scene.transition({
  target: 'GameOverScene',
  duration: 1000,
  moveAbove: true,           // Target renders above current
  data: { score: this.score },
  onUpdate: (progress) => {
    // progress: 0 to 1
    // Use for custom fade/wipe effects
    this.cameras.main.setAlpha(1 - progress);
  }
});

// Listen for transition events
this.events.on('transitionstart', (fromScene, duration) => {});
this.events.on('transitioncomplete', (toScene) => {});
this.events.on('transitionout', (toScene, duration) => {});
```

**Gotcha**: The target scene cannot be the same as the current scene. Attempting this returns `false`.

### Sleep/Wake Pattern (State Preservation)

Unlike `start()` which destroys the current scene, `sleep()`/`wake()` preserves all state:

```javascript
// Pause game, show menu (preserves game state)
this.scene.sleep('GameScene');
this.scene.run('PauseScene');

// Resume from pause menu
// In PauseScene:
this.scene.wake('GameScene', { resumed: true });
this.scene.stop('PauseScene');

// Handle wake event in GameScene
this.events.on('wake', (sys, data) => {
  if (data?.resumed) {
    // Re-enable input, unpause physics, etc.
  }
});
```

### Scene from Object Config (No Class Needed)

Useful for prototyping or dynamically generated scenes:

```javascript
const config = {
  scene: {
    init(data) { this.level = data.level || 1; },
    preload() { this.load.image('logo', 'logo.png'); },
    create() { this.add.image(400, 300, 'logo'); },
    update(time, delta) { /* game loop */ },
    extend: {
      // Additional properties/methods copied to scene
      customMethod() { return 'hello'; },
      data: { score: 0 }  // Merged into scene's Data Manager
    }
  }
};
```

### Scene Communication

```javascript
// Scene A emits event
this.events.emit('playerDied', { lives: this.lives });

// Scene B listens (when running in parallel)
const gameScene = this.scene.get('GameScene');
gameScene.events.on('playerDied', (data) => {
  this.updateLivesDisplay(data.lives);
});

// Via Scene's Data Manager (persists across scene lifecycle)
this.registry.set('highScore', 9999);
const hs = this.registry.get('highScore');

// Listen for registry changes
this.registry.events.on('changedata-highScore', (parent, value, prev) => {
  this.highScoreText.setText('High: ' + value);
});
```

---

## Common Runtime Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `Cannot read properties of null (reading 'source')` | `addBase64` failed silently (v3.88 now returns early) | Check return value, verify base64 data |
| `setTransform` 6 arguments required | Old browser (CEF) + Phaser v3.87 | Update to v3.88+ (uses legacy form) |
| Matter.js hangs/crashes browser | Returning to dormant tab causes huge delta | Update to v3.88+ (proper Runner config) |
| `getDisplayList is not a function` on Layer | Missing method on Layer Game Object | Update to v3.88+ (method added) |
| Timer events hang with negative delays | `repeat` + negative `delay` creates infinite loop | Update to v3.88+, use delay >= 0 |
| Particle emitter color not clearing | RGB arrays not cleared before repopulating | Update to v3.90 |
| Tween `persist: true` destroyed after `completeDelay` | Tween incorrectly cleaned up | Update to v3.88+ |
| `TweenChain` `onStart` not firing | Event dispatch bug | Update to v3.88+ |
| `DeathZone` uses local coordinates | Zone follows emitter position incorrectly | Update to v3.88+ (uses world coords) |

---

## Phaser Math Utilities

Commonly needed helpers that agents often reimplement unnecessarily:

```javascript
// Distance
Phaser.Math.Distance.Between(x1, y1, x2, y2);

// Angle between points (radians)
Phaser.Math.Angle.Between(x1, y1, x2, y2);

// Shortest angular distance (v3.90+)
Phaser.Math.Angle.GetShortestDistance(angle1, angle2);
Phaser.Math.Angle.GetClockwiseDistance(angle1, angle2);

// Clamp value to range
Phaser.Math.Clamp(value, min, max);

// Linear interpolation
Phaser.Math.Linear(a, b, t);  // t: 0-1

// Random
Phaser.Math.Between(min, max);           // Integer
Phaser.Math.FloatBetween(min, max);      // Float
Phaser.Math.RND.pick(array);             // Random element

// Snap to grid
Phaser.Math.Snap.To(value, gridSize);    // Snap to nearest grid point

// Degrees/Radians
Phaser.Math.DegToRad(degrees);
Phaser.Math.RadToDeg(radians);

// Wrap value (useful for angles)
Phaser.Math.Wrap(value, min, max);

// Percent between two values
Phaser.Math.Percent(value, min, max);
```
