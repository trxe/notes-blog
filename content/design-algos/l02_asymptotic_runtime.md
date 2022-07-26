---
title: Word RAM model and Asymptotic Runtime
usemathjax: true
permalink: /algo2/l02
---

# Word RAM Model

In the previous topic we went through the comparison and query models.
These models had comparisons and queries respectively as the basic operational unit 
for their algorithms.

For the rest of the module we use the Word RAM model. This model assumes:

- One processor (no concurrent operations)
- Operates in binary
- 1 word = 4 bytes = 1 unit of RAM
- Each logical operation (+, -, *, /, AND, OR, NOT) takes **constant** number of words

**Input size**:
Depends on the problem. Can be either

- Number of items in input (especially things involving lists)
- Number of bits to represent the input (especially larger numbers)

**Running time**: counted by the number of basic instructions.

# Asymptotic notations

## O-notation, o-notation

For any $n > n_0$,
$$
f(n) = O(g(n)) \Leftrightarrow 0 \le f(n) \le cg(n) \quad \text{for some } n_0, c>0
$$

Meanwhile o-notation is a tighter restriction:
$$
f(n) = O(g(n)) \Leftrightarrow \text{for \textcolor{red}{any} } c>0, \quad 
0 \le f(n) \textcolor{red}{<} cg(n)\text{ for some $n_0$ and } n > n_0
$$

<div class="question"> Show $n^2 - n \neq o(n^2)$. </div>

## $\Omega$-notation, $\omega$-notation

For any $n > n_0$,
$$
f(n) = \Omega(g(n)) \Leftrightarrow 0 \le cg(n) \le f(n) \quad \text{for some } n_0, c>0
$$

Meanwhile $\omega$-notation is a tighter restriction:
$$
f(n) = \omega(g(n)) \Leftrightarrow \text{for \textcolor{red}{any} } c>0, \quad 
0 \le cg(n) < f(n) \quad \text{for some $n_0$ and} , n > n_0
$$

<div class="question"> Show $n^2 + n \neq \omega(n^2)$. </div>

## $\Theta$-notation

$$
f(n) = \Theta(g(n)) \Leftrightarrow f(n) = O(g(n)) \text{ and } f(n) = \Omega(g(n))
$$

or in other words,

$$
f(n) = \Theta(g(n)) \Leftrightarrow 0 \le cg(n) \le f(n) \le dg(n) \quad \text{for some } n_0, d > c >0
$$