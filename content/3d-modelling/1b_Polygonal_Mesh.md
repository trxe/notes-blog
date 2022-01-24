---
layout: default
usemathjax: true
permalink: /3d/ch1b
---

# Object representation

How to represent a sphere?

- Polygonal (Triangular soup) mesh
- Parametric patches
- Density map
- Implicit equation ($x^2 + y^2 + z^2 = 1$ in raytracing)

### Polygonal Mesh

A polygonal mesh is merely an approximation. The question is triangle quality:

- how dense should our polygons (triangles) be?
- triangle minimal angle

Bezier curves alone cannot represent a circle. 

### Constructive solid geometry

A set of primitive shapes combined with different operations in infinitely many ways

- Useful for raytracing. You don't have to compute the shape, you check if the light ray hits the primitive (ANDing a bunch of booleans)
- Not easy to do with meshes.

### Spatial subdivision techniques

- 3D array of voxels
- If object occupies a voxel space, turn voxel on

## Mesh

- A collection of vertices, edges and triangles
- Each edge belongs to one face (no dangling edge)
  - Edge is a boundary edge if it only belongs to one face
  - Can more than 2 faces share one edge? Technically yes, but not manifold
- Each vertex belong to one edge

### Manifold

A mesh is a manifold 

- if every edge is adjacent to at most 2 faces.
- if every vertex is either on a disk or half disk (i.e. the faces the vertex is adjacent to can be flattened into a 2D surface)

![Non-manifold](/notes-blog/assets/img/3d/non-manifold.png)

A mesh is a polyhedron if

- it is a manifold and has no boundary (is closed)
- every vertex is on a disk 
  - vertex belongs to a cyclically ordered set of faces (local shape is a disk)

Orientation of faces (counterclockwise order = normal face)

A mesh is **well-oriented** if

- all faces can be oriented consistently (all CCW/CW) such that each edge has two opposite orientations

![Well-oriented definition](/notes-blog/assets/img/3d/well-oriented-def.png)

### Euler Poincare Characteristic $\Chi$

Defined for the surfaces of polyhedra as $\Chi = V - E + F$

- Any sphere like shape: $\Chi = 2$.
- Torus: $\Chi = 0$
- Bow Torus: $\Chi = -2$.

See Poincare Conjecture.

## Mesh representations

### Independent Faces

Vertex lists (3 `double`s per triangle in an array)

- Repeat vertex (disk space waste)
- No topological info
  - How to find the faces that share an **edge** with a particular face? $O(N)$ search
- Floating point errors (not the same point, but comparison leads to identifying same point)

### Indexed Face set

- Still no topology information
- But saves repeat vertices

### Adjacency Lists

3 adjacency lists: for vertices, edges and faces

- For each vertex/edge/face: All vertices, edges and faces adjacent

![Adjacency lists](/notes-blog/assets/img/3d/adjacency_lists.png)

### Trist 

Oriented triangles: Adding an arrow to identify edge face orientation. (ABC = face with vertices face up in order A, B, C). They may be the same triangle, but are different "versions"

`enext`: "rotate" different orientation of the triangles in direction of the arrow

`sym`: "flips" the arrow

`org(triangle)`: returns the first vertex of the oriented triangle

In $\mathbf{R}^3$, an edge can be shared by many triangles.

`fnext`: "rotate" orientation of the triangles in the right hand rule direction

- The arrow shouldn't ever change (`fnext` of abc should always be ab_)

Triangle has two representation:

- Vertex array (no orientation/version)
- we only have 6 versions of a triangle! How to concatenate the version number into one integer that can be stored with the vertex index number in the oriented triangle
  - `oriented_tri = (tIdx << 3) | version_number`

![Trist table](/notes-blog/assets/img/3d/tristtable.png)
