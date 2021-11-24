---
layout: default
usemathjax: true
permalink: /os/ch2
---

# 2 -- Process Abstraction

## Summary

1. How to represent a process in memory.
2. Separation of user space and kernel space to provide safety and ease of use
3. Communications between user space, kernel space and hardware

## Hardware components

CPU houses:

- Fetch unit: loads instruction (Location in Program Counter register)
- Functional units: for instruction execution
- Other registers: GPR, Special registers (PC, Stack Pointer, Frame pointer)

## Basic Instruction Execution

1. Load process instructions into memory (Text)
2. Set PC to Text start address
3. Instruction exec loop: load memory, process via Functional Units, store in memory

## Function Calls

Problem of where to allocate parameters, local variables, return values in memory:

- control flow: after a callee function's return statement, how do we know location of instruction to return to of caller function?
- data: no prior knowledge of how much space to pre-allocate for local variables.

### Stack Memory for function invocations

![Memory Stack](/notes-blog/assets/img/os/memstack.png)

Stack frame: memory region for a single function's invocation information. **Pushed** onto a stack at stack pointer (SP) address.

After each return, we **pop** the returned frame from the stack. Unlike other memory regions, stack memory can grow upwards. 

### Stack Pointer, Frame Pointer

SP @ top of the stack. Changes during stack frame lifetime.

FP @ fixed location in **each stack frame**. e.g. to access some value: `lw (FP - x)` Needs to be saved as well to return to after a frame is popped off the stack.

The existence of the FP and SP is architecture/platform dependent. A function invocation can be done purely with SP.

### Function call convention

- Prepare for a call
  - **Caller:** Pass parameters using registers and/or stack
  - **Caller:** Return PC saved on stack
- Transfer of control (caller --> callee)
  - **Callee:** Save the old stack pointer (SP), old frame pointer (FP)
  - **Callee:** Allocate space for local variables
  - **Callee:** Move SP to new stack top, FP to new location in frame.

### Stack frame teardown

- After return
  - **Callee:** Result into register
  - **Callee:** Restore saved SP, FP
- Transfer of control (callee --> caller)
- Execution continued.

### Saved Registers

What if we have more variables than registers? **Register spilling** allows saving some variables to memory and fetching them when needed. Saved in stack frame.

e.g. Functions can spill registers it intends to use locally before the function starts. After returning function, restore those registers from memory.

## Dynamic memory allocation

i.e. Acquired memory space during execution time. Can we use "Data" or "Stack" memory?

- *Size not known* during compilation (NOT DATA)
- No *de-alloc timing*, can be freed by garbage collecting (NOT STACK)
- Thus we need to manage the **HEAP**.
  - Variable size, flexible allocation/deallocation timing.

## Process Management

Allowing multiple programs A, B to share hardware usage by switching between them.

### Abstraction of program (Process)

Information required to describe a running program:

- **Memory** context:
  - Text, Data, Stack, Heap [page table](/notes-blog/os/ch9) **addresses, limits**
- **Hardware** context:
  - Registers, PC, SP, FP
- **OS** context:
  - PID, parent, child, open file descriptor list

### Process ID (PID) and State

- PIDs are reusable as long as no other running process shares this ID.
- max processes = max unique PID count
- reserved PIDs (kernel)

A process can also be ready to run in the **ready queue**, but not actually executing.

### Process state model

| General Process Model                                        | Unix Process Model                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <img src="/notes-blog/assets/img/os/procmodel.png" width="100%"> | <img src="/notes-blog/assets/img/os/unixprocmodel.png" width="100%"> |

## OS Communication

### System Calls

OS's API: User space callable functions.

- Well-defined, safe implementation of system operations
- Prevent from directly manipulating hardware devices (safety)
- Request for services via **interrupts**

Changes from user to (privileged) kernel mode.

In C/C++ the system call can almost be invoked **directly** (library version of the call with same name & params, acts as ***function wrapper***). Other than that the **library functions** acts as a ***function adapter***.

### Mechanism

1. Program invokes library call
2. Library call puts system call number in e.g. register
3. **TRAP** instruction to switch from user to kernel mode.
4. System call handler is determined.
   1. Save CPU context
   2. System call number as index
   3. Handled by **dispatcher**
5. System call handler executed
6. System call handler ends
   1. Restore CPU context
   2. Return library call
   3. Kernel to user mode.
7. Library call returns to program.

### Process Control Block/Process Table

The entire execution context of a process. Kernel maintains all processes' PCB.

- Scalability: How many process can you run concurrently at max?
- Efficiency: How to provide efficient access with minimum space wasted?

![PCBs contain metadata about these contexts, but not the memory region or hardware itself](../assets/img/os/pcb.png)

## Exceptions and errors

- Synchronous: Happens every time instruction executes
- Asynchronous: Caused by external event (I/O, Timer)

### Exception

**Synchronous**, happens at runtime due to program execution. An exception handler is then executed. `echo $?` prints the exit status of the last command.

- Segfault: e.g. trying to change a character of a constant string `char*`, or dereferencing a `NULL` location
- Arithmetic errors: Divide by 0

### Interrupt

Electronic alerting signals sent to the processor from hardware.

**Asynchronous**, external events can interrupt the execution of a program. An interrupt handler is then executed. Happens independent of program execution, due to I/O etc.

