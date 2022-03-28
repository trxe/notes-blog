---
layout: default
usemathjax: true
permalink: /3d/ch5
---

# Particle systems

Particles are objects 
- with **position** and **velocity**,
- without **shape** or **orientation**,
- that may have mass or may respond to forces.

Particle Systems consist of 
- many particles, or
- many particle systems (hierarchically organized)

Basic particle systems:
- A collection of many minute particles that together form a fuzzy object
  - Points to define shapes rather than polygons
- Procedural/Statistical
- Stand alone

## Particle lifetime

1. Seeding
2. Attribute assignment
3. Animation

## Seeding particles

Creation/Deletion of particles.

Location:
- Randomly within volume/surface
- At point of event
- In regions sparsely populated

When:
- at start
- per frame
- on event (e.g. particle death)

## Attributes

- Position
- Velocity
- Color
- Age
- Mass
- Radius
- Temperature
- Density
- *and so on.*

## Animation

Types of rules that govern motion:
- Procedural (follow specification/algorithm)
- Statistical (to add variation)
- Physics-based collision/dynamics

Update cycle:
1. Clear forces
2. Calculate and accumulate forces
3. Solve for accelerations
4. Integrate velocity/position
5. Record state
6. Update other variables

# Particle Physics

## Newtonian Particle

Each particle is an **ideal point mass**.

Six degrees of freedom: 3 position coords, 3 velocity coords.

Newton's second law: $F = ma$
- $f = mg$ for gravity
- $f = -k_d v$ for viscous drag
- $f = k_s (|x| -r ) - k_d v$ for Hooke's Law
  - first term: stiffness
  - second term: damping factor (viscous drag)
  - Modelling particles as springs.
  - Virtual springs between particles

Forces: depend on position velocity time

Acceleration (2nd derivative) $x'' = f(x, x', t) / m$ can be:
- constant (e.g. gravity)
- position/time dependent (force fields/collisions)
- velocity dependent (drag/damping)
- n-ary (many springs: make more stiff/flexible)
  - mass spring models

**Vector field** of velocities/accelerations (depends on what this field represents)

![Vector field](/notes-blog/assets/img/3d/vector_field.png)

Differential equations to solve **initial value problems**.

### Euler's method

Approximation. Break into discrete time steps.
Choie of $\Delta t$ (time interval) is thus critical.
Wrong choice can make movement **inaccurate** and **unstable**.

Velocity-time: $v_t = x'_t = f(x_t, t)$.

Position-time: $x_{t+1} = x_t + \Delta t v_t$.

**Midpoint method** is a modification of Euler's method.
1. $\Delta x = \Delta t f(x, t)$
2. $v_\text{mid} = f(x + \frac{\Delta x}{2}, t + \frac{\Delta t}{2})$
3. $x_{t + 1} = x_t + \Delta t v_\text{mid}$

Use the midpoint velocity to derive the next step's position, instead of the velocity at the start of the interval.

## Advanced Particle Systems

1. Mass Spring systems
2. Smoothed particle hydrodynamics (SPH)
3. Crowd simulation (human crowds)

### Mass spring systems

Hooke's law: $f_s = k_s(x - r) - k_d v$

- 1D line of springs (hair)
- 2D (cloth)
  - Triangulated particles (see below)
- 3D (jello)
  - More sophisticated methods FEMs (Finite elemental methods)

Right layout, stiffness, integration.

Layout of 2D mass spring particles in cloth:

![Cloth layout](/notes-blog/assets/img/3d/cloth.png)

$$ x'' = M^{-1} (- \frac{\delta})$$