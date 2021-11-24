---
layout: default
usemathjax: true
permalink: /os/ch2b
---

# 2b -- Process Abstraction in Unix

## Process initialization

- Makes a copy of the parent PCB with new PID.
  - Child process has a copy of parent's [file descriptors](/notes-blog/os/ch10) table.
- When copying memory context, creates virtual address space of child process.
  - Copy-on-Write: On **forking**, new process' virtual memory pages mapped to the **same frame locations as parent**
- Queued on the scheduler.

## The master process `init`

Fork creates the process tree from the most common ancestor (PID = 1). Continues running until shutdown.

## Process creation

Creates a new child process, a **duplicate** of the current executable image.

- Memory space is copied, not referenced.
- returns 0 if child, PID if parent.
- `clone` allows for partial copying.

## New program execution

- `execl, execlp, execle, execv, execvp, execvpe` for different parameters
- `argc`: number of command line arguments
- `argv` `char*` string array. Each element in in `argv[]` is a character string. `arg[0]` is always the program name.

`execl` replaces currently executing process image with a new one. 

```c
int main() {
	execl( "/bin/ls", "ls", "-al", NULL);
} // executes "ls -al"
```

## Parent/Child synchronization

`wait(int *status)` returns the PID of the terminated child process in the least significant byte of `*status`.  

`waitid()` suspends until the child specified by the PID argument **changes state**.

- parent blocks until **at least one** child terminates
- remainder of child system resources is cleaned up (remove all zombie process)
- does not block if no child.

Scheduling Running --> Ready: When the OS decides you are taking up too much CPU time, you will be sent back to the ready queue

## Process Termination

- returning `result` from `main()` implicitly calls `exit(result)` (open files are also flushed automatically)

### Zombie Processes

- After a child process `exit`s it immediately becomes a zombie as it still has a PCB in the process table.
- Needed for the parent to read its exit status. 
- After parent calls `wait` to read the child process' exit status, it is "reaped" and removed.
  - If parent dies before zombie reaped, zombie also becomes an [orphan](#orphan-processes).

### Orphan Processes

- parent process terminates before child.
- `init` process (which is the first process that boots in a Unix system) becomes the pseudo-parent of child.
  - `wait()` called by `init` periodically cleans up.
