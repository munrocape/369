### Non deadlock bugs

`atomicity violation:` 

Thread 1: Check value of something, do something with that something

Thread 2: Null out something

If Thread 1 checks value, Thread 2 nulls it out, THread 1 tries to do something...

`order violation:`

Thread 1: Init thing

Thread 2: Do something with initted thing

If Thread 2 goes before Thread 1, oops. Order violated. Thread 2 assumes the thing is always initted.

`fixes` condition variables, ...

### Deadlock bugs

Things that are required for deadlock:

1. `mutual exclusion` thread claims control of resource that they require
2. `hold-and-wait` threads hold resources allocated to them
3. `no preemption` cant be interrupted
4. `not necessary but necessary and sufficient with 1-3: circular wait` circular chain of threads that each have something the next wants

`fixes`

1. `circ wait` dont write your code that way. Provide a `total ordering`. E.g. require lock 1, then lock 2, ... instead of random order of acquiring. Even a partial ordering can be helpful. Could even acquire locks based on memory address: high to low, low to high, ...
2. `holdwait` get all locks at once, atomically. get a `getLock` lock, then your locks, do stuff, release the `getLock` lock. Decreases concurrency. Encapsulation works against us - know exactly which locks must be held and gotten ahead of time. 
3. `premption` tryLock - get, see if the next is free, if not, release and try again. May cause `livelock` - where two threads hot potato like this. Solution: add a random delay. Also, if you are deep in a routine, difficult to backtrack and free properly. 
4. `mutex` design things `wait-free`. compareAndSwap(v1, expected, new)

`avoidance`

scheduling things that require locks such that irrespective of what they do deadlock cannot occur

`detect & recover`

build resource graph and detect and cycles. usually powercycle things. 

selectively kill deadlocked processes until cyce is broken.

select preempt resources until cycle is broken (procs must be rolled back)

`ostrict algorithm` ignore the problem!!

`communicaiton deadlock` expectiong response, get nothing. Solution: try again, resend, timeouts, ...

#### Bankers algorithm

States: `safe`, `unsafe`, `deadlocked`

Each thread declares their resource requirements. Acquires/releases incrementally.

For each request:

1. Can the request be granted? If not, its impossible, block thread until possible
2. Assume granted - update state
3. Check if new state is safe - if so, continue. else, rollback and block thread.


### Logging

`transaction` collection of operations. read, write, write, read, ... [commit/abort]

`committed` transaction completed successfully

`aborted` transaction failed, rollback and start again

**write ahead**

before doing something, write it to a log (stable storage)

log can be used to undo/redo transactions. record data, old, new value, then start/commit/abort of transaction

**checkpoints**

time consuming to check entire log. write checkpoints as safe places - start recovery at last checkpoint.

**concurrent transactions**

1 option: transactions execute in a critical section. kills concurrency as most access different data

2nd option: allow overlap as long as they dont conflict (indistinguishable form sol 1)

`conflict` if access same data item and at least one is a write. if a serial schedule can be obtained by swapping them, then the og schedule is conflict serializable.

**ensuring serializability through two way locking**

growing: can acquire locks

shrinking: can only release locks

fix: prevent hold-and-wait by aborting if a lock is unavailable in grow phase

**timestamp protocol**

transaction get `timestamp` before it starts executing. earlier timestamp must complete before any later transactions. 

data items have two timestamps. `W-TS` last successfuly write. `R-TS` last successful read.

if you write to something that was more recently read, you have been overwritten. abort.

if you read and have an earlier timestamp than `W-TS`, then your read is overwritten. restart.

This can lead to starvation (abort, restart repeatedly)




