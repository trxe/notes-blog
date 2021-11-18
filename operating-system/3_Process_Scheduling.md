---
layout: default
usemathjax: true
permalink: /os/ch3
---

# 3 -- Process Scheduling

## Concurrent processes

- Virtual parallelism: 1 CPU core, **illusion** of parallelism [time-slicing]
  - **Interleaved Execution** (Context switch)
    - Switch from context of running process A and B. 
- Physical parallelism: Multi core, true parallel execution

## Scheduling in OS

- Scheduling problem: Multiple processes **ready-to-run**, which to choose?
- *Scheduler* (HOW, i.e. the mechanism): Making the scheduling decisions
- *Scheduling algorithm* (WHEN, WHICH i.e. the policy)
  - multiple ways to allocate (process environment influences a lot!)

## Process Behaviour

- **CPU-Activity.** Compute-bound processes spend majority of time here. Computation primarily.
- **IO-Activity.** IO-bound processes spend majority of time here. Request/receive from I/O devices.

## Processing environments

- **Batch processing.** No user interaction, responsiveness.
- **Interactive.** Active user interaction with system, responsive (low + consistent response time.)
- **Real time processing.** Deadlines are crucial, typically periodic.

## Criteria for scheduling algos

For **all** processing environments:

- *Fairness* of CPU time: per process/user basis. No starvation i.e. process is never run.
- *Utilize all* parts of the computing system (minimize idle parts).

**When** to perform scheduling:

- *Cooperative* (non-preemptive): A process is scheduled until **blocked** or **give up CPU** voluntarily.
- *Preemptive* (If a process is not cooperating): A process is given fixed time quota. After quota, block this process and pick another.

**Context switch**:

- Save a few register values from current process A to PCB
- Restore a same values from process B from PCB

![Context switch (oversimplified)](images/context_switch.jpg)

![Process scheduling (point 2 - 3 is described by Figure 1.)](images/context_switch2.png)

## Batch Processing

### Criteria

- Turnaround time: time from start to finish.
  - Waiting time: time spent waiting for CPU
  - Execution time
- Throughput: Number of tasks finished per unit time.
- CPU utilization: % time CPU is working on task (desired: 100%)

### First-come First-served (FCFS)

- Non-preemptive.
- Put on a FIFO queue. **No starvation** (since number of tasks before any task X is always decreasing.)
- However if tasks are added from most TUs to least, it will have the highest average waiting time
  - which can be shortened just be reordering the tasks to from the least to the most TUs
- Also **convoy effect**.
  - When highest *burst time* (computing time) compute-bound job is at the front of the queue, 
  - shorter IO-bound jobs are **waiting**, so **IO devices are idling**
  - after compute-bound job is done, IO-bound jobs are running, so **CPU is mostly idling**.

![convoy](convoy.png)

### Shortest Job First (SJF)

- Select task with smallest total CPU time (minimizes average waiting time)
- Requires total CPU time needed for a task in advance.
  - Guess via previous CPU-bound phases
  - $P_{n+1} = \alpha t_n + (1-\alpha)P_n$​​​ where $P_n$​​​ is the last predicted time of most recent task $n$​​, and $t_n$​​ is the actual run time of task $n$​​​, and $\alpha$​ is the weight of the most recent event

### Shortest Remaining Time (SRT)

- Between current task and new task, choose to block the task with the shorter remaining time
  - e.g. A is 8 TUs, and scheduled at $t = 0$. B is 3 TUs and is added to the schedule when it is introduced at $t=1$ as $3 < 8 - 1 = 7$.
  - This means that by continuously scheduling shorter processes, a longer process can be starved for a very long time.

## Interactive environment

- Aim: **predictable**, **low** response time (time from request to response)
  - Preemptive preferred.
  - Scheduler runs periodically:
    - How to ensure periodical "taking over" of scheduler?
    - How to ensure user program never stops the scheduler?

### Periodic scheduling

![Timer interrupt, that cannot be intercepted by any other program, invokes the scheduler.](timer_interrupt.jpg)

- Interval of Timer Interrupt (ITI): 1ms ~ 10ms **basically the time unit**.
  - Timer Interrupt: Response by the *processor*
- Time Quantum: $c \cdot \text{ITI}$​​​, the execution duration given to a process **how many TUs in a "cycle"**.
  - generally having too high $c$ is bad; it means too many unused interrupts.

## Scheduling Algorithms

### Round Robin

- Given $n$ tasks and quantum $q$, waiting time for a task is bound by $(n-1)q$
- Timer interrupt needed
- Time quantum:
  - Big quantum: Better CPU utilization, longer wait times.
  - Small quantum: Bigger overhead (worse CPU utilization), shorter wait times.

### Priority Scheduling

- Priority queue (each task given a priority value)
- Variants:
  - Preemptive ver.: New higher priority processes preempts running process with lower priority.
  - Non-preemptive ver.: Late-coming high priority process has to wait for the next round of scheduling.
- Problems:
  - Starvation of lower priority processes. *Possible solutions: 1. decrease the priority of the currently running processes after each time quantum. 2. give a time quantum for the running process.*
  - Priority Inversion. 
    - e.g. A, B and C have high to low priority, and appear last to first.
    - C is executed first and at some point locks a file x.
    - B then appears and executes.
    - A then appears. It tries to execute but fails because it requires file x which is locked by C.
    - if B is a long process, A is unduly starved.

### Multi-level Feedback Queue (MLFQ)

- How to schedule without perfect knowledge?
  - How to minimize both response time for IO-bound, turnaround time for CPU-bound
- Adaptive: learn process behaviour via a few simple rules!

1. *Basic rules.* Let $P(A)$ be the priority of job $A$.
   1. $P(A) > P(B) \Rightarrow A$ runs
   2. $P(A) = P(B) \Rightarrow A$ and $B$ run in round-robin.
2. Priority setting
   1. New job $\Rightarrow$ Highest priority
   2. used up time quantum $\Rightarrow$ Priority **decreases**
   3. gives up or blocks before quantum ends $\Rightarrow$​ Priority **retained**

### Lottery Scheduling

- Completely random assignment of ticket numbers. When a scheduling decision is needed, a ticket number is chosen and CPU time is awarded.
- Expected resource usage % = % of lottery tickets held.
