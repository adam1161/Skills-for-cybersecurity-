

===== FILE: skills/performing-active-directory-forest-trust-attack/SKILL.md =====
---
name: performing-active-directory-forest-trust-attack
description: Enumerate and audit Active Directory forest trust relationships using
  Impacket for SID filtering analysis, trust key extraction, cross-forest SID history
  abuse detection, and inter-realm Kerberos ticket assessment. Use when red-teaming
  multi-forest AD environments or auditing forest trusts for cross-forest privilege
  escalation and trust ticket forgery exposure.
domain: cybersecurity
subdomain: red-team
tags:
- active-directory
- forest-trust
- impacket
- SID-filtering
- kerberos
- red-team
- trust-enumeration
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1558.003
---

# Performing Active Directory Forest Trust Attack

## Overview

Active Directory forest trusts enable authentication across organizational boundaries but introduce attack surface if misconfigured. This skill uses impacket to enumerate trust relationships, analyze SID filtering configuration, detect SID history abuse vectors, perform cross-forest SID lookups via LSA/LSAT RPC calls, and assess inter-realm Kerberos ticket configurations for trust ticket forgery risks.


## When to Use

- When conducting security assessments that involve performing active directory forest trust attack
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Python 3.9+ with `impacket`, `ldap3`
- Domain credentials with read access to AD trust objects
- Network access to Domain Controllers (ports 389, 445, 88)
- Authorized penetration testing engagement or lab environment


> **Legal Notice:** This skill is for authorized security testing and educational purposes only. Unauthorized use against systems you do not own or have written permission to test is illegal and may violate computer fraud laws.

## Steps

1. Enumerate forest trust relationships via LDAP trusted domain objects
2. Query trust attributes and SID filtering status for each trust
3. Perform SID lookups across trust boundaries using LsarLookupNames3
4. Enumerate foreign security principals in trusted domains
5. Check for SID history on cross-forest accounts
6. Assess trust direction and transitivity for lateral movement paths
7. Generate trust security audit report with risk findings

## Expected Output

- JSON report listing all trust relationships, SID filtering status, foreign principals, trust direction/transitivity, and risk assessment
- Cross-forest attack path analysis with remediation recommendations

===== END FILE =====


===== FILE: skills/performing-active-directory-forest-trust-attack/LICENSE =====

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


===== FILE: skills/performing-active-directory-forest-trust-attack/references/api-reference.md =====
# API Reference — Performing Active Directory Forest Trust Attack

## Libraries Used
- **impacket**: SMB/RPC transport for LSA SID lookups via `lsat.hLsarLookupSids2()`
- **ldap3**: LDAP queries against `trustedDomain` objects and `foreignSecurityPrincipal` containers
- **json**: JSON serialization for audit reports

## CLI Interface
```
python agent.py --dc 10.0.0.1 --domain corp.local --username admin --password Pass123 trusts
python agent.py --dc 10.0.0.1 --domain corp.local --username admin --password Pass123 foreign
python agent.py --dc 10.0.0.1 --domain corp.local --username admin --password Pass123 lookup-sid --sid S-1-5-21-...
python agent.py --dc 10.0.0.1 --domain corp.local --username admin --password Pass123 full
```

## Core Functions

### `enumerate_trusts_ldap(dc_host, domain, username, password)` — Trust enumeration
LDAP search: `(objectClass=trustedDomain)` under `CN=System,DC=...`.
Attributes: trustPartner, trustDirection, trustType, trustAttributes, flatName.
Decodes trust attribute bitmask for SID filtering, forest transitivity, RC4 encryption.

### `enumerate_foreign_principals(dc_host, domain, username, password)` — Cross-forest members
LDAP search: `(objectClass=foreignSecurityPrincipal)` under `CN=ForeignSecurityPrincipals`.
Filters well-known SIDs (S-1-5-x with 3 dashes). Returns group memberships.

### `lookup_sid_cross_forest(dc_host, domain, username, password, target_sid)` — LSA SID resolution
Opens SMB transport to `\lsarpc`, binds MSRPC_UUID_LSAT, calls `hLsarLookupSids2()`.
Resolves SIDs across trust boundaries.

### `assess_trust_risk(trusts, foreign_principals)` — Risk scoring
Scoring: +40 SID filtering disabled, +20 RC4 encryption, +15 bidirectional trust,
+10 forest transitive.

### `full_audit(dc_host, domain, username, password)` — Comprehensive audit

## Trust Direction Values
| Value | Direction |
|-------|-----------|
| 0 | Disabled |
| 1 | Inbound |
| 2 | Outbound |
| 3 | Bidirectional |

## Trust Attribute Flags
| Flag | Hex | Description |
|------|-----|-------------|
| NON_TRANSITIVE | 0x01 | Trust does not extend transitively |
| QUARANTINED_DOMAIN | 0x04 | SID filtering enabled |
| FOREST_TRANSITIVE | 0x08 | Forest-wide transitive trust |
| USES_RC4_ENCRYPTION | 0x80 | RC4 trust key (weaker than AES) |

## Impacket RPC Calls
| Call | Module | Purpose |
|------|--------|---------|
| `hLsarOpenPolicy2` | lsad | Open LSA policy handle |
| `hLsarLookupSids2` | lsat | Resolve SIDs to names across trust |
| SMBTransport(`\lsarpc`) | transport | RPC transport over SMB |

## Dependencies
- `impacket` >= 0.11.0
- `ldap3` >= 2.9.0
- Network access to DC ports 389 (LDAP), 445 (SMB), 88 (Kerberos)

===== END FILE =====


===== FILE: skills/performing-active-directory-forest-trust-attack/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and educational environments only.
# Usage against targets without prior mutual consent is illegal.
# It is the end user's responsibility to obey all applicable local, state and federal laws.
"""Agent for AD forest trust enumeration and security assessment using impacket."""

import json
import argparse
from datetime import datetime

try:
    from impacket.dcerpc.v5 import transport, lsat, lsad
    from impacket.dcerpc.v5.dtypes import MAXIMUM_ALLOWED
    from impacket.dcerpc.v5.samr import SID_NAME_USE
except ImportError:
    transport = None

try:
    import ldap3
    from ldap3 import Server, Connection, ALL, SUBTREE
except ImportError:
    ldap3 = None

TRUST_DIRECTION = {0: "Disabled", 1: "Inbound", 2: "Outbound", 3: "Bidirectional"}
TRUST_TYPE = {1: "Downlevel (Windows NT)", 2: "Uplevel (Windows 2000+)", 3: "MIT Kerberos", 4: "DCE"}
TRUST_ATTRIBUTES = {
    0x00000001: "NON_TRANSITIVE",
    0x00000002: "UPLEVEL_ONLY",
    0x00000004: "QUARANTINED_DOMAIN (SID Filtering Enabled)",
    0x00000008: "FOREST_TRANSITIVE",
    0x00000010: "CROSS_ORGANIZATION",
    0x00000020: "WITHIN_FOREST",
    0x00000040: "TREAT_AS_EXTERNAL",
    0x00000080: "USES_RC4_ENCRYPTION",
    0x00000200: "CROSS_ORGANIZATION_NO_TGT_DELEGATION",
    0x00000400: "PIM_TRUST",
}


def enumerate_trusts_ldap(dc_host, domain, username, password):
    """Enumerate AD trust relationships via LDAP trustedDomain objects."""
    if not ldap3:
        return {"error": "ldap3 not installed: pip install ldap3"}
    server = Server(dc_host, get_info=ALL, use_ssl=False)
    base_dn = ",".join(f"DC={p}" for p in domain.split("."))
    conn = Connection(server, user=f"{domain}\\{username}", password=password, auto_bind=True)
    conn.search(
        search_base=f"CN=System,{base_dn}",
        search_filter="(objectClass=trustedDomain)",
        search_scope=SUBTREE,
        attributes=[
            "cn", "trustPartner", "trustDirection", "trustType",
            "trustAttributes", "securityIdentifier", "whenCreated",
            "flatName", "trustPosixOffset",
        ],
    )
    trusts = []
    for entry in conn.entries:
        attrs = entry.entry_attributes_as_dict
        direction_val = int(attrs.get("trustDirection", [0])[0])
        type_val = int(attrs.get("trustType", [0])[0])
        attr_val = int(attrs.get("trustAttributes", [0])[0])
        decoded_attrs = []
        for bit, name in TRUST_ATTRIBUTES.items():
            if attr_val & bit:
                decoded_attrs.append(name)
        sid_filtering = bool(attr_val & 0x00000004)
        forest_trust = bool(attr_val & 0x00000008)
        trusts.append({
            "trust_partner": str(attrs.get("trustPartner", [""])[0]),
            "flat_name": str(attrs.get("flatName", [""])[0]),
            "trust_direction": TRUST_DIRECTION.get(direction_val, str(direction_val)),
            "trust_type": TRUST_TYPE.get(type_val, str(type_val)),
            "trust_attributes_raw": attr_val,
            "trust_attributes": decoded_attrs,
            "sid_filtering_enabled": sid_filtering,
            "forest_transitive": forest_trust,
            "when_created": str(attrs.get("whenCreated", [""])[0]),
        })
    conn.unbind()
    return trusts


def enumerate_foreign_principals(dc_host, domain, username, password):
    """Find foreign security principals (cross-forest members) in the domain."""
    if not ldap3:
        return {"error": "ldap3 not installed"}
    server = Server(dc_host, get_info=ALL, use_ssl=False)
    base_dn = ",".join(f"DC={p}" for p in domain.split("."))
    conn = Connection(server, user=f"{domain}\\{username}", password=password, auto_bind=True)
    conn.search(
        search_base=f"CN=ForeignSecurityPrincipals,{base_dn}",
        search_filter="(objectClass=foreignSecurityPrincipal)",
        search_scope=SUBTREE,
        attributes=["cn", "objectSid", "whenCreated", "memberOf"],
    )
    principals = []
    for entry in conn.entries:
        attrs = entry.entry_attributes_as_dict
        sid = str(attrs.get("cn", [""])[0])
        member_of = [str(g) for g in attrs.get("memberOf", [])]
        principals.append({
            "sid": sid,
            "member_of_groups": member_of,
            "when_created": str(attrs.get("whenCreated", [""])[0]),
            "is_well_known": sid.startswith("S-1-5-") and sid.count("-") == 3,
        })
    conn.unbind()
    custom_principals = [p for p in principals if not p["is_well_known"]]
    return {
        "total_foreign_principals": len(principals),
        "custom_foreign_principals": len(custom_principals),
        "principals": custom_principals[:30],
    }


def lookup_sid_cross_forest(dc_host, domain, username, password, target_sid):
    """Resolve a SID across forest trust using LSA LookupSids RPC call."""
    if not transport:
        return {"error": "impacket not installed: pip install impacket"}
    rpctransport = transport.SMBTransport(dc_host, filename=r"\lsarpc")
    rpctransport.set_credentials(username, password, domain)
    dce = rpctransport.get_dce_rpc()
    dce.connect()
    dce.bind(lsat.MSRPC_UUID_LSAT)
    resp = lsad.hLsarOpenPolicy2(dce, MAXIMUM_ALLOWED)
    policy_handle = resp["PolicyHandle"]
    try:
        from impacket.dcerpc.v5.dtypes import RPC_SID
        sid = RPC_SID()
        sid.fromCanonical(target_sid)
        resp = lsat.hLsarLookupSids2(dce, policy_handle, [sid])
        names = []
        for item in resp["TranslatedNames"]["Names"]:
            names.append({
                "name": item["Name"],
                "sid_type": SID_NAME_USE.enumItems(item["Use"]).name if hasattr(SID_NAME_USE, 'enumItems') else str(item["Use"]),
                "domain_index": item["DomainIndex"],
            })
        return {"target_sid": target_sid, "resolved_names": names}
    except Exception as e:
        return {"target_sid": target_sid, "error": str(e)}
    finally:
        dce.disconnect()


def assess_trust_risk(trusts, foreign_principals):
    """Assess security risk of trust relationships."""
    findings = []
    for trust in trusts:
        risk = 0
        issues = []
        if not trust.get("sid_filtering_enabled"):
            risk += 40
            issues.append("SID filtering DISABLED — SID history attacks possible")
        if trust.get("trust_direction") == "Bidirectional":
            risk += 15
            issues.append("Bidirectional trust increases attack surface")
        if trust.get("forest_transitive"):
            risk += 10
            issues.append("Forest transitive trust — all domains reachable")
        if "USES_RC4_ENCRYPTION" in trust.get("trust_attributes", []):
            risk += 20
            issues.append("RC4 encryption — vulnerable to trust key cracking")
        risk = min(risk, 100)
        findings.append({
            "trust_partner": trust.get("trust_partner"),
            "risk_score": risk,
            "risk_level": "CRITICAL" if risk >= 70 else "HIGH" if risk >= 50 else "MEDIUM" if risk >= 25 else "LOW",
            "issues": issues,
            "recommendation": "Enable SID filtering and migrate to AES encryption"
                if risk >= 50 else "Review trust configuration",
        })
    return findings


def full_audit(dc_host, domain, username, password):
    """Run comprehensive forest trust security audit."""
    trusts = enumerate_trusts_ldap(dc_host, domain, username, password)
    foreign = enumerate_foreign_principals(dc_host, domain, username, password)
    risk = assess_trust_risk(trusts if isinstance(trusts, list) else [], foreign)
    critical = sum(1 for r in risk if r["risk_level"] == "CRITICAL")
    high = sum(1 for r in risk if r["risk_level"] == "HIGH")
    no_sid_filter = sum(1 for t in (trusts if isinstance(trusts, list) else []) if not t.get("sid_filtering_enabled"))
    return {
        "audit_type": "AD Forest Trust Security Assessment",
        "timestamp": datetime.utcnow().isoformat(),
        "domain": domain,
        "summary": {
            "total_trusts": len(trusts) if isinstance(trusts, list) else 0,
            "trusts_without_sid_filtering": no_sid_filter,
            "foreign_principals": foreign.get("custom_foreign_principals", 0),
            "critical_findings": critical,
            "high_findings": high,
        },
        "trusts": trusts,
        "foreign_principals": foreign,
        "risk_assessment": risk,
    }


def main():
    parser = argparse.ArgumentParser(description="AD Forest Trust Security Audit Agent")
    parser.add_argument("--dc", required=True, help="Domain Controller hostname or IP")
    parser.add_argument("--domain", required=True, help="Domain name (e.g., corp.local)")
    parser.add_argument("--username", required=True, help="Domain username")
    parser.add_argument("--password", required=True, help="Domain password")
    sub = parser.add_subparsers(dest="command")
    sub.add_parser("trusts", help="Enumerate trust relationships")
    sub.add_parser("foreign", help="List foreign security principals")
    p_sid = sub.add_parser("lookup-sid", help="Cross-forest SID lookup")
    p_sid.add_argument("--sid", required=True, help="Target SID to resolve")
    sub.add_parser("full", help="Full trust security audit")
    args = parser.parse_args()

    if args.command == "trusts":
        result = enumerate_trusts_ldap(args.dc, args.domain, args.username, args.password)
    elif args.command == "foreign":
        result = enumerate_foreign_principals(args.dc, args.domain, args.username, args.password)
    elif args.command == "lookup-sid":
        result = lookup_sid_cross_forest(args.dc, args.domain, args.username, args.password, args.sid)
    elif args.command == "full" or args.command is None:
        result = full_audit(args.dc, args.domain, args.username, args.password)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-red-team-with-covenant/SKILL.md =====
---
name: performing-red-team-with-covenant
description: Conducts red team operations using the Covenant C2 framework for authorized
  adversary simulation, covering listener setup, grunt deployment, task execution,
  and lateral movement tracking. Use when standing up or operating a Covenant command-and-control
  server for an authorized red team engagement.
domain: cybersecurity
subdomain: red-team
tags:
- red-team
- c2
- covenant
- adversary-simulation
- penetration-testing
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1021
---
# Performing Red Team Operations with Covenant C2

## Overview

Covenant is a collaborative .NET C2 framework for red teamers that provides a Swagger-documented REST API for managing listeners, launchers, grunts (agents), and tasks. This skill covers automating Covenant operations through its API for authorized red team engagements: creating HTTP/HTTPS listeners, generating binary and PowerShell launchers, deploying grunts, executing tasks on compromised hosts, and tracking lateral movement.


## When to Use

- When conducting security assessments that involve performing red team with covenant
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Covenant C2 server deployed (Docker or .NET 6)
- Python 3.9+ with `requests` library
- Covenant API token (obtained via /api/users/login)
- Written authorization for red team engagement
- Isolated lab or authorized target environment

## Steps

### Step 1: Authenticate to Covenant API
Obtain a JWT token by posting credentials to /api/users/login endpoint.

### Step 2: Create Listener
Configure an HTTP or HTTPS listener with callback URLs and bind address.

### Step 3: Generate Launcher
Create a binary, PowerShell, or MSBuild launcher tied to the listener for grunt deployment.

### Step 4: Deploy and Manage Grunts
Monitor grunt callbacks, execute tasks, and collect output from compromised hosts.

### Step 5: Document Operations
Generate an operations report documenting all actions, timestamps, and findings.

## Expected Output

JSON report with listener configuration, active grunts, executed tasks, and task output for engagement documentation.

===== END FILE =====


===== FILE: skills/performing-red-team-with-covenant/LICENSE =====

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


===== FILE: skills/performing-red-team-with-covenant/references/api-reference.md =====
# API Reference: Performing Red Team with Covenant C2

## Covenant REST API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| /api/users/login | POST | Authenticate and get JWT token |
| /api/listeners | GET | List all listeners |
| /api/listeners/http | POST | Create HTTP listener |
| /api/grunts | GET | List all grunts (agents) |
| /api/grunts/{id}/interact | POST | Execute task on grunt |
| /api/grunttasks/{id} | GET | Get task output |
| /api/launchers/binary | PUT | Generate binary launcher |
| /api/launchers/powershell | PUT | Generate PowerShell launcher |
| /api/launchers/msbuild | PUT | Generate MSBuild launcher |

## Authentication

```json
POST /api/users/login
{"userName": "admin", "password": "pass"}

Response: {"covenantToken": "eyJhbGciOi..."}
Header: Authorization: Bearer <token>
```

## Listener Configuration

| Field | Type | Description |
|-------|------|-------------|
| name | string | Listener display name |
| bindAddress | string | IP to bind (0.0.0.0 for all) |
| bindPort | int | Port for grunt callbacks |
| connectAddresses | array | Callback addresses for grunts |
| listenerTypeId | int | 1=HTTP, 2=Bridge |

## Grunt Status Values

| Status | Description |
|--------|-------------|
| Uninitialized | Grunt created but not connected |
| Stage0 | Initial callback received |
| Stage1 | Key exchange in progress |
| Stage2 | Fully staged and active |
| Active | Connected and ready for tasks |
| Lost | Missed check-in threshold |

## Built-in Tasks

| Task | Description |
|------|-------------|
| WhoAmI | Current user identity |
| GetHostname | Target hostname |
| ListDirectory | Directory listing |
| Download | Retrieve file from target |
| Upload | Upload file to target |
| PowerShell | Execute PowerShell command |
| Assembly | Load and execute .NET assembly |
| Mimikatz | Credential extraction |

## References

- Covenant GitHub: https://github.com/cobbr/Covenant
- Covenant Wiki: https://github.com/cobbr/Covenant/wiki
- Covenant API Docs: Swagger UI at https://<host>:7443/swagger
- Netwrix Tutorial: https://netwrix.com/en/resources/blog/covenant-c2-tutorial/

===== END FILE =====


===== FILE: skills/performing-red-team-with-covenant/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for red team operations with Covenant C2 framework.

Automates Covenant C2 operations through its REST API: listener
management, launcher generation, grunt monitoring, and task
execution for authorized adversary simulation engagements.
"""
# For authorized red team engagements only

import argparse
import json
import os
from datetime import datetime
from pathlib import Path

try:
    import requests
except ImportError:
    requests = None


class CovenantC2Agent:
    """Manages Covenant C2 operations via its REST API."""

    def __init__(self, covenant_url, username, password,
                 output_dir="./covenant_ops"):
        self.base_url = covenant_url.rstrip("/")
        self.token = None
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.operations_log = []
        self._authenticate(username, password)

    def _authenticate(self, username, password):
        """Authenticate and obtain JWT token from Covenant API."""
        if not requests:
            return
        try:
            resp = requests.post(
                f"{self.base_url}/api/users/login",
                json={"userName": username, "password": password},
                verify=False, timeout=10,
            )
            if resp.status_code == 200:
                data = resp.json()
                self.token = data.get("covenantToken") or data.get("token")
                self._log("authenticate", "success", {"user": username})
        except requests.RequestException as e:
            self._log("authenticate", "failed", {"error": str(e)})

    def _api(self, method, path, data=None):
        if not requests or not self.token:
            return None
        try:
            resp = requests.request(
                method, f"{self.base_url}/api{path}",
                headers={"Authorization": f"Bearer {self.token}",
                          "Content-Type": "application/json"},
                json=data, verify=False, timeout=15,
            )
            return resp
        except requests.RequestException:
            return None

    def _log(self, action, status, details=None):
        self.operations_log.append({
            "timestamp": datetime.utcnow().isoformat(),
            "action": action, "status": status,
            "details": details or {},
        })

    def list_listeners(self):
        """List all configured listeners."""
        resp = self._api("GET", "/listeners")
        if resp and resp.status_code == 200:
            listeners = resp.json()
            self._log("list_listeners", "success", {"count": len(listeners)})
            return [{"id": l["id"], "name": l["name"], "status": l["status"],
                     "bindAddress": l.get("bindAddress"),
                     "bindPort": l.get("bindPort"),
                     "listenerType": l.get("listenerType", {}).get("name")}
                    for l in listeners]
        return []

    def create_http_listener(self, name, bind_port=80, connect_addresses=None):
        """Create an HTTP listener for grunt callbacks."""
        listener_data = {
            "name": name,
            "bindAddress": "0.0.0.0",
            "bindPort": bind_port,
            "connectAddresses": connect_addresses or ["0.0.0.0"],
            "listenerTypeId": 1,
            "status": "Active",
        }
        resp = self._api("POST", "/listeners/http", listener_data)
        if resp and resp.status_code in (200, 201):
            result = resp.json()
            self._log("create_listener", "success",
                      {"name": name, "port": bind_port, "id": result.get("id")})
            return result
        self._log("create_listener", "failed",
                  {"status": resp.status_code if resp else 0})
        return None

    def list_grunts(self):
        """List all active grunts (agents)."""
        resp = self._api("GET", "/grunts")
        if resp and resp.status_code == 200:
            grunts = resp.json()
            self._log("list_grunts", "success", {"count": len(grunts)})
            return [{"id": g["id"], "name": g["name"], "status": g["status"],
                     "hostname": g.get("hostname"), "userName": g.get("userName"),
                     "ipAddress": g.get("ipAddress"),
                     "operatingSystem": g.get("operatingSystem"),
                     "integrity": g.get("integrity"),
                     "lastCheckIn": g.get("lastCheckIn")}
                    for g in grunts]
        return []

    def create_launcher(self, listener_id, launcher_type="Binary"):
        """Generate a launcher payload for grunt deployment."""
        resp = self._api("PUT", f"/launchers/{launcher_type.lower()}",
                         {"listenerId": listener_id})
        if resp and resp.status_code == 200:
            launcher = resp.json()
            self._log("create_launcher", "success",
                      {"type": launcher_type, "listener_id": listener_id})
            return {"type": launcher_type, "launcherString": launcher.get("launcherString", "")[:200]}
        return None

    def execute_task(self, grunt_id, task_name, parameters=None):
        """Assign and execute a task on a grunt."""
        task_data = {
            "gruntId": grunt_id,
            "taskName": task_name,
            "parameters": parameters or [],
        }
        resp = self._api("POST", f"/grunts/{grunt_id}/interact", task_data)
        if resp and resp.status_code in (200, 201):
            result = resp.json()
            self._log("execute_task", "success",
                      {"grunt_id": grunt_id, "task": task_name})
            return {"taskId": result.get("id"), "output": result.get("gruntTaskOutput", "")}
        self._log("execute_task", "failed",
                  {"grunt_id": grunt_id, "task": task_name})
        return None

    def get_task_output(self, task_id):
        """Retrieve output from a completed task."""
        resp = self._api("GET", f"/grunttasks/{task_id}")
        if resp and resp.status_code == 200:
            return resp.json().get("gruntTaskOutput", "")
        return None

    def generate_report(self):
        """Generate an operations report for engagement documentation."""
        listeners = self.list_listeners()
        grunts = self.list_grunts()

        report = {
            "report_date": datetime.utcnow().isoformat(),
            "covenant_url": self.base_url,
            "active_listeners": listeners,
            "active_grunts": grunts,
            "operations_log": self.operations_log,
            "total_operations": len(self.operations_log),
        }
        out = self.output_dir / "covenant_ops_report.json"
        with open(out, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))
        return report


def main():
    parser = argparse.ArgumentParser(
        description="Covenant C2 red team operations agent (authorized use only)"
    )
    parser.add_argument("covenant_url", help="Covenant server URL (e.g. https://10.0.0.5:7443)")
    parser.add_argument("--username", default="admin", help="Covenant username")
    parser.add_argument("--password", required=True, help="Covenant password")
    parser.add_argument("--output-dir", default="./covenant_ops",
                        help="Output directory for ops report")
    args = parser.parse_args()

    os.makedirs(args.output_dir, exist_ok=True)
    agent = CovenantC2Agent(args.covenant_url, args.username, args.password,
                            output_dir=args.output_dir)
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====
