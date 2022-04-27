---
layout: default
usemathjax: true
permalink: /render/ch4
---

# Texture Mapping

Refer to [CS3241 Chapter 8](/notes-blog/cg#chapter-8--texture-mapping).

- Surface Parameterization
  - Mapping 3D coordinates of surface point to to 2D texture coordinates
- Inverse mapping by bilinear interpolation
- Texture filtering
  - Interpolated texture coordinate may not correspond to a texel. Then nearest 4 texels are interpolated.
- Wrapping
- Aliasing and Anti-aliasing
- Mipmapping and selecting mipmap level

# Application

- Modulation *(**Additive/Multiplicative** texture color with primary color, and secondary color added, **transparency**)*
- Environment mapping (reflection, refraction)
- Bump mapping
- Illumination mapping (pre-computed lighting via light maps)
- Shadow mapping
- Displacement mapping (vs bump mapping)
  - Gives real geometry modification to the base material
  - Geometry introduced after rasterization in fragment shader
  - Self-occlusion
  - Often needs some kind of raytracing on the displacement map
- Image-based rendering (e.g. billboarding)
- Non-photorealistic rendering

## Texture Mapping

```c++
void init(void) {
    glActiveTexture(GL_TEXTURE0);
    
    GLuint texObj;
    glGenTextures(1, &texObj); // what's 1??
    // bind the current Active 2D texture to texObj
    glBindTexture(GL_TEXTURE_2D, texObj);
    
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, REPEAT);
    // texture magnification ( texture coordinate not in 1 texel )
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    // texture minification ( anti-aliasing )
    // GL_LINEAR_MIPMAP_LINEAR is trilinear interpolation
    glTexImage2D(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR)
}
```

##  `texture` vs `texelFetch`

| `texture` | `texelFetch` |
| --------- | ------------ |
| handles filtering | no filtering, directly accesses a texel from image |
| via **texture** coordinates $\in(0,1)$ | via **texel** coordinates $(0, w \text{or} h)$ |

*Note on fragment position*: 
- A fragment's 2D position is coordinates of fragment's center in window space. 
  - bottom most (0.5, 0.5)
- A texel coordinate is integer values
  - bottom most (0, 0)

## GLSL

`sampler2D` for 2D texture maps. `samplerCube` for texture maps.

To sample from a texture, GLSL inbuilt function `texture`.

## Mipmap Level Computation

The appropriate mipmap level $L$, given

- $W, H$ are the width and height of the base mipmap
- $x_\text{win}, y_\text{win}$ are the window coordinates

Higher valued mipmap $\Leftrightarrow$ Smaller mipmap texture size

$$
L = \log_2 \left( \max\left( 

\sqrt{\frac{\partial sW}{\partial x_\text{win}}^2 + \frac{\partial tH}{\partial x_\text{win}}^2}, 

\sqrt{\frac{\partial sW}{\partial y_\text{win}}^2 + \frac{\partial tH}{\partial y_\text{win}}^2}

\right) \right)
$$

Each derivative represents how fast is the texture coordinate changing from pixel to pixel on screen (in x or y axis)?
