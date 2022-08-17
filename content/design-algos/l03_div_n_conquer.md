---
title: Word RAM model and Asymptotic Runtime
usemathjax: true
permalink: /algo2/l03
---

$\require{color}$

# Iteration

Iterations = Loops sequentially processing input elements

How do we show correctness in iterative algorithms? **Loop invariant**

- If $P(0)$ holds and $P(i-1)$ is true for subarray $A[i-1]$ before the loop
  - **Initialization**
- and $P(i-1) \Rightarrow P(i)$ for subarray $A[i]$ at the end of the loop
  - **Maintenance**
- then $P(n)$ is true for all $n \ge 0$
  - **Termination**
  - The invariant should imply the algorithm is correct for $A[n]$ i.e. $P(n)$ is true
- Inductive proof!

# Recursion
