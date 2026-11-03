

===== FILE: skills/analyzing-sbom-for-supply-chain-vulnerabilities/SKILL.md =====
---
name: analyzing-sbom-for-supply-chain-vulnerabilities
description: 'Parses Software Bill of Materials (SBOM) in CycloneDX and SPDX JSON
  formats to identify supply chain vulnerabilities by correlating components against
  the NVD CVE database via the NVD 2.0 API. Builds dependency graphs, calculates risk
  scores, identifies transitive vulnerability paths, and generates compliance reports.
  Activates for requests involving SBOM analysis, software composition analysis, supply
  chain security assessment, dependency vulnerability scanning, CycloneDX/SPDX parsing,
  or CVE correlation.

  '
domain: cybersecurity
subdomain: supply-chain-security
tags:
- SBOM
- CycloneDX
- SPDX
- NVD
- CVE
- supply-chain
- dependency-analysis
- syft
- grype
version: 1.0.0
author: mukul975
license: Apache-2.0
atlas_techniques:
- AML.T0010
nist_ai_rmf:
- GOVERN-5.2
- MAP-1.6
- MANAGE-2.2
- GOVERN-1.1
- GOVERN-4.2
nist_csf:
- GV.SC-01
- GV.SC-03
- GV.SC-06
- GV.SC-07
mitre_attack:
- T1195.001
- T1195.002
- T1554
- T1190
---

# Analyzing SBOM for Supply Chain Vulnerabilities

## When to Use

- A new regulatory requirement (EO 14028, EU CRA) mandates SBOM analysis for software deliveries
- Security team needs to assess third-party risk by scanning vendor-provided SBOMs
- CI/CD pipeline requires automated vulnerability checks against generated SBOMs
- Incident response needs to determine if a newly disclosed CVE affects deployed software
- Procurement team requires supply chain risk assessment for a software acquisition

**Do not use** for runtime vulnerability scanning of live systems; use container scanning tools (Trivy, Grype CLI) or host-based vulnerability scanners (Nessus, Qualys) instead.

## Prerequisites

- SBOM file in CycloneDX JSON (v1.4+) or SPDX JSON (v2.3+) format
- Python 3.9+ with requests, networkx, and packaging libraries installed
- NVD API key (free, from https://nvd.nist.gov/developers/request-an-api-key) for higher rate limits
- Network access to NVD API (https://services.nvd.nist.gov/rest/json/cves/2.0)
- Optionally: syft for SBOM generation, grype for cross-validation

## Workflow

### Step 1: Generate SBOM (if not provided)

Use syft to create an SBOM from a container image or project directory:

```bash
# Generate CycloneDX JSON from a container image
syft alpine:latest -o cyclonedx-json > sbom-cyclonedx.json

# Generate SPDX JSON from a project directory
syft dir:/path/to/project -o spdx-json > sbom-spdx.json

# Generate from a running container
syft docker:my-app-container -o cyclonedx-json > sbom.json
```

Syft supports over 30 package ecosystems including npm, PyPI, Maven, Go modules, apt, apk, and RPM. The generated SBOM includes package names, versions, licenses, CPE identifiers, and PURL (Package URL) references.

### Step 2: Parse SBOM and Extract Components

Parse the SBOM to extract all software components with their identifiers:

**CycloneDX JSON Structure:**
```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "components": [
    {
      "type": "library",
      "name": "lodash",
      "version": "4.17.20",
      "purl": "pkg:npm/lodash@4.17.20",
      "cpe": "cpe:2.3:a:lodash:lodash:4.17.20:*:*:*:*:*:*:*",
      "licenses": [{"license": {"id": "MIT"}}]
    }
  ],
  "dependencies": [
    {"ref": "pkg:npm/express@4.18.2", "dependsOn": ["pkg:npm/lodash@4.17.20"]}
  ]
}
```

**SPDX JSON Structure:**
```json
{
  "spdxVersion": "SPDX-2.3",
  "packages": [
    {
      "name": "lodash",
      "versionInfo": "4.17.20",
      "externalRefs": [
        {"referenceType": "purl", "referenceLocator": "pkg:npm/lodash@4.17.20"},
        {"referenceType": "cpe23Type", "referenceLocator": "cpe:2.3:a:lodash:lodash:4.17.20:*:*:*:*:*:*:*"}
      ],
      "licenseConcluded": "MIT"
    }
  ],
  "relationships": [
    {"spdxElementId": "SPDXRef-express", "relatedSpdxElement": "SPDXRef-lodash",
     "relationshipType": "DEPENDS_ON"}
  ]
}
```

### Step 3: Correlate Components with NVD CVE Database

Query the NVD 2.0 API to find known vulnerabilities for each component:

```python
import requests

NVD_API = "https://services.nvd.nist.gov/rest/json/cves/2.0"

def search_cves_by_cpe(cpe_name, api_key=None):
    params = {"cpeName": cpe_name, "resultsPerPage": 50}
    headers = {"apiKey": api_key} if api_key else {}
    resp = requests.get(NVD_API, params=params, headers=headers, timeout=30)
    resp.raise_for_status()
    return resp.json().get("vulnerabilities", [])

def search_cves_by_keyword(keyword, version=None, api_key=None):
    params = {"keywordSearch": keyword, "resultsPerPage": 50}
    headers = {"apiKey": api_key} if api_key else {}
    resp = requests.get(NVD_API, params=params, headers=headers, timeout=30)
    resp.raise_for_status()
    return resp.json().get("vulnerabilities", [])
```

The NVD API supports searching by CPE name (most precise), keyword, CVE ID, and date ranges. Rate limits: 5 requests/30 seconds without API key, 50 requests/30 seconds with key.

### Step 4: Build Dependency Graph and Identify Transitive Risks

Construct a directed graph of dependencies to trace vulnerability propagation:

```python
import networkx as nx

def build_dependency_graph(sbom):
    G = nx.DiGraph()
    # Add nodes for each component
    for comp in sbom["components"]:
        G.add_node(comp["purl"], name=comp["name"], version=comp["version"])
    # Add edges from dependency relationships
    for dep in sbom.get("dependencies", []):
        for child in dep.get("dependsOn", []):
            G.add_edge(dep["ref"], child)
    return G
```

Transitive dependency analysis identifies components that are not directly included but are pulled in through dependency chains. A vulnerability in a deeply nested transitive dependency (e.g., 4 levels deep) still represents risk but may be harder to remediate.

Key graph metrics for risk assessment:
- **In-degree**: How many components depend on this one (high in-degree = high blast radius)
- **Shortest path to root**: Distance from application entry point (closer = more exploitable)
- **Betweenness centrality**: Components that sit on many dependency paths (bottleneck risk)

### Step 5: Calculate Risk Scores

Aggregate vulnerability data into component and overall risk scores:

```
Risk Score Calculation:
━━━━━━━━━━━━━━━━━━━━━━
Component Risk = max(CVSS scores of all CVEs affecting the component)

Weighted Risk = Component Risk * Dependency Factor
  where Dependency Factor = 1.0 + (0.1 * in_degree)
  (more dependents = higher organizational impact)

Overall SBOM Risk = weighted average of all component risks
  weighted by dependency centrality

Risk Levels:
  CRITICAL: CVSS >= 9.0 or known exploited (CISA KEV)
  HIGH:     CVSS >= 7.0
  MEDIUM:   CVSS >= 4.0
  LOW:      CVSS < 4.0
```

### Step 6: Cross-Validate with Grype

Use grype to independently scan the SBOM and compare findings:

```bash
# Scan CycloneDX SBOM with grype
grype sbom:sbom-cyclonedx.json -o json > grype-results.json

# Scan SPDX SBOM
grype sbom:sbom-spdx.json -o table

# Filter by severity
grype sbom:sbom-cyclonedx.json --only-fixed --fail-on critical
```

Grype pulls vulnerability data from NVD, GitHub Security Advisories, Alpine SecDB, Red Hat, Debian, Ubuntu, Amazon Linux, and Oracle security databases, providing broader coverage than NVD alone.

### Step 7: Generate Compliance Report

Produce a structured report suitable for regulatory compliance:

```
SBOM VULNERABILITY ANALYSIS REPORT
====================================
SBOM File:         app-sbom-cyclonedx.json
Format:            CycloneDX v1.5
Analysis Date:     2026-03-19
Total Components:  247
Total Dependencies: 1,842 (direct: 34, transitive: 213)

VULNERABILITY SUMMARY
  Critical:  3 components / 5 CVEs
  High:      11 components / 18 CVEs
  Medium:    27 components / 41 CVEs
  Low:       8 components / 12 CVEs

CRITICAL FINDINGS
1. lodash@4.17.20
   CVE-2021-23337 (CVSS 7.2) - Command Injection via template
   CVE-2020-28500 (CVSS 5.3) - ReDoS in trimEnd
   Dependents: 14 components (high blast radius)
   Fix: Upgrade to 4.17.21+

2. log4j-core@2.14.1
   CVE-2021-44228 (CVSS 10.0) - Log4Shell RCE [CISA KEV]
   CVE-2021-45046 (CVSS 9.0) - Incomplete fix bypass
   Dependents: 8 components
   Fix: Upgrade to 2.17.1+

DEPENDENCY GRAPH RISKS
  Most depended-on: core-util@1.2.3 (47 dependents)
  Deepest chain: app -> framework -> adapter -> codec -> zlib (5 levels)
  Bottleneck components: 3 components on >50% of dependency paths

LICENSE COMPLIANCE
  Copyleft licenses found: 2 (GPL-3.0 in libxml2, AGPL-3.0 in mongodb-driver)
  Review required for commercial distribution
```

## Key Concepts

| Term | Definition |
|------|------------|
| **SBOM** | Software Bill of Materials; a formal inventory of all components, libraries, and dependencies in a software product |
| **CycloneDX** | OWASP-maintained SBOM standard supporting JSON, XML, and protobuf formats with dependency graph and vulnerability data |
| **SPDX** | Linux Foundation SBOM standard focused on license compliance with support for package, file, and snippet-level detail |
| **PURL** | Package URL; a standardized scheme for identifying software packages across ecosystems (e.g., pkg:npm/lodash@4.17.21) |
| **CPE** | Common Platform Enumeration; NIST naming scheme for IT products used to correlate with NVD CVE data |
| **NVD** | National Vulnerability Database; US government repository of vulnerability data indexed by CVE identifiers |
| **Transitive Dependency** | A dependency not directly declared but pulled in through the dependency chain of direct dependencies |
| **CISA KEV** | CISA Known Exploited Vulnerabilities catalog; CVEs confirmed to be actively exploited in the wild |

## Tools & Systems

- **syft** (Anchore): Open-source SBOM generator supporting 30+ package ecosystems and CycloneDX/SPDX output
- **grype** (Anchore): Vulnerability scanner that accepts SBOMs as input and correlates against multiple advisory databases
- **cyclonedx-python-lib**: Python library for creating, parsing, and validating CycloneDX SBOMs programmatically
- **lib4sbom**: Python library for parsing both SPDX and CycloneDX format SBOMs
- **nvdlib**: Python wrapper for the NVD 2.0 API supporting CVE and CPE queries with rate limit management
- **OWASP Dependency-Track**: Platform for continuous SBOM analysis, vulnerability tracking, and policy enforcement

## Common Scenarios

### Scenario: Assessing Vendor Software After Log4Shell Disclosure

**Context**: After the Log4Shell (CVE-2021-44228) disclosure, the security team needs to determine which vendor-supplied applications contain vulnerable versions of log4j. Several vendors have provided SBOMs per contractual requirements.

**Approach**:
1. Collect all vendor SBOMs (CycloneDX or SPDX JSON format)
2. Parse each SBOM and search for log4j-core components with versions < 2.17.1
3. Query NVD API for the specific CVEs (CVE-2021-44228, CVE-2021-45046, CVE-2021-45105)
4. Build dependency graphs to identify which application components depend on log4j
5. Calculate blast radius: how many services and endpoints are exposed
6. Generate prioritized remediation report sorted by exposure and business criticality
7. Cross-validate findings with grype scan of the same SBOMs

**Pitfalls**:
- Vendor SBOMs may be incomplete, missing shaded/bundled JAR files that embed log4j
- SPDX and CycloneDX version differences may affect parser compatibility
- NVD API rate limits can slow analysis when scanning hundreds of components without an API key
- CPE names in SBOMs may not exactly match NVD entries, requiring fuzzy matching
- Transitive dependencies may include log4j even when it is not a direct dependency

===== END FILE =====


===== FILE: skills/analyzing-sbom-for-supply-chain-vulnerabilities/LICENSE =====

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


===== FILE: skills/analyzing-sbom-for-supply-chain-vulnerabilities/references/api-reference.md =====
# API Reference: SBOM Supply Chain Vulnerability Analysis

## NVD API 2.0 - Vulnerability Lookup

### Base URL
```
https://services.nvd.nist.gov/rest/json/cves/2.0
```

### Authentication
```
Header: apiKey: <your-api-key>
Get free key: https://nvd.nist.gov/developers/request-an-api-key
```

### Rate Limits
| Condition | Limit |
|-----------|-------|
| Without API key | 5 requests per 30 seconds |
| With API key | 50 requests per 30 seconds |

### Search by CPE Name
```bash
GET /rest/json/cves/2.0?cpeName=cpe:2.3:a:apache:log4j:2.14.1:*:*:*:*:*:*:*
```

```python
import requests

resp = requests.get(
    "https://services.nvd.nist.gov/rest/json/cves/2.0",
    params={"cpeName": "cpe:2.3:a:apache:log4j:2.14.1:*:*:*:*:*:*:*"},
    headers={"apiKey": "YOUR_KEY"},
    timeout=30
)
data = resp.json()
for vuln in data.get("vulnerabilities", []):
    cve = vuln["cve"]
    print(f"{cve['id']}: {cve['metrics']}")
```

### Search by Keyword
```bash
GET /rest/json/cves/2.0?keywordSearch=lodash+prototype+pollution
```

### Search by CVE ID
```bash
GET /rest/json/cves/2.0?cveId=CVE-2021-44228
```

### Response Structure
```json
{
  "resultsPerPage": 50,
  "startIndex": 0,
  "totalResults": 3,
  "vulnerabilities": [
    {
      "cve": {
        "id": "CVE-2021-44228",
        "published": "2021-12-10T10:15:00.000",
        "descriptions": [{"lang": "en", "value": "Apache Log4j2 ..."}],
        "metrics": {
          "cvssMetricV31": [{
            "cvssData": {
              "version": "3.1",
              "baseScore": 10.0,
              "baseSeverity": "CRITICAL"
            }
          }]
        },
        "references": [{"url": "https://..."}]
      }
    }
  ]
}
```

## CycloneDX JSON Format (v1.5)

### Minimal Structure
```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "serialNumber": "urn:uuid:...",
  "version": 1,
  "metadata": {
    "timestamp": "2026-03-19T00:00:00Z",
    "tools": [{"name": "syft", "version": "1.0.0"}]
  },
  "components": [],
  "dependencies": []
}
```

### Component Object
```json
{
  "type": "library",
  "name": "express",
  "version": "4.18.2",
  "purl": "pkg:npm/express@4.18.2",
  "cpe": "cpe:2.3:a:expressjs:express:4.18.2:*:*:*:*:node.js:*:*",
  "licenses": [{"license": {"id": "MIT"}}],
  "supplier": {"name": "OpenJS Foundation"}
}
```

### Dependency Graph
```json
{
  "dependencies": [
    {
      "ref": "pkg:npm/express@4.18.2",
      "dependsOn": [
        "pkg:npm/body-parser@1.20.1",
        "pkg:npm/cookie@0.5.0"
      ]
    }
  ]
}
```

## SPDX JSON Format (v2.3)

### Minimal Structure
```json
{
  "spdxVersion": "SPDX-2.3",
  "dataLicense": "CC0-1.0",
  "SPDXID": "SPDXRef-DOCUMENT",
  "name": "my-application",
  "packages": [],
  "relationships": []
}
```

### Package Object
```json
{
  "SPDXID": "SPDXRef-Package-npm-express",
  "name": "express",
  "versionInfo": "4.18.2",
  "downloadLocation": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
  "licenseConcluded": "MIT",
  "licenseDeclared": "MIT",
  "externalRefs": [
    {"referenceType": "purl", "referenceLocator": "pkg:npm/express@4.18.2"},
    {"referenceType": "cpe23Type", "referenceLocator": "cpe:2.3:a:expressjs:express:4.18.2:*:*:*:*:*:*:*"}
  ]
}
```

### Relationship Types
```json
{
  "spdxElementId": "SPDXRef-Package-npm-express",
  "relatedSpdxElement": "SPDXRef-Package-npm-body-parser",
  "relationshipType": "DEPENDS_ON"
}
```

## syft - SBOM Generation

### Installation
```bash
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
```

### Generate CycloneDX SBOM
```bash
syft <source> -o cyclonedx-json > sbom.json

# Sources: container image, directory, file archive
syft alpine:latest -o cyclonedx-json
syft dir:/app -o cyclonedx-json
syft file:archive.tar.gz -o spdx-json
```

### Output Formats
| Format | Flag |
|--------|------|
| CycloneDX JSON | `-o cyclonedx-json` |
| CycloneDX XML | `-o cyclonedx-xml` |
| SPDX JSON | `-o spdx-json` |
| SPDX Tag-Value | `-o spdx-tag-value` |
| Syft JSON | `-o json` (default) |
| Table | `-o table` |

## grype - Vulnerability Scanning

### Installation
```bash
curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin
```

### Scan SBOM for Vulnerabilities
```bash
# Scan CycloneDX SBOM
grype sbom:sbom-cyclonedx.json

# JSON output
grype sbom:sbom.json -o json > grype-results.json

# Filter by severity
grype sbom:sbom.json --only-fixed --fail-on critical

# Table output with severity filter
grype sbom:sbom.json -o table --only-fixed
```

### Grype Vulnerability Sources
- NVD (National Vulnerability Database)
- GitHub Security Advisories (GHSA)
- Alpine SecDB
- Red Hat Enterprise Linux
- Debian Security Tracker
- Ubuntu CVE Tracker
- Amazon Linux ALAS
- Oracle Linux ELSA
- Wolfi SecDB

## Python Libraries

### nvdlib - NVD API Wrapper
```python
import nvdlib

# Search CVEs by CPE
results = nvdlib.searchCVE(cpeName="cpe:2.3:a:apache:log4j:2.14.1:*:*:*:*:*:*:*")
for cve in results:
    print(f"{cve.id}: CVSS {cve.score[1]}")

# Search CVEs by keyword
results = nvdlib.searchCVE(keywordSearch="lodash prototype pollution")
```

### networkx - Dependency Graph
```python
import networkx as nx

G = nx.DiGraph()
G.add_edge("app", "framework")
G.add_edge("framework", "vulnerable-lib")

# Find all paths to a vulnerable component
paths = nx.all_simple_paths(G, "app", "vulnerable-lib")

# Betweenness centrality (bottleneck identification)
centrality = nx.betweenness_centrality(G)

# Longest dependency chain (DAG only)
longest = nx.dag_longest_path(G)
```

## CLI Usage Examples

```bash
# Full SBOM analysis with NVD correlation
python agent.py analyze sbom-cyclonedx.json --api-key YOUR_KEY -o report.json

# Offline analysis (skip NVD queries)
python agent.py analyze sbom.json --skip-nvd -o report.json

# Compare two SBOMs
python agent.py diff old-sbom.json new-sbom.json

# Parse and list components only
python agent.py parse sbom.json -o components.json

# Check license compliance
python agent.py licenses sbom.json
```

===== END FILE =====


===== FILE: skills/analyzing-sbom-for-supply-chain-vulnerabilities/scripts/agent.py =====
#!/usr/bin/env python3
"""SBOM supply chain vulnerability analysis agent.

Parses CycloneDX and SPDX JSON SBOMs, correlates components against the NVD 2.0 API
for known CVEs, builds dependency graphs with networkx, calculates risk scores, and
generates compliance reports.
"""

import os
import sys
import json
import time
import argparse
from pathlib import Path
from datetime import datetime
from collections import defaultdict

try:
    import requests
    HAS_REQUESTS = True
except ImportError:
    HAS_REQUESTS = False

try:
    import networkx as nx
    HAS_NETWORKX = True
except ImportError:
    HAS_NETWORKX = False

try:
    from packaging.version import Version, InvalidVersion
    HAS_PACKAGING = True
except ImportError:
    HAS_PACKAGING = False

# NVD API 2.0 configuration
NVD_CVE_API = "https://services.nvd.nist.gov/rest/json/cves/2.0"
NVD_CPE_API = "https://services.nvd.nist.gov/rest/json/cpes/2.0"
NVD_RATE_LIMIT_NO_KEY = 6.0   # seconds between requests without API key
NVD_RATE_LIMIT_WITH_KEY = 0.6  # seconds between requests with API key
NVD_RESULTS_PER_PAGE = 50

# CVSS severity thresholds
SEVERITY_THRESHOLDS = {
    "CRITICAL": 9.0,
    "HIGH": 7.0,
    "MEDIUM": 4.0,
    "LOW": 0.1,
}


class SBOMComponent:
    """Represents a single software component extracted from an SBOM."""

    def __init__(self, name, version, purl=None, cpe=None, component_type="library",
                 licenses=None, supplier=None):
        self.name = name
        self.version = version
        self.purl = purl
        self.cpe = cpe
        self.component_type = component_type
        self.licenses = licenses or []
        self.supplier = supplier
        self.cves = []
        self.max_cvss = 0.0
        self.risk_level = "NONE"

    def to_dict(self):
        return {
            "name": self.name,
            "version": self.version,
            "purl": self.purl,
            "cpe": self.cpe,
            "type": self.component_type,
            "licenses": self.licenses,
            "cves": self.cves,
            "max_cvss": self.max_cvss,
            "risk_level": self.risk_level,
        }


def detect_sbom_format(sbom_data):
    """Detect whether the SBOM is CycloneDX or SPDX format."""
    if isinstance(sbom_data, dict):
        if sbom_data.get("bomFormat") == "CycloneDX":
            return "cyclonedx"
        if "spdxVersion" in sbom_data:
            return "spdx"
        if "components" in sbom_data and any(
            "purl" in c for c in sbom_data.get("components", [])
        ):
            return "cyclonedx"
        if "packages" in sbom_data:
            return "spdx"
    return "unknown"


def parse_cyclonedx(sbom_data):
    """Parse CycloneDX JSON SBOM and extract components and dependencies."""
    components = []
    dependencies = {}

    spec_version = sbom_data.get("specVersion", "unknown")
    print(f"  Format: CycloneDX v{spec_version}")

    for comp in sbom_data.get("components", []):
        name = comp.get("name", "unknown")
        version = comp.get("version", "unknown")
        purl = comp.get("purl")
        cpe = None

        # Extract CPE from multiple possible locations
        if "cpe" in comp:
            cpe = comp["cpe"]
        for prop in comp.get("properties", []):
            if prop.get("name") == "syft:cpe23" or "cpe" in prop.get("name", "").lower():
                cpe = prop.get("value")
                break

        # Extract licenses
        licenses = []
        for lic_entry in comp.get("licenses", []):
            lic = lic_entry.get("license", {})
            if "id" in lic:
                licenses.append(lic["id"])
            elif "name" in lic:
                licenses.append(lic["name"])

        component = SBOMComponent(
            name=name,
            version=version,
            purl=purl,
            cpe=cpe,
            component_type=comp.get("type", "library"),
            licenses=licenses,
            supplier=comp.get("supplier", {}).get("name"),
        )
        components.append(component)

    # Parse dependency graph
    for dep_entry in sbom_data.get("dependencies", []):
        ref = dep_entry.get("ref", "")
        depends_on = dep_entry.get("dependsOn", [])
        dependencies[ref] = depends_on

    return components, dependencies


def parse_spdx(sbom_data):
    """Parse SPDX JSON SBOM and extract components and dependencies."""
    components = []
    dependencies = {}

    spdx_version = sbom_data.get("spdxVersion", "unknown")
    print(f"  Format: SPDX {spdx_version}")

    spdx_id_to_purl = {}

    for pkg in sbom_data.get("packages", []):
        name = pkg.get("name", "unknown")
        version = pkg.get("versionInfo", "unknown")
        spdx_id = pkg.get("SPDXID", "")
        purl = None
        cpe = None

        for ref in pkg.get("externalRefs", []):
            ref_type = ref.get("referenceType", "")
            locator = ref.get("referenceLocator", "")
            if ref_type == "purl" or "purl" in ref_type.lower():
                purl = locator
            elif ref_type == "cpe23Type" or "cpe" in ref_type.lower():
                cpe = locator

        licenses = []
        concluded = pkg.get("licenseConcluded", "NOASSERTION")
        if concluded and concluded != "NOASSERTION":
            licenses.append(concluded)
        declared = pkg.get("licenseDeclared", "NOASSERTION")
        if declared and declared != "NOASSERTION" and declared not in licenses:
            licenses.append(declared)

        component = SBOMComponent(
            name=name,
            version=version,
            purl=purl,
            cpe=cpe,
            component_type="library",
            licenses=licenses,
            supplier=pkg.get("supplier"),
        )
        components.append(component)
        spdx_id_to_purl[spdx_id] = purl or f"{name}@{version}"

    # Parse relationships
    for rel in sbom_data.get("relationships", []):
        rel_type = rel.get("relationshipType", "")
        if rel_type == "DEPENDS_ON":
            parent_id = rel.get("spdxElementId", "")
            child_id = rel.get("relatedSpdxElement", "")
            parent_ref = spdx_id_to_purl.get(parent_id, parent_id)
            child_ref = spdx_id_to_purl.get(child_id, child_id)
            if parent_ref not in dependencies:
                dependencies[parent_ref] = []
            dependencies[parent_ref].append(child_ref)

    return components, dependencies


def parse_sbom(sbom_path):
    """Load and parse an SBOM file, auto-detecting the format."""
    if not os.path.isfile(sbom_path):
        raise FileNotFoundError(f"SBOM file not found: {sbom_path}")

    with open(sbom_path, "r", encoding="utf-8") as f:
        sbom_data = json.load(f)

    fmt = detect_sbom_format(sbom_data)
    print(f"\n[INFO] Parsing SBOM: {sbom_path}")

    if fmt == "cyclonedx":
        return parse_cyclonedx(sbom_data), fmt
    elif fmt == "spdx":
        return parse_spdx(sbom_data), fmt
    else:
        raise ValueError(
            f"Unrecognized SBOM format. Expected CycloneDX or SPDX JSON. "
            f"Keys found: {list(sbom_data.keys())[:10]}"
        )


def query_nvd_by_cpe(cpe_name, api_key=None):
    """Query NVD 2.0 API for CVEs matching a CPE name."""
    if not HAS_REQUESTS:
        return []

    params = {"cpeName": cpe_name, "resultsPerPage": NVD_RESULTS_PER_PAGE}
    headers = {}
    if api_key:
        headers["apiKey"] = api_key

    try:
        resp = requests.get(NVD_CVE_API, params=params, headers=headers, timeout=30)
        if resp.status_code == 403:
            print(f"    [WARN] NVD API rate limited. Waiting...", file=sys.stderr)
            time.sleep(NVD_RATE_LIMIT_NO_KEY * 2)
            resp = requests.get(NVD_CVE_API, params=params, headers=headers, timeout=30)
        resp.raise_for_status()
        return resp.json().get("vulnerabilities", [])
    except requests.RequestException as e:
        print(f"    [WARN] NVD API error for {cpe_name}: {e}", file=sys.stderr)
        return []


def query_nvd_by_keyword(keyword, api_key=None):
    """Query NVD 2.0 API for CVEs matching a keyword search."""
    if not HAS_REQUESTS:
        return []

    params = {"keywordSearch": keyword, "resultsPerPage": NVD_RESULTS_PER_PAGE}
    headers = {}
    if api_key:
        headers["apiKey"] = api_key

    try:
        resp = requests.get(NVD_CVE_API, params=params, headers=headers, timeout=30)
        resp.raise_for_status()
        return resp.json().get("vulnerabilities", [])
    except requests.RequestException as e:
        print(f"    [WARN] NVD keyword search error for '{keyword}': {e}", file=sys.stderr)
        return []


def extract_cve_info(vuln_entry):
    """Extract structured CVE information from an NVD API response entry."""
    cve_data = vuln_entry.get("cve", {})
    cve_id = cve_data.get("id", "UNKNOWN")

    # Extract CVSS score (prefer v3.1, fallback to v3.0, then v2.0)
    cvss_score = 0.0
    cvss_version = "none"
    metrics = cve_data.get("metrics", {})

    for version_key in ["cvssMetricV31", "cvssMetricV30", "cvssMetricV2"]:
        metric_list = metrics.get(version_key, [])
        if metric_list:
            cvss_data = metric_list[0].get("cvssData", {})
            cvss_score = cvss_data.get("baseScore", 0.0)
            cvss_version = cvss_data.get("version", version_key)
            break

    # Extract description
    descriptions = cve_data.get("descriptions", [])
    description = ""
    for desc in descriptions:
        if desc.get("lang") == "en":
            description = desc.get("value", "")
            break

    # Determine severity
    severity = "LOW"
    for level, threshold in sorted(SEVERITY_THRESHOLDS.items(),
                                    key=lambda x: x[1], reverse=True):
        if cvss_score >= threshold:
            severity = level
            break

    # Check for known exploited (CISA KEV indicator)
    is_kev = False
    for ref in cve_data.get("references", []):
        if "cisa.gov" in ref.get("url", "").lower():
            is_kev = True
            break

    return {
        "cve_id": cve_id,
        "cvss_score": cvss_score,
        "cvss_version": cvss_version,
        "severity": severity,
        "description": description[:300],
        "is_kev": is_kev,
        "published": cve_data.get("published", ""),
    }


def correlate_cves(components, api_key=None, skip_nvd=False):
    """Correlate all SBOM components against NVD for known vulnerabilities."""
    rate_limit = NVD_RATE_LIMIT_WITH_KEY if api_key else NVD_RATE_LIMIT_NO_KEY
    total = len(components)
    vuln_count = 0

    print(f"\n[INFO] Correlating {total} components against NVD CVE database...")
    if not api_key:
        print(f"  [NOTE] No NVD API key. Rate limited to 1 request per {rate_limit}s.")
        print(f"  Get a free key at: https://nvd.nist.gov/developers/request-an-api-key")

    if skip_nvd:
        print(f"  [NOTE] NVD queries skipped (--skip-nvd flag). Using offline mode.")
        return components

    for idx, comp in enumerate(components):
        print(f"  [{idx+1}/{total}] {comp.name}@{comp.version}...", end="", flush=True)

        vulns = []
        # Try CPE-based search first (most precise)
        if comp.cpe:
            vulns = query_nvd_by_cpe(comp.cpe, api_key)

        # Fallback to keyword search if no CPE or no results
        if not vulns:
            keyword = f"{comp.name} {comp.version}"
            vulns = query_nvd_by_keyword(keyword, api_key)

        # Process results
        for v in vulns:
            cve_info = extract_cve_info(v)
            if cve_info["cvss_score"] > 0:
                comp.cves.append(cve_info)
                if cve_info["cvss_score"] > comp.max_cvss:
                    comp.max_cvss = cve_info["cvss_score"]

        # Assign risk level
        if comp.max_cvss >= SEVERITY_THRESHOLDS["CRITICAL"]:
            comp.risk_level = "CRITICAL"
        elif comp.max_cvss >= SEVERITY_THRESHOLDS["HIGH"]:
            comp.risk_level = "HIGH"
        elif comp.max_cvss >= SEVERITY_THRESHOLDS["MEDIUM"]:
            comp.risk_level = "MEDIUM"
        elif comp.max_cvss > 0:
            comp.risk_level = "LOW"

        cve_count = len(comp.cves)
        vuln_count += cve_count
        status = f" {cve_count} CVEs (max CVSS: {comp.max_cvss})" if cve_count else " clean"
        print(status)

        # Rate limiting
        if idx < total - 1:
            time.sleep(rate_limit)

    print(f"\n[INFO] Correlation complete. Found {vuln_count} total CVEs across all components.")
    return components


def build_dependency_graph(components, dependencies):
    """Build a directed dependency graph using networkx."""
    if not HAS_NETWORKX:
        print("[WARN] networkx not installed. Dependency graph analysis skipped.", file=sys.stderr)
        return None

    G = nx.DiGraph()

    # Build lookup for components by purl or name@version
    comp_lookup = {}
    for comp in components:
        ref = comp.purl or f"{comp.name}@{comp.version}"
        G.add_node(ref, name=comp.name, version=comp.version,
                   max_cvss=comp.max_cvss, risk_level=comp.risk_level,
                   cve_count=len(comp.cves))
        comp_lookup[ref] = comp

    # Add edges from dependency relationships
    for parent_ref, children in dependencies.items():
        if parent_ref not in G:
            G.add_node(parent_ref)
        for child_ref in children:
            if child_ref not in G:
                G.add_node(child_ref)
            G.add_edge(parent_ref, child_ref)

    return G


def analyze_dependency_graph(G):
    """Analyze the dependency graph for risk metrics."""
    if G is None or len(G.nodes) == 0:
        return {}

    analysis = {
        "total_nodes": G.number_of_nodes(),
        "total_edges": G.number_of_edges(),
        "is_dag": nx.is_directed_acyclic_graph(G),
    }

    # Find most depended-on components (highest in-degree)
    in_degrees = sorted(G.in_degree(), key=lambda x: x[1], reverse=True)
    analysis["most_depended_on"] = [
        {"ref": node, "dependents": deg, **G.nodes[node]}
        for node, deg in in_degrees[:10] if deg > 0
    ]

    # Find root nodes (no incoming edges - likely the application itself)
    roots = [n for n, d in G.in_degree() if d == 0]
    analysis["root_components"] = len(roots)

    # Find leaf nodes (no outgoing edges - no dependencies)
    leaves = [n for n, d in G.out_degree() if d == 0]
    analysis["leaf_components"] = len(leaves)

    # Calculate longest dependency chain
    if analysis["is_dag"] and len(G.nodes) > 0:
        try:
            longest_path = nx.dag_longest_path(G)
            analysis["deepest_chain_length"] = len(longest_path)
            analysis["deepest_chain"] = longest_path
        except nx.NetworkXError:
            analysis["deepest_chain_length"] = 0

    # Identify vulnerable components with high in-degree (blast radius)
    high_risk_hubs = []
    for node, deg in in_degrees:
        node_data = G.nodes.get(node, {})
        if node_data.get("max_cvss", 0) >= SEVERITY_THRESHOLDS["HIGH"] and deg > 0:
            high_risk_hubs.append({
                "ref": node,
                "dependents": deg,
                "max_cvss": node_data.get("max_cvss", 0),
                "risk_level": node_data.get("risk_level", "UNKNOWN"),
            })
    analysis["high_risk_hubs"] = high_risk_hubs

    # Betweenness centrality for bottleneck identification
    if len(G.nodes) > 1:
        centrality = nx.betweenness_centrality(G)
        top_central = sorted(centrality.items(), key=lambda x: x[1], reverse=True)[:5]
        analysis["bottleneck_components"] = [
            {"ref": node, "centrality": round(cent, 4)} for node, cent in top_central if cent > 0
        ]

    return analysis


def check_license_compliance(components):
    """Check for potentially problematic licenses in SBOM components."""
    copyleft_licenses = {
        "GPL-2.0", "GPL-2.0-only", "GPL-2.0-or-later",
        "GPL-3.0", "GPL-3.0-only", "GPL-3.0-or-later",
        "AGPL-3.0", "AGPL-3.0-only", "AGPL-3.0-or-later",
        "LGPL-2.1", "LGPL-2.1-only", "LGPL-2.1-or-later",
        "LGPL-3.0", "LGPL-3.0-only", "LGPL-3.0-or-later",
        "MPL-2.0", "EUPL-1.2", "CPAL-1.0", "OSL-3.0",
    }

    findings = {
        "copyleft_components": [],
        "unknown_license_components": [],
        "license_distribution": defaultdict(int),
    }

    for comp in components:
        if not comp.licenses or comp.licenses == ["NOASSERTION"]:
            findings["unknown_license_components"].append(
                {"name": comp.name, "version": comp.version}
            )
        for lic in comp.licenses:
            findings["license_distribution"][lic] += 1
            if lic in copyleft_licenses:
                findings["copyleft_components"].append({
                    "name": comp.name,
                    "version": comp.version,
                    "license": lic,
                })

    findings["license_distribution"] = dict(findings["license_distribution"])
    return findings


def generate_report(components, dependencies, graph_analysis, license_info,
                    sbom_path, sbom_format, output_path=None):
    """Generate a comprehensive vulnerability analysis report."""
    # Aggregate statistics
    vuln_components = [c for c in components if c.cves]
    total_cves = sum(len(c.cves) for c in components)
    severity_counts = defaultdict(lambda: {"components": 0, "cves": 0})

    for comp in components:
        if comp.risk_level != "NONE":
            severity_counts[comp.risk_level]["components"] += 1
            severity_counts[comp.risk_level]["cves"] += len(comp.cves)

    report_lines = []
    report_lines.append("=" * 60)
    report_lines.append("SBOM VULNERABILITY ANALYSIS REPORT")
    report_lines.append("=" * 60)
    report_lines.append(f"SBOM File:          {sbom_path}")
    report_lines.append(f"Format:             {sbom_format}")
    report_lines.append(f"Analysis Date:      {datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S UTC')}")
    report_lines.append(f"Total Components:   {len(components)}")

    direct_deps = len(dependencies)
    transitive = len(components) - direct_deps if direct_deps < len(components) else 0
    report_lines.append(f"Dependencies:       {len(dependencies)} direct, ~{transitive} transitive")
    report_lines.append("")

    report_lines.append("VULNERABILITY SUMMARY")
    report_lines.append("-" * 40)
    for level in ["CRITICAL", "HIGH", "MEDIUM", "LOW"]:
        counts = severity_counts.get(level, {"components": 0, "cves": 0})
        report_lines.append(
            f"  {level:10s}: {counts['components']:3d} components / {counts['cves']:3d} CVEs"
        )
    report_lines.append(f"  {'TOTAL':10s}: {len(vuln_components):3d} components / {total_cves:3d} CVEs")
    report_lines.append("")

    # Critical and high findings detail
    critical_high = sorted(
        [c for c in components if c.risk_level in ("CRITICAL", "HIGH")],
        key=lambda c: c.max_cvss, reverse=True
    )

    if critical_high:
        report_lines.append("CRITICAL & HIGH FINDINGS")
        report_lines.append("-" * 40)
        for i, comp in enumerate(critical_high[:20], 1):
            report_lines.append(f"\n  {i}. {comp.name}@{comp.version} [{comp.risk_level}]")
            for cve in sorted(comp.cves, key=lambda c: c["cvss_score"], reverse=True)[:5]:
                kev_flag = " [CISA KEV]" if cve.get("is_kev") else ""
                report_lines.append(
                    f"     {cve['cve_id']} (CVSS {cve['cvss_score']:.1f}){kev_flag}"
                )
                if cve["description"]:
                    desc_short = cve["description"][:120]
                    report_lines.append(f"       {desc_short}...")

    # Dependency graph analysis
    if graph_analysis:
        report_lines.append("")
        report_lines.append("DEPENDENCY GRAPH ANALYSIS")
        report_lines.append("-" * 40)
        report_lines.append(f"  Nodes: {graph_analysis.get('total_nodes', 0)}")
        report_lines.append(f"  Edges: {graph_analysis.get('total_edges', 0)}")
        report_lines.append(f"  DAG: {graph_analysis.get('is_dag', 'N/A')}")
        chain_len = graph_analysis.get("deepest_chain_length", 0)
        if chain_len:
            report_lines.append(f"  Deepest dependency chain: {chain_len} levels")

        hubs = graph_analysis.get("high_risk_hubs", [])
        if hubs:
            report_lines.append(f"\n  HIGH-RISK HUBS (vulnerable + many dependents):")
            for hub in hubs[:5]:
                report_lines.append(
                    f"    {hub['ref']}: {hub['dependents']} dependents, "
                    f"CVSS {hub['max_cvss']:.1f} [{hub['risk_level']}]"
                )

    # License compliance
    if license_info:
        report_lines.append("")
        report_lines.append("LICENSE COMPLIANCE")
        report_lines.append("-" * 40)
        copyleft = license_info.get("copyleft_components", [])
        unknown = license_info.get("unknown_license_components", [])
        report_lines.append(f"  Copyleft licenses found: {len(copyleft)}")
        for cl in copyleft[:10]:
            report_lines.append(f"    {cl['name']}@{cl['version']}: {cl['license']}")
        report_lines.append(f"  Unknown/missing licenses: {len(unknown)}")

    report_text = "\n".join(report_lines)
    print(f"\n{report_text}")

    # Build JSON result
    result = {
        "sbom_file": sbom_path,
        "sbom_format": sbom_format,
        "analysis_timestamp": datetime.utcnow().isoformat(),
        "summary": {
            "total_components": len(components),
            "vulnerable_components": len(vuln_components),
            "total_cves": total_cves,
            "severity_counts": dict(severity_counts),
        },
        "components": [c.to_dict() for c in components],
        "dependency_graph": graph_analysis or {},
        "license_compliance": license_info or {},
    }

    if output_path:
        with open(output_path, "w", encoding="utf-8") as f:
            json.dump(result, f, indent=2, default=str)
        print(f"\n[OK] Full report saved to {output_path}")

    return result


def analyze_sbom(sbom_path, api_key=None, output_path=None, skip_nvd=False):
    """Full SBOM analysis pipeline: parse, correlate CVEs, graph analysis, report."""
    (components, dependencies), sbom_format = parse_sbom(sbom_path)
    print(f"  Components: {len(components)}")
    print(f"  Dependency entries: {len(dependencies)}")

    # Correlate with NVD
    components = correlate_cves(components, api_key=api_key, skip_nvd=skip_nvd)

    # Build and analyze dependency graph
    G = build_dependency_graph(components, dependencies)
    graph_analysis = analyze_dependency_graph(G)

    # License compliance check
    license_info = check_license_compliance(components)

    # Generate report
    result = generate_report(
        components, dependencies, graph_analysis, license_info,
        sbom_path, sbom_format, output_path
    )

    return result


def compare_sboms(sbom_path_old, sbom_path_new, api_key=None):
    """Compare two SBOMs to identify added, removed, and changed components."""
    (comps_old, _), _ = parse_sbom(sbom_path_old)
    (comps_new, _), _ = parse_sbom(sbom_path_new)

    old_set = {f"{c.name}@{c.version}" for c in comps_old}
    new_set = {f"{c.name}@{c.version}" for c in comps_new}
    old_names = {c.name for c in comps_old}
    new_names = {c.name for c in comps_new}

    added = new_set - old_set
    removed = old_set - new_set

    # Version changes: same name, different version
    old_versions = {c.name: c.version for c in comps_old}
    new_versions = {c.name: c.version for c in comps_new}
    version_changes = []
    for name in old_names & new_names:
        if old_versions[name] != new_versions[name]:
            version_changes.append({
                "name": name,
                "old_version": old_versions[name],
                "new_version": new_versions[name],
            })

    print(f"\n{'='*60}")
    print(f"SBOM DIFF REPORT")
    print(f"{'='*60}")
    print(f"Old: {sbom_path_old} ({len(comps_old)} components)")
    print(f"New: {sbom_path_new} ({len(comps_new)} components)")
    print(f"\nAdded:   {len(added)} components")
    for a in sorted(added):
        print(f"  + {a}")
    print(f"\nRemoved: {len(removed)} components")
    for r in sorted(removed):
        print(f"  - {r}")
    print(f"\nVersion Changes: {len(version_changes)}")
    for vc in version_changes:
        print(f"  ~ {vc['name']}: {vc['old_version']} -> {vc['new_version']}")

    return {"added": sorted(added), "removed": sorted(removed),
            "version_changes": version_changes}


def main():
    parser = argparse.ArgumentParser(
        description="SBOM Supply Chain Vulnerability Analysis Agent"
    )
    subparsers = parser.add_subparsers(dest="command", help="Available commands")

    # Analyze SBOM
    analyze_parser = subparsers.add_parser("analyze", help="Analyze an SBOM for vulnerabilities")
    analyze_parser.add_argument("sbom_path", help="Path to SBOM file (CycloneDX or SPDX JSON)")
    analyze_parser.add_argument("--api-key", help="NVD API key for higher rate limits")
    analyze_parser.add_argument("--output", "-o", help="Save full report to JSON file")
    analyze_parser.add_argument("--skip-nvd", action="store_true",
                                help="Skip NVD API queries (offline mode)")

    # Compare two SBOMs
    diff_parser = subparsers.add_parser("diff", help="Compare two SBOMs for changes")
    diff_parser.add_argument("old_sbom", help="Path to old/baseline SBOM")
    diff_parser.add_argument("new_sbom", help="Path to new/current SBOM")
    diff_parser.add_argument("--api-key", help="NVD API key")

    # Parse only (no NVD queries)
    parse_parser = subparsers.add_parser("parse", help="Parse SBOM and list components")
    parse_parser.add_argument("sbom_path", help="Path to SBOM file")
    parse_parser.add_argument("--output", "-o", help="Save component list to JSON")

    # License check
    license_parser = subparsers.add_parser("licenses", help="Check license compliance")
    license_parser.add_argument("sbom_path", help="Path to SBOM file")

    args = parser.parse_args()

    if args.command == "analyze":
        if not HAS_REQUESTS:
            print("[ERROR] requests library required. Install: pip install requests",
                  file=sys.stderr)
            sys.exit(1)
        api_key = args.api_key or os.environ.get("NVD_API_KEY")
        analyze_sbom(args.sbom_path, api_key=api_key, output_path=args.output,
                     skip_nvd=args.skip_nvd)

    elif args.command == "diff":
        compare_sboms(args.old_sbom, args.new_sbom, api_key=args.api_key)

    elif args.command == "parse":
        (components, dependencies), fmt = parse_sbom(args.sbom_path)
        print(f"\n  Total components: {len(components)}")
        for comp in components:
            print(f"    {comp.name}@{comp.version} [{comp.component_type}] "
                  f"licenses={comp.licenses}")
        if args.output:
            data = {"format": fmt, "component_count": len(components),
                    "components": [c.to_dict() for c in components]}
            with open(args.output, "w") as f:
                json.dump(data, f, indent=2)
            print(f"\n[OK] Component list saved to {args.output}")

    elif args.command == "licenses":
        (components, _), _ = parse_sbom(args.sbom_path)
        info = check_license_compliance(components)
        print(f"\nLicense Distribution:")
        for lic, count in sorted(info["license_distribution"].items(),
                                  key=lambda x: x[1], reverse=True):
            print(f"  {lic}: {count}")
        if info["copyleft_components"]:
            print(f"\nCopyleft Components ({len(info['copyleft_components'])}):")
            for cl in info["copyleft_components"]:
                print(f"  {cl['name']}@{cl['version']}: {cl['license']}")
        if info["unknown_license_components"]:
            print(f"\nUnknown License ({len(info['unknown_license_components'])}):")
            for ul in info["unknown_license_components"]:
                print(f"  {ul['name']}@{ul['version']}")

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-dependency-confusion/SKILL.md =====
---
name: detecting-dependency-confusion
description: Detect and prevent dependency confusion (public-over-private package name resolution) in npm, PyPI, and Maven by enumerating claimable internal package names with tools like `confused` and OWASP `dep-scan`, then enforcing source restrictions via `.npmrc`, `pip.conf`/`pyproject.toml`, and Maven `settings.xml`. Use when onboarding a repo to a supply-chain security program, auditing lockfiles/manifests for confusable dependencies, or after an incident that may have leaked internal package names.
domain: cybersecurity
subdomain: supply-chain-security
tags:
- supply-chain-security
- dependency-confusion
- npm
- pypi
- maven
- package-management
- devsecops
- namespace-hijacking
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-09
mitre_attack:
- T1195.001
---
# Detecting Dependency Confusion

> **Legal Notice:** This skill is for authorized security testing, defensive engineering, and educational purposes only. Registering or claiming package namespaces you do not own, or testing build pipelines without written authorization, may be illegal and may violate the terms of service of public registries. Only run namespace-claiming or resolution-testing activities against names and infrastructure you control or are explicitly authorized to assess.

## Overview

Dependency confusion (also called a substitution or namespace-shadowing attack) was popularized by Alex Birsan in 2021 when he forced malicious code into the internal build systems of Apple, Microsoft, PayPal, and dozens of others. The root cause is that many package managers, when configured to resolve from both an internal/private registry and a public one, will prefer whichever copy has the **higher version number** rather than honoring the source. An attacker who learns the name of a private package (`@acme/internal-utils`, `acme-billing-sdk`) can publish a malicious package of the **same name** to the public registry (npmjs.com, PyPI, Maven Central) with a very high version (e.g. `99.0.0`). When the victim's CI/CD runner or a developer machine resolves dependencies, it pulls the attacker's public package, executes its install scripts, and the supply chain is compromised.

This skill covers both halves of the problem: **detection** — enumerating internal package names that are not registered (squatted defensively) on public registries and are therefore claimable, using `confused` and OWASP `dep-scan` — and **prevention** — pinning scopes/namespaces to private registries, registering placeholder packages, and enforcing source restrictions in `.npmrc`, `pip.conf`/`pyproject.toml`, and Maven `settings.xml`. Internal package names leak constantly: in committed lockfiles, sourcemaps, public JS bundles, Docker layers, and error stack traces, so this is treated as an attack-surface management problem, not a one-time check.

## When to Use

- When onboarding a repository or organization to a supply-chain security program and you need to baseline which internal packages are claimable on public registries.
- When CI/CD pipelines resolve dependencies from both private and public registries (mixed/hybrid feeds).
- After any incident where internal package names may have been exposed (leaked source, public bundle, breached repo).
- When auditing `package.json`, `requirements.txt`, `pom.xml`, `composer.json`, or `Gemfile.lock` files for confusable dependencies.
- As a recurring scheduled control to detect newly added internal packages that have not yet been defensively registered.

## Prerequisites

- Go 1.20+ to install `confused`:
  ```bash
  go install github.com/visma-prodsec/confused@latest
  # binary lands in $(go env GOPATH)/bin/confused
  ```
- Python 3.10+ for OWASP dep-scan:
  ```bash
  pip install owasp-depscan
  # or container: docker pull ghcr.io/owasp-dep-scan/dep-scan
  ```
- Node.js + npm (for `.npmrc` and `npm config` remediation) and access to your private registry (Artifactory, Nexus, Azure Artifacts, GitHub Packages, AWS CodeArtifact).
- Read access to the repositories / lockfiles being assessed and write access to your private registry for defensive registration.

## Objectives

- Enumerate every internal dependency declared in project manifests across npm, PyPI, Maven, Composer, and RubyGems.
- Determine which internal names are **not** present on the corresponding public registry and are therefore claimable.
- Distinguish true exposure from false positives (scoped packages, already-mirrored names).
- Apply registry-pinning and scope-restriction controls that make public substitution impossible.
- Defensively register placeholder packages for unclaimed internal names.
- Establish a recurring detection control in CI to catch newly introduced confusable dependencies.

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Relevance |
|--------------|----------------|-----------|
| T1195.001 | Supply Chain Compromise: Compromise Software Dependencies and Development Tools | Core technique — attacker substitutes a malicious public package for an internal dependency. |
| T1195.002 | Supply Chain Compromise: Compromise Software Supply Chain | Broader category covering the compromised build artifacts produced once confusion succeeds. |
| T1059.007 | Command and Scripting Interpreter: JavaScript | npm `preinstall`/`postinstall` lifecycle scripts execute attacker JavaScript on resolution. |
| T1071.001 | Application Layer Protocol: Web Protocols | Substituted package beacons stolen environment/credentials to attacker HTTP(S) endpoint. |

## Workflow

### 1. Inventory manifests across the codebase
Locate every dependency manifest so nothing is missed.
```bash
# Find all supported manifests in a monorepo
find . -type f \( \
  -name package.json -o \
  -name requirements.txt -o \
  -name pom.xml -o \
  -name composer.json -o \
  -name Gemfile.lock \
\) -not -path '*/node_modules/*' -print
```

### 2. Scan npm manifests with confused
`confused` reads the manifest and reports every dependency name **not found** on the public registry — those are candidates for confusion.
```bash
# npm (default language is npm)
confused -l npm package.json

# Treat your known-good scopes as secure to suppress false positives (supports wildcards)
confused -l npm -s '@acme/*,@acme-internal/*' package.json

# Verbose, to see each lookup
confused -l npm -v package.json
```

### 3. Scan PyPI, Maven, Composer, and RubyGems manifests
The `-l` flag selects the ecosystem; each maps to its standard manifest file.
```bash
confused -l pip requirements.txt          # PyPI  -> requirements.txt
confused -l mvn pom.xml                    # Maven -> pom.xml
confused -l composer composer.json        # PHP   -> composer.json
confused -l rubygems Gemfile.lock         # Ruby  -> Gemfile.lock
```

### 4. Cross-check with OWASP dep-scan private-namespace mode
dep-scan confirms confusion exposure for declared private namespaces and folds it into a broader risk audit.
```bash
# Flag private namespaces accidentally claimable on public registries
depscan --src $PWD --reports-dir ./reports \
  --private-ns acme,acme_internal,@acme

# Enable deep package risk audit (npm + pypi): typosquats, takeover risk, etc.
depscan --src $PWD --reports-dir ./reports --risk-audit
```

### 5. Triage candidates and confirm claimability
For each flagged name, verify it is genuinely absent on the public registry (a 404 means claimable).
```bash
# npm: a 404 status means the name is unregistered on the public registry
curl -s -o /dev/null -w "%{http_code}\n" https://registry.npmjs.org/@acme%2finternal-utils

# PyPI: 404 from the JSON API means the project name is free
curl -s -o /dev/null -w "%{http_code}\n" https://pypi.org/pypi/acme-billing-sdk/json
```

### 6. Remediate npm with scope-to-registry pinning
Bind every internal scope to the private registry so a public package of the same name can never be resolved.
```ini
# .npmrc (project root, committed)
@acme:registry=https://artifactory.example.com/api/npm/npm-internal/
//artifactory.example.com/api/npm/npm-internal/:_authToken=${NPM_TOKEN}

# Force the default registry to a single proxy that does NOT merge public + private
registry=https://artifactory.example.com/api/npm/npm-virtual/
```
Verify the resolution source before installing:
```bash
npm config get @acme:registry
npm install --dry-run   # confirm @acme/* resolves from the private host
```

### 7. Remediate PyPI and Maven
Pin Python index resolution and Maven mirroring so public sources cannot shadow internal artifacts.
```toml
# pyproject.toml (PEP 621 / pip >= 23): explicit index pinning
[tool.pip]
index-url = "https://artifactory.example.com/api/pypi/pypi-internal/simple/"
# Do NOT use extra-index-url for internal packages — pip merges and picks highest version.
```
```xml
<!-- ~/.m2/settings.xml: mirror everything through a single virtual repo -->
<mirrors>
  <mirror>
    <id>internal-virtual</id>
    <mirrorOf>*</mirrorOf>
    <url>https://artifactory.example.com/artifactory/maven-virtual</url>
  </mirror>
</mirrors>
```

### 8. Defensively register placeholder packages
For names you cannot fully isolate, claim the public name yourself with an empty, non-functional placeholder so an attacker cannot.
```bash
# npm placeholder claim (scoped, public)
mkdir acme-internal-utils && cd acme-internal-utils
npm init -y
npm pkg set version=0.0.1-placeholder description="Reserved internal name. Do not use."
npm publish --access public
```

### 9. Wire detection into CI
Fail the pipeline if any new confusable dependency appears.
```yaml
# .github/workflows/depconfusion.yml
name: dependency-confusion
on: [push, pull_request]
jobs:
  confused:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version: '1.22' }
      - run: go install github.com/visma-prodsec/confused@latest
      - name: Scan npm manifest
        run: $(go env GOPATH)/bin/confused -l npm -s '@acme/*' package.json
```

### 10. Run the bundled helper for batch triage
Use the included `agent.py` to scan a tree and emit a structured report combining `confused` and live registry probes.
```bash
python scripts/agent.py --path . --ecosystem npm \
  --secure-namespaces '@acme/*,@acme-internal/*' \
  --output report.json
```

## Tools and Resources

| Tool | Purpose | Source |
|------|---------|--------|
| confused | Detect lingering free namespaces for declared dependencies | https://github.com/visma-prodsec/confused |
| ConfusedDotnet | Same check for NuGet/.NET | https://github.com/visma-prodsec/ConfusedDotnet |
| OWASP dep-scan | Risk audit incl. `--private-ns` confusion check | https://github.com/owasp-dep-scan/dep-scan |
| OWASP CI/CD Top 10 | CICD-SEC-03 Dependency Chain Abuse | https://owasp.org/www-project-top-10-ci-cd-security-risks/ |
| Birsan research | Original dependency confusion writeup | https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610 |
| npm scopes docs | Scope-to-registry pinning reference | https://docs.npmjs.com/cli/v10/using-npm/scope |

## Validation Criteria

- [ ] All dependency manifests in the codebase enumerated.
- [ ] `confused` run for every relevant ecosystem with secure namespaces supplied.
- [ ] OWASP dep-scan `--private-ns` run and reconciled with `confused` output.
- [ ] Each flagged name confirmed claimable (404) or dismissed as a false positive.
- [ ] Internal scopes pinned to the private registry in `.npmrc` / `pip` config / Maven mirror.
- [ ] `extra-index-url` and merged virtual feeds reviewed for highest-version pull risk.
- [ ] Placeholder packages registered for names that cannot be isolated.
- [ ] CI job enforces the check on every push/PR.
- [ ] Findings documented with owner and remediation status.

===== END FILE =====


===== FILE: skills/detecting-dependency-confusion/LICENSE =====

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


===== FILE: skills/detecting-dependency-confusion/references/api-reference.md =====
# API and Command Reference

## confused (visma-prodsec/confused)

Install: `go install github.com/visma-prodsec/confused@latest`

Syntax: `confused [-l LANGUAGE] [-s SECURE_NAMESPACES] [-v] MANIFEST`

| Flag | Values / Example | Description |
|------|------------------|-------------|
| `-l` | `npm` (default), `pip`, `mvn`, `composer`, `rubygems` | Selects the package ecosystem / manifest type. |
| `-s` | `'@acme/*,@acme-internal/*'` | Comma-separated known-secure namespaces; supports `*` wildcards. Suppresses false positives. |
| `-v` | (flag) | Verbose output; prints every registry lookup. |

Manifest mapping: `npm`→`package.json`, `pip`→`requirements.txt`, `mvn`→`pom.xml`, `composer`→`composer.json`, `rubygems`→`Gemfile.lock`.

## OWASP dep-scan

Install: `pip install owasp-depscan`

| Argument | Example | Description |
|----------|---------|-------------|
| `--src` | `--src $PWD` | Path to source repo (or container image). |
| `--reports-dir` | `--reports-dir ./reports` | Output directory for JSON/HTML reports. |
| `--private-ns` | `--private-ns acme,@acme` | Comma-separated private namespaces to check for confusion exposure. |
| `--risk-audit` | (flag) | Deep package risk audit (npm/pypi): takeover, typosquat, maintenance risk. |
| `-t` / `--type` | `-t nodejs` | Restrict to a project type. |

## Public registry probe endpoints (claimability check)

| Registry | Endpoint | 404 means |
|----------|----------|-----------|
| npm | `https://registry.npmjs.org/<name>` (URL-encode `/` in scopes as `%2f`) | Name unregistered / claimable. |
| PyPI | `https://pypi.org/pypi/<name>/json` | Project name free. |
| Maven Central | `https://search.maven.org/solrsearch/select?q=g:<group>+AND+a:<artifact>` (empty `response.numFound`) | Coordinate not published. |
| RubyGems | `https://rubygems.org/api/v1/gems/<name>.json` | Gem not published. |

## Remediation config keys

| Ecosystem | File | Key |
|-----------|------|-----|
| npm | `.npmrc` | `@scope:registry=<private-url>`, top-level `registry=` |
| pip | `pyproject.toml` / `pip.conf` | `index-url` (avoid `extra-index-url` for internal pkgs) |
| Maven | `~/.m2/settings.xml` | `<mirror><mirrorOf>*</mirrorOf>` |
| Composer | `composer.json` | `repositories` + `"packagist.org": false` |

===== END FILE =====


===== FILE: skills/detecting-dependency-confusion/references/standards.md =====
# Standards and Framework Mapping

## MITRE ATT&CK

| ID | Name | Rationale |
|----|------|-----------|
| T1195.001 | Supply Chain Compromise: Compromise Software Dependencies and Development Tools | Dependency confusion substitutes a malicious public package for a private dependency, compromising the dev/build toolchain. |
| T1195.002 | Supply Chain Compromise: Compromise Software Supply Chain | Covers the tainted build artifacts shipped downstream once confusion succeeds. |
| T1059.007 | Command and Scripting Interpreter: JavaScript | npm install lifecycle scripts run attacker JS during resolution. |
| T1071.001 | Application Layer Protocol: Web Protocols | Substituted packages exfiltrate stolen secrets over HTTP(S). |

## NIST Cybersecurity Framework 2.0

| ID | Name | Rationale |
|----|------|-----------|
| ID.RA-09 | The authenticity and integrity of hardware and software are assessed prior to acquisition and use | Detecting confusable names and pinning registries validates that resolved packages are the authentic internal artifacts, not public substitutes. |

## Supporting Standards

- **OWASP Top 10 CI/CD Security Risks — CICD-SEC-03: Dependency Chain Abuse.** Dependency confusion is the canonical example of dependency chain abuse; remediation guidance aligns with this control.
- **NIST SP 800-161r1 — Cybersecurity Supply Chain Risk Management Practices.** Provides organizational SCRM controls (C-SCRM) into which namespace governance and registry pinning fit.
- **SLSA (Supply-chain Levels for Software Artifacts).** Source/build provenance requirements reduce the blast radius of a successful substitution.

===== END FILE =====


===== FILE: skills/detecting-dependency-confusion/scripts/agent.py =====
#!/usr/bin/env python3
"""
Dependency confusion detection helper.

Scans a source tree for package manifests, extracts declared dependency names,
and checks each against the corresponding PUBLIC registry. A dependency that
resolves on the public registry while being intended as private is a
substitution candidate; a name that 404s is defensively claimable.

Supports: npm (package.json), PyPI (requirements.txt), RubyGems (Gemfile.lock).
Optionally shells out to `confused` if it is installed for cross-validation.

Authorized defensive / authorized-testing use only.
"""
import argparse
import fnmatch
import json
import re
import shutil
import subprocess
import sys
import urllib.parse
import urllib.request
from pathlib import Path

REGISTRY = {
    "npm": "https://registry.npmjs.org/{name}",
    "pip": "https://pypi.org/pypi/{name}/json",
    "rubygems": "https://rubygems.org/api/v1/gems/{name}.json",
}
MANIFEST = {"npm": "package.json", "pip": "requirements.txt", "rubygems": "Gemfile.lock"}


def http_status(url: str, timeout: int = 15) -> int:
    req = urllib.request.Request(url, headers={"User-Agent": "depconf-agent/1.0"})
    try:
        with urllib.request.urlopen(req, timeout=timeout) as resp:
            return resp.getcode()
    except urllib.error.HTTPError as exc:
        return exc.code
    except Exception as exc:  # noqa: BLE001 - network errors are reported, not fatal
        print(f"[warn] lookup failed for {url}: {exc}", file=sys.stderr)
        return -1


def parse_npm(path: Path) -> list[str]:
    try:
        data = json.loads(path.read_text(encoding="utf-8"))
    except (json.JSONDecodeError, OSError) as exc:
        print(f"[warn] cannot parse {path}: {exc}", file=sys.stderr)
        return []
    names: set[str] = set()
    for key in ("dependencies", "devDependencies", "optionalDependencies", "peerDependencies"):
        names.update((data.get(key) or {}).keys())
    return sorted(names)


def parse_pip(path: Path) -> list[str]:
    names: set[str] = set()
    try:
        for line in path.read_text(encoding="utf-8").splitlines():
            line = line.strip()
            if not line or line.startswith(("#", "-", "git+", "http")):
                continue
            m = re.match(r"^([A-Za-z0-9._-]+)", line)
            if m:
                names.add(m.group(1))
    except OSError as exc:
        print(f"[warn] cannot read {path}: {exc}", file=sys.stderr)
    return sorted(names)


def parse_rubygems(path: Path) -> list[str]:
    names: set[str] = set()
    try:
        for line in path.read_text(encoding="utf-8").splitlines():
            m = re.match(r"^\s{4}([a-z0-9_-]+) \(", line)
            if m:
                names.add(m.group(1))
    except OSError as exc:
        print(f"[warn] cannot read {path}: {exc}", file=sys.stderr)
    return sorted(names)


PARSERS = {"npm": parse_npm, "pip": parse_pip, "rubygems": parse_rubygems}


def encode_name(eco: str, name: str) -> str:
    # npm scoped packages must URL-encode the slash
    if eco == "npm" and name.startswith("@"):
        return urllib.parse.quote(name, safe="@")
    return urllib.parse.quote(name, safe="")


def is_secure(name: str, patterns: list[str]) -> bool:
    return any(fnmatch.fnmatch(name, p) for p in patterns)


def run_confused(eco: str, manifest: Path, secure: str) -> str | None:
    if not shutil.which("confused"):
        return None
    cmd = ["confused", "-l", eco]
    if secure:
        cmd += ["-s", secure]
    cmd.append(str(manifest))
    try:
        out = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        return out.stdout + out.stderr
    except (subprocess.SubprocessError, OSError) as exc:
        print(f"[warn] confused failed: {exc}", file=sys.stderr)
        return None


def main() -> int:
    ap = argparse.ArgumentParser(description="Dependency confusion detector")
    ap.add_argument("--path", default=".", help="Root path to scan")
    ap.add_argument("--ecosystem", choices=list(REGISTRY), required=True)
    ap.add_argument("--secure-namespaces", default="", help="Comma-separated glob patterns")
    ap.add_argument("--output", help="Write JSON report to this file")
    args = ap.parse_args()

    eco = args.ecosystem
    root = Path(args.path)
    if not root.exists():
        print(f"[error] path not found: {root}", file=sys.stderr)
        return 2
    secure = [p.strip() for p in args.secure_namespaces.split(",") if p.strip()]

    manifests = [p for p in root.rglob(MANIFEST[eco]) if "node_modules" not in p.parts]
    if not manifests:
        print(f"[info] no {MANIFEST[eco]} found under {root}")
        return 0

    findings = []
    for manifest in manifests:
        names = PARSERS[eco](manifest)
        for name in names:
            if is_secure(name, secure):
                continue
            status = http_status(REGISTRY[eco].format(name=encode_name(eco, name)))
            verdict = (
                "CLAIMABLE (404 on public registry)" if status == 404
                else "present on public registry" if status == 200
                else f"unknown (HTTP {status})"
            )
            if status != 200:
                findings.append(
                    {"manifest": str(manifest), "name": name, "status": status, "verdict": verdict}
                )
                print(f"[!] {name}  ->  {verdict}  ({manifest})")

        cf = run_confused(eco, manifest, args.secure_namespaces)
        if cf:
            print(f"--- confused output for {manifest} ---\n{cf}")

    report = {"ecosystem": eco, "root": str(root), "findings": findings}
    if args.output:
        Path(args.output).write_text(json.dumps(report, indent=2), encoding="utf-8")
        print(f"[+] report written to {args.output}")
    print(f"[+] {len(findings)} confusion candidate(s) across {len(manifests)} manifest(s)")
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/detecting-malicious-npm-packages/SKILL.md =====
---
name: detecting-malicious-npm-packages
description: Triage npm packages and lockfiles for install-script malware, credential exfiltration, and worming behavior using GuardDog, manual tarball inspection, and dynamic detonation with network/filesystem monitoring. Use when vetting a new dependency, reviewing a package.json/package-lock.json during code review, checking lockfiles against a supply-chain advisory's known-bad versions, or investigating a host suspected of installing a trojanized package.
domain: cybersecurity
subdomain: supply-chain-security
tags:
- supply-chain-security
- npm
- malware-analysis
- guarddog
- install-scripts
- exfiltration
- static-analysis
- threat-detection
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-09
mitre_attack:
- T1195.002
---
# Detecting Malicious npm Packages

> **Legal Notice:** Analyze packages in an isolated, disposable environment. Some malicious packages execute on install (`npm install` runs lifecycle scripts automatically) or on import. Never analyze a suspect package on a workstation with credentials, SSH keys, cloud tokens, or network access to production. This skill is for defensive analysis and authorized incident response only.

## Overview

The npm registry is the largest software package ecosystem in the world and the most heavily targeted by supply-chain attackers. Malicious packages reach victims through typosquatting (`expresss`, `crossenv`), dependency confusion, account/maintainer takeover (the 2025 Shai-Hulud worm and the `event-stream` compromise are canonical examples), and starjacking. The defining danger of npm is that `npm install` automatically runs `preinstall`, `install`, and `postinstall` lifecycle scripts with the developer's full privileges **before any application code is invoked** — so simply installing a package is enough to be compromised. Roughly 2% of npm packages use install scripts, which makes them both common and a powerful malware delivery vehicle.

Typical malicious behaviors are: exfiltrating environment variables, `~/.npmrc` tokens, SSH keys, and cloud credentials to an attacker-controlled URL; opening reverse shells; dropping cryptominers; reading and posting `process.env`; obfuscating payloads with base64/eval; and self-propagating (worming) by stealing the maintainer's npm token and republishing trojanized versions of other packages they own.

This skill provides a repeatable triage workflow centered on **GuardDog** (Datadog's open-source heuristic scanner built on Semgrep + metadata rules), supplemented by manual tarball inspection, lockfile-based compromise checks against known-bad version lists, and dynamic detonation with network and filesystem monitoring. The goal is to decide, quickly and safely, whether a given package or a project's dependency tree contains malicious code.

## When to Use

- Triaging a specific npm package before adding it as a dependency.
- Vetting a full `package.json` / `package-lock.json` during code review or onboarding a third-party library.
- Responding to a supply-chain advisory (e.g., a worm campaign) and needing to check whether your lockfiles pulled a known-bad version.
- Investigating an endpoint or CI runner suspected of having installed a trojanized package.
- Building a pre-install gate in CI/CD that blocks packages exhibiting malicious indicators.

## Prerequisites

- An isolated VM or disposable container with **no production credentials** and snapshot/rollback capability.
- GuardDog:
  ```bash
  pip install guarddog
  # or run via Docker without local install:
  docker pull ghcr.io/datadog/guarddog
  alias guarddog='docker run --rm ghcr.io/datadog/guarddog'
  ```
- Node.js + npm (use `--ignore-scripts` when downloading for analysis).
- `jq`, `tar`, and optionally OSV-Scanner for known-vulnerability/known-malicious cross-checks:
  ```bash
  go install github.com/google/osv-scanner/cmd/osv-scanner@v1
  ```
- For dynamic analysis: a sandbox with egress logging (e.g., `tcpdump`, a DNS sink, or a network namespace).

## Objectives

- Statically scan an npm package (or a whole dependency tree) for malicious heuristics without executing it.
- Identify install-script abuse, environment/credential exfiltration, obfuscation, and silent process execution.
- Cross-check lockfile-pinned versions against known-malicious version lists / OSV.
- Safely detonate a suspect package and observe network and filesystem behavior.
- Extract indicators of compromise (URLs, IPs, hashes) for blocking and threat intel.
- Produce a defensible verdict (benign / suspicious / malicious) with evidence.

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Relevance |
|--------------|----------------|-----------|
| T1195.002 | Supply Chain Compromise: Compromise Software Supply Chain | Core technique — trojanized npm package delivered through the registry. |
| T1059.007 | Command and Scripting Interpreter: JavaScript | Malicious install scripts / module code execute attacker JavaScript. |
| T1552.001 | Unsecured Credentials: Credentials In Files | Packages steal `~/.npmrc`, `.env`, SSH keys, and cloud credential files. |
| T1041 | Exfiltration Over C2 Channel | Stolen secrets posted to attacker HTTP(S) endpoints. |
| T1027 | Obfuscated Files or Information | base64/eval/hex obfuscation hides the payload from review. |

## Workflow

### 1. Download the package without executing it
Fetch the tarball with scripts disabled so nothing runs during acquisition.
```bash
mkdir triage && cd triage
# Resolve the tarball URL and download it (no install, no scripts)
npm pack express@4.18.2            # produces express-4.18.2.tgz
# or for an arbitrary version:
npm view some-pkg@1.2.3 dist.tarball
curl -sL "$(npm view some-pkg@1.2.3 dist.tarball)" -o some-pkg.tgz
tar -xzf some-pkg.tgz              # extracts into ./package
```

### 2. Scan a single package with GuardDog
GuardDog applies metadata + source heuristics and prints which rules matched.
```bash
# Scan the latest published version from the registry
guarddog npm scan express

# Scan a specific version
guarddog npm scan some-pkg --version 1.2.3

# Scan the local tarball / extracted directory you downloaded above
guarddog npm scan ./some-pkg.tgz
guarddog npm scan ./package/
```

### 3. Verify an entire dependency tree
`verify` scans every dependency declared in a manifest — ideal for code review.
```bash
guarddog npm verify /path/to/repo/package.json
```

### 4. Focus on the highest-signal heuristics
Filter to the npm rules most indicative of malware to cut noise during triage.
```bash
guarddog npm scan some-pkg \
  --rules npm-install-script \
  --rules npm-serialize-environment \
  --rules npm-exec-base64 \
  --rules npm-silent-process-execution \
  --rules npm-obfuscation \
  --rules shady-links \
  --rules typosquatting
```

### 5. Emit machine-readable output for pipelines
JSON for tooling, SARIF for GitHub code scanning.
```bash
guarddog npm scan some-pkg --output-format=json   > guarddog.json
guarddog npm verify package.json --output-format=sarif > guarddog.sarif
```

### 6. Manually inspect lifecycle scripts and source
Lifecycle scripts are the first thing to read; obfuscation and outbound URLs are red flags.
```bash
# Show all lifecycle hooks
jq '.scripts' package/package.json

# Hunt for exfiltration / execution primitives in the source
grep -rEn "child_process|exec\(|spawn|eval\(|Buffer\.from\(.*base64|process\.env|https?://" package/ \
  --include='*.js' --include='*.ts' | head -50
```

### 7. Cross-check lockfiles against known-malicious versions
During an active campaign, compare pinned versions to the advisory's bad-version list, and run OSV.
```bash
# Extract resolved name@version pairs from a v3 lockfile
jq -r '.packages | to_entries[] | select(.key|startswith("node_modules/")) | "\(.key|ltrimstr("node_modules/"))@\(.value.version)"' package-lock.json

# OSV-Scanner flags known-vulnerable AND known-malicious (MAL-) advisories
osv-scanner --lockfile=package-lock.json
```

### 8. Detonate safely with monitoring (only if static is inconclusive)
Run the install inside a disposable, network-monitored sandbox.
```bash
# In a throwaway container / VM with egress capture running (tcpdump -w capture.pcap):
npm install ./some-pkg.tgz            # scripts WILL run — sandbox only
# Baseline-diff the filesystem afterwards for writes outside node_modules,
# and inspect capture.pcap for unexpected DNS / HTTP beacons.
```

### 9. Extract and operationalize IOCs
Pull URLs, IPs, and hashes for blocking and intel sharing.
```bash
grep -rhoE "https?://[a-zA-Z0-9./?=_%:-]+" package/ | sort -u > urls.txt
sha256sum some-pkg.tgz package/*.js > hashes.txt
```

### 10. Run the bundled triage helper
`agent.py` orchestrates GuardDog, lifecycle-script inspection, and IOC extraction into one report.
```bash
python scripts/agent.py --package some-pkg --version 1.2.3 --output verdict.json
# or against a local tarball:
python scripts/agent.py --tarball ./some-pkg.tgz --output verdict.json
```

## Tools and Resources

| Tool | Purpose | Source |
|------|---------|--------|
| GuardDog | Heuristic npm/PyPI/Go malware scanner | https://github.com/DataDog/guarddog |
| OSV-Scanner | Known-vulnerable & known-malicious (MAL-) advisory matching | https://github.com/google/osv-scanner |
| OSV malicious DB | Open-source malicious package advisories | https://github.com/ossf/malicious-packages |
| npm lifecycle docs | preinstall/install/postinstall semantics | https://docs.npmjs.com/cli/v10/using-npm/scripts |
| Datadog Security Labs | npm campaign writeups & rules | https://securitylabs.datadoghq.com/ |
| Semgrep | Rule engine GuardDog uses for source heuristics | https://semgrep.dev/ |

## Validation Criteria

- [ ] Package acquired with scripts disabled in an isolated environment.
- [ ] GuardDog `scan` run on the target version with results captured.
- [ ] Full dependency tree run through GuardDog `verify` where applicable.
- [ ] Lifecycle scripts (`preinstall`/`install`/`postinstall`) read and assessed.
- [ ] Lockfile versions cross-checked against OSV / known-bad lists.
- [ ] Dynamic detonation performed in a sandbox if static analysis was inconclusive.
- [ ] IOCs (URLs, IPs, hashes) extracted and recorded.
- [ ] Documented verdict (benign / suspicious / malicious) with supporting evidence.
- [ ] Malicious findings reported to the registry and shared as threat intel.

===== END FILE =====


===== FILE: skills/detecting-malicious-npm-packages/LICENSE =====

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


===== FILE: skills/detecting-malicious-npm-packages/references/api-reference.md =====
# API and Command Reference

## GuardDog (DataDog/guarddog)

Install: `pip install guarddog`  |  Docker: `ghcr.io/datadog/guarddog`

### Subcommands
| Command | Description |
|---------|-------------|
| `guarddog npm scan <pkg>` | Scan latest version from registry. |
| `guarddog npm scan <pkg> --version X.Y.Z` | Scan a specific version. |
| `guarddog npm scan <path.tgz \| dir>` | Scan a local tarball or extracted directory. |
| `guarddog npm verify <package.json>` | Scan every dependency in a manifest. |
| `guarddog pypi scan <pkg>` | Same for PyPI. |
| `guarddog go scan <module>` / `guarddog go verify go.mod` | Go modules. |
| `guarddog rubygems scan <gem>` | RubyGems. |

### Common flags
| Flag | Description |
|------|-------------|
| `--output-format=json` | Machine-readable JSON. |
| `--output-format=sarif` | SARIF for GitHub code scanning. |
| `--rules <rule>` (repeatable) | Run only the named rule(s). |
| `--exclude-rules <rule>` | Exclude the named rule(s). |
| `--log-level debug` | Verbose diagnostics. |

### Key npm heuristics
| Rule | Detects |
|------|---------|
| `npm-install-script` | preinstall/install/postinstall lifecycle scripts. |
| `npm-serialize-environment` | Exfiltration of environment variables. |
| `npm-exec-base64` | eval of base64-decoded payloads. |
| `npm-silent-process-execution` | Silent child-process execution. |
| `npm-obfuscation` | Common obfuscation patterns. |
| `shady-links` | Suspicious URLs in code. |
| `typosquatting` | Name similar to a popular package. |
| `potentially_compromised_email_domain` | Maintainer email on a lapsed domain. |

## OSV-Scanner

| Command | Description |
|---------|-------------|
| `osv-scanner --lockfile=package-lock.json` | Match pinned versions to OSV advisories incl. `MAL-` malicious entries. |
| `osv-scanner -r <dir>` | Recursively scan a directory. |
| `osv-scanner --format json` | JSON output. |

## npm acquisition (no execution)

| Command | Description |
|---------|-------------|
| `npm pack <pkg>@<ver>` | Download tarball without installing. |
| `npm view <pkg>@<ver> dist.tarball` | Print the tarball URL. |
| `npm install --ignore-scripts` | Install while skipping lifecycle scripts. |
| `jq '.scripts' package/package.json` | List lifecycle hooks. |

===== END FILE =====


===== FILE: skills/detecting-malicious-npm-packages/references/standards.md =====
# Standards and Framework Mapping

## MITRE ATT&CK

| ID | Name | Rationale |
|----|------|-----------|
| T1195.002 | Supply Chain Compromise: Compromise Software Supply Chain | Core technique — trojanized package shipped through the npm registry. |
| T1059.007 | Command and Scripting Interpreter: JavaScript | Install scripts and module code run attacker JavaScript on the victim. |
| T1552.001 | Unsecured Credentials: Credentials In Files | Packages harvest `.npmrc`, `.env`, SSH keys, and cloud credential files. |
| T1041 | Exfiltration Over C2 Channel | Stolen data is POSTed to attacker HTTP(S) endpoints. |
| T1027 | Obfuscated Files or Information | base64/eval/hex obfuscation conceals the payload. |

## NIST Cybersecurity Framework 2.0

| ID | Name | Rationale |
|----|------|-----------|
| DE.CM-09 | Computing hardware and software, runtime environments, and their data are monitored to find potentially adverse events | Static + dynamic triage of npm packages and lockfiles is the monitoring control that surfaces malicious dependencies. |

## Supporting Standards

- **OWASP Top 10 CI/CD Security Risks — CICD-SEC-03: Dependency Chain Abuse.** Malicious package ingestion is a primary dependency-chain abuse vector.
- **NIST SP 800-218 (SSDF) — PW.4 / PS.3.** Reuse and verify the integrity of acquired software components; triaging packages satisfies the verification practice.
- **SLSA provenance.** Verifying build provenance reduces the chance of consuming a tampered or republished package.

===== END FILE =====


===== FILE: skills/detecting-malicious-npm-packages/scripts/agent.py =====
#!/usr/bin/env python3
"""
Malicious npm package triage helper.

Orchestrates GuardDog static scanning, lifecycle-script inspection, and IOC
extraction for a single npm package (by name@version) or a local tarball, and
emits a structured verdict report.

WARNING: download with scripts disabled and run only in an isolated, disposable
environment with no production credentials. Defensive / authorized use only.
"""
import argparse
import json
import re
import shutil
import subprocess
import sys
import tarfile
import tempfile
import urllib.request
from pathlib import Path

URL_RE = re.compile(r"https?://[A-Za-z0-9./?=_%:&#-]+")
SUSPECT_RE = re.compile(
    r"child_process|\bexec\s*\(|\bspawn\b|\beval\s*\(|Buffer\.from\([^)]*base64|process\.env",
    re.IGNORECASE,
)
HIGH_SIGNAL_RULES = [
    "npm-install-script",
    "npm-serialize-environment",
    "npm-exec-base64",
    "npm-silent-process-execution",
    "npm-obfuscation",
    "shady-links",
    "typosquatting",
]


def run(cmd: list[str], timeout: int = 600) -> tuple[int, str, str]:
    try:
        p = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
        return p.returncode, p.stdout, p.stderr
    except FileNotFoundError:
        return 127, "", f"command not found: {cmd[0]}"
    except subprocess.SubprocessError as exc:
        return 1, "", f"subprocess error: {exc}"


def guarddog_scan(target: str, version: str | None) -> dict:
    if not shutil.which("guarddog"):
        return {"error": "guarddog not installed (pip install guarddog)"}
    cmd = ["guarddog", "npm", "scan", target, "--output-format=json"]
    if version:
        cmd += ["--version", version]
    for rule in HIGH_SIGNAL_RULES:
        cmd += ["--rules", rule]
    rc, out, err = run(cmd)
    try:
        return {"returncode": rc, "result": json.loads(out)} if out.strip() else {"returncode": rc, "stderr": err}
    except json.JSONDecodeError:
        return {"returncode": rc, "raw": out, "stderr": err}


def download_tarball(name: str, version: str | None, dest: Path) -> Path | None:
    spec = f"{name}@{version}" if version else name
    rc, out, err = run(["npm", "view", spec, "dist.tarball"])
    if rc != 0 or not out.strip():
        print(f"[warn] could not resolve tarball for {spec}: {err}", file=sys.stderr)
        return None
    url = out.strip().splitlines()[-1]
    tgz = dest / "package.tgz"
    try:
        urllib.request.urlretrieve(url, tgz)  # noqa: S310 - registry URL from npm
        return tgz
    except Exception as exc:  # noqa: BLE001
        print(f"[warn] download failed: {exc}", file=sys.stderr)
        return None


def inspect_tarball(tgz: Path, workdir: Path) -> dict:
    findings = {"lifecycle_scripts": {}, "suspicious_lines": [], "urls": []}
    extract = workdir / "package"
    try:
        with tarfile.open(tgz, "r:gz") as tf:
            members = [m for m in tf.getmembers() if not m.name.startswith(("/", ".."))]
            tf.extractall(extract, members=members)  # noqa: S202 - path-checked members
    except (tarfile.TarError, OSError) as exc:
        findings["error"] = f"extract failed: {exc}"
        return findings

    pkg_json = next(extract.rglob("package.json"), None)
    if pkg_json:
        try:
            data = json.loads(pkg_json.read_text(encoding="utf-8"))
            findings["lifecycle_scripts"] = {
                k: v for k, v in (data.get("scripts") or {}).items()
                if k in ("preinstall", "install", "postinstall")
            }
        except (json.JSONDecodeError, OSError):
            pass

    for src in extract.rglob("*.js"):
        try:
            text = src.read_text(encoding="utf-8", errors="ignore")
        except OSError:
            continue
        for m in SUSPECT_RE.finditer(text):
            findings["suspicious_lines"].append({"file": str(src.relative_to(extract)), "match": m.group(0)})
        findings["urls"].extend(URL_RE.findall(text))
    findings["urls"] = sorted(set(findings["urls"]))[:50]
    findings["suspicious_lines"] = findings["suspicious_lines"][:50]
    return findings


def verdict(gd: dict, insp: dict) -> str:
    score = 0
    res = gd.get("result")
    if isinstance(res, dict):
        for v in res.get("results", res).values() if isinstance(res.get("results", res), dict) else []:
            if v:
                score += 2
    if insp.get("lifecycle_scripts"):
        score += 2
    score += min(len(insp.get("suspicious_lines", [])), 5)
    if score >= 6:
        return "MALICIOUS (high confidence)"
    if score >= 2:
        return "SUSPICIOUS (manual review required)"
    return "benign (no strong indicators)"


def main() -> int:
    ap = argparse.ArgumentParser(description="npm malicious package triage")
    ap.add_argument("--package", help="npm package name")
    ap.add_argument("--version", help="specific version")
    ap.add_argument("--tarball", help="path to a local .tgz instead of downloading")
    ap.add_argument("--output", help="write JSON report")
    args = ap.parse_args()

    if not args.package and not args.tarball:
        ap.error("provide --package or --tarball")

    report: dict = {"package": args.package, "version": args.version}
    with tempfile.TemporaryDirectory() as td:
        work = Path(td)
        if args.tarball:
            tgz = Path(args.tarball)
            report["guarddog"] = (
                guarddog_scan(str(tgz), None) if shutil.which("guarddog") else {"error": "no guarddog"}
            )
        else:
            report["guarddog"] = guarddog_scan(args.package, args.version)
            tgz = download_tarball(args.package, args.version, work)

        report["inspection"] = inspect_tarball(tgz, work) if tgz and tgz.exists() else {"error": "no tarball"}

    report["verdict"] = verdict(report["guarddog"], report["inspection"])
    print(f"[+] verdict: {report['verdict']}")
    if report["inspection"].get("lifecycle_scripts"):
        print(f"[!] install scripts present: {report['inspection']['lifecycle_scripts']}")
    if report["inspection"].get("urls"):
        print(f"[i] {len(report['inspection']['urls'])} URL(s) found in source")

    if args.output:
        Path(args.output).write_text(json.dumps(report, indent=2), encoding="utf-8")
        print(f"[+] report written to {args.output}")
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/detecting-typosquatting-packages/SKILL.md =====
---
name: detecting-typosquatting-packages
description: Flag misspelled, brandjacked, and typosquatted package names across npm, PyPI, and crates.io before installation, using edit-distance, keyboard-proximity, and known-target corpus matching with typomania, Microsoft OSSGadget's oss-find-squats, and pypi-scan. Use before adding a new dependency, as a CI/CD gate on pull requests, when reviewing an AI-generated ("slopsquatting") dependency list, or when auditing a lockfile diff for a swapped package.
domain: cybersecurity
subdomain: supply-chain-security
tags:
- supply-chain-security
- typosquatting
- package-registry
- npm
- pypi
- typomania
- ossgadget
- dependency-screening
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-09
mitre_attack:
- T1195.002
---
# Detecting Typosquatting Packages

> **Authorized Use Only:** This skill is for defensive software-supply-chain security, package screening, and authorized research. Use the corpus-matching and registry-query techniques here only against registries you are permitted to query at scale and packages you intend to evaluate for your own organization. Mass automated registry scraping may violate registry terms of service.

## Overview

Typosquatting is a software-supply-chain attack (MITRE ATT&CK **T1195.002 — Supply Chain Compromise: Compromise Software Supply Chain**) in which an adversary publishes a malicious package whose name is a near-miss of a popular legitimate package — `reqeusts` for `requests`, `python-sqlite` for `sqlite3`, `crossenv` for `cross-env`. A developer who fat-fingers the name, copies a name from a poisoned tutorial, or trusts an AI-generated dependency list (the "slopsquatting" variant, where models hallucinate package names attackers then register) installs the squat instead. Because most ecosystems execute install-time scripts (`postinstall` in npm, `setup.py`/build hooks in PyPI), the payload runs immediately with the developer's privileges.

This skill covers proactive, pre-installation detection: screening a candidate package name against a corpus of popular/known-good names using the same name-mutation primitives attackers use, then triaging high-risk matches. The canonical open-source detector is **typomania** (Rust Foundation), a Rust port of the academic **typogard** tool ("Defending Against Package Typosquatting", University of Kansas); typomania powers crates.io's live typosquatting checks. Cross-ecosystem coverage comes from **Microsoft OSSGadget's `oss-find-squats`** and, for PyPI, **IQTLabs `pypi-scan`**. The ecosyste-ms **typosquatting-dataset** provides a curated ground-truth corpus of known squats mapped to their legitimate targets.

Sources: Rust Foundation `typomania` (https://github.com/rustfoundation/typomania), Microsoft OSSGadget, ecosyste-ms typosquatting-dataset, and OWASP CI/CD / SLSA supply-chain guidance.

## When to Use

- Before adding a new dependency to `package.json`, `requirements.txt`, `pyproject.toml`, or `Cargo.toml`
- As a CI/CD gate that screens every newly introduced dependency name in a pull request
- When triaging an AI-generated or tutorial-sourced dependency list ("slopsquatting" review)
- During security review of a lockfile diff to catch a swapped or newly-pinned squat
- When building a registry-side or proxy-side guardrail that blocks installs of suspected squats

## Prerequisites

- Rust toolchain (`cargo`) to build/use typomania, or a prebuilt OSSGadget release
- Python 3.8+ for pypi-scan and the helper script in this skill
- Network access to the target registry's public API (npmjs.org, pypi.org, crates.io)
- A corpus of "popular" package names for the ecosystem (download counts or a top-N list)

Install the tooling:

```bash
# typomania (library + example harness) — Rust Foundation
git clone https://github.com/rustfoundation/typomania
cd typomania
cargo build --release
cargo run --example registry   # demonstrates the Harness against a fake registry

# OSSGadget (Microsoft) — cross-ecosystem squat finder
# Download a release binary, then:
oss-find-squats pkg:npm/requests          # purl syntax
oss-find-squats pkg:pypi/reqeusts

# pypi-scan (IQTLabs) — PyPI typosquat enumerator
git clone https://github.com/IQTLabs/pypi-scan
cd pypi-scan
pip install -r requirements.txt

# ecosyste-ms ground-truth dataset of known squats
git clone https://github.com/ecosyste-ms/typosquatting-dataset
```

## Objectives

- Generate the candidate squat set for a given legitimate name using the standard mutation primitives
- Screen a candidate package name against a popular-name corpus and flag near-misses
- Enrich each suspected squat with registry metadata (age, downloads, maintainer, install scripts)
- Score and triage findings to suppress false positives (legitimate forks, scoped packages)
- Wire the check into CI/CD as a blocking gate on new dependencies

## MITRE ATT&CK Mapping

| Technique ID | Name | Tactic |
|--------------|------|--------|
| T1195.002 | Supply Chain Compromise: Compromise Software Supply Chain | Initial Access |

A typosquatted dependency is the delivery vehicle for T1195.002: the attacker compromises the victim's supply chain not by breaching a real package but by getting a malicious look-alike installed in its place. Related downstream behavior frequently includes T1059 (Command and Scripting Interpreter) via install hooks and T1041/T1567 (Exfiltration) of tokens and environment variables.

## Workflow

### Step 1: Build the popular-name corpus
The detector needs a reference set of legitimate names to compare against. Pull top packages by download count for the ecosystem.

```bash
# PyPI: top packages dataset (Hugo van Kemenade's top-pypi-packages)
curl -s https://hugovk.github.io/top-pypi-packages/top-pypi-packages.min.json \
  -o top-pypi-packages.json

# npm: query the registry's most-depended-upon search
curl -s 'https://registry.npmjs.org/-/v1/search?text=not:unstable&popularity=1.0&size=250' \
  -o npm-top.json

# crates.io: top crates by downloads
curl -s 'https://crates.io/api/v1/crates?sort=downloads&per_page=100' \
  -H 'User-Agent: typosquat-screen (security@example.com)' -o crates-top.json
```

### Step 2: Generate candidate squats with the standard mutation primitives
typomania/typogard apply a fixed set of name transformations that mirror real attacker behavior. Reproduce them to understand what a screen must catch:

```text
1. Repeated characters     requests  -> reqquests
2. Omitted characters      requests  -> requsts
3. Swapped/transposed      requests  -> reqeusts
4. Swapped words           python-dateutil -> dateutil-python
5. Common typos (1-edit)   requests  -> rewuests   (keyboard adjacency)
6. Homophones / vowel swap requests  -> requeasts
7. Version / suffix tricks lodash    -> lodashs, lodash-js
8. Delimiter swaps         cross-env -> crossenv, cross_env
9. Scope confusion (npm)   @types/node -> types-node
```

Run typomania's harness, which implements these as reusable primitives behind the `Corpus` and `Harness` traits:

```bash
# In the typomania checkout: feed your popular corpus, then check a name.
# The Harness::check method (parallelized via rayon) compares the candidate
# against every corpus entry using the squatting primitives.
cargo run --example registry -- --corpus top-pypi-packages.json --name reqeusts
```

### Step 3: Screen with OSSGadget oss-find-squats
OSSGadget queries the live registry, generates mutations of the supplied package, and reports which mutated names actually exist as published packages.

```bash
# Find names that squat on a legitimate package, checking which exist in the registry
oss-find-squats pkg:npm/lodash
oss-find-squats pkg:pypi/requests

# Reverse direction: given a SUSPECT name, find the legitimate package it mimics
oss-find-squats --quiet pkg:npm/loadsh
```

### Step 4: Enumerate PyPI squats with pypi-scan
```bash
cd pypi-scan
# Find candidate typosquats of a specific package
python pypi_scan.py -p requests

# Scan the top-N most-downloaded PyPI packages for existing squats
python pypi_scan.py -n 50
```

### Step 5: Enrich suspected squats with registry metadata
A near-miss name is only suspicious if it is also young, low-download, or ships install scripts. Pull metadata to triage:

```bash
# npm package metadata: creation time, maintainers, scripts
curl -s https://registry.npmjs.org/loadsh | \
  python -c 'import sys,json;d=json.load(sys.stdin);v=d["dist-tags"]["latest"];print("created:",d["time"]["created"]);print("scripts:",d["versions"][v].get("scripts",{}))'

# npm download counts (last week)
curl -s https://api.npmjs.org/downloads/point/last-week/loadsh

# PyPI JSON API: release history and author
curl -s https://pypi.org/pypi/reqeusts/json | \
  python -c 'import sys,json;d=json.load(sys.stdin);i=d["info"];print(i["name"],i["author"],i["home_page"]);print("releases:",list(d["releases"].keys()))'
```

### Step 6: Score, triage, and confirm
Combine signals into a risk score. High risk = small edit distance to a popular name AND (package age < 90 days OR downloads < 1000 OR presence of `postinstall`/`preinstall`/setup-time network calls). Cross-check against the ecosyste-ms known-squats dataset:

```bash
# Is this name a documented squat?
grep -i 'loadsh' typosquatting-dataset/data/*.csv
```

Confirm malicious intent by inspecting (in a sandbox/VM only) the install scripts and source tarball — never `npm install` or `pip install` a suspect on your workstation.

### Step 7: Enforce in CI/CD
Add a blocking gate that screens every new dependency name introduced by a PR:

```yaml
# .github/workflows/typosquat-gate.yml
name: typosquat-gate
on: [pull_request]
jobs:
  screen:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Screen new dependencies
        run: |
          git diff origin/${{ github.base_ref }}...HEAD -- package.json requirements.txt \
            | grep '^+' | python scripts/agent.py screen --ecosystem npm --corpus top.json --stdin
```

## Tools and Resources

| Tool | Purpose | Source |
|------|---------|--------|
| typomania | Rust typosquat-detection library (powers crates.io) | https://github.com/rustfoundation/typomania |
| OSSGadget `oss-find-squats` | Cross-ecosystem squat finder (purl) | https://github.com/microsoft/OSSGadget |
| pypi-scan | PyPI typosquat enumerator | https://github.com/IQTLabs/pypi-scan |
| ecosyste-ms typosquatting-dataset | Curated known-squat ground truth | https://github.com/ecosyste-ms/typosquatting-dataset |
| top-pypi-packages | PyPI popular-name corpus | https://hugovk.github.io/top-pypi-packages/ |
| OWASP CI/CD Security Top 10 | Supply-chain control guidance | https://owasp.org/www-project-top-10-ci-cd-security-risks/ |

## Mutation Primitives Reference

| Primitive | Legit | Squat | Why it works |
|-----------|-------|-------|--------------|
| Transposition | requests | reqeusts | Common typing slip |
| Omission | requests | requsts | Dropped character |
| Repetition | requests | reqquests | Stuck key |
| Delimiter swap | cross-env | crossenv | Hyphen vs none ambiguity |
| Word order | python-dateutil | dateutil-python | Reordered compound name |
| Homoglyph/vowel | requests | requeasts | Visual/phonetic similarity |
| Suffix/scope | lodash | lodash-js, loadsh | Plausible "official" variant |

## Validation Criteria

- [ ] Popular-name corpus downloaded for each in-scope ecosystem
- [ ] Mutation primitives reproduced and a known squat (e.g., `loadsh`) is correctly flagged
- [ ] typomania / OSSGadget / pypi-scan run against at least one real package
- [ ] Suspected squats enriched with age, download, and install-script metadata
- [ ] Findings cross-checked against the ecosyste-ms known-squats dataset
- [ ] Risk scoring suppresses obvious false positives (legit scoped/forked packages)
- [ ] CI/CD gate screens new dependency names on every pull request
- [ ] No suspect package installed outside a disposable sandbox

===== END FILE =====


===== FILE: skills/detecting-typosquatting-packages/LICENSE =====

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


===== FILE: skills/detecting-typosquatting-packages/references/api-reference.md =====
# API and Command Reference - Typosquatting Detection

## typomania (Rust library)

| Item | Description |
|------|-------------|
| `Harness` | Primary struct; `Harness::check(name)` compares a candidate against the corpus |
| `Corpus` trait | Implement to provide the popular-name reference set for any registry |
| `Package` trait | Implement to expose a package's name/metadata to the harness |
| `rayon` feature | Enabled by default; parallelizes `Harness::check` across many packages |
| `cargo run --example registry` | Runs the bundled example against a fake registry |

## OSSGadget oss-find-squats

| Command | Purpose |
|---------|---------|
| `oss-find-squats pkg:npm/<name>` | Generate squat candidates of an npm package and report which exist |
| `oss-find-squats pkg:pypi/<name>` | Same for PyPI |
| `oss-find-squats pkg:cargo/<name>` | Same for crates.io |
| `oss-find-squats --quiet <purl>` | Suppress non-finding output |

Package URL (purl) format: `pkg:<type>/<namespace>/<name>@<version>`

## pypi-scan (IQTLabs)

| Command | Purpose |
|---------|---------|
| `python pypi_scan.py -p <package>` | Find candidate typosquats of one package |
| `python pypi_scan.py -n <N>` | Scan the top-N most-downloaded PyPI packages |

## Registry metadata endpoints

| Endpoint | Returns |
|----------|---------|
| `https://registry.npmjs.org/<pkg>` | Full npm package doc (time, maintainers, versions, scripts) |
| `https://api.npmjs.org/downloads/point/last-week/<pkg>` | npm weekly download count |
| `https://pypi.org/pypi/<pkg>/json` | PyPI info, releases, author, urls |
| `https://crates.io/api/v1/crates/<pkg>` | crates.io crate metadata (requires User-Agent) |
| `https://registry.npmjs.org/-/v1/search?text=...&popularity=1.0` | npm popularity search |

## Risk-scoring signals

| Signal | High-risk value |
|--------|-----------------|
| Edit distance to popular name | 1–2 |
| Package age | < 90 days |
| Weekly downloads | < 1000 |
| Install scripts | `preinstall` / `postinstall` / network in `setup.py` |
| Maintainer overlap | Different maintainer than legit package |
| Repository URL | Missing, or points to legit project (impersonation) |

===== END FILE =====


===== FILE: skills/detecting-typosquatting-packages/references/standards.md =====
# Standards and References - Detecting Typosquatting Packages

## MITRE ATT&CK

| Technique ID | Name | Tactic | Rationale |
|--------------|------|--------|-----------|
| T1195.002 | Supply Chain Compromise: Compromise Software Supply Chain | Initial Access | A typosquatted package is the malicious artifact the attacker gets installed in place of the legitimate dependency, compromising the victim's software supply chain at install time. |

## NIST Cybersecurity Framework 2.0

| ID | Name | Rationale |
|----|------|-----------|
| ID.RA-09 | The authenticity and integrity of hardware and software is assessed prior to acquisition and use | Pre-installation typosquat screening is exactly the activity of assessing a package's authenticity before it enters the environment. |

## Official Resources

- typomania (Rust Foundation): https://github.com/rustfoundation/typomania
- typogard (original research tool): https://github.com/mt3443/typogard
- "Defending Against Package Typosquatting" (Taylor et al., University of Kansas)
- Microsoft OSSGadget: https://github.com/microsoft/OSSGadget
- IQTLabs pypi-scan: https://github.com/IQTLabs/pypi-scan
- ecosyste-ms typosquatting-dataset: https://github.com/ecosyste-ms/typosquatting-dataset
- OWASP CI/CD Security Top 10: https://owasp.org/www-project-top-10-ci-cd-security-risks/
- top-pypi-packages corpus: https://hugovk.github.io/top-pypi-packages/

## Key Research

- Rust Foundation: typomania powers crates.io typosquatting detection
- JFrog / Sonatype: annual reports on registry typosquatting and "slopsquatting" trends

===== END FILE =====


===== FILE: skills/detecting-typosquatting-packages/scripts/agent.py =====
#!/usr/bin/env python3
"""
Typosquatting screening helper.

Generates the standard typogard/typomania mutation set for a candidate package
name, screens names against a popular-name corpus, and enriches suspected
squats with live registry metadata (npm / PyPI) to support triage.

No third-party dependencies required (stdlib only). Network calls use urllib.

Examples:
    # Screen a single name against a PyPI corpus
    python agent.py screen --ecosystem pypi --corpus top-pypi-packages.json --name reqeusts

    # Screen names from stdin (e.g., a diff of requirements.txt)
    git diff | python agent.py screen --ecosystem npm --corpus npm-top.json --stdin

    # Show the generated mutation set for a legit name
    python agent.py mutate --name requests

    # Pull registry metadata for triage
    python agent.py enrich --ecosystem pypi --name reqeusts
"""
import argparse
import json
import sys
import urllib.request
import urllib.error

QWERTY_ADJ = {
    "q": "wa", "w": "qeas", "e": "wrsd", "r": "etdf", "t": "ryfg",
    "y": "tugh", "u": "yihj", "i": "uojk", "o": "ipkl", "p": "ol",
    "a": "qwsz", "s": "awedxz", "d": "serfcx", "f": "drtgvc",
    "g": "ftyhbv", "h": "gyujnb", "j": "huikmn", "k": "jiolm",
    "l": "kop", "z": "asx", "x": "zsdc", "c": "xdfv", "v": "cfgb",
    "b": "vghn", "n": "bhjm", "m": "njk",
    "0": "9", "1": "2", "2": "13", "3": "24", "4": "35",
}
DELIMS = ["-", "_", ".", ""]


def mutations(name):
    """Reproduce the typogard/typomania squat primitives for `name`."""
    out = set()
    n = name.lower()
    # 1. omission
    for i in range(len(n)):
        out.add(n[:i] + n[i + 1:])
    # 2. repetition
    for i in range(len(n)):
        out.add(n[:i] + n[i] + n[i:])
    # 3. transposition / swap of adjacent chars
    for i in range(len(n) - 1):
        out.add(n[:i] + n[i + 1] + n[i] + n[i + 2:])
    # 4. keyboard-adjacency 1-edit substitution
    for i, ch in enumerate(n):
        for repl in QWERTY_ADJ.get(ch, ""):
            out.add(n[:i] + repl + n[i + 1:])
    # 5. delimiter swaps
    for d in DELIMS:
        for d2 in DELIMS:
            if d and d in n:
                out.add(n.replace(d, d2))
    # 6. word-order swap for compound names
    for d in ("-", "_", "."):
        if d in n:
            parts = n.split(d)
            if len(parts) == 2:
                out.add(d.join(reversed(parts)))
    # 7. common suffixes
    for suf in ("js", "py", "lib", "cli", "2", "-ng"):
        out.add(n + suf)
        out.add(n + "-" + suf)
    out.discard(n)
    return {m for m in out if m}


def levenshtein(a, b):
    if len(a) < len(b):
        a, b = b, a
    prev = list(range(len(b) + 1))
    for i, ca in enumerate(a, 1):
        cur = [i]
        for j, cb in enumerate(b, 1):
            cur.append(min(prev[j] + 1, cur[j - 1] + 1, prev[j - 1] + (ca != cb)))
        prev = cur
    return prev[-1]


def load_corpus(path):
    with open(path, "r", encoding="utf-8") as fh:
        data = json.load(fh)
    names = set()
    # Accept several common corpus shapes
    if isinstance(data, dict) and "rows" in data:          # top-pypi-packages
        names = {r["project"] for r in data["rows"]}
    elif isinstance(data, dict) and "objects" in data:     # npm search
        names = {o["package"]["name"] for o in data["objects"]}
    elif isinstance(data, list):
        names = {(x if isinstance(x, str) else x.get("name", "")) for x in data}
    elif isinstance(data, dict):
        names = set(data.keys())
    return {n.lower() for n in names if n}


def http_json(url):
    req = urllib.request.Request(url, headers={"User-Agent": "typosquat-screen/1.0"})
    with urllib.request.urlopen(req, timeout=20) as resp:
        return json.loads(resp.read().decode("utf-8"))


def enrich(ecosystem, name):
    """Pull triage metadata from the live registry."""
    try:
        if ecosystem == "npm":
            d = http_json(f"https://registry.npmjs.org/{name}")
            latest = d.get("dist-tags", {}).get("latest")
            ver = d.get("versions", {}).get(latest, {})
            return {
                "name": name,
                "created": d.get("time", {}).get("created"),
                "maintainers": [m.get("name") for m in d.get("maintainers", [])],
                "scripts": ver.get("scripts", {}),
                "repository": ver.get("repository"),
            }
        elif ecosystem == "pypi":
            d = http_json(f"https://pypi.org/pypi/{name}/json")
            info = d.get("info", {})
            return {
                "name": name,
                "author": info.get("author"),
                "home_page": info.get("home_page"),
                "releases": list(d.get("releases", {}).keys()),
                "project_urls": info.get("project_urls"),
            }
    except urllib.error.HTTPError as exc:
        return {"name": name, "error": f"HTTP {exc.code} (name may not exist)"}
    except Exception as exc:  # noqa: BLE001
        return {"name": name, "error": str(exc)}
    return {"name": name, "error": "unsupported ecosystem"}


def screen_name(name, corpus, threshold=2):
    """Return list of (legit_name, distance) the candidate squats on."""
    name = name.lower().strip()
    hits = []
    if name in corpus:
        return []  # exact legit match is not a squat
    for legit in corpus:
        d = levenshtein(name, legit)
        if 0 < d <= threshold and abs(len(name) - len(legit)) <= threshold:
            hits.append((legit, d))
    hits.sort(key=lambda x: x[1])
    return hits


def cmd_mutate(args):
    for m in sorted(mutations(args.name)):
        print(m)


def cmd_screen(args):
    corpus = load_corpus(args.corpus)
    candidates = []
    if args.stdin:
        for line in sys.stdin:
            for tok in line.replace('"', " ").replace(",", " ").split():
                tok = tok.strip("+-=:'`@^~>< \t")
                if tok and tok[0].isalpha():
                    candidates.append(tok)
    if args.name:
        candidates.append(args.name)
    flagged = 0
    for cand in dict.fromkeys(candidates):
        hits = screen_name(cand, corpus, args.threshold)
        if hits:
            flagged += 1
            top = ", ".join(f"{l}(d={d})" for l, d in hits[:3])
            print(f"[FLAG] {cand} -> resembles {top}")
    print(f"\nScreened {len(set(candidates))} name(s), flagged {flagged}.", file=sys.stderr)
    sys.exit(1 if flagged else 0)


def cmd_enrich(args):
    print(json.dumps(enrich(args.ecosystem, args.name), indent=2))


def main():
    p = argparse.ArgumentParser(description="Typosquatting screening helper")
    sub = p.add_subparsers(dest="cmd", required=True)

    m = sub.add_parser("mutate", help="Print squat mutations of a name")
    m.add_argument("--name", required=True)
    m.set_defaults(func=cmd_mutate)

    s = sub.add_parser("screen", help="Screen names against a popular-name corpus")
    s.add_argument("--corpus", required=True)
    s.add_argument("--ecosystem", choices=["npm", "pypi", "cargo"], default="pypi")
    s.add_argument("--name")
    s.add_argument("--stdin", action="store_true")
    s.add_argument("--threshold", type=int, default=2)
    s.set_defaults(func=cmd_screen)

    e = sub.add_parser("enrich", help="Pull registry metadata for triage")
    e.add_argument("--ecosystem", choices=["npm", "pypi"], required=True)
    e.add_argument("--name", required=True)
    e.set_defaults(func=cmd_enrich)

    args = p.parse_args()
    args.func(args)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-typosquatting-packages-in-npm-pypi/SKILL.md =====
---
name: detecting-typosquatting-packages-in-npm-pypi
description: 'Detects typosquatting attacks in npm and PyPI package registries by
  analyzing package name similarity using Levenshtein distance and other string metrics,
  examining publish date heuristics to identify recently created packages mimicking
  established ones, and flagging download count anomalies where suspicious packages
  have disproportionately low usage compared to their legitimate targets. The analyst
  queries the PyPI JSON API and npm registry API to gather package metadata for automated
  comparison. Activates for requests involving package typosquatting detection, dependency
  confusion analysis, malicious package identification, or software supply chain threat
  hunting in package registries.

  '
domain: cybersecurity
subdomain: supply-chain-security
tags:
- typosquatting
- npm
- pypi
- supply-chain
- package-security
- Levenshtein
- dependency-confusion
- malicious-packages
version: 1.0.0
author: mukul975
license: Apache-2.0
nist_csf:
- GV.SC-01
- GV.SC-03
- GV.SC-06
- GV.SC-07
mitre_attack:
- T1195.001
- T1195.002
- T1608.001
- T1554
---
# Detecting Typosquatting Packages in npm and PyPI

## When to Use

- Auditing project dependencies to identify packages whose names are suspiciously similar to popular libraries
- Proactively scanning package registries for newly published packages that may be typosquats of your organization's packages
- Investigating a suspected supply chain compromise where a developer installed a misspelled package name
- Building automated monitoring that alerts when new packages appear with names close to critical dependencies
- Assessing the risk profile of unfamiliar packages before adding them to a project's dependency tree

**Do not use** as the sole determination of malicious intent; name similarity alone does not prove a package is malicious. Do not use for bulk automated takedown requests without manual review of flagged packages. Do not use against private registries without authorization.

## Prerequisites

- Python 3.9+ with `requests` and `python-Levenshtein` (or `rapidfuzz`) packages installed
- Network access to `https://pypi.org/pypi/<package>/json` (PyPI JSON API) and `https://registry.npmjs.org/<package>` (npm registry API)
- A list of popular or critical packages to monitor (e.g., top 1000 PyPI packages, organization's dependency list)
- Understanding of common typosquatting patterns: character omission, transposition, insertion, substitution, and hyphen/underscore manipulation

## Workflow

### Step 1: Build the Target Package Watchlist

Establish the set of legitimate packages to monitor for typosquats:

- **Extract project dependencies**: Parse `requirements.txt`, `Pipfile.lock`, `package.json`, or `package-lock.json` to extract all direct and transitive dependency names
- **Include popular packages**: Supplement with high-value targets from the top 1000 PyPI downloads (available from `https://hugovk.github.io/top-pypi-packages/`) or top npm packages by download count
- **Add organization packages**: Include any packages published by your organization that attackers might target with typosquats to intercept internal installations
- **Normalize names**: PyPI treats hyphens, underscores, and periods as equivalent (PEP 503 normalization: `re.sub(r"[-_.]+", "-", name).lower()`). npm package names are case-sensitive but scoped packages use `@scope/name` format. Normalize before comparison.

### Step 2: Generate Candidate Typosquat Names

Produce potential typosquat variants for each target package:

- **Character omission**: Remove each character one at a time (`requests` -> `rquests`, `requets`, `reqests`)
- **Character transposition**: Swap adjacent characters (`requests` -> `erquests`, `rqeuests`, `reques ts`)
- **Character substitution**: Replace characters with keyboard-adjacent keys using a QWERTY distance map (`requests` -> `rrquests`, `requesta`)
- **Character insertion**: Insert common characters at each position (`requests` -> `rrequests`, `reqquests`)
- **Separator manipulation**: For hyphenated names, try removing, doubling, or replacing separators (`my-package` -> `mypackage`, `my--package`, `my_package`)
- **Common prefix/suffix attacks**: Prepend or append common strings (`python-requests`, `requests-python`, `requests2`, `requests-lib`)

### Step 3: Query Registry APIs for Candidate Packages

Check whether generated candidate names actually exist in the registry:

- **PyPI JSON API**: Send `GET https://pypi.org/pypi/<candidate>/json` for each candidate. A `200` response means the package exists; `404` means it does not. Extract from the response: `info.name`, `info.version`, `info.author`, `info.summary`, `info.home_page`, `info.project_urls`, and `releases` (keyed by version with `upload_time_iso_8601` timestamps).
- **npm registry API**: Send `GET https://registry.npmjs.org/<candidate>` with `Accept: application/json`. Extract: `name`, `description`, `dist-tags.latest`, `time.created`, `time.modified`, `maintainers`, and `versions`.
- **Rate limiting**: PyPI has no published rate limits but respect reasonable request rates (1-2 requests/second). npm registry returns `429` when rate limited; implement exponential backoff.
- **Batch optimization**: For large candidate lists, parallelize requests with connection pooling (`requests.Session`) and limit concurrency to avoid triggering abuse protections.

### Step 4: Analyze Package Metadata for Suspicion Signals

Score each existing candidate package against multiple heuristic signals:

- **Levenshtein distance**: Calculate the edit distance between the candidate name and the target. Packages with distance 1-2 from a popular package are high-priority suspects. Historical analysis shows 18 of 40 known typosquats had Levenshtein distance of 2 or less from their targets.
- **Publish date recency**: Compare the candidate's first publish date against the target's. A package created years after its near-namesake is more suspicious. Flag packages created within the last 90 days that are similar to packages published years ago.
- **Download count disparity**: Compare weekly downloads. Legitimate similarly-named packages typically have comparable or explainable download counts. A package with 50 downloads versus its near-namesake with 5 million downloads is suspicious. PyPI download stats are available via BigQuery (`pypistats.org/api/`); npm provides download counts at `https://api.npmjs.org/downloads/point/last-week/<package>`.
- **Author and maintainer analysis**: Check if the candidate package author matches the legitimate package author. Different authors for near-identical names increase suspicion.
- **Description similarity**: Compare package descriptions. Typosquats frequently copy or closely paraphrase the target package description to appear legitimate.
- **Version count**: Legitimate packages typically have many versions over time. A package with only 1-2 versions and a name similar to a popular package is suspicious.
- **Repository URL analysis**: Check if the candidate links to the same repository as the target (likely legitimate fork/mirror) or has no repository URL (suspicious).

### Step 5: Score, Rank, and Report Findings

Combine signals into a composite risk score and generate an actionable report:

- **Weighted scoring**: Assign weights to each signal. Example: Levenshtein distance 1 = 40 points, Levenshtein distance 2 = 25 points, created < 90 days ago = 15 points, download ratio < 0.001 = 15 points, different author = 10 points, single version = 5 points. Total score out of 100.
- **Threshold classification**: Score >= 70: HIGH risk (likely typosquat), 40-69: MEDIUM risk (requires manual review), < 40: LOW risk (likely legitimate)
- **Generate report**: For each flagged package, include the target it mimics, all signal values, the composite score, direct links to both packages on the registry, and a recommendation (block, investigate, or allow)
- **Actionable output**: Produce a blocklist of flagged package names that can be imported into package manager deny-lists, CI/CD policy engines, or artifact repository proxy rules

## Key Concepts

| Term | Definition |
|------|------------|
| **Typosquatting** | Registering a package name that closely resembles a popular package, exploiting common typos to trick developers into installing malicious code |
| **Levenshtein Distance** | The minimum number of single-character edits (insertions, deletions, substitutions) required to transform one string into another; the primary metric for measuring name similarity |
| **Dependency Confusion** | A broader supply chain attack where attackers publish malicious packages to public registries with names matching private internal packages, exploiting package manager resolution order |
| **PEP 503 Normalization** | The Python packaging specification that treats hyphens, underscores, and periods as equivalent in package names, meaning `my-package`, `my_package`, and `my.package` resolve to the same package |
| **QWERTY Distance** | A keyboard-layout-aware distance metric measuring how far apart two keys are on a standard keyboard, used to detect substitutions from adjacent key mistyping |
| **Combosquatting** | A variant of typosquatting where attackers prepend or append common words to a package name (e.g., `requests-security`, `python-requests`) |
| **StarJacking** | An attack where a typosquat package links its repository URL to the legitimate package's GitHub repository to inflate apparent credibility |

## Tools & Systems

- **PyPI JSON API**: REST API at `https://pypi.org/pypi/<package>/json` returning package metadata including name, author, versions, upload timestamps, and project URLs
- **npm Registry API**: REST API at `https://registry.npmjs.org/<package>` returning package metadata including maintainers, version history, creation timestamps, and distribution info
- **python-Levenshtein / rapidfuzz**: Python libraries for fast string distance computation, supporting Levenshtein, Damerau-Levenshtein, Jaro-Winkler, and other similarity metrics
- **pypistats.org API**: Provides download statistics for PyPI packages, enabling download count comparison between suspected typosquats and their targets
- **npm download counts API**: Endpoint at `https://api.npmjs.org/downloads/point/<period>/<package>` providing download statistics for npm packages

## Common Scenarios

### Scenario: Auditing a Python Project for Typosquatted Dependencies

**Context**: A security team discovers that a developer's workstation was compromised after installing a Python package. The incident response team needs to audit all project dependencies for potential typosquats and establish ongoing monitoring.

**Approach**:
1. Parse `requirements.txt` and `Pipfile.lock` to extract all 87 direct and transitive dependencies
2. Generate typosquat candidates for each dependency using character omission, transposition, substitution, and separator manipulation, producing approximately 2,400 candidate names
3. Query the PyPI JSON API for each candidate, finding 34 that actually exist as published packages
4. Score each existing candidate: 3 packages score above 70 (HIGH risk) with Levenshtein distance 1, created within the last 60 days, single version, and fewer than 100 downloads
5. Manual review confirms 2 of the 3 are malicious typosquats containing obfuscated code that exfiltrates environment variables during installation
6. Block the malicious packages in the organization's artifact proxy, report to PyPI for takedown via `security@pypi.org`, and add all 87 dependencies to the ongoing monitoring watchlist
7. Implement the detection agent as a scheduled CI job that runs weekly and alerts on new HIGH-risk findings

**Pitfalls**:
- Not normalizing PyPI package names per PEP 503 before comparison, causing missed matches between hyphenated and underscored variants
- Setting the Levenshtein distance threshold too low (only 1) and missing typosquats at distance 2 that use double substitutions
- Relying solely on name similarity without checking metadata signals, leading to high false positive rates on legitimately similar package names
- Not accounting for npm scoped packages (`@scope/name`) which have different naming rules than unscoped packages
- Querying the registries too aggressively and getting rate-limited or IP-blocked

## Output Format

```
## Typosquatting Detection Report

**Scan Date**: 2026-03-19
**Registry**: PyPI
**Packages Monitored**: 87
**Candidates Generated**: 2,412
**Candidates Found in Registry**: 34
**Flagged as Suspicious**: 5

### HIGH Risk (Score >= 70)

| Suspect Package | Target Package | Levenshtein | Created | Downloads | Score |
|----------------|---------------|-------------|---------|-----------|-------|
| reqeusts       | requests      | 1           | 2026-02-28 | 43     | 92    |
| requsets       | requests      | 1           | 2026-03-01 | 12     | 88    |
| numpyy         | numpy         | 1           | 2026-01-15 | 67     | 78    |

### Recommendation
- BLOCK: reqeusts, requsets, numpyy (add to artifact proxy deny-list)
- REPORT: Submit malware reports to security@pypi.org with package names and evidence
- MONITOR: Continue weekly scans for the full dependency watchlist
```

===== END FILE =====


===== FILE: skills/detecting-typosquatting-packages-in-npm-pypi/LICENSE =====

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


===== FILE: skills/detecting-typosquatting-packages-in-npm-pypi/references/api-reference.md =====
# API Reference: Typosquatting Detection Agent for npm and PyPI

## Overview

Detects typosquatting attacks in npm and PyPI package registries by generating candidate typosquat names using string manipulation techniques, querying registry APIs to check which candidates exist, and scoring each against multiple heuristic signals including Levenshtein distance, publish date recency, download count disparity, author mismatch, and version count. Produces risk-scored reports for security review.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| requests | >=2.28 | HTTP requests to PyPI and npm registry APIs |
| python-Levenshtein | >=0.21 | Fast Levenshtein distance computation (optional; pure-Python fallback included) |
| rapidfuzz | >=3.0 | Alternative fast string distance library (optional) |

## CLI Usage

```bash
# Scan for typosquats of a single PyPI package
python agent.py scan requests --registry pypi

# Scan for typosquats of an npm package
python agent.py scan express --registry npm

# Scan with limited candidate count
python agent.py scan numpy --registry pypi --max-candidates 50

# Scan all dependencies in a requirements file
python agent.py scan-file requirements.txt --registry pypi

# Scan all dependencies in a package.json
python agent.py scan-file package.json --registry npm

# Check a specific candidate against a target
python agent.py check reqeusts requests --registry pypi

# Generate typosquat candidates without querying registries
python agent.py generate requests

# Custom output path
python agent.py scan flask --registry pypi --output flask_typosquat_report.json
```

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `command` | Yes | Subcommand: `scan`, `scan-file`, `check`, `generate` |
| `package` | For scan/generate | Target package name to analyze |
| `file` | For scan-file | Path to requirements.txt, package.json, or similar |
| `candidate` | For check | Candidate package name to evaluate |
| `target` | For check | Legitimate target package name to compare against |
| `--registry` | No | Registry to scan: `pypi` or `npm` (default: `pypi`) |
| `--max-candidates` | No | Maximum number of candidates to check per package |
| `--output` | No | Output report path (default: `typosquat_report.json`) |

## Key Functions

### `generate_typosquat_candidates(name)`
Generates potential typosquat variants using character omission, transposition, duplication, QWERTY keyboard-adjacent substitution, separator manipulation, and common prefix/suffix combosquatting. Returns a sorted list of unique candidate strings.

### `query_pypi_package(name, delay)`
Queries `GET https://pypi.org/pypi/<name>/json` and parses name, version, author, summary, version count, and first/latest upload timestamps from the response. Returns `None` for non-existent packages (HTTP 404).

### `query_npm_package(name, delay)`
Queries `GET https://registry.npmjs.org/<name>` and parses name, description, maintainers, version count, created/modified timestamps, license, and repository URL. Handles HTTP 429 rate limiting with exponential backoff.

### `get_pypi_downloads(name)`
Queries `https://pypistats.org/api/packages/<name>/recent` to retrieve last-week download count for download disparity analysis.

### `get_npm_downloads(name)`
Queries `https://api.npmjs.org/downloads/point/last-week/<name>` to retrieve last-week download count.

### `compute_suspicion_score(candidate_meta, target_meta, target_name, registry)`
Computes a weighted suspicion score (0-100) combining six signals: Levenshtein distance (up to 40pts), publish recency (up to 15pts), download ratio (up to 15pts), different author (10pts), low version count (5pts), and missing repository URL (5pts). Returns the score and a signal breakdown dictionary.

### `classify_risk(score)`
Maps composite score to risk level: HIGH (>=70), MEDIUM (40-69), LOW (<40).

### `scan_package(target_name, registry, max_candidates)`
End-to-end scan: fetches target metadata, generates candidates, queries registry for each, scores existing candidates, and returns ranked results sorted by descending score.

### `scan_dependency_file(filepath, registry, max_candidates_per_pkg)`
Parses a dependency file (requirements.txt, package.json, Pipfile), extracts package names, and runs `scan_package` for each. Returns aggregated results with high/medium/low summary counts.

### `normalize_pypi_name(name)`
Normalizes PyPI package names per PEP 503: replaces hyphens, underscores, and periods with a single hyphen and lowercases the result.

## Registry API Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `https://pypi.org/pypi/<name>/json` | GET | PyPI package metadata (info, releases, URLs) |
| `https://registry.npmjs.org/<name>` | GET | npm package metadata (versions, time, maintainers) |
| `https://pypistats.org/api/packages/<name>/recent` | GET | PyPI download statistics |
| `https://api.npmjs.org/downloads/point/last-week/<name>` | GET | npm download statistics |

## Scoring Weights

| Signal | Condition | Points |
|--------|-----------|--------|
| Levenshtein distance | Distance = 1 | 40 |
| Levenshtein distance | Distance = 2 | 25 |
| Levenshtein distance | Distance = 3 | 10 |
| Publish recency | Created <= 90 days ago | 15 |
| Publish recency | Created <= 180 days ago | 8 |
| Download ratio | candidate/target < 0.001 | 15 |
| Download ratio | candidate/target < 0.01 | 8 |
| Author mismatch | Different author/maintainer | 10 |
| Version count | <= 2 versions | 5 |
| Repository URL | Missing | 5 |

===== END FILE =====


===== FILE: skills/detecting-typosquatting-packages-in-npm-pypi/scripts/agent.py =====
#!/usr/bin/env python3
"""Typosquatting Detection Agent - Detects typosquatting packages in npm and PyPI
registries using Levenshtein distance analysis, publish date heuristics, and
download count anomalies."""

import json
import logging
import argparse
import re
import time
from datetime import datetime, timezone
from pathlib import Path

import requests

try:
    from Levenshtein import distance as levenshtein_distance
except ImportError:
    # Fallback pure-Python Levenshtein implementation
    def levenshtein_distance(s1, s2):
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


logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

PYPI_API = "https://pypi.org/pypi/{}/json"
NPM_API = "https://registry.npmjs.org/{}"
NPM_DOWNLOADS_API = "https://api.npmjs.org/downloads/point/last-week/{}"
PYPISTATS_API = "https://pypistats.org/api/packages/{}/recent"

SESSION = requests.Session()
SESSION.headers.update({"Accept": "application/json", "User-Agent": "typosquat-detector/1.0"})


def normalize_pypi_name(name):
    """Normalize a PyPI package name per PEP 503."""
    return re.sub(r"[-_.]+", "-", name).lower()


def generate_typosquat_candidates(name):
    """Generate potential typosquat variants of a package name.

    Produces candidates via character omission, transposition, insertion,
    substitution (keyboard-adjacent), and separator manipulation.
    """
    candidates = set()
    lower_name = name.lower()

    # Character omission: remove each character one at a time
    for i in range(len(lower_name)):
        candidate = lower_name[:i] + lower_name[i + 1:]
        if candidate and candidate != lower_name:
            candidates.add(candidate)

    # Character transposition: swap adjacent characters
    for i in range(len(lower_name) - 1):
        chars = list(lower_name)
        chars[i], chars[i + 1] = chars[i + 1], chars[i]
        candidate = "".join(chars)
        if candidate != lower_name:
            candidates.add(candidate)

    # Character duplication: double each character
    for i in range(len(lower_name)):
        candidate = lower_name[:i] + lower_name[i] + lower_name[i:]
        if candidate != lower_name:
            candidates.add(candidate)

    # Keyboard-adjacent substitution (QWERTY layout)
    qwerty_neighbors = {
        "q": "wa", "w": "qeas", "e": "wrds", "r": "etfs", "t": "ryg",
        "y": "tuh", "u": "yij", "i": "uok", "o": "ipl", "p": "ol",
        "a": "qwsz", "s": "wedxza", "d": "erfcxs", "f": "rtgvcd",
        "g": "tyhbvf", "h": "yujng", "j": "uikmh", "k": "ioljm",
        "l": "opk", "z": "asx", "x": "zsdc", "c": "xdfv", "v": "cfgb",
        "b": "vghn", "n": "bhjm", "m": "njk",
    }
    for i, ch in enumerate(lower_name):
        for neighbor in qwerty_neighbors.get(ch, ""):
            candidate = lower_name[:i] + neighbor + lower_name[i + 1:]
            if candidate != lower_name:
                candidates.add(candidate)

    # Separator manipulation for hyphenated/underscored names
    if "-" in lower_name or "_" in lower_name:
        candidates.add(lower_name.replace("-", "").replace("_", ""))
        candidates.add(lower_name.replace("-", "_"))
        candidates.add(lower_name.replace("_", "-"))
        candidates.add(lower_name.replace("-", "--"))

    # Common prefix/suffix combosquatting
    for affix in ["python-", "py-", "-python", "-py", "-lib", "-sdk", "2", "3"]:
        if affix.startswith("-"):
            candidates.add(lower_name + affix)
        else:
            candidates.add(affix + lower_name)

    # Remove the original name if present
    candidates.discard(lower_name)
    candidates.discard(name)

    return sorted(candidates)


def query_pypi_package(name, delay=0.5):
    """Query the PyPI JSON API for package metadata.

    Returns parsed metadata or None if the package does not exist.
    """
    url = PYPI_API.format(name)
    try:
        time.sleep(delay)
        resp = SESSION.get(url, timeout=15)
        if resp.status_code == 404:
            return None
        resp.raise_for_status()
        data = resp.json()
        info = data.get("info", {})
        releases = data.get("releases", {})

        # Find first and latest upload times
        upload_times = []
        for version_files in releases.values():
            for f in version_files:
                if f.get("upload_time_iso_8601"):
                    upload_times.append(f["upload_time_iso_8601"])

        first_upload = min(upload_times) if upload_times else None
        latest_upload = max(upload_times) if upload_times else None

        return {
            "registry": "pypi",
            "name": info.get("name", name),
            "version": info.get("version"),
            "author": info.get("author"),
            "author_email": info.get("author_email"),
            "summary": info.get("summary"),
            "home_page": info.get("home_page"),
            "project_url": info.get("project_url"),
            "requires_python": info.get("requires_python"),
            "license": info.get("license"),
            "version_count": len(releases),
            "first_upload": first_upload,
            "latest_upload": latest_upload,
            "exists": True,
        }
    except requests.RequestException as e:
        logger.warning("PyPI query failed for %s: %s", name, e)
        return None


def query_npm_package(name, delay=0.5):
    """Query the npm registry API for package metadata.

    Returns parsed metadata or None if the package does not exist.
    """
    url = NPM_API.format(name)
    try:
        time.sleep(delay)
        resp = SESSION.get(url, timeout=15)
        if resp.status_code == 404:
            return None
        if resp.status_code == 429:
            logger.warning("npm rate limited, waiting 10 seconds")
            time.sleep(10)
            resp = SESSION.get(url, timeout=15)
        resp.raise_for_status()
        data = resp.json()
        time_info = data.get("time", {})
        maintainers = data.get("maintainers", [])

        return {
            "registry": "npm",
            "name": data.get("name", name),
            "description": data.get("description"),
            "dist_tags_latest": data.get("dist-tags", {}).get("latest"),
            "created": time_info.get("created"),
            "modified": time_info.get("modified"),
            "maintainers": [m.get("name") for m in maintainers],
            "version_count": len(data.get("versions", {})),
            "license": data.get("license"),
            "homepage": data.get("homepage"),
            "repository": data.get("repository", {}).get("url") if isinstance(data.get("repository"), dict) else data.get("repository"),
            "exists": True,
        }
    except requests.RequestException as e:
        logger.warning("npm query failed for %s: %s", name, e)
        return None


def get_pypi_downloads(name):
    """Get recent download stats for a PyPI package from pypistats.org."""
    url = PYPISTATS_API.format(name)
    try:
        resp = SESSION.get(url, timeout=10)
        if resp.status_code != 200:
            return None
        data = resp.json().get("data", {})
        return data.get("last_week", 0)
    except requests.RequestException:
        return None


def get_npm_downloads(name):
    """Get last-week download count for an npm package."""
    url = NPM_DOWNLOADS_API.format(name)
    try:
        resp = SESSION.get(url, timeout=10)
        if resp.status_code != 200:
            return None
        return resp.json().get("downloads", 0)
    except requests.RequestException:
        return None


def compute_suspicion_score(candidate_meta, target_meta, target_name, registry):
    """Compute a weighted suspicion score for a candidate typosquat package.

    Signals:
    - Levenshtein distance (1 = 40pts, 2 = 25pts, 3 = 10pts)
    - Publish recency: created within 90 days = 15pts
    - Download ratio: candidate/target < 0.001 = 15pts
    - Different author/maintainer = 10pts
    - Low version count (<=2) = 5pts
    - No repository URL = 5pts
    """
    score = 0
    signals = {}
    candidate_name = candidate_meta.get("name", "")

    # Levenshtein distance
    if registry == "pypi":
        dist = levenshtein_distance(
            normalize_pypi_name(candidate_name),
            normalize_pypi_name(target_name),
        )
    else:
        dist = levenshtein_distance(candidate_name.lower(), target_name.lower())

    signals["levenshtein_distance"] = dist
    if dist == 1:
        score += 40
    elif dist == 2:
        score += 25
    elif dist == 3:
        score += 10

    # Publish recency
    now = datetime.now(timezone.utc)
    first_publish = candidate_meta.get("first_upload") or candidate_meta.get("created")
    if first_publish:
        try:
            if isinstance(first_publish, str):
                first_dt = datetime.fromisoformat(first_publish.replace("Z", "+00:00"))
            else:
                first_dt = first_publish
            days_old = (now - first_dt).days
            signals["days_since_first_publish"] = days_old
            if days_old <= 90:
                score += 15
            elif days_old <= 180:
                score += 8
        except (ValueError, TypeError):
            pass

    # Download disparity
    if registry == "pypi":
        candidate_dl = get_pypi_downloads(candidate_name)
        target_dl = get_pypi_downloads(target_name)
    else:
        candidate_dl = get_npm_downloads(candidate_name)
        target_dl = get_npm_downloads(target_name)

    signals["candidate_downloads_weekly"] = candidate_dl
    signals["target_downloads_weekly"] = target_dl
    if candidate_dl is not None and target_dl and target_dl > 0:
        ratio = candidate_dl / target_dl
        signals["download_ratio"] = round(ratio, 6)
        if ratio < 0.001:
            score += 15
        elif ratio < 0.01:
            score += 8

    # Author comparison
    if registry == "pypi":
        candidate_author = (candidate_meta.get("author") or "").lower().strip()
        target_author = (target_meta.get("author") or "").lower().strip()
    else:
        candidate_author = set(m.lower() for m in (candidate_meta.get("maintainers") or []))
        target_author = set(m.lower() for m in (target_meta.get("maintainers") or []))

    if candidate_author and target_author and candidate_author != target_author:
        score += 10
        signals["different_author"] = True
    else:
        signals["different_author"] = False

    # Version count
    version_count = candidate_meta.get("version_count", 0)
    signals["version_count"] = version_count
    if version_count <= 2:
        score += 5

    # Repository URL presence
    repo = candidate_meta.get("home_page") or candidate_meta.get("homepage") or candidate_meta.get("repository")
    signals["has_repository"] = bool(repo)
    if not repo:
        score += 5

    signals["total_score"] = score
    return score, signals


def classify_risk(score):
    """Classify risk level based on composite score."""
    if score >= 70:
        return "HIGH"
    elif score >= 40:
        return "MEDIUM"
    else:
        return "LOW"


def scan_package(target_name, registry="pypi", max_candidates=None):
    """Scan for typosquat candidates of a target package.

    Generates candidates, checks which exist in the registry, scores them,
    and returns ranked results.
    """
    logger.info("Scanning for typosquats of '%s' on %s", target_name, registry)

    # Fetch target package metadata
    if registry == "pypi":
        target_meta = query_pypi_package(target_name, delay=0.2)
    else:
        target_meta = query_npm_package(target_name, delay=0.2)

    if not target_meta:
        logger.warning("Target package '%s' not found on %s", target_name, registry)
        return {"target": target_name, "registry": registry, "error": "Target package not found"}

    # Generate candidates
    candidates = generate_typosquat_candidates(target_name)
    if max_candidates:
        candidates = candidates[:max_candidates]
    logger.info("Generated %d typosquat candidates for '%s'", len(candidates), target_name)

    # Query registry for each candidate
    results = []
    for i, candidate in enumerate(candidates):
        if registry == "pypi":
            meta = query_pypi_package(candidate, delay=0.3)
        else:
            meta = query_npm_package(candidate, delay=0.3)

        if meta and meta.get("exists"):
            score, signals = compute_suspicion_score(
                meta, target_meta, target_name, registry
            )
            risk = classify_risk(score)
            results.append({
                "candidate": candidate,
                "target": target_name,
                "registry": registry,
                "score": score,
                "risk": risk,
                "signals": signals,
                "metadata": meta,
            })
            logger.info(
                "  [%s] %s (score=%d, lev=%d)",
                risk, candidate, score, signals.get("levenshtein_distance", -1),
            )

        if (i + 1) % 50 == 0:
            logger.info("  Progress: %d/%d candidates checked", i + 1, len(candidates))

    # Sort by score descending
    results.sort(key=lambda r: r["score"], reverse=True)

    return {
        "target": target_name,
        "registry": registry,
        "target_metadata": target_meta,
        "candidates_generated": len(candidates),
        "candidates_found": len(results),
        "results": results,
    }


def scan_dependency_file(filepath, registry="pypi", max_candidates_per_pkg=None):
    """Scan all dependencies in a requirements file or package.json."""
    filepath = Path(filepath)
    if not filepath.exists():
        return {"error": f"File not found: {filepath}"}

    content = filepath.read_text()
    packages = []

    if filepath.name in ("requirements.txt", "requirements.in"):
        for line in content.splitlines():
            line = line.strip()
            if line and not line.startswith("#") and not line.startswith("-"):
                pkg = re.split(r"[><=!~\[]", line)[0].strip()
                if pkg:
                    packages.append(pkg)
    elif filepath.name == "package.json":
        try:
            pkg_json = json.loads(content)
            for dep_key in ("dependencies", "devDependencies", "peerDependencies"):
                packages.extend(pkg_json.get(dep_key, {}).keys())
        except json.JSONDecodeError as e:
            return {"error": f"Invalid JSON: {e}"}
    elif filepath.name in ("Pipfile",):
        for line in content.splitlines():
            line = line.strip()
            if "=" in line and not line.startswith("[") and not line.startswith("#"):
                pkg = line.split("=")[0].strip().strip('"')
                if pkg and not pkg.startswith("["):
                    packages.append(pkg)
    else:
        # Generic: one package per line
        for line in content.splitlines():
            line = line.strip()
            if line and not line.startswith("#"):
                packages.append(line.split()[0])

    packages = list(dict.fromkeys(packages))  # deduplicate preserving order
    logger.info("Found %d packages in %s", len(packages), filepath)

    all_results = {
        "file": str(filepath),
        "registry": registry,
        "packages_scanned": len(packages),
        "scan_results": [],
        "summary": {"high": 0, "medium": 0, "low": 0},
    }

    for pkg in packages:
        result = scan_package(pkg, registry, max_candidates_per_pkg)
        all_results["scan_results"].append(result)
        for r in result.get("results", []):
            risk = r.get("risk", "LOW").lower()
            all_results["summary"][risk] = all_results["summary"].get(risk, 0) + 1

    return all_results


def generate_report(data, output_path):
    """Write scan results to a JSON report file."""
    report = {
        "report_generated": datetime.now(timezone.utc).isoformat(),
        **data,
    }
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("Report written to %s", output_path)


def main():
    parser = argparse.ArgumentParser(
        description="Typosquatting Detection Agent for npm and PyPI"
    )
    sub = parser.add_subparsers(dest="command", required=True)

    # scan single package
    scan_p = sub.add_parser("scan", help="Scan for typosquats of a single package")
    scan_p.add_argument("package", help="Target package name to scan for typosquats")
    scan_p.add_argument("--registry", choices=["pypi", "npm"], default="pypi",
                        help="Package registry to scan (default: pypi)")
    scan_p.add_argument("--max-candidates", type=int, help="Limit candidates to check")

    # scan dependency file
    file_p = sub.add_parser("scan-file", help="Scan dependencies from a file")
    file_p.add_argument("file", help="Path to requirements.txt, package.json, etc.")
    file_p.add_argument("--registry", choices=["pypi", "npm"], default="pypi",
                        help="Package registry to scan (default: pypi)")
    file_p.add_argument("--max-candidates", type=int, help="Limit candidates per package")

    # check single candidate
    check_p = sub.add_parser("check", help="Check a specific package name against a target")
    check_p.add_argument("candidate", help="Candidate package name to check")
    check_p.add_argument("target", help="Legitimate target package name")
    check_p.add_argument("--registry", choices=["pypi", "npm"], default="pypi")

    # generate candidates only (no registry queries)
    gen_p = sub.add_parser("generate", help="Generate typosquat candidates without querying registry")
    gen_p.add_argument("package", help="Package name to generate candidates for")

    parser.add_argument("--output", default="typosquat_report.json", help="Output report path")
    args = parser.parse_args()

    result = {}

    if args.command == "scan":
        result = scan_package(args.package, args.registry, args.max_candidates)

    elif args.command == "scan-file":
        result = scan_dependency_file(args.file, args.registry, args.max_candidates)

    elif args.command == "check":
        if args.registry == "pypi":
            candidate_meta = query_pypi_package(args.candidate)
            target_meta = query_pypi_package(args.target)
        else:
            candidate_meta = query_npm_package(args.candidate)
            target_meta = query_npm_package(args.target)

        if not candidate_meta:
            result = {"candidate": args.candidate, "exists": False, "risk": "NONE"}
        elif not target_meta:
            result = {"error": f"Target package '{args.target}' not found"}
        else:
            score, signals = compute_suspicion_score(
                candidate_meta, target_meta, args.target, args.registry
            )
            result = {
                "candidate": args.candidate,
                "target": args.target,
                "registry": args.registry,
                "score": score,
                "risk": classify_risk(score),
                "signals": signals,
                "candidate_metadata": candidate_meta,
                "target_metadata": target_meta,
            }

    elif args.command == "generate":
        candidates = generate_typosquat_candidates(args.package)
        result = {
            "package": args.package,
            "candidate_count": len(candidates),
            "candidates": candidates,
        }

    print(json.dumps(result, indent=2, default=str))
    generate_report(result, args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/generating-and-analyzing-sboms/SKILL.md =====
---
name: generating-and-analyzing-sboms
description: Generate CycloneDX and SPDX SBOMs from container images and filesystems with Syft, correlate them to CVEs with Grype, and sign/attest them with Cosign. Use when you need a machine-readable dependency inventory for supply-chain risk, want to scan images or SBOMs for known vulnerabilities, or are embedding SBOM generation and vulnerability gating into CI/CD.
domain: cybersecurity
subdomain: supply-chain-security
tags:
- supply-chain-security
- sbom
- cyclonedx
- spdx
- syft
- grype
- vulnerability-management
- devsecops
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.AM-08
mitre_attack:
- T1195.001
---
# Generating and Analyzing SBOMs

> **Authorized Use Only:** Generate and scan SBOMs only for software and images you own or are authorized to assess. Treat SBOMs as sensitive inventory data — they reveal your dependency attack surface.

## Overview

A Software Bill of Materials (SBOM) is a formal, machine-readable inventory of every component, library, and dependency in a piece of software — the supply-chain equivalent of an ingredients label. SBOMs are central to defending against supply-chain compromise (CISA's SBOM initiative, US Executive Order 14028) because you cannot patch what you cannot see. The two dominant SBOM standards are:

- **CycloneDX** — an OWASP standard optimized for security use cases (vulnerabilities, VEX, dependency relationships).
- **SPDX** — a Linux Foundation / ISO standard (ISO/IEC 5962) strong on licensing and provenance.

The reference open-source toolchain is from Anchore:

- **Syft** generates SBOMs (CycloneDX, SPDX, or its native format) from container images and filesystems.
- **Grype** matches an SBOM (or image) against vulnerability databases to find CVEs.
- **Cosign** (Sigstore) signs SBOMs and attaches them to images as signed attestations for tamper-evident provenance.

This skill covers producing standards-compliant SBOMs, correlating them with vulnerability intelligence, and embedding the workflow into CI/CD.

## When to Use

- Establishing and maintaining a component inventory for applications and container images.
- Continuously detecting known vulnerabilities (including newly disclosed CVEs against existing artifacts).
- Satisfying procurement/regulatory SBOM requirements (CISA, EO 14028).
- Producing signed SBOM attestations for downstream supply-chain trust.

## Prerequisites

- Install Syft and Grype (official install scripts):
  ```bash
  curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
  curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin
  ```
- Install Cosign for signing/attestation:
  ```bash
  # via Go, or download a release from https://github.com/sigstore/cosign/releases
  go install github.com/sigstore/cosign/v2/cmd/cosign@latest
  ```
- Access to the target images/source and (for signing) a registry plus keys or keyless OIDC.

## Objectives

- Generate CycloneDX and SPDX SBOMs from images and directories.
- Scan SBOMs and images for vulnerabilities with Grype.
- Gate CI/CD builds on severity thresholds.
- Sign and attach SBOM attestations with Cosign and verify them.

## MITRE ATT&CK Mapping

| ID | Official Technique Name | Relevance to this skill |
|----|------------------------|--------------------------|
| T1195.001 | Supply Chain Compromise: Compromise Software Dependencies and Development Tools | SBOM generation and vulnerability correlation expose compromised or vulnerable dependencies — the attack surface adversaries abuse under this technique. |

This is a defensive supply-chain skill; the mapping reflects the adversary technique it is designed to detect and mitigate.

## Workflow

### 1. Generate a CycloneDX SBOM from a container image
`-o <format>` selects output; `cyclonedx-json` is security-oriented.
```bash
syft alpine:latest -o cyclonedx-json=alpine.cdx.json
```

### 2. Generate an SPDX SBOM from a source directory
Use the `dir:` source to inventory a checked-out repository; `spdx-json` for the SPDX standard.
```bash
syft dir:. -o spdx-json=app.spdx.json
```

### 3. Emit multiple formats at once
Produce both standards in a single pass for different consumers.
```bash
syft myorg/app:1.4.2 \
  -o cyclonedx-json=app.cdx.json \
  -o spdx-json=app.spdx.json \
  -o table
```

### 4. Scan the SBOM for vulnerabilities with Grype
Decoupling generation from scanning lets you re-scan stored SBOMs as new CVEs land — without rebuilding.
```bash
# Scan an existing SBOM
grype sbom:app.cdx.json -o table

# JSON report for automation
grype sbom:app.cdx.json -o json > app.vulns.json
```
You can also scan an image directly (Grype generates the SBOM internally):
```bash
grype myorg/app:1.4.2 -o table
```

### 5. Gate CI/CD on severity
`--fail-on` exits non-zero at or above a severity, failing the pipeline.
```bash
grype sbom:app.cdx.json --fail-on high
```
Filter out unfixable noise with a `.grype.yaml` policy (`only-fixed: true`) or `--only-fixed`:
```bash
grype sbom:app.cdx.json --only-fixed --fail-on critical
```

### 6. Sign and attach the SBOM as an attestation
Cosign records the SBOM as a signed, in-toto attestation alongside the image in the registry.
```bash
# Key-based signing
cosign attest --key cosign.key \
  --predicate app.spdx.json \
  --type spdxjson \
  myorg/app:1.4.2

# Keyless (Sigstore OIDC / Fulcio + Rekor)
COSIGN_EXPERIMENTAL=1 cosign attest \
  --predicate app.cdx.json \
  --type cyclonedx \
  myorg/app:1.4.2
```

### 7. Verify the attestation downstream
Consumers verify provenance before trusting an image.
```bash
cosign verify-attestation --key cosign.pub --type spdxjson myorg/app:1.4.2
```

### 8. Retrieve and re-scan attached SBOMs
Pull the attested SBOM from the registry and re-run Grype as part of continuous monitoring.
```bash
cosign download attestation myorg/app:1.4.2 \
  | jq -r '.payload' | base64 -d | jq '.predicate' > pulled.spdx.json
grype sbom:pulled.spdx.json -o table
```

### 9. Correlate to vulnerability intelligence
Feed Grype JSON into your vulnerability management workflow: deduplicate by CVE, enrich with EPSS/KEV for prioritization, and track remediation SLAs. Re-scan stored SBOMs on each Grype DB update to catch newly disclosed CVEs in unchanged artifacts.

## Tools and Resources

| Tool | Purpose | Link |
|------|---------|------|
| Syft | SBOM generation | https://github.com/anchore/syft |
| Grype | Vulnerability scanning of SBOMs/images | https://github.com/anchore/grype |
| Cosign | SBOM signing/attestation | https://github.com/sigstore/cosign |
| CycloneDX | Security-focused SBOM standard | https://cyclonedx.org/ |
| SPDX | ISO SBOM standard | https://spdx.dev/ |
| CISA SBOM | Guidance and minimum elements | https://www.cisa.gov/sbom |

## Format Comparison

| Aspect | CycloneDX | SPDX |
|--------|-----------|------|
| Steward | OWASP | Linux Foundation / ISO 5962 |
| Strength | Security, VEX, vulnerabilities | Licensing, provenance |
| Common syft `-o` values | `cyclonedx-json`, `cyclonedx-xml` | `spdx-json`, `spdx` (tag-value) |

## Validation Criteria

- [ ] CycloneDX SBOM generated from the target image
- [ ] SPDX SBOM generated from source where required
- [ ] SBOM scanned with Grype producing a CVE report
- [ ] CI/CD gated with `--fail-on` at an agreed severity
- [ ] SBOM signed and attached as an attestation with Cosign
- [ ] Attestation verified downstream
- [ ] Stored SBOMs re-scanned on Grype DB updates
- [ ] Findings correlated/prioritized (EPSS/KEV) and tracked to remediation

===== END FILE =====


===== FILE: skills/generating-and-analyzing-sboms/LICENSE =====

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


===== FILE: skills/generating-and-analyzing-sboms/references/api-reference.md =====
# SBOM Toolchain Command Reference

## Syft (SBOM generation)

Source prefixes: `<image>` (default = container image), `dir:<path>`, `file:<path>`,
`registry:<image>`, `docker:<image>`, `oci-archive:<path>`.

| Flag / form | Purpose |
|-------------|---------|
| `-o <format>[=<file>]` | Output format and optional file |
| `--scope <squashed\|all-layers>` | Layer scope for images |
| `--exclude <glob>` | Exclude paths |
| `syft <src> -o table` | Human-readable summary |

Common `-o` formats: `cyclonedx-json`, `cyclonedx-xml`, `spdx-json`, `spdx` (tag-value), `syft-json`, `table`.

```bash
syft alpine:latest -o cyclonedx-json=alpine.cdx.json
syft dir:. -o spdx-json=app.spdx.json
syft myorg/app:1.4.2 -o cyclonedx-json=app.cdx.json -o spdx-json=app.spdx.json -o table
```

## Grype (vulnerability scanning)

Source prefixes: `sbom:<file>`, `<image>`, `dir:<path>`, `registry:<image>`.

| Flag | Purpose |
|------|---------|
| `-o <format>` | `table`, `json`, `cyclonedx`, `sarif` |
| `--fail-on <severity>` | Exit non-zero at/above severity (`low\|medium\|high\|critical`) |
| `--only-fixed` | Report only vulns with a fix available |
| `--add-cpes-if-none` | Improve matching for SBOMs lacking CPEs |
| `db update` | Update the vulnerability database |

```bash
grype sbom:app.cdx.json -o table
grype sbom:app.cdx.json -o json > app.vulns.json
grype sbom:app.cdx.json --only-fixed --fail-on critical
grype myorg/app:1.4.2 -o table
grype db update
```

## Cosign (signing / attestation)

| Command | Purpose |
|---------|---------|
| `cosign attest --key <key> --predicate <sbom> --type <type> <image>` | Attach signed SBOM attestation |
| `cosign verify-attestation --key <pub> --type <type> <image>` | Verify attestation |
| `cosign download attestation <image>` | Retrieve attached attestation |
| `cosign generate-key-pair` | Create signing keys |

`--type` values: `spdxjson`, `cyclonedx`, `slsaprovenance`, or a custom URI.
Keyless mode: set `COSIGN_EXPERIMENTAL=1` and omit `--key` (uses Fulcio/Rekor).

```bash
cosign attest --key cosign.key --predicate app.spdx.json --type spdxjson myorg/app:1.4.2
cosign verify-attestation --key cosign.pub --type spdxjson myorg/app:1.4.2
cosign download attestation myorg/app:1.4.2
```

## Policy file (`.grype.yaml`)

```yaml
only-fixed: true
fail-on-severity: high
ignore:
  - vulnerability: CVE-2024-0000   # documented, risk-accepted
```

===== END FILE =====


===== FILE: skills/generating-and-analyzing-sboms/references/standards.md =====
# Standards and Framework Mapping — Generating and Analyzing SBOMs

## NIST Cybersecurity Framework 2.0

| ID | Name | Rationale |
|----|------|-----------|
| ID.AM-08 | Systems, hardware, software, services, and data are managed throughout their life cycles | SBOMs are the authoritative software-component inventory that underpins lifecycle asset management and supply-chain risk visibility. |

## MITRE ATT&CK

| ID | Name | Rationale |
|----|------|-----------|
| T1195.001 | Supply Chain Compromise: Compromise Software Dependencies and Development Tools | SBOM generation plus vulnerability correlation surfaces vulnerable/compromised dependencies, directly countering this technique. |

## SBOM Standards and Authorities

| Standard / Authority | Role |
|----------------------|------|
| CycloneDX (OWASP) | Security-focused SBOM format (VEX, vulnerabilities) |
| SPDX (ISO/IEC 5962) | Licensing/provenance-focused SBOM format |
| CISA SBOM Minimum Elements | Baseline required SBOM fields |
| US Executive Order 14028 | Mandates SBOMs for software sold to the US government |
| NTIA "Framing Software Component Transparency" | Foundational SBOM guidance |

## Supporting References

- CISA SBOM: https://www.cisa.gov/sbom
- CycloneDX: https://cyclonedx.org/
- SPDX: https://spdx.dev/
- Syft: https://github.com/anchore/syft
- Grype: https://github.com/anchore/grype
- Sigstore Cosign: https://github.com/sigstore/cosign

===== END FILE =====


===== FILE: skills/generating-and-analyzing-sboms/scripts/agent.py =====
#!/usr/bin/env python3
"""
SBOM generation + vulnerability correlation helper.

Wraps Syft (SBOM generation), Grype (vulnerability scanning), and Cosign
(attestation) with real flags. Can run the full pipeline (generate -> scan ->
optionally attest) and summarize Grype JSON by severity for CI gating.

References:
  https://github.com/anchore/syft
  https://github.com/anchore/grype
  https://github.com/sigstore/cosign
"""
import argparse
import json
import shutil
import subprocess
import sys
from collections import Counter

SEVERITY_ORDER = ["negligible", "low", "medium", "high", "critical"]


def require(tool):
    path = shutil.which(tool)
    if not path:
        print(f"[!] '{tool}' not found on PATH", file=sys.stderr)
    return path


def run(cmd, dry_run, capture=False):
    print("[*] " + " ".join(cmd), file=sys.stderr)
    if dry_run:
        return 0, ""
    try:
        if capture:
            proc = subprocess.run(cmd, check=False, text=True, capture_output=True)
            if proc.returncode != 0 and proc.stderr:
                print(proc.stderr, file=sys.stderr)
            return proc.returncode, proc.stdout
        return subprocess.run(cmd, check=False).returncode, ""
    except FileNotFoundError:
        print(f"[!] command not found: {cmd[0]}", file=sys.stderr)
        return 127, ""


def syft_generate(source, fmt, outfile, dry_run):
    syft = require("syft") or "syft"
    cmd = [syft, source, "-o", f"{fmt}={outfile}"]
    rc, _ = run(cmd, dry_run)
    return rc


def grype_scan(sbom_path, out_json, fail_on=None, only_fixed=False, dry_run=False):
    grype = require("grype") or "grype"
    cmd = [grype, f"sbom:{sbom_path}", "-o", "json"]
    if only_fixed:
        cmd.append("--only-fixed")
    if fail_on:
        cmd += ["--fail-on", fail_on]
    rc, out = run(cmd, dry_run, capture=True)
    if out and not dry_run:
        with open(out_json, "w", encoding="utf-8") as fh:
            fh.write(out)
    return rc, out


def summarize(grype_json_text):
    """Count vulnerabilities by severity from Grype JSON output."""
    try:
        data = json.loads(grype_json_text)
    except (json.JSONDecodeError, TypeError):
        print("[!] could not parse Grype JSON", file=sys.stderr)
        return Counter()
    counts = Counter()
    for match in data.get("matches", []):
        sev = (match.get("vulnerability", {})
                    .get("severity", "Unknown")).lower()
        counts[sev] += 1
    return counts


def print_summary(counts):
    total = sum(counts.values())
    print(f"[+] Total vulnerabilities: {total}")
    for sev in reversed(SEVERITY_ORDER):
        if counts.get(sev):
            print(f"    {sev:>10}: {counts[sev]}")
    for sev, n in counts.items():
        if sev not in SEVERITY_ORDER:
            print(f"    {sev:>10}: {n}")


def cosign_attest(image, predicate, attest_type, key, dry_run):
    cosign = require("cosign") or "cosign"
    cmd = [cosign, "attest", "--predicate", predicate, "--type", attest_type, image]
    if key:
        cmd[2:2] = ["--key", key]  # insert after 'attest'
    rc, _ = run(cmd, dry_run)
    return rc


def main():
    p = argparse.ArgumentParser(description="SBOM generate/scan/attest helper")
    p.add_argument("--dry-run", action="store_true")
    sub = p.add_subparsers(dest="cmd", required=True)

    g = sub.add_parser("generate", help="Generate an SBOM with Syft")
    g.add_argument("--source", required=True, help="image, dir:., file:..., etc.")
    g.add_argument("--format", default="cyclonedx-json")
    g.add_argument("--out", required=True)

    s = sub.add_parser("scan", help="Scan an SBOM with Grype")
    s.add_argument("--sbom", required=True)
    s.add_argument("--out", default="vulns.json")
    s.add_argument("--fail-on")
    s.add_argument("--only-fixed", action="store_true")

    pl = sub.add_parser("pipeline", help="generate -> scan -> summarize (+attest)")
    pl.add_argument("--source", required=True)
    pl.add_argument("--format", default="cyclonedx-json")
    pl.add_argument("--sbom-out", default="sbom.json")
    pl.add_argument("--vulns-out", default="vulns.json")
    pl.add_argument("--fail-on")
    pl.add_argument("--only-fixed", action="store_true")
    pl.add_argument("--attest-image", help="If set, attest the SBOM to this image")
    pl.add_argument("--attest-type", default="cyclonedx")
    pl.add_argument("--cosign-key")

    args = p.parse_args()

    if args.cmd == "generate":
        return syft_generate(args.source, args.format, args.out, args.dry_run)

    if args.cmd == "scan":
        rc, out = grype_scan(args.sbom, args.out, args.fail_on,
                             args.only_fixed, args.dry_run)
        if out:
            print_summary(summarize(out))
        return rc

    if args.cmd == "pipeline":
        rc = syft_generate(args.source, args.format, args.sbom_out, args.dry_run)
        if rc not in (0,) and not args.dry_run:
            print("[!] SBOM generation failed", file=sys.stderr)
            return rc
        scan_rc, out = grype_scan(args.sbom_out, args.vulns_out, args.fail_on,
                                  args.only_fixed, args.dry_run)
        if out:
            print_summary(summarize(out))
        if args.attest_image:
            cosign_attest(args.attest_image, args.sbom_out, args.attest_type,
                          args.cosign_key, args.dry_run)
        return scan_rc
    return 2


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/implementing-sigstore-for-software-signing/SKILL.md =====
---
name: implementing-sigstore-for-software-signing
description: 'Implements Sigstore-based software signing and verification using Cosign
  keyless signing, Rekor transparency log verification, and Fulcio certificate authority
  integration to establish cryptographic provenance for container images, binaries,
  and software artifacts, including OIDC-based identity binding and CI/CD pipeline
  integration. Use when setting up keyless container/artifact signing, verifying
  signatures against the Rekor transparency log, or deploying Sigstore in a supply-chain
  security workflow.

  '
domain: cybersecurity
subdomain: supply-chain-security
tags:
- sigstore
- cosign
- rekor
- fulcio
- software-signing
- supply-chain
- keyless-signing
- OIDC
- transparency-log
version: 1.0.0
author: mukul975
license: Apache-2.0
nist_csf:
- GV.SC-01
- GV.SC-03
- GV.SC-06
- GV.SC-07
mitre_attack:
- T1078
- T1190
- T1059
- T1610
- T1611
mitre_f3:
  version: '1.1'
  tactics:
  - resource-development
  - initial-access
  - stealth
  techniques:
  - id: T1195
    name: Supply Chain Compromise
    tactic: initial-access
    source: attack
  - id: T1608
    name: Stage Capabilities
    tactic: resource-development
    source: attack
  - id: T1608.006
    name: 'Stage Capabilities: SEO Poisoning'
    tactic: resource-development
    source: attack
  - id: T1586
    name: Compromise Accounts
    tactic: resource-development
    source: attack
  - id: T1070
    name: Indicator Removal
    tactic: stealth
    source: attack
---
# Implementing Sigstore for Software Signing

## When to Use

- Signing container images and software artifacts without managing long-lived cryptographic keys
- Establishing verifiable provenance for build outputs in CI/CD pipelines using OIDC identity binding
- Querying the Rekor transparency log to audit when and by whom an artifact was signed
- Verifying that container images pulled from registries were signed by authorized identities and issuers
- Integrating Sigstore verification into Kubernetes admission controllers to enforce signed-image policies

**Do not use** for signing artifacts that require air-gapped or offline signing workflows where OIDC authentication is unavailable, for environments that cannot reach the public Sigstore infrastructure (Fulcio, Rekor) and have no private instance deployed, or as a replacement for traditional PGP/GPG signing where regulatory compliance mandates specific key management procedures.

## Prerequisites

- Cosign CLI v2.4+ installed (`go install github.com/sigstore/cosign/v2/cmd/cosign@latest` or binary release)
- Access to an OIDC identity provider supported by Fulcio (Google, GitHub, Microsoft, or a custom OIDC issuer)
- Container registry credentials (for signing container images) with push access to store signature objects
- Python 3.9+ with `sigstore`, `requests`, and `cryptography` packages for the automation agent
- Network access to `fulcio.sigstore.dev`, `rekor.sigstore.dev`, and `tuf-repo-cdn.sigstore.dev` (or private Sigstore instance URLs)

## Workflow

### Step 1: Install and Configure Cosign

Install Cosign and verify it can reach the Sigstore infrastructure:

- **Install from binary release**: Download the appropriate binary from the Cosign GitHub releases page and verify its checksum. On Linux: `curl -LO https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64 && chmod +x cosign-linux-amd64 && sudo mv cosign-linux-amd64 /usr/local/bin/cosign`
- **Verify installation**: Run `cosign version` to confirm the version and check connectivity to Sigstore services with `cosign initialize` which fetches the TUF root of trust
- **Configure custom infrastructure** (optional): If running a private Sigstore stack, set `--fulcio-url`, `--rekor-url`, and `--oidc-issuer` flags or use environment variables `COSIGN_REKOR_URL` and `COSIGN_FULCIO_URL`

### Step 2: Keyless Signing with Cosign and Fulcio

Perform identity-based signing where Fulcio issues a short-lived certificate bound to your OIDC identity:

- **Sign a container image**: Run `cosign sign <IMAGE_DIGEST>` which triggers an OIDC authentication flow. Cosign generates an ephemeral key pair, obtains a short-lived certificate from Fulcio binding the public key to the OIDC identity, signs the image digest, and records the signing event in Rekor. The private key is destroyed immediately after signing.
- **Sign a blob (file)**: Run `cosign sign-blob <file> --bundle artifact.sigstore.json` to sign arbitrary files. The bundle contains the signature, certificate, timestamp, and Rekor inclusion proof.
- **Non-interactive signing in CI**: Set `SIGSTORE_ID_TOKEN` environment variable with a valid OIDC token (e.g., from GitHub Actions OIDC or GCP workload identity) to skip the browser-based authentication flow:
  ```bash
  export SIGSTORE_ID_TOKEN=$(curl -sH "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
    "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=sigstore" | jq -r '.value')
  cosign sign $IMAGE_DIGEST
  ```
- **Supported OIDC providers**: Google (`https://accounts.google.com`), GitHub (`https://github.com/login/oauth`), Microsoft (`https://login.microsoftonline.com`), GitLab (`https://gitlab.com`), and custom providers registered with a private Fulcio instance

### Step 3: Verify Signed Artifacts

Verify that artifacts were signed by expected identities from expected OIDC issuers:

- **Verify a container image**: Run `cosign verify <IMAGE_URI> --certificate-identity=name@example.com --certificate-oidc-issuer=https://accounts.google.com` to confirm the image was signed by the specified identity. Cosign validates the certificate chain, checks the Rekor inclusion proof, and verifies the signature matches the current image digest.
- **Verify a signed blob**: Run `cosign verify-blob <file> --bundle artifact.sigstore.json --certificate-identity=name@example.com --certificate-oidc-issuer=https://accounts.google.com`
- **Regex matching for CI identities**: Use `--certificate-identity-regexp` to match CI workflow identities:
  ```bash
  cosign verify $IMAGE --certificate-identity-regexp="https://github.com/myorg/myrepo/.*" \
    --certificate-oidc-issuer=https://token.actions.githubusercontent.com
  ```
- **Verification failure modes**: Cosign returns a non-zero exit code on failure. Common failures include certificate identity mismatch, expired certificates without a valid Rekor timestamp, missing Rekor entry, and image digest mismatch (image was modified after signing).

### Step 4: Query the Rekor Transparency Log

Search and verify entries in the Rekor transparency log to audit signing events:

- **Search by email identity**: Use `rekor-cli search --email user@example.com` to find all signing events for an identity
- **Search by artifact hash**: Use `rekor-cli search --sha sha256:<hash>` to find signing events for a specific artifact
- **Retrieve and verify an entry**: Use `rekor-cli get --uuid <entry_uuid>` to retrieve full entry details including the certificate, signature, and artifact hash
- **Verify log inclusion**: Use `rekor-cli verify --entry-uuid <uuid>` to verify the entry's inclusion proof against the signed tree head, confirming the entry exists in the append-only log and has not been tampered with
- **REST API queries**: Query `https://rekor.sigstore.dev/api/v1/index/retrieve` with POST body `{"hash": "sha256:<hash>"}` to retrieve entry UUIDs, then fetch full entries from `/api/v1/log/entries/<uuid>`
- **Monitor for consistency**: Use the rekor-monitor tool or Omniwitness to continuously verify the log remains append-only and entries are never mutated or removed

### Step 5: Integrate into CI/CD Pipelines

Embed signing and verification into build and deployment pipelines:

- **GitHub Actions**: Use `sigstore/cosign-installer` action to install Cosign, then sign images using the GitHub OIDC token as the identity. The signing identity will be the workflow URL (e.g., `https://github.com/org/repo/.github/workflows/build.yml@refs/heads/main`).
- **Kubernetes admission enforcement**: Deploy Sigstore Policy Controller or Kyverno with Cosign verification policies to reject unsigned or incorrectly signed images at admission time
- **Supply chain metadata**: Use `cosign attest` to attach in-toto attestations (SLSA provenance, SBOM, vulnerability scan results) to images, signed with the same keyless flow, enabling consumers to verify both the artifact and its build metadata

## Key Concepts

| Term | Definition |
|------|------------|
| **Keyless Signing** | Identity-based signing that uses short-lived certificates from Fulcio bound to OIDC identities instead of long-lived cryptographic keys, eliminating key management overhead |
| **Fulcio** | Sigstore's certificate authority that issues short-lived X.509 certificates after verifying OIDC tokens, binding an ephemeral public key to a verified identity |
| **Rekor** | Sigstore's immutable, append-only transparency log that records signing events with timestamps, enabling auditors to verify when and by whom an artifact was signed |
| **Cosign** | The primary CLI tool for signing and verifying container images and blobs using the Sigstore infrastructure (Fulcio + Rekor) |
| **TUF Root of Trust** | The Update Framework distribution mechanism for Sigstore's root CA certificate and Rekor public key, ensuring clients trust the correct Sigstore infrastructure |
| **OIDC Identity Binding** | The process where Fulcio verifies a user's identity through an OpenID Connect token and binds it to a short-lived signing certificate |
| **Inclusion Proof** | A cryptographic proof from Rekor demonstrating that a signing event entry exists within the transparency log's Merkle tree |

## Tools & Systems

- **Cosign**: CLI tool for signing containers and blobs, verifying signatures, and attaching attestations using Sigstore keyless signing or traditional key-based signing
- **Fulcio**: Free root certificate authority for code signing certificates issued based on OIDC identity verification with a validity period of approximately 10 minutes
- **Rekor**: Transparency log server providing tamper-evident storage of signing metadata, searchable by identity, artifact hash, or public key
- **Sigstore Policy Controller**: Kubernetes admission webhook that enforces image signing policies by verifying Cosign signatures and attestations before allowing pod creation
- **rekor-cli**: Command-line client for querying, uploading, and verifying entries in the Rekor transparency log

## Common Scenarios

### Scenario: Securing a Container Image Build Pipeline with Keyless Signing

**Context**: A DevOps team builds container images in GitHub Actions and deploys to a Kubernetes cluster. They need to ensure only images built by their CI pipeline can be deployed, preventing supply chain attacks from compromised registries or unauthorized pushes.

**Approach**:
1. Add `sigstore/cosign-installer@v3` to the GitHub Actions workflow and enable OIDC token permissions with `id-token: write`
2. After building and pushing the image, sign it with `cosign sign $IMAGE_DIGEST` using the GitHub Actions OIDC identity automatically
3. Deploy Sigstore Policy Controller to the Kubernetes cluster with a ClusterImagePolicy requiring signatures from `--certificate-identity-regexp=https://github.com/myorg/myrepo/.*` and `--certificate-oidc-issuer=https://token.actions.githubusercontent.com`
4. Verify the signing entry appears in Rekor by querying with the image digest hash to confirm the transparency log recorded the event
5. Test the admission controller by attempting to deploy an unsigned image and confirming it is rejected with a policy violation error

**Pitfalls**:
- Signing the image tag instead of the digest (`cosign sign myimage:latest` vs `cosign sign myimage@sha256:abc...`) means verification breaks when the tag is updated to point to a different digest
- Not pinning the `--certificate-oidc-issuer` during verification allows signatures from any OIDC provider to pass, defeating the purpose of identity binding
- Forgetting to set `id-token: write` permission in GitHub Actions results in OIDC token retrieval failure and signing errors
- Using `--certificate-identity-regexp=.*` in production verification policies effectively disables identity verification

## Output Format

```
## Sigstore Signing Verification Report

**Artifact**: ghcr.io/myorg/myapp@sha256:a1b2c3d4...
**Verification Status**: PASSED

**Certificate Details**:
  Subject: https://github.com/myorg/myapp/.github/workflows/build.yml@refs/heads/main
  Issuer: https://token.actions.githubusercontent.com
  Valid From: 2026-03-19T10:00:00Z
  Valid To: 2026-03-19T10:10:00Z

**Rekor Entry**:
  UUID: 24296fb24b8ad77a8d52...
  Log Index: 89234567
  Integrated Time: 2026-03-19T10:00:05Z
  Inclusion Proof: VERIFIED (tree size: 92000000, root hash: e4f5a6...)

**Policy Check**: Image signed by authorized CI workflow identity
```

===== END FILE =====


===== FILE: skills/implementing-sigstore-for-software-signing/LICENSE =====

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


===== FILE: skills/implementing-sigstore-for-software-signing/references/api-reference.md =====
# API Reference: Sigstore Software Signing Agent

## Overview

Automates Sigstore-based software signing and verification using Cosign keyless signing, Rekor transparency log queries, and Fulcio certificate authority integration. Wraps the Cosign CLI and Rekor REST API to sign artifacts, verify signatures against expected OIDC identities, search the transparency log, and audit signing events end-to-end.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| requests | >=2.28 | HTTP requests to Rekor REST API |
| cosign | >=2.4 (CLI) | Signing and verification of blobs and container images |
| rekor-cli | >=1.3 (CLI, optional) | Direct Rekor entry verification with inclusion proofs |

## CLI Usage

```bash
# Check cosign installation and Rekor connectivity
python agent.py check

# Sign a file blob (triggers OIDC auth flow)
python agent.py sign-blob myfile.tar.gz --bundle myfile.sigstore.json

# Verify a signed blob
python agent.py verify-blob myfile.tar.gz --bundle myfile.sigstore.json \
  --cert-identity user@example.com \
  --cert-oidc-issuer https://accounts.google.com

# Sign a container image (use digest, not tag)
python agent.py sign-container registry.io/myimage@sha256:abc123...

# Verify a container image
python agent.py verify-container registry.io/myimage@sha256:abc123... \
  --cert-identity user@example.com \
  --cert-oidc-issuer https://accounts.google.com

# Search Rekor by artifact hash
python agent.py search-rekor --hash <sha256-hash>

# Search Rekor by signer email
python agent.py search-rekor --email user@example.com

# Search Rekor by file (computes hash automatically)
python agent.py search-rekor --file myfile.tar.gz

# Retrieve a specific Rekor entry
python agent.py get-rekor-entry <uuid>

# Get Rekor transparency log state
python agent.py log-info

# Full audit of a signing event
python agent.py audit --file myfile.tar.gz \
  --cert-identity user@example.com \
  --cert-oidc-issuer https://accounts.google.com

# All commands support custom output path
python agent.py sign-blob myfile.tar.gz --output custom_report.json
```

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `command` | Yes | Subcommand: `check`, `sign-blob`, `verify-blob`, `sign-container`, `verify-container`, `search-rekor`, `get-rekor-entry`, `log-info`, `audit` |
| `--bundle` | Varies | Path to sigstore bundle file (required for verify-blob, optional for sign-blob) |
| `--cert-identity` | For verify | Expected signer identity (email or workflow URL) |
| `--cert-oidc-issuer` | For verify | Expected OIDC issuer URL (e.g., `https://accounts.google.com`) |
| `--rekor-url` | No | Custom Rekor server URL (default: `https://rekor.sigstore.dev`) |
| `--output` | No | Output report path (default: `sigstore_report.json`) |

## Key Functions

### `sign_blob_keyless(filepath, bundle_path)`
Signs a file using Cosign keyless signing. Triggers OIDC authentication, obtains a Fulcio certificate, records the event in Rekor, and outputs a sigstore bundle containing the signature, certificate, and inclusion proof.

### `verify_blob_keyless(filepath, bundle_path, cert_identity, cert_oidc_issuer)`
Verifies a signed blob against the expected certificate identity and OIDC issuer. Validates the certificate chain, Rekor inclusion proof, and signature integrity.

### `sign_container_keyless(image_uri)`
Signs a container image by digest using keyless signing. The signature is stored as an OCI artifact attached to the image in the registry.

### `verify_container_keyless(image_uri, cert_identity, cert_oidc_issuer)`
Verifies container image signatures and returns parsed verification details including all matching signatures.

### `search_rekor_by_hash(artifact_hash, rekor_url)`
Queries the Rekor REST API `POST /api/v1/index/retrieve` with a SHA-256 hash to find all log entries for an artifact.

### `search_rekor_by_email(email, rekor_url)`
Queries Rekor for all signing events associated with an email identity.

### `get_rekor_entry(uuid, rekor_url)`
Retrieves a specific Rekor log entry by UUID from `GET /api/v1/log/entries/<uuid>`, parsing log index, integrated time, inclusion proof presence, and signed entry timestamp.

### `get_rekor_log_info(rekor_url)`
Retrieves the current Rekor log state from `GET /api/v1/log`, including tree size, root hash, and signed tree head.

### `audit_signing_event(filepath, image_uri, cert_identity, cert_oidc_issuer, rekor_url)`
Performs a comprehensive audit combining artifact hash computation, Rekor log search, entry detail retrieval, inclusion proof verification, and signature verification into a single pass/fail report.

## Rekor REST API Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v1/log` | GET | Retrieve current log state (tree size, root hash) |
| `/api/v1/index/retrieve` | POST | Search entries by hash or email |
| `/api/v1/log/entries/<uuid>` | GET | Retrieve a specific log entry |

## Common OIDC Issuers

| Provider | Issuer URL |
|----------|-----------|
| Google | `https://accounts.google.com` |
| GitHub Actions | `https://token.actions.githubusercontent.com` |
| Microsoft | `https://login.microsoftonline.com` |
| GitLab | `https://gitlab.com` |

===== END FILE =====


===== FILE: skills/implementing-sigstore-for-software-signing/scripts/agent.py =====
#!/usr/bin/env python3
"""Sigstore Software Signing Agent - Automates cosign keyless signing, Rekor
transparency log verification, and Fulcio certificate inspection for container
images and software artifacts."""

import json
import logging
import argparse
import subprocess
import hashlib
import sys
from datetime import datetime, timezone
from pathlib import Path

import requests

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

REKOR_PUBLIC_URL = "https://rekor.sigstore.dev"
FULCIO_PUBLIC_URL = "https://fulcio.sigstore.dev"


def compute_sha256(filepath):
    """Compute SHA-256 hash of a file."""
    sha256 = hashlib.sha256()
    with open(filepath, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            sha256.update(chunk)
    return sha256.hexdigest()


def run_cosign(args, capture=True):
    """Execute a cosign CLI command and return the result."""
    cmd = ["cosign"] + args
    logger.info("Running: %s", " ".join(cmd))
    result = subprocess.run(cmd, capture_output=capture, text=True, timeout=120)
    if result.returncode != 0:
        logger.error("cosign failed (exit %d): %s", result.returncode, result.stderr)
    return result


def check_cosign_installed():
    """Verify cosign CLI is available and return version info."""
    result = run_cosign(["version"])
    if result.returncode != 0:
        logger.error("cosign is not installed or not in PATH")
        return None
    version_line = ""
    for line in result.stdout.splitlines():
        if "cosign" in line.lower() or "GitVersion" in line:
            version_line = line.strip()
            break
    return version_line or result.stdout.strip()


def sign_blob_keyless(filepath, bundle_path=None):
    """Sign a file blob using cosign keyless signing with Fulcio and Rekor.

    This triggers an OIDC authentication flow. In CI, set SIGSTORE_ID_TOKEN
    environment variable to provide the identity token non-interactively.
    """
    filepath = Path(filepath)
    if not filepath.exists():
        return {"error": f"File not found: {filepath}", "signed": False}

    if bundle_path is None:
        bundle_path = str(filepath) + ".sigstore.json"

    args = ["sign-blob", str(filepath), "--bundle", bundle_path, "--yes"]
    result = run_cosign(args)

    if result.returncode == 0:
        logger.info("Blob signed successfully: %s", filepath)
        bundle_data = {}
        try:
            with open(bundle_path, "r") as f:
                bundle_data = json.load(f)
        except (json.JSONDecodeError, FileNotFoundError):
            pass
        return {
            "signed": True,
            "file": str(filepath),
            "bundle": bundle_path,
            "sha256": compute_sha256(filepath),
            "has_rekor_entry": "rekorBundle" in bundle_data
            or "verificationMaterial" in bundle_data,
        }
    return {"signed": False, "file": str(filepath), "error": result.stderr.strip()}


def verify_blob_keyless(filepath, bundle_path, cert_identity, cert_oidc_issuer):
    """Verify a signed blob against expected identity and OIDC issuer."""
    filepath = Path(filepath)
    if not filepath.exists():
        return {"error": f"File not found: {filepath}", "verified": False}

    args = [
        "verify-blob",
        str(filepath),
        "--bundle",
        bundle_path,
        "--certificate-identity",
        cert_identity,
        "--certificate-oidc-issuer",
        cert_oidc_issuer,
    ]
    result = run_cosign(args)

    return {
        "verified": result.returncode == 0,
        "file": str(filepath),
        "certificate_identity": cert_identity,
        "certificate_oidc_issuer": cert_oidc_issuer,
        "output": result.stdout.strip() if result.returncode == 0 else result.stderr.strip(),
    }


def sign_container_keyless(image_uri):
    """Sign a container image using cosign keyless signing.

    The image_uri should include the digest (e.g., registry/image@sha256:abc...).
    Signing by tag instead of digest is unreliable because tags are mutable.
    """
    args = ["sign", image_uri, "--yes"]
    result = run_cosign(args)

    return {
        "signed": result.returncode == 0,
        "image": image_uri,
        "output": result.stdout.strip() if result.returncode == 0 else result.stderr.strip(),
    }


def verify_container_keyless(image_uri, cert_identity, cert_oidc_issuer):
    """Verify a container image signature against expected identity and issuer."""
    args = [
        "verify",
        image_uri,
        "--certificate-identity",
        cert_identity,
        "--certificate-oidc-issuer",
        cert_oidc_issuer,
    ]
    result = run_cosign(args)

    verification_details = []
    if result.returncode == 0:
        try:
            verification_details = json.loads(result.stdout)
        except json.JSONDecodeError:
            verification_details = [{"raw_output": result.stdout.strip()}]

    return {
        "verified": result.returncode == 0,
        "image": image_uri,
        "certificate_identity": cert_identity,
        "certificate_oidc_issuer": cert_oidc_issuer,
        "signatures": verification_details,
    }


def search_rekor_by_hash(artifact_hash, rekor_url=None):
    """Search the Rekor transparency log for entries matching an artifact hash.

    Queries the Rekor REST API /api/v1/index/retrieve endpoint.
    """
    base = rekor_url or REKOR_PUBLIC_URL
    url = f"{base}/api/v1/index/retrieve"
    payload = {"hash": f"sha256:{artifact_hash}"}

    try:
        resp = requests.post(url, json=payload, timeout=30)
        resp.raise_for_status()
        uuids = resp.json()
        logger.info("Found %d Rekor entries for hash %s", len(uuids), artifact_hash[:16])
        return {"hash": artifact_hash, "entry_uuids": uuids, "count": len(uuids)}
    except requests.RequestException as e:
        logger.error("Rekor search failed: %s", e)
        return {"hash": artifact_hash, "entry_uuids": [], "error": str(e)}


def search_rekor_by_email(email, rekor_url=None):
    """Search the Rekor transparency log for entries matching an email identity."""
    base = rekor_url or REKOR_PUBLIC_URL
    url = f"{base}/api/v1/index/retrieve"
    payload = {"email": email}

    try:
        resp = requests.post(url, json=payload, timeout=30)
        resp.raise_for_status()
        uuids = resp.json()
        logger.info("Found %d Rekor entries for email %s", len(uuids), email)
        return {"email": email, "entry_uuids": uuids, "count": len(uuids)}
    except requests.RequestException as e:
        logger.error("Rekor search failed: %s", e)
        return {"email": email, "entry_uuids": [], "error": str(e)}


def get_rekor_entry(uuid, rekor_url=None):
    """Retrieve a specific entry from the Rekor transparency log by UUID."""
    base = rekor_url or REKOR_PUBLIC_URL
    url = f"{base}/api/v1/log/entries/{uuid}"

    try:
        resp = requests.get(url, timeout=30)
        resp.raise_for_status()
        entry_data = resp.json()

        parsed = {"uuid": uuid, "raw": entry_data}
        for entry_uuid, entry_body in entry_data.items():
            parsed["log_index"] = entry_body.get("logIndex")
            parsed["integrated_time"] = entry_body.get("integratedTime")
            if parsed["integrated_time"]:
                parsed["integrated_time_iso"] = datetime.fromtimestamp(
                    parsed["integrated_time"], tz=timezone.utc
                ).isoformat()
            verification = entry_body.get("verification", {})
            parsed["has_inclusion_proof"] = "inclusionProof" in verification
            parsed["has_signed_entry_timestamp"] = "signedEntryTimestamp" in verification
            break

        return parsed
    except requests.RequestException as e:
        logger.error("Failed to retrieve Rekor entry %s: %s", uuid, e)
        return {"uuid": uuid, "error": str(e)}


def verify_rekor_entry(uuid, rekor_url=None):
    """Verify a Rekor entry's inclusion proof using the rekor-cli."""
    result = run_cosign(["env"])  # Check if rekor-cli is better
    rekor_result = subprocess.run(
        ["rekor-cli", "verify", "--rekor_server", rekor_url or REKOR_PUBLIC_URL,
         "--entry-uuid", uuid],
        capture_output=True, text=True, timeout=60,
    )
    return {
        "uuid": uuid,
        "inclusion_verified": rekor_result.returncode == 0,
        "output": rekor_result.stdout.strip() if rekor_result.returncode == 0
        else rekor_result.stderr.strip(),
    }


def get_rekor_log_info(rekor_url=None):
    """Retrieve the current Rekor transparency log state (tree size, root hash)."""
    base = rekor_url or REKOR_PUBLIC_URL
    url = f"{base}/api/v1/log"

    try:
        resp = requests.get(url, timeout=30)
        resp.raise_for_status()
        log_info = resp.json()
        return {
            "tree_size": log_info.get("treeSize"),
            "root_hash": log_info.get("rootHash"),
            "signed_tree_head": log_info.get("signedTreeHead"),
            "tree_id": log_info.get("treeID"),
        }
    except requests.RequestException as e:
        logger.error("Failed to get Rekor log info: %s", e)
        return {"error": str(e)}


def audit_signing_event(filepath=None, image_uri=None, cert_identity=None,
                        cert_oidc_issuer=None, rekor_url=None):
    """Perform a complete audit of a signing event: verify the artifact and
    cross-reference against the Rekor transparency log."""
    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "artifact": filepath or image_uri,
        "checks": [],
    }

    # Get Rekor log state
    log_info = get_rekor_log_info(rekor_url)
    report["rekor_log_state"] = log_info

    if filepath:
        artifact_hash = compute_sha256(filepath)
        report["artifact_sha256"] = artifact_hash

        # Search Rekor for this artifact
        rekor_search = search_rekor_by_hash(artifact_hash, rekor_url)
        report["rekor_entries"] = rekor_search
        report["checks"].append({
            "check": "rekor_entry_exists",
            "passed": rekor_search.get("count", 0) > 0,
            "detail": f"Found {rekor_search.get('count', 0)} Rekor entries",
        })

        # Retrieve entry details if found
        if rekor_search.get("entry_uuids"):
            first_uuid = rekor_search["entry_uuids"][0]
            entry_detail = get_rekor_entry(first_uuid, rekor_url)
            report["rekor_entry_detail"] = entry_detail
            report["checks"].append({
                "check": "inclusion_proof_present",
                "passed": entry_detail.get("has_inclusion_proof", False),
                "detail": "Inclusion proof found in Rekor entry"
                if entry_detail.get("has_inclusion_proof")
                else "No inclusion proof in Rekor entry",
            })

        # Verify blob if bundle and identity provided
        bundle_path = str(filepath) + ".sigstore.json"
        if Path(bundle_path).exists() and cert_identity and cert_oidc_issuer:
            verify_result = verify_blob_keyless(
                filepath, bundle_path, cert_identity, cert_oidc_issuer
            )
            report["verification"] = verify_result
            report["checks"].append({
                "check": "signature_verification",
                "passed": verify_result.get("verified", False),
                "detail": "Signature verified against identity and issuer"
                if verify_result.get("verified")
                else verify_result.get("output", "Verification failed"),
            })

    elif image_uri and cert_identity and cert_oidc_issuer:
        verify_result = verify_container_keyless(
            image_uri, cert_identity, cert_oidc_issuer
        )
        report["verification"] = verify_result
        report["checks"].append({
            "check": "container_signature_verification",
            "passed": verify_result.get("verified", False),
            "detail": f"Found {len(verify_result.get('signatures', []))} valid signatures"
            if verify_result.get("verified")
            else "Container signature verification failed",
        })

    # Summary
    passed = sum(1 for c in report["checks"] if c["passed"])
    total = len(report["checks"])
    report["summary"] = {
        "checks_passed": passed,
        "checks_total": total,
        "overall_status": "PASSED" if passed == total and total > 0 else "FAILED",
    }

    return report


def generate_report(results, output_path):
    """Write audit results to a JSON report file."""
    with open(output_path, "w") as f:
        json.dump(results, f, indent=2, default=str)
    logger.info("Report written to %s", output_path)


def main():
    parser = argparse.ArgumentParser(
        description="Sigstore Software Signing Agent - Keyless signing, "
        "Rekor verification, and Fulcio certificate inspection"
    )
    sub = parser.add_subparsers(dest="command", required=True)

    # sign-blob
    sign_blob_p = sub.add_parser("sign-blob", help="Sign a file blob with keyless signing")
    sign_blob_p.add_argument("file", help="Path to file to sign")
    sign_blob_p.add_argument("--bundle", help="Output bundle path (default: <file>.sigstore.json)")

    # verify-blob
    verify_blob_p = sub.add_parser("verify-blob", help="Verify a signed blob")
    verify_blob_p.add_argument("file", help="Path to signed file")
    verify_blob_p.add_argument("--bundle", required=True, help="Path to sigstore bundle")
    verify_blob_p.add_argument("--cert-identity", required=True, help="Expected certificate identity")
    verify_blob_p.add_argument("--cert-oidc-issuer", required=True, help="Expected OIDC issuer URL")

    # sign-container
    sign_cont_p = sub.add_parser("sign-container", help="Sign a container image")
    sign_cont_p.add_argument("image", help="Container image URI (use digest, not tag)")

    # verify-container
    verify_cont_p = sub.add_parser("verify-container", help="Verify a container image signature")
    verify_cont_p.add_argument("image", help="Container image URI")
    verify_cont_p.add_argument("--cert-identity", required=True, help="Expected certificate identity")
    verify_cont_p.add_argument("--cert-oidc-issuer", required=True, help="Expected OIDC issuer URL")

    # search-rekor
    search_p = sub.add_parser("search-rekor", help="Search Rekor transparency log")
    search_group = search_p.add_mutually_exclusive_group(required=True)
    search_group.add_argument("--hash", help="SHA-256 hash of artifact to search")
    search_group.add_argument("--email", help="Email identity to search")
    search_group.add_argument("--file", help="File to compute hash and search")
    search_p.add_argument("--rekor-url", help="Custom Rekor server URL")

    # get-rekor-entry
    entry_p = sub.add_parser("get-rekor-entry", help="Retrieve a Rekor log entry")
    entry_p.add_argument("uuid", help="Rekor entry UUID")
    entry_p.add_argument("--rekor-url", help="Custom Rekor server URL")

    # log-info
    log_p = sub.add_parser("log-info", help="Get Rekor transparency log state")
    log_p.add_argument("--rekor-url", help="Custom Rekor server URL")

    # audit
    audit_p = sub.add_parser("audit", help="Full audit of a signing event")
    audit_group = audit_p.add_mutually_exclusive_group(required=True)
    audit_group.add_argument("--file", help="Path to signed file")
    audit_group.add_argument("--image", help="Container image URI")
    audit_p.add_argument("--cert-identity", help="Expected certificate identity")
    audit_p.add_argument("--cert-oidc-issuer", help="Expected OIDC issuer URL")
    audit_p.add_argument("--rekor-url", help="Custom Rekor server URL")

    # check
    sub.add_parser("check", help="Verify cosign is installed and reachable")

    parser.add_argument("--output", default="sigstore_report.json", help="Output report path")
    args = parser.parse_args()

    result = {}

    if args.command == "check":
        version = check_cosign_installed()
        log_info = get_rekor_log_info()
        result = {
            "cosign_installed": version is not None,
            "cosign_version": version,
            "rekor_reachable": "error" not in log_info,
            "rekor_tree_size": log_info.get("tree_size"),
        }

    elif args.command == "sign-blob":
        result = sign_blob_keyless(args.file, args.bundle)

    elif args.command == "verify-blob":
        result = verify_blob_keyless(
            args.file, args.bundle, args.cert_identity, args.cert_oidc_issuer
        )

    elif args.command == "sign-container":
        result = sign_container_keyless(args.image)

    elif args.command == "verify-container":
        result = verify_container_keyless(
            args.image, args.cert_identity, args.cert_oidc_issuer
        )

    elif args.command == "search-rekor":
        rekor_url = getattr(args, "rekor_url", None)
        if args.hash:
            result = search_rekor_by_hash(args.hash, rekor_url)
        elif args.email:
            result = search_rekor_by_email(args.email, rekor_url)
        elif args.file:
            file_hash = compute_sha256(args.file)
            result = search_rekor_by_hash(file_hash, rekor_url)
            result["file"] = args.file
            result["computed_hash"] = file_hash

    elif args.command == "get-rekor-entry":
        result = get_rekor_entry(args.uuid, getattr(args, "rekor_url", None))

    elif args.command == "log-info":
        result = get_rekor_log_info(getattr(args, "rekor_url", None))

    elif args.command == "audit":
        result = audit_signing_event(
            filepath=getattr(args, "file", None),
            image_uri=getattr(args, "image", None),
            cert_identity=getattr(args, "cert_identity", None),
            cert_oidc_issuer=getattr(args, "cert_oidc_issuer", None),
            rekor_url=getattr(args, "rekor_url", None),
        )

    print(json.dumps(result, indent=2, default=str))
    generate_report(result, args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/verifying-build-provenance-with-slsa-sigstore/SKILL.md =====
---
name: verifying-build-provenance-with-slsa-sigstore
description: Verifies artifact signatures and SLSA provenance using Sigstore's
  cosign (verify, verify-attestation, verify-blob-attestation) and slsa-verifier
  (verify-artifact), enforcing keyless OIDC builder identity and source repo against
  SLSA Build levels. Use in CI/CD before deploying artifacts, when consuming third-party
  attestations, establishing a SLSA Build L3 pipeline, or confirming provenance
  during incident response or admission control.
domain: cybersecurity
subdomain: supply-chain-security
tags:
- supply-chain
- slsa
- sigstore
- cosign
- provenance
- attestation
- keyless-signing
- code-signing
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
mitre_attack:
- T1195
---
# Verifying Build Provenance with SLSA and Sigstore

## Overview

Build-provenance verification answers a question that defeats many supply-chain attacks: *was this artifact actually built from the source I think it was, by the builder I trust, without tampering?* Attackers who compromise a build system, swap a compiled release, or inject a malicious step (as in the SolarWinds and 3CX incidents) produce artifacts that look legitimate but lack verifiable provenance. SLSA (Supply-chain Levels for Software Artifacts, https://slsa.dev) defines Build levels (L1–L3) describing increasing provenance integrity, and Sigstore (https://www.sigstore.dev) provides the signing and transparency infrastructure: **cosign** for signing/verifying artifacts and attestations, **Fulcio** for short-lived keyless certificates bound to an OIDC identity, and **Rekor** as a tamper-evident transparency log.

This skill covers verifying signatures and SLSA provenance with **cosign** (`cosign verify`, `cosign verify-attestation`, `cosign verify-blob-attestation`) and **slsa-verifier** (`slsa-verifier verify-artifact`), enforcing the builder identity (the GitHub Actions workflow that produced the artifact) and the expected source repository. Keyless verification ties trust to an OIDC issuer (e.g., `https://token.actions.githubusercontent.com`) and a certificate identity rather than a long-lived private key.

This maps to MITRE ATT&CK **T1195 — Supply Chain Compromise** (provenance verification detects/blocks tampered artifacts) and NIST CSF **PR.DS-01** (the confidentiality, integrity, and availability of data-at-rest are protected; CSF 2.0 absorbed the retired 1.1 PR.DS-06 integrity-checking outcome here).

## When to Use

- In CI/CD before deploying or promoting any container image or release binary.
- When consuming third-party artifacts (base images, Go/npm releases) that publish attestations.
- When establishing a SLSA Build L3 producer pipeline and enforcing it at the consumer side.
- During incident response to confirm whether a deployed artifact's provenance is intact.
- In admission control (e.g., Kubernetes via policy-controller / Kyverno) to admit only verified images.

## Prerequisites

- **cosign** (Sigstore CLI):
  ```bash
  go install github.com/sigstore/cosign/v2/cmd/cosign@latest
  # or download a release binary from https://github.com/sigstore/cosign/releases
  ```
- **slsa-verifier**:
  ```bash
  go install github.com/slsa-framework/slsa-verifier/v2/cli/slsa-verifier@latest
  # or:
  curl -sSL https://github.com/slsa-framework/slsa-verifier/releases/latest/download/slsa-verifier-linux-amd64 \
    -o /usr/local/bin/slsa-verifier && chmod +x /usr/local/bin/slsa-verifier
  ```
- Network access to Rekor (`https://rekor.sigstore.dev`) and Fulcio for transparency-log verification.
- The artifact plus its provenance/attestation bundle (`.sigstore`, `.intoto.jsonl`, or attached OCI attestation).

## Objectives

- Verify a keyless cosign signature on a container image, pinning OIDC issuer and certificate identity.
- Verify a SLSA provenance attestation on an image with `cosign verify-attestation --type slsaprovenance`.
- Verify a release binary's provenance with `slsa-verifier verify-artifact`, pinning source repo and tag.
- Verify GitHub artifact attestations / blob bundles with `cosign verify-blob-attestation`.
- Gate CI and admission control on successful verification; understand SLSA Build L1–L3.

## MITRE ATT&CK Mapping

| ID | Tactic | Technique Name | Relevance |
|----|--------|----------------|-----------|
| T1195 | Initial Access | Supply Chain Compromise | Verifying provenance and signatures detects artifacts that were tampered with or substituted in the build/distribution chain, preventing supply-chain compromise from reaching deployment. |

## Workflow

### Step 1: Verify a keyless cosign signature on an image
Pin both the OIDC issuer and the certificate identity (the exact workflow that signed). A bare `cosign verify` without identity pinning is meaningless — anyone can sign.

```bash
cosign verify \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  --certificate-identity-regexp "^https://github.com/myorg/myrepo/.github/workflows/.*@refs/tags/v.*" \
  ghcr.io/myorg/myrepo:v1.2.3
```
A non-zero exit or empty result means verification failed — do not deploy.

### Step 2: Verify the SLSA provenance attestation on the image
The signature proves *who* signed; the provenance attestation proves *how it was built*. Verify the in-toto SLSA predicate type.

```bash
cosign verify-attestation \
  --type slsaprovenance \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  --certificate-identity "https://github.com/myorg/myrepo/.github/workflows/build-sign.yml@refs/heads/main" \
  ghcr.io/myorg/myrepo:v1.2.3
```
Supported predicate types include `slsaprovenance`, `slsaprovenance02`, and `slsaprovenance1`.

### Step 3: Inspect the provenance predicate
Decode the verified attestation to confirm the source repo, commit, and builder match expectations.

```bash
cosign verify-attestation --type slsaprovenance \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  --certificate-identity-regexp '.*' \
  ghcr.io/myorg/myrepo:v1.2.3 \
  | jq -r '.payload' | base64 -d | jq '.predicate.buildDefinition.externalParameters, .predicate.runDetails.builder.id'
```

### Step 4: Verify a release binary with slsa-verifier
For downloadable binaries (e.g., produced by `slsa-github-generator`), pin the source URI and the tag. slsa-verifier checks the cryptographic signature on the provenance and that the expected builder produced it.

```bash
slsa-verifier verify-artifact slsa-test-linux-amd64 \
  --provenance-path slsa-test-linux-amd64.intoto.jsonl \
  --source-uri github.com/myorg/myrepo \
  --source-tag v1.2.3

# Optionally pin the builder identity (SLSA L3)
slsa-verifier verify-artifact ./mybin \
  --provenance-path ./mybin.intoto.jsonl \
  --source-uri github.com/myorg/myrepo \
  --builder-id https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml@refs/tags/v2.0.0
```

### Step 5: Verify GitHub artifact attestations / blob bundles
For artifacts signed via `actions/attest-build-provenance`, the bundle uses the new Sigstore bundle format.

```bash
cosign verify-blob-attestation \
  --bundle ./myartifact.sigstore.json \
  --new-bundle-format \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  --certificate-identity-regexp="^https://github.com/myorg/myrepo/" \
  ./myartifact

# Equivalent native GitHub CLI verification
gh attestation verify ./myartifact --repo myorg/myrepo
```

### Step 6: Enforce verification as a gate
Wrap verification so the pipeline fails closed on any error.

```bash
#!/usr/bin/env bash
set -euo pipefail
IMG="ghcr.io/myorg/myrepo:v1.2.3"
cosign verify \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  --certificate-identity-regexp "^https://github.com/myorg/myrepo/" "$IMG" >/dev/null
cosign verify-attestation --type slsaprovenance \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  --certificate-identity-regexp "^https://github.com/myorg/myrepo/" "$IMG" >/dev/null
echo "[+] $IMG verified: signature + SLSA provenance OK"
```

### Step 7: Map findings to SLSA Build levels
Document the level each consumed artifact achieves:
- **Build L1** — provenance exists (the build process generates it), but it may be unsigned/forgeable.
- **Build L2** — provenance is signed by a hosted build service.
- **Build L3** — provenance is non-forgeable: generated on an isolated, hardened builder where secrets are unavailable to user-defined steps (e.g., `slsa-github-generator` reusable workflows). Require L3 for high-trust artifacts.

## Tools and Resources

| Tool / Resource | Purpose | Link |
|-----------------|---------|------|
| cosign | Sign/verify artifacts and attestations (keyless) | https://github.com/sigstore/cosign |
| slsa-verifier | Verify SLSA provenance from compliant builders | https://github.com/slsa-framework/slsa-verifier |
| slsa-github-generator | Produce SLSA L3 provenance in GitHub Actions | https://github.com/slsa-framework/slsa-github-generator |
| actions/attest-build-provenance | GitHub-native provenance attestation | https://github.com/actions/attest-build-provenance |
| SLSA specification | Build levels and provenance schema | https://slsa.dev/spec/v1.0/ |
| Sigstore docs | Fulcio, Rekor, cosign verification | https://docs.sigstore.dev/cosign/verifying/verify/ |

## Verification Identity Reference

| Field | Where it comes from | Why it matters |
|-------|--------------------|----------------|
| `--certificate-oidc-issuer` | The OIDC issuer (e.g., GitHub Actions) | Restricts who could have requested the signing cert |
| `--certificate-identity[-regexp]` | The exact/patterned workflow identity (SAN) | Restricts which workflow signed; prevents impersonation |
| `--source-uri` (slsa-verifier) | Expected source repo | Confirms the artifact came from your repo |
| `--source-tag` / `--source-versioned-tag` | Expected git tag | Prevents rollback/substitution |
| `--builder-id` | Trusted builder workflow ref | Enforces SLSA L3 non-forgeable builder |

## Validation Criteria

- [ ] cosign and slsa-verifier installed and report versions
- [ ] Image signature verified with pinned OIDC issuer AND certificate identity
- [ ] SLSA provenance attestation verified (`--type slsaprovenance`)
- [ ] Provenance predicate inspected; source repo/commit/builder match
- [ ] Release binary verified with slsa-verifier (source-uri + tag pinned)
- [ ] GitHub blob/bundle attestation verified
- [ ] Verification wired as a fail-closed CI/admission gate
- [ ] Each consumed artifact assigned a SLSA Build level

===== END FILE =====


===== FILE: skills/verifying-build-provenance-with-slsa-sigstore/LICENSE =====

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


===== FILE: skills/verifying-build-provenance-with-slsa-sigstore/references/api-reference.md =====
# cosign & slsa-verifier CLI Reference

Sources:
- https://github.com/sigstore/cosign
- https://github.com/slsa-framework/slsa-verifier
- https://docs.sigstore.dev/cosign/verifying/verify/

## cosign — verification commands

| Command | Purpose |
|---------|---------|
| `cosign verify <image>` | Verify image signature(s) |
| `cosign verify-attestation <image>` | Verify in-toto attestation attached to image |
| `cosign verify-blob <file>` | Verify a detached signature on a blob |
| `cosign verify-blob-attestation <file>` | Verify an attestation bundle for a blob |
| `cosign download attestation <image>` | Pull attestations for offline inspection |
| `cosign tree <image>` | Show signatures/attestations attached to an image |

## cosign — key verification flags

| Flag | Meaning |
|------|---------|
| `--certificate-oidc-issuer <url>` | Required keyless: OIDC issuer that minted the cert |
| `--certificate-identity <san>` | Exact certificate identity (workflow SAN) |
| `--certificate-identity-regexp <re>` | Regex form of identity |
| `--type <type>` | Predicate type: `slsaprovenance`, `slsaprovenance02`, `slsaprovenance1`, `spdx`, `cyclonedx`, `vuln`, custom |
| `--bundle <file>` | Sigstore bundle for blob attestation |
| `--new-bundle-format` | Use the new Sigstore bundle format |
| `--key <path>` | Verify with a fixed public key (non-keyless) |
| `--rekor-url <url>` | Transparency log (default https://rekor.sigstore.dev) |

### GitHub OIDC issuer (constant)
```
https://token.actions.githubusercontent.com
```

## slsa-verifier

| Command | Purpose |
|---------|---------|
| `slsa-verifier verify-artifact <artifact>` | Verify provenance for a binary/artifact |
| `slsa-verifier verify-image <image>` | Verify provenance for a container image |
| `slsa-verifier verify-npm-package <tarball>` | Verify npm package provenance |

### slsa-verifier flags

| Flag | Meaning |
|------|---------|
| `--provenance-path <file>` | Path to provenance (.intoto.jsonl / .sigstore) |
| `--source-uri <repo>` | Expected source repository (GitHub URIs) |
| `--source-tag <tag>` | Expected git tag |
| `--source-versioned-tag <tag>` | Semver-aware tag match |
| `--builder-id <ref>` | Pin the trusted builder workflow (SLSA L3) |
| `--print-provenance` | Print the verified provenance to stdout |

## GitHub CLI native verification

| Command | Purpose |
|---------|---------|
| `gh attestation verify <artifact> --repo <org>/<repo>` | Verify GitHub-generated build provenance |

===== END FILE =====


===== FILE: skills/verifying-build-provenance-with-slsa-sigstore/references/standards.md =====
# Standards Mapping — Verifying Build Provenance with SLSA and Sigstore

## MITRE ATT&CK

| ID | Technique Name | Rationale |
|----|----------------|-----------|
| T1195 | Supply Chain Compromise | Verifying signatures and SLSA provenance detects artifacts tampered with or substituted anywhere in the build and distribution chain, blocking supply-chain compromise before deployment. Ref: https://attack.mitre.org/techniques/T1195/ |

## NIST Cybersecurity Framework 2.0

| ID | Subcategory | Rationale |
|----|-------------|-----------|
| PR.DS-06 | Integrity-checking mechanisms are used to verify software, firmware, and information integrity | cosign signature verification and SLSA provenance verification are integrity-checking mechanisms that cryptographically confirm an artifact was built from the expected source by the expected builder and was not modified. |

## SLSA Build Levels

| Level | Guarantee |
|-------|-----------|
| Build L1 | Provenance exists (may be forgeable) |
| Build L2 | Provenance signed by a hosted build service |
| Build L3 | Non-forgeable provenance from a hardened, isolated builder |

Ref: https://slsa.dev/spec/v1.0/levels

## Supporting References

- Sigstore architecture (Fulcio short-lived certs, Rekor transparency log): https://docs.sigstore.dev/
- slsa-github-generator (L3 producer): https://github.com/slsa-framework/slsa-github-generator

===== END FILE =====


===== FILE: skills/verifying-build-provenance-with-slsa-sigstore/scripts/agent.py =====
#!/usr/bin/env python3
"""
provenance-verify — verify Sigstore signatures and SLSA provenance, fail closed.

Wraps the real cosign and slsa-verifier CLIs:
  - cosign:        https://github.com/sigstore/cosign
  - slsa-verifier: https://github.com/slsa-framework/slsa-verifier

Install the binaries first:
    go install github.com/sigstore/cosign/v2/cmd/cosign@latest
    go install github.com/slsa-framework/slsa-verifier/v2/cli/slsa-verifier@latest

Examples:
    python agent.py image \
        --image ghcr.io/myorg/myrepo:v1.2.3 \
        --oidc-issuer https://token.actions.githubusercontent.com \
        --identity-regexp '^https://github.com/myorg/myrepo/'

    python agent.py artifact \
        --artifact ./mybin \
        --provenance ./mybin.intoto.jsonl \
        --source-uri github.com/myorg/myrepo \
        --source-tag v1.2.3
"""
import argparse
import base64
import json
import shutil
import subprocess
import sys

GITHUB_OIDC = "https://token.actions.githubusercontent.com"


def _require(tool: str) -> None:
    if shutil.which(tool) is None:
        sys.exit(f"error: '{tool}' not found on PATH. See the skill prerequisites.")


def _run(argv: list, capture: bool = False) -> subprocess.CompletedProcess:
    print(f"[*] {' '.join(argv)}", file=sys.stderr)
    return subprocess.run(argv, capture_output=capture, text=True)


def verify_image(args) -> int:
    _require("cosign")
    issuer = args.oidc_issuer
    # 1) signature
    sig = ["cosign", "verify",
           "--certificate-oidc-issuer", issuer]
    if args.identity:
        sig += ["--certificate-identity", args.identity]
    else:
        sig += ["--certificate-identity-regexp", args.identity_regexp]
    sig.append(args.image)
    if _run(sig).returncode != 0:
        print("[!] signature verification FAILED", file=sys.stderr)
        return 1
    print("[+] signature OK", file=sys.stderr)

    # 2) SLSA provenance attestation
    att = ["cosign", "verify-attestation", "--type", args.predicate_type,
           "--certificate-oidc-issuer", issuer]
    if args.identity:
        att += ["--certificate-identity", args.identity]
    else:
        att += ["--certificate-identity-regexp", args.identity_regexp]
    att.append(args.image)
    proc = _run(att, capture=True)
    if proc.returncode != 0:
        print("[!] provenance verification FAILED", file=sys.stderr)
        print(proc.stderr, file=sys.stderr)
        return 1
    print("[+] SLSA provenance OK", file=sys.stderr)

    # 3) decode and surface key provenance fields
    try:
        payload = json.loads(proc.stdout.splitlines()[0])["payload"]
        decoded = json.loads(base64.b64decode(payload))
        pred = decoded.get("predicate", {})
        builder = (pred.get("runDetails", {}).get("builder", {}).get("id")
                   or pred.get("builder", {}).get("id"))
        print(json.dumps({"image": args.image, "verified": True,
                          "builder_id": builder}, indent=2))
    except Exception:
        print(json.dumps({"image": args.image, "verified": True}, indent=2))
    return 0


def verify_artifact(args) -> int:
    _require("slsa-verifier")
    argv = ["slsa-verifier", "verify-artifact", args.artifact,
            "--provenance-path", args.provenance,
            "--source-uri", args.source_uri]
    if args.source_tag:
        argv += ["--source-tag", args.source_tag]
    if args.builder_id:
        argv += ["--builder-id", args.builder_id]
    rc = _run(argv).returncode
    if rc != 0:
        print("[!] artifact provenance verification FAILED", file=sys.stderr)
        return 1
    print(json.dumps({"artifact": args.artifact, "verified": True,
                      "source_uri": args.source_uri,
                      "source_tag": args.source_tag}, indent=2))
    return 0


def main() -> int:
    p = argparse.ArgumentParser(description="Verify Sigstore signatures and SLSA provenance.")
    sub = p.add_subparsers(dest="cmd", required=True)

    im = sub.add_parser("image", help="verify container image signature + provenance")
    im.add_argument("--image", required=True)
    im.add_argument("--oidc-issuer", default=GITHUB_OIDC)
    g = im.add_mutually_exclusive_group(required=True)
    g.add_argument("--identity")
    g.add_argument("--identity-regexp")
    im.add_argument("--predicate-type", default="slsaprovenance")
    im.set_defaults(func=verify_image)

    ar = sub.add_parser("artifact", help="verify binary/artifact provenance via slsa-verifier")
    ar.add_argument("--artifact", required=True)
    ar.add_argument("--provenance", required=True)
    ar.add_argument("--source-uri", required=True)
    ar.add_argument("--source-tag")
    ar.add_argument("--builder-id")
    ar.set_defaults(func=verify_artifact)

    args = p.parse_args()
    return args.func(args)


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====
