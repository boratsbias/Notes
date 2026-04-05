# Kernel Synchronization

Shared data inside the kernel must be protected when multiple execution paths can access it at the same time. If several threads operate on the same resource concurrently, one thread may overwrite the changes made by another or read data that is only partially updated. These situations can lead to incorrect program behavior and subtle bugs.

Handling shared resources in the Linux kernel is challenging because the system supports symmetric multiprocessing. This means kernel code may execute on several CPU cores simultaneously. In addition, the Linux kernel is preemptive, allowing the scheduler to interrupt running kernel code and switch execution to another task.

Because of these factors, mechanisms are required to coordinate access to shared data. Concurrency control and synchronization tools are used to ensure that kernel operations remain correct even when multiple threads execute at the same time.

## Critical Regions

A **critical region** (also called a critical section) is a portion of code that accesses shared resources such as variables, files, or data structures.

If multiple threads execute the same critical region simultaneously, unexpected behavior can occur because operations may interfere with each other.

Consider a shared variable `i` that is incremented.

```c
i++;
```

Although this appears to be a single operation, it is typically executed as several machine instructions:

1. Load the value of `i` from memory
2. Add one to the value
3. Store the result back to memory

If two threads execute this sequence at the same time, their instructions may interleave in an unpredictable order. This can lead to incorrect results.

Such situations are called **race conditions**, where the outcome depends on the order in which threads access shared data.

To prevent this problem, access to the critical region must be controlled so that only one thread executes it at a time.

## Atomic Operations

An **atomic operation** is an operation that executes as a single indivisible unit. Once it begins, no other thread can observe an intermediate state.

Atomic instructions are the foundation of many synchronization techniques. Many processors provide hardware instructions that can read, modify, and write a value in one step.

Examples include atomic increment, compare and swap, and test and set.

These operations help implement higher level synchronization tools such as locks.

## Locks

A **lock** is a synchronization mechanism used to ensure that only one thread can enter a critical region at a time.

When a thread enters the protected section of code, it acquires the lock. Other threads attempting to enter must wait until the lock is released.

The typical sequence is:

1. Acquire the lock
2. Execute the critical section
3. Release the lock

Locks can be cooperative or mandatory.

**Cooperative locks**

These rely on all threads following the same locking rules. If a thread ignores the locking mechanism, it can still access the resource.

Most programming language locks follow this model.

**Mandatory locks**

These are enforced by the system, and attempts to access a locked resource without permission will trigger an error or exception.

Many locks are implemented using atomic instructions that test and modify a memory location in one step.

## Spin Locks

A **spin lock** repeatedly checks whether a lock has become available. Instead of sleeping, the thread continuously loops until the lock is released.

Example concept:

```c
volatile int lock = 0;

void acquire() {  
while (atomic_test_and_set(&lock, 1)) {  
// keep checking  
}  
}

void release() {  
atomic_write(&lock, 0);  
}
```

Spin locks are useful when the waiting time is expected to be very short. In such cases, spinning may be faster than suspending and waking up a thread through the operating system.

However, if the wait time is long, spin locks waste CPU resources.

## Mutex

A mutex is a synchronization primitive that ensures mutual exclusion. Only one thread can hold the mutex at a time.

A thread must acquire the mutex before entering a critical section and release it when the work is finished.

Unlike semaphores, a mutex has ownership. The thread that locks the mutex must also unlock it.

Mutexes are commonly used to protect shared variables or data structures from concurrent modification.

## Semaphores

A semaphore is a synchronization primitive used to control access to shared resources.

A semaphore maintains an integer counter that represents how many threads can access the resource simultaneously.

Two operations are used.

``wait()``

Decreases the semaphore value.  
If the value becomes negative, the calling thread blocks until another thread releases the resource.

`signal()`

Increases the semaphore value and wakes one waiting thread if one exists.

Two common types exist.

**Binary semaphore**

The value is limited to 0 or 1.  
This behaves similarly to a mutex and allows only one thread to access the resource at a time.

**Counting semaphore**

The value can be greater than one.  
This allows multiple threads to access a resource concurrently up to a defined limit.

Semaphores are commonly used in synchronization problems such as producer consumer systems.

## Reader Writer Locks

Reader writer locks allow multiple threads to read shared data simultaneously while ensuring exclusive access for writing.

When a thread acquires the lock in read mode, other readers can also access the resource at the same time.

When a thread acquires the lock in write mode, it receives exclusive access and no other readers or writers can access the resource.

This type of lock improves performance in systems where read operations occur much more frequently than write operations.

## Futex

A **futex** (fast userspace mutex) is a synchronization primitive commonly used in operating systems such as Linux.

It combines user space locking with kernel assistance.

The idea is that threads attempt to acquire the lock in user space first. If the lock is not available, the thread enters the kernel and sleeps until another thread wakes it.

Two main operations exist:

`FUTEX_WAIT`

If the value at a given memory address matches an expected value, the thread sleeps and waits for a wake signal.

``FUTEX_WAKE``

This operation wakes one or more threads waiting on the same futex variable.

This approach reduces kernel involvement and improves performance.

## Deadlocks

A **deadlock** occurs when a group of threads cannot proceed because each one is waiting for another to release a resource.

A simple example involves two locks and two threads:

- Thread A holds lock 1 and waits for lock 2
- Thread B holds lock 2 and waits for lock 1

Neither thread can continue, so the program stops making progress.

A special case is **self deadlock**, where a thread attempts to acquire a lock that it already holds.

Common strategies to prevent deadlocks include:

- Always acquire multiple locks in a consistent order
- Avoid holding locks longer than necessary
- Do not attempt to acquire the same lock multiple times unless the lock supports reentrancy

## Lock Free Programming

Lock free programming is an approach where threads coordinate without using traditional locks. Instead, algorithms rely on atomic operations to update shared data safely.

The goal is to guarantee that at least one thread can make progress even when others are delayed.

This technique can improve scalability but often requires more complex algorithm design.

## Memory Fences

A **memory fence** (also called a memory barrier) is a CPU instruction that enforces ordering constraints on memory operations.

Modern processors and compilers often reorder instructions to improve performance. Without control mechanisms, this reordering can break assumptions made by concurrent programs.

Memory fences ensure that certain memory operations occur in the intended order.

Common types include:

**Full fence**

Prevents memory operations from moving across the barrier in either direction.

**Release fence**

Ensures that operations before the fence complete before any operations that follow.

**Acquire fence**

Ensures that operations after the fence cannot be moved before the barrier.

Memory fences are important for implementing correct synchronization primitives and ensuring consistent behavior across multiple threads.
