---
layout: default
usemathjax: true
permalink: /parallel/ch5
---

# Performance

### Perspectives

1. User: Response Time (shorter program execution time)
2. Computer manager: Throughput (work/time)

### Performance Factors

In decreasing level of abstraction

1. Programming model
   1. Algorithm
2. Computational model
   1. Analysis
3. Architectural model
   1. Lowest level

- User CPU time: executing program
- System CPU time: time executing OS routines
- Waiting time: IO waiting time and execution of other programs.

### Memory Access Workflow

Read access (load)

Write access (store)

Let $N$ = instruction count

$$
T_{no_cache} = ( CPI(A) \cdot N + CPI_{rw} \cdot N_{rw} ) \cdot T_{cycle}
= ( 2N + 0.33 \times N_read \times 100 ) \cdot T_{cycle}
= (2 + 33)N \times \frac{1}{f}  = 35 N/f
$$

$$
T_{double} = ( CPI \cdot N + 0.33 N \times R_{miss} \times CPI_{rw} ) \cdot T_{cycle}
= (2N + 0.33 \times 0.02 \times N \times 200) \cdot T_{cycle}
= (2 + 0.66)N \times \frac{1}{f}  = 2.66 N/f
$$

Small problem size:
- Parallelism overhead > benefit
- high granularity
- Parallelism appropriate for small machine but not for large machine ???????????
  - Memroy contention

Large problem size:
- overhead < benefit
- Key workset may not fit in a smal lmachine, or key working set exceed cache capacity

Scaling constraints:
- Particles per processor in parallel N-body sim
- Transactions per proc in a distributed DB
- Problem size combines params

Properties
- PC (Problem constrained): same problem faster with parallel computer
- TC (Time constrained): work not exceeding time
- MC (Memory constrained): work not overflowing memory buffer

### Laws

#### Amdahl's Law

Speedup of parallel execution is limited by the fraction of the algorithm that cannot be parallelized (f).

$f$ is the bottleneck.

insert slide

#### Gustafson's Law

$S_p(n)  = \frac{T_\star(n)}$
