---
layout: default
usemathjax: true
permalink: /parallel/ch2
---

# Process

Breaking down a sequential algorithm:

1. Human programmer: **Decomposition** and **scheduling**
2. OS and libraries: Some scheduling and **mapping** of tasks on cores

A single **process**:
- One PID
  - View via `ps` or `top`
- Comprises of
  - **Text**: PC
  - **Data**: global data (OS resources)
  - register values
  - **Heap**
  - **Stack**
- Has it's own address space, not shared
- Explicit communication required across processes

### Multiprogramming/Multitasking processes

To **switch** between contexts, need to incur some **overhead**.
This is wasted time.
- `fork`: Copy on write the entire process context with new PID

Communication between processes expensive.
- Shared memory and channels for communication require system calls
- Must be handled by OS

To support multiprogramming
- Time slicing execution (context switching on the same core, pseudo-p)
- Parallel execution of processes on different cores/resources

### Unix commands

```cpp
int child_pid = fork();
if (child_pid == 0) {
    cout << "i'm in child" << endl; 
} else {
    cout << "i'm the parent with child " << child_pid << endl;
}
```

See the diagram [here](/notes-blog/os/ch2#process-state-model)

# Threads

Multiple independent control flows in **one process**

A sequential execution stream in one process, with different
- PC
- SP
- registers
- stack

No need to copy address space, only need to **copy**
- Stack
- Registers

Corresponds with shared-memory architecture, without extra setup.

In the **library**: you make **user-level** threads.
- `pthread`s are POSIX threads part of the POSIX library
- Usually mapped ONE-TO-ONE to an OS level thread.
- Mapping performed **by the library**.
- Fast switching thread context

**Kernel** threads are made by the OS
- 

```cpp
int main() {
    pthread_t thread1, thread2;
    char *message1 = "Thread 1";
    char *message2 = "Thread 2";
    int iret1, iret2;
    /* Create independent threads each of which will execute function */
    iret1 = pthread_create( &thread1, NULL, print_message_function, (void*) message1);
    iret2 = pthread_create( &thread2, NULL, print_message_function, (void*) message2);

    // NOTE: HERE ONLY 1 THREAD IS RUNNING THIS! The rest are in print_msg_fn

    /* Wait till threads are complete before main continues. Unless we */
    /* wait we run the risk of executing an exit which will terminate */
    /* the process and all threads before the threads have completed. */
    pthread_join( thread1, NULL);
    pthread_join( thread2, NULL);
    printf("Thread 1 returns: %d\n",iret1);
    printf("Thread 2 returns: %d\n",iret2);
    exit(0);
}
void *print_message_function( void *ptr ) {
    char *message;
    message = (char *) ptr;
    printf("%s \n", message);
}
```

Number of threads: Usually 2x the number of cores
At some point the overhead > time saved.
One good point: What is the inflection point?

# Synchronization

When two concurrent threads are accessing a shared variable, 
the access must be controlled to prevent wrong behaviour 
(race condition).

## Race Condition
1. 2 threads/processes **access** shared resource  without synch
2. At least 1 thread **modfies**

Create a critical section.

### Mutex (Mutual exclusion)
Creates large **atomic blocks** (cannot be interleaved).

Critical sections must have

1. Mutex [Safety]
2. Progress [Liveness]
3. Bounded waiting (no starvation) [Liveness]
4. Performance

## Mechanisms
- Lock
  - `acquire` and `release`
  - spinlock (spin)
  - mutex (block)
- Mutex
- Semaphores
  - Allow access to a fixed number of threads
  - `wait`: decrement, block till open
  - `signal`: increment, allow another thread.
  - Binary semaphore (mutex)
  - Counting semaphore
  - Maybe
  - Problems: **deadlocks**
- Monitors
- Conditional variables

## Problems

**Deadlock**: Every process is waiting
- mutex
- hold and wait (one process  holding one rsc and waiting for another)
- no pre-emption (no external abort)
- circular wait

**Starvation**: One process cannot make progress because another process has the resource.
- side effect of scheduling
- one thread beats another when lock acquisition.

**Livelock**: the 2 ppl in the corridor trying to walk past each other problem.

### Patterns (to solve classical synch problems)
- Bounded buffer
- Producer-consumer
  - e.g. System buffers
  - Finite buffer: (add another semaphore for number of spaces)
  - Infinite buffer
- Reader-writers
  - e.g. Files
- Dining philosophers
  - e.g. ??? roundabout
- Barbershop

**NEVER WAIT IN THE CRITICAL SECTION**.