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

Liveness "Something Good happens". 
Safety "Something Bad Never happens".

**Blocking**: Not being able to progress when process/thread is scheduled by the OS.

- Lock-based: **Only** locks to synchronize.
- Lock-free: **Any one thread can progress** regardless of other threads being blocked. 
- Wait-free: A lock-free program that all threads can simultaneously progress.

# Identifying races

Using the example of a lock-free queue implementation.

## Producer-Producer conflicts

In a push (producer) operation, there are the following main steps

1. Create new dummy
2. Lock the queue [REMOVE THIS]
   1. Set the back's data
   2. Set the back's next as our dummy
   3. Set back as our dummy
3. Unlock the queue [REMOVE THIS]

Problems are:

- Overwriting work
  - Two nodes get the same "back" as their working node
- Leaking nodes
  - Two nodes try storing their dummy to the same memory location
  - So one of the dummies is leaked (never cleaned)

**Solution**: Use `old_x = x.exchange(new_x)` which is an atomic swap between `new_x` and `old_x` stored in memory loc of `x`

## the  `then` problem: "Do X then do Y"

The problem is that the X and Y can be interleaved with something else that overwrites the state after finishing X.

How to check if the queue is empty? check if the next node is empty.

## Consumer-Consumer conflicts

1. Lock the queue [REMOVE THIS]
  1. Get the head
  2. If head is not dummy, set the head as the next
2. Unlock the old head [REMOVE THIS]
3. Get head's data
4. Free head
5. Return head's data

Problems are:

- If same non-dummy head is obtained by two or more processes
  - Multiple access to data
  - Double free

## Producer-Consumer/Consumer-Producer conflicts

- If the producer is halted in the middle of updating a node, and the consumer sees and pops the incomplete node.
  - After the `exchange`, producer still may not have be set the previous back to the new dummy
- If the consumer is in the middle of removing a node and producer sets the next of the node to the new dummy

## the `if-then` probem: "If X then do Y"

**Solution**: Use `bool succ = x.compare_exchange_weak(expected_x, new_x, mo_succ, mo_fail)` which is an atomic swap 
between `x` and `new_x` iff `x` is equivalent to `expected_x` at the point of comparison. returns whether the swap was successful.

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