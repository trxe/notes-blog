---
layout: default
usemathjax: true
permalink: /parallel/ch6
---

# GPGPU Programming

## GPU History

First GPUs were used to run shader programs, which were very customized to graphics pipeline.
Drawbacks included

- Hard to transfer data
- No **scatter** (i.e. memory write into a computed address, memory writes only into fixed addr.)
  - see MPI Scatter/Gather
- No communication between fragments
- Coarse thread synchronization
  - Fine-grained: possible to sync between instructions
  - Coarse-grained: sync only on thread termination

## GPU Architecture

One GPU has many **Streaming Multiprocessors** (SMs).

![Architecture of an SM](/notes-blog/assets/img/parallel/Architecture-of-a-streaming-multiprocessor-185.png)

Different compute capabilities have different SM architectures.
- How many SMs in GPU?
- How does each SM look like?
  - How does each core look like
  - How many FP, INT, SFU cores?
    - **SFU**: for special math functions e.g. $\sin$, $\cos$, $\sqrt$ etc.
  - Each core only do FP or only do INT operations?
  - Your FP cores are single precision (32b) or double (64b)?

The cores in an SM share:
1. Common L1 cache
2. Texture memory
3. Warp scheduler

# What is CUDA

Compute Unified Device Architecture (CUDA): is a general purpose programming model.

- Simple C extensions
- With mature software stack (high/low level access)
- **Launches batch threads**
- Load/store memory model
  - ISA that divides instructions into **memory access (load/store)** and ALU operations
  - CRCW: Concurrent read concurrent write

![CUDA Layers](/notes-blog/assets/img/parallel/cuda-layers.png)

**C $\rightarrow$  CUDA text intermediate representation (PTX) $\rightarrow$ device specific code (CUBIN)**

CUDA **C Runtime**:

- Minimal set of C language extensions
- Kernels are **C functions** in src code
- **Built on CUDA driver API**

CUDA **driver API**:

- Low-level C API to **load compiled kernels**
- Inspect parameters and launch

CUDA **linkers**
- cudart (runtime library)
- cuda (core library)

## Definitions

**Kernel**: Function running on device
- Older device: one kernel is executed sequentially
- Modern device: kernels can run async

**Host**: CPU

**Device**: GPU

**Thread**: thread of execution on an SM

- little creation overhead
- instant switching
- thread id

**Block**: All threads are organized in groups called blocks. 
One kernel is executed by a **grid** of thread **blocks** (note this config is **virtual**).
Threads in a block cooperate via

1. Shared memory
2. Atomic operations
3. Barrier sync

Programs can thus scale transparently to any number of processors.
Threads beyond a block cannot cooperate.

**Transparent scalability**: HW is free to schedule threads to **any processor at any time**.
- Kernel scales across any number of parallel multiprocessors
- Several blocks can reside concurrently on an SM.
- But running **too many kernels** on **one SM** will cause kernel to get stuck.
- SM resources also limited
  - Register file
  - Shared memory

## Thread mapping and architecture

SIMT (Single instruction multiple thread) execution model

SM creates/manages/schedules/executes threads in **warps**

**Warp**: Group of 32 parallel threads.

- Start together at the **same program address**
- But have **individual program counter/register states**
- Block --> warps always same way
- Hence block size should always be in multiples of 32.

## Memory model

On-chip device memory (on the SM):
- Registers
- Shared memory
- Const/Tex caches (aggressively cached) via L1 cache

Off-chip device memory:
- Local
  - array variables
- Global
- Constant memory
  - Uniform access readonly
- Texture memory
  - Spatially coherent random-access readonly data
  - Filtering, address clamp and wrapping

**Coalesced access to global memory**: threads in a warp (set of 32 threads) coalesced into a number of transactions.

- **number of transactions** = number of 32-byte transactions required to service all threads of that warp.
    - $k$th thread accesses the $k$th word.
**Strides**: If thread's global memory accesses with a stride of $k>1$ words within a warp, reduced efficiency/bandwidth
  - Stride of 2 words: 50% load store efficiency/bandwidth

# Optimization

- Memory bandwidth
  - Shared memory exploit
- Parallel execution
- Instruction throughput
  - High throughput arithmetic instructions
  - **Avoid branching** instructions **within a warp**.

## Memory optimization

1. Minimize host-device data transfer
   1. Peak on-off chip bandwidth >> Peak device-host bandwidth
   2. **Page-locked**pinned
2. Coalesce global memory accesses
3. Minimize global memory accesses
4. Minimize bank conflicts
5. Page locked (pinned) memory transfer
   1. Not cached
   2. Zero-copy that accesses directly host memory
   3. In CUDA its `__managed__` memory (Unified memory model)

![Pinned memory](/notes-blog/assets/img/parallel/pinned-memory.png)

## Execution config

Occupancy vs resource utilization

Avoid multiple contexts per GPU with same CUDA application.
*(i.e. avoid running multiple CUDA application processes as it means multiple contexts)*

# Instruction throughput

Max out fast arithmetic instructions!

- Trade precision for speed
- **Floats** provide best perf
- Integer division and modulo are **slow**
  - Use bitwise
- Signed loop counters (`int`) cos they are optimized aggressively by C
- use `int` instead of `char` or `short` if possible

Minimize warp divergence by control flow.

Optimize out sync points.
