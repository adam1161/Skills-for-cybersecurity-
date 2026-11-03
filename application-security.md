

===== FILE: skills/implementing-devsecops-security-scanning/SKILL.md =====
---
name: implementing-devsecops-security-scanning
description: 'Integrates SAST, DAST, and SCA into CI/CD pipelines using Semgrep for
  SAST, Trivy for SCA and container scanning, OWASP ZAP for DAST, and Gitleaks for
  secrets detection. Use when setting up automated security scanning in CI/CD, shifting
  security left, meeting compliance mandates (SOC 2, PCI-DSS, ISO 27001), or gating
  deployments on critical vulnerabilities.

  '
domain: cybersecurity
subdomain: application-security
tags:
- devsecops
- SAST
- DAST
- SCA
- semgrep
- trivy
- owasp-zap
- gitleaks
- CI-CD
- shift-left
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-04
- ID.RA-01
- PR.DS-10
mitre_attack:
- T1078
- T1190
- T1059
- T1610
- T1611
---

# Implementing DevSecOps Security Scanning

## When to Use

- Setting up automated security scanning in a new or existing CI/CD pipeline
- Shifting security left by catching vulnerabilities before code reaches production
- Meeting compliance requirements (SOC 2, PCI-DSS, ISO 27001) that mandate automated security testing
- Integrating SAST, DAST, and SCA together to achieve comprehensive application security coverage
- Establishing security gates that block deployments containing critical or high-severity vulnerabilities

**Do not use** as a replacement for manual penetration testing. Automated scanning catches common vulnerability patterns but cannot replace human-driven security assessments for business logic flaws and complex attack chains.

## Prerequisites

- CI/CD platform: GitHub Actions, GitLab CI, Jenkins, or Azure DevOps
- Container runtime (Docker) for running scanning tools
- A staging environment URL for DAST scanning (DAST cannot test static code)
- Repository access with permissions to modify CI/CD workflow files
- Tool-specific requirements:
  - Semgrep: free for open-source rulesets (`p/security-audit`, `p/owasp-top-ten`)
  - Trivy: free, no account required
  - OWASP ZAP: free, Docker image available
  - Gitleaks: free, no account required

## Workflow

### Step 1: Add Secrets Detection with Gitleaks

Secrets detection runs first because leaked credentials are the highest-priority finding. Add to `.github/workflows/security.yml`:

```yaml
name: DevSecOps Security Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  secrets-scan:
    name: Secrets Detection (Gitleaks)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for scanning all commits

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Configure `.gitleaks.toml` in the repository root for custom rules and allowlists:

```toml
[extend]
useDefault = true

[allowlist]
description = "Global allowlist"
paths = [
  '''\.gitleaks\.toml''',
  '''test/fixtures/.*''',
  '''docs/examples/.*'''
]

[[rules]]
id = "custom-internal-api-key"
description = "Internal API key pattern"
regex = '''INTERNAL_KEY_[A-Za-z0-9]{32}'''
tags = ["internal", "api-key"]
```

### Step 2: Add SAST Scanning with Semgrep

Semgrep performs static code analysis to find security vulnerabilities, bugs, and code patterns:

```yaml
  sast-scan:
    name: SAST (Semgrep)
    runs-on: ubuntu-latest
    container:
      image: semgrep/semgrep
    steps:
      - uses: actions/checkout@v4

      - name: Run Semgrep SAST scan
        run: |
          semgrep scan \
            --config p/security-audit \
            --config p/owasp-top-ten \
            --config p/secrets \
            --severity ERROR \
            --error \
            --json \
            --output semgrep-results.json \
            .

      - name: Upload SAST results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: semgrep-results
          path: semgrep-results.json
```

For custom rules, create `.semgrep/custom-rules.yml`:

```yaml
rules:
  - id: no-exec-user-input
    patterns:
      - pattern: exec($INPUT)
      - pattern-not: exec("...")
    message: >
      User input passed to exec(). This is a command injection vulnerability.
    severity: ERROR
    languages: [python]
    metadata:
      cwe: "CWE-78: OS Command Injection"
      owasp: "A03:2021 - Injection"

  - id: no-raw-sql-queries
    patterns:
      - pattern: cursor.execute(f"...")
      - pattern: cursor.execute("..." + ...)
    message: >
      SQL query built with string concatenation or f-strings. Use parameterized queries.
    severity: ERROR
    languages: [python]
    metadata:
      cwe: "CWE-89: SQL Injection"
      owasp: "A03:2021 - Injection"
```

### Step 3: Add SCA Scanning with Trivy

Trivy scans dependencies, container images, IaC files, and generates SBOM:

```yaml
  sca-scan:
    name: SCA & Container Scan (Trivy)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy filesystem scan (dependencies)
        uses: aquasecurity/trivy-action@0.28.0
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
          format: 'json'
          output: 'trivy-fs-results.json'

      - name: Run Trivy IaC scan (Terraform, CloudFormation)
        uses: aquasecurity/trivy-action@0.28.0
        with:
          scan-type: 'config'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
          format: 'json'
          output: 'trivy-iac-results.json'

      - name: Upload SCA results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: trivy-results
          path: trivy-*.json

  container-scan:
    name: Container Image Scan (Trivy)
    runs-on: ubuntu-latest
    needs: [sast-scan]  # Build image only after SAST passes
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t app:${{ github.sha }} .

      - name: Scan container image
        uses: aquasecurity/trivy-action@0.28.0
        with:
          image-ref: 'app:${{ github.sha }}'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
          format: 'json'
          output: 'trivy-image-results.json'

      - name: Generate SBOM
        uses: aquasecurity/trivy-action@0.28.0
        with:
          image-ref: 'app:${{ github.sha }}'
          format: 'cyclonedx'
          output: 'sbom.json'

      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.json
```

### Step 4: Add DAST Scanning with OWASP ZAP

DAST runs against a deployed staging environment. It is slower than SAST/SCA and should run asynchronously or on a schedule:

```yaml
  dast-scan:
    name: DAST (OWASP ZAP)
    runs-on: ubuntu-latest
    needs: [deploy-staging]  # Must run after app is deployed to staging
    steps:
      - uses: actions/checkout@v4

      - name: Run ZAP Baseline Scan (fast, suitable for CI)
        uses: zaproxy/action-baseline@v0.14.0
        with:
          target: ${{ vars.STAGING_URL }}
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a -j'

      # For nightly full scans, use action-full-scan instead:
      # - name: Run ZAP Full Scan (comprehensive, 30-60 min)
      #   uses: zaproxy/action-full-scan@v0.12.0
      #   with:
      #     target: ${{ vars.STAGING_URL }}
```

Create `.zap/rules.tsv` to configure alert thresholds:

```tsv
10010	IGNORE	(Cookie No HttpOnly Flag - acceptable for non-sensitive cookies)
10011	IGNORE	(Cookie Without Secure Flag - staging uses HTTP)
90033	WARN	(Loosely Scoped Cookie)
10038	FAIL	(Content Security Policy Header Not Set)
40012	FAIL	(Cross Site Scripting - Reflected)
40014	FAIL	(Cross Site Scripting - Persistent)
40018	FAIL	(SQL Injection)
90019	FAIL	(Server Side Code Injection)
90020	FAIL	(Remote OS Command Injection)
```

### Step 5: Aggregate Results and Enforce Security Gates

Create a summary job that aggregates all scan results and enforces pass/fail gates:

```yaml
  security-gate:
    name: Security Gate
    runs-on: ubuntu-latest
    needs: [secrets-scan, sast-scan, sca-scan, container-scan]
    if: always()
    steps:
      - name: Check scan results
        run: |
          echo "Checking security scan results..."

          # Fail the pipeline if any upstream job failed
          if [[ "${{ needs.secrets-scan.result }}" == "failure" ]]; then
            echo "BLOCKED: Secrets detected in repository"
            exit 1
          fi

          if [[ "${{ needs.sast-scan.result }}" == "failure" ]]; then
            echo "BLOCKED: SAST found critical/high vulnerabilities"
            exit 1
          fi

          if [[ "${{ needs.sca-scan.result }}" == "failure" ]]; then
            echo "BLOCKED: SCA found critical/high vulnerable dependencies"
            exit 1
          fi

          if [[ "${{ needs.container-scan.result }}" == "failure" ]]; then
            echo "BLOCKED: Container image has critical/high vulnerabilities"
            exit 1
          fi

          echo "All security gates passed"
```

### Step 6: Configure Branch Protection Rules

Enforce the security pipeline as a required status check:

```
GitHub Repository > Settings > Branches > Branch Protection Rules

Branch name pattern: main
  Require status checks to pass before merging: Enabled
    Required status checks:
      - Secrets Detection (Gitleaks)
      - SAST (Semgrep)
      - SCA & Container Scan (Trivy)
      - Security Gate
  Require branches to be up to date before merging: Enabled
```

### Step 7: Set Up Developer Feedback Loop

Configure pre-commit hooks so developers catch issues before pushing:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.22.1
    hooks:
      - id: gitleaks

  - repo: https://github.com/semgrep/semgrep
    rev: v1.102.0
    hooks:
      - id: semgrep
        args: ['--config', 'p/security-audit', '--config', 'p/owasp-top-ten', '--error']
```

Install and activate pre-commit:

```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files  # Test against existing codebase
```

## Key Concepts

| Term | Definition |
|------|------------|
| **SAST (Static Application Security Testing)** | Analyzes source code without executing it to find security vulnerabilities; runs fast, catches issues early, but cannot find runtime flaws |
| **DAST (Dynamic Application Security Testing)** | Tests a running application by sending requests and analyzing responses; finds runtime issues but requires a deployed environment |
| **SCA (Software Composition Analysis)** | Scans project dependencies against vulnerability databases (NVD, GitHub Advisory) to find known-vulnerable libraries |
| **SBOM (Software Bill of Materials)** | Machine-readable inventory of all components and dependencies in an application, used for vulnerability tracking and compliance |
| **Shift Left** | Security practice of moving security testing earlier in the SDLC, from post-deployment to pre-commit and CI stages |
| **Security Gate** | A CI/CD pipeline checkpoint that blocks deployment if security scan results exceed defined severity thresholds |
| **Pre-commit Hook** | Local Git hook that runs security checks before code is committed, providing the fastest developer feedback loop |

## Verification

- [ ] Gitleaks blocks commits and PRs containing hardcoded secrets (test with a dummy API key)
- [ ] Semgrep scan runs on every PR and reports findings as annotations or comments
- [ ] Trivy filesystem scan detects a known-vulnerable dependency (test by adding a vulnerable package)
- [ ] Trivy container scan runs successfully against the built Docker image
- [ ] SBOM is generated and stored as a build artifact in CycloneDX or SPDX format
- [ ] OWASP ZAP baseline scan runs against the staging URL without crashing
- [ ] Security gate job blocks merges to main when any scan finds critical/high severity issues
- [ ] Branch protection rules enforce required status checks before merge
- [ ] Pre-commit hooks catch secrets and SAST findings locally before push
- [ ] Developer documentation explains how to interpret scan results and fix common findings

===== END FILE =====


===== FILE: skills/implementing-devsecops-security-scanning/LICENSE =====

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


===== FILE: skills/implementing-devsecops-security-scanning/references/api-reference.md =====
# API Reference: DevSecOps Security Scanning

## Semgrep CLI (SAST)
```bash
# Scan with auto-detected rules
semgrep scan --config auto --json /path/to/code

# Scan with specific ruleset
semgrep scan --config p/owasp-top-ten --json /path/to/code

# Custom rule file
semgrep scan --config my_rules.yaml --json /path/to/code

# SARIF output for GitHub integration
semgrep scan --config auto --sarif -o results.sarif /path/to/code
```

## Trivy CLI (SCA / Container)
```bash
# Scan container image
trivy image --format json --quiet nginx:latest

# Scan filesystem for vulnerabilities
trivy fs --format json --scanners vuln,secret /path/to/project

# Scan with severity filter
trivy image --severity CRITICAL,HIGH --format json myapp:latest

# Scan IaC files
trivy config --format json /path/to/terraform/
```

## Gitleaks CLI (Secret Detection)
```bash
# Detect secrets in git repo
gitleaks detect --source /path/to/repo --report-format json --report-path report.json

# Scan specific commit range
gitleaks detect --source . --log-opts="HEAD~10..HEAD" --report-format json

# Protect mode (pre-commit)
gitleaks protect --staged --report-format json
```

## CI/CD Pipeline Gate Logic
| Severity | Exit Code | Action |
|----------|-----------|--------|
| CRITICAL | 1 (fail) | Block merge/deploy |
| HIGH | 1 (fail) | Block merge/deploy |
| MEDIUM | 0 (warn) | Warning in PR comment |
| LOW | 0 (pass) | Informational only |

## JSON Output Schema (Semgrep)
| Field | Description |
|-------|------------|
| results[].check_id | Rule identifier |
| results[].extra.severity | ERROR, WARNING, INFO |
| results[].path | Affected file path |
| results[].start.line | Line number |
| results[].extra.message | Finding description |

## JSON Output Schema (Trivy)
| Field | Description |
|-------|------------|
| Results[].Target | Scanned target name |
| Results[].Vulnerabilities[].VulnerabilityID | CVE identifier |
| Results[].Vulnerabilities[].Severity | CRITICAL/HIGH/MEDIUM/LOW |
| Results[].Vulnerabilities[].FixedVersion | Version with fix |

===== END FILE =====


===== FILE: skills/implementing-devsecops-security-scanning/scripts/agent.py =====
#!/usr/bin/env python3
"""DevSecOps security scanning pipeline agent.

Orchestrates Semgrep (SAST), Trivy (container/SCA), and Gitleaks (secrets)
scans via subprocess, aggregates findings, and enforces severity gates.
"""

import argparse
import json
import subprocess
import sys
import datetime


SEVERITY_ORDER = {"CRITICAL": 4, "HIGH": 3, "MEDIUM": 2, "LOW": 1, "INFO": 0}


def run_semgrep(target_dir, config="auto"):
    """Run Semgrep SAST scan and return findings."""
    cmd = ["semgrep", "scan", "--config", config, "--json", "--quiet", target_dir]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        data = json.loads(result.stdout) if result.stdout.strip() else {}
        findings = []
        for r in data.get("results", []):
            findings.append({
                "tool": "semgrep",
                "rule_id": r.get("check_id", ""),
                "severity": r.get("extra", {}).get("severity", "WARNING").upper(),
                "message": r.get("extra", {}).get("message", ""),
                "file": r.get("path", ""),
                "line": r.get("start", {}).get("line", 0),
            })
        return findings
    except FileNotFoundError:
        return [{"tool": "semgrep", "error": "semgrep not installed"}]
    except subprocess.TimeoutExpired:
        return [{"tool": "semgrep", "error": "scan timed out"}]
    except json.JSONDecodeError:
        return [{"tool": "semgrep", "error": "invalid JSON output"}]


def run_trivy(image_or_path, scan_type="image"):
    """Run Trivy vulnerability scan on image or filesystem."""
    cmd = ["trivy", scan_type, "--format", "json", "--quiet", image_or_path]
    if scan_type == "fs":
        cmd = ["trivy", "fs", "--format", "json", "--quiet", "--scanners", "vuln,secret", image_or_path]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        data = json.loads(result.stdout) if result.stdout.strip() else {}
        findings = []
        for res in data.get("Results", []):
            for vuln in res.get("Vulnerabilities", []):
                findings.append({
                    "tool": "trivy",
                    "rule_id": vuln.get("VulnerabilityID", ""),
                    "severity": vuln.get("Severity", "UNKNOWN").upper(),
                    "message": vuln.get("Title", ""),
                    "file": res.get("Target", ""),
                    "line": 0,
                    "fixed_version": vuln.get("FixedVersion", ""),
                })
        return findings
    except FileNotFoundError:
        return [{"tool": "trivy", "error": "trivy not installed"}]
    except (subprocess.TimeoutExpired, json.JSONDecodeError) as e:
        return [{"tool": "trivy", "error": str(e)}]


def run_gitleaks(repo_path):
    """Run Gitleaks secret detection scan."""
    cmd = ["gitleaks", "detect", "--source", repo_path, "--report-format", "json",
           "--report-path", "/dev/stdout", "--no-banner"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
        data = json.loads(result.stdout) if result.stdout.strip().startswith("[") else []
        findings = []
        for leak in data:
            findings.append({
                "tool": "gitleaks",
                "rule_id": leak.get("RuleID", ""),
                "severity": "HIGH",
                "message": leak.get("Description", ""),
                "file": leak.get("File", ""),
                "line": leak.get("StartLine", 0),
            })
        return findings
    except FileNotFoundError:
        return [{"tool": "gitleaks", "error": "gitleaks not installed"}]
    except (subprocess.TimeoutExpired, json.JSONDecodeError) as e:
        return [{"tool": "gitleaks", "error": str(e)}]


def enforce_gate(findings, fail_on="HIGH"):
    """Enforce security gate based on severity threshold."""
    threshold = SEVERITY_ORDER.get(fail_on.upper(), 3)
    blockers = [
        f for f in findings
        if not f.get("error") and SEVERITY_ORDER.get(f.get("severity", ""), 0) >= threshold
    ]
    return {
        "gate_threshold": fail_on,
        "blocking_findings": len(blockers),
        "passed": len(blockers) == 0,
    }


def aggregate_report(all_findings):
    """Aggregate findings from all scanners."""
    by_severity = {}
    by_tool = {}
    for f in all_findings:
        if f.get("error"):
            continue
        sev = f.get("severity", "UNKNOWN")
        tool = f.get("tool", "unknown")
        by_severity[sev] = by_severity.get(sev, 0) + 1
        by_tool[tool] = by_tool.get(tool, 0) + 1
    return {
        "total_findings": sum(1 for f in all_findings if not f.get("error")),
        "by_severity": by_severity,
        "by_tool": by_tool,
    }


def main():
    parser = argparse.ArgumentParser(description="DevSecOps security scanning pipeline")
    parser.add_argument("target", nargs="?", help="Directory or container image to scan")
    parser.add_argument("--semgrep", action="store_true", help="Run Semgrep SAST")
    parser.add_argument("--trivy", action="store_true", help="Run Trivy vulnerability scan")
    parser.add_argument("--trivy-type", default="fs", choices=["image", "fs"], help="Trivy scan type")
    parser.add_argument("--gitleaks", action="store_true", help="Run Gitleaks secret detection")
    parser.add_argument("--fail-on", default="HIGH", help="Fail gate on severity (default: HIGH)")
    parser.add_argument("--output", "-o", help="Output JSON report path")
    args = parser.parse_args()

    print("[*] DevSecOps Security Scanning Pipeline")
    report = {"timestamp": datetime.datetime.utcnow().isoformat() + "Z", "findings": []}

    if not args.target:
        print("[DEMO] Usage: python agent.py /path/to/code --semgrep --trivy --gitleaks")
        print("  Tools: semgrep (SAST), trivy (SCA/container), gitleaks (secrets)")
        print(json.dumps({"demo": True, "tools": ["semgrep", "trivy", "gitleaks"]}, indent=2))
        sys.exit(0)

    if args.semgrep:
        report["findings"].extend(run_semgrep(args.target))
    if args.trivy:
        report["findings"].extend(run_trivy(args.target, args.trivy_type))
    if args.gitleaks:
        report["findings"].extend(run_gitleaks(args.target))

    summary = aggregate_report(report["findings"])
    gate = enforce_gate(report["findings"], args.fail_on)
    report["summary"] = summary
    report["gate"] = gate

    print(f"[*] Total findings: {summary['total_findings']}")
    print(f"[*] By severity: {summary['by_severity']}")
    print(f"[*] Gate ({args.fail_on}): {'PASSED' if gate['passed'] else 'FAILED'}")

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)

    print(json.dumps({"gate_passed": gate["passed"], "total": summary["total_findings"]}, indent=2))
    sys.exit(0 if gate["passed"] else 1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-runtime-application-self-protection/SKILL.md =====
---
name: implementing-runtime-application-self-protection
description: Deploy Runtime Application Self-Protection (RASP) agents to detect and
  block attacks from within application runtime, covering OpenRASP integration, attack
  pattern detection, and security policy configuration for Java and Python web applications.
domain: cybersecurity
subdomain: application-security
tags:
- rasp
- application-security
- openrasp
- runtime-protection
- sqli
- xss
- rce
- devsecops
version: '1.0'
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- GOVERN-1.1
- MEASURE-2.7
- MANAGE-3.1
nist_csf:
- PR.PS-01
- PR.PS-04
- ID.RA-01
- PR.DS-10
mitre_attack:
- T1078
- T1190
- T1059
- T1059.007
---

# Implementing Runtime Application Self-Protection

## Overview

Runtime Application Self-Protection (RASP) instruments application code at runtime to detect and block attacks by examining actual execution context rather than relying solely on network traffic patterns. Unlike WAFs that inspect HTTP requests externally, RASP agents intercept dangerous operations (SQL queries, file operations, command execution, deserialization) at the function level inside the application, achieving near-zero false positives. This skill covers deploying OpenRASP for Java applications, configuring detection policies for OWASP Top 10 attacks, tuning alerting thresholds, and integrating RASP telemetry with SIEM platforms.


## When to Use

- When deploying or configuring implementing runtime application self protection capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Java 8+ application server (Tomcat, Spring Boot, or JBoss) or Python Flask/Django application
- OpenRASP agent package (rasp-java or equivalent)
- OpenRASP management console for centralized policy management
- SIEM integration endpoint (Splunk HEC, Elasticsearch, or syslog)
- Application staging environment for RASP testing before production

## Steps

### Step 1: Deploy RASP Agent

Install the RASP agent into the application server runtime using JVM agent attachment for Java or middleware hooks for Python.

### Step 2: Configure Detection Policies

Define detection rules for SQL injection, command injection, SSRF, path traversal, XXE, and deserialization attacks with block or monitor actions.

### Step 3: Tune and Baseline

Run the agent in monitor mode during normal operations to establish baseline behavior and tune policies to reduce false positives before switching to block mode.

### Step 4: Integrate with SIEM

Forward RASP alerts to the SIEM for correlation with WAF, IDS, and authentication events to build comprehensive attack timelines.

## Expected Output

JSON report containing RASP policy audit results, detected attack attempts with stack traces, blocked requests summary, and coverage assessment against OWASP Top 10.

===== END FILE =====


===== FILE: skills/implementing-runtime-application-self-protection/LICENSE =====

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


===== FILE: skills/implementing-runtime-application-self-protection/references/api-reference.md =====
# API Reference: Implementing Runtime Application Self-Protection

## OpenRASP Deployment

```bash
# Java agent attachment (Tomcat)
export CATALINA_OPTS="-javaagent:/opt/rasp/rasp.jar"

# Spring Boot
java -javaagent:/opt/rasp/rasp.jar -jar app.jar

# Verify agent loaded
curl -s http://localhost:8080/rasp/status
```

## OpenRASP Configuration (rasp.yaml)

```yaml
# Detection plugin settings
plugin:
  timeout: 100        # Plugin execution timeout (ms)
  maxstack: 100       # Max stack frames to capture

# Block vs monitor mode
block:
  status_code: 302    # HTTP status on block
  redirect_url: /blocked.html

# Log settings
log:
  maxstack: 50
  syslog:
    enable: true
    url: "udp://siem.example.com:514"
    tag: openrasp
```

## Detection Hooks

| Hook Point | Attack Type | OWASP |
|------------|-------------|-------|
| sql_query | SQL Injection | A03:2021 |
| command_exec | OS Command Injection | A03:2021 |
| file_open / file_read | Path Traversal | A01:2021 |
| http_request | SSRF | A10:2021 |
| xml_parse | XXE | A05:2021 |
| deserialize | Insecure Deserialization | A08:2021 |
| response_write | XSS (reflected) | A03:2021 |
| ldap_query | LDAP Injection | A03:2021 |
| code_eval | Remote Code Execution | A03:2021 |

## Attack Log Format (JSON)

```json
{
  "attack_type": "sqli",
  "action": "block",
  "client_ip": "192.168.1.50",
  "request_url": "/api/users?id=1 OR 1=1",
  "attack_params": {"id": "1 OR 1=1"},
  "stack_trace": "com.app.UserDAO.findById(UserDAO.java:42)",
  "plugin_name": "sql_injection",
  "timestamp": "2025-06-15T10:30:00Z"
}
```

## RASP vs WAF Comparison

| Feature | WAF | RASP |
|---------|-----|------|
| Inspection point | Network perimeter | Inside application |
| Context | HTTP request only | Request + execution context |
| False positive rate | Higher | Near-zero |
| Encrypted traffic | Requires TLS termination | Sees plaintext |
| Deployment | Network device/proxy | Application library |

## Python RASP (Flask Middleware Example)

```python
from flask import request, abort
import re

SQL_INJECTION_PATTERN = re.compile(
    r"(\b(union|select|insert|update|delete|drop|alter)\b.*\b(from|into|table|set)\b)",
    re.IGNORECASE
)

@app.before_request
def rasp_check():
    for value in request.values.values():
        if SQL_INJECTION_PATTERN.search(str(value)):
            abort(403, description="RASP: SQL injection blocked")
```

### References

- OpenRASP: https://github.com/baidu/openrasp
- OWASP RASP: https://owasp.org/www-community/Runtime_Application_Self-Protection
- NIST AppSec: https://csrc.nist.gov/publications/detail/sp/800-95/final

===== END FILE =====


===== FILE: skills/implementing-runtime-application-self-protection/scripts/agent.py =====
#!/usr/bin/env python3
"""RASP Agent - audits runtime application protection config, attack logs, and coverage."""

import json
import argparse
import logging
from collections import defaultdict
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

OWASP_TOP10_CHECKS = {
    "sqli": {"name": "SQL Injection", "owasp": "A03:2021", "hook": "sql_query"},
    "cmdi": {"name": "Command Injection", "owasp": "A03:2021", "hook": "command_exec"},
    "ssrf": {"name": "Server-Side Request Forgery", "owasp": "A10:2021", "hook": "http_request"},
    "path_traversal": {"name": "Path Traversal", "owasp": "A01:2021", "hook": "file_open"},
    "xxe": {"name": "XML External Entity", "owasp": "A05:2021", "hook": "xml_parse"},
    "deserialization": {"name": "Insecure Deserialization", "owasp": "A08:2021", "hook": "deserialize"},
    "xss": {"name": "Cross-Site Scripting", "owasp": "A03:2021", "hook": "response_write"},
    "ldap_injection": {"name": "LDAP Injection", "owasp": "A03:2021", "hook": "ldap_query"},
    "rce": {"name": "Remote Code Execution", "owasp": "A03:2021", "hook": "code_eval"},
}


def parse_rasp_config(config_file):
    """Parse OpenRASP or RASP agent configuration."""
    with open(config_file) as f:
        return json.load(f)


def audit_rasp_policy(config):
    """Audit RASP detection policies against OWASP Top 10 coverage."""
    findings = []
    enabled_checks = config.get("detection_plugins", {})
    for check_id, check_info in OWASP_TOP10_CHECKS.items():
        plugin = enabled_checks.get(check_id, {})
        if not plugin.get("enabled", False):
            findings.append({
                "check": check_id, "name": check_info["name"],
                "owasp": check_info["owasp"], "status": "disabled",
                "severity": "high",
                "recommendation": f"Enable {check_info['name']} detection plugin",
            })
        elif plugin.get("action", "log") != "block":
            findings.append({
                "check": check_id, "name": check_info["name"],
                "owasp": check_info["owasp"], "status": "monitor_only",
                "severity": "medium",
                "recommendation": f"Switch {check_info['name']} from monitor to block mode",
            })
    block_mode = config.get("global_action", "log")
    if block_mode == "log":
        findings.append({
            "check": "global_mode", "status": "monitor_only",
            "severity": "high",
            "recommendation": "Switch RASP from monitor to block mode after baseline period",
        })
    return findings


def analyze_attack_logs(log_file, limit=10000):
    """Analyze RASP attack detection logs."""
    attacks = []
    by_type = defaultdict(int)
    by_source = defaultdict(int)
    blocked = 0
    try:
        with open(log_file) as f:
            for i, line in enumerate(f):
                if i >= limit:
                    break
                try:
                    event = json.loads(line)
                    attack_type = event.get("attack_type", "unknown")
                    by_type[attack_type] += 1
                    by_source[event.get("client_ip", "")] += 1
                    if event.get("action") == "block":
                        blocked += 1
                    attacks.append(event)
                except json.JSONDecodeError:
                    continue
    except FileNotFoundError:
        logger.warning("RASP log file not found: %s", log_file)
    return {
        "total_attacks": len(attacks),
        "blocked": blocked,
        "block_rate": round(blocked / max(len(attacks), 1) * 100, 1),
        "by_attack_type": dict(sorted(by_type.items(), key=lambda x: x[1], reverse=True)),
        "top_sources": dict(sorted(by_source.items(), key=lambda x: x[1], reverse=True)[:10]),
    }


def check_agent_health(config):
    """Verify RASP agent deployment health."""
    findings = []
    if not config.get("agent_version"):
        findings.append({"issue": "Agent version unknown", "severity": "medium"})
    if config.get("cpu_limit_percent", 100) > 10:
        findings.append({
            "issue": f"CPU limit set to {config.get('cpu_limit_percent')}% (recommend < 5%)",
            "severity": "low",
        })
    if not config.get("siem_integration", {}).get("enabled"):
        findings.append({
            "issue": "SIEM log forwarding not configured",
            "severity": "high",
            "recommendation": "Enable syslog or webhook forwarding to SIEM",
        })
    return findings


def calculate_coverage(config):
    """Calculate OWASP Top 10 coverage percentage."""
    enabled_checks = config.get("detection_plugins", {})
    covered = sum(1 for check_id in OWASP_TOP10_CHECKS if enabled_checks.get(check_id, {}).get("enabled"))
    return {
        "total_checks": len(OWASP_TOP10_CHECKS),
        "enabled": covered,
        "coverage_percent": round(covered / len(OWASP_TOP10_CHECKS) * 100, 1),
    }


def generate_report(policy_findings, attack_analysis, health_findings, coverage, config):
    all_findings = policy_findings + health_findings
    return {
        "timestamp": datetime.utcnow().isoformat(),
        "rasp_agent": config.get("agent_type", "openrasp"),
        "agent_version": config.get("agent_version", "unknown"),
        "global_mode": config.get("global_action", "log"),
        "owasp_coverage": coverage,
        "policy_findings": policy_findings,
        "attack_analysis": attack_analysis,
        "health_findings": health_findings,
        "total_findings": len(all_findings),
        "critical_findings": sum(1 for f in all_findings if f.get("severity") == "critical"),
    }


def main():
    parser = argparse.ArgumentParser(description="RASP Security Audit Agent")
    parser.add_argument("--config", required=True, help="RASP agent config JSON file")
    parser.add_argument("--attack-log", help="RASP attack log file (JSON lines)")
    parser.add_argument("--output", default="rasp_audit_report.json")
    args = parser.parse_args()

    config = parse_rasp_config(args.config)
    policy_findings = audit_rasp_policy(config)
    attack_analysis = analyze_attack_logs(args.attack_log) if args.attack_log else {}
    health_findings = check_agent_health(config)
    coverage = calculate_coverage(config)
    report = generate_report(policy_findings, attack_analysis, health_findings, coverage, config)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("RASP audit: %.1f%% OWASP coverage, %d findings, mode: %s",
                coverage["coverage_percent"], report["total_findings"], config.get("global_action", "log"))
    print(json.dumps(report, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-fuzzing-with-aflplusplus/SKILL.md =====
---
name: performing-fuzzing-with-aflplusplus
description: 'Performs coverage-guided fuzzing of compiled binaries with AFL++, instrumenting
  targets via afl-cc/afl-clang-fast, minimizing corpora with afl-cmin and afl-tmin,
  running parallel campaigns with afl-fuzz, and triaging crashes with CASR or GDB
  scripts. Use for binary fuzzing, crash and memory-corruption discovery, coverage-guided
  testing, or running AFL++ fuzzing campaigns.

  '
domain: cybersecurity
subdomain: application-security
tags:
- fuzzing
- aflplusplus
- coverage-guided
- crash-triage
- binary-analysis
- security-testing
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
- PR.PS-01
- PR.PS-04
- ID.RA-01
- PR.DS-10
mitre_attack:
- T1078
- T1190
- T1059
- T1005
---
# Performing Fuzzing with AFL++

## Overview

AFL++ is a community-maintained fork of American Fuzzy Lop (AFL) that provides coverage-guided
fuzzing for compiled binaries. It instruments targets at compile time or via QEMU/Unicorn mode
for binary-only fuzzing, then mutates input corpora to discover new code paths. AFL++ includes
advanced scheduling (MOpt, rare), custom mutators, CMPLOG for input-to-state comparison solving,
and persistent mode for high-throughput fuzzing.


## When to Use

- When conducting security assessments that involve performing fuzzing with aflplusplus
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- AFL++ installed (`apt install afl++` or build from source)
- Target binary source code (for compile-time instrumentation) or QEMU mode for binary-only
- Initial seed corpus of valid inputs for the target format
- Linux system with /proc/sys/kernel/core_pattern configured

## Steps

1. Instrument the target binary with `afl-cc` or `afl-clang-fast`
2. Prepare seed corpus directory with minimal valid inputs
3. Minimize corpus with `afl-cmin` to remove redundant seeds
4. Run `afl-fuzz` with appropriate flags (-i input -o output)
5. Monitor fuzzing progress via afl-whatsup and UI stats
6. Triage crashes with `afl-tmin` minimization and CASR/GDB analysis
7. Report unique crashes with reproduction steps

## Expected Output

```
+++ Findings +++
  unique crashes: 12
  unique hangs: 3
  last crash: 00:02:15 ago
+++ Coverage +++
  map density: 4.23% / 8.41%
  paths found: 1847
  exec speed: 2145/sec
```

===== END FILE =====


===== FILE: skills/performing-fuzzing-with-aflplusplus/LICENSE =====

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


===== FILE: skills/performing-fuzzing-with-aflplusplus/references/api-reference.md =====
# AFL++ Fuzzing — API Reference

## Installation

```bash
apt install afl++                    # Ubuntu/Debian
# Or build from source:
git clone https://github.com/AFLplusplus/AFLplusplus && cd AFLplusplus && make all
```

## AFL++ CLI Tools

| Tool | Description |
|------|-------------|
| `afl-cc` / `afl-clang-fast` | Compile-time instrumentation compiler wrapper |
| `afl-fuzz` | Main fuzzer — coverage-guided mutation engine |
| `afl-cmin` | Corpus minimization — remove redundant seeds |
| `afl-tmin` | Test case minimization — shrink individual inputs |
| `afl-whatsup` | Multi-instance campaign status summary |
| `afl-plot` | Generate fuzzing progress plots |
| `afl-showmap` | Display coverage map for a single input |

## afl-fuzz Key Flags

| Flag | Description |
|------|-------------|
| `-i <dir>` | Input seed corpus directory |
| `-o <dir>` | Output directory for findings |
| `-m <MB>` | Memory limit (use `none` for ASAN) |
| `-t <ms>` | Execution timeout per test case |
| `-x <dict>` | Optional fuzzing dictionary |
| `-p <sched>` | Power schedule: fast, coe, explore, rare, mmopt |
| `-l <level>` | CMPLOG instrumentation level (2=transforms, 3=all) |
| `-c <bin>` | CMPLOG binary for input-to-state |
| `-M <name>` | Main fuzzer instance (parallel mode) |
| `-S <name>` | Secondary fuzzer instance (parallel mode) |
| `-Q` | QEMU mode (binary-only fuzzing) |
| `-U` | Unicorn mode |

## fuzzer_stats File Fields

| Field | Description |
|-------|-------------|
| `execs_done` | Total executions completed |
| `execs_per_sec` | Current execution speed |
| `corpus_count` | Total paths in corpus |
| `saved_crashes` | Unique crashes discovered |
| `saved_hangs` | Unique hangs discovered |
| `stability` | Execution stability percentage |
| `bitmap_cvg` | Code coverage bitmap density |

## Crash Triage Tools

| Tool | Purpose |
|------|---------|
| `casr-afl` | CASR crash severity analysis for AFL++ |
| `afl-tmin` | Minimize crash inputs |
| `gdb --batch -ex run` | Reproduce crash under debugger |

## External References

- [AFL++ Documentation](https://aflplus.plus/docs/)
- [AFL++ GitHub](https://github.com/AFLplusplus/AFLplusplus)
- [CASR Crash Triage](https://github.com/ispras/casr)

===== END FILE =====


===== FILE: skills/performing-fuzzing-with-aflplusplus/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized testing only
"""AFL++ fuzzing campaign management and crash triage agent."""

import json
import argparse
import os
import subprocess
from datetime import datetime


def instrument_target(source_path, output_path, compiler="afl-clang-fast",
                      sanitizer=None):
    """Compile target with AFL++ instrumentation."""
    cmd = [compiler, "-o", output_path, source_path]
    if sanitizer == "asan":
        cmd.insert(1, "-fsanitize=address")
    elif sanitizer == "ubsan":
        cmd.insert(1, "-fsanitize=undefined")
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    return {
        "compiler": compiler,
        "source": source_path,
        "output": output_path,
        "sanitizer": sanitizer,
        "success": result.returncode == 0,
        "stderr": result.stderr[:500] if result.stderr else "",
    }


def minimize_corpus(afl_cmin_path, target_binary, input_dir, output_dir):
    """Minimize seed corpus using afl-cmin."""
    cmd = [afl_cmin_path or "afl-cmin", "-i", input_dir, "-o", output_dir,
           "--", target_binary]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
    before = len(os.listdir(input_dir)) if os.path.isdir(input_dir) else 0
    after = len(os.listdir(output_dir)) if os.path.isdir(output_dir) else 0
    return {
        "before": before,
        "after": after,
        "reduction_pct": round((1 - after / max(before, 1)) * 100, 1),
        "success": result.returncode == 0,
    }


def parse_fuzzer_stats(output_dir):
    """Parse afl-fuzz fuzzer_stats file for campaign metrics."""
    stats_path = os.path.join(output_dir, "default", "fuzzer_stats")
    if not os.path.exists(stats_path):
        stats_path = os.path.join(output_dir, "fuzzer_stats")
    if not os.path.exists(stats_path):
        return {"error": f"fuzzer_stats not found in {output_dir}"}

    stats = {}
    with open(stats_path, "r") as f:
        for line in f:
            if ":" in line:
                key, val = line.split(":", 1)
                stats[key.strip()] = val.strip()

    return {
        "start_time": stats.get("start_time", ""),
        "last_update": stats.get("last_update", ""),
        "execs_done": int(stats.get("execs_done", 0)),
        "execs_per_sec": float(stats.get("execs_per_sec", 0)),
        "paths_total": int(stats.get("corpus_count", stats.get("paths_total", 0))),
        "paths_found": int(stats.get("paths_found", 0)),
        "unique_crashes": int(stats.get("saved_crashes", stats.get("unique_crashes", 0))),
        "unique_hangs": int(stats.get("saved_hangs", stats.get("unique_hangs", 0))),
        "stability": stats.get("stability", ""),
        "bitmap_cvg": stats.get("bitmap_cvg", ""),
        "command_line": stats.get("command_line", ""),
    }


def triage_crashes(output_dir):
    """Enumerate and classify crash files from AFL++ output."""
    crash_dirs = [
        os.path.join(output_dir, "default", "crashes"),
        os.path.join(output_dir, "crashes"),
    ]
    crash_dir = None
    for d in crash_dirs:
        if os.path.isdir(d):
            crash_dir = d
            break
    if not crash_dir:
        return {"crashes": [], "total": 0}

    crashes = []
    for filename in sorted(os.listdir(crash_dir)):
        if filename.startswith("README") or filename == ".state":
            continue
        filepath = os.path.join(crash_dir, filename)
        size = os.path.getsize(filepath)
        sig_parts = filename.split(",")
        signal = ""
        for part in sig_parts:
            if part.startswith("sig:"):
                signal = part.split(":")[1]
        crashes.append({
            "filename": filename,
            "size_bytes": size,
            "signal": signal,
            "path": filepath,
        })
    return {"crashes": crashes, "total": len(crashes)}


def minimize_crash(afl_tmin_path, target_binary, crash_file, output_file):
    """Minimize a crash test case with afl-tmin."""
    cmd = [afl_tmin_path or "afl-tmin", "-i", crash_file, "-o", output_file,
           "--", target_binary]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    orig_size = os.path.getsize(crash_file) if os.path.exists(crash_file) else 0
    min_size = os.path.getsize(output_file) if os.path.exists(output_file) else 0
    return {
        "original_size": orig_size,
        "minimized_size": min_size,
        "reduction_pct": round((1 - min_size / max(orig_size, 1)) * 100, 1),
        "success": result.returncode == 0,
    }


def run_whatsup(output_dir):
    """Run afl-whatsup to get multi-instance campaign summary."""
    cmd = ["afl-whatsup", "-s", output_dir]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
    return {"output": result.stdout[:2000] if result.stdout else result.stderr[:500]}


def run_audit(args):
    """Execute AFL++ fuzzing campaign audit."""
    print(f"\n{'='*60}")
    print(f"  AFL++ FUZZING CAMPAIGN AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    report = {}

    if args.output_dir:
        stats = parse_fuzzer_stats(args.output_dir)
        report["fuzzer_stats"] = stats
        print(f"--- FUZZER STATS ---")
        print(f"  Executions: {stats.get('execs_done', 0):,}")
        print(f"  Exec/sec: {stats.get('execs_per_sec', 0)}")
        print(f"  Paths: {stats.get('paths_total', 0)}")
        print(f"  Crashes: {stats.get('unique_crashes', 0)}")
        print(f"  Hangs: {stats.get('unique_hangs', 0)}")
        print(f"  Stability: {stats.get('stability', '')}")
        print(f"  Coverage: {stats.get('bitmap_cvg', '')}")

        crash_data = triage_crashes(args.output_dir)
        report["crash_triage"] = crash_data
        print(f"\n--- CRASH TRIAGE ({crash_data['total']} crashes) ---")
        for c in crash_data["crashes"][:20]:
            print(f"  {c['filename']} ({c['size_bytes']}B) signal={c['signal']}")

    if args.instrument_src and args.instrument_out:
        inst = instrument_target(args.instrument_src, args.instrument_out,
                                  sanitizer=args.sanitizer)
        report["instrumentation"] = inst
        print(f"\n--- INSTRUMENTATION ---")
        print(f"  {'SUCCESS' if inst['success'] else 'FAILED'}: {inst['source']}")

    return report


def main():
    parser = argparse.ArgumentParser(description="AFL++ Fuzzing Campaign Agent")
    parser.add_argument("--output-dir", help="AFL++ output directory to analyze")
    parser.add_argument("--instrument-src", help="Source file to instrument")
    parser.add_argument("--instrument-out", help="Output path for instrumented binary")
    parser.add_argument("--sanitizer", choices=["asan", "ubsan"],
                        help="Address or undefined behavior sanitizer")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_audit(args)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-supply-chain-attack-simulation/SKILL.md =====
---
name: performing-supply-chain-attack-simulation
description: >-
  Simulates and detects software supply chain attacks: typosquatting
  detection via Levenshtein distance against popular PyPI package names,
  dependency confusion testing against private registries, SHA-256 package
  hash verification, and known-CVE scanning with pip-audit. Use when auditing
  a project's dependencies for malicious or confused packages, or when
  assessing package-registry supply-chain risk.
domain: cybersecurity
subdomain: application-security
tags:
- supply-chain
- typosquatting
- dependency-confusion
- package-verification
- pip-audit
- PyPI
- software-composition-analysis
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.PS-04
- ID.RA-01
- PR.DS-10
mitre_attack:
- T1078
- T1190
- T1059
- T1195
- T1554
---

# Performing Supply Chain Attack Simulation

## Overview

Software supply chain attacks exploit trust in package registries through typosquatting (registering names similar to popular packages), dependency confusion (publishing higher-version public packages matching private names), and compromised package distribution. This skill detects these attack vectors by computing Levenshtein distance between package names and popular PyPI packages, verifying package integrity via SHA-256 hash comparison, scanning for known CVEs with pip-audit, and testing dependency resolution order for confusion vulnerabilities.


## When to Use

- When conducting security assessments that involve performing supply chain attack simulation
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Python 3.9+ with `pip-audit`, `Levenshtein`, `requests`
- Access to PyPI JSON API (https://pypi.org/pypi/{package}/json)
- Network access for package metadata retrieval


> **Legal Notice:** This skill is for authorized security testing and educational purposes only. Unauthorized use against systems you do not own or have written permission to test is illegal and may violate computer fraud laws.

## Key Detection Areas

1. **Typosquatting** — compare package names against top PyPI packages using edit distance thresholds
2. **Dependency confusion** — check if internal package names exist on public PyPI with higher version numbers
3. **Hash verification** — download packages and verify SHA-256 digests match published hashes
4. **Vulnerability scanning** — audit installed packages against OSV and PyPA advisory databases
5. **Metadata anomalies** — flag packages with suspicious author emails, missing homepages, or very recent first upload dates

## Output

JSON report with risk scores per package, detected attack vectors, hash verification results, and CVE findings.

===== END FILE =====


===== FILE: skills/performing-supply-chain-attack-simulation/LICENSE =====

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


===== FILE: skills/performing-supply-chain-attack-simulation/references/api-reference.md =====
# Supply Chain Attack Simulation Reference

## Tool Installation

```bash
pip install pip-audit python-Levenshtein requests
```

## PyPI JSON API

```bash
# Get package metadata
curl https://pypi.org/pypi/{package_name}/json

# Get specific version
curl https://pypi.org/pypi/{package_name}/{version}/json
```

### Response Structure

```json
{
  "info": {
    "name": "requests",
    "version": "2.31.0",
    "author": "Kenneth Reitz",
    "author_email": "me@kennethreitz.org",
    "home_page": "https://requests.readthedocs.io",
    "summary": "Python HTTP for Humans."
  },
  "urls": [
    {
      "filename": "requests-2.31.0.tar.gz",
      "packagetype": "sdist",
      "digests": {
        "sha256": "942c5a758f98d790eaed1a29cb6eefc7f0edf3fcb0fce8aea3fbd5951d bfcfeb"
      }
    }
  ]
}
```

## pip-audit CLI

```bash
# Audit current environment
pip-audit

# JSON output
pip-audit --format json

# Audit requirements file
pip-audit -r requirements.txt

# Hash-checking mode (pinned deps only)
pip-audit --require-hashes -r requirements.txt

# Fix vulnerabilities automatically
pip-audit --fix
```

## pip Hash Verification

```bash
# Download with hash verification
pip download --no-deps --require-hashes -r requirements.txt

# requirements.txt with hashes
requests==2.31.0 \
    --hash=sha256:942c5a758f98d790eaed1a29cb6eefc7f0edf3fcb0fce8aea3fbd5951dbfcfeb
```

## pypi-scan Typosquatting Detection

```bash
# Install
pip install pypi-scan

# Scan for typosquatting of a package
pypi-scan --package requests

# Scan with custom edit distance
pypi-scan --package numpy --edit-distance 2
```

## Levenshtein Distance Examples

| Package | Target | Distance | Risk |
|---------|--------|----------|------|
| `reqeusts` | `requests` | 1 | High |
| `requets` | `requests` | 1 | High |
| `request` | `requests` | 1 | High |
| `numpys` | `numpy` | 1 | High |
| `pandsa` | `pandas` | 1 | High |
| `flaask` | `flask` | 1 | High |

## Dependency Confusion Test Structure

```json
[
  {"name": "my-internal-lib", "version": "1.2.0"},
  {"name": "company-utils", "version": "0.5.3"},
  {"name": "private-auth-sdk", "version": "2.1.0"}
]
```

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Supply Chain Compromise | T1195.001 | Compromise software dependencies |
| Trusted Developer Utilities | T1127 | Abuse package manager trust |
| Ingress Tool Transfer | T1105 | Download malicious packages |

## Metadata Anomaly Indicators

| Indicator | Risk | Description |
|-----------|------|-------------|
| Disposable email author | High | Author uses throwaway email service |
| No homepage/repo URL | Medium | Package has no verifiable source |
| No author info | Medium | Anonymous package publication |
| Very short description | Low | Minimal package documentation |
| Recent first upload + popular name variant | Critical | Likely typosquatting attempt |

===== END FILE =====


===== FILE: skills/performing-supply-chain-attack-simulation/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and educational environments only.
# Usage against targets without prior mutual consent is illegal.
# It is the end user's responsibility to obey all applicable local, state and federal laws.
"""Simulate and detect software supply chain attacks: typosquatting, dependency confusion, hash verification."""

import argparse
import json
import subprocess
from datetime import datetime, timezone


def get_levenshtein_distance(s1, s2):
    """Compute Levenshtein edit distance between two strings."""
    if len(s1) < len(s2):
        return get_levenshtein_distance(s2, s1)
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


TOP_PYPI_PACKAGES = [
    "requests", "numpy", "pandas", "flask", "django", "boto3", "scipy",
    "tensorflow", "torch", "scikit-learn", "pillow", "matplotlib",
    "cryptography", "pyyaml", "sqlalchemy", "celery", "redis", "psycopg2",
    "paramiko", "beautifulsoup4", "selenium", "pytest", "setuptools",
    "urllib3", "certifi", "idna", "charset-normalizer", "pip", "wheel",
    "packaging", "six", "python-dateutil", "jinja2", "markupsafe",
    "pydantic", "fastapi", "uvicorn", "httpx", "aiohttp", "grpcio"
]


def check_typosquatting(package_name, threshold=2):
    """Check if package name is suspiciously similar to popular packages."""
    matches = []
    for popular in TOP_PYPI_PACKAGES:
        if package_name == popular:
            continue
        distance = get_levenshtein_distance(package_name.lower(), popular.lower())
        if 0 < distance <= threshold:
            matches.append({
                "popular_package": popular,
                "edit_distance": distance,
                "risk": "High" if distance == 1 else "Medium"
            })
    return matches


def query_pypi_metadata(package_name):
    """Fetch package metadata from PyPI JSON API."""
    try:
        import requests
        resp = requests.get(
            f"https://pypi.org/pypi/{package_name}/json",
            timeout=10
        )
        if resp.status_code == 200:
            return resp.json()
        return None
    except Exception:
        return None


def check_dependency_confusion(private_packages):
    """Check if private package names exist on public PyPI."""
    findings = []
    for pkg_info in private_packages:
        name = pkg_info["name"]
        internal_version = pkg_info.get("version", "0.0.0")
        metadata = query_pypi_metadata(name)
        if metadata:
            public_version = metadata.get("info", {}).get("version", "0.0.0")
            findings.append({
                "package": name,
                "internal_version": internal_version,
                "public_version": public_version,
                "risk": "Critical",
                "message": f"Private package '{name}' exists on public PyPI as version {public_version}",
                "attack_vector": "dependency_confusion"
            })
        else:
            findings.append({
                "package": name,
                "internal_version": internal_version,
                "risk": "Info",
                "message": f"Private package '{name}' not found on public PyPI (safe)"
            })
    return findings


def verify_package_hash(package_name, expected_hash=None):
    """Download package and verify SHA-256 hash against PyPI published digests."""
    metadata = query_pypi_metadata(package_name)
    if not metadata:
        return {"package": package_name, "status": "error", "message": "Package not found on PyPI"}

    releases = metadata.get("urls", [])
    if not releases:
        return {"package": package_name, "status": "error", "message": "No release files found"}

    sdist = None
    for release in releases:
        if release.get("packagetype") == "sdist":
            sdist = release
            break
    if not sdist:
        sdist = releases[0]

    published_sha256 = sdist.get("digests", {}).get("sha256", "")
    result = {
        "package": package_name,
        "version": metadata["info"]["version"],
        "filename": sdist["filename"],
        "published_sha256": published_sha256,
        "packagetype": sdist["packagetype"]
    }

    if expected_hash:
        if expected_hash == published_sha256:
            result["status"] = "verified"
            result["message"] = "Hash matches expected value"
        else:
            result["status"] = "mismatch"
            result["risk"] = "Critical"
            result["message"] = "Hash does NOT match expected value — possible tampering"
            result["expected_hash"] = expected_hash
    else:
        result["status"] = "retrieved"
        result["message"] = "Published hash retrieved for manual verification"

    return result


def run_pip_audit():
    """Run pip-audit to scan installed packages for known vulnerabilities."""
    try:
        proc = subprocess.run(
            ["pip-audit", "--format", "json", "--progress-spinner", "off"],
            capture_output=True, text=True, timeout=120
        )
        if proc.returncode == 0 or proc.stdout:
            return json.loads(proc.stdout) if proc.stdout.strip() else []
        return [{"error": proc.stderr.strip()}]
    except FileNotFoundError:
        return [{"error": "pip-audit not installed. Run: pip install pip-audit"}]
    except subprocess.TimeoutExpired:
        return [{"error": "pip-audit timed out after 120 seconds"}]
    except json.JSONDecodeError:
        return [{"error": "Failed to parse pip-audit output"}]


def analyze_metadata_anomalies(package_name):
    """Detect suspicious metadata patterns in a PyPI package."""
    metadata = query_pypi_metadata(package_name)
    if not metadata:
        return {"package": package_name, "status": "not_found"}

    info = metadata["info"]
    anomalies = []

    if not info.get("home_page") and not info.get("project_url"):
        anomalies.append({
            "check": "missing_homepage",
            "severity": "Medium",
            "message": "Package has no homepage or project URL"
        })

    if not info.get("author") and not info.get("author_email"):
        anomalies.append({
            "check": "missing_author",
            "severity": "Medium",
            "message": "Package has no author information"
        })

    if info.get("author_email") and any(
        domain in info["author_email"]
        for domain in ["mailinator.com", "guerrillamail.com", "tempmail.com", "throwaway.email"]
    ):
        anomalies.append({
            "check": "disposable_email",
            "severity": "High",
            "message": f"Author uses disposable email: {info['author_email']}"
        })

    summary = info.get("summary", "")
    if not summary or len(summary) < 10:
        anomalies.append({
            "check": "missing_description",
            "severity": "Low",
            "message": "Package has no meaningful description"
        })

    return {
        "package": package_name,
        "version": info.get("version"),
        "author": info.get("author"),
        "author_email": info.get("author_email"),
        "anomalies": anomalies,
        "anomaly_count": len(anomalies)
    }


def main():
    parser = argparse.ArgumentParser(
        description="Simulate and detect software supply chain attacks"
    )
    subparsers = parser.add_subparsers(dest="command", help="Attack simulation type")

    typo_parser = subparsers.add_parser("typosquat", help="Check for typosquatting")
    typo_parser.add_argument("packages", nargs="+", help="Package names to check")
    typo_parser.add_argument("--threshold", type=int, default=2, help="Max edit distance (default: 2)")

    confusion_parser = subparsers.add_parser("confusion", help="Test dependency confusion")
    confusion_parser.add_argument("--packages", required=True, help="JSON file with private packages [{name, version}]")

    hash_parser = subparsers.add_parser("verify-hash", help="Verify package hash")
    hash_parser.add_argument("package", help="Package name")
    hash_parser.add_argument("--expected-hash", help="Expected SHA-256 hash to compare")

    subparsers.add_parser("audit", help="Run pip-audit vulnerability scan")

    meta_parser = subparsers.add_parser("metadata", help="Check metadata anomalies")
    meta_parser.add_argument("packages", nargs="+", help="Package names to analyze")

    args = parser.parse_args()

    if args.command == "typosquat":
        results = []
        for pkg in args.packages:
            matches = check_typosquatting(pkg, args.threshold)
            results.append({"package": pkg, "typosquat_matches": matches, "is_suspicious": len(matches) > 0})
        print(json.dumps({"scan_type": "typosquatting", "results": results, "timestamp": datetime.now(timezone.utc).isoformat()}, indent=2))

    elif args.command == "confusion":
        with open(args.packages) as f:
            private_pkgs = json.load(f)
        results = check_dependency_confusion(private_pkgs)
        print(json.dumps({"scan_type": "dependency_confusion", "results": results, "timestamp": datetime.now(timezone.utc).isoformat()}, indent=2))

    elif args.command == "verify-hash":
        result = verify_package_hash(args.package, args.expected_hash)
        print(json.dumps({"scan_type": "hash_verification", "result": result, "timestamp": datetime.now(timezone.utc).isoformat()}, indent=2))

    elif args.command == "audit":
        results = run_pip_audit()
        print(json.dumps({"scan_type": "vulnerability_audit", "results": results, "timestamp": datetime.now(timezone.utc).isoformat()}, indent=2))

    elif args.command == "metadata":
        results = [analyze_metadata_anomalies(pkg) for pkg in args.packages]
        print(json.dumps({"scan_type": "metadata_analysis", "results": results, "timestamp": datetime.now(timezone.utc).isoformat()}, indent=2))

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====
