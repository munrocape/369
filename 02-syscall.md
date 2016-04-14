## System calls

When we want to do some OS service, we need to use a system call (remember privileged instructions, mode bit, ...)

### Calling a function

1. Before calling subroutine, caller saves EAX, ECX, EDX. Since subroutine can modify these. 
2. Passing args is done by pushing them onto the stack (last parameter first). Stack grows down - so first arg has lowest address
3. Expect return value to be in EAX.

When it returns, you yank the args from the stack. return things to before you called. restore EAX, ECX, EDX.

### System call

A system call is a function that calls the OS.

OS functionality is protected with interrupts.

Hardware interrupts: IO completion, ...

Interrupts trap to handler - jump to the OS to handle the interrupt.

`Q: What should the OS save when it goes to this handler?`

On system call interrupt, we toggle the mode bit to system.

The reason (or call) is stored in a register that is passed to the handler.

When we define sys calls, its like (NAME, arg type 1, arg type 1 name, type 2, name 2, ...)

SYSCALL_DEFINE#NUMARGS(name, 1, 1, 2, 2, 3, 3, .., n, n)
asmlinkage long sys_name(1, 2, .., n)

### Dispatch

1. Kernel assigns system calls a unique number (pointer in a table)
2. User proc sets up sys call number and args
3. User proc runs int N (0x80 on Linux)
4. Hardware switches to kernel mode, invokes handler, gets proper function, passes args
5. Kernel returns iret (interrupt return)

If you need more than 6 parameters, the 6th arg is a pointer to a struct that contains rest of them

But we could do some nasty things as a user, so we use safe copying and the like (copy_from_user(), copy_to_user())

## Threads

Proc is independent iff it cant be affected or affect other procs (no data sharing)

Cooperating iff not independent

- shared memory (fork, ...)
- message passing (args, ...)

Message passing types:

1. Direct: send, receive, pipes
2. Indirect: send/receive ports
3. System calls & data copying: costly

Forking can be costly in shared memory. We need a full copy of the proc to do something.

Instead, use a thread.

For a thread, you separate the address space from the execution state. Multiple threads can execute within one address space.

SO threads 

- lighter weight
- share code, heap, globals
- faster context switch times (potentially)

`thread:` single control flow through a program

If it is multithreaded, there are multiple control flows.

You split the stack up with buffers and have n SPs.

Modern OSes develoed tasks/lightweight processes to make them even more efficient. Thread operations implemented in the kernel.

This has some limitations: generalized to support all needs, for faster concurrency need cheaper threads

So we need them at the user level.

- thread is PC, registers, stack, and TCB (thread control block)
- procedure calls handle switching between threads, creating, synchronizing
- 100x faster than kernel threads

Limitations:
- invisible to the OS. so not easily optimized
- scheduling a proc with only idle threads 
- blocking a proc whose thread initiaed IO, even though other threads in that proc are ready
- descheduling a proc with a thread holding a lock

You could multiplex user level threads on top of kernel level threads, or associate user level thread with kernel level thread.

### PTHREADS (posix threads) library

`pthreads` lib specifies the interface, not implementation, of threads.











