# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This folder contains one self-contained static page, `fireworks.html`, an interactive fireworks demo drawn on an HTML5 canvas. The HTML, CSS and JS all live in that one file. There is no build step, package manager, test suite or linter, and the page loads no external resources.

## Running

Open the file in a browser:

```
open fireworks.html                       # default browser
open -a "Google Chrome" fireworks.html    # specific browser
```

To see edits, reload the page. No server is needed.

## Architecture (inside `fireworks.html`)

- **Render loop (`loop`)**: Each frame is painted with a translucent dark fill instead of being cleared, which is what leaves the motion trails. Particles are then drawn with `globalCompositeOperation = 'lighter'`, so overlapping colors blend additively. If you change the background color, update it in three places to match: the `rgba(5, 6, 15, …)` fade fill, the `#05060f` body background, and the Clear handler's fill.
- **Two entity arrays, `rockets` and `particles`**: Both are updated and drawn every frame, walking backward so dead entries can be spliced out safely.
  - A `Rocket` flies from the bottom of the screen to a target point over a set number of frames, dropping tail sparks (short-lived `Particle`s) along the way. When its `life` runs out, the loop calls `explode()` at its position.
  - A `Particle` is the one shared primitive. Its constructor arguments (gravity multiplier, life, friction, size, saturation) cover every look: burst sparks, tail sparks, and the burst flash (a big particle with zero gravity and a short life).
- **Burst shapes (`explode`)**: A `switch` on the shape name sets each particle's starting velocity (sphere, ring, heart via the parametric heart curve, star via interpolation between vertices, willow). To add a shape:
  1. Add a `case` in `explode`.
  2. Add its name to `SHAPES` so Random can pick it.
  3. Add an `<option>` to the `#shape` select.
- **Controls**: Every way to launch goes through `launch(x, y)`, which reads the selected shape and pushes a `Rocket`. These include canvas `pointerdown`, the Space key, and the Launch, Show and Auto buttons. `randomLaunch()` picks a random target in the upper part of the screen.
- **Canvas scaling**: `resize()` sizes the canvas buffer by `devicePixelRatio` (capped at 2) and applies `setTransform`, so all drawing code uses CSS-pixel coordinates (`W`, `H`).
