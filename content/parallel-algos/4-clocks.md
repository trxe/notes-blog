---
layout: default
title: Clocks
permalink: /palgo/ch4
---

# Clocks for distributed programs

## Motivation

For **order imposition**. Relying on wall clock time subjects one to inaccuracies 
of the individual physical clocks and synchronization error.

Thus distributed programs have to work with

1. No shared clock
2. No shared memory
3. No accurate failure detection

Distributed programs communicated via **message-passing**.

## Software clocks

1. Less overhead
2. Sufficient accuracy to correctly order system's events
   1. Software clocks are for inferring ordering
   2. Physical clock still needed for actual time

## Formalisms

Distributed program: $\{P_1, P_2, \dots, P_N\}$

Channel: on which messages are sent; edge between $(P_i, P_j)$.

Process: $\{S_1, S_2, \dots, S_K\}$

Run or computation: $\{E, \rightarrow\}$ where $\rightarrow$ is the 
**happens-before** relationship.

What does $a \rightarrow b$ mean (event $a$ happens-before event $b$): 

- If $a, b$ on same process, then $time(a) < time(b)$
  - **Process order**
- If $a, b$ on different process, then 
  - **Send-receive order**
  - either $a$ is the sending of msg and $b$ is the receiving of that msg, 
  - or **transitivity**
    - there exists a send event $s$ s.t. $a < s$
    - and a receive event $r$ for that msg s.t. $r < b$

If two events $a, b$ happen in isolated processes  such that $a \nrightarrow b$ and $b \nrightarrow a$, then $a$ *concurrent-with** $b$.

## Logical clock

Event s happens-before event t $\Rightarrow$ Event s's logical clock value $<$ event t's logical clock value.

$$s \rightarrow t \Rightarrow C(s) < C(t)$$

Example implementation:
- $C_{i}(0) = 1$
- Local computation/send: $C_{i}(m) = C_{i}(m-1) + 1$
- Receive: $C_{i}(m) = \max(C_{i}(m-1), V) + 1$
  - This algorithm can vary: +5 instead of +1, +Rng instead of +1, or $C_{i}(m-1) + V$.

**Proof** that the above ensures $s \rightarrow t \Rightarrow C(s) < C(t)$:
1. if $s, t$ are on same process and $s \rightarrow t$, then $C(t) \geq C(t-1) + 1 > C(s)$ Hence $C(s) < C(t)$.
2. if $s, t$ have send-receive rs on different process, then 
  - $C_a(t) = \max\{C_b(s), C_a(t-1)\} + 1 > C_b(s)$
3. else by transitivity of the above two cases the above holds

The reverse "s's logical clock value $<$ t's logical clock value $\nRightarrow$ event s happens-before event t"

## Vector clock

Guarantee should hold in **both directions** instead

$$s \rightarrow t \Leftrightarrow C(s) < C(t)$$

- Suppose we have $n$ processes.
- Each process $i$ has a local vector $C = \{c_1, c_2, \dots, c_n\}$
- Note that $C(s) < C(t)$ is a **partial order**: 
  - only valid if all values of $C(s)$ are less than all corresponding values of $C(t)$

Example implementation:
- $C_{i}(0) = \{\dots, c_i = 1, \dots\}$
- Local computation/send: $C_{i}(m) = C_{i}(m-1) + \{\dots, c_i = 1, \dots\}$
- Receive: $C_{i}(m) = \max(C_{i}(m-1), V) + \{\dots, c_i = 1, \dots\}$

**Proof**: 

1. Forward direction (trivial, follow logical clock example)
2. Reverse direction $C(s) < C(t) \Rightarrow s \rightarrow t$  
   1. If $s,t$ are on same process, trivial by definition.
   2. If $s,t$ not on the same process, then $\forall$ process $i$ we have $C(s)[i] \leq C(t)[i]$ and for some process $j$, $C(s)[j] < C(t)[j]$.

Know about the other processes.

## Matrix clock

- Suppose we have $n$ processes.
- Each process $i$ has a local **matrix** $M = \left[\begin{matrix}C_1 \\ C_2 \\ \vdots \\ C_n\end{matrix}\right]$ where each $C_i$ is process $i$'s **principle vector**
- Note that $C(s) < C(t)$ is a **partial order**: 
  - only valid if all values of $C(s)$ are less than all corresponding values of $C(t)$

Know about what the other processes know about the other processes.

$n$-dimensional clock: (Knowing about what the other processes)$^{n-1}$ know about the other processes.

$\infty$-dimensional clock: Common knowledge.