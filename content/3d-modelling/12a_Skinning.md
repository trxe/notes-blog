---
layout: default
usemathjax: true
permalink: /3d/ch12a
---

# Skinning

Secondary motion. Binds a skeleton to a mesh and deforming the mesh as the skeleton moves.

Rigging: Building a skeleton and binding it to a skin mesh.

One possible method:
- Pack spheres onto a mesh
- Construct a graph based on each centroid as a node
- Find the graph that most similarly resembles the target skeleton
- Continuous optimization
- Diffusion equilibrium equation on character surface to compute **bone weights**
  - For character subsurface deformation
- Online motion retargeting eliminates foot skate

May need/have
- Manual assignment of some joints to correct
- Cannot differentiate between materials
- Starting orientation to be similar to actual

## Rigid skinning

ONe vertex to one bone

## Linear blend skinning (LBS)

Transformation of vertices by a blend of multiple transforms.
- Bind matrix
- $p_{b,i}$ is the position of mesh vertex in bone frame for bone $i$
- $p_{w,t}$ is the psoition of mesh vertex (deformed) in world frame at time $t$
- $T_{i,t}$ is the transformation matrix at time $t$ for bone $i$

$$
p_{w,t} = \sum_{i=1}^n  w_iT_{i,t}p_{b,i}
$$

Linear blending of matrices becomes a general affine transformation, causing pinching artifacts.

## Example based skinning

> Example-based skinning methods optimize the structure of the skeleton hierarchy 
> and skinning weights so that the skinned animation approximates the example shapes well.
> - Tomohiko Mukai, Handbook of Human Motion

### Blendshapes

Runs in realtime (used in games), morph targets/shape interpolation

$$
f_j = \sum_k w_k b_{kj}
$$

- $f_j$ is the final position of jth vertex of model,
- $w_k$ is blending weight of shape $k$, sums to 1.0
- $b_{kj}$ is the position of vertex $j$ relative to shape $k$

### Pose Space deformation

Skeleton driven deformation: deforms mesh shape according to joint angle

Rather than interpolate in time, as with animation curves, or over space, as with meshes, 
PSD views animation as interpolation over the domain of the character's pose.

$$
\Phi(x) = e^{-\frac{x^2}{2\sigma^2}}
$$

Radial Basis Function (RBF): A real valued function $\phi$ whose value is only dependent 
on distance between input and a fixed point (an origin, or some other center).

Other types:
- Dual quaternion
- Physically based skinning