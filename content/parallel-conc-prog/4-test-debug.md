---
layout: default
usemathjax: true
title: Testing and Debugging Concurrent Programs
permalink: /pcp/ch4
---

# Concurrency related bugs

1. Unwanted blocking [Deadlock, Livelock, I/O]
2. Race conditions

## Race conditions

Data races: **Undefined behaviour** due to concurrent memory access by at least two threads, at least one of which is a write,
of the same memory location.

Broken invariants:

- Dangling ptrs: thread2 deletes memory that thread1 is accessing
- Random memory corruption: partial updates
- Double free: 2 threads popping the same value off a queue

Lifetime issues:

- thread outliving the accessed data
  - Always check and call `join()`
  - e.g. thread referencing local vars that go out of scope before thread function is done.

## Methods

Eliminate the concurrency

Run on a single core

How many threads can my program run for optimal performance?

Processor architectures:
x86 runs on TSO, so rel-acq may work better than expected lol

Library calls, operators may NOT be thread safe!

# Memory tools

Memory tools keep track of the state of the memory.
Computes **happens-before** to find the data races.

- Dynamic instrumentation 
  - Heavyweight binary instrumentation (x20)
  - but realtime
  - **Memory shadow**

## Memory shadow

For every byte of memory location:

1. A bit (is the byte accessible)
2. V bit (is byte initialized)

Detect and report incorrect memory access.

![Valgrind Shadow Memory](/notes-blog/assets/img/pcp/valgrind-shadow-mem.png)

## Helgrind

Detect 

- misuses of `pthread` API by **intercepting** POSIX functions.
- Potential deadlocks from lock-ordering
- data race
  - build a DAG of happens-before
  - monitor all mem access

# Sanitizers

ASan: Address Sanitizer.

- Necessary prereq for TSan, as memory needs to be checked for data races, or accessed out of lifetime etc.

TSan: Thread Sanitizer. Replaces your library calls with code that

- checks that ops on shared data are done with lock
- record lock sequence 
- give some threads priority

