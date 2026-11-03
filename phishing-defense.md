

===== FILE: skills/analyzing-malicious-url-with-urlscan/SKILL.md =====
---
name: analyzing-malicious-url-with-urlscan
description: URLScan.io is a free service for scanning and analyzing suspicious URLs.
  It captures screenshots, DOM content, HTTP transactions, JavaScript behavior, and
  network connections of web pages in an isolat
domain: cybersecurity
subdomain: phishing-defense
tags:
- phishing
- email-security
- social-engineering
- dmarc
- awareness
- url-analysis
- threat-intelligence
version: '1.0'
author: mahipal
license: Apache-2.0
atlas_techniques:
- AML.T0052
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566.002
- T1204.001
- T1598.003
---
# Analyzing Malicious URL with URLScan

## Overview
URLScan.io is a free service for scanning and analyzing suspicious URLs. It captures screenshots, DOM content, HTTP transactions, JavaScript behavior, and network connections of web pages in an isolated environment. This skill covers using URLScan's web interface and API to investigate phishing URLs, credential harvesting pages, and malicious redirects without exposing the analyst's system to risk.


## When to Use

- When investigating security incidents that require analyzing malicious url with urlscan
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites
- URLScan.io account (free tier available, API key for automation)
- Python 3.8+ with requests library
- Understanding of HTTP protocols and web technologies
- Familiarity with phishing URL patterns

## Key Concepts

### URLScan Capabilities
1. **Safe browsing**: Renders URLs in isolated Chromium instance
2. **Screenshot capture**: Visual snapshot of the rendered page
3. **DOM analysis**: Full HTML content after JavaScript execution
4. **Network log**: All HTTP requests made by the page (HAR format)
5. **Certificate analysis**: SSL/TLS certificate details
6. **Technology detection**: Identifies web frameworks and libraries
7. **IP/ASN mapping**: Infrastructure intelligence
8. **Verdict**: Community and automated classification

### Phishing URL Red Flags
- Newly registered domains (< 30 days)
- Free hosting services (Wix, GitHub Pages, Firebase)
- URL shorteners hiding final destination
- Excessive subdomain depth (login.microsoft.com.evil.com)
- Brand name in subdomain or path, not domain
- Non-standard ports
- Data URIs or base64-encoded content
- JavaScript-heavy pages with minimal HTML

## Workflow

### Step 1: Submit URL to URLScan
```
Web: Navigate to https://urlscan.io and submit the suspicious URL
API: POST https://urlscan.io/api/v1/scan/
     Header: API-Key: your-api-key
     Body: {"url": "https://suspicious-url.com", "visibility": "private"}
```

### Step 2: Analyze Results
- Review screenshot for brand impersonation
- Check redirects and final destination URL
- Examine DOM for credential input forms
- Review network requests for data exfiltration endpoints
- Check SSL certificate validity and issuer

### Step 3: Extract IOCs
- Domains and IPs contacted
- URLs in redirect chain
- SHA-256 hashes of page resources
- JavaScript file hashes

### Step 4: Cross-Reference with Threat Intelligence
Use the `scripts/process.py` to automate URL scanning, extract IOCs, and cross-reference with VirusTotal, PhishTank, and Google Safe Browsing.

## Tools & Resources
- **URLScan.io**: https://urlscan.io/
- **URLScan API**: https://urlscan.io/docs/api/
- **VirusTotal URL Scanner**: https://www.virustotal.com/
- **PhishTank**: https://phishtank.org/
- **Google Safe Browsing**: https://transparencyreport.google.com/safe-browsing/search
- **Any.Run**: https://any.run/ (interactive sandbox)
- **Hybrid Analysis**: https://www.hybrid-analysis.com/

## Validation
- Successfully scan a suspicious URL via API
- Extract screenshot and identify brand impersonation
- Document complete redirect chain
- Generate IOC list from scan results
- Cross-reference findings with at least 2 threat intelligence sources

===== END FILE =====


===== FILE: skills/analyzing-malicious-url-with-urlscan/LICENSE =====

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


===== FILE: skills/analyzing-malicious-url-with-urlscan/assets/template.md =====
# URL Analysis Report Template

## Analysis Information
- **Analyst**: [Name]
- **Date**: [YYYY-MM-DD]
- **Case ID**: [CASE-XXXX]
- **Source**: [User report / Email gateway / SIEM alert]

## URL Details
| Field | Value |
|---|---|
| Original URL | |
| Defanged URL | hxxps://... |
| Final URL (after redirects) | |
| URLScan UUID | |
| Scan visibility | private/public |

## Page Analysis
| Field | Value |
|---|---|
| Page Title | |
| HTTP Status | |
| Server | |
| Domain | |
| IP Address | |
| ASN | |
| Country | |
| Login Form Detected | Yes/No |

## TLS Certificate
| Field | Value |
|---|---|
| Issuer | |
| Subject | |
| Valid From | |
| Valid To | |
| Certificate Age | |

## Redirect Chain
| # | URL | Status |
|---|---|---|
| 1 (original) | | |
| 2 | | |
| 3 (final) | | |

## Threat Intelligence Cross-Reference
| Source | Result | Score |
|---|---|---|
| URLScan Verdict | | |
| VirusTotal | /XX engines | |
| PhishTank | | |
| Google Safe Browsing | | |
| AbuseIPDB | | |

## IOCs Extracted
### Domains
| Domain | Role | Reputation |
|---|---|---|
| | | |

### IP Addresses
| IP | ASN | Country | Reputation |
|---|---|---|---|
| | | | |

### File Hashes
| Hash (SHA-256) | Type | Size |
|---|---|---|
| | | |

## Classification
- [ ] Phishing - Credential Harvesting
- [ ] Phishing - Malware Delivery
- [ ] Scam / Fraud
- [ ] Benign
- [ ] Inconclusive

## Recommended Actions
- [ ] Block domain at proxy/firewall
- [ ] Block IP at firewall
- [ ] Add to email gateway blocklist
- [ ] Submit to PhishTank / APWG
- [ ] Notify affected users
- [ ] Request domain takedown

## Notes
[Additional analysis observations]

===== END FILE =====


===== FILE: skills/analyzing-malicious-url-with-urlscan/references/api-reference.md =====
# API Reference: urlscan.io URL Analysis

## Base URL
```
https://urlscan.io/api/v1
```

## Authentication
```
API-Key: YOUR_API_KEY
```

## Submit Scan
```
POST /scan/
```
```json
{"url": "https://example.com", "visibility": "private"}
```
| Field | Values | Description |
|-------|--------|-------------|
| `url` | URL string | URL to scan |
| `visibility` | public/unlisted/private | Scan visibility |

Response: `{"uuid": "...", "result": "https://urlscan.io/result/UUID/", "api": "..."}`

## Get Result
```
GET /result/{uuid}/
```
Returns 404 while scanning, 200 when complete.

## Search
```
GET /search/?q=domain:example.com&size=100
```
Query fields: `domain:`, `ip:`, `server:`, `country:`, `filename:`, `hash:`

## Result Structure
| Field | Description |
|-------|-------------|
| `page.url` | Final URL after redirects |
| `page.domain` | Domain name |
| `page.ip` | Resolved IP |
| `page.country` | Server country |
| `page.status` | HTTP status code |
| `page.title` | Page title |
| `page.server` | Server header |
| `page.tlsIssuer` | TLS certificate issuer |
| `verdicts.overall.malicious` | Boolean malicious verdict |
| `verdicts.overall.score` | Risk score (0-100) |
| `lists.ips` | List of contacted IPs |
| `lists.certificates` | TLS certificates observed |
| `stats.resourceStats` | Resource type statistics |

## Screenshot
```
GET /screenshots/{uuid}.png
```

## DOM Snapshot
```
GET /dom/{uuid}/
```

## Rate Limits
- Free: 100 scans/day, 1000 searches/day
- Paid: Higher limits per plan

===== END FILE =====


===== FILE: skills/analyzing-malicious-url-with-urlscan/references/standards.md =====
# Standards & References: Analyzing Malicious URLs with URLScan

## MITRE ATT&CK References
- **T1566.002**: Phishing: Spearphishing Link
- **T1204.001**: User Execution: Malicious Link
- **T1608.005**: Stage Capabilities: Link Target
- **T1071.001**: Application Layer Protocol: Web Protocols
- **T1102**: Web Service (for C2 via web)

## URLScan.io API Reference
| Endpoint | Method | Description |
|---|---|---|
| `/api/v1/scan/` | POST | Submit URL for scanning |
| `/api/v1/result/{uuid}/` | GET | Get scan results |
| `/api/v1/search/?q=` | GET | Search scan database |
| `/api/v1/result/{uuid}/screenshot/` | GET | Get page screenshot |
| `/api/v1/result/{uuid}/dom/` | GET | Get rendered DOM |

### Search Query Syntax
- `domain:example.com` - Search by domain
- `ip:1.2.3.4` - Search by IP
- `server:nginx` - Search by web server
- `filename:login.php` - Search by filename
- `hash:abc123` - Search by resource hash
- `page.domain:example.com AND date:>now-7d` - Combined queries

## Industry Standards
- **NIST SP 800-83**: Guide to Malware Incident Prevention and Handling
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response
- **RFC 3986**: Uniform Resource Identifier (URI) syntax

## URL Classification Indicators
| Indicator | Risk Level | Description |
|---|---|---|
| Domain age < 7 days | Critical | Very recently registered |
| Domain age < 30 days | High | Recently registered |
| Free TLS cert (Let's Encrypt) with brand impersonation | High | Common phishing pattern |
| URL shortener | Medium | Obfuscates destination |
| Credential input form on non-brand domain | Critical | Credential harvesting |
| JavaScript obfuscation | High | Evasion technique |
| Multiple redirects | Medium | Chain obfuscation |
| Data URI scheme | High | Inline content, hard to trace |

===== END FILE =====


===== FILE: skills/analyzing-malicious-url-with-urlscan/references/workflows.md =====
# Workflows: Analyzing Malicious URLs with URLScan

## Workflow 1: URL Triage Pipeline

```
Suspicious URL received (from user report / email gateway / SIEM)
  |
  v
[Step 1: Defang and document URL]
  +-- Replace http with hxxp, . with [.]
  +-- Record original context (email subject, sender, timestamp)
  |
  v
[Step 2: Submit to URLScan (private visibility)]
  +-- POST to /api/v1/scan/
  +-- Wait for scan completion (poll /api/v1/result/{uuid}/)
  |
  v
[Step 3: Analyze results]
  +-- Review screenshot for brand impersonation
  +-- Check redirect chain (original URL vs final URL)
  +-- Examine DOM for login forms / credential inputs
  +-- Review network requests for suspicious endpoints
  +-- Check SSL certificate details
  |
  v
[Step 4: Classify]
  +-- Phishing (credential harvesting)
  +-- Malware delivery
  +-- Scam / fraud
  +-- Benign (false positive)
  |
  v
[Step 5: Action]
  +-- If malicious: Extract IOCs, block domain/IP, update filters
  +-- If benign: Document and close
  +-- If uncertain: Escalate for deeper analysis
```

## Workflow 2: Bulk URL Analysis

```
URL list from email gateway / threat feed
  |
  v
[Batch submit to URLScan API]
  +-- Rate limit: 2 submissions/second (free tier)
  +-- Use private visibility for sensitive URLs
  |
  v
[Collect all results]
  +-- Poll each scan UUID for completion
  +-- Download screenshots and DOM content
  |
  v
[Automated triage]
  +-- Flag: credential input forms detected
  +-- Flag: brand impersonation in screenshot
  +-- Flag: known phishing infrastructure (IP/ASN)
  +-- Flag: newly registered domains
  |
  v
[Generate report]
  +-- Categorized URL list (malicious / suspicious / clean)
  +-- IOC extract for blocking
  +-- Statistics summary
```

## Workflow 3: IOC Extraction and Enrichment

```
URLScan result available
  |
  v
[Extract from scan]
  +-- All domains contacted
  +-- All IPs contacted
  +-- SSL certificate fingerprints
  +-- JavaScript file hashes
  +-- Page resource hashes
  +-- Final redirect URL
  |
  v
[Cross-reference]
  +-- VirusTotal: domain/IP/hash reputation
  +-- PhishTank: known phishing URL database
  +-- WHOIS: domain registration details
  +-- AbuseIPDB: IP abuse reports
  +-- Google Safe Browsing: malware/phishing flags
  |
  v
[Compile IOC package]
  +-- STIX/TAXII format for TIP
  +-- CSV for firewall/proxy rules
  +-- JSON for SIEM enrichment
```

===== END FILE =====


===== FILE: skills/analyzing-malicious-url-with-urlscan/scripts/agent.py =====
#!/usr/bin/env python3
"""URLScan.io Malicious URL Analysis Agent - Submits and analyzes URLs via the urlscan.io API."""

import json
import time
import logging
import argparse
from datetime import datetime

import requests

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

URLSCAN_API = "https://urlscan.io/api/v1"


def submit_url(url, api_key, visibility="private"):
    """Submit a URL to urlscan.io for scanning."""
    headers = {"API-Key": api_key, "Content-Type": "application/json"}
    payload = {"url": url, "visibility": visibility}
    resp = requests.post(f"{URLSCAN_API}/scan/", headers=headers, json=payload, timeout=30)
    resp.raise_for_status()
    data = resp.json()
    logger.info("Submitted URL: %s -> scan UUID: %s", url, data.get("uuid"))
    return data


def get_scan_result(uuid, api_key, max_wait=120):
    """Poll for scan results until complete."""
    headers = {"API-Key": api_key}
    for _ in range(max_wait // 5):
        try:
            resp = requests.get(f"{URLSCAN_API}/result/{uuid}/", headers=headers, timeout=30)
            if resp.status_code == 200:
                return resp.json()
        except requests.RequestException:
            pass
        time.sleep(5)
    return None


def search_urlscan(query, api_key, size=100):
    """Search urlscan.io for existing scans."""
    headers = {"API-Key": api_key}
    params = {"q": query, "size": size}
    resp = requests.get(f"{URLSCAN_API}/search/", headers=headers, params=params, timeout=30)
    resp.raise_for_status()
    return resp.json().get("results", [])


def analyze_result(result):
    """Analyze urlscan.io scan result for malicious indicators."""
    findings = []
    verdicts = result.get("verdicts", {})
    overall = verdicts.get("overall", {})
    urlscan_verdict = verdicts.get("urlscan", {})
    community = verdicts.get("community", {})

    if overall.get("malicious"):
        findings.append({"type": "Malicious verdict", "severity": "critical", "source": "overall", "score": overall.get("score", 0)})
    if urlscan_verdict.get("malicious"):
        findings.append({"type": "URLScan malicious", "severity": "critical", "score": urlscan_verdict.get("score", 0)})
    if community.get("score", 0) < 0:
        findings.append({"type": "Negative community score", "severity": "high", "score": community.get("score")})

    page = result.get("page", {})
    lists = result.get("lists", {})
    stats = result.get("stats", {})

    if lists.get("ips", []):
        for ip in lists["ips"]:
            if ip.get("malicious"):
                findings.append({"type": "Malicious IP contacted", "severity": "high", "ip": ip.get("ip"), "asn": ip.get("asn")})

    for cert in lists.get("certificates", []):
        if cert.get("validTo"):
            try:
                exp = datetime.fromisoformat(cert["validTo"].replace("Z", "+00:00"))
                if exp < datetime.now(exp.tzinfo):
                    findings.append({"type": "Expired TLS certificate", "severity": "medium", "subject": cert.get("subjectName")})
            except (ValueError, TypeError):
                pass

    js_count = len([r for r in stats.get("resourceStats", []) if "javascript" in r.get("type", "").lower()])
    if js_count > 20:
        findings.append({"type": "High JavaScript resource count", "severity": "medium", "count": js_count})

    redirects = stats.get("uniqCountries", 0)
    if result.get("data", {}).get("requests"):
        redirect_chain = [r.get("request", {}).get("redirectHasExtraInfo") for r in result["data"]["requests"][:5]]

    return {
        "url": page.get("url", ""),
        "domain": page.get("domain", ""),
        "ip": page.get("ip", ""),
        "country": page.get("country", ""),
        "server": page.get("server", ""),
        "status_code": page.get("status", 0),
        "title": page.get("title", ""),
        "mime_type": page.get("mimeType", ""),
        "tls_issuer": page.get("tlsIssuer", ""),
        "overall_malicious": overall.get("malicious", False),
        "overall_score": overall.get("score", 0),
        "findings": findings,
    }


def bulk_analyze(urls, api_key):
    """Submit and analyze multiple URLs."""
    results = []
    for url in urls:
        try:
            submission = submit_url(url, api_key)
            uuid = submission.get("uuid")
            if uuid:
                result = get_scan_result(uuid, api_key)
                if result:
                    analysis = analyze_result(result)
                    results.append(analysis)
                else:
                    results.append({"url": url, "error": "Scan timeout"})
        except requests.RequestException as e:
            results.append({"url": url, "error": str(e)})
    return results


def generate_report(analyses):
    """Generate URL analysis report."""
    malicious = [a for a in analyses if a.get("overall_malicious")]
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "urls_analyzed": len(analyses),
        "malicious_count": len(malicious),
        "results": analyses,
    }
    print(f"URLSCAN REPORT: {len(analyses)} URLs analyzed, {len(malicious)} malicious")
    return report


def main():
    parser = argparse.ArgumentParser(description="URLScan.io Malicious URL Analysis Agent")
    parser.add_argument("--api-key", required=True, help="urlscan.io API key")
    parser.add_argument("--url", help="Single URL to scan")
    parser.add_argument("--url-file", help="File with URLs (one per line)")
    parser.add_argument("--search", help="Search query for existing scans")
    parser.add_argument("--output", default="urlscan_report.json")
    args = parser.parse_args()

    urls = []
    if args.url:
        urls.append(args.url)
    if args.url_file:
        with open(args.url_file) as f:
            urls.extend(line.strip() for line in f if line.strip())

    if args.search:
        results = search_urlscan(args.search, args.api_key)
        analyses = [analyze_result(r) for r in results if "page" in r]
    else:
        analyses = bulk_analyze(urls, args.api_key)

    report = generate_report(analyses)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/analyzing-malicious-url-with-urlscan/scripts/process.py =====
#!/usr/bin/env python3
"""
URLScan.io URL Analysis Automation

Submits suspicious URLs to URLScan.io for analysis, retrieves results,
extracts IOCs, and cross-references with threat intelligence sources.

Usage:
    python process.py scan --url "https://suspicious-site.com"
    python process.py scan --url-file urls.txt
    python process.py result --uuid <scan-uuid>
    python process.py search --query "domain:evil.com"
    python process.py ioc --uuid <scan-uuid>
"""

import argparse
import json
import sys
import time
import os
import hashlib
from datetime import datetime, timezone
from pathlib import Path
from dataclasses import dataclass, field, asdict

try:
    import requests
    HAS_REQUESTS = True
except ImportError:
    HAS_REQUESTS = False

URLSCAN_API_KEY = os.environ.get("URLSCAN_API_KEY", "")
URLSCAN_BASE = "https://urlscan.io/api/v1"
VT_API_KEY = os.environ.get("VT_API_KEY", "")


@dataclass
class URLScanResult:
    """Parsed URLScan result."""
    uuid: str = ""
    url: str = ""
    effective_url: str = ""
    status_code: int = 0
    domain: str = ""
    ip: str = ""
    asn: str = ""
    asn_name: str = ""
    country: str = ""
    server: str = ""
    title: str = ""
    tls_issuer: str = ""
    tls_subject: str = ""
    tls_valid_from: str = ""
    tls_valid_to: str = ""
    screenshot_url: str = ""
    dom_url: str = ""
    technologies: list = field(default_factory=list)
    redirects: list = field(default_factory=list)
    domains_contacted: list = field(default_factory=list)
    ips_contacted: list = field(default_factory=list)
    urls_contacted: list = field(default_factory=list)
    has_login_form: bool = False
    resource_hashes: list = field(default_factory=list)
    verdicts: dict = field(default_factory=dict)
    is_malicious: bool = False
    risk_indicators: list = field(default_factory=list)


def submit_scan(url: str, visibility: str = "private",
                api_key: str = "") -> dict:
    """Submit a URL to URLScan for scanning."""
    if not api_key:
        api_key = URLSCAN_API_KEY
    if not api_key:
        print("Warning: No URLScan API key provided. Using public submission.", file=sys.stderr)

    headers = {"Content-Type": "application/json"}
    if api_key:
        headers["API-Key"] = api_key

    data = {"url": url, "visibility": visibility}

    try:
        resp = requests.post(f"{URLSCAN_BASE}/scan/", headers=headers,
                             json=data, timeout=30)
        if resp.status_code == 200:
            return resp.json()
        elif resp.status_code == 429:
            print("Rate limited. Waiting 10 seconds...", file=sys.stderr)
            time.sleep(10)
            resp = requests.post(f"{URLSCAN_BASE}/scan/", headers=headers,
                                 json=data, timeout=30)
            return resp.json() if resp.status_code == 200 else {"error": resp.text}
        else:
            return {"error": f"HTTP {resp.status_code}: {resp.text}"}
    except Exception as e:
        return {"error": str(e)}


def get_result(uuid: str, api_key: str = "", max_wait: int = 60) -> dict:
    """Get scan results, polling until ready."""
    if not api_key:
        api_key = URLSCAN_API_KEY

    headers = {}
    if api_key:
        headers["API-Key"] = api_key

    for attempt in range(max_wait // 5):
        try:
            resp = requests.get(f"{URLSCAN_BASE}/result/{uuid}/",
                                headers=headers, timeout=30)
            if resp.status_code == 200:
                return resp.json()
            elif resp.status_code == 404:
                time.sleep(5)
                continue
            else:
                return {"error": f"HTTP {resp.status_code}: {resp.text}"}
        except Exception as e:
            return {"error": str(e)}

    return {"error": "Timeout waiting for scan results"}


def search_scans(query: str, api_key: str = "", size: int = 10) -> list:
    """Search URLScan database."""
    if not api_key:
        api_key = URLSCAN_API_KEY

    headers = {}
    if api_key:
        headers["API-Key"] = api_key

    try:
        resp = requests.get(f"{URLSCAN_BASE}/search/?q={query}&size={size}",
                            headers=headers, timeout=30)
        if resp.status_code == 200:
            return resp.json().get("results", [])
    except Exception:
        pass
    return []


def parse_result(raw_result: dict) -> URLScanResult:
    """Parse raw URLScan API result into structured data."""
    result = URLScanResult()

    task = raw_result.get("task", {})
    result.uuid = task.get("uuid", "")
    result.url = task.get("url", "")

    page = raw_result.get("page", {})
    result.effective_url = page.get("url", "")
    result.status_code = page.get("status", 0)
    result.domain = page.get("domain", "")
    result.ip = page.get("ip", "")
    result.asn = page.get("asn", "")
    result.asn_name = page.get("asnname", "")
    result.country = page.get("country", "")
    result.server = page.get("server", "")
    result.title = page.get("title", "")

    # TLS info
    tls_list = raw_result.get("lists", {}).get("certificates", [])
    if tls_list:
        cert = tls_list[0]
        result.tls_issuer = cert.get("issuer", "")
        result.tls_subject = cert.get("subjectName", "")
        result.tls_valid_from = cert.get("validFrom", "")
        result.tls_valid_to = cert.get("validTo", "")

    # Screenshot and DOM URLs
    result.screenshot_url = f"https://urlscan.io/screenshots/{result.uuid}.png"
    result.dom_url = f"https://urlscan.io/dom/{result.uuid}/"

    # Technologies
    meta = raw_result.get("meta", {})
    for processor in meta.get("processors", {}).values():
        if isinstance(processor, dict) and "data" in processor:
            techs = processor["data"]
            if isinstance(techs, list):
                for tech in techs:
                    if isinstance(tech, dict) and "app" in tech:
                        result.technologies.append(tech["app"])

    # Redirects
    data = raw_result.get("data", {})
    for request in data.get("requests", [])[:5]:
        req_url = request.get("request", {}).get("request", {}).get("url", "")
        resp_url = request.get("response", {}).get("response", {}).get("url", "")
        if req_url != result.url:
            result.redirects.append(req_url)

    # Domains and IPs contacted
    lists = raw_result.get("lists", {})
    result.domains_contacted = lists.get("domains", [])
    result.ips_contacted = lists.get("ips", [])
    result.urls_contacted = lists.get("urls", [])[:50]

    # Resource hashes
    for request in data.get("requests", []):
        resp_data = request.get("response", {}).get("response", {})
        resp_hash = resp_data.get("hash", "")
        if resp_hash:
            result.resource_hashes.append({
                "url": resp_data.get("url", ""),
                "hash": resp_hash,
                "size": resp_data.get("size", 0),
                "mimeType": resp_data.get("mimeType", "")
            })

    # Check for login forms in DOM
    dom_content = raw_result.get("data", {}).get("dom", "")
    if isinstance(dom_content, str):
        if ('type="password"' in dom_content.lower() or
                'input type=password' in dom_content.lower() or
                '<form' in dom_content.lower()):
            result.has_login_form = True

    # Verdicts
    verdicts = raw_result.get("verdicts", {})
    result.verdicts = {
        "overall_score": verdicts.get("overall", {}).get("score", 0),
        "overall_malicious": verdicts.get("overall", {}).get("malicious", False),
        "urlscan_score": verdicts.get("urlscan", {}).get("score", 0),
        "engines": verdicts.get("engines", {}).get("malicious", []),
        "community_score": verdicts.get("community", {}).get("score", 0),
    }
    result.is_malicious = verdicts.get("overall", {}).get("malicious", False)

    # Risk indicators
    if result.has_login_form and result.domain != result.url.split("/")[2]:
        result.risk_indicators.append("Credential harvesting form on non-origin domain")
    if result.url != result.effective_url:
        result.risk_indicators.append(f"URL redirected: {result.url} -> {result.effective_url}")
    if result.is_malicious:
        result.risk_indicators.append("Flagged as malicious by URLScan verdicts")
    if len(result.redirects) > 3:
        result.risk_indicators.append(f"Excessive redirects ({len(result.redirects)})")

    return result


def extract_iocs(result: URLScanResult) -> dict:
    """Extract IOCs from scan result."""
    iocs = {
        "domains": list(set(result.domains_contacted)),
        "ips": list(set(result.ips_contacted)),
        "urls": [result.url, result.effective_url] + result.redirects,
        "hashes": [h["hash"] for h in result.resource_hashes if h.get("hash")],
        "tls_fingerprint": result.tls_subject,
        "scan_uuid": result.uuid,
        "scan_date": datetime.now(timezone.utc).isoformat(),
    }
    # Deduplicate URLs
    iocs["urls"] = list(set(u for u in iocs["urls"] if u))
    return iocs


def check_virustotal(url: str, api_key: str = "") -> dict:
    """Check URL against VirusTotal (requires API key)."""
    if not api_key:
        api_key = VT_API_KEY
    if not api_key:
        return {}

    url_id = hashlib.sha256(url.encode()).hexdigest()
    headers = {"x-apikey": api_key}

    try:
        resp = requests.get(f"https://www.virustotal.com/api/v3/urls/{url_id}",
                            headers=headers, timeout=15)
        if resp.status_code == 200:
            data = resp.json().get("data", {}).get("attributes", {})
            stats = data.get("last_analysis_stats", {})
            return {
                "malicious": stats.get("malicious", 0),
                "suspicious": stats.get("suspicious", 0),
                "harmless": stats.get("harmless", 0),
                "undetected": stats.get("undetected", 0),
            }
    except Exception:
        pass
    return {}


def format_report(result: URLScanResult) -> str:
    """Format scan result as text report."""
    lines = []
    lines.append("=" * 60)
    lines.append("  URL ANALYSIS REPORT (URLScan.io)")
    lines.append("=" * 60)
    lines.append(f"  Scan UUID: {result.uuid}")
    lines.append(f"  Submitted URL: {result.url}")
    lines.append(f"  Effective URL: {result.effective_url}")
    lines.append(f"  Status Code: {result.status_code}")
    lines.append(f"  Malicious: {'YES' if result.is_malicious else 'NO'}")
    lines.append("")

    lines.append("[PAGE INFO]")
    lines.append(f"  Title: {result.title}")
    lines.append(f"  Domain: {result.domain}")
    lines.append(f"  IP: {result.ip}")
    lines.append(f"  ASN: {result.asn} ({result.asn_name})")
    lines.append(f"  Country: {result.country}")
    lines.append(f"  Server: {result.server}")
    lines.append(f"  Login Form: {'DETECTED' if result.has_login_form else 'Not found'}")
    lines.append(f"  Screenshot: {result.screenshot_url}")
    lines.append("")

    if result.tls_issuer:
        lines.append("[TLS CERTIFICATE]")
        lines.append(f"  Issuer: {result.tls_issuer}")
        lines.append(f"  Subject: {result.tls_subject}")
        lines.append("")

    if result.redirects:
        lines.append(f"[REDIRECTS] ({len(result.redirects)} found)")
        for r in result.redirects[:10]:
            lines.append(f"  -> {r}")
        lines.append("")

    if result.risk_indicators:
        lines.append(f"[RISK INDICATORS] ({len(result.risk_indicators)})")
        for ind in result.risk_indicators:
            lines.append(f"  - {ind}")
        lines.append("")

    lines.append(f"[INFRASTRUCTURE]")
    lines.append(f"  Domains contacted: {len(result.domains_contacted)}")
    lines.append(f"  IPs contacted: {len(result.ips_contacted)}")
    lines.append(f"  Resource hashes: {len(result.resource_hashes)}")

    lines.append("=" * 60)
    return "\n".join(lines)


def main():
    parser = argparse.ArgumentParser(description="URLScan.io URL Analysis Tool")
    subparsers = parser.add_subparsers(dest="command")

    scan_parser = subparsers.add_parser("scan", help="Scan a URL")
    scan_parser.add_argument("--url", help="Single URL to scan")
    scan_parser.add_argument("--url-file", help="File with URLs (one per line)")
    scan_parser.add_argument("--visibility", default="private",
                             choices=["public", "unlisted", "private"])
    scan_parser.add_argument("--wait", action="store_true", help="Wait for results")

    result_parser = subparsers.add_parser("result", help="Get scan result")
    result_parser.add_argument("--uuid", required=True)

    search_parser = subparsers.add_parser("search", help="Search URLScan database")
    search_parser.add_argument("--query", "-q", required=True)
    search_parser.add_argument("--size", type=int, default=10)

    ioc_parser = subparsers.add_parser("ioc", help="Extract IOCs from scan")
    ioc_parser.add_argument("--uuid", required=True)

    parser.add_argument("--api-key", default=URLSCAN_API_KEY)
    parser.add_argument("--json", action="store_true")
    parser.add_argument("--output", "-o")

    args = parser.parse_args()

    if not HAS_REQUESTS:
        print("Error: 'requests' library required", file=sys.stderr)
        sys.exit(1)

    api_key = args.api_key

    if args.command == "scan":
        urls = []
        if args.url:
            urls.append(args.url)
        elif args.url_file:
            with open(args.url_file) as f:
                urls = [line.strip() for line in f if line.strip()]

        for url in urls:
            print(f"Scanning: {url}")
            scan_result = submit_scan(url, args.visibility, api_key)

            if "error" in scan_result:
                print(f"  Error: {scan_result['error']}", file=sys.stderr)
                continue

            uuid = scan_result.get("uuid", "")
            print(f"  UUID: {uuid}")
            print(f"  Result URL: https://urlscan.io/result/{uuid}/")

            if args.wait and uuid:
                print("  Waiting for results...")
                time.sleep(10)
                raw = get_result(uuid, api_key)
                if "error" not in raw:
                    result = parse_result(raw)
                    if args.json:
                        print(json.dumps(asdict(result), indent=2, default=str))
                    else:
                        print(format_report(result))

            if len(urls) > 1:
                time.sleep(2)  # Rate limiting

    elif args.command == "result":
        raw = get_result(args.uuid, api_key)
        if "error" in raw:
            print(f"Error: {raw['error']}", file=sys.stderr)
            sys.exit(1)
        result = parse_result(raw)
        if args.json:
            print(json.dumps(asdict(result), indent=2, default=str))
        else:
            print(format_report(result))

    elif args.command == "search":
        results = search_scans(args.query, api_key, args.size)
        for r in results:
            task = r.get("task", {})
            page = r.get("page", {})
            print(f"  {task.get('time', '')} | {task.get('url', '')} | "
                  f"{page.get('domain', '')} | {page.get('ip', '')}")

    elif args.command == "ioc":
        raw = get_result(args.uuid, api_key)
        if "error" in raw:
            print(f"Error: {raw['error']}", file=sys.stderr)
            sys.exit(1)
        result = parse_result(raw)
        iocs = extract_iocs(result)
        print(json.dumps(iocs, indent=2))

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/building-phishing-reporting-button-workflow/SKILL.md =====
---
name: building-phishing-reporting-button-workflow
description: Implement a phishing report button (Microsoft 365 built-in Report button
  or third-party like KnowBe4/Cofense) in email clients with a SOAR-driven automated
  triage workflow that classifies reported emails, extracts IOCs, takes remediation
  actions, and gives feedback to reporters. Use when deploying user-reported phishing
  intake or automating triage of the resulting reporting mailbox.
domain: cybersecurity
subdomain: phishing-defense
tags:
- phishing-reporting
- email-security
- incident-response
- security-awareness
- outlook
- microsoft-365
- soar
mitre_attack:
- T1566.001
- T1566.002
- T1598.003
- T1204.001
- T1534
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
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
---
# Building Phishing Reporting Button Workflow

## Overview
A phishing reporting button empowers users to flag suspicious emails directly from their email client, creating a critical feedback loop between end users and the security operations center. Microsoft's built-in Report button is now the recommended approach, replacing the deprecated Report Message and Report Phishing add-ins. When combined with automated triage using SOAR platforms, reported emails can be classified, IOCs extracted, and remediation actions taken within minutes. Organizations with effective phishing reporting programs see 70%+ report rates in phishing simulations.


## When to Use

- When deploying or configuring building phishing reporting button workflow capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites
- Microsoft 365 or Google Workspace with administrative access
- SOAR platform or automation capability (Microsoft Sentinel, Splunk SOAR, Cortex XSOAR)
- Dedicated reporting mailbox for phishing submissions
- Email security gateway with message retraction capability
- Security awareness training platform for feedback loop

## Workflow

### Step 1: Deploy Phishing Report Button
- Enable Microsoft built-in Report button via Security & Compliance Center
- Configure user reported settings: route to reporting mailbox and Microsoft
- For third-party: deploy KnowBe4 Phish Alert Button or Cofense Reporter
- Verify button appears in Outlook desktop, web, and mobile clients
- Configure report options: Report Phishing, Report Junk, Report Not Junk

### Step 2: Build Automated Triage Pipeline
- Configure reporting mailbox monitored by SOAR platform
- Auto-extract IOCs from reported emails: URLs, attachments, sender info, headers
- Submit URLs to VirusTotal, URLScan.io for reputation check
- Submit attachments to sandbox for dynamic analysis
- Check sender against known threat intelligence feeds
- Auto-classify: confirmed phishing, spam, simulation, legitimate

### Step 3: Implement Response Actions
- Confirmed phishing: auto-retract from all inboxes, block sender domain
- Confirmed spam: move to junk for all recipients
- Simulation email: mark as correctly reported, credit user
- Legitimate email: return to inbox, notify reporter
- Generate IOC report for threat intelligence team

### Step 4: Create Feedback Loop
- Send automated thank-you response to reporter within 5 minutes
- Include classification result when analysis completes
- Track reporter accuracy and engagement metrics
- Recognize top reporters in monthly security newsletter
- Feed reporting metrics into security awareness training program

### Step 5: Measure and Optimize
- Track mean time to triage (target: under 10 minutes automated)
- Monitor report volume trends and false positive rates
- Measure user reporting rate in phishing simulations
- Report on confirmed threats caught by user reports vs. gateway
- Optimize automation rules based on classification accuracy

## Tools & Resources
- **Microsoft Report Button**: Built-in Outlook phishing reporting
- **Cofense Reporter + Triage**: Enterprise phishing reporting and automated analysis
- **KnowBe4 Phish Alert Button**: Integrated reporting with simulation platform
- **Microsoft Sentinel**: SOAR automation for triage workflow
- **Proofpoint CLEAR**: Closed-loop email analysis and response

## Validation
- Report button visible and functional across all Outlook platforms
- Reported email arrives in dedicated mailbox within 60 seconds
- Automated triage classifies test phishing email correctly
- Auto-retraction removes confirmed phishing from all inboxes
- Reporter receives feedback notification with classification
- Metrics dashboard shows report volume and accuracy trends

===== END FILE =====


===== FILE: skills/building-phishing-reporting-button-workflow/LICENSE =====

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


===== FILE: skills/building-phishing-reporting-button-workflow/assets/template.md =====
# Phishing Reporting Button Workflow Template

## Reporting Button Configuration
| Setting | Value | Status |
|---|---|---|
| Button type | Microsoft built-in Report | |
| Reporting mailbox | phishing-reports@company.com | |
| Also send to Microsoft | Yes | |
| Supported platforms | Desktop, Web, Mobile | |

## Triage Automation Rules
| Classification | Criteria | Auto-Action |
|---|---|---|
| Confirmed Phishing | Score >= 50 | Retract + Block sender |
| Suspicious | Score 25-49 | Escalate to SOC analyst |
| Spam | Score 10-24 | Move to junk for all |
| Simulation | Matches sim subject | Credit reporter |
| Clean | Score < 10 | Return to inbox |

## Reporting Metrics Dashboard
| Metric | Target | Current |
|---|---|---|
| Report volume (monthly) | | |
| Mean time to triage | < 10 min | |
| Confirmed phishing caught | | |
| User report rate (sim) | > 70% | |
| False positive rate | < 30% | |
| Top reporter recognition | Monthly | |

===== END FILE =====


===== FILE: skills/building-phishing-reporting-button-workflow/references/api-reference.md =====
# API Reference: Phishing Reporting Button Workflow

## Email Parsing (Python email module)
```python
from email import policy
from email.parser import BytesParser

with open("report.eml", "rb") as f:
    msg = BytesParser(policy=policy.default).parse(f)
headers = {
    "from": msg["From"], "subject": msg["Subject"],
    "reply_to": msg["Reply-To"], "received": msg.get_all("Received")
}
```

## Phishing Indicators
| Indicator | Weight | Description |
|-----------|--------|-------------|
| Reply-To mismatch | 20 | From and Reply-To differ |
| SPF/DKIM fail | 25 | Authentication failure |
| Suspicious language | 10 | Urgency/credential patterns |
| Suspicious URL | 15 | Known bad TLDs or redirectors |
| Dangerous attachment | 30 | Executable file extensions |

## VirusTotal URL Scan
```
GET https://www.virustotal.com/api/v3/urls/{url_id}
x-apikey: YOUR_KEY
```
URL ID = base64url(url) or sha256(url)

## Dangerous File Extensions
| Category | Extensions |
|----------|-----------|
| Executables | `.exe`, `.scr`, `.bat`, `.cmd` |
| Scripts | `.js`, `.vbs`, `.ps1`, `.hta` |
| Disk images | `.iso`, `.img`, `.vhd` |
| Archives | `.zip` (password-protected), `.rar` |
| Documents | `.docm`, `.xlsm` (macro-enabled) |

## Verdict Classification
| Score | Verdict | Action |
|-------|---------|--------|
| >= 50 | Phishing | Block sender, quarantine, create ticket |
| 25-49 | Suspicious | Analyst review required |
| < 25 | Benign | Close report, notify user |

## Ticketing Integration
```
POST /api/v2/tickets
Authorization: Bearer TOKEN
{
  "title": "Phishing Report: ...",
  "severity": "high",
  "description": "...",
  "indicators": ["Reply-To mismatch", ...]
}
```

## Microsoft Report Message Add-in
```
POST https://graph.microsoft.com/v1.0/users/{id}/messages/{msgId}/move
{"destinationId": "phishing-mailbox-id"}
```

===== END FILE =====


===== FILE: skills/building-phishing-reporting-button-workflow/references/standards.md =====
# Standards & References: Building Phishing Reporting Button Workflow

## MITRE ATT&CK References
- **T1566.001**: Phishing: Spearphishing Attachment
- **T1566.002**: Phishing: Spearphishing Link
- **T1204**: User Execution
- **D3-RERE**: User Reporting (MITRE D3FEND)

## Industry Standards
- **NIST SP 800-61 Rev.2**: Computer Security Incident Handling Guide
- **CIS Controls v8 Control 14**: Security Awareness and Skills Training
- **ISO 27001 A.6.3**: Information Security Awareness, Education and Training

## Reporting Platform Comparison
| Platform | Type | Integration | Auto-Triage |
|---|---|---|---|
| Microsoft Report Button | Built-in | M365 native | Via Sentinel/API |
| Cofense Reporter + Triage | Third-party | M365, Google | Yes (Cofense Triage) |
| KnowBe4 PAB | Third-party | M365, Google | Yes (KMSAT) |
| Proofpoint CLEAR | Third-party | M365, Google | Yes (built-in) |
| Hoxhunt | Third-party | M365, Google | Yes (AI-powered) |

## Key Metrics
- **Report Rate**: Percentage of phishing simulations reported (target: >70%)
- **Mean Time to Triage**: Time from report to classification (target: <10 min)
- **False Positive Rate**: Legitimate emails reported as phishing
- **Threat Catch Rate**: Real threats first detected by user reports
- **Reporter Accuracy**: Percentage of reports that are actual threats

===== END FILE =====


===== FILE: skills/building-phishing-reporting-button-workflow/references/workflows.md =====
# Workflows: Building Phishing Reporting Button Workflow

## Workflow 1: Automated Phishing Report Triage

```
User clicks "Report Phishing" button
  |
  v
[Email forwarded to reporting mailbox]
  +-- Original email preserved with full headers
  +-- Reporter identity recorded
  |
  v
[SOAR platform ingests report]
  |
  v
[Automated IOC extraction]
  +-- Extract sender address and domain
  +-- Extract all URLs from body
  +-- Extract attachment hashes (MD5, SHA256)
  +-- Parse email headers for authentication results
  |
  v
[Automated analysis (parallel)]
  +-- URLs -> VirusTotal, URLScan.io, PhishTank
  +-- Attachments -> Sandbox detonation
  +-- Sender -> Threat intelligence lookup
  +-- Headers -> SPF/DKIM/DMARC validation
  |
  v
[Classification]
  +-- CONFIRMED PHISHING: High-confidence malicious
  +-- SUSPICIOUS: Moderate indicators, needs analyst review
  +-- SPAM: Unwanted but not malicious
  +-- SIMULATION: Matches internal phishing test
  +-- CLEAN: Legitimate email, false report
  |
  v
[Automated response by classification]
  +-- PHISHING: Retract from all inboxes + block sender
  +-- SUSPICIOUS: Escalate to SOC analyst
  +-- SPAM: Move to junk for all recipients
  +-- SIMULATION: Credit reporter in training platform
  +-- CLEAN: Return to inbox
  |
  v
[Feedback to reporter]
  +-- "Thank you for reporting" (immediate)
  +-- Classification result (when complete)
  +-- Training tip (if false positive)
```

## Workflow 2: SOC Analyst Escalation

```
SOAR classifies report as SUSPICIOUS
  |
  v
[SOC analyst reviews]
  +-- Examine full email content and headers
  +-- Verify automated analysis results
  +-- Check for similar reports from other users
  |
  v
[Analyst decision]
  +-- Confirm malicious --> Trigger remediation playbook
  +-- Confirm clean --> Close and notify reporter
  +-- Need more info --> Contact reporter for context
```

===== END FILE =====


===== FILE: skills/building-phishing-reporting-button-workflow/scripts/agent.py =====
#!/usr/bin/env python3
"""Phishing Reporting Button Workflow Agent - Processes user-reported phishing emails via button integration."""

import json
import logging
import argparse
import re
import hashlib
from datetime import datetime
from email import policy
from email.parser import BytesParser

import requests

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

SUSPICIOUS_PATTERNS = [
    re.compile(r"urgent|immediate action|verify your account|click here now", re.IGNORECASE),
    re.compile(r"password.*expir|account.*suspend|unusual.*activity", re.IGNORECASE),
    re.compile(r"wire transfer|bitcoin|gift card|western union", re.IGNORECASE),
    re.compile(r"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}", re.IGNORECASE),
]

URL_PATTERN = re.compile(r"https?://[^\s\"'<>]+")


def parse_reported_email(eml_path):
    """Parse a reported phishing email from .eml file."""
    with open(eml_path, "rb") as f:
        msg = BytesParser(policy=policy.default).parse(f)
    headers = {
        "from": msg.get("From", ""),
        "to": msg.get("To", ""),
        "subject": msg.get("Subject", ""),
        "date": msg.get("Date", ""),
        "return_path": msg.get("Return-Path", ""),
        "reply_to": msg.get("Reply-To", ""),
        "message_id": msg.get("Message-ID", ""),
        "received": [str(h) for h in msg.get_all("Received", [])],
    }
    body = ""
    if msg.is_multipart():
        for part in msg.walk():
            ct = part.get_content_type()
            if ct == "text/plain":
                body = part.get_content()
                break
            elif ct == "text/html" and not body:
                body = part.get_content()
    else:
        body = msg.get_content()
    attachments = []
    for part in msg.walk():
        fn = part.get_filename()
        if fn:
            content = part.get_payload(decode=True)
            attachments.append({"filename": fn, "size": len(content) if content else 0,
                                "sha256": hashlib.sha256(content).hexdigest() if content else ""})
    logger.info("Parsed email: subject='%s', %d attachments", headers["subject"], len(attachments))
    return {"headers": headers, "body": body[:5000], "attachments": attachments}


def analyze_email(parsed):
    """Analyze reported email for phishing indicators."""
    findings = []
    score = 0
    headers = parsed["headers"]
    body = parsed.get("body", "")
    from_addr = headers.get("from", "")
    reply_to = headers.get("reply_to", "")
    if reply_to and reply_to != from_addr:
        findings.append({"indicator": "Reply-To mismatch", "detail": f"From: {from_addr}, Reply-To: {reply_to}", "weight": 20})
        score += 20
    auth_results = headers.get("authentication_results", "")
    if "spf=fail" in auth_results.lower() or "dkim=fail" in auth_results.lower():
        findings.append({"indicator": "Email authentication failure", "detail": auth_results[:200], "weight": 25})
        score += 25
    for pat in SUSPICIOUS_PATTERNS:
        matches = pat.findall(body)
        if matches:
            findings.append({"indicator": "Suspicious language", "detail": matches[0][:100], "weight": 10})
            score += 10
    urls = URL_PATTERN.findall(body)
    suspicious_urls = [u for u in urls if any(kw in u.lower() for kw in [".tk", ".ml", ".ga", "bit.ly",
                       "tinyurl", "redirect", "login", "verify", "secure-"])]
    if suspicious_urls:
        findings.append({"indicator": "Suspicious URLs", "detail": suspicious_urls[:5], "weight": 15})
        score += 15 * len(suspicious_urls[:3])
    for att in parsed.get("attachments", []):
        fn = att["filename"].lower()
        if any(fn.endswith(ext) for ext in [".exe", ".scr", ".js", ".vbs", ".bat", ".ps1", ".hta", ".iso", ".img"]):
            findings.append({"indicator": "Dangerous attachment", "detail": att["filename"], "weight": 30})
            score += 30
    verdict = "phishing" if score >= 50 else "suspicious" if score >= 25 else "benign"
    return {"score": min(score, 100), "verdict": verdict, "findings": findings, "url_count": len(urls),
            "attachment_count": len(parsed.get("attachments", []))}


def check_urls_virustotal(urls, api_key):
    """Check extracted URLs against VirusTotal."""
    results = []
    for url in urls[:5]:
        try:
            url_id = hashlib.sha256(url.encode()).hexdigest()
            resp = requests.get(f"https://www.virustotal.com/api/v3/urls/{url_id}",
                                headers={"x-apikey": api_key}, timeout=10)
            if resp.status_code == 200:
                stats = resp.json().get("data", {}).get("attributes", {}).get("last_analysis_stats", {})
                results.append({"url": url, "malicious": stats.get("malicious", 0), "suspicious": stats.get("suspicious", 0)})
        except requests.RequestException:
            results.append({"url": url, "error": "lookup_failed"})
    return results


def create_ticket(analysis, parsed, ticketing_url=None, api_key=None):
    """Create incident ticket from analysis results."""
    ticket = {
        "title": f"Phishing Report: {parsed['headers'].get('subject', 'No Subject')[:80]}",
        "severity": "high" if analysis["verdict"] == "phishing" else "medium" if analysis["verdict"] == "suspicious" else "low",
        "description": f"User-reported phishing email. Verdict: {analysis['verdict']} (score: {analysis['score']})",
        "indicators": [f["indicator"] for f in analysis["findings"]],
        "reported_from": parsed["headers"].get("to", ""),
        "suspicious_sender": parsed["headers"].get("from", ""),
    }
    if ticketing_url and api_key:
        try:
            resp = requests.post(f"{ticketing_url}/api/v2/tickets",
                                 headers={"Authorization": f"Bearer {api_key}"}, json=ticket, timeout=10)
            ticket["ticket_id"] = resp.json().get("id")
        except requests.RequestException:
            ticket["ticket_id"] = "creation_failed"
    return ticket


def generate_report(parsed, analysis, ticket):
    """Generate phishing report workflow report."""
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "email_subject": parsed["headers"].get("subject", ""),
        "sender": parsed["headers"].get("from", ""),
        "analysis": analysis,
        "ticket": ticket,
    }
    print(f"PHISHING REPORT: {analysis['verdict'].upper()} (score={analysis['score']}), "
          f"{len(analysis['findings'])} indicators")
    return report


def main():
    parser = argparse.ArgumentParser(description="Phishing Reporting Button Workflow Agent")
    parser.add_argument("--eml-file", required=True, help="Path to reported .eml file")
    parser.add_argument("--vt-key", help="VirusTotal API key for URL checks")
    parser.add_argument("--output", default="phishing_report.json")
    args = parser.parse_args()

    parsed = parse_reported_email(args.eml_file)
    analysis = analyze_email(parsed)
    ticket = create_ticket(analysis, parsed)
    report = generate_report(parsed, analysis, ticket)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/building-phishing-reporting-button-workflow/scripts/process.py =====
#!/usr/bin/env python3
"""
Phishing Report Triage Engine

Processes user-reported phishing emails, extracts IOCs,
performs automated analysis, and classifies the report.

Usage:
    python process.py triage --eml-file reported_email.eml
    python process.py metrics --reports-file reports.json
    python process.py extract-iocs --eml-file reported_email.eml
"""

import argparse
import json
import re
import hashlib
import sys
from dataclasses import dataclass, field, asdict
from collections import Counter
from datetime import datetime


@dataclass
class ExtractedIOCs:
    """IOCs extracted from reported email."""
    sender_address: str = ""
    sender_domain: str = ""
    reply_to: str = ""
    urls: list = field(default_factory=list)
    domains: list = field(default_factory=list)
    attachment_names: list = field(default_factory=list)
    attachment_hashes: list = field(default_factory=list)
    ip_addresses: list = field(default_factory=list)
    subject: str = ""


@dataclass
class TriageResult:
    """Triage classification result."""
    report_id: str = ""
    reporter: str = ""
    classification: str = ""
    confidence: float = 0.0
    iocs: dict = field(default_factory=dict)
    indicators: list = field(default_factory=list)
    recommended_action: str = ""
    auto_actionable: bool = False


@dataclass
class ReportingMetrics:
    """Phishing reporting program metrics."""
    total_reports: int = 0
    confirmed_phishing: int = 0
    confirmed_spam: int = 0
    simulation_reports: int = 0
    false_positives: int = 0
    mean_triage_time_min: float = 0.0
    top_reporters: list = field(default_factory=list)
    report_rate: float = 0.0


PHISHING_INDICATORS = [
    (r'\burgent\b.*\b(action|response|attention)\b', "Urgency language", 15),
    (r'\b(verify|confirm|validate)\s+your\s+(account|identity|password)\b', "Credential request", 20),
    (r'\b(click|follow)\s+(here|this|the)\s+(link|button)\b', "Click-bait language", 10),
    (r'\b(suspended|locked|disabled|compromised)\s+(account|access)\b', "Fear language", 15),
    (r'\b(wire\s+transfer|payment|invoice|bank)\b', "Financial language", 10),
    (r'\bgift\s+card\b', "Gift card request", 20),
    (r'\bdo\s+not\s+(share|tell|discuss)\b', "Secrecy language", 15),
]


def extract_iocs(eml_content: str) -> ExtractedIOCs:
    """Extract IOCs from email content."""
    iocs = ExtractedIOCs()

    # Extract From
    from_match = re.search(r'^From:\s*(?:.*<)?([^>\s]+@[^>\s]+)', eml_content,
                           re.MULTILINE | re.IGNORECASE)
    if from_match:
        iocs.sender_address = from_match.group(1).strip()
        domain_match = re.search(r'@([\w.-]+)', iocs.sender_address)
        if domain_match:
            iocs.sender_domain = domain_match.group(1)

    # Extract Reply-To
    reply_match = re.search(r'^Reply-To:\s*(?:.*<)?([^>\s]+@[^>\s]+)', eml_content,
                            re.MULTILINE | re.IGNORECASE)
    if reply_match:
        iocs.reply_to = reply_match.group(1).strip()

    # Extract Subject
    subj_match = re.search(r'^Subject:\s*(.+)$', eml_content, re.MULTILINE | re.IGNORECASE)
    if subj_match:
        iocs.subject = subj_match.group(1).strip()

    # Extract URLs
    urls = re.findall(r'https?://[^\s<>"\']+', eml_content)
    iocs.urls = list(set(urls))

    # Extract domains from URLs
    for url in iocs.urls:
        domain_match = re.search(r'https?://([^/:\s]+)', url)
        if domain_match:
            domain = domain_match.group(1).lower()
            if domain not in iocs.domains:
                iocs.domains.append(domain)

    # Extract IP addresses from headers
    ips = re.findall(r'\b(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})\b', eml_content)
    iocs.ip_addresses = list(set(ips))

    # Extract attachment filenames
    attachments = re.findall(
        r'filename[*]?=(?:"([^"]+)"|([^\s;]+))',
        eml_content, re.IGNORECASE
    )
    for groups in attachments:
        name = groups[0] or groups[1]
        if name and name not in iocs.attachment_names:
            iocs.attachment_names.append(name)

    return iocs


def triage_report(eml_content: str, simulation_subjects: list = None) -> TriageResult:
    """Classify a reported email."""
    result = TriageResult()
    iocs = extract_iocs(eml_content)
    result.iocs = asdict(iocs)

    score = 0
    body_lower = eml_content.lower()

    # Check if it's a known simulation
    if simulation_subjects:
        for sim_subj in simulation_subjects:
            if sim_subj.lower() in iocs.subject.lower():
                result.classification = "simulation"
                result.confidence = 0.95
                result.recommended_action = "Credit reporter in training platform"
                result.auto_actionable = True
                return result

    # Check phishing indicators
    for pattern, desc, weight in PHISHING_INDICATORS:
        if re.search(pattern, body_lower):
            result.indicators.append(desc)
            score += weight

    # Check for authentication failures
    auth_results = re.search(r'Authentication-Results:.*?(spf=fail|dkim=fail|dmarc=fail)',
                             eml_content, re.IGNORECASE | re.DOTALL)
    if auth_results:
        result.indicators.append(f"Authentication failure: {auth_results.group(1)}")
        score += 20

    # Check Reply-To mismatch
    if iocs.reply_to and iocs.sender_address:
        reply_domain = re.search(r'@([\w.-]+)', iocs.reply_to)
        sender_domain = re.search(r'@([\w.-]+)', iocs.sender_address)
        if reply_domain and sender_domain:
            if reply_domain.group(1) != sender_domain.group(1):
                result.indicators.append("Reply-To domain mismatch")
                score += 15

    # Check for suspicious attachment types
    risky_extensions = ['.exe', '.scr', '.bat', '.cmd', '.ps1', '.vbs',
                        '.js', '.wsf', '.hta', '.iso', '.img']
    for att in iocs.attachment_names:
        if any(att.lower().endswith(ext) for ext in risky_extensions):
            result.indicators.append(f"Risky attachment: {att}")
            score += 25

    # Classify
    if score >= 50:
        result.classification = "confirmed_phishing"
        result.confidence = min(score / 100, 0.95)
        result.recommended_action = "Retract from all inboxes, block sender domain"
        result.auto_actionable = True
    elif score >= 25:
        result.classification = "suspicious"
        result.confidence = score / 100
        result.recommended_action = "Escalate to SOC analyst for manual review"
        result.auto_actionable = False
    elif score >= 10:
        result.classification = "spam"
        result.confidence = 0.6
        result.recommended_action = "Move to junk for all recipients"
        result.auto_actionable = True
    else:
        result.classification = "clean"
        result.confidence = 0.7
        result.recommended_action = "Return to inbox, notify reporter"
        result.auto_actionable = True

    return result


def calculate_metrics(reports: list) -> ReportingMetrics:
    """Calculate phishing reporting program metrics."""
    metrics = ReportingMetrics()
    metrics.total_reports = len(reports)

    reporter_counts = Counter()
    triage_times = []

    for report in reports:
        classification = report.get("classification", "")
        if classification == "confirmed_phishing":
            metrics.confirmed_phishing += 1
        elif classification == "spam":
            metrics.confirmed_spam += 1
        elif classification == "simulation":
            metrics.simulation_reports += 1
        elif classification == "clean":
            metrics.false_positives += 1

        reporter = report.get("reporter", "")
        if reporter:
            reporter_counts[reporter] += 1

        triage_time = report.get("triage_time_minutes", 0)
        if triage_time > 0:
            triage_times.append(triage_time)

    if triage_times:
        metrics.mean_triage_time_min = sum(triage_times) / len(triage_times)

    metrics.top_reporters = [
        {"reporter": r, "count": c}
        for r, c in reporter_counts.most_common(10)
    ]

    if metrics.total_reports > 0:
        metrics.report_rate = (
            (metrics.confirmed_phishing + metrics.simulation_reports) /
            metrics.total_reports * 100
        )

    return metrics


def main():
    parser = argparse.ArgumentParser(description="Phishing Report Triage Engine")
    subparsers = parser.add_subparsers(dest="command")

    triage_parser = subparsers.add_parser("triage", help="Triage reported email")
    triage_parser.add_argument("--eml-file", required=True)
    triage_parser.add_argument("--sim-subjects", nargs="*", default=[])

    metrics_parser = subparsers.add_parser("metrics", help="Calculate reporting metrics")
    metrics_parser.add_argument("--reports-file", required=True)

    ioc_parser = subparsers.add_parser("extract-iocs", help="Extract IOCs from email")
    ioc_parser.add_argument("--eml-file", required=True)

    parser.add_argument("--json", action="store_true")
    args = parser.parse_args()

    if args.command == "triage":
        with open(args.eml_file, 'r', errors='replace') as f:
            content = f.read()
        result = triage_report(content, args.sim_subjects)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(f"Classification: {result.classification}")
            print(f"Confidence: {result.confidence:.0%}")
            print(f"Action: {result.recommended_action}")
            print(f"Auto-actionable: {'Yes' if result.auto_actionable else 'No'}")
            if result.indicators:
                print(f"Indicators:")
                for ind in result.indicators:
                    print(f"  - {ind}")

    elif args.command == "metrics":
        with open(args.reports_file) as f:
            reports = json.load(f)
        result = calculate_metrics(reports)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(f"Total reports: {result.total_reports}")
            print(f"Confirmed phishing: {result.confirmed_phishing}")
            print(f"Spam: {result.confirmed_spam}")
            print(f"Simulations reported: {result.simulation_reports}")
            print(f"False positives: {result.false_positives}")
            print(f"Mean triage time: {result.mean_triage_time_min:.1f} min")

    elif args.command == "extract-iocs":
        with open(args.eml_file, 'r', errors='replace') as f:
            content = f.read()
        iocs = extract_iocs(content)
        print(json.dumps(asdict(iocs), indent=2))

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise/SKILL.md =====
---
name: detecting-business-email-compromise
description: Detect Business Email Compromise (BEC) fraud, where attackers impersonate
  executives or vendors to trick employees into wire transfers or data disclosure without
  malicious links, using email gateway rules, behavioral analytics, and financial process
  controls. Use when investigating a suspected executive/vendor impersonation email or
  building detection rules for social-engineering fund-transfer fraud.
domain: cybersecurity
subdomain: phishing-defense
tags:
- phishing
- email-security
- social-engineering
- dmarc
- awareness
- bec
- fraud
version: '1.0'
author: mahipal
license: Apache-2.0
atlas_techniques:
- AML.T0052
- AML.T0088
nist_ai_rmf:
- GOVERN-6.2
- MAP-5.2
d3fend_techniques:
- Restore Object
- Restore Configuration
- Application Configuration Hardening
- Application Hardening
- Disable Remote Access
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566.002
- T1534
- T1114.002
- T1657
- T1078.004
mitre_f3:
  version: '1.1'
  tactics:
  - initial-access
  - positioning
  - monetization
  - stealth
  techniques:
  - id: T1672
    name: Email Spoofing
    tactic: stealth
    source: attack
  - id: F1032
    name: Impersonate Official
    tactic: initial-access
    source: f3
  - id: F1036
    name: New Vendor Setup
    tactic: positioning
    source: f3
  - id: F1005.006
    name: 'Account Manipulation: Change of Payment Details'
    tactic: positioning
    source: f3
  - id: F1025.003
    name: 'Electronic Funds Transfer: Wire Transfer'
    tactic: monetization
    source: f3
  - id: F1022
    name: Delete Relevant Emails
    tactic: stealth
    source: f3
---
# Detecting Business Email Compromise

## Overview
Business Email Compromise (BEC) is a sophisticated fraud scheme where attackers impersonate executives, vendors, or trusted partners to trick employees into transferring funds, sharing sensitive data, or changing payment details. Unlike traditional phishing, BEC often contains no malicious links or attachments, relying purely on social engineering. This skill covers detection techniques using email gateway rules, behavioral analytics, and financial process controls.


## When to Use

- When investigating security incidents that require detecting business email compromise
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites
- Email security gateway with BEC detection capabilities
- Understanding of organizational financial processes and approval chains
- Access to email logs and SIEM platform
- Knowledge of social engineering tactics

## Key Concepts

### BEC Attack Types (FBI IC3 Classification)
1. **CEO Fraud**: Attacker impersonates CEO, requests urgent wire transfer
2. **Account Compromise**: Employee email compromised, used to request payments from vendors
3. **False Invoice Scheme**: Fake invoices from "vendor" with changed bank details
4. **Attorney Impersonation**: Impersonates legal counsel for urgent confidential transfers
5. **Data Theft**: Requests W-2, tax forms, or PII from HR

### Detection Indicators
- Urgency and secrecy language ("confidential", "do not discuss with others")
- New or changed payment instructions
- Executive communication outside normal patterns
- Display name matches executive but email domain differs
- Reply-to address differs from From address
- First-time communication pattern between sender and recipient
- Request for gift cards or cryptocurrency

## Workflow

### Step 1: Configure BEC-Specific Email Rules
- Flag emails with VIP display names from external domains
- Detect financial keywords combined with urgency language
- Alert on first-time sender to finance/accounting staff
- Check for Reply-To domain mismatch

### Step 2: Deploy Behavioral Analytics
- Baseline normal communication patterns per user
- Detect anomalous requests (unusual recipient, unusual time, unusual request type)
- Monitor for email forwarding rule changes (T1114.003)

### Step 3: Implement Financial Controls
- Dual-authorization for wire transfers above threshold
- Out-of-band verification for payment detail changes (phone callback)
- Vendor payment change verification process
- Finance team training on BEC red flags

### Step 4: Monitor for Account Compromise
- Detect impossible travel in email login locations
- Alert on email forwarding rule creation
- Monitor for mailbox delegation changes
- Check for inbox rules hiding BEC-related emails

## Tools & Resources
- **Microsoft Defender for O365 Anti-BEC**: Built-in BEC detection
- **Proofpoint Email Fraud Defense**: BEC-specific solution
- **Abnormal Security**: AI-driven BEC detection
- **FBI IC3 BEC Advisory**: https://www.ic3.gov/
- **FinCEN BEC Advisory**: Financial institution guidance

## Validation
- BEC detection rules trigger on test scenarios
- Financial controls prevent unauthorized transfers in drills
- Account compromise detection catches simulated attacks
- Reduced BEC susceptibility in awareness assessments

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise/LICENSE =====

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


===== FILE: skills/detecting-business-email-compromise/assets/template.md =====
# BEC Detection and Response Template

## VIP Protection List
| Name | Title | Email Domain | Protected Since |
|---|---|---|---|
| | CEO | | |
| | CFO | | |
| | CTO | | |

## BEC Detection Rules Deployed
| Rule | Condition | Action | Status |
|---|---|---|---|
| VIP name spoofing | External email with VIP display name | Quarantine + Alert | Active |
| Financial + Urgency | Wire transfer keywords + urgency | Tag + Alert | Active |
| Reply-To mismatch | Reply-To domain differs from From | Tag + Log | Active |
| Gift card request | Gift card keywords from "executive" | Quarantine | Active |
| Vendor payment change | Bank detail change language | Tag + Finance alert | Active |

## Financial Controls
- [ ] Dual authorization for transfers > $[threshold]
- [ ] Out-of-band verification for payment detail changes
- [ ] Vendor payment change requires callback on known number
- [ ] Gift card purchases require in-person manager approval

## BEC Incident Response Contacts
| Role | Name | Phone | Email |
|---|---|---|---|
| SOC Lead | | | |
| Finance Controller | | | |
| Bank Fraud Department | | | |
| FBI IC3 (filing) | N/A | N/A | ic3.gov |
| Legal Counsel | | | |

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise/references/api-reference.md =====
# API Reference: Detecting Business Email Compromise

## Python email Library

```python
import email
from email import policy

# Parse .eml file
with open("message.eml") as f:
    msg = email.message_from_file(f, policy=policy.default)

msg.get("From")                # sender header
msg.get("Reply-To")            # reply-to header
msg.get("Authentication-Results")  # SPF/DKIM/DMARC results
body = msg.get_body(preferencelist=("plain", "html"))
body.get_content()             # decoded body text
```

## Authentication Header Patterns

| Result | Meaning |
|--------|---------|
| `spf=pass` | Sender IP authorized by domain SPF record |
| `spf=fail` | Sender IP NOT in SPF record |
| `dkim=pass` | DKIM signature valid |
| `dkim=fail` | DKIM signature invalid or missing |
| `dmarc=pass` | SPF or DKIM aligned with From domain |
| `dmarc=fail` | Neither SPF nor DKIM aligned |

## BEC Attack Types (FBI IC3)

| Type | Description |
|------|-------------|
| CEO Fraud | Impersonates executive requesting wire transfer |
| Invoice Fraud | Fake invoice with changed bank details |
| Account Compromise | Compromised email used for payment requests |
| Attorney Impersonation | Urgent legal matter requiring funds |
| Data Theft | Requests for W-2 / PII from HR |

## BEC Indicator Regex Patterns

```python
# Financial urgency
r"\b(wire transfer|bank transfer|routing number)\b"
# Secrecy pressure
r"\b(confidential|do not share|keep this between us)\b"
# Gift card fraud
r"\b(gift card|bitcoin|crypto|western union)\b"
# Account change
r"\b(change.*(bank|account|payment))\b"
```

## Microsoft Graph API - Mail Security

```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=internetMessageHeaders/any(h: h/name eq 'Authentication-Results')
Authorization: Bearer {token}
```

## CLI Usage

```bash
python agent.py --email-file suspicious.eml --vip-names "John Smith" "Jane CEO"
python agent.py --scan-dir /var/mail/quarantine/ --vip-names "CFO Name"
```

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise/references/standards.md =====
# Standards & References: Detecting Business Email Compromise

## FBI IC3 BEC Classification
- **Type 1**: CEO Fraud / Executive Impersonation
- **Type 2**: Account Compromise (compromised employee email)
- **Type 3**: False Invoice / Vendor Email Compromise
- **Type 4**: Attorney Impersonation
- **Type 5**: Data Theft (W-2/PII requests)

## MITRE ATT&CK References
- **T1566.001/002**: Phishing (initial access for BEC)
- **T1534**: Internal Spearphishing
- **T1114.003**: Email Collection: Email Forwarding Rule
- **T1098.002**: Account Manipulation: Additional Email Delegate Access
- **T1586.002**: Compromise Accounts: Email Accounts

## NIST / Regulatory
- **NIST SP 800-177 Rev.1**: Trustworthy Email
- **FinCEN Advisory FIN-2019-A005**: Advisory on BEC targeting businesses
- **FBI IC3 Annual Report**: BEC statistics and trends ($2.9B losses in 2023)

## Detection Rule Categories
| Rule | Description | Priority |
|---|---|---|
| VIP impersonation | External email with internal VIP display name | Critical |
| Payment language | Wire transfer/payment keywords + urgency | High |
| Reply-to mismatch | Reply-to domain differs from From | High |
| First-time sender | No prior communication with recipient | Medium |
| Forwarding rule | New auto-forward rule to external address | Critical |
| Gift card request | Request for gift card purchase | High |
| Vendor change | Payment detail change notification | High |

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise/references/workflows.md =====
# Workflows: Detecting Business Email Compromise

## Workflow 1: BEC Detection Pipeline

```
Inbound email arrives
  |
  v
[Check: Does display name match VIP list?]
  +-- YES + external domain --> HIGH ALERT: Possible CEO fraud
  +-- NO --> Continue standard checks
  |
  v
[Check: Financial keywords present?]
  +-- "wire transfer", "payment", "invoice", "bank details" detected
  +-- Combined with urgency: "urgent", "confidential", "today"
  +-- YES --> ELEVATED: Flag for finance team review
  |
  v
[Check: Reply-To mismatch?]
  +-- Reply-To domain differs from From domain
  +-- YES --> HIGH: Likely BEC attempt
  |
  v
[Check: Communication pattern anomaly?]
  +-- First-time sender to finance/HR staff
  +-- Unusual time of day for this sender
  +-- YES --> MEDIUM: Requires verification
  |
  v
[Decision]
  +-- BLOCK: High-confidence BEC
  +-- QUARANTINE: Moderate confidence
  +-- TAG: Warning banner for recipient
  +-- DELIVER: Low risk
```

## Workflow 2: BEC Incident Response

```
BEC attempt detected or reported
  |
  v
[Immediate actions (first 30 minutes)]
  +-- Quarantine the email
  +-- Search for similar messages to other recipients
  +-- Alert affected users not to comply with request
  |
  v
[Investigation (next 2 hours)]
  +-- Analyze email headers for true origin
  +-- Check if any user already complied (check sent folders)
  +-- If payment was made: Initiate bank recall immediately
  +-- Search for compromised accounts (forwarding rules, login anomalies)
  |
  v
[Containment]
  +-- Block sender domain/IP
  +-- If account compromised: Force password reset, revoke sessions
  +-- Remove malicious forwarding rules
  +-- Notify finance to halt pending payments
  |
  v
[Recovery]
  +-- Work with bank for fund recovery
  +-- File FBI IC3 report (ic3.gov)
  +-- Notify affected parties
  +-- Update detection rules
  +-- Targeted training for affected employees
```

## Workflow 3: Vendor Payment Change Verification

```
Vendor requests payment detail change
  |
  v
[Do NOT use contact info from the email]
  |
  v
[Look up vendor contact from existing records]
  |
  v
[Call vendor using known phone number]
  +-- Verify the payment change request is legitimate
  |
  +-- CONFIRMED --> Process change with dual authorization
  +-- NOT CONFIRMED --> Report as BEC attempt, block sender
  +-- UNABLE TO REACH --> Hold payment, escalate
```

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise/scripts/agent.py =====
#!/usr/bin/env python3
"""BEC detection agent - analyzes email headers and content for Business Email Compromise indicators.

Parses email headers for spoofing signals, checks DMARC/SPF/DKIM alignment,
detects urgency language patterns, and flags financial request anomalies.
"""

import argparse
import email
import json
import re
from email import policy
from pathlib import Path

BEC_URGENCY_PATTERNS = [
    r"\b(urgent|immediately|asap|right away|time.?sensitive)\b",
    r"\b(confidential|do not share|keep this between us|don't tell)\b",
    r"\b(wire transfer|bank transfer|payment|invoice|routing number)\b",
    r"\b(gift card|bitcoin|crypto|western union|moneygram)\b",
    r"\b(ceo|cfo|president|director) (asked|requested|needs|wants)\b",
    r"\b(change.*(bank|account|payment)|new.*(bank|account|routing))\b",
    r"\b(act now|deadline today|end of day|before close)\b",
]

EXECUTIVE_TITLES = ["ceo", "cfo", "coo", "cto", "president", "chairman",
                    "managing director", "vice president", "vp", "director"]


def parse_email_file(filepath):
    with open(filepath, "r", encoding="utf-8", errors="replace") as f:
        return email.message_from_file(f, policy=policy.default)


def check_spf_dkim_dmarc(msg):
    results = {"spf": "none", "dkim": "none", "dmarc": "none"}
    auth_results = msg.get("Authentication-Results", "")
    if "spf=pass" in auth_results.lower():
        results["spf"] = "pass"
    elif "spf=fail" in auth_results.lower():
        results["spf"] = "fail"
    if "dkim=pass" in auth_results.lower():
        results["dkim"] = "pass"
    elif "dkim=fail" in auth_results.lower():
        results["dkim"] = "fail"
    if "dmarc=pass" in auth_results.lower():
        results["dmarc"] = "pass"
    elif "dmarc=fail" in auth_results.lower():
        results["dmarc"] = "fail"
    return results


def check_display_name_spoofing(msg, vip_names):
    from_header = msg.get("From", "")
    match = re.match(r'"?([^"<]+)"?\s*<([^>]+)>', from_header)
    if not match:
        return None
    display_name = match.group(1).strip().lower()
    email_addr = match.group(2).strip().lower()
    for vip in vip_names:
        if vip.lower() in display_name:
            domain = email_addr.split("@")[-1] if "@" in email_addr else ""
            return {"display_name": display_name, "email": email_addr,
                    "matched_vip": vip, "domain": domain,
                    "indicator": "Display name matches VIP but email may be external"}
    return None


def check_reply_to_mismatch(msg):
    from_addr = msg.get("From", "")
    reply_to = msg.get("Reply-To", "")
    if not reply_to:
        return None
    from_match = re.search(r'<([^>]+)>', from_addr) or re.search(r'(\S+@\S+)', from_addr)
    reply_match = re.search(r'<([^>]+)>', reply_to) or re.search(r'(\S+@\S+)', reply_to)
    if from_match and reply_match:
        from_email = from_match.group(1).lower()
        reply_email = reply_match.group(1).lower()
        from_domain = from_email.split("@")[-1]
        reply_domain = reply_email.split("@")[-1]
        if from_domain != reply_domain:
            return {"from": from_email, "reply_to": reply_email,
                    "indicator": "Reply-To domain differs from From domain"}
    return None


def detect_urgency_language(body):
    matches = []
    for pattern in BEC_URGENCY_PATTERNS:
        found = re.findall(pattern, body, re.IGNORECASE)
        if found:
            matches.extend(found)
    return matches


def calculate_bec_score(auth, spoofing, reply_mismatch, urgency_matches):
    score = 0
    if auth.get("spf") == "fail":
        score += 25
    if auth.get("dkim") == "fail":
        score += 20
    if auth.get("dmarc") == "fail":
        score += 30
    if spoofing:
        score += 35
    if reply_mismatch:
        score += 25
    score += min(len(urgency_matches) * 10, 40)
    return min(score, 100)


def analyze_email(filepath, vip_names):
    msg = parse_email_file(filepath)
    body = msg.get_body(preferencelist=("plain", "html"))
    body_text = body.get_content() if body else ""

    auth = check_spf_dkim_dmarc(msg)
    spoofing = check_display_name_spoofing(msg, vip_names)
    reply_mismatch = check_reply_to_mismatch(msg)
    urgency = detect_urgency_language(body_text)
    score = calculate_bec_score(auth, spoofing, reply_mismatch, urgency)

    risk = "CRITICAL" if score >= 70 else "HIGH" if score >= 50 else "MEDIUM" if score >= 30 else "LOW"

    return {
        "file": str(filepath),
        "from": msg.get("From", ""),
        "to": msg.get("To", ""),
        "subject": msg.get("Subject", ""),
        "date": msg.get("Date", ""),
        "authentication": auth,
        "display_name_spoofing": spoofing,
        "reply_to_mismatch": reply_mismatch,
        "urgency_indicators": urgency,
        "bec_score": score,
        "risk_level": risk,
    }


def main():
    parser = argparse.ArgumentParser(description="BEC Email Analyzer")
    parser.add_argument("--email-file", required=True, help="Path to .eml file")
    parser.add_argument("--vip-names", nargs="+", default=[], help="VIP display names to check")
    parser.add_argument("--scan-dir", help="Scan all .eml files in directory")
    args = parser.parse_args()

    results = []
    if args.scan_dir:
        for eml in Path(args.scan_dir).glob("*.eml"):
            results.append(analyze_email(str(eml), args.vip_names))
    else:
        results.append(analyze_email(args.email_file, args.vip_names))

    print(json.dumps(results, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise/scripts/process.py =====
#!/usr/bin/env python3
"""
Business Email Compromise (BEC) Detection Engine

Analyzes emails for BEC indicators including executive impersonation,
financial urgency, payment change requests, and communication anomalies.

Usage:
    python process.py detect --email-json email.json
    python process.py analyze-log --log-file email_log.json
    python process.py vip-list --add "John CEO" --email "john@company.com"
"""

import argparse
import json
import re
import sys
from datetime import datetime, timezone
from dataclasses import dataclass, field, asdict
from collections import defaultdict


@dataclass
class BECIndicator:
    """A BEC detection indicator."""
    category: str = ""
    description: str = ""
    severity: str = "medium"
    confidence: float = 0.0
    bec_type: str = ""


@dataclass
class BECAnalysis:
    """Complete BEC analysis result."""
    from_address: str = ""
    from_display_name: str = ""
    to_address: str = ""
    subject: str = ""
    indicators: list = field(default_factory=list)
    bec_score: float = 0.0
    bec_type: str = "unknown"
    is_bec: bool = False
    recommended_action: str = ""


# Financial keywords
FINANCIAL_KEYWORDS = [
    r'\bwire\s+transfer\b', r'\bbank\s+transfer\b', r'\bpayment\b',
    r'\binvoice\b', r'\bpurchase\s+order\b', r'\baccount\s+number\b',
    r'\brouting\s+number\b', r'\biban\b', r'\bswift\b', r'\back\b',
    r'\bgift\s+card\b', r'\bbitcoin\b', r'\bcrypto\b', r'\bvenmo\b',
    r'\bzelle\b', r'\bpaypal\b', r'\bw-2\b', r'\btax\s+form\b',
]

# Urgency keywords
URGENCY_KEYWORDS = [
    r'\burgent\b', r'\bimmediately\b', r'\basap\b', r'\btoday\b',
    r'\bright\s+now\b', r'\btime\s+sensitive\b', r'\bdo\s+not\s+(share|tell|discuss)\b',
    r'\bconfidential\b', r'\bkeep\s+this\s+between\b', r'\bquietly\b',
    r'\bbefore\s+end\s+of\s+day\b', r'\bcritical\b', r'\boverdue\b',
]

# Authority/impersonation keywords
AUTHORITY_KEYWORDS = [
    r'\bi\s+need\s+you\s+to\b', r'\bplease\s+handle\b',
    r'\bi\'m\s+in\s+a\s+meeting\b', r'\bi\'m\s+traveling\b',
    r'\bdon\'t\s+call\s+me\b', r'\bemail\s+me\s+back\b',
    r'\bcan\s+you\s+take\s+care\s+of\b', r'\bapproved\s+by\b',
]


def detect_bec(headers: dict, body: str = "", vip_list: list = None,
               internal_domains: list = None) -> BECAnalysis:
    """Analyze email for BEC indicators."""
    analysis = BECAnalysis()
    analysis.from_address = headers.get("from", "")
    analysis.from_display_name = headers.get("from_display_name", "")
    analysis.to_address = headers.get("to", "")
    analysis.subject = headers.get("subject", "")

    from_domain = ""
    match = re.search(r'@([\w.-]+)', analysis.from_address)
    if match:
        from_domain = match.group(1).lower()

    if internal_domains is None:
        internal_domains = []

    score = 0.0
    full_text = f"{analysis.subject} {body}".lower()

    # Check 1: VIP display name impersonation
    if vip_list and analysis.from_display_name:
        name_lower = analysis.from_display_name.lower()
        for vip in vip_list:
            vip_name = vip.get("name", "").lower()
            vip_domain = vip.get("domain", "").lower()
            if vip_name and vip_name in name_lower:
                if from_domain and vip_domain and from_domain != vip_domain:
                    analysis.indicators.append(BECIndicator(
                        category="vip_impersonation",
                        description=f"Display name '{analysis.from_display_name}' matches VIP "
                                    f"'{vip.get('name')}' but email is from external domain '{from_domain}'",
                        severity="critical",
                        confidence=0.9,
                        bec_type="ceo_fraud"
                    ))
                    score += 35

    # Check 2: Financial keywords
    financial_matches = []
    for pattern in FINANCIAL_KEYWORDS:
        if re.search(pattern, full_text, re.IGNORECASE):
            financial_matches.append(pattern)

    if financial_matches:
        analysis.indicators.append(BECIndicator(
            category="financial_language",
            description=f"Found {len(financial_matches)} financial keyword(s)",
            severity="medium",
            confidence=min(len(financial_matches) * 0.2, 0.8),
            bec_type="payment_fraud"
        ))
        score += min(len(financial_matches) * 5, 20)

    # Check 3: Urgency keywords
    urgency_matches = []
    for pattern in URGENCY_KEYWORDS:
        if re.search(pattern, full_text, re.IGNORECASE):
            urgency_matches.append(pattern)

    if urgency_matches:
        analysis.indicators.append(BECIndicator(
            category="urgency_language",
            description=f"Found {len(urgency_matches)} urgency/secrecy keyword(s)",
            severity="medium",
            confidence=min(len(urgency_matches) * 0.2, 0.8),
            bec_type="social_engineering"
        ))
        score += min(len(urgency_matches) * 5, 15)

    # Check 4: Combined financial + urgency = higher risk
    if financial_matches and urgency_matches:
        analysis.indicators.append(BECIndicator(
            category="combined_financial_urgency",
            description="Financial request combined with urgency/secrecy language - strong BEC signal",
            severity="high",
            confidence=0.8,
            bec_type="ceo_fraud"
        ))
        score += 20

    # Check 5: Authority language
    authority_matches = []
    for pattern in AUTHORITY_KEYWORDS:
        if re.search(pattern, full_text, re.IGNORECASE):
            authority_matches.append(pattern)

    if authority_matches and (financial_matches or urgency_matches):
        analysis.indicators.append(BECIndicator(
            category="authority_language",
            description="Authority/directive language combined with financial or urgency content",
            severity="high",
            confidence=0.7,
            bec_type="ceo_fraud"
        ))
        score += 15

    # Check 6: Reply-to mismatch
    reply_to = headers.get("reply_to", "")
    if reply_to:
        reply_domain = ""
        match = re.search(r'@([\w.-]+)', reply_to)
        if match:
            reply_domain = match.group(1).lower()
        if reply_domain and from_domain and reply_domain != from_domain:
            analysis.indicators.append(BECIndicator(
                category="reply_to_mismatch",
                description=f"Reply-To ({reply_domain}) differs from From ({from_domain})",
                severity="high",
                confidence=0.85,
                bec_type="account_compromise"
            ))
            score += 20

    # Check 7: External sender to finance/HR (if role info available)
    to_role = headers.get("to_role", "").lower()
    if from_domain and internal_domains and from_domain not in internal_domains:
        if any(r in to_role for r in ["finance", "accounting", "payroll", "hr", "human resources"]):
            analysis.indicators.append(BECIndicator(
                category="external_to_finance",
                description=f"External sender to {to_role} staff",
                severity="medium",
                confidence=0.5,
                bec_type="vendor_fraud"
            ))
            score += 10

    # Calculate final verdict
    analysis.bec_score = min(score, 100)
    if analysis.bec_score >= 60:
        analysis.is_bec = True
        analysis.recommended_action = "BLOCK and alert SOC"
    elif analysis.bec_score >= 40:
        analysis.is_bec = True
        analysis.recommended_action = "QUARANTINE for manual review"
    elif analysis.bec_score >= 20:
        analysis.recommended_action = "TAG with warning banner"
    else:
        analysis.recommended_action = "DELIVER normally"

    # Determine most likely BEC type
    type_scores = defaultdict(float)
    for ind in analysis.indicators:
        type_scores[ind.bec_type] += ind.confidence * 10
    if type_scores:
        analysis.bec_type = max(type_scores, key=type_scores.get)

    return analysis


def format_bec_report(analysis: BECAnalysis) -> str:
    """Format BEC analysis as text report."""
    lines = []
    lines.append("=" * 60)
    lines.append("  BUSINESS EMAIL COMPROMISE DETECTION REPORT")
    lines.append("=" * 60)
    lines.append(f"  BEC Score: {analysis.bec_score:.0f}/100")
    lines.append(f"  Verdict: {'BEC DETECTED' if analysis.is_bec else 'NOT DETECTED'}")
    lines.append(f"  BEC Type: {analysis.bec_type}")
    lines.append(f"  Action: {analysis.recommended_action}")
    lines.append("")
    lines.append(f"  From: {analysis.from_display_name} <{analysis.from_address}>")
    lines.append(f"  To: {analysis.to_address}")
    lines.append(f"  Subject: {analysis.subject}")
    lines.append("")

    if analysis.indicators:
        lines.append(f"[INDICATORS] ({len(analysis.indicators)})")
        for i, ind in enumerate(analysis.indicators, 1):
            lines.append(f"  {i}. [{ind.severity.upper()}] {ind.description}")
            lines.append(f"     Category: {ind.category} | Confidence: {ind.confidence:.0%}")
    lines.append("=" * 60)
    return "\n".join(lines)


def main():
    parser = argparse.ArgumentParser(description="BEC Detection Engine")
    subparsers = parser.add_subparsers(dest="command")

    detect_parser = subparsers.add_parser("detect", help="Detect BEC in email")
    detect_parser.add_argument("--email-json", help="Email JSON file")
    detect_parser.add_argument("--from", dest="from_addr")
    detect_parser.add_argument("--from-name", default="")
    detect_parser.add_argument("--to", dest="to_addr", default="")
    detect_parser.add_argument("--subject", default="")
    detect_parser.add_argument("--body", default="")
    detect_parser.add_argument("--vip-file", help="VIP list JSON file")
    detect_parser.add_argument("--internal-domains", nargs="+", default=[])

    log_parser = subparsers.add_parser("analyze-log", help="Analyze email log for BEC")
    log_parser.add_argument("--log-file", required=True)
    log_parser.add_argument("--vip-file")
    log_parser.add_argument("--internal-domains", nargs="+", default=[])

    parser.add_argument("--json", action="store_true")

    args = parser.parse_args()

    vip_list = []
    vip_file = getattr(args, "vip_file", None)
    if vip_file:
        with open(vip_file) as f:
            vip_list = json.load(f)

    if args.command == "detect":
        if args.email_json:
            with open(args.email_json) as f:
                email_data = json.load(f)
            headers = email_data.get("headers", email_data)
            body = email_data.get("body", "")
        else:
            headers = {
                "from": args.from_addr or "",
                "from_display_name": args.from_name,
                "to": args.to_addr,
                "subject": args.subject,
            }
            body = args.body

        analysis = detect_bec(headers, body, vip_list,
                              getattr(args, "internal_domains", []))
        if args.json:
            print(json.dumps(asdict(analysis), indent=2, default=str))
        else:
            print(format_bec_report(analysis))

    elif args.command == "analyze-log":
        with open(args.log_file) as f:
            log = json.load(f)

        bec_count = 0
        for entry in log:
            analysis = detect_bec(
                entry.get("headers", entry),
                entry.get("body", ""),
                vip_list,
                getattr(args, "internal_domains", [])
            )
            if analysis.is_bec:
                bec_count += 1
                if args.json:
                    print(json.dumps(asdict(analysis), indent=2, default=str))
                else:
                    print(format_bec_report(analysis))
                print()

        print(f"\nTotal analyzed: {len(log)}, BEC detected: {bec_count}")

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise-with-ai/SKILL.md =====
---
name: detecting-business-email-compromise-with-ai
description: Deploy AI and NLP-powered detection systems to identify business email
  compromise attacks by analyzing writing style, behavioral patterns, and contextual
  anomalies that evade traditional rule-based filters.
domain: cybersecurity
subdomain: phishing-defense
tags:
- bec
- ai
- nlp
- machine-learning
- email-security
- behavioral-analytics
- impersonation
- fraud-detection
version: '1.0'
author: mahipal
license: Apache-2.0
atlas_techniques:
- AML.T0073
- AML.T0052
- AML.T0088
nist_ai_rmf:
- GOVERN-6.2
- MAP-5.2
- GOVERN-6.1
- MEASURE-2.7
- MEASURE-2.5
d3fend_techniques:
- Sender MTA Reputation Analysis
- Email Filtering
- Sender Reputation Analysis
- Homoglyph Detection
- Message Analysis
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566.002
- T1534
- T1114.002
- T1657
- T1078.004
mitre_f3:
  version: '1.1'
  tactics:
  - initial-access
  - stealth
  - positioning
  - monetization
  techniques:
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: T1672
    name: Email Spoofing
    tactic: stealth
    source: attack
  - id: F1032
    name: Impersonate Official
    tactic: initial-access
    source: f3
  - id: F1005.006
    name: 'Account Manipulation: Change of Payment Details'
    tactic: positioning
    source: f3
  - id: F1022
    name: Delete Relevant Emails
    tactic: stealth
    source: f3
  - id: F1025.003
    name: 'Electronic Funds Transfer: Wire Transfer'
    tactic: monetization
    source: f3
---
# Detecting Business Email Compromise with AI

## Overview
AI-powered BEC detection uses machine learning, NLP, and behavioral analytics to identify sophisticated impersonation attacks that contain no malicious links or attachments. Traditional rule-based filters miss these attacks because BEC relies purely on social engineering. Modern AI approaches analyze writing style, tone, vocabulary, grammatical patterns, and behavioral context to determine if an email genuinely comes from the stated sender. BERT-based models achieve 98.65% accuracy in BEC detection, and AI-enhanced platforms show a 25% increase in phishing identification over keyword-based rules.


## When to Use

- When investigating security incidents that require detecting business email compromise with ai
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites
- AI-powered email security platform (Abnormal Security, Tessian, Microsoft Defender)
- Historical email data for baseline training (minimum 30 days)
- Integration with email platform (Microsoft 365 or Google Workspace)
- SIEM for alert correlation and investigation
- Understanding of BEC attack types (FBI IC3 classification)

## Workflow

### Step 1: Deploy AI Email Security Platform
- Select API-based solution (Abnormal Security, Tessian, Ironscales) or enhance existing SEG
- Connect to Microsoft Graph API or Google Workspace API
- Allow 48-hour baseline learning period on historical email data
- Configure integration to scan inbound, outbound, and internal email
- Verify API permissions for message access and remediation

### Step 2: Configure Behavioral Baselines
- AI learns normal communication patterns: who emails whom, frequency, tone
- Establish writing style profiles for each user (vocabulary, sentence structure)
- Map typical request types per role (finance processes payments, HR handles PII)
- Baseline email metadata: typical sending times, devices, locations
- Flag deviations from established baselines as anomalous

### Step 3: Train NLP Models for BEC Detection
- Deploy transformer-based models (BERT, GPT) for email content analysis
- Detect urgency and manipulation language patterns
- Identify mismatches between sender identity and writing style
- Analyze sentiment shifts indicating social engineering pressure
- Classify email intent: information request, payment request, credential request

### Step 4: Configure Detection Policies
- VIP impersonation: AI compares new email against known executive communication patterns
- Vendor impersonation: detect payment change requests from vendor lookalike domains
- Account compromise: detect sudden changes in employee email behavior
- Supply chain BEC: monitor for impersonation of trusted partners
- Configure confidence thresholds for auto-block vs. warning banner vs. analyst review

### Step 5: Integrate with Response Workflow
- Auto-quarantine high-confidence BEC detections
- Add warning banners for moderate-confidence detections
- Route suspicious emails to SOC analyst queue for review
- Integrate with SOAR for automated response playbooks
- Feed BEC verdicts back into training data for model improvement

## Tools & Resources
- **Abnormal Security**: API-based AI email security with behavioral analysis
- **Microsoft Defender for O365**: Built-in AI anti-BEC with Impostor Classifier
- **Tessian (Proofpoint)**: AI-powered email security with human layer protection
- **Ironscales**: AI + human-in-the-loop BEC detection
- **Darktrace Email**: Self-learning AI for email threat detection

## Validation
- AI detects test BEC email with no malicious indicators (pure social engineering)
- Writing style analysis identifies impersonation of known executive
- Behavioral baseline flags unusual payment request from compromised account
- NLP correctly classifies urgency manipulation in test scenario
- False positive rate below 0.05% after baseline training
- Detection rate exceeds traditional rule-based filters by 25%+

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise-with-ai/LICENSE =====

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


===== FILE: skills/detecting-business-email-compromise-with-ai/assets/template.md =====
# AI-Powered BEC Detection Template

## AI Platform Configuration
| Setting | Value | Status |
|---|---|---|
| Platform | | |
| Integration | API-based (Microsoft Graph) | |
| Baseline training period | 30+ days | |
| Scanning scope | Inbound + Internal + Outbound | |

## Detection Thresholds
| Score Range | Classification | Action |
|---|---|---|
| 90-100% | High-confidence BEC | Auto-quarantine + SOC alert |
| 70-89% | Moderate BEC | Warning banner + analyst queue |
| 50-69% | Low-confidence BEC | Warning banner only |
| < 50% | Likely legitimate | Deliver normally |

## VIP Protection List
| Name | Title | Email | Writing Style Profiled |
|---|---|---|---|
| | CEO | | |
| | CFO | | |
| | CTO | | |

## Model Performance Metrics
| Metric | Target | Current |
|---|---|---|
| Detection accuracy | > 98% | |
| False positive rate | < 0.05% | |
| Mean detection time | < 5 sec | |
| BEC catch rate vs. rules | +25% | |

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise-with-ai/references/api-reference.md =====
# API Reference: Detecting BEC with AI

## NLP Feature Extraction

| Feature | Description | BEC Signal |
|---------|-------------|------------|
| urgency_score | Ratio of urgency words to total | High = suspicious |
| pressure_score | Ratio of secrecy/pressure words | High = suspicious |
| financial_score | Ratio of financial terms | High = suspicious |
| authority_score | Ratio of executive title mentions | High = suspicious |
| caps_ratio | Uppercase character ratio | High = aggressive tone |
| unique_word_ratio | Vocabulary diversity metric | Low = template-like |

## scikit-learn Classification Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.ensemble import RandomForestClassifier

pipeline = Pipeline([
    ("tfidf", TfidfVectorizer(max_features=5000, ngram_range=(1, 2))),
    ("clf", RandomForestClassifier(n_estimators=100, random_state=42))
])
pipeline.fit(X_train, y_train)
predictions = pipeline.predict(X_test)
```

## Writing Style Analysis (Stylometry)

```python
# Sentence length distribution for author verification
import re, math
sentences = re.split(r'[.!?]+', text)
lengths = [len(s.split()) for s in sentences if s.strip()]
mean_len = sum(lengths) / len(lengths)
variance = sum((l - mean_len)**2 for l in lengths) / len(lengths)
std_dev = math.sqrt(variance)
```

## Microsoft Graph API - Suspicious Mail Rules

```http
GET https://graph.microsoft.com/v1.0/users/{id}/mailFolders/inbox/messageRules
Authorization: Bearer {token}

# Detect forwarding rules (T1114.003)
GET https://graph.microsoft.com/v1.0/users/{id}/mailFolders/inbox/messageRules?$filter=actions/forwardTo ne null
```

## Impersonation Signal Patterns

```python
# Mobile signature (creates urgency excuse)
r"sent from my (iphone|ipad|android|mobile)"
# Discourages verification
r"(please|kindly).*(do not|don't).*(reply|respond|call)"
# Unavailability excuse
r"(i am|i'm).*(in a meeting|traveling|on a flight)"
# Time pressure
r"(handle|process|complete).*(today|immediately|by end of day)"
```

## CLI Usage

```bash
python agent.py --file email_body.txt
python agent.py --file email_body.txt --baseline-file sender_style.json
```

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise-with-ai/references/standards.md =====
# Standards & References: Detecting BEC with AI

## MITRE ATT&CK References
- **T1566.001/002**: Phishing (Spearphishing Attachment/Link)
- **T1534**: Internal Spearphishing
- **T1656**: Impersonation
- **T1586.002**: Compromise Accounts: Email Accounts
- **T1114.003**: Email Collection: Email Forwarding Rule

## AI/ML Techniques for BEC Detection
| Technique | Application | Accuracy |
|---|---|---|
| BERT embeddings + SVC | Email classification | 98.65% |
| Transformer NLP | Writing style analysis | 96%+ |
| Anomaly detection | Behavioral baseline deviation | 94%+ |
| Graph neural networks | Communication pattern analysis | 93%+ |
| Sentiment analysis | Urgency/manipulation detection | 91%+ |

## FBI IC3 BEC Statistics
- $2.9 billion losses reported in 2023
- BEC accounts for 27% of all cybercrime financial losses
- Average loss per BEC incident: $125,000
- 21,832 BEC complaints filed in 2023

## Detection Categories
- **Impostor Detection**: AI identifies display name/domain impersonation
- **Account Takeover Detection**: Behavioral anomalies from compromised accounts
- **Writing Style Analysis**: NLP compares email to sender's historical style
- **Intent Classification**: ML classifies email as payment/credential/data request
- **Relationship Analysis**: Graph analysis of sender-recipient communication patterns

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise-with-ai/references/workflows.md =====
# Workflows: Detecting BEC with AI

## Workflow 1: AI-Powered BEC Detection Pipeline

```
Inbound email arrives
  |
  v
[Feature extraction]
  +-- Sender metadata (domain, IP, authentication)
  +-- Email content (subject, body, NLP features)
  +-- Behavioral context (communication history, timing)
  +-- Relationship graph (sender-recipient pattern)
  |
  v
[Multi-model analysis (parallel)]
  +-- Impostor classifier: Display name/domain impersonation
  +-- NLP model: Writing style vs. sender baseline
  +-- Behavioral model: Request anomaly detection
  +-- Intent classifier: Payment/credential/data request
  |
  v
[Confidence scoring]
  +-- Aggregate model outputs
  +-- Weight by model confidence and context
  +-- Generate overall BEC probability score
  |
  v
[Action]
  +-- Score >= 90%: Auto-quarantine + SOC alert
  +-- Score 70-89%: Warning banner + analyst queue
  +-- Score 50-69%: Warning banner only
  +-- Score < 50%: Deliver normally
```

## Workflow 2: Model Feedback Loop

```
BEC verdict generated
  |
  v
[User/analyst feedback]
  +-- User reports false positive (legitimate email flagged)
  +-- Analyst confirms true positive (BEC caught)
  +-- User reports missed BEC (false negative)
  |
  v
[Feedback integration]
  +-- Update sender trust score
  +-- Retrain model with corrected labels
  +-- Adjust confidence thresholds
  +-- Update behavioral baselines
```

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise-with-ai/scripts/agent.py =====
#!/usr/bin/env python3
"""AI-powered BEC detection agent using NLP features for email classification.

Extracts linguistic features (urgency, sentiment, writing style metrics) and
uses scikit-learn to classify emails as BEC or legitimate.
"""

import argparse
import json
import math
import re
from collections import Counter

URGENCY_WORDS = {"urgent", "immediately", "asap", "deadline", "critical",
                 "important", "expedite", "priority", "rush", "now"}
PRESSURE_WORDS = {"confidential", "secret", "private", "classified",
                  "between us", "do not share", "don't discuss", "quiet"}
FINANCIAL_WORDS = {"wire", "transfer", "payment", "invoice", "bank",
                   "account", "routing", "ach", "swift", "funds"}
AUTHORITY_WORDS = {"ceo", "cfo", "president", "director", "boss",
                   "chairman", "executive", "management", "vp"}


def extract_features(text):
    words = text.lower().split()
    word_count = len(words) if words else 1
    sentences = re.split(r'[.!?]+', text)
    sentence_count = len([s for s in sentences if s.strip()]) or 1

    word_freq = Counter(words)
    unique_ratio = len(set(words)) / word_count

    urgency_score = sum(1 for w in words if w.strip(".,!?") in URGENCY_WORDS) / word_count
    pressure_score = sum(1 for w in words if w.strip(".,!?") in PRESSURE_WORDS) / word_count
    financial_score = sum(1 for w in words if w.strip(".,!?") in FINANCIAL_WORDS) / word_count
    authority_score = sum(1 for w in words if w.strip(".,!?") in AUTHORITY_WORDS) / word_count

    exclamation_ratio = text.count("!") / sentence_count
    caps_ratio = sum(1 for c in text if c.isupper()) / max(len(text), 1)
    avg_word_len = sum(len(w) for w in words) / word_count
    avg_sentence_len = word_count / sentence_count

    return {
        "word_count": word_count,
        "sentence_count": sentence_count,
        "unique_word_ratio": round(unique_ratio, 4),
        "urgency_score": round(urgency_score, 4),
        "pressure_score": round(pressure_score, 4),
        "financial_score": round(financial_score, 4),
        "authority_score": round(authority_score, 4),
        "exclamation_ratio": round(exclamation_ratio, 4),
        "caps_ratio": round(caps_ratio, 4),
        "avg_word_length": round(avg_word_len, 2),
        "avg_sentence_length": round(avg_sentence_len, 2),
    }


def compute_bec_probability(features):
    weights = {
        "urgency_score": 3.5, "pressure_score": 3.0, "financial_score": 4.0,
        "authority_score": 2.5, "exclamation_ratio": 1.0, "caps_ratio": 1.5,
    }
    raw = sum(features.get(k, 0) * w for k, w in weights.items())
    probability = 1 / (1 + math.exp(-10 * (raw - 0.15)))
    return round(probability, 4)


def analyze_writing_style(text):
    sentences = [s.strip() for s in re.split(r'[.!?]+', text) if s.strip()]
    if not sentences:
        return {"style_consistency": 1.0}
    lengths = [len(s.split()) for s in sentences]
    mean_len = sum(lengths) / len(lengths)
    variance = sum((l - mean_len) ** 2 for l in lengths) / len(lengths)
    std_dev = math.sqrt(variance)
    return {
        "mean_sentence_length": round(mean_len, 2),
        "sentence_length_std": round(std_dev, 2),
        "style_consistency": round(1 - min(std_dev / 20, 1), 4),
    }


def detect_impersonation_signals(text, known_sender_style=None):
    signals = []
    if re.search(r"sent from my (iphone|ipad|android|mobile)", text, re.IGNORECASE):
        signals.append("mobile_signature_present")
    if re.search(r"(please|kindly).*(do not|don't).*(reply|respond|call)", text, re.IGNORECASE):
        signals.append("discourages_verification")
    if re.search(r"(i am|i'm).*(in a meeting|traveling|on a flight|busy)", text, re.IGNORECASE):
        signals.append("unavailability_excuse")
    if re.search(r"(handle|process|complete).*(today|immediately|by end of day)", text, re.IGNORECASE):
        signals.append("time_pressure")
    if known_sender_style:
        current = analyze_writing_style(text)
        diff = abs(current["mean_sentence_length"] - known_sender_style.get("mean_sentence_length", 15))
        if diff > 8:
            signals.append("writing_style_deviation")
    return signals


def analyze_email(text, known_style=None):
    features = extract_features(text)
    probability = compute_bec_probability(features)
    style = analyze_writing_style(text)
    signals = detect_impersonation_signals(text, known_style)

    risk = "CRITICAL" if probability > 0.8 else "HIGH" if probability > 0.6 else \
           "MEDIUM" if probability > 0.3 else "LOW"

    return {
        "features": features,
        "writing_style": style,
        "impersonation_signals": signals,
        "bec_probability": probability,
        "risk_level": risk,
    }


def main():
    parser = argparse.ArgumentParser(description="AI-Powered BEC Detection")
    parser.add_argument("--file", required=True, help="Email text file to analyze")
    parser.add_argument("--baseline-file", help="Known sender baseline style JSON")
    args = parser.parse_args()

    with open(args.file, "r", encoding="utf-8") as f:
        text = f.read()

    known_style = None
    if args.baseline_file:
        with open(args.baseline_file, "r") as f:
            known_style = json.load(f)

    result = analyze_email(text, known_style)
    print(json.dumps(result, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-business-email-compromise-with-ai/scripts/process.py =====
#!/usr/bin/env python3
"""
AI-Powered BEC Detection Engine

Combines NLP analysis, behavioral scoring, and impersonation
detection to identify Business Email Compromise attacks.

Usage:
    python process.py detect --email-json email.json --baseline-file baselines.json
    python process.py train-baseline --email-log emails.json --output baselines.json
"""

import argparse
import json
import re
import sys
import math
from dataclasses import dataclass, field, asdict
from collections import defaultdict, Counter


@dataclass
class BECAIResult:
    """AI-powered BEC detection result."""
    from_address: str = ""
    subject: str = ""
    impostor_score: float = 0.0
    nlp_score: float = 0.0
    behavioral_score: float = 0.0
    intent_class: str = ""
    overall_score: float = 0.0
    is_bec: bool = False
    confidence: float = 0.0
    action: str = ""
    indicators: list = field(default_factory=list)


# NLP feature patterns
URGENCY_PATTERNS = [
    (r'\burgent(ly)?\b', 3), (r'\bimmediately\b', 3), (r'\basap\b', 3),
    (r'\btime.?sensitive\b', 2), (r'\btoday\b', 1), (r'\bright\s+now\b', 2),
    (r'\bdeadline\b', 1), (r'\boverdue\b', 2), (r'\bcritical\b', 2),
]

SECRECY_PATTERNS = [
    (r'\bconfidential\b', 2), (r'\bdo\s+not\s+(share|tell|discuss)\b', 3),
    (r'\bkeep.*between\s+us\b', 3), (r'\bquietly\b', 2),
    (r'\bdon.t\s+mention\b', 2), (r'\bprivate\s+matter\b', 2),
]

FINANCIAL_PATTERNS = [
    (r'\bwire\s+transfer\b', 3), (r'\binvoice\b', 2), (r'\bpayment\b', 2),
    (r'\bbank\s+(account|details|transfer)\b', 3), (r'\bgift\s+card\b', 4),
    (r'\bbitcoin\b', 3), (r'\baccount\s+number\b', 2), (r'\bswift\b', 2),
]

AUTHORITY_PATTERNS = [
    (r'\bi\s+need\s+you\s+to\b', 2), (r'\bi.m\s+in\s+a\s+meeting\b', 3),
    (r'\bhandle\s+this\b', 2), (r'\bi\s+authorize\b', 2),
    (r'\bapproved\s+by\s+me\b', 3), (r'\bdon.t\s+call\b', 2),
]


def compute_nlp_score(text: str) -> tuple:
    """Compute NLP-based BEC score from email text."""
    text_lower = text.lower()
    scores = {"urgency": 0, "secrecy": 0, "financial": 0, "authority": 0}
    indicators = []

    for pattern, weight in URGENCY_PATTERNS:
        if re.search(pattern, text_lower):
            scores["urgency"] += weight
            indicators.append(f"Urgency: {pattern}")

    for pattern, weight in SECRECY_PATTERNS:
        if re.search(pattern, text_lower):
            scores["secrecy"] += weight
            indicators.append(f"Secrecy: {pattern}")

    for pattern, weight in FINANCIAL_PATTERNS:
        if re.search(pattern, text_lower):
            scores["financial"] += weight
            indicators.append(f"Financial: {pattern}")

    for pattern, weight in AUTHORITY_PATTERNS:
        if re.search(pattern, text_lower):
            scores["authority"] += weight
            indicators.append(f"Authority: {pattern}")

    total = sum(scores.values())
    normalized = min(total / 25.0, 1.0)  # Normalize to 0-1

    # Determine intent
    intent = "unknown"
    if scores["financial"] > 3:
        intent = "payment_request"
    elif scores["authority"] > 3 and scores["urgency"] > 2:
        intent = "directive"
    elif scores["secrecy"] > 2:
        intent = "sensitive_request"

    return normalized, intent, indicators


def compute_impostor_score(email_data: dict, vip_list: list = None) -> tuple:
    """Check for sender impersonation."""
    score = 0.0
    indicators = []

    from_name = email_data.get("from_display_name", "").lower()
    from_email = email_data.get("from", "")
    reply_to = email_data.get("reply_to", "")

    from_domain = ""
    match = re.search(r'@([\w.-]+)', from_email)
    if match:
        from_domain = match.group(1).lower()

    # VIP name match from external domain
    if vip_list:
        for vip in vip_list:
            vip_name = vip.get("name", "").lower()
            vip_domain = vip.get("domain", "").lower()
            if vip_name and vip_name in from_name:
                if from_domain and vip_domain and from_domain != vip_domain:
                    score += 0.5
                    indicators.append(
                        f"Display name '{from_name}' matches VIP from external domain"
                    )

    # Reply-to mismatch
    if reply_to:
        reply_domain = ""
        match = re.search(r'@([\w.-]+)', reply_to)
        if match:
            reply_domain = match.group(1).lower()
        if reply_domain and from_domain and reply_domain != from_domain:
            score += 0.3
            indicators.append(f"Reply-to domain mismatch: {reply_domain} vs {from_domain}")

    return min(score, 1.0), indicators


def compute_behavioral_score(email_data: dict, baseline: dict) -> tuple:
    """Score behavioral anomalies against baseline."""
    score = 0.0
    indicators = []

    sender = email_data.get("from", "")
    recipient = email_data.get("to", "")
    hour = email_data.get("send_hour", -1)

    sender_baseline = baseline.get(sender, {})

    # Check if first-time sender to this recipient
    known_recipients = sender_baseline.get("recipients", [])
    if recipient and known_recipients and recipient not in known_recipients:
        score += 0.3
        indicators.append("First-time communication with this recipient")

    # Check unusual sending time
    typical_hours = sender_baseline.get("typical_hours", [])
    if hour >= 0 and typical_hours and hour not in typical_hours:
        score += 0.2
        indicators.append(f"Unusual sending hour: {hour}:00")

    # Check if request type is unusual for sender
    typical_topics = sender_baseline.get("typical_topics", [])
    subject = email_data.get("subject", "").lower()
    if typical_topics:
        financial = any(w in subject for w in ["payment", "wire", "invoice", "transfer"])
        if financial and "financial" not in typical_topics:
            score += 0.3
            indicators.append("Financial request from sender who doesn't typically discuss finances")

    return min(score, 1.0), indicators


def detect_bec_ai(email_data: dict, baseline: dict = None,
                  vip_list: list = None) -> BECAIResult:
    """Run full AI BEC detection pipeline."""
    result = BECAIResult()
    result.from_address = email_data.get("from", "")
    result.subject = email_data.get("subject", "")

    body = email_data.get("body", "")
    full_text = f"{result.subject} {body}"

    # NLP analysis
    result.nlp_score, result.intent_class, nlp_indicators = compute_nlp_score(full_text)
    result.indicators.extend(nlp_indicators)

    # Impostor detection
    result.impostor_score, imp_indicators = compute_impostor_score(email_data, vip_list or [])
    result.indicators.extend(imp_indicators)

    # Behavioral analysis
    if baseline:
        result.behavioral_score, beh_indicators = compute_behavioral_score(
            email_data, baseline
        )
        result.indicators.extend(beh_indicators)

    # Weighted aggregate score
    weights = {"nlp": 0.35, "impostor": 0.40, "behavioral": 0.25}
    result.overall_score = (
        result.nlp_score * weights["nlp"] +
        result.impostor_score * weights["impostor"] +
        result.behavioral_score * weights["behavioral"]
    )
    result.confidence = min(result.overall_score * 1.2, 1.0)

    # Classification
    if result.overall_score >= 0.7:
        result.is_bec = True
        result.action = "AUTO-QUARANTINE + SOC alert"
    elif result.overall_score >= 0.5:
        result.is_bec = True
        result.action = "WARNING BANNER + analyst queue"
    elif result.overall_score >= 0.3:
        result.action = "WARNING BANNER only"
    else:
        result.action = "DELIVER normally"

    return result


def train_baseline(emails: list) -> dict:
    """Build behavioral baselines from historical email data."""
    baselines = defaultdict(lambda: {
        "recipients": set(),
        "typical_hours": [],
        "typical_topics": set(),
        "email_count": 0,
    })

    for email in emails:
        sender = email.get("from", "")
        if not sender:
            continue

        b = baselines[sender]
        b["email_count"] += 1

        recipient = email.get("to", "")
        if recipient:
            b["recipients"].add(recipient)

        hour = email.get("send_hour", -1)
        if hour >= 0:
            b["typical_hours"].append(hour)

        subject = email.get("subject", "").lower()
        if any(w in subject for w in ["payment", "wire", "invoice"]):
            b["typical_topics"].add("financial")
        if any(w in subject for w in ["meeting", "schedule", "calendar"]):
            b["typical_topics"].add("scheduling")

    # Convert sets to lists for JSON serialization
    result = {}
    for sender, data in baselines.items():
        hours = data["typical_hours"]
        unique_hours = list(set(hours)) if hours else []
        result[sender] = {
            "recipients": list(data["recipients"]),
            "typical_hours": unique_hours,
            "typical_topics": list(data["typical_topics"]),
            "email_count": data["email_count"],
        }

    return result


def main():
    parser = argparse.ArgumentParser(description="AI-Powered BEC Detection")
    subparsers = parser.add_subparsers(dest="command")

    detect_parser = subparsers.add_parser("detect", help="Detect BEC in email")
    detect_parser.add_argument("--email-json", required=True)
    detect_parser.add_argument("--baseline-file")
    detect_parser.add_argument("--vip-file")

    train_parser = subparsers.add_parser("train-baseline", help="Train behavioral baseline")
    train_parser.add_argument("--email-log", required=True)
    train_parser.add_argument("--output", required=True)

    parser.add_argument("--json", action="store_true")
    args = parser.parse_args()

    if args.command == "detect":
        with open(args.email_json) as f:
            email_data = json.load(f)
        baseline = {}
        if args.baseline_file:
            with open(args.baseline_file) as f:
                baseline = json.load(f)
        vip_list = []
        if args.vip_file:
            with open(args.vip_file) as f:
                vip_list = json.load(f)

        result = detect_bec_ai(email_data, baseline, vip_list)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(f"BEC Score: {result.overall_score:.0%}")
            print(f"Is BEC: {'YES' if result.is_bec else 'No'}")
            print(f"Intent: {result.intent_class}")
            print(f"Action: {result.action}")
            if result.indicators:
                print("Indicators:")
                for ind in result.indicators:
                    print(f"  - {ind}")

    elif args.command == "train-baseline":
        with open(args.email_log) as f:
            emails = json.load(f)
        baseline = train_baseline(emails)
        with open(args.output, 'w') as f:
            json.dump(baseline, f, indent=2)
        print(f"Baseline trained for {len(baseline)} senders")

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-qr-code-phishing-with-email-security/SKILL.md =====
---
name: detecting-qr-code-phishing-with-email-security
description: Detect and prevent QR code phishing (quishing) attacks that embed
  malicious URLs inside QR code images to bypass link-based email security,
  using image-based threat detection, OCR/QR decoding, and mobile-side scanning
  (Microsoft Defender for O365, Proofpoint TAP, Barracuda Multimodal AI). Use
  when configuring gateway rules against QR phishing or investigating suspicious
  emails containing QR codes.
domain: cybersecurity
subdomain: phishing-defense
tags:
- quishing
- qr-code
- phishing
- email-security
- image-analysis
- ocr
- mobile-security
version: '1.0'
author: mahipal
license: Apache-2.0
atlas_techniques:
- AML.T0052
- AML.T0024
- AML.T0035
nist_ai_rmf:
- MEASURE-2.8
- MAP-5.1
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566
- T1598
- T1534
- T1036
- T1027
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
# Detecting QR Code Phishing with Email Security

## Overview
QR code phishing (quishing) is a rapidly growing attack vector where malicious URLs are embedded in QR code images within phishing emails. Quishing incidents grew fivefold from 46,000 to 250,000 between August and November 2025, with credential phishing comprising 89.3% of detected incidents. Traditional email security filters struggle because QR codes cannot be read by humans or standard URL scanners, and when scanned, users typically use personal mobile devices that lack corporate security controls. Attackers have evolved to use split QR codes (two separate images), nested QR codes, and ASCII text-based QR codes to evade detection.


## When to Use

- When investigating security incidents that require detecting qr code phishing with email security
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites
- Email security gateway with image analysis capabilities
- Understanding of QR code structure and encoding
- Mobile device management (MDM) or mobile threat defense solution
- Security awareness training program
- SIEM platform for correlation and alerting

## Key Concepts

### Why Quishing Works
1. **Bypasses URL Scanners**: Traditional gateways scan text-based URLs but cannot decode image-embedded URLs
2. **Shifts to Unprotected Devices**: Corporate email arrives on secured systems but QR scan occurs on personal mobile devices
3. **User Trust**: QR codes are normalized in daily life (payments, menus, parking)
4. **Low Detection Rate**: Only 36% of quishing incidents are accurately identified by recipients

### Evasion Techniques (2025)
- **Split QR Codes**: QR code divided into two separate images that look benign individually (Gabagool PhaaS kit)
- **Nested QR Codes**: QR code within a QR code, with first scan leading to intermediate page
- **ASCII QR Codes**: QR rendered as text characters instead of images, bypassing image analysis (12% of attacks in Jan 2026)
- **Styled/Artistic QR Codes**: Custom-designed QR codes with logos that evade pattern matching
- **PDF Attachment QR**: QR code embedded in PDF attachment rather than email body

### Detection Challenges
- Pattern-based detection faces trade-off: aggressive tuning causes false positives, cautious tuning causes misses
- Average similarity score of 0.209 between quishing and legitimate QR emails
- QR codes in image attachments require OCR and deep image processing

## Workflow

### Step 1: Enable Image-Based Threat Detection
- Configure email gateway to scan embedded images for QR codes
- Enable OCR processing on image attachments (PNG, JPG, GIF, BMP)
- Deploy multimodal AI that combines image processing, OCR, and NLP analysis
- Configure PDF scanning to detect QR codes within attachments
- Set up detection for ASCII/text-based QR code rendering

### Step 2: Configure QR Code URL Analysis
- Extract URLs from detected QR codes and submit to URL reputation services
- Apply same URL scanning policies to QR-extracted URLs as text-based URLs
- Enable real-time sandbox analysis for QR-decoded destination pages
- Configure time-of-click protection for QR-extracted URLs where possible
- Block known phishing domains extracted from QR codes

### Step 3: Deploy Mobile-Side Protection
- Implement mobile threat defense (MTD) with QR code scanning capability
- Deploy Palo Alto ALFA or equivalent safe-by-design QR scanning
- Configure MDM policies to warn users before opening scanned URLs
- Enable corporate VPN/secure browser for QR-scanned destinations
- Block known credential harvesting domains at the mobile proxy level

### Step 4: Build Detection Rules
- Alert on emails containing only an image and minimal text (common quishing pattern)
- Flag emails with QR code images from external first-time senders
- Detect urgency language combined with QR code presence
- Alert on emails impersonating IT/security team requesting QR scan for MFA setup
- Monitor for common quishing themes: MFA reset, document signing, voicemail notification

### Step 5: Train Users on Quishing Recognition
- Update security awareness program to include QR code phishing scenarios
- Conduct quishing simulation campaigns using controlled QR codes
- Teach users to verify QR destination URLs before entering credentials
- Establish reporting process for suspicious QR code emails
- Distribute guidance on safe QR scanning practices

## Tools & Resources
- **Barracuda Multimodal AI**: OCR + deep image processing for QR detection
- **Palo Alto ALFA**: Safe-by-design QR code scanning assessment
- **Microsoft Defender for O365**: QR code detection in email images
- **Proofpoint TAP**: Image-based threat analysis with QR decoding
- **Lookout/Zimperium**: Mobile threat defense with QR scanning

## Validation
- QR code phishing emails detected in controlled testing
- Split QR code and ASCII QR code evasion techniques caught
- QR-extracted URLs submitted to sandbox analysis
- Mobile devices alert on malicious QR destinations
- User reporting rate for quishing simulations exceeds 50%
- False positive rate for QR detection below 1%

===== END FILE =====


===== FILE: skills/detecting-qr-code-phishing-with-email-security/LICENSE =====

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


===== FILE: skills/detecting-qr-code-phishing-with-email-security/assets/template.md =====
# QR Code Phishing Detection Template

## Quishing Detection Rules
| Rule | Condition | Action | Status |
|---|---|---|---|
| QR in image | QR code detected in embedded image | Extract URL + analyze | |
| QR in PDF | QR code detected in PDF attachment | Extract URL + analyze | |
| ASCII QR | Text-rendered QR code pattern | Decode + analyze | |
| Image-only email | Email with image but minimal text | Flag for review | |
| MFA QR theme | QR + MFA/authentication language | High priority alert | |

## Common Quishing Themes to Monitor
- [ ] MFA enrollment requiring QR code scan
- [ ] Document signing via QR code
- [ ] Voicemail access via QR code
- [ ] Package delivery confirmation QR
- [ ] IT security update with QR authentication
- [ ] Shared document access via QR

## Mobile Protection Controls
- [ ] Mobile threat defense deployed with QR scanning
- [ ] MDM policy warns before opening QR URLs
- [ ] Corporate VPN required for QR destinations
- [ ] User training on safe QR scanning completed

## Incident Response for Quishing
| Step | Action | Responsible |
|---|---|---|
| 1 | Decode QR and extract URL | SOC Analyst |
| 2 | Check if URL is active phishing page | SOC Analyst |
| 3 | Search for same email across all mailboxes | Email Admin |
| 4 | Retract email from all recipients | Email Admin |
| 5 | Block URL at web proxy/firewall | Network Security |
| 6 | Check if any user scanned and submitted credentials | SOC Lead |
| 7 | Force password reset for affected users | IAM Team |

===== END FILE =====


===== FILE: skills/detecting-qr-code-phishing-with-email-security/references/api-reference.md =====
# API Reference: QR Code Phishing Detection

## pyzbar — QR/Barcode Decoding

### Installation
```bash
pip install pyzbar Pillow
# On Linux: apt-get install libzbar0
```

### Core Functions
```python
from pyzbar.pyzbar import decode
from PIL import Image

results = decode(Image.open("qr.png"))
for r in results:
    print(r.type)     # "QRCODE"
    print(r.data)     # b"https://..."
    print(r.rect)     # Rect(left=40, top=40, width=200, height=200)
```

### Decoded Object Attributes
| Attribute | Type | Description |
|-----------|------|-------------|
| `data` | bytes | Decoded content |
| `type` | str | Barcode type (QRCODE, EAN13, etc.) |
| `rect` | Rect | Bounding rectangle |
| `polygon` | list | Corner points |
| `quality` | int | Decode quality score |

## Python email Module — EML Parsing

### Parsing an EML file
```python
import email
from email import policy

with open("message.eml", "rb") as f:
    msg = email.message_from_binary_file(f, policy=policy.default)

subject = msg["Subject"]
sender = msg["From"]
```

### Walking MIME Parts
```python
for part in msg.walk():
    ctype = part.get_content_type()
    if ctype.startswith("image/"):
        payload = part.get_payload(decode=True)
        filename = part.get_filename()
```

## URL Analysis Indicators

### Suspicious TLD List
`.xyz`, `.top`, `.club`, `.work`, `.buzz`, `.tk`, `.ml`, `.ga`, `.cf`, `.gq`

### Phishing URL Patterns
| Pattern | Risk |
|---------|------|
| IP address in domain | High |
| Domain > 40 chars | Medium |
| HTTP (no TLS) | Medium |
| 3+ subdomains | Medium |
| URL shortener | High |
| Base64 in path | High |

## Microsoft Defender for Office 365 — Safe Links API

### Check URL reputation
```http
POST https://graph.microsoft.com/v1.0/security/tiIndicators
Content-Type: application/json
Authorization: Bearer {token}

{
  "targetProduct": "Azure Sentinel",
  "threatType": "Phishing",
  "url": "https://suspicious-domain.xyz/login"
}
```

## VirusTotal URL Scan API

### Submit URL
```http
POST https://www.virustotal.com/api/v3/urls
x-apikey: {API_KEY}
Content-Type: application/x-www-form-urlencoded

url=https://suspicious-domain.xyz
```

### Response Fields
| Field | Description |
|-------|-------------|
| `data.attributes.last_analysis_stats.malicious` | Engines flagging as malicious |
| `data.attributes.last_analysis_stats.harmless` | Engines flagging as clean |
| `data.attributes.categories` | URL categorization |

===== END FILE =====


===== FILE: skills/detecting-qr-code-phishing-with-email-security/references/standards.md =====
# Standards & References: Detecting QR Code Phishing

## Industry Statistics (2025-2026)
- Quishing incidents grew from 46,000 to 250,000 between Aug-Nov 2025 (Kaspersky)
- 89.3% of QR code phishing targets credential theft
- 12% of January 2026 attacks used ASCII text-based QR codes
- Only 36% of quishing incidents accurately identified by recipients
- 25% year-over-year growth in quishing incidents

## MITRE ATT&CK References
- **T1566.001**: Phishing: Spearphishing Attachment (QR in PDF/image)
- **T1566.002**: Phishing: Spearphishing Link (QR-encoded URL)
- **T1204.001**: User Execution: Malicious Link (user scans QR)
- **T1598.003**: Phishing for Information: Spearphishing Link

## Quishing Attack Patterns
| Pattern | Description | Detection Difficulty |
|---|---|---|
| Inline QR image | QR code embedded directly in email body | Medium |
| PDF attachment QR | QR code inside attached PDF document | High |
| Split QR code | QR divided into two benign-looking images | Very High |
| ASCII QR code | QR rendered as text characters | Very High |
| Nested QR code | QR within QR with intermediate redirect | High |
| Styled QR code | Artistic QR with logos/colors | Medium |

## Common Quishing Themes
- MFA enrollment/reset requiring QR scan
- Document signing via QR code
- Voicemail notification with QR access
- Package delivery QR confirmation
- IT security update requiring QR authentication
- Shared document access via QR

## Detection Technologies
- Multimodal AI (OCR + deep image + NLP)
- Computer vision QR code detection
- URL reputation analysis for decoded URLs
- Mobile threat defense QR scanning
- Behavioral analysis of image-only emails

===== END FILE =====


===== FILE: skills/detecting-qr-code-phishing-with-email-security/references/workflows.md =====
# Workflows: Detecting QR Code Phishing

## Workflow 1: QR Code Email Detection Pipeline

```
Inbound email arrives at gateway
  |
  v
[Standard text/URL scanning]
  +-- Check text-based URLs (standard pipeline)
  +-- No malicious URLs found in text
  |
  v
[Image analysis module]
  +-- Scan all embedded images and attachments
  +-- Apply QR code detection algorithm
  +-- Check for ASCII/text-rendered QR codes
  +-- Scan PDF attachments for embedded QR codes
  |
  v
[QR code detected?]
  +-- NO --> Continue standard delivery
  +-- YES --> Extract encoded URL
  |
  v
[URL reputation and analysis]
  +-- Check URL against threat intelligence feeds
  +-- Check domain age and registration data
  +-- Submit to sandbox for real-time analysis
  +-- Check for credential harvesting indicators
  |
  v
[Decision]
  +-- MALICIOUS URL: Block email, alert SOC
  +-- SUSPICIOUS URL: Quarantine, add warning banner
  +-- UNKNOWN URL: Tag email with QR warning banner
  +-- CLEAN URL: Deliver with informational banner
```

## Workflow 2: Quishing Incident Response

```
User reports QR code phishing email
  |
  v
[Triage (15 minutes)]
  +-- Extract QR code and decode URL
  +-- Check if URL is active credential harvester
  +-- Search mailboxes for same email to other recipients
  |
  v
[Containment]
  +-- Block sender domain across email gateway
  +-- Retract email from all recipient inboxes
  +-- Block decoded URL at web proxy/firewall
  +-- If user scanned: check for credential compromise
  |
  v
[Investigation]
  +-- Did any user submit credentials on phishing page?
  +-- Check authentication logs for compromised accounts
  +-- If credentials entered: force password reset + revoke sessions
  +-- Review phishing page infrastructure
  |
  v
[Recovery and prevention]
  +-- Add QR URL pattern to detection rules
  +-- Update security awareness training
  +-- Send targeted alert to affected users
  +-- Document IOCs for threat intelligence sharing
```

## Workflow 3: Mobile QR Scanning Protection

```
User scans QR code with mobile device
  |
  v
[Mobile threat defense intercepts]
  +-- Decode QR destination URL
  +-- Check against mobile threat intelligence
  |
  v
[URL assessment]
  +-- KNOWN MALICIOUS: Block and alert user
  +-- SUSPICIOUS: Display warning, require confirmation
  +-- CREDENTIAL PAGE: Extra warning about entering passwords
  +-- CLEAN: Allow access
  |
  v
[If user proceeds to suspicious site]
  +-- Route through secure browser/VPN
  +-- Monitor for credential submission
  +-- Log URL and user action for SOC review
```

===== END FILE =====


===== FILE: skills/detecting-qr-code-phishing-with-email-security/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for detecting QR code phishing (quishing) in email attachments and bodies."""

import argparse
import email
import json
import os
import re
from datetime import datetime, timezone
from email import policy
from urllib.parse import urlparse

try:
    from PIL import Image
    from pyzbar.pyzbar import decode as qr_decode
    HAS_QR = True
except ImportError:
    HAS_QR = False

try:
    HAS_REQUESTS = True
except ImportError:
    HAS_REQUESTS = False


SUSPICIOUS_TLDS = {
    ".xyz", ".top", ".club", ".work", ".buzz", ".tk", ".ml", ".ga", ".cf",
    ".gq", ".info", ".online", ".site", ".icu",
}

PHISHING_KEYWORDS = [
    "verify", "account", "suspended", "confirm", "urgent", "expire",
    "password", "login", "credential", "security", "update", "click",
    "immediate", "unauthorized", "invoice",
]


def extract_images_from_eml(eml_path):
    """Extract image attachments and inline images from an .eml file."""
    images = []
    with open(eml_path, "rb") as f:
        msg = email.message_from_binary_file(f, policy=policy.default)
    for part in msg.walk():
        content_type = part.get_content_type()
        if content_type.startswith("image/"):
            payload = part.get_payload(decode=True)
            if payload:
                ext = content_type.split("/")[1].split(";")[0]
                fname = part.get_filename() or f"inline_image.{ext}"
                images.append({"filename": fname, "data": payload, "type": content_type})
    return images, msg


def decode_qr_from_bytes(image_data):
    """Decode QR codes from raw image bytes."""
    if not HAS_QR:
        return []
    import io
    img = Image.open(io.BytesIO(image_data))
    results = qr_decode(img)
    return [r.data.decode("utf-8", errors="replace") for r in results]


def analyze_url(url):
    """Score a URL for phishing risk indicators."""
    indicators = []
    parsed = urlparse(url)
    domain = parsed.netloc.lower()

    for tld in SUSPICIOUS_TLDS:
        if domain.endswith(tld):
            indicators.append(f"Suspicious TLD: {tld}")
            break

    if re.search(r"\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}", domain):
        indicators.append("URL uses IP address instead of domain")

    if len(domain) > 40:
        indicators.append(f"Unusually long domain: {len(domain)} chars")

    if domain.count(".") > 3:
        indicators.append(f"Many subdomains: {domain.count('.')} dots")

    if parsed.scheme == "http":
        indicators.append("Uses HTTP instead of HTTPS")

    path = parsed.path + (parsed.query or "")
    for kw in PHISHING_KEYWORDS:
        if kw in path.lower():
            indicators.append(f"Phishing keyword in URL path: '{kw}'")
            break

    return {
        "url": url,
        "domain": domain,
        "indicators": indicators,
        "risk_score": min(len(indicators) * 25, 100),
    }


def analyze_email(eml_path):
    """Full QR phishing analysis of an email file."""
    results = {
        "file": eml_path,
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "images_found": 0,
        "qr_codes_found": 0,
        "urls_extracted": [],
        "phishing_indicators": [],
        "risk_level": "LOW",
    }

    images, msg = extract_images_from_eml(eml_path)
    results["images_found"] = len(images)
    results["subject"] = msg.get("Subject", "")
    results["from"] = msg.get("From", "")

    subject_lower = results["subject"].lower()
    for kw in PHISHING_KEYWORDS:
        if kw in subject_lower:
            results["phishing_indicators"].append(f"Phishing keyword in subject: '{kw}'")

    all_urls = []
    for img_info in images:
        decoded = decode_qr_from_bytes(img_info["data"])
        for url in decoded:
            if url.startswith(("http://", "https://")):
                analysis = analyze_url(url)
                all_urls.append(analysis)

    results["qr_codes_found"] = len(all_urls)
    results["urls_extracted"] = all_urls

    max_risk = max((u["risk_score"] for u in all_urls), default=0)
    if max_risk >= 75:
        results["risk_level"] = "CRITICAL"
    elif max_risk >= 50:
        results["risk_level"] = "HIGH"
    elif max_risk >= 25:
        results["risk_level"] = "MEDIUM"

    return results


def scan_directory(dir_path):
    """Scan a directory for .eml files and analyze each."""
    all_results = []
    for root, _, files in os.walk(dir_path):
        for fname in files:
            if fname.lower().endswith(".eml"):
                fpath = os.path.join(root, fname)
                result = analyze_email(fpath)
                all_results.append(result)
    return all_results


def main():
    parser = argparse.ArgumentParser(
        description="Detect QR code phishing (quishing) in emails"
    )
    parser.add_argument("input", help="Path to .eml file or directory of .eml files")
    parser.add_argument("--output", "-o", help="Output JSON report path")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    print("[*] QR Code Phishing Detection Agent")
    print(f"[*] QR decoding available: {HAS_QR}")

    if os.path.isdir(args.input):
        results = scan_directory(args.input)
    else:
        results = [analyze_email(args.input)]

    report = {
        "scan_time": datetime.now(timezone.utc).isoformat(),
        "files_scanned": len(results),
        "qr_phishing_detected": sum(1 for r in results if r["risk_level"] in ("HIGH", "CRITICAL")),
        "results": results,
    }

    if args.verbose:
        for r in results:
            print(f"\n  File: {r['file']}")
            print(f"  Subject: {r.get('subject', 'N/A')}")
            print(f"  Images: {r['images_found']}, QR codes: {r['qr_codes_found']}")
            print(f"  Risk: {r['risk_level']}")

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"[*] Report saved to {args.output}")
    else:
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-qr-code-phishing-with-email-security/scripts/process.py =====
#!/usr/bin/env python3
"""
QR Code Phishing (Quishing) Detection Engine

Detects QR codes in images and email content, extracts encoded URLs,
and checks them against known phishing indicators.

Usage:
    python process.py scan-image --image qr_image.png
    python process.py scan-email --eml-file message.eml
    python process.py check-url --url "https://example.com/login"
"""

import argparse
import json
import re
import sys
import base64
from dataclasses import dataclass, field, asdict
from urllib.parse import urlparse

try:
    from PIL import Image
    HAS_PIL = True
except ImportError:
    HAS_PIL = False

try:
    from pyzbar.pyzbar import decode as qr_decode
    HAS_PYZBAR = True
except ImportError:
    HAS_PYZBAR = False


@dataclass
class QRCodeFinding:
    """A detected QR code and its analysis."""
    source: str = ""
    decoded_url: str = ""
    domain: str = ""
    is_suspicious: bool = False
    risk_score: int = 0
    indicators: list = field(default_factory=list)


@dataclass
class QuishingAnalysis:
    """Complete quishing analysis result."""
    source_file: str = ""
    qr_codes_found: int = 0
    findings: list = field(default_factory=list)
    email_indicators: list = field(default_factory=list)
    overall_risk: str = "low"
    recommended_action: str = ""


# Suspicious URL patterns for credential phishing
SUSPICIOUS_URL_PATTERNS = [
    r'login|signin|sign-in|log-in',
    r'verify|verification|validate',
    r'account|password|credential',
    r'microsoft|office365|outlook|sharepoint',
    r'google|gmail|workspace',
    r'secure|security|auth',
    r'update|confirm|suspend',
    r'\.tk$|\.ml$|\.ga$|\.cf$|\.gq$',
    r'bit\.ly|tinyurl|t\.co|is\.gd|cutt\.ly',
]

# Known phishing infrastructure patterns
PHISHING_INFRA_PATTERNS = [
    r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}',  # IP address URLs
    r'[a-z0-9]{20,}\.web\.app',  # Firebase hosting abuse
    r'[a-z0-9]{20,}\.netlify\.app',  # Netlify abuse
    r'[a-z0-9-]+\.glitch\.me',  # Glitch abuse
    r'[a-z0-9-]+\.workers\.dev',  # Cloudflare workers abuse
]

# Common quishing email indicators
QUISHING_EMAIL_PATTERNS = [
    r'scan\s+(this|the)\s+qr\s+code',
    r'scan\s+to\s+(verify|authenticate|confirm|access)',
    r'multi.?factor\s+authentication',
    r'mfa\s+(setup|enrollment|reset|update)',
    r'voicemail\s+(notification|message)',
    r'document\s+(sign|signing|review)',
    r'security\s+update\s+required',
    r'action\s+required',
]


def analyze_url(url: str) -> QRCodeFinding:
    """Analyze a URL extracted from a QR code for phishing indicators."""
    finding = QRCodeFinding(decoded_url=url)

    try:
        parsed = urlparse(url)
        finding.domain = parsed.netloc
    except Exception:
        finding.indicators.append("Could not parse URL")
        finding.is_suspicious = True
        finding.risk_score = 50
        return finding

    score = 0

    # Check suspicious URL patterns
    url_lower = url.lower()
    for pattern in SUSPICIOUS_URL_PATTERNS:
        if re.search(pattern, url_lower):
            finding.indicators.append(f"Suspicious URL pattern: {pattern}")
            score += 15

    # Check phishing infrastructure patterns
    for pattern in PHISHING_INFRA_PATTERNS:
        if re.search(pattern, url_lower):
            finding.indicators.append(f"Known phishing infrastructure pattern: {pattern}")
            score += 25

    # Check for URL shorteners (hiding true destination)
    shorteners = ['bit.ly', 'tinyurl.com', 't.co', 'is.gd', 'cutt.ly',
                  'rebrand.ly', 'ow.ly', 'buff.ly']
    if finding.domain in shorteners:
        finding.indicators.append(f"URL shortener detected: {finding.domain}")
        score += 20

    # Check for IP address URL
    if re.match(r'^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}', finding.domain):
        finding.indicators.append("URL uses IP address instead of domain name")
        score += 30

    # Check for excessive subdomains (common in phishing)
    subdomain_count = finding.domain.count('.')
    if subdomain_count > 3:
        finding.indicators.append(f"Excessive subdomains ({subdomain_count})")
        score += 15

    # Check for homoglyph characters in domain
    non_ascii = [c for c in finding.domain if ord(c) > 127]
    if non_ascii:
        finding.indicators.append("Non-ASCII characters in domain (possible homoglyph)")
        score += 25

    # Check protocol
    if not url.startswith('https://'):
        finding.indicators.append("URL does not use HTTPS")
        score += 10

    finding.risk_score = min(score, 100)
    finding.is_suspicious = score >= 30

    return finding


def scan_image_for_qr(image_path: str) -> list:
    """Scan an image file for QR codes and extract URLs."""
    findings = []

    if not HAS_PIL:
        print("Pillow not installed. Install with: pip install Pillow", file=sys.stderr)
        return findings
    if not HAS_PYZBAR:
        print("pyzbar not installed. Install with: pip install pyzbar", file=sys.stderr)
        return findings

    try:
        img = Image.open(image_path)
        decoded_objects = qr_decode(img)

        for obj in decoded_objects:
            data = obj.data.decode('utf-8', errors='replace')
            if data.startswith(('http://', 'https://', 'www.')):
                url = data if data.startswith('http') else f'https://{data}'
                finding = analyze_url(url)
                finding.source = image_path
                findings.append(finding)
            else:
                finding = QRCodeFinding(
                    source=image_path,
                    decoded_url=data,
                    indicators=["QR code contains non-URL data"],
                    risk_score=10
                )
                findings.append(finding)

    except Exception as e:
        print(f"Error scanning image: {e}", file=sys.stderr)

    return findings


def scan_email_content(eml_content: str) -> QuishingAnalysis:
    """Analyze email content for quishing indicators."""
    analysis = QuishingAnalysis()
    body_lower = eml_content.lower()

    # Check for quishing email patterns
    for pattern in QUISHING_EMAIL_PATTERNS:
        if re.search(pattern, body_lower):
            analysis.email_indicators.append(f"Quishing language pattern: {pattern}")

    # Check for image-heavy email with minimal text
    text_content = re.sub(r'<[^>]+>', '', eml_content)
    text_content = re.sub(r'\s+', ' ', text_content).strip()
    has_images = bool(re.search(r'<img|Content-Type:\s*image/', eml_content, re.IGNORECASE))
    text_words = len(text_content.split())

    if has_images and text_words < 50:
        analysis.email_indicators.append(
            "Image-heavy email with minimal text (common quishing pattern)"
        )

    # Check for base64-encoded images
    b64_images = re.findall(
        r'Content-Type:\s*image/\w+.*?Content-Transfer-Encoding:\s*base64\s*\n\n([\w+/=\n]+)',
        eml_content, re.DOTALL | re.IGNORECASE
    )

    if b64_images and HAS_PIL and HAS_PYZBAR:
        for i, b64_data in enumerate(b64_images):
            try:
                clean_data = b64_data.replace('\n', '')
                img_bytes = base64.b64decode(clean_data)
                import io
                img = Image.open(io.BytesIO(img_bytes))
                decoded = qr_decode(img)
                for obj in decoded:
                    data = obj.data.decode('utf-8', errors='replace')
                    if data.startswith(('http://', 'https://')):
                        finding = analyze_url(data)
                        finding.source = f"embedded_image_{i}"
                        analysis.findings.append(finding)
                        analysis.qr_codes_found += 1
            except Exception:
                continue

    # Calculate overall risk
    indicator_count = len(analysis.email_indicators)
    has_suspicious_qr = any(f.is_suspicious for f in analysis.findings)

    if has_suspicious_qr and indicator_count >= 2:
        analysis.overall_risk = "critical"
        analysis.recommended_action = "BLOCK and alert SOC"
    elif has_suspicious_qr or indicator_count >= 3:
        analysis.overall_risk = "high"
        analysis.recommended_action = "QUARANTINE for manual review"
    elif indicator_count >= 1:
        analysis.overall_risk = "medium"
        analysis.recommended_action = "TAG with QR phishing warning banner"
    else:
        analysis.overall_risk = "low"
        analysis.recommended_action = "DELIVER normally"

    return analysis


def format_report(analysis: QuishingAnalysis) -> str:
    """Format analysis as readable report."""
    lines = []
    lines.append("=" * 60)
    lines.append("  QR CODE PHISHING (QUISHING) ANALYSIS REPORT")
    lines.append("=" * 60)
    lines.append(f"  QR Codes Found: {analysis.qr_codes_found}")
    lines.append(f"  Overall Risk: {analysis.overall_risk.upper()}")
    lines.append(f"  Action: {analysis.recommended_action}")

    if analysis.email_indicators:
        lines.append(f"\n  [EMAIL INDICATORS] ({len(analysis.email_indicators)})")
        for i, ind in enumerate(analysis.email_indicators, 1):
            lines.append(f"    {i}. {ind}")

    if analysis.findings:
        lines.append(f"\n  [QR CODE FINDINGS] ({len(analysis.findings)})")
        for i, finding in enumerate(analysis.findings, 1):
            lines.append(f"    {i}. URL: {finding.decoded_url}")
            lines.append(f"       Domain: {finding.domain}")
            lines.append(f"       Risk Score: {finding.risk_score}/100")
            lines.append(f"       Suspicious: {'YES' if finding.is_suspicious else 'No'}")
            for ind in finding.indicators:
                lines.append(f"       - {ind}")

    lines.append("=" * 60)
    return "\n".join(lines)


def main():
    parser = argparse.ArgumentParser(description="QR Code Phishing Detection")
    subparsers = parser.add_subparsers(dest="command")

    img_parser = subparsers.add_parser("scan-image", help="Scan image for QR codes")
    img_parser.add_argument("--image", required=True)

    eml_parser = subparsers.add_parser("scan-email", help="Scan email for quishing")
    eml_parser.add_argument("--eml-file", required=True)

    url_parser = subparsers.add_parser("check-url", help="Check URL from QR code")
    url_parser.add_argument("--url", required=True)

    parser.add_argument("--json", action="store_true")
    args = parser.parse_args()

    if args.command == "scan-image":
        findings = scan_image_for_qr(args.image)
        analysis = QuishingAnalysis(
            source_file=args.image,
            qr_codes_found=len(findings),
            findings=findings
        )
        if args.json:
            print(json.dumps(asdict(analysis), indent=2))
        else:
            print(format_report(analysis))

    elif args.command == "scan-email":
        with open(args.eml_file, 'r', errors='replace') as f:
            content = f.read()
        analysis = scan_email_content(content)
        analysis.source_file = args.eml_file
        if args.json:
            print(json.dumps(asdict(analysis), indent=2))
        else:
            print(format_report(analysis))

    elif args.command == "check-url":
        finding = analyze_url(args.url)
        if args.json:
            print(json.dumps(asdict(finding), indent=2))
        else:
            print(f"URL: {finding.decoded_url}")
            print(f"Domain: {finding.domain}")
            print(f"Risk Score: {finding.risk_score}/100")
            print(f"Suspicious: {'YES' if finding.is_suspicious else 'No'}")
            for ind in finding.indicators:
                print(f"  - {ind}")

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-spearphishing-with-email-gateway/SKILL.md =====
---
name: detecting-spearphishing-with-email-gateway
description: Detect and block spearphishing emails that use personalized, researched
  content to evade generic spam filters, by configuring email security gateway
  (SEG) impersonation protection, URL rewriting, attachment sandboxing, and
  custom detection rules in Microsoft Defender for Office 365, Proofpoint,
  Mimecast, or Barracuda. Use when hardening a mail gateway against targeted
  phishing or investigating a suspected spearphishing campaign.
domain: cybersecurity
subdomain: phishing-defense
tags:
- phishing
- email-security
- social-engineering
- dmarc
- awareness
- spearphishing
- email-gateway
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566.001
- T1566.002
- T1204.001
- T1204.002
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - initial-access
  - stealth
  - resource-development
  techniques:
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: T1598
    name: Phishing for Information
    tactic: reconnaissance
    source: attack
  - id: T1672
    name: Email Spoofing
    tactic: stealth
    source: attack
  - id: F1032
    name: Impersonate Official
    tactic: initial-access
    source: f3
  - id: F1031
    name: Impersonate Account Holder
    tactic: initial-access
    source: f3
  - id: F1020.002
    name: 'Create Fake Materials: Fake Website'
    tactic: resource-development
    source: f3
---
# Detecting Spearphishing with Email Gateway

## Overview
Spearphishing targets specific individuals using personalized, researched content that bypasses generic spam filters. Email security gateways (SEGs) like Microsoft Defender for Office 365, Proofpoint, Mimecast, and Barracuda provide advanced detection capabilities including behavioral analysis, URL detonation, attachment sandboxing, and impersonation detection. This skill covers configuring these gateways to detect and block targeted phishing attacks.


## When to Use

- When investigating security incidents that require detecting spearphishing with email gateway
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites
- Access to email security gateway admin console
- Understanding of email flow architecture (MX records, transport rules)
- Familiarity with SPF/DKIM/DMARC authentication
- Knowledge of common spearphishing techniques and pretexts

## Key Concepts

### Spearphishing Characteristics
- **Targeted recipients**: Specific individuals, often executives or finance staff
- **Researched pretexts**: References to real projects, colleagues, or events
- **Impersonation**: Spoofs trusted senders (CEO, vendor, partner)
- **Low volume**: Few emails to avoid pattern-based detection
- **Urgent tone**: Creates pressure to act quickly

### Gateway Detection Layers
1. **Reputation filtering**: IP/domain/URL reputation scoring
2. **Authentication checks**: SPF, DKIM, DMARC validation
3. **Content analysis**: NLP-based analysis of email body
4. **Impersonation detection**: Display name and domain similarity matching
5. **URL analysis**: Real-time URL detonation and redirect following
6. **Attachment sandboxing**: Behavioral analysis of attachments in isolated environments
7. **Behavioral analytics**: Anomaly detection in communication patterns

## Workflow

### Step 1: Configure Impersonation Protection
```
Microsoft Defender for Office 365:
  Security > Anti-phishing policies > Impersonation settings
  - Enable user impersonation protection for VIPs
  - Enable domain impersonation protection
  - Add protected users (CEO, CFO, HR Director)
  - Set action: Quarantine message

Proofpoint:
  Email Protection > Impostor Classifier
  - Enable display name spoofing detection
  - Configure lookalike domain detection
  - Set Impostor threshold sensitivity
```

### Step 2: Configure URL Protection
- Enable Safe Links / URL rewriting
- Enable time-of-click URL detonation
- Block newly registered domains (< 30 days)
- Enable URL redirect chain following

### Step 3: Configure Attachment Sandboxing
- Enable Safe Attachments / attachment sandboxing
- Configure dynamic delivery (deliver body, hold attachments)
- Set sandbox detonation timeout to 60+ seconds
- Block macro-enabled Office documents from external senders

### Step 4: Create Custom Detection Rules
Use the `scripts/process.py` to analyze email gateway logs, identify spearphishing patterns, and generate custom detection rules.

### Step 5: Configure Alert and Response Actions
- Real-time alerts for impersonation attempts
- Automatic quarantine for high-confidence detections
- User notification with safety tips
- Integration with SIEM for correlation

## Tools & Resources
- **Microsoft Defender for Office 365**: https://security.microsoft.com
- **Proofpoint Email Protection**: https://www.proofpoint.com/us/products/email-security
- **Mimecast Email Security**: https://www.mimecast.com/products/email-security/
- **Barracuda Email Protection**: https://www.barracuda.com/products/email-protection

## Validation
- Impersonation protection correctly identifies spoofed VIP display names
- URL detonation catches malicious links in test phishing emails
- Attachment sandboxing detects weaponized documents
- Custom rules trigger on known spearphishing patterns
- SIEM integration receives gateway alerts

===== END FILE =====


===== FILE: skills/detecting-spearphishing-with-email-gateway/LICENSE =====

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


===== FILE: skills/detecting-spearphishing-with-email-gateway/assets/template.md =====
# Email Gateway Spearphishing Detection Configuration Template

## Gateway Information
- **Product**: [Microsoft Defender for O365 / Proofpoint / Mimecast / Barracuda]
- **Version**: []
- **Configuration Date**: [YYYY-MM-DD]
- **Configured By**: []

## VIP Protection List
| Name | Title | Email | Risk Level |
|---|---|---|---|
| | CEO | | Critical |
| | CFO | | Critical |
| | CTO | | Critical |
| | VP Finance | | High |
| | HR Director | | High |

## Impersonation Detection Rules
| Rule | Trigger | Action | Severity |
|---|---|---|---|
| VIP display name match | External email with VIP name | Quarantine + Alert | Critical |
| Lookalike domain | Levenshtein distance <= 2 | Quarantine | High |
| First-time sender to VIP | No prior communication | Tag warning | Medium |
| Reply-to mismatch | Reply-to differs from From | Tag + Log | Medium |

## URL Protection Settings
| Setting | Value |
|---|---|
| URL rewriting enabled | Yes |
| Time-of-click verification | Yes |
| Block new domains (< days) | 30 |
| Follow redirects | Yes (max 5 hops) |
| Detonate suspicious URLs | Yes |

## Attachment Protection Settings
| Setting | Value |
|---|---|
| Sandbox detonation | Enabled |
| Dynamic delivery | Enabled |
| Block macros from external | Yes |
| Block executable types | .exe, .scr, .bat, .cmd, .ps1, .vbs, .js |

## Alert Configuration
| Event | Alert Method | Recipients |
|---|---|---|
| VIP impersonation detected | Email + SIEM | SOC team |
| Credential harvest URL blocked | SIEM | SOC team |
| Malicious attachment blocked | Email + SIEM | SOC team |
| DMARC failure from partner domain | Email | Email admin |

## Quarterly Review Checklist
- [ ] Update VIP protection list
- [ ] Review false positive rates
- [ ] Analyze user-reported phishing misses
- [ ] Update domain allow/block lists
- [ ] Test detection with simulated phishing
- [ ] Review and update custom rules
- [ ] Verify SIEM integration working

===== END FILE =====


===== FILE: skills/detecting-spearphishing-with-email-gateway/references/api-reference.md =====
# API Reference: Spearphishing Detection via Email Gateway

## Python email Module

### Parse EML file
```python
import email
from email import policy

with open("message.eml", "rb") as f:
    msg = email.message_from_binary_file(f, policy=policy.default)
```

### Security Headers
| Header | Purpose |
|--------|---------|
| `Received-SPF` | SPF check result |
| `Authentication-Results` | SPF, DKIM, DMARC combined |
| `DKIM-Signature` | DKIM signing info |
| `ARC-Authentication-Results` | ARC chain results |
| `X-Mailer` | Client used to send |
| `Return-Path` | Envelope sender |

### Authentication-Results Values
```
Authentication-Results: mx.google.com;
    dkim=pass header.d=example.com;
    spf=pass smtp.mailfrom=example.com;
    dmarc=pass
```

## SPF Record Lookup
```bash
dig TXT example.com | grep "v=spf1"
# v=spf1 include:_spf.google.com ~all
```

### SPF Results
| Result | Meaning |
|--------|---------|
| `pass` | Authorized sender |
| `fail` | Unauthorized (reject) |
| `softfail` | Unauthorized (accept with mark) |
| `neutral` | No assertion |
| `none` | No SPF record |

## DKIM Verification
```bash
opendkim-testkey -d example.com -s selector -vvv
```

## DMARC Record
```bash
dig TXT _dmarc.example.com
# v=DMARC1; p=reject; rua=mailto:dmarc@example.com
```

## Microsoft Defender for Office 365 API

### Get email threat assessment
```http
POST https://graph.microsoft.com/v1.0/informationProtection/threatAssessmentRequests
Authorization: Bearer {token}

{
  "contentType": "mail",
  "expectedAssessment": "block",
  "category": "phishing",
  "mailInfo": {
    "internetMessageId": "<message-id>"
  }
}
```

## Proofpoint TAP API

### Get blocked messages
```http
GET https://tap-api-v2.proofpoint.com/v2/siem/messages/blocked
    ?sinceSeconds=3600
Authorization: Basic {base64_credentials}
```

### Response Fields
| Field | Description |
|-------|-------------|
| `spamScore` | Spam confidence (0-100) |
| `phishScore` | Phishing confidence (0-100) |
| `threatsInfoMap` | Threat details array |
| `fromAddress` | Envelope sender |

## Mimecast API — URL Protection

### Decode Mimecast URL
```http
POST https://api.mimecast.com/api/ttp/url/decode-url
Authorization: MC {access-key}:{secret-key}

{
  "data": [{"url": "https://protect.mimecast.com/..."}]
}
```

===== END FILE =====


===== FILE: skills/detecting-spearphishing-with-email-gateway/references/standards.md =====
# Standards & References: Detecting Spearphishing with Email Gateway

## MITRE ATT&CK References
- **T1566.001**: Phishing: Spearphishing Attachment
- **T1566.002**: Phishing: Spearphishing Link
- **T1566.003**: Phishing: Spearphishing via Service
- **T1598.002**: Phishing for Information: Spearphishing Attachment
- **T1598.003**: Phishing for Information: Spearphishing Link
- **T1534**: Internal Spearphishing

## NIST Guidelines
- **NIST SP 800-177 Rev.1**: Trustworthy Email
- **NIST SP 800-53 Rev.5**: SI-8 Spam Protection, SI-3 Malicious Code Protection
- **NIST CSF**: PR.AT (Awareness & Training), DE.CM (Security Continuous Monitoring)

## CIS Controls v8
- **CIS Control 9**: Email and Web Browser Protections
  - 9.1: Ensure only approved browsers and email clients are used
  - 9.2: Use DNS filtering services
  - 9.3: Maintain and enforce network-based URL filters
  - 9.6: Block unnecessary file types
  - 9.7: Deploy and maintain email server anti-malware protections

## Email Gateway Feature Matrix

| Feature | Microsoft Defender | Proofpoint | Mimecast | Barracuda |
|---|---|---|---|---|
| Impersonation detection | Anti-phishing policy | Impostor Classifier | Brand Exploit Protect | Impersonation Protection |
| URL detonation | Safe Links | URL Defense | URL Protect | Link Protection |
| Attachment sandbox | Safe Attachments | Targeted Attack Protection | Attachment Protect | Advanced Threat Protection |
| DMARC enforcement | Built-in | Built-in | DMARC Analyzer | Built-in |
| AI/ML detection | Yes (multiple models) | NexusAI | Yes | Yes |
| User reporting | Report Message add-in | PhishAlarm | Built-in | Phishline |
| SIEM integration | Microsoft Sentinel | Splunk, QRadar | Splunk, Sentinel | Various |
| Auto-remediation (ZAP) | Yes | CLEAR | Yes | Yes |

## Detection Indicators for Spearphishing

| Indicator | Weight | Description |
|---|---|---|
| Display name spoofing VIP | High | From name matches protected user but different email |
| Lookalike domain | High | Domain differs by 1-2 characters from legitimate |
| First-time sender to VIP | Medium | No prior communication history |
| Urgency keywords | Medium | "urgent", "immediately", "wire transfer", "confidential" |
| Reply-to mismatch | High | Reply-to differs from From address |
| External sender with internal branding | High | Email mimics internal templates |
| Newly registered domain | High | Sending domain < 30 days old |
| Authentication failure | High | SPF/DKIM/DMARC fail |

===== END FILE =====


===== FILE: skills/detecting-spearphishing-with-email-gateway/references/workflows.md =====
# Workflows: Detecting Spearphishing with Email Gateway

## Workflow 1: Multi-Layer Detection Pipeline

```
Inbound Email Arrives at Gateway
  |
  v
[Layer 1: Connection Filtering]
  +-- Check sender IP reputation
  +-- Check RBL/DNSBL blacklists
  +-- Rate limiting / throttling
  |
  v
[Layer 2: Authentication]
  +-- Verify SPF alignment
  +-- Verify DKIM signature
  +-- Evaluate DMARC policy
  +-- Check ARC headers (forwarded mail)
  |
  v
[Layer 3: Impersonation Detection]
  +-- Compare display name against VIP list
  +-- Check domain similarity (Levenshtein distance)
  +-- Evaluate sender reputation
  +-- First-time sender analysis
  |
  v
[Layer 4: Content Analysis]
  +-- NLP analysis for urgency/social engineering
  +-- Business context anomaly detection
  +-- Keyword pattern matching
  +-- Language analysis
  |
  v
[Layer 5: URL Analysis]
  +-- URL reputation check
  +-- Domain age verification
  +-- Real-time URL detonation
  +-- Redirect chain following
  +-- Visual similarity to legitimate sites
  |
  v
[Layer 6: Attachment Analysis]
  +-- File type validation
  +-- Sandbox detonation
  +-- Macro analysis
  +-- Embedded object detection
  |
  v
[Decision Engine]
  +-- Aggregate scores from all layers
  +-- Apply organizational policy
  |
  +-- DELIVER: Low risk
  +-- TAG: Add warning banner
  +-- QUARANTINE: Moderate risk
  +-- BLOCK: High risk, drop message
```

## Workflow 2: VIP Impersonation Detection

```
Email arrives with From display name matching VIP list
  |
  v
[Check: Is sending domain authorized for this VIP?]
  |
  +-- YES: Check DKIM/SPF --> If pass, deliver normally
  |
  +-- NO: Impersonation suspected
       |
       v
  [Calculate domain similarity score]
       |
       +-- Exact match (different email): CRITICAL - Block
       +-- Lookalike domain (1-2 char diff): HIGH - Quarantine
       +-- Similar but different: MEDIUM - Tag with warning
       |
       v
  [Additional checks]
       +-- Has this sender emailed before?
       +-- Is the sending infrastructure legitimate?
       +-- Does email content match typical VIP communication?
       |
       v
  [Action: Quarantine + Alert SOC + Notify recipient manager]
```

## Workflow 3: Spearphishing Response

```
Gateway detects potential spearphishing
  |
  v
[Automated Response]
  +-- Quarantine message
  +-- Generate alert in SIEM
  +-- Extract IOCs (sender, domain, URLs, hashes)
  |
  v
[SOC Analyst Review]
  +-- Review quarantined message
  +-- Analyze full headers
  +-- Investigate sending infrastructure
  +-- Check if other users received similar emails
  |
  +-- FALSE POSITIVE
  |     +-- Release from quarantine
  |     +-- Whitelist if legitimate
  |     +-- Update detection rules
  |
  +-- CONFIRMED SPEARPHISHING
        +-- Block sender domain organization-wide
        +-- Search mailboxes for similar messages (retroactive)
        +-- Auto-purge any delivered copies (ZAP)
        +-- Notify targeted users
        +-- Submit IOCs to threat intelligence
        +-- Check for any successful credential compromise
        +-- Update VIP protection list if needed
```

## Workflow 4: Gateway Tuning Cycle

```
Monthly Review
  |
  +-- Pull detection statistics from gateway
  +-- Analyze false positive rate
  +-- Analyze false negative rate (user-reported misses)
  +-- Review quarantine volumes
  |
  v
[Identify gaps]
  +-- New impersonation patterns?
  +-- New sending domains to whitelist/blacklist?
  +-- Policy thresholds too aggressive/permissive?
  |
  v
[Adjust configuration]
  +-- Update VIP protection list (new hires, departures)
  +-- Tune sensitivity thresholds
  +-- Add custom transport rules
  +-- Update URL/domain blocklists
  |
  v
[Validate changes]
  +-- Send test phishing emails
  +-- Verify legitimate mail still flows
  +-- Document changes
```

===== END FILE =====


===== FILE: skills/detecting-spearphishing-with-email-gateway/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for detecting spearphishing emails using email gateway log analysis."""

import argparse
import email
import json
import os
import re
from datetime import datetime, timezone
from email import policy


EXECUTIVE_TITLES = [
    "ceo", "cfo", "cto", "cio", "coo", "president", "director",
    "vp", "vice president", "managing director", "partner",
]
URGENCY_KEYWORDS = [
    "urgent", "immediate", "asap", "right away", "time sensitive",
    "confidential", "do not share", "wire transfer", "bank account",
    "update payment", "invoice attached", "past due",
]
SUSPICIOUS_EXTENSIONS = {
    ".exe", ".scr", ".bat", ".cmd", ".ps1", ".vbs", ".js",
    ".hta", ".lnk", ".iso", ".img", ".dll", ".msi",
}


def parse_email_headers(eml_path):
    """Extract security-relevant headers from an email."""
    with open(eml_path, "rb") as f:
        msg = email.message_from_binary_file(f, policy=policy.default)
    headers = {
        "from": msg.get("From", ""),
        "to": msg.get("To", ""),
        "subject": msg.get("Subject", ""),
        "reply_to": msg.get("Reply-To", ""),
        "return_path": msg.get("Return-Path", ""),
        "received_spf": msg.get("Received-SPF", ""),
        "dkim_signature": msg.get("DKIM-Signature", ""),
        "authentication_results": msg.get("Authentication-Results", ""),
        "x_mailer": msg.get("X-Mailer", ""),
        "message_id": msg.get("Message-ID", ""),
    }
    return headers, msg


def check_authentication(headers):
    """Verify SPF, DKIM, DMARC authentication results."""
    issues = []
    auth_results = headers.get("authentication_results", "").lower()
    if "spf=fail" in auth_results or "spf=softfail" in auth_results:
        issues.append("SPF validation failed")
    if "dkim=fail" in auth_results:
        issues.append("DKIM validation failed")
    if "dmarc=fail" in auth_results:
        issues.append("DMARC validation failed")
    if not headers.get("dkim_signature"):
        issues.append("No DKIM signature present")
    from_domain = re.search(r"@([\w.-]+)", headers.get("from", ""))
    reply_domain = re.search(r"@([\w.-]+)", headers.get("reply_to", ""))
    if from_domain and reply_domain and from_domain.group(1) != reply_domain.group(1):
        issues.append(f"Reply-To domain mismatch: {reply_domain.group(1)} vs {from_domain.group(1)}")
    return issues


def check_content_indicators(msg):
    """Analyze email body for spearphishing content indicators."""
    indicators = []
    body = ""
    for part in msg.walk():
        if part.get_content_type() in ("text/plain", "text/html"):
            payload = part.get_payload(decode=True)
            if payload:
                body += payload.decode("utf-8", errors="replace")

    body_lower = body.lower()
    for kw in URGENCY_KEYWORDS:
        if kw in body_lower:
            indicators.append(f"Urgency keyword: '{kw}'")

    urls = re.findall(r'https?://[^\s<>"\']+', body)
    for url in urls[:10]:
        if re.search(r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}', url):
            indicators.append(f"URL with IP address: {url[:80]}")
        if len(url) > 100:
            indicators.append(f"Unusually long URL: {url[:80]}...")

    attachments = []
    for part in msg.walk():
        fname = part.get_filename()
        if fname:
            ext = os.path.splitext(fname)[1].lower()
            attachments.append(fname)
            if ext in SUSPICIOUS_EXTENSIONS:
                indicators.append(f"Suspicious attachment: {fname}")
            if ".." in fname or ext == ".exe.pdf":
                indicators.append(f"Double extension: {fname}")

    return indicators, attachments


def analyze_email(eml_path):
    """Full spearphishing analysis of a single email."""
    headers, msg = parse_email_headers(eml_path)
    auth_issues = check_authentication(headers)
    content_indicators, attachments = check_content_indicators(msg)

    all_indicators = auth_issues + content_indicators
    score = len(all_indicators) * 15
    risk = "CRITICAL" if score >= 75 else "HIGH" if score >= 50 else "MEDIUM" if score >= 25 else "LOW"

    return {
        "file": eml_path,
        "from": headers["from"],
        "to": headers["to"],
        "subject": headers["subject"],
        "auth_issues": auth_issues,
        "content_indicators": content_indicators,
        "attachments": attachments,
        "risk_score": min(score, 100),
        "risk_level": risk,
    }


def main():
    parser = argparse.ArgumentParser(
        description="Detect spearphishing via email gateway analysis"
    )
    parser.add_argument("input", help=".eml file or directory")
    parser.add_argument("--output", "-o", help="Output JSON report")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    print("[*] Spearphishing Detection Agent")
    results = []

    if os.path.isdir(args.input):
        for root, _, files in os.walk(args.input):
            for f in files:
                if f.lower().endswith(".eml"):
                    results.append(analyze_email(os.path.join(root, f)))
    else:
        results.append(analyze_email(args.input))

    flagged = [r for r in results if r["risk_level"] in ("HIGH", "CRITICAL")]
    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "emails_scanned": len(results),
        "spearphishing_detected": len(flagged),
        "results": results,
    }

    print(f"[*] Scanned {len(results)} emails, {len(flagged)} flagged")

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"[*] Report saved to {args.output}")
    else:
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-spearphishing-with-email-gateway/scripts/process.py =====
#!/usr/bin/env python3
"""
Spearphishing Detection Engine for Email Gateway Logs

Analyzes email gateway logs to detect spearphishing patterns including
impersonation, lookalike domains, and behavioral anomalies. Generates
custom detection rules and threat reports.

Usage:
    python process.py analyze --log-file gateway_log.json
    python process.py detect --email-file email.eml
    python process.py rules --output detection_rules.yaml
"""

import argparse
import json
import re
import sys
import hashlib
from datetime import datetime, timezone
from pathlib import Path
from dataclasses import dataclass, field, asdict
from typing import Optional
from collections import defaultdict

try:
    import requests
    HAS_REQUESTS = True
except ImportError:
    HAS_REQUESTS = False


@dataclass
class SpearphishingIndicator:
    """A detected spearphishing indicator."""
    indicator_type: str = ""
    description: str = ""
    severity: str = "medium"
    confidence: float = 0.0
    raw_value: str = ""
    mitre_technique: str = ""


@dataclass
class DomainSimilarity:
    """Domain similarity analysis result."""
    original_domain: str = ""
    suspicious_domain: str = ""
    distance: int = 0
    technique: str = ""
    confidence: float = 0.0


@dataclass
class EmailAnalysis:
    """Complete spearphishing analysis for a single email."""
    message_id: str = ""
    from_address: str = ""
    from_display_name: str = ""
    from_domain: str = ""
    to_address: str = ""
    subject: str = ""
    date: str = ""
    indicators: list = field(default_factory=list)
    risk_score: float = 0.0
    risk_level: str = "low"
    domain_similarities: list = field(default_factory=list)
    is_spearphishing: bool = False


# VIP list for impersonation detection (configure per organization)
DEFAULT_VIP_NAMES = [
    "CEO", "CFO", "CTO", "CISO", "COO",
    "Chief Executive", "Chief Financial", "Chief Technology",
    "President", "Vice President", "Director",
]

# Common urgency keywords in spearphishing
URGENCY_KEYWORDS = [
    r'\burgent\b', r'\bimmediately\b', r'\basap\b', r'\btime.?sensitive\b',
    r'\bwire\s+transfer\b', r'\bbank\s+transfer\b', r'\bgift\s+card\b',
    r'\bconfidential\b', r'\bdo\s+not\s+share\b', r'\bsecret\b',
    r'\bpayment\b', r'\binvoice\b', r'\boverdue\b', r'\bfinal\s+notice\b',
    r'\baccount\s+suspen', r'\bverify\s+your\b', r'\bconfirm\s+your\b',
    r'\bunusual\s+activity\b', r'\bsecurity\s+alert\b',
]

# Legitimate domains for similarity comparison
COMMON_TARGETS = [
    "microsoft.com", "google.com", "apple.com", "amazon.com",
    "paypal.com", "netflix.com", "linkedin.com", "facebook.com",
    "dropbox.com", "docusign.com", "zoom.us", "slack.com",
    "office365.com", "outlook.com", "gmail.com",
]


def levenshtein_distance(s1: str, s2: str) -> int:
    """Calculate Levenshtein edit distance between two strings."""
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


def detect_homograph(domain: str) -> list:
    """Detect IDN homograph attacks in domain names."""
    homograph_map = {
        'a': ['@', 'а', 'ɑ'],   # Cyrillic а, Latin alpha
        'e': ['е', 'ё', 'э'],   # Cyrillic variants
        'o': ['о', '0', 'ο'],   # Cyrillic о, zero, Greek omicron
        'p': ['р', 'ρ'],        # Cyrillic р, Greek rho
        'c': ['с', 'ç'],        # Cyrillic с
        'x': ['х', 'χ'],        # Cyrillic х, Greek chi
        'y': ['у', 'γ'],        # Cyrillic у
        'i': ['і', 'і', '1', 'l'],
    }
    findings = []
    for i, char in enumerate(domain):
        for latin, lookalikes in homograph_map.items():
            if char in lookalikes:
                findings.append({
                    "position": i,
                    "character": char,
                    "looks_like": latin,
                    "type": "homograph"
                })
    return findings


def check_domain_similarity(domain: str, known_domains: list = None) -> list:
    """Check if a domain is similar to known legitimate domains."""
    if known_domains is None:
        known_domains = COMMON_TARGETS

    similarities = []
    domain_base = domain.split(".")[0] if "." in domain else domain

    for known in known_domains:
        known_base = known.split(".")[0]
        dist = levenshtein_distance(domain_base.lower(), known_base.lower())

        if 0 < dist <= 2:
            technique = "typosquatting"
            if len(domain_base) != len(known_base):
                technique = "character_addition" if len(domain_base) > len(known_base) else "character_omission"
            elif dist == 1:
                for i, (a, b) in enumerate(zip(domain_base, known_base)):
                    if a != b:
                        if i > 0 and domain_base[i-1:i+1] == known_base[i:i-2:-1] if i > 0 else False:
                            technique = "transposition"
                        break

            confidence = max(0, 1.0 - dist * 0.3)
            similarities.append(DomainSimilarity(
                original_domain=known,
                suspicious_domain=domain,
                distance=dist,
                technique=technique,
                confidence=confidence
            ))

    # Check for homographs
    homographs = detect_homograph(domain)
    if homographs:
        similarities.append(DomainSimilarity(
            original_domain="(homograph detection)",
            suspicious_domain=domain,
            distance=0,
            technique="homograph",
            confidence=0.9
        ))

    return sorted(similarities, key=lambda x: x.distance)


def detect_urgency(text: str) -> list:
    """Detect urgency patterns in email text."""
    findings = []
    text_lower = text.lower()
    for pattern in URGENCY_KEYWORDS:
        matches = re.findall(pattern, text_lower, re.IGNORECASE)
        if matches:
            findings.append({
                "pattern": pattern,
                "matches": matches,
                "count": len(matches)
            })
    return findings


def detect_impersonation(display_name: str, vip_names: list = None) -> list:
    """Check if display name impersonates a VIP."""
    if vip_names is None:
        vip_names = DEFAULT_VIP_NAMES

    findings = []
    name_lower = display_name.lower()

    for vip in vip_names:
        if vip.lower() in name_lower:
            findings.append({
                "matched_vip": vip,
                "display_name": display_name,
                "confidence": 0.8
            })

    return findings


def analyze_email(headers: dict, body: str = "",
                  vip_names: list = None,
                  known_domains: list = None) -> EmailAnalysis:
    """Analyze a single email for spearphishing indicators."""
    analysis = EmailAnalysis()

    analysis.from_address = headers.get("from", "")
    analysis.from_display_name = headers.get("from_display_name", "")
    analysis.from_domain = headers.get("from_domain", "")
    analysis.to_address = headers.get("to", "")
    analysis.subject = headers.get("subject", "")
    analysis.date = headers.get("date", "")
    analysis.message_id = headers.get("message_id", "")

    # Extract domain from from_address if not provided
    if not analysis.from_domain and analysis.from_address:
        match = re.search(r'@([\w.-]+)', analysis.from_address)
        if match:
            analysis.from_domain = match.group(1).lower()

    # Extract display name if not provided
    if not analysis.from_display_name and analysis.from_address:
        match = re.match(r'"?([^"<]+)"?\s*<', analysis.from_address)
        if match:
            analysis.from_display_name = match.group(1).strip()

    score = 0.0

    # Check 1: Domain similarity
    if analysis.from_domain:
        similarities = check_domain_similarity(analysis.from_domain, known_domains)
        analysis.domain_similarities = similarities
        for sim in similarities:
            if sim.distance <= 1:
                analysis.indicators.append(SpearphishingIndicator(
                    indicator_type="lookalike_domain",
                    description=f"Domain '{sim.suspicious_domain}' is {sim.distance} edit(s) "
                                f"from '{sim.original_domain}' ({sim.technique})",
                    severity="critical" if sim.distance == 1 else "high",
                    confidence=sim.confidence,
                    raw_value=sim.suspicious_domain,
                    mitre_technique="T1566.002"
                ))
                score += 30 * sim.confidence
            elif sim.distance == 2:
                analysis.indicators.append(SpearphishingIndicator(
                    indicator_type="similar_domain",
                    description=f"Domain '{sim.suspicious_domain}' resembles "
                                f"'{sim.original_domain}' (distance={sim.distance})",
                    severity="medium",
                    confidence=sim.confidence,
                    raw_value=sim.suspicious_domain,
                    mitre_technique="T1566.002"
                ))
                score += 15 * sim.confidence

    # Check 2: VIP impersonation
    if analysis.from_display_name:
        impersonations = detect_impersonation(analysis.from_display_name, vip_names)
        for imp in impersonations:
            analysis.indicators.append(SpearphishingIndicator(
                indicator_type="vip_impersonation",
                description=f"Display name '{imp['display_name']}' matches VIP "
                            f"keyword '{imp['matched_vip']}'",
                severity="high",
                confidence=imp["confidence"],
                raw_value=analysis.from_display_name,
                mitre_technique="T1566.001"
            ))
            score += 25 * imp["confidence"]

    # Check 3: Urgency indicators in subject
    urgency_subject = detect_urgency(analysis.subject)
    if urgency_subject:
        analysis.indicators.append(SpearphishingIndicator(
            indicator_type="urgency_subject",
            description=f"Subject contains {len(urgency_subject)} urgency pattern(s)",
            severity="medium",
            confidence=min(len(urgency_subject) * 0.3, 0.9),
            raw_value=analysis.subject,
            mitre_technique="T1566"
        ))
        score += min(len(urgency_subject) * 5, 20)

    # Check 4: Urgency indicators in body
    if body:
        urgency_body = detect_urgency(body)
        if urgency_body:
            total_matches = sum(u["count"] for u in urgency_body)
            analysis.indicators.append(SpearphishingIndicator(
                indicator_type="urgency_body",
                description=f"Body contains {total_matches} urgency keyword(s) "
                            f"across {len(urgency_body)} pattern(s)",
                severity="medium",
                confidence=min(total_matches * 0.15, 0.9),
                raw_value=f"{total_matches} matches",
                mitre_technique="T1566"
            ))
            score += min(total_matches * 3, 15)

    # Check 5: Authentication failures
    auth_results = headers.get("authentication_results", "")
    if auth_results:
        if "spf=fail" in auth_results.lower() or "spf=softfail" in auth_results.lower():
            analysis.indicators.append(SpearphishingIndicator(
                indicator_type="spf_failure",
                description="SPF authentication failed",
                severity="high",
                confidence=0.7,
                raw_value=auth_results,
                mitre_technique="T1566"
            ))
            score += 20

        if "dkim=fail" in auth_results.lower():
            analysis.indicators.append(SpearphishingIndicator(
                indicator_type="dkim_failure",
                description="DKIM authentication failed",
                severity="high",
                confidence=0.7,
                raw_value=auth_results,
                mitre_technique="T1566"
            ))
            score += 20

        if "dmarc=fail" in auth_results.lower():
            analysis.indicators.append(SpearphishingIndicator(
                indicator_type="dmarc_failure",
                description="DMARC authentication failed",
                severity="critical",
                confidence=0.8,
                raw_value=auth_results,
                mitre_technique="T1566"
            ))
            score += 25

    # Check 6: Reply-to mismatch
    reply_to = headers.get("reply_to", "")
    if reply_to and analysis.from_address:
        reply_domain = ""
        match = re.search(r'@([\w.-]+)', reply_to)
        if match:
            reply_domain = match.group(1).lower()
        if reply_domain and reply_domain != analysis.from_domain:
            analysis.indicators.append(SpearphishingIndicator(
                indicator_type="reply_to_mismatch",
                description=f"Reply-To domain ({reply_domain}) differs from "
                            f"From domain ({analysis.from_domain})",
                severity="high",
                confidence=0.85,
                raw_value=f"From: {analysis.from_domain}, Reply-To: {reply_domain}",
                mitre_technique="T1566"
            ))
            score += 20

    # Calculate final risk
    analysis.risk_score = min(score, 100)
    if analysis.risk_score >= 70:
        analysis.risk_level = "critical"
        analysis.is_spearphishing = True
    elif analysis.risk_score >= 50:
        analysis.risk_level = "high"
        analysis.is_spearphishing = True
    elif analysis.risk_score >= 30:
        analysis.risk_level = "medium"
    elif analysis.risk_score >= 10:
        analysis.risk_level = "low"
    else:
        analysis.risk_level = "clean"

    return analysis


def generate_detection_rules(indicators_db: list) -> str:
    """Generate YAML detection rules from accumulated indicators."""
    rules = []

    # Group by indicator type
    by_type = defaultdict(list)
    for ind in indicators_db:
        by_type[ind["indicator_type"]].append(ind)

    rule_id = 1
    for ind_type, indicators in by_type.items():
        values = list(set(ind.get("raw_value", "") for ind in indicators if ind.get("raw_value")))
        if not values:
            continue

        rule = {
            "id": f"SPEAR-{rule_id:04d}",
            "name": f"Spearphishing {ind_type.replace('_', ' ').title()} Detection",
            "type": ind_type,
            "severity": indicators[0].get("severity", "medium"),
            "mitre": indicators[0].get("mitre_technique", "T1566"),
            "description": indicators[0].get("description", ""),
            "action": "quarantine" if indicators[0].get("severity") in ("high", "critical") else "tag",
            "values_count": len(values),
            "sample_values": values[:5]
        }
        rules.append(rule)
        rule_id += 1

    # Format as YAML-like output
    output_lines = ["# Auto-generated Spearphishing Detection Rules",
                    f"# Generated: {datetime.now(timezone.utc).isoformat()}",
                    f"# Total rules: {len(rules)}", ""]

    for rule in rules:
        output_lines.append(f"- id: {rule['id']}")
        output_lines.append(f"  name: \"{rule['name']}\"")
        output_lines.append(f"  type: {rule['type']}")
        output_lines.append(f"  severity: {rule['severity']}")
        output_lines.append(f"  mitre: {rule['mitre']}")
        output_lines.append(f"  action: {rule['action']}")
        output_lines.append(f"  indicators_count: {rule['values_count']}")
        output_lines.append(f"  sample_values:")
        for val in rule["sample_values"]:
            output_lines.append(f"    - \"{val}\"")
        output_lines.append("")

    return "\n".join(output_lines)


def format_analysis_report(analysis: EmailAnalysis) -> str:
    """Format analysis as text report."""
    lines = []
    lines.append("=" * 60)
    lines.append("  SPEARPHISHING DETECTION REPORT")
    lines.append("=" * 60)
    lines.append(f"  Risk Level: {analysis.risk_level.upper()} "
                 f"(Score: {analysis.risk_score:.0f}/100)")
    lines.append(f"  Verdict: {'SPEARPHISHING DETECTED' if analysis.is_spearphishing else 'NOT DETECTED'}")
    lines.append("")
    lines.append(f"  From: {analysis.from_display_name} <{analysis.from_address}>")
    lines.append(f"  To: {analysis.to_address}")
    lines.append(f"  Subject: {analysis.subject}")
    lines.append(f"  Date: {analysis.date}")
    lines.append("")

    if analysis.indicators:
        lines.append(f"[INDICATORS] ({len(analysis.indicators)} found)")
        for i, ind in enumerate(analysis.indicators, 1):
            lines.append(f"  {i}. [{ind.severity.upper()}] {ind.description}")
            lines.append(f"     Type: {ind.indicator_type} | "
                         f"MITRE: {ind.mitre_technique} | "
                         f"Confidence: {ind.confidence:.0%}")
    else:
        lines.append("[INDICATORS] None found")

    if analysis.domain_similarities:
        lines.append(f"\n[DOMAIN ANALYSIS]")
        for sim in analysis.domain_similarities:
            lines.append(f"  {sim.suspicious_domain} ~ {sim.original_domain} "
                         f"(distance={sim.distance}, technique={sim.technique})")

    lines.append("=" * 60)
    return "\n".join(lines)


def main():
    parser = argparse.ArgumentParser(description="Spearphishing Detection Engine")
    subparsers = parser.add_subparsers(dest="command")

    analyze_parser = subparsers.add_parser("analyze", help="Analyze gateway log file")
    analyze_parser.add_argument("--log-file", required=True, help="JSON log file")
    analyze_parser.add_argument("--output", "-o", help="Output file")

    detect_parser = subparsers.add_parser("detect", help="Detect spearphishing in single email")
    detect_parser.add_argument("--from", dest="from_addr", required=True)
    detect_parser.add_argument("--from-name", default="")
    detect_parser.add_argument("--to", dest="to_addr", default="")
    detect_parser.add_argument("--subject", default="")
    detect_parser.add_argument("--body", default="")
    detect_parser.add_argument("--auth-results", default="")

    domain_parser = subparsers.add_parser("check-domain", help="Check domain similarity")
    domain_parser.add_argument("domain", help="Domain to check")

    rules_parser = subparsers.add_parser("rules", help="Generate detection rules from log")
    rules_parser.add_argument("--log-file", required=True)
    rules_parser.add_argument("--output", "-o", help="Output rules file")

    parser.add_argument("--json", action="store_true", help="Output as JSON")

    args = parser.parse_args()

    if args.command == "analyze":
        with open(args.log_file, "r") as f:
            log_entries = json.load(f)

        results = []
        for entry in log_entries:
            headers = entry.get("headers", entry)
            body = entry.get("body", "")
            analysis = analyze_email(headers, body)
            results.append(analysis)

        spearphishing_count = sum(1 for r in results if r.is_spearphishing)
        print(f"Analyzed {len(results)} emails, detected {spearphishing_count} spearphishing attempts")
        print()

        for analysis in results:
            if analysis.is_spearphishing:
                if args.json:
                    print(json.dumps(asdict(analysis), indent=2, default=str))
                else:
                    print(format_analysis_report(analysis))
                print()

    elif args.command == "detect":
        headers = {
            "from": args.from_addr,
            "from_display_name": args.from_name,
            "to": args.to_addr,
            "subject": args.subject,
            "authentication_results": args.auth_results,
        }
        analysis = analyze_email(headers, args.body)
        if args.json:
            print(json.dumps(asdict(analysis), indent=2, default=str))
        else:
            print(format_analysis_report(analysis))

    elif args.command == "check-domain":
        similarities = check_domain_similarity(args.domain)
        if similarities:
            print(f"Domain '{args.domain}' is similar to:")
            for sim in similarities:
                print(f"  - {sim.original_domain} (distance={sim.distance}, "
                      f"technique={sim.technique}, confidence={sim.confidence:.0%})")
        else:
            print(f"Domain '{args.domain}' has no known similarities")

    elif args.command == "rules":
        with open(args.log_file, "r") as f:
            log_entries = json.load(f)

        all_indicators = []
        for entry in log_entries:
            analysis = analyze_email(entry.get("headers", entry), entry.get("body", ""))
            for ind in analysis.indicators:
                all_indicators.append(asdict(ind))

        rules = generate_detection_rules(all_indicators)
        if args.output:
            with open(args.output, "w") as f:
                f.write(rules)
            print(f"Rules written to {args.output}")
        else:
            print(rules)

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-anti-phishing-training-program/SKILL.md =====
---
name: implementing-anti-phishing-training-program
description: Guides designing, deploying, and measuring an anti-phishing security awareness program - baseline phishing simulations, interactive training modules, just-in-time learning, and metric tracking - using platforms like KnowBe4, Proofpoint Security Awareness, or Cofense. Use when building or maturing a phishing awareness program, establishing training controls for compliance, or measuring phishing susceptibility and reporting rates over time.
domain: cybersecurity
subdomain: phishing-defense
tags:
- phishing
- email-security
- social-engineering
- dmarc
- awareness
- training
- security-culture
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566
- T1598
- T1534
- T1036
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - initial-access
  - stealth
  techniques:
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: T1598
    name: Phishing for Information
    tactic: reconnaissance
    source: attack
  - id: T1672
    name: Email Spoofing
    tactic: stealth
    source: attack
  - id: F1032
    name: Impersonate Official
    tactic: initial-access
    source: f3
  - id: F1031
    name: Impersonate Account Holder
    tactic: initial-access
    source: f3
---
# Implementing Anti-Phishing Training Program

## Overview
Security awareness training is the human layer of phishing defense. An effective anti-phishing training program combines regular simulations, interactive learning modules, metric tracking, and positive reinforcement to build a security-conscious culture. This skill covers designing, deploying, and measuring a comprehensive phishing awareness program using platforms like KnowBe4, Proofpoint Security Awareness, and open-source alternatives.


## When to Use

- When deploying or configuring implementing anti phishing training program capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites
- Management buy-in and budget approval
- Security awareness training platform (KnowBe4, Proofpoint SAT, Cofense)
- Employee email list and organizational structure
- Baseline phishing susceptibility data (from initial simulation)
- Learning management system (LMS) integration capability

## Key Concepts

### Training Program Pillars
1. **Baseline Assessment**: Initial phishing simulation to measure current susceptibility
2. **Interactive Training**: Role-based modules covering phishing identification
3. **Regular Simulations**: Monthly/quarterly phishing tests with progressive difficulty
4. **Just-in-Time Learning**: Immediate training after a user fails a simulation
5. **Positive Reinforcement**: Recognition for reporting phishing correctly
6. **Metrics & Reporting**: Track improvement over time by department and role

### SANS Security Awareness Maturity Model
- **Level 1**: Non-existent - No program
- **Level 2**: Compliance-focused - Annual checkbox training
- **Level 3**: Promoting Awareness - Engaging, regular content
- **Level 4**: Long-term Sustainment - Continuous program with culture change
- **Level 5**: Metrics Framework - Risk-based measurement and optimization

## Workflow

### Step 1: Establish Baseline
- Run initial phishing simulation across all departments
- Measure click rate, submit rate, and report rate
- Identify high-risk departments and roles

### Step 2: Design Curriculum
- **General awareness**: Phishing identification basics for all employees
- **Role-specific**: Finance (BEC/wire fraud), IT (credential phishing), Executives (whaling)
- **Progressive difficulty**: Beginner, intermediate, advanced modules
- **Micro-learning**: Short (3-5 minute) frequent sessions vs. annual marathon

### Step 3: Deploy Training Platform
- Configure KnowBe4/Proofpoint SAT with organizational groups
- Set up automated enrollment workflows
- Integrate with LMS for completion tracking
- Configure reporting dashboards

### Step 4: Run Continuous Simulations
- Monthly simulations with varied scenarios
- Increase difficulty based on organizational performance
- Include diverse attack types: links, attachments, QR codes, BEC

### Step 5: Measure and Optimize
Use `scripts/process.py` to analyze training completion, simulation results, and program effectiveness over time.

## Tools & Resources
- **KnowBe4**: https://www.knowbe4.com/
- **Proofpoint Security Awareness**: https://www.proofpoint.com/us/products/security-awareness-training
- **Cofense PhishMe**: https://cofense.com/
- **SANS Security Awareness**: https://www.sans.org/security-awareness-training/
- **Terranova Security**: https://terranovasecurity.com/

## Validation
- 90%+ training completion rate across organization
- Measurable reduction in phishing click rate over 6 months
- Increase in user phishing report rate
- Department-level improvement tracking

===== END FILE =====


===== FILE: skills/implementing-anti-phishing-training-program/LICENSE =====

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


===== FILE: skills/implementing-anti-phishing-training-program/assets/template.md =====
# Anti-Phishing Training Program Plan Template

## Program Overview
- **Organization**: [Company Name]
- **Program Owner**: [CISO / Security Awareness Manager]
- **Start Date**: [YYYY-MM-DD]
- **Review Cycle**: [Quarterly / Semi-annual]

## Baseline Metrics
| Metric | Current Value | 6-Month Target | 12-Month Target |
|---|---|---|---|
| Phish Click Rate | % | % | <5% |
| Submit Rate | % | % | <2% |
| Report Rate | % | % | >60% |
| Training Completion | % | 90% | 95% |
| Repeat Offenders | % | % | <3% |

## Training Curriculum
| Module | Audience | Duration | Frequency | Platform |
|---|---|---|---|---|
| Phishing Basics | All employees | 15 min | Annual + new hire | |
| BEC/Wire Fraud | Finance | 20 min | Quarterly | |
| Executive Threats | C-suite/VPs | 10 min | Quarterly | |
| IT-Targeted Phishing | IT Staff | 20 min | Quarterly | |
| Credential Protection | All employees | 10 min | Semi-annual | |
| Mobile Threats (Smishing) | All employees | 10 min | Annual | |

## Simulation Schedule
| Month | Scenario Type | Difficulty | Target Group |
|---|---|---|---|
| Jan | Password reset | Easy | All employees |
| Feb | Vendor invoice | Medium | Finance |
| Mar | IT notification | Easy | All employees |
| Apr | Package delivery | Easy | All employees |
| May | Executive request | Hard | Finance + Admin |
| Jun | Cloud storage share | Medium | All employees |
| Jul | HR policy update | Easy | All employees |
| Aug | Tax document | Medium | All employees |
| Sep | Conference invite | Medium | All employees |
| Oct | Security alert | Hard | IT Staff |
| Nov | Benefits enrollment | Medium | All employees |
| Dec | Holiday promotion | Hard | All employees |

## Escalation Policy for Repeat Offenders
| Offense | Action | Responsible |
|---|---|---|
| First click | Just-in-time training + auto-enroll in module | Automated |
| Second click | Enhanced training + coaching offer | Security team |
| Third click | Mandatory training + manager notification | HR + Security |
| Fourth+ click | Access review + remediation plan | HR + IT + Security |

## Budget
| Item | Annual Cost |
|---|---|
| Training platform license | $ |
| Simulation platform | $ |
| Content development | $ |
| Program administration | $ |
| Rewards/recognition | $ |
| **Total** | **$** |

## Reporting Schedule
- **Monthly**: Simulation results to security team
- **Quarterly**: Department-level report to leadership
- **Annually**: Full program assessment and ROI report to board

===== END FILE =====


===== FILE: skills/implementing-anti-phishing-training-program/references/api-reference.md =====
# API Reference: Implementing Anti-Phishing Training Program

## KnowBe4 API

```python
import requests

headers = {"Authorization": "Bearer <API_KEY>"}
base = "https://us.api.knowbe4.com/v1"

# List users
users = requests.get(f"{base}/users", headers=headers).json()

# Get phishing campaign results
campaigns = requests.get(f"{base}/phishing/campaigns", headers=headers).json()

# Get training enrollments
enrollments = requests.get(f"{base}/training/enrollments", headers=headers).json()
```

## Key Metrics

| Metric | Target | Calculation |
|--------|--------|-------------|
| Click Rate | < 15% | Clicked / Total Recipients |
| Submit Rate | < 5% | Submitted Creds / Total |
| Report Rate | > 70% | Reported / Total Recipients |
| Completion Rate | > 90% | Completed / Enrolled |

## pandas Simulation Analysis

```python
import pandas as pd
df = pd.read_csv("simulation_results.csv", parse_dates=["timestamp"])

# Department click rates
dept = df.groupby("department").agg(
    click_rate=("clicked", "mean"),
    report_rate=("reported", "mean"),
)

# Monthly trend
monthly = df.set_index("timestamp").resample("M")["clicked"].mean()
```

## SANS Maturity Model Levels

| Level | Name | Description |
|-------|------|-------------|
| 1 | Non-existent | No program |
| 2 | Compliance | Annual checkbox |
| 3 | Awareness | Engaging, regular |
| 4 | Sustainment | Culture change |
| 5 | Metrics | Risk-based optimization |

## GoPhish (Open-Source Alternative)

```bash
# Launch campaign
curl -X POST https://gophish:3333/api/campaigns \
  -H "Authorization: <API_KEY>" \
  -d '{"name":"Q1-2025","template":{"name":"IT Alert"},"groups":[{"name":"All Staff"}]}'
```

### References

- KnowBe4 API: https://developer.knowbe4.com/
- GoPhish: https://getgophish.com/
- SANS Security Awareness: https://www.sans.org/security-awareness-training/

===== END FILE =====


===== FILE: skills/implementing-anti-phishing-training-program/references/standards.md =====
# Standards & References: Anti-Phishing Training Program

## NIST Guidelines
- **NIST SP 800-50**: Building an Information Technology Security Awareness and Training Program
- **NIST SP 800-16**: Information Technology Security Training Requirements
- **NIST SP 800-53 Rev.5**: AT-1 through AT-6 - Awareness and Training family

## Regulatory Requirements
- **PCI DSS 4.0**: Requirement 12.6 - Security awareness training for all personnel
- **HIPAA**: 45 CFR 164.308(a)(5) - Security awareness and training
- **SOX**: Section 404 - Internal controls requiring security awareness
- **GDPR**: Article 39(1)(b) - Data protection awareness training
- **CMMC 2.0**: AT.L2-3.2.1/2/3 - Awareness and training practices
- **FFIEC**: Information Security Handbook - Security awareness training

## Industry Frameworks
- **SANS Security Awareness Maturity Model**: Five-level maturity assessment
- **AISA Phishing Resilience Protocol**: Australian standard for phishing testing
- **CISA Cybersecurity Awareness Program**: Federal awareness guidance

## MITRE ATT&CK Techniques Addressed by Training
- **T1566**: Phishing (all sub-techniques)
- **T1598**: Phishing for Information
- **T1204**: User Execution
- **T1534**: Internal Spearphishing

## Key Performance Indicators (KPIs)

| KPI | Description | Target |
|---|---|---|
| Phish-Prone Percentage | Users who click simulated phishing | < 5% |
| Training Completion Rate | Users completing assigned modules | > 95% |
| Report Rate | Users reporting simulated phishing | > 70% |
| Time to Report | Average time to report phishing | < 5 minutes |
| Repeat Offender Rate | Users failing multiple simulations | < 2% |
| Training Satisfaction | Post-training survey score | > 4/5 |
| Knowledge Assessment Score | Quiz/test average score | > 85% |

## Training Content Categories
1. Email phishing identification
2. Business email compromise (BEC)
3. Spearphishing and whaling
4. Vishing (voice phishing)
5. Smishing (SMS phishing)
6. QR code phishing (quishing)
7. Social media phishing
8. Credential harvesting
9. Malicious attachments
10. USB/physical social engineering

===== END FILE =====


===== FILE: skills/implementing-anti-phishing-training-program/references/workflows.md =====
# Workflows: Anti-Phishing Training Program

## Workflow 1: Annual Program Lifecycle

```
Q1: Baseline & Planning
  +-- Run baseline phishing simulation
  +-- Assess current awareness maturity level
  +-- Define annual targets and KPIs
  +-- Select/renew training platform
  +-- Design curriculum by role and department
  |
Q2: Foundation Training
  +-- Deploy core phishing awareness modules
  +-- Run monthly simulations (easy difficulty)
  +-- Launch phishing report button
  +-- Begin tracking metrics
  |
Q3: Advanced Training
  +-- Role-specific training (finance, IT, executives)
  +-- Increase simulation difficulty
  +-- Recognize security champions
  +-- Mid-year metrics review
  |
Q4: Assessment & Optimization
  +-- Run year-end assessment simulation
  +-- Compare against baseline
  +-- Generate annual report
  +-- Identify gaps for next year
  +-- Present ROI to leadership
```

## Workflow 2: Just-in-Time Training Flow

```
User interacts with simulated phishing email
  |
  v
[Did user click the link?]
  |
  +-- NO (ignored or reported) --> Positive outcome tracked
  |     |
  |     +-- [Did user report it?]
  |           +-- YES --> Send congratulations, award points
  |           +-- NO --> No action (not a failure)
  |
  +-- YES (clicked link)
       |
       v
  [Landing page shows "This was a test"]
       |
       v
  [Immediate micro-training module (2-3 min)]
       +-- What red flags were present
       +-- How to identify similar emails
       +-- How to report suspicious emails
       |
       v
  [Auto-enroll in refresher course within 7 days]
       |
       v
  [Manager receives aggregate report (not individual names)]
       |
       v
  [User included in next simulation cycle]
```

## Workflow 3: Repeat Offender Escalation

```
User fails first simulation
  |
  +-- Standard just-in-time training
  +-- Auto-enrolled in awareness module
  |
User fails second simulation (within 6 months)
  |
  +-- Enhanced training assignment
  +-- One-on-one coaching session offered
  +-- Manager notification (private)
  |
User fails third simulation
  |
  +-- Mandatory extended training
  +-- Access restrictions considered (additional MFA, restricted permissions)
  +-- HR involvement per policy
  +-- Monthly targeted simulations
  |
User passes subsequent simulation
  |
  +-- Return to normal simulation schedule
  +-- Positive reinforcement
```

## Workflow 4: Metrics-Driven Optimization

```
Monthly Data Collection
  |
  +-- Simulation results (click, submit, report rates)
  +-- Training completion rates
  +-- User-reported real phishing volume
  +-- Help desk phishing tickets
  |
  v
[Analyze by dimensions]
  +-- Department breakdown
  +-- Role/seniority breakdown
  +-- Location breakdown
  +-- Trend over time
  |
  v
[Identify patterns]
  +-- Which departments are improving?
  +-- Which scenarios are most effective?
  +-- Are repeat offenders decreasing?
  +-- Is report rate increasing?
  |
  v
[Adjust program]
  +-- Increase difficulty for high-performing groups
  +-- More training for struggling departments
  +-- New scenario types for common gaps
  +-- Update content for new threat trends
```

===== END FILE =====


===== FILE: skills/implementing-anti-phishing-training-program/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for managing and analyzing anti-phishing training program metrics."""

import json
import argparse
from datetime import datetime

import pandas as pd
import numpy as np


def load_simulation_results(csv_path):
    """Load phishing simulation results CSV."""
    df = pd.read_csv(csv_path, parse_dates=["timestamp"])
    return df


def calculate_department_metrics(df):
    """Calculate phishing susceptibility metrics per department."""
    results = []
    for dept, group in df.groupby("department"):
        total = len(group)
        clicked = group["clicked"].sum()
        submitted = group["submitted_credentials"].sum() if "submitted_credentials" in group.columns else 0
        reported = group["reported"].sum() if "reported" in group.columns else 0
        results.append({
            "department": dept,
            "total_recipients": int(total),
            "click_rate": round(clicked / total * 100, 1) if total > 0 else 0,
            "submission_rate": round(submitted / total * 100, 1) if total > 0 else 0,
            "report_rate": round(reported / total * 100, 1) if total > 0 else 0,
            "risk_level": "HIGH" if clicked / total > 0.3 else "MEDIUM" if clicked / total > 0.15 else "LOW",
        })
    return sorted(results, key=lambda x: x["click_rate"], reverse=True)


def analyze_trend(df):
    """Analyze phishing simulation trends over time."""
    df["month"] = df["timestamp"].dt.to_period("M")
    monthly = df.groupby("month").agg(
        total=("clicked", "count"),
        clicks=("clicked", "sum"),
    ).reset_index()
    monthly["click_rate"] = (monthly["clicks"] / monthly["total"] * 100).round(1)
    monthly["month"] = monthly["month"].astype(str)
    trend = monthly.to_dict(orient="records")
    if len(trend) >= 2:
        first_rate = trend[0]["click_rate"]
        last_rate = trend[-1]["click_rate"]
        improvement = round(first_rate - last_rate, 1)
    else:
        improvement = 0
    return {"monthly_data": trend, "improvement_pct": improvement}


def identify_repeat_clickers(df):
    """Identify users who repeatedly click phishing links."""
    clickers = df[df["clicked"] == True]
    repeat = clickers.groupby("email").agg(
        click_count=("clicked", "sum"),
        department=("department", "first"),
        name=("name", "first") if "name" in df.columns else ("email", "first"),
    ).reset_index()
    repeat = repeat[repeat["click_count"] >= 2].sort_values("click_count", ascending=False)
    return repeat.to_dict(orient="records")


def calculate_training_completion(training_df):
    """Calculate training module completion rates."""
    results = []
    for module, group in training_df.groupby("module_name"):
        total = len(group)
        completed = group["completed"].sum()
        results.append({
            "module": module,
            "enrolled": int(total),
            "completed": int(completed),
            "completion_rate": round(completed / total * 100, 1) if total > 0 else 0,
        })
    return sorted(results, key=lambda x: x["completion_rate"])


def generate_risk_score(dept_metrics):
    """Generate overall organization risk score based on phishing metrics."""
    if not dept_metrics:
        return {"score": 0, "grade": "N/A"}
    avg_click = np.mean([d["click_rate"] for d in dept_metrics])
    avg_report = np.mean([d["report_rate"] for d in dept_metrics])
    score = max(0, 100 - (avg_click * 2) + (avg_report * 0.5))
    if score >= 85:
        grade = "A"
    elif score >= 70:
        grade = "B"
    elif score >= 55:
        grade = "C"
    elif score >= 40:
        grade = "D"
    else:
        grade = "F"
    return {
        "score": round(score, 1),
        "grade": grade,
        "avg_click_rate": round(avg_click, 1),
        "avg_report_rate": round(avg_report, 1),
    }


def recommend_training(dept_metrics, repeat_clickers):
    """Generate training recommendations based on metrics."""
    recommendations = []
    high_risk_depts = [d for d in dept_metrics if d["risk_level"] == "HIGH"]
    for dept in high_risk_depts:
        recommendations.append({
            "target": dept["department"],
            "type": "department",
            "action": "Mandatory phishing awareness training",
            "priority": "HIGH",
            "reason": f"Click rate {dept['click_rate']}% exceeds 30% threshold",
        })
    for user in repeat_clickers[:20]:
        recommendations.append({
            "target": user.get("email", ""),
            "type": "individual",
            "action": "One-on-one coaching session",
            "priority": "CRITICAL",
            "reason": f"Clicked {user['click_count']} times across simulations",
        })
    return recommendations


def main():
    parser = argparse.ArgumentParser(description="Anti-Phishing Training Program Agent")
    parser.add_argument("--simulation-csv", help="Phishing simulation results CSV")
    parser.add_argument("--training-csv", help="Training completion CSV")
    parser.add_argument("--output", default="phishing_training_report.json")
    parser.add_argument("--action", choices=[
        "departments", "trends", "repeaters", "completion", "full_analysis"
    ], default="full_analysis")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "findings": {}}

    if args.simulation_csv:
        df = load_simulation_results(args.simulation_csv)
        print(f"[+] Loaded {len(df)} simulation results")

        if args.action in ("departments", "full_analysis"):
            metrics = calculate_department_metrics(df)
            report["findings"]["department_metrics"] = metrics
            report["findings"]["risk_score"] = generate_risk_score(metrics)
            print(f"[+] Departments analyzed: {len(metrics)}")

        if args.action in ("trends", "full_analysis"):
            trend = analyze_trend(df)
            report["findings"]["trend_analysis"] = trend
            print(f"[+] Improvement: {trend['improvement_pct']}%")

        if args.action in ("repeaters", "full_analysis"):
            repeaters = identify_repeat_clickers(df)
            report["findings"]["repeat_clickers"] = repeaters
            print(f"[+] Repeat clickers: {len(repeaters)}")

        if args.action == "full_analysis":
            metrics = report["findings"].get("department_metrics", [])
            repeaters = report["findings"].get("repeat_clickers", [])
            recs = recommend_training(metrics, repeaters)
            report["findings"]["recommendations"] = recs

    if args.training_csv:
        tdf = pd.read_csv(args.training_csv)
        completion = calculate_training_completion(tdf)
        report["findings"]["training_completion"] = completion
        print(f"[+] Training modules: {len(completion)}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-anti-phishing-training-program/scripts/process.py =====
#!/usr/bin/env python3
"""
Anti-Phishing Training Program Analytics

Tracks training completion, simulation results, and program effectiveness
over time. Generates reports comparing departments, identifying repeat
offenders, and measuring ROI.

Usage:
    python process.py dashboard --data program_data.json
    python process.py trend --data program_data.json --months 12
    python process.py repeat-offenders --data program_data.json
    python process.py department-report --data program_data.json
"""

import argparse
import json
import sys
from datetime import datetime, timezone
from collections import defaultdict
from dataclasses import dataclass, field, asdict


@dataclass
class UserRecord:
    """Training and simulation record for a single user."""
    email: str = ""
    name: str = ""
    department: str = ""
    role: str = ""
    simulations_sent: int = 0
    simulations_clicked: int = 0
    simulations_submitted: int = 0
    simulations_reported: int = 0
    trainings_assigned: int = 0
    trainings_completed: int = 0
    last_simulation_date: str = ""
    last_training_date: str = ""
    risk_level: str = "low"


@dataclass
class DepartmentMetrics:
    """Aggregated metrics for a department."""
    name: str = ""
    total_users: int = 0
    avg_click_rate: float = 0.0
    avg_submit_rate: float = 0.0
    avg_report_rate: float = 0.0
    training_completion: float = 0.0
    repeat_offenders: int = 0
    trend: str = "stable"


@dataclass
class ProgramDashboard:
    """Overall program dashboard metrics."""
    total_users: int = 0
    total_simulations_sent: int = 0
    overall_click_rate: float = 0.0
    overall_submit_rate: float = 0.0
    overall_report_rate: float = 0.0
    training_completion_rate: float = 0.0
    repeat_offender_count: int = 0
    repeat_offender_rate: float = 0.0
    maturity_level: int = 1
    departments: list = field(default_factory=list)
    top_risks: list = field(default_factory=list)
    monthly_trends: list = field(default_factory=list)


def calculate_risk_level(user: UserRecord) -> str:
    """Calculate risk level for a user based on simulation history."""
    if user.simulations_sent == 0:
        return "unknown"

    click_rate = user.simulations_clicked / user.simulations_sent
    submit_rate = user.simulations_submitted / user.simulations_sent

    if submit_rate > 0.3 or user.simulations_submitted >= 3:
        return "critical"
    elif click_rate > 0.4 or user.simulations_clicked >= 3:
        return "high"
    elif click_rate > 0.2:
        return "medium"
    elif user.simulations_reported > 0:
        return "low"
    else:
        return "low"


def assess_maturity(dashboard: ProgramDashboard) -> int:
    """Assess SANS Security Awareness Maturity level (1-5)."""
    if dashboard.total_simulations_sent == 0:
        return 1  # Non-existent

    if dashboard.training_completion_rate < 50:
        return 2  # Compliance-focused

    if dashboard.overall_click_rate > 15:
        return 2

    if dashboard.overall_click_rate > 5:
        return 3  # Promoting Awareness

    if dashboard.overall_report_rate > 50 and dashboard.overall_click_rate < 5:
        return 5  # Metrics Framework

    return 4  # Long-term Sustainment


def process_program_data(data: dict) -> ProgramDashboard:
    """Process raw program data into dashboard metrics."""
    dashboard = ProgramDashboard()

    users_data = data.get("users", [])
    simulations = data.get("simulations", [])
    trainings = data.get("trainings", [])

    # Build user records
    user_records = {}
    for u in users_data:
        record = UserRecord(
            email=u.get("email", ""),
            name=u.get("name", ""),
            department=u.get("department", "Unknown"),
            role=u.get("role", ""),
        )
        user_records[record.email] = record

    # Process simulation results
    for sim in simulations:
        for result in sim.get("results", []):
            email = result.get("email", "")
            if email in user_records:
                user = user_records[email]
                user.simulations_sent += 1
                if result.get("clicked"):
                    user.simulations_clicked += 1
                if result.get("submitted"):
                    user.simulations_submitted += 1
                if result.get("reported"):
                    user.simulations_reported += 1
                user.last_simulation_date = sim.get("date", "")

    # Process training completions
    for training in trainings:
        for completion in training.get("completions", []):
            email = completion.get("email", "")
            if email in user_records:
                user = user_records[email]
                user.trainings_assigned += 1
                if completion.get("completed"):
                    user.trainings_completed += 1
                user.last_training_date = training.get("date", "")

    # Calculate risk levels
    for user in user_records.values():
        user.risk_level = calculate_risk_level(user)

    # Aggregate overall metrics
    all_users = list(user_records.values())
    dashboard.total_users = len(all_users)

    total_sent = sum(u.simulations_sent for u in all_users)
    total_clicked = sum(u.simulations_clicked for u in all_users)
    total_submitted = sum(u.simulations_submitted for u in all_users)
    total_reported = sum(u.simulations_reported for u in all_users)
    total_assigned = sum(u.trainings_assigned for u in all_users)
    total_completed = sum(u.trainings_completed for u in all_users)

    dashboard.total_simulations_sent = total_sent
    dashboard.overall_click_rate = round(total_clicked / max(total_sent, 1) * 100, 1)
    dashboard.overall_submit_rate = round(total_submitted / max(total_sent, 1) * 100, 1)
    dashboard.overall_report_rate = round(total_reported / max(total_sent, 1) * 100, 1)
    dashboard.training_completion_rate = round(total_completed / max(total_assigned, 1) * 100, 1)

    # Repeat offenders (clicked 2+ times)
    repeat_offenders = [u for u in all_users if u.simulations_clicked >= 2]
    dashboard.repeat_offender_count = len(repeat_offenders)
    dashboard.repeat_offender_rate = round(
        len(repeat_offenders) / max(len(all_users), 1) * 100, 1
    )

    # Department breakdown
    dept_users = defaultdict(list)
    for user in all_users:
        dept_users[user.department].append(user)

    for dept_name, users in sorted(dept_users.items()):
        dept = DepartmentMetrics(name=dept_name, total_users=len(users))

        d_sent = sum(u.simulations_sent for u in users)
        d_clicked = sum(u.simulations_clicked for u in users)
        d_submitted = sum(u.simulations_submitted for u in users)
        d_reported = sum(u.simulations_reported for u in users)
        d_assigned = sum(u.trainings_assigned for u in users)
        d_completed = sum(u.trainings_completed for u in users)

        dept.avg_click_rate = round(d_clicked / max(d_sent, 1) * 100, 1)
        dept.avg_submit_rate = round(d_submitted / max(d_sent, 1) * 100, 1)
        dept.avg_report_rate = round(d_reported / max(d_sent, 1) * 100, 1)
        dept.training_completion = round(d_completed / max(d_assigned, 1) * 100, 1)
        dept.repeat_offenders = sum(1 for u in users if u.simulations_clicked >= 2)

        dashboard.departments.append(dept)

    # Top risk users
    risk_users = sorted(all_users, key=lambda u: u.simulations_submitted, reverse=True)
    dashboard.top_risks = [
        {"email": u.email, "name": u.name, "department": u.department,
         "click_count": u.simulations_clicked, "submit_count": u.simulations_submitted,
         "risk_level": u.risk_level}
        for u in risk_users[:20] if u.simulations_clicked > 0
    ]

    # Monthly trends from simulation data
    monthly = defaultdict(lambda: {"sent": 0, "clicked": 0, "submitted": 0, "reported": 0})
    for sim in simulations:
        month = sim.get("date", "")[:7]  # YYYY-MM
        for result in sim.get("results", []):
            monthly[month]["sent"] += 1
            if result.get("clicked"):
                monthly[month]["clicked"] += 1
            if result.get("submitted"):
                monthly[month]["submitted"] += 1
            if result.get("reported"):
                monthly[month]["reported"] += 1

    for month in sorted(monthly.keys()):
        m = monthly[month]
        dashboard.monthly_trends.append({
            "month": month,
            "sent": m["sent"],
            "click_rate": round(m["clicked"] / max(m["sent"], 1) * 100, 1),
            "submit_rate": round(m["submitted"] / max(m["sent"], 1) * 100, 1),
            "report_rate": round(m["reported"] / max(m["sent"], 1) * 100, 1),
        })

    dashboard.maturity_level = assess_maturity(dashboard)

    return dashboard


def format_dashboard(dashboard: ProgramDashboard) -> str:
    """Format dashboard as text report."""
    lines = []
    lines.append("=" * 65)
    lines.append("  ANTI-PHISHING TRAINING PROGRAM DASHBOARD")
    lines.append("=" * 65)
    lines.append(f"  Generated: {datetime.now(timezone.utc).strftime('%Y-%m-%d %H:%M UTC')}")
    lines.append(f"  Maturity Level: {dashboard.maturity_level}/5 (SANS Model)")
    lines.append("")

    lines.append("[PROGRAM OVERVIEW]")
    lines.append(f"  Total Users:              {dashboard.total_users}")
    lines.append(f"  Total Simulations Sent:   {dashboard.total_simulations_sent}")
    lines.append(f"  Overall Click Rate:       {dashboard.overall_click_rate}%")
    lines.append(f"  Overall Submit Rate:      {dashboard.overall_submit_rate}%")
    lines.append(f"  Overall Report Rate:      {dashboard.overall_report_rate}%")
    lines.append(f"  Training Completion:      {dashboard.training_completion_rate}%")
    lines.append(f"  Repeat Offenders:         {dashboard.repeat_offender_count} "
                 f"({dashboard.repeat_offender_rate}%)")
    lines.append("")

    lines.append("[DEPARTMENT BREAKDOWN]")
    lines.append(f"  {'Department':<20} {'Users':>6} {'Click%':>7} {'Submit%':>8} "
                 f"{'Report%':>8} {'Training%':>10} {'Repeat':>7}")
    lines.append("  " + "-" * 66)
    for dept in sorted(dashboard.departments, key=lambda d: d.avg_click_rate, reverse=True):
        lines.append(f"  {dept.name:<20} {dept.total_users:>6} {dept.avg_click_rate:>6.1f}% "
                     f"{dept.avg_submit_rate:>7.1f}% {dept.avg_report_rate:>7.1f}% "
                     f"{dept.training_completion:>9.1f}% {dept.repeat_offenders:>7}")
    lines.append("")

    if dashboard.top_risks:
        lines.append("[TOP RISK USERS]")
        for i, user in enumerate(dashboard.top_risks[:10], 1):
            lines.append(f"  {i}. {user['name']} ({user['department']}) - "
                         f"Clicked: {user['click_count']}, Submitted: {user['submit_count']} "
                         f"[{user['risk_level'].upper()}]")
        lines.append("")

    if dashboard.monthly_trends:
        lines.append("[MONTHLY TRENDS]")
        lines.append(f"  {'Month':<10} {'Sent':>6} {'Click%':>7} {'Submit%':>8} {'Report%':>8}")
        lines.append("  " + "-" * 39)
        for trend in dashboard.monthly_trends[-12:]:
            lines.append(f"  {trend['month']:<10} {trend['sent']:>6} "
                         f"{trend['click_rate']:>6.1f}% {trend['submit_rate']:>7.1f}% "
                         f"{trend['report_rate']:>7.1f}%")

    lines.append("")
    lines.append("=" * 65)
    return "\n".join(lines)


def main():
    parser = argparse.ArgumentParser(description="Anti-Phishing Training Program Analytics")
    subparsers = parser.add_subparsers(dest="command")

    dash_parser = subparsers.add_parser("dashboard", help="Generate program dashboard")
    dash_parser.add_argument("--data", required=True, help="Program data JSON file")
    dash_parser.add_argument("--output", "-o", help="Output file")

    dept_parser = subparsers.add_parser("department-report", help="Department breakdown")
    dept_parser.add_argument("--data", required=True)

    repeat_parser = subparsers.add_parser("repeat-offenders", help="List repeat offenders")
    repeat_parser.add_argument("--data", required=True)
    repeat_parser.add_argument("--threshold", type=int, default=2, help="Minimum click count")

    trend_parser = subparsers.add_parser("trend", help="Show monthly trends")
    trend_parser.add_argument("--data", required=True)
    trend_parser.add_argument("--months", type=int, default=12)

    parser.add_argument("--json", action="store_true", help="Output as JSON")

    args = parser.parse_args()

    if not args.command:
        parser.print_help()
        sys.exit(1)

    with open(args.data, "r") as f:
        data = json.load(f)

    dashboard = process_program_data(data)

    if args.command == "dashboard":
        if args.json:
            output = json.dumps(asdict(dashboard), indent=2, default=str)
        else:
            output = format_dashboard(dashboard)

        if args.output:
            with open(args.output, "w") as f:
                f.write(output)
            print(f"Dashboard written to {args.output}")
        else:
            print(output)

    elif args.command == "department-report":
        for dept in sorted(dashboard.departments, key=lambda d: d.avg_click_rate, reverse=True):
            if args.json:
                print(json.dumps(asdict(dept), indent=2))
            else:
                print(f"{dept.name}: {dept.total_users} users, "
                      f"click={dept.avg_click_rate}%, report={dept.avg_report_rate}%, "
                      f"training={dept.training_completion}%")

    elif args.command == "repeat-offenders":
        for user in dashboard.top_risks:
            if user["click_count"] >= args.threshold:
                print(f"  {user['name']} ({user['department']}): "
                      f"clicked {user['click_count']}x, submitted {user['submit_count']}x "
                      f"[{user['risk_level']}]")

    elif args.command == "trend":
        for trend in dashboard.monthly_trends[-args.months:]:
            print(f"  {trend['month']}: click={trend['click_rate']}%, "
                  f"submit={trend['submit_rate']}%, report={trend['report_rate']}%")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-dmarc-dkim-spf-email-security/SKILL.md =====
---
name: implementing-dmarc-dkim-spf-email-security
description: Configures SPF, DKIM, and DMARC DNS TXT records to authenticate outbound email, prevent domain spoofing, and enforce a rejection/quarantine policy on unauthenticated mail, including auditing a domain's current DNS state. Use when hardening a domain's email security posture or defending against phishing and spoofing attacks.
domain: cybersecurity
subdomain: phishing-defense
tags:
- phishing
- email-security
- social-engineering
- dmarc
- awareness
- dkim
- spf
- dns
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566
- T1598
- T1534
- T1036
---
# Implementing DMARC, DKIM, and SPF Email Security

## Overview
SPF, DKIM, and DMARC form the three pillars of email authentication. Together they prevent domain spoofing, validate message integrity, and define policies for handling unauthenticated mail. Proper implementation drastically reduces phishing attacks that impersonate your organization's domain.


## When to Use

- When deploying or configuring implementing dmarc dkim spf email security capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites
- DNS management access for your domain
- Access to email server/MTA configuration (Postfix, Exchange, Google Workspace, Microsoft 365)
- Basic understanding of DNS TXT records
- Python 3.8+ for validation scripts

## Key Concepts

### SPF (Sender Policy Framework)
Publishes a DNS TXT record listing authorized IP addresses and mail servers that can send email on behalf of your domain. Receiving servers check the envelope sender's IP against this list.

### DKIM (DomainKeys Identified Mail)
Adds a cryptographic signature to outgoing emails using a private key. The corresponding public key is published in DNS. Receivers verify the signature to ensure the message was not altered in transit.

### DMARC (Domain-based Message Authentication, Reporting and Conformance)
Builds on SPF and DKIM by specifying a policy (none/quarantine/reject) for messages that fail authentication, and provides a reporting mechanism to monitor spoofing attempts.

## Workflow

### Step 1: Audit Current State
```bash
# Check existing SPF record
dig TXT example.com | grep spf

# Check existing DKIM selector
dig TXT selector1._domainkey.example.com

# Check existing DMARC record
dig TXT _dmarc.example.com
```

### Step 2: Implement SPF
```
# DNS TXT record for example.com
v=spf1 ip4:203.0.113.0/24 include:_spf.google.com include:spf.protection.outlook.com -all
```

Key SPF mechanisms:
- `ip4:` / `ip6:` - Authorize specific IP ranges
- `include:` - Include another domain's SPF record
- `a` - Authorize domain's A record IPs
- `mx` - Authorize domain's MX record IPs
- `-all` - Hard fail all others (recommended)
- `~all` - Soft fail (monitoring phase)

### Step 3: Implement DKIM
```bash
# Generate DKIM key pair (2048-bit RSA)
openssl genrsa -out dkim_private.pem 2048
openssl rsa -in dkim_private.pem -pubout -out dkim_public.pem

# Format public key for DNS (remove headers, join lines)
grep -v "PUBLIC KEY" dkim_public.pem | tr -d '\n'
```

DNS TXT record at `selector1._domainkey.example.com`:
```
v=DKIM1; k=rsa; p=MIIBIjANBgkqhki...
```

### Step 4: Implement DMARC
```
# DNS TXT record at _dmarc.example.com
# Phase 1 (Monitor):
v=DMARC1; p=none; rua=mailto:dmarc-aggregate@example.com; ruf=mailto:dmarc-forensic@example.com; pct=100

# Phase 2 (Quarantine):
v=DMARC1; p=quarantine; rua=mailto:dmarc-aggregate@example.com; pct=25

# Phase 3 (Reject):
v=DMARC1; p=reject; rua=mailto:dmarc-aggregate@example.com; pct=100
```

### Step 5: Monitor and Analyze DMARC Reports
Use the `scripts/process.py` to parse DMARC aggregate XML reports and identify authentication failures, unauthorized senders, and spoofing attempts.

## Tools & Resources
- **MXToolbox**: https://mxtoolbox.com/SuperTool.aspx
- **DMARC Analyzer (dmarcian)**: https://dmarcian.com/
- **Google Postmaster Tools**: https://postmaster.google.com/
- **Valimail DMARC Monitor**: https://www.valimail.com/
- **DMARC Report Analyzer**: https://dmarc.postmarkapp.com/

## Validation
- SPF record passes validation at mxtoolbox.com
- DKIM signature verified on test emails
- DMARC record properly formatted and reporting enabled
- Test emails pass all three checks in recipient's Authentication-Results header

===== END FILE =====


===== FILE: skills/implementing-dmarc-dkim-spf-email-security/LICENSE =====

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


===== FILE: skills/implementing-dmarc-dkim-spf-email-security/assets/template.md =====
# Email Authentication Implementation Template

## Domain Information
- **Domain**: [example.com]
- **DNS Provider**: [Cloudflare / Route53 / GoDaddy]
- **Email Platform**: [Google Workspace / Microsoft 365 / Postfix]
- **Implementation Date**: [YYYY-MM-DD]

## Current State Assessment
| Check | Status | Record |
|---|---|---|
| SPF | Present/Missing | |
| DKIM | Present/Missing | |
| DMARC | Present/Missing | |

## Authorized Email Senders Inventory
| Service | Purpose | SPF Include | DKIM Selector |
|---|---|---|---|
| Primary MTA | Employee email | | |
| Google Workspace | Employee email | `_spf.google.com` | `google` |
| Microsoft 365 | Employee email | `spf.protection.outlook.com` | `selector1`, `selector2` |
| SendGrid | Transactional | `sendgrid.net` | `s1`, `s2` |
| Mailchimp | Marketing | `servers.mcsv.net` | `k1` |
| Amazon SES | Notifications | `amazonses.com` | Custom |
| Salesforce | CRM emails | `_spf.salesforce.com` | Custom |

## SPF Record Design
```
v=spf1 [mechanisms] [qualifier]
```

### DNS Lookup Budget (Max 10)
| # | Mechanism | Lookups |
|---|---|---|
| 1 | | |
| 2 | | |
| Total | | /10 |

## DKIM Configuration
| Selector | Key Length | Service | DNS Record |
|---|---|---|---|
| | 2048-bit | | |

## DMARC Rollout Plan
| Phase | Policy | pct | Duration | Start Date |
|---|---|---|---|---|
| 1 - Monitor | none | 100 | 4 weeks | |
| 2 - Quarantine Low | quarantine | 10 | 2 weeks | |
| 3 - Quarantine Medium | quarantine | 50 | 2 weeks | |
| 4 - Quarantine Full | quarantine | 100 | 2 weeks | |
| 5 - Reject Low | reject | 10 | 2 weeks | |
| 6 - Reject Medium | reject | 50 | 2 weeks | |
| 7 - Reject Full | reject | 100 | Ongoing | |

## DMARC Report Monitoring
- **Aggregate reports (rua)**: `mailto:dmarc-aggregate@[domain]`
- **Forensic reports (ruf)**: `mailto:dmarc-forensic@[domain]`
- **Analysis tool**: [dmarcian / Valimail / Postmark]
- **Review frequency**: Weekly during rollout, monthly after enforcement

## Validation Checklist
- [ ] SPF record validates at mxtoolbox.com/spf.aspx
- [ ] SPF DNS lookup count is under 10
- [ ] DKIM key is 2048-bit minimum
- [ ] DKIM signature verified on test email
- [ ] DMARC record validates at mxtoolbox.com/dmarc.aspx
- [ ] Aggregate reports receiving data
- [ ] All legitimate senders pass authentication
- [ ] No false positives in quarantine/reject

## Rollback Plan
If authentication causes delivery issues:
1. Change DMARC policy back to `p=none`
2. Investigate failing sources in aggregate reports
3. Update SPF/DKIM for legitimate failing senders
4. Re-start enforcement rollout at lower pct

===== END FILE =====


===== FILE: skills/implementing-dmarc-dkim-spf-email-security/references/api-reference.md =====
# API Reference: Implementing DMARC/DKIM/SPF Email Security

## dnspython Lookups

```python
import dns.resolver
# SPF
answers = dns.resolver.resolve("example.com", "TXT")
# DMARC
answers = dns.resolver.resolve("_dmarc.example.com", "TXT")
# DKIM
answers = dns.resolver.resolve("selector._domainkey.example.com", "TXT")
```

## SPF Record Syntax

| Mechanism | Example | Meaning |
|-----------|---------|---------|
| `include:` | `include:_spf.google.com` | Authorize sender |
| `ip4:` | `ip4:203.0.113.0/24` | Allow IP range |
| `-all` | End of record | Hard fail others |
| `~all` | End of record | Soft fail (weak) |
| `+all` | End of record | Allow all (insecure) |

## DMARC Policy Levels

| Policy | Action | Severity if Missing |
|--------|--------|---------------------|
| `p=reject` | Reject failing mail | Recommended |
| `p=quarantine` | Send to spam | Acceptable |
| `p=none` | Monitor only | HIGH risk |

## Recommended DNS Records

```
# SPF
v=spf1 include:_spf.google.com -all

# DMARC
v=DMARC1; p=reject; pct=100; rua=mailto:dmarc@example.com; adkim=s; aspf=s

# DKIM (provider-specific key)
selector._domainkey.example.com TXT "v=DKIM1; k=rsa; p=MIIBIjAN..."
```

### References

- SPF RFC 7208: https://www.rfc-editor.org/rfc/rfc7208
- DMARC RFC 7489: https://www.rfc-editor.org/rfc/rfc7489
- DKIM RFC 6376: https://www.rfc-editor.org/rfc/rfc6376
- dnspython: https://dnspython.readthedocs.io/

===== END FILE =====


===== FILE: skills/implementing-dmarc-dkim-spf-email-security/references/standards.md =====
# Standards & References: DMARC, DKIM, and SPF Email Security

## RFC Standards
- **RFC 7208 (SPF)**: Sender Policy Framework - authorizing sending hosts via DNS
- **RFC 6376 (DKIM)**: DomainKeys Identified Mail Signatures
- **RFC 7489 (DMARC)**: Domain-based Message Authentication, Reporting and Conformance
- **RFC 8301**: Cryptographic Algorithm and Key Usage Update to DKIM (mandates RSA 2048-bit minimum)
- **RFC 8616**: Email Authentication for Internationalized Mail
- **RFC 8617 (ARC)**: Authenticated Received Chain - preserving authentication through forwarding
- **RFC 7960**: Interoperability Issues between DMARC and Indirect Email Flows
- **RFC 6591**: Authentication Failure Reporting Using ARF
- **RFC 8601**: Authentication-Results header field

## NIST Guidelines
- **NIST SP 800-177 Rev.1**: Trustworthy Email - comprehensive email security deployment guide
  - Section 4.3: Sender Policy Framework
  - Section 4.4: DKIM
  - Section 4.5: DMARC
- **NIST SP 800-45 Ver.2**: Guidelines on Electronic Mail Security

## Government Mandates
- **BOD 18-01 (CISA/DHS)**: Binding Operational Directive requiring all federal agencies to implement DMARC p=reject
- **UK NCSC Mail Check**: Mandates DMARC for government domains
- **Australian ASD Essential Eight**: DMARC listed as a mitigation strategy

## MITRE ATT&CK
- **T1566**: Phishing (all sub-techniques)
- **T1586.002**: Compromise Accounts: Email Accounts
- **T1585.002**: Establish Accounts: Email Accounts

## Industry Best Practices
- **M3AAWG DMARC Training Series**: Messaging Anti-Abuse Working Group deployment guide
- **DMARC.org Implementation Guide**: Step-by-step deployment methodology
- **Google Email Authentication Requirements (2024)**: Bulk senders must have SPF, DKIM, and DMARC

## SPF Syntax Reference

| Mechanism | Description | Example |
|---|---|---|
| `ip4:` | IPv4 address/range | `ip4:192.168.1.0/24` |
| `ip6:` | IPv6 address/range | `ip6:2001:db8::/32` |
| `include:` | Include domain SPF | `include:_spf.google.com` |
| `a` | Domain A record IPs | `a:mail.example.com` |
| `mx` | Domain MX record IPs | `mx` |
| `redirect=` | Use another domain's SPF | `redirect=_spf.example.com` |
| `-all` | Hard fail | Reject unauthorized |
| `~all` | Soft fail | Mark but deliver |
| `?all` | Neutral | No assertion |

## DMARC Tag Reference

| Tag | Required | Description | Values |
|---|---|---|---|
| `v` | Yes | Version | `DMARC1` |
| `p` | Yes | Policy | `none`, `quarantine`, `reject` |
| `rua` | No | Aggregate report URI | `mailto:reports@example.com` |
| `ruf` | No | Forensic report URI | `mailto:forensic@example.com` |
| `pct` | No | Percentage of messages | `0-100` (default 100) |
| `sp` | No | Subdomain policy | `none`, `quarantine`, `reject` |
| `adkim` | No | DKIM alignment | `r` (relaxed), `s` (strict) |
| `aspf` | No | SPF alignment | `r` (relaxed), `s` (strict) |
| `fo` | No | Failure reporting | `0`, `1`, `d`, `s` |

===== END FILE =====


===== FILE: skills/implementing-dmarc-dkim-spf-email-security/references/workflows.md =====
# Workflows: Implementing DMARC, DKIM, and SPF

## Workflow 1: Phased DMARC Deployment

```
Phase 1: Discovery (Weeks 1-2)
  |
  +-- Inventory all email-sending services
  +-- Identify legitimate sources (marketing, transactional, internal)
  +-- Audit current SPF/DKIM/DMARC records
  |
Phase 2: SPF Setup (Weeks 2-4)
  |
  +-- Create SPF record with all authorized senders
  +-- Start with ~all (soft fail) for monitoring
  +-- Test with email validation tools
  |
Phase 3: DKIM Setup (Weeks 3-5)
  |
  +-- Generate 2048-bit RSA key pairs per sending service
  +-- Configure DKIM signing on MTA/email service
  +-- Publish public keys in DNS
  +-- Verify signatures with test emails
  |
Phase 4: DMARC Monitor (Weeks 5-8)
  |
  +-- Deploy DMARC with p=none
  +-- Set up aggregate report (rua) processing
  +-- Monitor for 4+ weeks
  +-- Identify and fix failing legitimate sources
  |
Phase 5: DMARC Quarantine (Weeks 9-12)
  |
  +-- Move to p=quarantine at pct=10
  +-- Gradually increase pct to 100
  +-- Monitor false positives
  |
Phase 6: DMARC Reject (Weeks 13+)
  |
  +-- Move to p=reject at pct=10
  +-- Gradually increase to pct=100
  +-- Ongoing monitoring and maintenance
```

## Workflow 2: SPF Record Construction

```
START: List all email sending sources
  |
  v
[Internal mail servers] --> ip4:x.x.x.x/y
  |
  v
[Cloud email (Google/M365)] --> include:_spf.google.com / include:spf.protection.outlook.com
  |
  v
[Marketing (Mailchimp, SendGrid)] --> include:servers.mcsv.net / include:sendgrid.net
  |
  v
[Transactional (SES, Postmark)] --> include:amazonses.com / include:spf.mtasv.net
  |
  v
[CRM (Salesforce, HubSpot)] --> include:_spf.salesforce.com / include:hubs.hubspot.com
  |
  v
[Combine all mechanisms, ensure < 10 DNS lookups]
  |
  v
[Add qualifier: ~all (monitor) or -all (enforce)]
  |
  v
[Publish TXT record at domain apex]
  |
  v
[Validate: mxtoolbox.com/spf.aspx]
```

## Workflow 3: DMARC Report Analysis

```
Daily DMARC aggregate reports (XML) received
  |
  v
[Parse XML reports with process.py]
  |
  v
[Categorize results]
  |
  +-- PASS (SPF + DKIM aligned) --> No action needed
  |
  +-- FAIL (unauthorized sender)
  |     |
  |     +-- Known service missing from SPF? --> Update SPF record
  |     +-- Known service missing DKIM? --> Configure DKIM signing
  |     +-- Unknown/suspicious sender --> Likely spoofing attempt
  |           |
  |           +-- Document source IPs
  |           +-- Add to threat intelligence
  |           +-- No SPF/DKIM changes needed (DMARC working correctly)
  |
  +-- PARTIAL PASS (SPF or DKIM only)
        |
        +-- Fix the failing mechanism
        +-- Check for forwarding/mailing list issues (consider ARC)
```

## Workflow 4: Ongoing Maintenance

```
Monthly:
  - Review DMARC aggregate reports for new unauthorized sources
  - Verify all third-party senders still authorized
  - Check SPF record is under 10 DNS lookup limit

Quarterly:
  - Rotate DKIM keys (update selector)
  - Review and update authorized sender inventory
  - Test authentication with external validation tools

Annually:
  - Full audit of email authentication configuration
  - Review DMARC policy strictness
  - Update documentation
```

===== END FILE =====


===== FILE: skills/implementing-dmarc-dkim-spf-email-security/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing and auditing DMARC, DKIM, and SPF email security."""

import json
import argparse
import re
from datetime import datetime

try:
    import dns.resolver
except ImportError:
    dns = None


def check_spf_record(domain):
    """Check SPF record for a domain."""
    try:
        answers = dns.resolver.resolve(domain, "TXT")
        for rdata in answers:
            txt = rdata.to_text().strip('"')
            if txt.startswith("v=spf1"):
                issues = []
                if "+all" in txt:
                    issues.append({"issue": "spf_plus_all", "severity": "CRITICAL"})
                if "~all" in txt:
                    issues.append({"issue": "spf_softfail", "severity": "MEDIUM"})
                lookups = txt.count("include:") + txt.count("a:") + txt.count("mx")
                if lookups > 10:
                    issues.append({"issue": "spf_too_many_lookups", "count": lookups, "severity": "HIGH"})
                return {"domain": domain, "record": txt, "valid": True, "issues": issues}
        return {"domain": domain, "record": None, "valid": False,
                "issues": [{"issue": "no_spf_record", "severity": "HIGH"}]}
    except Exception as e:
        return {"domain": domain, "error": str(e)}


def check_dmarc_record(domain):
    """Check DMARC record for a domain."""
    try:
        answers = dns.resolver.resolve(f"_dmarc.{domain}", "TXT")
        for rdata in answers:
            txt = rdata.to_text().strip('"')
            if txt.startswith("v=DMARC1"):
                issues = []
                policy_match = re.search(r"p=(\w+)", txt)
                policy = policy_match.group(1) if policy_match else "none"
                if policy == "none":
                    issues.append({"issue": "dmarc_policy_none", "severity": "HIGH"})
                pct_match = re.search(r"pct=(\d+)", txt)
                if pct_match and int(pct_match.group(1)) < 100:
                    issues.append({"issue": "dmarc_pct_not_100", "severity": "MEDIUM"})
                if "rua=" not in txt:
                    issues.append({"issue": "no_aggregate_reporting", "severity": "MEDIUM"})
                return {"domain": domain, "record": txt, "policy": policy, "issues": issues}
        return {"domain": domain, "record": None,
                "issues": [{"issue": "no_dmarc_record", "severity": "CRITICAL"}]}
    except Exception as e:
        return {"domain": domain, "error": str(e)}


def check_dkim_record(domain, selector="default"):
    """Check DKIM record for a domain and selector."""
    try:
        dkim_domain = f"{selector}._domainkey.{domain}"
        answers = dns.resolver.resolve(dkim_domain, "TXT")
        for rdata in answers:
            txt = rdata.to_text().strip('"')
            issues = []
            if "k=rsa" in txt:
                p_match = re.search(r"p=([A-Za-z0-9+/=]+)", txt)
                if p_match and len(p_match.group(1)) < 300:
                    issues.append({"issue": "weak_dkim_key", "severity": "HIGH"})
            return {"domain": domain, "selector": selector, "record": txt[:200], "issues": issues}
        return {"domain": domain, "selector": selector, "record": None}
    except Exception as e:
        return {"domain": domain, "selector": selector, "error": str(e)}


def audit_domains(domain_list):
    """Audit multiple domains for email security records."""
    results = []
    for domain in domain_list:
        result = {
            "domain": domain,
            "spf": check_spf_record(domain),
            "dmarc": check_dmarc_record(domain),
            "dkim": check_dkim_record(domain),
        }
        all_issues = (result["spf"].get("issues", []) + result["dmarc"].get("issues", [])
                     + result["dkim"].get("issues", []))
        critical = sum(1 for i in all_issues if i.get("severity") == "CRITICAL")
        result["risk_level"] = "CRITICAL" if critical > 0 else "HIGH" if len(all_issues) > 2 else "MEDIUM"
        results.append(result)
    return results


def generate_dns_records(domain, policy="reject"):
    """Generate recommended SPF, DMARC, and DKIM DNS records."""
    return {
        "spf": f'v=spf1 include:_spf.google.com include:spf.protection.outlook.com -all',
        "dmarc": f'v=DMARC1; p={policy}; pct=100; rua=mailto:dmarc-reports@{domain}; '
                 f'ruf=mailto:dmarc-forensics@{domain}; adkim=s; aspf=s',
        "dkim_selector": "google._domainkey",
        "notes": "DKIM key must be generated by your email provider",
    }


def main():
    parser = argparse.ArgumentParser(description="DMARC/DKIM/SPF Email Security Agent")
    parser.add_argument("--domains", nargs="+", help="Domains to audit")
    parser.add_argument("--generate", help="Domain to generate records for")
    parser.add_argument("--output", default="email_security_report.json")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "findings": {}}

    if args.domains:
        results = audit_domains(args.domains)
        report["findings"]["domain_audit"] = results
        for r in results:
            print(f"[+] {r['domain']}: {r['risk_level']}")

    if args.generate:
        records = generate_dns_records(args.generate)
        report["findings"]["recommended_records"] = records
        print(f"[+] Generated records for {args.generate}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-dmarc-dkim-spf-email-security/scripts/process.py =====
#!/usr/bin/env python3
"""
DMARC/DKIM/SPF Validator and DMARC Report Parser

Validates email authentication DNS records and parses DMARC aggregate
XML reports to identify unauthorized senders and authentication failures.

Usage:
    python process.py --check-domain example.com
    python process.py --parse-report dmarc_report.xml
    python process.py --parse-report-dir /path/to/reports/
"""

import argparse
import json
import sys
import xml.etree.ElementTree as ET
import gzip
import zipfile
import io
import os
from dataclasses import dataclass, field, asdict
from datetime import datetime, timezone
from pathlib import Path
from typing import Optional
from collections import defaultdict

try:
    import dns.resolver
    HAS_DNSPYTHON = True
except ImportError:
    HAS_DNSPYTHON = False

try:
    import requests
    HAS_REQUESTS = True
except ImportError:
    HAS_REQUESTS = False


@dataclass
class SPFRecord:
    """Parsed SPF record details."""
    raw: str = ""
    version: str = ""
    mechanisms: list = field(default_factory=list)
    includes: list = field(default_factory=list)
    ip4_ranges: list = field(default_factory=list)
    ip6_ranges: list = field(default_factory=list)
    qualifier: str = ""
    dns_lookup_count: int = 0
    valid: bool = False
    errors: list = field(default_factory=list)


@dataclass
class DKIMRecord:
    """Parsed DKIM record details."""
    selector: str = ""
    raw: str = ""
    version: str = ""
    key_type: str = ""
    public_key: str = ""
    key_length: int = 0
    valid: bool = False
    errors: list = field(default_factory=list)


@dataclass
class DMARCRecord:
    """Parsed DMARC record details."""
    raw: str = ""
    version: str = ""
    policy: str = ""
    subdomain_policy: str = ""
    pct: int = 100
    rua: list = field(default_factory=list)
    ruf: list = field(default_factory=list)
    adkim: str = "r"
    aspf: str = "r"
    fo: str = "0"
    valid: bool = False
    errors: list = field(default_factory=list)


@dataclass
class DMARCReportRecord:
    """Single record from a DMARC aggregate report."""
    source_ip: str = ""
    count: int = 0
    disposition: str = ""
    dkim_result: str = ""
    dkim_domain: str = ""
    spf_result: str = ""
    spf_domain: str = ""
    header_from: str = ""
    envelope_from: str = ""
    dkim_aligned: bool = False
    spf_aligned: bool = False


@dataclass
class DMARCReportSummary:
    """Summary of a parsed DMARC aggregate report."""
    org_name: str = ""
    report_id: str = ""
    date_begin: str = ""
    date_end: str = ""
    domain: str = ""
    total_messages: int = 0
    pass_count: int = 0
    fail_count: int = 0
    records: list = field(default_factory=list)
    top_failing_ips: list = field(default_factory=list)
    unauthorized_senders: list = field(default_factory=list)


def query_dns_txt(domain: str) -> list:
    """Query DNS TXT records for a domain."""
    if HAS_DNSPYTHON:
        try:
            answers = dns.resolver.resolve(domain, "TXT")
            results = []
            for rdata in answers:
                txt = b"".join(rdata.strings).decode("utf-8", errors="replace")
                results.append(txt)
            return results
        except (dns.resolver.NXDOMAIN, dns.resolver.NoAnswer,
                dns.resolver.NoNameservers, dns.resolver.Timeout):
            return []
    elif HAS_REQUESTS:
        try:
            resp = requests.get(
                f"https://dns.google/resolve?name={domain}&type=TXT",
                timeout=10
            )
            if resp.status_code == 200:
                data = resp.json()
                results = []
                for answer in data.get("Answer", []):
                    txt = answer.get("data", "").strip('"')
                    results.append(txt)
                return results
        except Exception:
            pass
    return []


def check_spf(domain: str) -> SPFRecord:
    """Check and validate SPF record for a domain."""
    record = SPFRecord()
    txt_records = query_dns_txt(domain)

    spf_records = [r for r in txt_records if r.startswith("v=spf1")]

    if not spf_records:
        record.errors.append("No SPF record found")
        return record

    if len(spf_records) > 1:
        record.errors.append(f"Multiple SPF records found ({len(spf_records)}) - RFC violation")

    record.raw = spf_records[0]
    record.version = "spf1"

    parts = record.raw.split()
    lookup_count = 0

    for part in parts[1:]:
        if part.startswith("include:"):
            domain_ref = part.split(":", 1)[1]
            record.includes.append(domain_ref)
            record.mechanisms.append(part)
            lookup_count += 1
        elif part.startswith("ip4:"):
            record.ip4_ranges.append(part.split(":", 1)[1])
            record.mechanisms.append(part)
        elif part.startswith("ip6:"):
            record.ip6_ranges.append(part.split(":", 1)[1])
            record.mechanisms.append(part)
        elif part.startswith(("a:", "a")):
            record.mechanisms.append(part)
            lookup_count += 1
        elif part.startswith(("mx:", "mx")):
            record.mechanisms.append(part)
            lookup_count += 1
        elif part.startswith("redirect="):
            record.mechanisms.append(part)
            lookup_count += 1
        elif part.startswith("exists:"):
            record.mechanisms.append(part)
            lookup_count += 1
        elif part in ("-all", "~all", "?all", "+all"):
            record.qualifier = part

    record.dns_lookup_count = lookup_count

    if lookup_count > 10:
        record.errors.append(f"SPF exceeds 10 DNS lookup limit ({lookup_count} lookups)")

    if record.qualifier == "+all":
        record.errors.append("SPF uses +all which allows any sender (insecure)")
    elif record.qualifier == "?all":
        record.errors.append("SPF uses ?all (neutral) - provides no protection")

    if not record.qualifier:
        record.errors.append("SPF record has no terminating mechanism (-all/~all)")

    record.valid = len(record.errors) == 0
    return record


def check_dkim(domain: str, selectors: list = None) -> list:
    """Check DKIM records for common selectors."""
    if selectors is None:
        selectors = [
            "selector1", "selector2",  # Microsoft 365
            "google", "default",  # Google Workspace
            "s1", "s2",  # Generic
            "dkim", "mail",  # Common
            "k1", "k2",  # Mailchimp
            "sm1", "sm2",  # SendGrid
        ]

    results = []
    for selector in selectors:
        record = DKIMRecord(selector=selector)
        dkim_domain = f"{selector}._domainkey.{domain}"
        txt_records = query_dns_txt(dkim_domain)

        dkim_records = [r for r in txt_records if "DKIM1" in r or "p=" in r]

        if dkim_records:
            record.raw = dkim_records[0]

            if "v=DKIM1" in record.raw:
                record.version = "DKIM1"

            import re
            key_match = re.search(r'k=(\w+)', record.raw)
            if key_match:
                record.key_type = key_match.group(1)
            else:
                record.key_type = "rsa"  # default

            pub_match = re.search(r'p=([A-Za-z0-9+/=]+)', record.raw)
            if pub_match:
                record.public_key = pub_match.group(1)
                import base64
                try:
                    key_bytes = base64.b64decode(record.public_key)
                    record.key_length = len(key_bytes) * 8
                except Exception:
                    pass

            if record.key_length and record.key_length < 2048:
                record.errors.append(
                    f"DKIM key is {record.key_length}-bit (2048-bit minimum recommended per RFC 8301)"
                )

            if not record.public_key:
                record.errors.append("DKIM record has empty public key (revoked)")

            record.valid = len(record.errors) == 0
            results.append(record)

    return results


def check_dmarc(domain: str) -> DMARCRecord:
    """Check and validate DMARC record for a domain."""
    record = DMARCRecord()
    dmarc_domain = f"_dmarc.{domain}"
    txt_records = query_dns_txt(dmarc_domain)

    dmarc_records = [r for r in txt_records if r.startswith("v=DMARC1")]

    if not dmarc_records:
        record.errors.append("No DMARC record found")
        return record

    record.raw = dmarc_records[0]
    record.version = "DMARC1"

    import re
    tags = {}
    for tag_match in re.finditer(r'(\w+)\s*=\s*([^;]+)', record.raw):
        tags[tag_match.group(1).strip()] = tag_match.group(2).strip()

    record.policy = tags.get("p", "")
    record.subdomain_policy = tags.get("sp", record.policy)
    record.adkim = tags.get("adkim", "r")
    record.aspf = tags.get("aspf", "r")
    record.fo = tags.get("fo", "0")

    if "pct" in tags:
        try:
            record.pct = int(tags["pct"])
        except ValueError:
            record.errors.append(f"Invalid pct value: {tags['pct']}")

    if "rua" in tags:
        record.rua = [uri.strip() for uri in tags["rua"].split(",")]
    if "ruf" in tags:
        record.ruf = [uri.strip() for uri in tags["ruf"].split(",")]

    if not record.policy:
        record.errors.append("DMARC record missing required p= tag")
    elif record.policy not in ("none", "quarantine", "reject"):
        record.errors.append(f"Invalid DMARC policy: {record.policy}")

    if record.policy == "none":
        record.errors.append("DMARC policy is 'none' (monitor only) - not enforcing")

    if not record.rua:
        record.errors.append("No aggregate report URI (rua) configured")

    record.valid = len([e for e in record.errors if "monitor only" not in e and "rua" not in e]) == 0
    return record


def parse_dmarc_report(xml_content: str) -> DMARCReportSummary:
    """Parse a DMARC aggregate XML report."""
    summary = DMARCReportSummary()

    try:
        root = ET.fromstring(xml_content)
    except ET.ParseError as e:
        print(f"Error parsing XML: {e}", file=sys.stderr)
        return summary

    # Report metadata
    metadata = root.find("report_metadata")
    if metadata is not None:
        summary.org_name = metadata.findtext("org_name", "")
        summary.report_id = metadata.findtext("report_id", "")
        date_range = metadata.find("date_range")
        if date_range is not None:
            begin = date_range.findtext("begin", "")
            end = date_range.findtext("end", "")
            if begin:
                summary.date_begin = datetime.fromtimestamp(
                    int(begin), tz=timezone.utc
                ).strftime("%Y-%m-%d")
            if end:
                summary.date_end = datetime.fromtimestamp(
                    int(end), tz=timezone.utc
                ).strftime("%Y-%m-%d")

    # Policy published
    policy = root.find("policy_published")
    if policy is not None:
        summary.domain = policy.findtext("domain", "")

    # Records
    failing_ips = defaultdict(int)

    for record_el in root.findall("record"):
        rec = DMARCReportRecord()

        row = record_el.find("row")
        if row is not None:
            rec.source_ip = row.findtext("source_ip", "")
            rec.count = int(row.findtext("count", "0"))

            policy_evaluated = row.find("policy_evaluated")
            if policy_evaluated is not None:
                rec.disposition = policy_evaluated.findtext("disposition", "")
                dkim_el = policy_evaluated.findtext("dkim", "")
                spf_el = policy_evaluated.findtext("spf", "")
                rec.dkim_aligned = dkim_el == "pass"
                rec.spf_aligned = spf_el == "pass"

        identifiers = record_el.find("identifiers")
        if identifiers is not None:
            rec.header_from = identifiers.findtext("header_from", "")
            rec.envelope_from = identifiers.findtext("envelope_from", "")

        auth_results = record_el.find("auth_results")
        if auth_results is not None:
            dkim_el = auth_results.find("dkim")
            if dkim_el is not None:
                rec.dkim_domain = dkim_el.findtext("domain", "")
                rec.dkim_result = dkim_el.findtext("result", "")

            spf_el = auth_results.find("spf")
            if spf_el is not None:
                rec.spf_domain = spf_el.findtext("domain", "")
                rec.spf_result = spf_el.findtext("result", "")

        summary.total_messages += rec.count
        if rec.dkim_aligned or rec.spf_aligned:
            summary.pass_count += rec.count
        else:
            summary.fail_count += rec.count
            failing_ips[rec.source_ip] += rec.count

        summary.records.append(rec)

    # Top failing IPs
    summary.top_failing_ips = sorted(
        failing_ips.items(), key=lambda x: x[1], reverse=True
    )[:20]

    return summary


def load_report_file(filepath: str) -> str:
    """Load a DMARC report file (handles .xml, .xml.gz, .zip)."""
    path = Path(filepath)

    if path.suffix == ".gz":
        with gzip.open(path, "rt", encoding="utf-8", errors="replace") as f:
            return f.read()
    elif path.suffix == ".zip":
        with zipfile.ZipFile(path) as zf:
            for name in zf.namelist():
                if name.endswith(".xml"):
                    with zf.open(name) as xf:
                        return xf.read().decode("utf-8", errors="replace")
    else:
        with open(path, "r", encoding="utf-8", errors="replace") as f:
            return f.read()
    return ""


def format_domain_check(domain: str, spf: SPFRecord, dkim_records: list,
                        dmarc: DMARCRecord) -> str:
    """Format domain authentication check results."""
    lines = []
    lines.append("=" * 70)
    lines.append(f"  EMAIL AUTHENTICATION CHECK: {domain}")
    lines.append("=" * 70)
    lines.append(f"  Date: {datetime.now(timezone.utc).strftime('%Y-%m-%d %H:%M UTC')}")
    lines.append("")

    # SPF
    status = "PASS" if spf.valid else "ISSUES"
    lines.append(f"[SPF] {status}")
    lines.append(f"  Record: {spf.raw}")
    lines.append(f"  IP4 Ranges: {', '.join(spf.ip4_ranges) or 'none'}")
    lines.append(f"  Includes: {', '.join(spf.includes) or 'none'}")
    lines.append(f"  Qualifier: {spf.qualifier}")
    lines.append(f"  DNS Lookups: {spf.dns_lookup_count}/10")
    for err in spf.errors:
        lines.append(f"  WARNING: {err}")
    lines.append("")

    # DKIM
    if dkim_records:
        for dkim in dkim_records:
            status = "PASS" if dkim.valid else "ISSUES"
            lines.append(f"[DKIM] {status} (selector: {dkim.selector})")
            lines.append(f"  Key Type: {dkim.key_type}")
            lines.append(f"  Key Length: {dkim.key_length} bits")
            for err in dkim.errors:
                lines.append(f"  WARNING: {err}")
    else:
        lines.append("[DKIM] NO RECORDS FOUND")
        lines.append("  Checked selectors: selector1, selector2, google, default, s1, s2, dkim, mail")
    lines.append("")

    # DMARC
    status = "PASS" if dmarc.valid else "ISSUES"
    lines.append(f"[DMARC] {status}")
    lines.append(f"  Record: {dmarc.raw}")
    lines.append(f"  Policy: {dmarc.policy}")
    lines.append(f"  Subdomain Policy: {dmarc.subdomain_policy}")
    lines.append(f"  Percentage: {dmarc.pct}%")
    lines.append(f"  DKIM Alignment: {dmarc.adkim} ({'relaxed' if dmarc.adkim == 'r' else 'strict'})")
    lines.append(f"  SPF Alignment: {dmarc.aspf} ({'relaxed' if dmarc.aspf == 'r' else 'strict'})")
    lines.append(f"  Aggregate Reports: {', '.join(dmarc.rua) or 'not configured'}")
    lines.append(f"  Forensic Reports: {', '.join(dmarc.ruf) or 'not configured'}")
    for err in dmarc.errors:
        lines.append(f"  WARNING: {err}")
    lines.append("")

    # Overall assessment
    lines.append("-" * 70)
    all_valid = spf.valid and dmarc.valid and any(d.valid for d in dkim_records)
    if all_valid and dmarc.policy == "reject":
        lines.append("  OVERALL: STRONG - Full email authentication with reject policy")
    elif all_valid and dmarc.policy == "quarantine":
        lines.append("  OVERALL: GOOD - Full authentication, consider upgrading to reject")
    elif all_valid:
        lines.append("  OVERALL: MONITORING - Authentication configured but DMARC not enforcing")
    else:
        lines.append("  OVERALL: WEAK - Email authentication has gaps")
    lines.append("=" * 70)

    return "\n".join(lines)


def format_report_summary(summary: DMARCReportSummary) -> str:
    """Format DMARC report summary."""
    lines = []
    lines.append("=" * 70)
    lines.append("  DMARC AGGREGATE REPORT SUMMARY")
    lines.append("=" * 70)
    lines.append(f"  Reporting Org: {summary.org_name}")
    lines.append(f"  Report ID: {summary.report_id}")
    lines.append(f"  Period: {summary.date_begin} to {summary.date_end}")
    lines.append(f"  Domain: {summary.domain}")
    lines.append("")
    lines.append(f"  Total Messages: {summary.total_messages}")
    lines.append(f"  Passed: {summary.pass_count} ({summary.pass_count*100//max(summary.total_messages,1)}%)")
    lines.append(f"  Failed: {summary.fail_count} ({summary.fail_count*100//max(summary.total_messages,1)}%)")
    lines.append("")

    if summary.top_failing_ips:
        lines.append("[TOP FAILING SOURCE IPs]")
        for ip, count in summary.top_failing_ips[:10]:
            lines.append(f"  {ip}: {count} messages")
        lines.append("")

    lines.append("[DETAILED RECORDS]")
    for rec in summary.records[:50]:
        status = "PASS" if (rec.dkim_aligned or rec.spf_aligned) else "FAIL"
        lines.append(f"  {rec.source_ip} ({rec.count} msgs) - {status}")
        lines.append(f"    Disposition: {rec.disposition} | "
                     f"DKIM: {rec.dkim_result} ({rec.dkim_domain}) | "
                     f"SPF: {rec.spf_result} ({rec.spf_domain})")

    lines.append("=" * 70)
    return "\n".join(lines)


def main():
    parser = argparse.ArgumentParser(
        description="DMARC/DKIM/SPF validator and DMARC report parser"
    )
    subparsers = parser.add_subparsers(dest="command")

    check_parser = subparsers.add_parser("check", help="Check domain authentication records")
    check_parser.add_argument("domain", help="Domain to check")
    check_parser.add_argument("--selectors", nargs="+", help="DKIM selectors to check")
    check_parser.add_argument("--json", action="store_true", help="Output as JSON")

    report_parser = subparsers.add_parser("report", help="Parse DMARC aggregate report")
    report_parser.add_argument("path", help="Path to XML report file or directory")
    report_parser.add_argument("--json", action="store_true", help="Output as JSON")

    # Support legacy --check-domain and --parse-report flags
    parser.add_argument("--check-domain", help="Domain to check (legacy)")
    parser.add_argument("--parse-report", help="Report file to parse (legacy)")
    parser.add_argument("--parse-report-dir", help="Directory of reports (legacy)")
    parser.add_argument("--json", action="store_true", help="Output as JSON")

    args = parser.parse_args()

    domain = getattr(args, "domain", None) or args.check_domain
    report_path = getattr(args, "path", None) or args.parse_report or args.parse_report_dir

    if domain:
        spf = check_spf(domain)
        dkim_records = check_dkim(domain, getattr(args, "selectors", None))
        dmarc = check_dmarc(domain)

        if args.json:
            result = {
                "domain": domain,
                "spf": asdict(spf),
                "dkim": [asdict(d) for d in dkim_records],
                "dmarc": asdict(dmarc),
            }
            print(json.dumps(result, indent=2))
        else:
            print(format_domain_check(domain, spf, dkim_records, dmarc))

    elif report_path:
        path = Path(report_path)
        if path.is_dir():
            for f in sorted(path.glob("*")):
                if f.suffix in (".xml", ".gz", ".zip"):
                    xml_content = load_report_file(str(f))
                    if xml_content:
                        summary = parse_dmarc_report(xml_content)
                        if args.json:
                            print(json.dumps(asdict(summary), indent=2, default=str))
                        else:
                            print(format_report_summary(summary))
                        print()
        else:
            xml_content = load_report_file(str(path))
            if xml_content:
                summary = parse_dmarc_report(xml_content)
                if args.json:
                    print(json.dumps(asdict(summary), indent=2, default=str))
                else:
                    print(format_report_summary(summary))
    else:
        parser.print_help()
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-email-sandboxing-with-proofpoint/SKILL.md =====
---
name: implementing-email-sandboxing-with-proofpoint
description: Email sandboxing detonates suspicious attachments and URLs in isolated
  environments to detect zero-day malware and evasive phishing payloads. Proofpoint
  Targeted Attack Protection (TAP) is an industry
domain: cybersecurity
subdomain: phishing-defense
tags:
- phishing
- email-security
- social-engineering
- dmarc
- awareness
- sandboxing
- proofpoint
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566
- T1598
- T1534
- T1036
- T1027
---
# Implementing Email Sandboxing with Proofpoint

## Overview
Email sandboxing detonates suspicious attachments and URLs in isolated environments to detect zero-day malware and evasive phishing payloads. Proofpoint Targeted Attack Protection (TAP) is an industry-leading solution that uses multi-stage sandboxing, URL rewriting, and predictive analysis. This skill covers configuring Proofpoint TAP, integrating with email flow, analyzing sandbox reports, and tuning detection policies.


## When to Use

- When deploying or configuring implementing email sandboxing with proofpoint capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites
- Proofpoint Email Protection license with TAP add-on
- Admin access to Proofpoint admin console
- Understanding of email delivery architecture (MX records, mail flow rules)
- SIEM integration capability

## Key Concepts

### Proofpoint TAP Capabilities
1. **Attachment sandboxing**: Detonates files in virtual machines (Windows, macOS, Android)
2. **URL Defense**: Rewrites URLs, detonates at time-of-click
3. **Threat Intelligence**: Proofpoint's NexusAI threat intelligence integration
4. **TAP Dashboard**: Real-time visibility into threats targeting the organization
5. **Campaign correlation**: Groups related attacks into campaigns
6. **Very Attacked People (VAP)**: Identifies most-targeted individuals

### Sandbox Evasion Techniques Detected
- Delayed execution (time-bomb malware)
- VM detection bypass
- User interaction requirements (click-to-enable macros)
- Sandbox-aware malware that checks for analysis environment
- Encrypted/password-protected attachments
- Multi-stage payloads with delayed C2 retrieval

## Workflow

### Step 1: Configure TAP in Proofpoint
- Enable TAP for inbound email policy
- Configure sandbox profiles (attachment types to detonate)
- Set URL Defense rewriting policy
- Configure quarantine actions for malicious verdicts

### Step 2: Tune Attachment Policies
```
Recommended attachment policy:
- Detonate: .exe, .dll, .scr, .doc(m), .xls(m), .ppt(m), .pdf, .zip, .rar, .7z, .iso
- Block without detonation: .bat, .cmd, .ps1, .vbs, .js, .wsf, .hta
- Password-protected archives: Attempt common passwords, then quarantine
- Dynamic delivery: Deliver email body, hold attachment until verdict
```

### Step 3: Configure URL Defense
- Enable URL rewriting for all inbound email
- Set time-of-click detonation
- Block access to malicious URLs
- Show warning page for suspicious (not confirmed malicious) URLs
- Configure allowed domains bypass list

### Step 4: Set Up TAP Dashboard Monitoring
- Configure daily threat digest emails to security team
- Set up real-time alerts for targeted attacks
- Monitor VAP report for high-risk users
- Review campaign clusters for coordinated attacks

### Step 5: Integrate with SIEM
- Configure syslog/API export to SIEM
- Create correlation rules for TAP alerts
- Set up automated response workflows

## Tools & Resources
- **Proofpoint TAP**: https://www.proofpoint.com/us/products/advanced-threat-protection
- **Proofpoint TAP Dashboard**: https://threatinsight.proofpoint.com/
- **Proofpoint API**: https://help.proofpoint.com/Threat_Insight_Dashboard/API_Documentation
- **Proofpoint Community**: https://community.proofpoint.com/

## Validation
- Attachment detonation catches EICAR test file and macro-enabled document
- URL Defense rewrites and blocks known phishing URLs
- TAP Dashboard displays threat summary
- SIEM receives and alerts on TAP events

===== END FILE =====


===== FILE: skills/implementing-email-sandboxing-with-proofpoint/LICENSE =====

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


===== FILE: skills/implementing-email-sandboxing-with-proofpoint/assets/template.md =====
# Email Sandboxing Configuration Template (Proofpoint TAP)

## Deployment Info
- **Product**: Proofpoint Email Protection + TAP
- **Deployment Date**: [YYYY-MM-DD]
- **MX Records Updated**: Yes/No
- **SIEM Integration**: [Splunk / Sentinel / QRadar]

## Attachment Sandbox Policy
| File Type | Action | Sandbox Env | Timeout |
|---|---|---|---|
| .exe, .dll, .scr | Detonate + Block | Win10, Win11 | 120s |
| .doc(m), .xls(m), .ppt(m) | Detonate (dynamic delivery) | Win10 + Office | 90s |
| .pdf | Detonate | Win10 + Reader | 60s |
| .zip, .rar, .7z | Extract + Detonate contents | All | 120s |
| .iso, .img | Detonate | Win10 | 120s |
| .js, .vbs, .ps1, .bat | Block (no detonation) | N/A | N/A |

## URL Defense Policy
| Setting | Value |
|---|---|
| URL rewriting | All inbound email |
| Time-of-click analysis | Enabled |
| Block malicious URLs | Yes |
| Suspicious URL interstitial | Enabled |
| Allowed domains bypass | [list internal domains] |

## Monitoring Checklist
- [ ] Daily: Review TAP Dashboard threat digest
- [ ] Daily: Check quarantine for false positives
- [ ] Weekly: Review VAP list
- [ ] Weekly: Analyze threat trends
- [ ] Monthly: Generate executive threat report
- [ ] Quarterly: Policy tuning review

===== END FILE =====


===== FILE: skills/implementing-email-sandboxing-with-proofpoint/references/api-reference.md =====
# API Reference: Implementing Email Sandboxing with Proofpoint

## Proofpoint TAP SIEM API

```python
import requests
resp = requests.get(
    "https://tap-api-v2.proofpoint.com/v2/siem/all",
    auth=(principal, secret),
    params={"sinceSeconds": 3600, "format": "json"})
data = resp.json()
# Keys: messagesDelivered, messagesBlocked, clicksPermitted, clicksBlocked
```

## TAP API Endpoints

| Endpoint | Description |
|----------|-------------|
| `/v2/siem/all` | All threat events |
| `/v2/siem/messages/blocked` | Blocked messages only |
| `/v2/siem/messages/delivered` | Delivered threats |
| `/v2/siem/clicks/blocked` | Blocked URL clicks |
| `/v2/siem/clicks/permitted` | Permitted URL clicks |

## Threat Categories

| Category | Description | Severity |
|----------|-------------|----------|
| Malware | Malicious attachment | CRITICAL |
| Phish | Credential harvesting | HIGH |
| Impostor | BEC/spoofing | HIGH |
| Spam | Unsolicited | LOW |

## URL Defense Configuration

```json
{
  "url_defense": {
    "rewrite_all_urls": true,
    "real_time_scanning": true,
    "sandbox_detonation": true,
    "click_time_protection": true
  }
}
```

## Splunk Integration

```spl
index=proofpoint sourcetype=tap:siem
| where classification="malicious"
| stats count by sender, threatType, subject
```

### References

- Proofpoint TAP API: https://help.proofpoint.com/Threat_Insight_Dashboard/API_Documentation
- Proofpoint Email Protection: https://www.proofpoint.com/us/products/email-security-and-protection

===== END FILE =====


===== FILE: skills/implementing-email-sandboxing-with-proofpoint/references/standards.md =====
# Standards & References: Email Sandboxing with Proofpoint

## MITRE ATT&CK Coverage
- **T1566.001**: Phishing: Spearphishing Attachment (primary detection)
- **T1566.002**: Phishing: Spearphishing Link (URL Defense)
- **T1204.001/002**: User Execution: Malicious Link/File
- **T1059**: Command and Scripting Interpreter (macro detection)
- **T1027**: Obfuscated Files or Information

## NIST Guidelines
- **NIST SP 800-177**: Trustworthy Email - attachment security
- **NIST SP 800-83 Rev.1**: Guide to Malware Incident Prevention
- **NIST SP 800-53**: SI-3 Malicious Code Protection, SI-8 Spam Protection

## Proofpoint TAP API Endpoints
| Endpoint | Description |
|---|---|
| `/v2/siem/all` | All threat events for SIEM |
| `/v2/siem/messages/blocked` | Blocked message events |
| `/v2/siem/messages/delivered` | Delivered message events with threats |
| `/v2/siem/clicks/blocked` | Blocked URL click events |
| `/v2/siem/clicks/permitted` | Permitted URL click events |
| `/v2/people/vap` | Very Attacked People list |
| `/v2/campaign/{id}` | Campaign details |

## Sandbox File Types
| Category | Extensions | Action |
|---|---|---|
| Executables | .exe, .dll, .scr, .com | Detonate + Block |
| Office docs | .doc(x/m), .xls(x/m), .ppt(x/m) | Detonate |
| PDF | .pdf | Detonate |
| Archives | .zip, .rar, .7z, .tar.gz | Extract + Detonate |
| Scripts | .js, .vbs, .ps1, .bat, .cmd | Block |
| Disk images | .iso, .img, .vhd | Detonate |

===== END FILE =====


===== FILE: skills/implementing-email-sandboxing-with-proofpoint/references/workflows.md =====
# Workflows: Email Sandboxing with Proofpoint

## Workflow 1: Attachment Detonation Pipeline
```
Email with attachment arrives at Proofpoint gateway
  |
  v
[Pre-filter: Check attachment type]
  +-- Blocked types (.bat, .ps1, .vbs) --> Quarantine immediately
  +-- Detonable types --> Send to sandbox
  +-- Known safe types (.txt, .csv) --> Deliver
  |
  v
[Sandbox detonation]
  +-- Execute in multiple environments (Win10, Win11, macOS)
  +-- Monitor: file system changes, registry, network, process creation
  +-- Timeout: 60-120 seconds per environment
  |
  v
[Verdict]
  +-- MALICIOUS --> Quarantine, alert, extract IOCs
  +-- SUSPICIOUS --> Quarantine for analyst review
  +-- CLEAN --> Deliver with dynamic delivery
```

## Workflow 2: URL Defense Time-of-Click
```
Email with URL arrives
  |
  v
[URL rewritten to Proofpoint URL Defense proxy]
  |
  v
[Email delivered to user]
  |
  v
[User clicks rewritten URL]
  |
  v
[Proofpoint performs real-time analysis]
  +-- Reputation check
  +-- Content analysis
  +-- Sandbox detonation of landing page
  |
  +-- SAFE --> Redirect to original URL
  +-- MALICIOUS --> Block access, show warning page
  +-- SUSPICIOUS --> Show interstitial warning, allow proceed
```

## Workflow 3: TAP Dashboard Monitoring
```
Daily operations:
  +-- Review TAP Dashboard threat digest
  +-- Check VAP (Very Attacked People) changes
  +-- Review campaign clusters
  +-- Investigate quarantined messages
  +-- Monitor false positive rate
  |
Weekly:
  +-- Analyze threat trends
  +-- Review sandboxing effectiveness
  +-- Tune policies based on FP/FN data
  +-- Update blocked file type list
  |
Monthly:
  +-- Generate executive report from TAP
  +-- Review VAP list with HR/management
  +-- Assess ROI and threat prevention metrics
```

===== END FILE =====


===== FILE: skills/implementing-email-sandboxing-with-proofpoint/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing and monitoring Proofpoint email sandboxing."""

import json
import argparse
from datetime import datetime

try:
    import requests
except ImportError:
    requests = None


def get_tap_threats(base_url, principal, secret, time_range="PT1H"):
    """Query Proofpoint TAP SIEM API for threats."""
    url = f"{base_url}/v2/siem/all"
    resp = requests.get(url, auth=(principal, secret),
                        params={"sinceSeconds": 3600, "format": "json"}, timeout=60)
    resp.raise_for_status()
    data = resp.json()
    return {
        "messages_delivered": len(data.get("messagesDelivered", [])),
        "messages_blocked": len(data.get("messagesBlocked", [])),
        "clicks_permitted": len(data.get("clicksPermitted", [])),
        "clicks_blocked": len(data.get("clicksBlocked", [])),
        "threats": data.get("messagesBlocked", [])[:50],
    }


def analyze_sandbox_results(results_path):
    """Analyze Proofpoint sandbox detonation results."""
    with open(results_path) as f:
        results = json.load(f)
    findings = []
    for result in results if isinstance(results, list) else results.get("results", []):
        verdict = result.get("verdict", result.get("classification", ""))
        score = result.get("score", result.get("threat_score", 0))
        if verdict.lower() in ("malicious", "phish", "spam") or int(score) > 70:
            findings.append({
                "message_id": result.get("message_id", ""),
                "sender": result.get("sender", result.get("from", "")),
                "subject": result.get("subject", ""),
                "verdict": verdict,
                "score": score,
                "threats_found": result.get("threats", []),
                "attachment": result.get("attachment_name", ""),
                "url_detonated": result.get("url", ""),
                "severity": "CRITICAL" if int(score) > 90 else "HIGH",
            })
    return findings


def calculate_email_metrics(log_path):
    """Calculate email security metrics from logs."""
    total = 0
    blocked = 0
    delivered = 0
    by_category = {}
    with open(log_path) as f:
        for line in f:
            try:
                entry = json.loads(line)
            except json.JSONDecodeError:
                continue
            total += 1
            action = entry.get("action", entry.get("policy_action", "")).lower()
            if action in ("block", "quarantine", "reject"):
                blocked += 1
            else:
                delivered += 1
            cat = entry.get("category", entry.get("threat_type", "clean"))
            by_category[cat] = by_category.get(cat, 0) + 1
    return {
        "total_messages": total, "blocked": blocked, "delivered": delivered,
        "block_rate": round(blocked / total * 100, 1) if total else 0,
        "by_category": by_category,
    }


def generate_url_defense_config():
    """Generate Proofpoint URL Defense configuration."""
    return {
        "url_defense": {
            "enabled": True,
            "rewrite_all_urls": True,
            "real_time_scanning": True,
            "sandbox_detonation": True,
            "click_time_protection": True,
        },
        "attachment_defense": {
            "enabled": True,
            "sandbox_analysis": True,
            "supported_types": ["exe", "dll", "doc", "docx", "xls", "xlsx",
                               "pdf", "zip", "rar", "iso", "lnk"],
            "action_on_malicious": "quarantine",
        },
    }


def main():
    parser = argparse.ArgumentParser(description="Proofpoint Email Sandboxing Agent")
    parser.add_argument("--tap-url", default="https://tap-api-v2.proofpoint.com")
    parser.add_argument("--principal", help="TAP API principal")
    parser.add_argument("--secret", help="TAP API secret")
    parser.add_argument("--results", help="Sandbox results JSON")
    parser.add_argument("--log", help="Email log (JSON lines)")
    parser.add_argument("--output", default="proofpoint_sandbox_report.json")
    parser.add_argument("--action", choices=["tap", "analyze", "metrics", "config", "full"],
                        default="full")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "findings": {}}

    if args.action in ("tap", "full") and args.principal and args.secret:
        data = get_tap_threats(args.tap_url, args.principal, args.secret)
        report["findings"]["tap_threats"] = data
        print(f"[+] Blocked: {data['messages_blocked']}, Delivered: {data['messages_delivered']}")

    if args.action in ("analyze", "full") and args.results:
        findings = analyze_sandbox_results(args.results)
        report["findings"]["sandbox_findings"] = findings
        print(f"[+] Malicious sandbox results: {len(findings)}")

    if args.action in ("metrics", "full") and args.log:
        metrics = calculate_email_metrics(args.log)
        report["findings"]["email_metrics"] = metrics
        print(f"[+] Block rate: {metrics['block_rate']}%")

    if args.action in ("config", "full"):
        config = generate_url_defense_config()
        report["findings"]["config"] = config
        print("[+] URL/Attachment Defense config generated")

    with open(args.output, "w") as fout:
        json.dump(report, fout, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-email-sandboxing-with-proofpoint/scripts/process.py =====
#!/usr/bin/env python3
"""
Proofpoint TAP API Integration and Analysis

Pulls threat data from Proofpoint TAP SIEM API, analyzes sandbox results,
identifies Very Attacked People, and generates threat reports.

Usage:
    python process.py threats --hours 24
    python process.py vap
    python process.py campaign --id <campaign-id>
    python process.py report --hours 168 --output report.html
"""

import argparse
import json
import sys
import os
from datetime import datetime, timezone, timedelta
from collections import defaultdict
from dataclasses import dataclass, field, asdict

try:
    import requests
    HAS_REQUESTS = True
except ImportError:
    HAS_REQUESTS = False

PP_SERVICE_PRINCIPAL = os.environ.get("PP_SERVICE_PRINCIPAL", "")
PP_SECRET = os.environ.get("PP_SECRET", "")
PP_BASE_URL = "https://tap-api-v2.proofpoint.com"


class ProofpointTAPClient:
    """Client for Proofpoint TAP SIEM API."""

    def __init__(self, principal: str, secret: str):
        self.auth = (principal, secret)
        self.base = PP_BASE_URL

    def _get(self, endpoint: str, params: dict = None) -> dict:
        resp = requests.get(f"{self.base}{endpoint}",
                            auth=self.auth, params=params, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def get_all_threats(self, since_seconds: int = 3600) -> dict:
        params = {"sinceSeconds": since_seconds, "format": "json"}
        return self._get("/v2/siem/all", params)

    def get_blocked_messages(self, since_seconds: int = 3600) -> dict:
        params = {"sinceSeconds": since_seconds, "format": "json"}
        return self._get("/v2/siem/messages/blocked", params)

    def get_delivered_threats(self, since_seconds: int = 3600) -> dict:
        params = {"sinceSeconds": since_seconds, "format": "json"}
        return self._get("/v2/siem/messages/delivered", params)

    def get_blocked_clicks(self, since_seconds: int = 3600) -> dict:
        params = {"sinceSeconds": since_seconds, "format": "json"}
        return self._get("/v2/siem/clicks/blocked", params)

    def get_permitted_clicks(self, since_seconds: int = 3600) -> dict:
        params = {"sinceSeconds": since_seconds, "format": "json"}
        return self._get("/v2/siem/clicks/permitted", params)

    def get_vap(self, window: int = 14) -> dict:
        params = {"window": window}
        return self._get("/v2/people/vap", params)

    def get_campaign(self, campaign_id: str) -> dict:
        return self._get(f"/v2/campaign/{campaign_id}")


def analyze_threats(threat_data: dict) -> dict:
    """Analyze threat data and produce summary statistics."""
    messages_blocked = threat_data.get("messagesBlocked", [])
    messages_delivered = threat_data.get("messagesDelivered", [])
    clicks_blocked = threat_data.get("clicksBlocked", [])
    clicks_permitted = threat_data.get("clicksPermitted", [])

    # Threat classification counts
    threat_types = defaultdict(int)
    threat_families = defaultdict(int)
    targeted_users = defaultdict(int)
    sender_domains = defaultdict(int)

    all_messages = messages_blocked + messages_delivered
    for msg in all_messages:
        for threat in msg.get("threatsInfoMap", []):
            threat_types[threat.get("classification", "unknown")] += 1
            if threat.get("threatType") == "attachment":
                threat_families[threat.get("threat", "unknown")] += 1

        for recipient in msg.get("recipient", []) if isinstance(msg.get("recipient"), list) else [msg.get("recipient", "")]:
            if recipient:
                targeted_users[recipient] += 1

        sender = msg.get("senderDomain", msg.get("fromAddress", ""))
        if sender:
            sender_domains[sender] += 1

    summary = {
        "total_messages_blocked": len(messages_blocked),
        "total_messages_delivered_with_threats": len(messages_delivered),
        "total_clicks_blocked": len(clicks_blocked),
        "total_clicks_permitted": len(clicks_permitted),
        "threat_type_breakdown": dict(threat_types),
        "top_threat_families": dict(sorted(threat_families.items(),
                                           key=lambda x: x[1], reverse=True)[:10]),
        "top_targeted_users": dict(sorted(targeted_users.items(),
                                          key=lambda x: x[1], reverse=True)[:10]),
        "top_sender_domains": dict(sorted(sender_domains.items(),
                                          key=lambda x: x[1], reverse=True)[:10]),
    }
    return summary


def format_threat_report(summary: dict, hours: int) -> str:
    """Format threat summary as text report."""
    lines = []
    lines.append("=" * 60)
    lines.append("  PROOFPOINT TAP THREAT REPORT")
    lines.append("=" * 60)
    lines.append(f"  Period: Last {hours} hours")
    lines.append(f"  Generated: {datetime.now(timezone.utc).strftime('%Y-%m-%d %H:%M UTC')}")
    lines.append("")

    lines.append("[OVERVIEW]")
    lines.append(f"  Messages Blocked:           {summary['total_messages_blocked']}")
    lines.append(f"  Delivered with Threats:      {summary['total_messages_delivered_with_threats']}")
    lines.append(f"  Clicks Blocked:              {summary['total_clicks_blocked']}")
    lines.append(f"  Clicks Permitted:            {summary['total_clicks_permitted']}")
    lines.append("")

    if summary["threat_type_breakdown"]:
        lines.append("[THREAT TYPES]")
        for t, count in sorted(summary["threat_type_breakdown"].items(),
                                key=lambda x: x[1], reverse=True):
            lines.append(f"  {t}: {count}")
        lines.append("")

    if summary["top_targeted_users"]:
        lines.append("[MOST TARGETED USERS]")
        for user, count in list(summary["top_targeted_users"].items())[:10]:
            lines.append(f"  {user}: {count} threats")
        lines.append("")

    if summary["top_sender_domains"]:
        lines.append("[TOP THREAT SENDER DOMAINS]")
        for domain, count in list(summary["top_sender_domains"].items())[:10]:
            lines.append(f"  {domain}: {count}")

    lines.append("=" * 60)
    return "\n".join(lines)


def main():
    parser = argparse.ArgumentParser(description="Proofpoint TAP Analysis")
    subparsers = parser.add_subparsers(dest="command")

    threats_parser = subparsers.add_parser("threats", help="Get recent threats")
    threats_parser.add_argument("--hours", type=int, default=24)

    vap_parser = subparsers.add_parser("vap", help="Get Very Attacked People")
    vap_parser.add_argument("--window", type=int, default=14, help="Days to look back")

    campaign_parser = subparsers.add_parser("campaign", help="Get campaign details")
    campaign_parser.add_argument("--id", required=True)

    report_parser = subparsers.add_parser("report", help="Generate threat report")
    report_parser.add_argument("--hours", type=int, default=168)
    report_parser.add_argument("--output", "-o")

    parser.add_argument("--json", action="store_true")
    parser.add_argument("--principal", default=PP_SERVICE_PRINCIPAL)
    parser.add_argument("--secret", default=PP_SECRET)

    args = parser.parse_args()

    if not HAS_REQUESTS:
        print("Error: requests library required", file=sys.stderr)
        sys.exit(1)

    principal = args.principal
    secret = args.secret

    if not principal or not secret:
        print("Error: Proofpoint TAP credentials required.", file=sys.stderr)
        print("Set PP_SERVICE_PRINCIPAL and PP_SECRET environment variables.", file=sys.stderr)
        sys.exit(1)

    client = ProofpointTAPClient(principal, secret)

    if args.command == "threats":
        seconds = args.hours * 3600
        data = client.get_all_threats(seconds)
        summary = analyze_threats(data)
        if args.json:
            print(json.dumps(summary, indent=2))
        else:
            print(format_threat_report(summary, args.hours))

    elif args.command == "vap":
        data = client.get_vap(args.window)
        users = data.get("users", [])
        print(f"Very Attacked People (last {args.window} days):")
        for user in users:
            identity = user.get("identity", {})
            print(f"  {identity.get('emails', [''])[0]} - "
                  f"Attacks: {user.get('threatStatistics', {}).get('attackIndex', 0)}")

    elif args.command == "campaign":
        data = client.get_campaign(args.id)
        if args.json:
            print(json.dumps(data, indent=2))
        else:
            print(f"Campaign: {data.get('name', 'Unknown')}")
            print(f"Description: {data.get('description', '')}")
            actors = data.get("actors", [])
            for actor in actors:
                print(f"  Actor: {actor.get('name', 'Unknown')}")

    elif args.command == "report":
        seconds = args.hours * 3600
        data = client.get_all_threats(seconds)
        summary = analyze_threats(data)
        report = format_threat_report(summary, args.hours)
        if args.output:
            with open(args.output, "w") as f:
                f.write(report)
            print(f"Report written to {args.output}")
        else:
            print(report)

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-google-workspace-phishing-protection/SKILL.md =====
---
name: implementing-google-workspace-phishing-protection
description: Configures Google Workspace advanced phishing and malware protection settings in the Admin Console — pre-delivery message scanning, attachment protection, spoofing/impersonation detection, and Enhanced Safe Browsing enforcement. Use when hardening Gmail against phishing, spoofing, and malware, or when tuning Workspace email security policies.
domain: cybersecurity
subdomain: phishing-defense
tags:
- google-workspace
- gmail
- phishing
- email-security
- safe-browsing
- anti-spoofing
- admin-console
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566
- T1598
- T1534
- T1036
- T1027
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - resource-development
  - initial-access
  - stealth
  techniques:
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: T1598
    name: Phishing for Information
    tactic: reconnaissance
    source: attack
  - id: T1672
    name: Email Spoofing
    tactic: stealth
    source: attack
  - id: F1032
    name: Impersonate Official
    tactic: initial-access
    source: f3
  - id: F1031
    name: Impersonate Account Holder
    tactic: initial-access
    source: f3
  - id: F1020.002
    name: 'Create Fake Materials: Fake Website'
    tactic: resource-development
    source: f3
---
# Implementing Google Workspace Phishing Protection

## Overview
Google Workspace provides advanced phishing and malware protection through the Admin Console under Apps > Google Workspace > Gmail > Safety. Key features include Enhanced Pre-Delivery Scanning that examines messages more thoroughly before they reach inboxes, attachment and link protection that scans for malware and checks against known malicious sites, and spoofing detection for domain and employee name impersonation. Google's Advanced Protection Program (APP) provides the strongest account security for high-privilege users.


## When to Use

- When deploying or configuring implementing google workspace phishing protection capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites
- Google Workspace Business Standard or higher license
- Gmail Settings administrator privilege
- Understanding of organizational email flow and third-party integrations
- Access to Google Admin Console (admin.google.com)
- DNS management access for SPF, DKIM, DMARC configuration

## Workflow

### Step 1: Configure Advanced Phishing Protection
- Navigate to Admin Console > Apps > Google Workspace > Gmail > Safety
- Enable "Protect against domain spoofing based on similar domain names"
- Enable "Protect against spoofing of employee names"
- Enable "Protect against inbound emails spoofing your domain"
- Set action for detected spoofing: quarantine or move to spam with warning banner
- Apply settings to all organizational units or specific high-risk groups

### Step 2: Enable Enhanced Pre-Delivery Scanning
- In Safety settings, enable "Enhanced pre-delivery message scanning"
- This adds additional delay (seconds) to scan messages more thoroughly
- Configure to detect phishing attempts that evade initial filters
- Enable "Identify links behind shortened URLs"
- Enable "Scan linked images" for image-based phishing detection

### Step 3: Configure Attachment Protection
- Enable "Protect against encrypted attachments from untrusted senders"
- Enable "Protect against attachments with scripts from untrusted senders"
- Enable "Protect against anomalous attachment types in emails"
- Configure action: warn users, move to spam, or quarantine
- Create exceptions for known legitimate encrypted file senders

### Step 4: Enable Enhanced Safe Browsing
- Navigate to Admin Console > Security > Gmail Enhanced Safe Browsing
- Enable Enhanced Safe Browsing for the organization (off by default)
- This provides real-time protection against phishing URLs in emails
- Configure at organizational unit level for phased rollout
- Monitor user feedback for false positive impact

### Step 5: Enroll High-Risk Users in Advanced Protection Program
- Identify high-privilege accounts: super admins, executives, finance leadership
- Enroll users in Google's Advanced Protection Program (APP)
- APP requires FIDO2 security keys for authentication
- APP blocks third-party app access unless explicitly approved
- APP provides enhanced scanning for Gmail and Drive downloads

### Step 6: Configure Email Authentication
- Publish SPF record: `v=spf1 include:_spf.google.com ~all`
- Enable DKIM signing in Admin Console > Apps > Google Workspace > Gmail > Authenticate email
- Configure DMARC with monitoring: `v=DMARC1; p=none; rua=mailto:dmarc@company.com`
- Progress DMARC to enforcement per organizational readiness

## Tools & Resources
- **Google Admin Console**: Central management for all security settings
- **Google Workspace Security Investigation Tool**: Threat analysis and response
- **Google Security Center**: Security health recommendations and dashboard
- **Gmail Security Sandbox**: Attachment detonation for enterprise licenses
- **Google Advanced Protection Program**: Strongest account security

## Validation
- Spoofing protection blocks test email with lookalike domain
- Pre-delivery scanning catches test phishing with delayed weaponization
- Attachment protection warns on test encrypted attachment
- Enhanced Safe Browsing blocks known phishing URL clicked in email
- APP-enrolled accounts reject non-FIDO2 authentication attempts
- SPF, DKIM, DMARC all pass for outbound messages

===== END FILE =====


===== FILE: skills/implementing-google-workspace-phishing-protection/LICENSE =====

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


===== FILE: skills/implementing-google-workspace-phishing-protection/assets/template.md =====
# Google Workspace Phishing Protection Configuration Template

## Safety Settings Checklist
- [ ] Domain spoofing protection enabled
- [ ] Employee name spoofing protection enabled
- [ ] Enhanced pre-delivery scanning enabled
- [ ] Encrypted attachment protection enabled
- [ ] Script attachment protection enabled
- [ ] Anomalous attachment type protection enabled
- [ ] Enhanced Safe Browsing enabled
- [ ] Security Sandbox enabled (Enterprise)

## Advanced Protection Program Enrollment
| User | Role | APP Enrolled | FIDO2 Key |
|---|---|---|---|
| | Super Admin | | |
| | CEO | | |
| | CFO | | |
| | VP Finance | | |

## Authentication Configuration
| Protocol | Status | Record |
|---|---|---|
| SPF | | v=spf1 include:_spf.google.com -all |
| DKIM | | google._domainkey.{domain} |
| DMARC | | v=DMARC1; p=reject; rua=... |

===== END FILE =====


===== FILE: skills/implementing-google-workspace-phishing-protection/references/api-reference.md =====
# API Reference: Implementing Google Workspace Phishing Protection

## Admin Console Path

```
Admin Console > Apps > Google Workspace > Gmail > Safety
```

## Gmail Safety Controls

| Control | Category | Severity if Missing |
|---------|----------|-------------------|
| Protect against similar domain spoofing | Spoofing | HIGH |
| Protect against employee name spoofing | Spoofing | HIGH |
| Protect against inbound domain spoofing | Spoofing | CRITICAL |
| Enhanced pre-delivery scanning | Scanning | HIGH |
| Attachment protection (encrypted/scripts) | Attachments | HIGH |
| Identify links behind shortened URLs | Links | MEDIUM |
| Show warning for unauthenticated email | Warnings | MEDIUM |
| Enhanced Safe Browsing | Browsing | HIGH |

## Admin SDK Directory API (GAM)

```bash
# List Gmail settings
gam print gmailsettings domain example.com
# List users with Advanced Protection
gam print users query "isAdvancedProtectionProgram=true"
```

## Google Workspace Alert Center API

```python
from google.oauth2 import service_account
from googleapiclient.discovery import build
creds = service_account.Credentials.from_service_account_file("sa.json",
    scopes=["https://www.googleapis.com/auth/apps.alerts"])
service = build("alertcenter", "v1beta1", credentials=creds)
alerts = service.alerts().list().execute()
```

## Recommended Actions for Spoofing

| Detection | Action |
|-----------|--------|
| Similar domain spoofing | Quarantine |
| Employee name spoofing | Show warning + move to spam |
| Inbound domain spoofing | Quarantine |
| Unauthenticated email | Show warning banner |

### References

- Google Workspace Admin Help - Phishing: https://support.google.com/a/answer/9157861
- Advanced Protection Program: https://landing.google.com/advancedprotection/
- Gmail Safety Settings: https://support.google.com/a/answer/7380368

===== END FILE =====


===== FILE: skills/implementing-google-workspace-phishing-protection/references/standards.md =====
# Standards & References: Google Workspace Phishing Protection

## Google Workspace Security Settings Path
- Admin Console > Apps > Google Workspace > Gmail > Safety
- Admin Console > Security > Gmail Enhanced Safe Browsing
- Admin Console > Apps > Google Workspace > Gmail > Authenticate email

## MITRE ATT&CK References
- **T1566.001**: Phishing: Spearphishing Attachment
- **T1566.002**: Phishing: Spearphishing Link
- **T1656**: Impersonation
- **T1586.002**: Compromise Accounts: Email Accounts

## Key Protection Settings
| Setting | Location | Default | Recommended |
|---|---|---|---|
| Domain spoofing protection | Safety | Off | On - Quarantine |
| Employee name spoofing | Safety | Off | On - Warning |
| Pre-delivery scanning | Safety | On | On (enhanced) |
| Attachment protection | Safety | Partial | Full - all options |
| Enhanced Safe Browsing | Security | Off | On |
| Gmail Security Sandbox | Safety | Off | On (Enterprise) |

## Google Workspace License Requirements
| Feature | Business Starter | Business Standard | Enterprise |
|---|---|---|---|
| Basic phishing protection | Yes | Yes | Yes |
| Enhanced pre-delivery scanning | Yes | Yes | Yes |
| Gmail Security Sandbox | No | No | Yes |
| Security Investigation Tool | No | Partial | Yes |
| Advanced Protection Program | Yes | Yes | Yes |

===== END FILE =====


===== FILE: skills/implementing-google-workspace-phishing-protection/references/workflows.md =====
# Workflows: Google Workspace Phishing Protection

## Workflow 1: Gmail Inbound Protection Pipeline

```
Inbound email arrives at Gmail
  |
  v
[Connection-level checks]
  +-- IP reputation
  +-- SPF validation
  +-- DKIM verification
  +-- DMARC evaluation
  |
  v
[Enhanced Pre-Delivery Scanning]
  +-- Content analysis for phishing indicators
  +-- URL expansion (shortened URLs)
  +-- Image scanning for embedded phishing
  +-- NLP analysis for social engineering
  |
  v
[Attachment Protection]
  +-- File type analysis
  +-- Script detection in attachments
  +-- Encrypted attachment from untrusted sender check
  +-- Security Sandbox detonation (Enterprise)
  |
  v
[Spoofing Detection]
  +-- Domain name similarity check
  +-- Employee name impersonation check
  +-- Internal domain spoofing check
  |
  v
[Delivery decision]
  +-- DELIVER: Clean message to inbox
  +-- WARN: Deliver with yellow warning banner
  +-- SPAM: Route to spam folder
  +-- QUARANTINE: Hold for admin review
  +-- REJECT: Block delivery entirely
```

## Workflow 2: Safe Browsing URL Protection

```
User clicks URL in Gmail
  |
  v
[Enhanced Safe Browsing check]
  +-- Real-time URL reputation lookup
  +-- Check against known phishing database
  +-- Dynamic page analysis
  |
  v
[Decision]
  +-- SAFE: Allow navigation
  +-- DANGEROUS: Display full-page warning
  +-- SUSPICIOUS: Display interstitial warning
```

===== END FILE =====


===== FILE: skills/implementing-google-workspace-phishing-protection/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for auditing Google Workspace phishing and malware protection settings."""

import json
import argparse
import subprocess
from datetime import datetime
from collections import Counter


def gam_command(args_list):
    """Run a GAM (Google Apps Manager) command."""
    cmd = ["gam"] + args_list
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    return result.stdout.strip(), result.stderr.strip(), result.returncode


def get_gmail_safety_settings(domain):
    """Retrieve Gmail safety settings via Admin SDK (simulated via GAM)."""
    stdout, stderr, rc = gam_command(["print", "gmailsettings", "domain", domain])
    if rc != 0:
        return {"error": stderr}
    return {"raw_settings": stdout}


def audit_phishing_protection(config_path):
    """Audit Google Workspace phishing protection configuration."""
    with open(config_path) as f:
        config = json.load(f)
    findings = []
    safety = config.get("gmail_safety", config.get("safety_settings", {}))

    checks = {
        "spoofing_similar_domains": {
            "key": "protect_against_similar_domain_spoofing",
            "description": "Protect against domain spoofing (similar names)",
            "severity": "HIGH",
        },
        "employee_name_spoofing": {
            "key": "protect_against_employee_name_spoofing",
            "description": "Protect against employee name spoofing",
            "severity": "HIGH",
        },
        "inbound_domain_spoofing": {
            "key": "protect_against_inbound_domain_spoofing",
            "description": "Protect against inbound domain spoofing",
            "severity": "CRITICAL",
        },
        "enhanced_pre_delivery_scanning": {
            "key": "enhanced_pre_delivery_scanning",
            "description": "Enhanced pre-delivery message scanning",
            "severity": "HIGH",
        },
        "attachment_protection": {
            "key": "attachment_protection_enabled",
            "description": "Attachment protection (encrypted/scripts)",
            "severity": "HIGH",
        },
        "links_and_external_images": {
            "key": "identify_links_behind_shortened_urls",
            "description": "Identify links behind shortened URLs",
            "severity": "MEDIUM",
        },
        "unauthenticated_email_warning": {
            "key": "show_warning_for_unauthenticated_email",
            "description": "Show warning for unauthenticated emails",
            "severity": "MEDIUM",
        },
        "safe_browsing_enhanced": {
            "key": "enhanced_safe_browsing",
            "description": "Enhanced Safe Browsing enabled",
            "severity": "HIGH",
        },
    }

    for check_name, check_info in checks.items():
        enabled = safety.get(check_info["key"], False)
        findings.append({
            "control": check_info["description"],
            "enabled": enabled,
            "status": "COMPLIANT" if enabled else "NON_COMPLIANT",
            "severity": "INFO" if enabled else check_info["severity"],
        })

    quarantine = safety.get("quarantine_action", "")
    if quarantine not in ("quarantine", "reject"):
        findings.append({
            "control": "Quarantine action for detected threats",
            "current": quarantine or "not configured",
            "status": "NON_COMPLIANT",
            "severity": "HIGH",
            "recommendation": "Set action to quarantine or reject",
        })

    return findings


def audit_dmarc_alignment(dmarc_report_path):
    """Analyze DMARC aggregate report for alignment issues."""
    with open(dmarc_report_path) as f:
        report = json.load(f)
    records = report if isinstance(report, list) else report.get("records", [])
    total = len(records)
    aligned = sum(1 for r in records if r.get("dkim_aligned") and r.get("spf_aligned"))
    failures = [r for r in records if not r.get("dkim_aligned") or not r.get("spf_aligned")]
    return {
        "total_records": total,
        "aligned": aligned,
        "alignment_rate": round(aligned / total * 100, 1) if total else 0,
        "top_failures": failures[:10],
    }


def analyze_phishing_incidents(incident_log_path):
    """Analyze phishing incident log for patterns."""
    with open(incident_log_path) as f:
        incidents = json.load(f)
    items = incidents if isinstance(incidents, list) else incidents.get("incidents", [])
    by_type = Counter(i.get("type", "unknown") for i in items)
    by_action = Counter(i.get("action_taken", "unknown") for i in items)
    clicked = sum(1 for i in items if i.get("user_clicked", False))
    reported = sum(1 for i in items if i.get("user_reported", False))
    return {
        "total_incidents": len(items),
        "by_type": dict(by_type),
        "by_action": dict(by_action),
        "click_rate": round(clicked / len(items) * 100, 1) if items else 0,
        "report_rate": round(reported / len(items) * 100, 1) if items else 0,
    }


def generate_recommended_settings():
    """Generate recommended Google Workspace phishing protection settings."""
    return {
        "gmail_safety": {
            "protect_against_similar_domain_spoofing": True,
            "protect_against_employee_name_spoofing": True,
            "protect_against_inbound_domain_spoofing": True,
            "enhanced_pre_delivery_scanning": True,
            "attachment_protection_enabled": True,
            "identify_links_behind_shortened_urls": True,
            "scan_linked_images": True,
            "show_warning_for_unauthenticated_email": True,
            "enhanced_safe_browsing": True,
            "quarantine_action": "quarantine",
            "move_to_spam_on_spoofing": True,
        },
        "advanced_protection_program": {
            "enabled_for_high_privilege_users": True,
            "target_groups": ["executives", "finance", "it-admins"],
        },
    }


def main():
    parser = argparse.ArgumentParser(description="Google Workspace Phishing Protection Agent")
    parser.add_argument("--config", help="Workspace safety config JSON to audit")
    parser.add_argument("--dmarc-report", help="DMARC aggregate report JSON")
    parser.add_argument("--incidents", help="Phishing incident log JSON")
    parser.add_argument("--action", choices=["audit", "dmarc", "incidents",
                                              "recommend", "full"], default="full")
    parser.add_argument("--output", default="gws_phishing_report.json")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "results": {}}

    if args.action in ("audit", "full") and args.config:
        findings = audit_phishing_protection(args.config)
        report["results"]["audit"] = findings
        non_comp = sum(1 for f in findings if f.get("status") == "NON_COMPLIANT")
        print(f"[+] Audit: {len(findings)} controls, {non_comp} non-compliant")

    if args.action in ("dmarc", "full") and args.dmarc_report:
        result = audit_dmarc_alignment(args.dmarc_report)
        report["results"]["dmarc"] = result
        print(f"[+] DMARC alignment: {result['alignment_rate']}%")

    if args.action in ("incidents", "full") and args.incidents:
        result = analyze_phishing_incidents(args.incidents)
        report["results"]["incidents"] = result
        print(f"[+] Incidents: {result['total_incidents']}, click rate: {result['click_rate']}%")

    if args.action in ("recommend", "full"):
        settings = generate_recommended_settings()
        report["results"]["recommended"] = settings
        print("[+] Recommended settings generated")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-google-workspace-phishing-protection/scripts/process.py =====
#!/usr/bin/env python3
"""
Google Workspace Phishing Protection Auditor

Audits Google Workspace Gmail safety settings configuration
and generates compliance recommendations.

Usage:
    python process.py audit --config-file gws_config.json
    python process.py check-auth --domain example.com
"""

import argparse
import json
import sys
from dataclasses import dataclass, field, asdict

try:
    import dns.resolver
    HAS_DNS = True
except ImportError:
    HAS_DNS = False


@dataclass
class GWSSecurityAudit:
    """Google Workspace security configuration audit."""
    domain: str = ""
    spoofing_protection: bool = False
    employee_name_spoofing: bool = False
    enhanced_predelivery: bool = False
    attachment_protection: dict = field(default_factory=dict)
    enhanced_safe_browsing: bool = False
    security_sandbox: bool = False
    app_enrolled_users: int = 0
    spf_configured: bool = False
    dkim_configured: bool = False
    dmarc_configured: bool = False
    dmarc_policy: str = ""
    score: int = 0
    max_score: int = 100
    findings: list = field(default_factory=list)
    recommendations: list = field(default_factory=list)


def audit_gws_config(config: dict) -> GWSSecurityAudit:
    """Audit Google Workspace Gmail safety configuration."""
    audit = GWSSecurityAudit()
    audit.domain = config.get("domain", "")

    safety = config.get("safety_settings", {})

    # Spoofing protection
    audit.spoofing_protection = safety.get("domain_spoofing_protection", False)
    if audit.spoofing_protection:
        audit.score += 15
    else:
        audit.findings.append("Domain spoofing protection not enabled")
        audit.recommendations.append(
            "Enable 'Protect against domain spoofing based on similar domain names'"
        )

    audit.employee_name_spoofing = safety.get("employee_name_spoofing", False)
    if audit.employee_name_spoofing:
        audit.score += 10
    else:
        audit.findings.append("Employee name spoofing protection not enabled")
        audit.recommendations.append(
            "Enable 'Protect against spoofing of employee names'"
        )

    # Pre-delivery scanning
    audit.enhanced_predelivery = safety.get("enhanced_predelivery_scanning", False)
    if audit.enhanced_predelivery:
        audit.score += 15
    else:
        audit.findings.append("Enhanced pre-delivery scanning not enabled")

    # Attachment protection
    att = safety.get("attachment_protection", {})
    audit.attachment_protection = att
    att_score = 0
    if att.get("encrypted_attachments", False):
        att_score += 5
    if att.get("script_attachments", False):
        att_score += 5
    if att.get("anomalous_types", False):
        att_score += 5
    audit.score += att_score
    if att_score < 15:
        audit.findings.append("Not all attachment protection options enabled")

    # Enhanced Safe Browsing
    audit.enhanced_safe_browsing = safety.get("enhanced_safe_browsing", False)
    if audit.enhanced_safe_browsing:
        audit.score += 10
    else:
        audit.findings.append("Enhanced Safe Browsing not enabled (off by default)")
        audit.recommendations.append(
            "Enable Enhanced Safe Browsing in Admin Console > Security"
        )

    # Security Sandbox
    audit.security_sandbox = safety.get("security_sandbox", False)
    if audit.security_sandbox:
        audit.score += 10
    else:
        audit.findings.append("Gmail Security Sandbox not enabled (requires Enterprise license)")

    # Advanced Protection Program
    audit.app_enrolled_users = config.get("app_enrolled_users", 0)
    if audit.app_enrolled_users > 0:
        audit.score += 10
    else:
        audit.recommendations.append(
            "Enroll super admins and executives in Advanced Protection Program"
        )

    # Authentication
    auth = config.get("authentication", {})
    audit.spf_configured = auth.get("spf", False)
    audit.dkim_configured = auth.get("dkim", False)
    audit.dmarc_configured = auth.get("dmarc", False)
    audit.dmarc_policy = auth.get("dmarc_policy", "none")

    if audit.spf_configured:
        audit.score += 5
    if audit.dkim_configured:
        audit.score += 5
    if audit.dmarc_configured:
        audit.score += 5
        if audit.dmarc_policy == "reject":
            audit.score += 10
        elif audit.dmarc_policy == "quarantine":
            audit.score += 5

    return audit


def check_google_auth(domain: str) -> dict:
    """Check SPF, DKIM, DMARC for Google Workspace domain."""
    result = {"domain": domain, "spf": False, "dkim": False, "dmarc": False, "issues": []}

    if not HAS_DNS:
        result["issues"].append("dnspython not installed")
        return result

    # SPF
    try:
        answers = dns.resolver.resolve(domain, 'TXT')
        for rdata in answers:
            txt = str(rdata).strip('"')
            if 'v=spf1' in txt and '_spf.google.com' in txt:
                result["spf"] = True
                break
        if not result["spf"]:
            result["issues"].append("SPF does not include _spf.google.com")
    except Exception:
        result["issues"].append("No SPF record found")

    # DKIM (Google default selector)
    try:
        dns.resolver.resolve(f"google._domainkey.{domain}", 'TXT')
        result["dkim"] = True
    except Exception:
        result["issues"].append("DKIM not configured for 'google' selector")

    # DMARC
    try:
        answers = dns.resolver.resolve(f"_dmarc.{domain}", 'TXT')
        for rdata in answers:
            txt = str(rdata).strip('"')
            if 'v=DMARC1' in txt:
                result["dmarc"] = True
                import re
                policy = re.search(r'p=(\w+)', txt)
                result["dmarc_policy"] = policy.group(1) if policy else "unknown"
                break
    except Exception:
        result["issues"].append("No DMARC record found")

    return result


def main():
    parser = argparse.ArgumentParser(description="Google Workspace Phishing Protection Auditor")
    subparsers = parser.add_subparsers(dest="command")

    audit_parser = subparsers.add_parser("audit", help="Audit GWS security config")
    audit_parser.add_argument("--config-file", required=True)

    auth_parser = subparsers.add_parser("check-auth", help="Check email authentication")
    auth_parser.add_argument("--domain", required=True)

    parser.add_argument("--json", action="store_true")
    args = parser.parse_args()

    if args.command == "audit":
        with open(args.config_file) as f:
            config = json.load(f)
        result = audit_gws_config(config)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(f"Security Score: {result.score}/{result.max_score}")
            if result.findings:
                print(f"\nFindings ({len(result.findings)}):")
                for i, f_item in enumerate(result.findings, 1):
                    print(f"  {i}. {f_item}")
            if result.recommendations:
                print(f"\nRecommendations ({len(result.recommendations)}):")
                for i, rec in enumerate(result.recommendations, 1):
                    print(f"  {i}. {rec}")

    elif args.command == "check-auth":
        result = check_google_auth(args.domain)
        print(json.dumps(result, indent=2))

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-mimecast-targeted-attack-protection/SKILL.md =====
---
name: implementing-mimecast-targeted-attack-protection
description: >-
  Deploys and configures Mimecast Targeted Threat Protection (TTP) modules --
  URL Protect (click-time URL rewriting/analysis), Attachment Protect (sandbox
  detonation), Impersonation Protect (BEC/whaling detection), and Internal Email
  Protect -- for Microsoft 365 or Google Workspace. Use when defending against
  phishing, spearphishing, or business email compromise, or configuring TTP
  policies in the Mimecast Administration Console.
domain: cybersecurity
subdomain: phishing-defense
tags:
- mimecast
- email-security
- targeted-threat-protection
- url-protect
- impersonation
- attachment-sandboxing
- phishing
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566
- T1598
- T1534
- T1036
mitre_f3:
  version: '1.1'
  tactics:
  - initial-access
  - reconnaissance
  - stealth
  - resource-development
  techniques:
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: T1598
    name: Phishing for Information
    tactic: reconnaissance
    source: attack
  - id: T1672
    name: Email Spoofing
    tactic: stealth
    source: attack
  - id: F1032
    name: Impersonate Official
    tactic: initial-access
    source: f3
  - id: F1020.002
    name: 'Create Fake Materials: Fake Website'
    tactic: resource-development
    source: f3
---
# Implementing Mimecast Targeted Attack Protection

## Overview
Mimecast Targeted Threat Protection (TTP) is a suite of advanced email security services designed to protect against sophisticated phishing, spearphishing, and targeted attacks. TTP consists of four core modules: URL Protect (real-time URL rewriting and click-time analysis), Attachment Protect (sandbox detonation of suspicious attachments), Impersonation Protect (BEC and whaling detection), and Internal Email Protect (scanning internal/outbound email for threats). As of November 2025, Mimecast enabled URL Pre-Delivery Action with Hold setting for all customers by default.


## When to Use

- When deploying or configuring implementing mimecast targeted attack protection capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites
- Mimecast Email Security license with TTP add-on
- Administrative access to Mimecast Administration Console
- Microsoft 365 or Google Workspace environment
- MX records configured to route through Mimecast
- Understanding of email authentication (SPF, DKIM, DMARC)

## Key Concepts

### TTP Module Overview
| Module | Function | Key Capability |
|---|---|---|
| URL Protect | Rewrites and scans URLs at click time | Real-time sandbox, pre-delivery hold |
| Attachment Protect | Sandboxes suspicious attachments | Static + dynamic analysis |
| Impersonation Protect | Detects BEC/whaling attacks | VIP name matching, header analysis |
| Internal Email Protect | Scans internal/outbound email | Lateral phishing detection |

### Impersonation Protection Scenarios
- **Hit 3 (Default)**: Flags emails matching 3+ impersonation indicators
- **Hit 1 (VIP)**: Flags emails matching 1+ indicator for designated VIP users
- Key identifiers: display name similarity, domain similarity, reply-to mismatch, newly registered domains

### URL Protect Modes
- **Rewrite**: URLs rewritten to route through Mimecast proxy at click time
- **Pre-Delivery Action (Hold)**: URLs checked before message delivery; held if suspicious
- **Pre-Delivery Action (None)**: URLs checked pre-delivery but not held

## Workflow

### Step 1: Configure URL Protect Policy
- Navigate to Administration > Gateway > Policies > Targeted Threat Protection - URL Protect
- Create URL Protect definition with rewriting enabled for inbound messages
- Enable URL Pre-Delivery Action set to "Hold" for maximum protection
- Configure scan mode: aggressive for high-risk users, moderate for general population
- Set action for malicious URLs: block page with user notification
- Enable URL logging for all click events

### Step 2: Configure Attachment Protect Policy
- Navigate to Administration > Gateway > Policies > Targeted Threat Protection - Attachment Protect
- Create Attachment Protect definition for inbound email
- Select sandbox mode: "Safe File" (converts to safe format) or "Dynamic Configuration" (full sandbox)
- Configure attachment types to scan: executables, Office documents, PDFs, archives
- Set timeout for sandbox analysis (default: up to 7 minutes for complex files)
- Enable pre-emptive sandboxing for attachments from unknown senders

### Step 3: Configure Impersonation Protect
- Create Default Impersonation Protect Definition (Hit 3) for all inbound email
- Create VIP Impersonation Protect Definition (Hit 1) for executive protection
- Build VIP list: CEO, CFO, CTO, board members, finance leadership
- Configure detection identifiers: display name, domain similarity, newly observed sender
- Set actions: quarantine high-confidence impersonation, tag moderate confidence
- Enable end-user warning banners for flagged messages

### Step 4: Enable Internal Email Protect
- Configure journaling from Microsoft 365/Google Workspace to Mimecast
- Enable URL scanning for internal emails
- Enable attachment scanning for internal emails
- Configure alerts for internal account compromise indicators
- Set up detection for internal phishing (compromised account sending malware)

### Step 5: Create Test Group and Validate
- Create pilot group of 50-100 users across departments
- Apply TTP policies to pilot group first
- Send test emails with known-safe test URLs and EICAR test files
- Verify URL rewriting, attachment sandboxing, and impersonation detection
- Monitor false positive rate for 1-2 weeks before organization-wide deployment

### Step 6: Deploy Organization-Wide and Tune
- Extend TTP policies to all users
- Monitor Mimecast Threat Dashboard for detection metrics
- Review and whitelist legitimate applications triggering false positives
- Tune impersonation sensitivity based on false positive feedback
- Configure exception policies for automated systems and mailing lists

## Tools & Resources
- **Mimecast Administration Console**: Policy configuration and management
- **Mimecast Threat Dashboard**: Real-time threat visibility and analytics
- **Mimecast Awareness Training**: Integrated security awareness platform
- **Mimecast API**: Programmatic access to logs and threat data
- **Message Center**: Quarantine management for admins and users

## Validation
- URL Protect rewrites URLs in test messages and blocks known-malicious at click
- Attachment Protect sandboxes test file and returns verdict within SLA
- Impersonation Protect flags test BEC email impersonating VIP
- Internal Email Protect detects test lateral phishing scenario
- Pre-delivery hold catches weaponized URL before reaching inbox
- False positive rate below organizational threshold after tuning

===== END FILE =====


===== FILE: skills/implementing-mimecast-targeted-attack-protection/LICENSE =====

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


===== FILE: skills/implementing-mimecast-targeted-attack-protection/assets/template.md =====
# Mimecast Targeted Threat Protection Deployment Template

## TTP Policy Configuration
| Module | Status | Mode | Scope |
|---|---|---|---|
| URL Protect | | Rewrite + Pre-delivery Hold | All inbound |
| Attachment Protect | | Dynamic sandbox | All inbound |
| Impersonation Protect (Default) | | Hit 3 | All inbound |
| Impersonation Protect (VIP) | | Hit 1 | VIP senders |
| Internal Email Protect | | URL + Attachment scan | Journaled |

## VIP List for Impersonation Protect
| Name | Title | Email | Domain |
|---|---|---|---|
| | CEO | | |
| | CFO | | |
| | CTO | | |
| | VP Finance | | |
| | General Counsel | | |

## Deployment Phases
- [ ] Phase 1: Configure URL Protect (pilot group)
- [ ] Phase 2: Configure Attachment Protect (pilot group)
- [ ] Phase 3: Configure Impersonation Protect with VIP list
- [ ] Phase 4: Enable Internal Email Protect via journaling
- [ ] Phase 5: Roll out to all users
- [ ] Phase 6: Tune policies based on false positive feedback

## Validation Checklist
- [ ] URL Protect rewrites links in test email
- [ ] URL Pre-delivery Hold stops weaponized link
- [ ] Attachment Protect sandboxes EICAR test file
- [ ] Impersonation Protect flags test BEC email
- [ ] Internal Email Protect detects test internal phishing
- [ ] Threat Dashboard shows detection metrics

===== END FILE =====


===== FILE: skills/implementing-mimecast-targeted-attack-protection/references/api-reference.md =====
# API Reference: Implementing Mimecast Targeted Attack Protection

## Mimecast API Authentication

```python
import requests
headers = {"Authorization": "MC access_key:secret_key",
           "x-mc-app-id": "app-id"}
resp = requests.post("https://us-api.mimecast.com/api/ttp/url/get-logs",
                     headers=headers, json={"data": [{"from": "2024-01-01"}]})
```

## TTP API Endpoints

| Endpoint | Description |
|----------|-------------|
| `/api/ttp/url/get-logs` | URL Protection logs |
| `/api/ttp/attachment/get-logs` | Attachment sandbox logs |
| `/api/ttp/impersonation/get-logs` | Impersonation detections |

## URL Protection Actions

| Action | Description |
|--------|-------------|
| allow | URL permitted |
| block | URL blocked (malicious) |
| warn | User warned before click |
| sandbox | Deferred for sandbox analysis |

## Attachment Sandbox Results

| Result | Severity |
|--------|----------|
| safe | INFO |
| suspicious | MEDIUM |
| malicious | CRITICAL |
| sandbox_timeout | HIGH |

## Impersonation Types

| Type | Description |
|------|-------------|
| Internal | Employee name spoofing |
| External | Vendor/partner spoofing |
| Domain | Similar domain detection |

### References

- Mimecast API: https://developer.services.mimecast.com/
- TTP URL Protection: https://developer.services.mimecast.com/docs/threatsintel/1/routes/ttp/url/get-logs/post

===== END FILE =====


===== FILE: skills/implementing-mimecast-targeted-attack-protection/references/standards.md =====
# Standards & References: Implementing Mimecast Targeted Attack Protection

## MITRE ATT&CK References
- **T1566.001**: Phishing: Spearphishing Attachment
- **T1566.002**: Phishing: Spearphishing Link
- **T1534**: Internal Spearphishing
- **T1598**: Phishing for Information
- **T1656**: Impersonation
- **T1204.001**: User Execution: Malicious Link
- **T1204.002**: User Execution: Malicious File

## Mimecast TTP Components
| Component | Version | Key Feature |
|---|---|---|
| URL Protect | Current | Pre-delivery hold (Nov 2025 default) |
| Attachment Protect | Current | Safe File + Dynamic sandbox |
| Impersonation Protect | Current | Hit 1 (VIP) / Hit 3 (Default) |
| Internal Email Protect | Current | Journal-based internal scanning |

## Compliance Alignment
- **CIS Controls v8 Control 9.2**: Use DNS filtering services
- **CIS Controls v8 Control 9.6**: Block unnecessary file types
- **NIST SP 800-177**: Trustworthy Email
- **SOC 2 CC6.1**: Logical and physical access controls
- **ISO 27001 A.8.23**: Web filtering

## Impersonation Protection Indicators
| Indicator | Description | Weight |
|---|---|---|
| Display name match | External sender uses internal VIP display name | High |
| Domain similarity | Sender domain visually similar to internal domain | High |
| Reply-to mismatch | Reply-to differs from From address | Medium |
| Newly observed | First-time sender to recipient | Medium |
| Header anomaly | Unusual mail routing or missing authentication | Medium |

## Integration Points
- Microsoft 365 via connector and journaling
- Google Workspace via gateway routing
- SIEM via syslog or API (Splunk, QRadar, Sentinel)
- SOAR platforms via Mimecast API
- Directory sync via Active Directory / Azure AD

===== END FILE =====


===== FILE: skills/implementing-mimecast-targeted-attack-protection/references/workflows.md =====
# Workflows: Implementing Mimecast Targeted Attack Protection

## Workflow 1: URL Protect Processing

```
Inbound email with URLs arrives
  |
  v
[Pre-delivery URL check]
  +-- Extract all URLs from body and attachments
  +-- Check against Mimecast threat intelligence
  +-- KNOWN MALICIOUS --> Hold/Block message
  +-- SUSPICIOUS --> Hold for deeper analysis
  +-- CLEAN --> Proceed to rewriting
  |
  v
[URL rewriting]
  +-- Replace original URLs with Mimecast proxy URLs
  +-- Preserve original URL in encoded format
  +-- Apply per-policy rewriting rules
  |
  v
[Message delivered to user inbox]
  |
  v
[User clicks rewritten URL]
  |
  v
[Time-of-click analysis]
  +-- Real-time page scan and sandbox
  +-- Check for credential harvesting forms
  +-- Check for malware downloads
  |
  v
[Decision]
  +-- SAFE: Redirect to original URL
  +-- MALICIOUS: Display block page
  +-- SUSPICIOUS: Display warning with proceed option
```

## Workflow 2: Attachment Protect Pipeline

```
Email with attachment arrives
  |
  v
[Static analysis]
  +-- File type identification
  +-- Signature matching
  +-- Known malware hash check
  |
  v
[Policy evaluation]
  +-- Safe File mode: Convert to safe format (PDF)
  +-- Dynamic mode: Full sandbox detonation
  +-- Bypass: Whitelisted sender/type
  |
  v
[Dynamic sandbox (if configured)]
  +-- Execute in isolated environment
  +-- Monitor for malicious behavior
  +-- Check for C2 callbacks, file drops, registry changes
  +-- Timeout: up to 7 minutes
  |
  v
[Verdict]
  +-- CLEAN: Deliver original attachment
  +-- MALICIOUS: Quarantine, notify admin
  +-- TIMEOUT: Deliver with warning or hold per policy
```

## Workflow 3: Impersonation Protect Analysis

```
Inbound email arrives
  |
  v
[Identifier check against VIP list]
  +-- Compare display name to VIP names
  +-- Check domain similarity to internal domains
  +-- Verify reply-to alignment
  +-- Check if sender is newly observed
  |
  v
[Hit scoring]
  +-- Hit 1 policy (VIP): 1+ indicator match
  +-- Hit 3 policy (Default): 3+ indicator matches
  |
  v
[Action based on hit level]
  +-- QUARANTINE: High confidence impersonation
  +-- TAG: Moderate confidence, add warning banner
  +-- LOG: Low confidence, record for analysis
  +-- DELIVER: No indicators matched
```

===== END FILE =====


===== FILE: skills/implementing-mimecast-targeted-attack-protection/scripts/agent.py =====
#!/usr/bin/env python3
"""Mimecast Targeted Attack Protection Agent - monitors TAP events and URL/attachment threats."""

import json
import argparse
import base64
import hashlib
import hmac
import logging
import os
import subprocess
import uuid
from collections import defaultdict
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

MIMECAST_BASE = os.environ.get("MIMECAST_BASE_URL", "https://us-api.mimecast.com")


def mimecast_request(base_url, app_id, app_key, access_key, secret_key, endpoint, data=None):
    """Execute authenticated Mimecast API request."""
    request_id = str(uuid.uuid4())
    hdr_date = datetime.utcnow().strftime("%a, %d %b %Y %H:%M:%S") + " UTC"
    data_to_sign = f"{hdr_date}:{request_id}:{endpoint}:{app_key}"
    hmac_sha1 = hmac.new(base64.b64decode(secret_key), data_to_sign.encode(), hashlib.sha1)
    sig = base64.b64encode(hmac_sha1.digest()).decode()
    headers = {
        "Authorization": f"MC {access_key}:{sig}",
        "x-mc-app-id": app_id,
        "x-mc-date": hdr_date,
        "x-mc-req-id": request_id,
        "Content-Type": "application/json",
    }
    cmd = ["curl", "-s", "-X", "POST", f"{base_url}{endpoint}"]
    for k, v in headers.items():
        cmd.extend(["-H", f"{k}: {v}"])
    if data:
        cmd.extend(["-d", json.dumps({"data": [data]})])
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    return json.loads(result.stdout) if result.stdout else {}


def get_url_logs(base_url, app_id, app_key, access_key, secret_key, date_from=None):
    """Get URL Protection logs."""
    data = {}
    if date_from:
        data["from"] = date_from
    return mimecast_request(base_url, app_id, app_key, access_key, secret_key,
                            "/api/ttp/url/get-logs", data)


def get_impersonation_logs(base_url, app_id, app_key, access_key, secret_key, date_from=None):
    """Get impersonation protection logs."""
    data = {}
    if date_from:
        data["from"] = date_from
    return mimecast_request(base_url, app_id, app_key, access_key, secret_key,
                            "/api/ttp/impersonation/get-logs", data)


def get_attachment_logs(base_url, app_id, app_key, access_key, secret_key, date_from=None):
    """Get attachment protection logs."""
    data = {}
    if date_from:
        data["from"] = date_from
    return mimecast_request(base_url, app_id, app_key, access_key, secret_key,
                            "/api/ttp/attachment/get-logs", data)


def analyze_url_threats(url_logs):
    """Analyze URL click patterns and threat categories."""
    categories = defaultdict(int)
    blocked = 0
    permitted = 0
    users_clicked = defaultdict(int)
    for entry in url_logs:
        cat = entry.get("scanResult", "unknown")
        categories[cat] += 1
        if entry.get("action") == "block":
            blocked += 1
        else:
            permitted += 1
        users_clicked[entry.get("userEmailAddress", "")] += 1
    top_clickers = sorted(users_clicked.items(), key=lambda x: x[1], reverse=True)[:10]
    return {
        "total_url_events": len(url_logs),
        "blocked": blocked,
        "permitted": permitted,
        "categories": dict(categories),
        "top_clickers": [{"email": e, "clicks": c} for e, c in top_clickers],
    }


def analyze_impersonation(imp_logs):
    """Analyze impersonation attack patterns."""
    senders = defaultdict(int)
    targets = defaultdict(int)
    for entry in imp_logs:
        senders[entry.get("senderAddress", "")] += 1
        targets[entry.get("recipientAddress", "")] += 1
    return {
        "total_impersonation_events": len(imp_logs),
        "unique_senders": len(senders),
        "top_impersonated_senders": dict(sorted(senders.items(), key=lambda x: x[1], reverse=True)[:10]),
        "most_targeted_users": dict(sorted(targets.items(), key=lambda x: x[1], reverse=True)[:10]),
    }


def generate_report(url_analysis, imp_analysis, attachment_count):
    return {
        "timestamp": datetime.utcnow().isoformat(),
        "url_protection": url_analysis,
        "impersonation_protection": imp_analysis,
        "attachment_threats": attachment_count,
        "total_threats": url_analysis["blocked"] + imp_analysis["total_impersonation_events"] + attachment_count,
    }


def main():
    parser = argparse.ArgumentParser(description="Mimecast Targeted Attack Protection Agent")
    parser.add_argument("--base-url", default=MIMECAST_BASE)
    parser.add_argument("--app-id", required=True, help="Mimecast Application ID")
    parser.add_argument("--app-key", required=True, help="Mimecast Application Key")
    parser.add_argument("--access-key", required=True, help="Mimecast Access Key")
    parser.add_argument("--secret-key", required=True, help="Mimecast Secret Key")
    parser.add_argument("--date-from", help="Start date YYYY-MM-DD")
    parser.add_argument("--output", default="mimecast_tap_report.json")
    args = parser.parse_args()

    url_resp = get_url_logs(args.base_url, args.app_id, args.app_key, args.access_key, args.secret_key, args.date_from)
    url_logs = url_resp.get("data", [{}])[0].get("clickLogs", [])
    imp_resp = get_impersonation_logs(args.base_url, args.app_id, args.app_key, args.access_key, args.secret_key, args.date_from)
    imp_logs = imp_resp.get("data", [{}])[0].get("impersonationLogs", [])
    att_resp = get_attachment_logs(args.base_url, args.app_id, args.app_key, args.access_key, args.secret_key, args.date_from)
    att_logs = att_resp.get("data", [{}])[0].get("attachmentLogs", [])

    url_analysis = analyze_url_threats(url_logs)
    imp_analysis = analyze_impersonation(imp_logs)
    report = generate_report(url_analysis, imp_analysis, len(att_logs))
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("Total threats: %d (URL blocked: %d, impersonation: %d, attachments: %d)",
                report["total_threats"], url_analysis["blocked"],
                imp_analysis["total_impersonation_events"], len(att_logs))
    print(json.dumps(report, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-mimecast-targeted-attack-protection/scripts/process.py =====
#!/usr/bin/env python3
"""
Mimecast TTP Policy Configuration Auditor

Audits Mimecast Targeted Threat Protection policy configuration
and generates compliance reports. Can also analyze Mimecast
TTP log exports for threat detection metrics.

Usage:
    python process.py audit-config --config-file mimecast_config.json
    python process.py analyze-logs --log-file ttp_logs.json
    python process.py vip-check --vip-file vip_list.json --email-json email.json
"""

import argparse
import json
import re
import sys
from dataclasses import dataclass, field, asdict
from collections import defaultdict, Counter
from datetime import datetime


@dataclass
class PolicyAudit:
    """TTP policy configuration audit result."""
    url_protect_enabled: bool = False
    url_rewriting: bool = False
    url_predelivery_hold: bool = False
    attachment_protect_enabled: bool = False
    attachment_sandbox_mode: str = ""
    impersonation_protect_enabled: bool = False
    impersonation_vip_count: int = 0
    internal_email_protect: bool = False
    findings: list = field(default_factory=list)
    score: int = 0
    max_score: int = 100


@dataclass
class TTPMetrics:
    """TTP detection metrics from log analysis."""
    total_emails: int = 0
    url_threats_blocked: int = 0
    attachment_threats_blocked: int = 0
    impersonation_detected: int = 0
    internal_threats: int = 0
    top_threat_types: dict = field(default_factory=dict)
    top_targeted_users: list = field(default_factory=list)
    daily_stats: dict = field(default_factory=dict)


@dataclass
class ImpersonationCheck:
    """VIP impersonation check result."""
    from_display_name: str = ""
    from_email: str = ""
    matched_vip: str = ""
    indicators: list = field(default_factory=list)
    hit_count: int = 0
    is_impersonation: bool = False
    action: str = ""


REQUIRED_POLICIES = {
    "url_protect": {
        "enabled": True,
        "rewriting": True,
        "predelivery_hold": True,
        "weight": 25
    },
    "attachment_protect": {
        "enabled": True,
        "sandbox_mode": "dynamic",
        "weight": 25
    },
    "impersonation_protect": {
        "enabled": True,
        "min_vips": 5,
        "weight": 25
    },
    "internal_email_protect": {
        "enabled": True,
        "weight": 25
    }
}


def audit_config(config: dict) -> PolicyAudit:
    """Audit Mimecast TTP configuration against best practices."""
    audit = PolicyAudit()

    # Check URL Protect
    url_config = config.get("url_protect", {})
    audit.url_protect_enabled = url_config.get("enabled", False)
    audit.url_rewriting = url_config.get("rewriting", False)
    audit.url_predelivery_hold = url_config.get("predelivery_hold", False)

    if audit.url_protect_enabled:
        audit.score += 10
        if audit.url_rewriting:
            audit.score += 8
        else:
            audit.findings.append("URL rewriting not enabled - URLs not protected at click time")
        if audit.url_predelivery_hold:
            audit.score += 7
        else:
            audit.findings.append(
                "URL Pre-Delivery Hold not enabled - "
                "Mimecast recommends Hold setting (default since Nov 2025)"
            )
    else:
        audit.findings.append("CRITICAL: URL Protect not enabled")

    # Check Attachment Protect
    att_config = config.get("attachment_protect", {})
    audit.attachment_protect_enabled = att_config.get("enabled", False)
    audit.attachment_sandbox_mode = att_config.get("sandbox_mode", "none")

    if audit.attachment_protect_enabled:
        audit.score += 10
        if audit.attachment_sandbox_mode == "dynamic":
            audit.score += 15
        elif audit.attachment_sandbox_mode == "safe_file":
            audit.score += 10
            audit.findings.append(
                "Attachment Protect in Safe File mode - consider Dynamic for full sandbox"
            )
        else:
            audit.findings.append("Attachment sandbox mode not configured")
    else:
        audit.findings.append("CRITICAL: Attachment Protect not enabled")

    # Check Impersonation Protect
    imp_config = config.get("impersonation_protect", {})
    audit.impersonation_protect_enabled = imp_config.get("enabled", False)
    audit.impersonation_vip_count = len(imp_config.get("vip_list", []))

    if audit.impersonation_protect_enabled:
        audit.score += 10
        if audit.impersonation_vip_count >= 5:
            audit.score += 15
        elif audit.impersonation_vip_count > 0:
            audit.score += 8
            audit.findings.append(
                f"Only {audit.impersonation_vip_count} VIPs configured - "
                "recommend adding all executives and finance leadership"
            )
        else:
            audit.findings.append("No VIPs configured for Impersonation Protect")
    else:
        audit.findings.append("CRITICAL: Impersonation Protect not enabled")

    # Check Internal Email Protect
    int_config = config.get("internal_email_protect", {})
    audit.internal_email_protect = int_config.get("enabled", False)

    if audit.internal_email_protect:
        audit.score += 25
    else:
        audit.findings.append(
            "Internal Email Protect not enabled - "
            "lateral phishing from compromised accounts undetected"
        )

    return audit


def analyze_ttp_logs(logs: list) -> TTPMetrics:
    """Analyze Mimecast TTP logs for threat metrics."""
    metrics = TTPMetrics()
    threat_types = Counter()
    targeted_users = Counter()
    daily = defaultdict(lambda: {"total": 0, "threats": 0})

    for entry in logs:
        metrics.total_emails += 1
        date_str = entry.get("date", "unknown")
        daily[date_str]["total"] += 1

        category = entry.get("category", "").lower()
        if category in ("url_threat", "url_blocked"):
            metrics.url_threats_blocked += 1
            daily[date_str]["threats"] += 1
            threat_types["URL Threat"] += 1
        elif category in ("attachment_threat", "attachment_blocked"):
            metrics.attachment_threats_blocked += 1
            daily[date_str]["threats"] += 1
            threat_types["Attachment Threat"] += 1
        elif category in ("impersonation", "impersonation_detected"):
            metrics.impersonation_detected += 1
            daily[date_str]["threats"] += 1
            threat_types["Impersonation"] += 1
        elif category in ("internal_threat",):
            metrics.internal_threats += 1
            daily[date_str]["threats"] += 1
            threat_types["Internal Threat"] += 1

        recipient = entry.get("recipient", "")
        if recipient and category != "clean":
            targeted_users[recipient] += 1

    metrics.top_threat_types = dict(threat_types.most_common(10))
    metrics.top_targeted_users = [
        {"user": user, "threat_count": count}
        for user, count in targeted_users.most_common(10)
    ]
    metrics.daily_stats = dict(daily)

    return metrics


def check_impersonation(email_data: dict, vip_list: list) -> ImpersonationCheck:
    """Check if an email impersonates a VIP."""
    check = ImpersonationCheck()
    check.from_display_name = email_data.get("from_display_name", "")
    check.from_email = email_data.get("from", "")

    from_domain = ""
    match = re.search(r'@([\w.-]+)', check.from_email)
    if match:
        from_domain = match.group(1).lower()

    name_lower = check.from_display_name.lower()

    for vip in vip_list:
        vip_name = vip.get("name", "").lower()
        vip_domain = vip.get("domain", "").lower()

        # Check display name match
        if vip_name and vip_name in name_lower:
            check.indicators.append(f"Display name matches VIP: {vip.get('name')}")
            check.hit_count += 1
            check.matched_vip = vip.get("name", "")

            # Check domain mismatch
            if from_domain and vip_domain and from_domain != vip_domain:
                check.indicators.append(
                    f"Email domain ({from_domain}) differs from VIP domain ({vip_domain})"
                )
                check.hit_count += 1

        # Check domain similarity (lookalike)
        if vip_domain and from_domain:
            if _domain_similarity(from_domain, vip_domain) > 0.8 and from_domain != vip_domain:
                check.indicators.append(
                    f"Domain '{from_domain}' is visually similar to '{vip_domain}'"
                )
                check.hit_count += 1

    # Check reply-to mismatch
    reply_to = email_data.get("reply_to", "")
    if reply_to and reply_to != check.from_email:
        check.indicators.append("Reply-To differs from From address")
        check.hit_count += 1

    # Determine action
    if check.hit_count >= 3:
        check.is_impersonation = True
        check.action = "QUARANTINE (Hit 3 - Default policy)"
    elif check.hit_count >= 1 and check.matched_vip:
        check.is_impersonation = True
        check.action = "QUARANTINE (Hit 1 - VIP policy)"
    else:
        check.action = "DELIVER"

    return check


def _domain_similarity(d1: str, d2: str) -> float:
    """Calculate visual similarity between two domain names."""
    if d1 == d2:
        return 1.0
    longer = max(len(d1), len(d2))
    if longer == 0:
        return 0.0
    matches = sum(1 for a, b in zip(d1, d2) if a == b)
    return matches / longer


def main():
    parser = argparse.ArgumentParser(description="Mimecast TTP Auditor")
    subparsers = parser.add_subparsers(dest="command")

    audit_parser = subparsers.add_parser("audit-config", help="Audit TTP configuration")
    audit_parser.add_argument("--config-file", required=True)

    log_parser = subparsers.add_parser("analyze-logs", help="Analyze TTP logs")
    log_parser.add_argument("--log-file", required=True)

    vip_parser = subparsers.add_parser("vip-check", help="Check for VIP impersonation")
    vip_parser.add_argument("--vip-file", required=True)
    vip_parser.add_argument("--email-json", required=True)

    parser.add_argument("--json", action="store_true")
    args = parser.parse_args()

    if args.command == "audit-config":
        with open(args.config_file) as f:
            config = json.load(f)
        result = audit_config(config)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(f"TTP Configuration Score: {result.score}/{result.max_score}")
            print(f"URL Protect: {'Enabled' if result.url_protect_enabled else 'DISABLED'}")
            print(f"Attachment Protect: {'Enabled' if result.attachment_protect_enabled else 'DISABLED'}")
            print(f"Impersonation Protect: {'Enabled' if result.impersonation_protect_enabled else 'DISABLED'}")
            print(f"Internal Email Protect: {'Enabled' if result.internal_email_protect else 'DISABLED'}")
            if result.findings:
                print(f"\nFindings ({len(result.findings)}):")
                for i, f_item in enumerate(result.findings, 1):
                    print(f"  {i}. {f_item}")

    elif args.command == "analyze-logs":
        with open(args.log_file) as f:
            logs = json.load(f)
        result = analyze_ttp_logs(logs)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(f"Total emails: {result.total_emails}")
            print(f"URL threats: {result.url_threats_blocked}")
            print(f"Attachment threats: {result.attachment_threats_blocked}")
            print(f"Impersonation: {result.impersonation_detected}")
            print(f"Internal threats: {result.internal_threats}")

    elif args.command == "vip-check":
        with open(args.vip_file) as f:
            vip_list = json.load(f)
        with open(args.email_json) as f:
            email_data = json.load(f)
        result = check_impersonation(email_data, vip_list)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(f"From: {result.from_display_name} <{result.from_email}>")
            print(f"Matched VIP: {result.matched_vip or 'None'}")
            print(f"Hit Count: {result.hit_count}")
            print(f"Impersonation: {'YES' if result.is_impersonation else 'No'}")
            print(f"Action: {result.action}")

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-proofpoint-email-security-gateway/SKILL.md =====
---
name: implementing-proofpoint-email-security-gateway
description: Deploy and configure Proofpoint Email Protection as a secure email gateway
  to detect and block phishing, malware, BEC, and spam before messages reach user
  inboxes.
domain: cybersecurity
subdomain: phishing-defense
tags:
- email-security
- proofpoint
- secure-email-gateway
- phishing
- anti-spam
- anti-malware
- bec
- email-filtering
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566
- T1598
- T1534
- T1036
- T1027
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - initial-access
  - stealth
  - positioning
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
  - id: F1032
    name: Impersonate Official
    tactic: initial-access
    source: f3
  - id: F1029
    name: Gather Customer Information
    tactic: reconnaissance
    source: f3
  - id: F1005.006
    name: 'Account Manipulation: Change of Payment Details'
    tactic: positioning
    source: f3
---
# Implementing Proofpoint Email Security Gateway

## Overview
Proofpoint Email Protection is a cloud-native secure email gateway (SEG) that acts as a security checkpoint where all inbound and outbound mail traffic routes through the gateway before reaching user inboxes. It combines signature-based detection for known malware, machine learning algorithms for emerging threats, real-time threat intelligence feeds, URL rewriting with time-of-click sandboxing, and behavioral analysis for BEC detection. Proofpoint processes over 2.8 billion emails daily and blocks over 1 million extortion attempts per day.


## When to Use

- When deploying or configuring implementing proofpoint email security gateway capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites
- Proofpoint Email Protection license (PPS on-premises or Proofpoint on Demand cloud)
- Administrative access to DNS management for MX record changes
- Microsoft 365 or Google Workspace email environment
- Understanding of mail flow architecture and SPF/DKIM/DMARC
- Network firewall rules permitting Proofpoint IP ranges

## Key Concepts

### Deployment Models
1. **MX-Based Gateway (Traditional SEG)**: All mail routes through Proofpoint via MX record changes; intercepts threats before delivery
2. **API-Based Integration**: Connects directly to Microsoft 365 or Google Workspace via API; no MX changes required; can be operational within 48 hours
3. **Hybrid Deployment**: Combines gateway and API for layered protection

### Core Detection Technologies
- **Impostor Classifier**: ML model detecting BEC/impersonation with no malicious URLs or attachments
- **URL Defense**: Rewrites URLs and performs real-time sandboxing at time of click
- **Attachment Defense**: Sandboxes suspicious attachments in virtual environments
- **Nexus Threat Graph**: Cross-customer threat intelligence correlation engine
- **Supplier Threat Detection**: Identifies compromised vendor email accounts

### Protection Layers
| Layer | Technology | Threat Type |
|---|---|---|
| Connection | IP reputation, rate limiting | Spam botnets |
| Authentication | SPF, DKIM, DMARC enforcement | Spoofing |
| Content | ML classifiers, NLP analysis | BEC, phishing |
| URL | Rewriting + time-of-click sandbox | Credential theft |
| Attachment | Static + dynamic sandboxing | Malware, ransomware |
| Post-delivery | TRAP (auto-retraction) | Weaponized after delivery |

## Workflow

### Step 1: Plan Mail Flow Architecture
- Document current MX records and mail flow path
- Identify all legitimate sending sources (marketing platforms, CRM, ticketing systems)
- Map inbound connectors and transport rules in Microsoft 365 or Google Workspace
- Plan IP allowlisting for Proofpoint egress IPs on receiving infrastructure
- Configure SPF record to include Proofpoint: `v=spf1 include:spf.protection.outlook.com include:spf-a.proofpoint.com -all`

### Step 2: Configure Proofpoint Policies
- Create organizational units matching business structure
- Define inbound mail policies: anti-spam, anti-virus, impostor detection
- Configure Smart Search quarantine with end-user digest notifications
- Set up Proofpoint Encryption for sensitive outbound messages
- Enable Targeted Attack Protection (TAP) for URL and attachment sandboxing

### Step 3: Deploy Email Authentication
- Configure DKIM signing through Proofpoint for outbound messages
- Set DMARC policy to monitor mode initially: `v=DMARC1; p=none; rua=mailto:dmarc@company.com`
- Enable inbound DMARC enforcement to reject spoofed messages
- Configure anti-spoofing rules for executive impersonation protection

### Step 4: Enable Advanced Threat Protection
- Activate URL Defense with rewriting enabled for all inbound messages
- Configure Attachment Defense sandbox policies (safe attachment mode)
- Enable Threat Response Auto-Pull (TRAP) for post-delivery remediation
- Set up TAP Dashboard alerts for targeted attack campaigns
- Configure Supplier Risk monitoring for vendor email compromise

### Step 5: Migrate MX Records
- Lower MX record TTL to 300 seconds 48 hours before cutover
- Update MX records to point to Proofpoint: `company-com.mail.protection.proofpoint.com`
- Configure connector restrictions in Microsoft 365 to accept mail only from Proofpoint IPs
- Monitor mail flow through Proofpoint Message Trace for 48-72 hours
- Verify no legitimate mail is being blocked or delayed

### Step 6: Tune and Optimize
- Review quarantine and false positive/negative rates weekly for first month
- Adjust spam thresholds based on organizational tolerance
- Add approved senders and safe lists for legitimate bulk mail
- Configure data loss prevention (DLP) rules for outbound sensitive content
- Enable email warning banners for external sender identification

## Tools & Resources
- **Proofpoint TAP Dashboard**: Real-time threat visibility and campaign tracking
- **Proofpoint TRAP**: Automated post-delivery email retraction
- **Proofpoint SER (Spam/End-user Release)**: Self-service quarantine management
- **Proofpoint Closed-Loop Email Analysis (CLEAR)**: Phishing report button integration
- **MX Toolbox**: DNS record verification and mail flow testing

## Validation
- All inbound email routes through Proofpoint (verify MX records and message headers)
- TAP Dashboard shows threat detections and blocked campaigns
- URL Defense rewrites links in test messages and sandboxes at click time
- Attachment Defense detonates test malware samples in sandbox
- TRAP successfully retracts test phishing message from inboxes post-delivery
- False positive rate below 0.1% after initial tuning period
- DMARC/SPF/DKIM authentication passes for all legitimate outbound mail

===== END FILE =====


===== FILE: skills/implementing-proofpoint-email-security-gateway/LICENSE =====

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


===== FILE: skills/implementing-proofpoint-email-security-gateway/assets/template.md =====
# Proofpoint Email Security Gateway Deployment Template

## Pre-Deployment Checklist
- [ ] Proofpoint license type confirmed (PPS / PoD)
- [ ] Current MX records documented
- [ ] All legitimate sending sources inventoried
- [ ] SPF record updated with Proofpoint include
- [ ] DKIM keys generated and DNS records published
- [ ] DMARC record configured in monitoring mode
- [ ] Firewall rules updated for Proofpoint IP ranges
- [ ] Microsoft 365 / Google Workspace connector configured

## Policy Configuration
| Policy | Scope | Action | Status |
|---|---|---|---|
| Anti-spam (inbound) | All users | Quarantine high confidence | |
| Anti-virus | All users | Block + notify admin | |
| Impostor detection | VIP list | Quarantine + SOC alert | |
| URL Defense | All users | Rewrite + sandbox at click | |
| Attachment Defense | All users | Sandbox suspicious types | |
| TRAP auto-pull | All users | Retract post-delivery threats | |
| DLP (outbound) | All users | Block + manager notify | |

## VIP Protection List
| Name | Title | Email | Protected |
|---|---|---|---|
| | CEO | | Yes |
| | CFO | | Yes |
| | CTO | | Yes |
| | VP Finance | | Yes |

## MX Record Migration
| Record Type | Priority | Old Value | New Value |
|---|---|---|---|
| MX | 10 | | {org}.mail.protection.proofpoint.com |

## Post-Deployment Validation
- [ ] Test inbound mail delivery through Proofpoint
- [ ] Verify message headers show Proofpoint processing
- [ ] Test URL Defense rewriting on inbound links
- [ ] Test Attachment Defense with EICAR test file
- [ ] Verify TRAP can retract delivered message
- [ ] Confirm quarantine digest notifications working
- [ ] Validate SPF/DKIM/DMARC pass for outbound mail
- [ ] Review false positive rate after 48 hours

===== END FILE =====


===== FILE: skills/implementing-proofpoint-email-security-gateway/references/api-reference.md =====
# API Reference: Proofpoint Email Security Gateway

## Libraries Used

| Library | Purpose |
|---------|---------|
| `requests` | HTTP client for Proofpoint TAP API v2 |
| `json` | Parse threat and message event data |
| `os` | Read `PROOFPOINT_SP` and `PROOFPOINT_SECRET` credentials |
| `datetime` | Build ISO-8601 time range queries |

## Installation

```bash
pip install requests
```

## Authentication

Proofpoint TAP API uses HTTP Basic Auth with service principal and secret:

```python
import requests
import os
from requests.auth import HTTPBasicAuth

PROOFPOINT_URL = "https://tap-api-v2.proofpoint.com"
auth = HTTPBasicAuth(
    os.environ["PROOFPOINT_SP"],       # Service Principal
    os.environ["PROOFPOINT_SECRET"],   # Secret
)
```

## TAP API v2 Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/v2/siem/messages/blocked` | Messages blocked by Proofpoint |
| GET | `/v2/siem/messages/delivered` | Messages delivered (with threats) |
| GET | `/v2/siem/clicks/blocked` | Blocked URL clicks |
| GET | `/v2/siem/clicks/permitted` | Permitted URL clicks (with threats) |
| GET | `/v2/siem/all` | All events (messages + clicks) |
| GET | `/v2/siem/issues` | Campaign and threat issues |
| GET | `/v2/people/vap` | Very Attacked People report |
| GET | `/v2/forensics` | Threat forensics detail |
| POST | `/v2/quarantine/release` | Release message from quarantine |
| POST | `/v2/quarantine/delete` | Delete message from quarantine |

## Core Operations

### Fetch Blocked Messages
```python
from datetime import datetime, timedelta

def get_blocked_messages(hours_back=1):
    since = (datetime.utcnow() - timedelta(hours=hours_back)).strftime(
        "%Y-%m-%dT%H:%M:%SZ"
    )
    resp = requests.get(
        f"{PROOFPOINT_URL}/v2/siem/messages/blocked",
        auth=auth,
        params={
            "sinceTime": since,
            "format": "json",
        },
        timeout=60,
    )
    resp.raise_for_status()
    return resp.json().get("messagesBlocked", [])
```

### Fetch Permitted Clicks with Threats
```python
def get_permitted_clicks(hours_back=24):
    since = (datetime.utcnow() - timedelta(hours=hours_back)).strftime(
        "%Y-%m-%dT%H:%M:%SZ"
    )
    resp = requests.get(
        f"{PROOFPOINT_URL}/v2/siem/clicks/permitted",
        auth=auth,
        params={"sinceTime": since, "format": "json"},
        timeout=60,
    )
    resp.raise_for_status()
    return resp.json().get("clicksPermitted", [])
```

### Get All SIEM Events
```python
def get_all_events(hours_back=1):
    since = (datetime.utcnow() - timedelta(hours=hours_back)).strftime(
        "%Y-%m-%dT%H:%M:%SZ"
    )
    resp = requests.get(
        f"{PROOFPOINT_URL}/v2/siem/all",
        auth=auth,
        params={"sinceTime": since, "format": "json"},
        timeout=120,
    )
    resp.raise_for_status()
    data = resp.json()
    return {
        "messages_blocked": data.get("messagesBlocked", []),
        "messages_delivered": data.get("messagesDelivered", []),
        "clicks_blocked": data.get("clicksBlocked", []),
        "clicks_permitted": data.get("clicksPermitted", []),
    }
```

### Get Very Attacked People (VAP)
```python
def get_vap_report(days=30):
    resp = requests.get(
        f"{PROOFPOINT_URL}/v2/people/vap",
        auth=auth,
        params={"window": days, "size": 100},
        timeout=60,
    )
    resp.raise_for_status()
    return resp.json().get("users", [])
```

### Extract Threat IOCs
```python
def extract_iocs(events):
    iocs = {"urls": set(), "senders": set(), "subjects": set(), "sha256": set()}
    for msg in events.get("messages_blocked", []) + events.get("messages_delivered", []):
        iocs["senders"].add(msg.get("sender", ""))
        iocs["subjects"].add(msg.get("subject", ""))
        for threat in msg.get("threatsInfoMap", []):
            if threat.get("threatUrl"):
                iocs["urls"].add(threat["threatUrl"])
            if threat.get("sha256"):
                iocs["sha256"].add(threat["sha256"])
    return {k: list(v) for k, v in iocs.items()}
```

## Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `sinceTime` | ISO-8601 | Start time (required, max 1 hour back for `/all`) |
| `sinceSeconds` | int | Seconds before now (alternative to sinceTime) |
| `format` | string | Response format: `json` (default) or `syslog` |
| `threatType` | string | Filter: `url`, `attachment`, `messageText` |
| `threatStatus` | string | Filter: `active`, `cleared`, `falsePositive` |

## Output Format

```json
{
  "messagesBlocked": [
    {
      "GUID": "abc123-def456",
      "QID": "r1234567",
      "sender": "attacker@malicious.example.com",
      "recipient": ["user@company.com"],
      "subject": "Invoice #12345 Attached",
      "messageTime": "2025-01-15T10:30:00Z",
      "threatsInfoMap": [
        {
          "threat": "https://evil.example.com/payload",
          "threatType": "url",
          "threatStatus": "active",
          "classification": "phish",
          "sha256": "a1b2c3d4e5f6..."
        }
      ],
      "malwareScore": 100,
      "phishScore": 95,
      "spamScore": 0
    }
  ]
}
```

===== END FILE =====


===== FILE: skills/implementing-proofpoint-email-security-gateway/references/standards.md =====
# Standards & References: Implementing Proofpoint Email Security Gateway

## Industry Standards
- **NIST SP 800-177 Rev.1**: Trustworthy Email - guidelines for email security deployment
- **RFC 7208**: Sender Policy Framework (SPF) for authorizing use of domains in email
- **RFC 6376**: DomainKeys Identified Mail (DKIM) Signatures
- **RFC 7489**: Domain-based Message Authentication, Reporting & Conformance (DMARC)
- **CIS Controls v8 Control 9**: Email and Web Browser Protections

## MITRE ATT&CK References
- **T1566.001**: Phishing: Spearphishing Attachment
- **T1566.002**: Phishing: Spearphishing Link
- **T1566.003**: Phishing: Spearphishing via Service
- **T1534**: Internal Spearphishing
- **T1598**: Phishing for Information
- **T1114.003**: Email Collection: Email Forwarding Rule

## Proofpoint-Specific References
- **Proofpoint Email Protection (PPS)**: On-premises protection platform
- **Proofpoint on Demand (PoD)**: Cloud-hosted email security service
- **Proofpoint TAP (Targeted Attack Protection)**: Advanced threat detection
- **Proofpoint TRAP (Threat Response Auto-Pull)**: Post-delivery remediation
- **Proofpoint Nexus Threat Graph**: Cross-customer threat intelligence

## Compliance Alignment
| Framework | Control | Description |
|---|---|---|
| SOC 2 | CC6.1 | Logical and physical access controls |
| HIPAA | 164.312(a)(1) | Access control for ePHI |
| PCI DSS 4.0 | 5.2 | Anti-malware solutions |
| NIST CSF 2.0 | PR.DS-1 | Data-at-rest and data-in-transit protection |
| ISO 27001 | A.8.23 | Web filtering |

## Email Security Gateway Market
- Proofpoint processes 2.8+ billion messages daily (2024)
- Gartner Magic Quadrant Leader for Email Security (2019-2024)
- Over 50% of Fortune 100 companies use Proofpoint

===== END FILE =====


===== FILE: skills/implementing-proofpoint-email-security-gateway/references/workflows.md =====
# Workflows: Implementing Proofpoint Email Security Gateway

## Workflow 1: Inbound Mail Processing Pipeline

```
External sender sends email
  |
  v
[DNS MX lookup resolves to Proofpoint]
  |
  v
[Connection-level filtering]
  +-- IP reputation check (Proofpoint Nexus)
  +-- Rate limiting and connection throttling
  +-- REJECT if known-bad IP
  |
  v
[Authentication checks]
  +-- SPF validation
  +-- DKIM signature verification
  +-- DMARC policy evaluation
  +-- FAIL actions: quarantine or reject per policy
  |
  v
[Content analysis]
  +-- Anti-spam scoring (ML classifier)
  +-- Anti-virus scanning (multi-engine)
  +-- Impostor classifier (BEC detection)
  +-- NLP analysis for social engineering language
  |
  v
[URL Defense]
  +-- Extract all URLs from body and attachments
  +-- Rewrite URLs through Proofpoint proxy
  +-- Pre-delivery URL reputation check
  +-- BLOCK if known malicious
  |
  v
[Attachment Defense]
  +-- Static analysis (signatures, heuristics)
  +-- Dynamic sandbox detonation (if suspicious)
  +-- Wait for sandbox verdict (up to 7 minutes)
  +-- QUARANTINE if malicious
  |
  v
[Policy action]
  +-- DELIVER: Clean email to mailbox
  +-- TAG: Add warning banner for external/suspicious
  +-- QUARANTINE: Hold for admin/user review
  +-- REJECT: Block with NDR to sender
```

## Workflow 2: Post-Delivery Threat Response (TRAP)

```
Threat intelligence update received
  |
  v
[TRAP scans delivered messages retroactively]
  +-- URL becomes malicious after delivery
  +-- New malware signature matches delivered attachment
  |
  v
[Auto-Pull action triggered]
  +-- Move message from user inbox to quarantine
  +-- Log retraction in TRAP dashboard
  +-- Notify SOC team of post-delivery threat
  |
  v
[SOC investigation]
  +-- Review TRAP alert and threat details
  +-- Check if user clicked URL before retraction
  +-- If clicked: initiate incident response
  +-- If not clicked: close as contained
  |
  v
[Update policies]
  +-- Add sender/domain to block list if needed
  +-- Create detection rule for similar campaigns
  +-- Update TAP Dashboard threat tracking
```

## Workflow 3: Phishing Report and CLEAR Integration

```
User receives suspicious email
  |
  v
[User clicks "Report Phishing" button (Proofpoint CLEAR)]
  |
  v
[Email forwarded to Proofpoint analysis pipeline]
  +-- Automated classification (phishing/spam/clean)
  +-- URL and attachment analysis
  |
  v
[CLEAR verdict]
  +-- MALICIOUS: Auto-retract from all inboxes that received it
  +-- SPAM: Move to junk for all recipients
  +-- CLEAN: Return to inbox, thank reporter
  |
  v
[Metrics and feedback]
  +-- Track reporter accuracy rate
  +-- Update user risk score
  +-- Feed into security awareness metrics
```

===== END FILE =====


===== FILE: skills/implementing-proofpoint-email-security-gateway/scripts/agent.py =====
#!/usr/bin/env python3
"""Proofpoint email security gateway audit agent.

Audits Proofpoint TAP (Targeted Attack Protection) via the SIEM API
to retrieve blocked threats, clicked URLs, delivered messages, and
campaign attribution data for email security monitoring.
"""
import argparse
import json
import os
import sys
from datetime import datetime, timezone, timedelta

try:
    import requests
    from requests.auth import HTTPBasicAuth
except ImportError:
    print("[!] 'requests' required: pip install requests", file=sys.stderr)
    sys.exit(1)

PROOFPOINT_BASE = "https://tap-api-v2.proofpoint.com"


def get_pp_config():
    """Return Proofpoint TAP API credentials."""
    principal = os.environ.get("PROOFPOINT_PRINCIPAL", "")
    secret = os.environ.get("PROOFPOINT_SECRET", "")
    if not principal or not secret:
        print("[!] Set PROOFPOINT_PRINCIPAL and PROOFPOINT_SECRET env vars", file=sys.stderr)
        sys.exit(1)
    return principal, secret


def pp_api(endpoint, principal, secret, params=None):
    """Make authenticated Proofpoint TAP API call."""
    url = f"{PROOFPOINT_BASE}{endpoint}"
    resp = requests.get(url, auth=HTTPBasicAuth(principal, secret),
                        params=params, timeout=30)
    resp.raise_for_status()
    return resp.json()


def get_blocked_clicks(principal, secret, hours=24):
    """Get URLs that were blocked when clicked."""
    print(f"[*] Fetching blocked clicks (last {hours}h)...")
    since = (datetime.now(timezone.utc) - timedelta(hours=hours)).strftime("%Y-%m-%dT%H:%M:%SZ")
    data = pp_api("/v2/siem/clicks/blocked", principal, secret,
                  params={"sinceTime": since, "format": "json"})
    clicks = data.get("clicksBlocked", [])
    print(f"[+] {len(clicks)} blocked clicks")
    return clicks


def get_blocked_messages(principal, secret, hours=24):
    """Get messages that were blocked."""
    print(f"[*] Fetching blocked messages (last {hours}h)...")
    since = (datetime.now(timezone.utc) - timedelta(hours=hours)).strftime("%Y-%m-%dT%H:%M:%SZ")
    data = pp_api("/v2/siem/messages/blocked", principal, secret,
                  params={"sinceTime": since, "format": "json"})
    messages = data.get("messagesBlocked", [])
    print(f"[+] {len(messages)} blocked messages")
    return messages


def get_delivered_threats(principal, secret, hours=24):
    """Get threats that were delivered (missed by filters)."""
    print(f"[*] Fetching delivered threats (last {hours}h)...")
    since = (datetime.now(timezone.utc) - timedelta(hours=hours)).strftime("%Y-%m-%dT%H:%M:%SZ")
    data = pp_api("/v2/siem/messages/delivered", principal, secret,
                  params={"sinceTime": since, "format": "json"})
    messages = data.get("messagesDelivered", [])
    threats = [m for m in messages if m.get("threatsInfoMap")]
    print(f"[+] {len(messages)} delivered, {len(threats)} with threats")
    return threats


def get_permitted_clicks(principal, secret, hours=24):
    """Get URLs that were permitted when clicked (potential misses)."""
    print(f"[*] Fetching permitted clicks (last {hours}h)...")
    since = (datetime.now(timezone.utc) - timedelta(hours=hours)).strftime("%Y-%m-%dT%H:%M:%SZ")
    data = pp_api("/v2/siem/clicks/permitted", principal, secret,
                  params={"sinceTime": since, "format": "json"})
    clicks = data.get("clicksPermitted", [])
    print(f"[+] {len(clicks)} permitted clicks")
    return clicks


def analyze_threats(blocked_msgs, delivered_threats, blocked_clicks, permitted_clicks):
    """Analyze threat data for security insights."""
    findings = []

    # Delivered threats are highest priority
    for msg in delivered_threats:
        for threat_info in msg.get("threatsInfoMap", []):
            findings.append({
                "type": "delivered_threat",
                "severity": "CRITICAL",
                "threat_type": threat_info.get("threatType", ""),
                "classification": threat_info.get("classification", ""),
                "threat_url": threat_info.get("threat", "")[:100],
                "recipient": msg.get("recipient", [""])[0] if msg.get("recipient") else "",
                "sender": msg.get("sender", ""),
                "subject": msg.get("subject", "")[:80],
                "timestamp": msg.get("messageTime", ""),
            })

    # Summarize blocked activity
    threat_types = {}
    for msg in blocked_msgs:
        for t in msg.get("threatsInfoMap", []):
            tt = t.get("threatType", "unknown")
            threat_types[tt] = threat_types.get(tt, 0) + 1

    if threat_types:
        findings.append({
            "type": "blocked_summary",
            "severity": "INFO",
            "detail": f"Blocked threats by type: {json.dumps(threat_types)}",
            "total_blocked": len(blocked_msgs),
        })

    # Permitted clicks on potentially malicious URLs
    for click in permitted_clicks:
        if click.get("threatStatus") == "active":
            findings.append({
                "type": "permitted_malicious_click",
                "severity": "HIGH",
                "url": click.get("url", "")[:100],
                "user": click.get("recipient", [""])[0] if click.get("recipient") else "",
                "click_time": click.get("clickTime", ""),
            })

    return findings


def format_summary(findings, blocked_msgs, delivered, blocked_clicks, permitted_clicks):
    """Print email security summary."""
    print(f"\n{'='*60}")
    print(f"  Proofpoint Email Security Report")
    print(f"{'='*60}")
    print(f"  Blocked Messages  : {len(blocked_msgs)}")
    print(f"  Delivered Threats  : {len(delivered)}")
    print(f"  Blocked Clicks     : {len(blocked_clicks)}")
    print(f"  Permitted Clicks   : {len(permitted_clicks)}")
    print(f"  Security Findings  : {len(findings)}")

    critical = [f for f in findings if f["severity"] == "CRITICAL"]
    if critical:
        print(f"\n  CRITICAL - Threats That Bypassed Filters ({len(critical)}):")
        for f in critical[:10]:
            print(f"    {f.get('threat_type', 'N/A'):15s} | "
                  f"To: {f.get('recipient', 'N/A'):30s} | "
                  f"{f.get('subject', '')[:40]}")

    severity_counts = {}
    for f in findings:
        sev = f.get("severity", "INFO")
        severity_counts[sev] = severity_counts.get(sev, 0) + 1
    return severity_counts


def main():
    parser = argparse.ArgumentParser(description="Proofpoint email security audit agent")
    parser.add_argument("--principal", help="TAP API principal (or PROOFPOINT_PRINCIPAL env)")
    parser.add_argument("--secret", help="TAP API secret (or PROOFPOINT_SECRET env)")
    parser.add_argument("--hours", type=int, default=24, help="Hours to look back (default: 24)")
    parser.add_argument("--output", "-o", help="Output JSON report")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    if args.principal:
        os.environ["PROOFPOINT_PRINCIPAL"] = args.principal
    if args.secret:
        os.environ["PROOFPOINT_SECRET"] = args.secret

    principal, secret = get_pp_config()

    blocked_msgs = get_blocked_messages(principal, secret, args.hours)
    delivered = get_delivered_threats(principal, secret, args.hours)
    blocked_clicks = get_blocked_clicks(principal, secret, args.hours)
    permitted_clicks = get_permitted_clicks(principal, secret, args.hours)

    findings = analyze_threats(blocked_msgs, delivered, blocked_clicks, permitted_clicks)
    severity_counts = format_summary(findings, blocked_msgs, delivered, blocked_clicks, permitted_clicks)

    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "Proofpoint TAP",
        "period_hours": args.hours,
        "blocked_messages": len(blocked_msgs),
        "delivered_threats": len(delivered),
        "findings": findings,
        "severity_counts": severity_counts,
        "risk_level": (
            "CRITICAL" if severity_counts.get("CRITICAL", 0) > 0
            else "HIGH" if severity_counts.get("HIGH", 0) > 0
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


===== FILE: skills/implementing-proofpoint-email-security-gateway/scripts/process.py =====
#!/usr/bin/env python3
"""
Proofpoint Email Security Gateway Configuration Validator

Validates MX records, SPF/DKIM/DMARC alignment, and Proofpoint
mail flow configuration for a given domain. Uses DNS lookups to
verify that email is routing through Proofpoint correctly.

Usage:
    python process.py check-mx --domain example.com
    python process.py check-auth --domain example.com
    python process.py validate-headers --eml-file message.eml
"""

import argparse
import json
import re
import sys
from dataclasses import dataclass, field, asdict
from typing import Optional

try:
    import dns.resolver
    HAS_DNS = True
except ImportError:
    HAS_DNS = False


@dataclass
class MXCheckResult:
    """MX record validation result."""
    domain: str = ""
    mx_records: list = field(default_factory=list)
    routes_through_proofpoint: bool = False
    proofpoint_mx: str = ""
    issues: list = field(default_factory=list)


@dataclass
class AuthCheckResult:
    """Email authentication check result."""
    domain: str = ""
    spf_record: str = ""
    spf_includes_proofpoint: bool = False
    dmarc_record: str = ""
    dmarc_policy: str = ""
    dkim_selector: str = ""
    issues: list = field(default_factory=list)


@dataclass
class HeaderAnalysis:
    """Email header forensic analysis."""
    from_address: str = ""
    return_path: str = ""
    received_chain: list = field(default_factory=list)
    spf_result: str = ""
    dkim_result: str = ""
    dmarc_result: str = ""
    proofpoint_headers: list = field(default_factory=list)
    x_proofpoint_spam_details: str = ""
    x_proofpoint_virus_version: str = ""
    passed_through_proofpoint: bool = False
    issues: list = field(default_factory=list)


PROOFPOINT_MX_PATTERNS = [
    r'\.pphosted\.com$',
    r'\.proofpoint\.com$',
    r'mail\.protection\.proofpoint\.com$',
]

PROOFPOINT_SPF_INCLUDES = [
    'spf-a.proofpoint.com',
    'spf-b.proofpoint.com',
    'spf.proofpoint.com',
    'pphosted.com',
]

PROOFPOINT_HEADER_MARKERS = [
    'X-Proofpoint-Spam-Details',
    'X-Proofpoint-Virus-Version',
    'X-Proofpoint-GUID',
    'X-Proofpoint-ORIG-GUID',
]


def check_mx_records(domain: str) -> MXCheckResult:
    """Check if domain MX records route through Proofpoint."""
    result = MXCheckResult(domain=domain)

    if not HAS_DNS:
        result.issues.append("dnspython not installed. Install with: pip install dnspython")
        return result

    try:
        answers = dns.resolver.resolve(domain, 'MX')
        for rdata in sorted(answers, key=lambda r: r.preference):
            mx_host = str(rdata.exchange).rstrip('.')
            result.mx_records.append({
                "priority": rdata.preference,
                "host": mx_host
            })
            for pattern in PROOFPOINT_MX_PATTERNS:
                if re.search(pattern, mx_host, re.IGNORECASE):
                    result.routes_through_proofpoint = True
                    result.proofpoint_mx = mx_host
                    break
    except dns.resolver.NXDOMAIN:
        result.issues.append(f"Domain {domain} does not exist")
    except dns.resolver.NoAnswer:
        result.issues.append(f"No MX records found for {domain}")
    except Exception as e:
        result.issues.append(f"DNS query failed: {str(e)}")

    if not result.routes_through_proofpoint and result.mx_records:
        result.issues.append(
            "MX records do not point to Proofpoint. "
            "Expected pattern: *.pphosted.com or *.proofpoint.com"
        )

    return result


def check_authentication(domain: str) -> AuthCheckResult:
    """Check SPF, DKIM, and DMARC records for Proofpoint alignment."""
    result = AuthCheckResult(domain=domain)

    if not HAS_DNS:
        result.issues.append("dnspython not installed. Install with: pip install dnspython")
        return result

    # Check SPF
    try:
        answers = dns.resolver.resolve(domain, 'TXT')
        for rdata in answers:
            txt = str(rdata).strip('"')
            if txt.startswith('v=spf1'):
                result.spf_record = txt
                for include in PROOFPOINT_SPF_INCLUDES:
                    if include in txt:
                        result.spf_includes_proofpoint = True
                        break
                break
    except Exception:
        result.issues.append("Could not retrieve SPF record")

    if result.spf_record and not result.spf_includes_proofpoint:
        result.issues.append(
            "SPF record does not include Proofpoint. "
            "Add: include:spf-a.proofpoint.com"
        )

    # Check DMARC
    try:
        dmarc_domain = f"_dmarc.{domain}"
        answers = dns.resolver.resolve(dmarc_domain, 'TXT')
        for rdata in answers:
            txt = str(rdata).strip('"')
            if txt.startswith('v=DMARC1'):
                result.dmarc_record = txt
                policy_match = re.search(r'p=(\w+)', txt)
                if policy_match:
                    result.dmarc_policy = policy_match.group(1)
                break
    except Exception:
        result.issues.append("No DMARC record found")

    if result.dmarc_policy == "none":
        result.issues.append(
            "DMARC policy is set to 'none' (monitoring only). "
            "Plan rollout to 'quarantine' then 'reject'"
        )

    # Check Proofpoint DKIM selector
    try:
        dkim_domain = f"proofpoint._domainkey.{domain}"
        answers = dns.resolver.resolve(dkim_domain, 'TXT')
        for rdata in answers:
            result.dkim_selector = "proofpoint"
            break
    except Exception:
        pass

    return result


def analyze_headers(eml_content: str) -> HeaderAnalysis:
    """Analyze email headers for Proofpoint routing and authentication."""
    analysis = HeaderAnalysis()

    # Extract From header
    from_match = re.search(r'^From:\s*(.+)$', eml_content, re.MULTILINE | re.IGNORECASE)
    if from_match:
        analysis.from_address = from_match.group(1).strip()

    # Extract Return-Path
    rp_match = re.search(r'^Return-Path:\s*(.+)$', eml_content, re.MULTILINE | re.IGNORECASE)
    if rp_match:
        analysis.return_path = rp_match.group(1).strip()

    # Extract Received chain
    received_headers = re.findall(
        r'^Received:\s*(.*?)(?=\n\S|\nReceived:|\n\n)',
        eml_content, re.MULTILINE | re.DOTALL | re.IGNORECASE
    )
    for hdr in received_headers:
        clean = ' '.join(hdr.split())
        analysis.received_chain.append(clean)
        if any(p.replace(r'$', '').replace(r'\.', '.') in clean.lower()
               for p in ['pphosted.com', 'proofpoint.com']):
            analysis.passed_through_proofpoint = True

    # Extract Authentication-Results
    auth_match = re.search(
        r'^Authentication-Results:\s*(.*?)(?=\n\S)',
        eml_content, re.MULTILINE | re.DOTALL | re.IGNORECASE
    )
    if auth_match:
        auth_text = auth_match.group(1)
        spf_match = re.search(r'spf=(\w+)', auth_text)
        if spf_match:
            analysis.spf_result = spf_match.group(1)
        dkim_match = re.search(r'dkim=(\w+)', auth_text)
        if dkim_match:
            analysis.dkim_result = dkim_match.group(1)
        dmarc_match = re.search(r'dmarc=(\w+)', auth_text)
        if dmarc_match:
            analysis.dmarc_result = dmarc_match.group(1)

    # Check for Proofpoint-specific headers
    for marker in PROOFPOINT_HEADER_MARKERS:
        marker_match = re.search(
            rf'^{re.escape(marker)}:\s*(.+)$',
            eml_content, re.MULTILINE | re.IGNORECASE
        )
        if marker_match:
            analysis.proofpoint_headers.append({
                "header": marker,
                "value": marker_match.group(1).strip()
            })
            if marker == 'X-Proofpoint-Spam-Details':
                analysis.x_proofpoint_spam_details = marker_match.group(1).strip()
            elif marker == 'X-Proofpoint-Virus-Version':
                analysis.x_proofpoint_virus_version = marker_match.group(1).strip()

    if not analysis.passed_through_proofpoint and not analysis.proofpoint_headers:
        analysis.issues.append("Email does not appear to have routed through Proofpoint")

    if analysis.spf_result and analysis.spf_result != 'pass':
        analysis.issues.append(f"SPF check failed: {analysis.spf_result}")
    if analysis.dkim_result and analysis.dkim_result != 'pass':
        analysis.issues.append(f"DKIM check failed: {analysis.dkim_result}")
    if analysis.dmarc_result and analysis.dmarc_result != 'pass':
        analysis.issues.append(f"DMARC check failed: {analysis.dmarc_result}")

    return analysis


def format_report(title: str, data: dict) -> str:
    """Format check results as a readable report."""
    lines = []
    lines.append("=" * 60)
    lines.append(f"  {title}")
    lines.append("=" * 60)

    for key, value in data.items():
        if key == 'issues':
            if value:
                lines.append(f"\n  [ISSUES]")
                for i, issue in enumerate(value, 1):
                    lines.append(f"    {i}. {issue}")
        elif isinstance(value, list):
            lines.append(f"\n  {key}:")
            for item in value:
                if isinstance(item, dict):
                    lines.append(f"    - {json.dumps(item)}")
                else:
                    lines.append(f"    - {item}")
        elif isinstance(value, bool):
            status = "YES" if value else "NO"
            lines.append(f"  {key}: {status}")
        else:
            lines.append(f"  {key}: {value}")

    lines.append("=" * 60)
    return "\n".join(lines)


def main():
    parser = argparse.ArgumentParser(description="Proofpoint Email Gateway Validator")
    subparsers = parser.add_subparsers(dest="command")

    mx_parser = subparsers.add_parser("check-mx", help="Check MX records for Proofpoint routing")
    mx_parser.add_argument("--domain", required=True, help="Domain to check")

    auth_parser = subparsers.add_parser("check-auth", help="Check email authentication records")
    auth_parser.add_argument("--domain", required=True, help="Domain to check")

    hdr_parser = subparsers.add_parser("validate-headers", help="Analyze email headers")
    hdr_parser.add_argument("--eml-file", required=True, help="Path to .eml file")

    parser.add_argument("--json", action="store_true", help="Output as JSON")

    args = parser.parse_args()

    if args.command == "check-mx":
        result = check_mx_records(args.domain)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(format_report("PROOFPOINT MX RECORD CHECK", asdict(result)))

    elif args.command == "check-auth":
        result = check_authentication(args.domain)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(format_report("EMAIL AUTHENTICATION CHECK", asdict(result)))

    elif args.command == "validate-headers":
        with open(args.eml_file, 'r', errors='replace') as f:
            content = f.read()
        result = analyze_headers(content)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(format_report("EMAIL HEADER ANALYSIS", asdict(result)))

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-adversary-in-the-middle-phishing-detection/SKILL.md =====
---
name: performing-adversary-in-the-middle-phishing-detection
description: Detect and respond to Adversary-in-the-Middle (AiTM) phishing attacks
  that use reverse proxy kits like EvilProxy, Evilginx, and Tycoon 2FA to bypass MFA
  and steal session tokens, correlating Azure AD/Entra sign-in logs, SIEM alerts,
  and EDR telemetry. Use when investigating suspected MFA-bypass phishing or session
  token theft, or building detection and response playbooks against reverse-proxy
  phishing kits.
domain: cybersecurity
subdomain: phishing-defense
tags:
- aitm
- evilproxy
- evilginx
- phishing
- mfa-bypass
- session-hijacking
- reverse-proxy
- credential-theft
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566
- T1598
- T1534
- T1036
- T1003
mitre_f3:
  version: '1.1'
  tactics:
  - initial-access
  - positioning
  techniques:
  - id: T1557
    name: Adversary-in-the-Middle
    tactic: initial-access
    source: attack
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: F1004
    name: Access with Stolen Session Cookie
    tactic: initial-access
    source: f3
  - id: T1539
    name: Steal Web Session Cookie
    tactic: positioning
    source: attack
  - id: T1185
    name: Browser Session Hijacking
    tactic: positioning
    source: attack
  - id: F1006
    name: Account Takeover
    tactic: initial-access
    source: f3
---
# Performing Adversary-in-the-Middle Phishing Detection

## Overview
Adversary-in-the-Middle (AiTM) phishing attacks use reverse-proxy infrastructure to sit between the victim and the legitimate authentication service, intercepting both credentials and session cookies in real time. This allows attackers to bypass multi-factor authentication (MFA). The most prevalent PhaaS kits in 2025 include Tycoon 2FA, Sneaky 2FA, EvilProxy, and Evilginx. Over 1 million PhaaS attacks were detected in January-February 2025 alone. These attacks have evolved from QR codes to HTML attachments and SVG files for link distribution.


## When to Use

- When conducting security assessments that involve performing adversary in the middle phishing detection
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites
- Azure AD / Entra ID Conditional Access policies
- SIEM with authentication log ingestion (Azure AD sign-in logs)
- Web proxy with SSL inspection and URL categorization
- Endpoint Detection and Response (EDR) solution
- FIDO2/phishing-resistant MFA capability

## Key Concepts

### How AiTM Works
1. Victim receives phishing email with link to attacker-controlled domain
2. Attacker domain runs reverse proxy that mirrors legitimate login page
3. Victim enters credentials on proxied page; credentials captured in transit
4. Reverse proxy forwards credentials to real authentication service
5. MFA challenge sent to victim; victim completes MFA on proxied page
6. Attacker captures session cookie returned by legitimate service
7. Attacker replays session cookie to access victim's account without MFA

### Major AiTM Kits (2025)
| Kit | Type | Primary Targets | Evasion |
|---|---|---|---|
| Tycoon 2FA | PhaaS | Microsoft 365, Google | CAPTCHA, Cloudflare turnstile |
| EvilProxy | PhaaS | Microsoft 365, Google, Okta | Random URLs, IP rotation |
| Evilginx | Open-source | Any web application | Custom phishlets |
| Sneaky 2FA | PhaaS | Microsoft 365 | Anti-bot checks |
| NakedPages | PhaaS | Multiple | Minimal infrastructure |

### Detection Indicators
- Authentication from unusual IP not matching user profile
- Session cookie reuse from different IP/device than authentication
- Login page served from non-Microsoft/non-Google infrastructure
- CDN requests to legitimate auth providers from phishing domains
- Impossible travel between authentication and session usage

## Workflow

### Step 1: Deploy Phishing-Resistant MFA
- Implement FIDO2 security keys or Windows Hello for Business for high-value accounts
- Configure Conditional Access to require phishing-resistant MFA for admins
- Enable certificate-based authentication where possible
- Disable SMS and voice MFA for privileged accounts
- AiTM cannot intercept FIDO2 because authentication is bound to origin domain

### Step 2: Configure Conditional Access Policies
- Require compliant/managed device for sensitive application access
- Block authentication from anonymous proxies and Tor exit nodes
- Enforce token binding to limit session cookie replay
- Configure continuous access evaluation (CAE) for real-time token revocation
- Implement sign-in risk policies that require re-authentication for risky sign-ins

### Step 3: Build AiTM Detection Rules
- Alert on sign-in followed by session from different IP within 10 minutes
- Detect authentication where proxy IP does not match user's expected location
- Monitor for impossible travel patterns in session usage
- Alert on inbox rules created immediately after authentication (common post-compromise)
- Detect new MFA method registration from suspicious sign-in

### Step 4: Monitor Web Proxy for AiTM Infrastructure
- Log and analyze DNS queries to newly registered domains
- Detect connections to known PhaaS infrastructure IPs
- Alert on authentication page backgrounds loaded from legitimate CDNs through proxy domains
- Monitor for SSL certificates issued to domains mimicking corporate login pages
- Block access to known EvilProxy/Evilginx infrastructure via threat intelligence

### Step 5: Implement Post-Compromise Detection
- Alert on mailbox forwarding rules created after suspicious authentication
- Detect OAuth app consent after AiTM sign-in
- Monitor for email sending patterns indicating BEC follow-up
- Alert on SharePoint/OneDrive mass download after session hijack
- Track lateral movement from compromised account

## Tools & Resources
- **Microsoft Entra ID Protection**: Risk-based Conditional Access
- **Azure AD Sign-in Logs**: Authentication event analysis
- **Okta ThreatInsight**: AiTM proxy detection at IdP level
- **Sekoia TDR**: AiTM campaign tracking and intelligence
- **Evilginx (defensive)**: Understanding attack mechanics for detection

## Validation
- Phishing-resistant MFA blocks AiTM session capture in test scenario
- Conditional Access denies session replay from different device/IP
- SIEM alerts fire on simulated AiTM sign-in patterns
- Web proxy blocks connections to known PhaaS infrastructure
- Post-compromise rules detect inbox rule creation after suspicious auth

===== END FILE =====


===== FILE: skills/performing-adversary-in-the-middle-phishing-detection/LICENSE =====

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


===== FILE: skills/performing-adversary-in-the-middle-phishing-detection/assets/template.md =====
# AiTM Phishing Detection Template

## Phishing-Resistant MFA Deployment
| User Group | MFA Method | AiTM Resistant | Status |
|---|---|---|---|
| Global Admins | FIDO2 Security Key | Yes | |
| Privileged Admins | Windows Hello for Business | Yes | |
| Finance/HR | FIDO2 Security Key | Yes | |
| All Users | Microsoft Authenticator (number match) | Partial | |

## Conditional Access Policies for AiTM Prevention
| Policy | Condition | Action | Status |
|---|---|---|---|
| Require managed device | All cloud apps | Block if unmanaged | |
| Block anonymous proxy | Sign-in risk | Block | |
| Require phishing-resistant MFA | Admin roles | Enforce FIDO2 | |
| Token binding | Sensitive apps | Bind to device | |
| Continuous access evaluation | Exchange/SharePoint | Enable CAE | |

## AiTM Detection Rules
| Rule | Data Source | Alert Priority |
|---|---|---|
| Session IP mismatch within 10min | Azure AD sign-in logs | Critical |
| Impossible travel | Azure AD sign-in logs | High |
| Inbox rule creation post-auth | Exchange audit logs | High |
| OAuth consent post-risky-sign-in | Azure AD audit logs | High |
| New MFA method from new IP | Azure AD audit logs | Medium |
| Connection to PhaaS infrastructure | Web proxy logs | Medium |

## Incident Response Contacts
| Role | Name | Contact |
|---|---|---|
| SOC Lead | | |
| Identity Team | | |
| Email Security | | |

===== END FILE =====


===== FILE: skills/performing-adversary-in-the-middle-phishing-detection/references/api-reference.md =====
# Adversary-in-the-Middle (AiTM) Phishing Detection - API Reference

## AiTM Attack Overview

AiTM phishing uses a reverse proxy between the victim and legitimate login page to intercept session cookies in real-time, bypassing MFA. Common frameworks: Evilginx2, Modlishka, Muraena.

**Attack Chain:**
1. Victim clicks phishing link
2. Reverse proxy forwards request to real login page
3. Victim enters credentials and completes MFA
4. Proxy captures session cookie
5. Attacker replays session cookie from different location

## Azure AD / Entra ID Sign-In Logs

### Export via Microsoft Graph API
```
GET https://graph.microsoft.com/v1.0/auditLogs/signIns
```

### Key Fields

| Field | Type | Description |
|-------|------|-------------|
| `userPrincipalName` | string | User email |
| `createdDateTime` | ISO-8601 | Sign-in timestamp |
| `ipAddress` | string | Source IP address |
| `location.latitude` | float | Geo-location latitude |
| `location.longitude` | float | Geo-location longitude |
| `deviceDetail.displayName` | string | Device name |
| `correlationId` | string | Session correlation ID |
| `userAgent` | string | Browser user agent |

## Detection Methods

### Impossible Travel
Calculates Haversine great-circle distance between consecutive logins. If `distance / time > 900 km/h` (commercial flight speed) and distance > 100km, flags as suspicious.

### Suspicious Inbox Rules
AiTM attackers commonly create rules to:
- Forward emails to external address (`forwardTo`, `redirectTo`)
- Delete incoming emails (`moveToDeletedItems`, `permanentDelete`)
- Auto-read messages (`markAsRead`)
- Filter on keywords: invoice, payment, wire, bank, password

### Token Replay Detection
Multiple IPs and devices in a short timeframe for the same user session indicates stolen session token replay.

## Inbox Rules Format

```json
[
  {
    "displayName": "rule1",
    "mailboxOwner": "user@example.com",
    "actions": {"forwardTo": [{"emailAddress": {"address": "attacker@evil.com"}}]},
    "conditions": {"subjectContains": ["invoice", "payment"]},
    "createdDateTime": "2024-01-15T10:00:00Z"
  }
]
```

## Haversine Formula

```python
from math import radians, cos, sin, asin, sqrt
def haversine_km(lat1, lon1, lat2, lon2):
    lat1, lon1, lat2, lon2 = map(radians, [lat1, lon1, lat2, lon2])
    dlat, dlon = lat2 - lat1, lon2 - lon1
    a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
    return 2 * 6371 * asin(sqrt(a))
```

## Output Schema

```json
{
  "report": "aitm_phishing_detection",
  "total_sign_ins_analyzed": 5000,
  "total_findings": 8,
  "severity_summary": {"critical": 3, "high": 5},
  "findings": [{"type": "impossible_travel", "severity": "critical"}]
}
```

## CLI Usage

```bash
python agent.py --logs signin_logs.json --inbox-rules rules.json --output report.json
```

===== END FILE =====


===== FILE: skills/performing-adversary-in-the-middle-phishing-detection/references/standards.md =====
# Standards & References: AiTM Phishing Detection

## MITRE ATT&CK References
- **T1557**: Adversary-in-the-Middle
- **T1539**: Steal Web Session Cookie
- **T1550.004**: Use Alternate Authentication Material: Web Session Cookie
- **T1566.002**: Phishing: Spearphishing Link
- **T1114.003**: Email Collection: Email Forwarding Rule
- **T1098.005**: Account Manipulation: Device Registration

## AiTM PhaaS Landscape (2025)
- Over 1 million PhaaS attacks detected in Jan-Feb 2025 (Barracuda)
- Tycoon 2FA most prevalent followed by EvilProxy and Evilginx
- Transition from QR codes to HTML attachments and SVG files
- Average time from compromise to inbox rule creation: under 30 minutes

## Detection Rule Categories
| Rule | Data Source | Confidence |
|---|---|---|
| Session IP mismatch | Azure AD sign-in logs | High |
| Impossible travel | Azure AD + session logs | High |
| Inbox rule post-auth | Exchange audit logs | High |
| New MFA method post-risky-sign-in | Azure AD audit | Medium |
| OAuth consent post-auth | Azure AD audit | Medium |
| Proxy CDN pattern | Web proxy logs | Medium |
| New domain phishing page | DNS + CT logs | Low |

## Phishing-Resistant MFA Standards
- **FIDO2 WebAuthn**: Origin-bound authentication prevents AiTM
- **Certificate-Based Auth**: Client certificate bound to device
- **Windows Hello for Business**: Hardware-bound credential
- **NIST SP 800-63B AAL3**: Phishing-resistant authenticator requirement

===== END FILE =====


===== FILE: skills/performing-adversary-in-the-middle-phishing-detection/references/workflows.md =====
# Workflows: AiTM Phishing Detection

## Workflow 1: AiTM Attack Detection

```
User clicks phishing link
  |
  v
[Reverse proxy serves mirrored login page]
  +-- Page loads assets from legitimate CDN
  +-- SSL cert issued for lookalike domain
  |
  v
[User enters credentials + completes MFA]
  |
  v
[Attacker captures session cookie]
  |
  v
[DETECTION POINTS]
  +-- Web proxy: Connection to newly registered domain
  +-- Azure AD: Sign-in from proxy IP (unfamiliar location)
  +-- Session: Cookie replay from different IP within minutes
  +-- Exchange: Inbox rule created post-authentication
  +-- Azure AD: New OAuth app consent
  |
  v
[Automated response]
  +-- Revoke all sessions for user
  +-- Require re-authentication with phishing-resistant MFA
  +-- Remove suspicious inbox rules
  +-- Revoke OAuth app consents
  +-- Block attacker IP at firewall
```

## Workflow 2: AiTM Incident Response

```
AiTM compromise confirmed
  |
  v
[Immediate containment (first 30 minutes)]
  +-- Revoke all user sessions and tokens
  +-- Force password reset
  +-- Remove all inbox forwarding rules
  +-- Revoke OAuth app consents granted post-compromise
  +-- Disable compromised MFA methods
  |
  v
[Investigation (next 2-4 hours)]
  +-- Review Azure AD sign-in logs for compromise timeline
  +-- Check email sent items for BEC/phishing sent from account
  +-- Review SharePoint/OneDrive access for data exfiltration
  +-- Check for lateral movement to other accounts
  +-- Identify all affected users (same phishing campaign)
  |
  v
[Remediation]
  +-- Enroll user in phishing-resistant MFA (FIDO2)
  +-- Block phishing domain at email gateway and web proxy
  +-- Retract phishing email from all mailboxes
  +-- Update Conditional Access policies
  +-- Notify all targeted users
  |
  v
[Post-incident]
  +-- Add IOCs to threat intelligence
  +-- Create SIEM detection rules for observed TTPs
  +-- Update security awareness training
  +-- Assess FIDO2 rollout for broader user population
```

===== END FILE =====


===== FILE: skills/performing-adversary-in-the-middle-phishing-detection/scripts/agent.py =====
#!/usr/bin/env python3
"""Adversary-in-the-Middle (AiTM) Phishing Detection agent - analyzes sign-in
logs and inbox rules to detect AiTM phishing campaigns that bypass MFA by
proxying authentication sessions."""

import argparse
import json
from collections import Counter, defaultdict
from datetime import datetime
from pathlib import Path
from math import radians, cos, sin, asin, sqrt


def haversine_km(lat1: float, lon1: float, lat2: float, lon2: float) -> float:
    """Calculate great-circle distance between two points."""
    lat1, lon1, lat2, lon2 = map(radians, [lat1, lon1, lat2, lon2])
    dlat = lat2 - lat1
    dlon = lon2 - lon1
    a = sin(dlat / 2) ** 2 + cos(lat1) * cos(lat2) * sin(dlon / 2) ** 2
    return 2 * 6371 * asin(sqrt(a))


def load_sign_in_logs(log_path: str) -> list[dict]:
    """Load Azure AD / Entra ID sign-in logs in JSON format."""
    content = Path(log_path).read_text(encoding="utf-8")
    try:
        return json.loads(content)
    except json.JSONDecodeError:
        return [json.loads(line) for line in content.strip().splitlines() if line.strip()]


def detect_impossible_travel(logs: list[dict], max_speed_kmh: float = 900) -> list[dict]:
    """Detect impossible travel - logins from distant locations in short time."""
    findings = []
    user_logins = defaultdict(list)
    for log in logs:
        user = log.get("userPrincipalName", "")
        ts = log.get("createdDateTime", "")
        lat = log.get("location", {}).get("latitude")
        lon = log.get("location", {}).get("longitude")
        ip = log.get("ipAddress", "")
        if user and ts and lat is not None and lon is not None:
            try:
                dt = datetime.fromisoformat(ts.replace("Z", "+00:00"))
                user_logins[user].append({"dt": dt, "lat": lat, "lon": lon, "ip": ip})
            except ValueError:
                continue

    for user, logins in user_logins.items():
        logins.sort(key=lambda x: x["dt"])
        for i in range(1, len(logins)):
            prev, curr = logins[i - 1], logins[i]
            dist = haversine_km(prev["lat"], prev["lon"], curr["lat"], curr["lon"])
            hours = (curr["dt"] - prev["dt"]).total_seconds() / 3600
            if hours > 0 and dist / hours > max_speed_kmh and dist > 100:
                findings.append({
                    "type": "impossible_travel",
                    "severity": "critical",
                    "user": user,
                    "distance_km": round(dist, 1),
                    "time_hours": round(hours, 2),
                    "speed_kmh": round(dist / hours, 0),
                    "from_ip": prev["ip"],
                    "to_ip": curr["ip"],
                    "detail": f"Login from {round(dist)}km away in {round(hours, 1)}h ({round(dist/hours)}km/h)",
                })
    return findings


def detect_suspicious_inbox_rules(rules_path: str) -> list[dict]:
    """Detect inbox rules commonly created by AiTM attackers."""
    findings = []
    rules = json.loads(Path(rules_path).read_text(encoding="utf-8"))
    suspicious_actions = {"moveToDeletedItems", "permanentDelete", "forwardTo",
                          "redirectTo", "markAsRead"}
    suspicious_keywords = {"invoice", "payment", "wire", "bank", "urgent",
                           "password", "mfa", "security", "verify"}

    for rule in rules:
        actions = set(rule.get("actions", {}).keys())
        matched_actions = actions & suspicious_actions
        conditions = json.dumps(rule.get("conditions", {})).lower()
        matched_keywords = {kw for kw in suspicious_keywords if kw in conditions}

        if matched_actions:
            severity = "critical" if "forwardTo" in matched_actions or "redirectTo" in matched_actions else "high"
            findings.append({
                "type": "suspicious_inbox_rule",
                "severity": severity,
                "rule_name": rule.get("displayName", "unnamed"),
                "user": rule.get("mailboxOwner", "unknown"),
                "suspicious_actions": sorted(matched_actions),
                "keyword_triggers": sorted(matched_keywords),
                "created": rule.get("createdDateTime", ""),
                "detail": f"Rule with {', '.join(matched_actions)} actions",
            })
    return findings


def detect_token_replay(logs: list[dict]) -> list[dict]:
    """Detect potential session token replay from new device/location."""
    findings = []
    user_sessions = defaultdict(list)
    for log in logs:
        user = log.get("userPrincipalName", "")
        session_id = log.get("correlationId", "")
        device = log.get("deviceDetail", {}).get("displayName", "unknown")
        ip = log.get("ipAddress", "")
        user_agent = log.get("userAgent", "")
        if user:
            user_sessions[user].append({
                "session": session_id, "device": device,
                "ip": ip, "ua": user_agent,
            })

    for user, sessions in user_sessions.items():
        ips = set(s["ip"] for s in sessions)
        devices = set(s["device"] for s in sessions)
        if len(ips) > 3 and len(devices) > 3:
            findings.append({
                "type": "possible_token_replay",
                "severity": "high",
                "user": user,
                "unique_ips": len(ips),
                "unique_devices": len(devices),
                "detail": f"{len(ips)} IPs and {len(devices)} devices in session data",
            })
    return findings


def generate_report(log_path: str, rules_path: str = None,
                    max_speed: float = 900) -> dict:
    """Run all detection checks and build consolidated report."""
    logs = load_sign_in_logs(log_path)
    findings = []
    findings.extend(detect_impossible_travel(logs, max_speed))
    findings.extend(detect_token_replay(logs))
    if rules_path:
        findings.extend(detect_suspicious_inbox_rules(rules_path))

    severity_counts = Counter(f["severity"] for f in findings)
    return {
        "report": "aitm_phishing_detection",
        "generated_at": datetime.utcnow().isoformat() + "Z",
        "total_sign_ins_analyzed": len(logs),
        "total_findings": len(findings),
        "severity_summary": dict(severity_counts),
        "findings": findings,
    }


def main():
    parser = argparse.ArgumentParser(description="AiTM Phishing Detection Agent")
    parser.add_argument("--logs", required=True, help="Azure AD sign-in logs JSON file")
    parser.add_argument("--inbox-rules", help="Inbox rules JSON export")
    parser.add_argument("--max-speed", type=float, default=900, help="Max travel speed km/h (default: 900)")
    parser.add_argument("--output", help="Output JSON file path")
    args = parser.parse_args()

    report = generate_report(args.logs, args.inbox_rules, args.max_speed)
    output = json.dumps(report, indent=2)
    if args.output:
        Path(args.output).write_text(output, encoding="utf-8")
        print(f"Report written to {args.output}")
    else:
        print(output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-adversary-in-the-middle-phishing-detection/scripts/process.py =====
#!/usr/bin/env python3
"""
AiTM Phishing Detection Engine

Analyzes Azure AD sign-in logs and session data to detect
Adversary-in-the-Middle phishing attacks including session
cookie replay and impossible travel patterns.

Usage:
    python process.py detect --signin-log signins.json
    python process.py check-session --session-log sessions.json
    python process.py analyze-domain --domain suspicious-login.com
"""

import argparse
import json
import re
import sys
import math
from dataclasses import dataclass, field, asdict
from datetime import datetime, timedelta
from collections import defaultdict


@dataclass
class AiTMIndicator:
    """An AiTM detection indicator."""
    indicator_type: str = ""
    description: str = ""
    severity: str = "medium"
    confidence: float = 0.0
    user: str = ""
    timestamp: str = ""
    details: dict = field(default_factory=dict)


@dataclass
class AiTMAnalysis:
    """Complete AiTM analysis result."""
    total_signins: int = 0
    suspicious_signins: int = 0
    session_replays_detected: int = 0
    impossible_travel_detected: int = 0
    post_compromise_indicators: int = 0
    indicators: list = field(default_factory=list)
    affected_users: list = field(default_factory=list)
    recommended_actions: list = field(default_factory=list)


# Known AiTM infrastructure patterns
AITM_DOMAIN_PATTERNS = [
    r'login.*microsoft.*\.(top|xyz|info|click|online)',
    r'auth.*office.*\.(top|xyz|info|click|online)',
    r'sso.*\.(top|xyz|info|click|online)',
    r'verify.*account.*\.(top|xyz|info|click|online)',
    r'.*\.workers\.dev$',
    r'.*\.pages\.dev$',
    r'.*-login-.*\.(com|net|org)',
]

# Known PhaaS hosting patterns
PHAAS_INFRA = [
    'cloudflare-ipfs.com',
    'workers.dev',
    'pages.dev',
    'web.app',
    'firebaseapp.com',
    'glitch.me',
    'netlify.app',
    'vercel.app',
]


def haversine_distance(lat1, lon1, lat2, lon2):
    """Calculate distance in km between two coordinates."""
    R = 6371  # Earth radius in km
    lat1_r, lat2_r = math.radians(lat1), math.radians(lat2)
    dlat = math.radians(lat2 - lat1)
    dlon = math.radians(lon2 - lon1)
    a = (math.sin(dlat / 2) ** 2 +
         math.cos(lat1_r) * math.cos(lat2_r) * math.sin(dlon / 2) ** 2)
    c = 2 * math.asin(math.sqrt(a))
    return R * c


def detect_aitm_signins(signins: list) -> AiTMAnalysis:
    """Analyze sign-in logs for AiTM indicators."""
    analysis = AiTMAnalysis()
    analysis.total_signins = len(signins)

    user_signins = defaultdict(list)
    for signin in signins:
        user = signin.get("userPrincipalName", "")
        user_signins[user].append(signin)

    affected = set()

    for user, events in user_signins.items():
        sorted_events = sorted(events, key=lambda x: x.get("createdDateTime", ""))

        for i in range(len(sorted_events)):
            event = sorted_events[i]
            ip = event.get("ipAddress", "")
            location = event.get("location", {})
            risk_level = event.get("riskLevelDuringSignIn", "none")
            is_interactive = event.get("isInteractive", True)
            app = event.get("appDisplayName", "")
            timestamp = event.get("createdDateTime", "")

            # Check for anonymous proxy
            if event.get("isFromAnonymousProxy", False):
                analysis.indicators.append(AiTMIndicator(
                    indicator_type="anonymous_proxy",
                    description=f"Sign-in from anonymous proxy/VPN",
                    severity="high",
                    confidence=0.7,
                    user=user,
                    timestamp=timestamp,
                    details={"ip": ip}
                ))
                analysis.suspicious_signins += 1
                affected.add(user)

            # Check for impossible travel
            if i > 0:
                prev = sorted_events[i - 1]
                prev_loc = prev.get("location", {})
                prev_time = prev.get("createdDateTime", "")

                if (location.get("geoCoordinates") and
                        prev_loc.get("geoCoordinates")):
                    lat1 = prev_loc["geoCoordinates"].get("latitude", 0)
                    lon1 = prev_loc["geoCoordinates"].get("longitude", 0)
                    lat2 = location["geoCoordinates"].get("latitude", 0)
                    lon2 = location["geoCoordinates"].get("longitude", 0)

                    distance = haversine_distance(lat1, lon1, lat2, lon2)

                    try:
                        t1 = datetime.fromisoformat(prev_time.replace('Z', '+00:00'))
                        t2 = datetime.fromisoformat(timestamp.replace('Z', '+00:00'))
                        hours = (t2 - t1).total_seconds() / 3600

                        if hours > 0 and distance > 0:
                            speed = distance / hours
                            if speed > 900:  # Faster than commercial flight
                                analysis.indicators.append(AiTMIndicator(
                                    indicator_type="impossible_travel",
                                    description=(
                                        f"Impossible travel: {distance:.0f}km in "
                                        f"{hours:.1f}h ({speed:.0f}km/h)"
                                    ),
                                    severity="high",
                                    confidence=0.85,
                                    user=user,
                                    timestamp=timestamp,
                                    details={
                                        "from_ip": prev.get("ipAddress"),
                                        "to_ip": ip,
                                        "distance_km": round(distance),
                                        "speed_kmh": round(speed)
                                    }
                                ))
                                analysis.impossible_travel_detected += 1
                                affected.add(user)
                    except (ValueError, TypeError):
                        pass

            # Check for session from different IP shortly after auth
            if i < len(sorted_events) - 1:
                next_event = sorted_events[i + 1]
                next_ip = next_event.get("ipAddress", "")
                next_time = next_event.get("createdDateTime", "")

                if ip and next_ip and ip != next_ip:
                    try:
                        t1 = datetime.fromisoformat(timestamp.replace('Z', '+00:00'))
                        t2 = datetime.fromisoformat(next_time.replace('Z', '+00:00'))
                        minutes = (t2 - t1).total_seconds() / 60

                        if 0 < minutes < 10:
                            analysis.indicators.append(AiTMIndicator(
                                indicator_type="session_ip_switch",
                                description=(
                                    f"Session IP changed within {minutes:.0f}min "
                                    f"({ip} -> {next_ip})"
                                ),
                                severity="critical",
                                confidence=0.9,
                                user=user,
                                timestamp=timestamp,
                                details={
                                    "auth_ip": ip,
                                    "session_ip": next_ip,
                                    "time_delta_min": round(minutes)
                                }
                            ))
                            analysis.session_replays_detected += 1
                            affected.add(user)
                    except (ValueError, TypeError):
                        pass

    analysis.affected_users = list(affected)
    if affected:
        analysis.recommended_actions = [
            "Revoke all sessions for affected users",
            "Force password reset with phishing-resistant MFA",
            "Check for inbox forwarding rules created post-compromise",
            "Review OAuth app consents for affected accounts",
            "Block source IPs at firewall",
            "Retract phishing email from all mailboxes"
        ]

    return analysis


def analyze_domain(domain: str) -> dict:
    """Check if domain matches known AiTM/PhaaS patterns."""
    result = {
        "domain": domain,
        "is_suspicious": False,
        "indicators": [],
        "risk_score": 0
    }

    domain_lower = domain.lower()

    for pattern in AITM_DOMAIN_PATTERNS:
        if re.search(pattern, domain_lower):
            result["indicators"].append(f"Matches AiTM domain pattern: {pattern}")
            result["risk_score"] += 30
            result["is_suspicious"] = True

    for infra in PHAAS_INFRA:
        if domain_lower.endswith(infra):
            result["indicators"].append(f"Hosted on known PhaaS infrastructure: {infra}")
            result["risk_score"] += 25
            result["is_suspicious"] = True

    # Check for brand impersonation in domain
    brands = ['microsoft', 'office', 'outlook', 'google', 'okta', 'azure']
    for brand in brands:
        if brand in domain_lower and not domain_lower.endswith(f'.{brand}.com'):
            result["indicators"].append(f"Contains brand name '{brand}' in non-official domain")
            result["risk_score"] += 20
            result["is_suspicious"] = True

    result["risk_score"] = min(result["risk_score"], 100)
    return result


def main():
    parser = argparse.ArgumentParser(description="AiTM Phishing Detection")
    subparsers = parser.add_subparsers(dest="command")

    detect_parser = subparsers.add_parser("detect", help="Detect AiTM in sign-in logs")
    detect_parser.add_argument("--signin-log", required=True)

    domain_parser = subparsers.add_parser("analyze-domain", help="Check domain for AiTM")
    domain_parser.add_argument("--domain", required=True)

    parser.add_argument("--json", action="store_true")
    args = parser.parse_args()

    if args.command == "detect":
        with open(args.signin_log) as f:
            signins = json.load(f)
        result = detect_aitm_signins(signins)
        if args.json:
            print(json.dumps(asdict(result), indent=2, default=str))
        else:
            print(f"Total sign-ins: {result.total_signins}")
            print(f"Suspicious: {result.suspicious_signins}")
            print(f"Session replays: {result.session_replays_detected}")
            print(f"Impossible travel: {result.impossible_travel_detected}")
            print(f"Affected users: {len(result.affected_users)}")
            for ind in result.indicators:
                print(f"  [{ind.severity.upper()}] {ind.description} (user: {ind.user})")

    elif args.command == "analyze-domain":
        result = analyze_domain(args.domain)
        print(json.dumps(result, indent=2))

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-dmarc-policy-enforcement-rollout/SKILL.md =====
---
name: performing-dmarc-policy-enforcement-rollout
description: Execute a phased DMARC rollout by inventorying sending sources, configuring
  SPF/DKIM alignment, and progressing DNS policy from p=none monitoring through p=quarantine
  to p=reject enforcement, ensuring all legitimate email sources authenticate before
  unauthorized senders are blocked. Use when deploying or advancing an organization's
  DMARC anti-spoofing posture, or when meeting bulk-sender authentication requirements
  from Google and Yahoo.
domain: cybersecurity
subdomain: phishing-defense
tags:
- dmarc
- spf
- dkim
- email-authentication
- anti-spoofing
- phishing
- dns
- email-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566
- T1598
- T1534
- T1036
---
# Performing DMARC Policy Enforcement Rollout

## Overview
Domain-based Message Authentication, Reporting and Conformance (DMARC) is the cornerstone of email anti-spoofing protection. A DMARC rollout progresses through three phases: monitoring (p=none), quarantine (p=quarantine), and full enforcement (p=reject). When configured at p=reject, any email that fails both SPF and DKIM checks is outright rejected. Google and Yahoo now require DMARC for bulk senders (5,000+ emails), driving a 65% reduction in unauthenticated messages. The rollout typically takes 3-6 months for safe deployment.


## When to Use

- When conducting security assessments that involve performing dmarc policy enforcement rollout
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites
- Administrative access to DNS management for the domain
- Understanding of SPF, DKIM, and DMARC protocols (RFC 7208, 6376, 7489)
- Complete inventory of all legitimate email sending sources
- DMARC reporting analysis tool (EasyDMARC, DMARCLY, Valimail, or dmarcian)
- Email gateway with DMARC enforcement capability

## Key Concepts

### DMARC Policy Levels
| Policy | Behavior | Use Case |
|---|---|---|
| p=none | Monitor only, no action on failures | Discovery phase |
| p=quarantine | Send failing messages to spam/junk | Transition phase |
| p=reject | Block failing messages entirely | Full enforcement |

### DMARC Record Anatomy
```
v=DMARC1; p=quarantine; pct=25; rua=mailto:dmarc-agg@company.com; ruf=mailto:dmarc-forensic@company.com; adkim=r; aspf=r; fo=1
```
- **p**: Policy for organizational domain
- **sp**: Policy for subdomains
- **pct**: Percentage of messages subject to policy (for gradual rollout)
- **rua**: Aggregate report destination (daily XML reports)
- **ruf**: Forensic report destination (per-failure reports)
- **adkim**: DKIM alignment mode (r=relaxed, s=strict)
- **aspf**: SPF alignment mode (r=relaxed, s=strict)
- **fo**: Failure reporting options (0=both fail, 1=either fails)

### SPF and DKIM Alignment
- **SPF Alignment**: The domain in the Return-Path (envelope sender) must match the From header domain
- **DKIM Alignment**: The d= domain in the DKIM signature must match the From header domain
- **Relaxed**: Organizational domain match (sub.example.com matches example.com)
- **Strict**: Exact domain match required

## Workflow

### Step 1: Inventory All Sending Sources (Week 1-2)
- Audit all systems sending email as your domain (marketing, CRM, ticketing, transactional)
- Document third-party services: Salesforce, Mailchimp, SendGrid, Zendesk, etc.
- Identify internal mail servers, applications, and relay hosts
- Check for shadow IT email sending (departments using unauthorized services)

### Step 2: Configure SPF and DKIM (Week 2-4)
- Consolidate SPF record with all legitimate sending IPs and includes
- Ensure SPF record stays under 10 DNS lookup limit
- Generate and publish DKIM keys for each sending source
- Verify DKIM signing works for all outbound mail paths
- Test with MX Toolbox or dmarcian SPF/DKIM validators

### Step 3: Deploy DMARC in Monitoring Mode (Week 4-6)
- Publish initial DMARC record: `v=DMARC1; p=none; rua=mailto:dmarc@company.com; fo=1`
- Wait 1-2 weeks to collect representative aggregate reports
- Analyze reports to identify unauthorized senders and alignment failures
- Fix SPF/DKIM for all legitimate sources showing failures
- Iterate until all legitimate mail passes DMARC

### Step 4: Move to Quarantine with pct Tag (Week 6-12)
- Update to quarantine at 10%: `v=DMARC1; p=quarantine; pct=10; rua=...`
- Monitor for false positives (legitimate mail being quarantined)
- Increase pct gradually: 10% -> 25% -> 50% -> 75% -> 100%
- Each increase: wait 1-2 weeks and review reports before advancing
- Fix any remaining alignment issues discovered at each stage

### Step 5: Advance to Reject Policy (Week 12-20)
- After stable quarantine at 100%, move to reject at 10%: `v=DMARC1; p=reject; pct=10; rua=...`
- Gradually increase pct: 10% -> 25% -> 50% -> 100%
- Monitor closely for legitimate mail being rejected
- Establish emergency rollback procedure (revert to quarantine)
- Apply subdomain policy: `sp=reject` for subdomains

### Step 6: Ongoing Monitoring and Maintenance
- Continuously monitor DMARC aggregate reports
- Add new sending sources before they start sending
- Review forensic reports for spoofing attempts
- Maintain SPF record as sending infrastructure changes
- Rotate DKIM keys annually

## Tools & Resources
- **EasyDMARC**: DMARC monitoring dashboard with aggregate/forensic report analysis
- **DMARCLY**: SPF, DKIM, DMARC monitoring with auto-DNS updates
- **dmarcian**: DMARC deployment and management platform
- **Valimail**: Automated DMARC enforcement with hosted authentication
- **MX Toolbox**: DNS record lookup and DMARC validator
- **Google Admin Toolbox**: DMARC check and diagnostic tools

## Validation
- DMARC record published and resolving correctly at _dmarc.domain.com
- All legitimate sending sources pass SPF and/or DKIM alignment
- Aggregate reports show >99% legitimate mail passing DMARC
- Spoofed messages from unauthorized senders are rejected
- No legitimate mail blocked after full p=reject enforcement
- Subdomain policy (sp=) also set to reject

===== END FILE =====


===== FILE: skills/performing-dmarc-policy-enforcement-rollout/LICENSE =====

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


===== FILE: skills/performing-dmarc-policy-enforcement-rollout/assets/template.md =====
# DMARC Policy Enforcement Rollout Template

## Sending Source Inventory
| Source | Type | SPF Included | DKIM Configured | Status |
|---|---|---|---|---|
| Exchange Online | Primary mail | include:spf.protection.outlook.com | selector1/selector2 | |
| SendGrid | Transactional | include:sendgrid.net | sg._domainkey | |
| Mailchimp | Marketing | include:servers.mcsv.net | k1._domainkey | |
| Salesforce | CRM | include:_spf.salesforce.com | salesforce._domainkey | |

## DMARC Rollout Schedule
| Week | Phase | Record Change | Monitoring |
|---|---|---|---|
| 1-2 | Discovery | (no DMARC yet) | Audit sending sources |
| 3-4 | SPF/DKIM | (configure auth) | Test outbound auth |
| 5-6 | Monitor | p=none; rua=... | Daily report review |
| 7-8 | Quarantine 10% | p=quarantine; pct=10 | Check false positives |
| 9-10 | Quarantine 50% | p=quarantine; pct=50 | Validate stability |
| 11-12 | Quarantine 100% | p=quarantine; pct=100 | Confirm all passing |
| 13-14 | Reject 10% | p=reject; pct=10 | Monitor rejections |
| 15-16 | Reject 50% | p=reject; pct=50 | Near full enforcement |
| 17-20 | Reject 100% | p=reject | FULL ENFORCEMENT |

## Emergency Rollback Procedure
- [ ] Reduce pct to previous stable value OR
- [ ] Revert policy to previous level (reject->quarantine or quarantine->none)
- [ ] Investigate failing source in DMARC reports
- [ ] Fix authentication issue
- [ ] Resume rollout after confirmation

## Sign-off
| Milestone | Approved By | Date |
|---|---|---|
| SPF/DKIM configured | | |
| p=none deployed | | |
| p=quarantine 100% | | |
| p=reject 100% | | |

===== END FILE =====


===== FILE: skills/performing-dmarc-policy-enforcement-rollout/references/api-reference.md =====
# API Reference — Performing DMARC Policy Enforcement Rollout

## Libraries Used
- **dnspython** (dns.resolver): DNS TXT record queries for DMARC, SPF, DKIM

## CLI Interface

```
python agent.py check --domain example.com
python agent.py audit --domains example.com example.org [--selectors default google k1]
```

## Core Functions

### `check_dmarc(domain)` — Query `_dmarc.<domain>` TXT
### `check_spf(domain)` — Query domain TXT for `v=spf1`
### `check_dkim(domain, selector)` — Query `<selector>._domainkey.<domain>`
### `audit_domains(domains, selectors)` — Full DMARC/SPF/DKIM audit with scoring

## DMARC Policy Levels
| Policy | Enforcement | Score |
|--------|------------|-------|
| `none` | No enforcement (monitoring only) | 0 |
| `quarantine` | Suspicious mail sent to spam | +20 |
| `reject` | Unauthorized mail rejected | +40 |

## Dependencies
```
pip install dnspython>=2.4
```

===== END FILE =====


===== FILE: skills/performing-dmarc-policy-enforcement-rollout/references/standards.md =====
# Standards & References: Performing DMARC Policy Enforcement Rollout

## Core Protocol Standards
- **RFC 7489**: Domain-based Message Authentication, Reporting, and Conformance (DMARC)
- **RFC 7208**: Sender Policy Framework (SPF) for Authorizing Use of Domains in Email
- **RFC 6376**: DomainKeys Identified Mail (DKIM) Signatures
- **RFC 8616**: Email Authentication for Internationalized Mail
- **RFC 8601**: Message Header Field for Indicating Message Authentication Status

## Regulatory Requirements (2025)
- **Google Bulk Sender Requirements**: DMARC required for senders of 5,000+ messages/day to Gmail
- **Yahoo Sender Requirements**: DMARC authentication required for bulk senders
- **PCI DSS 4.0 Requirement 5.3**: Anti-phishing mechanisms including email authentication
- **NIST SP 800-177 Rev.1**: Trustworthy Email guidance

## DMARC Policy Progression
| Phase | Duration | Record | pct | Monitoring |
|---|---|---|---|---|
| Discovery | 2-4 weeks | p=none | N/A | Daily report review |
| Soft enforce | 4-6 weeks | p=quarantine | 10->100 | Twice-weekly review |
| Hard enforce | 4-6 weeks | p=reject | 10->100 | Weekly review |
| Maintenance | Ongoing | p=reject | 100 | Monthly review |

## SPF Lookup Limit
- Maximum 10 DNS lookups per SPF evaluation (RFC 7208 Section 4.6.4)
- Each `include:`, `a:`, `mx:`, `redirect=`, and `exists:` counts as one lookup
- Exceeding 10 lookups causes SPF permerror (treated as fail by some receivers)
- Use SPF flattening tools to stay within limit

## MITRE ATT&CK References
- **T1566.001**: Phishing: Spearphishing Attachment
- **T1566.002**: Phishing: Spearphishing Link
- **T1586.002**: Compromise Accounts: Email Accounts
- **T1656**: Impersonation

===== END FILE =====


===== FILE: skills/performing-dmarc-policy-enforcement-rollout/references/workflows.md =====
# Workflows: Performing DMARC Policy Enforcement Rollout

## Workflow 1: DMARC Phased Rollout

```
Week 1-2: Discovery
  |
  v
[Inventory all legitimate email sending sources]
  +-- Internal mail servers
  +-- Third-party SaaS (marketing, CRM, support)
  +-- Transactional email services
  +-- Application-generated email
  |
  v
Week 2-4: Foundation
  |
  v
[Configure SPF and DKIM for all sources]
  +-- Publish SPF record with all includes
  +-- Validate SPF under 10 lookup limit
  +-- Generate DKIM keys per sending source
  +-- Test outbound authentication
  |
  v
Week 4-6: Monitor
  |
  v
[Publish p=none DMARC record]
  +-- Collect aggregate reports for 2 weeks
  +-- Analyze: who is sending as your domain?
  +-- Fix alignment failures for legitimate sources
  +-- Identify unauthorized/spoofing sources
  |
  v
Week 6-12: Quarantine
  |
  v
[Move to p=quarantine with gradual pct increase]
  +-- pct=10 (2 weeks) -> check false positives
  +-- pct=25 (2 weeks) -> verify clean
  +-- pct=50 (1 week) -> validate stability
  +-- pct=100 (2 weeks) -> confirm all legitimate passes
  |
  v
Week 12-20: Reject
  |
  v
[Move to p=reject with gradual pct increase]
  +-- pct=10 (2 weeks) -> monitor rejections
  +-- pct=25 (2 weeks) -> verify no legitimate blocked
  +-- pct=50 (1 week) -> near full enforcement
  +-- pct=100 -> FULL ENFORCEMENT ACHIEVED
  |
  v
Ongoing: Maintenance
  +-- Monitor aggregate reports monthly
  +-- Update SPF/DKIM for new sending sources
  +-- Rotate DKIM keys annually
```

## Workflow 2: DMARC Report Analysis

```
Aggregate report received (daily XML)
  |
  v
[Parse report in DMARC analyzer]
  |
  v
[Categorize sending sources]
  +-- PASS: Legitimate, properly authenticated
  +-- FAIL (known): Legitimate source with auth issue -> FIX
  +-- FAIL (unknown): Unauthorized sender -> INVESTIGATE
  |
  v
[For each FAIL (known)]
  +-- Identify missing SPF include or DKIM config
  +-- Update DNS records
  +-- Wait for next report to confirm fix
  |
  v
[For each FAIL (unknown)]
  +-- Is it spoofing? -> Document for enforcement case
  +-- Is it shadow IT? -> Onboard or decommission
  +-- Is it forwarding? -> ARC chain may be needed
```

## Workflow 3: Emergency Rollback

```
Legitimate email being rejected (false positive detected)
  |
  v
[Immediate: Roll back pct or policy]
  +-- Reduce pct to previous stable level
  +-- OR roll back from reject to quarantine
  +-- OR roll back from quarantine to none
  |
  v
[Investigate root cause]
  +-- Check aggregate reports for failing source
  +-- Verify SPF/DKIM configuration for source
  +-- Check for forwarding or mailing list issues
  |
  v
[Fix and re-advance]
  +-- Correct authentication issue
  +-- Verify fix in next report cycle
  +-- Resume gradual pct advancement
```

===== END FILE =====


===== FILE: skills/performing-dmarc-policy-enforcement-rollout/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing DMARC policy enforcement rollout with DNS record analysis."""

import json
import argparse
from datetime import datetime

try:
    import dns.resolver
except ImportError:
    dns = None


def check_dmarc(domain):
    """Query and parse DMARC record for a domain."""
    try:
        answers = dns.resolver.resolve(f"_dmarc.{domain}", "TXT")
        for rdata in answers:
            txt = rdata.to_text().strip('"')
            if txt.startswith("v=DMARC1"):
                return parse_dmarc_tags(txt, domain)
        return {"domain": domain, "dmarc_found": False}
    except (dns.resolver.NXDOMAIN, dns.resolver.NoAnswer, Exception) as e:
        return {"domain": domain, "dmarc_found": False, "error": str(e)}


def parse_dmarc_tags(record, domain):
    tags = {}
    for part in record.split(";"):
        part = part.strip()
        if "=" in part:
            k, _, v = part.partition("=")
            tags[k.strip()] = v.strip()
    policy = tags.get("p", "none")
    findings = []
    if policy == "none":
        findings.append({"severity": "HIGH", "finding": "DMARC policy is 'none' — no enforcement"})
    elif policy == "quarantine":
        findings.append({"severity": "MEDIUM", "finding": "DMARC policy is 'quarantine' — partial enforcement"})
    if "rua" not in tags:
        findings.append({"severity": "MEDIUM", "finding": "No aggregate report URI (rua) configured"})
    pct = int(tags.get("pct", "100"))
    if pct < 100:
        findings.append({"severity": "INFO", "finding": f"Policy applied to {pct}% of messages"})
    sp = tags.get("sp", policy)
    if sp == "none" and policy != "none":
        findings.append({"severity": "MEDIUM", "finding": "Subdomain policy (sp) is 'none'"})
    return {
        "domain": domain, "dmarc_found": True, "record": record,
        "policy": policy, "subdomain_policy": sp, "percentage": pct,
        "rua": tags.get("rua"), "ruf": tags.get("ruf"),
        "adkim": tags.get("adkim", "r"), "aspf": tags.get("aspf", "r"),
        "findings": findings,
        "enforcement_level": "full" if policy == "reject" and pct == 100 else "partial" if policy != "none" else "none",
    }


def check_spf(domain):
    """Query and analyze SPF record."""
    try:
        answers = dns.resolver.resolve(domain, "TXT")
        for rdata in answers:
            txt = rdata.to_text().strip('"')
            if txt.startswith("v=spf1"):
                mechanisms = txt.split()
                qualifier = mechanisms[-1] if mechanisms else "~all"
                return {"domain": domain, "spf_found": True, "record": txt,
                        "mechanisms": mechanisms, "qualifier": qualifier,
                        "strict": qualifier == "-all"}
        return {"domain": domain, "spf_found": False}
    except Exception as e:
        return {"domain": domain, "spf_found": False, "error": str(e)}


def check_dkim(domain, selector="default"):
    """Query DKIM public key record."""
    try:
        fqdn = f"{selector}._domainkey.{domain}"
        answers = dns.resolver.resolve(fqdn, "TXT")
        for rdata in answers:
            txt = rdata.to_text().strip('"')
            if "p=" in txt:
                return {"domain": domain, "selector": selector, "dkim_found": True, "record": txt[:200]}
        return {"domain": domain, "selector": selector, "dkim_found": False}
    except Exception as e:
        return {"domain": domain, "selector": selector, "dkim_found": False, "error": str(e)}


def audit_domains(domains, selectors=None):
    """Audit multiple domains for DMARC/SPF/DKIM readiness."""
    selectors = selectors or ["default", "google", "selector1", "selector2", "k1"]
    results = []
    for domain in domains:
        domain = domain.strip()
        if not domain:
            continue
        dmarc = check_dmarc(domain)
        spf = check_spf(domain)
        dkim_results = []
        for sel in selectors:
            dkim = check_dkim(domain, sel)
            if dkim.get("dkim_found"):
                dkim_results.append(dkim)
        score = 0
        if dmarc.get("policy") == "reject":
            score += 40
        elif dmarc.get("policy") == "quarantine":
            score += 20
        if spf.get("strict"):
            score += 30
        elif spf.get("spf_found"):
            score += 15
        if dkim_results:
            score += 30
        results.append({
            "domain": domain, "dmarc": dmarc, "spf": spf,
            "dkim": dkim_results, "security_score": score,
        })
    return {"timestamp": datetime.utcnow().isoformat(), "domains": results}


def main():
    if not dns:
        print(json.dumps({"error": "dnspython not installed"}))
        return
    parser = argparse.ArgumentParser(description="DMARC Policy Enforcement Rollout Agent")
    sub = parser.add_subparsers(dest="command")
    d = sub.add_parser("check", help="Check single domain")
    d.add_argument("--domain", required=True)
    a = sub.add_parser("audit", help="Audit multiple domains")
    a.add_argument("--domains", nargs="+", required=True)
    a.add_argument("--selectors", nargs="*", default=None)
    args = parser.parse_args()
    if args.command == "check":
        result = {"dmarc": check_dmarc(args.domain), "spf": check_spf(args.domain)}
    elif args.command == "audit":
        result = audit_domains(args.domains, args.selectors)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-dmarc-policy-enforcement-rollout/scripts/process.py =====
#!/usr/bin/env python3
"""
DMARC Policy Enforcement Rollout Analyzer

Checks current DMARC, SPF, and DKIM status for a domain and provides
rollout recommendations. Parses DMARC aggregate reports to identify
sending sources and authentication failures.

Usage:
    python process.py check --domain example.com
    python process.py parse-report --report-file aggregate.xml
    python process.py spf-audit --domain example.com
"""

import argparse
import json
import re
import sys
import xml.etree.ElementTree as ET
from dataclasses import dataclass, field, asdict
from collections import defaultdict

try:
    import dns.resolver
    HAS_DNS = True
except ImportError:
    HAS_DNS = False


@dataclass
class DMARCStatus:
    """Current DMARC deployment status."""
    domain: str = ""
    dmarc_record: str = ""
    dmarc_policy: str = ""
    dmarc_pct: int = 100
    subdomain_policy: str = ""
    rua_configured: bool = False
    ruf_configured: bool = False
    spf_record: str = ""
    spf_valid: bool = False
    spf_lookup_count: int = 0
    dkim_found: bool = False
    current_phase: str = ""
    next_action: str = ""
    issues: list = field(default_factory=list)


@dataclass
class ReportSummary:
    """DMARC aggregate report summary."""
    report_org: str = ""
    report_domain: str = ""
    date_range: str = ""
    total_messages: int = 0
    pass_count: int = 0
    fail_count: int = 0
    pass_rate: float = 0.0
    sources: list = field(default_factory=list)
    failing_sources: list = field(default_factory=list)


def count_spf_lookups(spf_record: str) -> int:
    """Count DNS lookups in SPF record (max 10 allowed)."""
    lookup_count = 0
    lookup_mechanisms = ['include:', 'a:', 'mx:', 'redirect=', 'exists:']
    for mechanism in lookup_mechanisms:
        lookup_count += spf_record.lower().count(mechanism)
    # bare 'a' and 'mx' without ':' also count
    parts = spf_record.split()
    for part in parts:
        p = part.lstrip('+').lstrip('-').lstrip('~').lstrip('?')
        if p in ('a', 'mx'):
            lookup_count += 1
    return lookup_count


def check_dmarc_status(domain: str) -> DMARCStatus:
    """Check DMARC, SPF, and DKIM status for a domain."""
    status = DMARCStatus(domain=domain)

    if not HAS_DNS:
        status.issues.append("dnspython not installed. Install with: pip install dnspython")
        return status

    # Check DMARC
    try:
        answers = dns.resolver.resolve(f"_dmarc.{domain}", 'TXT')
        for rdata in answers:
            txt = str(rdata).strip('"')
            if txt.startswith('v=DMARC1'):
                status.dmarc_record = txt
                policy = re.search(r'p=(\w+)', txt)
                if policy:
                    status.dmarc_policy = policy.group(1)
                pct = re.search(r'pct=(\d+)', txt)
                if pct:
                    status.dmarc_pct = int(pct.group(1))
                sp = re.search(r'sp=(\w+)', txt)
                if sp:
                    status.subdomain_policy = sp.group(1)
                status.rua_configured = 'rua=' in txt
                status.ruf_configured = 'ruf=' in txt
                break
    except Exception:
        status.issues.append("No DMARC record found - start with p=none")

    # Check SPF
    try:
        answers = dns.resolver.resolve(domain, 'TXT')
        for rdata in answers:
            txt = str(rdata).strip('"')
            if txt.startswith('v=spf1'):
                status.spf_record = txt
                status.spf_lookup_count = count_spf_lookups(txt)
                status.spf_valid = status.spf_lookup_count <= 10
                if not status.spf_valid:
                    status.issues.append(
                        f"SPF record exceeds 10 lookup limit "
                        f"({status.spf_lookup_count} lookups). Use SPF flattening."
                    )
                break
    except Exception:
        status.issues.append("No SPF record found")

    # Check common DKIM selectors
    selectors = ['default', 'google', 'selector1', 'selector2', 'proofpoint',
                 'mimecast', 's1', 's2', 'k1']
    for selector in selectors:
        try:
            dns.resolver.resolve(f"{selector}._domainkey.{domain}", 'TXT')
            status.dkim_found = True
            break
        except Exception:
            continue

    if not status.dkim_found:
        status.issues.append("No DKIM record found for common selectors")

    # Determine current phase and next action
    if not status.dmarc_record:
        status.current_phase = "Not started"
        status.next_action = "Publish DMARC record with p=none and rua= for monitoring"
    elif status.dmarc_policy == "none":
        status.current_phase = "Monitoring (p=none)"
        status.next_action = "Analyze reports, fix auth issues, then move to p=quarantine pct=10"
    elif status.dmarc_policy == "quarantine":
        if status.dmarc_pct < 100:
            status.current_phase = f"Quarantine at {status.dmarc_pct}%"
            next_pct = min(status.dmarc_pct * 2, 100)
            status.next_action = f"Increase pct to {next_pct} after reviewing reports"
        else:
            status.current_phase = "Quarantine at 100%"
            status.next_action = "Move to p=reject pct=10 after stable period"
    elif status.dmarc_policy == "reject":
        if status.dmarc_pct < 100:
            status.current_phase = f"Reject at {status.dmarc_pct}%"
            next_pct = min(status.dmarc_pct * 2, 100)
            status.next_action = f"Increase pct to {next_pct} after reviewing reports"
        else:
            status.current_phase = "FULL ENFORCEMENT (p=reject 100%)"
            status.next_action = "Maintain: monitor reports, update auth for new sources"

    if not status.rua_configured and status.dmarc_record:
        status.issues.append("No rua= tag - add aggregate report destination")

    if status.dmarc_policy == "reject" and not status.subdomain_policy:
        status.issues.append("No subdomain policy (sp=) - subdomains can still be spoofed")

    return status


def parse_dmarc_report(xml_file: str) -> ReportSummary:
    """Parse DMARC aggregate report XML."""
    summary = ReportSummary()

    tree = ET.parse(xml_file)
    root = tree.getroot()

    # Report metadata
    metadata = root.find('.//report_metadata')
    if metadata is not None:
        org = metadata.find('org_name')
        if org is not None:
            summary.report_org = org.text or ""

    policy = root.find('.//policy_published')
    if policy is not None:
        domain = policy.find('domain')
        if domain is not None:
            summary.report_domain = domain.text or ""

    date_range = root.find('.//date_range')
    if date_range is not None:
        begin = date_range.find('begin')
        end = date_range.find('end')
        if begin is not None and end is not None:
            summary.date_range = f"{begin.text} - {end.text}"

    # Process records
    for record in root.findall('.//record'):
        row = record.find('row')
        if row is None:
            continue

        source_ip = ""
        count = 0
        spf_result = ""
        dkim_result = ""
        disposition = ""

        ip_elem = row.find('source_ip')
        if ip_elem is not None:
            source_ip = ip_elem.text or ""

        count_elem = row.find('count')
        if count_elem is not None:
            count = int(count_elem.text or 0)

        policy_eval = row.find('policy_evaluated')
        if policy_eval is not None:
            dkim_elem = policy_eval.find('dkim')
            spf_elem = policy_eval.find('spf')
            disp_elem = policy_eval.find('disposition')
            if dkim_elem is not None:
                dkim_result = dkim_elem.text or ""
            if spf_elem is not None:
                spf_result = spf_elem.text or ""
            if disp_elem is not None:
                disposition = disp_elem.text or ""

        summary.total_messages += count
        passed = (spf_result == "pass" or dkim_result == "pass")

        source_info = {
            "source_ip": source_ip,
            "count": count,
            "spf": spf_result,
            "dkim": dkim_result,
            "disposition": disposition,
            "passed": passed
        }
        summary.sources.append(source_info)

        if passed:
            summary.pass_count += count
        else:
            summary.fail_count += count
            summary.failing_sources.append(source_info)

    if summary.total_messages > 0:
        summary.pass_rate = (summary.pass_count / summary.total_messages) * 100

    return summary


def format_status_report(status: DMARCStatus) -> str:
    """Format DMARC status as readable report."""
    lines = []
    lines.append("=" * 60)
    lines.append("  DMARC ENFORCEMENT ROLLOUT STATUS")
    lines.append("=" * 60)
    lines.append(f"  Domain: {status.domain}")
    lines.append(f"  Current Phase: {status.current_phase}")
    lines.append(f"  Next Action: {status.next_action}")
    lines.append("")
    lines.append(f"  DMARC Record: {status.dmarc_record or 'NOT FOUND'}")
    lines.append(f"  DMARC Policy: {status.dmarc_policy or 'N/A'}")
    lines.append(f"  DMARC pct: {status.dmarc_pct}%")
    lines.append(f"  Subdomain Policy: {status.subdomain_policy or 'Not set'}")
    lines.append(f"  RUA Reports: {'Yes' if status.rua_configured else 'No'}")
    lines.append(f"  RUF Reports: {'Yes' if status.ruf_configured else 'No'}")
    lines.append("")
    lines.append(f"  SPF Record: {status.spf_record or 'NOT FOUND'}")
    lines.append(f"  SPF Lookups: {status.spf_lookup_count}/10")
    lines.append(f"  SPF Valid: {'Yes' if status.spf_valid else 'No'}")
    lines.append(f"  DKIM Found: {'Yes' if status.dkim_found else 'No'}")

    if status.issues:
        lines.append(f"\n  [ISSUES] ({len(status.issues)})")
        for i, issue in enumerate(status.issues, 1):
            lines.append(f"    {i}. {issue}")

    lines.append("=" * 60)
    return "\n".join(lines)


def main():
    parser = argparse.ArgumentParser(description="DMARC Policy Enforcement Rollout Analyzer")
    subparsers = parser.add_subparsers(dest="command")

    check_parser = subparsers.add_parser("check", help="Check DMARC deployment status")
    check_parser.add_argument("--domain", required=True)

    report_parser = subparsers.add_parser("parse-report", help="Parse DMARC aggregate report")
    report_parser.add_argument("--report-file", required=True)

    spf_parser = subparsers.add_parser("spf-audit", help="Audit SPF record")
    spf_parser.add_argument("--domain", required=True)

    parser.add_argument("--json", action="store_true")

    args = parser.parse_args()

    if args.command == "check":
        result = check_dmarc_status(args.domain)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(format_status_report(result))

    elif args.command == "parse-report":
        result = parse_dmarc_report(args.report_file)
        if args.json:
            print(json.dumps(asdict(result), indent=2))
        else:
            print(f"Report from: {result.report_org}")
            print(f"Domain: {result.report_domain}")
            print(f"Total messages: {result.total_messages}")
            print(f"Pass rate: {result.pass_rate:.1f}%")
            if result.failing_sources:
                print(f"\nFailing sources ({len(result.failing_sources)}):")
                for src in result.failing_sources:
                    print(f"  IP: {src['source_ip']} | Count: {src['count']} | "
                          f"SPF: {src['spf']} | DKIM: {src['dkim']}")

    elif args.command == "spf-audit":
        status = check_dmarc_status(args.domain)
        print(f"SPF Record: {status.spf_record}")
        print(f"DNS Lookups: {status.spf_lookup_count}/10")
        print(f"Valid: {'Yes' if status.spf_valid else 'OVER LIMIT'}")

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-phishing-simulation-with-gophish/SKILL.md =====
---
name: performing-phishing-simulation-with-gophish
description: Deploy and run authorized phishing awareness campaigns with GoPhish, covering admin panel setup, SMTP sending profiles, email template and landing page creation, target user groups, and campaign reporting to measure click and credential-submission rates. Use when planning or executing a phishing simulation for employee security-awareness testing or measuring susceptibility to social engineering.
domain: cybersecurity
subdomain: phishing-defense
tags:
- phishing
- email-security
- social-engineering
- dmarc
- awareness
- gophish
- simulation
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
- DE.AE-02
mitre_attack:
- T1566
- T1598
- T1534
- T1036
mitre_f3:
  version: '1.1'
  tactics:
  - resource-development
  - initial-access
  - reconnaissance
  techniques:
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: T1598
    name: Phishing for Information
    tactic: reconnaissance
    source: attack
  - id: F1020.002
    name: 'Create Fake Materials: Fake Website'
    tactic: resource-development
    source: f3
  - id: T1583.001
    name: 'Acquire Infrastructure: Domains'
    tactic: resource-development
    source: attack
  - id: T1557
    name: Adversary-in-the-Middle
    tactic: initial-access
    source: attack
  - id: F1031
    name: Impersonate Account Holder
    tactic: initial-access
    source: f3
---
# Performing Phishing Simulation with GoPhish

## Overview
GoPhish is an open-source phishing simulation framework used by security teams to conduct authorized phishing awareness campaigns. It provides campaign management, email template creation, landing page cloning, and comprehensive reporting. This skill covers deploying GoPhish, creating realistic phishing scenarios, and analyzing campaign results to measure and improve organizational resilience.


## When to Use

- When conducting security assessments that involve performing phishing simulation with gophish
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites
- GoPhish binary or Docker image (https://github.com/gophish/gophish)
- SMTP server or relay for sending test emails
- Written authorization from management for phishing simulation
- Target email list (HR-approved)
- SSL/TLS certificate for landing pages
- Python 3.8+ for automation scripts

## Key Concepts

### GoPhish Architecture
- **Admin Panel**: Web UI for campaign management (default port 3333)
- **Phishing Server**: Serves landing pages and tracks clicks (default port 80/443)
- **SMTP Configuration**: Outbound email sending profile
- **Campaign Engine**: Orchestrates email delivery, tracking, and reporting

### Campaign Components
1. **Sending Profile**: SMTP server configuration for outbound email
2. **Email Template**: The phishing email content with tracking
3. **Landing Page**: The fake page users are directed to
4. **User Group**: Target recipients for the campaign
5. **Campaign**: Combines all components with scheduling

## Workflow

### Step 1: Deploy GoPhish
```bash
# Docker deployment
docker pull gophish/gophish
docker run -d --name gophish -p 3333:3333 -p 8080:80 gophish/gophish

# Or binary deployment
wget https://github.com/gophish/gophish/releases/latest/download/gophish-v0.12.1-linux-64bit.zip
unzip gophish-v0.12.1-linux-64bit.zip
chmod +x gophish
./gophish
```

### Step 2: Configure Sending Profile
- Name: "Internal Mail Server"
- SMTP From: awareness-test@yourdomain.com
- Host: smtp.yourdomain.com:587
- Username/Password: Service account credentials
- Enable TLS

### Step 3: Create Email Template
- Use realistic scenarios: password reset, IT notification, HR update
- Include GoPhish tracking pixel: `{{.Tracker}}`
- Include phishing link: `{{.URL}}`
- Personalize with `{{.FirstName}}`, `{{.LastName}}`, `{{.Position}}`

### Step 4: Create Landing Page
- Clone legitimate login page using GoPhish's import feature
- Enable credential capture (for authorized testing only)
- Configure redirect to training page after submission
- Add SSL certificate for HTTPS

### Step 5: Import Users and Launch Campaign
- Import CSV with: First Name, Last Name, Email, Position
- Set campaign schedule (stagger sends to avoid detection)
- Launch and monitor in real-time

### Step 6: Analyze Results with process.py
Use the automation script to pull campaign data via GoPhish API and generate detailed analytics reports.

## Tools & Resources
- **GoPhish**: https://getgophish.com/
- **GoPhish API Docs**: https://docs.getgophish.com/api-documentation/
- **GoPhish GitHub**: https://github.com/gophish/gophish
- **Evilginx2** (for advanced AiTM testing): https://github.com/kgretzky/evilginx2
- **King Phisher**: https://github.com/rsmusllp/king-phisher

## Validation
- Successfully deploy GoPhish and access admin panel
- Create and send a test phishing email to a test mailbox
- Capture simulated credentials on landing page
- Generate campaign report with open/click/submit rates
- Redirect users to awareness training after interaction

===== END FILE =====


===== FILE: skills/performing-phishing-simulation-with-gophish/LICENSE =====

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


===== FILE: skills/performing-phishing-simulation-with-gophish/assets/template.md =====
# Phishing Simulation Campaign Plan Template

## Campaign Information
- **Campaign Name**: [e.g., Q1-2026 Password Reset Simulation]
- **Campaign Owner**: [Security Awareness Team Lead]
- **Authorization Date**: [YYYY-MM-DD]
- **Authorized By**: [CISO / VP of Security]
- **Campaign Period**: [Start Date] to [End Date]

## Authorization Checklist
- [ ] Written authorization from executive management
- [ ] Legal review completed
- [ ] HR notification and approval
- [ ] IT operations notified (email gateway whitelisting)
- [ ] Privacy impact assessment completed
- [ ] Data handling procedures documented

## Campaign Objectives
| Objective | Target Metric |
|---|---|
| Measure click susceptibility | Click rate < 10% |
| Measure credential submission | Submit rate < 5% |
| Measure reporting behavior | Report rate > 50% |
| Identify high-risk departments | Department-level breakdown |

## Scenario Design
- **Pretext**: [Password reset / IT maintenance / HR policy update]
- **Sender**: [it-support@company-domain.com]
- **Subject Line**: [Urgent: Password Expiration Notice]
- **Call to Action**: [Click link to reset password]
- **Landing Page**: [Fake login page mimicking internal portal]
- **Post-Interaction**: [Redirect to training page]

## Target Audience
| Group | Count | Department | Difficulty Level |
|---|---|---|---|
| Group A | | | |
| Group B | | | |

## GoPhish Configuration
### Sending Profile
```json
{
  "name": "Campaign SMTP Profile",
  "host": "smtp.server.com:587",
  "from_address": "IT Support <it-support@domain.com>",
  "username": "service-account",
  "ignore_cert_errors": false
}
```

### Email Template Variables
- `{{.FirstName}}` - Recipient first name
- `{{.LastName}}` - Recipient last name
- `{{.Position}}` - Recipient job title
- `{{.Email}}` - Recipient email
- `{{.From}}` - Sender address
- `{{.URL}}` - Phishing URL (tracked)
- `{{.Tracker}}` - Tracking pixel
- `{{.TrackingURL}}` - Full tracking URL
- `{{.RId}}` - Unique recipient ID

## Success Criteria
| Metric | Poor | Fair | Good | Excellent |
|---|---|---|---|---|
| Click Rate | >25% | 15-25% | 5-15% | <5% |
| Submit Rate | >15% | 8-15% | 3-8% | <3% |
| Report Rate | <10% | 10-30% | 30-60% | >60% |

## Post-Campaign Actions
- [ ] Generate campaign report
- [ ] Force password reset for credential submitters
- [ ] Send targeted training to clickers
- [ ] Send positive reinforcement to reporters
- [ ] Present aggregate findings to leadership
- [ ] Update training curriculum based on results
- [ ] Plan next campaign with adjusted difficulty

===== END FILE =====


===== FILE: skills/performing-phishing-simulation-with-gophish/references/api-reference.md =====
# API Reference — Performing Phishing Simulation with GoPhish

## Libraries Used
- **requests**: HTTP client for GoPhish REST API

## CLI Interface
```
python agent.py --url https://gophish.local:3333 --api-key <key> campaigns
python agent.py --url <url> --api-key <key> metrics --id 1
python agent.py --url <url> --api-key <key> resources
python agent.py --url <url> --api-key <key> report --id 1
python agent.py --url <url> --api-key <key> launch --name "Q1 Test" --template-id 1 --page-id 1 --smtp-id 1 --group-ids 1 2 --phish-url https://phish.local
```

## GoPhishClient API Endpoints

### `GET /api/campaigns/` — List all campaigns
### `GET /api/campaigns/{id}` — Campaign details with results
### `POST /api/campaigns/` — Create and launch campaign
### `GET /api/groups/` — List target groups
### `GET /api/templates/` — List email templates
### `GET /api/smtp/` — List sending profiles

## Core Functions

### `get_campaign_metrics(...)` — Campaign performance analysis
Tracks: sent, opened, clicked, submitted, reported. Calculates percentage rates.

### `generate_report(...)` — Risk assessment with recommendations
Risk levels: CRITICAL (>10% credential submission), HIGH (>20% click rate), MEDIUM.

### `list_resources(...)` — Enumerate available GoPhish configurations

## Campaign Status Tracking
| Status | Description |
|--------|-------------|
| Email Sent | Email delivered to target |
| Email Opened | Tracking pixel loaded |
| Clicked Link | Target clicked phishing URL |
| Submitted Data | Target entered credentials |
| Reported | Target reported phishing email |

## Dependencies
```
pip install requests
```

===== END FILE =====


===== FILE: skills/performing-phishing-simulation-with-gophish/references/standards.md =====
# Standards & References: Phishing Simulation with GoPhish

## Legal & Compliance Framework
- **Computer Fraud and Abuse Act (CFAA)**: Ensure written authorization before conducting simulations
- **GDPR (EU)**: Data protection requirements for handling employee email addresses and click data
- **CCPA (California)**: Employee data privacy considerations
- **Company Acceptable Use Policy**: Must align simulation with organizational policies

## Industry Standards
- **NIST SP 800-50**: Building an Information Technology Security Awareness and Training Program
- **NIST SP 800-16**: Information Technology Security Training Requirements
- **SANS Security Awareness Maturity Model**: Five levels from non-existent to metrics framework
- **ISO 27001:2022**: A.6.3 - Information security awareness, education and training

## MITRE ATT&CK References
- **T1566.001**: Phishing: Spearphishing Attachment
- **T1566.002**: Phishing: Spearphishing Link
- **T1598**: Phishing for Information
- **T1204.001**: User Execution: Malicious Link
- **T1204.002**: User Execution: Malicious File

## GoPhish Technical Reference

### API Endpoints
| Endpoint | Method | Description |
|---|---|---|
| `/api/campaigns/` | GET | List all campaigns |
| `/api/campaigns/` | POST | Create new campaign |
| `/api/campaigns/{id}` | GET | Get campaign details |
| `/api/campaigns/{id}/results` | GET | Get campaign results |
| `/api/campaigns/{id}/summary` | GET | Get campaign summary |
| `/api/templates/` | GET/POST | Manage email templates |
| `/api/pages/` | GET/POST | Manage landing pages |
| `/api/smtp/` | GET/POST | Manage sending profiles |
| `/api/groups/` | GET/POST | Manage user groups |
| `/api/import/email` | POST | Import email template |
| `/api/import/site` | POST | Import/clone website |

### Campaign Event Types
| Event | Description |
|---|---|
| Email Sent | Email delivered to target |
| Email Opened | Tracking pixel loaded |
| Clicked Link | User clicked phishing URL |
| Submitted Data | User entered credentials |
| Email Reported | User reported via plugin |

## Phishing Simulation Best Practices
1. **Always obtain written authorization** from executive management
2. **Coordinate with IT/security teams** to whitelist simulation infrastructure
3. **Start with easier-to-identify phishing** and increase difficulty gradually
4. **Never punish employees** for failing - focus on education
5. **Provide immediate training** after user interaction
6. **Run campaigns regularly** (monthly/quarterly) for sustained awareness
7. **Vary scenarios** across campaign types (credential harvesting, attachment, link)
8. **Respect opt-outs** where legally required
9. **Protect campaign data** - treat click/submit data as sensitive
10. **Report metrics anonymously** when possible at department level

===== END FILE =====


===== FILE: skills/performing-phishing-simulation-with-gophish/references/workflows.md =====
# Workflows: Phishing Simulation with GoPhish

## Workflow 1: End-to-End Campaign Execution

```
Phase 1: Authorization & Planning
  |
  +-- Obtain written authorization from management
  +-- Define campaign objectives and success criteria
  +-- Select target groups (by department, role, risk level)
  +-- Choose phishing scenario (credential harvest, link click, attachment)
  +-- Set campaign timeline
  |
Phase 2: Infrastructure Setup
  |
  +-- Deploy GoPhish server (Docker or bare metal)
  +-- Configure SSL/TLS certificate for landing page
  +-- Set up SMTP sending profile
  +-- Whitelist GoPhish IP in email gateway
  +-- Configure DNS for phishing domain
  +-- Test email deliverability
  |
Phase 3: Content Creation
  |
  +-- Design email template with GoPhish variables
  +-- Create or clone landing page
  +-- Set up redirect to training page
  +-- Configure credential capture (if authorized)
  +-- Test with internal team first
  |
Phase 4: Target Preparation
  |
  +-- Import user list (CSV: first,last,email,position)
  +-- Segment into groups if needed
  +-- Verify email addresses are valid
  |
Phase 5: Campaign Launch
  |
  +-- Set send schedule (staggered over hours/days)
  +-- Launch campaign
  +-- Monitor real-time dashboard
  +-- Handle any delivery issues
  |
Phase 6: Analysis & Reporting
  |
  +-- Wait for campaign duration to complete
  +-- Export results via API
  +-- Generate analytics report
  +-- Present findings to stakeholders
  +-- Identify high-risk groups for targeted training
```

## Workflow 2: Progressive Difficulty Model

```
Quarter 1: Easy to Detect
  +-- Generic greeting, spelling errors
  +-- Unrelated external domain
  +-- Obvious call to action
  +-- Expected: < 20% click rate
  |
Quarter 2: Moderate Difficulty
  +-- Personalized with name/department
  +-- Look-alike domain
  +-- Relevant pretext (IT maintenance, HR policy)
  +-- Expected: < 15% click rate
  |
Quarter 3: Difficult
  +-- Highly targeted content
  +-- Convincing sender spoofing
  +-- Timely pretext (tax season, annual review)
  +-- Expected: < 10% click rate
  |
Quarter 4: Advanced
  +-- Spear-phishing with OSINT
  +-- Multi-step pretext
  +-- Mimics real vendor communication
  +-- Expected: < 5% click rate
```

## Workflow 3: Automated Campaign via API

```
[Python Script] --> GoPhish API
  |
  +-- POST /api/smtp/ (create sending profile)
  +-- POST /api/templates/ (create email template)
  +-- POST /api/pages/ (create landing page)
  +-- POST /api/groups/ (import target users)
  +-- POST /api/campaigns/ (launch campaign)
  |
  [Wait for campaign duration]
  |
  +-- GET /api/campaigns/{id}/summary
  +-- GET /api/campaigns/{id}/results
  |
  [Generate report with metrics]
  |
  +-- Calculate: open rate, click rate, submit rate, report rate
  +-- Compare against baseline and industry benchmarks
  +-- Export to PDF/HTML report
```

## Workflow 4: Post-Campaign Remediation

```
Campaign Results Available
  |
  v
[Identify users who submitted credentials]
  |
  +-- Immediately: Force password reset
  +-- Within 24h: Send targeted training content
  +-- Within 1 week: Manager notification (aggregate only)
  |
  v
[Identify users who clicked but did not submit]
  |
  +-- Send phishing awareness micro-training
  +-- Include specific red flags they missed
  |
  v
[Identify users who reported the email]
  |
  +-- Send positive reinforcement
  +-- Recognize in security champions program
  |
  v
[Aggregate department-level metrics]
  |
  +-- Present to leadership
  +-- Identify highest-risk departments
  +-- Plan targeted training interventions
  +-- Schedule next campaign
```

===== END FILE =====


===== FILE: skills/performing-phishing-simulation-with-gophish/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing phishing simulation campaigns with GoPhish API."""

import json
import os
import argparse
from datetime import datetime

try:
    import requests
    from requests.packages.urllib3.exceptions import InsecureRequestWarning
    requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
except ImportError:
    requests = None


class GoPhishClient:
    """Client for GoPhish REST API."""

    def __init__(self, base_url, api_key):
        self.base_url = base_url.rstrip("/")
        self.headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}

    def _get(self, path):
        resp = requests.get(f"{self.base_url}{path}", headers=self.headers, verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true", timeout=30)  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        resp.raise_for_status()
        return resp.json()

    def _post(self, path, data):
        resp = requests.post(f"{self.base_url}{path}", headers=self.headers, json=data, verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true", timeout=30)  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        resp.raise_for_status()
        return resp.json()

    def get_campaigns(self):
        return self._get("/api/campaigns/")

    def get_campaign(self, campaign_id):
        return self._get(f"/api/campaigns/{campaign_id}")

    def get_campaign_results(self, campaign_id):
        return self._get(f"/api/campaigns/{campaign_id}/results")

    def create_campaign(self, name, template_id, page_id, smtp_id, group_ids, url, launch_date=None):
        data = {"name": name, "template": {"id": template_id}, "page": {"id": page_id},
                "smtp": {"id": smtp_id}, "groups": [{"id": gid} for gid in group_ids], "url": url}
        if launch_date:
            data["launch_date"] = launch_date
        return self._post("/api/campaigns/", data)

    def get_groups(self):
        return self._get("/api/groups/")

    def get_templates(self):
        return self._get("/api/templates/")

    def get_sending_profiles(self):
        return self._get("/api/smtp/")


def list_campaigns(base_url, api_key):
    """List all phishing simulation campaigns."""
    client = GoPhishClient(base_url, api_key)
    campaigns = client.get_campaigns()
    return {
        "total_campaigns": len(campaigns),
        "campaigns": [{"id": c["id"], "name": c["name"], "status": c["status"],
                        "created_date": c.get("created_date"), "launch_date": c.get("launch_date")}
                       for c in campaigns],
    }


def get_campaign_metrics(base_url, api_key, campaign_id):
    """Get detailed metrics for a phishing campaign."""
    client = GoPhishClient(base_url, api_key)
    campaign = client.get_campaign(campaign_id)
    results = campaign.get("results", [])
    stats = {"total": len(results), "sent": 0, "opened": 0, "clicked": 0, "submitted": 0, "reported": 0}
    for r in results:
        status = r.get("status", "").lower()
        if "sent" in status or "email sent" in status:
            stats["sent"] += 1
        if "opened" in status or "email opened" in status:
            stats["opened"] += 1
        if "clicked" in status:
            stats["clicked"] += 1
        if "submitted" in status:
            stats["submitted"] += 1
        if "reported" in status:
            stats["reported"] += 1
    total = max(stats["sent"], 1)
    return {
        "campaign_id": campaign_id, "name": campaign.get("name"),
        "status": campaign.get("status"),
        "stats": stats,
        "rates": {
            "open_rate": round(stats["opened"] / total * 100, 1),
            "click_rate": round(stats["clicked"] / total * 100, 1),
            "submit_rate": round(stats["submitted"] / total * 100, 1),
            "report_rate": round(stats["reported"] / total * 100, 1),
        },
    }


def launch_campaign(base_url, api_key, name, template_id, page_id, smtp_id, group_ids, url):
    """Launch a new phishing simulation campaign."""
    client = GoPhishClient(base_url, api_key)
    result = client.create_campaign(name, template_id, page_id, smtp_id, group_ids, url)
    return {"campaign_created": True, "campaign": result}


def list_resources(base_url, api_key):
    """List available templates, groups, and sending profiles."""
    client = GoPhishClient(base_url, api_key)
    return {
        "groups": [{"id": g["id"], "name": g["name"], "targets": len(g.get("targets", []))}
                   for g in client.get_groups()],
        "templates": [{"id": t["id"], "name": t["name"]} for t in client.get_templates()],
        "sending_profiles": [{"id": s["id"], "name": s["name"], "host": s.get("host")}
                             for s in client.get_sending_profiles()],
    }


def generate_report(base_url, api_key, campaign_id):
    """Generate phishing simulation report with recommendations."""
    metrics = get_campaign_metrics(base_url, api_key, campaign_id)
    recommendations = []
    if metrics["rates"]["click_rate"] > 20:
        recommendations.append("HIGH click rate — mandatory security awareness training needed")
    if metrics["rates"]["submit_rate"] > 10:
        recommendations.append("CRITICAL — over 10% submitted credentials, implement MFA immediately")
    if metrics["rates"]["report_rate"] < 5:
        recommendations.append("Low report rate — train users on how to report phishing")
    return {
        "generated": datetime.utcnow().isoformat(),
        **metrics,
        "risk_level": "CRITICAL" if metrics["rates"]["submit_rate"] > 10 else "HIGH" if metrics["rates"]["click_rate"] > 20 else "MEDIUM",
        "recommendations": recommendations,
    }


def main():
    if not requests:
        print(json.dumps({"error": "requests not installed"}))
        return
    parser = argparse.ArgumentParser(description="GoPhish Phishing Simulation Agent")
    parser.add_argument("--url", required=True, help="GoPhish server URL")
    parser.add_argument("--api-key", required=True, help="GoPhish API key")
    sub = parser.add_subparsers(dest="command")
    sub.add_parser("campaigns", help="List campaigns")
    m = sub.add_parser("metrics", help="Campaign metrics")
    m.add_argument("--id", type=int, required=True)
    sub.add_parser("resources", help="List templates, groups, profiles")
    r = sub.add_parser("report", help="Generate campaign report")
    r.add_argument("--id", type=int, required=True)
    l = sub.add_parser("launch", help="Launch new campaign")
    l.add_argument("--name", required=True)
    l.add_argument("--template-id", type=int, required=True)
    l.add_argument("--page-id", type=int, required=True)
    l.add_argument("--smtp-id", type=int, required=True)
    l.add_argument("--group-ids", nargs="+", type=int, required=True)
    l.add_argument("--phish-url", required=True)
    args = parser.parse_args()
    if args.command == "campaigns":
        result = list_campaigns(args.url, args.api_key)
    elif args.command == "metrics":
        result = get_campaign_metrics(args.url, args.api_key, args.id)
    elif args.command == "resources":
        result = list_resources(args.url, args.api_key)
    elif args.command == "report":
        result = generate_report(args.url, args.api_key, args.id)
    elif args.command == "launch":
        result = launch_campaign(args.url, args.api_key, args.name, args.template_id,
                                 args.page_id, args.smtp_id, args.group_ids, args.phish_url)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-phishing-simulation-with-gophish/scripts/process.py =====
#!/usr/bin/env python3
"""
GoPhish Campaign Automation and Analytics

Automates phishing simulation campaigns via the GoPhish REST API.
Creates campaigns, monitors progress, and generates detailed analytics reports.

Usage:
    python process.py create --config campaign.json
    python process.py status --campaign-id 1
    python process.py report --campaign-id 1 --output report.html
    python process.py list
"""

import argparse
import json
import sys
import csv
import os
from datetime import datetime, timezone
from pathlib import Path
from dataclasses import dataclass, field
from typing import Optional
from collections import defaultdict

try:
    import requests
    HAS_REQUESTS = True
except ImportError:
    HAS_REQUESTS = False


GOPHISH_API_URL = os.environ.get("GOPHISH_API_URL", "https://localhost:3333")
GOPHISH_API_KEY = os.environ.get("GOPHISH_API_KEY", "")


class GoPhishClient:
    """Client for interacting with the GoPhish REST API."""

    def __init__(self, api_url: str, api_key: str, verify_ssl: bool = False):
        self.api_url = api_url.rstrip("/")
        self.api_key = api_key
        self.verify_ssl = verify_ssl
        self.session = requests.Session()
        self.session.headers.update({
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        })
        self.session.verify = verify_ssl

    def _get(self, endpoint: str) -> dict:
        resp = self.session.get(f"{self.api_url}{endpoint}")
        resp.raise_for_status()
        return resp.json()

    def _post(self, endpoint: str, data: dict) -> dict:
        resp = self.session.post(f"{self.api_url}{endpoint}", json=data)
        resp.raise_for_status()
        return resp.json()

    def _delete(self, endpoint: str) -> bool:
        resp = self.session.delete(f"{self.api_url}{endpoint}")
        resp.raise_for_status()
        return resp.status_code == 200

    # Sending Profiles
    def create_sending_profile(self, name: str, host: str, from_address: str,
                                username: str = "", password: str = "",
                                ignore_cert: bool = False) -> dict:
        data = {
            "name": name,
            "host": host,
            "from_address": from_address,
            "username": username,
            "password": password,
            "ignore_cert_errors": ignore_cert
        }
        return self._post("/api/smtp/", data)

    def list_sending_profiles(self) -> list:
        return self._get("/api/smtp/")

    # Email Templates
    def create_template(self, name: str, subject: str, html: str,
                        text: str = "", attachments: list = None) -> dict:
        data = {
            "name": name,
            "subject": subject,
            "html": html,
            "text": text,
            "attachments": attachments or []
        }
        return self._post("/api/templates/", data)

    def import_email(self, raw_email: str, convert_links: bool = True) -> dict:
        data = {
            "content": raw_email,
            "convert_links": convert_links
        }
        return self._post("/api/import/email", data)

    def list_templates(self) -> list:
        return self._get("/api/templates/")

    # Landing Pages
    def create_page(self, name: str, html: str, capture_credentials: bool = True,
                    capture_passwords: bool = False, redirect_url: str = "") -> dict:
        data = {
            "name": name,
            "html": html,
            "capture_credentials": capture_credentials,
            "capture_passwords": capture_passwords,
            "redirect_url": redirect_url
        }
        return self._post("/api/pages/", data)

    def import_site(self, url: str) -> dict:
        data = {"url": url, "include_resources": False}
        return self._post("/api/import/site", data)

    def list_pages(self) -> list:
        return self._get("/api/pages/")

    # User Groups
    def create_group(self, name: str, targets: list) -> dict:
        data = {
            "name": name,
            "targets": targets
        }
        return self._post("/api/groups/", data)

    def import_group_csv(self, name: str, csv_path: str) -> dict:
        targets = []
        with open(csv_path, "r", encoding="utf-8") as f:
            reader = csv.DictReader(f)
            for row in reader:
                target = {
                    "first_name": row.get("First Name", row.get("first_name", "")),
                    "last_name": row.get("Last Name", row.get("last_name", "")),
                    "email": row.get("Email", row.get("email", "")),
                    "position": row.get("Position", row.get("position", ""))
                }
                if target["email"]:
                    targets.append(target)
        return self.create_group(name, targets)

    def list_groups(self) -> list:
        return self._get("/api/groups/")

    # Campaigns
    def create_campaign(self, name: str, template_name: str, page_name: str,
                        smtp_name: str, group_names: list, url: str,
                        launch_date: str = "", send_by_date: str = "") -> dict:
        templates = self.list_templates()
        template = next((t for t in templates if t["name"] == template_name), None)

        pages = self.list_pages()
        page = next((p for p in pages if p["name"] == page_name), None)

        smtps = self.list_sending_profiles()
        smtp = next((s for s in smtps if s["name"] == smtp_name), None)

        groups_list = self.list_groups()
        groups = [g for g in groups_list if g["name"] in group_names]

        if not all([template, page, smtp, groups]):
            missing = []
            if not template:
                missing.append(f"template '{template_name}'")
            if not page:
                missing.append(f"page '{page_name}'")
            if not smtp:
                missing.append(f"smtp '{smtp_name}'")
            if not groups:
                missing.append(f"groups {group_names}")
            raise ValueError(f"Missing components: {', '.join(missing)}")

        data = {
            "name": name,
            "template": {"name": template_name},
            "page": {"name": page_name},
            "smtp": {"name": smtp_name},
            "groups": [{"name": g["name"]} for g in groups],
            "url": url
        }
        if launch_date:
            data["launch_date"] = launch_date
        if send_by_date:
            data["send_by_date"] = send_by_date

        return self._post("/api/campaigns/", data)

    def get_campaign(self, campaign_id: int) -> dict:
        return self._get(f"/api/campaigns/{campaign_id}")

    def get_campaign_summary(self, campaign_id: int) -> dict:
        return self._get(f"/api/campaigns/{campaign_id}/summary")

    def get_campaign_results(self, campaign_id: int) -> dict:
        return self._get(f"/api/campaigns/{campaign_id}/results")

    def list_campaigns(self) -> list:
        return self._get("/api/campaigns/")

    def complete_campaign(self, campaign_id: int) -> dict:
        return self._get(f"/api/campaigns/{campaign_id}/complete")


def calculate_campaign_metrics(results: dict) -> dict:
    """Calculate detailed campaign metrics from GoPhish results."""
    timeline = results.get("timeline", [])
    total_targets = len(results.get("results", []))

    events = defaultdict(set)
    for event in timeline:
        email = event.get("email", "")
        message = event.get("message", "")

        if "Email Sent" in message:
            events["sent"].add(email)
        elif "Email Opened" in message:
            events["opened"].add(email)
        elif "Clicked Link" in message:
            events["clicked"].add(email)
        elif "Submitted Data" in message:
            events["submitted"].add(email)
        elif "Email Reported" in message:
            events["reported"].add(email)

    sent = len(events["sent"])
    opened = len(events["opened"])
    clicked = len(events["clicked"])
    submitted = len(events["submitted"])
    reported = len(events["reported"])

    metrics = {
        "total_targets": total_targets,
        "emails_sent": sent,
        "emails_opened": opened,
        "links_clicked": clicked,
        "data_submitted": submitted,
        "emails_reported": reported,
        "open_rate": round(opened / max(sent, 1) * 100, 1),
        "click_rate": round(clicked / max(sent, 1) * 100, 1),
        "submit_rate": round(submitted / max(sent, 1) * 100, 1),
        "report_rate": round(reported / max(sent, 1) * 100, 1),
        "click_to_submit_rate": round(submitted / max(clicked, 1) * 100, 1),
        "resilience_score": round((1 - submitted / max(sent, 1)) * 100, 1),
    }

    # Department breakdown
    dept_stats = defaultdict(lambda: {"sent": 0, "opened": 0, "clicked": 0, "submitted": 0})
    for result in results.get("results", []):
        dept = result.get("position", "Unknown")
        email = result.get("email", "")
        dept_stats[dept]["sent"] += 1
        if email in events["opened"]:
            dept_stats[dept]["opened"] += 1
        if email in events["clicked"]:
            dept_stats[dept]["clicked"] += 1
        if email in events["submitted"]:
            dept_stats[dept]["submitted"] += 1

    metrics["department_breakdown"] = dict(dept_stats)

    return metrics


def generate_html_report(campaign: dict, metrics: dict) -> str:
    """Generate an HTML campaign report."""
    name = campaign.get("name", "Unknown Campaign")
    created = campaign.get("created_date", "")
    status = campaign.get("status", "")

    html = f"""<!DOCTYPE html>
<html>
<head>
<title>Phishing Simulation Report: {name}</title>
<style>
body {{ font-family: Arial, sans-serif; margin: 40px; background: #f5f5f5; }}
.container {{ max-width: 900px; margin: 0 auto; background: white; padding: 30px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }}
h1 {{ color: #2c3e50; border-bottom: 2px solid #3498db; padding-bottom: 10px; }}
h2 {{ color: #34495e; margin-top: 30px; }}
.metric-grid {{ display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; margin: 20px 0; }}
.metric-card {{ background: #f8f9fa; padding: 20px; border-radius: 6px; text-align: center; border-left: 4px solid #3498db; }}
.metric-value {{ font-size: 32px; font-weight: bold; color: #2c3e50; }}
.metric-label {{ font-size: 14px; color: #7f8c8d; margin-top: 5px; }}
.risk-high {{ border-left-color: #e74c3c; }}
.risk-medium {{ border-left-color: #f39c12; }}
.risk-low {{ border-left-color: #27ae60; }}
table {{ width: 100%; border-collapse: collapse; margin: 15px 0; }}
th {{ background: #34495e; color: white; padding: 12px; text-align: left; }}
td {{ padding: 10px; border-bottom: 1px solid #ecf0f1; }}
tr:hover {{ background: #f8f9fa; }}
.bar {{ height: 20px; background: #3498db; border-radius: 3px; }}
.bar-container {{ background: #ecf0f1; border-radius: 3px; overflow: hidden; }}
.footer {{ margin-top: 30px; padding-top: 15px; border-top: 1px solid #ecf0f1; color: #95a5a6; font-size: 12px; }}
</style>
</head>
<body>
<div class="container">
<h1>Phishing Simulation Report</h1>
<p><strong>Campaign:</strong> {name}<br>
<strong>Date:</strong> {created}<br>
<strong>Status:</strong> {status}<br>
<strong>Generated:</strong> {datetime.now(timezone.utc).strftime('%Y-%m-%d %H:%M UTC')}</p>

<h2>Campaign Metrics</h2>
<div class="metric-grid">
<div class="metric-card">
    <div class="metric-value">{metrics['emails_sent']}</div>
    <div class="metric-label">Emails Sent</div>
</div>
<div class="metric-card {'risk-medium' if metrics['open_rate'] > 50 else 'risk-low'}">
    <div class="metric-value">{metrics['open_rate']}%</div>
    <div class="metric-label">Open Rate</div>
</div>
<div class="metric-card {'risk-high' if metrics['click_rate'] > 20 else 'risk-medium' if metrics['click_rate'] > 10 else 'risk-low'}">
    <div class="metric-value">{metrics['click_rate']}%</div>
    <div class="metric-label">Click Rate</div>
</div>
<div class="metric-card {'risk-high' if metrics['submit_rate'] > 10 else 'risk-medium' if metrics['submit_rate'] > 5 else 'risk-low'}">
    <div class="metric-value">{metrics['submit_rate']}%</div>
    <div class="metric-label">Submit Rate</div>
</div>
<div class="metric-card risk-low">
    <div class="metric-value">{metrics['report_rate']}%</div>
    <div class="metric-label">Report Rate</div>
</div>
<div class="metric-card">
    <div class="metric-value">{metrics['resilience_score']}%</div>
    <div class="metric-label">Resilience Score</div>
</div>
</div>

<h2>Funnel Analysis</h2>
<table>
<tr><th>Stage</th><th>Count</th><th>Rate</th><th>Visual</th></tr>
<tr><td>Emails Sent</td><td>{metrics['emails_sent']}</td><td>100%</td>
    <td><div class="bar-container"><div class="bar" style="width:100%"></div></div></td></tr>
<tr><td>Emails Opened</td><td>{metrics['emails_opened']}</td><td>{metrics['open_rate']}%</td>
    <td><div class="bar-container"><div class="bar" style="width:{metrics['open_rate']}%"></div></div></td></tr>
<tr><td>Links Clicked</td><td>{metrics['links_clicked']}</td><td>{metrics['click_rate']}%</td>
    <td><div class="bar-container"><div class="bar" style="width:{metrics['click_rate']}%"></div></div></td></tr>
<tr><td>Data Submitted</td><td>{metrics['data_submitted']}</td><td>{metrics['submit_rate']}%</td>
    <td><div class="bar-container"><div class="bar" style="width:{metrics['submit_rate']}%"></div></div></td></tr>
<tr><td>Emails Reported</td><td>{metrics['emails_reported']}</td><td>{metrics['report_rate']}%</td>
    <td><div class="bar-container"><div class="bar" style="width:{metrics['report_rate']}%"></div></div></td></tr>
</table>

<h2>Department Breakdown</h2>
<table>
<tr><th>Department</th><th>Sent</th><th>Opened</th><th>Clicked</th><th>Submitted</th><th>Click Rate</th></tr>"""

    for dept, stats in sorted(metrics.get("department_breakdown", {}).items()):
        dept_click_rate = round(stats["clicked"] / max(stats["sent"], 1) * 100, 1)
        html += f"""
<tr><td>{dept}</td><td>{stats['sent']}</td><td>{stats['opened']}</td>
<td>{stats['clicked']}</td><td>{stats['submitted']}</td><td>{dept_click_rate}%</td></tr>"""

    html += f"""
</table>

<h2>Industry Benchmarks</h2>
<table>
<tr><th>Metric</th><th>Your Result</th><th>Industry Average</th><th>Target</th></tr>
<tr><td>Click Rate</td><td>{metrics['click_rate']}%</td><td>11-15%</td><td>&lt;5%</td></tr>
<tr><td>Submit Rate</td><td>{metrics['submit_rate']}%</td><td>3-5%</td><td>&lt;2%</td></tr>
<tr><td>Report Rate</td><td>{metrics['report_rate']}%</td><td>10-15%</td><td>&gt;70%</td></tr>
</table>

<h2>Recommendations</h2>
<ul>"""

    if metrics['click_rate'] > 20:
        html += "<li><strong>High Priority:</strong> Click rate exceeds 20%. Implement mandatory phishing awareness training for all employees.</li>"
    if metrics['submit_rate'] > 10:
        html += "<li><strong>Critical:</strong> Submit rate exceeds 10%. Deploy MFA across all applications to mitigate credential harvesting risk.</li>"
    if metrics['report_rate'] < 20:
        html += "<li><strong>Improve Reporting:</strong> Report rate is low. Deploy phishing report button in email client and incentivize reporting.</li>"

    html += """
</ul>

<div class="footer">
<p>This report was generated by the GoPhish Campaign Analytics tool.
Campaign data is confidential and should be handled according to organizational data protection policies.</p>
</div>
</div>
</body>
</html>"""

    return html


def generate_text_report(campaign: dict, metrics: dict) -> str:
    """Generate a text-based campaign report."""
    name = campaign.get("name", "Unknown")
    lines = []
    lines.append("=" * 60)
    lines.append("  PHISHING SIMULATION CAMPAIGN REPORT")
    lines.append("=" * 60)
    lines.append(f"  Campaign: {name}")
    lines.append(f"  Status: {campaign.get('status', '')}")
    lines.append(f"  Created: {campaign.get('created_date', '')}")
    lines.append("")
    lines.append("[METRICS]")
    lines.append(f"  Emails Sent:     {metrics['emails_sent']}")
    lines.append(f"  Emails Opened:   {metrics['emails_opened']} ({metrics['open_rate']}%)")
    lines.append(f"  Links Clicked:   {metrics['links_clicked']} ({metrics['click_rate']}%)")
    lines.append(f"  Data Submitted:  {metrics['data_submitted']} ({metrics['submit_rate']}%)")
    lines.append(f"  Emails Reported: {metrics['emails_reported']} ({metrics['report_rate']}%)")
    lines.append(f"  Resilience:      {metrics['resilience_score']}%")
    lines.append("")
    lines.append("[DEPARTMENT BREAKDOWN]")
    for dept, stats in sorted(metrics.get("department_breakdown", {}).items()):
        rate = round(stats["clicked"] / max(stats["sent"], 1) * 100, 1)
        lines.append(f"  {dept}: {stats['sent']} sent, {stats['clicked']} clicked ({rate}%)")
    lines.append("=" * 60)
    return "\n".join(lines)


def main():
    parser = argparse.ArgumentParser(description="GoPhish Campaign Automation")
    subparsers = parser.add_subparsers(dest="command")

    # Create campaign from config
    create_parser = subparsers.add_parser("create", help="Create campaign from config")
    create_parser.add_argument("--config", required=True, help="Campaign config JSON file")

    # Campaign status
    status_parser = subparsers.add_parser("status", help="Get campaign status")
    status_parser.add_argument("--campaign-id", type=int, required=True)

    # Generate report
    report_parser = subparsers.add_parser("report", help="Generate campaign report")
    report_parser.add_argument("--campaign-id", type=int, required=True)
    report_parser.add_argument("--output", "-o", help="Output file path")
    report_parser.add_argument("--format", choices=["html", "text", "json"], default="text")

    # List campaigns
    subparsers.add_parser("list", help="List all campaigns")

    # Import users
    import_parser = subparsers.add_parser("import-users", help="Import user group from CSV")
    import_parser.add_argument("--csv", required=True, help="CSV file path")
    import_parser.add_argument("--group-name", required=True, help="Group name")

    parser.add_argument("--api-url", default=GOPHISH_API_URL)
    parser.add_argument("--api-key", default=GOPHISH_API_KEY)
    parser.add_argument("--no-verify-ssl", action="store_true")

    args = parser.parse_args()

    if not HAS_REQUESTS:
        print("Error: 'requests' library required. Install with: pip install requests",
              file=sys.stderr)
        sys.exit(1)

    api_url = args.api_url
    api_key = args.api_key

    if not api_key:
        print("Error: GoPhish API key required. Set GOPHISH_API_KEY env var or use --api-key",
              file=sys.stderr)
        sys.exit(1)

    client = GoPhishClient(api_url, api_key, verify_ssl=not args.no_verify_ssl)

    if args.command == "create":
        with open(args.config, "r") as f:
            config = json.load(f)
        result = client.create_campaign(**config)
        print(f"Campaign created: ID={result.get('id')}, Name={result.get('name')}")

    elif args.command == "status":
        summary = client.get_campaign_summary(args.campaign_id)
        print(json.dumps(summary, indent=2))

    elif args.command == "report":
        campaign = client.get_campaign(args.campaign_id)
        results = client.get_campaign_results(args.campaign_id)
        metrics = calculate_campaign_metrics(results)

        if args.format == "html":
            output = generate_html_report(campaign, metrics)
        elif args.format == "json":
            output = json.dumps({"campaign": campaign, "metrics": metrics}, indent=2, default=str)
        else:
            output = generate_text_report(campaign, metrics)

        if args.output:
            with open(args.output, "w", encoding="utf-8") as f:
                f.write(output)
            print(f"Report written to {args.output}")
        else:
            print(output)

    elif args.command == "list":
        campaigns = client.list_campaigns()
        for c in campaigns:
            print(f"  ID: {c['id']} | Name: {c['name']} | Status: {c['status']} | "
                  f"Created: {c.get('created_date', '')}")

    elif args.command == "import-users":
        result = client.import_group_csv(args.group_name, args.csv)
        targets = result.get("targets", [])
        print(f"Group '{args.group_name}' created with {len(targets)} targets")

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====
