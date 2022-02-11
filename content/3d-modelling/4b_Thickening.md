---
layout: default
usemathjax: true
permalink: /3d/ch4b
---

# Mesh Thickening

Thickening out a single sheet of triangulation. (For medical purposes etc.)

## Methods

1. Move up each vertex according to **triangle normal**
   - Moves the edges up
   - Edge gaps when convex
   - Vertex overlaps when concave
2. Move vertex according to **shared vertex normal**
   - Sharpness of edges

Smoother methods:

1. Concave vertices: move along **shared vertex** normal
2. Convex vertices: move along **triangle** normal\
   - How to fill in the gaps? See [offsets](#offsets)

![Thickening](/notes-blog/assets/img/3d/methods_thick.png)

## Offsets

![Offset edges](/notes-blog/assets/img/3d/offset_edges.png)

![Offset vertex](/notes-blog/assets/img/3d/offset_vertex.png)

## Problems

Thickening could lead to **self-intersection**. But do we always want to solve it (i.e. do we insist on the topology)?
