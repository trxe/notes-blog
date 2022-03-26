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

 ## Ray Tracing Acceleration
 Research mostly within
 - Accelerating ray-scene intersection
 - Extension to simulate better global illumination
 - Real Time raytracing

### Adaptive recursion depth control
Not very effective.
Modifies the allowed recursion depth based on pixel.

### First hit speed up using z-buffer
Also not very effective.
Uses item buffer to identify first-hit object at each pixel.

### Bounding volumes
Replaces complex object ray-intersection computation with simple object ray-intersection cocmputation.
Ray-intersection with each object's bounding volume; when no intersection, discard this object altogether.

Examples of bounding volumes:
- Spheres
- AABB (most common)
- OBB (oriented bounding boxes, slower than AABB but more precise)
Tighter box: Less likelihood of false positive (intersection when there isn't), 
but less efficient to compute.

### Bounding volume hierarchy
First compute intersection with root bounding box ($a$). 
Then compute for the next bounding box containing that object (e.g. $c$).
Then compute for the next bounding box containing that object (e.g. $c_3$).
Keep computing until discard or lighting computation.

Usually good hierarchies are constructed manually.

### Spatial subdivision

Subdivision of 3D space into regions
Each regions has a list of objects in it.
When ray traced into region, **query the object list** and **perform intersection tests** 
on those objects.
Since we are looking for the nearest intersection, the ray should be traced from a **front-to-back order** through the regions.

#### Uniform Grid
- Memory inefficient and non-adaptive
  - Allocation of regions for empty space in an non-uniformly distributed map.

### Octree
- Each cubic region $\rightarrow$ 8 equal subregions
  - recursively
  - *conditionally*

Conditional schemes:
- subdivide if it is occupied by $> 1$ object
- subdivide if it is occupied by *any* object until max allowable depth.

## Distribution Ray Tracing

![](/notes-blog/assets/img/render/distribution.png)

| Whitted | Distribution |
| ------- | ------------ |
| hard shadows | area lights, soft shadows |
| blur specular highlight (phong computation), with sharp reflections | blurred reflections and refractions both |
| aliasing | anti-aliasing |
| no caustics (focusing of light on surface) | NO caustics |
| no color bleeding of diffuse surfaces | NO color bleeding |

Distribution ray tracing also boasts depth of field and motion blur.

![](/notes-blog/assets/img/render/dof.png)

Per **pixel**: shoot out random multiple primary rays, whose fetched colors are averaged.
Per **intersection**: *reflection, refraction, shadow* rays are randomly **perturbed**.
- Pick a random point in area light source as a **point** light source (shadow ray)
  - Using random sampling in the stratified area (jittered sampling)
- Perturb the reflection ray by a random amount (in the gray bubble)

### Raytracing on GPU
- Per frame, draw fullscreen quad (to generate all fragments)
- Each fragment shader invocation traces a primary ray to intersect the scene
- GLSL does not allow recursion:
  - manually unroll the recursion into multiple copies of the function
  - Use an array of FBOs as stack.