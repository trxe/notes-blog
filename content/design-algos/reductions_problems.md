---
layout: qna
title: Reductions (Questions)
usemathjax: true
permalink: /algo2/reductions
---

# Reductions

## General

**NP-complete**: To prove that $B$ is NP-complete, choose a known NP-complete problem and show $A \leq_P B$.

**Reduction**: To prove that $A \leq_P B$, i.e. $A$ can be **reduced** to $B$​, show that

1. The transformation of $\alpha$ to $\beta$ takes polynomial time in size of $A$.
2. $\alpha$​ is a YES-instance of $A$​​ **if and only if** $\beta$​ is a YES-instance of $B$​.

**Some transformations**:

1. Independent Set $\leq_P$ Vertex Cover
2. Vertex Cover $\leq_P$​ Set Cover
3. 3-SAT $\leq_P$​ Independent Set
4. Set Cover $\leq_P$​ Integer Programming

## Question 1

## Question 2

<div class="question"> MAX-CLIQUE: Given an undirected graph $G = (V, E)$ and an integer $k$ decide whether there exists a maximal clique of size at least $k$ in $G$ (i.e., as a subgraph of $G$). Show that MAX-CLIQUE is NP-complete.</div>

Note: A maximal clique is a subgraph $S$ such that 

1. every vertex in $S$​ is adjacent to each other,
2. no vertex in $V \setminus S$ is adjacent to all $v \in S$.

*Solution:* We will show that INDEPENDENT-SET is reducible to MAX-CLIQUE. 

**Lemma**: INDEPENDENT-SET $\leq_P$ MAX-CLIQUE. 

*Proof:* For input graph for INDEPENDENT-SET, $G(V,E)$ and integer $k$, we construct $G'(V, E')$ where $E' = K(|V|) \setminus E$. That is, removing of the edges in $E$ from the complete graph of $|V|$ vertices. This will be our input for MAX-CLIQUE.

$(\Rightarrow)$ If we have a YES-instance of INDEPENDENT-SET of size $k$, then there exists some $S \subset G$ where $S$ is completely disconnected from each other. Thus in $G'$, the induced subgraph of the $k$ vertices in $S$ must be completely connected, i.e. a maximal clique of size $k$. Hence we have a YES-instance of MAX-CLIQUE.

$(\Leftarrow)$​ If we have a YES-instance of MAX-CLIQUE of size $k$​, then there exists some $S \subset G$​ where $S$​ is completely connected. Then in the ​​

If MAX-CLIQUE is can be solved in polynomial time, then INDEPENDENT-SET can be solved in polynomial time too, and thus we have a contradiction. Hence, MAX-CLIQUE is NP-complete.

## Question 3

