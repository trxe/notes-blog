---
layout: qna
title: Introduction to Algorithms
usemathjax: true
permalink: /algo2/l01
---

# Introduction to Algorithms

## Definitions

**Problem**: A set consisting of pairs of $(x, S)$, where
- $x$: valid input
- $S$: set of valid solutions

e.g. $\textsf{Mul} = \{((x,y), \{xy\}) \mid x,y \in \mathbb{R}\}$ 
means the problem $\textsf{Mul}$ is the set of all inputs $x,y$ with the set of 
their product $xy$.

<div class="question"> How would you define the problem $\textsf{Factor}$? </div>

**Algorithm**: Procedure that outputs a valid solution to a problem given some input.

In this module we mainly care about algorithms'
- Correctness: Even with worst case-input, algorithm does not return a wrong solution.
- Running time: Number of steps executed by algorithm according to a **computational model**.

**Computational Model**: Defines
- Which fixed **operations** are allowed?
- How many **time units** each fixed operation takes?

## Adversary Arguments

These are used to prove lower bounds of runtime, i.e.
*There exists no algorithm* $M$ *that solves* $\textsf{Problem}$ *in less than some runtime.*

**Adversary**. Input model that returns the worst case answer for any operation the algorithm may perform.

### Outline of proving algorithmic lower bounds $L$

<div class="important">
*Theorem.* Show that any algorithm solving (insert problem) has runtime $ge$ (insert runtime) by the (insert comparison model).

1. Come up with a representation model the adversary can visualize the state of the problem with. (Graph, tree, etc.) The Algorithm is constructing this representation model with each query.
2. For each operation, the adversary must set the input such that the operation is as **least informative** as possible.
3. Suppose some algorithm with runtime $<$ given amount **outputs the correct answer for some input**. Then in **another modified input**, the same algorithm will return the same answer but would be wrong.
</div>

### For the Zero-One problem

Let $\textsf{Zero-One}$ bef the problem: given an array $A$ of size $n$ of 0s and 1s, determine if there are more 0s than 1s in the array.

<div class="question">
*Theorem.* Any algorithm solving $\textsf{Zero-One}$ has runtime $\ge n$
by the query model (each query = one time unit).
</div>

*Proof.* Suppose there exists an algorithm $\mathcal{M}$ which solves $\textsf{Zero-One}$ in time $< n$ with the query model.
1. [Can omit]
2. Let array $A$ be an array of size $n$ where $n$ is odd. 
   - Consider an adversary which alternates between yes and no to the query "Is $A[i] == 0$?" Let $q \le n-1$ be the number of queries that $\mathcal{M}$ makes
   - Note that the difference between the number of 0s and the number of 1s is 1 if the $q$ is odd, and 0 if $q$ is even.
   - Suppose $\mathcal{M}$ determines correctly that there are more 0s than 1s in $A$.
   - Let $A'$ be the array such that the remaining unqueried elements are 1.
3. $\mathcal{M}$ cannot distinguish between $A, A'$.
   - If $q$ is odd, there are at least 2 elements not queried and since $2 > 1$, there will be more 1s than 0s in the array $A$. A similar argument can be made if $q$ is even.
   - As such $\mathcal{M}$ will err on at least one of the inputs.

### For the Maximum problem

Let $\textsf{Max}$ be the problem: given an array $A$ of numbers,
return the number with maximum value in $A$.

<div class="question">
*Theorem.* Any algorithm solving $\textsf{Max}$ has runtime $\ge n - 1$
by the comparison model (each comparison between 2 elements = one time unit).
</div>

*Proof.* Suppose there exists an algorithm $\mathcal{M}$ which solves $\textsf{Max}$ in time $< n - 1$ with the comparison model.
1. Construct a graph of $n$ nodes where each node corresponds to an index.
   - Let each established comparison where $i < j$ be represented as a directed edge $i \rightarrow j$.
2. [Can omit] For $n-1$ comparisons, the adversary can respond with either Yes or No to the comparison "Is $i < j$?"
3. After any $m < n-1$ comparisons, note that the graph is still disconnected. 
   - Hence we can partition the graph $(C_1, C_2)$ such that there are no edges between $C_1$ and $C_2$.
   - Suppose $\mathcal{M}$ returns element $A_k$ on input $A$.
   - Let input $A'$ be such that for $i \in \{1 \dots n\}$,

$$
A'_i = \begin{cases}
    A_i & \text{if $i \in C_1$} \\
    A_i + m & \text{if $i \in C_2$} \\
\end{cases}
$$
   - Suppose $m$ is so large such that some $A'_i = A_i + m > A_k$.
   - $\mathcal{M}$ does not make any comparison between $C_1$ and $C_2$ and thus cannot distinguish between $A'$ and $A$ and will return the same answer for both.
   - Thus it will err on at least one of the two inputs.

### For the Sorting problem

Let $\textsf{Sort}$ be the problem: given an array $A$ of numbers,
return the same array but in sorted order $B$ such that $B_1 < B_2 < \dots < B_n$.

<div class="question">
*Theorem.* Any algorithm solving $\textsf{Sort}$ has runtime $\ge \log(n!)$
by the comparison model.
</div>

*Proof.* Suppose there exists an algorithm $\mathcal{M}$ which solves $\textsf{Max}$ in time $< n - 1$ with the comparison model.
1. Let the set $U$ be the set of all the possible permutated inputs of the indices of $A$.
   - Note that the size of $U$ is $n!$.
2. Let $U_i$ be the number of valid inputs consistent with the previous $i$ comparisons made by the algorithm.
   - Let $U_{i, A_j < A_k}$ be the subset of $U_i$ where $A_j < A_k$.
   - Let $U_{i, A_j \ge A_k}$ be the subset of $U_i$ where $A_j \ge A_k$.
   - The adversary should respond to "Is $A_j < A_k$?" such that

$$
U_{i+1} = \begin{cases}
    U_{i, A_j < A_k} & \text{if $\|U_{i, A_j < A_k}\| \ge \|U\|/2 $ } & \text{(YES)}\\
    U_{i, A_j \ge A_k} & \text{if $\|U_{i, A_j \ge A_k}\| \ge \|U\|/2 $ } & \text{(NO)}
\end{cases}
$$
   - This ensures that on each round $U_{i+1}$ is no less than half the size of $U_i$.
3. After any $m < \log(n!)$ rounds of comparisons, the size $U_{m}$ is more than 1.
   - For any two inputs $A, A' \in U_m$, the algorithm $\mathcal{M}$ will not be able to distinguish between them and will return the same answer.
   - This answer will be wrong for at least one of the two inputs $A, A'$.

### For the 