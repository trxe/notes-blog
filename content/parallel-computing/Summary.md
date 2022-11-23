---
layout: default
usemathjax: true
title: Summary
permalink: /parallel/summary
---

# Shared Memory Systems

+ No need to partition data
+ More efficient communication opposed to distributed
- Synchronization constructs
- Lack of scalability due to memory contention

# Examples

**OpenMP**:

+ easy to add parallelism (by just adding compiler directives on top of C/C++ program)
+ No need to copy memory to a separate device
- heavyweight threads
- unrestricted resources: access has to be coordinated and synchronized

**CUDA**:

+ lightweight threads that are numerous, easy to create and destroy
+ reduce memory overheads and contention by exploiting good use of shared memory (only shared amongst threads)
- requires code that can run efficiently in lockstep and is slowed down by conditionals

# Foster's Methodology

**Decomposition**: partition **data** or **tasks**.

Task granularity: Impact on **communication**/**thread formation**

- Fine grained task partition
  - parallelism overhead (creation and merging of threads)
  - communication overhead
- Coarse grained task partition
  - less parallelism
  - but less overhead

**Communication**: local (parallel) or global (sequential)

Rules of thumb:

- Balanced amongst tasks
- Performedin parallel
- Overlap with computation

**Agglomeration**: Combine groups of tasks for sending/receiving

- improve performance
- improve scalability

**Mapping**: Assigning of tasks to execution units

- 

# Parallel Programming models

1. Task pool
2. Parbegin-parend
3. SIMD/SPMD
4. Master-Worker
5. Client-Server/MPMD
6. Task pool
7. Producer Consumer
8. Pipelining

# Metrics

`perf list`:
- branch instructions
- page faults
- cache misses
- cycles
- instructions
- floating point operations

`perf stat`

# Algorithm description

must include:

1. Data distribution
2. Parallel programming model
3. Key constructs (MPI, CUDA, OpenMP)
4. Metrics (utilization of processes, resources, idle time, cache)
5. Interconnection (if distributed)
6. Sources of inefficiency
   1. Waiting/idle time
   2. Overheads
   3. Cache misses/thrashing/memory contention
7. Prevention of deadlock/data race
   1. Odd even often works
  
# CUDA Programming: Memory Management

- Coalescing access to global memory in 32 byte chunks
- Shared memory usage
  - Bank conflict
  - Strided access
- 