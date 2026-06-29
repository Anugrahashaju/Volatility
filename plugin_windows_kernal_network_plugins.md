# Windows Kernel, Network, and Executable Analysis Plugins in Volatility 3

## Introduction

Memory forensics is not limited to examining user processes. Many advanced threats operate within the Windows kernel, manipulate kernel modules, create hidden synchronization objects, establish covert network connections, or execute malicious Portable Executable (PE) files directly from memory.

Volatility 3 provides several plugins that help investigators analyze **kernel modules**, **network artifacts**, **kernel threads**, **Portable Executable (PE) files**, and **Windows synchronization objects**.

This guide explains the following plugins:

* `windows.modscan`
* `windows.modules`
* `windows.mutantscan`
* `windows.netscan`
* `windows.netstat`
* `windows.orphan_kernel_threads`
* `windows.pesymbols`
* `windows.pedump`

For each plugin, this guide covers:

* What the plugin does
* Why it is important
* How it works
* How to run it
* Artifacts collected
* Investigation use cases

---

# Table of Contents

1. windows.modscan
2. windows.modules
3. windows.mutantscan
4. windows.netscan
5. windows.netstat
6. windows.orphan_kernel_threads
7. windows.pesymbols
8. windows.pedump
9. Recommended Investigation Workflow
10. Summary

---

# 1. windows.modscan

## What is it?

`windows.modscan` scans physical memory for **kernel module (driver)** objects.

Unlike `windows.modules`, which reads the active module list maintained by Windows, `modscan` searches raw memory to recover module objects that may have been hidden or partially removed.

---

## Why do we need it?

Kernel-mode rootkits often unlink malicious drivers from Windows' active module list.

Because `modscan` scans memory directly, it can recover hidden or unlinked drivers.

This makes it valuable for rootkit detection.

---

## How it works

The plugin scans memory for **_LDR_DATA_TABLE_ENTRY** structures and validates them as kernel modules.

It does not rely on Windows linked lists.

---

## Command

```bash
python vol.py -f memory.raw windows.modscan
```

---

## Artifacts Collected

* Driver Name
* Driver Base Address
* Driver Size
* Driver Path
* Driver Object
* Hidden Drivers
* Unlinked Kernel Modules

---

## Investigation Use Cases

* Rootkit detection
* Hidden driver discovery
* Driver integrity verification
* Kernel malware analysis

---

# 2. windows.modules

## What is it?

`windows.modules` lists every kernel module (driver) currently loaded by Windows.

It is similar to `lsmod` in Linux.

---

## Why do we need it?

Kernel drivers execute with the highest system privileges.

Malicious drivers may:

* Hide files
* Hide processes
* Hide registry keys
* Disable security software
* Intercept system calls

Listing loaded drivers is one of the first steps in kernel forensic analysis.

---

## How it works

The plugin walks the Windows **PsLoadedModuleList**, which stores all active kernel modules.

---

## Command

```bash
python vol.py -f memory.raw windows.modules
```

---

## Artifacts Collected

* Driver Name
* Driver Base Address
* Driver Size
* Driver Path
* Load Order
* Loaded Kernel Modules

---

## Investigation Use Cases

* Identify suspicious drivers
* Compare with `windows.modscan`
* Detect unsigned drivers
* Driver timeline analysis

---

# 3. windows.mutantscan

## What is it?

`windows.mutantscan` scans memory for **Mutant (Mutex)** kernel objects.

A mutex (Mutant object in Windows) is a synchronization mechanism that prevents multiple processes or threads from accessing the same resource simultaneously.

---

## Why do we need it?

Many malware families create unique mutex names to ensure that only one copy of the malware runs.

Example:

```
Global\MyMalwareMutex
```

The mutex name can uniquely identify a malware family.

---

## How it works

The plugin scans memory for `_KMUTANT` objects and extracts their associated metadata.

---

## Command

```bash
python vol.py -f memory.raw windows.mutantscan
```

---

## Artifacts Collected

* Mutex Name
* Object Address
* Owner Thread
* Process
* Handle Count
* Reference Count

---

## Investigation Use Cases

* Malware identification
* Malware family attribution
* Process synchronization analysis
* Persistence investigations

---

# 4. windows.netscan

## What is it?

`windows.netscan` scans memory for active and recently closed network connections.

It is one of the most frequently used networking plugins in Volatility.

---

## Why do we need it?

Attackers often communicate with:

* Command-and-Control (C2) servers
* Remote shells
* Botnets
* Malware download servers

Network artifacts reveal attacker communication.

---

## How it works

The plugin scans memory for TCP and UDP endpoint structures.

It identifies:

* TCP Connections
* UDP Endpoints
* Listening Ports
* Remote Connections

---

## Command

```bash
python vol.py -f memory.raw windows.netscan
```

---

## Artifacts Collected

* Local IP Address
* Remote IP Address
* Local Port
* Remote Port
* Protocol
* Connection State
* PID
* Process Name
* Connection Creation Time

---

## Investigation Use Cases

* Detect C2 traffic
* Identify reverse shells
* Investigate suspicious connections
* Timeline reconstruction

---

# 5. windows.netstat

## What is it?

`windows.netstat` displays active network connections similarly to the Windows **netstat** command.

Unlike `netscan`, it relies on operating system structures rather than scanning memory.

---

## Why do we need it?

Useful for quickly identifying active sockets and correlating them with running processes.

---

## How it works

The plugin enumerates active networking structures maintained by Windows.

---

## Command

```bash
python vol.py -f memory.raw windows.netstat
```

---

## Artifacts Collected

* Local Address
* Remote Address
* Ports
* Connection State
* Process Name
* PID

---

## Investigation Use Cases

* Active connection analysis
* Compare with `windows.netscan`
* Verify process network activity

---

# 6. windows.orphan_kernel_threads

## What is it?

Detects kernel threads that are no longer associated with a valid kernel module.

These are known as **orphan kernel threads**.

---

## Why do we need it?

Kernel malware sometimes unloads its driver while leaving malicious kernel threads running.

The thread continues executing even though its originating driver has disappeared.

---

## How it works

The plugin compares kernel thread start addresses with loaded kernel modules.

Threads executing outside valid drivers are flagged as suspicious.

---

## Command

```bash
python vol.py -f memory.raw windows.orphan_kernel_threads
```

---

## Artifacts Collected

* Thread Address
* Thread ID
* Start Address
* Missing Driver
* Kernel Thread Information

---

## Investigation Use Cases

* Kernel rootkit detection
* Hidden driver detection
* Thread persistence analysis

---

# 7. windows.pesymbols

## What is it?

`windows.pesymbols` extracts exported symbols from Portable Executable (PE) files loaded in memory.

Symbols represent functions, variables, and exported routines inside executables and DLLs.

---

## Why do we need it?

Symbols help analysts understand:

* API functions
* Exported routines
* Reverse engineering targets
* Malware capabilities

---

## How it works

The plugin parses the PE Export Table and Symbol Table.

---

## Command

```bash
python vol.py -f memory.raw windows.pesymbols
```

---

## Artifacts Collected

* Exported Functions
* Function Addresses
* Symbol Names
* DLL Information
* Executable Metadata

---

## Investigation Use Cases

* Reverse engineering
* Malware capability analysis
* API identification

---

# 8. windows.pedump

## What is it?

`windows.pedump` extracts Portable Executable (PE) files directly from memory.

This includes:

* EXE files
* DLLs
* Drivers

---

## Why do we need it?

Sometimes malware only exists in memory.

PEDump allows investigators to recover the executable for further analysis using:

* Ghidra
* IDA Pro
* PEStudio
* Detect It Easy (DIE)
* VirusTotal

---

## How it works

The plugin locates PE headers in memory, reconstructs the executable image, and writes it to disk.

---

## Command

```bash
python vol.py -f memory.raw windows.pedump --pid <PID> --dump
```

Example:

```bash
python vol.py -f memory.raw windows.pedump --pid 4456 --dump
```

---

## Artifacts Collected

* Executable File
* DLL Files
* Driver Files
* PE Headers
* Import Table
* Export Table
* Section Information

---

## Investigation Use Cases

* Malware extraction
* Static malware analysis
* Reverse engineering
* Hash calculation
* Signature generation

---

# Recommended Investigation Workflow

A recommended workflow for kernel, network, and executable analysis is:

1. Run `windows.modules` to list active drivers.
2. Run `windows.modscan` to identify hidden or unlinked drivers.
3. Compare the results of `modules` and `modscan`.
4. Run `windows.orphan_kernel_threads` to detect suspicious kernel threads.
5. Execute `windows.mutantscan` to identify malware mutexes.
6. Run `windows.netstat` to view active network connections.
7. Run `windows.netscan` to recover active and recently closed network connections.
8. Use `windows.pedump` to extract suspicious executables from memory.
9. Analyze exported functions with `windows.pesymbols`.
10. Perform static analysis on dumped executables using reverse engineering tools.

---

# Summary

These Volatility 3 plugins focus on **kernel modules**, **network activity**, **kernel threads**, **Portable Executable analysis**, and **Windows synchronization objects**, enabling investigators to uncover advanced malware that operates beyond ordinary user processes.

| Plugin                          | Primary Purpose                                               |
| ------------------------------- | ------------------------------------------------------------- |
| `windows.modscan`               | Scan memory for hidden or unlinked kernel drivers             |
| `windows.modules`               | List currently loaded Windows kernel modules                  |
| `windows.mutantscan`            | Recover Windows mutex (Mutant) objects used by processes      |
| `windows.netscan`               | Recover active and closed TCP/UDP network connections         |
| `windows.netstat`               | Display active network connections using OS structures        |
| `windows.orphan_kernel_threads` | Detect kernel threads without valid backing drivers           |
| `windows.pesymbols`             | Extract exported symbols from PE files in memory              |
| `windows.pedump`                | Recover executable files (EXE, DLL, SYS) directly from memory |

Together, these plugins provide investigators with comprehensive visibility into Windows kernel activity, network communications, synchronization objects, and executable code, making them essential tools for malware hunting, incident response, and advanced memory forensic investigations.
