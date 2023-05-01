---
layout: default
usemathjax: true
title: Asynchronous Rust
permalink: /pcp/ch10
---

# Non-blocking I/O: Another zero-cost abstraction

On blocking I/O: What is the cost of blocking?

1. Context switch: when a method with a **system call** blocks, the entire process halts
   1. Storing regs
   2. Switch stack space
   3. Clear cache
2. Memory overhead
   1. growing stack space (because each thread has a minimum stack size)
   2. 5000 threads = 5000 stack segments = 40GB at 8MB/stack!

Instead of full threads, we could use lightweight threads that can be swapped out for each other.
However  they need a runtime.

How to 

## Case study: Reading from a stream

```rust

fn main() {
  let listener = TcpListener::bind("127.0.0.1::25565").unwrap();
  for stream_res in listener.incoming() {
    let mut stream = stream_res.unwrap();
    thread::spawn(move || {
      let mut str = String::new();
      stream.read_to_string(&mut str).unwrap(); // WHEN THE read() SYSTEM CALL BLOCKS, THE ENTIRE THING BLOCKS
    });
  }
}
```

Resolution with `epoll`: a linux kernel mechanism 

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
