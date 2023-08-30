---
layout: default
title: Capacity Planning
permalink: /pa/ch2
---

# Terminology

- Latency: time spent waiting to be serviced
  - Overloaded term (sometimes it can refer to response time)
- Response time: time for an operation to complete (waiting time/latency + service time)
- Saturation: amt/degree of queued work
- Bottleneck: limiting factor (like reagent) for the system
- Workload: Input to the system or load applied, jobs
- Cache: Faster storage area tier

PA: Analysis --> detect bottleneck --> find bottleneck --> solve bottleneck --> analysis

Time scales

**Utilization**:
1. **Time** based: E[time server was busy]
   1. $U = \frac{B}{T}$; U = utilization; B = total time system busy over T the observation period
2. **Capacity** based: System/component's ability to deliver amount of throughput
   1. Proportion of system/component's resources currently working 
   2. @100% capacity-based utilization; **saturation** has been reached. <100%: no worries

**Caching**
- caches are also implemented in software.
- caching algorithm may be the bottleneck.
- hit ratio = hits / (hits + misses); higher is better
- cache miss rate: misses per second; lower is better

# Perspectives in System Performance

1. Resource Analysis Perspective [by sysadmin]
   1. Start @devices (resource level)
   2. Includes **perf issue investigations**, and **capacity planning**
   3. Demand supply
2. Workload Analysis Perspective [by devs]
   1. Targets 
      1. Requests (workload applied) 
      2. Latency (response time) 
      3. Completion (error rate)
   2. Metrics
      1. Throughput
      2. Latency

# Methodologies

No clear methodology until recently!

1. **Tools** (OS specific application tools)
   1. List avail. perf tools
   2. For tool T, list useful metrics
   3. For metric M, list ways of interpretation
2. **USE** (Utilization - Saturation - Errors)
   1.  Resources: CPU/RAM/NIC/STORAGE/ACCELERATORS
   2.  Some rscs cannot be fully monitored?
   3.  Machine health
3. **RED** (Request rate - Errors - Duration)
   1. Usu. cloud services/microservice. Check per svc.
   2. User health
4. Workload characterzation
   1. Inputs rather than resultant perf.
   2. **Who** causes laod
   3. **Why** load called
   4. **What** are attributes of the load
      1. IOPS/Throughput/Direction (R/W).
      2. Include the variance when possible/appropriate
   5. **How** load changing over time
5. **Monitoring**
   1. perfstats over time.
   2. For capacity planing/quantifying growth/peak usage
   3. Time series: Historic values (time-based patterns)

# Capacity planning

- How well handling load as it scales.
- Predicting when service fails as the workload evolves
- Determine cost-effective ways to delay saturation.
- Challenges:
  - usage load fluctuation 
  - peak usage
  - cost of provisioning

**Availability**: 
- Total system uptime/Total observation period x 100%
- On-premise uptime availability stndard: 99.999%
- Service Outages: Opposite of availability

This is only relevant for **fixed amount of resources** (c.f. 15 years ago?):
1. Resource for peak load (waste rscs)
2. Under-provision (lose potential revenue)

Nowadays we have **automatic scaling**.

Provisioning for resources have 3 main strategies:
1. lead
   1. add cap in **anticipation** of dd
2. lag
   1. add cap **after** full cap reached
3. match
   1. Automatic scalers: increment in small units as dd increases.

Capacity of s system:
- Inverse rls of throughput and response time.

![throughput and response time](/notes-blog/assets/img/pa/throughput_response.png)

Service-Level agreements (SLA):
- determination of what app users can expect for
  - response time
  - throughput
  - system avail
  - reliability
- tie IT costs to SLA.
- if these are not met, pay fine etc.

![Inputs and Outputs of the system](/notes-blog/assets/img/pa/io_sys.png)

1. Understand the environment: create a **workload** model
2. Workload characterization
3. Workload model validation + calibration
4. Workload forecasting: Use the workload model
5. Performance model development: Create a **performance** model
6. Performance model validation + calibration
7. Performance prediction: Use the performance model
8. Cost Model development: Use the workload model to create a **cost** model
9. Cost prediction: Use the cost model
10. **Cost/Performance analysis**

### Understanding the environment

### Workload characterization

Only via observation. Measurements can be taken; existing knowledge (e.g. another system).

We refine this model until validation/calibration passes.

- Workload intensity
- Service demand
- trying to find more general characteristics
- easier parameters

Data collection: How do we determine param values for basic components?

1. No data collection facilities available:
   1. Benchmark (Synthetic workload e.g. locust.io)
   2. Industry practice
   3. Rules of thumb (ROTs)
2. Some:
   1. All of the above and measurements
3. All/Detailed:
   1. Use measurements only

