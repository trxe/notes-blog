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
  - platform universal
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

At runtime, the main thread runs `main()`.  `std::thread` is used to add more threads.

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
  // Thread as function ptr
  std::thread t1(hello);

  // Thread as callable object
  CallableHello ch;
  std::thread t2(ch);
  t1.join(); t2.join();
}
```

1. `main()`
2. `std::thread t(funcptr)`, `t.join()` to join
   - Using a function object (make Callable by overriding `()` operator)

### Confusing C++ stuff and initialization

Note that `CallableHello()` is a **function pointer type**! 
Hence t1 is actually a function declaration because of the type.
`std::thread` is the return type 

```cpp
// function decl of return type thread
std::thread t1(CallableHello()); // FUNCTION DECLARATION
// return-type name(param-type);
std::thread t2{CallableHello()}; // THREAD INITIALIZATION
std::thread t2_alt((CallableHello())); // THREAD INITIALIZATION
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
   1. `join()` exactly once per `std::thread`
      1. Caller waits for that thread to finish
   2. `joinable()` to check if thread is done
   3. `try { ... } catch(...) { t.join(); throw; } t.join()` make sure error thread joined!
   4. Always make sure even when early end parent thread, the child threads are always joined
2. Detach: 
   1. Lets a child thread run independently of parent thread's execution
   2. **WARNING** Child thread may discard variables from parent thread when ending before parent thread done (RAII)
   3. `joinable()` is false after calling detach.
  
## Ownership in C++

**Owner**: object with a **pointer** to a memory allocation created 
by `new` (on heap) and must be `delete`d.

Every object on free store (dynamic memory, heap) has **exactly one**
owner. If multiple pointers, only 1 is the owner. 
Objects on stack do not have owners.

### Aside: Qualifiers

Aside: The `const` key word can be placed in multiple places:

```cpp
const int m1 = 100;     // m1 is an int that is constant
int const m1_alt = 100; // same as m1; m1_alt is a constant int

int meme = 42069;
const int& n1 = meme; // n1 is a reference to an int that is constant
int const& n2 = meme; // n2 is a reference to a constant int
const int* n3 = &meme; // n3 is a ptr to an int that is constant
int const* n4 = &meme; // n4 is a ptr to a constant int
int *const n5 = &meme; // n5 is a constant ptr to an int
// meme doesn't have to be constant!

meme = 5; // ok!
*n5 = 10; // ok!
// NOT ALLOWED:
// n1 = 10;  n2 = 10;
// *n3 = 10;  *n4 = 10;
```

### Passing variables

```cpp
int c = 0;
void foo(int i, std::string const& s); // must be string const&
void oops() {
  char buf[1024];
  std::thread t(foo, 3, buf); // WILL NOT COMPILE
  // unless the variable is wrapped e.g.
  // std::ref(buf), buf will be COPIED and CASTED to t
  assert c == 0;
  t.detach();
}
void pass_by_ref_ok() {
  char buf[1024];
  std::thread t(foo, 3, std::ref(buf));
  assert c == 1; // REFERENCE modified
  t.detach();
}
void pass_by_val_ok() {
  char buf[1024];
  std::thread t(foo, 3, std::string(buf)); 
  // buf is casted to string first before being passed by val as param
  assert c == 0;
  t.detach();
}
```

- To pass function vars in, add the args after the function/function object/lambda's name.
- `oops` will never compile as main may detach t before the `buf` is converted to `std::string`.
  - This means `foo` may never run
  - Workaround: `thread t (foo, 3, std::string(buf));`

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

### Ownership of **Threads**

`std::thread` instances own resource.

Instances of `std::thread` are **movable** (change owner) 
and **not copyable**.

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

l-values: Values to the left of the assign operator.
Evaluation determines the **identity** of an object/bit-field/function.

- objects stored on the stack/heap
- members of named objects
- **anything with defined storage location**

r-values: Values to the right of the assign operator. 
Evaluation initializes an object/bit-field computed from the expression.

- literals
- temporaries
- function calls 
- variables **without fixed memory address**

C++11 has **r-value references** with declarator `&&`. 
They bind to rvalues and not to lvalues.
Distinguish r-values from l-value references with declarator`&`

#### Move semantics

Transfer of resources from **temporary objects** to l-values.

This is useful for `std::thread`, `std::unique_lock`, `std::unique_ptr` and other such classes to move their resources around
and prevent multiple copies.

See the examples below from [Microsoft](https://learn.microsoft.com/en-us/cpp/cpp/move-constructors-and-move-assignment-operators-cpp?view=msvc-170).

```cpp
// Move constructor.
MemoryBlock(MemoryBlock&& other) noexcept
   : _data(nullptr)
   , _length(0)
{
   // Copy the data pointer and its length from the
   // source object.
   _data = other._data;
   _length = other._length;

   // Release the data pointer from the source object so that
   // the destructor does not free the memory multiple times.
   other._data = nullptr;
   other._length = 0;
}

// Move assignment operator.
MemoryBlock& operator=(MemoryBlock&& other) noexcept
{
   if (this != &other)
   {
      // Free the existing resource.
      delete[] _data;

      // Copy the data pointer and its length from the
      // source object.
      _data = other._data;
      _length = other._length;

      // Release the data pointer from the source object so that
      // the destructor does not free the memory multiple times.
      other._data = nullptr;
      other._length = 0;
   }
   return *this;
}
```

## Lock free programming

Race condition: a semantic flaw occuring when **timing** of events affects **correctness**.

Data race: Undefined behaviour caused by two memory accesses where

- both target same location
- both concurrent by two threads
- not reads
- not sync ops.

Wrap data structure in a **protection mechanism**.

### `std::mutex`

- The standard guarantees
  -  prior `unlock` on the same mutex synchronize with the next `lock`. 
- The OS guarantees
  - `futex()` linux syscall
  - `futex`'s release-acquire semantics ensures synchronises-with.

### Locks in cpp

- `lock_guard`: locks a single mutex and unlocks at end of lifetime
  - No `lock()` and `unlock()` methods
- `scoped_lock`
  - guards against deadlock when locking/unlocking in the wrong order with **deadlock avoidance algorithm**
  - The order in which it locks
- `unique_lock`: doesn't always own the mutex it is associated with! it may be moved around
  - `std::defer_lock_t` tells the compile to lock later.
  - `std::try_to_lock`
  - `lock` and `unlock` the unique lock yourself (what makes unique lock more powerful than the others.)
- `lock`: locks multiple mutexes
  - ensure locking in the same order to prevent deadlocks

### Conditional Variable

```cpp
// spin-lock implementation
bool flag;  // variable to check for
mutex m;
void wait_for_flag() {
  std::unique_lock<std::mutex> lk(m);
  while (!flag) {
    lk.unlock(); // release to others
    wait(1000ms);
    lk.lock(); // obtain mutex again
  }
}
```

The above is a form of busy-waiting, which if run for too long may
prevent other threads from running. Instead we can use the 
synchronization primitive **conditional variable**.

A cond_var is associated with a **condition** to be met.

- $\ge$ 1 threads can **wait** on this condition to be satisfied.
- On satisfication, **notify** one of the threads waiting on the cond_var.

### Behaviour

1. `cv.wait(lk, []{ return conditional; })` is called by a thread.
   1. make sure the Predicate passed in does not have side effects.
   2. checks Predicate $x$ number of times with mutex acquired
    1. Release mutex each time
    2. Spurious wake: OS randomly wakes up this thread to check
   3. returns when Predicate returns `true`
2. `cv.notify_one()` or `cv.notify_all()` to unblock one or all waiting threads.

**Spurious wakes** and **Notify** both must be supported.

```cpp
// ...
std::conditional_variable data_cond;
void thread1() {
  while (true) {
    std::unique_lock lk(mut);
    // The below is an example of a monitor implementation
    // while (!data_queue.empty()) { data_cond.wait(lk); }
    data_cond.wait(lk, []{ return !data_queue.empty(); });
    // Crit. S.
    lk.unlock();
    // DO OTHER STUFF
  }
}
```
