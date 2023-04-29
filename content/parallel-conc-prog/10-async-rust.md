---
layout: default
usemathjax: true
title: Asynchronous Rust
permalink: /pcp/ch10
---

Threads have a lot of overhead.

- thread give up the rest of the CPU time slice when blocked!
- Memory overhead
  - each thread has its own stack space
- lightweight threads require a runtime

non-blocking I/O

- `epoll` gets a list of file descriptors ready for I/O
  - event loop
  - upon client request, `epoll` returns the list of fds
  - client can `read()` from each

## Futures

State of each thread needs to be managed. e.g.
- where you are in the thread's progress
- was i waiting for something to send to me
- what did the last client send me

A `Future` allows us to keep track of a in-progress operation together 
with the state in one package.

## Zero cost abstraction

code that you can't write better by hand because the layers of abstraction disappear at compile time.

Futures are also a zero-cost abstraction. The binary code has no allocation and no runtime overhead

- building async state machines
- `Future` trait: `poll(context)`

reactor: Event loop executor + a Context

when `poll` is called, a `Context` structure is passed in, which includes a wake() function.

An executor loops over futures until some thread makes progress. 

`Tokio`: `mio.rs` + `futures.rs`

executors can be **single/multi** threaded

## Combinators

Futures are combined with **combinators** e.g. `and_then`

Poor ergonomics: mixing of synchronous and async ops in the chain + compilcated syntax

However syntactic sugar introduced:
- `async` and `await?`
- `async fn` returns a future

Never block in an async code. Async tasks are cooperative and not preemptive.
await is only used in async functions
