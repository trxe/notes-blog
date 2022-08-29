---
layout: default
usemathjax: true
permalink: /parallel/ch3
---

# Parallel Computing Architectures (Hardware)

How to enable 

- parallel execution and
- accessing memory (bandwidth)

to understand and optimize the performance of parallel programs.

## Concurrency vs Parallelism

| **Concurrency**                     | **Parallelism**                                           |
|-------------------------------------|-----------------------------------------------------------|
| Tasks have overlapping time periods | Tasks can run **simultaneously** exactly at the same time |
| Interleaiving ok                    | Interleaving not happening                                |

Due to the core's pipeline, simultaneous execution is possible at the level of the core
- Multiple data used by instruction
- Multiple instruction can happen at the same time
- Multiple threads can run at the same time

## Forms of parallelism

| Type of parallelism | Description                                         |
|---------------------|-----------------------------------------------------|
| Superscalar         | Multiple instructions from same thread at same time |
| Multithreading      | Multiple threads execute at same time               |
| Multiprocessing     | Multiple threads/processes execute at same time     |

### Bit-level parallelism

Since we mostly operate on multiple bits at once in a single operation, we often have bit-level parallelism.

We often operate on **words**.

**Word size**:
- Unit of transfer between **processor and memory**
- size of a memory addr
- size of an `int`
- size of a single precision `float`

### Instruction-level parallelism

| **Across time** | **Across space** |
|-----------------|------------------|
| Pipelining      | Superscalar      |

![5 stage instruction execution](/notes-blog/assets/img/parallel/l3-datapath.png)

#### Pipelineing

- Fetch - Decode - Execute - (Mem) - Write-back
- Multiple instruction occupy **different stages in one clock cycle**
  - Provided no data/control dependencies
- Max speedup is linked to the number of pipeline stages.

**Problems**:
- Instruction dependency problems
- "Bubbles": the pipeline gets blocked by dependencies etc or repeat work.
- Hazards: due to the control flow we don't have the correct data at the time we need it
- Speculation
- Out of order execution
- Towards 2000 the gains in efficiency from increasing number of stages **tapered off**

#### Superscalar

![Superscalar duplicating the ALU](/notes-blog/assets/img/parallel/l3-superscalar.jpeg)

Duplicates all or some the stages of the pipeline. Instead of 1 instruction in the fetch stage, 
we can have 2 instructions in the stage at the pipeline.

-  **All the instructions coming the same thread in the same core.**
  -  Processor has to find independent instruction to run simultaneously
  -  Finding instructions that can run in parallel on instruction level is **very difficult**.
  -  Scheduling too challenging.

Summary:

![ILP](/notes-blog/assets/img/parallel/l3-instruction-level-parallelism.png)

The speedup is still bottlenecked by the instruction execution, which all comes from the same thread.

### Thread-level parallelism

While software can run multiple threads concurrently, the processor can run multiple threads in **parallel**!
- SMT (Simultaneous multithreading) at the hardware level
  - Hyperthreading (Proprietary to Intel)

#### Implementations

**Fine-grained** multithreading: switch threads per **instruction**

**Coarse-grained** multithreading: switch threads per **stall** (types of stalls below)
- Time-slice MT (switch on predefined time-slide)
- Switch-on-event MT (switch if processor is waiting for event)
- SMT: scheduling insn from **different threads in one cycle**

- Processors with up to 8 hardware threads
  - one core may have a few hardware threads
  - thread = "logical cores"

The processor can also have multiple cores!

### Core-level parallelism

These days we put multiple cores in the same processor. 

- Totally independent execution
- Different contexts
  - Each core has all stages of pipeline, different registers etc.

## Flynn's Parallel Architecture Taxonomy

### SISD (Single insn single data)

1. One stream of instruction
2. One instruction one piece of data
3. Uniprocessors

![SISD](/notes-blog/assets/img/parallel/l3-sisd.png)

### SIMD (Single insn multiple data)

1. One stream of instructions
2. One instruction **multiple data**
   1. Data parallelism / Vector processor
3. Modern processors all have some form 
   1. SSE
   2. AVX
4. GPGPUs will have

![SIMD](/notes-blog/assets/img/parallel/l3-simd.png)

### MISD (Multiple insn single data)

### MIMD (Multiple insn multiple data)

### Stream processor variants (SIMD + MIMD)

SMT:  2 threads with the same context on thesame core at the same time, immediately
- no need  forocntext switch
- 2 threads ready to fire instruction at the same time
- interleaving instructions

Superscalar: 1 thread with one context, to fire instruction from another thread

Kernel thread/Process. A kernel thread is almost like a process today.

## Data Design

## Memory Organizaiton

Memory latency

Memory bandwidth (The bottleneck nowadays!)

Memory bus is fully occupied, cannot give out data faste rthan the bandwidht

- Fetch frmo memory less often
  - Reuse data
  - Share data across threads
  - Store/reload values

DDRx the memory sped is constant, but bandwidth is improving

Cache coherence problem
- Cache coherence protocol to make sure when a value in a cache is modified
- the overhead of updating all of the same piece of data in cache

Memory consistency problem
- Consistency model