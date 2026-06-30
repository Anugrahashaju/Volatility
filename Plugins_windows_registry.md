# Windows Registry Analysis Plugins in Volatility 3

# Introduction

The **Windows Registry** is one of the most valuable sources of forensic evidence during a memory investigation. It stores operating system configuration, user settings, installed software, security information, certificates, application execution history, and authentication data.

Even if a system is powered off or certain registry files are deleted from disk, many registry hives and keys remain cached in memory. Volatility 3 can extract this information directly from a RAM image using dedicated registry plugins.

This guide explains the following registry analysis plugins:

* `windows.registry.amacache`
* `windows.registry.cachedump`
* `windows.registry.certificates`
* `windows.registry.getcellroutine`
* `windows.registry.hashdump`

For each plugin, this guide covers:

* What the plugin does
* Why it is important
* How it works
* How to run it
* Artifacts collected
* Investigation use cases

---

# Table of Contents

1. windows.registry.amcache
2. windows.registry.cachedump
3. windows.registry.certificates
4. windows.registry.getcellroutine
5. windows.registry.hashdump
6. Recommended Investigation Workflow
7. Summary

---

# 1. windows.registry.amcache

## What is it?

`windows.registry.amcache` extracts information from the **Amcache** registry hive.

The **Amcache** database records metadata about applications and executable files that have been run or installed on the system. It is widely used in digital forensics to determine what programs were executed, even if the files have since been deleted.

---

## Why do we need it?

Attackers often delete malware after execution to remove evidence.

However, Amcache may still retain metadata about the executable, making it possible to identify previously executed programs.

Amcache can also help identify:

* Malware execution
* Portable applications
* Installed software
* USB-executed programs
* Deleted executables

---

## How it works

The plugin locates the Amcache registry hive in memory and parses registry keys related to application execution history.

---

## Command

```bash
python vol.py -f memory.raw windows.registry.amcache
```

---

## Artifacts Collected

* Executable Name
* Full File Path
* SHA-1 Hash (if available)
* File Size
* Compilation Timestamp
* Installation Information
* Program ID
* Last Modified Time

---

## Investigation Use Cases

* Malware execution history
* Application inventory
* Timeline reconstruction
* Deleted executable identification

---

# 2. windows.registry.cachedump

## What is it?

`windows.registry.cachedump` extracts **cached domain logon credentials** stored in the Windows SECURITY registry hive.

Windows caches domain credentials to allow users to log on even when a Domain Controller is unavailable.

---

## Why do we need it?

Cached credentials are valuable during forensic investigations because they indicate:

* Domain users who logged onto the system
* Cached authentication information
* Potential credential theft targets

Although the cached credentials are encrypted, they can be used in offline password-cracking attacks if obtained legally during an investigation.

---

## How it works

The plugin reads the SECURITY registry hive and extracts cached credential entries along with the required encryption metadata.

---

## Command

```bash
python vol.py -f memory.raw windows.registry.cachedump
```

---

## Artifacts Collected

* Username
* Domain Name
* Cached Credential Hash
* Cache Index
* Security Metadata

---

## Investigation Use Cases

* Domain user identification
* Credential theft investigations
* Lateral movement analysis
* Password audit support

---

# 3. windows.registry.certificates

## What is it?

`windows.registry.certificates` extracts digital certificates stored within the Windows Registry.

Windows stores certificates for users and the operating system in registry-based certificate stores.

---

## Why do we need it?

Certificates can be used for:

* Code signing
* Email encryption
* VPN authentication
* TLS/SSL authentication
* Smart card authentication

Attackers may install rogue or malicious certificates to bypass security controls or establish trusted communications.

---

## How it works

The plugin parses certificate store registry keys and extracts certificate metadata.

---

## Command

```bash
python vol.py -f memory.raw windows.registry.certificates
```

---

## Artifacts Collected

* Certificate Subject
* Issuer
* Serial Number
* Thumbprint
* Valid From Date
* Expiration Date
* Certificate Store
* Public Key Information

---

## Investigation Use Cases

* Rogue certificate detection
* Code-signing investigations
* TLS certificate analysis
* Trust store verification

---

# 4. windows.registry.getcellroutine

## What is it?

`windows.registry.getcellroutine` detects registry cell access routines that may have been modified by malware.

Registry cells are the fundamental storage units used within Windows registry hives.

---

## Why do we need it?

Kernel-mode malware and rootkits sometimes hook registry access routines to:

* Hide registry keys
* Hide malicious values
* Redirect registry reads
* Prevent security software from seeing registry modifications

This plugin helps identify such tampering.

---

## How it works

The plugin examines registry cell handling routines in memory and checks whether they have been altered or redirected from their expected locations.

---

## Command

```bash
python vol.py -f memory.raw windows.registry.getcellroutine
```

---

## Artifacts Collected

* Routine Address
* Hook Status
* Modified Function Pointer
* Registry Callback Information
* Suspicious Registry Hooks

---

## Investigation Use Cases

* Registry rootkit detection
* Registry hook analysis
* Hidden registry key investigations
* Kernel tampering detection

---

# 5. windows.registry.hashdump

## What is it?

`windows.registry.hashdump` extracts local Windows user password hashes from the **SAM (Security Account Manager)** registry hive.

The hashes are stored as **NTLM password hashes**, not as plaintext passwords.

---

## Why do we need it?

Password hashes help investigators:

* Identify local user accounts
* Verify account existence
* Perform authorized offline password auditing
* Detect compromised accounts

Because hashes represent authentication credentials, they should be handled securely and only used in accordance with legal and organizational policies.

---

## How it works

The plugin reads the SAM and SYSTEM registry hives, reconstructs the boot key used for encryption, and decrypts stored NTLM password hashes.

---

## Command

```bash
python vol.py -f memory.raw windows.registry.hashdump
```

---

## Artifacts Collected

* Username
* Relative Identifier (RID)
* NTLM Hash
* LM Hash (if present)
* User Account Information

---

## Investigation Use Cases

* Local account enumeration
* Credential auditing
* User account verification
* Incident response

---

# Recommended Investigation Workflow

When analyzing Windows registry artifacts, the following workflow is recommended:

1. Run `windows.registry.amcache` to identify executed applications.
2. Run `windows.registry.certificates` to inspect trusted certificates.
3. Execute `windows.registry.cachedump` to identify cached domain credentials.
4. Run `windows.registry.hashdump` to enumerate local user password hashes.
5. Use `windows.registry.getcellroutine` to detect registry hooks or kernel-level registry manipulation.
6. Correlate findings with process, network, and file system artifacts from other Volatility plugins.

---

# Summary

The Windows Registry plugins in Volatility 3 provide investigators with access to valuable configuration, authentication, and application execution data stored within memory.

| Plugin                            | Primary Purpose                                                               |
| --------------------------------- | ----------------------------------------------------------------------------- |
| `windows.registry.amcache`        | Recover application execution and installation metadata from the Amcache hive |
| `windows.registry.cachedump`      | Extract cached domain logon credentials from the SECURITY hive                |
| `windows.registry.certificates`   | Recover digital certificates stored in Windows registry certificate stores    |
| `windows.registry.getcellroutine` | Detect registry cell routine hooks and registry manipulation by malware       |
| `windows.registry.hashdump`       | Extract local Windows NTLM password hashes from the SAM registry hive         |

Together, these plugins enable investigators to reconstruct application execution history, identify authentication artifacts, verify trusted certificates, detect registry tampering, and recover account information. They are essential tools for Windows memory forensic investigations, incident response, malware analysis, and digital evidence collection.
