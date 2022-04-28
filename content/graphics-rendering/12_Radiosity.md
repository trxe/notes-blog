---
layout: default
usemathjax: true
permalink: /render/ch12
---

$\require{color}$

# Radiosity Methods `LD*E`

**Diffuse-diffuse** interactions are modelled, with principles of radiative heat transfer.

Scene discretized into perfectly diffuse patches.

View dependent, radiosity map contains **constant radiosity per patch** in scene.

Derived from **Area formulation** of rendering equation.

![](/notes-blog/assets/img/render/radiosity_procedure.png)

$$
B(x) = E(x) + R(x) \int_S B(x') F(x', x) dA \\
B(x)dA_i = E(x)dA_i + R_i \int_j B_j F_{ji} dA_j \\
B(x)dA = E(x)dA + R_i \int_j B_j F_{ij} dA_i \\
\text{radiosity $\times$ area = emitted power $+$ reflected power}
$$

- $R \in [0, 1]$ is reflectivity/albedo.
- $F_ji$ is **form factor** of energy **leaving $j$** and entering $i$.

## Form factors and Reciprocity

$F_{ij}$: Energy leaving $i$ and striking $j$ $\div$ Total energy leaving $i$ in hemisphere.

$$
F_{ij} = \frac{1}{A_i} \int_{A_i}\int_{A_j} \frac{\cos \phi_i \phi_j}{\pi r^2} d_{A_i}d_{A_j}
$$

Approximation: The form factor from $i$ to $j$ can be approximated with the form factor from
$dA_i$ to patch $j$.

$$
F_{dA_iA_j} = \int_{A_j} \frac{\cos \phi_i \phi_j}{\pi r^2} d_{A_j}
$$

$$
F_{ji}dA_j = F_{ij}dA_i $ \le 1 \quad \text{ for all } i,j
$$

### Hemicube

**Delta form factor**: The form factor on each pixel on the hemicube

**Form factor**: $F_{ij} = \sum_k F_k$ for each pixel $k$ covered by projection patch to $j$ on the hemicube.

![](/notes-blog/assets/img/render/hemicube_ff.png)

Computing hemicube delta form factors:
- Take the vector $v$ from bottom-center of hemicube to **center** of the pixel.
- $r = \sqrt{\text{length}(v)}$
- $\cos \phi_i = \text{norm}(v) \cdot (0, 0, 1)$
- $\cos \phi_j = \text{norm}(v) \cdot \text{opposite of face direction }\pm x, \pm y, \pm z$
- Compute area $\Delta A = \frac{2.0}{h} * \frac{2.0}{w}$

![](/notes-blog/assets/img/render/computing_dff.png)

## System of linear equations

From each patch's radiosity equation $B_i = E-i + R_i \int_j B_j F_{ij}$, we can get a system of 
$n$ linear equations that we can represent in a matrix.

![](/notes-blog/assets/img/render/radiosity_matrix.png)

1. Gaussian elimination $O(n^3)$ is inefficient
2. Iterative methods (Jacobi iteration/Gauss-Seidel) $O(kn^2)$ for $k$ iterations

### Iterative methods to solve $Ax = E$

Iteratively compute the equation below for each iteration $k$.

Jacobi:

$$
x_{i, k+1} = (E_i - a_{i1}x_{i,k} - \dots - a_{i,i-1}x_{i-1, k} - a_{i,i+1}x_{i+1, k} - \dots - a_{i,n}x_{n, k})/a_{ii}
$$

Gauss-Seidel:

$$
x_{i, k+1} = (E_i - a_{i1}x_{i,\textcolor{red}{k+1}} - \dots - a_{i,i-1}x_{i-1, \textcolor{red}{k+1}} - a_{i,i+1}x_{i+1, k} - \dots - a_{i,n}x_{n, k})/a_{ii}
$$

## Patch sizes

Shooter patches:
- many, too small: Too much computation
- few, too large and far apart: Banding artifacts due to "multiple point light sources"

![](/notes-blog/assets/img/render/banding_artifacts.png)

Gatherer patches:
- too large: blocky and jagged edges that are not sharp. **Solvable by adapative subdivision/substructuring**

### Adaptive subdivision

1. Use initial patches to compute radiosity
2. If local radiosity variation too large, subdivide that patch
3. Completely recompute radiosity
4. Repeat steps 2 and 3 until local radiosity is small enough.

### Substructuring

1. Use initial patches to compute radiosity
2. If local radiosity variation too large, subdivide that patch
3. Compute only the **form factors** from each new subpatch **to initial patches**
4. Compute radiosity by $B_{iq} = E_i + R_i \sum_{j=1}^n B_jF_{(iq), j}$
5. Repeat steps 2 and 3 until local radiosity is small enough.

## Approaches

## Gathering

![](/notes-blog/assets/img/render/gathering.png)

## Shooting

![](/notes-blog/assets/img/render/shooting.png)