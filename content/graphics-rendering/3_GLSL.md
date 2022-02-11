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

```glsl
#version 450 core /* version number profile */
```

## Syntax

### Primitives

- `float`
- `double`
- `int`
- `uint`
- `bool`
- `_vec#` (`dvec, ivec, uvec, bvec`. by default `vec` is for floats)
- `_mat#` (only `dmat`)

![](/notes-blog/assets/img/render/glsl_types_mat.png)

### Constructors 

- `vec3 rgb = vec3(vec4(1.0, 0.5, 0.5, 0.0))`
- `vec4 rgba = vec4(rgb, 1.0)`
- `vec3 white = vec3(1.0)`
- `mat3(4.0)` $ = \left[\begin{matrix} 4.0 & 0.0 & 0.0 \\ 0.0 & 4.0 & 0.0 \\ 0.0 & 0.0 & 4.0 \\ \end{matrix} \right]$​

Matrices are specified in **column-major order**.

![](/notes-blog/assets/img/render/row_col_major.png)

### Accessors

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

### Swizzling

```glsl
vec3 red3 = color.rrr; // multiplying
color = color.abgr; // reordering
```

### Arrays

```glsl
float coeff[3] = float[3](2.38, 3.14, 0.0);

for (int i = 0; i < coeff.length(); i++)
    coeff[i] *= 2.0;
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

## Storage Qualifiers

- `const` read only variable
- `in`, `out` 
- `uniform`value is passed to shader from application and is constant across a single primitive (e.g. one triangle)
- `buffer` read-write memory shared with application
- `shared` (compute shaders only)

## Overloaded Operators

important: matrix vector multiplication

```glsl
vec3 v, u;
mat3 m;

u = v * m; // u.x = dot(v, m[0]), u.y = dot(v, m[1]), u.z = dot(v, m[2])
mat3 n;
n = m * v; // usual Mv matrix * vector
```

## Control flow

- if-else, switch
- for, while, do-while
- break, continue, return
- `discard`: discards a **fragment**

No recursion allowed.

## Return Types

- void
- GLSL data type
- struct
- array (size must be explicitly specified)

GLSL has no concept of pointer/reference. Functions are thus called by value-return (copied in/out, read-only/read-write).

## Built-in Variables

### Vertex Shader

```glsl
in int gl_VertexID;
in int gl_InstanceID;

out gl_PerVertex { // this is not a struct!
	vec4 gl_Position; // most important. is the *clip space position*
    float gl_PointSize;
    float gl_ClipDistance;
}
```

### Fragment Shader

```glsl
in vec4 gl_FragCoord; // window space coordinates
// z is the z-depth value.
// w is -z_e (z_e is the z coordinate of the fragment in eye space)
in bool gl_FrontFacing;

out float gl_FragDepth; // default: gl_FragCoord.z
```

