# wire_sphere

An interactive 3D wire-sphere visualization that runs in any browser with no dependencies.

## What it is

A single self-contained HTML file (`orbital_sphere.html`) that renders a glowing sphere
built from three concentric tilted shells of latitude rings, surrounded by a parallax star
field and orbiting moons. Everything is drawn on an HTML5 Canvas using vanilla JavaScript.

## How the visuals work

### The sphere
Three shells sit at slightly different radii (default: 228, 238, 248 px). Each shell is a
set of latitude rings computed from spherical coordinates, then rotated by a tilt angle
(default 39°) so the rings slice diagonally across the sphere rather than running
horizontally. The three shells are colored red, green, and blue, giving the layered neon look.

Each ring is stored as an array of 3D points. On every frame the renderer projects all
points through a perspective camera to 2D canvas coordinates, then draws filled dots at
each point with a glow shadow.

### The camera
The camera orbits the sphere automatically by advancing an azimuth angle each frame. A
slow sine wave on the elevation angle causes the camera to gently bob above and below the
equator. The user can drag (mouse or touch) to spin the view manually, or lock the camera
in place. Perspective projection (configurable focal length) is used — nearer points
appear larger.

### Star field
Three concentric spherical shells of stars at distances ~8 000–28 000 px provide
parallax depth. Each shell has independently adjustable brightness and count.

### Moons
Up to 8 moons orbit the sphere on configurable inclined orbital planes using proper
Euler-angle orbital mechanics. Each moon is colored by hue, spaced evenly. Orbit dot
trails and an equatorial-lock mode are available.

## File structure

```
wire_sphere/
  orbital_sphere.html   ← main visualization (v4)
  orbital_qr.png        ← QR code linking to the live file
  PLAN.md               ← this file
  HANDOFF.md            ← project history
  archive/
    rings_minimal.html            ← latitude-ring prototype (single shell)
    rings_shells.html             ← three-shell dot renderer (key breakthrough)
    orbital_sphere_sidebar_ui.html ← sidebar UI experiment
    proto_3d_base.html            ← original orbit-axis ring engine
    proto_rings.html              ← orbit-axis refactor
    proto_touch.html              ← added touch support
    proto_touch_v2.html           ← simplified touch handling
```

## Controls (overlay panel, top-right)

| Control | What it does |
|---|---|
| star field | toggle star layer visibility |
| stars far/mid/near bright | brightness per star layer |
| stars far/mid/near count | point count per layer |
| orbit lines | toggle moon orbit dot trails |
| equatorial orbits | force all moon orbits to the equatorial plane |
| lock camera | pause auto-rotation |
| zoom (cam dist) | distance of camera from origin |
| focal len (mm) | perspective strength |
| inner radius | radius of innermost shell |
| shell gap | spacing between the three shells |
| rings | latitude rings per shell |
| dots/ring | segments per ring (point count) |
| dot size | rendered dot radius in px |
| tilt (deg) | shell rotation angle (0 = equatorial rings, 90 = meridian rings) |
| speed | camera auto-rotation speed |
| moon count | number of orbiting moons (0–8) |
| moon size / orbit r / incline / speed / brightness | per-moon properties |
| incline spread / orbit var / size var | randomization ranges for moon variation |
| orbit dots | dot count for moon orbit trails |

All slider values persist across page loads via `localStorage`.
