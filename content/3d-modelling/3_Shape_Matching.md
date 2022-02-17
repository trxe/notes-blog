---
layout: default
usemathjax: true
permalink: /3d/ch3
---

# Shape Matching

## Rigid Motion

*Orientation-preserving isometry* of 3D Euclidean plane.

- Distance preserving
- Angle preserving
- Composition of rigid motion is also a rigid motion.

**Translation**: $f(\mathbf{p}) = \mathbf{p + v}$

**Rotation**: $f(\mathbf{p}) = R\mathbf{p}$

## Iterative Closest Pair (ICP)

For objects $K_1, K_2$, sample $S$ and $R$ 

1. For each $r_i \in R$​​ find closest neighbour in $s_i \in S$​​​​ via kd-trees/octrees etc.
2. Translate centroids of $s_i, r_i$​ for each $i$ to origin
3. **Optimal transformation**: Compute rigid motion $\mathcal{M}$ minimizing $\textcolor{red}{\sum_{i=0}^{n-1}\|\mathcal{M}r_i - s_i\|^2}$​​
4. Repeat from step 1 until this sum converges to some stable value

### Optimal rotation

Summary: To minimize the error $\sum_{i=0}^{n-1}\|\mathcal{M}r_i - s_i\|^2$, is to maximize
$$
q^T\left(\sum_{i=0}^{n-1}\bar{R}_i^TS_i\right)q
$$

Goal is to find $q$​​​. Can do so with **Rodrigues' Formula**.

$\bar{Q}^T Q$ is a rotation by $\theta$ around the axis defined by a vector $(u_x, u_y, u_z)$ if
$$
Q = q = \cos (\theta/2) + \sin (\theta/2)(u_x \mathbf{i} + u_y \mathbf{j} + u_z \mathbf{k})
$$

#### Proof

#### Detailed algo

Build the table matching each $s \in S$ to $r \in R$​ and compute each pair's centroids

 ![](/notes-blog/assets/img/3d/icp1.png)

Froim each vertex, minus its pair-centroid

 ![](/notes-blog/assets/img/3d/icp2.png)

Each vertex is converted into the imaginary part of the quarternion $(1, v_x, v_y, v_z)$.

 ![](/notes-blog/assets/img/3d/qconv.png)

 ![](/notes-blog/assets/img/3d/icp3.png)

Get $\bar{R}_i^TS_i$​ and find the sum across all $i$

 ![](/notes-blog/assets/img/3d/icp4.png)

Find the eigenvector of $\sum_{i=0}^{n-1}\bar{R}_i^TS_i$. Let the eigenvector with biggest eigenvalue be $Q$

Find $\theta$​ and axis $(u_x, u_y, u_z)$ by rodriguez formula.

## Database Matching

e.g. Image matching in Google's database. Comparison of extracted features:

- Colors, Fourier transform
- Text
- AI recognition
- Entropy
- etc.

How to parallel in shape matching with databases?

### Shape distribution/Histogram

Shoot rays from centroid to vertices on a sphere around the object. Generate a (1D or 2D) histogram based on the hit objects.

![](/notes-blog/assets/img/3d/histogram.png)

### Spherical Harmonics

???

Better for rounded simpler shapes.

### Reflective Symmetry Descriptors

Detects how strong is the symmetry across all principal axes, and generates this red thing.

![](/notes-blog/assets/img/3d/symmetry.png)

### Skeleton Graphs
