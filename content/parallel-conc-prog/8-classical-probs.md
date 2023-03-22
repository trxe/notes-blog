---
layout: default
usemathjax: true
title: Classical Synchronization Problems
permalink: /pcp/ch8
---

# Classical Synchronization Problems

## Barrier 

Wait until all threads reach a point.

Two turnstiles needed in order for the state of the barrier to remain "un-reset"
in the case that one thread is ahead by 1 lap.

1. C++
   1. Single use barrier (`std::latch`)
   1. Reusable barrier (`std::barrier`)
      - Combining tree barrier: hierarchical way of implementing barrier
      - Avoids all threads spinning together

## Producer Consumer

## Readers Writers

## Dining Philosopher

Limited resources to a group of processes in deadlock-free + starve-free manner.

Deadlock avoidance algorithm: use `scoped_lock`

`footman_sem` to limit the number of competing philosophers
But starvation possible because semaphore isn't fair!

## Barbershop

Coordinate processor execution

## FIFO Semaphore

Avoid starvation and improve fairness

No concept of ownership. Arbitrarily scheduled, sem may be available but the next in line may be sleeping.

GO Implementation: Use a buffered channel (already FIFO).

C++ shared mem Implementation: Ticket Queue

## $H_2 O$

Allocating resources ($H_2$ and $O_2$ are being produced) to process

1. Must have exactly 2 $H$ and 1 $O$.
2. The currently unfinished $H_2 O$ must bond first before the next one's bonds can start forming.

## Cigarette Smokers

Agent represents OS allocating resources. (Additional task of assembling)

Smokers are the applications requiring resources.