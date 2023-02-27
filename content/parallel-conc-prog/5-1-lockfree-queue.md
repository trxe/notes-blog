---
layout: default
usemathjax: true
title: Lock Free MPMC Queue
permalink: /pcp/ch5-1
---

# Lock Free Multi-Producer Multi-Concurrent Queue

## Key takeaways

1. 'lock-free'
2. 'then' 'if-then' problem
3. compare-and-swap
4. ABA problem

## How to do concurrent programs

1. Performance reqs
   1. Time Complexity
   2. Space Complexity
2. Correct Sequential Implementation
   1. Single thread queue
   2. Invariants

Liveness "Something Good happens"
Safety "Something Bad Never happens"

Lock-based: **Only** locks to synchronize.

Lock-free: **Can** use locks and atomics where at least 1 thread is progressing at any time.

Wait-free: A lock-free program that all threads are progressing

## Maintain invariants (before and after the op)

Lock-based: simply locked

Lock-free: alternative with atomics, what are the pitfalls?

- Producer-Producer conflicts: Interleaving
  - Overwriting
  - Leaking nodes

## the  `then` problem: "Do X then do Y"

The problem is that the X and Y can be interleaved with something else that overwrites the state after finishing X.

**Solution**: Try to use `.exchange` which is an atomic swap between your current val stored.

How to check if the queue is empty? check if the next node is empty.

- Producer-Consumer conflicts: Interleaving
  - If the producer is halted in the middle of updating a node, and the consumer sees and pops the incomplete node.

## the `if-then` probem: "If X then do Y"

- Consumer-Consumer conflicts: Interleaving
  - old_front, new_front, if  (new_front) else ___
  - Need CAS (Compare and swap)
    - Load a value, 
    - do some unsynced work, 
    - we check if the value is still the same and exchange the value.
    - else try to load the value again.

```cpp
// if (front == old_front) { front = new_front; return true;}
// else                    { old_front = front; return false;}
bool ok = front.compare_exchange_weak(old_front, new_front);
```

is there still a problem? 
**our invariant can have broken and restored between the compare_exchange attempts...**

## ABA problem

condition: raceon a if-then condition

state A: old.
state B: massive  changes to the  old state
state A: by our check condition, old == new even though old.otherattributes != new.otherattributes

solutions:?
- know that old != new
- change thenew: unique address on every new id
  - epochs
  - **generation counter**
    - i'm in the xth generation now
  - UUID? GUID?? ALWAYS INCREMENT THE SAME NUMBER
- change the old: 
  - don't reuse theold address.
  - referencecounting
  - hazard pointers
  - free all on destructor - garbage collection

## Generation counter

on init, set a generation number (I'm in the xth generation.)

when we init a new object, we always increment the generation.

overflowing the size of the generation number: we assume we don't overflow... use a big integer???
- how do you even atomically compare a big integer?? you can't.

```cpp
struct alignas(16) GenNodePtr {
    Node* node;
    uintptr_t gen; // the generation counter
}
```

## Use after free

soln: 

- No free: (leak the memory)
- No freewhile queue in use (free list/memory arena, just not in use)
- No free while threadis  using (ref counting, hazard pointers)

They must be lock free too :" ). OTherwise your thing isn't lock free.