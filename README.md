#
# LinuxKernelDevelopment-Reading-Notes
# Create in 2016.12 as a review of previous reading
# Book info: the third edition, 2010, by Robert Love
# Better view with raw format
# 

Chatper 1: Introduction to the Linux Kernel

1.1 
In the summer of 1969, Bell Lab programmers sketched out a filesystem design
that ultimately evolved into Unix.

1.2 
A handful of characteristics of Unix are at the core of its stength.
First, Unix is simple. Only hundreds of system calls and have a 
straightforward, even basic, design.
Second, in Unix, everything is a file.
Third, written in C which gives Unix amazing portability to deverse hareware 
architectures and accessibility to a wide range of developers.
Fourth, fast process creation time the the unique fork() system call.
Finally, enable the creation of simple programs that "do one thing and do 
it well". Unix exhibit clean layering, with a strong separation between 
policy and mechanism.

1.3 
Typical components of a kernel are:
interrupt handlers to service interrupt requests,
a scheduler to share processor time among multiple processes, 
a memory management system to manage process address spaces,
and systems such as networking and interprocess communication.
    
1.4 
When an application executes a system call, we say that the kernel is 
executing on behalf of the application.
Furthermore, the application is said to be executing a system call in 
kernel-space, and the kernel is running in process context.
    
1.5 
When hardware wants to communicate with the system, it issues an interrupt 
that iterally interrupts the processor, which in turn interrupts the kernel. 
A number identifies interrupts and the kernel uses this number to execute a 
specific interrupt handler to process and respond to the interrupt.
    
1.6 
The interrupt handlers do not run in a process context. Instead, they run in 
a special interrupt context that is not associated with any process.
    
1.7 
In Linux, we can generalize that each processor is doing exactly one of 
three things at any given moment:
- In user-space, executing usr code in a process
- In kernel-space, in process context, executing on behalf of a process
- In kernel-space, in interrupt context, not associated with a process, 
  handling an interrupt.
    
1.8 
Linux is a monolithic kernel; that is; the Linux kernel executes in a single 
address space entirely in kernel mode.
Linux borrows much of good from microkernels(Windows XP, Vista, 7, Mach):
- Linux boasts a modular design
- the capability to preempt itself(kernel preemption)
- support for kernel threads
- the capability to dynamically load seperate binaries(kernel modules) into 
the kernel image.
    
1.9 
(version 2.6.1.1)The minor release (6) also determines whether the kernel is 
a stable or development kernel; an even number is stable; whereas an odd 
number is development.
    
    
Chapter 2. Getting started with the Kernel

2.1 
The linux kernel has serveral unique attributes as compared to a normal 
user-space application:
- The kernel has access to neither the C library nor the standard C headers.
  (the kernel is not linked against the standard C library- or any other 
  library)
- The kernel is coded in GNU C.
  (The linux kernel developers use both ISO C99 and GNU C extensions to the 
  C language)
- The kernel lacks the memory protection afforded to user-space.
  (Kernel memory is not pageable, therefore, every byte of memory you 
  consume is one less byte of available physical memory.) 
- The kernel cannot easily execute floating-point operations.
- The kernel has a small per-process fixed-size stack.
  (The kernel stack is neither large nor dynamic, it is small and fixed in 
  size. Historically, the kernel stack is two pages, which generally implies 
  that it is 8KB on 32-bit architecture and 16KB on 64-bit)
- Because the kernel has asynchronous interrupts, is preemptive, and 
  supports SMP, synchronization and concurrency are major concerns within 
  the kernel. 
- Portability is important.
  (A handful of rules - such as remain endian neutral, be 64-bit clean, do 
  not assume the word or page size)

Chapter 3. Process Management

3.1 
Each thread includes a unique program counter, process stack, and set of 
processor registers.

3.2 
Process lifecycle: fork() creates a new process by duplicating an existing 
one - parent. After fork() the parent resumes execution and the child starts 
execution at the same place. Often, immediately after a fork it is desirable 
to execute a new, different program. The exec() family of function calls 
creates a new address space and loads a new program into it. 
Finally, a program exists via the exit() system call. The function 
terminates the process and frees all its resources. A parent process can 
inquire about the status of a terminated child via the wait4() system call. 
When a process exits, it is placed into a special zombie state that 
represents terminated processes until the parent call wait() or waitpid().
    
3.3 
The kernel stores the list of processes in a circular doubly linked list 
called the task list. Each element in the task list is a process descriptor 
of the type struct task_struct, which contains the data that describes the 
executing program-open files, the process's address space, pending signals, 
the process's state, and much more.
    
3.4 
The task_struct structure is allocated via the slab allocator to provide 
object reuse and cache coloring. A new structure, struct thread_info, was 
created that again lives at the bottom of the stack(for stacks grow down)
Inside the kernel, tasks are typically referenced directly by a pointer to 
their task_struct structure.
     
3.5 
Process state. Each process on the system is in exactly one of five 
different states:
- TASK_RUNNING: The process is runnable; it is either currently running or 
   on a runqueue waiting to run. This is the only possible state for a 
   process executing in user-space; it can also apply to a process in 
   kernel-space that is actively running.
- TASK_INTERRUPTBLE: The process is sleeping, waiting for some condition 
   to exist. 
- TASK_UNINTERRUPTIBLE: This state is identical to TASK_INTERRUPTBLE except 
   that it does not wake up and become runnable if it receives a signal. 
   (The case you cannot send it a SIGKILL signal)
- TASK_TRACED: The process is being traced by another process, such as a 
   debugger, via ptrace.
- TASK_STOPPED: Process execution has stopped. 
       
3.6 
All processes are descendants of the init process, whose PID is one. The 
kernel starts init in the last step of the boot process. The init process, 
in turn, reads the system initscripts and executes more programs, 
eventually completing the boot process.
      
3.7 
The fork() creates a child process that is a copy of the current task. 
exec() loads a new executable into the address space and begins executing.

3.8 
Copy-on-Write delays the copying of each page in the address space until 
it is actually written to. if exec() is called immediately after 
fork()-they never need to be copied.
       
3.9 
To the Linux kernel, there is no concept of a thread. Linux implements 
all threads as standard processes. The linux kernel does not provide any
special scheduling semantics or data structures to represent threads. 
Instead, a thread is merely a process that shares certain resources with
other processes. Each thread has a unique task_struct and appears to the 
kernel as a normal process-threads just happen to share resources, such
as an address space, with other processes.
       
3.10 
Kernel thread is used to perform some operations in the background. It
is standard processes that exist solely in kernel-space. Kernel threads
are schedulable and preemptable, the same as normal processes.

3.11
Indeed, a kernel thread can be created only by another kernel thread. The
kernel handles this automatically by forking all new kernel threads off
the kthreadd kenel process.

3.12
Generally, process destruction is self-induced. It occurs when the process
calls the exit() system call, either explicitly when it is ready to terminate
or implicitly on return from the main subroutine of any program. The C
complier places a call to exit() after main() returns. A process can also
terminate involunarily.
bulk of the work handled by do_exit():
- Set PF_EXITING flag in the flag member of the task_struct.
- Call del_timer_sync() to remove any kernel timers. 
- If process accounting is enabled, call acct_update_integrals() to write
  out accounting information.
- Call exit_mm() to release the mm_structure. If no other process is using the
  address space, the kernel will destroy it.
- Call exit_sem() if the process is queued waiting for an IPC semaphore.
- Call exit_files() and eixt_fs() to decrement the usage count of objects
  related to the file descriptors and filesystem data, respectively.
- Set exit_code member of task_struct for optional retrieval by the parent.
- Call exit_notify() to send signals to the task's parent, reparents any of
  the task's children to another thread in their thread group or the init 
  process, and sets the exit_state in task_struct to EXIT_ZOMBIE.
- call schedule() to switch to a new process. do_exit() never returns.

3.13
When the process is in EXIT_ZOMBIE status, the only memory it occupies is
its kernel stack, the thread_info, and task_struct structure. The task exists
solely to provide information to its parent. After the parent retrieves the 
information, or notifies the kernel that it is uninterested, the remaining
memory held by the process is freed and returned to the system for use.

3.14
When the time to finally deallocate the process descriptor, the release_task()
is invoked:
- Call __exit_signal(), which call __unhash_process(), which in turns calls
  detach_pid() to rmove the process from the pidhash and remove the process
  from the task list.
- __exit_signal() releases any remaining resources used by the now dead process
  and finalizes statistics and bookkeeping.
- Call put_task_struct() to free the pages containing the process's kernel stack
  and thread_info structure and deallocate the slab cache containing the task_
  struct.
 
Chapter 4. Process Scheduling

4.1
The process scheduler decides which process runs, when, and for how long.
Linux implements preemptive multitasking.

4.2
Processes can be classified as either I/O bound or processor bound.
The scheduling policy in a system must attempt to satisfy two conflicting goals:
fast process response time(low latency) and maximal system utilization(high 
throughput).
Linux optimizes for process response, thus favoring I/O-bound process.

4.3 process priority
The linux kernel implements two separate priority ranges.
The first is the nice value, from -20 to +19 with default 0, Large nice values
correspond to a lower priority. In Linux, nice is a control over the proportion
of of timeslice. 
The second range is the real-time priority, by default range from 0 to 99.
higher real-time priority values correspond to a greater priority.
A value of "-" means the process is not real-time.

4.4 timeslice
On linux, the nice value of process acts as a weight, changing the proportion
of the processor time each process recieves.

4.5 
The linux scheduler is modular, enabling different algorithms to schedule 
different types of processes. This modularity is called scheduler classes.
The Completely Fair Scheduler(CFS) is the registered scheduler class for normal
processes.

4.6
CFS will run each process for some amount of time, round-robin, selecting next
the process that has run the least. Rather than assign each process a timeslice,
CFS calculates how long a process should run as a function of the total number
of runnable processes. CFS imposes a floor on the timeslice assigned to each
process. This floor is called the minimum granularity. By default is 1 
millisecond.

4.7 The scheduler entity structure
CFS does not have the notion of a timeslice, but it must still keep account for
the time that each process runs. It uses struct sched_entity to keep track of 
process accounting, as a member variable of struct task_struct, named se.

4.8 vruntime
update_curr() is invoked periodically by the system timer and also whenever a
process becomes runnable or blocks, becoming unrunnable. 

4.9
CFS uses a reb-black tree to manage the list of runnable processes and 
efficiently find the process with the smallest vruntime - "run the process
represented by the leftmost node in the rbtree, cached by rb_leftmost".
CFS removes processes from the reb-black tree when a process blockes (becomes
unrunnable) or terminates (ceases to exist).

4.10
schedule() finds the highest priority scheduler class with a runnable process
and asks it what to run next.

4.11
The task marks itself as sleeping, puts itself on a wait queue, removes
itself from the reb-black tree of runnable, and calls schedule() to select
a new process to execute. Waking back up is the inverse: the task is set as
runnable, removed from the wait queue, and added back to the red-black tree.


4.12
Sleeping is handled via wait queues. A wait queue is a simple list of processes
waiting for an event to occur. Processes put themselves on a wait queue and mark
themselves not runnable. 

4.13
Context switching, the switching from one runnable task to another, is handled
by the context_switch() function in kernel/sched.c. It is called by schedule()
when a new process has been selected to run. It does two basic jobs:
- call switch_mm(), to switch the virtual memory mapping from the previous
process's to that of the new process.
- call switch_to(), to switch the processor state from the previous process's 
to the current's. This involves saving and restoring stack information and the
processor registers and any other architecture-specific state that must be
managed and restored on a per-process basis.
The kernel provides the need_resched flag to signify whether a reshedule should
be performed. The flag is a single bit of a special variable inside thread_info
structure.

4.14
User preemption occurs when the kernel is about to return to user-space,
need_resched is set, and the scheduler is invoked. It can occur:
- when returning to user-space from a system call
- when returning to user-space from an interrupt handler

4.15
The kernel can preempt a task running in the kernel so long as it does not 
hold a lock. Because the kernel is SMP-safe, if a lock is not held, the current
code is reentrant and capable of being preempted.
There is counter preempt_count to each process's thread_info, if preempt_count
is nonzero - a lock is held, it is unsafe to reschedule.
Kernel preemption can occur:
- when an interrupt handler exits, before returning to kernel-space
- when kernel code becomes preemptible again
- if a task in the kernel explicitly calls schedule()
- if a task in the kernel blocks(which results in a call to schedule())

4.16
Linux provides two real-time scheduling policies, SCHED_FIFO and SCHED_RR. 
The normal, not real-time scheduling policy is SCHED_NORMAL. Via the scheduling
classes framework, these real-time policies are managed not by CFS but a
special real-time scheduler. SCHED_RR IS SCHED_FIFO with timeslices. Both
real time scheduling policies implement static priorities. This ensures that
a real-time process at a given priority always preempts a process at a lower
priority.

Linux provide soft real-time behavior, refers to the notion that the kernel
tries to schedule applications within timing deadlines, but the kernel does
not promise to always achieve these goals.

4.17
The nice() function calls the kernel's set_user_nice() function, which sets 
the static_prio and prio values in the task's task_struct as appropriate.

4.18
A process only ever runs on a processor whose bit is set in the cpus_allowed
field of its process descriptor. sched_setaffinity() syscall could change it.

4.19
In user space, sched_yield() system call is a mechanism for a process to 
explicitly yield the processor to other waiting processes.  
In kernel code, call yield(), which ensures that the task's state is 
TASK_RUNNING and then call sched_yield().

4.20
A large number of runnable processes, scalability concerns, trade-offs between
latency and throughput, and the demands of various workloads make a one-size-
fits-all algorithm hard to achive. 



Chatper 5 system calls

5.1
System calls provide a layer between the hardware and user-space processes.
- Provides an abstracted hareware interface for user-space.
- Ensure system security and stability.
- A single common layer between user-space and the rest of the system allows
  for the virtualized system provided to processes. System calls are the only 
  means user-space has of interfacing with the kernel, are the only legal
  entry point into the kernel other than exceptions and traps.
  
5.2
A meme related to interfaces in Unix is "Provide mechanism, not policy".
In other words, Unix system calls exist to provide a specific function in an
abstract sense. The manner in which the function is used is not any of the
kernel's business.

5.3
The C library, when a system call returns an error, writes a special error
code into the global errno variable. This variable can be translated into 
human readable errors via library functions such as perror().

5.4
asmlinkage long sys_getpid(void)
asmlinkage is a directive to tell the compiler to look only on the stack for 
this function's arguments, p.s by default via registers. The function returns 
long for compatibility between 32 and 64 bit systems.

5.5
The kernel keeps a list of all registered system calls in the system call table,
on x86-64 it is defined in arch/x86/kernel/syscall_64.c.

5.6
The mechanism to signal the kernel is a software interrupt: Incur an exception,
and the system will switch to kernel mode and execute the exception handler.
The exception handler, in this case, is actually the system call handler.
The defined software interrupt on x86 is interrupt number 128, i.e 0x80.
On x86, the syscall number is fed to the kernel via the eax register.

5.7 syscall parameter passing
Most syscalls require that one or more parameters be passed to them. On x86-32,
the registers ebx, ecx, edx, esi and edi contain, in order, the first five 
arguments. If there are six or more arguments, a single register is used to
hold a pointer to user-space where all the paremeters are stored.
on x86, the return value is written to eax register.

5.8
When you write a system call, you need to realize the need for portability and
robustness, not just today but in the future. 
Every parameter must be checked to ensure it is not just valid and legal, but 
correct. The kernel code msut never blindly follow a pointer into user-space.

5.9
For writing into user-space, the method copy_to_user() is provided. It takes
three parameters. The first is the destination memory address in the process's
address space. The second is the source pointer in kernel-space. The third is 
the size in bytes of the data to copy. copy_from_user() is analogous to 
copy_to_user(). Both of them may block, which occurs if the page containing
the user data is not in physical memory but is swapped to disk. In that case,
the process sleeps until the page fault handler can bring the page from the 
swap file on disk into physical memory.

5.10
The kernel is in process context during the execution of a system call. The
current pointer points to the current task, which is the process that issued
the syscall. In process context, the kernel is capable of sleeping and is fully
preemptible. 
- Interrupt handlers cannot sleep and thus are much more limited in
  what they can do than system calls running in process context.
- Because the new task may then execute the same system call, care must be
  exercised to ensure that system calls are reentrant.
When the system call returns, control continues in system_call(), the syscall 
handler, which ultimately switches to user-space and continues the execution
of the user process.

5.11
The steps in binding a system call - foo():
- Add sys_foo() to the system call table, like entry.S file: ".long sys_foo"
- Add system call number to <asm/unistd.h>: "#define _NR_foo 338"
- add actual foo() in kernel/sys.c. "asmlinkage long sys_foo(void) {return
  THREAD_SIZE;}"

5.12
One alternative of adding a new system call is implementing a device node and
read() and write() to it. Use ioctl() to manipulate specific settings or 
retrieve specific information.


Chapter 6 Kernel data structures

6.1
What data structure to use, when
If your primary access method is iterating over all your data, use a linked list
If your code follows the producer/consumer pattern, use a queue.
If you need to map a UID to an object, use a map.
If you need to store a large amount of data and look it up efficiently, consider
a red-black tree. If you are not performing many time-critical look-up 
operations, a red-black tree probably isn't your best bet.
Only after exhausting all kernel-provided solutions should you consider
"rolling your own" data structure.

Chapter 7. Interrupts and Interrupt Handlers

7.1
An interrupt is physically produced by electronic signals originating from the
hardware devices and directed into input pins on an interrupt controller, a 
simple chip that multiplexes multiple interrupt lines into a single line to
the processor. The processor detects this signal and interrupts its current
execution to handle the interrupt.

7.2
Different devices can be associated with different interrupts by means of a 
unique value associated with each interrupt, these values are called interrupt
request (IRQ) lines. on the classic PC, IRQ zero is the timer interrupt and IRQ
one is the keyboard interrupt. 

7.3
Exceptions are produced by the processor while executing instructions either
in response to a programming error (i.e divide by zero) or abnormal conditions
that must be handled by the kernel(i.e a page fault). 
interrupts are asynchronous interrupts generate by "hardware", with respect to
the processor clock. 
exceptions are synchronous interrupts generated by the "processor".

7.4
The interrupt handler for a device is part of the device's driver - the kernel
code that manages the device, which run in "interrupt context".

7.5
These two goals-that an interrupt handler execute quickly and perform a large
amount of work-clearly conflict with on another. Because of these competing 
goals, the processing of interrupts is split into two parts, or halves.
The interrupt handler is the top half.
When network cards receive packets from the network, the interrupt runs,
acknowledges the hardware, copies the new networking packets into main memory,
and readies the network card for more packets. After the networking data is
safely in the main memory, the interrupt's job is done, and it can return 
control of the system to whatever code was interrupted when the interrupt was
generated. The rest of the processing and handling of the packets occurs later,
in the bottom half. 

7.6
request_irq() can sleep/block, so cannot be called from interrupt context or 
other situations where code cannot block. On registration, an entry 
corresponding to the interrupt is created in /proc/irq. The function proc_mkdir
() creates new procfs entries. This function calls proc_create(), which in turn
call kmalloc() to allocate memory. kmalloc() can sleep. So there you go!
A call to free_irq() must be made from process context.

7.7
The declaration of an interrupt handler:
static irqreturn_t intr_handler(int irq, void *dev)
Interrupt handlers in Linux need not be reentrant. When a given interrupt 
handler is executing, the corresponding interrupt line is masked out on all
processors, preventing another interrupt on the same line from being received.
Normally all other interrupts are enabled, so other interrupts are serviced, 
but the current line is always disabled. 


7.8
Interrupt context, different from process context, is not associated with a 
process. Interrupt context cannot sleep, therefore, you cannot call certain
functions from interrupt context. 
Historically, interrupt handlers did not receive their own stack, instead,
they would share the stack of the process that they interrupted. 
(A process is always running. When nothing else is schedulable, the idle
task runs) The kernel stack is two pages in size; so the interrupt handlers
share the stack, they must be exceptionally frugal with what data they allocate
there.

7.9
/proc/interrupts is populated with statistics related to interrupts on the 
system.

7.10
Reasons to control the interrupt system generally boil down to needing to 
provide synchronization. The lock provides protection against concurrent access
from another processor, whereas disabling interrupts provides protection against
concurrent access from a possible interrupt handler.

7.11
irqs_disabled() returns nonzero if the interrupt system on the local processor
is disabled. in_interrupt() and in_irq() provide an interface to check the 
kernel's current context.


Chapter 8. Bottom Halves and Deferring Work

8.1 top v.s bottom half
- If the work is time sensitive, perform it in the interrupt handler.
- If the work is related to the hardware, perform it in the interrupt handler.
- If the work needs to ensure that another interrupt does not interrupt it, 
  perform it in the interrupt handler.
- For everything else, consider performing the work in the bottom half.

8.2
Often, bottom halves run immediately after the interrupt returns. The key is
that they run with all interrupts enabled.

8.3
Softirqs are a set of statically defined bottom halves that can run 
simultaneously on any processor; even two of the same type can run concurrently.
Tasklets are dynamically created bottom halves built on top of softirqs.
two of the same type of tasklet cannot run simultaneously. Softirqs are useful
when the performance is critical, such as networking, softirqs must be registed
statically at compile time. Conversely, code can dynamically register tasklets.

8.4
Softirqs are reserved for the most timing-critical and important bottom-half
processing on the system. In kernel 2.6, only two subsystems - networking and
block devies - derictly use softirqs.

8.5
Tasklets have a simpler interface and relaxed locking rules. As a device driver
author, the decision whether to use softirqs versus tasklets is simple: You 
almost always want to use tasklets. All tasklets are multiplexed on top of two
softirqs, HI_SOFTIRQ and TASKLET_SOFTIRQ.
As with softirqs, tasklets cannot sleep. This means you cannot use semaphores
or other blocking functions in a tasklet.

8.6
Work Queues defer work into a kernel thread - this bottom half always runs in
process context. If the deferred work needs to sleep, work queues are used.
The work handlers cannot access user-space memory because there is no associated
user-space memory map for kernel threads. The kernel can access user memory only
when running on behalf of a user-space process, such as when executing a system
call. Only then is user memory mapped in.

Chapter 9. An introduction to Kernel Synchronization

9.1
It is a bug if it is possible for two threads of execution to be simultaneously
executing within the same critical region. We call it a race condition, so-named
because the threads raced to get there first. Ensuring that unsafe concurrency 
is prevented and that race conditions do not occur is called synchronization.

9.2
Process preemption with same critical region(share memory, file descriptor),
or within a single program with signals, because signals can occur 
asynchronously. This type of concurrency - in which two things do not actually
happen at the same time but interleave with each other- is called pseudo-
concurrency.
On a symmetrical multiprocessing machine, two processes can actually be executed
in a critical region at the exact same time, this is called true concurrency.
Both kinds of concurrency can result in the same race conditions and require the
same sort of protection.

9.3
The kernel has similar causes of concurrency:
Interrupts, Softirqs and tasklets, Kernel preemption, Sleeping and 
synchronization with user-space, symmetrical multiprocessing.

9.4
Implementing the actual locking in your code to protect shared data is not
difficult, especially when done early on during design phase of development.
The tricky part is identifying the actual shared data and the corresponding 
critical sections. interrupt-safe, smp-safe, preempt-safe.

9.5
Lock data, not code.
Whenever you write kernel code, ask yourself these questions:
Is the data global?
Is the data shared between process context and interrupt context? Is it shared
between two different interrupt handlers?
If a process is preempted while accessing this data, can the newly scheduled
process access the same data?
Can the current process sleep(block) on anything? If it does, in what state does
that leave any shared data?
What prevents the data from being freed out from under me?
What happens if this function is called again on another processor?

9.6
A deadlock is a condition involving one or more threads of execution and one or
more resources, such that each thread waits for one of the resources, but all 
the resources are already held. The threads all wait for each other, but they 
never make any progress toward releasing the resources that they already hold.

