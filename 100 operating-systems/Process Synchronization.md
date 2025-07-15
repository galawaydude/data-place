
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

This is similar to TSL, and is also hardware based implementation. Its the same as the above thing, but even more useless, does not bring any thing new.
```c
// Shared unsigned integer 'L', initialized to 0.
// L = 0 corresponds to lock being available.
// Any non-zero value corresponds to lock being not available (or held).
unsigned int L = 0;

void Acquire_Lock(int process_id) {
    // In Fetch-and-Add(L, 1), 'L' is incremented, and its OLD value is returned.
    // If the old value was 0, this process successfully acquired the lock.
    // If the old value was non-zero, another process already holds/is waiting for the lock.
    while (Fetch_And_Add(L, 1) != 0) {
        // Busy wait (spin) until Fetch_And_Add returns 0.
        // This means the lock was available (L=0) and we just made it L=1.
        // Subsequent calls will return 1, 2, 3... and busy-wait.
    }
}

void Release_Lock(int process_id) {
    L = 0; // Set L back to 0 to make the lock available.
}
```

**Stuff about Fetch and Add

 - **Does L overflow?**
    - Yes, `L` can overflow. If many processes try to acquire the lock concurrently, `L` will keep incrementing. If it's an `unsigned int`, it will eventually wrap around to `0` after reaching its maximum value. If it wraps to `0`, it will incorrectly signal that the lock is available, even if processes are waiting.

-  **Does L take on non-zero values when the lock is actually available?**
    - No. `L` is initialized to `0`. When a process successfully acquires the lock, `Fetch_And_Add(L, 1)` will return `0` (the old value) and set `L` to `1`. When `Release_Lock` is called, `L` is explicitly set back to `0`. The only way `L` would be non-zero when released is if `Release_Lock` itself was flawed or if `L` were somehow externally modified. In this specific implementation, it correctly returns to `0`.

-  **Does it work correctly without starvation?**
    - No, it does not guarantee freedom from starvation. Like TSL, it's a busy-waiting mechanism. If multiple processes are spinning, the scheduler might repeatedly favor one process over others, leading to some processes never getting the `Fetch_And_Add` operation to return `0`.

**Trace Example of Fetch-and-Add:**

Assume `L` starts at `0`. `P1`, `P2`, `P3`, `Pn` try to acquire the lock.

| Step | Process | Action (Acquire_Lock)         | `Fetch_And_Add(L, 1)` Returns | `L`   | Current State                                             |
| :--- | :------ | :---------------------------- | :---------------------------- | :---- | :-------------------------------------------------------- |
| 1    |         | Initial state                 | -                             | 0     | Lock available.                                           |
| 2    | P1      | `Fetch_And_Add(L, 1)`         | 0                             | 1     | P1 gets 0, so it proceeds to CS. Other processes will get non-zero. |
| 3    | P2      | `Fetch_And_Add(L, 1)`         | 1                             | 2     | P2 gets 1, busy-waits.                                    |
| 4    | P3      | `Fetch_And_Add(L, 1)`         | 2                             | 3     | P3 gets 2, busy-waits.                                    |
| ...  | ...     | ...                           | ...                           | ...   | ...                                                       |
| N    | Pn      | `Fetch_And_Add(L, 1)`         | N-1                           | N     | Pn gets N-1, busy-waits.                                  |
|      | P1      | (P1 finishes CS, `Release_Lock`) | -                             | 0     | P1 sets L back to 0. Now all waiting processes will see 0. |

This trace shows that processes `P2`, `P3`, ..., `Pn` will be busy-waiting because `Fetch_And_Add` returned a non-zero value. When P1 releases the lock (setting `L` to `0`), *all* the waiting processes will see `L` as `0` and race again. The one that wins the `Fetch_And_Add` will get `0` and enter the CS, while others increment `L` further. This structure doesn't inherently give priority to the longest waiting process.

### Disabling Interrupts

This is a simple, yet powerful, synchronization mechanism, often used in operating system kernels, but rarely in user-level applications.
 When a process wants to enter its critical section, it **disables interrupts**. This prevents the CPU from being interrupted by the scheduler, I/O devices, or other processes. Once the critical section is finished, the process **enables interrupts** again.
 ```
// Entry Section
disable_interrupts();

// Critical Section (CS)
// ... code to access shared resources ...

// Exit Section
enable_interrupts();

// Remainder Section (NCS)
```

**Some stuff about this

- **Mutual Exclusion:** **YES**. If interrupts are disabled, no other process can be scheduled on the same CPU, thus guaranteeing that only one process executes its critical section at a time. (Note: This only works for single-processor systems or for protecting data on the *specific* processor that disables interrupts; on multi-processor systems, other CPUs can still access the shared data unless a more sophisticated mechanism is used in conjunction).
-  **Progress:** **YES**. If a process wants to enter, it can (assuming no one else is currently in CS with interrupts disabled).
-  **Bounded Waiting:** **NO**. Not guaranteed. If a process disables interrupts and then crashes or gets stuck in an infinite loop *within its critical section*, interrupts will never be re-enabled, and the entire system will halt. Also, it doesn't ensure fairness; a process could repeatedly enter its CS.
-  **Architectural Neutrality:** **NO**. This method is highly dependent on hardware (CPU's ability to disable/enable interrupts) and operating system kernel privileges (user-level programs typically cannot disable interrupts directly).

**Problem with Disabling Interrupts (for user processes):

The major problem is that you **cannot trust user processes** to properly re-enable interrupts. If a user process disables interrupts and then crashes, or enters an infinite loop, or maliciously refuses to re-enable them, the entire system will freeze. Due to this severe risk, it is **rarely used** as a general synchronization method for user-level processes. It is primarily reserved for critical sections within the OS kernel itself, where the code is tightly controlled and highly trusted.

### **Strict Alternation Approach (Turn Variable)

Same shit, its a software based approach, implemented at the user level, and its meant to work only for two processes.

A shared turn variable indicates which process is should execute its cs
     `turn = 0`: P0 is allowed to enter CS.
     `turn = 1`: P1 is allowed to enter CS.


 ```c
// Shared variable, initially turn = 0
int turn = 0;

// For Process P0:
// Entry Section
while (turn != 0) {
    // Busy wait: P0 waits if it's not its turn
}
// Critical Section (CS)
// ...

// Exit Section
turn = 1; // Give turn to P1

// Remainder Section (NCS)


// For Process P1:
// Entry Section
while (turn != 1) {
    // Busy wait: P1 waits if it's not its turn
}
// Critical Section (CS)
// ...

// Exit Section
turn = 0; // Give turn to P0

// Remainder Section (NCS)
```

**Some stuff about this

- **Mutual Exclusion:** **YES**. Guaranteed. If `turn` is `0`, only P0 can enter. If `turn` is `1`, only P1 can enter. Since `turn` can only be `0` or `1`, only one process can enter at a time.
-   **Progress:** **NO**. Not guaranteed. This is a significant drawback.
    -   **Scenario:** Suppose `turn = 0` and P0 completes its CS and sets `turn = 1`. Now P1 is allowed.
    -   What if P1 is much slower, or simply doesn't need to enter its critical section right now? P1 stays in its remainder section.
    -   P0 now wants to enter its critical section again. It checks `turn != 0`, but `turn` is currently `1` (because P1 hasn't taken its turn yet). So, P0 must busy-wait, even though the critical section is empty!
    -   This violates the "Progress" requirement, which states that if the CS is empty, a process that wants to enter should not be indefinitely prevented.
-   **Bounded Waiting:** **YES**. Guaranteed. Each process only has to wait at most one turn for the other process to complete its critical section. After the other process completes its CS, it passes the turn.
-   **Architectural Neutrality:** **YES**. It's a purely software-based solution, so it works on any architecture.


### Peterson's Solution

This is the goat in this category, solves everything properly.
its a software based methods, and builds upon the strict alternation and add a thing called interest flags on top of that. because of this, this too works only for 2 processes.
this too is implemented in user mode.
```c
// Shared variables
#define N 2
#define TRUE 1
#define FALSE 0
bool interested[N] = {FALSE, FALSE}; // Initialize both to FALSE
int turn; // No initial value specified, but effectively set by the first process

void Entry_section(int process) { // 'process' is 0 or 1
    int other = 1 - process;

    interested[process] = TRUE; // 1. Declare interest
    turn = other;               // 2. Give turn to the other process

    // 3. Busy wait if:
    //    a) The other process is also interested AND
    //    b) It's currently the other process's turn
    while (interested[other] == TRUE && turn == other) {
        // Spin
    }
}

void Exit_section(int process) {
    interested[process] = FALSE; // 4. Declare no longer interested
}

/*
// Usage for Process P0:
Entry_section(0);
// Critical Section (CS)
Exit_section(0);
// Remainder Section (NCS)

// Usage for Process P1:
Entry_section(1);
// Critical Section (CS)
Exit_section(1);
// Remainder Section (NCS)
*/
```

**How the thing works (pretty good actually, very simple, but brilliant approach)

1.  **Process `i` declares `interested[i] = TRUE;`**
    *   This signals its intention to enter the critical section.

2.  **Process `i` sets `turn = other;`**
    *   This is the crucial step that prevents deadlock. `P_i` is being polite: it says, "After I declare my interest, I'll give you (the other process) the first chance to go if you also want to." This resolves the "both interested and stuck" scenario from the previous attempt.

3.  **Process `i` enters the `while` loop:** `while (interested[other] == TRUE && turn == other)`
    *   **Case A: Only one process is interested (e.g., P0 wants in, P1 doesn't).**
        *   `interested[P1]` is `FALSE`. The loop condition `(interested[P1] == TRUE && ...)` immediately becomes `FALSE`.
        *   P0 enters its critical section without waiting. This satisfies **Progress**.
    *   **Case B: Both processes become interested (almost) simultaneously.**
        *   Let's say P0 calls `Entry_section(0)` and P1 calls `Entry_section(1)` very close in time.
        *   **P0 steps:** `interested[0]=TRUE`, `turn=1`.
        *   **P1 steps:** `interested[1]=TRUE`, `turn=0`.
        *   One of them will execute `turn = other;` last. Let's assume P1 sets `turn = 0` *after* P0 sets `turn = 1`. So, the final value of `turn` before either enters their `while` loop is `0`.
        *   Now, P0's loop condition: `(interested[1] == TRUE && turn == 1)`
            *   `interested[1]` is `TRUE`.
            *   `turn` is `0`. So `turn == 1` is `FALSE`.
            *   The whole condition is `FALSE`. P0 **enters CS**.
        *   P1's loop condition: `(interested[0] == TRUE && turn == 0)`
            *   `interested[0]` is `TRUE`.
            *   `turn` is `0`. So `turn == 0` is `TRUE`.
            *   The whole condition is `TRUE`. P1 **busy-waits**.
        *   The process that *last* set `turn = other` is the one that will be forced to wait. The other process proceeds. This guarantees **Mutual Exclusion** and resolves the deadlock scenario.
    *   **Case C: A process is interrupted after setting `turn` but before entering the loop.**
        *   This is handled by the last-setter-waits rule. If P0 sets `turn=1`, and then P1 sets `turn=0`, P1 will enter CS and P0 will wait. If P1 then releases CS, it sets `interested[1]=FALSE`. P0's loop condition `(interested[1] == TRUE && turn == 1)` will now evaluate to `(FALSE && turn == 1)`, allowing P0 to enter.


**Some stuff about this 

It satisfies all the criteria we discussed above, pretty great as i said.


## Non-Busy Methods

Busy methods are easy to implement, cause you do not have to worry about, blocking the process. but this is highly inefficient especially on single processor systems, (You know why), so now we will be looking non-busy methods

### **Sleep and Wake

The sleep and wake primitives, allow processes to block themselves, when a condition is not met and can be awakened by another process when the condition changes. This is used in Producer Consumer problem.

### Producer/Consumer Problem

*   **Producer:** Produces data items and puts them into a shared buffer.
*   **Consumer:** Consumes data items from the shared buffer.
*   **Shared Buffer:** Has a finite size (e.g., `N` slots).
*   **Synchronization Challenge:**
    *   Producer must not try to add to a full buffer.
    *   Consumer must not try to remove from an empty buffer.

**Pseudo-code using Sleep/Wake primitives:**

```c
// Shared variables
#define N 100        // Total slots in buffer
int count = 0;       // Current number of items in buffer (initially 0)

void producer(void) {
    int item;
    while (TRUE) {
        item = produce_item(); // Generate an item

        if (count == N) {      // If buffer is full
            sleep();           // Producer sleeps
        }
        insert_item(item);     // Add item to buffer
        count++;

        if (count == 1) {      // If buffer was empty and now has 1 item
            wakeup(consumer);  // Wake up consumer (if it was sleeping)
        }
    }
}

void consumer(void) {
    int item;
    while (TRUE) {
        if (count == 0) {      // If buffer is empty
            sleep();           // Consumer sleeps
        }
        item = remove_item();  // Remove item from buffer
        count--;

        if (count == N - 1) {  // If buffer was full and now has one space
            wakeup(producer);  // Wake up producer (if it was sleeping)
        }
        consume_item(item);    // Process the item
    }
}
```

**Problems with sleep and wake

This direct implementation of `sleep` and `wakeup` suffers from a classic **race condition** that can lead to **deadlock**.

**Deadlock Scenario Example (Consumer and `count == 0`):**

1.  **Buffer is Empty (`count = 0`).**
2.  **Consumer** executes `if (count == 0)`. The condition `(0 == 0)` is `TRUE`.
3.  **Consumer is preempted** *just before* it executes `sleep()`.
4.  **Producer** runs. It produces an item, `insert_item()`, and increments `count` to `1`.
5.  **Producer** then executes `if (count == 1)`. This is `TRUE`.
6.  **Producer** calls `wakeup(consumer)`. However, the consumer is *not yet sleeping* (it was preempted before calling `sleep()`). So, this `wakeup` call is lost. The signal is sent, but no one is listening.
7.  **Producer** continues its work, eventually fills the buffer, and then calls `sleep()` itself (because `count == N`).
8.  **Consumer** resumes. It executes `sleep()`. Now the consumer is actually sleeping.
9.  **Result:** Both Producer and Consumer are now sleeping, believing they are waiting for each other, but the `wakeup` signal was missed. The system is in a **deadlock**.

Btw, as i said before, the main cause for most of these sou



