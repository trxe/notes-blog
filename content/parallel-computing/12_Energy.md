---
layout: default
usemathjax: true
permalink: /parallel/ch12
---

# Energy Efficient computing

**Motivation**: 

1. Pack more transistors on a chip
   1. Too many transistors, core becomes too hot
2. Pack more cores on a chip
   1. The socket gets too hot
3. Around 1997, cooling was required for chip
4. How to dissipate power??

Energy efficiency required in 

- Mobile computing (high powered screen, need high battery life)
- Enterprise computing

### Heterogenous computing: big.LITTLE

- Fast processors
  - Run high performance tasks
- Slow processors
  - Run small, sequential background tasks

Nowadays hardware itself can migrate tasks to another core when running low performance tasks.

# Modern HPC Infrastructure

### Data Centers

A data center is a facility that centralizes an organization's shared IT operations and equipment for the purposes of storing, processing, and disseminating data and applications

Achieves heterogenous computing with different **server generations**

### Cloud Computing

**Delivery** of computing services (including servers, storage, databases, networks, software etc.) **over the internet**.

- Can be physically located on a data center
- Or owned and operated remotely by a 3rd party provider

## How to reduce Energy Consumption

1. Move less data
   1. **Reduce data transfer** to/fro memory
   2. Exploit **locality**
   3. **Compression**
2. Specialized processing:
   1. Compute less (Avoid parallelizing)
   2. CPU-like cores + GPU-like cores