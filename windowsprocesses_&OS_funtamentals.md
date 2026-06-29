# Understanding Windows Processes and Operating System Fundamentals

## Introduction

Every program you open on your computer—such as Chrome, Notepad, Microsoft Word, or Command Prompt—must be managed by the operating system (OS). The operating system is responsible for loading programs into memory, allocating CPU time, managing resources, and ensuring that multiple programs can run simultaneously without interfering with one another.

The basic unit of execution in an operating system is called a **process**. Understanding processes is essential for system administrators, software developers, malware analysts, and digital forensic investigators.

---

# Table of Contents

1. What is an Operating System?
2. What is a Program?
3. What is a Process?
4. Process Lifecycle
5. Why Do We Need Processes?
6. Process Components
7. What is a Thread?
8. Multithreading
9. What is a Session?
10. How Windows Works Internally
11. Windows Boot Process
12. Windows Process Creation
13. Parent and Child Processes
14. Windows Process Hierarchy
15. Windows Process States
16. Process Scheduling
17. Memory Management
18. Why Process Analysis Matters in Digital Forensics
19. Common Windows System Processes
20. Summary

---

# 1. What is an Operating System?

An **Operating System (OS)** is system software that acts as an interface between the user, applications, and computer hardware.

It manages:

- CPU
- Memory (RAM)
- Hard Disk
- Keyboard
- Mouse
- Network
- Files
- Running applications

Without an operating system, application software cannot communicate directly with the hardware.

Examples:

- Windows
- Linux
- macOS
- Android

---

# 2. What is a Program?

A **Program** is simply a collection of instructions stored on disk.

Examples:

```
chrome.exe
notepad.exe
cmd.exe
python.exe
```

A program is **passive**.

It does nothing until it is executed.

Think of a program like a recipe in a cookbook.

The recipe exists, but nothing happens until someone starts cooking.

---

# 3. What is a Process?

A **Process** is a program that is currently executing.

When you double-click an application:

```
Program
↓

Operating System

↓

Process Created

↓

Running in RAM
```

A process contains:

- Executable code
- Virtual memory
- Process ID (PID)
- Threads
- Handles
- Security Token
- Loaded DLLs
- Stack
- Heap

Example:

```
Program:

chrome.exe

↓

Running

↓

Process

PID = 4520
Memory = 500 MB
Threads = 85
```

A process is an active instance of a program.

---

# 4. Process Lifecycle

Every process goes through several stages.

```
New

↓

Ready

↓

Running

↓

Waiting

↓

Running

↓

Terminated
```

## New

The process is created.

## Ready

Waiting for CPU.

## Running

Currently executing.

## Waiting

Waiting for disk, keyboard, network, or another resource.

## Terminated

Execution has finished.

---

# 5. Why Do We Need Processes?

Imagine opening:

- Chrome
- Spotify
- VS Code
- Word
- Calculator

Without processes:

- One application would overwrite another.
- Programs could not run simultaneously.
- Memory would become corrupted.

Processes provide:

- Isolation
- Security
- Resource management
- Scheduling
- Stability

Each process has its own:

- Memory
- Variables
- Stack
- Handles
- Security context

---

# 6. Components of a Process

Every process contains:

## Process ID (PID)

Unique identifier.

Example:

```
chrome.exe

PID = 4520
```

---

## Parent Process ID (PPID)

The process that created another process.

Example:

```
explorer.exe

↓

cmd.exe

↓

python.exe
```

---

## Virtual Memory

Each process receives its own virtual address space.

Example:

```
Chrome

0x00000000

↓

Heap

↓

DLLs

↓

Stack
```

Other processes cannot directly access this memory.

---

## Handles

References to operating system objects.

Examples:

- Files
- Registry
- Network sockets
- Mutexes
- Events

---

## Security Token

Contains:

- User SID
- Group memberships
- Privileges
- Integrity level

Determines what the process is allowed to do.

---

# 7. What is a Thread?

A **Thread** is the smallest unit of execution inside a process.

A process must contain **at least one thread**.

```
Process

│

├── Thread 1

├── Thread 2

├── Thread 3

└── Thread 4
```

Think of:

Process = Company

Threads = Employees

The company cannot do work without employees.

Similarly,

A process cannot execute without threads.

---

# Why Threads?

Threads allow:

- Parallel execution
- Faster performance
- Better responsiveness

Example:

Chrome browser

One thread:

- UI

Another:

- Downloading

Another:

- Audio

Another:

- JavaScript

Everything works simultaneously.

---

# 8. Multithreading

Multithreading means:

A process contains multiple threads executing concurrently.

Example:

```
Video Player

Thread 1

Video decoding

Thread 2

Audio playback

Thread 3

Subtitle rendering

Thread 4

User interface
```

Benefits:

- Faster applications
- Better CPU utilization
- Improved responsiveness

Disadvantages:

- Synchronization problems
- Race conditions
- Deadlocks

---

# 9. What is a Session?

A **Session** represents a user login environment.

Every logged-in user receives a separate session.

Example:

```
Session 0

Windows Services

Session 1

User A

Session 2

Remote Desktop User
```

Processes running in one session cannot normally interact with another.

---

# Session 0 Isolation

Older Windows versions allowed services and users to share Session 0.

Modern Windows isolates services in Session 0 to improve security.

Users typically log into Session 1 or higher.

---

# 10. How Windows Works Internally

Simplified architecture:

```
Applications

↓

Windows API

↓

Kernel

↓

Hardware
```

Applications never communicate directly with hardware.

Everything goes through the Windows kernel.

The kernel manages:

- CPU scheduling
- Memory
- Processes
- Threads
- Drivers
- I/O

---

# 11. Windows Boot Process

When Windows starts:

```
Power On

↓

BIOS / UEFI

↓

Windows Boot Manager

↓

Windows Loader

↓

ntoskrnl.exe

↓

Kernel Initialization

↓

smss.exe

↓

csrss.exe

↓

wininit.exe

↓

services.exe

↓

lsass.exe

↓

winlogon.exe

↓

explorer.exe
```

These are core Windows processes.

---

# 12. How Windows Creates a Process

Suppose the user opens Notepad.

```
Double Click

↓

explorer.exe

↓

CreateProcess()

↓

Kernel

↓

Allocate Memory

↓

Create EPROCESS

↓

Create Initial Thread

↓

Load DLLs

↓

Start Execution
```

Windows performs:

1. Creates EPROCESS object
2. Assigns PID
3. Creates virtual memory
4. Loads executable
5. Loads required DLLs
6. Creates the primary thread
7. Schedules execution

---

# 13. Parent and Child Processes

Processes create other processes.

Example:

```
explorer.exe

↓

cmd.exe

↓

powershell.exe

↓

python.exe
```

Parent:

Creates another process.

Child:

The newly created process.

---

# 14. Windows Process Hierarchy

Typical process tree:

```
System

└── smss.exe

    ├── csrss.exe

    ├── wininit.exe

    │

    ├── services.exe

    │      ├── svchost.exe

    │      ├── spoolsv.exe

    │      └── WmiPrvSE.exe

    │

    ├── lsass.exe

    │

    └── winlogon.exe

            └── explorer.exe

                    ├── chrome.exe

                    ├── notepad.exe

                    └── cmd.exe
```

This hierarchy is useful during forensic investigations.

---

# 15. Windows Process States

Processes can be:

- Running
- Ready
- Waiting
- Suspended
- Terminated

Investigators often identify suspended or terminated processes during memory analysis.

---

# 16. Process Scheduling

The Windows Scheduler decides:

- Which process runs
- When it runs
- For how long

The scheduler rapidly switches between processes.

This creates the illusion that all applications run simultaneously.

---

# 17. Memory Management

Every process receives its own virtual memory.

Advantages:

- Isolation
- Security
- Stability

If one application crashes:

Other applications continue running.

---

# 18. Why Process Analysis Matters in Digital Forensics

Processes reveal:

- Running malware
- Hidden processes
- Parent-child relationships
- Command-line arguments
- Injected code
- Network activity
- User actions

Volatility plugins used:

- pslist
- psscan
- pstree
- cmdline
- dlllist
- handles
- malfind
- netscan

---

# 19. Common Windows System Processes

| Process | Purpose |
|----------|---------|
| System | Core Windows kernel process |
| smss.exe | Session Manager |
| csrss.exe | Client/Server Runtime |
| wininit.exe | Initializes Windows services |
| services.exe | Service Control Manager |
| svchost.exe | Hosts Windows services |
| lsass.exe | Local Security Authority |
| winlogon.exe | Handles user logon |
| explorer.exe | Windows graphical shell |
| cmd.exe | Command Prompt |
| powershell.exe | PowerShell interpreter |

---

# 20. Summary

Understanding processes is fundamental to operating systems and digital forensics.

Key concepts include:

- A **program** is a file stored on disk.
- A **process** is a running instance of a program.
- Every process has a unique **PID** and its own memory space.
- A **thread** is the smallest unit of execution within a process.
- **Multithreading** allows multiple tasks to run concurrently within the same process.
- A **session** isolates users and services into separate execution environments.
- The **Windows kernel** manages process creation, memory allocation, scheduling, and hardware access.
- During system startup, Windows creates a hierarchy of core processes that eventually launch the user's desktop (`explorer.exe`).
- Digital forensic tools such as Volatility analyze process structures in memory to identify active, hidden, or suspicious processes, making process analysis one of the first and most important steps in memory forensics.
