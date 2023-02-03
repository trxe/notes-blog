---
layout: default
usemathjax: true
permalink: /pcp/ch1
---

# Introduction

Concurrency vs Parallelism (again):

**Concurrency**

- interleaving
- illusion of multiple tasks executing at the same time 
- but actually may not be happening at the same instance 
  - illusion of concurrency vs true concurrency
  - possible through hardware improvements

**Parallel**

- tasks actually simultaneously executes
- multiple hardware threads per core

Multiple processes vs Multiple Threads:

**Processes**
- More safety
- context switch is rsc demanding
- but no race conditions
- creation (via forking) is costly

**Threads**
- more performance
  - optimization strategy
  - take advantage of hardware
- independent control flows
- lightweight, less rsc demanding
- Share context for all except stack, race conditions
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

1. Two threads access shared resource at the same time without synchronization
2. At least one modifies the shared resource

Solution: synchronization (buffer/queues, lists, hash table)

Mutex as a synchronization method:
- critical section is a code sequence using mutual exclusion
  - Requirements: **safety** and **liveness**
- Locks: is primitive, hardware based, used to build the others (e.g. *TestAndSet*)
- Semaphores
- Monitors
- Messages

### Deadlock

Waiting program is holding onto a resource that is required for the current program to finish

### Livelock

### Starvation

## Types of Parallelism

Task vs Data parallelism:

**Task parallellism**:


## Executing a concurrent program

1. Compile
2. Link
3. Load
4. Execute

in C/C++:
1. Preprocessing (into HLL)
2. Compiler (source to assembly)
3. Assembler (assembly code into machine code)
4. Linker (final output from the .obj files ofthe Assmebler)