---
layout: default
usemathjax: true
title: Atomics and Memory Model in C++
permalink: /pcp/ch3
---

# Synchronized program behaviour

- Mem ops
- Sequentially consistent interleaved exeution
  - Performance of code decrease
- Write ops have to be
  - Atomic
  - Globally visible to all processors

## Operation reordering

- W to W
- R to W
- W to R
- Removal of redundant write operations
  - If we run with one thread, invisible

## As-if rule

C++ compiler can perform **any changes** so long as these are obeyed:

- Data written to files is exactly as if the program was executed as written
- Prompting text sent to device must be shown before wait forinput

But program with **undefined behaviour** is free from the as-if rule.

- Serious bug
- To avoid undefined behaviour, understand **modification order**.

## MM in C++11

Multithreading aware memory model: Using mutex/futex/barrier don't require understanding of the MM.

Essential to make all MT facilities work.

### Structure

Each var = 1 obj.
Obj occupy **at least 1** memory location
Fundemantl types: exactly 1 memory location, whatever their size, even if they are part of an array.
Adjacent bit field $\in$ same memory location

### Concurrency

Mult threads access the same memory location.

Data race:

- **Lack of Ordering** between accesses
- **At least one** is not atomic
- **At least one** is a write

###  Critical section implementation

1. Locks
2. Ordered Atomics
3. Transactional Memory

#### Modification Order

Sequence of all writes to an object from all threads in the program.

Each **object** has a modification order.

In data race, diff threads see distinct sequences of values for a single variable.

**Once a thread see a particular entry**
- Subsequent reads must return **later values**
- Subseuent writes  occurs **after**

Each individual object must have a agreed on modification order

Different objects can have different relative ordering of the modifications.

#### Atomic operations and types

Indivisible operation, e.g. Load/store

Non-atomic operations may be seen as **partially complete**.

```atomic``` weapons for

- low-level synchronization
- reduce to 1-2 CPU insn
- `is_lock_free`

## SC `memory_order_seq_cst`

If the target architecture is a weakly ordered machine, SC incurs penalty as
extra insns have to beinserted.

## Relaxed Ordering

Load: give a value (load): she will give

# Tutorial 

Operational model (program behaviour) vs axiomatic model (expected behaviour) which needs the proofs.

- Most of the time you don't do concurrency
- But you need to handle concurrency with libraries.
  - Stateless machine, abstracting away concurrency.
- Otherwise if you really need the performance, first **primitives** (locks, semaphores, fences, barriers)
- ...
- Exhausted everything? Fine use atomics.

## The fundamental question of concurrency (Lamport)

Exists on every level

When high level ops depend on low level ops, we assume that low-level systems are atomic

BUT THEY AREN'T! Turtles all the way down...

To reason correct performance, we do need atomic ops. Abstract away all the non-atomicity.

Atomic handling starts to unravel these promises/abstractions.

C++ abstracts over

- Compiler optimizations
- CPU memory management, op reordering, dirty bits (write back memory)

1. Don't remove read writes? `std::atomic` (or `volatile`)
2. No reordering allowed? `std::memory_order_seq_cst`
3. Anyone who sees X should see my past work `std::memory_order_acq_rel`
4. Donjust care about sharing X, but not my past work `std::memory_order_relaxed`
5. I want threads to agree on the same order `seq_cst`. Specific single total order requires SC.
6. Increment counter? `relaxed` don't care about
7. Read/update X, then update Y in that order and mustbe visible to all other `acq_rel`

- Without atomic, var cache can be unrefreshed.
- The entire test may even be optimized out if the variable is determined to never change.

SC guarantees that there is a fixed total ordering of all operations across all variables that all the threads see.

Acquire Release not longer guarantees this total ordering across different vars seen by each thread may be different.

Create cycles to check if something is impossible.

Relaxed: Operations are not reordered by the compiler, but the processor relaxes constraints on that level.