---
layout: default
usemathjax: true
permalink: /render/ch3
---

# OpenGL Shader Language (GLSL)

Part of OpenGL since OpenGL 2.0 (i.e. available in compatibility profile)

OpenGL API introduces new functions to deal with shaders

- Compile, link, install, feed data
- Shaders can easily obtain OpenGL states

#### Primitives

- `float`
- `double`
- `int`
- `uint`
- `bool`
- `_vec#`
- `_mat#` (no integer matrices!)

![](/notes-blog/assets/img/render/glsl_types_mat.png)

#### Constructors 

- `vec3 rgb = vec3(vec4(1.0, 0.5, 0.5, 0.0))`
- `vec4 rgba = vec4(rgb, 1.0)`
- `vec3 white = vec3(1.0)`
- `mat3(4.0)` $ = \left[\begin{matrix} 4.0 & 0.0 & 0.0 \\ 0.0 & 4.0 & 0.0 \\ 0.0 & 0.0 & 4.0 \\ \end{matrix} \right]$​

Matrices are specified in **column-major order**.

![](/notes-blog/assets/img/render/row_col_major.png)

#### Accessors

| Component Accessors | Description         |
| ------------------- | ------------------- |
| `x,y,z,w`           | Positions           |
| `r,g,b,a`           | Colors              |
| `s,t,p,q`           | Texture coordinates |

```glsl
mat4 m = mat4(2.0);
vec4 zVec = m[2];
float yScale = m[1][1]; // or m[1].y
```

#### Swizzling

```glsl
vec3 red3 = color.rrr; // multiplying
color = color.abgr; // reordering
```

### Structures

```
struct Particle {
	float lifetime;
	vec3 position;
	vec3 velocity;
}
Particle p = (1.0, vec3(1.0, 0.0, 0.0), vec3(0.0, 1.0, 1.0));
pos = p.position;
```

