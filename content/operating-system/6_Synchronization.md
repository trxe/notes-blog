---
layout: default
usemathjax: true
permalink: /os/ch6
header-includes:
  - \usepackage{hyperref}
  - \hypersetup{colorlinks=true,
            linkcolor=blue,
            linkbordercolor=red,
            pdfborderstyle={/S/U/W 1}}
---

# 6 -- Synchronization

## Concurrent Execution

- Interleaving execution $\require{color}$
- Sharing modifiable resource
  - P1 load, P2 load, P1 operate, P2 operate, P1 store, P2 store
  - P1 load ... **P1 store -> P2 load** ... P2 store

Examples of **unsynchronized access** that may lead to non-deterministic outcome (based on order of access/modification, aka **race conditions**.)

### Solving race conditions

**Critical section**: does critical **work** accessing shared modifiable resources. At any point only **one program** can execute in that critical section.

### Implementing Critical Sections

- **Mutual exclusion:** If $P_i$ in CS, all other processes prevented.
- **Progress:** If no process in CS, allow 1 waiting process to enter
- **Bounded wait:** No process is starved (upper bound on number of times other processes can access CS before any process $P_i$)
- **Independence:** Process not in CS should never block other processes (in CS or not).

## Incorrect Synchronization

- **Deadlock:** No progress, all processes blocked.
- **Livelock:** Usually due to deadlock avoidance. Processes jump between states avoiding deadlock, but not making progress
- **Starvation:** Some processes blocked forever

## Implementations

### Assembly level

`TestAndSet( Register, MemLocation )`

- Loads `*MemLoc` into `$Reg`
- Stores 1 into `*MemLoc`
- Returns `$Reg` contents
- Is a **single/atomic** machine operation, so there won't be sync issues.

```
void EnterCS (int *lock) {
	/* i.e. while the CS process has set *MemLoc to 1,
	 * other processes trying to set access is stuck in while loop
     * as TestAndSet returns 1. */
	while (TestAndSet( lock ) == 1);
}

void ExitCS (int *lock) {
	*lock = 0;
}
```

**Problems**: busy waiting wastes processing power.

**Solution**: variants of the instructions.

**Variants**: Compare and Exchange, Atomic Swap, Load Link/Store Conditional

### High-level Language

1. Simulate "Test and Set", but **non-atomic**. We can try preventing interleaving by **disabling interrupts**, but

![High level Test-and-Set](/notes-blog/assets/img/os/testandset.png)

- If multiprocessor, unintended concurrent access can happen anyway
- Otherwise, no other process (including the OS) can interfere you e.g. if you have an infinite loop
- Normal programs don't have enough privilege to disable interrupt.

2. `Turn` array. While `Turn != P.turn`, busy wait.

![Take **turns** to enter critical section.](/notes-blog/assets/img/os/turn.png)

- Starvation: If P0 is never executed, neither will P1.
- No independence/progress

3. `Want` array. Busy wait while someone else wants the turn.

![Wait until no-one else **wants** the turn](/notes-blog/assets/img/os/want.png)

- Stuck in infinite loop: Deadlock!

4. Combine `Turn` and `Want`: **Peterson's algorithm**

- Note that `Turn` cannot be 1 and 0 at once, so only one of the `while` loops ends at first
- Note that `Turn` now indicates which process's turn is it.

![Peterson's Algorithm](/notes-blog/assets/img/os/peterson.png)

We assume that line 2 (writing to `Turn` in each process) is atomic.

### High-level Abstraction

#### Semaphores:

Generalized sync mechanism.

- Block processes (become **sleeping processes**)
- Wake up sleeping processes

Contains one `int` representing capacity,

- `wait()` blocks if capacity $\leq 0$, otherwise decrement.
  - successful if decrement
- `signal()` increments capacity, wakes up 1 sleeping process.
  - always successful

Invariant for semaphore $s$:


$$
s_\text{curr} = s_\text{init} + \# \text{signals executed}(s) - \# \text{waits completed}(s)
$$

#### Mutex:

Binary semaphores with capacity of 1. 

Number of processes in critical section: $N_{CS} = \text{waits completed}(s) - \text{signals executed}(s)$​​


$$
s_\text{curr} = \textcolor{blue}{1 - N_{CS}} \geq 0 \Rightarrow \textcolor{red}{N_{CS} \leq 1}
$$



No **deadlock** where $N_{CS} = s_\text{curr} = 0$​​​, granted correct usage.

No **starvation** if fair scheduling.

#### Conditional variable:

How to wait for another thread to do something specific?

- `wait(condition)` waits for `condition` to become true i.e. signaled
- `signal(condition)` wakes up one sleeping process
- `broadcast(condition)` wakes up all

*Note:* after a thread is woken up, the condition may not hold anymore.

#### Monitor:

A collection of procedures manipulating a shared data structure, one lock must be held when accessing the shared data.

## Synchronization patterns

From [The Little Book of Semaphores](https://greenteapress.com/wp/semaphores/).

### Rendezvous

![How to ensure `a1` before `b2`, `b1` before `a2`](/notes-blog/assets/img/os/rendezvous.png)

```
# Thread A
semA = 0
statement a1
signal(semA)
wait(semB)
statement a2

# Thread B
semB = 0
statement b1
signal(semB)
wait(semA)
statement b2
```

### Barrier

```
rendezvous // all n-1 threads block until nth thread reaches rendezvous
critical point
```

$n$ threads. No thread executes critical point until all have reached rendezvous.

```
wait(mutex)
	waitingThreads += 1
	if (waitingThreads == n) signal(barrier) -- (1)
signal(mutex)

wait(barrier)
signal(barrier) -- (2)

critical point
```

(1) Triggers first waiting thread to pass through. Now value of barrier is $-(n-2)$​​.

(2) You **must** `signal(barrier)` after the each thread is done waiting for the barrier, otherwise the next in queue will not be able to execute (deadlock). This pattern of `wait(b), signal(b)` is also called a **turnstile**.

### Reusable barrier

How to lock the barrier again after all the threads have passed?

```
wait(mutex)
	waitingThreads += 1
	if (waitingThreads == n) 
		wait(t2) -- wait for nth thread to unlock 2nd turnstile
		signal(t1) -- signals that the 1st turnstile is unlocked
signal(mutex)

wait(t1)
signal(t1)

critical point

wait(mutex)
	waitingThreads -= 1
	if (waitingThreads == 0) 
		wait(t1) -- wait for nth thread to unlock 1st turnstile
		signal(t2) -- signals that the 2nd turnstile is unlocked
signal(mutex)

wait(t2)
signal(t2)
```

Two-phase barrier. Note we cannot solve this problem with a single turnstile, e.g. 

![Wrong reusable barrier](/notes-blog/assets/img/os/wrongreusablebarrier.png)

If the $n$​th thread passes through the CP, decrements count without interruption and then returns to the rendezvous (in a loop), and increments count again, it restores the count to $n$ and thus gets ahead of the other threads.

## Classical synchronization problems

### Producers-consumers

Shared bounded buffer of size $K$​. Constraints are:

- Producers insert products into buffer only when $n < K$ items. 
- Consumers remove products from buffer only when $n > 0$ items.

```
# Producers: notFull initialized to K
wait(notFull) -- (1)
	wait(mutex)
        buffer[in] = item
        in = (in+1) % K
	signal(mutex)
signal(notEmpty)

# Consumers: notEmpty initialized to 0
wait(notEmpty) -- (1)
	wait(mutex)
        item = buffer[out]
        out = (out-1) % K
	signal(mutex)
signal(notFull)
use item	
```

(1) Note this is a rendezvous. Removes the need to check the item count in that buffer.

### Readers-writers

Example of **categorical mutual exclusion**. Readers don't exclude other threads, but writers do. Constraints:

- Any number of readers can be present in the critical section concurrently.
- A writer in the critical section must always be alone.

```
# Writers: notEmpty initialized to 1
wait(openToWriters)
	/* prioritization 
	 * wait(noWriters)
	 * 	write
	 * signal(noWriters)
	 */
signal(openToWriters)

# Readers
wait(mutex)
	readerCount += 1
	if (readerCount == 1) 
		wait(openToWriters) -- (1)
signal(mutex)
read
/* prioritization: 
 * signal(noWriters)
 */
wait(mutex)
	readerCount -= 1
	if (readerCount == 0) signal(openToWriters)
return(mutex)
signal
```

(1) Prior to this step, if `readerCount == 1` then wait until all writers gone (`openToWriters`)

- Value of `openToWriters` is the same immediately before and immediately after writer
- Value of `openToWriters` is only increased by `readerCount == 0`. 
- Grabbing the `openToWriters` lock closes it off to writers (since it a [mutex](.#mutex:)).

**Drawbacks**: starvation of writer possible.

**Solution**: Priority to writers. (see `/* prioritization */` snippets)

### Dining philosophers

| Image                                                        | Specification                                                |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <img src="/notes-blog/assets/img/os/diningphilosophers.png"> | - 5 philosophers seated around a table.<br />- 5 single chopsticks between them<br />- To eat, one must acquire his left and right chopstick<br />- Deadlock-free and starve-free to allow philosophers to freely eat |

### Base solution

- Think -> Wait until can pickup left -> Wait until can pickup right -> Eat -> Put down left -> Put down right
  - Deadlock (all pickup left concurrently)
- Think -> **(While right not picked up) Pickup left, put down if right not free** -> Wait until can pickup right -> Eat -> Put down left -> Put down right
  - Livelock (all pickup left concurrently and put down left concurrently)

### Simple solutions

- Limiting eaters: Allow at most 4 philosophers at the table.
- One right-hander: All other philosophers are left-handers (pickup left first) except one.

Simple informal argument (for "One right-hander". Let **R** be the right hander):

1. If R grabs left: R can eat (no deadlock)
2. If R tries to grab left but it is taken: the philosopher to R's left has grabbed his right and is eating (no deadlock)
3. If R tries to grab right but it is taken: Many situations but in the worst case scenario everyone else has picked up their left chopstick except R. i.e. philosopher to R's left has his right chopstick free and can start eating (no deadlock)

### Tanenbaum solution

![Tanenbaum solution](/notes-blog/assets/img/os/tanenbaum.png)

| Take chopsticks                                              | Safe to eat                                                  | Put chopsticks                                               |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| <img src="/notes-blog/assets/img/os/takechop.png" width="100%"> | <img src="/notes-blog/assets/img/os/safetoeat.png" width="100%"> | <img src="/notes-blog/assets/img/os/putchop.png" width="100%"> |

