# Introduction to the Operating System (OS)

An Operating System (OS) is a fundamental software layer that acts as a crucial intermediary between a computer's hardware and the end-user. In essence, you cannot directly communicate with the hardware (CPU, memory, storage devices); the OS provides a manageable and accessible interface to do so.

The OS has two primary goals that often exist in a delicate balance: convenience and efficiency.

- The primary goal is convenience for the user. The OS hides the immense complexity of the hardware, providing a user-friendly environment (like a graphical user interface) to run applications and manage files.
- The secondary goal is efficiency. The OS must manage the computer's resources—such as CPU time, memory space, and file storage—in the most efficient way possible, ensuring that the system's power is maximized and tasks are completed quickly.

To achieve these goals, the OS functions as both a Resource Allocator and a comprehensive Manager. It decides which processes get to use the CPU, how memory is allocated, and how files are stored and retrieved, ensuring fair and orderly access to all system resources.

## Types of Operating Systems

Operating systems have evolved over time to meet different computational demands. Here are some of the key types:

### Batch Operating System

In the early days of computing, Batch OS was used to maximize the use of the machine without constant human supervision. Jobs with similar requirements were grouped together into a "batch" and fed to the computer for processing. The system would then execute these jobs one after another without any user interaction. The major drawbacks of this approach were its complete lack of interactivity and its low efficiency, as the CPU would often sit idle if a job had to wait for a slow I/O operation. This system could also lead to starvation, where a low-priority job in a batch might wait for an extremely long time to execute.

### Multiprogramming Operating System

Multiprogramming was developed to solve the problem of CPU inefficiency. The core idea is to keep multiple jobs in main memory simultaneously. When one process has to wait for an I/O operation to complete (a task that doesn't require the CPU), the OS intelligently switches the CPU to another job that is ready to run. This ensures that the CPU is kept busy as much as possible, dramatically increasing system throughput and efficiency compared to a batch system.

### Multitasking (Time-Sharing) Operating System

Multitasking is a logical extension of multiprogramming that introduces user interactivity. The OS switches the CPU between different processes so frequently (often multiple times per second) that it creates the illusion that all processes are running at the same time. This is also known as time-sharing. This rapid switching allows users to interact with multiple applications simultaneously (e.g., browsing the web while listening to music). While multiprogramming focuses on keeping the CPU busy, multitasking focuses on providing a responsive experience for the user.

### Multiprocessing Operating System

Multiprocessing refers to systems that have two or more CPUs (or cores) working in parallel. Unlike multitasking, which creates an illusion of simultaneous execution on a single CPU, multiprocessing achieves true parallelism. Multiple processes can be executing at the exact same time, each on a different CPU. This leads to a significant increase in performance and computational power, allowing the system to handle more complex tasks and a heavier workload.

### Real-time Operating System (RTOS)

A Real-time Operating System is designed for environments where tasks must be completed within strict, non-negotiable deadlines. The correctness of the system depends not only on the logical result of the computation but also on the time at which the results are produced. These systems are critical in applications like military defense systems, industrial control robots, and automotive safety features (e.g., airbag deployment), where a delayed response could lead to catastrophic failure.

## Process Management

A process is one of the most fundamental concepts in a modern OS. While a program is a passive set of instructions stored on a disk a process is an active instance of a program in execution When you run a program, the OS creates a process, which is a dynamic entity that requires system resources like CPU time and memory to complete its task.

The OS creates a specific data structure in memory for each process. This structure has a defined boundary and is typically divided into four main sections:

- Stack: Used for local variables, function parameters, and return addresses. It grows and shrinks as functions are called and return.
- Heap: Dynamically allocated memory that the process can request during runtime.
- Data Segment: Contains global and static variables.
- Text Segment: Holds the compiled, executable program code.

### The Process Control Block (PCB)

To manage all the running processes, the OS maintains a data structure for ==**each one**== called the Process Control Block (PCB). The PCB is like a passport or an ID card for the process; it contains all the essential information that the OS needs to know about it. Whenever the OS needs to switch from running one process to another (an action called a context switch), ==it saves the state of the current process in its PCB and loads the state of the new process from its PCB.==

The PCB, also known as the context of the process, stores the following key attributes:

1. Process ID (PID): A unique identifier for the process.
2. Process State: The current state of the process (e.g., ready, running, waiting).
3. Program Counter: The address of the next instruction to be executed. This is vital for resuming the process exactly where it left off.
4. CPU Registers: A snapshot of the general-purpose registers, which hold data the process was actively working on.
5. Priority: A priority value used by the scheduler to decide which process to run next.
6. Memory Management Information: Details about the memory allocated to the process (e.g., base and limit registers).
7. List of Open Files & Devices: A record of all files and I/O devices currently being used by the process.
8. Protection Information: Security attributes and access permissions.

Basically because, each process can be segregated into states, there are (assume process can be in three states) 3 linked lists, for each state, storing a process which is in that particular state, and each node is the PCB of that process.

## States of a Process

Throughout its lifecycle, a process transitions through several states. These states define the current status of the process in relation to the CPU and other resources.

1. New: This is the initial state where the process is being created. The OS has acknowledged the request to start a new process but has not yet loaded it into main memory. It resides in secondary memory.
2. Ready: Once a process is loaded into main memory, it enters the Ready state. It has everything it needs to run but is waiting in a queue for its turn on the CPU. There can be many processes in the ready state at once.
3. Running: The process has been selected by the OS scheduler and its instructions are actively being executed by the CPU. In a system with a single CPU, only one process can be in the running state at any given moment.
4. Blocked (or Wait): A process transitions to the Blocked state if it must wait for an event to occur. This is typically an I/O operation, such as waiting for user input or for data to be read from a disk. While blocked, the process cannot run, even if the CPU is free.
5. Terminated (or Completed): The process has finished its execution or has been explicitly killed. In this state, the OS reclaims all the resources that were allocated to the process and removes its PCB from the system.

### Suspended States and Swapping

To manage memory effectively, especially when main memory (RAM) is full, the OS can introduce two additional "suspended" states. This process is known as swapping.

- Suspend Ready: A process that was in the Ready state (in main memory) is moved to secondary memory (disk) to make room for other processes. It is still ready to run but must be brought back into main memory first.
- Suspend Block (or Suspend Wait): A process that was in the Blocked state is moved to secondary memory. This is often more efficient, as the process was already waiting for an I/O event and not eligible to use the CPU anyway. When the I/O event it was waiting for completes, it transitions to the Suspend Ready state, as it is now ready to run but still resides on disk.

## Process State Transition Diagram

![[Process-State-Diagram.webp]]

The movement of a process between these states is governed by the OS schedulers.

## Long-Term Scheduler (LTS) – Job Scheduler

The **Long-Term Scheduler** is responsible for **controlling the admission of new processes into the system**. It selects jobs from the **job pool** in **secondary storage** (e.g., disk) and loads them into **main memory**, placing them into the **Ready** state.

### Key Characteristics:

- **Primary Role**: Controls the **degree of multiprogramming** (i.e., how many processes are in memory at once).
- **Runs**: **Infrequently**, typically when a process finishes or system load is low.
- **Selection Criteria**: Can be based on priority, expected execution time, I/O requirements, or user type.
- **Impact**: Helps balance CPU-bound and I/O-bound processes to optimize system performance.

> It decides **which jobs are allowed to enter the system** and begin competing for CPU time.

## Short-Term Scheduler (STS) – CPU Scheduler

The **Short-Term Scheduler** is responsible for **selecting one process** from the **Ready queue** (i.e., processes already in main memory and ready to execute) and **assigning the CPU to it**.

### Key Characteristics:

- **Primary Role**: Allocates the CPU to one of the ready processes.
- **Runs**: **Very frequently**, on every **context switch**, **I/O completion**, or **interrupt**.
- **Selection Criteria**: Based on a scheduling algorithm (e.g., First-Come-First-Served, Round Robin, Priority Scheduling).

> It decides **which process will run next** on the CPU.

## **Dispatcher** (Component of the Short-Term Scheduler)

The **dispatcher** is the component that **executes the decision made by the Short-Term Scheduler**. It performs the **context switch** and begins execution of the selected process.

### **Functions of the Dispatcher:**

1. **Context Switching**: Saves the state of the currently running process and restores the state of the selected one.
2. **Switching to User Mode**: Ensures safe execution by transitioning the CPU from kernel mode to user mode.
3. **Starting Execution**: Transfers control to the selected process’s program counter.

### **Key Characteristics:**

- **Does not make decisions**: It simply carries out the scheduling decision.
- **Performance-Critical**: Must be fast to reduce **dispatch latency**, ensuring smooth and responsive task switching.

> The dispatcher is the mechanism that **physically performs the switch** between processes after the scheduler chooses the next one.

## Medium-Term Scheduler (MTS)

The **Medium-Term Scheduler** is responsible for **swapping processes in and out of main memory**, primarily to manage memory usage and maintain an optimal level of multiprogramming.

### **Key Characteristics:**

- **Primary Role**: Temporarily removes (suspends) processes from memory and stores them in secondary storage when memory is limited.
- **Runs**: **Occasionally**, depending on system load and memory pressure.
- **Target Processes**: Often selects blocked or low-priority processes for suspension.
- **Transitions Managed**:
  - Moves processes from **Ready** to **Suspend Ready**.
  - Moves processes from **Blocked** to **Suspend Wait**, and back.

> It manages **which processes stay in memory** and which are moved out to keep the system efficient and responsive.

> Degree of multiprogramming refers to the maximum number of process that can be present in ready state. this is determined by the long-term scheduler  
> swapping is basically suspending a process and then resuming it  
> dispatcher is something that does the context switching,  
> short term scheduler just chooses one process to run from all the ready ones, and the rets is done by dispatcher

The entire system of state transitions, driven by preemption and scheduling, is what enables modern operating systems to be both interactive (multitasking) and efficient (multiprogramming). The dispatcher is the specific component that performs the context switching work, saving the state of the old process and loading the state of the new one.


# **CPU Scheduling and Queues

In a multi programming operating system, multiple processes are kept in the memory at the same time, to manage them, the OS has multiple queues. 
The process lifecycle involves moving between these queues, until they are executed

![[Untitled.jpeg]]

- Job Queue: This queue resides in secondary memory and contains all the processes in the system, waiting to be brought into main memory. The Long-Term Scheduler (LTS) selects processes from this queue.
- Ready Queue: This queue holds all processes that are in main memory and are ready and waiting to execute. A new process arrives in this queue from the Job Queue. The Short-Term Scheduler (STS), or CPU scheduler, selects a process from this queue to run.
- Device Queues (I/O Queues): When a process requires an I/O operation (e.g., reading from a disk), it is placed in an I/O queue associated with that specific device. It remains there until the I/O operation is complete, after which it moves back to the Ready Queue.

![[Pasted image 20250714154326.png]]
Ready state: either there is no process in the ready state, or all the processes which are there would be in the running state
Running state: either no process is in the running state, or each CPU has one process using it, so the max number of processes in running state, would be equal to the total number of CPUs
Blocked state: either no process is in the blocked state, or all the processes are in the blocked state.
### **Metrics for Scheduling algorithms
1) Arrival Time: This is the specific point in time when a process enters the ready queue for the first time. (basically comes into the ram for the first time)
2) Burst Time (Service time): This is the total amount of CPU time a process needs to complete its execution, this represents the duration the process needs to be in the running state
3) Completion 