---
layout: default
usemathjax: true
permalink: /os/ch4
---

# 4 -- Inter-process Communication

## Shared-Memory in *nix

- Process $P_1$ creates a shared memory region $M$ (OS involved)
- Process $P_1$ and $P_2$ attach $M$ to its memory space (OS involved)
- $P_1, P_2$​ can now communicate via $M$ (OS **not** involved)
- Read and write in $M$
- Detach $M$​ from memory space
- Destroy $M$ (only one process has to do, $M$ **must be detached** from all processes)

Master program creates SHM.

| Master program (creates SHM)               | function                                                    |
| ------------------------------------------ | ----------------------------------------------------------- |
| `shmid = shmget(IPC_PRIVATE, size, flags)` | creating shared memory region                               |
| `shm = (int*) shmat(shmid, NULL, 0)`       | attach shared memory to this process                        |
| `while (shm[0] == 0) {...}`                | `shm[0]` equals 1 if ready, 0 if not.                       |
| `shmdt( (char*) shm )`                     | detaches memory                                             |
| `shmctl( shmid, IPC_RMID, 0 )`             | `IPC_RMID` marks for destroying after last process detached |

Slave program attaches to SHM.

| Slave program         | function                            |
| --------------------- | ----------------------------------- |
| `scanf("%d", &shmid)` | `shmid` passed from master to slave |
| `shm = shmat(...)`    | attach                              |
| `shm[0] = 1`          | informs master of completion        |
| `shmdt((char*) shm)`  | detach                              |

#### Pros

- Efficient (space is obvious, time because OS is not involved.)
- Ease of use (shared memory region behaves the same as normal memory space.)

#### Cons

- Synchronization of access (shared resource)
- Harder to implement usually.

## Passing messages

- $P_1$​ passes message to $P_2$​ to communicate (*OVERSIMPLIFIED!*)
  - the message has to be in **kernel memory space**.
  - send and receive goes through OS.

### Direct communication

- explicit naming the other party e.g. `send(P2, msg)` and `receive(P1, msg)`
- needs to know the identity of the other party.
- Unix domain socket

### Indirect communication

- sent to a mailbox/port shared across multiple processes.
- `send(MB, msg)` and `receive(MB, msg)`

### Synchronization Behaviours

Q: What if you're trying to receive a message but the sender hasn't sent yet?

A: 2 methods of synchronization:

- Blocking primitives
  - Receiver is blocked until msg arrives
- Non-blocking primitives
  - If message is available, receives message. Otherwise receives indication that message isn't ready.

Sender doesn't block unless buffer is full. Unless the sender requires an acknowledgement.

#### Pros

- Portable (easily implemented on different Processing Environments)
- Easier synchronization (whenever synchronous primitive is used)

#### Cons

- inefficient (requires OS intervention)
- extra copying

## Unix Pipes `|`

Connects `stdout` of Process A to the WRITE end of the of the pipe, which passes on to the READ end of the pipe to `stdin` of Process B. Note it is **FIFO**.

Is a **circular bounded byte buffer**. Writers wait when buffer is full, readers wait when buffer is empty.

Its syscall in C is `pipe(args[2])`, where `args[0]` is the READ end and `args[1]` is the WRITE end.

```C
// parent
close(pipeFd[READ_END]);
write(pipeFd[WRITE_END], str, strlen(str) + 1);
close(pipeFd[WRITE_END]);
//child
close(pipeFd[WRITE_END]);
read(pipeFd[READ_END], buf, sizeof(buf));
close(pipeFd[READ_END]);
```

Possible to change/attach the `std__` communication channels to any one of the pipe ends.

## Unix Signal

- Asynchronous notification regarding some event
  - Recipient does signal **handling** via a default set of handlers OR supplied handler
- Sent to some process/thread
- `void sigint_handler(int signum)` must be defined in your program to replace the default signal handling. (requires `signal.h`)
- `if (signal(SIGINT, sigkill_handler) == NULL)` the `sigkill` handler cannot be registered (cannot replace the default signal kill handling)

