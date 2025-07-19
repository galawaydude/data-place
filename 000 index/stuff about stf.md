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
and because we are dealing with the sizes of memory here, we can either say byte, or even better say word. `page size = frame size = P words`

now that we have this sorted, we already discussed the structure of both the addresses [[add a link to the thing here, the link to the structur of an address here]]

So, a logical address would have the page number and the page offset, and this offset value would be same in both the logical address and the physical address.
this offset basically tells, the position inside a page, or a frame. now this is basic common sense, as we saw before. the number of bits, we need would be log2 of that things size.



**Page Size and Offset Bits**

*   **Relationship:** `Page Size  Frame Size = P words`.
*   The `Offset` part of an address indicates the position *within* a page.
*   **Number of bits required for Offset:** `log2(Page Size in bytes)`.
    *   If Page Size is `P` bytes, then `log2(P)` bits are needed for the offset.
    *   Example: `Page Size = 4 KB`, `Word Size = 4 B`
        *   `4 KB = 4 * 1024 B = 2^2 * 2^10 B = 2^12 B`
        *   Number of words per page = `4 KB / 4 B = 1024 words = 2^10 words`.
        *   Number of bits for Offset = `log2(4KB) = log2(2^12) = 12 bits`. (This is the offset in bytes)
        *   If the offset is in *words*, it would be `log2(1024) = 10 bits`. The notes show 10 bits. This means the offset refers to words, not bytes. It's usually bytes.
        *   **Standard convention:** Offset refers to byte address within the page. So for 4KB page, it's 12 bits.

The Page Table is a crucial data structure for address translation in a paging system.

*   **Purpose:** It maps a process's logical pages to physical frames.
*   **Location:** The Page Table itself resides in **main memory**.
*   **Size:** The size of the Page Table depends on:
    *   The **number of pages** in the logical address space.
    *   The **size of each Page Table Entry (PTE)**.

**Calculations & Formulas:**

*   **Physical Address Space (PAS):** Main Memory Size = `M` words. Physical Address (PA) = `log2(M)` bits (let's call it `m` bits).
*   **Logical Address Space (LAS):** Process Size = `L` words. Logical Address (LA) = `log2(L)` bits (let's call it `l` bits).
*   **Page Size (PS):** `P` bytes (or words). Offset needs `log2(P)` bits (let's call it `p` bits).

**Address Structure:**
*   **Logical Address (LA):** `l` bits total.
    *   `LA = (Page Number (l-p bits), Page Offset (p bits))`
*   **Physical Address (PA):** `m` bits total.
    *   `PA = (Frame Number (m-p bits), Frame Offset (p bits))`

**Number of Pages:**
*   `Number of Pages = LAS / PS = L words / P words = 2^(l-p)` pages.

**Number of Frames:**
*   `Number of Frames = PAS / PS = M words / P words = 2^(m-p)` frames.

**Page Table Entries:**
*   The Page Table will contain one entry for each page in the process's logical address space.
*   **Number of entries in Page Table = Number of Pages = 2^(l-p)**.

**Size of Each Page Table Entry (PTE):**
*   Each entry needs to store the `Frame Number`. The Frame Number is `m-p` bits long.
*   Additionally, each PTE contains control bits (Present/Absent, Modified, Referenced, Protection bits, etc.). Let `e` be the total size of an entry in bits or bytes.
*   So, size of PTE = `(m-p) bits + control bits`. (Your notes denote this as `e` bytes or `m-p` bits directly if only frame no. is considered).

**Total Size of Page Table:**
*   `Size of Page Table = Number of Entries * Size of Each Entry`
*   `Size of Page Table = 2^(l-p) * e (bits or bytes)`

**Example (from notes):**
*   `LA = 27 bits` (So, `l=27`)
*   `PA = 20 bits` (So, `m=20`)
*   `Poff (Offset) = 12 bits` (So, `p=12`)

*   **Calculate Page Number bits:** `l - p = 27 - 12 = 15 bits`.
*   **Calculate Frame Number bits:** `m - p = 20 - 12 = 8 bits`.

*   **Number of entries in Page Table:** `2^(l-p) = 2^15` entries.
*   **Size of each Page Table Entry:** Needs to store Frame Number (8 bits) + control bits. If `e` is assumed to be 8 bits (1 byte) just for the Frame number:
*   **Size of Page Table = `2^15 entries * 8 bits/entry = 2^15 * 1 byte = 32 KB`.**
