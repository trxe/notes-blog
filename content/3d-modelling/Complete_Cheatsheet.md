---
layout: default
usemathjax: true
permalink: /3d/cheatsheet
---

# Rotation

Counter clockwise (right hand rule with thumb on arrow side) is positive.

## Rotation Matrix

Rotate about (0, 0, 1) by angle $a$.

- $x' = x\cos a - y \sin a$
- $y' = x\sin a + y \cos a$
- $z' = z$

$$
\left[
\begin{matrix}
\cos \alpha & -\sin \alpha & 0 & 0 \\
\sin \alpha & \cos \alpha & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{matrix}
\right]
$$

### Important Case: Rotate axis $(0, 0, 1)$ to $(1, 1, 1)$

Unit target vector = $\sqrt{1/3}, \sqrt{1/3}, \sqrt{1/3}$

## Axis-angle

Rotate about axis $a$ by angle $\theta$.

### Important Case: Rotate axis $(0, 0, 1)$ to $(1, 1, 1)$

Axis of rotation = normalize(target vector $\times$ orig vector)

$$
\begin{aligned}
& \text{norm}(a_f \times a_0) \\
&= \text{norm}((1, 1, 1) \times (0, 0, 1)) \\
&= (1, -1, 0) / \sqrt{1^2 + (-1)^2 + 0} \\
&= (\sqrt{1/2}, -\sqrt{1/2}, 0)
\end{aligned}
$$

Angle of rotation = $\cos^{-1}$(unit target dot unit orig)

- Unit target vector = $(\sqrt{1/3}, \sqrt{1/3}, \sqrt{1/3})$
- Unit orig vector = $(0, 0, 1)$

$$
\cos\theta = \hat{a}_t \times \hat{a}_0 = \sqrt{1/3}\\
\sin\theta = \sqrt{1 - \cos^2(\theta)} = \sqrt{2/3}
$$

Substitute into the matrix:

![Axis Angle to Rotation Matrix](/notes-blog/assets/img/3d/axisanglematrix.png)

## Quaternions

**Cross product** of two vectors:

$$
\left[\begin{matrix}a_1 \\ a_2 \\ a_3\end{matrix}\right]
\times
\left[\begin{matrix}b_1 \\ b_2 \\ b_3\end{matrix}\right]
= \left[\begin{matrix}
a_2b_3 - a_3b_2 \\ 
a_3b_1 - a_1b_3 \\ 
a_1b_2 - a_2b_1
\end{matrix}\right]
$$

**Multiplication**: 

$$
\begin{aligned}
q_1 q_2 &= (w_1, \mathbf{v_1}) \times (w_2, \mathbf{v_2})\\
&= (w_1w_2 - \mathbf{v_1}\times\mathbf{v_2}, \quad\quad & \text{\small (Scalar)} \\
& w_1\mathbf{v_2} + w_2\mathbf{v_1} + \mathbf{v_1 \times v_2} ) \quad\quad & \text{\small (Vector)}
\end{aligned}
$$