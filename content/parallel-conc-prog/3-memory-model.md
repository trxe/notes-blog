---
layout: default
usemathjax: true
title: Atomics and Memory Model in C++
permalink: /pcp/ch3
---

# Memory model and atomics

[Memory model and atomics in C++](/notes-blog/pcp/ch3-1)

# Synchronized program behaviour

Sequential consistency (Lamport's definition):

> “…the results … is the same as if the **operations of all** 
> the processors were executed in some sequential order, and
> the operations of each individual processor appear in this sequence 
> in the order **specified by the program**”

Simultaneous operations (concurrent-with): no happens-before ordering.

Atomicity: Operation guaranteed to execute **without visible intermediate** state.

In a **correctly synchronized program**:
- Memory ops are executed in apparently **sequentially consistent** *interleaved* way
- Write ops have to be
  - Apparently atomic
  - Globally visible to all processors
  - (i.e. Cache coherent)

# What happens to our code

SC is very inefficient to guarantee. The following changes may happen to the execution:

- Compiler optimizations
- Processor out of order executions (operation reordering)
- Cache storage (private shared caches without cache coherency)

```cpp
flag1 = 0; flag2 = 0;

void thread1() {
  while (true) {
    flag1 = 1;
    if (flag2 != 0) {
      // resolve contention
    } else {
      // critical section
      flag1 = 0;
    }
  }
}

void thread2() {
  while (true) {
    flag2 = 1;
    if (flag1 != 0) {
      // resolve contention
    } else {
      // critical section
      flag1 = 0;
    }
  }
}
```

The above execution is subject to:
- cache incoherence (of flag1 and flag2)
- reordering: compiler is allowed to read flag1 before flag2 is set in thread2, and read flag2 before flag1 in thread1. Both lead to semantic errors

## As-if rule

C++ compiler can perform **any changes** so long as these are obeyed in 
any **single-thread**:

- *`volatile` ops are not reordered wrt to other `volatile` ops on same thread*
- Data written to files is exactly **as if** the program was executed as written
  - Observable effect on single-thread is the same.
- Prompting text sent to device must be shown before wait for input

Compilers don't know which memory locations are **mutable shared**
and can asynchronously change due to op in **another thread**

Also program with **undefined behaviour** is free from the as-if rule.

- Serious bug
- To avoid undefined behaviour, understand **modification order**.

### Operation reordering

- W to W
- R to W
- W to R
- Removal of redundant write operations
  - If we run with one thread, invisible

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

## Ordering

**Operation/Transaction**: **atomic** logical operation on related data

- no intermediate state
- thread in a valid state before and after the execution
- correct independent of other transactions

Transactional memory: hardware guarantees for atomic load-stores of 
objects.

### Acquire/Release barrier

One way barriers.

A release store `x.store(rel)` makes **prior accesses** (of `x`) visible
to a thread performing acquire load `x.load(acq)` **pairing** with the store.

1. Acq/Rel **without SC** can be reordered past each other
2. With SC, they cannot.

With locks the compiler **cannot reorder** `mtx.lock()` before `mtx.unlock`
because it would violate the internal acquire/release operations!

#### CPP Objects

Every var is an object.

- Every object occupies at least one memory location
  - Fundamental types take up **exactly one** memory location
  - **Adj bit fields** are part of the same memory location

####  Critical section implementation

1. Locks
2. Ordered Atomics
3. Transactional Memory

#### Modification Order

**Sequence of all writes to an object** from all threads in the program.

Each **object** has a modification order.

In data race, diff threads see distinct sequences of values for a single variable.

**Once a thread see a particular entry**
- Subsequent reads must return **later values**
- Subseuent writes  occurs **after**

Each individual object must have a agreed on modification order

Different objects can have different relative ordering of the modifications.

#### Atomic operations and types

Every atomic operation is an indivisible **transaction**, e.g. Load/store.

Non-atomic operations may be seen as **partially complete**.

```atomic``` weapons for

- low-level synchronization
- reduce to 1-2 CPU insn
- `is_lock_free`: if all ops of a given type done directly with atomic ops
  - **Often lock-free programming is slower than using `std::mutex`**
- `std::atomic` only provides overloads for atomic operations
  - `++x` $\neq$ `x+=1` $\neq$ `x = x + 1` when `x` is atomic.