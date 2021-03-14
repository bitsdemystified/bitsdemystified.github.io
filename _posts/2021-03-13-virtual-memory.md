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

With virtual addressing, the CPU accesses main memory by generating a vir- tual address (VA), which is converted to the appropriate physical address before being sent to the memory. The task of converting a virtual address to a physical one is known as address translation. Like exception handling, address translation requires close cooperation between the CPU hardware and the operating sys- tem. Dedicated hardware on the CPU chip called the memory management unit (MMU) translates virtual addresses on the fly, using a look-up table stored in main memory whose contents are managed by the operating system.

<img src="http://bitsdemystified.github.io/assets/img/virtual-memory/cpu-vm-intro.png" align="center">
