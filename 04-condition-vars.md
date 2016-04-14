# Condition variables and monitors

Problem with semaphorescan be P operation.

`if x == 0 and (y > 0 or z > 0) then sleep`

This is checked outside of P(). But, checking requires mutex. easy to slip up.

```python

no_go = 0
mutex = 1
x, y, z = 0

def compute():
  P(mutex)
  x = f1(x)
  y = f2(y)
  z = f3(z)
  if (x != 0 or (y <= 0 and z <= 0)):
    V(no_go)
  V(mutex)
  
def use():
  P(mutex)
  if (x == 0 and (y > 0 or z > 0)):
    P(no_go)
  process(x, y, z)
  V(mutex)
```

problems:

1. deadlock: `use()` has to wait on `no_go` then `compute` cannot get mutex.
2. changing state: t1: compute, new x,y,z, good => v(no_go); t2: compute, x=0,y=1,z=0, bad => skip; t3: use, sees bad x,y,z values, P(no_go) P returns immediatey as count was 1, even though x,y,z are unusale

### Condition variables

ADT that encapsulates pattern of "release mutex, sleep, reacquire"

Internal data is just a queue of waiting threads.

Have a couple ops.

- `cv_wait(cv, mutex)` - release lock, reacquire mutex before return
- `cv_signal(cv)` - wake 1
- `cv_broadcast(cv)` - wake all

unlike semaphore, if no waiters, signal and broadcast are not saved.

**always** use a CV with a lock.

```Python

lock_acquire(lock)
while(condition == false):
  cv_wait(condition, lock)
cv_signal(condition)
lock_release(lock)
```

When you create these, you init it to `pthread_cond_t cv = PTHREAD_COND_INITIALIZER`


### Monitors

ADT with restriction that only one proc at a time can be active within the monitor.

enforces mutex.

local data accessed only by monitor procedure.

process enters the monitor by invoking one of the procedures.

other processes that attempt to enter monitor are blocked.

### Hoare v Mesa

Hoare was OG. 

- `signal` immediately switches from the caller to a waiting thread.
- condition that the waiter was blocked on is guaranteed to hold
- need another quue for the signaler, if signaler needs monitor back

Mesa

- signal places a waiter on the ready queue. signaler continues inside monitor
- condition is not necc. true when waiter resumes
- must recheck condition, basically.

```Python

def hoare():
  if empty:
    wait(condition)
  // do stuff
def mesa():
  while empty:
    wait(condition)
  // dostuff
```

Hoare makes i easier to reason about program. Mesa monitors are easier to implement and can support braodcast.

```Python
N = 100

buf = 
  {
    data[N]
    inpos
    outpos
    numelements
    buflock
    notFull
    notEMpty
  }

def add_to_buff(buf, val):
  lock_acquire(b.buflock)
  while (b.elements == N):
    cv_wait(b.notFull, b.bufLock)
  b.data = value
  b.inpos = b.inpos + 1 % n
  b.nelements++
  cv_signal(b.notEmpty, b.bufLock)
  lock_release(b.bufLock)
  
def remove_from_buff(buf):
  lock_acquire(b.bufLock)
  while(b.nelements == 0):
    cv_wait(b.notEmpty, b.bufLock)
  val = b.data[b.outpos]
  b.outpos = b.outpos + 1
  b.nelements --
  cv_signal(b.notFull, b.bufLock)
  lock_release(b.bufLock)
  return val
```

** call cv_wait only in while loops **

```Python

def get_f_from_cache(fname):
  data = []
  lock_acquire(cache_lock)
  data = cache_lookup(fname)
  if (data == []):
    fdata = cache_get_space(fname)
    read_from_disk(fdata, fname)
  lock_release(cache_lock)
  return dfata
```

This works, but is _slow_. Only one lock - for every file!

We should rework.

Create a struct that holds info on files - in cache, data, filename

### Deadlock, starvation

Deadlock: two procs holding resource other needs - wont release until they get it

Starvation: thread waiting indefinitely because other threads are in the way (scheduling problem)











