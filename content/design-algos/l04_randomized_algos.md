---
title: Word RAM model and Asymptotic Runtime
usemathjax: true
permalink: /algo2/l04
---

$\require{color}$

# Average-case running time of Quicksort

Pathological case: $\Theta(n^2)$ running time on arrays that are sorted.

What would the runtime be for a "generic" input?

### Definitions

- $U$: Set of all inputs
- $D_n$: Probability distribution of inputs of size $n$.
- $A(n)$: average case run time

$$
A(n)  = \mathbb{E}_{x ~ D_n} \text{[Runtime of algorithm on $x$]}
$$