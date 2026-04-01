---  
title: Linux  
--- 
  
# Linux
# Linux

Linux is an open-source operating system kernel. Many large internet companies run their infrastructure on Linux-based systems, making it useful for developers to understand how Linux works.

When people refer to Linux, they often mean a complete operating system that includes the kernel, system utilities, and libraries. Technically, however, **Linux itself refers only to the kernel**, while the surrounding components are part of the operating system built around it.

## The Kernel

The **kernel** is the core component of an operating system and provides the platform on which applications run.

Some of the key functions handled by the kernel include:

- Handling hardware interrupts  
- Scheduling processes  
- Managing memory  
- Providing networking functionality  

The memory region where the kernel executes is called **kernel space**, while the memory allocated to user programs is known as **user space**.

When the CPU executes kernel code, the system is said to be running in **kernel mode**. When normal applications are running, the processor operates in **user mode**.

Programs running in user mode interact with the kernel through **system calls**. These are functions provided by the kernel that allow applications to request operations requiring kernel privileges.

System calls are usually triggered through functions provided by libraries, such as the C standard library. When an application performs a system call, the request originates in user space while the kernel executes the operation in process context.

The kernel also manages communication with hardware devices. Hardware components notify the kernel of events using **interrupts**. When an interrupt occurs, the processor pauses its current execution and jumps to kernel code that was registered during system startup to handle that interrupt.

Linux follows a **monolithic kernel architecture**, meaning most operating system services run within a single kernel address space. This differs from **microkernel architectures**, where many system services are implemented as separate server processes.

## Linux Distributions

A **Linux distribution** is a complete operating system built around the Linux kernel. It typically includes system libraries, utilities, and tools needed to operate the system.

Some commonly used Linux distributions include **Ubuntu**, **CentOS**, and **Arch Linux**.

Most distributions also include a **package manager**, which allows users to install, update, and manage software easily.