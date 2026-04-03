# Kernel Memory Management  
  
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
  
| Zone | Purpose |  
|-----|-----|  
| ZONE_DMA | memory usable for DMA devices |  
| ZONE_NORMAL | normal kernel memory |  
| ZONE_HIGHMEM | memory not permanently mapped into kernel space |  
  
During allocation, the kernel selects pages from the appropriate zone.  

## Page Allocation  
  
The main function used for page allocation is:  
  
```c  
struct page *alloc_pages(gfp_t gfp_mask, unsigned int order);

- `order` determines the number of contiguous pages requested.
- the number of pages allocated is 2order2^{order}2order

Example:

- order 0 → 1 page
- order 1 → 2 pages
- order 2 → 4 pages

To free pages:

void __free_pages(struct page *page, unsigned int order);
```
```

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

kfree(ptr);

---

## vmalloc

`vmalloc()` allocates **virtually contiguous memory**.

Unlike `kmalloc`, the memory **does not need to be physically contiguous**.

This is useful for large memory allocations when contiguous physical memory is not available.

---

## GFP Flags

Memory allocation functions take **GFP flags** that control allocation behavior.

Common flags:

|Flag|Meaning|
|---|---|
|GFP_KERNEL|normal allocation that may sleep|
|GFP_ATOMIC|allocation cannot sleep, used in interrupt context|
|GFP_USER|memory allocated for user processes|
|GFP_DMA|allocation from DMA capable memory|

`GFP_KERNEL` is the most commonly used flag.

---

## Slab Allocator

The **slab allocator** is used to efficiently allocate frequently used kernel objects.

Instead of allocating memory every time, the kernel maintains **object caches**.

Structure:

Cache → Slabs → Objects

- **Cache** stores objects of a specific type
- **Slabs** contain multiple objects
- Objects are reused when freed

This approach improves performance and reduces memory fragmentation.

---

## High Memory

Some systems contain **high memory**, which is not permanently mapped in kernel space.

To access such memory, the kernel temporarily maps it using:

kmap()

The mapping is removed using:

kunmap()

This allows the kernel to access high memory when needed.

  
This version is **closer to what interviewers expect** while still being good for GitHub notes.  
  
If you want, I can also show you **what interviewers usually ask from this topic** (about 6–7 very common questions) so you know exactly what parts of these notes matter.

i just told u to never include --- these hrs

Got it. Here is the **clean interview focused Markdown without horizontal rules**.

# Kernel Memory Management  
  
## Pages  
  
The Linux kernel manages memory using **pages**, which are fixed size blocks of memory.  
  
- A page is the fundamental unit of memory management.  
- Page size depends on the architecture.  
- Common page sizes are **4 KB or 8 KB**.  
  
Each physical page is represented using a structure called `struct page`.  
  
Important fields typically include:  
  
- page status flags  
- reference count  
- mapping information  
- virtual address if mapped  
  
The reference count indicates how many components are currently using the page. When the count reaches zero, the page can be reused.  
  
## Memory Zones  
  
Physical memory is divided into **zones** because hardware devices have different memory access limitations.  
  
Common zones include:  
  
| Zone | Purpose |  
|-----|-----|  
| ZONE_DMA | memory used by DMA capable devices |  
| ZONE_NORMAL | normal memory used by the kernel |  
| ZONE_HIGHMEM | memory not permanently mapped in kernel space |  
  
The kernel chooses the appropriate zone during memory allocation.  
  
## Page Allocation  
  
The primary function used to allocate pages is:  
  
```c  
struct page *alloc_pages(gfp_t gfp_mask, unsigned int order);

`order` determines how many contiguous pages are allocated.

Number of pages allocated:

2^order

Example:

- order 0 → 1 page
- order 1 → 2 pages
- order 2 → 4 pages

To release allocated pages:

void __free_pages(struct page *page, unsigned int order);

## kmalloc

`kmalloc()` allocates memory in the kernel similar to `malloc()`.

Characteristics:

- physically contiguous
- virtually contiguous
- efficient for small allocations

Example:

void *ptr = kmalloc(size, GFP_KERNEL);

Memory is freed using:

kfree(ptr);

## vmalloc

`vmalloc()` allocates **virtually contiguous memory**.

Unlike `kmalloc`, the memory does not need to be physically contiguous.

This is useful when allocating larger memory regions where contiguous physical memory may not be available.

## GFP Flags

Memory allocation functions use **GFP flags** to control how allocation behaves.

Common flags:

|Flag|Meaning|
|---|---|
|GFP_KERNEL|normal allocation that may sleep|
|GFP_ATOMIC|allocation cannot sleep|
|GFP_USER|memory allocated for user processes|
|GFP_DMA|allocation from DMA capable memory|

`GFP_KERNEL` is the most commonly used flag for normal kernel allocations.

## Slab Allocator

The **slab allocator** is used for efficient allocation of frequently used kernel objects.

Instead of allocating and freeing memory repeatedly, the kernel maintains **caches of objects**.

Structure:

Cache → Slabs → Objects

- A cache stores objects of a specific type.
- A slab contains multiple objects.
- Freed objects can be reused quickly.

This improves allocation speed and reduces memory fragmentation.

## High Memory

Some systems have **high memory**, which is not permanently mapped in the kernel address space.

To access such memory, the kernel temporarily maps it using:

kmap()

After usage, the mapping is removed with:

kunmap()

This allows the kernel to access high memory safely when required.