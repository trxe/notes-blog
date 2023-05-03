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

How to fix this? Async await:

1. Solves high stack usage
2. Solves unnecessary context switching due to blocking
3. Achieves high degree of concurrency
4. Maximizes I/O bound processes' time slice duration.

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

Resolution with `epoll`: a linux kernel mechanism for polling file descriptors that are **ready for I/O**.

```
while true {
  ready_ids = epoll() // POLLS ALL THE READY FDs
  for each id in ready_ids {
    read(id)
  }
}
```

## Futures

State of each thread needs to be managed. e.g.
- where you are in the thread's progress
- was i waiting for something to send to me
- what did the last client send me

A `Future` allows us to keep track of a in-progress operation together 
with the state in one package.

## Zero cost abstraction

- No **global cost**: a ZCA should not hinder the performance of programs not using it
- Optimal performance: Compiles to the best performing implementation as if handwritten with primitives

Futures are also a zero-cost abstraction. The binary code has no allocation and no runtime overhead

## `async/.await` in rust

```rust 
async fn func() { }
// The above async fn is syntactic sugar for

fn func() -> impl Future<Output=()> { }

// The below is definition of the Future trait
trait Future {
  type Output;
  fn poll(&mut self, cs: &mut Context) -> Poll<Self::Output>;
  // Context includes the wake() function, as shown with the below alternative params
  // fn poll(&mut self, wake: fn()) -> Poll<Self::Output>;
}

enum Poll {
  Ready(T),
  Pending,
}
```

Futures in Rust are **lazy** so they will not do anything until they are polled i.e. when `await` is called.

Polling can only occur inside an `async` function (i.e. `await`) can only be called within `async fn`.
Then who polls the `main` function?

### Executors, Context and how futures run

Note that waiting $\neq$ blocking!

1. **Executor** required
   1. `poll()` called on the first future `main`
   2. runs code until first `.await`
   3. runs the `.await`ed future until it cannot progress
      1. If future is complete it returns `Poll::Ready(T)`
      2. Else it returns `Poll::Pending`
   4. If result was pending, *run another future*.
2. **Context** required
   1. How does the executor know which other future can be run? (1.4)
   2. When `.wake()` is called BY ONE OR MORE of the futures
      1. Executor sleeps until awoken by futures' `wait`s.
   3. Executor can use `Context` to see which Future can be polled.
   4. Implemented internally with system calls

- building async state machines
- `Future` trait: `poll(context)`

reactor: Event loop executor + a Context

when `poll` is called, a `Context` structure is passed in, which includes a wake() function.

An executor loops over futures until some thread makes progress. 

`Tokio`: `mio.rs` + `futures.rs`

executors can be **single/multi** threaded


## Combinators

Futures are combined with **combinators** e.g. `and_then`

Poor ergonomics: 
1. mixing of synchronous and async ops in the chain 
2. compilcated syntax
3. SHARING MUTABLE DATA across diff ops, painful when only 1 closure can touch the data.

However syntactic sugar introduced:
- `async` and `await?`
- `async fn` returns a future
- futures are lazy and do not run until await is called on them
- NEVER BLOCK in async code.
  - If code within a future causes the thread to sleep, the executor running that code is going to sleep
  - Asynchronous code needs to use non-blocking versions of everything (provided by `tokio`)

## State management and stackless coroutines

- Futures contain states of the possible execution.
- Executor thread has a stack but it **doesn't store states** when task-switching
- states are contained in self-generated future.

The Future returned by an async function needs to have a fixed size known at compile time.

Rust async functions are nearly optimal in terms of memory usage and allocations (zero cost abstraction).

## Tool comparison

1. Rust: async in synchronous style
2. JS: Promises but with more **dynamic memalloc** less efficient
3. Go: Goroutines are the async tasks, but **not stackless**
   1. Resizable stacks (garbage collection)
   2. Runtime knows pointers and reallocation of memory possible
4. C++20 (new)