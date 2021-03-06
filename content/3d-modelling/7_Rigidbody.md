---
layout: default
usemathjax: true
permalink: /3d/ch7
---

# Introduction to animation

Conventional animation principles:
- anticipation, action, follow-through
- squash-stretch
- timing
  - Speed conveys mass and personality
  - Traditional animators would draw a timechart with a time scale to indicate the generation of in-betweens

**Keyframe**: position defining the start and end points of any smooth transition

**Computer-assisted animation**:
- Automation of in-between generation
- Viewed as a set of 2D functions $y=f(x)$

## Position interpolation

Given $t \in [t_i, t_{i+1})$,

$$
x(t) = x(t_i) \frac{t_{i+1} - t}{t_{i+1} - t_i} + x(t_{i+1}) \frac{t - t_i}{t_{i+1} - t_i}
$$

Interpolation can happen on rotation angle and/or position.