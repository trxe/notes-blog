---
layout: default
usemathjax: true
permalink: /render/ch7
---

# Image processing

### Operations

- Blurring
- Sharpening
- Edge detection
- Thresholding
- Mapping
- Warping
- Image reduction

## Approach

blah blah

## Digital Convolution

Replaces a pixel's value with a weighted sum of pixels in the neighbourhood of the pixel (e.g. 3x3). A **convolution filter/kernel** is a matrix that specifies the weights.

![](/notes-blog/assets/img/render/convolution.png)

## Gaussian Blur

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