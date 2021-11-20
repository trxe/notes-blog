---
layout: default
usemathjax: true
permalink: /os/ch11
---

# 11 - File System Implementations

## Data structures Summary

| File block allocation                     | Free space management                        | Directory structure         |
| ----------------------------------------- | -------------------------------------------- | --------------------------- |
| [Contiguous](#contiguous-allocation)      | [Disk block bitmap](#block-bitmap)           | [Linear list](#linear-list) |
| [Linked List](#linked-list-allocation)    | [Disk block linked list](#block-linked-list) | [Hash table](#hash-table)   |
| [FAT](#file-allocation-table)             |                                              |                             |
| [Indexed allocation](#indexed-allocation) |                                              |                             |



## Disk organization

- Sector 0: Master Boot Record (**MBR**)
  - Simple boot code
  - Partition table
- Sector $\geq$ 1: Partitions

| OS Boot Block | Partition Details | Directory Structure | File Info | File Data |
| ------------- | ----------------- | ------------------- | --------- | --------- |
|               |                   |                     |           |           |
| ------------- | ----------------- | ------------------- | --------- | --------- |

### General Disk Structure: 1D Logical Block array

**Logical block**: Smallest accessible unit (~512B to 4KB), mapped into disk sectors (layout id disk sectors hardware dependent, dictated by drivers).

## File info and data

**File**: Collection of logical blocks. When file size $\neq$​ multiple of logical blocks: internal fragmentation.

- **Track** logical blocks
- **Efficient** access
- Disk space used **effectively**

### Data Allocation on disk

#### Contiguous allocation

- Knowledge of start block and size
- Pro: Easy write and access
- Con: External Fragmentation (same as memory)

#### Linked list allocation

- Each block contains a link to the next available block, knowledge of start and end block
- Pro: No external fragmentation
- Cons: 
  - Overhead (each block has to store links)
  - lost links = dead sectors = less reliable
  - Random access becomes $O(n)$​ time for size $n$​ file.

#### File Allocation Table

- Move all block pointers into a single FAT, which is always in **memory**
- Pro: No external fragmentation, faster than normal LL as it is in RAM
- Cons:
  - Huge amount of space in RAM (e.g. 1TB of disk, 512B blocks and 4B file entries = $\approx$​ 3GB of RAM for FAT)
  - Must still be persistently stored in disk

#### Indexed Allocation

- i.e. Linked list per file
- Each file has **one index block** of disk block addresses, and `indices[N] ==` Nth block address
- Pros: 
  - Less memory overhead
  - Random access becomes $O(1)$ time again
- Cons:
  - Max number of blocks per file = Max number of entries in index block
  - 1 block of overhead

### Free space management

2 operations: **Allocate** (file creation/expansion) and **free** (deletion/truncation)

![Bitmap](/notes-blog/assets/img/os/diskblockbitmap.png)

#### Block bitmap

- Pro: Good set of manipulations
- Con: Need to store lists in memory to be efficient, takes up a lot of memory

#### Block linked list

- **One** disk block contains: number of free disk blocks or a pointer to the next free space disk block.	
- Pros: 
  - Easy location of free block
  - Only 1st pointer needed
- Con: high overhead

### Directory structure

Track files + file information in a directory, then maps file name to file info.

Given a full path name, recursive search of directories necessary to arrive at file info.

#### Linear list

- Each entry represents a file, one entry stores file name, metadata
- Con: Linear search necessary, bad for deep trees
  - Use cache to remember last few searches (DFS).

#### Hash Table

- File name is hashed into $K \in [0, N-1]$​
- Chained collision resolution
- Pro: Fast lookup
- Cons:
  - Limited size $N$​
  - Depends on quality of hash function

#### File Information

- Includes file name, metadata, block information
- Methods of storage: 
  - Everything is stored in directory entry
  - Only file name + pointers to the other information in other DS

## Case studies

### Microsoft FAT File System

- Block allocation info kept in a linked list.
- Data block pointers kept in the File Allocation Table [FAT](#data-allocation-on-disk)
- One entry per data block/cluster
- Stores disk block information
- OS caches in RAM for linked list traversal.

**FAT entry:** `FREE / Block num / EOF / BAD`. To find a file, you need to have the index of the **first block** of the file.

**Directory:** Special type of file. (*Root* directory is in a special location. Other directories are in data blocks.) Every file/subdirectory = **directory entry.**

| Order | Directory entry item     | Bytes |
| ----- | ------------------------ | ----- |
| 1     | File name                | 8     |
| 2     | File ext.                | 3     |
| 3     | Attributes               | 1     |
| 4     | *Reserved*               | *10*  |
| 5     | Creation date            | 2     |
| 6     | Creation time            | 2     |
| 7     | First disk block (FAT16) | **2** |
| 8     | File size                | 4     |

FAT16 can only have $2^{16}$​​​ block indices, and thus blocks, with 16 bits.

To access a file we start from the initial block and access the initial block's next block, and so on.

![Path of access](/notes-blog/assets/img/os/fatacc.png)

To search for an empty block, you have to do a linear pass through the whole file to find `FREE` blocks.

| Boot          | FAT                           | FAT dup. | Root dir.      | Data blocks                |
| ------------- | ----------------------------- | -------- | -------------- | -------------------------- |
| OS boot block | Partition details + File info | -        | Dir. structure | Dir. structure + File data |

### Extended File System 2

- Disk space split into **blocks**
- Blocks split into **block groups**
- Each file/directory described by an **I-Node**
  - File metadata (access right, creation time)
  - Data block addresses
- **Nothing** has to be in memory, but most recent I-nodes be cached for speed.

| Order | Block Group Item  | Info                                                         |
| ----- | ----------------- | ------------------------------------------------------------ |
| 1     | Superblock        | Total I-node \#, I-nodes per group, total disk blocks, disk blocks per group |
| 2     | Group descriptors | Number of free disk blocks, free I-nodes, location of bitmaps |
| 3     | Block bitmap      | One bit per block in BG. (0/1)                               |
| 4     | I-node bitmap     | One bit per I-node in BG. (0/1)                              |
| 5     | I-node table      | Array of indexed I-nodes of this BG.                         |
| 6     | Data blocks       |                                                              |

[1, 2] are duplicated in every block group.

| Order | I-node details                       | Bytes         |
| ----- | ------------------------------------ | ------------- |
| 1     | Mode                                 | 2             |
| 2     | Owner info                           | 4             |
| 3     | File size                            | 4 or 8        |
| 4     | Timestamps                           | $3 \times 4$  |
| 5     | **Data block pointers**              | $15 \times 4$ |
| 6     | Reference count [e.g. \# hard links] | 2             |
| ...   | etc...                               | ...           |
| TOTAL |                                      | **128**       |

Note that file name is not in the I-node, i.e. it is contained in the data blocks the I-node points to.

These are the data blocks that constitute the file data. **Indirect blocks do not contain file data itself.**

- pointers 1 to 12: Direct blocks (level 0)
- ptr 13: Indirect blocks (level 1)
- ptr 14: Double indirect blocks (level 2)
- ptr 15: Triple indirect blocks (level 3)

![I-node tree](/notes-blog/assets/img/os/ext2inode.png)

How much space? (4B block address, 1KiB disk block)

$$
\begin{aligned}
n &= 2^{10} / 4 \quad \text{block addresses per disk block} \\
\text{blocks} &= 12 + n + n^2 + n^3 \\
\text{size} &= \# \text{blocks} \times 2^{10} \approx 16 \text{GB}
\end{aligned}
$$

**Directory**: Contains a linked list of **directory entries** = files/sub-directories' info within this directory. (Root directory is at I-node 2.)

To **access** a file: `/sub/file`

1. [I-node Table] `/` at I-node 2
2. [I-node 2] File metadata for `sub` at disk-block 15
3. [Disk block 15] `sub` at I-node 8
4. [I-node 8] File metadata for `file` at disk-block 28
5. [Disk block 28] `file` at I-node 4
6. [I-node 4] actual file at disk-block 13.

To **delete** a file:

1. Remove directory entry from parent (by removing from file-metadata linked list)
2. Update I-node bitmap: mark as 0
3. Update block bitmap: mark as 0

![Directory entry](/notes-blog/assets/img/os/direntry.png)

**Hard Links**:

- Points to the same I-node.
- When original pointer A is deleted, the I-node remains attached to the hard link B.
- With many references, when should I delete an I-node?
  - When I-node reference count = 0
  - Which is decremented (for both A and B) on each deletion

**Soft/Symbolic link**:

- Points to a filepath to the desired I-node.
- ![Symbolic link](/notes-blog/assets/img/os/symlink.png)

