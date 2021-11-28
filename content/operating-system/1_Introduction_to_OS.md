---
layout: default
usemathjax: true
permalink: /os/ch1
---

# 1 - Introduction to OS

## What is an OS

**Definition.** An OS is a program that is an intermediary between a user and computer hardware.

## Types of OS

| OS Type          | Usage                                                        | Advantages                                                   | Disadvantages                                                |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| No OS            | Changing physical config                                     | - minimal overhead                                           | **not portable**, not efficient use                          |
| Batch OS         | (Mainframes) similar jobs are bundled with instructions that OS execs in order. | - computer does more work                                    | - **no protection** <br />- infinite loops <br />- hard to debug <br />- **no concurrency** |
| Time-sharing OS  | Multiple users interact with machine via **terminal**. job scheduling mimics concurrency. | - memory management<br />- shares CPU time, memory, storage.<br />-  **Virtualization** |                                                              |
| PC OS -- Windows | Single user at once, but multiple user access                | (dedicated machine)                                          |                                                              |
| PC OS -- Unix    | One user at workstation, but others can remote access.       | (general time sharing model)                                 |                                                              |

## Motivations for OS

**OS is an abstraction.** 

- Hardware components have same core functionality but different attributes (speed, storage etc.)
- Virtualization: each program executes as if it has all the resources to itself (e.g. memory space, files as contiguous entities when they are actually disjoint, illusion of process always running)

**OS is a resource allocator.** Manage resources and arbitrate conflicting requests.

- Concurrent file usage/memory access
- Process switching to optimize CPU and IO activities

**OS is a control program.** Prevents accidents and malicious attacks, provides security isolation and protection.

## Implementation -- High Level View

**Kernel**: complete access to hardware resources; **User**: partial access.

- **User space**: User application, User interface
- **Kernel space**: Operating system

Kernel: Deals with hardware issues, provides **system call interface**, special code for **interrupt handlers**, device **drivers**.

## Implementation -- Components

![In C/C++ system calls can be made via the library version (D --> C --> A)](/notes-blog/assets/img/os/systemcomms.png)

- A: OS exec machine instructions
- B: Normal exec machine instructions
- C: System calls (interface)
  - D: Program calls library code
  - In C/C++ system calls can be made via the **library** version (D -> C -> A)

## Implementation

Code organization:

- Machine independent HLL
- Machine dependent HLL
- Machine dependent assembly code

OS Structures:

|                                    | Monolithic                                      | Microkernel                           |
| ---------------------------------- | ----------------------------------------------- | ------------------------------------- |
| Size                               | Bigger                                          | Smaller                               |
| Execution                          | Fast                                            | Slow                                  |
| Extensibility                      | Highly coupled, difficult to extend             | Ease of extensibility                 |
| Security (e.g. a service crashing) | Highly coupled, may cause whole system to crash | Causes only that microkernel to crash |
| Examples                           | Unix/Win/BSD                                    | QNX, Symbian, L4Linux                 |

## Virtual Machines

OS assumes total control of hardware. So what is we want to run multiple OS on the same hardware? (e.g. Cloud computing)

Software emulation of hardware = **Virtualization** of underlying hardware (Illusion of hardware access)

**Hypervisor**/VMM

- Type 1: Runs directly on hardware.
  - Provides individual virtual machines to multiple guest OSes
- Type 2: Runs as a program on a Host OS