---
layout: default
usemathjax: true
title: Testing and Debugging Concurrent Programs
permalink: /pcp/ch4
---

# Concurrency related bugs

1. Unwanted blocking [Deadlock, Livelock, I/O]
2. Race conditions

# Memory Debugging tools

Memory tools keep track of the state of the memory.
Computes **happens-before** to find the data races. 
(2 accesses without happens-before accessing same memory location, one of them being a write).

## Valgrind

Tool for memory debugging, memory leak detection, profiling

- Dynamic instrumentation 
  - Heavyweight binary instrumentation (x20)
  - but realtime
  - **Memory shadow**

### Memcheck: Memory shadow

For every byte of memory location:

1. A bit (is the byte accessible)
2. V bit (is byte initialized)

Detect and report incorrect memory access:
- uninitialized memory
- use after free
- out of bound access
- memory leak

![Valgrind Shadow Memory](/notes-blog/assets/img/pcp/valgrind-shadow-mem.png)

### Helgrind

Detect data race and race conditions

- misuses of `pthread` API by **intercepting** POSIX functions.
- Potential deadlocks from lock-ordering
- data race
  - build a DAG of happens-before relationships between mem accesses
  - when new lock acquired, add an edge
  - if cycle is formed, deadlock is detected.

# Sanitizers

Examples:
- AddressSanitizer
  - detects address issues (use after free/scope, overflow)
- LeakSanitizer
  - detects memory leak
- ThreadSanitizer
  - detects data races, thread leaks etc.
- MemorySanitizer
- HWASan (Hardware Assisted Address Sanitizer)
- UBSan (Undefined Behaviour Sanitizer)



### ASan: Address Sanitizer

- Necessary prereq for TSan, as memory needs to be checked for data races, or accessed out of lifetime etc.

### TSan: Thread Sanitizer

Replaces your library calls with code that

- checks that ops on shared data are done with lock
- record lock sequence 
- give some threads priority

## Race conditions

Data races: **Undefined behaviour** due to concurrent memory access by at least two threads, at least one of which is a write,
of the same memory location.

Broken invariants:

- Dangling ptrs: thread2 deletes memory that thread1 is accessing
- Random memory corruption: partial updates
- Double free: 2 threads popping the same value off a queue

Lifetime issues:

- thread outliving the accessed data
  - Always check and call `join()`
  - e.g. thread referencing local vars that go out of scope before thread function is done.

## Methods

Eliminate the concurrency

Run on a single core

How many threads can my program run for optimal performance?

Processor architectures:
x86 runs on TSO, so rel-acq may work better than expected lol

Library calls, operators may NOT be thread safe!

# Tutorial: Example with `shared_ptr`

Lock version:

```cpp
template <typename T>
class shared_ptr {
private:
    std::mutex* mut;
    T* val;
    int* rc;
public:
    shared_ptr(T _val) : val(new T(_val)), rc(new int(1)), mut(new std::mutex) {}

    shared_ptr(const shared_ptr<T>& copy) :
        mut(copy.mut), val(copy.val), rc(copy.rc)
     {
        std::lock_guard lock(*mut);
        *rc += 1;
    }

    shared_ptr& operator=(const shared_ptr<T>& copy) {
        std::lock_guard lock(*copy.mut);
        mut = copy.mut;
        val = copy.val;
        rc = copy.rc;
        *rc += 1;
        return this;
    }

    ~shared_ptr() {
        std::lock_guard lock(*mut);
        *rc -= 1;
        if (*rc == 0) {
            delete val;
            delete rc;
            delete mut;
            std::cerr << "terminated!" << std::endl;
        }
    }

    T* get() { return val; }

    void set(T _val) {
        std::lock_guard lock(*mut);
        *val = _val;
    }

    const T* get() const { return val; }

    /*
    int use_count() {
        return rc->load();
    }
    */
};
```

Atomic rc version:
```cpp
template <typename T>
class shared_ptr {
private:
    T* val;
    std::atomic<int>* rc = nullptr;
public:
    shared_ptr(T _val) : rc(new std::atomic<int>), val (new T(_val)) {}

    shared_ptr(const shared_ptr<T>& copy) :
        val(copy.val), rc(copy.rc) {
        rc->fetch_add(1);
    }

    shared_ptr& operator=(const shared_ptr<T>& copy) {
        val = copy.val;
        rc = copy.rc;
        rc->fetch_add(1);
        return this;
    }

    ~shared_ptr() {
        if (rc->fetch_sub(1) == 0) {
            delete val;
            delete rc;
            std::cerr << "terminated!" << std::endl;
        }
    }

    T* get() { return val; }

    const T* get() const { return val; }
};
```