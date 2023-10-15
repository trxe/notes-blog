---
layout: default
title: Tools and Observability
permalink: /pa/ch7
---

# Observabiity

Tool coverage, types, observability sources

### Categories

1. Observabilty
2. Static
3. Benchmark (load test)
4. Tooling

### PA Problem statement

Without something suspicious observation is just monitoring.

1. Suspicious
2. History of performacne
3. History of hw/sw changes
4. Perf degradation measurement
5. Does poblem affect other ppl?
6. Environment?

## Methods

### USE Method

Utilization, Saturation, Errors

### RED Method

### Monitoring

Recording of performance statistics over time.
Historic values help provide patterns for our current.

## Observability tools

1. Counters: hw counters help provide state/activity data
2. Profiling: use of tools performing sampling of events
3. Tracing: event trace.

- Counters: 
  - integer vars
  - hard-coded in the software
- performance tools sample the counters' values and calculate the stats
- metrics are the statistics selected to evaluate the target.

Time series metrics are all you need to resolve perf issues.

## Profiling

taking subsets of measuremenets to paint coarse picture of **target**

CPU / GPU are common targets

## Tracing

System calls `strace` (traces the TRAP event from USER and KERNEL space.)

Network packets:  `tcpdump`

General purpose tracing tools: Ftrace, BCC, bpftrace etc.

- Static instrumentation
- Dynamic instrumentation
- BPF [Berkeley Packet Filter] for programmablitty
  - eBPF is a suite of tools that are useful for system event tracing.

## tool Coverage

1. Single multi-resource
   1. Multitools: `perf`, `Ftrace`, BCC, bpftrace etc.
   2. Most tools are single device/resource.


## Some linux tools

- uptime
  - 1, 5, 15min
- top
  - system and per-process interval mem
  - Sometimes misses short-lived processes (5 to 10s interval snapshots)
- vmstat
- iostat
- free
- strace
  - not for the whole system cos it's too expensive to trace all the traps
- tcpdump
  - packet sniffer
  - packet sequences study with timestamps.
- netstat

Fixed counters vs Event-based counters

- Fixed: HW counters
- Event-based counters: you should enable them only when you need.
  - As they introduce overhead.
  - Profiling
  - Tracing

The overhead of monitoring has to be taken into account as well.

`sar`:  System Activity Reporter.
- uses `cron` to exec some jobs.
- 