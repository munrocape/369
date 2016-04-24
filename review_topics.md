# Final Review Topics

*An attempt to address the topics mentioned in last lecture*

#### What is an operating system?

- An operating system is the most important software that runs on a computer. It
manages the computer's memory, processes, and all of its software and hardware.

- Two main goals of the OS is convenience for the user, and efficient operation
of the computer system.

## Processes & Threads

#### What is a process? What is a thread?

- A process is an instance of a computer program that is being executed. It
contains the program code and its current activity. Depending on the operating system (OS), a process may be made up of multiple threads of execution that execute instructions concurrently.

- A thread of execution is the smallest sequence of programmed instructions
that can be managed independently by a scheduler, which is typically a part
of the operating system.[1] The implementation of threads and processes
differs between operating systems, but in most cases a thread is a component
of a process. Multiple threads can exist within one process, executing
concurrently (one starting before others finish) and share resources such as
memory, while different processes do not share these resources. In
particular, the threads of a process share its instructions (executable code)
and its context (the values of its variables at any given time).

#### What is the difference between user-level threads and kernel-level threads?

- Kernel-level threads are managed by the OS
- User-level threads are managed entirely by the run-time system (user-level
library)

##### User-level threads are small and fast

- A thread is simply represented by a PC, registers, stack, and small thread
control block (TCB)
- Creating a new thread, switching between threads, and synchronizing
threads are done via procedure call (no kernel involvement)
- User-level thread operations are up to 100x faster than kernel threads
- But this depends on the quality of both implementations!

*Note: See L2 slides for U/L thread limitations*

#### How are new processes created? Deleted? Zombies?

- In Unix, processes are created using fork() system call `int fork()`
- `exit()` causes a process to voluntarily release all it's resources -
although OS doesn't necessarily toss everything immediately. The process must
be stopped in order to free all its resources. There must also be a *context
switch* to another process. Additionally the parent may be waiting or asking
for the return value.

##### Zombies!

When a process exits most of resources are deallocated - the address space is
freed, files are closed etc. However, there are some OS data structures which
return the process's exit state. The process retains it's id (`PID`) and
remains a **zombie** until its parent cleans it up.

##### fork()
- Creates and inits a new PCB (Process Control Block)
- Creates a new address spaces and inits it with a copy of the entire contents
of the address space of the parent
- Inits the kernel resources to point to the resources used by parent.
- Fork returns **twice** -> the child PID to the parent and `0` to the child

- A process must always be created by another process.

#### What does the address space look like? PCB?

A program layout in memeory:

![Program Layout in Memeory](/images/memoryModel.png)

The **PCB** (Process control block) is an OS data structure, usually a struct.
Which includes:

- process state (ready, running, blocked ...)
- program counter: address of the next instruction
- CPU registers: must be saved at an interrupt
- CPU scheduling info: process priorities
- memory management info: page tables
- accounting info: resource usage
- I/O status info: list of open files

#### What states can a process be in?

*The OS maintains a collection of queues that represent the state of all
processes in the system. Each PCB is queued on a state queue according to its
current state As a process changes state, its PCB is unlinked from one queue
and linked into another.*

![Process States](/images/processStates.png)

#### How do threads relate to virtual address spaces?

- A process includes address space and execution state. Separate them, then
multiple **threads** can execute in a single address space.

- Why?
1. Sharing - share code, heap, and globals
2. Lighter weight - faster to create/destroy, faster context switching
3. Concurrent programming gains - overlapping computation and I/O

![Multithreaded address space](/images/multiAddress.png)

## System Calls

#### What are the protection domains? Why do we need them?

*Note: There may actually be more depending on the setup, these are the two
standard modes*

- Hardware can run in two modes: **user** or **system**. Some instructions are
"privileged" and can only run in **system** mode.

##### Privileged instructions (enforced by CPU hardware):

1. Access I/O device - poll for IO, catch hardware interrupt
2. Manipulate memory management - page tables, flush TLB and CPU caches
3. Configure mode bits - interrupt priority level, software trap vectors etc..
4. Call halt instruction - put CPU in low power or idle until interrupt

#### How do interrupts work? Why do we need them?

- They can be caused by hardware or software.
- They signal the CPU that a hardware device has an event that needs attention
(e.g. disk I/O completes)
- They signal errors or requests for OS intervention (system calls), this is
often called an "exception" or "trap"
- The way they work is: after an interrupt is triggered, the CPU jumps to a
pre-defined routine known as the **interrupt handler**
- An OS is an event-driven program, in other words, the OS "repsonds" to
requests.

- We need interrupts for the reasons mentioned above.

#### What happens when a process makes a system call?

*System call == a function call that invokes the OS*

1. When the interrupt for the system call occurs, there reason for the
interrupt is stored in a register, that register is used to invoke a specific
**handler** which is just a function.
2. The mode bit is first switched to allow "privileged" instructions to occur.
3. Special instructions are executed tp trap to system mode
4. Syscall handler figures out which system call is needed and calls a routin
for that operation
5. Return of system call is returned in register **EAX**.

#### How and when does a context switch happen?

- A context switch means the CPU switches to running a different process. 

##### It happens by:

1. Saving the state of the old process, and
2. loading the saved state for the new process

##### It happens when:

1. a process calls the `yield()` system call (voluntarily)
2. a process makes other system call and is blocked
3. timer interrupt handler decides to switch processes - needed for scheduling! 

## Concurrency

#### What is the critical section problem?

- The critical section problem is the need to enforce singular use of a shared
resource (e.g. kprintf_lock ensures that only one thread can print output to
the console at a time)


#### What properties does a solution (to the crit section prob) need to have?

1. **Mutual exclusion** - one thread in the CS and no others.
2. **Progress** - Only threads not in "remainder" section can influence the choice
of which thread enters, and choice cannot be postponed forever. Essentially, if
no threads are in the CS, and one wants to enter, it should be able to do so
without being restricted by threads in the "remainder"
3. **No Starvation** - if some thread `T` is waiting on the CS, then there is a
limit on the number of times other threads can enter the CS before this thread
is granted access (no one thread hogs the CS).
4. **Performance** - the overhead of enter/exit the CS is small compared to
work being done within it.

#### What is a race condition?

- If two concurrent threads manipulate a **shared resource** without any
synchronization, then output will depend on hte order in which the accesses
happen, hence "race". So we *need* synchronization.

#### Synchronization primitives

##### S/W solutions (no H/W support): Peterson’s algorithm
- Threads share variables `turn` and `flag` where `flag` is an array

1. Set own `flag` (to indicate interest) and set `turn` to self
2. Spin waiting while turn is self **AND** other flag set (is interested)
3. If both threads try to enter their CS at the same time, `turn` will be set
to both 0 and 1 at roughly the same time. Only one of these assignments will
stick. The final value of `turn` decides which of the two threads is allowed
to enter its CS first.

*Peterson’s algo can be extended to support `N` threads*

##### Bakery algorithm!
1. Upon entering each customer (thread) gets a number/ID
2. The customer with the lowest number is served next
3. However, no guarantee that 2 threads do not get the same number
- In the case of a tie, thread with lowest ID is served first
- Thread ID's must be unique and totally ordered

- Does Bakery meet all three criteria? (mutual ex, progress, starv) Not sure...

##### Hardware instructions
*Note: on a uniprocessor the OS must disable interrupts before entering the
critical section (to prevent context switches). However, disabling interrupts
is insufficient on a multiprocessor, whyyyyyy?*

**Test‐And‐Set:** (hardware executes atomically)
1. Record the old value of the variable.
2. Set the variable to some non-zero value.
3. Return the old value.

Used for simple lock variables:
``` C
boolean test_and_set(boolean *lock) {
    boolean old = *lock;
    *lock = true;
    return old;
}
```

So either lock was `true` (locked) already or `false` (available) but now the
caller holds it, so it comes back true either way from test-and-set.

**Swap/Exchange:**
- Operates on two words atomically
- Can also be used to solve CS problem

Example:
``` C
// in each thread
boolean key = true; // local to each thread
while(key) Swap(&lock, &key); // ENTRY

// Critical section
Swap(&lock, &key); // EXIT
```

**Locks:**
A spinlock ("busy waiting" because it executes continually until it gets the
lock):
``` C
boolean lock;

void acquire(boolean *lock) {
    while(test_and_set(lock));
}

void release(boolean *lock) {
    *lock = false;
}
```

A sleeplock:
- Instead of spinning, just puts the thread to sleep (becomes "blocked") while
waiting for a lock.
- Requires a queue of waiting threads, sometimes called a "wait channel"

```C
wait_event(queue, condition);
wake_up(wait_queue_head_t *queue);
```

**Semphores:**

*Semphore is like a bouncer at a club lineup*

Includes:
- a counter variable (accessed through 2 atomic ops)
- the atomic op `wait`, also called `P` or `Decrement`, this decrements the
counter variable and blocks until the semaphore is free
- the atomic op `signal`, also called `V` or `increment` - increments the
counter, unblocks a waiting thread if there are any
- A queue of waiting threads

Different Types:

1. Mutex (binary): single resource access, mutual exclusion to a CS
2. Counting semaphore:
    1. A resource with many units available, or a resource that allows certain
    kinds of unsynchronized concurrent access (e.g. reading from a file)
    2. Multiple threads can pass the semaphore
    3. Max threads determine by semaphore's initial value, `count` (a mutex
    has `count = 1`, a counting sem has `count = N`)

``` C
#include <semaphore.h>
sem_t mysem;
sem_init(&mysem, 0, VALUE);
sem_wait(&mysem); // s.wait or P()
sem_post(&mysem); // s.signal or V()
```

*See L3 slides, and tutorial/lecture exercises for example of semaphore
problems*


**CVs:**
- Stronger semantics than locks or semaphores. Used if you want a more complex
wait condition than the other two.

- Abstract data type that encapsulates pattern of “release mutex,
sleep, re-acquire mutex”
- Internal data is just a queue of waiting threads
    - Recall we didn’t need the semaphore count
- Operations are (each of these is atomic):
    - `cv_wait(struct cv *cv, struct mtx *mutex)`
    - Releases lock, waits, re-acquires mutex before return
    - `cv_signal(struct cv *cv)`
    - Wake one enqueued thread
    - `cv_broadcast(struct cv *cv)`
    - Wakes all enqueued threads

*CAUTION: if no one is waiting, signal or broadcast has no effect. Not recorded
for later use, as with semaphore V().*

- CVs must be used with locks to protect the shared data which is
modified/tested

Example:
``` C
lock_acquire(lock);
    while(condition not true) {
    cv_wait(cond, lock);
}
... // do stuff
cv_signal(cond); //or cv_broadcast(cond)
lock_release(lock);
```


**Monitors:**

- an abstract data type (data and operations on the data) with the
restriction that only one process at a time can be active within the
monitor
    - Monitor enforces mutual exclusion
    - Local data accessed only by the monitor’s procedures (not by any
    external procedure)
    - A process enters the monitor by invoking one of its procedures
    - Other processes that attempt to enter monitor are blocked

![Monitor diagram](/images/monitor.png)

- Monitors can have **HOARE** or **MESA** semantics.
    - HOARE: `signal()` immediately switchs from the caller to a waiting
      thread, the condition that the waiter was blocked on is guaranteed to
      hold when the waiter resumes. Need another queue for the signaler, if
      signaler was not done using the monitor.
    - MESA: `signal()` places a waiter on the ready queue, but signaler
    continues inside monitor. Condition is not necessarily true when waiter
    resumes. So must check condition again.

- HOAREs are easier to reason about
- MESAs are just easier...to implement, more efficient, and can support
additional ops like `broadcast`.

#### Revisit synchronization problems!
- Producer‐Consumer, Readers‐Writers problems from tutorials/lectures

#### How would you write a monitor (see last tutorial too!!)

*Note: add/remove functions take a pointer to the buffer to
operate on, so we can have multiple buffers.
Producers would call “add_to_buff(&buffer, newvalue);”
Consumers would call “remove_from_buff(&buffer);”*

``` C
#define N 100
typedef struct buf_s {
    int data[N]; /* buffer storage */
    int inpos; /* producer inserts here */
    int outpos; /* consumer removes from here */
    int numelements; /* # items in buffer */
    struct lock *buflock;/* access to monitor */
    struct cv *notFull; /* for producers to wait */
    struct cv *notEmpty; /* for consumers to wait */
} buf_t;

buf_t buffer;

void add_to_buff(buf_t *b, int value) {
    lock_acquire(b->buflock);
    while (b->nelements == N) {
        /* buffer b is full, wait */
        cv_wait(b->notFull, b->buflock);
    }
    b->data[b->inpos] = value;
    b->inpos = (b->inpos + 1) % N;
    b->nelements++;
    cv_signal(b->notEmpty, b->buflock);
    lock_release(b->buflock);
}

int remove_from_buff(buf_t *b) {
    int val;
    lock_acquire(b->buflock);
        while (b->nelements == 0) {
            /* buffer b is empty, wait */
            cv_wait(b->notEmpty, b->buflock);
        }
        val = b->data[b->outpos];
        b->outpos = (b->outpos + 1) % N;
        b->nelements--;
        cv_signal(b->notFull, b->buflock);
    lock_release(b->buflock);
    return val;
}

void add_to_buff(buf_t *b, int value);
int remove_from_buff(buf_t *b);
```

Diagram of possible MESA execution:

![MESA diagram](/images/MESA.png)

Extra notes on monitors:

Bounded buffer example illustrates pattern for using locks and
condition variables in monitors:

1. The first step in any function is to acquire the lock on the monitor
2. The last step before returning is to release the lock on the monitor
    - Be careful with early returns due to errors!
3. Call cv_wait() only in while loops
4. Whenever one of the conditions being waited on might have changed
from FALSE to TRUE, signal the corresponding condition variable
    - Signaller need not know previous state, only that current state is TRUE

**SEE L4 slides for a full example of a monitor implementation**

## Scheduling

#### Goals in developing a “good” scheduling algorithm

- On all systems:
    - Fairness: each thread should get its fair share of CPU
    - Avoid starvation
    - Policy enforcement: usage policies should be met
    - Balance: all parts of the system should be busy

- On "batch" systems:
    - Throughput: maximize jobs completed per hour
    - Turnaround time: minimize time between submission and completion
    - CPU utilization: keep it busy at all times.

- Interactive Systems
    - Response time - minimize time between receiving request
    and starting to produce output
    - Proportionality - “simple” tasks complete quickly
- Real-time systems
    - Meet deadlines
    - Predictability

*Goals sometimes conflict with each other!*

*See L5 Slides for more details about types of scheduling non-premptive
vs preemptive etc...*

#### Know the properties of different algorithms we discussed

##### FCSF
"First come, first served"

- Is non-premptive
- Choose thread at head of ready queue
    - queue is maintained in FIFO order
- Average wait time with FCFS is often quite long
    - There is a convoy effect that happens: all threads wait for the one big
    thread to release the CPU

##### SJF
"Shortest job first"

- The pre-emptive version is "shortest remaining time"
- Choose thread with the shortest expected prcessing time. Based on:
    - Programmer estimate
    - History stats
    - Can be shortest-next-CPU-burst for interactive jobs
- Proably optimatal w.r.t. to average wait time.

##### RR
"Round Robin"

- Pre-emptive
- Designed for time-sharing systems
- Ready queue is circular
    - Each threead is allowed to run for time quantum `q` before being
    preempted and put back on queue.
- Choice of quantum (aka time slice) is critical
    - as `q` -> inf, RR -> FCFS; as `q` -> 0, RR -> processor sharing (PS)
- Want `q` to be large w.r.t the context switch time - otherwise there is too
much overhead

![Round robin diagram](/images/RR.png)

##### Priority scheduling

- A priority, p, is associated with each thread
- Highest priority job is selected from Ready queue
    - Can be pre-emptive or non-preemptive
- Enforcing this policy is tricky
    - A low priority task may prevent a high priority task from
making progress by holding a resource (priority inversion)
    - A low priority task may never get to run (starvation)

*Mars rover is example of this going wrong (see slides L5)*

##### MLFQ (Multi level feedback queue)

- Have multiple ready queues
    - each runnable thread is on only one queue
- Threads are permanently assigned to a queue
    - Critera include job class, priority, etc..
- Each queue has its own scheduling algo
    - Usually just RR
- Another level of scheduling decides which queue to choose next
    - Usuaully priority-based

Feeback Scheduling:
- Adjust criteria for choosing a particular thread based on past
history
    - Can boost priority of threads that have waited a long time (aging)
    - Can prefer threads that do not use full quantum
    - Can boost priority following a user-input event
    - Can adjust expected next-CPU-burst
- Use a multi-level queue (MLQ) to move threads between queues
    - Use “feedback” to decide when to move to other queues
    - This is called multi-level feedback queue (MLFQ) – ACM Turing Award!

![MLFQ](/images/multiLevel.png)

#### Which schedulers are best suited for different workloads?

- Long-term scheduling
    - aka admission scheduling, admission control
    - Used in batch systems, not common today
- Medium-term scheduling – happens infrequently
    - aka memory scheduling
    - Decides which processes are swapped out to disk
    - We’ll talk about this later with memory management
    - Sometimes called “long-term” with admission control ignored
- Short-term scheduler – happens frequently
    - aka dispatching
    - Needs to be efficient (fast context switches, fast queue manipulation)


#### Which schedulers may cause starvation?

- Priority scheduling - a low prio may never get run
- Shortest job first? - a long job may never get run?

#### How can process properties (compute‐bound, I/O bound) be used by schedulers?







## Memory Management - overview

#### Why is memory management important?


#### What are the goals of virtual memory?


#### Why do we have virtual memory if it is so complex?


#### What are the mechanisms for implementing mem management?
- Physical and virtual addressing
- Partitioning types, paging
- Page tables, TLB


#### What are the overheads related to providing memory management?


#### What are the policies related to MM?


#### Page replacement


## Virtualizing Memory

#### What is the difference between a physical and virtual address?


#### What is the difference between fixed and variable partitioning?
• How do base and limit registers work?


#### What is internal fragmentation?


#### What is external fragmentation?


#### What is a protection fault?


## Paging

#### How is paging different from partitioning?

#### What are the advantages/disadvantages of paging?

#### What are page tables?

#### What are page table entries (PTE)?

#### What are all of the PTE bits used for?

#### Know these terms really well
- Virtual page number (VPN), page frame number (PFN), offset

#### Know how to break down virtual addresses into page numbers, offset
- And how to translate virtual to physical


## Page Tables

#### Page tables introduce overhead
- Space for storing them
- Time to use them for translation
- What techniques can be used to reduce their overhead?

#### How do linear/multi‐level page tables work? (Exercises!)

#### Know the terminology : PTE, PDE, PTBR, PDBR, etc.

#### Translate manually between hexadecimal, binary, decimal then work with them
- Hex digits: 4 bits, 0‐9, a‐f
- You must be able to do this to do well in the 0x171 exam!
- Practice! No calculators on the exam


## TLBs

#### What problem does the TLB solve?

#### How do TLBs work?

#### Why are TLBs effective?

#### How are TLBs managed?

#### What happens on a TLB miss fault?

#### What is the difference between a hardware and software managed TLB?


## Page Faults

#### What is a page fault?

#### How is it used to implement demand paged virtual memory?

#### What is the complete sequence of steps? 
- Go from a TLB miss to paging in from disk, for translating a virtual address
to a physical address?
- What is done in hardware, what is done in software?


## Page Replacement

####What is the purpose of the page replacement algorithm?

####What application behavior does page replacement try to exploit?

####When is the page replacement algorithm used?

####Refresh these thoroughly
- Belady’s (OPTimal), FIFO, LRU, LRU, Clock (second chance), Belady’s anomaly,
Working Set model, Page Fault Frequency


## Advanced mem management

#### What is thrashing? Possible solutions?

#### Multiprogramming correlation with CPU utilization?

#### What is shared memory?

#### What is copy on write?

#### How does the operating system leverage virtual memory to provide these?


## File Systems

#### Some Topics
- Files
- Directories
- Sharing
- Protection
- Layouts
- Buffer Cache

#### What is a file system?

#### Why are file systems useful (why do we have
them)?

## Files and Directories

#### What is a file?
- What operations are supported?
- What characteristics do they have?
- What are file access methods?

#### What is a directory?
- What are they used for?
- How are the implemented?
- What is a directory entry?

#### How are directories used to do path name translation?

#### What is a hard link? A symbolic link?

#### What’s the content of a file/directory/link?


## File System Layouts

#### What are file system layouts used for?
#### What are the general strategies?
- Contiguous, linked, indexed?

#### What are the tradeoffs for those strategies?
- In what special circumstances might you prefer a method that is not
suitable for a general purpose file system? (e.g. contiguous allocation)

#### What is an inode?
- How are inodes different from directories?
- How are inodes and directories used to do path resolution, find files?

#### Refresh A3
- What’s the structure of inodes, directory entries, superblock, etc.?


## Even more FS concepts

#### How do we go about building a file system? VSFS..
- What are the levels of indirection in inodes?
- Advantages/disadvantages of extent‐based approach?

#### Performance optimizations
- What’s the file buffer cache, and why do operating systems
use one?
- Why is buffering writes useful?
- What is a major tradeoff when it comes to caching and
buffering?
- What is read ahead and why is it important?


## Disk

#### Understand the memory hierarchy concept, locality

#### Physical disk structure
- Platters, surfaces, tracks, sectors, cylinders, arms, heads
- What are some hardware optimizations?

#### Disk performance
- What steps determine disk request performance?
- What are seek, rotation, transfer?
- Why try to allocate related data close together?
- What is FFS, and how is it an improvement over the
original Unix file system?


## Disk Scheduling

#### How can disk scheduling improve performance?
- What are the components of disk access time?
- What effect does disk scheduling have on each?

#### What are the issues in disk scheduling?
- Response time, throughput, fairness

#### Refresh disk scheduling algorithms
- FCFS, SSTF, SCAN, C‐SCAN, LOOK, C‐LOOK


## Advanced FS Topics

#### Theme: crash consistency, optimizing writes, redundancy

#### What are some crash recovery mechanisms?
• What is fsck? What are its limitations?

#### How is journaling performed in modern file systems like ext3?

#### Advantages (of ext3) over fsck? Metadata journaling?

#### What is LFS? What was the key idea in its design? Advantages and drawbacks?

#### Can we handle complete disk crashes? What’s the idea behind RAID?

#### SSDs (probably not on the exam though, but highly relevant)


## Deadlocks

#### What is the definition of a deadlock?

#### What are the conditions for deadlock?

#### What is Deadlock Prevention/Avoidance/Detection & recovery?

#### How does the Banker’s Algorithm work? Safe states? (again, revisit all tutorial exercises!)

#### What is a resource allocation graph, what is it used for?

#### No questions on concurrent transactions on this topic