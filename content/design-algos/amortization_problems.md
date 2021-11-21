---
layout: qna
title: Amortized Analysis
usemathjax: true
permalink: /algo2/amort_prob
---

# Amortized Analysis

## Summary

### Aggregate analysis

Let $T(n)$ be the total real cost of $n$​ operations. We want our amortized cost to be


$$
\hat c_i = \frac{T(n)}{n} \quad \text{for all } n \in \mathbb{Z}_{\geq 0}.
$$

### Accounting method

Let $c_i$​​ be the actual cost of an operation. We want our amortized cost $\hat c_i$​ such that


$$
\sum_{i=1}^n \hat c_i \geq \sum_{i=1}^n c_i \quad \text{for all } n \in \mathbb{Z}_{\geq 0}.
$$

### Potential method

Let $D_i$​ be the data structure the algorithm operates on. Find a **valid** potential function $\Phi : D \rightarrow \mathbb{R_{\geq 0}}$​ (*note: valid if for all* $d, \Phi(d) \geq 0$​), and let our amortized cost, defined as $\hat c_i = c_i + \Phi(D_i) - \Phi(D_{i-1})$​, is such that for a sequence of $n$​ operations,


$$
\sum_{i=1}^n \hat c_i = \sum_{i=1}^n c_i + \Phi(D_n) - \Phi(D_0) \leq \text{an upper bound.}
$$


## Question 1

<div class="question"> Suppose we have a list $L$ supporting $\texttt{INSERT(x,L)}$ that inserts an integer $x$ with cost 1, and $\texttt{REPLACE-SUM(L)}$ that sums all the integers in $L$, clears $L$ and store this sum in as the only element. Use accounting and potential methods to show the cost of both operations is $O(1)$.</div>

**Accounting**: 

| Operation        | Real cost $c$ | Amortized cost $\hat c$ |
| ---------------- | ------------- | ----------------------- |
| `INSERT(x,L)`    | 1             | 3                       |
| `REPLACE-SUM(L)` | $\|L\|$​       | 1                       |

On each insertion, the operation will pay 1 unit upfront and save 2 units as credit balance. When replace-sum is run, each remaining element's credit balance can be used as such

- 1 unit for removing from stack,
- 1 unit for adding value to sum.

Hence we need to charge replace-sum only 1 unit of extra cost to put the sum back into the stack $L$. Hence both operations have $O(1)$ amortized cost.

**Potential**: Let $\Phi(L_i) = \|L_i\|$​​​​ where $\|L_i\|$​​​​​ is the size of the list after the $i$​​th operation.

| Operation        | $\Phi(L_i) - \Phi(L_{i-1})$   | $\hat c_i = c_i + \Phi(L_{i-1}) - \Phi(L_i)$​​ |
| ---------------- | ----------------------------- | -------------------------------------------- |
| `INSERT(x,L)`    | $\|L_i\| - (\|L_i\| - 1) = 1$ | $1-1 = 0$                                    |
| `REPLACE-SUM(L)` | $1 - \|L_{i-1}\|$             | $\|L_{i-1}\| + 1 + 1 - \|L_{i-1}\| = 2$​      |

Since $\hat c_i$ of each operation is a positive constant, both operations have $O(1)$ amortized cost.

## Question 2

*skipped*

## Question 3

<div class="question"> Suppose we implement the following to solve the Set-Union problem: Each set is initially given a distinct color, on $\texttt{union(A,B)}$ of two sets $A,B$, each element in the smaller set is colored the same as the larger set, and on $\texttt{sameSet(x,y)}$ two elements are checked if they belong to the same set. Each node's checking and coloring, as well as requesting the set size, is $O(1)$. Use aggregate method to show that the amortized cost of $\texttt{union}$ is $O(\log n)$.</div>

**Aggregate**: We must show that for a sequence of $m$ union operations, $T(m) = O(m\log n)$.

Note that after $m$​​​​​ union operations, **at most** $2m$​​​​ elements were involved.

**Lemma**: Each element can only be colored at most $\log n$​​​​ times.

*Proof*: If the element is in a set of size $k \leq n$​​, and it has been recolored on every merge, then it must be in the smaller sets of size less than $k /2, k/4, \dots 1$. Since there are only $\log k \leq \log n$ of such sets, it can at most be recolored $\log n$ times.

Hence the total number of recolorings for at most $2m$ elements is at most $2m\log n = O(m \log n)$.

## Question 4

<div class="question"> Let $F_k$ denote the $k$th Fibonacci number. Suppose we implement a dynamic table as such: <br />
	The current table size is now $F_k$. After an insertion, if the number of elements is now $F_{k-1}$, allocate a new table of size $F_{k+1}$, move all the elements and free the original table. After a deletion if the number of elements is now $F_{k-3}$, allocate a new table of size $F_{k_1}$, move all the elements and free the original table. <br />
    Now show that for any sequence of insertions/deletions, the amortized cost per operation is still $O(1)$.
</div>

**Accounting**:

| Operation         | Real cost           | Amortized cost                         |
| ----------------- | ------------------- | -------------------------------------- |
| Insert            | 1                   | $\phi^4 + \phi^2 \approx 9.472 = O(1)$​​ |
| Insert and expand | $F_{k+1} + F_{k-1}$ | 0                                      |
| Delete            | 1                   | $\phi^4 + \phi^2 \approx 5.854 = O(1)$ |
| Delete and shrink | $F_{k-2} + F_{k-4}$​ | 0                                      |

Suppose that there are now $F_{k-1}$​​​​ elements in the table, and it has been expanded to size $F_{k+1}$​​​​. The table will next be expanded when nett $F_{k+2-2} - F_{k-1} = F_{k-2}$​​​​ elements have been newly added. When an element is inserted, it will pay 1 unit upfront and store $\phi^5 + \phi^2$​​​​ units as credit balance. Hence after the $F_{k-2}$​​​​th element has been added, we now have at least $F_{k-2} (\phi^5 + \phi^2)$​​​​ units of credit balance. The real cost of the table allocation ($F_{k+2}$​​​​) and moving of each element ($F_k$​​​​) can be paid for by the total credit balance left as


$$
F_{k+2} + F_{k} = F_{k+1}\phi  + F_{k} = F_{k}(\phi^2+1) = F_{k-2}(\phi^4 + \phi^2)
$$


Now suppose there are $F_{k-3}$ elements in the table and it has been shrunk to size $F_{k-1}$. Then table will next be shrunk when nett $F_{k-3} - F_{k-4} = F_{k-5}$ elements have been deleted. When an element is deleted it will pay 1 unit upfront and store $\phi^3 + \phi$ units as credit balance. After the $F_{k-5}$th element has been deleted we now have $F_{k-5}(\phi^3 + \phi)$​​ units of credit balance. Similar to above, the real cost of allocating the table and moving each element can be paid for by the credit balance.


$$
F_{k-2} + F_{k-4} = F_{k-3}\phi  + F_{k-4} = F_{k-4}(\phi^2+1) = F_{k-5}(\phi^3 + \phi)
$$


Hence by the accounting method we can see each operation is $O(1)$.