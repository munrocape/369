## Synchronization

### Scheduling reminder

We have:

- multiple threads/procs ready to run
- mechanism to switch (context switch)
- a policy to choose the next proc
  - may be pre-emptive (threads cant anticipate when it may be forced to yield)
  - by design difficult to do that anyway

### Synchronization

Procs interact in a multiprogrammed system

share resources, coordinate execution, ...

arbitrarily choosing threads can have unexpected consequences.

we want way to restrict possible interleavings

**synchronization** allows us to do this

There are two main use cases:

1. Enforce single use of shared resource (critical section problem, withdraw deposit withdraw)
2. Control order of thread execution (parent before child, prod before consumer, ...)

### What aint shared

local variables (each thread has its own stack, local vars not on this stack, never share pointers to local vars)

### What is shared

global variables (stored in static data segment, accessed by all thread

dynamic objects / heap objects - allocated from heap with malloc/free or new/delete

## Mutual Exclusion

Given n threads, with a set of shared resources, and a segment of code which accesses these shared resources (critical section)

We want 

1. Only one thread at a time can execute in critical section
2. All other threads have to wait to enter when there is a thread in CS
3. When a thread leaves, it is clear for another to enter

We ultimately want to satisfy

1. Mutual exclusion (only one at a time)
2. Progress (if no one in CS, and a thread wants to enter, they should be able to - unrestricted by threads in "remainder")
3. Bounded waiting (no starvation) (limit on the number of times other threads can enter CS before this thread is granted)
4. Performance (dont want huge overhead costs)

### Petersons algorithm

```
int turn;
int flags[2]; // [0, 0]

do_work(id_t id) {
  flag[id] = 1;
  turn = id;
  while (turn == id && flag[1-id]); // spin
  //
  // critical section work
  //
  flag[id] = 0;
}
```

### Bakery algo

Each thread gets a # when entering

Thread with lowest number is served next

Not all threads guaranteed to have mutex numbers - lowest pid goes in case of tie

```

Entering[] // init to 0. number of elements = num threads
Number[] // array init to 0. number of elements = num threads

lock(i) {
  entering[i] = 1;
  number[i] = 1 + max (number[])
  entering[i] = 0;
  for (int j = 1; j < NUM_THREADS; j++) {
    while(entering[j]) {} ; // spin
    while ((number[j] != 0) && ((number[j], j) < (number[i], i))) {} // spin
    // basically says wait until thread j receives its number
    // wait until all threads with smaller priorities finish their work
    // instead of PID they are using index - but maybe that is the same anyway.
  }
}

unlock(i) {
  Number[i] = 0;
}

thread(int i) {
  while (true) {
    lock(i);
    //
    // critical section
    //
    unlock(i);
  }

}

```
## Semaphores

Contain

- counter variable, modified through two atomic operations
- `wait` or P or decrement (block until free)
- `signal` or V or increment (unblock waiting if there are any)
- queue of waiting threads

`mutex`: 0/1. 

Max number of threads is dictated by initial value, count. mutex has count = 1, counting semaphore has count = N.

### Test and set

Atomic instruction.

record the old value, set the value, return the old value.

```
t_and_s(bool *lock) {
  old = *lock;
  *lock = True;
  return old;
}

get_lock(bool *lock) {
  while(t_and_s(lock));
}

release_lock(bool *lock){
  *lock = false;
}

do_it(bool *lock){
  get_lock(lock);
  // do stuff
  release_lock(lock);
}
```

What this acomplishes is lets you know what is going on. If it returns true, its locked but if false, it sets it locked and you can proceed.

This is a spinlock.

Dangerous.

1. busy waiting (does work while waiting)
2. starvation is possible. cant guarantee scheduling
3. deadlock is possible through priority inversion

## Sleep lock

Go to sleep. Wake up thread on event.

`wait_event(queue, condition);`

`wake_up(queue);`















