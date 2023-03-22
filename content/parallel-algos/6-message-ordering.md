---
layout: default
title: Message Ordering
usemathjax: true
permalink: /palgo/ch6
---

# Message Ordering

## Causal ordering

If a send event $s_1$ caused a send event $s_2, then causal order requires $r_1 < r_2$
($r_1, r_2$ on same process)

**How do we know** if $s_1$ causes $s_2$?

**Causal order**: If $s_1 < s_2$, and $r_1, r_2$ on the same process, then $r_1 < r_2$.

![Naive causal ordering protocol](/notes-blog/assets/img/palgo/naive_causal_protocol.png)

1. Piggyback blue message on red (so that even if red reaches first, blue can be obtained in the same msg)
   1. Problem: arbitrarily long
2. Hold onto red until blue is delivered
   1. Problem: need to keep track of all blue messages not received yet.

But both require attaching **all messages** the process has ever sent out and received.
Problem: TOO EXPENSIVE!

## Protocol

Matrix $T$ is piggybacked on every message from $i$ to $j$. 

Let Matrix $M$ be pairwise_max$(M,T)$.

**DELIVER** message if (LOOKING AT MY COLUMN...)

$$
\begin{cases}
T[k,j] \leq M[k,j] \quad \text{for all } k \neq i \quad \text{(My history includes the history of the sender)} \\
T[i,j] = M[i,j] + 1 \quad \text{(This means that this message is definitely the next consecutive one from $i$)}
\end{cases}
$$