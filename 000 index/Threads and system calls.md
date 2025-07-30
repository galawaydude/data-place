### System calls

A system call is a programmatic way in which a program requests a service from the kernel of the operating system on which it is executed. System call provided the services of the operating system to the user programs via a API. System calls are the only entry points into the kernel system, and are executed in kernel mode.![[introduction_to_system_call.webp]]

A system call is initiated by  by the program, executing a specific instruction, which triggers a switch to kernel mode, allowing the program to request a service from the OS. The OS then handles the request, performs the necessary operations and returns the result back to the program. System calls are essential for the proper functioning of an operating system, as they provide a standardized way for programs to access system resources. Without system calls, each program would need to implement its methods for accessing hardware and system services, leading to inconsistent and error-prone behavior.
> the kernel is the core part of the OS that has complete control over everything in the system.
> it provides
### Features of system calls