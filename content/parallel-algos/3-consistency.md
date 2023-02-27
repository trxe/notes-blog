---
layout: default
title: Consistency
permalink: /palgo/ch3
---

# Consistency

**Consistency**: Specifies what **behaviour** is allowed when a shared object is accessed by multiple procs.

- How to make specification **sufficiently strong**
- How to **implement** in a real program
- Determines the "good-ness" of an outcome

**Shared object**: Abstract Data Type that supports access from multiple processes.

**Mutual exclusion** is actually a way to ensure consistency by preventing unwanted behaviour

## Sequential consistency

Lamport's informal definition: The **results** are the same as if all operations by **multiple processors**
were executed in some **sequential order** specified by the program 
(i.e. executed by a single processor with no interleaving operations.)

### Formalism

**Operation**: An **invocation/response pair** of a **single** shared object by a process
- $proc(e)$: invoking process
- $obj(e)$: invoked object
- $inv(e)$: invocation wall clock time
- $resp(e)$: response wall clock time

**History** $H$: sequence of inv/res totally ordered by wall clock time

**Sequential** history:
- invocation is immediately followed by response **without interleaving of operations**
  - $\sim(inv(i) < inv(j) < resp(i) \wedge inv(i) < resp(j) < resp(i))$
- otherwise considered concurrent.

How to tell if a history is sequential:
- ordering of ops that are **atomic** i.e. no interleaving within process
- the results are possible to get with some ordering above

**Legal** sequential history:
- following what the results of a single process running this program should be like.
- sequential semantics of the data type

**Subhistory**: The subsequence of all events of either process $p$ or object $o$.
- process subhistory $(H \mid p)$
  - trivial: $H \mid p$ is **always sequential**
- object subhistory $(H \mid o)$

**Equivalency**: Two histories equivalent if they have the same set of events,
implying **all responses' values** are the **same** (though event ordering may be different).

#### Sequentially consistent history $H$ formal definition

History $H$ is sequentially consistent if
1. it is equivalent to some legal sequential history $S$
   1. *Legal means the response must be valid semantically*
   2. *Sequential means the response must immediately follow invocation in each process*
2. and $S$ preserves the **program** order in $H$.
   1. *You are free to reorder the interleaving of operations across processes.*

## Linearizability

**Linearizability**: Induces $<_H$ partial order. 
$o_1 <_H o_2$ if $resp(o_1) < resp(o_2)$ in $H$ (**occured-before** order).

This order is known as **external order**.

Linearizable history $H$:
- **Definition 1**: Execution is equivalent to some execution such that each operation happens instantaneously at some point between the invocation and response (linearization point)
  - i.e. you can come up with a set of linearization points that are legal.
- **Definition 2**: it is equivalent to some legal sequential history $S$ and $S$ preserves the **external order** in $H$
  - External order = program order + wall clock time order.

## Local property

Linearizability is a local property.

> $H$ is linearizable if and only if $H \mid x$ is linearizable (for any object $x$).

Sequential consistency is **not** a local property. See slides for example.

### Proof of linearizability being a local property

> $H$ is linearizable if and only if $H \mid x$ is linearizable (for any object $x$).

**Only if (trivial)**: if $H$ is linearizable, then it is equivalent to some legal sequential history $S$ that adheres to external order from $H$. 
The subhistory $S \mid x$ must also still be a legal sequential history that adheres to external order from $H$ which includes $H \mid x$. This shows that $H \mid x$ is linearizable

**If (nontrivial)**: Suppose $H \mid x$ is linearizable for any object $x$.

We define a digraph $G$ such that each op is a node in $G$, and each pair of nodes $a,b$ has an edge $a \rightarrow b$ 
- if $a$ directly precedes $b$ in object order of $H \mid x$ or 
- $a$ precedes $b$ in external order.

![DAG](/notes-blog/assets/img/palgo/linearizable_dag_proof.png)

*Lemma*: The digraph $G$ will always be a DAG.

*Proof*: 
Suppose for contradiction that a cycle may exist. Such a cycle **must** be of the form
$A \rightarrow B \rightarrow C \rightarrow D \rightarrow A$

1. $A \rightarrow B$ due to obj x 
2. $B \rightarrow C$ due to H
   1. We can't not have this as $B$ cannot be in both $H \mid x$ and $H \mid y$ by definition (operations are always on a single shared object).
3. $C \rightarrow D$ due to obj y
4. $D \rightarrow A$ due to H
   1. see $B \rightarrow C$

Since $H$ has an external order imposed by wall clock time and $H \mid y$ also preserves external order, then $B < C < D < A$. 
However since $H \mid x$ also preserves external order we need $A < B$ to hold as well.
Hence we have a contradiction.

**Proof of local property**: By lemma, we know that we can always achieve a 
topological ordering of the operations in the history $H$ across all the objs.
Hence this ordering $S$ is a sequential history that adheres to the external order.

But is $S$ **legal**?

- Suppose $S_x$ is the linearization of $H \mid x$
- Suppose $S_y$ is the linearization of $H \mid y$
- Let $S$ be the topological ordering, of which $S_x$ and $S_y$ are subsequences.
- WLOG for any pair of ops $(a,b)$ in $S_x$, an interleaving op $o \in S_y$ will **never change the behaviour** of $b$.
  - This is because $b \notin S_y$, and 
  - thus no op $o$ can modify the input parameters of $b$.

### Proof of the equivalence of the two definitions

**Def 1** $\Rightarrow$ **Def 2**.

- Legal seq. history: By ordering all ops by linearization points, we have a legal sequential history $S$.
- External order preserved: 
  - $inv(a) < a^* < resp(a)$ for any operation $a$.
  - If $a <_H b$ in external order, then $inv(a) < resp(b)$.
  - Since $a^* < b^*$, we are guaranteed $inv(a) < a^* < b^* < resp(b)$
  - Hence external order is preserved with the linearization points.

**Def 2** $\Rightarrow$ **Def 1**.

Let the legal sequential history $S$ be $a, b, c \dots$.

![Def2 => Def1](/notes-blog/assets/img/palgo/linearizable_def-1.png)

![Def2 => Def1 prove buffer part](/notes-blog/assets/img/palgo/linearizable_def-2.png)