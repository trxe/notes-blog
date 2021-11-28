---
layout: default
usemathjax: true
permalink: /os/ch9
---

# 9 - Virtual Memory Management

[Helpful questions for determining page table sizes etc.](https://www.cs.utexas.edu/~lorenzo/corsi/cs372/06F/hw/3sol.html)

## Motivation

Removing the assumption that physical memory must be large enough to contain $\geq 1$ processes with **complete** memory space.

### Secondary storage capacity

split logical address space into chunks (extension of paging)

- some in physical memory
- others in secondary storage

![Logical (Virtual) Memory space](/notes-blog/assets/img/os/logical_memspace.png)

### Extended paging scheme

- Still "Page Table :: Virtual -> Physical Address"
- 1 Virtual address = 1 Physical Address (may not be byte-addressed!)
  - VMS = $2^\text{virtual addr. bits} (\text{space 1 physical addr is mapped to})$
- 2 Page types (1-bit identifier)
  - Memory resident
  - Non-memory resident
- **CPU can only access memory resident pages**
  - other wise page fault
  - OS' job to move non-memory resident to physical memory

Steps:

1. (Check page table) X is memory resident ? done : trigger-
2. **Page Fault:**

- Trap to OS
- Global/Local page replacement (evicts another page)
  - Writes to the secondary storage (if needed, i.e. write bit = 1)
- Locate page X in secondary storage
- Load page X in physical memory
- Update page table
- Flush TLB (if needed, i.e. present bit = 1)
- Return from Trap

Note that the page is **never removed from secondary storage**

- save time moving back to secondary storage again if cannot be allocated any memory space

#### Thrashing:

- If memory access keeps hitting *Page Fault* and triggering expensive access operations
- Locality principles:
  - temporal: most recently used memory addresses likely to be used again
  - spatial: nearby addresses

### Summary (overall benefits)

1. Complete separation of logical from physical memory
2. More efficient use of physical memory
3. More processes allowed to reside in memory

#### Demand Paging

Scenario: a process needs $n$ pages of memory (too slow to load all into memory at once at startup).

- VM allows you to **lazily load** only the needed page  and run CPU at once

## Page Table Structures

Tradeoffs between performance and memory consumption.

Given the following constraints, the following will be an example of time and memory needed.

- Processes: 1 process **64KB** = $2^{16}$, 2 processes **2MB** = $2^{21}$
- Virtual address bits: 32
- Physical memory size: 8GB = $2^{33}$​​
- Page size: 4KB = $2^{12}$
- 3 access bits + 1 dirty bit
- Disk access time for 1 page: 5ns
- RAM access time: 30ns
- TLB access time: 1ns

### Page Table Entry size

$$
\text{PTE size} = \left\lceil \left( \frac{\text{Physical memory size}}{\text{Page size}} + \text{Protection bits} \right) \div 8 \text{bits} \right\rceil
$$


Hence PTE size is 4B here.

### Time for page hit

Next, due to the TLB, best time is always: 1ns (TLB) + 30ns (physical addr) = 31ns. The TLB has an associative memory of <**page**\#, frame\#> pairs, not page directory, so the pages can be found instantly.

### Direct paging

**Time for page fault**: 1ns (TLB) + 30ns (page table, page fault) + 5ms (write) + 5ms (locate and load from swap) $\approx$ 10ms

**Page table space**:

- Level 1 page table size: $2^{32-12} (4) = 2^{22}$​
- Number of page tables: 3 **(one per process)**
- Total: $2^{22}(3)$​​ = 12MB

### Two-level paging

- Process most likely won't use whole virtual memory space
- Split level 1 page table into $2^m$ level 2 page tables, only **some** of which are in physical memory
- Level 1 = Page directory, Level 2 = Page table.
- Page directory entry (**PDE**) needs $m$​ bits to index each page table, which needs $n$ bits to index each page.

The TLB must have access to the page directory **base register** to access the page tables.

![Address breakdown](/notes-blog/assets/img/os/address_breakdown.png)

#### Time and Memory space

**Time for page fault**: 1ns (TLB) + 10ms (Level **1** page table, page fault, write old page, load new page) + 10ms (Level **2** page table, page fault, write old page, load new page) $\approx$ 20ms

**Memory Space**:

Pages needed: $(2^{16} , 2^{21} , 2^{21}) / 2^{12} = 2^4 , 2^{9} , 2^{9}$ (3 page tables, each region takes up < 1 page table.)​

| Level              | Size                                                         | Count |
| ------------------ | ------------------------------------------------------------ | ----- |
| 2 (page table)     | $(\text{page size / PTE size})(\text{PTE size}) = 2^{12}$​​ = 4KB | 3     |
| 1 (page directory) | $2^{\text{vaddr-offset-lvl2}}(\text{PTE size}) = 2^{32-10-12}(4) = 2^{12}$​ = 4KB​ | 1     |

Total: 4KB + (3)4KB = 16KB

### Inverted Page Table

**Motivation**: With $M$​ processes, we have $M$​ page tables, but only $N$​ memory frames can be occupied ($N < M$​ overhead).

Inverted PT : frame $\mapsto$​​ <pid, page\#>. [PHYSICAL $\mapsto$​​ VIRTUAL]. Then lookup with the VIRTUAL memory.

**Input**: <pid, page\#, D>: pid to O(n) lookup IPT for page\#. Once found, the index $i$ is the frame.

- Pros: huge memory savings (all pages for all processes in one table)
- Cons: very slow translation (O(n) lookup for IPT for correct PID.)

#### Time and Memory space

Suppose each entry is 8B, bigger than the PTE due to PID.

**Time**: Going through every frame i.e. $2^{20}(30)$ns.

**Memory Space**:

- Number of frames: $2^{32-12} = 2^{20}$
- Size of table: $2^{20} (8)$​ = 8MB

## Page Replacement Algorithms

When a page must be **evicted**:

1. Identify the **process and page number** in the chosen frame
   - **If no IPT**: Scan through every process's page table.
2. Mark **old** process as non-memory resident.
3. Mark **new** process as **memory** resident.

Memory references are modelled as **memory reference strings**, a **sequence** of page numbers. $p_0, p_1, \dots$

$T_\text{access} = (1 - p)T_\text{mem} + pT_\text{page fault}$: a good algo **minimizes page faults** i.e. $p$​, and thus $T_\text{access}$.

### Optimal Page Replacement (OPT)

- Replaces the page that **won't be used for the longest** time
  - i.e. **CRITERIA**: Time of Next Usage
  - Prediction necessary
- Theoretically unfeasible

### First in First out (FIFO)

- Evicts the **oldest** memory page **created**
  - i.e. **CRITERIA**: Time of Page Creation
- Simple queue maintained by OS, dequeue and enqueue during page fault
- Pros: HW support not needed
- Cons:
  - **Belady's anomaly**: As frames increase, so do page faults.
  - Increased frame count means the changes the order of item removal (different set of last N frames created)

### Least Recently Used (LRU)

- **CRITERIA**: Time of last usage
- Pros:
  - Temporal locality
  - No Belady's anomaly (same for all stack algorithms, because top N frames order are always the same)
  - Approximates OPT
- Cons:
  - Needs substantial HW support

**Implementation A**: use a counter.

- Search through all pages in table for least recently used frame
- Logical time is forever increasing (overflow)

![Store the logical time when reference occurs](/notes-blog/assets/img/os/lru_counter.png)

**Implementation B**: stack.

- If frame X in stack is reference, it is removed from where it is and pushed to the top.
- Replace page at stack bottom as needed
- Hard to do HW support

![Stack implementation of LRU](/notes-blog/assets/img/os/lru_stack.png)

### Second-Change Page Replacement (CLOCK)

Modified FIFO. 

- If reference bit = 0: page is replaced
- If reference bit = 1: page ref bit set to 0
- Degenerates into FIFO once all pages are 1 or 0.

| Reference bit marking                                        | Circular nature                                             |
| ------------------------------------------------------------ | ----------------------------------------------------------- |
| <img src="/notes-blog/assets/img/os/fifoclock.png" width="80%"> | <img src="/notes-blog/assets/img/os/clock.png" width="50%"> |

## Frame Allocation Policies

$M$ policies and $N$ frames.

**Equal allocation**: each process gets $N/M$

**Proportional allocation**: each process gets $\frac{x_i}{X} (N)$ frames for $X$ memory space, $x_i$ being size of process $i$​.

**Global Replacement**: Process A can steal a frame from Process B

- Pros: Dynamic adjustment of memory usage 
- Cons: Cascading Thrashing (Process A steals B's frame, leading to B stealing C's frame...)

**Local replacement**: 

- Pros: Performance is stable (frames/process remains constant throughout), limits thrashing to that process.
- Cons: If not enough frames allocated, hinder progress
- Thrashing: Heavy I/O to bring pages into RAM means a single process can hog I/O and degrade other processes' performance

### Working Set

Set of pages in the virtual address space of the process that are currently resident in physical memory.

- Observation: Set of pages currently used typically remains constant in a short period (locality).
  - Set changes gradually over time
- Working set window $\Delta$: one interval of logical time
- `W(t,` $\Delta$​ `)`: active pages within $\Delta$​ of time `t`. Allocate just enough frames for pages in `W(t, \Delta)` for each time `t`

![Working set changes over time](/notes-blog/assets/img/os/workingset.png)

![$\Delta = 5$ in this example](/notes-blog/assets/img/os/deltaworkingset.png)

## MMU - Hardware component

![Memory management unit](/notes-blog/assets/img/os/mmu.png)

1. CPU needs page 2
2. Memory management unit (MMU) checks TLB
   - If in TLB, then MMU can just access the frame directly
3. Not in TLB, MMU accesses the page table (in memory space of OS)

4. Not in page, MMU signals a page fault and control passed to OS.
   - The MMU may also generate illegal access error/invalid page faults leading to segfaults/bus errors.
5. OS brings from secondary storage to the physical memory.

The page table is not in the PCB. Rather it is connected by links, and held in a register (**page directory register**) which is populated at the context switch