---
layout: default
usemathjax: true
permalink: /3d/ch9
---

# Skeletal animation

Bone **translations** and joint **rotations**.

- Node: a point of joint rotation
- Arc: Connects two nodes (edge)

**Kinematics**: study of movement positions without considering the forces that brought it into motion.

**Degrees of Freedom**: Set of *independent* translational/rotational displacement specifying an object's pose.

Typical set of DoF for one bone: ${x, y, z, \text{roll, pitch, yaw}}$

Work space: The space in which object's coordinates are defined (usually $R^3, R^2$).

Configuration space: (aka Joint space) The space of possible object **configurations** (usually defined by degrees of freedom).

# Forward Kinematics

Input: Joint Angles
Output: Location of end effectors.

![Forward Kinematics](/notes-blog/assets/img/3d/fk_example1.png)

Depth first traversal of skeleton gives a path to the effector.

$$
T = (L_0R_0)L_1R_1L_2R_2\dots L_kR_k
$$

![FK Matrix](/notes-blog/assets/img/3d/fk_matrix.png)

- $L_0R_0$ is the position and rotation of the root segment.
- Each subsequent $L_k$ is the $k$th link translation
  - Link translation: Frame location relative to parent.
- Each subsequent $R__k$ is the $k$th joint rotation
- Can be achieved with `glPushMatrix, glTranslate, glRotate, draw, ...`

# Inverse Kinematics

Input: Desired location of end effectors
Output: Joint angles required

![Inverse Kinematics](/notes-blog/assets/img/3d/fk_exmaple1.png)
