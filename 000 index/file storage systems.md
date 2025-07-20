Of course! Thank you for providing these excellent and well-organized notes. They cover the core concepts of file systems and disk scheduling from an operating systems course.

I will now convert them into a comprehensive, in-depth Markdown document. I will follow the structure of your notes, explaining each topic, adding context, and expanding on key points to create a complete study guide for you.

---

# A Deep Dive into File Systems and Disk Scheduling

This document provides a detailed explanation of the concepts of file systems, including their structure, management, and implementation, as well as the algorithms used for efficient disk scheduling.

## Table of Contents
1.  [The File Concept](#1-the-file-concept)
    *   [File Attributes](#file-attributes)
    *   [File Operations](#file-operations)
    *   [Open File Tables](#open-file-tables)
2.  [File Access Methods](#2-file-access-methods)
3.  [Directory Structures](#3-directory-structures)
    *   [Single-Level Directory](#single-level-directory)
    *   [Two-Level Directory](#two-level-directory)
    *   [Tree-Structured Directory](#tree-structured-directory)
    *   [Acyclic-Graph Directory](#acyclic-graph-directory)
4.  [File System Implementation](#4-file-system-implementation)
    *   [Layered File System Architecture](#layered-file-system-architecture)
    *   [On-Disk vs. In-Memory Data Structures](#on-disk-vs-in-memory-data-structures)
    *   [Directory Implementation Methods](#directory-implementation-methods)
5.  [Disk Space Allocation Methods](#5-disk-space-allocation-methods)
    *   [Contiguous Allocation](#contiguous-allocation)
    *   [Linked Allocation](#linked-allocation)
    *   [Indexed Allocation](#indexed-allocation)
6.  [Free Space Management](#6-free-space-management)
7.  [Disk Scheduling Algorithms](#7-disk-scheduling-algorithms)
    *   [FCFS (First-Come, First-Served)](#fcfs-first-come-first-served)
    *   [SSTF (Shortest Seek Time First)](#sstf-shortest-seek-time-first)
    *   [SCAN (Elevator Algorithm)](#scan-elevator-algorithm)
    *   [C-SCAN (Circular SCAN)](#c-scan-circular-scan)
    *   [LOOK and C-LOOK](#look-and-c-look)

---

## 1. The File Concept

A **file** is a logical unit of storage, a named collection of related information that is recorded on secondary storage (like a hard disk). From a user's perspective, a file is the smallest allotment of logical secondary storage; data cannot be written to secondary storage unless it is within a file.

### File Attributes

The operating system associates every file with a set of properties, known as **metadata**. This information is stored within the directory structure.

*   **Name:** The symbolic file name in human-readable format (e.g., `report.docx`).
*   **Identifier:** A unique tag (often a number) that identifies the file within the file system. It is non-human-readable.
*   **Type:** Information needed for systems that support different file types (e.g., `.exe`, `.txt`, `.jpg`).
*   **Location:** A pointer to a device and the location of the file on that device.
*   **Size:** The current size of the file in bytes, words, or blocks.
*   **Protection:** Access-control information that determines who can do what. This is typically managed via permissions (Read, Write, Execute) for an owner, a group, and all other users.
*   **Time & Date:** Timestamps for creation, last modification, and last access. This is vital for system administration, backups, and security.

### File Operations

The operating system provides system calls to perform operations on files:

1.  **Creating a file:** Two steps are necessary. First, space in the file system must be found for the file. Second, an entry for the new file must be made in the directory.
2.  **Writing a file:** A system call specifies both the file name/identifier and the information to be written. The system maintains a *write pointer* to the location in the file where the next write is to take place.
3.  **Reading a file:** A system call specifies the file and where in memory to put the contents. The system maintains a *read pointer* to the location in the file where the next read should occur. (Read and write pointers are often combined into a single *current-file-position* pointer).
4.  **Repositioning within a file:** Also known as a "seek," this moves the current-file-position pointer to a specific location. This allows for direct (random) access without reading or writing.
5.  **Deleting a file:** The system reclaims all file space so it can be reused by other files and erases the directory entry.
6.  **Truncating a file:** The user may want to erase the contents of a file but keep its attributes. The file length is reset to zero, and its space is released.

### Open File Tables

To avoid constantly searching the disk for a file's metadata every time it's accessed, the OS uses **in-memory tables** to manage open files. This is a critical performance optimization.

As your diagram correctly shows, there are two primary tables:

1.  **Per-Process Open-File Table:**
    *   Each process has its own private table of open files.
    *   An entry in this table is created when a process executes an `open()` system call.
    *   It contains:
        *   A **File Pointer**: The process's current position in the file (read/write offset). This is unique to each process, allowing multiple processes to read the same file at different positions.
        *   A pointer to the corresponding entry in the system-wide table.
    *   The index into this table is the **file descriptor** (in UNIX/Linux) or **handle** (in Windows) that user programs use to refer to the file.

2.  **System-Wide (Global) Open-File Table:**
    *   There is only one of these tables for the entire system.
    *   It contains an entry for every file currently open by *any* process.
    *   It holds information that is independent of any single process, such as:
        *   The **File Control Block (FCB)**: A copy of the file's metadata (location on disk, size, permissions) brought in from the disk. This is the "Name, FCB, HDD" part of your diagram.
        *   **File Open Count:** A counter that tracks how many processes currently have this file open. When a process closes the file, this count is decremented. When the count reaches zero, the entry is removed from the table.



This two-table structure efficiently manages access. The system only needs to look up the file's physical location from the disk directory once (when the first process opens it). Subsequent opens by other processes are much faster as the FCB is already in the system-wide table.

---

## 2. File Access Methods

This defines how the information in a file is accessed.

1.  **Sequential Access:**
    *   This is the simplest and most common method. Information is processed in order, one record after another.
    *   Read/write operations automatically advance the file pointer.
    *   **Example:** Editors, compilers, text processors. Reading or writing a source code file is almost always done sequentially.

2.  **Direct Access (or Random Access):**
    *   A file is viewed as a numbered sequence of logical blocks or records.
    *   We can read or write blocks in any order, without needing to process them sequentially.
    *   This is essential for applications that need rapid access to specific data.
    *   **Example:** Database systems. To retrieve information about a specific customer, the system can jump directly to that customer's record without reading all preceding records.

3.  **Indexed Access:**
    *   This is a variation built on top of direct access. It involves creating an **index** for the file.
    *   The index, like an index at the back of a book, contains pointers to various blocks. To find a record, you first search the index, which then provides a direct pointer to the block containing the desired data.
    *   This is very effective for large files where even a direct access calculation might be complex.

### Additional Point: Block Size and Internal Fragmentation

As your notes point out, the choice of **block size** is a critical design decision with a direct trade-off:

*   **Internal Fragmentation:** Data is stored in fixed-size blocks. If a file's size is not an exact multiple of the block size, the last block will have unused space. This wasted space *inside* a block is called internal fragmentation.
    *   Your notes correctly identify the average wastage as `Block Size / 2`. For a 512-byte block, the wastage can be anywhere from 0 bytes (if the file ends perfectly) to 511 bytes (if the file needs just 1 byte of a new block). The average is 256 bytes.
*   **The Trade-off:**
    *   **Larger Block Size:**
        *   **Pro:** Faster I/O transfer rates (more data read per seek). Less metadata overhead (fewer blocks to track).
        *   **Con:** More wasted space due to internal fragmentation.
    *   **Smaller Block Size:**
        *   **Pro:** Less wasted space. More efficient for storing many small files.
        *   **Con:** Slower I/O (more seeks needed to read the same amount of data). More metadata overhead.

The optimal block size is experimentally determined based on the expected average file size for a given system. 512 bytes, 1 KB, 2 KB, and 4 KB are common sizes.

---

## 3. Directory Structures

A **directory** is a special file that contains information about other files. As your notes state, it's a "translation table" that maps human-readable file names to their corresponding file control blocks (metadata). The structure of these directories dictates how files are organized.

**Operations a directory should support:**
*   Search for a file
*   Create a file
*   Delete a file
*   List a directory's contents
*   Rename a file
*   Traverse the file system

The core problem is managing millions of files. A single directory for everything is not feasible due to performance bottlenecks and naming conflicts. This leads to different directory structures.

### Single-Level Directory

All files are contained in the same directory.



*   **Pros:** Simple to implement.
*   **Cons:**
    *   **Naming Collisions:** All files must have unique names. This is impossible on a multi-user system.
    *   **Organization:** Difficult for a user to manage and group their own files.

### Two-Level Directory

A master file directory (MFD) exists, and each user has their own user file directory (UFD).



*   **Pros:** Solves the naming collision problem between users. User 1 can have a `test.c` and User 2 can also have a `test.c`.
*   **Cons:**
    *   **No Sub-grouping:** A user cannot create subdirectories to organize their projects.
    *   **Isolation:** It's difficult for users to cooperate and share files.

### Tree-Structured Directory

This is the most common modern structure. It extends the two-level directory into a multi-level tree.



*   **Key Concepts:**
    *   **Root Directory:** The top of the tree.
    *   **Path Name:** The unique path from the root to a file.
    *   **Absolute Path:** Specifies the location from the root (e.g., `/root/user1/bin/proj/b1.exe`). Always starts with a `/`.
    *   **Relative Path:** Specifies the location relative to the current working directory (e.g., if `pwd` is `/root/user1/bin`, the relative path to `b1.exe` could be `proj/b1.exe`).

*   **Pros:** Highly flexible, allows for complex organization, intuitive for users.
*   **Cons:** Sharing files is still somewhat rigid.

### Acyclic-Graph Directory

This structure enhances the tree by allowing directories to have shared subdirectories and files. A tree is a graph with no cycles, but here we allow sharing, creating a structure that is a directed acyclic graph (DAG).



*   **Implementation:** Sharing is typically achieved through **links** or **shortcuts**.
    *   **Hard Link:** Multiple directory entries point to the same physical file metadata (the same FCB/i-node).
    *   **Symbolic (Soft) Link:** A special file is created that simply contains the path to the actual file.

*   **Problem: Deletion and Dangling Pointers**
    *   As your notes highlight, if User 1 deletes file `F1`, what happens to User 2's link? If we simply deallocate the file, User 2's entry becomes a "dangling pointer" that points to invalid data.
*   **Solution: Reference Counting**
    *   The system maintains a **reference count** within the file's metadata (FCB/i-node) that tracks how many directory entries are pointing to it.
    *   When a link is created, the count is incremented.
    *   When a link is deleted, the count is decremented.
    *   The file's physical space is only deallocated when the reference count reaches **zero**.

---

## 4. File System Implementation

### Layered File System Architecture

File systems are often implemented in a layered architecture to manage complexity. Each layer provides services to the layer above it and uses services from the layer below it.

1.  **Application Programs:** The user interacts here (e.g., using a text editor).
2.  **Logical File System:** Manages metadata, directory structures, and protection/permissions. It translates a symbolic file name into the information needed to access it.
3.  **File-Organization Module:** Knows about files and their logical blocks. It translates logical block addresses to physical block addresses. It also manages free space on the disk.
4.  **Basic File System:** Issues generic commands to the appropriate device driver to read and write physical blocks on the disk. It deals with I/O buffering.
5.  **I/O Control:** Consists of device drivers and interrupt handlers. This is the code that actually communicates with the hardware controllers.
6.  **Devices:** The physical hardware (e.g., hard disk drives, solid-state drives).

### On-Disk vs. In-Memory Data Structures

The file system relies on data structures both on the physical disk (for persistence) and in main memory (for performance).

#### On-Disk Structures

These structures must persist even when the computer is turned off.

*   **Master Boot Record (MBR) / Partition Table:** The very first sector of a disk. It contains a small program to boot the system and a partition table that defines the location and size of the partitions on the disk.
*   **Boot Control Block (per volume/partition):** The first block of a partition. It contains information needed to boot an OS from that partition (e.g., `NTLDR` in Windows NTFS, `grub` in Linux).
*   **Volume Control Block (Superblock in Unix):** Contains details about the entire partition, such as the total number of blocks, block size, free block count, and pointers to free-space management structures.
*   **Directory Structure:** Used to organize the files. In Unix-like systems, this is a series of file names and their corresponding i-node numbers.
*   **File Control Block (FCB or i-node in Unix):** Contains all the metadata about a single file: permissions, owner, size, and crucially, the pointers to the actual data blocks on disk.

#### In-Memory Structures

These are loaded from disk to RAM to speed up access. They are discarded on shutdown or dismount.

*   **In-Memory Mount Table:** Tracks all the mounted file systems (partitions).
*   **In-Memory Directory-Structure Cache:** Caches recently accessed directories to speed up name-to-i-node translation.
*   **System-Wide Open-File Table:** As discussed before, holds an FCB for every open file.
*   **Per-Process Open-File Table:** As discussed before, holds pointers to the system-wide table for each file a process has open.

### Directory Implementation Methods

1.  **Linear List:**
    *   A simple list of file names with pointers to their corresponding data blocks or FCBs.
    *   **Advantage:** Simple to implement.
    *   **Disadvantage:** Searching is very slow, requiring a linear scan (**O(n)**). Deleting files can be complex (either mark the entry as unused, or shift all subsequent entries).

2.  **Hash Table:**
    *   A hash function is applied to the file name to compute an index into a hash table. The table entry then points to the file's metadata.
    *   **Advantage:** Searching is very fast, on average **O(1)**.
    *   **Disadvantage:** Subject to **collisions** (when two file names hash to the same location), which must be handled. The table size is generally fixed.

---

## 5. Disk Space Allocation Methods

This section describes how disk blocks are allocated for files.

### Contiguous Allocation

Each file occupies a contiguous set of blocks on the disk. The directory entry only needs to store the starting block address and the file's length.



*   **Advantages:**
    *   **Excellent read performance.** Since all blocks are together, seek time is minimal once the head is at the start block.
    *   Simple to implement. Supports direct/random access easily.
*   **Disadvantages:**
    *   **External Fragmentation.** The free space on the disk gets broken up into small, non-contiguous chunks. It can become difficult to find a large enough contiguous block of space for a new file, even if the total free space is sufficient.
    *   Files cannot easily be extended once created. The required size must be known in advance.

### Linked Allocation

Each file is a linked list of disk blocks, which may be scattered anywhere on the disk. The directory entry contains a pointer to the first block. Each block contains a pointer to the next block in the chain.



*   **Advantages:**
    *   **No external fragmentation.** Any free block can be used.
    *   Files can be grown easily by just adding a new block to the end of the chain.
*   **Disadvantages:**
    *   **No efficient random access.** To get to block `i` of a file, you must start at the beginning and follow the `i-1` pointers. This is very slow.
    *   **Overhead:** Space in each block is consumed by the pointer.
    *   **Low reliability:** A single lost or damaged pointer can cause the rest of the file to be lost.

#### Variation: File Allocation Table (FAT)

This is a clever optimization of linked allocation, famously used by MS-DOS and early Windows. The pointer chain is taken out of the data blocks and stored in a separate table at the beginning of the volume, called the **File Allocation Table (FAT)**.



*   **How it works:** The FAT is an array, with one entry for each disk block. The directory entry points to the starting block number of the file. The FAT entry corresponding to that block number contains the block number of the next block in the file. A special end-of-file (EOF) value marks the last block.
*   **Advantages:**
    *   Random access is much improved, as the entire pointer chain can be traversed quickly in the in-memory FAT.
*   **Disadvantages:**
    *   The FAT itself can become very large and must be cached in memory, which can consume significant RAM.
    *   Your notes provide an excellent example: For a 20GB disk with 1KB blocks, you have 20 million blocks. If each FAT entry is 4 bytes, the FAT itself would require `20M * 4B = 80MB` of RAM.

### Indexed Allocation

This method brings all the pointers for a file together into one location: an **index block**.



*   **How it works:** Each file has its own index block, which is an array of disk-block addresses. The *i*-th entry in the index block points to the *i*-th block of the file. The directory entry points to this index block.
*   **Advantages:**
    *   Solves the external fragmentation problem.
    *   Provides efficient direct/random access.
*   **Disadvantages:**
    *   **Wasted space:** For small files, the index block itself might be mostly empty but still consumes an entire block.
    *   **File size limit:** The maximum file size is limited by the number of pointers that can fit in a single index block.

#### Handling Large Files with Indexed Allocation

To overcome the size limit, several schemes are used:

*   **Linked Scheme:** If a file needs more pointers, you can link multiple index blocks together.
*   **Multi-Level Index:** Use a "master" index block that points to other index blocks, which in turn point to the data blocks.
*   **Combined Scheme (The Unix i-node approach):** This is a brilliant, practical hybrid used in most Unix-like systems (like Linux). The i-node (index block) contains:
    *   **Direct Blocks:** A set of (e.g., 10-12) pointers that point *directly* to data blocks. This provides extremely fast access for small files.
    *   **Single Indirect Pointer:** Points to an index block full of pointers to data blocks.
    *   **Double Indirect Pointer:** Points to a block, where each entry points to another index block.
    *   **Triple Indirect Pointer:** Adds another level of indirection for astronomically large files.

![Combined Scheme (i-node)](https://i.imgur.com/n1477l8.png)

#### Example Problems (from your notes)

**GATE '04 Example:**
*   **Given:** 10 direct, 1 single indirect, 1 double indirect, 1 triple indirect. Disk block size = 1KB (1024 B). Disk block address = 4 B.
*   **Pointers per block:** `1024 B / 4 B = 256` pointers.
*   **Calculation:**
    *   Direct: `10 blocks`
    *   Single Indirect: `256 blocks`
    *   Double Indirect: `256 * 256 = 65,536 blocks`
    *   Triple Indirect: `256 * 256 * 256 = 16,777,216 blocks`
*   **Max File Size:** `(10 + 256 + 65536 + 16777216) * 1024 B` ≈ 16 TB.
    *(Your notes have a slight calculation error, showing 170 pointers per block. 1024/4 is 256. But the logic is correct!)*

**GATE '12 Example:**
*   **Given:** 8 direct, 1 single indirect, 1 double indirect. Block size = 128 B. Block address = 8 B.
*   **Pointers per block:** `128 B / 8 B = 16` pointers.
*   **Calculation:**
    *   Direct: `8 blocks`
    *   Single Indirect: `16 blocks`
    *   Double Indirect: `16 * 16 = 256 blocks`
*   **Total Blocks:** `8 + 16 + 256 = 280` blocks.
*   **Max File Size:** `280 blocks * 128 B/block = 35,840 B = 35 KB`.
    *(The calculation in your notes `(8 + 16 + 16*16)` is correct! The final size calculation `(1+2+32)KB` seems to be a simplification or mistake. The correct result is 35 KB.)*

---

## 6. Free Space Management

The file system must track which blocks are free to be allocated.

1.  **Bit Vector (or Bitmap):**
    *   The free-space list is implemented as a bit vector. Each block is represented by 1 bit. If the block is free, the bit is `1`; if it's allocated, the bit is `0` (or vice-versa).
    *   **Pros:** Simple and efficient for finding the first free block, or a contiguous chunk of `n` free blocks.
    *   **Cons:** The bitmap itself requires space. For a 1TB disk with 4KB blocks, the bitmap would be `(1TB / 4KB) / 8 = 32MB`, which must be kept in memory for performance.

2.  **Linked List:**
    *   All the free disk blocks are linked together, with a pointer in a special location on disk (e.g., in the superblock) pointing to the first free block.
    *   **Pros:** No wasted space on a bitmap. Easy to find one free block.
    *   **Cons:** Very inefficient for finding contiguous blocks of space.

---

## 7. Disk Scheduling Algorithms

The OS must manage I/O requests for the hard disk. Since mechanical disks are slow, and the slowest part is moving the read/write head (**seek time**), the OS uses algorithms to schedule requests in an order that minimizes total seek time.

**Common Scenario (from your notes):**
*   Request Queue (cylinders): `23, 89, 132, 42, 187`
*   Disk Head starts at cylinder: `100`
*   Total Cylinders: 0-199 (assumed)

### FCFS (First-Come, First-Served)

The simplest algorithm. Requests are serviced in the order they arrive.

*   **Path:** `100 -> 23 -> 89 -> 132 -> 42 -> 187`
*   **Head Movement:** `(100-23) + (89-23) + (132-89) + (132-42) + (187-42) = 77 + 66 + 43 + 90 + 145 = **421 cylinders**`
*   **Pros:** Absolutely fair, no starvation.
*   **Cons:** Wild head movement, extremely inefficient.

### SSTF (Shortest Seek Time First)

Selects the request with the minimum seek time from the current head position.

*   **Path:** `100 -> 89 -> 42 -> 23 -> 132 -> 187`
*   **Head Movement:** `(100-89) + (89-42) + (42-23) + (132-23) + (187-132) = 11 + 47 + 19 + 109 + 55 = **241 cylinders**`
*(Your note's calculation of 273 seems to have a different path. Following pure SSTF from 100, 89 is closest. From 89, 42 is closest. From 42, 23 is closest. From 23, 132 is closest. From 132, 187 is closest. The total is 241.)*
*   **Pros:** Greatly reduces average seek time.
*   **Cons:** **Starvation** is possible. If new requests keep arriving close to the current head position, requests that are far away may never be serviced.

### SCAN (Elevator Algorithm)

The disk arm starts at one end of the disk and moves toward the other end, servicing requests as it goes. When it reaches the other end, it reverses direction.

*   **Path (assuming moving towards 0 first):** `100 -> 89 -> 42 -> 23 -> 0 -> 132 -> 187`
*   **Head Movement:** `(100-89) + (89-42) + (42-23) + (23-0) + (132-0) + (187-132) = 11 + 47 + 19 + 23 + 132 + 55 = **287 cylinders**`
*   **Pros:** Prevents starvation, more uniform than SSTF.
*   **Cons:** Favors requests in the middle cylinders and penalizes requests at the far ends.

### C-SCAN (Circular SCAN)

Like SCAN, but when the head reaches the end, it immediately returns to the beginning of the disk without servicing any requests on the return trip.

*   **Path (assuming moving towards 199 first):** `100 -> 132 -> 187 -> 199 -> 0 -> 23 -> 42 -> 89`
*   **Head Movement:** `(132-100) + (187-132) + (199-187) + (199-0) + (23-0) + (42-23) + (89-42) = 32 + 55 + 12 + 199 + 23 + 19 + 47 = **387 cylinders**`
*(Your note's path seems to move inwards first, which gives a different result. The principle is the key.)*
*   **Pros:** Provides a more uniform and fair waiting time than SCAN.

### LOOK and C-LOOK

These are practical optimizations of SCAN and C-SCAN. The arm only goes as far as the last request in each direction, instead of all the way to the end of the disk.

*   **LOOK:** (Like SCAN, but "looks" for the last request)
    *   **Path (towards 0):** `100 -> 89 -> 42 -> 23 -> 132 -> 187`
    *   **Head Movement:** `(100-23) + (187-23) = 77 + 164 = **241 cylinders**`
    *(Your notes have the same total, excellent!)*

*   **C-LOOK:** (Like C-SCAN, but "looks" for the first/last request)
    *   This is the most common practical algorithm.
    *   **Example from notes:** Queue `98, 183, 37, 122, 14, 124, 65, 67`, Head at `53`.
    *   **Path (moving towards 199):** `53 -> 65 -> 67 -> 98 -> 122 -> 124 -> 183` (last request) then jumps to `14` (first request) and moves towards 199 again `-> 37`.
    *   **Head Movement:** `(65-53) + (67-65) + (98-67) + (122-98) + (124-122) + (183-124) + (183-14) + (37-14) = 12 + 2 + 31 + 24 + 2 + 59 + 169 + 23 = **322 cylinders**`
    *(Your calculation and diagram are perfect!)*

---

This covers all the topics from your handwritten notes in a detailed, structured, and expanded format. I hope this digital version serves as an excellent resource for your studies! Let me know if you have any more notes to convert.