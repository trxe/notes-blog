---
layout: default
usemathjax: true
permalink: /3d/ch2
---

# Mesh Modification

Two types of **topology** (i.e. connectivity):

- Mesh connectivity
- Surface connectivity

Hence mesh modification is **geometric** (i.e. no changes to mesh connectivity but can change underlying surface.)

- UV map
- Smoothing
- *Note: possible to destroy topology!*

Whereas surface modification is **topological** (i.e. no changes to the underlying surface, but allow changes within mesh connectivity.)

.Underlying space: "the structure" of the object.

## Vertex insertion

A new vertex can be inserted onto 

- a triangle
- an edge
- an existing vertex (bad!)

### Barycentric subdivision

Recursively subdivide into 6 triangles (angles get very small but mathematically easy to do.)

A type of vertex insertion; new vertices are inserted into the centroid of the subdivided triangles.

![Barycentric subdivision](/notes-blog/assets/img/3d/barycentric_sub.png)

### Loop subdivision

Add a vertex to each edge and cut. Can be done with and without smoothing.

![Loop subdivision](/notes-blog/assets/img/3d/loop_sub.png)

### Partial Barycentric

![Partial Barycentric subdivision](/notes-blog/assets/img/3d/partial_bary.png)

## Edge swapping

Delete 2 triangles and add the opposite edge, recreating two triangles. Used for mesh refining

![Edge swap](/notes-blog/assets/img/3d/edge_swap.png)

## Edge contraction

- Merge two adjacent vertices
- Delete two original triangles and extra edges
- Stitch the empty hole

BE CAREFUL! Overlapping edges may result in some cases.

![Edge contraction](/notes-blog/assets/img/3d/edge_contr.png)

You can only contract an edge $Lk(a) \cup Lk(b) = Lk(ab)$​ Lk is a link function​

These are sufficient, other operations are a combination of the above. How to do these efficiently?

