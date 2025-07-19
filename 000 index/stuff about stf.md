### Word and Address Space

**Word:**  A unit of data that the CPU processes at once. Its size depends on the system architecture and is typically 2, 4, 8, bytes, The smallest addressable unit in most architectures is a byte.

> A **bit** is the smallest unit of data in computing, its 1 or 0
> A **byte** is made up of 8 bits, its the smallest addressable unit in memory

**Physical Address Space (PAS):** The total range of physical addresses that the hardware can use to access the physical memory (RAM),
    *   If Main Memory Size = `M` words, then `Physical Address (PA)` requires `log2(M)` bits.
    *   Example: `PAS = 128 MB`, `WS = 4 B` (Word Size = 4 Bytes)
        *   `128 MB = 128 * 1024 * 1024 B = 2^7 * 2^10 * 2^10 B = 2^27 B`
        *   If `WS = 4 B = 2^2 B`, then `128 MB` contains `2^27 B / 2^2 B = 2^25` words.
        *   Therefore, `PA` (physical address) needs `log2(2^25)` = **25 bits**.

*   **Logical Address Space (LAS):** The total range of logical addresses that a process can generate. This is equal to the size of the process (or virtual memory space allocated to it).
    *   If Process Size = `L` words, then `Logical Address (LA)` requires `log2(L)` bits.
    *   Example: `LAS = 256 MB`, `WS = 4 B`
        *   `256 MB = 2^8 * 2^10 * 2^10 B = 2^28 B`
        *   If `WS = 4 B = 2^2 B`, then `256 MB` contains `2^28 B / 2^2 B = 2^26` words.
        *   Therefore, `LA` (logical address) needs `log2(2^26)` = **26 bits**.