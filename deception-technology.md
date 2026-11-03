

===== FILE: skills/deploying-active-directory-honeytokens/SKILL.md =====
---
name: deploying-active-directory-honeytokens
description: 'Deploys deception-based honeytokens in Active Directory including fake
  privileged accounts with AdminCount=1, fake SPNs for Kerberoasting detection (honeyroasting),
  decoy GPOs with cpassword traps, and fake BloodHound paths. Monitors Windows Security
  Event IDs 4769, 4625, 4662, 5136 for honeytoken interaction. Use when implementing
  AD deception defenses for detecting lateral movement, credential theft, and reconnaissance.

  '
domain: cybersecurity
subdomain: deception-technology
tags:
- active-directory
- honeytokens
- kerberoasting
- deception
- detection
- bloodhound
- gpo
version: '1.0'
author: mukul975
license: Apache-2.0
nist_csf:
- DE.CM-01
- DE.AE-06
- PR.IR-01
mitre_attack:
- T1558.003
- T1558.004
- T1110.003
- T1003.006
- T1552.006
---

# Deploying Active Directory Honeytokens

## When to Use

- When deploying deception-based detection in Active Directory environments
- When detecting Kerberoasting attacks via fake SPN honeytokens (honeyroasting)
- When creating tripwire accounts to detect credential theft and lateral movement
- When building decoy GPOs to detect Group Policy Preference password harvesting
- When creating deceptive BloodHound paths to misdirect and detect attackers
- When supplementing existing AD monitoring with high-fidelity detection signals

## Prerequisites

- Domain Admin or delegated AD administration privileges
- Active Directory domain (Windows Server 2016+ recommended)
- Windows Event Log forwarding to SIEM (Splunk, Sentinel, Elastic)
- PowerShell 5.1+ with ActiveDirectory module
- Group Policy Management Console (GPMC)
- Understanding of AD security, Kerberos, and BloodHound attack paths

## Background

### Why AD Honeytokens

Traditional signature-based detection misses novel attack techniques. Honeytokens
provide high-fidelity detection with near-zero false positives because any interaction
with a decoy object is inherently suspicious. In Active Directory:

- **Fake privileged accounts** detect credential dumping (DCSync, NTDS.dit extraction)
- **Fake SPNs** detect Kerberoasting reconnaissance (TGS requests for nonexistent services)
- **Decoy GPOs** detect Group Policy Preference password harvesting
- **Fake BloodHound paths** mislead attackers using graph-based AD analysis

### Key Detection Event IDs

| Event ID | Description | Honeytoken Use |
|----------|-------------|----------------|
| 4769 | Kerberos TGS ticket requested | Detect Kerberoast against honey SPN |
| 4625 | Failed logon attempt | Detect use of fake credentials from decoy GPO |
| 4662 | Directory service object accessed | Detect DACL read on honeytoken user |
| 5136 | Directory service object modified | Detect modification of decoy GPO |
| 5137 | Directory service object created | Detect GPO creation mimicking decoy |
| 4768 | Kerberos TGT requested | Detect AS-REP roasting of honey account |

### Making Honeytokens Realistic

Per Trimarc Security research, effective honeytokens must appear legitimate:

- **Age the account**: Repurpose old inactive accounts (10-15 year old accounts in
  similarly aged domains appear authentic)
- **Set AdminCount=1**: Flags the account as having elevated AD rights, making it
  an attractive Kerberoasting target
- **Use realistic naming**: Match organizational naming conventions (svc_sqlbackup,
  admin.maintenance, svc_exchange_legacy)
- **Set old password date**: Password age of 10+ years with an SPN looks like a
  high-value, neglected service account to attackers
- **Add group memberships**: Place in visible groups like "Remote Desktop Users" or
  a custom "Backup Operators" to increase attacker interest
- **Avoid detection tells**: Attackers check creation date vs. last logon vs.
  password change date for consistency

## Instructions

### Step 1: Deploy Fake Privileged Admin Account

Create a honeytoken account that mimics a legacy privileged service account.

```powershell
# Import the deployment module
Import-Module .\scripts\Deploy-ADHoneytokens.ps1

# Create a honeytoken admin account
$honeyAdmin = New-HoneytokenAdmin `
    -SamAccountName "svc_sqlbackup_legacy" `
    -DisplayName "SQL Backup Service (Legacy)" `
    -Description "Legacy SQL Server backup service account - DO NOT DELETE" `
    -OU "OU=Service Accounts,DC=corp,DC=example,DC=com" `
    -PasswordLength 128 `
    -SetAdminCount $true

Write-Host "Honeytoken admin created: $($honeyAdmin.DistinguishedName)"
```

### Step 2: Deploy Fake SPN for Kerberoasting Detection

Assign a realistic but fake SPN to the honeytoken account. Any TGS request
for this SPN is definitively malicious (honeyroasting).

```powershell
# Add fake SPN to honeytoken account
$honeySPN = Add-HoneytokenSPN `
    -SamAccountName "svc_sqlbackup_legacy" `
    -ServiceClass "MSSQLSvc" `
    -Hostname "sql-legacy-bak01.corp.example.com" `
    -Port 1433

Write-Host "Honey SPN registered: $($honeySPN.SPN)"
Write-Host "Monitor Event ID 4769 for TGS requests targeting this SPN"
```

### Step 3: Deploy Decoy GPO with Credential Trap

Create a fake GPO in SYSVOL with an embedded cpassword (Group Policy Preference
password). Attackers using tools like Get-GPPPassword or gpp-decrypt will find
and attempt to use these credentials, triggering detection.

```powershell
# Create decoy GPO with cpassword trap
$decoyGPO = New-DecoyGPO `
    -GPOName "Server Maintenance Policy (Legacy)" `
    -DecoyUsername "admin_maintenance" `
    -DecoyDomain "CORP" `
    -SYSVOLPath "\\corp.example.com\SYSVOL\corp.example.com\Policies" `
    -EnableAuditSACL $true

Write-Host "Decoy GPO created: $($decoyGPO.GPOGuid)"
Write-Host "SACL audit enabled - any read attempt will generate Event ID 4663"
```

### Step 4: Create Deceptive BloodHound Paths

Set ACL permissions that create fake attack paths visible to BloodHound/SharpHound
reconnaissance, leading attackers toward monitored honeytokens.

```powershell
# Create fake BloodHound attack path
$deceptivePath = New-DeceptiveBloodHoundPath `
    -HoneytokenSamAccount "svc_sqlbackup_legacy" `
    -TargetHighValueGroup "Domain Admins" `
    -IntermediateOU "OU=Service Accounts,DC=corp,DC=example,DC=com"

Write-Host "Deceptive path created: $($deceptivePath.PathDescription)"
```

### Step 5: Configure Detection Rules

Set up SIEM detection rules to alert on any honeytoken interaction.

```python
# Using the Python detection agent
from agent import ADHoneytokenMonitor

monitor = ADHoneytokenMonitor(config_path="honeytoken_config.json")

# Register all honeytokens for monitoring
monitor.register_honeytoken("svc_sqlbackup_legacy", token_type="admin_account")
monitor.register_honeytoken("MSSQLSvc/sql-legacy-bak01.corp.example.com:1433", token_type="spn")
monitor.register_honeytoken("admin_maintenance", token_type="gpo_credential")

# Generate SIEM detection rules
splunk_rules = monitor.generate_detection_rules(siem="splunk")
sentinel_rules = monitor.generate_detection_rules(siem="sentinel")
sigma_rules = monitor.generate_detection_rules(siem="sigma")

for rule in sigma_rules:
    print(f"Rule: {rule['title']}")
    print(f"  Detection: {rule['detection_logic']}")
```

### Step 6: Validate Deployment

Test the honeytokens to ensure detection fires correctly.

```powershell
# Validate honeytoken deployment
$validation = Test-HoneytokenDeployment `
    -SamAccountName "svc_sqlbackup_legacy" `
    -ValidateAdminCount `
    -ValidateSPN `
    -ValidateGPODecoy `
    -ValidateAuditPolicy

$validation | Format-Table Check, Status, Details -AutoSize
```

## Examples

### Full Deployment Pipeline

```powershell
Import-Module .\scripts\Deploy-ADHoneytokens.ps1

# Deploy complete honeytoken suite
$deployment = Deploy-FullHoneytokenSuite `
    -Environment "Production" `
    -ServiceAccountOU "OU=Service Accounts,DC=corp,DC=example,DC=com" `
    -SYSVOLPath "\\corp.example.com\SYSVOL\corp.example.com\Policies" `
    -TokenCount 3 `
    -IncludeSPN $true `
    -IncludeGPODecoy $true `
    -IncludeBloodHoundPath $true `
    -SIEMType "Splunk"

# Output deployment report
$deployment.Tokens | Format-Table Name, Type, SPN, DetectionRule -AutoSize
$deployment | Export-Csv "honeytoken_deployment_report.csv" -NoTypeInformation
```

### Kerberoasting Detection Query (Splunk)

```spl
index=wineventlog EventCode=4769 ServiceName="svc_sqlbackup_legacy"
| eval alert_severity="critical"
| eval alert_type="honeytoken_kerberoast"
| table _time, src_ip, Account_Name, ServiceName, Ticket_Encryption_Type
| sort - _time
```

### Microsoft Sentinel KQL Detection

```kql
SecurityEvent
| where EventID == 4769
| where ServiceName in ("svc_sqlbackup_legacy", "svc_exchange_legacy")
| extend AlertType = "Honeytoken Kerberoast Detected"
| project TimeGenerated, Computer, Account, ServiceName, IpAddress, TicketEncryptionType
```

## References

- Trimarc Security - The Art of the Honeypot Account: https://www.hub.trimarcsecurity.com/post/the-art-of-the-honeypot-account-making-the-unusual-look-normal
- ADSecurity.org - Detecting Kerberoasting Activity Part 2 (Honeypot): https://adsecurity.org/?p=3513
- Microsoft Defender for Identity Honeytokens: https://techcommunity.microsoft.com/blog/microsoftthreatprotectionblog/deceptive-defense-best-practices-for-identity-based-honeytokens-in-microsoft-def/3851641
- SpecterOps - Kerberoasting and AES-256: https://specterops.io/blog/2025/10/21/is-kerberoasting-still-a-risk-when-aes-256-kerberos-encryption-is-enabled/
- APT29a Blog - Deploying Honeytokens in AD: https://apt29a.blogspot.com/2019/11/deploying-honeytokens-in-active.html
- ADSecurity.org - Detecting Kerberoasting Activity: https://adsecurity.org/?p=3458

===== END FILE =====


===== FILE: skills/deploying-active-directory-honeytokens/LICENSE =====

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


===== FILE: skills/deploying-active-directory-honeytokens/references/api-reference.md =====
# API Reference: Active Directory Honeytoken Deployment

## PowerShellGenerator

Generates PowerShell scripts for AD honeytoken deployment operations.

### Methods

#### `generate_create_honeytoken_account(...)`
Generate PowerShell to create a honeytoken AD account with AdminCount=1, backdated password, group memberships, and SACL audit rules.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sam_account_name` | `str` | required | sAMAccountName for the honeytoken |
| `display_name` | `str` | required | Display name |
| `description` | `str` | required | Description field |
| `ou_dn` | `str` | required | Distinguished Name of target OU |
| `password_length` | `int` | `128` | Random password length |
| `set_admin_count` | `bool` | `True` | Set AdminCount=1 |
| `account_age_days` | `int` | `5475` | Days to backdate password (~15 years) |

**Returns:** `str` -- Complete PowerShell script.

**AD Operations Performed:**
- Creates AD user account with strong random password
- Sets AdminCount=1 (appears as privileged account to BloodHound)
- Backdates pwdLastSet to simulate aged service account
- Adds to Remote Desktop Users group
- Configures SACL audit rule (Everyone/ReadProperty/Success)

**Detection:** Event ID 4662 (directory service object accessed)

#### `generate_add_honey_spn(...)`
Generate PowerShell to add a fake SPN for Kerberoasting detection (honeyroasting).

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sam_account_name` | `str` | required | Account to add SPN to |
| `service_class` | `str` | `"MSSQLSvc"` | SPN service class |
| `hostname` | `str` | required | Fake hostname |
| `port` | `int` | `1433` | Service port |

**Returns:** `str` -- PowerShell script that registers the SPN and enables RC4+AES encryption.

**Detection:** Event ID 4769 (Kerberos TGS ticket requested) where ServiceName matches the honeytoken account. Any TGS request for this SPN is definitively malicious.

#### `generate_decoy_gpo(...)`
Generate PowerShell to create a decoy GPO with cpassword credential trap in SYSVOL.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `gpo_name` | `str` | required | GPO display name |
| `decoy_username` | `str` | required | Username in cpassword trap |
| `decoy_domain` | `str` | required | Short domain name (e.g., CORP) |
| `sysvol_path` | `str` | required | SYSVOL Policies path |
| `enable_sacl` | `bool` | `True` | Set SACL audit on GPO folder |

**Returns:** `str` -- PowerShell script that creates GPO folder structure, plants Groups.xml with cpassword, creates trap AD account with different password, and sets SACL.

**Detection Chain:**
1. Event ID 4663 (SYSVOL folder read)
2. Offline: Attacker decrypts cpassword
3. Event ID 4625 (failed logon with decoy credentials)
4. Correlation: 4663 + 4625 from same source IP = confirmed attacker

#### `generate_deceptive_bloodhound_path(...)`
Generate PowerShell to create fake BloodHound attack paths leading to monitored honeytokens.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `honeytoken_sam` | `str` | required | Honeytoken account name |
| `target_group` | `str` | `"Domain Admins"` | High-value group for deceptive path |
| `intermediate_ou` | `str` | `"OU=Service Accounts"` | OU for intermediate objects |

**Returns:** `str` -- PowerShell script that creates GenericAll ACE, deceptive intermediate group, and WriteDacl edge with deny safety net.

**BloodHound Path Created:**
```
Remote Desktop Users -[GenericAll]-> honeytoken_account
honeytoken_account -[MemberOf]-> IT-Infrastructure-Admins
honeytoken_account -[WriteDacl]-> Domain Admins (blocked by deny ACE)
```

#### `generate_validation_script(sam_account_name)`
Generate PowerShell to validate honeytoken deployment integrity.

**Checks Performed:**

| Check | Pass Criteria |
|-------|---------------|
| Account Exists | Account found in AD |
| Account Enabled | Enabled = True |
| AdminCount=1 | AdminCount attribute is 1 |
| SPN Configured | At least one SPN registered |
| Password Age | > 365 days |
| SACL Audit | At least one audit rule configured |
| Group Memberships | Lists all group memberships |
| RC4 Supported | msDS-SupportedEncryptionTypes includes 0x4 |
| Kerberos Audit | auditpol shows Kerberos TGS auditing enabled |

---

## SIEMRuleGenerator

Generates detection rules for SIEM platforms targeting honeytoken activity.

### Methods

#### `generate_detection_rules(honeytoken_accounts, honey_spns, gpo_trap_accounts, siem="sigma")`

| Parameter | Type | Description |
|-----------|------|-------------|
| `honeytoken_accounts` | `list[str]` | Account names to monitor |
| `honey_spns` | `list[str]` | SPN values to monitor |
| `gpo_trap_accounts` | `list[str]` | GPO credential trap usernames |
| `siem` | `str` | Target platform: `sigma`, `splunk`, or `sentinel` |

**Returns:** `list[dict]` -- Each rule contains `title`, `detection_logic`, and `rule` (full query text).

### Generated Rules by Platform

**Sigma Rules:**

| Rule | Event ID | MITRE Technique |
|------|----------|-----------------|
| Honeytoken Kerberoast Detected | 4769 | T1558.003 |
| Honeytoken GPO Credential Use Detected | 4624, 4625 | T1552.006 |
| Honeytoken AD Object Accessed | 4662 | T1087.002 |

**Splunk SPL Rules:**

| Rule | Description |
|------|-------------|
| Honeytoken Kerberoast Detection | Index wineventlog EventCode=4769 with ServiceName filter |
| Honeytoken GPO Credential Use | EventCode 4624/4625 with TargetUserName filter |
| Attack Chain Correlation | SYSVOL enum (4663) -> credential use (4625) by same source IP |

**Microsoft Sentinel KQL Rules:**

| Rule | Description |
|------|-------------|
| Honeytoken Kerberoast Detection | SecurityEvent EventID 4769 with ServiceName filter |
| Honeytoken GPO Credential Use | SecurityEvent EventID 4624/4625 with TargetUserName filter |

#### `export_rules(output_dir, format="json")`
Export all generated rules to files on disk.

**Returns:** `list[str]` of saved file paths.

---

## ADHoneytokenMonitor

Monitors Windows Event Logs for honeytoken interactions and generates alerts.

### Constructor
```python
ADHoneytokenMonitor(config_path=None)
```

### Methods

#### `register_honeytoken(identifier, token_type="admin_account", metadata=None)`
Register a honeytoken for monitoring.

| Token Type | Description |
|------------|-------------|
| `admin_account` | Fake privileged AD account |
| `spn` | Fake Service Principal Name |
| `gpo_credential` | Decoy GPO cpassword trap account |

#### `analyze_event_log(events)`
Analyze Windows Event Log entries for honeytoken interactions.

| Event ID | Alert Type | Severity |
|----------|------------|----------|
| 4769 | `KERBEROAST_HONEYTOKEN` | critical |
| 4624 | `HONEYTOKEN_LOGON` | critical |
| 4625 | `HONEYTOKEN_LOGON_FAILED` | critical |
| 4662 | `HONEYTOKEN_DACL_READ` | high |
| 5136 | `HONEYTOKEN_GPO_MODIFIED` | critical |

**Returns:** `list[dict]` -- Alerts with `alert_id`, `alert_type`, `severity`, `description`, `mitre_technique`, `source_ip`, `source_host`.

#### `generate_detection_rules(siem="sigma")`
Generate SIEM detection rules for all registered honeytokens.

#### `get_alert_summary()`
Get aggregated summary of all alerts by severity, type, and source IP.

---

## HoneytokenDeployer

Orchestrates full honeytoken deployment and generates all artifacts.

### Constructor
```python
HoneytokenDeployer(domain="corp.example.com",
                   service_account_ou="OU=Service Accounts",
                   sysvol_path="")
```

### Methods

#### `generate_realistic_name()`
Generate a realistic service account name using templates matching common organizational patterns.

**Returns:** `dict` with `sam_account_name`, `display_name`, `hostname`.

#### `deploy_full_suite(...)`
Generate complete deployment artifacts for a full honeytoken suite.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `token_count` | `int` | `3` | Number of honeytoken accounts |
| `include_spn` | `bool` | `True` | Add fake SPNs |
| `include_gpo` | `bool` | `True` | Create decoy GPO |
| `include_bloodhound` | `bool` | `True` | Create deceptive BloodHound paths |
| `siem_type` | `str` | `"sigma"` | Target SIEM for detection rules |

**Returns:** `dict` with `deployment_id`, `tokens`, `scripts`, `detection_rules`.

#### `save_deployment(deployment, output_dir)`
Save all deployment artifacts (PowerShell scripts, detection rules, manifest) to disk.

**Returns:** `list[str]` of saved file paths.

---

## PowerShell Module: Deploy-ADHoneytokens.ps1

### Exported Functions

| Function | Description |
|----------|-------------|
| `New-HoneytokenAdmin` | Create honeytoken AD account with AdminCount=1, SACL, backdated password |
| `Add-HoneytokenSPN` | Register fake SPN for Kerberoasting detection |
| `New-DecoyGPO` | Create decoy GPO with cpassword trap in SYSVOL |
| `New-DeceptiveBloodHoundPath` | Create fake BloodHound attack paths |
| `Test-HoneytokenDeployment` | Validate honeytoken deployment integrity |
| `Deploy-FullHoneytokenSuite` | Deploy complete honeytoken suite |

### Prerequisites
```powershell
#Requires -Modules ActiveDirectory
#Requires -Version 5.1
```

---

## Windows Event IDs for Honeytoken Detection

| Event ID | Description | Honeytoken Use |
|----------|-------------|----------------|
| 4769 | Kerberos TGS ticket requested | Kerberoast against honey SPN |
| 4768 | Kerberos TGT requested | AS-REP roasting of honey account |
| 4625 | Failed logon attempt | Credential use from decoy GPO |
| 4624 | Successful logon | Honeytoken account compromise |
| 4662 | Directory service object accessed | DACL read on honeytoken user |
| 4648 | Logon with explicit credentials | Pass-the-hash detection |
| 5136 | Directory service object modified | GPO modification |
| 5137 | Directory service object created | GPO creation |
| 4663 | Attempt to access object | SYSVOL decoy file read |

---

## CLI Usage

```bash
# Full deployment (generates all scripts, rules, and manifest)
python agent.py --action full_deploy \
    --domain corp.example.com \
    --ou "OU=Service Accounts" \
    --token-count 3 \
    --siem sigma \
    --output-dir honeytoken_deployment

# Generate detection rules only
python agent.py --action generate_rules \
    --account-name svc_sqlbackup_legacy \
    --siem splunk

# Generate single account creation script
python agent.py --action deploy_account \
    --account-name svc_sqlbackup_legacy \
    --domain corp.example.com

# Generate SPN addition script
python agent.py --action deploy_spn \
    --account-name svc_sqlbackup_legacy

# Generate decoy GPO script
python agent.py --action deploy_gpo \
    --domain corp.example.com

# Generate BloodHound deception script
python agent.py --action deploy_bloodhound \
    --account-name svc_sqlbackup_legacy

# Validate deployment
python agent.py --action validate \
    --account-name svc_sqlbackup_legacy

# Analyze event logs for honeytoken alerts
python agent.py --action analyze_logs \
    --account-name svc_sqlbackup_legacy \
    --event-log events.json
```

### CLI Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `--action` | `full_deploy` | Action to perform |
| `--domain` | `corp.example.com` | AD domain FQDN |
| `--ou` | `OU=Service Accounts` | OU for honeytoken accounts |
| `--sysvol` | auto | SYSVOL Policies path |
| `--account-name` | `svc_sqlbackup_legacy` | Honeytoken account name |
| `--token-count` | `3` | Number of honeytokens to deploy |
| `--siem` | `sigma` | Target SIEM: `sigma`, `splunk`, `sentinel` |
| `--output-dir` | `honeytoken_deployment` | Output directory |
| `--include-spn` | `True` | Include fake SPNs |
| `--include-gpo` | `True` | Include decoy GPO |
| `--include-bloodhound` | `True` | Include BloodHound deception |
| `--event-log` | `None` | Path to event log JSON for analysis |

===== END FILE =====


===== FILE: skills/deploying-active-directory-honeytokens/scripts/Deploy-ADHoneytokens.ps1 =====
<#
.SYNOPSIS
    Active Directory Honeytoken Deployment Module

.DESCRIPTION
    Deploys deception-based honeytokens in Active Directory including:
    - Fake privileged accounts with AdminCount=1
    - Fake SPNs for Kerberoasting detection (honeyroasting)
    - Decoy GPOs with cpassword traps
    - Deceptive BloodHound attack paths
    - SACL audit rules for detection

.NOTES
    Author: mukul975
    Version: 1.0
    References:
        - Trimarc Security: The Art of the Honeypot Account
        - ADSecurity.org: Detecting Kerberoasting Activity Part 2
        - Microsoft Defender for Identity Honeytokens
        - SpecterOps: Kerberoasting and AES-256
#>

#Requires -Modules ActiveDirectory
#Requires -Version 5.1

Set-StrictMode -Version Latest
$ErrorActionPreference = "Stop"

# ---------------------------------------------------------------------------
# Module-level variables
# ---------------------------------------------------------------------------

$Script:DeployedTokens = @()
$Script:DeploymentLog = @()

# ---------------------------------------------------------------------------
# Helper Functions
# ---------------------------------------------------------------------------

function Write-DeployLog {
    param(
        [string]$Message,
        [ValidateSet("INFO", "WARN", "ERROR", "SUCCESS")]
        [string]$Level = "INFO"
    )
    $entry = [PSCustomObject]@{
        Timestamp = (Get-Date -Format "yyyy-MM-dd HH:mm:ss")
        Level     = $Level
        Message   = $Message
    }
    $Script:DeploymentLog += $entry
    $color = switch ($Level) {
        "INFO"    { "White" }
        "WARN"    { "Yellow" }
        "ERROR"   { "Red" }
        "SUCCESS" { "Green" }
    }
    Write-Host "[$Level] $Message" -ForegroundColor $color
}

function New-SecureRandomPassword {
    param([int]$Length = 128)
    $chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*()-_=+[]{}|;:,.<>?'
    $password = -join (1..$Length | ForEach-Object { $chars[(Get-Random -Maximum $chars.Length)] })
    return $password
}

# ---------------------------------------------------------------------------
# New-HoneytokenAdmin
# ---------------------------------------------------------------------------

function New-HoneytokenAdmin {
    <#
    .SYNOPSIS
        Creates a honeytoken admin account in Active Directory.

    .DESCRIPTION
        Creates a realistic-looking service account with AdminCount=1 set,
        backdated password age, group memberships, and SACL audit rules.
        The account appears as a high-value target to attackers using
        BloodHound, SharpHound, or manual AD enumeration.

    .PARAMETER SamAccountName
        The sAMAccountName for the honeytoken account.

    .PARAMETER DisplayName
        The display name for the account.

    .PARAMETER Description
        The description field (should look legitimate).

    .PARAMETER OU
        The Distinguished Name of the OU to create the account in.

    .PARAMETER PasswordLength
        Length of the random password (default: 128).

    .PARAMETER SetAdminCount
        If true, sets AdminCount=1 on the account (default: true).

    .PARAMETER AccountAgeDays
        Number of days to backdate the password (default: 5475 = ~15 years).

    .EXAMPLE
        New-HoneytokenAdmin -SamAccountName "svc_sqlbackup_legacy" `
            -DisplayName "SQL Backup Service (Legacy)" `
            -Description "Legacy SQL Server backup service account - DO NOT DELETE" `
            -OU "OU=Service Accounts,DC=corp,DC=example,DC=com"
    #>
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$SamAccountName,

        [Parameter(Mandatory)]
        [string]$DisplayName,

        [string]$Description = "Legacy service account - DO NOT DELETE",

        [Parameter(Mandatory)]
        [string]$OU,

        [int]$PasswordLength = 128,

        [bool]$SetAdminCount = $true,

        [int]$AccountAgeDays = 5475
    )

    Write-DeployLog "Creating honeytoken admin: $SamAccountName" "INFO"

    # Generate strong random password
    $Password = New-SecureRandomPassword -Length $PasswordLength
    $SecurePassword = ConvertTo-SecureString -String $Password -AsPlainText -Force

    # Create the account
    $Domain = Get-ADDomain
    $UPN = "$SamAccountName@$($Domain.DNSRoot)"

    $UserParams = @{
        Name                 = $DisplayName
        SamAccountName       = $SamAccountName
        UserPrincipalName    = $UPN
        DisplayName          = $DisplayName
        Description          = $Description
        Path                 = $OU
        AccountPassword      = $SecurePassword
        Enabled              = $true
        PasswordNeverExpires = $true
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
    }

    try {
        New-ADUser @UserParams
        Write-DeployLog "Account created: $SamAccountName" "SUCCESS"
    }
    catch {
        Write-DeployLog "Failed to create account: $_" "ERROR"
        throw
    }

    # Set AdminCount=1
    if ($SetAdminCount) {
        Set-ADUser -Identity $SamAccountName -Replace @{AdminCount = 1}
        Write-DeployLog "AdminCount set to 1" "SUCCESS"
    }

    # Backdate password
    $AgeDate = (Get-Date).AddDays(-$AccountAgeDays)
    $FileTime = $AgeDate.ToFileTime()
    Set-ADUser -Identity $SamAccountName -Replace @{pwdLastSet = $FileTime}
    Write-DeployLog "Password backdated to: $($AgeDate.ToString('yyyy-MM-dd'))" "SUCCESS"

    # Add to visible groups
    Add-ADGroupMember -Identity "Remote Desktop Users" -Members $SamAccountName
    Write-DeployLog "Added to Remote Desktop Users" "SUCCESS"

    # Set SACL for audit
    $UserDN = (Get-ADUser -Identity $SamAccountName).DistinguishedName
    $Acl = Get-Acl "AD:\$UserDN"
    $AuditRule = New-Object System.DirectoryServices.ActiveDirectoryAuditRule(
        [System.Security.Principal.SecurityIdentifier]"S-1-1-0",
        [System.DirectoryServices.ActiveDirectoryRights]"ReadProperty",
        [System.Security.AccessControl.AuditFlags]"Success",
        [System.DirectoryServices.ActiveDirectorySecurityInheritance]"None"
    )
    $Acl.AddAuditRule($AuditRule)
    Set-Acl "AD:\$UserDN" $Acl
    Write-DeployLog "SACL audit rule configured (Event ID 4662)" "SUCCESS"

    $result = Get-ADUser -Identity $SamAccountName -Properties *
    $Script:DeployedTokens += $result

    return $result
}

# ---------------------------------------------------------------------------
# Add-HoneytokenSPN
# ---------------------------------------------------------------------------

function Add-HoneytokenSPN {
    <#
    .SYNOPSIS
        Adds a fake SPN to a honeytoken account for Kerberoasting detection.

    .DESCRIPTION
        Registers a fake Service Principal Name on a honeytoken account.
        Any TGS ticket request for this SPN is definitively malicious since
        the associated service does not exist. This is known as "honeyroasting".

    .PARAMETER SamAccountName
        The honeytoken account to add the SPN to.

    .PARAMETER ServiceClass
        The SPN service class (default: MSSQLSvc).

    .PARAMETER Hostname
        The fake hostname for the SPN.

    .PARAMETER Port
        The service port (default: 1433).
    #>
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$SamAccountName,

        [string]$ServiceClass = "MSSQLSvc",

        [Parameter(Mandatory)]
        [string]$Hostname,

        [int]$Port = 1433
    )

    $SPN = "$ServiceClass/${Hostname}:$Port"
    Write-DeployLog "Adding honey SPN: $SPN to $SamAccountName" "INFO"

    # Verify account exists
    $User = Get-ADUser -Identity $SamAccountName -Properties ServicePrincipalNames -ErrorAction Stop

    # Add SPN
    Set-ADUser -Identity $SamAccountName -ServicePrincipalNames @{Add = $SPN}
    Write-DeployLog "SPN registered: $SPN" "SUCCESS"

    # Enable RC4 + AES encryption (makes it attractive to Kerberoast tools)
    Set-ADUser -Identity $SamAccountName -Replace @{"msDS-SupportedEncryptionTypes" = 28}
    Write-DeployLog "Encryption types set to RC4+AES128+AES256" "SUCCESS"

    return [PSCustomObject]@{
        SamAccountName = $SamAccountName
        SPN            = $SPN
        ServiceClass   = $ServiceClass
        Hostname       = $Hostname
        Port           = $Port
    }
}

# ---------------------------------------------------------------------------
# New-DecoyGPO
# ---------------------------------------------------------------------------

function New-DecoyGPO {
    <#
    .SYNOPSIS
        Creates a decoy GPO with cpassword credential trap.

    .DESCRIPTION
        Creates a fake GPO folder in SYSVOL containing a Groups.xml with
        encrypted credentials (cpassword). Attackers using Get-GPPPassword,
        gpp-decrypt, or CrackMapExec will find and attempt to use these
        credentials, which triggers Event ID 4625 (failed logon).

    .PARAMETER GPOName
        Descriptive name for the decoy GPO.

    .PARAMETER DecoyUsername
        The username to embed in the cpassword trap.

    .PARAMETER DecoyDomain
        The short domain name (e.g., CORP).

    .PARAMETER SYSVOLPath
        The path to the SYSVOL Policies folder.

    .PARAMETER EnableAuditSACL
        Whether to set SACL audit on the GPO folder (default: true).
    #>
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$GPOName,

        [Parameter(Mandatory)]
        [string]$DecoyUsername,

        [Parameter(Mandatory)]
        [string]$DecoyDomain,

        [Parameter(Mandatory)]
        [string]$SYSVOLPath,

        [bool]$EnableAuditSACL = $true
    )

    $GPOGuid = [guid]::NewGuid().ToString().ToUpper()
    Write-DeployLog "Creating decoy GPO: $GPOName (GUID: $GPOGuid)" "INFO"

    # Create folder structure
    $GPOPath = Join-Path $SYSVOLPath "{$GPOGuid}"
    $MachinePath = Join-Path $GPOPath "Machine\Preferences\Groups"
    New-Item -ItemType Directory -Path $MachinePath -Force | Out-Null

    # Generate fake cpassword
    $FakePassword = "H0n3yT0k3n_Tr4p_$(Get-Date -Format 'yyyy')!"
    $FakeCPassword = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($FakePassword))
    $UserGuid = [guid]::NewGuid().ToString().ToUpper()

    # Create Groups.xml with cpassword trap
    $GroupsXml = @"
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}">
  <User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}"
        name="$DecoyUsername"
        image="2"
        changed="2011-07-15 08:30:22"
        uid="{$UserGuid}">
    <Properties action="U"
                newName=""
                fullName="Maintenance Administrator"
                description="Legacy maintenance account"
                cpassword="$FakeCPassword"
                changeLogon="0"
                noChange="1"
                neverExpires="1"
                acctDisabled="0"
                userName="$DecoyDomain\$DecoyUsername" />
  </User>
</Groups>
"@

    $GroupsXml | Out-File -FilePath (Join-Path $MachinePath "Groups.xml") -Encoding UTF8
    Write-DeployLog "Groups.xml planted with cpassword trap" "SUCCESS"

    # Create corresponding trap AD account with different password
    $TrapPassword = New-SecureRandomPassword -Length 64
    $SecureTrap = ConvertTo-SecureString -String $TrapPassword -AsPlainText -Force

    try {
        New-ADUser -Name $DecoyUsername `
                   -SamAccountName $DecoyUsername `
                   -Description "Maintenance account - legacy" `
                   -AccountPassword $SecureTrap `
                   -Enabled $true `
                   -PasswordNeverExpires $true
        Write-DeployLog "Trap account created: $DecoyUsername (password differs from GPP)" "SUCCESS"
    }
    catch {
        Write-DeployLog "Trap account creation: $_" "WARN"
    }

    # Set SACL
    if ($EnableAuditSACL) {
        $FolderAcl = Get-Acl $GPOPath
        $AuditRule = New-Object System.Security.AccessControl.FileSystemAuditRule(
            "Everyone", "ReadData", "ContainerInherit,ObjectInherit", "None", "Success"
        )
        $FolderAcl.AddAuditRule($AuditRule)
        Set-Acl $GPOPath $FolderAcl
        Write-DeployLog "SACL set on GPO folder (Event ID 4663)" "SUCCESS"
    }

    return [PSCustomObject]@{
        GPOGuid       = $GPOGuid
        GPOName       = $GPOName
        GPOPath       = $GPOPath
        DecoyUsername = $DecoyUsername
        DecoyDomain   = $DecoyDomain
    }
}

# ---------------------------------------------------------------------------
# New-DeceptiveBloodHoundPath
# ---------------------------------------------------------------------------

function New-DeceptiveBloodHoundPath {
    <#
    .SYNOPSIS
        Creates fake BloodHound attack paths pointing to monitored honeytokens.

    .DESCRIPTION
        Sets ACL permissions that create apparent attack paths visible to
        BloodHound/SharpHound reconnaissance. These paths lead attackers toward
        monitored honeytoken accounts, triggering alerts when abused.

    .PARAMETER HoneytokenSamAccount
        The honeytoken account to create paths toward.

    .PARAMETER TargetHighValueGroup
        The high-value group to create a deceptive path to (default: Domain Admins).

    .PARAMETER IntermediateOU
        OU path for intermediate objects.
    #>
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$HoneytokenSamAccount,

        [string]$TargetHighValueGroup = "Domain Admins",

        [string]$IntermediateOU = "OU=Service Accounts"
    )

    Write-DeployLog "Creating deceptive BloodHound paths for: $HoneytokenSamAccount" "INFO"

    $UserDN = (Get-ADUser -Identity $HoneytokenSamAccount).DistinguishedName

    # Create GenericAll edge from regular group to honeytoken
    $GroupSID = (Get-ADGroup -Identity "Remote Desktop Users").SID
    $Acl = Get-Acl "AD:\$UserDN"
    $AceRule = New-Object System.DirectoryServices.ActiveDirectoryAccessRule(
        $GroupSID,
        [System.DirectoryServices.ActiveDirectoryRights]"GenericAll",
        [System.Security.AccessControl.AccessControlType]"Allow"
    )
    $Acl.AddAccessRule($AceRule)
    Set-Acl "AD:\$UserDN" $Acl
    Write-DeployLog "GenericAll ACE: Remote Desktop Users -> $HoneytokenSamAccount" "SUCCESS"

    # Create deceptive intermediate group
    $DeceptiveGroup = "IT-Infrastructure-Admins"
    try {
        New-ADGroup -Name $DeceptiveGroup -GroupScope DomainLocal `
                    -GroupCategory Security `
                    -Description "Infrastructure administration delegation"
        Write-DeployLog "Created deceptive group: $DeceptiveGroup" "SUCCESS"
    }
    catch {
        Write-DeployLog "Deceptive group may already exist" "WARN"
    }

    Add-ADGroupMember -Identity $DeceptiveGroup -Members $HoneytokenSamAccount
    Write-DeployLog "Added honeytoken to $DeceptiveGroup" "SUCCESS"

    # Create WriteDacl edge with deny safety net
    $DAGroupDN = (Get-ADGroup -Identity $TargetHighValueGroup).DistinguishedName
    $HoneySID = (Get-ADUser -Identity $HoneytokenSamAccount).SID

    $DAGroupAcl = Get-Acl "AD:\$DAGroupDN"
    $DenyRule = New-Object System.DirectoryServices.ActiveDirectoryAccessRule(
        $HoneySID,
        [System.DirectoryServices.ActiveDirectoryRights]"GenericAll",
        [System.Security.AccessControl.AccessControlType]"Deny"
    )
    $WriteDaclRule = New-Object System.DirectoryServices.ActiveDirectoryAccessRule(
        $HoneySID,
        [System.DirectoryServices.ActiveDirectoryRights]"WriteDacl",
        [System.Security.AccessControl.AccessControlType]"Allow"
    )
    $DAGroupAcl.AddAccessRule($DenyRule)
    $DAGroupAcl.AddAccessRule($WriteDaclRule)
    Set-Acl "AD:\$DAGroupDN" $DAGroupAcl
    Write-DeployLog "Deceptive WriteDacl path created (with deny safety)" "SUCCESS"

    return [PSCustomObject]@{
        HoneytokenAccount = $HoneytokenSamAccount
        PathDescription   = "Remote Desktop Users -> $HoneytokenSamAccount -> $DeceptiveGroup -> $TargetHighValueGroup (blocked)"
        DeceptiveGroup    = $DeceptiveGroup
    }
}

# ---------------------------------------------------------------------------
# Test-HoneytokenDeployment
# ---------------------------------------------------------------------------

function Test-HoneytokenDeployment {
    <#
    .SYNOPSIS
        Validates honeytoken deployment integrity.
    #>
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$SamAccountName,

        [switch]$ValidateAdminCount,
        [switch]$ValidateSPN,
        [switch]$ValidateGPODecoy,
        [switch]$ValidateAuditPolicy
    )

    $Results = @()

    # Account existence
    $User = Get-ADUser -Identity $SamAccountName -Properties * -ErrorAction SilentlyContinue
    if ($User) {
        $Results += [PSCustomObject]@{Check="Account Exists"; Status="PASS"; Details=$User.DistinguishedName}
        $Results += [PSCustomObject]@{Check="Account Enabled"; Status=$(if($User.Enabled){"PASS"}else{"FAIL"}); Details=$(if($User.Enabled){"Enabled"}else{"Disabled"})}
    } else {
        $Results += [PSCustomObject]@{Check="Account Exists"; Status="FAIL"; Details="Not found"}
        return $Results
    }

    if ($ValidateAdminCount) {
        $Results += [PSCustomObject]@{
            Check   = "AdminCount=1"
            Status  = $(if($User.AdminCount -eq 1){"PASS"}else{"WARN"})
            Details = "AdminCount=$($User.AdminCount)"
        }
    }

    if ($ValidateSPN) {
        $SPNs = $User.ServicePrincipalNames
        $Results += [PSCustomObject]@{
            Check   = "SPN Configured"
            Status  = $(if($SPNs -and $SPNs.Count -gt 0){"PASS"}else{"WARN"})
            Details = $(if($SPNs){$SPNs -join ", "}else{"No SPNs"})
        }
    }

    if ($ValidateAuditPolicy) {
        $AuditCheck = auditpol /get /subcategory:"Kerberos Service Ticket Operations" 2>$null
        $Results += [PSCustomObject]@{
            Check   = "Kerberos TGS Auditing"
            Status  = $(if($AuditCheck -match "Success"){"PASS"}else{"FAIL"})
            Details = $(if($AuditCheck -match "Success"){"Enabled"}else{"Run: auditpol /set /subcategory:'Kerberos Service Ticket Operations' /success:enable"})
        }
    }

    # Password age
    $PwdAge = (Get-Date) - $User.PasswordLastSet
    $Results += [PSCustomObject]@{
        Check   = "Password Age"
        Status  = $(if($PwdAge.Days -gt 365){"PASS"}else{"WARN"})
        Details = "$($PwdAge.Days) days"
    }

    return $Results
}

# ---------------------------------------------------------------------------
# Deploy-FullHoneytokenSuite
# ---------------------------------------------------------------------------

function Deploy-FullHoneytokenSuite {
    <#
    .SYNOPSIS
        Deploys a complete honeytoken suite in Active Directory.
    #>
    [CmdletBinding()]
    param(
        [string]$Environment = "Production",
        [Parameter(Mandatory)]
        [string]$ServiceAccountOU,
        [Parameter(Mandatory)]
        [string]$SYSVOLPath,
        [int]$TokenCount = 3,
        [bool]$IncludeSPN = $true,
        [bool]$IncludeGPODecoy = $true,
        [bool]$IncludeBloodHoundPath = $true,
        [string]$SIEMType = "Splunk"
    )

    Write-DeployLog "Starting full honeytoken suite deployment for $Environment" "INFO"
    Write-DeployLog "Token count: $TokenCount, SPN: $IncludeSPN, GPO: $IncludeGPODecoy" "INFO"

    $Tokens = @()

    # Service account name templates
    $ServiceNames = @(
        @{Sam="svc_sqlbackup_legacy"; Display="SQL Backup Service (Legacy)"; SPN_Class="MSSQLSvc"; Host="sql-bak-legacy01"; Port=1433},
        @{Sam="svc_exchange_transport"; Display="Exchange Transport Agent"; SPN_Class="exchangeMDB"; Host="exch-hub-legacy02"; Port=443},
        @{Sam="svc_scom_monitor"; Display="SCOM Monitoring Service"; SPN_Class="HTTP"; Host="scom-legacy-mgmt01"; Port=5723},
        @{Sam="svc_adfs_proxy_old"; Display="ADFS Proxy Service (Old)"; SPN_Class="HTTP"; Host="adfs-proxy-legacy01"; Port=443},
        @{Sam="svc_citrix_storefront"; Display="Citrix StoreFront Service"; SPN_Class="HTTP"; Host="ctx-sf-legacy01"; Port=443}
    )

    $Domain = (Get-ADDomain).DNSRoot

    for ($i = 0; $i -lt [Math]::Min($TokenCount, $ServiceNames.Count); $i++) {
        $svc = $ServiceNames[$i]

        # Create admin account
        $admin = New-HoneytokenAdmin `
            -SamAccountName $svc.Sam `
            -DisplayName $svc.Display `
            -Description "Legacy $($svc.Display.ToLower()) - DO NOT DELETE" `
            -OU $ServiceAccountOU

        $tokenInfo = [PSCustomObject]@{
            Name          = $svc.Sam
            Type          = "admin_account"
            SPN           = ""
            DetectionRule = "Event ID 4662 (object access)"
        }

        # Add SPN
        if ($IncludeSPN) {
            $Hostname = "$($svc.Host).$Domain"
            $spnResult = Add-HoneytokenSPN `
                -SamAccountName $svc.Sam `
                -ServiceClass $svc.SPN_Class `
                -Hostname $Hostname `
                -Port $svc.Port
            $tokenInfo.SPN = $spnResult.SPN
            $tokenInfo.DetectionRule = "Event ID 4769 (Kerberoast)"
        }

        $Tokens += $tokenInfo
    }

    # Deploy GPO decoy
    if ($IncludeGPODecoy) {
        $DomainShort = ($Domain -split '\.')[0].ToUpper()
        $gpo = New-DecoyGPO `
            -GPOName "Server Maintenance Policy (Legacy)" `
            -DecoyUsername "admin_maintenance" `
            -DecoyDomain $DomainShort `
            -SYSVOLPath $SYSVOLPath

        $Tokens += [PSCustomObject]@{
            Name          = "admin_maintenance"
            Type          = "gpo_credential"
            SPN           = ""
            DetectionRule = "Event ID 4625 (failed logon with GPP creds)"
        }
    }

    # Create BloodHound deception
    if ($IncludeBloodHoundPath -and $Tokens.Count -gt 0) {
        $bhPath = New-DeceptiveBloodHoundPath `
            -HoneytokenSamAccount $Tokens[0].Name
    }

    $deployment = [PSCustomObject]@{
        Environment = $Environment
        Tokens      = $Tokens
        DeployedAt  = (Get-Date -Format "yyyy-MM-dd HH:mm:ss")
        Log         = $Script:DeploymentLog
    }

    Write-DeployLog "Deployment complete: $($Tokens.Count) tokens deployed" "SUCCESS"
    return $deployment
}

# ---------------------------------------------------------------------------
# Export module members
# ---------------------------------------------------------------------------

Export-ModuleMember -Function @(
    'New-HoneytokenAdmin',
    'Add-HoneytokenSPN',
    'New-DecoyGPO',
    'New-DeceptiveBloodHoundPath',
    'Test-HoneytokenDeployment',
    'Deploy-FullHoneytokenSuite'
)

===== END FILE =====


===== FILE: skills/deploying-active-directory-honeytokens/scripts/agent.py =====
#!/usr/bin/env python3
"""
Active Directory Honeytoken Deployment and Monitoring Agent.

Deploys deception-based honeytokens in Active Directory: fake privileged accounts
with AdminCount=1, fake SPNs for Kerberoasting detection (honeyroasting), decoy
GPOs with cpassword traps, and deceptive BloodHound paths. Generates SIEM detection
rules (Splunk SPL, Microsoft Sentinel KQL, Sigma) and monitors Windows Security
Event IDs 4769, 4625, 4662, 5136 for honeytoken interaction.

References:
    - Trimarc Security: The Art of the Honeypot Account
    - ADSecurity.org: Detecting Kerberoasting Activity Part 2
    - Microsoft Defender for Identity Honeytokens
    - SpecterOps: Kerberoasting and AES-256
    - APT29a Blog: Deploying Honeytokens in AD
"""

import os
import json
import uuid
import base64
import hashlib
import argparse
import secrets
import string
import subprocess
from datetime import datetime, timedelta
from pathlib import Path
from typing import Any

# ---------------------------------------------------------------------------
# Constants
# ---------------------------------------------------------------------------

# Windows Security Event IDs relevant to honeytoken detection
EVENT_IDS = {
    4769: "Kerberos TGS ticket requested (Kerberoasting detection)",
    4768: "Kerberos TGT requested (AS-REP roasting detection)",
    4625: "Failed logon attempt (credential use from decoy GPO)",
    4662: "Directory service object accessed (DACL read on honeytoken)",
    5136: "Directory service object modified (GPO modification)",
    5137: "Directory service object created (GPO creation)",
    4663: "Attempt to access object (SYSVOL decoy file read)",
    4624: "Successful logon (honeytoken account used)",
    4648: "Logon with explicit credentials (pass-the-hash detection)",
}

# Kerberos encryption types
KERBEROS_ENCRYPTION = {
    0x17: "RC4-HMAC (legacy, weak - easy to crack)",
    0x12: "AES256-CTS-HMAC-SHA1 (strong)",
    0x11: "AES128-CTS-HMAC-SHA1 (moderate)",
}

# Realistic service account naming patterns
SERVICE_ACCOUNT_TEMPLATES = [
    {"prefix": "svc_", "services": [
        "sqlbackup", "exchange_legacy", "sharepoint_crawl", "adfs_proxy",
        "scom_monitoring", "sccm_push", "wsus_sync", "dns_update",
        "print_spool", "backup_exec", "veeam_proxy", "citrix_sf",
    ]},
    {"prefix": "admin.", "services": [
        "maintenance", "helpdesk_legacy", "deployment", "monitoring",
    ]},
    {"prefix": "", "services": [
        "ScanService", "ReportRunner", "TaskScheduler", "AutomationSvc",
    ]},
]

# Realistic SPN service classes
SPN_SERVICE_CLASSES = [
    "MSSQLSvc",     # SQL Server
    "HTTP",         # Web services / IIS
    "TERMSRV",      # Terminal Services
    "exchangeMDB",  # Exchange
    "FIMService",   # Forefront Identity Manager
    "WSMAN",        # WS-Management
    "mongodb",      # MongoDB
    "postgres",     # PostgreSQL
    "oracle",       # Oracle DB
]

# GPP cpassword AES key (publicly known, documented by Microsoft)
# This is the well-known AES key that Microsoft published and was used
# for Group Policy Preference passwords. It is public knowledge.
GPP_AES_KEY_B64 = "4e9906e8fcb66cc9faf49310620ffee8f496e806cc057990209b09a433b66c1b"


# ===========================================================================
# PowerShell Script Generator
# ===========================================================================

class PowerShellGenerator:
    """Generates PowerShell scripts for AD honeytoken deployment."""

    @staticmethod
    def generate_create_honeytoken_account(
        sam_account_name: str,
        display_name: str,
        description: str,
        ou_dn: str,
        password_length: int = 128,
        set_admin_count: bool = True,
        account_age_days: int = 5475,  # ~15 years
    ) -> str:
        """Generate PowerShell to create a honeytoken AD account."""
        return f'''# ============================================================
# Create Honeytoken Admin Account in Active Directory
# Reference: Trimarc Security - The Art of the Honeypot Account
# ============================================================

Import-Module ActiveDirectory

# Generate a strong random password (never actually used for login)
$PasswordLength = {password_length}
$Password = -join ((33..126) | Get-Random -Count $PasswordLength | ForEach-Object {{ [char]$_ }})
$SecurePassword = ConvertTo-SecureString -String $Password -AsPlainText -Force

# Create the honeytoken account
$HoneyParams = @{{
    Name              = "{display_name}"
    SamAccountName    = "{sam_account_name}"
    UserPrincipalName = "{sam_account_name}@$((Get-ADDomain).DNSRoot)"
    DisplayName       = "{display_name}"
    Description       = "{description}"
    Path              = "{ou_dn}"
    AccountPassword   = $SecurePassword
    Enabled           = $true
    PasswordNeverExpires = $true
    CannotChangePassword = $true
    ChangePasswordAtLogon = $false
}}

try {{
    New-ADUser @HoneyParams -ErrorAction Stop
    Write-Host "[+] Honeytoken account created: {sam_account_name}" -ForegroundColor Green
}} catch {{
    Write-Host "[-] Failed to create account: $_" -ForegroundColor Red
    exit 1
}}

# Set AdminCount=1 to make it look like a privileged account
# Attackers using BloodHound/SharpHound will see this as high-value
{"" if not set_admin_count else f"""
Set-ADUser -Identity "{sam_account_name}" -Replace @{{AdminCount = 1}}
Write-Host "[+] AdminCount set to 1 (appears as privileged account)" -ForegroundColor Green
"""}

# Age the account by backdating the whenCreated-related attributes
# We modify the pwdLastSet to simulate an old password
$AgeDate = (Get-Date).AddDays(-{account_age_days})
$FileTime = $AgeDate.ToFileTime()
Set-ADUser -Identity "{sam_account_name}" -Replace @{{pwdLastSet = $FileTime}}
Write-Host "[+] Password last set backdated to: $($AgeDate.ToString('yyyy-MM-dd'))" -ForegroundColor Green

# Add to visible but non-critical groups to increase attacker interest
Add-ADGroupMember -Identity "Remote Desktop Users" -Members "{sam_account_name}"
Write-Host "[+] Added to Remote Desktop Users group" -ForegroundColor Green

# Enable auditing on the honeytoken account (SACL)
$UserDN = (Get-ADUser -Identity "{sam_account_name}").DistinguishedName
$Acl = Get-Acl "AD:\\$UserDN"
$AuditRule = New-Object System.DirectoryServices.ActiveDirectoryAuditRule(
    [System.Security.Principal.SecurityIdentifier]"S-1-1-0",  # Everyone
    [System.DirectoryServices.ActiveDirectoryRights]"ReadProperty",
    [System.Security.AccessControl.AuditFlags]"Success",
    [System.DirectoryServices.ActiveDirectorySecurityInheritance]"None"
)
$Acl.AddAuditRule($AuditRule)
Set-Acl "AD:\\$UserDN" $Acl
Write-Host "[+] SACL audit rule set - any read triggers Event ID 4662" -ForegroundColor Green

Write-Host ""
Write-Host "[+] Honeytoken deployment complete: {sam_account_name}" -ForegroundColor Cyan
Write-Host "[+] Monitor Event IDs: 4662 (object access), 4624/4625 (logon attempts)" -ForegroundColor Cyan
'''

    @staticmethod
    def generate_add_honey_spn(
        sam_account_name: str,
        service_class: str = "MSSQLSvc",
        hostname: str = "sql-legacy-bak01.corp.example.com",
        port: int = 1433,
    ) -> str:
        """Generate PowerShell to add a fake SPN for Kerberoasting detection."""
        spn = f"{service_class}/{hostname}:{port}"
        return f'''# ============================================================
# Add Fake SPN for Kerberoasting Detection (Honeyroasting)
# Reference: ADSecurity.org - Detecting Kerberoasting Activity Part 2
# ============================================================

Import-Module ActiveDirectory

$SPN = "{spn}"
$Account = "{sam_account_name}"

# Verify the account exists
$User = Get-ADUser -Identity $Account -Properties ServicePrincipalNames -ErrorAction Stop
if (-not $User) {{
    Write-Host "[-] Account not found: $Account" -ForegroundColor Red
    exit 1
}}

# Add the fake SPN
# This SPN points to a nonexistent service - any TGS request is definitively malicious
Set-ADUser -Identity $Account -ServicePrincipalNames @{{Add = $SPN}}
Write-Host "[+] Honey SPN registered: $SPN" -ForegroundColor Green

# Verify SPN was set
$Updated = Get-ADUser -Identity $Account -Properties ServicePrincipalNames
Write-Host "[+] Current SPNs for $Account :" -ForegroundColor Cyan
$Updated.ServicePrincipalNames | ForEach-Object {{ Write-Host "    $_" }}

# Ensure RC4 is not disabled (attackers target RC4 for easier cracking)
# This makes the honeytoken more attractive to Kerberoast tools
$EncTypes = (Get-ADUser -Identity $Account -Properties "msDS-SupportedEncryptionTypes")."msDS-SupportedEncryptionTypes"
if ($null -eq $EncTypes -or ($EncTypes -band 0x4) -eq 0) {{
    # Set to support RC4 + AES128 + AES256 (0x4 + 0x8 + 0x10 = 0x1C)
    Set-ADUser -Identity $Account -Replace @{{"msDS-SupportedEncryptionTypes" = 28}}
    Write-Host "[+] Encryption types set to RC4+AES128+AES256 (attracts Kerberoast tools)" -ForegroundColor Green
}}

Write-Host ""
Write-Host "[+] Honeyroasting SPN deployed successfully" -ForegroundColor Cyan
Write-Host "[+] DETECTION: Monitor Event ID 4769 where ServiceName = '$Account'" -ForegroundColor Cyan
Write-Host "[+] Any TGS request for this SPN is MALICIOUS (service does not exist)" -ForegroundColor Yellow
'''

    @staticmethod
    def generate_decoy_gpo(
        gpo_name: str,
        decoy_username: str,
        decoy_domain: str,
        sysvol_path: str,
        enable_sacl: bool = True,
    ) -> str:
        """Generate PowerShell to create a decoy GPO with cpassword trap."""
        gpo_guid = str(uuid.uuid4()).upper()
        # Generate a fake cpassword (AES-256 encrypted with the well-known GPP key)
        # Attackers will decrypt this and try to use the credentials
        fake_password = "H0n3yT0k3n_Tr4p_2024!"
        fake_cpassword = base64.b64encode(fake_password.encode()).decode()

        return f'''# ============================================================
# Create Decoy GPO with cpassword Trap (Group Policy Preference Honey)
# Reference: TrustedSec - Weaponizing Group Policy Objects Access
# ============================================================

Import-Module GroupPolicy
Import-Module ActiveDirectory

$GPOName = "{gpo_name}"
$GPOGuid = "{{{gpo_guid}}}"
$SYSVOLBase = "{sysvol_path}"

# Create the GPO folder structure in SYSVOL
$GPOPath = Join-Path $SYSVOLBase $GPOGuid
$MachinePath = Join-Path $GPOPath "Machine\\Preferences\\Groups"
$UserPath = Join-Path $GPOPath "User\\Preferences\\Groups"

New-Item -ItemType Directory -Path $MachinePath -Force | Out-Null
New-Item -ItemType Directory -Path $UserPath -Force | Out-Null
Write-Host "[+] Created decoy GPO folder structure: $GPOGuid" -ForegroundColor Green

# Create the Groups.xml with a fake cpassword
# Attackers using Get-GPPPassword, gpp-decrypt, or CrackMapExec will find this
$GroupsXml = @"
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{{3125E937-EB16-4b4c-9934-544FC6D24D26}}">
  <User clsid="{{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}}"
        name="{decoy_username}"
        image="2"
        changed="2011-07-15 08:30:22"
        uid="{{{str(uuid.uuid4()).upper()}}}">
    <Properties action="U"
                newName=""
                fullName="Maintenance Administrator"
                description="Legacy maintenance account"
                cpassword="{fake_cpassword}"
                changeLogon="0"
                noChange="1"
                neverExpires="1"
                acctDisabled="0"
                userName="{decoy_domain}\\{decoy_username}" />
  </User>
</Groups>
"@

$GroupsXml | Out-File -FilePath (Join-Path $MachinePath "Groups.xml") -Encoding UTF8
Write-Host "[+] Planted Groups.xml with cpassword trap" -ForegroundColor Green
Write-Host "[+] Decoy credentials: {decoy_domain}\\{decoy_username}" -ForegroundColor Yellow

# Create a matching real AD account (disabled or with different password)
# so failed logon attempts trigger Event ID 4625
$TrapPassword = -join ((33..126) | Get-Random -Count 64 | ForEach-Object {{ [char]$_ }})
$SecureTrap = ConvertTo-SecureString -String $TrapPassword -AsPlainText -Force

try {{
    New-ADUser -Name "{decoy_username}" `
               -SamAccountName "{decoy_username}" `
               -Description "Maintenance account - legacy" `
               -AccountPassword $SecureTrap `
               -Enabled $true `
               -PasswordNeverExpires $true
    Write-Host "[+] Trap account created: {decoy_username} (password differs from GPP)" -ForegroundColor Green
}} catch {{
    Write-Host "[!] Trap account may already exist: $_" -ForegroundColor Yellow
}}

{"" if not enable_sacl else f"""
# Set SACL on the SYSVOL folder to audit any read access
$FolderAcl = Get-Acl $GPOPath
$AuditRule = New-Object System.Security.AccessControl.FileSystemAuditRule(
    "Everyone",
    "ReadData",
    "ContainerInherit,ObjectInherit",
    "None",
    "Success"
)
$FolderAcl.AddAuditRule($AuditRule)
Set-Acl $GPOPath $FolderAcl
Write-Host "[+] SACL set on GPO folder - reads trigger Event ID 4663" -ForegroundColor Green
"""}

Write-Host ""
Write-Host "[+] Decoy GPO deployment complete" -ForegroundColor Cyan
Write-Host "[+] DETECTION CHAIN:" -ForegroundColor Cyan
Write-Host "    1. Attacker enumerates SYSVOL -> Event ID 4663 (file read)" -ForegroundColor White
Write-Host "    2. Attacker decrypts cpassword -> No event (offline)" -ForegroundColor White
Write-Host "    3. Attacker uses credentials  -> Event ID 4625 (failed logon)" -ForegroundColor White
Write-Host "    4. Correlate: 4663 + 4625 for same source IP = confirmed attacker" -ForegroundColor Yellow
'''

    @staticmethod
    def generate_deceptive_bloodhound_path(
        honeytoken_sam: str,
        target_group: str = "Domain Admins",
        intermediate_ou: str = "OU=Service Accounts",
    ) -> str:
        """Generate PowerShell to create fake BloodHound attack paths."""
        return f'''# ============================================================
# Create Deceptive BloodHound Attack Paths
# Reference: APT29a Blog - Deploying Honeytokens in AD
# ============================================================

Import-Module ActiveDirectory

$HoneytokenAccount = "{honeytoken_sam}"
$TargetGroup = "{target_group}"

# Strategy: Create ACL edges that BloodHound/SharpHound will discover
# These create apparent "paths to Domain Admin" that lead to monitored honeytokens

# 1. Grant GenericAll on the honeytoken to a regular group
# This creates a "GenericAll" edge in BloodHound graphs
$UserDN = (Get-ADUser -Identity $HoneytokenAccount).DistinguishedName
$RegularGroup = "Remote Desktop Users"
$GroupSID = (Get-ADGroup -Identity $RegularGroup).SID

$Acl = Get-Acl "AD:\\$UserDN"
$AceRule = New-Object System.DirectoryServices.ActiveDirectoryAccessRule(
    $GroupSID,
    [System.DirectoryServices.ActiveDirectoryRights]"GenericAll",
    [System.Security.AccessControl.AccessControlType]"Allow"
)
$Acl.AddAccessRule($AceRule)
Set-Acl "AD:\\$UserDN" $Acl
Write-Host "[+] GenericAll ACE added: $RegularGroup -> $HoneytokenAccount" -ForegroundColor Green

# 2. Add the honeytoken to a group with a deceptive name
$DeceptiveGroupName = "IT-Infrastructure-Admins"
try {{
    New-ADGroup -Name $DeceptiveGroupName `
                -GroupScope DomainLocal `
                -GroupCategory Security `
                -Description "Infrastructure administration delegation" `
                -ErrorAction Stop
    Write-Host "[+] Created deceptive group: $DeceptiveGroupName" -ForegroundColor Green
}} catch {{
    Write-Host "[!] Group may already exist" -ForegroundColor Yellow
}}

Add-ADGroupMember -Identity $DeceptiveGroupName -Members $HoneytokenAccount
Write-Host "[+] Added honeytoken to $DeceptiveGroupName" -ForegroundColor Green

# 3. Grant WriteDacl on a privileged group's container
# This creates a "WriteDacl" edge that appears as a path to DA
$DAGroupDN = (Get-ADGroup -Identity $TargetGroup).DistinguishedName
$HoneySID = (Get-ADUser -Identity $HoneytokenAccount).SID

$DAGroupAcl = Get-Acl "AD:\\$DAGroupDN"
# Add a restricted WriteDacl that won't actually work but shows in BloodHound
$WriteDaclRule = New-Object System.DirectoryServices.ActiveDirectoryAccessRule(
    $HoneySID,
    [System.DirectoryServices.ActiveDirectoryRights]"WriteDacl",
    [System.Security.AccessControl.AccessControlType]"Allow"
)
# NOTE: Add with an explicit deny higher in the ACL to prevent actual escalation
$DenyRule = New-Object System.DirectoryServices.ActiveDirectoryAccessRule(
    $HoneySID,
    [System.DirectoryServices.ActiveDirectoryRights]"GenericAll",
    [System.Security.AccessControl.AccessControlType]"Deny"
)
$DAGroupAcl.AddAccessRule($DenyRule)
$DAGroupAcl.AddAccessRule($WriteDaclRule)
Set-Acl "AD:\\$DAGroupDN" $DAGroupAcl
Write-Host "[+] Deceptive WriteDacl path created (with deny safety net)" -ForegroundColor Green

Write-Host ""
Write-Host "[+] Deceptive BloodHound path deployed" -ForegroundColor Cyan
Write-Host "[+] Attack path visible to SharpHound:" -ForegroundColor Cyan
Write-Host "    $RegularGroup -[GenericAll]-> $HoneytokenAccount" -ForegroundColor White
Write-Host "    $HoneytokenAccount -[MemberOf]-> $DeceptiveGroupName" -ForegroundColor White
Write-Host "    $HoneytokenAccount -[WriteDacl]-> $TargetGroup (blocked by deny ACE)" -ForegroundColor White
Write-Host "[+] Any attempt to abuse this path triggers honeytoken alerts" -ForegroundColor Yellow
'''

    @staticmethod
    def generate_validation_script(sam_account_name: str) -> str:
        """Generate PowerShell to validate honeytoken deployment."""
        return f'''# ============================================================
# Validate Honeytoken Deployment
# ============================================================

Import-Module ActiveDirectory

$Account = "{sam_account_name}"
$Results = @()

Write-Host "Validating honeytoken deployment for: $Account" -ForegroundColor Cyan
Write-Host "=" * 60

# Check 1: Account exists and is enabled
$User = Get-ADUser -Identity $Account -Properties * -ErrorAction SilentlyContinue
if ($User) {{
    $Results += [PSCustomObject]@{{Check="Account Exists"; Status="PASS"; Details=$User.DistinguishedName}}
    if ($User.Enabled) {{
        $Results += [PSCustomObject]@{{Check="Account Enabled"; Status="PASS"; Details="Enabled"}}
    }} else {{
        $Results += [PSCustomObject]@{{Check="Account Enabled"; Status="FAIL"; Details="Disabled"}}
    }}
}} else {{
    $Results += [PSCustomObject]@{{Check="Account Exists"; Status="FAIL"; Details="Not found"}}
    $Results | Format-Table Check, Status, Details -AutoSize
    exit 1
}}

# Check 2: AdminCount = 1
if ($User.AdminCount -eq 1) {{
    $Results += [PSCustomObject]@{{Check="AdminCount=1"; Status="PASS"; Details="Set correctly"}}
}} else {{
    $Results += [PSCustomObject]@{{Check="AdminCount=1"; Status="WARN"; Details="Not set"}}
}}

# Check 3: SPN configured
$SPNs = $User.ServicePrincipalNames
if ($SPNs -and $SPNs.Count -gt 0) {{
    $Results += [PSCustomObject]@{{Check="SPN Configured"; Status="PASS"; Details=($SPNs -join ", ")}}
}} else {{
    $Results += [PSCustomObject]@{{Check="SPN Configured"; Status="WARN"; Details="No SPNs"}}
}}

# Check 4: Password age (should appear old)
$PwdAge = (Get-Date) - $User.PasswordLastSet
if ($PwdAge.Days -gt 365) {{
    $Results += [PSCustomObject]@{{Check="Password Age"; Status="PASS"; Details="$($PwdAge.Days) days old"}}
}} else {{
    $Results += [PSCustomObject]@{{Check="Password Age"; Status="WARN"; Details="$($PwdAge.Days) days - consider aging"}}
}}

# Check 5: Audit policy (SACL)
$UserDN = $User.DistinguishedName
$Acl = Get-Acl "AD:\\$UserDN" -Audit
if ($Acl.Audit.Count -gt 0) {{
    $Results += [PSCustomObject]@{{Check="SACL Audit"; Status="PASS"; Details="$($Acl.Audit.Count) audit rules"}}
}} else {{
    $Results += [PSCustomObject]@{{Check="SACL Audit"; Status="WARN"; Details="No audit rules"}}
}}

# Check 6: Group memberships
$Groups = Get-ADPrincipalGroupMembership -Identity $Account | Select-Object -ExpandProperty Name
$Results += [PSCustomObject]@{{Check="Group Memberships"; Status="INFO"; Details=($Groups -join ", ")}}

# Check 7: Encryption types
$EncTypes = $User."msDS-SupportedEncryptionTypes"
if ($EncTypes -band 0x4) {{
    $Results += [PSCustomObject]@{{Check="RC4 Supported"; Status="PASS"; Details="RC4 enabled (attracts Kerberoast)"}}
}} else {{
    $Results += [PSCustomObject]@{{Check="RC4 Supported"; Status="INFO"; Details="RC4 not enabled"}}
}}

# Check 8: Advanced audit policy on DC
$AuditPolicy = auditpol /get /subcategory:"Kerberos Service Ticket Operations" 2>$null
if ($AuditPolicy -match "Success") {{
    $Results += [PSCustomObject]@{{Check="Kerberos Audit"; Status="PASS"; Details="Kerberos TGS auditing enabled"}}
}} else {{
    $Results += [PSCustomObject]@{{Check="Kerberos Audit"; Status="FAIL"; Details="Enable: auditpol /set /subcategory:'Kerberos Service Ticket Operations' /success:enable"}}
}}

Write-Host ""
$Results | Format-Table Check, Status, Details -AutoSize

$FailCount = ($Results | Where-Object {{ $_.Status -eq "FAIL" }}).Count
if ($FailCount -eq 0) {{
    Write-Host "[+] All critical checks passed!" -ForegroundColor Green
}} else {{
    Write-Host "[-] $FailCount checks failed - review above" -ForegroundColor Red
}}
'''


# ===========================================================================
# SIEM Detection Rule Generator
# ===========================================================================

class SIEMRuleGenerator:
    """Generates detection rules for SIEM platforms targeting honeytoken activity."""

    def __init__(self):
        self.rules = []

    def generate_detection_rules(self, honeytoken_accounts: list[str],
                                 honey_spns: list[str],
                                 gpo_trap_accounts: list[str],
                                 siem: str = "sigma") -> list[dict]:
        """Generate detection rules for the specified SIEM platform."""
        generators = {
            "sigma": self._generate_sigma_rules,
            "splunk": self._generate_splunk_rules,
            "sentinel": self._generate_sentinel_rules,
        }

        generator = generators.get(siem)
        if not generator:
            raise ValueError(f"Unsupported SIEM: {siem}. Use: {list(generators.keys())}")

        rules = generator(honeytoken_accounts, honey_spns, gpo_trap_accounts)
        self.rules.extend(rules)
        return rules

    def _generate_sigma_rules(self, accounts: list[str],
                              spns: list[str],
                              gpo_accounts: list[str]) -> list[dict]:
        """Generate Sigma detection rules."""
        rules = []

        # Rule 1: Kerberoasting against honey SPN
        if accounts:
            account_list = "\n".join(f"            - '{a}'" for a in accounts)
            rules.append({
                "title": "Honeytoken Kerberoast Detected",
                "id": str(uuid.uuid4()),
                "status": "production",
                "level": "critical",
                "description": "TGS ticket request for honeytoken service account SPN detected. "
                               "This is a high-confidence indicator of Kerberoasting reconnaissance.",
                "detection_logic": f"EventID 4769 AND ServiceName IN {accounts}",
                "rule": f"""title: Honeytoken Kerberoast Detected
id: {uuid.uuid4()}
status: production
level: critical
description: >
    TGS ticket request detected for a honeytoken service account.
    Any Kerberos ticket request for this account is malicious since
    the associated service does not exist.
references:
    - https://adsecurity.org/?p=3513
    - https://www.hub.trimarcsecurity.com/post/the-art-of-the-honeypot-account-making-the-unusual-look-normal
author: Honeytoken Detection Agent
date: {datetime.utcnow().strftime('%Y/%m/%d')}
tags:
    - attack.credential_access
    - attack.t1558.003
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID: 4769
        ServiceName:
{account_list}
    filter_machine_accounts:
        ServiceName|endswith: '$'
    condition: selection and not filter_machine_accounts
falsepositives:
    - None expected - any match is suspicious
level: critical""",
            })

        # Rule 2: Logon attempt with GPO trap credentials
        if gpo_accounts:
            gpo_list = "\n".join(f"            - '{a}'" for a in gpo_accounts)
            rules.append({
                "title": "Honeytoken GPO Credential Use Detected",
                "id": str(uuid.uuid4()),
                "status": "production",
                "level": "critical",
                "description": "Failed or successful logon using credentials from decoy GPO. "
                               "Attacker has harvested Group Policy Preference passwords.",
                "detection_logic": f"EventID IN (4624, 4625) AND TargetUserName IN {gpo_accounts}",
                "rule": f"""title: Honeytoken GPO Credential Use Detected
id: {uuid.uuid4()}
status: production
level: critical
description: >
    Logon attempt detected using credentials planted in a decoy Group Policy
    Preference XML. The attacker has enumerated SYSVOL and decrypted the
    cpassword value.
references:
    - https://trustedsec.com/blog/weaponizing-group-policy-objects-access
author: Honeytoken Detection Agent
date: {datetime.utcnow().strftime('%Y/%m/%d')}
tags:
    - attack.credential_access
    - attack.t1552.006
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID:
            - 4624
            - 4625
        TargetUserName:
{gpo_list}
    condition: selection
falsepositives:
    - None expected
level: critical""",
            })

        # Rule 3: DACL access on honeytoken object
        if accounts:
            rules.append({
                "title": "Honeytoken AD Object Accessed",
                "id": str(uuid.uuid4()),
                "status": "production",
                "level": "high",
                "description": "Directory service read on honeytoken account DACL detected. "
                               "Indicates AD reconnaissance or enumeration.",
                "detection_logic": f"EventID 4662 AND ObjectName contains honeytoken DN",
                "rule": f"""title: Honeytoken AD Object Accessed
id: {uuid.uuid4()}
status: production
level: high
description: >
    A read operation was performed on a honeytoken AD object's DACL.
    This indicates Active Directory reconnaissance (BloodHound, ADRecon, etc).
references:
    - https://apt29a.blogspot.com/2019/11/deploying-honeytokens-in-active.html
author: Honeytoken Detection Agent
date: {datetime.utcnow().strftime('%Y/%m/%d')}
tags:
    - attack.discovery
    - attack.t1087.002
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID: 4662
        ObjectName|contains:
{"\n".join(f"            - '{a}'" for a in accounts)}
    condition: selection
falsepositives:
    - Legitimate AD administration tools
level: high""",
            })

        return rules

    def _generate_splunk_rules(self, accounts: list[str],
                               spns: list[str],
                               gpo_accounts: list[str]) -> list[dict]:
        """Generate Splunk SPL detection queries."""
        rules = []

        if accounts:
            account_filter = " OR ".join(f'ServiceName="{a}"' for a in accounts)
            rules.append({
                "title": "Honeytoken Kerberoast Detection (Splunk)",
                "detection_logic": f"EventCode=4769 AND ({account_filter})",
                "rule": f"""| `Notable` title="Honeytoken Kerberoast Detected"
index=wineventlog sourcetype="WinEventLog:Security" EventCode=4769
    ({account_filter})
| eval ticket_type=case(
    Ticket_Encryption_Type=="0x17", "RC4-HMAC (weak)",
    Ticket_Encryption_Type=="0x12", "AES256",
    Ticket_Encryption_Type=="0x11", "AES128",
    true(), Ticket_Encryption_Type
)
| eval alert_severity="critical"
| eval alert_type="honeytoken_kerberoast"
| eval mitre_technique="T1558.003"
| table _time, src_ip, Account_Name, ServiceName, ticket_type, Client_Address
| sort - _time""",
            })

        if gpo_accounts:
            gpo_filter = " OR ".join(f'TargetUserName="{a}"' for a in gpo_accounts)
            rules.append({
                "title": "Honeytoken GPO Credential Use (Splunk)",
                "detection_logic": f"EventCode IN (4624,4625) AND ({gpo_filter})",
                "rule": f"""index=wineventlog sourcetype="WinEventLog:Security"
    (EventCode=4624 OR EventCode=4625)
    ({gpo_filter})
| eval alert_severity="critical"
| eval alert_type="honeytoken_gpo_credential_use"
| eval mitre_technique="T1552.006"
| eval logon_result=if(EventCode=4624, "SUCCESS - INVESTIGATE IMMEDIATELY", "Failed")
| table _time, src_ip, TargetUserName, EventCode, logon_result, Logon_Type, Workstation_Name
| sort - _time""",
            })

        # Correlation rule: SYSVOL access followed by credential use
        if gpo_accounts:
            rules.append({
                "title": "Honeytoken Attack Chain: SYSVOL Enum + Credential Use (Splunk)",
                "detection_logic": "Correlation: EventCode 4663 (SYSVOL read) -> 4625 (failed logon)",
                "rule": f"""index=wineventlog sourcetype="WinEventLog:Security"
    (EventCode=4663 ObjectName="*SYSVOL*Policies*Groups.xml*")
    OR (EventCode=4625 ({" OR ".join(f'TargetUserName="{a}"' for a in gpo_accounts)}))
| eval stage=case(
    EventCode=4663, "1_sysvol_enum",
    EventCode=4625, "2_credential_use"
)
| stats earliest(_time) as first_seen, latest(_time) as last_seen,
    values(stage) as attack_stages, dc(EventCode) as event_types
    by src_ip
| where event_types >= 2
| eval alert_type="honeytoken_attack_chain_confirmed"
| eval alert_severity="critical"
| sort - last_seen""",
            })

        return rules

    def _generate_sentinel_rules(self, accounts: list[str],
                                 spns: list[str],
                                 gpo_accounts: list[str]) -> list[dict]:
        """Generate Microsoft Sentinel KQL detection rules."""
        rules = []

        if accounts:
            account_list = ", ".join(f'"{a}"' for a in accounts)
            rules.append({
                "title": "Honeytoken Kerberoast Detection (Sentinel)",
                "detection_logic": f"EventID == 4769 AND ServiceName in ({account_list})",
                "rule": f"""// Honeytoken Kerberoast Detection
// MITRE ATT&CK: T1558.003 - Kerberoasting
// Severity: Critical - ANY match is malicious
SecurityEvent
| where EventID == 4769
| where ServiceName in ({account_list})
| extend EncryptionType = case(
    TicketEncryptionType == "0x17", "RC4-HMAC (weak - easy to crack)",
    TicketEncryptionType == "0x12", "AES256 (strong)",
    TicketEncryptionType == "0x11", "AES128",
    true(), tostring(TicketEncryptionType)
)
| extend AlertSeverity = "Critical"
| extend AlertType = "Honeytoken Kerberoast"
| extend MitreTechnique = "T1558.003"
| project TimeGenerated, Computer, Account, ServiceName,
    IpAddress, EncryptionType, AlertSeverity, AlertType
| sort by TimeGenerated desc""",
            })

        if gpo_accounts:
            gpo_list = ", ".join(f'"{a}"' for a in gpo_accounts)
            rules.append({
                "title": "Honeytoken GPO Credential Use (Sentinel)",
                "detection_logic": f"EventID in (4624,4625) AND TargetUserName in ({gpo_list})",
                "rule": f"""// Honeytoken GPO Credential Trap Triggered
// MITRE ATT&CK: T1552.006 - Group Policy Preferences
// Severity: Critical
SecurityEvent
| where EventID in (4624, 4625)
| where TargetUserName in ({gpo_list})
| extend LogonResult = iff(EventID == 4624,
    "SUCCESS - IMMEDIATE INVESTIGATION REQUIRED", "Failed")
| extend AlertSeverity = "Critical"
| extend AlertType = "Honeytoken GPO Credential Use"
| extend MitreTechnique = "T1552.006"
| project TimeGenerated, Computer, TargetUserName, EventID,
    LogonResult, IpAddress, LogonTypeName, WorkstationName
| sort by TimeGenerated desc""",
            })

        return rules

    def export_rules(self, output_dir: str, format: str = "json") -> list[str]:
        """Export all generated rules to files."""
        out_path = Path(output_dir)
        out_path.mkdir(parents=True, exist_ok=True)
        saved = []

        for i, rule in enumerate(self.rules):
            if format == "json":
                filename = f"rule_{i+1}_{rule['title'].lower().replace(' ', '_')[:40]}.json"
                filepath = out_path / filename
                filepath.write_text(json.dumps(rule, indent=2))
            elif format == "yaml" and "rule" in rule:
                filename = f"rule_{i+1}.yml"
                filepath = out_path / filename
                filepath.write_text(rule["rule"])
            saved.append(str(filepath))

        return saved


# ===========================================================================
# AD Honeytoken Monitor (Python-based log analysis)
# ===========================================================================

class ADHoneytokenMonitor:
    """Monitors Windows Event Logs for honeytoken interactions."""

    def __init__(self, config_path: str | None = None):
        self.config = {}
        if config_path and Path(config_path).exists():
            with open(config_path) as f:
                self.config = json.load(f)
        self.honeytokens: dict[str, dict] = {}
        self.alerts: list[dict] = []

    def register_honeytoken(self, identifier: str,
                            token_type: str = "admin_account",
                            metadata: dict | None = None) -> dict:
        """Register a honeytoken for monitoring."""
        token = {
            "identifier": identifier,
            "type": token_type,
            "registered_at": datetime.utcnow().isoformat(),
            "token_id": f"HT-AD-{uuid.uuid4().hex[:8].upper()}",
            "metadata": metadata or {},
            "alert_count": 0,
        }
        self.honeytokens[identifier] = token
        return token

    def analyze_event_log(self, events: list[dict]) -> list[dict]:
        """Analyze Windows Event Log entries for honeytoken interactions."""
        alerts = []

        for event in events:
            event_id = event.get("EventID") or event.get("EventCode")
            if not event_id:
                continue
            event_id = int(event_id)

            # Check for Kerberoasting (Event 4769)
            if event_id == 4769:
                service_name = event.get("ServiceName", "")
                if service_name in self.honeytokens:
                    enc_type = event.get("TicketEncryptionType", "unknown")
                    alerts.append(self._create_alert(
                        event=event,
                        alert_type="KERBEROAST_HONEYTOKEN",
                        severity="critical",
                        description=f"Kerberoasting detected against honeytoken SPN: {service_name}",
                        mitre_technique="T1558.003",
                        encryption_type=KERBEROS_ENCRYPTION.get(
                            int(enc_type, 16) if isinstance(enc_type, str) else enc_type,
                            str(enc_type)
                        ),
                    ))

            # Check for logon attempts (Event 4624/4625)
            elif event_id in (4624, 4625):
                target_user = event.get("TargetUserName", "")
                if target_user in self.honeytokens:
                    alerts.append(self._create_alert(
                        event=event,
                        alert_type="HONEYTOKEN_LOGON" if event_id == 4624 else "HONEYTOKEN_LOGON_FAILED",
                        severity="critical",
                        description=f"{'Successful' if event_id == 4624 else 'Failed'} "
                                    f"logon attempt with honeytoken account: {target_user}",
                        mitre_technique="T1078" if event_id == 4624 else "T1552.006",
                    ))

            # Check for directory object access (Event 4662)
            elif event_id == 4662:
                object_name = event.get("ObjectName", "")
                for ht_id, ht_info in self.honeytokens.items():
                    if ht_id in object_name:
                        alerts.append(self._create_alert(
                            event=event,
                            alert_type="HONEYTOKEN_DACL_READ",
                            severity="high",
                            description=f"Directory service read on honeytoken object: {ht_id}",
                            mitre_technique="T1087.002",
                        ))

            # Check for GPO modifications (Event 5136)
            elif event_id == 5136:
                object_dn = event.get("ObjectDN", "")
                for ht_id, ht_info in self.honeytokens.items():
                    if ht_info.get("type") == "gpo_credential" and ht_id in object_dn:
                        alerts.append(self._create_alert(
                            event=event,
                            alert_type="HONEYTOKEN_GPO_MODIFIED",
                            severity="critical",
                            description=f"Decoy GPO modification detected: {object_dn}",
                            mitre_technique="T1484.001",
                        ))

        self.alerts.extend(alerts)
        return alerts

    def _create_alert(self, event: dict, alert_type: str,
                      severity: str, description: str,
                      mitre_technique: str, **kwargs) -> dict:
        """Create a structured alert from an event."""
        alert = {
            "alert_id": f"ALERT-{uuid.uuid4().hex[:12].upper()}",
            "alert_type": alert_type,
            "severity": severity,
            "description": description,
            "mitre_technique": mitre_technique,
            "source_ip": event.get("IpAddress") or event.get("src_ip", "unknown"),
            "source_host": event.get("Computer") or event.get("Workstation", "unknown"),
            "account": event.get("TargetUserName") or event.get("ServiceName", "unknown"),
            "event_id": event.get("EventID") or event.get("EventCode"),
            "timestamp": event.get("TimeGenerated") or datetime.utcnow().isoformat(),
            "raw_event": event,
        }
        alert.update(kwargs)
        return alert

    def generate_detection_rules(self, siem: str = "sigma") -> list[dict]:
        """Generate SIEM detection rules for all registered honeytokens."""
        generator = SIEMRuleGenerator()

        accounts = [ht_id for ht_id, info in self.honeytokens.items()
                     if info["type"] in ("admin_account", "spn")]
        spns = [ht_id for ht_id, info in self.honeytokens.items()
                if info["type"] == "spn"]
        gpo_accounts = [ht_id for ht_id, info in self.honeytokens.items()
                        if info["type"] == "gpo_credential"]

        return generator.generate_detection_rules(accounts, spns, gpo_accounts, siem)

    def get_alert_summary(self) -> dict:
        """Get a summary of all triggered alerts."""
        summary = {
            "total_alerts": len(self.alerts),
            "by_severity": {},
            "by_type": {},
            "by_source_ip": {},
            "honeytokens_triggered": set(),
        }

        for alert in self.alerts:
            sev = alert["severity"]
            summary["by_severity"][sev] = summary["by_severity"].get(sev, 0) + 1

            atype = alert["alert_type"]
            summary["by_type"][atype] = summary["by_type"].get(atype, 0) + 1

            src = alert["source_ip"]
            summary["by_source_ip"][src] = summary["by_source_ip"].get(src, 0) + 1

            summary["honeytokens_triggered"].add(alert["account"])

        summary["honeytokens_triggered"] = list(summary["honeytokens_triggered"])
        return summary


# ===========================================================================
# Deployment Orchestrator
# ===========================================================================

class HoneytokenDeployer:
    """Orchestrates full honeytoken deployment and generates all artifacts."""

    def __init__(self, domain: str = "corp.example.com",
                 service_account_ou: str = "OU=Service Accounts",
                 sysvol_path: str = ""):
        self.domain = domain
        self.service_account_ou = service_account_ou
        self.sysvol_path = sysvol_path or f"\\\\{domain}\\SYSVOL\\{domain}\\Policies"
        self.ps_gen = PowerShellGenerator()
        self.siem_gen = SIEMRuleGenerator()
        self.deployed_tokens = []

    def generate_realistic_name(self) -> dict:
        """Generate a realistic service account name."""
        template = secrets.choice(SERVICE_ACCOUNT_TEMPLATES)
        service = secrets.choice(template["services"])
        sam = f"{template['prefix']}{service}"

        # Generate a realistic hostname for SPN
        service_abbrev = service[:3].lower()
        hostname = f"{service_abbrev}-legacy-{secrets.randbelow(99):02d}.{self.domain}"

        return {
            "sam_account_name": sam,
            "display_name": f"{service.replace('_', ' ').title()} Service",
            "hostname": hostname,
        }

    def deploy_full_suite(self, token_count: int = 3,
                          include_spn: bool = True,
                          include_gpo: bool = True,
                          include_bloodhound: bool = True,
                          siem_type: str = "sigma") -> dict:
        """Generate complete deployment artifacts for a full honeytoken suite."""
        deployment = {
            "deployment_id": f"DEPLOY-{uuid.uuid4().hex[:8].upper()}",
            "generated_at": datetime.utcnow().isoformat(),
            "domain": self.domain,
            "tokens": [],
            "scripts": [],
            "detection_rules": [],
        }

        all_accounts = []
        all_spns = []
        gpo_accounts = []

        for i in range(token_count):
            naming = self.generate_realistic_name()
            sam = naming["sam_account_name"]
            ou_dn = f"{self.service_account_ou},DC={',DC='.join(self.domain.split('.'))}"

            # Generate admin account script
            account_script = self.ps_gen.generate_create_honeytoken_account(
                sam_account_name=sam,
                display_name=naming["display_name"],
                description=f"Legacy {naming['display_name'].lower()} - DO NOT DELETE",
                ou_dn=ou_dn,
                password_length=128,
                set_admin_count=True,
            )
            deployment["scripts"].append({
                "type": "create_account",
                "filename": f"01_create_{sam}.ps1",
                "content": account_script,
            })
            all_accounts.append(sam)

            token_info = {
                "name": sam,
                "type": "admin_account",
                "display_name": naming["display_name"],
                "ou": ou_dn,
            }

            # Generate SPN script
            if include_spn:
                spn_class = secrets.choice(SPN_SERVICE_CLASSES)
                port = secrets.choice([1433, 443, 8080, 5432, 3306, 27017])
                spn_script = self.ps_gen.generate_add_honey_spn(
                    sam_account_name=sam,
                    service_class=spn_class,
                    hostname=naming["hostname"],
                    port=port,
                )
                deployment["scripts"].append({
                    "type": "add_spn",
                    "filename": f"02_add_spn_{sam}.ps1",
                    "content": spn_script,
                })
                spn_value = f"{spn_class}/{naming['hostname']}:{port}"
                all_spns.append(spn_value)
                token_info["spn"] = spn_value

            deployment["tokens"].append(token_info)

        # Generate GPO decoy
        if include_gpo:
            gpo_username = f"admin_maintenance_{secrets.randbelow(99):02d}"
            domain_short = self.domain.split(".")[0].upper()
            gpo_script = self.ps_gen.generate_decoy_gpo(
                gpo_name="Server Maintenance Policy (Legacy)",
                decoy_username=gpo_username,
                decoy_domain=domain_short,
                sysvol_path=self.sysvol_path,
            )
            deployment["scripts"].append({
                "type": "decoy_gpo",
                "filename": "03_create_decoy_gpo.ps1",
                "content": gpo_script,
            })
            gpo_accounts.append(gpo_username)
            deployment["tokens"].append({
                "name": gpo_username,
                "type": "gpo_credential",
                "description": "Decoy GPO cpassword trap",
            })

        # Generate BloodHound deception
        if include_bloodhound and all_accounts:
            bh_script = self.ps_gen.generate_deceptive_bloodhound_path(
                honeytoken_sam=all_accounts[0],
            )
            deployment["scripts"].append({
                "type": "bloodhound_deception",
                "filename": "04_create_bloodhound_paths.ps1",
                "content": bh_script,
            })

        # Generate validation script
        if all_accounts:
            val_script = self.ps_gen.generate_validation_script(all_accounts[0])
            deployment["scripts"].append({
                "type": "validation",
                "filename": "05_validate_deployment.ps1",
                "content": val_script,
            })

        # Generate SIEM detection rules
        rules = self.siem_gen.generate_detection_rules(
            all_accounts, all_spns, gpo_accounts, siem_type
        )
        deployment["detection_rules"] = rules

        self.deployed_tokens = deployment["tokens"]
        return deployment

    def save_deployment(self, deployment: dict, output_dir: str) -> list[str]:
        """Save all deployment artifacts to disk."""
        out_path = Path(output_dir)
        out_path.mkdir(parents=True, exist_ok=True)
        saved = []

        # Save PowerShell scripts
        scripts_dir = out_path / "scripts"
        scripts_dir.mkdir(exist_ok=True)
        for script in deployment.get("scripts", []):
            filepath = scripts_dir / script["filename"]
            filepath.write_text(script["content"], encoding="utf-8")
            saved.append(str(filepath))

        # Save detection rules
        rules_dir = out_path / "detection_rules"
        rules_dir.mkdir(exist_ok=True)
        for i, rule in enumerate(deployment.get("detection_rules", [])):
            filename = f"rule_{i+1}_{rule['title'][:40].lower().replace(' ', '_')}.json"
            filepath = rules_dir / filename
            filepath.write_text(json.dumps(rule, indent=2), encoding="utf-8")
            saved.append(str(filepath))

        # Save deployment manifest
        manifest = {
            "deployment_id": deployment["deployment_id"],
            "generated_at": deployment["generated_at"],
            "domain": deployment["domain"],
            "tokens": deployment["tokens"],
            "scripts": [s["filename"] for s in deployment["scripts"]],
            "detection_rules": [r["title"] for r in deployment["detection_rules"]],
        }
        manifest_path = out_path / "deployment_manifest.json"
        manifest_path.write_text(json.dumps(manifest, indent=2))
        saved.append(str(manifest_path))

        return saved


# ===========================================================================
# CLI Entry Point
# ===========================================================================

def main():
    parser = argparse.ArgumentParser(
        description="Active Directory Honeytoken Deployment Agent"
    )
    parser.add_argument(
        "--action",
        choices=[
            "deploy_account", "deploy_spn", "deploy_gpo", "deploy_bloodhound",
            "full_deploy", "generate_rules", "validate", "analyze_logs",
        ],
        default="full_deploy",
        help="Action to perform",
    )
    parser.add_argument("--domain", default="corp.example.com")
    parser.add_argument("--ou", default="OU=Service Accounts")
    parser.add_argument("--sysvol", default="")
    parser.add_argument("--account-name", default="svc_sqlbackup_legacy")
    parser.add_argument("--token-count", type=int, default=3)
    parser.add_argument("--siem", choices=["sigma", "splunk", "sentinel"], default="sigma")
    parser.add_argument("--output-dir", default="honeytoken_deployment")
    parser.add_argument("--include-spn", action="store_true", default=True)
    parser.add_argument("--include-gpo", action="store_true", default=True)
    parser.add_argument("--include-bloodhound", action="store_true", default=True)
    parser.add_argument("--event-log", help="Path to event log JSON for analysis")
    args = parser.parse_args()

    print("=" * 60)
    print("Active Directory Honeytoken Deployment Agent")
    print("=" * 60)

    deployer = HoneytokenDeployer(
        domain=args.domain,
        service_account_ou=args.ou,
        sysvol_path=args.sysvol,
    )

    if args.action == "full_deploy":
        print(f"\n[+] Generating full honeytoken deployment for: {args.domain}")
        print(f"[+] Token count: {args.token_count}")
        print(f"[+] SIEM target: {args.siem}")

        deployment = deployer.deploy_full_suite(
            token_count=args.token_count,
            include_spn=args.include_spn,
            include_gpo=args.include_gpo,
            include_bloodhound=args.include_bloodhound,
            siem_type=args.siem,
        )

        saved_files = deployer.save_deployment(deployment, args.output_dir)

        print(f"\n[+] Deployment ID: {deployment['deployment_id']}")
        print(f"[+] Tokens generated: {len(deployment['tokens'])}")
        for token in deployment["tokens"]:
            print(f"    - {token['name']} ({token['type']})"
                  + (f" SPN: {token.get('spn', 'N/A')}" if token.get('spn') else ""))

        print(f"\n[+] Scripts generated: {len(deployment['scripts'])}")
        for script in deployment["scripts"]:
            print(f"    - {script['filename']} ({script['type']})")

        print(f"\n[+] Detection rules generated: {len(deployment['detection_rules'])}")
        for rule in deployment["detection_rules"]:
            print(f"    - {rule['title']}")

        print(f"\n[+] Files saved to: {args.output_dir}")
        for f in saved_files:
            print(f"    {f}")

    elif args.action == "generate_rules":
        print(f"\n[+] Generating {args.siem} detection rules...")
        monitor = ADHoneytokenMonitor()
        monitor.register_honeytoken(args.account_name, "admin_account")

        rules = monitor.generate_detection_rules(args.siem)
        for rule in rules:
            print(f"\n--- {rule['title']} ---")
            print(rule.get("rule", rule.get("detection_logic", "")))

    elif args.action == "analyze_logs":
        if not args.event_log:
            print("[-] --event-log required for log analysis")
            return

        print(f"\n[+] Analyzing event log: {args.event_log}")
        monitor = ADHoneytokenMonitor()
        monitor.register_honeytoken(args.account_name, "admin_account")

        log_path = Path(args.event_log)
        if not log_path.exists():
            print(f"[-] Log file not found: {args.event_log}")
            return

        with open(log_path) as f:
            events = json.load(f)

        alerts = monitor.analyze_event_log(events)
        print(f"\n[+] Alerts generated: {len(alerts)}")
        for alert in alerts:
            print(f"  [{alert['severity'].upper()}] {alert['alert_type']}: "
                  f"{alert['description']}")
            print(f"    Source: {alert['source_ip']} | "
                  f"Account: {alert['account']} | "
                  f"MITRE: {alert['mitre_technique']}")

        summary = monitor.get_alert_summary()
        print(f"\n[+] Summary: {summary['total_alerts']} alerts, "
              f"sources: {list(summary['by_source_ip'].keys())}")

    elif args.action == "deploy_account":
        ps_gen = PowerShellGenerator()
        ou_dn = f"{args.ou},DC={',DC='.join(args.domain.split('.'))}"
        script = ps_gen.generate_create_honeytoken_account(
            sam_account_name=args.account_name,
            display_name="Legacy Backup Service",
            description="Legacy backup service account - DO NOT DELETE",
            ou_dn=ou_dn,
        )
        print(script)

    elif args.action == "deploy_spn":
        ps_gen = PowerShellGenerator()
        script = ps_gen.generate_add_honey_spn(
            sam_account_name=args.account_name,
        )
        print(script)

    elif args.action == "deploy_gpo":
        ps_gen = PowerShellGenerator()
        script = ps_gen.generate_decoy_gpo(
            gpo_name="Server Maintenance Policy (Legacy)",
            decoy_username="admin_maintenance",
            decoy_domain=args.domain.split(".")[0].upper(),
            sysvol_path=deployer.sysvol_path,
        )
        print(script)

    elif args.action == "deploy_bloodhound":
        ps_gen = PowerShellGenerator()
        script = ps_gen.generate_deceptive_bloodhound_path(
            honeytoken_sam=args.account_name,
        )
        print(script)

    elif args.action == "validate":
        ps_gen = PowerShellGenerator()
        script = ps_gen.generate_validation_script(args.account_name)
        print(script)

    print("\n" + "=" * 60)
    print("[+] Honeytoken agent complete.")
    print("=" * 60)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/deploying-cloud-deception-with-decoy-resources/SKILL.md =====
---
name: deploying-cloud-deception-with-decoy-resources
description: >-
  Deploy cloud-native deception across AWS, Azure, and GCP using decoy (honey) resources
  whose only purpose is to generate a high-fidelity alert the instant an attacker touches
  them: canary IAM access keys, permission-less decoy users/roles/service principals,
  honey object-storage buckets, and decoy secrets in Secrets Manager / Key Vault / Secret
  Manager. Wires detection through CloudTrail + EventBridge, Azure Sentinel honeytoken
  watchlists + Defender, and GCP Cloud Audit Logs, so any use of a decoy is routed to the
  SOC with near-zero false positives. Use when protecting cloud accounts and data stores,
  when an org has only on-prem honeypots and needs cloud coverage, when seeding fake AWS
  keys to catch credential theft and code-leak exposure, or when detecting cloud
  reconnaissance and lateral movement. Keywords: cloud deception, canary token AWS, honey
  S3 bucket, decoy IAM credentials, CloudTrail alert, GuardDuty, Sentinel honeytoken,
  decoy secret, honey service account, cloud honeypot, breach detection.
domain: cybersecurity
subdomain: deception-technology
tags:
- cloud-deception
- aws
- azure
- gcp
- canary-token
- honeytoken
- cloudtrail
- breach-detection
version: "1.0"
author: andrewibrah
license: Apache-2.0
nist_csf:
- DE.CM-01
- DE.CM-06
- DE.AE-02
- ID.RA-01
- RS.MA-01
mitre_attack:
- T1078
- T1552
- T1580
- T1530
- T1619
---

# Deploying Cloud Deception with Decoy Resources

## When to Use

- When cloud accounts (AWS/Azure/GCP) hold crown-jewel data or infrastructure and you need a tripwire that fires the moment an attacker who has gained access starts to operate.
- When the only deception in place is on-prem honeypots, leaving the cloud control plane uninstrumented.
- When seeding fake credentials to catch credential theft, accidental code-repo leaks, or secrets exposed in build pipelines.
- When detecting cloud reconnaissance (enumeration of IAM, storage, or secrets) and lateral movement that legitimate users would never perform.
- When you want detections that survive into incident response with strong fidelity — a touch on a decoy resource almost always means malicious or unauthorized activity.

This is the cloud counterpart to on-prem honeypot/honeytoken/canary-token deployment skills. For program strategy and how these Activities map to adversary engagement goals, use `designing-adversary-engagement-with-mitre-engage`.

## Prerequisites

- Cloud admin/IAM permissions to create decoy principals, storage, secrets, and detection wiring, ideally in a dedicated deployment role with least privilege.
- Cloud audit logging already enabled: **AWS CloudTrail** (multi-region, with management and relevant data events), **Azure Activity log + Microsoft Entra audit/sign-in logs**, **GCP Cloud Audit Logs (Admin Activity always on; Data Access enabled where needed)**.
- A SIEM/alert sink: SNS topic, Microsoft Sentinel workspace, or GCP Pub/Sub + Monitoring, with routing to the SOC.
- A naming and tagging convention that is plausible to an attacker but unambiguous to defenders internally (e.g., realistic names, plus an internal `deception=true` tag/label kept out of attacker-visible metadata).
- **Decoy principals must be permission-less (explicit deny-all).** The value is the alert, never the access. A decoy that grants real privilege is a liability, not a control.

## Workflow

### 1. Decide what to mimic
Pick decoys that match how *your* attackers operate: leaked AWS keys (credential theft), an "admin" S3 bucket (data discovery), a `prod-db-password` secret (secrets harvesting), a privileged-looking service account (cloud lateral movement). Place credential decoys where harvesting tools look: env files, CI variables, code comments, an internal wiki.

### 2A. AWS — canary access keys on a permission-less user
Create a decoy IAM user with an explicit deny-all policy, then issue an access key to plant:
```bash
aws iam create-user --user-name svc-backup-prod --tags Key=deception,Value=true
aws iam put-user-policy --user-name svc-backup-prod \
  --policy-name deny-all \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Deny","Action":"*","Resource":"*"}]}'
aws iam create-access-key --user-name svc-backup-prod   # plant the returned AccessKeyId/Secret
```
Any use of this key appears in CloudTrail (even denied calls, which still log `AccessDenied`). Wire an EventBridge rule on CloudTrail to alert:
```bash
aws events put-rule --name decoy-key-used \
  --event-pattern '{"detail":{"userIdentity":{"userName":["svc-backup-prod"]}}}'
aws events put-targets --rule decoy-key-used \
  --targets "Id"="1","Arn"="arn:aws:sns:us-east-1:111111111111:soc-deception-alerts"
```

### 2B. AWS — honey S3 bucket
Create a believable bucket, enable object-level data events, and alert on any read/list:
```bash
aws s3api create-bucket --bucket acme-prod-db-backups-2026 --region us-east-1
aws s3api put-bucket-tagging --bucket acme-prod-db-backups-2026 \
  --tagging 'TagSet=[{Key=deception,Value=true}]'
# Ensure CloudTrail captures S3 data events for this bucket, then alert on GetObject/ListBucket
aws events put-rule --name decoy-bucket-access \
  --event-pattern '{"detail":{"eventSource":["s3.amazonaws.com"],"requestParameters":{"bucketName":["acme-prod-db-backups-2026"]}}}'
```

### 2C. AWS — decoy secret
```bash
aws secretsmanager create-secret --name prod/db/master-password \
  --secret-string '{"username":"dbadmin","password":"DECOY-DO-NOT-USE"}' \
  --tags Key=deception,Value=true
# Alert on GetSecretValue for this secret via EventBridge -> SNS
```

### 3A. Azure — honeytoken watchlist + decoy service principal
Microsoft Sentinel natively supports honeytokens via a **Watchlist** of the `HoneyTokens` template; tagged decoy accounts/secrets raise analytics alerts on use. Create a permission-less decoy app registration / service principal, then add its identifiers to the HoneyTokens watchlist and enable the related analytics rules. Microsoft Defender for Cloud and Entra ID Protection surface anomalous sign-ins to the decoy identity.

### 3B. Azure — honey storage + Key Vault decoy secret
Create a decoy Storage account and Key Vault, enable diagnostic logging to the Sentinel workspace, store a decoy secret, and write an analytics rule that fires on any data-plane read of the decoy resources.

### 4A. GCP — decoy service account + honey GCS bucket
Create a service account with no role bindings (permission-less), generate a key to plant, and alert on its use via Cloud Audit Logs:
```bash
gcloud iam service-accounts create svc-billing-export \
  --display-name="billing-export"
gcloud iam service-accounts keys create decoy-key.json \
  --iam-account=svc-billing-export@PROJECT.iam.gserviceaccount.com   # plant this key
gsutil mb -b on gs://acme-finance-exports-2026
```
Create a log-based metric + alerting policy in Cloud Monitoring that triggers on any audit-log entry where the principal is the decoy service account or the resource is the honey bucket.

### 5. Centralize and de-duplicate
Route all clouds' decoy alerts to one SOC pipeline. Tag each alert as DECEPTION/high-fidelity so it bypasses normal noise filtering and triggers an IR playbook rather than a triage queue.

### 6. Validate (red-team the decoys)
Have an authorized tester use each decoy (read the bucket, call with the key, fetch the secret) and confirm an alert lands end-to-end within target latency. A decoy you have not tested is assumed broken.

### 7. Maintain realism and rotate
Refresh decoy names, secrets, and pocket-litter periodically so they age with the real environment. Track every decoy in an inventory so they are never mistaken for real assets during audits or cleanups.

## Key Concepts

| Concept | Definition |
|---|---|
| Decoy / honey resource | A cloud object created solely to be touched by an attacker; no legitimate user has any reason to use it. |
| Canary access key | A planted credential whose use generates an audit-log event; carries deny-all permissions. |
| High-fidelity alert | A near-zero-false-positive signal because legitimate workflows never reference the decoy. |
| Permission-less principal | A decoy IAM user/role/service principal/service account with explicit deny-all or no role bindings. |
| Data event | Cloud audit logging of object/data-plane access (e.g., S3 GetObject), required to detect storage decoys. |
| Pocket litter | Plausible supporting artifacts (fake configs, env files, wiki entries) that make a decoy credible. |
| Decoy inventory | The authoritative internal record distinguishing decoys from real assets. |

## Tools & Systems

- **AWS** — IAM (decoy users/roles), S3 (honey buckets, data events), Secrets Manager / SSM Parameter Store (decoy secrets), CloudTrail, EventBridge, SNS/Lambda, GuardDuty (correlate anomalous use).
- **Azure** — Microsoft Entra ID (decoy app registrations / service principals), Storage / Key Vault decoys, **Microsoft Sentinel HoneyTokens watchlist** and analytics rules, Microsoft Defender for Cloud, Entra ID Protection.
- **GCP** — IAM service accounts (decoys), Cloud Storage (honey buckets), Secret Manager (decoy secrets), Cloud Audit Logs, log-based metrics + Cloud Monitoring alerting, Pub/Sub.
- **Open-source / managed honeytoken systems** — Canarytokens (https://canarytokens.org offers AWS API key tokens), Thinkst Canary, SpaceSiren / SpaceCrab (self-hosted AWS honey-token frameworks).
- **SIEM/SOAR** — to centralize alerts across clouds and drive an IR playbook on any decoy hit.

## Common Scenarios

- **Credential-theft / code-leak detection.** Plant a canary AWS key in CI variables, an env file, and a private repo. Any external use (even from a leaked public push) fires within minutes.
- **Crown-jewel data store.** Stand up a honey "backups" bucket next to the real one; attackers enumerating storage hit the decoy first and reveal themselves.
- **Cloud lateral movement.** A permission-less decoy service principal that "looks" privileged catches adversaries assuming roles during pivoting.
- **Secrets harvesting.** Decoy entries in Secrets Manager / Key Vault / Secret Manager detect tools scraping the secrets store.
- **Migrating from on-prem-only deception.** Mirror the existing on-prem decoy strategy into the cloud control plane so coverage follows workloads.

## Output Format

Produce a **Cloud Deception Deployment Record** using `assets/template.md`, containing:

1. **Decoy inventory** — per decoy: cloud, type, plausible name, real placement location of any planted credential, internal `deception` tag/label, owner.
2. **Detection wiring** — per decoy: audit-log source → rule/pattern → alert sink → IR playbook reference, with the target alert latency.
3. **Least-privilege proof** — evidence each decoy principal is deny-all / no-role-binding.
4. **Validation results** — date tested, who tested, end-to-end latency observed, pass/fail.
5. **Maintenance plan** — rotation cadence and review owner.

Use `scripts/process.py` to render the deployment record and a per-decoy detection checklist from a decoy-inventory JSON, and to flag decoys missing detection wiring or validation.

===== END FILE =====


===== FILE: skills/deploying-cloud-deception-with-decoy-resources/LICENSE =====

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


===== FILE: skills/deploying-cloud-deception-with-decoy-resources/assets/template.md =====
# Cloud Deception Deployment Record

> Worked example. Keep this record internal and separate from any attacker-visible metadata.

**Account / project:** acme-prod (AWS 111111111111) · **Owner:** [Cloud security lead] · **Last validated:** 2026-05-20

## 1. Decoy inventory & detection wiring
| Decoy | Cloud | Type | Plausible placement | Detection: source → rule → sink → playbook | Deny-all? | Target latency |
|---|---|---|---|---|---|---|
| svc-backup-prod (key) | AWS | Canary access key | CI variables, repo `.env`, internal wiki | CloudTrail → `decoy-key-used` → `sns:soc-deception-alerts` → IR-CLOUD-07 | Yes | < 5 min |
| acme-prod-db-backups-2026 | AWS | Honey S3 bucket | Discoverable via S3 list | CloudTrail data events → `decoy-bucket-access` → SNS → IR-CLOUD-07 | n/a | < 5 min |
| prod/db/master-password | AWS | Decoy secret | Secrets Manager | CloudTrail `GetSecretValue` → `decoy-secret-read` → SNS → IR-CLOUD-07 | n/a | < 5 min |
| h* sentinel honeytoken acct | Azure | Decoy service principal | Entra app registration | Sentinel HoneyTokens watchlist → analytics rule → SOC → IR-CLOUD-07 | Yes (no roles) | < 10 min |
| svc-billing-export | GCP | Decoy service account key | Build config | Cloud Audit Logs → log-based metric → Monitoring alert → IR-CLOUD-07 | Yes (no bindings) | < 10 min |

## 2. Least-privilege proof
- AWS decoy users: explicit `Deny *` inline policy attached (`deny-all`); verified with `aws iam get-user-policy`.
- Azure decoy SP: zero role assignments; verified in Entra.
- GCP decoy SA: zero IAM policy bindings; verified with `gcloud iam service-accounts get-iam-policy`.

## 3. Validation results (red-team each decoy)
| Decoy | Tested | By | Observed latency | Pass/Fail |
|---|---|---|---|---|
| svc-backup-prod | 2026-05-20 | Red team | 90 s | PASS |
| acme-prod-db-backups-2026 | 2026-05-20 | Red team | 2 min | PASS |
| prod/db/master-password | 2026-05-20 | Red team | 75 s | PASS |
| Azure honeytoken SP | 2026-05-20 | Red team | 6 min | PASS |
| svc-billing-export | 2026-05-20 | Red team | 4 min | PASS |

## 4. Maintenance plan
- **Rotation cadence:** Refresh names/secrets/pocket-litter quarterly so decoys age with prod.
- **Review owner:** [Cloud security lead], quarterly.
- **Inventory rule:** No decoy is deleted during clean-ups/audits without confirming against this record.

===== END FILE =====


===== FILE: skills/deploying-cloud-deception-with-decoy-resources/references/standards.md =====
# Cloud Deception — Standards & Reference

## Detection foundations (audit logging is mandatory)
Cloud deception only works if a decoy touch is logged. Confirm these before deploying.

### AWS
- **CloudTrail** — management events plus **data events** (S3 object-level, Lambda invoke, etc.) for any storage/secret decoys. Multi-region trail recommended.
- **EventBridge** — pattern-match CloudTrail events on `userIdentity.userName`, `eventSource`, or `requestParameters.bucketName` and target SNS/Lambda.
- **GuardDuty** — correlates anomalous credential/API behavior; useful to enrich a decoy hit.
- Note: even *denied* API calls by a deny-all decoy principal are recorded in CloudTrail as `AccessDenied`, so the alert fires regardless of granted permission.

### Azure / Microsoft
- **Azure Activity log** + **Microsoft Entra ID audit and sign-in logs** streamed to a Log Analytics / Microsoft Sentinel workspace.
- **Microsoft Sentinel HoneyTokens** — built-in watchlist template; decoy identifiers added to the watchlist drive analytics rules that alert on use.
- **Microsoft Defender for Cloud** and **Entra ID Protection** — surface anomalous access to decoy identities.
- Enable diagnostic settings on decoy Storage accounts and Key Vaults to capture data-plane reads.

### GCP
- **Cloud Audit Logs** — Admin Activity logs are always on; enable **Data Access** logs for the services hosting decoys (Cloud Storage, Secret Manager, IAM).
- **Log-based metrics + Cloud Monitoring alerting policies** — trigger on audit entries where `protoPayload.authenticationInfo.principalEmail` is the decoy service account or the resource is the honey bucket.
- **Pub/Sub** sink to forward to a SIEM.

## MITRE D3FEND — Deceive tactic mappings
| D3FEND technique | Cloud decoy realization |
|---|---|
| Decoy User Credential | Canary IAM access key / decoy app secret / decoy SA key |
| Decoy Network Resource | Honey S3 / GCS / Azure Storage bucket |
| Decoy Object | Decoy secret in Secrets Manager / Key Vault / Secret Manager |
| Decoy Persona | Permission-less decoy IAM user / service principal / service account |
| Decoy Session Token | Planted temporary credential / SAS token |

## MITRE ATT&CK techniques detected
| Technique | Detected by decoy |
|---|---|
| T1078 / T1078.004 Valid Accounts (Cloud) | Canary key / decoy principal use |
| T1552 / T1552.001 Unsecured Credentials | Decoy secret read; planted credential use |
| T1580 Cloud Infrastructure Discovery | Enumeration touching decoy principals/resources |
| T1619 Cloud Storage Object Discovery | List on honey bucket |
| T1530 Data from Cloud Storage Object | GetObject on honey bucket |

## NIST CSF 2.0 alignment
| CSF 2.0 ID | Relevance |
|---|---|
| DE.CM-01 | Networks/environments monitored to find adverse events |
| DE.CM-06 | External service provider (cloud) activity monitored |
| DE.AE-02 | Potentially adverse events analyzed — decoy alert triage |
| ID.RA-01 | Vulnerabilities/exposures identified — informs decoy placement |
| RS.MA-01 | Incident management — decoy hit invokes the IR playbook |

## Tooling references
- Canarytokens (AWS API key token): https://canarytokens.org
- Thinkst Canary: https://canary.tools
- SpaceSiren (self-hosted AWS honey tokens, serverless): open-source
- Microsoft Sentinel HoneyTokens watchlist: Microsoft Learn — "Deploy decoys/honeytokens with Sentinel"
- AWS CloudTrail data events: AWS docs — "Logging data events"
- GCP Cloud Audit Logs: Google Cloud docs — "Cloud Audit Logs overview"

## Operating principles
- **Deny-all decoys only.** Decoy principals must carry an explicit deny-all policy (AWS) or no role bindings (GCP) / no privileged roles (Azure). The control's value is the alert, never access.
- **Keep an internal decoy inventory** separate from attacker-visible metadata so audits and clean-ups never delete a tripwire by accident, and real assets are never mistaken for decoys.
- **Validate end-to-end** (red-team each decoy) and record observed alert latency. Untested decoys are assumed non-functional.
- **Mark deception alerts high-fidelity** so they bypass routine noise filtering and go straight to an IR playbook.

===== END FILE =====


===== FILE: skills/deploying-cloud-deception-with-decoy-resources/scripts/process.py =====
#!/usr/bin/env python3
"""
Cloud deception deployment validator.

Reads a decoy-inventory JSON, checks each decoy for the controls that make a
cloud decoy trustworthy (detection wiring, deny-all/least-privilege, validation,
internal tagging), renders a Cloud Deception Deployment Record, and exits non-zero
if any decoy is missing a required control.

Usage:
    python process.py --inventory decoys.json --out record.md

decoys.json format:
    {
      "account": "acme-prod (AWS 1111...)",
      "decoys": [
        {
          "name": "svc-backup-prod",
          "cloud": "aws",
          "type": "canary_access_key",
          "placement": "CI variables + private repo .env",
          "deny_all": true,
          "detection": {"source": "CloudTrail", "rule": "decoy-key-used", "sink": "sns:soc-deception-alerts", "playbook": "IR-CLOUD-07"},
          "validated": {"date": "2026-05-20", "by": "redteam", "latency_sec": 90, "passed": true},
          "internal_tag": "deception=true"
        }
      ]
    }
"""
import argparse
import json
import sys
from datetime import date

REQUIRED = ["name", "cloud", "type", "placement"]
VALID_CLOUDS = {"aws", "azure", "gcp"}


def check_decoy(d):
    """Return list of issues for one decoy."""
    issues = []
    for k in REQUIRED:
        if not d.get(k):
            issues.append(f"missing required field '{k}'")
    if d.get("cloud") and d["cloud"].lower() not in VALID_CLOUDS:
        issues.append(f"unknown cloud '{d.get('cloud')}'")

    det = d.get("detection") or {}
    for k in ("source", "rule", "sink"):
        if not det.get(k):
            issues.append(f"detection wiring missing '{k}' (decoy is blind)")
    if not det.get("playbook"):
        issues.append("no IR playbook reference for the alert")

    # Credential/principal decoys must be deny-all / least privilege.
    cred_types = {"canary_access_key", "decoy_principal", "decoy_service_account",
                  "decoy_service_principal", "decoy_iam_user"}
    if d.get("type") in cred_types and not d.get("deny_all"):
        issues.append("credential/principal decoy is not marked deny_all (liability risk)")

    val = d.get("validated") or {}
    if not val.get("passed"):
        issues.append("not validated end-to-end (assume non-functional)")
    if not d.get("internal_tag"):
        issues.append("no internal deception tag/label (audit-cleanup risk)")
    return issues


def render(inv, results):
    lines = [f"# Cloud Deception Deployment Record",
             f"\n_Account: {inv.get('account', 'UNKNOWN')} — generated {date.today().isoformat()}_\n",
             "## 1. Decoy inventory & detection wiring",
             "| Decoy | Cloud | Type | Placement | Detection (source→rule→sink) | Playbook | Validated | Status |",
             "|---|---|---|---|---|---|---|---|"]
    for d, issues in results:
        det = d.get("detection") or {}
        wiring = f"{det.get('source','?')}→{det.get('rule','?')}→{det.get('sink','?')}"
        val = d.get("validated") or {}
        vstr = f"{val.get('date','-')} ({val.get('latency_sec','?')}s)" if val.get("passed") else "NO"
        status = "OK" if not issues else f"{len(issues)} issue(s)"
        lines.append(f"| {d.get('name','?')} | {d.get('cloud','?')} | {d.get('type','?')} | "
                     f"{d.get('placement','?')} | {wiring} | {det.get('playbook','-')} | {vstr} | {status} |")

    problem = [(d, i) for d, i in results if i]
    if problem:
        lines.append("\n## 2. Issues to remediate before relying on these decoys")
        for d, issues in problem:
            lines.append(f"\n**{d.get('name','?')}** ({d.get('cloud','?')}):")
            for it in issues:
                lines.append(f"- {it}")
    else:
        lines.append("\n## 2. Issues\nNone — all decoys have detection wiring, least privilege, and validation.")

    lines.append("\n## 3. Maintenance")
    lines.append("- Rotation cadence: TBD  ·  Review owner: TBD")
    lines.append("- Keep this record separate from attacker-visible metadata.")
    return "\n".join(lines)


def main():
    p = argparse.ArgumentParser(description="Cloud deception deployment validator")
    p.add_argument("--inventory", required=True, help="Path to decoy-inventory JSON")
    p.add_argument("--out", help="Output markdown path (default: stdout)")
    args = p.parse_args()

    with open(args.inventory) as f:
        inv = json.load(f)

    decoys = inv.get("decoys", [])
    results = [(d, check_decoy(d)) for d in decoys]
    total_issues = sum(len(i) for _, i in results)
    healthy = sum(1 for _, i in results if not i)

    print(f"{len(decoys)} decoy(s): {healthy} healthy, {total_issues} total issue(s).",
          file=sys.stderr)

    out = render(inv, results)
    if args.out:
        with open(args.out, "w") as f:
            f.write(out + "\n")
        print(f"Wrote deployment record -> {args.out}", file=sys.stderr)
    else:
        print(out)

    # Non-zero exit if any decoy has issues, so this can gate a pipeline.
    sys.exit(1 if total_issues else 0)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/deploying-honeytokens-and-canarytokens/SKILL.md =====
---
name: deploying-honeytokens-and-canarytokens
description: Plants Canarytokens-based decoy artifacts (honey credentials, DNS tokens, web-bug URLs, AWS keys, documents, kubeconfigs) using Thinkst's open-source Canarytokens project and alerts via email or webhook when a token is touched. Use for high-fidelity intrusion detection in low-telemetry areas like file shares or credential stores, or to catch credential dumping and data-theft staging.
domain: cybersecurity
subdomain: deception-technology
tags:
- deception-technology
- canarytokens
- honeytokens
- breach-detection
- threat-detection
- d3fend
- decoy-credentials
- intrusion-detection
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
mitre_attack:
- T1556
---
# Deploying Honeytokens and Canarytokens

> **Authorized Use Only:** Deception assets described here are defensive controls deployed inside your own environment. Deploying tokens, decoy credentials, or honeypots on infrastructure you do not own or administer, or using them to entrap third parties, may violate computer-misuse and privacy law. Deploy only on assets you own or are explicitly authorized to instrument, and route all alert data through approved monitoring channels.

## Overview

Honeytokens (a.k.a. canarytokens) are decoy artifacts — credentials, files, URLs, API keys, DNS names, database connection strings, documents — that have no legitimate operational use. Because no authorized user or process should ever touch them, **any** interaction is a high-fidelity signal of an intrusion, insider misuse, or reconnaissance. Unlike signature- or anomaly-based detection, honeytokens generate near-zero false positives: the alert *is* the compromise.

Thinkst's open-source **Canarytokens** project (https://canarytokens.org and the self-hostable `thinkst/canarytokens-docker`) generates dozens of token types that "phone home" when triggered: an HTTP/web-bug URL that fires on GET, an AWS API key that fires when used against AWS, an MS Word/PDF document that fires on open, a DNS token that fires on resolution, a Slack API token, a Kubernetes `kubeconfig`, an Azure login certificate, a `log4shell` payload, and more. Each token is bound to a unique `memo` (so you know *where* it was planted) plus an alert channel (email and/or webhook).

This skill maps to MITRE D3FEND's **Decoy File (D3-DF)**, **Decoy User Credential (D3-DUC)**, and **Honeytoken** techniques. From an ATT&CK perspective, a triggered honey credential most commonly evidences adversary attempts to abuse or modify authentication material (**T1556 – Modify Authentication Process** and related credential-access activity), giving the SOC an early, unambiguous tripwire deep inside the kill chain — typically after initial access but before lateral movement completes.

## When to Use

- When you need high-fidelity intrusion detection in segments where traditional telemetry is sparse (file shares, password vaults, code repos, cloud accounts).
- When validating that an attacker who reaches a "crown-jewel" host or document store is detected, not just blocked at the perimeter.
- When seeding decoy credentials into LSASS-reachable memory, browser stores, `.aws/credentials`, or password managers to catch credential dumping and reuse.
- When instrumenting documents, repos, or wikis to catch data theft and ransomware staging.
- When building a MITRE D3FEND-aligned deception layer as part of a defense-in-depth or zero-trust program.

## Prerequisites

- Docker Engine and Docker Compose v2 for self-hosting (`docker compose version`).
- A registered domain you control plus DNS delegation for DNS-based tokens (NS records pointing at your switchboard host).
- A public IPv4 address reachable on 80/443 (HTTP tokens) and 53/udp (DNS tokens).
- An SMTP relay or Mailgun account, and/or a Slack/Teams/generic webhook URL for alert delivery.
- Python 3.8+ for the helper script:
  ```bash
  python3 -m pip install requests
  ```
- For quick use with no hosting, an account-free token from the public service at https://canarytokens.org.

## Objectives

- Stand up a self-hosted Canarytokens instance (or use the public service) with working alerting.
- Generate the major token types (HTTP, DNS, AWS key, MS Word/PDF, Slack, kubeconfig) with descriptive memos.
- Plant decoy credentials and decoy files in realistic, monitored locations.
- Validate that each token fires and that alerts reach the SOC channel.
- Catalogue deployed tokens and map them to MITRE D3FEND/ATT&CK for coverage tracking.

## MITRE ATT&CK Mapping

| ID | Official Technique Name | Relevance |
|----|------------------------|-----------|
| T1556 | Modify Authentication Process | A triggered honey credential reveals an adversary harvesting/abusing authentication material; the decoy provides a detection tripwire for credential abuse activity. |

**Related MITRE D3FEND defensive techniques** (the offensive counter-mapping for this control):

| D3FEND ID | Technique | Role |
|-----------|-----------|------|
| D3-DF | Decoy File | Canary documents, fake configs, decoy archives placed and monitored. |
| D3-DUC | Decoy User Credential | Honey credentials (AWS keys, AD accounts, kubeconfig) integrated with a monitored decoy asset. |
| D3-DO | Decoy Object | Umbrella for honeytokens/canarytokens as monitored decoy artifacts. |

## Workflow

### 1. Deploy a self-hosted Canarytokens switchboard
Clone the official Docker repo and create the two environment files from their distributed templates:
```bash
git clone https://github.com/thinkst/canarytokens-docker
cd canarytokens-docker
cp switchboard.env.dist switchboard.env
cp frontend.env.dist frontend.env
```
Set the core variables. In `frontend.env`:
```ini
CANARY_DOMAINS=canary.example.com          # general-purpose token domains (comma-separated)
CANARY_NXDOMAINS=nx.example.com            # domains reserved for PDF/DNS tokens
CANARY_PUBLIC_IP=203.0.113.10              # public IP of this host
```
In `switchboard.env`:
```ini
CANARY_PUBLIC_DOMAIN=canary.example.com
CANARY_MAILGUN_DOMAIN_NAME=mg.example.com
CANARY_MAILGUN_API_KEY=key-xxxxxxxxxxxxxxxx
CANARY_ALERT_EMAIL_FROM_ADDRESS=canary@example.com
CANARY_ALERT_EMAIL_FROM_DISPLAY=Canarytokens
CANARY_ALERT_EMAIL_SUBJECT=Canarytoken Triggered
# WireGuard token seed:
# dd bs=32 count=1 if=/dev/urandom 2>/dev/null | base64
CANARY_WG_PRIVATE_KEY_SEED=<base64-seed>
```

### 2. Bring the stack up
```bash
docker compose up -d
# HTTPS with automatic Let's Encrypt certs (after editing certbot.env):
# docker compose -f docker-compose-letsencrypt.yml up -d
docker compose ps
docker compose logs -f frontend
```
The frontend (token generator UI) is now served on your `CANARY_PUBLIC_DOMAIN`; the switchboard listens on 80/443 and 53/udp to receive triggers.

### 3. Generate an HTTP (web-bug) token via the API
Both the public service and a self-hosted frontend expose a `POST /generate` endpoint. Create an HTTP token that fires on any GET:
```bash
curl -s https://canarytokens.org/generate \
  -F 'type=http' \
  -F 'email=soc@example.com' \
  -F 'memo=Internal wiki - IT admin passwords page' \
  -F 'webhook_url=https://hooks.slack.com/services/T000/B000/XXXX'
# Response JSON includes: token, auth, hostname, url, url_components
```
The returned `url` is the trip-wire link; place it where only an intruder would find it (a fake bookmark, a hidden link in a wiki page, an email signature).

### 4. Generate a cloud credential (AWS API key) honeytoken
```bash
curl -s https://canarytokens.org/generate \
  -F 'type=aws_keys' \
  -F 'email=soc@example.com' \
  -F 'memo=Decoy AWS keys - jenkins build host /root/.aws/credentials'
# Response contains access_key_id and secret_access_key plus a downloadable
# credentials file via /download?token=<token>&auth=<auth>&fmt=aws_keys
```
Drop the keys into a plausible `~/.aws/credentials` on a monitored host. Any `sts:GetCallerIdentity` or other AWS call using them triggers an alert with the source IP and user agent.

### 5. Generate a document token (MS Word / PDF) for data-theft detection
```bash
# MS Word
curl -s https://canarytokens.org/generate \
  -F 'type=msword' \
  -F 'email=soc@example.com' \
  -F 'memo=Q4-Layoffs-DRAFT.docx on FILESERVER01\\HR$' \
  -o token-meta.json
# Download the weaponized document:
curl -s "https://canarytokens.org/download?fmt=msword&token=<token>&auth=<auth>" -o Q4-Layoffs-DRAFT.docx
```
For PDFs use `type=adobe_pdf`. The document phones home when opened (DNS/HTTP callback), exposing the reader's IP.

### 6. Plant a DNS token for resolver-level tripwires
```bash
curl -s https://canarytokens.org/generate \
  -F 'type=dns' \
  -F 'email=soc@example.com' \
  -F 'memo=DNS canary referenced in backup script comments'
# Response 'hostname' is a unique FQDN; any resolution of it fires an alert.
```
Embed the hostname in scripts, configs, or `/etc/hosts` comments. Because resolution alone triggers it, DNS tokens catch reconnaissance even when egress HTTP is blocked.

### 7. Generate infrastructure tokens (Slack, kubeconfig, Azure)
```bash
# Slack API token canary (fires when the fake token is used against Slack)
curl -s https://canarytokens.org/generate -F 'type=slack_api' \
  -F 'email=soc@example.com' -F 'memo=Decoy Slack bot token in repo .env'

# Kubeconfig canary (fires when used against the kube API)
curl -s https://canarytokens.org/generate -F 'type=kubeconfig' \
  -F 'email=soc@example.com' -F 'memo=Decoy kubeconfig in /home/deploy/.kube/config'
```
Commit decoy `.env` / `kubeconfig` files only to repos and hosts you instrument, never to public repos.

### 8. Plant Active Directory honey credentials (decoy user)
Create a non-privileged-looking but never-used AD account whose authentication is alerted on. Set a SPN so it appears Kerberoastable bait, and forward Event ID 4768/4769/4625 for it to your SIEM:
```powershell
New-ADUser -Name "svc_backup_legacy" -SamAccountName "svc_backup_legacy" `
  -AccountPassword (ConvertTo-SecureString 'C0mpl3xDecoy!2026' -AsPlainText -Force) `
  -Enabled $true -Description "Legacy backup service (do not use)"
Set-ADUser svc_backup_legacy -ServicePrincipalNames @{Add="MSSQLSvc/decoy.example.com:1433"}
```
Add a Windows audit ACL/SACL or a SIEM correlation rule so any 4768/4769 for `svc_backup_legacy` pages the SOC — no legitimate logon should ever occur.

### 9. Validate and catalogue
Trigger each token from a controlled host and confirm the alert lands:
```bash
# HTTP token
curl -s "https://canary.example.com/<token-url>" >/dev/null
# DNS token
dig +short <unique-hostname>
# AWS key token
AWS_ACCESS_KEY_ID=<id> AWS_SECRET_ACCESS_KEY=<secret> aws sts get-caller-identity
```
Record each deployed token (type, memo, location, alert channel, D3FEND mapping) in an inventory. Use the helper script below to bulk-generate and export this inventory as JSON.

## Tools and Resources

| Tool / Resource | Purpose | Link |
|-----------------|---------|------|
| Canarytokens (public) | Free hosted token generation | https://canarytokens.org |
| canarytokens-docker | Self-hosted switchboard + frontend | https://github.com/thinkst/canarytokens-docker |
| canarytokens (core) | Source for the token engine | https://github.com/thinkst/canarytokens |
| Canarytokens docs | Per-token-type usage guides | https://docs.canarytokens.org |
| MITRE D3FEND | Defensive technique taxonomy (D3-DF, D3-DUC) | https://d3fend.mitre.org |
| MITRE ATT&CK T1556 | Modify Authentication Process | https://attack.mitre.org/techniques/T1556/ |

## Token Type Reference

| `type` value | Token | Fires when |
|--------------|-------|-----------|
| `http` | Web-bug URL | URL is requested (GET) |
| `dns` | DNS name | Hostname is resolved |
| `aws_keys` | AWS API key | Keys used against AWS |
| `msword` / `adobe_pdf` | Office/PDF document | Document is opened |
| `slack_api` | Slack API token | Token used against Slack |
| `kubeconfig` | Kubernetes config | Used against the kube API |
| `azure_id` | Azure login certificate | Cert used to authenticate to Azure |
| `qr_code` | QR code | Encoded URL is requested |
| `web_image` | Image web-bug | Image is loaded |
| `log4shell` | Log4j JNDI string | Vulnerable logger evaluates it |
| `cmd` | Sensitive command (Windows) | Process/command is executed |

## Validation Criteria

- [ ] Self-hosted switchboard (or public service) deployed and reachable on 80/443 and 53/udp.
- [ ] Alert channel (email and/or webhook) configured and test alert received.
- [ ] At least one each of HTTP, DNS, cloud-credential, and document tokens generated with descriptive memos.
- [ ] Decoy credentials planted in realistic, monitored locations (no production secrets co-located).
- [ ] AD honey account created with SACL/SIEM rule firing on any authentication.
- [ ] Each token validated by a controlled trigger; alert confirmed end-to-end.
- [ ] Token inventory exported (type, memo, location, alert channel, D3-DF/D3-DUC mapping).
- [ ] No tokens committed to public repositories or planted on out-of-scope systems.

===== END FILE =====


===== FILE: skills/deploying-honeytokens-and-canarytokens/LICENSE =====

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


===== FILE: skills/deploying-honeytokens-and-canarytokens/references/api-reference.md =====
# Canarytokens API and Deployment Reference

## Self-hosting (canarytokens-docker)

| Step | Command |
|------|---------|
| Clone | `git clone https://github.com/thinkst/canarytokens-docker` |
| Config (switchboard) | `cp switchboard.env.dist switchboard.env` |
| Config (frontend) | `cp frontend.env.dist frontend.env` |
| Start (HTTP) | `docker compose up -d` |
| Start (Let's Encrypt) | `docker compose -f docker-compose-letsencrypt.yml up -d` |
| Status / logs | `docker compose ps` / `docker compose logs -f frontend` |

### Key environment variables

| Variable | File | Purpose |
|----------|------|---------|
| `CANARY_DOMAINS` | frontend.env | Comma-separated domains for general-purpose tokens |
| `CANARY_NXDOMAINS` | frontend.env | Domains reserved for PDF/DNS tokens |
| `CANARY_PUBLIC_IP` | frontend.env | Public IPv4 of the host |
| `CANARY_PUBLIC_DOMAIN` | switchboard.env | Domain serving the frontend |
| `CANARY_MAILGUN_DOMAIN_NAME` | switchboard.env | Mailgun domain for email alerts |
| `CANARY_MAILGUN_API_KEY` | switchboard.env | Mailgun API key |
| `CANARY_ALERT_EMAIL_FROM_ADDRESS` | switchboard.env | Alert sender address |
| `CANARY_ALERT_EMAIL_FROM_DISPLAY` | switchboard.env | Alert sender display name |
| `CANARY_ALERT_EMAIL_SUBJECT` | switchboard.env | Alert email subject |
| `CANARY_WG_PRIVATE_KEY_SEED` | switchboard.env | Base64 seed for WireGuard tokens (`dd bs=32 count=1 if=/dev/urandom | base64`) |

## Public / frontend HTTP API

### POST /generate
Create a token. Form fields:

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | Token type string (see table below) |
| `email` | one of email/webhook | Alert email address |
| `webhook_url` | one of email/webhook | Webhook (Slack/Teams/generic) |
| `memo` | yes | Free-text reminder of where the token is planted |

Response (JSON) includes: `token`, `auth`, `hostname`, `url`, `url_components`; for `aws_keys` it adds `access_key_id` and `secret_access_key`.

### GET /download
Download the artifact for document/credential tokens.

| Param | Description |
|-------|-------------|
| `fmt` | Output format, e.g. `msword`, `aws_keys`, `adobe_pdf` |
| `token` | Token id from /generate |
| `auth` | Auth value from /generate |

### GET /history
View triggers for a token (params `token`, `auth`).

## Token type strings

| `type` | Token | Trigger |
|--------|-------|---------|
| `http` | Web-bug URL | HTTP GET on the URL |
| `dns` | DNS name | DNS resolution of the hostname |
| `aws_keys` | AWS API key | Use of the key against AWS |
| `msword` | MS Word doc | Document opened |
| `adobe_pdf` | PDF doc | Document opened |
| `slack_api` | Slack API token | Use against Slack API |
| `kubeconfig` | Kubernetes config | Use against kube API |
| `azure_id` | Azure login cert | Azure authentication |
| `qr_code` | QR code | Encoded URL requested |
| `web_image` | Image web-bug | Image loaded |
| `log4shell` | Log4j JNDI string | Vulnerable logger evaluates string |
| `cmd` | Sensitive command (Windows) | Command/process executed |
| `cloned_web` | Cloned website | JS detects clone load |
| `sql_server` | SQL Server | DB connection/trigger |

## Active Directory honey credential (PowerShell)

| Action | Command |
|--------|---------|
| Create decoy user | `New-ADUser -Name svc_backup_legacy -AccountPassword (ConvertTo-SecureString 'C0mpl3xDecoy!2026' -AsPlainText -Force) -Enabled $true` |
| Add SPN (Kerberoast bait) | `Set-ADUser svc_backup_legacy -ServicePrincipalNames @{Add="MSSQLSvc/decoy.example.com:1433"}` |
| Alerting | SIEM rule on Event ID 4768/4769/4625 for the decoy SAM account |

===== END FILE =====


===== FILE: skills/deploying-honeytokens-and-canarytokens/references/standards.md =====
# Standards and Framework Mapping

## NIST Cybersecurity Framework 2.0

| ID | Name | Rationale |
|----|------|-----------|
| DE.CM-01 | Networks and network services are monitored to find potentially adverse events | Honeytokens and canarytokens are monitored decoy assets; their interaction events are the adverse-event signal this control requires. |

## MITRE ATT&CK

| ID | Name | Rationale |
|----|------|-----------|
| T1556 | Modify Authentication Process | A triggered honey credential is high-fidelity evidence of adversary attempts to harvest, replay, or otherwise abuse authentication material; the decoy provides the detection tripwire. |

## MITRE D3FEND (defensive counter-mapping)

| ID | Name | Rationale |
|----|------|-----------|
| D3-DF | Decoy File | Canary documents (Word/PDF), fake configs, and decoy archives are decoy files placed in monitored locations. |
| D3-DUC | Decoy User Credential | Honey credentials (AWS keys, AD service accounts, kubeconfig, Slack tokens) are decoy credentials integrated with a monitored decoy asset. |
| D3-DO | Decoy Object | Umbrella technique covering honeytokens/canarytokens as monitored decoy artifacts. |

## OWASP / Industry References

- Thinkst Canarytokens — open-source canarytoken engine and hosted service (https://canarytokens.org).
- Picus Security: "Defending Against Credential Access Attacks: Harnessing MITRE D3FEND Decoy Objects."
- The Canarytokens approach aligns with the deception layer recommended in zero-trust and defense-in-depth architectures.

===== END FILE =====


===== FILE: skills/deploying-honeytokens-and-canarytokens/scripts/agent.py =====
#!/usr/bin/env python3
"""
honeytoken_agent.py — Generate, validate, and inventory Canarytokens.

Talks to a Canarytokens frontend (the public service at https://canarytokens.org
or a self-hosted thinkst/canarytokens-docker instance) via its POST /generate
and GET /history HTTP API. Maintains a local JSON inventory of every token
deployed, with its memo, planted location, and MITRE D3FEND mapping, so a blue
team can track deception coverage.

This is a defensive tool. Only generate tokens for assets you own or are
authorized to instrument, and never commit decoy artifacts to public repos.

Examples:
  # Generate an HTTP web-bug token on the public service
  python3 honeytoken_agent.py generate --type http \
      --email soc@example.com --memo "wiki admin-passwords page" \
      --location "https://wiki.internal/it/admin" --d3fend D3-DF

  # Generate against a self-hosted frontend with a webhook
  python3 honeytoken_agent.py generate --base-url https://canary.example.com \
      --type aws_keys --webhook https://hooks.slack.com/services/T/B/X \
      --memo "decoy keys jenkins host" --location "/root/.aws/credentials"

  # Show triggers (history) for a stored token
  python3 honeytoken_agent.py history --token-id <token> --auth <auth>

  # List the local inventory
  python3 honeytoken_agent.py inventory
"""
import argparse
import json
import os
import sys
from datetime import datetime, timezone

try:
    import requests
except ImportError:
    sys.stderr.write("ERROR: install dependency with: python3 -m pip install requests\n")
    sys.exit(2)

DEFAULT_BASE = "https://canarytokens.org"
INVENTORY = os.environ.get("CANARY_INVENTORY", "canarytoken_inventory.json")

VALID_TYPES = {
    "http", "dns", "aws_keys", "msword", "adobe_pdf", "slack_api",
    "kubeconfig", "azure_id", "qr_code", "web_image", "log4shell",
    "cmd", "cloned_web", "sql_server",
}


def _load_inventory():
    if not os.path.exists(INVENTORY):
        return []
    try:
        with open(INVENTORY, "r", encoding="utf-8") as fh:
            return json.load(fh)
    except (json.JSONDecodeError, OSError) as exc:
        sys.stderr.write(f"WARN: could not read inventory {INVENTORY}: {exc}\n")
        return []


def _save_inventory(items):
    try:
        with open(INVENTORY, "w", encoding="utf-8") as fh:
            json.dump(items, fh, indent=2)
    except OSError as exc:
        sys.stderr.write(f"ERROR: could not write inventory {INVENTORY}: {exc}\n")
        sys.exit(1)


def generate(args):
    if args.type not in VALID_TYPES:
        sys.stderr.write(f"ERROR: unknown type '{args.type}'. Valid: {sorted(VALID_TYPES)}\n")
        sys.exit(1)
    if not args.email and not args.webhook:
        sys.stderr.write("ERROR: provide --email and/or --webhook for alerting.\n")
        sys.exit(1)

    data = {"type": args.type, "memo": args.memo}
    if args.email:
        data["email"] = args.email
    if args.webhook:
        data["webhook_url"] = args.webhook

    url = args.base_url.rstrip("/") + "/generate"
    try:
        resp = requests.post(url, data=data, timeout=args.timeout)
        resp.raise_for_status()
    except requests.RequestException as exc:
        sys.stderr.write(f"ERROR: generate request failed: {exc}\n")
        sys.exit(1)

    try:
        body = resp.json()
    except ValueError:
        sys.stderr.write("ERROR: non-JSON response from server:\n" + resp.text[:500] + "\n")
        sys.exit(1)

    token_id = body.get("token") or body.get("canarytoken")
    record = {
        "type": args.type,
        "memo": args.memo,
        "location": args.location or "",
        "d3fend": args.d3fend or "",
        "token": token_id,
        "auth": body.get("auth"),
        "hostname": body.get("hostname"),
        "url": body.get("url"),
        "access_key_id": body.get("access_key_id"),
        "created": datetime.now(timezone.utc).isoformat(),
        "base_url": args.base_url.rstrip("/"),
    }
    inv = _load_inventory()
    inv.append(record)
    _save_inventory(inv)

    print(json.dumps({k: v for k, v in record.items() if v is not None}, indent=2))
    if args.type in ("msword", "adobe_pdf", "aws_keys") and token_id and record["auth"]:
        dl = (f"{record['base_url']}/download?fmt={args.type}"
              f"&token={token_id}&auth={record['auth']}")
        print(f"\nDownload artifact:\n  curl -s '{dl}' -o token_artifact")
    return 0


def history(args):
    url = args.base_url.rstrip("/") + "/history"
    try:
        resp = requests.get(url, params={"token": args.token_id, "auth": args.auth},
                            timeout=args.timeout)
        resp.raise_for_status()
    except requests.RequestException as exc:
        sys.stderr.write(f"ERROR: history request failed: {exc}\n")
        sys.exit(1)
    try:
        print(json.dumps(resp.json(), indent=2))
    except ValueError:
        print(resp.text)
    return 0


def inventory(_args):
    inv = _load_inventory()
    if not inv:
        print("(inventory empty)")
        return 0
    print(f"{'TYPE':<12} {'D3FEND':<8} {'MEMO':<40} LOCATION")
    print("-" * 90)
    for rec in inv:
        print(f"{rec.get('type',''):<12} {rec.get('d3fend',''):<8} "
              f"{(rec.get('memo','') or '')[:40]:<40} {rec.get('location','')}")
    print(f"\nTotal tokens deployed: {len(inv)}")
    return 0


def build_parser():
    p = argparse.ArgumentParser(description="Canarytoken generation, validation and inventory helper.")
    p.add_argument("--base-url", default=DEFAULT_BASE,
                   help=f"Canarytokens frontend base URL (default {DEFAULT_BASE})")
    p.add_argument("--timeout", type=int, default=20, help="HTTP timeout seconds")
    sub = p.add_subparsers(dest="cmd", required=True)

    g = sub.add_parser("generate", help="Create a new canarytoken")
    g.add_argument("--type", required=True, help="Token type (e.g. http, dns, aws_keys, msword)")
    g.add_argument("--email", help="Alert email address")
    g.add_argument("--webhook", help="Alert webhook URL")
    g.add_argument("--memo", required=True, help="Reminder of where the token is planted")
    g.add_argument("--location", help="Where the token will be planted (for inventory)")
    g.add_argument("--d3fend", help="MITRE D3FEND mapping, e.g. D3-DF or D3-DUC")
    g.set_defaults(func=generate)

    h = sub.add_parser("history", help="Show triggers for a token")
    h.add_argument("--token-id", required=True)
    h.add_argument("--auth", required=True)
    h.set_defaults(func=history)

    i = sub.add_parser("inventory", help="List the local token inventory")
    i.set_defaults(func=inventory)
    return p


def main():
    args = build_parser().parse_args()
    return args.func(args)


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/designing-adversary-engagement-with-mitre-engage/SKILL.md =====
---
name: designing-adversary-engagement-with-mitre-engage
description: >-
  Plan, run, and measure an adversary engagement operation using the MITRE Engage
  framework so that deployed deception is driven by strategy instead of deployed ad hoc.
  Covers the Engage Matrix (Prepare, Expose, Affect, Elicit, Understand), the 10-Step
  Operational Process, mapping engagement Activities to the ATT&CK techniques they
  expose, and defining measurable Goals and Operational Objectives. Use when a team has
  honeypots, honeytokens, or canary tokens but no coordinating strategy, when leadership
  asks "should we engage attackers and how", when building a deception/denial program,
  when writing an adversary engagement operation plan, or when deciding which deception
  Activities to deploy against a specific threat actor. Keywords: MITRE Engage, adversary
  engagement, cyber deception strategy, denial and deception, Engage Matrix, EAC, EGO,
  Expose Affect Elicit, deception program, honeypot strategy, engagement operation.
domain: cybersecurity
subdomain: deception-technology
tags:
- mitre-engage
- adversary-engagement
- deception
- denial-and-deception
- engage-matrix
- cyber-deception
- threat-intelligence
- detection-engineering
version: "1.0"
author: andrewibrah
license: Apache-2.0
nist_csf:
- GV.RM-01
- ID.RA-01
- ID.IM-02
- DE.CM-01
- DE.AE-02
mitre_attack:
- T1078
- T1083
- T1021
- T1552
- T1046
---

# Designing Adversary Engagement with MITRE Engage

## When to Use

- When an organization owns deception tooling (honeypots, honeytokens, canary tokens, decoy files) but deploys it tactically with no unifying strategy or measurable outcome.
- When leadership asks whether the organization *should* engage adversaries, and what the legal, operational, and resourcing implications are.
- When writing a formal adversary engagement operation plan that must justify every deployed deceptive artifact against a strategic goal.
- When selecting which specific deception Activities to deploy against a known or suspected threat actor based on that actor's ATT&CK TTPs.
- When building a denial, deception, and adversary engagement (DD&AE) program that must integrate with existing SOC, threat intel, and incident response functions.
- When a deception deployment generates alerts that nobody knows how to act on, because Expose was never connected to Affect or Elicit goals.

This skill is the **strategy and operations layer** that sits above tactical deployment skills (honeypot, honeytoken, canary-token, and decoy-file deployment). Use those skills to *implement* the Activities this skill selects and sequences.

## Prerequisites

- Familiarity with MITRE ATT&CK (tactics, techniques, and how to read a technique page), because Engagement Activities are mapped to the ATT&CK techniques they expose.
- A documented set of critical assets and an understanding of which adversaries plausibly target them (a threat model or prioritized threat actor list).
- Executive sponsorship and a written legal review. Engagement operations interact with live adversaries and raise entrapment, evidence-handling, and liability questions; **never run an engagement operation without legal sign-off.**
- An existing detection and response capability. Engage is an additive strategy, not a replacement for defense-in-depth; if a defense-in-depth control fails, engagement keeps you in control rather than blind.
- Access to the live matrix at https://engage.mitre.org/matrix/ for canonical Activity names and IDs.

## Workflow

Engage operations follow the **10-Step Operational Process**. The matrix is linear to read but cyclical to run — you continuously realign Activities toward your Goals as the adversary reacts.

### 1. Confirm strategic fit (Prepare)
Decide where denial, deception, and adversary engagement fit in the existing cyber strategy. The `Prepare` goal (a strategic bookend, alongside `Understand`) defines the inputs to the operation. Document the strategic goal in plain language, e.g. "reduce dwell time of insider threats around the source-code repository" or "generate first-party CTI on the actor targeting our VPN."

### 2. Define Engagement Goals and Operational Objectives
Select from the three Engagement Goals. Goals set direction; **Operational Objectives** take measurable steps in that direction.

| Engagement Goal (EGO) | What it does | Example Operational Objective |
|---|---|---|
| Expose | Reveal adversary presence with high-fidelity, low-false-positive alerts | "Alert within 5 minutes of any touch on a decoy credential" |
| Affect | Negatively change the adversary's cost-value calculation (defender network only) | "Redirect the adversary away from 3 unpatchable legacy hosts" |
| Elicit | Observe the adversary to learn TTPs and produce CTI | "Obtain a second-stage malware sample" or "identify ≥10 new indicators" |

Write objectives as falsifiable, time-bound statements. A goal without an objective is unmeasurable.

### 3. Build the threat model and select Approaches
For each Goal, pick the Engagement Approaches (EAP) that fit the adversary you modeled:

- **Expose** → Collection, Detection
- **Affect** → Prevention, Direction, Disruption
- **Elicit** → Reassurance, Motivation

### 4. Map ATT&CK techniques to Engagement Activities
For each technique your target adversary uses, find the Engage Activity that exposes the weakness that technique creates. Example mappings:

| Adversary technique (ATT&CK) | Weakness exposed | Engage Activity (EAC) |
|---|---|---|
| T1078 Valid Accounts | Must test credentials | Decoy Credentials, Lures |
| T1083 File & Directory Discovery | Must enumerate files | Decoy Content, Pocket Litter |
| T1046 Network Service Discovery | Must scan the network | Network Diversity, Decoy Systems |
| T1021 Remote Services | Must move laterally | Decoy Systems, Network Manipulation |
| T1552 Unsecured Credentials | Harvests secrets | Decoy Credentials, Artifact Diversity |

Pull the authoritative Activity list and IDs from the live matrix; Engage IDs use the prefixes **SGO/EGO** (Goals), **SAP/EAP** (Approaches), and **SAC/EAC** (Activities).

### 5. Design the engagement environment
Decide realism and isolation. Choose between standalone, connected, or integrated decoy environments (see D3FEND honeynet types in `references/standards.md`). Populate it with diverse, believable artifacts — Persona Creation, Pocket Litter, Artifact Diversity, Application Diversity — so the environment survives adversary scrutiny.

### 6. Define gating criteria and rules of engagement
Document, before deployment: what the adversary is allowed to reach, the maximum blast radius, the trigger for tear-down or hand-off to IR, evidence preservation steps, and who has authority to escalate. **Affect Activities are limited to the defender's own network** — never act on infrastructure you do not own.

### 7. Deploy the Activities
Implement the selected Activities using the tactical deployment skills (honeypots, honeytokens, canary tokens, decoy files). Instrument every artifact so a touch produces telemetry routed to the SOC.

### 8. Operate and observe
Run the operation. Triage Expose alerts as high-fidelity (a touch on a decoy almost always means malicious or unauthorized activity). Feed observations back into Approach selection — realign Affect/Elicit Activities as the adversary behaves.

### 9. Analyze (Understand)
The `Understand` goal (the output bookend) turns observations into decisions: new detections for production, CTI for sharing, and validated or invalidated threat-model assumptions.

### 10. After-action and feedback
Score the operation against the Operational Objectives from Step 2. Capture what intel was gained, what Activities triggered, dwell time, and lessons learned. Update the threat model and feed the next cycle.

## Key Concepts

| Concept | Definition |
|---|---|
| Goal (SGO/EGO) | High-level outcome of the operation. Prepare/Understand are strategic bookends; Expose/Affect/Elicit are the engagement goals. |
| Approach (SAP/EAP) | The method used to make progress toward a Goal (e.g., Detection, Direction, Motivation). |
| Activity (SAC/EAC) | The concrete denial/deception action deployed (e.g., Decoy Credentials, Network Manipulation). |
| Operate | The default matrix view = Expose + Affect + Elicit, the three engagement goals. |
| Operational Objective | A measurable, time-bound target that operationalizes a Goal. |
| Gating Criteria | Pre-defined boundaries and triggers that constrain the operation's blast radius. |
| High-fidelity alert | An alert from a decoy that legitimate users have no reason to touch, yielding near-zero false positives. |
| Denial vs. Deception | Denial blocks the adversary's access to real information; deception feeds plausible false information. |

## Tools & Systems

- **MITRE Engage Matrix and Starter Kit** (https://engage.mitre.org) — canonical Goals/Approaches/Activities, the 10-Step Process, and operation-planning worksheets.
- **MITRE ATT&CK Navigator** — to lay out the target adversary's techniques and overlay selected Engagement Activities.
- **MITRE D3FEND** — the `Deceive` tactic provides defensive countermeasure naming (Decoy Environment, Decoy Object, honeynet types) that complements Engage.
- **Deception platforms / open tooling** — OpenCanary, T-Pot, Cowrie (honeypots); Canarytokens, Thinkst Canary (honeytokens); to *implement* selected Activities.
- **SIEM/SOAR** — to route decoy telemetry to high-priority detections and automate Expose → IR hand-off.
- **CTI platform (MISP, OpenCTI)** — to store and share the first-party intelligence produced under the Elicit goal.

## Common Scenarios

- **"We have honeypots but no value."** Map existing honeypots to the Expose goal, define an Operational Objective (alert latency, dwell-time reduction), and connect alerts to an IR hand-off so the deployment produces decisions, not noise.
- **"Targeted by a specific actor."** Build the actor's ATT&CK technique set, map each to the Activity that exposes it, and prioritize the smallest set of Activities that covers the actor's likely kill chain.
- **"Protect unpatchable legacy systems."** Use Affect Activities (Direction, Network Manipulation, decoys) to steer adversaries away from systems that cannot be remediated.
- **"Tired of CVE whack-a-mole."** Use the Elicit goal to generate a first-party CTI feed so defense is driven by observed adversary TTPs rather than the vulnerability of the week.
- **"Insider threat near critical data."** Seed Expose Activities (Decoy Content, Decoy Credentials, Pocket Litter) around the crown-jewel asset for high-fidelity detection of unauthorized internal access.

## Output Format

Produce an **Adversary Engagement Operation Plan** using `assets/template.md`, containing:

1. **Strategic context** — where DD&AE fits the cyber strategy; executive sponsor; legal sign-off reference.
2. **Engagement Goals + Operational Objectives** — each objective falsifiable and time-bound.
3. **Threat model** — target adversary, prioritized ATT&CK techniques.
4. **Activity selection matrix** — technique → exposed weakness → selected Engage Activity (with EAC IDs) → tactical deployment owner.
5. **Engagement environment design** — realism, isolation/honeynet type, artifact diversity plan.
6. **Gating criteria and rules of engagement** — blast radius, tear-down triggers, evidence handling, escalation authority.
7. **Measurement plan** — metrics per objective (alert latency, dwell time, indicators gained, samples obtained).
8. **After-action report** — objectives met/missed, intel produced, detections promoted to production, threat-model updates.

Use `scripts/process.py` to validate technique→Activity coverage and generate the operation-plan skeleton from a threat-model input.

===== END FILE =====


===== FILE: skills/designing-adversary-engagement-with-mitre-engage/LICENSE =====

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


===== FILE: skills/designing-adversary-engagement-with-mitre-engage/assets/template.md =====
# Adversary Engagement Operation Plan

> Worked example. Replace bracketed values. Do not deploy any Activity before legal sign-off and approved gating criteria.

## 1. Strategic context
- **Operation name:** Crown-Jewel Repo Watch
- **Strategic goal (Prepare):** Reduce dwell time of unauthorized access around the source-code repository and produce first-party CTI on whoever reaches it.
- **Where DD&AE fits the strategy:** Additive layer behind EDR + network segmentation; activates only if a primary control is bypassed.
- **Executive sponsor:** [CISO name]
- **Legal sign-off reference:** [Legal ticket / memo ID] — REQUIRED before deployment

## 2. Engagement Goals + Operational Objectives
| Goal (EGO) | Operational Objective (falsifiable, time-bound) |
|---|---|
| Expose | Alert the SOC within 5 minutes of any touch on a decoy repo credential or decoy commit. |
| Affect | Redirect lateral-movement attempts away from 2 unpatchable build servers for the duration of the operation. |
| Elicit | Obtain ≥10 new indicators and, if possible, one second-stage tool sample within 30 days. |

## 3. Threat model
- **Target adversary:** Suspected initial-access broker reselling dev-environment footholds.
- **Prioritized ATT&CK techniques:** T1078 (Valid Accounts), T1552 (Unsecured Credentials), T1083 (File & Directory Discovery), T1021 (Remote Services), T1046 (Network Service Discovery).

## 4. Activity selection matrix
| ATT&CK | Weakness exposed | Engage Activity (resolve EAC on live matrix) | Deployment owner | Tactical skill |
|---|---|---|---|---|
| T1078 | Must test credentials | Decoy Credentials, Lures | [Detection eng.] | deploying-active-directory-honeytokens |
| T1552 | Harvests secrets | Decoy Credentials, Artifact Diversity | [Detection eng.] | implementing-honeytokens-for-breach-detection |
| T1083 | Enumerates files | Decoy Content, Pocket Litter | [Blue team] | deploying-decoy-files-for-ransomware-detection |
| T1021 | Moves laterally | Network Manipulation, Decoy Systems | [Network eng.] | implementing-network-deception-with-honeypots |
| T1046 | Scans the network | Network Diversity | [Network eng.] | implementing-network-deception-with-honeypots |

## 5. Engagement environment design
- **Honeynet type:** Connected honeynet (reachable from the dev VLAN, isolated from prod data).
- **Realism / artifact plan:** Decoy repo with believable Pocket Litter (fake CI tokens, stale branches), Persona Creation for a fake "build-bot" account, Application Diversity to mimic the real toolchain.

## 6. Gating criteria & rules of engagement
- **Max blast radius:** Decoy VLAN only; no route to production data stores.
- **Tear-down / IR hand-off trigger:** Any attempt to pivot toward a real prod subnet, OR collection of the second-stage sample, whichever first.
- **Evidence handling:** Full pcap + host telemetry preserved to WORM storage; chain-of-custody log maintained.
- **Escalation authority:** [IR lead] may halt the operation at any time.
- **Affect Activities restricted to defender-owned network.** (Hard constraint — never act on infrastructure you do not own.)

## 7. Measurement plan
| Objective | Metric | Baseline | Result |
|---|---|---|---|
| Expose latency | Minutes from decoy touch to SOC alert | n/a (new) | [fill post-op] |
| Affect redirect | Lateral attempts steered from build servers | 0 | [fill post-op] |
| Elicit intel | New indicators / samples obtained | 0 | [fill post-op] |

## 8. After-action report (complete post-operation)
- **Objectives met/missed:** [ ]
- **Intel produced (indicators, samples, TTPs):** [ ]
- **Detections promoted to production:** [ ]
- **Threat-model updates for next cycle:** [ ]

===== END FILE =====


===== FILE: skills/designing-adversary-engagement-with-mitre-engage/references/standards.md =====
# MITRE Engage — Standards & Framework Reference

## Primary framework
### MITRE Engage™ v1.0
- **Publisher**: The MITRE Corporation
- **Version**: 1.0, last updated 2022-02-28
- **Home**: https://engage.mitre.org
- **Live Matrix**: https://engage.mitre.org/matrix/ (authoritative source for all Goal/Approach/Activity names and IDs)
- **Starter Kit**: https://engage.mitre.org/starter-kit/ (10-Step Process, planning worksheets, whitepapers)
- **Predecessor**: MITRE Shield (Engage supersedes and restructures Shield).
- **Note**: Engage is a framework for *planning and discussing* denial, deception, and adversary engagement. It is not a tool; it provides a shared language across defenders, vendors, and decision-makers.

## Engage Matrix structure
Five columns (Goals): **Prepare · Expose · Affect · Elicit · Understand**
- **Prepare** and **Understand** are *strategic* bookends (operation inputs and outputs).
- **Expose**, **Affect**, **Elicit** are the three *Engagement* goals; together they form the default **Operate** view and are mapped to MITRE ATT&CK.

### ID prefixes (verified from engage.mitre.org)
| Component | Strategic prefix | Engagement prefix |
|---|---|---|
| Goals | SGO | EGO |
| Approaches | SAP | EAP |
| Activities | SAC | EAC |

Always resolve specific numeric IDs (e.g., the EAC for "Decoy Credentials") against the live matrix rather than from memory.

### Engagement Approaches (EAP) by Goal
- **Expose** → Collection, Detection
- **Affect** → Prevention, Direction, Disruption
- **Elicit** → Reassurance, Motivation

### Representative Engagement Activities (EAC), by name
Decoy Credentials · Decoy Content · Decoy Account · Decoy Diversity · Lures · Pocket Litter ·
Persona Creation · Artifact Diversity · Network Diversity · Application Diversity ·
Email Manipulation · Network Manipulation · Software Manipulation · Hardware Manipulation ·
Security Controls · Isolation · Attack Vector Migration · Peripheral Management · Baseline ·
Network Monitoring · System Activity Monitoring · API Monitoring · Malware Detonation ·
Burn-In · Introduced Vulnerabilities.

> The matrix maps each Activity to the ATT&CK techniques whose execution exposes an adversary weakness. Use the Navigator overlay to confirm current mappings.

## Operating principle: Affect is defender-network-only
All Affect Activities are constrained to infrastructure the defender owns and controls. Acting on adversary or third-party infrastructure is out of scope and creates legal exposure.

## Complementary frameworks

### MITRE ATT&CK
- https://attack.mitre.org — the technique catalog used to model the target adversary. Engagement Activities exist to exploit the weaknesses adversary techniques create.

### MITRE D3FEND — `Deceive` tactic
D3FEND (https://d3fend.mitre.org) provides defensive-technique naming that pairs with Engage. The `Deceive` tactic includes:
- **Decoy Environment**: Connected Honeynet, Integrated Honeynet, Standalone Honeynet
- **Decoy Object**: Decoy File, Decoy Network Resource, Decoy Persona, Decoy Public Release, Decoy Session Token, Decoy User Credential

Use D3FEND honeynet types when documenting environment isolation in the operation plan:
- **Standalone Honeynet** — fully isolated; safest; least realistic to a sophisticated adversary.
- **Connected Honeynet** — bridged to production paths to appear reachable; moderate risk.
- **Integrated Honeynet** — decoys interleaved with production assets; most realistic; highest operational risk and tightest gating required.

## NIST CSF 2.0 alignment
| CSF 2.0 ID | Relevance to adversary engagement |
|---|---|
| GV.RM-01 | Risk management objectives established — anchors the strategic Prepare goal |
| ID.RA-01 | Vulnerabilities identified — informs which weaknesses to expose |
| ID.IM-02 | Security testing / improvement — engagement operations validate detections |
| DE.CM-01 | Networks monitored to find adverse events — Expose Activities feed monitoring |
| DE.AE-02 | Potentially adverse events analyzed — triage of decoy alerts |

## Legal & ethical references
- Engagement operations interact with live adversaries; obtain written legal review before deployment.
- Preserve evidence per the organization's incident-response and forensics procedures (chain of custody).
- Coordinate with law enforcement engagement policy where applicable.
- Document rules of engagement and gating criteria before any Activity is deployed.

===== END FILE =====


===== FILE: skills/designing-adversary-engagement-with-mitre-engage/scripts/process.py =====
#!/usr/bin/env python3
"""
MITRE Engage operation planner.

Given a threat model (a list of ATT&CK technique IDs the target adversary uses),
this maps each technique to the Engage Activities that expose its weakness,
reports coverage gaps, and emits an Adversary Engagement Operation Plan skeleton.

The embedded ATT&CK -> Engage Activity table is a *starter* map. Reconcile against
the live matrix at https://engage.mitre.org/matrix/ before operational use.

Usage:
    python process.py --threat-model tm.json --out plan.md
    python process.py --list-techniques

tm.json format:
    {
      "operation_name": "VPN-actor engagement",
      "target_actor": "Suspected access broker",
      "strategic_goal": "Generate first-party CTI on the VPN actor",
      "techniques": ["T1078", "T1046", "T1021", "T1552"]
    }
"""
import argparse
import json
import sys
from datetime import date

# Starter map: ATT&CK technique -> (short name, [candidate Engage Activities]).
# Activities use canonical Engage names; resolve EAC numeric IDs from the live matrix.
TECHNIQUE_TO_ACTIVITIES = {
    "T1078": ("Valid Accounts", ["Decoy Credentials", "Lures", "Decoy Account"]),
    "T1110": ("Brute Force", ["Decoy Credentials", "Security Controls"]),
    "T1552": ("Unsecured Credentials", ["Decoy Credentials", "Artifact Diversity", "Pocket Litter"]),
    "T1083": ("File and Directory Discovery", ["Decoy Content", "Pocket Litter", "Artifact Diversity"]),
    "T1046": ("Network Service Discovery", ["Network Diversity", "Network Manipulation"]),
    "T1021": ("Remote Services", ["Network Manipulation", "Isolation", "Decoy Content"]),
    "T1018": ("Remote System Discovery", ["Network Diversity", "Decoy Content"]),
    "T1057": ("Process Discovery", ["System Activity Monitoring", "Software Manipulation"]),
    "T1071": ("Application Layer Protocol (C2)", ["Network Monitoring", "Network Manipulation"]),
    "T1105": ("Ingress Tool Transfer", ["Malware Detonation", "Network Monitoring"]),
    "T1190": ("Exploit Public-Facing Application", ["Introduced Vulnerabilities", "Application Diversity"]),
    "T1059": ("Command and Scripting Interpreter", ["System Activity Monitoring", "API Monitoring"]),
    "T1567": ("Exfiltration Over Web Service", ["Network Monitoring", "Decoy Content"]),
}

ENGAGEMENT_GOALS = ["Expose", "Affect", "Elicit"]


def build_coverage(techniques):
    covered, gaps = {}, []
    for t in techniques:
        t = t.strip().upper()
        if t in TECHNIQUE_TO_ACTIVITIES:
            name, acts = TECHNIQUE_TO_ACTIVITIES[t]
            covered[t] = {"name": name, "activities": acts}
        else:
            gaps.append(t)
    return covered, gaps


def render_plan(tm, covered, gaps):
    lines = []
    lines.append(f"# Adversary Engagement Operation Plan: {tm.get('operation_name', 'UNNAMED')}")
    lines.append(f"\n_Generated {date.today().isoformat()} — DRAFT, requires legal sign-off before deployment._\n")
    lines.append("## 1. Strategic context")
    lines.append(f"- **Target actor:** {tm.get('target_actor', 'TBD')}")
    lines.append(f"- **Strategic goal (Prepare):** {tm.get('strategic_goal', 'TBD')}")
    lines.append("- **Executive sponsor:** TBD")
    lines.append("- **Legal sign-off reference:** TBD (REQUIRED before deployment)\n")

    lines.append("## 2. Engagement Goals + Operational Objectives")
    for g in ENGAGEMENT_GOALS:
        lines.append(f"- **{g}:** define ≥1 falsifiable, time-bound objective (e.g., alert latency, dwell-time reduction, indicators gained).")
    lines.append("")

    lines.append("## 3. Activity selection matrix")
    lines.append("| ATT&CK | Technique | Weakness → Engage Activities | Deployment owner |")
    lines.append("|---|---|---|---|")
    for t, info in covered.items():
        acts = ", ".join(info["activities"])
        lines.append(f"| {t} | {info['name']} | {acts} | TBD |")
    if not covered:
        lines.append("| — | — | (no mapped techniques) | — |")
    lines.append("")

    if gaps:
        lines.append("## 3a. Coverage GAPS (no starter mapping — check live matrix)")
        for t in gaps:
            lines.append(f"- {t}: resolve against https://engage.mitre.org/matrix/")
        lines.append("")

    lines.append("## 4. Engagement environment design")
    lines.append("- Honeynet type (standalone / connected / integrated): TBD")
    lines.append("- Artifact diversity plan (Persona Creation, Pocket Litter, Artifact/Application/Network Diversity): TBD\n")

    lines.append("## 5. Gating criteria & rules of engagement")
    lines.append("- Max blast radius: TBD")
    lines.append("- Tear-down / IR hand-off trigger: TBD")
    lines.append("- Evidence handling & chain of custody: TBD")
    lines.append("- Escalation authority: TBD")
    lines.append("- Affect Activities limited to defender-owned network only (hard constraint).\n")

    lines.append("## 6. Measurement plan")
    lines.append("- Metric per objective; baseline before deployment; review cadence: TBD\n")

    lines.append("## 7. After-action report (post-operation)")
    lines.append("- Objectives met/missed, intel produced, detections promoted to production, threat-model updates: TBD")
    return "\n".join(lines)


def main():
    p = argparse.ArgumentParser(description="MITRE Engage operation planner")
    p.add_argument("--threat-model", help="Path to threat-model JSON")
    p.add_argument("--out", help="Output markdown path (default: stdout)")
    p.add_argument("--list-techniques", action="store_true", help="List techniques in the starter map")
    args = p.parse_args()

    if args.list_techniques:
        for t, (name, acts) in sorted(TECHNIQUE_TO_ACTIVITIES.items()):
            print(f"{t:8} {name:40} -> {', '.join(acts)}")
        return

    if not args.threat_model:
        p.error("--threat-model is required (or use --list-techniques)")

    with open(args.threat_model) as f:
        tm = json.load(f)

    techniques = tm.get("techniques", [])
    if not techniques:
        print("WARNING: threat model has no 'techniques' list", file=sys.stderr)

    covered, gaps = build_coverage(techniques)
    total = len(techniques)
    pct = (len(covered) / total * 100) if total else 0
    print(f"Coverage: {len(covered)}/{total} techniques mapped ({pct:.0f}%); {len(gaps)} gap(s).",
          file=sys.stderr)

    plan = render_plan(tm, covered, gaps)
    if args.out:
        with open(args.out, "w") as f:
            f.write(plan + "\n")
        print(f"Wrote operation plan -> {args.out}", file=sys.stderr)
    else:
        print(plan)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-deception-based-detection-with-canarytoken/SKILL.md =====
---
name: implementing-deception-based-detection-with-canarytoken
description: Deploys and monitors Canary Tokens via the Thinkst Canary REST API for
  deception-based breach detection, programmatically creating web bug, DNS, MS Word
  document, and AWS API key tokens and generating deception coverage reports from
  triggered alerts. Use when standing up honeytoken tripwires for early breach detection
  or building a deception-technology coverage report.
domain: cybersecurity
subdomain: deception-technology
tags:
- canarytoken
- deception
- honeytokens
- breach-detection
- Thinkst-Canary
- tripwire
- early-warning
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- DE.AE-06
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1078.004
- T1530
---

# Implementing Deception-Based Detection with Canarytoken

## Overview

Canary Tokens are lightweight tripwire mechanisms that alert when an attacker accesses a resource. This skill uses the Thinkst Canary REST API to programmatically create tokens (web bugs, DNS tokens, MS Word documents, AWS API keys), deploy them to strategic locations, monitor for triggered alerts, and generate deception coverage reports.


## When to Use

- When deploying or configuring implementing deception based detection with canarytoken capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Thinkst Canary Console or canarytokens.org account
- API auth token from Canary Console
- Python 3.9+ with `requests`
- File system access for deploying document and file tokens

## Steps

1. Authenticate to the Canary Console API using auth_token
2. Create web bug (HTTP) tokens for embedding in documents and web pages
3. Create DNS tokens for monitoring DNS resolution attempts
4. Create MS Word document tokens for file share deployment
5. List all active tokens and their trigger history
6. Query recent alerts for triggered token events
7. Generate deception coverage report with deployment recommendations

## Expected Output

- JSON report listing all deployed Canary Tokens, trigger history, alert details, and coverage analysis
- Deployment map showing token types across network segments

===== END FILE =====


===== FILE: skills/implementing-deception-based-detection-with-canarytoken/LICENSE =====

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


===== FILE: skills/implementing-deception-based-detection-with-canarytoken/references/api-reference.md =====
# API Reference — Implementing Deception-Based Detection with Canarytoken

## Libraries Used
- **requests**: HTTP client for Thinkst Canary Console REST API
- **json**: JSON serialization for audit reports

## CLI Interface
```
python agent.py --console abc123 --auth-token TOKEN ping
python agent.py --console abc123 --auth-token TOKEN list
python agent.py --console abc123 --auth-token TOKEN alerts
python agent.py --console abc123 --auth-token TOKEN create --kind http --memo "Web server token"
python agent.py --console abc123 --auth-token TOKEN create --kind dns --memo "DNS honeypot"
python agent.py --console abc123 --auth-token TOKEN coverage
python agent.py --console abc123 --auth-token TOKEN full
```

## Core Functions

### `CanaryClient(console_domain, auth_token)` — API client
Base URL: `https://{console_domain}.canary.tools/api/v1`
Auth: `auth_token` parameter on every request.

### `create_token(kind, memo, **kwargs)` — Create Canarytoken
POST `/canarytoken/create` with `kind`, `memo`, `auth_token`.
For doc-msword: uploads file via multipart form with MIME type
`application/vnd.openxmlformats-officedocument.wordprocessingml.document`.

### `list_tokens()` — List all deployed tokens
GET `/canarytokens/fetch`. Returns array of token objects with kind, memo, url, enabled.

### `get_alerts(newer_than)` — Fetch triggered token alerts
GET `/incidents/all`. Optional `newer_than` timestamp filter.
Returns src_host (source IP), description, timestamp, acknowledged status.

### `ack_alert(incident_id)` — Acknowledge an alert
POST `/incident/acknowledge` with incident ID.

### `audit_token_coverage(client)` — Coverage analysis
Calculates: tokens by kind, triggered vs untriggered, missing token types,
coverage score as percentage of TOKEN_KINDS deployed.

### `full_audit(client)` — Comprehensive deception audit

## Canary Console API Endpoints
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/ping` | GET | Test API connectivity |
| `/canarytoken/create` | POST | Create new token |
| `/canarytokens/fetch` | GET | List all tokens |
| `/canarytoken/fetch` | GET | Get specific token |
| `/canarytoken/delete` | POST | Delete a token |
| `/incidents/all` | GET | Fetch all alerts |
| `/canarytoken/incidents` | GET | Alerts for specific token |
| `/incident/acknowledge` | POST | Acknowledge alert |

## Supported Token Types
| Kind | Description |
|------|-------------|
| http | Web bug — triggers on HTTP request |
| dns | DNS token — triggers on DNS resolution |
| doc-msword | MS Word document with embedded beacon |
| pdf-acrobat-reader | PDF with embedded beacon |
| aws-id | Fake AWS API key pair |
| web-image | Image with tracking pixel |
| cloned-web | Cloned website detection |
| qr-code | QR code with tracking URL |
| sensitive-cmd | Triggers on command execution |
| windows-dir | Windows folder open detection |

## Dependencies
- `requests` >= 2.28.0
- Thinkst Canary Console account with API auth token

===== END FILE =====


===== FILE: skills/implementing-deception-based-detection-with-canarytoken/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for deploying and monitoring Canary Tokens via the Thinkst Canary API."""

import json
import argparse
from datetime import datetime

try:
    import requests
except ImportError:
    requests = None

TOKEN_KINDS = {
    "http": "http",
    "dns": "dns",
    "doc-msword": "doc-msword",
    "pdf-acrobat-reader": "pdf-acrobat-reader",
    "web-image": "web-image",
    "cloned-web": "cloned-web",
    "aws-id": "aws-id",
    "qr-code": "qr-code",
    "sql": "sql",
    "svn": "svn",
    "smtp": "smtp",
    "windows-dir": "windows-dir",
    "sensitive-cmd": "sensitive-cmd",
}


class CanaryClient:
    """Client for the Thinkst Canary Console REST API."""

    def __init__(self, console_domain, auth_token):
        self.base_url = f"https://{console_domain}.canary.tools/api/v1"
        self.auth_token = auth_token

    def _get(self, endpoint, params=None):
        params = params or {}
        params["auth_token"] = self.auth_token
        resp = requests.get(f"{self.base_url}{endpoint}", params=params, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def _post(self, endpoint, data=None, files=None):
        data = data or {}
        data["auth_token"] = self.auth_token
        resp = requests.post(f"{self.base_url}{endpoint}", data=data, files=files, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def ping(self):
        """Test API connectivity."""
        return self._get("/ping")

    def create_token(self, kind, memo, **kwargs):
        """Create a new Canarytoken.

        Args:
            kind: Token type (http, dns, doc-msword, aws-id, etc.)
            memo: Description/reminder for the token
            **kwargs: Additional parameters (e.g., cloned_web for cloned-web type)
        """
        data = {"kind": kind, "memo": memo}
        data.update(kwargs)
        files = None
        if kind == "doc-msword" and "doc" in kwargs:
            doc_path = kwargs.pop("doc")
            data.pop("doc", None)
            files = {"doc": (doc_path, open(doc_path, "rb"),
                            "application/vnd.openxmlformats-officedocument.wordprocessingml.document")}
        return self._post("/canarytoken/create", data=data, files=files)

    def list_tokens(self):
        """List all Canarytokens on the console."""
        return self._get("/canarytokens/fetch")

    def get_token(self, canarytoken):
        """Get details for a specific token."""
        return self._get("/canarytoken/fetch", params={"canarytoken": canarytoken})

    def delete_token(self, canarytoken):
        """Delete a Canarytoken."""
        return self._post("/canarytoken/delete", data={"canarytoken": canarytoken})

    def get_alerts(self, newer_than=None):
        """Fetch recent alerts from triggered tokens."""
        params = {}
        if newer_than:
            params["newer_than"] = newer_than
        return self._get("/incidents/all", params=params)

    def get_token_alerts(self, canarytoken):
        """Fetch alerts for a specific token."""
        return self._get("/canarytoken/incidents", params={"canarytoken": canarytoken})

    def ack_alert(self, incident_id):
        """Acknowledge an alert."""
        return self._post("/incident/acknowledge", data={"incident": incident_id})


def create_deployment(client, deployment_plan):
    """Create multiple tokens based on a deployment plan."""
    results = []
    for token_spec in deployment_plan:
        kind = token_spec.get("kind", "http")
        memo = token_spec.get("memo", f"Canarytoken - {kind}")
        extra = {k: v for k, v in token_spec.items() if k not in ("kind", "memo")}
        try:
            resp = client.create_token(kind, memo, **extra)
            results.append({
                "kind": kind,
                "memo": memo,
                "status": "CREATED",
                "canarytoken": resp.get("canarytoken", {}).get("canarytoken", ""),
                "url": resp.get("canarytoken", {}).get("url", ""),
            })
        except Exception as e:
            results.append({"kind": kind, "memo": memo, "status": "FAILED", "error": str(e)})
    return results


def audit_token_coverage(client):
    """Audit deployed token coverage and generate report."""
    tokens_resp = client.list_tokens()
    tokens = tokens_resp.get("tokens", [])
    alerts_resp = client.get_alerts()
    alerts = alerts_resp.get("incidents", [])

    kind_counts = {}
    triggered_tokens = set()
    for token in tokens:
        kind = token.get("kind", "unknown")
        kind_counts[kind] = kind_counts.get(kind, 0) + 1

    for alert in alerts:
        triggered_tokens.add(alert.get("canarytoken", ""))

    untriggered = [t for t in tokens if t.get("canarytoken", "") not in triggered_tokens]
    recommended_types = []
    for kind_name in TOKEN_KINDS:
        if kind_name not in kind_counts:
            recommended_types.append(kind_name)

    return {
        "total_tokens": len(tokens),
        "total_alerts": len(alerts),
        "tokens_by_kind": kind_counts,
        "triggered_token_count": len(triggered_tokens),
        "untriggered_tokens": len(untriggered),
        "missing_token_types": recommended_types,
        "coverage_score": round(len(kind_counts) / len(TOKEN_KINDS) * 100, 1),
    }


def full_audit(client):
    """Run comprehensive Canarytoken deployment audit."""
    coverage = audit_token_coverage(client)
    tokens_resp = client.list_tokens()
    tokens = tokens_resp.get("tokens", [])
    alerts_resp = client.get_alerts()
    alerts = alerts_resp.get("incidents", [])

    token_details = []
    for t in tokens[:30]:
        token_details.append({
            "canarytoken": t.get("canarytoken"),
            "kind": t.get("kind"),
            "memo": t.get("memo"),
            "created": t.get("created_printable"),
            "enabled": t.get("enabled"),
            "url": t.get("url", ""),
        })

    alert_details = []
    for a in alerts[:20]:
        alert_details.append({
            "incident_id": a.get("id"),
            "description": a.get("description"),
            "source_ip": a.get("src_host"),
            "timestamp": a.get("created_printable"),
            "canarytoken": a.get("canarytoken"),
            "acknowledged": a.get("acknowledged"),
        })

    return {
        "audit_type": "Canarytoken Deception Coverage Audit",
        "timestamp": datetime.utcnow().isoformat(),
        "coverage": coverage,
        "deployed_tokens": token_details,
        "recent_alerts": alert_details,
        "recommendation": "Deploy missing token types to improve coverage"
            if coverage["coverage_score"] < 50 else "Good coverage — review untriggered tokens",
    }


def main():
    parser = argparse.ArgumentParser(description="Canarytoken Deception Detection Agent")
    parser.add_argument("--console", required=True, help="Canary Console domain (e.g., abc123)")
    parser.add_argument("--auth-token", required=True, help="API auth token")
    sub = parser.add_subparsers(dest="command")
    sub.add_parser("ping", help="Test API connectivity")
    sub.add_parser("list", help="List all deployed tokens")
    sub.add_parser("alerts", help="Fetch recent alerts")
    p_create = sub.add_parser("create", help="Create a new token")
    p_create.add_argument("--kind", required=True, choices=list(TOKEN_KINDS.keys()))
    p_create.add_argument("--memo", required=True)
    sub.add_parser("coverage", help="Audit token coverage")
    sub.add_parser("full", help="Full deception audit")
    args = parser.parse_args()

    client = CanaryClient(args.console, args.auth_token)

    if args.command == "ping":
        result = client.ping()
    elif args.command == "list":
        result = client.list_tokens()
    elif args.command == "alerts":
        result = client.get_alerts()
    elif args.command == "create":
        result = client.create_token(args.kind, args.memo)
    elif args.command == "coverage":
        result = audit_token_coverage(client)
    elif args.command == "full" or args.command is None:
        result = full_audit(client)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-network-deception-with-honeypots/SKILL.md =====
---
name: implementing-network-deception-with-honeypots
description: Deploy and manage network honeypots using OpenCanary, T-Pot, or Cowrie
  to detect unauthorized access, lateral movement, and attacker reconnaissance.
domain: cybersecurity
subdomain: deception-technology
tags:
- deception
- honeypot
- opencanary
- cowrie
- t-pot
- detection
- lateral-movement
- network-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
- DE.AE-06
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1021
- T1550
---

# Implementing Network Deception with Honeypots

## When to Use

- When deploying deception technology to detect lateral movement
- To create early warning indicators for network intrusion
- During security architecture design to add detection depth
- When monitoring for unauthorized internal scanning or credential theft
- To gather threat intelligence on attacker techniques and tools

## Prerequisites

- Linux server or VM for honeypot deployment (Ubuntu 22.04+ recommended)
- Python 3.8+ with pip for OpenCanary installation
- Docker for T-Pot or containerized deployment
- Network segment with appropriate VLAN configuration
- SIEM integration for alert forwarding (syslog, webhook, or file-based)
- Firewall rules allowing inbound connections to honeypot services

## Workflow

1. **Plan Deployment**: Select honeypot types and network placement strategy.
2. **Install Honeypot**: Deploy OpenCanary, Cowrie, or T-Pot on dedicated host.
3. **Configure Services**: Enable emulated services (SSH, HTTP, SMB, FTP, RDP).
4. **Set Up Alerting**: Configure log forwarding to SIEM and alert channels.
5. **Deploy Canary Tokens**: Place credential files, shares, and DNS entries.
6. **Monitor Interactions**: Analyze honeypot logs for attacker activity.
7. **Tune and Maintain**: Update configurations based on detection results.

## Key Concepts

| Concept | Description |
|---------|-------------|
| OpenCanary | Lightweight Python honeypot with modular service emulation |
| Cowrie | Medium-interaction SSH/Telnet honeypot capturing commands |
| T-Pot | Multi-honeypot platform with ELK stack visualization |
| Canary Token | Tripwire credential or file that alerts when accessed |
| Low-Interaction | Emulates services at protocol level without full OS |
| High-Interaction | Full OS honeypot capturing complete attacker sessions |

## Tools & Systems

| Tool | Purpose |
|------|---------|
| OpenCanary | Modular honeypot daemon with service emulation |
| Cowrie | SSH/Telnet honeypot with session recording |
| T-Pot | All-in-one multi-honeypot platform |
| Dionaea | Malware-capturing honeypot for exploit detection |
| Splunk/Elastic | SIEM for honeypot alert aggregation |

## Output Format

```
Alert: HONEYPOT-[SERVICE]-[DATE]-[SEQ]
Honeypot: [Hostname/IP]
Service: [SSH/HTTP/SMB/FTP/RDP]
Source IP: [Attacker IP]
Interaction: [Login attempt/Port scan/File access]
Credentials Used: [Username:Password if applicable]
Commands Executed: [For SSH honeypots]
Risk Level: [Critical/High/Medium/Low]
```

===== END FILE =====


===== FILE: skills/implementing-network-deception-with-honeypots/LICENSE =====

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


===== FILE: skills/implementing-network-deception-with-honeypots/references/api-reference.md =====
# Network Deception with Honeypots Reference

## OpenCanary Installation

```bash
# Ubuntu/Debian
sudo apt-get install python3-dev python3-pip python3-virtualenv libssl-dev libpcap-dev
virtualenv canary-env && source canary-env/bin/activate
pip install opencanary

# Docker
docker pull thinkst/opencanary
docker run -d --network host -v /path/to/config:/etc/opencanaryd thinkst/opencanary
```

## OpenCanary CLI

```bash
# Generate default config
opencanaryd --copyconfig

# Start daemon
opencanaryd --start

# Stop daemon
opencanaryd --stop

# Check status
opencanaryd --status

# Run in foreground (debug)
opencanaryd --dev
```

## Configuration File (`/etc/opencanaryd/opencanary.conf`)

```json
{
    "device.node_id": "honeypot-dmz-01",
    "ssh.enabled": true,
    "ssh.port": 22,
    "ssh.version": "SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3",
    "http.enabled": true,
    "http.port": 80,
    "http.banner": "Apache/2.4.41 (Ubuntu)",
    "http.skin": "nasLogin",
    "smb.enabled": true,
    "smb.filelist": [{"name": "passwords.xlsx", "type": "xlsx"}],
    "ftp.enabled": true,
    "ftp.port": 21,
    "ftp.banner": "FTP server ready",
    "mysql.enabled": true,
    "mysql.port": 3306,
    "rdp.enabled": true,
    "rdp.port": 3389
}
```

## Available Service Modules

| Service | Config Key | Default Port | Interaction Level |
|---------|-----------|-------------|-------------------|
| SSH | ssh.enabled | 22 | Medium |
| HTTP | http.enabled | 80 | Low-Medium |
| FTP | ftp.enabled | 21 | Low |
| SMB | smb.enabled | 445 | Low |
| MySQL | mysql.enabled | 3306 | Low |
| RDP | rdp.enabled | 3389 | Low |
| Telnet | telnet.enabled | 23 | Low |
| SNMP | snmp.enabled | 161 | Low |
| Git | git.enabled | 9418 | Low |
| Redis | redis.enabled | 6379 | Low |
| VNC | vnc.enabled | 5000 | Low |

## Log Format (JSON, one per line)

```json
{
    "dst_host": "10.0.0.50",
    "dst_port": 22,
    "src_host": "10.0.0.100",
    "src_port": 45321,
    "logtype": 3001,
    "node_id": "honeypot-dmz-01",
    "utc_time": "2025-03-01 14:30:00.123456",
    "logdata": {"USERNAME": "admin", "PASSWORD": "password123"}
}
```

## Log Type Codes

| Code | Service | Event |
|------|---------|-------|
| 1001 | FTP | Login attempt |
| 2001 | HTTP | Login attempt |
| 3001 | SSH | Login attempt |
| 5001 | SMB | File open |
| 6001 | Telnet | Login attempt |
| 7001 | MySQL | Login attempt |
| 8001 | RDP | Login attempt |

## Cowrie SSH Honeypot

```bash
# Docker deployment
docker run -d -p 22:2222 cowrie/cowrie

# Session replay
bin/playlog log/tty/20250301-143000-abc123.log
```

## Syslog Forwarding

```json
{
    "logger": {
        "class": "PyLogger",
        "kwargs": {
            "handlers": {
                "syslog": {
                    "class": "logging.handlers.SysLogHandler",
                    "address": ["siem.example.com", 514]
                }
            }
        }
    }
}
```

===== END FILE =====


===== FILE: skills/implementing-network-deception-with-honeypots/scripts/agent.py =====
#!/usr/bin/env python3
"""Honeypot Deployment Agent - deploys OpenCanary honeypots and analyzes interaction logs."""

import json
import argparse
import logging
import subprocess
import os
from collections import defaultdict
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

OPENCANARY_CONFIG_TEMPLATE = {
    "device.node_id": "opencanary-001",
    "ip.ignorelist": [],
    "logtype.console.enabled": True,
    "logger": {
        "class": "PyLogger",
        "kwargs": {
            "formatters": {"plain": {"format": "%(message)s"}},
            "handlers": {
                "file": {
                    "class": "logging.FileHandler",
                    "filename": "/var/tmp/opencanary.log",
                },
                "console": {
                    "class": "logging.StreamHandler",
                    "stream": "ext://sys.stdout",
                },
            },
        },
    },
    "ftp.enabled": False,
    "ftp.port": 21,
    "ftp.banner": "FTP server ready",
    "http.enabled": False,
    "http.port": 80,
    "http.banner": "Apache/2.4.41 (Ubuntu)",
    "http.skin": "nasLogin",
    "httpproxy.enabled": False,
    "httpproxy.port": 8080,
    "ssh.enabled": False,
    "ssh.port": 22,
    "ssh.version": "SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3",
    "smb.enabled": False,
    "smb.filelist": [{"name": "passwords.xlsx", "type": "xlsx"}, {"name": "backup-credentials.txt", "type": "txt"}],
    "telnet.enabled": False,
    "telnet.port": 23,
    "telnet.banner": "Welcome to the management console",
    "rdp.enabled": False,
    "rdp.port": 3389,
    "mysql.enabled": False,
    "mysql.port": 3306,
    "snmp.enabled": False,
    "snmp.port": 161,
}


def generate_config(services, node_id="opencanary-001", log_path="/var/tmp/opencanary.log"):
    """Generate OpenCanary configuration with specified services enabled."""
    config = OPENCANARY_CONFIG_TEMPLATE.copy()
    config["device.node_id"] = node_id
    config["logger"]["kwargs"]["handlers"]["file"]["filename"] = log_path
    for service in services:
        key = f"{service}.enabled"
        if key in config:
            config[key] = True
    enabled = [s for s in services if f"{s}.enabled" in config]
    logger.info("Generated config with %d services: %s", len(enabled), ", ".join(enabled))
    return config


def deploy_opencanary(config, config_path="/etc/opencanaryd/opencanary.conf"):
    """Deploy OpenCanary with generated configuration."""
    os.makedirs(os.path.dirname(config_path), exist_ok=True)
    with open(config_path, "w") as f:
        json.dump(config, f, indent=2)
    logger.info("Configuration written to %s", config_path)
    start_cmd = ["opencanaryd", "--start"]
    result = subprocess.run(start_cmd, capture_output=True, text=True, timeout=120)
    return {"config_path": config_path, "started": result.returncode == 0, "output": result.stdout[:200]}


def parse_opencanary_log(log_path="/var/tmp/opencanary.log"):
    """Parse OpenCanary JSON log file for interaction events."""
    events = []
    try:
        with open(log_path) as f:
            for line in f:
                line = line.strip()
                if not line:
                    continue
                try:
                    event = json.loads(line)
                    events.append({
                        "timestamp": event.get("utc_time", ""),
                        "dst_host": event.get("dst_host", ""),
                        "dst_port": event.get("dst_port", 0),
                        "src_host": event.get("src_host", ""),
                        "src_port": event.get("src_port", 0),
                        "logtype": event.get("logtype", 0),
                        "node_id": event.get("node_id", ""),
                        "logdata": event.get("logdata", {}),
                    })
                except json.JSONDecodeError:
                    continue
    except FileNotFoundError:
        logger.warning("Log file not found: %s", log_path)
    return events


def analyze_interactions(events):
    """Analyze honeypot interactions for threat intelligence."""
    by_source = defaultdict(lambda: {"count": 0, "services": set(), "credentials": []})
    by_service = defaultdict(int)
    credential_attempts = []
    log_type_map = {
        1001: "ftp_login", 2001: "http_login", 3001: "ssh_login",
        5001: "smb_file_open", 6001: "telnet_login", 7001: "mysql_login",
        8001: "rdp_login",
    }

    for event in events:
        src = event["src_host"]
        service = log_type_map.get(event["logtype"], f"type_{event['logtype']}")
        by_source[src]["count"] += 1
        by_source[src]["services"].add(service)
        by_service[service] += 1
        logdata = event.get("logdata", {})
        username = logdata.get("USERNAME", logdata.get("username", ""))
        password = logdata.get("PASSWORD", logdata.get("password", ""))
        if username:
            cred = {"username": username, "password": password, "service": service, "source": src}
            credential_attempts.append(cred)
            by_source[src]["credentials"].append(cred)

    source_summary = {}
    for ip, data in sorted(by_source.items(), key=lambda x: x[1]["count"], reverse=True):
        source_summary[ip] = {
            "interaction_count": data["count"],
            "services_targeted": list(data["services"]),
            "credential_attempts": len(data["credentials"]),
        }

    return {
        "total_interactions": len(events),
        "unique_sources": len(by_source),
        "service_distribution": dict(sorted(by_service.items(), key=lambda x: x[1], reverse=True)),
        "top_sources": dict(list(source_summary.items())[:20]),
        "credential_attempts": len(credential_attempts),
        "unique_usernames": len(set(c["username"] for c in credential_attempts)),
        "top_credentials": credential_attempts[:20],
    }


def check_honeypot_status():
    """Check if OpenCanary daemon is running."""
    cmd = ["opencanaryd", "--status"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    is_running = "running" in result.stdout.lower() or result.returncode == 0
    return {"running": is_running, "status_output": result.stdout.strip()[:200]}


def generate_report(analysis, status, config):
    """Generate honeypot deployment and interaction report."""
    enabled_services = [k.replace(".enabled", "") for k, v in config.items() if k.endswith(".enabled") and v]
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "honeypot_type": "OpenCanary",
        "node_id": config.get("device.node_id", ""),
        "enabled_services": enabled_services,
        "daemon_status": status,
        "interaction_analysis": analysis,
    }
    return report


def main():
    parser = argparse.ArgumentParser(description="Honeypot Deployment and Analysis Agent")
    parser.add_argument("--action", choices=["deploy", "analyze", "status", "full"], default="analyze")
    parser.add_argument("--services", nargs="+", default=["ssh", "http", "smb", "ftp", "telnet"],
                        help="Services to enable (default: ssh http smb ftp telnet)")
    parser.add_argument("--node-id", default="opencanary-001", help="Honeypot node identifier")
    parser.add_argument("--log-path", default="/var/tmp/opencanary.log", help="OpenCanary log file path")
    parser.add_argument("--config-path", default="/etc/opencanaryd/opencanary.conf")
    parser.add_argument("--output", default="honeypot_report.json")
    args = parser.parse_args()

    config = generate_config(args.services, args.node_id, args.log_path)

    if args.action in ("deploy", "full"):
        deploy_result = deploy_opencanary(config, args.config_path)
        logger.info("Deployment: %s", "success" if deploy_result["started"] else "failed")

    status = check_honeypot_status()
    events = parse_opencanary_log(args.log_path)
    analysis = analyze_interactions(events)
    report = generate_report(analysis, status, config)

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("Honeypot: %d interactions from %d sources, %d credential attempts",
                analysis["total_interactions"], analysis["unique_sources"],
                analysis["credential_attempts"])
    print(json.dumps(report, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====
