---
layout: default
title: Leader Election
usemathjax: true
permalink: /palgo/ch7
---

# Leader Election Problem

## Motivation

- Controller/symmetry breaker in logical ring topology 
- Coordinator for centralized algorithms.
- Token first holder.

## Candidates

1. Coordinator based/Token based: NOT OK. 
   1. deciding which process to serve as coordinator/token holder is precisely the leader election problem
2. Lamport's bakery: take smallest uid
   1. Too slow: every process have to communicate with every other process in worst case
3. **Ring-based** algorithms (no unique ids)
   1. **Deterministic** algorithm in an **anonymous** ring not possible.
      1. We need the system to terminate in a non-symmetric state.
      2. Initially the system is in complete symmetry, **every node is identical** as they are anon
      3. The same process has to be run on every node. 
      4. An adverserial execution causes all nodes to once again reach a symmetric state after execution.
      5. Since the non-symmetric state isn't reached, repeat steps 2 to 5 again **forever**: the algorithm never terminates.
      6. Contradiction
   2. **Randomized algorithm**
      1. Chang-Roberts Algorithm
      2. Hirschberg-Sinclair Algorithm
4. **General graphs**
   1. **Complete graph**: send id to all other nodes. once received all, biggest id wins.
   2. **Any connected graph**: flood id to all other nodes
      1. Qn: how to know when received all? We need an **auxiliary protocol** to calculate the number of nodes.

## Chang-Roberts Algo

Cannot be anonymous; each node has UUID.

1. Nodes send `msg(myId)` clockwise
2. On rcv `msg(otherId)`,
   1. if `myId > otherId` discard
   1. else if `myId < otherId` forward `msg(otherId)`
   2. else assert `myId == otherId` and `myId` becomes leader.

**Best** case: $2n-1$ where largest next sends to smallest and smallest is ordered to largest.

So after the first pass, the only message left is the largest which just needs to pass all the way to the original.

**Worst** case: $O(n^2) = \sum_i^n i$ where largest sends to second largest and all the way until the smallest.

After each pass the number of messages in the ring is reduced by 1.

**Average** case: $O(n \log n)$

1. There are $(n-1)!$ possible non-isomorphic orderings of a ring
2. Let $x_k$ be the number of msgs caused by the election msg from $k$.
3. We calculate $E[\sum x_k] = \sum E[x_k]$.

How to calculate the expected sum:

$$
E[x_k] = \sum_{i=1}^k (i \cdot \Pr [x_k = i])
$$

1. $\Pr [x_k = 1] = \frac{n-k}{n-1}$
   - The probability that the very next node's id is greater than `myId` = $k$ is $n-k$ of $n-1$.
2. $\Pr[x_k = 2 \mid x_k > 1] = \frac{n-k}{n-2}$
   - Given that $x_k > 1$ we already know the next node's id $< k$. Hence the number of nodes that have a chance of being $>k$ is $n-2$.

![Probability stack](/notes-blog/assets/img/palgo/leader-ave-msg-proof.png)

Define $p = \frac{n-k}{n-1}$. Note that each $\Pr[x_k = i \mid x_k > i-1] = \frac{n-k}{n-i} \geq p$.

Since $\Pr[x_k = i] < \Pr[x_k=1] = \frac{1}{p}$, 

$$
E[x_k] \leq E[y] = \frac{1}{p} = \frac{(n-1)}{(n-k)}
$$

Hence 
$$
\begin{eqnarray*}
\sum_{k=1}^n E[X_k] &= n + \sum_{k=1}^{n-1} E[X_k] \\
&< n + \sum_{k=1}^{n-1} \frac{(n-1)}{(n-k)} \\
&= n + (n-1)\sum_{k=1}^{n-1} \frac{1}{k} \\
&= n + (n-1) \cdot O(\log n) \quad \text{(harmonic series)} \\
&= O(n \log n)
\end{eqnarray*}
$$

## Leader election in a general graph

### Counting nodes using spanning tree

Goal of this protocol is for **every node** to know its **own parent and children** only.

1. Initiator node sends out a request to its neighbours to become **children**
   1. children must reply if they have become their requester's child
   2. If a nodes is already a child, it will reply "I AM NOT YOUR CHILD".
      3. 