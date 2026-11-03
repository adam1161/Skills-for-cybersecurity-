

===== FILE: skills/configuring-host-based-intrusion-detection/SKILL.md =====
---
name: configuring-host-based-intrusion-detection
description: 'Configures host-based intrusion detection systems (HIDS) to monitor
  endpoint file integrity, system calls, and configuration changes for security violations.
  Use when deploying OSSEC, Wazuh, or AIDE for endpoint monitoring, building file
  integrity monitoring (FIM) policies, or meeting compliance requirements for change
  detection. Activates for requests involving HIDS configuration, file integrity monitoring,
  OSSEC/Wazuh deployment, or host-based detection.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- HIDS
- Wazuh
- OSSEC
- file-integrity-monitoring
- intrusion-detection
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1059
- T1543
- T1547
- T1070
- T1055
---
# Configuring Host-Based Intrusion Detection

## When to Use

Use this skill when:
- Deploying HIDS agents (Wazuh, OSSEC, AIDE) across Windows and Linux endpoints
- Configuring file integrity monitoring (FIM) for compliance (PCI DSS 11.5, NIST SI-7)
- Monitoring system configuration changes, rootkit detection, and security policy violations
- Integrating HIDS alerts with SIEM platforms for centralized monitoring

**Do not use** this skill for network-based IDS (Suricata, Snort) or for EDR deployment.

## Prerequisites

- Wazuh server (manager) deployed and accessible from endpoints
- Administrative access to target endpoints
- Network connectivity: agents to Wazuh manager on port 1514 (TCP/UDP) and 1515 (TCP enrollment)
- Wazuh dashboard (OpenSearch Dashboards) for alert visualization
- Understanding of critical files/directories to monitor per OS

## Workflow

### Step 1: Install Wazuh Agent

**Windows**:
```powershell
# Download and install Wazuh agent
Invoke-WebRequest -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.0-1.msi" `
  -OutFile "wazuh-agent.msi"
msiexec /i wazuh-agent.msi /q WAZUH_MANAGER="wazuh-manager.corp.com" `
  WAZUH_REGISTRATION_SERVER="wazuh-manager.corp.com" WAZUH_AGENT_GROUP="windows-workstations"
net start WazuhSvc
```

**Linux (Debian/Ubuntu)**:
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" \
  > /etc/apt/sources.list.d/wazuh.list
apt-get update && apt-get install wazuh-agent -y
sed -i 's/MANAGER_IP/wazuh-manager.corp.com/' /var/ossec/etc/ossec.conf
systemctl daemon-reload && systemctl enable --now wazuh-agent
```

### Step 2: Configure File Integrity Monitoring (FIM)

Edit agent configuration (`/var/ossec/etc/ossec.conf` or `C:\Program Files (x86)\ossec-agent\ossec.conf`):

```xml
<syscheck>
  <!-- Scan frequency: every 12 hours -->
  <frequency>43200</frequency>
  <scan_on_start>yes</scan_on_start>
  <alert_new_files>yes</alert_new_files>

  <!-- Linux critical directories -->
  <directories check_all="yes" realtime="yes">/etc</directories>
  <directories check_all="yes" realtime="yes">/usr/bin</directories>
  <directories check_all="yes" realtime="yes">/usr/sbin</directories>
  <directories check_all="yes" realtime="yes">/bin</directories>
  <directories check_all="yes" realtime="yes">/sbin</directories>
  <directories check_all="yes">/boot</directories>

  <!-- Windows critical directories -->
  <directories check_all="yes" realtime="yes">C:\Windows\System32</directories>
  <directories check_all="yes" realtime="yes">C:\Windows\SysWOW64</directories>
  <directories check_all="yes" realtime="yes">%PROGRAMFILES%</directories>

  <!-- Windows registry monitoring -->
  <windows_registry>HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run</windows_registry>
  <windows_registry>HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce</windows_registry>
  <windows_registry>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services</windows_registry>

  <!-- Ignore frequently changing files -->
  <ignore>/etc/mtab</ignore>
  <ignore>/etc/resolv.conf</ignore>
  <ignore type="sregex">.log$</ignore>
</syscheck>
```

### Step 3: Configure Rootkit Detection

```xml
<rootcheck>
  <disabled>no</disabled>
  <frequency>43200</frequency>
  <rootkit_files>/var/ossec/etc/shared/rootkit_files.txt</rootkit_files>
  <rootkit_trojans>/var/ossec/etc/shared/rootkit_trojans.txt</rootkit_trojans>
  <system_audit>/var/ossec/etc/shared/system_audit_rcl.txt</system_audit>
  <check_dev>yes</check_dev>
  <check_files>yes</check_files>
  <check_if>yes</check_if>
  <check_pids>yes</check_pids>
  <check_ports>yes</check_ports>
  <check_sys>yes</check_sys>
  <check_trojans>yes</check_trojans>
  <check_unixaudit>yes</check_unixaudit>
</rootcheck>
```

### Step 4: Configure Log Analysis Rules

```xml
<!-- Custom rules in /var/ossec/etc/rules/local_rules.xml -->
<group name="local,syscheck,">
  <!-- Alert on critical binary modifications -->
  <rule id="100001" level="12">
    <if_sid>550</if_sid>
    <match>/usr/bin/|/usr/sbin/|/bin/|/sbin/</match>
    <description>Critical system binary modified: $(file)</description>
    <group>syscheck,pci_dss_11.5,</group>
  </rule>

  <!-- Alert on new executable in temp directories -->
  <rule id="100002" level="10">
    <if_sid>554</if_sid>
    <match>/tmp/|/var/tmp/</match>
    <description>New file created in temp directory: $(file)</description>
    <group>syscheck,malware,</group>
  </rule>

  <!-- Alert on SSH configuration changes -->
  <rule id="100003" level="10">
    <if_sid>550</if_sid>
    <match>/etc/ssh/sshd_config</match>
    <description>SSH configuration modified</description>
    <group>syscheck,authentication,</group>
  </rule>
</group>
```

### Step 5: Configure Active Response

```xml
<!-- Auto-block IP after repeated authentication failures -->
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5712</rules_id>
  <timeout>600</timeout>
</active-response>

<!-- Disable account after brute force detection -->
<active-response>
  <disabled>no</disabled>
  <command>disable-account</command>
  <location>local</location>
  <rules_id>100100</rules_id>
  <timeout>3600</timeout>
</active-response>
```

### Step 6: Integrate with SIEM

```
# Wazuh to Splunk via Filebeat
# Edit /etc/filebeat/filebeat.yml:
filebeat.inputs:
  - type: log
    paths:
      - /var/ossec/logs/alerts/alerts.json
    json.keys_under_root: true
output.elasticsearch:
  hosts: ["https://splunk-hec:8088"]

# Wazuh to Elastic via direct integration
# Wazuh indexer feeds directly into OpenSearch/Elasticsearch
# Dashboard: https://wazuh-dashboard:5601
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **HIDS** | Host-based Intrusion Detection System; monitors individual endpoints for malicious activity |
| **FIM** | File Integrity Monitoring; detects unauthorized changes to files by comparing cryptographic hashes |
| **Syscheck** | Wazuh/OSSEC module for file integrity monitoring and registry monitoring |
| **Rootcheck** | Wazuh/OSSEC module for rootkit and malware detection |
| **Active Response** | Automated defensive action triggered by HIDS alert (IP block, account disable) |
| **CDB List** | Constant Database list used for custom lookups in Wazuh rules |

## Tools & Systems

- **Wazuh**: Open-source HIDS platform (fork of OSSEC) with manager, agent, and dashboard
- **OSSEC**: Original open-source HIDS (predecessor to Wazuh)
- **AIDE (Advanced Intrusion Detection Environment)**: Standalone file integrity checker for Linux
- **Tripwire**: Commercial file integrity monitoring solution
- **Samhain**: Open-source HIDS focused on file integrity and log monitoring

## Common Pitfalls

- **Monitoring too many directories**: FIM on entire filesystems generates excessive alerts. Focus on critical system binaries, configuration files, and web roots.
- **Not excluding noisy files**: Frequently changing files (logs, temp, caches) generate false positive FIM alerts. Maintain exclusion lists.
- **Ignoring baseline establishment**: First FIM scan creates a baseline. Changes detected before baseline stabilization are noise, not threats. Allow 48 hours for baseline.
- **Active response without testing**: Auto-blocking IPs or disabling accounts can cause outages. Test active response rules in a non-production environment first.
- **Agent enrollment failures**: Agents must successfully enroll with the manager before monitoring begins. Verify firewall rules allow port 1514 and 1515 traffic.

===== END FILE =====


===== FILE: skills/configuring-host-based-intrusion-detection/LICENSE =====

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


===== FILE: skills/configuring-host-based-intrusion-detection/assets/template.md =====
# HIDS Deployment Template

## Deployment Information

| Field | Value |
|-------|-------|
| HIDS Platform | Wazuh / OSSEC / AIDE |
| Manager Address | |
| Agent Version | |
| Target Endpoints | |
| Deployment Date | |

## FIM Configuration

| Directory | Real-time | Check All | Exclusions |
|-----------|----------|-----------|------------|
| /etc | Yes | Yes | mtab, resolv.conf |
| /usr/bin | Yes | Yes | |
| /usr/sbin | Yes | Yes | |
| C:\Windows\System32 | Yes | Yes | *.log |

## Monitoring Modules

| Module | Status | Frequency |
|--------|--------|-----------|
| Syscheck (FIM) | Enabled | 12 hours |
| Rootcheck | Enabled | 12 hours |
| Log Analysis | Enabled | Real-time |
| Active Response | Enabled | Real-time |
| Vulnerability Detection | Enabled | 12 hours |

## Custom Rules

| Rule ID | Description | Level | Trigger |
|---------|-------------|-------|---------|
| | | | |

## Sign-Off

| Role | Name | Date |
|------|------|------|
| Security Engineer | | |
| SOC Analyst | | |

===== END FILE =====


===== FILE: skills/configuring-host-based-intrusion-detection/references/api-reference.md =====
# Host-Based Intrusion Detection — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| requests | `pip install requests` | Wazuh REST API client |
| osquery | Binary install | SQL-based host inspection |
| hashlib | stdlib | File integrity hash computation |

## Wazuh API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/security/user/authenticate` | Obtain JWT token |
| GET | `/agents` | List managed agents |
| GET | `/agents/{id}` | Agent details |
| GET | `/sca/{agent_id}` | Security Configuration Assessment results |
| GET | `/rootcheck/{agent_id}` | Rootkit check results |
| GET | `/alerts` | Query security alerts |
| GET | `/rules` | List detection rules |

## Key osquery Tables

| Table | Description |
|-------|-------------|
| `processes` | Running processes with user, path, cmdline |
| `listening_ports` | Open network ports and bound processes |
| `users` | System user accounts |
| `file` | File metadata and hashes |
| `suid_bin` | SUID/SGID binaries |
| `crontab` | Scheduled cron jobs |

## OSSEC Rule IDs

| Rule ID Range | Category |
|---------------|----------|
| 500-599 | File integrity monitoring |
| 5700-5799 | SSH authentication |
| 18100-18199 | Linux audit events |
| 31100-31199 | Web attack detection |

## External References

- [Wazuh API Reference](https://documentation.wazuh.com/current/user-manual/api/reference.html)
- [osquery Schema](https://osquery.io/schema/)
- [OSSEC Rule Syntax](https://www.ossec.net/docs/syntax/head_rules.html)

===== END FILE =====


===== FILE: skills/configuring-host-based-intrusion-detection/references/standards.md =====
# Standards & References - Configuring Host-Based Intrusion Detection

## Primary Standards

### NIST SP 800-94 Rev 1 - Guide to Intrusion Detection and Prevention Systems
- **Publisher**: NIST
- **Scope**: Architecture, deployment, and management of IDPS including host-based systems

### PCI DSS 4.0 Requirement 11.5 - File Integrity Monitoring
- **Publisher**: PCI SSC
- **Requirement**: Deploy FIM to alert on unauthorized modification of critical files
- **Scope**: System files, configuration files, content files on in-scope systems

### CIS Control 3 - Data Protection (v8)
- **Publisher**: CIS
- **Relevance**: Sub-control 3.14 requires monitoring for unauthorized changes to sensitive data

## Compliance Mappings

| Framework | Requirement | HIDS Coverage |
|-----------|------------|--------------|
| PCI DSS 4.0 | 11.5.2 - FIM mechanism deployed | Wazuh syscheck module |
| NIST 800-53 | SI-7 Software, Firmware, and Information Integrity | File integrity monitoring |
| NIST 800-53 | SI-4 System Monitoring | HIDS log analysis and alerting |
| HIPAA | 164.312(b) - Audit controls | File access and change monitoring |
| ISO 27001 | A.12.4.1 - Event logging | HIDS event collection and analysis |

## Supporting References

- **Wazuh Documentation**: https://documentation.wazuh.com/
- **OSSEC Documentation**: https://www.ossec.net/docs/
- **Wazuh FIM Reference**: https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/
- **Wazuh Ruleset**: https://github.com/wazuh/wazuh-ruleset

===== END FILE =====


===== FILE: skills/configuring-host-based-intrusion-detection/references/workflows.md =====
# Workflows - Configuring Host-Based Intrusion Detection

## Workflow 1: Wazuh HIDS Deployment

```
[Deploy Wazuh Manager]
    │
    ▼
[Configure FIM, rootcheck, and log analysis modules]
    │
    ▼
[Deploy agents to pilot endpoints]
    │
    ▼
[Establish baseline (48 hours)]
    │
    ▼
[Tune rules: suppress false positives, add exclusions]
    │
    ▼
[Deploy agents to production fleet]
    │
    ▼
[Integrate with SIEM]
    │
    ▼
[Create dashboards and alert workflows]
```

## Workflow 2: FIM Alert Investigation

```
[FIM alert: File modified]
    │
    ▼
[Check file path and change details]
    │
    ├── Known system update ──► [Correlate with patch window, close alert]
    ├── Authorized config change ──► [Verify change ticket, close alert]
    └── Unauthorized change ──► [Investigate]
                                     │
                                     ├── Determine who/what changed the file
                                     ├── Review process tree and timeline
                                     │
                                     ├── Malicious ──► [Escalate to IR]
                                     └── Operational ──► [Update change process]
```

===== END FILE =====


===== FILE: skills/configuring-host-based-intrusion-detection/scripts/agent.py =====
#!/usr/bin/env python3
"""Host-based intrusion detection agent using OSSEC/Wazuh API and osquery."""

import json
import os
import sys
import argparse
import subprocess
from datetime import datetime

try:
    import requests
    requests.packages.urllib3.disable_warnings()
except ImportError:
    print("Install: pip install requests")
    sys.exit(1)


class WazuhClient:
    """Wazuh API client for HIDS management."""

    def __init__(self, base_url, username, password):
        self.url = base_url.rstrip("/")
        self.token = self._authenticate(username, password)

    def _authenticate(self, username, password):
        resp = requests.post(f"{self.url}/security/user/authenticate",
                             auth=(username, password),
                             verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true", timeout=30)  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        resp.raise_for_status()
        return resp.json()["data"]["token"]

    def _get(self, endpoint, params=None):
        resp = requests.get(f"{self.url}/{endpoint}",
                            headers={"Authorization": f"Bearer {self.token}"},
                            params=params,
                            verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true", timeout=30)  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        resp.raise_for_status()
        return resp.json()

    def list_agents(self, status=None):
        params = {}
        if status:
            params["status"] = status
        return self._get("agents", params)

    def get_agent_alerts(self, agent_id, limit=20):
        return self._get(f"alerts", {"agent.id": agent_id, "limit": limit})

    def get_sca_results(self, agent_id):
        return self._get(f"sca/{agent_id}")

    def get_rootcheck(self, agent_id):
        return self._get(f"rootcheck/{agent_id}")


def run_osquery_check(query):
    """Execute osquery for host inspection."""
    cmd = ["osqueryi", "--json", query]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        return json.loads(result.stdout) if result.stdout.strip() else []
    except (FileNotFoundError, json.JSONDecodeError):
        return [{"error": "osquery not available"}]


def check_file_integrity(paths):
    """Check file integrity for key system files."""
    checks = []
    import hashlib, os
    for path in paths:
        if os.path.exists(path):
            with open(path, "rb") as f:
                sha256 = hashlib.sha256(f.read()).hexdigest()
            stat = os.stat(path)
            checks.append({
                "path": path,
                "sha256": sha256,
                "size": stat.st_size,
                "modified": datetime.fromtimestamp(stat.st_mtime).isoformat(),
                "status": "present",
            })
        else:
            checks.append({"path": path, "status": "missing", "severity": "HIGH"})
    return checks


def run_audit(wazuh_url=None, username=None, password=None, agent_id=None):
    """Execute HIDS audit."""
    print(f"\n{'='*60}")
    print(f"  HOST-BASED INTRUSION DETECTION AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    if wazuh_url and username and password:
        client = WazuhClient(wazuh_url, username, password)
        agents = client.list_agents()
        data = agents.get("data", {})
        items = data.get("affected_items", [])
        print(f"--- WAZUH AGENTS ({data.get('total_affected_items', 0)}) ---")
        for a in items[:10]:
            print(f"  {a.get('name', 'N/A')} ({a.get('id', '')}): {a.get('status', '')}")

        if agent_id:
            sca = client.get_sca_results(agent_id)
            sca_data = sca.get("data", {}).get("affected_items", [])
            print(f"\n--- SCA RESULTS (agent {agent_id}) ---")
            for s in sca_data[:5]:
                print(f"  {s.get('name', '')}: pass={s.get('pass', 0)} fail={s.get('fail', 0)}")

    system_files = ["/etc/passwd", "/etc/shadow", "/etc/sudoers", "/etc/ssh/sshd_config"]
    integrity = check_file_integrity(system_files)
    print(f"\n--- FILE INTEGRITY CHECK ---")
    for f in integrity:
        print(f"  {f['path']}: {f['status']}")

    processes = run_osquery_check("SELECT name, pid, uid FROM processes WHERE uid = 0")
    print(f"\n--- ROOT PROCESSES ({len(processes)}) ---")
    for p in processes[:10]:
        if "error" not in p:
            print(f"  PID {p.get('pid', '')}: {p.get('name', '')}")

    return {"integrity": integrity, "processes": processes}


def main():
    parser = argparse.ArgumentParser(description="HIDS Audit Agent")
    parser.add_argument("--wazuh-url", help="Wazuh API URL (https://host:55000)")
    parser.add_argument("--username", help="Wazuh API username")
    parser.add_argument("--password", help="Wazuh API password")
    parser.add_argument("--agent-id", help="Specific agent ID to audit")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_audit(args.wazuh_url, args.username, args.password, args.agent_id)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-host-based-intrusion-detection/scripts/process.py =====
#!/usr/bin/env python3
"""
HIDS Alert Analyzer

Parses Wazuh/OSSEC alerts JSON and generates summary reports for
file integrity monitoring and intrusion detection events.
"""

import json
import sys
import os
from collections import defaultdict, Counter
from datetime import datetime


def parse_wazuh_alerts(json_path: str) -> list:
    """Parse Wazuh alerts JSON file (one JSON object per line)."""
    alerts = []

    with open(json_path, "r", encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            try:
                alert = json.loads(line)
                alerts.append({
                    "timestamp": alert.get("timestamp", ""),
                    "rule_id": alert.get("rule", {}).get("id", ""),
                    "rule_description": alert.get("rule", {}).get("description", ""),
                    "rule_level": alert.get("rule", {}).get("level", 0),
                    "rule_groups": alert.get("rule", {}).get("groups", []),
                    "agent_name": alert.get("agent", {}).get("name", ""),
                    "agent_ip": alert.get("agent", {}).get("ip", ""),
                    "syscheck_path": alert.get("syscheck", {}).get("path", ""),
                    "syscheck_event": alert.get("syscheck", {}).get("event", ""),
                    "syscheck_md5_after": alert.get("syscheck", {}).get("md5_after", ""),
                    "src_ip": alert.get("data", {}).get("srcip", ""),
                    "full_log": alert.get("full_log", "")[:300],
                })
            except json.JSONDecodeError:
                continue

    return alerts


def analyze_alerts(alerts: list) -> dict:
    """Analyze parsed alerts for patterns and summary statistics."""
    analysis = {
        "total_alerts": len(alerts),
        "by_level": Counter(),
        "by_rule": Counter(),
        "by_agent": Counter(),
        "by_group": Counter(),
        "fim_events": {
            "modified": 0,
            "added": 0,
            "deleted": 0,
            "top_modified_files": Counter(),
        },
        "high_severity": [],
        "attack_sources": Counter(),
    }

    for alert in alerts:
        level = alert["rule_level"]
        analysis["by_level"][level] += 1
        analysis["by_rule"][f"{alert['rule_id']}: {alert['rule_description']}"] += 1
        analysis["by_agent"][alert["agent_name"]] += 1

        for group in alert["rule_groups"]:
            analysis["by_group"][group] += 1

        if "syscheck" in alert["rule_groups"] or alert["syscheck_path"]:
            event = alert["syscheck_event"]
            if event == "modified":
                analysis["fim_events"]["modified"] += 1
                analysis["fim_events"]["top_modified_files"][alert["syscheck_path"]] += 1
            elif event == "added":
                analysis["fim_events"]["added"] += 1
            elif event == "deleted":
                analysis["fim_events"]["deleted"] += 1

        if level >= 10:
            analysis["high_severity"].append({
                "timestamp": alert["timestamp"],
                "agent": alert["agent_name"],
                "rule": alert["rule_description"],
                "level": level,
                "detail": alert["full_log"],
            })

        if alert["src_ip"]:
            analysis["attack_sources"][alert["src_ip"]] += 1

    return analysis


def generate_report(analysis: dict, output_path: str) -> None:
    """Generate HIDS alert analysis report."""
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "total_alerts": analysis["total_alerts"],
        "severity_distribution": dict(analysis["by_level"]),
        "top_rules": dict(analysis["by_rule"].most_common(20)),
        "top_agents": dict(analysis["by_agent"].most_common(20)),
        "alert_groups": dict(analysis["by_group"].most_common(15)),
        "file_integrity": {
            "files_modified": analysis["fim_events"]["modified"],
            "files_added": analysis["fim_events"]["added"],
            "files_deleted": analysis["fim_events"]["deleted"],
            "top_modified": dict(analysis["fim_events"]["top_modified_files"].most_common(20)),
        },
        "high_severity_alerts": analysis["high_severity"][:50],
        "top_attack_sources": dict(analysis["attack_sources"].most_common(20)),
    }

    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <wazuh_alerts.json>")
        print()
        print("Analyzes Wazuh/OSSEC alerts JSON for HIDS event patterns.")
        sys.exit(1)

    json_path = sys.argv[1]
    if not os.path.exists(json_path):
        print(f"Error: File not found: {json_path}")
        sys.exit(1)

    print("Parsing Wazuh alerts...")
    alerts = parse_wazuh_alerts(json_path)
    print(f"Parsed {len(alerts)} alerts")

    print("Analyzing alert patterns...")
    analysis = analyze_alerts(alerts)

    base = os.path.splitext(os.path.basename(json_path))[0]
    out_dir = os.path.dirname(json_path) or "."
    report_path = os.path.join(out_dir, f"{base}_analysis.json")
    generate_report(analysis, report_path)
    print(f"Analysis report: {report_path}")

    print(f"\n--- HIDS Alert Summary ---")
    print(f"Total alerts: {analysis['total_alerts']}")
    print(f"High severity (level >= 10): {len(analysis['high_severity'])}")
    print(f"FIM: {analysis['fim_events']['modified']} modified, "
          f"{analysis['fim_events']['added']} added, "
          f"{analysis['fim_events']['deleted']} deleted")

===== END FILE =====


===== FILE: skills/configuring-windows-defender-advanced-settings/SKILL.md =====
---
name: configuring-windows-defender-advanced-settings
description: 'Configures Microsoft Defender for Endpoint (MDE) advanced protection
  settings including attack surface reduction rules, controlled folder access, network
  protection, and exploit protection. Use when hardening Windows endpoints beyond
  default Defender settings, deploying enterprise-grade endpoint protection, or meeting
  compliance requirements for advanced malware defense. Activates for requests involving
  Windows Defender configuration, ASR rules, MDE tuning, or Microsoft endpoint security.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- windows-security
- Microsoft-Defender
- ASR
- exploit-protection
- MDE
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1685
- T1204.002
- T1059.001
- T1055
- T1547.001
---
# Configuring Windows Defender Advanced Settings

## When to Use

Use this skill when:
- Configuring Microsoft Defender for Endpoint (MDE) beyond default settings for enhanced protection
- Implementing Attack Surface Reduction (ASR) rules to block common attack techniques
- Enabling controlled folder access for ransomware protection
- Configuring network protection and exploit protection features
- Deploying Defender settings via Intune, SCCM, or Group Policy at enterprise scale

**Do not use** this skill for third-party EDR deployment (CrowdStrike, SentinelOne) or for Microsoft Defender for Cloud (Azure workload protection).

## Prerequisites

- Windows 10/11 Enterprise with Microsoft Defender Antivirus enabled
- Microsoft 365 E5 or Microsoft Defender for Endpoint Plan 2 license (for full MDE features)
- Microsoft Intune or SCCM for enterprise policy deployment
- Microsoft 365 Defender portal access (security.microsoft.com)
- Endpoints not running third-party AV in active mode (Defender enters passive mode)

## Workflow

### Step 1: Configure Attack Surface Reduction (ASR) Rules

ASR rules block specific behaviors commonly used by malware and attackers:

```powershell
# Enable ASR rules via PowerShell (or deploy via Intune/GPO)
# Mode: 0=Disabled, 1=Block, 2=Audit, 6=Warn

# Block executable content from email client and webmail
Set-MpPreference -AttackSurfaceReductionRules_Ids BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550 `
  -AttackSurfaceReductionRules_Actions 1

# Block all Office applications from creating child processes
Set-MpPreference -AttackSurfaceReductionRules_Ids D4F940AB-401B-4EFC-AADC-AD5F3C50688A `
  -AttackSurfaceReductionRules_Actions 1

# Block Office applications from creating executable content
Set-MpPreference -AttackSurfaceReductionRules_Ids 3B576869-A4EC-4529-8536-B80A7769E899 `
  -AttackSurfaceReductionRules_Actions 1

# Block Office applications from injecting code into other processes
Set-MpPreference -AttackSurfaceReductionRules_Ids 75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84 `
  -AttackSurfaceReductionRules_Actions 1

# Block JavaScript or VBScript from launching downloaded executable content
Set-MpPreference -AttackSurfaceReductionRules_Ids D3E037E1-3EB8-44C8-A917-57927947596D `
  -AttackSurfaceReductionRules_Actions 1

# Block execution of potentially obfuscated scripts
Set-MpPreference -AttackSurfaceReductionRules_Ids 5BEB7EFE-FD9A-4556-801D-275E5FFC04CC `
  -AttackSurfaceReductionRules_Actions 1

# Block Win32 API calls from Office macros
Set-MpPreference -AttackSurfaceReductionRules_Ids 92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B `
  -AttackSurfaceReductionRules_Actions 1

# Block credential stealing from Windows LSASS
Set-MpPreference -AttackSurfaceReductionRules_Ids 9E6C4E1F-7D60-472F-BA1A-A39EF669E4B2 `
  -AttackSurfaceReductionRules_Actions 1

# Block process creations from PSExec and WMI commands
Set-MpPreference -AttackSurfaceReductionRules_Ids D1E49AAC-8F56-4280-B9BA-993A6D77406C `
  -AttackSurfaceReductionRules_Actions 1

# Block untrusted and unsigned processes from USB
Set-MpPreference -AttackSurfaceReductionRules_Ids B2B3F03D-6A65-4F7B-A9C7-1C7EF74A9BA4 `
  -AttackSurfaceReductionRules_Actions 1

# Block persistence through WMI event subscription
Set-MpPreference -AttackSurfaceReductionRules_Ids E6DB77E5-3DF2-4CF1-B95A-636979351E5B `
  -AttackSurfaceReductionRules_Actions 1

# Block abuse of exploited vulnerable signed drivers
Set-MpPreference -AttackSurfaceReductionRules_Ids 56A863A9-875E-4185-98A7-B882C64B5CE5 `
  -AttackSurfaceReductionRules_Actions 1
```

### Step 2: Configure Controlled Folder Access (Ransomware Protection)

```powershell
# Enable Controlled Folder Access
Set-MpPreference -EnableControlledFolderAccess Enabled

# Default protected folders: Documents, Pictures, Videos, Music, Desktop, Favorites
# Add custom protected folders
Add-MpPreference -ControlledFolderAccessProtectedFolders "C:\CriticalData"
Add-MpPreference -ControlledFolderAccessProtectedFolders "D:\SharedDrives"

# Allow specific applications to access protected folders
Add-MpPreference -ControlledFolderAccessAllowedApplications "C:\Program Files\CustomApp\app.exe"
Add-MpPreference -ControlledFolderAccessAllowedApplications "C:\Program Files\Backup\backup.exe"

# Set to Audit mode first to identify legitimate applications that need access
Set-MpPreference -EnableControlledFolderAccess AuditMode
# Event ID 1124 in Microsoft-Windows-Windows Defender/Operational log
```

### Step 3: Configure Network Protection

```powershell
# Enable Network Protection (blocks connections to malicious domains/IPs)
Set-MpPreference -EnableNetworkProtection Enabled

# Network Protection leverages Microsoft SmartScreen intelligence
# Blocks: phishing sites, exploit hosting domains, C2 domains, malware download URLs

# Set to Audit mode first:
Set-MpPreference -EnableNetworkProtection AuditMode
# Event Log: Microsoft-Windows-Windows Defender/Operational, Event ID 1125

# Configure Web Content Filtering (requires MDE P2 license)
# Managed via Microsoft 365 Defender portal:
# Settings → Endpoints → Web content filtering → Add policy
# Categories to block: Malware, Phishing, Adult content, High bandwidth
```

### Step 4: Configure Exploit Protection

```powershell
# Export current exploit protection settings
Get-ProcessMitigation -RegistryConfigFilePath "C:\Defender\current_mitigations.xml"

# Configure system-level mitigations
Set-ProcessMitigation -System -Enable DEP, SEHOP, ForceRelocateImages, BottomUp

# Configure per-application mitigations
# Example: Harden Microsoft Office against exploitation
Set-ProcessMitigation -Name "WINWORD.EXE" `
  -Enable DEP, SEHOP, ForceRelocateImages, CFG, StrictHandle

Set-ProcessMitigation -Name "EXCEL.EXE" `
  -Enable DEP, SEHOP, ForceRelocateImages, CFG, StrictHandle

Set-ProcessMitigation -Name "POWERPNT.EXE" `
  -Enable DEP, SEHOP, ForceRelocateImages, CFG, StrictHandle

# Import exploit protection configuration from XML template
Set-ProcessMitigation -PolicyFilePath "C:\Defender\exploit_protection_template.xml"
```

### Step 5: Configure Cloud-Delivered Protection

```powershell
# Enable cloud-delivered protection (real-time threat intelligence)
Set-MpPreference -MAPSReporting Advanced
Set-MpPreference -SubmitSamplesConsent SendAllSamples

# Enable Block at First Sight (BAFS)
# Requires: Cloud protection enabled + sample submission enabled
Set-MpPreference -DisableBlockAtFirstSeen $false

# Set cloud block timeout to maximum (60 seconds)
Set-MpPreference -CloudBlockLevel High
Set-MpPreference -CloudExtendedTimeout 50

# Enable potentially unwanted application (PUA) protection
Set-MpPreference -PUAProtection Enabled
```

### Step 6: Configure Scan and Update Settings

```powershell
# Configure real-time protection
Set-MpPreference -DisableRealtimeMonitoring $false
Set-MpPreference -DisableBehaviorMonitoring $false
Set-MpPreference -DisableIOAVProtection $false
Set-MpPreference -DisableScriptScanning $false

# Configure scheduled scan
Set-MpPreference -ScanScheduleQuickScanTime 12:00:00
Set-MpPreference -ScanParameters QuickScan
Set-MpPreference -ScanScheduleDay 0  # Every day
Set-MpPreference -RemediationScheduleDay 0

# Configure signature updates
Set-MpPreference -SignatureUpdateInterval 1  # Check every hour
Set-MpPreference -SignatureFallbackOrder "MicrosoftUpdateServer|MMPC"

# Enable tamper protection (prevents unauthorized changes to Defender settings)
# Managed via Microsoft 365 Defender portal:
# Settings → Endpoints → Advanced features → Tamper Protection: On
```

### Step 7: Deploy via Intune (Enterprise)

```
Intune Deployment Path:
1. Endpoint Security → Attack Surface Reduction → Create Profile
   - Platform: Windows 10 and later
   - Profile: Attack surface reduction rules
   - Configure each ASR rule to Block or Audit

2. Endpoint Security → Antivirus → Create Profile
   - Microsoft Defender Antivirus
   - Configure: Cloud protection, PUA, real-time protection

3. Endpoint Security → Antivirus → Create Profile
   - Microsoft Defender Antivirus Exclusions
   - Add path/process/extension exclusions for LOB apps

4. Devices → Configuration profiles → Create profile
   - Endpoint protection → Microsoft Defender Exploit Guard
   - Configure: Controlled Folder Access, Network Protection
```

### Step 8: Monitor in Microsoft 365 Defender Portal

```
Dashboard monitoring:
1. security.microsoft.com → Reports → Endpoints
   - Device health: Protection status across fleet
   - ASR rule detections: Which rules are triggering
   - Vulnerable devices: Missing security updates

2. Threat analytics:
   - Active threat campaigns and Defender coverage
   - Recommended security actions

3. Advanced hunting (KQL):
   DeviceEvents
   | where ActionType startswith "Asr"
   | summarize Count=count() by ActionType, FileName
   | sort by Count desc

   DeviceEvents
   | where ActionType == "ControlledFolderAccessViolationBlocked"
   | project Timestamp, DeviceName, FileName, FolderPath
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **ASR Rules** | Attack Surface Reduction rules that block specific high-risk behaviors at the endpoint level |
| **Controlled Folder Access** | Ransomware protection feature that prevents unauthorized applications from modifying files in protected folders |
| **Network Protection** | Blocks outbound connections to low-reputation or known-malicious domains using SmartScreen intelligence |
| **Exploit Protection** | System and per-application memory mitigations (DEP, ASLR, CFG) to prevent exploitation |
| **BAFS (Block at First Sight)** | Cloud-based zero-day protection that holds suspicious files for cloud analysis before allowing execution |
| **Tamper Protection** | Prevents unauthorized changes to Defender security settings, even by local administrators |

## Tools & Systems

- **Microsoft 365 Defender Portal**: security.microsoft.com for centralized management and reporting
- **Microsoft Intune**: Cloud-based endpoint management for Defender policy deployment
- **PowerShell (Set-MpPreference)**: Local configuration of Defender settings
- **WDAC (Windows Defender Application Control)**: Complementary application control technology
- **Microsoft Defender for Endpoint API**: REST API for automation and custom integrations

## Common Pitfalls

- **Enabling all ASR rules in Block mode immediately**: Some ASR rules cause false positives with legitimate software (Office macros, admin scripts). Always deploy in Audit mode first and monitor for 2-4 weeks.
- **Not configuring Controlled Folder Access exclusions**: Backup software, database applications, and development tools may be blocked from writing to protected folders. Add exclusions proactively.
- **Ignoring tamper protection**: Without tamper protection, malware or insiders can disable Defender via PowerShell or registry edits. Enable tamper protection through the M365 Defender portal.
- **Running Defender alongside third-party AV**: Defender enters passive mode when third-party AV is present. Ensure you are using the intended AV solution and configure Defender appropriately (EDR-only mode if keeping third-party AV).
- **Forgetting cloud connectivity requirements**: Cloud-delivered protection and BAFS require endpoints to reach Microsoft cloud services. Verify proxy/firewall rules allow Defender cloud traffic.

===== END FILE =====


===== FILE: skills/configuring-windows-defender-advanced-settings/LICENSE =====

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


===== FILE: skills/configuring-windows-defender-advanced-settings/assets/template.md =====
# Windows Defender Advanced Configuration Template

## Environment Information

| Field | Value |
|-------|-------|
| Endpoint OS | Windows 10/11 Enterprise |
| MDE License | E5 / Defender P2 / Standalone |
| Deployment Method | Intune / SCCM / GPO |
| Configuration Date | |
| Configured By | |

## ASR Rule Configuration

| GUID | Rule Name | Mode | Exclusions |
|------|----------|------|------------|
| BE9BA2D9-... | Block executable content from email | Block / Audit | |
| D4F940AB-... | Block Office child processes | Block / Audit | |
| 3B576869-... | Block Office executable creation | Block / Audit | |
| 75668C1F-... | Block Office code injection | Block / Audit | |
| D3E037E1-... | Block JS/VBS launching executables | Block / Audit | |
| 5BEB7EFE-... | Block obfuscated scripts | Block / Audit | |
| 92E97FA1-... | Block Win32 API from macros | Block / Audit | |
| 9E6C4E1F-... | Block LSASS credential stealing | Block / Audit | |
| D1E49AAC-... | Block PSExec/WMI processes | Block / Audit | |
| B2B3F03D-... | Block untrusted USB processes | Block / Audit | |
| E6DB77E5-... | Block WMI persistence | Block / Audit | |
| 56A863A9-... | Block vulnerable signed drivers | Block / Audit | |

## Controlled Folder Access

| Setting | Value |
|---------|-------|
| Mode | Enabled / Audit |
| Protected Folders | Default + custom |
| Allowed Applications | |

### Custom Protected Folders

| Path | Justification |
|------|--------------|
| | |

### Allowed Applications

| Application Path | Reason |
|-----------------|--------|
| | |

## Network Protection

| Setting | Value |
|---------|-------|
| Network Protection | Enabled / Audit |
| Web Content Filtering | Enabled / Disabled |
| Blocked Categories | |

## Cloud Protection

| Setting | Value |
|---------|-------|
| MAPS Reporting | Advanced |
| Sample Submission | SendAllSamples |
| Block at First Sight | Enabled |
| Cloud Block Level | High |
| Cloud Timeout | 50 seconds |
| Tamper Protection | On |

## Exclusions Register

| Type | Value | Reason | Approved By | Review Date |
|------|-------|--------|-------------|-------------|
| Path | | | | |
| Process | | | | |
| Extension | | | | |

## Validation Checklist

- [ ] All ASR rules active (Block or Audit)
- [ ] Controlled Folder Access enabled
- [ ] Network Protection enabled
- [ ] Cloud protection set to Advanced
- [ ] Tamper Protection enabled in M365 portal
- [ ] Signature update interval set to hourly
- [ ] PUA protection enabled
- [ ] No excessive exclusions configured

## Sign-Off

| Role | Name | Date |
|------|------|------|
| Security Engineer | | |
| Endpoint Admin | | |

===== END FILE =====


===== FILE: skills/configuring-windows-defender-advanced-settings/references/api-reference.md =====
# Windows Defender Advanced Settings — API Reference

## PowerShell Cmdlets

| Cmdlet | Description |
|--------|-------------|
| `Get-MpComputerStatus` | Defender service status, signature version |
| `Get-MpPreference` | All Defender configuration preferences |
| `Set-MpPreference` | Modify Defender settings |
| `Get-MpThreatDetection` | Recent threat detections |
| `Add-MpPreference -AttackSurfaceReductionRules_Ids` | Enable ASR rules |

## Critical ASR Rule GUIDs

| GUID | Rule |
|------|------|
| `be9ba2d9-53ea-4cdc-84e5-9b1eeee46550` | Block executable from email |
| `d4f940ab-401b-4efc-aadc-ad5f3c50688a` | Block Office child processes |
| `3b576869-a4ec-4529-8536-b80a7769e899` | Block Office executable creation |
| `5beb7efe-fd9a-4556-801d-275e5ffc04cc` | Block obfuscated scripts |
| `56a863a9-875e-4185-98a7-b882c64b5ce5` | Block exploited signed drivers |

## ASR Rule Actions

| Value | Action |
|-------|--------|
| 0 | Disabled |
| 1 | Block |
| 2 | Audit |
| 6 | Warn |

## External References

- [Microsoft ASR Rules Reference](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-reference)
- [Defender PowerShell Reference](https://learn.microsoft.com/en-us/powershell/module/defender/)
- [Tamper Protection](https://learn.microsoft.com/en-us/defender-endpoint/prevent-changes-to-security-settings-with-tamper-protection)

===== END FILE =====


===== FILE: skills/configuring-windows-defender-advanced-settings/references/standards.md =====
# Standards & References - Configuring Windows Defender Advanced Settings

## Primary Standards

### Microsoft Defender for Endpoint Documentation
- **Publisher**: Microsoft
- **URL**: https://learn.microsoft.com/en-us/defender-endpoint/
- **Scope**: Complete reference for MDE deployment, configuration, and management
- **Key sections**: ASR rules, exploit protection, network protection, controlled folder access

### MITRE ATT&CK Mapping for ASR Rules
- **Relevance**: Each ASR rule maps to specific ATT&CK techniques it mitigates
- **Key mappings**:
  - Block Office child processes → T1566.001 (Spearphishing Attachment)
  - Block credential stealing from LSASS → T1003.001 (OS Credential Dumping: LSASS)
  - Block PSExec/WMI → T1047 (WMI), T1570 (Lateral Tool Transfer)
  - Block WMI persistence → T1546.003 (WMI Event Subscription)

### CIS Microsoft Windows 11 Benchmark - Section 18.10
- **Publisher**: CIS
- **Relevance**: CIS benchmark sections covering Windows Defender configuration settings

## Compliance Mappings

| Framework | Requirement | MDE Coverage |
|-----------|------------|-------------|
| PCI DSS 4.0 | 5.2 - Anti-malware mechanisms | Defender AV + ASR rules + cloud protection |
| PCI DSS 4.0 | 5.3.1 - Anti-malware kept current | Automatic signature and engine updates |
| NIST 800-53 | SI-3 Malicious Code Protection | Defender real-time protection + BAFS |
| NIST 800-53 | SI-4 System Monitoring | MDE telemetry + advanced hunting |
| HIPAA | 164.308(a)(5)(ii)(B) - Malware protection | Defender AV + ransomware protection |
| ISO 27001 | A.12.2.1 - Controls against malware | Full MDE stack |
| ACSC E8 | Configure Microsoft Office macro settings | ASR rules for Office macro blocking |

## ASR Rule Reference

| GUID | Rule Name | Recommended Mode |
|------|----------|-----------------|
| BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550 | Block executable content from email | Block |
| D4F940AB-401B-4EFC-AADC-AD5F3C50688A | Block Office child processes | Block |
| 3B576869-A4EC-4529-8536-B80A7769E899 | Block Office from creating executables | Block |
| 75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84 | Block Office from injecting into processes | Block |
| D3E037E1-3EB8-44C8-A917-57927947596D | Block JS/VBS launching executables | Block |
| 5BEB7EFE-FD9A-4556-801D-275E5FFC04CC | Block obfuscated scripts | Audit first |
| 92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B | Block Win32 API calls from macros | Block |
| 9E6C4E1F-7D60-472F-BA1A-A39EF669E4B2 | Block credential stealing from LSASS | Block |
| D1E49AAC-8F56-4280-B9BA-993A6D77406C | Block PSExec/WMI process creation | Audit first |
| B2B3F03D-6A65-4F7B-A9C7-1C7EF74A9BA4 | Block untrusted USB processes | Block |
| E6DB77E5-3DF2-4CF1-B95A-636979351E5B | Block WMI event subscription persistence | Block |
| 56A863A9-875E-4185-98A7-B882C64B5CE5 | Block vulnerable signed drivers | Block |

## Supporting References

- **ASR Rules Reference**: https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-reference
- **Controlled Folder Access**: https://learn.microsoft.com/en-us/defender-endpoint/controlled-folders
- **Network Protection**: https://learn.microsoft.com/en-us/defender-endpoint/network-protection
- **Exploit Protection**: https://learn.microsoft.com/en-us/defender-endpoint/exploit-protection

===== END FILE =====


===== FILE: skills/configuring-windows-defender-advanced-settings/references/workflows.md =====
# Workflows - Configuring Windows Defender Advanced Settings

## Workflow 1: ASR Rule Deployment

```
[Identify ASR rules to deploy]
    │
    ▼
[Deploy all rules in Audit mode via Intune/GPO]
    │
    ▼
[Monitor ASR audit events for 2-4 weeks]
    │
    ├── Review events in M365 Defender portal
    ├── Identify false positives per rule
    │
    ▼
[Create exclusions for legitimate applications]
    │
    ▼
[Switch low-risk rules to Block mode]
    │  (Office rules, email content, USB)
    │
    ▼
[Monitor for 1 week]
    │
    ├── No issues ──► [Switch remaining rules to Block mode]
    │
    └── Issues found ──► [Add exclusions, maintain Audit mode for affected rules]
                              │
                              ▼
                         [Re-evaluate after 2 weeks]
```

## Workflow 2: Controlled Folder Access Deployment

```
[Enable Controlled Folder Access in Audit mode]
    │
    ▼
[Monitor Event ID 1124 for blocked write attempts]
    │
    ▼
[Categorize blocked applications]
    │
    ├── Legitimate business app ──► [Add to allowed applications list]
    │
    ├── Backup/sync software ──► [Add to allowed applications list]
    │
    └── Unknown/suspicious ──► [Investigate, potentially malicious]
    │
    ▼
[Switch to Enabled (Block) mode]
    │
    ▼
[Add custom protected folders beyond defaults]
    │
    ▼
[Ongoing monitoring via M365 Defender dashboard]
```

## Workflow 3: Defender Configuration Audit

```
[Quarterly Defender Configuration Review]
    │
    ▼
[Export current Defender settings from all endpoints]
    │
    ├── PowerShell: Get-MpPreference | Export-Clixml
    ├── Intune: Endpoint security reports
    │
    ▼
[Compare against security baseline]
    │
    ├── All settings match baseline ──► [Document compliance, next review]
    │
    └── Drift detected ──► [Investigate cause]
                                │
                                ├── Unauthorized change ──► [Security incident, restore settings]
                                │
                                └── Authorized exception ──► [Document, update baseline]
```

## Workflow 4: False Positive Handling

```
[User reports blocked application]
    │
    ▼
[Identify which Defender feature blocked it]
    │
    ├── ASR rule ──► [Check ASR event log for specific rule GUID]
    │                     │
    │                     ▼
    │                [Create ASR exclusion for file/folder/process]
    │
    ├── Controlled Folder ──► [Add application to allowed list]
    │
    ├── Network Protection ──► [Review URL/domain, submit false positive to Microsoft]
    │
    └── Real-time AV ──► [Submit file for analysis, create AV exclusion if clean]
    │
    ▼
[Deploy exclusion via Intune/GPO]
    │
    ▼
[Verify application works, document exclusion]
```

===== END FILE =====


===== FILE: skills/configuring-windows-defender-advanced-settings/scripts/agent.py =====
#!/usr/bin/env python3
"""Windows Defender advanced configuration audit agent."""

import json
import argparse
import subprocess
from datetime import datetime


def get_defender_status():
    """Get Windows Defender status via PowerShell."""
    cmd = ["powershell", "-Command", "Get-MpComputerStatus | ConvertTo-Json"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        return json.loads(result.stdout) if result.stdout.strip() else {"error": "No output"}
    except (FileNotFoundError, json.JSONDecodeError, subprocess.TimeoutExpired) as e:
        return {"error": str(e)}


def get_defender_preferences():
    """Get Windows Defender preference settings."""
    cmd = ["powershell", "-Command", "Get-MpPreference | ConvertTo-Json"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        return json.loads(result.stdout) if result.stdout.strip() else {"error": "No output"}
    except (FileNotFoundError, json.JSONDecodeError, subprocess.TimeoutExpired) as e:
        return {"error": str(e)}


def audit_asr_rules():
    """Audit Attack Surface Reduction rules configuration."""
    cmd = ["powershell", "-Command",
           "Get-MpPreference | Select-Object -ExpandProperty AttackSurfaceReductionRules_Ids | ConvertTo-Json"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        rule_ids = json.loads(result.stdout) if result.stdout.strip() else []
    except (FileNotFoundError, json.JSONDecodeError):
        rule_ids = []
    critical_asr_rules = {
        "be9ba2d9-53ea-4cdc-84e5-9b1eeee46550": "Block executable content from email and webmail",
        "d4f940ab-401b-4efc-aadc-ad5f3c50688a": "Block all Office applications from creating child processes",
        "3b576869-a4ec-4529-8536-b80a7769e899": "Block Office applications from creating executable content",
        "75668c1f-73b5-4cf0-bb93-3ecf5cb7cc84": "Block Office applications from injecting code",
        "d3e037e1-3eb8-44c8-a917-57927947596d": "Block JavaScript or VBScript from launching downloaded content",
        "5beb7efe-fd9a-4556-801d-275e5ffc04cc": "Block execution of potentially obfuscated scripts",
        "92e97fa1-2edf-4476-bdd6-9dd0b4dddc7b": "Block Win32 API calls from Office macros",
        "56a863a9-875e-4185-98a7-b882c64b5ce5": "Block abuse of exploited vulnerable signed drivers",
    }
    configured = set(rule_ids) if isinstance(rule_ids, list) else set()
    missing = []
    for rule_id, desc in critical_asr_rules.items():
        if rule_id not in configured:
            missing.append({"rule_id": rule_id, "description": desc, "severity": "HIGH"})
    return {"configured_count": len(configured), "missing_critical": missing}


def check_tamper_protection():
    """Check tamper protection status."""
    cmd = ["powershell", "-Command",
           "(Get-MpComputerStatus).IsTamperProtected"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=15)
        enabled = "true" in result.stdout.strip().lower()
        return {"tamper_protection": enabled,
                "severity": "CRITICAL" if not enabled else "INFO"}
    except (FileNotFoundError, subprocess.TimeoutExpired):
        return {"error": "Cannot check tamper protection"}


def run_audit():
    """Execute Windows Defender audit."""
    print(f"\n{'='*60}")
    print(f"  WINDOWS DEFENDER ADVANCED SETTINGS AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    status = get_defender_status()
    print(f"--- DEFENDER STATUS ---")
    if "error" not in status:
        print(f"  Real-time protection: {status.get('RealTimeProtectionEnabled', 'N/A')}")
        print(f"  Behavior monitoring: {status.get('BehaviorMonitorEnabled', 'N/A')}")
        print(f"  Cloud protection: {status.get('OnAccessProtectionEnabled', 'N/A')}")
        print(f"  Signature version: {status.get('AntivirusSignatureVersion', 'N/A')}")

    asr = audit_asr_rules()
    print(f"\n--- ASR RULES ---")
    print(f"  Configured: {asr['configured_count']}")
    print(f"  Missing critical: {len(asr['missing_critical'])}")
    for rule in asr["missing_critical"][:5]:
        print(f"    [{rule['severity']}] {rule['description']}")

    tamper = check_tamper_protection()
    print(f"\n--- TAMPER PROTECTION ---")
    print(f"  Enabled: {tamper.get('tamper_protection', 'N/A')}")

    return {"status": status, "asr": asr, "tamper": tamper}


def main():
    parser = argparse.ArgumentParser(description="Windows Defender Audit Agent")
    parser.add_argument("--audit", action="store_true", help="Run full audit")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    if args.audit:
        report = run_audit()
        if args.output:
            with open(args.output, "w") as f:
                json.dump(report, f, indent=2, default=str)
            print(f"\n[+] Report saved to {args.output}")
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-windows-defender-advanced-settings/scripts/process.py =====
#!/usr/bin/env python3
"""
Windows Defender Configuration Auditor

Collects and audits Microsoft Defender for Endpoint settings across endpoints,
identifies configuration gaps, and generates compliance reports.
"""

import json
import subprocess
import sys
import os
from datetime import datetime


RECOMMENDED_SETTINGS = {
    "RealTimeProtectionEnabled": True,
    "BehaviorMonitoringEnabled": True,
    "IoavProtectionEnabled": True,
    "AntispywareEnabled": True,
    "AntivirusEnabled": True,
    "MAPSReporting": 2,  # Advanced
    "SubmitSamplesConsent": 3,  # SendAllSamples
    "PUAProtection": 1,  # Enabled
    "DisableBlockAtFirstSeen": False,
    "CloudBlockLevel": 2,  # High
    "CloudExtendedTimeout": 50,
    "EnableNetworkProtection": 1,  # Enabled
    "EnableControlledFolderAccess": 1,  # Enabled
    "SignatureUpdateInterval": 1,  # Hourly
}

RECOMMENDED_ASR_RULES = {
    "BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550": {"name": "Block executable content from email", "mode": 1},
    "D4F940AB-401B-4EFC-AADC-AD5F3C50688A": {"name": "Block Office child processes", "mode": 1},
    "3B576869-A4EC-4529-8536-B80A7769E899": {"name": "Block Office executable creation", "mode": 1},
    "75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84": {"name": "Block Office code injection", "mode": 1},
    "D3E037E1-3EB8-44C8-A917-57927947596D": {"name": "Block JS/VBS launching executables", "mode": 1},
    "5BEB7EFE-FD9A-4556-801D-275E5FFC04CC": {"name": "Block obfuscated scripts", "mode": 1},
    "92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B": {"name": "Block Win32 API from macros", "mode": 1},
    "9E6C4E1F-7D60-472F-BA1A-A39EF669E4B2": {"name": "Block LSASS credential stealing", "mode": 1},
    "D1E49AAC-8F56-4280-B9BA-993A6D77406C": {"name": "Block PSExec/WMI processes", "mode": 1},
    "B2B3F03D-6A65-4F7B-A9C7-1C7EF74A9BA4": {"name": "Block untrusted USB processes", "mode": 1},
    "E6DB77E5-3DF2-4CF1-B95A-636979351E5B": {"name": "Block WMI persistence", "mode": 1},
    "56A863A9-875E-4185-98A7-B882C64B5CE5": {"name": "Block vulnerable signed drivers", "mode": 1},
}


def collect_defender_settings() -> dict:
    """Collect current Defender settings via PowerShell."""
    ps_cmd = """
    $prefs = Get-MpPreference
    $status = Get-MpComputerStatus
    $result = @{
        Preferences = @{
            RealTimeProtectionEnabled = $prefs.DisableRealtimeMonitoring -eq $false
            BehaviorMonitoringEnabled = $prefs.DisableBehaviorMonitoring -eq $false
            IoavProtectionEnabled = $prefs.DisableIOAVProtection -eq $false
            AntispywareEnabled = $status.AntispywareEnabled
            AntivirusEnabled = $status.AntivirusEnabled
            MAPSReporting = $prefs.MAPSReporting
            SubmitSamplesConsent = $prefs.SubmitSamplesConsent
            PUAProtection = $prefs.PUAProtection
            DisableBlockAtFirstSeen = $prefs.DisableBlockAtFirstSeen
            CloudBlockLevel = $prefs.CloudBlockLevel
            CloudExtendedTimeout = $prefs.CloudExtendedTimeout
            EnableNetworkProtection = $prefs.EnableNetworkProtection
            EnableControlledFolderAccess = $prefs.EnableControlledFolderAccess
            SignatureUpdateInterval = $prefs.SignatureUpdateInterval
        }
        ASRRules = @{}
        Status = @{
            AMEngineVersion = $status.AMEngineVersion
            AMProductVersion = $status.AMProductVersion
            AntispywareSignatureVersion = $status.AntispywareSignatureVersion
            AntivirusSignatureVersion = $status.AntivirusSignatureVersion
            AntivirusSignatureLastUpdated = $status.AntivirusSignatureLastUpdated.ToString('o')
            FullScanEndTime = if($status.FullScanEndTime) { $status.FullScanEndTime.ToString('o') } else { 'Never' }
            QuickScanEndTime = if($status.QuickScanEndTime) { $status.QuickScanEndTime.ToString('o') } else { 'Never' }
            RealTimeProtectionEnabled = $status.RealTimeProtectionEnabled
            TamperProtectionSource = $status.IsTamperProtected
        }
    }
    $ids = $prefs.AttackSurfaceReductionRules_Ids
    $actions = $prefs.AttackSurfaceReductionRules_Actions
    if ($ids -and $actions) {
        for ($i=0; $i -lt $ids.Count; $i++) {
            $result.ASRRules[$ids[$i].ToString().ToUpper()] = $actions[$i]
        }
    }
    $result | ConvertTo-Json -Depth 3
    """

    try:
        result = subprocess.run(
            ["powershell", "-NoProfile", "-Command", ps_cmd],
            capture_output=True, text=True, timeout=30,
        )
        if result.returncode == 0 and result.stdout.strip():
            return json.loads(result.stdout)
        else:
            return {"error": result.stderr or "Failed to collect Defender settings"}
    except FileNotFoundError:
        return {"error": "PowerShell not available (requires Windows)"}
    except subprocess.TimeoutExpired:
        return {"error": "PowerShell command timed out"}
    except json.JSONDecodeError as e:
        return {"error": f"Failed to parse PowerShell output: {e}"}


def audit_settings(settings: dict) -> dict:
    """Audit collected settings against recommended baseline."""
    findings = {
        "compliant": [],
        "non_compliant": [],
        "asr_rules": {
            "enabled_block": [],
            "enabled_audit": [],
            "disabled": [],
            "missing": [],
        },
        "score": 0.0,
    }

    prefs = settings.get("Preferences", {})
    total_checks = 0
    passed_checks = 0

    for setting, expected in RECOMMENDED_SETTINGS.items():
        total_checks += 1
        actual = prefs.get(setting)

        if actual == expected:
            passed_checks += 1
            findings["compliant"].append({
                "setting": setting,
                "expected": expected,
                "actual": actual,
            })
        else:
            findings["non_compliant"].append({
                "setting": setting,
                "expected": expected,
                "actual": actual,
                "severity": "high" if setting in (
                    "RealTimeProtectionEnabled", "AntivirusEnabled",
                    "EnableNetworkProtection", "MAPSReporting",
                ) else "medium",
            })

    asr_rules = settings.get("ASRRules", {})
    for rule_guid, rule_info in RECOMMENDED_ASR_RULES.items():
        total_checks += 1
        mode = asr_rules.get(rule_guid)

        if mode == 1:
            passed_checks += 1
            findings["asr_rules"]["enabled_block"].append({
                "guid": rule_guid, "name": rule_info["name"],
            })
        elif mode == 2:
            findings["asr_rules"]["enabled_audit"].append({
                "guid": rule_guid, "name": rule_info["name"],
            })
        elif mode == 0:
            findings["asr_rules"]["disabled"].append({
                "guid": rule_guid, "name": rule_info["name"],
            })
        else:
            findings["asr_rules"]["missing"].append({
                "guid": rule_guid, "name": rule_info["name"],
            })

    if total_checks > 0:
        findings["score"] = round((passed_checks / total_checks) * 100, 2)

    return findings


def generate_report(settings: dict, findings: dict, output_path: str) -> None:
    """Generate configuration audit report."""
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "defender_status": settings.get("Status", {}),
        "compliance_score": findings["score"],
        "total_compliant": len(findings["compliant"]),
        "total_non_compliant": len(findings["non_compliant"]),
        "asr_summary": {
            "block_mode": len(findings["asr_rules"]["enabled_block"]),
            "audit_mode": len(findings["asr_rules"]["enabled_audit"]),
            "disabled": len(findings["asr_rules"]["disabled"]),
            "not_configured": len(findings["asr_rules"]["missing"]),
        },
        "non_compliant_settings": findings["non_compliant"],
        "asr_details": findings["asr_rules"],
    }

    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2)


if __name__ == "__main__":
    output_dir = sys.argv[1] if len(sys.argv) > 1 else "."

    print("Collecting Microsoft Defender settings...")
    settings = collect_defender_settings()

    if "error" in settings:
        print(f"Error: {settings['error']}")
        print("This tool requires Windows with Microsoft Defender for Endpoint.")
        sys.exit(1)

    print("Auditing against recommended baseline...")
    findings = audit_settings(settings)

    report_path = os.path.join(output_dir, "defender_audit_report.json")
    generate_report(settings, findings, report_path)
    print(f"Audit report: {report_path}")

    print(f"\n--- Defender Configuration Audit ---")
    print(f"Compliance Score: {findings['score']}%")
    print(f"Compliant settings: {len(findings['compliant'])}")
    print(f"Non-compliant: {len(findings['non_compliant'])}")
    print(f"\nASR Rules:")
    print(f"  Block mode: {len(findings['asr_rules']['enabled_block'])}")
    print(f"  Audit mode: {len(findings['asr_rules']['enabled_audit'])}")
    print(f"  Disabled: {len(findings['asr_rules']['disabled'])}")
    print(f"  Not configured: {len(findings['asr_rules']['missing'])}")

    if findings["non_compliant"]:
        print(f"\nNon-compliant settings requiring remediation:")
        for item in findings["non_compliant"]:
            print(f"  [{item['severity'].upper()}] {item['setting']}: expected={item['expected']}, actual={item['actual']}")

===== END FILE =====


===== FILE: skills/configuring-windows-event-logging-for-detection/SKILL.md =====
---
name: configuring-windows-event-logging-for-detection
description: 'Configures Windows Event Logging with advanced audit policies to generate
  high-fidelity security events for threat detection and forensic investigation. Use
  when enabling audit policies for logon events, process creation, privilege use,
  and object access to feed SIEM detection rules. Activates for requests involving
  Windows audit policy, event log configuration, security logging, or detection-oriented
  logging.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- windows-security
- event-logging
- audit-policy
- detection-engineering
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1685.005
- T1059.001
- T1053.005
- T1047
- T1543.003
---
# Configuring Windows Event Logging for Detection

## When to Use

Use this skill when:
- Configuring Windows Advanced Audit Policy for security monitoring
- Enabling process creation auditing with command line logging (Event 4688)
- Setting up logon/logoff auditing for authentication monitoring
- Sizing event log storage and forwarding to SIEM platforms

**Do not use** for Sysmon configuration (separate skill) or Linux audit logging.

## Prerequisites

- Windows Server or Windows 10/11 systems with Group Policy management access
- Active Directory environment with Group Policy Object (GPO) creation privileges
- SIEM platform configured to receive Windows Event Log forwarding
- Understanding of Windows security event IDs and audit categories

## Workflow

### Step 1: Configure Advanced Audit Policy via GPO

```
Computer Configuration → Windows Settings → Security Settings
  → Advanced Audit Policy Configuration → Audit Policies

Recommended settings:
Account Logon:
  - Audit Credential Validation: Success, Failure
  - Audit Kerberos Authentication: Success, Failure

Account Management:
  - Audit Security Group Management: Success
  - Audit User Account Management: Success, Failure

Logon/Logoff:
  - Audit Logon: Success, Failure
  - Audit Logoff: Success
  - Audit Special Logon: Success
  - Audit Other Logon/Logoff Events: Success, Failure

Object Access:
  - Audit File Share: Success, Failure
  - Audit Removable Storage: Success, Failure
  - Audit SAM: Success

Policy Change:
  - Audit Audit Policy Change: Success, Failure
  - Audit Authentication Policy Change: Success

Privilege Use:
  - Audit Sensitive Privilege Use: Success, Failure

Detailed Tracking:
  - Audit Process Creation: Success
  - Audit DPAPI Activity: Success, Failure
```

### Step 2: Enable Command Line in Process Creation Events

```powershell
# Registry: Enable command line logging in Event 4688
New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" `
  -Name ProcessCreationIncludeCmdLine_Enabled -Value 1 -PropertyType DWORD -Force

# GPO: Computer Configuration → Administrative Templates → System → Audit Process Creation
# "Include command line in process creation events" → Enabled
```

### Step 3: Configure Event Log Sizes

```powershell
# Increase Security log to 1 GB (default 20 MB is insufficient)
wevtutil sl Security /ms:1073741824

# Increase PowerShell Operational log
wevtutil sl "Microsoft-Windows-PowerShell/Operational" /ms:536870912

# Set log retention to overwrite as needed
wevtutil sl Security /rt:false

# Configure via GPO:
# Computer Configuration → Administrative Templates → Windows Components
#   → Event Log Service → Security
# Maximum log file size (KB): 1048576
```

### Step 4: Configure Windows Event Forwarding (WEF)

```powershell
# On collector server:
wecutil qc /q

# Create subscription for high-value events:
# Event IDs: 4624 (logon), 4625 (failed logon), 4688 (process create),
# 4672 (special privilege), 4720 (user created), 4728 (group membership),
# 7045 (service installed), 1102 (log cleared)

# On source endpoints (GPO):
# Configure WinRM: winrm quickconfig
# Configure event forwarding: Computer Configuration → Admin Templates
#   → Windows Components → Event Forwarding
# Configure target Subscription Manager: Server=http://collector:5985/wsman/SubscriptionManager/WEC
```

### Step 5: Key Event IDs for Detection

```
Authentication Events:
  4624 - Successful logon (Type 2=Interactive, 3=Network, 10=RemoteInteractive)
  4625 - Failed logon attempt
  4648 - Logon using explicit credentials (RunAs, pass-the-hash indicator)
  4672 - Special privileges assigned (admin logon)
  4776 - NTLM credential validation

Process Events:
  4688 - Process creation (with command line if enabled)
  4689 - Process termination

Account Events:
  4720 - User account created
  4722 - User account enabled
  4724 - Password reset attempted
  4728 - Member added to security group
  4732 - Member added to local group
  4756 - Member added to universal group

Service/System Events:
  7045 - New service installed (persistence indicator)
  1102 - Audit log cleared (evidence tampering)
  4697 - Service installed in the system

Lateral Movement Indicators:
  4648 + 4624(Type 3) - Credential-based lateral movement
  5140 - Network share accessed
  5145 - Network share access check (detailed file share)
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Advanced Audit Policy** | Granular audit subcategories (58 subcategories vs. 9 basic categories) |
| **Event ID 4688** | Process creation event; essential for tracking execution on endpoints |
| **WEF** | Windows Event Forwarding; centralized log collection without third-party agents |
| **Logon Type** | Numeric code indicating authentication method (2=interactive, 3=network, 10=RDP) |

## Tools & Systems

- **Windows Event Forwarding (WEF)**: Built-in centralized log collection
- **NXLog**: Open-source log forwarding agent for Windows events
- **Winlogbeat**: Elastic Agent for shipping Windows event logs to Elasticsearch
- **Palantir WEF Configuration**: Open-source WEF subscription templates

## Common Pitfalls

- **Using basic audit policy instead of advanced**: Basic and advanced audit policies conflict. Always use advanced audit policy exclusively.
- **Default log size too small**: 20 MB Security log fills in minutes on busy servers. Set minimum 1 GB.
- **Missing command line logging**: Event 4688 without command line content has minimal detection value. Always enable ProcessCreationIncludeCmdLine_Enabled.
- **Not forwarding logs**: Local event logs are lost when endpoints are wiped by ransomware. Forward to centralized SIEM immediately.

===== END FILE =====


===== FILE: skills/configuring-windows-event-logging-for-detection/LICENSE =====

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


===== FILE: skills/configuring-windows-event-logging-for-detection/assets/template.md =====
# Windows Event Logging Configuration Template

## Audit Policy Settings
| Subcategory | Setting | Status |
|-------------|---------|--------|
| Credential Validation | Success, Failure | |
| Logon | Success, Failure | |
| Process Creation | Success | |
| Special Logon | Success | |

## Log Sizes
| Log | Size | Retention |
|-----|------|-----------|
| Security | 1 GB | Overwrite |
| PowerShell/Operational | 512 MB | Overwrite |

## Sign-Off
| Role | Name | Date |
|------|------|------|
| Security | | |

===== END FILE =====


===== FILE: skills/configuring-windows-event-logging-for-detection/references/api-reference.md =====
# Windows Event Logging for Detection — API Reference

## Key PowerShell Cmdlets

| Cmdlet | Description |
|--------|-------------|
| `auditpol /get /category:*` | View advanced audit policy |
| `auditpol /set /subcategory:"Process Creation" /success:enable` | Enable audit subcategory |
| `Get-WinEvent -ListLog *` | List available event logs |
| `wevtutil sl Security /ms:1073741824` | Set Security log max size to 1 GB |

## Critical Event IDs for Detection

| Event ID | Log | Description |
|----------|-----|-------------|
| 4624/4625 | Security | Successful/failed logon |
| 4662 | Security | Directory service object access |
| 4688 | Security | Process creation (with command line) |
| 4698 | Security | Scheduled task created |
| 4720 | Security | User account created |
| 4732 | Security | Member added to security group |
| 4768/4769 | Security | Kerberos TGT/service ticket |
| 1 | Sysmon | Process creation with hashes |
| 3 | Sysmon | Network connection |
| 7 | Sysmon | Image loaded (DLL) |
| 11 | Sysmon | File creation |
| 4104 | PowerShell | Script block logging |

## Recommended Log Sizes

| Log | Minimum Size |
|-----|-------------|
| Security | 1 GB |
| Sysmon/Operational | 512 MB |
| PowerShell/Operational | 256 MB |
| System | 256 MB |

## External References

- [Microsoft Audit Policy](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/advanced-security-auditing)
- [Sysmon Configuration](https://github.com/SwiftOnSecurity/sysmon-config)
- [MITRE ATT&CK Data Sources](https://attack.mitre.org/datasources/)

===== END FILE =====


===== FILE: skills/configuring-windows-event-logging-for-detection/references/standards.md =====
# Standards & References
- **NIST SP 800-92**: Guide to Computer Security Log Management
- **CIS Benchmark Section 17**: Advanced Audit Policy Configuration
- **NSA/CISA Event Forwarding Guidance**: Recommended events for Windows monitoring
- **Palantir WEF Config**: https://github.com/palantir/windows-event-forwarding
- **SANS Windows Logging Cheat Sheet**: https://www.sans.org/posters/

===== END FILE =====


===== FILE: skills/configuring-windows-event-logging-for-detection/references/workflows.md =====
# Workflows
## Event Logging Deployment
```
[Audit current logging configuration] → [Enable Advanced Audit Policy via GPO]
  → [Enable command line logging] → [Increase log sizes]
  → [Configure WEF or agent-based forwarding] → [Verify events in SIEM]
  → [Build detection rules from high-value events] → [Quarterly logging audit]
```

===== END FILE =====


===== FILE: skills/configuring-windows-event-logging-for-detection/scripts/agent.py =====
#!/usr/bin/env python3
"""Windows event logging configuration audit agent."""

import json
import argparse
import subprocess
from datetime import datetime


CRITICAL_AUDIT_POLICIES = {
    "Logon/Logoff": {"Logon": "Success,Failure", "Logoff": "Success"},
    "Account Logon": {"Credential Validation": "Success,Failure", "Kerberos Authentication Service": "Success,Failure"},
    "Object Access": {"File System": "Success,Failure", "Registry": "Success,Failure"},
    "Privilege Use": {"Sensitive Privilege Use": "Success,Failure"},
    "Process Tracking": {"Process Creation": "Success"},
    "DS Access": {"Directory Service Access": "Success,Failure"},
    "Policy Change": {"Audit Policy Change": "Success,Failure", "Authentication Policy Change": "Success"},
}


def get_audit_policy():
    """Get current advanced audit policy configuration."""
    cmd = ["auditpol", "/get", "/category:*"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=15)
        lines = result.stdout.strip().split("\n")
        policies = {}
        current_category = ""
        for line in lines:
            stripped = line.strip()
            if not stripped or "Machine Name" in stripped or "Category" in stripped:
                continue
            if not stripped.startswith("  "):
                current_category = stripped
                policies[current_category] = {}
            else:
                parts = stripped.rsplit("  ", 1)
                if len(parts) == 2:
                    policies[current_category][parts[0].strip()] = parts[1].strip()
        return policies
    except (FileNotFoundError, subprocess.TimeoutExpired) as e:
        return {"error": str(e)}


def check_sysmon_installed():
    """Check if Sysmon is installed and running."""
    cmd = ["powershell", "-Command", "Get-Service Sysmon* | ConvertTo-Json"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=15)
        if result.stdout.strip():
            service = json.loads(result.stdout)
            if isinstance(service, list):
                service = service[0]
            return {"installed": True, "status": service.get("Status", ""),
                    "name": service.get("Name", "")}
        return {"installed": False, "severity": "HIGH",
                "recommendation": "Install Sysmon with SwiftOnSecurity config"}
    except (FileNotFoundError, json.JSONDecodeError):
        return {"installed": False}


def check_log_sizes():
    """Check event log maximum sizes."""
    logs = ["Security", "System", "Application", "Microsoft-Windows-Sysmon/Operational",
            "Microsoft-Windows-PowerShell/Operational"]
    results = []
    for log_name in logs:
        cmd = ["powershell", "-Command",
               f"(Get-WinEvent -ListLog '{log_name}' -ErrorAction SilentlyContinue).MaximumSizeInBytes"]
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
            size_bytes = int(result.stdout.strip()) if result.stdout.strip() else 0
            size_mb = round(size_bytes / (1024 * 1024), 1)
            results.append({
                "log": log_name,
                "max_size_mb": size_mb,
                "severity": "MEDIUM" if size_mb < 100 else "INFO",
            })
        except (ValueError, subprocess.TimeoutExpired):
            results.append({"log": log_name, "error": "Cannot query"})
    return results


def check_powershell_logging():
    """Check PowerShell script block logging and transcription."""
    checks = {}
    for name, path in [
        ("ScriptBlockLogging", r"HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"),
        ("Transcription", r"HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription"),
    ]:
        cmd = ["powershell", "-Command", f"Get-ItemProperty -Path '{path}' -ErrorAction SilentlyContinue | ConvertTo-Json"]
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
            checks[name] = json.loads(result.stdout) if result.stdout.strip() else {"enabled": False}
        except (json.JSONDecodeError, subprocess.TimeoutExpired):
            checks[name] = {"enabled": False}
    return checks


def run_audit():
    """Execute Windows event logging audit."""
    print(f"\n{'='*60}")
    print(f"  WINDOWS EVENT LOGGING AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    sysmon = check_sysmon_installed()
    print(f"--- SYSMON ---")
    print(f"  Installed: {sysmon.get('installed', False)}")
    print(f"  Status: {sysmon.get('status', 'N/A')}")

    logs = check_log_sizes()
    print(f"\n--- LOG SIZES ---")
    for l in logs:
        if "error" not in l:
            print(f"  {l['log']}: {l['max_size_mb']} MB [{l['severity']}]")

    ps_logging = check_powershell_logging()
    print(f"\n--- POWERSHELL LOGGING ---")
    for name, config in ps_logging.items():
        enabled = config.get("EnableScriptBlockLogging", config.get("EnableTranscripting", False))
        print(f"  {name}: {'Enabled' if enabled else 'Disabled'}")

    return {"sysmon": sysmon, "log_sizes": logs, "powershell": ps_logging}


def main():
    parser = argparse.ArgumentParser(description="Windows Event Logging Audit Agent")
    parser.add_argument("--audit", action="store_true", help="Run full audit")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    if args.audit:
        report = run_audit()
        if args.output:
            with open(args.output, "w") as f:
                json.dump(report, f, indent=2, default=str)
            print(f"\n[+] Report saved to {args.output}")
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-windows-event-logging-for-detection/scripts/process.py =====
#!/usr/bin/env python3
"""Windows Event Logging Auditor - Checks current audit policy configuration."""

import json, subprocess, sys, os
from datetime import datetime


def get_audit_policy() -> dict:
    """Query current advanced audit policy via auditpol."""
    try:
        result = subprocess.run(
            ["auditpol", "/get", "/category:*"],
            capture_output=True, text=True, timeout=15,
        )
        if result.returncode != 0:
            return {"error": result.stderr}

        policies = {}
        current_category = ""
        for line in result.stdout.splitlines():
            line = line.strip()
            if not line:
                continue
            if "  " not in line and line.endswith(":"):
                continue
            parts = line.rsplit("  ", 1)
            if len(parts) == 2:
                name = parts[0].strip()
                setting = parts[1].strip()
                policies[name] = setting
        return policies
    except FileNotFoundError:
        return {"error": "auditpol not available (requires Windows)"}


RECOMMENDED = {
    "Credential Validation": "Success and Failure",
    "Security Group Management": "Success",
    "User Account Management": "Success and Failure",
    "Logon": "Success and Failure",
    "Logoff": "Success",
    "Special Logon": "Success",
    "Process Creation": "Success",
    "Audit Policy Change": "Success",
    "Sensitive Privilege Use": "Success and Failure",
}


if __name__ == "__main__":
    policies = get_audit_policy()
    if "error" in policies:
        print(f"Error: {policies['error']}")
        sys.exit(1)

    compliant = 0
    total = len(RECOMMENDED)
    for setting, expected in RECOMMENDED.items():
        actual = policies.get(setting, "No Auditing")
        status = "PASS" if expected in actual else "FAIL"
        if status == "PASS":
            compliant += 1
        print(f"[{status}] {setting}: {actual} (expected: {expected})")

    print(f"\nScore: {compliant}/{total} ({round(compliant/total*100)}%)")

===== END FILE =====


===== FILE: skills/deploying-edr-agent-with-crowdstrike/SKILL.md =====
---
name: deploying-edr-agent-with-crowdstrike
description: 'Deploys and configures CrowdStrike Falcon EDR agents across enterprise
  endpoints to enable real-time threat detection, behavioral analysis, and automated
  response. Use when onboarding endpoints to EDR coverage, configuring detection policies,
  or integrating Falcon telemetry with SIEM platforms. Activates for requests involving
  CrowdStrike deployment, Falcon sensor installation, EDR policy configuration, or
  endpoint detection and response.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- edr
- CrowdStrike
- Falcon
- threat-detection
- sensor-deployment
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- GOVERN-1.1
- MEASURE-2.7
- MANAGE-3.1
- MAP-5.1
- MANAGE-2.4
atlas_techniques:
- AML.T0070
- AML.T0066
- AML.T0082
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1003.001
- T1055
- T1059.001
- T1486
- T1071.001
---
# Deploying EDR Agent with CrowdStrike

## When to Use

Use this skill when:
- Deploying CrowdStrike Falcon sensors to Windows, macOS, or Linux endpoints
- Configuring Falcon prevention and detection policies for different endpoint groups
- Integrating CrowdStrike telemetry with SIEM (Splunk, Elastic, Sentinel) for correlated detection
- Troubleshooting sensor connectivity, performance, or detection issues

**Do not use** this skill for deploying other EDR solutions (Carbon Black, SentinelOne) or for Falcon cloud workload protection (use cloud-specific deployment guides).

## Prerequisites

- CrowdStrike Falcon console access with Falcon Administrator role
- Customer ID (CID) and Falcon sensor installer package
- Administrative/root access on target endpoints
- Network access: endpoints must reach CrowdStrike cloud (ts01-b.cloudsink.net on port 443)
- Deployment tool: SCCM, Intune, GPO, Ansible, or manual installation

## Workflow

### Step 1: Obtain Falcon Sensor Installer and CID

```
1. Log into Falcon Console: https://falcon.crowdstrike.com
2. Navigate: Host setup and management → Sensor downloads
3. Download the appropriate installer:
   - Windows: WindowsSensor_<version>.exe
   - macOS: FalconSensorMacOS_<version>.pkg
   - Linux: falcon-sensor_<version>_amd64.deb / .rpm
4. Copy the Customer ID (CID) from the Sensor downloads page
   - CID format: <32-char-hex>-<2-char-checksum>
```

### Step 2: Deploy Falcon Sensor - Windows

**Silent installation via command line**:
```cmd
WindowsSensor_7.18.17106.exe /install /quiet /norestart CID=<YOUR_CID>
```

**SCCM deployment**:
```
1. Create an Application in SCCM
2. Deployment type: Script Installer
3. Install command: WindowsSensor_7.18.17106.exe /install /quiet /norestart CID=<CID>
4. Detection method: Registry key exists
   - HKLM\SYSTEM\CrowdStrike\{9b03c1d9-3138-44ed-9fae-d9f4c034b88d}\{16e0423f-7058-48c9-a204-725362b67639}\Default
5. Deploy to target collection
6. Deployment purpose: Required (for mandatory installation)
```

**Microsoft Intune deployment**:
```
1. Navigate: Devices → Windows → Configuration profiles
2. Create Win32 app deployment
3. Upload .intunewin package (wrapped sensor installer)
4. Install command: WindowsSensor_7.18.17106.exe /install /quiet /norestart CID=<CID>
5. Detection rule: File exists C:\Windows\System32\drivers\CrowdStrike\csagent.sys
6. Assign to device group
```

**GPO deployment**:
```powershell
# Create startup script that checks for existing installation
$sensorPath = "C:\Windows\System32\drivers\CrowdStrike\csagent.sys"
if (-not (Test-Path $sensorPath)) {
    Start-Process -FilePath "\\fileserver\CrowdStrike\WindowsSensor.exe" `
      -ArgumentList "/install /quiet /norestart CID=<CID>" -Wait
}
```

### Step 3: Deploy Falcon Sensor - Linux

```bash
# Debian/Ubuntu
sudo dpkg -i falcon-sensor_7.18.0-17106_amd64.deb
sudo /opt/CrowdStrike/falconctl -s -f --cid=<YOUR_CID>
sudo systemctl start falcon-sensor
sudo systemctl enable falcon-sensor

# RHEL/CentOS
sudo yum install falcon-sensor-7.18.0-17106.el8.x86_64.rpm
sudo /opt/CrowdStrike/falconctl -s -f --cid=<YOUR_CID>
sudo systemctl start falcon-sensor
sudo systemctl enable falcon-sensor

# Verify sensor is running and connected
sudo /opt/CrowdStrike/falconctl -g --rfm-state
# Expected output: rfm-state=false (sensor is communicating with cloud)
```

### Step 4: Deploy Falcon Sensor - macOS

```bash
# Install sensor package
sudo installer -pkg FalconSensorMacOS_7.18.pkg -target /

# Set CID
sudo /Applications/Falcon.app/Contents/Resources/falconctl license <YOUR_CID>

# Grant Full Disk Access and System Extension via MDM profile
# Required for macOS Ventura+ (manual approval or MDM PPPC profile)
# MDM payload: com.crowdstrike.falcon.Agent → SystemExtension + Full Disk Access

# Verify sensor status
sudo /Applications/Falcon.app/Contents/Resources/falconctl stats
```

### Step 5: Configure Prevention Policies

In Falcon Console, navigate to Configuration → Prevention Policies:

**Recommended prevention policy settings**:
```
Machine Learning:
  - Cloud ML: Aggressive (extra protection, may increase false positives)
  - Sensor ML: Moderate
  - Adware & PUP: Moderate

Behavioral Protection:
  - On Write: Enabled (detect malware on file creation)
  - On Sensor ML: Enabled
  - Interpreter-Only: Enabled (detect script-based attacks)

Exploit Mitigation:
  - Exploit behavior protection: Enabled
  - Memory scanning: Enabled (detects in-memory attacks)
  - Code injection: Enabled

Ransomware:
  - Ransomware protection: Enabled
  - Shadow copy protection: Enabled
  - MBR protection: Enabled
```

**Create separate policies for**:
- Workstations (aggressive settings)
- Servers (moderate settings to avoid false positives on server workloads)
- Critical infrastructure (maximum protection with exception lists)

### Step 6: Configure Response Policies

```
Real-Time Response:
  - Enable RTR for all sensor groups
  - Configure RTR admin vs. RTR responder roles
  - Enable script execution (for IR teams)
  - Enable file extraction (for forensics)

Network Containment:
  - Pre-authorize containment for specific host groups
  - Configure containment exclusions (allow management traffic)

Automated Response:
  - Enable automated remediation for high-confidence detections
  - Configure kill process action for ransomware detections
  - Enable quarantine for malware file detections
```

### Step 7: Validate Deployment

```powershell
# Windows: Check Falcon sensor status
sc query csagent
# Expected: RUNNING

# Check sensor version
reg query "HKLM\SYSTEM\CrowdStrike\{9b03c1d9-3138-44ed-9fae-d9f4c034b88d}\{16e0423f-7058-48c9-a204-725362b67639}\Default" /v AgentVersion

# Verify cloud connectivity
# In Falcon Console: Host Management → Hosts → search for hostname
# Status should show "Online" with last seen timestamp < 5 minutes
```

**Test detection capability**:
```powershell
# CrowdStrike provides test detection samples
# Download CsTestDetect.exe from Falcon Console → Host setup
# Run on endpoint to generate a test detection
.\CsTestDetect.exe
# Verify detection appears in Falcon Console within 60 seconds
```

### Step 8: SIEM Integration

```
# Falcon SIEM Connector (Streaming API)
# Configure in Falcon Console: Support → API Clients and Keys

# Create API client with scope: Event Streams → Read
# Use falcon-siem-connector or Falcon Data Replicator (FDR)

# Splunk integration:
# Install CrowdStrike Falcon Event Streams Technical Add-on from Splunkbase
# Configure: Settings → Data inputs → CrowdStrike Falcon Event Streams
# Enter API Client ID and Secret
# Index: crowdstrike_events

# Elastic integration:
# Use Elastic Agent with CrowdStrike module
# Configure: Fleet → Agent policies → Add integration → CrowdStrike
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Falcon Sensor** | Lightweight kernel-mode agent (25-30 MB) that collects endpoint telemetry and enforces prevention policies |
| **CID (Customer ID)** | Unique identifier that associates the sensor with your CrowdStrike Falcon tenant |
| **RFM (Reduced Functionality Mode)** | State where sensor operates with limited capability due to cloud connectivity loss |
| **Sensor Grouping Tags** | Labels applied during installation to auto-assign hosts to groups and policies |
| **RTR (Real-Time Response)** | Remote shell capability for incident responders to interact with endpoints through Falcon |
| **IOA (Indicators of Attack)** | Behavioral detections based on adversary techniques rather than static signatures |

## Tools & Systems

- **CrowdStrike Falcon Console**: Cloud-hosted management platform for all Falcon modules
- **Falcon SIEM Connector**: Streams detection and audit events to SIEM platforms
- **Falcon Data Replicator (FDR)**: Streams raw endpoint telemetry to S3/cloud storage for hunting
- **CrowdStrike Falcon API (OAuth2)**: RESTful API for automation, integration, and custom workflows
- **PSFalcon**: PowerShell module for CrowdStrike Falcon API automation

## Common Pitfalls

- **Missing CID during installation**: Sensor installs but never connects to Falcon cloud. Always pass CID during install, not after.
- **Proxy not configured**: In environments with web proxies, configure proxy during installation: `/install /quiet CID=<CID> APP_PROXYNAME=proxy.corp.com APP_PROXYPORT=8080`.
- **macOS System Extension blocked**: macOS requires explicit approval for kernel/system extensions. Use MDM to pre-approve CrowdStrike extensions before deployment.
- **Conflicting security products**: Running multiple EDR/AV products causes performance issues and false positives. Coordinate exclusions or remove legacy AV before Falcon deployment.
- **Sensor version pinning**: Falcon auto-updates sensors by default. Pin sensor versions in the console for change-controlled environments before testing new versions.

===== END FILE =====


===== FILE: skills/deploying-edr-agent-with-crowdstrike/LICENSE =====

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


===== FILE: skills/deploying-edr-agent-with-crowdstrike/assets/template.md =====
# CrowdStrike Falcon EDR Deployment Template

## Deployment Information

| Field | Value |
|-------|-------|
| Falcon Tenant | |
| Customer ID (CID) | |
| Sensor Version | |
| Deployment Tool | SCCM / Intune / GPO / Ansible / Manual |
| Target Environment | Production / Staging / Dev |
| Deployment Date | |
| Deployment Lead | |

## Pre-Deployment Checklist

- [ ] Falcon Console access verified with Administrator role
- [ ] CID obtained from Sensor downloads page
- [ ] Sensor installer downloaded for all target OS platforms
- [ ] Network connectivity verified: endpoints can reach ts01-b.cloudsink.net:443
- [ ] Proxy configuration documented (if applicable)
- [ ] Prevention policies created for each endpoint group
- [ ] Sensor update policy configured (auto-update or pinned version)
- [ ] Existing AV/EDR removal plan documented
- [ ] Exclusion list prepared for known LOB applications
- [ ] Change management ticket approved

## Deployment Scope

| Endpoint Group | Count | OS | Policy | Deployment Phase |
|---------------|-------|-----|--------|-----------------|
| Workstations | | Windows 11 | WS-Prevention-L1 | Phase 1 |
| Standard Servers | | Windows Server 2022 | SRV-Prevention | Phase 2 |
| Linux Servers | | Ubuntu 22.04 | LNX-Prevention | Phase 2 |
| macOS Endpoints | | macOS Ventura | MAC-Prevention | Phase 3 |
| Critical Servers | | Mixed | CRIT-Prevention | Phase 4 |

## Policy Configuration

### Prevention Policy Settings

| Setting | Workstations | Servers | Critical |
|---------|-------------|---------|----------|
| Cloud ML | Aggressive | Moderate | Aggressive |
| Sensor ML | Moderate | Cautious | Moderate |
| On Write | Enabled | Enabled | Enabled |
| Script-based Execution | Enabled | Enabled | Enabled |
| Ransomware Protection | Enabled | Enabled | Enabled |
| Exploit Protection | Enabled | Enabled | Enabled |

### Exclusions Applied

| Path/Process | Reason | Policy Group | Approved By |
|-------------|--------|-------------|-------------|
| | | | |

## Post-Deployment Validation

- [ ] All target endpoints show "Online" in Falcon Console
- [ ] Correct prevention policy applied to each host group
- [ ] Test detection generated with CsTestDetect and visible in console
- [ ] No RFM (Reduced Functionality Mode) hosts
- [ ] No critical application performance degradation reported
- [ ] SIEM integration receiving events
- [ ] RTR access verified for IR team

## Deployment Metrics

| Metric | Target | Actual |
|--------|--------|--------|
| Coverage (% of endpoints with sensor) | 100% | |
| Online rate | >98% | |
| Sensor version compliance | >95% same version | |
| Policy assignment accuracy | 100% | |
| Mean time to deploy (per phase) | <5 business days | |

## Issues and Remediation

| Issue | Affected Hosts | Root Cause | Resolution | Status |
|-------|---------------|------------|-----------|--------|
| | | | | |

## Sign-Off

| Role | Name | Date |
|------|------|------|
| Security Engineer | | |
| IT Operations | | |
| SOC Manager | | |

===== END FILE =====


===== FILE: skills/deploying-edr-agent-with-crowdstrike/references/api-reference.md =====
# CrowdStrike EDR Deployment — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| crowdstrike-falconpy | `pip install crowdstrike-falconpy` | Official CrowdStrike Falcon SDK |

## Key FalconPy Service Classes

| Class | Description |
|-------|-------------|
| `Hosts(client_id, client_secret)` | Host/device management |
| `Detections(client_id, client_secret)` | Detection queries and management |
| `RealTimeResponse(client_id, client_secret)` | RTR session management |
| `SensorDownload(client_id, client_secret)` | Sensor installer download |
| `Prevention(client_id, client_secret)` | Prevention policy management |

## Key Methods

| Method | Description |
|--------|-------------|
| `hosts.query_devices_by_filter(filter=, limit=)` | Query host IDs |
| `hosts.get_device_details(ids=[])` | Get host details |
| `hosts.perform_action(action_name="contain", ids=[])` | Contain/lift containment |
| `detections.query_detects(filter=, sort=)` | Query detection IDs |
| `detections.get_detect_summaries(body={"ids": []})` | Get detection details |

## FQL Filter Examples

```
platform_name:'Windows' + status:'normal'
last_seen:>='2024-01-01T00:00:00Z'
hostname:'*server*'
```

## External References

- [FalconPy Documentation](https://www.falconpy.io/)
- [CrowdStrike API Swagger](https://assets.falcon.crowdstrike.com/support/api/swagger.html)
- [FalconPy GitHub](https://github.com/CrowdStrike/falconpy)

===== END FILE =====


===== FILE: skills/deploying-edr-agent-with-crowdstrike/references/standards.md =====
# Standards & References - Deploying EDR Agent with CrowdStrike

## Primary Standards

### MITRE ATT&CK Enterprise Framework
- **Publisher**: MITRE Corporation
- **URL**: https://attack.mitre.org/
- **Relevance**: CrowdStrike Falcon maps detections to ATT&CK techniques; understanding ATT&CK is essential for tuning detection policies
- **Key tactics for EDR**: Initial Access, Execution, Persistence, Privilege Escalation, Defense Evasion, Lateral Movement

### NIST SP 800-83 Rev 1 - Guide to Malware Incident Prevention and Handling
- **Publisher**: NIST
- **Relevance**: Defines endpoint protection architecture including EDR placement and malware prevention controls

### CIS Control 10 - Malware Defenses
- **Publisher**: Center for Internet Security
- **Relevance**: CIS Controls v8 Control 10 mandates deploying anti-malware with centralized management and automated updates

## CrowdStrike-Specific References

### CrowdStrike Falcon Deployment Guide
- **Scope**: Official deployment procedures for Windows, macOS, Linux sensors
- **Key sections**: Silent install parameters, proxy configuration, sensor grouping tags
- **Access**: Falcon Console → Support → Documentation

### CrowdStrike Falcon API Documentation
- **URL**: https://falcon.crowdstrike.com/documentation/
- **Scope**: OAuth2 authentication, host management, detection management, RTR APIs
- **Key endpoints**: /devices/queries/devices/v1, /detects/queries/detects/v1

### Falcon SIEM Integration Guide
- **Scope**: Event streaming via SIEM Connector, FDR, and direct API
- **Supported SIEMs**: Splunk, Elastic, Microsoft Sentinel, IBM QRadar, ArcSight

## Compliance Mappings

| Framework | Requirement | CrowdStrike Coverage |
|-----------|------------|---------------------|
| PCI DSS 4.0 | 5.2 - Anti-malware on systems | Falcon sensor on all in-scope endpoints |
| HIPAA | 164.308(a)(5)(ii)(B) - Protection from malware | Falcon prevention + detection |
| SOC 2 | CC6.8 - Malicious software prevention | Falcon sensor with prevention policies |
| NIST 800-171 | 3.14.2 - Malicious code protection | Falcon ML + behavioral detection |
| ISO 27001 | A.12.2.1 - Controls against malware | Falcon sensor with automated response |

## Industry Benchmarks

- **MITRE Engenuity ATT&CK Evaluations**: CrowdStrike Falcon regularly achieves high detection scores in Enterprise evaluations
- **Gartner Magic Quadrant for Endpoint Protection**: CrowdStrike positioned as Leader
- **Forrester Wave Endpoint Detection and Response**: CrowdStrike rated Strong Performer/Leader

===== END FILE =====


===== FILE: skills/deploying-edr-agent-with-crowdstrike/references/workflows.md =====
# Workflows - Deploying EDR Agent with CrowdStrike

## Workflow 1: Enterprise Sensor Rollout

```
[Plan Deployment]
    │
    ├── Obtain Falcon Console access and CID
    ├── Download sensor installer for each OS
    ├── Create deployment groups (Workstations, Servers, VDI)
    │
    ▼
[Configure Policies Before Deployment]
    │
    ├── Create prevention policies per group
    ├── Configure sensor update policies (pinned vs. auto-update)
    ├── Set sensor grouping tags for auto-assignment
    │
    ▼
[Pilot Deployment (5% of endpoints)]
    │
    ├── Deploy via SCCM/Intune to pilot group
    ├── Monitor for 1 week: performance impact, false positives
    ├── Tune exclusions for LOB applications
    │
    ▼
[Validation]
    │
    ├── All pilot hosts show "Online" in Falcon Console
    ├── Test detection with CsTestDetect
    ├── No critical application breakage
    │
    ▼
[Production Rollout (phased)]
    │
    ├── Phase 1: Workstations (2 weeks)
    ├── Phase 2: Standard servers (2 weeks)
    ├── Phase 3: Critical servers (1 week, change window)
    │
    ▼
[Post-Deployment]
    │
    ├── Enable SIEM integration
    ├── Configure automated response policies
    ├── Establish exclusion review cadence (monthly)
    └── Train SOC on Falcon Console workflows
```

## Workflow 2: Detection Triage in Falcon Console

```
[New Detection Alert]
    │
    ▼
[Review Detection in Falcon Console]
    │
    ├── Severity: Critical/High/Medium/Low/Informational
    ├── Tactic & Technique (ATT&CK mapping)
    ├── Process tree visualization
    ├── Network connections
    │
    ▼
[Assess: True Positive or False Positive?]
    │
    ├── True Positive ──► [Contain Host via Network Containment]
    │                          │
    │                          ▼
    │                     [Launch RTR session for investigation]
    │                          │
    │                          ▼
    │                     [Collect artifacts, kill malicious processes]
    │                          │
    │                          ▼
    │                     [Remediate and release from containment]
    │
    └── False Positive ──► [Create exclusion rule]
                                │
                                ▼
                           [Document exclusion with justification]
                                │
                                ▼
                           [Mark detection as false positive]
```

## Workflow 3: Sensor Troubleshooting

```
[Sensor Issue Reported]
    │
    ▼
[Check Falcon Console Host Status]
    │
    ├── Online ──► [Issue is not connectivity; check policy assignment]
    │
    └── Offline / RFM ──► [Check network connectivity]
                               │
                               ├── Can reach ts01-b.cloudsink.net:443?
                               │     │
                               │     ├── Yes ──► [Check proxy settings]
                               │     │              ▼
                               │     │          [Reconfigure: falconctl -s --apd=false --aph=proxy --app=8080]
                               │     │
                               │     └── No ──► [Firewall blocking; add CrowdStrike domains to allowlist]
                               │
                               ▼
                          [Check sensor service status]
                               │
                               ├── Service running ──► [Review sensor logs in C:\Windows\System32\drivers\CrowdStrike\]
                               │
                               └── Service stopped ──► [Restart: sc start csagent (Windows) or systemctl start falcon-sensor (Linux)]
```

## Workflow 4: Sensor Version Upgrade

```
[New Sensor Version Available]
    │
    ▼
[Review Release Notes in Falcon Console]
    │
    ▼
[Test on pilot group (N-1 update policy)]
    │
    ├── No issues after 1 week ──► [Move production to N update policy]
    │
    └── Issues found ──► [Hold on current version, file support ticket]
                              │
                              ▼
                         [Pin current version in sensor update policy]
```

===== END FILE =====


===== FILE: skills/deploying-edr-agent-with-crowdstrike/scripts/agent.py =====
#!/usr/bin/env python3
"""CrowdStrike EDR deployment and monitoring agent using FalconPy."""

import json
import sys
import argparse
from datetime import datetime

try:
    from falconpy import Hosts, Detections
except ImportError:
    print("Install: pip install crowdstrike-falconpy")
    sys.exit(1)


def list_hosts(client_id, client_secret, filter_query=None):
    """List managed hosts with sensor details."""
    hosts = Hosts(client_id=client_id, client_secret=client_secret)
    params = {"limit": 100}
    if filter_query:
        params["filter"] = filter_query
    id_resp = hosts.query_devices_by_filter(**params)
    if id_resp["status_code"] != 200:
        return []
    device_ids = id_resp["body"]["resources"]
    if not device_ids:
        return []
    detail_resp = hosts.get_device_details(ids=device_ids)
    results = []
    for device in detail_resp["body"].get("resources", []):
        results.append({
            "hostname": device.get("hostname", ""),
            "device_id": device.get("device_id", ""),
            "platform": device.get("platform_name", ""),
            "os_version": device.get("os_version", ""),
            "sensor_version": device.get("agent_version", ""),
            "status": device.get("status", ""),
            "last_seen": device.get("last_seen", ""),
        })
    return results


def get_detections(client_id, client_secret, severity=None):
    """Retrieve recent detections."""
    detections = Detections(client_id=client_id, client_secret=client_secret)
    params = {"limit": 50, "sort": "last_behavior|desc"}
    if severity:
        params["filter"] = f"max_severity_displayname:'{severity}'"
    id_resp = detections.query_detects(**params)
    if id_resp["status_code"] != 200:
        return []
    detect_ids = id_resp["body"]["resources"]
    if not detect_ids:
        return []
    detail_resp = detections.get_detect_summaries(body={"ids": detect_ids})
    results = []
    for det in detail_resp["body"].get("resources", []):
        results.append({
            "detection_id": det.get("detection_id", ""),
            "hostname": det.get("device", {}).get("hostname", ""),
            "severity": det.get("max_severity_displayname", ""),
            "tactic": det.get("behaviors", [{}])[0].get("tactic", "") if det.get("behaviors") else "",
            "technique": det.get("behaviors", [{}])[0].get("technique", "") if det.get("behaviors") else "",
            "status": det.get("status", ""),
            "timestamp": det.get("last_behavior", ""),
        })
    return results


def check_sensor_versions(hosts_data):
    """Audit sensor version compliance across fleet."""
    versions = {}
    for host in hosts_data:
        ver = host.get("sensor_version", "unknown")
        versions[ver] = versions.get(ver, 0) + 1
    return {"version_distribution": versions, "total_hosts": len(hosts_data)}


def run_audit(client_id, client_secret):
    """Execute CrowdStrike EDR audit."""
    print(f"\n{'='*60}")
    print(f"  CROWDSTRIKE EDR DEPLOYMENT AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    hosts_data = list_hosts(client_id, client_secret)
    print(f"--- MANAGED HOSTS ({len(hosts_data)}) ---")
    for h in hosts_data[:10]:
        print(f"  {h['hostname']}: {h['platform']} v{h['sensor_version']} ({h['status']})")

    versions = check_sensor_versions(hosts_data)
    print(f"\n--- SENSOR VERSIONS ---")
    for ver, count in sorted(versions["version_distribution"].items()):
        print(f"  {ver}: {count} hosts")

    detections = get_detections(client_id, client_secret)
    print(f"\n--- RECENT DETECTIONS ({len(detections)}) ---")
    for d in detections[:10]:
        print(f"  [{d['severity']}] {d['hostname']}: {d['tactic']} / {d['technique']}")

    return {"hosts": len(hosts_data), "versions": versions, "detections": detections}


def main():
    parser = argparse.ArgumentParser(description="CrowdStrike EDR Agent")
    parser.add_argument("--client-id", required=True, help="CrowdStrike API client ID")
    parser.add_argument("--client-secret", required=True, help="CrowdStrike API client secret")
    parser.add_argument("--audit", action="store_true", help="Run full audit")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    if args.audit:
        report = run_audit(args.client_id, args.client_secret)
        if args.output:
            with open(args.output, "w") as f:
                json.dump(report, f, indent=2, default=str)
            print(f"\n[+] Report saved to {args.output}")
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/deploying-edr-agent-with-crowdstrike/scripts/process.py =====
#!/usr/bin/env python3
"""
CrowdStrike Falcon Deployment Verification Tool

Queries the CrowdStrike Falcon API to verify sensor deployment coverage,
identify unmanaged endpoints, and generate deployment status reports.
"""

import json
import sys
import os
import time
import csv
from datetime import datetime, timedelta
from urllib.request import Request, urlopen
from urllib.parse import urlencode
from urllib.error import HTTPError


FALCON_BASE_URL = os.environ.get("FALCON_BASE_URL", "https://api.crowdstrike.com")
FALCON_CLIENT_ID = os.environ.get("FALCON_CLIENT_ID", "")
FALCON_CLIENT_SECRET = os.environ.get("FALCON_CLIENT_SECRET", "")


def get_oauth_token() -> str:
    """Obtain OAuth2 bearer token from CrowdStrike API."""
    url = f"{FALCON_BASE_URL}/oauth2/token"
    data = urlencode({
        "client_id": FALCON_CLIENT_ID,
        "client_secret": FALCON_CLIENT_SECRET,
    }).encode()

    req = Request(url, data=data, method="POST")
    req.add_header("Content-Type", "application/x-www-form-urlencoded")

    with urlopen(req) as resp:
        body = json.loads(resp.read())
        return body["access_token"]


def api_get(token: str, endpoint: str, params: dict = None) -> dict:
    """Make authenticated GET request to Falcon API."""
    url = f"{FALCON_BASE_URL}{endpoint}"
    if params:
        url += "?" + urlencode(params)

    req = Request(url, method="GET")
    req.add_header("Authorization", f"Bearer {token}")
    req.add_header("Accept", "application/json")

    with urlopen(req) as resp:
        return json.loads(resp.read())


def get_all_host_ids(token: str) -> list:
    """Retrieve all host device IDs from Falcon."""
    all_ids = []
    offset = 0
    limit = 5000

    while True:
        result = api_get(token, "/devices/queries/devices-scroll/v1", {
            "limit": limit,
            "offset": offset,
        })
        resources = result.get("resources", [])
        if not resources:
            break
        all_ids.extend(resources)
        offset += limit
        if len(resources) < limit:
            break

    return all_ids


def get_host_details(token: str, host_ids: list) -> list:
    """Retrieve detailed host information for given IDs (batches of 100)."""
    all_details = []

    for i in range(0, len(host_ids), 100):
        batch = host_ids[i:i + 100]
        url = f"{FALCON_BASE_URL}/devices/entities/devices/v2"
        data = json.dumps({"ids": batch}).encode()

        req = Request(url, data=data, method="POST")
        req.add_header("Authorization", f"Bearer {token}")
        req.add_header("Content-Type", "application/json")

        with urlopen(req) as resp:
            body = json.loads(resp.read())
            all_details.extend(body.get("resources", []))

    return all_details


def analyze_deployment(hosts: list) -> dict:
    """Analyze deployment coverage and sensor health."""
    now = datetime.utcnow()
    stale_threshold = now - timedelta(days=7)

    analysis = {
        "total_hosts": len(hosts),
        "os_breakdown": {},
        "status_breakdown": {"online": 0, "offline": 0, "stale": 0},
        "sensor_versions": {},
        "rfm_hosts": [],
        "stale_hosts": [],
        "unprotected_hosts": [],
    }

    for host in hosts:
        platform = host.get("platform_name", "Unknown")
        analysis["os_breakdown"][platform] = analysis["os_breakdown"].get(platform, 0) + 1

        version = host.get("agent_version", "Unknown")
        analysis["sensor_versions"][version] = analysis["sensor_versions"].get(version, 0) + 1

        status = host.get("status", "unknown")
        last_seen = host.get("last_seen", "")

        if last_seen:
            try:
                last_seen_dt = datetime.fromisoformat(last_seen.replace("Z", "+00:00")).replace(tzinfo=None)
                if last_seen_dt < stale_threshold:
                    analysis["status_breakdown"]["stale"] += 1
                    analysis["stale_hosts"].append({
                        "hostname": host.get("hostname", ""),
                        "last_seen": last_seen,
                        "platform": platform,
                    })
                elif status == "normal":
                    analysis["status_breakdown"]["online"] += 1
                else:
                    analysis["status_breakdown"]["offline"] += 1
            except (ValueError, TypeError):
                analysis["status_breakdown"]["offline"] += 1

        reduced_functionality = host.get("reduced_functionality_mode", "no")
        if reduced_functionality == "yes":
            analysis["rfm_hosts"].append({
                "hostname": host.get("hostname", ""),
                "reason": host.get("device_policies", {}).get("prevention", {}).get("policy_type", "unknown"),
            })

        prevention_policy = host.get("device_policies", {}).get("prevention", {})
        if not prevention_policy.get("applied", False):
            analysis["unprotected_hosts"].append({
                "hostname": host.get("hostname", ""),
                "platform": platform,
                "reason": "Prevention policy not applied",
            })

    return analysis


def generate_deployment_report(analysis: dict, output_path: str) -> None:
    """Generate deployment status report."""
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "deployment_summary": {
            "total_managed_hosts": analysis["total_hosts"],
            "online": analysis["status_breakdown"]["online"],
            "offline": analysis["status_breakdown"]["offline"],
            "stale_7_days": analysis["status_breakdown"]["stale"],
            "in_rfm": len(analysis["rfm_hosts"]),
            "unprotected": len(analysis["unprotected_hosts"]),
        },
        "os_distribution": analysis["os_breakdown"],
        "sensor_version_distribution": analysis["sensor_versions"],
        "hosts_requiring_attention": {
            "stale_hosts": analysis["stale_hosts"][:50],
            "rfm_hosts": analysis["rfm_hosts"][:50],
            "unprotected_hosts": analysis["unprotected_hosts"][:50],
        },
    }

    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2)


def export_stale_hosts_csv(stale_hosts: list, output_path: str) -> None:
    """Export stale hosts to CSV for remediation tracking."""
    with open(output_path, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow(["Hostname", "Platform", "Last Seen", "Action Required"])
        for host in stale_hosts:
            writer.writerow([
                host["hostname"],
                host.get("platform", ""),
                host["last_seen"],
                "Investigate connectivity / reinstall sensor",
            ])


if __name__ == "__main__":
    if not FALCON_CLIENT_ID or not FALCON_CLIENT_SECRET:
        print("Error: Set FALCON_CLIENT_ID and FALCON_CLIENT_SECRET environment variables")
        print()
        print("Required environment variables:")
        print("  FALCON_CLIENT_ID     - API client ID from Falcon Console")
        print("  FALCON_CLIENT_SECRET - API client secret")
        print("  FALCON_BASE_URL      - (Optional) API base URL (default: https://api.crowdstrike.com)")
        sys.exit(1)

    print("Authenticating with CrowdStrike Falcon API...")
    token = get_oauth_token()

    print("Retrieving host inventory...")
    host_ids = get_all_host_ids(token)
    print(f"Found {len(host_ids)} managed hosts")

    print("Fetching host details...")
    hosts = get_host_details(token, host_ids)

    print("Analyzing deployment coverage...")
    analysis = analyze_deployment(hosts)

    report_path = "falcon_deployment_report.json"
    generate_deployment_report(analysis, report_path)
    print(f"\nDeployment report: {report_path}")

    if analysis["stale_hosts"]:
        csv_path = "falcon_stale_hosts.csv"
        export_stale_hosts_csv(analysis["stale_hosts"], csv_path)
        print(f"Stale hosts CSV: {csv_path}")

    print(f"\n--- Deployment Summary ---")
    print(f"Total hosts: {analysis['total_hosts']}")
    print(f"Online: {analysis['status_breakdown']['online']}")
    print(f"Offline: {analysis['status_breakdown']['offline']}")
    print(f"Stale (>7 days): {analysis['status_breakdown']['stale']}")
    print(f"In RFM: {len(analysis['rfm_hosts'])}")
    print(f"Unprotected: {len(analysis['unprotected_hosts'])}")
    print(f"\nOS Distribution: {json.dumps(analysis['os_breakdown'], indent=2)}")
    print(f"Sensor Versions: {json.dumps(analysis['sensor_versions'], indent=2)}")

===== END FILE =====


===== FILE: skills/deploying-osquery-for-endpoint-monitoring/SKILL.md =====
---
name: deploying-osquery-for-endpoint-monitoring
description: 'Deploys and configures osquery for real-time endpoint monitoring using
  SQL-based queries to inspect running processes, open ports, installed software,
  and system configuration. Use when building visibility into endpoint state, threat
  hunting across fleet, or implementing compliance monitoring. Activates for requests
  involving osquery deployment, endpoint visibility, fleet management, or SQL-based
  endpoint querying.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- osquery
- endpoint-monitoring
- threat-hunting
- fleet-management
mitre_attack:
- T1547.001
- T1053.005
- T1543.003
- T1057
- T1071.001
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
---
# Deploying Osquery for Endpoint Monitoring

## When to Use

Use this skill when:
- Deploying osquery across Windows, macOS, and Linux endpoints for fleet-wide visibility
- Building threat hunting queries using osquery's SQL interface
- Monitoring endpoint compliance (installed software, open ports, running services)
- Integrating osquery data with SIEM or Kolide/Fleet for centralized management

**Do not use** for real-time alerting (osquery is periodic/on-demand; use EDR for real-time).

## Prerequisites

- Osquery package for target OS (https://osquery.io/downloads)
- Fleet management server (Kolide Fleet or FleetDM) for enterprise deployment
- TLS certificates for secure agent-to-server communication
- Log aggregation pipeline (Filebeat, Fluentd) for osquery result logs

## Workflow

### Step 1: Install Osquery

```bash
# Ubuntu/Debian
export OSQUERY_KEY=1484120AC4E9F8A1A577AEEE97A80C63C9D8B80B
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys $OSQUERY_KEY
add-apt-repository 'deb [arch=amd64] https://pkg.osquery.io/deb deb main'
apt-get update && apt-get install osquery -y

# Windows (MSI)
# Download from https://osquery.io/downloads/official
msiexec /i osquery-5.12.1.msi /quiet

# macOS
brew install osquery
```

### Step 2: Configure Osquery

```json
// /etc/osquery/osquery.conf (Linux/macOS) or C:\ProgramData\osquery\osquery.conf
{
  "options": {
    "config_plugin": "filesystem",
    "logger_plugin": "filesystem",
    "logger_path": "/var/log/osquery",
    "disable_logging": "false",
    "schedule_splay_percent": "10",
    "events_expiry": "3600",
    "verbose": "false",
    "worker_threads": "2",
    "enable_monitor": "true",
    "disable_events": "false",
    "disable_audit": "false",
    "audit_allow_config": "true",
    "host_identifier": "hostname",
    "enable_syslog": "true"
  },
  "schedule": {
    "process_monitor": {
      "query": "SELECT pid, name, path, cmdline, uid, parent FROM processes WHERE on_disk = 0;",
      "interval": 300,
      "description": "Detect processes running without on-disk binary (fileless)"
    },
    "listening_ports": {
      "query": "SELECT DISTINCT p.name, p.path, lp.port, lp.protocol, lp.address FROM listening_ports lp JOIN processes p ON lp.pid = p.pid WHERE lp.port != 0;",
      "interval": 600,
      "description": "Monitor listening network ports"
    },
    "persistence_check": {
      "query": "SELECT name, path, source FROM startup_items;",
      "interval": 3600,
      "description": "Monitor persistence mechanisms"
    },
    "installed_packages": {
      "query": "SELECT name, version, source FROM deb_packages;",
      "interval": 86400,
      "description": "Daily software inventory"
    },
    "users_and_groups": {
      "query": "SELECT u.username, u.uid, u.gid, u.shell, u.directory FROM users u WHERE u.uid >= 1000;",
      "interval": 3600
    },
    "crontab_monitor": {
      "query": "SELECT * FROM crontab;",
      "interval": 3600,
      "description": "Monitor scheduled tasks"
    },
    "suid_binaries": {
      "query": "SELECT path, username, permissions FROM suid_bin;",
      "interval": 86400,
      "description": "Detect SUID binaries"
    }
  },
  "packs": {
    "incident-response": "/usr/share/osquery/packs/incident-response.conf",
    "ossec-rootkit": "/usr/share/osquery/packs/ossec-rootkit.conf",
    "vuln-management": "/usr/share/osquery/packs/vuln-management.conf"
  }
}
```

### Step 3: Threat Hunting Queries

```sql
-- Detect processes with no on-disk binary (potential fileless malware)
SELECT pid, name, path, cmdline FROM processes WHERE on_disk = 0;

-- Find listening ports not associated with known services
SELECT lp.port, lp.protocol, p.name, p.path
FROM listening_ports lp JOIN processes p ON lp.pid = p.pid
WHERE lp.port NOT IN (22, 80, 443, 3306, 5432);

-- Detect unauthorized SSH keys
SELECT * FROM authorized_keys WHERE NOT key LIKE '%admin-team%';

-- Find recently modified system binaries
SELECT path, mtime, size FROM file
WHERE path LIKE '/usr/bin/%' AND mtime > (strftime('%s', 'now') - 86400);

-- Detect processes connecting to external IPs
SELECT DISTINCT p.name, p.path, pn.remote_address, pn.remote_port
FROM process_open_sockets pn JOIN processes p ON pn.pid = p.pid
WHERE pn.remote_address NOT LIKE '10.%'
  AND pn.remote_address NOT LIKE '172.16.%'
  AND pn.remote_address NOT LIKE '192.168.%'
  AND pn.remote_address != '127.0.0.1'
  AND pn.remote_address != '0.0.0.0';

-- Windows: Detect unsigned running executables
SELECT p.name, p.path, a.result AS signature_status
FROM processes p JOIN authenticode a ON p.path = a.path
WHERE a.result != 'trusted';
```

### Step 4: Deploy FleetDM for Centralized Management

```bash
# FleetDM provides centralized osquery management
# Deploy FleetDM server, configure agents to report to it
# Agents use TLS enrollment and config from Fleet

# Agent configuration for Fleet:
# --tls_hostname=fleet.corp.com
# --tls_server_certs=/etc/osquery/fleet.pem
# --enroll_secret_path=/etc/osquery/enroll_secret
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Osquery** | Open-source endpoint agent that exposes OS state as SQL tables for querying |
| **Schedule** | Periodic queries that run at defined intervals and log results |
| **Pack** | Collection of related queries grouped for specific use cases (IR, compliance) |
| **FleetDM** | Open-source osquery fleet management platform |
| **Differential Results** | Osquery logs only changes between query executions, reducing data volume |

## Tools & Systems

- **Osquery**: https://osquery.io/ - endpoint visibility agent
- **FleetDM**: https://fleetdm.com/ - centralized fleet management
- **Kolide**: Cloud-based osquery management with Slack integration
- **osquery-go**: Go client library for osquery extensions

## Common Pitfalls

- **Query performance**: Complex queries with large table scans impact endpoint performance. Use WHERE clauses and test query cost with `EXPLAIN`.
- **Schedule intervals too aggressive**: Running heavy queries every 60 seconds causes CPU spikes. Use 300-3600 second intervals for most queries.
- **Not using differential mode**: Without differential logging, osquery logs all results every interval. Differential mode logs only changes.
- **Missing event tables**: Some osquery tables require events framework enabled (process_events, socket_events). Enable with `--disable_events=false`.

===== END FILE =====


===== FILE: skills/deploying-osquery-for-endpoint-monitoring/LICENSE =====

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


===== FILE: skills/deploying-osquery-for-endpoint-monitoring/assets/template.md =====
# Osquery Deployment Template

## Deployment Info
| Field | Value |
|-------|-------|
| Osquery Version | |
| Fleet Server | |
| Target Endpoints | |
| Enrollment Secret | [stored securely] |

## Scheduled Queries
| Query Name | Interval | Description | Status |
|-----------|----------|-------------|--------|
| process_monitor | 300s | Fileless process detection | Active |
| listening_ports | 600s | Open port monitoring | Active |
| persistence_check | 3600s | Startup item monitoring | Active |

## Sign-Off
| Role | Name | Date |
|------|------|------|
| Security | | |
| IT Ops | | |

===== END FILE =====


===== FILE: skills/deploying-osquery-for-endpoint-monitoring/references/api-reference.md =====
# osquery Endpoint Monitoring — API Reference

## Installation

| Platform | Command |
|----------|---------|
| macOS | `brew install osquery` |
| Ubuntu | `apt install osquery` |
| Windows | MSI installer from osquery.io |

## Key osquery Tables

| Table | Description |
|-------|-------------|
| `processes` | Running processes with pid, name, cmdline, uid |
| `listening_ports` | Open network ports with bound process |
| `suid_bin` | SUID/SGID binaries on the system |
| `crontab` | Scheduled cron jobs |
| `authorized_keys` | SSH authorized keys per user |
| `kernel_modules` | Loaded kernel modules |
| `docker_containers` | Docker container status |
| `startup_items` | Boot/login startup items |
| `file` | File metadata, hashes, timestamps |

## Fleet API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/fleet/hosts` | List enrolled hosts |
| GET | `/api/v1/fleet/hosts/{id}` | Host details |
| POST | `/api/v1/fleet/queries` | Create scheduled query |
| GET | `/api/v1/fleet/queries` | List queries |

## osquery CLI

```bash
osqueryi --json "SELECT * FROM processes LIMIT 5"
osqueryctl start   # Start osquery daemon
osqueryctl config-check  # Validate configuration
```

## External References

- [osquery Schema](https://osquery.io/schema/)
- [Fleet Documentation](https://fleetdm.com/docs)
- [osquery Packs](https://github.com/osquery/osquery/tree/master/packs)

===== END FILE =====


===== FILE: skills/deploying-osquery-for-endpoint-monitoring/references/standards.md =====
# Standards & References
- **Osquery Documentation**: https://osquery.readthedocs.io/
- **Osquery Schema**: https://osquery.io/schema/
- **FleetDM Documentation**: https://fleetdm.com/docs
- **NIST SP 800-53 SI-4**: System Monitoring - osquery provides endpoint visibility
- **CIS Control 1**: Inventory of Enterprise Assets - osquery software/hardware inventory
- **CIS Control 2**: Software Inventory - osquery package/process queries

===== END FILE =====


===== FILE: skills/deploying-osquery-for-endpoint-monitoring/references/workflows.md =====
# Workflows

## Workflow 1: Osquery Fleet Deployment
```
[Install FleetDM server] → [Generate enrollment secret]
  → [Package osquery with fleet config] → [Deploy to pilot group]
  → [Verify enrollment and scheduled queries] → [Deploy to production]
  → [Create dashboards from query results] → [Ongoing monitoring]
```

## Workflow 2: Threat Hunt with Osquery
```
[Define hypothesis] → [Write SQL query targeting hypothesis]
  → [Execute via FleetDM live query across fleet]
  → [Analyze results] → [Investigate anomalies]
  → [Document findings] → [Create scheduled detection if recurrent]
```

===== END FILE =====


===== FILE: skills/deploying-osquery-for-endpoint-monitoring/scripts/agent.py =====
#!/usr/bin/env python3
"""osquery endpoint monitoring agent for security auditing."""

import json
import argparse
import subprocess
from datetime import datetime


SECURITY_QUERIES = {
    "listening_ports": "SELECT p.pid, p.name, l.port, l.protocol, l.address FROM listening_ports l JOIN processes p ON l.pid = p.pid WHERE l.port != 0",
    "suid_binaries": "SELECT path, username, permissions FROM suid_bin",
    "crontab_entries": "SELECT command, path, event FROM crontab",
    "authorized_keys": "SELECT uid, username, key_file FROM authorized_keys",
    "logged_in_users": "SELECT user, host, type, time FROM logged_in_users",
    "kernel_modules": "SELECT name, size, used_by, status FROM kernel_modules WHERE status = 'Live'",
    "processes_high_cpu": "SELECT pid, name, uid, resident_size, percent_processor_time FROM processes WHERE percent_processor_time > 50",
    "docker_containers": "SELECT id, name, image, status, started_at FROM docker_containers",
    "browser_extensions": "SELECT name, identifier, version, path, browser_type FROM chrome_extensions UNION ALL SELECT name, identifier, version, path, browser_type FROM firefox_addons",
    "startup_items": "SELECT name, path, source FROM startup_items",
}


def run_osquery(query, output_format="json"):
    """Execute osquery and return results."""
    cmd = ["osqueryi", f"--{output_format}", query]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        if output_format == "json" and result.stdout.strip():
            return json.loads(result.stdout)
        return [{"raw": result.stdout[:1000]}]
    except FileNotFoundError:
        return [{"error": "osquery not installed. Install from https://osquery.io/downloads/"}]
    except (json.JSONDecodeError, subprocess.TimeoutExpired) as e:
        return [{"error": str(e)}]


def check_fleet_status(fleet_url, api_token):
    """Check Fleet server host enrollment status."""
    import requests
    headers = {"Authorization": f"Bearer {api_token}"}
    try:
        resp = requests.get(f"{fleet_url}/api/v1/fleet/hosts", headers=headers, timeout=10)
        resp.raise_for_status()
        hosts = resp.json().get("hosts", [])
        return [{
            "hostname": h.get("hostname", ""),
            "platform": h.get("platform", ""),
            "osquery_version": h.get("osquery_version", ""),
            "status": h.get("status", ""),
            "last_seen": h.get("seen_time", ""),
        } for h in hosts]
    except Exception as e:
        return [{"error": str(e)}]


def run_audit(queries=None, fleet_url=None, api_token=None):
    """Execute osquery security audit."""
    print(f"\n{'='*60}")
    print(f"  OSQUERY ENDPOINT MONITORING AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    selected = queries or list(SECURITY_QUERIES.keys())
    results = {}
    for name in selected:
        if name in SECURITY_QUERIES:
            data = run_osquery(SECURITY_QUERIES[name])
            results[name] = data
            count = len(data) if isinstance(data, list) else 0
            print(f"--- {name.upper()} ({count} results) ---")
            for row in (data[:5] if isinstance(data, list) else []):
                if "error" not in row:
                    print(f"  {json.dumps(row)[:100]}")

    if fleet_url and api_token:
        fleet = check_fleet_status(fleet_url, api_token)
        results["fleet_hosts"] = fleet
        print(f"\n--- FLEET HOSTS ({len(fleet)}) ---")
        for h in fleet[:10]:
            if "error" not in h:
                print(f"  {h['hostname']}: {h['platform']} ({h['status']})")

    return results


def main():
    parser = argparse.ArgumentParser(description="osquery Monitoring Agent")
    parser.add_argument("--queries", nargs="+", choices=list(SECURITY_QUERIES.keys()),
                        help="Specific queries to run")
    parser.add_argument("--fleet-url", help="Fleet server URL")
    parser.add_argument("--api-token", help="Fleet API token")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_audit(args.queries, args.fleet_url, args.api_token)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/deploying-osquery-for-endpoint-monitoring/scripts/process.py =====
#!/usr/bin/env python3
"""Osquery Results Analyzer - Parses osquery JSON results for anomaly detection."""

import json
import sys
import os
from collections import Counter, defaultdict
from datetime import datetime


def parse_osquery_results(json_path: str) -> list:
    """Parse osquery result log (JSON lines format)."""
    results = []
    with open(json_path, "r") as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            try:
                entry = json.loads(line)
                results.append(entry)
            except json.JSONDecodeError:
                continue
    return results


def analyze_results(results: list) -> dict:
    """Analyze osquery results for security anomalies."""
    analysis = {
        "total_entries": len(results),
        "queries": Counter(),
        "hosts": Counter(),
        "added_items": [],
        "removed_items": [],
    }

    for entry in results:
        name = entry.get("name", "unknown")
        analysis["queries"][name] += 1
        analysis["hosts"][entry.get("hostIdentifier", "unknown")] += 1

        action = entry.get("action", "")
        columns = entry.get("columns", {})

        if action == "added":
            analysis["added_items"].append({
                "query": name,
                "host": entry.get("hostIdentifier", ""),
                "timestamp": entry.get("unixTime", ""),
                "data": columns,
            })
        elif action == "removed":
            analysis["removed_items"].append({
                "query": name,
                "host": entry.get("hostIdentifier", ""),
                "data": columns,
            })

    return analysis


def generate_report(analysis: dict, output_path: str) -> None:
    """Generate osquery analysis report."""
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "total_entries": analysis["total_entries"],
        "queries_executed": dict(analysis["queries"]),
        "hosts_reporting": dict(analysis["hosts"].most_common(50)),
        "new_items_detected": len(analysis["added_items"]),
        "items_removed": len(analysis["removed_items"]),
        "recent_additions": analysis["added_items"][:50],
    }
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <osqueryd.results.log>")
        sys.exit(1)
    results = parse_osquery_results(sys.argv[1])
    analysis = analyze_results(results)
    out = os.path.join(os.path.dirname(sys.argv[1]) or ".", "osquery_analysis.json")
    generate_report(analysis, out)
    print(f"Entries: {analysis['total_entries']} | New items: {len(analysis['added_items'])}")

===== END FILE =====


===== FILE: skills/detecting-evasion-techniques-in-endpoint-logs/SKILL.md =====
---
name: detecting-evasion-techniques-in-endpoint-logs
description: 'Detects defense evasion techniques used by adversaries in endpoint logs
  including log tampering, timestomping, process injection, and security tool disabling.
  Use when investigating suspicious endpoint behavior, building detection rules for
  evasion tactics, or conducting threat hunting for stealthy adversary activity. Activates
  for requests involving evasion detection, defense evasion analysis, log tampering
  detection, or MITRE ATT&CK TA0005.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- edr
- threat-hunting
- defense-evasion
- MITRE-ATT&CK
- detection-engineering
version: 1.0.0
author: mahipal
license: Apache-2.0
d3fend_techniques:
- File Metadata Consistency Validation
- Content Format Conversion
- File Content Analysis
- Platform Hardening
- File Format Verification
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
- T1027
---
# Detecting Evasion Techniques in Endpoint Logs

## When to Use

Use this skill when:
- Hunting for adversary defense evasion techniques (MITRE ATT&CK TA0005) in endpoint telemetry
- Building detection rules for common evasion methods (process injection, timestomping, log clearing)
- Investigating incidents where adversaries disabled or bypassed security tools
- Analyzing endpoint logs for indicators of living-off-the-land binary (LOLBin) abuse

**Do not use** this skill for network-level evasion (use network traffic analysis) or for malware reverse engineering.

## Prerequisites

- Sysmon installed and configured with comprehensive logging rules (SwiftOnSecurity or Olaf Hartong config)
- Windows Security Event Log with advanced audit policy enabled
- EDR telemetry (CrowdStrike, SentinelOne, Microsoft Defender for Endpoint)
- SIEM platform for log correlation (Splunk, Elastic, Sentinel)
- MITRE ATT&CK Enterprise matrix for technique reference

## Workflow

### Step 1: Detect Log Tampering (T1070)

**Windows Event Log clearing (T1070.001)**:
```
# Sysmon Event ID 1 (Process Create) for wevtutil
EventID: 1
CommandLine contains: "wevtutil cl" OR "wevtutil clear-log"

# Security Event ID 1102 - Audit log was cleared
EventID: 1102
Source: Microsoft-Windows-Eventlog

# System Event ID 104 - Event log was cleared
EventID: 104

# PowerShell log clearing
EventID: 1 (Sysmon)
CommandLine contains: "Clear-EventLog" OR "Remove-EventLog"

# Splunk query:
index=windows (EventCode=1102 OR EventCode=104)
  OR (EventCode=1 CommandLine="*wevtutil*cl*")
  OR (EventCode=1 CommandLine="*Clear-EventLog*")
| table _time host user CommandLine EventCode
```

**Timestomping (T1070.006)**:
```
# Sysmon Event ID 2 - File creation time changed
EventID: 2
# Look for creation times set far in the past on recently-written files
# Correlate with Event ID 11 (FileCreate) - if FileCreate is recent but
# creation time in Event ID 2 is old, timestomping is likely

# MDE Advanced Hunting (KQL):
DeviceFileEvents
| where ActionType == "FileTimestampModified"
| where Timestamp > ago(7d)
| extend TimeDiff = datetime_diff('day', Timestamp, ReportedFileCreationTime)
| where TimeDiff > 30
| project Timestamp, DeviceName, FileName, FolderPath,
    ReportedFileCreationTime, InitiatingProcessFileName
```

### Step 2: Detect Process Injection (T1055)

```
# Sysmon Event ID 8 - CreateRemoteThread
EventID: 8
# Alert when source process is unusual (not system processes)
# Filter out known legitimate: antivirus, debugging tools
SourceImage NOT IN ("C:\Windows\System32\csrss.exe",
                     "C:\Windows\System32\lsass.exe")

# Sysmon Event ID 10 - ProcessAccess with suspicious access masks
EventID: 10
GrantedAccess contains: "0x1F0FFF" OR "0x1FFFFF" OR "0x001F0FFF"
# PROCESS_ALL_ACCESS = 0x1F0FFF (common in injection)
# Filter legitimate: AV accessing all processes

# Sysmon Event ID 25 - Process Tampering
EventID: 25
Type: "Image is replaced"  # Process hollowing indicator

# Splunk detection:
index=sysmon EventCode=8
| where NOT match(SourceImage, "(?i)(csrss|svchost|MsMpEng|defender)")
| stats count by SourceImage TargetImage host
| where count < 5
| sort - count
```

### Step 3: Detect Security Tool Disabling (T1562)

```
# Service stopped events for security services
EventID: 7045 (new service) OR 7036 (service state change)
ServiceName IN ("WinDefend", "Sense", "CrowdStrike Falcon Sensor",
                 "SentinelAgent", "csagent", "MBAMService")

# Sysmon Event ID 1 - Processes that disable Defender
CommandLine contains: "Set-MpPreference -DisableRealtimeMonitoring"
  OR "sc stop WinDefend"
  OR "sc config WinDefend start= disabled"
  OR "net stop" AND ("windefend" OR "sense" OR "csagent")

# Registry modification to disable security features
# Sysmon Event ID 13 - Registry value set
TargetObject contains: "DisableAntiSpyware"
  OR "DisableRealtimeMonitoring"
  OR "DisableBehaviorMonitoring"
Details: "DWORD (0x00000001)"

# MDE KQL:
DeviceRegistryEvents
| where RegistryValueName in ("DisableAntiSpyware", "DisableRealtimeMonitoring")
| where RegistryValueData == "1"
| project Timestamp, DeviceName, RegistryKey, InitiatingProcessFileName
```

### Step 4: Detect Masquerading (T1036)

```
# Sysmon Event ID 1 - Process with legitimate name from unusual path
EventID: 1
Image contains: "svchost.exe" AND Image NOT starts with: "C:\Windows\System32\"
Image contains: "csrss.exe" AND Image NOT starts with: "C:\Windows\System32\"
Image contains: "lsass.exe" AND Image NOT starts with: "C:\Windows\System32\"

# Process name mismatch (original filename vs. current name)
# Sysmon captures OriginalFileName from PE header
EventID: 1
OriginalFileName != (parsed filename from Image path)

# Double extension files
EventID: 11 (FileCreate)
TargetFilename matches: "*\.pdf\.exe" OR "*\.doc\.exe" OR "*\.jpg\.exe"

# Splunk:
index=sysmon EventCode=1
| eval process_name=mvindex(split(Image,"\\"),-1)
| where (process_name="svchost.exe" AND NOT match(Image,"(?i)C:\\\\Windows\\\\System32"))
  OR (process_name="csrss.exe" AND NOT match(Image,"(?i)C:\\\\Windows\\\\System32"))
| table _time host Image ParentImage CommandLine User
```

### Step 5: Detect LOLBin Abuse (T1218, T1127)

```
# Common LOLBin abuse patterns:

# mshta.exe executing remote content
EventID: 1
Image ends with: "mshta.exe"
CommandLine contains: "http" OR "javascript:" OR "vbscript:"

# certutil.exe downloading files
EventID: 1
Image ends with: "certutil.exe"
CommandLine contains: "-urlcache" OR "-decode" OR "-encode"

# regsvr32.exe executing scriptlets
EventID: 1
Image ends with: "regsvr32.exe"
CommandLine contains: "/s /n /u /i:" OR "scrobj.dll"

# rundll32.exe with unusual DLLs
EventID: 1
Image ends with: "rundll32.exe"
CommandLine contains: "javascript:" OR ".js" OR "http:"

# MSBuild executing inline tasks
EventID: 1
Image contains: "MSBuild.exe"
CommandLine NOT contains: ".sln" AND NOT contains: ".csproj"
```

### Step 6: Build Detection Rule Correlation

```
# Combine multiple weak signals into high-confidence detection:

# Rule: Potential post-exploitation evasion chain
# Trigger when 3+ evasion techniques observed on same host within 1 hour

# Splunk correlation search:
index=sysmon host=*
| eval technique=case(
    EventCode=2, "timestomping",
    EventCode=8 AND NOT match(SourceImage,"csrss|svchost"), "process_injection",
    EventCode=1 AND match(CommandLine,"(?i)wevtutil.*cl"), "log_clearing",
    EventCode=13 AND match(TargetObject,"DisableRealtimeMonitoring"), "security_disable",
    EventCode=1 AND match(CommandLine,"(?i)(mshta|certutil.*urlcache|regsvr32.*/s.*/n)"), "lolbin_abuse",
    true(), NULL
)
| where isnotnull(technique)
| bin _time span=1h
| stats dc(technique) as technique_count values(technique) as techniques by host _time
| where technique_count >= 3
| sort - technique_count
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Defense Evasion (TA0005)** | MITRE ATT&CK tactic where adversaries attempt to avoid detection during operations |
| **Process Injection (T1055)** | Technique of injecting code into another process's memory space to execute in a trusted context |
| **Timestomping (T1070.006)** | Modifying file timestamps to make malicious files appear old and blend with legitimate files |
| **Masquerading (T1036)** | Naming malicious files or processes to match legitimate system files to avoid detection |
| **LOLBin** | Living Off the Land Binary; legitimate Windows tool repurposed by adversaries |
| **Indicator Removal (T1070)** | Clearing logs, deleting files, or modifying artifacts to remove evidence of compromise |

## Tools & Systems

- **Sysmon**: Advanced Windows system monitoring with kernel-level visibility
- **Microsoft Defender for Endpoint**: EDR with advanced hunting (KQL) for evasion detection
- **CrowdStrike Falcon**: IOA-based behavioral detection for evasion techniques
- **Elastic Security**: SIEM with prebuilt detection rules for ATT&CK evasion techniques
- **Sigma Rules**: Vendor-agnostic detection rule format with extensive evasion rule library

## Common Pitfalls

- **Alert fatigue from process injection rules**: Many legitimate tools (AV, accessibility) perform process injection. Maintain an allowlist of known-good source processes.
- **Missing Sysmon Event ID 8/10**: Default Sysmon configurations may not capture CreateRemoteThread or ProcessAccess. Use a comprehensive Sysmon config.
- **Ignoring parent process context**: A suspicious command line from cmd.exe is concerning only if the parent of cmd.exe is unusual (e.g., Excel spawning cmd.exe).
- **Not correlating across event types**: Single events are often benign. Combine multiple weak signals (process creation + network connection + file creation) for high-confidence detections.

===== END FILE =====


===== FILE: skills/detecting-evasion-techniques-in-endpoint-logs/LICENSE =====

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


===== FILE: skills/detecting-evasion-techniques-in-endpoint-logs/assets/template.md =====
# Evasion Detection Hunt Template

## Hunt Information

| Field | Value |
|-------|-------|
| Hunt Name | |
| Target Technique | MITRE ATT&CK ID |
| Hypothesis | |
| Data Sources | Sysmon / Windows Security / EDR |
| Time Range | |
| Analyst | |

## Detection Queries

| SIEM | Query | Expected Results |
|------|-------|-----------------|
| Splunk | | |
| Elastic KQL | | |
| MDE Advanced Hunting | | |

## Findings

| Timestamp | Host | Technique | Severity | Evidence | True/False Positive |
|-----------|------|-----------|----------|----------|-------------------|
| | | | | | |

## Tuning Actions

| Finding | Action | New Exclusion | Status |
|---------|--------|---------------|--------|
| | Allow / Investigate / Block | | |

## Sign-Off

| Role | Name | Date |
|------|------|------|
| Threat Hunter | | |
| SOC Lead | | |

===== END FILE =====


===== FILE: skills/detecting-evasion-techniques-in-endpoint-logs/references/api-reference.md =====
# API Reference: Detecting Evasion Techniques in Endpoint Logs

## Key Windows Event IDs for Evasion

| Event ID | Source | Evasion Technique |
|----------|--------|-------------------|
| 1102 | Security | Audit log cleared (T1070.001) |
| Sysmon 2 | Sysmon | Timestomping (T1070.006) |
| Sysmon 8 | Sysmon | CreateRemoteThread (T1055) |
| Sysmon 10 | Sysmon | Process Access / LSASS (T1003) |
| 4688 | Security | Process creation with cmdline |

## python-evtx Usage

```python
import Evtx.Evtx as evtx
with evtx.Evtx("Sysmon.evtx") as log:
    for record in log.records():
        xml = record.xml()
        # Parse EventID, CommandLine, SourceImage, TargetImage
```

## Evasion Detection Patterns

```python
# Log clearing
r"wevtutil\s+(cl|clear-log)"
r"Clear-EventLog"
# Security tool disable
r"Set-MpPreference\s+-DisableRealtimeMonitoring\s+\$true"
r"sc\s+(stop|delete)\s+WinDefend"
# AMSI bypass
r"[Ref].Assembly.GetType.*AMSI"
r"amsiInitFailed"
```

## MITRE ATT&CK TA0005 Techniques

| Technique | ID | Detection |
|-----------|----|-----------|
| Indicator Removal | T1070 | Log clearing, file deletion |
| Timestomping | T1070.006 | Sysmon Event ID 2 |
| Process Injection | T1055 | Sysmon Event ID 8 |
| Impair Defenses | T1562.001 | AV/EDR disabling commands |
| AMSI Bypass | T1562.001 | PowerShell AMSI patching |

## Splunk SPL Detection

```spl
index=sysmon (EventCode=2 OR EventCode=8 OR EventCode=10)
| eval technique=case(
    EventCode=2, "Timestomping",
    EventCode=8, "Process Injection",
    EventCode=10, "Process Access")
| stats count by technique, SourceImage, Computer
```

## CLI Usage

```bash
python agent.py --evtx-file Sysmon.evtx
python agent.py --evtx-file Security.evtx
```

===== END FILE =====


===== FILE: skills/detecting-evasion-techniques-in-endpoint-logs/references/standards.md =====
# Standards & References - Detecting Evasion Techniques in Endpoint Logs

## Primary Standards

### MITRE ATT&CK TA0005 - Defense Evasion
- **URL**: https://attack.mitre.org/tactics/TA0005/
- **Scope**: 43 techniques and 80+ sub-techniques for defense evasion
- **Key techniques**: T1055 (Process Injection), T1070 (Indicator Removal), T1036 (Masquerading), T1218 (System Binary Proxy Execution), T1562 (Impair Defenses)

### Sigma Detection Rules
- **URL**: https://github.com/SigmaHQ/sigma
- **Scope**: Community-maintained detection rules in vendor-agnostic format
- **Evasion rules**: rules/windows/process_creation/proc_creation_win_*, rules/windows/sysmon/

### LOLBAS Project
- **URL**: https://lolbas-project.github.io/
- **Scope**: Catalog of Windows binaries that can be used for defense evasion, with detection recommendations

## Compliance Mappings

| Framework | Requirement | Detection Coverage |
|-----------|------------|-------------------|
| NIST 800-53 | SI-4 System Monitoring | Evasion detection via endpoint logging |
| NIST 800-53 | AU-6 Audit Record Review | Log analysis for tampering indicators |
| PCI DSS 4.0 | 10.4.1 - Audit log review | Automated detection of log tampering |
| ISO 27001 | A.12.4.1 - Event logging | Integrity monitoring of security logs |

## Supporting References

- **Sysmon Configuration**: https://github.com/SwiftOnSecurity/sysmon-config
- **Olaf Hartong Sysmon Modular**: https://github.com/olafhartong/sysmon-modular
- **SANS Hunt Evil Poster**: Comprehensive guide to suspicious Windows process behavior
- **Elastic Detection Rules**: https://github.com/elastic/detection-rules

===== END FILE =====


===== FILE: skills/detecting-evasion-techniques-in-endpoint-logs/references/workflows.md =====
# Workflows - Detecting Evasion Techniques in Endpoint Logs

## Workflow 1: Evasion Technique Threat Hunt

```
[Select evasion technique to hunt]
    │
    ├── T1055 Process Injection
    ├── T1070 Log Tampering
    ├── T1036 Masquerading
    ├── T1562 Security Tool Disabling
    │
    ▼
[Craft detection query (Splunk/KQL/Elastic)]
    │
    ▼
[Execute across 30-90 days of endpoint telemetry]
    │
    ▼
[Triage results]
    │
    ├── Known-good (allowlist) ──► [Add to baseline, refine query]
    ├── Suspicious ──► [Deep investigation]
    │                       │
    │                       ├── Correlate with other telemetry
    │                       ├── Check process tree
    │                       ├── Review network connections
    │                       │
    │                       ├── True positive ──► [Escalate to IR]
    │                       └── False positive ──► [Tune detection]
    │
    └── No results ──► [Validate logging covers technique]
```

## Workflow 2: Detection Rule Deployment

```
[Create Sigma/SIEM detection rule]
    │
    ▼
[Test against historical data]
    │
    ├── High false positive rate ──► [Refine exclusions]
    │
    └── Acceptable FP rate ──► [Deploy in alert mode]
                                     │
                                     ▼
                                [Monitor for 2 weeks]
                                     │
                                     ▼
                                [Review alert quality]
                                     │
                                     ▼
                                [Promote to production detection]
```

## Workflow 3: Evasion Incident Response

```
[Evasion technique detected]
    │
    ▼
[Assess scope: Which endpoints affected?]
    │
    ▼
[Correlate with initial access and persistence]
    │
    ▼
[Determine if adversary achieved objectives]
    │
    ├── Active intrusion ──► [Full incident response]
    │
    └── Isolated event ──► [Remediate endpoint, enhance detection]
```

===== END FILE =====


===== FILE: skills/detecting-evasion-techniques-in-endpoint-logs/scripts/agent.py =====
#!/usr/bin/env python3
"""Defense evasion detection agent for endpoint logs.

Detects MITRE ATT&CK TA0005 evasion techniques including log clearing,
timestomping, process injection indicators, and security tool disabling
by analyzing Sysmon and Windows Security event logs.
"""

import argparse
import json
import re
from datetime import datetime

try:
    import Evtx.Evtx as evtx
except ImportError:
    evtx = None

EVASION_EVENT_IDS = {
    1102: {"name": "Audit Log Cleared", "severity": "CRITICAL", "mitre": "T1070.001"},
    4688: {"name": "Process Creation", "severity": "INFO", "mitre": "T1059"},
    4689: {"name": "Process Termination", "severity": "INFO", "mitre": ""},
}

SYSMON_EVASION_IDS = {
    1: "Process Create",
    2: "File creation time changed (Timestomping)",
    8: "CreateRemoteThread",
    10: "Process Access",
    12: "Registry Object Create/Delete",
    13: "Registry Value Set",
}

TIMESTOMP_INDICATORS = [
    r"SetFileTime", r"timestomp", r"\$STANDARD_INFORMATION",
    r"NtSetInformationFile", r"SetFileInformationByHandle",
]

LOG_CLEARING_COMMANDS = [
    r"wevtutil\s+(cl|clear-log)",
    r"Clear-EventLog",
    r"Remove-EventLog",
    r"del\s+.*\.evtx",
    r"wmic\s+nteventlog.*clear",
]

SECURITY_TOOL_DISABLE = [
    r"(Stop|Disable)-Service.*(Windows Defender|WinDefend|MsMpSvc)",
    r"Set-MpPreference\s+-DisableRealtimeMonitoring\s+\$true",
    r"sc\s+(stop|delete)\s+(WinDefend|MsMpSvc|Sense)",
    r"netsh\s+advfirewall\s+set\s+.*state\s+off",
    r"reg\s+add.*DisableAntiSpyware.*1",
    r"taskkill.*/im\s+(MsMpEng|avp|avgui|mbam)",
]

AMSI_BYPASS_PATTERNS = [
    r"amsi(Init|Scan)Buffer",
    r"AmsiUtils",
    r"amsiContext",
    r"[Ref].Assembly.GetType.*AMSI",
]


def analyze_evtx_for_evasion(filepath):
    if evtx is None:
        return {"error": "python-evtx not installed: pip install python-evtx"}
    findings = []
    with evtx.Evtx(filepath) as log:
        for record in log.records():
            xml = record.xml()
            event_id_match = re.search(r'<EventID[^>]*>(\d+)</EventID>', xml)
            if not event_id_match:
                continue
            event_id = int(event_id_match.group(1))
            time_match = re.search(r'SystemTime="([^"]+)"', xml)
            timestamp = time_match.group(1) if time_match else ""

            if event_id == 1102:
                findings.append({
                    "event_id": 1102, "timestamp": timestamp,
                    "severity": "CRITICAL", "mitre": "T1070.001",
                    "description": "Security audit log was cleared",
                })

            if event_id == 2:
                findings.append({
                    "event_id": 2, "timestamp": timestamp,
                    "severity": "HIGH", "mitre": "T1070.006",
                    "description": "File creation time modified (timestomping)",
                })

            if event_id == 8:
                source = re.search(r'<Data Name="SourceImage">([^<]+)', xml)
                target = re.search(r'<Data Name="TargetImage">([^<]+)', xml)
                findings.append({
                    "event_id": 8, "timestamp": timestamp,
                    "source": source.group(1) if source else "",
                    "target": target.group(1) if target else "",
                    "severity": "HIGH", "mitre": "T1055",
                    "description": "CreateRemoteThread detected (process injection)",
                })

            if event_id in (1, 4688):
                cmdline = re.search(r'<Data Name="CommandLine">([^<]+)', xml)
                if not cmdline:
                    cmdline = re.search(r'<Data Name="NewProcessName">([^<]+)', xml)
                if cmdline:
                    cmd = cmdline.group(1)
                    for pattern in LOG_CLEARING_COMMANDS:
                        if re.search(pattern, cmd, re.IGNORECASE):
                            findings.append({
                                "event_id": event_id, "timestamp": timestamp,
                                "command": cmd[:200], "severity": "CRITICAL",
                                "mitre": "T1070.001",
                                "description": "Log clearing command detected",
                            })
                    for pattern in SECURITY_TOOL_DISABLE:
                        if re.search(pattern, cmd, re.IGNORECASE):
                            findings.append({
                                "event_id": event_id, "timestamp": timestamp,
                                "command": cmd[:200], "severity": "CRITICAL",
                                "mitre": "T1562.001",
                                "description": "Security tool disabling detected",
                            })
                    for pattern in AMSI_BYPASS_PATTERNS:
                        if re.search(pattern, cmd, re.IGNORECASE):
                            findings.append({
                                "event_id": event_id, "timestamp": timestamp,
                                "command": cmd[:200], "severity": "HIGH",
                                "mitre": "T1562.001",
                                "description": "AMSI bypass attempt detected",
                            })
    return findings


def main():
    parser = argparse.ArgumentParser(description="Defense Evasion Detector")
    parser.add_argument("--evtx-file", required=True, help="EVTX file (Sysmon or Security)")
    args = parser.parse_args()

    findings = analyze_evtx_for_evasion(args.evtx_file)
    if isinstance(findings, dict) and "error" in findings:
        results = findings
    else:
        results = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "source_file": args.evtx_file,
            "findings": findings,
            "total_findings": len(findings),
            "by_severity": {},
        }
        for f in findings:
            sev = f.get("severity", "UNKNOWN")
            results["by_severity"][sev] = results["by_severity"].get(sev, 0) + 1

    print(json.dumps(results, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-evasion-techniques-in-endpoint-logs/scripts/process.py =====
#!/usr/bin/env python3
"""
Endpoint Evasion Technique Detector

Analyzes Windows event logs (exported as EVTX/CSV) for common defense
evasion techniques mapped to MITRE ATT&CK TA0005.
"""

import json
import csv
import re
import sys
import os
from collections import defaultdict
from datetime import datetime


EVASION_PATTERNS = {
    "T1070.001-log_clearing": {
        "name": "Indicator Removal: Clear Windows Event Logs",
        "severity": "high",
        "patterns": [
            r"wevtutil\s+(cl|clear-log)",
            r"Clear-EventLog",
            r"Remove-EventLog",
        ],
        "event_ids": ["1102", "104"],
    },
    "T1055-process_injection": {
        "name": "Process Injection",
        "severity": "high",
        "sysmon_event_ids": ["8", "10", "25"],
        "patterns": [
            r"VirtualAllocEx",
            r"WriteProcessMemory",
            r"CreateRemoteThread",
            r"NtMapViewOfSection",
        ],
    },
    "T1562.001-disable_security": {
        "name": "Impair Defenses: Disable or Modify Tools",
        "severity": "critical",
        "patterns": [
            r"Set-MpPreference\s+-Disable",
            r"sc\s+(stop|config)\s+(WinDefend|Sense|MBAMService)",
            r"net\s+stop\s+(windefend|sense|csagent)",
            r"DisableAntiSpyware",
            r"DisableRealtimeMonitoring",
        ],
    },
    "T1036-masquerading": {
        "name": "Masquerading",
        "severity": "medium",
        "suspicious_paths": {
            "svchost.exe": r"C:\\Windows\\System32\\svchost\.exe",
            "csrss.exe": r"C:\\Windows\\System32\\csrss\.exe",
            "lsass.exe": r"C:\\Windows\\System32\\lsass\.exe",
            "smss.exe": r"C:\\Windows\\System32\\smss\.exe",
            "services.exe": r"C:\\Windows\\System32\\services\.exe",
        },
    },
    "T1218-lolbin_abuse": {
        "name": "System Binary Proxy Execution",
        "severity": "high",
        "patterns": [
            r"mshta\.exe.*https?://",
            r"mshta\.exe.*javascript:",
            r"certutil\.exe.*-urlcache",
            r"certutil\.exe.*-decode",
            r"regsvr32\.exe.*/s.*/n.*/u.*/i:",
            r"rundll32\.exe.*javascript:",
            r"MSBuild\.exe(?!.*\.(sln|csproj|vbproj))",
            r"installutil\.exe.*/logfile=",
        ],
    },
    "T1070.006-timestomping": {
        "name": "Indicator Removal: Timestomp",
        "severity": "medium",
        "sysmon_event_ids": ["2"],
    },
}


def analyze_sysmon_csv(csv_path: str) -> list:
    """Analyze Sysmon events exported as CSV for evasion techniques."""
    detections = []

    with open(csv_path, "r", encoding="utf-8-sig") as f:
        reader = csv.DictReader(f)
        for row in reader:
            event_id = row.get("Event ID", row.get("EventID", ""))
            message = row.get("Message", row.get("Details", ""))
            command_line = row.get("CommandLine", "")
            image = row.get("Image", row.get("Process Name", ""))
            timestamp = row.get("Date and Time", row.get("TimeCreated", ""))

            full_text = f"{message} {command_line} {image}".lower()

            for technique_id, technique in EVASION_PATTERNS.items():
                detected = False
                detection_detail = ""

                if "event_ids" in technique and event_id in technique["event_ids"]:
                    detected = True
                    detection_detail = f"Event ID {event_id} detected"

                if "sysmon_event_ids" in technique and event_id in technique["sysmon_event_ids"]:
                    detected = True
                    detection_detail = f"Sysmon Event ID {event_id}"

                for pattern in technique.get("patterns", []):
                    if re.search(pattern, full_text, re.IGNORECASE):
                        detected = True
                        detection_detail = f"Pattern match: {pattern}"
                        break

                if "suspicious_paths" in technique and image:
                    image_lower = image.lower()
                    for proc_name, expected_path in technique["suspicious_paths"].items():
                        if proc_name in image_lower:
                            if not re.match(expected_path, image, re.IGNORECASE):
                                detected = True
                                detection_detail = f"Masquerading: {proc_name} from unexpected path {image}"

                if detected:
                    detections.append({
                        "timestamp": timestamp,
                        "technique_id": technique_id.split("-")[0],
                        "technique_name": technique["name"],
                        "severity": technique["severity"],
                        "event_id": event_id,
                        "process": image,
                        "command_line": command_line[:300],
                        "detail": detection_detail,
                        "host": row.get("Computer", row.get("host", "")),
                    })

    return detections


def generate_detection_report(detections: list, output_path: str) -> None:
    """Generate evasion detection report."""
    by_technique = defaultdict(list)
    by_severity = defaultdict(int)
    by_host = defaultdict(int)

    for d in detections:
        by_technique[d["technique_id"]].append(d)
        by_severity[d["severity"]] += 1
        by_host[d["host"]] += 1

    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "total_detections": len(detections),
        "summary": {
            "by_severity": dict(by_severity),
            "by_technique": {k: len(v) for k, v in by_technique.items()},
            "by_host": dict(by_host),
        },
        "detections": detections[:200],
    }

    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <sysmon_events.csv>")
        print()
        print("Analyzes exported Sysmon/Windows event logs for defense evasion techniques.")
        sys.exit(1)

    csv_path = sys.argv[1]
    if not os.path.exists(csv_path):
        print(f"Error: File not found: {csv_path}")
        sys.exit(1)

    print("Analyzing endpoint logs for evasion techniques...")
    detections = analyze_sysmon_csv(csv_path)

    base = os.path.splitext(os.path.basename(csv_path))[0]
    out_dir = os.path.dirname(csv_path) or "."

    report_path = os.path.join(out_dir, f"{base}_evasion_report.json")
    generate_detection_report(detections, report_path)
    print(f"Detection report: {report_path}")

    print(f"\n--- Evasion Detection Summary ---")
    print(f"Total detections: {len(detections)}")

    severity_counts = defaultdict(int)
    technique_counts = defaultdict(int)
    for d in detections:
        severity_counts[d["severity"]] += 1
        technique_counts[d["technique_id"]] += 1

    for sev in ["critical", "high", "medium", "low"]:
        if severity_counts[sev]:
            print(f"  {sev.upper()}: {severity_counts[sev]}")

    if technique_counts:
        print(f"\nBy technique:")
        for tid, count in sorted(technique_counts.items(), key=lambda x: -x[1]):
            print(f"  {tid}: {count}")

===== END FILE =====


===== FILE: skills/detecting-fileless-attacks-on-endpoints/SKILL.md =====
---
name: detecting-fileless-attacks-on-endpoints
description: 'Detects fileless malware and in-memory attacks that execute entirely
  in RAM without writing persistent files to disk, evading traditional antivirus.
  Use when building detections for PowerShell-based attacks, reflective DLL injection,
  WMI persistence, and registry-resident malware. Activates for requests involving
  fileless malware detection, in-memory attacks, PowerShell exploitation, or living-off-the-land
  techniques.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- fileless-malware
- memory-attacks
- PowerShell
- detection-engineering
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
- T1053
---
# Detecting Fileless Attacks on Endpoints

## When to Use

Use this skill when:
- Building detection rules for fileless malware that operates entirely in memory
- Hunting for PowerShell-based attacks, reflective DLL injection, and WMI abuse
- Configuring endpoint telemetry (Sysmon, AMSI, PowerShell logging) to capture fileless indicators
- Investigating incidents where traditional AV found no malicious files

**Do not use** for detecting file-based malware or for malware reverse engineering.

## Prerequisites

- Sysmon with process creation and WMI event logging enabled
- PowerShell Script Block Logging and Module Logging enabled
- AMSI (Antimalware Scan Interface) enabled for script content inspection
- EDR with behavioral detection capabilities (MDE, CrowdStrike, SentinelOne)

## Workflow

### Step 1: Enable Required Telemetry

```powershell
# Enable PowerShell Script Block Logging (GPO or registry)
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
  -Name EnableScriptBlockLogging -Value 1 -PropertyType DWORD -Force

# Enable PowerShell Module Logging
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging" `
  -Name EnableModuleLogging -Value 1 -PropertyType DWORD -Force

# Enable PowerShell Transcription
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription" `
  -Name EnableTranscripting -Value 1 -PropertyType DWORD -Force

# Sysmon config for fileless detection (key events):
# Event ID 1: Process creation (captures CommandLine)
# Event ID 7: Image loaded (DLL loading)
# Event ID 8: CreateRemoteThread (injection)
# Event ID 10: Process access (LSASS access)
# Event ID 19/20/21: WMI events
```

### Step 2: Detect PowerShell-Based Attacks

```
# Indicators of malicious PowerShell:

# Encoded command execution
EventID: 1
CommandLine contains: "powershell" AND ("-enc" OR "-e " OR "-encodedcommand" OR "FromBase64String")

# Download cradle patterns
CommandLine contains: "IEX" AND ("Net.WebClient" OR "DownloadString" OR "Invoke-WebRequest")
CommandLine contains: "Invoke-Expression" AND "New-Object"

# AMSI bypass attempts (Event ID 4104 - Script Block)
ScriptBlock contains: ("Amsi"+"Utils") OR ("amsi"+"InitFailed") OR "SetValue.*amsi"

# Splunk query for suspicious PowerShell:
index=windows source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104
| where match(ScriptBlockText, "(?i)(iex|invoke-expression|downloadstring|net\.webclient|frombase64|bypass|amsi.utils)")
| table _time host ScriptBlockText
```

### Step 3: Detect Process Injection Techniques

```
# Reflective DLL injection - loads DLL from memory without touching disk
# Detection: Sysmon Event 7 (ImageLoaded) where image path is unusual
EventID: 7
ImageLoaded NOT starts with: "C:\Windows\" AND NOT starts with: "C:\Program Files"

# Process hollowing - creates process in suspended state, replaces memory
# Detection: Process creation followed by immediate memory write
EventID: 1 + 10 correlation
# Process created then accessed with PROCESS_VM_WRITE

# APC injection - queues code to thread's async procedure call queue
# Detection: Sysmon CreateRemoteThread from non-system process
EventID: 8
SourceImage NOT IN (known_legitimate_sources)

# MDE KQL:
DeviceEvents
| where ActionType in ("CreateRemoteThreadApiCall", "NtAllocateVirtualMemoryApiCall")
| where InitiatingProcessFileName !in ("MsMpEng.exe", "svchost.exe")
| project Timestamp, DeviceName, ActionType, InitiatingProcessFileName,
    InitiatingProcessCommandLine, FileName
```

### Step 4: Detect WMI-Based Persistence

```
# Sysmon Event IDs 19/20/21 for WMI events
EventID: 19  # WmiEventFilter activity detected
EventID: 20  # WmiEventConsumer activity detected
EventID: 21  # WmiEventConsumerToFilter activity detected

# Any WMI event subscription creation is suspicious unless expected
# Common malicious WMI persistence:
Consumer contains: "CommandLineEventConsumer" OR "ActiveScriptEventConsumer"

# Query for WMI subscriptions via osquery or PowerShell:
Get-WMIObject -Namespace root\Subscription -Class __EventFilter
Get-WMIObject -Namespace root\Subscription -Class __EventConsumer
Get-WMIObject -Namespace root\Subscription -Class __FilterToConsumerBinding
```

### Step 5: Detect Registry-Based Execution

```
# Malware stored in registry values and executed via PowerShell
# Sysmon Event 13 - Registry value set with encoded content
EventID: 13
TargetObject contains: "CurrentVersion\Run"
Details: unusually long value or Base64-encoded content

# Detection query:
index=sysmon EventCode=13
| where match(Details, "[A-Za-z0-9+/=]{100,}")
| table _time host TargetObject Details Image
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Fileless Malware** | Malware that operates entirely in memory without writing executable files to disk |
| **AMSI** | Antimalware Scan Interface; Windows API allowing security products to inspect script content before execution |
| **Reflective DLL Injection** | Loading a DLL from memory rather than disk, avoiding file-based detection |
| **Process Hollowing** | Creating a legitimate process in suspended state and replacing its memory with malicious code |
| **Script Block Logging** | PowerShell logging feature that captures deobfuscated script content (Event ID 4104) |

## Tools & Systems

- **Sysmon**: Kernel-level process, DLL, and WMI monitoring
- **AMSI**: Windows script content inspection API
- **PowerShell Logging**: Script Block, Module, and Transcription logging
- **Microsoft Defender for Endpoint**: Behavioral detection for fileless techniques
- **Volatility 3**: Memory forensics for post-incident fileless malware analysis

## Common Pitfalls

- **Relying on file-based AV**: Traditional AV that scans files on disk will miss fileless attacks entirely. Behavioral detection and AMSI are required.
- **Disabled PowerShell logging**: Without Script Block Logging, deobfuscated PowerShell commands are invisible to defenders.
- **AMSI bypass not detected**: Sophisticated attackers bypass AMSI before executing payloads. Detect AMSI bypass attempts as a high-priority alert.
- **Not monitoring WMI events**: WMI persistence is a favored technique of APT groups. Sysmon events 19-21 must be enabled.

===== END FILE =====


===== FILE: skills/detecting-fileless-attacks-on-endpoints/LICENSE =====

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


===== FILE: skills/detecting-fileless-attacks-on-endpoints/assets/template.md =====
# Fileless Attack Detection Template

## Telemetry Status
| Source | Enabled | Event IDs |
|--------|---------|-----------|
| Sysmon | Yes/No | 1,7,8,10,19,20,21 |
| PowerShell Script Block | Yes/No | 4104 |
| AMSI | Yes/No | 1116 |

## Detection Rules
| Rule Name | Technique | SIEM Query | Status |
|-----------|-----------|-----------|--------|
| | T1059.001 | | Active/Draft |

## Sign-Off
| Role | Name | Date |
|------|------|------|
| Detection Engineer | | |
| SOC Lead | | |

===== END FILE =====


===== FILE: skills/detecting-fileless-attacks-on-endpoints/references/api-reference.md =====
# API Reference: Detecting Fileless Attacks on Endpoints

## Key Event Sources

| Source | Event ID | Detection |
|--------|----------|-----------|
| PowerShell Script Block | 4104 | Malicious script content |
| Sysmon Process Create | 1 | Encoded command execution |
| Sysmon CreateRemoteThread | 8 | Reflective DLL injection |
| Sysmon WMI EventFilter | 19 | WMI persistence |
| Sysmon WMI EventConsumer | 20 | WMI persistence |
| Sysmon WMI Binding | 21 | WMI persistence |

## python-evtx Usage

```python
import Evtx.Evtx as evtx
with evtx.Evtx("PowerShell-Operational.evtx") as log:
    for record in log.records():
        xml = record.xml()
        # Parse Event 4104 ScriptBlockText
```

## Suspicious PowerShell Patterns

```python
# Dynamic execution
r"Invoke-Expression|IEX\s*\("
# Reflective loading
r"System\.Reflection\.Assembly.*Load"
# Memory injection APIs
r"VirtualAlloc|VirtualProtect|CreateThread"
# WMI persistence
r"Register-WMI|__EventFilter|__EventConsumer"
# Encoded commands
r"-enc\s|-encodedcommand\s"
```

## Splunk SPL - Fileless Detection

```spl
index=powershell EventCode=4104
| where match(ScriptBlockText, "(?i)(Invoke-Expression|IEX|VirtualAlloc|FromBase64)")
| stats count by ScriptBlockText, Computer, UserID
```

## AMSI (Anti-Malware Scan Interface)

```powershell
# Enable AMSI logging
Set-MpPreference -EnableNetworkProtection Enabled
# Check AMSI status
Get-MpComputerStatus | Select AMServiceEnabled, AntispywareEnabled
```

## WMI Persistence Detection

```powershell
# List WMI event subscriptions
Get-WMIObject -Namespace root\Subscription -Class __EventFilter
Get-WMIObject -Namespace root\Subscription -Class __EventConsumer
Get-WMIObject -Namespace root\Subscription -Class __FilterToConsumerBinding
```

## CLI Usage

```bash
python agent.py --ps-log PowerShell-Operational.evtx
python agent.py --sysmon-log Sysmon.evtx --check-wmi --check-injection
```

===== END FILE =====


===== FILE: skills/detecting-fileless-attacks-on-endpoints/references/standards.md =====
# Standards & References
- **MITRE ATT&CK T1059.001**: PowerShell execution
- **MITRE ATT&CK T1055**: Process Injection (all sub-techniques)
- **MITRE ATT&CK T1546.003**: WMI Event Subscription persistence
- **MITRE ATT&CK T1620**: Reflective Code Loading
- **Microsoft AMSI Documentation**: https://learn.microsoft.com/en-us/windows/win32/amsi/
- **PowerShell Logging**: https://learn.microsoft.com/en-us/powershell/scripting/windows-powershell/wmf/whats-new/script-logging

===== END FILE =====


===== FILE: skills/detecting-fileless-attacks-on-endpoints/references/workflows.md =====
# Workflows
## Fileless Attack Detection
```
[Enable telemetry (Sysmon, PS logging, AMSI)] → [Build detection rules per technique]
  → [Deploy rules in SIEM] → [Threat hunt for historical fileless indicators]
  → [Triage alerts] → [Investigate memory for confirmed incidents]
  → [Extract IOCs from memory analysis] → [Tune detections]
```

===== END FILE =====


===== FILE: skills/detecting-fileless-attacks-on-endpoints/scripts/agent.py =====
#!/usr/bin/env python3
"""Fileless attack detection agent for endpoint logs.

Detects in-memory attacks by analyzing PowerShell script block logs (Event 4104),
WMI persistence events, and reflective DLL injection indicators from Sysmon.
"""

import argparse
import json
import re
from datetime import datetime

try:
    import Evtx.Evtx as evtx
except ImportError:
    evtx = None

SUSPICIOUS_PS_PATTERNS = {
    r"Invoke-Expression|IEX\s*\(": ("T1059.001", "HIGH", "Dynamic code execution"),
    r"Invoke-Mimikatz|Invoke-Kerberoast": ("T1003", "CRITICAL", "Credential tool"),
    r"System\.Reflection\.Assembly.*Load": ("T1620", "HIGH", "Reflective assembly load"),
    r"Net\.WebClient.*Download(String|Data|File)": ("T1105", "HIGH", "Remote download"),
    r"FromBase64String|Convert.*Base64": ("T1140", "MEDIUM", "Base64 decode"),
    r"VirtualAlloc|VirtualProtect|CreateThread": ("T1055", "CRITICAL", "Memory injection APIs"),
    r"New-Object.*IO\.MemoryStream": ("T1620", "HIGH", "In-memory stream"),
    r"-enc\s|-encodedcommand\s": ("T1027", "HIGH", "Encoded PowerShell"),
    r"Invoke-Shellcode|Invoke-ReflectivePEInjection": ("T1055", "CRITICAL", "Injection framework"),
    r"Win32_Process.*Create|WMI.*Process": ("T1047", "HIGH", "WMI process creation"),
    r"Register-WMI|__EventFilter|__EventConsumer": ("T1546.003", "CRITICAL", "WMI persistence"),
    r"HKCU:\\.*\\Run|HKLM:\\.*\\Run": ("T1547.001", "HIGH", "Registry run key"),
    r"Add-MpPreference.*ExclusionPath": ("T1562.001", "HIGH", "Defender exclusion"),
}

WMI_PERSISTENCE_EVENTS = {
    19: "WMI EventFilter created",
    20: "WMI EventConsumer created",
    21: "WMI EventConsumerToFilter binding",
}


def parse_powershell_scriptblock(filepath):
    if evtx is None:
        return {"error": "python-evtx not installed: pip install python-evtx"}
    findings = []
    with evtx.Evtx(filepath) as log:
        for record in log.records():
            xml = record.xml()
            if "<EventID>4104</EventID>" not in xml:
                continue
            script_block = re.search(r'<Data Name="ScriptBlockText">([^<]+)', xml)
            if not script_block:
                continue
            script = script_block.group(1)
            time_match = re.search(r'SystemTime="([^"]+)"', xml)

            for pattern, (mitre, severity, desc) in SUSPICIOUS_PS_PATTERNS.items():
                if re.search(pattern, script, re.IGNORECASE):
                    findings.append({
                        "event_id": 4104,
                        "timestamp": time_match.group(1) if time_match else "",
                        "pattern": desc,
                        "mitre": mitre,
                        "severity": severity,
                        "script_excerpt": script[:300],
                    })
    return findings


def parse_sysmon_wmi_persistence(filepath):
    if evtx is None:
        return {"error": "python-evtx not installed"}
    findings = []
    with evtx.Evtx(filepath) as log:
        for record in log.records():
            xml = record.xml()
            event_id_match = re.search(r'<EventID[^>]*>(\d+)</EventID>', xml)
            if not event_id_match:
                continue
            event_id = int(event_id_match.group(1))
            if event_id not in WMI_PERSISTENCE_EVENTS:
                continue
            time_match = re.search(r'SystemTime="([^"]+)"', xml)
            name = re.search(r'<Data Name="Name">([^<]+)', xml)
            operation = re.search(r'<Data Name="Operation">([^<]+)', xml)
            consumer = re.search(r'<Data Name="Destination">([^<]+)', xml)
            user = re.search(r'<Data Name="User">([^<]+)', xml)

            findings.append({
                "event_id": event_id,
                "type": WMI_PERSISTENCE_EVENTS[event_id],
                "timestamp": time_match.group(1) if time_match else "",
                "name": name.group(1) if name else "",
                "operation": operation.group(1) if operation else "",
                "destination": consumer.group(1) if consumer else "",
                "user": user.group(1) if user else "",
                "severity": "CRITICAL",
                "mitre": "T1546.003",
            })
    return findings


def parse_sysmon_injection(filepath):
    if evtx is None:
        return {"error": "python-evtx not installed"}
    findings = []
    with evtx.Evtx(filepath) as log:
        for record in log.records():
            xml = record.xml()
            if "<EventID>8</EventID>" not in xml:
                continue
            source = re.search(r'<Data Name="SourceImage">([^<]+)', xml)
            target = re.search(r'<Data Name="TargetImage">([^<]+)', xml)
            time_match = re.search(r'SystemTime="([^"]+)"', xml)

            findings.append({
                "event_id": 8,
                "timestamp": time_match.group(1) if time_match else "",
                "source_image": source.group(1) if source else "",
                "target_image": target.group(1) if target else "",
                "severity": "HIGH",
                "mitre": "T1055",
                "description": "CreateRemoteThread - possible reflective injection",
            })
    return findings


def main():
    parser = argparse.ArgumentParser(description="Fileless Attack Detector")
    parser.add_argument("--ps-log", help="PowerShell EVTX file (Event 4104)")
    parser.add_argument("--sysmon-log", help="Sysmon EVTX file")
    parser.add_argument("--check-wmi", action="store_true", help="Check WMI persistence")
    parser.add_argument("--check-injection", action="store_true", help="Check injection")
    args = parser.parse_args()

    results = {"timestamp": datetime.utcnow().isoformat() + "Z", "findings": []}

    if args.ps_log:
        ps = parse_powershell_scriptblock(args.ps_log)
        if isinstance(ps, dict) and "error" in ps:
            results["error"] = ps["error"]
        else:
            results["findings"].extend(ps)

    if args.sysmon_log and args.check_wmi:
        wmi = parse_sysmon_wmi_persistence(args.sysmon_log)
        if not isinstance(wmi, dict):
            results["findings"].extend(wmi)

    if args.sysmon_log and args.check_injection:
        inj = parse_sysmon_injection(args.sysmon_log)
        if not isinstance(inj, dict):
            results["findings"].extend(inj)

    results["total_findings"] = len(results["findings"])
    print(json.dumps(results, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-fileless-attacks-on-endpoints/scripts/process.py =====
#!/usr/bin/env python3
"""Fileless Attack Detector - Scans PowerShell logs for fileless attack indicators."""

import json, csv, re, sys, os
from collections import Counter
from datetime import datetime

FILELESS_PATTERNS = {
    "encoded_command": r"(?i)(-enc\s|-e\s|-encodedcommand|frombase64string)",
    "download_cradle": r"(?i)(downloadstring|invoke-webrequest|net\.webclient|wget\s|curl\s)",
    "amsi_bypass": r"(?i)(amsiutils|amsiinitfailed|amsi\.dll)",
    "reflection": r"(?i)(system\.reflection|loadassembly|gettype.*invoke)",
    "wmi_abuse": r"(?i)(win32_process.*create|wmiclass|managementclass)",
    "credential_access": r"(?i)(mimikatz|invoke-mimikatz|sekurlsa|logonpasswords)",
    "invoke_expression": r"(?i)(iex\s|invoke-expression)",
}


def scan_powershell_logs(csv_path: str) -> list:
    detections = []
    with open(csv_path, "r", encoding="utf-8-sig") as f:
        for row in csv.DictReader(f):
            script = row.get("ScriptBlockText", row.get("Message", ""))
            for pattern_name, pattern in FILELESS_PATTERNS.items():
                if re.search(pattern, script):
                    detections.append({
                        "timestamp": row.get("TimeCreated", row.get("Date and Time", "")),
                        "host": row.get("Computer", row.get("MachineName", "")),
                        "technique": pattern_name,
                        "script_excerpt": script[:300],
                    })
                    break
    return detections


def generate_report(detections: list, output_path: str) -> None:
    by_technique = Counter(d["technique"] for d in detections)
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "total_detections": len(detections),
        "by_technique": dict(by_technique),
        "detections": detections[:100],
    }
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <powershell_logs.csv>")
        sys.exit(1)
    detections = scan_powershell_logs(sys.argv[1])
    out = os.path.join(os.path.dirname(sys.argv[1]) or ".", "fileless_detection_report.json")
    generate_report(detections, out)
    print(f"Detections: {len(detections)}")

===== END FILE =====


===== FILE: skills/hardening-linux-endpoint-with-cis-benchmark/SKILL.md =====
---
name: hardening-linux-endpoint-with-cis-benchmark
description: 'Hardens Linux endpoints using CIS Benchmark recommendations for Ubuntu,
  RHEL, and CentOS to reduce attack surface, enforce security baselines, and meet
  compliance requirements. Use when deploying new Linux servers, remediating audit
  findings, or establishing security baselines for Linux infrastructure. Activates
  for requests involving Linux hardening, CIS benchmarks for Linux, server security
  baselines, or Linux configuration compliance.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- hardening
- linux-security
- CIS-benchmark
- Ubuntu
- RHEL
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
---
# Hardening Linux Endpoint with CIS Benchmark

## When to Use

Use this skill when:
- Hardening Linux servers (Ubuntu, RHEL, CentOS, Debian) against CIS benchmarks
- Automating Linux security baselines using Ansible, OpenSCAP, or shell scripts
- Meeting compliance requirements (PCI DSS, HIPAA, SOC 2) for Linux endpoints
- Remediating findings from vulnerability scans or security audits

**Do not use** for Windows hardening (use hardening-windows-endpoint-with-cis-benchmark).

## Prerequisites

- Root or sudo access on target Linux endpoints
- CIS Benchmark PDF for target distribution (from cisecurity.org)
- OpenSCAP or CIS-CAT for automated assessment
- Ansible for enterprise-scale remediation (optional)

## Workflow

### Step 1: Filesystem Configuration (Section 1)

```bash
# 1.1.1 Disable unused filesystems
cat >> /etc/modprobe.d/CIS.conf << 'EOF'
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install squashfs /bin/true
install udf /bin/true
EOF

# 1.1.2 Ensure /tmp is a separate partition with nodev,nosuid,noexec
# /etc/fstab entry:
# tmpfs /tmp tmpfs defaults,rw,nosuid,nodev,noexec,relatime 0 0
systemctl unmask tmp.mount
systemctl enable tmp.mount

# 1.1.8 Ensure nodev option on /dev/shm
mount -o remount,nodev,nosuid,noexec /dev/shm
echo "tmpfs /dev/shm tmpfs defaults,nodev,nosuid,noexec 0 0" >> /etc/fstab

# 1.4 Secure boot settings
chown root:root /boot/grub/grub.cfg
chmod 600 /boot/grub/grub.cfg
# Set GRUB password
grub-mkpasswd-pbkdf2  # Generate hash, add to /etc/grub.d/40_custom
```

### Step 2: Services and Network (Sections 2-3)

```bash
# 2.1 Disable unnecessary services
systemctl disable --now avahi-daemon
systemctl disable --now cups
systemctl disable --now rpcbind
systemctl disable --now xinetd

# 2.2 Ensure NTP is configured
apt install chrony -y  # or systemd-timesyncd
systemctl enable --now chrony

# 3.1 Network parameters (host only, not router)
cat >> /etc/sysctl.d/99-cis.conf << 'EOF'
net.ipv4.ip_forward = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.tcp_syncookies = 1
net.ipv6.conf.all.accept_ra = 0
net.ipv6.conf.default.accept_ra = 0
EOF
sysctl --system

# 3.4 Configure firewall (UFW or firewalld)
ufw enable
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
```

### Step 3: Access Control (Sections 4-5)

```bash
# 5.2 SSH Server Configuration (/etc/ssh/sshd_config)
sed -i 's/#Protocol 2/Protocol 2/' /etc/ssh/sshd_config
cat >> /etc/ssh/sshd_config << 'EOF'
LogLevel VERBOSE
MaxAuthTries 4
PermitRootLogin no
PermitEmptyPasswords no
PasswordAuthentication no
X11Forwarding no
MaxStartups 10:30:60
LoginGraceTime 60
AllowTcpForwarding no
ClientAliveInterval 300
ClientAliveCountMax 3
EOF
systemctl restart sshd

# 5.3 Password policy (PAM)
# /etc/security/pwquality.conf
minlen = 14
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1

# 5.4 User account settings
# /etc/login.defs
PASS_MAX_DAYS 365
PASS_MIN_DAYS 1
PASS_WARN_AGE 7

# Lock inactive accounts
useradd -D -f 30
```

### Step 4: Audit and Logging (Section 4)

```bash
# Install and configure auditd
apt install auditd audispd-plugins -y
systemctl enable --now auditd

# /etc/audit/rules.d/cis.rules
cat > /etc/audit/rules.d/cis.rules << 'EOF'
-w /etc/sudoers -p wa -k scope
-w /etc/sudoers.d/ -p wa -k scope
-w /var/log/sudo.log -p wa -k actions
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time-change
-a always,exit -F arch=b64 -S sethostname -S setdomainname -k system-locale
-w /etc/group -p wa -k identity
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /var/log/faillog -p wa -k logins
-w /var/log/lastlog -p wa -k logins
-a always,exit -F arch=b64 -S chmod -S fchmod -S fchmodat -k perm_mod
-a always,exit -F arch=b64 -S unlink -S rmdir -S rename -k delete
-w /sbin/insmod -p x -k modules
-w /sbin/modprobe -p x -k modules
-e 2
EOF
augenrules --load

# Configure rsyslog for remote logging
echo "*.* @@syslog-server.corp.com:514" >> /etc/rsyslog.d/50-remote.conf
systemctl restart rsyslog
```

### Step 5: Assess with OpenSCAP

```bash
# Install OpenSCAP
apt install openscap-scanner scap-security-guide -y

# Run CIS benchmark assessment
oscap xccdf eval \
  --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
  --results /tmp/cis_results.xml \
  --report /tmp/cis_report.html \
  /usr/share/xml/scap/ssg/content/ssg-ubuntu2204-ds.xml

# View HTML report in browser for detailed results
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **OpenSCAP** | Open-source SCAP (Security Content Automation Protocol) scanner for automated compliance |
| **auditd** | Linux audit framework for monitoring system calls and file access |
| **PAM** | Pluggable Authentication Modules; configurable authentication framework for Linux |
| **sysctl** | Linux kernel parameter configuration for network and system security tuning |
| **AIDE** | Advanced Intrusion Detection Environment; file integrity checker for Linux |

## Tools & Systems

- **OpenSCAP**: Automated CIS benchmark assessment for Linux
- **Ansible Lockdown**: Ansible roles for automated CIS benchmark remediation
- **Lynis**: Open-source security auditing tool for Linux/Unix systems
- **AIDE**: File integrity monitoring for Linux endpoints
- **auditd**: Linux audit framework for system call monitoring

## Common Pitfalls

- **Applying server benchmarks to workstations**: CIS provides separate benchmarks for server and workstation profiles. Server benchmarks disable desktop services.
- **Breaking SSH access**: Misconfiguring sshd_config (especially PermitRootLogin, PasswordAuthentication) can lock out administrators. Always test SSH configuration changes from a second session.
- **Not testing firewall rules**: Enabling UFW without allowing SSH first will disconnect remote sessions permanently.
- **Kernel parameter changes without testing**: Some sysctl settings can break application networking. Test in staging first.

===== END FILE =====


===== FILE: skills/hardening-linux-endpoint-with-cis-benchmark/LICENSE =====

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


===== FILE: skills/hardening-linux-endpoint-with-cis-benchmark/assets/template.md =====
# Linux CIS Hardening Template

## Endpoint Information
| Field | Value |
|-------|-------|
| Hostname | |
| Distribution | Ubuntu 22.04 / RHEL 9 |
| CIS Benchmark Version | |
| Profile | Level 1 Server / Level 2 Server |
| Assessment Date | |

## Compliance Results
| Metric | Value |
|--------|-------|
| Total Rules | |
| Passed | |
| Failed | |
| Score | % |

## Exception Register
| CIS ID | Recommendation | Justification | Compensating Control | Approved By |
|--------|---------------|---------------|---------------------|-------------|
| | | | | |

## Sign-Off
| Role | Name | Date |
|------|------|------|
| System Admin | | |
| Security | | |

===== END FILE =====


===== FILE: skills/hardening-linux-endpoint-with-cis-benchmark/references/api-reference.md =====
# API Reference: Linux CIS Benchmark Hardening

## CIS Benchmark Sections

| Section | Topic |
|---------|-------|
| 1 | Initial Setup (filesystem, updates, secure boot) |
| 2 | Services (inetd, special purpose) |
| 3 | Network Configuration (parameters, firewall) |
| 4 | Logging and Auditing (auditd, rsyslog) |
| 5 | Access, Authentication, Authorization (SSH, PAM) |
| 6 | System Maintenance (file permissions) |

## Key sysctl Parameters

### Network Hardening
```bash
sysctl -w net.ipv4.ip_forward=0
sysctl -w net.ipv4.conf.all.send_redirects=0
sysctl -w net.ipv4.conf.all.accept_source_route=0
sysctl -w net.ipv4.conf.all.accept_redirects=0
sysctl -w net.ipv4.conf.all.log_martians=1
sysctl -w net.ipv4.tcp_syncookies=1
```

### Persistent Configuration
```bash
# /etc/sysctl.d/99-hardening.conf
net.ipv4.ip_forward = 0
net.ipv4.conf.all.send_redirects = 0
```

## SSH Hardening (/etc/ssh/sshd_config)

| Parameter | Recommended Value |
|-----------|-------------------|
| PermitRootLogin | no |
| PasswordAuthentication | no |
| Protocol | 2 |
| MaxAuthTries | 4 |
| ClientAliveInterval | 300 |
| ClientAliveCountMax | 3 |
| X11Forwarding | no |
| AllowTcpForwarding | no |

## Service Management

### Disable unnecessary services
```bash
systemctl disable avahi-daemon
systemctl disable cups
systemctl disable rpcbind
systemctl mask service_name
```

### Check enabled services
```bash
systemctl list-unit-files --type=service --state=enabled
```

## Audit Rules (/etc/audit/rules.d/)

### Monitor critical files
```bash
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/sudoers -p wa -k sudoers
```

### Monitor system calls
```bash
-a always,exit -F arch=b64 -S execve -k exec
-a always,exit -F arch=b64 -S mount -k mounts
```

## File Permissions

| File | Owner | Permissions |
|------|-------|-------------|
| `/etc/passwd` | root:root | 644 |
| `/etc/shadow` | root:shadow | 000 or 640 |
| `/etc/group` | root:root | 644 |
| `/etc/gshadow` | root:shadow | 000 or 640 |

## Automated Tools

### OpenSCAP
```bash
oscap xccdf eval --profile cis \
    --results results.xml \
    /usr/share/xml/scap/ssg/content/ssg-ubuntu2204-ds.xml
```

### Lynis
```bash
lynis audit system --cronjob --quiet
```

===== END FILE =====


===== FILE: skills/hardening-linux-endpoint-with-cis-benchmark/references/standards.md =====
# Standards & References

## Primary Standards
- **CIS Ubuntu Linux 22.04 LTS Benchmark v2.0.0**: https://www.cisecurity.org/benchmark/ubuntu_linux
- **CIS Red Hat Enterprise Linux 9 Benchmark v2.0.0**: https://www.cisecurity.org/benchmark/red_hat_linux
- **NIST SP 800-123**: Guide to General Server Security
- **DISA STIG**: Security Technical Implementation Guide for RHEL/Ubuntu

## Compliance Mappings
| Framework | Requirement | Linux Hardening Coverage |
|-----------|------------|------------------------|
| PCI DSS 4.0 | 2.2 - Configuration standards | CIS benchmark application |
| NIST 800-53 | CM-6 Configuration Settings | Kernel, service, and auth hardening |
| NIST 800-53 | AU-2 Audit Events | auditd configuration |
| HIPAA | 164.312(a)(1) Access Control | SSH hardening, PAM configuration |

## Supporting References
- **Ansible Lockdown**: https://github.com/ansible-lockdown
- **OpenSCAP**: https://www.open-scap.org/
- **Lynis**: https://cisofy.com/lynis/
- **SCAP Security Guide**: https://github.com/ComplianceAsCode/content

===== END FILE =====


===== FILE: skills/hardening-linux-endpoint-with-cis-benchmark/references/workflows.md =====
# Workflows

## Workflow 1: Linux CIS Hardening Deployment
```
[Select CIS Benchmark for distro/version] → [Choose L1 or L2 profile]
  → [Run OpenSCAP baseline assessment] → [Review initial compliance score]
  → [Apply remediations (Ansible/manual)] → [Re-assess with OpenSCAP]
  → [Document exceptions] → [Deploy to production fleet]
  → [Schedule quarterly reassessment]
```

## Workflow 2: Automated Remediation with Ansible
```
[Clone Ansible Lockdown role for target distro]
  → [Configure variables (skip list, exceptions)]
  → [Test against staging servers]
  → [Review changes and application compatibility]
  → [Deploy to production in rolling batches]
  → [Run OpenSCAP validation after each batch]
```

===== END FILE =====


===== FILE: skills/hardening-linux-endpoint-with-cis-benchmark/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for auditing Linux endpoints against CIS Benchmark hardening controls."""

import argparse
import json
import os
import re
import shlex
import subprocess
from datetime import datetime, timezone


def run_cmd(cmd, timeout=10):
    """Run a command and return stdout."""
    try:
        # Strip shell stderr redirects since we use subprocess.DEVNULL
        clean_cmd = cmd.replace("2>/dev/null", "").strip()
        # Use shell only for commands with shell operators
        needs_shell = any(op in clean_cmd for op in ("|", ";", "&&", "||"))
        return subprocess.check_output(
            clean_cmd if needs_shell else shlex.split(clean_cmd),
            shell=needs_shell, text=True, errors="replace",
            timeout=timeout, stderr=subprocess.DEVNULL
        ).strip()
    except subprocess.SubprocessError:
        return ""


def check_filesystem_config():
    """CIS Section 1 — Filesystem Configuration."""
    findings = []
    cramfs = run_cmd("modprobe -n -v cramfs 2>/dev/null")
    if "install /bin/true" not in cramfs and "install /bin/false" not in cramfs:
        findings.append({"check": "1.1.1.1", "issue": "cramfs module not disabled", "severity": "LOW"})
    tmp_mount = run_cmd("findmnt /tmp")
    if not tmp_mount:
        findings.append({"check": "1.1.2", "issue": "/tmp not a separate partition", "severity": "MEDIUM"})
    elif "nodev" not in tmp_mount or "nosuid" not in tmp_mount:
        findings.append({"check": "1.1.3", "issue": "/tmp missing nodev/nosuid options", "severity": "MEDIUM"})
    return findings


def check_services():
    """CIS Section 2 — Services."""
    findings = []
    unnecessary = ["avahi-daemon", "cups", "dhcpd", "slapd", "nfs-server",
                    "rpcbind", "named", "vsftpd", "httpd", "dovecot", "smb", "squid"]
    for svc in unnecessary:
        status = run_cmd(f"systemctl is-enabled {svc} 2>/dev/null")
        if status == "enabled":
            findings.append({"check": "2.x", "issue": f"Unnecessary service enabled: {svc}", "severity": "MEDIUM"})
    return findings


def check_network_parameters():
    """CIS Section 3 — Network Parameters."""
    findings = []
    params = {
        "net.ipv4.ip_forward": ("0", "3.1.1", "IP forwarding enabled"),
        "net.ipv4.conf.all.send_redirects": ("0", "3.1.2", "ICMP redirects enabled"),
        "net.ipv4.conf.all.accept_source_route": ("0", "3.2.1", "Source routing accepted"),
        "net.ipv4.conf.all.accept_redirects": ("0", "3.2.2", "ICMP redirects accepted"),
        "net.ipv4.conf.all.log_martians": ("1", "3.2.4", "Martian logging disabled"),
        "net.ipv4.tcp_syncookies": ("1", "3.2.8", "SYN cookies disabled"),
    }
    for param, (expected, cis_id, desc) in params.items():
        value = run_cmd(f"sysctl -n {param} 2>/dev/null")
        if value != expected:
            findings.append({"check": cis_id, "issue": desc, "current": value, "expected": expected, "severity": "MEDIUM"})
    return findings


def check_access_auth():
    """CIS Section 5 — Access, Authentication, Authorization."""
    findings = []
    sshd_config = run_cmd("cat /etc/ssh/sshd_config 2>/dev/null")
    if "PermitRootLogin yes" in sshd_config:
        findings.append({"check": "5.2.10", "issue": "SSH root login permitted", "severity": "HIGH"})
    if "PasswordAuthentication yes" in sshd_config:
        findings.append({"check": "5.2.12", "issue": "SSH password authentication enabled", "severity": "MEDIUM"})
    if "Protocol 1" in sshd_config:
        findings.append({"check": "5.2.4", "issue": "SSH Protocol 1 enabled", "severity": "HIGH"})

    passwd_maxdays = run_cmd("grep PASS_MAX_DAYS /etc/login.defs 2>/dev/null")
    if passwd_maxdays:
        match = re.search(r'PASS_MAX_DAYS\s+(\d+)', passwd_maxdays)
        if match and int(match.group(1)) > 365:
            findings.append({"check": "5.4.1.1", "issue": f"Password max age: {match.group(1)} days", "severity": "MEDIUM"})
    return findings


def check_audit_logging():
    """CIS Section 4 — Logging and Auditing."""
    findings = []
    auditd = run_cmd("systemctl is-active auditd 2>/dev/null")
    if auditd != "active":
        findings.append({"check": "4.1.1", "issue": "auditd not active", "severity": "HIGH"})
    rsyslog = run_cmd("systemctl is-active rsyslog 2>/dev/null")
    if rsyslog != "active":
        findings.append({"check": "4.2.1", "issue": "rsyslog not active", "severity": "MEDIUM"})
    return findings


def check_file_permissions():
    """CIS Section 6 — System File Permissions."""
    findings = []
    critical_files = {
        "/etc/passwd": "644",
        "/etc/shadow": "000",
        "/etc/group": "644",
        "/etc/gshadow": "000",
    }
    for fpath, expected in critical_files.items():
        if os.path.isfile(fpath):
            mode = oct(os.stat(fpath).st_mode)[-3:]
            if mode > expected:
                findings.append({"check": "6.1.x", "issue": f"{fpath}: mode {mode} > {expected}", "severity": "MEDIUM"})
    return findings


def main():
    parser = argparse.ArgumentParser(
        description="Audit Linux endpoint against CIS Benchmark"
    )
    parser.add_argument("--output", "-o", help="Output JSON report")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    print("[*] Linux CIS Benchmark Hardening Audit Agent")
    all_findings = []
    all_findings.extend(check_filesystem_config())
    all_findings.extend(check_services())
    all_findings.extend(check_network_parameters())
    all_findings.extend(check_access_auth())
    all_findings.extend(check_audit_logging())
    all_findings.extend(check_file_permissions())

    high = sum(1 for f in all_findings if f["severity"] == "HIGH")
    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "findings": all_findings,
        "total": len(all_findings),
        "high_severity": high,
        "compliance_score": max(0, 100 - len(all_findings) * 5),
    }
    print(f"[*] Findings: {len(all_findings)} (HIGH: {high})")

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"[*] Report saved to {args.output}")
    else:
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/hardening-linux-endpoint-with-cis-benchmark/scripts/process.py =====
#!/usr/bin/env python3
"""Linux CIS Benchmark Compliance Processor - Parses OpenSCAP XCCDF results."""

import xml.etree.ElementTree as ET
import json
import csv
import sys
import os
from datetime import datetime


def parse_oscap_results(xml_path: str) -> dict:
    """Parse OpenSCAP XCCDF results XML."""
    tree = ET.parse(xml_path)
    root = tree.getroot()
    ns = {"xccdf": "http://checklists.nist.gov/xccdf/1.2"}

    results = {"total": 0, "passed": 0, "failed": 0, "notapplicable": 0,
               "findings": [], "score": 0.0}

    for rr in root.iter():
        if rr.tag.endswith("}rule-result"):
            results["total"] += 1
            result_elem = None
            for child in rr:
                if child.tag.endswith("}result"):
                    result_elem = child
            if result_elem is not None:
                val = result_elem.text
                if val == "pass":
                    results["passed"] += 1
                elif val == "fail":
                    results["failed"] += 1
                    results["findings"].append({
                        "rule_id": rr.get("idref", ""),
                        "severity": rr.get("severity", ""),
                        "result": "FAIL",
                    })
                elif val == "notapplicable":
                    results["notapplicable"] += 1

    scored = results["passed"] + results["failed"]
    if scored > 0:
        results["score"] = round((results["passed"] / scored) * 100, 2)
    return results


def generate_report(results: dict, output_path: str) -> None:
    """Generate compliance report."""
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "score": results["score"],
        "total": results["total"],
        "passed": results["passed"],
        "failed": results["failed"],
        "not_applicable": results["notapplicable"],
        "status": "COMPLIANT" if results["score"] >= 95 else "NON-COMPLIANT",
        "failed_controls": results["findings"][:100],
    }
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <oscap_results.xml>")
        sys.exit(1)
    results = parse_oscap_results(sys.argv[1])
    out = os.path.join(os.path.dirname(sys.argv[1]) or ".", "linux_cis_report.json")
    generate_report(results, out)
    print(f"Score: {results['score']}% | Passed: {results['passed']} | Failed: {results['failed']}")

===== END FILE =====


===== FILE: skills/hardening-windows-endpoint-with-cis-benchmark/SKILL.md =====
---
name: hardening-windows-endpoint-with-cis-benchmark
description: 'Hardens Windows endpoints using CIS (Center for Internet Security) Benchmark
  recommendations to reduce attack surface, enforce security baselines, and meet compliance
  requirements. Use when deploying new Windows workstations or servers, remediating
  audit findings, or establishing organization-wide security baselines. Activates
  for requests involving Windows hardening, CIS benchmarks, GPO security baselines,
  or endpoint configuration compliance.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- hardening
- windows-security
- CIS-benchmark
- GPO
- baseline-configuration
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
---
# Hardening Windows Endpoint with CIS Benchmark

## When to Use

Use this skill when:
- Deploying new Windows 10/11 or Server 2019/2022 endpoints that require security hardening
- Establishing organization-wide security baselines using CIS Level 1 or Level 2 profiles
- Remediating findings from compliance audits (PCI DSS, HIPAA, SOC 2) that reference CIS benchmarks
- Validating existing endpoint configurations against current CIS benchmark versions

**Do not use** this skill for Linux endpoints (use hardening-linux-endpoint-with-cis-benchmark) or for cloud-native workloads that require CIS cloud benchmarks.

## Prerequisites

- Windows 10/11 Enterprise or Windows Server 2019/2022 target endpoints
- Active Directory Group Policy Management Console (GPMC) for enterprise deployment
- CIS-CAT Pro Assessor or CIS-CAT Lite for automated benchmark assessment
- Administrative access to target endpoints or domain controller
- Current CIS Benchmark PDF for the target Windows version (download from cisecurity.org)

## Workflow

### Step 1: Select CIS Benchmark Profile Level

CIS provides two profile levels for Windows endpoints:

**Level 1 (L1) - Corporate/Enterprise Environment**:
- Practical hardening settings that can be applied to most organizations
- Minimal impact on functionality and user experience
- Covers: password policy, audit policy, user rights, security options, Windows Firewall

**Level 2 (L2) - High Security/Sensitive Data**:
- Includes all L1 settings plus additional restrictions
- May impact usability (disabling autorun, restricting remote desktop, enhanced audit logging)
- Appropriate for systems handling PII, PHI, PCI data, or classified information

Select profile based on data classification and risk tolerance of the endpoint.

### Step 2: Import CIS GPO Baselines

CIS provides pre-built GPO templates (Build Kits) for each benchmark version:

```powershell
# Download CIS Build Kit from CIS WorkBench (requires CIS SecureSuite membership)
# Extract the GPO backup to a staging directory

# Import the CIS GPO into Active Directory
Import-GPO -BackupGpoName "CIS Microsoft Windows 11 Enterprise v3.0.0 L1" `
  -TargetName "CIS-Win11-L1-Baseline" `
  -Path "C:\CIS-GPO-Backups\Win11-Enterprise" `
  -CreateIfNeeded

# Link GPO to target OU
New-GPLink -Name "CIS-Win11-L1-Baseline" `
  -Target "OU=Workstations,DC=corp,DC=example,DC=com" `
  -LinkEnabled Yes
```

### Step 3: Apply Key CIS Benchmark Categories

**Account Policies (Section 1)**:
```
Password Policy:
  - Minimum password length: 14 characters (1.1.4)
  - Maximum password age: 365 days (1.1.3)
  - Password complexity: Enabled (1.1.5)
  - Store passwords using reversible encryption: Disabled (1.1.6)

Account Lockout Policy:
  - Account lockout threshold: 5 invalid logon attempts (1.2.1)
  - Account lockout duration: 15 minutes (1.2.2)
  - Reset account lockout counter after: 15 minutes (1.2.3)
```

**Local Policies - Audit Policy (Section 17)**:
```
Audit Policy Configuration:
  - Audit Credential Validation: Success and Failure (17.1.1)
  - Audit Security Group Management: Success (17.2.5)
  - Audit Logon: Success and Failure (17.5.1)
  - Audit Process Creation: Success (17.6.1)
  - Audit Removable Storage: Success and Failure (17.6.4)
```

**Security Options (Section 2.3)**:
```
  - Interactive logon: Do not display last user name: Enabled (2.3.7.1)
  - Interactive logon: Machine inactivity limit: 900 seconds (2.3.7.3)
  - Network access: Do not allow anonymous enumeration of SAM accounts: Enabled (2.3.10.2)
  - Network security: LAN Manager authentication level: Send NTLMv2 response only (2.3.11.7)
  - UAC: Run all administrators in Admin Approval Mode: Enabled (2.3.17.6)
```

**Windows Firewall (Section 9)**:
```
  - Domain Profile: Firewall state: On (9.1.1)
  - Domain Profile: Inbound connections: Block (9.1.2)
  - Private Profile: Firewall state: On (9.2.1)
  - Public Profile: Firewall state: On (9.3.1)
  - Public Profile: Inbound connections: Block (9.3.2)
```

### Step 4: Validate with CIS-CAT Assessment

```powershell
# Run CIS-CAT Pro Assessor against target endpoint
# CIS-CAT produces an HTML/XML report with pass/fail per recommendation

.\Assessor-CLI.bat `
  -b "benchmarks\CIS_Microsoft_Windows_11_Enterprise_Benchmark_v3.0.0-xccdf.xml" `
  -p "Level 1 (L1) - Corporate/Enterprise Environment" `
  -rd "C:\CIS-Reports" `
  -nts

# Review report for failed controls
# Score target: 95%+ for L1, 90%+ for L2 (due to operational exceptions)
```

### Step 5: Document Exceptions and Compensating Controls

For each CIS recommendation that cannot be applied:
1. Document the specific recommendation ID and title
2. State the business justification for the exception
3. Define the compensating control that addresses the residual risk
4. Set a review date (quarterly) to reassess the exception
5. Obtain sign-off from the information security officer

Example exception:
```
Recommendation: 2.3.7.3 - Interactive logon: Machine inactivity limit: 900 seconds
Exception: Kiosk systems in manufacturing floor require 1800 seconds
Compensating Control: Physical badge-access to manufacturing area, CCTV monitoring
Review Date: 2026-06-01
Approved By: CISO
```

### Step 6: Continuous Compliance Monitoring

Configure recurring CIS-CAT scans via scheduled tasks or SCCM:
```powershell
# Create scheduled task for weekly CIS-CAT assessment
$action = New-ScheduledTaskAction -Execute "C:\CIS-CAT\Assessor-CLI.bat" `
  -Argument "-b benchmarks\CIS_Win11_v3.0.0-xccdf.xml -p Level1 -rd C:\CIS-Reports -nts"
$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 2am
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -RunLevel Highest
Register-ScheduledTask -TaskName "CIS-Benchmark-Scan" -Action $action `
  -Trigger $trigger -Principal $principal
```

Feed results into SIEM for drift detection and dashboard reporting.

## Key Concepts

| Term | Definition |
|------|-----------|
| **CIS Benchmark** | Consensus-based security configuration guide developed by CIS with input from government, industry, and academia |
| **Level 1 Profile** | Practical security baseline suitable for most organizations with minimal operational impact |
| **Level 2 Profile** | Extended security baseline for high-security environments that may reduce functionality |
| **CIS-CAT** | CIS Configuration Assessment Tool that automates benchmark compliance checking |
| **Build Kit** | Pre-configured GPO templates provided by CIS that implement benchmark recommendations |
| **Scoring** | CIS recommendations are either Scored (compliance-measurable) or Not Scored (best-practice guidance) |

## Tools & Systems

- **CIS-CAT Pro Assessor**: Automated benchmark compliance scanner (requires CIS SecureSuite license)
- **Microsoft Security Compliance Toolkit (SCT)**: Microsoft's own GPO baselines (complementary to CIS)
- **Group Policy Management Console (GPMC)**: Enterprise GPO deployment and management
- **LGPO.exe**: Microsoft tool for applying GPOs to standalone (non-domain) systems
- **Nessus/Tenable**: Vulnerability scanner with CIS benchmark audit files

## Common Pitfalls

- **Applying L2 to all endpoints**: Level 2 restrictions (disabling Autoplay, restricting Remote Desktop) break workflows on standard workstations. Reserve L2 for endpoints handling sensitive data.
- **Not testing GPOs in pilot OU**: Deploy CIS GPOs to a test OU with representative hardware/software before organization-wide rollout to avoid breaking line-of-business applications.
- **Ignoring CIS benchmark version updates**: CIS benchmarks update with each Windows feature release. Running an outdated benchmark misses new security settings and generates false compliance reports.
- **Forgetting local admin accounts**: CIS benchmarks assume domain-joined endpoints. Standalone systems require LGPO.exe or Microsoft Intune for baseline enforcement.
- **No exception process**: Applying 100% of CIS recommendations is rarely feasible. Without a formal exception process, teams either ignore hardening or break applications.

===== END FILE =====


===== FILE: skills/hardening-windows-endpoint-with-cis-benchmark/LICENSE =====

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


===== FILE: skills/hardening-windows-endpoint-with-cis-benchmark/assets/template.md =====
# CIS Benchmark Hardening Assessment Template

## Endpoint Information

| Field | Value |
|-------|-------|
| Hostname | |
| OS Version | Windows 11 Enterprise 23H2 |
| Domain/Workgroup | |
| CIS Benchmark Version | v3.0.0 |
| Profile Applied | Level 1 / Level 2 |
| Assessment Date | |
| Assessor | |

## Pre-Hardening Checklist

- [ ] Verified current OS patch level
- [ ] Documented installed applications and services
- [ ] Created system restore point / VM snapshot
- [ ] Identified business-critical applications for compatibility testing
- [ ] Obtained CIS Build Kit GPO for target OS version
- [ ] Confirmed Active Directory OU structure for GPO deployment

## CIS-CAT Assessment Results

| Metric | Value |
|--------|-------|
| Total Rules Assessed | |
| Passed | |
| Failed | |
| Not Applicable | |
| Compliance Score | % |
| Target Score | 95% (L1) / 90% (L2) |

## Failed Controls Summary

| CIS ID | Recommendation | Severity | Remediation Plan | Exception? |
|--------|---------------|----------|-----------------|------------|
| | | | | |

## Exception Register

| CIS ID | Recommendation | Business Justification | Compensating Control | Review Date | Approved By |
|--------|---------------|----------------------|---------------------|-------------|-------------|
| | | | | | |

## Post-Hardening Validation

- [ ] Re-ran CIS-CAT assessment after GPO application
- [ ] Verified all business applications function correctly
- [ ] Confirmed remote management tools (RDP, WinRM) still accessible
- [ ] Validated endpoint joins domain and receives policies
- [ ] Tested user login and MFA functionality
- [ ] Verified antivirus/EDR agent status

## Compliance Tracking

| Scan Date | Score | Delta | New Failures | Resolved | Notes |
|-----------|-------|-------|-------------|----------|-------|
| | | | | | Initial baseline |

## Sign-Off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| System Administrator | | | |
| Security Analyst | | | |
| CISO / Security Manager | | | |

===== END FILE =====


===== FILE: skills/hardening-windows-endpoint-with-cis-benchmark/references/api-reference.md =====
# API Reference: Windows CIS Benchmark Hardening

## CIS Benchmark Sections

| Section | Topic |
|---------|-------|
| 1 | Account Policies (passwords, lockout) |
| 2 | Local Policies (audit, user rights, security options) |
| 9 | Windows Firewall |
| 17 | Advanced Audit Policy |
| 18 | Administrative Templates |
| 19 | User Configuration |

## PowerShell Commands

### Password Policy
```powershell
net accounts
# or
Get-ADDefaultDomainPasswordPolicy
```

### Audit Policy
```powershell
auditpol /get /category:*
```

### Firewall Status
```powershell
Get-NetFirewallProfile | Select-Object Name, Enabled
```

### Registry Checks
```powershell
Get-ItemProperty -Path 'HKLM:\...' -Name 'ValueName'
```

## Key Registry Settings

| Path | Value | Recommended |
|------|-------|-------------|
| `HKLM\SYSTEM\CurrentControlSet\Control\Lsa\LmCompatibilityLevel` | DWORD | 5 (NTLMv2 only) |
| `HKLM\...\Policies\System\EnableLUA` | DWORD | 1 (UAC enabled) |
| `HKLM\...\LanManServer\Parameters\SMB1` | DWORD | 0 (disabled) |
| `HKLM\...\Policies\System\ConsentPromptBehaviorAdmin` | DWORD | 2 |

## Password Policy Settings

| Setting | CIS Recommendation |
|---------|-------------------|
| Minimum length | >= 14 characters |
| Maximum age | <= 365 days |
| Minimum age | >= 1 day |
| Complexity | Enabled |
| Lockout threshold | <= 5 attempts |
| Lockout duration | >= 15 minutes |

## Advanced Audit Policy

| Subcategory | Recommended |
|-------------|-------------|
| Credential Validation | Success and Failure |
| Logon/Logoff | Success and Failure |
| Account Management | Success |
| Process Creation | Success |
| Policy Change | Success |

## GPO Export and Analysis

### Export GPO
```powershell
gpresult /H gpo-report.html
```

### Secedit Export
```cmd
secedit /export /cfg security-config.inf
```

## Automated Tools

### Microsoft Security Compliance Toolkit
```powershell
# Download from Microsoft
# Includes GPO baselines and LGPO tool
LGPO.exe /g .\GPO-Backup
```

### CIS-CAT
```bash
# CIS Configuration Assessment Tool
cis-cat.bat -b benchmark.xml -p "CIS Windows 11 Enterprise"
```

## Windows Optional Features

### Check SMBv1
```powershell
Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

### Disable SMBv1
```powershell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

===== END FILE =====


===== FILE: skills/hardening-windows-endpoint-with-cis-benchmark/references/standards.md =====
# Standards & References - Hardening Windows Endpoint with CIS Benchmark

## Primary Standards

### CIS Microsoft Windows 11 Enterprise Benchmark v3.0.0
- **Publisher**: Center for Internet Security (CIS)
- **URL**: https://www.cisecurity.org/benchmark/microsoft_windows_desktop
- **Scope**: 400+ security configuration recommendations organized into 19 sections
- **Profiles**: Level 1 (Corporate), Level 2 (High Security), BitLocker (BL), Next Generation Windows Security (NG)

### CIS Microsoft Windows Server 2022 Benchmark v3.0.0
- **Publisher**: CIS
- **URL**: https://www.cisecurity.org/benchmark/microsoft_windows_server
- **Scope**: Server-specific recommendations for Member Server and Domain Controller roles
- **Profiles**: Level 1 (MS/DC), Level 2 (MS/DC)

### NIST SP 800-123 - Guide to General Server Security
- **Publisher**: NIST
- **Relevance**: Foundational server hardening guidance that CIS benchmarks operationalize
- **Key sections**: OS hardening, user account management, resource access controls

## Compliance Mappings

### PCI DSS v4.0 Mapping
| PCI DSS Requirement | CIS Benchmark Section |
|---------------------|----------------------|
| 2.2 - Develop configuration standards | CIS Benchmark entire scope |
| 5.2 - Anti-malware mechanisms | Section 18.10 (Windows Defender) |
| 8.3 - Strong authentication | Section 1.1 (Password Policy) |
| 10.2 - Audit trail implementation | Section 17 (Advanced Audit Policy) |

### NIST 800-53 Rev 5 Mapping
| NIST Control | CIS Benchmark Section |
|-------------|----------------------|
| CM-6 Configuration Settings | Entire CIS Benchmark |
| AC-2 Account Management | Section 1 (Account Policies) |
| AU-2 Audit Events | Section 17 (Advanced Audit Policy) |
| SC-7 Boundary Protection | Section 9 (Windows Firewall) |

### HIPAA Security Rule Mapping
| HIPAA Requirement | CIS Benchmark Section |
|------------------|----------------------|
| 164.312(a)(1) Access Control | Section 2.3 (Security Options) |
| 164.312(b) Audit Controls | Section 17 (Advanced Audit Policy) |
| 164.312(a)(2)(iv) Encryption | BitLocker Profile |
| 164.312(c)(1) Integrity | Section 18.9 (Windows Defender) |

## Supporting References

- **Microsoft Security Baselines**: https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines
- **CIS Controls v8**: Maps to Implementation Group (IG) 1, 2, and 3 for prioritized implementation
- **STIGs (DoD)**: DISA Security Technical Implementation Guides provide overlapping but more restrictive benchmarks for government systems
- **ACSC Essential Eight**: Australian Cyber Security Centre hardening framework with overlap to CIS benchmark categories

===== END FILE =====


===== FILE: skills/hardening-windows-endpoint-with-cis-benchmark/references/workflows.md =====
# Workflows - Hardening Windows Endpoint with CIS Benchmark

## Workflow 1: Initial Baseline Deployment

```
[Select Windows Version & CIS Benchmark]
    │
    ▼
[Choose Profile Level (L1 or L2)]
    │
    ▼
[Download CIS Build Kit GPOs from CIS WorkBench]
    │
    ▼
[Import GPOs into Active Directory via GPMC]
    │
    ▼
[Link GPO to Pilot OU with 5-10 test endpoints]
    │
    ▼
[Run CIS-CAT assessment on pilot endpoints]
    │
    ├── Score >= 95% ──► [Test application compatibility for 2 weeks]
    │                         │
    │                         ├── No issues ──► [Deploy GPO to production OUs]
    │                         │
    │                         └── Issues found ──► [Document exceptions, add compensating controls]
    │                                                  │
    │                                                  ▼
    │                                              [Redeploy with exceptions]
    │
    └── Score < 95% ──► [Investigate GPO application failures]
                              │
                              ▼
                         [Fix WMI filters, security filtering, or GPO precedence]
                              │
                              ▼
                         [Re-run CIS-CAT assessment]
```

## Workflow 2: Continuous Compliance Monitoring

```
[Weekly CIS-CAT Scheduled Scan]
    │
    ▼
[Parse XML results → ingest into SIEM]
    │
    ▼
[Compare against baseline score]
    │
    ├── Score drift > 2% ──► [Generate compliance drift alert]
    │                              │
    │                              ▼
    │                         [Identify changed settings via GPResult /H]
    │                              │
    │                              ▼
    │                         [Determine if change was authorized]
    │                              │
    │                              ├── Authorized ──► [Update baseline, document exception]
    │                              │
    │                              └── Unauthorized ──► [Revert change, investigate as security incident]
    │
    └── Score stable ──► [Log compliance status, update dashboard]
```

## Workflow 3: New CIS Benchmark Version Upgrade

```
[CIS releases new benchmark version]
    │
    ▼
[Download updated benchmark and Build Kit]
    │
    ▼
[Diff new vs. current benchmark recommendations]
    │
    ▼
[Identify new/changed/removed recommendations]
    │
    ▼
[Impact assessment: Will new settings break applications?]
    │
    ├── Yes ──► [Lab test, document new exceptions]
    │
    └── No ──► [Update GPO with new Build Kit]
                    │
                    ▼
               [Deploy to pilot OU first]
                    │
                    ▼
               [Run CIS-CAT with new benchmark profile]
                    │
                    ▼
               [Production rollout after 2-week pilot]
```

## Workflow 4: Standalone Endpoint Hardening (Non-Domain)

```
[Identify standalone Windows endpoint]
    │
    ▼
[Download LGPO.exe from Microsoft SCT]
    │
    ▼
[Export CIS Build Kit GPO to local policy format]
    │
    ▼
[Apply via LGPO.exe: LGPO.exe /g C:\CIS-Policy]
    │
    ▼
[Run CIS-CAT Lite assessment]
    │
    ▼
[Document results and exceptions]
    │
    ▼
[Schedule quarterly manual reassessment]
```

===== END FILE =====


===== FILE: skills/hardening-windows-endpoint-with-cis-benchmark/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for auditing Windows endpoints against CIS Benchmark hardening controls."""

import argparse
import json
import subprocess
import sys
from datetime import datetime, timezone


def run_ps(command, timeout=15):
    """Run PowerShell command and return output."""
    try:
        return subprocess.check_output(
            ["powershell", "-NoProfile", "-Command", command],
            text=True, errors="replace", timeout=timeout
        ).strip()
    except subprocess.SubprocessError:
        return ""


def check_password_policy():
    """CIS 1.1 — Password Policy."""
    findings = []
    result = run_ps("net accounts")
    for line in result.splitlines():
        if "Minimum password length" in line:
            val = int(line.split(":")[-1].strip())
            if val < 14:
                findings.append({"check": "1.1.4", "issue": f"Min password length: {val} (should be >= 14)", "severity": "HIGH"})
        if "Maximum password age" in line:
            val = line.split(":")[-1].strip()
            if val == "Unlimited":
                findings.append({"check": "1.1.2", "issue": "No max password age", "severity": "MEDIUM"})
        if "Lockout threshold" in line:
            val = line.split(":")[-1].strip()
            if val == "Never" or (val.isdigit() and int(val) > 10):
                findings.append({"check": "1.2.1", "issue": f"Account lockout threshold: {val}", "severity": "MEDIUM"})
    return findings


def check_audit_policy():
    """CIS 17 — Advanced Audit Policy."""
    findings = []
    result = run_ps("auditpol /get /category:*")
    required_audits = {
        "Credential Validation": "Success and Failure",
        "Logon": "Success and Failure",
        "Security Group Management": "Success",
        "User Account Management": "Success and Failure",
        "Process Creation": "Success",
    }
    for subcategory, expected in required_audits.items():
        if subcategory in result:
            for line in result.splitlines():
                if subcategory in line:
                    if "No Auditing" in line:
                        findings.append({
                            "check": "17.x", "issue": f"Audit not configured: {subcategory}",
                            "severity": "HIGH",
                        })
    return findings


def check_windows_firewall():
    """CIS 9 — Windows Firewall."""
    findings = []
    profiles = ["Domain", "Private", "Public"]
    for profile in profiles:
        result = run_ps(f"Get-NetFirewallProfile -Name {profile} | Select-Object Enabled | ConvertTo-Json")
        try:
            data = json.loads(result) if result else {}
            if not data.get("Enabled"):
                findings.append({"check": "9.1", "issue": f"Firewall {profile} profile disabled", "severity": "HIGH"})
        except json.JSONDecodeError:
            pass
    return findings


def check_security_options():
    """CIS 2.3 — Security Options."""
    findings = []
    checks = [
        ("HKLM:\\SYSTEM\\CurrentControlSet\\Control\\Lsa", "LmCompatibilityLevel", 5,
         "2.3.11.7", "LAN Manager auth level not NTLMv2 only"),
        ("HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Policies\\System", "EnableLUA", 1,
         "2.3.17.1", "UAC not enabled"),
        ("HKLM:\\SYSTEM\\CurrentControlSet\\Services\\LanManServer\\Parameters", "SMB1", 0,
         "18.3.2", "SMBv1 not disabled"),
    ]
    for path, name, expected, cis_id, desc in checks:
        result = run_ps(
            f"(Get-ItemProperty -Path '{path}' -Name '{name}' -ErrorAction SilentlyContinue).{name}"
        )
        if result.strip().isdigit() and int(result.strip()) != expected:
            findings.append({"check": cis_id, "issue": desc, "current": result.strip(), "severity": "HIGH"})
    return findings


def check_windows_features():
    """CIS 18 — Windows Features."""
    findings = []
    risky_features = ["SMB1Protocol", "TelnetClient", "TFTP"]
    for feature in risky_features:
        result = run_ps(
            f"Get-WindowsOptionalFeature -Online -FeatureName {feature} 2>$null | "
            f"Select-Object State | ConvertTo-Json"
        )
        try:
            data = json.loads(result) if result else {}
            if data.get("State") == 1:  # Enabled
                findings.append({"check": "18.x", "issue": f"Risky feature enabled: {feature}", "severity": "MEDIUM"})
        except json.JSONDecodeError:
            pass
    return findings


def main():
    parser = argparse.ArgumentParser(
        description="Audit Windows endpoint against CIS Benchmark"
    )
    parser.add_argument("--output", "-o", help="Output JSON report")
    args = parser.parse_args()

    print("[*] Windows CIS Benchmark Hardening Audit Agent")
    if sys.platform != "win32":
        print("[!] This agent requires Windows")
        return

    all_findings = []
    all_findings.extend(check_password_policy())
    all_findings.extend(check_audit_policy())
    all_findings.extend(check_windows_firewall())
    all_findings.extend(check_security_options())
    all_findings.extend(check_windows_features())

    high = sum(1 for f in all_findings if f["severity"] == "HIGH")
    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "findings": all_findings,
        "total": len(all_findings),
        "high_severity": high,
        "compliance_score": max(0, 100 - len(all_findings) * 5),
    }
    print(f"[*] Findings: {len(all_findings)} (HIGH: {high})")

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"[*] Report saved to {args.output}")
    else:
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/hardening-windows-endpoint-with-cis-benchmark/scripts/process.py =====
#!/usr/bin/env python3
"""
CIS Benchmark Compliance Processor

Parses CIS-CAT Pro XML assessment results, calculates compliance scores,
identifies failed controls, and generates remediation reports.
"""

import xml.etree.ElementTree as ET
import json
import csv
import sys
import os
from datetime import datetime
from collections import defaultdict


def parse_ciscat_xccdf_results(xml_path: str) -> dict:
    """Parse CIS-CAT XCCDF results XML into structured findings."""
    tree = ET.parse(xml_path)
    root = tree.getroot()

    ns = {
        "xccdf": "http://checklists.nist.gov/xccdf/1.2",
        "cpe": "http://cpe.mitre.org/dictionary/2.0",
    }

    results = {
        "benchmark_id": "",
        "benchmark_title": "",
        "profile": "",
        "target": "",
        "scan_date": "",
        "total_rules": 0,
        "passed": 0,
        "failed": 0,
        "not_applicable": 0,
        "error": 0,
        "score": 0.0,
        "findings": [],
    }

    benchmark = root.find(".//xccdf:benchmark", ns)
    if benchmark is not None:
        results["benchmark_id"] = benchmark.get("id", "")

    title = root.find(".//xccdf:title", ns)
    if title is not None:
        results["benchmark_title"] = title.text or ""

    test_result = root.find(".//xccdf:TestResult", ns)
    if test_result is not None:
        results["scan_date"] = test_result.get("end-time", "")
        target = test_result.find("xccdf:target", ns)
        if target is not None:
            results["target"] = target.text or ""

        profile = test_result.find("xccdf:profile", ns)
        if profile is not None:
            results["profile"] = profile.get("idref", "")

        for rule_result in test_result.findall("xccdf:rule-result", ns):
            rule_id = rule_result.get("idref", "")
            result_elem = rule_result.find("xccdf:result", ns)
            result_value = result_elem.text if result_elem is not None else "unknown"

            results["total_rules"] += 1

            if result_value == "pass":
                results["passed"] += 1
            elif result_value == "fail":
                results["failed"] += 1
                results["findings"].append(
                    {
                        "rule_id": rule_id,
                        "result": "FAIL",
                        "severity": rule_result.get("severity", "unknown"),
                    }
                )
            elif result_value == "notapplicable":
                results["not_applicable"] += 1
            else:
                results["error"] += 1

    scored_rules = results["passed"] + results["failed"]
    if scored_rules > 0:
        results["score"] = round((results["passed"] / scored_rules) * 100, 2)

    return results


def generate_remediation_csv(findings: list, output_path: str) -> None:
    """Generate CSV of failed controls for remediation tracking."""
    with open(output_path, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow(
            [
                "Rule ID",
                "Severity",
                "Result",
                "Remediation Status",
                "Assigned To",
                "Due Date",
                "Notes",
            ]
        )
        for finding in findings:
            writer.writerow(
                [
                    finding["rule_id"],
                    finding["severity"],
                    finding["result"],
                    "Open",
                    "",
                    "",
                    "",
                ]
            )


def compare_scan_results(current: dict, previous: dict) -> dict:
    """Compare two CIS-CAT scan results to detect compliance drift."""
    drift = {
        "current_score": current["score"],
        "previous_score": previous["score"],
        "score_delta": round(current["score"] - previous["score"], 2),
        "new_failures": [],
        "resolved_failures": [],
        "persistent_failures": [],
    }

    current_failed = {f["rule_id"] for f in current["findings"]}
    previous_failed = {f["rule_id"] for f in previous["findings"]}

    drift["new_failures"] = sorted(current_failed - previous_failed)
    drift["resolved_failures"] = sorted(previous_failed - current_failed)
    drift["persistent_failures"] = sorted(current_failed & previous_failed)

    return drift


def generate_compliance_report(results: dict, output_path: str) -> None:
    """Generate a compliance summary report in JSON format."""
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "benchmark": results["benchmark_title"],
        "profile": results["profile"],
        "target_host": results["target"],
        "scan_date": results["scan_date"],
        "compliance_score": results["score"],
        "summary": {
            "total_rules_assessed": results["total_rules"],
            "passed": results["passed"],
            "failed": results["failed"],
            "not_applicable": results["not_applicable"],
            "errors": results["error"],
        },
        "failed_controls": results["findings"],
        "compliance_status": "COMPLIANT"
        if results["score"] >= 95.0
        else "NON-COMPLIANT",
    }

    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2)


def categorize_findings_by_section(findings: list) -> dict:
    """Group failed findings by CIS benchmark section for prioritized remediation."""
    sections = defaultdict(list)

    section_map = {
        "1.1": "Password Policy",
        "1.2": "Account Lockout Policy",
        "2.2": "User Rights Assignment",
        "2.3": "Security Options",
        "5": "System Services",
        "9": "Windows Firewall",
        "17": "Advanced Audit Policy",
        "18": "Administrative Templates",
        "19": "Administrative Templates (User)",
    }

    for finding in findings:
        rule_id = finding["rule_id"]
        categorized = False
        for prefix, section_name in section_map.items():
            if f"_section_{prefix}" in rule_id.lower() or f".{prefix}." in rule_id:
                sections[section_name].append(finding)
                categorized = True
                break
        if not categorized:
            sections["Other"].append(finding)

    return dict(sections)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <ciscat-results.xml> [previous-results.xml]")
        print()
        print("Parses CIS-CAT XCCDF results and generates compliance reports.")
        print()
        print("Arguments:")
        print("  ciscat-results.xml      Current CIS-CAT assessment results")
        print("  previous-results.xml    (Optional) Previous results for drift detection")
        sys.exit(1)

    xml_path = sys.argv[1]
    if not os.path.exists(xml_path):
        print(f"Error: File not found: {xml_path}")
        sys.exit(1)

    results = parse_ciscat_xccdf_results(xml_path)
    base_name = os.path.splitext(os.path.basename(xml_path))[0]
    output_dir = os.path.dirname(xml_path) or "."

    report_path = os.path.join(output_dir, f"{base_name}_compliance_report.json")
    generate_compliance_report(results, report_path)
    print(f"Compliance report: {report_path}")

    if results["findings"]:
        csv_path = os.path.join(output_dir, f"{base_name}_remediation.csv")
        generate_remediation_csv(results["findings"], csv_path)
        print(f"Remediation tracker: {csv_path}")

    print(f"\nCompliance Score: {results['score']}%")
    print(f"Status: {'COMPLIANT' if results['score'] >= 95.0 else 'NON-COMPLIANT'}")
    print(f"Passed: {results['passed']} | Failed: {results['failed']} | N/A: {results['not_applicable']}")

    if len(sys.argv) >= 3:
        prev_path = sys.argv[2]
        if os.path.exists(prev_path):
            prev_results = parse_ciscat_xccdf_results(prev_path)
            drift = compare_scan_results(results, prev_results)
            drift_path = os.path.join(output_dir, f"{base_name}_drift.json")
            with open(drift_path, "w") as f:
                json.dump(drift, f, indent=2)
            print(f"\nDrift report: {drift_path}")
            print(f"Score delta: {drift['score_delta']:+.2f}%")
            print(f"New failures: {len(drift['new_failures'])}")
            print(f"Resolved: {len(drift['resolved_failures'])}")

===== END FILE =====


===== FILE: skills/implementing-application-whitelisting-with-applocker/SKILL.md =====
---
name: implementing-application-whitelisting-with-applocker
description: 'Implements application whitelisting using Windows AppLocker to restrict
  unauthorized software execution on endpoints, reducing attack surface from malware,
  unauthorized tools, and shadow IT. Use when enforcing application control policies,
  meeting compliance requirements for software restriction, or preventing execution
  of unsigned or untrusted binaries. Activates for requests involving AppLocker, application
  whitelisting, software restriction, or executable control.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- AppLocker
- application-whitelisting
- windows-security
- software-restriction
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
- T1027
---
# Implementing Application Whitelisting with AppLocker

## When to Use

Use this skill when:
- Implementing application control to prevent unauthorized software execution on Windows endpoints
- Meeting compliance requirements (PCI DSS 6.4.3, NIST 800-53 CM-7, ACSC Essential Eight)
- Blocking common attack vectors: living-off-the-land binaries (LOLBins), script-based attacks, unauthorized admin tools
- Restricting software installation in kiosk, POS, or high-security environments

**Do not use** this skill for macOS/Linux application control (use OS-native tools like Gatekeeper or AppArmor) or for enterprise-grade WDAC (Windows Defender Application Control) deployments.

## Prerequisites

- Windows 10/11 Enterprise or Education, or Windows Server 2016+
- Application Identity service (AppIDSvc) enabled on target endpoints
- Active Directory with Group Policy Management Console (GPMC)
- Complete application inventory of approved software
- Test OU with representative endpoints for policy validation

## Workflow

### Step 1: Inventory Approved Applications

Before creating AppLocker rules, catalog all legitimate software:

```powershell
# Generate application inventory on reference endpoint
Get-AppLockerFileInformation -Directory "C:\Program Files" -Recurse `
  -FileType Exe | Export-Csv "C:\AppLocker\app_inventory_progfiles.csv" -NoTypeInformation

Get-AppLockerFileInformation -Directory "C:\Program Files (x86)" -Recurse `
  -FileType Exe | Export-Csv "C:\AppLocker\app_inventory_progfiles86.csv" -NoTypeInformation

# Include Windows system executables
Get-AppLockerFileInformation -Directory "C:\Windows" -Recurse `
  -FileType Exe | Export-Csv "C:\AppLocker\app_inventory_windows.csv" -NoTypeInformation
```

### Step 2: Create AppLocker Policy with Default Rules

```powershell
# In Group Policy Editor (gpedit.msc) or GPMC:
# Navigate to: Computer Configuration → Policies → Windows Settings
#   → Security Settings → Application Control Policies → AppLocker

# Enable default rules for each rule collection:
# - Executable Rules: Allow Everyone to run files in Program Files and Windows
# - Windows Installer Rules: Allow Everyone to run digitally signed MSIs
# - Script Rules: Allow Everyone to run scripts in Program Files and Windows
# - Packaged App Rules: Allow Everyone to run signed packaged apps

# PowerShell: Generate default rules
$defaultRules = Get-AppLockerPolicy -Local -Xml
Set-AppLockerPolicy -XmlPolicy $defaultRules -Merge
```

### Step 3: Create Publisher-Based Rules (Preferred)

Publisher rules are the most maintainable since they survive application updates:

```xml
<!-- Example AppLocker policy XML for publisher rules -->
<RuleCollection Type="Exe" EnforcementMode="AuditOnly">
  <!-- Default: Allow Windows binaries -->
  <FilePublisherRule Id="a9e18c21-ff54-4677-b3ac-4b9a03261f6c"
    Name="Allow Microsoft signed" Description="Allow all Microsoft-signed executables"
    UserOrGroupSid="S-1-1-0" Action="Allow">
    <Conditions>
      <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION*"
        ProductName="*" BinaryName="*">
        <BinaryVersionRange LowSection="*" HighSection="*"/>
      </FilePublisherCondition>
    </Conditions>
  </FilePublisherRule>

  <!-- Allow specific third-party vendor -->
  <FilePublisherRule Id="b2e28c32-aa65-5788-c4bd-5c0b14372e7d"
    Name="Allow Adobe Acrobat" Description="Allow Adobe-signed Acrobat executables"
    UserOrGroupSid="S-1-1-0" Action="Allow">
    <Conditions>
      <FilePublisherCondition PublisherName="O=ADOBE INC.*"
        ProductName="ADOBE ACROBAT*" BinaryName="*">
        <BinaryVersionRange LowSection="*" HighSection="*"/>
      </FilePublisherCondition>
    </Conditions>
  </FilePublisherRule>
</RuleCollection>
```

### Step 4: Block Known-Abused Binaries (LOLBins)

```powershell
# Deny rules for commonly abused living-off-the-land binaries
# These are legitimate Windows tools frequently used by attackers

$denyPaths = @(
    "%SYSTEM32%\mshta.exe",
    "%SYSTEM32%\wscript.exe",
    "%SYSTEM32%\cscript.exe",
    "%SYSTEM32%\regsvr32.exe",
    "%SYSTEM32%\certutil.exe",
    "%SYSTEM32%\msbuild.exe",
    "%SYSTEM32%\installutil.exe",
    "%WINDIR%\Microsoft.NET\Framework\*\msbuild.exe",
    "%WINDIR%\Microsoft.NET\Framework64\*\msbuild.exe"
)

# Create deny rules in AppLocker policy for standard users
# Important: Deny rules take precedence over Allow rules
# Apply only to standard users (not admins who may need these tools)
```

### Step 5: Configure Script Rules

```
Script Rules (critical for preventing script-based attacks):

Allow:
  - Scripts in C:\Program Files\* (publisher or path-based)
  - Scripts in C:\Windows\* (default Windows scripts)
  - Approved admin scripts from \\fileserver\scripts\*

Deny (for standard users):
  - PowerShell scripts from user-writable directories
  - VBScript from %TEMP%, %APPDATA%, %USERPROFILE%\Downloads
  - JavaScript (.js) from any user-writable location

DLL Rules (optional, high performance impact):
  - Enable only in high-security environments
  - Allow signed DLLs from Program Files and Windows directories
  - Performance impact: 5-10% CPU increase during DLL loading
```

### Step 6: Deploy in Audit Mode First

```powershell
# CRITICAL: Always deploy AppLocker in Audit mode before Enforce mode
# Audit mode logs what would be blocked without actually blocking

# Set enforcement mode to Audit Only in GPO:
# AppLocker → Executable Rules → Properties → Configured: Audit only
# AppLocker → Script Rules → Properties → Configured: Audit only
# AppLocker → Windows Installer Rules → Properties → Configured: Audit only

# Ensure Application Identity service is running
Set-Service -Name AppIDSvc -StartupType Automatic
Start-Service AppIDSvc

# Link GPO to test OU
New-GPLink -Name "AppLocker-Audit-Policy" `
  -Target "OU=AppLocker-Pilot,DC=corp,DC=example,DC=com"

# Monitor audit logs for 2-4 weeks
# Event Log: Applications and Services Logs → Microsoft → Windows → AppLocker
# Event IDs:
#   8003 = EXE/DLL would be blocked
#   8006 = Script/MSI would be blocked
#   8023 = Packaged app would be blocked
```

### Step 7: Analyze Audit Logs and Refine Rules

```powershell
# Export AppLocker audit events
Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" `
  -FilterXPath "*[System[EventID=8003]]" |
  Select-Object TimeCreated,
    @{N='User';E={$_.Properties[0].Value}},
    @{N='FilePath';E={$_.Properties[1].Value}},
    @{N='FileHash';E={$_.Properties[4].Value}} |
  Export-Csv "C:\AppLocker\audit_blocked_exes.csv" -NoTypeInformation

# Review blocked applications
# For each blocked legitimate application:
#   1. Create a publisher rule (if signed) or path rule (if unsigned)
#   2. Add to AppLocker policy
#   3. Re-audit for 1 additional week
```

### Step 8: Switch to Enforce Mode

```powershell
# After audit period with no legitimate applications blocked:
# Change enforcement mode from Audit to Enforce

# Update GPO:
# AppLocker → Executable Rules → Properties → Configured: Enforce rules
# AppLocker → Script Rules → Properties → Configured: Enforce rules

# Phased enforcement:
# Week 1: Enforce EXE rules only
# Week 2: Enforce Script rules
# Week 3: Enforce MSI rules
# Week 4: (Optional) Enforce DLL rules

# Maintain monitoring: Event IDs 8004 (blocked EXE), 8007 (blocked script)
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Application Whitelisting** | Security model that allows only pre-approved applications to execute, denying everything else by default |
| **Publisher Rule** | AppLocker rule based on digital signature; most resilient to application updates |
| **Path Rule** | AppLocker rule based on file system path; less secure as attackers can place files in allowed paths |
| **Hash Rule** | AppLocker rule based on file hash; most restrictive but breaks on every application update |
| **LOLBin** | Living Off the Land Binary; legitimate OS tool abused by attackers to avoid detection |
| **Audit Mode** | AppLocker logs policy violations without blocking; essential for rule refinement |
| **Enforcement Mode** | AppLocker actively blocks applications that violate policy rules |

## Tools & Systems

- **AppLocker (built-in)**: Windows application control feature in Enterprise/Education editions
- **WDAC (Windows Defender Application Control)**: More advanced successor to AppLocker for modern Windows
- **Microsoft LAPS**: Manages local admin passwords to prevent bypassing AppLocker via admin rights
- **WDAC Wizard**: GUI tool for creating Windows Defender Application Control policies
- **AaronLocker**: Open-source AppLocker rule generator by Microsoft employee (GitHub)

## Common Pitfalls

- **Skipping Audit mode**: Deploying AppLocker in Enforce mode without audit period will block critical applications and cause outages.
- **Relying solely on path rules**: Users with write access to allowed paths (C:\Windows\Temp) can bypass path-based rules. Prefer publisher rules.
- **Not blocking user-writable directories**: The most common gap is allowing execution from %TEMP%, Downloads, or %APPDATA%.
- **Forgetting Application Identity service**: AppLocker requires the AppIDSvc service running. If it stops, all rules stop enforcing.
- **Admin bypass**: Local administrators can bypass AppLocker by default. For full enforcement, combine with WDAC which enforces for all users including admins.
- **DLL rule performance**: Enabling DLL rules creates significant performance overhead. Only enable in high-security environments where the tradeoff is justified.

===== END FILE =====


===== FILE: skills/implementing-application-whitelisting-with-applocker/LICENSE =====

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


===== FILE: skills/implementing-application-whitelisting-with-applocker/assets/template.md =====
# AppLocker Application Whitelisting Template

## Policy Information

| Field | Value |
|-------|-------|
| Policy Name | |
| Target OS | Windows 10/11 Enterprise |
| Profile | Workstation / Server / Kiosk |
| Enforcement Mode | Audit Only / Enforce |
| GPO Name | |
| Target OU | |
| Last Updated | |

## Approved Application Inventory

| Application | Publisher | Version | Rule Type | Justification |
|------------|----------|---------|-----------|---------------|
| Microsoft Office | Microsoft Corporation | 365 | Publisher | Business productivity |
| Google Chrome | Google LLC | * | Publisher | Approved browser |
| Adobe Acrobat | Adobe Inc. | * | Publisher | PDF processing |
| | | | | |

## Rule Collection Configuration

### Executable Rules (EXE, COM)

| Rule Name | Type | Action | Scope | Conditions |
|-----------|------|--------|-------|------------|
| Default - Windows | Path | Allow | Everyone | %WINDIR%\* |
| Default - Program Files | Path | Allow | Everyone | %PROGRAMFILES%\* |
| Deny - LOLBins | Path | Deny | Standard Users | mshta.exe, wscript.exe, etc. |
| | | | | |

### Script Rules (PS1, BAT, CMD, VBS, JS)

| Rule Name | Type | Action | Scope | Conditions |
|-----------|------|--------|-------|------------|
| Default - Windows scripts | Path | Allow | Everyone | %WINDIR%\* |
| Default - Program Files scripts | Path | Allow | Everyone | %PROGRAMFILES%\* |
| Deny - User profile scripts | Path | Deny | Standard Users | %USERPROFILE%\* |
| | | | | |

### Windows Installer Rules (MSI, MSP, MST)

| Rule Name | Type | Action | Scope | Conditions |
|-----------|------|--------|-------|------------|
| Default - Signed MSI | Publisher | Allow | Everyone | All signed installers |
| | | | | |

## LOLBin Deny List

| Binary | Path | ATT&CK Technique | Risk |
|--------|------|------------------|------|
| mshta.exe | %SYSTEM32% | T1218.005 | HTA execution for code delivery |
| wscript.exe | %SYSTEM32% | T1059.005 | VBScript execution |
| cscript.exe | %SYSTEM32% | T1059.005 | Command-line scripting |
| regsvr32.exe | %SYSTEM32% | T1218.010 | COM scriptlet execution |
| certutil.exe | %SYSTEM32% | T1140 | File download and decode |
| msbuild.exe | .NET Framework | T1127.001 | Inline task execution |

## Audit Results Tracking

| Audit Period | Blocked Events | Legitimate Blocks | Rules Added | Remaining Issues |
|-------------|---------------|-------------------|-------------|-----------------|
| | | | | |

## Exception Register

| Application | Reason for Exception | Compensating Control | Approved By | Review Date |
|------------|---------------------|---------------------|-------------|-------------|
| | | | | |

## Sign-Off

| Role | Name | Date |
|------|------|------|
| Security Engineer | | |
| IT Operations Lead | | |
| Change Manager | | |

===== END FILE =====


===== FILE: skills/implementing-application-whitelisting-with-applocker/references/api-reference.md =====
# API Reference: Implementing Application Whitelisting with AppLocker

## PowerShell AppLocker Management

```powershell
# Export current policy
Get-AppLockerPolicy -Effective -Xml | Out-File applocker_policy.xml

# Import policy from XML
Set-AppLockerPolicy -XmlPolicy applocker_policy.xml

# Test if file is allowed
Test-AppLockerPolicy -XmlPolicy policy.xml -Path "C:\app.exe" -User Everyone

# Get AppLocker event logs
Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL"
```

## AppLocker Event IDs

| Event ID | Type | Meaning |
|----------|------|---------|
| 8002 | EXE/DLL | Allowed |
| 8003 | EXE/DLL | Blocked |
| 8004 | EXE/DLL | Would block (audit) |
| 8005 | Script | Allowed |
| 8006 | Script | Blocked |
| 8007 | Script | Would block (audit) |

## Rule Collections

| Collection | File Types |
|------------|------------|
| Executable | .exe, .com |
| Windows Installer | .msi, .msp, .mst |
| Script | .ps1, .bat, .cmd, .vbs, .js |
| DLL | .dll, .ocx |
| Packaged App | AppX/MSIX |

## GPO Configuration Path

```
Computer Configuration > Policies > Windows Settings >
  Security Settings > Application Control Policies > AppLocker
```

## Default Rule Paths

```
%PROGRAMFILES%\*     - Allow Everyone
%WINDIR%\*           - Allow Everyone
*                    - Allow BUILTIN\Administrators
```

### References

- AppLocker: https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/applocker/applocker-overview
- AppLocker PowerShell: https://learn.microsoft.com/en-us/powershell/module/applocker/

===== END FILE =====


===== FILE: skills/implementing-application-whitelisting-with-applocker/references/standards.md =====
# Standards & References - Implementing Application Whitelisting with AppLocker

## Primary Standards

### NIST SP 800-167 - Guide to Application Whitelisting
- **Publisher**: NIST
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-167/final
- **Scope**: Comprehensive guidance on planning, implementing, and maintaining application whitelisting
- **Key sections**: Technology overview, planning process, policy creation, maintenance operations

### ACSC Essential Eight - Application Control
- **Publisher**: Australian Cyber Security Centre
- **URL**: https://www.cyber.gov.au/resources-business-and-government/essential-cyber-security/essential-eight
- **Scope**: Application control is Mitigation Strategy #1 in the Essential Eight
- **Maturity levels**: L1 (block executables in user profiles), L2 (block from all user-writable paths), L3 (Microsoft recommended block rules + WDAC)

### CIS Control 2 - Software Inventory and Control
- **Publisher**: Center for Internet Security
- **Relevance**: CIS Controls v8 Control 2 requires software allowlisting for authorized applications

## Compliance Mappings

| Framework | Requirement | AppLocker Coverage |
|-----------|------------|-------------------|
| PCI DSS 4.0 | 6.4.3 - Restrict active content | AppLocker script rules block unauthorized scripts |
| NIST 800-53 | CM-7 - Least Functionality | AppLocker enforces minimum required software |
| NIST 800-53 | CM-11 - User-Installed Software | AppLocker prevents unauthorized software installation |
| NIST 800-171 | 3.4.8 - Application whitelisting | Direct requirement for application control |
| ISO 27001 | A.12.5.1 - Installation of software on operational systems | AppLocker restricts installation capability |
| HIPAA | 164.312(a)(1) - Access Control | Restricts executable access to authorized applications |

## Microsoft Documentation

- **AppLocker Overview**: https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/applocker/applocker-overview
- **AppLocker Policies Design Guide**: https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/applocker/applocker-policies-design-guide
- **WDAC and AppLocker Feature Availability**: Comparison of capabilities between AppLocker and WDAC
- **Microsoft Recommended Block Rules**: https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/design/applications-that-can-bypass-appcontrol

## Supporting References

- **LOLBAS Project**: https://lolbas-project.github.io/ - Living Off The Land Binaries reference for deny rule creation
- **AaronLocker (GitHub)**: Open-source toolkit for generating robust AppLocker policies
- **UltimateAppLockerByPassList**: Security research on AppLocker bypass techniques for defense awareness

===== END FILE =====


===== FILE: skills/implementing-application-whitelisting-with-applocker/references/workflows.md =====
# Workflows - Implementing Application Whitelisting with AppLocker

## Workflow 1: Initial AppLocker Deployment

```
[Application Inventory]
    │
    ├── Scan reference endpoints for installed applications
    ├── Catalog all approved software by publisher/path/hash
    ├── Identify admin tools vs. standard user applications
    │
    ▼
[Policy Design]
    │
    ├── Create default allow rules (Program Files, Windows)
    ├── Create publisher rules for third-party vendors
    ├── Create deny rules for LOLBins (standard users only)
    ├── Create script control rules
    │
    ▼
[Audit Mode Deployment]
    │
    ├── Deploy via GPO to pilot OU (Audit Only)
    ├── Enable Application Identity service
    ├── Monitor for 2-4 weeks
    │
    ▼
[Audit Log Analysis]
    │
    ├── Export blocked events (8003, 8006)
    ├── Identify legitimate applications being blocked
    │
    ├── Blocked app is legitimate ──► [Create allow rule]
    │                                       │
    │                                       ▼
    │                                  [Re-audit 1 week]
    │
    └── All blocked apps are unauthorized ──► [Proceed to enforcement]
                                                    │
                                                    ▼
                                               [Switch to Enforce mode (phased)]
                                                    │
                                                    ├── Week 1: EXE rules
                                                    ├── Week 2: Script rules
                                                    ├── Week 3: MSI rules
                                                    └── Week 4: DLL rules (optional)
```

## Workflow 2: New Application Approval

```
[User requests new application]
    │
    ▼
[Security review of application]
    │
    ├── Is it signed by trusted publisher? ──► [Create publisher rule]
    │
    ├── Unsigned but necessary? ──► [Create hash rule + document exception]
    │
    └── Fails security review ──► [Deny request, document reason]
    │
    ▼
[Add rule to AppLocker GPO]
    │
    ▼
[Deploy to pilot OU, verify no conflicts]
    │
    ▼
[Deploy to production OU]
    │
    ▼
[Update application inventory]
```

## Workflow 3: AppLocker Bypass Incident Response

```
[Detection: Unauthorized execution despite AppLocker]
    │
    ▼
[Identify bypass technique]
    │
    ├── LOLBin not blocked ──► [Add deny rule for specific binary]
    │
    ├── Execution from allowed path ──► [Restrict path rule scope]
    │
    ├── Admin user bypass ──► [Evaluate WDAC migration for admin enforcement]
    │
    └── DLL side-loading ──► [Enable DLL rules or deploy WDAC]
    │
    ▼
[Update AppLocker policy with fix]
    │
    ▼
[Verify fix in audit mode on test endpoint]
    │
    ▼
[Deploy fix to production]
    │
    ▼
[Update threat model and rule documentation]
```

## Workflow 4: AppLocker to WDAC Migration

```
[Decision: Migrate from AppLocker to WDAC]
    │
    ▼
[Audit current AppLocker policy]
    │
    ├── Export AppLocker rules as XML
    ├── Identify rules that need WDAC equivalents
    │
    ▼
[Create WDAC policy using WDAC Wizard]
    │
    ├── Convert publisher rules to WDAC signer rules
    ├── Convert path rules to WDAC filepath rules
    ├── Add Microsoft recommended block rules
    │
    ▼
[Deploy WDAC in Audit mode alongside AppLocker]
    │
    ▼
[Monitor WDAC audit events for 4 weeks]
    │
    ▼
[Resolve WDAC audit findings]
    │
    ▼
[Switch WDAC to Enforce mode]
    │
    ▼
[Disable AppLocker policy]
```

===== END FILE =====


===== FILE: skills/implementing-application-whitelisting-with-applocker/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing and auditing AppLocker application whitelisting policies."""

import json
import argparse
import re
import xml.etree.ElementTree as ET
from datetime import datetime
from pathlib import Path


APPLOCKER_EVENT_IDS = {
    8002: ("EXE/DLL allowed", "INFO"),
    8003: ("EXE/DLL denied", "HIGH"),
    8004: ("EXE/DLL would be denied (audit)", "MEDIUM"),
    8005: ("Script allowed", "INFO"),
    8006: ("Script denied", "HIGH"),
    8007: ("Script would be denied (audit)", "MEDIUM"),
    8020: ("Packaged app allowed", "INFO"),
    8021: ("Packaged app denied", "HIGH"),
    8022: ("Packaged app would be denied (audit)", "MEDIUM"),
}


def parse_applocker_policy(xml_path):
    """Parse an exported AppLocker policy XML file."""
    tree = ET.parse(xml_path)
    root = tree.getroot()
    ns = {"al": "http://schemas.microsoft.com/windows/2006/applocker"}
    rules = []
    for collection in root.findall(".//al:RuleCollection", ns):
        rule_type = collection.get("Type", "")
        mode = collection.get("EnforcementMode", "NotConfigured")
        for rule in collection:
            rule_data = {
                "collection": rule_type,
                "enforcement_mode": mode,
                "name": rule.get("Name", ""),
                "action": rule.get("Action", ""),
                "user_or_group": rule.get("UserOrGroupSid", ""),
                "type": rule.tag.replace(f"{{{ns.get('al', '')}}}", ""),
            }
            conditions = rule.findall(".//*")
            for cond in conditions:
                if "Path" in cond.tag:
                    rule_data["path"] = cond.get("Path", "")
                elif "Publisher" in cond.tag:
                    rule_data["publisher"] = cond.get("PublisherName", "")
                elif "Hash" in cond.tag:
                    rule_data["hash"] = cond.get("Data", "")
            rules.append(rule_data)
    return rules


def audit_applocker_rules(rules):
    """Audit AppLocker rules for security weaknesses."""
    findings = []
    for rule in rules:
        if rule.get("enforcement_mode") == "AuditOnly":
            findings.append({
                "collection": rule["collection"],
                "issue": "audit_mode_only",
                "severity": "MEDIUM",
                "recommendation": "Switch to Enforce mode after validation",
            })
        if rule.get("enforcement_mode") == "NotConfigured":
            findings.append({
                "collection": rule["collection"],
                "issue": "not_configured",
                "severity": "HIGH",
                "recommendation": "Enable enforcement for this rule collection",
            })
        path = rule.get("path", "")
        if path and rule.get("action") == "Allow":
            risky_paths = [r"\\Users\\", r"\\Temp\\", r"\\Downloads\\",
                          r"\\AppData\\", r"\\ProgramData\\"]
            for rp in risky_paths:
                if re.search(rp, path, re.IGNORECASE):
                    findings.append({
                        "rule_name": rule["name"],
                        "path": path,
                        "issue": "allow_from_user_writable_path",
                        "severity": "CRITICAL",
                    })
                    break
        if rule.get("user_or_group") == "S-1-1-0" and rule.get("action") == "Allow":
            findings.append({
                "rule_name": rule["name"],
                "issue": "allow_for_everyone",
                "severity": "MEDIUM",
            })
    return findings


def analyze_applocker_events(log_path):
    """Analyze AppLocker event logs for blocked and audit events."""
    events = []
    with open(log_path) as f:
        for line in f:
            try:
                entry = json.loads(line)
            except json.JSONDecodeError:
                continue
            event_id = int(entry.get("EventID", entry.get("event_id", 0)))
            if event_id in APPLOCKER_EVENT_IDS:
                desc, severity = APPLOCKER_EVENT_IDS[event_id]
                events.append({
                    "event_id": event_id,
                    "description": desc,
                    "severity": severity,
                    "timestamp": entry.get("TimeCreated", entry.get("timestamp", "")),
                    "computer": entry.get("Computer", entry.get("hostname", "")),
                    "user": entry.get("User", entry.get("user", "")),
                    "file_path": entry.get("FilePath", entry.get("file_path", "")),
                    "publisher": entry.get("Publisher", ""),
                })
    denied = [e for e in events if "denied" in e["description"].lower()]
    audit = [e for e in events if "audit" in e["description"].lower()]
    return {"total_events": len(events), "denied": denied, "audit_blocks": audit}


def generate_baseline_policy():
    """Generate a baseline AppLocker policy recommendation."""
    return {
        "exe_rules": {
            "enforcement_mode": "Enforce",
            "default_rules": [
                {"action": "Allow", "path": "%PROGRAMFILES%\\*", "scope": "Everyone"},
                {"action": "Allow", "path": "%WINDIR%\\*", "scope": "Everyone"},
                {"action": "Allow", "path": "*", "scope": "BUILTIN\\Administrators"},
            ],
        },
        "script_rules": {
            "enforcement_mode": "Enforce",
            "default_rules": [
                {"action": "Allow", "path": "%PROGRAMFILES%\\*", "scope": "Everyone"},
                {"action": "Allow", "path": "%WINDIR%\\*", "scope": "Everyone"},
            ],
        },
        "dll_rules": {
            "enforcement_mode": "AuditOnly",
            "note": "Start with audit mode due to high volume",
        },
    }


def main():
    parser = argparse.ArgumentParser(description="AppLocker Whitelisting Agent")
    parser.add_argument("--policy", help="Exported AppLocker policy XML")
    parser.add_argument("--events", help="AppLocker event log (JSON lines)")
    parser.add_argument("--output", default="applocker_audit_report.json")
    parser.add_argument("--action", choices=["audit", "events", "baseline", "full"],
                        default="full")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "findings": {}}

    if args.action in ("audit", "full") and args.policy:
        rules = parse_applocker_policy(args.policy)
        findings = audit_applocker_rules(rules)
        report["findings"]["policy_audit"] = findings
        report["findings"]["total_rules"] = len(rules)
        print(f"[+] Policy rules: {len(rules)}, Issues: {len(findings)}")

    if args.action in ("events", "full") and args.events:
        result = analyze_applocker_events(args.events)
        report["findings"]["event_analysis"] = result
        print(f"[+] Events: {result['total_events']}, Denied: {len(result['denied'])}")

    if args.action in ("baseline", "full"):
        baseline = generate_baseline_policy()
        report["findings"]["baseline_policy"] = baseline
        print("[+] Baseline policy generated")

    with open(args.output, "w") as fout:
        json.dump(report, fout, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-application-whitelisting-with-applocker/scripts/process.py =====
#!/usr/bin/env python3
"""
AppLocker Audit Log Analyzer

Parses Windows AppLocker audit event logs to identify blocked applications,
generate rule recommendations, and track policy effectiveness.
"""

import json
import csv
import sys
import os
import xml.etree.ElementTree as ET
from collections import defaultdict, Counter
from datetime import datetime


def parse_applocker_evtx_export(csv_path: str) -> list:
    """
    Parse AppLocker events exported from Event Viewer as CSV.
    Export: Event Viewer → AppLocker/EXE and DLL → Save All Events As → CSV
    """
    events = []

    with open(csv_path, "r", encoding="utf-8-sig") as f:
        reader = csv.DictReader(f)
        for row in reader:
            event = {
                "timestamp": row.get("Date and Time", ""),
                "event_id": row.get("Event ID", ""),
                "level": row.get("Level", ""),
                "source": row.get("Source", ""),
                "message": row.get("Message", ""),
                "user": "",
                "file_path": "",
                "file_hash": "",
                "publisher": "",
                "rule_name": "",
            }

            msg = event["message"]
            if msg:
                if "was allowed to run" in msg:
                    event["action"] = "allowed"
                elif "was prevented from running" in msg:
                    event["action"] = "blocked"
                elif "would have been prevented" in msg:
                    event["action"] = "audit_block"
                else:
                    event["action"] = "unknown"

                lines = msg.split("\n")
                for line in lines:
                    line = line.strip()
                    if line.startswith("File path:") or line.startswith("Filer path:"):
                        event["file_path"] = line.split(":", 1)[1].strip()
                    elif line.startswith("User:"):
                        event["user"] = line.split(":", 1)[1].strip()
                    elif line.startswith("Publisher name:"):
                        event["publisher"] = line.split(":", 1)[1].strip()
                    elif line.startswith("File hash:"):
                        event["file_hash"] = line.split(":", 1)[1].strip()
                    elif line.startswith("Rule name:"):
                        event["rule_name"] = line.split(":", 1)[1].strip()

            events.append(event)

    return events


def analyze_blocked_applications(events: list) -> dict:
    """Analyze which applications are being blocked and why."""
    analysis = {
        "total_events": len(events),
        "blocked_events": 0,
        "allowed_events": 0,
        "audit_block_events": 0,
        "unique_blocked_files": set(),
        "blocked_by_path": defaultdict(int),
        "blocked_by_user": defaultdict(int),
        "blocked_by_publisher": defaultdict(int),
        "top_blocked_files": Counter(),
        "user_writable_blocks": [],
        "potential_legitimate": [],
    }

    user_writable_indicators = [
        "\\users\\", "\\appdata\\", "\\temp\\", "\\downloads\\",
        "\\desktop\\", "\\documents\\", "%temp%", "%appdata%",
    ]

    signed_publishers = set()

    for event in events:
        action = event.get("action", "")
        if action in ("blocked", "audit_block"):
            analysis["blocked_events"] += 1
            file_path = event.get("file_path", "").lower()
            user = event.get("user", "Unknown")
            publisher = event.get("publisher", "Unknown")

            analysis["unique_blocked_files"].add(file_path)
            analysis["blocked_by_path"][file_path] += 1
            analysis["blocked_by_user"][user] += 1
            analysis["top_blocked_files"][file_path] += 1

            if publisher and publisher != "Unknown":
                analysis["blocked_by_publisher"][publisher] += 1
                signed_publishers.add(publisher)

            is_user_writable = any(ind in file_path for ind in user_writable_indicators)
            if is_user_writable:
                analysis["user_writable_blocks"].append({
                    "file": file_path,
                    "user": user,
                    "timestamp": event.get("timestamp", ""),
                })

            if publisher and publisher != "Unknown" and not is_user_writable:
                analysis["potential_legitimate"].append({
                    "file": file_path,
                    "publisher": publisher,
                    "user": user,
                    "recommendation": "Consider creating publisher rule",
                })

        elif action == "allowed":
            analysis["allowed_events"] += 1

        if action == "audit_block":
            analysis["audit_block_events"] += 1

    analysis["unique_blocked_files"] = len(analysis["unique_blocked_files"])
    analysis["signed_publishers_blocked"] = list(signed_publishers)

    return analysis


def generate_rule_recommendations(analysis: dict) -> list:
    """Generate AppLocker rule recommendations based on audit analysis."""
    recommendations = []

    for item in analysis.get("potential_legitimate", []):
        recommendations.append({
            "type": "publisher_rule",
            "action": "allow",
            "publisher": item["publisher"],
            "file": item["file"],
            "reason": f"Signed application blocked for user {item['user']}",
            "priority": "high",
        })

    top_blocked = analysis.get("top_blocked_files", Counter())
    for file_path, count in top_blocked.most_common(20):
        if count >= 10:
            is_in_recommendations = any(
                r["file"] == file_path for r in recommendations
            )
            if not is_in_recommendations:
                recommendations.append({
                    "type": "investigate",
                    "action": "review",
                    "file": file_path,
                    "count": count,
                    "reason": f"Blocked {count} times - determine if legitimate",
                    "priority": "medium",
                })

    return recommendations


def export_analysis_report(analysis: dict, recommendations: list, output_path: str) -> None:
    """Export analysis and recommendations to JSON report."""
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "summary": {
            "total_events": analysis["total_events"],
            "blocked": analysis["blocked_events"],
            "audit_blocked": analysis["audit_block_events"],
            "allowed": analysis["allowed_events"],
            "unique_blocked_files": analysis["unique_blocked_files"],
        },
        "top_blocked_files": dict(analysis["top_blocked_files"].most_common(30)),
        "blocked_by_user": dict(analysis["blocked_by_user"]),
        "signed_publishers_blocked": analysis.get("signed_publishers_blocked", []),
        "user_writable_path_blocks": len(analysis.get("user_writable_blocks", [])),
        "rule_recommendations": recommendations,
    }

    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2)


def export_blocked_apps_csv(analysis: dict, output_path: str) -> None:
    """Export blocked applications to CSV for review."""
    with open(output_path, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow(["File Path", "Block Count", "Action Needed", "Rule Type"])
        for file_path, count in analysis["top_blocked_files"].most_common(100):
            writer.writerow([file_path, count, "Review", "TBD"])


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <applocker_events.csv>")
        print()
        print("Analyzes AppLocker audit events exported from Windows Event Viewer.")
        print("Export events from: Event Viewer → AppLocker/EXE and DLL → Save All Events As CSV")
        sys.exit(1)

    csv_path = sys.argv[1]
    if not os.path.exists(csv_path):
        print(f"Error: File not found: {csv_path}")
        sys.exit(1)

    print("Parsing AppLocker audit events...")
    events = parse_applocker_evtx_export(csv_path)
    print(f"Parsed {len(events)} events")

    print("Analyzing blocked applications...")
    analysis = analyze_blocked_applications(events)

    print("Generating rule recommendations...")
    recommendations = generate_rule_recommendations(analysis)

    base_name = os.path.splitext(os.path.basename(csv_path))[0]
    output_dir = os.path.dirname(csv_path) or "."

    report_path = os.path.join(output_dir, f"{base_name}_analysis.json")
    export_analysis_report(analysis, recommendations, report_path)
    print(f"Analysis report: {report_path}")

    blocked_csv = os.path.join(output_dir, f"{base_name}_blocked_apps.csv")
    export_blocked_apps_csv(analysis, blocked_csv)
    print(f"Blocked apps CSV: {blocked_csv}")

    print(f"\n--- AppLocker Audit Summary ---")
    print(f"Total events: {analysis['total_events']}")
    print(f"Blocked: {analysis['blocked_events']}")
    print(f"Audit-blocked: {analysis['audit_block_events']}")
    print(f"Unique blocked files: {analysis['unique_blocked_files']}")
    print(f"Rule recommendations: {len(recommendations)}")

===== END FILE =====


===== FILE: skills/implementing-disk-encryption-with-bitlocker/SKILL.md =====
---
name: implementing-disk-encryption-with-bitlocker
description: 'Implements full disk encryption using Microsoft BitLocker on Windows
  endpoints to protect data at rest from unauthorized access in case of device loss
  or theft. Use when deploying encryption for compliance requirements, securing mobile
  workstations, or implementing data protection controls across the enterprise. Activates
  for requests involving BitLocker encryption, disk encryption, TPM configuration,
  or data-at-rest protection.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- encryption
- BitLocker
- TPM
- data-protection
- windows-security
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
- T1573
---
# Implementing Disk Encryption with BitLocker

## When to Use

Use this skill when:
- Encrypting Windows endpoints to protect data at rest for compliance (PCI DSS, HIPAA, GDPR)
- Deploying BitLocker across enterprise fleet via Intune, SCCM, or GPO
- Configuring TPM-based encryption with PIN or USB startup key for enhanced security
- Managing BitLocker recovery keys in Active Directory or Azure AD

**Do not use** this skill for Linux disk encryption (use LUKS/dm-crypt) or macOS (use FileVault).

## Prerequisites

- Windows 10/11 Pro, Enterprise, or Education edition
- TPM 2.0 chip (recommended; TPM 1.2 supported with limitations)
- UEFI firmware with Secure Boot enabled (recommended)
- Separate system partition (200 MB minimum, created automatically by Windows installer)
- Active Directory or Azure AD for recovery key escrow

## Workflow

### Step 1: Verify TPM and System Requirements

```powershell
# Check TPM status
Get-Tpm
# ManufacturerId, ManufacturerVersion, TpmPresent, TpmReady, TpmEnabled

# Check TPM version (2.0 required for best compatibility)
(Get-WmiObject -Namespace "root\cimv2\security\microsofttpm" -Class Win32_Tpm).SpecVersion

# Check UEFI/Secure Boot
Confirm-SecureBootUEFI
# Returns True if Secure Boot is enabled

# Check BitLocker readiness
$vol = Get-BitLockerVolume -MountPoint "C:"
$vol.VolumeStatus  # Should be "FullyDecrypted"
$vol.ProtectionStatus  # Should be "Off"
```

### Step 2: Configure BitLocker GPO Settings

```
Computer Configuration → Administrative Templates → Windows Components → BitLocker Drive Encryption

Operating System Drives:
  - Require additional authentication at startup: Enabled
    - Allow BitLocker without compatible TPM: Disabled (enforce TPM)
    - Configure TPM startup: Allow TPM
    - Configure TPM startup PIN: Allow startup PIN with TPM
    - Configure TPM startup key: Allow startup key with TPM

  - Choose how BitLocker-protected OS drives can be recovered: Enabled
    - Allow data recovery agent: True
    - Configure storage of recovery information to AD DS: Enabled
    - Save recovery info to AD DS for OS drives: Store recovery passwords and key packages
    - Do not enable BitLocker until recovery information is stored: Enabled

  - Choose drive encryption method and cipher strength:
    - OS drives: XTS-AES 256-bit (Windows 10 1511+)
    - Fixed drives: XTS-AES 256-bit
    - Removable drives: AES-CBC 256-bit (for cross-platform compatibility)

Fixed Data Drives:
  - Choose how BitLocker-protected fixed drives can be recovered: Enabled
    - Store recovery passwords in AD DS: Enabled

Removable Data Drives:
  - Control use of BitLocker on removable drives: Enabled
  - Configure use of passwords for removable drives: Require complexity
```

### Step 3: Enable BitLocker - Command Line

```powershell
# Enable BitLocker with TPM-only protector (transparent to user)
Enable-BitLocker -MountPoint "C:" -EncryptionMethod XtsAes256 `
  -TpmProtector -SkipHardwareTest

# Enable BitLocker with TPM + PIN (recommended for laptops)
$pin = ConvertTo-SecureString "123456" -AsPlainText -Force
Enable-BitLocker -MountPoint "C:" -EncryptionMethod XtsAes256 `
  -TpmAndPinProtector -Pin $pin

# Add recovery password protector
Add-BitLockerKeyProtector -MountPoint "C:" -RecoveryPasswordProtector

# Backup recovery key to Active Directory
Backup-BitLockerKeyProtector -MountPoint "C:" `
  -KeyProtectorId (Get-BitLockerVolume -MountPoint "C:").KeyProtector[1].KeyProtectorId

# Encrypt fixed data drives
Enable-BitLocker -MountPoint "D:" -EncryptionMethod XtsAes256 `
  -RecoveryPasswordProtector -AutoUnlockEnabled
```

### Step 4: Deploy via Intune (Enterprise)

```
Intune → Endpoint Security → Disk encryption → Create Profile

Platform: Windows 10 and later
Profile: BitLocker

Settings:
  BitLocker base settings:
    - Encryption for operating system drives: Require
    - Encryption for fixed data drives: Require
    - Encryption for removable data drives: Require

  Operating system drive settings:
    - Additional authentication at startup: Require
    - TPM startup: Allowed
    - TPM startup PIN: Required (for high-security endpoints)
    - Encryption method: XTS-AES 256-bit
    - Recovery: Escrow to Azure AD

  Fixed drive settings:
    - Encryption method: XTS-AES 256-bit
    - Recovery: Escrow to Azure AD

  Assign to: All managed Windows devices (or specific groups)
```

### Step 5: Manage Recovery Keys

```powershell
# View recovery key on local system
(Get-BitLockerVolume -MountPoint "C:").KeyProtector |
  Where-Object {$_.KeyProtectorType -eq "RecoveryPassword"} |
  Select-Object KeyProtectorId, RecoveryPassword

# Retrieve recovery key from Active Directory (requires RSAT)
Get-ADObject -Filter {objectClass -eq "msFVE-RecoveryInformation"} `
  -SearchBase "CN=COMPUTER01,OU=Workstations,DC=corp,DC=example,DC=com" `
  -Properties msFVE-RecoveryPassword |
  Select-Object -ExpandProperty msFVE-RecoveryPassword

# Retrieve recovery key from Azure AD
# Azure Portal → Azure AD → Devices → [device] → BitLocker keys
# Or via Microsoft Graph API:
# GET /devices/{id}/bitlockerRecoveryKeys
```

### Step 6: Monitor Encryption Status

```powershell
# Check encryption status across fleet
manage-bde -status C:

# Expected output for encrypted drive:
#   Conversion Status: Fully Encrypted
#   Percentage Encrypted: 100.0%
#   Encryption Method: XTS-AES 256
#   Protection Status: Protection On
#   Key Protectors: TPM, Numerical Password

# PowerShell compliance check
$vol = Get-BitLockerVolume -MountPoint "C:"
if ($vol.ProtectionStatus -eq "On" -and $vol.VolumeStatus -eq "FullyEncrypted") {
    Write-Host "COMPLIANT: BitLocker enabled and fully encrypted"
} else {
    Write-Host "NON-COMPLIANT: BitLocker status - Protection: $($vol.ProtectionStatus), Volume: $($vol.VolumeStatus)"
}
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **TPM (Trusted Platform Module)** | Hardware security chip that stores BitLocker encryption keys and provides measured boot integrity |
| **XTS-AES 256** | Encryption cipher used by BitLocker; XTS mode provides better protection for disk encryption than CBC |
| **Recovery Key** | 48-digit numerical password used to unlock BitLocker-encrypted drive when TPM authentication fails |
| **Key Protector** | Method used to unlock BitLocker (TPM, TPM+PIN, recovery password, startup key, smart card) |
| **Used Space Only Encryption** | Encrypts only sectors containing data; faster initial encryption but may leave remnant data in free space |
| **Full Disk Encryption** | Encrypts entire volume including free space; slower but more secure for drives that previously contained data |

## Tools & Systems

- **BitLocker (built-in)**: Windows full disk encryption feature
- **manage-bde.exe**: Command-line BitLocker management tool
- **BitLocker Recovery Password Viewer**: RSAT tool for viewing recovery keys in Active Directory
- **MBAM (Microsoft BitLocker Administration and Monitoring)**: Enterprise BitLocker management (legacy, replaced by Intune)
- **Microsoft Intune**: Cloud-based BitLocker policy deployment and recovery key management

## Common Pitfalls

- **Not escrowing recovery keys before encryption**: If recovery keys are not saved to AD/Azure AD before encryption, they may be permanently lost if the TPM fails.
- **Using TPM-only without PIN**: TPM-only mode is transparent but vulnerable to cold boot attacks and evil maid attacks. Add a startup PIN for laptops leaving the office.
- **Encrypting used space only on repurposed drives**: If a drive previously contained sensitive data, "used space only" encryption leaves deleted data unencrypted in free space. Use full disk encryption for repurposed drives.
- **Forgetting removable drives**: USB drives and external disks are common data loss vectors. Enforce BitLocker To Go for removable media.
- **No pre-provisioning for SCCM deployments**: Pre-provision BitLocker during OSD task sequence to encrypt before OS deployment, avoiding the lengthy post-deployment encryption process.

===== END FILE =====


===== FILE: skills/implementing-disk-encryption-with-bitlocker/LICENSE =====

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


===== FILE: skills/implementing-disk-encryption-with-bitlocker/assets/template.md =====
# BitLocker Disk Encryption Template

## Endpoint Information

| Field | Value |
|-------|-------|
| Hostname | |
| OS Version | |
| TPM Version | 2.0 |
| UEFI/Secure Boot | Enabled |
| Encryption Date | |

## Encryption Configuration

| Setting | OS Drive | Fixed Drives | Removable |
|---------|----------|-------------|-----------|
| Encryption Method | XTS-AES 256 | XTS-AES 256 | AES-CBC 256 |
| Key Protectors | TPM + PIN | Auto-unlock | Password |
| Recovery Key Escrow | AD DS / Azure AD | AD DS / Azure AD | N/A |
| Full/Used Space | Full disk | Used space only | Used space only |

## Recovery Key Register

| Volume | Key Protector ID | Escrowed To | Verified |
|--------|-----------------|-------------|----------|
| C: | | AD DS / Azure AD | Yes / No |
| D: | | AD DS / Azure AD | Yes / No |

## Compliance Checklist

- [ ] TPM 2.0 present and enabled
- [ ] Secure Boot enabled
- [ ] OS drive encrypted with XTS-AES 256
- [ ] Recovery key escrowed to AD/Azure AD
- [ ] PIN configured for laptop endpoints
- [ ] Fixed data drives encrypted
- [ ] Removable drive encryption policy active
- [ ] BitLocker cannot be disabled without admin

## Sign-Off

| Role | Name | Date |
|------|------|------|
| Endpoint Admin | | |
| Security Analyst | | |

===== END FILE =====


===== FILE: skills/implementing-disk-encryption-with-bitlocker/references/api-reference.md =====
# API Reference: Implementing Disk Encryption with BitLocker

## manage-bde CLI

```powershell
# Check status
manage-bde -status C:

# Enable BitLocker with TPM
manage-bde -on C: -RecoveryPassword -EncryptionMethod AES256

# Backup recovery key to AD
manage-bde -protectors -adbackup C: -ID {protector-id}

# Lock/unlock
manage-bde -lock D:
manage-bde -unlock D: -RecoveryPassword 123456-...
```

## PowerShell BitLocker Cmdlets

```powershell
# Get BitLocker volume
Get-BitLockerVolume -MountPoint "C:"

# Enable with TPM + PIN
Enable-BitLocker -MountPoint "C:" -EncryptionMethod Aes256 `
  -TpmAndPinProtector -Pin (ConvertTo-SecureString "1234" -AsPlainText -Force)

# Add recovery password
Add-BitLockerKeyProtector -MountPoint "C:" -RecoveryPasswordProtector

# Backup to AD
Backup-BitLockerKeyProtector -MountPoint "C:" -KeyProtectorId $id
```

## Compliance Checks

| Check | Severity | Requirement |
|-------|----------|-------------|
| BitLocker enabled | CRITICAL | All OS drives |
| AES-256 encryption | MEDIUM | FIPS/enterprise |
| TPM protector | HIGH | Hardware-backed |
| Recovery key escrowed | HIGH | AD DS or Azure AD |
| Full disk encrypted | MEDIUM | Not used-space only |

## Microsoft Graph API (Intune)

```python
import requests
headers = {"Authorization": "Bearer <token>"}
resp = requests.get(
    "https://graph.microsoft.com/v1.0/deviceManagement/managedDevices"
    "?$select=deviceName,isEncrypted",
    headers=headers)
```

### References

- BitLocker: https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/
- BitLocker PowerShell: https://learn.microsoft.com/en-us/powershell/module/bitlocker/

===== END FILE =====


===== FILE: skills/implementing-disk-encryption-with-bitlocker/references/standards.md =====
# Standards & References - Implementing Disk Encryption with BitLocker

## Primary Standards

### NIST SP 800-111 - Guide to Storage Encryption Technologies
- **Publisher**: NIST
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-111/final
- **Scope**: Guidance on planning, implementing, and maintaining storage encryption solutions

### FIPS 140-2/3 - Security Requirements for Cryptographic Modules
- **Publisher**: NIST
- **Relevance**: BitLocker uses FIPS 140-2 validated cryptographic modules when configured in FIPS-compliant mode

## Compliance Mappings

| Framework | Requirement | BitLocker Coverage |
|-----------|------------|-------------------|
| PCI DSS 4.0 | 3.5.1 - Render PAN unreadable in storage | BitLocker full disk encryption |
| HIPAA | 164.312(a)(2)(iv) - Encryption/decryption | BitLocker protects ePHI at rest |
| GDPR | Article 32(1)(a) - Encryption of personal data | BitLocker for data-at-rest protection |
| NIST 800-53 | SC-28 Protection of Information at Rest | BitLocker encryption |
| NIST 800-171 | 3.13.16 - Confidentiality of CUI at rest | BitLocker on CUI systems |
| ISO 27001 | A.10.1.1 - Cryptographic controls policy | BitLocker implementation |

## Microsoft References

- **BitLocker Overview**: https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/
- **BitLocker Group Policy Reference**: https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/bitlocker-group-policy-settings
- **BitLocker Recovery Guide**: https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/bitlocker-recovery-guide-plan
- **TPM Recommendations**: https://learn.microsoft.com/en-us/windows/security/hardware-security/tpm/tpm-recommendations

===== END FILE =====


===== FILE: skills/implementing-disk-encryption-with-bitlocker/references/workflows.md =====
# Workflows - Implementing Disk Encryption with BitLocker

## Workflow 1: Enterprise BitLocker Deployment

```
[Pre-deployment assessment]
    │
    ├── Verify TPM 2.0 across fleet
    ├── Confirm UEFI/Secure Boot
    ├── Plan recovery key escrow (AD DS or Azure AD)
    │
    ▼
[Configure GPO/Intune policy]
    │
    ├── Set encryption method (XTS-AES 256)
    ├── Configure key protectors (TPM + PIN for laptops, TPM for desktops)
    ├── Enable recovery key escrow
    │
    ▼
[Pilot deployment (test group)]
    │
    ├── Verify encryption completes without errors
    ├── Test recovery key retrieval
    ├── Verify no boot issues
    │
    ▼
[Production rollout (phased)]
    │
    ▼
[Monitor encryption status via Intune/SCCM reports]
    │
    ▼
[Verify 100% coverage, address failures]
```

## Workflow 2: BitLocker Recovery Process

```
[User locked out (BitLocker recovery screen)]
    │
    ▼
[User provides Recovery Key ID to helpdesk]
    │
    ▼
[Helpdesk retrieves recovery key]
    │
    ├── AD DS: RSAT BitLocker Recovery Password Viewer
    ├── Azure AD: Azure Portal → Devices → BitLocker keys
    ├── Intune: Intune Portal → Devices → Recovery keys
    │
    ▼
[User enters 48-digit recovery key]
    │
    ▼
[Investigate why recovery was triggered]
    │
    ├── BIOS/firmware update ──► [Expected, no action]
    ├── TPM failure ──► [Replace TPM or re-encrypt]
    ├── Boot configuration change ──► [Review change, re-seal TPM]
    └── Potential tampering ──► [Security investigation]
```

## Workflow 3: Key Rotation

```
[Quarterly key rotation policy]
    │
    ▼
[Generate new recovery password]
    │
    ▼
[Backup new key to AD/Azure AD]
    │
    ▼
[Remove old recovery password protector]
    │
    ▼
[Verify new key works in test recovery]
```

===== END FILE =====


===== FILE: skills/implementing-disk-encryption-with-bitlocker/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for auditing and managing BitLocker disk encryption across endpoints."""

import json
import argparse
import subprocess
from datetime import datetime


def get_bitlocker_status():
    """Get BitLocker status on local machine via manage-bde."""
    try:
        result = subprocess.run(
            ["manage-bde", "-status"], capture_output=True, text=True, timeout=30)
        volumes = []
        current = {}
        for line in result.stdout.splitlines():
            line = line.strip()
            if line.startswith("Volume"):
                if current:
                    volumes.append(current)
                current = {"volume": line}
            elif ":" in line:
                key, _, value = line.partition(":")
                current[key.strip()] = value.strip()
        if current:
            volumes.append(current)
        return volumes
    except (subprocess.TimeoutExpired, FileNotFoundError):
        return []


def parse_bitlocker_report(report_path):
    """Parse BitLocker compliance report (CSV or JSON)."""
    entries = []
    if report_path.endswith(".json"):
        with open(report_path) as f:
            entries = json.load(f)
    else:
        import csv
        with open(report_path, newline="", encoding="utf-8-sig") as f:
            entries = list(csv.DictReader(f))
    return entries


def audit_bitlocker_compliance(devices):
    """Audit BitLocker compliance across fleet."""
    findings = []
    for device in devices:
        hostname = device.get("hostname", device.get("ComputerName", ""))
        protection = device.get("protection_status", device.get("ProtectionStatus", ""))
        encryption = device.get("encryption_method", device.get("EncryptionMethod", ""))
        key_protector = device.get("key_protector", device.get("KeyProtector", ""))
        recovery_key = device.get("recovery_key_escrowed",
                                  device.get("RecoveryKeyEscrowed", ""))
        if "off" in str(protection).lower() or protection == "0":
            findings.append({
                "hostname": hostname, "issue": "bitlocker_disabled",
                "severity": "CRITICAL",
            })
        if encryption and "aes" not in str(encryption).lower():
            findings.append({
                "hostname": hostname, "issue": "weak_encryption_method",
                "value": encryption, "severity": "HIGH",
            })
        if "128" in str(encryption):
            findings.append({
                "hostname": hostname, "issue": "aes_128_not_256",
                "severity": "MEDIUM",
                "recommendation": "Upgrade to AES-256",
            })
        if not key_protector or "tpm" not in str(key_protector).lower():
            findings.append({
                "hostname": hostname, "issue": "no_tpm_protector",
                "severity": "HIGH",
            })
        if str(recovery_key).lower() in ("no", "false", "0", ""):
            findings.append({
                "hostname": hostname, "issue": "recovery_key_not_escrowed",
                "severity": "HIGH",
                "recommendation": "Escrow recovery key to Active Directory",
            })
    return findings


def generate_gpo_recommendations():
    """Generate Group Policy recommendations for BitLocker."""
    return {
        "Computer Configuration": {
            "path": "Administrative Templates > Windows Components > BitLocker Drive Encryption",
            "settings": [
                {"name": "Choose drive encryption method (OS)",
                 "value": "AES-256", "policy": "Enabled"},
                {"name": "Require additional authentication at startup",
                 "value": "Allow BitLocker without compatible TPM: Disabled",
                 "policy": "Enabled"},
                {"name": "Choose how BitLocker-protected OS drives can be recovered",
                 "value": "Save to AD DS, Do not enable until stored",
                 "policy": "Enabled"},
                {"name": "Enforce drive encryption type on OS drives",
                 "value": "Full encryption", "policy": "Enabled"},
            ],
        },
    }


def calculate_compliance_metrics(devices, findings):
    """Calculate fleet encryption compliance metrics."""
    total = len(devices)
    encrypted = total - sum(1 for f in findings if f["issue"] == "bitlocker_disabled")
    strong_enc = encrypted - sum(1 for f in findings if f["issue"] in
                                  ("weak_encryption_method", "aes_128_not_256"))
    escrowed = total - sum(1 for f in findings if f["issue"] == "recovery_key_not_escrowed")
    return {
        "total_devices": total,
        "encrypted": encrypted,
        "encryption_rate": round(encrypted / total * 100, 1) if total else 0,
        "strong_encryption": strong_enc,
        "recovery_keys_escrowed": escrowed,
        "escrow_rate": round(escrowed / total * 100, 1) if total else 0,
    }


def main():
    parser = argparse.ArgumentParser(description="BitLocker Disk Encryption Agent")
    parser.add_argument("--report", help="BitLocker report CSV/JSON")
    parser.add_argument("--local", action="store_true", help="Check local machine")
    parser.add_argument("--output", default="bitlocker_audit_report.json")
    parser.add_argument("--action", choices=["audit", "local", "gpo", "full"], default="full")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "findings": {}}

    if args.action in ("local", "full") and args.local:
        status = get_bitlocker_status()
        report["findings"]["local_status"] = status
        print(f"[+] Local volumes: {len(status)}")

    if args.action in ("audit", "full") and args.report:
        devices = parse_bitlocker_report(args.report)
        findings = audit_bitlocker_compliance(devices)
        metrics = calculate_compliance_metrics(devices, findings)
        report["findings"]["compliance_audit"] = findings
        report["findings"]["metrics"] = metrics
        print(f"[+] Devices: {metrics['total_devices']}, Encrypted: {metrics['encryption_rate']}%")

    if args.action in ("gpo", "full"):
        gpo = generate_gpo_recommendations()
        report["findings"]["gpo_recommendations"] = gpo
        print("[+] GPO recommendations generated")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-disk-encryption-with-bitlocker/scripts/process.py =====
#!/usr/bin/env python3
"""
BitLocker Compliance Checker

Checks BitLocker encryption status across endpoints and generates
compliance reports for audit purposes.
"""

import json
import subprocess
import sys
import os
import csv
from datetime import datetime


def check_bitlocker_status() -> dict:
    """Check BitLocker status on local Windows endpoint."""
    ps_cmd = """
    $volumes = Get-BitLockerVolume
    $results = @()
    foreach ($vol in $volumes) {
        $protectors = @()
        foreach ($kp in $vol.KeyProtector) {
            $protectors += @{
                Type = $kp.KeyProtectorType.ToString()
                Id = $kp.KeyProtectorId
            }
        }
        $results += @{
            MountPoint = $vol.MountPoint
            VolumeStatus = $vol.VolumeStatus.ToString()
            ProtectionStatus = $vol.ProtectionStatus.ToString()
            EncryptionMethod = $vol.EncryptionMethod.ToString()
            EncryptionPercentage = $vol.EncryptionPercentage
            VolumeType = $vol.VolumeType.ToString()
            KeyProtectors = $protectors
            AutoUnlockEnabled = $vol.AutoUnlockEnabled
            AutoUnlockKeyStored = $vol.AutoUnlockKeyStored
        }
    }
    $tpm = Get-Tpm
    @{
        Hostname = $env:COMPUTERNAME
        Volumes = $results
        TPM = @{
            Present = $tpm.TpmPresent
            Ready = $tpm.TpmReady
            Enabled = $tpm.TpmEnabled
            Activated = $tpm.TpmActivated
        }
    } | ConvertTo-Json -Depth 4
    """

    try:
        result = subprocess.run(
            ["powershell", "-NoProfile", "-Command", ps_cmd],
            capture_output=True, text=True, timeout=30,
        )
        if result.returncode == 0 and result.stdout.strip():
            return json.loads(result.stdout)
        return {"error": result.stderr or "Failed to check BitLocker status"}
    except FileNotFoundError:
        return {"error": "PowerShell not available (requires Windows)"}
    except subprocess.TimeoutExpired:
        return {"error": "Command timed out"}
    except json.JSONDecodeError as e:
        return {"error": f"Parse error: {e}"}


def assess_compliance(status: dict) -> dict:
    """Assess BitLocker compliance against security baseline."""
    findings = {
        "hostname": status.get("Hostname", "unknown"),
        "overall_compliant": True,
        "tpm_status": "compliant",
        "volume_findings": [],
    }

    tpm = status.get("TPM", {})
    if not tpm.get("Present") or not tpm.get("Ready"):
        findings["tpm_status"] = "non-compliant"
        findings["overall_compliant"] = False

    for vol in status.get("Volumes", []):
        vol_finding = {
            "mount_point": vol["MountPoint"],
            "compliant": True,
            "issues": [],
        }

        if vol["VolumeStatus"] != "FullyEncrypted":
            vol_finding["compliant"] = False
            vol_finding["issues"].append(f"Not fully encrypted: {vol['VolumeStatus']}")

        if vol["ProtectionStatus"] != "On":
            vol_finding["compliant"] = False
            vol_finding["issues"].append(f"Protection not enabled: {vol['ProtectionStatus']}")

        if vol["EncryptionMethod"] not in ("XtsAes256", "XtsAes128"):
            vol_finding["issues"].append(f"Weak encryption method: {vol['EncryptionMethod']}")

        protector_types = [kp["Type"] for kp in vol.get("KeyProtectors", [])]
        has_recovery = "RecoveryPassword" in protector_types or "NumericalPassword" in protector_types
        if not has_recovery:
            vol_finding["compliant"] = False
            vol_finding["issues"].append("No recovery password protector configured")

        has_tpm = any(t in protector_types for t in ["Tpm", "TpmPin", "TpmPinStartupKey"])
        if vol["VolumeType"] == "OperatingSystem" and not has_tpm:
            vol_finding["compliant"] = False
            vol_finding["issues"].append("OS volume missing TPM protector")

        if not vol_finding["compliant"]:
            findings["overall_compliant"] = False

        findings["volume_findings"].append(vol_finding)

    return findings


def generate_report(status: dict, compliance: dict, output_path: str) -> None:
    """Generate BitLocker compliance report."""
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "hostname": status.get("Hostname", "unknown"),
        "overall_compliance": "COMPLIANT" if compliance["overall_compliant"] else "NON-COMPLIANT",
        "tpm_status": compliance["tpm_status"],
        "tpm_details": status.get("TPM", {}),
        "volumes": [],
    }

    for vol, finding in zip(status.get("Volumes", []), compliance["volume_findings"]):
        report["volumes"].append({
            "mount_point": vol["MountPoint"],
            "status": vol["VolumeStatus"],
            "protection": vol["ProtectionStatus"],
            "encryption_method": vol["EncryptionMethod"],
            "encryption_percent": vol["EncryptionPercentage"],
            "key_protectors": [kp["Type"] for kp in vol.get("KeyProtectors", [])],
            "compliant": finding["compliant"],
            "issues": finding["issues"],
        })

    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2)


if __name__ == "__main__":
    output_dir = sys.argv[1] if len(sys.argv) > 1 else "."

    print("Checking BitLocker status...")
    status = check_bitlocker_status()

    if "error" in status:
        print(f"Error: {status['error']}")
        print("This tool requires Windows with BitLocker support.")
        sys.exit(1)

    print("Assessing compliance...")
    compliance = assess_compliance(status)

    report_path = os.path.join(output_dir, "bitlocker_compliance.json")
    generate_report(status, compliance, report_path)
    print(f"Compliance report: {report_path}")

    print(f"\n--- BitLocker Compliance ---")
    print(f"Hostname: {compliance['hostname']}")
    print(f"Overall: {'COMPLIANT' if compliance['overall_compliant'] else 'NON-COMPLIANT'}")
    print(f"TPM: {compliance['tpm_status']}")

    for vf in compliance["volume_findings"]:
        status_icon = "PASS" if vf["compliant"] else "FAIL"
        print(f"\n  [{status_icon}] {vf['mount_point']}")
        for issue in vf["issues"]:
            print(f"    - {issue}")

===== END FILE =====


===== FILE: skills/implementing-endpoint-dlp-controls/SKILL.md =====
---
name: implementing-endpoint-dlp-controls
description: 'Implements endpoint Data Loss Prevention (DLP) controls to detect and
  prevent sensitive data exfiltration through email, USB, cloud storage, and printing.
  Use when deploying DLP agents, creating content inspection policies, or preventing
  unauthorized data movement from endpoints. Activates for requests involving DLP,
  data exfiltration prevention, content inspection, or sensitive data protection on
  endpoints.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- DLP
- data-loss-prevention
- data-protection
- content-inspection
version: 1.0.0
author: mahipal
license: Apache-2.0
atlas_techniques:
- AML.T0024
- AML.T0056
nist_ai_rmf:
- GOVERN-1.1
- MEASURE-2.7
- MANAGE-3.1
- MAP-5.1
- MANAGE-2.4
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
- T1048
---
# Implementing Endpoint DLP Controls

## When to Use

Use this skill when:
- Deploying endpoint DLP to prevent sensitive data (PII, PHI, PCI) from leaving the organization
- Configuring content inspection rules for email attachments, USB transfers, and cloud uploads
- Implementing Microsoft Purview DLP or Symantec DLP endpoint policies
- Meeting compliance requirements for data protection (GDPR, HIPAA, PCI DSS)

**Do not use** for network DLP (inline proxy-based) or cloud-only DLP (CASB).

## Prerequisites

- Microsoft 365 E5 or standalone Microsoft Purview DLP license
- Microsoft Purview compliance portal access (compliance.microsoft.com)
- Sensitive Information Types (SITs) defined for organization data
- Endpoint onboarded to Microsoft Purview (via Intune or SCCM)

## Workflow

### Step 1: Define Sensitive Information Types

```
Microsoft Purview → Data Classification → Sensitive info types

Built-in SITs for common data:
- Credit card number (PCI)
- Social Security Number (PII)
- Health records (HIPAA)
- Passport number
- Bank account number

Custom SIT example (Employee ID):
  Pattern: EMP-[0-9]{6}
  Confidence: High
  Keywords: "employee id", "emp id", "staff number"
```

### Step 2: Create DLP Policy

```
Microsoft Purview → Data loss prevention → Policies → Create policy

Policy Configuration:
1. Template: Financial / Medical / PII (or custom)
2. Locations: Devices (endpoint DLP)
3. Conditions:
   - Content contains: Credit card numbers (min 5 instances)
   - OR Content contains: SSN (min 1 instance)
4. Actions:
   - Block: Prevent copy to USB, cloud, email
   - Audit: Log but allow (for initial deployment)
   - Notify: Show user notification with policy tip
5. User notifications:
   - "This file contains sensitive data and cannot be copied to this location"
   - Allow override with business justification (optional)
```

### Step 3: Configure Endpoint DLP Activities

```
Monitored endpoint activities:
- Upload to cloud service (OneDrive, Dropbox, Google Drive)
- Copy to removable media (USB drives)
- Copy to network share
- Print document
- Copy to clipboard
- Access by unallowed browser (non-managed browser)
- Access by unallowed app
- Copy to Remote Desktop session

For each activity, configure:
- Audit only (log the action)
- Block with override (user can justify and proceed)
- Block (prevent action entirely)
```

### Step 4: Deploy in Audit Mode

```
Deploy DLP policy in "Test mode with notifications" first:
1. Policy runs in audit mode for 2-4 weeks
2. Review DLP alerts in Activity Explorer
3. Identify false positives
4. Tune SIT patterns and conditions
5. Add exclusions for legitimate workflows
6. Switch to "Turn on the policy" (enforcement)
```

### Step 5: Monitor and Respond

```
Purview → Data loss prevention → Activity explorer

Key metrics:
- DLP policy matches per day/week
- Top matched sensitive info types
- Top users triggering DLP
- Top activities blocked (USB, cloud, email)
- Override rate (percentage of blocks overridden)

DLP incident response:
1. Review DLP alert with matched content
2. Verify sensitivity of detected data
3. Assess intent (accidental vs. intentional)
4. If intentional exfiltration → escalate to security incident
5. If accidental → educate user, refine policy
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **DLP** | Data Loss Prevention; technology that detects and prevents unauthorized transmission of sensitive data |
| **SIT** | Sensitive Information Type; pattern matching rules for identifying sensitive data (regex, keywords, ML classifiers) |
| **Policy Tip** | User-facing notification explaining why an action was blocked and how to request an override |
| **Content Inspection** | Deep inspection of file contents to identify sensitive data patterns |
| **Exact Data Match (EDM)** | DLP matching against a specific database of known sensitive values (exact SSNs, employee records) |

## Tools & Systems

- **Microsoft Purview DLP**: Cloud-managed endpoint DLP included in M365 E5
- **Symantec DLP (Broadcom)**: Enterprise DLP with endpoint, network, and cloud modules
- **Digital Guardian**: Endpoint DLP with data classification and protection
- **Forcepoint DLP**: Unified DLP platform with endpoint agent
- **Code42 Incydr**: Insider risk detection with file exfiltration monitoring

## Common Pitfalls

- **Over-blocking in enforcement mode**: Deploy DLP in audit mode first. Blocking common workflows without warning causes productivity loss.
- **Too many SIT false positives**: Phone numbers, dates, and random number sequences can match PCI/SSN patterns. Tune confidence levels and require corroborating keywords.
- **Ignoring user education**: DLP is most effective when users understand why data is protected. Policy tips should explain the restriction and provide approved alternatives.
- **Not monitoring overrides**: If users frequently override DLP blocks, the policy is either too restrictive or users are ignoring data protection requirements. Review override reasons.

===== END FILE =====


===== FILE: skills/implementing-endpoint-dlp-controls/LICENSE =====

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


===== FILE: skills/implementing-endpoint-dlp-controls/assets/template.md =====
# Endpoint DLP Template

## Policy Summary
| Policy Name | SITs | Action | Scope |
|------------|------|--------|-------|
| | | Audit / Block | Devices |

## Monitoring Metrics
| Metric | Value |
|--------|-------|
| Policy matches (weekly) | |
| Override rate | % |
| Top triggered SIT | |
| False positive rate | % |

## Sign-Off
| Role | Name | Date |
|------|------|------|
| Data Protection Officer | | |
| Security | | |

===== END FILE =====


===== FILE: skills/implementing-endpoint-dlp-controls/references/api-reference.md =====
# API Reference: Implementing Endpoint DLP Controls

## Sensitive Data Patterns

| Pattern | Regex | Severity |
|---------|-------|----------|
| SSN | `\d{3}-\d{2}-\d{4}` | HIGH |
| Credit Card | `4[0-9]{12}(?:[0-9]{3})?` | HIGH |
| AWS Key | `AKIA[0-9A-Z]{16}` | CRITICAL |
| Private Key | `-----BEGIN.*PRIVATE KEY-----` | CRITICAL |
| API Key | `api[_-]?key\s*[:=]\s*[a-zA-Z0-9]{20,}` | HIGH |

## DLP Channels

| Channel | Monitoring Method |
|---------|-------------------|
| USB/Removable | Device event logs |
| Cloud Storage | URL/domain filtering |
| Email | Attachment scanning |
| Clipboard | Process monitoring |
| Print | Print spooler events |

## Microsoft Purview DLP API

```python
import requests
headers = {"Authorization": "Bearer <token>"}
resp = requests.get(
    "https://graph.microsoft.com/v1.0/security/alerts_v2",
    headers=headers,
    params={"$filter": "category eq 'DataLossPrevention'"})
```

## CrowdStrike Falcon DLP

```bash
curl -X GET "https://api.crowdstrike.com/dlp/entities/policies/v1" \
  -H "Authorization: Bearer $TOKEN"
```

## File Scanning

```python
from pathlib import Path
import re
SENSITIVE_EXTS = {".pem", ".key", ".env", ".kdbx", ".pfx"}
for f in Path("/data").rglob("*"):
    if f.suffix in SENSITIVE_EXTS or re.search(r"AKIA", f.read_text()):
        print(f"ALERT: {f}")
```

### References

- Microsoft Purview DLP: https://learn.microsoft.com/en-us/purview/dlp-learn-about-dlp
- CrowdStrike Falcon DLP: https://www.crowdstrike.com/platform/data-protection/

===== END FILE =====


===== FILE: skills/implementing-endpoint-dlp-controls/references/standards.md =====
# Standards & References
- **NIST SP 800-53 SC-7**: Boundary Protection - DLP enforces data boundaries
- **PCI DSS 4.0 Req 3**: Protect stored account data
- **GDPR Article 32**: Security of processing - preventing unauthorized data transfer
- **HIPAA 164.312(e)(1)**: Transmission security for ePHI
- **Microsoft Purview DLP**: https://learn.microsoft.com/en-us/purview/dlp-learn-about-dlp

===== END FILE =====


===== FILE: skills/implementing-endpoint-dlp-controls/references/workflows.md =====
# Workflows
## DLP Deployment
```
[Identify sensitive data types] → [Create SITs and policies]
  → [Deploy in audit mode] → [Review Activity Explorer for 2-4 weeks]
  → [Tune rules and exclusions] → [Enable enforcement]
  → [Monitor alerts and override rates] → [Quarterly policy review]
```

===== END FILE =====


===== FILE: skills/implementing-endpoint-dlp-controls/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing and monitoring endpoint DLP controls."""

import json
import argparse
import re
from datetime import datetime
from pathlib import Path


SENSITIVE_PATTERNS = {
    "SSN": r"\b\d{3}-\d{2}-\d{4}\b",
    "Credit Card": r"\b(?:4[0-9]{12}(?:[0-9]{3})?|5[1-5][0-9]{14}|3[47][0-9]{13})\b",
    "Email": r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b",
    "AWS Key": r"AKIA[0-9A-Z]{16}",
    "Private Key": r"-----BEGIN (RSA |EC )?PRIVATE KEY-----",
    "API Key": r"(?:api[_-]?key|apikey)\s*[:=]\s*['\"]?[a-zA-Z0-9]{20,}",
}

SENSITIVE_EXTENSIONS = [
    ".pem", ".key", ".pfx", ".p12", ".kdbx", ".env",
    ".sql", ".bak", ".dump", ".mdb",
]


def scan_file_for_sensitive_data(file_path):
    """Scan a single file for sensitive data patterns."""
    try:
        with open(file_path, "r", encoding="utf-8", errors="ignore") as f:
            content = f.read(1024 * 1024)
    except (OSError, PermissionError):
        return None
    matches = {}
    for pattern_name, pattern in SENSITIVE_PATTERNS.items():
        found = re.findall(pattern, content)
        if found:
            matches[pattern_name] = len(found)
    ext = Path(file_path).suffix.lower()
    is_sensitive_ext = ext in SENSITIVE_EXTENSIONS
    if not matches and not is_sensitive_ext:
        return None
    return {
        "file": str(file_path),
        "matches": matches,
        "sensitive_extension": is_sensitive_ext,
        "file_size": Path(file_path).stat().st_size,
        "severity": "CRITICAL" if "Private Key" in matches or "AWS Key" in matches
                    else "HIGH" if matches else "MEDIUM",
    }


def scan_directory(dir_path, max_files=10000):
    """Scan a directory for files containing sensitive data."""
    findings = []
    count = 0
    for filepath in Path(dir_path).rglob("*"):
        if filepath.is_file() and count < max_files:
            count += 1
            result = scan_file_for_sensitive_data(filepath)
            if result:
                findings.append(result)
    return sorted(findings, key=lambda x: x["severity"])


def monitor_usb_transfers(event_log_path):
    """Monitor file transfers to USB/removable devices."""
    findings = []
    with open(event_log_path) as f:
        for line in f:
            try:
                entry = json.loads(line)
            except json.JSONDecodeError:
                continue
            dest = entry.get("destination", entry.get("target_path", "")).lower()
            if any(ind in dest for ind in ["removable", "usb", "external"]):
                findings.append({
                    "timestamp": entry.get("timestamp", ""),
                    "user": entry.get("user", ""),
                    "file": entry.get("file_path", entry.get("source", "")),
                    "destination": dest,
                    "size_bytes": entry.get("size", entry.get("bytes", 0)),
                    "severity": "HIGH",
                    "channel": "USB",
                })
    return findings


def monitor_cloud_uploads(event_log_path):
    """Monitor file uploads to cloud storage services."""
    cloud_domains = ["drive.google.com", "dropbox.com", "onedrive.live.com",
                     "box.com", "wetransfer.com", "mega.nz"]
    findings = []
    with open(event_log_path) as f:
        for line in f:
            try:
                entry = json.loads(line)
            except json.JSONDecodeError:
                continue
            url = entry.get("url", entry.get("destination", "")).lower()
            if any(domain in url for domain in cloud_domains):
                findings.append({
                    "timestamp": entry.get("timestamp", ""),
                    "user": entry.get("user", ""),
                    "url": url[:200],
                    "file": entry.get("file_name", entry.get("filename", "")),
                    "severity": "HIGH",
                    "channel": "cloud_upload",
                })
    return findings


def generate_dlp_policy():
    """Generate endpoint DLP policy recommendations."""
    return {
        "data_classification": ["PII", "Financial", "Credentials", "Source Code"],
        "channels_monitored": ["USB", "Cloud Storage", "Email Attachments",
                               "Clipboard", "Print", "Screen Capture"],
        "actions": {
            "PII_to_USB": "block_and_notify",
            "credentials_to_cloud": "block_and_alert_soc",
            "source_code_to_email": "encrypt_and_log",
            "financial_to_print": "log_and_watermark",
        },
    }


def main():
    parser = argparse.ArgumentParser(description="Endpoint DLP Controls Agent")
    parser.add_argument("--scan-dir", help="Directory to scan for sensitive data")
    parser.add_argument("--usb-log", help="USB transfer event log")
    parser.add_argument("--cloud-log", help="Cloud upload event log")
    parser.add_argument("--output", default="endpoint_dlp_report.json")
    parser.add_argument("--action", choices=["scan", "usb", "cloud", "policy", "full"],
                        default="full")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "findings": {}}

    if args.action in ("scan", "full") and args.scan_dir:
        findings = scan_directory(args.scan_dir)
        report["findings"]["sensitive_data_scan"] = findings
        print(f"[+] Sensitive files found: {len(findings)}")

    if args.action in ("usb", "full") and args.usb_log:
        findings = monitor_usb_transfers(args.usb_log)
        report["findings"]["usb_transfers"] = findings
        print(f"[+] USB transfer events: {len(findings)}")

    if args.action in ("cloud", "full") and args.cloud_log:
        findings = monitor_cloud_uploads(args.cloud_log)
        report["findings"]["cloud_uploads"] = findings
        print(f"[+] Cloud upload events: {len(findings)}")

    if args.action in ("policy", "full"):
        policy = generate_dlp_policy()
        report["findings"]["dlp_policy"] = policy
        print("[+] DLP policy generated")

    with open(args.output, "w") as fout:
        json.dump(report, fout, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-endpoint-dlp-controls/scripts/process.py =====
#!/usr/bin/env python3
"""DLP Policy Analyzer - Analyzes DLP alert exports for policy tuning."""

import json, csv, sys, os
from collections import Counter
from datetime import datetime


def parse_dlp_alerts(csv_path: str) -> list:
    alerts = []
    with open(csv_path, "r", encoding="utf-8-sig") as f:
        for row in csv.DictReader(f):
            alerts.append({
                "timestamp": row.get("Date", ""),
                "user": row.get("User", ""),
                "activity": row.get("Activity", ""),
                "policy": row.get("Policy", ""),
                "sit": row.get("Sensitive Info Type", ""),
                "action": row.get("Action", ""),
                "location": row.get("Location", ""),
                "overridden": row.get("Override", "").lower() == "true",
            })
    return alerts


def analyze(alerts: list) -> dict:
    return {
        "total": len(alerts),
        "by_policy": dict(Counter(a["policy"] for a in alerts).most_common(20)),
        "by_user": dict(Counter(a["user"] for a in alerts).most_common(20)),
        "by_activity": dict(Counter(a["activity"] for a in alerts).most_common(10)),
        "by_sit": dict(Counter(a["sit"] for a in alerts).most_common(10)),
        "override_rate": round(sum(1 for a in alerts if a["overridden"]) / max(len(alerts), 1) * 100, 2),
        "blocked": sum(1 for a in alerts if "block" in a["action"].lower()),
    }


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <dlp_alerts.csv>")
        sys.exit(1)
    alerts = parse_dlp_alerts(sys.argv[1])
    result = analyze(alerts)
    out = os.path.join(os.path.dirname(sys.argv[1]) or ".", "dlp_analysis.json")
    with open(out, "w") as f:
        json.dump({"report_generated": datetime.utcnow().isoformat() + "Z", **result}, f, indent=2)
    print(f"Total: {result['total']} | Blocked: {result['blocked']} | Override rate: {result['override_rate']}%")

===== END FILE =====


===== FILE: skills/implementing-file-integrity-monitoring-with-aide/SKILL.md =====
---
name: implementing-file-integrity-monitoring-with-aide
description: Configures AIDE (Advanced Intrusion Detection Environment) for file integrity monitoring on Linux, covering baseline database creation, scheduled integrity checks via cron, change detection, and alerting on unauthorized modifications. Use when setting up host-based file integrity monitoring, detecting unauthorized file changes, or meeting compliance requirements for FIM on Linux systems.
domain: cybersecurity
subdomain: endpoint-security
tags:
- aide
- file-integrity
- hids
- baseline
- intrusion-detection
- compliance
- linux-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
---

# Implementing File Integrity Monitoring with AIDE

## Overview

AIDE (Advanced Intrusion Detection Environment) is a host-based intrusion detection system that monitors file and directory integrity using cryptographic checksums. This skill covers generating AIDE configuration files, initializing baseline databases, running integrity checks, parsing change reports, and setting up automated cron-based monitoring with alerting.


## When to Use

- When deploying or configuring implementing file integrity monitoring with aide capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- AIDE installed on target Linux system (apt install aide / yum install aide)
- Root or sudo access for file system scanning
- Python 3.8+ with standard library

## Steps

1. **Generate AIDE Configuration** — Create aide.conf with monitoring rules for critical directories (/etc, /bin, /sbin, /usr/bin, /boot)
2. **Initialize Baseline Database** — Run aide --init to create the initial file integrity baseline
3. **Run Integrity Check** — Execute aide --check to compare current state against baseline
4. **Parse Change Report** — Extract added, removed, and changed files from AIDE output
5. **Configure Automated Monitoring** — Generate cron job for scheduled integrity checks
6. **Generate Compliance Report** — Produce structured report of all file changes with severity classification

## Expected Output

- AIDE configuration file (aide.conf)
- Baseline database creation status
- JSON report of file changes (added/removed/changed) with severity
- Cron job configuration for automated monitoring

===== END FILE =====


===== FILE: skills/implementing-file-integrity-monitoring-with-aide/LICENSE =====

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


===== FILE: skills/implementing-file-integrity-monitoring-with-aide/references/api-reference.md =====
# AIDE File Integrity Monitoring API Reference

## AIDE CLI Commands

```bash
# Initialize baseline database
aide --init --config=/etc/aide/aide.conf

# Copy new database as baseline
cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Run integrity check against baseline
aide --check --config=/etc/aide/aide.conf

# Update baseline with current state
aide --update --config=/etc/aide/aide.conf

# Compare two databases
aide --compare --config=/etc/aide/aide.conf

# Validate configuration syntax
aide --config-check --config=/etc/aide/aide.conf
```

## Configuration File Syntax (aide.conf)

```
# Database locations
database_in=file:/var/lib/aide/aide.db
database_out=file:/var/lib/aide/aide.db.new

# Report output
report_url=stdout
report_url=file:/var/log/aide/aide.log

# Gzip compression
gzip_dbout=yes

# Rule definitions
# p=permissions i=inode n=links u=user g=group s=size
# b=block_count m=mtime c=ctime sha256=SHA-256 hash
NORMAL = p+i+n+u+g+s+b+m+c+sha256
PERMS = p+i+u+g
LOG = p+u+g+i

# Monitor paths
/bin NORMAL
/sbin NORMAL
/etc PERMS
/boot NORMAL

# Exclusions (prefix with !)
!/var/log/.*
!/proc
!/sys
!/tmp
```

## Available Check Attributes

| Attribute | Description |
|-----------|-------------|
| `p` | File permissions |
| `i` | Inode number |
| `n` | Number of hard links |
| `u` | User ownership |
| `g` | Group ownership |
| `s` | File size |
| `b` | Block count |
| `m` | Modification time |
| `c` | Change time (ctime) |
| `sha256` | SHA-256 hash |
| `sha512` | SHA-512 hash |
| `md5` | MD5 hash |
| `xattrs` | Extended attributes |
| `acl` | Access control lists |
| `selinux` | SELinux context |

## Cron Automation

```bash
# Daily integrity check at 5 AM
0 5 * * * root /usr/bin/aide --check --config=/etc/aide/aide.conf \
  | mail -s "AIDE Report" security@company.com

# Weekly baseline update
0 3 * * 0 root /usr/bin/aide --update --config=/etc/aide/aide.conf \
  && cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

## Installation

```bash
# Debian/Ubuntu
apt install aide

# RHEL/CentOS
yum install aide

# Initialize after install
aideinit       # Debian wrapper
aide --init    # Direct command
```

===== END FILE =====


===== FILE: skills/implementing-file-integrity-monitoring-with-aide/scripts/agent.py =====
#!/usr/bin/env python3
"""AIDE (Advanced Intrusion Detection Environment) configuration generator and check runner."""

import os
import json
import re
import subprocess
import argparse
from datetime import datetime

AIDE_RULE_SETS = {
    "critical_binaries": {
        "paths": ["/bin", "/sbin", "/usr/bin", "/usr/sbin"],
        "rule": "p+i+n+u+g+s+b+m+c+sha256+xattrs",
        "description": "Monitor all attributes and SHA-256 hash for system binaries",
    },
    "config_files": {
        "paths": ["/etc"],
        "rule": "p+i+u+g+sha256",
        "description": "Monitor permissions, inode, ownership, and hash for config files",
    },
    "boot_files": {
        "paths": ["/boot"],
        "rule": "p+i+n+u+g+s+b+m+c+sha256",
        "description": "Full attribute monitoring for boot files and kernel images",
    },
    "library_files": {
        "paths": ["/lib", "/lib64", "/usr/lib"],
        "rule": "p+i+n+u+g+s+b+sha256",
        "description": "Monitor shared libraries for unauthorized modifications",
    },
    "log_files": {
        "paths": ["/var/log"],
        "rule": "p+u+g+i",
        "description": "Monitor log file permissions and ownership (not content)",
    },
}

AIDE_EXCLUSIONS = [
    "!/var/log/.*", "!/var/spool/.*", "!/var/cache/.*",
    "!/proc", "!/sys", "!/dev", "!/run", "!/tmp",
    "!/var/lib/dpkg/.*", "!/var/lib/apt/.*",
]


def generate_aide_config(rule_sets=None, db_path="/var/lib/aide/aide.db",
                         db_new_path="/var/lib/aide/aide.db.new",
                         log_path="/var/log/aide/aide.log"):
    """Generate aide.conf configuration file content."""
    if rule_sets is None:
        rule_sets = list(AIDE_RULE_SETS.keys())

    lines = [
        "# AIDE Configuration - Generated by agent.py",
        f"# Generated: {datetime.utcnow().isoformat()}",
        "",
        f"database_in=file:{db_path}",
        f"database_out=file:{db_new_path}",
        f"database_new=file:{db_new_path}",
        "",
        "# Gzip output database",
        "gzip_dbout=yes",
        "",
        "# Report settings",
        "report_url=stdout",
        f"report_url=file:{log_path}",
        "",
        "# Rule definitions",
        "NORMAL = p+i+n+u+g+s+b+m+c+sha256",
        "PERMS = p+i+u+g",
        "LOG = p+u+g+i",
        "FULL = p+i+n+u+g+s+b+m+c+sha256+xattrs",
        "",
        "# Exclusions",
    ]

    for excl in AIDE_EXCLUSIONS:
        lines.append(excl)

    lines.append("")
    lines.append("# Monitored paths")

    for rs_name in rule_sets:
        rs = AIDE_RULE_SETS.get(rs_name)
        if not rs:
            continue
        lines.append(f"# {rs['description']}")
        for path in rs["paths"]:
            lines.append(f"{path} {rs['rule']}")
        lines.append("")

    return "\n".join(lines)


def initialize_baseline():
    """Run aide --init to create the baseline database."""
    result = subprocess.run(
        ["aide", "--init", "--config=/etc/aide/aide.conf"],
        capture_output=True, text=True, timeout=600
    )
    if result.returncode == 0:
        subprocess.run(
            ["cp", "/var/lib/aide/aide.db.new", "/var/lib/aide/aide.db"],
            capture_output=True, text=True,
            timeout=120,
        )
        return {"status": "success", "output": result.stdout[:500]}
    return {"status": "error", "stderr": result.stderr[:500]}


def run_integrity_check():
    """Run aide --check and parse the results."""
    result = subprocess.run(
        ["aide", "--check", "--config=/etc/aide/aide.conf"],
        capture_output=True, text=True, timeout=600
    )
    return parse_aide_output(result.stdout, result.returncode)


def parse_aide_output(output, return_code):
    """Parse AIDE check output into structured findings."""
    findings = {"added": [], "removed": [], "changed": [], "summary": {}}

    section = None
    for line in output.splitlines():
        line = line.strip()
        if "Added entries:" in line or "added:" in line.lower():
            section = "added"
        elif "Removed entries:" in line or "removed:" in line.lower():
            section = "removed"
        elif "Changed entries:" in line or "changed:" in line.lower():
            section = "changed"
        elif line.startswith("f") or line.startswith("d") or line.startswith("l"):
            match = re.match(r"^[fdl!]\s+(.+)", line)
            if match and section:
                filepath = match.group(1).strip()
                severity = _classify_severity(filepath, section)
                findings[section].append({
                    "path": filepath, "severity": severity, "change_type": section,
                })
        elif "Number of entries" in line:
            match = re.match(r"Number of entries:\s*(\d+)", line)
            if match:
                findings["summary"]["total_entries"] = int(match.group(1))

    findings["summary"]["added_count"] = len(findings["added"])
    findings["summary"]["removed_count"] = len(findings["removed"])
    findings["summary"]["changed_count"] = len(findings["changed"])
    findings["summary"]["return_code"] = return_code
    findings["summary"]["integrity_status"] = "clean" if return_code == 0 else "changes_detected"
    return findings


def _classify_severity(filepath, change_type):
    """Classify severity based on file path and change type."""
    critical_paths = ["/bin/", "/sbin/", "/usr/bin/", "/usr/sbin/", "/boot/", "/etc/shadow",
                      "/etc/passwd", "/etc/sudoers", "/etc/ssh/"]
    high_paths = ["/etc/", "/lib/", "/lib64/", "/usr/lib/"]

    if change_type == "removed" and any(filepath.startswith(p) for p in critical_paths):
        return "critical"
    if any(filepath.startswith(p) for p in critical_paths):
        return "critical" if change_type != "added" else "high"
    if any(filepath.startswith(p) for p in high_paths):
        return "high"
    return "medium"


def generate_cron_config(schedule="0 5 * * *", email=None):
    """Generate cron job for automated AIDE integrity checks."""
    cron_script = "#!/bin/bash\n"
    cron_script += "# AIDE automated integrity check\n"
    cron_script += "LOGFILE=/var/log/aide/aide-check-$(date +%Y%m%d).log\n"
    cron_script += "aide --check --config=/etc/aide/aide.conf > $LOGFILE 2>&1\n"
    cron_script += "EXIT_CODE=$?\n"
    cron_script += "if [ $EXIT_CODE -ne 0 ]; then\n"
    if email:
        cron_script += f'  mail -s "AIDE Alert: Integrity changes detected" {email} < $LOGFILE\n'
    cron_script += '  logger -t aide "Integrity check found changes (exit code $EXIT_CODE)"\n'
    cron_script += "fi\n"

    cron_entry = f"{schedule} root /usr/local/bin/aide-check.sh"
    return {"script": cron_script, "cron_entry": cron_entry, "schedule": schedule}


def generate_report(config_content, baseline_status, check_findings, cron_config):
    """Generate AIDE deployment and check report."""
    return {
        "report_time": datetime.utcnow().isoformat(),
        "config_lines": len(config_content.splitlines()),
        "monitored_rule_sets": list(AIDE_RULE_SETS.keys()),
        "exclusion_count": len(AIDE_EXCLUSIONS),
        "baseline_status": baseline_status,
        "integrity_check": check_findings,
        "cron_schedule": cron_config,
    }


def main():
    parser = argparse.ArgumentParser(description="AIDE File Integrity Monitoring Agent")
    parser.add_argument("--action", choices=["generate", "init", "check", "full"],
                        default="generate", help="Action to perform")
    parser.add_argument("--config-output", default="/etc/aide/aide.conf", help="Config output path")
    parser.add_argument("--alert-email", help="Email for cron alert notifications")
    parser.add_argument("--output", default="aide_report.json")
    parser.add_argument("--rule-sets", nargs="+", default=None,
                        choices=list(AIDE_RULE_SETS.keys()), help="Rule sets to enable")
    args = parser.parse_args()

    config_content = generate_aide_config(args.rule_sets)
    baseline_status = {}
    check_findings = {}

    if args.action in ("generate", "full"):
        os.makedirs(os.path.dirname(args.config_output), exist_ok=True)
        with open(args.config_output, "w") as f:
            f.write(config_content)
        print(f"[+] AIDE config written to {args.config_output}")

    if args.action in ("init", "full"):
        baseline_status = initialize_baseline()
        print(f"[+] Baseline init: {baseline_status['status']}")

    if args.action in ("check", "full"):
        check_findings = run_integrity_check()
        s = check_findings.get("summary", {})
        print(f"[+] Check: added={s.get('added_count', 0)} removed={s.get('removed_count', 0)} "
              f"changed={s.get('changed_count', 0)}")

    cron_config = generate_cron_config(email=args.alert_email)
    report = generate_report(config_content, baseline_status, check_findings, cron_config)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-memory-protection-with-dep-aslr/SKILL.md =====
---
name: implementing-memory-protection-with-dep-aslr
description: 'Implements memory protection mechanisms including DEP (Data Execution
  Prevention), ASLR (Address Space Layout Randomization), CFG (Control Flow Guard),
  and other exploit mitigations to prevent memory corruption attacks. Use when hardening
  endpoints against buffer overflow exploits, ROP chains, and code injection. Activates
  for requests involving memory protection, exploit mitigation, DEP, ASLR, or CFG
  configuration.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- memory-protection
- DEP
- ASLR
- exploit-mitigation
- CFG
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
- T1190
---
# Implementing Memory Protection with DEP and ASLR

## When to Use

Use this skill when hardening endpoints against memory-based exploits by configuring DEP, ASLR, CFG, and Windows Exploit Protection system-wide and per-application mitigations.

## Prerequisites

- Windows 10/11 or Windows Server 2016+ with administrative privileges
- Group Policy management access for enterprise-wide deployment
- Understanding of memory corruption attack techniques (buffer overflow, ROP chains)
- Test environment for validating application compatibility with exploit mitigations

## Workflow

### Step 1: Configure System-Level Mitigations

```powershell
# Enable system-wide DEP (Data Execution Prevention)
# Boot configuration: OptIn (default), OptOut (recommended), AlwaysOn
bcdedit /set nx AlwaysOn

# Verify ASLR status (enabled by default on modern Windows)
Get-ProcessMitigation -System
# MandatoryASLR, BottomUpASLR, HighEntropyASLR should be ON

# Enable all system-level mitigations
Set-ProcessMitigation -System -Enable DEP,SEHOP,ForceRelocateImages,BottomUp,HighEntropy
```

### Step 2: Configure Per-Application Mitigations

```powershell
# Harden high-risk applications (browsers, Office, PDF readers)
Set-ProcessMitigation -Name "WINWORD.EXE" -Enable DEP,SEHOP,ForceRelocateImages,CFG,StrictHandle
Set-ProcessMitigation -Name "EXCEL.EXE" -Enable DEP,SEHOP,ForceRelocateImages,CFG,StrictHandle
Set-ProcessMitigation -Name "AcroRd32.exe" -Enable DEP,SEHOP,ForceRelocateImages,CFG
Set-ProcessMitigation -Name "chrome.exe" -Enable DEP,CFG,ForceRelocateImages
Set-ProcessMitigation -Name "msedge.exe" -Enable DEP,CFG,ForceRelocateImages

# Export configuration for deployment
Get-ProcessMitigation -RegistryConfigFilePath "C:\exploit_protection.xml"
# Deploy via Intune or GPO
```

### Step 3: Deploy via Intune/GPO

```
Intune: Endpoint Security → Attack Surface Reduction → Exploit Protection
  Import exploit_protection.xml template

GPO: Computer Configuration → Admin Templates → Windows Components
  → Windows Defender Exploit Guard → Exploit Protection
  → "Use a common set of exploit protection settings" → Enabled
  → Point to XML file on network share
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **DEP** | Marks memory pages as non-executable to prevent shellcode execution in data regions |
| **ASLR** | Randomizes memory addresses of loaded modules to defeat hardcoded ROP gadgets |
| **CFG** | Validates indirect call targets at runtime to prevent control flow hijacking |
| **SEHOP** | Validates SEH chain integrity to prevent SEH-based exploitation |

## Tools & Systems
- **Windows Exploit Protection**: Built-in per-process mitigation management
- **EMET (legacy)**: Enhanced Mitigation Experience Toolkit (predecessor, now deprecated)
- **ProcessMitigations PowerShell**: Get/Set-ProcessMitigation cmdlets

## Common Pitfalls
- **DEP compatibility**: Legacy 32-bit applications may crash with DEP AlwaysOn. Use OptOut with exceptions.
- **Mandatory ASLR breaking apps**: Some applications are not ASLR-compatible. Test before enforcing ForceRelocateImages.
- **CFG limited to compiled-in support**: CFG only works for applications compiled with /guard:cf. Cannot be retroactively applied.

===== END FILE =====


===== FILE: skills/implementing-memory-protection-with-dep-aslr/LICENSE =====

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


===== FILE: skills/implementing-memory-protection-with-dep-aslr/assets/template.md =====
# Memory Protection Configuration Template

## System-Level Mitigations
| Mitigation | Status | Notes |
|-----------|--------|-------|
| DEP | AlwaysOn / OptOut | |
| ASLR (BottomUp) | Enabled | |
| ASLR (HighEntropy) | Enabled | |
| SEHOP | Enabled | |
| ForceRelocateImages | Enabled | |

## Per-Application Mitigations
| Application | DEP | ASLR | CFG | SEHOP |
|------------|-----|------|-----|-------|
| WINWORD.EXE | On | On | On | On |
| EXCEL.EXE | On | On | On | On |

## Sign-Off
| Role | Name | Date |
|------|------|------|
| Security | | |

===== END FILE =====


===== FILE: skills/implementing-memory-protection-with-dep-aslr/references/api-reference.md =====
# API Reference: Implementing Memory Protection with DEP and ASLR

## Windows PowerShell Commands

```powershell
# Check system-wide mitigations
Get-ProcessMitigation -System
# Check specific process
Get-ProcessMitigation -Name chrome.exe
# Set system-wide DEP
Set-ProcessMitigation -System -Enable DEP
# Import XML policy
Set-ProcessMitigation -PolicyFilePath policy.xml
# Export current policy
Get-ProcessMitigation -RegistryConfigFilePath export.xml
```

## Memory Protection Mechanisms

| Mechanism | OS | Description |
|-----------|-----|------------|
| DEP/NX | Windows/Linux | Prevent code execution from data pages |
| ASLR | Windows/Linux | Randomize memory layout |
| CFG | Windows | Control Flow Guard |
| SEHOP | Windows | SEH Overwrite Protection |
| Stack Canary | Linux | Detect stack buffer overflow |
| PIE | Linux | Position-Independent Executable |
| RELRO | Linux | Read-Only Relocations |
| FORTIFY_SOURCE | Linux | Buffer overflow checks |

## Linux ASLR Check

```bash
# Check ASLR level (0=off, 1=conservative, 2=full)
cat /proc/sys/kernel/randomize_va_space
# Enable full ASLR
echo 2 > /proc/sys/kernel/randomize_va_space
```

## ELF Binary Check (checksec)

```bash
checksec --file=/usr/bin/target
# Or with readelf
readelf -l binary | grep GNU_STACK
readelf -d binary | grep BIND_NOW
```

## GCC Compilation Flags

| Flag | Protection |
|------|-----------|
| `-fstack-protector-strong` | Stack canary |
| `-D_FORTIFY_SOURCE=2` | Buffer overflow checks |
| `-pie -fPIE` | Position-independent |
| `-Wl,-z,relro,-z,now` | Full RELRO |
| `-Wl,-z,noexecstack` | NX stack |

### References

- Windows Exploit Protection: https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/exploit-protection
- checksec: https://github.com/slimm609/checksec.sh
- ASLR: https://en.wikipedia.org/wiki/Address_space_layout_randomization

===== END FILE =====


===== FILE: skills/implementing-memory-protection-with-dep-aslr/references/standards.md =====
# Standards & References
- **MITRE ATT&CK T1055**: Process Injection - mitigated by DEP, CFG, ASLR
- **CIS Benchmark Section 18.9**: Exploit Protection configuration
- **NIST 800-53 SI-16**: Memory Protection
- **Microsoft Exploit Protection**: https://learn.microsoft.com/en-us/defender-endpoint/exploit-protection

===== END FILE =====


===== FILE: skills/implementing-memory-protection-with-dep-aslr/references/workflows.md =====
# Workflows
## Memory Protection Deployment
```
[Audit current mitigations: Get-ProcessMitigation -System]
  → [Enable system-level DEP, SEHOP, ASLR]
  → [Configure per-app mitigations for high-risk applications]
  → [Export XML, deploy via Intune/GPO]
  → [Test application compatibility] → [Monitor for crashes]
  → [Tune exceptions for incompatible apps]
```

===== END FILE =====


===== FILE: skills/implementing-memory-protection-with-dep-aslr/scripts/agent.py =====
#!/usr/bin/env python3
"""DEP/ASLR Memory Protection Agent - audits processes and binaries for memory protection mitigations."""

import json
import argparse
import logging
import subprocess
import os
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def check_windows_dep_policy():
    """Check Windows DEP/NX policy via bcdedit."""
    cmd = ["bcdedit", "/enum", "{current}"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    dep_policy = "unknown"
    for line in result.stdout.split("\n"):
        if "nx" in line.lower():
            dep_policy = line.split()[-1] if line.split() else "unknown"
    return {"dep_policy": dep_policy, "recommended": "AlwaysOn"}


def check_windows_process_mitigations():
    """Check process mitigation policies using PowerShell Get-ProcessMitigation."""
    cmd = ["powershell", "-Command",
           "Get-Process | ForEach-Object { try { $m = Get-ProcessMitigation -Id $_.Id -ErrorAction Stop; "
           "[PSCustomObject]@{Name=$_.Name;PID=$_.Id;DEP=$m.DEP.Enable;ASLR=$m.ASLR.ForceRelocateImages;"
           "CFG=$m.CFG.Enable;StrictHandle=$m.StrictHandle.Enable} } catch {} } | ConvertTo-Json"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    try:
        processes = json.loads(result.stdout) if result.stdout else []
        if isinstance(processes, dict):
            processes = [processes]
        return processes
    except json.JSONDecodeError:
        return []


def check_linux_aslr():
    """Check Linux ASLR configuration."""
    try:
        with open("/proc/sys/kernel/randomize_va_space") as f:
            value = int(f.read().strip())
        levels = {0: "disabled", 1: "partial", 2: "full"}
        return {"aslr_level": value, "status": levels.get(value, "unknown"), "recommended": 2}
    except (FileNotFoundError, ValueError):
        return {"aslr_level": -1, "status": "unknown"}


def check_linux_nx_support():
    """Check NX bit support on Linux."""
    result = subprocess.run(["grep", "nx", "/proc/cpuinfo"], capture_output=True, text=True, timeout=120)
    return {"nx_supported": "nx" in result.stdout.lower()}


def check_elf_pie_relro(binary_path):
    """Check ELF binary for PIE, RELRO, stack canary, and NX using readelf/checksec."""
    findings = {"binary": binary_path, "pie": False, "relro": "none", "nx": False, "canary": False}
    readelf_cmd = subprocess.run(["readelf", "-h", binary_path], capture_output=True, text=True, timeout=120)
    if "DYN" in readelf_cmd.stdout:
        findings["pie"] = True
    readelf_d = subprocess.run(["readelf", "-d", binary_path], capture_output=True, text=True, timeout=120)
    if "BIND_NOW" in readelf_d.stdout:
        findings["relro"] = "full"
    elif "GNU_RELRO" in readelf_d.stdout:
        findings["relro"] = "partial"
    readelf_s = subprocess.run(["readelf", "-s", binary_path], capture_output=True, text=True, timeout=120)
    if "__stack_chk_fail" in readelf_s.stdout:
        findings["canary"] = True
    readelf_l = subprocess.run(["readelf", "-l", binary_path], capture_output=True, text=True, timeout=120)
    for line in readelf_l.stdout.split("\n"):
        if "GNU_STACK" in line and "RWE" not in line:
            findings["nx"] = True
    score = sum([findings["pie"], findings["relro"] == "full", findings["nx"], findings["canary"]])
    findings["protection_score"] = f"{score}/4"
    return findings


def scan_directory_binaries(directory, extensions=None):
    """Scan directory for binaries and check protections."""
    if extensions is None:
        extensions = [""]
    results = []
    for root, dirs, files in os.walk(directory):
        for fname in files:
            fpath = os.path.join(root, fname)
            if os.path.isfile(fpath) and os.access(fpath, os.X_OK):
                try:
                    result = check_elf_pie_relro(fpath)
                    results.append(result)
                except Exception:
                    continue
        if len(results) >= 100:
            break
    return results


def generate_report(system_checks, binary_checks):
    weak_binaries = [b for b in binary_checks if not b.get("pie") or b.get("relro") == "none"]
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "system_protections": system_checks,
        "binaries_scanned": len(binary_checks),
        "weak_binaries": len(weak_binaries),
        "protection_summary": {
            "pie_enabled": sum(1 for b in binary_checks if b.get("pie")),
            "full_relro": sum(1 for b in binary_checks if b.get("relro") == "full"),
            "nx_enabled": sum(1 for b in binary_checks if b.get("nx")),
            "canary_present": sum(1 for b in binary_checks if b.get("canary")),
        },
        "weak_binary_details": weak_binaries[:20],
    }
    return report


def main():
    parser = argparse.ArgumentParser(description="DEP/ASLR Memory Protection Audit Agent")
    parser.add_argument("--scan-dir", default="/usr/bin", help="Directory to scan binaries")
    parser.add_argument("--platform", choices=["linux", "windows", "auto"], default="auto")
    parser.add_argument("--output", default="memory_protection_report.json")
    args = parser.parse_args()

    platform = args.platform
    if platform == "auto":
        platform = "windows" if os.name == "nt" else "linux"

    system_checks = {}
    if platform == "linux":
        system_checks["aslr"] = check_linux_aslr()
        system_checks["nx"] = check_linux_nx_support()
        binary_checks = scan_directory_binaries(args.scan_dir)
    else:
        system_checks["dep"] = check_windows_dep_policy()
        process_mitigations = check_windows_process_mitigations()
        binary_checks = []
        for proc in process_mitigations:
            binary_checks.append({
                "binary": proc.get("Name", ""),
                "pid": proc.get("PID", 0),
                "pie": bool(proc.get("ASLR")),
                "nx": bool(proc.get("DEP")),
                "canary": False,
                "relro": "n/a",
            })

    report = generate_report(system_checks, binary_checks)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("Scanned %d binaries, %d with weak protections",
                report["binaries_scanned"], report["weak_binaries"])
    print(json.dumps(report, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-memory-protection-with-dep-aslr/scripts/process.py =====
#!/usr/bin/env python3
"""Memory Protection Auditor - Checks exploit mitigation status on Windows."""

import json, subprocess, sys, os
from datetime import datetime


def check_mitigations() -> dict:
    ps_cmd = """
    $sys = Get-ProcessMitigation -System
    $apps = Get-ProcessMitigation -Name * 2>$null | Select-Object -First 20
    @{System = $sys; Apps = $apps} | ConvertTo-Json -Depth 3
    """
    try:
        r = subprocess.run(["powershell", "-NoProfile", "-Command", ps_cmd],
                          capture_output=True, text=True, timeout=30)
        return json.loads(r.stdout) if r.returncode == 0 else {"error": r.stderr}
    except Exception as e:
        return {"error": str(e)}


if __name__ == "__main__":
    result = check_mitigations()
    if "error" in result:
        print(f"Error: {result['error']}")
        print("This tool requires Windows with Exploit Protection support.")
        sys.exit(1)
    out = sys.argv[1] if len(sys.argv) > 1 else "memory_protection_audit.json"
    with open(out, "w") as f:
        json.dump({"generated": datetime.utcnow().isoformat() + "Z", **result}, f, indent=2)
    print(f"Report: {out}")

===== END FILE =====


===== FILE: skills/implementing-usb-device-control-policy/SKILL.md =====
---
name: implementing-usb-device-control-policy
description: 'Implements USB device control policies to restrict unauthorized removable
  media access on endpoints, preventing data exfiltration and malware introduction
  via USB devices. Use when deploying device control via Group Policy, Intune, or
  EDR platforms to enforce USB restrictions. Activates for requests involving USB
  control, removable media policy, device control, or data loss prevention via USB.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- USB-control
- device-control
- data-loss-prevention
- removable-media
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
- T1048
---
# Implementing USB Device Control Policy

## When to Use

Use this skill when:
- Restricting USB storage devices to prevent data exfiltration or malware introduction
- Implementing device control policies via GPO, Intune, or EDR device control modules
- Creating USB whitelists for authorized devices while blocking all others
- Meeting compliance requirements for removable media control (PCI DSS, HIPAA)

**Do not use** for network-based DLP or cloud storage restrictions.

## Prerequisites

- Active Directory GPO or Microsoft Intune for policy deployment
- Device Instance IDs of authorized USB devices
- EDR with device control module (CrowdStrike, Microsoft Defender for Endpoint)
- Understanding of USB device classes (mass storage, HID, printer, etc.)

## Workflow

### Step 1: Inventory Current USB Usage

```powershell
# Enumerate currently connected USB devices
Get-PnpDevice -Class USB | Select-Object InstanceId, FriendlyName, Status

# Query USB storage history from registry
Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Enum\USBSTOR\*\*" |
  Select-Object FriendlyName, ContainerID, HardwareID

# Collect USB usage across fleet (via EDR or scripts)
# CrowdStrike: Investigate → USB Device Activity
# MDE: DeviceEvents | where ActionType == "UsbDriveMounted"
```

### Step 2: Configure GPO Device Control

```
Computer Configuration → Administrative Templates → System → Removable Storage Access

- All Removable Storage classes: Deny all access → Enabled
  (Block read AND write for all removable storage)

OR for granular control:
- CD and DVD: Deny read access → Enabled
- Removable Disks: Deny write access → Enabled (read-only USB)
- Tape Drives: Deny all access → Enabled
- WPD Devices: Deny all access → Enabled

To allow specific approved USB devices:
Computer Configuration → Administrative Templates → System → Device Installation
  → Device Installation Restrictions

- Prevent installation of devices not described by other policy settings → Enabled
- Allow installation of devices that match any of these device IDs → Enabled
  Add approved Device IDs: USB\VID_0781&PID_5583 (example: SanDisk Cruzer)
```

### Step 3: Deploy via Microsoft Defender for Endpoint

```xml
<!-- MDE Device Control policy (XML format) -->
<PolicyGroups>
  <Group Id="{d9a81dc0-1234-5678-9abc-def012345678}"
    Type="Device" Name="Approved USB Devices">
    <MatchClause>
      <MatchType>VID_PID</MatchType>
      <MatchData>0781_5583</MatchData> <!-- SanDisk -->
    </MatchClause>
  </Group>
</PolicyGroups>

<PolicyRules>
  <Rule Id="{rule-guid}" Name="Block unapproved USB storage">
    <IncludedIdList>
      <PrimaryId>RemovableMediaDevices</PrimaryId>
    </IncludedIdList>
    <ExcludedIdList>
      <GroupId>{d9a81dc0-1234-5678-9abc-def012345678}</GroupId>
    </ExcludedIdList>
    <Entry>
      <Type>Deny</Type>
      <AccessMask>63</AccessMask> <!-- All access -->
      <Options>4</Options> <!-- Show notification -->
    </Entry>
  </Rule>
</PolicyRules>
```

### Step 4: Audit and Monitor

```
# Monitor USB events in SIEM:
# Windows Event ID 6416 - New external device recognized
# Windows Event ID 4663 - File access on removable media
# MDE: DeviceEvents where ActionType contains "Usb"

# Generate USB activity reports monthly
# Track: blocked attempts, approved device usage, exception requests
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **VID/PID** | Vendor ID and Product ID that uniquely identify USB device models |
| **Device Instance ID** | Unique identifier for a specific physical USB device |
| **Device Control** | EDR/endpoint feature restricting device access based on type, vendor, or serial number |
| **USB Class** | USB device category (mass storage 08h, HID 03h, printer 07h) |

## Tools & Systems

- **Microsoft Defender Device Control**: MDE module for USB restriction policies
- **CrowdStrike Falcon Device Control**: EDR-based USB policy enforcement
- **Group Policy (Removable Storage Access)**: Built-in Windows USB restriction via GPO
- **Endpoint Protector**: Third-party device control and DLP solution

## Common Pitfalls

- **Blocking all USB without exception**: Keyboards and mice are USB HID devices. Block only mass storage class, not all USB.
- **Not communicating policy to users**: USB blocks without user notification generate helpdesk tickets. Display a notification explaining the policy.
- **Ignoring USB-C and Thunderbolt**: Modern devices use USB-C for docking, charging, and storage. Policies must distinguish between USB storage and USB peripherals.
- **No approved device process**: Users with legitimate USB needs (presentations, field data collection) require an exception process with approved, encrypted devices.

===== END FILE =====


===== FILE: skills/implementing-usb-device-control-policy/LICENSE =====

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


===== FILE: skills/implementing-usb-device-control-policy/assets/template.md =====
# USB Device Control Policy Template

## Policy Configuration
| Setting | Value |
|---------|-------|
| Default Action | Block all removable storage |
| Enforcement Mode | Audit / Enforce |
| Notification | Display user message |

## Approved Devices
| Device Name | VID:PID | Serial | Issued To | Expiry |
|-------------|---------|--------|-----------|--------|
| | | | | |

## Exception Register
| User | Business Justification | Device | Approved By | Expiry |
|------|----------------------|--------|-------------|--------|
| | | | | |

## Sign-Off
| Role | Name | Date |
|------|------|------|
| Security | | |
| IT Ops | | |

===== END FILE =====


===== FILE: skills/implementing-usb-device-control-policy/references/api-reference.md =====
# API Reference: USB Device Control Policy Audit

## Libraries Used

| Library | Purpose |
|---------|---------|
| `subprocess` | Execute PowerShell, udevadm, and registry query commands |
| `json` | Parse device inventory and policy status |
| `platform` | Detect operating system for platform-specific checks |
| `re` | Parse device IDs and USB vendor/product codes |

## Installation

```bash
# No external packages — uses standard library and OS tools
```

## Windows USB Device Audit

### List Connected USB Devices (PowerShell)
```python
import subprocess
import json

def list_usb_devices_windows():
    cmd = [
        "powershell", "-Command",
        "Get-PnpDevice -Class USB | Select-Object Status, Class, FriendlyName, InstanceId | ConvertTo-Json"
    ]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
    return json.loads(result.stdout) if result.stdout else []
```

### Check USB Storage Policy (Registry)
```python
def check_usb_storage_policy():
    """Check if USB mass storage is disabled via registry."""
    cmd = [
        "powershell", "-Command",
        'Get-ItemProperty -Path "HKLM:\\SYSTEM\\CurrentControlSet\\Services\\USBSTOR" -Name Start | Select-Object Start | ConvertTo-Json'
    ]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=15)
    if result.stdout:
        data = json.loads(result.stdout)
        start_value = data.get("Start", 3)
        return {
            "usb_storage_disabled": start_value == 4,
            "registry_value": start_value,
            "policy": "disabled" if start_value == 4 else "enabled",
            "detail": {
                3: "USB storage ENABLED (default)",
                4: "USB storage DISABLED",
            }.get(start_value, f"Unknown value: {start_value}"),
        }
    return {"usb_storage_disabled": False, "error": "Could not read registry"}
```

### Check Group Policy for Removable Storage
```python
def check_gpo_removable_storage():
    """Check GPO settings for removable storage restrictions."""
    policies = {
        "deny_read": r"HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}\Deny_Read",
        "deny_write": r"HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}\Deny_Write",
        "deny_execute": r"HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}\Deny_Execute",
    }
    results = {}
    for name, path in policies.items():
        cmd = ["reg", "query", path.rsplit("\\", 1)[0], "/v", path.rsplit("\\", 1)[1]]
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
        results[name] = "1" in result.stdout if result.returncode == 0 else False
    return results
```

### USB Device History (Windows)
```python
def get_usb_history_windows():
    """List previously connected USB storage devices from registry."""
    cmd = [
        "powershell", "-Command",
        'Get-ItemProperty "HKLM:\\SYSTEM\\CurrentControlSet\\Enum\\USBSTOR\\*\\*" | Select-Object FriendlyName, DeviceDesc, Mfg | ConvertTo-Json'
    ]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
    return json.loads(result.stdout) if result.stdout else []
```

## Linux USB Device Audit

### List USB Devices
```python
def list_usb_devices_linux():
    result = subprocess.run(
        ["lsusb"], capture_output=True, text=True, timeout=10
    )
    devices = []
    for line in result.stdout.strip().split("\n"):
        if line:
            devices.append(line.strip())
    return devices
```

### Check USBGuard Policy
```python
def check_usbguard_status():
    """Check if USBGuard is installed and active."""
    # Check service status
    result = subprocess.run(
        ["systemctl", "is-active", "usbguard"],
        capture_output=True, text=True, timeout=10,
    )
    service_active = result.stdout.strip() == "active"

    # List current policy rules
    rules = []
    if service_active:
        result = subprocess.run(
            ["usbguard", "list-rules"],
            capture_output=True, text=True, timeout=10,
        )
        rules = result.stdout.strip().split("\n") if result.stdout else []

    return {
        "usbguard_installed": service_active or result.returncode != 127,
        "service_active": service_active,
        "policy_rules": len(rules),
        "default_policy": "block" if any("block" in r for r in rules) else "allow",
    }
```

### Check udev Rules for USB Control
```python
def check_udev_rules():
    """Check for USB control udev rules."""
    result = subprocess.run(
        ["find", "/etc/udev/rules.d/", "-name", "*usb*", "-type", "f"],
        capture_output=True, text=True, timeout=10,
    )
    rules_files = result.stdout.strip().split("\n") if result.stdout.strip() else []
    return {"udev_usb_rules": rules_files, "count": len(rules_files)}
```

## Device Whitelist Management

```python
APPROVED_DEVICES = [
    {"vendor_id": "046d", "product_id": "c52b", "name": "Logitech Receiver"},
    {"vendor_id": "0781", "product_id": "5583", "name": "SanDisk Encrypted Drive"},
]

def check_against_whitelist(connected_devices, approved=APPROVED_DEVICES):
    approved_ids = {(d["vendor_id"], d["product_id"]) for d in approved}
    findings = []
    for device in connected_devices:
        vid = device.get("vendor_id", "")
        pid = device.get("product_id", "")
        if (vid, pid) not in approved_ids:
            findings.append({
                "device": device.get("name", "Unknown"),
                "vendor_id": vid,
                "product_id": pid,
                "issue": "Device not in approved whitelist",
                "severity": "medium",
            })
    return findings
```

## Output Format

```json
{
  "platform": "windows",
  "usb_storage_disabled": true,
  "gpo_deny_read": true,
  "gpo_deny_write": true,
  "connected_devices": 3,
  "unapproved_devices": 1,
  "historical_devices": 12,
  "findings": [
    {
      "device": "Unknown USB Mass Storage",
      "vendor_id": "0951",
      "product_id": "1666",
      "issue": "Device not in approved whitelist",
      "severity": "medium"
    }
  ]
}
```

===== END FILE =====


===== FILE: skills/implementing-usb-device-control-policy/references/standards.md =====
# Standards & References

## Primary Standards
- **NIST SP 800-53 MP-7**: Media Use - restricting removable media types and usage
- **PCI DSS 4.0 Req 3.4.2**: Restrict removable electronic media for cardholder data
- **CIS Control 10.4**: Configure device control to prevent removable media autorun
- **ISO 27001 A.8.3.1**: Management of removable media

## Supporting References
- **MDE Device Control**: https://learn.microsoft.com/en-us/defender-endpoint/device-control-overview
- **CrowdStrike Device Control**: Falcon Console documentation
- **GPO Removable Storage**: https://learn.microsoft.com/en-us/windows/client-management/mdm/policy-csp-remotemanagement

===== END FILE =====


===== FILE: skills/implementing-usb-device-control-policy/references/workflows.md =====
# Workflows

## Workflow 1: USB Device Control Deployment
```
[Audit current USB usage across fleet] → [Identify legitimate USB needs]
  → [Create approved device whitelist] → [Configure block policy with exceptions]
  → [Deploy in audit mode for 2 weeks] → [Review blocked events]
  → [Add missing legitimate devices] → [Switch to enforce mode]
  → [Communicate policy to users] → [Monitor and maintain]
```

## Workflow 2: USB Exception Request
```
[User requests USB access] → [Verify business justification]
  → [Issue approved encrypted USB device] → [Add device ID to whitelist]
  → [Deploy updated policy] → [Log exception with expiry date]
```

===== END FILE =====


===== FILE: skills/implementing-usb-device-control-policy/scripts/agent.py =====
#!/usr/bin/env python3
"""USB device control policy audit agent.

Audits USB device control policies on Linux and Windows systems by
checking udev rules, USBGuard configuration, Windows Group Policy
settings, and connected device history. Reports unauthorized or
unwhitelisted USB devices.
"""
import argparse
import json
import os
import subprocess
import sys
from datetime import datetime, timezone


def audit_linux_usbguard():
    """Audit USBGuard configuration and rules on Linux."""
    findings = []
    print("[*] Auditing USBGuard configuration...")

    # Check if USBGuard is installed and running
    result = subprocess.run(
        ["systemctl", "is-active", "usbguard"],
        capture_output=True, text=True, timeout=10,
    )
    if result.stdout.strip() == "active":
        findings.append({"check": "USBGuard service", "status": "PASS",
                         "severity": "INFO", "detail": "USBGuard is running"})
    else:
        findings.append({"check": "USBGuard service", "status": "FAIL",
                         "severity": "HIGH", "detail": "USBGuard is not running"})
        return findings

    # List current USB devices and their authorization
    result = subprocess.run(
        ["usbguard", "list-devices"],
        capture_output=True, text=True, timeout=15,
    )
    if result.returncode == 0:
        devices = []
        for line in result.stdout.strip().splitlines():
            parts = line.split()
            if len(parts) >= 3:
                dev_id = parts[0].rstrip(":")
                policy = parts[1]
                name = " ".join(parts[2:])
                devices.append({"id": dev_id, "policy": policy, "name": name})
                if policy == "allow":
                    findings.append({"check": f"USB Device {dev_id}", "status": "INFO",
                                    "severity": "INFO", "detail": f"Allowed: {name}"})
                elif policy == "block":
                    findings.append({"check": f"USB Device {dev_id}", "status": "BLOCKED",
                                    "severity": "INFO", "detail": f"Blocked: {name}"})

    # Check default policy
    result = subprocess.run(
        ["usbguard", "get-parameter", "ImplicitPolicyTarget"],
        capture_output=True, text=True, timeout=10,
    )
    if result.returncode == 0:
        policy = result.stdout.strip()
        if policy == "block":
            findings.append({"check": "Default USB policy", "status": "PASS",
                            "severity": "INFO", "detail": "Default: block (deny by default)"})
        else:
            findings.append({"check": "Default USB policy", "status": "FAIL",
                            "severity": "HIGH",
                            "detail": f"Default: {policy} (should be 'block')"})

    # List rules
    result = subprocess.run(
        ["usbguard", "list-rules"],
        capture_output=True, text=True, timeout=10,
    )
    if result.returncode == 0:
        rules = result.stdout.strip().splitlines()
        findings.append({"check": "USBGuard rules", "status": "INFO",
                        "severity": "INFO", "detail": f"{len(rules)} rules configured"})

    return findings


def audit_linux_udev():
    """Check for USB-related udev rules on Linux."""
    findings = []
    udev_dirs = ["/etc/udev/rules.d", "/lib/udev/rules.d", "/usr/lib/udev/rules.d"]
    usb_rules_found = False

    for udev_dir in udev_dirs:
        if not os.path.isdir(udev_dir):
            continue
        for fname in os.listdir(udev_dir):
            fpath = os.path.join(udev_dir, fname)
            if not os.path.isfile(fpath):
                continue
            try:
                with open(fpath, "r") as f:
                    content = f.read()
                if "usb" in content.lower() and ("authorize" in content.lower() or "block" in content.lower()):
                    usb_rules_found = True
                    findings.append({"check": f"udev USB rule: {fname}", "status": "INFO",
                                    "severity": "INFO", "detail": fpath})
            except (IOError, PermissionError):
                pass

    if not usb_rules_found:
        findings.append({"check": "udev USB rules", "status": "WARN",
                         "severity": "MEDIUM", "detail": "No USB-specific udev rules found"})
    return findings


def list_connected_usb_devices():
    """List currently connected USB devices."""
    devices = []
    if sys.platform == "win32":
        ps_cmd = (
            "Get-PnpDevice -Class USB | "
            "Select-Object InstanceId, FriendlyName, Status, Class | "
            "ConvertTo-Json"
        )
        result = subprocess.run(
            ["powershell", "-Command", ps_cmd],
            capture_output=True, text=True, timeout=30,
        )
        if result.returncode == 0 and result.stdout.strip():
            try:
                raw = json.loads(result.stdout)
                if isinstance(raw, dict):
                    raw = [raw]
                for dev in raw:
                    devices.append({
                        "instance_id": dev.get("InstanceId", ""),
                        "name": dev.get("FriendlyName", "Unknown"),
                        "status": dev.get("Status", ""),
                        "class": dev.get("Class", "USB"),
                    })
            except json.JSONDecodeError:
                pass
    else:
        result = subprocess.run(
            ["lsusb"],
            capture_output=True, text=True, timeout=10,
        )
        if result.returncode == 0:
            for line in result.stdout.strip().splitlines():
                parts = line.split("ID ")
                if len(parts) >= 2:
                    devices.append({
                        "bus_info": parts[0].strip(),
                        "id": parts[1].split()[0] if parts[1].split() else "",
                        "name": " ".join(parts[1].split()[1:]) if len(parts[1].split()) > 1 else "Unknown",
                    })

    return devices


def format_summary(findings, devices):
    """Print audit summary."""
    print(f"\n{'='*60}")
    print(f"  USB Device Control Policy Audit")
    print(f"{'='*60}")
    print(f"  Connected Devices: {len(devices)}")
    print(f"  Policy Findings  : {len(findings)}")

    severity_counts = {}
    for f in findings:
        sev = f.get("severity", "INFO")
        severity_counts[sev] = severity_counts.get(sev, 0) + 1

    pass_count = sum(1 for f in findings if f["status"] == "PASS")
    fail_count = sum(1 for f in findings if f["status"] == "FAIL")
    print(f"  Passed : {pass_count}")
    print(f"  Failed : {fail_count}")

    if devices:
        print(f"\n  Connected USB Devices:")
        for d in devices:
            print(f"    {d.get('name', 'Unknown'):40s} | {d.get('id', d.get('instance_id', 'N/A'))}")

    if findings:
        print(f"\n  Policy Checks:")
        for f in findings:
            icon = "OK" if f["status"] == "PASS" else "!!" if f["status"] == "FAIL" else "--"
            print(f"    [{icon}] {f['check']}: {f.get('detail', '')[:50]}")

    return severity_counts


def main():
    parser = argparse.ArgumentParser(description="USB device control policy audit agent")
    parser.add_argument("--list-devices", action="store_true", help="List connected USB devices")
    parser.add_argument("--output", "-o", help="Output JSON report")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    findings = []
    devices = list_connected_usb_devices()

    if sys.platform != "win32":
        findings.extend(audit_linux_usbguard())
        findings.extend(audit_linux_udev())

    if not findings:
        findings.append({"check": "USB control policy", "status": "WARN",
                         "severity": "HIGH",
                         "detail": "No USB device control mechanism detected"})

    severity_counts = format_summary(findings, devices)

    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "USB Device Control Audit",
        "devices": devices,
        "findings": findings,
        "severity_counts": severity_counts,
        "risk_level": (
            "HIGH" if severity_counts.get("HIGH", 0) > 0
            else "MEDIUM" if severity_counts.get("MEDIUM", 0) > 0
            else "LOW"
        ),
    }

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"\n[+] Report saved to {args.output}")
    elif args.verbose:
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-usb-device-control-policy/scripts/process.py =====
#!/usr/bin/env python3
"""USB Device Control Audit - Analyzes USB device activity from endpoint logs."""

import json
import csv
import sys
import os
from collections import Counter, defaultdict
from datetime import datetime


def parse_usb_events(csv_path: str) -> list:
    """Parse USB device events from exported Windows event logs or EDR data."""
    events = []
    with open(csv_path, "r", encoding="utf-8-sig") as f:
        reader = csv.DictReader(f)
        for row in reader:
            events.append({
                "timestamp": row.get("Timestamp", row.get("Date and Time", "")),
                "host": row.get("Computer", row.get("DeviceName", "")),
                "user": row.get("User", row.get("AccountName", "")),
                "action": row.get("Action", row.get("ActionType", "")),
                "device_name": row.get("DeviceName", row.get("FriendlyName", "")),
                "device_id": row.get("DeviceId", row.get("HardwareId", "")),
                "vid_pid": row.get("VID_PID", ""),
                "serial": row.get("SerialNumber", ""),
                "blocked": row.get("Blocked", row.get("ActionType", "")).lower() in ("blocked", "deny", "prevented"),
            })
    return events


def analyze_usb_activity(events: list) -> dict:
    """Analyze USB events for policy violations and usage patterns."""
    analysis = {
        "total_events": len(events),
        "blocked_events": sum(1 for e in events if e["blocked"]),
        "unique_devices": len({e["device_id"] for e in events if e["device_id"]}),
        "devices_by_host": defaultdict(set),
        "top_users": Counter(),
        "blocked_devices": Counter(),
        "allowed_devices": Counter(),
    }

    for event in events:
        if event["host"] and event["device_id"]:
            analysis["devices_by_host"][event["host"]].add(event["device_id"])
        if event["user"]:
            analysis["top_users"][event["user"]] += 1
        if event["blocked"]:
            analysis["blocked_devices"][event.get("device_name", event["device_id"])] += 1
        else:
            analysis["allowed_devices"][event.get("device_name", event["device_id"])] += 1

    analysis["devices_by_host"] = {k: len(v) for k, v in analysis["devices_by_host"].items()}
    return analysis


def generate_report(analysis: dict, output_path: str) -> None:
    """Generate USB activity report."""
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "summary": {
            "total_events": analysis["total_events"],
            "blocked": analysis["blocked_events"],
            "unique_devices": analysis["unique_devices"],
        },
        "top_users": dict(analysis["top_users"].most_common(20)),
        "top_blocked_devices": dict(analysis["blocked_devices"].most_common(20)),
        "hosts_with_most_devices": dict(sorted(
            analysis["devices_by_host"].items(), key=lambda x: -x[1])[:20]),
    }
    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <usb_events.csv>")
        sys.exit(1)

    csv_path = sys.argv[1]
    events = parse_usb_events(csv_path)
    analysis = analyze_usb_activity(events)

    out = os.path.join(os.path.dirname(csv_path) or ".", "usb_audit_report.json")
    generate_report(analysis, out)
    print(f"Report: {out}")
    print(f"Total events: {analysis['total_events']}, Blocked: {analysis['blocked_events']}")

===== END FILE =====


===== FILE: skills/performing-endpoint-forensics-investigation/SKILL.md =====
---
name: performing-endpoint-forensics-investigation
description: 'Performs digital forensics investigation on compromised endpoints including
  memory acquisition, disk imaging, artifact analysis, and timeline reconstruction.
  Use when investigating security incidents, collecting evidence for legal proceedings,
  or analyzing endpoint compromise scope. Activates for requests involving endpoint
  forensics, memory analysis, disk forensics, or incident investigation.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- forensics
- memory-analysis
- disk-imaging
- incident-investigation
- Volatility
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
- T1005
---
# Performing Endpoint Forensics Investigation

## When to Use

Use this skill when:
- Investigating a confirmed or suspected endpoint compromise requiring forensic analysis
- Collecting volatile and non-volatile evidence for incident response or legal proceedings
- Analyzing memory dumps for malware, injected code, or credential theft artifacts
- Reconstructing attacker timelines from endpoint artifacts (prefetch, shimcache, amcache)

**Do not use** this skill for live threat hunting (use EDR/SIEM) or network forensics.

## Prerequisites

- Forensic workstation with analysis tools (Volatility 3, KAPE, Autopsy, Eric Zimmerman tools)
- Write-blocker for disk imaging (hardware or software)
- Secure evidence storage with chain-of-custody documentation
- Memory acquisition tool (WinPMEM, FTK Imager, Magnet RAM Capture)
- Administrative access to the target endpoint (or physical access)

## Workflow

### Step 1: Evidence Preservation (Order of Volatility)

Collect evidence from most volatile to least volatile:
```
1. System memory (RAM) - Most volatile
2. Network connections and routing tables
3. Running processes and open files
4. Disk contents (file system)
5. Removable media
6. Logs and backup data - Least volatile
```

**Memory Acquisition**:
```powershell
# WinPMEM (Windows)
winpmem_mini_x64.exe memdump.raw

# FTK Imager - Create memory capture via GUI
# File → Capture Memory → Destination path → Capture Memory

# Linux (LiME kernel module)
sudo insmod lime.ko "path=/evidence/memory.lime format=lime"
```

**Volatile Data Collection**:
```powershell
# Capture running processes
Get-Process | Export-Csv "evidence\processes.csv" -NoTypeInformation
tasklist /v > "evidence\tasklist.txt"

# Capture network connections
netstat -anob > "evidence\netstat.txt"
Get-NetTCPConnection | Export-Csv "evidence\tcp_connections.csv"

# Capture logged-on users
query user > "evidence\logged_users.txt"

# Capture scheduled tasks
schtasks /query /fo CSV /v > "evidence\scheduled_tasks.csv"

# Capture services
Get-Service | Export-Csv "evidence\services.csv"

# Capture DNS cache
ipconfig /displaydns > "evidence\dns_cache.txt"
```

### Step 2: Disk Imaging

```powershell
# FTK Imager - Create forensic disk image
# File → Create Disk Image → Physical Drive → E01 format
# Always verify image hash (MD5/SHA1) matches source

# dd (Linux)
sudo dc3dd if=/dev/sda of=/evidence/disk.dd hash=sha256 log=/evidence/imaging.log

# Verify image integrity
sha256sum /evidence/disk.dd
# Compare with hash generated during imaging
```

### Step 3: Memory Analysis with Volatility 3

```bash
# Identify OS profile
vol -f memdump.raw windows.info

# List running processes
vol -f memdump.raw windows.pslist
vol -f memdump.raw windows.pstree

# Find hidden processes
vol -f memdump.raw windows.psscan

# Analyze network connections
vol -f memdump.raw windows.netscan

# Detect process injection
vol -f memdump.raw windows.malfind

# Extract command line arguments
vol -f memdump.raw windows.cmdline

# Analyze DLLs loaded by processes
vol -f memdump.raw windows.dlllist --pid 1234

# Extract files from memory
vol -f memdump.raw windows.filescan | grep -i "suspicious"
vol -f memdump.raw windows.dumpfiles --pid 1234

# Detect credential theft
vol -f memdump.raw windows.hashdump
vol -f memdump.raw windows.lsadump

# Registry analysis from memory
vol -f memdump.raw windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"
```

### Step 4: Windows Artifact Analysis

```
Key forensic artifacts and their tools:

Prefetch Files (C:\Windows\Prefetch\):
  Tool: PECmd.exe (Eric Zimmerman)
  Shows: Program execution history with timestamps and run counts
  Command: PECmd.exe -d "C:\Windows\Prefetch" --csv output\

ShimCache (AppCompatCache):
  Tool: AppCompatCacheParser.exe
  Shows: Programs that existed on system (even if deleted)
  Command: AppCompatCacheParser.exe -f SYSTEM --csv output\

AmCache (C:\Windows\appcompat\Programs\Amcache.hve):
  Tool: AmcacheParser.exe
  Shows: Program execution with SHA1 hashes and install timestamps
  Command: AmcacheParser.exe -f Amcache.hve --csv output\

NTFS artifacts ($MFT, $UsnJrnl, $LogFile):
  Tool: MFTECmd.exe
  Shows: Complete file system timeline including deleted files
  Command: MFTECmd.exe -f "$MFT" --csv output\

Event Logs:
  Tool: EvtxECmd.exe
  Shows: Security, System, PowerShell, Sysmon events
  Command: EvtxECmd.exe -d "C:\Windows\System32\winevt\Logs" --csv output\

Registry Hives (SAM, SYSTEM, SOFTWARE, NTUSER.DAT):
  Tool: RECmd.exe with batch files
  Shows: User accounts, services, installed software, USB history
  Command: RECmd.exe -d "C:\Windows\System32\config" --bn BatchExamples\RECmd_Batch_MC.reb --csv output\
```

### Step 5: Timeline Reconstruction

```bash
# Use KAPE for automated artifact collection
kape.exe --tsource C: --tdest C:\evidence\kape_output \
  --target KapeTriage --module !EZParser

# Create super timeline with plaso/log2timeline
log2timeline.py timeline.plaso disk_image.E01
psort.py -o l2tcsv timeline.plaso -w timeline.csv

# Filter timeline around incident timeframe
psort.py -o l2tcsv timeline.plaso "date > '2026-02-20' AND date < '2026-02-22'" -w filtered_timeline.csv
```

### Step 6: Document Findings

Structure forensic report:
```
1. Executive Summary
2. Scope and Methodology
3. Evidence Inventory (with chain of custody)
4. Timeline of Events
5. Findings and Analysis
   - Initial access vector
   - Persistence mechanisms
   - Lateral movement
   - Data access/exfiltration
6. Indicators of Compromise (IOCs)
7. Recommendations
8. Appendices (tool output, hashes, raw evidence)
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Order of Volatility** | Evidence collection priority from most volatile (RAM) to least volatile (backups) |
| **Chain of Custody** | Documented record of evidence handling from collection to presentation |
| **Write Blocker** | Hardware or software device that prevents modification of source evidence |
| **Super Timeline** | Consolidated chronological view of all artifact timestamps for incident reconstruction |
| **Prefetch** | Windows artifact recording program execution history |
| **ShimCache** | Application compatibility artifact tracking program existence on endpoint |

## Tools & Systems

- **Volatility 3**: Memory forensics framework for analyzing RAM dumps
- **KAPE (Kroll Artifact Parser and Extractor)**: Automated triage collection and parsing
- **Eric Zimmerman Tools**: Suite of Windows artifact parsers (PECmd, MFTECmd, RECmd, etc.)
- **Autopsy/Sleuth Kit**: Disk forensics platform for file system analysis
- **FTK Imager**: Forensic imaging and memory acquisition tool
- **Plaso/log2timeline**: Super timeline creation framework

## Common Pitfalls

- **Modifying evidence on live system**: Always image before analysis. Running tools on a live system alters timestamps and memory state.
- **Forgetting chain of custody**: Evidence without documented chain of custody is inadmissible in legal proceedings.
- **Analyzing only disk, ignoring memory**: In-memory-only malware (fileless attacks) leaves no disk artifacts. Always capture memory first.
- **Not hashing evidence**: All evidence must be cryptographically hashed at collection time to prove integrity.
- **Tunnel vision**: Focusing on one artifact when the timeline tells a broader story. Always build a comprehensive timeline.

===== END FILE =====


===== FILE: skills/performing-endpoint-forensics-investigation/LICENSE =====

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


===== FILE: skills/performing-endpoint-forensics-investigation/assets/template.md =====
# Endpoint Forensics Investigation Template

## Case Information

| Field | Value |
|-------|-------|
| Case ID | |
| Incident Type | |
| Target Endpoint | |
| Investigation Date | |
| Lead Investigator | |

## Evidence Inventory

| Evidence ID | Type | Source | Hash (SHA256) | Collection Time | Collected By |
|-------------|------|--------|---------------|-----------------|-------------|
| | Memory dump | | | | |
| | Disk image | | | | |
| | Volatile data | | | | |

## Chain of Custody

| Date/Time | Action | From | To | Notes |
|-----------|--------|------|-----|-------|
| | Collected | Endpoint | Evidence storage | |

## Findings Summary

| Finding | ATT&CK Technique | Evidence | Confidence |
|---------|------------------|----------|------------|
| Initial access | | | |
| Persistence | | | |
| Lateral movement | | | |
| Data access | | | |

## IOCs Extracted

| Type | Value | Context |
|------|-------|---------|
| SHA256 | | |
| IP | | |
| Domain | | |
| File path | | |

## Sign-Off

| Role | Name | Date |
|------|------|------|
| Lead Investigator | | |
| IR Manager | | |

===== END FILE =====


===== FILE: skills/performing-endpoint-forensics-investigation/references/api-reference.md =====
# API Reference — Performing Endpoint Forensics Investigation

## Libraries Used

| Library | Purpose |
|---------|---------|
| `subprocess` | Execute Windows forensic commands (wmic, netstat, reg, schtasks) |
| `hashlib` | Calculate MD5, SHA1, SHA256 hashes for evidence integrity |
| `csv` | Parse WMIC CSV output |
| `json` | Structure and export forensic triage results |
| `datetime` | Timestamp evidence collection |
| `argparse` | CLI argument parsing for triage modes |

## CLI Interface

```bash
python agent.py triage      # Full forensic triage
python agent.py processes   # Running processes with PIDs and command lines
python agent.py network     # Active network connections
python agent.py autoruns    # Persistence entries
python agent.py hash --file <filepath>  # Hash file for evidence
```

## Core Functions

### `full_triage()` — Run all collection functions
```python
def full_triage():
    """Execute full forensic triage and return combined results."""
    return {
        "timestamp": datetime.now().isoformat(),
        "hostname": collect_system_info()["hostname"],
        "system_info": collect_system_info(),
        "processes": collect_running_processes(),
        "network": collect_network_connections(),
        "autoruns": collect_autoruns(),
        "users": collect_user_accounts(),
    }
```

### `collect_system_info()` — Hostname, OS version, network config, uptime
```python
def collect_system_info():
    result = subprocess.run(
        ["systeminfo"], capture_output=True, text=True, timeout=60,
    )
    info = {}
    for line in result.stdout.split("\n"):
        if ":" in line:
            key, _, val = line.partition(":")
            info[key.strip()] = val.strip()
    return {
        "hostname": info.get("Host Name", ""),
        "os_name": info.get("OS Name", ""),
        "os_version": info.get("OS Version", ""),
        "system_boot_time": info.get("System Boot Time", ""),
        "total_physical_memory": info.get("Total Physical Memory", ""),
        "domain": info.get("Domain", ""),
    }
```

### `collect_running_processes()` — Process list via `wmic process get`
```python
def collect_running_processes():
    result = subprocess.run(
        ["wmic", "process", "get",
         "ProcessId,Name,ExecutablePath,CommandLine,ParentProcessId",
         "/format:csv"],
        capture_output=True, text=True, timeout=30,
    )
    processes = []
    reader = csv.DictReader(result.stdout.strip().split("\n"))
    for row in reader:
        if row.get("Name"):
            processes.append({
                "pid": row.get("ProcessId"),
                "name": row.get("Name"),
                "path": row.get("ExecutablePath", ""),
                "cmdline": row.get("CommandLine", ""),
                "ppid": row.get("ParentProcessId"),
            })
    return processes
```

### `collect_network_connections()` — Active connections via `netstat -ano`
```python
def collect_network_connections():
    result = subprocess.run(
        ["netstat", "-ano"], capture_output=True, text=True, timeout=15,
    )
    connections = []
    for line in result.stdout.strip().split("\n")[4:]:
        parts = line.split()
        if len(parts) >= 5:
            connections.append({
                "proto": parts[0],
                "local_address": parts[1],
                "remote_address": parts[2],
                "state": parts[3] if parts[3] != parts[-1] else "",
                "pid": parts[-1],
            })
    return connections
```

### `collect_autoruns()` — Registry Run keys and scheduled tasks
```python
RUN_KEYS = [
    r"HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run",
    r"HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce",
    r"HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run",
    r"HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce",
]

def collect_autoruns():
    autoruns = {"registry_run_keys": [], "scheduled_tasks": []}

    for key in RUN_KEYS:
        result = subprocess.run(
            ["reg", "query", key], capture_output=True, text=True, timeout=10,
        )
        for line in result.stdout.strip().split("\n"):
            parts = line.strip().split("    ")
            if len(parts) >= 3:
                autoruns["registry_run_keys"].append({
                    "key": key,
                    "name": parts[0].strip(),
                    "value": parts[-1].strip(),
                })

    result = subprocess.run(
        ["schtasks", "/query", "/fo", "csv", "/v"],
        capture_output=True, text=True, timeout=30,
    )
    reader = csv.DictReader(result.stdout.strip().split("\n"))
    for row in reader:
        if row.get("TaskName") and row.get("Status") == "Ready":
            autoruns["scheduled_tasks"].append({
                "name": row.get("TaskName"),
                "next_run": row.get("Next Run Time"),
                "task_to_run": row.get("Task To Run"),
                "run_as_user": row.get("Run As User"),
            })

    return autoruns
```

### `collect_user_accounts()` — Local user enumeration
```python
def collect_user_accounts():
    result = subprocess.run(
        ["net", "user"], capture_output=True, text=True, timeout=10,
    )
    users = []
    for line in result.stdout.strip().split("\n")[4:]:
        for name in line.split():
            if name and not name.startswith("-"):
                users.append(name)
    return users
```

### `hash_file(filepath)` — MD5/SHA1/SHA256 hash calculation
```python
def hash_file(filepath):
    """Calculate cryptographic hashes for evidence integrity."""
    md5 = hashlib.md5()
    sha1 = hashlib.sha1()
    sha256 = hashlib.sha256()

    with open(filepath, "rb") as f:
        while chunk := f.read(8192):
            md5.update(chunk)
            sha1.update(chunk)
            sha256.update(chunk)

    return {
        "file": filepath,
        "md5": md5.hexdigest(),
        "sha1": sha1.hexdigest(),
        "sha256": sha256.hexdigest(),
    }
```

## Output Format

```json
{
  "timestamp": "2025-01-15T10:30:00",
  "hostname": "WORKSTATION-01",
  "system_info": {
    "os_name": "Microsoft Windows 10 Pro",
    "os_version": "10.0.19045",
    "domain": "CORP"
  },
  "processes": [
    {"pid": "4532", "name": "powershell.exe", "cmdline": "powershell -enc ..."}
  ],
  "network": [
    {"proto": "TCP", "local_address": "10.0.0.5:49721", "remote_address": "198.51.100.42:443", "state": "ESTABLISHED", "pid": "4532"}
  ],
  "autoruns": {
    "registry_run_keys": [
      {"key": "HKCU\\...\\Run", "name": "WindowsUpdate", "value": "C:\\Users\\Public\\update.exe"}
    ],
    "scheduled_tasks": 45
  }
}
```

## Dependencies

No external packages — uses Windows built-in commands and Python standard library.

===== END FILE =====


===== FILE: skills/performing-endpoint-forensics-investigation/references/standards.md =====
# Standards & References

## Primary Standards

### NIST SP 800-86 - Guide to Integrating Forensic Techniques
- **Publisher**: NIST
- **Scope**: Forensic process for digital evidence collection, examination, and analysis

### ISO/IEC 27037 - Guidelines for Digital Evidence
- **Publisher**: ISO
- **Scope**: Identification, collection, acquisition, and preservation of digital evidence

### RFC 3227 - Guidelines for Evidence Collection and Archiving
- **Publisher**: IETF
- **Scope**: Best practices for evidence collection including order of volatility

## Compliance Mappings

| Framework | Requirement | Forensics Coverage |
|-----------|------------|-------------------|
| NIST 800-53 | IR-4 Incident Handling | Forensic investigation procedures |
| NIST 800-53 | AU-10 Non-repudiation | Evidence integrity via hashing |
| PCI DSS 4.0 | 12.10.5 - Incident response with forensics | Post-incident forensic analysis |
| HIPAA | 164.308(a)(6)(ii) - Response and Reporting | Forensic investigation of breaches |

## Tool References

- **Volatility 3**: https://github.com/volatilityfoundation/volatility3
- **KAPE**: https://www.kroll.com/en/insights/publications/cyber/kroll-artifact-parser-extractor-kape
- **Eric Zimmerman Tools**: https://ericzimmerman.github.io/
- **Autopsy**: https://www.autopsy.com/
- **SANS Windows Forensic Analysis Poster**: https://www.sans.org/posters/

===== END FILE =====


===== FILE: skills/performing-endpoint-forensics-investigation/references/workflows.md =====
# Workflows

## Workflow 1: Endpoint Forensic Investigation

```
[Incident Detected / Investigation Authorized]
    │
    ▼
[Preserve Evidence (Order of Volatility)]
    │
    ├── 1. Capture memory (WinPMEM/FTK Imager)
    ├── 2. Capture volatile data (processes, network, users)
    ├── 3. Create forensic disk image (E01/dd)
    ├── 4. Hash all evidence, document chain of custody
    │
    ▼
[Analysis Phase]
    │
    ├── Memory analysis (Volatility 3)
    ├── Artifact parsing (KAPE + EZ tools)
    ├── Timeline reconstruction (plaso)
    ├── Malware analysis (if samples found)
    │
    ▼
[Correlate Findings]
    │
    ├── Initial access vector identified
    ├── Persistence mechanisms documented
    ├── Scope of compromise determined
    │
    ▼
[Generate IOCs and Report]
    │
    ▼
[Handoff to Remediation Team]
```

## Workflow 2: Memory Analysis

```
[Memory dump acquired]
    │
    ▼
[Identify OS profile: vol windows.info]
    │
    ▼
[Process analysis: pslist → pstree → psscan]
    │
    ├── Hidden processes found ──► [Analyze with malfind, dlllist]
    │
    ▼
[Network analysis: netscan]
    │
    ├── Suspicious connections ──► [Extract IOCs (IPs, domains)]
    │
    ▼
[Injection detection: malfind]
    │
    ├── Injected code found ──► [Dump and analyze with YARA]
    │
    ▼
[Credential analysis: hashdump, lsadump]
    │
    ▼
[Document all findings with screenshots and hashes]
```

===== END FILE =====


===== FILE: skills/performing-endpoint-forensics-investigation/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing endpoint forensics investigation on Windows systems."""

import json
import argparse
import subprocess
import os
import hashlib
from datetime import datetime


def collect_system_info():
    """Collect basic system information for forensic context."""
    info = {}
    commands = {
        "hostname": ["hostname"],
        "os_version": ["wmic", "os", "get", "Caption,Version,BuildNumber", "/format:list"],
        "network_config": ["ipconfig", "/all"],
        "logged_users": ["query", "user"],
        "uptime": ["wmic", "os", "get", "LastBootUpTime", "/format:list"],
    }
    for key, cmd in commands.items():
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
            info[key] = result.stdout.strip()[:1000]
        except Exception as e:
            info[key] = f"Error: {e}"
    return {"timestamp": datetime.utcnow().isoformat(), "system_info": info}


def collect_running_processes():
    """Collect running processes with parent PIDs and command lines."""
    try:
        result = subprocess.run(
            ["wmic", "process", "get", "Name,ProcessId,ParentProcessId,CommandLine,ExecutablePath", "/format:csv"],
            capture_output=True, text=True, timeout=30
        )
    except Exception as e:
        return {"error": str(e)}
    import csv
    from io import StringIO
    processes = []
    reader = csv.DictReader(StringIO(result.stdout))
    for row in reader:
        if row.get("Name"):
            processes.append({
                "name": row.get("Name", ""),
                "pid": row.get("ProcessId", ""),
                "ppid": row.get("ParentProcessId", ""),
                "path": row.get("ExecutablePath", ""),
                "cmdline": row.get("CommandLine", "")[:500],
            })
    return {"total": len(processes), "processes": processes}


def collect_network_connections():
    """Collect active network connections."""
    try:
        result = subprocess.run(
            ["netstat", "-ano"], capture_output=True, text=True, timeout=15
        )
    except Exception as e:
        return {"error": str(e)}
    connections = []
    for line in result.stdout.split("\n")[4:]:
        parts = line.split()
        if len(parts) >= 5:
            connections.append({
                "protocol": parts[0],
                "local_addr": parts[1],
                "remote_addr": parts[2],
                "state": parts[3] if len(parts) > 4 else "",
                "pid": parts[-1],
            })
    established = [c for c in connections if c.get("state") == "ESTABLISHED"]
    listening = [c for c in connections if c.get("state") == "LISTENING"]
    return {
        "total": len(connections),
        "established": len(established),
        "listening": len(listening),
        "connections": connections,
    }


def collect_autoruns():
    """Collect common persistence locations."""
    autoruns = {}
    reg_keys = [
        r"HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run",
        r"HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run",
    ]
    for key in reg_keys:
        try:
            result = subprocess.run(["reg", "query", key], capture_output=True, text=True, timeout=10)
            autoruns[key] = result.stdout.strip()[:1000]
        except Exception:
            continue
    try:
        result = subprocess.run(["schtasks", "/query", "/fo", "CSV"], capture_output=True, text=True, timeout=30)
        autoruns["scheduled_tasks_count"] = result.stdout.count("\n") - 1
    except Exception:
        pass
    return autoruns


def hash_file(filepath):
    """Calculate MD5, SHA1, SHA256 hashes of a file for evidence integrity."""
    hashes = {}
    algos = {"md5": hashlib.md5(), "sha1": hashlib.sha1(), "sha256": hashlib.sha256()}
    try:
        with open(filepath, "rb") as f:
            while True:
                chunk = f.read(8192)
                if not chunk:
                    break
                for algo in algos.values():
                    algo.update(chunk)
        for name, algo in algos.items():
            hashes[name] = algo.hexdigest()
        hashes["file"] = str(filepath)
        hashes["size"] = os.path.getsize(filepath)
    except Exception as e:
        hashes["error"] = str(e)
    return hashes


def full_triage():
    """Run full endpoint forensic triage collection."""
    return {
        "timestamp": datetime.utcnow().isoformat(),
        "system_info": collect_system_info(),
        "processes": collect_running_processes(),
        "network": collect_network_connections(),
        "autoruns": collect_autoruns(),
    }


def main():
    parser = argparse.ArgumentParser(description="Endpoint Forensics Investigation Agent")
    sub = parser.add_subparsers(dest="command")
    sub.add_parser("triage", help="Full forensic triage collection")
    sub.add_parser("processes", help="Collect running processes")
    sub.add_parser("network", help="Collect network connections")
    sub.add_parser("autoruns", help="Collect autorun/persistence entries")
    h = sub.add_parser("hash", help="Hash a file for evidence")
    h.add_argument("--file", required=True)
    args = parser.parse_args()
    if args.command == "triage":
        result = full_triage()
    elif args.command == "processes":
        result = collect_running_processes()
    elif args.command == "network":
        result = collect_network_connections()
    elif args.command == "autoruns":
        result = collect_autoruns()
    elif args.command == "hash":
        result = hash_file(args.file)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-endpoint-forensics-investigation/scripts/process.py =====
#!/usr/bin/env python3
"""
Forensic Evidence Processor

Parses and correlates forensic artifacts from KAPE/EZ tool output
to generate consolidated timeline and IOC reports.
"""

import json
import csv
import sys
import os
from datetime import datetime
from collections import defaultdict


def parse_prefetch_csv(csv_path: str) -> list:
    """Parse PECmd output for program execution history."""
    entries = []
    with open(csv_path, "r", encoding="utf-8-sig") as f:
        reader = csv.DictReader(f)
        for row in reader:
            entries.append({
                "artifact": "prefetch",
                "timestamp": row.get("LastRun", ""),
                "executable": row.get("ExecutableName", ""),
                "run_count": row.get("RunCount", ""),
                "path": row.get("SourceFilename", ""),
                "hash": row.get("Hash", ""),
                "volume": row.get("Volume0Name", ""),
            })
    return entries


def parse_shimcache_csv(csv_path: str) -> list:
    """Parse AppCompatCacheParser output."""
    entries = []
    with open(csv_path, "r", encoding="utf-8-sig") as f:
        reader = csv.DictReader(f)
        for row in reader:
            entries.append({
                "artifact": "shimcache",
                "timestamp": row.get("LastModifiedTimeUTC", ""),
                "path": row.get("Path", ""),
                "executed": row.get("Executed", ""),
            })
    return entries


def parse_amcache_csv(csv_path: str) -> list:
    """Parse AmcacheParser output for installed programs."""
    entries = []
    with open(csv_path, "r", encoding="utf-8-sig") as f:
        reader = csv.DictReader(f)
        for row in reader:
            entries.append({
                "artifact": "amcache",
                "timestamp": row.get("FileKeyLastWriteTimestamp", ""),
                "path": row.get("FullPath", row.get("Name", "")),
                "sha1": row.get("SHA1", ""),
                "publisher": row.get("Publisher", ""),
                "product": row.get("ProductName", ""),
            })
    return entries


def parse_mft_csv(csv_path: str) -> list:
    """Parse MFTECmd output for file system timeline."""
    entries = []
    with open(csv_path, "r", encoding="utf-8-sig") as f:
        reader = csv.DictReader(f)
        for row in reader:
            entries.append({
                "artifact": "mft",
                "timestamp_created": row.get("Created0x10", ""),
                "timestamp_modified": row.get("LastModified0x10", ""),
                "path": row.get("ParentPath", "") + "\\" + row.get("FileName", ""),
                "size": row.get("FileSize", ""),
                "in_use": row.get("InUse", ""),
                "is_directory": row.get("IsDirectory", ""),
            })
    return entries


def build_timeline(all_entries: list) -> list:
    """Build consolidated timeline from all artifact sources."""
    timeline = []

    for entry in all_entries:
        ts = ""
        for key in ["timestamp", "timestamp_created", "timestamp_modified"]:
            if entry.get(key):
                ts = entry[key]
                break

        if ts:
            timeline.append({
                "timestamp": ts,
                "artifact": entry.get("artifact", "unknown"),
                "description": entry.get("path", entry.get("executable", "")),
                "details": {k: v for k, v in entry.items()
                            if k not in ("timestamp", "artifact")},
            })

    timeline.sort(key=lambda x: x["timestamp"])
    return timeline


def extract_iocs(all_entries: list) -> dict:
    """Extract potential IOCs from forensic artifacts."""
    iocs = {
        "file_hashes": set(),
        "suspicious_paths": [],
        "executables": set(),
    }

    suspicious_dirs = [
        "\\temp\\", "\\tmp\\", "\\appdata\\local\\temp\\",
        "\\users\\public\\", "\\programdata\\",
        "\\recycle", "\\windows\\debug\\",
    ]

    for entry in all_entries:
        for hash_key in ["hash", "sha1", "md5"]:
            h = entry.get(hash_key, "")
            if h and len(h) >= 32:
                iocs["file_hashes"].add(h)

        path = entry.get("path", "").lower()
        if any(d in path for d in suspicious_dirs):
            if path.endswith((".exe", ".dll", ".ps1", ".bat", ".vbs", ".js")):
                iocs["suspicious_paths"].append({
                    "path": entry.get("path", ""),
                    "artifact": entry.get("artifact", ""),
                    "timestamp": entry.get("timestamp", ""),
                })

        exe = entry.get("executable", "")
        if exe:
            iocs["executables"].add(exe)

    iocs["file_hashes"] = sorted(iocs["file_hashes"])
    iocs["executables"] = sorted(iocs["executables"])
    return iocs


def generate_report(timeline: list, iocs: dict, output_path: str) -> None:
    """Generate forensic analysis report."""
    report = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "timeline_entries": len(timeline),
        "iocs": {
            "file_hashes": iocs["file_hashes"][:100],
            "suspicious_files": iocs["suspicious_paths"][:50],
            "unique_executables": len(iocs["executables"]),
        },
        "timeline_sample": timeline[:100],
    }

    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <kape_output_directory>")
        print()
        print("Parses KAPE/EZ tool CSV output and generates timeline + IOC report.")
        sys.exit(1)

    kape_dir = sys.argv[1]
    if not os.path.isdir(kape_dir):
        print(f"Error: Directory not found: {kape_dir}")
        sys.exit(1)

    all_entries = []

    for root, dirs, files in os.walk(kape_dir):
        for f in files:
            if not f.endswith(".csv"):
                continue
            path = os.path.join(root, f)
            fl = f.lower()
            try:
                if "prefetch" in fl or "pecmd" in fl:
                    all_entries.extend(parse_prefetch_csv(path))
                elif "shimcache" in fl or "appcompat" in fl:
                    all_entries.extend(parse_shimcache_csv(path))
                elif "amcache" in fl:
                    all_entries.extend(parse_amcache_csv(path))
                elif "mft" in fl:
                    all_entries.extend(parse_mft_csv(path))
            except Exception as e:
                print(f"Warning: Could not parse {path}: {e}")

    print(f"Parsed {len(all_entries)} artifact entries")

    timeline = build_timeline(all_entries)
    iocs = extract_iocs(all_entries)

    report_path = os.path.join(kape_dir, "forensic_analysis.json")
    generate_report(timeline, iocs, report_path)
    print(f"Forensic report: {report_path}")

    print(f"\n--- Forensic Summary ---")
    print(f"Timeline entries: {len(timeline)}")
    print(f"Unique file hashes: {len(iocs['file_hashes'])}")
    print(f"Suspicious file paths: {len(iocs['suspicious_paths'])}")
    print(f"Unique executables: {len(iocs['executables'])}")

===== END FILE =====


===== FILE: skills/performing-endpoint-vulnerability-remediation/SKILL.md =====
---
name: performing-endpoint-vulnerability-remediation
description: 'Performs vulnerability remediation on endpoints by prioritizing CVEs
  based on risk scoring, deploying patches, applying configuration changes, and validating
  fixes. Use when remediating findings from vulnerability scans, responding to critical
  CVE advisories, or maintaining endpoint compliance with patch management SLAs. Activates
  for requests involving vulnerability remediation, CVE patching, endpoint vulnerability
  management, or security fix deployment.

  '
domain: cybersecurity
subdomain: endpoint-security
tags:
- endpoint
- vulnerability-management
- patching
- CVE
- remediation
- CVSS
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-02
- DE.CM-01
- PR.IR-01
mitre_attack:
- T1055
- T1547
- T1059
- T1036
---
# Performing Endpoint Vulnerability Remediation

## When to Use

Use this skill when:
- Remediating vulnerabilities identified by scanners (Nessus, Qualys, Rapid7)
- Responding to zero-day CVE advisories requiring immediate patching
- Maintaining compliance with patch management SLAs (critical within 14 days, high within 30 days)
- Building a prioritized remediation plan from vulnerability scan results

**Do not use** this skill for vulnerability scanning itself (use scanning tools) or for application-layer vulnerability remediation (use DevSecOps processes).

## Prerequisites

- Vulnerability scan results (Nessus, Qualys, or Rapid7 export in CSV/XML format)
- Patch management platform (WSUS, SCCM, Intune, or third-party like Automox)
- Administrative access to target endpoints or deployment infrastructure
- Change management process for production endpoint patching
- Testing environment for patch validation before production rollout

## Workflow

### Step 1: Import and Prioritize Vulnerability Findings

```
Priority scoring combines:
1. CVSS Base Score (0-10)
2. EPSS (Exploit Prediction Scoring System) - probability of exploitation
3. CISA KEV (Known Exploited Vulnerabilities) catalog membership
4. Asset criticality (business impact of affected endpoint)
5. Network exposure (internet-facing vs. internal)

Priority Matrix:
  P1 (Critical - 14 days SLA):
    - CVSS >= 9.0 OR
    - Listed in CISA KEV OR
    - Active exploitation in the wild + CVSS >= 7.0

  P2 (High - 30 days SLA):
    - CVSS 7.0-8.9 AND
    - EPSS > 0.5 (50% probability of exploitation)

  P3 (Medium - 60 days SLA):
    - CVSS 4.0-6.9 OR
    - CVSS 7.0-8.9 with EPSS < 0.1

  P4 (Low - 90 days SLA):
    - CVSS < 4.0 AND
    - No known exploit
```

### Step 2: Identify Remediation Actions

For each vulnerability, determine the appropriate remediation:

```
Remediation Types:
1. Patch: Apply vendor security update (most common)
2. Configuration change: Modify settings to mitigate (registry, GPO)
3. Upgrade: Update to newer software version
4. Workaround: Apply temporary mitigation when patch unavailable
5. Compensating control: Network segmentation, WAF rule, EDR rule
6. Accept risk: Document accepted risk with CISO sign-off
```

### Step 3: Deploy Patches via WSUS/SCCM

```powershell
# WSUS: Approve patches for deployment
# 1. Open WSUS Console
# 2. Navigate to Updates → Security Updates
# 3. Approve selected KBs for target computer groups

# SCCM: Create Software Update Group
# 1. Software Library → Software Updates → All Software Updates
# 2. Select required KBs → Create Software Update Group
# 3. Deploy to target collection with maintenance window

# Intune: Create Windows Update Ring
# Devices → Windows → Update rings
# Configure: Quality updates deferral = 0 days (for critical)
# Feature updates deferral = per policy

# PowerShell: Force Windows Update check
Install-Module PSWindowsUpdate -Force
Get-WindowsUpdate -KBArticleID "KB5034441" -Install -AcceptAll -AutoReboot

# Verify patch installation
Get-HotFix -Id "KB5034441"
systeminfo | findstr "KB5034441"
```

### Step 4: Apply Configuration-Based Remediations

```powershell
# Example: Disable SMBv1 (CVE-2017-0144 - EternalBlue)
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart

# Example: Disable Print Spooler on non-print servers (CVE-2021-34527 - PrintNightmare)
Stop-Service -Name Spooler -Force
Set-Service -Name Spooler -StartupType Disabled

# Example: Disable LLMNR (credential theft mitigation)
# Via GPO: Computer Configuration → Admin Templates → Network → DNS Client
# Turn off multicast name resolution: Enabled
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" `
  -Name EnableMulticast -Value 0 -PropertyType DWORD -Force

# Example: Restrict NTLM authentication
# Via GPO: Security Settings → Local Policies → Security Options
# Network security: Restrict NTLM: Audit/Deny
```

### Step 5: Handle Zero-Day Vulnerabilities (No Patch Available)

```
When vendor patch is not yet available:

1. Check vendor advisory for workarounds
   - Microsoft: https://msrc.microsoft.com/update-guide
   - Adobe: https://helpx.adobe.com/security.html
   - Linux: Distribution security trackers

2. Apply temporary mitigations:
   - Disable vulnerable feature/service
   - Deploy EDR detection rule for exploitation attempt
   - Apply network-level blocking (WAF/firewall rules)
   - Restrict access to vulnerable application

3. Monitor for patch release:
   - Subscribe to vendor security mailing list
   - Monitor CISA KEV additions
   - Set calendar reminder for next Patch Tuesday

4. Document workaround with expiration date
```

### Step 6: Validate Remediation

```powershell
# Re-scan remediated endpoints to confirm vulnerability closure
# Option 1: Targeted vulnerability scan
nessuscli scan --target 192.168.1.0/24 --plugin-id 12345

# Option 2: PowerShell verification
# Check specific KB is installed
$kb = Get-HotFix -Id "KB5034441" -ErrorAction SilentlyContinue
if ($kb) {
    Write-Host "PASS: KB5034441 installed on $(hostname)" -ForegroundColor Green
} else {
    Write-Host "FAIL: KB5034441 missing on $(hostname)" -ForegroundColor Red
}

# Check service is disabled
$svc = Get-Service -Name Spooler
if ($svc.StartType -eq 'Disabled') {
    Write-Host "PASS: Print Spooler disabled" -ForegroundColor Green
}

# Check registry configuration
$val = Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" `
  -Name SMB1 -ErrorAction SilentlyContinue
if ($val.SMB1 -eq 0) {
    Write-Host "PASS: SMBv1 disabled" -ForegroundColor Green
}
```

### Step 7: Report and Track

Generate remediation status report:
```
Remediation Metrics:
  - Total vulnerabilities: X
  - Remediated: Y (Z%)
  - Pending (within SLA): A
  - Overdue (past SLA): B
  - Accepted risk: C
  - Mean time to remediate (MTTR): D days
  - SLA compliance rate: E%
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **CVSS** | Common Vulnerability Scoring System; 0-10 severity scale for vulnerabilities |
| **EPSS** | Exploit Prediction Scoring System; probability (0-1) that a CVE will be exploited in the wild within 30 days |
| **CISA KEV** | CISA Known Exploited Vulnerabilities catalog; federal mandate to patch these CVEs within specified timeframes |
| **SLA** | Service Level Agreement for remediation timelines based on vulnerability severity |
| **MTTR** | Mean Time To Remediate; average days from vulnerability discovery to confirmed fix |
| **Compensating Control** | Alternative security measure when direct remediation is not feasible |

## Tools & Systems

- **Nessus/Tenable.io**: Vulnerability scanning and remediation tracking
- **Qualys VMDR**: Vulnerability management, detection, and response platform
- **Rapid7 InsightVM**: Vulnerability assessment with live dashboards
- **WSUS/SCCM/Intune**: Microsoft patch deployment infrastructure
- **Automox**: Cloud-native patch management for Windows, macOS, Linux
- **CISA KEV Catalog**: https://www.cisa.gov/known-exploited-vulnerabilities-catalog

## Common Pitfalls

- **Patching without testing**: Apply patches to a test group first. Some patches cause application compatibility issues or BSOD.
- **Ignoring EPSS scores**: A CVSS 9.8 vulnerability with EPSS 0.01 may be less urgent than a CVSS 7.5 with EPSS 0.95 (actively exploited).
- **Not validating remediation**: Deploying a patch does not guarantee installation. Always re-scan to confirm closure.
- **Excluding critical servers from patching**: Servers that "cannot be rebooted" accumulate critical vulnerabilities. Schedule maintenance windows.
- **Treating all CVEs equally**: Risk-based prioritization (CVSS + EPSS + asset criticality + exposure) is more effective than patching all criticals first.

===== END FILE =====


===== FILE: skills/performing-endpoint-vulnerability-remediation/LICENSE =====

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


===== FILE: skills/performing-endpoint-vulnerability-remediation/assets/template.md =====
# Endpoint Vulnerability Remediation Template

## Scan Information

| Field | Value |
|-------|-------|
| Scanner | Nessus / Qualys / Rapid7 |
| Scan Date | |
| Scope | |
| Total Findings | |
| Scan Policy | |

## Remediation Priority Summary

| Priority | Count | SLA | Deadline | Status |
|----------|-------|-----|----------|--------|
| P1 (Critical) | | 14 days | | |
| P2 (High) | | 30 days | | |
| P3 (Medium) | | 60 days | | |
| P4 (Low) | | 90 days | | |

## CISA KEV Vulnerabilities (Mandatory Remediation)

| CVE | Affected Hosts | CVSS | Due Date | Remediation | Status |
|-----|---------------|------|----------|-------------|--------|
| | | | | | |

## Remediation Actions

| CVE/Plugin | Host(s) | Action Type | Details | Assigned To | Status |
|-----------|---------|-------------|---------|-------------|--------|
| | | Patch | | | |
| | | Config change | | | |
| | | Workaround | | | |
| | | Accept risk | | | |

## Patch Deployment Schedule

| Phase | Target | Date | Maintenance Window | Rollback Plan |
|-------|--------|------|--------------------|---------------|
| Test ring | 5% endpoints | | | |
| Pilot ring | 20% endpoints | | | |
| Production | Remaining | | | |

## Risk Acceptance Register

| CVE | Host | CVSS | Business Justification | Compensating Control | Approved By | Expiry |
|-----|------|------|----------------------|---------------------|-------------|--------|
| | | | | | | |

## Validation Results

| CVE/Plugin | Host | Pre-Remediation | Post-Remediation | Verified By |
|-----------|------|-----------------|------------------|-------------|
| | | Vulnerable | Fixed / Still Vulnerable | |

## Metrics

| Metric | Value |
|--------|-------|
| Total vulnerabilities | |
| Remediated | |
| Remediation rate | % |
| MTTR (Mean Time to Remediate) | days |
| SLA compliance | % |
| Overdue count | |

## Sign-Off

| Role | Name | Date |
|------|------|------|
| Vulnerability Analyst | | |
| Patch Manager | | |
| Security Manager | | |

===== END FILE =====


===== FILE: skills/performing-endpoint-vulnerability-remediation/references/api-reference.md =====
# API Reference — Performing Endpoint Vulnerability Remediation

## Libraries Used

| Library | Purpose |
|---------|---------|
| `csv` | Parse vulnerability scan CSV exports (Nessus, Qualys, Rapid7) |
| `subprocess` | Check installed Windows patches via `wmic qfe` and PowerShell |
| `socket` | Validate port-based remediation via TCP connect |
| `json` | Read/write remediation plans and reports |
| `argparse` | CLI argument parsing for scan file and host parameters |
| `datetime` | Track patch dates and SLA deadlines |

## CLI Interface

```bash
python agent.py parse --scan-file scan.csv
python agent.py patches
python agent.py validate --host 10.0.0.1 --port 445
python agent.py report --scan-file scan.csv [--output plan.json]
```

## Core Functions

### `parse_scan_report(csv_file)` — Parse and prioritize vulnerabilities by severity
```python
def parse_scan_report(csv_file):
    """Parse Nessus/Qualys CSV export, group by host, sort by severity."""
    with open(csv_file, newline="") as f:
        reader = csv.DictReader(f)
        vulns = []
        for row in reader:
            vulns.append({
                "host": row.get("Host", row.get("IP")),
                "plugin_id": row.get("Plugin ID", row.get("QID")),
                "severity": row.get("Risk", row.get("Severity", "Info")),
                "name": row.get("Name", row.get("Title")),
                "cve": row.get("CVE", ""),
                "solution": row.get("Solution", row.get("Fix", "")),
            })
    severity_order = {"Critical": 0, "High": 1, "Medium": 2, "Low": 3, "Info": 4}
    return sorted(vulns, key=lambda v: severity_order.get(v["severity"], 5))
```

### `check_windows_patches()` — List installed Windows hotfixes via WMIC
```python
def check_windows_patches():
    """Query installed patches on a Windows endpoint."""
    result = subprocess.run(
        ["wmic", "qfe", "get", "HotFixID,InstalledOn,Description", "/format:csv"],
        capture_output=True, text=True, timeout=30,
    )
    patches = []
    for line in result.stdout.strip().split("\n")[1:]:
        parts = line.strip().split(",")
        if len(parts) >= 4:
            patches.append({
                "hotfix_id": parts[1],
                "description": parts[2],
                "installed_on": parts[3],
            })
    return patches
```

### `validate_remediation(host, port)` — TCP connect to verify port closure
```python
def validate_remediation(host, port):
    """Verify that a vulnerable port has been closed after remediation."""
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(5)
    try:
        result = sock.connect_ex((host, int(port)))
        return {
            "host": host,
            "port": port,
            "status": "open" if result == 0 else "closed",
            "remediated": result != 0,
        }
    finally:
        sock.close()
```

### `generate_remediation_report(scan_file, output)` — Group vulns by host for remediation
```python
def generate_remediation_report(scan_file, output=None):
    """Generate a prioritized remediation plan from scan results."""
    vulns = parse_scan_report(scan_file)
    by_host = {}
    for v in vulns:
        by_host.setdefault(v["host"], []).append(v)

    report = {
        "total_vulns": len(vulns),
        "hosts_affected": len(by_host),
        "by_severity": {},
        "remediation_plan": [],
    }
    for severity in ["Critical", "High", "Medium", "Low"]:
        count = sum(1 for v in vulns if v["severity"] == severity)
        report["by_severity"][severity] = count

    for host, host_vulns in sorted(by_host.items()):
        report["remediation_plan"].append({
            "host": host,
            "vuln_count": len(host_vulns),
            "critical": sum(1 for v in host_vulns if v["severity"] == "Critical"),
            "patches": [v["name"] for v in host_vulns[:10]],
        })

    if output:
        with open(output, "w") as f:
            json.dump(report, f, indent=2)
    return report
```

## Output Format

```json
{
  "total_vulns": 245,
  "hosts_affected": 42,
  "by_severity": {
    "Critical": 8,
    "High": 35,
    "Medium": 112,
    "Low": 90
  },
  "remediation_plan": [
    {
      "host": "10.0.0.50",
      "vuln_count": 12,
      "critical": 2,
      "patches": ["MS17-010: EternalBlue", "CVE-2024-21887: Ivanti RCE"]
    }
  ]
}
```

## Dependencies

No external packages — Python standard library only.

===== END FILE =====


===== FILE: skills/performing-endpoint-vulnerability-remediation/references/standards.md =====
# Standards & References - Performing Endpoint Vulnerability Remediation

## Primary Standards

### NIST SP 800-40 Rev 4 - Guide to Enterprise Patch Management Planning
- **Publisher**: NIST
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-40/rev-4/final
- **Scope**: Enterprise patch management lifecycle, risk-based prioritization, and operational planning

### CISA Binding Operational Directive 22-01 - Known Exploited Vulnerabilities
- **Publisher**: CISA
- **URL**: https://www.cisa.gov/binding-operational-directive-22-01
- **Scope**: Federal mandate to remediate CVEs in the Known Exploited Vulnerabilities catalog within specified timelines
- **Timelines**: Typically 14 days for internet-facing, 28 days for internal systems

### FIRST CVSS v3.1 Specification
- **Publisher**: FIRST (Forum of Incident Response and Security Teams)
- **URL**: https://www.first.org/cvss/specification-document
- **Scope**: Scoring methodology for vulnerability severity

### FIRST EPSS Model
- **Publisher**: FIRST
- **URL**: https://www.first.org/epss/
- **Scope**: Machine learning model predicting probability of CVE exploitation within 30 days

## Compliance Mappings

| Framework | Requirement | Remediation Coverage |
|-----------|------------|---------------------|
| PCI DSS 4.0 | 6.3.3 - Patch within one month of release | Patch SLA tracking and compliance |
| PCI DSS 4.0 | 11.3.1 - Internal vulnerability scans quarterly | Scan-remediate-validate cycle |
| NIST 800-53 | SI-2 Flaw Remediation | Vulnerability identification and patching |
| NIST 800-53 | RA-5 Vulnerability Monitoring and Scanning | Ongoing scan-remediate process |
| HIPAA | 164.308(a)(1)(ii)(B) - Risk Management | Vulnerability remediation as risk reduction |
| ISO 27001 | A.12.6.1 - Management of technical vulnerabilities | Systematic vulnerability remediation |
| SOC 2 | CC7.1 - Detect and address vulnerabilities | Vulnerability management program |

## Remediation SLA Benchmarks

| Severity | CVSS Range | Industry Standard SLA | CISA KEV Timeline |
|----------|-----------|----------------------|-------------------|
| Critical | 9.0-10.0 | 14 days | Per directive (usually 14 days) |
| High | 7.0-8.9 | 30 days | Per directive |
| Medium | 4.0-6.9 | 60 days | N/A unless in KEV |
| Low | 0.1-3.9 | 90 days | N/A |

## Supporting References

- **CISA KEV Catalog**: https://www.cisa.gov/known-exploited-vulnerabilities-catalog
- **NVD (National Vulnerability Database)**: https://nvd.nist.gov/
- **EPSS Data**: https://api.first.org/data/v1/epss
- **Microsoft Security Update Guide**: https://msrc.microsoft.com/update-guide

===== END FILE =====


===== FILE: skills/performing-endpoint-vulnerability-remediation/references/workflows.md =====
# Workflows - Performing Endpoint Vulnerability Remediation

## Workflow 1: Standard Vulnerability Remediation Cycle

```
[Vulnerability Scan Complete]
    │
    ▼
[Import scan results into tracking system]
    │
    ▼
[Risk-based prioritization]
    │
    ├── CVSS + EPSS + CISA KEV + Asset criticality
    │
    ▼
[Assign priorities: P1/P2/P3/P4]
    │
    ▼
[Identify remediation action per CVE]
    │
    ├── Patch available ──► [Schedule patch deployment]
    ├── Config change needed ──► [Create change request]
    ├── No patch available ──► [Apply workaround/compensating control]
    └── Accept risk ──► [Document with CISO approval]
    │
    ▼
[Test patches in staging environment]
    │
    ▼
[Deploy to production (phased rollout)]
    │
    ▼
[Re-scan to validate remediation]
    │
    ├── Vulnerability closed ──► [Mark resolved in tracker]
    │
    └── Still open ──► [Investigate failure, re-remediate]
```

## Workflow 2: Emergency Zero-Day Response

```
[Zero-day CVE announced (CISA alert / vendor advisory)]
    │
    ▼
[Assess exposure: How many endpoints affected?]
    │
    ▼
[Is patch available?]
    │
    ├── Yes ──► [Emergency patch deployment (skip staging)]
    │               │
    │               ▼
    │          [Monitor for deployment failures]
    │               │
    │               ▼
    │          [Validate patch across fleet]
    │
    └── No ──► [Apply vendor workaround immediately]
                    │
                    ├── Disable vulnerable service/feature
                    ├── Deploy network-level mitigation
                    ├── Create EDR detection rule
                    │
                    ▼
               [Monitor for patch release]
                    │
                    ▼
               [Deploy patch when available]
                    │
                    ▼
               [Remove workaround, validate fix]
```

## Workflow 3: Patch Deployment Pipeline

```
[Patch Tuesday (or vendor release)]
    │
    ▼
[Download and catalog new patches]
    │
    ▼
[Risk assessment: Which patches are critical?]
    │
    ▼
[Deploy to test ring (5% of fleet) - Day 1-3]
    │
    ├── Test application compatibility
    ├── Monitor for BSOD, crashes, performance issues
    │
    ▼
[Deploy to pilot ring (20% of fleet) - Day 4-7]
    │
    ├── Broader application testing
    ├── User feedback collection
    │
    ▼
[Deploy to production ring (remaining fleet) - Day 8-14]
    │
    ▼
[Generate compliance report]
    │
    ├── Endpoints patched: X%
    ├── Pending reboot: Y
    └── Failed deployments: Z (investigate)
```

## Workflow 4: SLA Compliance Tracking

```
[Weekly SLA Review]
    │
    ▼
[Query open vulnerabilities grouped by SLA status]
    │
    ├── Within SLA ──► [Track progress, no action needed]
    │
    ├── Approaching SLA (7 days) ──► [Escalate to endpoint team]
    │
    └── Overdue (past SLA) ──► [Escalate to management]
                                     │
                                     ├── Remediation feasible ──► [Emergency remediation]
                                     │
                                     └── Blocked (dependency) ──► [Document exception, compensating control]
```

===== END FILE =====


===== FILE: skills/performing-endpoint-vulnerability-remediation/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing endpoint vulnerability remediation tracking and validation."""

import json
import argparse
import subprocess
import csv
from datetime import datetime


def parse_scan_report(csv_file):
    """Parse a vulnerability scan CSV report and prioritize remediation."""
    with open(csv_file, "r", encoding="utf-8", errors="replace") as f:
        reader = csv.DictReader(f)
        rows = list(reader)
    vulns = []
    for row in rows:
        severity = row.get("Severity", row.get("severity", row.get("Risk", ""))).lower()
        vulns.append({
            "host": row.get("Host", row.get("IP", row.get("ip", ""))),
            "port": row.get("Port", row.get("port", "")),
            "cve": row.get("CVE", row.get("cve", row.get("Plugin ID", ""))),
            "title": row.get("Name", row.get("title", row.get("Summary", "")))[:200],
            "severity": severity,
            "solution": row.get("Solution", row.get("Fix", ""))[:300],
        })
    by_severity = {}
    for v in vulns:
        s = v["severity"]
        by_severity[s] = by_severity.get(s, 0) + 1
    critical_high = [v for v in vulns if v["severity"] in ("critical", "high")]
    return {
        "total_vulns": len(vulns),
        "by_severity": by_severity,
        "critical_high_count": len(critical_high),
        "remediation_queue": sorted(critical_high, key=lambda x: 0 if x["severity"] == "critical" else 1),
    }


def check_windows_patches():
    """Check installed Windows patches and identify missing ones."""
    try:
        result = subprocess.run(
            ["wmic", "qfe", "get", "HotFixID,InstalledOn,Description", "/format:csv"],
            capture_output=True, text=True, timeout=30
        )
        from io import StringIO
        reader = csv.DictReader(StringIO(result.stdout))
        patches = [{"id": r.get("HotFixID"), "date": r.get("InstalledOn"), "desc": r.get("Description")}
                    for r in reader if r.get("HotFixID")]
        return {"installed_patches": len(patches), "patches": patches}
    except Exception as e:
        return {"error": str(e)}


def validate_remediation(host, port, check_type="port_open"):
    """Validate that a vulnerability has been remediated."""
    import socket
    result = {"host": host, "port": int(port), "check_type": check_type}
    if check_type == "port_open":
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(5)
        try:
            sock.connect((host, int(port)))
            result["status"] = "STILL_OPEN"
            result["remediated"] = False
        except (socket.timeout, ConnectionRefusedError, OSError):
            result["status"] = "CLOSED"
            result["remediated"] = True
        finally:
            sock.close()
    return result


def generate_remediation_report(scan_file, output=None):
    """Generate a remediation plan from scan results."""
    parsed = parse_scan_report(scan_file)
    plan = {"generated": datetime.utcnow().isoformat(), "source": scan_file}
    plan["summary"] = parsed["by_severity"]
    plan["total"] = parsed["total_vulns"]
    hosts = {}
    for v in parsed["remediation_queue"]:
        h = v["host"]
        if h not in hosts:
            hosts[h] = []
        hosts[h].append(v)
    plan["by_host"] = {h: {"count": len(vs), "vulns": vs} for h, vs in hosts.items()}
    if output:
        with open(output, "w") as f:
            json.dump(plan, f, indent=2)
    return plan


def main():
    parser = argparse.ArgumentParser(description="Endpoint Vulnerability Remediation Agent")
    sub = parser.add_subparsers(dest="command")
    p = sub.add_parser("parse", help="Parse vulnerability scan report")
    p.add_argument("--scan-file", required=True)
    sub.add_parser("patches", help="Check installed Windows patches")
    v = sub.add_parser("validate", help="Validate remediation")
    v.add_argument("--host", required=True)
    v.add_argument("--port", required=True)
    r = sub.add_parser("report", help="Generate remediation report")
    r.add_argument("--scan-file", required=True)
    r.add_argument("--output", help="Output JSON file")
    args = parser.parse_args()
    if args.command == "parse":
        result = parse_scan_report(args.scan_file)
    elif args.command == "patches":
        result = check_windows_patches()
    elif args.command == "validate":
        result = validate_remediation(args.host, args.port)
    elif args.command == "report":
        result = generate_remediation_report(args.scan_file, args.output)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-endpoint-vulnerability-remediation/scripts/process.py =====
#!/usr/bin/env python3
"""
Endpoint Vulnerability Remediation Prioritizer

Parses vulnerability scan exports (Nessus CSV), enriches with EPSS scores,
cross-references CISA KEV catalog, and generates prioritized remediation plans.
"""

import json
import csv
import sys
import os
from datetime import datetime, timedelta
from collections import defaultdict
from urllib.request import urlopen
from urllib.error import URLError


CISA_KEV_URL = "https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json"
EPSS_API_URL = "https://api.first.org/data/v1/epss"


def load_nessus_csv(csv_path: str) -> list:
    """Parse Nessus scan export CSV into vulnerability records."""
    vulns = []

    with open(csv_path, "r", encoding="utf-8-sig") as f:
        reader = csv.DictReader(f)
        for row in reader:
            cvss = 0.0
            try:
                cvss = float(row.get("CVSS v3.0 Base Score", "0") or
                             row.get("CVSS v2.0 Base Score", "0") or "0")
            except ValueError:
                pass

            cve_list = [c.strip() for c in row.get("CVE", "").split(",") if c.strip()]

            vuln = {
                "plugin_id": row.get("Plugin ID", ""),
                "plugin_name": row.get("Name", ""),
                "severity": row.get("Risk", "None"),
                "host": row.get("Host", ""),
                "protocol": row.get("Protocol", ""),
                "port": row.get("Port", ""),
                "cvss_score": cvss,
                "cves": cve_list,
                "solution": row.get("Solution", ""),
                "synopsis": row.get("Synopsis", ""),
                "description": row.get("Description", "")[:200],
            }
            vulns.append(vuln)

    return vulns


def fetch_cisa_kev() -> set:
    """Fetch CISA Known Exploited Vulnerabilities catalog."""
    try:
        with urlopen(CISA_KEV_URL, timeout=15) as resp:
            data = json.loads(resp.read())
            return {v["cveID"] for v in data.get("vulnerabilities", [])}
    except (URLError, json.JSONDecodeError, KeyError) as e:
        print(f"Warning: Could not fetch CISA KEV catalog: {e}")
        return set()


def fetch_epss_scores(cves: list) -> dict:
    """Fetch EPSS scores for a list of CVEs."""
    epss_scores = {}
    batch_size = 100

    for i in range(0, len(cves), batch_size):
        batch = cves[i:i + batch_size]
        cve_param = ",".join(batch)
        url = f"{EPSS_API_URL}?cve={cve_param}"
        try:
            with urlopen(url, timeout=15) as resp:
                data = json.loads(resp.read())
                for item in data.get("data", []):
                    epss_scores[item["cve"]] = float(item.get("epss", 0))
        except (URLError, json.JSONDecodeError) as e:
            print(f"Warning: EPSS lookup failed for batch: {e}")

    return epss_scores


def prioritize_vulnerabilities(vulns: list, kev_cves: set, epss_scores: dict) -> list:
    """Assign remediation priority based on risk scoring."""
    for vuln in vulns:
        cvss = vuln["cvss_score"]
        cves = vuln["cves"]

        in_kev = any(cve in kev_cves for cve in cves)
        max_epss = max((epss_scores.get(cve, 0) for cve in cves), default=0)

        vuln["in_cisa_kev"] = in_kev
        vuln["epss_score"] = round(max_epss, 4)

        if cvss >= 9.0 or in_kev or (cvss >= 7.0 and max_epss > 0.7):
            vuln["priority"] = "P1"
            vuln["sla_days"] = 14
        elif cvss >= 7.0 and max_epss > 0.5:
            vuln["priority"] = "P2"
            vuln["sla_days"] = 30
        elif cvss >= 4.0:
            vuln["priority"] = "P3"
            vuln["sla_days"] = 60
        else:
            vuln["priority"] = "P4"
            vuln["sla_days"] = 90

        vuln["sla_deadline"] = (
            datetime.utcnow() + timedelta(days=vuln["sla_days"])
        ).strftime("%Y-%m-%d")

    vulns.sort(key=lambda v: (
        {"P1": 0, "P2": 1, "P3": 2, "P4": 3}.get(v["priority"], 4),
        -v["cvss_score"],
        -v["epss_score"],
    ))

    return vulns


def generate_remediation_plan(vulns: list, output_path: str) -> None:
    """Generate prioritized remediation plan in JSON."""
    summary = defaultdict(int)
    hosts_affected = defaultdict(set)

    for vuln in vulns:
        summary[vuln["priority"]] += 1
        hosts_affected[vuln["priority"]].add(vuln["host"])

    plan = {
        "report_generated": datetime.utcnow().isoformat() + "Z",
        "summary": {
            "total_vulnerabilities": len(vulns),
            "by_priority": dict(summary),
            "unique_hosts": {p: len(h) for p, h in hosts_affected.items()},
        },
        "kev_vulnerabilities": [v for v in vulns if v.get("in_cisa_kev")],
        "remediation_queue": [
            {
                "priority": v["priority"],
                "sla_deadline": v["sla_deadline"],
                "host": v["host"],
                "plugin_name": v["plugin_name"],
                "cvss": v["cvss_score"],
                "epss": v["epss_score"],
                "cisa_kev": v["in_cisa_kev"],
                "cves": v["cves"],
                "solution": v["solution"],
            }
            for v in vulns
        ],
    }

    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(plan, f, indent=2)


def export_remediation_csv(vulns: list, output_path: str) -> None:
    """Export remediation plan to CSV for team assignment."""
    with open(output_path, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow([
            "Priority", "SLA Deadline", "Host", "Vulnerability",
            "CVSS", "EPSS", "CISA KEV", "CVEs", "Solution",
            "Status", "Assigned To", "Notes",
        ])
        for v in vulns:
            writer.writerow([
                v["priority"], v["sla_deadline"], v["host"],
                v["plugin_name"], v["cvss_score"], v["epss_score"],
                v["in_cisa_kev"], "; ".join(v["cves"]),
                v["solution"][:200], "Open", "", "",
            ])


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python process.py <nessus_export.csv>")
        print()
        print("Parses Nessus CSV export, enriches with EPSS/CISA KEV data,")
        print("and generates a prioritized remediation plan.")
        sys.exit(1)

    csv_path = sys.argv[1]
    if not os.path.exists(csv_path):
        print(f"Error: File not found: {csv_path}")
        sys.exit(1)

    print("Parsing vulnerability scan results...")
    vulns = load_nessus_csv(csv_path)
    print(f"Loaded {len(vulns)} vulnerability findings")

    all_cves = list({cve for v in vulns for cve in v["cves"]})
    print(f"Unique CVEs: {len(all_cves)}")

    print("Fetching CISA KEV catalog...")
    kev_cves = fetch_cisa_kev()
    print(f"KEV catalog contains {len(kev_cves)} CVEs")

    print("Fetching EPSS scores...")
    epss_scores = fetch_epss_scores(all_cves) if all_cves else {}
    print(f"Retrieved EPSS scores for {len(epss_scores)} CVEs")

    print("Prioritizing vulnerabilities...")
    vulns = prioritize_vulnerabilities(vulns, kev_cves, epss_scores)

    base = os.path.splitext(os.path.basename(csv_path))[0]
    out_dir = os.path.dirname(csv_path) or "."

    plan_path = os.path.join(out_dir, f"{base}_remediation_plan.json")
    generate_remediation_plan(vulns, plan_path)
    print(f"Remediation plan: {plan_path}")

    csv_out = os.path.join(out_dir, f"{base}_remediation.csv")
    export_remediation_csv(vulns, csv_out)
    print(f"Remediation CSV: {csv_out}")

    print(f"\n--- Remediation Summary ---")
    for p in ["P1", "P2", "P3", "P4"]:
        count = sum(1 for v in vulns if v["priority"] == p)
        if count:
            print(f"  {p}: {count} vulnerabilities")

    kev_count = sum(1 for v in vulns if v.get("in_cisa_kev"))
    if kev_count:
        print(f"\n  CISA KEV matches: {kev_count} (mandatory remediation)")

===== END FILE =====
