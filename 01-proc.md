### Goals of an OS

1. Primary: convenience for the user
2. Efficient operation

Sometimes these goals are contradictory (which one takes precidence depends on context)

### Roles of the OS

1. Virtual machine (extends, simplifies interface to physical machine)
2. Resource allocator (provides environment for multiple programs to use hardware/software/data)
3. Control program (controls execution, prevents errors, ensures safety)

### Guts of the OS

1. Software (User apps)
2. Software (OS: synchronization, scheduling, file system, memory management, ...)
3. Hardware (CPU, busses, I/O, ...)

### Storage

Processor registers and main memory are the rudimentary memory.

The hierarchy is based on speed and cost (and volatility).

Caches are helpful to hide performance differences.

From small (& expensive) to big (& cheap): L1 cache, L2/L3, main memory, disk, tape

### Themes of an OS

1. Virtualization (turn physical resources into easy to use forms, multiple copies of one resource, ...)
2. Concurrency (coordinate multiple procs)
3. Persistence (survive crashes, power cord trippings, ...)

### How to virtualize (well)

JVM is slow. But it acomplishes virtualization.

We want to use _limited direct execution_

Direct execution is taking a program, getting the next instruction, and running it.

The limited part is we don't just let it do whatever it wants. 

### Difference between program and process

A program is a thing that has the potential to run. A process is a running program.

### How is it limited?

1. Mode bit (user, system)
2. Interrupts (trap to handler)
3. Timers (hardware based)
4. Memory management unit
5. Other hardware (usual ...)

The mode bit comes into play - if you want to do a protected instruction, and you are in user mode, you will trap to the handler.

### System calls

To get around this, a user mode can make a system call which will change the mode bit and control to the system who then can make a judgement call on whether or not to run the priviledged instruction and then run it.

- setting mode bit
- disabling interrupts
- enabling interrupts
- writing to registers
- performing DMA
- halting CPU

_ALL_ privileged instructions.

### Interrupts

An interrupt is a hardware signal that causes the CPU to jump to an instruction called the interrupt handler. Sometimes `0x80`.

This helps efficient virtualization two ways

1. Any illegal instruction will be caught.
2. Periodic hardware timer interrupts help to keep the OS from losing control to one proc.

### Bootstrapping

When the power button gets clicked, some things need to go on.

1. BIOS (basic input output system) is the first thing to run.
2. It gets in touch with disk/keyboard/display, inits data structures, creates `init` proc, switches mode bit to user, starts first process
3. Now computer waits. OS is fundamentally event driven

## Processes

`Process:` OS abstraction for execution. Job/task/sequential process. Basically program in execution.

(0xFFFF) OS HEAP
BOOT STACK
OS STATIC DATA
ALL OTHER OS CODE
EXECEPTION VECTOR CODE
STACK (sp at bottom)
HEAP
STATIC DATA
CODE (PC at bottom)
(0x0000) UNUSED

There are some things we need to keep track of for a process.

1. Address space: code, data, execution stack
2. Program counter: where the next instruction is at
3. Registers with current values
4. Operating system resources (fds, network connections, ...)
5. Context for kernel execution (thread & stack)

Processes are identified with their pids
PROCESS CONTROL BLOCK (`PCB`) is where OS data is stored. 

### Process Control Block

1. Process state (ready, running, blocked, ...)
2. Program counter
3. CPU registers (must be saved at interrupt)
4. CPU scheduling information (priority)
5. Memory management info (page tables)
6. Accounting info (resource use info, ...)
7. I/O status info (list of open files, ...)

### State queue

The states has a queue for each state. `running`, `ready`, `blocked`.

A PCB is on one of these queues. When it changes state, it is unlinked and hooked to another PCB.

A flow chart to describe actions for a new, existing, and exiting proc:

NEW -> (`admit`) READY
READY -> (`dispatch`) RUNNING
RUNNING -> (`time-out`) READY
RUNNING -> (`event wait`) BLOCKED
RUNNING -> (`release`) EXIT
BLOCKED -> (`event occur`) READY

### New proc

When `NEW`, we create a new PCB and user address space structure, and allocate memory.

Then, we load the executable (init start state, change state to ready)

Finally, we dispatch the proc to running.

### Context switches

This is when we save the state of the old process and load the saved state for the new process.

This can happen when a proc `yields` due to the system call, makes a system call and is blocked, or hits a timer interrupt.

### Exit

On exit, there is some cleanup.

But we cant throw everything out right away.

We want to stop running the process, free resources, return code.

Want to ensure we give enough time for parents to know so we don't have zombies.


## API (fork, ...)

Fork:

1. Create and init a new PCB
2. Create a new address space
3. Init the address apce with a copy of the parent
4. Init kernel sresources to point to resources of parent (file descriptors, etc)
5. Place PCB on ready
6. FORK RETURNS TWICE! returns child PID to parent, 0 to the child

Exec:

1. Stops current process
2. Loads program into the process' address space
3. Init hardware context, args
4. Place PCB on ready queue
5. This does **not** create a new process.

### Comm between procs

1. You can communicate by passing arguments
2. By return codes
3. Sending signals
4. Shared FS
5. message passing, shared memory, synchronization, ...
