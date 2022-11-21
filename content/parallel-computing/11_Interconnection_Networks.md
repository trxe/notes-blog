---
layout: default
usemathjax: true
permalink: /parallel/ch11
---

# Interconnections

Hardware parallelism

3 parts:
- Cache coherence
- Memory consistency
- Interconnects

Any kind of connection between parts of a computer

## Sorting on linear array

Sorting $N$ unmbers on $N$-PEs linear array

This isn't easy to parallelise. No 

### Odd Even transposition sort

While not in sequence:
- Check left and right neighbours
- Swap left and right if $l > r$.
- $O(n)$.

It is optimal for our Ps to be connected in linear interconnect.

## Sorting on a 2D mesh

# Topology

direct interconnect
indirect (usu for elements of diff types)

diameter max d btw any 2 nodes
degree (small degree reduces hw overhead)

bisection width bg
- total bw btwn 2 halves of a network

node connectivity nc
- min num nodes to disconnect

edge connectivity ec
- 

CCC:
reduces the degree of the hypercube
each original node of  the hypercube becomes a ring
encoding:
- each ring still assigned gray code
- indiv node on Ring has a number

CLockwise and anticlockwise

indirect interconnection: Overview
- Cost: number of switches / links
- Concurrent connections

Bus network

Crossbar network: $n \times m$ switches (worse than direct interconnect)