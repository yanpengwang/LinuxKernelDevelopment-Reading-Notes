#
# LinuxKernelDevelopment-Reading-Notes in 2016.12
# Book author Robert Love
# Better view with raw format
# 
Chatper 1: Introduction to the Linux Kernel

1.1 In the summer of 1969, Bell Lab programmers sketched out a filesystem design that ultimately evolved into Unix.

1.2 A handful of characteristics of Unix are at the core of its stength.
    First, Unix is simple. Only hundreds of system calls and have a straightforward, even basic, design.
    Second, in Unix, everything is a file.
    Third, written in C which gives Unix amazing portability to deverse hareware architectures and accessibility to 
    a wide range of developers.
    Fourth, fast process creation time the the unique fork() system call.
    Finally, enable the creation of simple programs that "do one thing and do it well".
    Unix exhibit clean layering, with a strong separation between policy and mechanism.

1.3 Typical components of a kernel are:
    interrupt handlers to service interrupt requests,
    a scheduler to share processor time among multiple processes, 
    a memory management system to manage process address spaces,
    and systems such as networking and interprocess communication.
    
1.4 When an application executes a system call, we say that the kernel is executing on behalf of the application.
    Furthermore, the application is said to be executing a system call in kernel-space, and the kernel is running
    in process context.
    
1.5 When hardware wants to communicate with the system, it issues an interrupt that iterally interrupts the processor,
    which in turn interrupts the kernel. A number identifies interrupts and the kernel uses this number to execute a 
    specific interrupt handler to process and respond to the interrupt.
    
1.6 The interrupt handlers do not run in a process context. Instead, they run in a special interrupt context that is not
    associated with any process.
    
1.7 In Linux, we can generalize that each processor is doing exactly one of three things at any given moment:
    - In user-space, executing usr code in a process
    - In kernel-space, in process context, executing on behalf of a specific process
    - In kernel-space, in interrupt context, not associated with a process, handling an interrupt.
    
    
    
    
    
