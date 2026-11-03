

===== FILE: skills/implementing-zero-trust-with-beyondcorp/SKILL.md =====
---
name: implementing-zero-trust-with-beyondcorp
description: Configures Google BeyondCorp Enterprise Identity-Aware Proxy (IAP) as the access enforcement point for web applications, defining Access Context Manager access levels from device trust and network attributes, and auditing the resulting policies for compliance. Use when eliminating perimeter/VPN trust for GCP resources or internal apps, or when setting up identity- and device-posture-based access controls on Google Cloud.
domain: cybersecurity
subdomain: zero-trust
tags:
- zero-trust
- beyondcorp
- google-cloud
- iap
- context-aware-access
- device-trust
- identity
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
mitre_attack:
- T1078
- T1550
- T1021
- T1556
- T1078.004
---

# Implementing Zero Trust with BeyondCorp

## Overview

Google BeyondCorp Enterprise implements the zero trust security model by eliminating the concept of a trusted network perimeter. Instead of relying on VPNs and network location, BeyondCorp authenticates and authorizes every request based on user identity, device posture, and contextual attributes. Identity-Aware Proxy (IAP) serves as the enforcement point, intercepting all requests to protected resources and evaluating them against Access Context Manager policies. This skill covers configuring IAP for web applications, defining access levels based on device trust and network attributes, and auditing access policies for compliance.


## When to Use

- When deploying or configuring implementing zero trust with beyondcorp capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Google Cloud project with BeyondCorp Enterprise license
- IAP API enabled (iap.googleapis.com)
- Access Context Manager API enabled (accesscontextmanager.googleapis.com)
- GCP resources to protect (Compute Engine, App Engine, or GKE services)
- Endpoint Verification deployed on managed devices
- Python 3.9+ with google-cloud-iap library

## Steps

### Step 1: Enable IAP on Target Resources
Configure Identity-Aware Proxy on Compute Engine, App Engine, or HTTPS load balancer backends.

### Step 2: Define Access Levels
Create Access Context Manager access levels based on IP ranges, device attributes (OS version, encryption, screen lock), and geographic location.

### Step 3: Bind Access Policies
Apply access levels as IAP conditions to enforce context-aware access decisions on protected resources.

### Step 4: Audit and Monitor
Query IAP audit logs, verify policy enforcement, and identify gaps in zero trust coverage.

## Expected Output

JSON report containing IAP-protected resources, access level definitions, policy binding audit results, and zero trust coverage metrics.

===== END FILE =====


===== FILE: skills/implementing-zero-trust-with-beyondcorp/LICENSE =====

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


===== FILE: skills/implementing-zero-trust-with-beyondcorp/references/api-reference.md =====
# API Reference: Implementing Zero Trust with BeyondCorp

## gcloud IAP Commands

```bash
# Enable IAP on backend service
gcloud iap web enable --resource-type=backend-services \
  --service=my-backend --project=my-project

# Get IAP IAM policy
gcloud iap web get-iam-policy --project=my-project

# Grant IAP access with access level condition
gcloud iap web add-iam-policy-binding --project=my-project \
  --member="group:team@example.com" \
  --role="roles/iap.httpsResourceAccessor" \
  --condition="expression=accessPolicies/123/accessLevels/corp_device,title=CorpDevice"

# Enable required APIs
gcloud services enable iap.googleapis.com
gcloud services enable accesscontextmanager.googleapis.com
gcloud services enable beyondcorp.googleapis.com
```

## Access Context Manager Commands

```bash
# Create access policy
gcloud access-context-manager policies create --organization=ORG_ID --title="Corp Policy"

# Create access level (device + IP)
gcloud access-context-manager levels create corp_trusted \
  --policy=POLICY_ID --title="Corporate Trusted" \
  --basic-level-spec=level_spec.yaml

# List access levels
gcloud access-context-manager levels list --policy=POLICY_ID --format=json
```

## Access Level Spec (YAML)

```yaml
conditions:
  - ipSubnetworks:
      - "10.0.0.0/8"
      - "172.16.0.0/12"
    devicePolicy:
      requireScreenlock: true
      osConstraints:
        - osType: DESKTOP_WINDOWS
          minimumVersion: "10.0.19041"
        - osType: DESKTOP_MAC
          minimumVersion: "12.0.0"
      allowedEncryptionStatuses:
        - ENCRYPTED
    regions:
      - "US"
      - "GB"
```

## IAP Roles

| Role | Description |
|------|-------------|
| roles/iap.httpsResourceAccessor | Access IAP-protected resources |
| roles/iap.admin | Full IAP administration |
| roles/iap.settingsAdmin | Modify IAP settings |
| roles/iap.tunnelResourceAccessor | Access via IAP TCP tunneling |

## Python SDK

```python
from google.cloud import iap_v1
client = iap_v1.IdentityAwareProxyAdminServiceClient()
# List tunnel destinations
request = iap_v1.ListTunnelDestGroupsRequest(parent=f"projects/{project}/iap_tunnel/locations/-")
```

## Audit Log Query (Cloud Logging)

```
resource.type="gce_backend_service"
logName="projects/PROJECT/logs/cloudaudit.googleapis.com%2Fdata_access"
protoPayload.methodName="AuthorizeUser"
protoPayload.authenticationInfo.principalEmail!=""
```

### References

- BeyondCorp Enterprise: https://cloud.google.com/beyondcorp
- IAP Concepts: https://cloud.google.com/iap/docs/concepts-overview
- Access Context Manager: https://cloud.google.com/access-context-manager/docs

===== END FILE =====


===== FILE: skills/implementing-zero-trust-with-beyondcorp/scripts/agent.py =====
#!/usr/bin/env python3
"""BeyondCorp Zero Trust Agent - audits IAP configuration, access levels, and policy bindings."""

import json
import argparse
import logging
import subprocess
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def gcloud_json(args_list):
    """Execute gcloud command and return JSON output."""
    cmd = ["gcloud"] + args_list + ["--format=json"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
    return json.loads(result.stdout) if result.returncode == 0 and result.stdout else []


def list_iap_protected_resources(project):
    """List resources protected by Identity-Aware Proxy."""
    backends = gcloud_json(["compute", "backend-services", "list", "--project", project])
    protected = []
    for backend in backends:
        iap = backend.get("iap", {})
        protected.append({
            "name": backend.get("name", ""),
            "iap_enabled": iap.get("enabled", False),
            "oauth2_client_id": bool(iap.get("oauth2ClientId", "")),
        })
    return protected


def get_access_levels(policy_name):
    """Retrieve Access Context Manager access levels."""
    levels = gcloud_json(["access-context-manager", "levels", "list", "--policy", policy_name])
    parsed = []
    for level in levels:
        basic = level.get("basic", {})
        conditions = basic.get("conditions", [])
        parsed.append({
            "name": level.get("name", "").split("/")[-1],
            "title": level.get("title", ""),
            "combining_function": basic.get("combiningFunction", "AND"),
            "condition_count": len(conditions),
            "has_ip_restriction": any(c.get("ipSubnetworks") for c in conditions),
            "has_device_policy": any(c.get("devicePolicy") for c in conditions),
            "has_region_restriction": any(c.get("regions") for c in conditions),
        })
    return parsed


def audit_iap_iam_bindings(project):
    """Audit IAM bindings on IAP-protected resources."""
    findings = []
    cmd = ["gcloud", "iap", "web", "get-iam-policy", "--project", project, "--format=json"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
    if result.returncode != 0:
        return [{"issue": "Cannot retrieve IAP IAM policy", "severity": "high"}]
    policy = json.loads(result.stdout) if result.stdout else {}
    for binding in policy.get("bindings", []):
        role = binding.get("role", "")
        members = binding.get("members", [])
        condition = binding.get("condition")
        if role == "roles/iap.httpsResourceAccessor":
            if "allUsers" in members or "allAuthenticatedUsers" in members:
                findings.append({
                    "role": role, "issue": "Public access via allUsers/allAuthenticatedUsers",
                    "severity": "critical",
                    "recommendation": "Restrict to specific user/group identities",
                })
            if not condition:
                findings.append({
                    "role": role, "members": members[:5],
                    "issue": "IAP binding without access level condition",
                    "severity": "high",
                    "recommendation": "Add access level condition for context-aware enforcement",
                })
    return findings


def audit_access_level_strength(access_levels):
    """Audit access levels for security strength."""
    findings = []
    for level in access_levels:
        if not level["has_device_policy"]:
            findings.append({
                "access_level": level["name"],
                "issue": "No device policy requirement",
                "severity": "medium",
                "recommendation": "Add device trust requirements (encryption, screen lock, OS version)",
            })
        if not level["has_ip_restriction"] and not level["has_region_restriction"]:
            findings.append({
                "access_level": level["name"],
                "issue": "No network or geographic restriction",
                "severity": "medium",
                "recommendation": "Consider adding corporate IP range or geo restrictions",
            })
        if level["condition_count"] == 0:
            findings.append({
                "access_level": level["name"],
                "issue": "Empty access level with no conditions",
                "severity": "high",
            })
    return findings


def check_endpoint_verification(project):
    """Check Endpoint Verification deployment status."""
    devices = gcloud_json(["endpoint-verification", "list", "--project", project])
    total = len(devices)
    compliant = sum(1 for d in devices if d.get("complianceState") == "COMPLIANT")
    return {
        "total_devices": total,
        "compliant": compliant,
        "non_compliant": total - compliant,
        "compliance_rate": round(compliant / max(total, 1) * 100, 1),
    }


def generate_report(protected, access_levels, iam_findings, level_findings, endpoint_status):
    iap_enabled = sum(1 for r in protected if r["iap_enabled"])
    all_findings = iam_findings + level_findings
    return {
        "timestamp": datetime.utcnow().isoformat(),
        "framework": "BeyondCorp Enterprise / Zero Trust",
        "iap_protected_resources": len(protected),
        "iap_enabled_count": iap_enabled,
        "iap_coverage": round(iap_enabled / max(len(protected), 1) * 100, 1),
        "access_levels_defined": len(access_levels),
        "access_level_details": access_levels,
        "endpoint_verification": endpoint_status,
        "iam_findings": iam_findings,
        "access_level_findings": level_findings,
        "total_findings": len(all_findings),
        "critical_findings": sum(1 for f in all_findings if f.get("severity") == "critical"),
    }


def main():
    parser = argparse.ArgumentParser(description="BeyondCorp Zero Trust Audit Agent")
    parser.add_argument("--project", required=True, help="GCP project ID")
    parser.add_argument("--access-policy", required=True, help="Access Context Manager policy ID")
    parser.add_argument("--output", default="beyondcorp_audit_report.json")
    args = parser.parse_args()

    protected = list_iap_protected_resources(args.project)
    access_levels = get_access_levels(args.access_policy)
    iam_findings = audit_iap_iam_bindings(args.project)
    level_findings = audit_access_level_strength(access_levels)
    endpoint_status = check_endpoint_verification(args.project)
    report = generate_report(protected, access_levels, iam_findings, level_findings, endpoint_status)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("BeyondCorp: %.1f%% IAP coverage, %d access levels, %d findings",
                report["iap_coverage"], report["access_levels_defined"], report["total_findings"])
    print(json.dumps(report, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====
