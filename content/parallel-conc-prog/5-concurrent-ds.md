---
layout: default
usemathjax: true
title: Design Concurrent Data Structures
permalink: /pcp/ch5
---

# Conc ds

1. Multiple thread concurrently access
   1. Same/distinct ops
2. Self consistent view

Thread safe!!
- no data loss/corruption
- invariants
- no RC

## mutex

lose true concurrent access

Serialization: performing actual access becomes **serialized**

How to do true concurrency?

## making invariants hold

- often broken during DS update (e.g. no. of items containing the item list count)
- broken invariants SHOULD NOT BE VISIBLE from the outside.

Use invariants to reason about program correctness.

## e.g. Node deletion from linked  list

one thread breaks the linked list.

however other threads SHOULD NOT SEE this breakage!

## Thread safety of DS

1. Ensure no thread see invariants broken by a particular thread
2. Avoid race conditions by providing **complete operations**
   1. double free
   2. use after free
   3. data race
3. Behavious in presence of **exception**
4. Minimize deadlock when using DS by using properly scoped locks
   1. Lock at the appropriate granularity
5. constructor and destructor require **exclusive access** to the data structure.
   1. if you have assignment, swap(), copy constructor.
   2. can they be called at the same time?
6. Don't break up ops to prevent interleaving.

```shared_mutex```: can have different  level of mutex access (shared and exclusive)
- can solve readers-writers problem

notify_all
notify_one

make sure that the constructor for an object being used has finished before you start using the object.

# Designing for concurrency:

Try to protect the SMALLEST POSSIBLE REGION of the code.

