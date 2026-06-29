---

# Memory Management and Pages

One of the primary responsibilities of the operating system is managing **memory (RAM)**. Every running process requires memory to store its executable code, variables, libraries, and data. Since physical RAM is limited, Windows uses **Virtual Memory** and divides memory into small fixed-size blocks called **pages**.

Instead of allocating one large continuous block of memory to a process, Windows allocates many small pages. This approach makes memory management more efficient and flexible.

---

# What is a Page?

A **page** is the smallest fixed-size block of virtual memory managed by the operating system.

On most modern Windows systems:

- Page Size = **4 KB (4096 bytes)**

Some systems also support larger page sizes (such as 2 MB or 1 GB), but the standard page size used by Windows applications is **4 KB**.

Imagine a book:

```
Book
│
├── Page 1
├── Page 2
├── Page 3
└── Page 4
```

Similarly, memory is divided into pages:

```
Virtual Memory

+-------------+
| Page 1      |
+-------------+
| Page 2      |
+-------------+
| Page 3      |
+-------------+
| Page 4      |
+-------------+
```

---

# Why Does Windows Use Pages?

Without pages, Windows would need to find one large continuous block of memory for every process.

Example:

Suppose a program needs **100 MB** of RAM.

Without paging:

```
RAM

□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□
Need one continuous 100 MB block
```

If RAM becomes fragmented, Windows may not find a sufficiently large contiguous block.

With paging:

```
RAM

Page 1
Page 2
Page 3
Page 4
Page 5
...
Page 25,600
```

The pages can be stored anywhere in physical memory, allowing Windows to use RAM more efficiently.

Benefits include:

- Efficient memory allocation
- Reduced fragmentation
- Easier memory protection
- Virtual memory support
- Faster memory management

---

# Virtual Memory

Every process receives its own **Virtual Address Space**.

For example:

```
Chrome.exe

Virtual Memory

0x00000000

↓

Executable Code

↓

DLLs

↓

Heap

↓

Stack

↓

Unused Space
```

The addresses used by the process are **virtual addresses**, not physical RAM addresses.

Windows translates virtual addresses into physical addresses using the **Memory Management Unit (MMU)** and **page tables**.

---

# What is a Page Table?

A **Page Table** is a data structure maintained by the operating system that maps **virtual memory pages** to **physical memory frames**.

Example:

```
Virtual Page      Physical Frame

Page 0  ---------> Frame 15

Page 1  ---------> Frame 200

Page 2  ---------> Frame 85

Page 3  ---------> Frame 910
```

When a process accesses memory, the CPU consults the page table to determine the corresponding location in physical RAM.

---

# Virtual Address vs Physical Address

Every process works with virtual addresses.

Example:

```
Application

Address:
0x00007FF6A1230000
```

The CPU cannot access RAM directly using this virtual address.

Instead:

```
Application

↓

Virtual Address

↓

Page Table

↓

Physical Address

↓

RAM
```

This translation is completely transparent to the application.

---

# What is a Page Frame?

A **Page Frame** is a fixed-size block of **physical RAM** that stores one page.

```
Virtual Memory

Page 5

↓

Page Table

↓

Physical RAM

Frame 128
```

Think of it like this:

- **Page** → Virtual memory block
- **Frame** → Physical RAM block

Both are typically **4 KB** in size.

---

# Demand Paging

Windows does not load an entire program into RAM immediately.

Instead, it loads pages **only when they are needed**.

Example:

```
Program

100 MB

↓

Only 12 MB currently needed

↓

Windows loads only those pages
```

This technique is called **Demand Paging**.

Advantages:

- Faster application startup
- Reduced RAM usage
- Better multitasking
- Improved overall system performance

---

# What is a Page Fault?

A **Page Fault** occurs when a process tries to access a page that is not currently loaded into physical memory.

Example:

```
Application requests Page 25

↓

Page not in RAM

↓

Page Fault

↓

Windows loads the page from disk

↓

Execution continues
```

Page faults are a normal part of modern operating systems.

There are two types:

### Soft Page Fault

The required page already exists in RAM but is not mapped to the current process.

### Hard Page Fault

The required page must be read from the page file or another storage device.

Hard page faults are slower because they involve disk I/O.

---

# What is the Page File?

When RAM becomes full, Windows moves less frequently used pages to a special disk file called the **Page File** (`pagefile.sys`).

```
RAM Full

↓

Move inactive pages

↓

pagefile.sys

↓

Free RAM for active processes
```

This allows Windows to continue running applications even when physical RAM is nearly exhausted.

However, accessing the page file is much slower than accessing RAM.

---

# Pages in Digital Forensics

Pages are extremely important in memory forensics because they may contain:

- Running process code
- Passwords
- Encryption keys
- Browser history
- Network data
- Malware code
- Injected shellcode
- Command-line arguments
- Chat messages
- File fragments

Even after a process terminates, some of its memory pages may remain in RAM, allowing forensic investigators to recover valuable evidence.

Tools such as **Volatility** analyze these pages to reconstruct process memory and identify malicious activity.

---

# Summary

- Memory is divided into fixed-size **pages** (typically **4 KB**).
- Each process uses its own **virtual address space**.
- Windows translates virtual addresses to physical RAM using **page tables**.
- Physical RAM is divided into **page frames**.
- **Demand Paging** loads memory only when needed.
- A **Page Fault** occurs when a requested page is not currently in RAM.
- The **page file (`pagefile.sys`)** extends available memory by storing inactive pages on disk.
- Understanding pages is essential for operating systems, memory management, and digital forensics because most evidence in RAM is organized and managed at the page level.
