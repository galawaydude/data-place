
Memory management is an important function of an operating system, its main goal is to allocate and deallocate memory space for processes, ensuring efficient utilization of memory resources, protection between processes, and enabling a high degree of multiprogramming.
### Some components

The CPU interacts with various memory components in a hierarchy.
*   **Registers (Regs):** Smallest, fastest, and most expensive memory, located inside the CPU. Used to hold data and instructions currently being processed.
*   **Cache:** A small, fast memory that stores copies of data from frequently used main memory locations. It's faster than main memory but slower than registers.
*   **Main Memory (RAM - Random Access Memory):** The primary storage where programs and data are loaded for execution. Much larger than cache but slower.
*   **Hard Disk Drive (HDD) / Solid State Drive (SSD):** Secondary storage, much larger and cheaper than RAM, but significantly slower. Used for persistent storage of programs and data.

### CPU Utilization

A key metric in multiprogramming is **CPU Utilization**, which measures the percentage of time the CPU is actively executing instructions, rather than waiting (e.g., for I/O operations).
**Let's define `p`:**
p = The probability that a process is waiting for I/O (Input/Output) to complete.
Processes don't just compute; they also frequently perform I/O operations (reading from disk, writing to a screen, network communication, user input). During an I/O operation, the CPU sits idle for that process. If p is the probability of a process waiting for I/O, then (1-p) is the probability that the process is actively using the CPU (performing a CPU burst).

**Consider a system with n processes in memory (degree of multiprogramming).**
For the CPU to be idle, all n processes must *simultaneously* be waiting for I/O.
The probability of one process waiting for I/O is p.
The probability of n processes all waiting for I/O (assuming independence) is p * p * ... (n times) = $1 - p^n$

Therefore, the probability that at least one process is *not* waiting for I/O (i.e., at least one process is ready to use the CPU) is $1 - p^n$.

The CPU Utilization is given by $1 - p^n$.
Where `n` is the number of processes that can be present in memory at the same time.

**Key takeaway:** CPU utilization is directly proportional to the degree of multiprogramming (`n`).
*   As `n -> ∞` (n approaches infinity), CPU utilization `-> 1` (100%).
*   **However, for CPU utilization to be 100%, `n` would theoretically need to be infinite.** This is not practical.
*   For this, Main Memory size should be very, very costly.
*   **The solution:** We need to **use the memory in such a way that we can achieve a high degree of multiprogramming without acquiring an infinite (or prohibitively expensive) amount of physical memory.** This is the fundamental challenge memory management addresses.

### Object Code Relocation & Linkers

Before a program can run, it undergoes several transformations. One critical phase involves handling addresses, particularly when combining different parts of a program or linking with external libraries.

The journey from source code to an executable program typically involves these steps:

1.  **Source Code:** Human-readable code (e.g., C++, Java).
2.  **Compiler:** Translates source code into **assembly code**. It performs lexical analysis, syntax analysis, semantic analysis, intermediate code generation, code optimization, and target code generation.
3.  **Assembly Code:** A low-level programming language that is a symbolic representation of machine code.
4.  **Assembler:** Translates assembly code into **object code (machine code)**. It also builds initial symbol tables and resolves symbolic addresses within the current module.
5.  **Object Code (Object Modules):** Machine-readable code containing instructions and data, but typically with "placeholder" addresses for external references. These are often referred to as relocatable object files. *we do more on this shit, later*
6.  **Linker:** Combines multiple object modules (and libraries) into a single **executable file**. It resolves external symbol references and performs relocation.
7.  **Loader:** Loads the executable file into main memory, assigning actual physical addresses, so the CPU can execute it.

An object code file (or object module) is not just raw machine instructions. It's structured with various sections to facilitate linking and loading:

*   **Header:** Contains basic information about the module, such as its size, creation date, and entry points.
*   **Text Segment (Code Segment):** Contains the actual machine instructions (the executable code) for the module.
*   **Data Segment:** Contains initialized global and static variables.
*   **Relocation Info (Relocation Table):** This is crucial. It lists all the instructions and data locations within the module that depend on absolute memory addresses. These addresses need to be adjusted (relocated) when the module is loaded into memory.
    *   **Absolute Address:** A fixed, precise memory location (e.g., `0x12345678`).
    *   **Relocatable Address:** An address specified relative to the start of the module or another base address. This address will change when the module is loaded.
    *   **Relocation:** The process of converting relocatable addresses into absolute addresses once the program's load-time memory location is known.
*   **Symbol Table:** Contains information about the symbols (functions and global variables) defined *within* this object module and symbols that are *referenced externally* by this module but defined elsewhere.
    *   Example: `min()`, `printf()`, `scanf()` functions with their addresses (which might be placeholders initially, or offsets).
    *   **Resolution:** The process of finding the actual memory addresses for these external symbols. This is done by the **linker**.
*   **Debugging Info:** Optional information used by debuggers, such as mappings from machine code back to source code lines, variable names, etc.

#### The Linker

The linker is a system program that takes one or more object modules generated by the assembler/compiler and combines them into a single executable file.

**Why is it needed?**
*   **Modular Programming:** Programs are often split into multiple source files (modules) for better organization and reusability. Each module is compiled/assembled separately into an object file. The linker brings them together.
*   **Library Usage:** Programs often use functions from standard libraries (e.g., `printf` from C standard library). The linker includes the necessary library code or resolves references to it.

**Linker Process (Typically Two Passes):**

**Phase I (Pass 1):**
1.  **Build Segment Table:** The linker scans all input object modules (A, B, C, D, E, etc., and system libraries). For each module, it determines the size of its code and data segments.
2.  **Build Global Symbol Table:** It collects all *definitions* of global symbols (functions and global variables) from all input object modules and their *relative addresses*. It notes down which module defines which symbol and its offset within that module.
    *   Example: If `object code A` calls `B, C, D, E`, and `object code B` calls `P, Q`. The linker will find the actual locations of `B, C, D, E` and `P, Q` (if defined in other modules or libraries) and record them.

**Phase II (Pass 2):**
1.  **Generate Executable Code:** The linker now combines the code and data segments from all object modules into a single, contiguous block of memory within the executable file.
2.  **Resolve Undefined References:** Using the global symbol table built in Phase I, the linker goes back through the code. For every external call (like `call printf()`) or reference to an external variable, it fills in the correct **relative address** (or a placeholder that the loader will later convert to an absolute address).
3.  **Perform Relocation:** Based on the relocation information in each object module, the linker adjusts all addresses that were relative to the start of their original modules, making them relative to the start of the *combined executable file*.

**Linker Responsibilities:**

1.  **Relocation:** Adjusting relative addresses within object modules to reflect their positions within the final combined executable file.
2.  **Symbol Resolution:** Finding the definitions for all external symbols referenced by modules and linking them correctly. This includes linking with library routines.
    *   **Stub (or Glue Code):** For dynamically linked libraries (DLLs/shared libraries), the linker might insert a small piece of code called a "stub" or "glue code." Instead of directly embedding the library function's code, this stub is a placeholder that, at runtime, will resolve to the actual library function's address if it's not already loaded. For example, for a `printf()` call, a stub might point to the dynamic linker which then finds the `printf` function in a shared library.

### The Loader

The loader is the final system software component involved in preparing a program for execution.

*   **Role:** The loader takes the **linked object code (the executable file)**, which is typically "relocatable" (meaning its addresses are relative to its own start), and places it into the main memory at an available starting address.
*   **Responsibilities:**
    1.  **Program Loading:** Loading the program's text and data segments from the executable file on disk into main memory.
    2.  **Relocation:** Performing the final adjustment of addresses from being relative to the executable file's start to being absolute physical addresses within RAM. This is done by adding the program's actual load address (base address) to all relative addresses.
    3.  **Symbol Resolution (if needed):** For dynamically linked libraries, the loader (or a dynamic linker invoked by the loader) will perform symbol resolution at run time.

**Relationship between Linker and Loader:**

*   **Linker:** Performs symbol resolution and relocation *to create a single executable file from multiple object files*. The output is typically **relocatable code**.
*   **Loader:** Performs program loading and *final relocation to absolute physical addresses* when placing the executable into RAM. For dynamic linking, it might also do run-time symbol resolution.


### Different 'Times' in Program Execution

The process of fixing addresses and resolving symbols can happen at different stages, influencing flexibility and performance:

1.  **Compile Time:**
    *   **Concept:** If the program's exact physical memory location is known *before compilation*, the compiler can generate absolute addresses directly.
    *   **Problem:** This is extremely inflexible. The program can only run at that specific address. If multiple programs need to run or if the memory layout changes, it won't work. This is rarely used in modern general-purpose OS.
    *   **Example from notes:** "Symbol constants -> relocatable address." This is a bit misleading here. It usually implies that if fixed, it's fixed *before* linking/loading.

2.  **Link Time:**
    *   **Concept:** The linker generates a relocatable executable file, but the addresses within this file are *relative* to the start of the executable. If the program is always loaded at the *same* starting address in memory, the linker can fix all addresses to be absolute.
    *   **Use case:** Simple embedded systems or very specific scenarios where the load address is fixed.
    *   **Example from notes:** "Relocatable code -> relocatable addresses ext. obj." Meaning, external object references are resolved to be relocatable within the final combined executable.

3.  **Load Time:**
    *   **Concept:** The linker produces a relocatable executable. The loader, when placing the program into memory, gets to know the program's *actual starting physical address*. It then performs the final **relocation** by adding this starting address to all the program's internal (relative) addresses.
    *   **Advantage:** Program can be loaded anywhere in memory. This is common for older systems or simpler executables.
    *   **Example from notes:** "Relocatable address -> absolute addresses."

4.  **Run Time (Dynamic Linking):**
    *   **Concept:** Relocation and symbol resolution can be postponed until the program is already running. This is typically done for **Dynamic Link Libraries (DLLs in Windows, Shared Objects/Libraries in Linux)**.
    *   **How it works:** When a program needs a function from a dynamic library, the linker doesn't embed the library's code. Instead, it places a stub. When the function is called for the first time, an OS component (the dynamic linker/loader) intervenes, finds the library in memory (or loads it if not present), and resolves the actual address of the function. Subsequent calls directly use the resolved address.
    *   **Advantages:**
        *   **Memory Saving:** Multiple programs can share a single copy of a library in memory.
        *   **Flexibility:** Library updates don't require recompiling or relinking all applications that use them.
        *   **Smaller Executables:** The executable file size is smaller as library code is not embedded.
    *   **Disadvantage:** Runtime overhead for resolving addresses.
    *   **Requirement:** Requires **OS Support** (e.g., dynamic linker).



### Memory Partitioning

So, yeah, basically because each process has to get memory, and since we are dealing with multiprogramming here, there would be multiple processes in the memory at once right, so obviously, we would have to divide the memory, so that each process gets allocated some space, now there are two ways of doing that.

### Contiguous Memory Allocation

The core idea of this is that, the entire process must reside in one continuous adjacent block of physical memory, this approach offers the advantage of speed and simplicity, however this simplicity has some issues.
And also there are two ways of implementing the above thing

#### Fixed Size Partitioning

Also known as static partitioning, this was one of the earliest methods for enabling multiprogramming. In this scheme, the main memory is divided into a number of partitions of a fixed size before any processes are executed. When a process arrives, it is loaded into a partition that is large enough to contain it. This approach is simple for the operating system to manage but suffers from severe drawbacks:

- **Limited Degree of Multiprogramming**: The number of processes that can run concurrently is strictly limited by the number of partitions created in memory. If all partitions are occupied, no new processes can start, regardless of their size or the amount of free space within those partitions.
    
- **Internal Fragmentation:**  A more critical issue is the waste of memory known as internal fragmentation. This occurs when the memory block allocated to a process is larger than what the process actually requires. The unused space within the allocated partition cannot be used by any other process and is therefore wasted.
  For instance, if a 5MB process is placed in a 10MB partition, 5MB of memory is lost to internal fragmentation.4 This waste is *internal* to the allocated block.


#### Variable Size Partitioning

To address the significant waste caused by internal fragmentation, the variable-size partitioning scheme (or dynamic partitioning) was developed. In this model, the operating system does not partition memory beforehand. Instead, it maintains a list of free blocks of memory and allocates a partition of the exact size needed by a process upon its arrival. This dynamic approach successfully eliminates internal fragmentation because a process is given only as much memory as it requests.

However, this flexibility introduces the need for the operating system to manage the free blocks and decide which one to allocate. This decision is governed by an allocation strategy. Common strategies include:

- **First-Fit**: The OS scans the list of free blocks (holes) and allocates the first one that is large enough to accommodate the process. This strategy is fast but may break up large blocks, leaving smaller, less useful ones behind.

- **Best-Fit**: The OS searches the entire list of holes to find the smallest one that is large enough for the process. This approach aims to preserve larger holes for future, larger processes. However, it is slower due to the exhaustive search and tends to create a large number of very small, often unusable holes.

- **Worst-Fit**: The OS allocates the largest available hole to the process. The rationale is that the leftover portion of the hole will be large enough to be useful for another process. However, this strategy can quickly eliminate large blocks needed for large processes.

- Next-fit: 


While variable partitioning solves the problem of internal fragmentation, its dynamic nature gives rise to a more insidious and system-crippling issue: external fragmentation.


### Data Structures for Dynamic Partitioning

See, obv there is a decent amount of overhead in dynamic partitioning, so to make it somewhat more efficient, we can use some data structures, there are two ways of implementing this method.

#### Bit-Map

*   **Concept:** The main memory is divided into fixed-size "allocation units" (e.g., 4 bytes, 1KB). A bit-map is then maintained, where each bit corresponds to an allocation unit.
    *   If a bit is `0`, the corresponding allocation unit is free.
    *   If a bit is `1`, the corresponding allocation unit is occupied.
*   **Diagram:**
    *   Imagine memory blocks (2MB, 4MB, 4MB, 4MB, 2MB) being pulled back from a process.
    *   Below, there are "Memory allocation units" (e.g., 8 units shown).
    *   A bit array `[000111100]` maps to these units:
        *   `000` (first 3 units) are free (Hole).
        *   `1111` (next 4 units) are occupied (Process).
        *   `00` (last 2 units) are free (Hole).

*   **Allocation Unit Size and Memory Waste:**
    *   If **1 bit** in the bitmap represents **4 bytes** of memory:
        *   This means for every 4 bytes of memory, we use 1 bit (1/8 of a byte) to track it.
        *   The overhead for the bitmap itself is (1/8) / 4 = 1/32 or roughly 3% of memory.
        *   **Problem:** If a process needs 5 bytes, and the allocation unit is 4 bytes, it will be allocated 8 bytes (two 4-byte units). This leads to internal fragmentation if the process size is not a multiple of the allocation unit size.
        *   Your notes state "fraction of memory used by bit map 1/33". This calculation implies 1 bit represents 4 bytes. (1 bit / (4 bytes * 8 bits/byte)) = 1/32 of memory. If it's 1/33, it might mean the total includes the bitmap itself: (1/32) / (1 + 1/32) approx 1/33.

*   **Impact of Allocation Unit Size:**
    *   **Large Allocation Unit:** (e.g., 4 KB)
        *   **Fewer bits required for the bitmap:** Bitmap is smaller, less memory overhead for the bitmap itself.
        *   **Increased space wastage (internal fragmentation):** A process might need 1 KB, but gets 4 KB, wasting 3 KB.
    *   **Small Allocation Unit:** (e.g., 4 bytes)
        *   **More bits required for the bitmap:** Bitmap is larger, more memory overhead for the bitmap.
        *   **Decreased space wastage (internal fragmentation):** A process needing 5 bytes gets 8 bytes, wasting 3 bytes, which is a smaller absolute waste than the previous example.
*   **Conclusion:** The choice of allocation unit size is a trade-off between the memory consumed by the bit-map and the internal fragmentation experienced. It's not widely used for dynamic partitioning as **linked lists** are often more flexible.
#### Linked List

   **Concept:** Maintain a linked list of free and allocated memory blocks. Each node in the list describes a memory segment (either a process `P` or a hole `H`).
*   **Node Structure:** Each node typically contains:
    *   **Type:** `P` (Process) or `H` (Hole)
    *   **Start Address:** The beginning memory address of the block.
    *   **End Address (or Size):** The ending memory address or the size of the block.
*   **Diagram:**
    *   Memory: OS, then blocks P (0-99), H (100-199), P (200-299), H (300-399).
    *   Linked List: `Head -> (P, 0, 99) -> (H, 100, 199) -> (P, 200, 299) -> (H, 300, 399)`
*   **Advantages of Linked List:**
    1.  **Memory Required is Less than Bit-Map:** Especially if memory is sparsely used, or if allocation units are small (making the bitmap huge). This is empirically found (as your notes state "found experimentally").
    2.  **Maintain List in Increasing Order of Starting Address:** This simplifies management, especially for:
        *   **Merging Adjacent Holes:** When a process frees its memory, the OS checks if the newly freed block is adjacent to an existing free hole. If so, they can be merged into a single, larger hole, reducing external fragmentation. This is much easier with a sorted list.
        *   **Deleting and Inserting:** Efficiently managing the list of blocks.
    3.  **Doubly Linked List is More Beneficial:** Allows for easier traversal in both directions, making merging of adjacent holes even more efficient (checking both preceding and succeeding blocks).

### Overlays

This is somewhat of an extension on contiguous memory allocation. What is the size of the program is greater than the size of the biggest partition (or available physical memory). The above methods would prevent such a program from running.

For this we use Overlays. Overlays is a program structuring technique. the program is divided into a root segment (which is always in memory) and overlays segment. Only one overlay segment from a particular level needs to be in memory at a particular time. When an overlay segment is needed, It is loaded into the memory area previously occupied by another overlay from the same level, hence overlaying it.
Common routines data that are used by multiple overlays or the root are kept in the root segment or a shared memory area

An overlay tree graphically represents the dependencies and structure of a program's overlays.
*   **Root:** The part of the program that is always in memory.
*   **Nodes:** Represent overlay segments.
*   **Branches:** Indicate that child nodes are mutually exclusive with their siblings (i.e., only one child from a set of siblings can be loaded at a time).
*   **Finding Minimum Memory:** To calculate the minimum physical memory required to run a program structured with overlays, you need to find the "heaviest path" from the root to any leaf node. This path represents the maximum amount of code that needs to be residing in main memory concurrently.
**read more about this shit, they seem to have asked a decent amount of questions about this**



### Non-Contiguous Memory Allocation

#### Paging

 The solution was to decouple a process's logical view of memory from its physical placement. Paging is the most fundamental and widely used non-contiguous memory allocation technique that achieves this, forming the bedrock of all modern virtual memory systems.
 
Paging directly attacks the root cause of external fragmentation, the use of variable-sized memory blocks, by imposing a uniform, fixed-size structure on both logical and physical memory. This allows a process's physical address space to be non-contiguous, meaning its constituent parts can be scattered throughout physical memory.
#### Pages and Frames

The paging model is built on two core concepts:

- **Frames**: Physical memory is divided into a series of fixed-size blocks called frames. The size of a frame is determined by the computer's hardware architecture and is always a power of two, with common sizes being 4KB, 16KB, or even 2MB or 1GB for **large pages**.

- **Pages**: A process's logical address space is also broken down into fixed-size blocks of the exact same size as frames. These logical blocks are called pages.

The central principle of paging is that any page from a process can be loaded into any available frame in physical memory. **The frames holding the pages of a single process do not need to be adjacent to one another**. The operating system simply maintains a list of all free frames and, when a process requiring
n pages needs to be loaded, it finds any n available frames wherever they may be located in physical memory and loads the pages into them.

This approach completely solves the problem of external fragmentation. Since all allocation units (frames) are of a fixed size, there are no variable-sized *holes* left behind when a process terminates. A frame is either in use or it is free; there is no unusable space between allocated blocks.

While paging masterfully eliminates external fragmentation, it does reintroduce internal fragmentation, albeit in a far more constrained and manageable form. This occurs because a process's total memory requirement is rarely an exact multiple of the page size. As a result, the final page of the process is typically only partially filled. However, this partially filled page must occupy an entire frame in physical memory. The unused space within this last frame constitutes internal fragmentation.

For example, consider a process that is 75KB in size with a system page size of 4KB. To store this process, the system will require 75/4=18.75 pages, which is rounded up to 19 pages. The first 18 pages will be full. The final, 19th page will contain only 75−(18×4)=3KB of data. When this page is loaded into a 4KB frame, 1KB of space within that frame is wasted.4

This level of waste is considered an acceptable trade-off. The maximum amount of internal fragmentation for any given process is always less than one full page size. This small, predictable loss of efficiency is vastly preferable to the catastrophic and unpredictable failures caused by external fragmentation in contiguous systems.

#### Logical And Physical Address Spaces

Paging formalizes the distinction between two different types of addresses:

- **Logical Address (or Virtual Address)**: This is the address generated by the CPU for every memory reference *(basically see, in one program, you may write a code, that would reference some other file, and this other file, or something, could be in some other memory space, this is what it means, when cpu generates an address, because to complete execution of this process, it needs, the data at this memory address)*. From the perspective of the running process, it sees a single, large, contiguous address space, typically starting at address 0 and extending up to its maximum size. The programmer and compiler work exclusively with these logical addresses.29

- **Physical Address:** This is the actual address of a location in the physical RAM hardware. The memory controller hardware uses physical addresses to access specific memory cells.

The core task of a paged memory management system is to translate every logical address generated by the CPU into a corresponding physical address before the memory is accessed.

#### [[Structure of an Address]]

The translation mechanism is elegantly implemented by interpreting the bit pattern of a logical address in a specific way. A logical address is split into two components.

- **Virtual Page Number:** The most significant bits of the logical address. This number serves as an index into a data structure called the page table.
    
- **Page Offset** The least significant bits of the logical address. This value represents the displacement, or offset, of the desired byte from the beginning of the page.
    

Similarly, a physical address is composed of a Physical Frame Number and a Frame Offset The key insight is that the offset is never translated; the location of a byte within a page is the same as its location within the frame that holds that page. Therefore, the offset bits from the logical address are carried over directly to the physical address.

#### The MMU

To overcome the rigidities of fixed memory allocation and ensure process isolation, hardware support is crucial. This is provided by the **Memory Management Unit (MMU)**, which is a hardware device.

*   **Logical Address (Virtual Address):** The address generated by the CPU. Processes generate logical addresses.
*   **Physical Address (Real Address):** The actual address in main memory. The MMU translates logical addresses into physical addresses.

**How MMU works with Relocation and Limit Registers:**

The MMU uses two special registers for address translation and protection in simple contiguous allocation schemes:

1.  **Relocation Register (Base Register):**
    *   **Purpose:** Stores the starting physical address (base address) where the current process is loaded in main memory.
    *   **Mechanism:** When the CPU generates a logical address, the MMU *adds* the value from the Relocation Register to it to produce the corresponding physical address.
        *   `Physical Address = Logical Address + Relocation Register Value`
    *   **Ensures Relocation:** This allows a program to be loaded anywhere in physical memory. The logical addresses within the program remain unchanged; only the Relocation Register's value changes, making the program *relocatable* at load time or even runtime.

2.  **Limit Register:
    *   **Purpose:** Stores the *size* (or limit) of the current process's logical address space.
    *   **Mechanism:** Before the MMU performs the addition for relocation, it first checks if the generated `Logical Address` is *less than* the value in the Limit Register (`Logical Address < Limit Register`).
    *   **Ensures Protection:**
        *   If `Logical Address < Limit Register`: The address is valid and within the process's allowed memory space. Translation proceeds.
        *   If `Logical Address >= Limit Register`: The address is *outside* the process's bounds. This indicates an illegal memory access attempt (e.g., trying to access another process's memory or OS memory). The MMU traps to the OS, which usually terminates the offending process.
    *   **Example from notes:**
        *   `RR = 100K`, `Limit = 100` (assuming units like 100KB, and 100 words/units).
        *   If CPU generates `Logical Address = 50K`
            *   Is `50K < 100K`? Yes. (Protection Check)
            *   `Physical Address = 50K + 100K = 150K`.
    *   **Important:** Both the Relocation Register and Limit Register are **privileged registers**, meaning only the Operating System (in kernel mode) can modify their contents. This prevents user processes from bypassing memory protection mechanisms.

> the above thing is how it would work in a contiguous memory management system (its just how it works in general, not in reference to paging). the below thing is how it would work in paging.

### [[Word and Address Space]]

^3b6d52

- **Bit**: The smallest unit of data in computing. It can be either 0 or 1.
- **Byte**: A group of 8 bits. It is the smallest addressable unit in most computer architectures.
- **Word**: A unit of data that the CPU processes at once. Its size depends on the system architecture — typically 2, 4, or 8 bytes.  
The smallest addressable unit in most architectures is a **byte**, not a word.
#### Physical Address Space (PAS)


The total range of physical addresses that the hardware can use to access the physical memory (RAM). It is essentially the size of the main memory.
Assume:  
Memory size = 128 MB  
Word size (WS) = 4 bytes
*and we need to find how many bits we need to represent all the words in the physical memory??*

**Convert MB to bytes**
1 MB = 2¹⁰ × 2¹⁰ = 2²⁰ bytes  
128 MB = 128 × 2²⁰ = 2⁷ × 2²⁰ = 2²⁷ bytes
So, the total physical memory = 2²⁷ bytes

**Convert bytes to words**
Word size = 4 bytes = 2² bytes  
Number of words = Total bytes / Bytes per word  
= 2²⁷ / 2² = 2²⁵ words
So, physical memory contains 2²⁵ words

**Address bits needed**
To uniquely address 2²⁵ words, we need:
log₂(2²⁵) = 25 bits

So, the physical address must be 25 bits long.
#### Logical Address Space (LAS)  
The total range of logical addresses that a process can generate. This is equal to the size of the process (i.e., the virtual memory allocated to it). Each process has its own LAS.
*same question as before, how many bits, do we need to represent all the addresses in the logical address space??*
Assume:  
Logical address space = 256 MB  (this is basically the size of the process)
Word size = 4 bytes

**Convert MB to bytes**
256 MB = 2⁸ × 2¹⁰ × 2¹⁰ = 2²⁸ bytes (try to write everything in powers of 2)

**Convert bytes to words**
Word size = 4 bytes = 2² bytes  
Number of words = 2²⁸ / 2² = 2²⁶ words

**Address bits needed**
To uniquely address 2²⁶ words, we need:
log₂(2²⁶) = 26 bits

So, the logical address must be 26 bits long.

| Term                      | Value   | Explanation                  |
| ------------------------- | ------- | ---------------------------- |
| Physical memory (PAS)     | 128 MB  | Total RAM size               |
| Word size (WS)            | 4 bytes | One word = 4 bytes           |
| Words in PAS              | 2²⁵     | Total addressable words      |
| Bits for physical address | 25 bits | log₂(2²⁵)                    |
| Logical memory (LAS)      | 256 MB  | Virtual memory for a process |
| Words in LAS              | 2²⁶     | Total virtual words          |
| Bits for logical address  | 26 bits | log₂(2²⁶)                    |

### The Relations

Basically everything, that we learnt till now, has to be connected, we basically need to able to solve all the problems related to paging, and for that, we need to actually know how the translation happens from virtual address to physical address.

alright so first off, the size of one page should be equal to the size of one frame.
and because we are dealing with the sizes of memory here, we can either say byte, or even better, say word. 
`page size = frame size = P words`

now that we have this sorted, we already discussed the structure of both addresses [[#^8889de| structure]]


now let us try to find how many bits, would the address need to be, obv, we already found the number of bits, required to represent all the addresses [[#^3b6d52|representation]], but we need to know how many bits to assign to each part, cause there are two parts.
So, a logical address would have the page number and the page offset, and this offset value would be same in both the logical address and the physical address.
this offset basically tells, the position inside a page, or a frame. now this is basic common sense, as we saw before. **the number of bits, we need would be log2 of the size of the page or the frame (in bytes)**

Lets assume the size of a page/frame is 4 KB
lets convert this to bytes
1 KB = $2^{10}$ bytes
4 KB = $2^{12}$ bytes
>You can convert this to words also, if you have know 1 word has how many bytes
### All The Formulae for the above thing
#### Physical Address Space (PAS)
- Main Memory Size = $M$ words  
- Number of bits in Physical Address (PA) = $\log_2(M)$ bits  
  Let this be $m$ bits
#### Logical Address Space (LAS)
- Process Size = $L$ words  
- Number of bits in Logical Address (LA) = $\log_2(L)$ bits  
  Let this be $l$ bits
#### Page Size (PS)
- Page Size = $P$ bytes (or words)  
- Number of bits for offset = $\log_2(P)$  
  Let this be $p$ bits
#### Address Structure

^8889de

**Logical Address (LA):**
- Total = $l$ bits  
- Structure:  
  $LA = \text{Page Number (}l - p\text{ bits)} \; \| \; \text{Offset (}p\text{ bits)}$

**Physical Address (PA):**
- Total = $m$ bits  
- Structure:  
  $PA = \text{Frame Number (}m - p\text{ bits)} \; \| \; \text{Offset (}p\text{ bits)}$
#### Number of Pages
- $\text{Number of Pages} = \frac{\text{LAS}}{\text{Page Size}} = \frac{L}{P} = 2^{l - p}$
#### Number of Frames
- $\text{Number of Frames} = \frac{\text{PAS}}{\text{Page Size}} = \frac{M}{P} = 2^{m - p}$
#### Page Table Entries
- One entry per page  
- $\text{Number of Entries} = 2^{l - p}$
#### Size of Each Page Table Entry (PTE)
- Each PTE stores the Frame Number: $(m - p)$ bits  
- Plus control bits (Present/Absent, Modified, etc.)  
- Let total PTE size = $e$ bits or bytes
#### Total Size of Page Table
- $\text{Page Table Size} = 2^{l - p} \times e$ (in bits or bytes)

### Structure of a Page Table Entry (PTE)

A page table is an array of Page Table Entries (PTEs). Each PTE holds not just the translation information but also a collection of control bits that enable advanced memory management features like protection and virtual memory.

|                             |                                                                                                                                                    |                                                                                                                                                                                                                                                                                                                                                                   |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Field                       | Description                                                                                                                                        | Purpose and Significance                                                                                                                                                                                                                                                                                                                                          |
| Physical Frame Number (PFN) | The base address of the physical frame in RAM where the corresponding virtual page is located.                                                     | This is the core data required for the address translation itself.20                                                                                                                                                                                                                                                                                              |
| Present/ Valid Bit          | A single bit indicating whether the page is currently in physical memory (1 = valid/present) or resides on secondary storage (0 = invalid/absent). | This bit is fundamental to implementing virtual memory. If a process accesses a page marked as invalid, it triggers a page fault, signaling the OS to load the page from disk.20                                                                                                                                                                                  |
| Protection Bits             | A set of bits specifying the access permissions for the page (e.g., Read, Write, Execute).                                                         | These bits enable memory protection. On every memory access, the hardware checks these permissions. An attempt to perform a forbidden operation (like writing to a read-only page) causes a trap to the OS, which typically terminates the offending process.21                                                                                                   |
| Modified / Dirty Bit        | A single bit that is automatically set to 1 by the hardware whenever a write operation is performed on any byte within the page.                   | This bit is crucial for optimizing page replacement in a virtual memory system. If a page's dirty bit is 0 (it is "clean"), it means its contents in memory are identical to its copy on disk. It can be replaced without being written back. If the bit is 1 (it is "dirty"), the OS must write the modified page back to disk before its frame can be reused.34 |
| Referenced / Accessed Bit   | A single bit that is automatically set to 1 by the hardware whenever the page is accessed (either for a read or a write).                          | This bit is used by page replacement algorithms to determine which pages are actively being used. The OS can periodically clear these bits to track recent usage patterns, which is essential for algorithms like Least Recently Used (LRU) approximations.34                                                                                                     |
| Caching Disabled Bit        | A single bit that tells the hardware whether the contents of this page should be cached by the CPU's data caches.                                  | This is critical for pages that are mapped to device control registers (memory-mapped I/O), where the values can change asynchronously. Caching such pages would lead to stale data.                                                                                                                                                                              |

**CPU to Physical Memory Access with Page Table:**
1.  CPU generates `Logical Address (PNO, Offset)`.
2.  MMU accesses a special register (e.g., **PT-Start Base Register**) which holds the physical base address of the current process's page table.
3.  MMU calculates the address of the specific PTE: `PTE Address = PT-Start Base Register + (PNO * Size of PTE)`.
4.  MMU fetches the PTE from main memory.
5.  MMU checks the Present/Absent bit.
    *   If Absent: Page Fault. OS intervenes.
    *   If Present: MMU extracts the `Frame Number` from the PTE.
6.  MMU forms the `Physical Address = (Frame Number, Offset)`.
7.  MMU accesses the data/instruction at this `Physical Address` in Main Memory.


### Virtual Memory

Virtual Memory is a memory management capability of an operating system, that creates the illusion for a process that it has a large, private, and contiguous block of memory (called an address space), when in reality its physical memory may be non-contiguous and even partially stored on secondary storage.

   **Advantages**
    1.  **Freedom from Physical Memory Constraints:** Programs can be larger than the physical RAM available.
    2.  **Process Isolation:** Each process gets its own virtual address space, preventing it from interfering with other processes. This is a cornerstone of modern OS security and stability.
    3.  **Efficient Process Creation:** The OS can create processes faster because it only needs to load the necessary parts of the program into memory to start, a concept called **Demand Paging**.

### Performance Issues with Paging

The biggest drawback of the basic paging scheme is its performance, for every single memory access a program makes, the system actually has to perform two memory accesses.
First to the main memory the access the page number and get the frame number for the actual physical address. and then again to the main memory to access the actual data or instruction at the physical address.

**The Effective Access Time (EAT)** is double, `EAT = m (for page table) + m (for data) = 2m`


Now assume we increase the physical address space, assume its a 64-bit system
#### The Performance Bottleneck and the TLB
This hardware-based translation introduces a major performance bottleneck. Since page tables are stored in main memory, a naive MMU implementation would need to perform at least one extra memory access for each program memory access (and more for multi-level page tables).21 This would, at a minimum, double the effective memory access time, slowing the system to an unacceptable crawl.

To overcome this bottleneck, the MMU includes a small, extremely fast, hardware cache called the Translation Lookaside Buffer (TLB).21 The TLB is an associative cache that stores a small number of recently used

VPN -> PFN mappings. It is more accurately described as an "address-translation cache".42 The interaction between the MMU and the TLB is as follows:

- TLB Hit: When the MMU receives a virtual address, it first checks the TLB to see if it holds the translation for the given VPN. Because the TLB is implemented in specialized, fast hardware, this check is performed in parallel and is extremely quick (often in a single clock cycle). If the entry is found (a TLB hit), the PFN is retrieved directly from the TLB, and the page table in main memory is not accessed at all. The physical address is formed, and the memory access proceeds with almost no overhead. Due to the principle of locality (programs tend to access the same memory regions repeatedly), TLB hits are the common case, with hit rates often exceeding 99%.42
    
- TLB Miss: If the VPN is not found in the TLB (a TLB miss), the hardware must find the translation the slow way. The MMU (in hardware-managed systems) initiates a "page walk" by accessing the page table(s) in main memory to retrieve the correct PTE. Once the PFN is found, the VPN -> PFN mapping is installed in the TLB, often evicting an existing entry according to a replacement policy like LRU. The instruction that caused the miss is then restarted. This time, the translation will be found in the TLB, resulting in a hit.42
#### Context Switching and the TLB

A critical implication of the TLB is its interaction with process context switching. Since each process has its own page table, the translations cached in the TLB are specific to the currently running process. When the operating system switches context to a different process, the contents of the TLB become invalid for the new process. The most straightforward solution is to flush the TLB—invalidate all its entries—on every context switch. The new process then starts with an empty TLB and will incur a series of TLB misses until its working set of translations is cached. This TLB flush is a significant component of the overhead associated with context switching.45 To mitigate this, some advanced architectures include an

Address Space Identifier (ASID) in each TLB entry, which allows the TLB to hold entries for multiple processes simultaneously, avoiding the need for a full flush.37

The intricate dance between the OS, the MMU, and the TLB highlights a deep, symbiotic relationship. The operating system, as a software entity, defines the high-level policy of memory management: it creates and manages the page tables, decides where pages are placed, and handles exceptions. The hardware, in the form of the MMU and TLB, provides the low-level mechanism to execute these policies with extreme efficiency. The software defines a powerful but potentially slow abstraction (paging), and the hardware accelerates its core operations. When this acceleration itself introduces a new bottleneck (page table lookups), even more specialized hardware (the TLB) is introduced to solve it. This iterative refinement through hardware-software co-design is a central theme in the development of modern


