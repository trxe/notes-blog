---
layout: default
title: Clocks
permalink: /palgo/ch4
---

# Clocks for distributed programs

1. No shared clock
2. No shared memory
3. No accurate failure detection

Distributed programs communicated via **message-passing**.

## Formalisms

Distributed program: $\{P_1, P_2, \dots, P_N\}$

Channel: on which messages are sent; edge between $(P_i, P_j)$.

Process: $\{S_1, S_2, \dots, S_K\}$

Run or computation: $\{E, \rightarrow\}$ where $\rightarrow$ is the 
**happens-before** relationship.

**Happens-before**:

1. Partial order amongst events
2. Process order with send-receive order, apply transitivity.

## Logical clock

Event s happens-before event t $\Rightarrow$ Event s's logical clock value $<$ event t's logical clock value.

- $C_{i,0} = 1$
- $C_{i,m} = \max(C_{i,m-1}, V) + 1$
  - This algorithm can vary: +5 instead of +1, +Rng instead of +1, or $C_{i, m-1} + V$.

The reverse "s's logical clock value $<$ t's logical clock value $\nRightarrow$ event s happens-before event t"

## Vector clock

