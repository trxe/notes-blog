---
layout: default
usemathjax: true
permalink: /render/ch6
---

# Frame Buffer Objects

## Framebuffer Data

`glCopyTexImage2D` copies framebuffer data to texture objects.

`internalFormat` refers to either the colorbuffer (`GL_RGBA`) or the depth buffer (`GL_DEPTH_COMPONENT`).

```
glCopyTexImage2D(GL_TEXTURE_2D, 0, internalFormat, 0, 0, viewportWidth, viewportHeight, 0);
```

**Limitations:**

1. Time to copy data from framebuffer to texture object
2. Texture width/height is limited by window size
3. Data format limited by framebuffer format (8bit fixed point for each RGBA channel)
   - e.g. HDR not representable in this RGBA format
4. Cannot output to multiple color buffers in 1 pass

**Texture internal formats:**

## Multipass rendering

Rendering a 3D scene multiple times to collect different intermediate image data, and then combining the images to synthesise final frame.

Limitation: 

- Width and height of texture is limited to window size
- Data format: Limited by the framebuffer's data format (8-bit per channel color bit).
- Cannot output to multiple color buffers
- Fragment shader output clamped to [0.0, 1.0]

## FBO

Allows creation of **non-displayable framebuffers**. OpenGL can redirect output to FBO, each of which contains a collection of rendering destinations.

Many color attachment points allow multiple render targets (MRT) that allow color to be rendered to multiple destinations at once.

![](/notes-blog/assets/img/render/fbo_attach.png)

- Texture Object: Renders to a texture that can be used by shader.
- Renderbuffer Object: Renders offscreen. 

### Texture buffer

```C++
// Generate and bind framebuffer
glGenFramebuffers(1, &fboHandle);
glBindFramebuffer(GL_FRAMEBUFFER, fboHandle);

// Create first texture object
GLuint renderTexA;
glGenTextures(1, &renderTexA);
glActiveTexture(GL_TEXTURE0); 
	// Use the texture unit 0
glBindTexture(GL_TEXTURE_2D, renderTexA); 
	// Bind the texture to 2D mode
glTexImage2D(...); 
	// define texture images that can be used by the shader
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT2, GL_TEXTURE_2D, renderTexA, 0);

// Create second texture object
// ...
glBindTexture(GL_TEXTURE_2D, renderTexB);
// ...

glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT3, GL_TEXTURE_2D, renderTexB, 0);
```

### Renderbuffer 

```c++
// Generate and bind framebuffer, same as above

// Create depth renderbuffer
GLUint depthBuf;
glGenRenderbuffers(1, &depthBuf);
glBindRenderbuffer(GL_RENDERBUFFER, depthBuf);
glRenderbufferStorage(GL_RENDERBUFFER, GL__DEPTH_COMPONENT, fboWidth, fboHeight);

// Bind the depth buffer to FBO
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_RENDERBUFFER, depthBuf);

// Set the target for the fragment shader outputs
GLenum drawBufs[] = { GL_COLOR_ATTACHMENT2, GL_COLOR_ATTACHMENT3 };
glDrawBuffers(2, drawBufs);

// Default framebuffer
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

### Drawing to the texture

```c++
glBindFramebuffer(GL_FRAMEBUFFER, fboHandle);
glViewport(0, 0, fboWidth, fboHeight);
glClear(GL_DEPTH_BUFFER_BIT);

// clear the color buffers
const GLfloat lightGreen[4] = {0.5f, 1.0f, 0.5f, 1.0f};
const GLfloat lightRed[4] = {1.0f, 0.5f, 0.5f, 1.0f};
glClearBufferfv(GL_COLOR, 0, lightGreen);
glClearBufferfv(GL_COLOR, 1, lightRed);

// Setup the projection matrix and view matrix, then render texture
renderTextureScene(); // 1st pass
```

### Final Render

```C++
glBindFramebuffer(GL_FRAMEBUFFER, 0);
glViewport(0, 0, winWidth, winHeight);
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

// Projection View matrix
renderScene();
```

### Fragment shaders 

One for rendering to textures, the other for rendering to the scene.

```glsl
// Fragment shader for render to textures

layout (location = 0) out vec4 FragColorA; // Attachment point 2
layout (location = 1) out vec4 FragColorB; // Attachment point 3

void main() { FragColorA = ..., FragColorB = ... }
```

```glsl
// Fragment shader for final render

uniform sampler2D texMapA; // Texture Unit 0
uniform sampler2D texMapB; // Texture Unit 1
```

# Shadows

How to do it is rasterization? (Without raytracing/radiosity): No direct method in polygon based graphics renderers.

Only generates the shadow shapes, but the color of the shadows itself is highly inaccurate.

- Point lights cause hard edges
- Extended (diffused) lights causes soft edges (umbra, penumbra)

Two main methods: **Shadow volume** and **shadow mapping**, 

- assume point light sources
- "Geometric shadows"  (intensities in the shadow regions are arbitrarily assigned)

## Shadow volume

An **occluder** casts a shadow volume.

A **ray** enters (**+1**) and exits (**-1**) the shadow volume. If any point on the ray is $> 0$​, then it is in one or more shadow volumes.

### Stencil Buffer

- Number of entries and exits from shadow volumes per pixel
- Same resolution as color buffer
- Each pixel is integer counter, can mask color buffer

### Depth Pass method

1. Find silhouette edges

2. Draw volume between light source and shadow silhouette edges
   - Shadow volume polygons enclose the shadow
   
3. Render scene as if completely in shadow (i.e. ambient lighting only).

   - Clear color, depth buffer
   - Clear stencil buffer to all 0
   - Render with ambient lighting

4. Disable writing to depth and color buffers

   - Prevent shadow volume polygons from modifying them
   - Current depth buffer can be used for depth testing shadow volume polygons

5. There are **front facing** and **back facing** shadow volume polygons, 

   - On **depth pass** i.e. occlusion of polygons between viewpoint and shadow applied

   - Draw front faces, increment stencil buffer by 1 on **depth pass**.
   - Draw back faces, decrement stencil buffer by 1 on **depth pass**.

6. Set depth comparison mode to `GL_LEQUAL`

   - Since the depth buffer has been written to before, but we still want to replace those values.

7. Render scene again as completely lit but stencil buffer masks the shadowed areas.
   1. Drawing only on pixels whose stencil value is 0
   2. Only regions where front facing shadow polygons -> object -> back facing shadow polygons will be rendered

![](/notes-blog/assets/img/render/stencil_front.png)

![](/notes-blog/assets/img/render/stencil_back.png)

### Limitation

(Depth pass method) If viewpoint is in shadow volume or too close to shadow volume, the **near plane** clips out the front facing polygons. Results in negative values in the depth buffer, undefined behavior and may result in "holes" in the image.

Fixed with **depth fail** method, which traces from the reverse direction instead (polygon to viewpoint).

## Shadow mapping

Uses texture mapping.

First remember all surfaces visible from light source (equates to all lit surfaces).

Save depth buffer from light source (no shadows, all depth buffer channels are the closest value).

Then on rendering from another view point, compare the depth buffer channel to the saved LightSourceDepthBuffer at position A. 

- if LS[A] > VP[A], then fragment is obscured by light source. 
- else if LS[A] $\approx$ VP[A], then it is visible 

Then map the 3D location of the fragment to the original lightsource view. If the depth values are the same, then the fragment is visible from the lightsource.

1. Render scene with light source as viewpoint
2. Save depth buffer (**shadow map**)
3. Clear the framebuffer
4. Render from the camera's viewpoint
   - For each fragment, change coordinate space from world space to light's view space to determine which fragment it is.
   - If LS[A] > 
