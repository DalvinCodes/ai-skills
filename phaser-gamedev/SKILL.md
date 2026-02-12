---
name: phaser-gamedev
description: "Use when building 2D browser games with Phaser 3 (JS/TS), implementing scene-based architecture, physics systems, tilemaps, sprite animations, or optimizing game performance with pooling and culling."
allowed-tools: Read, Bash, Grep, Glob
---

# Phaser Game Development

Build fast, polished 2D browser games using Phaser 3's scene-based architecture and physics systems.

## Philosophy: Games as Living Systems

Games are not static UIs—they are **dynamic systems** where entities interact, state evolves, and player input drives everything.

**Before building, ask**:
- What **scenes** does this game need? (Boot, Menu, Game, Pause, GameOver, UI overlay)
- What are the **core entities** and how do they interact?
- What **state** must persist across scenes (and what must not)?
- What **physics** model fits? (Arcade for speed, Matter for realism)
- What **input methods** will players use? (keyboard/gamepad/touch)

**Core principles**:
1. **Scene-first architecture**: Organize code around scene lifecycle and transitions.
2. **Composition over inheritance**: Build entities from sprite/body/controllers, not deep class trees.
3. **Physics-aware design**: Choose collision model early; don't retrofit physics late.
4. **Asset pipeline discipline**: Preload everything; reference by keys; keep loading deterministic.
5. **Frame-rate independence**: Use `delta` for motion and timers; avoid frame counting.

## Workflow: Build the Spine First

- Define the minimal playable loop (movement + win/lose + restart) before content/polish.
- Decide scene graph and transitions (including UI overlay scene if needed).
- Choose physics system early:
  - **Arcade** for most games (platformers/top-down/shooters).
  - **Matter** for realistic collision/constraints/ragdolls.
  - **None** for non-physics scenes (menus, VN, card games).
- Implement a reliable asset-loading path (Boot scene), then scale out content.
- Add debug toggles/profiling early so performance doesn't become a late surprise.

## References (Progressive Disclosure)

- `references/core-patterns.md`: config, scene lifecycle, objects, input, animations, asset loading, project structure.
- `references/arcade-physics.md`: Arcade physics deep dive (including pooling patterns).
- `references/tilemaps.md`: Tiled tilemap workflows, collision setup, object layers.
- `references/performance.md`: optimization strategies (pooling, batching, culling, camera, texture atlases).
- `references/spritesheets-nineslice.md`: spritesheet loading (spacing/margin), nine-slice UI panels, asset inspection protocol.
- `references/version-gotchas.md`: v3.86–v3.90 breaking changes, camera system, audio gotchas, scene transitions.

## Quick Reference: Version Awareness

**Current stable: Phaser 3.90 "Tsugumi" (May 2025)**

Critical gotchas agents must know:
- `DynamicTexture`/`RenderTexture` default to `forceEven: true` since v3.88 — odd sizes round up silently
- Arcade Physics circle separation was broken before v3.88 — old workarounds are now unnecessary
- iOS 17.5+ audio stops on tab focus changes — handled automatically since v3.88
- `scrollFactor` values other than 1 are **NOT** used in physics collision calculations
- Hex tilemap pixel dimensions were miscalculated before v3.88

See `references/version-gotchas.md` for the full list.

## Anti-Patterns to Avoid

| Anti-Pattern | Why Bad | Better |
|---|---|---|
| Global state soup (`window.*` / module globals) | Scene transitions become fragile; debugging chaotic | Scene data, registries, or a dedicated state manager |
| Loading in `create()` | Assets may not be ready when referenced | Load in `preload()`; use a Boot scene |
| Frame-dependent logic (frame count) | Game speed varies with frame rate | Scale by `(delta / 1000)` |
| Physics overkill (Matter for simple collisions) | Unnecessary complexity + perf cost | Arcade covers most 2D needs |
| Monolithic scenes | Hard to extend; UI/menus/pause become hacks | Separate gameplay vs UI overlay vs menus |
| Magic numbers everywhere | Balancing painful; changes cause regressions | Config objects/constants |
| No pooling for spammy objects | GC spikes cause stutters | Object pooling with groups |
| Assuming spritesheet frame dimensions | Wrong dims cause silent corruption | Open asset, measure, calculate with spacing/margin |
| Ignoring spritesheet spacing | Frames shift progressively | Check source for gaps; use `spacing: N` |
| Scaling discontinuous UI art | Transparent gutters scale too | Slice and stitch cached textures |

## Variation Guidance

Outputs should vary based on:
- **Genre** (platformer vs top-down vs shmup changes physics/input/camera)
- **Target platform** (mobile touch, desktop keyboard, gamepad support)
- **Art style** (pixel art scaling vs HD smoothing; atlas usage)
- **Performance envelope** (many sprites -> pooling/culling; few sprites -> simpler code)
- **Team size / complexity** (prototype vs production architecture)

Avoid converging on one "default" resolution, always defaulting to Arcade, or copy-pasting boilerplate without adapting.

## Remember

Phaser gives powerful primitives—scenes, sprites, physics, input—but **architecture is your responsibility**.

Think in systems: define the scenes, define the entities, define their interactions—then implement.

**AI agents are capable of building complete, polished Phaser games. These guidelines illuminate the path—they don't fence it.**
