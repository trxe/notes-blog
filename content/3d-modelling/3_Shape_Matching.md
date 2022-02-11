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

## Iterative Closest Pair (ICP)

For oibjects $K_1, K_2$, sample $S$ and $R$ 

1. For each $r_i \in R$​​ find closest neighbour in $s_i \in S$​​​ via kd-trees/octrees etc,
2. Translate centroids of $S, R$, to origin
3. **Optimal transformation**: Compute rigid motion $\mathcal{M}$ minimizing $\textcolor{red}{\sum_{i=0}^{n-1}\|\mathcal{M}r_i - s_i\|^2}$​​
4. Repeat from step 1 until this sum converges to some stable value

### Optimal rotation

summary: To minimize the error $\sum_{i=0}^{n-1}\|\mathcal{M}r_i - s_i\|^2$, is to maximize
$$
q^T\left(\sum_{i=0}^{n-1}\bar{R}_i^TS_i\right)q
$$
