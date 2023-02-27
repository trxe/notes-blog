---
layout: default
title: Global Snapshot
usemathjax: true
permalink: /palgo/ch5
---

# Global snapshot

## Motivation

Snapshot of local states on $n$ processes **at the exact same time** for

- Debugging
- Backup/Checkpoint in case of crash
- Calculating a global predicate (e.g. value of a global state such as sum)

### Why nontrivial

We don't have a magical camera that can record the state 
at the exact same time across $n$ processes, given that they sync by message passing.

- Flow of messages means data may be lost
- Double counting of values due to recording of a state before send and after receive
- We don't have completely accurate physical time (otherwise trivial!)
  - How much inaccuracy in time can we tolerate?

### What to do

We have to weaken the goal: Snapshot of local states on $n$ processes that
**could have happened** some**time in the past**.

## Formalism

**Global snapshot**: A set of events $S$ such that 
- if $e_2 \in S$ and $e_1 < e_2$ (happens-before in **process** order)
- then $e_1 \in S$.

**Consistent Global snapshot**: A global snapshot (i.e. incld process order) such that
- if $e_2 \in S$ and $e_1 \rightarrow e_2$ (happens-before in **send-receive** order)
- then $e_1 \in S$.

Consistent global snapshot encapsulates the idea of "could have happened in the past":
a possible current state of the program. 
(outcome of a send cannot happen before the send.)

![Red is not a consistent global snapshot.](/notes-blog/assets/img/palgo/cgs_red.png)

### Existence proof

Consider a totally ordered sequence of events $e_1, e_2, \dots$.

**Theorem**: For any positive integer $m$, $\exists$ consistent global snapshot 
$S$ such that 

$$
\begin{cases}
e_i\in S \quad \text{ for } i \leq m \\
e_i\notin S \quad \text{ for } i > m
\end{cases}
$$

**Proof intuition**:

Let $e_m$ be in $S$ where $S$ is a consistent global snapshot. 

Then for all $e_i \rightarrow e_m$ we have $e_i \in S$. 
Since $e_i \rightarrow e_m$, we have $i < m$.

Let $e_j \notin S$ for all $j > m$. As we do not have any $e_j \rightarrow e_m$ since
$e_j > e_m$, we can do this.

For all $k < m$ such that $e_k < e_m$, we can include them in the set $S$ and it will 
still be a consistent global snapshot, as for any $e_l \rightarrow e_k$, 
we have $l < k < m$ so $e_l \in S$.

# Protocol

## Communication model

Assumptions:

1. No message is lost
2. Communication channel is unidirectional
3. FIFO delivery
   1. Can be ensured by assign message number per channel
   2. Receiver **only delivers messages in order** (see TCP)


![If Process 1 snapshot **after** send](/notes-blog/assets/img/palgo/cgs_protocol-1.png)

![If Process 1 snapshot **before** send](/notes-blog/assets/img/palgo/cgs_protocol-2.png)

## Chandy-Lamport Protocol

![Message marker](/notes-blog/assets/img/palgo/cgs_protocol-3.png)

**Right before** process 1 sends message M to process 2, 
a special message is sent from process 1 to process 2 when process 1 wants to tell process 2 to take a snapshot.

Upon receiving the special message, process 2 needs to take a snapshot before it can receive $M$.

hence any process will receive $n-1$ special messages.

There is an initiator global snapshot, which sends out special messages to **all** 
of the other processes in the system.

Every process is either in white state (no local snapshot taken) or red state (taken).

Upon receiving a special message for the first time (turn from white to red) it will 
send out $n-1$ special messages to all the rest of the processes too.

Now we are able to capture the **processes' state**, 
but **not yet the messages on the fly**!

The only state in which this exists is when process 1 snapshot after send 
and process 2 snapshot before receive.

![Message marker](/notes-blog/assets/img/palgo/cgs_protocol-4.png)

To achieve this the special message from $i$ to $j$ has a dual purpose of being the 
**endpoint** of which $i$ to $j$'s messages must be marked (if $j$ is already red).