### Word and Address Space

**Word:**  A unit of data that the CPU processes at once. Its size depends on the system architecture and is typically 2, 4, 8, bytes, The smallest addressable unit in most architectures is a byte.

> A **bit** is the smallest unit of data in computing, its 1 or 0
> A **byte** is made up of 8 bits, its the smallest addressable unit in memory

**Physical Address Space (PAS):** The total range of physical addresses that the hardware can use to access the physical memory (RAM. Its basically the size of the main memory.

Assume, we have a memory which is 128 MB, lets convert this to bytes first, cause that is the standard unit.
**make this conversion thing please**
1 MB = 2^10 × 2^10 = 2^20 bytes
So, 128 MB = 128 × 2^20 bytes = 2^7 × 2^20 = 2^27 bytes

So, the entire memory is 2^27 bytes.
and assume that 1 word is 4 bytes, this basically means, that at once, the cpu can process 4 bytes of data at once.

So, basically if i have 2^27 bytes, and each word is 4 bytes, then how many total words do i have

number of words = total bytes / bytes per word. = 2^25 words, meaning there are these many words in the physical memory, so basically what we are trying to answer here, s how many bits are needed to uniqurly address each word in physical memory


**Logical Address Space (LAS):** The total range of logical addresses that a process can generate. This is equal to the size of the process (or virtual memory space allocated to it).
    *   If Process Size = `L` words, then `Logical Address (LA)` requires `log2(L)` bits.
    *   Example: `LAS = 256 MB`, `WS = 4 B`
        *   `256 MB = 2^8 * 2^10 * 2^10 B = 2^28 B`
        *   If `WS = 4 B = 2^2 B`, then `256 MB` contains `2^28 B / 2^2 B = 2^26` words.
        *   Therefore, `LA` (logical address) needs `log2(2^26)` = **26 bits**.




### The Relations

Basically everything, that we learnt till now, has to be connected, we basically need to able to solve all the problems related to paging, and for that, we need to actually know how the translation happens from virtual address to physical address.

alright so first off, the size of one page should be equal to the size of one frame.
and because we are dealing with the sizes of memory here, we can either say byte, or even better say word.

`page size = frame size = p words`