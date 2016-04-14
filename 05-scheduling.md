## Scheduling

OS manages procs. Procs have threads. How is all of this scheduled?

### Policy 1

Process runs to completion. First come, first served.

If you show up late, and there is a lot in front of you, service time is a long time away.

### Shortest job first

starvation!

### Goals of scheduler

1. fairness (each thread receives equal time)
2. Avoid starvation
3. Policy enforcement - usage policies should be met
4. Balance - all parts of system should be busy (IO, CPU, ...)

Batch systems like to prioritize throughput and turnaround time and CPU util.

Interactive systems like to prioritize response time - dont want system hanging for user. and proportionality - simple tasks finish quickly.

Real time systems like to meet deadlines and be predictable.

All goals overall are at odds with one another.

1. long term: admission scheduling, used in batch systems, uncommon today
2. medium term: memory sched, decides which procs are swapped out to disk
3. short term: dispatching: needs fast context switch, fast queue manipulation

### Dispatch

select next thread, save current execution state, swap next to running

We scheduling when thread enters ready state, thread blocks (or exits), or on a timer interrupt.

### Types of scheduling

1. Preemptive: cpu can be taken from a running thread whenever.
2. Non-preemptive: cpu belongs to one thread until it terminates or blocks.

### FCFS

non preemptive. choose thread at head of ready queue. average waiting time quite long - convoy effect.

### Shortest job first

provably optimal for average wait time.

### Round robin

ready queue is circular. time sharing optimal. there is a timeslice (quantum) q -> inf, RR -> FCFS, q -> 0, RR -> proc sharing

### Priority scheduling

threads have priorities. highest priority job is selected.

`priority inversion`: low priority holds resource high priority wants. 
`starvation`: low priority never run.

### Multi level queue scheduling

multiple queues. thread on one queue. permanently assigned a queue. queue has own schedlung algo (usually RR). another level of scheduling picks next queue (usually priority).

### Feedback scheduling

adjust criteria for selection based on past history. aging, dont use full quantum, user input required, next CPU burst expectation, ...

1. You use full timeslice, you priority is reduced.
2. You give up CPU, you stay at same prioity
3. Algo selects job at front of highest priority to run next.











