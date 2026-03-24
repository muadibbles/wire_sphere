# Moon Orbital Mechanics Reference

Reference notes for the moon system in `orbital_sphere.html`. Covers the physics
behind realistic multi-moon visualization.

---

## 1. The Six Orbital Elements

Every orbit is fully specified by six numbers at a given epoch.

| Element | Symbol | Meaning |
|---------|--------|---------|
| Semi-major axis | a | Half the long axis of the ellipse; determines orbit size |
| Eccentricity | e | Shape: 0 = circle, approaching 1 = very elongated |
| Inclination | i | Tilt of orbital plane relative to planet's equator |
| Longitude of ascending node | Ω | Rotates the tilted plane around the z-axis |
| Argument of periapsis | ω | Rotates the ellipse within the plane |
| Mean anomaly | M | Time parameter — increases linearly with time |

**Periapsis/apoapsis distances:**
```
r_periapsis = a(1 - e)
r_apoapsis  = a(1 + e)
```

**Inclination convention:**
- i < 90° → prograde (same direction as planet's spin)
- i > 90° → retrograde (opposite direction; captured objects)

---

## 2. Kepler's Laws

**First law:** Orbits are ellipses; the planet sits at one focus.

**Second law:** The planet–moon line sweeps equal areas in equal times.
- Consequence: the moon moves faster near periapsis and slower near apoapsis.
- A constant angular-velocity approximation is wrong for e > ~0.05.

**Third law:** T² ∝ a³

Full form:
```
T² = (4π² / GM) × a³
n  = 2π / T = √(GM / a³)    (mean motion, radians/s)
```

For a multi-moon system: if one moon is at distance a and another at 2a, the outer
one has period T × 2^1.5 ≈ 2.83 × T. Using a constant speed multiplier per moon
ignores this and produces unrealistic relative motion.

---

## 3. Computing 3D Position from Orbital Elements

The full pipeline per frame, per moon:

### Step 1 — Mean anomaly at time t
```
M(t) = M₀ + n · (t − t₀)     (mod 2π)
```

### Step 2 — Eccentric anomaly E (Kepler's equation)
```
M = E − e · sin(E)
```
This has no closed-form solution. Solve with Newton–Raphson:
```javascript
let E = M;
for (let i = 0; i < 6; i++) {
    E -= (E - e * Math.sin(E) - M) / (1 - e * Math.cos(E));
}
```
Converges in 3–5 iterations for any physically reasonable eccentricity.

### Step 3 — True anomaly ν
```javascript
const nu = 2 * Math.atan2(
    Math.sqrt(1 + e) * Math.sin(E / 2),
    Math.sqrt(1 - e) * Math.cos(E / 2)
);
```

### Step 4 — Position in orbital plane
```javascript
const r   = a * (1 - e * Math.cos(E));
const xOrb = r * Math.cos(nu);
const yOrb = r * Math.sin(nu);
```

### Step 5 — Rotate to reference frame
Three successive rotations: ω (around z), then i (around x), then Ω (around z):
```javascript
// Rotate by ω
let x1 =  xOrb * cos(ω) - yOrb * sin(ω);
let y1 =  xOrb * sin(ω) + yOrb * cos(ω);

// Rotate by i
let x2 = x1;
let y2 = y1 * cos(i);
let z2 = y1 * sin(i);

// Rotate by Ω
let x = x2 * cos(Ω) - y2 * sin(Ω);
let y = x2 * sin(Ω) + y2 * cos(Ω);
let z = z2;
```

---

## 4. Hill Sphere

The maximum distance at which a moon can hold its own satellites (or, for a
planet, the region where moons can orbit stably):

```
r_hill ≈ a · (m / 3M)^(1/3)

a = moon's semi-major axis around planet
m = moon's mass
M = planet's mass
```

Real examples:
- Earth's Moon: r_hill ≈ 60,000 km (no stable sub-moons)
- Phobos (Mars): r_hill ≈ 200 km (negligible)
- Titan (Saturn): r_hill ≈ 52,000 km

For a visualization with user-adjustable moon counts and distances, this sets a
natural upper bound on how far out a moon can be placed without being torn away.

---

## 5. Orbital Resonance

Resonance occurs when orbital periods are related by small integer ratios. Gravity
then applies periodic tugs in the same direction, locking the configuration.

**Io–Europa–Ganymede (Jupiter), 1:2:4 Laplace resonance:**
- Io:       T ≈ 1.77 days
- Europa:   T ≈ 3.55 days  (2 × Io)
- Ganymede: T ≈ 7.16 days  (4 × Io)

The resonance keeps Io's orbit slightly eccentric, which drives constant tidal
flexing and makes Io the most volcanically active body in the Solar System.

**Neptune–Pluto, 2:3 resonance:**
Pluto completes 2 orbits for every 3 of Neptune's. Despite crossing Neptune's
orbital path, Pluto and Neptune never come close to each other.

Visualization note: in a resonant system, moons return to the same relative
configuration periodically. You can show this with orbit-phase markers.

---

## 6. Retrograde vs Prograde

| | Prograde | Retrograde |
|--|---------|-----------|
| Inclination | i < 90° | i > 90° |
| Origin | formed with planet | usually captured |
| Tidal effect | slow outward drift | slow inward spiral |
| Examples | Moon, Galilean moons | Triton, Phoebe |

**Triton (Neptune):** i ≈ 157°, nearly circular (e ≈ 0), but spiraling inward
due to tidal braking. Expected to reach Neptune's Roche limit in ~3.6 Gyr and
break apart into a ring.

---

## 7. Notable Real Moon Behaviors

### Tidal lock (synchronous rotation)
All large moons in the Solar System have rotation period = orbital period. The
same face always points toward the planet.

Visualization: spin rate = orbital angular velocity (n radians/s).

### Phobos — fast orbital decay
Phobos (Mars) is only ~9,400 km from Mars center and decays ~1.8 cm/year.
Already within Mars' eventual Roche limit; will be torn apart in ~50 Myr.

Visualization: if simulating long timescales, Phobos-like moons should visibly
spiral inward.

### Hyperion — chaotic tumbling
Highly irregular shape + Saturn's tidal torques = no stable rotation period.
Hyperion tumbles chaotically; its orientation is unpredictable over timescales of
even a few weeks.

Visualization: randomize spin angle per frame (or use a simple chaos map) rather
than a smooth spin rate.

### Enceladus — tidal heating + geysers
Orbital resonance with Dione (2:1) keeps eccentricity nonzero → tidal heating →
subsurface liquid ocean → water geysers at south pole.

---

## 8. What the Current Code Does vs Full Mechanics

The current `orbital_sphere.html` moon system:
- Uses inclination (i) and ascending node (Ω) correctly for orientation
- Uses a fixed angular speed per moon (uniform circular motion)

What it does not do:
- Enforce T² ∝ a³ (moons at different radii could have Kepler-correct periods)
- Apply eccentricity (orbits are circular; no periapsis/apoapsis variation)
- Solve Kepler's equation (because e = 0, it isn't needed yet)

The simplification is intentional and fine for the current aesthetic goal. If
eccentricity or physically correct relative speeds are added later, the Step 2–4
pipeline above is the drop-in replacement for the angular-position computation.

---

## Quick-Reference Formulas

```
T² = (4π²/GM) × a³               Period from semi-major axis
n  = 2π / T                       Mean motion (rad/s)
M  = M₀ + n·t                     Mean anomaly at time t
M  = E − e·sin(E)                 Kepler's equation (solve for E)
ν  = 2·atan2(√(1+e)·sin(E/2),    True anomaly from E
             √(1−e)·cos(E/2))
r  = a·(1 − e·cos(E))             Distance from focus
r_hill ≈ a·(m/3M)^(1/3)           Hill sphere radius
```
