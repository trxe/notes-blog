---
layout: default
usemathjax: true
permalink: /parallel/ch1
---

# Introduction

## Serial Computing

1. Divide problem into discrete instructions
2. Execute instructions in sequence, one at a time

**Memory wall**: Disparity between processor (1ns) and memory speed ($\approx$ 100 to 1000ns)

PU is based on von Neumann model.

![von Neumann Model](/notes-blog/assets/img/parallel/von_neumann.png)

## Performance boost methods

1. Higher clock freq (Work **hard**)
2. Pipelining, superscalar (Work **smart**)
3. Multicore, cluster (Get **help**)

## Parallel Computing

|              | Concurrency                               | Parallelism              |
|--------------|-------------------------------------------|--------------------------|
| $>=2$ tasks  | Start/Run/Complete in overlapping periods | Execute simultaneously   |
| Progress via | Interleaving their execution periods      | Running at the same time |

**Motivation**:
- **Overcome serial computing limits**
- **Speedup**/Save time
- Utilizing non-local resources
- Cost-saving
- Overcome memory constraints

# Parallel Computing Approach

![The Parallel Method](/notes-blog/assets/img/parallel/parallel-method.png)

## Parallel solution: Sum of $n$ numbers

- $p$ cores
- $n$ numbers
- Each core **partial sum** of $n/p$ numbers
  - private vars
  - $O(n/p)$
- Master core adds up all values
  - into global sum
  - $O(p)$
- Optimization:
  - neighboring cores add up values ($\log(p)$ iterations, each taking $O(1)$ time)

## Misc

**Supercomputer**:
- computer performing at the highest operational rate for computers.
- many PUs with many nodes and memory spaces
- uses **parallel processing** to communicate and solve problems