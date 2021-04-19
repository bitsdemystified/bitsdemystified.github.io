---
layout: post
title: Virtual memory
subtitle: Vitual memory role in computer systems
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [systemprogramming, virtualmemory,csapp]
---

Virtual memory is a powerful concept and plays a vital role in memory management. 

With virtual addressing, the CPU accesses main memory by generating a virtual address (VA), which is converted to the appropriate physical address before being sent to the memory. The task of converting a virtual address to a physical one is known as address translation. Like exception handling, address translation requires close cooperation between the CPU hardware and the operating sys- tem. Dedicated hardware on the CPU chip called the memory management unit (MMU) translates virtual addresses on the fly, using a look-up table stored in main memory whose contents are managed by the operating system.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/cpu-vm-intro.png" align="center">

## Virtual and Physical Page
In virtual memory systems, main memory (DRAM) acts a cache like SRAM (L1, L2, L3) act as cache for CPU.<br/>
A virtual memory is organized as an array of N contiguous byte-sized cells stored on disk. The contents of the array on disk are cached in main memory. As with any other cache in the memory hierarchy, the data on disk (the lower level) is partitioned into blocks that serve as the transfer units between the disk and the main memory (the upper level). VM systems handle this by partitioning the virtual memory into fixed-sized blocks called virtual pages (VPs). Each virtual page is P = 2^p bytes in size. Similarly, physical memory is partitioned into physical pages (PPs), also P bytes in size. (Physical pages are also referred to as page frames.)<br/>
At any point in time, the set of virtual pages is partitioned into three disjoint subsets:
* Unallocated: Pages that have not yet been allocated (or created) by the VM system. Unallocated blocks do not have any data associated with them, and thus do not occupy any space on disk
* Cached: Allocated pages that are currently cached in physical memory.
* Uncached: Allocated pages that are not cached in physical memory.

Misses in DRAM are very expensive because then we need to access the disk. Due to this large miss penalty and the expense of accessing the first byte, virtual pages tend to be large, typically 4 KB to 2 MB. Finally, because of the large access time of disk, DRAM caches always use write-back instead of write-through.

## Page Table
The VM system must have some way to determine if a virtual page is cached somewhere in DRAM. If so, the system must determine which physical page it is cached in. If there is a miss, the system must determine where the virtual page is stored on disk, select a victim page in physical memory, and copy the virtual page from disk to DRAM, replacing the victim page.
These capabilities are provided by a combination of operating system software, address translation hardware in the MMU (memory management unit), and a data structure stored in physical memory known as a page table that maps virtual pages to physical pages. The address translation hardware reads the page table each time it converts a virtual address to a physical address. The operating system is responsible for maintaining the contents of the page table and transferring pages back and forth between disk and DRAM.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/page-table-intro.png" align="center">

the address translation hardware uses the virtual address as an index to locate PTE 2 and read it from memory. Since the valid bit is set, the address translation hardware knows that VP 2 is cached in memory.

## Page Faults
A DRAM cache miss is known as a page fault. The address translation hardware reads PTE 3 from memory, infers from the valid bit that VP 3 is not cached, and triggers a page fault exception.
The page fault exception invokes a page fault exception handler in the kernel, which selects a victim page, in this case VP 4 stored in PP 3. If VP 4 has been modified, then the kernel copies it back to disk. In either case, the kernel modifies the page table entry for VP 4 to reflect the fact that VP 4 is no longer cached in main memory.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/page-fault.png" align="center">

In virtual memory lingo, blocks are known as pages. The activity of transferring a page between disk and memory is known as swapping or paging. Pages are swapped in (paged in) from disk to DRAM, and swapped out (paged out) from DRAM to disk. The strategy of waiting until the last moment to swap in a page, when a miss occurs, is known as demand paging.

Page allocation: **malloc**  operating system allocates a new page of virtual memory, for example, as a result of calling malloc.
In the example, VP 5 is allocated by creating room on disk and updating PTE 5 to point to the newly created page on disk.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/page-alloc.png" align="center">

### Locality and thrashing
Given the large miss penalties, we worry that paging will destroy program performance. In practice, virtual memory works well, mainly because of our old friend locality.<br/>
Although the total number of distinct pages that programs reference during an entire run might exceed the total size of physical memory, the principle of locality promises that at any point in time they will tend to work on a smaller set of active pages known as the working set or resident set. After an initial overhead where the working set is paged into memory, subsequent references to the working set result in hits, with no additional disk traffic.<br/>
As long as our programs have good temporal locality, virtual memory systems work quite well. But of course, not all programs exhibit good temporal locality. If the working set size exceeds the size of physical memory, then the program can produce an unfortunate situation known as thrashing, where pages are swapped in and out continuously.

## Memory management
operating systems provide a separate page table, and thus a separate virtual address space, for each process. Figure 9.9 shows the basic idea.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/process-page-table.png" align="center">

The combination of demand paging and separate virtual address spaces has a profound impact on the way that memory is used and managed in a system. In particular, VM simplifies linking and loading, the sharing of code and data, and allocating memory to applications.
* Simplifying linking. A separate address space allows each process to use the same basic format for its memory image, regardless of where the code and data actually reside in physical memory.
* Simplifying loading. Virtual memory also makes it easy to load executable and shared object files into memory. the .text and .data sections in ELF executables are contiguous. To load these sections into a newly created process, the Linux loader allocates a contiguous chunk of virtual pages starting at address 0x08048000 (32-bit address spaces) or 0x400000 (64-bit address spaces), marks them as invalid (i.e., not cached), and points their page table entries to the appropriate locations in the object file. The interesting point is that the loader never actually copies any data from disk into memory. The data is paged in automatically and on demand by the virtual memory system the first time each page is referenced, either by the CPU when it fetches an instruction, or by an executing instruction when it references a memory location. This notion of mapping a set of contiguous virtual pages to an arbitrary location in an arbitrary file is known as **memory mapping**.
* Simplifying sharing. Separate address spaces provide the operating system with a consistent mechanism for managing sharing between user processes and the operating system itself. In general, each process has its own private code, data, heap, and stack areas that are not shared with any other process. In this case, the operating system creates page tables that map the corresponding virtual pages to disjoint physical pages. However, in some instances it is desirable for processes to share code and data. For example, every process must call the same operating system kernel code, and every C program makes calls to routines in the standard C library such as printf. Rather than including separate copies of the kernel and standard C library in each process, the operating system can arrange for multiple processes to share a single copy of this code by mapping the appropriate virtual pages in different processes to the same physical pages
* Simplifying memory allocation. Virtual memory provides a simple mechanism for allocating additional memory to user processes. When a program running in a user process requests additional heap space (e.g., as a result of calling malloc), the operating system allocates an appropriate number, say, k, of contiguous virtual memory pages, and maps them to k arbitrary physical pages located anywhere in physical memory. Because of the way page tables work, there is no need for the operating system to locate k contiguous pages of physical memory. The pages can be scattered randomly in physical memory.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/shared-page.png" align="center">

## Address Translation
A control register in the CPU, the page table base register (PTBR) points to the current page table. The n-bit virtual address has two components: a p-bit virtual page offset (VPO) and an (n − p)-bit virtual page number (VPN). The MMU uses the VPN to select the appropriate PTE. For example, VPN 0 selects PTE 0, VPN 1 selects PTE 1, and so on. The corresponding physical address is the concatenation of the physical page number (PPN) from the page table entry and the VPO from the virtual address. Notice that since the physical and virtual pages are both P bytes, the physical page offset (PPO) is identical to the VPO.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/address-translation.png" align="center">

Steps that the CPU hardware performs when there is a page hit.
* Step 1: The processor generates a virtual address and sends it to the MMU.
* Step 2: The MMU generates the PTE address and requests it from the
cache/main memory.
* Step 3: The cache/main memory returns the PTE to the MMU.
* Step 3: The MMU constructs the physical address and sends it to cache/main memory.
* Step 4: The cache/main memory returns the requested data word to the processor.

Unlike a page hit, which is handled entirely by hardware, handling a page fault requires cooperation between hardware and the operating system kernel.
* Steps 1 to 3: The same as Steps 1 to 3 in page hit.
* Step 4: The valid bit in the PTE is zero, so the MMU triggers an exception, which transfers control in the CPU to a page fault exception handler in the operating system kernel.
* Step 5: The fault handler identifies a victim page in physical memory, and if that page has been modified, pages it out to disk.
* Step 6: The fault handler pages in the new page and updates the PTE in memory.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/address-pagehiit-fault.png" align="center">

## Translation Lookaside buffer - TLB
every time the CPU generates a virtual address, the MMU must refer to a PTE in order to translate the virtual address into a physical address. In the worst case, this requires an additional fetch from memory, at a cost of tens to hundreds of cycles. If the PTE happens to be cached in L1, then the cost goes down to one or two cycles. However, many systems try to eliminate even this cost by including a small cache of PTEs in the MMU called a translation lookaside buffer (TLB). A TLB is a small, virtually addressed cache where each line holds a block consisting of a single PTE.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/tlb.png" align="center">
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/tlb-hit-miss.png" align="center">

## Multi-level page table
we have assumed that the system uses a single page table to do address translation. But if we had a 32-bit address space, 4 KB pages, and a 4-byte PTE, then we would need a 4 MB page table resident in memory at all times. The common approach for compacting the page table is to use a hierarchy of page tables instead. The idea is easiest to understand with a concrete example. Consider a 32-bit virtual address space partitioned into 4 KB pages, with page table entries that are 4 bytes each.<br/>
Figure shows how we might construct a two-level page table hierarchy for this virtual address space. Each PTE in the level-1 table is responsible for mapping a 4 MB chunk of the virtual address space, where each chunk consists of 1024 contiguous pages. Given that the address space is 4 GB, 1024 PTEs are sufficient to cover the entire space.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/multi-level-page-table.png" align="center">

This scheme reduces memory requirements in two ways. First, if a PTE in the level 1 table is null, then the corresponding level 2 page table does not even have to exist. This represents a significant potential savings, since most of the 4 GB virtual address space for a typical program is unallocated. Second, only the level 1 table needs to be in main memory at all times. The level 2 page tables can be created and paged in and out by the VM system as they are needed, which reduces pressure on main memory. Only the most heavily used level 2 page tables need to be cached in main memory.

## Memory Mapping
Linux initializes the contents of a virtual memory area by associating it with an object on disk, a process known as Memory Mapping. Areas can be mapped to one of two types of objects:
* Regular file in the Unix file system: An area can be mapped to a contiguous section of a regular disk file, such as an executable object file. The file section is divided into page-sized pieces, with each piece containing the initial contents of a virtual page.
* Anonymous file: An area can also be mapped to an anonymous file, created by the kernel, that contains all binary zeros. The first time the CPU touches a virtual page in such an area, the kernel finds an appropriate victim page in physical memory, swaps out the victim page if it is dirty, overwrites the victim page with binary zeros, and updates the page table to mark the page as resident. Notice that no data is actually transferred between disk and memory. For this reason, pages in areas that are mapped to anonymous files are sometimes called demand-zero pages.<br/>
In either case, once a virtual page is initialized, it is swapped back and forth between a special swap file maintained by the kernel. The swap file is also known as the swap space or the swap area.

## Shared Objects
As we have seen, the process abstraction promises to provide each process with its own private virtual address space that is protected from errant writes or reads by other processes. However, many processes have identical read-only text areas. For example, each process that runs the Unix shell program tcsh has the same text area. Further, many programs need to access identical copies of read-only run-time library code. For example, every C program requires functions from the standard C library such as printf<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/Shared-objects.png" align="center">
An object can be mapped into an area of virtual memory as either a shared object or a private object. If a process maps a shared object into an area of its virtual address space, then any writes that the process makes to that area are visible to any other processes that have also mapped the shared object into their virtual memory. Further, the changes are also reflected in the original object on disk.<br/>
Changes made to an area mapped to a private object, on the other hand, are not visible to other processes, and any writes that the process makes to the area are not reflected back to the object on disk. A virtual memory area into which a shared object is mapped is often called a shared area. Similarly for a private area.<br/>
Since each object has a unique file name, the kernel can quickly determine that process 1 has already mapped this object and can point the page table entries in process 2 to the appropriate physical pages. The key point is that only a single copy of the shared object needs to be stored in physical memory, even though the object is mapped into multiple shared areas.<br/>
Private objects are mapped into virtual memory using a clever technique known as copy-on-write. A private object begins life in exactly the same way as a shared object, with only one copy of the private object stored in physical memory. For example, Figure 9.30(a) shows a case where two processes have mapped a private object into different areas of their virtual memories but share the same physical copy of the object. For each process that maps the private object, the page table entries for the corresponding private area are flagged as read-only, and the area struct is flagged as private copy-on-write. So long as neither process attempts to write to its respective private area, they continue to share a single copy of the object in physical memory. However, as soon as a process attempts to write to some page in the private area, the write triggers a protection fault.<br/>
When the fault handler notices that the protection exception was caused by the process trying to write to a page in a private copy-on-write area, it creates a new copy of the page in physical memory, updates the page table entry to point to the new copy, and then restores write permissions to the page, as shown in Figure 9.30(b). When the fault handler returns, the CPU reexecutes the write, which now proceeds normally on the newly created page.<br/>
By deferring the copying of the pages in private objects until the last possible moment, copy-on-write makes the most efficient use of scarce physical memory.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/shared-obj-copy-write.png" align="center">

## Fork Function
When the fork function is called by the current process, the kernel creates various data structures for the new process and assigns it a unique PID. To create the virtual memory for the new process, it creates exact copies of the current process’s mm_struct, area structs, and page tables. It flags each page in both processes as read-only, and flags each area struct in both processes as private copy- on-write.<br/>
When the fork returns in the new process, the new process now has an exact copy of the virtual memory as it existed when the fork was called. When either of the processes performs any subsequent writes, the copy-on-write mechanism creates new pages, thus preserving the abstraction of a private address space for each process.

## Dynamic Memory Allocation
While it is certainly possible to use the low-level mmap and munmap functions to create and delete areas of virtual memory, C programmers typically find it more convenient and more portable to use a dynamic memory allocator when they need to acquire additional virtual memory at run time.<br/>
A dynamic memory allocator maintains an area of a process’s virtual memory known as the heap. The heap is an area of demand-zero memory that begins immediately after the uninitialized bss area and grows upward (toward higher addresses). For each process, the kernel maintains a variable brk (pronounced “break”) that points to the top of the heap.<br/>
An allocator maintains the heap as a collection of various-sized blocks. Each block is a contiguous chunk of virtual memory that is either allocated or free. Allocators come in two basic styles. Both styles require the application to explicitly allocate blocks. They differ about which entity is responsible for freeing allocated blocks.
* Explicit allocators require the application to explicitly free any allocated blocks. For example, the C standard library provides an explicit allocator called the malloc package.
* Implicit allocators, on the other hand, require the allocator to detect when an allocated block is no longer being used by the program and then free the block. Implicit allocators are also known as garbage collectors<br/>
Dynamic memory allocators such as malloc can allocate or deallocate heap memory explicitly by using the mmap and munmap functions, or they can use the sbrk function<br/>
void *sbrk(intptr_t incr);
The sbrk function grows or shrinks the heap by adding incr to the kernel’s brk pointer. Programs free allocated heap blocks by calling the free function.<br/>
The total amount of virtual memory allocated by all of the processes in a system is limited by the amount of swap space on disk<br/>

## Fragmentation
There are two forms of fragmentation: internal fragmentation and external fragmentation.<br/>
Internal fragmentation occurs when an allocated block is larger than the pay- load. This might happen for a number of reasons. For example, the implementation of an allocator might impose a minimum size on allocated blocks that is greater than some requested payload.<br/>
External fragmentation occurs when there is enough aggregate free memory to satisfy an allocate request, but no single free block is large enough to handle the request.

## Allocator Implementation Issues
A practical allocator that strikes a better balance between throughput and utilization must consider the following issues:
* Free block organization: How do we keep track of free blocks?
* Placement: How do we choose an appropriate free block in which to place a newly allocated block?
* Splitting: After we place a newly allocated block in some free block, what do we do with the remainder of the free block?
* Coalescing: What do we do with a block that has just been freed?

## Implicit Free Lists
Any practical allocator needs some data structure that allows it to distinguish block boundaries and to distinguish between allocated and free blocks. Most allocators embed this information in the blocks themselves. One simple approach is shown in Figure below
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/malloc-heap-bloc.png" align="center">

In this case, a block consists of a one-word header, the payload, and possibly some additional padding. The header encodes the block size (including the header and any padding) as well as whether the block is allocated or free. If we impose a double-word alignment constraint, then the block size is always a multiple of eight and the 3 low-order bits of the block size are always zero. Thus, we need to store only the 29 high-order bits of the block size, freeing the remaining 3 bits to encode other information. In this case, we are using the least significant of these bits (the allocated bit) to indicate whether the block is allocated or free.<br/>
The header is followed by the payload that the application requested when it called malloc. The payload is followed by a chunk of unused padding that can be any size. There are a number of reasons for the padding. For example, the padding might be part of an allocator’s strategy for combating external fragmentation. Or it might be needed to satisfy the alignment requirement.
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/imp-free-list.png" align="center">

We call this organization an implicit free list because the free blocks are linked implicitly by the size fields in the headers. The allocator can indirectly traverse the entire set of free blocks by traversing all of the blocks in the heap. Notice that we need some kind of specially marked end block, in this example a terminating header with the allocated bit set and a size of zero<br/>
The advantage of an implicit free list is simplicity. A significant disadvantage is that the cost of any operation, such as placing allocated blocks, that requires a search of the free list will be linear in the total number of allocated and free blocks in the heap.<br/>
It is important to realize that the system’s alignment requirement and the allocator’s choice of block format impose a minimum block size on the allocator. No allocated or free block may be smaller than this minimum. For example, if we assume a double-word alignment requirement, then the size of each block must be a multiple of two words (8 bytes). Thus, the block format in Figure 9.35 induces a minimum block size of two words: one word for the header, and another to maintain the alignment requirement. Even if the application were to request a single byte, the allocator would still create a two-word block.<br/>

When an application requests a block of k bytes, the allocator searches the free list for a free block that is large enough to hold the requested block. The manner in which the allocator performs this search is determined by the placement policy. Some common policies are first fit, next fit, and best fit.<br/>
First fit searches the free list from the beginning and chooses the first free block that fits. Next fit is similar to first fit, but instead of starting each search at the beginning of the list, it starts each search where the previous search left off. Best fit examines every free block and chooses the free block with the smallest size that fits.<br/>

## Getting Addtional Heap memory
What happens if the allocator is unable to find a fit for the requested block? One option is to try to create some larger free blocks by merging (coalescing) free blocks that are physically adjacent in memory (next section). However, if this does not yield a sufficiently large block, or if the free blocks are already maximally coalesced, then the allocator asks the kernel for additional heap memory by calling the sbrk function. The allocator transforms the additional memory into one large free block, inserts the block into the free list, and then places the requested block in this new free block

## Coalescing Free Blocks
When the allocator frees an allocated block, there might be other free blocks that are adjacent to the newly freed block. Such adjacent free blocks can cause a phenomenon known as false fragmentation, where there is a lot of available free memory chopped up into small, unusable free blocks. For example, Figure 9.38 shows the result of freeing the block that was allocated in Figure 9.37. The result is two adjacent free blocks with payloads of three words each. As a result, a subsequent request for a payload of four words would fail, even though the aggregate size of the two free blocks is large enough to satisfy the request.<br/>
To combat false fragmentation, any practical allocator must merge adjacent free blocks in a process known as coalescing. This raises an important policy decision about when to perform coalescing. The allocator can opt for immediate coalescing by merging any adjacent blocks each time a block is freed. Or it can opt for deferred coalescing by waiting to coalesce free blocks at some later time. For example, the allocator might defer coalescing until some allocation request fails, and then scan the entire heap, coalescing all free blocks.<br/>
Immediate coalescing is straightforward and can be performed in constant time, but with some request patterns it can introduce a form of thrashing where a block is repeatedly coalesced and then split soon thereafter. For example, in Fig- ure 9.38 a repeated pattern of allocating and freeing a three-word block would introduce a lot of unnecessary splitting and coalescing.<br/>

## Garbage Collection
A garbage collector is a dynamic storage allocator that automatically frees al- located blocks that are no longer needed by the program. Such blocks are known as garbage. The process of automatically re- claiming heap storage is known as garbage collection.<br/>
A garbage collector views memory as a directed reachability graph of the form shown in Figure 9.49. The nodes of the graph are partitioned into a set of root nodes and a set of heap nodes. Each heap node corresponds to an allocated block in the heap. A directed edge p → q means that some location in block p points to some location in block q. Root nodes correspond to locations not in the heap that contain pointers into the heap. These locations can be registers, variables on the stack, or global variables in the read-write data area of virtual memory.<br/>
We say that a node p is reachable if there exists a directed path from any root node to p.<br/>
<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/gc-mem-view.png" align="center">

## Summary:
Virtual memory provides three important capabilities. First, it automatically caches recently used contents of the virtual address space stored on disk in main memory. The block in a virtual memory cache is known as a page. A reference to a page on disk triggers a page fault that transfers control to a fault handler in the operating system. The fault handler copies the page from disk to the main memory cache, writing back the evicted page if necessary. Second, virtual memory simplifies memory management, which in turn simplifies linking, sharing data between processes, the allocation of memory for processes, and program loading. Finally, virtual memory simplifies memory protection by incorporating protection bits into every page table entry.