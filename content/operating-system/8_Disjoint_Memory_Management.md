---
layout: default
usemathjax: true
permalink: /os/ch8

---

# 8 - Disjoint Memory Management

[Important reference for Page Tables](https://www.cs.cornell.edu/courses/cs4410/2015su/lectures/lec14-pagetables.html)

## Summary

| Type                     | Page Logical Address                                  | Request                           |
| ------------------------ | ----------------------------------------------------- | --------------------------------- |
| Paging                   | page size $\times$ page number $+$ offset             | <page id, offset>                 |
| Segmentation             | <segment id, offset>                                  | <segment id, offset>              |
| Segmentation with paging | <segment id, page size $\times$ page number + offset> | <segment id, page number, offset> |

## Paging

**Physical Frames**: Regions of fixed size in physical memory

**Logical Pages**: Logical memory is split into regions of same size

Pages **loaded** into any available memory frame. Implications:

- **logical** memory space still **contiguous**
- **physical** memory region **disjoined**

**Page Table**: Lookup table that contains the mapping of logical pages to frames.

- $f : \text{ logical pages } \mapsto \text{ frames }$​.
- access with **<page id, page offset>**.

### Logical Address Translation

Given page table $P$, page size $n$​ and the instruction `load x` from logical address, how to convert to physical memory address?

1. Get logical page number $p = \lfloor x/n\rfloor$
2. Get frame number $m = P(p)$
3. Get physical memory address $a = mn + (x \mod n)$
   - because of this multiplication, set $n = 2^k$​ so multiplication becomes a bit shift instead [Design]

Crucial **assumption**: page size = frame size [Design]

#### Criteria

- **External fragmentation**: No. Any available page not used can be used.
- **Internal fragmentation**: Yes. If a logical page does not take up the whole frame space, there will be.

### Implementation

- Each process must have a different page table (no sharing memory!)
- **2 memory accesses**: First to get the frame \# from the page table. Then use the computed memory address to get actual data
- **Associative cache**: the Translation Look-Aside Buffer (**TLB**) is a lookup page table **with cached frame address** to save recomputing for this process.
  - Computing average request time with 40% of page table in TLB
  - Hit: 1ns (TLB) + 50ns (physical mem address)
  - Miss: 1ns (TLB) + 50ns (page table in physical mem) + 50ns (actual access)
    - $\text{hit + miss} = 0.40(1 + 50) + 0.60(1 + 50 + 50)$
  - When context switch occurs, the TLB is **flushed** so that incorrect translation is prevented

![Cache the physical memory addresses translated to pages.](/notes-blog/assets/img/os/tlb.gif)

### Protection

- 3-bit **access rights**: `wrx`
- 1-bit **valid bit**: is the page valid to access?
  - Out of range access for some process (e.g. only using first 60% of the page)

### Page sharing

- Mapping 2 or more processes memory to the same physical frames

### Copy-on-write

- On **forking, before** overwriting memory, map the new process' pages to the **same frame locations**
- Mark the location as **read**-only with the access bits
- Only when a write is required, will the OS copy the page to a new memory location with write access, and reassign write access to the previous page.

## Segmentation

Text, Data, Heap, Stack become individual segments.

- Each segment is contiguous
- Put into different partitions

### Logical Address Translation

Access with **<segment id, offset>**.

- If offset > limit: triggers the `SIGSEGV` signal **(segmentation fault)**.

| Segment   | Base (Starting address) | Limit (Max offset) |
| --------- | ----------------------- | ------------------ |
| 0 (Text)  | ...                     | ...                |
| 1 (Data)  | ...                     | ...                |
| 2 (Heap)  | ...                     | ...                |
| 3 (Stack) | ...                     | ...                |

![Logical Address Translation](/notes-blog/assets/img/os/LAT.png)

### Hardware Support

![Hardware operations](/notes-blog/assets/img/os/hwsupportforseg.png)

Note: The segment table above is in physical memory, not cached. The addressing error is the issued interrupt signal, i.e. **invalid physical address of any sort is never returned**.

**Pros:**

- Each segment can grow/shrink independently
- Each segment can be protected/shared independently

**Cons**:

- Variable-size contiguous memory regions
  - **External fragmentation**

## Segmentation with Paging

Each segment has its own page table as an entry in the segment table.

Now each request is **<segment id, page number, page offset>**.

Segment table consists of **<segment id, page table address>**.

Each page table then contains the **<page number, frame number>**.

![Segment memory regions](/notes-blog/assets/img/os/segpaging.png)

## Memory Management Unit

![Memory Management Unit](/notes-blog/assets/img/os/mmu.png)

The memory management unit (to be better understood in [Chapter 10](/notes-blog/os/ch10)) is providing the hardware support for checking the address requests.
