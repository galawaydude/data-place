
## **Process Synchronization

When multiple processes or threads need to interact with each other, they often do so by **communicating** or **competing** for shared resources. A common way for processes to communicate is by using **shared memory**. However, this introduces a significant challenge.

> I would assume this thing is for the sake of simplicity

So, there is a shared memory, which as the name suggests is a memory shared by all processes, this is said explicitly, cause we discussed before that every process has its own, resources, and information, but if processes want to communicate with each other, and basically want to work on the same data, the data needs to be public, that is basically in the shared memory (what about other resources, how does that, work, like is everything that is supposed to be shared brough to the ram, i have to check this out)

Now, obviously, processes, are programs that are running, and programs are basically a set of instructions, now we know, in most cases, shit is preemptive, so basically, its not that, the entire process is executed at once, it may be preempted. now this is where shit gets bad, before it gets preempted its state gets saved. so, assume some variables which get saved in this state, get saved, and then they are changed by some other process, this one has no idea right. So basically there needs to be some synchronization between processes.

#### **The Problem of Data Inconsistency (Race Condition)

Consider a scenario where two processes, Process 1 (P1) and Process 2 (P2), both access a shared variable called `count`. Imagine `count` is initially `5`.

*   **P1's Goal:** Increment `count` (`count++`)
*   **P2's Goal:** Decrement `count` (`count--`)

Ideally, if `count` starts at `5`, after P1 increments it and P2 decrements it, the final value should be `5` again. Let's break down how `count++` and `count--` might be translated into machine-level instructions:

**For `count++` (P1):**
1.  `MOV R0, count` (Load the value of `count` into register `R0`)
2.  `INC R0` (Increment the value in `R0`)
3.  `MOV count, R0` (Store the value from `R0` back into `count`)

**For `count--` (P2):**
1.  `MOV R1, count` (Load the value of `count` into register `R1`)
2.  `DEC R1` (Decrement the value in `R1`)
3.  `MOV count, R1` (Store the value from `R1` back into `count`)

Now, let's trace different execution orders (interleaving) of these instructions, assuming `count` starts at `5`:

##### Scenario 1: No Preemption (Ideal Order)

**Order 1: P1 executes completely, then P2 executes completely.**

| Step | Process | Instruction             | `count` | `R0` (P1) | `R1` (P2) | Explanation                                 |
| :--- | :------ | :---------------------- | :------ | :-------- | :-------- | :------------------------------------------ |
| 1    |         | Initial state           | 5       | -         | -         | `count` is 5.                               |
| 2    | P1      | `MOV R0, count`         | 5       | 5         | -         | P1 loads 5 into R0.                         |
| 3    | P1      | `INC R0`                | 5       | 6         | -         | P1 increments R0 to 6.                      |
| 4    | P1      | `MOV count, R0`         | 6       | 6         | -         | P1 stores 6 back to `count`.                |
| 5    | P2      | `MOV R1, count`         | 6       | 6         | 6         | P2 loads 6 into R1.                         |
| 6    | P2      | `DEC R1`                | 6       | 6         | 5         | P2 decrements R1 to 5.                      |
| 7    | P2      | `MOV count, R1`         | 5       | 6         | 5         | P2 stores 5 back to `count`.                |

**Result:** `count` ends at **5**. This is a **consistent** result. This happens when there is no **preemption** (one process finishes its critical work before another starts).

#### Scenario 2: With Preemption (Inconsistent Results)

**Order 2: P1 starts, gets preempted, P2 executes completely, P1 resumes.**

| Step | Process | Instruction             | `count` | `R0` (P1) | `R1` (P2) | Explanation                                 |
| :--- | :------ | :---------------------- | :------ | :-------- | :-------- | :------------------------------------------ |
| 1    |         | Initial state           | 5       | -         | -         | `count` is 5.                               |
| 2    | P1      | `MOV R0, count`         | 5       | 5         | -         | P1 loads 5 into R0.                         |
| 3    | P1      | `INC R0`                | 5       | 6         | -         | P1 increments R0 to 6.                      |
| **4**| **P2**  | **`MOV R1, count`**     | **5**   | 6         | **5**     | **P1 is preempted.** P2 loads current `count` (which is still 5) into R1. |
| 5    | P2      | `DEC R1`                | 5       | 6         | 4         | P2 decrements R1 to 4.                      |
| 6    | P2      | `MOV count, R1`         | 4       | 6         | 4         | P2 stores 4 back to `count`.                |
| **7**| **P1**  | **`MOV count, R0`**     | **6**   | 6         | 4         | **P2 is preempted.** P1 resumes, using its *stale* R0 value (6) to update `count`. |

**Result:** `count` ends at **6**. This is an **inconsistent** result.

**Order 3: P1 starts, P2 starts, P1 continues, P2 continues, etc.**

| Step | Process | Instruction             | `count` | `R0` (P1) | `R1` (P2) | Explanation                                 |
| :--- | :------ | :---------------------- | :------ | :-------- | :-------- | :------------------------------------------ |
| 1    |         | Initial state           | 5       | -         | -         | `count` is 5.                               |
| 2    | P1      | `MOV R0, count`         | 5       | 5         | -         | P1 loads 5 into R0.                         |
| 3    | P2      | `MOV R1, count`         | 5       | 5         | 5         | P2 loads 5 into R1. (Both have stale data)  |
| 4    | P1      | `INC R0`                | 5       | 6         | 5         | P1 increments R0 to 6.                      |
| 5    | P2      | `DEC R1`                | 5       | 6         | 4         | P2 decrements R1 to 4.                      |
| 6    | P1      | `MOV count, R0`         | 6       | 6         | 4         | P1 stores 6 to `count`.                     |
| 7    | P2      | `MOV count, R1`         | 4       | 6         | 4         | P2 stores 4 to `count`. (Overwrites P1's update) |

**Result:** `count` ends at **4**. This is also an **inconsistent** result.

Now, basically what you saw there is a race condition, 
a race condition is basically a condition, when the outcome of different processes accessing shared date, depends on the relative order of their execution.

### **The Critical Section Problem

See, now as I said before, every process is basically a code being executed. now its not the case that, all the instructions in the code, are accessing the shared space, some instructions may be working on the resources allocated to that particular process only right. The main issue of syncing comes, when the critical section part is being executed.
> A **Critical Section** is a segment of code where processes access shared resources (data, files, devices). Only one process or thread should be allowed to execute its critical section at any given time to ensure data consistency.

So, basically to solve this entire problem of race condition, and sync, our main goal is to create a protocol, that keeps track of these important sections of a code

*   **Entry Section:** The code requesting permission to enter the critical section.
*   **Critical Section (CS):** The code that accesses the shared resources.
*   **Exit Section:** The code executed after a process finishes its critical section, releasing its hold on the shared resources.
*   **Remainder Section (NCS):** The rest of the code in the process, not involving shared resources.

And any solution that, we make should be able to satisfy the below criteria:

#### Primary Requirements:

1.  **Mutual Exclusion:**
    *   **Definition:** If one process is executing in its critical section, then no other process is allowed to be executing in its critical section.
    *   **Importance:** This is the absolute minimum requirement to prevent race conditions and ensure data consistency.

2.  **Progress:**
    *   **Definition:** If no process is executing in its critical section and some processes want to enter their critical section, then only those processes that are not in their remainder section can participate in deciding which process will enter the critical section next. This selection cannot be postponed indefinitely.
    *   **Importance:** Prevents deadlocks. If a critical section is free, processes waiting to enter should eventually be allowed in. Processes that have no interest in entering the critical section should not block others from entering.
    > to explain the above thing in a simple way, what the guy means is that, like assume p1, p2, p3 are three processes, and p3 has finished all of its, cs section, and is now in its rs, and p1 and p2, need to execute its cs. so basically in this fight to decide who gets to do it first, p3 should not be playing any role, this should be pretty obvious. but this is pretty important. what this means is that, assume p3 was 

#### Secondary Requirements:

3.  **Bounded Waiting:**
    *   **Definition:** There must be a limit on the number of times that other processes are allowed to enter their critical sections after a process has made a request to enter its critical section and before that request is granted.
    *   **Importance:** Prevents starvation. It ensures that every process that wants to enter its critical section will eventually get a chance to do so, preventing infinite waiting.

4.  **Portability / Architectural Neutrality:**
    *   **Definition:** The synchronization solution should not be dependent upon specific hardware features or operating system functionalities. It should be able to run on all platforms.
    *   **Importance:** A solution that works only on a particular CPU architecture or OS is not generally useful. Ideally, solutions should be implementable in software, making them platform-independent.

