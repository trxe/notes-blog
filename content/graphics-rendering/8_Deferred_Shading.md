---
layout: default
usemathjax: true
permalink: /render/ch8
---

# Deferred Shading

Techniques to avoid applying time-consuming shaders not in final image.

1. Pass #1: Simple and fast shader renders into off-screen buffer (+ depth buffering)
    - Data for shading computation in G-buffer (**geometric** buffer)
    - e.g. Fragment position, normal, surface material
2. Pass #2: Screen filling quad + frag shader using G-buffer elements

> Q: How is time saved? 
> 
> A: In the final pass, we only do expensive lighting computation only for pixels viewable
> in the final frame 

**Advantage**:
- Whatever fragment ends up in the G-buffer is the fragment that will be seen in the final render.
- **Useful** if high depth complexity and scene fills almost whole viewport i.e. many overdrawn fragments.
- Lighting is calculated only once per pixel.

**Disadvantage**:
- **Memory** required is very high, so try to pack some data into Textures, or reduce precision.

*Note*: `flat` indicates a variable to be passed to rasterizer should **not** be interpolated

![Deferred Overview](/notes-blog/assets/img/render/deferred_overview.png)

## Data packing precisions

Some data and the space required:
- Fragment's world space position (2 x 32b FP)
- Frag's world space normal (3 x 16b FP)
- Frag's albedo/diffuse color (3 x 16b FP)
- Frag's object/material index (32b int)
- Frag's specular power factor (32b FP)

Q: Why vertex normal is less precise than vertex world space position?

A: Reduced precision in world space precision causes **jagged edge** artifacts
since there are fewer depth values, which reduces the number of fragments and 
makes the produced image blocky.
Also leads to worse **z-fighting** as the likelihood of z-values colliding is higher.

### Z-fighting

When two fragments have identical z-buffer values $\in [0, 1]$ and competing fragments are randomly discarded.
- Near and far plane greater $\Rightarrow$ worsen z-fighting as less precision is given to the fraction component.
- Lower precision, worse z-fighting

Example of linear depth buffering:

$$
z_\text{depth} = \frac{z - \text{near}}{\text{far} - \text{near}}
$$

Non-linear depth-buffering:

$$
z_\text{depth} = \frac{1/\text{near} - 1/z}{1/\text{near} - 1/\text{far}}
$$

# Screen Space Technique: Ambient Occlusion

Ambient lighting in Phong lighting model is flat and unrealistic on non-illuminated surfaces.

Ambient **occlusion** simulates illuminating diffuse surfaces by a **uniform hemisphere area light source**.

![](/notes-blog/assets/img/render/ambient_occlusion.png)

For each point, determine whether that point is visible from **what proportion of viewpoints around the center** 
i.e. this **accessibility factor**, which modulates the direct diffuse illumination vector. 

In practice, this computation can be done at every **vertex**:

```glsl
// Accessibility factor computed by raytracing from each vertex and testing for occlusion 
// This does NOT work for dynamic scenes.
layout (location = 2) in float Accessbility;
...
v2fColor = Accessibility * MatlAmbient + LightDiffuse * MatlDiffuse * N_dot_L;
```

Limitations of pre-computed AO:

1. No dynamic scene support
2. Expensive to compute
3. Inconvenient to compute in 3D object space, done mostly on CPU, and hard to move to GPU

We can better harness GPU computation by **approximating** the solution by computing per-pixel accessibility in screen space.

### Approach

1. Render a depth map (PASS 1)
   1. Texture 1: **Direct diffuse illuminated color**
   2. Texture 2: View-space **normal** (xyz), View-space **z-coord** (w)
2. Per fragment, use depth map's geometry info to determine visibility
   1. Shoot multiple rays from frag's eye space
   2. If ray intersect depth map, it is occluded
   3. Is any point along the ray behind the depth map?
3. Get `Accessibility` as the proportion of unoccluded rays.

### Ray marching

```glsl
vec4 NZ = texelFetch(sNormalEyeZ, ivec2(gl_FragCoord.xy), 0);
vec3 N = NZ.xyz;
vec3 P = vec3(gl_FragCoord.xy, NZ.w); // w is the z-coordinate
uint occ = 0;

for (uint i = 0; i < numRays; i++) {
   // uniform random_dir passed in from app
   vec3 dir = random_dir[i].xyz;
   // make hemispherical
   if (dot(N, dir) < 0.0) dir = -dir;
   // March along the ray
   for (uint j = 0; j < numSteps; j++) {
      // march one more step from P
      vec3 Q = P + dir * (j+1) * stepSize;
      thatEyeZ = texelFetch(sNormalEyeZ, ivec2(Q.xy), 0).w;
      // if Q is deeper than the z-coord at it's world pos, then it is occluded.
      if (Q.z < thatEyeZ) {
         occ += 1.0;
         // stop marching this way
         break;
      }
   }
}

float ao_amt = float(numRays - occ) / float(numRays);

vec4 object_color = texelFetch(sColor, ivec2(gl_FragCoord.xy), 0);
// ssao_level * vec4(0.2) + (1-ssao_level) * ao_amt
FragColor = object_level * object_color + mix(vec4(0.2), vec4(ao_amt), ssao_level);
```