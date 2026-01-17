+++
title = 'Pwning rConfig: Multiple Vulnerabilities and RCE Chains'
date = 2020-08-27
tags = ['vulnerability-research', 'rce', 'web-security', 'cve']
categories = ['research']
+++

## Part I: Initial Vulnerability Discovery

I recently conducted a security assessment of rConfig version 3.9.5 ([www.rconfig.com](http://www.rconfig.com/)), a network monitoring tool, and discovered several critical vulnerabilities. This is the first part of a multi-part series detailing these findings.

---

### Local File Inclusion (LFI):

**Location**: `/lib/crud/configcompare.crud.php`

When comparing the `path_a` parameter to `path_b`, the file specified in `path_a` is disclosed if both files are completely different. The following example demonstrates reading `/etc/passwd`:

<img src="/research/rconfig/lfi-config-1.png" alt="LFI Configuration Compare" style="max-width: 800px;">
<br><br>
<img src="/research/rconfig/lfi-config-2.png" alt="LFI /etc/passwd disclosure">

---

### Arbitrary File Deletion:

**Location**: `/lib/ajaxHandlers/ajaxDeleteAllLoggingFiles.php`

This vulnerability allows deletion of any file by specifying:
- The file path using the `path` parameter
- The file extension using the `ext` parameter

![File deletion request](/research/rconfig/file-delete-1.png)

![File deletion confirmation](/research/rconfig/file-delete-2.png)

---

### Server-Side Request Forgery (SSRF):

**Location**: `/lib/ajaxHandlers/ajaxDeviceStatus.php`

An attacker can establish connections to internal services using:
- `deviceIpAddr` parameter for the target
- `connPort` parameter for the port

![SSRF request](/research/rconfig/ssrf-request.png)

**Open port response:**

![SSRF open port](/research/rconfig/ssrf-open-port.png)

---

### Cross-Site Scripting (XSS) Vulnerabilities:

#### XSS #1 - Device Management:

**Location**: `/devices.php` > Add device

1. Inject payload `<svg onload=alert(1)>` in the Model field
2. Fill remaining required fields
3. Click Save
4. Visit `/devicemgmt.php?deviceId=1&device=devicename`

<img src="/research/rconfig/xss-device.png" alt="XSS in device management" style="max-width: 800px;">

#### XSS #2 - Commands:

**Location**: `/commands.php` > Add command

1. Inject payload `<svg onload=alert(1)>` in the Command field
2. Click Save

![XSS in commands](/research/rconfig/xss-command.png)

#### XSS #3 - Snippets:

**Location**: `/snippets.php` > Add snippet

1. Inject payload `<svg onload=alert(1)>` in the Snippet field
2. Fill remaining required fields
3. Click Save

![XSS in snippets](/research/rconfig/xss-snippet.png)

---

### Privilege Escalation Vulnerabilities:

#### Method #1: Using sudo zip

![Privilege escalation via zip](/research/rconfig/privesc-zip.png)

#### Method #2: Using sudo crontab

Execute `sudo crontab -e` and then escape to shell with `:!/bin/bash`

#### Method #3: Arbitrary File Read

Leverage sudo permissions with tail following [GTFOBins documentation](https://gtfobins.github.io/gtfobins/tail/)

---

## Part II: Authentication Bypass and Remote Code Execution

Following the initial vulnerability disclosure, I discovered three authenticated Remote Code Execution (RCE) vulnerabilities along with two authentication bypass methods (one leveraging information disclosure).

**Important Note**: Many previously disclosed vulnerabilities (with assigned CVEs) remained present in version 3.9.5, providing additional exploitation vectors.

### Full Technical Details:

Complete technical analysis and proof-of-concept exploits are available in the SSD Advisory:

**[SSD Advisory - rConfig Unauthenticated RCE](https://ssd-disclosure.com/ssd-advisory-rconfig-unauthenticated-rce/)**

---

## Impact Summary:

The combination of these vulnerabilities allows for:
- Unauthenticated remote code execution
- Complete system compromise
- Arbitrary file operations
- Internal network reconnaissance via SSRF
- Privilege escalation to root

---

## Timeline:

- **Discovery**: August 2020
- **Public Disclosure**: August 27, 2020

---

*For questions or additional details about this research, please contact me via the channels listed on my [About](/about) page.*