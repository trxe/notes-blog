---
layout: default
usemathjax: true
permalink: /parallel/ch9
---

# Data distribution

**Decomposition** of array

# Data distribution of 1D

1. Blockwise ($P_j$ takes elements $\left[(j-1)B \dots JB - 1\right]$)
   1. Exploits **locality**
   2. If you know the size of your input, blockwise is best
2. Cyclic ($P_J$ takes $\left[j, j + B, j + 2B \dots j + (c-2)p\right]$) where $c$
   1. Better **load balancing**
   2. Cache misses on almost every access, thrashing

# Data distribution of 2D

1. Blockwise column
2. Blockwise row
3. Cyclic column
4. Cyclic row
5. Block-cyclic with block size $b$ columns/rows
6. Checkerboard
   1. Blockwise
   2. Cyclic
   3. Block-cyclic

Developing a solution in MPI:
1. Data distribution
2. Topology
   1. consider utilization: are all links utilized roughly equally?
3. Task distribution (what each node does)
   1. Is task granularity appropriate?
4. Communication