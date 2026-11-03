

===== FILE: skills/analyzing-active-directory-acl-abuse/SKILL.md =====
---
name: analyzing-active-directory-acl-abuse
description: Detect dangerous ACL misconfigurations in Active Directory using ldap3
  to identify GenericAll, WriteDACL, and WriteOwner abuse paths
domain: cybersecurity
subdomain: identity-security
tags:
- active-directory
- acl-abuse
- ldap
- privilege-escalation
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.AA-06
mitre_attack:
- T1098
- T1098.007
- T1484.001
- T1222.001
- T1078.002
---


# Analyzing Active Directory ACL Abuse

## Overview

Active Directory Access Control Lists (ACLs) define permissions on AD objects through Discretionary Access Control Lists (DACLs) containing Access Control Entries (ACEs). Misconfigured ACEs can grant non-privileged users dangerous permissions such as GenericAll (full control), WriteDACL (modify permissions), WriteOwner (take ownership), and GenericWrite (modify attributes) on sensitive objects like Domain Admins groups, domain controllers, or GPOs.

This skill uses the ldap3 Python library to connect to a Domain Controller, query objects with their nTSecurityDescriptor attribute, parse the binary security descriptor into SDDL (Security Descriptor Definition Language) format, and identify ACEs that grant dangerous permissions to non-administrative principals. These misconfigurations are the basis for ACL-based attack paths discovered by tools like BloodHound.


## When to Use

- When investigating security incidents that require analyzing active directory acl abuse
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Python 3.9 or later with ldap3 library (`pip install ldap3`)
- Domain user credentials with read access to AD objects
- Network connectivity to Domain Controller on port 389 (LDAP) or 636 (LDAPS)
- Understanding of Active Directory security model and SDDL format

## Steps

1. **Connect to Domain Controller**: Establish an LDAP connection using ldap3 with NTLM or simple authentication. Use LDAPS (port 636) for encrypted connections in production.

2. **Query target objects**: Search the target OU or entire domain for objects including users, groups, computers, and OUs. Request the `nTSecurityDescriptor`, `distinguishedName`, `objectClass`, and `sAMAccountName` attributes.

3. **Parse security descriptors**: Convert the binary nTSecurityDescriptor into its SDDL string representation. Parse each ACE in the DACL to extract the trustee SID, access mask, and ACE type (allow/deny).

4. **Resolve SIDs to principals**: Map security identifiers (SIDs) to human-readable account names using LDAP lookups against the domain. Identify well-known SIDs for built-in groups.

5. **Check for dangerous permissions**: Compare each ACE's access mask against dangerous permission bitmasks: GenericAll (0x10000000), WriteDACL (0x00040000), WriteOwner (0x00080000), GenericWrite (0x40000000), and WriteProperty for specific extended rights.

6. **Filter non-admin trustees**: Exclude expected administrative trustees (Domain Admins, Enterprise Admins, SYSTEM, Administrators) and flag ACEs where non-privileged users or groups hold dangerous permissions.

7. **Map attack paths**: For each finding, document the potential attack chain (e.g., GenericAll on user allows password reset, WriteDACL on group allows adding self to group).

8. **Generate remediation report**: Output a JSON report with all dangerous ACEs, affected objects, non-admin trustees, and recommended remediation steps.

## Expected Output

```json
{
  "domain": "corp.example.com",
  "objects_scanned": 1247,
  "dangerous_aces_found": 8,
  "findings": [
    {
      "severity": "critical",
      "target_object": "CN=Domain Admins,CN=Users,DC=corp,DC=example,DC=com",
      "target_type": "group",
      "trustee": "CORP\\helpdesk-team",
      "permission": "GenericAll",
      "access_mask": "0x10000000",
      "ace_type": "ACCESS_ALLOWED",
      "attack_path": "GenericAll on Domain Admins group allows adding arbitrary members",
      "remediation": "Remove GenericAll ACE for helpdesk-team on Domain Admins"
    }
  ]
}
```

===== END FILE =====


===== FILE: skills/analyzing-active-directory-acl-abuse/LICENSE =====

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


===== FILE: skills/analyzing-active-directory-acl-abuse/references/api-reference.md =====
# Active Directory ACL Abuse API Reference

## ldap3 Python Connection

```python
from ldap3 import Server, Connection, ALL, NTLM, SUBTREE

server = Server("192.168.1.10", get_info=ALL, use_ssl=False)
conn = Connection(server, user="DOMAIN\\user", password="pass",
                  authentication=NTLM, auto_bind=True)

# Search with nTSecurityDescriptor
conn.search(
    "DC=corp,DC=example,DC=com",
    "(objectClass=group)",
    search_scope=SUBTREE,
    attributes=["distinguishedName", "sAMAccountName",
                "objectClass", "nTSecurityDescriptor"],
)
```

## SDDL ACE Format

```
ACE String: (ace_type;ace_flags;rights;object_guid;inherit_guid;trustee_sid)
Example:    (A;;GA;;;S-1-5-21-xxx-512)
```

| Component | Description |
|-----------|-------------|
| `A` | Access Allowed |
| `D` | Access Denied |
| `OA` | Object Access Allowed |
| `GA` | Generic All |
| `GW` | Generic Write |
| `WD` | Write DACL |
| `WO` | Write Owner |

## Dangerous Permission Bitmasks

| Permission | Hex Mask | Risk |
|-----------|----------|------|
| GenericAll | `0x10000000` | Full control over object |
| GenericWrite | `0x40000000` | Modify all writable attributes |
| WriteDACL | `0x00040000` | Modify object permissions |
| WriteOwner | `0x00080000` | Take object ownership |
| WriteProperty | `0x00000020` | Write specific properties |
| ExtendedRight | `0x00000100` | Extended rights (password reset, etc.) |
| Self | `0x00000008` | Self-membership modification |
| Delete | `0x00010000` | Delete the object |

## BloodHound Cypher Queries for ACL Paths

```cypher
-- Find all users with GenericAll on Domain Admins
MATCH p=(n:User)-[r:GenericAll]->(g:Group {name:"DOMAIN ADMINS@CORP.COM"})
RETURN p

-- Find WriteDACL paths from non-admins to high-value targets
MATCH (n:User {admincount:false})
MATCH p=allShortestPaths((n)-[r:WriteDacl|WriteOwner|GenericAll*1..]->(m:Group))
WHERE m.highvalue = true
RETURN p

-- Find GenericWrite on computers for RBCD attacks
MATCH p=(n:User)-[r:GenericWrite]->(c:Computer)
WHERE NOT n.admincount
RETURN n.name, c.name

-- Enumerate all outbound ACL edges for a principal
MATCH p=(n {name:"HELPDESK@CORP.COM"})-[r:GenericAll|GenericWrite|WriteDacl|WriteOwner|Owns]->(m)
RETURN type(r), m.name, labels(m)

-- Find shortest ACL abuse path to Domain Admin
MATCH (n:User {name:"JSMITH@CORP.COM"})
MATCH (da:Group {name:"DOMAIN ADMINS@CORP.COM"})
MATCH p=shortestPath((n)-[r:MemberOf|GenericAll|GenericWrite|WriteDacl|WriteOwner|Owns|ForceChangePassword*1..]->(da))
RETURN p
```

## PowerView Commands for ACL Enumeration

```powershell
# Get ACL for Domain Admins group
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs

# Find interesting ACEs for non-admin users
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {
    $_.ActiveDirectoryRights -match "GenericAll|WriteDacl|WriteOwner"
}

# Get ACL for specific OU
Get-DomainObjectAcl -SearchBase "OU=Servers,DC=corp,DC=com" -ResolveGUIDs
```

===== END FILE =====


===== FILE: skills/analyzing-active-directory-acl-abuse/scripts/agent.py =====
#!/usr/bin/env python3
"""Active Directory ACL abuse detection using ldap3 to find dangerous permissions."""

import argparse
import json
import struct

from ldap3 import Server, Connection, ALL, NTLM, SUBTREE


DANGEROUS_MASKS = {
    "GenericAll": 0x10000000,
    "GenericWrite": 0x40000000,
    "WriteDACL": 0x00040000,
    "WriteOwner": 0x00080000,
    "WriteProperty": 0x00000020,
    "Self": 0x00000008,
    "ExtendedRight": 0x00000100,
    "DeleteChild": 0x00000002,
    "Delete": 0x00010000,
}

ADMIN_SIDS = {
    "S-1-5-18",
    "S-1-5-32-544",
    "S-1-5-9",
}

ADMIN_RID_SUFFIXES = {
    "-500",
    "-512",
    "-516",
    "-518",
    "-519",
    "-498",
}

ATTACK_PATHS = {
    "GenericAll": {
        "user": "Full control allows password reset, Kerberoasting via SPN, or shadow credential attack",
        "group": "Full control allows adding arbitrary members to the group",
        "computer": "Full control allows resource-based constrained delegation attack",
        "organizationalUnit": "Full control allows linking malicious GPO or moving objects",
    },
    "WriteDACL": {
        "user": "Can modify DACL to grant self GenericAll, then reset password",
        "group": "Can modify DACL to grant self write membership, then add self",
        "computer": "Can modify DACL to grant self full control on machine account",
        "organizationalUnit": "Can modify DACL to gain control over OU child objects",
    },
    "WriteOwner": {
        "user": "Can take ownership then modify DACL to escalate privileges",
        "group": "Can take ownership of group then modify membership",
        "computer": "Can take ownership then configure delegation abuse",
        "organizationalUnit": "Can take ownership then control OU policies",
    },
    "GenericWrite": {
        "user": "Can write scriptPath for logon script execution or modify SPN for Kerberoasting",
        "group": "Can modify group attributes including membership",
        "computer": "Can write msDS-AllowedToActOnBehalfOfOtherIdentity for RBCD attack",
        "organizationalUnit": "Can modify OU attributes and link GPO",
    },
}


def is_admin_sid(sid: str, domain_sid: str) -> bool:
    if sid in ADMIN_SIDS:
        return True
    for suffix in ADMIN_RID_SUFFIXES:
        if sid == domain_sid + suffix:
            return True
    return False


def parse_sid(raw: bytes) -> str:
    if len(raw) < 8:
        return ""
    revision = raw[0]
    sub_auth_count = raw[1]
    authority = int.from_bytes(raw[2:8], byteorder="big")
    subs = []
    for i in range(sub_auth_count):
        offset = 8 + i * 4
        if offset + 4 > len(raw):
            break
        subs.append(struct.unpack("<I", raw[offset:offset + 4])[0])
    return f"S-{revision}-{authority}-" + "-".join(str(s) for s in subs)


def parse_acl(descriptor_bytes: bytes) -> list:
    aces = []
    if len(descriptor_bytes) < 20:
        return aces
    revision = descriptor_bytes[0]
    control = struct.unpack("<H", descriptor_bytes[2:4])[0]
    dacl_offset = struct.unpack("<I", descriptor_bytes[16:20])[0]
    if dacl_offset == 0 or dacl_offset >= len(descriptor_bytes):
        return aces
    dacl = descriptor_bytes[dacl_offset:]
    if len(dacl) < 8:
        return aces
    acl_size = struct.unpack("<H", dacl[2:4])[0]
    ace_count = struct.unpack("<H", dacl[4:6])[0]
    offset = 8
    for _ in range(ace_count):
        if offset + 4 > len(dacl):
            break
        ace_type = dacl[offset]
        ace_flags = dacl[offset + 1]
        ace_size = struct.unpack("<H", dacl[offset + 2:offset + 4])[0]
        if ace_size < 4 or offset + ace_size > len(dacl):
            break
        if ace_type in (0x00, 0x05):
            if offset + 8 <= len(dacl):
                access_mask = struct.unpack("<I", dacl[offset + 4:offset + 8])[0]
                sid_offset = offset + 8
                if ace_type == 0x05:
                    sid_offset = offset + 8 + 32
                if sid_offset < offset + ace_size:
                    sid_bytes = dacl[sid_offset:offset + ace_size]
                    sid_str = parse_sid(sid_bytes)
                    matched_perms = []
                    for perm_name, mask_val in DANGEROUS_MASKS.items():
                        if access_mask & mask_val:
                            matched_perms.append(perm_name)
                    if matched_perms:
                        aces.append({
                            "ace_type": "ACCESS_ALLOWED" if ace_type in (0x00, 0x05) else "OTHER",
                            "access_mask": f"0x{access_mask:08x}",
                            "trustee_sid": sid_str,
                            "permissions": matched_perms,
                        })
        offset += ace_size
    return aces


def resolve_sid(conn: Connection, base_dn: str, sid: str) -> str:
    try:
        conn.search(base_dn, f"(objectSid={sid})", attributes=["sAMAccountName", "cn"])
        if conn.entries:
            entry = conn.entries[0]
            return str(entry.sAMAccountName) if hasattr(entry, "sAMAccountName") else str(entry.cn)
    except Exception:
        pass
    return sid


def get_domain_sid(conn: Connection, base_dn: str) -> str:
    conn.search(base_dn, "(objectClass=domain)", attributes=["objectSid"])
    if conn.entries:
        raw = conn.entries[0].objectSid.raw_values[0]
        return parse_sid(raw)
    return ""


def analyze_acls(dc_ip: str, domain: str, username: str, password: str,
                 target_ou: str) -> dict:
    server = Server(dc_ip, get_info=ALL, use_ssl=False)
    domain_parts = domain.split(".")
    base_dn = ",".join(f"DC={p}" for p in domain_parts)
    search_base = target_ou if target_ou else base_dn
    ntlm_user = f"{domain}\\{username}"

    conn = Connection(server, user=ntlm_user, password=password,
                      authentication=NTLM, auto_bind=True)
    domain_sid = get_domain_sid(conn, base_dn)

    conn.search(
        search_base,
        "(|(objectClass=user)(objectClass=group)(objectClass=computer)(objectClass=organizationalUnit))",
        search_scope=SUBTREE,
        attributes=["distinguishedName", "sAMAccountName", "objectClass", "nTSecurityDescriptor"],
    )

    findings = []
    objects_scanned = 0
    sid_cache = {}

    for entry in conn.entries:
        objects_scanned += 1
        dn = str(entry.distinguishedName)
        obj_classes = [str(c) for c in entry.objectClass.values] if hasattr(entry, "objectClass") else []
        obj_type = "unknown"
        for oc in obj_classes:
            if oc.lower() in ("user", "group", "computer", "organizationalunit"):
                obj_type = oc.lower()
                break

        if not hasattr(entry, "nTSecurityDescriptor"):
            continue
        raw_sd = entry.nTSecurityDescriptor.raw_values
        if not raw_sd:
            continue
        sd_bytes = raw_sd[0]
        aces = parse_acl(sd_bytes)

        for ace in aces:
            trustee_sid = ace["trustee_sid"]
            if is_admin_sid(trustee_sid, domain_sid):
                continue
            if trustee_sid not in sid_cache:
                sid_cache[trustee_sid] = resolve_sid(conn, base_dn, trustee_sid)
            trustee_name = sid_cache[trustee_sid]

            for perm in ace["permissions"]:
                if perm in ("Delete", "DeleteChild", "Self", "WriteProperty", "ExtendedRight"):
                    severity = "medium"
                else:
                    severity = "critical"
                attack = ATTACK_PATHS.get(perm, {}).get(obj_type,
                         f"{perm} on {obj_type} may allow privilege escalation")
                findings.append({
                    "severity": severity,
                    "target_object": dn,
                    "target_type": obj_type,
                    "trustee": trustee_name,
                    "trustee_sid": trustee_sid,
                    "permission": perm,
                    "access_mask": ace["access_mask"],
                    "ace_type": ace["ace_type"],
                    "attack_path": attack,
                    "remediation": f"Remove {perm} ACE for {trustee_name} on {dn}",
                })

    conn.unbind()
    findings.sort(key=lambda f: 0 if f["severity"] == "critical" else 1)
    return {
        "domain": domain,
        "domain_sid": domain_sid,
        "search_base": search_base,
        "objects_scanned": objects_scanned,
        "dangerous_aces_found": len(findings),
        "findings": findings,
    }


def main():
    parser = argparse.ArgumentParser(description="Active Directory ACL Abuse Analyzer")
    parser.add_argument("--dc-ip", required=True, help="Domain Controller IP address")
    parser.add_argument("--domain", required=True, help="AD domain name (e.g., corp.example.com)")
    parser.add_argument("--username", required=True, help="Domain username for LDAP bind")
    parser.add_argument("--password", required=True, help="Domain user password")
    parser.add_argument("--target-ou", default=None,
                        help="Target OU distinguished name to scope the search")
    parser.add_argument("--output", default=None, help="Output JSON file path")
    args = parser.parse_args()

    result = analyze_acls(args.dc_ip, args.domain, args.username,
                          args.password, args.target_ou)
    report = json.dumps(result, indent=2)
    if args.output:
        with open(args.output, "w") as f:
            f.write(report)
    print(report)


if __name__ == "__main__":
    main()

===== END FILE =====
