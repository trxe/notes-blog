---
layout: qna
title: Dynamic Programming and Greedy Algorithms (Questions)
usemathjax: true
permalink: /algo2/dp_greedy_prob
---

# DP, Greedy Problems

## General

1. If the question asks for an algorithm that extends LCS, then don't bother writing out the new algorithm, just deriving the recursive solution would do (see Question [2](#question-2))
2. Remember to do **housekeeping**:
   1. Time complexity
   2. Optimal substructure (via a **cut-and-paste** argument)

## Question 1

<div class="question"> Longest common subsequence for $m$​​ input strings. </div>

## Question 2

<div class="question"> Minimum edit distance algorithm based on LCS. Edits allowed are add, delete, replace. </div>

*Solution*: Let $e(n,m)$​​ be the minimum edit distance for the substring $X[1,n]$​​ and $Y[1,m]$​​.

- Base cases $e(0,j)$​​ or $e(i,0)$: $X$​​ or $Y$​​​​​ is empty, hence all the characters of the other string being `delete`d is the optimal solution
- Induction hypothesis: Suppose for some $n,m$ that $e(n, m-1), e(n-1, m-1), e(n-1, m)$ are optimal.

If the $n$th character of $X$ is not the same as the $m$th character of $Y$​​, the optimal edit distance must be that of the substrings excluding this last character.

If the $n$​​​th character of $X$​​​ is not the same as the $m$​​​th character of $Y$​​​, it can be either added to $X$​​​​, changed from the last of $Y$​​​ or added to $Y$​​​​​. Hence we choose the option that would have had the least edits so far, and add one more edit to the count.


$$
e(n,m) = 
\begin{cases}
m & \text{if } n = 0 \\
n & \text{if } m = 0 \\
e(n-1, m-1) & \text{if } X_n = Y_m \\
1 + \min\{e(n, m-1), e(n-1, m-1), e(n-1, m)\} & \text{otherwise } \\
\end{cases}
$$


**Housekeeping:**

1. Time complexity: We need to build an $nm$ array with each value being calculated in $\Theta(1)$​ by the scheme above.
2. Optimal substructure: From the equation shown, if the subproblems $e(n, m-1), e(n-1, m-1), e(n-1, m)$​ are not optimal, then by the cut-and-paste argument, $e(n,m)$​​ is not optimal as well.

## Question 3

<div class="question"> Given $\text{price}[k]$ which returns the price of a rod of length $1 \leq k \leq n$, derive an algorithm to cut and maximize the potential revenue of selling a rod of length $n$.</div>

## Question 4

<div class="question"> You are given an array $A$ of n positive integers with total sum $S$ and a value $k$. Design a dynamic programming algorithm to find (if exists) $k$ (disjoint) partitions of the input array so that the
    sum of numbers in each partition is $S/k$.</div>

*Solution*: 

1. Edge cases:

   1. If $n < k$​ return **False**.
   2. If $k = 1$​​ return **True** (the partition is $A$​​).
   3. If $S \mod k \neq 0$ return **False**.

2. Represent the array as a bitmask $\overline{x_1 x_2 \dots x_n}$​​, with $x_i = 1$​​​​ indicating presence in the partition

3. Let $\text{dp}(\overline{x_1 \dots x_n})$​ be **sum** of all the elements in this mask's partition (indicated by 1 bits), **modulo** $k$.

   - $\text{dp}(\overline{x_1\dots x_n}) = {\sum_{i=1}^m A[i] (x_i) \mod k}$

   - We want to find a partitioning using all elements in $S$​​, i.e., verify if $\text{dp}(1 \dots 1) = 0$.

4. Due to the additive property of congruence, $(x \mod k) + (y \mod k) = (x+y) \mod k$​. Hence we have​​

$$
\text{dp}(\overline{x_j0\dots 0x_i\dots x_n}) = (\text{dp}(\overline{x_i\dots x_n}) + A[j]) \mod k 
$$

```python
# initialize dp (size 2^n)
dp = [-1 for i in range(2^n)]

for mask in range(2^n):
	for i in range(n):
		if (mask & (1 << i) == 0) and dp[mask] + a[i] <= target:
			dp[mask | (1 << i)] = (dp[mask] + a[i]) % target

return dp[(1 << N) - 1] == 0
```

## Question 5

<div class="question"> Is the following greedy strategy for changing $n$ with denominations $\{Q, D, N, P\}$ optimal? <br/> Suppose the amount left to change is $n$; take the largest coin that is no more than $n$; subtract this coin’s value from $n$, and repeat.</div>

**Note:** $\{Q, D, N, P\} = \{25, 10, 5, 1\}$, as in quarter, dime, nickel, penny.

*Solution*:

1. Setup:
   - Let $S$​​ be the set of coins in an optimal solution.
   - Let $i = \max(n, Q, D, N, P)$​​​​ i.e. the first coin chosen by the greedy algorithm.
   - If $k$, the largest denomination in $S$ is such that $k \neq i$, is another solution $S'$ starting with $i$. 
2. Hence our strategy is to show for each case that if $k < i$​, then must be a better solution $S'$.

**Case 1**: $i = N$​​​​ = 5. The solution $S$​​​​ must have 5 or more $P$​​​. Since $S' = S \setminus \{5P\} \cap \{N\}$​ is smaller than $S$​, this contradicts the optimality of $S$​.

**Case 2:** $i = D$​​​ = 10. The solution $S$​​​ must have either:

- $10P$​​​​​​​. Since $S' = S \setminus \{10P\} \cap \{D\}$​​​​​​​ is smaller than $S$​​​​​​​, ...
- $5P, 1N$​​. Since $S' = S \setminus \{5P, 1N\} \cap \{D\}$​​ is smaller than $S$​​​, ...
- $2N$​​​​. Since $S' = S \setminus \{2N\} \cap \{D\}$​​​​ is smaller than $S$​​​​​, ...
- There are no combinations of $P, N$ that do not sum to 10.

**Case 3:** $i = Q$​​​​ = 25. The solution $S$​​​​ must have either:

- A combination of coins summing to 25: Since $S' = S \setminus \{\text{sum 25 coins}\} \cap \{Q\}$​​ is smaller than $S$​​​, ...
- **No such combination**: Minimum number of such coins is $3D$​ = 30. Since $S' = S \setminus \{3D\} \cap \{Q, N\}$​ is smaller than $S$​​, ...

Hence in all three cases, we proved by the cut and paste argument that the optimal solution should always pick the largest possible coin, as per the greedy algorithm.

**Housekeeping**:

1. Optimal Substructure Property: Let $P(n)$​​​​ be the problem of making $n$​​​​. Suppose the greedy solution first chooses $k$​​​​ cent coin. Let $S(n-k)$​​​ be optimal solution to $P(n-k)$​​, and $S(n)$ be the optimal solution to $P(n)$​. Then clearly minCost(S) = minCost(S′)+1, which is optimal as $n > n-k$. Thus the optimal substructure property clearly follows.

<div style="background-color: #eee; padding: 0.5em">
	Contradiction for non-standard denominations: <br />
    If we let $\{Q, D, N, P\} = \{1, 30, 110, 200\}$, and $n = 120$, by the greedy strategy, since $120 = 110 + (120-110)(1)$, the optimal solution (minimum coins) is 11. However 120 can be changed with 4 coins ($120 = 4 (30)$).
</div> 

## Question 6

<div class="question"> You are given $n$ events, each taking 1 unit of time. You earn $g_i$ if you schedule the event before $t_i \in \mathbb{R}$ for each event $x_i$. Derive an scheduling algorithm to maximize your profit.</div>

*Solution*: **IN PROGRESS, TOO CONFUSING**

**Important:** Since each even only takes 1 unit of time, an optimal scheduling can be back to back from $t = 0$, i.e. schedule is $x_i \rightarrow [0,1), x_j \rightarrow [1,2), \dots$​ etc.​ Hence we only need to compare amongst the events that have the same $\lfloor t_i \rfloor$​ time, as every other unit time will be scheduled.

- First sort the jobs by time (smallest to largest in this example). 
- For each unique time $t$, put all the jobs such that $\lfloor t_i \rfloor = t$ in a Priority Queue with $g_i$ as the key.
- `extractMax` $k$​ from the queue and assign event $k$ to $t$​.

**Greedy choice property:** Consider an optimal solution $S$ in which $x+1$ events are scheduled at time $0, 1, \dots x$​​. Let event $k$ be the last job run in $S$.

**Case 1**: $\lfloor t_k \rfloor = \lfloor t_n \rfloor$​​.  By our greedy choice: $g_k \geq g_n$​

```python
for i in range(n):
	if time[i] >= T:
		schedule i
		T = time[i] + 1
```

## Question 7

*skipped*

## Question 8

<div class="question"> Given a set of $\{x_1 \leq x_2 \leq \dots \leq x_n\}$ points on a real line, determine the smallest set of unit-length closed intervals containing all points $x_i$. </div>

*Solution:* For every last uncovered point, add a new interval and skipped the points that are covered by the new interval.

**Greedy choice property**: Let $x_k$ be the point with the maximum value. There will exist an optimal solution containing

