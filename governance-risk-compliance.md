

===== FILE: skills/performing-soc2-type2-audit-preparation/SKILL.md =====
---
name: performing-soc2-type2-audit-preparation
description: 'Automates SOC 2 Type II audit preparation including gap assessment against
  AICPA Trust Services Criteria (CC1-CC9), evidence collection from cloud providers
  and identity systems, control testing validation, remediation tracking, and continuous
  compliance monitoring. Covers all five TSC categories (Security, Availability, Processing
  Integrity, Confidentiality, Privacy) with automated evidence gathering from AWS,
  Azure, GCP, Okta, GitHub, and Jira. Use when preparing for or maintaining SOC 2
  Type II certification.

  '
domain: cybersecurity
subdomain: governance-risk-compliance
tags:
- soc2
- compliance
- grc
- aicpa-tsc
- audit-preparation
- governance-risk-compliance
version: '1.0'
author: mukul975
license: Apache-2.0
nist_csf:
- GV.OC-01
- GV.RM-01
- GV.PO-01
- GV.OV-01
mitre_attack:
- T1078
- T1190
- T1059
- T1071
- T1095
---

# Performing SOC 2 Type II Audit Preparation

## When to Use

- When preparing for a SOC 2 Type II audit engagement with a CPA firm
- When conducting a gap assessment against AICPA Trust Services Criteria
- When automating evidence collection across cloud infrastructure and identity providers
- When validating that controls have operated effectively over the audit period (3-12 months)
- When building continuous compliance monitoring to maintain SOC 2 posture between audits
- When remediating control gaps identified during readiness assessment

## Prerequisites

- Familiarity with AICPA Trust Services Criteria (CC1-CC9)
- Access to cloud provider APIs (AWS, Azure, or GCP) with read-only permissions
- Access to identity provider (Okta, Azure AD, Google Workspace)
- Access to version control system (GitHub, GitLab)
- Access to ticketing system (Jira, Linear, ServiceNow)
- Python 3.8+ with `boto3`, `requests`, `pyyaml` dependencies
- Appropriate authorization to collect compliance evidence

## Instructions

### 1. Understand the Trust Services Criteria

SOC 2 is built on five Trust Services Categories defined by AICPA. Security (Common Criteria CC1-CC9) is mandatory; the others are selected based on business relevance:

| Category | Criteria | Focus |
|----------|----------|-------|
| Security (mandatory) | CC1-CC9 | Control environment, risk, access, operations, change management |
| Availability | A1 | System uptime and disaster recovery |
| Processing Integrity | PI1 | Accurate and complete data processing |
| Confidentiality | C1 | Protection of confidential information |
| Privacy | P1-P8 | Personal information lifecycle |

### 2. Common Criteria Breakdown (CC1-CC9)

**CC1 - Control Environment:** Board oversight, management structure, integrity and ethical values, HR policies, accountability.

**CC2 - Communication and Information:** Internal/external communication of security policies, system boundaries, roles, and responsibilities.

**CC3 - Risk Assessment:** Risk identification, fraud risk analysis, change impact assessment, risk tolerance definition.

**CC4 - Monitoring Activities:** Ongoing control evaluations, deficiency identification, remediation tracking, internal audit.

**CC5 - Control Activities:** Policy-to-procedure mapping, technology controls, deployment of controls across the entity.

**CC6 - Logical and Physical Access Controls:** Authentication, authorization, access provisioning/deprovisioning, physical security, encryption.

**CC7 - System Operations:** Anomaly detection, incident response, vulnerability management, change detection, event monitoring.

**CC8 - Change Management:** Change authorization, testing, approval workflows, emergency changes, rollback procedures.

**CC9 - Risk Mitigation:** Vendor risk management, business continuity, insurance, residual risk acceptance.

### 3. Conduct Gap Assessment

Before the audit period begins, perform a readiness assessment 8-12 weeks in advance:

```python
# Define control matrix against CC criteria
gap_assessment = {
    "CC1": {
        "CC1.1": {
            "criteria": "COSO Principle 1: Demonstrates commitment to integrity",
            "control": "Code of conduct signed annually by all employees",
            "evidence": "Signed acknowledgments in HR system",
            "status": "implemented",
            "gap": None,
        },
        "CC1.2": {
            "criteria": "COSO Principle 2: Board exercises oversight",
            "control": "Quarterly board security reviews",
            "evidence": "Board meeting minutes with security agenda items",
            "status": "partial",
            "gap": "No documented security committee charter",
        },
    },
}
```

### 4. Automate Evidence Collection

Collect evidence continuously throughout the audit period from integrated systems:

```python
import boto3

# CC6 Evidence: AWS IAM access controls
iam = boto3.client("iam")

# Collect MFA status for all IAM users
users = iam.list_users()["Users"]
mfa_evidence = []
for user in users:
    mfa_devices = iam.list_mfa_devices(UserName=user["UserName"])
    mfa_evidence.append({
        "user": user["UserName"],
        "mfa_enabled": len(mfa_devices["MFADevices"]) > 0,
        "created": user["CreateDate"].isoformat(),
    })

# CC7 Evidence: AWS CloudTrail logging status
cloudtrail = boto3.client("cloudtrail")
trails = cloudtrail.describe_trails()["trailList"]
logging_evidence = []
for trail in trails:
    status = cloudtrail.get_trail_status(Name=trail["TrailARN"])
    logging_evidence.append({
        "trail": trail["Name"],
        "is_logging": status["IsLogging"],
        "multi_region": trail.get("IsMultiRegionTrail", False),
        "log_validation": trail.get("LogFileValidationEnabled", False),
    })
```

### 5. Validate Control Effectiveness

For Type II audits, demonstrate controls operated effectively over the entire audit period:

```python
import requests

# CC8 Evidence: Change management - verify all production changes
# had tickets, approvals, and testing before deployment
headers = {"Authorization": f"token {github_token}"}
prs = requests.get(
    "https://api.github.com/repos/org/repo/pulls",
    params={"state": "closed", "base": "main", "per_page": 100},
    headers=headers,
).json()

change_evidence = []
for pr in prs:
    if not pr.get("merged_at"):
        continue
    reviews = requests.get(pr["url"] + "/reviews", headers=headers).json()
    approved = any(r["state"] == "APPROVED" for r in reviews)
    change_evidence.append({
        "pr_number": pr["number"],
        "title": pr["title"],
        "merged_at": pr["merged_at"],
        "approved": approved,
    })

# Flag PRs merged without approval (control exception)
exceptions = [c for c in change_evidence if not c["approved"]]
```

### 6. Continuous Compliance Monitoring

Set up automated checks that run daily to detect control drift:

```python
# Daily compliance check - run via cron or Lambda
checks = [
    {"control": "CC6.1", "check": "All IAM users have MFA enabled"},
    {"control": "CC6.6", "check": "No public S3 buckets"},
    {"control": "CC7.1", "check": "CloudTrail logging enabled"},
    {"control": "CC7.2", "check": "GuardDuty findings under threshold"},
    {"control": "CC8.1", "check": "All PRs have required reviews"},
]

for check in checks:
    result = run_compliance_check(check["control"])
    if not result["passing"]:
        send_alert(
            channel="#compliance",
            message=f"Control drift: {check['control']} - {check['check']}",
            details=result["findings"],
        )
```

### 7. Prepare Evidence Packages for Auditors

Organize collected evidence into structured packages per criteria:

```python
evidence_package = {
    "audit_period": {"start": "2025-04-01", "end": "2026-03-31"},
    "criteria_packages": {
        "CC1_Control_Environment": {
            "CC1.1": ["signed_acknowledgments.csv"],
            "CC1.2": ["board_minutes_q1.pdf", "board_minutes_q2.pdf"],
        },
        "CC6_Logical_Physical_Access": {
            "CC6.1": ["okta_mfa_policy.json", "iam_users_mfa_status.csv"],
            "CC6.2": ["access_review_q1.csv", "access_review_q2.csv"],
            "CC6.3": ["offboarding_tickets.csv", "terminated_user_audit.csv"],
        },
        "CC7_System_Operations": {
            "CC7.1": ["cloudtrail_config.json", "siem_dashboard.png"],
            "CC7.2": ["guardduty_findings_summary.csv"],
            "CC7.3": ["vulnerability_scan_reports/"],
        },
        "CC8_Change_Management": {
            "CC8.1": ["merged_prs_with_approvals.csv"],
        },
    },
}
```

## Examples

### Automated Access Review for CC6.2

```python
import boto3
from datetime import datetime, timedelta

iam = boto3.client("iam")

# Find users with no activity in 90 days
inactive_threshold = datetime.utcnow() - timedelta(days=90)
report = iam.get_credential_report()["Content"].decode()

inactive_users = []
for line in report.strip().split("\n")[1:]:
    fields = line.split(",")
    username = fields[0]
    last_used = fields[4]
    if last_used not in ("N/A", "no_information"):
        last_date = datetime.strptime(last_used, "%Y-%m-%dT%H:%M:%S+00:00")
        if last_date < inactive_threshold:
            inactive_users.append({"user": username, "last_active": last_used})
```

### Vulnerability Management Evidence for CC7.2

```python
import requests

headers = {"Authorization": f"Bearer {scanner_token}"}
scans = requests.get(
    "https://scanner.example.com/api/v1/scans",
    params={"status": "completed", "since": "2025-04-01"},
    headers=headers,
).json()

vuln_evidence = {"scan_count": len(scans), "critical_findings": 0, "high_findings": 0}
for scan in scans:
    findings = requests.get(
        f"https://scanner.example.com/api/v1/scans/{scan['id']}/findings",
        headers=headers,
    ).json()
    vuln_evidence["critical_findings"] += len([f for f in findings if f["severity"] == "critical"])
    vuln_evidence["high_findings"] += len([f for f in findings if f["severity"] == "high"])
```

### Incident Response Evidence for CC7.3

```python
incidents = requests.get(
    "https://pagerduty.com/api/v1/incidents",
    params={"since": "2025-04-01", "until": "2026-03-31"},
    headers={"Authorization": f"Token token={pd_token}"},
).json()

ir_evidence = {
    "total_incidents": len(incidents["incidents"]),
    "incidents": [
        {
            "id": inc["id"],
            "title": inc["title"],
            "severity": inc["urgency"],
            "created": inc["created_at"],
            "resolved": inc.get("last_status_change_at"),
        }
        for inc in incidents["incidents"]
    ],
}
```

===== END FILE =====


===== FILE: skills/performing-soc2-type2-audit-preparation/LICENSE =====

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


===== FILE: skills/performing-soc2-type2-audit-preparation/assets/template.md =====
# SOC 2 Type II Audit Preparation Checklist

## Organization Information
| Field | Value |
|-------|-------|
| Organization Name | |
| Audit Period Start | |
| Audit Period End | |
| Audit Firm | |
| TSC Categories In Scope | Security / Availability / Processing Integrity / Confidentiality / Privacy |
| Preparation Lead | |
| Assessment Date | |

---

## Phase 1: Scoping

### TSC Category Selection
- [ ] Security (Common Criteria) - Mandatory
- [ ] Availability - Include if uptime SLAs exist
- [ ] Processing Integrity - Include if data processing services provided
- [ ] Confidentiality - Include if confidential data handled
- [ ] Privacy - Include if personal information processed

### System Boundaries
- [ ] In-scope services identified
- [ ] Infrastructure components documented (servers, databases, networks)
- [ ] Cloud services and regions listed
- [ ] Data flows mapped (input, processing, storage, output)
- [ ] Subservice organizations identified (e.g., AWS, Azure, GCP)
- [ ] Carve-out vs. inclusive method determined for subservice orgs
- [ ] Out-of-scope systems and justifications documented

### Audit Firm Selection
- [ ] CPA firm selected with SOC 2 experience
- [ ] Engagement letter signed
- [ ] Audit timeline agreed upon
- [ ] Fee structure confirmed
- [ ] Point of contact established

---

## Phase 2: Control Design

### Control Matrix
- [ ] All applicable TSC criteria identified
- [ ] Existing controls inventoried and mapped to criteria
- [ ] Control gaps identified and new controls designed
- [ ] Each control has:
  - [ ] Unique control ID
  - [ ] Clear description of control activity
  - [ ] Mapped TSC criteria
  - [ ] Control type (Preventive/Detective/Corrective)
  - [ ] Frequency (Continuous/Daily/Weekly/Monthly/Quarterly/Annual)
  - [ ] Assigned owner
  - [ ] Defined evidence type
- [ ] Coverage verified: every TSC criterion mapped to at least one control

### Key Controls by Series

#### CC1 - Control Environment
- [ ] Code of conduct/ethics policy published
- [ ] Board/management oversight documented
- [ ] Organizational chart with security roles
- [ ] Background check process for new hires
- [ ] Performance evaluation includes security responsibilities

#### CC2 - Communication and Information
- [ ] Security policies communicated to employees
- [ ] External security commitments documented (SLAs, contracts)
- [ ] Incident notification procedures defined

#### CC3 - Risk Assessment
- [ ] Annual risk assessment completed
- [ ] Risk register maintained
- [ ] Fraud risk assessment performed

#### CC4 - Monitoring Activities
- [ ] Internal control testing programme
- [ ] Control deficiency tracking and remediation

#### CC5 - Control Activities
- [ ] Information security policies approved and current
- [ ] Procedures documented for key processes

#### CC6 - Logical and Physical Access
- [ ] MFA enabled for all production and sensitive systems
- [ ] Role-based access control (RBAC) implemented
- [ ] Quarterly user access reviews performed
- [ ] Access provisioning/deprovisioning process documented
- [ ] Terminated user access removed within 24 hours
- [ ] Physical access restricted to authorized personnel
- [ ] Encryption at rest (AES-256) and in transit (TLS 1.2+)
- [ ] Endpoint protection deployed to all devices
- [ ] Firewall and network segmentation configured

#### CC7 - System Operations
- [ ] SIEM deployed with alerting rules
- [ ] 24/7 monitoring capability (or justified exception)
- [ ] Vulnerability scanning (weekly/monthly)
- [ ] Annual penetration testing by third party
- [ ] Incident response plan documented and tested
- [ ] Security event evaluation procedures

#### CC8 - Change Management
- [ ] Change management policy and process documented
- [ ] Changes require peer review and management approval
- [ ] Separation of duties between development and deployment
- [ ] Emergency change process defined

#### CC9 - Risk Mitigation
- [ ] Vendor management programme implemented
- [ ] Critical vendor SOC reports reviewed annually
- [ ] Vendor security assessments performed
- [ ] Insurance coverage evaluated

---

## Phase 3: Evidence Collection

### Evidence Repository
- [ ] Centralized evidence storage established
- [ ] Folder structure organized by TSC criterion
- [ ] Naming convention defined and communicated
- [ ] Evidence collection calendar created
- [ ] Owners assigned for each evidence type

### Periodic Evidence Checklist

#### Monthly
- [ ] Security metrics report
- [ ] Incident summary report
- [ ] Change management summary
- [ ] Backup verification logs

#### Quarterly
- [ ] User access review completion (all systems)
- [ ] Risk register update
- [ ] Vendor review status
- [ ] Management/board security briefing

#### Annually
- [ ] Penetration test report
- [ ] Security awareness training records
- [ ] Business continuity/DR test results
- [ ] Policy review and approval records
- [ ] Risk assessment report
- [ ] Background check process verification

### Evidence Quality Checks
- [ ] Evidence covers the complete audit period
- [ ] No gaps in periodic control evidence
- [ ] Screenshots include timestamps and system identification
- [ ] Reports are in PDF or export format (not editable)
- [ ] Approval chains clearly visible in tickets

---

## Phase 4: Pre-Audit Readiness

### Documentation
- [ ] System description document completed and reviewed
- [ ] Management assertion letter drafted
- [ ] Subservice organization disclosures accurate
- [ ] CUECs (Complementary User Entity Controls) documented
- [ ] CSOCs (Complementary Subservice Organization Controls) documented

### Walkthrough Testing
- [ ] Sample of each control type tested internally
- [ ] Control operation verified against design
- [ ] Evidence adequacy confirmed
- [ ] Control owners can explain and demonstrate controls

### Gap Remediation
- [ ] All identified gaps documented
- [ ] Remediation plans with owners and timelines
- [ ] Critical gaps resolved before audit starts
- [ ] Compensating controls documented where needed

### Audit Team Preparation
- [ ] Control owners briefed on audit process
- [ ] Interview preparation materials distributed
- [ ] Evidence request list anticipated and pre-staged
- [ ] Auditor access to systems and tools arranged
- [ ] Communication channel with audit firm established

---

## Phase 5: Audit Execution

### Audit Support
- [ ] Kick-off meeting completed
- [ ] Information requests responded to within SLA
- [ ] Population lists provided (users, changes, incidents, vendors)
- [ ] Interviews scheduled and completed
- [ ] Exception responses prepared promptly

### Post-Audit
- [ ] Draft report reviewed for factual accuracy
- [ ] Exception descriptions verified
- [ ] Management responses drafted for exceptions
- [ ] Final report received and distributed
- [ ] Remediation plan created for all exceptions
- [ ] Lessons learned documented
- [ ] Next audit cycle planning initiated

---

## Summary Status

| Phase | Status | Completion % | Notes |
|-------|--------|-------------|-------|
| Scoping | | | |
| Control Design | | | |
| Evidence Collection | | | |
| Pre-Audit Readiness | | | |
| Audit Execution | | | |
| Post-Audit | | | |

## Exception Tracker

| Exception # | Control ID | TSC Criteria | Description | Root Cause | Remediation | Owner | Due Date | Status |
|-------------|-----------|-------------|-------------|------------|-------------|-------|----------|--------|
| | | | | | | | | |

## Sign-off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| SOC 2 Lead | | | |
| CISO | | | |
| CTO | | | |

===== END FILE =====


===== FILE: skills/performing-soc2-type2-audit-preparation/references/api-reference.md =====
# API Reference: SOC 2 Type II Audit Preparation

## AICPA Trust Services Criteria (2017, updated 2022)

### Five Trust Services Categories

| Category | Required | Focus Area |
|----------|----------|------------|
| Security (CC1-CC9) | Mandatory | Common criteria for all SOC 2 audits |
| Availability (A1) | Optional | System uptime, DR, capacity planning |
| Processing Integrity (PI1) | Optional | Data processing accuracy and completeness |
| Confidentiality (C1) | Optional | Confidential information protection |
| Privacy (P1-P8) | Optional | Personal information lifecycle |

### Common Criteria Detail

| ID | Name | Key Controls |
|----|------|-------------|
| CC1 | Control Environment | Ethics, board oversight, org structure, competence, accountability |
| CC2 | Communication and Information | Internal/external communications, system boundaries |
| CC3 | Risk Assessment | Risk identification, fraud risk, change impact analysis |
| CC4 | Monitoring Activities | Control evaluations, deficiency identification |
| CC5 | Control Activities | Policy implementation, technology controls |
| CC6 | Logical and Physical Access | Auth, access provisioning, encryption, MFA |
| CC7 | System Operations | Monitoring, anomaly detection, incident response, vuln management |
| CC8 | Change Management | Change authorization, testing, approval, documentation |
| CC9 | Risk Mitigation | Vendor management, business continuity, disaster recovery |

## AWS Evidence Collection APIs

### IAM (CC6 - Access Controls)

```python
import boto3

iam = boto3.client("iam")

# List all users
users = iam.list_users()

# Check MFA devices per user
mfa = iam.list_mfa_devices(UserName="username")

# Get password policy
policy = iam.get_account_password_policy()

# Generate credential report
iam.generate_credential_report()
report = iam.get_credential_report()

# List access keys and their last used dates
keys = iam.list_access_keys(UserName="username")
last_used = iam.get_access_key_last_used(AccessKeyId="AKIA...")

# List attached policies
policies = iam.list_attached_user_policies(UserName="username")
```

### CloudTrail (CC7 - System Operations)

```python
ct = boto3.client("cloudtrail")

# Describe all trails
trails = ct.describe_trails()

# Get trail logging status
status = ct.get_trail_status(Name="trail-arn")

# Lookup recent events
events = ct.lookup_events(
    LookupAttributes=[
        {"AttributeKey": "EventName", "AttributeValue": "ConsoleLogin"}
    ],
    StartTime=datetime(2025, 4, 1),
    EndTime=datetime(2026, 3, 31),
)
```

### S3 (CC6 - Data Protection)

```python
s3 = boto3.client("s3")

# List all buckets
buckets = s3.list_buckets()

# Check public access block
pab = s3.get_public_access_block(Bucket="bucket-name")

# Check encryption
enc = s3.get_bucket_encryption(Bucket="bucket-name")

# Check versioning
ver = s3.get_bucket_versioning(Bucket="bucket-name")

# Check logging
log = s3.get_bucket_logging(Bucket="bucket-name")
```

### GuardDuty (CC7 - Anomaly Detection)

```python
gd = boto3.client("guardduty")

# List detectors
detectors = gd.list_detectors()

# Get findings (high severity)
findings = gd.list_findings(
    DetectorId="detector-id",
    FindingCriteria={"Criterion": {"severity": {"Gte": 7}}}
)

# Get finding details
details = gd.get_findings(
    DetectorId="detector-id",
    FindingIds=findings["FindingIds"]
)
```

## GitHub Evidence Collection APIs

### Pull Request Evidence (CC8 - Change Management)

```python
import requests

headers = {
    "Authorization": "token ghp_xxx",
    "Accept": "application/vnd.github.v3+json",
}

# List merged PRs
prs = requests.get(
    "https://api.github.com/repos/org/repo/pulls",
    params={"state": "closed", "base": "main", "per_page": 100},
    headers=headers,
)

# Get PR reviews
reviews = requests.get(
    "https://api.github.com/repos/org/repo/pulls/123/reviews",
    headers=headers,
)

# Get branch protection rules
protection = requests.get(
    "https://api.github.com/repos/org/repo/branches/main/protection",
    headers=headers,
)
```

## Okta Evidence Collection APIs (CC6 - Identity)

```python
headers = {
    "Authorization": "SSWS okta-api-token",
    "Accept": "application/json",
}

# List all users
users = requests.get("https://org.okta.com/api/v1/users", headers=headers)

# Get user MFA factors
factors = requests.get(
    "https://org.okta.com/api/v1/users/userId/factors",
    headers=headers,
)

# List authentication policies
policies = requests.get(
    "https://org.okta.com/api/v1/policies?type=ACCESS_POLICY",
    headers=headers,
)

# Get system log events
logs = requests.get(
    "https://org.okta.com/api/v1/logs",
    params={"since": "2025-04-01T00:00:00Z"},
    headers=headers,
)
```

## Compliance Automation Platforms

### Vanta API (GraphQL)

```python
headers = {"Authorization": "Bearer vanta-token"}

query = """
{
  controls {
    id
    name
    status
    lastTestedAt
    evidence { id name collectedAt }
  }
}
"""
resp = requests.post(
    "https://api.vanta.com/graphql",
    json={"query": query},
    headers=headers,
)
```

### Drata API (REST)

```python
headers = {"Authorization": "Bearer drata-token"}

# List controls
controls = requests.get("https://public-api.drata.com/controls", headers=headers)

# Get control evidence
evidence = requests.get(
    "https://public-api.drata.com/controls/{controlId}/evidence",
    headers=headers,
)

# List monitors
monitors = requests.get("https://public-api.drata.com/monitors", headers=headers)
```

## SOC 2 Type I vs Type II

| Aspect | Type I | Type II |
|--------|--------|---------|
| Scope | Point in time | Period of time (3-12 months) |
| Assessment | Controls designed properly | Controls operated effectively |
| Evidence | Current state docs | Historical evidence over period |
| Duration | 1-2 months prep | 3-12 month observation window |
| Cost | $20,000 - $50,000 | $30,000 - $100,000+ |

## Evidence Collection Frequency

| Evidence Type | Frequency | Example |
|--------------|-----------|---------|
| Automated scans | Daily/Continuous | IAM MFA status, S3 public access |
| Access reviews | Quarterly | User access certification |
| Vulnerability scans | Weekly/Monthly | Nessus, Qualys reports |
| Penetration tests | Annual | Third-party pentest report |
| Policy reviews | Annual | Security policy updates |
| Training records | Annual | Security awareness completion |
| Incident reports | Per-incident | IR documentation |
| Board minutes | Quarterly | Security committee meeting notes |

### References

- AICPA Trust Services Criteria: https://us.aicpa.org/interestareas/frc/assuranceadvisoryservices/trustservicescriteria
- SOC 2 Guide: https://soc2.fyi/
- Vanta SOC 2: https://www.vanta.com/collection/soc-2
- Drata Trust Services Criteria: https://drata.com/blog/trust-services-criteria
- Secureframe SOC 2 Checklist: https://secureframe.com/blog/soc-2-compliance-checklist
- SANS SOC 2 Trust Categories: https://www.sans.org/blog/soc-2-trust-services-categories
- Splunk SOC 2 Compliance: https://www.splunk.com/en_us/blog/learn/soc-2-compliance-checklist.html

===== END FILE =====


===== FILE: skills/performing-soc2-type2-audit-preparation/references/standards.md =====
# SOC 2 Type II Standards Reference

## Primary Standards

### AICPA Trust Services Criteria (TSC) 2017 (Revised 2022)
- **Governing Body**: American Institute of Certified Public Accountants (AICPA)
- **Basis**: Built on COSO 2013 Internal Control Framework
- **Revision**: Points of Focus updated in 2022 to address cloud, supply chain, and evolving technology risks
- **Description Criteria**: Updated July 2025

### SSAE 18 (Statement on Standards for Attestation Engagements No. 18)
- **Standard**: AT-C Section 105, 205, and 320
- **Purpose**: Governs the attestation engagement performed by the CPA firm
- **Effective**: May 1, 2017 (replaced SSAE 16)

## Trust Services Criteria Detail

### Security (Common Criteria) - MANDATORY
CC1: Control Environment
- CC1.1: COSO Principle 1 - Demonstrates commitment to integrity and ethical values
- CC1.2: COSO Principle 2 - Board exercises oversight responsibility
- CC1.3: COSO Principle 3 - Management establishes structures, reporting lines, authorities
- CC1.4: COSO Principle 4 - Demonstrates commitment to attract, develop, retain competent individuals
- CC1.5: COSO Principle 5 - Holds individuals accountable for internal control responsibilities

CC2: Communication and Information
- CC2.1: COSO Principle 13 - Uses relevant, quality information to support internal control
- CC2.2: COSO Principle 14 - Internally communicates information supporting internal control
- CC2.3: COSO Principle 15 - Communicates with external parties regarding internal control

CC3: Risk Assessment
- CC3.1: COSO Principle 6 - Specifies objectives with sufficient clarity
- CC3.2: COSO Principle 7 - Identifies risks to achievement of objectives
- CC3.3: COSO Principle 8 - Considers potential for fraud
- CC3.4: COSO Principle 9 - Identifies and assesses changes that could impact internal control

CC4: Monitoring Activities
- CC4.1: COSO Principle 16 - Selects, develops, performs ongoing and separate evaluations
- CC4.2: COSO Principle 17 - Evaluates and communicates internal control deficiencies

CC5: Control Activities
- CC5.1: COSO Principle 10 - Selects and develops control activities
- CC5.2: COSO Principle 11 - Selects and develops general controls over technology
- CC5.3: COSO Principle 12 - Deploys through policies and procedures

CC6: Logical and Physical Access Controls
- CC6.1: Logical access security software, infrastructure, and architectures
- CC6.2: Prior to credential issuance, registration and authorization processes
- CC6.3: Access removal, modification upon changes to roles
- CC6.4: Physical access restrictions to facilities and protected information assets
- CC6.5: Changes in physical access restrictions are managed
- CC6.6: Logical access security measures against threats from external sources
- CC6.7: Restricts transmission, movement, and removal of information
- CC6.8: Controls against threats from deployment of unauthorized or malicious code

CC7: System Operations
- CC7.1: Detection and monitoring for anomalies and events
- CC7.2: Activities monitored against security event criteria
- CC7.3: Procedures exist to evaluate security events
- CC7.4: Response to identified security incidents
- CC7.5: Identification and remediation of identified vulnerabilities

CC8: Change Management
- CC8.1: Changes to infrastructure, data, software, and procedures are authorized, designed, developed, tested, approved, and implemented

CC9: Risk Mitigation
- CC9.1: Risk mitigation activities are considered through risk assessment
- CC9.2: Assesses and manages risks through vendor/business partner activities

### Availability (Optional)
- A1.1: Performance and capacity maintenance
- A1.2: Environmental protections, software, data backup and recovery
- A1.3: Recovery plan testing

### Processing Integrity (Optional)
- PI1.1: Obtains or generates, uses, and communicates relevant quality information
- PI1.2: System inputs are complete, accurate, and timely
- PI1.3: Processing is complete, valid, accurate, timely, and authorized
- PI1.4: System output is complete, valid, accurate, timely, and authorized
- PI1.5: Data stored is complete, valid, accurate, timely, and authorized

### Confidentiality (Optional)
- C1.1: Identifies and maintains confidential information
- C1.2: Disposes of confidential information

### Privacy (Optional)
- P1.0-P8.0: Covers notice, choice, collection, use, retention, disclosure, access, quality, and monitoring/enforcement

## SOC 2 Report Structure

### Section I: Independent Service Auditor's Report
- Auditor opinion on control design and operating effectiveness
- Scope of examination and applicable criteria

### Section II: Management's Assertion
- Management's representation regarding system description and control effectiveness

### Section III: Description of the System
- Nature of services, principal service commitments, system requirements
- Components: infrastructure, software, people, procedures, data
- Boundaries and subservice organizations

### Section IV: Description of Criteria, Controls, Tests, and Results
- Each TSC criterion with mapped controls
- Test procedures performed by auditor
- Results of testing (no exceptions / exception noted)

### Section V: Other Information (Optional)
- Complementary User Entity Controls (CUECs)
- Complementary Subservice Organization Controls (CSOCs)

## Related Standards
- SOC 1 (SSAE 18/ISAE 3402): Financial reporting controls
- SOC 3 (Trust Services Criteria): Public-facing summary report
- ISO 27001: Information Security Management System
- NIST CSF: Cybersecurity Framework (mappings available)

===== END FILE =====


===== FILE: skills/performing-soc2-type2-audit-preparation/references/workflows.md =====
# SOC 2 Type II Audit Preparation Workflows

## Workflow 1: Scoping and TSC Selection

```
Start
  |
  v
[Analyze Customer Requirements]
  - Review customer security questionnaires
  - Identify frequently requested TSC categories
  - Review contractual obligations
  |
  v
[Define System Boundaries]
  - Identify in-scope services and applications
  - Map infrastructure components
  - Identify data flows and storage locations
  - Determine subservice organizations (e.g., AWS, Azure)
  |
  v
[Select TSC Categories]
  - Security (CC) - Always included
  |
  +--> [SaaS with uptime SLAs?] --> Include Availability (A)
  |
  +--> [Process financial/sensitive data?] --> Include Processing Integrity (PI)
  |
  +--> [Handle confidential customer data?] --> Include Confidentiality (C)
  |
  +--> [Collect/process personal data?] --> Include Privacy (P)
  |
  v
[Document Scope and Boundaries]
  |
  v
[Select Audit Firm]
  - Verify CPA firm qualifications
  - Confirm experience with your industry
  - Agree on audit timeline and fees
  |
  v
End
```

## Workflow 2: Control Design and Mapping

```
Start
  |
  v
[Review TSC Requirements]
  - Identify all applicable criteria and points of focus
  |
  v
[Map Existing Controls to TSC]
  - Inventory current security controls
  - Map each control to TSC criteria
  - Identify coverage gaps
  |
  v
[Design New Controls for Gaps]
  - Define control objective
  - Define control activity
  - Set control frequency
  - Assign control owner
  - Define evidence requirements
  |
  v
[Classify Control Types]
  |
  +--> Preventive: Stops events before they occur
  |     (e.g., MFA, firewall rules, access reviews)
  |
  +--> Detective: Identifies events after they occur
  |     (e.g., SIEM alerts, audit logs, vulnerability scans)
  |
  +--> Corrective: Remedies events after detection
        (e.g., incident response, patching, access revocation)
  |
  v
[Document Control Matrix]
  - TSC Criterion -> Control ID -> Description -> Type ->
    Frequency -> Owner -> Evidence -> Test Procedure
  |
  v
[Implement Controls]
  |
  v
[Validate Control Operation]
  |
  v
End
```

## Workflow 3: Evidence Collection Process

```
Start
  |
  v
[Establish Evidence Repository]
  - Create structured folder hierarchy by TSC category
  - Set naming conventions (YYYY-MM_CC6.1_AccessReview)
  - Assign evidence collection responsibilities
  |
  v
[Continuous Controls (Daily/Real-time)]
  - SIEM log collection and alerting
  - Intrusion detection monitoring
  - Endpoint protection status
  - Encryption enforcement
  |
  v
[Periodic Controls]
  |
  +--> Weekly:
  |     - Vulnerability scan reports
  |     - Backup verification
  |
  +--> Monthly:
  |     - Security metric reports
  |     - Incident summary
  |     - Change management review
  |
  +--> Quarterly:
  |     - User access reviews (CC6.3)
  |     - Risk assessment updates (CC3.2)
  |     - Vendor security reviews (CC9.2)
  |     - Board/management reporting (CC1.2)
  |
  +--> Annually:
        - Penetration testing (CC7.1)
        - Security awareness training (CC1.4)
        - Business continuity testing (A1.3)
        - Policy reviews and updates (CC5.3)
        - Risk assessment (CC3.1)
  |
  v
[Organize and Label Evidence]
  - Screenshot with timestamps
  - Export system reports to PDF
  - Preserve ticket/approval chains
  - Document manual control execution
  |
  v
[Quality Check Evidence]
  - Verify coverage for entire audit period
  - Confirm no gaps in periodic controls
  - Validate evidence matches control description
  |
  v
End
```

## Workflow 4: Readiness Assessment

```
Start
  |
  v
[Perform Walkthrough Testing]
  - Select sample of each control type
  - Trace control from input to evidence
  - Verify control operates as designed
  |
  v
[Identify Gaps and Exceptions]
  |
  +--> [Control Not Operating?]
  |     - Document exception
  |     - Implement remediation
  |     - Restart evidence collection
  |
  +--> [Evidence Missing?]
  |     - Locate alternative evidence
  |     - Implement improved capture
  |
  +--> [Control Design Insufficient?]
        - Redesign control
        - Implement and begin new evidence period
  |
  v
[Prepare System Description]
  - Overview of organization
  - Principal service commitments
  - System components description
  - Subservice organization relationships
  - CUECs and CSOCs
  |
  v
[Prepare Management Assertion]
  - Confirm system description is fairly presented
  - Assert controls were suitably designed
  - Assert controls operated effectively
  |
  v
[Brief Control Owners]
  - Explain audit process
  - Review evidence expectations
  - Prepare for auditor interviews
  |
  v
End
```

## Workflow 5: Audit Execution Support

```
Start
  |
  v
[Kick-off Meeting with Auditor]
  - Confirm scope and timeline
  - Provide system description
  - Share evidence repository access
  - Establish communication cadence
  |
  v
[Respond to Information Requests]
  - Provide requested populations (user lists, change tickets, etc.)
  - Facilitate system access for auditor testing
  - Schedule interviews with control owners
  |
  v
[Auditor Performs Testing]
  - Inquiry (interviews with control owners)
  - Observation (watch control operation)
  - Inspection (review evidence documents)
  - Reperformance (re-execute control steps)
  |
  v
[Address Exceptions]
  |
  +--> [Exception Found]
  |     - Investigate root cause
  |     - Determine if compensating controls exist
  |     - Provide additional context to auditor
  |     - Document remediation plan
  |
  +--> [No Exceptions] --> Continue
  |
  v
[Review Draft Report]
  - Check system description accuracy
  - Verify control descriptions
  - Review exception descriptions
  - Confirm factual accuracy
  |
  v
[Receive Final Report]
  |
  v
[Plan Remediation for Exceptions]
  |
  v
End
```

===== END FILE =====


===== FILE: skills/performing-soc2-type2-audit-preparation/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for SOC 2 Type II audit preparation, evidence collection, and compliance monitoring."""

import os
import sys
import json
import argparse
from datetime import datetime, timedelta, timezone

import requests

try:
    import boto3
except ImportError:
    boto3 = None

try:
    import yaml
except ImportError:
    yaml = None


# AICPA Trust Services Criteria - Common Criteria (CC1-CC9)
TRUST_SERVICES_CRITERIA = {
    "CC1": {
        "name": "Control Environment",
        "description": "Organization demonstrates commitment to integrity, ethical values, oversight, structure, authority, responsibility, and competence.",
        "controls": {
            "CC1.1": "Demonstrates commitment to integrity and ethical values",
            "CC1.2": "Board exercises oversight responsibility",
            "CC1.3": "Management establishes structure, authority, and responsibility",
            "CC1.4": "Demonstrates commitment to competence",
            "CC1.5": "Enforces accountability",
        },
    },
    "CC2": {
        "name": "Communication and Information",
        "description": "Organization communicates internal and external information necessary to support the functioning of internal control.",
        "controls": {
            "CC2.1": "Obtains or generates relevant quality information",
            "CC2.2": "Internally communicates information including objectives and responsibilities",
            "CC2.3": "Communicates with external parties regarding matters affecting functioning of internal control",
        },
    },
    "CC3": {
        "name": "Risk Assessment",
        "description": "Organization identifies risks to the achievement of objectives and analyzes risks to determine how they should be managed.",
        "controls": {
            "CC3.1": "Specifies objectives with clarity to enable identification of risks",
            "CC3.2": "Identifies risks to achievement of objectives and analyzes them",
            "CC3.3": "Considers potential for fraud in assessing risks",
            "CC3.4": "Identifies and assesses changes that could impact internal control",
        },
    },
    "CC4": {
        "name": "Monitoring Activities",
        "description": "Organization selects, develops, and performs evaluations to ascertain whether controls are present and functioning.",
        "controls": {
            "CC4.1": "Selects, develops, and performs ongoing and separate evaluations",
            "CC4.2": "Evaluates and communicates internal control deficiencies in a timely manner",
        },
    },
    "CC5": {
        "name": "Control Activities",
        "description": "Organization deploys control activities through policies and procedures that put directives into actions.",
        "controls": {
            "CC5.1": "Selects and develops control activities that contribute to mitigation of risks",
            "CC5.2": "Selects and develops general controls over technology",
            "CC5.3": "Deploys control activities through policies and procedures",
        },
    },
    "CC6": {
        "name": "Logical and Physical Access Controls",
        "description": "Organization restricts logical and physical access to information assets.",
        "controls": {
            "CC6.1": "Implements logical access security software, infrastructure, and architectures",
            "CC6.2": "Prior to issuing system credentials, registers and authorizes new users",
            "CC6.3": "Removes access to protected information assets when appropriate",
            "CC6.4": "Restricts physical access to facilities and protected information assets",
            "CC6.5": "Implements controls to prevent and detect unauthorized access",
            "CC6.6": "Manages points of interaction with external systems",
            "CC6.7": "Restricts the transmission, movement, and removal of information",
            "CC6.8": "Implements controls to prevent or detect unauthorized or malicious software",
        },
    },
    "CC7": {
        "name": "System Operations",
        "description": "Organization detects and monitors system components and anomalies that represent events.",
        "controls": {
            "CC7.1": "Detects and monitors system configuration changes and new vulnerabilities",
            "CC7.2": "Monitors system components for anomalies indicative of malicious acts",
            "CC7.3": "Evaluates detected events and determines whether they constitute incidents",
            "CC7.4": "Responds to identified security incidents",
            "CC7.5": "Identifies, develops, and implements activities to recover from incidents",
        },
    },
    "CC8": {
        "name": "Change Management",
        "description": "Organization authorizes, designs, develops, configures, documents, tests, approves, and implements changes.",
        "controls": {
            "CC8.1": "Authorizes, designs, develops, configures, documents, tests, approves, and implements changes to infrastructure and software",
        },
    },
    "CC9": {
        "name": "Risk Mitigation",
        "description": "Organization identifies, selects, and develops risk mitigation activities for risks arising from business disruption and use of vendors.",
        "controls": {
            "CC9.1": "Identifies and assesses risk mitigation activities for risks from business disruptions",
            "CC9.2": "Assesses and manages risks associated with vendors and business partners",
        },
    },
}


def perform_gap_assessment(controls_status):
    """Perform a gap assessment against all Trust Services Criteria CC1-CC9."""
    results = {
        "assessment_date": datetime.now(timezone.utc).isoformat(),
        "summary": {
            "total_controls": 0, "implemented": 0, "partial": 0,
            "not_implemented": 0, "not_assessed": 0,
        },
        "gaps": [],
        "criteria_status": {},
    }

    for cc_id, cc_data in TRUST_SERVICES_CRITERIA.items():
        criteria_result = {"name": cc_data["name"], "controls": {}}
        for ctrl_id, ctrl_desc in cc_data["controls"].items():
            results["summary"]["total_controls"] += 1
            status_info = controls_status.get(ctrl_id, {})
            status = status_info.get("status", "not_assessed")

            criteria_result["controls"][ctrl_id] = {
                "description": ctrl_desc,
                "status": status,
                "evidence": status_info.get("evidence", ""),
                "gap": status_info.get("gap", ""),
                "owner": status_info.get("owner", ""),
                "due_date": status_info.get("due_date", ""),
            }

            if status == "implemented":
                results["summary"]["implemented"] += 1
            elif status == "partial":
                results["summary"]["partial"] += 1
                results["gaps"].append({
                    "control": ctrl_id, "criteria": cc_id,
                    "description": ctrl_desc,
                    "gap": status_info.get("gap", ""),
                    "severity": "medium",
                })
            elif status == "not_implemented":
                results["summary"]["not_implemented"] += 1
                results["gaps"].append({
                    "control": ctrl_id, "criteria": cc_id,
                    "description": ctrl_desc,
                    "gap": status_info.get("gap", ""),
                    "severity": "high",
                })
            else:
                results["summary"]["not_assessed"] += 1

        results["criteria_status"][cc_id] = criteria_result

    total = results["summary"]["total_controls"]
    implemented = results["summary"]["implemented"]
    results["summary"]["readiness_score"] = (
        round((implemented / total) * 100, 1) if total > 0 else 0
    )
    return results


def collect_aws_iam_evidence():
    """Collect AWS IAM evidence for CC6 (Access Controls)."""
    if boto3 is None:
        return {"status": "error", "error": "boto3 not installed"}

    iam = boto3.client("iam")
    evidence = {
        "collected_at": datetime.now(timezone.utc).isoformat(),
        "criteria": "CC6",
        "type": "aws_iam",
    }

    try:
        users = iam.list_users()["Users"]
        user_details = []
        users_without_mfa = []

        for user in users:
            mfa_devices = iam.list_mfa_devices(UserName=user["UserName"])
            has_mfa = len(mfa_devices["MFADevices"]) > 0
            user_info = {
                "username": user["UserName"],
                "created": user["CreateDate"].isoformat(),
                "mfa_enabled": has_mfa,
                "arn": user["Arn"],
            }
            if "PasswordLastUsed" in user:
                user_info["password_last_used"] = user["PasswordLastUsed"].isoformat()
            user_details.append(user_info)
            if not has_mfa:
                users_without_mfa.append(user["UserName"])

        evidence["users"] = {
            "total": len(users),
            "with_mfa": len(users) - len(users_without_mfa),
            "without_mfa": len(users_without_mfa),
            "mfa_compliance_rate": round(
                ((len(users) - len(users_without_mfa)) / max(len(users), 1)) * 100, 1
            ),
            "users_without_mfa": users_without_mfa,
            "details": user_details,
        }
    except Exception as e:
        evidence["users"] = {"error": str(e)}

    try:
        policy = iam.get_account_password_policy()["PasswordPolicy"]
        evidence["password_policy"] = {
            "minimum_length": policy.get("MinimumPasswordLength", 0),
            "require_symbols": policy.get("RequireSymbols", False),
            "require_numbers": policy.get("RequireNumbers", False),
            "require_uppercase": policy.get("RequireUppercaseCharacters", False),
            "require_lowercase": policy.get("RequireLowercaseCharacters", False),
            "max_age_days": policy.get("MaxPasswordAge", 0),
            "password_reuse_prevention": policy.get("PasswordReusePrevention", 0),
        }
    except Exception as e:
        evidence["password_policy"] = {"error": str(e)}

    return evidence


def collect_aws_cloudtrail_evidence():
    """Collect AWS CloudTrail evidence for CC7 (System Operations)."""
    if boto3 is None:
        return {"status": "error", "error": "boto3 not installed"}

    ct = boto3.client("cloudtrail")
    evidence = {
        "collected_at": datetime.now(timezone.utc).isoformat(),
        "criteria": "CC7",
        "type": "aws_cloudtrail",
    }

    try:
        trails = ct.describe_trails()["trailList"]
        trail_details = []
        for trail in trails:
            status = ct.get_trail_status(Name=trail["TrailARN"])
            trail_details.append({
                "name": trail["Name"],
                "arn": trail["TrailARN"],
                "is_logging": status["IsLogging"],
                "multi_region": trail.get("IsMultiRegionTrail", False),
                "log_validation": trail.get("LogFileValidationEnabled", False),
                "s3_bucket": trail.get("S3BucketName", ""),
                "kms_key": trail.get("KmsKeyId", "none"),
            })
        evidence["trails"] = trail_details
        evidence["all_logging"] = all(t["is_logging"] for t in trail_details)
        evidence["all_multi_region"] = all(t["multi_region"] for t in trail_details)
    except Exception as e:
        evidence["trails"] = {"error": str(e)}

    return evidence


def collect_aws_s3_public_access_evidence():
    """Collect AWS S3 public access evidence for CC6 (Access Controls)."""
    if boto3 is None:
        return {"status": "error", "error": "boto3 not installed"}

    s3 = boto3.client("s3")
    evidence = {
        "collected_at": datetime.now(timezone.utc).isoformat(),
        "criteria": "CC6",
        "type": "aws_s3_public_access",
    }

    try:
        buckets = s3.list_buckets()["Buckets"]
        bucket_details = []
        public_buckets = []

        for bucket in buckets:
            name = bucket["Name"]
            try:
                pab = s3.get_public_access_block(Bucket=name)
                cfg = pab["PublicAccessBlockConfiguration"]
                is_public_blocked = all([
                    cfg.get("BlockPublicAcls", False),
                    cfg.get("IgnorePublicAcls", False),
                    cfg.get("BlockPublicPolicy", False),
                    cfg.get("RestrictPublicBuckets", False),
                ])
            except Exception:
                is_public_blocked = False

            try:
                encryption = s3.get_bucket_encryption(Bucket=name)
                has_encryption = True
                enc_algorithm = (
                    encryption["ServerSideEncryptionConfiguration"]["Rules"][0]
                    ["ApplyServerSideEncryptionByDefault"]["SSEAlgorithm"]
                )
            except Exception:
                has_encryption = False
                enc_algorithm = "none"

            bucket_details.append({
                "name": name,
                "created": bucket["CreationDate"].isoformat(),
                "public_access_blocked": is_public_blocked,
                "encrypted": has_encryption,
                "encryption_algorithm": enc_algorithm,
            })
            if not is_public_blocked:
                public_buckets.append(name)

        evidence["buckets"] = {
            "total": len(buckets),
            "public_access_blocked": len(buckets) - len(public_buckets),
            "potentially_public": len(public_buckets),
            "public_bucket_names": public_buckets,
            "details": bucket_details,
        }
    except Exception as e:
        evidence["buckets"] = {"error": str(e)}

    return evidence


def collect_github_change_management_evidence(org, repo, token, since=None):
    """Collect GitHub PR evidence for CC8 (Change Management)."""
    if since is None:
        since = (datetime.now(timezone.utc) - timedelta(days=90)).isoformat()

    headers = {
        "Authorization": "token " + token,
        "Accept": "application/vnd.github.v3+json",
    }
    evidence = {
        "collected_at": datetime.now(timezone.utc).isoformat(),
        "criteria": "CC8",
        "type": "github_change_management",
        "repository": org + "/" + repo,
    }

    try:
        page = 1
        all_prs = []
        while page <= 10:
            resp = requests.get(
                "https://api.github.com/repos/" + org + "/" + repo + "/pulls",
                params={
                    "state": "closed", "base": "main",
                    "per_page": 100, "page": page,
                    "sort": "updated", "direction": "desc",
                },
                headers=headers, timeout=30,
            )
            resp.raise_for_status()
            prs = resp.json()
            if not prs:
                break
            for pr in prs:
                if pr.get("merged_at") and pr["merged_at"] >= since:
                    all_prs.append(pr)
            page += 1

        pr_details = []
        prs_without_approval = []
        for pr in all_prs:
            reviews_resp = requests.get(
                pr["url"] + "/reviews", headers=headers, timeout=15,
            )
            reviews = reviews_resp.json() if reviews_resp.ok else []
            approved = any(r["state"] == "APPROVED" for r in reviews)
            reviewers = list({r["user"]["login"] for r in reviews if "user" in r})

            pr_details.append({
                "number": pr["number"],
                "title": pr["title"],
                "author": pr["user"]["login"],
                "merged_at": pr["merged_at"],
                "approved": approved,
                "reviewers": reviewers,
                "additions": pr.get("additions", 0),
                "deletions": pr.get("deletions", 0),
            })
            if not approved:
                prs_without_approval.append(pr["number"])

        total = len(pr_details)
        approved_count = total - len(prs_without_approval)
        evidence["pull_requests"] = {
            "total_merged": total,
            "with_approval": approved_count,
            "without_approval": len(prs_without_approval),
            "approval_rate": round((approved_count / max(total, 1)) * 100, 1),
            "exceptions": prs_without_approval,
            "details": pr_details,
        }
    except Exception as e:
        evidence["pull_requests"] = {"error": str(e)}

    return evidence


def collect_branch_protection_evidence(org, repo, token):
    """Collect GitHub branch protection evidence for CC8."""
    headers = {
        "Authorization": "token " + token,
        "Accept": "application/vnd.github.v3+json",
    }
    evidence = {
        "collected_at": datetime.now(timezone.utc).isoformat(),
        "criteria": "CC8",
        "type": "github_branch_protection",
    }

    try:
        resp = requests.get(
            "https://api.github.com/repos/" + org + "/" + repo + "/branches/main/protection",
            headers=headers, timeout=15,
        )
        if resp.status_code == 200:
            protection = resp.json()
            pr_reviews = protection.get("required_pull_request_reviews", {})
            evidence["main_branch"] = {
                "protected": True,
                "required_reviews": pr_reviews.get("required_approving_review_count", 0),
                "dismiss_stale_reviews": pr_reviews.get("dismiss_stale_reviews", False),
                "require_code_owner_reviews": pr_reviews.get("require_code_owner_reviews", False),
                "status_checks_required": protection.get("required_status_checks", {}).get("strict", False),
                "enforce_admins": protection.get("enforce_admins", {}).get("enabled", False),
            }
        elif resp.status_code == 404:
            evidence["main_branch"] = {
                "protected": False,
                "gap": "No branch protection configured",
            }
        else:
            evidence["main_branch"] = {"error": "HTTP " + str(resp.status_code)}
    except Exception as e:
        evidence["main_branch"] = {"error": str(e)}

    return evidence


def run_continuous_compliance_checks(aws_enabled=True):
    """Run a suite of continuous compliance checks and return findings."""
    findings = {
        "check_time": datetime.now(timezone.utc).isoformat(),
        "checks": [], "passing": 0, "failing": 0, "errors": 0,
    }
    checks = []

    if aws_enabled and boto3 is not None:
        # CC6.1: MFA enforcement
        try:
            iam = boto3.client("iam")
            users = iam.list_users()["Users"]
            users_without_mfa = []
            for user in users:
                mfa_devices = iam.list_mfa_devices(UserName=user["UserName"])
                if not mfa_devices["MFADevices"]:
                    users_without_mfa.append(user["UserName"])
            checks.append({
                "control": "CC6.1",
                "check": "All IAM users have MFA enabled",
                "passing": len(users_without_mfa) == 0,
                "details": {"total_users": len(users), "without_mfa": users_without_mfa},
            })
        except Exception as e:
            checks.append({
                "control": "CC6.1",
                "check": "All IAM users have MFA enabled",
                "passing": None, "error": str(e),
            })

        # CC6.6: No public S3 buckets
        try:
            s3 = boto3.client("s3")
            buckets = s3.list_buckets()["Buckets"]
            public_buckets = []
            for bucket in buckets:
                try:
                    pab = s3.get_public_access_block(Bucket=bucket["Name"])
                    cfg = pab["PublicAccessBlockConfiguration"]
                    if not all([
                        cfg.get("BlockPublicAcls", False),
                        cfg.get("IgnorePublicAcls", False),
                        cfg.get("BlockPublicPolicy", False),
                        cfg.get("RestrictPublicBuckets", False),
                    ]):
                        public_buckets.append(bucket["Name"])
                except Exception:
                    public_buckets.append(bucket["Name"])
            checks.append({
                "control": "CC6.6",
                "check": "No public S3 buckets",
                "passing": len(public_buckets) == 0,
                "details": {"public_buckets": public_buckets},
            })
        except Exception as e:
            checks.append({
                "control": "CC6.6",
                "check": "No public S3 buckets",
                "passing": None, "error": str(e),
            })

        # CC7.1: CloudTrail logging enabled
        try:
            ct = boto3.client("cloudtrail")
            trails = ct.describe_trails()["trailList"]
            inactive_trails = []
            for trail in trails:
                status = ct.get_trail_status(Name=trail["TrailARN"])
                if not status["IsLogging"]:
                    inactive_trails.append(trail["Name"])
            checks.append({
                "control": "CC7.1",
                "check": "CloudTrail logging enabled on all trails",
                "passing": len(inactive_trails) == 0,
                "details": {"inactive_trails": inactive_trails},
            })
        except Exception as e:
            checks.append({
                "control": "CC7.1",
                "check": "CloudTrail logging enabled on all trails",
                "passing": None, "error": str(e),
            })

    findings["checks"] = checks
    for check in checks:
        if check.get("passing") is True:
            findings["passing"] += 1
        elif check.get("passing") is False:
            findings["failing"] += 1
        else:
            findings["errors"] += 1

    total = findings["passing"] + findings["failing"]
    findings["compliance_score"] = round(
        (findings["passing"] / max(total, 1)) * 100, 1
    )
    return findings


def generate_remediation_plan(gap_assessment):
    """Generate a prioritized remediation plan from gap assessment results."""
    plan = {
        "generated_at": datetime.now(timezone.utc).isoformat(),
        "total_gaps": len(gap_assessment.get("gaps", [])),
        "remediation_items": [],
    }

    severity_priority = {"high": 1, "medium": 2, "low": 3}
    sorted_gaps = sorted(
        gap_assessment.get("gaps", []),
        key=lambda g: severity_priority.get(g.get("severity", "low"), 3),
    )

    remediation_map = {
        "CC1": (["Draft information security policy", "Establish security committee charter", "Create security awareness training", "Document org structure"], "2-4 weeks"),
        "CC2": (["Publish security policy to all staff", "Document system boundaries", "Establish external communication procedures"], "2-4 weeks"),
        "CC3": (["Conduct formal risk assessment", "Create risk register with owners", "Document fraud risk analysis", "Assess change impacts"], "4-8 weeks"),
        "CC4": (["Implement continuous control monitoring", "Establish internal audit program", "Create deficiency tracking process"], "3-6 weeks"),
        "CC5": (["Map policies to procedures", "Deploy technology controls", "Create control activities documentation"], "3-6 weeks"),
        "CC6": (["Enforce MFA across all accounts", "Automate access provisioning/deprovisioning", "Establish quarterly access reviews", "Deploy encryption at rest and in transit"], "2-4 weeks"),
        "CC7": (["Enable CloudTrail/audit logging everywhere", "Deploy SIEM with anomaly alerting", "Implement weekly vulnerability scanning", "Document and test IR procedures"], "3-6 weeks"),
        "CC8": (["Enforce branch protection with required reviews", "Implement CI/CD with automated testing", "Create change advisory board process", "Document emergency change procedures"], "1-3 weeks"),
        "CC9": (["Implement vendor risk management program", "Create BCP and DR plans", "Conduct annual DR testing", "Collect vendor SOC 2 reports"], "4-8 weeks"),
    }

    for i, gap in enumerate(sorted_gaps, 1):
        actions, effort = remediation_map.get(
            gap["criteria"],
            (["Review and implement appropriate controls"], "2-4 weeks"),
        )
        plan["remediation_items"].append({
            "priority": i,
            "control": gap["control"],
            "criteria": gap["criteria"],
            "description": gap["description"],
            "severity": gap.get("severity", "medium"),
            "recommended_actions": actions,
            "estimated_effort": effort,
        })

    return plan


def generate_evidence_manifest(audit_start, audit_end):
    """Generate a manifest of required evidence packages organized by criteria."""
    manifest = {
        "audit_period": {"start": audit_start, "end": audit_end},
        "generated_at": datetime.now(timezone.utc).isoformat(),
        "evidence_packages": {},
    }

    evidence_map = {
        "CC1": [
            {"name": "Code of Conduct Acknowledgments", "source": "HR system", "frequency": "annual"},
            {"name": "Board Meeting Minutes", "source": "Board secretary", "frequency": "quarterly"},
            {"name": "Organizational Chart", "source": "HR system", "frequency": "annual"},
            {"name": "Background Check Policy", "source": "HR system", "frequency": "annual"},
            {"name": "Security Training Completion Records", "source": "LMS", "frequency": "annual"},
        ],
        "CC2": [
            {"name": "Information Security Policy", "source": "Policy repo", "frequency": "annual"},
            {"name": "System Description Document", "source": "Engineering", "frequency": "annual"},
        ],
        "CC3": [
            {"name": "Risk Assessment Report", "source": "GRC platform", "frequency": "annual"},
            {"name": "Risk Register", "source": "GRC platform", "frequency": "quarterly"},
        ],
        "CC4": [
            {"name": "Control Monitoring Dashboard", "source": "Compliance platform", "frequency": "monthly"},
            {"name": "Internal Audit Reports", "source": "Internal audit", "frequency": "annual"},
        ],
        "CC5": [
            {"name": "IT General Controls Matrix", "source": "GRC platform", "frequency": "annual"},
        ],
        "CC6": [
            {"name": "IAM User MFA Status", "source": "AWS IAM / Okta", "frequency": "daily"},
            {"name": "Access Provisioning Tickets", "source": "Jira/ServiceNow", "frequency": "continuous"},
            {"name": "Quarterly Access Reviews", "source": "IAM system", "frequency": "quarterly"},
            {"name": "Terminated User Access Removal", "source": "HR + IAM", "frequency": "continuous"},
            {"name": "S3 Public Access Report", "source": "AWS S3", "frequency": "daily"},
            {"name": "Encryption Configuration", "source": "AWS KMS", "frequency": "daily"},
            {"name": "Password Policy Config", "source": "AWS IAM / Okta", "frequency": "monthly"},
        ],
        "CC7": [
            {"name": "CloudTrail Logging Status", "source": "AWS CloudTrail", "frequency": "daily"},
            {"name": "SIEM Alert Summaries", "source": "SIEM", "frequency": "monthly"},
            {"name": "Vulnerability Scan Reports", "source": "Scanner", "frequency": "weekly"},
            {"name": "Incident Response Reports", "source": "IR team", "frequency": "per-incident"},
            {"name": "GuardDuty Findings", "source": "AWS GuardDuty", "frequency": "daily"},
        ],
        "CC8": [
            {"name": "PR Approval Records", "source": "GitHub", "frequency": "continuous"},
            {"name": "Branch Protection Config", "source": "GitHub", "frequency": "monthly"},
            {"name": "CI/CD Pipeline Config", "source": "GitHub Actions", "frequency": "monthly"},
        ],
        "CC9": [
            {"name": "Vendor Risk Assessments", "source": "GRC platform", "frequency": "annual"},
            {"name": "Business Continuity Plan", "source": "BCP team", "frequency": "annual"},
            {"name": "DR Test Results", "source": "Engineering", "frequency": "annual"},
            {"name": "Vendor SOC 2 Reports", "source": "Vendors", "frequency": "annual"},
        ],
    }

    for cc_id, items in evidence_map.items():
        criteria_name = TRUST_SERVICES_CRITERIA.get(cc_id, {}).get("name", cc_id)
        manifest["evidence_packages"][cc_id] = {
            "criteria_name": criteria_name,
            "evidence_items": items,
            "item_count": len(items),
        }

    manifest["total_evidence_items"] = sum(
        pkg["item_count"] for pkg in manifest["evidence_packages"].values()
    )
    return manifest


def generate_readiness_report(gap_assessment=None, compliance_checks=None):
    """Generate a comprehensive audit readiness report."""
    report = {
        "report_type": "SOC 2 Type II Audit Readiness",
        "generated_at": datetime.now(timezone.utc).isoformat(),
        "overall_readiness": "unknown",
        "sections": {},
    }

    if gap_assessment:
        report["sections"]["gap_assessment"] = {
            "readiness_score": gap_assessment["summary"]["readiness_score"],
            "total_controls": gap_assessment["summary"]["total_controls"],
            "implemented": gap_assessment["summary"]["implemented"],
            "gaps_remaining": len(gap_assessment["gaps"]),
            "high_severity_gaps": len(
                [g for g in gap_assessment["gaps"] if g["severity"] == "high"]
            ),
        }

    if compliance_checks:
        report["sections"]["continuous_compliance"] = {
            "compliance_score": compliance_checks["compliance_score"],
            "checks_passing": compliance_checks["passing"],
            "checks_failing": compliance_checks["failing"],
            "checks_errored": compliance_checks["errors"],
        }

    scores = []
    if gap_assessment:
        scores.append(gap_assessment["summary"]["readiness_score"])
    if compliance_checks:
        scores.append(compliance_checks["compliance_score"])

    if scores:
        avg_score = sum(scores) / len(scores)
        if avg_score >= 90:
            report["overall_readiness"] = "ready"
        elif avg_score >= 70:
            report["overall_readiness"] = "conditionally_ready"
        elif avg_score >= 50:
            report["overall_readiness"] = "needs_work"
        else:
            report["overall_readiness"] = "not_ready"
        report["overall_score"] = round(avg_score, 1)

    return report


def main():
    parser = argparse.ArgumentParser(
        description="SOC 2 Type II Audit Preparation Agent"
    )
    parser.add_argument(
        "--action",
        choices=[
            "gap-assessment", "collect-aws-iam", "collect-aws-cloudtrail",
            "collect-aws-s3", "collect-github-changes", "collect-branch-protection",
            "compliance-checks", "remediation-plan", "evidence-manifest",
            "readiness-report", "list-criteria", "full-assessment",
        ],
        default="list-criteria",
    )
    parser.add_argument("--output", default="soc2_report.json")
    parser.add_argument("--github-org")
    parser.add_argument("--github-repo")
    parser.add_argument("--github-token", default=os.getenv("GITHUB_TOKEN"))
    parser.add_argument("--audit-start", default="2025-04-01")
    parser.add_argument("--audit-end", default="2026-03-31")
    parser.add_argument("--controls-file")
    parser.add_argument("--aws", action="store_true")
    args = parser.parse_args()

    report = {"generated_at": datetime.now(timezone.utc).isoformat(), "action": args.action}

    if args.action == "list-criteria":
        print("[*] AICPA Trust Services Criteria - Common Criteria (CC1-CC9):\n")
        for cc_id, cc_data in TRUST_SERVICES_CRITERIA.items():
            print("  " + cc_id + ": " + cc_data["name"])
            for ctrl_id, ctrl_desc in cc_data["controls"].items():
                print("       " + ctrl_id + ": " + ctrl_desc[:70])
            print()
        report["criteria"] = TRUST_SERVICES_CRITERIA

    elif args.action == "gap-assessment":
        controls_status = {}
        if args.controls_file and os.path.exists(args.controls_file):
            with open(args.controls_file) as f:
                controls_status = json.load(f)
        else:
            print("[!] No controls file provided. Run with --controls-file")
        result = perform_gap_assessment(controls_status)
        report["gap_assessment"] = result
        s = result["summary"]
        print("[+] Readiness Score: " + str(s["readiness_score"]) + "%")
        print("[+] Implemented: " + str(s["implemented"]) + "/" + str(s["total_controls"]))
        print("[+] Gaps: " + str(len(result["gaps"])))

    elif args.action == "collect-aws-iam":
        print("[+] Collecting AWS IAM evidence (CC6)...")
        result = collect_aws_iam_evidence()
        report["aws_iam_evidence"] = result

    elif args.action == "collect-aws-cloudtrail":
        print("[+] Collecting AWS CloudTrail evidence (CC7)...")
        result = collect_aws_cloudtrail_evidence()
        report["aws_cloudtrail_evidence"] = result

    elif args.action == "collect-aws-s3":
        print("[+] Collecting AWS S3 evidence (CC6)...")
        result = collect_aws_s3_public_access_evidence()
        report["aws_s3_evidence"] = result

    elif args.action == "collect-github-changes":
        if not all([args.github_org, args.github_repo, args.github_token]):
            print("[-] --github-org, --github-repo, and --github-token required")
            sys.exit(1)
        print("[+] Collecting GitHub change evidence...")
        result = collect_github_change_management_evidence(
            args.github_org, args.github_repo, args.github_token,
            since=args.audit_start,
        )
        report["github_change_evidence"] = result

    elif args.action == "collect-branch-protection":
        if not all([args.github_org, args.github_repo, args.github_token]):
            print("[-] --github-org, --github-repo, and --github-token required")
            sys.exit(1)
        result = collect_branch_protection_evidence(
            args.github_org, args.github_repo, args.github_token,
        )
        report["branch_protection_evidence"] = result

    elif args.action == "compliance-checks":
        print("[+] Running continuous compliance checks...")
        result = run_continuous_compliance_checks(aws_enabled=args.aws)
        report["compliance_checks"] = result
        print("[+] Compliance Score: " + str(result["compliance_score"]) + "%")
        for check in result["checks"]:
            p = check.get("passing")
            status = "PASS" if p is True else "FAIL" if p is False else "ERROR"
            print("    [" + status + "] " + check["control"] + ": " + check["check"])

    elif args.action == "remediation-plan":
        controls_status = {}
        if args.controls_file and os.path.exists(args.controls_file):
            with open(args.controls_file) as f:
                controls_status = json.load(f)
        gap = perform_gap_assessment(controls_status)
        plan = generate_remediation_plan(gap)
        report["remediation_plan"] = plan
        print("[+] Remediation plan: " + str(plan["total_gaps"]) + " items")
        for item in plan["remediation_items"]:
            print("    " + str(item["priority"]) + ". [" + item["severity"].upper() + "] " + item["control"])

    elif args.action == "evidence-manifest":
        manifest = generate_evidence_manifest(args.audit_start, args.audit_end)
        report["evidence_manifest"] = manifest
        print("[+] Evidence manifest: " + str(manifest["total_evidence_items"]) + " items")
        for cc_id, pkg in manifest["evidence_packages"].items():
            print("    " + cc_id + " (" + pkg["criteria_name"] + "): " + str(pkg["item_count"]) + " items")

    elif args.action == "readiness-report":
        controls_status = {}
        if args.controls_file and os.path.exists(args.controls_file):
            with open(args.controls_file) as f:
                controls_status = json.load(f)
        gap = perform_gap_assessment(controls_status)
        compliance = run_continuous_compliance_checks(aws_enabled=args.aws)
        readiness = generate_readiness_report(gap_assessment=gap, compliance_checks=compliance)
        report["readiness_report"] = readiness
        print("[+] Overall Readiness: " + readiness["overall_readiness"].upper())

    elif args.action == "full-assessment":
        print("[+] Running full SOC 2 Type II assessment...\n")
        controls_status = {}
        if args.controls_file and os.path.exists(args.controls_file):
            with open(args.controls_file) as f:
                controls_status = json.load(f)
        gap = perform_gap_assessment(controls_status)
        report["gap_assessment"] = gap
        print("[+] Gap Assessment - Readiness: " + str(gap["summary"]["readiness_score"]) + "%")

        manifest = generate_evidence_manifest(args.audit_start, args.audit_end)
        report["evidence_manifest"] = manifest
        print("[+] Evidence Manifest: " + str(manifest["total_evidence_items"]) + " items")

        compliance = run_continuous_compliance_checks(aws_enabled=args.aws)
        report["compliance_checks"] = compliance
        print("[+] Compliance Checks: " + str(compliance["compliance_score"]) + "%")

        plan = generate_remediation_plan(gap)
        report["remediation_plan"] = plan
        print("[+] Remediation Items: " + str(plan["total_gaps"]))

        readiness = generate_readiness_report(gap_assessment=gap, compliance_checks=compliance)
        report["readiness_report"] = readiness
        print("\n[+] OVERALL READINESS: " + readiness["overall_readiness"].upper())
        if "overall_score" in readiness:
            print("[+] OVERALL SCORE: " + str(readiness["overall_score"]) + "%")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print("\n[+] Report saved to " + args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-soc2-type2-audit-preparation/scripts/process.py =====
#!/usr/bin/env python3
"""
SOC 2 Type II Audit Preparation Automation

Automates control mapping to Trust Services Criteria, evidence tracking,
readiness assessment, and audit preparation for SOC 2 Type II examinations.
"""

import json
import csv
import os
from datetime import datetime, timedelta
from pathlib import Path
from dataclasses import dataclass, field, asdict
from enum import Enum
from typing import Optional


class TSCCategory(Enum):
    SECURITY = "Security (Common Criteria)"
    AVAILABILITY = "Availability"
    PROCESSING_INTEGRITY = "Processing Integrity"
    CONFIDENTIALITY = "Confidentiality"
    PRIVACY = "Privacy"


class ControlFrequency(Enum):
    CONTINUOUS = "Continuous"
    DAILY = "Daily"
    WEEKLY = "Weekly"
    MONTHLY = "Monthly"
    QUARTERLY = "Quarterly"
    SEMI_ANNUAL = "Semi-Annual"
    ANNUAL = "Annual"
    AS_NEEDED = "As Needed"


class EvidenceStatus(Enum):
    COLLECTED = "Collected"
    PENDING = "Pending"
    MISSING = "Missing"
    NOT_DUE = "Not Due"


class ControlTestResult(Enum):
    NO_EXCEPTION = "No Exception"
    EXCEPTION_NOTED = "Exception Noted"
    NOT_TESTED = "Not Tested"


# Trust Services Criteria - Common Criteria (Security)
TSC_CRITERIA = {
    "CC1.1": {"series": "CC1", "title": "Demonstrates commitment to integrity and ethical values", "category": "Security"},
    "CC1.2": {"series": "CC1", "title": "Board exercises oversight responsibility", "category": "Security"},
    "CC1.3": {"series": "CC1", "title": "Management establishes structures, reporting lines, and authorities", "category": "Security"},
    "CC1.4": {"series": "CC1", "title": "Demonstrates commitment to attract, develop, and retain competent individuals", "category": "Security"},
    "CC1.5": {"series": "CC1", "title": "Holds individuals accountable for internal control responsibilities", "category": "Security"},
    "CC2.1": {"series": "CC2", "title": "Obtains or generates relevant, quality information", "category": "Security"},
    "CC2.2": {"series": "CC2", "title": "Internally communicates information supporting internal control", "category": "Security"},
    "CC2.3": {"series": "CC2", "title": "Communicates with external parties", "category": "Security"},
    "CC3.1": {"series": "CC3", "title": "Specifies objectives with sufficient clarity", "category": "Security"},
    "CC3.2": {"series": "CC3", "title": "Identifies risks to the achievement of objectives", "category": "Security"},
    "CC3.3": {"series": "CC3", "title": "Considers potential for fraud", "category": "Security"},
    "CC3.4": {"series": "CC3", "title": "Identifies and assesses changes that could impact internal control", "category": "Security"},
    "CC4.1": {"series": "CC4", "title": "Selects, develops, and performs ongoing and separate evaluations", "category": "Security"},
    "CC4.2": {"series": "CC4", "title": "Evaluates and communicates internal control deficiencies", "category": "Security"},
    "CC5.1": {"series": "CC5", "title": "Selects and develops control activities", "category": "Security"},
    "CC5.2": {"series": "CC5", "title": "Selects and develops general controls over technology", "category": "Security"},
    "CC5.3": {"series": "CC5", "title": "Deploys through policies that establish expectations and procedures", "category": "Security"},
    "CC6.1": {"series": "CC6", "title": "Logical access security software, infrastructure, and architectures", "category": "Security"},
    "CC6.2": {"series": "CC6", "title": "Prior to credential issuance, registration and authorization", "category": "Security"},
    "CC6.3": {"series": "CC6", "title": "Access removal and modification upon changes", "category": "Security"},
    "CC6.4": {"series": "CC6", "title": "Physical access restrictions to facilities", "category": "Security"},
    "CC6.5": {"series": "CC6", "title": "Discontinuation of physical access", "category": "Security"},
    "CC6.6": {"series": "CC6", "title": "Logical access security against threats from external sources", "category": "Security"},
    "CC6.7": {"series": "CC6", "title": "Restricts transmission, movement, and removal of information", "category": "Security"},
    "CC6.8": {"series": "CC6", "title": "Controls against threats from unauthorized or malicious code", "category": "Security"},
    "CC7.1": {"series": "CC7", "title": "Detection and monitoring procedures for anomalies", "category": "Security"},
    "CC7.2": {"series": "CC7", "title": "Monitors system components for anomalies", "category": "Security"},
    "CC7.3": {"series": "CC7", "title": "Evaluates security events to determine incidents", "category": "Security"},
    "CC7.4": {"series": "CC7", "title": "Responds to identified security incidents", "category": "Security"},
    "CC7.5": {"series": "CC7", "title": "Identifies and remediates vulnerabilities", "category": "Security"},
    "CC8.1": {"series": "CC8", "title": "Authorizes, designs, develops, tests, approves, and implements changes", "category": "Security"},
    "CC9.1": {"series": "CC9", "title": "Identifies and assesses risk mitigation activities", "category": "Security"},
    "CC9.2": {"series": "CC9", "title": "Assesses and manages risks associated with vendors and partners", "category": "Security"},
    # Availability criteria
    "A1.1": {"series": "A1", "title": "Maintains, monitors, and evaluates current processing capacity", "category": "Availability"},
    "A1.2": {"series": "A1", "title": "Environmental protections, software, data backup and recovery", "category": "Availability"},
    "A1.3": {"series": "A1", "title": "Tests recovery plan procedures", "category": "Availability"},
    # Confidentiality criteria
    "C1.1": {"series": "C1", "title": "Identifies and maintains confidential information", "category": "Confidentiality"},
    "C1.2": {"series": "C1", "title": "Disposes of confidential information", "category": "Confidentiality"},
    # Processing Integrity criteria
    "PI1.1": {"series": "PI1", "title": "Obtains or generates relevant, quality information", "category": "Processing Integrity"},
    "PI1.2": {"series": "PI1", "title": "System inputs are complete, accurate, and timely", "category": "Processing Integrity"},
    "PI1.3": {"series": "PI1", "title": "Processing is complete, valid, accurate, timely, and authorized", "category": "Processing Integrity"},
    "PI1.4": {"series": "PI1", "title": "System output is complete, valid, accurate, timely, and authorized", "category": "Processing Integrity"},
    "PI1.5": {"series": "PI1", "title": "Data stored is complete, valid, accurate, timely, and authorized", "category": "Processing Integrity"},
}


@dataclass
class Control:
    control_id: str
    description: str
    tsc_criteria: list
    control_type: str  # Preventive, Detective, Corrective
    frequency: str
    owner: str
    evidence_type: str
    automated: bool = False
    test_result: str = ControlTestResult.NOT_TESTED.value
    exceptions: list = field(default_factory=list)


@dataclass
class EvidenceItem:
    evidence_id: str
    control_id: str
    tsc_criterion: str
    description: str
    period_start: str
    period_end: str
    collection_date: str = ""
    status: str = EvidenceStatus.PENDING.value
    file_reference: str = ""
    notes: str = ""


@dataclass
class ReadinessItem:
    category: str
    item: str
    status: bool
    notes: str = ""
    remediation: str = ""
    due_date: str = ""


class SOC2AuditPrep:
    """Manages SOC 2 Type II audit preparation."""

    def __init__(self, output_dir: str = "./soc2_output",
                 audit_start: str = "", audit_end: str = ""):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.controls: list[Control] = []
        self.evidence: list[EvidenceItem] = []
        self.audit_start = audit_start or (datetime.now() - timedelta(days=365)).strftime("%Y-%m-%d")
        self.audit_end = audit_end or datetime.now().strftime("%Y-%m-%d")
        self.selected_categories: list[str] = ["Security"]

    def select_tsc_categories(self, categories: list[str]) -> dict:
        """Select applicable TSC categories for the audit."""
        print("\n" + "=" * 70)
        print("TSC CATEGORY SELECTION")
        print("=" * 70)

        self.selected_categories = categories
        applicable_criteria = {}

        for crit_id, crit_info in TSC_CRITERIA.items():
            if crit_info["category"] in categories:
                applicable_criteria[crit_id] = crit_info

        print(f"\n  Selected Categories: {', '.join(categories)}")
        print(f"  Applicable Criteria: {len(applicable_criteria)}")
        print(f"  Audit Period: {self.audit_start} to {self.audit_end}")

        for cat in categories:
            count = sum(1 for c in applicable_criteria.values() if c["category"] == cat)
            print(f"    {cat}: {count} criteria")

        return applicable_criteria

    def create_control_matrix(self, controls: list[dict]) -> list[Control]:
        """Create control matrix mapping controls to TSC criteria."""
        print("\n" + "=" * 70)
        print("CONTROL MATRIX")
        print("=" * 70)

        for ctrl_data in controls:
            ctrl = Control(**ctrl_data)
            self.controls.append(ctrl)
            criteria_str = ", ".join(ctrl.tsc_criteria)
            print(f"\n  [{ctrl.control_id}] {ctrl.description[:60]}...")
            print(f"    TSC: {criteria_str}")
            print(f"    Type: {ctrl.control_type} | Frequency: {ctrl.frequency}")
            print(f"    Owner: {ctrl.owner} | Automated: {ctrl.automated}")

        # Coverage analysis
        covered_criteria = set()
        for ctrl in self.controls:
            covered_criteria.update(ctrl.tsc_criteria)

        applicable_criteria = {
            k for k, v in TSC_CRITERIA.items()
            if v["category"] in self.selected_categories
        }
        uncovered = applicable_criteria - covered_criteria

        print(f"\n  Total Controls: {len(self.controls)}")
        print(f"  Criteria Covered: {len(covered_criteria)} / {len(applicable_criteria)}")
        if uncovered:
            print(f"  GAPS - Uncovered Criteria: {', '.join(sorted(uncovered))}")
        else:
            print(f"  All applicable criteria covered")

        # Save control matrix
        matrix_path = self.output_dir / "control_matrix.json"
        with open(matrix_path, "w") as f:
            json.dump([asdict(c) for c in self.controls], f, indent=2)

        # CSV for auditor
        csv_path = self.output_dir / "control_matrix.csv"
        with open(csv_path, "w", newline="") as f:
            writer = csv.writer(f)
            writer.writerow([
                "Control ID", "Description", "TSC Criteria", "Control Type",
                "Frequency", "Owner", "Evidence Type", "Automated",
                "Test Result", "Exceptions"
            ])
            for ctrl in self.controls:
                writer.writerow([
                    ctrl.control_id,
                    ctrl.description,
                    "; ".join(ctrl.tsc_criteria),
                    ctrl.control_type,
                    ctrl.frequency,
                    ctrl.owner,
                    ctrl.evidence_type,
                    "Yes" if ctrl.automated else "No",
                    ctrl.test_result,
                    "; ".join(ctrl.exceptions) if ctrl.exceptions else "",
                ])

        print(f"  Control Matrix saved to: {matrix_path}")
        return self.controls

    def track_evidence(self, evidence_items: list[dict]) -> dict:
        """Track evidence collection status for audit period."""
        print("\n" + "=" * 70)
        print("EVIDENCE COLLECTION TRACKER")
        print("=" * 70)

        for item_data in evidence_items:
            item = EvidenceItem(**item_data)
            self.evidence.append(item)

        # Status summary
        status_counts = {}
        for item in self.evidence:
            status_counts.setdefault(item.status, 0)
            status_counts[item.status] += 1

        total = len(self.evidence)
        collected = status_counts.get(EvidenceStatus.COLLECTED.value, 0)
        pending = status_counts.get(EvidenceStatus.PENDING.value, 0)
        missing = status_counts.get(EvidenceStatus.MISSING.value, 0)

        print(f"\n  Evidence Items: {total}")
        print(f"  Collected: {collected} ({collected/total*100:.1f}%)" if total > 0 else "")
        print(f"  Pending: {pending}")
        print(f"  Missing: {missing}")

        if missing > 0:
            print(f"\n  ALERT: {missing} evidence items are missing!")
            for item in self.evidence:
                if item.status == EvidenceStatus.MISSING.value:
                    print(f"    - [{item.evidence_id}] {item.description} (Control: {item.control_id})")

        # Save evidence tracker
        tracker_path = self.output_dir / "evidence_tracker.json"
        with open(tracker_path, "w") as f:
            json.dump([asdict(e) for e in self.evidence], f, indent=2)

        print(f"\n  Evidence Tracker saved to: {tracker_path}")
        return status_counts

    def assess_readiness(self) -> dict:
        """Perform SOC 2 Type II audit readiness assessment."""
        print("\n" + "=" * 70)
        print("SOC 2 TYPE II READINESS ASSESSMENT")
        print("=" * 70)

        readiness_checks = [
            ReadinessItem("Documentation", "System Description Document completed", False),
            ReadinessItem("Documentation", "Management Assertion Letter drafted", False),
            ReadinessItem("Documentation", "Control matrix documented and reviewed", bool(self.controls)),
            ReadinessItem("Documentation", "Security policies current and approved", False),
            ReadinessItem("Documentation", "Risk assessment completed within audit period", False),

            ReadinessItem("Controls", "All TSC criteria mapped to at least one control", False),
            ReadinessItem("Controls", "Control owners identified and briefed", False),
            ReadinessItem("Controls", "No control design gaps identified", False),
            ReadinessItem("Controls", "Compensating controls documented for exceptions", False),

            ReadinessItem("Evidence", "Evidence collection covers full audit period", False),
            ReadinessItem("Evidence", "Quarterly access reviews completed", False),
            ReadinessItem("Evidence", "Annual penetration test completed", False),
            ReadinessItem("Evidence", "Security awareness training records available", False),
            ReadinessItem("Evidence", "Change management tickets with approvals", False),
            ReadinessItem("Evidence", "Incident response logs available", False),
            ReadinessItem("Evidence", "Vulnerability scan reports for audit period", False),

            ReadinessItem("Operations", "Background checks completed for new hires", False),
            ReadinessItem("Operations", "MFA enabled for all in-scope systems", False),
            ReadinessItem("Operations", "Encryption at rest and in transit verified", False),
            ReadinessItem("Operations", "Backup and recovery testing documented", False),

            ReadinessItem("Vendor", "Subservice organizations identified", False),
            ReadinessItem("Vendor", "Carve-out or inclusive method determined", False),
            ReadinessItem("Vendor", "CUECs documented", False),
            ReadinessItem("Vendor", "Vendor SOC reports reviewed", False),
        ]

        # Auto-check based on data
        if self.controls:
            readiness_checks[5].status = True  # Controls mapped

        if self.evidence:
            collected = sum(1 for e in self.evidence if e.status == EvidenceStatus.COLLECTED.value)
            if collected == len(self.evidence) and len(self.evidence) > 0:
                readiness_checks[9].status = True

        # Display results
        categories = {}
        for check in readiness_checks:
            categories.setdefault(check.category, [])
            categories[check.category].append(check)

        total_checks = len(readiness_checks)
        passed = sum(1 for c in readiness_checks if c.status)
        pct = (passed / total_checks * 100) if total_checks > 0 else 0

        print(f"\n  Readiness Score: {passed}/{total_checks} ({pct:.1f}%)")

        for cat, checks in categories.items():
            print(f"\n  {cat}:")
            for check in checks:
                icon = "[PASS]" if check.status else "[FAIL]"
                print(f"    {icon} {check.item}")

        # Recommendation
        if pct >= 90:
            print(f"\n  RECOMMENDATION: Ready for audit. Schedule with audit firm.")
        elif pct >= 70:
            print(f"\n  RECOMMENDATION: Address remaining items within 2-4 weeks.")
        else:
            print(f"\n  RECOMMENDATION: Significant gaps remain. Delay audit until addressed.")

        # Save readiness report
        report = {
            "date": datetime.now().isoformat(),
            "audit_period": f"{self.audit_start} to {self.audit_end}",
            "readiness_percentage": pct,
            "checks": [asdict(c) for c in readiness_checks],
        }
        report_path = self.output_dir / "readiness_assessment.json"
        with open(report_path, "w") as f:
            json.dump(report, f, indent=2)

        print(f"\n  Readiness Report saved to: {report_path}")
        return report

    def generate_audit_summary(self) -> dict:
        """Generate overall audit preparation summary dashboard."""
        print("\n" + "=" * 70)
        print("SOC 2 TYPE II AUDIT PREPARATION SUMMARY")
        print("=" * 70)

        summary = {
            "generated": datetime.now().isoformat(),
            "audit_period": f"{self.audit_start} to {self.audit_end}",
            "tsc_categories": self.selected_categories,
            "controls": {
                "total": len(self.controls),
                "preventive": sum(1 for c in self.controls if c.control_type == "Preventive"),
                "detective": sum(1 for c in self.controls if c.control_type == "Detective"),
                "corrective": sum(1 for c in self.controls if c.control_type == "Corrective"),
                "automated": sum(1 for c in self.controls if c.automated),
                "manual": sum(1 for c in self.controls if not c.automated),
            },
            "evidence": {
                "total": len(self.evidence),
                "collected": sum(1 for e in self.evidence if e.status == EvidenceStatus.COLLECTED.value),
                "pending": sum(1 for e in self.evidence if e.status == EvidenceStatus.PENDING.value),
                "missing": sum(1 for e in self.evidence if e.status == EvidenceStatus.MISSING.value),
            },
        }

        print(f"\n  Audit Period: {summary['audit_period']}")
        print(f"  TSC Categories: {', '.join(summary['tsc_categories'])}")
        print(f"\n  Controls: {summary['controls']['total']}")
        print(f"    Preventive: {summary['controls']['preventive']}")
        print(f"    Detective: {summary['controls']['detective']}")
        print(f"    Corrective: {summary['controls']['corrective']}")
        print(f"    Automated: {summary['controls']['automated']}")
        print(f"\n  Evidence: {summary['evidence']['total']}")
        print(f"    Collected: {summary['evidence']['collected']}")
        print(f"    Pending: {summary['evidence']['pending']}")
        print(f"    Missing: {summary['evidence']['missing']}")

        summary_path = self.output_dir / "audit_summary.json"
        with open(summary_path, "w") as f:
            json.dump(summary, f, indent=2)

        print(f"\n  Summary saved to: {summary_path}")
        return summary


def main():
    """Run SOC 2 Type II audit preparation."""
    prep = SOC2AuditPrep(
        audit_start="2024-01-01",
        audit_end="2024-12-31"
    )

    # Select TSC categories
    criteria = prep.select_tsc_categories(["Security", "Availability", "Confidentiality"])

    # Create control matrix
    sample_controls = [
        {
            "control_id": "CTRL-001",
            "description": "Multi-factor authentication is required for all access to production systems and sensitive data",
            "tsc_criteria": ["CC6.1", "CC6.6"],
            "control_type": "Preventive",
            "frequency": ControlFrequency.CONTINUOUS.value,
            "owner": "IT Security Manager",
            "evidence_type": "System configuration screenshot",
            "automated": True,
        },
        {
            "control_id": "CTRL-002",
            "description": "Quarterly user access reviews are performed for all in-scope systems with formal approval",
            "tsc_criteria": ["CC6.2", "CC6.3"],
            "control_type": "Detective",
            "frequency": ControlFrequency.QUARTERLY.value,
            "owner": "IT Manager",
            "evidence_type": "Access review completion report with approvals",
            "automated": False,
        },
        {
            "control_id": "CTRL-003",
            "description": "Security events are monitored 24/7 through SIEM with automated alerting for critical events",
            "tsc_criteria": ["CC7.1", "CC7.2", "CC7.3"],
            "control_type": "Detective",
            "frequency": ControlFrequency.CONTINUOUS.value,
            "owner": "SOC Team Lead",
            "evidence_type": "SIEM dashboard and alert configuration",
            "automated": True,
        },
        {
            "control_id": "CTRL-004",
            "description": "All changes to production systems follow change management process with peer review and management approval",
            "tsc_criteria": ["CC8.1"],
            "control_type": "Preventive",
            "frequency": ControlFrequency.AS_NEEDED.value,
            "owner": "Engineering Manager",
            "evidence_type": "Change tickets with approval chain",
            "automated": False,
        },
        {
            "control_id": "CTRL-005",
            "description": "Annual penetration testing is conducted by qualified third party with findings remediated",
            "tsc_criteria": ["CC7.1", "CC7.5"],
            "control_type": "Detective",
            "frequency": ControlFrequency.ANNUAL.value,
            "owner": "CISO",
            "evidence_type": "Penetration test report and remediation tracker",
            "automated": False,
        },
        {
            "control_id": "CTRL-006",
            "description": "Data at rest is encrypted using AES-256 and data in transit uses TLS 1.2 or higher",
            "tsc_criteria": ["CC6.1", "CC6.7", "C1.1"],
            "control_type": "Preventive",
            "frequency": ControlFrequency.CONTINUOUS.value,
            "owner": "Cloud Architect",
            "evidence_type": "Encryption configuration verification",
            "automated": True,
        },
        {
            "control_id": "CTRL-007",
            "description": "Incident response process is documented, tested annually, and incidents are tracked to resolution",
            "tsc_criteria": ["CC7.3", "CC7.4"],
            "control_type": "Corrective",
            "frequency": ControlFrequency.AS_NEEDED.value,
            "owner": "Security Incident Manager",
            "evidence_type": "IR plan, tabletop exercise report, incident tickets",
            "automated": False,
        },
        {
            "control_id": "CTRL-008",
            "description": "Vendor security assessments are performed prior to engagement and annually thereafter",
            "tsc_criteria": ["CC9.2"],
            "control_type": "Preventive",
            "frequency": ControlFrequency.ANNUAL.value,
            "owner": "Vendor Management",
            "evidence_type": "Vendor assessment questionnaires and SOC reports",
            "automated": False,
        },
    ]
    controls = prep.create_control_matrix(sample_controls)

    # Track evidence
    sample_evidence = [
        {
            "evidence_id": "EVD-001",
            "control_id": "CTRL-001",
            "tsc_criterion": "CC6.1",
            "description": "MFA configuration screenshots for production systems",
            "period_start": "2024-01-01",
            "period_end": "2024-12-31",
            "status": EvidenceStatus.COLLECTED.value,
            "collection_date": "2024-12-15",
            "file_reference": "evidence/CC6.1/mfa_config_2024.pdf",
        },
        {
            "evidence_id": "EVD-002",
            "control_id": "CTRL-002",
            "tsc_criterion": "CC6.3",
            "description": "Q1 2024 User Access Review - All Systems",
            "period_start": "2024-01-01",
            "period_end": "2024-03-31",
            "status": EvidenceStatus.COLLECTED.value,
            "collection_date": "2024-04-05",
            "file_reference": "evidence/CC6.3/q1_access_review.pdf",
        },
        {
            "evidence_id": "EVD-003",
            "control_id": "CTRL-005",
            "tsc_criterion": "CC7.1",
            "description": "Annual penetration test report",
            "period_start": "2024-01-01",
            "period_end": "2024-12-31",
            "status": EvidenceStatus.COLLECTED.value,
            "collection_date": "2024-10-30",
            "file_reference": "evidence/CC7.1/pentest_report_2024.pdf",
        },
        {
            "evidence_id": "EVD-004",
            "control_id": "CTRL-002",
            "tsc_criterion": "CC6.3",
            "description": "Q4 2024 User Access Review - All Systems",
            "period_start": "2024-10-01",
            "period_end": "2024-12-31",
            "status": EvidenceStatus.PENDING.value,
        },
    ]
    prep.track_evidence(sample_evidence)

    # Readiness assessment
    prep.assess_readiness()

    # Generate summary
    prep.generate_audit_summary()

    print("\n" + "=" * 70)
    print("SOC 2 TYPE II AUDIT PREPARATION COMPLETE")
    print("=" * 70)


if __name__ == "__main__":
    main()

===== END FILE =====
