---
layout: default
usemathjax: true
permalink: /os/ch7
---

# 7 - Memory Management

## Memory Hardware

Physical memory storage (RAM) is a contiguous **byte-addressed** array, each address being a **physical address**.

### Memory Hierarchy

| Name             | Speed   | Typical Amount | Cost       |
| ---------------- | ------- | -------------- | ---------- |
| CPU registers    | 1ns     | 512B           | Expensive  |
| Cache            | 10ns    | 12MB           | $150/MB    |
| RAM              | 100ns   | 8GB            | $0.58/MB   |
| HDD              | 10ms    | 2TB            | $0.0025/MB |
| Tape drives etc. | Seconds | PBs            | Dirt cheap |

### Memory from source code to executable

1. memory address for each component is assigned by **compiler** and **linker** [includes **Code** and **Data**]
2. `exec` syscall to Load and Run the executable which has the instructions of where to load and run included

### Loading into physical memory

1. `fork` a process to
   1. obtain info about process for kernel
   2. make a new entry in PCB
   3. initialize other components
2. `execve` fills 3 regions into the [SWAP](/notes-blog/os/ch9#secondary-storage-capacity) while loading:
   1. **Text**: your code (in assembly) that is within the binary
   2. **Data**: global variables
   3. **Stack**: initial stack frame
   4. Note that **heap** is empty and not initialized!

![Memory execution](/notes-blog/assets/img/os/memory_execution.png)

### Data types

- Transient Data: Local/Stack variables
- Persistent Data: Text/Global/Heap variables
  - valid for the whole duration until **explicitly removed**

Both can grow/shrink.

## Memory Abstraction

- How to access memory location of process
- Multiple processes sharing memory
- Protection of address space

**Option 1**: Use physical memory directly.

| Pros                         | Cons                                    |
| ---------------------------- | --------------------------------------- |
| No further conversion (fast) | Hard to protect memory space            |
|                              | Processes will erroneously share memory |

**Option 2**: Translate all memory addresses at **load time**

| Pros            | Cons                                             |
| --------------- | ------------------------------------------------ |
| Is correct      | Slow                                             |
| Protects memory | Not easy to distinguish references from integers |

**Option 3**: Base + limit

| Pros                                                         | Cons                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Is correct                                                   | Slow memory access (Each memory access requires 1 **add** and 1 **comp**.) |
| Protects memory                                              |                                                              |
| Fast compile (addresses translated by *Base* at **runtime**) `Addr = $Base + LogicalAddr` |                                                              |
| *Limit* protects the memory space of processes `Addr < $Limit` |                                                              |

**Option 4**: Segmentation mechanisms

Note what options 2, 3, 4 do is provide a **mapping** from **logical** addresses to **physical** addresses, which is performed by the OS.

## Contiguous Memory Management

**Requirements:** 

1. Stored program computer (instructions in memory)
2. Load-store memory (ALU and Memory operations)

**Assumptions:**

1. Each process memory space is contiguous
2. Physical memory is large enough for the processes' complete memory space.

**Problems:**

1. **Internal fragmentation:** Allotted memory space > Required memory space
2. **External fragmentation:** Many small non-contiguous blocks not-allocable to any process

When memory is full, remove terminated processes OR [swap](/notes-blog/os/ch9#secondary-storage-capacity) blocked processes to secondary storage.

### Memory Partitioning

**Fixed-size partitions:** physical memory split into a fixed **number of partitions**. Each process occupies a partition

| Pros                                     | Cons                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| Easy to manage                           | Partition size needs to be $\max(\{p_1 \dots p_n\})$​ <br />i.e. small programs waste memory space (**internal fragmentation**) |
| No need further computations to allocate |                                                              |

**Variable-size partitions:** size allocated based on actual size of process. OS tracks occupied/free regions, splits and merges as necessary.

| Pros                      | Cons                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Better memory utilization | Many holes from termination/swapping/creation (**external fragmentation**) |
|                           | Needs to maintain more information in OS                     |
|                           | Allocation algorithm needed                                  |

### Allocation Algorithms

1. First-fit: First hole large enough
2. Best-fit: Smallest hole large enough
3. Worst-fit: Biggest hole large enough

**Split** the hole into `N` which becomes the partition and `M-N` which becomes the new hole (allocation).

Upon freeing of partitions (deallocation):

- **Merge** with adjacent holes
- **Compaction** to move occupied partitions together to consolidate holes (expensive!)

| Merging                                           | Compaction                                              |
| ------------------------------------------------- | ------------------------------------------------------- |
| ![Merging](/notes-blog/assets/img/os/merging.png) | ![Compaction](/notes-blog/assets/img/os/compaction.png) |

Note allocation is $O(n)$​ but deallocation is $O(1)$​ **assuming** that the address of the memory block the process occupies is saved somewhere.

### Bitmap representation

OS represents each equal memory partition with a bit.

Incurs overhead for bit manipulation operations because the memory is not bit-addressable.

### Linked list representation

OS represents the partitions as a linked list. In dynamic partitioning above, to allocate a new process:

1. Find the first available partition that can accommodate the process
2. Split the partition into the size of the process and the remaining partition space left
3. Make the available space the next node in the linked list.
4. Allocate the space found.

To de-allocate a process, check the partitions before and after if they can be merged.

### Buddy system

Free blocks recursively **split in half** to meet request. Each pair of neighbouring blocks become **buddy blocks**. 

![Buddy blocks](/notes-blog/assets/img/os/buddyblocks.png)

Once both are freed, can be merged to become larger block.

![Allocation is now O(1) and deallocation O(n).](/notes-blog/assets/img/os/buddysystem.png)

**Allocation** of process of memory block size $n$

1. Check if any free block of size $2^{\lceil \log n \rceil}$ exists at $A[\lceil \log n \rceil]$
2. If not, look upwards for bigger blocks to split.

At most $\log M$ steps for fixed total physical memory size $M$: $O(1)$

**Deallocation**: Check if any free block in $A[\lceil \log n \rceil]$ is the buddy (worst case $O(n)$ time for $\lceil \log n \rceil = 1$)

- Note that the buddy blocks of size $k = \lceil \log n \rceil$​​ are: `xx...xx000..0` and `xx...xx100..0` with $k$ trailing zeroes

**Internal fragmentation**: all memory requests are rounded up to the nearest power of two.

**External fragmentation**: due to limited coalescing ability of this allocator (unable to coalesce blocks of different sizes, if they are not buddy blocks)
