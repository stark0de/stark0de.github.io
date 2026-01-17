+++
title = 'How to Accidentally Crash Windows Defender (and Explorer.exe) - Research Summary'
date = 2022-11-21
tags = ['windows-internals', 'windows-defender', 'vulnerability-research', 'protected-processes']
categories = ['research']
+++

## Note

**This article is a summary of research originally published in Spanish** at [Deep Hacking](https://blog.deephacking.tech/es/posts/como-cargarse-windows-defender-sin-querer/).

---

## Overview

This research explores an interesting discovery made while studying Windows process access rights and protected processes. The investigation led to accidentally suspending Windows Defender's main process (MsMpEng.exe) and causing explorer.exe to crash - all through legitimate use of Windows API calls.

---

## Background: Protected Processes in Windows

Windows implements a security mechanism called "Protected Processes" that prevents even administrator-level users from tampering with critical system processes. Many system processes, antivirus solutions, and EDRs use this protection level, including Windows Defender.

### Understanding Process Access Rights

When attempting to open a handle to a process using the `OpenProcess` Windows API function, different access rights can be requested. For protected processes, most access rights are denied by default, even with administrator privileges.

---

## The Discovery

While researching which access rights are permitted against protected processes, it was discovered that the following rights are **not explicitly prohibited**:

- `SYNCHRONIZE`
- `PROCESS_SUSPEND_RESUME`
- `PROCESS_QUERY_LIMITED_INFORMATION`

Among these, `PROCESS_SUSPEND_RESUME` stood out as particularly interesting for security research.

---

## The Experiment

### Suspending Windows Defender:

Using the `PROCESS_SUSPEND_RESUME` access right, it was possible to suspend the Windows Defender service process (MsMpEng.exe). The unexpected side effects were:

1. **Windows Defender remained suspended** - The process entered a suspended state
2. **Explorer.exe crashed** - The graphical environment hung, with only the cmd window remaining functional

### Verification with DTrace:

To verify the process suspension, DTrace (an open-source tool with a Windows version) was used to log system calls made by the process. After an initial period of no system calls (expected for a suspended process), MsMpEng.exe surprisingly began making system calls again, possibly due to driver-level interactions invisible to user-mode tools.

---

## Investigation into Explorer.exe Crash:

Initial hypothesis suggested there might be an open handle from MsMpEng to explorer.exe, but analysis of MsMpEng's handles showed only self-references. The exact cause of the explorer.exe crash remains unclear and is left open for further research.

---

## Technical Details:

### Key Windows Concepts:

- **Handles**: Identifiers required to access objects in Windows
- **Objects**: In Windows, everything is an object (similar to "everything is a file" in Linux)
- **EPROCESS**: The kernel object representing a process
- **Protected Processes**: Security mechanism preventing tampering with critical processes

### Access Rights Documentation:

Complete documentation on process security and access rights: [Microsoft Process Security Documentation](https://learn.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights)

---

## Impact and Conclusions:

This research demonstrates that even with Windows' protected process mechanisms, there are edge cases where legitimate API calls can be used to disrupt critical system services - albeit requiring administrator privileges.

**Key Takeaways:**
- Protected processes aren't entirely immune to interference
- Administrator privileges can still impact protected processes in specific ways
- The interaction between suspended protected processes and the Windows GUI warrants further investigation

---

## Related Research:

For more advanced techniques working with protected processes and anti-malware solutions, see:
- [Reversing Windows Internals - Handles, Callbacks & ObjectTypes](https://medium.com/@ashabdalhalim/a-light-on-windows-10s-object-header-typeindex-value-e8f907e7073a)
- [Backstab - A tool to kill antimalware protected processes](https://github.com/Yaxser/Backstab)

---

## Additional Resources:

The original article (in Spanish) contains more technical details, code snippets, and screenshots. The author plans to release a tool for experimentation in the future.

**Original Article**: [Cómo cargarse Windows Defender (sin querer)](https://blog.deephacking.tech/es/posts/como-cargarse-windows-defender-sin-querer/)

---

*This summary is provided in English for accessibility. For the complete technical implementation and Spanish-language documentation, please visit the original article.*