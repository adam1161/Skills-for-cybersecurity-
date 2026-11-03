

===== FILE: skills/performing-firmware-extraction-with-binwalk/SKILL.md =====
---
name: performing-firmware-extraction-with-binwalk
description: 'Performs firmware image extraction and analysis using binwalk to identify
  embedded filesystems, compressed archives, bootloaders, kernel images, and cryptographic
  material. Covers entropy analysis for detecting encrypted or compressed regions,
  recursive extraction of nested archives, SquashFS/CramFS/JFFS2 filesystem mounting,
  and string analysis for credential and configuration discovery. Activates for requests
  involving firmware reverse engineering, IoT device analysis, embedded system security
  assessment, or router/camera firmware extraction.

  '
domain: cybersecurity
subdomain: firmware-analysis
tags:
- firmware
- binwalk
- extraction
- entropy
- IoT-security
- reverse-engineering
version: 1.0.0
author: mukul975
license: Apache-2.0
nist_csf:
- ID.RA-01
- PR.PS-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1003
- T1110
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - initial-access
  techniques:
  - id: T1555
    name: Credentials from Password Stores
    tactic: reconnaissance
    source: attack
  - id: F1029
    name: Gather Customer Information
    tactic: reconnaissance
    source: f3
  - id: T1110.001
    name: 'Brute Force: Password Guessing'
    tactic: initial-access
    source: attack
  - id: F1006.001
    name: 'Account Takeover: Exposed API Key'
    tactic: initial-access
    source: f3
  - id: F1006.002
    name: 'Account Takeover: Exposed Login Credential'
    tactic: initial-access
    source: f3
---

# Performing Firmware Extraction with Binwalk

## When to Use

- Analyzing IoT device firmware downloaded from vendor sites or extracted from flash chips
- Reverse engineering router, camera, or embedded device firmware for vulnerability research
- Identifying embedded filesystems (SquashFS, CramFS, JFFS2, UBIFS) within firmware blobs
- Detecting encrypted or compressed regions using entropy analysis
- Extracting hardcoded credentials, API keys, certificates, or configuration files from firmware
- Performing security assessments of embedded devices in authorized penetration tests

**Do not use** for analyzing standard desktop application binaries or malware samples that are not firmware images; use dedicated malware analysis tools instead.

## Prerequisites

- binwalk v3.x installed (`pip install binwalk3` or from system package manager)
- Python 3.8+ with standard libraries (struct, math, hashlib, subprocess)
- SquashFS tools (`unsquashfs`) for mounting extracted SquashFS filesystems
- Jefferson for JFFS2 filesystem extraction (`pip install jefferson`)
- Sasquatch for non-standard SquashFS variants used by vendors like TP-Link and D-Link
- `strings` utility (GNU binutils) for string extraction
- Optional: firmware-mod-kit for repacking modified firmware images

## Workflow

### Step 1: Initial Firmware Reconnaissance

Perform a signature scan to identify embedded file types and their offsets:

```bash
# Basic signature scan - identify all recognized file types
binwalk firmware.bin

# Scan with verbose output showing confidence levels
binwalk -v firmware.bin

# Scan for specific file types only
binwalk -y "squashfs" firmware.bin
binwalk -y "gzip\|lzma\|xz" firmware.bin

# Opcode scan to identify CPU architecture
binwalk -A firmware.bin

# Scan for raw strings to find version info, URLs, credentials
binwalk -R "password" firmware.bin
binwalk -R "http://" firmware.bin
```

### Step 2: Entropy Analysis

Analyze entropy to identify encrypted, compressed, and plaintext regions:

```bash
# Generate entropy plot
binwalk -E firmware.bin

# Entropy with specific block size for higher resolution
binwalk -E -K 256 firmware.bin

# Combined entropy and signature scan
binwalk -BE firmware.bin
```

Interpreting entropy values:
- **0.0 - 1.0**: Empty or padding regions (null bytes, 0xFF fill)
- **1.0 - 5.0**: Plaintext data, code, ASCII strings, configuration
- **5.0 - 7.0**: Compressed data (gzip, LZMA, zlib)
- **7.0 - 7.99**: Strongly compressed or encrypted data
- **~8.0**: Maximum entropy, likely encrypted or random data

### Step 3: Extract Embedded Files

Extract all identified components from the firmware image:

```bash
# Automatic extraction of known file types
binwalk -e firmware.bin

# Recursive extraction (matryoshka mode) for nested archives
binwalk -Me firmware.bin

# Recursive extraction with depth limit
binwalk -Me -d 5 firmware.bin

# Extract specific file type with custom handler
binwalk -D "squashfs filesystem:squashfs:unsquashfs %e" firmware.bin

# Manual extraction of data at a known offset
dd if=firmware.bin of=extracted.squashfs bs=1 skip=327680 count=4194304
```

### Step 4: Mount and Inspect Extracted Filesystems

Mount extracted filesystems for deep inspection:

```bash
# Mount SquashFS filesystem
mkdir /tmp/squashfs_root
unsquashfs -d /tmp/squashfs_root extracted.squashfs

# Mount CramFS filesystem
mkdir /tmp/cramfs_root
mount -t cramfs -o loop extracted.cramfs /tmp/cramfs_root

# Extract JFFS2 filesystem
jefferson extracted.jffs2 -d /tmp/jffs2_root

# Inspect the extracted filesystem
ls -la /tmp/squashfs_root/
find /tmp/squashfs_root -name "*.conf" -o -name "*.cfg" -o -name "*.key"
find /tmp/squashfs_root -name "passwd" -o -name "shadow"
```

### Step 5: String Analysis and Credential Discovery

Search extracted filesystem and raw firmware for sensitive data:

```bash
# Extract all printable strings
strings -a firmware.bin > all_strings.txt
strings -n 12 firmware.bin | sort -u > long_strings.txt

# Search for credentials and secrets
grep -rni "password\|passwd\|secret\|api_key\|token" /tmp/squashfs_root/etc/
grep -rni "BEGIN.*PRIVATE KEY" /tmp/squashfs_root/

# Find hardcoded URLs and endpoints
grep -rnoE "https?://[a-zA-Z0-9./?=_-]+" /tmp/squashfs_root/

# Search for certificate files
find /tmp/squashfs_root -name "*.pem" -o -name "*.crt" -o -name "*.key" -o -name "*.p12"

# Identify busybox and service versions
strings /tmp/squashfs_root/bin/busybox | grep "BusyBox v"
cat /tmp/squashfs_root/etc/banner 2>/dev/null
```

### Step 6: Generate Firmware Analysis Report

Compile comprehensive extraction and analysis findings:

```
Report should include:
- Firmware metadata (vendor, model, version, build date)
- Identified components with offsets and sizes (bootloader, kernel, filesystem, config)
- Entropy analysis summary with regions of interest
- Extracted filesystem structure and key contents
- Discovered credentials, keys, certificates
- Identified services, daemons, and their versions
- Known CVEs applicable to identified component versions
- Recommendations for hardening or vulnerability remediation
```

## Key Concepts

| Term | Definition |
|------|------------|
| **Firmware** | Software embedded in hardware devices providing low-level control; typically contains a bootloader, kernel, root filesystem, and configuration data |
| **Entropy Analysis** | Statistical measurement of randomness in binary data; high entropy indicates encryption or compression, low entropy indicates plaintext or structured data |
| **SquashFS** | Read-only compressed filesystem commonly used in embedded Linux devices; supports LZMA, gzip, LZO, and zstd compression |
| **Magic Bytes** | Known byte sequences at fixed offsets that identify file types; binwalk uses a database of magic signatures to detect embedded files |
| **Matryoshka Extraction** | Recursive extraction mode where binwalk re-scans extracted files for additional embedded content, handling deeply nested archives |
| **CramFS** | Compressed ROM filesystem designed for embedded systems with limited flash storage; supports only zlib compression |
| **JFFS2** | Journalling Flash File System version 2, designed for NOR and NAND flash memory in embedded devices |

## Tools & Systems

- **binwalk**: Primary firmware analysis tool for signature scanning, entropy analysis, and automated extraction of embedded files
- **unsquashfs**: SquashFS extraction utility for mounting read-only compressed filesystems found in router and IoT firmware
- **jefferson**: Python tool for extracting JFFS2 flash filesystem images commonly found in embedded devices
- **sasquatch**: Patched SquashFS utility supporting non-standard vendor-modified SquashFS variants
- **firmware-mod-kit**: Toolkit for extracting, modifying, and repacking firmware images for security testing

## Common Scenarios

### Scenario: Extracting and Auditing Router Firmware for Hardcoded Credentials

**Context**: A security researcher is performing an authorized assessment of a consumer router. The firmware update file was downloaded from the vendor's support page. The goal is to identify hardcoded credentials, insecure default configurations, and known vulnerable components.

**Approach**:
1. Run `binwalk -e firmware.bin` to perform initial extraction
2. Use `binwalk -E firmware.bin` to check entropy and identify encrypted regions
3. Locate the SquashFS root filesystem in the extracted output
4. Mount with `unsquashfs` and inspect `/etc/passwd`, `/etc/shadow`, and web server configs
5. Search for hardcoded credentials with `grep -rni "password" /tmp/root/etc/`
6. Identify service versions and cross-reference with CVE databases
7. Check for debug interfaces (telnet, UART, JTAG references) in startup scripts
8. Examine web application code for authentication bypass or command injection

**Pitfalls**:
- Some vendors use non-standard SquashFS with custom compression; use sasquatch instead of unsquashfs
- Encrypted firmware requires decryption keys often found in bootloader or previous unencrypted versions
- Firmware headers may need to be stripped before binwalk can identify the embedded filesystem
- Obfuscated strings may evade simple grep searches; use entropy analysis to locate data blobs

## Output Format

```
FIRMWARE EXTRACTION REPORT
====================================
Firmware:         TP-Link TL-WR841N v14
File:             wr841nv14_en_3_16_9_up.bin
Size:             3,932,160 bytes (3.75 MB)
SHA-256:          a1b2c3d4e5f6...

SIGNATURE SCAN RESULTS
Offset       Type                          Size
------       ----                          ----
0x00000000   U-Boot bootloader header      64 bytes
0x00020000   LZMA compressed data          1,048,576 bytes
0x00120000   SquashFS filesystem v4.0      2,752,512 bytes
0x003B0000   Configuration partition       131,072 bytes

ENTROPY ANALYSIS
Region 0x000000-0x020000: 4.21 (bootloader - plaintext code)
Region 0x020000-0x120000: 7.89 (kernel - LZMA compressed)
Region 0x120000-0x3B0000: 7.45 (filesystem - SquashFS compressed)
Region 0x3B0000-0x3C0000: 1.12 (config - mostly empty)

EXTRACTED FILESYSTEM
Root filesystem: SquashFS v4.0, LZMA compression
Total files: 847
Total dirs: 112
BusyBox version: 1.19.4

SECURITY FINDINGS
[CRITICAL] Hardcoded root password in /etc/shadow (hash: $1$...)
[HIGH]     Telnet daemon enabled by default in /etc/init.d/rcS
[HIGH]     Private RSA key at /etc/ssl/private/server.key
[MEDIUM]   BusyBox 1.19.4 (CVE-2021-42373, CVE-2021-42374)
[MEDIUM]   Dropbear SSH 2014.63 (CVE-2016-3116)
[LOW]      UPnP service enabled by default
```

===== END FILE =====


===== FILE: skills/performing-firmware-extraction-with-binwalk/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/performing-firmware-extraction-with-binwalk/references/api-reference.md =====
# API Reference: Binwalk Firmware Extraction Tools

## binwalk - Firmware Analysis Tool

### Signature Scan
```bash
binwalk firmware.bin                      # Basic signature scan
binwalk -v firmware.bin                   # Verbose output
binwalk -B firmware.bin                   # Explicit signature scan flag
binwalk -A firmware.bin                   # Opcode/architecture scan
binwalk -R "string" firmware.bin          # Raw string search
```

### Extraction
```bash
binwalk -e firmware.bin                   # Extract known file types
binwalk -Me firmware.bin                  # Recursive (matryoshka) extraction
binwalk -Me -d 5 firmware.bin             # Recursive with depth limit
binwalk -C /output/dir -e firmware.bin    # Custom output directory
binwalk -D "type:ext:cmd" firmware.bin    # Custom extraction rule
```

### Entropy Analysis
```bash
binwalk -E firmware.bin                   # Entropy analysis with plot
binwalk -E -K 256 firmware.bin            # Custom block size
binwalk -BE firmware.bin                  # Combined signature + entropy
```

### Key Flags
| Flag | Description |
|------|-------------|
| `-B, --signature` | Scan for file signatures |
| `-e, --extract` | Extract identified file types |
| `-M, --matryoshka` | Recursive extraction |
| `-d, --depth=N` | Matryoshka recursion depth (default: 8) |
| `-E, --entropy` | Entropy analysis |
| `-K, --block=N` | Entropy block size in bytes |
| `-A, --opcodes` | Scan for CPU opcode signatures |
| `-R, --raw=STR` | Search for raw byte string |
| `-y, --include=STR` | Include only matching results |
| `-x, --exclude=STR` | Exclude matching results |
| `-m, --magic=FILE` | Use custom magic signature file |
| `-C, --directory=DIR` | Output directory for extraction |
| `-v, --verbose` | Verbose output |
| `--threads=N` | Number of worker threads |

## unsquashfs - SquashFS Extraction

### Syntax
```bash
unsquashfs -d /output/dir image.squashfs          # Extract to directory
unsquashfs -l image.squashfs                       # List contents
unsquashfs -ll image.squashfs                      # Long listing
unsquashfs -s image.squashfs                       # Show superblock info
unsquashfs -f -d /output image.squashfs            # Force overwrite
```

### Key Flags
| Flag | Description |
|------|-------------|
| `-d DIR` | Extract to specified directory |
| `-l` | List filesystem contents |
| `-ll` | Detailed listing with permissions |
| `-s` | Display superblock information |
| `-f` | Overwrite existing files |
| `-n` | No progress bar |
| `-e FILE` | Extract only specified files |

## jefferson - JFFS2 Extraction

### Syntax
```bash
jefferson image.jffs2 -d /output/dir              # Extract JFFS2
jefferson -v image.jffs2 -d /output/dir            # Verbose extraction
```

## sasquatch - Vendor SquashFS

### Syntax
```bash
sasquatch -d /output/dir image.squashfs            # Extract non-standard SquashFS
sasquatch -p 1 -d /output image.squashfs           # Single-threaded extraction
```

Handles vendor-modified SquashFS variants from TP-Link, D-Link, Netgear, and others that use non-standard compression or block sizes.

## strings - String Extraction

### Syntax
```bash
strings firmware.bin                               # Default (4+ chars)
strings -n 12 firmware.bin                         # Minimum 12 chars
strings -a firmware.bin                            # Scan entire file
strings -t x firmware.bin                          # Show hex offsets
strings -e l firmware.bin                          # Little-endian 16-bit
```

### Key Flags
| Flag | Description |
|------|-------------|
| `-n N` | Minimum string length |
| `-a` | Scan entire file (not just data sections) |
| `-t x` | Print offset in hexadecimal |
| `-t d` | Print offset in decimal |
| `-e l` | 16-bit little-endian encoding |
| `-e b` | 16-bit big-endian encoding |

## dd - Manual Extraction

### Syntax
```bash
dd if=firmware.bin of=output.bin bs=1 skip=OFFSET count=SIZE
dd if=firmware.bin of=output.bin bs=1 skip=$((0x120000)) count=$((0x2A0000))
```

### Key Parameters
| Parameter | Description |
|-----------|-------------|
| `if=FILE` | Input file |
| `of=FILE` | Output file |
| `bs=N` | Block size (use 1 for byte-precise extraction) |
| `skip=N` | Skip N blocks from input start |
| `count=N` | Copy only N blocks |

## Python binwalk Module (v2 API)

### Programmatic Usage
```python
import binwalk

# Signature scan
for module in binwalk.scan(firmware_path, signature=True, quiet=True):
    for result in module.results:
        print(f"0x{result.offset:08X}  {result.description}")

# Extract files
binwalk.scan(firmware_path, signature=True, extract=True, quiet=True)

# Entropy analysis
for module in binwalk.scan(firmware_path, entropy=True, quiet=True):
    for result in module.results:
        print(f"0x{result.offset:08X}  entropy={result.entropy}")

# Recursive extraction
binwalk.scan(firmware_path, signature=True, extract=True,
             matryoshka=True, depth=5, quiet=True)
```

===== END FILE =====


===== FILE: skills/performing-firmware-extraction-with-binwalk/scripts/agent.py =====
#!/usr/bin/env python3
"""Firmware extraction and analysis agent using binwalk for signature scanning,
entropy analysis, filesystem extraction, and string-based credential discovery."""

import argparse
import struct
import hashlib
import math
import os
import sys
import subprocess
import re
import json
from collections import Counter
from pathlib import Path

DISCLAIMER = """
==========================================================================
  AUTHORIZED USE ONLY -- This tool is intended for authorized security
  testing, firmware research, and educational purposes. Ensure you have
  explicit written permission before analyzing any firmware image you do
  not own. Unauthorized access to or reverse engineering of proprietary
  firmware may violate applicable laws and vendor agreements.
==========================================================================
"""


# ---------------------------------------------------------------------------
# Entropy Analysis
# ---------------------------------------------------------------------------

def calculate_entropy(data):
    """Calculate Shannon entropy of a byte sequence (0.0 = uniform, 8.0 = max random)."""
    if not data:
        return 0.0
    counter = Counter(data)
    length = len(data)
    entropy = -sum(
        (count / length) * math.log2(count / length)
        for count in counter.values()
    )
    return round(entropy, 4)


def entropy_map(file_path, block_size=4096):
    """Generate a block-by-block entropy map of a firmware image."""
    results = []
    with open(file_path, "rb") as f:
        offset = 0
        while True:
            block = f.read(block_size)
            if not block:
                break
            ent = calculate_entropy(block)
            results.append({
                "offset": offset,
                "offset_hex": f"0x{offset:08X}",
                "entropy": ent,
                "classification": classify_entropy(ent),
            })
            offset += len(block)
    return results


def classify_entropy(value):
    """Classify an entropy value into a human-readable category."""
    if value < 1.0:
        return "empty/padding"
    elif value < 5.0:
        return "plaintext/code"
    elif value < 7.0:
        return "compressed"
    elif value < 7.9:
        return "highly-compressed"
    else:
        return "encrypted/random"


def detect_entropy_regions(entropy_data, threshold_high=7.0, threshold_low=1.0):
    """Identify contiguous regions of high or low entropy in a firmware image."""
    regions = []
    current_region = None
    for entry in entropy_data:
        classification = entry["classification"]
        if current_region and current_region["classification"] == classification:
            current_region["end_offset"] = entry["offset"]
            current_region["block_count"] += 1
        else:
            if current_region:
                regions.append(current_region)
            current_region = {
                "start_offset": entry["offset"],
                "start_hex": entry["offset_hex"],
                "end_offset": entry["offset"],
                "classification": classification,
                "block_count": 1,
            }
    if current_region:
        regions.append(current_region)
    return regions


# ---------------------------------------------------------------------------
# Firmware Header Parsing
# ---------------------------------------------------------------------------

MAGIC_SIGNATURES = {
    b"\x27\x05\x19\x56": "U-Boot image header (uImage)",
    b"\x68\x73\x71\x73": "SquashFS filesystem (little-endian)",
    b"\x73\x71\x73\x68": "SquashFS filesystem (big-endian)",
    b"\x45\x3D\xCD\x28": "CramFS filesystem",
    b"\x85\x19\x01\x20": "JFFS2 filesystem (little-endian)",
    b"\x19\x85\x20\x01": "JFFS2 filesystem (big-endian)",
    b"\x1F\x8B\x08": "gzip compressed data",
    b"\x5D\x00\x00": "LZMA compressed data",
    b"\xFD\x37\x7A\x58\x5A\x00": "XZ compressed data",
    b"\x30\x37\x30\x37\x30\x31": "CPIO archive",
    b"\x55\xAA": "x86 boot sector",
    b"\xD0\x0D\xFE\xED": "Device Tree Blob (DTB)",
    b"\x4D\x5A": "PE/COFF executable (EFI binary)",
    b"\x7F\x45\x4C\x46": "ELF executable",
    b"\x89\x50\x4E\x47": "PNG image",
    b"\xFF\xD8\xFF": "JPEG image",
}


def scan_signatures(file_path, chunk_size=65536):
    """Scan a firmware image for known magic byte signatures."""
    matches = []
    file_size = os.path.getsize(file_path)
    with open(file_path, "rb") as f:
        offset = 0
        while offset < file_size:
            f.seek(offset)
            data = f.read(chunk_size)
            if not data:
                break
            for magic, description in MAGIC_SIGNATURES.items():
                pos = 0
                while True:
                    idx = data.find(magic, pos)
                    if idx == -1:
                        break
                    absolute_offset = offset + idx
                    matches.append({
                        "offset": absolute_offset,
                        "offset_hex": f"0x{absolute_offset:08X}",
                        "magic_hex": magic.hex().upper(),
                        "description": description,
                    })
                    pos = idx + 1
            offset += chunk_size - max(len(m) for m in MAGIC_SIGNATURES) + 1
    matches.sort(key=lambda x: x["offset"])
    return matches


def parse_uboot_header(file_path, offset=0):
    """Parse a U-Boot image header at the given offset."""
    with open(file_path, "rb") as f:
        f.seek(offset)
        header = f.read(64)
    if len(header) < 64:
        return None
    magic = struct.unpack(">I", header[0:4])[0]
    if magic != 0x27051956:
        return None
    header_crc = struct.unpack(">I", header[4:8])[0]
    timestamp = struct.unpack(">I", header[8:12])[0]
    data_size = struct.unpack(">I", header[12:16])[0]
    load_addr = struct.unpack(">I", header[16:20])[0]
    entry_point = struct.unpack(">I", header[20:24])[0]
    data_crc = struct.unpack(">I", header[24:28])[0]
    os_type = header[28]
    arch = header[29]
    image_type = header[30]
    comp_type = header[31]
    name = header[32:64].split(b"\x00")[0].decode("ascii", errors="replace")

    OS_TYPES = {0: "Invalid", 1: "OpenBSD", 2: "NetBSD", 3: "FreeBSD",
                4: "4_4BSD", 5: "Linux", 6: "SVR4", 7: "Esix", 8: "Solaris",
                9: "Irix", 10: "SCO", 11: "Dell", 12: "NCR", 14: "QNX",
                15: "U-Boot", 16: "RTEMS"}
    ARCH_TYPES = {0: "Invalid", 1: "Alpha", 2: "ARM", 3: "x86", 4: "IA64",
                  5: "MIPS", 6: "MIPS64", 7: "PowerPC", 8: "S390",
                  9: "SuperH", 10: "SPARC", 11: "SPARC64", 12: "M68K",
                  15: "AArch64", 22: "RISC-V"}
    COMP_TYPES = {0: "none", 1: "gzip", 2: "bzip2", 3: "lzma", 4: "lzo",
                  5: "lz4", 6: "zstd"}

    return {
        "magic": f"0x{magic:08X}",
        "header_crc": f"0x{header_crc:08X}",
        "data_size": data_size,
        "load_address": f"0x{load_addr:08X}",
        "entry_point": f"0x{entry_point:08X}",
        "data_crc": f"0x{data_crc:08X}",
        "os": OS_TYPES.get(os_type, f"Unknown({os_type})"),
        "architecture": ARCH_TYPES.get(arch, f"Unknown({arch})"),
        "compression": COMP_TYPES.get(comp_type, f"Unknown({comp_type})"),
        "name": name,
    }


# ---------------------------------------------------------------------------
# String Analysis
# ---------------------------------------------------------------------------

SENSITIVE_PATTERNS = [
    (re.compile(rb"password\s*[:=]\s*\S+", re.IGNORECASE), "Hardcoded password"),
    (re.compile(rb"passwd\s*[:=]\s*\S+", re.IGNORECASE), "Hardcoded password"),
    (re.compile(rb"api[_-]?key\s*[:=]\s*\S+", re.IGNORECASE), "API key"),
    (re.compile(rb"secret\s*[:=]\s*\S+", re.IGNORECASE), "Secret value"),
    (re.compile(rb"token\s*[:=]\s*\S+", re.IGNORECASE), "Authentication token"),
    (re.compile(rb"-----BEGIN\s+(RSA |DSA |EC )?PRIVATE KEY-----"), "Private key"),
    (re.compile(rb"-----BEGIN CERTIFICATE-----"), "X.509 certificate"),
    (re.compile(rb"https?://\S+"), "URL/endpoint"),
    (re.compile(rb"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b"), "IP address"),
    (re.compile(rb"root:\$[156]\$"), "Root password hash"),
    (re.compile(rb"telnetd|dropbear|sshd|httpd|uHTTPd"), "Network service"),
]


def scan_strings(file_path, min_length=8):
    """Extract printable ASCII strings and scan for sensitive patterns."""
    findings = []
    file_size = os.path.getsize(file_path)
    with open(file_path, "rb") as f:
        data = f.read()
    # Extract ASCII strings
    ascii_pattern = re.compile(rb"[\x20-\x7E]{%d,}" % min_length)
    strings_found = ascii_pattern.findall(data)
    # Scan each string against sensitive patterns
    for s in strings_found:
        for pattern, description in SENSITIVE_PATTERNS:
            if pattern.search(s):
                offset = data.find(s)
                findings.append({
                    "offset": f"0x{offset:08X}",
                    "type": description,
                    "value": s[:120].decode("ascii", errors="replace"),
                })
                break
    return findings


# ---------------------------------------------------------------------------
# Binwalk Subprocess Interface
# ---------------------------------------------------------------------------

def run_binwalk_scan(firmware_path):
    """Run binwalk signature scan via subprocess and return parsed output."""
    try:
        result = subprocess.run(
            ["binwalk", firmware_path],
            capture_output=True, text=True, timeout=120,
        )
        return {"stdout": result.stdout, "stderr": result.stderr, "rc": result.returncode}
    except FileNotFoundError:
        return {"stdout": "", "stderr": "binwalk not found in PATH", "rc": -1}
    except subprocess.TimeoutExpired:
        return {"stdout": "", "stderr": "binwalk scan timed out", "rc": -2}


def run_binwalk_extract(firmware_path, output_dir=None, recursive=False):
    """Run binwalk extraction via subprocess."""
    cmd = ["binwalk", "-e"]
    if recursive:
        cmd.append("-M")
    if output_dir:
        cmd.extend(["-C", output_dir])
    cmd.append(firmware_path)
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        return {"stdout": result.stdout, "stderr": result.stderr, "rc": result.returncode}
    except FileNotFoundError:
        return {"stdout": "", "stderr": "binwalk not found in PATH", "rc": -1}
    except subprocess.TimeoutExpired:
        return {"stdout": "", "stderr": "binwalk extraction timed out", "rc": -2}


def run_binwalk_entropy(firmware_path):
    """Run binwalk entropy analysis via subprocess."""
    try:
        result = subprocess.run(
            ["binwalk", "-E", firmware_path],
            capture_output=True, text=True, timeout=120,
        )
        return {"stdout": result.stdout, "stderr": result.stderr, "rc": result.returncode}
    except FileNotFoundError:
        return {"stdout": "", "stderr": "binwalk not found in PATH", "rc": -1}
    except subprocess.TimeoutExpired:
        return {"stdout": "", "stderr": "binwalk entropy analysis timed out", "rc": -2}


# ---------------------------------------------------------------------------
# Firmware Metadata
# ---------------------------------------------------------------------------

def get_firmware_metadata(file_path):
    """Compute basic metadata for a firmware image file."""
    file_size = os.path.getsize(file_path)
    sha256 = hashlib.sha256()
    md5 = hashlib.md5()
    with open(file_path, "rb") as f:
        while True:
            chunk = f.read(65536)
            if not chunk:
                break
            sha256.update(chunk)
            md5.update(chunk)
    return {
        "file": os.path.basename(file_path),
        "path": str(Path(file_path).resolve()),
        "size_bytes": file_size,
        "size_human": f"{file_size / (1024*1024):.2f} MB" if file_size > 1048576
                      else f"{file_size / 1024:.2f} KB",
        "sha256": sha256.hexdigest(),
        "md5": md5.hexdigest(),
    }


# ---------------------------------------------------------------------------
# Main Entry Point
# ---------------------------------------------------------------------------

def analyze_firmware(firmware_path):
    """Perform a complete firmware analysis pipeline."""
    print("=" * 65)
    print("  Firmware Extraction & Analysis Agent (binwalk)")
    print("=" * 65)

    if not os.path.isfile(firmware_path):
        print(f"[ERROR] File not found: {firmware_path}")
        return

    # Metadata
    meta = get_firmware_metadata(firmware_path)
    print(f"\n[*] File:    {meta['file']}")
    print(f"[*] Size:    {meta['size_human']} ({meta['size_bytes']} bytes)")
    print(f"[*] SHA-256: {meta['sha256']}")
    print(f"[*] MD5:     {meta['md5']}")

    # Signature scan
    print("\n--- Signature Scan ---")
    sigs = scan_signatures(firmware_path)
    if sigs:
        for s in sigs:
            print(f"  {s['offset_hex']}  {s['description']}  (magic: {s['magic_hex']})")
    else:
        print("  No known signatures detected.")

    # U-Boot header
    for s in sigs:
        if "U-Boot" in s["description"]:
            print(f"\n--- U-Boot Header at {s['offset_hex']} ---")
            hdr = parse_uboot_header(firmware_path, s["offset"])
            if hdr:
                for k, v in hdr.items():
                    print(f"  {k}: {v}")

    # Entropy analysis
    print("\n--- Entropy Analysis ---")
    emap = entropy_map(firmware_path, block_size=8192)
    regions = detect_entropy_regions(emap)
    for r in regions:
        size_bytes = (r["block_count"]) * 8192
        print(f"  {r['start_hex']} - 0x{r['end_offset']:08X}  "
              f"({size_bytes:>8} bytes)  [{r['classification']}]")

    # String analysis for sensitive data
    print("\n--- Sensitive String Analysis ---")
    findings = scan_strings(firmware_path, min_length=8)
    if findings:
        seen = set()
        for f in findings[:30]:
            key = (f["type"], f["value"][:60])
            if key not in seen:
                seen.add(key)
                print(f"  [{f['type']}] @ {f['offset']}: {f['value'][:80]}")
    else:
        print("  No sensitive strings detected.")

    # Binwalk subprocess scan
    print("\n--- Binwalk Scan Output ---")
    bw = run_binwalk_scan(firmware_path)
    if bw["rc"] == 0:
        print(bw["stdout"][:2000])
    elif bw["rc"] == -1:
        print("  [WARN] binwalk binary not found; install with: pip install binwalk3")
    else:
        print(f"  [ERROR] binwalk returned code {bw['rc']}: {bw['stderr'][:200]}")

    print("\n[*] Analysis complete.")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Firmware extraction and analysis agent using binwalk for "
                    "signature scanning, entropy analysis, filesystem extraction, "
                    "and string-based credential discovery.",
        epilog="Authorized use only. Ensure you have permission to analyze the target firmware.",
    )
    parser.add_argument(
        "firmware",
        help="Path to a firmware image file (.bin, .img, .rom)",
    )
    parser.add_argument(
        "--block-size", "-b",
        type=int, default=8192,
        help="Block size in bytes for entropy analysis (default: 8192)",
    )
    parser.add_argument(
        "--min-string-length", "-s",
        type=int, default=8,
        help="Minimum string length for sensitive string scanning (default: 8)",
    )
    parser.add_argument(
        "--extract", "-e",
        action="store_true",
        help="Run binwalk extraction after analysis",
    )
    parser.add_argument(
        "--recursive", "-M",
        action="store_true",
        help="Enable recursive (matryoshka) extraction",
    )
    parser.add_argument(
        "--output-dir", "-o",
        type=str, default=None,
        help="Output directory for extraction results",
    )
    parser.add_argument(
        "--json-output", "-j",
        action="store_true",
        help="Output results in JSON format instead of text",
    )

    args = parser.parse_args()
    print(DISCLAIMER)
    analyze_firmware(args.firmware)

    if args.extract:
        print("\n--- Running Binwalk Extraction ---")
        result = run_binwalk_extract(args.firmware, args.output_dir, args.recursive)
        if result["rc"] == 0:
            print(result["stdout"][:2000])
        else:
            print(f"  [ERROR] Extraction failed: {result.get('stderr', '')[:200]}")

===== END FILE =====
