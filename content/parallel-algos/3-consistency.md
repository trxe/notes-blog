---
layout: default
title: Consistency
permalink: /palgo/ch3
---

# Consistency

Specifies what **behaviour** is allowed when a shared object is accessed by multiple procs.

- How to make specification **sufficiently strong**
- How to guarntee efficiency

## basic shared abstract data types

- see slides

## Sequential consistency

Using sequential order
program order must preserve (exact order of each statement executed)

Formalism

we are more comofrtable with sequential execution, SINGLE PROCESS

inv(e) res(e)

history: sequences of inv/res ordered by wall clock time. this ensures TOTAL ORDER

**Sequential** history $H$:
- invocation is immediately followed by response without interleaving
- otherwise concurrent

**Legal** sequential history $H$:
- following what the results of a single process running this program would be like.

subhistory:
- process subhistory
- object subhistory

Equivalency: exactly the same set of events

A history H is SC if it is equivalent to a Seq history  preserving program order.

note that SC doesn't mean no data race


**External** histsoyr: if o1 finishes before o2 starts (and o2 doesn't start bfore o1 ends), then we have ext history/order partial order and o1 $<$ o2.

