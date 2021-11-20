---
layout: default
usemathjax: true
permalink: /os/ch10
---

# 10 - File Management

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

## File system

Physical memory is volatile, so external/secondary storage is needed to **store persistent info**.

Where to find information (storage media) on a device when moved?

- Abstraction of physical media
- High level resource management scheme
- Protection between processes and users
- Sharing between processes and users

### Criteria

- Self-contained (can plug-and-play):
  - Info on media enough to describe whole organization
- Persistent
  - last beyond lifetime of OS/processes
- Efficient
  - management of free/used space
  - minimum overhead

A mental picture of secondary storage:

1. Metadata is contained within the file itself
2. The files in a folder may not be in a contiguous region (similar fragmentation problems)
3. Data of a file is in contiguous region

### Comparison with memory management

|                    | Memory Management                                           | File System Management                         |
| ------------------ | ----------------------------------------------------------- | ---------------------------------------------- |
| Storage            | RAM                                                         | Disk                                           |
| Access speed       | Constant                                                    | Variable disk I/O time                         |
| Unit of addressing | Physical memory address                                     | Disk sector                                    |
| Usage              | Address space of process, implicit during process execution | Non-volatile data, explicit access             |
| Organization       | Paging/Segmentation                                         | ext* (Linux), FAT/NTFS (Windows), HFS* (MacOS) |

every time OS has to access a different file system, the file management has to be replaced.

File

Definition: A logical unit of information created by a process

- Abstract Data Type (ADT)
- Common operations with various implementation
- Contains **Data**: info structure in some way
- Contains **Metadata**: additional info  associated with file (attributes)
  - Name (human readable fileref), Identifier (unique id), Type (e.g. executable vs text files vs object file vs directory), size, protection (access), time-date-owner, table of contents (how to access a file).

Definition, Metadata, Data (Structure, access methods), File Operation

### Some differences between Linus `ext` and Windows `NTFS` filesystems

|                    | Linux                                               | Windows                      |
| ------------------ | --------------------------------------------------- | ---------------------------- |
| File names         | Case-sensitive                                      | Case-insensitive             |
| Special characters | Allowed                                             | Not allowed                  |
| File type          | Determined by first 8 bits (first byte) of the file | Determined by file extension |

**File type:** a description of the information contained in the file (based on a certain format etc.)

**File extension:** part of the file name after the dot, used in Windows etc. to identify file type

**Magic number:** set of k-byte (usually $k=2$​) identifiers at start of file.

**Access Control List (ACL):**

- Minimal ACL (same as permission bits) -- `drwxr--r--` (user - group - others)
- Extended ACL (added named users/group) -- `getfacl`

## File system architecture

**File data structure options:**

- Array of bytes (each has specific offset from the start.)
- Fixed length records (array of records that can grow/shrink) **similar to fixed memory partition, internal fragmentation**
- Variable length records (flexible but harder to locate) **similar to variable-size memory partition, external fragmentation** 

**Access methods:**

- **Sequential access**: read in order, starting from beginning (can be rewound but not instantly access a location)
- **Random access**: access any byte directly 
  - `Read(offset)`: explicitly state position to access
  - `Seek(offset)`: special operation move cursor to new location in file
  - Windows, Linux use `seek` (`lseek`)
  - Random access is a special case of direct access where `1 record == 1 byte`
- **Direct access**: access any record directly
  - Used for fixed length records

The operation of `open` has an important role to keep references to the all the pointers necessary to access a physical disk, removing the need to access the full filepath every time. It also tracks the **file offset**.

**File operations**:

| Operation | Description                                        |
| --------- | -------------------------------------------------- |
| Create    | New file, no data                                  |
| Open      | Prepare necessary information for later operations |
| Read      | Read data from current position                    |
| Write     | Write data to current position                     |
| Seek      | Move position to a new location                    |
| Truncate  | Remove data from current position to end of file   |

- Current position is maintained by the **file pointer**.
- Via system calls (protection, concurrent and efficient access)
- **File descriptor** should link to the following information for each opened file:
  - File pointer (current location in file)
  - Disk location (file location in disk)
  - Open count (how many process has this file opened)

**File information (3 tables)**:

- Per-process open-file table
  - Each PCB contains a file descriptor (fd) table
  - Each fd entry stores a pointer to an entry in sys-wide open-file table
  - Open files for a process
  - Each entry points to the system-wide open-file table
- System-wide open-file table (accessed on `open` syscall)
  - Open files for the system
  - Each entry points to a V-node entry
- System-wide V-node entry
  - Link with file on physical drive ([I-node](/notes-blog/os/ch11#extended-file-system-2))

![The three file information tables](/notes-blog/assets/img/os/filetables.png)

*Op. type*: If a file is open for reading only but has an attempt to write to it, it will not be allowed.

1. **Case 1**: Same file, 2 processes, independent offsets.
   - Different PCB -> Different fd tables -> Different system-wide open file table entry -> **Same V-node entry**
2. **Case 2**: Same file, 1 parent and 1 child process (just forked, no mod)
   - Different PCB -> **Cloned fd tables** -> **Same system-wide open file table entry** -> Same V-node entry

| Different FD table                                           | Same FD table                                                |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![diff fd](/notes-blog/assets/img/os/difffd_same_file_access.png) | ![same fd](/notes-blog/assets/img/os/samefd_same_file_access.png) |

## Directories

**Purpose**: Provide a **logical grouping** (user view), **keep track** of files (actual system usage)

Possible data structures:

- Single level: A directory cannot contain other directories.
- Tree-structured: A directory can contain other directories.
  - Absolute pathname: path from root dir to file
  - Relative pathname: path from **current working directory (CWD)** to file.
  - CWD can be explicitly/implicitly changed by moving to a new dir in shell prompt.
- DAG: Used to show the nature of symlinks
  - One file appears linked to multiple directories
  - **Hard links** can only be made from interior node to leaves to prevent cycles.
- General graph: 
  - Not desirable: hard to traverse (cycle), and hard to determine when to remove a file/directory
  - **Symbolic links**

**Directory read write access:** This does **not** extend to the contents of the directory!

- Read: can this directory's list be read?
- Write: can we modify the **list** of files in the directory (by creating, renaming or deleting files, but not modifying content)?
- Exec: can this directory be used as working directory?

## Disk scheduling

In HDD, two fundamental movements to access a particular disk location: **rotate disk** and **move head**.

Traditional Disk scheduling algorithms to minimize latency necessary:

- FCFS
- Shortest Seek First (SSF)
- SCAN Family (Elevator)
  - Bi-directional [Inner to outermost and back]
  - One-directional [Outer to innermost]

![SCAN](/notes-blog/assets/img/os/scan.png)

Newer Disk scheduling algorithms

- Deadline - 3 queues
  - **Sorted**  (depending on what type of storage device, e.g. if in disk then sort by min head movement/rotation etc.)
  - Read FIFO (when deadline approaches, instead of popping from sorted, pop from the FIFOs)
  - Write FIFO
- noop (No-operation) - no sorting (FIFO)
- cfq (Completely fair queueing) - time slice and per-process sorted queues
- bfq (Budget fair queueing) (Multiqueue) - fair sharing based on \# sectors requested

## Buffering

**Motivation:** File operations are inherently expensive.

1. Each file operation requires a syscall requiring execution mode (user -> kernel) change
2. High disk access latency
