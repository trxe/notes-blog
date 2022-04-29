---
layout: default
usemathjax: true
permalink: /render/shaders
---

# Shaders

Collection of shader snippets with their techniques summarized.

## Perturb vector

**Fragment** Shader below.

```glsl
void main() {
    // Set up the tangent frame
    vec3 N = normalize(ecNormal);
    vec3 B = normalize(cross(N, ecTangent));
    vec3 T = cross(B, N);

    // ... Other computations ...

    // Compute a perturbed normal.
    // T = x-axis, B = y-axis, N = z-axis
    vec3 tanPerturbedNormal = normalize(vec3(T_offset, B_offset, N_offset));
    vec3 ecPerturbedNormal = tanPerturbedNormal.x * T
                            + tanPerturbedNormal.y * B
                            + tanPerturbedNormal.z * N;

    // Use eye space perturbed normal.
}
```

## Normal Computation without Precomputed Tangent

*To figure out*

```glsl
void compute_tangent_vectors( vec3 N, vec3 p, vec2 uv, out vec3 T, out vec3 B )
{
    // get edge vectors of the pixel triangle
    vec3 dp1 = dFdx( p );
    vec3 dp2 = dFdy( p );
    vec2 duv1 = dFdx( uv );
    vec2 duv2 = dFdy( uv );

    // solve the linear system
    vec3 dp2perp = cross( dp2, N );
    vec3 dp1perp = cross( N, dp1 );
    T = normalize( dp2perp * duv1.x + dp1perp * duv2.x );  // Tangent vector
    B = normalize( dp2perp * duv1.y + dp1perp * duv2.y );  // Binormal vector
}
```

## Procedural Texture Generation

**Given**:
- `texCoord`
- A tile density value
- Other information for texture generation

**Fragment** Shader below.

```glsl
in vec2 texCoord;
uniform float TileDensity;

void main() {
    // To shrink texture size by 1/TileDensity.
    vec2 c = TileDensity * texCoord;
    vec2 adjustedTexCoord = fract(c) - vec2(0.5);
    // ...
}
```
## Screen-space techniques

```
ivec2 pix = ivec2(gl_FragCoord.xy);
// Weight[0] is the center value of the kernel
vec4 sum = texelFetch(RenderTex, pix, 0) * Weight[0];

// Example kernel filtering
for (int i = 1; i < 5; i++)  {
    sum += texelFetchOffset(RenderTex, pix, 0, ivec2(0, PixOffset[i])) * Weight[i];
    sum += texelFetchOffset(RenderTex, pix, 0, ivec2(0, -PixOffset[i])) * Weight[i];
}
FragColor = sum;
```

## Multipass Shader

```glsl
// ...
uniform int RenderPass;

void main() {
    if (RenderPass == 0) {
        pass1();
    } else {
        pass2();
    }
}
```

How to **pass uniforms out** of shader to save to texture:

How to **render final image**: pass in textures as uniforms.

```
// pass 1: saving to texture
layout (location = 0) out vec4 FragColorA;
layout (location = 1) out vec4 FragColorB;

// pass 2: rendering final image
uniform sampler2D textureA;
uniform sampler2D textureB;

void main() {
    ...
    FragColorA = ...;
    FragColorB = ...;
}
```
