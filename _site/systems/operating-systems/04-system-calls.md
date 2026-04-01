# System Calls

## Introduction

System calls form the interface between user programs and the operating system kernel. They allow programs running in user space to request services that require higher privileges, such as interacting with the filesystem, creating processes, or communicating with hardware.

A user program cannot directly execute kernel code because the kernel operates in a protected memory region. System calls provide a controlled entry point that allows user programs to safely request these operations.

This layer offers several advantages:

- It hides hardware details behind a consistent interface.
- It protects system stability by letting the kernel manage permissions and access.
- It enables process isolation and virtual memory.

Without system calls, many operating system features such as memory virtualization and secure resource access would not be possible.

## APIs, POSIX, and the C Library

Applications usually do not invoke system calls directly. Instead, they interact with higher level APIs provided by libraries.

This separation makes software portable. Programs can rely on a standard API while the operating system internally maps those API calls to its own system call implementation.

A widely used standard is POSIX (Portable Operating System Interface). It defines a set of APIs that Unix-like systems aim to support. Even non Unix systems often implement parts of the POSIX interface for compatibility.

On Linux, the C standard library provides most of these APIs. Many POSIX functions are implemented within the C library and internally trigger the appropriate system calls.

## Syscalls

System calls are typically accessed through functions defined in the C library.

If a system call fails, the library records an error code in a global variable named `errno`. Programs can convert this error code into a readable message using helper functions such as `perror()`.

System calls themselves are implemented in the kernel. For example, the call that returns the current process ID is implemented in the kernel as a function similar to:

```
SYSCALL_DEFINE0(getpid)
{
    return task_tgid_vnr(current); /* returns current->tgid */
}
```

The macro `SYSCALL_DEFINE0` defines a system call that takes zero arguments.

Inside the kernel, the actual implementation usually follows a naming convention where the system call name begins with `sys_`. For instance, the kernel implementation of `getpid()` is `sys_getpid()`.

Because the kernel must support both 32 bit and 64 bit systems, return values that appear as `int` in user space are typically represented as `long` in the kernel.

## System Call Numbers

Each system call has a unique identifier called a system call number. Once assigned, this number must remain unchanged because compiled programs depend on it.

These numbers are stored in a table maintained by the kernel. This table maps each system call number to the function that implements it.

The table is architecture specific, meaning different processor architectures maintain their own versions.

## System Call Handler

User programs enter kernel mode through controlled mechanisms such as traps or exceptions.

When a program requests a system call, the processor switches to kernel mode and transfers control to a system call handler.

The handler determines which system call was requested and invokes the corresponding kernel function.

For example, on x86 systems the system call number is placed in a register. The handler checks that the number is valid and then looks up the corresponding function in the system call table.

## Parameter Passing

Arguments for a system call are passed through processor registers.

On x86 systems, registers such as `ebx`, `ecx`, `edx`, `esi`, and `edi` are used to store parameters.

If a system call requires more parameters than available registers, a register may contain a pointer to a memory location in user space that holds the additional arguments.

## Adding a System Call

Introducing a new system call into the Linux kernel generally involves three steps.

1. Add the new system call to the system call table for each supported architecture.
2. Define the system call number in the appropriate header file.
3. Implement the system call in the kernel source code so it becomes part of the kernel image.

For example, if a new system call named `foo()` were introduced, it would be added to the system call table and assigned a unique number. A corresponding macro would also be defined in the system call number definitions.

The implementation might look like this:

```
#include <asm/page.h>

/*
* sys_foo – everyone’s favorite system call.
*
* Returns the size of the per-process kernel stack.
*/
asmlinkage long sys_foo(void)
{
    return THREAD_SIZE;
}
```

## Accessing System Calls from User Space

Linux provides macros that allow programs to invoke system calls directly when needed.

These macros follow the pattern `_syscalln`, where `n` indicates the number of parameters the call expects. The macro prepares the required registers and triggers the trap that transfers control to the kernel.

For instance, if a system call takes three parameters, the `_syscall3` macro may be used to create a wrapper function that handles the low level interaction with the kernel.

In practice, most applications rely on standard library functions rather than invoking these macros directly.

## Summary

System calls are the primary mechanism through which user programs request services from the operating system kernel. They enforce protection boundaries, allow controlled access to hardware resources, and provide the foundation for many operating system features.

Most applications interact with system calls indirectly through standard libraries, which simplify usage and improve portability across systems.

