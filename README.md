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




