# Demand-Paged Virtual Memory Subsystem for x86 Operating System

## Overview

This project implements the **virtual memory subsystem** of an educational x86 operating system. The implementation introduces **two-level paging**, **demand paging**, **page fault handling**, **page replacement**, and **disk-backed swapping** to enable execution with limited physical memory.

The implementation is primarily contained in:

* `memory.c`
* `memory.h`

These files manage page allocation, page table creation, virtual-to-physical address translation, swapping, and page replacement.

---

# Objectives

The goal of this implementation is to:

* Implement two-level page tables for each process.
* Support demand paging through page faults.
* Allocate physical frames dynamically.
* Swap pages between memory and disk when physical memory is full.
* Implement a FIFO page replacement policy.
* Share kernel memory while maintaining isolated user address spaces.

---

# File Description

## memory.h

`memory.h` contains the constants, data structures, and function declarations required by the virtual memory subsystem.

### Constants

The file defines important paging parameters including:

* Page size (4 KB)
* Number of entries per page table
* Physical memory limits
* Page directory and page table masks
* Page entry flag definitions
* Address translation helpers

These constants are used throughout the memory manager to manipulate page tables and virtual addresses.

---

### Page Map Entry

Each physical page is represented using:

```c
page_map_entry_t
```

which stores:

* Virtual address currently mapped
* Swap location on disk
* Swap size
* Associated page directory
* Free/allocated status
* Pinned status (cannot be replaced)

This structure allows the operating system to keep track of every physical frame.

---

### Function Prototypes

The header declares routines for:

* Physical page allocation
* Page table initialization
* Page fault handling
* Swapping pages in/out
* Page replacement
* Address translation helpers

---

# memory.c

`memory.c` contains the complete implementation of the virtual memory subsystem.

---

## Address Translation

The functions

* `get_dir_idx()`
* `get_tab_idx()`

extract the page directory index and page table index from a virtual address using bit masking.

These functions simplify page table traversal.

---

## Physical Page Allocation

`page_alloc()` allocates a free physical page.

If a free page exists:

* marks it as allocated
* clears the page contents
* inserts the page into the FIFO queue (if pageable)

If physical memory is exhausted:

* invokes the page replacement policy
* swaps a victim page to disk
* reuses the freed frame

This enables execution with more virtual memory than available physical memory.

---

## Kernel Memory Initialization

`init_memory()` initializes the memory subsystem during kernel startup.

It performs the following tasks:

* Initializes the page map.
* Creates the kernel page directory.
* Allocates kernel page tables.
* Identity maps kernel memory.
* Configures page permissions.
* Initializes synchronization primitives.

Kernel pages are marked as **pinned**, preventing them from being selected for replacement.

---

## Process Address Space Creation

`setup_page_table()` creates a page directory for every new process.

The implementation:

* Shares kernel mappings with all processes.
* Creates user page tables.
* Allocates user stack pages.
* Sets page permissions.
* Initializes page table entries.

Each user process therefore has its own virtual address space while sharing the kernel.

---

## Page Fault Handling

`page_fault_handler()` is invoked whenever a process accesses a page that is not currently present in memory.

The handler:

1. Determines the faulting virtual address.
2. Allocates a free physical frame.
3. Swaps the required page into memory.
4. Updates the page tables.
5. Flushes the TLB entry.
6. Resumes execution.

This implements **demand paging**, where pages are loaded only when first accessed.

---

## Swapping

The implementation supports disk-backed virtual memory through:

* `page_swap_out()`
* `page_swap_in()`

### Swap Out

When memory becomes full:

* A victim page is selected.
* Its contents are written to disk.
* The page table entry is marked as not present.
* Swap metadata is stored.

### Swap In

Upon a page fault:

* The required page is read back from disk.
* A physical frame is allocated.
* The page table is updated.
* Execution continues transparently.

---

## Page Replacement Policy

The implementation uses the **FIFO (First-In, First-Out)** page replacement algorithm.

A queue maintains the order in which pageable frames were allocated.

When replacement is required:

1. The oldest unpinned page is selected.
2. The page is swapped to disk.
3. The freed frame is reused.

Pinned kernel pages are never selected for eviction.

---

## Synchronization

The memory subsystem uses locks to protect shared data structures during page allocation and swapping, preventing race conditions when multiple threads access memory management routines.

---

# Features Implemented

* Two-level page tables
* Demand paging
* Page fault handling
* Physical frame allocation
* FIFO page replacement
* Disk-backed swapping
* Kernel memory mapping
* User address space creation
* TLB updates after page table modifications
* Pinned kernel pages

---

# Building the Project

Clone the repository:

```bash
git clone https://github.com/<your-username>/<repository>.git
cd <repository>
```

Compile the operating system:

```bash
make
```

This generates the bootable kernel image.

---

# Running

Run the operating system using the provided emulator configuration.

For example:

```bash
make run
```

or

```bash
qemu-system-i386 -hda image
```

depending on the project setup.

---

# Concepts Demonstrated

* Operating Systems
* Virtual Memory
* Paging
* Demand Paging
* Page Tables
* Page Replacement
* Swapping
* Memory Management
* Interrupt Handling
* x86 Architecture

---

# Authors

Virtual Memory Subsystem (`memory.c`, `memory.h`) implemented as part of an Operating Systems project.
