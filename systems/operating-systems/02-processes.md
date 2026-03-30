# Processes

## Introduction

A **process** is a program that is currently executing. The program itself is stored as compiled object code, while the process represents the running instance of that program. Many processes can run the same program at the same time.

A process contains several resources such as:

- open files  
- processor state  
- a memory address space  
- one or more execution threads  
- global data

A **thread of execution** includes the program counter, processor registers, and a stack. In Linux the scheduler operates at the thread level.

Linux treats threads and processes in a very similar way. A thread is essentially a task that shares certain resources such as memory with other tasks.

Processes rely on two important abstractions.

**Virtual processor**  
Each process behaves as if it has its own CPU even though many processes may be sharing the same physical processor.

**Virtual memory**  
A process appears to have full access to memory. In practice the kernel manages memory and keeps processes isolated from one another.

Threads share the same memory space but each thread maintains its own execution state.

## Process lifecycle

A new process is usually created using the `fork()` system call. This produces a **child process** from an existing **parent process**.

Both processes continue execution after the call. The parent resumes execution normally while the child begins from the same point in the program.

Internally the `fork()` operation relies on the `clone()` system call.

The `exec()` family of system calls replaces the current address space with a new program. After the call completes the process begins running the new program.

A process ends when it calls the `exit()` system call. At this stage the kernel releases most of the resources associated with the process.

Until the parent collects the exit status using `wait()` or `waitpid()`, the terminated process remains in a **zombie state**.

## Task structure

Inside the Linux kernel a process is represented as a **task**.

Each task is described using a structure called `task_struct`. This structure stores information about the process including its state, scheduling information, memory data, and identifiers.

All tasks are stored in a circular doubly linked list known as the **task list**.

The `task_struct` object is relatively large and is allocated using the slab allocator so that memory can be reused efficiently.

Older Linux kernels placed the process descriptor at the end of the kernel stack. Modern kernels instead use a `thread_info` structure that provides access to the corresponding `task_struct`.

## Process identifiers

Every process receives a unique **process identifier** called a **PID**.

The PID is a numeric value used by the operating system to track and manage processes. The maximum number of processes that can exist at once depends on system configuration.

Kernel code often needs to access the currently running process. This is done using the `current()` macro which returns a pointer to the active `task_struct`.

The way this macro works depends on the processor architecture. Some systems keep a pointer to the current task in a register, while others derive it from the kernel stack.