# Process Scheduling

## Introduction

Process scheduling is a fundamental responsibility of the operating system kernel. The scheduler determines which process should run on the CPU and how long it should execute.

Modern operating systems support **multitasking**, where many processes appear to run at the same time even though the system has a limited number of processors.

Two main approaches to multitasking exist.

**Preemptive multitasking**

The operating system decides when a running process should stop and another process should begin execution. The scheduler assigns each process a limited amount of CPU time.

Linux uses this approach.

**Cooperative multitasking**

Processes voluntarily give up control of the processor when they finish their work. If a process refuses to yield the CPU, the entire system can become unresponsive.

## Scheduling policies

A **scheduling policy** defines the rules used by the scheduler to decide which process runs next.

Scheduling must handle two main types of workloads.

**I/O bound processes**

These processes spend most of their time waiting for input or output operations such as disk access, network communication, or user input. Interactive programs like text editors or graphical applications usually fall into this category.

They often run briefly and then block while waiting for external events.

**CPU bound processes**

These processes spend most of their time performing computations. They rarely block for I/O and therefore consume large amounts of processor time.

Examples include scientific simulations or heavy mathematical calculations.

Some applications alternate between both behaviors depending on what they are currently doing.

## Process priority

Many scheduling systems assign a **priority value** to each process. The scheduler prefers processes with higher priority.

If multiple processes have the same priority, they are usually executed in a round robin fashion.

Linux maintains two different priority systems.

**Nice value**

The nice value ranges from **-20 to 19**.  
The default value is **0**.

Lower nice values represent higher priority. A process with a lower nice value receives a larger portion of CPU time. Higher nice values reduce the share of processor time.

The idea is that a process can be "nice" by allowing other tasks to run more frequently.

**Real time priority**

Real time priorities range from **0 to 99**.

These priorities are separate from nice values. Real time processes always have higher priority than normal processes.

Higher values correspond to higher priority.

## CPU time allocation

The amount of time a process is allowed to run before being interrupted is often called a **timeslice**.

If the timeslice is too long the system may feel slow or unresponsive.  
If the timeslice is too short the CPU spends too much time switching between processes.

Traditional systems used fixed timeslices such as 10 milliseconds.

Linux instead uses a different approach through the **Completely Fair Scheduler (CFS)**.

## Completely Fair Scheduler

The Completely Fair Scheduler attempts to distribute CPU time fairly among runnable processes.

The basic idea is to simulate an ideal system where each process receives an equal share of the processor.

If there are **n runnable processes**, each one should receive approximately **1/n** of the CPU time.

Instead of fixed timeslices, each process receives CPU time proportional to its **weight**, which is derived from its nice value.

Processes with higher priority receive larger shares of processor time.

## Fair scheduling concept

The scheduler keeps track of how much processor time each task has used.

Processes that have received less CPU time are scheduled earlier than processes that have already consumed more time.

This strategy improves responsiveness for interactive applications.

For example:

A system running a video encoder and a text editor behaves differently for each process type.

The video encoder performs continuous computations and uses the CPU heavily.  
The text editor spends most of its time waiting for keyboard input.

When the user presses a key, the text editor wakes up. Because it has used little CPU time recently, the scheduler immediately allows it to run so the interface remains responsive.

## Scheduler classes

Linux supports multiple scheduling algorithms known as **scheduler classes**.

Each class handles a different type of workload and has its own priority level.

The scheduler checks these classes in priority order. The highest priority class that contains runnable processes selects the next process to run.

Normal user processes are handled by the Completely Fair Scheduler.

## Virtual runtime

The Completely Fair Scheduler measures fairness using a value called **virtual runtime**.

Virtual runtime represents the amount of CPU time a process has used, adjusted according to its priority.

Processes with smaller virtual runtime values are scheduled first because they have consumed less processor time.

## Runnable process management

Runnable processes are stored in a **red black tree** ordered by their virtual runtime.

This tree structure allows the scheduler to efficiently select the process that has used the least CPU time.

The process with the smallest virtual runtime is located at the leftmost node of the tree and is chosen to run next.

When a process blocks or terminates it is removed from the tree. When it becomes runnable again it is inserted back into the structure.

## Sleeping and waking processes

A process may enter a **sleeping state** when it is waiting for an event such as file input or network activity.

Sleeping processes are not eligible to run until the event they are waiting for occurs.

There are two primary sleep states.

**TASK_INTERRUPTIBLE**

The process can be awakened by signals.

**TASK_UNINTERRUPTABLE**

The process ignores signals and wakes only when the required event occurs.

Processes waiting for events are placed in **wait queues**, which are lists of tasks that will be awakened when the event happens.

## Context switching

A **context switch** occurs when the operating system switches execution from one process to another.

During a context switch the kernel must:

- save the current process state
- load the state of the next process
- update memory mappings
- restore register and stack values

Although necessary, context switching introduces overhead, so schedulers try to minimize excessive switching.

## Kernel preemption

Modern Linux kernels support **kernel preemption**. This means a task running in kernel mode can be interrupted and replaced by another task if the system is in a safe state.

Each process maintains a **preemption counter**. If the value is zero the kernel can safely switch to another task.

Preemption can occur when:

- an interrupt handler finishes
- kernel code becomes preemptible again
- a process explicitly calls the scheduler
- a process blocks while waiting for an event

## Real time scheduling policies

Linux provides two policies designed for real time tasks.

**SCHED_FIFO**

Processes run in a first in first out order. A task continues running until it blocks or yields the CPU.

Only a task with higher real time priority can interrupt it.

**SCHED_RR**

This policy behaves like SCHED_FIFO but introduces a timeslice so tasks of the same priority share CPU time in a round robin manner.

Real time scheduling in Linux provides **soft real time guarantees**, meaning the system attempts to meet timing constraints but does not guarantee strict deadlines.

## Scheduler system calls

Linux exposes system calls that allow programs to modify scheduling behavior.

Important examples include:

- `nice()` adjusts the nice value of a process
- `sched_setscheduler()` changes scheduling policy
- `sched_getscheduler()` retrieves the scheduling policy
- `sched_setparam()` sets real time priority
- `sched_getparam()` retrieves real time priority
- `sched_yield()` voluntarily gives up the processor

## Processor affinity

Processor affinity determines which CPU cores a process is allowed to run on.

This information is stored as a **bitmask** indicating the permitted processors.

By default a process can run on any available CPU, but the affinity mask can be changed using scheduling system calls.

Restricting a process to specific processors may improve cache usage and performance.

## Yielding the processor

A process can voluntarily give up the CPU by calling `sched_yield()`.

When this happens the scheduler allows other runnable tasks to execute before returning the processor to the yielding process.

## Summary

The process scheduler allows multiple programs to share limited CPU resources efficiently.

Linux uses the Completely Fair Scheduler to distribute processor time among processes in a balanced manner while maintaining responsiveness for interactive workloads.

By considering priorities, process behavior, and system load, the scheduler ensures that CPU resources are shared effectively among all tasks.