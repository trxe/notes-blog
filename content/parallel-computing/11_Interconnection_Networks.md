---
layout: default
usemathjax: true
permalink: /parallel/ch11
---

# Interconnections

Back bone for parallelising distributed memory systems

## Sorting on linear array

Sorting $N$ unmbers on $N$-PEs linear array

This isn't easy to parallelise. No 

### Odd Even transposition sort

While not in sequence:
- Check left and right neighbours
- Swap left and right if $l > r$.
- $O(n)$.

It is optimal for our Ps to be connected in linear interconnect.

## Shear sort: Sorting on a 2D mesh

# Direct Interconnection

direct links that form a graph.

## Metrics

1. Degree: 
   1. Affects hardware overhead
2. Diameter:
   1. maximum message transmission distance
3. Bisection width: min. number of edges to **remove** to divide the network into two halves.
   1. Capacity of network to transmit messages at once
4. Node connectivity: min. number of **nodes** that must **fail** to disconnect the network.
   1. Robustness of network
5. Edge connectivity: min. number of **edges** that must **fail** to disconnect the network.
   1. Independent paths

## Different topologies

- Complete/Linear/Ring
- 2D Mesh/Torus
  - XY **routing**: Move in X direction until `srcX == dstX`, move in Y direction until `srcY == dstY`
- Complete binary tree
- Hypercube $n= 2^k$
  - E-Cube **routing**: based on hamming distance
  - While current bit is not the LSB, set current bit to the first different bit, and move to the neighboring node with the bit flipped
- Cube connected cycles (CCC) $n = k2^k$
  - reduces the degree of a hypercube
  - make each original node a ring of size $k$.
    - each ring still assigned a binary code
    - each individual node on the ring has an index
    - numbering can be clockwise or anticlockwise.

# Indirect interconnection

Reducing hardware costs by **sharing switches/links**, dynamic configuration

## Metrics

- Cost: number of switches / links
- Concurrent connections

## Different networks

- **Bus** network: for limited number of processors.
- **Crossbar** networks: configure switches to be straight or direction change
- **Multistage switching** networks:
  - Omega network: ONe unique path for each input to output
    - Connections between stages are regular (patterned)
  - Butterfly network
  - Baseline network

Crossbar network: $n \times m$ switches (worse than direct interconnect)