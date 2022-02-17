---
layout: default
usemathjax: true
permalink: /3d/ch5
---

# Surface Reconstruction

We have an implicit function $f(x,y,z)$ defining a volume. The surface is defined by $f(x,y,z)$ and we can generate either a point cloud or voxels at uniform intervals. How to use this data to reconstruct a mesh surface?

# Voronoi Complexes

Let $S$ be a set of vertices $\{p_1, p_2, \dots\}$.

The **Voronoi region** $v_i$ of $p_i$ is defined as
$$
\nu_i = \{ x \in \mathbb{R}^d \mid \pi_i(x) \leq \pi_j(x), \forall p_j \in S\}
$$
i.e. the set of all points closer to $p_i$​ than to any other point in $S$.

Intersections of all the bisected spaces between every 2 points.

1. Every point in S has a non-empty Voronoi region
2. Every Voronoi is convex
3. Voronoi region can be bound or unbound. It is unbound if the point is on the convex hull of $S$.
4. The intersections of the interiors of Voronoi regions are empty.
5. $\bigcup_{p_j \in S} \nu_j = \mathbb{R}^d$​​​

The **Voronoi cell** is the intersection of the Voronoi regions of vertices in subset $X \subseteq S$​.
$$
\nu_X = \bigcap_{p_i \in X} \nu_i
$$
The **Voronoi complex** of $S$​ is the set of all of all of its Voronoi cells (all non-empty faces of $S$).
$$
\nu_S = \{\nu_X \mid X \subseteq S, \nu_X \neq \emptyset \}
$$

# Delaunay Complexes

Empty circumdisk: No point $p \in S$​ lies in the interior of any Delaunay triangle's circumcircle.

A Delaunay Complex is defined as such
$$
D_S = \{ \text{convexHull}(X) \mid \nu_X \in \nu_S\}
$$
No $d+2$ points are spherical if the Voronoi Complex is in $\mathbb{R}^d$. *(e.g. in 2D space we will get a quad.)*

## Restricted Delaunay Triangulation

A Delaunay edge is added between 2 vertices $p_i,p_j$​​ only if the Voronoi cell $\nu_{ij}$​ intersects the surface of the implicitly defined mesh (usually can identify by checking if $f(u)f(v) < 0$​, where $u,v$​​ are the extreme points of the Voronoi cell).

## Topological Ball Theorem

For each Voronoi cell, if it intersects the surface as a topological ball, then the RDT is homeomorphic to the surface

- 1D ball: line (curve segment)
- 2D ball: manifold with boundary

# Voxelization

Isosurfaces are the surfaces where all the points on that surface are of a certain value.

At fixed intervals the implicit equation is computed with those coordinates to generate a scalar field, on which we can run Marching Squares/Cubes algorithm to reflect the separation at the isosurface (change in values on that scalar field).

## Marching Cubes

$2^8$​ combinations of black/white assigning of vertices in a cube.

However only 22 unique ones, the rest are all symmetries

Many ambiguities: caused by hole formation and/or symmetrical division.

