---
layout: default
usemathjax: true
permalink: /os/ch5
---

# 5 -- Threads

## Summary

| Process                         | Thread                                  |
| ------------------------------- | --------------------------------------- |
| Context duplication             | No need                                 |
| Difficult IPC                   | No need                                 |
| Protection (isolation)          | If one thread crashes, all threads stop |
| Simpler concurrency enforcement | Race conditions prone                   |
| Slow (context switching)        | Fast                                    |

**Thread types** also covered.

## Why Thread

- Processes expensive
  - Process creation requires duplication of memory space and context
  - Context switching requires saving and restoring
- Communications difficult
  - Independent memory space i.e. Non-trivial information passing (kernel space mode switch)

Traditionally processes have a **single thread of control**. (One instruction at any one time.)

Adding more threads of control to the same process allows multiple independent parts of the program to happen at the same time. Different threads spawned have different PCs.

### Process vs Thread

- Since all threads are shared within 1 process, they share:
  - MEM context: Text, Data, Heap
    - Stack: Conceptually not shared. Each thread has a different call stack.
    - But all threads share same memory address space (page table)
  - HW context: None (not SP, FP, PC, cos dedicated call frame! not GPR also cos wrong values.)
  - OS context: process id, files, network sockets etc.
- Process context switch vs Thread switch
  - Process: OS, HW and MEM all needs to be switched
  - Thread: Only HW (reigsters) and "Stack" which is really just changing the FP and SP, still in the same memory address
- **Pros:** Thus thread has more
  - economy
  - better resource sharing
  - responsiveness
  - scalability (multiple CPU)
- **Cons:** But problems with
  - Safety
  - Concurrency
    - Parallel library calls possible (must make sure these calls are multithread safe!)
  - Process behaviour (Implementation dependent):
    - Does `fork`ing a 2-thread process give you a 2-threaded child? (Unix: no, 1 thread in child.)
    - `exit`: does it end all threads? (Unix: yes.)
    - `exec`: does it end all other threads other than the one called? (Unix: yes.)

## Types of thread

### User thread

Handled by runtime system and implemented as a user library, not informed to kernel at all.

- Benefits: Very lightweight, no need for syscalls.
- Weakness: OS cannot put a thread on a different CPU core to maximize the CPU utlization

### Kernel thread

Thread is implemented in OS and handled via system calls. Scheduling on thread level possible, use threads for its own execution.

- Benefits: Scheduling based on threads, CPU cores max. utilized
- Weakness: Slower, more resource intensive due to syscalls. Less flexible.

### Hybrid thread

![Hybrid threading](/notes-blog/assets/img/os/hybridthread.png)

OS will schedule on the **kernel threads** whereas **user threads** bind to some kernel thread(s).

### Modern threading 

e.g. **Simultaneous multi-threading** (SMT/Hyperthreading) that supplies multiple sets of GPRs + special registers allowing threads to run natively and in parallel on the **same core**.

## POSIX threads `pthread`

Standard definition of API and behaviour of threads. However the implementation isn't specified by POSIX. So either user or kernel thread possible.

- `-lpthread` flag in `gcc` call and `#include <pthread.h>` necessary
- `pthread_t` is the datatype of a thread id.
- `pthread_attr_t` is the datatype of thread attributes.
- Create and terminate via the following:

```C
int pthread_create(
	pthread_t *tid_created,
	const pthread_attr_t *thread_attrs,
	void* (*callFunction) (void*)
	void *arg_for_startroutine
	);

int pthread_exit(void *exitValue); // for syncing
// if pthread_exit isn't used, 
// NO WAY to return an exit value and thread terminates
// once startroutine is done.
```

- Synchronization via `pthread_join()` (block until this other thread is complete.)

```c
int pthread_join(pthread_t tid, void **status) 
    // returns 0 if succeed, !0 if error.
    // status == exit value, tid is the pthread to wait for
```

