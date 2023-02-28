---
layout: default
usemathjax: true
title: Concurrency in Go
permalink: /pcp/ch6
---

# Concurrency in Go

Task switching traditionaly used to achieve concurrency.

- **Concurrency**: concurrent **structure** (possible to not be in true parallel)
- **Parallel**: concurrent **execution**

C++ paradigm:
1. Model programs as threads
2. Synchronize memory access
3. Use thread pools

**Go** paradigm:
1. Model programs as tasks
2. Synchronize tasks by **communication**

## Features of Go

1. Memory safety
2. Garbage collection 
3. CSP: Communicating Sequential Processes

## Concurrent design

### Data parallelism

Design && Limiting factor
1. More processes but limited carts
2. More processes and carts but bottleneck @shared receptacle & need to synchronize.
3. More processes and carts and **distributed** receptacle

### Task parallelism

Finer grained concurrency. **Pipeline** parallelism.

## Task dependency graph

DAG
-  node:task
   -  value:expected exec time.
-  edge:control dependency
-  properties:
   -  critical path length: slowest completion time
   -  degree of concurrency

# Performance Speedup

Speedup: best sequential execution time over the parallel execution time.

$$
S_p(n) = \frac{T_\text{best seq}(n)}{T_p(n)}
$$

Amdahl's law: speed up limited by the sequential fraction $f$ 
(time of program that **must** execute sequentially.

## Communicating Sequential Processes (Which are actually **Functions**)

Developed by Hoare 1978.

Refined to **process calculus**.

# Go stuff

## Abstractions

1. **Goroutines** [Concurrency]
   1. Each `go function` runs on OS thread.
   2. Think task in a task pool
   3. Cheaper than threads.
   4. **Same address space** as other goroutines (shared memory)
2. **Channels** [Communication]
   1. Goroutines can write to and read from **FIFO** channel (like a **pipe** in linux).
      1. `channel<-` and `<-channel`
   2. `select` statement
   3. Ensure dependencies are fulfilled

### Goroutines

Runtime multiplexes goroutines onto OS threads. 
Multiple goroutines in a thread "bucket".
**Decouples concurrency from parallelism**.

Scheduling algorithm

When a goroutine blocks, the **thread** blocks.

Goroutines are not garbage collected.

Goroutines need to have join point specified

- Using a **sync.WaitGroup**
- `defer` to exit scope

### Channels

