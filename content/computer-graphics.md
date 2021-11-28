---
layout: default
title: Computer Graphics
usemathjax: true
permalink: /cg
geometry:
- top=2cm
- left=2cm
- right=2cm
- bottom=2cm
---

# Computer Graphics

The contents follow the NUS module CS3241 (taken in AY2021/22 S1).

* Table of Contents
{:toc}
## Pipeline summary


- Modelling (in API)
  - [x] Provides set of vertices
  - Scene processing to reduce geometric data down pipeline
    - [x] view-frustum culling
    - [x] occlusion culling
- Vertex Processing
  - [x] Model-View transformation
  - [x] Lighting computing
  - [x] Texture coordinates
  - [x] Projection Matrix
- **Primitive assembly etc.**
  - [x] Primitive assembly
    - vertex data --> complete primitives for clipping & culling
  - [x] Clipping
  - [x] Perspective division
    - To NDC space
  - [x] Viewport transformation
    - To window space
    - Depth range scaling
  - [x] Back face culling
    - throw away faces that are back-facing the viewport
- **Rasterization** in window space
  - [x] Produces **fragments** of unclipped-out primitives (potential pixels [*location + color + depth*])
  - [x] Attributes are interpolated over the primitive
    - e.g. Bilinear interpolation
- Fragment processing
  - [x] Fragment pixel --> Frame buffer pixel
  - [x] Fragment color modifies by texture mapping
    - Texture access, application
  - [x] **Z-buffer** Hidden surface removal: fragment discarded if occlused by pixel already in frame buffer
  - [x] **Blended**: blending with pixels already in frame buffer

$\require{color}$

# Chapter 1 -- Intro

## Image Formation

**Scene**: **VOLM** (Viewer, object, light source, material)

![Synthetic Camera Model](/notes-blog/assets/img/cg/vantagepointedit.png)

Let $d$ be the distance between the virtual plane and the pinhole. By similar triangles, $x_p = \frac{dx}{z}, y_p  = \frac{dy}{z}, z_p = d$.

If $O \neq C$​, then for each point $P (x,y,z)$​ on the 3D object in the world space, $\overrightarrow{CP}= \overrightarrow{OP} - \overrightarrow{OC}$​​​. 

**Color**: RGB corresponding to visual tristimulus.

## Graphics System Design

**API**: specifies **scene (VOLM)**, configures/controls the system.

**Renderer**: produces **images** using scene info + system config.

### Rendering approaches

**Ray-tracing**: Following rays of light from $C$ to some point $P$ on objects, or go off into infinity.

- global effects (recursive reflections, translucents via refraction)
- slow (must have whole database available)

**Radiosity**: Slow, non-general energy based approach

**Polygon Rasterization**: For each new polygon (primitive) generated, its pixels are assigned.

- **Transform**: mapping of polygon **3D** $\rightarrow$​ **2D**
- **Rasterize**: assigning **colors** to pixels
- Local lighting
- Polygon-based rendering
- Efficient

### Representation of Objects

Objects are meshes of polygonal primitives.

Each primitive is rendered through this pipeline.

![GL Pipeline](/notes-blog/assets/img/cg/glpipeline.png)

1. Vertex Processor
   - Vertices are processed via **vertex shaders**.
2. Clipper, primitive assembler
   - Perspective division
   - By the end of this, vertices of polygons are mapped to 2D Canvas.
3. Rasterization
   - Input: polygon vertices
   - Output: list of pixels that are **turned on** in the polygon (**fragment**)
4. Fragment processing
   - Determining the color of fragments
   - Texture mapping/interpolation of vertex colors
   - Blocking/Occlusion
5. Per-fragment & Frame Buffer operations
   - Hidden surface removal
   - Occlusion handling

### Frame Buffer

The region of memory that holds color and depth data via the attached color and depth buffers.

## API

Required: Functions to specify **VOLM**, I/O, system capacities (e.g. how many light sources can be supported?)

**Primitives** are defined with vertices.

- Points (0D)
- Line segments (1D)
- Polygons (2D)
- Some curves, surfaces (Quadrics, parametric polynomials)

```
glBegin(GL_POLYGON) 
	glVertex3f(0.0, 0.0, 0.0);
	glVertex3f(0.0, 1.0, 0.0);
	glVertex3f(0.0, 0.0, 1.0);
glEnd();
```

**Camera** is specified with

- Position $(x,y,z)$
- Rotation $(\theta_x,\theta_y,\theta_z)$​ of film plane
- Lens
- Film size

**Light source** is specified with

- Point vs distributed
- Spot lights
- Distance
- Color

**Material** attributes are

- Absorption: color properties
- Scattering (diffuse, specular)



# Chapter 2 -- Elementary OpenGL

## History of OpenGL

The success of **IRIS GL**, the graphics API for the world's first graphics hardware pipeline, led to **OpenGL (1992)**. It's a platform-independent API.

- Focus on rendering
- Omits I/O and windowing to avoid dependencies
- OpenGL core library is `OpenGL32` on Windows, `libGL.a` on *nix.
- OpenGL Utility Toolkit (GLUT) 
  - **System independent** windowing, I/O, menus
  - Event-driven mode of execution
  - Lacks a lot of functionality due to lack of specificity

### Functions

- Primitives
- Attributes
- Transformations (viewing, modelling)
- Control (GLUT)
- I/O (GLUT)
- Query

### State Machine

**Primitive Generation**: if primitive is visible, then try output. *Appearance depends on the primitive's **state***.

**State Changing**: e.g. Color changing is a state changing function. Changes the "pen color" state to $\textcolor{red}{\textsf{RED}}$ for instance, then it assigns those primitives affected to state $\textcolor{red}{\textsf{RED}}$​​.

### OpenGL pipeline

![OpenGL pipeline](/notes-blog/assets/img/cg/openGLpipeline.png)

### Function format

All vectors are represented as a 4D vector by default internally.

<img src="/notes-blog/assets/img/cg/glfuncformat.png" alt="Format" style="zoom:50%;" />

## Program structure

Commands are sent to a **command queue/buffer**.

```c++
int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB);
    glutInitWindowSize(h, w);
    glutInitWindowPosition(0, 0);
    glutCreateWindow("simple");
    glutDisplayFunc(mydisplay); // mydisplay is the display callback
    
    init();
    
    glutMainLoop(); // if anything happens, calls mydisplay to refresh
}
```

- `main` defines **callback**, opens **windows**, enters event **loop**.
- `init` sets state variables (viewing, attributes).
- **callbacks**: display callback, input and window functions.

```c++
void init() {
	glClearColor(0.0, 0.0, 0.0, 1.0); // black clear color + opaque window
    
    glColor3f(1.0, 1.0, 1.0); // set fill to white
    
    glMatrixMode(GL_PROJECTION); 
    glLoadIdentity();
    glOrtho(-1.0, 1.0, -1.0, 1.0, -1.0, 1.0); // FOV setting.
    // the square is *enclosed* in the FOV.
}
```

## Coordinate Systems

At the beginning, **object coordinates** = **world coordinates**. The camera is positioned in **world**. 

Then a **viewing volume** must be set. It is specified in **eye coordinates**.

Internally OpenGL converts vertices to **eye coordinates**, and then to **window coordinates**.

**Viewing volume**: The finite volume in world space in which objects can be seen. The coordinates are set relative to the camera's coordinates (**eye coordinates**).

**Orthographic projection/viewing**: Th

near plane (negative z direction), far plane (positive z direction)

### Polygon problems

OpenGL only renders **Simple, Convex, Flat** polygons.

**Convex:**

- Every internal angle $\leq 180$​ degrees.
- Every line segment between two points in/on the polygon's boundary is fully contained in the polygon.
- Polygon is the convex hull of its edges.

### RGB Color Coordinates

`glShadeModel(GL_SMOOTH)` allows smooth interpolation of colors across the interior of the polygon.

`glShadeModel(GL_FLAT)` picks **one vertex** to follow its color

### Viewport

`glViewport(x,y,w,h)`

## 3D drawing

Here the **order** in which polygons are drawn matters, otherwise hidden surface removal would be necessary to prevent surfaces "clipping" through one another.

How do we make a 3D Sierpinski Gasket given the code for a 2D one?

### Cheap Trick

![3D Gasket from subdividing 4 surfaces.](/notes-blog/assets/img/cg/3dgasket.png)

4 different Sierpinski triangles (see reds, blues, blacks and greens).

However the triangles are drawn in the order defined by the program, the black , red and blue triangles may not always be in front of the reds as desired.

![Wrong depths of surfaces](/notes-blog/assets/img/cg/hidden_surface.png)

#### Hidden surface removal

OpenGL now must use HSR via the **z-buffer algorithm** that saves depth info (via depth buffer, aka z-buffer) as only objects with lowest z-value at that pixel are rendered.

# Chapter 3 -- Input and Interaction

**The display loop:** 

Input **Physical** properties -- Mouse, Keyboard etc. **Contains trigger** which sends a signal to the OS.

Input **Logical** properties -- What is returned to the program via an API (the **measure**).

### Event mode

**Trigger** generates **event**. Event measure is entered into **event queue**.

### Callbacks

The main programming interface for event-driven input. For each type of event the system recognizes, we define a callback.

e.g. `glutMouseFunc(mymouse)` means `mymouse` is set as the mouse callback to handle any mouse clicking.

#### GLUT Callback loop

On each pass of the event loop, for each event in the queue, GLUT calls the callback function if it is defined.

e.g. The display callback is executed whenever window needs to be refreshed. 

- However many things need to update the screen! Multiple executions of the display on a single pass
- `glutPostRedisplay()` when called, sets a **flag**.
- if **flag** is on, display CB is called just once in the event loop

### Display animation

To redraw the display, we usually call `glClear()` first. But the frame buffer is decoupled from the display of the contents when cleared. Partially drawn display will then **flicker**.

**Double buffering:** 

- front buffer -- connected to display, but not written to
- back buffer -- written to, but not displayed
- **SWAPPING** between the two.
- `glutInitDisplayMode(GLUT_RGB | GLUT_DOUBLE)` in `main()`

```
void mydisplay() {
	glClear(GL_COLOR_BUFFER_BIT|...) // on the back buffer
	/* draw my graphics*/
	glutSwapBuffers(); // push the back to the front and front to the back
}
```

**Idle callback:** `glutIdleFunc(myidle)`

```
void myidle() {
	t += dt
	glutPostRedisplay(); // mark to update image.
}
void mydisplay() { /* draws something that depends on t */}
```

Note that since the signatures of all GLUT callbacks are fixed, to pass around variables we need to use globals.

### Examples of callbacks

1. Mouse callback
2. Positioning (note openGL window coordinates start from *bottom-left*, mouse coordinates  start from *top-left*)
3. Motion callback `glutMotionFunc(...)` and `glutPassiveMotionFunc(...)`
4. Keyboard callbacks `glutKeyboardFunc(...)`
5. Window resizing `glutReshapeFunc(...)`: good place to put viewing functions, since it is invoked from window first opening.

### Reshape callbacks

**Template**:

```C++
void myReshape(int w, int h) {
	glViewport(0, 0, w, h);
	glMatrixMode(GL_PROJECTION);
	glLoadIdentity();
    /* set projection matrix here! */
	glMatrixMode(GL_MODELVIEW);
}
```

Viewport aspect ratio = Window aspect ratio:

```C++
// l, r, b, t are the original left right bottom top coordinates.
if (w <= h)
	gluOrtho2D( l, r, b * (GLfloat) h / w, t * (GLfloat) h / w );
else
	gluOrtho2D( l * (GLfloat) w / h, r * (GLfloat) w / h, b, t );
```

**Resize exactly** (clip if necessary):

```C++
// l, x, b, y are the original left (right-left) bottom (top-bottom) values.
// This version anchors to the bottom left.
gluOrtho2D( l, l + x * h/origWinWidth, b, b + y * h/origWinHeight );
```



# Chapter 4 -- Vector Geometry

- Vector: Quantity with *direction* and *magnitude*
  - Has inverse (direction inverted, magnitude the same)
  - Zero vector (undefined direction, 0 magnitude)
  - Head to Tail axiom for summing vectors

## Representation of Vector Space

**Affine space.** Vector space without a fixed origin. Between 2 points we have a displacement/translation vector.

**Frame.** Adding a single point to the affine space as an origin.

## Transformations

*Note: we only need to transform endpoints of line segments, points on the line segment itself is drawn by the implementation.* 

| Type                                                         | Matrix                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Translation $T(d_x, d_y, d_z)$<br />$T^{-1}(v) = T(-v)$​      | $\left[ \matrix{1 & 0 & 0 & d_x \\\\ 0 & 1 & 0 & d_y \\\\0 & 0 & 1 & d_z \\\\ 0 & 0 & 0 & 1} \right]$ |
| Rotation $R_z(\theta)$​<br />$R^{-1}(\theta) = R(-\theta)\\\\ = R(\theta)^T$​ | $\left[ \matrix{\cos\theta & \sin\theta & 0 & d_x \\\\ -\sin\theta & \cos\theta & 0 & d_y \\\\ 0 & 0 & 1 & d_z \\\\ 0 & 0 & 0 & 1} \right]$ |
| Scaling $S(s_x, s_y, s_z)$​<br />$S^{-1}(v) = S(1/v)$         | $\left[ \matrix{s_x & 0 & 0 & 0 \\\\ 0 & s_t & 0 & 0 \\\\ 0 & 0 & s_z & 0 \\\\ 0 & 0 & 0 & 1} \right]$ |
| Shear $H(\theta)$​​<br />(Note that this<br /> shear matrix does $x' = x+ y\cot\theta$​.) | $\left[ \matrix{1 & \cot\theta & 0 & 0 \\\\ 0 & 1 & 0 & 0 \\\\ 0 & 0 & 1 & 0 \\\\ 0 & 0 & 0 & 1} \right]$ |

**Instancing.** Starting with a simple object @origin, @original size, we can apply an **instance transformation** to scale/orientate/locate.

**Rotate** object about point $p_f$: $M = T(p_f)R(\theta)T(-p_f)$.

### OpenGL Transformations

**Matrix modes.** Matrices are part of the state, and matrix modes provide a set of functions for manipulation. `GL_MODELVIEW` and `GL_PROJECTION` are two such matrices.

![A vertex position is transformed by the following matrices and operations in each of the subsequent frames.\label{GLtransformations}](/notes-blog/assets/img/cg/openGLmatrix.png)

Each mode has a 4x4 homogenous coordinate matrix. The **current transformation matrix (CTM)** is part of this state and applied to all vertices down this pipeline. $p' = Cp$ where $C$ is the CTM.

- Load
- Post-multiply (to layer transformations)

`GL_MODELVIEW` used to transform objects into world space (also position camera, but `gluLookAt` does that better). `GL_PROJECTION` is used to define the view volume.

| function (each has `double` ver. too) | effect                                                      |
| ------------------------------------- | ----------------------------------------------------------- |
| `glLoadIdentity()`                    | loads $I$                                                   |
| `glRotatef(theta, vx, vy, vz)`        | post-multiply $R(\theta)$. $[vx, vy, vz]$​​​ are rotation axes |
| `glTranslatef(dx, dy, dz)`            | post-multiply $T(dx, dy, dz)$                               |
| `glScalef(sx, sy, sz)`                | post-multiply $S(sx, sy, sz)$​​​​                               |
| `glLoadMatrixf(m)`                    | loads 4x4 matrix $M$​ as a `float[16]`                       |
| `glMultMatrixf(m)`                    | post-multiply $M$.                                          |

#### Matrix Stacks

- OpenGL maintains a stack of transformation matrices for each matrix mode for future reuse
- `glPushMatrix` and `glPopMatrix`

```C++
...
glPushMatrix();
	glRotated(i*45.0, 0.0, 0.0, 1.0); // pushed each of these onto a stack
	glTranslated(10.0, 0.0, 0.0);
	glScaled(0.5, 0.5, 0.5);
	DrawSquare(); // Apply stacked matrix to the element (square)
glPopMatrix(); // Emptied out stack
...
```



# Chapter 5 -- Camera posing

## Camera pose

**Pose.** Position + Orientation. Done by *view transformation*.

**Projection.** Orthographic vs Perspective.

**Prerequisite:** object geometry to be set and placed the world coordinate.

### Projections

**Projectors**. lines that either

- converge at a **center** of projection
- parallel to a **direction** of projection

They preserve lines but not necessarily angles (in the 2D image). Preserving here means that the angles between 2 vectors in the object are the same as that in the image.

#### Perspective projection

- Further = smaller [dimunition]
- Equal distances along a line are not projected to equal distances. [non-uniform foreshortening]

## Computer vision

- Positioning of camera [setting the model-view matrix]
- Selecting a lens [setting the projection matrix]
  - Perspective/Orthographic
  - Sets the FOV of the camera (field/clipping volume -- the region of the world in viewport)

`MODELVIEW` actually is a combination of the modelling transformation first (to the world space), and the view transformation (to the eye/camera space). `PROJECTION` matrix brings this to the clip space, and so on.

## Spaces

- **Local/Modelling/Object space**
  - Circle of radius $r$: defined as points equidistant from **circle origin (center)**
  - Cube of length $l$: defined with one corner as **cube origin**
- **World space**
  - Common coordinate frame for all objects. Local space transformed to world space
    - With respect to position of object origins in space.
  - What's *transformed* to here?
    - Vertices, vector normals
  - What's *defined* here?
    - Lights, camera pose
- **Camera coordinate frame**
  - Local frame to the camera, where camera is @origin
  - **Projections** specified wrt the camera frame.
  - Default `MODELVIEW` matrix is $I$ so initially world frame = camera frame
  - Default `PROJECTION` matrix is $I$ and orthographic ().
  - View volume (default: $2^3$​​ cube centered @world origin) is defined wrt the camera frame.

![Camera](/notes-blog/assets/img/cg/camera.png)

## View Transformation

`gluLookAt(eyex, eyey, eyez, atx, aty, atz, upx, upy, upz)`

- `eye`: origin of camera lens (*viewpoint*)
  - Defined in world space
- `at`: any point along the *viewing direction*
  - Defined in world space
- `up`: $y$ axis of the camera
  - By right, defined in world space **that coincides with $y_c$**
  - In practice, the vector `up` doesn't have to be perpendicular to `eye - at`, as long as $y_c$ can be found by a linear combination of these two

Camera's **coordinate vectors** $u, v, n$​​ are then derived in the $x,y,z$​​ directions respectively.

- $n$​ ($z$-axis): normalize($eye - at$)
- $u$ ($x$-axis): normalize($n \times up$)
- $v$ ($y$-axis): $n \times u$

#### View transformation matrix

So all points in **world frame** are expressed wrt the **camera frame**.

- OpenGL generates $M_{view} =  RT$ based on the camera coordinates. This is the **view transformation**.
  - The last transformation in the model-view matrix.
  - $T$: Moves camera to origin.
  - $R$: Rotates camera to axes of world frame.
- Multiply $M_{view}$​ to points in the world frame to get the camera frame.


$$
M_{view} = \left[ \begin{matrix}
u_x & u_y & u_z & 0 \\
v_x & v_y & v_z & 0 \\
n_x & n_y & n_z & 0 \\
0 & 0 & 0 & 1
\end{matrix} \right] \left[ \begin{matrix}
1 & 0 & 0 & -e_x \\ 
0 & 1 & 0 & -e_y \\ 
0 & 0 & 1 & -e_z \\ 
0 & 0 & 0 & 1 \\ 
\end{matrix} \right]
$$



*(We can use $u, v, n$​ directly in $R$​ because they are unit vectors in the desired directions)*

#### Normal vectors' transformation matrix:

$M_n = (M_t^{-1})^T$, where $M_t$ is the top left 3x3 submatrix of $M$.

3x3 matrix is enough cos it does not have position (thus $M_n$ would be an affine transformation).

**Since rotation matrices are orthogonal**, $R^{-1} = R^T$. Thus if $M$ is a rotational matrix, $M_n = M_t$.

## Projection matrix

Defined with a view (clipping) volume in the camera frame, and is specified in camera space.

- Orthographic: `glOrtho(left, right, bottom, top, near, far)`
- Perspective: `glFrustum(left, right bottom, top, near, far)` 
- Note `near` is in +z direction, and `far` is in -z direction, but provide `near < far`

The projection matrix is computed such that it maps to the **canonical view volume** ($2^3$ cube) in Normalized Device Coordinates (NDC) space. Criteria

- retain depth (z-buffer information)
- preserve lines

### Orthographic projection

**Mapping** from $(right,top,-far) \rightarrow (1, 1, 1)$, and $(left,bottom,-near) \rightarrow (-1, -1, -1)$.

![Projection mapping](/notes-blog/assets/img/cg/projection_mapping.png)

#### Orthographic projection matrix (Clip to NDC):


$$
\begin{aligned}
M_{ortho} =& S(\frac{2}{right - left}, \frac{2}{top-bottom}, \frac{2}{near-far}) \cdot \\
& T(\frac{-(right+left)}{2}, \frac{-(top+bottom)}{2}, \frac{-(far+near)}{2})
\end{aligned}
$$



1. Translate the midpoint (take the averages of the dimensions)
2. Scale the cuboid to the $2^3$​ cube.
3. Now each coordinate $x_{NDC},y_{NDC},z_{NDC} \in [-1, 1]$​.

#### Viewport transformation (NDC to Window Space):

The viewport here is defined by $(x_{vp}, y_{vp})$​ (the lower left corner), $w,h$​.


$$
\begin{aligned}
\frac{x_{NDC} - (-1)}{2} &= \frac{x_{win} - x_{vp}}{w} \Rightarrow x_{win} = x_{vp} + \frac{w(x_{NDC} + 1)}{2} \\
\frac{y_{NDC} - (-1)}{2} &= \frac{y_{win} - y_{vp}}{h} \Rightarrow y_{win} = y_{vp} + \frac{h(y_{NDC} + 1)}{2} \\
&z_{win} = \frac{z_{NDC} + 1}{2}
\end{aligned}
$$

Squashed into the $x,y$ plane. $z$ is retained (between 0 and 1) for hidden surface removal.

### Perspective Projection

**Mapping** from $(right,top,-far) \rightarrow (1, 1, 1)$, and $(left,bottom,-near) \rightarrow (-1, -1, -1)$.

![Frustum Mapping](/notes-blog/assets/img/cg/frustum_mapping.png)

#### Perspective Division:

Essentially using similar triangles (as described in first chapter) to get coordinates adjusted for perspective depth. note that $z_p = d$​​​​, the projection plane.


$$
\begin{aligned}
&\frac{x_p}{x} = \frac{d}{z}, \quad \frac{y_p}{y} = \frac{d}{z}, \quad z_p = d \\
\Rightarrow& x_p = \frac{x}{z/d}, x_p = \frac{y}{z/d}, z_p = \frac{z}{z/d} \\
\end{aligned}
$$

![Perspective projection matrix](/notes-blog/assets/img/cg/mpersp.png)

`gluPerspective(fovy, aspect, near, far)` specifies a symmetric view volume.

![gluPerspective](/notes-blog/assets/img/cg/gluPerspective.png)

# Chapter 6 -- Clipping, Rasterization & Hidden-Surface Removal

## Clipping algorithms for line segments

#### Naive (Brute force):

Find intersections of 2D line segments with the clipping window.

#### Cohen-Sutherland:

Eliminate all **easy** cases without computing intersections.

1. Both endpoints in window: Accept as-is
2. One inside window, one outside: Compute intersection
3. Both endpoints outside in **same line**: Reject
4. Both outside **different boundary** lines: May have part inside, compute intersection

**Outcodes** are defined for each area:

![Outcodes](/notes-blog/assets/img/cg/outcodes.png)

Let $F(A,B)$​ represent the clipping requirement of the line segment $AB$​. $F(A,B)$​​ equals 2 if it is not clipped out, 1 if it is partially clipped out, and 0 if it is fully clipped out.



$$
F(A, B) = \begin{cases}
2 & A = 0 \text{ and } B = 0 \\
1 & A = 0 \text{ and } B \neq 0 \\
0 & A \text{ AND } B \text{ (bitwise)} \neq 0 \\
1 & A \neq 0 \text{ and } B \neq 0 \text{ and } A \text{ AND } B \text{ (bitwise)} = 0
\end{cases}
$$



##### Implementation:

To compute the last case: **shorten** line segment by intersecting with one of the sides, recursively compute outcode until $A = B = 0$. For 3D, we have 6-bit outcodes ($9 \times 3 = 27$​ of them, each prefixed with 00, 01, or 10).

## Clipping in clip space

**Clip space**: After projection matrix, before perspective division.

If vertex $[x_{clip}, y_{clip}, z_{clip}, w_{clip}]$ is in canonical view volume $[-1,1]^3$ after perspective division, we must have $-1 \leq x_{clip} / w_{clip} \leq 1 \Rightarrow -w_{clip} \leq x_{clip} \leq w_{clip}$​, and similarly for $y, z$. These define the clipping planes.

## Polygon Clipping

| To be clipped   | Yields   |
| --------------- | -------- |
| Line segment    | 1        |
| General Polygon | $\geq 1$ |
| Convex polygon  | 1        |

Hence change polygons to convex polygons (*tessellation*), commonly using **triangulation**.

### Pipeline clipping

All line segments pass through top clipping, then bottom clipping, and so on until final form. Similar for polygons (with front and back clippers.)

However small increase in latency.

### Simple Early Acceptance and Rejection

Not implemented in pipeline.

1. Draw a **bounding box**
   - max and min of $x, y$​​ in window.
2. Is the whole bounding box inside? Accept? Wholly outside? Reject
3. Otherwise, enforce detailed clipping

## Rasterization

### Digital Differential Analyzer (DDA)

Straight lines $y = mx + b$​ satisfies **differential equation** $\frac{dy}{dx} = \frac{y_e - y_0}{x_e - x_0}$​​.

Case 1: $0 \leq \|m\| \leq 1$​​​. Loop through each pixel $(x_i, y_i) = (x_0 + i, y_{i-1} + m)$​​.

Case 2: $\|m\| > 1$​​​​​. Then note that $0 \leq \frac{1}{\|m\|} \leq 1$​​​​​. Hence loop through each pixel  $(x_i, y_i) = (x_{i-1} + m, y_0 + i)$​​​​​​.

### Bressenham's Algorithm

Saves time because of no floating point computations.

Here since $d_{upper} > d_{lower}$​, we choose to shade $(x_k + 1, y_k)$​​​.But how does it save FP computation time?


$$
\begin{aligned}
p_k =& \Delta x(d_{lower} - d_{upper}) \\
=& \Delta x(y - y_k - (y_k+1) + y) \\
=& \Delta x(2y - 2y_k - 1) \\
=& \text{ to be filled in} \\
=& 2x_k \Delta y - 2y_k \Delta x + c
\end{aligned}
$$


If $p_k < 0$​ plot lower pixel, if $p_k > 0$​ plot upper pixel.

![Candidates for binary choice](/notes-blog/assets/img/cg/bressenham.png)

#### Incremental form: 

$p_0 = 2\Delta y - \Delta x$. Since $p_{k+1} - p_k = 2(x_{k+1} - x_k)\Delta y + 2(y_{k+1} - y_k)\Delta x$​, we have

- $p_k < 0 \Rightarrow p_{k+1} =p_k +2\Delta y$​. (because $y_{k+1} = y_k$)
- $p_k > 0 \Rightarrow p_{k+1} = p_k +2\Delta y - 2\Delta x$. (because $y_{k+1} = y_k + 1$)

## Polygon Scan Conversion (Fill)

Horizontal scanlines: Get both endpoints by interpolating the line segment they are each on.

![The scanline is then formed by interpolating c4, c5](/notes-blog/assets/img/cg/scanline.png)

### Flood-fill

from a seed point located inside, scan-convert the edges using edge color (recursively).

```C
flood_fill(int x, int y) {
	if (A[x][y] == WHITE) {
		A[x][y] = BLACK;
		flood_fill(x-1, y);
		flood_fill(x+1, y);
		flood_fill(x, y+1);
		flood_fill(x, y-1);
	}
}
```

## Back-face culling

Eliminate **back=facing** + **invisible** polygons (e.g. cube only have 3 faces facing viewport at all times. The rest can be culled.)

If the vertices on a plane are specified **counterclockwise**, that face is **front-facing**.

# Chapter 7 -- Illumination and Shading

## Illumination

Given **point on surface, light source, viewpoint**: compute **color** of **point**

### Local Illumination

- **ONE** light source, **ONE** surface point, view point
- No interactions

#### OpenGL implementation

- Since Gouraud shading is per-vertex, takes place at **Vertex Processing** stage, in **Camera space**

- `glEnable(GL_LIGHTING)` enables shading calculations, disables `glColor`

- `glEnable(GL_LIGHTi)` for $i \in [0 \dots 7]$​
  - `glLightModeli(parameter, GL_TRUE)`
  
- Defining a point light source: `GLfloat[]` for each
  - `ambienti`, `diffusei`, `speculari`
  - `lighti_pos`: if `lighti_pos[3] == 1.0`: point light. else if `== 0.0`: directional light
  - `glLightModelfv(GL_LIGHT_MODEL_AMBIENT, global_ambient)` sets the **global** ambient light $I_{ga}$
  
- `glMaterialfv(GL_FRONT, GL_X, type X)`
  - `AMBIENT, DIFFUSE, SPECULAR` = `GLfloat[]` i.e. $I_{a,i}, I_{d,i}, I_{s,i}$
  - `SHININESS` = `GLfloat`
  - `GL_FRONT | GL_BACK | GL_FRONT_AND_BACK`
  - `EMISSION` = `GLfloat[]` **Emissive** term $k_e$​ (glowing)
  
  


$$
I_\text{OpenGL} = k_e +I_{ga}k_a + \sum_i [I_{a,i}k_a + I_{d,i}k_d(N \cdot L_i) + L_{s,i}k_s(R_i \cdot V)^n]
$$

### Phong Illumination Equation

Efficient, acceptable image quality. Ambient ($I_a k_a$​​​​), diffuse ($f_{\text{att}} I_p k_d (N \cdot L)$​​​​), and specular ($f_{\text{att}} I_p k_s (R \cdot L)^n$​​​​​​) components. Single light source below:


$$
I_{\text{Phong}} = \left[ \begin{matrix} r \\ g \\ a \end{matrix} \right]
= I_a k_a + f_{\text{att}} I_p k_d (N \cdot L) + f_{\text{att}} I_p k_s (R \cdot L)^n
$$


Multiple light sources $L_i$:


$$
I_{\text{Phong}} = \left[ \begin{matrix} r \\ g \\ a \end{matrix} \right]
= I_a k_a + f_{\text{att}} I_p \sum_{i}[ k_d (N \cdot L_i) + k_s (R \cdot L_i)^n ]
$$

Simplified:
$$
I_{\text{Phong}} = I_a k_a + I_p [k_d (N \cdot L) + k_s (R \cdot L)^n]
$$


![Illustration of unit vectors and angles for the Phong model.](/notes-blog/assets/img/cg/phong.png)

- **Ambient:** Flat constant color $\begin{matrix} [I_{ar} k_{ar} & I_{ag} k_{ag} & I_{ab} k_{ab}] \end{matrix}^T$​
- **Diffuse:** Factors in orientation of the surface wrt light source via $N \cdot L$​.
  - Lambert's cosine law: diffuse reflection is proportional to $\cos\beta$​​ above.
  - Attenuation: loss of flux intensity when passing through a medium. (inverse square law)
- **Specular:** Highlights (more intense when $R \cdot V$​ approaches 1 i.e. light source parallel to reflection)
  - Narrow range of specularity means sharper and smaller highlights: higher shininess
  - $R = 2(N \cdot L)N - L$ (use **projection** of $L$ on $N$​)

The "constant" variables below are the material properties

| Variable  | Name                        | Task                                                         |
| --------- | --------------------------- | ------------------------------------------------------------ |
| $I_a$     | Ambient illumination        | RGB triplet lighting all visible surfaces                    |
| $k_a$     | Ambient constant (0..1)     | RGB triplet for the particular material                      |
| $f_{att}$ | Attenuation factor          | $\frac{1}{a + bd + cd^2}$​ for some constants $a,b,c$​. In many cases, $f_\text{att} = 1$​. |
| $I_p$     | Point illumination          | RGB triplet for this light source                            |
| $k_d$​​     | Diffuse constant (0..1)     | RGB triplet for the particular material                      |
| $k_s$     | Specular constant           | RGB triplet for the particular material                      |
| $n$       | Shininess constant (1..128) | Controls the range of angles that get strong specularity     |

### Blinn-Phong Illumination

$$
I_{\text{Blinn-Phong}} = I_a k_a + f_{\text{att}} I_p k_d (N \cdot L) + f_{\text{att}} I_p k_s (\textcolor{blue}{N \cdot H})^n
$$

$H$​​​ is the halfway vector between $L$​​​ and $V$​​​​​. Since $\| L \| = \| V \| = 1$​, we have $H = (L + V)/ \|L + V \|$​​. 

Since generally $\theta_{H, N} \leq \theta_{R, V}$​​, the specular highlights in Blinn-Phong are larger.

Advantage over Phong: omission of $R$ reduces computation necessary

- If the view vector is far away, the view directions $V$​ tend to be very close to being parallel.
- $V, H$​ does not have to be recalculated, but $R$​​ has to be 
- Hence Blinn-Phong more efficient at rendering surfaces at a distance.

<img src="/notes-blog/assets/img/cg/blinnphong.png" alt="Blinn-Phong model" style="zoom:50%;" />

### Surface normals

- Flat surface: cross product of two vectors in the plane.
- Curved surface: cross product of two vectors in the **tangent** plane
- `glNormal3f` or `glNormal3fv` to set normals before each vertex (make sure its unit vector!!)
  - or else `glEnable(GL_NORMALIZE)` at performance penalty

### Vertex normals

- Average of polygon normals of the polygons sharing vertex $v$
- From analytical surface normal vector (e.g. from some centroid)

### Global illumination

- **All** light sources and surfaces
- e.g. Inter-reflections, shadows

## Shading

Given **polygon, rasterization**: compute **color** of **fragment**

![Note that Phong Shading is not the same as PIE.](/notes-blog/assets/img/cg/shading.png)

### Shading types

- Flat: Pick any vertex in the polygon and fill in the polygon with its PIE color.
  - `glShadeModel(GL_FLAT)`
- Gouraud: Each vertex is assigned the **average normal** of the **polygons sharing** the vertex. Its PIE **color** using that normal is then **interpolated** across the polygon.
  - `glShadeModel(GL_SMOOTH)`
- Phong: Instead of interpolating color, we interpolate the normal vector instead. **per-pixel** lighting computation (more expensive)

#### Shading a sphere:

A sphere can be broken into $n$​ latitudinal lines and $2m$​​​ longitudinal lines. each polygon formed by intersecting two adjacent longitudinal lines and two adjacent latitudinal lines share a normal in flat shading.

![Sphere reference](/notes-blog/assets/img/cg/spherevertex.png)

Note: point on sphere on lines $(i,j)$​ ($i$ is latitude, $j$​ is longitude) has 
$$
p = \left[ \begin{matrix} 
r\sin(i\pi/n)\cos(j\pi/m) \\ 
r\cos(i\pi/n)\cos(j\pi/m) \\
r\sin(j\pi/m)
\end{matrix} \\ \right]
$$

- Precise normal: circle origin to point $p(x,y,z)$ on sphere​​
- Flat shading: polygon normal average of the four vertices.

# Chapter 8 -- Texture mapping

The **forward mapping** (texture to screen) involves:

1. **Surface parameterization**: Texture space $(s,t)$​ to Object space $(x,y,z)$​​.
2. **Projection**: Object space to Window space (with other polygons).

There also is a different implementation using **inverse mapping**: given pixel $(x_s, y_s)$​​, *lookup* the pixel's pre-image in texture space. This is more suitable for polygon-based raster graphics, and is the implementation in OpenGL.

![Forward and inverse mapping](/notes-blog/assets/img/cg/texmap.png)

## Surface parameterization

Defines **surface point** to **texel** mapping: $(x,y,z) \leftrightarrow (s,t)$​. (Range of texture coordinates $(s,t) \in [0, 1]^2$​​.)

Per polygon, only the vertices coordinates' mappings are specified (the rest are interpolated by the [rasterizer](#rasterization)), either manually (for complex objects), or via two-part texture mapping (for simple objects).

**Two-part texture mapping** involves:

- **S-mapping** (texture to simple intermediate 3D surface)
- **O-mapping** (intermediate surface to object surface)

**S-mappings** include mappings to plane, cylinder, sphere etc.

**O-mappings** include:

| Mapping                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <img src="/notes-blog/assets/img/cg/reflectedray.png" width="50%"> | Dependent on view vector. <br />Useful for environment mapping. |
| <img src="/notes-blog/assets/img/cg/objnormal.png" width="50%"> | Ray from surface point<br /> along object surface normal     |
| <img src="/notes-blog/assets/img/cg/objcentroid.png" width="50%"> | Ray from center through surface point                        |
| <img src="/notes-blog/assets/img/cg/intnormal.png" width="50%"> | Ray from intermediate surface point <br />through object surface point |

### Inverse mapping by bilinear interpolation

**Pre-image** is the texture area that the fragment maps to in texture space.

For each **texture coordinate** $(s, t)$ we can get:

- $(s', t', w) = (s/z_e, t/z_e, 1/z_e)$ where $z_e$ is depth in eye space
- $(s'/w, t'/w)$​​ as the final window space coordinates

| Interpolation in a triangle polygon                          | Interpolation of 4 texels                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <img src="/notes-blog/assets/img/cg/interpolationtriangle.png"> | <img src="/notes-blog/assets/img/cg/interpolationtexel.png"> |

**Texture filtering**: determine the texture color of a fragment using the colors of the nearest 4 texels. May be implemented via bilinear interpolation.

### Wrapping modes

**Clamp**: $wrap(u, v)  = (\|u\|, \|v\|)$

**Repeated** (useful for tiling): $wrap(u, v)  = (u\ \% \ 1.0, v\ \% \ 1.0)$​

## Aliasing

Can happen due to:

- **Point-sampling**: Choosing a single texel color after being inverse mapped
- during **texture minification**: Many texels crammed into area of fragment (random picking of texel)
  - **NOT** during texture magnification: blocky appearance is not aliasing.

### Anti-aliasing via mipmapping

By **area-sampling** each fragment, and then **averaging** the color that it is mapped to in the texture space.

**Mipmapping**: By **pre-filtering** texture maps, makes AA real-time performance-friendly. **Successively averages** each 2x2 texels until size = 1x1. Mipmap Level 0 is the original image, and Mipmap Level $\log(n)$​ is the 1x1 map. ("Level": level of texture minification.)

![Mipmapping](/notes-blog/assets/img/cg/mipmap.png)

**Selecting** mipmap levels: If pre-image corresponds to $n^2$​​ texels we choose Mipmap Level $\lfloor \log n \rfloor$​. 

Or in the case of **trilinear texture map interpolation**, we interpolate between $\lfloor \log n \rfloor$ and $\lceil \log n \rceil$.

## Applications

1. Modulation textures (modulates diffuse color, specularity, transparency)
2. Environment mapping
3. Bump mapping, light mapping, shadow mapping, displacement mapping...
4. Image-based rendering, non-photorealistic rendering...

### Environment mapping

Shortcut to rendering shiny objects: 

1. Mapping the image of the surrounding into a texture map (cubemap)
2. Use the [reflected eye ray](#surface-parameterization) O-mapping method to map to the object

Drawbacks compared to raytracing

- Cannot self-reflect (not captured in environment map)
- The closer the environment the bigger the error

### Bump (displacement) mapping

Provides a set of normals to offset the real surface's normals by for use during lighting computation.

![Bump mapping](/notes-blog/assets/img/cg/bumpmap.png)

### Image based rendering (billboarding)

The texture plane is always rotated to face the view ray. (Imagine the billboards turning to face you however you turn.)

## OpenGL Texture mapping

**Fragment processing** stage: the fragment color can be modified by texture application, mapped via texture access (using interpolated coordinates).

| Function                                                  | Task                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| `glGenTextures(int n, uint* textures)`                    | Creates $n$ textures, puts ids in `textures`.                |
| `glBindTextures(enum target, uint texture)`               | Binds texture to target.                                     |
| `glTexParameteri(enum target, enum pname, int param)`     | Sets parameter of target.                                    |
| `glTexEnvf(enum target, enum pname, const float* params)` | Texture environment specification. i.e. How should target texture be applied onto fragment? |



```C++
void init(void) {
    // Specifies memory alignment of each pixel, 1 -> byte-alignment.
    glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
    glGenTextures(1, &texObj); 
    glBindTexture(GL_TEXTURE_2D, texObj);
    
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    // GL_LINEAR: 4 nearest texel interpolation. not mipmapped.
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    
    // Maps texture image onto each primitive active for texturing.
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, imageWidth, imageHeight, 0, GL_RGBA, GL_UNSIGNED_BYTE, texImage);
}

void display(void) {
    // Activates texturing
    glEnable(GL_TEXTURE_2D);
    // how texture values are applied to the fragment 
    glTexEnvf(GL_TEXTURE_ENV, GL_TEXTURE_ENV_MODE, GL_DECAL);
    glBindTexture(GL_TEXTURE_2D, texObj);
    // map Texture Coordinates <-> Objects coordinates
    glBegin(GL_QUADS);
	    glTexCoord2f(0.0, 0.0); glVertex3f(-2.0, -1.0, 0.0);
    	// ...
    glEnd();
}
```

To generate mipmaps:

```C++
void init(void) {
    // ...
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
    gluBuild2DMipmaps(GL_TEXTURE_2D, GL_RGBA, imageWidth, imageHeight, GL_RGBA, GL_UNSIGNED_BYTE, texImage)
}
```



# Chapter 9 -- Ray casting & tracing

## Idea

### Ray casting

1. Fixed viewpoint
2. For each pixel, shoot a ray **from the viewpoint through the pixel**
   1. For each object,
   2. If it has an intersection with the ray, and is the closest,
   3. Transfer the color of the object to the pixel.

### Ray tracing

Upon each intersection, the object hit may continue to shoot out **secondary rays** that are:

- reflected to intersect with other objects as well.
- refracted if it is transparent/translucent
- shadow rays (a ray shot towards a light source, and indicates shadow is it is **occluded/blocked** by another object between the light source and surface point).

### Whitted (Recursive) Ray Tracing

- Free Hidden surface removal (ray casting)
- Global (**partial**) reflection
- Local reflection
- Easy Shadows
- **One simple framework!**

### Details

$$
\begin{aligned}
I &= I_{\text{local}} + k_{rg}I_\text{reflection} + k_{tg}I_\text{transmitted} \\
I_\text{local} &= I_ak_a + I_\text{source}[k_d(N \cdot L) + k_r(R \cdot V)^n + k_t(T \cdot V)^m]
\end{aligned}
$$

![Example of ray-tracing](/notes-blog/assets/img/cg/raytracepaths.png)

At each intersection we need to do a lighting computation and produce the reflected rays.

| Variable               | Name                               | Task                                                         |
| ---------------------- | ---------------------------------- | ------------------------------------------------------------ |
| $k_r$                  | Reflected specular constant (0..1) | Replaces the specular constant $k_s$                         |
| $k_t$​                  | Transmitted constant (0..1)        | For transparent/translucent materials                        |
| $I_\text{transmitted}$ | Intensity of transmitted light     | Back-propagated from the next intersection of a **refracted** ray |

**Reflected specular**: Essentially the same as the specular except that that its the reflected light, not just the highlight

Note that this equation is recursive due to the $I_\text{reflection}$ and $I_\text{transmitted}$ values to be computed.​

![Left: reflected specular; Right: transmission](/notes-blog/assets/img/cg/reflectrefractangles.png)

## Computation

### Reflection

$R = 2N\cos \theta - L = 2(N \cdot L)N - L$

### Refraction

Snell's law: $\mu_1 \sin \theta = \mu_2 \sin \phi$​. Let $\mu = \frac{\mu_1}{\mu_2}$​​, then


$$
\begin{aligned}
T &= -\mu L + \left( \mu(\cos\theta) - \sqrt{1 - \mu^2(1-\cos^2\theta)} \right)N \\
&= -\mu L + \left( \mu(N \cdot L) - \sqrt{1 - \mu^2(1-(N \cdot L)^2)} \right)N
\end{aligned}
$$

### Ray Tree (Recursion)

$k$ levels of recursion = $k$ reflection/refraction vectors in this case (i.e. 0 levels of recursion means eye to object ray only.)
$$
\begin{aligned}
I &= I_{\text{local}, 0} + I_{\text{reflected}, 0} + I_{\text{transmitted}, 0} \\
&= I_{\text{local}, 0} + I_{\text{local}, 1} + I_{\text{reflected}, 1} + I_{\text{local}, 1} + I_{\text{transmitted}, 1} \\
&= \dots
\end{aligned}
$$

![Ray Tree](/notes-blog/assets/img/cg/raytree.png)

### Shadow Rays

$$
I_\text{local} = I_ak_a + \textcolor{red}{k_\text{shadow}}I_\text{source}[\dots],\quad  \textcolor{red}{k_\text{shadow} \in \{0,1\}}
$$

Shoot another ray from view ray's intersection point towards the light source to tell if that spot is illuminated.

Hence shadow constant turns the diffuse, specular and transmitted values on or off. 	

Only need to find one opaque occluder to determine occlusion.

#### Translucent shadow rays (occlusion by translucent object)

- Light attenuation by $k_\text{tg}$
- Refraction is **ignored**

### Total ray count

- One primary ray per pixel
- (1 primary + $l$ shadow rays for $l$ light sources) + $k(1+l)$ for the recursed levels

### Scene Descriptions in Ray tracing

- `gluLookAt` Camera position and orientation (World frame)
- `gluPerspective` without near and far (FOV)
- Image resolution: (pixels per dimension)

- Point light source:
  - Position
  - $I_\text{source}$ (RGB)
  - $I_a$​​ (ambient, RGB)
  - Spotlight etc.
- Object surface
  - all $k$ constants, $n$, $m$, refractive index $\mu$
  - implicit representations (plane/spher/quadrics)
  - polygon
  - parametric (e.g. bicubic bezier patches)
  - volumetric (e.g. fog)

#### Stop recursion when...

- Surface is diffuse (no transmitted/specular reflected light)
- Secondary ray hits nothing
- Max recursion depth
- the contribution from secondary ray is almost 0

### Intersections

Ray = $P(t) = O(x,y,z) + t(\text{direction}(x,y,z))$​

For the following primitives, substitute $p = O(x,y,z) + tD$ in and find $t$.

**Plane** = $Ax + By + Cz + D = 0 \Rightarrow N \cdot p + D = 0$​ for point $p$​ on plane​

**Sphere** = $x^2 + y^2 + z^2 - r^2 = 0 \Rightarrow p\cdot p - r^2 = 0$​ for point $p$​​ from sphere origin.

- If sphere isn't at origin, just translate the ray origin to sphere's local coordinate frame

**Axis-aligned box** (AABB):

- typically used for quickly determining if a ray will hit the object (hitbox) or not
- specify the two **opposite** corners

**Triangle** (polygon): Using the **barycentric coordinates** method

$P = \alpha A + \beta B + \gamma C = \textcolor{limegreen}{A + \beta (B - A) + \gamma (C - A)}$​​ where $\alpha + \beta + \gamma = 1$​​ and $0 \leq \alpha, \beta, \gamma \leq 1$​​​.



$$
\begin{aligned}
O+ \textcolor{royalblue}{t}D &= A + \textcolor{royalblue}{\beta} (B - A) + \textcolor{royalblue}{\gamma} (C - A) \\ 
A - O &= \textcolor{royalblue}{\beta} (A - B) + \textcolor{royalblue}{\gamma} (A - C) +  \textcolor{royalblue}{t}D  \\
\left[  \begin{matrix} 
A_x - O_x \\
A_y - O_y \\
A_z - O_z \\
\end{matrix} \right] &= 
\left[  \begin{matrix} 
\beta (A_x - B_x) + \gamma (A_x - C_x) + tD_x \\
\beta (A_y - B_y) + \gamma (A_y - C_y) + tD_y \\
\beta (A_z - B_z) + \gamma (A_z - C_z) + tD_z \\
\end{matrix} \right] \\ &= 
\left[  \begin{matrix} 
(A_x - B_x) & (A_x - C_x) & D_x \\
(A_y - B_y) & (A_y - C_y) & D_y \\
(A_z - B_z) & (A_z - C_z) & D_z \\
\end{matrix} \right]
\left[  \begin{matrix} 
\beta \\
\gamma \\
t \\
\end{matrix} \right] \\
& \text{can be solved with Cramer's rule.}
\end{aligned}
$$

Note: the centroid of a triangle is located at $p = \frac{1}{3} (A+B+C)$

**General polygon**:


1. Compute ray-plane intersection
2. Determine if intersection is in polygon.

### Epsilon problem

Due to floating point errors, the intersection may be "slightly below the surface". This leads to self occlusion during shadow feeling. Methods to deal with it:

1. Use an $\epsilon$ and accept an intersection only if $t > \epsilon$ threshold.
2. When a new ray is spawned, advance the ray ($O + tD$) origin by $\epsilon D$​.

$\epsilon$ is based on the floating point accuracy.

![Self-occlusion](/notes-blog/assets/img/cg/shadowacne.png)

### Limitations of Whitted Raytracing

1. Hard shadows only. $k_\text{shadow} \in \{0,1\}$

2. Inconsistency between highlights and reflections:

   a. Light: Phong Illumination Model

   b. Shadow: Shadow ray reflection

3. Aliasing (jaggies). each area either intersects or doesn't.

4. Computes only some kinds of light transports. Doesn't include:

   1. Caustics (focus light on bottom of pool)
   2. Color bleeding

### Acceleration techniques

1. Adaptive recursive depth control
2. First-hit speed up with z-buffer
3. Bounding volumes (**hierarchies**)
   1. Must enclose the object as tightly as possible
   2. Must be fast to compute
   3. In order to accurately skip the expensive intersection computation between a ray and a complex object
4. **Spatial subdivision**
   1. Uniform grid,
   2. Octree
   3. Binary Space Partitioning (BSP)

### Vs Rasterization

Rasterization converts **primitives to pixels**

- hard to do global illumination

Raytracing converts **pixels to primitives**

- needs whole scene data

# Chapter 10 -- Curves and Surfaces

### Advantages

1. More compact representation than group of straight line segments
2. Scalable geometry primitives
3. Smoother, continuous primitives
4. Faster animation, collision detection

### Design

- Local Control of Shape
- Smoothness, continuity
- Evaluate derivatives
  - Derives the normal vectors for surface etc
- Stability 
  - With a small change in the variable, the curve shouldn't change drastically

## Formulations

### Explicit form

- Dependent variable given in terms of independent variable 
  - e.g. curves 2D: $y = x^2 + 2x + 1, y = mx + c$ etc.
  - e.g. curves 3D: $y = f(x), z = g(x)$
  - e.g. surfaces (always 3D): $z = f(x,y)$​
- Some curves and surfaces like circles and spheres cannot be represented in explicit form.

### Implicit form

- Single equation containing all variables that equals to 0
  - curves 2D: $f(x,y) = 0$
  - curves 3D: $f(x,y,z) = 0$, $g(x,y,z) = 0$​
  - surfaces 3D: $f(x,y,z) = 0$

- Hard to get points from implicit form.

### Parametric polynomial curves

Curves $p$ in 2D/3D, and their tangents $p'$: 1 parameter $u$​.


$$
p(u) = \left[ \begin{matrix} x(u) \\ y(u) \\ z(u) \end{matrix} \right], \quad p'(u) = \left[ \begin{matrix} x'(u) \\ y'(u) \\ z'(u) \end{matrix} \right]
$$


Surfaces $p$ in 3D, and their normals: 2 parameters $u,v$​.


$$
p(u,v) = \left[ \begin{matrix} x(u,v) \\ y(u,v) \\ z(u,v) \end{matrix} \right], \quad \vec{n} = \frac{\partial p(u,v)}{\partial u} \times \frac{\partial p(u,v)}{\partial v}
$$

Parametric curves are not unique. But they are the most used because they fit the criteria [here](#design) (and are easy to render too). *Note on **stability**: not generally true, but holds for lower degree polynomials*.

For **curve segments** in 2D/3D, bound by $u$​​ and represented as:



$$
p(u) = \left[ \begin{matrix} x(u) \\ y(u) \\ z(u) \end{matrix} \right]
= \sum_{k=0}^n u^k\left[ \begin{matrix} c_{x,k} \\ c_{y,k} \\ c_{z,k} \end{matrix} \right] \quad \text{where $c_{\_, k}$ are constants, and } u\in[0, 1].
$$



For **surface patches** in 3D, bound by $u$​​ and represented as:



$$
p(u) = \left[ \begin{matrix} x(u) \\ y(u) \\ z(u) \end{matrix} \right]
= \sum_{j=0}^n\sum_{k=0}^n u_j v_k\left[ \begin{matrix} c_{x,j,k} \\ c_{y,j,k} \\ c_{z,j,k} \end{matrix} \right] \quad \text{where $c_{\_, k}$ are constants, and } u\in[0, 1].
$$



You can obtain $p$ via matrix multiplication as such, where $c = [c_{x,k} \quad c_{t,k} \quad c_{x,k}]^T$.


$$
p(u) = \left[ \begin{matrix} 1 & u & u^2 & u^3 \end{matrix} \right] 
\left[ \begin{matrix} c_0 \\ c_1 \\ c_2 \\ c_3 \end{matrix} \right] = u^{T}c
$$

## Curve implementations

### Cubic interpolating curves


In practice we specify curve segments **geometrically** with control points.

| Type                                                         | Polynomial constraints                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Curve segment**<br /><img src="/notes-blog/assets/img/cg/cubicinter.png" width="50%"> | $p(0) = p_0$​​<br />$p(1/3) = p_1$​​<br />$p(2/3) = p_2$​​<br />$p(1) = p_3$​​. |
| **Surface patch**<br /><img src="/notes-blog/assets/img/cg/cubicinterpatch.png" width="50%"> | Interpolate the points along <br />each red curve respectively. |

**Blending functions** interpolate in a way that satisfies exactly its variations along the edges of a square domain, graphical representation below.

![Cubic interpolating graph](/notes-blog/assets/img/cg/cubicintergraph.png)

$$
\begin{aligned}
p_{0,1,2,3} &= \left[ \begin{matrix} p_0 \\ p_1 \\ p_2 \\ p_3 \end{matrix} \right] = Ac = \left[ 
\begin{matrix} 
0 & 0 & 0 & 0 \\
1 & 1/3 & (1/3)^2 & (1/3)^3 \\
1 & 2/3 & (2/3)^2 & (2/3)^3 \\
1 & 1 & 1 & 1  
\end{matrix} 
\right]
\left[ \begin{matrix} c_0 \\ c_1 \\ c_2 \\ c_3 \end{matrix} \right] \\
\Rightarrow c &= A^{-1}p_{0,1,2,3} \\
\Rightarrow p(u) &= u^{T}c = \textcolor{red}{u^TA^{-1}}p_{0,1,2,3} \\
&= \textcolor{red}{
\left[ \begin{matrix} b_0(u) & b_1(u) & b_2(u) & b_3(u) \end{matrix} \right]}
\left[ \begin{matrix} p_0 \\ p_1 \\ p_2 \\ p_3 \end{matrix} \right] \quad \textbf{(blending functions)} \\
\end{aligned}
$$

### Cubic interpolating patch

$$
p(u,v) = \sum_{i=0}^3 \sum_{j=0}^3 \textcolor{red}{u^iv^j} c_{ij} = u^TCv
$$

where $v = [1 \quad v \quad v^2 \quad v^3]^T$. Note that the interpolating patch does not use the blending functions.

### Geometric and parametric continuity

| Type                                                | Illustration                                                 |
| --------------------------------------------------- | ------------------------------------------------------------ |
| $C^0$​ **parametric** continuity <br />(also $G^0$): | <img src="/notes-blog/assets/img/cg/c0cont.png" width="300px"> |
| $C^1$ **parametric** continuity                     | <img src="/notes-blog/assets/img/cg/c1cont.png" width="300px"> |
| $G^1$ **geometric** continuity                      | $p'(1) = \alpha q'(0)$​ for some $\alpha > 0$​<br />i.e. $p'(1)$​ is parallel to $q'(0)$​ |
| $C^2$ **parametric** continuity                     | $p"(1) = q"(0)$​                                              |

Generally we have

- $C^n \Rightarrow G^n$
- $G^n \not\Rightarrow C^n$
- $C^{n \pm 1} \not\Rightarrow C^n$
- $C^2, G^2$ look smooth.

### Bezier curves

A Bezier curve of degree $n$ is a linear interpolation of 2 corresponding points in Bezier curves of degree $n-1$​. A Bezier curve of degree $n$ has $n$ control points $P_0, P_1, \dots, P_n$:

- $B_{P_0}(t) = P_0$​
- $B(t) = B_{P_0 P_1 \dots P_n}(t) = (1-t)B_{P_0 \dots P_{n-1}}(t) + tB_{P_1 \dots P_n}(t)$​

As such a Bezier curve of degree $n$ has $B'(0) = n(p_1-p_0), B'(1) = n(p_n-p_{n-1})$.​

### Cubic Bezier curves

| Type                                                         | Polynomial constraints                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Curve segment**<br /><img src="/notes-blog/assets/img/cg/cubicbezier.png" width="50%"> | $p(0) = p_0$​​​​​<br />$p(1) = p_3$​​​​​<br />$p'(0) = 3(p_1 - p_0)$​​​<br />$p'(1) = 3(p_3 - p_2)$<br /> |
| **Surface patch**<br /><img src="/notes-blog/assets/img/cg/cubicbezierpatch.png" width="50%"> | Interpolate the points along <br />each red curve respectively. |

$$
\begin{aligned}
M_B &= \left[
\begin{matrix} 
1 & 0 & 0 & 0 \\
-3 & 3 & 0 & 0 \\
3 & 6 & 3 & 0 \\
-1 & 3 & -3 & 1  
\end{matrix} 
\right] \\
\Rightarrow c &= M_B p_{0,1,2,3} \\
\Rightarrow p(u) &= u^{T}c = \textcolor{red}{u^T M_B}p_{0,1,2,3} \\
&= \textcolor{red}{
\left[ \begin{matrix} (1-u)^3 \\ 3u(1-u)^2 \\ 3u^2(1-u) \\ 1 \end{matrix} \right] } \cdot
\left[ \begin{matrix} p_0 \\ p_1 \\ p_2 \\ p_3 \end{matrix} \right] \quad \textbf{(Bernstein polynomials)}\\
\end{aligned}
$$



Blending functions of the cubic Bezier graph are also called Bernstein polynomials (of degree 3). For $0 \leq u \leq 1$, they satisfy:

- $0 \leq b_i(u) \leq 1$
- $\sum_{i=1}^3 b_i(u) = 1$​

The below shows the Bernstein polynomials. Note that the sum $p(u) = \sum_i b_i(u)p_i$​​ is a convex sum, and the curve is contained in a convex hull.

![Bernstein](/notes-blog/assets/img/cg/bernstein.png)

### Bicubic Bezier surface patch

$$
p(u,v) = \sum_{i=0}^3 \sum_{j=0}^3 \textcolor{red}{b_i(u) b_j(u)} p_{ij} = (u^T M_B) P (M_B^Tv)
$$

where $v = [1 \quad v \quad v^2 \quad v^3]^T$​.​ Note that the Bezier patch uses the blending functions.

## Rendering

### Surface patches advantages over polygons mesh

- **Level of subdivision** of a Bezier patch: adaptive to size of image area
  - e.g. when small, less divisions, nearer (and thus bigger), more divisions
  - More efficient than fixed polygons, scalable
- A mesh model will have too many polygons if small (poor **efficiency**), too few polygons if big (poor rendering quality)

### Rendering polynomial curves

Long polynomial curves involve $\leq 4$​ join points. To evaluate $p(u)$ at a sequence of join points $u_k$​, use **Horner's method**:
$$
p(u) = c_0 + u(c_1 + u(c_2 + u(\dots +c_n)))
$$
![polyline to curve](/notes-blog/assets/img/cg/polynomialcurve.png)

### Rendering Bezier curves

**De Casteljau's algorithm**: Consecutive control points are recursively interpolated over $u \in [0, 1]$​, in the case of the curve.

![De Casteljau](/notes-blog/assets/img/cg/decasteljau.png)

## Curve translation summary

| Bezier curves $p$                                     | Interpolating curves $q$                              |
| ----------------------------------------------------- | ----------------------------------------------------- |
| known $c \rightarrow$ find unknown control points $p$ | known $c \rightarrow$ find unknown control points $q$ |
| known $p \rightarrow$​ find unknown $c$​                | known $q \rightarrow$​ find unknown $c$                |
| Simple blending functions ($M_B$)                     | Rather messy blending functions ($M_I = u^TA^{-1}$)   |
| Includes $M_B$​ in bezier patch                        | Doesn't include $M_I$​ in interpolation patch          |

