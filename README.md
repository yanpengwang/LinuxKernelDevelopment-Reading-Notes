#
# LinuxKernelDevelopment-Reading-Notes
# Create in 2016.12 as a review of previous reading
# Book info: the third edition, 2010, by Robert Love
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
    
1.8 Linux is a monolithic kernel; that is; the Linux kernel executes in a single address space entirely in kernel mode.
    Linux borrows much of good from microkernels(Windows XP, Vista, 7, Mach):
    - Linux boasts a modular design
    - the capability to preempt itself(kernel preemption)
    - support for kernel threads
    - the capability to dynamically load seperate binaries(kernel modules) into the kernel image.
    
1.9 (version 2.6.1.1)The minor release (6) also determines whether the kernel is a stable or development kernel; 
    an even number is stable; whereas an odd number is development.
    
    
Chapter 2. Getting started with the Kernel

2.1 The linux kernel has serveral unique attributes as compared to a normal user-space application:
    - The kernel has access to neither the C library nor the standard C headers.
    - The kernel is coded in GNU C.
    - The kernel lacks the memory protection afforded to user-space.
    - The kernel cannot easily execute floating-point operations.
    - The kernel has a small per-process fixed-size stack.
    - Because the kernel has asynchronous interrupts, is preemptive, and supports SMP, synchronization and 
      concurrency are major concerns within the kernel. 
    - Portability is important.

