---
layout: default
usemathjax: true
title: Threads, Tasks and Synchronization
permalink: /pcp/ch2
---

# Threads, Tasks and Synchronization in C++

To perform thread/task parallelization

## History of C++

until 2011, C++ don't have MT support
- assuming a **sequential abstract machine**
- no clarity to how the changes to other variables will be seen by other threads.
- compiler specific extension for memory model
    - few formal MT mem models provided by compiler vendors
    - don't work in the same way.
- C++98
- Application frameworks wrap underlying platform specific API

C++11:
- thread aware mem model
- stuff it can do:
  - managing threads
  - shared data
  - synchronizing between threads
  - low-level atomic ops
- conurrency to improve application performance
  - atomic ops library (closest to the hardware)
  - low abstraction penalty when using wrapper classes for the low-level facilities

## Thread mgmt

1. `main()`
2. `std::thread t(funcptr)`, `t.join()` to join
   - Using a function object (make Callable by overriding `()` operator)
3. Wait: Always make sure even when early end parent thread, the child threads are always joined
4. Detach: Child thread may discard variables in the main thread if it ends their lifetime before the thread ends.

use `std::ref` as a reference instead of `&`
open a curly bracket, close a curly bracket: that's a scope in between the brackets.

## RAII: Resource Acquisition is Initialization

enter scope = allocate and initialize (on heap if necessary).

exit scope = deallocate and destroy (from heap if necessary).

### r-values and l-values

l-values: Values to the left of the assign operator

r-values: Values to the right of the assign operator (temporaries, variables without a fixed memory address)

You pass r-value arguments in via `&&`. Can be used for move 

### `std::mutex`

- The standard guarantees
  -  prior `unlock` on the samemutex synchronize with `lock`. 
- The OS guarantees
  - `futex()` linux syscall
  - `futex`'s release-acquire semantics ensures synchronises-with.

### Locks in cpp

- `lock_guard`
- `scoped_lock`
  - guards against deadlock when locking/unlocking in the wrong order with **deadlock avoidance algorithm**
  - The order in which it locks
- `unique_lock` 
  - `std::defer_lock_t` tells the compile to lock later.
  - `std::try_to_lock`
  - `unlock` the unique lock yourself (what makes unique lock more powerful than the others.)