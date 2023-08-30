---
layout: default
title: Probability Distributions
permalink: /pa/ch3a
---

# Probability Distributions

Statistical models for workload are based on **prob. distributions**.

1. **Model** workload/service centers
2. **Solve** queueing models

## Glossary

**Experiment**: Process which has *various possible outcomes*.

**Sample space**: Set of all possible outcomes of experiment.

**Random variable**: $R : S \mapsto T$ where $S$ is some experiment's sample space and $T$ is some set of a number type.

### Random Variables

**Discrete Random Variable**: $0 \leq size(S) < \infty$ i.e. sample space is finite

**pmf** (Probability Mass Function): $p : x \mapsto \text{probability (out of 1) of } x$. 

- For all $x \in S$, $0 \leq p(x) \leq 1$
- $\sum_{x \in S} p(x) = 1$

**cdf** (Cumulative Distribution Function):

$$
p(x) = \sum_i 
$$

**Continuous Random Variable**: $0 \leq size(S) < \infty$ i.e. sample space is finite

## Distributions

Different distributions have different characteristics of their spread of probability values. These are controlled by parameters e.g.

- location
- size
- shape

### Discrete Distributions

**Uniform**: $r \sim U(m,n)$

- $m, n$: lower and upper limit
- evenly distributed probability throughout range of RV

**Poisson**: $ r \sim Poi(\lambda)$.

- $\lambda$: mean number of arrivals over interval $T$.
- Commonly used in queueing models to model **number of arrivals over interval**.
- $Poi_T(\lambda) = Poi_{kT}(k\lambda)$ 
  - $N(t) \sim Poi(\lambda)$ refers to a **poisson process** i.e. probability distribution of number of arrivals over a given interval length $t$.
- Splitting and Pooling

![Splitting and Pooling](/notes-blog/img/assets/pa/poisson-splitting-pooling.png)

### Continuous Distributions

**Uniform**: $r \sim U(a,b)$

- $a, b$: lower and upper limit
- Commonly used with a **bounded** rv without any other additional info available

**Exponential**: $r \sim \exp(\alpha)$

- $\alpha = \frac{1}{\lambda}$
- Commonly models **inter-arrival times** between jobs, or **lifetime** of something:
  - $\alpha = $ 1 unit time over mean number of arrived jobs in that unit time.
- Memoryless: $P(x > s + t \mid x > s) = p(t)$
  
**Normal**: $r \sim N(\mu, \sigma)$

- Commonly models: 
  - measurement error amounts
  - known extra-model factors
- Also commonly achieved when:
  - **distribution sampling** large number of independent observations for some purpose.
- z-scores
- symmetry

**Triangular**: $r \sim T(a,b,c)$

- Commonly used when, 
  - similar to Uniform, data is not easily obtained
  - knowledge of characteristics establish bounds

### Relations between Poisson distribution and Exponential Distribution

If the number of arrivals is Poisson distributed then the
time between successive arrivals (inter-arrivals) is
exponentially distributed with mean of $1/\lambda$ (converse is
also true).

## Estimators

e.g. mean, median, mode.

**Arithmetic** mean:

$$
\bar x_A  = \frac{1}{n} \sum_{i=1}^n x_i
$$

**Harmonic** mean:

$$
\bar x_H  = \frac{n}{\sum_{i=1}^n \frac{1}{x_i}} 
$$

**Geometric** mean:

$$
\bar x_G  = \left(\prod_{i=1}^n x_i \right) ^ \frac{1}{n}
$$
