### System calls

A system call is a programmatic way in which a program requests a service from the kernel of the operating system on which it is executed. System call provided the services of the operating system to the user programs via a API. System calls are the only entry points into the kernel system, and are executed in kernel mode.![[introduction_to_system_call.webp]]

A system call is initiated by  by the program, executing a specific instruction, which triggers a switch to kernel mode, allowing the program to request a service from the OS. The OS then handles the request, performs the necessary operations and returns the result back to the program. System calls are essential for the proper functioning of an operating system, as they provide a standardized way for programs to access system resources. Without system calls, each program would need to implement its methods for accessing hardware and system services, leading to inconsistent and error-prone behavior.
> the kernel is the core part of the OS that has complete control over everything in the system.
> it provides a set of interfaces (APIs) through which user programs can request services like 
> File operations (open, read, write, close)
> Process control (fork, exec, exit)
> Memory management (allocate, deallocate)
> Device handling (I/O operations)
> 
> by default a program would be in user mode, and when it needs to perform a task that requires privileged access, it makes a system call. This switches the cpu from user mode to kernel mode.

### How does a System call work

Sometimes user programs need to do some special things that can't be done without the permission of the OS like reading from a file, writing to a file, getting any information from the hardware or requesting a space in memory. There are special predefined instructions to make a request to the operating system. These instructions are system calls. The program uses these system calls in its code when its needed. 

When the OS sees the system call, then it recognizes that the program needs help at this time so it temporarily stops the program execution and gives all the control to a special part of itself call the kernel, Now the kernel solves the need of the program. Now the operating system performs the operating that is requested by the program, after performing the operating the OS gives control back to the program for further execution.

### An example of the above thing

Let’s say the program wants to read some text from a file called `notes.txt`. It uses code like this:

```c
#include <stdio.h>

int main() {
    FILE *file = fopen("notes.txt", "r");
    if (file == NULL) {
        perror("Error opening file");
        return 1;
    }

    char buffer[100];
    fgets(buffer, 100, file);
    printf("First line: %s\n", buffer);

    fclose(file);
    return 0;
}
```

This code looks simple, but behind the scenes, it contains system calls. Functions like `fopen()`, `fgets()`, and `fclose()` are actually calling lower-level system calls like `open()`, `read()`, and `close()`.

#### What the OS Does

- When the program reaches `fopen("notes.txt", "r")`, it knows it needs access to the disk, which it can't do directly.
- So, it makes a system call.
- The OS notices this and recognizes that the program needs help.
- It temporarily **pauses** the program and transfers control to a special part of the OS called the **kernel**.
- The kernel handles the file opening task.
- Once done, the OS gives control back to the program so it can continue running.

The same process happens when calling `fgets()` (to read data) and `fclose()` (to close the file).

### Types of System Calls

Services provided by the OS are typically related to any kind of operating that a user program can perform like creation, termination, forking, moving, communication, etc. Similar types of operating are grouped into one single system call category.
![[Types-of-System-Calls-(3)-(2).png]]

### Process Control

This category encompasses system calls that govern the creation, termination, and management of processes, the fundamental units of execution.

- Creation and Execution: The most fundamental process control calls are those that create new processes and load new programs. In UNIX-like systems, this is famously a two-step operation: fork() creates a new child process that is an identical copy of the parent, and exec() (a family of calls like execlp, execv, etc.) replaces the current process's memory image with that of a new program.2 Windows, in contrast, combines these steps into a single, more complex API call,  
    CreateProcess()
- Termination and Synchronization: The exit() call allows a process to terminate itself voluntarily, typically returning a status code to its parent.1 The  
    wait() family of calls provides a synchronization mechanism, allowing a parent process to pause its own execution until one of its child processes has terminated.1 The Windows equivalent for this synchronization is often  
    WaitForSingleObject().
- Identification: The getpid() call in UNIX returns the unique process identifier (PID) of the calling process, a crucial piece of information for process management and signaling. Windows provides the GetCurrentProcessId() call for the same purpose.


### File And Device Management

This is arguably the most frequently used category of system calls, providing the interface for all Input/Output (I/O) operations. In UNIX-like systems, a powerful abstraction treats hardware devices as if they were files, allowing a small, consistent set of system calls to be used for interacting with a wide variety of resources.5

- Core Operations: The canonical file management calls are open(), read(), write(), and close().
- open(): Takes a file path and access flags, and upon success, returns a small, non-negative integer called a file descriptor. This descriptor is a handle used in all subsequent operations on that file.1
- read(): Reads a specified number of bytes from an open file descriptor into a memory buffer provided by the application.2
- write(): Writes a specified number of bytes from an application's memory buffer to an open file descriptor.2
    
- close(): Informs the kernel that the process is finished with a file descriptor, allowing the kernel to release associated resources.2
    

- Device Control: For operations that do not fit the simple byte-stream model of read/write, such as configuring a serial port or ejecting a CD-ROM, UNIX provides the ioctl() (I/O control) system call. This acts as a general-purpose, device-specific escape hatch.7 Windows uses a more structured set of API calls for device management, such as  
    SetConsoleMode() for configuring the command-line console.5


### Information Maintenance and Inter-Process Communication

This broad category includes system calls for retrieving or setting system information and for facilitating communication between different processes (Inter-Process Communication, or IPC).

- Information Maintenance: These calls allow a process to query or modify its environment or the system's state. Examples include getpid() for process information, alarm() or SetTimer() for scheduling a future event, and sleep() for pausing execution for a set duration.2
    
- Inter-Process Communication (IPC): Since processes have isolated address spaces, they require kernel-mediated mechanisms to communicate.
    

- pipe(): Creates a simple, unidirectional communication channel, typically used between a parent and child process.5
    
- socket(): Creates a network socket, which is a versatile endpoint for communication between processes on the same machine or across a network.2
    
- Shared Memory: For high-bandwidth communication, calls like shmget() (in System V IPC) or mmap() (in POSIX) allow multiple processes to map the same region of physical memory into their respective virtual address spaces, enabling direct memory sharing.5 Windows provides analogous functionality with  
    CreateFileMapping() and MapViewOfFile().7

> write about the system calls in windows and linux
