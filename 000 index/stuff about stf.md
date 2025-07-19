### Word and Address Space

**Word:**  A unit of data that the CPU processes at once. Its size depends on the system architecture and is typically 2, 4, 8, bytes, The smallest addressable unit in most architectures is a byte.

> A **bit** is the smallest unit of data in computing, its 1 or 0
> A **byte** is made up of 8 bits, its the smallest addressable unit in memory

**Physical Address Space (PAS):** The total range of physical addresses that the hardware can use to access the physical memory (RAM. Its basically the size of the main memory.

Assume, we have a memory which is 128 MB, lets convert this to bytes first, cause that is the standard unit.

1 MB = 1024 × 1024 bytes = 2^20 bytes
So, 128 MB = 128 × 2^20 bytes = 2^7 × 2^20 = 2^27 bytes

*   **Logical Address Space (LAS):** The total range of logical addresses that a process can generate. This is equal to the size of the process (or virtual memory space allocated to it).
    *   If Process Size = `L` words, then `Logical Address (LA)` requires `log2(L)` bits.
    *   Example: `LAS = 256 MB`, `WS = 4 B`
        *   `256 MB = 2^8 * 2^10 * 2^10 B = 2^28 B`
        *   If `WS = 4 B = 2^2 B`, then `256 MB` contains `2^28 B / 2^2 B = 2^26` words.
        *   Therefore, `LA` (logical address) needs `log2(2^26)` = **26 bits**.