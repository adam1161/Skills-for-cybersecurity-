

===== FILE: skills/auditing-uefi-firmware-with-chipsec/SKILL.md =====
---
name: auditing-uefi-firmware-with-chipsec
description: Use Intel CHIPSEC to assess platform firmware configuration, SPI flash write protection, BIOS lock, SMM/SMRR, and Secure Boot variable state, dump SPI flash, and triage UEFI variables for firmware-level threats.
domain: cybersecurity
subdomain: hardware-firmware-security
tags:
- hardware-firmware-security
- uefi
- chipsec
- spi-flash
- bios-write-protection
- secure-boot
- firmware-assessment
- platform-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.AM-02
mitre_attack:
- T1542.001
---
# Auditing UEFI Firmware with CHIPSEC

> **Authorized Use Only:** CHIPSEC loads a kernel driver and reads/writes low-level hardware registers, SPI flash, and SMM. Run it only on systems you own or are explicitly authorized to assess, ideally on dedicated test hardware. Misuse (especially write/modify modules) can brick a machine. Never run write-capable modules on production systems.

## Overview

CHIPSEC is the open-source **Platform Security Assessment Framework** created by Intel's Advanced Threat Research team. It inspects the low-level security configuration of x86 platform firmware and hardware — the layer below the operating system where bootkits and firmware implants live. CHIPSEC loads a signed kernel driver (Linux, Windows, or it can run from the UEFI shell) to read and write hardware registers, Model-Specific Registers (MSRs), PCI config space, SPI flash, and UEFI variables, then runs an automated test suite that checks whether the platform's defensive locks are actually engaged.

The threat CHIPSEC addresses is MITRE ATT&CK **T1542.001 — Pre-OS Boot: System Firmware**: adversaries who modify system firmware (the BIOS/UEFI image on SPI flash) to gain stealthy, persistent, OS-survivable control. Firmware implants persist across OS reinstall and disk replacement and are invisible to most EDR. CHIPSEC's value is verifying the *prerequisites* that prevent such implants: that the SPI flash BIOS region is write-protected (BIOS_CNTL `BLE`/`SMM_BWP`, SPI Protected Ranges), that the flash descriptor locks region access, that SMRAM/SMRR are configured, and that Secure Boot variables are protected. It also dumps the SPI flash for offline forensic comparison.

Sources: Intel/CHIPSEC project (https://github.com/chipsec/chipsec), CHIPSEC documentation (https://chipsec.github.io/).

## When to Use

- Baseline firmware-security assessment of a new laptop/server platform or fleet image
- Verifying that BIOS write protection and SPI flash locks are correctly enabled by the OEM
- Firmware forensics: dumping SPI flash to compare against a known-good image
- Validating Secure Boot variable protection and S3 boot-script protection
- Hunting for evidence of a firmware implant or misconfiguration enabling one

## Prerequisites

- Physical or admin/root access to the target x86 platform (Intel or AMD)
- Linux (root) or Windows (Administrator), or a UEFI shell environment
- Ability to load a kernel driver (Secure Boot may need to allow the CHIPSEC driver, or use `--no_driver` for limited checks)
- Python 3.8+ and a C compiler/build tools for the kernel module on Linux
- Dedicated test hardware strongly recommended

Install CHIPSEC:

```bash
# From PyPI
pip install chipsec

# Or from source (builds the kernel helper/driver)
git clone https://github.com/chipsec/chipsec
cd chipsec
python setup.py install        # builds and installs, including the Linux driver

# Verify
sudo chipsec_main --help
sudo chipsec_util --help
```

## Objectives

- Run the full automated platform-security test suite and interpret PASS/FAIL/WARNING
- Verify BIOS write protection (BIOS_CNTL) and SPI Protected Ranges
- Verify the SPI flash descriptor locks region read/write access
- Verify SMRAM/SMRR and SMI handler protections
- Verify Secure Boot variable protection and S3 boot-script protection
- Dump SPI flash and decode it for offline analysis
- Enumerate UEFI variables and detect anomalous/unexpected entries

## MITRE ATT&CK Mapping

| Technique ID | Name | Tactic |
|--------------|------|--------|
| T1542.001 | Pre-OS Boot: System Firmware | Persistence / Defense Evasion |

CHIPSEC defends against T1542.001 by verifying that the controls preventing unauthorized firmware modification are enabled. A FAIL on `common.bios_wp` (BIOS not write-protected) or `chipsec.modules.common.spi_lock` (flash descriptor unlocked) means an attacker with OS privileges could rewrite the SPI flash and implant persistent firmware — exactly the precondition for this technique.

## Workflow

### Step 1: Run the full automated test suite
`chipsec_main` with no module argument runs every applicable security check for the detected platform and prints a summary of PASS/FAIL/WARNING/INFORMATION results.

```bash
sudo chipsec_main

# Save machine-readable output for reporting / diffing
sudo chipsec_main -j results.json -x results.xml -l chipsec.log
```

### Step 2: Run the core firmware-protection modules individually
The `common` module group contains the OEM-independent security checks. Run the group or specific modules:

```bash
# Run the whole common group
sudo chipsec_main -m common

# BIOS write protection: checks BIOS_CNTL BLE/SMM_BWP and SPI protected ranges
sudo chipsec_main -m common.bios_wp

# SPI flash descriptor lock (FLOCKDN) — are flash region accesses locked?
sudo chipsec_main -m common.spi_lock

# SMRR programming — protects SMRAM from cache-based attacks
sudo chipsec_main -m common.smrr

# SMM BIOS write protection
sudo chipsec_main -m common.smm

# S3 resume boot-script protection (against bootscript table attacks)
sudo chipsec_main -m common.uefi.s3bootscript
```

### Step 3: Verify Secure Boot variable protection
```bash
# Checks that Secure Boot UEFI variables are properly protected
sudo chipsec_main -m common.secureboot.variables

# To actively test write protection of the variables (test hardware ONLY):
sudo chipsec_main -m common.secureboot.variables -a modify
```

### Step 4: Inspect SPI flash region access permissions
```bash
# Report SPI flash regions, descriptor, and access permissions
sudo chipsec_util spi info

# Check the SPI access-control module
sudo chipsec_main -m common.spi_access
```

### Step 5: Dump SPI flash for offline forensics
Dumping the flash lets you decode the firmware volumes and compare against a known-good OEM image.

```bash
# Dump the entire SPI flash to a file
sudo chipsec_util spi dump rom.bin

# Decode the dumped image: extracts firmware volumes, files, NVRAM variables, etc.
sudo chipsec_util decode rom.bin
```

### Step 6: Enumerate and triage UEFI variables
```bash
# List all UEFI variables from the runtime interface
sudo chipsec_util uefi var-list

# List variables directly from the SPI image (offline)
sudo chipsec_util uefi var-find PK
sudo chipsec_util uefi var-read db <GUID> db.bin

# Decode the UEFI firmware structure
sudo chipsec_util uefi decode rom.bin
```

### Step 7: Limited assessment without a kernel driver
Where loading the driver is impossible (locked-down Secure Boot), some checks still run read-only.

```bash
sudo chipsec_main -n            # --no_driver: skip checks that need the driver
sudo chipsec_main -p <PLATFORM> # force platform code if auto-detect fails
```

### Step 8: Triage results and report
- **FAIL** on `bios_wp` / `spi_lock` → firmware is rewritable from the OS: high risk for T1542.001.
- **FAIL** on `secureboot.variables` → Secure Boot policy can be tampered.
- Compare the `spi dump` against the OEM's known-good image (hash firmware volumes) to detect unauthorized modification.
- Record platform, BIOS version, and every FAIL/WARNING with the relevant register values for the report.

## Tools and Resources

| Tool | Purpose | Source |
|------|---------|--------|
| chipsec_main | Automated platform-security test suite | https://github.com/chipsec/chipsec |
| chipsec_util | Manual hardware/firmware access (spi, uefi, decode) | https://chipsec.github.io/ |
| UEFITool | GUI/CLI parsing of dumped UEFI images | https://github.com/LongSoft/UEFITool |
| Binarly fwhunt | Firmware vulnerability/implant hunting rules | https://github.com/binarly-io/fwhunt-scan |
| NSA UEFI Secure Boot guidance | Hardening reference | https://media.defense.gov/ |

## Core Module Reference

| Module | Checks |
|--------|--------|
| common.bios_wp | BIOS_CNTL BLE / SMM_BWP and SPI Protected Ranges |
| common.spi_lock | SPI flash descriptor FLOCKDN |
| common.spi_access | SPI flash region read/write permissions |
| common.smrr | System Management Range Registers programming |
| common.smm | SMM BIOS write protection |
| common.secureboot.variables | Secure Boot variable protection |
| common.uefi.s3bootscript | S3 resume boot-script protection |

## Validation Criteria

- [ ] CHIPSEC installed and driver loads (or `-n` documented if not)
- [ ] Full `chipsec_main` suite executed with JSON/XML/log output saved
- [ ] `common.bios_wp` result interpreted (write protection state)
- [ ] `common.spi_lock` / `spi_access` result interpreted (descriptor lock)
- [ ] SMRR/SMM module results recorded
- [ ] Secure Boot variable protection checked
- [ ] SPI flash dumped and decoded for offline analysis
- [ ] UEFI variables enumerated and triaged
- [ ] All FAIL/WARNING findings documented with platform/BIOS version
- [ ] Write/modify modules NOT run on production hardware

===== END FILE =====


===== FILE: skills/auditing-uefi-firmware-with-chipsec/LICENSE =====

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


===== FILE: skills/auditing-uefi-firmware-with-chipsec/references/api-reference.md =====
# Command Reference - CHIPSEC

## chipsec_main (test suite)

| Command / Flag | Purpose |
|----------------|---------|
| `chipsec_main` | Run all applicable security modules for the platform |
| `-m, --module <name>` | Run a specific module, e.g. `-m common.bios_wp` |
| `-m common` | Run the whole OEM-independent module group |
| `-mx <modules>` | Exclude listed modules |
| `-a, --module_args` | Pass arguments to a module (e.g. `-a modify`) |
| `-p, --platform <code>` | Force platform code when auto-detect fails |
| `-n, --no_driver` | Skip checks requiring the kernel driver |
| `-l, --log <file>` | Write output to a log file |
| `-j, --json <file>` | JSON results output |
| `-x, --xml <file>` | JUnit-style XML results output |
| `-v / -vv / -d` | Verbose / very verbose / debug logging |

## Key security modules

| Module | Checks |
|--------|--------|
| `common.bios_wp` | BIOS_CNTL BLE / SMM_BWP and SPI Protected Ranges |
| `common.spi_lock` | SPI flash descriptor FLOCKDN lock |
| `common.spi_access` | SPI flash region access permissions |
| `common.smrr` | SMRR programming (SMRAM cache protection) |
| `common.smm` | SMM BIOS write protection |
| `common.secureboot.variables` | Secure Boot variable protection (`-a modify` to test writes) |
| `common.uefi.s3bootscript` | S3 resume boot-script protection |

## chipsec_util (manual access)

| Command | Purpose |
|---------|---------|
| `chipsec_util spi info` | Report SPI flash regions/descriptor/permissions |
| `chipsec_util spi dump rom.bin` | Dump entire SPI flash to file |
| `chipsec_util spi read <addr> <len> out.bin` | Read SPI flash range |
| `chipsec_util decode rom.bin` | Decode dumped image into volumes/files/variables |
| `chipsec_util uefi var-list` | List UEFI variables (runtime) |
| `chipsec_util uefi var-find <name>` | Find a UEFI variable |
| `chipsec_util uefi var-read <name> <GUID> out.bin` | Read a variable |
| `chipsec_util uefi decode rom.bin` | Decode UEFI firmware structure from image |
| `chipsec_util platform` | Show detected platform info |

## Result interpretation

| Result | Meaning |
|--------|---------|
| PASSED | Protection is correctly enabled |
| FAILED | Protection missing/misconfigured — exploitable |
| WARNING | Potential issue / needs manual review |
| INFORMATION | Informational, no pass/fail |

===== END FILE =====


===== FILE: skills/auditing-uefi-firmware-with-chipsec/references/standards.md =====
# Standards and References - Auditing UEFI Firmware with CHIPSEC

## MITRE ATT&CK

| Technique ID | Name | Tactic | Rationale |
|--------------|------|--------|-----------|
| T1542.001 | Pre-OS Boot: System Firmware | Persistence / Defense Evasion | CHIPSEC verifies the SPI flash and BIOS write-protection locks whose absence enables adversaries to modify system firmware for stealthy, OS-survivable persistence. |

## NIST Cybersecurity Framework 2.0

| ID | Name | Rationale |
|----|------|-----------|
| ID.AM-02 | Inventories of software, services, and systems managed by the organization are maintained | Firmware version, SPI flash image, and platform-security configuration are part of the asset/software inventory that CHIPSEC enumerates and baselines. |

## Official Resources

- CHIPSEC project: https://github.com/chipsec/chipsec
- CHIPSEC documentation: https://chipsec.github.io/
- Running CHIPSEC: https://chipsec.github.io/usage/Running-Chipsec.html
- common.bios_wp module: https://chipsec.github.io/modules/
- common.secureboot.variables module docs
- UEFITool: https://github.com/LongSoft/UEFITool
- Binarly fwhunt-scan: https://github.com/binarly-io/fwhunt-scan

## Key Research

- Intel ATR: original CHIPSEC framework and BlackHat arsenal presentations
- "Exploring Your System Deeper with CHIPSEC" (CSW)
- NSA UEFI Secure Boot Customization / firmware hardening guidance

===== END FILE =====


===== FILE: skills/auditing-uefi-firmware-with-chipsec/scripts/agent.py =====
#!/usr/bin/env python3
"""
chipsec-audit — run a CHIPSEC firmware-posture baseline and parse results.

Wraps the real CHIPSEC CLIs:
  - chipsec_main : module runner
  - chipsec_util : SPI/UEFI utilities
Project: https://github.com/chipsec/chipsec

Install:
    pip install chipsec     # needs root and a kernel driver build

Examples:
    sudo python agent.py baseline --log chipsec.log --json report.json
    sudo python agent.py module --name common.bios_wp
    sudo python agent.py dump --out rom.bin
"""
import argparse
import json
import os
import re
import shutil
import subprocess
import sys

# CHIPSEC prints lines like:  [+] PASSED: BIOS region write protection ...
RESULT_RE = re.compile(r"\[[+\-!*]\]\s*(PASSED|FAILED|WARNING|ERROR|NOT APPLICABLE|INFORMATION)\b[:\s]*(.*)",
                       re.IGNORECASE)

CORE_MODULES = [
    "common.bios_wp",
    "common.spi_lock",
    "common.spi_desc",
    "common.smm",
    "common.secureboot.variables",
]


def _require(tool: str) -> None:
    if shutil.which(tool) is None:
        sys.exit(f"error: '{tool}' not found on PATH. Install CHIPSEC: pip install chipsec")


def _warn_root() -> None:
    if hasattr(os, "geteuid") and os.geteuid() != 0:
        print("[!] warning: CHIPSEC needs root/Administrator to load its driver.", file=sys.stderr)


def _run(argv: list) -> subprocess.CompletedProcess:
    print(f"[*] {' '.join(argv)}", file=sys.stderr)
    return subprocess.run(argv, capture_output=True, text=True)


def parse_results(stdout: str) -> list:
    findings = []
    for line in stdout.splitlines():
        m = RESULT_RE.search(line)
        if m:
            findings.append({"result": m.group(1).upper(), "detail": m.group(2).strip()})
    return findings


def cmd_baseline(args) -> int:
    _require("chipsec_main")
    _warn_root()
    argv = ["chipsec_main"]
    if args.log:
        argv += ["-l", args.log]
    proc = _run(argv)
    findings = parse_results(proc.stdout)
    fails = [f for f in findings if f["result"] in ("FAILED", "ERROR")]
    warns = [f for f in findings if f["result"] == "WARNING"]
    summary = {
        "total_results": len(findings),
        "failed": len(fails),
        "warnings": len(warns),
        "failures": fails,
        "warning_items": warns,
    }
    if args.json:
        with open(args.json, "w", encoding="utf-8") as fh:
            json.dump({"summary": summary, "all": findings}, fh, indent=2)
        print(f"[+] report written to {args.json}", file=sys.stderr)
    print(json.dumps(summary, indent=2))
    return 1 if fails else 0


def cmd_module(args) -> int:
    _require("chipsec_main")
    _warn_root()
    targets = [args.name] if args.name else CORE_MODULES
    out = {}
    rc = 0
    for mod in targets:
        proc = _run(["chipsec_main", "-m", mod])
        res = parse_results(proc.stdout)
        out[mod] = res
        if any(r["result"] in ("FAILED", "ERROR") for r in res):
            rc = 1
    print(json.dumps(out, indent=2))
    return rc


def cmd_dump(args) -> int:
    _require("chipsec_util")
    _warn_root()
    _run(["chipsec_util", "spi", "info"])
    proc = _run(["chipsec_util", "spi", "dump", args.out])
    if proc.returncode != 0 or not os.path.exists(args.out):
        print("[!] SPI dump failed", file=sys.stderr)
        print(proc.stderr, file=sys.stderr)
        return 1
    size = os.path.getsize(args.out)
    print(json.dumps({"dump": args.out, "bytes": size}, indent=2))
    print("[*] decode offline with: chipsec_util uefi decode " + args.out, file=sys.stderr)
    return 0


def main() -> int:
    p = argparse.ArgumentParser(description="CHIPSEC firmware-posture audit helper.")
    sub = p.add_subparsers(dest="cmd", required=True)

    b = sub.add_parser("baseline", help="run full chipsec_main and summarize")
    b.add_argument("--log", help="CHIPSEC log file path")
    b.add_argument("--json", help="write parsed JSON report here")
    b.set_defaults(func=cmd_baseline)

    m = sub.add_parser("module", help="run one or the core security modules")
    m.add_argument("--name", help="specific module, e.g. common.bios_wp (default: core set)")
    m.set_defaults(func=cmd_module)

    d = sub.add_parser("dump", help="dump SPI flash (read-only)")
    d.add_argument("--out", default="rom.bin")
    d.set_defaults(func=cmd_dump)

    args = p.parse_args()
    return args.func(args)


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/detecting-secure-boot-bypass/SKILL.md =====
---
name: detecting-secure-boot-bypass
description: Detect UEFI Secure Boot bypasses and bootkits such as BlackLotus and
  Bootkitty by verifying Secure Boot state, checking dbx revocation currency, and
  hashing EFI boot binaries against known-bad sets using mokutil, efi-readvar/dbxtool,
  CHIPSEC, sbverify/pesign, and Windows Confirm-SecureBootUEFI. Use when auditing
  fleet dbx rollout after a bootkit advisory or hunting for pre-OS persistence on
  a suspected-compromised endpoint.
domain: cybersecurity
subdomain: hardware-firmware-security
tags:
- hardware-firmware-security
- secure-boot
- uefi
- bootkit
- dbx-revocation
- blacklotus
- chipsec
- firmware-integrity
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
mitre_attack:
- T1542.003
---
# Detecting Secure Boot Bypass

> **Legal Notice:** Firmware and Secure Boot assessment must only be performed on systems you own or are explicitly authorized to test. CHIPSEC's write/modify modes and EFI variable manipulation can brick hardware. Run destructive checks only in a lab. This skill is for defensive verification and authorized assessment.

## Overview

UEFI Secure Boot is the firmware-enforced trust chain that only permits boot components signed by keys in the platform's allow-list (db) and not present in the revocation list (dbx). Bootkits defeat this layer to gain pre-OS, persistent, kernel-level control that survives OS reinstalls and disk wipes. **BlackLotus** (2023) was the first publicly observed UEFI bootkit to bypass Secure Boot on fully patched Windows 11, abusing **CVE-2022-21894** ("baton drop") in a vulnerable, signed Windows boot manager to neutralize Secure Boot and disable BitLocker, HVCI, and Defender. **Bootkitty** (2024) was the first PoC UEFI bootkit targeting Linux. Microsoft's **CVE-2023-24932** addressed a related Secure Boot bypass that required a phased dbx (revocation) rollout because revoking the vulnerable boot managers can render systems unbootable if applied carelessly.

The core defensive insight is that patching the OS is **not sufficient** — the platform remains exploitable until the vulnerable, signed binaries are revoked in **dbx**. Detection therefore combines: (1) confirming Secure Boot is actually enabled, (2) verifying dbx is current and contains the relevant revocations, (3) checking the integrity and protection of Secure Boot EFI variables with CHIPSEC, (4) hashing on-disk EFI boot binaries and comparing them against the revocation list and known-bad sets, and (5) inspecting firmware/ESP for bootkit artifacts. This skill provides a cross-platform (Linux + Windows) workflow using `mokutil`, `efi-readvar`/`dbxtool`, `chipsec`, `sbverify`/`pesign`, and Windows `Confirm-SecureBootUEFI` / `Get-SecureBootUEFI`.

## When to Use

- Verifying that an estate's Secure Boot configuration is enabled, locked, and current after the CVE-2023-24932 / BlackLotus advisories.
- Hunting for UEFI bootkit indicators on a suspected-compromised endpoint.
- Validating that dbx revocations (e.g., vulnerable Windows boot managers, Kaspersky/other vulnerable bootloaders) have actually applied across the fleet.
- Auditing firmware integrity and Secure Boot variable protections during a hardware security assessment.
- Building a recurring measured/baseline check for boot-chain tampering.

## Prerequisites

- Root/administrator on the target (firmware reads require privilege).
- Linux tooling:
  ```bash
  sudo apt install mokutil efitools sbsigntool dbxtool      # Debian/Ubuntu
  sudo dnf install mokutil efitools sbsigntools dbxtool     # Fedora/RHEL
  ```
- CHIPSEC (run from a live USB or controlled host; loads a kernel driver):
  ```bash
  pip install chipsec        # or build from https://github.com/chipsec/chipsec
  ```
- Windows tooling: PowerShell (built-in `Confirm-SecureBootUEFI`, `Get-SecureBootUEFI`), and optionally the UEFI dbx update package from Microsoft.
- The current `dbxupdate` files from https://uefi.org/revocationlistfile to compare against.

## Objectives

- Confirm Secure Boot is enabled and in user (not setup) mode.
- Enumerate db, dbx, KEK, and PK contents and assess freshness of dbx.
- Verify the relevant CVE revocations are present in dbx.
- Validate Secure Boot EFI variables are authenticated and protected (CHIPSEC).
- Hash ESP boot binaries and check them against dbx and known-bad hash sets.
- Identify bootkit artifacts and report exploitable gaps with remediation.

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Relevance |
|--------------|----------------|-----------|
| T1542.003 | Pre-OS Boot: Bootkit | Core technique — bootkit subverts the boot chain below the OS. |
| T1542 | Pre-OS Boot | Parent technique covering firmware/boot-component tampering. |
| T1542.001 | Pre-OS Boot: System Firmware | Adjacent: firmware modification used to persist or weaken Secure Boot. |
| T1014 | Rootkit | Bootkits provide rootkit-level concealment and persistence. |
| T1562.001 | Impair Defenses: Disable or Modify Tools | BlackLotus disables BitLocker/HVCI/Defender after bypassing Secure Boot. |

## Workflow

### 1. Confirm Secure Boot state (Linux)
A disabled or setup-mode platform offers no protection.
```bash
mokutil --sb-state                       # "SecureBoot enabled" expected
bootctl status | grep -i "secure boot"   # systemd-boot view
# 6 = enabled+user mode on the EFI SecureBoot/SetupMode vars:
od -An -t u1 /sys/firmware/efi/efivars/SecureBoot-8be4df61-93ca-11d2-aa0d-00e098032b8c
```

### 2. Confirm Secure Boot state (Windows)
```powershell
Confirm-SecureBootUEFI        # $true if enabled
# Inspect the raw dbx variable from Windows:
[System.BitConverter]::ToString((Get-SecureBootUEFI dbx).bytes) | Out-File dbx.hex
```

### 3. Enumerate the Secure Boot databases
List db (allowed), dbx (revoked), KEK, and PK.
```bash
efi-readvar                      # dumps PK, KEK, db, dbx
efi-readvar -v dbx -o dbx.esl    # export dbx to a file for offline analysis
mokutil --list-enrolled          # MOK (shim) enrolled keys
mokutil --db                     # platform db entries via shim
```

### 4. Assess dbx freshness and applied revocations
Compare on-system dbx to the current official UEFI revocation list.
```bash
dbxtool --list                                   # current dbx entries + count
# Download latest dbxupdate from uefi.org/revocationlistfile, then:
dbxtool --dbx ./DBXUpdate.bin --apply --dry-run  # show what WOULD be added (no write)
```
A low dbx entry count or absence of recent revocations indicates the platform is behind and likely still vulnerable to known bypasses.

### 5. Check Secure Boot variable protection with CHIPSEC
Verify the SB key variables are authenticated and not freely writable.
```bash
# Verify Secure Boot is enabled and the SB variables are properly protected:
sudo chipsec_main -m common.secureboot.variables

# Check S3 resume boot-script protections (an SMM/firmware bypass vector):
sudo chipsec_main -m common.uefi.s3bootscript

# Dump SPI flash for offline firmware diffing:
sudo chipsec_util spi dump rom.bin
```

### 6. Hash ESP boot binaries and check signatures
Verify bootloaders are signed and not present in dbx.
```bash
# Locate and hash EFI boot binaries
find /boot/efi -iname '*.efi' -exec sha256sum {} \;
# Validate a binary's signature against the platform db
sbverify --list /boot/efi/EFI/BOOT/bootx64.efi
sbverify --cert /etc/secureboot/db.crt /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi
# pesign equivalent on RHEL-family:
pesign -S -i /boot/efi/EFI/BOOT/bootx64.efi
```

### 7. Cross-check against known-bad bootkit hashes
Compare collected hashes to revocation/known-bad sets (e.g., LoFP / vendor advisories).
```bash
# Example: confirm a binary's SHA-256 is NOT one of the revoked CVE-2022-21894 boot managers
sha256sum /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi
# Compare against the hash list extracted from the latest dbxupdate / advisory.
```

### 8. Inspect for bootkit artifacts
Look for unauthorized ESP modifications and self-deployment markers.
```bash
# Unexpected files / recently modified binaries on the ESP
ls -laR /boot/efi/EFI/
find /boot/efi -newermt "-30 days" -iname '*.efi'
# On Windows, examine the EFI partition for rogue \EFI\Microsoft\Boot entries.
```

### 9. Validate measured-boot evidence (optional pivot)
If a TPM is present, current PCR[7] reflects Secure Boot policy; deviations corroborate tampering.
```bash
tpm2_pcrread sha256:7        # Secure Boot policy PCR
```

### 10. Run the bundled assessment helper
`agent.py` collects SB state, dbx counts, ESP binary hashes, and CHIPSEC results into one report.
```bash
sudo python scripts/agent.py --check-chipsec --output secureboot_report.json
```

## Tools and Resources

| Tool | Purpose | Source |
|------|---------|--------|
| mokutil | Secure Boot state and enrolled keys (Linux) | https://github.com/lcp/mokutil |
| efitools (efi-readvar) | Dump PK/KEK/db/dbx | https://git.kernel.org/pub/scm/linux/kernel/git/jejb/efitools.git |
| dbxtool | Inspect and apply dbx updates | https://github.com/rhboot/dbxtool |
| CHIPSEC | Firmware / Secure Boot variable assessment | https://github.com/chipsec/chipsec |
| sbsigntool / pesign | EFI binary signature verification | https://github.com/jejb/sbsigntools |
| UEFI Revocation List | Official dbx update files | https://uefi.org/revocationlistfile |
| Microsoft KB CVE-2023-24932 | Secure Boot bypass guidance | https://support.microsoft.com/topic/kb5025885 |
| ESET BlackLotus analysis | Bootkit technical writeup | https://www.welivesecurity.com/2023/03/01/blacklotus-uefi-bootkit-myth-confirmed/ |

## Validation Criteria

- [ ] Secure Boot confirmed enabled and in user mode on the target.
- [ ] PK/KEK/db/dbx enumerated and exported for analysis.
- [ ] dbx compared against the current official UEFI revocation list.
- [ ] CVE-2022-21894 / CVE-2023-24932 revocations confirmed present (or gap flagged).
- [ ] CHIPSEC `secureboot.variables` and `s3bootscript` modules run.
- [ ] ESP boot binaries hashed and signature-verified.
- [ ] Hashes cross-checked against known-bad / revoked sets.
- [ ] ESP inspected for unauthorized or recently modified binaries.
- [ ] Findings documented with remediation (dbx update / firmware update) per host.

===== END FILE =====


===== FILE: skills/detecting-secure-boot-bypass/LICENSE =====

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


===== FILE: skills/detecting-secure-boot-bypass/references/api-reference.md =====
# Command Reference

## mokutil (Linux Secure Boot state)

| Command | Description |
|---------|-------------|
| `mokutil --sb-state` | Report whether Secure Boot is enabled. |
| `mokutil --list-enrolled` | List enrolled MOK (Machine Owner Keys). |
| `mokutil --db` | Show platform db entries (via shim). |
| `mokutil --dbx` | Show dbx (revoked) entries via shim. |

## efitools

| Command | Description |
|---------|-------------|
| `efi-readvar` | Dump PK, KEK, db, dbx. |
| `efi-readvar -v dbx -o dbx.esl` | Export dbx to an EFI signature list file. |

## dbxtool

| Command | Description |
|---------|-------------|
| `dbxtool --list` | List current dbx entries and count. |
| `dbxtool --dbx DBXUpdate.bin --apply --dry-run` | Show revocations a given update would add (no write). |
| `dbxtool --dbx DBXUpdate.bin --apply` | Apply a dbx update (write — caution). |

## CHIPSEC

| Command | Description |
|---------|-------------|
| `chipsec_main -m common.secureboot.variables` | Verify SB key variables are authenticated/protected. |
| `chipsec_main -m common.secureboot.variables -a modify` | Attempt to write/corrupt SB vars (destructive test). |
| `chipsec_main -m common.uefi.s3bootscript` | Check S3 resume boot-script protections. |
| `chipsec_util spi dump rom.bin` | Dump SPI flash for offline analysis. |

## Signature verification

| Command | Description |
|---------|-------------|
| `sbverify --list <file.efi>` | List signatures on an EFI binary. |
| `sbverify --cert db.crt <file.efi>` | Verify a binary against a db cert. |
| `pesign -S -i <file.efi>` | Show signatures (RHEL family). |

## Windows PowerShell

| Cmdlet | Description |
|--------|-------------|
| `Confirm-SecureBootUEFI` | Returns `$true` if Secure Boot is enabled. |
| `Get-SecureBootUEFI dbx` | Retrieve the raw dbx variable bytes. |
| `Get-SecureBootUEFI db` | Retrieve the allowed-signatures database. |

## TPM corroboration

| Command | Description |
|---------|-------------|
| `tpm2_pcrread sha256:7` | Read PCR[7] (Secure Boot policy measurement). |

## Key references

- UEFI revocation list files: https://uefi.org/revocationlistfile
- Microsoft KB5025885 (CVE-2023-24932): https://support.microsoft.com/topic/kb5025885

===== END FILE =====


===== FILE: skills/detecting-secure-boot-bypass/references/standards.md =====
# Standards and Framework Mapping

## MITRE ATT&CK

| ID | Name | Rationale |
|----|------|-----------|
| T1542.003 | Pre-OS Boot: Bootkit | Core technique — bootkit subverts the boot chain below the OS to persist. |
| T1542 | Pre-OS Boot | Parent technique for firmware/boot-component tampering. |
| T1542.001 | Pre-OS Boot: System Firmware | Firmware modification used to weaken or bypass Secure Boot. |
| T1014 | Rootkit | Bootkits deliver rootkit-level stealth and persistence. |
| T1562.001 | Impair Defenses: Disable or Modify Tools | Post-bypass, BlackLotus disables BitLocker/HVCI/Defender. |

## NIST Cybersecurity Framework 2.0

| ID | Name | Rationale |
|----|------|-----------|
| DE.CM-01 | Networks and network services are monitored to find potentially adverse events | Applied here to boot-chain/firmware integrity monitoring — verifying Secure Boot, dbx, and EFI binaries surfaces tampering. |

## Supporting Standards and References

- **CVE-2022-21894 (Baton Drop).** Vulnerable signed Windows boot manager abused by BlackLotus to bypass Secure Boot.
- **CVE-2023-24932.** Secure Boot Security Feature Bypass; remediated via phased dbx revocation rollout (Microsoft KB5025885).
- **NSA UEFI Secure Boot Customization guidance.** Hardening and verification of the UEFI Secure Boot trust chain.
- **NIST SP 800-147 / 800-193 — Platform Firmware Resiliency.** Protection, detection, and recovery requirements for firmware integrity.
- **UEFI Specification — Secure Boot (db/dbx/KEK/PK).** Authoritative source for the variable model checked in this skill.

===== END FILE =====


===== FILE: skills/detecting-secure-boot-bypass/scripts/agent.py =====
#!/usr/bin/env python3
"""
Secure Boot bypass / bootkit detection helper (Linux + Windows-aware).

Collects:
  - Secure Boot enabled state
  - dbx (revocation list) entry count and freshness signal
  - SHA-256 hashes of EFI boot binaries on the ESP
  - CHIPSEC secureboot.variables result (optional, requires root + chipsec)

Emits a JSON report and a human-readable summary. Read-only by default.
Authorized assessment use only; run from a trusted/live environment.
"""
import argparse
import glob
import hashlib
import json
import os
import platform
import shutil
import subprocess
import sys
from pathlib import Path

ESP_PATHS = ["/boot/efi/EFI", "/boot/EFI", "/efi/EFI"]


def run(cmd: list[str], timeout: int = 300) -> tuple[int, str, str]:
    try:
        p = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
        return p.returncode, p.stdout, p.stderr
    except FileNotFoundError:
        return 127, "", f"not found: {cmd[0]}"
    except subprocess.SubprocessError as exc:
        return 1, "", str(exc)


def secure_boot_state() -> dict:
    state = {"platform": platform.system()}
    if platform.system() == "Linux":
        if shutil.which("mokutil"):
            rc, out, _ = run(["mokutil", "--sb-state"])
            state["mokutil_sb_state"] = out.strip() or "unknown"
            state["enabled"] = "enabled" in out.lower()
        else:
            # Fall back to reading the EFI variable directly
            var = glob.glob("/sys/firmware/efi/efivars/SecureBoot-*")
            if var:
                try:
                    data = Path(var[0]).read_bytes()
                    state["enabled"] = bool(data and data[-1] == 1)
                    state["efivar_raw"] = data.hex()
                except OSError as exc:
                    state["error"] = f"efivar read failed: {exc}"
            else:
                state["error"] = "no mokutil and no SecureBoot efivar"
    elif platform.system() == "Windows":
        rc, out, err = run(
            ["powershell", "-NoProfile", "-Command", "Confirm-SecureBootUEFI"]
        )
        state["confirm_secureboot"] = out.strip()
        state["enabled"] = out.strip().lower() == "true"
    return state


def dbx_status() -> dict:
    info = {}
    if shutil.which("dbxtool"):
        rc, out, err = run(["dbxtool", "--list"])
        # Each revocation is one line; count non-empty lines as an approximation
        lines = [l for l in out.splitlines() if l.strip()]
        info["dbxtool_entries"] = len(lines)
        info["sample"] = lines[:5]
        if len(lines) < 50:
            info["warning"] = "low dbx entry count; platform may be behind on revocations"
    elif shutil.which("efi-readvar"):
        rc, out, err = run(["efi-readvar", "-v", "dbx"])
        info["efi_readvar_dbx_excerpt"] = out[:800]
    else:
        info["error"] = "neither dbxtool nor efi-readvar available"
    return info


def hash_esp_binaries() -> list[dict]:
    results = []
    base = next((p for p in ESP_PATHS if os.path.isdir(p)), None)
    if not base:
        return [{"error": "no ESP path found (run as root with EFI mounted)"}]
    for path in Path(base).rglob("*.efi"):
        try:
            digest = hashlib.sha256(path.read_bytes()).hexdigest()
            results.append({"file": str(path), "sha256": digest, "size": path.stat().st_size})
        except OSError as exc:
            results.append({"file": str(path), "error": str(exc)})
    return results


def chipsec_secureboot() -> dict:
    if not shutil.which("chipsec_main"):
        return {"error": "chipsec_main not installed"}
    rc, out, err = run(["chipsec_main", "-m", "common.secureboot.variables"], timeout=600)
    verdict = "PASSED" if "PASSED" in out else "FAILED" if "FAILED" in out else "UNKNOWN"
    return {"verdict": verdict, "tail": out[-1200:]}


def main() -> int:
    ap = argparse.ArgumentParser(description="Secure Boot / bootkit assessment")
    ap.add_argument("--check-chipsec", action="store_true", help="Run CHIPSEC secureboot module (root)")
    ap.add_argument("--output", help="Write JSON report")
    args = ap.parse_args()

    if platform.system() == "Linux" and os.geteuid() != 0:
        print("[warn] not running as root; some checks may be incomplete", file=sys.stderr)

    report = {
        "secure_boot": secure_boot_state(),
        "dbx": dbx_status(),
        "esp_binaries": hash_esp_binaries(),
    }
    if args.check_chipsec:
        report["chipsec_secureboot_variables"] = chipsec_secureboot()

    sb = report["secure_boot"]
    print(f"[+] Secure Boot enabled: {sb.get('enabled')}")
    if not sb.get("enabled"):
        print("[!] Secure Boot is NOT enabled — platform is unprotected against bootkits")
    if "warning" in report["dbx"]:
        print(f"[!] dbx: {report['dbx']['warning']}")
    print(f"[+] {len([b for b in report['esp_binaries'] if 'sha256' in b])} EFI binaries hashed")
    if args.check_chipsec:
        print(f"[+] CHIPSEC secureboot.variables: {report['chipsec_secureboot_variables'].get('verdict')}")

    if args.output:
        Path(args.output).write_text(json.dumps(report, indent=2), encoding="utf-8")
        print(f"[+] report written to {args.output}")
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/hunting-bootkits-in-efi-system-partition/SKILL.md =====
---
name: hunting-bootkits-in-efi-system-partition
description: Baseline the EFI System Partition and hunt malicious EFI binaries such as ESPecter, BlackLotus, Bootkitty, and Glupteba by mounting the ESP, hashing and verifying boot loaders, scanning with YARA, and flagging anomalous non-EFI files or out-of-band bootloader changes. Use during proactive threat hunts for firmware/bootkit persistence (MITRE ATT&CK T1542.003) or when investigating suspected UEFI-level compromise that survives OS reinstall.
domain: cybersecurity
subdomain: hardware-firmware-security
tags:
- bootkit
- uefi
- efi-system-partition
- secure-boot
- firmware-forensics
- threat-hunting
- yara
- measured-boot
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
mitre_attack:
- T1542.003
---
# Hunting Bootkits in the EFI System Partition

## Overview

The EFI System Partition (ESP) is a small FAT32-formatted partition that the platform firmware reads at power-on to locate and execute the operating system's boot loader. Because it executes *before* the operating system, kernel, and any EDR agent, the ESP is one of the most coveted persistence locations for advanced adversaries. UEFI bootkits that live on the ESP survive OS reinstallation, disk reformatting of the OS partition, and most endpoint defenses.

This skill follows the detection research published by **Eclypsium** ("Enhanced Threat Detection: Bootloaders, Bootkits, and Secure Boot", and the *Bootkitty* and *Glupteba* analyses) and aligns with the ESP-hunting methodology Rapid7 documented for Velociraptor (`Windows.Forensics.UEFI`, `Windows.Detection.Yara.UEFI`). The threat context is concrete and current:

- **ESPecter** (ESET, 2021) — a UEFI bootkit that persists on the ESP as a patched Windows Boot Manager (`bootmgfw.efi`) plus malicious kernel-mode drivers. It marked the move of UEFI threats *from SPI flash to the ESP*, where they are far easier to deploy.
- **BlackLotus** (2023) — the first in-the-wild UEFI bootkit able to bypass Secure Boot on fully patched Windows 11 by exploiting CVE-2022-21894 ("baton drop") and staging a vulnerable signed `bootmgfw.efi` on the ESP.
- **Bootkitty** (2024) — the first UEFI bootkit targeting Linux, dropped onto the ESP.
- **Glupteba** — commodity malware whose UEFI variant replaces software on the EFI partition.

The core detection insight from Eclypsium and Rapid7 is that the bootloader normally changes **only** during a vendor or OS update; an out-of-band change to ESP binaries, an unsigned or untrusted-signed boot loader, or any file in the ESP root that is *not* under the `EFI/` directory is a high-fidelity indicator of compromise. This skill builds that baseline and hunts deviations from it.

## When to Use

- During proactive threat hunts for firmware/bootkit persistence (MITRE ATT&CK T1542.003 — Pre-OS Boot: Bootkit)
- After an incident where an adversary achieved SYSTEM/root and may have established below-OS persistence
- When validating Secure Boot posture across a fleet and confirming boot-chain integrity
- As a periodic integrity check, comparing the current ESP contents against a trusted golden baseline
- When triaging hosts that show measured-boot (TPM PCR) mismatches

## Prerequisites

- Administrative/root access to the target host (mounting and reading the ESP requires elevation)
- Linux analysis tooling — install on Debian/Ubuntu:
  ```bash
  sudo apt-get update
  sudo apt-get install -y sbsigntool pesign efitools efibootmgr binwalk yara
  pip install pefile
  ```
- `UEFITool` for inspecting firmware/binaries (download from https://github.com/LongSoft/UEFITool/releases)
- A trusted **golden baseline** of EFI binary hashes for the OS/vendor versions in scope (build it once on a known-clean, freshly imaged host)
- For Windows targets: WinPE or a forensic boot environment, or Velociraptor with the `Windows.Forensics.UEFI` artifact

## Objectives

- Mount and enumerate the ESP read-only without altering evidence
- Inventory every EFI binary and compute cryptographic hashes
- Verify Secure Boot signatures on each boot loader against trusted keys
- Compare contents against a golden baseline to surface out-of-band changes
- YARA-scan ESP binaries for known bootkit signatures
- Flag anomalies: files outside `EFI/`, unsigned loaders, modified `bootmgfw.efi`/`grubx64.efi`, and tampered boot entries
- Confirm measured-boot (TPM PCR 0/2/4) integrity where TPM is present

## MITRE ATT&CK Mapping

| ID | Name | Relevance |
|----|------|-----------|
| T1542.003 | Pre-OS Boot: Bootkit | Adversaries place malicious boot loaders on the ESP to execute before the OS and EDR, achieving stealthy, resilient persistence. This skill hunts exactly that artifact. |

## Workflow

### Step 1: Identify and mount the ESP read-only
The ESP is a FAT32 partition, usually flagged `EF00` (GPT) or `esp,boot`. Locate it and mount read-only to preserve evidence.
```bash
# Identify the ESP (type code EF00 / "EFI System")
sudo lsblk -o NAME,FSTYPE,PARTTYPENAME,MOUNTPOINT
sudo fdisk -l | grep -i "EFI System"

# Mount the ESP read-only (replace /dev/sda1 with the identified partition)
sudo mkdir -p /mnt/esp
sudo mount -o ro,umask=077 /dev/sda1 /mnt/esp

# Confirm the canonical layout: EFI/ should be the only top-level dir
ls -la /mnt/esp
```

### Step 2: Detect anomalous top-level entries (high-fidelity hunt)
Per Rapid7/Velociraptor guidance, the ESP root should contain **only** the `EFI/` directory (and possibly a vendor `System Volume Information`). Anything else is suspicious.
```bash
# Any path in the ESP root NOT under EFI/ is a red flag
find /mnt/esp -maxdepth 1 -mindepth 1 ! -name 'EFI' ! -iname 'System Volume Information'

# Hunt for boot binaries dropped outside expected vendor folders
find /mnt/esp -type f \( -iname '*.efi' -o -iname '*.sys' -o -iname '*.dll' \) -printf '%p\t%s bytes\t%TY-%Tm-%Td\n'
```

### Step 3: Inventory and hash every EFI binary
Compute SHA-256 of all boot binaries for baseline comparison and threat-intel lookup.
```bash
# Recursively hash all EFI/PE binaries on the ESP
find /mnt/esp -type f \( -iname '*.efi' -o -iname '*.sys' \) -print0 \
  | xargs -0 sha256sum | tee /tmp/esp_hashes.txt

# Quick triage on the primary loaders
sha256sum /mnt/esp/EFI/Microsoft/Boot/bootmgfw.efi 2>/dev/null
sha256sum /mnt/esp/EFI/Boot/bootx64.efi 2>/dev/null
sha256sum /mnt/esp/EFI/*/grubx64.efi /mnt/esp/EFI/*/shimx64.efi 2>/dev/null
```
Submit unknown hashes to VirusTotal / the LOLDrivers and Binarly catalogs to identify known-vulnerable or malicious loaders (e.g., the BlackLotus-abused `bootmgfw.efi` builds).

### Step 4: Verify Secure Boot signatures on each loader
A legitimate loader is signed by Microsoft UEFI CA (Windows/shim) or the distro vendor. Use `sbverify` to list/verify signatures and `pesign` for the certificate chain.
```bash
# List embedded signatures on a loader
sbverify --list /mnt/esp/EFI/Microsoft/Boot/bootmgfw.efi

# Verify against the platform's db certificate (export it first)
sudo efi-readvar -v db -o /tmp/db.esl   # dump Secure Boot db
sbverify --cert /path/to/MicrosoftUEFICA.pem /mnt/esp/EFI/Boot/bootx64.efi

# Inspect the PE certificate chain
pesign -S -i /mnt/esp/EFI/Microsoft/Boot/bootmgfw.efi
```
An **unsigned** loader, a loader signed by an unexpected/self-signed certificate, or a known-vulnerable signed binary staged by an attacker (BlackLotus technique) is a confirmed finding.

### Step 5: Compare against the golden baseline
Diff the live ESP hash inventory against a trusted baseline captured from a clean image of the same OS/vendor build.
```bash
# Sort both inventories on the hash column and diff
awk '{print $1}' /tmp/esp_hashes.txt | sort > /tmp/live.sha256
sort baseline_esp_hashes.sha256 > /tmp/base.sha256

# Hashes present live but absent from the baseline = unexpected/new binaries
comm -23 /tmp/live.sha256 /tmp/base.sha256
```
Because the bootloader changes only during legitimate updates, any new hash that does not correspond to a known patch is an actionable lead.

### Step 6: YARA-scan ESP binaries for bootkit signatures
Run YARA rules for known bootkits (ESPecter, BlackLotus, Bootkitty, CosmicStrand) across the ESP — the same approach as Velociraptor's `Windows.Detection.Yara.UEFI`.
```bash
# Scan all ESP files recursively with a bootkit ruleset
yara -r -w bootkit_rules.yar /mnt/esp/

# Example: scan only the binaries collected in Step 3
yara -r bootkit_rules.yar /mnt/esp/EFI/
```
Source rules from the YARA-Rules project, Eclypsium/Binarly publications, and ESET's bootkit reports.

### Step 7: Inspect and validate UEFI boot entries
ESPecter-class bootkits manipulate the boot order/entries. Enumerate NVRAM boot variables and confirm each points to an expected, signed loader.
```bash
# List boot entries and the current order (Bootkitty/ESPecter tamper here)
sudo efibootmgr -v

# Confirm Secure Boot is enabled and look for revoked hashes in dbx
mokutil --sb-state
sudo efi-readvar -v dbx -o /tmp/dbx.esl
```
A boot entry referencing a file outside `\EFI\` or a loader whose hash appears in `dbx` (revoked) but is still present indicates tampering.

### Step 8: Validate measured boot against TPM PCRs (where available)
Measured boot records the boot-chain components into TPM PCRs (notably PCR 0, 2, 4). A bootkit that alters loaders changes these measurements.
```bash
# Read PCRs that cover firmware and the boot loader
sudo tpm2_pcrread sha256:0,2,4,7

# Parse the TCG event log to see what was measured into each PCR
sudo tpm2_eventlog /sys/kernel/security/tpm0/binary_bios_measurements
```
Compare measured values against the host's known-good attestation baseline; an unexplained PCR 4 change correlates with a modified boot loader.

### Step 9: Document findings and preserve evidence
```bash
# Make a forensic image of the ESP for offline analysis before remediation
sudo dd if=/dev/sda1 of=/evidence/esp_$(hostname)_$(date +%F).img bs=4M conv=noerror,sync
sha256sum /evidence/esp_*.img > /evidence/esp_image.sha256
sudo umount /mnt/esp
```
Record every anomalous binary (path, hash, signature status, baseline diff result, YARA match) in the case file before any cleanup.

## Tools and Resources

| Tool | Purpose | Source |
|------|---------|--------|
| sbsigntool (`sbverify`) | List/verify Secure Boot signatures on EFI binaries | https://git.kernel.org/pub/scm/linux/kernel/git/jejb/sbsigntools.git |
| pesign | Inspect PE/COFF signatures and certificate chains | https://github.com/rhboot/pesign |
| efitools (`efi-readvar`) | Dump Secure Boot variables (db/dbx/KEK/PK) | https://git.kernel.org/pub/scm/linux/kernel/git/jejb/efitools.git |
| efibootmgr / mokutil | Enumerate boot entries and Secure Boot state | https://github.com/rhboot/efibootmgr |
| UEFITool | Parse and inspect UEFI binaries/firmware | https://github.com/LongSoft/UEFITool |
| YARA | Signature-scan ESP binaries for bootkits | https://github.com/VirusTotal/yara |
| tpm2-tools | Read PCRs and TCG event log for measured boot | https://github.com/tpm2-software/tpm2-tools |
| Velociraptor `Windows.Forensics.UEFI` | Scale ESP hunting across a fleet | https://docs.velociraptor.app/ |
| Eclypsium bootkit research | Threat context and detection methodology | https://eclypsium.com/blog/threat-detection-bootloaders-bootkits-secureboot/ |
| Rapid7 UEFI hunting blog | ESP hunting with Velociraptor artifacts | https://www.rapid7.com/blog/post/2024/02/29/how-to-hunt-for-uefi-malware-using-velociraptor/ |

## Known Bootkit Indicators

| Bootkit | ESP Artifact / Behavior | Reference |
|---------|-------------------------|-----------|
| ESPecter | Patched `bootmgfw.efi` + kernel drivers on ESP | ESET WeLiveSecurity 2021 |
| BlackLotus | Vulnerable signed `bootmgfw.efi` staged to bypass Secure Boot (CVE-2022-21894) | ESET 2023 |
| Bootkitty | Malicious EFI loader on ESP targeting Linux | ESET / Eclypsium 2024 |
| Glupteba (UEFI) | Replaces software on the EFI partition | Eclypsium |
| CosmicStrand | UEFI firmware implant hooking the boot chain | Eclypsium / Kaspersky |

## Validation Criteria

- [ ] ESP located and mounted read-only without modifying evidence
- [ ] Top-level ESP contents enumerated; any non-`EFI/` entries flagged
- [ ] All EFI binaries inventoried with SHA-256 hashes
- [ ] Secure Boot signatures verified on every boot loader
- [ ] Live inventory diffed against a trusted golden baseline
- [ ] YARA bootkit ruleset run across the ESP with no unexplained matches
- [ ] Boot entries (`efibootmgr`) validated; no references outside `\EFI\`
- [ ] Secure Boot confirmed enabled; no present binary matches a `dbx`-revoked hash
- [ ] TPM PCR 0/2/4 measurements compared to baseline (where TPM present)
- [ ] Forensic ESP image preserved and all findings documented

===== END FILE =====


===== FILE: skills/hunting-bootkits-in-efi-system-partition/LICENSE =====

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


===== FILE: skills/hunting-bootkits-in-efi-system-partition/references/api-reference.md =====
# Command Reference — ESP Bootkit Hunting

## ESP discovery and mounting

| Command | Purpose |
|---------|---------|
| `lsblk -o NAME,FSTYPE,PARTTYPENAME,MOUNTPOINT` | List partitions with type names to find the ESP |
| `fdisk -l \| grep -i "EFI System"` | Identify the ESP partition device |
| `mount -o ro,umask=077 /dev/sdXN /mnt/esp` | Mount the ESP read-only (evidence-safe) |
| `umount /mnt/esp` | Unmount after analysis |

## sbverify (sbsigntool) — Secure Boot signatures

| Command | Purpose |
|---------|---------|
| `sbverify --list <binary.efi>` | List all embedded signatures on an EFI binary |
| `sbverify --cert <ca.pem> <binary.efi>` | Verify a binary against a known-good signing certificate |
| `sbverify --detached <sig> <binary.efi>` | Verify a detached signature |

## pesign — PE/COFF signature inspection

| Command | Purpose |
|---------|---------|
| `pesign -S -i <binary.efi>` | Show signatures / certificate chain on a PE binary |
| `pesign -h -i <binary.efi>` | Print the PE authenticode hash |

## efitools / efibootmgr / mokutil — Secure Boot state

| Command | Purpose |
|---------|---------|
| `efi-readvar -v db -o db.esl` | Dump the Secure Boot allow-list (db) |
| `efi-readvar -v dbx -o dbx.esl` | Dump the Secure Boot revocation list (dbx) |
| `efibootmgr -v` | List boot entries and boot order with loader paths |
| `mokutil --sb-state` | Report whether Secure Boot is enabled |
| `mokutil --list-enrolled` | List enrolled Machine Owner Keys |

## YARA — bootkit scanning

| Command | Purpose |
|---------|---------|
| `yara -r -w <rules.yar> /mnt/esp/` | Recursively scan the ESP, suppress warnings |
| `yara -r -s <rules.yar> /mnt/esp/EFI/` | Scan with matched-string output |

## tpm2-tools — measured boot

| Command | Purpose |
|---------|---------|
| `tpm2_pcrread sha256:0,2,4,7` | Read firmware/boot-loader/Secure-Boot PCRs |
| `tpm2_eventlog /sys/kernel/security/tpm0/binary_bios_measurements` | Parse the TCG measured-boot event log |

## Hashing and baseline diff

| Command | Purpose |
|---------|---------|
| `find /mnt/esp -type f \( -iname '*.efi' -o -iname '*.sys' \) -print0 \| xargs -0 sha256sum` | Inventory and hash all boot binaries |
| `comm -23 live.sha256 base.sha256` | Show hashes present live but absent from baseline |
| `dd if=/dev/sdXN of=esp.img bs=4M conv=noerror,sync` | Forensically image the ESP |

## Velociraptor artifacts (fleet hunting)

| Artifact | Purpose |
|----------|---------|
| `Windows.Forensics.UEFI` | Parse the ESP partition table and FAT to enumerate files |
| `Windows.Detection.Yara.UEFI` | YARA-scan ESP contents at scale |
| `Generic.System.EfiSignatures` | Collect EFI signature data |
| `Windows.Forensics.UEFI.BootApplication` | Parse Measured Boot TCG logs |

===== END FILE =====


===== FILE: skills/hunting-bootkits-in-efi-system-partition/references/standards.md =====
# Standards Mapping — Hunting Bootkits in the EFI System Partition

## MITRE ATT&CK

| ID | Name | Rationale |
|----|------|-----------|
| T1542.003 | Pre-OS Boot: Bootkit | Adversaries install malicious boot loaders on the ESP to execute before the OS and security tooling; this skill hunts and validates those ESP boot artifacts. |

## NIST Cybersecurity Framework (CSF 2.0)

| ID | Name | Rationale |
|----|------|-----------|
| DE.CM-01 | Networks and network services are monitored to find potentially adverse events (extended here to host boot-chain integrity monitoring) | Continuous baselining and integrity monitoring of ESP boot binaries detects out-of-band bootkit persistence. |

## Supporting References

- Eclypsium — "Enhanced Threat Detection: Bootloaders, Bootkits, and Secure Boot": https://eclypsium.com/blog/threat-detection-bootloaders-bootkits-secureboot/
- Eclypsium — "Bootkitty and Linux Bootkits": https://eclypsium.com/blog/bootkitty-linux-bootkit/
- ESET — "UEFI threats moving to the ESP: Introducing ESPecter bootkit": https://www.welivesecurity.com/2021/10/05/uefi-threats-moving-esp-introducing-especter-bootkit/
- Rapid7 — "How To Hunt For UEFI Malware Using Velociraptor": https://www.rapid7.com/blog/post/2024/02/29/how-to-hunt-for-uefi-malware-using-velociraptor/
- NSA — "UEFI Secure Boot Customization" guidance
- UEFI Specification — EFI System Partition layout (`\EFI\` directory structure)

===== END FILE =====


===== FILE: skills/hunting-bootkits-in-efi-system-partition/scripts/agent.py =====
#!/usr/bin/env python3
"""
esp_bootkit_hunter.py — Baseline and hunt malicious EFI binaries on the EFI System Partition.

Mounts (or reads an already-mounted) ESP, inventories EFI/PE boot binaries, computes
SHA-256 hashes, verifies Secure Boot signatures with sbverify, flags files outside the
canonical EFI/ directory, diffs against a golden baseline, and optionally YARA-scans.

Real tooling used: sbverify (sbsigntool), yara, sha256 hashing. No placeholders.

Usage:
  sudo python3 esp_bootkit_hunter.py --esp /mnt/esp --baseline baseline.sha256
  sudo python3 esp_bootkit_hunter.py --device /dev/sda1 --mount --yara bootkit.yar
"""
import argparse
import hashlib
import json
import os
import shutil
import subprocess
import sys
import tempfile

EFI_EXTENSIONS = (".efi", ".sys", ".dll")


def run(cmd):
    """Run a command, return (returncode, stdout, stderr)."""
    try:
        p = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
        return p.returncode, p.stdout, p.stderr
    except FileNotFoundError:
        return 127, "", f"binary not found: {cmd[0]}"
    except subprocess.TimeoutExpired:
        return 124, "", "timeout"


def mount_esp(device, mountpoint):
    os.makedirs(mountpoint, exist_ok=True)
    rc, _, err = run(["mount", "-o", "ro,umask=077", device, mountpoint])
    if rc != 0:
        sys.exit(f"[!] Failed to mount {device}: {err.strip()}")
    print(f"[+] Mounted {device} read-only at {mountpoint}")


def sha256_file(path):
    h = hashlib.sha256()
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(65536), b""):
            h.update(chunk)
    return h.hexdigest()


def find_efi_binaries(esp_root):
    found = []
    for dirpath, _, files in os.walk(esp_root):
        for name in files:
            if name.lower().endswith(EFI_EXTENSIONS):
                found.append(os.path.join(dirpath, name))
    return found


def detect_anomalous_root(esp_root):
    """Per Velociraptor/Rapid7: EFI/ should be the only top-level dir on the ESP."""
    allowed = {"efi", "system volume information"}
    anomalies = []
    for entry in os.listdir(esp_root):
        if entry.lower() not in allowed:
            anomalies.append(os.path.join(esp_root, entry))
    return anomalies


def verify_signature(path):
    """Use sbverify --list to confirm the binary carries a Secure Boot signature."""
    if not shutil.which("sbverify"):
        return "sbverify-missing"
    rc, out, _ = run(["sbverify", "--list", path])
    if rc != 0:
        return "UNSIGNED-or-invalid"
    # sbverify --list prints 'signature N' lines for each embedded signature
    return "signed" if "signature" in out.lower() else "UNSIGNED"


def load_baseline(path):
    baseline = set()
    if not path or not os.path.exists(path):
        return baseline
    with open(path) as f:
        for line in f:
            tok = line.strip().split()
            if tok:
                baseline.add(tok[0].lower())
    return baseline


def yara_scan(rules, esp_root):
    if not shutil.which("yara"):
        return "yara-missing"
    rc, out, err = run(["yara", "-r", "-w", rules, esp_root])
    return out.strip() or ("(no matches)" if rc == 0 else err.strip())


def main():
    ap = argparse.ArgumentParser(description="Hunt bootkits on the EFI System Partition.")
    ap.add_argument("--esp", default="/mnt/esp", help="ESP mountpoint")
    ap.add_argument("--device", help="ESP block device to mount (e.g. /dev/sda1)")
    ap.add_argument("--mount", action="store_true", help="Mount --device read-only first")
    ap.add_argument("--baseline", help="Golden baseline sha256 file to diff against")
    ap.add_argument("--yara", help="YARA ruleset to scan the ESP with")
    ap.add_argument("--json", help="Write findings to this JSON path")
    args = ap.parse_args()

    if os.geteuid() != 0:
        print("[!] Warning: ESP access typically requires root.", file=sys.stderr)

    if args.mount:
        if not args.device:
            sys.exit("--mount requires --device")
        mount_esp(args.device, args.esp)

    if not os.path.isdir(args.esp):
        sys.exit(f"[!] ESP path not found: {args.esp}")

    report = {"esp": args.esp, "binaries": [], "root_anomalies": [],
              "baseline_new": [], "yara": None}

    # Step 2: anomalous root entries
    report["root_anomalies"] = detect_anomalous_root(args.esp)
    for a in report["root_anomalies"]:
        print(f"[!] ANOMALY: non-EFI entry in ESP root -> {a}")

    # Steps 3-4: inventory, hash, verify signatures
    baseline = load_baseline(args.baseline)
    for path in find_efi_binaries(args.esp):
        try:
            digest = sha256_file(path)
        except OSError as e:
            print(f"[!] Cannot read {path}: {e}", file=sys.stderr)
            continue
        sig = verify_signature(path)
        new = bool(baseline) and digest.lower() not in baseline
        entry = {"path": path, "sha256": digest, "signature": sig, "new_vs_baseline": new}
        report["binaries"].append(entry)
        flag = ""
        if sig.startswith("UNSIGNED"):
            flag += " [UNSIGNED]"
        if new:
            flag += " [NOT-IN-BASELINE]"
            report["baseline_new"].append(digest)
        print(f"[+] {digest}  {sig:18s} {path}{flag}")

    # Step 6: YARA
    if args.yara:
        report["yara"] = yara_scan(args.yara, args.esp)
        print(f"[+] YARA results:\n{report['yara']}")

    # Summary
    n_unsigned = sum(1 for b in report["binaries"] if b["signature"].startswith("UNSIGNED"))
    print(f"\n[=] {len(report['binaries'])} EFI binaries | "
          f"{n_unsigned} unsigned | {len(report['baseline_new'])} new vs baseline | "
          f"{len(report['root_anomalies'])} root anomalies")

    if args.json:
        with open(args.json, "w") as f:
            json.dump(report, f, indent=2)
        print(f"[+] Wrote findings to {args.json}")

    # Non-zero exit if anything suspicious was found (CI/hunt automation friendly)
    if n_unsigned or report["baseline_new"] or report["root_anomalies"]:
        sys.exit(2)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/validating-tpm-measured-boot-attestation/SKILL.md =====
---
name: validating-tpm-measured-boot-attestation
description: Verifies TPM 2.0 measured-boot integrity and remote attestation with
  tpm2-tools -- reading PCRs (tpm2_pcrread), replaying the boot event log, generating
  and checking signed quotes (tpm2_quote/tpm2_checkquote), and sealing secrets to
  a PCR policy. Use to confirm a system booted trusted firmware/kernel for Zero
  Trust device posture, detect boot-chain tampering via PCR drift, or build a golden-value
  baseline for fleet attestation.
domain: cybersecurity
subdomain: hardware-firmware-security
tags:
- hardware-firmware-security
- tpm
- measured-boot
- remote-attestation
- pcr
- tpm2-tools
- integrity-verification
- trusted-computing
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
mitre_attack:
- T1542
---
# Validating TPM Measured Boot and Attestation

> **Legal Notice:** Perform TPM operations only on systems you own or are authorized to assess. Some operations (clearing the TPM, taking ownership, defining NV indices) are destructive or can lock the device. This skill is for defensive integrity verification and authorized assessment.

## Overview

A Trusted Platform Module (TPM 2.0) is a hardware root of trust that supports **measured boot**: each stage of the boot chain hashes ("measures") the next stage and *extends* that measurement into a Platform Configuration Register (PCR) before handing off control. PCRs cannot be set arbitrarily — they can only be extended (`new = hash(old || measurement)`), so the final PCR value is a tamper-evident summary of everything that executed. The TCG firmware profile assigns specific meanings: UEFI firmware code/config measure into **PCR 0–7** (PCR 7 specifically captures Secure Boot policy), bootloaders measure the kernel into **PCR 8–9**, and Linux IMA measures executables into **PCR 10**.

Two complementary verifications matter. **Local validation** reads current PCRs (`tpm2_pcrread`) and replays the TPM event log (`tpm2_eventlog` on `/sys/kernel/security/tpm0/binary_bios_measurements`) to confirm the recorded measurements reproduce the live PCR values — proving the log is authentic and revealing exactly *what* changed if a value drifts from baseline. **Remote attestation** has the TPM produce a signed quote (`tpm2_quote`) over selected PCRs using an Attestation Key (AK), with a fresh nonce to prevent replay; a verifier then independently checks the signature and PCR digest (`tpm2_checkquote`) and compares against a golden/expected value. This lets a server cryptographically establish that a remote machine booted approved firmware and kernel before granting access, sealing secrets, or admitting it to a Zero Trust network.

This skill provides the full `tpm2-tools` workflow for enrolling an AK, capturing and verifying quotes, replaying event logs, sealing/unsealing data to a PCR policy, and building a golden-value baseline for fleet attestation.

## When to Use

- Establishing that endpoints/servers booted a known-good firmware and kernel before granting access (Zero Trust device posture).
- Detecting boot-chain tampering (bootkits, unauthorized firmware/kernel changes) via PCR drift from a baseline.
- Verifying measured-boot integrity after a Secure Boot or firmware incident.
- Sealing secrets (disk keys, credentials) to a measured-boot state so they only release on a trusted boot.
- Building or auditing a remote-attestation service (e.g., Keylime, custom verifier).

## Prerequisites

- A system with a TPM 2.0 device and the resource manager:
  ```bash
  sudo apt install tpm2-tools tpm2-abrmd        # Debian/Ubuntu
  sudo dnf install tpm2-tools tpm2-abrmd        # Fedora/RHEL
  ```
- Access to the TPM (`/dev/tpm0` / `/dev/tpmrm0`) and the kernel measurement log at `/sys/kernel/security/tpm0/binary_bios_measurements`.
- Root for reading some sysfs entries and for NV/AK operations.
- For remote attestation: a verifier host and a transport for the quote/nonce exchange.

## Objectives

- Read and interpret PCR banks and their TCG-assigned meanings.
- Verify the TPM event log reproduces live PCR values (event-log replay).
- Create an Attestation Key and produce a nonce-bound signed quote.
- Independently verify the quote signature and PCR digest on a verifier.
- Establish golden PCR baselines and detect drift across a fleet.
- Optionally seal/unseal a secret to a measured-boot PCR policy.

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Relevance |
|--------------|----------------|-----------|
| T1542 | Pre-OS Boot | Measured boot detects pre-OS tampering this technique relies on. |
| T1542.001 | Pre-OS Boot: System Firmware | PCR 0–7 drift reveals unauthorized firmware modification. |
| T1542.003 | Pre-OS Boot: Bootkit | Bootloader/kernel measurements (PCR 8–10) expose bootkit changes. |
| T1014 | Rootkit | IMA measurements (PCR 10) and quote verification surface concealed tampering. |
| T1601.001 | Modify System Image: Patch System Image | Attestation against golden values flags unauthorized image patches. |

## Workflow

### 1. Confirm the TPM is present and read its properties
```bash
tpm2_getcap properties-fixed | grep -i manufacturer
tpm2_getcap pcrs                 # list supported PCR banks (sha1, sha256, ...)
```

### 2. Read current PCR values
PCR 7 = Secure Boot policy; PCR 0–7 = firmware; PCR 8–9 = bootloader/kernel; PCR 10 = IMA.
```bash
tpm2_pcrread sha256                       # all sha256 PCRs
tpm2_pcrread sha256:0,1,2,3,4,5,6,7       # firmware + Secure Boot policy
tpm2_pcrread sha256:7                      # Secure Boot policy only
```

### 3. Replay the TPM event log against live PCRs
Confirm the recorded log reproduces the current PCR values (authenticity check).
```bash
tpm2_eventlog /sys/kernel/security/tpm0/binary_bios_measurements > eventlog.yaml
# The YAML includes a "pcrs" section with the calculated values — compare to step 2.
# Any mismatch means the event log and the live PCRs disagree (tampering or stale log).
```

### 4. Create a primary key and an Attestation Key (AK)
The AK signs quotes; its public part is shared with the verifier out-of-band.
```bash
tpm2_createprimary -C e -g sha256 -G rsa -c primary.ctx
tpm2_create -C primary.ctx -G rsa -u ak.pub -r ak.priv \
  -a 'fixedtpm|fixedparent|sensitivedataorigin|userwithauth|restricted|sign'
tpm2_load -C primary.ctx -u ak.pub -r ak.priv -c ak.ctx
tpm2_readpublic -c ak.ctx -o ak.pem -f pem      # export AK public key for the verifier
```

### 5. Produce a nonce-bound quote (attestor side)
The verifier supplies a fresh random nonce to defeat replay.
```bash
NONCE=$(openssl rand -hex 20)
tpm2_quote -c ak.ctx -l sha256:0,1,2,3,4,5,6,7,8,9 \
  -q "$NONCE" -m quote.msg -s quote.sig -o quote.pcrs -g sha256
# Send quote.msg, quote.sig, quote.pcrs (and the nonce) to the verifier.
```

### 6. Verify the quote (verifier side)
Independently validate the signature, nonce, and PCR digest with the AK public key.
```bash
tpm2_checkquote -u ak.pem -m quote.msg -s quote.sig -f quote.pcrs \
  -q "$NONCE" -g sha256
# Exit 0 + matching PCR digest == authentic, fresh, untampered quote.
```

### 7. Build and compare against a golden baseline
Compare attested PCRs to known-good values captured from a trusted reference build.
```bash
# Capture golden values on a trusted reference machine:
tpm2_pcrread sha256:0,7 > golden_pcrs.txt
# On each attested host, diff the quote's PCR section against golden_pcrs.txt.
diff <(grep -A8 'sha256' quote.pcrs) golden_pcrs.txt
```

### 8. Seal a secret to a measured-boot policy (optional)
Bind a secret so the TPM only releases it when PCRs match the trusted state.
```bash
tpm2_createpolicy --policy-pcr -l sha256:7 -L pcr7.policy -f pcr7.dat
echo -n "diskkey" | tpm2_create -C primary.ctx -L pcr7.policy \
  -i - -u sealed.pub -r sealed.priv
tpm2_load -C primary.ctx -u sealed.pub -r sealed.priv -c sealed.ctx
# Unseal succeeds only while PCR 7 matches the sealed policy:
tpm2_unseal -c sealed.ctx -p pcr:sha256:7
```

### 9. Inspect IMA runtime measurements (PCR 10)
If IMA is enabled, the runtime measurement list extends PCR 10.
```bash
tpm2_pcrread sha256:10
head -20 /sys/kernel/security/ima/ascii_runtime_measurements
```

### 10. Run the bundled attestation helper
`agent.py` reads PCRs, replays the event log, optionally produces+verifies a quote, and diffs a baseline.
```bash
sudo python scripts/agent.py --pcrs 0,1,2,3,4,5,6,7 --baseline golden_pcrs.json --output attest.json
```

## Tools and Resources

| Tool | Purpose | Source |
|------|---------|--------|
| tpm2-tools | CLI for all TPM 2.0 operations | https://github.com/tpm2-software/tpm2-tools |
| tpm2-tss | TSS2 stack the tools build on | https://github.com/tpm2-software/tpm2-tss |
| Keylime | Scalable remote attestation framework | https://github.com/keylime/keylime |
| TCG PC Client Platform Firmware Profile | PCR usage specification | https://trustedcomputinggroup.org/ |
| Linux IMA docs | Integrity Measurement Architecture | https://sourceforge.net/p/linux-ima/wiki/Home/ |
| RFC 9683 | Remote integrity verification of TPM devices | https://datatracker.ietf.org/doc/html/rfc9683 |

## Validation Criteria

- [ ] TPM 2.0 presence and supported PCR banks confirmed.
- [ ] Current PCR values read for firmware and Secure Boot policy PCRs.
- [ ] Event log replayed and confirmed to reproduce live PCR values.
- [ ] Attestation Key created and its public key exported to the verifier.
- [ ] Nonce-bound quote produced and independently verified with `tpm2_checkquote`.
- [ ] Attested PCRs compared against a golden baseline; drift flagged.
- [ ] (Optional) Secret sealed/unsealed against a PCR policy successfully.
- [ ] IMA runtime measurements reviewed where enabled.
- [ ] Results documented per host with pass/fail and any drift evidence.

===== END FILE =====


===== FILE: skills/validating-tpm-measured-boot-attestation/LICENSE =====

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


===== FILE: skills/validating-tpm-measured-boot-attestation/references/api-reference.md =====
# tpm2-tools Command Reference

## Discovery
| Command | Description |
|---------|-------------|
| `tpm2_getcap properties-fixed` | TPM manufacturer / fixed properties. |
| `tpm2_getcap pcrs` | Supported PCR banks (sha1, sha256, ...). |
| `tpm2_pcrread sha256` | Read all sha256 PCRs. |
| `tpm2_pcrread sha256:0,7` | Read specific PCRs (here firmware + Secure Boot policy). |

## Event log
| Command | Description |
|---------|-------------|
| `tpm2_eventlog /sys/kernel/security/tpm0/binary_bios_measurements` | Parse + replay the firmware event log; output includes calculated PCRs. |

## Attestation key
| Command | Description |
|---------|-------------|
| `tpm2_createprimary -C e -g sha256 -G rsa -c primary.ctx` | Create an endorsement-hierarchy primary key. |
| `tpm2_create -C primary.ctx -G rsa -u ak.pub -r ak.priv -a 'fixedtpm\|fixedparent\|sensitivedataorigin\|userwithauth\|restricted\|sign'` | Create a restricted signing AK. |
| `tpm2_load -C primary.ctx -u ak.pub -r ak.priv -c ak.ctx` | Load the AK into the TPM. |
| `tpm2_readpublic -c ak.ctx -o ak.pem -f pem` | Export the AK public key (PEM) for the verifier. |

## Quote / verify
| Command | Description |
|---------|-------------|
| `tpm2_quote -c ak.ctx -l sha256:0,1,...,9 -q <nonce> -m quote.msg -s quote.sig -o quote.pcrs -g sha256` | Produce a nonce-bound signed quote. |
| `tpm2_checkquote -u ak.pem -m quote.msg -s quote.sig -f quote.pcrs -q <nonce> -g sha256` | Verify signature, nonce, and PCR digest (verifier side). |

## Sealing to PCR policy
| Command | Description |
|---------|-------------|
| `tpm2_createpolicy --policy-pcr -l sha256:7 -L pcr7.policy -f pcr7.dat` | Build a PCR-bound auth policy. |
| `tpm2_create -C primary.ctx -L pcr7.policy -i - -u sealed.pub -r sealed.priv` | Seal a secret to the policy. |
| `tpm2_unseal -c sealed.ctx -p pcr:sha256:7` | Release the secret only if PCRs match. |

## IMA (PCR 10)
| Command | Description |
|---------|-------------|
| `tpm2_pcrread sha256:10` | Read the IMA aggregate PCR. |
| `cat /sys/kernel/security/ima/ascii_runtime_measurements` | View the IMA runtime measurement list. |

## TCG PCR index meanings (PC Client profile)
| PCR | Measures |
|-----|----------|
| 0 | UEFI firmware code (CRTM, boot block). |
| 1 | UEFI firmware configuration / host platform config. |
| 2–3 | Option ROM code and config. |
| 4 | Boot manager / MBR code. |
| 5 | Boot manager config / GPT. |
| 6 | Platform-specific events. |
| 7 | Secure Boot policy (PK/KEK/db/dbx state). |
| 8–9 | Bootloader-measured kernel/initrd. |
| 10 | Linux IMA runtime measurements. |

===== END FILE =====


===== FILE: skills/validating-tpm-measured-boot-attestation/references/standards.md =====
# Standards and Framework Mapping

## MITRE ATT&CK

| ID | Name | Rationale |
|----|------|-----------|
| T1542 | Pre-OS Boot | Measured boot detects the pre-OS tampering this technique relies on. |
| T1542.001 | Pre-OS Boot: System Firmware | PCR 0–7 drift reveals unauthorized firmware modification. |
| T1542.003 | Pre-OS Boot: Bootkit | Bootloader/kernel measurements (PCR 8–10) expose bootkit changes. |
| T1014 | Rootkit | Quote verification and IMA measurements surface concealed tampering. |
| T1601.001 | Modify System Image: Patch System Image | Attestation against golden values flags unauthorized image patches. |

## NIST Cybersecurity Framework 2.0

| ID | Name | Rationale |
|----|------|-----------|
| PR.PS-01 | Configuration management practices are established and applied | Measured-boot baselining and attestation enforce and verify the approved firmware/kernel configuration of platforms. |

## Supporting Standards and References

- **TCG TPM 2.0 Library Specification.** Defines PCRs, extend semantics, quotes, and attestation keys.
- **TCG PC Client Platform Firmware Profile.** Assigns PCR index meanings (PCR 0–7 firmware, PCR 7 Secure Boot, PCR 8–10 OS/IMA).
- **RFC 9683 — Remote Integrity Verification of Network Devices Containing TPMs.** Standardizes TPM-based remote attestation.
- **NIST SP 800-155 — BIOS Integrity Measurement Guidelines.** Measured-boot and integrity reporting guidance.
- **NSA UEFI/Boot Security guidance.** Recommends measured boot + attestation alongside Secure Boot.

===== END FILE =====


===== FILE: skills/validating-tpm-measured-boot-attestation/scripts/agent.py =====
#!/usr/bin/env python3
"""
TPM 2.0 measured-boot and attestation helper.

Reads PCRs via tpm2-tools, replays the firmware event log, optionally produces
and verifies a nonce-bound quote, and diffs measured PCRs against a golden
baseline JSON. Emits a structured report.

Requires tpm2-tools (tpm2_pcrread, tpm2_eventlog, tpm2_quote, tpm2_checkquote).
Authorized integrity-verification use only.
"""
import argparse
import json
import os
import re
import secrets
import shutil
import subprocess
import sys
import tempfile
from pathlib import Path

EVENTLOG = "/sys/kernel/security/tpm0/binary_bios_measurements"


def run(cmd: list[str], timeout: int = 120) -> tuple[int, str, str]:
    try:
        p = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
        return p.returncode, p.stdout, p.stderr
    except FileNotFoundError:
        return 127, "", f"not found: {cmd[0]}"
    except subprocess.SubprocessError as exc:
        return 1, "", str(exc)


def need(tool: str) -> bool:
    if not shutil.which(tool):
        print(f"[error] required tool missing: {tool}", file=sys.stderr)
        return False
    return True


def read_pcrs(selection: str) -> dict:
    if not need("tpm2_pcrread"):
        return {"error": "tpm2_pcrread missing"}
    rc, out, err = run(["tpm2_pcrread", f"sha256:{selection}"])
    if rc != 0:
        return {"error": err.strip() or "tpm2_pcrread failed", "returncode": rc}
    pcrs = {}
    for line in out.splitlines():
        m = re.match(r"\s*(\d+)\s*:\s*(0x[0-9A-Fa-f]+)", line)
        if m:
            pcrs[m.group(1)] = m.group(2).lower()
    return pcrs


def replay_eventlog() -> dict:
    if not need("tpm2_eventlog"):
        return {"error": "tpm2_eventlog missing"}
    if not os.path.exists(EVENTLOG):
        return {"error": f"event log not found at {EVENTLOG} (run as root?)"}
    rc, out, err = run(["tpm2_eventlog", EVENTLOG])
    if rc != 0:
        return {"error": err.strip() or "tpm2_eventlog failed"}
    # Pull the calculated pcrs section emitted by the tool
    calc = {}
    for line in out.splitlines():
        m = re.match(r"\s*(\d+)\s*:\s*(0x[0-9A-Fa-f]+)", line)
        if m:
            calc[m.group(1)] = m.group(2).lower()
    return {"calculated_pcrs": calc, "event_count": out.count("EventNum")}


def attest(selection: str, workdir: Path) -> dict:
    for tool in ("tpm2_createprimary", "tpm2_create", "tpm2_load", "tpm2_readpublic",
                 "tpm2_quote", "tpm2_checkquote"):
        if not need(tool):
            return {"error": f"{tool} missing"}
    primary = workdir / "primary.ctx"
    akpub, akpriv, akctx, akpem = (workdir / f for f in ("ak.pub", "ak.priv", "ak.ctx", "ak.pem"))
    qmsg, qsig, qpcrs = (workdir / f for f in ("quote.msg", "quote.sig", "quote.pcrs"))
    nonce = secrets.token_hex(20)

    steps = [
        ["tpm2_createprimary", "-C", "e", "-g", "sha256", "-G", "rsa", "-c", str(primary)],
        ["tpm2_create", "-C", str(primary), "-G", "rsa", "-u", str(akpub), "-r", str(akpriv),
         "-a", "fixedtpm|fixedparent|sensitivedataorigin|userwithauth|restricted|sign"],
        ["tpm2_load", "-C", str(primary), "-u", str(akpub), "-r", str(akpriv), "-c", str(akctx)],
        ["tpm2_readpublic", "-c", str(akctx), "-o", str(akpem), "-f", "pem"],
        ["tpm2_quote", "-c", str(akctx), "-l", f"sha256:{selection}", "-q", nonce,
         "-m", str(qmsg), "-s", str(qsig), "-o", str(qpcrs), "-g", "sha256"],
    ]
    for cmd in steps:
        rc, out, err = run(cmd)
        if rc != 0:
            return {"error": f"{cmd[0]} failed: {err.strip()}"}

    rc, out, err = run(["tpm2_checkquote", "-u", str(akpem), "-m", str(qmsg),
                        "-s", str(qsig), "-f", str(qpcrs), "-q", nonce, "-g", "sha256"])
    return {"nonce": nonce, "verified": rc == 0,
            "checkquote_output": (out or err)[:800]}


def diff_baseline(pcrs: dict, baseline_path: str) -> dict:
    try:
        baseline = json.loads(Path(baseline_path).read_text(encoding="utf-8"))
    except (OSError, json.JSONDecodeError) as exc:
        return {"error": f"cannot read baseline: {exc}"}
    drift = {k: {"expected": baseline[k], "actual": pcrs.get(k)}
             for k in baseline if baseline.get(k) != pcrs.get(k)}
    return {"match": not drift, "drift": drift}


def main() -> int:
    ap = argparse.ArgumentParser(description="TPM measured-boot attestation helper")
    ap.add_argument("--pcrs", default="0,1,2,3,4,5,6,7", help="comma-separated PCR indices")
    ap.add_argument("--quote", action="store_true", help="produce + verify an AK quote")
    ap.add_argument("--baseline", help="golden PCR baseline JSON to diff against")
    ap.add_argument("--save-baseline", help="write current PCRs as a baseline JSON")
    ap.add_argument("--output", help="write JSON report")
    args = ap.parse_args()

    report = {"pcr_selection": args.pcrs}
    pcrs = read_pcrs(args.pcrs)
    report["pcrs"] = pcrs
    if "error" not in pcrs:
        for idx, val in sorted(pcrs.items(), key=lambda x: int(x[0])):
            print(f"PCR[{idx}] = {val}")

    report["eventlog"] = replay_eventlog()
    if isinstance(pcrs, dict) and "calculated_pcrs" in report["eventlog"]:
        calc = report["eventlog"]["calculated_pcrs"]
        mismatches = {k: (pcrs.get(k), calc.get(k)) for k in calc if pcrs.get(k) != calc.get(k)}
        report["eventlog_replay_match"] = not mismatches
        if mismatches:
            print(f"[!] event-log replay MISMATCH: {mismatches}")
        else:
            print("[+] event-log replay matches live PCRs")

    if args.save_baseline and "error" not in pcrs:
        Path(args.save_baseline).write_text(json.dumps(pcrs, indent=2), encoding="utf-8")
        print(f"[+] baseline saved to {args.save_baseline}")

    if args.baseline and "error" not in pcrs:
        report["baseline_diff"] = diff_baseline(pcrs, args.baseline)
        print(f"[+] baseline match: {report['baseline_diff'].get('match')}")

    if args.quote:
        with tempfile.TemporaryDirectory() as td:
            report["attestation"] = attest(args.pcrs, Path(td))
        print(f"[+] quote verified: {report['attestation'].get('verified')}")

    if args.output:
        Path(args.output).write_text(json.dumps(report, indent=2), encoding="utf-8")
        print(f"[+] report written to {args.output}")
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====
