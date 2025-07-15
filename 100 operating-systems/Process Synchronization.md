
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
    > to explain the above thing in a simple way, what the guy means is that, like assume p1, p2, p3 are three processes, and p3 has finished all of its, cs section, and is now in its rs, and p1 and p2, need to execute its cs. so basically in this fight to decide who gets to do it first, p3 should not be playing any role, this should be pretty obvious. but this is pretty important. what this means is that, assume p3 was using cs, so after its done, it should relinquish, its ability to use the cs, cause if it still holds the power to be in the cs, even if it does not need, it, shit would be bad right. 

#### Secondary Requirements:

3.  **Bounded Waiting:**
    *   **Definition:** There must be a limit on the number of times that other processes are allowed to enter their critical sections after a process has made a request to enter its critical section and before that request is granted.
    *   **Importance:** Prevents starvation. It ensures that every process that wants to enter its critical section will eventually get a chance to do so, preventing infinite waiting.

4.  **Portability / Architectural Neutrality:**
    *   **Definition:** The synchronization solution should not be dependent upon specific hardware features or operating system functionalities. It should be able to run on all platforms.
    *   **Importance:** A solution that works only on a particular CPU architecture or OS is not generally useful. Ideally, solutions should be implementable in software, making them platform-independent.


Alright, so now that we know what to keep in mind to design a solution, basically any solution that we design, can be classified into two types.

### **Busy Waiting and Non-Busy Waiting

Basically, like assume P1, needs to use its cs, but someone else is using it, so either this thing can still have the cpu, and keep on checking (Busy Waiting), or it can free the cpu, and just wait (Non-Busy Waiting)

| Feature            | With Busy Waiting (Spinlocks)                                                                    | Without Busy Waiting (Blocking/Sleeping)                                                         |
| :----------------- | :----------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------- |
| **Mechanism**      | Process continuously checks a condition in a loop.                                               | Process gives up the CPU and goes into a waiting state.                                          |
| **CPU Usage**      | Wastes CPU cycles while waiting, as the CPU is "busy" doing nothing useful.                      | Does not waste CPU cycles while waiting, as the process is suspended.                            |
| **Context Switch** | No context switch required to wait.                                                              | Requires context switch to move process to waiting queue and back.                               |
| **When used**      | For very short critical sections, or on multi-core systems where other cores can do useful work. | For longer critical sections, or on single-core systems where busy waiting would stall progress. |
| **Example**        | Spinlocks, simple software solutions (like some of the ones we'll discuss).                      | Semaphores, Mutexes (with blocking queues).                                                      |
Busy waiting is also known as "spinning," and the locks that use it are called **spinlocks**. While it avoids context switch overhead, it's generally inefficient on a single CPU system because the waiting process prevents any other process from running.

Alright, so now lets see some solutions, people came up with

## Busy Waiting 

### Lock Variable

This is a simple software solution, its implemented at the user mode, and does not rely on OS support.

There is a shared variable called lock, which has a Boolean value, and when some process is executing the critical section, its set to 1, 0 otherwise.
```
// Shared variable, initially LOCK = 0
int LOCK = 0;

// Entry Section
while (LOCK == 1) {
    // Busy wait: Loop here until LOCK becomes 0
    // CPU is busy but doing nothing useful (spinning)
}
LOCK = 1; // Acquire the lock

// Critical Section (CS)
// ... code to access shared resources ...

// Exit Section
LOCK = 0; // Release the lock

// Remainder Section (NCS)
```

**Problems with the above thing**:

**Process P1 (entering CS):**
1.  `LOAD R0, LOCK` (Load `LOCK` value into R0)
2.  `CMP R0, #0` (Compare R0 with 0)
3.  `JNZ Step1` (Jump back to Step 1 if not zero, i.e., LOCK is 1)
4.  `STORE #1, LOCK` (Set `LOCK` to 1)

**Process P2 (also entering CS):**
(Similar logic)

Consider the following execution interleaving:

| Step  | Process | Instruction                      | `LOCK` | `R0` (P1) | `R0` (P2) | Explanation                                                                |
| :---- | :------ | :------------------------------- | :----- | :-------- | :-------- | :------------------------------------------------------------------------- |
| 1     |         | Initial state                    | 0      | -         | -         | `LOCK` is 0 (vacant).                                                      |
| 2     | P1      | `LOAD R0, LOCK`                  | 0      | 0         | -         | P1 loads `LOCK` (0) into its R0.                                           |
| 3     | I       | `CMP R0, #0`                     | 0      | 0         | -         | P1 compares R0 (0) with 0. Result is True.                                 |
| **4** | **P1**  | **(Preempted before JNZ fails)** | **0**  | 0         | -         | **P1 is preempted.** It *knows* LOCK is 0, but hasn't set it to 1 yet.     |
| 5     | P2      | `LOAD R0, LOCK`                  | 0      | -         | 0         | P2 loads `LOCK` (still 0) into its R0.                                     |
| 6     | P2      | `CMP R0, #0`                     | 0      | -         | 0         | P2 compares R0 (0) with 0. Result is True.                                 |
| 7     | P2      | `STORE #1, LOCK`                 | 1      | -         | 0         | P2 sets `LOCK` to 1, enters CS.                                            |
| **8** | **P1**  | **`STORE #1, LOCK`**             | **1**  | 0         | 0         | **P2 is preempted.** P1 resumes and also sets `LOCK` to 1, then enters CS. |

**Result:** Both P1 and P2 are simultaneously in their critical sections!

This this shit isn't satisfying the mutual exclusion thing.

This happens because, there exists preemption, and i already discussed this thing before with you guys (refer up, somewhere its written).

The problem is that the sequence of operations (load `LOCK`, check `LOCK`, set `LOCK`) is **not atomic**.
A process can load the value of `LOCK` (which is `0`), get preempted **before** it can set `LOCK` to `1`. During this preemption, another process can also load `LOCK` (which is still `0`), proceed, and enter its critical section. When the first process resumes, it still believes `LOCK` is `0` (based on its loaded register value) and proceeds to set it to `1` and enter its critical section.

**To stop this from happening:** We need to ensure that no one can "come in the middle" of the "check-then-set" operation. This requires making the operation **atomic**.


### **Test and Set Lock (TSL)

Since software solutions like the simple Lock Variable fail to guarantee mutual exclusion due to potential preemption during critical read-modify-write sequences, hardware support becomes necessary. One such hardware instruction is `Test-and-Set Lock`.

> The **Test-and-Set Lock (TSL)** instruction is an atomic hardware instruction. It performs two actions in a single, indivisible CPU cycle:
1.  **Reads** the current value of a memory location (e.g., `LOCK`).
2.  **Sets** that memory location to a new value (e.g., `1`).
3.  **Returns** the original value read.

Because it's atomic, no other process can interrupt between the read and the write.
```
// Shared variable, initially LOCK = 0
int LOCK = 0;

// Entry Section
while (TSL(R0, LOCK)) { // TSL reads LOCK into R0, sets LOCK to 1. Returns original value.
    // If original value of LOCK was 1, loop (busy wait)
    // If original value of LOCK was 0, it means we acquired the lock. R0 will be 0, loop terminates.
}
// Now LOCK is 1, and we are in CS.

// Critical Section (CS)
// ... code to access shared resources ...

// Exit Section
LOCK = 0; // Release the lock

// Remainder Section (NCS)
```

**Some Stuff about TSL Solution**

*   **Mutual Exclusion:** **YES**. Guaranteed. If `TSL` returns `0`, it means `LOCK` was `0` and has now been atomically set to `1` by *this* process. Any other process trying to `TSL` will now get `1` and busy wait.
*   **Progress:** **YES**. If the critical section is free, a process can acquire the lock and enter.
*   **Bounded Waiting:** **NO**. Not guaranteed. A process can repeatedly lose the "race" to acquire the lock and might busy-wait indefinitely if the scheduler continuously favors other processes. This leads to **starvation**.
*   **Architectural Neutrality:** **NO**. This solution relies on a specific hardware instruction (`TSL`). It's not purely a software solution and requires hardware support.

**Problems with TSL

Even though TSL guarantees mutual exclusion, it can lead to another problem called **Priority Inversion**, especially in multi-priority scheduling environments.

**Scenario:**
 - Process `P1` (low priority) enters its critical section and acquires the lock using TSL.
 - Process `P2` (high priority) becomes ready and wants to enter its critical section (which uses the same lock as P1).
 - The scheduler preempts `P1` and schedules `P2` due to `P2`'s higher priority.
 - `P2` tries to acquire the lock using TSL. Since `P1` holds the lock, `TSL` returns `1`, and `P2` goes into a busy-waiting loop (a **spinlock**).
 - Because `P2` is busy-waiting at a higher priority than `P1`, `P2` will effectively prevent `P1` from running again to release the lock.
 - `P1` can never release the lock, and `P2` can never stop busy-waiting. This creates a state similar to a **deadlock**, where `P2` (high priority) is effectively blocked by `P1` (low priority) because of the synchronization primitive. This is what's called **Priority Inversion**.
In this situation, the high-priority process `P2` is "spinning" in its entry section, consuming CPU cycles, while the low-priority process `P1` that holds the lock is stuck in the ready queue, unable to run and release the lock.

So normally when the above thing happens, one solution for this thing is the OS temporarily boosts the priority of the low-priority process, this way the low priority process quickly finishes its critical section, and releases the lock.

So to conclude This implementation guarantees **Mutual Exclusion** and **Progress**, but not **Bounded Waiting** (due to starvation possibility) and not **Architectural Neutrality** (due to reliance on `test_and_set` hardware instruction). It is subject to **Priority Inversion**.


### **Fetch and Add instruction
