---
layout: default
usemathjax: true
permalink: /render/ch11
---

# Global Illumination

2 forms of rendering equations: **Hemispherical** and **Area** formulations.

Hemispherical: Integrates incoming radiance from the hemisphere around point $x$.

Area: Integrates incoming radiance from **all surfaces** in the scene. (Involved in radiosity computation).

Involves sending many rays, so it cannot be computed analytically. 
Instead can use iterative methods such as MCRT below.

| Partial Global Illumination | Global Illumination |
| --------------------------- | ------------------- |
| Whitted Raytracing `LD?S*E` | Path tracing `L(D|S)*E` | 
| Distribution Raytracing `LD?S*E` | Two-pass ray tracing `LS+DS*E` |
| Radiosity `LD*E` | Photon-mapping `LS+DS*E` |

## Local Model: Whitted Ray tracing

![](/notes-blog/assets/img/render/whittedrt.png)

## Monte Carlo Raytracing

Monte Carlo techniques for solving integrals without analytic solution.

Algorithms are:
- View dependent
- Recursive
- Includes:
  - Distribution raytracing
  - Path tracing 
  - Two-pass ray tracing 
  - Photon mapping

Generates **random samples** of integrand and averages the result.

$$
\begin{aligned}
I &= \int_a^b f(x) dx \\
&\approx \frac{b-a}{N} \sum^N_{i=1} f(\xi _i) \\
&= I_m
\end{aligned}
$$

Note that $\lim_{N \rightarrow \infty} I_m = I$.

For $N$ uniformly distributed random samples, 

$$
\left| I_m - I \right| = \frac{k}{\sqrt{n}}
$$

### Stratified sampling

Sample once per stratum (an equally sized subregion).

Error reduced to $k/N$.

![](/notes-blog/assets/img/render/stratified_sampling.png)

### Importance sampling

Sampling values with higher probabilities.

Works with knowledge of material's BRDFs.

![](/notes-blog/assets/img/render/impt_sampling.png)

Problem: Hard to know the sampling distribution (even with BRDF) beforehand (e.g. **caustics**).

## Distribution Ray Tracing `LD?S*E`

See previous chapter.

Same as Whitted ray tracing except all **specular paths** are calculated.

View dependent, partial global illumination.

1. Area lights, soft shadows
2. Blurred reflections/refractions (glossy)
3. Anti-aliasing
4. Depth of Field
5. Motion Blur

![](/notes-blog/assets/img/render/distrt.png)

## Path tracing `L(D|S)*E`

- At each intersection, a **random ray** (either transmission or reflection, doesn't have to be in specular lobe) is shot.
- Termination with probability $M$ (material absorption) at every intersection.

View dependent, complete global illumination.

**High computational cost**: Needs to shoot many paths per pixel to reduce **noise**.

![](/notes-blog/assets/img/render/pathrt.png)

![](/notes-blog/assets/img/render/pathalgo.png)

### Examples of Noise

This happens due to lack of control over in which direction does the traced reflection/refraction ray shoot toward.

![](/notes-blog/assets/img/render/pathalgo.png)

## Two pass ray tracing `LS*DS*E` 

Rays shot from light source (light "view") and stored in lightmap, requires surface parameterization.

1. **Light pass**: LS*D [View independent, saves results to **light map**]
2. **Eye pass**: D?S*E [View dependent, uses results from light map]

![](/notes-blog/assets/img/render/twopassrt.png)

## Photon mapping `LS*DS*E` 

Photons shot from light source (light "view") and stored in lightmap.

### Difference from 2-pass raytracing: Data structures

Key-value pair: Photon map values, unlike in light map, are stored with respect to a **3D position vector**. Includes
- Photon **incoming direction**
- Photon **power**

Data structure is **kd-tree**:
Efficient for querying $k$ closest photons about any 3D locaiton (for estimating outgoing radiance per surface point)

### Maps

1. Normal photon map
   1. Light source shoots photons
   2. Intersection with diffuse surface $\Rightarrow$ store value
   3. Randomly **reflect, transmit or absorb** [Russian Roulette, re-emission direction sampled from BRDF].
2. Caustics photon map
   1. Light source shoots photons
   2. Intersection with diffuse surface $\Rightarrow$ store value **and terminate**
3. Trace ray from eye and compute **reflected radiance**

### Reflected radiance

| Rendering value | Explanation | Example |
| --------------- | ----------- | ------- |
| Direct illumination | Shadow ray | <img src="/notes-blog/assets/img/cg/photon_direct.png" width="50%"> |
| Specular reflection | Specular/glossy reflections | <img src="/notes-blog/assets/img/cg/photon_specular.png" width="50%"> |
| Caustics | Caustics photon map | <img src="/notes-blog/assets/img/cg/photon_caustics.png" width="50%"> |
| Indirect illumination | Tracing secondary rays to photon map | <img src="/notes-blog/assets/img/cg/photon_indirect.png" width="50%"> |