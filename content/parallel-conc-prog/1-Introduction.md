---
layout: default
usemathjax: true
permalink: /pcp/ch1
---

# Introduction

Concurrency vs Parallelism (again):

**Concurrency**

- two or more executions in progress at same time.
- but actually may not be happening at the same instance in time
  - illusion of concurrency / true concurrency
  - traditionally thru task-switching

**Parallel**

- tasks actually simultaneously execute
- only possible with 
  - multiple cores
  - multiple **hardware threads** per core
    - simultaneous multithreading = multiple hardware threads

Multiple processes vs Multiple Threads:

**Processes**:
- More safety (separate Process Control Blocks)
  - different pid, memory region info (TEXT, DATA, HEAP, STACK)
- context switch is rsc demanding
- no race conditions as **separate memory spaces**
- creation (via forking) is costly (clone entire PCB)

**Threads**:
- more performance as shared address space 
  - independent control flows
    - different PC, SP, registers to be swappedout
  - different STACK portions, but TEXT, DATA, HEAP same
    - **race conditions** on heap data/shared STACK data possible.
  - optimization strategy
  - take advantage of hardware threads
- lightweight, less rsc demanding
- multiple threads can be a part of a single process
  - separation of concerns

Exceptions and Interrupts:

**Exceptions**
- machine instruction execution exception
- synchronous (due to program execution)
- exception handler

**Interrupts**
- independent of execution, some external signal
- asynchronous (not due to execution)
- interrupt hanlder

User level thread vs Kernel thread

**User-level**
- library thread, user API for the kernel thread 
- mapped to 1 or more kernel thread
    - can be other types of mapping than 1 to 1

**Kernel thread**
- the ones actually managed by OS
- can actually run in parallel

## Issues in concurrency

### Race conditions

**Race condition**: Different ordering of concurrent events may 
leading to different (one of them incorrect) program execution.

**Data race:**
1. Two threads access shared resource at the same time without synchronization
2. At least one modifies the shared resource

Solution: synchronization (especially on shared buffer/queues, lists, hash table)

### Mutual Exclusion and Correctness

Code sequence protected with mutex is **Critical Section**.

Crit. Section Requirements:
1. Mutual Exclusion
2. Progress (if T not in c.s., then T cannot stop S from entering c.s.)
3. No starvation/Bounded wait (if S wants to enter, it eventually can)

Correct Concurrent Program Requirements:
1. Safety: 'nothing bad happens'
   1. end with invalid state
   2. data race
   3. no deadlock/livelock
2. Liveness: 'something good happens'
   1. the program must terminate in a valid state
   2. no starvation
   3. fair access in case of contention

### Mechanism
- Locks: is primitive, hardware based, used to build the others (e.g. *TestAndSet*)
- Semaphores
- Monitors
- Messages

### Deadlock

Waiting program is holding onto a resource that is required for the current program to finish.

**Conditions**:
1. Mutual exclusion
  - $\geq 1$ resource is held and unsharable
2. Hold and wait
  - a process is holding onto a resource and waiting for another process
3. No preemption
  - crit. sections cannot be externally aborted.
4. Circular wait
  - There must be a set of processes $\{P_1, P_2 \dots P_n\}$
  - $P_1$ waiting for $P_2$, $P_k$ waiting for $P_{k+1}$ and 
  - $P_n$ waiting for $P_1$.

### Livelock

Similar to a deadlock, except 
- instead of "hold and wait", the processes release and unblock
- change to another state where the 3 other conditions are met again
  - At the **same time**
- This is avoided by only **one process taking action**
  - With no interleaving allowed!

### Starvation

A process is prevented from making progress due to 
**another process holding the resource required**.

## Types of Parallelism

Task vs Data parallelism

## Executing a concurrent program

1. Compile
2. Link
3. Load
4. Execute

### Types of code formats

1. Machine Code: Binary executable
2. Object Code: Parts of machine code that requires **linker** to join
3. Assembly code: Text format, human readable with 1:1 analog with machine insns. 
   - Platform specific (because map to machine insn)
4. High level language, must be converted to assembly

### C/C++ Compilation

1. Preprocessing (C/C++ into intermediate HLL)
   1. Replacing pre-processing directives
2. Compiler (HLL processed source to assembly) 
3. Assembler (assembly code into .obj)
4. Linker (final output from the .obj files of the assembler)

