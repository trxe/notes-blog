---
layout: default
title: Probability Distributions
permalink: /pa/ch4
---

# Markov models

markov models: General Birth-death processing queueing models

- Markov Chain
  - Stochastic
  - Pr(e) depends only on prev(e)
    - neighbouring states only
    - History doesn't matter: **memoryless**
  - Transition from one state to the next is $\exp$ dist as a result

**exponential** interarrival, exponential **service time** is M/M/x; hence all markov models are of this Kendall notation.

Slide 7: $\lambda_i \def$ rate of arrivals per unit time from $n=i$ to $n=i+1$.

Note on distribution of arrival rate: the general case is intractable

$$
\begin{aligned}
\Pr_j[\text{1 arrival}/dt] &= \lambda_j dt
\Pr_j[\text{1 departure}/dt] &= \mu_j dt
\Pr_j[\text{no change}/dt] = \mu_j dt
\end{aligned}
$$
