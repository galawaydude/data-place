**Introduction to CPU Scheduling and Queues

In a multiprogramming operating system, multiple processes are kept in memory simultaneously. To manage their execution, the OS maintains several scheduling queues. These are queues where processes wait for their turn for various system resources.

The process lifecycle involves moving between these queues:

![[Untitled.jpeg]]
- Job Queue: This queue resides in secondary memory and contains all the processes in the system, waiting to be brought into main memory. The Long-Term Scheduler (LTS) selects processes from this queue.
- Ready Queue: This queue holds all processes that are in main memory and are ready and waiting to execute. A new process arrives in this queue from the Job Queue. The Short-Term Scheduler (STS), or CPU scheduler, selects a process from this queue to run.
- Device Queues (I/O Queues): When a process requires an I/O operation (e.g., reading from a disk), it is placed in an I/O queue associated with that specific device. It remains there until the I/O operation is complete, after which it moves back to the Ready Queue.

##### Maximum and Minimum Processes in Queues

Consider a system with 'N' CPU processors and 'M' total processes. The number of processes in each state is bounded:

- Ready State:
    
    - Minimum: 0 (if all M processes are either running on the N CPUs or are in a blocked state).
    - Maximum: M (if all M processes are loaded in memory and are waiting for the CPU).
- Running State:
    
    - Minimum: 0 (if the system is idle).
    - Maximum: N (one process can be running on each of the N available CPUs at any given time).
- Block State:
    
    - Minimum: 0 (if no process is waiting for an I/O event).
    - Maximum: M (if all M processes are simultaneously waiting for I/O).

#### Performance Metrics for Scheduling Algorithms

To compare the performance of different CPU scheduling algorithms, we use several key metrics. These parameters help us analyze how efficiently and fairly an algorithm manages processes.

- Arrival Time (AT): This is the specific point in time when a process enters the Ready Queue for the first time.
- Burst Time (BT) or Service Time (S): This is the total amount of CPU time a process requires to complete its execution. It represents the duration the process needs to be in the "Running" state.
- Completion Time (CT): This is the specific point in time when a process finishes its execution and exits the system.
- Turnaround Time (TAT): This is the total time a process spends in the system, from its arrival to its completion. It is a measure of total duration.
    > Formula: Turnaround Time (TAT) = Completion Time (CT) - Arrival Time (AT)​
- Waiting Time (WT): This is the total amount of time a process spends waiting in the Ready Queue for its turn on the CPU. It is a measure of the time spent being ready but not running.
    > Formula: Waiting Time (WT) = Turnaround Time (TAT) - Burst Time (BT)​
- Response Time (RT): This is the time elapsed from when a process arrives in the ready queue until it gets scheduled on the CPU for the very first time. This metric is particularly important for interactive systems, as it measures how quickly the system responds to a user request.
    > Formula: Response Time (RT) = Time of First Execution - Arrival Time (AT)​

#### 3. CPU Scheduling: The Core Concept

CPU Scheduling is the fundamental task of selecting a process from the Ready Queue and allocating the CPU to it. This decision is made by the Short-Term Scheduler (also known as the CPU scheduler) and carried out by the Dispatcher.

A scheduling decision must be made by the OS whenever a process transitions between states, specifically when a process:

1. Moves from Running to Terminated (it has finished).
2. Moves from Running to Waiting (it requests I/O).
3. Moves from Running to Ready (it is preempted by an interrupt or timer).
4. Moves from Waiting to Ready (its I/O operation completes).

Scheduling that occurs only under conditions 1 and 2 is called non-preemptive (the process runs until it's done or blocks itself). Scheduling that can also occur under conditions 3 and 4 is called preemptive (the OS can force a process off the CPU).

## Scheduling Algorithms

### First-Come, First-Served (FCFS)

FCFS is the simplest scheduling algorithm. As the name implies, the process that arrives in the ready queue first is the first one to get the CPU.

- Criteria: Arrival Time (AT)
- Mode: Non-Preemptive
- Implementation: Easily managed with a simple FIFO (First-In, First-Out) queue data structure. The time complexity to implement is O(n).
- Tie-Breaker: If two processes arrive at the same time, the one with the lower Process ID (PID) is usually executed first.
- Note: In a non-preemptive algorithm like FCFS, the Waiting Time is equal to the Response Time, because a process must wait its entire turn before it ever gets to run for the first time.

#### The Convoy Effect: A Major Flaw in FCFS

The FCFS algorithm suffers from a significant problem known as the Convoy Effect. This occurs when a process with a very long burst time arrives before other processes that have very short burst times. The short processes get stuck waiting behind the long one, much like a convoy of fast cars stuck behind a slow truck on a single-lane road. This leads to a drastic increase in the average waiting time for the system.

Example of the Convoy Effect:  
Consider three processes arriving at nearly the same time:

- P1: AT=0, BT=20
- P2: AT=1, BT=2
- P3: AT=2, BT=1

If scheduled with FCFS, P1 runs first. P2 and P3 have to wait for 20 units of time for P1 to finish.

- Average Waiting Time = (0 + (20-1) + (22-2)) / 3 = (0 + 19 + 20) / 3 = 13.0 (approx)

Now, consider if they arrived in a different order (P2, P3, P1):

- P2: AT=0, BT=2
- P3: AT=1, BT=1
- P1: AT=2, BT=20

The short processes run first. P1 only has to wait for 3 units of time.

- Average Waiting Time = (0 + (2-1) + (3-2)) / 3 = (0 + 1 + 1) / 3 = 0.67 (approx)

As you can see, the average waiting time is dramatically better, demonstrating the severe impact of the convoy effect.

---

### Shortest Job First (SJF)

SJF is an algorithm that prioritizes processes based on their CPU burst time. It selects the process with the smallest burst time from the ready queue.

- Criteria: Burst Time (BT)
- Mode: Non-Preemptive (in this version)
- Tie-Breaker: If two processes have the same burst time, FCFS is used (the one that arrived earlier is chosen).
- Implementation: Can be implemented with a min-heap data structure, leading to a time complexity of O(n log n).

SJF is provably optimal in terms of minimizing the average waiting time and average turnaround time for a given set of processes.

#### Advantages and Disadvantages of SJF

- Advantages:
    - Provides the maximum throughput because it processes more jobs in a given time period.
    - Achieves the minimum average waiting time and turnaround time.
- Disadvantages:
    
    - Starvation: Longer jobs may never get to run if a steady stream of shorter jobs keeps arriving.
    - Impracticality: The biggest issue is that the exact CPU burst time of a process cannot be known in advance. This makes the true SJF algorithm impossible to implement in a real-world general-purpose OS.

### Predicting Burst Time for a Practical SJF

To overcome the impracticality of SJF, we can try to predict the next CPU burst time based on a process's previous behavior.

1. Static Prediction: Prediction is based on fixed attributes of the process.
    
    - Process Size: Assume larger processes will have longer burst times.
    - Process Type: Differentiate between OS processes (often short bursts), interactive foreground processes (short bursts), and background batch processes (long bursts).
2. Dynamic Prediction (Exponential Averaging / Aging): This is a more common and effective technique. It calculates a weighted average of previous burst times, giving more weight to recent behavior.
    
    > Formula: τ_n+1 = α * t_n + (1 - α) * τ_n​  
    > Where:
    > 
    > - ​τ_n+1​ is the predicted burst time for the next burst.
    > - ​t_n​ is the duration of the actual most recent CPU burst.
    > - ​τ_n​ was the predicted value for the most recent burst.
    > - ​α​ (alpha) is the smoothing factor (0 ≤ α ≤ 1). A value of α​ close to 1 gives high weight to the most recent burst, while a value close to 0 gives more weight to the past history.

---

### Shortest Remaining Time First (SRTF)

SRTF is the preemptive version of SJF. It is one of the most important scheduling algorithms.

- Criteria: Remaining Burst Time
- Mode: Preemptive

How it works: The scheduler always allocates the CPU to the process that has the shortest remaining time to completion. If a new process arrives in the ready queue with a burst time shorter than the remaining time of the currently executing process, the OS will preempt the current process and start executing the new, shorter one.

- Note: SRTF is also optimal, providing the minimum average waiting and turnaround times. It also suffers from the same issues as SJF: the need to know burst times and the potential for starvation of long processes.

---

### Round Robin (RR) Algorithm

Round Robin is designed specifically for time-sharing systems. It is a preemptive algorithm that provides a fair share of CPU time to all processes.

- Criteria: Time Quantum (TQ) + Arrival Time
- Mode: Preemptive

How it works:

1. Processes are kept in a FIFO ready queue.
2. The scheduler picks the first process and lets it run for a fixed duration called a time quantum (or time slice).
3. If the process finishes within its time quantum, it terminates and leaves the system.
4. If the process does not finish, it is preempted, and the OS moves it to the tail of the ready queue. The scheduler then picks the next process at the head of the queue.

#### The Role of the Time Quantum (TQ)

The performance of RR is highly dependent on the size of the time quantum:

- Large TQ: If the time quantum is very large (e.g., larger than any process's burst time), RR behaves exactly like the FCFS algorithm.
    
- Small TQ: If the time quantum is very small, it results in a large number of context switches. While this improves response time, the overhead of frequent context switching can degrade the overall performance and throughput of the system.
    
- Trade-off:
    
    - Increasing TQ​ ➞ ↓​ Context Switches, ↑​ Average Response Time.
    - Decreasing TQ​ ➞ ↑​ Context Switches, ↓​ Average Response Time.  
        A TQ of infinity makes RR equivalent to FCFS.

---

### Other Scheduling Algorithms

#### Longest Job First (LJF) & Longest Remaining Time First (LRTF)

These algorithms are the inverse of SJF and SRTF. They prioritize the process with the longest burst time.

- LJF: Non-preemptive. Schedules the job with the longest BT.
- LRTF: Preemptive. Schedules the job with the longest remaining BT.

These algorithms are generally very poor for average waiting and turnaround times and can lead to severe starvation for short jobs. They are rarely used in practice but serve as useful academic comparisons.

#### Highest Response Ratio Next (HRRN)

HRRN is a non-preemptive algorithm that tries to correct the starvation problem of SJF while still favoring shorter jobs.

- Criteria: Response Ratio
- Mode: Non-Preemptive

How it works: At each scheduling point, the algorithm calculates a "response ratio" for every process in the ready queue and selects the one with the highest ratio.

> Formula: Response Ratio = (Waiting Time + Burst Time) / Burst Time​  
> ​Response Ratio = (W + S) / S​

By including the waiting time (W​) in the numerator, HRRN ensures that as a long process waits, its response ratio increases, eventually becoming high enough for it to be scheduled. This "aging" technique effectively prevents starvation while still giving a natural advantage to shorter jobs (since burst time S​ is in the denominator).

---

### Priority Scheduling

This is a general class of algorithms where each process is assigned a priority number, and the CPU is allocated to the process with the highest priority. (Conventionally, a lower number means higher priority).

- Static Priority: The priority of a process is fixed when it is created and does not change. This is simple but can lead to starvation if a low-priority process never gets to run.
- Dynamic Priority: The priority of a process can change during its execution. A common technique is aging, where the priority of a process is gradually increased the longer it waits in the ready queue. This solves the starvation problem.

Priority scheduling can be implemented in two modes:

- Non-Preemptive Priority: When a process is given the CPU, it runs to completion or until it blocks, regardless of whether a higher-priority process arrives.
- Preemptive Priority: If a new process arrives with a priority higher than that of the currently running process, the current process is preempted and the new process is scheduled.

### Multi-Level Queue Scheduling

This algorithm partitions the ready queue into several separate queues. For example, you might have:

1. System Processes (Highest Priority)
2. Interactive Processes (Medium Priority)
3. Batch Processes (Lowest Priority)

Each queue has its own scheduling algorithm (e.g., RR for interactive, FCFS for batch). Scheduling between the queues is typically a fixed-priority preemptive scheme. For instance, no process in the batch queue can run unless the queues for system and interactive processes are empty.

- Advantage: Allows the use of different, appropriate algorithms for different classes of processes.
- Disadvantage: Starvation is a major problem. Processes in low-priority queues might never execute if there is a continuous supply of higher-priority processes.

### Multi-Level Feedback Queue Scheduling

This is an enhancement of the multi-level queue that addresses the starvation problem by allowing processes to move between queues. This is the most complex, but also one of the most flexible and widely used, scheduling algorithms.

How it works:

- If a process uses too much CPU time in a high-priority queue, it can be moved to a lower-priority queue. This reserves the high-priority queues for short, interactive jobs.
- If a process waits for too long in a low-priority queue, it can be moved to a higher-priority queue. This process, known as aging, ensures that no process starves.

By allowing processes to "feedback" to different queues, the system can dynamically adjust to the behavior of processes, providing a good balance of responsiveness, throughput, and fairness.
