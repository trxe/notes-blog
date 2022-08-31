---
layout: default
usemathjax: true
permalink: /parallel/ch4
---

# Parallel Programming Models (High level)

## Parallelism

Average num. units that can be performed in parallel per unit time.
- MIPS, MFLOPS performable in parallel per unit itme
  - MIPS: Millions of Instructions per second
  - MFLOPS: Mega Floating-point operations per second

Parallelism constraints:
1. Data/control dependencies
2. Runtime environment
   1. e.g. Memory contention: when **multiple cores issue memory request to the same block at the same time**.

Thus 

$$
\text{Work} = \text{tasks} + \text{dependencies}
$$

memory too  busy, difficult to answer and transfer data

### Data parallelism

Partition data used amongst the processing units, which carries out similar ops on its chunk of data

- SIMD are designed to exploit data parallelism
- If ops are independent, you can execute them in parallel at the same time
- Problems: accessing chunk of data

**Loop parallelism**
- filter the angle of the terrain to spawn the trees and also point up 
- Traverse a large DS as a loop
- If iterations are independent, can be exec. in parallelA

OpenMP

SPMD: 

### Task parallelism

totally different tasks sare executed in parallel

Partition work into tasks: Topological sort

Decompose so that no need for a sequential algo

Task Dependency Graph (a DAG)
- Properties
  - Crotocal path length (shoter is better[[[]]])
  - Degreen o f ocncurrency (bigger is better $crticial path total / )

TAs grading qns 1 to 5:
- TAs grade different questions (different tasks)

if processor handle same chunk of data: data parallelism

mapping: map to which cores?

### Overheads

- Starting a parallel task
- Coordination
  - Shared addr space
    - need hw support to implement efficiently
    - confusing of shared addr space with shared memory system
      - best impl: addr space implmeneted wiith shared memort syystem
      - BUT shared addr space can also be done on a distributed system
  - Data parallel 
  - Msg passing

### Data Parallel

- rigid computation structure
- Same operation on each element of an array. 
- MAPP A FN onto a LARGE DATA colelction
  - for looop or functional side-efficet free
- GPGPU with CUDA programming
- lockstep

### Message passing

- Tasks operate within their own private address spaces
- Explicitly send/receive messages
- MPI (message passing interface)

## Parallel Programaming Models

Fine/Coarse grain
- Instructions in sequences --> Function/Method as a chunk of statements

Foster's Desing Methodology

1. Partitioning
   1. Data/Computation centric $\Leftrightarrow$ Data/Task parallelism
2. Comms
3. Agglomeration
4. Mapping

Partitioning:
- 10x more tasks than cores
  - you want many tasks on the cores, so that if one blocks, you can easily find another to execute
- tasks approx same size
- minimize redundant computation
- tasks num should increase with problem size

Communication:
- Balance amongst tasks
- Each task only comms with a group of neighbors
- Tasks can perform comms in parallel
- Overlap computation with communication (some tasks comms while others compute)

Agglomeration: Combining tasks into larger tasks
- Reduce cost of task creation+ communication (send/receive)
- Maintaining stability

Mapping: 2 diverging goals 
- maximize processor utilitzation
- minimize inter-processor communication

- Optimal mapping (NP-hard) heuristics
- One task per core VS Multiple tasks per core
  - problem size decreases
- Static VS dynamic task allocation
  - static: task to cores should be 10:1

Automatic parallelization:
- decomposition and scheduling
- drawbacks:
  - pointer-based computations or indirect addressing cannot 
  - execution time or fn calls without known bounds is very difficult to predict.
  - We aren't there yet

Functional Programming:
- fns that areindependent of each other, easier to find independent tasks
- dk how long the recursive (or even otherwise) fn takes
- extract the right level of recursion

### Models of Coordination

### Program Parallelization