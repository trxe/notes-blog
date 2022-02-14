---
layout: default
usemathjax: true
permalink: /render/ch6
---

# Frame Buffer Objects

## Multipass rendering

Rendering a 3D scene multiple times to collect different intermediate image data, and then combining the images to synthesise final frame.

Limitation: 

- Width and height of texture is limited to window size
- Data format: Limited by the framebuffer's data format (8-bit per channel color bit).
- Cannot output to multiple color buffers
- Fragment shader output clamped to [0.0, 1.0]

## FBO

Allows creation of non-displayable framebuffers. OpenGL can redirect output to FBO, each of which contains a collection of rendering destinations.

Texture Object

Renderbuffer Object

# Shadows

HOw to do it is rasterization? (Without rayytracing): No direct method in polygon based graphics renderers.

Only generates the shadow shapes, but the color of the shadows itself is highly inaccurate.

## Shadow volume

?

## Shadow mapping

Uses texture mapping.
