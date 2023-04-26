---
layout: default
usemathjax: true
title: Atomics in C++
permalink: /pcp/ch3-1
---

# Memory Model at language level with atomics

Since 2011, by default C++ guarantees SC-DRF (**default** sequential consistency
for data-race free programs).

However you can use atomics to 

```cpp
std::atomic<bool> x, y;
// ... in one thread
x.store(true, std::memory_order_seq_cst);
while (x.load(std::memory_order_seq_cst)) {};
// ... in another thread
while (!x.load(std::memory_order_seq_cst)) {};
x.store(false, std::memory_order_seq_cst);
```

As can be seen above atomics can take in an **optional memory-ordering argument**.

```cpp
enum std::memory_order {
  memory_order_seq_cst, // default
  memory_order_relaxed,
  // LOAD ACQUIRE - STORE RELEASE
  memory_order_release, // store ops only
  memory_order_acquire, // load ops only
  // RMW ops can do all of the above plus
  memory_order_acq_rel,
}
```

# Operation/Transaction ordering

- sequenced-before (sb)
- synchronizes-with (sw)
- read-from (rf)
- modification order (mo)

Useful tool to generate memory order graph: [http://svr-pes20-cppmem.cl.cam.ac.uk/cppmem/]

![Example Memory Order](/notes-blog/assets/img/pcp/example-memory-order.png)

### Sequenced before

(Program order) If eval A is sequenced before B, then `resp(A) < inv(B)`.

### Synchronizes-with relationship

(Object order) Given atomic store on `x` by thread 1,
and atomic load on `x` by thread 2 reads the value written by thread 1 or by any thread **after**,
then the `store` synchronizes-with the `load`.

Between atomic **load** and **store** operations,

- Atomic write `W` tagged with either SC or Acq-Rel or Rel
- Atomic read `R` tagged with either SC or Acq-Rel or Acq
- `R` reads value stored by
  - write `W`
  - an atomic write **after** `W` (i.e. W can be the original value)
    - You can have a synchronizes-with with the original value and the atomic load.
  - RMW `rmw` where `rmw`'s read detects value stored by  `W`

### Inter-thread happens-before

1. A synchronizes-with B (object order)
2. A sync-with X, X seq-before B
3. A seq-before X, X inter-thread happens-before B
4. A inter-thread happens-before X, X inter-thread happens-before B

![Inter-thread happens-before](/notes-blog/assets/img/pcp/interthread-happens-before.png)

### Happens-before

1. A is sequenced-before B
2. A interthread happens-before B

### Visible side effect

The side effect A on a scalar M is **visible** to B on M if

1. A happen-before B AND
2. No other side effect on M where A hb X and X hb B

### Modification order

Object order of writes

- A **happens-before** B
  - In process sequenced-before
  - Or interthread happens-before
- No other side effect X to M s.t. A happens before X and X happens before B (cyclic)

# Memory Orderings

## Sequential Consistency `memory_order_seq_cst`

Sequential Consistency (SC) guarantees that **all threads must see the same process order**.

If the target architecture is a weakly ordered machine, SC incurs penalty as
extra insns have to be inserted.

**Sequential orddering**:
- ALL Threads must see the same order of operations
- No reordering
- Every store synchronizes-with every load

Non-SC orderings only agree on the **modification order** of each object
but not the global order.

## Relaxed Ordering

Relaxed ordering only guarantees that **all threads will see the same object order**.

- Within the same thread, SC is obeyed (object order and process order maintained)
- Across different threads, no such guarantees.

### Example of relaxed ordering

For a variable `v`, 

- if Thread A stored `4` into `v`
- and Thread B stored `5` into `v`
- when Thread C loads `v` it can get either `4` or `5`.
  - if Thread C loads `v == 5`
  - and Thread D loads `v` afterwards, 
  - Thread D **cannot see** `v == 4`  

## Acquire release ordering

If a thread A does an atomic store tagged `memory_order_release`
and a thread B does an atomic load tagged `memory_order_acquire`

**all ops** before the thread A's atomic store op become  visible to thread B.

This includes **non-atomic** operations!!

# Tutorial 

1. Don't remove read writes? `std::atomic` (or `volatile`)
2. No reordering allowed? `std::memory_order_seq_cst`
3. Anyone who sees X should see my past work `std::memory_order_acq_rel`
4. Just care about sharing X, but not my past work `std::memory_order_relaxed`
5. I want threads to agree on the same order `seq_cst`. Specific single total order requires SC.
6. Increment counter? `relaxed` don't care about
7. Read/update X, then update Y in that order and mustbe visible to all other `acq_rel`

- Without atomic, var cache can be unrefreshed.
- The entire test may even be optimized out if the variable is determined to never change.

SC guarantees that there is a fixed total ordering of all operations across all variables that all the threads see.

Acquire Release not longer guarantees this total ordering across different vars seen by each thread may be different.

Attempt to create DAG; if cycles are formed then deadlock exists.

Relaxed: Operations are not reordered by the compiler, but the processor relaxes constraints on that level.