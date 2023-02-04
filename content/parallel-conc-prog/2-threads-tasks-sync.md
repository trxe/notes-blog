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

**C++11**:
- **thread aware memory model** with well-defined specification
- stuff it can do:
  - managing threads
  - shared data
  - synchronizing between threads
  - low-level atomic ops
- conurrency to improve application performance
  - **low level facilities**
    - atomic ops library (closest to the hardware)
  - **low abstraction penalty** when using wrapper classes for the low-level facilities
    - as fast as possible on the hardware even when abstracted

## Thread mgmt with `std::thread`

```cpp
// Functional object callable type
class CallableHello {
  public:
  void operator()() const {
    cout << "callable hey\n";
  }
}

// function ptr
void hello() { cout << "hey\n"; }

int main() {
  std::thread t1(hello);
  CallableHello ch;
  std::thread t2(ch);
  t1.join(); t2.join();
}
```

1. `main()`
2. `std::thread t(funcptr)`, `t.join()` to join
   - Using a function object (make Callable by overriding `()` operator)

### Confusing C++ stuff and initialization

```cpp
// function decl of return type thread
std::thread t1(CallableHello()); // FUNCTION DECLARATION
std::thread t2{CallableHello()}; // THREAD INITIALIZATION
std::thread t3((CallableHello())); // THREAD INITIALIZATION
std::thread t4([]{ cout << "ok\n"; }); // THREAD INITIALIZATION
std::thread t4_alt([](){ cout << "ok\n"; }); // THREAD INITIALIZATION

int main() {
  t2.join(); // OK!
  t3.join(); // OK!
  t4.join(); // OK!
  // t1.join(); // CANNOT RUN!
}
```

use `std::ref` as a reference instead of `&`
open a curly bracket, close a curly bracket: that's a scope in between the brackets.

### Wait/Detach

1. Wait: 
   1. `join()` exactly once
   2. `joinable()` to check if thread is done
   3. `try { ... } catch(...) { t.join(); throw; } t.join()` make sure error thread joined!
   4. Always make sure even when early end parent thread, the child threads are always joined
2. Detach: 
   1. Lets a child thread run independently of parent thread's execution
   2. **WARNING** Child thread may discard variables from parent thread when ending before parent thread done (RAII)
   3. `joinable()` is false after calling detach.

Wait

Detach

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