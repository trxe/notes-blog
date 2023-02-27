---
layout: default
title: Mutual Exclusion Problem
usemathjax: true
permalink: /palgo/ch1
---

# How and why to implement a critical section?

Why: to prevent race conditions.

1. Two or more processes accessing the same memory loc AND
2. At least one of the processes is writing to it.

Properties needed:

1. Mutual Exclusion
   - At most one process in CS
   - To prove using contradiction: show impossible for $\geq 2$ processes in CS
2. Progress
   - If both processes are not in CS, then one of them can go into CS when it needs to.
3. No starvation
   - If one process is in CS and **assuming it eventually leaves**, the other processes can enter CS

## Peterson's algorithm

## Lamport's bakery algorithm
