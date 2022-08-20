---
title: Word RAM model and Asymptotic Runtime
usemathjax: true
permalink: /algo2/l03
---

$\require{color}$

# Iteration

Iterations = Loops sequentially processing input elements

How do we show correctness in iterative algorithms? **Loop invariant**

- If $P(0)$ holds and $P(i-1)$ is true for subarray $A[i-1]$ before the loop
  - **Initialization**
- and $P(i-1) \Rightarrow P(i)$ for subarray $A[i]$ at the end of the loop
  - **Maintenance**
- then $P(n)$ is true for all $n \ge 0$
  - **Termination**
  - The invariant should imply the algorithm is correct for $A[n]$ i.e. $P(n)$ is true
- Inductive proof!

# Divide and Conquer (via Recursion)

Correctness is proved via **strong induction**:
- Prove the **base case(s)**
- Show that $P(k-1) \wedge P(k-2) \wedge \dots \wedge P(n_0) \Rightarrow P(k)$.

Hence derive and solve a recurrence!

<div class="question"> 
Use a recursion tree to give an asymptotically tight upper bound to the recurrence 
$T(n) = T(n-a) + T(a) + cn$ where $T(a)$ is a constant and $a \ge 1, c > 0$.
</div>

# Solving Recurrences

## Method 1: Substitution

Guess a bound ($O(f(n))$) and use **induction** to prove it.

## Method 2: Recursion Tree

Convert the recurrence into a tree. It can be used to prove the master theorem (see below).


## Method 3: Master Theorem

Solving recursions of the form 

$$
\textcolor{red}{ T(n) = a T(n/b) + f(n) }
$$

(note $a \ge 1, b > 1, \lim_{n \rightarrow \infty} f(n) >  0$).

## $f(n)$ grows slower than $n^{\log_b{a}}$

$$ 
f(n) = O(n^{\log_b{a} \textcolor{red}{- \epsilon}}) , \epsilon > 0 
\Rightarrow \textcolor{red}{T(n) = \Theta(n^{\log_b{a}})}
$$ 

## $f(n)$ grows similar rate as $n^{\log_b{a}}$

$$ 
f(n) = O(n^{\log_b{a}}\textcolor{blue}{\lg^k n}), k \ge 0
\Rightarrow \textcolor{blue}{T(n) = \Theta(n^{\log_b{a}}\lg^{k+1}{a})}
$$ 

## $f(n)$ grows faster than $n^{\log_b{a}}$

$$ 
f(n) = O(n^{\log_b{a} \textcolor{green}{+ \epsilon}}) , \epsilon > 0 
\textcolor{green}{\text{ and } af(n/b) \ge c f(n)}
\Rightarrow \textcolor{green}{T(n) = \Theta(n^{\log_b{a}})}
$$ 

The master theorem can be proved for all three cases with the recursion tree.

The full proof is left as an exercise fo rhte reader.

![Recursion tree of the master theorem](/notes-blog/assets/img/algos2/l03-recursiontree-master-thm.png)

<div class="question">
Compute $f(n,m) = {a^n \mod m}$ for any integer $n,m$ (assuming $n,m$ takes up constant number of words).
</div>

<div class="question">
Given a list of stock prices by day, find the pair of dates to buy and sell on
that returns the greatest profit. 
Come up with solutions to this problem by 1. iteration and 2. recursion.
</div>