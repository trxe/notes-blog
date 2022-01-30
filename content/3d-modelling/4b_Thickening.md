layout: default
usemathjax: true
permalink: /3d/ch4b

# Mesh Thickening

Lifting up the surface.

Methods

1. Move up each vertex according to **triangle normal**
   1. Overlaps in vertices
2. Move vertex according to **shared vertex normal**

Smoother methods:

1. Concave vertices: move along **shared vertex** normal
2. Convex vertices: move alone **triangle** normal

Problems:

Thickening could lead to **self-intersection**. But do we always want to solve it (i.e. do we insist on the topology)?

- Yes