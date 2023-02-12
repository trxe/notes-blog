---
layout: default
usemathjax: true
title: Atomics and Memory Model in C++
permalink: /pcp/ch3
---

# Memory model and atomics

[Memory model and atomics in C++](/notes-blog/pcp/ch3-1)

# Synchronized program behaviour

- Memory ops are executed in apparently **sequentially consistent** *interleaved* way
- Write ops have to be
  - Apparently atomic
  - Globally visible to all processors

**Atomicity**: Operation guaranteed to execute **without visible intermediate** state.

SC is very inefficient to guarantee. As such **operation reordering**  will most likely happen.

## Operation reordering

- W to W
- R to W
- W to R
- Removal of redundant write operations
  - If we run with one thread, invisible

## As-if rule

C++ compiler can perform **any changes** so long as these are obeyed:

- Data written to files is exactly **as if** the program was executed as written
- Prompting text sent to device must be shown before wait for input

But program with **undefined behaviour** is free from the as-if rule.

- Serious bug
- To avoid undefined behaviour, understand **modification order**.

## MM in C++11

Multithreading aware memory model: Using mutex/futex/barrier don't require understanding of the MM.

Essential to make all MT facilities work.

### Structure

Each var = 1 obj.
Obj occupy **at least 1** memory location
Fundemantal types: exactly 1 memory location, whatever their size, even if they are part of an array.
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
- `is_lock_free`: if all ops of a given type done directly with atomic ops
  - **Often lock-free programming is slower than using `std::mutex`**
- `std::atomic` only provides overloads for atomic operations
  - `++x` $\neq$ `x+=1` $\neq$ `x = x + 1` when `x` is atomic.