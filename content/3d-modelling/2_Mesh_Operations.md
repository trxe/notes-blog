---
layout: default
usemathjax: true
permalink: /3d/ch1b
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

### Vertex insertion

A new vertex can be inserted onto 

- a triangle
- an edge
- an existing vertex (bad!)

### Barycentric subdivision

Recursively subdivide into 6 triangles (angles getv small but mathematically easy)

Loop subdivision: add three vertices and cut

Loop subdivision with smoothing

Partial Barycentric

### Edge swapping

Delete 2 triangles and add the opposite edge, recreating two triangles

### Edge contraction

BE CAREFUL! Overalapping edges

- You can only contract an edge $Lk(a) \cup Lk(b) = Lk(ab)$​ Lk is a link function​

These are sufficient, other operations are a combination of the above. How to do these efficiently?

