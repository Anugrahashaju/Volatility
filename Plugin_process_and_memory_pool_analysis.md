# Windows Process and Memory Pool Analysis Plugins in Volatility 3

# Introduction

One of the primary goals of Windows memory forensics is to identify **running processes**, **hidden processes**, **process hierarchies**, **process privileges**, and **kernel memory objects**. Malware often attempts to hide its presence by manipulating process lists, escalating privileges, or allocating malicious objects in kernel memory.

Volatility 3 provides several plugins that allow investigators to analyze Windows processes and kernel memory structures directly from a RAM image.

This guide explains the following plugins:

* `windows.poolscanner`
* `windows.privileges.privs`
* `windows.pslist`
* `windows.psscan`
* `windows.pstree`

For each plugin, this guide covers:

* What the plugin does
* Why it is important
* How it works
* How to run it
* Artifacts collected
* Investigation use cases

---

# Table of Contents

1. windows.poolscanner
2. windows.privileges.privs
3. windows.pslist
4. windows.psscan
5. windows.pstree
6. Recommended Investigation Workflow
7. Summary

---

# 1. windows.poolscanner

## What is it?

`windows.poolscanner` is a low-level memory analysis plugin that scans Windows **kernel memory pools** for objects instead of relying on operating system linked lists.

Windows stores many kernel objects inside **memory pools**, including:

* Processes
* Threads
* Drivers
* Files
* Mutexes
* Registry objects
* Network objects

PoolScanner searches memory directly to recover these objects, even if they have been hidden or partially deleted.

---

## Why do we need it?

Malware and rootkits often remove kernel objects from normal Windows lists to avoid detection.

Since PoolScanner scans raw memory rather than trusting operating system structures, it can recover hidden objects that standard plugins may miss.

It is commonly used to detect:

* Hidden processes
* Hidden drivers
* Hidden threads
* DKOM (Direct Kernel Object Manipulation) attacks
* Rootkits

---

## How it works

Windows allocates kernel objects inside memory pools.

Each object begins with a **pool header** containing a **pool tag**, which identifies the object type.

PoolScanner searches physical memory for these pool tags and reconstructs the associated kernel objects.

### Workflow

```text
Memory Dump
      │
      ▼
Scan Physical Memory
      │
      ▼
Locate Pool Tags
      │
      ▼
Validate Kernel Objects
      │
      ▼
Recover Hidden Objects
```

---

## Command

```bash
python vol.py -f memory.raw windows.poolscanner
```

---

## Artifacts Collected

* Kernel Object Type
* Pool Tag
* Object Address
* Allocation Size
* Process Objects
* Thread Objects
* Driver Objects
* Hidden Kernel Objects

---

## Investigation Use Cases

* Rootkit detection
* Hidden process discovery
* DKOM detection
* Kernel object recovery

---

# 2. windows.privileges.privs

## What is it?

`windows.privileges.privs` displays the Windows security privileges assigned to each process access token.

Every Windows process runs under a security token that determines what actions it is permitted to perform.

---

## Why do we need it?

Attackers frequently steal or modify access tokens to gain elevated privileges.

Examples include:

* SeDebugPrivilege
* SeImpersonatePrivilege
* SeBackupPrivilege
* SeRestorePrivilege
* SeLoadDriverPrivilege

Processes possessing powerful privileges may indicate privilege escalation or token theft.

---

## How it works

The plugin reads each process token and extracts all assigned privileges.

For each privilege it reports:

* Present
* Enabled
* Default Enabled

---

## Command

```bash
python vol.py -f memory.raw windows.privileges.privs
```

---

## Artifacts Collected

* Process Name
* PID
* User SID
* Privilege Name
* Privilege State
* Enabled Privileges
* Disabled Privileges

---

## Common Suspicious Privileges

| Privilege                | Why It Matters                        |
| ------------------------ | ------------------------------------- |
| SeDebugPrivilege         | Access and manipulate other processes |
| SeImpersonatePrivilege   | Token impersonation attacks           |
| SeLoadDriverPrivilege    | Load kernel drivers                   |
| SeBackupPrivilege        | Read protected files                  |
| SeRestorePrivilege       | Modify protected system files         |
| SeTakeOwnershipPrivilege | Take ownership of objects             |

---

## Investigation Use Cases

* Privilege escalation detection
* Token theft investigations
* Lateral movement analysis
* Credential theft investigations

---

# 3. windows.pslist

## What is it?

`windows.pslist` lists all **active running processes** by reading the Windows Active Process List.

It is one of the most frequently used Volatility plugins and serves as the starting point for most Windows memory investigations.

---

## Why do we need it?

Before analyzing malware, investigators must know:

* Which processes are running
* Their Process IDs (PIDs)
* Parent Process IDs (PPIDs)
* Creation times
* Exit times (if available)

This provides an overview of the system at the time the memory image was captured.

---

## How it works

Windows maintains a doubly linked list called **ActiveProcessLinks**.

Each process is represented by an **EPROCESS** structure.

`pslist` walks this linked list and extracts process information.

### Workflow

```text
Memory Dump
      │
      ▼
Locate PsActiveProcessHead
      │
      ▼
Traverse ActiveProcessLinks
      │
      ▼
Read EPROCESS Structures
      │
      ▼
Display Active Processes
```

---

## Command

```bash
python vol.py -f memory.raw windows.pslist
```

---

## Artifacts Collected

* Process Name
* PID
* PPID
* Thread Count
* Handle Count
* Session ID
* Wow64 Status
* Create Time
* Exit Time

---

## Investigation Use Cases

* Enumerate running processes
* Identify suspicious executables
* Detect unusual parent-child relationships
* Build process timelines

---

# 4. windows.psscan

## What is it?

`windows.psscan` scans physical memory for **EPROCESS** structures instead of relying on the Active Process List.

Unlike `pslist`, it can recover hidden and terminated processes.

---

## Why do we need it?

Rootkits often unlink malicious processes from the Active Process List.

Since `psscan` searches raw memory, it can detect:

* Hidden processes
* Unlinked processes
* Terminated processes
* DKOM attacks

---

## How it works

The plugin scans physical memory for EPROCESS object signatures and validates each discovered structure.

It does not depend on Windows linked lists.

### Workflow

```text
Memory Dump
      │
      ▼
Scan Physical Memory
      │
      ▼
Locate EPROCESS Structures
      │
      ▼
Validate Objects
      │
      ▼
Recover Hidden Processes
```

---

## Command

```bash
python vol.py -f memory.raw windows.psscan
```

---

## Artifacts Collected

* Process Name
* PID
* PPID
* Create Time
* Exit Time
* Physical Address
* Hidden Process Indicators

---

## Investigation Use Cases

* Rootkit detection
* Hidden process discovery
* Terminated process recovery
* Compare with `pslist`

---

# 5. windows.pstree

## What is it?

`windows.pstree` displays running processes in a hierarchical parent-child tree.

Instead of presenting processes as a flat list, it illustrates which process created another process.

---

## Why do we need it?

Malware often launches child processes from unusual parent processes.

Examples include:

* `winword.exe` spawning `powershell.exe`
* `excel.exe` spawning `cmd.exe`
* `explorer.exe` spawning `rundll32.exe`
* `services.exe` spawning unknown executables

Viewing the process tree helps investigators quickly identify suspicious behavior.

---

## How it works

The plugin reads the Parent Process ID (PPID) stored within each EPROCESS structure and reconstructs the process hierarchy.

### Workflow

```text
Process List
      │
      ▼
Read PPID Values
      │
      ▼
Link Parent and Child Processes
      │
      ▼
Display Process Tree
```

---

## Command

```bash
python vol.py -f memory.raw windows.pstree
```

---

## Artifacts Collected

* Process Name
* PID
* Parent PID
* Child Processes
* Process Hierarchy
* Creation Time

---

## Investigation Use Cases

* Parent-child relationship analysis
* Malware execution chain reconstruction
* Suspicious process identification
* Timeline analysis

---

# Recommended Investigation Workflow

For process analysis, a recommended sequence is:

1. Run `windows.pslist` to enumerate active processes.
2. Run `windows.pstree` to visualize parent-child relationships.
3. Execute `windows.psscan` to detect hidden or terminated processes.
4. Compare `pslist` and `psscan` results to identify discrepancies.
5. Use `windows.privileges.privs` to inspect process privileges.
6. Run `windows.poolscanner` if rootkit activity or hidden kernel objects are suspected.

---

# Summary

These plugins form the foundation of Windows process analysis in Volatility 3. They enable investigators to enumerate processes, identify hidden malware, inspect security privileges, analyze process hierarchies, and recover kernel objects that may have been concealed by sophisticated attacks.

| Plugin                     | Primary Purpose                                       |
| -------------------------- | ----------------------------------------------------- |
| `windows.poolscanner`      | Scan kernel memory pools for hidden kernel objects    |
| `windows.privileges.privs` | Display process security privileges and access tokens |
| `windows.pslist`           | Enumerate active running processes                    |
| `windows.psscan`           | Recover hidden or terminated processes from memory    |
| `windows.pstree`           | Display parent-child process relationships            |

Together, these plugins provide a comprehensive view of process activity within a Windows memory image. By correlating information from all five plugins, investigators can detect hidden processes, privilege escalation, malicious process trees, and rootkit techniques that may not be visible through a single source of evidence.
