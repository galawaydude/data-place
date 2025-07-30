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