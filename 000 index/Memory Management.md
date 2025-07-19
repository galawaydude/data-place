
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
The probability of n processes all waiting for I/O (assuming independence) is `p * p * ... (n times) = p^n`.

Therefore, the probability that at least one process is *not* waiting for I/O (i.e., at least one process is ready to use the CPU) is `1 - p^n`.

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

This is also c

