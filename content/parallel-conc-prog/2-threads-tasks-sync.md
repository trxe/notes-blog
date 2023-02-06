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
  
### Passing variables

```cpp
void foo(int i, std::string const& s);
void oops() {
  char buf[1024];
  std::thread t(foo, 3, buf);
  assert c == 0;
  t.detach();
}
```

- To pass function vars in, add the args after the function/function object/lambda's name.
- The above example may detach t before the `buf` is converted to `std::string`.
  - This means `foo` may never run
  - Workaround: `thread t (foo, 3, std::string(buf));`
- In fact, `buf` cannot be used at all by the thread t running `foo`!
  - As such, our workaround here is **actually a pass by value**.
  - So is the below

```cpp
void bar(int& change) { change++; }

void pass_by_val() {
  int c = 0;
  std::thread t(bar, change);
  t.join();
  assert(c == 0);
}

void pass_by_ref() {
  int c = 0;
  std::thread t(bar, std::ref(change));
  t.join();
  assert(c == 1);
}
```

## RAII: Resource Acquisition is Initialization

enter scope = allocate and initialize (on heap if necessary).

exit scope = deallocate and destroy (from heap if necessary).

### Lifetimes

Start-of-life begins **after**

1. Storage with alignment **obtained**
2. **Initialization** completed

End-of-life

1. Non-class type: object is destroyed @ end-of-life
2. Class type: destructor call invoked
3. Either: storage occupied is **released** or reused by another obj outside

### Ownership

`std::thread` instances own resource.

Instances of `std::thread` are **movable** and **not copyable**.

Transfer of ownership can be done with the idioms:

```cpp
// TRANSFER OUT, no parameters
std::thread f() {
  void func();
  return std::thread(func);
}

// TRANSFER OUT, with parameters
std::thread g() {
  void func(int);
  return std::thread(func, 42);
}

void moveHere(std::thread t);

// TRANSFER IN
void g() {
  void func();
  moveHere(std::thread(func));
  std::thread t(func);
  moveHere(std::move(t));
}
```

### r-values and l-values

l-values: Values to the left of the assign operator

- objects stored on the stack/heap
- members of named objects
- **anything with defined storage location**

r-values: Values to the right of the assign operator 

- literals
- temporaries
- variables without fixed memory address

C++11 has **r-value references** `&&`. They bind to rvalues and not to lvalues.

They are useful for performing **move operations**: moving values from the rvalue which will be destroyed anyway,
to the lvalues in the method being called.

This is useful for `std::thread`, `std::unique_lock`, `std::unique_ptr` and other such classes to move their resources around
and prevent multiple copies.

## Lock free programming

Race condition: a flaw occuring when **timing** of events affects **correctness**.

Data race: two memory accesses where

- both target same location
- both concurrent by two threads
- not reads
- not sync ops.

Wrap data structure in a **protection mechanism**.

### `std::mutex`

- The standard guarantees
  -  prior `unlock` on the samemutex synchronize with `lock`. 
- The OS guarantees
  - `futex()` linux syscall
  - `futex`'s release-acquire semantics ensures synchronises-with.

### Locks in cpp

- `lock_guard`: locks a single mutex and unlocks at end of lifetime
- `scoped_lock`
  - guards against deadlock when locking/unlocking in the wrong order with **deadlock avoidance algorithm**
  - The order in which it locks
- `unique_lock`: doesn't always own the mutex it is associated with! it may be moved around
  - `std::defer_lock_t` tells the compile to lock later.
  - `std::try_to_lock`
  - `unlock` the unique lock yourself (what makes unique lock more powerful than the others.)
- `lock`: locks multiple mutexes
  - ensure locking in the same order to prevent deadlocks

### Conditional Variable

A cond_var is associated with a **condition** to be met.

- $\ge$ 1 threads can **wait** on this condition to be satisfied.
- On satisfication, **notify** one of the threads waiting on the cond_var.

With cond_vars you can create **monitors**.

Allows threads to 
1. have mutex
2. block **until a certain condition turns false**.

Monitors must have 

1. Lock object (mutex)
2. Condition variable
3. A condition

```cpp
// ...
std::conditional_variable data_cond;
void thread1() {
  while (true) {
    std::unique_lock lk(mut);
    // ALT: while (!data_queue.empty()) { data_cond.wait(lk); }
    data_cond.wait(lk, []{ return !data_queue.empty(); });
    // PROCESS!
    lk.unlock();
    // DO OTHER STUFF
  }
}
```
if condition is satisfied, return.

Otherwise, `wait()` **unlocks** the mutex and puts thread in waiting state.
