# Memory Management  
  
## Pages  
  
The Linux kernel manages memory using **pages**, which are fixed size blocks of memory.  
  
- A page is the basic unit of memory management.  
- Page size depends on architecture.  
- Common sizes are **4 KB or 8 KB**.  
  
Each physical page is represented by a structure called `struct page`.  
  
Important information stored in this structure includes:  
  
- status of the page  
- reference count  
- mapping information  
- virtual address if mapped  
  
The reference count indicates how many components are currently using the page.  

## Memory Zones  
  
Physical memory is divided into **zones** to handle hardware limitations.  
  
Some devices can only access certain memory ranges, especially when using DMA.  
  
Common zones:  
  
| Zone           | Description                 | Physical Memory |
| -------------- | --------------------------- | --------------- |
| `ZONE_DMA`     | DMA-able pages.             | < 16 MB         |
| `ZONE_NORMAL`  | Normally addressable pages. | 16–896 MB       |
| `ZONE_HIGHMEM` | Dynamically mapped pages.   | > 896 MB        |
  
During allocation, the kernel selects pages from the appropriate zone.  

## Page Allocation  
  
The main function used for page allocation is:  
  
```c  
struct page *alloc_pages(gfp_t gfp_mask, unsigned int order);
```

- `order` determines the number of contiguous pages requested.
- the number of pages allocated is 2^order

Example:

- order 0 → 1 page
- order 1 → 2 pages
- order 2 → 4 pages

To free pages:

```c
void __free_pages(struct page *page, unsigned int order);
```

## kmalloc

`kmalloc()` allocates memory inside the kernel similar to `malloc()`.

Characteristics:

- physically contiguous
- virtually contiguous
- efficient for small allocations

Example:

```c
void *ptr = kmalloc(size, GFP_KERNEL);
```

Memory is released using:

```c
kfree(ptr);
```

## vmalloc

`vmalloc()` allocates **virtually contiguous memory**.

Unlike `kmalloc`, the memory **does not need to be physically contiguous**.

This is useful for large memory allocations when contiguous physical memory is not available.

## GFP Flags

Memory allocation functions take **GFP flags** that control allocation behavior.

Common flags:

| Flag         | Meaning                                            |
| ------------ | -------------------------------------------------- |
| `GFP_KERNEL` | normal allocation that may sleep                   |
| `GFP_ATOMIC` | allocation cannot sleep, used in interrupt context |
| `GFP_USER`   | memory allocated for user processes                |
| `GFP_DMA`    | allocation from DMA capable memory                 |

`GFP_KERNEL` is the most commonly used flag.

## Slab Allocator

The **slab allocator** is used to efficiently allocate frequently used kernel objects.

Instead of allocating memory every time, the kernel maintains **object caches**.

Structure:

Cache → Slabs → Objects

- **Cache** stores objects of a specific type
- **Slabs** contain multiple objects
- Objects are reused when freed

This approach improves performance and reduces memory fragmentation.

## High Memory

Some systems contain **high memory**, which is not permanently mapped in kernel space.

To access such memory, the kernel temporarily maps it using:

`kmap()`

The mapping is removed using:

`kunmap()`

This allows the kernel to access high memory when needed.