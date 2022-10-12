---
layout: default
usemathjax: true
permalink: /parallel/quizzes
---

# Quiz 2

### Question 2
A program X has a constant sequential fraction value regardless of problem size. 
Amdahl's and Gustafson's Laws predict different maximum speedups.

- False
- CONSTANT SEQUENTIAL FRACTION value
- Gustafson's assumption: sequential fraction becomes smaller with larger problem size.

### Question 4

Given a program with 95% arithmeticinstructions and 5% memory instructions, 
Amdahl's Law instructs programmer to optimize arithmetic calculation before looking into memory accessses

- FALSE
- 95% of the computation is arithmetic BUT Sequential fraction is about the **time taken** by a part of a program
- Amdahl's law tells us to optimize the bottle neck but this question doesn't tell us what the bottle neck is.

### Question 6

Data parallelism: Within the main simulation there are no parallel heterogenous tasks.
Ad d pragma omp above for loop in line 18

Parallel patterns:
- Task pool: not good, Task pool does not have coordination and usually handles heterogenous tasks
- Master worker: homogenous work, and have a coordinator.


# Quiz 3

### Question 1 + 2

- False sharing
- Different data members of a struct are laid out contiguously, and they are writing to the same entity
- Overhead from updating the same cache line
- When thread A writes to position, thread B's cache will be invalidated and have to be updated before the state can be updated.

Assumptions:
- Write allocate cache being used
  - A cache with a write-through policy (and write-allocate) reads an entire block (cacheline) from memory on a cache miss and writes only the updated item to memory for a store. Evictions do not need to write to memory.

### Question 3

Relaxed consistency models address the concern that software complexity is increased by the need for unnecessary sychronization.

- False
- In fact, dealing with relaxed consistency requires more attention to synchronization
- It's a way to improve performance rather than complexity
- Software complexity: How difficult it is to write the software to run.

### Question 4 + 6

Cache coherence is the property that multiple caches contain the same set of items.

- False
- Caches don't have the contain the same set of itmes