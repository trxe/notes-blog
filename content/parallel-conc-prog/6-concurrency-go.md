---
layout: default
usemathjax: true
title: Concurrency in Go
permalink: /pcp/ch6
---

# Concurrency in Go

Task switching traditionaly used to achieve concurrency.

- **Concurrency**: concurrent **structure** (possible to not be in true parallel)
- **Parallel**: concurrent **execution**

C++ paradigm:
1. Model programs as threads
2. Synchronize memory access
3. Use thread pools

**Go** paradigm:
1. Model programs as tasks
2. Synchronize tasks by **communication**

## Features of Go

1. Memory safety
2. Garbage collection 
3. CSP: Communicating Sequential Processes

## Concurrent design

### Data parallelism

Design && Limiting factor
1. More processes but limited carts
2. More processes and carts but bottleneck @shared receptacle & need to synchronize.
3. More processes and carts and **distributed** receptacle

### Task parallelism

Finer grained concurrency. **Pipeline** parallelism.

## Task dependency graph

DAG
-  node:task
   -  value:expected exec time.
-  edge:control dependency
-  properties:
   -  critical path length: slowest completion time
   -  degree of concurrency

# Performance Speedup

Speedup: best sequential execution time over the parallel execution time.

$$
S_p(n) = \frac{T_\text{best seq}(n)}{T_p(n)}
$$

Amdahl's law: speed up limited by the sequential fraction $f$ 
(time of program that **must** execute sequentially.

## Communicating Sequential Processes (Which are actually **Functions**)

Developed by Hoare 1978.

Refined to **process calculus**.

# Go stuff

## Abstractions

1. **Goroutines** [Concurrency]
   1. Each `go function` runs on OS thread.
   2. Think task in a task pool
   3. Cheaper than threads.
   4. **Same address space** as other goroutines (shared memory)
2. **Channels** [Communication]
   1. Goroutines can write to and read from **FIFO** channel (like a **pipe** in linux).
      1. `channel<-` and `<-channel`
   2. `select` statement
   3. Ensure dependencies are fulfilled

### Goroutines

Runtime multiplexes goroutines onto OS threads. 
Multiple goroutines in a thread "bucket".
**Decouples concurrency from parallelism**.

Scheduling algorithm

When a goroutine blocks, the **thread** blocks.

Goroutines are not garbage collected.

Goroutines need to have join point specified

- Using a **sync.WaitGroup** as a **barrier**
- `defer` to execute on exit scope

### Channels

- Bidirectional: `make(chan myType)`
  - By default send/rcv **blocks** until the other side is ready.
  - By default has a buffer of size 1
  - Reading from closed channel yields default value
    - Close a channel to immediately unblock all
- Receive chan: passed in as a view via parameter such as `func(myCh <-chan myType)`
  - To receive from a channel: `myVal := <-myCh`
- Send chan: `func(myCh chan<- myType)`
  - To send through a channel: `myCh <- val`

To loop over an entire channel via **range**;

```go
for integer := range intStream {
   fmt.Printf("%v ", integer)
}
```

**Buffered channel**: `make(chan myType, bufCount)`

Send will block once buffer is full, until a value is read from the buffer.

![Go Channel Specification](/notes-blog/asset/img/pcp/go-chan-api-spec.png)

### Ownership of a channel

**Owner**: Write-access view of a channel
  1. instantiates, 
  2. writes, 
  3. passes ownership,
  4. closes a channel

**Consumer**: Read-access view of a channel
  1. Know when channel is closed
  2. handle blocking

### `select` statements

Go's blocking equivalent of switch. it
- Polls on a number of channels
- **Switches** to **non-blocking** channels in cases.
- Always non-blocking if there is a `default` case.

1. Composing channels together 
2. Handles multiple channels requiring different methods e.g.
   1. cancellations
   2. timeouts
   3. waiting
   4. default vals (closed channel)
3. Note that cases statements are not tested sequentially (any non-blocking can be chosen)

## Tips

Pseudo-random:

```go
c1 := make(chan int); close(c1)
c2 := make(chan int); close(c2)
v1 := 0
v2 := 0
for i = 0; i < 1000; i++ {
   select { // v1 approx same as v2
      case <-c1:
         v1++
      case <-c2:
         v2++
   }
}
```
Timeout:

```go
select {
   case <-chanToWaitFor:
      doSmth()
   case <-time.After(1 * time.Second): // select may still choose this when there are non-blocking channels
      fmt.Println("Timeout")
   // default: // do some work
}
```

## Go memory model

1. Happens before
   1. RW in single goroutine must obey **program order** (sequenced before)
   2. Op order **observed** by two different goroutines **may differ**
   3. Transitive closure of sequenced before and synchronized before relationships
2. Synchronization of gorotines and channels (sync-before)
   1. `go` stmt that **starts** new goroutine is **sync-before** the goroutine's execution
   2. `send` is sync-before `rcv` on the same channel
   3. `close` is sync-before a `rcv` returning zero-value
   4. `rcv` on unbuffered channel is sync-before send on that channel
   5. `k`th rcv on buffered channel C of size $C$ is sync-before `k+C`th send
3. 