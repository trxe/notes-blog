---
layout: default
usemathjax: true
title: Concurrency Patterns in Go
permalink: /pcp/ch7
---

# Concurrency Patterns in Go

**Separation of concerns**:
- chunks of data **confined**
- error-handling
- data-processing **pipeline**

## Confinement

1. **Ad-hoc** : Data only modified from one go-routine (though accessible from many)
2. **Lexical** : Restrict access to shared locations
   1. e.g. exposing read-only or write-only access of a channel
   2. e.g. exposing only a slice of the array

## `for-select` loop

```go
for { // infinite loop or on range
   select {
      // Work with channels
      // Keep `select` as short as possible.
   }
}
```

## Stopping reader goroutines

Creating **exit conditions** to prevent accumulating of inactive readers: use of `done` channels




## Fan-in Fan-out

**Fan-out**: Multiple goroutines handle input from the pipeline.
- Cannot control order in which concurrent routines run
- Make sure no dependencies

Fan-in: combine into one channel
- No guanratees on order of merging

Rule of thumb to best maximize concurrency: fan-out `runtime.NumCPU()` goroutines.

## Context Management

## Fan-out fan-in

- Gen Envents Channel
- input channel
- workers
    - indepedent process inputs
- output channel
  - ADD A SERIALIZER
- printer.

`fmt.Println` is not threadsafe.

1. sort output
   1. print after **all** outputs only
   2. deadlock if interactive IO
2. Reorder buffer
   1. if waiting for event 2 and 3 comes, then put into buffer pos 3
   2. else if waiting for 3 and
3. decentralized queue (passing a token??)
   1. higher order channels

Reordering between different channels/queues.

Fix with 
- mutex like CS
- Logical clock
- consensus algo

The inherent problem in message passing.