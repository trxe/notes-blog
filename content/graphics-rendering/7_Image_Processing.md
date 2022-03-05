---
layout: default
usemathjax: true
permalink: /render/ch7
---

$\require{color}$

# Image processing

### Operations

- Blurring
- Sharpening
- Edge detection
- Thresholding
- Mapping
- Warping
- Image reduction

## General Approach

- Pass #1:
  1. Bind to FBO
  2. Render 3D scene to a texture
- Pass #2:
  1. Bind to **default framebuffer**
  2. Draw **screen-filling** quad
  3. Each fragment performs operation to texture from **pass #1**

## Digital Convolution

Replaces a pixel's value with a weighted sum of pixels in the neighbourhood of the pixel (e.g. 3x3). A **convolution filter/kernel** is a matrix that specifies the weights.

![](/notes-blog/assets/img/render/convolution.png)

## Edge detection

Sobel operator. Apply $S_x, S_y$ to the 3x3 pixel grid.

$$
S_x = \left[ \begin{matrix} 
-1 & 0 & 1 \\
-2 & 0 & 2 \\
-1 & 0 & 1 \\
\end{matrix} \right], \quad
S_y = \left[ \begin{matrix} 
-1 & -2 & -1 \\
0 & 0 & 0 \\
1 & 2 & 1 \\
\end{matrix} \right]
$$

If $\textcolor{red}{g = \sqrt{s_x^2 + s_y^2} > d}$ where $d$ is some threshold, then it is an edge pixel.

1. Setup FBO (same dimension as main window)
2. Select the FBO and clear color/depth buffers
3. Feed values to the uniforms 
4. Set `passNum` to 1
5. Draw scene (first pass)
6. Deselect FBO and revert to default framebuffer
7. Set the modelview and projection matrices to identity $\mathbf{I}$
    - Now in NDC space.
8. Feed values to uniforms
9. Set `passNum` to 2
10. Draw in a single quad (**second pass**)
    - `texelFetchOffset(texture, pixCoord, 0, offset)`
    - Not affected by filtering, mipmapping, etc., looks up texel only.
    - Not the same as `texture2D`. 

```glsl
// rounding down xy clip coords to get the pixel position (screenfilling quad.)
ivec2 pix = ivec2(gl_FragCoord.xy);
// texel coordinates of texture map == fragment position in output image.
// the texel values from the render texture.
float s00 = luminance(texelFetchOffset(RenderTex, pix, 0, ivec2(-1, 1)).rgb)
...
float s22 = luminance(texelFetchOffset(RenderTex, pix, 0, ivec2(1, -1)).rgb)

float sx = s00 + 2 * s10 + s20 - (s02 + 2 * s12 + s22);
float sy = s00 + 2 * s01 + s02 - (s20 + 2 * s21 + s22);
float g = sx * sx + sy * sy;
```

*Note on fragment position*: 
- A fragment's 2D position is coordinates of fragment's center in window space. 
  - bottom most (0.5, 0.5)
- A texel coordinate is integer values
  - bottom most (0, 0)

## Gaussian Blur

Lens blur/bloom useful.

The Gaussian Kernel/Filter is defined as such:

$$
G(x,y) = \frac{1}{2\pi\sigma} e^{-\frac{x^2 + y^2}{2\sigma^2}}
$$

It i s a production of two 1D Gaussian functions , i.e. $G(x,y) = G(x)G(y)$.

Hence the 2D digital convolution:

$$
C_{l,m} = \sum_{i= -4}^4 \sum_{j=-4}^4 G(i,j) C_{l+i, m+j} 
$$

### Efficient Gaussian Blur

1. First pass: Compute sum over $j$ (vertical) and store the result in a temporary texture, $O(n)$ for $n$x$n$ matrix.
2. Second pass: Compute sum over $i$ (horizontal) with the results from the previous pass, $O(n)$ for $n$x$n$ matrix.

### Approach

1. 