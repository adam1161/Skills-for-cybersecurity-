

===== FILE: skills/analyzing-uefi-bootkit-persistence/SKILL.md =====
---
name: analyzing-uefi-bootkit-persistence
description: 'Analyzes UEFI bootkit persistence (SPI flash implants, ESP modifications,
  Secure Boot bypass, UEFI variable manipulation) using chipsec for firmware integrity
  verification, detecting known families like BlackLotus, LoJax, and MoonBounce.
  Use for UEFI malware analysis, firmware persistence investigation, or Secure Boot
  bypass detection.

  '
domain: cybersecurity
subdomain: firmware-security
tags:
- UEFI
- bootkit
- firmware
- Secure-Boot
- chipsec
- ESP
- persistence
version: 1.0.0
author: mukul975
license: Apache-2.0
d3fend_techniques:
- Platform Hardening
- Restore Object
- Platform Monitoring
- Firmware Verification
- Firmware Embedded Monitoring Code
nist_csf:
- ID.RA-01
- PR.PS-01
- PR.PS-02
mitre_attack:
- T1542.001
- T1542.003
- T1553.006
- T1542
- T1014
---

# Analyzing UEFI Bootkit Persistence

## When to Use

- A compromised system re-establishes C2 communication after OS reinstallation or disk replacement
- Secure Boot has been tampered with, disabled, or shows unexpected Machine Owner Key (MOK) enrollment
- Firmware integrity verification fails against vendor-provided baselines
- Memory forensics reveals rootkit components loading during early boot phase
- Investigating advanced persistent threat (APT) campaigns known to deploy UEFI implants
- Auditing firmware security posture for enterprise endpoint hardening

**Do not use** for standard MBR-based bootkits on legacy BIOS systems without UEFI; use MBR/VBR bootkit analysis instead.

## Prerequisites

- chipsec framework for SPI flash dumping, UEFI variable inspection, and firmware security modules
- UEFITool / UEFIExtract for firmware volume parsing and DXE driver extraction
- Python 3.8+ with struct, hashlib, subprocess, and os modules
- Bootable Linux live USB for offline analysis (avoid running compromised OS)
- Volatility 3 for memory forensics of boot-phase artifacts
- YARA with UEFI malware rule sets for pattern-based detection
- Access to vendor firmware baselines for integrity comparison

## Workflow

### Step 1: Dump SPI Flash Firmware

Acquire the UEFI firmware from the SPI flash chip for offline analysis:

```bash
# Using chipsec to dump SPI flash contents
python chipsec_util.py spi dump firmware_dump.rom

# Using flashrom as an alternative
flashrom -p internal -r firmware_dump.rom

# Verify dump integrity
sha256sum firmware_dump.rom

# Read SPI flash descriptor information
python chipsec_util.py spi info

# Check SPI flash region access permissions
python chipsec_main.py -m common.spi_access

# Verify BIOS write protection is enabled
python chipsec_main.py -m common.bios_wp

# Check SPI flash controller lock
python chipsec_main.py -m common.spi_lock
```

### Step 2: Inspect UEFI Variables

Enumerate and analyze UEFI variables for unauthorized modifications:

```bash
# List all UEFI variables on a live system
python chipsec_util.py uefi var-list

# List UEFI variables from a SPI flash dump
python chipsec_util.py uefi var-list-spi firmware_dump.rom

# Read specific Secure Boot variables
python chipsec_util.py uefi var-read SecureBoot 8BE4DF61-93CA-11D2-AA0D-00E098032B8C
python chipsec_util.py uefi var-read SetupMode 8BE4DF61-93CA-11D2-AA0D-00E098032B8C
python chipsec_util.py uefi var-read PK 8BE4DF61-93CA-11D2-AA0D-00E098032B8C
python chipsec_util.py uefi var-read KEK 8BE4DF61-93CA-11D2-AA0D-00E098032B8C
python chipsec_util.py uefi var-read db D719B2CB-3D3A-4596-A3BC-DAD00E67656F

# Dump UEFI key databases for analysis
python chipsec_util.py uefi keys

# Check Secure Boot configuration module
python chipsec_main.py -m common.secureboot.variables
```

### Step 3: Analyze EFI System Partition (ESP)

Inspect the ESP for unauthorized or modified boot components:

```bash
# Mount ESP (typically the first FAT32 partition, ~100-500MB)
mkdir /mnt/esp
mount /dev/sda1 /mnt/esp

# List all files on ESP with timestamps
find /mnt/esp -type f -exec ls -la {} \;

# Check for BlackLotus indicators - custom directory under ESP:/system32/
ls -la /mnt/esp/system32/ 2>/dev/null

# Verify Windows Boot Manager signature
sigcheck -a /mnt/esp/EFI/Microsoft/Boot/bootmgfw.efi

# Hash all EFI binaries for comparison against known-good values
find /mnt/esp -name "*.efi" -exec sha256sum {} \;

# Check for unauthorized .efi files outside standard directories
find /mnt/esp -name "*.efi" | grep -v "Microsoft\|Boot\|ubuntu\|grub"

# Look for grubx64.efi planted by BlackLotus
find /mnt/esp -name "grubx64.efi" -exec sha256sum {} \;

# Examine MeasuredBoot logs for anomalies (Windows)
# Logs located at C:\Windows\Logs\MeasuredBoot\
```

### Step 4: Scan Firmware for Known Bootkit Signatures

Analyze the firmware dump for known UEFI malware patterns:

```bash
# Extract all firmware modules with UEFIExtract
UEFIExtract firmware_dump.rom all

# Generate firmware module whitelist from vendor baseline
python chipsec_main.py -m tools.uefi.whitelist -a generate,baseline.json,firmware_vendor.rom

# Compare current firmware against whitelist
python chipsec_main.py -m tools.uefi.whitelist -a check,baseline.json,firmware_dump.rom

# Scan firmware with UEFI-specific YARA rules
yara -r uefi_bootkits.yar firmware_dump.rom

# Scan extracted modules individually
find firmware_dump.rom.dump -name "*.efi" -exec yara -r uefi_bootkits.yar {} \;

# Check for modified CORE_DXE module (targeted by MoonBounce, CosmicStrand)
# Compare GUID and hash against vendor baseline
```

### Step 5: Detect Secure Boot Bypass Mechanisms

Check for known Secure Boot bypass techniques:

```bash
# Check if Secure Boot is enabled
python chipsec_main.py -m common.secureboot.variables

# Verify SMM (System Management Mode) protections
python chipsec_main.py -m common.smm

# Check SMM BIOS write protection
python chipsec_main.py -m common.bios_smi

# On Windows - check boot configuration for bypass indicators
bcdedit /enum firmware
bcdedit /v

# Check for testsigning/nointegritychecks/debug flags
bcdedit | findstr /i "testsigning nointegritychecks debug"

# Verify HVCI (Hypervisor-enforced Code Integrity) is not disabled
# BlackLotus sets HKLM:\...\DeviceGuard\...\HypervisorEnforcedCodeIntegrity Enabled=0
reg query "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\HypervisorEnforcedCodeIntegrity" /v Enabled

# Check Secure Boot state via PowerShell
# Confirm-SecureBootUEFI returns True if properly enabled
```

### Step 6: Perform Boot Chain Integrity Verification

Verify every component in the boot chain from firmware through kernel:

```bash
# Verify firmware integrity against vendor hash
sha256sum firmware_dump.rom
# Compare with vendor-published hash

# Verify bootloader signatures
sigcheck -a C:\Windows\Boot\EFI\bootmgfw.efi
sigcheck -a C:\Windows\System32\winload.efi
sigcheck -a C:\Windows\System32\ntoskrnl.exe

# Check for unsigned or invalid boot drivers
sigcheck -u -e C:\Windows\System32\drivers\

# Analyze Measured Boot logs for unexpected EFI_Boot_Services_Application entries
# BlackLotus components appear as EV_EFI_Boot_Services_Application

# Memory forensics for boot-phase artifacts
vol3 -f memory.dmp windows.modules
vol3 -f memory.dmp windows.driverscan
```

### Step 7: Document UEFI Bootkit Analysis Findings

Compile a comprehensive analysis report:

```
Report should include:
- Firmware version, vendor, and platform identification
- SPI flash protection status (write protect, lock bits, access control)
- Secure Boot configuration and any bypass indicators detected
- UEFI variable anomalies (unauthorized keys, modified db/dbx, MOK enrollment)
- ESP contents inventory with hash verification against known-good baselines
- Firmware module comparison against vendor whitelist (added, modified, removed)
- Known bootkit family attribution with confidence level
- Boot chain integrity verification results for each component
- Remediation steps (reflash, key rotation, hardware replacement)
- MITRE ATT&CK mapping (T1542.001 - System Firmware, T1542.003 - Bootkit)
```

## Key Concepts

| Term | Definition |
|------|------------|
| **UEFI Bootkit** | Malware that persists in UEFI firmware or the boot process, executing before the operating system loads and surviving OS reinstallation |
| **SPI Flash** | Serial Peripheral Interface flash memory chip on the motherboard storing UEFI firmware; firmware-level bootkits like LoJax and MoonBounce modify SPI flash contents |
| **EFI System Partition (ESP)** | FAT32 partition containing EFI bootloaders and drivers; bootkits like BlackLotus and ESPecter modify files on the ESP for persistence |
| **Secure Boot** | UEFI security feature that verifies digital signatures of boot components; can be bypassed via vulnerabilities (CVE-2022-21894) or MOK enrollment |
| **DXE Driver** | Driver Execution Environment driver loaded during UEFI boot; firmware implants inject malicious DXE drivers that execute before the OS |
| **Machine Owner Key (MOK)** | User-installable Secure Boot key; BlackLotus enrolls attacker-controlled MOKs to sign malicious bootloaders |
| **chipsec** | Intel platform security assessment framework for analyzing SPI flash, UEFI variables, Secure Boot, and hardware security configurations |
| **HVCI** | Hypervisor-enforced Code Integrity, a Windows security feature that bootkits disable to load unsigned kernel drivers |

## Tools & Systems

- **chipsec**: Intel framework for dumping SPI flash, reading UEFI variables, verifying firmware write protection, and Secure Boot configuration auditing
- **UEFITool**: Open-source UEFI firmware image parser for inspecting firmware volumes, extracting DXE drivers, and comparing module GUIDs
- **sigcheck**: Sysinternals utility for verifying digital signatures of EFI binaries and boot chain components
- **flashrom**: Open-source SPI flash programmer for reading and writing firmware chips on supported platforms
- **YARA**: Pattern matching engine used with UEFI-specific rule sets to detect known bootkit signatures in firmware dumps

## Common Scenarios

### Scenario: Investigating Persistent Compromise Surviving OS Reinstallation

**Context**: An enterprise endpoint was reimaged after a confirmed breach, but identical C2 beaconing resumed within hours. The endpoint has UEFI firmware with Secure Boot enabled, and a TPM 2.0 chip. The security team suspects a UEFI-level implant similar to BlackLotus or LoJax.

**Approach**:
1. Boot the system from a trusted Linux live USB to avoid executing any compromised OS components
2. Dump SPI flash firmware using `chipsec_util.py spi dump` for offline analysis
3. Mount the ESP and hash all `.efi` files for comparison against known-good values from identical hardware
4. Check for the `ESP:/system32/` directory (BlackLotus indicator) and unauthorized `grubx64.efi`
5. Extract firmware modules with UEFIExtract and compare GUID inventory against vendor baseline
6. Verify Secure Boot variables -- look for unauthorized MOK enrollment or modified db/dbx
7. Check SPI flash write protection and lock bits using chipsec modules
8. Scan firmware dump and extracted modules with UEFI-specific YARA rules
9. If BlackLotus is suspected, check registry for HVCI disabled and MeasuredBoot logs for anomalous entries

**Pitfalls**:
- Running analysis from the compromised OS (rootkit components hide from live analysis)
- Only checking the ESP without examining SPI flash firmware (misses firmware-level implants like LoJax, MoonBounce)
- Assuming Secure Boot prevents all bootkits (CVE-2022-21894 and other bypasses exist)
- Not preserving the original firmware dump before remediation (critical forensic evidence)
- Reflashing firmware without verifying the vendor image is authentic and unmodified

## Output Format

```
UEFI BOOTKIT PERSISTENCE ANALYSIS REPORT
============================================
System:           Lenovo ThinkPad X1 Carbon Gen 11
Firmware:         N3HET82W (1.54) - Lenovo UEFI BIOS
Platform:         Intel 13th Gen (Raptor Lake)
TPM:              2.0 (Infineon SLB 9672)
Secure Boot:      ENABLED (BYPASSED via CVE-2022-21894)
Analysis Method:  Linux live USB + chipsec + UEFITool

SPI FLASH PROTECTION STATUS
BIOS Write Protection:    DISABLED [!]
SPI Flash Lock (FLOCKDN): SET [OK]
SMM BIOS Write Protect:   DISABLED [!]
SPI Protected Ranges:     Region 0 only (descriptor)

UEFI VARIABLE ANALYSIS
SecureBoot:        Enabled (value=1)
SetupMode:         Disabled (value=0)
PK:                Lenovo Ltd. (legitimate)
KEK:               Microsoft + Lenovo (legitimate)
db:                MODIFIED - contains unauthorized entry [!]
  [!] Unknown certificate: CN=Secure Boot Signing, O=Unknown
  [!] Not present in vendor baseline db
MOK:               1 unauthorized key enrolled [!]
  [!] MOK enrolled: CN=shim, self-signed, not from distro vendor

ESP PARTITION ANALYSIS
Total EFI binaries:     12
Verified (signed):      9
Modified (hash mismatch): 2 [!]
Unauthorized:           1 [!]

  [!] EFI/Microsoft/Boot/bootmgfw.efi - MODIFIED
      Expected SHA-256: a3f2c8...
      Current SHA-256:  7b1e4d...
      Signature:        Valid (signed with unauthorized MOK)

  [!] EFI/Microsoft/Boot/grubx64.efi - UNAUTHORIZED
      SHA-256:  e9c1a7...
      Not present in vendor baseline
      Matches BlackLotus stage-2 loader signature

  [!] system32/ directory present on ESP (BlackLotus artifact)
      Directory empty (files deleted post-installation)

FIRMWARE MODULE ANALYSIS
Total firmware modules:   312
Vendor baseline modules:  312
Added modules:            0
Modified modules:         0
SPI flash integrity:      CLEAN (no firmware-level implant detected)

BOOTKIT ATTRIBUTION
Family:           BlackLotus
Confidence:       HIGH
Persistence:      ESP-based (not SPI flash)
Bypass Method:    CVE-2022-21894 (baton drop)
MITRE ATT&CK:    T1542.003 (Bootkit), T1553.006 (Code Signing Policy Modification)

INDICATORS OF COMPROMISE
- ESP:/system32/ directory (empty, post-cleanup artifact)
- ESP:/EFI/Microsoft/Boot/grubx64.efi (unauthorized, BlackLotus loader)
- Modified bootmgfw.efi (re-signed with attacker MOK)
- HVCI disabled via registry: DeviceGuard\...\Enabled = 0
- Unauthorized MOK enrollment in UEFI variable store
- MeasuredBoot log shows EV_EFI_Boot_Services_Application for grubx64.efi

REMEDIATION
1. Replace bootmgfw.efi with authentic copy from Windows installation media
2. Delete unauthorized grubx64.efi and system32/ directory from ESP
3. Reset Secure Boot keys to factory defaults (clear MOK, restore PK/KEK/db)
4. Enable BIOS write protection and verify SPI flash lock bits
5. Apply firmware update to latest version (patches CVE-2022-21894)
6. Enable HVCI and verify via Group Policy
7. Reimport only trusted certificates into Secure Boot db
8. Monitor MeasuredBoot logs for anomalous boot component loading
```

===== END FILE =====


===== FILE: skills/analyzing-uefi-bootkit-persistence/LICENSE =====

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


===== FILE: skills/analyzing-uefi-bootkit-persistence/references/api-reference.md =====
# API Reference: UEFI Bootkit Analysis Tools

## chipsec - Platform Security Assessment Framework

### SPI Flash Operations
```bash
python chipsec_util.py spi info                          # SPI flash info
python chipsec_util.py spi dump firmware.rom             # Dump entire SPI flash
python chipsec_util.py spi read 0x700000 0x100000 bios.bin  # Read specific region
python chipsec_util.py spi write 0x0 0x1000 data.bin     # Write to SPI flash
```

### UEFI Variable Operations
```bash
python chipsec_util.py uefi var-list                     # List all UEFI variables
python chipsec_util.py uefi var-list-spi firmware.rom    # List vars from dump
python chipsec_util.py uefi var-read <name> <GUID>       # Read specific variable
python chipsec_util.py uefi var-find <name>              # Find variable by name
python chipsec_util.py uefi keys                         # Dump Secure Boot keys
python chipsec_util.py uefi tables                       # List UEFI tables
python chipsec_util.py uefi decode firmware.rom          # Decode firmware image
```

### Security Assessment Modules
```bash
python chipsec_main.py -m <module>                       # Run security module
python chipsec_main.py -m common.secureboot.variables    # Secure Boot check
python chipsec_main.py -m common.bios_wp                 # BIOS write protection
python chipsec_main.py -m common.spi_lock                # SPI flash lock bits
python chipsec_main.py -m common.spi_access              # SPI region permissions
python chipsec_main.py -m common.spi_desc                # SPI descriptor check
python chipsec_main.py -m common.smm                     # SMM protection
python chipsec_main.py -m common.bios_smi                # SMI suppression
```

### Firmware Whitelist Module
```bash
# Generate whitelist from known-good firmware
python chipsec_main.py -m tools.uefi.whitelist -a generate,baseline.json,vendor.rom

# Check firmware against whitelist
python chipsec_main.py -m tools.uefi.whitelist -a check,baseline.json,suspect.rom
```

### Key Modules Reference
| Module | Purpose |
|--------|---------|
| `common.secureboot.variables` | Verify Secure Boot PK, KEK, db, dbx variables |
| `common.bios_wp` | Check BIOS region write protection (BIOSWE, BLE, SMM_BWP) |
| `common.spi_lock` | Verify SPI flash controller lock (FLOCKDN) |
| `common.spi_access` | Check SPI flash region read/write permissions |
| `common.spi_desc` | Verify SPI flash descriptor is write-protected |
| `common.smm` | Verify SMRAM range register protection (SMRR) |
| `common.bios_smi` | Check SMI event configuration and suppression |
| `tools.uefi.whitelist` | Generate and verify firmware module whitelists |
| `tools.uefi.scan_image` | Scan firmware image for known vulnerabilities |
| `tools.uefi.uefivar_fuzz` | Fuzz UEFI variable interface for vulnerabilities |

## UEFITool / UEFIExtract

### UEFIExtract CLI
```bash
UEFIExtract firmware.rom all                             # Extract all modules
UEFIExtract firmware.rom <GUID> body                     # Extract specific module
UEFIExtract firmware.rom report                          # Generate report
```

### Output Structure
Extracted firmware is organized by GUID into a directory tree containing:
- PEI modules (Pre-EFI Initialization)
- DXE drivers (Driver Execution Environment)
- SMM drivers (System Management Mode)
- Option ROMs
- NVRAM variables

## Secure Boot Variable GUIDs

| Variable | GUID | Description |
|----------|------|-------------|
| `SecureBoot` | `8BE4DF61-93CA-11D2-AA0D-00E098032B8C` | Secure Boot enable status |
| `SetupMode` | `8BE4DF61-93CA-11D2-AA0D-00E098032B8C` | Setup mode (keys not enrolled) |
| `PK` | `8BE4DF61-93CA-11D2-AA0D-00E098032B8C` | Platform Key (root of trust) |
| `KEK` | `8BE4DF61-93CA-11D2-AA0D-00E098032B8C` | Key Exchange Key |
| `db` | `D719B2CB-3D3A-4596-A3BC-DAD00E67656F` | Signature database (allowed) |
| `dbx` | `D719B2CB-3D3A-4596-A3BC-DAD00E67656F` | Forbidden signature database |
| `MokList` | `605DAB50-E046-4300-ABB6-3DD810DD8B23` | Machine Owner Key list |

## flashrom - SPI Flash Programmer

### Syntax
```bash
flashrom -p internal -r firmware.rom                     # Read/dump flash
flashrom -p internal -w clean.rom                        # Write/reflash
flashrom -p internal --verify clean.rom                  # Verify contents
flashrom -p internal --flash-size                        # Show flash size
flashrom -L                                              # List supported chips
```

## sigcheck - Signature Verification (Windows)

### Syntax
```bash
sigcheck -a file.efi                                     # Full signature info
sigcheck -u -e C:\Windows\System32\drivers\              # Find unsigned drivers
sigcheck -c -h file.efi                                  # CSV output with hashes
```

## bcdedit - Boot Configuration (Windows)

### Syntax
```bash
bcdedit /enum firmware                                   # List firmware entries
bcdedit /v                                               # Verbose boot config
bcdedit | findstr /i "testsigning nointegritychecks"      # Check bypass flags
```

## YARA - Firmware Pattern Scanning

### UEFI Bootkit Rules
```bash
yara -r uefi_bootkits.yar firmware.rom                   # Scan firmware dump
yara -s -r rules.yar firmware.rom                        # Show matching strings
```

### Example UEFI Detection Rule
```yara
rule BlackLotus_ESP_Indicator {
    meta:
        description = "Detects BlackLotus ESP-based bootkit artifacts"
        reference = "ESET Research 2023"
    strings:
        $mok_enroll = { 4D 00 6F 00 6B 00 4C 00 69 00 73 00 74 }
        $esp_path = "\\EFI\\Microsoft\\Boot\\grubx64.efi"
        $hvci_disable = "HypervisorEnforcedCodeIntegrity"
    condition:
        any of them
}
```

===== END FILE =====


===== FILE: skills/analyzing-uefi-bootkit-persistence/scripts/agent.py =====
#!/usr/bin/env python3
"""UEFI bootkit persistence analysis agent for detecting firmware implants,
ESP modifications, Secure Boot bypasses, and UEFI variable manipulation."""

import argparse
import struct
import hashlib
import os
import sys
import subprocess
import re
import math
import json
from collections import Counter
from pathlib import Path

DISCLAIMER = """
==========================================================================
  AUTHORIZED USE ONLY -- This tool is intended for authorized firmware
  security assessments, incident response, and defensive security research.
  Analyzing UEFI firmware and boot components requires appropriate system
  access and authorization. Unauthorized firmware modification or Secure
  Boot key manipulation may render systems unbootable or violate policy.
==========================================================================
"""


# ---------------------------------------------------------------------------
# Known Bootkit Signatures and IOCs
# ---------------------------------------------------------------------------

KNOWN_BOOTKITS = {
    "BlackLotus": {
        "description": "First in-the-wild UEFI bootkit bypassing Secure Boot on fully patched Windows 11",
        "cve": "CVE-2022-21894",
        "persistence": "ESP-based (modifies bootmgfw.efi, enrolls attacker MOK)",
        "esp_indicators": ["system32/", "grubx64.efi"],
        "registry_indicators": {
            r"SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\HypervisorEnforcedCodeIntegrity": {
                "Enabled": 0
            }
        },
        "mitre": "T1542.003",
    },
    "LoJax": {
        "description": "First SPI flash firmware implant found in the wild (APT28/Fancy Bear)",
        "cve": None,
        "persistence": "SPI flash (injects DXE driver into firmware volume)",
        "firmware_indicators": ["rpcnetp.exe", "autoche.exe"],
        "dxe_modifications": True,
        "mitre": "T1542.001",
    },
    "MoonBounce": {
        "description": "SPI flash implant modifying CORE_DXE module to hook GetVariable()",
        "cve": None,
        "persistence": "SPI flash (modifies CORE_DXE firmware module)",
        "firmware_indicators": ["CORE_DXE modification", "GetVariable hook"],
        "dxe_modifications": True,
        "mitre": "T1542.001",
    },
    "CosmicStrand": {
        "description": "Firmware rootkit modifying CORE_DXE to hook kernel initialization",
        "cve": None,
        "persistence": "SPI flash (patches CORE_DXE)",
        "firmware_indicators": ["CORE_DXE modification", "kernel callback shellcode"],
        "dxe_modifications": True,
        "mitre": "T1542.001",
    },
    "ESPecter": {
        "description": "ESP-based bootkit that patches winload.efi to disable DSE",
        "cve": None,
        "persistence": "ESP-based (modifies Windows Boot Manager)",
        "esp_indicators": ["modified winload.efi", "unsigned kernel driver"],
        "mitre": "T1542.003",
    },
    "MosaicRegressor": {
        "description": "Multi-component UEFI implant using NTFS file drops via READY_TO_BOOT callbacks",
        "cve": None,
        "persistence": "SPI flash (READY_TO_BOOT callback for NTFS drops)",
        "firmware_indicators": ["fTA variable", "READY_TO_BOOT callback"],
        "dxe_modifications": True,
        "mitre": "T1542.001",
    },
    "Bootkitty": {
        "description": "First UEFI bootkit targeting Linux systems",
        "cve": None,
        "persistence": "ESP-based (modifies GRUB bootloader)",
        "esp_indicators": ["modified grubx64.efi"],
        "mitre": "T1542.003",
    },
}

# UEFI Secure Boot variable GUIDs
SECUREBOOT_GUID = "8BE4DF61-93CA-11D2-AA0D-00E098032B8C"
IMAGE_SECURITY_GUID = "D719B2CB-3D3A-4596-A3BC-DAD00E67656F"

# Standard UEFI firmware volume GUIDs
KNOWN_FV_GUIDS = {
    "8C8CE578-8A3D-4F1C-9935-896185C32DD3": "Firmware File System (FFS) v2",
    "5473C07A-3DCB-4DCA-BD6F-1E9689E7349A": "Firmware File System (FFS) v3",
    "04ADEEAD-61FF-4D31-B6BA-64F8BF901F5A": "Apple ROM section",
    "16B45DA2-7D70-4AEA-A58D-760E9ECB841D": "DXE Core volume",
}


# ---------------------------------------------------------------------------
# ESP Partition Analysis
# ---------------------------------------------------------------------------

def scan_esp_partition(esp_mount_path):
    """Scan a mounted EFI System Partition for bootkit indicators."""
    findings = []
    if not os.path.isdir(esp_mount_path):
        return [{"severity": "ERROR", "message": f"ESP path not found: {esp_mount_path}"}]

    # Check for BlackLotus system32 directory
    system32_path = os.path.join(esp_mount_path, "system32")
    if os.path.exists(system32_path):
        findings.append({
            "severity": "CRITICAL",
            "indicator": "BlackLotus",
            "message": f"BlackLotus artifact: system32/ directory found on ESP at {system32_path}",
            "path": system32_path,
        })

    # Enumerate all EFI binaries
    efi_files = []
    for root, dirs, files in os.walk(esp_mount_path):
        for fname in files:
            if fname.lower().endswith(".efi"):
                full_path = os.path.join(root, fname)
                rel_path = os.path.relpath(full_path, esp_mount_path)
                file_hash = hash_file(full_path)
                file_size = os.path.getsize(full_path)
                efi_files.append({
                    "path": rel_path,
                    "full_path": full_path,
                    "sha256": file_hash,
                    "size": file_size,
                })

    # Check for unauthorized grubx64.efi (BlackLotus indicator)
    for ef in efi_files:
        if "grubx64.efi" in ef["path"].lower():
            # grubx64.efi on a Windows-only system is suspicious
            findings.append({
                "severity": "HIGH",
                "indicator": "BlackLotus/Bootkitty",
                "message": f"Suspicious grubx64.efi found: {ef['path']} ({ef['size']} bytes)",
                "sha256": ef["sha256"],
            })

    # Check for files outside standard EFI directories
    standard_dirs = {"efi", "boot", "microsoft", "ubuntu", "debian", "fedora", "grub"}
    for ef in efi_files:
        parts = Path(ef["path"]).parts
        top_dirs = {p.lower() for p in parts[:-1]}
        if not top_dirs.intersection(standard_dirs):
            findings.append({
                "severity": "MEDIUM",
                "indicator": "Unknown",
                "message": f"EFI binary in non-standard location: {ef['path']}",
                "sha256": ef["sha256"],
            })

    return findings, efi_files


def hash_file(file_path):
    """Compute SHA-256 hash of a file."""
    sha256 = hashlib.sha256()
    with open(file_path, "rb") as f:
        while True:
            chunk = f.read(65536)
            if not chunk:
                break
            sha256.update(chunk)
    return sha256.hexdigest()


# ---------------------------------------------------------------------------
# Firmware Analysis
# ---------------------------------------------------------------------------

EFI_FV_HEADER_MAGIC = b"_FVH"
PE_MAGIC = b"MZ"


def scan_firmware_dump(firmware_path):
    """Scan a raw firmware dump for EFI firmware volumes and PE/COFF executables."""
    if not os.path.isfile(firmware_path):
        return {"error": f"Firmware file not found: {firmware_path}"}

    file_size = os.path.getsize(firmware_path)
    with open(firmware_path, "rb") as f:
        data = f.read()

    firmware_hash = hashlib.sha256(data).hexdigest()
    results = {
        "file": os.path.basename(firmware_path),
        "size": file_size,
        "sha256": firmware_hash,
        "firmware_volumes": [],
        "pe_executables": [],
        "suspicious_strings": [],
    }

    # Find firmware volume headers (_FVH signature at offset 0x28 in FV header)
    offset = 0
    while offset < len(data) - 0x40:
        idx = data.find(EFI_FV_HEADER_MAGIC, offset)
        if idx == -1:
            break
        # FV header signature is at offset 0x28 from the start of the volume
        fv_start = idx - 0x28
        if fv_start >= 0:
            # Parse FV header length (8 bytes at offset 0x20)
            fv_length = struct.unpack_from("<Q", data, fv_start + 0x20)[0]
            # Extract GUID (16 bytes at offset 0x10)
            guid_bytes = data[fv_start + 0x10:fv_start + 0x20]
            guid_str = format_guid(guid_bytes)
            results["firmware_volumes"].append({
                "offset": f"0x{fv_start:08X}",
                "length": fv_length,
                "guid": guid_str,
                "description": KNOWN_FV_GUIDS.get(guid_str, "Unknown firmware volume"),
            })
        offset = idx + 4

    # Find PE/COFF executables (EFI binaries)
    offset = 0
    while offset < len(data) - 64:
        idx = data.find(PE_MAGIC, offset)
        if idx == -1:
            break
        # Verify PE header: check for "PE\0\0" at e_lfanew offset
        if idx + 0x3C < len(data):
            pe_offset = struct.unpack_from("<I", data, idx + 0x3C)[0]
            if idx + pe_offset + 4 < len(data):
                pe_sig = data[idx + pe_offset:idx + pe_offset + 4]
                if pe_sig == b"PE\x00\x00":
                    results["pe_executables"].append({
                        "offset": f"0x{idx:08X}",
                        "pe_header_offset": f"0x{idx + pe_offset:08X}",
                    })
        offset = idx + 2

    # Scan for suspicious strings in firmware
    suspicious_patterns = [
        (rb"rpcnetp", "LoJax dropper component"),
        (rb"autoche", "LoJax persistence component"),
        (rb"SmmAccessDxe", "Potential DXE driver modification target"),
        (rb"CORE_DXE", "Core DXE module (MoonBounce/CosmicStrand target)"),
        (rb"GetVariable", "UEFI runtime service (hooking target)"),
        (rb"SetVariable", "UEFI runtime service (variable manipulation)"),
        (rb"READY_TO_BOOT", "Boot event callback (MosaicRegressor indicator)"),
        (rb"\\EFI\\Microsoft\\Boot", "Windows boot path reference"),
        (rb"cmd\.exe", "Command shell reference in firmware (suspicious)"),
        (rb"powershell", "PowerShell reference in firmware (suspicious)"),
    ]
    for pattern, description in suspicious_patterns:
        matches = [m.start() for m in re.finditer(pattern, data, re.IGNORECASE)]
        if matches:
            results["suspicious_strings"].append({
                "pattern": pattern.decode("ascii", errors="replace"),
                "description": description,
                "occurrences": len(matches),
                "offsets": [f"0x{o:08X}" for o in matches[:5]],
            })

    return results


def format_guid(guid_bytes):
    """Format 16 raw GUID bytes into standard XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX form."""
    if len(guid_bytes) != 16:
        return guid_bytes.hex().upper()
    part1 = struct.unpack_from("<IHH", guid_bytes, 0)
    part2 = guid_bytes[8:16]
    return (f"{part1[0]:08X}-{part1[1]:04X}-{part1[2]:04X}-"
            f"{part2[0]:02X}{part2[1]:02X}-"
            f"{part2[2]:02X}{part2[3]:02X}{part2[4]:02X}"
            f"{part2[5]:02X}{part2[6]:02X}{part2[7]:02X}")


# ---------------------------------------------------------------------------
# Secure Boot Verification
# ---------------------------------------------------------------------------

def check_secure_boot_status():
    """Check Secure Boot status on the local system (Linux)."""
    results = {}
    # Try reading from efivarfs (Linux)
    efivar_path = "/sys/firmware/efi/efivars"
    if os.path.isdir(efivar_path):
        results["efi_available"] = True
        secureboot_var = os.path.join(
            efivar_path,
            f"SecureBoot-{SECUREBOOT_GUID.lower()}"
        )
        if os.path.exists(secureboot_var):
            with open(secureboot_var, "rb") as f:
                raw = f.read()
            # First 4 bytes are attributes, 5th byte is the value
            if len(raw) >= 5:
                value = raw[4]
                results["secure_boot_enabled"] = value == 1
                results["secure_boot_value"] = value
        setupmode_var = os.path.join(
            efivar_path,
            f"SetupMode-{SECUREBOOT_GUID.lower()}"
        )
        if os.path.exists(setupmode_var):
            with open(setupmode_var, "rb") as f:
                raw = f.read()
            if len(raw) >= 5:
                results["setup_mode"] = raw[4] == 1
    else:
        results["efi_available"] = False
        results["note"] = "Not a UEFI system or efivarfs not mounted"
    return results


# ---------------------------------------------------------------------------
# Chipsec Subprocess Interface
# ---------------------------------------------------------------------------

def run_chipsec_module(module_name, args=None):
    """Run a chipsec module via subprocess and return output."""
    cmd = ["python", "chipsec_main.py", "-m", module_name]
    if args:
        cmd.extend(["-a", args])
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
        return {
            "module": module_name,
            "stdout": result.stdout,
            "stderr": result.stderr,
            "rc": result.returncode,
            "passed": "PASSED" in result.stdout,
            "failed": "FAILED" in result.stdout or "WARNING" in result.stdout,
        }
    except FileNotFoundError:
        return {"module": module_name, "error": "chipsec not found in PATH", "rc": -1}
    except subprocess.TimeoutExpired:
        return {"module": module_name, "error": "chipsec module timed out", "rc": -2}


def run_chipsec_spi_dump(output_path):
    """Dump SPI flash contents via chipsec."""
    cmd = ["python", "chipsec_util.py", "spi", "dump", output_path]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        return {"stdout": result.stdout, "stderr": result.stderr, "rc": result.returncode}
    except FileNotFoundError:
        return {"error": "chipsec not found in PATH", "rc": -1}
    except subprocess.TimeoutExpired:
        return {"error": "SPI dump timed out", "rc": -2}


def run_firmware_security_audit():
    """Run a comprehensive set of chipsec security modules."""
    modules = [
        ("common.bios_wp", "BIOS region write protection"),
        ("common.spi_lock", "SPI flash controller lock"),
        ("common.spi_access", "SPI flash region access permissions"),
        ("common.spi_desc", "SPI flash descriptor security"),
        ("common.secureboot.variables", "Secure Boot variable configuration"),
        ("common.smm", "SMM protection (SMRAM range)"),
        ("common.bios_smi", "SMI suppression / BIOS write via SMI"),
    ]
    results = {}
    for module, description in modules:
        print(f"  Running: {module} ({description})...")
        result = run_chipsec_module(module)
        result["description"] = description
        results[module] = result
    return results


# ---------------------------------------------------------------------------
# Entropy Analysis for Firmware Regions
# ---------------------------------------------------------------------------

def firmware_entropy_map(firmware_path, block_size=4096):
    """Generate block-level entropy map to detect encrypted/compressed firmware regions."""
    results = []
    with open(firmware_path, "rb") as f:
        offset = 0
        while True:
            block = f.read(block_size)
            if not block:
                break
            counter = Counter(block)
            length = len(block)
            if length == 0:
                entropy = 0.0
            else:
                entropy = -sum(
                    (c / length) * math.log2(c / length)
                    for c in counter.values()
                )
            classification = "empty" if entropy < 1.0 else \
                            "code/data" if entropy < 5.0 else \
                            "compressed" if entropy < 7.5 else "encrypted/random"
            results.append({
                "offset": f"0x{offset:08X}",
                "entropy": round(entropy, 4),
                "classification": classification,
            })
            offset += len(block)
    return results


# ---------------------------------------------------------------------------
# Main Entry Point
# ---------------------------------------------------------------------------

def analyze_uefi_bootkit(target_path, target_type="firmware"):
    """Perform UEFI bootkit persistence analysis on a firmware dump or ESP mount point."""
    print("=" * 65)
    print("  UEFI Bootkit Persistence Analysis Agent")
    print("=" * 65)

    if target_type == "firmware" and os.path.isfile(target_path):
        print(f"\n[*] Analyzing firmware dump: {target_path}")
        print(f"[*] File size: {os.path.getsize(target_path)} bytes")
        print(f"[*] SHA-256: {hash_file(target_path)}")

        # Firmware volume and PE scan
        print("\n--- Firmware Structure Analysis ---")
        fw_results = scan_firmware_dump(target_path)
        print(f"  Firmware volumes found: {len(fw_results['firmware_volumes'])}")
        for fv in fw_results["firmware_volumes"]:
            print(f"    {fv['offset']}  GUID={fv['guid']}  Size={fv['length']}  [{fv['description']}]")
        print(f"  PE/COFF executables found: {len(fw_results['pe_executables'])}")
        for pe in fw_results["pe_executables"][:10]:
            print(f"    {pe['offset']}  (PE header at {pe['pe_header_offset']})")

        # Suspicious strings
        if fw_results["suspicious_strings"]:
            print("\n--- Suspicious Strings in Firmware ---")
            for ss in fw_results["suspicious_strings"]:
                print(f"  [!] {ss['description']}: \"{ss['pattern']}\" "
                      f"({ss['occurrences']} occurrences)")
                for off in ss["offsets"]:
                    print(f"      at {off}")

        # Entropy analysis
        print("\n--- Firmware Entropy Analysis ---")
        emap = firmware_entropy_map(target_path, block_size=16384)
        region_counts = Counter(e["classification"] for e in emap)
        for classification, count in region_counts.most_common():
            print(f"  {classification}: {count} blocks")

    elif target_type == "esp" and os.path.isdir(target_path):
        print(f"\n[*] Analyzing ESP mount point: {target_path}")

        # ESP analysis
        print("\n--- ESP Partition Analysis ---")
        findings, efi_files = scan_esp_partition(target_path)
        print(f"  Total EFI binaries: {len(efi_files)}")
        for ef in efi_files:
            print(f"    {ef['path']}  ({ef['size']} bytes)  SHA-256={ef['sha256'][:16]}...")

        if findings:
            print("\n--- Bootkit Indicators ---")
            for f in findings:
                print(f"  [{f['severity']}] {f['message']}")
        else:
            print("\n  No bootkit indicators found on ESP.")

    else:
        print(f"\n[ERROR] Invalid target: {target_path} (type={target_type})")
        return

    # Known bootkit reference
    print("\n--- Known UEFI Bootkit Families ---")
    for name, info in KNOWN_BOOTKITS.items():
        print(f"  {name}: {info['description']}")
        print(f"    Persistence: {info['persistence']}")
        print(f"    MITRE: {info['mitre']}")

    print("\n[*] Analysis complete.")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="UEFI bootkit persistence analysis agent for detecting firmware "
                    "implants, ESP modifications, Secure Boot bypasses, and UEFI "
                    "variable manipulation.",
        epilog="Authorized use only. Requires appropriate system access for firmware analysis.",
    )
    parser.add_argument(
        "target",
        help="Path to a firmware dump (.rom, .bin) or a mounted ESP directory",
    )
    parser.add_argument(
        "--type", "-t",
        choices=["firmware", "esp", "auto"],
        default="auto",
        help="Target type: 'firmware' for SPI flash dumps, 'esp' for mounted ESP "
             "partition, 'auto' to detect (default: auto)",
    )
    parser.add_argument(
        "--check-secureboot", "-s",
        action="store_true",
        help="Check Secure Boot status on the local system (Linux efivarfs)",
    )
    parser.add_argument(
        "--run-chipsec-audit", "-c",
        action="store_true",
        help="Run comprehensive chipsec firmware security audit modules",
    )
    parser.add_argument(
        "--baseline", "-b",
        type=str, default=None,
        help="Path to known-good firmware baseline for comparison",
    )
    parser.add_argument(
        "--json-output", "-j",
        action="store_true",
        help="Output results in JSON format instead of text",
    )
    parser.add_argument(
        "--list-bootkits",
        action="store_true",
        help="List all known UEFI bootkit families in the database and exit",
    )

    args = parser.parse_args()
    print(DISCLAIMER)

    if args.list_bootkits:
        print("Known UEFI Bootkit Families:")
        print("-" * 50)
        for name, info in KNOWN_BOOTKITS.items():
            print(f"\n  {name}")
            print(f"    {info['description']}")
            print(f"    Persistence: {info['persistence']}")
            print(f"    MITRE ATT&CK: {info['mitre']}")
            if info.get("cve"):
                print(f"    CVE: {info['cve']}")
        sys.exit(0)

    target_type = args.type
    if target_type == "auto":
        target_type = "esp" if os.path.isdir(args.target) else "firmware"

    analyze_uefi_bootkit(args.target, target_type)

    if args.check_secureboot:
        print("\n--- Local Secure Boot Status ---")
        sb_status = check_secure_boot_status()
        for k, v in sb_status.items():
            print(f"  {k}: {v}")

    if args.run_chipsec_audit:
        print("\n--- Chipsec Firmware Security Audit ---")
        audit_results = run_firmware_security_audit()
        for module, result in audit_results.items():
            status = "PASSED" if result.get("passed") else "FAILED" if result.get("failed") else "UNKNOWN"
            print(f"  [{status}] {module}: {result.get('description', '')}")

===== END FILE =====
