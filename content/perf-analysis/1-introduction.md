---
layout: default
title: Introduction to Performance Analysis
permalink: /pa/ch1
---

# Introduction

## Three types of system evaluation methods

1. Measurement: experiment with actual system
2. Analytical soln: exp with math model of system
2. Simulation: exp with fully executable model of system

Evaluation of how diff params affect model/real systems.

## Performance

Performance is how well a system performs job
- latency (time)
- bandwidth (rate)

Application OR Server/Machine usually required to fulfil constraints of 
- time deadline
- cost of rscs (good utilization)
- energy cost

Inputs: #of jobs (workload)
^
|  (latency)
v
outputs

due to these dimensions and interrelatedness
complexity of measurements
sharing of rscs (congestion; queue. 
e.g. memory bandwidth. limited, contenton issued)
how does performance scale?

PERFORMANCE COST RATIO
- pick 2 trade off: High perf, On time, Inexpensive

how is pwrf evaled

- on high (expensive rsc)
  - overhead of comp rsc
  - measurement, simulation of model, analytic of model
- on low end: trend analysis and rules of thumb

analytic
jobs move from queue X to Y in a certain pattern 

simulatiom:
modelling of impt features of syst

measurement:
the most credinle
only works if the system or app already exists
may be intrusive to the other parts or other tasks
requires ext software tools

Goal: 
- client promises
- a balance system wo bottlenecks
- Compare alt designs
- capacity planning:increase in number of jobs inevjtable
  - how to deal with system when it saturated
  - linear scalability until **knee point**
  - 100% utilization = saturation pt

## Knowns and Unknowns

pa tools often check known knowns/unknowns

but impossible to look into unknown unknowns

## sys perf in real systems

full computer system, usually models not scalable

usually use tools

Full Stack means from application to baremetal

## SDLC additions

- canary testing
- blue green dev

## Perspectives

- top down (app devs): 
- bottom up (sys admins): rsc analysus

## Challenges

- subjectivity: need to cmp with past data etc.
- complexity: 
    - hypotheses required and experiements
    - failures can cascade: one failure's res can cause another
    - unexpected rls btwn bottlenecks
    - variation of workloads


## What is Observability:

can we actually mrasure our things?

we cant measure without observability tools

- program counters (HW)
- profilers (flame graph. cpu is common target)
- tracing (recording of event data)

in prod envs. such tools will steal workload from
prod tasks through resource contention

Virtual mAchines cannot directly access such HW counters
but progress is being made

Alerts
^
|
Metrics
^
|
Statistics
^
|
Counters

## Types of parameters and outputs

Workload params vs System params

Synthetic workload .ust follow the discovered workload parameters
