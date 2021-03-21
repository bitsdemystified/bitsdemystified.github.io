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


<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/.png" align="center">
