# wire_sphere — Project History & Handoff

## Origin

This project started inside a repository called `gfx_pipeline` (a general graphics
tooling repo) and was later extracted into its own repo because it grew into a
standalone interactive visualization tool with nothing to do with a pipeline.

The goal from the start was a glowing sphere made of layered orbital rings — a neon
wire-frame look on a black background, animating in real time in a browser tab.

---

## Development history

### Stage 1 — `proto_3d_base.html` / `proto_rings.html`
**The orbit-axis approach.**

The first two prototypes used a fundamentally different mental model for the rings: each
ring set was defined by a rotation axis in 3D space, and individual rings were generated
by rotating a perpendicular vector around that axis using Rodrigues' rotation formula.
Four ring sets were hard-coded:

| Set | Tilt X | Tilt Z | Color | Style |
|-----|--------|--------|-------|-------|
| 0 | 0° | 0° | Blue | solid |
| 1 | 47° | 22° | Green | solid |
| 2 | −47° | −22° | Red | solid |
| 3 | 20° | 55° | Yellow-green | dashed |

The camera orbited at a fixed distance (680 px) with a sine-wave elevation bob. Rings
were drawn as stroked paths with Canvas shadow glow.

`proto_rings.html` was a minor refactor of `proto_3d_base` — same structure, slightly
cleaned up axis calculation and render loop.

The main limitation of this approach: all rings shared the same radius (200 px), so the
sphere was flat/hollow rather than having visual depth.

### Stage 2 — `proto_touch.html` / `proto_touch_v2.html`
**Touch support + axis bug fix.**

These two builds kept the orbit-axis ring system but added interactive drag rotation
(mouse + touch). A bug in the axis vector formula from `proto_rings` was also corrected
here — the rotation axes weren't being computed correctly from the tilt angles.

`proto_touch_v2` switched from `addEventListener('touchstart', ...)` to direct
`ontouchstart` property assignment, added `touch-action:none` on the canvas element to
prevent the browser from intercepting touch scroll, and simplified the code slightly.

### Stage 3 — `rings_minimal.html`
**Complete redesign: latitude rings.**

This was the key architectural pivot. Instead of rotation-axis rings, the sphere is now
built from **latitude rings** — circles at evenly spaced polar angles across the sphere
surface, computed directly from spherical coordinates:

```
phi = π*(i + 0.5)/N − π/2   (latitude angle)
y   = R * sin(phi)
r   = R * cos(phi)           (ring radius at this latitude)
```

Each ring's 3D points are pre-computed once and stored. The render loop just projects
them each frame — much simpler and cheaper than the Rodrigues approach.

This build is single-shell, single-color (blue), 20 rings. Minimal and clean. It
established the foundation for everything that followed.

### Stage 4 — `rings_shells.html`
**Three concentric tilted shells — the "wire sphere" look.**

Built directly on `rings_minimal`, this build introduced two major ideas:

1. **Three shells** at radii 228, 238, 248 px (red, green, blue) giving visual depth and
   the layered neon appearance.
2. **Tilt** — the entire shell is rotated 39° by applying a rotation matrix to the y/z
   components of each ring point. This makes the rings slice diagonally instead of
   running horizontally, which is what gives the sphere its distinctive look.

Rendering switched from stroked lines to filled **dots** (small arcs), giving a softer,
particle-field appearance.

This is the visual design that the project settled on.

### Stage 5 — `orbital_sphere_sidebar_ui.html`
**UI panel added.**

Added a proper sidebar panel with sliders for the key parameters: camera distance, focal
length, ring count, segment count, dot size, tilt angle, animation speed. Layout used a
fixed-width right-side panel with the canvas filling the remaining space.

This build established the live-parameter editing workflow (sliders wired directly to
render params, no rebuild needed for most changes).

### Stage 6 — `orbital_sphere.html` (v4, current)
**Full feature build — star field, moons, floating overlay UI.**

Complete rewrite. Major additions:

**Star field**
Three concentric shells of stars at distances ~8 000–28 000 px. Each shell has
independently adjustable brightness (opacity) and point count. Stars provide parallax
depth — when the camera moves, near stars shift more than far stars, giving a sense of
scale. Points are distributed uniformly on a sphere using the `(θ, acos(2u−1))` method
to avoid polar clustering.

**Moon system**
Up to 8 moons orbiting the sphere on independently inclined orbital planes. Orbital
positions use proper Euler-angle mechanics (inclination + ascending node via the golden
angle `φ = 2.39996` rad for even distribution). Each moon is colored by hue, evenly
spread around the color wheel. Moon orbit trails (dot arcs) can be shown or hidden.
An equatorial-lock mode forces all orbits to the equatorial plane regardless of the
incline setting.

**Camera lock**
A checkbox stops the auto-rotation, letting you examine the sphere from a fixed angle.

**Overlay panel**
Switched from the sidebar layout to a floating semi-transparent overlay panel in the
top-right corner with a collapse toggle. This lets the canvas fill the full window.

**localStorage persistence**
Every slider value is saved to `localStorage` under `gfx_*` keys and restored on load.
The prefix was inherited from the original `gfx_pipeline` repo and left as-is.

**Rendering order**
Back-moons are drawn before the sphere dots; front-moons are drawn after. This gives
correct occlusion without a depth sort of individual points.

---

## Key technical decisions

**Why dots instead of lines?**
Stroked lines create visual artifacts when rings pass edge-on to the camera (the line
collapses to a point). Dots are consistent at any angle and give a softer, more
particle-like look.

**Why pre-compute ring points?**
Ring geometry only changes when parameters change (via sliders). Pre-computing into
arrays and just projecting each frame is much faster than recomputing trig every frame.

**Why three shells instead of one?**
A single shell reads as flat. Three shells at slightly different radii give the sphere
visual thickness and the layered color effect that defines the look.

**Why 39° tilt?**
Empirically chosen — it's far enough from 0° (equatorial) and 90° (meridional) to look
dynamic, and produces an appealing crossing pattern when all three shells are shown.

---

## What's not here

- No build system, no bundler, no dependencies
- No server required — open the HTML file directly
- `mov2gif.py` (a screen-capture-to-gif utility) was left behind in `gfx_pipeline`
  and is not part of this project
