---
layout: default
usemathjax: true
permalink: /3d/ch2
---

$\require{color}$

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

A contractable edge $ab$ must satisfy $\textcolor{red}{Lk(a) \cup Lk(b) = Lk(ab)}$​​​​​​.

$Lk$ is the link function​.

- $Lk(v)$​ where $v$​ is a vertex: All the **neighbouring vertices** and **surrounding edges** (not edges connected to) of $v$​.
- $Lk(uv)$​ where $uv$​ is an edge: All the **vertices on neighbouring triangles** excluding $u$​ and $v$​.

![Non-contractable case](/notes-blog/assets/img/3d/noncontract.png)

These are sufficient, other operations are a combination of the above. How to do these efficiently?

# Self-intersection

Any operation that moves vertices/edges can cause self-intersection. However we often want our model to be **water-tight** (closed manifold with no-intersection)

### Simplicial complex

No self intersection = mesh is a **simplicial complex**.

Affinely independent

The convex hull of $d$​ affinely independent points is called a d-simplex.

- -1 simplex: empty set.

Given 2 triangles, how do we tell if 2 triangles are intersecting?

- **Na\"ive**: Brute force checking every pair of triangles $O(n^2)$
- **Binary search tree**:  kd-trees for 2 dimensions.



- Octtree: finds the bounding box of a mesh. Adaptive method that stops cutting a region when there's very few vertices in it.



if 2 triangles share a common edge, we don't consider them as intersecting (we can detect by checking the fnexts).

## Precision

### Exact arithmetic

- Based on **Long integers**
- Allowed operations: $+, -, \times$
- Disallowed: $\div, \sqrt{}$​
- **Non-integer values** are represented as **combinations of arithmetic operations**

### Geometric Computation

- Mostly in terms of predicates
- Turn left, turn right or collinear

### Line segment intersection

a, b on opposite sides of c,d. and c, d on opposite sides of a, b.

$\Rightarrow \text{SignedArea}(a,b,d) \cdot \text{SignedArea}(a,b,c)$ or $\text{SignedArea}(c,d,a) \cdot \text{SignedArea}(c,d,b)$

### Speeding up Exact arithmetic

Arithmetic filter: only do exact arithmetic computation when falls within a certain epsilon of 0.
