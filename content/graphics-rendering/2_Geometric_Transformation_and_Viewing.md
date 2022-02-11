---
layout: default
usemathjax: true
permalink: /render/ch2
---

# 2 - Geometric Transformation and Viewing

## Coordinates Pipeline

![](/notes-blog/assets/img/render/coordinates_pipeline.png)

- Local coordinates:
  - Local coordinate frame (e.g. centering at origin (sphere)/corner (cuboid))
  - Convenient for modelling
- World coordinates:
  - Common space for all actors to share locations relative to each other.
  - Objects, Lights, Camera pose defined here
- [Camera coordinates](/notes-blog/cg#view-transformation):
  - Origin: center of projection of the camera
  - Right handed coordinate system (thumb-index-middle: x-y-z)
  - Look vector: -z axis
  - Up vector: +y axis (without this, the camera would be freely rolling)
  - `gluLookAt` gives OpenGL $u,v,n$ vectors to perform view transformation
  - Vertex View transformation: $M_\text{view} = R T$
    - $R$: **rotate** the camera position back to world **axes**
    - $T$: **move** the camera position back to world **origin**
  - Normal View transformation: $M_\text{view, normal} = (M_t^{-1})^T$
    - $M_t$: the upper left 3x3 submatrix of $M_\text{view}$
    - If $M_t$ is just a rotation, $M_\text{view, normal} = (M_t^T)^T = M_t$
- Clip space:
  - Clips out stuff outside of a defined view volume
  - Projection matrix: maps the points in the view volume to a **canonical view volume**
- Normalized Device Coordinates (NDC) space:
  - 2x2x2 cube defined by planes $x=\pm1, y=\pm1, z=\pm1$
- Window space

After reaching the **window** space, we will have the xy-coordinates of the fragment, and its depth (z-value) and reciprocal of

## GLM Library

You can use arrays to represent the 4x4 matrix, arranged in column major order.

![](/notes-blog/assets/img/render/row_col_major.png)

## Lighting computation: Eye space

With a local illumination model e.g. Phong Model (i.e. ignore other objects' reflections)

Vertex shader -- Gouraud shading

```glsl
// ec = eye coordinate
vec3 ecNormal = normalize(NormalMatrix * vNormal);
vec4 ecPosition4 = MVMatrix * vPosition; // homogenous coordinates
vec3 ecPosition = vec3(ecPosition4) / ecPosition4.w; // euclidean space
gl_position = MVPMatrix * vPosition; // clip space coordinates

vec3 viewVec = -normalize(ecPosition); // as eye is at origin
vec3 lightPos = vec3(LightPosition);
vec3 lightVec = normalize(lightPos - ecPosition);
vec3 reflectVec= reflect(-lightVec, ecNormal); // note lightVec must be inverted!

// our dot products
float N_dot_L = max(0.0, dot(ecNormal, lightVec));
float R_dot_V = max(0.0, dot(reflectVec, viewVec));

// exponent n
float pf = (R_dot_V == 0.0) ? 0.0 : pow(R_dot_V, MatlShininess);

v2fColor = LightAmbient * MatlAmbient + 
    LightDiffuse * MatlDiffuse * N_dot_L + 
    LightSpecular * MatlSpecular * pf;
```

Fragment shader

```glsl
fColor = v2fColor;
```

Vertex shader -- Phong shading

```glsl
out vec3 ecPosition;
out vec3 ecNormal;
// computed as above, gl_position set as above
```

Fragment shader -- Phong shading

```glsl
uniform mat4 ViewMatrix;
uniform mat4 MVMatrix;
uniform mat4 MVPMatrix;
// matrial attrbutes

vec3 viewVec= -normalize(ecPosition); // ecPosition is the interpolated coordinate from the rasterizer
vec3 newNormal = normalize(ecNormal);
vec3 lightPos = vec3(LightPosition);
vec3 lightVec = normalize(lightPos - ecPosition);
vec3 reflectVec= reflect(-lightVec, ecNormal); // note lightVec must be inverted!

// our dot products
float N_dot_L = max(0.0, dot(ecNormal, lightVec));
float R_dot_V = max(0.0, dot(reflectVec, viewVec));

// exponent n
float pf = (R_dot_V == 0.0) ? 0.0 : pow(R_dot_V, MatlShininess);

v2fColor = LightAmbient * MatlAmbient + 
    LightDiffuse * MatlDiffuse * N_dot_L + 
    LightSpecular * MatlSpecular * pf;
```

Basically move the lighting computation from the vertex shader to the fragment shader.
