# Concurrency

## Introduction

Concurrency is a programming approach where multiple tasks execute during overlapping time periods. These tasks may run truly in parallel on multiple processors, or they may be interleaved by the operating system scheduler on a single processor.

Using concurrency effectively can significantly improve the performance and responsiveness of software systems, especially when programs must handle many operations at once.

Two common interaction models exist in concurrent systems.

**Shared memory**

Concurrent units communicate by reading and writing the same memory locations. Multiple processors or threads can access shared variables, which allows them to exchange information directly through memory.

Examples include threads within the same process or processors sharing the same physical memory.

**Message passing**

Concurrent units communicate by sending messages through communication channels. Instead of sharing memory, each unit exchanges information using explicit messages.

Examples include processes communicating through pipes, sockets, or other communication mechanisms.

## Processes and Threads

A **process** is a running instance of a program. Each process normally has its own memory space and is isolated from other processes running on the same system.

A **thread** is a unit of execution inside a process. Threads within the same process share the same address space and resources, which allows them to communicate quickly through shared variables.

Because threads share memory, they must coordinate access to shared resources carefully.

A **fiber** is similar to a thread but uses cooperative scheduling. In this model, a running fiber voluntarily yields control so another fiber can execute. In contrast, threads usually rely on preemptive scheduling where the operating system decides when to switch execution.

## Critical Regions

A **critical region** (also called a critical section) is a portion of code that accesses shared resources such as variables, files, or data structures.

If multiple threads execute the same critical region simultaneously, unexpected behavior can occur because operations may interfere with each other.

Consider a shared variable `i` that is incremented.

```
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

```
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

## Futex

A **futex** (fast userspace mutex) is a synchronization primitive commonly used in operating systems such as Linux.

It combines user space locking with kernel assistance.

The idea is that threads attempt to acquire the lock in user space first. If the lock is not available, the thread enters the kernel and sleeps until another thread wakes it.

Two main operations exist:

**FUTEX_WAIT**

If the value at a given memory address matches an expected value, the thread sleeps and waits for a wake signal.

**FUTEX_WAKE**

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
