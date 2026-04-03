# Filesystem

## Filesystem

A filesystem is the mechanism used by an operating system to organize, store, and retrieve data on storage devices such as disks or SSDs. It defines how files are structured, named, and accessed.

In Linux, filesystems are designed using abstraction layers so that the kernel can interact with different storage formats using the same interface.


## Virtual File System (VFS)

The Virtual File System (VFS) provides a unified interface for all filesystem implementations in Linux.

Instead of each filesystem being accessed directly, the kernel interacts with the VFS layer. VFS then calls the appropriate filesystem specific functions.

Example flow when writing to a file:

Application → system call (`write`) → VFS → specific filesystem → storage device

This abstraction allows Linux to support many filesystems such as ext4, FAT, and network filesystems without modifying kernel logic.


## Unix Filesystem Concepts

Unix based systems organize storage using several core abstractions.

### File

A file is a sequence of bytes stored on disk. Files store both data and metadata such as permissions, owner, and timestamps.

### Directory

A directory is a special type of file that stores mappings between filenames and inode numbers. It allows hierarchical organization of files.

### Inode

An inode (index node) stores metadata about a file, including:

file size  
permissions  
timestamps  
ownership  
disk block pointers

The inode does not store the filename. Instead, directories associate filenames with inode numbers.

### Mount Point

A mount point is a directory where another filesystem is attached. Once mounted, the new filesystem appears as part of the same directory hierarchy.


## VFS Core Objects

The VFS uses several key data structures to represent filesystem components.

superblock  
inode  
dentry  
file

Each object represents a different part of the filesystem abstraction and contains operation tables that define how the kernel interacts with them.


## Superblock

The superblock stores metadata describing an entire filesystem. It contains information such as:

filesystem type  
block size  
maximum file size  
location of the root directory  
mount options

When a filesystem is mounted, the kernel reads the superblock from disk and stores it in memory so the filesystem can be managed efficiently.


## Inodes

An inode represents a filesystem object such as a file or directory.

Each inode contains metadata describing that object and pointers to the blocks where the file’s actual data is stored.

Important inode properties include:

file size  
number of hard links  
owner and permissions  
timestamps for access and modification  
pointers to disk blocks

Inodes allow the system to separate file metadata from filenames.


## Dentries

A dentry (directory entry) represents a component of a file path.

For example in the path:

/bin/bash

each component (`/`, `bin`, `bash`) is represented by a dentry.

Dentries help the kernel resolve paths efficiently by caching directory lookups in the dentry cache.


## File Object

A file object represents an open file in the kernel.

It is created when a process calls `open()` and destroyed when the file is closed.

Multiple processes can open the same file, which means multiple file objects may point to the same inode.

A file object tracks:

current file position  
access mode  
flags used during opening  
pointer to file operations


## Block Devices

Block devices are storage devices that provide random access to fixed size blocks of data.

Examples include:

hard drives  
SSDs  
flash storage

The smallest addressable unit on these devices is typically a sector (often 512 bytes).


## Buffers and Buffer Heads

When disk blocks are loaded into memory they are stored in buffers.

A buffer represents a disk block in RAM. The kernel uses a structure called a buffer head to track metadata about the buffer.

The buffer head records:

which disk block the buffer corresponds to  
the memory page storing the data  
buffer state flags such as dirty or locked

If a buffer is marked dirty, it means its contents have been modified and must eventually be written back to disk.


## The bio Structure

The bio structure represents an active block I/O request.

Instead of representing a single disk block, a bio can represent multiple memory segments involved in a single I/O operation.

This structure allows the kernel to efficiently handle read and write operations that involve several memory pages.


## Request Queues

Block devices maintain request queues that hold pending I/O operations.

Each request may consist of one or more bio structures.

The device driver processes requests from the queue and sends them to the storage hardware.


## I/O Schedulers

I/O schedulers determine the order in which disk requests are processed.

Their goal is to improve performance by reducing disk seek time and ensuring fair access among processes.

Schedulers typically perform:

sorting requests by disk location  
merging nearby requests  
preventing starvation of requests


## Deadline Scheduler

The Deadline scheduler prioritizes read operations while ensuring that write operations are not delayed indefinitely.

Each request is assigned a deadline. If the deadline expires, the request is executed even if it is not optimal in terms of disk location.

This approach reduces starvation and improves responsiveness for read operations.


## Anticipatory Scheduler

The Anticipatory scheduler waits briefly after completing a read request before processing other operations.

The idea is that applications often perform sequential reads. By waiting a short time, the scheduler may receive another nearby request and avoid expensive disk seeks.


## Completely Fair Queuing (CFQ)

The CFQ scheduler maintains separate request queues for different processes.

Requests from these queues are served in a round robin manner to provide fairness between processes.

This scheduler attempts to balance performance with equal access to disk resources.


## Noop Scheduler

The Noop scheduler is a very simple scheduler that performs minimal processing.

It mainly merges adjacent requests but does not attempt complex sorting.

This scheduler works well for storage devices such as SSDs where seek time is negligible.