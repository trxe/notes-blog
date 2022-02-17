---
layout: default
usemathjax: true
permalink: /3d/ch4a
---

$\require{color}$

# Mesh simplification

## Motivation

Save memory from

1. Oversampled scan data
2. Overtessellation (from isosurfaces, voxel data, or geometric manipulations)
3. LODs (level of detail) for efficient geometry processing
4. Adapt to smaller/weaker display/rendering/memory hardware

## Wishful thinking

Optimize between **minimizing number of triangles** and **minimizing difference in quality**.

- Fairness criteria
  - Normal deviation
  - Triangle shape
  - Scalar attributes

## Vertex clustering

One whole cluster of vertices $\rightarrow$​​ One vertex.

![Vertex clustering](/notes-blog/assets/img/3d/vertex_clustering.png)

1. Gridding the model into a uniform 3D grid recursively.
2. Continue subdividing until each cube has **approximately the same number of triangles**.
3. Cluster these vertices into 1 vertex
   - How to position this new vertex?

![Hierarchical subdivision](/notes-blog/assets/img/3d/grid_sub.png)

### Ways to position

| Average                                           | Geometric median                                             | Error quadrics                                               |
| ------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![Average](/notes-blog/assets/img/3d/average.png) | ![Geometric median](/notes-blog/assets/img/3d/geom_median.png) | ![Error quadrics](/notes-blog/assets/img/3d/error_quadrics.png) |



1. **Average**: Vector sum of deleted vertices and dividing by their count
   - Acts similarly to a low pass filter: smoothens out any sharp edges.
2. **Median**: The point closest to all of the points
   - Similar to subsampling
3. **Error quadrics (best)**: The point that minimizes the sum of distance to all the planes of the triangles.
   - $\sum (v \cdot q)^2 = \sum (v \cdot q)(q \cdot v) = \sum v^Tqq^Tv = \textcolor{red}{v^T \sum qq^T v}$​​
   - Preserves features.

### Retriangulation

After deleting the discard vertices, to fill up the holes we now need to

- Edge contraction without destroying the representative
- Join all the boundary vertices to the representative

Note: this could lead to topology changes, where mesh connectivity changes etc.

![Topological changes](/notes-blog/assets/img/3d/vertex_cluster_topo_change.png)

## Iterative decimation

1. Set up goal (triangle count, error margin, etc.)
2. Choose a single decimation operation for a vertex/edge/triangle from a priority queue (**decimation priority**: which triangles you want to decimate first?)
   - Vertex deletion/Edge contaction/Vertex merging one at a time
3. Modify the priority queue if needed
4. Repeat steps 1 and 2 until goal is reached

Vertex deletion

- DOF on how the retriangulation should occur

Edge contraction

- No DOF, one of the two vertices will act as the "new position"

Merge 2 vertices

- DOF on the exact position of the merged vertex

### Potential issues

1. Change in topology (due to vertex merging): handled by link check
2. Flipped triangles: handled by normal checking of neighbouring triangles

![Flipped triangles](/notes-blog/assets/img/3d/flipped_triangles.png)

### Possible Criteria for deletion

- Shortest edge
- Least difference in volume after decimation
- Curvature
- Triangle quality
- Local error metrics: **Distance** of previous point from new planes
- Global error metrics: Create an envelope before decimation, to guarantee **any shift in position remains within the boundary**
- **Hausdorff distance**: Max distance between all the minimum distances between 2 shapes.
- user-defined: e.g. coloring of faces so the boundaries are less decimated, highlighted regions.

#### Quality

- **Triangle Shape:** Ratio of the circumscribed circle's radius to the shortest edge (bigger is worse)

![Circumcircle](/notes-blog/assets/img/3d/circumcircle.png)

- **Dihedral angles**: Angles between intersecting planes.
- **Valence balance**: Roughly equal degree of vertices