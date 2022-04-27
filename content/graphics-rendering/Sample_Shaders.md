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
