---
layout: default
usemathjax: true
permalink: /render/ch9
---

# Raytracing

Most info can be found at [CS3241 Chapter 9](/notes-blog/cg#chapter-9--ray-casting--tracing).

Whitted ray tracing:

$$
I = I_\text{local} + k_{rg}I_\text{reflection} + k_{tg}I_\text{transmitted}
$$

Lighting computation:

$$
I_\text{local} = I_ak_a + k_\text{shadow}I_\text{source}[k_d(N \cdot L) + k_r(R \cdot V)^n + k_t(T \cdot V)^m]
$$

Scene description in raytracing:
- Camera view & image resolution: `gluLookAt`
- Field of view: `gluPerspective` but no near/far plane
- Image resolution: pixels per unit
- Each point/spot light source
  - Position
  - $I_source$ RGB values
  - $I_a$ RGB ambient color
- Each object surface material
  - $k_{rg}, k_{tg}, k_a, k_d, k_r, k_t$
    - Note usually global $r, t$ are the same as local $r, t$ values.
    - If $k_{rg} = 0$, not reflective.
    - If $k_{tg} = 0$, not transmittive.
    - If both $0$, completely diffuse.
  - Refractice  index $\mu$ if $k_{tg} \neq 0$ or $k_t \neq 0$.
- **Objects**
  - Implicit representation (plane, sphere or quadrics)
  - Polygon
  - Parametric/Volumetric

 