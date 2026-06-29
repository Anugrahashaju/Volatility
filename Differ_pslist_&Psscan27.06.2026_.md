# Process Enumeration in Memory Forensics
## Understanding `pslist` and `psscan` (Volatility)

## Introduction

During memory forensics, one of the first tasks performed by an investigator is identifying the processes that were running on the system at the time the RAM image was captured.

Volatility provides several plugins to enumerate processes. The two most commonly used plugins are:

- `pslist`
- `psscan`

Although both display process information, they work differently and provide different forensic artifacts.

Understanding the differences between these plugins is essential for detecting hidden, terminated, or malicious processes.

---

# 1. What is pslist?

`pslist` enumerates processes by walking the Windows Active Process Linked List.

It reads the operating system's active process list by following the `ActiveProcessLinks` pointers inside the EPROCESS structures.

In simple words:

> pslist shows the processes that Windows officially knows are currently running.

Because it relies on the linked list maintained by the Windows kernel, it only displays active processes that are properly linked into the operating system.

---

## How pslist Works

Windows maintains a doubly linked list containing every active process.

Each process contains an EPROCESS structure.

Inside this structure is a pointer called: Active process links


Volatility simply follows this linked list until it reaches the end.

Diagram:
System -> smss.exe -> csrss.exe -> wininit.exe -> services.exe -> lsass.exe -> explorer.exe

Any process removed from this linked list will not appear in pslist.

---

## Information Returned by pslist

Typical output includes:

- Process Name
- PID (Process ID)
- PPID (Parent Process ID)
- Thread Count
- Handle Count
- Session ID
- WOW64 Flag
- Creation Time
- Exit Time (if available)

Example:

---

# Advantages of pslist

- Fast
- Shows currently active processes
- Displays correct parent-child relationships
- Useful for timeline analysis
- Good starting point during investigations

---

# Limitations of pslist

It cannot detect:

- Hidden processes
- DKOM manipulated processes
- Unlinked EPROCESS structures
- Some terminated processes still remaining in memory

---

# 2. What is psscan?

`psscan` scans the entire RAM image looking for EPROCESS structures.

Instead of following Windows' linked list, it searches memory signatures and pool allocations to locate process objects.

In simple words:

> psscan searches the entire memory image for anything that looks like a process.

Even if Windows has removed the process from its active list, psscan may still discover it.

---

## How psscan Works

Instead of reading:

it scans raw physical memory.

It looks for Windows pool tags associated with process objects.

Example:
Memory

Whenever Volatility detects an EPROCESS signature, it extracts process information.

---

## Information Returned by psscan

Typically includes:

- Process Name
- PID
- PPID
- Offset
- Threads
- Handles
- Session
- Creation Time
- Exit Time

Example

Offset PID PPID Name

0x85b34040 4 0 System
0x85b44000 328 4 smss.exe
0x85b87000 468 328 csrss.exe
0x85d46000 4456 700 malware.exe


---

# Advantages of psscan

- Detects hidden processes
- Detects terminated processes
- Detects DKOM manipulated malware
- Scans the entire RAM
- Useful for anti-rootkit investigations

---

# Limitations of psscan

- Slower than pslist
- Can display terminated processes
- May show orphaned EPROCESS objects
- Requires further verification

---

# Difference Between pslist and psscan

| Feature | pslist | psscan |
|---------|---------|---------|
| Enumeration Method | Active Process Linked List | Memory Pool Scan |
| Reads ActiveProcessLinks | Yes | No |
| Detect Hidden Processes | No | Yes |
| Detect Terminated Processes | No | Yes |
| Detect DKOM Attacks | No | Yes |
| Speed | Fast | Slower |
| Shows Active Processes | Yes | Yes |
| Shows Unlinked Processes | No | Yes |

---

# Why Compare pslist and psscan?

One of the most common forensic techniques is comparing the outputs of pslist and psscan.

Possible situations:

### Case 1

Process appears in both.
pslist
explorer.exe

psscan
explorer.exe


Interpretation:

Normal process.

---

### Case 2

Process appears only in psscan.
pslist
(Not Found)

psscan
malware.exe


Possible reasons:

- Hidden by rootkit
- DKOM attack
- Recently terminated process
- Suspicious malware

This requires further investigation.

---

### Case 3

Process appears in pslist but not psscan.

Very uncommon.

Usually indicates:

- Memory corruption
- Incomplete memory dump
- Parsing issue

---

# Artifacts That Can Be Extracted

The following forensic artifacts can be obtained:

## 1. Process Name

Example
chrome.exe
cmd.exe
powershell.exe
malware.exe


Helps identify suspicious executables.

---

## 2. Process ID (PID)

Unique identifier assigned by Windows.

Useful for correlating:

- Network connections
- DLLs
- Handles
- Threads

---

## 3. Parent Process ID (PPID)

Shows which process created another process.

Example
explorer.exe -> cmd.exe -> powershell.exe


Useful for detecting process spawning.

---

## 4. Creation Time

Determines when the process started.

Useful for:

- Attack timelines
- Incident reconstruction

---

## 5. Exit Time

Shows whether a process has terminated.

Can indicate malware that executed briefly and exited.

---

## 6. Thread Count

High thread counts may indicate:

- Browsers
- Malware
- Cryptocurrency miners

---

## 7. Handle Count

Shows resources used by the process.

Large handle counts may indicate:

- Heavy activity
- Resource abuse

---

## 8. Session ID

Identifies the Windows user session.

Useful for distinguishing:

- System services
- User applications

---

## 9. Memory Offset

The location of the EPROCESS object in RAM.

Useful for:

- Deep forensic analysis
- Manual memory carving

---

## 10. Hidden Processes

Found by comparing: pslist vs psscan


A process found only in psscan may indicate stealth techniques.

---

## 11. DKOM Manipulation

Rootkits may unlink a process from the Active Process Linked List.

pslist cannot see it.

psscan still detects it.

---

## 12. Suspicious Parent-Child Relationships

Example
explorer.exe -> poweshell.exe -> cmd.exe -> rundll32.exe


May indicate malicious execution chains.

---

## 13. Malware Indicators

Look for:

- Random process names
- Executables in unusual locations
- Unsigned processes (verify with other plugins)
- Unexpected parent processes
- Processes with very short lifetimes

---

# Recommended Investigation Workflow

1. Run `pslist` to enumerate active processes.
2. Run `psscan` to identify all EPROCESS structures in memory.
3. Compare both outputs.
4. Investigate processes present only in `psscan`.
5. Review suspicious parent-child relationships using `pstree`.
6. Inspect command-line arguments with `cmdline`.
7. Check loaded DLLs using `dlllist`.
8. Review network activity using `netscan`.
9. Dump suspicious process memory using `procdump`.
10. Correlate findings with timeline analysis.

---

# Related Volatility Plugins

| Plugin | Purpose |
|---------|---------|
| pslist | Lists active processes |
| psscan | Scans memory for EPROCESS objects |
| pstree | Displays process hierarchy |
| cmdline | Shows process command-line arguments |
| dlllist | Lists loaded DLLs |
| handles | Displays open handles |
| netscan | Lists active network connections |
| malfind | Detects injected code and suspicious memory regions |
| procdump | Dumps process executables for analysis |
| vadinfo | Displays virtual memory regions |

---

# Key Takeaways

- `pslist` reads the Windows active process list and is ideal for identifying processes that the operating system currently recognizes.
- `psscan` scans raw memory for EPROCESS structures, allowing investigators to recover hidden, terminated, or unlinked processes.
- Comparing `pslist` and `psscan` is a fundamental memory forensic technique for detecting stealth malware, DKOM attacks, and suspicious process activity.
- The process information obtained from these plugins—such as PID, PPID, creation time, exit time, threads, handles, and session IDs—serves as a foundation for deeper investigations using other Volatility plugins.
