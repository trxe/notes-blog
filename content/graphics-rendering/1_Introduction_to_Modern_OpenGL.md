---
layout: default
usemathjax: true
permalink: /render/ch1
---

# 1 - Introduction to Modern OpenGL

OpenGL (like Direct3D, Metal, etc.) is a **graphics API**, which provides the interface for specialized hardware on the GPU.

These APIs need a lot of code to optimize hardware utilization and often becomes a performance bottleneck. Need to download updates for drivers often. Also heavily rely on single core performance.

**Low-level graphics APIs** such as Vulkan shrinks the size of drivers, moving API closer to hardware again

## Core Profile OpenGL

Removes:

- `glBegin`, `glEnd`
- General polygons and quad primitives 
  - Only points, lines, and triangles left, and introduces **patches**
- Fixed function (Phong) lighting computation 
- Matrix stacks and geometric transformation e.g.`glRotate`, `glFrustum`
  - Matrices must be handled by yourself/external libraries

**Shaders handle all rendering**: to draw anything you have to write your own shaders.

Shader: a user program loaded and executed in a stage of the rendering pipeline.

![OpenGL 4.5 pipeline](/notes-blog/assets/img/render/openglv4-5.png)

### Evolution Trend of OpenGL

Rending application can easily become CPU limited: analyse triangles/sec

CPU bottleneck:

- 90s: SGI Reality Engine 2 draws **200K triangles/s**, with CPU running at 100MHz
  - i.e. 100MHz $\div$​ 200K triangles/s = 500 cycles/triangle
- today: high end GPU draws 6 billion triangles/s, CPU running at 3GHz
  - i.e. 0.5 cycles/triangle (with a single core CPU you can't render even 1 triangle in 1 cycle!)

Modern way: Put all the data to pass to the GPU in memory, and a few function calls to pass the data to the GPU.

## Pipeline

Shaders replace **fixed-function pipeline** parts that had hard-coded functionality.

### Vertex Fetch

Gets all info of the vertices of primitives to be drawn by GPU.

### Vertex shader

Per-vertex execution. Produces vertices in [**clip space**](/notes-blog/cg/#projection-matrix). 

Replaces in the fixed-function stage:

- View transformation and projection
- Normal transformation and normalization
  - Model to view space
- Texture coordinate generation and transformation
- Lighting computation
  - Compute in view space
  - Resulting color replaces input vertex's color attribute
  - Application of material to color (?)

### Tessellation

[Not covered] Breaking primitives into smaller triangles. e.g. Bezier patch -> triangles. 

#### Tessellation Control Shader (TCS)

Per-patch vertex (control points) execution. Produces **tessellation output patch vertices**, i.e. **tessellation level factors** (level of subdivision), outer and inner.

#### Tessellation Primitive Generator (TPG)

Per-tessellation level factors execution. Performs tessellation and generates tessellation coordinates. 

#### Tessellation Evaluation Shader (TES)

Per-tessellation coordinate execution. Produces vertex position from tessellation coordinate

![Tessellation engine](/notes-blog/assets/img/render/tessellation_engine.png)

### Geometry Shading

[Assume vertex shader -> geometry shader. Another optional stage.]

Per-primitive execution.

- e.g. triangle: once each of its three vertices reach the geometry shader, then the geometry shader can be executed

Produces either:

- 0 primitives: culling (can reduce load further down pipeline!)
- $\geq$ 1 primitives: geometry amplification e.g. subdivision

Can transform a primitive (e.g. one point $\rightarrow$ triangle(s))

### Primitive assembly etc.

Special hardware to handle these.

- Primitive assembly
- Clipping
- Perspective division
  - [Clip to NDC space](/notes-blog/cg/#perspective-division)
- Viewport transformation
  - [NDC to window space](/notes-blog/cg/#viewport-transformation-ndc-to-window-space)
- Culling
  - Back-face culling: In OpenGL, get the signed 2D area of a triangle and cull if value $\le 0$​.

### Rasterization

If not clipped out, produces a set of fragments (window space pixel location, color, depth attributes)

Can interpolate any type of vertex attributes over primitives.

### Fragment Shader

Per-fragment execution, hence the busiest stage. Produces color of corresponding pixels in framebuffer.

Replaces in the fixed-function stage:

- Texture access (using interpolated texture coordinates)
  - multi-textures with multiple sets of textures
- Texture application
  - combination with the primitive's fragment color/applying previous layer of texture
  - replace, modulate, decal, blend (addition/multiply)
- Color sum
  - combining primary and secondary colors
  - lighting computation result is stored in two parts. 
    - primary = ambient + diffuse
    - secondary = specular
  - Separating specular: We don't want our specular color to be affected by the texture (e.g. when modulated with multiply effect, almost certainly will darken the specular). But our texture mapping should be combined with both our primary components.

### Per-Fragment Operations

Special hardware operations. Includes:

- Alpha to coverage
- Stencil test
- Z-buffer test
- Occlusion query
- Blending
- SRGB conversion
- Dithering 
- Logic op
- Multisample fragment operations

### Frame Buffer

![Pipeline from Wright et al.](/notes-blog/assets/img/render/wright_simplified_pipeline.png)

## OpenGL programs

### Old OpenGL

GLUT: `glutMainLoop` initiates an infinite **event loop**. In each loop, GLUT will:

- Look at event queue
- For each event, execute corresponding callback if present, else ignore event.

### New OpenGL

GLEW: Extension loading library

- Initialize entry points of New OpenGL functions
- Check for availability of any extension

GLM: OpenGL Mathematics library

- Header only library based on GLSL specifications
- Handles e.g. matrices

