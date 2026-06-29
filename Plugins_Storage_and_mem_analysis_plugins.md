# Windows Storage and Memory Analysis Plugins in Volatility 3

## Introduction

Windows stores critical forensic artifacts not only in running processes but also in memory structures related to storage devices, file systems, and virtual memory. During a memory forensic investigation, analysts can recover information about the **Master Boot Record (MBR)**, **memory mappings**, **Master File Table (MFT)** entries, **Alternate Data Streams (ADS)**, and **resident file data**.

Volatility 3 provides several plugins that extract these artifacts directly from a memory image, allowing investigators to identify hidden files, bootkits, fileless malware, deleted metadata, and process memory mappings.

This guide explains the following plugins:

* `windows.mbrscan`
* `windows.memmap`
* `windows.mftscan.ADS`
* `windows.mftscan.MFTScan`
* `windows.mftscan.ResidentData`

For each plugin, this guide covers:

* What the plugin does
* Why it is important
* How it works
* How to run it
* Artifacts collected
* Investigation use cases

---

# Table of Contents

1. windows.mbrscan
2. windows.memmap
3. windows.mftscan.ADS
4. windows.mftscan.MFTScan
5. windows.mftscan.ResidentData
6. Recommended Investigation Workflow
7. Summary

---

# 1. windows.mbrscan

## What is it?

`windows.mbrscan` scans physical memory for **Master Boot Record (MBR)** structures.

The **Master Boot Record** is the first sector of a storage device (Sector 0) and contains:

* Bootloader code
* Disk signature
* Partition table

Because bootkits and rootkits often modify the MBR to execute before Windows starts, analyzing the MBR is an important step in malware investigations.

---

## Why do we need it?

Attackers can infect the MBR to gain control before the operating system loads.

Examples include:

* Bootkits
* MBR Rootkits
* Persistent Malware

Since the MBR executes before Windows starts, malicious code located there may bypass many security products.

---

## How it works

The plugin scans physical memory looking for:

* Valid MBR signatures
* Boot code
* Partition table entries
* Disk signatures

It compares discovered MBR structures against the expected MBR format and reports suspicious or modified boot records.

---

## Command

```bash
python vol.py -f memory.raw windows.mbrscan
```

---

## Artifacts Collected

* Master Boot Record
* Disk Signature
* Bootloader Code
* Partition Table Entries
* MBR Signature (0x55AA)
* Possible Bootkit Indicators

---

## Investigation Use Cases

* Detect MBR rootkits
* Detect bootkits
* Verify boot record integrity
* Recover partition information
* Analyze pre-boot malware

---

# 2. windows.memmap

## What is it?

`windows.memmap` displays the memory map of a process.

It shows how a process's virtual memory is mapped to physical memory.

This plugin is commonly used after identifying a suspicious process with plugins such as `pslist`, `psscan`, or `malfind`.

---

## Why do we need it?

Every running process has its own virtual memory layout.

Understanding this layout helps investigators:

* Locate injected code
* Identify executable pages
* Dump process memory
* Correlate suspicious memory regions
* Analyze memory allocation

---

## How it works

The plugin walks the process page tables and translates:

* Virtual Addresses
* Physical Addresses
* Page Offsets

It displays how each virtual page is mapped into RAM.

---

## Command

```bash
python vol.py -f memory.raw windows.memmap --pid <PID>
```

Example:

```bash
python vol.py -f memory.raw windows.memmap --pid 4456
```

---

## Artifacts Collected

* Virtual Addresses
* Physical Addresses
* Memory Offsets
* Memory Pages
* Page Permissions
* Process Memory Layout

---

## Investigation Use Cases

* Process memory reconstruction
* Malware memory extraction
* Memory carving
* Shellcode analysis
* Virtual-to-physical address translation

---

# 3. windows.mftscan.ADS

## What is it?

`windows.mftscan.ADS` scans memory for **Alternate Data Streams (ADS)** stored within the NTFS Master File Table.

Alternate Data Streams allow additional hidden data to be attached to files without changing the visible file contents.

Example:

```text
document.txt

↓

document.txt:hidden.exe
```

The hidden stream is invisible during normal directory listings.

---

## Why do we need it?

Attackers frequently hide malware inside ADS because many users and security tools overlook alternate streams.

ADS may contain:

* Malware
* Scripts
* PowerShell payloads
* Configuration files
* Stolen data

---

## How it works

The plugin scans cached MFT records in memory and searches for NTFS Alternate Data Stream attributes.

It extracts metadata about each discovered stream.

---

## Command

```bash
python vol.py -f memory.raw windows.mftscan.ADS
```

---

## Artifacts Collected

* File Name
* ADS Name
* Parent Directory
* MFT Entry
* File Attributes
* Hidden Stream Information

---

## Investigation Use Cases

* Hidden malware detection
* ADS discovery
* NTFS artifact recovery
* Data hiding investigations

---

# 4. windows.mftscan.MFTScan

## What is it?

`windows.mftscan.MFTScan` scans memory for cached **Master File Table (MFT)** entries.

The Master File Table is the primary metadata database of the NTFS file system.

Every file and directory on an NTFS volume has an MFT record.

---

## Why do we need it?

Even if files have been deleted, fragments of their MFT records may remain in memory.

This allows investigators to recover valuable metadata.

---

## How it works

The plugin searches memory for NTFS MFT record signatures and reconstructs file records.

---

## Command

```bash
python vol.py -f memory.raw windows.mftscan.MFTScan
```

---

## Artifacts Collected

* File Name
* File Path
* MFT Entry Number
* Creation Time
* Modification Time
* Access Time
* File Size
* File Attributes
* Deleted File Metadata

---

## Investigation Use Cases

* Recover deleted file metadata
* Timeline analysis
* File system reconstruction
* NTFS investigations

---

# 5. windows.mftscan.ResidentData

## What is it?

`windows.mftscan.ResidentData` extracts **resident file data** stored directly inside NTFS MFT records.

Small files are often stored completely within their MFT entry instead of separate disk clusters.

These are called **resident files**.

---

## Why do we need it?

Attackers sometimes hide:

* Configuration files
* Scripts
* Small malware payloads
* Registry exports

inside resident MFT data.

Even if the file has been deleted, its resident data may still remain in memory.

---

## How it works

The plugin scans cached MFT entries and extracts resident data attributes stored within those records.

---

## Command

```bash
python vol.py -f memory.raw windows.mftscan.ResidentData
```

---

## Artifacts Collected

* Resident File Content
* File Name
* MFT Entry
* Resident Attributes
* Small File Data
* NTFS Metadata

---

## Investigation Use Cases

* Recover small deleted files
* Recover embedded scripts
* Recover malware configuration files
* File content reconstruction

---

# Recommended Investigation Workflow

When investigating storage-related artifacts, the following workflow is recommended:

1. Run `windows.mbrscan` to verify the integrity of the Master Boot Record.
2. Use `windows.pslist` or `windows.psscan` to identify suspicious processes.
3. Run `windows.memmap` against suspicious PIDs to inspect memory mappings.
4. Execute `windows.mftscan.MFTScan` to recover NTFS file metadata.
5. Run `windows.mftscan.ADS` to detect hidden Alternate Data Streams.
6. Use `windows.mftscan.ResidentData` to recover resident file contents.
7. Correlate recovered file artifacts with process activity and timeline analysis.

---

# Summary

The storage and memory mapping plugins in Volatility 3 provide investigators with powerful capabilities for analyzing both memory and NTFS file system artifacts.

| Plugin                         | Primary Purpose                                                   |
| ------------------------------ | ----------------------------------------------------------------- |
| `windows.mbrscan`              | Scan memory for Master Boot Record structures and detect bootkits |
| `windows.memmap`               | Display virtual-to-physical memory mappings for a process         |
| `windows.mftscan.ADS`          | Detect NTFS Alternate Data Streams hidden within files            |
| `windows.mftscan.MFTScan`      | Recover Master File Table entries and NTFS metadata               |
| `windows.mftscan.ResidentData` | Extract resident file contents stored directly inside MFT records |

Together, these plugins help investigators detect boot-level malware, analyze process memory, recover deleted file metadata, identify hidden Alternate Data Streams, and reconstruct small files stored within NTFS. They are essential tools for comprehensive Windows memory and file system forensic investigations.
