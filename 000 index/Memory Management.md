
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
5.  **Object Code (Object Modules):** Machine-readable code containing instructions and data, but typically with "placeholder" addresses for external references. These are often referred to as relocatable object files.
6.  **Linker:** Combines multiple object modules (and libraries) into a single **executable file**. It resolves external symbol references and performs relocation.
7.  **Loader:** Loads the executable file into main memory, assigning actual physical addresses, so the CPU can execute it.



