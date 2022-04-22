---
layout: default
usemathjax: true
permalink: /render/ch11
---

# Motion Capture

Tracking motion of reference joints.

Joints' positions $\rightarrow$ joint angles which drive 
an articulated model/deformable surface.

## Technologies

**Passive optical systems**:
- Retroreflective markers that reflect light generated near camera lens.
- Sampling of only the bright reflective markers
- Pros: higher fps
- Cons: no glossy/reflective materials, occlusion of markers by props/limbs

e.g. Acclaim Motion Analysis
- 240Hz, non-real time
- 3 markers per body part (to achieve 6 dof)
- 2+ cameras for 3D position
- Expensive ($100K)

**Magnetic systmes**:
- position and orientation calculations via magnetic flux 
- 3 orthogonal coils on transmitter and each receiver
- sensor output is 6DOF
- metal in environment can cause interference
- low frequency (30 to 120Hz), few markers
- Less expensive ($40K)

**Monkey**:
- mechanical puppet with potentiometers measuring flex of joints
- High accuracy and frequency
- But needs animator.

## Processing

- Filtering raw data (e.g. Retargeting)
- Inverse Kinematics application

## Retargeting

Taking captured motion, applying it to **another arbitrary** figure without destroying motion.

Subproblems:
- Cyclification ?? 
- Transitions
- Generalizing motions
- Extracting style
- Scaling

### Process

1. Define **constraints**
2. Apply to a new character
3. Add any translation offsets (obtaining a approximate answer)
4. Solve for constraints from part 1 (non-linear constrained optimization)
   1. e.g. Foot doesn't penetrate through floor/clip through leg etc.

### Motion graph

Selecting a sequence of motions from raw animation data to generate a new animation sequence

**Edges**: The motion clip (e.g. walking, running, squatting, jumping)

**Nodes**: Choice points for possible successor edges

**Transitions**: movements which connect two different animation samples at a node.

To create a transition:
1. Select transition point candidates
2. Elimination of non ideal transition points
3. Blending frames

To extract a sequence of motions: Cast as a graph search problem, with a branch-and-bound method

To synthesise path: estimate actual path $P'$ and compare with desired path $P$,
transforming movement until error towards $P$