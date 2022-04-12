---
layout: default
usemathjax: true
permalink: /render/ch10
---

# Photorealistic rendering

Inputs (camera, geometry, lighting, **materials**) must all be accurate and detailed enough to support.

## Material Appearances

- Reflection: BRDF (Bidirectional Reflection Distribution Function)
- Transmission: BTDF (Bidirectional Transmission Distribution Function)

Reflection by local reflection models such as
- Phong (empirical, not physically based)
- Cook-Torrance (physically based)

Local reflection models: rs between 
1. One light source
2. A single surface point
3. The view point
4. No interactions between other objects.

**Global** illumination considers **all** light sources and surfaces, inter-reflections and shadows.

## Radiometry

Radiometry: Physical measurement of light/radiation energy

Radiant flux/power $Phi$:
- **Total** energy flowing from/to/through surface per unit time
- Light power (Watts)

Irradiance $E$:
- **Incoming** (Incident) radiant power on surface per unit surface area
- W/$m^2$
- $E= \frac{d\Phi}{dA}$

Radiant exitance/radiosity $M$, or $B$ (surface that's totally diffuse):
- **Outgoing** (Exitance) radiant power on surface per unit surface area
- W/$m^2$
- $M$ (radiant exitance, more general), $B$ radiosity.

**Radiance** $L$:
- "Intensity" of light
- Radiant power per unit **projected surface area** per unit **solid angle**
  - Solid angle: The apparent size of the light source from the center view point.
  - Solid angle: (in steradians) the Sphere surface area intercepted by a cone, divided by square of the sphere's radius $r$.
  - Solid angle: $2\pi r^2 = 2\pi$
  - Projected surface area: From an oblique projection angle, the surface area that the light hits changes
  - Projected surface area: surface area $\times \cos \theta$
  - The reason being we don't want the size of the light source (solid angle) and direction of the light (projected surface area) to affect the computation of radiance.
- $W/(srm^2)$

$$
L = \frac{d^2 \Phi}{d\omega \cdot dA cos\theta}
$$

![](/notes-blog/assets/img/render/radiance.png)