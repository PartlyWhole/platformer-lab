# Platformer Lab — design spec

**Date:** 2026-06-18
**Status:** approved (inline, user)

## Purpose

The third entry in the lab series. Collision Lab answers *"do these two boxes
overlap now?"*; Movement Lab answers *"where is the box next frame?"*. Platformer
Lab answers **"now put them together and build the thing."**

It is a **ladder of standalone, complete, runnable pygame programs**. Each rung
adds **exactly one new idea** over the previous rung. The student can run, copy,
and tweak any rung in isolation, building a robust internal model that flows
Movement → Collision → a clean basic platformer.

## Container

- New 3rd sibling page, repo `PartlyWhole/platformer-lab`, live at
  `https://partlywhole.github.io/platformer-lab/`.
- Reuses `run.html` and `support.js` **verbatim** and the lab design system.
- `index.html` is rewritten (DC component) to hold the ladder.

## Rung panel anatomy

Each rung is one panel in the lab design system:
- Header bar: rung number (Press Start 2P) + SHORT-CAPS title + `<code>` signature chip.
- One-line **why this matters**.
- Code panel (fake titlebar + ▶ RUN LIVE) over `<pre>`, with **lines new/changed
  since the previous rung auto-highlighted** (line-level diff vs previous rung:
  accent left-border + faint accent bg). The RUN LIVE program IS the demo.
- A **`tweak →`** box: one concrete experiment to try.

## The ladder (3 acts, 15 rungs — one new idea each)

### ACT I · MOVEMENT — where is it next frame?
1. THE GAME LOOP — blank window; init/set_mode/clock/while/events+QUIT/fill/flip/tick.
2. A SQUARE AT A POSITION — draw a rect at (x, y); position is just numbers.
3. CONSTANT VELOCITY — `x += vx` each frame.
4. KEYBOARD INPUT — set `vx` from arrow keys via `key.get_pressed()`.
5. GRAVITY — `vy += GRAVITY`; the fall/arc.
6. THE FLOOR — clamp at ground (`if y > GROUND: y = GROUND; vy = 0`). Deliberate
   scaffolding: a floor is the simplest collision; superseded by rung 11.
7. JUMP — on SPACE if `on_ground`: `vy = -JUMP`. Impulse + the on_ground gate.
8. FRICTION + TERMINAL VELOCITY — `vx *= K`; cap `vy` at `MAX_FALL`.

### ACT II · COLLISION — do these two boxes overlap?
9.  TWO RECTS, ONE TEST — `player.colliderect(block)`; color flips on hit.
10. STOP AT A WALL (naïve) — move, then undo if overlapping. Sticky → motivates per-axis.
11. ⭐ RESOLVE PER AXIS — move X then snap to wall edge; move Y then snap, set
    `on_ground` on landing, zero `vy`. The core platformer technique.
12. MANY BLOCKS — loop the per-axis resolve over a list of tile Rects.

### ACT III · THE PLATFORMER — synthesis
13. A LEVEL FROM A MAP — build `solids` from a text grid of `#` tiles; draw + collide.
14. FLOAT POSITION, INT RECT — keep `fx, fy` floats; `rect.x = round(fx)` (Rect-is-int
    callback to Collision Lab / Movement F2). Smooth sub-pixel motion.
15. 🎮 THE PLATFORMER — clean ~70-line capstone: tilemap, float pos+vel, gravity +
    terminal velocity, run + friction, jump gated by downward-axis `on_ground`,
    per-axis tile collision. No camera (kept readable). `tweak →` points to
    coyote-time / variable jump / camera as "going further".

## Code constraints (every rung)

- Complete, standalone program. Standard pygame loop: `for event in
  pygame.event.get(): if event.type == pygame.QUIT: raise SystemExit`, ending with
  `pygame.display.flip()` and `clock.tick(60)`. No manual asyncio (the runner
  AST-transforms the blocking loop to cooperative async).
- Window sized for the runner canvas (~480–560 wide).
- Clean, simple, direct: minimal abstraction; a helper only where it removes real
  duplication (e.g., `move_and_collide` in rungs 11–15). Sparse, meaningful comments.
- **Consistent names across rungs** so diffs stay clean: `vx, vy`; `GRAVITY, JUMP,
  SPEED, K, MAX_FALL, GROUND` constants in CAPS; `solids` = list of `pygame.Rect`;
  float position `fx, fy` once introduced.

## Build & verification

A Workflow authors the full ladder coherently (one author for cross-rung
consistency), then adversarially verifies each rung in parallel:
- syntax (`python3 -m py_compile`) + runner-compatibility (loop shape, real pygame APIs),
- physics/collision correctness via a pure-Python headless harness (extract the
  update math, assert: lands on floor at right y, snaps to wall edges, no tunneling,
  correct `on_ground`),
- pedagogy/delta (exactly one new idea vs prior rung; clean/simple/direct; names consistent).
Findings feed a synthesis/fix pass; re-verify until clean. Then assemble the page,
deploy, and verify live (run rungs end-to-end, especially 11 and 15).

## Out of scope

Camera/scrolling, coyote-time, variable jump height, animation, enemies, sound —
mentioned only as `tweak →` "going further" hints in rung 15.
