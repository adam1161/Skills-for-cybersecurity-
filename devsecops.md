

===== FILE: skills/building-devsecops-pipeline-with-gitlab-ci/SKILL.md =====
---
name: building-devsecops-pipeline-with-gitlab-ci
description: Configure a GitLab CI/CD pipeline that embeds SAST (Semgrep, SpotBugs, Gosec, Bandit, NodeJsScan), DAST, container scanning, dependency scanning, and secret detection via GitLab's managed security templates. Use when building a shift-left DevSecOps pipeline in GitLab, adding automated vulnerability scanning stages to .gitlab-ci.yml, or triaging scanner findings with GitLab Duo AI before deployment.
domain: cybersecurity
subdomain: devsecops
tags:
- gitlab-ci
- devsecops
- sast
- dast
- container-scanning
- dependency-scanning
- secret-detection
- cicd-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195.001
- T1195.002
- T1552.001
- T1190
- T1610
---

# Building DevSecOps Pipeline with GitLab CI

## Overview

GitLab provides an integrated DevSecOps platform that embeds security testing directly into the CI/CD pipeline. By leveraging GitLab's built-in security scanners---SAST, DAST, container scanning, dependency scanning, secret detection, and license compliance---teams can shift security left, catching vulnerabilities during development rather than post-deployment. GitLab Duo AI assists with false positive detection for SAST vulnerabilities, helping security teams focus on genuine issues.


## When to Use

- When deploying or configuring building devsecops pipeline with gitlab ci capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- GitLab Ultimate license (required for full security scanner suite)
- GitLab Runner configured (shared or self-hosted)
- `.gitlab-ci.yml` pipeline configuration familiarity
- Docker-in-Docker (DinD) or Kaniko for container builds
- Application deployed to a staging environment for DAST scanning

## Core Security Scanning Stages

### Static Application Security Testing (SAST)

SAST analyzes source code for vulnerabilities before compilation. GitLab supports 14+ languages using analyzers such as Semgrep, SpotBugs, Gosec, Bandit, and NodeJsScan. The simplest inclusion uses GitLab's managed templates.

### Dynamic Application Security Testing (DAST)

DAST tests running applications by simulating attack payloads against HTTP endpoints. It detects XSS, SQLi, CSRF, and other runtime vulnerabilities that static analysis cannot find. DAST requires a deployed, accessible target URL.

### Container Scanning

Uses Trivy to scan Docker images for known CVEs in OS packages and application dependencies. Runs after the Docker build stage to gate images before they reach a registry.

### Dependency Scanning

Inspects dependency manifests (package.json, requirements.txt, pom.xml, Gemfile.lock) for known vulnerable versions. Operates at the source code level, complementing container scanning.

### Secret Detection

Scans commits for accidentally committed credentials, API keys, tokens, and private keys using pattern matching and entropy analysis. Runs on every commit to prevent secrets from reaching the repository.

## Implementation

### Complete Pipeline Configuration

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test
  - security
  - deploy-staging
  - dast
  - deploy-production

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  SECURE_LOG_LEVEL: "info"

# Include GitLab managed security templates
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml
  - template: DAST.gitlab-ci.yml
  - template: Security/License-Scanning.gitlab-ci.yml

build:
  stage: build
  image: docker:24.0
  services:
    - docker:24.0-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
  rules:
    - if: $CI_COMMIT_BRANCH

unit-tests:
  stage: test
  image: $DOCKER_IMAGE
  script:
    - npm ci
    - npm run test:coverage
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      junit: junit-report.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Override SAST to run in security stage
sast:
  stage: security
  variables:
    SAST_EXCLUDED_PATHS: "spec,test,tests,tmp,node_modules"
    SEARCH_MAX_DEPTH: 10

# Override container scanning
container_scanning:
  stage: security
  variables:
    CS_IMAGE: $DOCKER_IMAGE
    CS_SEVERITY_THRESHOLD: "HIGH"

# Override dependency scanning
dependency_scanning:
  stage: security

# Override secret detection
secret_detection:
  stage: security

# License compliance scanning
license_scanning:
  stage: security

deploy-staging:
  stage: deploy-staging
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/app app=$DOCKER_IMAGE -n staging
    - kubectl rollout status deployment/app -n staging --timeout=300s
  environment:
    name: staging
    url: https://staging.example.com
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

# DAST runs against deployed staging
dast:
  stage: dast
  variables:
    DAST_WEBSITE: https://staging.example.com
    DAST_FULL_SCAN_ENABLED: "true"
    DAST_BROWSER_SCAN: "true"
  needs:
    - deploy-staging
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

deploy-production:
  stage: deploy-production
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/app app=$DOCKER_IMAGE -n production
    - kubectl rollout status deployment/app -n production --timeout=300s
  environment:
    name: production
    url: https://app.example.com
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### Security Approval Policies

Configure scan execution policies to enforce mandatory security scans:

1. Navigate to Security & Compliance > Policies
2. Create a "Scan Execution Policy" requiring SAST and secret detection on all branches
3. Create a "Merge Request Approval Policy" requiring security team approval when critical vulnerabilities are detected

### Custom SAST Ruleset Configuration

Create `.gitlab/sast-ruleset.toml` to customize analyzer behavior:

```toml
[semgrep]
  [[semgrep.ruleset]]
    dirs = ["src"]

  [[semgrep.passthrough]]
    type = "url"
    target = "/sgrep-rules/custom-rules.yml"
    value = "https://semgrep.dev/p/owasp-top-ten"

  [[semgrep.passthrough]]
    type = "url"
    target = "/sgrep-rules/java-rules.yml"
    value = "https://semgrep.dev/p/java"
```

## Security Dashboard and Vulnerability Management

### Vulnerability Report

GitLab consolidates all scanner findings into a single Vulnerability Report accessible at Security & Compliance > Vulnerability Report. Each vulnerability includes:
- Severity rating (Critical, High, Medium, Low, Info)
- Scanner source (SAST, DAST, Container, Dependency, Secret)
- Location in source code or image layer
- Remediation guidance and suggested fixes
- Status tracking (Detected, Confirmed, Dismissed, Resolved)

### Merge Request Security Widget

Every merge request displays a security scanning widget showing:
- New vulnerabilities introduced by the MR
- Fixed vulnerabilities resolved by the MR
- Comparison against the target branch baseline

## Pipeline Optimization

- **Parallel execution**: Security scanners run concurrently in the security stage
- **Caching**: Use CI cache for dependency downloads to speed up scanning
- **Incremental scanning**: SAST can scan only changed files using `SAST_INCREMENTAL: "true"`
- **Fail conditions**: Set `allow_failure: false` on critical scanners to enforce quality gates

## Monitoring and Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| Pipeline security coverage | Percentage of projects with all scanners enabled | > 95% |
| Critical vulnerability MTTR | Time from detection to resolution for critical findings | < 48 hours |
| False positive rate | Percentage of dismissed-as-false-positive findings | < 15% |
| Secret detection block rate | Percentage of secret commits blocked by push rules | > 99% |

## References

- [GitLab Security Scanning Documentation](https://docs.gitlab.com/ee/user/application_security/)
- [GitLab SAST Analyzers](https://docs.gitlab.com/ee/user/application_security/sast/)
- [GitLab DAST Configuration](https://docs.gitlab.com/ee/user/application_security/dast/)
- [GitLab Security Policies](https://docs.gitlab.com/ee/user/application_security/policies/)
- [GitLab Vulnerability Management](https://docs.gitlab.com/ee/user/application_security/vulnerability_report/)

===== END FILE =====


===== FILE: skills/building-devsecops-pipeline-with-gitlab-ci/LICENSE =====

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


===== FILE: skills/building-devsecops-pipeline-with-gitlab-ci/assets/template.md =====
# GitLab DevSecOps Pipeline Implementation Template

## Pipeline Security Scanner Checklist

| Scanner | Enabled | Template Included | Threshold Set | Blocking |
|---------|---------|-------------------|---------------|----------|
| SAST | [ ] | [ ] | Severity: _____ | [ ] |
| DAST | [ ] | [ ] | Severity: _____ | [ ] |
| Container Scanning | [ ] | [ ] | Severity: _____ | [ ] |
| Dependency Scanning | [ ] | [ ] | Severity: _____ | [ ] |
| Secret Detection | [ ] | [ ] | N/A | [ ] |
| License Scanning | [ ] | [ ] | Policy: _____ | [ ] |

## Security Policy Configuration

| Policy Type | Name | Scope | Enforcement |
|-------------|------|-------|-------------|
| Scan Execution | | [ ] All branches [ ] Default only | [ ] Required |
| MR Approval | | Severity trigger: _____ | Approvers: _____ |

## Environment-Specific DAST Targets

| Environment | URL | Auth Method | Scan Type | Schedule |
|-------------|-----|-------------|-----------|----------|
| Staging | | [ ] None [ ] Token [ ] Cookie | [ ] Passive [ ] Full | |
| Pre-production | | [ ] None [ ] Token [ ] Cookie | [ ] Passive [ ] Full | |

## Vulnerability SLA Targets

| Severity | Detection to Triage | Triage to Fix | Total SLA |
|----------|--------------------|--------------|-----------|
| Critical | 4 hours | 24 hours | 48 hours |
| High | 24 hours | 5 days | 7 days |
| Medium | 48 hours | 14 days | 30 days |
| Low | 1 week | 30 days | 90 days |

===== END FILE =====


===== FILE: skills/building-devsecops-pipeline-with-gitlab-ci/references/api-reference.md =====
# API Reference: GitLab CI DevSecOps Pipeline

## GitLab Security Templates
| Template | Stage |
|----------|-------|
| `Security/SAST.gitlab-ci.yml` | Static analysis |
| `Security/DAST.gitlab-ci.yml` | Dynamic testing |
| `Security/Dependency-Scanning.gitlab-ci.yml` | Dependency audit |
| `Security/Container-Scanning.gitlab-ci.yml` | Container scan |
| `Security/Secret-Detection.gitlab-ci.yml` | Secret detection |
| `Security/IaC-Scanning.gitlab-ci.yml` | IaC security |

## .gitlab-ci.yml Structure
```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
stages:
  - build
  - test
  - security
  - deploy
variables:
  SECURE_LOG_LEVEL: info
```

## GitLab CI Lint API
```
POST /api/v4/projects/:id/ci/lint
PRIVATE-TOKEN: your-token
Body: {"content": "yaml-string"}
```

## Security Variables
| Variable | Description |
|----------|-------------|
| `SAST_DEFAULT_ANALYZERS` | Comma-separated analyzer list |
| `SAST_EXCLUDED_ANALYZERS` | Analyzers to skip |
| `CS_IMAGE` | Container image to scan |
| `DAST_WEBSITE` | Target URL for DAST |
| `SECRET_DETECTION_HISTORIC_SCAN` | Scan full history |

## Vulnerability Report API
```
GET /api/v4/projects/:id/vulnerability_findings
```

## Security Scanning Tools
| Tool | Type | Language |
|------|------|----------|
| Semgrep | SAST | Multi-language |
| Bandit | SAST | Python |
| Trivy | Container | Container images |
| Gitleaks | Secret | Git history |
| KICS | IaC | Terraform/CloudFormation |
| ZAP | DAST | Web applications |

===== END FILE =====


===== FILE: skills/building-devsecops-pipeline-with-gitlab-ci/references/standards.md =====
# Standards and Compliance Reference

## OWASP DevSecOps Pipeline Maturity Model

| Level | SAST | DAST | SCA | Container | Secrets | License |
|-------|------|------|-----|-----------|---------|---------|
| Level 1 (Basic) | Manual runs | None | Manual dependency check | None | Pre-commit hooks | None |
| Level 2 (Integrated) | CI-triggered on MR | Scheduled scans | CI-triggered | Image scan on build | CI scan on commits | CI-triggered |
| Level 3 (Enforced) | Required for merge | Gate before deploy | Block on critical CVE | Block vulnerable images | Push protection | Policy enforcement |
| Level 4 (Optimized) | Custom rules, tuned FP | Authenticated full scan | Auto-remediation PRs | Signed images only | Auto-rotation | SBOM generation |

## NIST SP 800-218 (SSDF) Mapping

| SSDF Practice | GitLab Feature | Pipeline Stage |
|---------------|----------------|----------------|
| PO.1 Define security requirements | Security policies | Policy configuration |
| PW.1 Design software securely | Threat modeling integration | Pre-build |
| PW.4 Reuse well-secured software | Dependency scanning | Security stage |
| PW.5 Create source code securely | SAST, secret detection | Security stage |
| PW.7 Review and test code | MR security widget | Merge request |
| PW.8 Test executable code | DAST | Post-deploy staging |
| PW.9 Configure software securely | Container scanning | Security stage |
| RV.1 Identify vulnerabilities | Vulnerability report | Dashboard |
| RV.2 Assess and prioritize | Severity classification | Triage workflow |
| RV.3 Remediate vulnerabilities | Issue tracking integration | Sprint planning |

## CIS Software Supply Chain Security

- **SCS-1**: Secure source code management with protected branches and signed commits
- **SCS-2**: Secure build pipelines with pinned template versions and runner isolation
- **SCS-3**: Verified dependencies through dependency scanning and license compliance
- **SCS-4**: Secure artifacts with container scanning and signed images
- **SCS-5**: Deployment security with manual gates and environment approvals

## GitLab Scanner Coverage Matrix

| Vulnerability Type | Primary Scanner | Secondary Scanner |
|--------------------|-----------------|-------------------|
| SQL Injection | SAST (Semgrep) | DAST |
| XSS | SAST | DAST |
| SSRF | SAST | DAST |
| Command Injection | SAST | DAST |
| Insecure Deserialization | SAST | N/A |
| Known CVE in dependency | Dependency Scanning | Container Scanning |
| Hardcoded credentials | Secret Detection | SAST |
| License violation | License Scanning | N/A |
| OS-level CVE in image | Container Scanning | N/A |
| Authentication flaws | DAST | SAST |

===== END FILE =====


===== FILE: skills/building-devsecops-pipeline-with-gitlab-ci/references/workflows.md =====
# GitLab DevSecOps Pipeline Workflows

## Workflow 1: Merge Request Security Review

```
Developer creates merge request
           |
   Pipeline triggers security scanners in parallel:
   [SAST] [Secret Detection] [Dependency Scanning] [License Scanning]
           |
   MR Security Widget displays results:
   - New vulnerabilities introduced
   - Existing vulnerabilities fixed
   - Comparison with target branch
           |
   [No Critical/High] --> Reviewers can approve and merge
   [Critical/High found] --> MR blocked by approval policy
           |
   Security team reviews findings
           |
   [Confirmed] --> Developer remediates and re-pushes
   [False Positive] --> Dismissed with documented reason
           |
   All findings resolved --> MR eligible for merge
```

## Workflow 2: Container Image Security Gate

```
Docker image built in CI
           |
   Container scanning (Trivy) analyzes image layers
           |
   Findings categorized by severity
           |
   [Below threshold] --> Image pushed to registry with metadata
   [Above threshold] --> Pipeline fails, image not pushed
           |
   Registry stores scan results as artifact
           |
   Deployment pulls only scanned/approved images
```

## Workflow 3: DAST Against Staging Environment

```
Application deployed to staging
           |
   DAST browser scan initiated against staging URL
           |
   Authenticated scan crawls application pages
           |
   Active testing for XSS, SQLi, CSRF, etc.
           |
   Results added to vulnerability report
           |
   [Pass] --> Manual deploy-to-production gate enabled
   [Fail on critical] --> Staging deployment rolled back
           |
   Production deploy requires manual approval
```

## Workflow 4: Vulnerability Lifecycle Management

```
Scanner detects vulnerability
           |
   Status: "Detected" in vulnerability report
           |
   Security analyst triages finding
           |
   [Confirmed vulnerability]        [False positive]
           |                              |
   Status: "Confirmed"              Status: "Dismissed"
   Issue created automatically      Reason documented
           |
   Developer assigned fix
           |
   Fix merged, scanner re-runs
           |
   Vulnerability no longer detected
           |
   Status: "Resolved"
```

===== END FILE =====


===== FILE: skills/building-devsecops-pipeline-with-gitlab-ci/scripts/agent.py =====
#!/usr/bin/env python3
"""DevSecOps Pipeline Builder Agent - Generates GitLab CI security scanning pipeline configurations."""

import json
import logging
import argparse
from datetime import datetime

import requests

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

SECURITY_STAGES = {
    "sast": {
        "template": "Security/SAST.gitlab-ci.yml",
        "description": "Static Application Security Testing",
        "tools": ["semgrep", "bandit", "eslint-security"],
    },
    "dast": {
        "template": "Security/DAST.gitlab-ci.yml",
        "description": "Dynamic Application Security Testing",
        "tools": ["zap"],
    },
    "dependency_scanning": {
        "template": "Security/Dependency-Scanning.gitlab-ci.yml",
        "description": "Dependency vulnerability scanning",
        "tools": ["gemnasium", "retire.js"],
    },
    "container_scanning": {
        "template": "Security/Container-Scanning.gitlab-ci.yml",
        "description": "Container image vulnerability scanning",
        "tools": ["trivy", "grype"],
    },
    "secret_detection": {
        "template": "Security/Secret-Detection.gitlab-ci.yml",
        "description": "Secret and credential detection",
        "tools": ["gitleaks"],
    },
    "license_scanning": {
        "template": "Jobs/License-Scanning.gitlab-ci.yml",
        "description": "License compliance scanning",
        "tools": ["license-finder"],
    },
    "iac_scanning": {
        "template": "Security/IaC-Scanning.gitlab-ci.yml",
        "description": "Infrastructure as Code scanning",
        "tools": ["kics"],
    },
}


def generate_gitlab_ci(stages, project_type="python", registry="$CI_REGISTRY"):
    """Generate .gitlab-ci.yml with security scanning stages."""
    ci_config = {
        "stages": ["build", "test", "security", "deploy"],
        "include": [],
        "variables": {
            "SECURE_LOG_LEVEL": "info",
            "SAST_EXCLUDED_ANALYZERS": "",
        },
    }
    for stage_name in stages:
        stage = SECURITY_STAGES.get(stage_name)
        if stage:
            ci_config["include"].append({"template": stage["template"]})

    if "container_scanning" in stages:
        ci_config["variables"]["CS_IMAGE"] = f"{registry}/$CI_PROJECT_PATH:$CI_COMMIT_SHA"

    if project_type == "python":
        ci_config["variables"]["SAST_DEFAULT_ANALYZERS"] = "semgrep,bandit"
    elif project_type == "javascript":
        ci_config["variables"]["SAST_DEFAULT_ANALYZERS"] = "semgrep,eslint"

    return ci_config


def validate_pipeline(gitlab_url, token, project_id, ci_content):
    """Validate CI configuration via GitLab API."""
    headers = {"PRIVATE-TOKEN": token, "Content-Type": "application/json"}
    data = {"content": json.dumps(ci_content)}
    try:
        resp = requests.post(f"{gitlab_url}/api/v4/projects/{project_id}/ci/lint", headers=headers, json=data, timeout=15)
        return resp.json()
    except requests.RequestException as e:
        return {"error": str(e)}


def assess_pipeline_coverage(stages):
    """Assess security coverage of the pipeline."""
    all_stages = set(SECURITY_STAGES.keys())
    covered = set(stages) & all_stages
    missing = all_stages - covered
    coverage = len(covered) / len(all_stages) * 100
    return {
        "coverage_percent": round(coverage, 1),
        "covered_stages": list(covered),
        "missing_stages": list(missing),
        "recommendation": "Add " + ", ".join(missing) if missing else "Full coverage",
    }


def generate_report(ci_config, coverage, stages):
    """Generate DevSecOps pipeline report."""
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "stages_configured": stages,
        "security_coverage": coverage,
        "gitlab_ci_config": ci_config,
    }
    print(f"DEVSECOPS REPORT: {len(stages)} stages, {coverage['coverage_percent']}% coverage")
    return report


def main():
    parser = argparse.ArgumentParser(description="DevSecOps Pipeline Builder Agent")
    parser.add_argument("--stages", nargs="*", choices=list(SECURITY_STAGES.keys()), default=list(SECURITY_STAGES.keys()))
    parser.add_argument("--project-type", choices=["python", "javascript", "java", "go"], default="python")
    parser.add_argument("--gitlab-url", help="GitLab URL for validation")
    parser.add_argument("--token", help="GitLab private token")
    parser.add_argument("--project-id", help="GitLab project ID")
    parser.add_argument("--output", default="devsecops_report.json")
    args = parser.parse_args()

    ci_config = generate_gitlab_ci(args.stages, args.project_type)
    coverage = assess_pipeline_coverage(args.stages)
    report = generate_report(ci_config, coverage, args.stages)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/building-devsecops-pipeline-with-gitlab-ci/scripts/process.py =====
#!/usr/bin/env python3
"""
GitLab DevSecOps Pipeline Security Report Generator

Queries GitLab API to aggregate security scanning results
across projects and generate compliance reports.
"""

import json
import os
import sys
import urllib.request
import urllib.error
from datetime import datetime
from collections import defaultdict


def gitlab_api_get(url: str, token: str) -> list | dict:
    headers = {"PRIVATE-TOKEN": token, "Content-Type": "application/json"}
    results = []
    page = 1
    while True:
        sep = "&" if "?" in url else "?"
        paginated = f"{url}{sep}per_page=100&page={page}"
        req = urllib.request.Request(paginated, headers=headers)
        try:
            with urllib.request.urlopen(req) as resp:
                data = json.loads(resp.read().decode())
                if isinstance(data, list):
                    if not data:
                        break
                    results.extend(data)
                    page += 1
                else:
                    return data
        except urllib.error.HTTPError as e:
            print(f"HTTP {e.code}: {e.read().decode()}")
            break
    return results


def get_group_projects(base_url: str, token: str, group_id: str) -> list:
    url = f"{base_url}/api/v4/groups/{group_id}/projects?include_subgroups=true"
    return gitlab_api_get(url, token)


def get_project_vulnerabilities(base_url: str, token: str, project_id: int) -> list:
    url = f"{base_url}/api/v4/projects/{project_id}/vulnerabilities?state=detected"
    return gitlab_api_get(url, token)


def get_pipeline_jobs(base_url: str, token: str, project_id: int, pipeline_id: int) -> list:
    url = f"{base_url}/api/v4/projects/{project_id}/pipelines/{pipeline_id}/jobs"
    return gitlab_api_get(url, token)


def check_security_scanners(jobs: list) -> dict:
    scanner_names = {
        "sast": False,
        "secret_detection": False,
        "dependency_scanning": False,
        "container_scanning": False,
        "dast": False,
        "license_scanning": False,
    }
    for job in jobs:
        name = job.get("name", "").lower()
        for scanner in scanner_names:
            if scanner.replace("_", "-") in name or scanner in name:
                scanner_names[scanner] = True
    return scanner_names


def generate_report(base_url: str, token: str, group_id: str) -> dict:
    projects = get_group_projects(base_url, token, group_id)
    report = {
        "gitlab_instance": base_url,
        "group_id": group_id,
        "generated_at": datetime.utcnow().isoformat() + "Z",
        "total_projects": len(projects),
        "scanner_coverage": defaultdict(int),
        "severity_totals": defaultdict(int),
        "project_details": [],
    }

    for project in projects:
        pid = project["id"]
        name = project["path_with_namespace"]

        vulns = get_project_vulnerabilities(base_url, token, pid)
        severity_counts = defaultdict(int)
        scanner_counts = defaultdict(int)
        for v in vulns:
            severity_counts[v.get("severity", "unknown")] += 1
            scanner_counts[v.get("scanner", {}).get("name", "unknown")] += 1
            report["severity_totals"][v.get("severity", "unknown")] += 1

        pipelines = gitlab_api_get(
            f"{base_url}/api/v4/projects/{pid}/pipelines?per_page=1&status=success", token
        )
        scanners_enabled = {}
        if pipelines and isinstance(pipelines, list):
            jobs = get_pipeline_jobs(base_url, token, pid, pipelines[0]["id"])
            scanners_enabled = check_security_scanners(jobs)
            for scanner, enabled in scanners_enabled.items():
                if enabled:
                    report["scanner_coverage"][scanner] += 1

        report["project_details"].append({
            "project": name,
            "open_vulnerabilities": len(vulns),
            "by_severity": dict(severity_counts),
            "by_scanner": dict(scanner_counts),
            "scanners_enabled": scanners_enabled,
        })

    report["scanner_coverage"] = dict(report["scanner_coverage"])
    report["severity_totals"] = dict(report["severity_totals"])
    return report


def print_report(report: dict) -> None:
    print(f"\n{'='*65}")
    print(f"GitLab DevSecOps Security Report")
    print(f"Instance: {report['gitlab_instance']}")
    print(f"Generated: {report['generated_at']}")
    print(f"{'='*65}")
    print(f"\nTotal Projects: {report['total_projects']}")

    print(f"\nScanner Coverage:")
    for scanner, count in sorted(report["scanner_coverage"].items()):
        pct = count / report["total_projects"] * 100 if report["total_projects"] > 0 else 0
        print(f"  {scanner:25s}: {count:3d}/{report['total_projects']} ({pct:.0f}%)")

    print(f"\nVulnerability Summary:")
    for sev in ["critical", "high", "medium", "low", "info", "unknown"]:
        count = report["severity_totals"].get(sev, 0)
        if count > 0:
            print(f"  {sev.upper():12s}: {count}")

    total = sum(report["severity_totals"].values())
    print(f"  {'TOTAL':12s}: {total}")

    print(f"\nTop 10 Projects by Open Vulnerabilities:")
    sorted_projects = sorted(
        report["project_details"], key=lambda p: p["open_vulnerabilities"], reverse=True
    )
    for p in sorted_projects[:10]:
        print(f"  {p['project']:45s} | Vulns: {p['open_vulnerabilities']}")


def main():
    token = os.environ.get("GITLAB_TOKEN")
    base_url = os.environ.get("GITLAB_URL", "https://gitlab.com")
    group_id = os.environ.get("GITLAB_GROUP_ID")

    if not token:
        print("Error: GITLAB_TOKEN environment variable required")
        sys.exit(1)
    if not group_id:
        print("Error: GITLAB_GROUP_ID environment variable required")
        sys.exit(1)

    report = generate_report(base_url, token, group_id)
    print_report(report)

    output = f"gitlab_devsecops_report_{datetime.utcnow().strftime('%Y%m%d')}.json"
    with open(output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\nReport saved to: {output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-aqua-security-for-container-scanning/SKILL.md =====
---
name: implementing-aqua-security-for-container-scanning
description: Deploy Aqua Security's Trivy scanner to detect vulnerabilities, misconfigurations,
  secrets, and license issues in container images across CI/CD pipelines and registries.
domain: cybersecurity
subdomain: devsecops
tags:
- aqua-security
- trivy
- container-scanning
- vulnerability-scanning
- sbom
- image-security
- supply-chain
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
- T1610
- T1611
---

# Implementing Aqua Security for Container Scanning

## Overview

Aqua Security provides Trivy, the world's most popular open-source universal security scanner, designed to find vulnerabilities, misconfigurations, secrets, SBOM data, and license issues in containers, Kubernetes, code repositories, and cloud environments. Trivy covers OS packages (Alpine, Debian, Ubuntu, RHEL, etc.) and language-specific dependencies (npm, pip, Maven, Go modules, Cargo, etc.) with vulnerability databases sourced from NVD, vendor advisories, and GitHub Security Advisories. The enterprise Aqua Platform extends Trivy with centralized policy management, runtime protection, and compliance reporting.


## When to Use

- When deploying or configuring implementing aqua security for container scanning capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Docker installed for local image scanning
- CI/CD platform (GitHub Actions, GitLab CI, Jenkins, etc.)
- Container registry access (Docker Hub, ECR, GCR, ACR, Harbor)
- Trivy CLI (`trivy`) or Trivy Operator for Kubernetes
- Aqua Platform license for enterprise features (optional)

## Core Scanning Capabilities

### Image Vulnerability Scanning

Trivy scans container images layer by layer, identifying CVEs in OS packages and application dependencies. It supports scanning local images, remote registry images, and tar archives.

```bash
# Scan a remote image
trivy image python:3.11-slim

# Scan with severity filter
trivy image --severity HIGH,CRITICAL nginx:latest

# Scan and fail CI if critical CVEs found
trivy image --exit-code 1 --severity CRITICAL myapp:latest

# Generate SBOM in CycloneDX format
trivy image --format cyclonedx --output sbom.json myapp:latest
```

### Filesystem and Repository Scanning

```bash
# Scan project directory for vulnerabilities in dependencies
trivy fs --scanners vuln,secret,misconfig .

# Scan a specific lockfile
trivy fs --scanners vuln package-lock.json

# Scan git repository
trivy repo https://github.com/org/project
```

### Kubernetes Scanning with Trivy Operator

The Trivy Operator runs inside a Kubernetes cluster and continuously scans workloads:

```bash
# Install Trivy Operator via Helm
helm repo add aqua https://aquasecurity.github.io/helm-charts/
helm repo update
helm install trivy-operator aqua/trivy-operator \
  --namespace trivy-system \
  --create-namespace \
  --set trivy.severity="HIGH,CRITICAL" \
  --set operator.scanJobTimeout="5m"
```

The operator creates VulnerabilityReport and ConfigAuditReport custom resources for each workload.

### IaC Misconfiguration Scanning

```bash
# Scan Terraform files
trivy config --severity HIGH,CRITICAL ./terraform/

# Scan Dockerfile for misconfigurations
trivy config Dockerfile

# Scan Kubernetes manifests
trivy config ./k8s-manifests/
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Container Security Scan
on:
  push:
    branches: [main]
  pull_request:

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'

      - name: Upload Trivy scan results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'
```

### GitLab CI

```yaml
container_scanning:
  stage: security
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  variables:
    FULL_IMAGE_NAME: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  script:
    - trivy image --exit-code 0 --format template --template "@/contrib/gitlab.tpl"
      --output gl-container-scanning-report.json $FULL_IMAGE_NAME
    - trivy image --exit-code 1 --severity CRITICAL $FULL_IMAGE_NAME
  artifacts:
    reports:
      container_scanning: gl-container-scanning-report.json
```

### Jenkins Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        stage('Security Scan') {
            steps {
                sh '''
                    trivy image --exit-code 1 \
                      --severity HIGH,CRITICAL \
                      --format json \
                      --output trivy-report.json \
                      myapp:${BUILD_NUMBER}
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.json'
                }
            }
        }
    }
}
```

## Policy Configuration

### Trivy Policy with OPA/Rego

Create `.trivy/policy.rego` for custom policy enforcement:

```rego
package trivy

deny[msg] {
    input.Results[_].Vulnerabilities[_].Severity == "CRITICAL"
    msg := "Critical vulnerabilities found in image"
}

deny[msg] {
    input.Results[_].Vulnerabilities[vuln]
    vuln.FixedVersion != ""
    vuln.Severity == "HIGH"
    msg := sprintf("Fixable HIGH vulnerability: %s", [vuln.VulnerabilityID])
}
```

### Ignore File Configuration

Create `.trivyignore` for accepted risks:

```
# Accepted risk: vulnerability in test dependency only
CVE-2023-12345

# Accepted until expiry date
CVE-2024-67890 exp:2025-06-01
```

## SBOM Generation and Management

```bash
# Generate CycloneDX SBOM
trivy image --format cyclonedx --output sbom-cyclonedx.json myapp:latest

# Generate SPDX SBOM
trivy image --format spdx-json --output sbom-spdx.json myapp:latest

# Scan an existing SBOM for new vulnerabilities
trivy sbom sbom-cyclonedx.json
```

## Monitoring and Reporting

| Metric | Description | Target |
|--------|-------------|--------|
| Images scanned per day | Total images passing through scanning pipeline | All production images |
| Critical CVE count | Open critical vulnerabilities across all images | 0 in production |
| Mean time to patch | Average days from CVE publication to patched image | < 7 days |
| SBOM coverage | Percentage of production images with generated SBOMs | 100% |
| Scan duration | Average time per image scan | < 2 minutes |

## References

- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [Trivy GitHub Repository](https://github.com/aquasecurity/trivy)
- [Trivy Operator for Kubernetes](https://aquasecurity.github.io/trivy-operator/)
- [Aqua Security Platform](https://www.aquasec.com/products/)
- [CycloneDX SBOM Specification](https://cyclonedx.org/specification/overview/)

===== END FILE =====


===== FILE: skills/implementing-aqua-security-for-container-scanning/LICENSE =====

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


===== FILE: skills/implementing-aqua-security-for-container-scanning/assets/template.md =====
# Container Scanning Implementation Template

## Image Registry Scanning Configuration

| Registry | URL | Auth Method | Auto-scan | Schedule |
|----------|-----|-------------|-----------|----------|
| | | | [ ] Yes | |

## Severity Threshold Policy

| Environment | Block Critical | Block High | Block Medium | Block Unfixable |
|-------------|---------------|------------|--------------|-----------------|
| Development | [ ] | [ ] | [ ] | [ ] |
| Staging | [x] | [ ] | [ ] | [ ] |
| Production | [x] | [x] | [ ] | [ ] |

## Accepted Risk Register (.trivyignore)

| CVE ID | Package | Reason | Accepted By | Expiry Date |
|--------|---------|--------|-------------|-------------|
| | | | | |

## SBOM Tracking

| Image | SBOM Format | Storage Location | Last Generated |
|-------|-------------|------------------|----------------|
| | [ ] CycloneDX [ ] SPDX | | |

===== END FILE =====


===== FILE: skills/implementing-aqua-security-for-container-scanning/references/api-reference.md =====
# API Reference: Container Image Vulnerability Scanner (Trivy)

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| trivy CLI | >=0.50 | Container image scanning (invoked via subprocess) |

## CLI Usage

```bash
python scripts/agent.py \
  --images nginx:latest alpine:3.19 myapp:v1.2 \
  --severity CRITICAL,HIGH \
  --sbom \
  --output-dir /reports/ \
  --output trivy_report.json
```

## Functions

### `check_trivy_installed() -> bool`
Verifies Trivy CLI is available in PATH by running `trivy --version`.

### `scan_image(image, severity, ignore_unfixed) -> dict`
Runs `trivy image --format json --severity CRITICAL,HIGH <image>`. Returns parsed JSON output.

### `scan_image_misconfig(image) -> dict`
Runs `trivy image --scanners misconfig` to detect Dockerfile and config issues.

### `scan_image_secrets(image) -> dict`
Runs `trivy image --scanners secret` to find embedded secrets in image layers.

### `generate_sbom(image, output_path) -> bool`
Runs `trivy image --format cyclonedx` to produce CycloneDX SBOM.

### `parse_vuln_results(scan_data) -> list`
Extracts vulnerability details from Trivy JSON: VulnerabilityID, PkgName, InstalledVersion, FixedVersion, Severity.

### `compute_summary(vulns) -> dict`
Counts vulnerabilities by severity level (CRITICAL/HIGH/MEDIUM/LOW) and fixable count.

### `scan_multiple_images(images, severity) -> dict`
Orchestrates scanning of multiple images and aggregates results.

## Trivy CLI Commands Used

| Command | Purpose |
|---------|---------|
| `trivy image --format json` | Vulnerability scan with JSON output |
| `trivy image --scanners misconfig` | Misconfiguration detection |
| `trivy image --scanners secret` | Secret detection in layers |
| `trivy image --format cyclonedx` | SBOM generation |

## Output Schema

```json
{
  "images": [{"image": "nginx:latest", "summary": {"CRITICAL": 2, "HIGH": 5, "fixable": 4}}],
  "overall_summary": {"CRITICAL": 2, "HIGH": 5, "total": 7}
}
```

===== END FILE =====


===== FILE: skills/implementing-aqua-security-for-container-scanning/references/standards.md =====
# Standards Reference for Container Scanning

## NIST SP 800-190: Application Container Security Guide

| Recommendation | Trivy Coverage |
|---------------|---------------|
| 4.1 Image vulnerabilities | CVE scanning of OS packages and app dependencies |
| 4.2 Image configuration defects | IaC misconfig scanning of Dockerfiles |
| 4.3 Embedded malware | Secret scanning detects embedded credentials |
| 4.4 Embedded cleartext secrets | Secret scanner with regex and entropy detection |
| 4.5 Use of untrusted images | Registry scanning and image provenance verification |

## CIS Docker Benchmark v1.6 Alignment

| CIS Control | Trivy Check |
|-------------|-------------|
| 4.1 Ensure image is created from a trusted base | Base image vulnerability scanning |
| 4.3 Ensure unnecessary packages are not installed | SBOM generation reveals full package inventory |
| 4.6 Add HEALTHCHECK instruction | Dockerfile misconfiguration check |
| 4.9 Ensure COPY instead of ADD | Dockerfile misconfiguration check |
| 4.10 Ensure secrets are not stored in Dockerfiles | Secret detection in filesystem scan |

## Vulnerability Database Sources

| Source | Coverage | Update Frequency |
|--------|----------|------------------|
| NVD (NIST) | All CVEs | Continuous |
| Alpine SecDB | Alpine Linux packages | Daily |
| Debian Security Tracker | Debian packages | Daily |
| Ubuntu CVE Tracker | Ubuntu packages | Daily |
| Red Hat OVAL | RHEL/CentOS packages | Daily |
| GitHub Advisory Database | Language packages | Continuous |
| Go Vulnerability Database | Go modules | Continuous |
| RustSec Advisory Database | Rust crates | Continuous |

===== END FILE =====


===== FILE: skills/implementing-aqua-security-for-container-scanning/references/workflows.md =====
# Aqua Security Container Scanning Workflows

## Workflow 1: CI/CD Image Gate

```
Developer commits code
       |
Docker image built in CI
       |
Trivy scans image for vulnerabilities
       |
[No Critical/High] --> Image pushed to registry
[Critical found] --> Pipeline fails, image rejected
       |
SBOM generated and stored alongside image
       |
Image tagged with scan metadata
       |
Kubernetes admission controller validates scan results
       |
Deployment proceeds only with scanned images
```

## Workflow 2: Registry Continuous Scanning

```
New image pushed to container registry
       |
Trivy Operator detects new image tag
       |
Scheduled scan triggered
       |
VulnerabilityReport CR created in cluster
       |
New CVE published in NVD
       |
Re-scan of all running images
       |
Alert generated for newly affected images
       |
Remediation ticket created automatically
```

## Workflow 3: SBOM-Based Vulnerability Tracking

```
Image scanned, SBOM generated (CycloneDX/SPDX)
       |
SBOM stored in artifact repository
       |
New CVE published
       |
SBOM re-scanned without rebuilding image
       |
Affected images identified across fleet
       |
Prioritized patching based on exposure and severity
```

===== END FILE =====


===== FILE: skills/implementing-aqua-security-for-container-scanning/scripts/agent.py =====
#!/usr/bin/env python3
"""Container image vulnerability scanning agent using Trivy CLI via subprocess."""

import argparse
import json
import logging
import os
import subprocess
import sys
from datetime import datetime
from typing import List

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def check_trivy_installed() -> bool:
    """Check if Trivy CLI is available."""
    try:
        result = subprocess.run(["trivy", "--version"], capture_output=True, text=True, timeout=10)
        return result.returncode == 0
    except FileNotFoundError:
        return False


def scan_image(image: str, severity: str = "CRITICAL,HIGH",
               ignore_unfixed: bool = True) -> dict:
    """Scan a container image for vulnerabilities using Trivy."""
    cmd = ["trivy", "image", "--format", "json", "--severity", severity, image]
    if ignore_unfixed:
        cmd.append("--ignore-unfixed")
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        if result.stdout:
            return json.loads(result.stdout)
    except (subprocess.TimeoutExpired, json.JSONDecodeError) as exc:
        logger.error("Trivy scan failed for %s: %s", image, exc)
    return {}


def scan_image_misconfig(image: str) -> dict:
    """Scan a container image for misconfigurations."""
    cmd = ["trivy", "image", "--format", "json", "--scanners", "misconfig", image]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        if result.stdout:
            return json.loads(result.stdout)
    except (subprocess.TimeoutExpired, json.JSONDecodeError) as exc:
        logger.error("Misconfig scan failed: %s", exc)
    return {}


def scan_image_secrets(image: str) -> dict:
    """Scan a container image for embedded secrets."""
    cmd = ["trivy", "image", "--format", "json", "--scanners", "secret", image]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        if result.stdout:
            return json.loads(result.stdout)
    except (subprocess.TimeoutExpired, json.JSONDecodeError) as exc:
        logger.error("Secret scan failed: %s", exc)
    return {}


def generate_sbom(image: str, output_path: str) -> bool:
    """Generate SBOM in CycloneDX format for an image."""
    cmd = ["trivy", "image", "--format", "cyclonedx", "--output", output_path, image]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        return result.returncode == 0
    except subprocess.TimeoutExpired:
        return False


def parse_vuln_results(scan_data: dict) -> List[dict]:
    """Parse Trivy JSON output into structured vulnerability list."""
    vulns = []
    for result in scan_data.get("Results", []):
        target = result.get("Target", "")
        for vuln in result.get("Vulnerabilities", []):
            vulns.append({
                "target": target,
                "vuln_id": vuln.get("VulnerabilityID", ""),
                "pkg_name": vuln.get("PkgName", ""),
                "installed": vuln.get("InstalledVersion", ""),
                "fixed": vuln.get("FixedVersion", ""),
                "severity": vuln.get("Severity", ""),
                "title": vuln.get("Title", ""),
            })
    return vulns


def compute_summary(vulns: List[dict]) -> dict:
    """Compute severity summary from vulnerability list."""
    summary = {"CRITICAL": 0, "HIGH": 0, "MEDIUM": 0, "LOW": 0}
    for v in vulns:
        sev = v.get("severity", "").upper()
        if sev in summary:
            summary[sev] += 1
    summary["total"] = len(vulns)
    fixable = sum(1 for v in vulns if v.get("fixed"))
    summary["fixable"] = fixable
    return summary


def scan_multiple_images(images: List[str], severity: str) -> dict:
    """Scan multiple container images and aggregate results."""
    report = {"scan_date": datetime.utcnow().isoformat(), "images": []}
    for image in images:
        logger.info("Scanning %s...", image)
        scan_data = scan_image(image, severity)
        vulns = parse_vuln_results(scan_data)
        report["images"].append({
            "image": image, "vulnerabilities": vulns,
            "summary": compute_summary(vulns),
        })
    totals = {"CRITICAL": 0, "HIGH": 0, "MEDIUM": 0, "LOW": 0, "total": 0}
    for img in report["images"]:
        for k in totals:
            totals[k] += img["summary"].get(k, 0)
    report["overall_summary"] = totals
    return report


def main():
    parser = argparse.ArgumentParser(description="Container Image Vulnerability Scanner (Trivy)")
    parser.add_argument("--images", nargs="+", required=True, help="Container images to scan")
    parser.add_argument("--severity", default="CRITICAL,HIGH", help="Severity filter")
    parser.add_argument("--sbom", action="store_true", help="Generate SBOM for each image")
    parser.add_argument("--output-dir", default=".", help="Output directory")
    parser.add_argument("--output", default="trivy_report.json")
    args = parser.parse_args()

    if not check_trivy_installed():
        logger.error("Trivy CLI not found. Install: https://aquasecurity.github.io/trivy/")
        sys.exit(1)

    os.makedirs(args.output_dir, exist_ok=True)
    report = scan_multiple_images(args.images, args.severity)

    if args.sbom:
        for image in args.images:
            sbom_name = image.replace("/", "_").replace(":", "_") + "_sbom.json"
            sbom_path = os.path.join(args.output_dir, sbom_name)
            generate_sbom(image, sbom_path)

    out_path = os.path.join(args.output_dir, args.output)
    with open(out_path, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", out_path)
    print(json.dumps(report["overall_summary"], indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-aqua-security-for-container-scanning/scripts/process.py =====
#!/usr/bin/env python3
"""
Trivy Container Scanning Report Aggregator

Processes Trivy JSON scan results and generates consolidated
vulnerability reports across multiple container images.
"""

import json
import os
import sys
import subprocess
from datetime import datetime
from collections import defaultdict


def run_trivy_scan(image: str, output_file: str) -> dict:
    cmd = [
        "trivy", "image",
        "--format", "json",
        "--output", output_file,
        "--severity", "CRITICAL,HIGH,MEDIUM,LOW",
        image,
    ]
    result = subprocess.run(cmd, capture_output=True, text=True)
    if result.returncode not in (0, 1):
        print(f"Trivy scan failed for {image}: {result.stderr}")
        return {}
    with open(output_file) as f:
        return json.load(f)


def parse_trivy_results(scan_data: dict) -> dict:
    summary = {
        "vulnerabilities": [],
        "severity_counts": defaultdict(int),
        "fixable_count": 0,
        "packages_affected": set(),
    }
    for result in scan_data.get("Results", []):
        target = result.get("Target", "")
        target_type = result.get("Type", "")
        for vuln in result.get("Vulnerabilities", []):
            entry = {
                "id": vuln.get("VulnerabilityID"),
                "severity": vuln.get("Severity", "UNKNOWN"),
                "package": vuln.get("PkgName"),
                "installed_version": vuln.get("InstalledVersion"),
                "fixed_version": vuln.get("FixedVersion"),
                "title": vuln.get("Title", ""),
                "target": target,
                "target_type": target_type,
            }
            summary["vulnerabilities"].append(entry)
            summary["severity_counts"][entry["severity"]] += 1
            summary["packages_affected"].add(entry["package"])
            if entry["fixed_version"]:
                summary["fixable_count"] += 1

    summary["packages_affected"] = list(summary["packages_affected"])
    summary["severity_counts"] = dict(summary["severity_counts"])
    return summary


def generate_fleet_report(images: list) -> dict:
    report = {
        "generated_at": datetime.utcnow().isoformat() + "Z",
        "total_images": len(images),
        "total_vulnerabilities": 0,
        "total_critical": 0,
        "total_fixable": 0,
        "severity_summary": defaultdict(int),
        "top_cves": defaultdict(int),
        "image_reports": [],
    }

    for i, image in enumerate(images):
        print(f"Scanning {i+1}/{len(images)}: {image}")
        output_file = f"/tmp/trivy_scan_{i}.json"
        scan_data = run_trivy_scan(image, output_file)
        if not scan_data:
            continue

        parsed = parse_trivy_results(scan_data)
        vuln_count = len(parsed["vulnerabilities"])
        report["total_vulnerabilities"] += vuln_count
        report["total_critical"] += parsed["severity_counts"].get("CRITICAL", 0)
        report["total_fixable"] += parsed["fixable_count"]

        for sev, count in parsed["severity_counts"].items():
            report["severity_summary"][sev] += count

        for vuln in parsed["vulnerabilities"]:
            report["top_cves"][vuln["id"]] += 1

        report["image_reports"].append({
            "image": image,
            "total_vulnerabilities": vuln_count,
            "severity_counts": parsed["severity_counts"],
            "fixable": parsed["fixable_count"],
            "critical_vulns": [
                v for v in parsed["vulnerabilities"] if v["severity"] == "CRITICAL"
            ],
        })

    report["severity_summary"] = dict(report["severity_summary"])
    top_sorted = sorted(report["top_cves"].items(), key=lambda x: x[1], reverse=True)[:20]
    report["top_cves"] = dict(top_sorted)
    return report


def print_fleet_report(report: dict) -> None:
    print(f"\n{'='*60}")
    print(f"Container Fleet Vulnerability Report")
    print(f"Generated: {report['generated_at']}")
    print(f"{'='*60}")
    print(f"Images scanned: {report['total_images']}")
    print(f"Total vulnerabilities: {report['total_vulnerabilities']}")
    print(f"Total critical: {report['total_critical']}")
    print(f"Total fixable: {report['total_fixable']}")
    print(f"\nSeverity Breakdown:")
    for sev in ["CRITICAL", "HIGH", "MEDIUM", "LOW", "UNKNOWN"]:
        count = report["severity_summary"].get(sev, 0)
        if count:
            print(f"  {sev:12s}: {count}")
    print(f"\nImages by Risk (sorted by critical count):")
    for img in sorted(
        report["image_reports"],
        key=lambda x: x["severity_counts"].get("CRITICAL", 0),
        reverse=True,
    ):
        crits = img["severity_counts"].get("CRITICAL", 0)
        print(f"  {img['image']:50s} | Critical: {crits} | Total: {img['total_vulnerabilities']}")


def main():
    images_env = os.environ.get("SCAN_IMAGES", "")
    if images_env:
        images = [i.strip() for i in images_env.split(",") if i.strip()]
    else:
        images = [
            "python:3.11-slim",
            "node:20-alpine",
            "nginx:latest",
            "golang:1.22-alpine",
        ]
        print("No SCAN_IMAGES env var set, using default image list")

    report = generate_fleet_report(images)
    print_fleet_report(report)

    output = f"container_scan_report_{datetime.utcnow().strftime('%Y%m%d_%H%M%S')}.json"
    with open(output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\nReport saved to: {output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-code-signing-for-artifacts/SKILL.md =====
---
name: implementing-code-signing-for-artifacts
description: 'Implements code signing for build artifacts (binaries, packages, containers)
  using GPG, Sigstore, and platform-specific signing tools, establishing trust chains
  and verifying signatures in deployment pipelines. Use when establishing artifact
  integrity checks against supply-chain tampering, proving authenticity to customers,
  building zero-trust pipelines that reject unsigned artifacts, or meeting SLSA Level
  2+ provenance requirements.

  '
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- cicd
- code-signing
- supply-chain
- sigstore
- secure-sdlc
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
- T1610
- T1611
---

# Implementing Code Signing for Artifacts

## When to Use

- When establishing artifact integrity verification to prevent supply chain tampering
- When compliance requires cryptographic proof that build artifacts are authentic and unmodified
- When distributing software to customers who need to verify publisher identity
- When implementing zero-trust deployment pipelines that reject unsigned artifacts
- When meeting SLSA Level 2+ requirements for provenance and integrity

**Do not use** for encrypting artifacts (signing provides integrity, not confidentiality), for container image signing specifically (use cosign), or for source code authentication (use commit signing).

## Prerequisites

- GPG key pair for traditional signing or Sigstore account for keyless signing
- Code signing certificate from a Certificate Authority for public distribution
- CI/CD pipeline with access to signing keys or identity provider
- Verification infrastructure in deployment pipelines

## Workflow

### Step 1: Generate and Manage Signing Keys

```bash
# Generate GPG key for artifact signing
gpg --full-generate-key --batch <<EOF
Key-Type: eddsa
Key-Curve: ed25519
Subkey-Type: eddsa
Subkey-Curve: ed25519
Name-Real: CI Build System
Name-Email: ci-signing@company.com
Expire-Date: 1y
%no-protection
EOF

# Export public key for distribution
gpg --armor --export ci-signing@company.com > signing-key.pub

# Export private key for CI/CD (store in secrets manager)
gpg --armor --export-secret-keys ci-signing@company.com > signing-key.priv
```

### Step 2: Sign Build Artifacts in CI/CD

```yaml
# .github/workflows/build-sign.yml
name: Build and Sign

on:
  push:
    tags: ['v*']

jobs:
  build-sign:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      id-token: write  # For Sigstore keyless signing
    steps:
      - uses: actions/checkout@v4

      - name: Build artifacts
        run: |
          make build
          sha256sum dist/* > dist/checksums.sha256

      - name: Import GPG Key
        run: |
          echo "${{ secrets.GPG_PRIVATE_KEY }}" | gpg --batch --import
          gpg --list-secret-keys

      - name: Sign artifacts
        run: |
          for file in dist/*; do
            gpg --detach-sign --armor --local-user ci-signing@company.com "$file"
          done

      - name: Install cosign for keyless signing
        uses: sigstore/cosign-installer@v3

      - name: Keyless sign with Sigstore
        run: |
          for file in dist/*.tar.gz; do
            cosign sign-blob "$file" \
              --output-signature "${file}.sig" \
              --output-certificate "${file}.cert" \
              --yes
          done

      - name: Create Release with signed artifacts
        uses: softprops/action-gh-release@v2
        with:
          files: |
            dist/*
            dist/*.asc
            dist/*.sig
            dist/*.cert
```

### Step 3: Verify Signatures in Deployment Pipeline

```bash
# Verify GPG signature
gpg --import signing-key.pub
gpg --verify artifact.tar.gz.asc artifact.tar.gz

# Verify Sigstore keyless signature
cosign verify-blob artifact.tar.gz \
  --signature artifact.tar.gz.sig \
  --certificate artifact.tar.gz.cert \
  --certificate-identity ci-signing@company.com \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com

# Verify checksums
sha256sum --check checksums.sha256
```

### Step 4: Sign npm Packages with Provenance

```json
{
  "scripts": {
    "prepublishOnly": "npm run build && npm run test"
  },
  "publishConfig": {
    "provenance": true
  }
}
```

```bash
# Publish npm package with provenance attestation
npm publish --provenance
```

## Key Concepts

| Term | Definition |
|------|------------|
| Code Signing | Cryptographic process of signing software artifacts to verify publisher identity and artifact integrity |
| Detached Signature | Signature stored in a separate file from the artifact, allowing independent distribution |
| Keyless Signing | Sigstore's approach using short-lived certificates tied to OIDC identities instead of long-lived keys |
| Provenance | Metadata describing how, where, and by whom an artifact was built |
| Transparency Log | Append-only log (Rekor) that records all signing events for public auditability |
| Trust Chain | Hierarchical chain from root CA to signing certificate establishing trust in the signer's identity |
| SLSA | Supply-chain Levels for Software Artifacts — framework defining levels of supply chain security |

## Tools & Systems

- **GPG/PGP**: Traditional asymmetric cryptography tool for signing and verifying artifacts
- **Sigstore (cosign)**: Modern keyless signing infrastructure using OIDC identity and transparency logs
- **Rekor**: Sigstore's transparency log recording all signing events immutably
- **Fulcio**: Sigstore's certificate authority issuing short-lived certificates bound to OIDC identities
- **notation**: Microsoft's artifact signing tool for OCI registries (Project Notary v2)

## Common Scenarios

### Scenario: Establishing Signed Release Pipeline

**Context**: An open-source project needs to sign release artifacts so users can verify authenticity and detect tampering.

**Approach**:
1. Use Sigstore keyless signing in GitHub Actions (no key management overhead)
2. Sign all release binaries with `cosign sign-blob` using OIDC identity
3. Generate and sign checksums file for bulk verification
4. Upload signatures, certificates, and checksums alongside release artifacts
5. Document verification instructions in the project README
6. Add verification step to the Homebrew formula or apt repository

**Pitfalls**: GPG key compromise requires revoking and re-signing all artifacts. Sigstore keyless signing avoids this by using ephemeral keys. Long-lived signing keys in CI/CD secrets are a supply chain risk if the CI system is compromised.

## Output Format

```
Artifact Signing Report
========================
Pipeline: Build and Sign v2.3.0
Date: 2026-02-23
Signing Method: Sigstore Keyless + GPG

SIGNED ARTIFACTS:
  app-v2.3.0-linux-amd64.tar.gz
    GPG:      PASS (ci-signing@company.com, EdDSA/Ed25519)
    Sigstore: PASS (Rekor entry: 24658135, Fulcio cert issued)
    SHA256:   a1b2c3d4...

  app-v2.3.0-darwin-arm64.tar.gz
    GPG:      PASS
    Sigstore: PASS (Rekor entry: 24658136)
    SHA256:   e5f6g7h8...

  checksums.sha256
    GPG:      PASS (detached signature)

TRANSPARENCY LOG:
  Entries recorded: 3
  Log index range: 24658135-24658137
  Verification: https://search.sigstore.dev
```

===== END FILE =====


===== FILE: skills/implementing-code-signing-for-artifacts/LICENSE =====

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


===== FILE: skills/implementing-code-signing-for-artifacts/assets/template.md =====
# Code Signing Templates

## GitHub Actions: Build, Sign, and Release

```yaml
name: Signed Release
on:
  push:
    tags: ['v*']

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: make build-all
      - uses: sigstore/cosign-installer@v3
      - name: Sign artifacts
        run: |
          for f in dist/*.tar.gz; do
            cosign sign-blob "$f" --output-signature "${f}.sig" --output-certificate "${f}.cert" --yes
          done
          sha256sum dist/*.tar.gz > dist/checksums.sha256
      - uses: softprops/action-gh-release@v2
        with:
          files: dist/*
```

## Verification Script

```bash
#!/bin/bash
set -euo pipefail
# verify-release.sh <artifact> <signature> <certificate>
ARTIFACT="$1"
SIGNATURE="$2"
CERTIFICATE="$3"

echo "Verifying: $ARTIFACT"
sha256sum "$ARTIFACT"

cosign verify-blob "$ARTIFACT" \
  --signature "$SIGNATURE" \
  --certificate "$CERTIFICATE" \
  --certificate-identity "https://github.com/org/repo/.github/workflows/release.yml@refs/tags/*" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com"

echo "Verification: PASSED"
```

===== END FILE =====


===== FILE: skills/implementing-code-signing-for-artifacts/references/api-reference.md =====
# API Reference: Code Signing Verification Agent

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| cryptography | >=41.0 | Ed25519 key generation, signing, and verification |

## CLI Usage

```bash
# Generate keypair
python scripts/agent.py --generate-keys --output-dir /keys/

# Sign artifact
python scripts/agent.py --sign build/app.tar.gz --private-key /keys/signing_key.pem

# Verify artifacts
python scripts/agent.py \
  --artifacts build/app.tar.gz build/lib.so \
  --public-key /keys/signing_key.pub \
  --output-dir /reports/
```

## Functions

### `generate_ed25519_keypair(output_dir) -> dict`
Calls `Ed25519PrivateKey.generate()`, serializes to PEM using `private_bytes()` and `public_bytes()`.

### `sign_artifact(file_path, private_key_path) -> dict`
Loads PEM key via `serialization.load_pem_private_key()`, calls `private_key.sign(data)`. Writes 64-byte signature to `.sig` file.

### `verify_signature(file_path, signature_path, public_key_path) -> dict`
Loads public key, calls `public_key.verify(signature, data)`. Catches `InvalidSignature`.

### `verify_cosign_signature(image) -> dict`
Runs `cosign verify <image>` via subprocess for container image signature verification.

### `batch_verify(artifacts, public_key_path) -> list`
Verifies multiple artifacts against the same public key.

## cryptography API Used

| Class/Method | Purpose |
|-------------|---------|
| `Ed25519PrivateKey.generate()` | Generate signing keypair |
| `private_key.sign(data)` | Sign data (returns 64 bytes) |
| `public_key.verify(signature, data)` | Verify signature |
| `serialization.load_pem_private_key()` | Load PEM private key |

## Output Schema

```json
{
  "summary": {"total": 3, "valid": 2, "invalid": 1},
  "verifications": [{"file": "app.tar.gz", "valid": true, "algorithm": "Ed25519"}]
}
```

===== END FILE =====


===== FILE: skills/implementing-code-signing-for-artifacts/references/standards.md =====
# Standards Reference: Code Signing for Artifacts

## SLSA Framework

### Level 1: Build Provenance
- Document the build process (source, builder, dependencies)
- Provenance metadata available for all artifacts

### Level 2: Signed Provenance
- Build provenance is signed by the build platform
- Signatures tied to authenticated builder identity

### Level 3: Hardened Build Platform
- Build process runs on a hardened, isolated platform
- Provenance is non-falsifiable by the build service

## NIST SSDF (SP 800-218)

### PS.2: Provide Mechanism for Verifying Software Release Integrity
- PS.2.1: Make integrity verification information available to consumers
- Code signing provides cryptographic proof of artifact integrity
- Transparency logs provide public auditability of signing events

### PW.4: Reuse Existing, Well-Secured Software
- Verify signatures of third-party dependencies before integration
- Establish trust anchors for acceptable signing identities

## CIS Software Supply Chain Security

### Artifacts (AR) Controls
- AR-1: Sign all build artifacts using cryptographic signatures
- AR-2: Verify artifact signatures before deployment
- AR-3: Use transparency logs for signing event auditability
- AR-4: Rotate signing keys on a defined schedule

## Executive Order 14028

- Section 4(e): Agencies shall employ tools and processes to maintain trusted source code supply chains
- SBOM and artifact signing required for federal software suppliers
- Sigstore adoption recommended by CISA for open-source supply chain security

===== END FILE =====


===== FILE: skills/implementing-code-signing-for-artifacts/references/workflows.md =====
# Workflow Reference: Code Signing for Artifacts

## Signing Pipeline Flow

```
Build Artifacts
       │
       ▼
┌──────────────────┐
│ Generate          │
│ Checksums         │
└──────┬───────────┘
       │
       ├──────────────────────┐
       ▼                      ▼
┌──────────────┐    ┌──────────────┐
│ GPG Sign     │    │ Sigstore     │
│ (detached)   │    │ Keyless Sign │
└──────┬───────┘    └──────┬───────┘
       │                    │
       │                    ▼
       │            ┌──────────────┐
       │            │ Rekor Log    │
       │            │ Entry        │
       │            └──────┬───────┘
       │                    │
       └──────────┬─────────┘
                  ▼
       ┌──────────────────┐
       │ Publish Release  │
       │ + Signatures     │
       └──────────────────┘
```

## Signing Methods Comparison

| Method | Key Management | Identity | Verification | Best For |
|--------|---------------|----------|--------------|----------|
| GPG | Manual key lifecycle | Key fingerprint | gpg --verify | Traditional projects |
| Sigstore Keyless | No keys to manage | OIDC identity | cosign verify-blob | Modern CI/CD |
| Code Signing Cert | CA-issued certificate | Organization name | Platform-specific | Windows/macOS apps |
| npm Provenance | Automated | GitHub Actions OIDC | npm audit signatures | npm packages |

===== END FILE =====


===== FILE: skills/implementing-code-signing-for-artifacts/scripts/agent.py =====
#!/usr/bin/env python3
"""Code signing verification agent using cryptography library for Ed25519/RSA signature operations."""

import argparse
import hashlib
import json
import logging
import os
import subprocess
import sys
from datetime import datetime
from typing import List

try:
    from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
    from cryptography.hazmat.primitives import serialization
    from cryptography.exceptions import InvalidSignature
except ImportError:
    sys.exit("cryptography required: pip install cryptography")

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def compute_file_hash(file_path: str, algorithm: str = "sha256") -> str:
    """Compute hash digest of a file."""
    h = hashlib.new(algorithm)
    with open(file_path, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            h.update(chunk)
    return h.hexdigest()


def generate_ed25519_keypair(output_dir: str) -> dict:
    """Generate Ed25519 signing keypair."""
    private_key = Ed25519PrivateKey.generate()
    private_bytes = private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=serialization.NoEncryption(),
    )
    public_bytes = private_key.public_key().public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo,
    )
    priv_path = os.path.join(output_dir, "signing_key.pem")
    pub_path = os.path.join(output_dir, "signing_key.pub")
    with open(priv_path, "wb") as f:
        f.write(private_bytes)
    with open(pub_path, "wb") as f:
        f.write(public_bytes)
    return {"private_key": priv_path, "public_key": pub_path, "algorithm": "Ed25519"}


def sign_artifact(file_path: str, private_key_path: str) -> dict:
    """Sign a file artifact using Ed25519."""
    with open(private_key_path, "rb") as f:
        private_key = serialization.load_pem_private_key(f.read(), password=None)
    with open(file_path, "rb") as f:
        data = f.read()
    signature = private_key.sign(data)
    sig_path = file_path + ".sig"
    with open(sig_path, "wb") as f:
        f.write(signature)
    return {
        "file": file_path,
        "signature_file": sig_path,
        "hash_sha256": hashlib.sha256(data).hexdigest(),
        "algorithm": "Ed25519",
    }


def verify_signature(file_path: str, signature_path: str, public_key_path: str) -> dict:
    """Verify an Ed25519 signature against a file."""
    with open(public_key_path, "rb") as f:
        public_key = serialization.load_pem_public_key(f.read())
    with open(file_path, "rb") as f:
        data = f.read()
    with open(signature_path, "rb") as f:
        signature = f.read()
    try:
        public_key.verify(signature, data)
        return {"file": file_path, "valid": True, "algorithm": "Ed25519"}
    except InvalidSignature:
        return {"file": file_path, "valid": False, "error": "Invalid signature"}


def verify_cosign_signature(image: str) -> dict:
    """Verify container image signature using cosign CLI."""
    try:
        result = subprocess.run(
            ["cosign", "verify", image], capture_output=True, text=True, timeout=30)
        return {"image": image, "verified": result.returncode == 0,
                "output": result.stdout[:500]}
    except FileNotFoundError:
        return {"image": image, "error": "cosign not installed"}


def batch_verify(artifacts: List[dict], public_key_path: str) -> List[dict]:
    """Verify signatures for multiple artifacts."""
    results = []
    for art in artifacts:
        result = verify_signature(art["file"], art["signature"], public_key_path)
        results.append(result)
    return results


def generate_report(artifacts: List[str], public_key_path: str) -> dict:
    """Generate code signing verification report."""
    report = {"analysis_date": datetime.utcnow().isoformat(), "verifications": []}
    for art_path in artifacts:
        sig_path = art_path + ".sig"
        if os.path.isfile(sig_path):
            result = verify_signature(art_path, sig_path, public_key_path)
        else:
            result = {"file": art_path, "valid": False, "error": "No signature file found"}
        result["hash_sha256"] = compute_file_hash(art_path)
        report["verifications"].append(result)
    valid = sum(1 for v in report["verifications"] if v.get("valid"))
    report["summary"] = {
        "total": len(report["verifications"]),
        "valid": valid,
        "invalid": len(report["verifications"]) - valid,
    }
    return report


def main():
    parser = argparse.ArgumentParser(description="Code Signing Verification Agent")
    parser.add_argument("--artifacts", nargs="+", help="Files to verify")
    parser.add_argument("--public-key", help="Path to public key PEM")
    parser.add_argument("--generate-keys", action="store_true", help="Generate new keypair")
    parser.add_argument("--sign", help="File to sign (requires --private-key)")
    parser.add_argument("--private-key", help="Path to private key PEM")
    parser.add_argument("--output-dir", default=".")
    parser.add_argument("--output", default="signing_report.json")
    args = parser.parse_args()

    os.makedirs(args.output_dir, exist_ok=True)
    if args.generate_keys:
        keys = generate_ed25519_keypair(args.output_dir)
        print(json.dumps(keys, indent=2))
        return
    if args.sign and args.private_key:
        result = sign_artifact(args.sign, args.private_key)
        print(json.dumps(result, indent=2))
        return
    if args.artifacts and args.public_key:
        report = generate_report(args.artifacts, args.public_key)
        out_path = os.path.join(args.output_dir, args.output)
        with open(out_path, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report["summary"], indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-code-signing-for-artifacts/scripts/process.py =====
#!/usr/bin/env python3
"""
Artifact Code Signing Pipeline Script

Signs build artifacts using GPG and/or Sigstore cosign,
generates checksums, and produces a signing report.

Usage:
    python process.py --artifacts-dir ./dist --method gpg --gpg-key ci-signing@company.com
    python process.py --artifacts-dir ./dist --method sigstore --output signing-report.json
"""

import argparse
import hashlib
import json
import os
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime, timezone
from pathlib import Path


@dataclass
class SigningResult:
    artifact: str
    sha256: str
    gpg_signature: str = ""
    gpg_verified: bool = False
    sigstore_signature: str = ""
    sigstore_certificate: str = ""
    sigstore_log_index: str = ""
    error: str = ""


def compute_sha256(file_path: str) -> str:
    """Compute SHA256 hash of a file."""
    sha256 = hashlib.sha256()
    with open(file_path, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            sha256.update(chunk)
    return sha256.hexdigest()


def sign_with_gpg(file_path: str, gpg_key: str) -> dict:
    """Sign a file with GPG detached signature."""
    sig_path = f"{file_path}.asc"
    cmd = [
        "gpg", "--batch", "--yes",
        "--detach-sign", "--armor",
        "--local-user", gpg_key,
        "--output", sig_path,
        file_path
    ]

    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        if proc.returncode == 0:
            return {"signature": sig_path, "error": ""}
        return {"signature": "", "error": proc.stderr[:200]}
    except (subprocess.TimeoutExpired, FileNotFoundError) as e:
        return {"signature": "", "error": str(e)}


def sign_with_cosign(file_path: str) -> dict:
    """Sign a file with Sigstore cosign keyless signing."""
    sig_path = f"{file_path}.sig"
    cert_path = f"{file_path}.cert"

    cmd = [
        "cosign", "sign-blob", file_path,
        "--output-signature", sig_path,
        "--output-certificate", cert_path,
        "--yes"
    ]

    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
        if proc.returncode == 0:
            log_index = ""
            for line in proc.stderr.split("\n"):
                if "tlog entry" in line.lower() or "log index" in line.lower():
                    parts = line.split(":")
                    if len(parts) > 1:
                        log_index = parts[-1].strip()
            return {
                "signature": sig_path,
                "certificate": cert_path,
                "log_index": log_index,
                "error": ""
            }
        return {"signature": "", "certificate": "", "log_index": "", "error": proc.stderr[:200]}
    except (subprocess.TimeoutExpired, FileNotFoundError) as e:
        return {"signature": "", "certificate": "", "log_index": "", "error": str(e)}


def verify_gpg_signature(file_path: str, sig_path: str) -> bool:
    """Verify a GPG detached signature."""
    cmd = ["gpg", "--verify", sig_path, file_path]
    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
        return proc.returncode == 0
    except (subprocess.TimeoutExpired, FileNotFoundError):
        return False


def generate_checksums(artifacts_dir: str, artifacts: list) -> str:
    """Generate checksums file for all artifacts."""
    checksums_path = os.path.join(artifacts_dir, "checksums.sha256")
    lines = []
    for result in artifacts:
        filename = os.path.basename(result.artifact)
        lines.append(f"{result.sha256}  {filename}")

    with open(checksums_path, "w") as f:
        f.write("\n".join(lines) + "\n")

    return checksums_path


def main():
    parser = argparse.ArgumentParser(description="Artifact Code Signing Pipeline")
    parser.add_argument("--artifacts-dir", required=True, help="Directory containing artifacts to sign")
    parser.add_argument("--method", default="both", choices=["gpg", "sigstore", "both"])
    parser.add_argument("--gpg-key", default=None, help="GPG key identity for signing")
    parser.add_argument("--output", default="signing-report.json")
    parser.add_argument("--extensions", nargs="*", default=[".tar.gz", ".zip", ".whl", ".deb", ".rpm"],
                        help="File extensions to sign")
    args = parser.parse_args()

    artifacts_dir = os.path.abspath(args.artifacts_dir)
    results = []

    files_to_sign = []
    for f in sorted(Path(artifacts_dir).iterdir()):
        if f.is_file() and any(str(f).endswith(ext) for ext in args.extensions):
            files_to_sign.append(str(f))

    if not files_to_sign:
        print(f"[WARN] No artifacts found matching extensions: {args.extensions}")
        sys.exit(0)

    print(f"[*] Signing {len(files_to_sign)} artifacts in {artifacts_dir}")

    for file_path in files_to_sign:
        filename = os.path.basename(file_path)
        result = SigningResult(artifact=file_path, sha256=compute_sha256(file_path))

        if args.method in ("gpg", "both") and args.gpg_key:
            gpg_result = sign_with_gpg(file_path, args.gpg_key)
            result.gpg_signature = gpg_result["signature"]
            if gpg_result["signature"]:
                result.gpg_verified = verify_gpg_signature(file_path, gpg_result["signature"])
                print(f"  [GPG] {filename}: {'OK' if result.gpg_verified else 'FAILED'}")
            else:
                print(f"  [GPG] {filename}: ERROR - {gpg_result['error']}")

        if args.method in ("sigstore", "both"):
            cosign_result = sign_with_cosign(file_path)
            result.sigstore_signature = cosign_result.get("signature", "")
            result.sigstore_certificate = cosign_result.get("certificate", "")
            result.sigstore_log_index = cosign_result.get("log_index", "")
            if result.sigstore_signature:
                print(f"  [Sigstore] {filename}: OK (log: {result.sigstore_log_index})")
            else:
                print(f"  [Sigstore] {filename}: ERROR - {cosign_result.get('error', 'unknown')}")
                result.error = cosign_result.get("error", "")

        results.append(result)

    checksums_path = generate_checksums(artifacts_dir, results)
    print(f"\n[*] Checksums: {checksums_path}")

    report = {
        "metadata": {
            "signing_date": datetime.now(timezone.utc).isoformat(),
            "method": args.method,
            "artifacts_count": len(results)
        },
        "artifacts": [
            {
                "file": os.path.basename(r.artifact),
                "sha256": r.sha256,
                "gpg_signed": bool(r.gpg_signature),
                "gpg_verified": r.gpg_verified,
                "sigstore_signed": bool(r.sigstore_signature),
                "sigstore_log_index": r.sigstore_log_index
            }
            for r in results
        ]
    }

    output_path = os.path.abspath(args.output)
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[*] Report: {output_path}")

    all_signed = all(
        (r.gpg_verified or args.method == "sigstore") and
        (bool(r.sigstore_signature) or args.method == "gpg")
        for r in results
    )
    print(f"\n[{'PASS' if all_signed else 'FAIL'}] All artifacts signed: {all_signed}")
    if not all_signed:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-fuzz-testing-in-cicd-with-aflplusplus/SKILL.md =====
---
name: implementing-fuzz-testing-in-cicd-with-aflplusplus
description: Integrates AFL++ coverage-guided fuzzing into CI/CD pipelines, covering harness construction, AFL++/AddressSanitizer/CmpLog instrumentation builds, and persistent-mode fuzzing to discover memory-corruption and input-handling vulnerabilities in C/C++ code. Use when adding automated fuzz testing to a build pipeline or hunting for memory-safety bugs in native/compiled applications.
domain: cybersecurity
subdomain: devsecops
tags:
- aflplusplus
- fuzz-testing
- cicd
- coverage-guided-fuzzing
- security-testing
- vulnerability-discovery
- afl
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
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
- T1005
- T1059
---

# Implementing Fuzz Testing in CI/CD with AFL++

## Overview

AFL++ (American Fuzzy Lop Plus Plus) is a community-maintained fork of AFL that provides state-of-the-art coverage-guided fuzz testing for discovering vulnerabilities in compiled applications. AFL++ uses genetic algorithms to mutate inputs, tracking code coverage to find new execution paths that trigger crashes, hangs, and undefined behavior. In CI/CD environments, AFL++ can be integrated to continuously test parsers, protocol handlers, file format processors, and any code that handles untrusted input. AFL++ supports persistent mode for high-speed fuzzing (up to 100,000+ executions per second), custom mutators, QEMU mode for binary-only fuzzing, and CmpLog/RedQueen for automatic dictionary extraction.


## When to Use

- When deploying or configuring implementing fuzz testing in cicd with aflplusplus capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Linux-based CI runners (AFL++ does not support Windows natively)
- GCC or Clang compiler toolchain
- AFL++ installed (`apt install aflplusplus` or built from source)
- Target application with harness functions isolating input processing
- Seed corpus of valid input samples

## Core Concepts

### Coverage-Guided Fuzzing

AFL++ instruments the target binary at compile time (or via QEMU/Frida for binary-only targets) to track which code paths each input exercises. When a mutated input triggers a new code path, it is saved to the corpus for further mutation. This feedback loop enables AFL++ to systematically explore program state space.

### Instrumentation Modes

| Mode | Use Case | Performance |
|------|----------|-------------|
| `afl-clang-fast` (LTO) | Source available, best performance | Highest |
| `afl-clang-fast` | Source available, standard | High |
| `afl-gcc-fast` | GCC-based projects | High |
| `QEMU mode` | Binary-only, no source | Medium |
| `Frida mode` | Binary-only, cross-platform | Medium |
| `Unicorn mode` | Firmware, embedded | Low |

### Persistent Mode

Persistent mode avoids fork overhead by fuzzing within a loop:

```c
#include <unistd.h>

__AFL_FUZZ_INIT();

int main() {
    __AFL_INIT();
    unsigned char *buf = __AFL_FUZZ_TESTCASE_BUF;

    while (__AFL_LOOP(10000)) {
        int len = __AFL_FUZZ_TESTCASE_LEN;
        // Process buf[0..len-1]
        parse_input(buf, len);
    }
    return 0;
}
```

## Workflow

### Step 1 --- Build the Fuzzing Harness

Create a harness that feeds AFL++ input to the target function:

```c
// fuzz_harness.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "target_parser.h"

__AFL_FUZZ_INIT();

int main() {
    __AFL_INIT();
    unsigned char *buf = __AFL_FUZZ_TESTCASE_BUF;

    while (__AFL_LOOP(10000)) {
        int len = __AFL_FUZZ_TESTCASE_LEN;
        if (len < 4) continue;

        // Reset state between iterations
        parser_context_t ctx;
        parser_init(&ctx);
        parser_process(&ctx, buf, len);
        parser_cleanup(&ctx);
    }
    return 0;
}
```

### Step 2 --- Compile with AFL++ Instrumentation

```bash
# Standard instrumentation
export CC=afl-clang-fast
export CXX=afl-clang-fast++

# Enable AddressSanitizer for better crash detection
export AFL_USE_ASAN=1

# Build the target with instrumentation
$CC -o fuzz_harness fuzz_harness.c -ltarget_parser -fsanitize=address

# Build a CmpLog binary for better coverage
$CC -o fuzz_harness_cmplog fuzz_harness.c -ltarget_parser \
  -fsanitize=address -DCMPLOG
```

### Step 3 --- Prepare Seed Corpus

```bash
mkdir -p corpus/
# Add valid input samples
cp test_inputs/* corpus/
# Minimize the corpus
afl-cmin -i corpus/ -o corpus_min/ -- ./fuzz_harness @@
# Further minimize individual inputs
mkdir -p corpus_tmin/
for f in corpus_min/*; do
    afl-tmin -i "$f" -o "corpus_tmin/$(basename $f)" -- ./fuzz_harness @@
done
```

### Step 4 --- Configure CI/CD Integration

**GitHub Actions:**

```yaml
name: Fuzz Testing
on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'  # Nightly fuzzing

jobs:
  fuzz:
    runs-on: ubuntu-latest
    timeout-minutes: 120
    steps:
      - uses: actions/checkout@v4

      - name: Install AFL++
        run: |
          sudo apt-get update
          sudo apt-get install -y aflplusplus

      - name: Restore corpus cache
        uses: actions/cache@v4
        with:
          path: corpus/
          key: fuzz-corpus-${{ github.sha }}
          restore-keys: fuzz-corpus-

      - name: Build fuzzing harness
        run: |
          export CC=afl-clang-fast
          export AFL_USE_ASAN=1
          make fuzz_harness

      - name: Run AFL++ fuzzing (CI mode)
        env:
          AFL_CMPLOG_ONLY_NEW: 1
          AFL_FAST_CAL: 1
          AFL_NO_STARTUP_CALIBRATION: 1
        run: |
          mkdir -p findings/
          timeout 7200 afl-fuzz \
            -S ci_fuzzer \
            -i corpus/ \
            -o findings/ \
            -t 5000 \
            -- ./fuzz_harness @@ || true

      - name: Check for crashes
        run: |
          CRASHES=$(find findings/ -path "*/crashes/*" -not -name "README.txt" | wc -l)
          echo "Found $CRASHES unique crashes"
          if [ "$CRASHES" -gt 0 ]; then
            echo "::error::AFL++ found $CRASHES crashes"
            for crash in findings/*/crashes/*; do
              [ -f "$crash" ] && echo "Crash: $crash ($(wc -c < $crash) bytes)"
            done
            exit 1
          fi

      - name: Update corpus cache
        if: always()
        run: |
          afl-cmin -i findings/ci_fuzzer/queue/ -o corpus/ -- ./fuzz_harness @@
```

### Step 5 --- Parallel Fuzzing for Nightly Runs

```bash
# Launch multiple secondary instances for better coverage
for i in $(seq 1 $(nproc)); do
    afl-fuzz -S fuzzer_$i \
      -i corpus/ \
      -o findings/ \
      -- ./fuzz_harness @@ &
done

# Wait for all fuzzers
wait

# Merge and minimize corpus
afl-cmin -i findings/*/queue/ -o corpus_merged/ -- ./fuzz_harness @@
```

### Step 6 --- Crash Triage

```bash
# Reproduce and categorize crashes
for crash in findings/*/crashes/*; do
    echo "=== Testing: $crash ==="
    timeout 5 ./fuzz_harness_asan "$crash" 2>&1 | head -20
    echo "---"
done

# Deduplicate crashes by stack trace
afl-collect findings/ crashes_deduped/ -- ./fuzz_harness @@
```

## CI/CD Best Practices for AFL++

| Setting | CI Short Run | Nightly Long Run |
|---------|-------------|-----------------|
| Duration | 30-60 min | 4-24 hours |
| Mode | `-S` (secondary only) | `-S` (no `-M` for CI) |
| `AFL_CMPLOG_ONLY_NEW` | 1 | 1 |
| `AFL_FAST_CAL` | 1 | 0 |
| `AFL_NO_STARTUP_CALIBRATION` | 1 | 0 |
| Corpus caching | Required | Required |
| Parallel instances | 1-2 | nproc |

## Monitoring Fuzzing Campaigns

```bash
# View fuzzing statistics
afl-whatsup findings/

# Key metrics to track:
# - Total paths found (code coverage indicator)
# - Unique crashes / unique hangs
# - Stability percentage (should be >90%)
# - Exec speed (execs/sec)
# - Cycles done (full corpus cycles completed)
```

## References

- [AFL++ Documentation](https://aflplus.plus/docs/)
- [AFL++ GitHub Repository](https://github.com/AFLplusplus/AFLplusplus)
- [AFL++ Fuzzing in Depth Guide](https://aflplus.plus/docs/fuzzing_in_depth/)
- [Google Testing Handbook - AFL++](https://appsec.guide/docs/fuzzing/c-cpp/aflpp/)
- [OWASP Fuzzing Guide](https://owasp.org/www-community/Fuzzing)

===== END FILE =====


===== FILE: skills/implementing-fuzz-testing-in-cicd-with-aflplusplus/LICENSE =====

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


===== FILE: skills/implementing-fuzz-testing-in-cicd-with-aflplusplus/assets/template.md =====
# Fuzz Testing Implementation Template

## Target Application

| Field | Value |
|-------|-------|
| Application Name | |
| Target Function | |
| Language | [ ] C [ ] C++ [ ] Other |
| Input Type | [ ] File [ ] Network [ ] Stdin |

## Fuzzing Configuration

| Parameter | Value |
|-----------|-------|
| Instrumentation | [ ] afl-clang-fast [ ] afl-gcc-fast [ ] QEMU |
| Sanitizer | [ ] ASan [ ] UBSan [ ] MSan [ ] TSan |
| Mode | [ ] Persistent [ ] Fork |
| CmpLog | [ ] Enabled [ ] Disabled |
| Timeout per exec | ms |
| CI run duration | minutes |
| Nightly duration | hours |

## Corpus Management

| Item | Location |
|------|----------|
| Seed corpus | |
| Minimized corpus | |
| CI cache key | |

## Crash Tracking

| Crash ID | CWE | Severity | Crash File | Stack Trace Summary | Fix Status |
|----------|-----|----------|------------|---------------------|------------|
| | | | | | |

===== END FILE =====


===== FILE: skills/implementing-fuzz-testing-in-cicd-with-aflplusplus/references/api-reference.md =====
# API Reference — Implementing Fuzz Testing in CI/CD with AFL++

## Libraries Used
- **subprocess**: Execute AFL++ toolchain commands (afl-clang-fast, afl-fuzz, afl-cmin)
- **pathlib**: File system operations for corpus and crash management

## CLI Interface

```
python agent.py compile --source target.c --output target_fuzz [--compiler afl-clang-fast]
python agent.py fuzz --binary ./target_fuzz --input seeds/ --output findings/ [--duration 300]
python agent.py triage --binary ./target_fuzz --crashes-dir findings/default/crashes/
python agent.py stats --stats-file findings/default/fuzzer_stats
```

## Core Functions

### `compile_target(source_file, output_binary, compiler)`
Compiles target with AFL++ instrumentation. Sets `AFL_HARDEN=1` for memory sanitizers.

### `run_fuzzer(binary, input_dir, output_dir, duration_seconds, memory_limit)`
Runs `afl-fuzz` with headless mode (`AFL_NO_UI=1`), time-limited (`-V` flag).

**Environment Variables Set:**
| Variable | Value | Purpose |
|----------|-------|---------|
| `AFL_SKIP_CPUFREQ` | 1 | Skip CPU frequency check (CI/CD) |
| `AFL_NO_UI` | 1 | Headless mode for CI environments |
| `AFL_I_DONT_CARE_ABOUT_MISSING_CRASHES` | 1 | Continue on crash dir issues |

### `parse_fuzzer_stats(stats_file)`
Parses AFL++ `fuzzer_stats` file. Key metrics: `execs_per_sec`, `paths_total`, `saved_crashes`, `bitmap_cvg`.

### `triage_crashes(binary, crashes_dir)`
Re-runs crash inputs through the binary and classifies by signal (SIGSEGV, SIGABRT, etc.).

### `minimize_corpus(binary, input_dir, output_dir, timeout)`
Runs `afl-cmin` to remove redundant seeds from the corpus.

## AFL++ Commands Used

| Command | Purpose |
|---------|---------|
| `afl-clang-fast` | Compile with LLVM-based instrumentation |
| `afl-fuzz -i <in> -o <out> -- <binary>` | Main fuzzing loop |
| `afl-cmin -i <in> -o <out> -- <binary>` | Corpus minimization |
| `afl-tmin -i <crash> -o <min> -- <binary>` | Test case minimization |

## Dependencies
AFL++ must be installed: `apt install aflplusplus` or build from source.
```
pip install  # No Python packages needed beyond stdlib
```

===== END FILE =====


===== FILE: skills/implementing-fuzz-testing-in-cicd-with-aflplusplus/references/standards.md =====
# Standards Reference for Fuzz Testing

## NIST SP 800-53 Rev 5 Controls

| Control | Description | Fuzzing Alignment |
|---------|-------------|-------------------|
| SA-11(5) | Penetration Testing | Fuzz testing discovers vulnerabilities through automated input mutation |
| SA-11(8) | Dynamic Code Analysis | AFL++ provides runtime analysis with instrumented binaries |
| SI-10 | Information Input Validation | Fuzzing validates input handling robustness |
| SI-17 | Fail-Safe Procedures | Crash detection ensures failures are handled safely |

## OWASP Testing Guide v4.2

- **WSTG-INPV-07**: Testing for Input Validation --- AFL++ systematically tests boundary conditions
- **WSTG-ERRH-01**: Error Handling --- Crash analysis reveals improper error handling

## CWE Categories Commonly Found by Fuzzing

| CWE | Name | AFL++ Detection Method |
|-----|------|----------------------|
| CWE-120 | Buffer Overflow | ASan crash on out-of-bounds write |
| CWE-125 | Out-of-Bounds Read | ASan crash on invalid read |
| CWE-416 | Use After Free | ASan detects freed memory access |
| CWE-476 | NULL Pointer Dereference | SIGSEGV on null deref |
| CWE-190 | Integer Overflow | UBSan detects arithmetic overflow |
| CWE-787 | Out-of-Bounds Write | ASan detects heap/stack buffer overflow |
| CWE-400 | Uncontrolled Resource Consumption | Timeout detection for hangs |

## Fuzzing Maturity Levels

| Level | Description | CI Integration |
|-------|-------------|----------------|
| 1 Basic | Manual ad-hoc fuzzing | None |
| 2 Structured | Harness-based with corpus management | PR-triggered short runs |
| 3 Continuous | Nightly campaigns with crash tracking | Nightly + corpus caching |
| 4 Optimized | Multi-tool (AFL++, libFuzzer), crash dedup, coverage tracking | Full CI/CD integration with gating |

===== END FILE =====


===== FILE: skills/implementing-fuzz-testing-in-cicd-with-aflplusplus/references/workflows.md =====
# AFL++ Fuzz Testing Workflows

## Workflow 1: CI Pipeline Integration

```
Code pushed to branch
       |
Fuzzing harness compiled with afl-clang-fast + ASan
       |
Corpus restored from CI cache
       |
AFL++ runs in secondary mode for fixed duration
       |
[No crashes] --> Corpus updated in cache, pipeline passes
[Crashes found] --> Pipeline fails, crash artifacts uploaded
       |
Developer triages crashes
       |
Fix applied, re-run confirms no regression
```

## Workflow 2: Nightly Fuzzing Campaign

```
Scheduled nightly trigger (cron)
       |
Build instrumented binary + CmpLog binary
       |
Restore merged corpus from last run
       |
Launch parallel AFL++ instances (nproc count)
       |
Run for 4-8 hours
       |
Collect results from all instances
       |
afl-cmin merges and minimizes corpus
       |
Deduplicate crashes by stack hash
       |
New crashes create Jira/GitHub issues automatically
       |
Updated corpus cached for next run
```

## Workflow 3: Crash Triage and Fix

```
Crash file identified in findings/
       |
Reproduce crash with ASan-instrumented binary
       |
Capture ASan stack trace and error type
       |
Minimize crash input with afl-tmin
       |
Identify root cause from stack trace
       |
Develop fix and add crash input as regression test
       |
Verify fix by re-running AFL++ with crash input
       |
Update corpus to include edge case inputs
```

===== END FILE =====


===== FILE: skills/implementing-fuzz-testing-in-cicd-with-aflplusplus/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing AFL++ fuzz testing in CI/CD pipelines."""

import json
import argparse
import subprocess
import os
from pathlib import Path


def compile_target(source_file, output_binary, compiler="afl-clang-fast"):
    """Compile target binary with AFL++ instrumentation."""
    cmd = [compiler, "-g", "-O1", "-fno-omit-frame-pointer", "-o", output_binary, source_file]
    env = os.environ.copy()
    env["AFL_HARDEN"] = "1"
    result = subprocess.run(cmd, capture_output=True, text=True, env=env, timeout=120)
    return {
        "source": source_file,
        "binary": output_binary,
        "compiler": compiler,
        "returncode": result.returncode,
        "stdout": result.stdout[:500],
        "stderr": result.stderr[:500],
        "instrumented": result.returncode == 0,
    }


def prepare_corpus(seed_dir, corpus_dir):
    """Prepare and minimize seed corpus using afl-cmin."""
    Path(corpus_dir).mkdir(parents=True, exist_ok=True)
    seeds = list(Path(seed_dir).glob("*"))
    if not seeds:
        # Create a minimal seed if none provided
        minimal = Path(seed_dir) / "seed_minimal"
        minimal.write_bytes(b"AAAA")
        seeds = [minimal]
    return {
        "seed_dir": str(seed_dir),
        "corpus_dir": str(corpus_dir),
        "seed_count": len(seeds),
        "seeds": [str(s) for s in seeds[:50]],
    }


def minimize_corpus(binary, input_dir, output_dir, timeout=60):
    """Minimize seed corpus using afl-cmin."""
    Path(output_dir).mkdir(parents=True, exist_ok=True)
    cmd = ["afl-cmin", "-i", input_dir, "-o", output_dir, "-t", str(timeout * 1000), "--", binary]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    minimized = list(Path(output_dir).glob("*"))
    return {
        "input_count": len(list(Path(input_dir).glob("*"))),
        "output_count": len(minimized),
        "returncode": result.returncode,
    }


def run_fuzzer(binary, input_dir, output_dir, duration_seconds=300, memory_limit="512"):
    """Run AFL++ fuzzer for a specified duration."""
    Path(output_dir).mkdir(parents=True, exist_ok=True)
    env = os.environ.copy()
    env["AFL_SKIP_CPUFREQ"] = "1"
    env["AFL_I_DONT_CARE_ABOUT_MISSING_CRASHES"] = "1"
    env["AFL_NO_UI"] = "1"
    cmd = [
        "afl-fuzz",
        "-i", input_dir,
        "-o", output_dir,
        "-m", memory_limit,
        "-V", str(duration_seconds),
        "--", binary,
    ]
    result = subprocess.run(cmd, capture_output=True, text=True, env=env, timeout=duration_seconds + 60)
    stats = parse_fuzzer_stats(os.path.join(output_dir, "default", "fuzzer_stats"))
    crashes_dir = os.path.join(output_dir, "default", "crashes")
    crash_files = list(Path(crashes_dir).glob("id:*")) if os.path.isdir(crashes_dir) else []
    return {
        "binary": binary,
        "duration_seconds": duration_seconds,
        "returncode": result.returncode,
        "stats": stats,
        "crashes_found": len(crash_files),
        "crash_files": [str(f) for f in crash_files[:50]],
    }


def parse_fuzzer_stats(stats_file):
    """Parse AFL++ fuzzer_stats file into a dict."""
    stats = {}
    try:
        with open(stats_file, "r") as f:
            for line in f:
                if ":" in line:
                    key, _, value = line.partition(":")
                    stats[key.strip()] = value.strip()
    except FileNotFoundError:
        return {"error": "fuzzer_stats not found"}
    return {
        "execs_done": stats.get("execs_done", "0"),
        "execs_per_sec": stats.get("execs_per_sec", "0"),
        "paths_total": stats.get("paths_total", "0"),
        "paths_found": stats.get("paths_found", "0"),
        "unique_crashes": stats.get("saved_crashes", "0"),
        "unique_hangs": stats.get("saved_hangs", "0"),
        "stability": stats.get("stability", "unknown"),
        "bitmap_cvg": stats.get("bitmap_cvg", "unknown"),
    }


def triage_crashes(binary, crashes_dir):
    """Triage crash inputs to deduplicate and classify."""
    crash_files = sorted(Path(crashes_dir).glob("id:*"))
    results = []
    for crash_file in crash_files[:100]:
        cmd = [binary]
        try:
            proc = subprocess.run(
                cmd, input=crash_file.read_bytes(),
                capture_output=True, timeout=5
            )
            results.append({
                "file": str(crash_file),
                "returncode": proc.returncode,
                "signal": -proc.returncode if proc.returncode < 0 else None,
                "stderr_snippet": proc.stderr[:200].decode("utf-8", errors="replace"),
                "crash_type": _classify_signal(proc.returncode),
            })
        except subprocess.TimeoutExpired:
            results.append({"file": str(crash_file), "crash_type": "hang/timeout"})
    return {
        "total_crashes": len(crash_files),
        "triaged": len(results),
        "by_type": _count_by(results, "crash_type"),
        "results": results,
    }


def _classify_signal(returncode):
    signal_map = {-6: "SIGABRT", -11: "SIGSEGV", -8: "SIGFPE", -4: "SIGILL", -7: "SIGBUS"}
    return signal_map.get(returncode, f"exit({returncode})")


def _count_by(items, key):
    counts = {}
    for item in items:
        val = item.get(key, "unknown")
        counts[val] = counts.get(val, 0) + 1
    return counts


def main():
    parser = argparse.ArgumentParser(description="AFL++ Fuzz Testing CI/CD Agent")
    sub = parser.add_subparsers(dest="command")
    c = sub.add_parser("compile", help="Compile target with AFL++ instrumentation")
    c.add_argument("--source", required=True)
    c.add_argument("--output", required=True)
    c.add_argument("--compiler", default="afl-clang-fast")
    f = sub.add_parser("fuzz", help="Run AFL++ fuzzer")
    f.add_argument("--binary", required=True)
    f.add_argument("--input", required=True)
    f.add_argument("--output", required=True)
    f.add_argument("--duration", type=int, default=300, help="Duration in seconds")
    f.add_argument("--memory", default="512", help="Memory limit in MB")
    t = sub.add_parser("triage", help="Triage crash inputs")
    t.add_argument("--binary", required=True)
    t.add_argument("--crashes-dir", required=True)
    s = sub.add_parser("stats", help="Parse fuzzer stats")
    s.add_argument("--stats-file", required=True)
    args = parser.parse_args()
    if args.command == "compile":
        result = compile_target(args.source, args.output, args.compiler)
    elif args.command == "fuzz":
        result = run_fuzzer(args.binary, args.input, args.output, args.duration, args.memory)
    elif args.command == "triage":
        result = triage_crashes(args.binary, args.crashes_dir)
    elif args.command == "stats":
        result = parse_fuzzer_stats(args.stats_file)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-fuzz-testing-in-cicd-with-aflplusplus/scripts/process.py =====
#!/usr/bin/env python3
"""
AFL++ Fuzzing Results Analyzer

Parses AFL++ output directories and generates reports on
crash findings, corpus growth, and coverage statistics.
"""

import json
import os
import sys
from datetime import datetime
from pathlib import Path
from collections import defaultdict


def parse_fuzzer_stats(stats_file: str) -> dict:
    stats = {}
    if not os.path.exists(stats_file):
        return stats
    with open(stats_file) as f:
        for line in f:
            line = line.strip()
            if ":" in line:
                key, value = line.split(":", 1)
                stats[key.strip()] = value.strip()
    return stats


def count_files_in_dir(directory: str) -> int:
    if not os.path.isdir(directory):
        return 0
    return len([f for f in os.listdir(directory) if f != "README.txt" and os.path.isfile(os.path.join(directory, f))])


def analyze_fuzzer_instance(instance_dir: str) -> dict:
    name = os.path.basename(instance_dir)
    stats = parse_fuzzer_stats(os.path.join(instance_dir, "fuzzer_stats"))

    return {
        "name": name,
        "start_time": stats.get("start_time", ""),
        "last_update": stats.get("last_update", ""),
        "execs_done": int(stats.get("execs_done", 0)),
        "execs_per_sec": float(stats.get("execs_per_sec", 0)),
        "corpus_count": count_files_in_dir(os.path.join(instance_dir, "queue")),
        "crashes_total": count_files_in_dir(os.path.join(instance_dir, "crashes")),
        "hangs_total": count_files_in_dir(os.path.join(instance_dir, "hangs")),
        "paths_total": int(stats.get("paths_total", 0)),
        "paths_found": int(stats.get("paths_found", 0)),
        "stability": stats.get("stability", ""),
        "cycles_done": int(stats.get("cycles_done", 0)),
        "bitmap_cvg": stats.get("bitmap_cvg", ""),
        "command_line": stats.get("command_line", ""),
    }


def collect_crash_info(instance_dir: str) -> list:
    crashes_dir = os.path.join(instance_dir, "crashes")
    crashes = []
    if not os.path.isdir(crashes_dir):
        return crashes
    for fname in sorted(os.listdir(crashes_dir)):
        if fname == "README.txt":
            continue
        fpath = os.path.join(crashes_dir, fname)
        if os.path.isfile(fpath):
            crashes.append({
                "file": fname,
                "path": fpath,
                "size": os.path.getsize(fpath),
                "instance": os.path.basename(instance_dir),
            })
    return crashes


def analyze_campaign(findings_dir: str) -> dict:
    report = {
        "findings_dir": findings_dir,
        "analyzed_at": datetime.utcnow().isoformat() + "Z",
        "instances": [],
        "total_execs": 0,
        "total_crashes": 0,
        "total_hangs": 0,
        "total_corpus": 0,
        "all_crashes": [],
        "avg_execs_per_sec": 0,
    }

    instance_dirs = []
    for entry in sorted(os.listdir(findings_dir)):
        full_path = os.path.join(findings_dir, entry)
        if os.path.isdir(full_path) and os.path.exists(os.path.join(full_path, "fuzzer_stats")):
            instance_dirs.append(full_path)

    if not instance_dirs:
        print(f"No fuzzer instances found in {findings_dir}")
        return report

    exec_speeds = []
    for inst_dir in instance_dirs:
        inst = analyze_fuzzer_instance(inst_dir)
        report["instances"].append(inst)
        report["total_execs"] += inst["execs_done"]
        report["total_crashes"] += inst["crashes_total"]
        report["total_hangs"] += inst["hangs_total"]
        report["total_corpus"] += inst["corpus_count"]
        if inst["execs_per_sec"] > 0:
            exec_speeds.append(inst["execs_per_sec"])

        crashes = collect_crash_info(inst_dir)
        report["all_crashes"].extend(crashes)

    if exec_speeds:
        report["avg_execs_per_sec"] = round(sum(exec_speeds) / len(exec_speeds), 1)

    return report


def print_report(report: dict) -> None:
    print(f"\n{'='*60}")
    print(f"AFL++ Fuzzing Campaign Report")
    print(f"{'='*60}")
    print(f"Findings directory: {report['findings_dir']}")
    print(f"Analyzed at: {report['analyzed_at']}")
    print(f"Fuzzer instances: {len(report['instances'])}")
    print(f"\nAggregate Statistics:")
    print(f"  Total executions: {report['total_execs']:,}")
    print(f"  Avg exec/sec: {report['avg_execs_per_sec']:,.1f}")
    print(f"  Total corpus entries: {report['total_corpus']}")
    print(f"  Total unique crashes: {report['total_crashes']}")
    print(f"  Total hangs: {report['total_hangs']}")

    print(f"\nInstance Details:")
    for inst in report["instances"]:
        print(f"  {inst['name']:20s} | Execs: {inst['execs_done']:>12,} | "
              f"Speed: {inst['execs_per_sec']:>8.1f}/s | "
              f"Crashes: {inst['crashes_total']:3d} | "
              f"Corpus: {inst['corpus_count']:5d} | "
              f"Cycles: {inst['cycles_done']}")

    if report["all_crashes"]:
        print(f"\nCrash Files ({len(report['all_crashes'])} total):")
        for crash in report["all_crashes"][:20]:
            print(f"  [{crash['instance']}] {crash['file']} ({crash['size']} bytes)")
        if len(report["all_crashes"]) > 20:
            print(f"  ... and {len(report['all_crashes']) - 20} more")

    verdict = "PASS" if report["total_crashes"] == 0 else "FAIL"
    print(f"\nCI Verdict: {verdict}")


def main():
    if len(sys.argv) < 2:
        print("Usage: python process.py <findings_directory>")
        sys.exit(1)

    findings_dir = sys.argv[1]
    if not os.path.isdir(findings_dir):
        print(f"Directory not found: {findings_dir}")
        sys.exit(1)

    report = analyze_campaign(findings_dir)
    print_report(report)

    output = os.path.join(findings_dir, "campaign_report.json")
    with open(output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\nReport saved to: {output}")

    sys.exit(1 if report["total_crashes"] > 0 else 0)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-github-advanced-security-for-code-scanning/SKILL.md =====
---
name: implementing-github-advanced-security-for-code-scanning
description: Configures GitHub Advanced Security (code scanning with CodeQL, secret scanning, dependency review, and Dependabot alerts) to perform automated static analysis and vulnerability detection across repositories at enterprise scale, including custom CodeQL queries and CI workflow integration. Use when setting up or tuning code scanning, rolling out CodeQL across an organization, or shifting SAST left into pull request workflows.
domain: cybersecurity
subdomain: devsecops
tags:
- github-advanced-security
- codeql
- sast
- code-scanning
- supply-chain-security
- devops-security
- shift-left
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
---

# Implementing GitHub Advanced Security for Code Scanning

## Overview

GitHub Advanced Security (GHAS) integrates CodeQL-powered static application security testing directly into the GitHub development workflow. CodeQL treats code as data, enabling semantic analysis that identifies security vulnerabilities such as SQL injection, cross-site scripting, buffer overflows, and authentication flaws with significantly fewer false positives than traditional pattern-matching scanners. GHAS encompasses code scanning, secret scanning, dependency review, and Dependabot alerts to provide a comprehensive security posture for repositories.


## When to Use

- When deploying or configuring implementing github advanced security for code scanning capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- GitHub Enterprise Cloud or GitHub Enterprise Server 3.0+ with GHAS license
- Repository admin or organization owner permissions
- Familiarity with GitHub Actions workflow syntax (YAML)
- Supported languages: C/C++, C#, Go, Java/Kotlin, JavaScript/TypeScript, Python, Ruby, Swift

## Core Concepts

### CodeQL Analysis Engine

CodeQL compiles source code into a queryable database, then executes security-focused queries against that database. The query suites ship with hundreds of checks mapped to CWE identifiers and cover OWASP Top 10, SANS Top 25, and language-specific vulnerability patterns. Custom queries can be authored using the CodeQL query language (QL) to detect organization-specific anti-patterns.

### Default Setup vs. Advanced Setup

**Default Setup** enables code scanning with a single click from the repository's Code Security settings. GitHub automatically determines the languages present, selects appropriate query suites, and configures scanning triggers. This approach requires no workflow file and is ideal for rapid onboarding.

**Advanced Setup** generates a `.github/workflows/codeql.yml` workflow file that can be customized. Teams control scheduling, language matrices, build commands for compiled languages, additional query packs, and integration with third-party SARIF producers. Advanced setup is required when custom build steps, monorepo configurations, or private query packs are needed.

### Organization-Wide Rollout

For enterprises managing hundreds of repositories, GHAS supports configuring code scanning at scale using the organization-level security overview. Administrators can enable default setup across all eligible repositories, define custom security configurations, and monitor adoption through the security coverage dashboard.

## Workflow

### Step 1 --- Enable GHAS on the Organization

1. Navigate to Organization Settings > Code security and analysis
2. Enable GitHub Advanced Security for all repositories or selected repositories
3. Confirm license seat allocation (GHAS is billed per active committer)

### Step 2 --- Configure Default Setup for Quick Wins

1. Go to Repository Settings > Code security > Code scanning
2. Click "Set up" in the CodeQL analysis row and select "Default"
3. Review the auto-detected languages and query suite (default or extended)
4. Click "Enable CodeQL" to activate scanning on push and pull request events

### Step 3 --- Advanced Setup with Custom Workflow

Create `.github/workflows/codeql-analysis.yml`:

```yaml
name: "CodeQL Analysis"

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '30 2 * * 1'  # Weekly Monday 2:30 AM UTC

jobs:
  analyze:
    name: Analyze (${{ matrix.language }})
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
      actions: read

    strategy:
      fail-fast: false
      matrix:
        language: ['javascript-typescript', 'python', 'java-kotlin']

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          queries: +security-extended,security-and-quality
          # For compiled languages, add build commands below

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{ matrix.language }}"
```

### Step 4 --- Custom Query Packs

Install organization-specific query packs by referencing them in the workflow:

```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: java-kotlin
    packs: |
      my-org/java-custom-queries@1.0.0
      codeql/java-queries:cwe/cwe-089
```

### Step 5 --- Configure Branch Protection Rules

1. Navigate to Repository Settings > Branches > Branch protection rules
2. Enable "Require status checks to pass" and add the CodeQL analysis check
3. Enable "Require code scanning results" and set severity thresholds (e.g., block on High/Critical)

### Step 6 --- Secret Scanning and Push Protection

1. Enable secret scanning from Code security settings
2. Activate push protection to block commits containing detected secrets
3. Configure custom patterns for organization-specific secrets (API keys, internal tokens)

### Step 7 --- Dependency Review and Dependabot

1. Enable Dependabot alerts and security updates
2. Configure `.github/dependabot.yml` for automated dependency version updates
3. Enable dependency review enforcement on pull requests to block PRs that introduce known vulnerable dependencies

## Query Suite Reference

| Suite | Description | Use Case |
|-------|-------------|----------|
| `default` | High-confidence security queries | Production scanning with minimal false positives |
| `security-extended` | Broader security queries including lower-severity findings | Comprehensive security coverage |
| `security-and-quality` | Security plus code quality queries | Teams wanting both security and maintainability checks |
| Custom packs | Organization-authored queries | Detecting internal anti-patterns and compliance violations |

## Integration with Security Workflows

### SARIF Upload from Third-Party Tools

GHAS accepts SARIF (Static Analysis Results Interchange Format) uploads from external tools:

```yaml
- name: Upload SARIF
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: results.sarif
    category: "semgrep"
```

### Security Overview Dashboard

The organization-level security overview provides:
- Risk view showing repositories with open alerts by severity
- Coverage view showing GHAS feature enablement across repositories
- Alert trends over time for tracking remediation progress
- Filter by team, language, and alert type for targeted review

## Monitoring and Metrics

- Track mean time to remediate (MTTR) for code scanning alerts
- Monitor false positive rates and tune query configurations accordingly
- Review alert dismissal reasons to identify areas for developer training
- Use the API (`/repos/{owner}/{repo}/code-scanning/alerts`) for custom reporting dashboards

## Common Pitfalls

1. **Compiled language build failures** --- CodeQL requires successful compilation for C/C++, Java, C#, Go, and Swift; ensure build dependencies are available in the Actions runner
2. **Ignoring scheduled scans** --- Push/PR scanning misses vulnerabilities in dependencies; weekly scheduled scans catch newly disclosed CVEs in existing code
3. **Over-alerting with security-and-quality** --- Start with `default` suite and expand gradually to avoid developer alert fatigue
4. **Missing GHAS license seats** --- Only active committers to GHAS-enabled repositories consume license seats; plan capacity accordingly

## References

- [GitHub Code Scanning Documentation](https://docs.github.com/en/code-security/code-scanning)
- [CodeQL Documentation](https://codeql.github.com/docs/)
- [CodeQL Query Repository](https://github.com/github/codeql)
- [SARIF Specification](https://sarifweb.azurewebsites.net/)
- [GitHub Security Overview](https://docs.github.com/en/code-security/security-overview/about-security-overview)

===== END FILE =====


===== FILE: skills/implementing-github-advanced-security-for-code-scanning/LICENSE =====

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


===== FILE: skills/implementing-github-advanced-security-for-code-scanning/assets/template.md =====
# GHAS Code Scanning Implementation Template

## Organization Security Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| Organization | `_______________` | |
| GHAS License Seats | `_______________` | Active committers |
| Default Query Suite | [ ] default [ ] security-extended [ ] security-and-quality | |
| Branch Protection Enabled | [ ] Yes [ ] No | |
| Secret Scanning Enabled | [ ] Yes [ ] No | |
| Push Protection Enabled | [ ] Yes [ ] No | |
| Dependabot Enabled | [ ] Yes [ ] No | |

## Repository Enablement Tracker

| Repository | Languages | Setup Type | Scanning Active | Open Alerts | Date Enabled |
|------------|-----------|------------|-----------------|-------------|--------------|
| | | [ ] Default [ ] Advanced | [ ] Yes [ ] No | | |
| | | [ ] Default [ ] Advanced | [ ] Yes [ ] No | | |
| | | [ ] Default [ ] Advanced | [ ] Yes [ ] No | | |

## Custom Query Pack Registry

| Pack Name | Version | Description | Target Languages |
|-----------|---------|-------------|------------------|
| | | | |

## Alert Severity Gate Configuration

| Environment | Block on Critical | Block on High | Block on Medium | Block on Low |
|-------------|-------------------|---------------|-----------------|--------------|
| Production (main) | [x] Yes | [x] Yes | [ ] Yes | [ ] No |
| Staging (develop) | [x] Yes | [ ] Yes | [ ] No | [ ] No |
| Feature branches | [x] Yes | [ ] Yes | [ ] No | [ ] No |

## Secret Scanning Custom Patterns

| Pattern Name | Regex | Description | Alert Enabled | Push Protection |
|--------------|-------|-------------|---------------|-----------------|
| | | | [ ] Yes [ ] No | [ ] Yes [ ] No |

## Weekly Security Review Checklist

- [ ] Review new critical and high severity alerts
- [ ] Check alert dismissal reasons for quality
- [ ] Verify new repositories have scanning enabled
- [ ] Review Dependabot alerts and merge security updates
- [ ] Check secret scanning alerts for exposed credentials
- [ ] Update security overview dashboard metrics
- [ ] Review MTTR trends and identify bottlenecks

## Escalation Matrix

| Alert Severity | Response SLA | Escalation Contact | Action Required |
|----------------|-------------|--------------------|-----------------|
| Critical | 24 hours | Security Lead | Immediate remediation, potential incident |
| High | 72 hours | Team Lead | Prioritize in current sprint |
| Medium | 2 weeks | Developer | Schedule for next sprint |
| Low | 30 days | Developer | Add to backlog |

===== END FILE =====


===== FILE: skills/implementing-github-advanced-security-for-code-scanning/references/api-reference.md =====
# API Reference: Implementing GitHub Advanced Security for Code Scanning

## GitHub Code Scanning API

```bash
# List code scanning alerts
gh api /repos/OWNER/REPO/code-scanning/alerts?state=open

# Get specific alert
gh api /repos/OWNER/REPO/code-scanning/alerts/ALERT_NUMBER

# List analyses
gh api /repos/OWNER/REPO/code-scanning/analyses

# Upload SARIF
gh api /repos/OWNER/REPO/code-scanning/sarifs -X POST \
  -f commit_sha=SHA -f ref=refs/heads/main -f sarif=@results.sarif.gz
```

## Secret Scanning API

```bash
# List secret alerts
gh api /repos/OWNER/REPO/secret-scanning/alerts?state=open

# Update alert state
gh api /repos/OWNER/REPO/secret-scanning/alerts/ALERT_NUMBER -X PATCH \
  -f state=resolved -f resolution=revoked
```

## CodeQL Query Suites

| Suite | Description | False Positive Rate |
|-------|-------------|-------------------|
| `default` | High-confidence security | Low |
| `security-extended` | Broader security coverage | Medium |
| `security-and-quality` | Security + code quality | Higher |

## CodeQL Workflow (GitHub Actions)

```yaml
- uses: github/codeql-action/init@v3
  with:
    languages: ${{ matrix.language }}
    queries: +security-extended
- uses: github/codeql-action/autobuild@v3
- uses: github/codeql-action/analyze@v3
```

## Supported Languages

| Language | Build Required | Query Pack |
|----------|---------------|-----------|
| Python | No | codeql/python-queries |
| JavaScript/TypeScript | No | codeql/javascript-queries |
| Java/Kotlin | Yes | codeql/java-queries |
| C/C++ | Yes | codeql/cpp-queries |
| C# | Yes | codeql/csharp-queries |
| Go | Yes | codeql/go-queries |
| Ruby | No | codeql/ruby-queries |
| Swift | Yes | codeql/swift-queries |

### References

- GHAS Docs: https://docs.github.com/en/code-security/code-scanning
- CodeQL: https://codeql.github.com/docs/
- CodeQL Queries: https://github.com/github/codeql
- SARIF Spec: https://sarifweb.azurewebsites.net/

===== END FILE =====


===== FILE: skills/implementing-github-advanced-security-for-code-scanning/references/standards.md =====
# Standards and Frameworks Reference

## OWASP Top 10 (2021) Coverage by CodeQL

| OWASP Category | CodeQL CWE Coverage | Query Suite |
|----------------|---------------------|-------------|
| A01 Broken Access Control | CWE-22, CWE-284, CWE-639 | security-extended |
| A02 Cryptographic Failures | CWE-259, CWE-327, CWE-328 | security-extended |
| A03 Injection | CWE-77, CWE-78, CWE-79, CWE-89 | default |
| A04 Insecure Design | CWE-209, CWE-256, CWE-501 | security-and-quality |
| A05 Security Misconfiguration | CWE-16, CWE-611 | security-extended |
| A06 Vulnerable Components | Dependency Review / Dependabot | N/A (separate feature) |
| A07 Auth Failures | CWE-287, CWE-798 | default |
| A08 Data Integrity Failures | CWE-502, CWE-829 | security-extended |
| A09 Logging Failures | CWE-117, CWE-778 | security-and-quality |
| A10 SSRF | CWE-918 | default |

## NIST SP 800-218 (SSDF) Alignment

- **PO.3**: Define security requirements --- CodeQL enforces security policies through query suites
- **PW.4**: Reuse existing, well-secured software --- Dependabot ensures dependencies are patched
- **PW.7**: Review and test code for vulnerabilities --- Automated code scanning on every PR
- **PW.8**: Test executable code --- SARIF integration enables combining SAST with DAST results
- **RV.1**: Identify and confirm vulnerabilities --- Security overview tracks alerts across the organization

## CIS Software Supply Chain Security Guide

- **SCS-1**: Source code management security --- Branch protection rules, required reviewers
- **SCS-2**: Build pipelines --- CodeQL runs in GitHub Actions with pinned action versions
- **SCS-5**: Artifact management --- Dependency review prevents vulnerable packages from merging

## ISO 27001 Control Mapping

| ISO 27001 Control | GHAS Feature |
|--------------------|--------------|
| A.8.25 Secure development lifecycle | CodeQL in CI/CD pipeline |
| A.8.26 Application security requirements | Custom query packs for org standards |
| A.8.28 Secure coding | Real-time scanning on pull requests |
| A.8.29 Security testing in dev and acceptance | Required status checks with severity gates |
| A.8.31 Separation of environments | Branch protection and deployment rules |

===== END FILE =====


===== FILE: skills/implementing-github-advanced-security-for-code-scanning/references/workflows.md =====
# GHAS Implementation Workflows

## Workflow 1: Organization-Wide Enablement

```
1. Audit current repository inventory
   - List all repositories in the organization
   - Identify languages and build systems in use
   - Estimate active committer count for licensing
   |
2. Pilot phase (2-4 weeks)
   - Enable GHAS on 5-10 representative repositories
   - Use default setup for initial scanning
   - Collect baseline alert counts and false positive rates
   |
3. Triage pilot results
   - Review alerts by severity (Critical, High, Medium, Low)
   - Dismiss confirmed false positives with documented reasons
   - Create remediation issues for confirmed vulnerabilities
   |
4. Tune configuration
   - Adjust query suites based on false positive feedback
   - Write custom queries for organization-specific patterns
   - Configure alert dismissal policies
   |
5. Broad rollout
   - Enable default setup across remaining repositories
   - Configure organization-level security configurations
   - Set branch protection rules requiring code scanning checks
   |
6. Continuous monitoring
   - Review security overview dashboard weekly
   - Track MTTR for code scanning alerts
   - Report metrics to security leadership monthly
```

## Workflow 2: Pull Request Security Gate

```
Developer pushes code to feature branch
           |
   PR is created targeting main
           |
   CodeQL analysis triggers automatically
           |
   Dependency review checks for vulnerable dependencies
           |
   Secret scanning checks for hardcoded credentials
           |
   Results posted as PR check and inline annotations
           |
   [Pass] All checks pass --> PR is eligible for merge
   [Fail] Critical/High findings --> PR is blocked
           |
   Developer reviews findings and applies fixes
           |
   Re-push triggers re-analysis
           |
   Merge after all checks pass and reviewer approval
```

## Workflow 3: Custom CodeQL Query Development

```
1. Identify recurring vulnerability pattern not caught by default queries
   |
2. Set up CodeQL development environment
   - Install CodeQL CLI
   - Clone CodeQL standard library repository
   - Create workspace with target codebase database
   |
3. Author the query in QL language
   - Define source, sink, and taint-tracking configuration
   - Add metadata (@name, @description, @kind, @problem.severity, @security-severity, @precision, @id, @tags)
   |
4. Test the query
   - Create test cases with expected results
   - Run `codeql test run` against test database
   - Validate precision and recall
   |
5. Package the query
   - Create qlpack.yml with version and dependencies
   - Publish to GitHub Container Registry or internal package registry
   |
6. Deploy to scanning workflow
   - Reference the query pack in codeql-action/init step
   - Monitor results for the new query across repositories
```

## Workflow 4: SARIF Integration with External Tools

```
External SAST/DAST tool runs scan
           |
   Tool outputs results in SARIF 2.1.0 format
           |
   GitHub Actions uploads SARIF via codeql-action/upload-sarif
           |
   Results appear in Security tab alongside CodeQL findings
           |
   Unified triage workflow across all scanning tools
           |
   Alert deduplication based on location and rule ID
```

===== END FILE =====


===== FILE: skills/implementing-github-advanced-security-for-code-scanning/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for managing GitHub Advanced Security code scanning with CodeQL."""

import json
import argparse
import subprocess
from datetime import datetime
from collections import Counter


def gh_api(endpoint, method="GET"):
    """Call GitHub API via gh CLI."""
    cmd = ["gh", "api", endpoint, "--method", method]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
    if result.returncode != 0:
        return {"error": result.stderr.strip()}
    try:
        return json.loads(result.stdout) if result.stdout.strip() else {}
    except json.JSONDecodeError:
        return {"raw": result.stdout.strip()}


def get_code_scanning_alerts(owner, repo, state="open"):
    """Get code scanning alerts for a repository."""
    alerts = gh_api(f"/repos/{owner}/{repo}/code-scanning/alerts?state={state}&per_page=100")
    if isinstance(alerts, dict) and "error" in alerts:
        return alerts
    return alerts if isinstance(alerts, list) else []


def get_secret_scanning_alerts(owner, repo, state="open"):
    """Get secret scanning alerts for a repository."""
    alerts = gh_api(f"/repos/{owner}/{repo}/secret-scanning/alerts?state={state}&per_page=100")
    if isinstance(alerts, dict) and "error" in alerts:
        return alerts
    return alerts if isinstance(alerts, list) else []


def get_dependabot_alerts(owner, repo, state="open"):
    """Get Dependabot alerts for a repository."""
    alerts = gh_api(f"/repos/{owner}/{repo}/dependabot/alerts?state={state}&per_page=100")
    if isinstance(alerts, dict) and "error" in alerts:
        return alerts
    return alerts if isinstance(alerts, list) else []


def analyze_code_scanning_alerts(alerts):
    """Analyze code scanning alerts and produce summary."""
    if not isinstance(alerts, list):
        return {"error": "No alerts data"}
    by_severity = Counter()
    by_rule = Counter()
    by_tool = Counter()
    critical_alerts = []
    for alert in alerts:
        rule = alert.get("rule", {})
        severity = rule.get("security_severity_level", rule.get("severity", "unknown"))
        by_severity[severity] += 1
        by_rule[rule.get("id", "unknown")] += 1
        tool = alert.get("tool", {}).get("name", "unknown")
        by_tool[tool] += 1
        if severity in ("critical", "high"):
            critical_alerts.append({
                "number": alert.get("number"),
                "rule": rule.get("id", ""),
                "description": rule.get("description", "")[:120],
                "severity": severity,
                "state": alert.get("state", ""),
                "created_at": alert.get("created_at", ""),
                "html_url": alert.get("html_url", ""),
            })
    return {
        "total_alerts": len(alerts),
        "by_severity": dict(by_severity),
        "by_rule": dict(by_rule.most_common(10)),
        "by_tool": dict(by_tool),
        "critical_and_high": critical_alerts[:20],
    }


def analyze_secret_alerts(alerts):
    """Analyze secret scanning alerts."""
    if not isinstance(alerts, list):
        return {"error": "No alerts data"}
    by_type = Counter()
    for alert in alerts:
        by_type[alert.get("secret_type_display_name", alert.get("secret_type", "unknown"))] += 1
    return {
        "total_secrets": len(alerts),
        "by_type": dict(by_type),
        "alerts": [
            {"number": a.get("number"), "type": a.get("secret_type_display_name", ""),
             "state": a.get("state", ""), "created_at": a.get("created_at", "")}
            for a in alerts[:20]
        ],
    }


def analyze_dependabot_alerts(alerts):
    """Analyze Dependabot vulnerability alerts."""
    if not isinstance(alerts, list):
        return {"error": "No alerts data"}
    by_severity = Counter()
    by_ecosystem = Counter()
    for alert in alerts:
        vuln = alert.get("security_vulnerability", alert.get("security_advisory", {}))
        severity = vuln.get("severity", alert.get("severity", "unknown"))
        by_severity[severity] += 1
        dep = alert.get("dependency", {})
        pkg = dep.get("package", {})
        by_ecosystem[pkg.get("ecosystem", "unknown")] += 1
    return {
        "total_alerts": len(alerts),
        "by_severity": dict(by_severity),
        "by_ecosystem": dict(by_ecosystem),
    }


def generate_codeql_workflow(languages, query_suite="security-extended"):
    """Generate a CodeQL analysis GitHub Actions workflow."""
    lang_list = ", ".join(f"'{l}'" for l in languages)
    return f"""name: CodeQL Analysis
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '30 2 * * 1'
jobs:
  analyze:
    name: Analyze (${{{{ matrix.language }}}})
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
      actions: read
    strategy:
      fail-fast: false
      matrix:
        language: [{lang_list}]
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{{{ matrix.language }}}}
          queries: +{query_suite}
      - uses: github/codeql-action/autobuild@v3
      - uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{{{ matrix.language }}}}"
"""


def full_security_audit(owner, repo):
    """Run full GHAS security audit for a repository."""
    code_alerts = get_code_scanning_alerts(owner, repo)
    secret_alerts = get_secret_scanning_alerts(owner, repo)
    dependabot_alerts = get_dependabot_alerts(owner, repo)
    return {
        "code_scanning": analyze_code_scanning_alerts(code_alerts),
        "secret_scanning": analyze_secret_alerts(secret_alerts),
        "dependabot": analyze_dependabot_alerts(dependabot_alerts),
    }


def main():
    parser = argparse.ArgumentParser(description="GitHub Advanced Security Agent")
    parser.add_argument("--owner", help="Repository owner")
    parser.add_argument("--repo", help="Repository name")
    parser.add_argument("--action", choices=["audit", "code-alerts", "secrets",
                                              "dependabot", "gen-workflow"],
                        default="audit")
    parser.add_argument("--languages", nargs="+", default=["python", "javascript-typescript"])
    parser.add_argument("--output", default="ghas_report.json")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "results": {}}

    if args.action == "audit" and args.owner and args.repo:
        results = full_security_audit(args.owner, args.repo)
        report["results"] = results
        cs = results["code_scanning"]
        print(f"[+] Code scanning: {cs.get('total_alerts', 0)} alerts")
        print(f"[+] Secrets: {results['secret_scanning'].get('total_secrets', 0)}")
        print(f"[+] Dependabot: {results['dependabot'].get('total_alerts', 0)}")

    elif args.action == "code-alerts" and args.owner and args.repo:
        alerts = get_code_scanning_alerts(args.owner, args.repo)
        analysis = analyze_code_scanning_alerts(alerts)
        report["results"]["code_scanning"] = analysis
        print(f"[+] {analysis.get('total_alerts', 0)} code scanning alerts")

    elif args.action == "secrets" and args.owner and args.repo:
        alerts = get_secret_scanning_alerts(args.owner, args.repo)
        analysis = analyze_secret_alerts(alerts)
        report["results"]["secret_scanning"] = analysis
        print(f"[+] {analysis.get('total_secrets', 0)} secret alerts")

    elif args.action == "gen-workflow":
        workflow = generate_codeql_workflow(args.languages)
        report["results"]["workflow"] = workflow
        print("[+] CodeQL workflow generated")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-github-advanced-security-for-code-scanning/scripts/process.py =====
#!/usr/bin/env python3
"""
GitHub Advanced Security Code Scanning Alert Management

Uses the GitHub REST API to query, triage, and report on CodeQL
code scanning alerts across an organization's repositories.
"""

import json
import os
import sys
import urllib.request
import urllib.error
from datetime import datetime, timedelta
from collections import defaultdict


def get_github_headers(token: str) -> dict:
    return {
        "Authorization": f"Bearer {token}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28",
    }


def github_api_get(url: str, token: str) -> list | dict:
    headers = get_github_headers(token)
    results = []
    page = 1
    while True:
        paginated_url = f"{url}{'&' if '?' in url else '?'}per_page=100&page={page}"
        req = urllib.request.Request(paginated_url, headers=headers)
        try:
            with urllib.request.urlopen(req) as resp:
                data = json.loads(resp.read().decode())
                if isinstance(data, list):
                    if not data:
                        break
                    results.extend(data)
                    page += 1
                else:
                    return data
        except urllib.error.HTTPError as e:
            print(f"HTTP {e.code} for {paginated_url}: {e.read().decode()}")
            break
    return results


def list_org_repos(org: str, token: str) -> list:
    url = f"https://api.github.com/orgs/{org}/repos?type=all"
    repos = github_api_get(url, token)
    return [r["full_name"] for r in repos if isinstance(r, dict)]


def get_code_scanning_alerts(repo: str, token: str, state: str = "open") -> list:
    url = f"https://api.github.com/repos/{repo}/code-scanning/alerts?state={state}"
    return github_api_get(url, token)


def categorize_alerts(alerts: list) -> dict:
    categories = defaultdict(lambda: defaultdict(int))
    for alert in alerts:
        rule = alert.get("rule", {})
        severity = rule.get("security_severity_level", "unknown")
        cwe_tags = [t for t in rule.get("tags", []) if t.startswith("cwe-")]
        tool_name = alert.get("tool", {}).get("name", "unknown")
        categories["by_severity"][severity] += 1
        categories["by_tool"][tool_name] += 1
        for cwe in cwe_tags:
            categories["by_cwe"][cwe] += 1
    return dict(categories)


def calculate_mttr(alerts: list) -> dict:
    resolved = [a for a in alerts if a.get("state") == "fixed"]
    if not resolved:
        return {"total_resolved": 0, "avg_mttr_hours": None}

    durations = []
    for alert in resolved:
        created = datetime.fromisoformat(alert["created_at"].replace("Z", "+00:00"))
        fixed = datetime.fromisoformat(
            alert.get("fixed_at", alert.get("dismissed_at", "")).replace("Z", "+00:00")
        )
        durations.append((fixed - created).total_seconds() / 3600)

    return {
        "total_resolved": len(resolved),
        "avg_mttr_hours": round(sum(durations) / len(durations), 1),
        "min_mttr_hours": round(min(durations), 1),
        "max_mttr_hours": round(max(durations), 1),
    }


def generate_org_report(org: str, token: str) -> dict:
    repos = list_org_repos(org, token)
    report = {
        "organization": org,
        "generated_at": datetime.utcnow().isoformat() + "Z",
        "total_repositories": len(repos),
        "repositories_with_scanning": 0,
        "total_open_alerts": 0,
        "severity_summary": defaultdict(int),
        "top_cwes": defaultdict(int),
        "repo_details": [],
    }

    for repo in repos:
        alerts = get_code_scanning_alerts(repo, token, state="open")
        if not alerts:
            continue

        report["repositories_with_scanning"] += 1
        report["total_open_alerts"] += len(alerts)
        categories = categorize_alerts(alerts)

        for sev, count in categories.get("by_severity", {}).items():
            report["severity_summary"][sev] += count
        for cwe, count in categories.get("by_cwe", {}).items():
            report["top_cwes"][cwe] += count

        closed_alerts = get_code_scanning_alerts(repo, token, state="fixed")
        mttr = calculate_mttr(closed_alerts)

        report["repo_details"].append(
            {
                "repository": repo,
                "open_alerts": len(alerts),
                "severity_breakdown": dict(categories.get("by_severity", {})),
                "mttr": mttr,
            }
        )

    report["severity_summary"] = dict(report["severity_summary"])
    top_cwes_sorted = sorted(report["top_cwes"].items(), key=lambda x: x[1], reverse=True)[:10]
    report["top_cwes"] = dict(top_cwes_sorted)
    return report


def print_report(report: dict) -> None:
    print(f"\n{'='*60}")
    print(f"GHAS Code Scanning Report: {report['organization']}")
    print(f"Generated: {report['generated_at']}")
    print(f"{'='*60}")
    print(f"Total repositories: {report['total_repositories']}")
    print(f"Repositories with scanning enabled: {report['repositories_with_scanning']}")
    coverage = (
        report["repositories_with_scanning"] / report["total_repositories"] * 100
        if report["total_repositories"] > 0
        else 0
    )
    print(f"Coverage: {coverage:.1f}%")
    print(f"Total open alerts: {report['total_open_alerts']}")
    print(f"\nSeverity Summary:")
    for sev in ["critical", "high", "medium", "low", "unknown"]:
        count = report["severity_summary"].get(sev, 0)
        if count > 0:
            print(f"  {sev.upper():12s}: {count}")
    print(f"\nTop CWEs:")
    for cwe, count in report.get("top_cwes", {}).items():
        print(f"  {cwe:15s}: {count}")
    print(f"\nRepository Details:")
    for repo in sorted(report["repo_details"], key=lambda r: r["open_alerts"], reverse=True):
        mttr_str = (
            f"{repo['mttr']['avg_mttr_hours']}h" if repo["mttr"]["avg_mttr_hours"] else "N/A"
        )
        print(f"  {repo['repository']:40s} | Open: {repo['open_alerts']:4d} | Avg MTTR: {mttr_str}")


def main():
    token = os.environ.get("GITHUB_TOKEN")
    if not token:
        print("Error: GITHUB_TOKEN environment variable is required")
        sys.exit(1)

    org = os.environ.get("GITHUB_ORG")
    if not org:
        print("Error: GITHUB_ORG environment variable is required")
        sys.exit(1)

    print(f"Fetching code scanning data for organization: {org}")
    report = generate_org_report(org, token)
    print_report(report)

    output_file = f"ghas_report_{org}_{datetime.utcnow().strftime('%Y%m%d')}.json"
    with open(output_file, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\nDetailed report saved to: {output_file}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-infrastructure-as-code-security-scanning/SKILL.md =====
---
name: implementing-infrastructure-as-code-security-scanning
description: 'Implements automated security scanning for Infrastructure as Code using
  Checkov, tfsec, and KICS to detect misconfigurations in Terraform, CloudFormation,
  Kubernetes manifests, and Helm charts, plus policy-based governance and CI/CD
  integration. Use when validating cloud infrastructure before deployment or blocking
  insecure changes (public S3 buckets, open security groups) in pull requests.

  '
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- cicd
- iac-security
- checkov
- tfsec
- terraform
- secure-sdlc
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
- T1078.004
- T1530
---

# Implementing Infrastructure as Code Security Scanning

## When to Use

- When provisioning cloud infrastructure with Terraform, CloudFormation, or Pulumi and needing automated security validation
- When compliance frameworks require evidence of infrastructure configuration review before deployment
- When preventing common cloud misconfigurations like public S3 buckets, open security groups, or unencrypted storage
- When establishing guardrails that block insecure infrastructure changes in pull requests
- When managing multi-cloud environments requiring consistent security policies across AWS, Azure, and GCP

**Do not use** for scanning application source code (use SAST), for monitoring already-deployed infrastructure drift (use cloud security posture management tools), or for container image vulnerability scanning (use Trivy).

## Prerequisites

- Checkov v3.x installed (`pip install checkov`) or tfsec installed
- Terraform, CloudFormation, or Kubernetes IaC files in the repository
- CI/CD pipeline with access to IaC directories
- Bridgecrew API key (optional, for Checkov platform integration)

## Workflow

### Step 1: Run Checkov Against Terraform Files

```bash
# Scan all Terraform files in a directory
checkov -d ./terraform/ --framework terraform --output cli --output json --output-file-path ./results

# Scan specific file
checkov -f main.tf --output json

# Scan Terraform plan (more accurate for dynamic values)
terraform init && terraform plan -out=tfplan
terraform show -json tfplan > tfplan.json
checkov -f tfplan.json --framework terraform_plan

# Scan with specific checks only
checkov -d ./terraform/ --check CKV_AWS_18,CKV_AWS_19,CKV_AWS_20

# Skip specific checks
checkov -d ./terraform/ --skip-check CKV_AWS_145,CKV2_AWS_6
```

### Step 2: Integrate IaC Scanning into GitHub Actions

```yaml
# .github/workflows/iac-security.yml
name: IaC Security Scan

on:
  pull_request:
    paths:
      - 'terraform/**'
      - 'cloudformation/**'
      - 'k8s/**'

jobs:
  checkov:
    name: Checkov IaC Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: terraform/
          framework: terraform
          output_format: cli,sarif
          output_file_path: console,checkov.sarif
          soft_fail: false
          skip_check: CKV_AWS_145

      - name: Upload SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: checkov.sarif
          category: checkov-iac

  tfsec:
    name: tfsec Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tfsec
        uses: aquasecurity/tfsec-action@v1.0.3
        with:
          working_directory: terraform/
          sarif_file: tfsec.sarif
          soft_fail: false

      - name: Upload SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: tfsec.sarif
          category: tfsec
```

### Step 3: Create Custom Checkov Policies

```python
# custom_checks/s3_versioning.py
from checkov.terraform.checks.resource.base_resource_check import BaseResourceCheck
from checkov.common.models.enums import CheckResult, CheckCategories


class S3BucketVersioning(BaseResourceCheck):
    def __init__(self):
        name = "Ensure S3 bucket has versioning enabled"
        id = "CKV_CUSTOM_1"
        supported_resources = ["aws_s3_bucket"]
        categories = [CheckCategories.GENERAL_SECURITY]
        super().__init__(name=name, id=id, categories=categories,
                         supported_resources=supported_resources)

    def scan_resource_conf(self, conf):
        versioning = conf.get("versioning", [{}])
        if isinstance(versioning, list) and len(versioning) > 0:
            if versioning[0].get("enabled", [False])[0]:
                return CheckResult.PASSED
        return CheckResult.FAILED


check = S3BucketVersioning()
```

### Step 4: Configure Baseline and Suppressions

```yaml
# .checkov.yaml
branch: main
compact: true
directory:
  - terraform/
  - cloudformation/
framework:
  - terraform
  - cloudformation
  - kubernetes
output:
  - cli
  - sarif
skip-check:
  - CKV_AWS_145    # S3 default encryption with CMK (using SSE-S3 is acceptable)
  - CKV2_AWS_6     # S3 bucket request logging (handled at CloudTrail level)
soft-fail: false
```

### Step 5: Scan Kubernetes Manifests and Helm Charts

```bash
# Scan Kubernetes manifests
checkov -d ./k8s/ --framework kubernetes

# Scan Helm charts (renders templates first)
checkov -d ./charts/myapp/ --framework helm

# Scan with KICS (Keeping Infrastructure as Code Secure)
docker run -v $(pwd)/k8s:/path checkmarx/kics:latest scan \
  --path /path \
  --output-path /path/results \
  --type Kubernetes \
  --report-formats json,sarif
```

## Key Concepts

| Term | Definition |
|------|------------|
| IaC Scanning | Automated analysis of infrastructure code templates to detect security misconfigurations before deployment |
| Policy as Code | Security policies defined as executable code that can be version-controlled, tested, and enforced automatically |
| CKV Check ID | Checkov's unique identifier for each security check (e.g., CKV_AWS_18 for S3 public access) |
| Terraform Plan Scanning | Scanning the resolved Terraform plan JSON which includes computed values and module expansions |
| Graph-based Scanning | Checkov's ability to analyze relationships between resources, not just individual resource configs |
| Drift Detection | Identifying differences between IaC definitions and actual deployed infrastructure state |
| Custom Policy | Organization-specific security checks authored in Python or YAML to enforce internal standards |

## Tools & Systems

- **Checkov**: Open-source IaC scanner by Bridgecrew with 2500+ built-in policies covering major cloud providers
- **tfsec**: Terraform-focused static analysis tool by Aqua Security with deep HCL understanding
- **KICS**: Open-source IaC scanner by Checkmarx supporting 15+ IaC frameworks
- **Terrascan**: IaC scanner with OPA Rego policy support for custom policy authoring
- **Snyk IaC**: Commercial IaC scanner integrated with the Snyk platform

## Common Scenarios

### Scenario: Preventing Public S3 Buckets in Terraform

**Context**: A development team repeatedly creates S3 buckets without proper access controls. A recent incident exposed customer data through a public bucket.

**Approach**:
1. Enable Checkov in the CI/CD pipeline for all Terraform changes
2. Enforce CKV_AWS_18 (no public read ACL), CKV_AWS_19 (encryption), CKV_AWS_20 (no public access block disabled)
3. Create a custom policy requiring the `aws_s3_bucket_public_access_block` resource for every S3 bucket
4. Set `soft_fail: false` to block PR merges when S3 security checks fail
5. Provide Terraform modules with security defaults that teams can reuse

**Pitfalls**: Scanning only `.tf` files misses dynamically computed values. Use Terraform plan scanning for higher accuracy. Checkov's resource-relationship checks (CKV2 prefix) require graph analysis mode.

## Output Format

```
IaC Security Scan Report
==========================
Framework: Terraform
Directory: terraform/
Scan Date: 2026-02-23

Checkov Results:
  Passed: 187
  Failed: 12
  Skipped: 3
  Unknown: 0

FAILED CHECKS:
  CKV_AWS_18  [HIGH]   S3 Bucket has public read ACL
              Resource: aws_s3_bucket.data_lake
              File:     terraform/storage.tf:15-28

  CKV_AWS_24  [HIGH]   CloudWatch log group not encrypted
              Resource: aws_cloudwatch_log_group.app
              File:     terraform/monitoring.tf:3-8

  CKV_AWS_79  [MEDIUM] Instance metadata service v1 enabled
              Resource: aws_instance.web
              File:     terraform/compute.tf:12-30

QUALITY GATE: FAILED (2 HIGH severity findings)
```

===== END FILE =====


===== FILE: skills/implementing-infrastructure-as-code-security-scanning/LICENSE =====

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


===== FILE: skills/implementing-infrastructure-as-code-security-scanning/assets/template.md =====
# IaC Security Scanning Templates

## Checkov Configuration File

```yaml
# .checkov.yaml
branch: main
compact: true
directory:
  - terraform/
  - cloudformation/
  - k8s/
framework:
  - terraform
  - cloudformation
  - kubernetes
output:
  - cli
  - sarif
skip-check:
  - CKV_AWS_145   # CMK encryption for S3 (SSE-S3 acceptable)
  - CKV2_AWS_6    # S3 request logging (CloudTrail covers this)
soft-fail: false
```

## GitHub Actions Pipeline

```yaml
# .github/workflows/iac-security.yml
name: IaC Security

on:
  pull_request:
    paths: ['terraform/**', 'k8s/**', 'cloudformation/**']

jobs:
  checkov:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: bridgecrewio/checkov-action@v12
        with:
          directory: terraform/
          framework: terraform
          output_format: cli,sarif
          output_file_path: console,checkov.sarif
          soft_fail: false
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: checkov.sarif
```

## Secure Terraform Module Template

```hcl
# modules/secure-s3-bucket/main.tf
resource "aws_s3_bucket" "this" {
  bucket = var.bucket_name
  tags   = var.tags
}

resource "aws_s3_bucket_versioning" "this" {
  bucket = aws_s3_bucket.this.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "this" {
  bucket = aws_s3_bucket.this.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
      kms_master_key_id = var.kms_key_id
    }
    bucket_key_enabled = true
  }
}

resource "aws_s3_bucket_public_access_block" "this" {
  bucket = aws_s3_bucket.this.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_logging" "this" {
  bucket        = aws_s3_bucket.this.id
  target_bucket = var.logging_bucket
  target_prefix = "s3-access-logs/${var.bucket_name}/"
}
```

===== END FILE =====


===== FILE: skills/implementing-infrastructure-as-code-security-scanning/references/api-reference.md =====
# API Reference: Implementing Infrastructure as Code Security Scanning

## Checkov CLI

```bash
# Scan Terraform directory
checkov -d /path/to/tf --framework terraform --output json
# Scan specific file
checkov -f main.tf
# Scan CloudFormation
checkov -d . --framework cloudformation
# Scan Kubernetes manifests
checkov -d . --framework kubernetes
# Skip specific checks
checkov -d . --skip-check CKV_AWS_18,CKV_AWS_21
```

## tfsec CLI

```bash
# Scan directory
tfsec /path/to/tf --format json
# Exclude specific rules
tfsec . --exclude aws-s3-enable-bucket-logging
# Minimum severity
tfsec . --minimum-severity HIGH
```

## Common IaC Security Checks

| Check ID | Description | Severity |
|----------|-------------|----------|
| CKV_AWS_18 | S3 bucket logging | MEDIUM |
| CKV_AWS_19 | S3 bucket encryption | HIGH |
| CKV_AWS_23 | Security group open to 0.0.0.0/0 | HIGH |
| CKV_AWS_41 | RDS encryption | HIGH |
| CKV_AWS_145 | KMS key rotation | MEDIUM |
| CKV_K8S_1 | Pod privileged container | CRITICAL |

## GitHub Actions Integration

```yaml
- uses: bridgecrewio/checkov-action@master
  with:
    directory: .
    framework: terraform
    output_format: sarif
    soft_fail: false
```

### References

- Checkov: https://www.checkov.io/
- tfsec: https://aquasecurity.github.io/tfsec/
- KICS: https://kics.io/
- Bridgecrew: https://www.bridgecrew.io/

===== END FILE =====


===== FILE: skills/implementing-infrastructure-as-code-security-scanning/references/standards.md =====
# Standards Reference: IaC Security Scanning

## CIS Cloud Benchmarks

### CIS AWS Foundations Benchmark v3.0
- Maps directly to Checkov CKV_AWS_* checks
- Covers IAM, logging, monitoring, networking, and storage security
- Automated scanning validates 100+ benchmark controls

### CIS Azure Foundations Benchmark v2.1
- Maps to Checkov CKV_AZURE_* checks
- Covers identity, security center, storage, database, and network controls

### CIS GCP Foundations Benchmark v2.0
- Maps to Checkov CKV_GCP_* checks
- Covers IAM, logging, networking, VM, storage, and database controls

## NIST SP 800-53 Mapping

| NIST Control | IaC Check | Checkov ID |
|-------------|-----------|------------|
| AC-3 Access Enforcement | S3 bucket public access | CKV_AWS_18, CKV_AWS_20 |
| AU-2 Audit Events | CloudTrail enabled | CKV_AWS_35 |
| SC-8 Transmission Confidentiality | HTTPS/TLS enforcement | CKV_AWS_2 |
| SC-28 Protection at Rest | Encryption at rest | CKV_AWS_19, CKV_AWS_17 |
| SI-4 System Monitoring | CloudWatch/logging | CKV_AWS_24, CKV_AWS_66 |

## OWASP SAMM - Secure Architecture

### Security Architecture Level 2
- Validate infrastructure configurations against security standards before deployment
- Use automated tools to enforce architecture security requirements

### Security Architecture Level 3
- Custom policies encode organization-specific architecture requirements
- Continuous validation prevents configuration drift from approved patterns

## NIST SSDF (SP 800-218)

### PO.1: Define Security Requirements
- IaC security policies translate security requirements into enforceable checks
- Custom policies capture organization-specific requirements

### PW.5: Configure Software Securely
- PW.5.1: Configure software to have secure settings by default
- IaC scanning enforces secure defaults in infrastructure provisioning

===== END FILE =====


===== FILE: skills/implementing-infrastructure-as-code-security-scanning/references/workflows.md =====
# Workflow Reference: IaC Security Scanning

## IaC Scanning Pipeline

```
Terraform/IaC Code Change
       │
       ▼
┌──────────────────┐
│ PR Created       │
└──────┬───────────┘
       │
       ├──────────────────────┐
       ▼                      ▼
┌──────────────┐    ┌──────────────┐
│ Checkov      │    │ tfsec        │
│ (2500+ rules)│    │ (Terraform)  │
└──────┬───────┘    └──────┬───────┘
       │                    │
       └──────────┬─────────┘
                  ▼
       ┌──────────────────┐
       │ SARIF Upload     │
       │ to GitHub        │
       └──────┬───────────┘
              │
              ▼
       ┌──────────────────┐
       │ Quality Gate     │
       │ (Block on HIGH+) │
       └──────┬───────────┘
              │
    ┌─────────┴──────────┐
    ▼                    ▼
 PASS                  FAIL
 terraform apply      Block merge
 permitted            + Fix required
```

## Checkov Command Reference

| Command | Purpose |
|---------|---------|
| `checkov -d ./terraform/` | Scan directory |
| `checkov -f main.tf` | Scan single file |
| `checkov -f tfplan.json --framework terraform_plan` | Scan Terraform plan |
| `checkov --list` | List all available checks |
| `checkov -d . --check CKV_AWS_18` | Run specific check |
| `checkov -d . --skip-check CKV_AWS_145` | Skip specific check |
| `checkov -d . --bc-api-key KEY` | Upload to Bridgecrew |
| `checkov -d . --create-baseline` | Create baseline file |
| `checkov -d . --baseline BASELINE` | Scan against baseline |
| `checkov -d . --external-checks-dir ./custom/` | Use custom checks |
| `checkov -d . --compact` | Compact output |
| `checkov -d . --output sarif` | SARIF format output |

## Common Misconfigurations by Cloud Provider

### AWS Top 10 IaC Misconfigurations
1. S3 bucket public access enabled (CKV_AWS_18, CKV_AWS_20)
2. Security group with open ingress 0.0.0.0/0 (CKV_AWS_23)
3. RDS instance not encrypted (CKV_AWS_16)
4. CloudTrail not enabled (CKV_AWS_35)
5. EBS volume not encrypted (CKV_AWS_3)
6. IAM policy with wildcard actions (CKV_AWS_1)
7. ALB not using HTTPS (CKV_AWS_2)
8. CloudWatch logs not encrypted (CKV_AWS_24)
9. IMDSv2 not required (CKV_AWS_79)
10. VPC flow logs not enabled (CKV_AWS_9)

### Kubernetes Top Misconfigurations
1. Container running as root (CKV_K8S_6)
2. Privileged container (CKV_K8S_16)
3. No resource limits (CKV_K8S_11, CKV_K8S_13)
4. No readiness/liveness probes (CKV_K8S_9)
5. hostNetwork enabled (CKV_K8S_19)

===== END FILE =====


===== FILE: skills/implementing-infrastructure-as-code-security-scanning/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for scanning Infrastructure as Code templates for security misconfigurations."""

import json
import argparse
import subprocess
from datetime import datetime
from collections import Counter
from pathlib import Path


def run_checkov(target_path, framework=None):
    """Run Checkov IaC security scanner."""
    cmd = ["checkov", "-d", target_path, "--output", "json", "--quiet"]
    if framework:
        cmd.extend(["--framework", framework])
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    try:
        return json.loads(result.stdout) if result.stdout.strip() else {"error": result.stderr}
    except json.JSONDecodeError:
        return {"raw": result.stdout[:2000], "error": result.stderr}


def run_tfsec(target_path):
    """Run tfsec Terraform security scanner."""
    cmd = ["tfsec", target_path, "--format", "json"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    try:
        return json.loads(result.stdout) if result.stdout.strip() else {"error": result.stderr}
    except json.JSONDecodeError:
        return {"raw": result.stdout[:2000]}


def analyze_checkov_results(results):
    """Analyze Checkov scan results and categorize findings."""
    if isinstance(results, list):
        all_checks = results
    elif isinstance(results, dict):
        all_checks = results.get("results", {}).get("failed_checks", [])
    else:
        return {"error": "Invalid results format"}

    by_severity = Counter()
    by_resource_type = Counter()
    by_check = Counter()
    findings = []

    for check in all_checks:
        severity = check.get("severity", check.get("check_result", {}).get("severity", "MEDIUM"))
        by_severity[severity] += 1
        resource = check.get("resource", "unknown")
        by_resource_type[resource.split(".")[0]] += 1
        by_check[check.get("check_id", "unknown")] += 1
        if severity in ("CRITICAL", "HIGH"):
            findings.append({
                "check_id": check.get("check_id", ""),
                "check_name": check.get("check", check.get("name", "")),
                "resource": resource,
                "file": check.get("file_path", check.get("repo_file_path", "")),
                "line": check.get("file_line_range", []),
                "severity": severity,
                "guideline": check.get("guideline", ""),
            })

    return {
        "total_failures": len(all_checks),
        "by_severity": dict(by_severity),
        "by_resource_type": dict(by_resource_type.most_common(10)),
        "top_checks": dict(by_check.most_common(10)),
        "critical_findings": findings[:30],
    }


def scan_terraform_files(dir_path):
    """Scan Terraform files for common security misconfigurations."""
    findings = []
    tf_files = list(Path(dir_path).rglob("*.tf"))

    risky_patterns = {
        '"0.0.0.0/0"': {"issue": "Open CIDR block", "severity": "HIGH"},
        "cidr_blocks = [": {"issue": "Check CIDR block restrictions", "severity": "MEDIUM"},
        "publicly_accessible": {"issue": "Public accessibility setting", "severity": "HIGH"},
        "encrypted = false": {"issue": "Encryption disabled", "severity": "CRITICAL"},
        "enable_logging = false": {"issue": "Logging disabled", "severity": "HIGH"},
        "versioning {": {"issue": "Check versioning enabled", "severity": "MEDIUM"},
        'protocol = "-1"': {"issue": "All protocols allowed", "severity": "HIGH"},
        "from_port = 0": {"issue": "All ports open", "severity": "HIGH"},
    }

    for tf_file in tf_files:
        try:
            content = tf_file.read_text(encoding="utf-8", errors="ignore")
            for pattern, info in risky_patterns.items():
                if pattern in content:
                    lines = [i + 1 for i, line in enumerate(content.split("\n"))
                             if pattern in line]
                    for line in lines[:5]:
                        findings.append({
                            "file": str(tf_file),
                            "line": line,
                            "pattern": pattern,
                            "issue": info["issue"],
                            "severity": info["severity"],
                        })
        except (OSError, PermissionError):
            continue

    return findings


def generate_ci_config(scanner="checkov", framework="terraform"):
    """Generate CI/CD pipeline config for IaC scanning."""
    configs = {
        "github_actions": f"""name: IaC Security Scan
on: [push, pull_request]
jobs:
  iac-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run {scanner}
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: {framework}
          soft_fail: false
          output_format: sarif
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: results.sarif
""",
        "gitlab_ci": f"""iac-scan:
  image: bridgecrew/checkov:latest
  script:
    - checkov -d . --framework {framework} --output junitxml > checkov.xml
  artifacts:
    reports:
      junit: checkov.xml
""",
    }
    return configs


def main():
    parser = argparse.ArgumentParser(description="IaC Security Scanning Agent")
    parser.add_argument("--scan-dir", help="Directory to scan")
    parser.add_argument("--framework", choices=["terraform", "cloudformation",
                                                  "kubernetes", "helm", "all"])
    parser.add_argument("--scanner", choices=["checkov", "tfsec", "builtin"], default="builtin")
    parser.add_argument("--gen-ci", action="store_true", help="Generate CI config")
    parser.add_argument("--output", default="iac_security_report.json")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "results": {}}

    if args.scan_dir:
        if args.scanner == "checkov":
            raw = run_checkov(args.scan_dir, args.framework)
            analysis = analyze_checkov_results(raw)
            report["results"]["checkov"] = analysis
            print(f"[+] Checkov: {analysis.get('total_failures', 0)} failures")
        elif args.scanner == "tfsec":
            raw = run_tfsec(args.scan_dir)
            report["results"]["tfsec"] = raw
            print("[+] tfsec scan complete")
        else:
            findings = scan_terraform_files(args.scan_dir)
            report["results"]["builtin_scan"] = findings
            print(f"[+] Built-in scan: {len(findings)} findings")

    if args.gen_ci:
        configs = generate_ci_config(args.scanner or "checkov", args.framework or "terraform")
        report["results"]["ci_configs"] = configs
        print("[+] CI configs generated")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-infrastructure-as-code-security-scanning/scripts/process.py =====
#!/usr/bin/env python3
"""
IaC Security Scanning Pipeline Script

Runs Checkov and/or tfsec against IaC directories, aggregates findings,
evaluates quality gates, and generates reports.

Usage:
    python process.py --iac-dir ./terraform --framework terraform
    python process.py --iac-dir ./k8s --framework kubernetes --output report.json
"""

import argparse
import json
import os
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime, timezone
from typing import Optional


SEVERITY_MAP = {"CRITICAL": 0, "HIGH": 1, "MEDIUM": 2, "LOW": 3}


@dataclass
class IaCFinding:
    check_id: str
    check_name: str
    severity: str
    resource: str
    file_path: str
    guideline: str
    framework: str
    tool: str
    file_line_range: list = field(default_factory=list)


def run_checkov(iac_dir: str, framework: str = "terraform",
                skip_checks: Optional[list] = None,
                custom_checks_dir: Optional[str] = None,
                baseline: Optional[str] = None) -> dict:
    """Run Checkov scan and return JSON results."""
    cmd = [
        "checkov", "-d", iac_dir,
        "--framework", framework,
        "--output", "json",
        "--compact",
        "--quiet"
    ]

    if skip_checks:
        cmd.extend(["--skip-check", ",".join(skip_checks)])
    if custom_checks_dir:
        cmd.extend(["--external-checks-dir", custom_checks_dir])
    if baseline:
        cmd.extend(["--baseline", baseline])

    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        if proc.stdout:
            try:
                return json.loads(proc.stdout)
            except json.JSONDecodeError:
                return {"error": "Failed to parse Checkov JSON output"}
        return {"error": proc.stderr[:500] if proc.stderr else "No output from Checkov"}
    except subprocess.TimeoutExpired:
        return {"error": "Checkov scan timed out"}
    except FileNotFoundError:
        return {"error": "checkov not found. Install with: pip install checkov"}


def run_tfsec(iac_dir: str) -> dict:
    """Run tfsec scan and return JSON results."""
    cmd = ["tfsec", iac_dir, "--format", "json", "--no-color"]

    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        if proc.stdout:
            try:
                return json.loads(proc.stdout)
            except json.JSONDecodeError:
                return {"error": "Failed to parse tfsec JSON output"}
        return {"error": proc.stderr[:500] if proc.stderr else "No output from tfsec"}
    except FileNotFoundError:
        return {"error": "tfsec not found. Install from https://aquasecurity.github.io/tfsec/"}


def parse_checkov_results(checkov_json: dict) -> tuple:
    """Parse Checkov JSON output into findings."""
    findings = []
    passed_count = 0
    skipped_count = 0

    results_list = checkov_json if isinstance(checkov_json, list) else [checkov_json]

    for result_block in results_list:
        for check_type in result_block.get("results", {}):
            if check_type == "passed_checks":
                passed_count += len(result_block["results"].get("passed_checks", []))
            elif check_type == "skipped_checks":
                skipped_count += len(result_block["results"].get("skipped_checks", []))
            elif check_type == "failed_checks":
                for check in result_block["results"].get("failed_checks", []):
                    severity = check.get("severity", "MEDIUM")
                    if not severity or severity == "UNKNOWN":
                        severity = "MEDIUM"

                    findings.append(IaCFinding(
                        check_id=check.get("check_id", ""),
                        check_name=check.get("check_result", {}).get("name", check.get("name", "")),
                        severity=severity.upper(),
                        resource=check.get("resource", ""),
                        file_path=check.get("file_path", ""),
                        guideline=check.get("guideline", ""),
                        framework=check.get("check_type", "terraform"),
                        tool="checkov",
                        file_line_range=check.get("file_line_range", [])
                    ))

    return findings, passed_count, skipped_count


def parse_tfsec_results(tfsec_json: dict) -> list:
    """Parse tfsec JSON output into findings."""
    findings = []
    for result in tfsec_json.get("results", []):
        findings.append(IaCFinding(
            check_id=result.get("rule_id", result.get("long_id", "")),
            check_name=result.get("rule_description", ""),
            severity=result.get("severity", "MEDIUM").upper(),
            resource=result.get("resource", ""),
            file_path=result.get("location", {}).get("filename", ""),
            guideline=result.get("resolution", ""),
            framework="terraform",
            tool="tfsec",
            file_line_range=[
                result.get("location", {}).get("start_line", 0),
                result.get("location", {}).get("end_line", 0)
            ]
        ))
    return findings


def evaluate_quality_gate(findings: list, threshold: str) -> dict:
    """Evaluate quality gate based on finding severities."""
    threshold_level = SEVERITY_MAP.get(threshold.upper(), 1)
    blocking = [f for f in findings if SEVERITY_MAP.get(f.severity, 3) <= threshold_level]

    severity_counts = {}
    for f in findings:
        severity_counts[f.severity] = severity_counts.get(f.severity, 0) + 1

    return {
        "passed": len(blocking) == 0,
        "threshold": threshold.upper(),
        "total_findings": len(findings),
        "blocking_count": len(blocking),
        "severity_counts": severity_counts,
        "blocking_details": [
            {
                "check_id": f.check_id,
                "name": f.check_name,
                "severity": f.severity,
                "resource": f.resource,
                "file": f.file_path,
                "lines": f.file_line_range
            }
            for f in blocking[:25]
        ]
    }


def main():
    parser = argparse.ArgumentParser(description="IaC Security Scanning Pipeline")
    parser.add_argument("--iac-dir", required=True, help="Directory containing IaC files")
    parser.add_argument("--framework", default="terraform",
                        choices=["terraform", "cloudformation", "kubernetes", "helm", "all"])
    parser.add_argument("--output", default="iac-security-report.json")
    parser.add_argument("--severity-threshold", default="high",
                        choices=["critical", "high", "medium", "low"])
    parser.add_argument("--fail-on-findings", action="store_true")
    parser.add_argument("--skip-checks", nargs="*", help="Check IDs to skip")
    parser.add_argument("--custom-checks-dir", default=None)
    parser.add_argument("--run-tfsec", action="store_true", help="Also run tfsec")
    args = parser.parse_args()

    iac_dir = os.path.abspath(args.iac_dir)
    all_findings = []

    print(f"[*] Scanning IaC: {iac_dir} (framework: {args.framework})")

    checkov_json = run_checkov(iac_dir, args.framework, args.skip_checks, args.custom_checks_dir)
    if "error" in checkov_json:
        print(f"[WARN] Checkov: {checkov_json['error']}")
    else:
        findings, passed, skipped = parse_checkov_results(checkov_json)
        all_findings.extend(findings)
        print(f"    Checkov: {len(findings)} failed, {passed} passed, {skipped} skipped")

    if args.run_tfsec and args.framework in ("terraform", "all"):
        tfsec_json = run_tfsec(iac_dir)
        if "error" not in tfsec_json:
            tfsec_findings = parse_tfsec_results(tfsec_json)
            all_findings.extend(tfsec_findings)
            print(f"    tfsec: {len(tfsec_findings)} findings")

    quality_gate = evaluate_quality_gate(all_findings, args.severity_threshold)

    report = {
        "metadata": {
            "directory": iac_dir,
            "framework": args.framework,
            "scan_date": datetime.now(timezone.utc).isoformat()
        },
        "quality_gate": quality_gate,
        "findings": [
            {
                "check_id": f.check_id, "name": f.check_name,
                "severity": f.severity, "resource": f.resource,
                "file": f.file_path, "lines": f.file_line_range,
                "guideline": f.guideline, "tool": f.tool
            }
            for f in sorted(all_findings, key=lambda x: SEVERITY_MAP.get(x.severity, 3))
        ]
    }

    output_path = os.path.abspath(args.output)
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[*] Report: {output_path}")

    if quality_gate["passed"]:
        print(f"\n[PASS] Quality gate passed. {quality_gate['total_findings']} findings, none blocking.")
    else:
        print(f"\n[FAIL] {quality_gate['blocking_count']} blocking findings:")
        for d in quality_gate["blocking_details"][:10]:
            print(f"  [{d['severity']}] {d['check_id']}: {d['resource']} ({d['file']})")

    if args.fail_on_findings and not quality_gate["passed"]:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-policy-as-code-with-open-policy-agent/SKILL.md =====
---
name: implementing-policy-as-code-with-open-policy-agent
description: 'Implements policy-as-code enforcement with Open Policy Agent (OPA)
  and Gatekeeper for Kubernetes and CI/CD pipelines, covering writing Rego policies,
  deploying OPA Gatekeeper as a Kubernetes admission controller, testing policies
  in development, and integrating policy evaluation into deployment pipelines. Use
  when writing Rego policies, deploying Gatekeeper admission control, or gating
  CI/CD pipelines with policy-as-code checks.

  '
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- cicd
- opa
- gatekeeper
- policy-as-code
- kubernetes
- secure-sdlc
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- GOVERN-1.1
- MEASURE-2.7
- MANAGE-3.1
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
- T1610
- T1611
---

# Implementing Policy as Code with Open Policy Agent

## When to Use

- When enforcing organizational security policies across Kubernetes clusters programmatically
- When requiring admission control that blocks non-compliant resources from being created
- When implementing policy governance that can be version-controlled, tested, and audited
- When standardizing security rules across multiple clusters and environments
- When needing a flexible policy engine that extends beyond Kubernetes to APIs and CI/CD

**Do not use** for vulnerability scanning (use Trivy/Checkov), for runtime threat detection (use Falco), or for network policy enforcement (use Kubernetes NetworkPolicy or Calico).

## Prerequisites

- Kubernetes cluster with admin access for Gatekeeper installation
- Helm for Gatekeeper deployment
- OPA CLI or conftest for local policy testing
- Rego knowledge for policy authoring

## Workflow

### Step 1: Install OPA Gatekeeper

```bash
# Install Gatekeeper via Helm
helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
helm install gatekeeper gatekeeper/gatekeeper \
  --namespace gatekeeper-system --create-namespace \
  --set replicas=3 \
  --set audit.replicas=1 \
  --set audit.writeToRAMDisk=true
```

### Step 2: Create Constraint Templates

```yaml
# templates/k8s-required-labels.yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("Missing required labels: %v", [missing])
        }

---
# templates/k8s-container-limits.yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8scontainerlimits
spec:
  crd:
    spec:
      names:
        kind: K8sContainerLimits
      validation:
        openAPIV3Schema:
          type: object
          properties:
            cpu:
              type: string
            memory:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8scontainerlimits
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.cpu
          msg := sprintf("Container %v has no CPU limit", [container.name])
        }
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.memory
          msg := sprintf("Container %v has no memory limit", [container.name])
        }

---
# templates/k8s-block-privileged.yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sblockprivileged
spec:
  crd:
    spec:
      names:
        kind: K8sBlockPrivileged
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sblockprivileged
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          container.securityContext.privileged == true
          msg := sprintf("Privileged container not allowed: %v", [container.name])
        }
        violation[{"msg": msg}] {
          container := input.review.object.spec.initContainers[_]
          container.securityContext.privileged == true
          msg := sprintf("Privileged init container not allowed: %v", [container.name])
        }
```

### Step 3: Apply Constraints

```yaml
# constraints/require-labels.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-team-labels
spec:
  enforcementAction: deny
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
      - apiGroups: ["apps"]
        kinds: ["Deployment", "StatefulSet"]
    excludedNamespaces:
      - kube-system
      - gatekeeper-system
  parameters:
    labels:
      - "team"
      - "environment"
      - "cost-center"

---
# constraints/block-privileged.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sBlockPrivileged
metadata:
  name: block-privileged-containers
spec:
  enforcementAction: deny
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
      - apiGroups: ["apps"]
        kinds: ["Deployment", "DaemonSet", "StatefulSet"]
    excludedNamespaces:
      - kube-system
```

### Step 4: Test Policies with conftest

```bash
# Install conftest
brew install conftest

# Test Kubernetes manifests against OPA policies locally
conftest test deployment.yaml --policy policies/ --output json

# Test Terraform against OPA policies
conftest test terraform/main.tf --policy policies/terraform/ --parser hcl2

# Test Dockerfiles
conftest test Dockerfile --policy policies/docker/
```

```rego
# policies/kubernetes/deny_latest_tag.rego
package kubernetes

deny[msg] {
  input.kind == "Deployment"
  container := input.spec.template.spec.containers[_]
  endswith(container.image, ":latest")
  msg := sprintf("Container %v uses :latest tag. Pin to specific version.", [container.name])
}

deny[msg] {
  input.kind == "Deployment"
  container := input.spec.template.spec.containers[_]
  not contains(container.image, ":")
  msg := sprintf("Container %v has no tag. Pin to specific version.", [container.name])
}
```

### Step 5: Integrate Policy Testing in CI/CD

```yaml
# .github/workflows/policy-test.yml
name: Policy Validation

on:
  pull_request:
    paths: ['k8s/**', 'terraform/**', 'policies/**']

jobs:
  conftest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install conftest
        run: |
          wget -q https://github.com/open-policy-agent/conftest/releases/download/v0.50.0/conftest_0.50.0_Linux_x86_64.tar.gz
          tar xzf conftest_0.50.0_Linux_x86_64.tar.gz
          sudo mv conftest /usr/local/bin/
      - name: Test K8s manifests
        run: conftest test k8s/**/*.yaml --policy policies/kubernetes/ --output json
      - name: Test Terraform
        run: conftest test terraform/*.tf --policy policies/terraform/ --parser hcl2
```

## Key Concepts

| Term | Definition |
|------|------------|
| OPA | Open Policy Agent — general-purpose policy engine using Rego language for policy decisions |
| Rego | OPA's declarative query language for writing policy rules |
| Gatekeeper | Kubernetes-native OPA integration implementing admission control via ConstraintTemplates |
| ConstraintTemplate | CRD defining the Rego policy logic and parameters schema for a class of constraints |
| Constraint | Instance of a ConstraintTemplate with specific parameters and scope (which resources to check) |
| Admission Controller | Kubernetes component that intercepts API requests before persistence and can allow or deny them |
| conftest | CLI tool for testing structured data (YAML, JSON, HCL) against OPA policies |

## Tools & Systems

- **Open Policy Agent (OPA)**: General-purpose policy engine for unified policy enforcement
- **Gatekeeper**: Kubernetes admission controller built on OPA with CRD-based configuration
- **conftest**: Testing framework for OPA policies against configuration files
- **Kyverno**: Alternative Kubernetes policy engine using YAML-based policies (no Rego required)
- **Styra DAS**: Commercial OPA management platform with policy authoring, testing, and distribution

## Common Scenarios

### Scenario: Enforcing Container Security Standards Across Clusters

**Context**: Multiple development teams deploy to shared Kubernetes clusters. Some teams run privileged containers and images without resource limits, causing security and stability issues.

**Approach**:
1. Deploy Gatekeeper on all clusters via GitOps (Helm chart in a FluxCD repository)
2. Create ConstraintTemplates for: no privileged containers, required resource limits, required labels, no latest tag
3. Start with `enforcementAction: warn` to identify violations without blocking deployments
4. Notify teams of violations and provide a 2-week remediation window
5. Switch to `enforcementAction: deny` after the remediation period
6. Add `excludedNamespaces` for kube-system and monitoring namespaces

**Pitfalls**: Deploying Gatekeeper with deny mode immediately can break existing workloads. Always start with warn mode. Overly restrictive policies without exemptions for system namespaces can prevent cluster components from functioning.

## Output Format

```
OPA Policy Evaluation Report
==============================
Cluster: production-east
Date: 2026-02-23
Gatekeeper Version: 3.16.0

CONSTRAINT SUMMARY:
  K8sRequiredLabels:        12 violations (warn)
  K8sBlockPrivileged:        0 violations (deny)
  K8sContainerLimits:        8 violations (deny)
  K8sBlockLatestTag:         3 violations (deny)

BLOCKED DEPLOYMENTS (deny):
  [K8sContainerLimits] deployment/api-server in ns/payments
    - Container 'api' has no memory limit
  [K8sBlockLatestTag] deployment/frontend in ns/web
    - Container 'nginx' uses :latest tag

AUDIT VIOLATIONS (warn):
  [K8sRequiredLabels] namespace/staging
    - Missing labels: {cost-center}
```

===== END FILE =====


===== FILE: skills/implementing-policy-as-code-with-open-policy-agent/LICENSE =====

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


===== FILE: skills/implementing-policy-as-code-with-open-policy-agent/assets/template.md =====
# OPA Policy as Code Templates

## Gatekeeper ConstraintTemplate Library

```yaml
# Block containers running as root
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sblockrootuser
spec:
  crd:
    spec:
      names:
        kind: K8sBlockRootUser
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sblockrootuser
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          container.securityContext.runAsUser == 0
          msg := sprintf("Container %v runs as root (UID 0)", [container.name])
        }
        violation[{"msg": msg}] {
          input.review.object.spec.securityContext.runAsUser == 0
          msg := "Pod runs as root (UID 0)"
        }
```

## conftest Policy for CI/CD

```rego
# policies/kubernetes/security.rego
package kubernetes

deny[msg] {
  input.kind == "Deployment"
  container := input.spec.template.spec.containers[_]
  container.securityContext.privileged == true
  msg := sprintf("Privileged container '%v' not allowed", [container.name])
}

deny[msg] {
  input.kind == "Deployment"
  container := input.spec.template.spec.containers[_]
  not container.resources.limits
  msg := sprintf("Container '%v' missing resource limits", [container.name])
}

deny[msg] {
  input.kind == "Deployment"
  container := input.spec.template.spec.containers[_]
  endswith(container.image, ":latest")
  msg := sprintf("Container '%v' uses :latest tag", [container.name])
}
```

## GitHub Actions Integration

```yaml
name: Policy Check
on:
  pull_request:
    paths: ['k8s/**']
jobs:
  conftest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: open-policy-agent/conftest-action@v1
        with:
          files: k8s/
          policy: policies/kubernetes/
```

===== END FILE =====


===== FILE: skills/implementing-policy-as-code-with-open-policy-agent/references/api-reference.md =====
# API Reference: Open Policy Agent (OPA) Policy-as-Code

## Libraries Used

| Library | Purpose |
|---------|---------|
| `requests` | HTTP client for OPA REST API |
| `json` | Parse OPA decision responses |
| `subprocess` | Run `opa eval` and `opa test` CLI commands |
| `yaml` | Parse Kubernetes admission review objects |

## Installation

```bash
# OPA binary
curl -L -o opa https://openpolicyagent.org/downloads/latest/opa_linux_amd64_static
chmod 755 opa && sudo mv opa /usr/local/bin/

# Python dependencies
pip install requests pyyaml
```

## OPA REST API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| PUT | `/v1/policies/{id}` | Create or update a policy module |
| GET | `/v1/policies/{id}` | Retrieve a policy module |
| DELETE | `/v1/policies/{id}` | Delete a policy module |
| GET | `/v1/policies` | List all policy modules |
| PUT | `/v1/data/{path}` | Create or overwrite a document |
| GET | `/v1/data/{path}` | Evaluate a rule or retrieve data |
| POST | `/v1/data/{path}` | Evaluate a rule with input |
| PATCH | `/v1/data/{path}` | Patch a data document |
| POST | `/v1/query` | Execute ad-hoc Rego query |
| POST | `/v1/compile` | Partially evaluate a query |
| GET | `/health` | Health check (liveness) |
| GET | `/health?bundles` | Health check including bundle status |

## Core Operations

### Upload a Rego Policy
```python
import requests
import os

OPA_URL = os.environ.get("OPA_URL", "http://localhost:8181")

policy_rego = """
package authz

default allow := false

allow if {
    input.user.role == "admin"
}

allow if {
    input.user.role == "editor"
    input.action == "read"
}
"""

resp = requests.put(
    f"{OPA_URL}/v1/policies/authz",
    data=policy_rego,
    headers={"Content-Type": "text/plain"},
    timeout=10,
)
resp.raise_for_status()
```

### Evaluate a Policy Decision
```python
decision_input = {
    "input": {
        "user": {"role": "editor", "name": "alice"},
        "action": "read",
        "resource": "/api/reports",
    }
}

resp = requests.post(
    f"{OPA_URL}/v1/data/authz/allow",
    json=decision_input,
    timeout=10,
)
result = resp.json()
allowed = result.get("result", False)  # True
```

### Upload Data Documents
```python
role_permissions = {
    "admin": ["read", "write", "delete", "admin"],
    "editor": ["read", "write"],
    "viewer": ["read"],
}

resp = requests.put(
    f"{OPA_URL}/v1/data/roles",
    json=role_permissions,
    timeout=10,
)
```

### List All Policies
```python
resp = requests.get(f"{OPA_URL}/v1/policies", timeout=10)
policies = resp.json().get("result", [])
for p in policies:
    print(f"  {p['id']} — {len(p.get('raw', ''))} bytes")
```

## OPA CLI Reference

```bash
# Evaluate a policy locally
opa eval -i input.json -d policy.rego "data.authz.allow"

# Run Rego unit tests
opa test ./policies/ -v

# Check policy syntax
opa check policy.rego

# Format Rego files
opa fmt -w policy.rego

# Start OPA as a server
opa run --server --addr :8181 ./policies/ ./data/

# Build an OPA bundle
opa build -b ./policies/ -o bundle.tar.gz
```

## Kubernetes Gatekeeper Integration

```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          provided := {l | input.review.object.metadata.labels[l]}
          required := {l | l := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("Missing required labels: %v", [missing])
        }
```

## Output Format

```json
{
  "decision_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "result": true,
  "policy": "data.authz.allow",
  "input": {
    "user": {"role": "admin"},
    "action": "delete",
    "resource": "/api/users/42"
  }
}
```

===== END FILE =====


===== FILE: skills/implementing-policy-as-code-with-open-policy-agent/references/standards.md =====
# Standards Reference: Policy as Code with OPA

## NIST SP 800-53 - Security and Privacy Controls

| Control | OPA Policy | Description |
|---------|-----------|-------------|
| AC-3 | Block unauthorized access | Enforce RBAC and namespace isolation |
| AC-6 | Least privilege | Block privileged containers and host access |
| CM-2 | Baseline configuration | Require resource limits and labels |
| CM-6 | Configuration settings | Enforce approved image registries |
| SI-7 | Software integrity | Require image signatures and digests |

## CIS Kubernetes Benchmark Mapping

- 5.1.1: Ensure RBAC is enabled → OPA can enforce RBAC policies
- 5.2.1: Minimize privileged containers → K8sBlockPrivileged constraint
- 5.2.2: Minimize host namespace sharing → Block hostNetwork/hostPID
- 5.2.5: Ensure allowPrivilegeEscalation is false → OPA constraint
- 5.7.1: Create administrative boundaries between resources → Namespace policies

## OWASP Kubernetes Security Cheat Sheet

- Enforce Pod Security Standards via admission control
- Restrict container capabilities using OPA policies
- Enforce network policies and resource quotas
- Validate image provenance and signatures

===== END FILE =====


===== FILE: skills/implementing-policy-as-code-with-open-policy-agent/references/workflows.md =====
# Workflow Reference: Policy as Code with OPA

## Policy Lifecycle

```
Author Rego Policy
       │
       ▼
┌──────────────────┐
│ Unit Test with   │
│ OPA test         │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Integration Test │
│ with conftest    │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Deploy to Cluster│
│ (warn mode)      │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Monitor + Triage │
│ Violations       │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Switch to deny   │
│ mode             │
└──────────────────┘
```

## OPA/Gatekeeper Architecture

```
API Request → Kubernetes API Server → Gatekeeper Webhook
                                           │
                                    ┌──────┴──────┐
                                    │ OPA Engine  │
                                    │ (Rego eval) │
                                    └──────┬──────┘
                                           │
                                    ┌──────┴──────┐
                                    │ Constraint  │
                                    │ Templates   │
                                    └──────┬──────┘
                                           │
                                      Allow / Deny
```

===== END FILE =====


===== FILE: skills/implementing-policy-as-code-with-open-policy-agent/scripts/agent.py =====
#!/usr/bin/env python3
"""Open Policy Agent (OPA) policy-as-code agent.

Evaluates security policies against infrastructure configurations using
the OPA REST API or CLI. Supports evaluating Rego policies for Kubernetes
admission control, Terraform plans, IAM policies, and custom security rules.
"""
import argparse
import json
import os
import subprocess
import sys
from datetime import datetime, timezone

try:
    import requests
except ImportError:
    requests = None


def find_opa_binary():
    """Locate the OPA binary on the system."""
    custom_path = os.environ.get("OPA_PATH")
    if custom_path and os.path.isfile(custom_path):
        return custom_path
    for name in ["opa", "opa.exe"]:
        for directory in os.environ.get("PATH", "").split(os.pathsep):
            full_path = os.path.join(directory, name)
            if os.path.isfile(full_path):
                return full_path
    return None


def eval_policy_cli(opa_bin, policy_path, input_path, data_path=None, query="data"):
    """Evaluate a Rego policy using OPA CLI."""
    cmd = [opa_bin, "eval", "--format", "json"]
    cmd.extend(["--bundle", policy_path])
    if input_path:
        cmd.extend(["--input", input_path])
    if data_path:
        cmd.extend(["--data", data_path])
    cmd.append(query)

    print(f"[*] Running: {' '.join(cmd)}")
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        timeout=120,
    )
    if result.returncode != 0:
        print(f"[!] OPA error: {result.stderr}", file=sys.stderr)
        return None
    try:
        return json.loads(result.stdout)
    except json.JSONDecodeError:
        print(f"[!] Failed to parse OPA output", file=sys.stderr)
        return None


def eval_policy_api(opa_url, policy_path, input_data):
    """Evaluate a policy via OPA REST API."""
    if not requests:
        print("[!] 'requests' library required for API mode", file=sys.stderr)
        sys.exit(1)
    url = f"{opa_url}/v1/data/{policy_path.replace('.', '/')}"
    print(f"[*] Querying OPA API: {url}")
    resp = requests.post(
        url,
        json={"input": input_data},
        timeout=30,
    )
    resp.raise_for_status()
    return resp.json()


def test_policies(opa_bin, policy_dir):
    """Run OPA test suite against policy directory."""
    cmd = [opa_bin, "test", "--format", "json", policy_dir, "-v"]
    print(f"[*] Running policy tests: {' '.join(cmd)}")
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        timeout=120,
    )
    try:
        test_results = json.loads(result.stdout)
    except json.JSONDecodeError:
        test_results = []
    return test_results, result.returncode


def check_policy_syntax(opa_bin, policy_path):
    """Check Rego policy syntax."""
    cmd = [opa_bin, "check", "--format", "json", policy_path]
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        timeout=30,
    )
    if result.returncode == 0:
        print(f"[+] Policy syntax valid: {policy_path}")
        return True, []
    try:
        errors = json.loads(result.stdout)
    except json.JSONDecodeError:
        errors = [{"message": result.stderr}]
    print(f"[!] Syntax errors in {policy_path}")
    return False, errors


def extract_violations(eval_result, violation_key="violations"):
    """Extract policy violations from OPA evaluation result."""
    violations = []
    if not eval_result:
        return violations

    results = eval_result.get("result", [])
    if isinstance(results, list):
        for entry in results:
            bindings = entry.get("bindings", {})
            expressions = entry.get("expressions", [])
            for expr in expressions:
                value = expr.get("value", {})
                if isinstance(value, dict):
                    for key, val in value.items():
                        if key == violation_key and isinstance(val, list):
                            violations.extend(val)
                        elif isinstance(val, dict):
                            nested_violations = val.get(violation_key, [])
                            if isinstance(nested_violations, list):
                                violations.extend(nested_violations)
    elif isinstance(results, dict):
        violations = results.get(violation_key, [])

    return violations


def format_summary(violations, test_results, policy_path, input_path):
    """Print evaluation summary."""
    print(f"\n{'='*60}")
    print(f"  OPA Policy Evaluation Report")
    print(f"{'='*60}")
    print(f"  Policy    : {policy_path}")
    print(f"  Input     : {input_path or 'N/A'}")
    print(f"  Violations: {len(violations)}")

    if test_results:
        passed = sum(1 for t in test_results if t.get("pass", t.get("result") == "pass"))
        failed = len(test_results) - passed
        print(f"  Tests     : {passed} passed, {failed} failed")

    if violations:
        severity_counts = {}
        for v in violations:
            sev = "HIGH"
            if isinstance(v, dict):
                sev = v.get("severity", v.get("level", "HIGH"))
            severity_counts[sev] = severity_counts.get(sev, 0) + 1

        print(f"\n  Violations by Severity:")
        for sev, count in sorted(severity_counts.items()):
            print(f"    {sev:10s}: {count}")

        print(f"\n  Violation Details:")
        for v in violations[:20]:
            if isinstance(v, dict):
                msg = v.get("msg", v.get("message", str(v)))
                resource = v.get("resource", v.get("name", ""))
                sev = v.get("severity", "HIGH")
                print(f"    [{sev:6s}] {resource:30s} | {msg[:60]}")
            else:
                print(f"    {str(v)[:80]}")

    return len(violations)


def main():
    parser = argparse.ArgumentParser(
        description="Open Policy Agent policy-as-code evaluation agent"
    )
    sub = parser.add_subparsers(dest="command", help="Action")

    p_eval = sub.add_parser("eval", help="Evaluate policy against input")
    p_eval.add_argument("--policy", required=True, help="Path to Rego policy or bundle dir")
    p_eval.add_argument("--input", dest="input_file", help="Path to input JSON")
    p_eval.add_argument("--data", help="Path to external data JSON")
    p_eval.add_argument("--query", default="data", help="OPA query (default: data)")
    p_eval.add_argument("--violation-key", default="violations",
                        help="Key in result containing violations")

    p_api = sub.add_parser("api", help="Evaluate via OPA REST API")
    p_api.add_argument("--url", default="http://localhost:8181", help="OPA server URL")
    p_api.add_argument("--policy-path", required=True, help="OPA document path (e.g., authz.allow)")
    p_api.add_argument("--input", dest="input_file", required=True, help="Input JSON file")

    p_test = sub.add_parser("test", help="Run OPA test suite")
    p_test.add_argument("--policy-dir", required=True, help="Directory containing policies and tests")

    p_check = sub.add_parser("check", help="Check Rego syntax")
    p_check.add_argument("--policy", required=True, help="Policy file or directory")

    parser.add_argument("--output", "-o", help="Output JSON report path")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    if not args.command:
        parser.print_help()
        sys.exit(1)

    opa_bin = find_opa_binary()
    violations = []
    test_results = []

    if args.command == "eval":
        if not opa_bin:
            print("[!] OPA binary not found", file=sys.stderr)
            sys.exit(1)
        eval_result = eval_policy_cli(
            opa_bin, args.policy, args.input_file, args.data, args.query
        )
        violations = extract_violations(eval_result, args.violation_key)
        format_summary(violations, [], args.policy, args.input_file)

    elif args.command == "api":
        with open(args.input_file, "r") as f:
            input_data = json.load(f)
        eval_result = eval_policy_api(args.url, args.policy_path, input_data)
        violations = extract_violations(eval_result)
        format_summary(violations, [], args.policy_path, args.input_file)

    elif args.command == "test":
        if not opa_bin:
            print("[!] OPA binary not found", file=sys.stderr)
            sys.exit(1)
        test_results, returncode = test_policies(opa_bin, args.policy_dir)
        format_summary([], test_results, args.policy_dir, None)

    elif args.command == "check":
        if not opa_bin:
            print("[!] OPA binary not found", file=sys.stderr)
            sys.exit(1)
        valid, errors = check_policy_syntax(opa_bin, args.policy)
        if not valid:
            for e in errors:
                print(f"    Error: {e}")

    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "Open Policy Agent",
        "command": args.command,
        "violations_count": len(violations),
        "violations": violations,
        "test_results": test_results,
        "risk_level": (
            "CRITICAL" if len(violations) > 10
            else "HIGH" if len(violations) > 0
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


===== FILE: skills/implementing-policy-as-code-with-open-policy-agent/scripts/process.py =====
#!/usr/bin/env python3
"""
OPA Policy Evaluation Pipeline Script

Runs conftest against Kubernetes manifests and Terraform files,
evaluates policy compliance, and generates reports.

Usage:
    python process.py --manifests-dir ./k8s --policies-dir ./policies
    python process.py --manifests-dir ./terraform --policies-dir ./policies --parser hcl2
"""

import argparse
import json
import os
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime, timezone
from pathlib import Path


@dataclass
class PolicyViolation:
    file: str
    rule: str
    message: str
    severity: str = "HIGH"


def run_conftest(manifests_dir: str, policies_dir: str,
                 parser: str = "yaml") -> dict:
    """Run conftest and return JSON results."""
    files = []
    extensions = {"yaml": [".yaml", ".yml"], "hcl2": [".tf"], "dockerfile": ["Dockerfile"]}
    for ext in extensions.get(parser, [".yaml", ".yml"]):
        files.extend(str(p) for p in Path(manifests_dir).rglob(f"*{ext}"))

    if not files:
        return {"results": [], "error": f"No {parser} files found in {manifests_dir}"}

    cmd = [
        "conftest", "test",
        "--policy", policies_dir,
        "--output", "json",
        "--parser", parser
    ] + files

    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
        if proc.stdout:
            return {"results": json.loads(proc.stdout), "error": ""}
        return {"results": [], "error": proc.stderr[:300]}
    except subprocess.TimeoutExpired:
        return {"results": [], "error": "conftest timed out"}
    except FileNotFoundError:
        return {"results": [], "error": "conftest not found"}
    except json.JSONDecodeError:
        return {"results": [], "error": "Failed to parse conftest output"}


def parse_conftest_results(results: list) -> list:
    """Parse conftest JSON results into violations."""
    violations = []
    for result in results:
        filename = result.get("filename", "unknown")
        for failure in result.get("failures", []):
            violations.append(PolicyViolation(
                file=filename,
                rule=failure.get("metadata", {}).get("rule", "unknown"),
                message=failure.get("msg", ""),
                severity="HIGH"
            ))
        for warning in result.get("warnings", []):
            violations.append(PolicyViolation(
                file=filename,
                rule=warning.get("metadata", {}).get("rule", "unknown"),
                message=warning.get("msg", ""),
                severity="MEDIUM"
            ))
    return violations


def check_gatekeeper_violations(namespace: str = "") -> list:
    """Query Gatekeeper audit violations from the cluster."""
    cmd = ["kubectl", "get", "constraints", "-o", "json"]
    if namespace:
        cmd.extend(["-n", namespace])

    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        if proc.returncode != 0:
            return []

        data = json.loads(proc.stdout)
        violations = []
        for item in data.get("items", []):
            status = item.get("status", {})
            for v in status.get("violations", []):
                violations.append(PolicyViolation(
                    file=f"{v.get('kind', '')}/{v.get('name', '')}",
                    rule=item.get("kind", ""),
                    message=v.get("message", ""),
                    severity="HIGH" if item.get("spec", {}).get("enforcementAction") == "deny" else "MEDIUM"
                ))
        return violations
    except (subprocess.TimeoutExpired, FileNotFoundError, json.JSONDecodeError):
        return []


def main():
    parser = argparse.ArgumentParser(description="OPA Policy Evaluation Pipeline")
    parser.add_argument("--manifests-dir", required=True)
    parser.add_argument("--policies-dir", required=True)
    parser.add_argument("--parser", default="yaml", choices=["yaml", "hcl2", "dockerfile"])
    parser.add_argument("--output", default="policy-report.json")
    parser.add_argument("--fail-on-violations", action="store_true")
    parser.add_argument("--check-cluster", action="store_true",
                        help="Also check Gatekeeper violations in cluster")
    args = parser.parse_args()

    violations = []

    print(f"[*] Evaluating policies from {args.policies_dir} against {args.manifests_dir}")
    result = run_conftest(os.path.abspath(args.manifests_dir),
                          os.path.abspath(args.policies_dir), args.parser)

    if result.get("error"):
        print(f"[WARN] {result['error']}")
    else:
        violations.extend(parse_conftest_results(result["results"]))
        print(f"    conftest: {len(violations)} violations")

    if args.check_cluster:
        cluster_violations = check_gatekeeper_violations()
        violations.extend(cluster_violations)
        print(f"    cluster: {len(cluster_violations)} audit violations")

    report = {
        "metadata": {"date": datetime.now(timezone.utc).isoformat()},
        "summary": {
            "total_violations": len(violations),
            "high": sum(1 for v in violations if v.severity == "HIGH"),
            "medium": sum(1 for v in violations if v.severity == "MEDIUM")
        },
        "violations": [
            {"file": v.file, "rule": v.rule, "message": v.message, "severity": v.severity}
            for v in violations
        ]
    }

    output_path = os.path.abspath(args.output)
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[*] Report: {output_path}")

    passed = len(violations) == 0
    print(f"\n[{'PASS' if passed else 'FAIL'}] {len(violations)} policy violations found")
    for v in violations[:10]:
        print(f"  [{v.severity}] {v.file}: {v.message[:100]}")

    if args.fail_on_violations and not passed:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-secret-scanning-with-gitleaks/SKILL.md =====
---
name: implementing-secret-scanning-with-gitleaks
description: 'This skill covers implementing Gitleaks for detecting and preventing
  hardcoded secrets in git repositories. It addresses configuring pre-commit hooks,
  CI/CD pipeline integration, custom rule authoring for organization-specific secrets,
  baseline management for existing repositories, and remediation workflows for exposed
  credentials.

  '
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- cicd
- secret-scanning
- gitleaks
- pre-commit
- secure-sdlc
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
- T1003
- T1110
---

# Implementing Secret Scanning with Gitleaks

## When to Use

- When developers may accidentally commit API keys, passwords, tokens, or private keys to repositories
- When establishing pre-commit gates that prevent secrets from entering the git history
- When scanning existing repository history for previously committed secrets that need rotation
- When compliance requirements mandate secret detection across all source code repositories
- When migrating from manual secret audits to automated continuous scanning

**Do not use** for detecting secrets in running applications or memory (use runtime secret detection), for managing secrets after detection (use Vault or AWS Secrets Manager), or for scanning container images (use Trivy or Grype).

## Prerequisites

- Gitleaks v8.18+ installed via binary, Go install, or Docker
- Pre-commit framework installed for local hook integration
- Git repository with history to scan
- CI/CD platform access (GitHub Actions, GitLab CI, or equivalent)

## Workflow

### Step 1: Install and Run Initial Repository Scan

Perform a baseline scan of the repository to identify all existing secrets in the git history.

```bash
# Install Gitleaks
brew install gitleaks  # macOS
# or download binary from https://github.com/gitleaks/gitleaks/releases

# Scan entire git history for secrets
gitleaks detect --source . --report-format json --report-path gitleaks-report.json -v

# Scan only staged changes (for pre-commit)
gitleaks protect --staged --report-format json --report-path gitleaks-staged.json

# Scan specific commit range
gitleaks detect --source . --log-opts="HEAD~10..HEAD" --report-format json

# Scan without git history (filesystem only)
gitleaks detect --source . --no-git --report-format json
```

### Step 2: Configure Pre-Commit Hook

Set up Gitleaks as a pre-commit hook to prevent secrets from being committed.

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.21.2
    hooks:
      - id: gitleaks
        name: gitleaks
        description: Detect hardcoded secrets using Gitleaks
        entry: gitleaks protect --staged --verbose --redact
        language: golang
        pass_filenames: false
```

```bash
# Install pre-commit framework
pip install pre-commit

# Install hooks defined in .pre-commit-config.yaml
pre-commit install

# Run against all files (not just staged)
pre-commit run gitleaks --all-files

# Test the hook with a deliberate secret
echo 'AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"' >> test.txt
git add test.txt
git commit -m "test"  # Should be blocked by gitleaks
```

### Step 3: Integrate into GitHub Actions

```yaml
# .github/workflows/secret-scanning.yml
name: Secret Scanning

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  gitleaks:
    name: Gitleaks Secret Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for comprehensive scanning

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITLEAKS_LICENSE: ${{ secrets.GITLEAKS_LICENSE }}  # Required for gitleaks-action v2

      # Alternative: Run Gitleaks directly
      - name: Install Gitleaks
        run: |
          wget -q https://github.com/gitleaks/gitleaks/releases/download/v8.21.2/gitleaks_8.21.2_linux_x64.tar.gz
          tar -xzf gitleaks_8.21.2_linux_x64.tar.gz
          chmod +x gitleaks

      - name: Scan for secrets
        run: |
          if [ "${{ github.event_name }}" == "pull_request" ]; then
            ./gitleaks detect \
              --source . \
              --log-opts="${{ github.event.pull_request.base.sha }}..${{ github.event.pull_request.head.sha }}" \
              --report-format sarif \
              --report-path gitleaks.sarif \
              --exit-code 1
          else
            ./gitleaks detect \
              --source . \
              --report-format sarif \
              --report-path gitleaks.sarif \
              --exit-code 1 \
              --baseline-path .gitleaks-baseline.json
          fi

      - name: Upload SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: gitleaks.sarif
          category: gitleaks
```

### Step 4: Author Custom Detection Rules

Create organization-specific rules for internal secret patterns.

```toml
# .gitleaks.toml
title = "Organization Gitleaks Configuration"

[extend]
useDefault = true  # Include all default rules

# Custom rule for internal API tokens
[[rules]]
id = "internal-api-token"
description = "Internal API token for service-to-service auth"
regex = '''(?i)x-internal-token["\s:=]+["\']?([a-zA-Z0-9_\-]{40,})["\']?'''
entropy = 3.5
keywords = ["x-internal-token"]
tags = ["internal", "api"]

[[rules]]
id = "database-connection-string"
description = "Database connection string with embedded credentials"
regex = '''(?i)(postgres|mysql|mongodb|redis)://[^:]+:[^@]+@[^/]+/\w+'''
keywords = ["postgres://", "mysql://", "mongodb://", "redis://"]
tags = ["database", "credentials"]

[[rules]]
id = "jwt-secret"
description = "JWT signing secret"
regex = '''(?i)(jwt[_-]?secret|jwt[_-]?key)["\s:=]+["\']?([a-zA-Z0-9/+_\-]{32,})["\']?'''
entropy = 3.0
keywords = ["jwt_secret", "jwt-secret", "jwt_key", "jwt-key"]

# Allowlist for test files and known safe patterns
[allowlist]
description = "Global allowlist"
paths = [
  '''(^|/)test(s)?/''',
  '''(^|/)spec/''',
  '''\.test\.(js|ts|py)$''',
  '''\.spec\.(js|ts|py)$''',
  '''__mocks__/''',
  '''fixtures/''',
  '''(^|/)vendor/''',
  '''node_modules/'''
]
regexes = [
  '''EXAMPLE''',
  '''example\.com''',
  '''test[-_]?(key|secret|token|password)''',
  '''(?i)placeholder''',
  '''000000+'''
]
```

### Step 5: Manage Baselines for Existing Repositories

Create a baseline of known findings to avoid blocking development while historical secrets are being rotated.

```bash
# Generate baseline from current state
gitleaks detect --source . --report-format json --report-path .gitleaks-baseline.json

# Subsequent scans compare against baseline (only new findings trigger failures)
gitleaks detect --source . --baseline-path .gitleaks-baseline.json --exit-code 1

# Review baseline periodically and remove entries as secrets are rotated
cat .gitleaks-baseline.json | python3 -m json.tool | head -50
```

### Step 6: Remediate Exposed Secrets

When a secret is detected, follow the rotation and history cleanup procedure.

```bash
# 1. Immediately rotate the exposed credential
#    - Revoke the old API key/token in the service provider
#    - Generate a new credential
#    - Store the new credential in a secrets manager

# 2. Remove secret from git history using git-filter-repo
pip install git-filter-repo

# Create expressions file for secrets to remove
cat > /tmp/expressions.txt << 'EOF'
regex:AKIA[0-9A-Z]{16}==>REDACTED_AWS_KEY
regex:(?i)password\s*=\s*"[^"]*"==>password="REDACTED"
EOF

git filter-repo --replace-text /tmp/expressions.txt --force

# 3. Force-push the cleaned history (coordinate with team)
# git push --force --all  # WARNING: Requires team coordination

# 4. Add the secret pattern to .gitleaks.toml rules
# 5. Update the baseline file to remove the resolved finding
```

## Key Concepts

| Term | Definition |
|------|------------|
| Secret | Any credential, token, key, or sensitive string that should not appear in source code |
| Pre-commit Hook | Git hook that runs before a commit is created, blocking commits containing detected secrets |
| Entropy | Measure of randomness in a string; high-entropy strings are more likely to be secrets |
| Baseline | Snapshot of existing findings used to differentiate new secrets from pre-existing ones |
| Allowlist | Configuration specifying paths, patterns, or commits to exclude from detection |
| SARIF | Static Analysis Results Interchange Format for uploading findings to security dashboards |
| git-filter-repo | Tool for rewriting git history to remove sensitive data from all commits |

## Tools & Systems

- **Gitleaks**: Open-source secret detection tool supporting pre-commit hooks, CI/CD, and historical scanning
- **pre-commit**: Framework for managing and maintaining multi-language pre-commit hooks
- **git-filter-repo**: History rewriting tool for removing secrets from git history
- **TruffleHog**: Alternative secret scanner with verified secret detection capabilities
- **GitHub Secret Scanning**: Native GitHub feature that detects secrets matching partner patterns

## Common Scenarios

### Scenario: Onboarding Secret Scanning on a Legacy Repository

**Context**: A 5-year-old repository has never been scanned. The team needs to enable secret scanning without blocking all development while historical secrets are rotated.

**Approach**:
1. Run `gitleaks detect` against full history and generate a baseline JSON file
2. Triage each finding: classify as active (needs rotation), inactive (already rotated), or false positive
3. Immediately rotate all active secrets and update consuming services
4. Commit the baseline file (excluding active secrets that have been fixed)
5. Enable pre-commit hooks for new development immediately
6. Add CI/CD scanning with the baseline to catch only new secrets
7. Progressively reduce the baseline as historical secrets are rotated

**Pitfalls**: Generating a baseline without triaging means accepting risk on unrotated secrets. Never assume a historical secret is inactive without verifying with the service provider. Running git-filter-repo on a shared repository without coordination will cause rebase conflicts for all team members.

## Output Format

```
Gitleaks Secret Scanning Report
=================================
Repository: org/web-application
Scan Type: Full History
Commits Scanned: 4,523
Date: 2026-02-23

FINDINGS:
  Total: 12
  New (not in baseline): 3
  Baseline (pre-existing): 9

NEW FINDINGS (blocking):
  [1] AWS Access Key ID
      Rule: aws-access-key-id
      File: src/config/aws.py:23
      Commit: a1b2c3d (2026-02-22, dev@company.com)
      Secret: AKIA...REDACTED
      Entropy: 3.8

  [2] GitHub Personal Access Token
      Rule: github-pat
      File: scripts/deploy.sh:15
      Commit: d4e5f6g (2026-02-21, ops@company.com)
      Secret: ghp_...REDACTED
      Entropy: 4.2

  [3] Internal API Token
      Rule: internal-api-token
      File: src/services/auth.py:89
      Commit: h7i8j9k (2026-02-20, dev@company.com)

QUALITY GATE: FAILED (3 new findings)
Action: Rotate exposed credentials immediately.
```

===== END FILE =====


===== FILE: skills/implementing-secret-scanning-with-gitleaks/LICENSE =====

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


===== FILE: skills/implementing-secret-scanning-with-gitleaks/assets/template.md =====
# Gitleaks Secret Scanning Templates

## Pre-Commit Configuration

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.21.2
    hooks:
      - id: gitleaks
        name: Detect secrets with Gitleaks
        entry: gitleaks protect --staged --verbose --redact
        language: golang
        pass_filenames: false
```

## Organization Gitleaks Configuration

```toml
# .gitleaks.toml
title = "Organization Secret Scanning Rules"

[extend]
useDefault = true

# ─── Custom Rules ───

[[rules]]
id = "internal-service-token"
description = "Internal service-to-service authentication token"
regex = '''(?i)(service[_-]?token|internal[_-]?key)["\s:=]+["']?([A-Za-z0-9_\-]{36,})["']?'''
entropy = 3.5
keywords = ["service_token", "service-token", "internal_key", "internal-key"]

[[rules]]
id = "database-url-with-password"
description = "Database connection URL with embedded password"
regex = '''(?i)(DATABASE_URL|DB_URL|SQLALCHEMY_DATABASE_URI)\s*=\s*["']?(postgres|mysql|mongodb)\+?[a-z]*://[^:]+:[^@]+@'''
keywords = ["DATABASE_URL", "DB_URL", "SQLALCHEMY_DATABASE_URI"]

[[rules]]
id = "encryption-key-hex"
description = "Encryption key in hexadecimal format"
regex = '''(?i)(encryption[_-]?key|aes[_-]?key|secret[_-]?key)\s*=\s*["']?([0-9a-fA-F]{32,64})["']?'''
entropy = 3.0
keywords = ["encryption_key", "aes_key", "secret_key"]

# ─── Allowlist ───

[allowlist]
description = "Global allowlist for false positive reduction"
paths = [
  '''(^|/)test(s)?/''',
  '''(^|/)spec(s)?/''',
  '''\.test\.(js|ts|py|go|rb)$''',
  '''\.spec\.(js|ts|py|go|rb)$''',
  '''(^|/)__tests__/''',
  '''(^|/)__mocks__/''',
  '''(^|/)fixtures/''',
  '''(^|/)testdata/''',
  '''(^|/)vendor/''',
  '''(^|/)node_modules/''',
  '''\.md$''',
  '''CHANGELOG'''
]
regexes = [
  '''(?i)EXAMPLE''',
  '''(?i)PLACEHOLDER''',
  '''(?i)CHANGEME''',
  '''(?i)your[-_]?(api[-_]?key|secret|token|password)''',
  '''(?i)test[-_]?(key|secret|token|password|credential)''',
  '''example\.com''',
  '''localhost''',
  '''0{8,}''',
  '''x{8,}'''
]

# Per-rule allowlists
[[rules.allowlist]]
description = "Allow AWS example keys"
regexes = ["AKIAIOSFODNN7EXAMPLE"]
```

## GitHub Actions Workflow

```yaml
# .github/workflows/secret-scanning.yml
name: Secret Scanning

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * *'

jobs:
  gitleaks:
    name: Gitleaks
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Gitleaks
        run: |
          GITLEAKS_VERSION="8.21.2"
          wget -qO- "https://github.com/gitleaks/gitleaks/releases/download/v${GITLEAKS_VERSION}/gitleaks_${GITLEAKS_VERSION}_linux_x64.tar.gz" | tar xz
          sudo mv gitleaks /usr/local/bin/

      - name: Run scan
        run: |
          if [ "${{ github.event_name }}" == "pull_request" ]; then
            gitleaks detect \
              --source . \
              --log-opts="${{ github.event.pull_request.base.sha }}..${{ github.sha }}" \
              --report-format sarif \
              --report-path gitleaks.sarif \
              --exit-code 1 \
              --verbose
          else
            gitleaks detect \
              --source . \
              --baseline-path .gitleaks-baseline.json \
              --report-format sarif \
              --report-path gitleaks.sarif \
              --exit-code 1 \
              --verbose
          fi

      - name: Upload SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: gitleaks.sarif
          category: gitleaks
```

## Incident Response Template for Exposed Secrets

```markdown
## Secret Exposure Incident Report

**Date Detected**: YYYY-MM-DD
**Detected By**: Gitleaks CI scan / Pre-commit hook / Manual review
**Repository**: org/repo-name
**Severity**: Critical / High

### Exposed Credential Details
- **Type**: [AWS Access Key | GitHub PAT | Database Password | etc.]
- **Rule ID**: [gitleaks rule that detected it]
- **File**: [path/to/file:line]
- **Commit**: [short SHA]
- **Author**: [email]
- **Date Committed**: [date]
- **Exposure Duration**: [time from commit to detection]

### Remediation Actions
- [ ] Credential revoked at service provider
- [ ] New credential generated and stored in secrets manager
- [ ] Consuming services updated to use new credential
- [ ] Service functionality verified
- [ ] Git history cleaned (if required)
- [ ] Baseline updated
- [ ] Root cause documented

### Root Cause
[Why was the secret committed? Missing pre-commit hook? Developer education gap?]

### Preventive Measures
[What changes prevent recurrence? Hook enforcement? Rule addition?]
```

===== END FILE =====


===== FILE: skills/implementing-secret-scanning-with-gitleaks/references/api-reference.md =====
# API Reference: Gitleaks Secret Scanning

## Libraries Used

| Library | Purpose |
|---------|---------|
| `subprocess` | Execute gitleaks CLI commands |
| `json` | Parse gitleaks JSON report output |
| `pathlib` | Handle repository and report file paths |
| `os` | Read `GITLEAKS_CONFIG` environment variable |

## Installation

```bash
# Install gitleaks binary
# macOS
brew install gitleaks

# Linux
curl -sSfL https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks_linux_x64 -o gitleaks
chmod +x gitleaks && sudo mv gitleaks /usr/local/bin/

# Docker
docker pull ghcr.io/gitleaks/gitleaks:latest
```

## CLI Commands

### Scan a Git Repository
```bash
gitleaks git --source=/path/to/repo --report-format=json --report-path=results.json
```

### Scan a Directory (Non-Git)
```bash
gitleaks dir --source=/path/to/code --report-format=json --report-path=results.json
```

### Scan from stdin
```bash
echo "aws_secret_access_key=AKIAIOSFODNN7EXAMPLE" | gitleaks stdin
```

### Key CLI Flags

| Flag | Description |
|------|-------------|
| `--source` | Path to repository or directory to scan |
| `--config`, `-c` | Path to custom gitleaks.toml config |
| `--report-format`, `-f` | Output format: `json`, `csv`, `junit`, `sarif` |
| `--report-path`, `-r` | Path to write the report file |
| `--baseline-path` | Ignore known findings from baseline file |
| `--exit-code` | Exit code when leaks found (default: 1) |
| `--redact` | Redact secrets in output (percent: 0-100) |
| `--verbose`, `-v` | Show verbose scan output |
| `--no-git` | Treat source as plain directory |
| `--log-level` | Log level: trace, debug, info, warn, error |
| `--max-target-megabytes` | Skip files larger than this size |

## Custom Configuration (.gitleaks.toml)

```toml
title = "Custom Gitleaks Config"

[extend]
useDefault = true  # Extend the default ruleset

[[rules]]
id = "custom-internal-token"
description = "Internal API token pattern"
regex = '''(?i)internal[_-]?token\s*[:=]\s*['"]?([a-zA-Z0-9]{32,})'''
tags = ["internal", "token"]
keywords = ["internal_token", "internal-token"]

[[rules]]
id = "custom-db-password"
description = "Database password in config"
regex = '''(?i)(db|database|mysql|postgres)[_-]?pass(word)?\s*[:=]\s*['"]?[^\s'"]{8,}'''
tags = ["database", "password"]

[rules.allowlist]
paths = ['''test/.*''', '''mock/.*''']
regexTarget = "line"
regexes = ['''(?i)example|placeholder|changeme|test''']

[[allowlist.paths]]
regex = '''vendor/.*'''

[[allowlist.commits]]
sha = "abc123def456"
```

## Python Integration

### Run Gitleaks and Parse Results
```python
import subprocess
import json
from pathlib import Path

def scan_repository(repo_path, config_path=None):
    cmd = [
        "gitleaks", "git",
        "--source", str(repo_path),
        "--report-format", "json",
        "--report-path", "/tmp/gitleaks-report.json",
        "--exit-code", "0",
    ]
    if config_path:
        cmd.extend(["--config", str(config_path)])

    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)

    report_path = Path("/tmp/gitleaks-report.json")
    if report_path.exists():
        with open(report_path) as f:
            findings = json.load(f)
        return findings
    return []
```

### Categorize Findings by Severity
```python
HIGH_SEVERITY_RULES = {
    "aws-access-key", "aws-secret-key", "gcp-api-key",
    "github-pat", "private-key", "generic-api-key",
}

def categorize_findings(findings):
    high, medium, low = [], [], []
    for f in findings:
        rule = f.get("RuleID", "")
        if rule in HIGH_SEVERITY_RULES:
            high.append(f)
        elif "password" in rule or "token" in rule:
            medium.append(f)
        else:
            low.append(f)
    return {"high": high, "medium": medium, "low": low}
```

## GitHub Actions Integration

```yaml
name: Gitleaks Secret Scan
on: [push, pull_request]
jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Output Format

```json
[
  {
    "Description": "Detected a Generic API Key",
    "StartLine": 42,
    "EndLine": 42,
    "StartColumn": 15,
    "EndColumn": 55,
    "Match": "REDACTED",
    "Secret": "REDACTED",
    "File": "config/settings.py",
    "Commit": "a1b2c3d4e5f6",
    "Author": "developer@example.com",
    "Date": "2025-01-15T10:30:00Z",
    "RuleID": "generic-api-key",
    "Tags": ["api", "key"],
    "Fingerprint": "a1b2c3d4:config/settings.py:generic-api-key:42"
  }
]
```

===== END FILE =====


===== FILE: skills/implementing-secret-scanning-with-gitleaks/references/standards.md =====
# Standards Reference: Secret Scanning with Gitleaks

## OWASP Top 10 - A07:2021 Identification and Authentication Failures

- Hardcoded credentials in source code enable unauthorized access
- Gitleaks detects API keys, passwords, tokens, and private keys before they reach repositories
- CWE-798: Use of Hard-coded Credentials is a direct mapping

## NIST SSDF (SP 800-218)

### PW.1: Design Software to Meet Security Requirements
- PW.1.1: Identify security requirements including credential management
- Secrets should never be stored in source code; use environment variables or secrets managers

### PS.1: Protect All Forms of Code
- PS.1.1: Store code securely with access controls and secret scanning
- Implement pre-commit hooks to prevent secrets from entering version control

### PS.2: Provide a Mechanism for Verifying Software Release Integrity
- Code signing and secret scanning ensure software releases do not contain embedded credentials

## CIS Software Supply Chain Security Guide

### Source Code Controls
- SC-1: Automated secret scanning on all commits
- SC-5: No hardcoded credentials in source repositories
- SC-6: Secrets detection integrated into CI/CD pipeline

## OWASP SAMM - Secure Build

### Maturity Level 1
- Scan repositories for common secret patterns using default rulesets
- Alert developers when secrets are detected in pull requests

### Maturity Level 2
- Custom rules for organization-specific secret patterns
- Pre-commit hooks prevent secrets from entering history
- Baseline management for legacy codebases

### Maturity Level 3
- Automated secret rotation workflow triggered by detection
- Correlation with secrets management systems for validation
- Historical scanning with git-filter-repo remediation

## PCI DSS v4.0

- Requirement 6.3.1: Security vulnerabilities identified through a defined process including code analysis
- Requirement 8.6.1: Secrets stored in code or configuration files must be protected
- Requirement 8.6.3: Passwords/passphrases for application and system accounts are protected against misuse

## SOC 2 Trust Service Criteria

- CC6.1: Logical access security over protected information assets including credential management
- CC6.7: Restrict transmission of data to authorized external parties (prevent credential leakage)

===== END FILE =====


===== FILE: skills/implementing-secret-scanning-with-gitleaks/references/workflows.md =====
# Workflow Reference: Secret Scanning with Gitleaks

## Secret Detection Pipeline

```
Developer Workstation          CI/CD Pipeline              Security Response
     │                              │                           │
     ▼                              │                           │
┌──────────────┐                    │                           │
│ Pre-commit   │                    │                           │
│ Hook (local) │                    │                           │
└──────┬───────┘                    │                           │
       │                            │                           │
  ┌────┴────┐                       │                           │
  │         │                       │                           │
PASS      FAIL                      │                           │
  │     (blocked)                   │                           │
  ▼                                 │                           │
Push to                             │                           │
Remote                              │                           │
  │                                 ▼                           │
  │                        ┌──────────────┐                     │
  └───────────────────────>│ Gitleaks CI  │                     │
                           │ Scan (PR)    │                     │
                           └──────┬───────┘                     │
                                  │                             │
                            ┌─────┴─────┐                      │
                            │           │                       │
                          PASS        FAIL                      │
                            │     ┌─────┴──────┐               │
                            │     │ Block PR   │               │
                            │     │ + Alert    │──────────────>│
                            │     └────────────┘    ┌──────────┴──────┐
                            │                       │ Rotate Secret   │
                            │                       │ Update Baseline │
                            │                       │ Clean History   │
                            │                       └─────────────────┘
                            ▼
                    Merge Permitted
```

## Gitleaks Rule Configuration Deep Dive

### Rule Anatomy
```toml
[[rules]]
id = "rule-unique-identifier"          # Unique rule ID
description = "Human-readable desc"     # What this rule detects
regex = '''pattern'''                   # Detection regex
entropy = 3.5                           # Minimum entropy threshold (optional)
secretGroup = 1                         # Regex capture group containing secret
keywords = ["key1", "key2"]             # Fast pre-filter keywords
tags = ["aws", "credential"]            # Categorization tags
path = '''\.env$'''                     # Path filter regex (optional)
```

### Built-in Rule Categories
| Category | Example Rules | Count |
|----------|--------------|-------|
| Cloud Provider Keys | aws-access-key-id, gcp-service-account | 15+ |
| API Tokens | github-pat, gitlab-pat, slack-token | 20+ |
| Private Keys | private-key, rsa-private-key | 5+ |
| Database Credentials | generic-password, connection-string | 10+ |
| Service Tokens | stripe-api-key, sendgrid-api-key | 30+ |

### Entropy Scoring
- Entropy measures string randomness (Shannon entropy)
- Random-looking strings (API keys) have entropy > 3.5
- Regular English text has entropy around 2.0-3.0
- Setting entropy threshold reduces false positives on non-random strings
- Combine entropy with regex for highest accuracy

## Remediation Process

### Secret Rotation Checklist
1. Identify the exposed secret type and associated service
2. Log into the service provider and revoke the exposed credential
3. Generate a new credential with the same permissions
4. Store the new credential in a secrets manager (Vault, AWS SM, etc.)
5. Update all consuming services to use the new credential
6. Verify service functionality with the new credential
7. Update the Gitleaks baseline to remove the resolved finding
8. Optionally clean git history with git-filter-repo

### History Cleanup Decision Matrix
| Factor | Clean History | Keep History |
|--------|--------------|--------------|
| Secret is rotated | Optional | Acceptable |
| Repo is public | Required | Never |
| Compliance mandate | Required | Not compliant |
| Active contributor count | < 10 preferred | > 10 difficult |
| Secret exposure duration | Long (high risk) | Short (lower risk) |

===== END FILE =====


===== FILE: skills/implementing-secret-scanning-with-gitleaks/scripts/agent.py =====
#!/usr/bin/env python3
"""Gitleaks secret scanning agent.

Wraps the Gitleaks CLI to scan git repositories, directories, or
specific commits for hardcoded secrets, API keys, tokens, and
credentials. Parses JSON output into structured findings.
"""
import argparse
import json
import os
import subprocess
import sys
from datetime import datetime, timezone


def find_gitleaks_binary():
    """Locate the gitleaks binary on the system."""
    custom_path = os.environ.get("GITLEAKS_PATH")
    if custom_path and os.path.isfile(custom_path):
        return custom_path
    for name in ["gitleaks", "gitleaks.exe"]:
        for directory in os.environ.get("PATH", "").split(os.pathsep):
            full_path = os.path.join(directory, name)
            if os.path.isfile(full_path):
                return full_path
    print("[!] gitleaks binary not found. Install: https://github.com/gitleaks/gitleaks",
          file=sys.stderr)
    sys.exit(1)


def run_detect(gitleaks_bin, target, config=None, baseline=None,
               log_opts=None, no_git=False, verbose=False):
    """Run gitleaks detect on a repository or directory."""
    cmd = [gitleaks_bin, "detect"]
    if no_git:
        cmd.extend(["--no-git", "--source", target])
    else:
        cmd.extend(["--source", target])
    cmd.extend(["--report-format", "json", "--report-path", "/dev/stdout"])
    if config:
        cmd.extend(["--config", config])
    if baseline:
        cmd.extend(["--baseline-path", baseline])
    if log_opts:
        cmd.extend(["--log-opts", log_opts])
    if verbose:
        cmd.append("--verbose")
    cmd.extend(["--exit-code", "0"])

    print(f"[*] Running: {' '.join(cmd)}")
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        timeout=600,
    )
    if result.returncode not in (0, 1):
        print(f"[!] Gitleaks error (exit {result.returncode}): {result.stderr}",
              file=sys.stderr)
    return result.stdout, result.stderr, result.returncode


def run_protect(gitleaks_bin, target, config=None, staged=False, verbose=False):
    """Run gitleaks protect for pre-commit scanning."""
    cmd = [gitleaks_bin, "protect", "--source", target]
    cmd.extend(["--report-format", "json", "--report-path", "/dev/stdout"])
    if staged:
        cmd.append("--staged")
    if config:
        cmd.extend(["--config", config])
    if verbose:
        cmd.append("--verbose")
    cmd.extend(["--exit-code", "0"])

    print(f"[*] Running protect mode: {' '.join(cmd)}")
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        timeout=300,
    )
    return result.stdout, result.stderr, result.returncode


def parse_findings(raw_json):
    """Parse gitleaks JSON output into structured findings."""
    findings = []
    if not raw_json or not raw_json.strip():
        return findings
    try:
        results = json.loads(raw_json)
    except json.JSONDecodeError:
        return findings
    if not isinstance(results, list):
        results = [results]
    for item in results:
        secret_val = item.get("Secret", "")
        findings.append({
            "rule_id": item.get("RuleID", "unknown"),
            "description": item.get("Description", ""),
            "secret": secret_val[:8] + "..." if secret_val else "",
            "file": item.get("File", ""),
            "line": item.get("StartLine", 0),
            "commit": (item.get("Commit", "") or "")[:12],
            "author": item.get("Author", ""),
            "email": item.get("Email", ""),
            "date": item.get("Date", ""),
            "message": (item.get("Message", "") or "")[:80],
            "entropy": item.get("Entropy", 0),
            "fingerprint": item.get("Fingerprint", ""),
            "tags": item.get("Tags", []),
        })
    return findings


def format_summary(findings, target):
    """Print human-readable summary of secrets found."""
    print(f"\n{'='*60}")
    print(f"  Gitleaks Secret Scan Report")
    print(f"{'='*60}")
    print(f"  Target    : {target}")
    print(f"  Secrets   : {len(findings)}")

    if not findings:
        print(f"  Status    : CLEAN - No secrets detected")
        print(f"{'='*60}")
        return

    by_rule = {}
    for f in findings:
        rule = f["rule_id"]
        by_rule.setdefault(rule, []).append(f)

    print(f"\n  Findings by Rule:")
    for rule, items in sorted(by_rule.items(), key=lambda x: -len(x[1])):
        print(f"    {rule:40s}: {len(items)} occurrence(s)")

    by_file = {}
    for f in findings:
        by_file.setdefault(f["file"], []).append(f)

    print(f"\n  Affected Files: {len(by_file)}")
    for filepath, items in sorted(by_file.items(), key=lambda x: -len(x[1]))[:10]:
        print(f"    {filepath} ({len(items)} secret(s))")

    print(f"\n  Top Findings:")
    for f in findings[:15]:
        print(f"    [{f['rule_id']:30s}] {f['file']}:{f['line']} "
              f"(commit: {f['commit']}, secret: {f['secret']})")


def main():
    parser = argparse.ArgumentParser(
        description="Gitleaks secret scanning agent"
    )
    parser.add_argument("--target", required=True,
                        help="Repository path or directory to scan")
    parser.add_argument("--mode", choices=["detect", "protect"], default="detect",
                        help="Scan mode: detect (full history) or protect (pre-commit)")
    parser.add_argument("--config", help="Path to custom .gitleaks.toml config")
    parser.add_argument("--baseline", help="Path to baseline file for known findings")
    parser.add_argument("--log-opts", help="Git log options (e.g., '--since=2026-01-01')")
    parser.add_argument("--no-git", action="store_true",
                        help="Scan files without git history")
    parser.add_argument("--staged", action="store_true",
                        help="Only scan staged changes (protect mode)")
    parser.add_argument("--output", "-o", help="Output JSON report path")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    gitleaks_bin = find_gitleaks_binary()
    print(f"[*] Using gitleaks: {gitleaks_bin}")

    if args.mode == "protect":
        raw_json, stderr, exit_code = run_protect(
            gitleaks_bin, args.target, args.config, args.staged, args.verbose
        )
    else:
        raw_json, stderr, exit_code = run_detect(
            gitleaks_bin, args.target, args.config, args.baseline,
            args.log_opts, args.no_git, args.verbose
        )

    findings = parse_findings(raw_json)
    format_summary(findings, args.target)

    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "gitleaks",
        "target": args.target,
        "mode": args.mode,
        "secrets_found": len(findings),
        "findings": findings,
        "risk_level": (
            "CRITICAL" if len(findings) > 10
            else "HIGH" if len(findings) > 0
            else "LOW"
        ),
    }

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"\n[+] Report saved to {args.output}")
    elif args.verbose:
        print(json.dumps(report, indent=2))

    if findings:
        print(f"\n[!] {len(findings)} secret(s) detected - remediation required")
    else:
        print(f"\n[+] No secrets detected")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-secret-scanning-with-gitleaks/scripts/process.py =====
#!/usr/bin/env python3
"""
Gitleaks Secret Scanning Pipeline Script

Runs Gitleaks scans, manages baselines, evaluates findings,
and generates remediation reports.

Usage:
    python process.py --repo-path /path/to/repo --scan-type detect
    python process.py --repo-path . --scan-type protect --staged
    python process.py --repo-path . --baseline .gitleaks-baseline.json --output report.json
"""

import argparse
import json
import os
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime, timezone
from typing import Optional


@dataclass
class SecretFinding:
    rule_id: str
    description: str
    file: str
    line: int
    commit: str
    author: str
    date: str
    secret: str
    entropy: float
    match: str
    tags: list = field(default_factory=list)
    is_new: bool = True


@dataclass
class ScanResult:
    findings: list = field(default_factory=list)
    new_findings: list = field(default_factory=list)
    baseline_findings: list = field(default_factory=list)
    commits_scanned: int = 0
    scan_duration: float = 0.0
    error: str = ""


def run_gitleaks(repo_path: str, scan_type: str = "detect",
                 baseline_path: Optional[str] = None,
                 commit_range: Optional[str] = None,
                 staged: bool = False,
                 config_path: Optional[str] = None) -> dict:
    """Execute Gitleaks and return JSON results."""
    cmd = ["gitleaks", scan_type, "--source", repo_path,
           "--report-format", "json", "--report-path", "/dev/stdout"]

    if baseline_path and os.path.exists(baseline_path):
        cmd.extend(["--baseline-path", baseline_path])

    if commit_range:
        cmd.extend(["--log-opts", commit_range])

    if staged:
        cmd.append("--staged")

    if config_path:
        cmd.extend(["--config", config_path])

    cmd.append("--verbose")

    try:
        proc = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            timeout=300
        )

        findings = []
        if proc.stdout.strip():
            try:
                findings = json.loads(proc.stdout)
            except json.JSONDecodeError:
                pass

        return {
            "findings": findings if isinstance(findings, list) else [],
            "exit_code": proc.returncode,
            "stderr": proc.stderr
        }

    except subprocess.TimeoutExpired:
        return {"findings": [], "exit_code": -1, "stderr": "Scan timed out after 300s"}
    except FileNotFoundError:
        return {"findings": [], "exit_code": -1,
                "stderr": "gitleaks not found. Install from https://github.com/gitleaks/gitleaks"}


def parse_findings(raw_findings: list) -> list:
    """Parse raw Gitleaks JSON findings into SecretFinding objects."""
    findings = []
    for f in raw_findings:
        redacted_secret = redact_secret(f.get("Secret", ""))
        findings.append(SecretFinding(
            rule_id=f.get("RuleID", "unknown"),
            description=f.get("Description", ""),
            file=f.get("File", ""),
            line=f.get("StartLine", 0),
            commit=f.get("Commit", "")[:8],
            author=f.get("Author", ""),
            date=f.get("Date", ""),
            secret=redacted_secret,
            entropy=f.get("Entropy", 0.0),
            match=f.get("Match", "")[:100],
            tags=f.get("Tags", [])
        ))
    return findings


def redact_secret(secret: str) -> str:
    """Redact a secret, showing only first 4 and last 4 characters."""
    if len(secret) <= 12:
        return "*" * len(secret)
    return secret[:4] + "..." + secret[-4:]


def load_baseline(baseline_path: str) -> set:
    """Load baseline fingerprints for comparison."""
    if not os.path.exists(baseline_path):
        return set()

    with open(baseline_path, "r") as f:
        try:
            baseline = json.load(f)
        except json.JSONDecodeError:
            return set()

    fingerprints = set()
    for entry in baseline:
        fp = f"{entry.get('RuleID', '')}:{entry.get('File', '')}:{entry.get('Commit', '')}"
        fingerprints.add(fp)

    return fingerprints


def classify_findings(findings: list, baseline_fingerprints: set) -> tuple:
    """Separate findings into new and baseline (pre-existing)."""
    new_findings = []
    baseline_findings = []

    for f in findings:
        fp = f"{f.rule_id}:{f.file}:{f.commit}"
        if fp in baseline_fingerprints:
            f.is_new = False
            baseline_findings.append(f)
        else:
            f.is_new = True
            new_findings.append(f)

    return new_findings, baseline_findings


def generate_report(scan_result: ScanResult, repo_path: str) -> dict:
    """Generate a structured scan report."""
    rule_summary = {}
    for f in scan_result.findings:
        rule_summary[f.rule_id] = rule_summary.get(f.rule_id, 0) + 1

    author_summary = {}
    for f in scan_result.new_findings:
        author_summary[f.author] = author_summary.get(f.author, 0) + 1

    return {
        "report_metadata": {
            "repository": repo_path,
            "scan_date": datetime.now(timezone.utc).isoformat(),
            "duration_seconds": scan_result.scan_duration,
            "commits_scanned": scan_result.commits_scanned
        },
        "summary": {
            "total_findings": len(scan_result.findings),
            "new_findings": len(scan_result.new_findings),
            "baseline_findings": len(scan_result.baseline_findings),
            "unique_rules_triggered": len(rule_summary),
            "rules_breakdown": rule_summary,
            "authors_with_new_findings": author_summary
        },
        "quality_gate": {
            "passed": len(scan_result.new_findings) == 0,
            "blocking_count": len(scan_result.new_findings)
        },
        "new_findings": [
            {
                "rule_id": f.rule_id,
                "description": f.description,
                "file": f.file,
                "line": f.line,
                "commit": f.commit,
                "author": f.author,
                "date": f.date,
                "secret_preview": f.secret,
                "entropy": f.entropy
            }
            for f in scan_result.new_findings
        ],
        "remediation_steps": [
            f"1. Rotate the {f.rule_id} credential found in {f.file}"
            for f in scan_result.new_findings
        ]
    }


def main():
    parser = argparse.ArgumentParser(description="Gitleaks Secret Scanning Pipeline")
    parser.add_argument("--repo-path", required=True, help="Path to git repository")
    parser.add_argument("--scan-type", default="detect", choices=["detect", "protect"],
                        help="Scan type: detect (history) or protect (staged/pre-commit)")
    parser.add_argument("--baseline", default=None, help="Path to baseline JSON file")
    parser.add_argument("--commit-range", default=None,
                        help="Git log range (e.g., HEAD~10..HEAD)")
    parser.add_argument("--staged", action="store_true",
                        help="Scan only staged changes (for pre-commit)")
    parser.add_argument("--config", default=None, help="Path to .gitleaks.toml config")
    parser.add_argument("--output", default="gitleaks-report.json", help="Output report path")
    parser.add_argument("--fail-on-findings", action="store_true",
                        help="Exit non-zero on new findings")
    parser.add_argument("--create-baseline", action="store_true",
                        help="Generate baseline from current findings")
    args = parser.parse_args()

    repo_path = os.path.abspath(args.repo_path)
    start_time = datetime.now(timezone.utc)

    print(f"[*] Running Gitleaks {args.scan_type} on {repo_path}")

    raw_result = run_gitleaks(
        repo_path,
        scan_type=args.scan_type,
        baseline_path=args.baseline,
        commit_range=args.commit_range,
        staged=args.staged,
        config_path=args.config
    )

    if raw_result["exit_code"] == -1:
        print(f"[ERROR] {raw_result['stderr']}")
        sys.exit(2)

    scan_result = ScanResult()
    scan_result.findings = parse_findings(raw_result["findings"])
    scan_result.scan_duration = (datetime.now(timezone.utc) - start_time).total_seconds()

    if args.baseline:
        baseline_fps = load_baseline(args.baseline)
        scan_result.new_findings, scan_result.baseline_findings = classify_findings(
            scan_result.findings, baseline_fps
        )
    else:
        scan_result.new_findings = scan_result.findings
        scan_result.baseline_findings = []

    if args.create_baseline:
        baseline_path = os.path.join(repo_path, ".gitleaks-baseline.json")
        with open(baseline_path, "w") as f:
            json.dump(raw_result["findings"], f, indent=2)
        print(f"[*] Baseline created: {baseline_path}")
        print(f"    Contains {len(raw_result['findings'])} findings")
        return

    report = generate_report(scan_result, repo_path)

    output_path = os.path.abspath(args.output)
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[*] Report: {output_path}")

    print(f"\n[*] Total: {len(scan_result.findings)} | "
          f"New: {len(scan_result.new_findings)} | "
          f"Baseline: {len(scan_result.baseline_findings)}")

    if scan_result.new_findings:
        print("\n[!] New secrets detected:")
        for f in scan_result.new_findings:
            print(f"  [{f.rule_id}] {f.file}:{f.line} (commit: {f.commit}, author: {f.author})")
            print(f"    Secret: {f.secret}")

    if report["quality_gate"]["passed"]:
        print("\n[PASS] No new secrets detected.")
    else:
        print(f"\n[FAIL] {len(scan_result.new_findings)} new secrets found. Rotate immediately.")

    if args.fail_on_findings and not report["quality_gate"]["passed"]:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-secrets-scanning-in-ci-cd/SKILL.md =====
---
name: implementing-secrets-scanning-in-ci-cd
description: Integrate gitleaks and trufflehog into CI/CD pipelines to detect leaked
  secrets before deployment
domain: cybersecurity
subdomain: devsecops
tags:
- secrets-scanning
- gitleaks
- trufflehog
- ci-cd
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
---


# Implementing Secrets Scanning in CI/CD

## Overview

This skill covers implementing automated secrets scanning in CI/CD pipelines using gitleaks and trufflehog. It enables security teams to detect API keys, tokens, passwords, and other credentials that have been accidentally committed to source code repositories, providing a CI gate that blocks deployments containing high-severity findings.

Gitleaks scans git repositories and directories for hardcoded secrets using regex patterns and entropy analysis. TruffleHog performs filesystem and git history scans with optional secret verification against live services. Together they provide comprehensive coverage for secrets detection.


## When to Use

- When deploying or configuring implementing secrets scanning in ci cd capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Python 3.9 or later
- gitleaks v8.x installed and available on PATH
- trufflehog v3.x installed and available on PATH
- A git repository or directory to scan
- Access to CI/CD platform (GitHub Actions, GitLab CI, Jenkins)

## Steps

1. **Install scanning tools**: Install gitleaks via package manager or binary download. Install trufflehog via `brew install trufflehog` or download from GitHub releases.

2. **Configure gitleaks**: Create a `.gitleaks.toml` configuration file in the repository root to define custom rules, allowlists, and path exclusions. Use `--config` flag to point to custom configs.

3. **Run gitleaks directory scan**: Execute `gitleaks dir --source . --report-format json --report-path gitleaks-report.json` to scan the working directory and generate a JSON report.

4. **Run trufflehog filesystem scan**: Execute `trufflehog filesystem /path/to/repo --json > trufflehog-report.json` to scan files and output JSON findings to a report file.

5. **Parse and filter findings**: Use the agent script to parse both JSON reports, filter findings by severity (critical, high, medium, low), and determine whether the CI pipeline should pass or fail.

6. **Integrate into CI pipeline**: Add the scanning step to your GitHub Actions workflow, GitLab CI config, or Jenkins pipeline as a pre-deployment gate. Use `--exit-code` flag in gitleaks to control pipeline behavior.

7. **Configure pre-commit hooks**: Set up gitleaks as a pre-commit hook using `gitleaks protect --staged` to catch secrets before they are committed.

8. **Review and triage findings**: Examine the JSON output for false positives, add legitimate entries to `.gitleaksignore`, and rotate any confirmed leaked credentials immediately.

## Expected Output

The agent script produces a JSON report containing:
- Total findings count from each scanner
- Findings grouped by severity level
- Individual finding details including file path, line number, rule ID, and redacted secret
- A CI gate verdict (pass/fail) based on the configured severity threshold
- Execution metadata including scan duration and tool versions

```json
{
  "scan_summary": {
    "tool": "both",
    "total_findings": 3,
    "critical": 1,
    "high": 1,
    "medium": 1,
    "low": 0,
    "ci_gate": "FAIL",
    "fail_reason": "Found 1 critical and 1 high severity findings"
  },
  "findings": [...]
}
```

===== END FILE =====


===== FILE: skills/implementing-secrets-scanning-in-ci-cd/LICENSE =====

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


===== FILE: skills/implementing-secrets-scanning-in-ci-cd/references/api-reference.md =====
# Secrets Scanning API Reference

## Gitleaks CLI

### Subcommands
```bash
gitleaks git    # Scan git repositories
gitleaks dir    # Scan directories or files
gitleaks stdin  # Detect secrets from stdin
```

### Key Flags
| Flag | Description |
|------|-------------|
| `--source, -s` | Path to source directory or repository |
| `--config, -c` | Path to .gitleaks.toml config file |
| `--report-format, -f` | Output format: json, csv, junit, sarif, template |
| `--report-path, -r` | File path to write report |
| `--exit-code` | Exit code when leaks found (default: 1) |
| `--redact` | Redact secrets from output (0-100%, default: 100) |
| `--baseline-path, -b` | Path to baseline report to ignore known findings |
| `--no-banner` | Suppress banner output |
| `--verbose, -v` | Show verbose scan details |
| `--log-level, -l` | Log level: trace, debug, info, warn, error, fatal |
| `--max-target-megabytes` | Skip files larger than specified MB |
| `--enable-rule` | Only enable specific rule IDs |
| `--gitleaks-ignore-path, -i` | Path to .gitleaksignore file |

### Pre-commit Hook
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.21.2
    hooks:
      - id: gitleaks
```

### Protect Staged Files
```bash
gitleaks protect --staged --report-format json --report-path staged-report.json
```

## TruffleHog CLI

### Subcommands
```bash
trufflehog git <repo-url>            # Scan git repository
trufflehog filesystem <path>         # Scan local filesystem
trufflehog github --org=<org>        # Scan GitHub org
trufflehog gitlab --token=<token>    # Scan GitLab instance
trufflehog s3 --bucket=<name>        # Scan S3 bucket
trufflehog docker --image=<img>      # Scan Docker image
```

### Key Flags
| Flag | Description |
|------|-------------|
| `--json, -j` | Output in JSON format (one object per line) |
| `--no-verification` | Skip live secret verification |
| `--concurrency` | Number of concurrent workers (default: 20) |
| `--results` | Filter: verified, unknown, unverified, filtered_unverified |
| `--force-skip-binaries` | Skip binary files during scan |
| `--force-skip-archives` | Skip archive files during scan |
| `--include-paths` | Path to file with include path patterns |
| `--exclude-paths` | Path to file with exclude path patterns |
| `--log-level` | Verbosity: 0 (info) to 5 (trace) |

## GitHub Secret Scanning API

### List Alerts
```bash
curl -H "Authorization: Bearer $TOKEN" \
  https://api.github.com/repos/{owner}/{repo}/secret-scanning/alerts
```

### Update Alert State
```bash
curl -X PATCH \
  -H "Authorization: Bearer $TOKEN" \
  https://api.github.com/repos/{owner}/{repo}/secret-scanning/alerts/{alert_number} \
  -d '{"state": "resolved", "resolution": "revoked"}'
```

## GitHub Actions Integration

```yaml
- name: Gitleaks Scan
  uses: gitleaks/gitleaks-action@v2
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

- name: TruffleHog Scan
  uses: trufflesecurity/trufflehog@main
  with:
    extra_args: --results verified,unknown
```

===== END FILE =====


===== FILE: skills/implementing-secrets-scanning-in-ci-cd/scripts/agent.py =====
#!/usr/bin/env python3
"""Secrets scanning CI/CD gate using gitleaks and trufflehog."""

import argparse
import json
import os
import subprocess
import sys
import tempfile
import time


SEVERITY_ORDER = {"critical": 4, "high": 3, "medium": 2, "low": 1, "info": 0}

GITLEAKS_SEVERITY_MAP = {
    "aws-access-key": "critical",
    "aws-secret-key": "critical",
    "github-pat": "critical",
    "private-key": "critical",
    "generic-api-key": "high",
    "slack-webhook": "high",
    "stripe-api-key": "critical",
    "twilio-api-key": "high",
    "sendgrid-api-key": "high",
    "npm-access-token": "high",
    "pypi-upload-token": "high",
    "gcp-api-key": "critical",
    "heroku-api-key": "high",
    "jwt": "medium",
    "password-in-url": "high",
    "generic-password": "medium",
}


def run_gitleaks(scan_path: str) -> list:
    report_file = tempfile.NamedTemporaryFile(suffix=".json", delete=False)
    report_path = report_file.name
    report_file.close()
    cmd = [
        "gitleaks", "dir",
        "--source", scan_path,
        "--report-format", "json",
        "--report-path", report_path,
        "--exit-code", "0",
        "--no-banner",
    ]
    proc = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    findings = []
    if os.path.exists(report_path) and os.path.getsize(report_path) > 0:
        with open(report_path, "r") as f:
            raw = json.load(f)
        for item in raw:
            rule_id = item.get("RuleID", "unknown")
            severity = GITLEAKS_SEVERITY_MAP.get(rule_id, "medium")
            secret_val = item.get("Secret", "")
            redacted = secret_val[:4] + "****" if len(secret_val) > 4 else "****"
            findings.append({
                "tool": "gitleaks",
                "rule_id": rule_id,
                "description": item.get("Description", ""),
                "file": item.get("File", ""),
                "line": item.get("StartLine", 0),
                "severity": severity,
                "redacted_secret": redacted,
                "commit": item.get("Commit", ""),
                "author": item.get("Author", ""),
                "entropy": item.get("Entropy", 0.0),
            })
    os.unlink(report_path)
    return findings


def run_trufflehog(scan_path: str) -> list:
    cmd = [
        "trufflehog", "filesystem", scan_path,
        "--json",
        "--no-verification",
        "--concurrency", "10",
    ]
    proc = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    findings = []
    for line in proc.stdout.strip().split("\n"):
        if not line.strip():
            continue
        item = json.loads(line)
        detector = item.get("DetectorName", item.get("DetectorType", "unknown"))
        source_meta = item.get("SourceMetadata", {})
        fs_data = source_meta.get("Data", {}).get("Filesystem", {})
        raw_secret = item.get("Raw", "")
        redacted = raw_secret[:4] + "****" if len(raw_secret) > 4 else "****"
        severity = "critical" if item.get("Verified", False) else "high"
        findings.append({
            "tool": "trufflehog",
            "rule_id": detector,
            "description": f"Detected {detector} secret",
            "file": fs_data.get("file", ""),
            "line": fs_data.get("line", 0),
            "severity": severity,
            "redacted_secret": redacted,
            "verified": item.get("Verified", False),
            "decoder_name": item.get("DecoderName", ""),
        })
    return findings


def evaluate_gate(findings: list, fail_on_severity: str) -> dict:
    threshold = SEVERITY_ORDER.get(fail_on_severity, 2)
    counts = {"critical": 0, "high": 0, "medium": 0, "low": 0, "info": 0}
    for f in findings:
        sev = f.get("severity", "medium")
        counts[sev] = counts.get(sev, 0) + 1
    gate_failed = False
    fail_reasons = []
    for sev, count in counts.items():
        if count > 0 and SEVERITY_ORDER.get(sev, 0) >= threshold:
            gate_failed = True
            fail_reasons.append(f"Found {count} {sev} severity findings")
    return {
        "ci_gate": "FAIL" if gate_failed else "PASS",
        "fail_reason": "; ".join(fail_reasons) if fail_reasons else None,
        "total_findings": len(findings),
        **counts,
    }


def main():
    parser = argparse.ArgumentParser(description="Secrets scanning CI/CD gate")
    parser.add_argument("--path", required=True, help="Path to scan for secrets")
    parser.add_argument("--tool", choices=["gitleaks", "trufflehog", "both"],
                        default="both", help="Scanner tool to use")
    parser.add_argument("--fail-on-severity", choices=["critical", "high", "medium", "low"],
                        default="high", help="Minimum severity to fail CI gate")
    parser.add_argument("--output", default=None, help="Output JSON file path")
    args = parser.parse_args()

    if not os.path.exists(args.path):
        print(json.dumps({"error": f"Path does not exist: {args.path}"}))
        sys.exit(1)

    start_time = time.time()
    all_findings = []

    if args.tool in ("gitleaks", "both"):
        all_findings.extend(run_gitleaks(args.path))
    if args.tool in ("trufflehog", "both"):
        all_findings.extend(run_trufflehog(args.path))

    gate_result = evaluate_gate(all_findings, args.fail_on_severity)
    elapsed = round(time.time() - start_time, 2)

    report = {
        "scan_summary": {
            "tool": args.tool,
            "scanned_path": args.path,
            "fail_on_severity": args.fail_on_severity,
            "scan_duration_seconds": elapsed,
            **gate_result,
        },
        "findings": all_findings,
    }

    output_json = json.dumps(report, indent=2)
    if args.output:
        with open(args.output, "w") as f:
            f.write(output_json)
    print(output_json)

    if gate_result["ci_gate"] == "FAIL":
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-semgrep-for-custom-sast-rules/SKILL.md =====
---
name: implementing-semgrep-for-custom-sast-rules
description: Write custom Semgrep SAST rules in YAML to detect application-specific
  vulnerabilities, enforce coding standards, and integrate into CI/CD pipelines.
domain: cybersecurity
subdomain: devsecops
tags:
- semgrep
- sast
- static-analysis
- custom-rules
- devsecops
- code-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
---

# Implementing Semgrep for Custom SAST Rules

## Overview

Semgrep is an open-source static analysis tool that uses pattern-matching to find bugs, enforce code standards, and detect security vulnerabilities. Custom rules are written in YAML using Semgrep's pattern syntax, making it accessible without requiring compiler knowledge. It supports 30+ languages including Python, JavaScript, Go, Java, and C.


## When to Use

- When deploying or configuring implementing semgrep for custom sast rules capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Python 3.8+ or Docker
- Semgrep CLI installed
- Target codebase in a supported language

## Installation

```bash
# Install via pip
pip install semgrep

# Install via Homebrew
brew install semgrep

# Run via Docker
docker run -v "${PWD}:/src" returntocorp/semgrep semgrep --config auto /src

# Verify
semgrep --version
```

## Running Semgrep

```bash
# Auto-detect rules for your code
semgrep --config auto .

# Use Semgrep registry rules
semgrep --config r/python.lang.security

# Use custom rule file
semgrep --config my-rules.yaml .

# Use multiple configs
semgrep --config auto --config ./custom-rules/ .

# JSON output
semgrep --config auto --json . > results.json

# SARIF output for GitHub
semgrep --config auto --sarif . > results.sarif

# Filter by severity
semgrep --config auto --severity ERROR .
```

## Writing Custom Rules

### Basic Pattern Matching

```yaml
# rules/sql-injection.yaml
rules:
  - id: sql-injection-string-format
    languages: [python]
    severity: ERROR
    message: |
      Potential SQL injection via string formatting.
      Use parameterized queries instead.
    pattern: |
      cursor.execute(f"..." % ...)
    metadata:
      cwe: ["CWE-89"]
      owasp: ["A03:2021"]
      category: security
    fix: |
      cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

### Pattern Operators

```yaml
rules:
  - id: hardcoded-secret-in-code
    languages: [python, javascript, typescript]
    severity: ERROR
    message: Hardcoded secret detected in source code
    patterns:
      - pattern-either:
          - pattern: $VAR = "..."
          - pattern: $VAR = '...'
      - metavariable-regex:
          metavariable: $VAR
          regex: (?i)(password|secret|api_key|token|aws_secret)
      - pattern-not: $VAR = ""
      - pattern-not: $VAR = "changeme"
      - pattern-not: $VAR = "PLACEHOLDER"
    metadata:
      cwe: ["CWE-798"]
      category: security
```

### Taint Analysis

```yaml
rules:
  - id: xss-taint-tracking
    languages: [python]
    severity: ERROR
    message: User input flows to HTML response without sanitization
    mode: taint
    pattern-sources:
      - pattern: request.args.get(...)
      - pattern: request.form.get(...)
      - pattern: request.form[...]
    pattern-sinks:
      - pattern: return render_template_string(...)
      - pattern: Markup(...)
    pattern-sanitizers:
      - pattern: bleach.clean(...)
      - pattern: escape(...)
    metadata:
      cwe: ["CWE-79"]
      owasp: ["A03:2021"]
```

### Multiple Language Rule

```yaml
rules:
  - id: insecure-random
    languages: [python, javascript, go, java]
    severity: WARNING
    message: |
      Using insecure random number generator. Use cryptographically
      secure alternatives for security-sensitive operations.
    pattern-either:
      # Python
      - pattern: random.random()
      - pattern: random.randint(...)
      # JavaScript
      - pattern: Math.random()
      # Go
      - pattern: math/rand.Intn(...)
      # Java
      - pattern: new java.util.Random()
    metadata:
      cwe: ["CWE-330"]
```

### Enforce Coding Standards

```yaml
rules:
  - id: require-error-handling
    languages: [go]
    severity: WARNING
    message: Error return value not checked
    pattern: |
      $VAR, _ := $FUNC(...)
    fix: |
      $VAR, err := $FUNC(...)
      if err != nil {
        return fmt.Errorf("$FUNC failed: %w", err)
      }

  - id: no-console-log-in-production
    languages: [javascript, typescript]
    severity: WARNING
    message: Remove console.log before merging to production
    pattern: console.log(...)
    paths:
      exclude:
        - "tests/*"
        - "*.test.*"
```

### JWT Security Rules

```yaml
rules:
  - id: jwt-none-algorithm
    languages: [python]
    severity: ERROR
    message: JWT decoded without algorithm verification - allows token forgery
    patterns:
      - pattern: jwt.decode($TOKEN, ..., algorithms=["none"], ...)
    metadata:
      cwe: ["CWE-347"]

  - id: jwt-no-verification
    languages: [python]
    severity: ERROR
    message: JWT decoded with verification disabled
    patterns:
      - pattern: jwt.decode($TOKEN, ..., options={"verify_signature": False}, ...)
    metadata:
      cwe: ["CWE-345"]
```

## Rule Testing

```yaml
# rules/test-sql-injection.yaml
rules:
  - id: sql-injection-format-string
    languages: [python]
    severity: ERROR
    message: SQL injection via format string
    pattern: |
      cursor.execute(f"...{$VAR}...")

# Test annotation in test file:
# test-sql-injection.py
def bad_query(user_id):
    # ruleid: sql-injection-format-string
    cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

def good_query(user_id):
    # ok: sql-injection-format-string
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

```bash
# Run rule tests
semgrep --test rules/

# Test specific rule
semgrep --config rules/sql-injection.yaml --test
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Semgrep SAST
on: [pull_request]

jobs:
  semgrep:
    runs-on: ubuntu-latest
    container:
      image: returntocorp/semgrep
    steps:
      - uses: actions/checkout@v4

      - name: Run Semgrep
        run: |
          semgrep --config auto \
            --config ./custom-rules/ \
            --sarif --output results.sarif \
            --severity ERROR \
            .

      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: results.sarif
```

### GitLab CI

```yaml
semgrep:
  stage: test
  image: returntocorp/semgrep
  script:
    - semgrep --config auto --config ./custom-rules/ --json --output semgrep.json .
  artifacts:
    reports:
      sast: semgrep.json
```

## Configuration File

```yaml
# .semgrep.yaml
rules:
  - id: my-org-rules
    # ... rules here

# .semgrepignore
tests/
node_modules/
vendor/
*.min.js
```

## Best Practices

1. **Start with auto config** then add custom rules for org-specific patterns
2. **Test rules** with `# ruleid:` and `# ok:` annotations
3. **Use taint mode** for data flow vulnerabilities (XSS, SQLi, SSRF)
4. **Include metadata** (CWE, OWASP) for vulnerability classification
5. **Provide fix suggestions** with the `fix` key where possible
6. **Exclude test files** to reduce false positives
7. **Version control rules** in a shared repository
8. **Run in CI as a blocking check** for ERROR severity findings

===== END FILE =====


===== FILE: skills/implementing-semgrep-for-custom-sast-rules/LICENSE =====

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


===== FILE: skills/implementing-semgrep-for-custom-sast-rules/references/api-reference.md =====
# API Reference: Semgrep Custom SAST Rules

## Libraries Used

| Library | Purpose |
|---------|---------|
| `subprocess` | Execute semgrep CLI scans |
| `json` | Parse semgrep JSON output |
| `yaml` | Read and write custom Semgrep rule files |
| `pathlib` | Handle source code and rule file paths |

## Installation

```bash
# Python package
pip install semgrep

# Homebrew (macOS)
brew install semgrep

# Docker
docker pull semgrep/semgrep:latest
```

## CLI Reference

### Core Commands

```bash
# Scan with auto-detected rules
semgrep scan --config auto --json --output results.json /path/to/code

# Scan with specific rulesets from Semgrep Registry
semgrep scan --config p/python --config p/owasp-top-ten /path/to/code

# Scan with a custom rule file
semgrep scan --config my-rules.yaml /path/to/code

# Scan with multiple configs
semgrep scan --config p/security-audit --config ./custom-rules/ /path/to/code
```

### Key CLI Flags

| Flag | Description |
|------|-------------|
| `--config`, `-c` | Rule source: registry key, YAML file, or directory |
| `--json` | Output results in JSON format |
| `--sarif` | Output in SARIF format (for CI/CD integration) |
| `--output`, `-o` | Write results to file |
| `--severity` | Filter by severity: `INFO`, `WARNING`, `ERROR` |
| `--include` | Only scan files matching glob pattern |
| `--exclude` | Skip files matching glob pattern |
| `--lang` | Restrict scan to specific language |
| `--max-target-bytes` | Skip files larger than N bytes |
| `--timeout` | Per-rule timeout in seconds (default: 5) |
| `--jobs`, `-j` | Number of parallel jobs |
| `--verbose`, `-v` | Show detailed scan progress |
| `--metrics off` | Disable anonymous metrics |

## Custom Rule Syntax

### Basic Pattern Rule
```yaml
rules:
  - id: hardcoded-password
    pattern: password = "..."
    message: "Hardcoded password detected — use environment variables"
    languages: [python]
    severity: ERROR
    metadata:
      cwe: ["CWE-798: Use of Hard-coded Credentials"]
      owasp: ["A07:2021 - Identification and Authentication Failures"]
```

### Pattern Operators
```yaml
rules:
  - id: sql-injection-format-string
    patterns:
      - pattern: |
          cursor.execute($QUERY % ...)
      - pattern-not: |
          cursor.execute("..." % ())
    message: "SQL injection via string formatting — use parameterized queries"
    languages: [python]
    severity: ERROR

  - id: unsafe-deserialization
    pattern-either:
      - pattern: pickle.loads(...)
      - pattern: pickle.load(...)
      - pattern: yaml.load(..., Loader=yaml.Loader)
      - pattern: yaml.unsafe_load(...)
    message: "Unsafe deserialization — may allow remote code execution"
    languages: [python]
    severity: ERROR

  - id: missing-timeout-requests
    patterns:
      - pattern: requests.$METHOD(...)
      - pattern-not: requests.$METHOD(..., timeout=..., ...)
    message: "HTTP request without timeout — may hang indefinitely"
    languages: [python]
    severity: WARNING
```

### Metavariable Patterns
```yaml
rules:
  - id: eval-user-input
    patterns:
      - pattern: |
          $INPUT = request.$METHOD(...)
          ...
          eval($INPUT)
    message: "User input passed to eval() — command injection risk"
    languages: [python]
    severity: ERROR
```

## Python Integration

```python
import subprocess
import json

def run_semgrep(target_path, config="auto", severity=None):
    cmd = [
        "semgrep", "scan",
        "--config", config,
        "--json",
        "--metrics", "off",
        str(target_path),
    ]
    if severity:
        cmd.extend(["--severity", severity])

    result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
    output = json.loads(result.stdout)
    return output.get("results", [])

def summarize_findings(results):
    by_severity = {"ERROR": [], "WARNING": [], "INFO": []}
    for r in results:
        sev = r.get("extra", {}).get("severity", "INFO")
        by_severity[sev].append({
            "rule": r["check_id"],
            "file": r["path"],
            "line": r["start"]["line"],
            "message": r["extra"]["message"],
        })
    return by_severity
```

## Semgrep Registry Rule Packs

| Pack | Description |
|------|-------------|
| `p/python` | Python-specific security and correctness rules |
| `p/javascript` | JavaScript/TypeScript rules |
| `p/owasp-top-ten` | OWASP Top 10 vulnerability patterns |
| `p/security-audit` | Broad security audit rules across languages |
| `p/secrets` | Secret and credential detection |
| `p/ci` | Rules optimized for CI/CD pipelines |
| `p/docker` | Dockerfile security best practices |
| `p/terraform` | Terraform IaC security rules |

## Output Format

```json
{
  "results": [
    {
      "check_id": "python.lang.security.audit.eval-detected",
      "path": "app/views.py",
      "start": {"line": 42, "col": 5},
      "end": {"line": 42, "col": 28},
      "extra": {
        "message": "Detected eval() usage — avoid with untrusted input",
        "severity": "ERROR",
        "metadata": {
          "cwe": ["CWE-95"],
          "owasp": ["A03:2021 - Injection"]
        }
      }
    }
  ],
  "errors": [],
  "stats": {
    "findings": 3,
    "errors": 0,
    "total_time": 2.45
  }
}
```

===== END FILE =====


===== FILE: skills/implementing-semgrep-for-custom-sast-rules/references/standards.md =====
# Standards - Semgrep Custom SAST Rules

## OWASP Top 10 (2021) Coverage

| Category | Semgrep Detection |
|----------|------------------|
| A01 Broken Access Control | Authorization bypass patterns |
| A02 Cryptographic Failures | Weak crypto, hardcoded secrets |
| A03 Injection | SQL, XSS, command injection (taint mode) |
| A04 Insecure Design | Missing input validation |
| A05 Security Misconfiguration | Debug mode, insecure defaults |
| A06 Vulnerable Components | Deprecated API usage |
| A07 Auth Failures | JWT misconfig, session issues |
| A08 Software/Data Integrity | Deserialization, unsigned data |
| A09 Logging Failures | Missing audit logging |
| A10 SSRF | Server-side request forgery (taint mode) |

## CWE Coverage
Common CWEs detectable via Semgrep custom rules: CWE-79 (XSS), CWE-89 (SQLi), CWE-798 (Hardcoded Credentials), CWE-330 (Insecure Random), CWE-502 (Deserialization), CWE-918 (SSRF)

## NIST SP 800-53 Rev 5
- SA-11: Developer Security Testing
- SA-15: Development Process, Standards, and Tools

## Compliance
- PCI DSS v4.0 Req 6.3.2: Secure development with automated tools
- SOC 2 CC8.1: Change management with code scanning

===== END FILE =====


===== FILE: skills/implementing-semgrep-for-custom-sast-rules/scripts/agent.py =====
#!/usr/bin/env python3
"""Semgrep SAST scanning agent.

Wraps the Semgrep CLI to perform static application security testing
using built-in rulesets and custom rules. Parses JSON output to produce
structured vulnerability findings with severity, CWE, and OWASP mappings.
"""
import argparse
import json
import os
import subprocess
import sys
from datetime import datetime, timezone


def find_semgrep_binary():
    """Locate the semgrep binary on the system."""
    custom_path = os.environ.get("SEMGREP_PATH")
    if custom_path and os.path.isfile(custom_path):
        return custom_path
    for name in ["semgrep", "semgrep.exe"]:
        for directory in os.environ.get("PATH", "").split(os.pathsep):
            full_path = os.path.join(directory, name)
            if os.path.isfile(full_path):
                return full_path
    print("[!] semgrep not found. Install: pip install semgrep", file=sys.stderr)
    sys.exit(1)


def run_scan(semgrep_bin, target, configs=None, severity=None,
             exclude=None, include=None, max_target_bytes=None,
             timeout_per_rule=None, verbose=False):
    """Run semgrep scan and return JSON results."""
    cmd = [semgrep_bin, "scan", "--json"]

    if configs:
        for cfg in configs:
            cmd.extend(["--config", cfg])
    else:
        cmd.extend(["--config", "auto"])

    if severity:
        cmd.extend(["--severity", severity])
    if exclude:
        for pattern in exclude:
            cmd.extend(["--exclude", pattern])
    if include:
        for pattern in include:
            cmd.extend(["--include", pattern])
    if max_target_bytes:
        cmd.extend(["--max-target-bytes", str(max_target_bytes)])
    if timeout_per_rule:
        cmd.extend(["--timeout", str(timeout_per_rule)])
    if verbose:
        cmd.append("--verbose")

    cmd.append(target)

    print(f"[*] Running: {' '.join(cmd)}")
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        timeout=900,
    )
    return result.stdout, result.stderr, result.returncode


def parse_findings(raw_json):
    """Parse semgrep JSON output into structured findings."""
    findings = []
    if not raw_json:
        return findings, {}
    try:
        data = json.loads(raw_json)
    except json.JSONDecodeError:
        return findings, {}

    errors = data.get("errors", [])
    for result in data.get("results", []):
        metadata = result.get("extra", {}).get("metadata", {})
        finding = {
            "rule_id": result.get("check_id", "unknown"),
            "message": result.get("extra", {}).get("message", ""),
            "severity": result.get("extra", {}).get("severity", "WARNING"),
            "path": result.get("path", ""),
            "start_line": result.get("start", {}).get("line", 0),
            "end_line": result.get("end", {}).get("line", 0),
            "matched_code": result.get("extra", {}).get("lines", ""),
            "fix": result.get("extra", {}).get("fix", ""),
            "cwe": metadata.get("cwe", []),
            "owasp": metadata.get("owasp", []),
            "confidence": metadata.get("confidence", ""),
            "references": metadata.get("references", []),
            "category": metadata.get("category", ""),
            "technology": metadata.get("technology", []),
        }
        findings.append(finding)

    stats = {
        "total_findings": len(findings),
        "files_scanned": data.get("paths", {}).get("scanned", []),
        "files_scanned_count": len(data.get("paths", {}).get("scanned", [])),
        "errors": len(errors),
        "parse_errors": [e.get("message", "") for e in errors[:5]],
    }
    return findings, stats


def format_summary(findings, stats, target):
    """Print human-readable scan summary."""
    print(f"\n{'='*60}")
    print(f"  Semgrep SAST Scan Report")
    print(f"{'='*60}")
    print(f"  Target       : {target}")
    print(f"  Files Scanned: {stats.get('files_scanned_count', 0)}")
    print(f"  Findings     : {len(findings)}")
    print(f"  Parse Errors : {stats.get('errors', 0)}")

    severity_counts = {}
    for f in findings:
        sev = f.get("severity", "WARNING")
        severity_counts[sev] = severity_counts.get(sev, 0) + 1

    print(f"\n  By Severity:")
    for sev in ["ERROR", "WARNING", "INFO"]:
        count = severity_counts.get(sev, 0)
        if count > 0:
            print(f"    {sev:10s}: {count}")

    by_rule = {}
    for f in findings:
        by_rule.setdefault(f["rule_id"], []).append(f)

    print(f"\n  Top Rules ({len(by_rule)} unique):")
    for rule, items in sorted(by_rule.items(), key=lambda x: -len(x[1]))[:10]:
        short_rule = rule.split(".")[-1] if "." in rule else rule
        print(f"    {short_rule:45s}: {len(items)} hit(s)")

    by_file = {}
    for f in findings:
        by_file.setdefault(f["path"], []).append(f)

    print(f"\n  Most Affected Files ({len(by_file)} files):")
    for filepath, items in sorted(by_file.items(), key=lambda x: -len(x[1]))[:10]:
        print(f"    {filepath:50s}: {len(items)} finding(s)")

    if findings:
        print(f"\n  Critical/Error Findings:")
        for f in findings[:15]:
            if f["severity"] == "ERROR":
                cwe = f["cwe"][0] if f["cwe"] else ""
                print(f"    {f['path']}:{f['start_line']} [{cwe}] {f['rule_id']}")

    return severity_counts


def main():
    parser = argparse.ArgumentParser(
        description="Semgrep SAST scanning agent"
    )
    parser.add_argument("--target", required=True,
                        help="Path to source code directory or file to scan")
    parser.add_argument("--config", nargs="+", default=None,
                        help="Semgrep config(s): auto, p/security-audit, p/owasp-top-ten, or path to .yaml")
    parser.add_argument("--severity", choices=["INFO", "WARNING", "ERROR"],
                        help="Minimum severity to report")
    parser.add_argument("--exclude", nargs="+",
                        help="File patterns to exclude (e.g., tests/ vendor/)")
    parser.add_argument("--include", nargs="+",
                        help="File patterns to include (e.g., *.py *.js)")
    parser.add_argument("--timeout-per-rule", type=int, default=30,
                        help="Timeout per rule in seconds (default: 30)")
    parser.add_argument("--output", "-o", help="Output JSON report path")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    semgrep_bin = find_semgrep_binary()
    print(f"[*] Using semgrep: {semgrep_bin}")

    raw_json, stderr, exit_code = run_scan(
        semgrep_bin, args.target, args.config, args.severity,
        args.exclude, args.include, timeout_per_rule=args.timeout_per_rule,
        verbose=args.verbose
    )

    findings, stats = parse_findings(raw_json)
    severity_counts = format_summary(findings, stats, args.target)

    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "semgrep",
        "target": args.target,
        "configs": args.config or ["auto"],
        "stats": stats,
        "severity_counts": severity_counts,
        "findings_count": len(findings),
        "findings": findings,
        "risk_level": (
            "CRITICAL" if severity_counts.get("ERROR", 0) > 5
            else "HIGH" if severity_counts.get("ERROR", 0) > 0
            else "MEDIUM" if severity_counts.get("WARNING", 0) > 0
            else "LOW"
        ),
    }

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"\n[+] Report saved to {args.output}")
    elif args.verbose:
        print(json.dumps(report, indent=2))

    print(f"\n[*] Risk Level: {report['risk_level']}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/integrating-dast-with-owasp-zap-in-pipeline/SKILL.md =====
---
name: integrating-dast-with-owasp-zap-in-pipeline
description: Integrates OWASP ZAP (Zed Attack Proxy) into GitHub Actions and GitLab CI pipelines, covering baseline, full, and API scan configuration against running applications, ZAP finding interpretation, scan policy tuning, and DAST quality gates. Use when testing running web apps or REST/GraphQL APIs for XSS, SQLi, CSRF, and auth/authz flaws, or when SAST alone is insufficient and runtime DAST is required for compliance or release gating.
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- cicd
- dast
- owasp-zap
- dynamic-testing
- secure-sdlc
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
---

# Integrating DAST with OWASP ZAP in Pipeline

## When to Use

- When testing running web applications for vulnerabilities like XSS, SQLi, CSRF, and misconfigurations
- When SAST alone is insufficient and runtime behavior testing is required
- When compliance mandates dynamic security testing of web applications before production
- When testing APIs (REST/GraphQL) for authentication, authorization, and injection flaws
- When establishing continuous DAST scanning in staging environments before production deployment

**Do not use** for scanning source code (use SAST), for scanning dependencies (use SCA), or for infrastructure configuration scanning (use IaC scanning tools).

## Prerequisites

- OWASP ZAP Docker image or installed locally (zaproxy/zap-stable or zaproxy/action-*)
- Running target application accessible from the CI/CD runner (staging URL or Docker service)
- ZAP scan rules configuration (optional, for tuning)
- OpenAPI/Swagger specification for API scanning (optional)

## Workflow

### Step 1: Configure ZAP Baseline Scan in GitHub Actions

```yaml
# .github/workflows/dast-scan.yml
name: DAST Security Scan

on:
  deployment_status:
  workflow_dispatch:
    inputs:
      target_url:
        description: 'Target URL to scan'
        required: true

jobs:
  zap-baseline:
    name: ZAP Baseline Scan
    runs-on: ubuntu-latest
    services:
      webapp:
        image: ${{ github.repository }}:${{ github.sha }}
        ports:
          - 8080:8080
        options: --health-cmd="curl -f http://localhost:8080/health" --health-interval=10s --health-timeout=5s --health-retries=5

    steps:
      - uses: actions/checkout@v4

      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.12.0
        with:
          target: 'http://webapp:8080'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a -j'
          allow_issue_writing: false

      - name: Upload ZAP Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: zap-baseline-report
          path: report_html.html
```

### Step 2: Configure ZAP Full Scan for Comprehensive Testing

```yaml
  zap-full-scan:
    name: ZAP Full Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: ZAP Full Scan
        uses: zaproxy/action-full-scan@v0.12.0
        with:
          target: ${{ github.event.inputs.target_url || 'https://staging.example.com' }}
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a -j -T 60'

      - name: Upload Reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: zap-full-report
          path: |
            report_html.html
            report_json.json
```

### Step 3: Configure API Scan with OpenAPI Specification

```yaml
  zap-api-scan:
    name: ZAP API Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: ZAP API Scan
        uses: zaproxy/action-api-scan@v0.12.0
        with:
          target: 'https://staging.example.com/api/openapi.json'
          format: openapi
          rules_file_name: '.zap/api-rules.tsv'
          cmd_options: '-a -j'
```

### Step 4: Configure ZAP Scan Rules

```tsv
# .zap/rules.tsv
# Rule ID	Action (IGNORE/WARN/FAIL)	Description
10003	IGNORE	# Vulnerable JS Library (handled by SCA)
10015	WARN	# Incomplete or No Cache-control Header
10021	FAIL	# X-Content-Type-Options Missing
10035	FAIL	# Strict-Transport-Security Missing
10038	FAIL	# Content Security Policy Missing
10098	IGNORE	# Cross-Domain Misconfiguration (CDN)
40012	FAIL	# Cross Site Scripting (Reflected)
40014	FAIL	# Cross Site Scripting (Persistent)
40018	FAIL	# SQL Injection
40019	FAIL	# SQL Injection (MySQL)
40032	FAIL	# .htaccess Information Leak
90033	FAIL	# Loosely Scoped Cookie
```

### Step 5: Run ZAP with Docker Compose for Local Testing

```yaml
# docker-compose.zap.yml
version: '3.8'
services:
  webapp:
    build: .
    ports:
      - "8080:8080"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 10s
      retries: 5

  zap:
    image: zaproxy/zap-stable:latest
    depends_on:
      webapp:
        condition: service_healthy
    command: >
      zap-baseline.py
        -t http://webapp:8080
        -r /zap/wrk/report.html
        -J /zap/wrk/report.json
        -c /zap/wrk/rules.tsv
        -I
    volumes:
      - ./zap-reports:/zap/wrk
      - ./.zap/rules.tsv:/zap/wrk/rules.tsv
```

## Key Concepts

| Term | Definition |
|------|------------|
| DAST | Dynamic Application Security Testing — tests running applications by sending requests and analyzing responses |
| Baseline Scan | Quick passive scan that spiders the application without active attacks, suitable for CI/CD |
| Full Scan | Active scan including attack payloads for XSS, SQLi, and other injection vulnerabilities |
| API Scan | Targeted scan using OpenAPI/Swagger specs to test all documented API endpoints |
| Spider | ZAP's crawler that discovers application pages and endpoints by following links |
| Active Scan | Phase where ZAP sends attack payloads to discovered endpoints to find exploitable vulnerabilities |
| Passive Scan | Analysis of HTTP responses for security headers, cookies, and information disclosure without sending attacks |
| Scan Policy | Configuration defining which attack types to enable and their intensity levels |

## Tools & Systems

- **OWASP ZAP**: Open-source web application security scanner for DAST testing
- **zaproxy/action-baseline**: GitHub Action for ZAP passive baseline scanning
- **zaproxy/action-full-scan**: GitHub Action for ZAP active full scanning
- **zaproxy/action-api-scan**: GitHub Action for API-focused scanning with OpenAPI support
- **Nuclei**: Alternative vulnerability scanner with template-based detection for CI/CD integration

## Common Scenarios

### Scenario: Integrating DAST into a Staging Deployment Pipeline

**Context**: A team deploys to staging before production and needs automated DAST scanning between stages to catch runtime vulnerabilities.

**Approach**:
1. Add a DAST job in the pipeline that triggers after successful staging deployment
2. Run ZAP baseline scan first for quick passive feedback (2-5 minutes)
3. Follow with a targeted API scan using the application's OpenAPI specification
4. Configure rules.tsv to FAIL on critical findings (XSS, SQLi) and WARN on headers/cookies
5. Upload ZAP reports as pipeline artifacts for review
6. Block production deployment if any FAIL-level findings are detected
7. Schedule weekly full scans against staging for deeper coverage

**Pitfalls**: ZAP full scans can take 30+ minutes and may overwhelm staging servers with attack traffic. Use baseline scans in CI and full scans on schedule. Running DAST against production without coordination can trigger WAF blocks and incident alerts.

## Output Format

```
ZAP DAST Scan Report
======================
Target: https://staging.example.com
Scan Type: Baseline + API
Date: 2026-02-23
Duration: 4m 32s

FINDINGS:
  FAIL: 3
  WARN: 7
  INFO: 12
  PASS: 45

FAILING ALERTS:
  [HIGH] 40012 - Cross Site Scripting (Reflected)
    URL: https://staging.example.com/search?q=<script>
    Method: GET
    Evidence: <script>alert(1)</script>

  [MEDIUM] 10021 - X-Content-Type-Options Missing
    URL: https://staging.example.com/api/v1/*
    Evidence: Response header missing

  [MEDIUM] 10035 - Strict-Transport-Security Missing
    URL: https://staging.example.com/
    Evidence: HSTS header not present

QUALITY GATE: FAILED (1 HIGH, 2 MEDIUM findings)
```

===== END FILE =====


===== FILE: skills/integrating-dast-with-owasp-zap-in-pipeline/LICENSE =====

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


===== FILE: skills/integrating-dast-with-owasp-zap-in-pipeline/assets/template.md =====
# DAST with OWASP ZAP Templates

## ZAP Rules File

```tsv
# .zap/rules.tsv
# Rule ID	Action	Description
10003	IGNORE	# Vulnerable JS Library (handled by SCA tools)
10015	WARN	# Incomplete Cache-control Header
10020	WARN	# X-Frame-Options Header Not Set
10021	FAIL	# X-Content-Type-Options Missing
10035	FAIL	# Strict-Transport-Security Missing
10038	FAIL	# Content Security Policy Missing
10098	IGNORE	# Cross-Domain Misconfiguration (CDN expected)
40012	FAIL	# XSS Reflected
40014	FAIL	# XSS Persistent
40018	FAIL	# SQL Injection
40019	FAIL	# SQL Injection (MySQL)
40024	FAIL	# SQL Injection (PostgreSQL)
40032	WARN	# .htaccess Info Leak
90033	WARN	# Loosely Scoped Cookie
```

## GitHub Actions: Complete DAST Pipeline

```yaml
# .github/workflows/dast.yml
name: DAST Pipeline

on:
  workflow_run:
    workflows: ["Deploy to Staging"]
    types: [completed]

jobs:
  zap-scan:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - uses: actions/checkout@v4

      - name: Wait for staging
        run: |
          for i in $(seq 1 30); do
            if curl -sf https://staging.example.com/health; then break; fi
            sleep 10
          done

      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.12.0
        with:
          target: 'https://staging.example.com'
          rules_file_name: '.zap/rules.tsv'

      - name: ZAP API Scan
        uses: zaproxy/action-api-scan@v0.12.0
        with:
          target: 'https://staging.example.com/api/v1/openapi.json'
          format: openapi
          rules_file_name: '.zap/rules.tsv'

      - name: Upload Reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: dast-reports
          path: report_*.html
```

## Docker Compose for Local DAST Testing

```yaml
# docker-compose.dast.yml
services:
  app:
    build: .
    ports: ["8080:8080"]
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 5s
      retries: 10

  zap:
    image: zaproxy/zap-stable:latest
    depends_on:
      app:
        condition: service_healthy
    volumes:
      - ./zap-reports:/zap/wrk
      - ./.zap/rules.tsv:/zap/wrk/rules.tsv
    command: >
      zap-baseline.py
        -t http://app:8080
        -r /zap/wrk/report.html
        -J /zap/wrk/report.json
        -c /zap/wrk/rules.tsv
        -I
```

===== END FILE =====


===== FILE: skills/integrating-dast-with-owasp-zap-in-pipeline/references/api-reference.md =====
# API Reference: OWASP ZAP DAST Pipeline Integration

## ZAP Docker Scan Scripts

### Baseline Scan (Passive Only)
```bash
docker run --rm -v $(pwd):/zap/wrk zaproxy/zap-stable \
  zap-baseline.py -t https://target.com -J report.json -I
```

### Full Scan (Active + Passive)
```bash
docker run --rm -v $(pwd):/zap/wrk zaproxy/zap-stable \
  zap-full-scan.py -t https://target.com -J report.json -m 5 -I
```

### API Scan (OpenAPI/Swagger)
```bash
docker run --rm -v $(pwd):/zap/wrk zaproxy/zap-stable \
  zap-api-scan.py -t https://target.com/openapi.json -f openapi -J report.json
```

### Return Codes
| Code | Meaning |
|------|---------|
| 0 | No alerts above threshold |
| 1 | Warnings found |
| 2 | Failures found |

### Common Flags
| Flag | Description |
|------|-------------|
| `-t` | Target URL |
| `-J` | JSON report filename |
| `-m` | Max scan duration in minutes |
| `-I` | Do not return failure on warnings |
| `-f` | API spec format (`openapi`, `soap`) |
| `-r` | HTML report filename |
| `-c` | Config file for rule tuning |

## ZAP JSON Report Structure
```json
{"site": [{"alerts": [{"name": "...", "riskdesc": "High (Medium)",
  "cweid": "79", "count": 3, "solution": "..."}]}]}
```

### Risk Levels
| Level | Action |
|-------|--------|
| High | Block deployment |
| Medium | Require review |
| Low | Track as tech debt |
| Informational | Log only |

## References
- ZAP Docker: https://www.zaproxy.org/docs/docker/
- ZAP Automation: https://www.zaproxy.org/docs/automate/

===== END FILE =====


===== FILE: skills/integrating-dast-with-owasp-zap-in-pipeline/references/standards.md =====
# Standards Reference: DAST with OWASP ZAP

## OWASP Top 10 - DAST Coverage

| OWASP Category | ZAP Detection | Alert IDs |
|----------------|---------------|-----------|
| A01: Broken Access Control | Partial | 10020, 10035, 40012 |
| A02: Cryptographic Failures | Yes | 10003, 10041 |
| A03: Injection | Yes | 40012, 40014, 40018, 40019 |
| A05: Security Misconfiguration | Yes | 10015, 10020, 10021, 10035 |
| A07: Auth Failures | Partial | 10010, 10054 |
| A09: Logging Failures | Yes | 10035 |
| A10: SSRF | Partial | Custom scan |

## OWASP SAMM - Verification: Security Testing

### Level 2: DAST Integration
- Automated DAST scanning in CI/CD pipelines
- Baseline scans on every deployment, full scans weekly
- Findings tracked and triaged with defined SLAs

### Level 3: Advanced DAST
- Authenticated scanning with session management
- API-specific scanning with OpenAPI specifications
- Custom scan policies tuned for application-specific risks

## NIST SSDF (SP 800-218)

### PW.8: Test Executable Code
- PW.8.1: Test executable code using dynamic analysis
- DAST validates running application behavior against security requirements
- Integration into CI/CD automates regular testing

## PCI DSS v4.0

- 6.2.4: Use automated methods to prevent common attacks (XSS, SQLi)
- 6.4.1: Web applications are protected against known attacks
- 11.3.1: Internal vulnerability scans performed at least quarterly

===== END FILE =====


===== FILE: skills/integrating-dast-with-owasp-zap-in-pipeline/references/workflows.md =====
# Workflow Reference: DAST with OWASP ZAP

## DAST Pipeline Integration

```
Build & Deploy to Staging
       │
       ▼
┌──────────────────┐
│ Health Check      │
│ (wait for ready)  │
└──────┬───────────┘
       │
       ├──────────────┐
       ▼              ▼
┌───────────┐  ┌───────────┐
│ ZAP       │  │ ZAP API   │
│ Baseline  │  │ Scan      │
│ Scan      │  │ (OpenAPI) │
└─────┬─────┘  └─────┬─────┘
      │               │
      └───────┬───────┘
              ▼
       ┌──────────────┐
       │ Report Gen   │
       │ + Upload     │
       └──────┬───────┘
              │
       ┌──────┴──────┐
       ▼             ▼
     PASS          FAIL
     Deploy to     Block +
     Production    Alert
```

## ZAP Scan Types Comparison

| Scan Type | Duration | Coverage | CI/CD Suitable | Active Attacks |
|-----------|----------|----------|----------------|----------------|
| Baseline | 2-5 min | Passive only | Yes | No |
| Full Scan | 30-120 min | Comprehensive | Scheduled | Yes |
| API Scan | 5-15 min | API endpoints | Yes | Yes |

## ZAP Docker Commands

```bash
# Baseline scan (passive)
docker run --rm -v $(pwd):/zap/wrk zaproxy/zap-stable \
  zap-baseline.py -t http://target:8080 \
  -r report.html -J report.json -c rules.tsv

# Full scan (active)
docker run --rm -v $(pwd):/zap/wrk zaproxy/zap-stable \
  zap-full-scan.py -t http://target:8080 \
  -r report.html -J report.json -c rules.tsv -T 60

# API scan (OpenAPI)
docker run --rm -v $(pwd):/zap/wrk zaproxy/zap-stable \
  zap-api-scan.py -t http://target:8080/openapi.json \
  -f openapi -r report.html -J report.json
```

## Authenticated Scanning Configuration

```yaml
# zap-auth-config.yaml
authentication:
  method: form
  loginUrl: http://target:8080/login
  parameters:
    username: testuser
    password: testpass123
  loggedInIndicator: "\\QWelcome\\E"
  loggedOutIndicator: "\\QSign In\\E"

context:
  name: "auth-context"
  include:
    - "http://target:8080/.*"
  exclude:
    - "http://target:8080/logout"
    - "http://target:8080/static/.*"
```

===== END FILE =====


===== FILE: skills/integrating-dast-with-owasp-zap-in-pipeline/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for running OWASP ZAP DAST scans and parsing results for CI/CD integration."""

import subprocess
import json
import argparse
import sys
import os


def run_baseline_scan(target_url, output_dir):
    """Run ZAP baseline scan (passive only, fast)."""
    print(f"[*] Running ZAP baseline scan on {target_url}...")
    report_path = os.path.join(output_dir, "zap_baseline.json")
    cmd = ["docker", "run", "--rm", "-v", f"{os.path.abspath(output_dir)}:/zap/wrk",
           "zaproxy/zap-stable", "zap-baseline.py", "-t", target_url,
           "-J", "zap_baseline.json", "-I"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
    print(f"  Return code: {result.returncode} (0=pass, 1=warn, 2=fail)")
    return report_path, result.returncode


def run_full_scan(target_url, output_dir, mins=5):
    """Run ZAP full scan (active + passive, slower)."""
    print(f"[*] Running ZAP full scan on {target_url} ({mins} min)...")
    report_path = os.path.join(output_dir, "zap_full.json")
    cmd = ["docker", "run", "--rm", "-v", f"{os.path.abspath(output_dir)}:/zap/wrk",
           "zaproxy/zap-stable", "zap-full-scan.py", "-t", target_url,
           "-J", "zap_full.json", "-m", str(mins), "-I"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=mins * 120)
    print(f"  Return code: {result.returncode}")
    return report_path, result.returncode


def run_api_scan(target_spec, output_dir, spec_format="openapi"):
    """Run ZAP API scan against an OpenAPI/Swagger spec."""
    print(f"[*] Running ZAP API scan with {spec_format} spec...")
    report_path = os.path.join(output_dir, "zap_api.json")
    cmd = ["docker", "run", "--rm", "-v", f"{os.path.abspath(output_dir)}:/zap/wrk",
           "zaproxy/zap-stable", "zap-api-scan.py", "-t", target_spec,
           "-f", spec_format, "-J", "zap_api.json", "-I"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
    print(f"  Return code: {result.returncode}")
    return report_path, result.returncode


def parse_zap_report(report_path):
    """Parse ZAP JSON report and extract findings by risk level."""
    if not os.path.exists(report_path):
        print(f"  [-] Report not found: {report_path}")
        return []
    with open(report_path) as f:
        data = json.load(f)
    alerts = []
    for site in data.get("site", []):
        for alert in site.get("alerts", []):
            alerts.append({
                "name": alert.get("name"), "risk": alert.get("riskdesc", "").split()[0],
                "confidence": alert.get("confidence"), "count": alert.get("count", 0),
                "cweid": alert.get("cweid"), "wascid": alert.get("wascid"),
                "solution": alert.get("solution", "")[:200],
            })
    risk_counts = {}
    for a in alerts:
        risk_counts[a["risk"]] = risk_counts.get(a["risk"], 0) + 1
    print(f"\n[*] Findings: {len(alerts)} unique alerts")
    for risk, count in sorted(risk_counts.items()):
        print(f"  {risk}: {count}")
    return alerts


def apply_quality_gate(alerts, fail_on="High"):
    """Apply quality gate: fail if findings at or above threshold."""
    severity_order = {"Informational": 0, "Low": 1, "Medium": 2, "High": 3}
    threshold = severity_order.get(fail_on, 3)
    blocking = [a for a in alerts if severity_order.get(a.get("risk", ""), 0) >= threshold]
    if blocking:
        print(f"\n[!] QUALITY GATE FAILED: {len(blocking)} findings at {fail_on}+ severity")
        for b in blocking[:5]:
            print(f"  - {b['risk']}: {b['name']}")
        return False
    print(f"\n[+] Quality gate passed (threshold: {fail_on})")
    return True


def generate_sarif(alerts, output_path):
    """Convert ZAP findings to SARIF format for GitHub Advanced Security."""
    rules, results = [], []
    for i, a in enumerate(alerts):
        rule_id = f"ZAP-{a.get('cweid', i)}"
        rules.append({"id": rule_id, "shortDescription": {"text": a["name"]},
                       "defaultConfiguration": {"level": "warning"}})
        results.append({"ruleId": rule_id, "message": {"text": a["name"]},
                         "level": "warning" if a["risk"] != "High" else "error"})
    sarif = {"$schema": "https://json.schemastore.org/sarif-2.1.0.json", "version": "2.1.0",
             "runs": [{"tool": {"driver": {"name": "OWASP ZAP", "rules": rules}},
                        "results": results}]}
    with open(output_path, "w") as f:
        json.dump(sarif, f, indent=2)
    print(f"[*] SARIF report: {output_path}")


def main():
    parser = argparse.ArgumentParser(description="OWASP ZAP DAST Pipeline Agent")
    parser.add_argument("action", choices=["baseline", "full", "api", "parse", "gate"])
    parser.add_argument("--target", help="Target URL or API spec URL")
    parser.add_argument("--report", help="Path to existing ZAP JSON report")
    parser.add_argument("--fail-on", default="High", choices=["Low", "Medium", "High"])
    parser.add_argument("--scan-mins", type=int, default=5)
    parser.add_argument("-o", "--output", default=".")
    args = parser.parse_args()

    os.makedirs(args.output, exist_ok=True)
    if args.action == "baseline":
        rpt, _ = run_baseline_scan(args.target, args.output)
        alerts = parse_zap_report(rpt)
        apply_quality_gate(alerts, args.fail_on)
    elif args.action == "full":
        rpt, _ = run_full_scan(args.target, args.output, args.scan_mins)
        alerts = parse_zap_report(rpt)
        apply_quality_gate(alerts, args.fail_on)
    elif args.action == "api":
        rpt, _ = run_api_scan(args.target, args.output)
        alerts = parse_zap_report(rpt)
        apply_quality_gate(alerts, args.fail_on)
    elif args.action == "parse":
        alerts = parse_zap_report(args.report)
        generate_sarif(alerts, os.path.join(args.output, "zap.sarif"))
    elif args.action == "gate":
        alerts = parse_zap_report(args.report)
        passed = apply_quality_gate(alerts, args.fail_on)
        sys.exit(0 if passed else 1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/integrating-dast-with-owasp-zap-in-pipeline/scripts/process.py =====
#!/usr/bin/env python3
"""
OWASP ZAP DAST Pipeline Script

Orchestrates ZAP scans, parses results, evaluates quality gates,
and generates consolidated DAST reports.

Usage:
    python process.py --target http://localhost:8080 --scan-type baseline
    python process.py --target http://staging.example.com --scan-type api --openapi-url /openapi.json
"""

import argparse
import json
import os
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime, timezone
from typing import Optional


RISK_LEVELS = {"High": 0, "Medium": 1, "Low": 2, "Informational": 3}


@dataclass
class ZapAlert:
    alert_id: int
    name: str
    risk: str
    confidence: str
    description: str
    url: str
    method: str
    evidence: str
    solution: str
    cwe_id: int = 0
    wasc_id: int = 0
    instances_count: int = 1


def run_zap_scan(target: str, scan_type: str = "baseline",
                 rules_file: Optional[str] = None,
                 openapi_url: Optional[str] = None,
                 report_dir: str = "/tmp/zap") -> dict:
    """Run ZAP scan via Docker and return results."""
    os.makedirs(report_dir, exist_ok=True)

    scan_scripts = {
        "baseline": "zap-baseline.py",
        "full": "zap-full-scan.py",
        "api": "zap-api-scan.py"
    }

    script = scan_scripts.get(scan_type, "zap-baseline.py")

    cmd = [
        "docker", "run", "--rm",
        "-v", f"{report_dir}:/zap/wrk",
        "--network", "host",
        "zaproxy/zap-stable",
        script,
        "-t", target,
        "-J", "report.json",
        "-r", "report.html",
        "-w", "report.md"
    ]

    if rules_file and os.path.exists(rules_file):
        cmd.extend(["-c", f"/zap/wrk/{os.path.basename(rules_file)}"])

    if scan_type == "api" and openapi_url:
        cmd.extend(["-f", "openapi"])

    cmd.extend(["-I"])  # Don't return error on warnings

    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=3600)

        json_report = os.path.join(report_dir, "report.json")
        if os.path.exists(json_report):
            with open(json_report, "r") as f:
                return json.load(f)

        return {"error": f"No report generated. stderr: {proc.stderr[:300]}"}

    except subprocess.TimeoutExpired:
        return {"error": "ZAP scan timed out after 3600 seconds"}
    except FileNotFoundError:
        return {"error": "Docker not found. Install Docker to run ZAP scans."}


def parse_zap_results(zap_json: dict) -> list:
    """Parse ZAP JSON report into ZapAlert objects."""
    alerts = []

    for site in zap_json.get("site", []):
        for alert in site.get("alerts", []):
            instances = alert.get("instances", [])
            first_instance = instances[0] if instances else {}

            alerts.append(ZapAlert(
                alert_id=int(alert.get("pluginid", 0)),
                name=alert.get("name", ""),
                risk=alert.get("riskdesc", "").split(" ")[0] if alert.get("riskdesc") else "Informational",
                confidence=alert.get("confidence", ""),
                description=alert.get("desc", "")[:300],
                url=first_instance.get("uri", ""),
                method=first_instance.get("method", ""),
                evidence=first_instance.get("evidence", "")[:200],
                solution=alert.get("solution", "")[:300],
                cwe_id=int(alert.get("cweid", 0)),
                wasc_id=int(alert.get("wascid", 0)),
                instances_count=len(instances)
            ))

    return alerts


def apply_rules(alerts: list, rules_file: Optional[str]) -> list:
    """Apply rules.tsv to override alert actions."""
    if not rules_file or not os.path.exists(rules_file):
        return alerts

    rules = {}
    with open(rules_file, "r") as f:
        for line in f:
            line = line.strip()
            if not line or line.startswith("#"):
                continue
            parts = line.split("\t")
            if len(parts) >= 2:
                rules[int(parts[0])] = parts[1].upper()

    filtered = []
    for alert in alerts:
        action = rules.get(alert.alert_id, "WARN")
        if action != "IGNORE":
            alert.confidence = action  # Reuse field for action tracking
            filtered.append(alert)

    return filtered


def evaluate_quality_gate(alerts: list, threshold: str,
                          rules_file: Optional[str] = None) -> dict:
    """Evaluate quality gate based on alert risk levels."""
    threshold_level = RISK_LEVELS.get(threshold, 1)

    rules = {}
    if rules_file and os.path.exists(rules_file):
        with open(rules_file, "r") as f:
            for line in f:
                line = line.strip()
                if not line or line.startswith("#"):
                    continue
                parts = line.split("\t")
                if len(parts) >= 2:
                    rules[int(parts[0])] = parts[1].upper()

    blocking = []
    for a in alerts:
        action = rules.get(a.alert_id, "WARN")
        if action == "FAIL" and RISK_LEVELS.get(a.risk, 3) <= threshold_level:
            blocking.append(a)

    risk_counts = {}
    for a in alerts:
        risk_counts[a.risk] = risk_counts.get(a.risk, 0) + 1

    return {
        "passed": len(blocking) == 0,
        "threshold": threshold,
        "total_alerts": len(alerts),
        "blocking_count": len(blocking),
        "risk_counts": risk_counts,
        "blocking_details": [
            {"id": a.alert_id, "name": a.name, "risk": a.risk,
             "url": a.url, "cwe": a.cwe_id}
            for a in blocking[:15]
        ]
    }


def main():
    parser = argparse.ArgumentParser(description="OWASP ZAP DAST Pipeline")
    parser.add_argument("--target", required=True, help="Target URL to scan")
    parser.add_argument("--scan-type", default="baseline", choices=["baseline", "full", "api"])
    parser.add_argument("--rules-file", default=None, help="Path to rules.tsv")
    parser.add_argument("--openapi-url", default=None, help="OpenAPI spec URL for API scan")
    parser.add_argument("--output", default="dast-report.json")
    parser.add_argument("--report-dir", default="/tmp/zap")
    parser.add_argument("--risk-threshold", default="Medium", choices=["High", "Medium", "Low"])
    parser.add_argument("--fail-on-findings", action="store_true")
    args = parser.parse_args()

    print(f"[*] Running ZAP {args.scan_type} scan against {args.target}")

    zap_json = run_zap_scan(
        args.target, args.scan_type,
        rules_file=args.rules_file,
        openapi_url=args.openapi_url,
        report_dir=args.report_dir
    )

    if "error" in zap_json:
        print(f"[ERROR] {zap_json['error']}")
        sys.exit(2)

    alerts = parse_zap_results(zap_json)
    quality_gate = evaluate_quality_gate(alerts, args.risk_threshold, args.rules_file)

    report = {
        "metadata": {
            "target": args.target,
            "scan_type": args.scan_type,
            "scan_date": datetime.now(timezone.utc).isoformat()
        },
        "quality_gate": quality_gate,
        "alerts": [
            {"id": a.alert_id, "name": a.name, "risk": a.risk,
             "url": a.url, "method": a.method, "cwe": a.cwe_id,
             "solution": a.solution, "instances": a.instances_count}
            for a in sorted(alerts, key=lambda x: RISK_LEVELS.get(x.risk, 3))
        ]
    }

    output_path = os.path.abspath(args.output)
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[*] Report: {output_path}")

    if quality_gate["passed"]:
        print(f"\n[PASS] Quality gate passed. {quality_gate['total_alerts']} alerts, none blocking.")
    else:
        print(f"\n[FAIL] {quality_gate['blocking_count']} blocking alerts:")
        for d in quality_gate["blocking_details"]:
            print(f"  [{d['risk']}] {d['id']} - {d['name']} ({d['url'][:80]})")

    if args.fail_on_findings and not quality_gate["passed"]:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/integrating-sast-into-github-actions-pipeline/SKILL.md =====
---
name: integrating-sast-into-github-actions-pipeline
description: Integrates CodeQL and Semgrep SAST scanning into GitHub Actions, covering scans on pull requests/pushes, rule tuning to cut false positives, SARIF upload to GitHub Advanced Security, and merge-blocking quality gates for high-severity findings. Use when adding automated code vulnerability detection to CI, enforcing consistent SAST org-wide, or producing SOC 2/PCI DSS/NIST SSDF compliance evidence.
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- cicd
- sast
- codeql
- semgrep
- secure-sdlc
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
---

# Integrating SAST into GitHub Actions Pipeline

## When to Use

- When development teams need automated code-level vulnerability detection on every pull request
- When security teams require consistent SAST enforcement across all repositories in an organization
- When migrating from manual or periodic security reviews to continuous security testing
- When compliance frameworks (SOC 2, PCI DSS, NIST SSDF) require evidence of automated code analysis
- When multiple languages coexist in a monorepo and need unified scanning under one workflow

**Do not use** for runtime vulnerability detection (use DAST instead), for scanning third-party dependencies (use SCA tools like Snyk), or for infrastructure-as-code scanning (use Checkov or tfsec).

## Prerequisites

- GitHub repository with GitHub Actions enabled
- GitHub Advanced Security license (required for CodeQL on private repos; free for public repos)
- Semgrep account for managed rules and Semgrep App dashboard (free tier available)
- Repository code in a supported language: Python, JavaScript/TypeScript, Java, C/C++, C#, Go, Ruby, Swift, Kotlin

## Workflow

### Step 1: Configure CodeQL Analysis Workflow

Create a CodeQL workflow that runs on pull requests and on a weekly schedule to catch vulnerabilities in existing code.

```yaml
# .github/workflows/codeql-analysis.yml
name: "CodeQL Analysis"

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '30 2 * * 1'  # Weekly Monday 2:30 AM

jobs:
  analyze:
    name: Analyze (${{ matrix.language }})
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          queries: security-extended,security-and-quality

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{ matrix.language }}"
```

### Step 2: Add Semgrep Scanning for Custom Rules

Semgrep complements CodeQL with faster scans and support for custom pattern-based rules. Configure it to upload SARIF results to the same GitHub Security tab.

```yaml
# .github/workflows/semgrep.yml
name: "Semgrep SAST Scan"

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  semgrep:
    name: Semgrep Scan
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read

    container:
      image: semgrep/semgrep:latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run Semgrep
        run: |
          semgrep ci \
            --config auto \
            --config p/owasp-top-ten \
            --config p/cwe-top-25 \
            --sarif --output semgrep-results.sarif \
            --severity ERROR \
            --error
        env:
          SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}

      - name: Upload SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: semgrep-results.sarif
          category: semgrep
```

### Step 3: Create Custom Semgrep Rules for Organization Patterns

Write organization-specific rules to catch patterns unique to your codebase, such as deprecated internal APIs or insecure configuration patterns.

```yaml
# .semgrep/custom-rules.yml
rules:
  - id: hardcoded-database-url
    patterns:
      - pattern: |
          $DB_URL = "...$PROTO://...:...$PASS@..."
    message: |
      Hardcoded database connection string with credentials detected.
      Use environment variables or a secrets manager instead.
    languages: [python, javascript, typescript]
    severity: ERROR
    metadata:
      cwe: "CWE-798: Use of Hard-coded Credentials"
      owasp: "A07:2021 - Identification and Authentication Failures"

  - id: unsafe-deserialization
    patterns:
      - pattern-either:
          - pattern: pickle.loads(...)
          - pattern: yaml.load(..., Loader=yaml.Loader)
          - pattern: yaml.load(..., Loader=yaml.FullLoader)
    message: |
      Unsafe deserialization detected. Use safe alternatives to prevent
      remote code execution vulnerabilities.
    languages: [python]
    severity: ERROR
    metadata:
      cwe: "CWE-502: Deserialization of Untrusted Data"

  - id: missing-csrf-protection
    patterns:
      - pattern: |
          @app.route("...", methods=["POST"])
          def $FUNC(...):
              ...
      - pattern-not-inside: |
          @csrf.exempt
          ...
    message: "POST endpoint may lack CSRF protection."
    languages: [python]
    severity: WARNING
```

### Step 4: Establish Quality Gates with Branch Protection

Configure branch protection rules that require SAST checks to pass before merging, preventing vulnerable code from reaching production branches.

```bash
# Use GitHub CLI to set branch protection requiring SAST checks
gh api repos/{owner}/{repo}/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["Analyze (javascript)","Analyze (python)","Semgrep Scan"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1}'
```

### Step 5: Tune and Suppress False Positives

Manage false positives through CodeQL query filters and Semgrep nosemgrep annotations to maintain developer trust in scan results.

```yaml
# codeql-config.yml - Custom CodeQL configuration
name: "Custom CodeQL Config"
queries:
  - uses: security-extended
  - uses: security-and-quality
  - excludes:
      id: js/unused-local-variable
paths-ignore:
  - '**/test/**'
  - '**/tests/**'
  - '**/vendor/**'
  - '**/node_modules/**'
  - '**/*.test.js'
  - '**/*.spec.py'
```

```python
# Example: Suppressing a known false positive in Semgrep
import subprocess

def run_safe_command(cmd_list):
    # nosemgrep: python.lang.security.audit.dangerous-subprocess-use
    result = subprocess.run(cmd_list, capture_output=True, text=True, shell=False)
    return result.stdout
```

### Step 6: Aggregate and Report Findings

Use the GitHub Security Overview dashboard and configure notifications for security alerts across repositories.

```bash
# Query SARIF results via GitHub API for reporting
gh api repos/{owner}/{repo}/code-scanning/alerts \
  --jq '.[] | select(.state=="open") | {rule: .rule.id, severity: .rule.security_severity_level, file: .most_recent_instance.location.path, line: .most_recent_instance.location.start_line}'

# Count open alerts by severity
gh api repos/{owner}/{repo}/code-scanning/alerts \
  --jq '[.[] | select(.state=="open")] | group_by(.rule.security_severity_level) | map({severity: .[0].rule.security_severity_level, count: length})'
```

## Key Concepts

| Term | Definition |
|------|------------|
| SAST | Static Application Security Testing — analyzes source code without executing it to find security vulnerabilities |
| SARIF | Static Analysis Results Interchange Format — standardized JSON format for expressing results from static analysis tools |
| CodeQL | GitHub's semantic code analysis engine that treats code as data and queries it for vulnerability patterns |
| Semgrep | Lightweight static analysis tool using pattern matching to find bugs and security issues across many languages |
| Security Extended | CodeQL query suite that includes additional security queries beyond the default set for deeper analysis |
| Quality Gate | Automated checkpoint that blocks code from progressing through the pipeline unless security criteria are met |
| False Positive | A scan finding that incorrectly identifies secure code as vulnerable, requiring suppression or tuning |

## Tools & Systems

- **CodeQL**: GitHub's semantic code analysis engine with deep dataflow and taint tracking analysis
- **Semgrep**: Fast, lightweight pattern-matching SAST tool with 3000+ community rules and custom rule support
- **GitHub Advanced Security**: Platform providing code scanning, secret scanning, and dependency review in GitHub
- **SARIF Viewer**: VS Code extension for reviewing SARIF results locally during development
- **GitHub Security Overview**: Organization-level dashboard aggregating security alerts across all repositories

## Common Scenarios

### Scenario: Monorepo with Multiple Languages Needs Unified SAST

**Context**: A platform team manages a monorepo containing Python microservices, TypeScript frontends, and Go infrastructure tools. Security reviews happen manually every quarter, missing vulnerabilities between reviews.

**Approach**:
1. Configure CodeQL with a matrix strategy covering Python, JavaScript, and Go languages
2. Add Semgrep with `--config auto` to detect language automatically and apply relevant rulesets
3. Create path-based triggers so only changed language directories trigger their respective scans
4. Upload all SARIF results to GitHub Security tab with unique categories per tool and language
5. Set branch protection requiring all SAST jobs to pass before merge
6. Schedule weekly full-repository scans to catch issues in unchanged code from newly published CVE patterns

**Pitfalls**: Setting CodeQL to analyze all languages on every PR increases CI time significantly. Use path filters to trigger only relevant language scans. Semgrep's `--config auto` may enable rules that conflict with CodeQL findings, creating duplicate alerts.

### Scenario: Reducing Alert Fatigue from High False Positive Rate

**Context**: After enabling SAST, developers ignore findings because 40% are false positives, undermining the security program.

**Approach**:
1. Export all current alerts and categorize them as true positive, false positive, or informational
2. Create a custom CodeQL config excluding noisy query IDs that produce the most false positives
3. Write `.semgrepignore` patterns for test files, generated code, and vendored dependencies
4. Establish a weekly triage meeting where security and development leads review new rule additions
5. Track false positive rate as a metric and target below 15% for developer trust

**Pitfalls**: Over-suppressing rules to reduce noise can create blind spots. Always validate suppressions against the OWASP Top 10 and CWE Top 25 to ensure critical vulnerability classes remain covered.

## Output Format

```
SAST Pipeline Scan Report
==========================
Repository: org/web-application
Branch: feature/user-auth-refactor
Scan Date: 2026-02-23
Commit: a1b2c3d4

CodeQL Results:
  Language    Queries Run   Findings   Critical   High   Medium
  javascript  312           4          1          2      1
  python      287           2          0          1      1

Semgrep Results:
  Ruleset          Rules Matched   Findings   Errors   Warnings
  auto             1,847           3          1        2
  owasp-top-ten    186             2          1        1
  custom-rules     12              1          0        1

QUALITY GATE: FAILED
  Blocking findings: 2 Critical/High severity issues
  - [CRITICAL] CWE-89: SQL Injection in src/api/users.py:47
  - [HIGH] CWE-79: Cross-site Scripting in src/components/Search.tsx:123

Action Required: Fix blocking findings before merge is permitted.
```

===== END FILE =====


===== FILE: skills/integrating-sast-into-github-actions-pipeline/LICENSE =====

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


===== FILE: skills/integrating-sast-into-github-actions-pipeline/assets/template.md =====
# SAST Pipeline Configuration Templates

## GitHub Actions: Combined CodeQL + Semgrep Workflow

```yaml
# .github/workflows/sast-pipeline.yml
name: "SAST Security Pipeline"

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 3 * * 1'

concurrency:
  group: sast-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # ─────────────── CodeQL Analysis ───────────────
  codeql:
    name: CodeQL (${{ matrix.language }})
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']
    steps:
      - uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          queries: security-extended
          config-file: .github/codeql/codeql-config.yml

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform Analysis
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{ matrix.language }}"

  # ─────────────── Semgrep Scan ───────────────
  semgrep:
    name: Semgrep Scan
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    container:
      image: semgrep/semgrep:latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Semgrep
        run: |
          semgrep ci \
            --config auto \
            --config p/owasp-top-ten \
            --config p/cwe-top-25 \
            --config .semgrep/ \
            --sarif --output semgrep.sarif \
            --severity ERROR \
            --error
        env:
          SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}

      - name: Upload SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: semgrep.sarif
          category: semgrep

  # ─────────────── Quality Gate ───────────────
  security-gate:
    name: Security Quality Gate
    needs: [codeql, semgrep]
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Check SAST Results
        run: |
          if [ "${{ needs.codeql.result }}" == "failure" ] || [ "${{ needs.semgrep.result }}" == "failure" ]; then
            echo "::error::SAST security gate failed. Review findings in the Security tab."
            exit 1
          fi
          echo "Security gate passed."
```

## CodeQL Custom Configuration

```yaml
# .github/codeql/codeql-config.yml
name: "Organization CodeQL Config"

queries:
  - uses: security-extended
  - uses: security-and-quality

paths-ignore:
  - '**/test/**'
  - '**/tests/**'
  - '**/spec/**'
  - '**/vendor/**'
  - '**/node_modules/**'
  - '**/__mocks__/**'
  - '**/*.test.js'
  - '**/*.test.ts'
  - '**/*.spec.py'
  - '**/migrations/**'

query-filters:
  - exclude:
      id: js/unused-local-variable
  - exclude:
      id: py/unused-import
```

## Semgrep Ignore File

```
# .semgrepignore
# Test files
*_test.go
*_test.py
*.test.js
*.test.ts
*.spec.js
*.spec.ts
test/
tests/
__tests__/
spec/

# Generated code
*_generated.go
*.pb.go
**/generated/**

# Vendored dependencies
vendor/
node_modules/
third_party/

# Build artifacts
dist/
build/
out/
```

## Branch Protection Configuration (Terraform)

```hcl
# branch-protection.tf
resource "github_branch_protection" "main" {
  repository_id = github_repository.app.node_id
  pattern       = "main"

  required_status_checks {
    strict   = true
    contexts = [
      "CodeQL (javascript)",
      "CodeQL (python)",
      "Semgrep Scan",
      "Security Quality Gate"
    ]
  }

  required_pull_request_reviews {
    required_approving_review_count = 1
    dismiss_stale_reviews           = true
  }

  enforce_admins = true

  allows_deletions    = false
  allows_force_pushes = false
}
```

## SARIF Report Aggregation Script

```bash
#!/bin/bash
# aggregate-sarif.sh - Merge multiple SARIF files for unified upload
set -euo pipefail

OUTPUT="merged-results.sarif"
SARIF_FILES=($(find . -name "*.sarif" -type f))

if [ ${#SARIF_FILES[@]} -eq 0 ]; then
  echo "No SARIF files found"
  exit 0
fi

# Use jq to merge SARIF runs
jq -s '{
  "$schema": "https://raw.githubusercontent.com/oasis-tcs/sarif-spec/main/sarif-2.1/schema/sarif-schema-2.1.0.json",
  "version": "2.1.0",
  "runs": [.[].runs[]]
}' "${SARIF_FILES[@]}" > "$OUTPUT"

echo "Merged ${#SARIF_FILES[@]} SARIF files into $OUTPUT"
TOTAL=$(jq '[.runs[].results | length] | add' "$OUTPUT")
echo "Total findings: $TOTAL"
```

===== END FILE =====


===== FILE: skills/integrating-sast-into-github-actions-pipeline/references/api-reference.md =====
# API Reference: SAST in GitHub Actions Pipeline

## Semgrep CLI

### Installation
```bash
pip install semgrep
```

### Scan Commands
```bash
semgrep scan --config auto --json .           # Auto-detect rules
semgrep scan --config p/owasp-top-ten --json . # OWASP rules
semgrep scan --config p/ci --sarif .           # CI-optimized rules
```

### JSON Output Structure
```json
{"results": [{"check_id": "rule-id", "path": "file.py",
  "start": {"line": 10}, "extra": {"severity": "ERROR",
  "message": "...", "metadata": {"cwe": ["CWE-89"], "owasp": ["A03"]}}}]}
```

### Severity Levels
| Level | Action |
|-------|--------|
| ERROR | Block merge |
| WARNING | Require review |
| INFO | Advisory only |

## GitHub Actions Integration

### Semgrep Action
```yaml
- uses: returntocorp/semgrep-action@v1
  with:
    config: auto
    generateSarif: "1"
```

### SARIF Upload
```yaml
- uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: semgrep.sarif
```

### SARIF 2.1.0 Schema
| Field | Description |
|-------|-------------|
| `runs[].tool.driver.name` | Scanner name |
| `runs[].tool.driver.rules` | Rule definitions |
| `runs[].results` | Finding instances |
| `results[].ruleId` | Matching rule ID |
| `results[].level` | `error`, `warning`, `note` |

## References
- Semgrep: https://semgrep.dev/docs/
- GitHub Code Scanning: https://docs.github.com/en/code-security/code-scanning
- SARIF spec: https://docs.oasis-open.org/sarif/sarif/v2.1.0/

===== END FILE =====


===== FILE: skills/integrating-sast-into-github-actions-pipeline/references/standards.md =====
# Standards Reference: SAST in GitHub Actions

## OWASP SAMM - Verification: Security Testing

### Maturity Level 1
- Perform automated SAST scanning with default rulesets on all application code
- Results are visible to development teams through IDE or CI/CD integration

### Maturity Level 2
- Customize SAST rules to reduce false positives below 20%
- Track and triage all findings with defined SLAs per severity
- Integrate SAST results into a centralized vulnerability management system

### Maturity Level 3
- Correlate SAST findings with DAST and SCA results for comprehensive coverage
- Measure and improve detection accuracy through benchmarking against known vulnerabilities
- Custom rules cover organization-specific vulnerability patterns and deprecated APIs

## NIST SSDF (SP 800-218) - Produce Well-Secured Software

### PW.7: Review and Analyze Code
- PW.7.1: Determine whether SAST tools should be used and select appropriate tools
- PW.7.2: Use SAST tools to analyze source code and identify vulnerabilities
- Configure tools to analyze code for compliance with secure coding standards

### PW.8: Test Executable Code
- Integration of SAST into CI/CD ensures code is tested before deployment
- Findings are tracked and remediated according to organizational policy

## CIS Software Supply Chain Security Guide

### Source Code (SC) Controls
- SC-2: Enforce branch protection requiring SAST checks to pass
- SC-3: Require code review in addition to automated scanning
- SC-4: Automate security testing in the build pipeline

### Build (BD) Controls
- BD-1: Define and enforce security requirements for build processes
- BD-2: Integrate multiple security testing tools for defense in depth

## OWASP Top 10 Coverage Matrix

| OWASP Category | CodeQL | Semgrep | Combined |
|----------------|--------|---------|----------|
| A01: Broken Access Control | Partial | Yes | Yes |
| A02: Cryptographic Failures | Yes | Yes | Yes |
| A03: Injection | Yes | Yes | Yes |
| A04: Insecure Design | No | Partial | Partial |
| A05: Security Misconfiguration | Partial | Yes | Yes |
| A06: Vulnerable Components | No | No | No (Use SCA) |
| A07: Auth Failures | Yes | Yes | Yes |
| A08: Software Integrity | No | Partial | Partial |
| A09: Logging Failures | Partial | Yes | Yes |
| A10: SSRF | Yes | Yes | Yes |

## PCI DSS v4.0 Mapping

- Requirement 6.2.4: Software engineering techniques or automated methods prevent or mitigate common software attacks
- Requirement 6.3.2: An inventory of bespoke and custom software and third-party software components is maintained
- Requirement 6.5.4: SAST tools are run as part of the software development lifecycle

## SOC 2 Trust Service Criteria

- CC7.1: Deploy detection and monitoring mechanisms for anomalies indicative of actual or attempted attacks
- CC8.1: Authorize, design, develop or acquire, configure, document, test, approve, and implement changes to infrastructure, data, software, and procedures

===== END FILE =====


===== FILE: skills/integrating-sast-into-github-actions-pipeline/references/workflows.md =====
# Workflow Reference: SAST in GitHub Actions Pipeline

## End-to-End SAST Integration Workflow

```
Developer Push/PR
       │
       ▼
┌──────────────────┐
│ GitHub Actions    │
│ Trigger           │
└──────┬───────────┘
       │
       ├──────────────────────┐
       ▼                      ▼
┌──────────────┐    ┌──────────────┐
│ CodeQL Init  │    │ Semgrep CI   │
│ + Autobuild  │    │ + Custom     │
│ + Analyze    │    │   Rules      │
└──────┬───────┘    └──────┬───────┘
       │                    │
       ▼                    ▼
┌──────────────┐    ┌──────────────┐
│ SARIF Upload │    │ SARIF Upload │
│ (CodeQL)     │    │ (Semgrep)    │
└──────┬───────┘    └──────┬───────┘
       │                    │
       └──────────┬─────────┘
                  ▼
       ┌──────────────────┐
       │ GitHub Security  │
       │ Tab / Dashboard  │
       └──────┬───────────┘
              │
              ▼
       ┌──────────────────┐
       │ Branch Protection│
       │ Quality Gate     │
       └──────┬───────────┘
              │
    ┌─────────┴──────────┐
    ▼                    ▼
 PASS: Merge          FAIL: Block
 Permitted            + Notify Dev
```

## CodeQL Analysis Deep Dive

### Database Creation Phase
1. CodeQL extracts source code into a relational database
2. For compiled languages (Java, C++, C#, Go), the build process is intercepted
3. For interpreted languages (Python, JavaScript, Ruby), source files are parsed directly
4. The database captures the full AST, data flow, and control flow of the program

### Query Execution Phase
1. Security queries analyze the database for known vulnerability patterns
2. Taint tracking follows data from untrusted sources to dangerous sinks
3. Dataflow analysis tracks variable assignments across method boundaries
4. Results are deduplicated and ranked by confidence and severity

### Query Suites
- **default**: Core security queries with high precision and low false positive rate
- **security-extended**: Additional queries covering more vulnerability types
- **security-and-quality**: All security queries plus code quality checks

## Semgrep Rule Authoring Process

### Rule Development Lifecycle
1. Identify a vulnerability pattern from a recent security incident or code review
2. Write the pattern using Semgrep syntax with `pattern`, `pattern-either`, `pattern-not`
3. Test the rule against known vulnerable and safe code samples
4. Add metadata: CWE ID, OWASP category, severity, remediation guidance
5. Deploy via `.semgrep/` directory or Semgrep App registry
6. Monitor false positive rate and refine patterns

### Pattern Operators Reference
| Operator | Purpose |
|----------|---------|
| `pattern` | Match a single code pattern |
| `pattern-either` | Match any of multiple patterns (OR) |
| `pattern-not` | Exclude specific patterns from matches |
| `pattern-inside` | Match only within a containing pattern |
| `pattern-not-inside` | Exclude matches within a containing pattern |
| `metavariable-regex` | Constrain metavariable values with regex |
| `metavariable-comparison` | Compare metavariable values numerically |

## SARIF Processing Pipeline

### SARIF Structure
```json
{
  "$schema": "https://raw.githubusercontent.com/oasis-tcs/sarif-spec/main/sarif-2.1/schema/sarif-schema-2.1.0.json",
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "Semgrep",
          "rules": []
        }
      },
      "results": [
        {
          "ruleId": "hardcoded-database-url",
          "level": "error",
          "message": { "text": "..." },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": { "uri": "src/config.py" },
                "region": { "startLine": 42 }
              }
            }
          ]
        }
      ]
    }
  ]
}
```

## Triage and Remediation Workflow

### Severity-Based SLA
| Severity | Triage SLA | Remediation SLA | Escalation |
|----------|-----------|-----------------|------------|
| Critical | 1 business day | 3 business days | Security Lead + VP Eng |
| High | 3 business days | 10 business days | Security Lead |
| Medium | 5 business days | 30 business days | Team Lead |
| Low | 10 business days | 90 business days | Backlog |

### Finding States
1. **Open**: New finding not yet reviewed
2. **Confirmed**: Finding validated as true positive
3. **False Positive**: Finding dismissed with justification
4. **Fixed**: Remediation committed and verified by rescan
5. **Won't Fix**: Accepted risk with documented justification and risk owner

===== END FILE =====


===== FILE: skills/integrating-sast-into-github-actions-pipeline/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for running Semgrep SAST scans and generating SARIF for GitHub Advanced Security."""

import subprocess
import json
import argparse
import sys
import os


def run_semgrep_scan(target_dir, config="auto", output_format="json"):
    """Run Semgrep SAST scan on a code directory."""
    print(f"[*] Running Semgrep scan on {target_dir} (config={config})...")
    cmd = ["semgrep", "scan", "--config", config, "--json", "--no-git", target_dir]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
    if result.returncode not in (0, 1):
        print(f"  [-] Semgrep error: {result.stderr[:300]}")
        return {}
    data = json.loads(result.stdout) if result.stdout else {}
    results = data.get("results", [])
    errors = data.get("errors", [])
    print(f"  Findings: {len(results)}, Errors: {len(errors)}")
    return data


def parse_semgrep_results(scan_data):
    """Parse Semgrep findings into structured format."""
    results = scan_data.get("results", [])
    findings = []
    severity_counts = {}
    for r in results:
        severity = r.get("extra", {}).get("severity", "WARNING")
        finding = {
            "rule_id": r.get("check_id", "unknown"),
            "message": r.get("extra", {}).get("message", ""),
            "severity": severity,
            "file": r.get("path", ""),
            "line": r.get("start", {}).get("line", 0),
            "cwe": r.get("extra", {}).get("metadata", {}).get("cwe", []),
            "owasp": r.get("extra", {}).get("metadata", {}).get("owasp", []),
        }
        findings.append(finding)
        severity_counts[severity] = severity_counts.get(severity, 0) + 1
    print(f"\n[*] Severity breakdown:")
    for sev, count in sorted(severity_counts.items()):
        print(f"  {sev}: {count}")
    return findings


def generate_sarif(findings, output_path):
    """Convert Semgrep findings to SARIF 2.1.0 format."""
    rules, results = [], []
    seen_rules = set()
    for f in findings:
        rid = f["rule_id"]
        if rid not in seen_rules:
            rules.append({"id": rid, "shortDescription": {"text": f["message"][:200]},
                           "defaultConfiguration": {"level": "warning"}})
            seen_rules.add(rid)
        results.append({
            "ruleId": rid, "message": {"text": f["message"][:500]},
            "level": "error" if f["severity"] == "ERROR" else "warning",
            "locations": [{"physicalLocation": {
                "artifactLocation": {"uri": f["file"]},
                "region": {"startLine": f["line"]}}}],
        })
    sarif = {"$schema": "https://json.schemastore.org/sarif-2.1.0.json", "version": "2.1.0",
             "runs": [{"tool": {"driver": {"name": "Semgrep", "rules": rules}},
                        "results": results}]}
    with open(output_path, "w") as f:
        json.dump(sarif, f, indent=2)
    print(f"[*] SARIF report: {output_path}")


def apply_quality_gate(findings, fail_on="ERROR"):
    """Apply quality gate based on severity threshold."""
    severity_order = {"INFO": 0, "WARNING": 1, "ERROR": 2}
    threshold = severity_order.get(fail_on, 2)
    blocking = [f for f in findings if severity_order.get(f["severity"], 0) >= threshold]
    if blocking:
        print(f"\n[!] QUALITY GATE FAILED: {len(blocking)} findings at {fail_on}+")
        for b in blocking[:5]:
            print(f"  {b['file']}:{b['line']} - {b['rule_id']}")
        return False
    print(f"\n[+] Quality gate passed (threshold: {fail_on})")
    return True


def generate_github_actions_workflow(config="auto"):
    """Generate a GitHub Actions SAST workflow YAML."""
    workflow = f"""name: SAST Scan
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
jobs:
  semgrep:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: returntocorp/semgrep-action@v1
        with:
          config: {config}
          generateSarif: "1"
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: semgrep.sarif
        if: always()
"""
    print("[*] Generated GitHub Actions workflow:")
    print(workflow)
    return workflow


def main():
    parser = argparse.ArgumentParser(description="SAST GitHub Actions Pipeline Agent")
    parser.add_argument("action", choices=["scan", "parse", "sarif", "gate", "gen-workflow"])
    parser.add_argument("--target", default=".", help="Directory to scan")
    parser.add_argument("--config", default="auto", help="Semgrep config (auto, p/ci, p/owasp)")
    parser.add_argument("--report", help="Existing Semgrep JSON report")
    parser.add_argument("--fail-on", default="ERROR", choices=["INFO", "WARNING", "ERROR"])
    parser.add_argument("-o", "--output", default=".")
    args = parser.parse_args()

    os.makedirs(args.output, exist_ok=True)
    if args.action == "scan":
        data = run_semgrep_scan(args.target, args.config)
        findings = parse_semgrep_results(data)
        generate_sarif(findings, os.path.join(args.output, "semgrep.sarif"))
        apply_quality_gate(findings, args.fail_on)
    elif args.action == "parse":
        with open(args.report) as f:
            data = json.load(f)
        parse_semgrep_results(data)
    elif args.action == "sarif":
        with open(args.report) as f:
            data = json.load(f)
        findings = parse_semgrep_results(data)
        generate_sarif(findings, os.path.join(args.output, "semgrep.sarif"))
    elif args.action == "gate":
        with open(args.report) as f:
            data = json.load(f)
        findings = parse_semgrep_results(data)
        passed = apply_quality_gate(findings, args.fail_on)
        sys.exit(0 if passed else 1)
    elif args.action == "gen-workflow":
        generate_github_actions_workflow(args.config)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/integrating-sast-into-github-actions-pipeline/scripts/process.py =====
#!/usr/bin/env python3
"""
SAST Pipeline Orchestration Script

Runs CodeQL and Semgrep scans, aggregates SARIF results, evaluates quality gates,
and produces a consolidated report. Designed to be invoked from GitHub Actions
or any CI/CD platform.

Usage:
    python process.py --repo-path /path/to/repo --output report.json
    python process.py --repo-path . --severity-threshold high --fail-on-findings
"""

import argparse
import json
import os
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime, timezone
from pathlib import Path
from typing import Optional


@dataclass
class ScanFinding:
    rule_id: str
    severity: str
    message: str
    file_path: str
    start_line: int
    end_line: int
    tool: str
    cwe: str = ""
    owasp: str = ""
    fingerprint: str = ""


@dataclass
class ScanResult:
    tool: str
    findings: list = field(default_factory=list)
    rules_evaluated: int = 0
    scan_duration_seconds: float = 0.0
    exit_code: int = 0
    error_message: str = ""


SEVERITY_ORDER = {"critical": 0, "high": 1, "medium": 2, "low": 3, "note": 4, "none": 5}


def run_semgrep(repo_path: str, config: str = "auto", extra_configs: Optional[list] = None) -> ScanResult:
    """Run Semgrep scan and return structured results."""
    result = ScanResult(tool="semgrep")
    sarif_output = os.path.join(repo_path, "semgrep-results.sarif")

    cmd = [
        "semgrep", "ci",
        "--config", config,
        "--sarif",
        "--output", sarif_output,
        "--json",
        "--quiet"
    ]

    if extra_configs:
        for cfg in extra_configs:
            cmd.extend(["--config", cfg])

    start_time = datetime.now(timezone.utc)

    try:
        proc = subprocess.run(
            cmd,
            cwd=repo_path,
            capture_output=True,
            text=True,
            timeout=600
        )
        result.exit_code = proc.returncode

        if proc.returncode not in (0, 1):
            result.error_message = proc.stderr[:500]
            return result

    except subprocess.TimeoutExpired:
        result.error_message = "Semgrep scan timed out after 600 seconds"
        result.exit_code = -1
        return result
    except FileNotFoundError:
        result.error_message = "semgrep binary not found. Install with: pip install semgrep"
        result.exit_code = -1
        return result

    result.scan_duration_seconds = (datetime.now(timezone.utc) - start_time).total_seconds()

    if os.path.exists(sarif_output):
        result.findings = parse_sarif(sarif_output, "semgrep")
        with open(sarif_output, "r") as f:
            sarif_data = json.load(f)
            for run in sarif_data.get("runs", []):
                result.rules_evaluated = len(run.get("tool", {}).get("driver", {}).get("rules", []))

    return result


def run_codeql_query(repo_path: str, language: str, database_path: str) -> ScanResult:
    """Run CodeQL analysis on a pre-created database and return structured results."""
    result = ScanResult(tool=f"codeql-{language}")
    sarif_output = os.path.join(repo_path, f"codeql-{language}-results.sarif")

    cmd = [
        "codeql", "database", "analyze",
        database_path,
        f"codeql/{language}-queries:codeql-suites/{language}-security-extended.qls",
        "--format=sarifv2.1.0",
        f"--output={sarif_output}",
        "--threads=0"
    ]

    start_time = datetime.now(timezone.utc)

    try:
        proc = subprocess.run(
            cmd,
            cwd=repo_path,
            capture_output=True,
            text=True,
            timeout=1200
        )
        result.exit_code = proc.returncode

        if proc.returncode != 0:
            result.error_message = proc.stderr[:500]

    except subprocess.TimeoutExpired:
        result.error_message = "CodeQL analysis timed out after 1200 seconds"
        result.exit_code = -1
        return result
    except FileNotFoundError:
        result.error_message = "codeql binary not found. Install from https://github.com/github/codeql-cli-binaries"
        result.exit_code = -1
        return result

    result.scan_duration_seconds = (datetime.now(timezone.utc) - start_time).total_seconds()

    if os.path.exists(sarif_output):
        result.findings = parse_sarif(sarif_output, f"codeql-{language}")
        with open(sarif_output, "r") as f:
            sarif_data = json.load(f)
            for run in sarif_data.get("runs", []):
                result.rules_evaluated = len(run.get("tool", {}).get("driver", {}).get("rules", []))

    return result


def parse_sarif(sarif_path: str, tool_name: str) -> list:
    """Parse a SARIF file and extract findings as ScanFinding objects."""
    findings = []

    with open(sarif_path, "r") as f:
        sarif_data = json.load(f)

    for run in sarif_data.get("runs", []):
        rules_map = {}
        for rule in run.get("tool", {}).get("driver", {}).get("rules", []):
            rule_id = rule.get("id", "")
            properties = rule.get("properties", {})
            cwe_tags = [t for t in properties.get("tags", []) if t.startswith("CWE")]
            owasp_tags = [t for t in properties.get("tags", []) if "owasp" in t.lower()]
            rules_map[rule_id] = {
                "cwe": cwe_tags[0] if cwe_tags else "",
                "owasp": owasp_tags[0] if owasp_tags else "",
                "severity": rule.get("defaultConfiguration", {}).get("level", "warning")
            }

        for result in run.get("results", []):
            rule_id = result.get("ruleId", "unknown")
            rule_info = rules_map.get(rule_id, {})

            level = result.get("level", rule_info.get("severity", "warning"))
            severity_map = {"error": "high", "warning": "medium", "note": "low", "none": "none"}
            severity = severity_map.get(level, "medium")

            security_severity = None
            for rule in run.get("tool", {}).get("driver", {}).get("rules", []):
                if rule.get("id") == rule_id:
                    security_severity = rule.get("properties", {}).get("security-severity")
                    break

            if security_severity:
                score = float(security_severity)
                if score >= 9.0:
                    severity = "critical"
                elif score >= 7.0:
                    severity = "high"
                elif score >= 4.0:
                    severity = "medium"
                else:
                    severity = "low"

            locations = result.get("locations", [{}])
            physical = locations[0].get("physicalLocation", {}) if locations else {}
            artifact = physical.get("artifactLocation", {})
            region = physical.get("region", {})

            findings.append(ScanFinding(
                rule_id=rule_id,
                severity=severity,
                message=result.get("message", {}).get("text", ""),
                file_path=artifact.get("uri", "unknown"),
                start_line=region.get("startLine", 0),
                end_line=region.get("endLine", region.get("startLine", 0)),
                tool=tool_name,
                cwe=rule_info.get("cwe", ""),
                owasp=rule_info.get("owasp", ""),
                fingerprint=str(result.get("fingerprints", {}).get("primaryLocationLineHash", ""))
            ))

    return findings


def evaluate_quality_gate(findings: list, severity_threshold: str) -> dict:
    """Evaluate quality gate based on finding severities."""
    threshold_level = SEVERITY_ORDER.get(severity_threshold.lower(), 1)

    blocking_findings = [
        f for f in findings
        if SEVERITY_ORDER.get(f.severity.lower(), 5) <= threshold_level
    ]

    severity_counts = {}
    for f in findings:
        sev = f.severity.lower()
        severity_counts[sev] = severity_counts.get(sev, 0) + 1

    return {
        "passed": len(blocking_findings) == 0,
        "threshold": severity_threshold,
        "total_findings": len(findings),
        "blocking_findings": len(blocking_findings),
        "severity_counts": severity_counts,
        "blocking_details": [
            {
                "rule_id": f.rule_id,
                "severity": f.severity,
                "file": f.file_path,
                "line": f.start_line,
                "tool": f.tool,
                "message": f.message[:200]
            }
            for f in blocking_findings
        ]
    }


def generate_report(scan_results: list, quality_gate: dict, repo_path: str) -> dict:
    """Generate a consolidated SAST report."""
    all_findings = []
    for sr in scan_results:
        all_findings.extend(sr.findings)

    cwe_counts = {}
    for f in all_findings:
        if f.cwe:
            cwe_counts[f.cwe] = cwe_counts.get(f.cwe, 0) + 1

    report = {
        "report_metadata": {
            "generated_at": datetime.now(timezone.utc).isoformat(),
            "repository": repo_path,
            "report_version": "1.0.0"
        },
        "scan_summary": [
            {
                "tool": sr.tool,
                "findings_count": len(sr.findings),
                "rules_evaluated": sr.rules_evaluated,
                "duration_seconds": sr.scan_duration_seconds,
                "status": "success" if sr.exit_code in (0, 1) else "error",
                "error": sr.error_message
            }
            for sr in scan_results
        ],
        "quality_gate": quality_gate,
        "top_cwes": sorted(cwe_counts.items(), key=lambda x: x[1], reverse=True)[:10],
        "findings": [
            {
                "rule_id": f.rule_id,
                "severity": f.severity,
                "tool": f.tool,
                "file": f.file_path,
                "line": f.start_line,
                "cwe": f.cwe,
                "owasp": f.owasp,
                "message": f.message[:300]
            }
            for f in sorted(all_findings, key=lambda x: SEVERITY_ORDER.get(x.severity.lower(), 5))
        ]
    }

    return report


def main():
    parser = argparse.ArgumentParser(description="SAST Pipeline Orchestration")
    parser.add_argument("--repo-path", required=True, help="Path to the repository to scan")
    parser.add_argument("--output", default="sast-report.json", help="Output report file path")
    parser.add_argument("--severity-threshold", default="high",
                        choices=["critical", "high", "medium", "low"],
                        help="Minimum severity to block pipeline")
    parser.add_argument("--fail-on-findings", action="store_true",
                        help="Exit with non-zero code if quality gate fails")
    parser.add_argument("--semgrep-config", default="auto",
                        help="Semgrep configuration (default: auto)")
    parser.add_argument("--semgrep-extra-configs", nargs="*",
                        help="Additional Semgrep config paths")
    parser.add_argument("--skip-semgrep", action="store_true", help="Skip Semgrep scan")
    parser.add_argument("--skip-codeql", action="store_true", help="Skip CodeQL scan")
    parser.add_argument("--codeql-language", default=None, help="Language for CodeQL analysis")
    parser.add_argument("--codeql-db-path", default=None, help="Path to CodeQL database")
    parser.add_argument("--sarif-only", nargs="*",
                        help="Only parse existing SARIF files instead of running scans")
    args = parser.parse_args()

    repo_path = os.path.abspath(args.repo_path)
    scan_results = []

    if args.sarif_only:
        for sarif_file in args.sarif_only:
            tool_name = Path(sarif_file).stem
            findings = parse_sarif(sarif_file, tool_name)
            sr = ScanResult(tool=tool_name, findings=findings)
            scan_results.append(sr)
    else:
        if not args.skip_semgrep:
            print("[*] Running Semgrep scan...")
            semgrep_result = run_semgrep(
                repo_path,
                config=args.semgrep_config,
                extra_configs=args.semgrep_extra_configs
            )
            scan_results.append(semgrep_result)
            print(f"    Found {len(semgrep_result.findings)} findings in {semgrep_result.scan_duration_seconds:.1f}s")

            if semgrep_result.error_message:
                print(f"    Warning: {semgrep_result.error_message}")

        if not args.skip_codeql and args.codeql_language and args.codeql_db_path:
            print(f"[*] Running CodeQL analysis for {args.codeql_language}...")
            codeql_result = run_codeql_query(repo_path, args.codeql_language, args.codeql_db_path)
            scan_results.append(codeql_result)
            print(f"    Found {len(codeql_result.findings)} findings in {codeql_result.scan_duration_seconds:.1f}s")

    all_findings = []
    for sr in scan_results:
        all_findings.extend(sr.findings)

    quality_gate = evaluate_quality_gate(all_findings, args.severity_threshold)

    report = generate_report(scan_results, quality_gate, repo_path)

    output_path = os.path.abspath(args.output)
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)
    print(f"\n[*] Report written to {output_path}")

    if quality_gate["passed"]:
        print(f"[PASS] Quality gate passed. {quality_gate['total_findings']} findings, none blocking.")
    else:
        print(f"[FAIL] Quality gate failed. {quality_gate['blocking_findings']} blocking findings:")
        for detail in quality_gate["blocking_details"]:
            print(f"  - [{detail['severity'].upper()}] {detail['rule_id']} in {detail['file']}:{detail['line']}")

    if args.fail_on_findings and not quality_gate["passed"]:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-container-image-hardening/SKILL.md =====
---
name: performing-container-image-hardening
description: 'Harden container images by minimizing attack surface, stripping unnecessary
  packages, implementing multi-stage builds, configuring non-root users, and applying
  CIS Docker Benchmark recommendations to produce secure, production-ready images.
  Use when building production container images, when compliance requires CIS Docker
  Benchmark adherence, or when shrinking image size to reduce vulnerability exposure
  from unused packages.

  '
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- cicd
- container-hardening
- docker
- cis-benchmark
- secure-sdlc
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
- T1610
- T1611
---

# Performing Container Image Hardening

## When to Use

- When building production container images that need minimal attack surface
- When compliance requires CIS Docker Benchmark adherence for container configurations
- When reducing image size to minimize vulnerability exposure from unused packages
- When implementing defense-in-depth for containerized workloads
- When migrating from fat base images to distroless or minimal images

**Do not use** for runtime container security monitoring (use Falco), for host-level Docker daemon hardening (use CIS Docker Benchmark host checks), or for container orchestration security (use Kubernetes security scanning).

## Prerequisites

- Docker or BuildKit for multi-stage builds
- Base image options: distroless, Alpine, slim, or scratch
- Container scanning tool (Trivy) for validation
- CIS Docker Benchmark reference

## Workflow

### Step 1: Use Multi-Stage Builds to Minimize Image Size

```dockerfile
# Build stage with all dependencies
FROM python:3.12-bookworm AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt
COPY src/ ./src/
RUN python -m compileall src/

# Production stage with minimal base
FROM python:3.12-slim-bookworm AS production
RUN apt-get update && \
    apt-get install -y --no-install-recommends libpq5 && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false

COPY --from=builder /install /usr/local
COPY --from=builder /build/src /app/src

RUN groupadd -r appuser && useradd -r -g appuser -d /app -s /sbin/nologin appuser
RUN chown -R appuser:appuser /app

USER appuser
WORKDIR /app

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8080/health')" || exit 1

EXPOSE 8080
ENTRYPOINT ["python", "-m", "src.main"]
```

### Step 2: Use Distroless Base Images

```dockerfile
# Go application with distroless
FROM golang:1.22 AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o /server .

FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=builder /server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```

### Step 3: Remove Unnecessary Components

```dockerfile
# Hardened image checklist
FROM ubuntu:24.04 AS base

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      libssl3 && \
    # Remove package manager to prevent runtime package installation
    apt-get purge -y --auto-remove apt dpkg && \
    rm -rf /var/lib/apt/lists/* \
           /var/cache/apt/* \
           /tmp/* \
           /var/tmp/* \
           /usr/share/doc/* \
           /usr/share/man/* \
           /usr/share/info/* \
           /root/.cache

# Remove shells if not needed
RUN rm -f /bin/sh /bin/bash /usr/bin/sh 2>/dev/null || true

# Remove setuid/setgid binaries
RUN find / -perm /6000 -type f -exec chmod a-s {} + 2>/dev/null || true
```

### Step 4: Configure Read-Only Filesystem

```yaml
# Kubernetes deployment with read-only root filesystem
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hardened-app
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: app
          image: app:hardened
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: cache
              mountPath: /app/cache
      volumes:
        - name: tmp
          emptyDir:
            sizeLimit: 100Mi
        - name: cache
          emptyDir:
            sizeLimit: 50Mi
```

### Step 5: Pin Base Image by Digest

```dockerfile
# Pin to exact image digest for reproducibility
FROM python:3.12-slim-bookworm@sha256:abcdef1234567890 AS production
# This ensures the exact same base image is used every time
```

### Step 6: Validate Hardening with Automated Scanning

```bash
# Scan hardened image with Trivy
trivy image --severity HIGH,CRITICAL hardened-app:latest

# Check CIS Docker Benchmark compliance
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/docker-bench-security

# Verify no root processes
docker run --rm hardened-app:latest whoami
# Expected: appuser (NOT root)

# Verify read-only filesystem
docker run --rm hardened-app:latest touch /test 2>&1
# Expected: Read-only file system error
```

## Key Concepts

| Term | Definition |
|------|------------|
| Multi-Stage Build | Docker build technique using multiple FROM stages to separate build and runtime, reducing final image size |
| Distroless | Google-maintained minimal container images containing only the application and runtime dependencies |
| Non-Root User | Running container processes as unprivileged user to limit impact of container escape exploits |
| Read-Only Root | Mounting the container root filesystem as read-only to prevent runtime modification |
| Image Digest | SHA256 hash uniquely identifying an exact image version, more precise than mutable tags |
| Scratch Image | Empty Docker base image used for statically compiled binaries requiring no OS |
| Security Context | Kubernetes pod/container-level security settings controlling privileges, filesystem, and capabilities |

## Tools & Systems

- **Docker BuildKit**: Advanced Docker build engine supporting multi-stage builds and build secrets
- **Distroless Images**: Google's minimal container base images (static, base, java, python, nodejs)
- **docker-bench-security**: Script checking CIS Docker Benchmark compliance
- **Trivy**: Container image vulnerability and misconfiguration scanner
- **Hadolint**: Dockerfile linter enforcing best practices

## Common Scenarios

### Scenario: Reducing a 1.2GB Python Image to Under 150MB

**Context**: A data science team uses `python:3.12` as base image (1.2GB) with scientific computing packages. The image has 200+ known CVEs from unnecessary system packages.

**Approach**:
1. Switch to `python:3.12-slim-bookworm` as base (150MB) and install only required system libraries
2. Use multi-stage build: compile C extensions in builder stage, copy wheels to production
3. Pin numpy, pandas, and scipy to pre-built wheels to avoid build dependencies in production
4. Remove pip, setuptools, and wheel from the final image
5. Create non-root user and set filesystem permissions
6. Validate with Trivy: expect CVE count to drop from 200+ to under 20

**Pitfalls**: Some Python packages require shared libraries at runtime (libgomp, libstdc++). Test the application thoroughly after removing system packages. Alpine-based images use musl libc which can cause compatibility issues with numpy and pandas.

## Output Format

```
Container Image Hardening Report
==================================
Image: app:hardened
Base: python:3.12-slim-bookworm
Date: 2026-02-23

SIZE COMPARISON:
  Before hardening: 1,247 MB (python:3.12)
  After hardening:  143 MB  (python:3.12-slim + multi-stage)
  Reduction: 88.5%

SECURITY CHECKS:
  [PASS] Non-root user configured (appuser:1000)
  [PASS] HEALTHCHECK instruction present
  [PASS] No setuid/setgid binaries found
  [PASS] Package manager removed
  [PASS] Base image pinned by digest
  [PASS] No shell access (/bin/sh removed)
  [WARN] /tmp writable (emptyDir mounted)

VULNERABILITY COMPARISON:
  Before: 234 CVEs (12 Critical, 45 High)
  After:  18 CVEs (0 Critical, 3 High)
  Reduction: 92.3%
```

===== END FILE =====


===== FILE: skills/performing-container-image-hardening/LICENSE =====

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


===== FILE: skills/performing-container-image-hardening/assets/template.md =====
# Container Image Hardening Templates

## Hardened Python Dockerfile

```dockerfile
FROM python:3.12-slim-bookworm AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

FROM python:3.12-slim-bookworm AS production
RUN apt-get update && apt-get install -y --no-install-recommends libpq5 && \
    rm -rf /var/lib/apt/lists/* && \
    groupadd -r appuser && useradd -r -g appuser -s /sbin/nologin appuser && \
    find / -perm /6000 -type f -exec chmod a-s {} + 2>/dev/null || true
COPY --from=builder /install /usr/local
COPY --chown=appuser:appuser src/ /app/src/
USER appuser
WORKDIR /app
HEALTHCHECK --interval=30s --timeout=3s CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8080/health')" || exit 1
EXPOSE 8080
ENTRYPOINT ["python", "-m", "src.main"]
```

## Hardened Go Dockerfile (Distroless)

```dockerfile
FROM golang:1.22 AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o /server .

FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=builder /server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```

## Hadolint Configuration

```yaml
# .hadolint.yaml
ignored:
  - DL3008  # Pin versions in apt-get (use --no-install-recommends instead)
trustedRegistries:
  - docker.io
  - gcr.io
  - ghcr.io
```

===== END FILE =====


===== FILE: skills/performing-container-image-hardening/references/api-reference.md =====
# API Reference: Container Image Hardening Audit

## Libraries Used

| Library | Purpose |
|---------|---------|
| `subprocess` | Execute Trivy, Docker, and hadolint CLI commands |
| `json` | Parse vulnerability scan and inspection results |
| `re` | Analyze Dockerfile instructions |
| `pathlib` | Handle Dockerfile and image paths |

## Installation

```bash
# Trivy vulnerability scanner
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# Hadolint Dockerfile linter
wget -O /usr/local/bin/hadolint https://github.com/hadolint/hadolint/releases/latest/download/hadolint-Linux-x86_64
chmod +x /usr/local/bin/hadolint

# Docker CLI (already installed in most environments)
```

## Trivy Image Scanning

### Scan Image for Vulnerabilities
```python
import subprocess
import json

def scan_image(image_name, severity="CRITICAL,HIGH"):
    cmd = [
        "trivy", "image",
        "--format", "json",
        "--severity", severity,
        "--exit-code", "0",
        image_name,
    ]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    return json.loads(result.stdout) if result.stdout else {}
```

### Scan for Secrets in Image
```python
def scan_secrets(image_name):
    cmd = [
        "trivy", "image",
        "--format", "json",
        "--scanners", "secret",
        image_name,
    ]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    return json.loads(result.stdout) if result.stdout else {}
```

### Scan for Misconfigurations
```python
def scan_misconfig(image_name):
    cmd = [
        "trivy", "image",
        "--format", "json",
        "--scanners", "misconfig",
        image_name,
    ]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    return json.loads(result.stdout) if result.stdout else {}
```

## Docker Image Inspection

### Inspect Image Metadata
```python
def inspect_image(image_name):
    cmd = ["docker", "inspect", image_name]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
    data = json.loads(result.stdout)[0]
    config = data.get("Config", {})
    return {
        "image": image_name,
        "user": config.get("User", "root"),
        "exposed_ports": list(config.get("ExposedPorts", {}).keys()),
        "env_vars": config.get("Env", []),
        "entrypoint": config.get("Entrypoint"),
        "cmd": config.get("Cmd"),
        "labels": config.get("Labels", {}),
        "layers": len(data.get("RootFS", {}).get("Layers", [])),
        "size_mb": round(data.get("Size", 0) / 1048576, 1),
    }
```

### Check for Root User
```python
def check_non_root(image_name):
    inspection = inspect_image(image_name)
    user = inspection["user"]
    return {
        "image": image_name,
        "runs_as_root": user in ("", "root", "0"),
        "user": user or "root (default)",
        "severity": "high" if user in ("", "root", "0") else "pass",
    }
```

## Hadolint Dockerfile Linting

```python
def lint_dockerfile(dockerfile_path):
    cmd = [
        "hadolint",
        "--format", "json",
        str(dockerfile_path),
    ]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
    findings = json.loads(result.stdout) if result.stdout else []
    return [
        {
            "line": f["line"],
            "code": f["code"],
            "level": f["level"],
            "message": f["message"],
        }
        for f in findings
    ]
```

## Hardening Checks

### Common Dockerfile Issues
```python
def audit_dockerfile(dockerfile_path):
    findings = []
    with open(dockerfile_path) as f:
        lines = f.readlines()

    has_user = False
    has_healthcheck = False

    for i, line in enumerate(lines, 1):
        stripped = line.strip()
        if stripped.startswith("USER") and stripped.split()[-1] not in ("root", "0"):
            has_user = True
        if stripped.startswith("HEALTHCHECK"):
            has_healthcheck = True
        if stripped.startswith("FROM") and ":latest" in stripped:
            findings.append({
                "line": i, "severity": "medium",
                "issue": "Using :latest tag — pin specific version",
            })
        if "ADD" in stripped and ("http://" in stripped or "https://" in stripped):
            findings.append({
                "line": i, "severity": "high",
                "issue": "ADD with remote URL — use COPY + curl for verification",
            })

    if not has_user:
        findings.append({"line": 0, "severity": "high", "issue": "No USER instruction — runs as root"})
    if not has_healthcheck:
        findings.append({"line": 0, "severity": "low", "issue": "No HEALTHCHECK instruction"})

    return findings
```

## Output Format

```json
{
  "image": "myapp:v1.2.3",
  "vulnerabilities": {
    "critical": 2,
    "high": 8,
    "medium": 15,
    "low": 23
  },
  "runs_as_root": false,
  "size_mb": 142.5,
  "layers": 12,
  "dockerfile_issues": 3,
  "secrets_found": 0,
  "findings": [
    {
      "type": "vulnerability",
      "package": "openssl",
      "installed": "3.0.2",
      "fixed": "3.0.13",
      "severity": "CRITICAL",
      "cve": "CVE-2024-0727"
    }
  ]
}
```

===== END FILE =====


===== FILE: skills/performing-container-image-hardening/references/standards.md =====
# Standards Reference: Container Image Hardening

## CIS Docker Benchmark v1.6.0 - Image Controls

- 4.1: Create a user for the container
- 4.2: Use trusted base images
- 4.3: Do not install unnecessary packages
- 4.4: Scan and rebuild images for security patches
- 4.6: Add HEALTHCHECK instruction
- 4.7: Do not use update instructions alone
- 4.9: Use COPY instead of ADD
- 4.10: Do not store secrets in Dockerfiles
- 4.11: Install verified packages only

## NIST SP 800-190 Application Container Security Guide

### Image Hardening (Section 4.1)
- Use minimal base images to reduce attack surface
- Remove unnecessary tools, shells, and package managers
- Scan images for vulnerabilities before deployment
- Sign images for integrity verification

## OWASP Docker Security Cheat Sheet

- Use multi-stage builds
- Run as non-root user
- Use read-only root filesystem
- Pin base images to digests
- Drop all Linux capabilities
- Enable seccomp profile

===== END FILE =====


===== FILE: skills/performing-container-image-hardening/references/workflows.md =====
# Workflow Reference: Container Image Hardening

## Hardening Pipeline

```
Base Image Selection
       │
       ▼
┌──────────────────┐
│ Multi-Stage Build │
│ (builder + prod)  │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Remove Packages  │
│ + Set User/Perms │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Trivy Scan       │
│ + Hadolint       │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ CIS Benchmark    │
│ Validation       │
└──────────────────┘
```

## Base Image Selection Guide

| Base Image | Size | Use Case | Packages |
|------------|------|----------|----------|
| scratch | 0 MB | Static Go/Rust binaries | None |
| distroless/static | 2 MB | Static binaries + CA certs | ca-certificates |
| distroless/base | 20 MB | Dynamic binaries | glibc, libssl |
| alpine:3.19 | 7 MB | General minimal | musl, busybox |
| debian:bookworm-slim | 80 MB | Debian ecosystem | apt, glibc |
| ubuntu:24.04 | 78 MB | Ubuntu ecosystem | apt, glibc |

## Dockerfile Hardening Checklist

- [ ] Multi-stage build separates build and runtime
- [ ] Minimal base image selected
- [ ] Non-root USER instruction present
- [ ] HEALTHCHECK instruction defined
- [ ] Base image pinned by digest
- [ ] COPY used instead of ADD
- [ ] No secrets in ENV or ARG
- [ ] Package cache cleaned (rm -rf /var/lib/apt/lists/*)
- [ ] Unnecessary packages removed
- [ ] setuid/setgid bits cleared
- [ ] Shell removed (if not needed)

===== END FILE =====


===== FILE: skills/performing-container-image-hardening/scripts/agent.py =====
#!/usr/bin/env python3
"""Container image hardening audit agent.

Audits Docker container images for security hardening best practices
using Trivy for vulnerability scanning, Dockle for CIS Docker Benchmark
compliance, and Dockerfile analysis for security anti-patterns.
"""
import argparse
import json
import os
import subprocess
import sys
from datetime import datetime, timezone


def find_binary(name):
    """Locate a binary on the system PATH."""
    for ext in ["", ".exe"]:
        for directory in os.environ.get("PATH", "").split(os.pathsep):
            full_path = os.path.join(directory, name + ext)
            if os.path.isfile(full_path):
                return full_path
    return None


def run_trivy_image_scan(image, severity="CRITICAL,HIGH", ignore_unfixed=True):
    """Scan a container image with Trivy for vulnerabilities."""
    trivy_bin = find_binary("trivy")
    if not trivy_bin:
        print("[!] trivy not found. Install: https://github.com/aquasecurity/trivy",
              file=sys.stderr)
        return None

    cmd = [trivy_bin, "image", "--format", "json", "--severity", severity]
    if ignore_unfixed:
        cmd.append("--ignore-unfixed")
    cmd.append(image)

    print(f"[*] Running Trivy scan: {' '.join(cmd)}")
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
    if result.returncode != 0 and not result.stdout:
        print(f"[!] Trivy error: {result.stderr[:300]}", file=sys.stderr)
        return None
    try:
        return json.loads(result.stdout)
    except json.JSONDecodeError:
        return None


def run_dockle_scan(image):
    """Scan a container image with Dockle for CIS benchmark compliance."""
    dockle_bin = find_binary("dockle")
    if not dockle_bin:
        print("[*] dockle not found, skipping CIS benchmark check", file=sys.stderr)
        return None

    cmd = [dockle_bin, "--format", "json", image]
    print(f"[*] Running Dockle scan: {' '.join(cmd)}")
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    try:
        return json.loads(result.stdout)
    except json.JSONDecodeError:
        return None


def analyze_dockerfile(dockerfile_path):
    """Analyze a Dockerfile for security anti-patterns."""
    findings = []
    if not os.path.isfile(dockerfile_path):
        return findings

    with open(dockerfile_path, "r") as f:
        lines = f.readlines()

    runs_as_root = True
    has_healthcheck = False
    uses_latest_tag = False

    for i, line in enumerate(lines, 1):
        stripped = line.strip()
        upper = stripped.upper()

        if upper.startswith("FROM") and ":latest" in stripped:
            uses_latest_tag = True
            findings.append({
                "check": "FROM uses :latest tag",
                "line": i,
                "severity": "HIGH",
                "description": "Pin image to a specific version for reproducibility and security",
                "content": stripped,
            })

        if upper.startswith("FROM") and "scratch" not in stripped.lower():
            base = stripped.split()[-1] if stripped.split() else ""
            if "alpine" not in base.lower() and "distroless" not in base.lower():
                findings.append({
                    "check": "Non-minimal base image",
                    "line": i,
                    "severity": "MEDIUM",
                    "description": "Consider using Alpine or distroless base for smaller attack surface",
                    "content": stripped,
                })

        if upper.startswith("USER") and stripped.split()[-1] not in ("root", "0"):
            runs_as_root = False

        if upper.startswith("HEALTHCHECK"):
            has_healthcheck = True

        if upper.startswith("RUN") and ("chmod 777" in stripped or "chmod -R 777" in stripped):
            findings.append({
                "check": "Overly permissive chmod 777",
                "line": i,
                "severity": "HIGH",
                "description": "chmod 777 grants world-writable permissions; use specific permissions",
                "content": stripped,
            })

        if upper.startswith("RUN") and "curl" in stripped and "| sh" in stripped:
            findings.append({
                "check": "Pipe to shell from curl",
                "line": i,
                "severity": "CRITICAL",
                "description": "Piping curl output to shell is risky; download, verify, then execute",
                "content": stripped,
            })

        if upper.startswith("ENV") and any(kw in upper for kw in ["PASSWORD", "SECRET", "TOKEN", "API_KEY"]):
            findings.append({
                "check": "Secrets in ENV instruction",
                "line": i,
                "severity": "CRITICAL",
                "description": "Never embed secrets in Dockerfile; use build args or secrets mount",
                "content": stripped,
            })

        if upper.startswith("ADD") and not stripped.endswith(".tar.gz"):
            findings.append({
                "check": "ADD instead of COPY",
                "line": i,
                "severity": "LOW",
                "description": "Use COPY unless you need ADD's auto-extraction; COPY is more explicit",
                "content": stripped,
            })

        if upper.startswith("EXPOSE") and any(p in stripped for p in ["22", "23", "3389"]):
            findings.append({
                "check": "Exposed management port",
                "line": i,
                "severity": "HIGH",
                "description": "SSH/Telnet/RDP ports should not be exposed in containers",
                "content": stripped,
            })

    if runs_as_root:
        findings.append({
            "check": "Container runs as root",
            "line": 0,
            "severity": "HIGH",
            "description": "Add a USER instruction to run as non-root for least privilege",
        })

    if not has_healthcheck:
        findings.append({
            "check": "Missing HEALTHCHECK",
            "line": 0,
            "severity": "LOW",
            "description": "Add HEALTHCHECK to enable container health monitoring",
        })

    return findings


def extract_trivy_findings(trivy_data):
    """Extract vulnerability findings from Trivy JSON output."""
    findings = []
    if not trivy_data:
        return findings
    results = trivy_data.get("Results", [])
    for result in results:
        target = result.get("Target", "")
        for vuln in result.get("Vulnerabilities", []):
            findings.append({
                "source": "trivy",
                "target": target,
                "vulnerability_id": vuln.get("VulnerabilityID", ""),
                "pkg_name": vuln.get("PkgName", ""),
                "installed_version": vuln.get("InstalledVersion", ""),
                "fixed_version": vuln.get("FixedVersion", ""),
                "severity": vuln.get("Severity", "UNKNOWN"),
                "title": vuln.get("Title", ""),
                "description": vuln.get("Description", "")[:200],
            })
    return findings


def extract_dockle_findings(dockle_data):
    """Extract CIS benchmark findings from Dockle JSON output."""
    findings = []
    if not dockle_data:
        return findings
    for detail in dockle_data.get("details", []):
        severity_map = {"FATAL": "CRITICAL", "WARN": "HIGH", "INFO": "MEDIUM", "SKIP": "LOW"}
        findings.append({
            "source": "dockle",
            "code": detail.get("code", ""),
            "title": detail.get("title", ""),
            "level": detail.get("level", "INFO"),
            "severity": severity_map.get(detail.get("level", "INFO"), "MEDIUM"),
            "alerts": detail.get("alerts", []),
        })
    return findings


def format_summary(image, trivy_findings, dockle_findings, dockerfile_findings):
    """Print combined audit summary."""
    all_findings = trivy_findings + dockle_findings + dockerfile_findings
    print(f"\n{'='*60}")
    print(f"  Container Image Hardening Audit")
    print(f"{'='*60}")
    print(f"  Image          : {image}")
    print(f"  Vulnerabilities: {len(trivy_findings)} (Trivy)")
    print(f"  CIS Benchmark  : {len(dockle_findings)} (Dockle)")
    print(f"  Dockerfile     : {len(dockerfile_findings)}")
    print(f"  Total Findings : {len(all_findings)}")

    severity_counts = {}
    for f in all_findings:
        sev = f.get("severity", "UNKNOWN")
        severity_counts[sev] = severity_counts.get(sev, 0) + 1

    print(f"\n  By Severity:")
    for sev in ["CRITICAL", "HIGH", "MEDIUM", "LOW", "UNKNOWN"]:
        count = severity_counts.get(sev, 0)
        if count > 0:
            print(f"    {sev:10s}: {count}")

    if trivy_findings:
        print(f"\n  Top Vulnerabilities:")
        for f in trivy_findings[:10]:
            print(f"    {f['vulnerability_id']:16s} | {f['severity']:8s} | "
                  f"{f['pkg_name']}:{f['installed_version']} -> {f.get('fixed_version', 'N/A')}")

    if dockerfile_findings:
        print(f"\n  Dockerfile Issues:")
        for f in dockerfile_findings:
            print(f"    [{f['severity']:8s}] Line {f.get('line', 0):3d}: {f['check']}")

    return severity_counts


def main():
    parser = argparse.ArgumentParser(
        description="Container image hardening audit agent"
    )
    parser.add_argument("--image", required=True, help="Container image to scan (e.g., nginx:1.25)")
    parser.add_argument("--dockerfile", help="Path to Dockerfile for static analysis")
    parser.add_argument("--severity", default="CRITICAL,HIGH",
                        help="Trivy severity filter (default: CRITICAL,HIGH)")
    parser.add_argument("--skip-trivy", action="store_true", help="Skip Trivy scan")
    parser.add_argument("--skip-dockle", action="store_true", help="Skip Dockle scan")
    parser.add_argument("--output", "-o", help="Output JSON report path")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    trivy_findings = []
    dockle_findings = []
    dockerfile_findings = []

    if not args.skip_trivy:
        trivy_data = run_trivy_image_scan(args.image, args.severity)
        trivy_findings = extract_trivy_findings(trivy_data)

    if not args.skip_dockle:
        dockle_data = run_dockle_scan(args.image)
        dockle_findings = extract_dockle_findings(dockle_data)

    if args.dockerfile:
        dockerfile_findings = analyze_dockerfile(args.dockerfile)

    severity_counts = format_summary(
        args.image, trivy_findings, dockle_findings, dockerfile_findings
    )

    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "Container Hardening Audit",
        "image": args.image,
        "severity_counts": severity_counts,
        "trivy_findings": trivy_findings,
        "dockle_findings": dockle_findings,
        "dockerfile_findings": dockerfile_findings,
        "total_findings": len(trivy_findings) + len(dockle_findings) + len(dockerfile_findings),
        "risk_level": (
            "CRITICAL" if severity_counts.get("CRITICAL", 0) > 0
            else "HIGH" if severity_counts.get("HIGH", 0) > 0
            else "MEDIUM" if severity_counts.get("MEDIUM", 0) > 0
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


===== FILE: skills/performing-container-image-hardening/scripts/process.py =====
#!/usr/bin/env python3
"""
Container Image Hardening Validation Script

Validates container images against hardening best practices,
checks for non-root user, minimal packages, and CIS compliance.

Usage:
    python process.py --image myapp:latest --output hardening-report.json
"""

import argparse
import json
import os
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime, timezone


@dataclass
class HardeningCheck:
    check_id: str
    name: str
    passed: bool
    details: str
    severity: str


def run_docker_inspect(image: str) -> dict:
    """Inspect a Docker image and return configuration."""
    cmd = ["docker", "inspect", image]
    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        if proc.returncode == 0:
            data = json.loads(proc.stdout)
            return data[0] if data else {}
        return {"error": proc.stderr}
    except (subprocess.TimeoutExpired, FileNotFoundError, json.JSONDecodeError) as e:
        return {"error": str(e)}


def check_non_root_user(config: dict) -> HardeningCheck:
    """Check if image runs as non-root user."""
    user = config.get("Config", {}).get("User", "")
    if user and user != "0" and user != "root":
        return HardeningCheck("CIS-4.1", "Non-root user configured", True,
                              f"User: {user}", "HIGH")
    return HardeningCheck("CIS-4.1", "Non-root user configured", False,
                          "Image runs as root. Add USER instruction.", "HIGH")


def check_healthcheck(config: dict) -> HardeningCheck:
    """Check if HEALTHCHECK is defined."""
    healthcheck = config.get("Config", {}).get("Healthcheck")
    if healthcheck and healthcheck.get("Test"):
        return HardeningCheck("CIS-4.6", "HEALTHCHECK defined", True,
                              f"Test: {healthcheck['Test']}", "MEDIUM")
    return HardeningCheck("CIS-4.6", "HEALTHCHECK defined", False,
                          "No HEALTHCHECK instruction found.", "MEDIUM")


def check_exposed_ports(config: dict) -> HardeningCheck:
    """Check for unnecessary exposed ports."""
    ports = config.get("Config", {}).get("ExposedPorts", {})
    port_list = list(ports.keys()) if ports else []
    if len(port_list) <= 2:
        return HardeningCheck("HARD-1", "Minimal ports exposed", True,
                              f"Ports: {port_list}", "LOW")
    return HardeningCheck("HARD-1", "Minimal ports exposed", False,
                          f"Many ports exposed: {port_list}. Minimize exposed ports.", "LOW")


def check_image_size(image: str) -> HardeningCheck:
    """Check image size against recommended limits."""
    cmd = ["docker", "image", "inspect", image, "--format", "{{.Size}}"]
    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
        size_bytes = int(proc.stdout.strip())
        size_mb = size_bytes / (1024 * 1024)
        if size_mb < 200:
            return HardeningCheck("HARD-2", "Image size within limits", True,
                                  f"Size: {size_mb:.0f} MB (< 200 MB)", "MEDIUM")
        elif size_mb < 500:
            return HardeningCheck("HARD-2", "Image size within limits", False,
                                  f"Size: {size_mb:.0f} MB (> 200 MB, consider optimizing)", "MEDIUM")
        else:
            return HardeningCheck("HARD-2", "Image size within limits", False,
                                  f"Size: {size_mb:.0f} MB (> 500 MB, use multi-stage build)", "HIGH")
    except (subprocess.TimeoutExpired, ValueError):
        return HardeningCheck("HARD-2", "Image size within limits", False,
                              "Could not determine image size", "LOW")


def check_env_secrets(config: dict) -> HardeningCheck:
    """Check for potential secrets in environment variables."""
    env_vars = config.get("Config", {}).get("Env", [])
    secret_keywords = ["password", "secret", "key", "token", "credential", "api_key"]
    suspicious = []
    for env in env_vars:
        name = env.split("=")[0].lower()
        if any(kw in name for kw in secret_keywords):
            suspicious.append(env.split("=")[0])
    if not suspicious:
        return HardeningCheck("CIS-4.10", "No secrets in ENV", True,
                              "No suspicious environment variables found", "HIGH")
    return HardeningCheck("CIS-4.10", "No secrets in ENV", False,
                          f"Suspicious ENV vars: {suspicious}. Use secrets manager.", "HIGH")


def check_shell_available(image: str) -> HardeningCheck:
    """Check if shell is available in the image."""
    cmd = ["docker", "run", "--rm", "--entrypoint", "", image, "sh", "-c", "echo shell_available"]
    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=15)
        if "shell_available" in proc.stdout:
            return HardeningCheck("HARD-3", "Shell removed from image", False,
                                  "Shell (/bin/sh) is available. Consider removing for production.", "LOW")
        return HardeningCheck("HARD-3", "Shell removed from image", True,
                              "No shell available in image", "LOW")
    except (subprocess.TimeoutExpired, FileNotFoundError):
        return HardeningCheck("HARD-3", "Shell removed from image", True,
                              "Could not test shell access (likely unavailable)", "LOW")


def main():
    parser = argparse.ArgumentParser(description="Container Image Hardening Validation")
    parser.add_argument("--image", required=True, help="Docker image to validate")
    parser.add_argument("--output", default="hardening-report.json")
    parser.add_argument("--fail-on-findings", action="store_true")
    args = parser.parse_args()

    print(f"[*] Validating hardening: {args.image}")

    config = run_docker_inspect(args.image)
    if "error" in config:
        print(f"[ERROR] {config['error']}")
        sys.exit(2)

    checks = [
        check_non_root_user(config),
        check_healthcheck(config),
        check_exposed_ports(config),
        check_image_size(args.image),
        check_env_secrets(config),
        check_shell_available(args.image),
    ]

    passed = sum(1 for c in checks if c.passed)
    failed = sum(1 for c in checks if not c.passed)

    report = {
        "metadata": {"image": args.image, "date": datetime.now(timezone.utc).isoformat()},
        "summary": {"total": len(checks), "passed": passed, "failed": failed},
        "checks": [
            {"id": c.check_id, "name": c.name, "passed": c.passed,
             "details": c.details, "severity": c.severity}
            for c in checks
        ]
    }

    output_path = os.path.abspath(args.output)
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)

    for c in checks:
        status = "PASS" if c.passed else "FAIL"
        print(f"  [{status}] {c.check_id}: {c.name} - {c.details}")

    print(f"\n[*] {passed}/{len(checks)} checks passed")
    if args.fail_on_findings and failed > 0:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-sca-dependency-scanning-with-snyk/SKILL.md =====
---
name: performing-sca-dependency-scanning-with-snyk
description: 'This skill covers implementing Software Composition Analysis (SCA) using
  Snyk to detect vulnerable open-source dependencies in CI/CD pipelines. It addresses
  scanning package manifests and lockfiles, automated fix pull request generation,
  license compliance checking, continuous monitoring of deployed applications, and
  integration with GitHub, GitLab, and Jenkins pipelines.

  '
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- cicd
- sca
- snyk
- dependency-scanning
- secure-sdlc
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
---

# Performing SCA Dependency Scanning with Snyk

## When to Use

- When applications use open-source packages that may contain known vulnerabilities
- When compliance requires tracking and remediating vulnerable dependencies (PCI DSS, SOC 2)
- When needing automated fix PRs for vulnerable dependencies in CI/CD
- When license compliance requires visibility into open-source license obligations
- When continuous monitoring is needed for newly disclosed vulnerabilities in deployed dependencies

**Do not use** for scanning proprietary application code for logic vulnerabilities (use SAST), for runtime vulnerability detection (use DAST), or for container OS package scanning alone (use Trivy for a free alternative).

## Prerequisites

- Snyk account (free tier covers up to 200 tests per month for open source)
- Snyk CLI installed or Snyk GitHub/GitLab integration configured
- SNYK_TOKEN environment variable set with API authentication token
- Project with supported package manifests: package.json, requirements.txt, pom.xml, go.mod, Gemfile, etc.

## Workflow

### Step 1: Install and Authenticate Snyk CLI

```bash
# Install Snyk CLI
npm install -g snyk

# Authenticate with Snyk
snyk auth $SNYK_TOKEN

# Test the connection
snyk test --json | jq '.summary'
```

### Step 2: Scan Dependencies in CI/CD Pipeline

```yaml
# .github/workflows/dependency-scan.yml
name: Dependency Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 8 * * 1'  # Weekly Monday 8am

jobs:
  snyk-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: >
            --severity-threshold=high
            --fail-on=upgradable
            --json-file-output=snyk-results.json

      - name: Upload results to Snyk
        if: always()
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          command: monitor
          args: --project-name=${{ github.repository }}

      - name: Upload SARIF
        if: always()
        run: |
          npx snyk-to-html -i snyk-results.json -o snyk-report.html
```

### Step 3: Configure Snyk for Multiple Languages

```bash
# Python project scanning
snyk test --file=requirements.txt --severity-threshold=high --json > snyk-python.json

# Java/Maven project
snyk test --file=pom.xml --severity-threshold=medium --json > snyk-java.json

# Go module scanning
snyk test --file=go.mod --severity-threshold=high --json > snyk-go.json

# Docker image dependency scanning
snyk container test myapp:latest --severity-threshold=high --json > snyk-container.json

# Monorepo: scan all projects
snyk test --all-projects --severity-threshold=high --json > snyk-all.json

# IaC scanning (bonus)
snyk iac test terraform/ --severity-threshold=medium --json > snyk-iac.json
```

### Step 4: Configure Snyk Policies for Organization

```yaml
# .snyk policy file
version: v1.25.0
ignore:
  SNYK-JS-LODASH-1018905:
    - '*':
        reason: "Prototype pollution in lodash. Not exploitable in our usage - no user input reaches affected function."
        expires: 2026-06-01T00:00:00.000Z
        created: 2026-02-23T00:00:00.000Z

  SNYK-PYTHON-REQUESTS-6241864:
    - '*':
        reason: "SSRF in requests redirect handling. Mitigated by allowlist at proxy layer."
        expires: 2026-04-01T00:00:00.000Z

patch: {}

# Severity threshold for CI failures
failOnSeverity: high
```

### Step 5: Enable Automated Fix Pull Requests

```bash
# Snyk fix: generate fix PRs for vulnerable dependencies
snyk fix --dry-run  # Preview changes

# Apply fixes locally
snyk fix

# Enable auto-fix PRs via Snyk dashboard:
# 1. Navigate to Organization Settings > Integrations > GitHub
# 2. Enable "Automatic fix pull requests"
# 3. Set "Fix only direct dependencies" or "Fix direct and transitive"
# 4. Configure branch target (main or develop)
```

### Step 6: License Compliance Scanning

```bash
# Check license compliance
snyk test --json | jq '.licensesPolicy'

# Snyk license policy configuration via organization settings:
# - Approved licenses: MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC
# - Restricted licenses: GPL-3.0, AGPL-3.0 (copyleft risk)
# - Unknown licenses: Flag for manual review
```

## Key Concepts

| Term | Definition |
|------|------------|
| SCA | Software Composition Analysis — identifies vulnerabilities and license risks in open-source dependencies |
| Transitive Dependency | A dependency of a direct dependency, often invisible to developers but still a vulnerability vector |
| Fix PR | Automated pull request generated by Snyk that upgrades a vulnerable dependency to a patched version |
| Snyk Monitor | Continuous monitoring mode that watches deployed projects for newly disclosed vulnerabilities |
| Exploit Maturity | Snyk's assessment of whether a vulnerability has known exploits, proof-of-concept, or no known exploit |
| Reachable Vulnerability | A vulnerability in a function that is actually called by the application code, not just present in the dependency |
| License Policy | Organization-level rules defining which open-source licenses are approved, restricted, or require review |

## Tools & Systems

- **Snyk Open Source**: SCA tool for scanning dependencies across 10+ language ecosystems
- **Snyk CLI**: Command-line interface for local and CI/CD scanning of dependencies
- **Snyk Advisor**: Package health scoring tool evaluating maintenance, popularity, and security signals
- **OWASP Dependency-Check**: Free alternative SCA tool using NVD data for vulnerability matching
- **npm audit / pip-audit**: Language-specific built-in audit tools for basic vulnerability checking

## Common Scenarios

### Scenario: Triaging a Critical Transitive Dependency Vulnerability

**Context**: Snyk reports a critical RCE vulnerability in a transitive dependency (log4j in a Java application). The direct dependency has not released a patch.

**Approach**:
1. Use `snyk test --json` and examine the dependency path to identify which direct dependency pulls in the vulnerable transitive
2. Check exploit maturity: if "Mature" or "Proof of Concept", prioritize immediately
3. If no direct fix exists, use Snyk's patch mechanism or override the transitive version in the build config
4. For Maven: add `<dependencyManagement>` section to force the safe version of the transitive dependency
5. For npm: add an `overrides` section in package.json to pin the safe version
6. Add a Snyk ignore with expiration date if no patch is available yet
7. Monitor the direct dependency for a release that updates the transitive

**Pitfalls**: Ignoring transitive vulnerabilities because "we don't use that function directly" is risky. Attackers can chain vulnerabilities across dependency boundaries. Version overrides can break API compatibility between the direct and transitive dependency.

## Output Format

```
Snyk Dependency Scan Report
=============================
Project: org/web-application
Manifest: package.json
Dependencies: 342 (47 direct, 295 transitive)
Scan Date: 2026-02-23

VULNERABILITY SUMMARY:
  Critical: 1  (1 fixable)
  High: 4      (3 fixable)
  Medium: 12   (8 fixable)
  Low: 23      (15 fixable)

CRITICAL:
  SNYK-JS-EXPRESS-1234567
    Package: express@4.17.1 (direct)
    Severity: Critical (CVSS 9.8)
    Exploit: Mature
    Fix: Upgrade to express@4.21.0
    Path: express@4.17.1

HIGH:
  SNYK-JS-JSONWEBTOKEN-5678901
    Package: jsonwebtoken@8.5.1 (transitive)
    Severity: High (CVSS 7.6)
    Exploit: Proof of Concept
    Fix: Upgrade passport@0.7.0 (which upgrades jsonwebtoken)
    Path: passport@0.6.0 > jsonwebtoken@8.5.1

LICENSE ISSUES:
  [RESTRICTED] GPL-3.0: some-package@1.2.3 (transitive via other-pkg)

QUALITY GATE: FAILED (1 Critical with fix available)
```

===== END FILE =====


===== FILE: skills/performing-sca-dependency-scanning-with-snyk/LICENSE =====

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


===== FILE: skills/performing-sca-dependency-scanning-with-snyk/assets/template.md =====
# Snyk SCA Scanning Templates

## GitHub Actions: Multi-Language SCA Pipeline

```yaml
# .github/workflows/sca-scanning.yml
name: SCA Dependency Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
    paths:
      - 'package*.json'
      - 'requirements*.txt'
      - 'Pipfile*'
      - 'pom.xml'
      - 'build.gradle*'
      - 'go.mod'
      - 'Gemfile*'

jobs:
  snyk-node:
    name: Snyk Node.js
    runs-on: ubuntu-latest
    if: hashFiles('package.json') != ''
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high --fail-on=upgradable

  snyk-python:
    name: Snyk Python
    runs-on: ubuntu-latest
    if: hashFiles('requirements.txt') != ''
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.12' }
      - run: pip install -r requirements.txt
      - uses: snyk/actions/python-3.10@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  monitor:
    name: Snyk Monitor
    needs: [snyk-node, snyk-python]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          command: monitor
          args: --all-projects
```

## Snyk Policy File

```yaml
# .snyk
version: v1.25.0
ignore:
  # Accepted risk with documented justification and expiry
  SNYK-JS-LODASH-1018905:
    - '*':
        reason: "Not exploitable: user input never reaches _.template()"
        expires: 2026-06-01T00:00:00.000Z
        created: 2026-02-23T00:00:00.000Z

patch: {}
```

## Snyk Configuration for License Compliance

```json
{
  "org_settings": {
    "license_policy": {
      "approved": ["MIT", "Apache-2.0", "BSD-2-Clause", "BSD-3-Clause", "ISC", "Unlicense", "0BSD"],
      "review_required": ["LGPL-2.1", "LGPL-3.0", "MPL-2.0", "CDDL-1.0"],
      "restricted": ["GPL-2.0", "GPL-3.0", "AGPL-3.0", "SSPL-1.0"],
      "severity_for_unknown": "high"
    }
  }
}
```

## OWASP Dependency-Check Alternative (Free)

```yaml
# .github/workflows/dependency-check.yml
name: OWASP Dependency Check

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 8 * * 1'

jobs:
  dependency-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run OWASP Dependency-Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'my-app'
          path: '.'
          format: 'HTML'
          args: >
            --failOnCVSS 7
            --enableRetired
      - name: Upload report
        uses: actions/upload-artifact@v4
        with:
          name: dependency-check-report
          path: reports/
```

===== END FILE =====


===== FILE: skills/performing-sca-dependency-scanning-with-snyk/references/api-reference.md =====
# SCA Dependency Scanning with Snyk - API Reference

## Snyk CLI Commands

### snyk test
Scans project dependencies for known vulnerabilities.

```bash
snyk test --json --severity-threshold=high
snyk test --json --all-projects          # Monorepo support
snyk test --json --file=package-lock.json
```

Exit codes:
- 0: No vulnerabilities found
- 1: Vulnerabilities found
- 2: Failure (missing manifest, auth error)

### snyk monitor
Creates a project snapshot for continuous monitoring on snyk.io.

```bash
snyk monitor --json --project-name="my-app"
```

### snyk auth
Authenticate with Snyk API token.

```bash
snyk auth <API_TOKEN>
export SNYK_TOKEN=<API_TOKEN>
```

## JSON Output Structure

### Test Result Fields

| Field | Type | Description |
|-------|------|-------------|
| `vulnerabilities` | array | List of vulnerability objects |
| `ok` | boolean | True if no vulns found |
| `dependencyCount` | int | Total dependencies scanned |
| `packageManager` | string | npm, pip, maven, etc. |
| `uniqueCount` | int | Unique vulnerability count |

### Vulnerability Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Snyk vulnerability ID (e.g., `SNYK-JS-LODASH-590103`) |
| `title` | string | Human-readable title |
| `severity` | string | critical, high, medium, low |
| `cvssScore` | float | CVSS v3.1 score (0-10) |
| `packageName` | string | Affected package name |
| `version` | string | Installed version |
| `fixedIn` | array | Versions with fix available |
| `exploit` | string | Exploit maturity: Mature, Proof of Concept, Not Defined |
| `isUpgradable` | boolean | Can be fixed by upgrading direct dependency |
| `isPatchable` | boolean | Snyk patch available |
| `from` | array | Dependency path from root |

## SARIF Integration

Snyk results can be converted to SARIF 2.1.0 for GitHub Code Scanning. The SARIF schema is at:
`https://raw.githubusercontent.com/oasis-tcs/sarif-spec/main/sarif-2.1/schema/sarif-schema-2.1.0.json`

## Severity Mapping

| Snyk Severity | CVSS Range | SARIF Level |
|---------------|-----------|-------------|
| critical | 9.0 - 10.0 | error |
| high | 7.0 - 8.9 | error |
| medium | 4.0 - 6.9 | warning |
| low | 0.1 - 3.9 | warning |

## CLI Usage

```bash
python agent.py --project /app --severity high --max-critical 0 --max-high 5 --output report.json
```

===== END FILE =====


===== FILE: skills/performing-sca-dependency-scanning-with-snyk/references/standards.md =====
# Standards Reference: SCA Dependency Scanning with Snyk

## OWASP Top 10 - A06:2021 Vulnerable and Outdated Components

- Applications using components with known vulnerabilities may be exploitable
- SCA tools like Snyk identify vulnerable versions and provide upgrade paths
- Includes both direct and transitive dependency scanning

## NIST SSDF (SP 800-218)

### PW.4: Reuse Existing, Well-Secured Software
- PW.4.1: Verify that acquired software meets security requirements
- PW.4.2: Review, analyze, and test software to identify vulnerabilities
- SCA scanning of all third-party components before integration

### PW.4.4: Maintain Provenance Data
- Track the origin and version of all third-party software components
- Snyk monitor provides continuous tracking of dependency versions

## CIS Software Supply Chain Security

### Dependencies (DP) Controls
- DP-1: Pin dependencies to specific versions
- DP-2: Automate dependency vulnerability scanning in CI/CD
- DP-3: Review and approve new dependency additions
- DP-4: Monitor deployed dependencies for newly disclosed vulnerabilities

## OWASP SAMM - Software Security

### Security Testing - Maturity Level 1
- Automated dependency scanning using default configurations
- Visibility of vulnerable components to development teams

### Security Testing - Maturity Level 2
- Custom policies for severity thresholds and license compliance
- Automated fix PRs for upgradable vulnerabilities
- Tracking of exploit maturity to prioritize remediation

### Security Testing - Maturity Level 3
- Reachability analysis to identify actually exploitable vulnerabilities
- Integration with vulnerability management for SLA tracking
- Correlation with runtime monitoring for risk-based prioritization

## PCI DSS v4.0

- 6.2.4: Use automated methods to prevent common software attacks
- 6.3.2: Maintain an inventory of custom and third-party software components
- 6.3.3: Software components not needed for operation removed or identified

## Executive Order 14028 (US Federal)

- Section 4(e): Agencies shall employ automated tools for continuous monitoring of vulnerabilities in software
- SBOM requirement: All software suppliers must provide SBOMs listing all components including open-source
- Aligns with Snyk's SBOM generation and continuous monitoring capabilities

## License Compliance Framework

| License Type | Risk Level | Policy | Examples |
|--------------|------------|--------|----------|
| Permissive | Low | Auto-approve | MIT, BSD-2, BSD-3, ISC, Apache-2.0 |
| Weak Copyleft | Medium | Review | LGPL-2.1, LGPL-3.0, MPL-2.0 |
| Strong Copyleft | High | Restrict | GPL-2.0, GPL-3.0, AGPL-3.0 |
| Unknown/Custom | High | Manual Review | Proprietary, SSPL, BSL |

===== END FILE =====


===== FILE: skills/performing-sca-dependency-scanning-with-snyk/references/workflows.md =====
# Workflow Reference: SCA Dependency Scanning with Snyk

## Dependency Scanning Pipeline

```
Code Push / PR
       │
       ▼
┌──────────────────┐
│ Install Deps     │
│ (npm ci, pip     │
│  install, etc.)  │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Snyk Test        │──── Report JSON ───> Artifact Storage
│ (vuln scan)      │
└──────┬───────────┘
       │
  ┌────┴────┐
  │         │
PASS      FAIL ──────> PR Comment with vuln details
  │                     │
  │                     ▼
  │              ┌──────────────┐
  │              │ Snyk Fix PR  │
  │              │ (auto-gen)   │
  │              └──────────────┘
  ▼
┌──────────────────┐
│ Snyk Monitor     │
│ (continuous)     │
└──────┬───────────┘
       │
       ▼
  Ongoing alerts for
  new disclosures
```

## Snyk CLI Command Reference

### Scanning Commands
```bash
# Basic vulnerability test
snyk test

# Test with severity filter
snyk test --severity-threshold=high

# Test with exploit maturity filter
snyk test --severity-threshold=high

# Test specific manifest
snyk test --file=package-lock.json

# Test all projects in monorepo
snyk test --all-projects

# Test with dev dependencies excluded
snyk test --production

# Output in JSON
snyk test --json --json-file-output=results.json

# Output in SARIF
snyk test --sarif --sarif-file-output=results.sarif
```

### Monitoring Commands
```bash
# Monitor project for new vulnerabilities
snyk monitor --project-name="my-app-prod"

# Monitor specific branch
snyk monitor --target-reference=main

# Monitor with tags
snyk monitor --project-tags=env=production,team=platform
```

### Fix Commands
```bash
# Preview available fixes
snyk fix --dry-run

# Apply fixes to direct dependencies
snyk fix

# Apply fixes including dev dependencies
snyk fix --dev
```

## Vulnerability Prioritization Matrix

| Factor | Score Weight | Description |
|--------|-------------|-------------|
| CVSS Score | 30% | Base vulnerability severity |
| Exploit Maturity | 25% | Mature > POC > No Known Exploit |
| Reachability | 20% | Function called > Imported > Present |
| Fix Availability | 15% | Upgrade available > Patch > None |
| Dependency Depth | 10% | Direct > Transitive (1 hop) > Deep transitive |

## Snyk Integration Options

| Platform | Integration Method | Features |
|----------|--------------------|----------|
| GitHub | GitHub App | Auto-scan PRs, fix PRs, SARIF upload |
| GitLab | GitLab Integration | MR comments, dependency scanning |
| Jenkins | Snyk Plugin | Pipeline step, HTML reports |
| Azure DevOps | Extension | Pipeline task, dashboard widget |
| Bitbucket | Bitbucket App | PR checks, fix PRs |
| CLI | npm/binary | Local scanning, CI/CD integration |

## Remediation Strategy by Vulnerability Type

### Direct Dependency Vulnerability
1. Check if upgrade is available: `snyk test --json | jq '.vulnerabilities[] | select(.isUpgradable)'`
2. If upgradable: run `snyk fix` or manually upgrade
3. Verify no breaking changes in the upgrade
4. If not upgradable: check for patch or accept risk with ignore

### Transitive Dependency Vulnerability
1. Identify the dependency chain: `snyk test --json | jq '.vulnerabilities[].from'`
2. Check if upgrading the direct dependency resolves it
3. If not: use version overrides in package manager
4. npm: `overrides` in package.json
5. Maven: `dependencyManagement` in pom.xml
6. Gradle: `constraints` in build.gradle
7. Poetry: `tool.poetry.extras` or constraint resolution

===== END FILE =====


===== FILE: skills/performing-sca-dependency-scanning-with-snyk/scripts/agent.py =====
#!/usr/bin/env python3
"""SCA Dependency Scanning with Snyk agent — runs Snyk CLI to test
project dependencies for known vulnerabilities, generates SARIF output,
and enforces quality gates."""

import argparse
import json
import subprocess
from datetime import datetime
from pathlib import Path


def run_snyk_test(project_path: str, severity_threshold: str = "low",
                  extra_args: list = None) -> dict:
    """Run snyk test and return parsed JSON results."""
    cmd = ["snyk", "test", "--json", f"--severity-threshold={severity_threshold}"]
    if extra_args:
        cmd.extend(extra_args)
    try:
        result = subprocess.run(cmd, capture_output=True, text=True,
                                cwd=project_path, timeout=300)
        if result.stdout:
            return json.loads(result.stdout)
        return {"error": result.stderr, "exit_code": result.returncode}
    except subprocess.TimeoutExpired:
        return {"error": "Snyk test timed out after 300s"}
    except FileNotFoundError:
        return {"error": "snyk CLI not found. Install: npm install -g snyk"}
    except json.JSONDecodeError:
        return {"error": "Failed to parse Snyk output", "raw": result.stdout[:2000]}


def run_snyk_monitor(project_path: str) -> dict:
    """Run snyk monitor to create a snapshot for continuous monitoring."""
    cmd = ["snyk", "monitor", "--json"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True,
                                cwd=project_path, timeout=300)
        if result.stdout:
            return json.loads(result.stdout)
        return {"error": result.stderr}
    except (subprocess.TimeoutExpired, FileNotFoundError, json.JSONDecodeError) as e:
        return {"error": str(e)}


def parse_vulnerabilities(snyk_result: dict) -> list[dict]:
    """Extract and normalize vulnerability findings."""
    vulns = snyk_result.get("vulnerabilities", [])
    findings = []
    for v in vulns:
        findings.append({
            "id": v.get("id", ""),
            "title": v.get("title", ""),
            "severity": v.get("severity", "unknown"),
            "cvss_score": v.get("cvssScore", 0),
            "package": v.get("packageName", ""),
            "version": v.get("version", ""),
            "fixed_in": v.get("fixedIn", []),
            "exploit_maturity": v.get("exploit", "Not Defined"),
            "is_upgradable": v.get("isUpgradable", False),
            "is_patchable": v.get("isPatchable", False),
            "from_path": v.get("from", []),
        })
    return findings


def apply_quality_gate(findings: list[dict], max_critical: int = 0,
                       max_high: int = 5) -> dict:
    """Apply quality gate based on severity counts."""
    counts = {"critical": 0, "high": 0, "medium": 0, "low": 0}
    for f in findings:
        sev = f.get("severity", "low").lower()
        counts[sev] = counts.get(sev, 0) + 1

    passed = counts["critical"] <= max_critical and counts["high"] <= max_high
    return {
        "passed": passed,
        "severity_counts": counts,
        "gate_criteria": {"max_critical": max_critical, "max_high": max_high},
        "reason": "PASS" if passed else f"FAIL: {counts['critical']} critical (max {max_critical}), {counts['high']} high (max {max_high})",
    }


def generate_sarif(findings: list[dict], project_path: str) -> dict:
    """Convert findings to SARIF 2.1.0 format for GitHub integration."""
    rules = []
    results = []
    seen_ids = set()
    for f in findings:
        rule_id = f["id"]
        if rule_id not in seen_ids:
            rules.append({
                "id": rule_id,
                "shortDescription": {"text": f["title"]},
                "defaultConfiguration": {
                    "level": "error" if f["severity"] in ("critical", "high") else "warning"
                },
            })
            seen_ids.add(rule_id)
        results.append({
            "ruleId": rule_id,
            "message": {"text": f"{f['title']} in {f['package']}@{f['version']}"},
            "level": "error" if f["severity"] in ("critical", "high") else "warning",
        })

    return {
        "$schema": "https://raw.githubusercontent.com/oasis-tcs/sarif-spec/main/sarif-2.1/schema/sarif-schema-2.1.0.json",
        "version": "2.1.0",
        "runs": [{
            "tool": {"driver": {"name": "Snyk", "rules": rules}},
            "results": results,
        }],
    }


def generate_report(project_path: str, severity_threshold: str,
                    max_critical: int, max_high: int) -> dict:
    """Run full scan and build consolidated report."""
    snyk_result = run_snyk_test(project_path, severity_threshold)
    if "error" in snyk_result and "vulnerabilities" not in snyk_result:
        return {"report": "sca_dependency_scan", "error": snyk_result["error"]}

    findings = parse_vulnerabilities(snyk_result)
    gate = apply_quality_gate(findings, max_critical, max_high)
    sarif = generate_sarif(findings, project_path)

    return {
        "report": "sca_dependency_scan",
        "generated_at": datetime.utcnow().isoformat() + "Z",
        "project_path": project_path,
        "total_vulnerabilities": len(findings),
        "quality_gate": gate,
        "unique_packages_affected": len(set(f["package"] for f in findings)),
        "upgradable_count": sum(1 for f in findings if f["is_upgradable"]),
        "patchable_count": sum(1 for f in findings if f["is_patchable"]),
        "findings": findings,
        "sarif": sarif,
    }


def main():
    parser = argparse.ArgumentParser(description="SCA Dependency Scanning with Snyk Agent")
    parser.add_argument("--project", required=True, help="Project directory to scan")
    parser.add_argument("--severity", default="low", choices=["low", "medium", "high", "critical"])
    parser.add_argument("--max-critical", type=int, default=0, help="Max critical vulns for quality gate")
    parser.add_argument("--max-high", type=int, default=5, help="Max high vulns for quality gate")
    parser.add_argument("--output", help="Output JSON file path")
    args = parser.parse_args()

    report = generate_report(args.project, args.severity, args.max_critical, args.max_high)
    output = json.dumps(report, indent=2)
    if args.output:
        Path(args.output).write_text(output, encoding="utf-8")
        print(f"Report written to {args.output}")
    else:
        print(output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-sca-dependency-scanning-with-snyk/scripts/process.py =====
#!/usr/bin/env python3
"""
Snyk SCA Dependency Scanning Pipeline Script

Orchestrates Snyk dependency scans, evaluates quality gates,
and generates consolidated vulnerability reports.

Usage:
    python process.py --project-path /path/to/project --severity-threshold high
    python process.py --project-path . --manifest package.json --output report.json
"""

import argparse
import json
import os
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime, timezone
from typing import Optional


SEVERITY_ORDER = {"critical": 0, "high": 1, "medium": 2, "low": 3}


@dataclass
class VulnFinding:
    snyk_id: str
    title: str
    severity: str
    cvss_score: float
    package_name: str
    installed_version: str
    fixed_version: str
    exploit_maturity: str
    is_upgradable: bool
    is_patchable: bool
    dependency_path: list = field(default_factory=list)
    cwe: list = field(default_factory=list)


@dataclass
class LicenseIssue:
    package_name: str
    version: str
    license_id: str
    severity: str
    dependency_type: str


def run_snyk_test(project_path: str, manifest: Optional[str] = None,
                  severity_threshold: str = "low",
                  all_projects: bool = False) -> dict:
    """Execute Snyk test and return JSON results."""
    cmd = ["snyk", "test", "--json"]

    if manifest:
        cmd.extend(["--file", manifest])
    if all_projects:
        cmd.append("--all-projects")
    cmd.extend(["--severity-threshold", severity_threshold])

    try:
        proc = subprocess.run(
            cmd,
            cwd=project_path,
            capture_output=True,
            text=True,
            timeout=300
        )

        if proc.stdout:
            try:
                return json.loads(proc.stdout)
            except json.JSONDecodeError:
                return {"error": "Failed to parse Snyk JSON output"}
        return {"error": proc.stderr[:500]}

    except subprocess.TimeoutExpired:
        return {"error": "Snyk test timed out after 300 seconds"}
    except FileNotFoundError:
        return {"error": "snyk CLI not found. Install with: npm install -g snyk"}


def parse_vulnerabilities(snyk_json: dict) -> list:
    """Parse Snyk JSON output into VulnFinding objects."""
    vulns = []
    vuln_list = snyk_json.get("vulnerabilities", [])

    for v in vuln_list:
        vulns.append(VulnFinding(
            snyk_id=v.get("id", ""),
            title=v.get("title", ""),
            severity=v.get("severity", "low"),
            cvss_score=v.get("cvssScore", 0.0),
            package_name=v.get("packageName", ""),
            installed_version=v.get("version", ""),
            fixed_version=v.get("fixedIn", ["none"])[0] if v.get("fixedIn") else "none",
            exploit_maturity=v.get("exploit", "No Known Exploit"),
            is_upgradable=v.get("isUpgradable", False),
            is_patchable=v.get("isPatchable", False),
            dependency_path=v.get("from", []),
            cwe=v.get("identifiers", {}).get("CWE", [])
        ))

    return vulns


def deduplicate_vulns(vulns: list) -> list:
    """Remove duplicate vulnerability entries (same ID + package)."""
    seen = set()
    unique = []
    for v in vulns:
        key = f"{v.snyk_id}:{v.package_name}:{v.installed_version}"
        if key not in seen:
            seen.add(key)
            unique.append(v)
    return unique


def evaluate_quality_gate(vulns: list, threshold: str,
                          fail_on: str = "all") -> dict:
    """Evaluate quality gate based on vulnerability severity."""
    threshold_level = SEVERITY_ORDER.get(threshold.lower(), 1)

    blocking = []
    for v in vulns:
        if SEVERITY_ORDER.get(v.severity.lower(), 3) <= threshold_level:
            if fail_on == "upgradable" and not v.is_upgradable:
                continue
            blocking.append(v)

    severity_counts = {}
    for v in vulns:
        sev = v.severity.lower()
        severity_counts[sev] = severity_counts.get(sev, 0) + 1

    fixable_count = sum(1 for v in vulns if v.is_upgradable or v.is_patchable)

    return {
        "passed": len(blocking) == 0,
        "threshold": threshold,
        "fail_on": fail_on,
        "total_vulnerabilities": len(vulns),
        "blocking_count": len(blocking),
        "fixable_count": fixable_count,
        "severity_counts": severity_counts,
        "blocking_details": [
            {
                "id": v.snyk_id,
                "title": v.title,
                "severity": v.severity,
                "package": f"{v.package_name}@{v.installed_version}",
                "fix": v.fixed_version,
                "upgradable": v.is_upgradable,
                "exploit": v.exploit_maturity
            }
            for v in blocking[:20]
        ]
    }


def generate_report(vulns: list, quality_gate: dict, snyk_json: dict,
                    project_path: str) -> dict:
    """Generate consolidated SCA report."""
    dep_count = snyk_json.get("dependencyCount", 0)

    exploit_summary = {}
    for v in vulns:
        exploit_summary[v.exploit_maturity] = exploit_summary.get(v.exploit_maturity, 0) + 1

    return {
        "report_metadata": {
            "project": project_path,
            "scan_date": datetime.now(timezone.utc).isoformat(),
            "total_dependencies": dep_count
        },
        "quality_gate": quality_gate,
        "exploit_maturity_breakdown": exploit_summary,
        "vulnerabilities": [
            {
                "id": v.snyk_id,
                "title": v.title,
                "severity": v.severity,
                "cvss": v.cvss_score,
                "package": v.package_name,
                "version": v.installed_version,
                "fixed_in": v.fixed_version,
                "exploit": v.exploit_maturity,
                "upgradable": v.is_upgradable,
                "path": " > ".join(v.dependency_path[:4]),
                "cwe": v.cwe
            }
            for v in sorted(vulns, key=lambda x: SEVERITY_ORDER.get(x.severity.lower(), 3))
        ]
    }


def main():
    parser = argparse.ArgumentParser(description="Snyk SCA Dependency Scanning Pipeline")
    parser.add_argument("--project-path", required=True, help="Path to project")
    parser.add_argument("--manifest", default=None, help="Manifest file (e.g., package.json)")
    parser.add_argument("--output", default="snyk-report.json", help="Output report path")
    parser.add_argument("--severity-threshold", default="high",
                        choices=["critical", "high", "medium", "low"])
    parser.add_argument("--fail-on", default="all", choices=["all", "upgradable"],
                        help="Fail on all vulns or only upgradable ones")
    parser.add_argument("--fail-on-findings", action="store_true")
    parser.add_argument("--all-projects", action="store_true",
                        help="Scan all projects in monorepo")
    parser.add_argument("--monitor", action="store_true",
                        help="Also run snyk monitor for continuous tracking")
    args = parser.parse_args()

    project_path = os.path.abspath(args.project_path)
    print(f"[*] Scanning dependencies in {project_path}")

    snyk_json = run_snyk_test(
        project_path,
        manifest=args.manifest,
        severity_threshold="low",
        all_projects=args.all_projects
    )

    if "error" in snyk_json:
        print(f"[ERROR] {snyk_json['error']}")
        sys.exit(2)

    vulns = parse_vulnerabilities(snyk_json)
    vulns = deduplicate_vulns(vulns)

    quality_gate = evaluate_quality_gate(vulns, args.severity_threshold, args.fail_on)
    report = generate_report(vulns, quality_gate, snyk_json, project_path)

    output_path = os.path.abspath(args.output)
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[*] Report: {output_path}")

    print(f"\n[*] Dependencies: {snyk_json.get('dependencyCount', 'N/A')}")
    print(f"[*] Vulnerabilities: {len(vulns)} (fixable: {quality_gate['fixable_count']})")
    for sev, count in sorted(quality_gate["severity_counts"].items(),
                             key=lambda x: SEVERITY_ORDER.get(x[0], 3)):
        print(f"    {sev.upper()}: {count}")

    if quality_gate["passed"]:
        print(f"\n[PASS] Quality gate passed.")
    else:
        print(f"\n[FAIL] {quality_gate['blocking_count']} blocking vulnerabilities.")
        for d in quality_gate["blocking_details"][:10]:
            fix_info = f"fix: {d['fix']}" if d['upgradable'] else "no fix available"
            print(f"  [{d['severity'].upper()}] {d['id']}: {d['package']} ({fix_info})")

    if args.monitor:
        print("\n[*] Running Snyk monitor for continuous tracking...")
        subprocess.run(
            ["snyk", "monitor", "--project-name", os.path.basename(project_path)],
            cwd=project_path
        )

    if args.fail_on_findings and not quality_gate["passed"]:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-threat-modeling-with-owasp-threat-dragon/SKILL.md =====
---
name: performing-threat-modeling-with-owasp-threat-dragon
description: >-
  Uses OWASP Threat Dragon (web or desktop) to build data flow diagrams,
  identify threats with STRIDE, LINDDUN, CIA, DIE, or PLOT4ai methodologies
  via its auto-generation rule engine, and produce PDF threat model reports.
  Use during secure design review of an application architecture to build a
  formal threat model and document mitigations for GRC compliance.
domain: cybersecurity
subdomain: devsecops
tags:
- threat-modeling
- owasp
- threat-dragon
- stride
- linddun
- secure-design
- dfd
- data-flow
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
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
---

# Performing Threat Modeling with OWASP Threat Dragon

## Overview

OWASP Threat Dragon is an open-source threat modeling tool that enables security teams and developers to create threat model diagrams, identify threats using established methodologies (STRIDE, LINDDUN, CIA, DIE, PLOT4ai), and generate comprehensive reports. Threat Dragon runs as both a web application and desktop application (Windows, macOS, Linux), supporting distributed teams working collaboratively on threat models. Version 2.x provides drag-and-drop diagram creation, an auto-generation rule engine for threats and mitigations, and PDF report output for documentation and GRC compliance.


## When to Use

- When conducting security assessments that involve performing threat modeling with owasp threat dragon
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- OWASP Threat Dragon desktop application or web instance
- Understanding of data flow diagram (DFD) notation
- Familiarity with STRIDE or LINDDUN threat classification
- Application architecture documentation and network diagrams
- Stakeholder access for design review sessions

## Threat Modeling Methodologies

### STRIDE

| Category | Threat Type | Description | Example |
|----------|-------------|-------------|---------|
| S | Spoofing | Impersonating a user or system | Stolen session tokens |
| T | Tampering | Modifying data in transit or at rest | SQL injection altering records |
| R | Repudiation | Denying an action occurred | Missing audit logs |
| I | Information Disclosure | Exposing sensitive data | API returning excessive fields |
| D | Denial of Service | Making a service unavailable | Resource exhaustion attack |
| E | Elevation of Privilege | Gaining unauthorized access | Broken access control |

### LINDDUN (Privacy-Focused)

| Category | Threat Type | Description |
|----------|-------------|-------------|
| L | Linkability | Associating data items across contexts |
| I | Identifiability | Identifying an individual from data |
| N | Non-repudiation | Inability to deny an action (privacy risk) |
| D | Detectability | Determining if data about a subject exists |
| D | Disclosure | Exposing personal information |
| U | Unawareness | User unaware of data collection |
| N | Non-compliance | Violating privacy regulations |

## Workflow

### Step 1 --- Install Threat Dragon

**Desktop Application:**
Download the installer from the [OWASP Threat Dragon releases](https://github.com/OWASP/threat-dragon/releases) page for Windows (.exe), macOS (.dmg), or Linux (.AppImage/.deb/.rpm).

**Web Application (Docker):**
```bash
docker run -p 3000:3000 \
  -e ENCRYPTION_JWT_SIGNING_KEY=$(openssl rand -hex 32) \
  -e ENCRYPTION_JWT_REFRESH_SIGNING_KEY=$(openssl rand -hex 32) \
  -e ENCRYPTION_KEYS='[{"isPrimary":true,"id":0,"value":"'$(openssl rand -hex 16)'"}]' \
  -e NODE_ENV=production \
  owasp/threat-dragon:latest
```

### Step 2 --- Define the Scope

Before creating diagrams, document the scope:
- System name and description
- Assets being protected (user data, credentials, payment info)
- External dependencies (third-party APIs, cloud services)
- Compliance requirements (GDPR, HIPAA, PCI DSS)
- Trust boundaries (network segments, authentication zones)

### Step 3 --- Create Data Flow Diagrams

In Threat Dragon, create a new threat model and add diagrams using the following DFD elements:

**Processes**: Applications, microservices, API endpoints that transform data. Represented as circles/rounded rectangles.

**Data Stores**: Databases, file systems, caches, message queues that persist data. Represented as parallel lines.

**External Entities**: Users, external systems, third-party services outside the trust boundary. Represented as rectangles.

**Data Flows**: Communication channels between elements showing data direction. Represented as arrows with labels describing the data.

**Trust Boundaries**: Dashed lines separating zones of different trust levels (internet/DMZ/internal network, user/admin).

### Step 4 --- Identify Threats

For each DFD element, apply the STRIDE methodology:

| Element Type | Applicable STRIDE Categories |
|-------------|------------------------------|
| External Entity | Spoofing, Repudiation |
| Process | Spoofing, Tampering, Repudiation, Information Disclosure, DoS, Elevation of Privilege |
| Data Store | Tampering, Information Disclosure, DoS |
| Data Flow | Tampering, Information Disclosure, DoS |

Threat Dragon's rule engine automatically suggests threats based on element types. Review each suggestion and mark as:
- **Mitigated**: Existing controls address the threat
- **Not Applicable**: Threat does not apply to this context
- **Open**: Threat needs to be addressed (assign priority and owner)

### Step 5 --- Define Mitigations

For each open threat, document:
- Mitigation strategy (prevent, detect, respond, transfer)
- Specific technical controls (encryption, authentication, rate limiting)
- Owner responsible for implementation
- Priority and timeline for remediation

### Step 6 --- Generate Reports

Threat Dragon produces PDF reports containing:
- Executive summary of the threat model
- Data flow diagrams with annotations
- Threat inventory with severity ratings
- Mitigation status and recommendations
- Compliance mapping where applicable

### Step 7 --- Integrate into SDLC

- Conduct threat modeling during the design phase of new features
- Update threat models when architecture changes occur
- Review threat models during security design reviews
- Store threat model files in version control alongside code
- Reference threat model findings in security acceptance criteria

## Threat Model File Format

Threat Dragon uses JSON format for threat models, enabling version control and programmatic manipulation:

```json
{
  "version": "2.2.0",
  "summary": {
    "title": "E-Commerce Application",
    "owner": "Security Team",
    "description": "Threat model for the checkout flow"
  },
  "detail": {
    "contributors": [
      {"name": "Security Architect"}
    ],
    "diagrams": [
      {
        "id": 0,
        "title": "Checkout Flow",
        "diagramType": "STRIDE",
        "cells": []
      }
    ]
  }
}
```

## CycloneDX TMBOM Integration

Threat Dragon participates in the CycloneDX Threat Model Bill of Materials (TMBOM) effort, enabling export to a common format that can be consumed by other threat modeling tools and GRC platforms, preventing vendor lock-in.

## Best Practices

1. **Start simple**: Begin with high-level DFDs (Level 0) before decomposing into detailed diagrams
2. **Involve developers**: Include development team members in threat modeling sessions for realistic threat assessment
3. **Time-box sessions**: Limit initial sessions to 90 minutes; iterate in follow-up sessions
4. **Prioritize by risk**: Use severity ratings (Critical, High, Medium, Low) to prioritize mitigations
5. **Living documents**: Treat threat models as living documents that evolve with the system
6. **Automate where possible**: Use the rule engine for initial threat generation, then refine manually

## References

- [OWASP Threat Dragon](https://owasp.org/www-project-threat-dragon/)
- [Threat Dragon GitHub Repository](https://github.com/OWASP/threat-dragon)
- [OWASP Threat Modeling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)
- [STRIDE Threat Model](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats)
- [LINDDUN Privacy Threat Modeling](https://www.linddun.org/)

===== END FILE =====


===== FILE: skills/performing-threat-modeling-with-owasp-threat-dragon/LICENSE =====

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


===== FILE: skills/performing-threat-modeling-with-owasp-threat-dragon/assets/template.md =====
# Threat Modeling Session Template

## System Information

| Field | Value |
|-------|-------|
| System Name | |
| Version/Release | |
| Owner | |
| Date | |
| Participants | |
| Methodology | [ ] STRIDE [ ] LINDDUN [ ] CIA [ ] DIE |

## Scope Definition

**Assets to Protect:**
1.
2.
3.

**Trust Boundaries:**
1.
2.

**External Dependencies:**
1.
2.

## Threat Register

| ID | Element | STRIDE Category | Threat Description | Severity | Status | Mitigation | Owner |
|----|---------|-----------------|-------------------|----------|--------|------------|-------|
| T-001 | | | | | [ ] Open [ ] Mitigated [ ] N/A | | |
| T-002 | | | | | [ ] Open [ ] Mitigated [ ] N/A | | |

## Action Items

| Item | Description | Owner | Due Date | Status |
|------|-------------|-------|----------|--------|
| | | | | |

## Review Schedule

- [ ] Initial threat model created
- [ ] Peer review completed
- [ ] Architecture review board sign-off
- [ ] Quarterly review scheduled
- [ ] Model updated for last architecture change

===== END FILE =====


===== FILE: skills/performing-threat-modeling-with-owasp-threat-dragon/references/api-reference.md =====
# API Reference: Threat Modeling with OWASP Threat Dragon

## Threat Dragon JSON Model Structure

| Field | Description |
|-------|-------------|
| `version` | Threat Dragon version (e.g., "2.2.0") |
| `summary.title` | Threat model name |
| `summary.owner` | Model owner |
| `detail.diagrams[]` | Array of DFD diagrams |
| `detail.diagrams[].cells[]` | DFD elements within a diagram |
| `detail.diagrams[].diagramType` | Methodology (STRIDE, LINDDUN, CIA) |

## DFD Element Types

| Type | Threat Dragon Class | STRIDE Categories |
|------|--------------------|--------------------|
| Process | `tm.Process` | S, T, R, I, D, E |
| Data Store | `tm.Store` | T, I, D |
| Data Flow | `tm.Flow` | T, I, D |
| External Entity | `tm.Actor` | S, R |
| Trust Boundary | `tm.Boundary` | N/A |

## STRIDE Categories

| Letter | Threat | Mitigation |
|--------|--------|------------|
| S | Spoofing | Strong authentication, MFA |
| T | Tampering | Integrity checks, HMAC |
| R | Repudiation | Audit logging |
| I | Information Disclosure | Encryption, least privilege |
| D | Denial of Service | Rate limiting, auto-scaling |
| E | Elevation of Privilege | RBAC, authorization checks |

## Threat Status Values

| Status | Description |
|--------|-------------|
| Open | Threat needs mitigation |
| Mitigated | Controls address the threat |
| Not Applicable | Threat does not apply |

## Docker Deployment

```bash
docker run -p 3000:3000 \
  -e ENCRYPTION_JWT_SIGNING_KEY=$(openssl rand -hex 32) \
  -e ENCRYPTION_JWT_REFRESH_SIGNING_KEY=$(openssl rand -hex 32) \
  owasp/threat-dragon:latest
```

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `json` | stdlib | Threat Dragon model serialization |
| `uuid` | stdlib | Generate unique element IDs |

## References

- OWASP Threat Dragon: https://owasp.org/www-project-threat-dragon/
- Threat Dragon GitHub: https://github.com/OWASP/threat-dragon
- STRIDE Model: https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats
- LINDDUN: https://www.linddun.org/

===== END FILE =====


===== FILE: skills/performing-threat-modeling-with-owasp-threat-dragon/references/standards.md =====
# Standards Reference for Threat Modeling

## OWASP Threat Modeling Process

1. **Decompose the application**: Create DFDs showing data flows, trust boundaries, entry points
2. **Determine and rank threats**: Apply STRIDE per element, rank by DREAD or risk matrix
3. **Determine countermeasures and mitigations**: Map threats to controls
4. **Review and validate**: Peer review the model, validate against architecture

## NIST SP 800-154: Guide to Data-Centric System Threat Modeling

- Identify data assets and their sensitivity levels
- Map data flows through system components
- Identify threat actors and attack vectors targeting data
- Assess risk based on data exposure and impact
- Document countermeasures protecting data at rest, in transit, and in use

## ISO 27005 Risk Assessment Alignment

| ISO 27005 Step | Threat Dragon Activity |
|----------------|----------------------|
| Context establishment | Define system scope and trust boundaries |
| Risk identification | STRIDE threat enumeration per DFD element |
| Risk analysis | Severity rating and likelihood assessment |
| Risk evaluation | Prioritize threats by risk score |
| Risk treatment | Define mitigations (mitigate, accept, transfer, avoid) |

## STRIDE-per-Element Mapping

| DFD Element | S | T | R | I | D | E |
|-------------|---|---|---|---|---|---|
| External Entity | x | | x | | | |
| Process | x | x | x | x | x | x |
| Data Store | | x | | x | x | |
| Data Flow | | x | | x | x | |

## Threat Severity Rating Scale

| Rating | Score | Description |
|--------|-------|-------------|
| Critical | 9-10 | Immediate exploitation possible, severe business impact |
| High | 7-8 | Likely exploitation, significant business impact |
| Medium | 4-6 | Possible exploitation, moderate business impact |
| Low | 1-3 | Unlikely exploitation or minimal impact |

===== END FILE =====


===== FILE: skills/performing-threat-modeling-with-owasp-threat-dragon/references/workflows.md =====
# Threat Modeling Workflows

## Workflow 1: Feature-Level Threat Modeling

```
Product team proposes new feature
       |
Security champion schedules threat modeling session
       |
Gather architecture docs, API specs, DFDs
       |
Workshop session (60-90 min):
  - Review DFD for new feature
  - Apply STRIDE to each element
  - Identify threats and rank by severity
  - Document mitigations needed
       |
Threats documented in Threat Dragon
       |
Mitigation tasks created in backlog
       |
Security acceptance criteria added to user stories
       |
Feature development includes mitigations
       |
Threat model updated during code review
```

## Workflow 2: Architecture Review Threat Model

```
New system or major refactor proposed
       |
Create Level 0 DFD (context diagram)
       |
Identify trust boundaries and data sensitivity
       |
Decompose into Level 1 DFDs per subsystem
       |
Apply STRIDE per element in each diagram
       |
Auto-generate threats using Threat Dragon rule engine
       |
Review and refine auto-generated threats
       |
Assign severity ratings
       |
Document existing controls as mitigations
       |
Identify gaps (open threats without mitigations)
       |
Generate PDF report for architecture review board
       |
Remediation plan approved and tracked
```

## Workflow 3: Continuous Threat Modeling in Agile

```
Sprint planning identifies security-relevant stories
       |
Pull existing threat model from version control
       |
Update DFD to reflect planned changes
       |
Quick STRIDE review (30 min) focused on changes
       |
New threats added, mitigations defined
       |
Threat model committed alongside code changes
       |
Sprint retrospective reviews threat model accuracy
       |
Quarterly full threat model review cycle
```

===== END FILE =====


===== FILE: skills/performing-threat-modeling-with-owasp-threat-dragon/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for threat modeling with OWASP Threat Dragon.

Programmatically creates Threat Dragon JSON threat models, applies
STRIDE analysis to DFD elements, manages threat inventory, and
generates summary reports for security design reviews.
"""

import json
import sys
import uuid
from datetime import datetime


STRIDE_BY_ELEMENT = {
    "process": ["Spoofing", "Tampering", "Repudiation",
                "Information Disclosure", "Denial of Service",
                "Elevation of Privilege"],
    "data_store": ["Tampering", "Information Disclosure",
                   "Denial of Service"],
    "data_flow": ["Tampering", "Information Disclosure",
                  "Denial of Service"],
    "external_entity": ["Spoofing", "Repudiation"],
}

STRIDE_MITIGATIONS = {
    "Spoofing": ["Implement strong authentication (MFA)",
                 "Use mutual TLS for service-to-service"],
    "Tampering": ["Use integrity checks (HMAC, digital signatures)",
                  "Implement input validation"],
    "Repudiation": ["Enable comprehensive audit logging",
                    "Use tamper-evident log storage"],
    "Information Disclosure": ["Encrypt data at rest and in transit",
                               "Implement least-privilege access"],
    "Denial of Service": ["Implement rate limiting",
                          "Use auto-scaling and circuit breakers"],
    "Elevation of Privilege": ["Enforce RBAC and least privilege",
                               "Validate authorization on every request"],
}


class ThreatModelAgent:
    """Creates and manages OWASP Threat Dragon threat models."""

    def __init__(self, title, owner="Security Team", description=""):
        self.model = {
            "version": "2.2.0",
            "summary": {
                "title": title,
                "owner": owner,
                "description": description,
                "id": 0,
            },
            "detail": {
                "contributors": [],
                "diagrams": [],
                "diagramTop": 0,
                "reviewer": "",
                "threatTop": 0,
            },
        }
        self.threats = []
        self.threat_counter = 0

    def add_diagram(self, title, diagram_type="STRIDE"):
        """Add a new data flow diagram to the threat model."""
        diagram_id = len(self.model["detail"]["diagrams"])
        diagram = {
            "id": diagram_id,
            "title": title,
            "diagramType": diagram_type,
            "placeholder": f"New {diagram_type} diagram",
            "thumbnail": "",
            "version": "2.2.0",
            "cells": [],
        }
        self.model["detail"]["diagrams"].append(diagram)
        return diagram_id

    def add_element(self, diagram_id, element_type, name,
                    x=100, y=100, description=""):
        """Add a DFD element to a diagram."""
        element_id = str(uuid.uuid4())
        type_map = {
            "process": "tm.Process",
            "data_store": "tm.Store",
            "data_flow": "tm.Flow",
            "external_entity": "tm.Actor",
            "trust_boundary": "tm.Boundary",
        }
        cell = {
            "type": type_map.get(element_type, "tm.Process"),
            "id": element_id,
            "name": name,
            "description": description,
            "position": {"x": x, "y": y},
            "size": {"width": 100, "height": 60},
            "threats": [],
            "hasOpenThreats": False,
        }
        self.model["detail"]["diagrams"][diagram_id]["cells"].append(cell)
        return element_id

    def apply_stride(self, diagram_id, element_id, element_type):
        """Apply STRIDE analysis to a DFD element and generate threats."""
        categories = STRIDE_BY_ELEMENT.get(element_type, [])
        generated = []
        diagram = self.model["detail"]["diagrams"][diagram_id]
        element = next((c for c in diagram["cells"] if c["id"] == element_id), None)
        if not element:
            return []

        for category in categories:
            self.threat_counter += 1
            threat = {
                "id": str(self.threat_counter),
                "title": f"{category} - {element['name']}",
                "type": category,
                "status": "Open",
                "severity": "Medium",
                "description": f"Potential {category.lower()} threat "
                               f"against {element['name']}",
                "mitigation": "; ".join(
                    STRIDE_MITIGATIONS.get(category, ["Review required"])),
                "modelType": "STRIDE",
                "element_id": element_id,
            }
            element["threats"].append(threat)
            element["hasOpenThreats"] = True
            self.threats.append(threat)
            generated.append(threat)

        self.model["detail"]["threatTop"] = self.threat_counter
        return generated

    def update_threat_status(self, threat_id, status, mitigation=None):
        """Update a threat's status (Open, Mitigated, Not Applicable)."""
        for threat in self.threats:
            if threat["id"] == str(threat_id):
                threat["status"] = status
                if mitigation:
                    threat["mitigation"] = mitigation
                return threat
        return None

    def get_threat_summary(self):
        """Summarize threats by status and category."""
        summary = {"total": len(self.threats), "by_status": {},
                   "by_type": {}, "by_severity": {}}
        for t in self.threats:
            summary["by_status"][t["status"]] = \
                summary["by_status"].get(t["status"], 0) + 1
            summary["by_type"][t["type"]] = \
                summary["by_type"].get(t["type"], 0) + 1
            summary["by_severity"][t["severity"]] = \
                summary["by_severity"].get(t["severity"], 0) + 1
        return summary

    def save_model(self, output_path):
        """Save the threat model as Threat Dragon JSON file."""
        with open(output_path, "w") as f:
            json.dump(self.model, f, indent=2)
        return output_path

    def generate_report(self):
        """Generate threat model assessment report."""
        summary = self.get_threat_summary()
        report = {
            "title": self.model["summary"]["title"],
            "owner": self.model["summary"]["owner"],
            "report_date": datetime.utcnow().isoformat(),
            "diagrams": len(self.model["detail"]["diagrams"]),
            "threat_summary": summary,
            "open_threats": [t for t in self.threats if t["status"] == "Open"],
            "mitigated_threats": [t for t in self.threats
                                  if t["status"] == "Mitigated"],
        }
        print(json.dumps(report, indent=2))
        return report


def main():
    title = sys.argv[1] if len(sys.argv) > 1 else "Sample Application"
    output = sys.argv[2] if len(sys.argv) > 2 else "./threat_model.json"

    agent = ThreatModelAgent(title, owner="Security Team",
                             description="Automated threat model")
    did = agent.add_diagram("Main Data Flow")
    web = agent.add_element(did, "external_entity", "Web Browser", 50, 50)
    api = agent.add_element(did, "process", "API Gateway", 250, 50)
    db = agent.add_element(did, "data_store", "Database", 450, 50)
    flow1 = agent.add_element(did, "data_flow", "HTTPS Request", 150, 100)

    agent.apply_stride(did, web, "external_entity")
    agent.apply_stride(did, api, "process")
    agent.apply_stride(did, db, "data_store")
    agent.apply_stride(did, flow1, "data_flow")

    agent.save_model(output)
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-threat-modeling-with-owasp-threat-dragon/scripts/process.py =====
#!/usr/bin/env python3
"""
OWASP Threat Dragon Model Analyzer

Parses Threat Dragon JSON threat model files and generates
summary statistics, coverage reports, and mitigation gap analysis.
"""

import json
import sys
import os
from collections import defaultdict
from datetime import datetime


def load_threat_model(filepath: str) -> dict:
    with open(filepath) as f:
        return json.load(f)


def extract_threats(model: dict) -> list:
    threats = []
    detail = model.get("detail", {})
    for diagram in detail.get("diagrams", []):
        diagram_title = diagram.get("title", "Untitled")
        for cell in diagram.get("cells", []):
            cell_data = cell.get("data", {})
            cell_threats = cell_data.get("threats", [])
            for threat in cell_threats:
                threats.append({
                    "diagram": diagram_title,
                    "element": cell_data.get("name", cell.get("id", "unknown")),
                    "element_type": cell_data.get("type", "unknown"),
                    "title": threat.get("title", ""),
                    "description": threat.get("description", ""),
                    "severity": threat.get("severity", "Unknown"),
                    "status": threat.get("status", "Open"),
                    "type": threat.get("type", ""),
                    "mitigation": threat.get("mitigation", ""),
                    "model_type": threat.get("modelType", "STRIDE"),
                })
    return threats


def analyze_coverage(threats: list) -> dict:
    coverage = {
        "total_threats": len(threats),
        "by_status": defaultdict(int),
        "by_severity": defaultdict(int),
        "by_type": defaultdict(int),
        "by_element_type": defaultdict(int),
        "mitigated_count": 0,
        "open_count": 0,
        "not_applicable_count": 0,
        "with_mitigation_text": 0,
    }

    for threat in threats:
        status = threat["status"]
        coverage["by_status"][status] += 1
        coverage["by_severity"][threat["severity"]] += 1
        coverage["by_type"][threat["type"]] += 1
        coverage["by_element_type"][threat["element_type"]] += 1

        if status.lower() == "mitigated":
            coverage["mitigated_count"] += 1
        elif status.lower() == "open":
            coverage["open_count"] += 1
        elif status.lower() in ("not applicable", "n/a"):
            coverage["not_applicable_count"] += 1

        if threat["mitigation"].strip():
            coverage["with_mitigation_text"] += 1

    coverage["by_status"] = dict(coverage["by_status"])
    coverage["by_severity"] = dict(coverage["by_severity"])
    coverage["by_type"] = dict(coverage["by_type"])
    coverage["by_element_type"] = dict(coverage["by_element_type"])
    return coverage


def identify_gaps(threats: list) -> list:
    gaps = []
    for threat in threats:
        if threat["status"].lower() == "open" and not threat["mitigation"].strip():
            gaps.append({
                "diagram": threat["diagram"],
                "element": threat["element"],
                "threat_title": threat["title"],
                "severity": threat["severity"],
                "type": threat["type"],
            })
    return sorted(gaps, key=lambda g: {"Critical": 0, "High": 1, "Medium": 2, "Low": 3}.get(g["severity"], 4))


def stride_coverage_check(threats: list) -> dict:
    stride_categories = {
        "Spoofing": False,
        "Tampering": False,
        "Repudiation": False,
        "Information disclosure": False,
        "Denial of service": False,
        "Elevation of privilege": False,
    }
    for threat in threats:
        threat_type = threat.get("type", "")
        for category in stride_categories:
            if category.lower() in threat_type.lower():
                stride_categories[category] = True
    return stride_categories


def print_report(model: dict, coverage: dict, gaps: list, stride: dict) -> None:
    summary = model.get("summary", {})
    print(f"\n{'='*60}")
    print(f"Threat Model Analysis Report")
    print(f"{'='*60}")
    print(f"Title: {summary.get('title', 'Unknown')}")
    print(f"Owner: {summary.get('owner', 'Unknown')}")
    print(f"Description: {summary.get('description', '')}")
    print(f"Generated: {datetime.utcnow().isoformat()}Z")

    diagrams = model.get("detail", {}).get("diagrams", [])
    print(f"\nDiagrams: {len(diagrams)}")
    for d in diagrams:
        print(f"  - {d.get('title', 'Untitled')} ({d.get('diagramType', 'Unknown')} type)")

    print(f"\nThreat Summary:")
    print(f"  Total threats: {coverage['total_threats']}")
    print(f"  Mitigated: {coverage['mitigated_count']}")
    print(f"  Open: {coverage['open_count']}")
    print(f"  Not Applicable: {coverage['not_applicable_count']}")
    print(f"  With mitigation documented: {coverage['with_mitigation_text']}")

    if coverage["total_threats"] > 0:
        mitigation_rate = coverage["mitigated_count"] / coverage["total_threats"] * 100
        print(f"  Mitigation rate: {mitigation_rate:.1f}%")

    print(f"\nBy Severity:")
    for sev in ["Critical", "High", "Medium", "Low", "Unknown"]:
        count = coverage["by_severity"].get(sev, 0)
        if count:
            print(f"  {sev:12s}: {count}")

    print(f"\nSTRIDE Coverage:")
    for category, covered in stride.items():
        status = "COVERED" if covered else "MISSING"
        print(f"  {category:25s}: {status}")

    if gaps:
        print(f"\nMitigation Gaps ({len(gaps)} open threats without mitigations):")
        for gap in gaps:
            print(f"  [{gap['severity']}] {gap['threat_title']}")
            print(f"    Element: {gap['element']} | Diagram: {gap['diagram']}")
    else:
        print(f"\nNo mitigation gaps found.")


def main():
    if len(sys.argv) < 2:
        print("Usage: python process.py <threat_model.json>")
        print("  Analyzes an OWASP Threat Dragon JSON threat model file")
        sys.exit(1)

    filepath = sys.argv[1]
    if not os.path.exists(filepath):
        print(f"File not found: {filepath}")
        sys.exit(1)

    model = load_threat_model(filepath)
    threats = extract_threats(model)
    coverage = analyze_coverage(threats)
    gaps = identify_gaps(threats)
    stride = stride_coverage_check(threats)
    print_report(model, coverage, gaps, stride)

    output = filepath.replace(".json", "_analysis.json")
    analysis = {
        "model_title": model.get("summary", {}).get("title"),
        "analysis_date": datetime.utcnow().isoformat() + "Z",
        "coverage": coverage,
        "gaps": gaps,
        "stride_coverage": stride,
    }
    with open(output, "w") as f:
        json.dump(analysis, f, indent=2)
    print(f"\nAnalysis saved to: {output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/scanning-containers-with-trivy-in-cicd/SKILL.md =====
---
name: scanning-containers-with-trivy-in-cicd
description: 'Integrates Aqua Security''s Trivy scanner into CI/CD pipelines to detect
  OS package and application dependency CVEs, Dockerfile misconfigurations, and issues
  in filesystems or git repositories, and to enforce severity-based quality gates that
  block vulnerable images from being deployed. Use when building Docker images in
  CI/CD and needing automated vulnerability scanning and pass/fail gates before registry
  push or production deployment.

  '
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- cicd
- trivy
- container-security
- vulnerability-scanning
- secure-sdlc
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
- T1610
- T1611
---

# Scanning Containers with Trivy in CI/CD

## When to Use

- When building Docker container images in CI/CD and needing automated vulnerability scanning before registry push
- When establishing quality gates that prevent images with critical or high CVEs from reaching production
- When compliance requirements mandate vulnerability scanning of all container images before deployment
- When scanning IaC files (Dockerfiles, Kubernetes manifests) alongside container image scanning
- When needing a single tool to scan OS packages, language-specific dependencies, and misconfigurations

**Do not use** for runtime container security monitoring (use Falco), for scanning running containers in production (use runtime agents), or when only scanning application source code without containerization (use SAST tools).

## Prerequisites

- Trivy CLI installed (v0.50+) or access to aquasecurity/trivy-action GitHub Action
- Docker daemon available in CI/CD for building and scanning images
- Container registry credentials for pulling base images and pushing scanned images
- Trivy vulnerability database accessible (downloaded automatically or cached)

## Workflow

### Step 1: Configure Trivy Scanning in GitHub Actions

Set up a GitHub Actions workflow that builds a Docker image and scans it with Trivy before pushing to a container registry.

```yaml
# .github/workflows/container-security.yml
name: Container Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
    paths:
      - 'Dockerfile'
      - 'docker-compose*.yml'
      - 'src/**'
      - 'requirements*.txt'
      - 'package*.json'

jobs:
  build-and-scan:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t app:${{ github.sha }} .

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@0.28.0
        with:
          image-ref: 'app:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
          ignore-unfixed: true

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'
          category: 'trivy-container'

      - name: Run Trivy misconfiguration scanner
        uses: aquasecurity/trivy-action@0.28.0
        with:
          scan-type: 'config'
          scan-ref: '.'
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
```

### Step 2: Scan Dockerfiles for Misconfigurations

Trivy detects common Dockerfile security issues such as running as root, using latest tags, and exposing unnecessary ports.

```bash
# Scan Dockerfile for misconfigurations
trivy config --severity HIGH,CRITICAL ./Dockerfile

# Scan with custom policy directory
trivy config --policy ./security-policies --severity MEDIUM,HIGH,CRITICAL .

# Example secure Dockerfile practices Trivy checks for:
# - USER instruction present (not running as root)
# - HEALTHCHECK instruction defined
# - Base image uses specific tag, not :latest
# - No secrets in ENV or ARG instructions
# - COPY preferred over ADD
```

### Step 3: Integrate with GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - build
  - scan
  - push

variables:
  TRIVY_CACHE_DIR: .trivycache/

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker save $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -o image.tar
  artifacts:
    paths:
      - image.tar

trivy-scan:
  stage: scan
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  cache:
    paths:
      - .trivycache/
  script:
    - trivy image
        --input image.tar
        --exit-code 1
        --severity CRITICAL,HIGH
        --ignore-unfixed
        --format json
        --output trivy-report.json
    - trivy image
        --input image.tar
        --severity CRITICAL,HIGH,MEDIUM
        --format table
  artifacts:
    reports:
      container_scanning: trivy-report.json
    paths:
      - trivy-report.json
  allow_failure: false

push:
  stage: push
  needs: [trivy-scan]
  script:
    - docker load -i image.tar
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

### Step 4: Configure Trivy Ignore and Exception Handling

Manage false positives and accepted risks through Trivy's ignore file and VEX statements.

```yaml
# .trivyignore.yaml
vulnerabilities:
  - id: CVE-2023-44487    # HTTP/2 rapid reset - mitigated at load balancer
    statement: "Mitigated by WAF rate limiting at ingress layer"
    expires: 2026-06-01

  - id: CVE-2024-21626    # runc container escape - patched in base image update
    statement: "Tracked in JIRA-SEC-1234, base image update scheduled"
    expires: 2026-03-15

misconfigurations:
  - id: DS002             # User not set - required for init containers
    paths:
      - "docker/init-container/Dockerfile"
    statement: "Init container requires root for volume permission setup"
```

### Step 5: Implement Database Caching and Offline Scanning

Cache the Trivy vulnerability database in CI/CD to reduce scan times and enable air-gapped environments.

```yaml
# GitHub Actions with database caching
- name: Cache Trivy DB
  uses: actions/cache@v4
  with:
    path: /tmp/trivy-db
    key: trivy-db-${{ hashFiles('.github/workflows/container-security.yml') }}
    restore-keys: trivy-db-

- name: Run Trivy with cached DB
  uses: aquasecurity/trivy-action@0.28.0
  with:
    image-ref: 'app:${{ github.sha }}'
    cache-dir: /tmp/trivy-db
    format: 'json'
    output: 'trivy-results.json'
    severity: 'CRITICAL,HIGH'
    exit-code: '1'
```

```bash
# Air-gapped: Download DB manually and mount
trivy image --download-db-only --cache-dir /path/to/cache
# Transfer cache to air-gapped system
trivy image --skip-db-update --cache-dir /path/to/cache myimage:tag
```

### Step 6: Generate SBOM and Scan for License Compliance

Use Trivy to generate Software Bill of Materials alongside vulnerability scanning.

```bash
# Generate SBOM in CycloneDX format
trivy image --format cyclonedx --output sbom.cdx.json app:latest

# Generate SBOM in SPDX format
trivy image --format spdx-json --output sbom.spdx.json app:latest

# Scan SBOM for vulnerabilities (decouple generation from scanning)
trivy sbom sbom.cdx.json --severity CRITICAL,HIGH

# Scan with license detection
trivy image --scanners vuln,license --severity HIGH,CRITICAL app:latest
```

## Key Concepts

| Term | Definition |
|------|------------|
| CVE | Common Vulnerabilities and Exposures — standardized identifiers for publicly known security vulnerabilities |
| Vulnerability DB | Trivy's regularly updated database aggregating CVE data from NVD, vendor advisories, and language-specific sources |
| Misconfiguration | Security-relevant configuration issue in Dockerfiles, Kubernetes manifests, or IaC templates |
| SBOM | Software Bill of Materials — complete inventory of all components and dependencies in a container image |
| Ignore Unfixed | Flag to skip CVEs without available patches, reducing noise from vulnerabilities with no actionable fix |
| VEX | Vulnerability Exploitability eXchange — machine-readable statements about whether a vulnerability is exploitable in context |
| Exit Code | Non-zero return code from Trivy when findings exceed the severity threshold, used to fail CI/CD pipelines |

## Tools & Systems

- **Trivy**: Open-source vulnerability scanner by Aqua Security supporting images, filesystems, repos, and IaC
- **trivy-action**: Official GitHub Action for running Trivy scans in GitHub Actions workflows
- **Trivy Operator**: Kubernetes operator that continuously scans cluster workloads with Trivy
- **Grype**: Alternative image scanner by Anchore for comparison and validation of scan results
- **Harbor**: Container registry with built-in Trivy integration for automatic image scanning on push

## Common Scenarios

### Scenario: Multi-Stage Build with Separate Scan and Push

**Context**: A team builds multi-stage Docker images and needs to scan the final production image before pushing to ECR, while also scanning the build stage for supply chain risks.

**Approach**:
1. Build the Docker image with `--target production` for the final stage
2. Run Trivy with `--severity CRITICAL,HIGH --exit-code 1 --ignore-unfixed` to block on exploitable issues
3. Generate an SBOM in CycloneDX format and store as a build artifact
4. Upload SARIF results to GitHub Security tab for visibility
5. Only push to ECR if the Trivy scan exits with code 0
6. Tag the pushed image with the scan timestamp and Trivy DB version for audit traceability

**Pitfalls**: Scanning only the final stage misses vulnerable packages that were present in build stages and may have influenced the build. Run `trivy fs` on the build context separately. Caching the Trivy DB too aggressively (weekly) means newly published CVEs take days to appear in scans.

## Output Format

```
Trivy Container Scan Report
=============================
Image: app:a1b2c3d4
Base Image: python:3.12-slim-bookworm
Scan Date: 2026-02-23
DB Version: 2026-02-23T00:15:00Z

VULNERABILITY SUMMARY:
  Total: 47
  Critical: 2
  High: 5
  Medium: 18
  Low: 22
  Unfixed: 8 (excluded from gate)

CRITICAL FINDINGS:
  CVE-2025-12345  libssl3    3.0.11-1  3.0.13-1  OpenSSL buffer overflow
  CVE-2025-67890  curl       7.88.1-10 7.88.1-12 curl HSTS bypass

HIGH FINDINGS:
  CVE-2025-11111  zlib1g     1.2.13    1.2.13.1  zlib heap buffer overflow
  CVE-2025-22222  python3.12 3.12.1    3.12.3    CPython path traversal
  CVE-2025-33333  requests   2.31.0    2.32.0    requests SSRF in redirects

MISCONFIGURATION:
  DS002  [HIGH]   Dockerfile: USER instruction not set (running as root)
  DS026  [MEDIUM] Dockerfile: No HEALTHCHECK defined

QUALITY GATE: FAILED (2 Critical, 5 High findings)
```

===== END FILE =====


===== FILE: skills/scanning-containers-with-trivy-in-cicd/LICENSE =====

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


===== FILE: skills/scanning-containers-with-trivy-in-cicd/assets/template.md =====
# Trivy Container Scanning Templates

## GitHub Actions: Full Container Security Pipeline

```yaml
# .github/workflows/container-security.yml
name: Container Security

on:
  push:
    branches: [main]
  pull_request:
    paths: ['Dockerfile', 'docker-compose*.yml', 'src/**']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-scan-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      security-events: write

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build image
        uses: docker/build-push-action@v6
        with:
          context: .
          load: true
          tags: ${{ env.IMAGE_NAME }}:scan
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Cache Trivy DB
        uses: actions/cache@v4
        with:
          path: /tmp/trivy
          key: trivy-db-${{ github.run_id }}
          restore-keys: trivy-db-

      - name: Trivy vulnerability scan
        uses: aquasecurity/trivy-action@0.28.0
        with:
          image-ref: ${{ env.IMAGE_NAME }}:scan
          format: sarif
          output: trivy-vuln.sarif
          severity: CRITICAL,HIGH
          exit-code: '1'
          ignore-unfixed: true
          cache-dir: /tmp/trivy

      - name: Upload vulnerability SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: trivy-vuln.sarif
          category: trivy-vulnerabilities

      - name: Trivy misconfiguration scan
        uses: aquasecurity/trivy-action@0.28.0
        with:
          scan-type: config
          scan-ref: .
          format: sarif
          output: trivy-config.sarif
          severity: CRITICAL,HIGH
          exit-code: '1'

      - name: Upload config SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: trivy-config.sarif
          category: trivy-misconfigurations

      - name: Generate SBOM
        uses: aquasecurity/trivy-action@0.28.0
        with:
          image-ref: ${{ env.IMAGE_NAME }}:scan
          format: cyclonedx
          output: sbom.cdx.json

      - name: Upload SBOM artifact
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.cdx.json

      - name: Login to GHCR
        if: github.event_name == 'push'
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Push image
        if: github.event_name == 'push'
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
```

## Trivy Ignore File Template

```yaml
# .trivyignore.yaml
vulnerabilities:
  # Accepted risk: mitigated at infrastructure level
  - id: CVE-YYYY-NNNNN
    statement: "Mitigated by WAF rules. Risk accepted by security team."
    expires: 2026-12-31

misconfigurations:
  # Init containers require root
  - id: DS002
    paths:
      - "docker/init/*.Dockerfile"
    statement: "Init containers need root for volume permissions"
```

## Secure Dockerfile Template

```dockerfile
# syntax=docker/dockerfile:1

# Build stage
FROM python:3.12-slim-bookworm AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Production stage
FROM python:3.12-slim-bookworm AS production

# Security: Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser -s /bin/false appuser

# Security: Install only runtime dependencies, remove cache
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      libpq5 \
    && rm -rf /var/lib/apt/lists/*

# Copy installed packages from builder
COPY --from=builder /install /usr/local

# Copy application code
WORKDIR /app
COPY --chown=appuser:appuser src/ ./src/

# Security: Run as non-root
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8080/health')" || exit 1

EXPOSE 8080
ENTRYPOINT ["python", "-m", "src.main"]
```

## Trivy Operator for Kubernetes

```yaml
# Install Trivy Operator via Helm
# helm repo add aqua https://aquasecurity.github.io/helm-charts/
# helm install trivy-operator aqua/trivy-operator \
#   --namespace trivy-system --create-namespace \
#   --set trivy.severity=CRITICAL,HIGH

# Sample VulnerabilityReport CRD
apiVersion: aquasecurity.github.io/v1alpha1
kind: ClusterComplianceReport
metadata:
  name: cis-benchmark
spec:
  cron: "0 */6 * * *"
  compliance:
    id: cis
    title: CIS Kubernetes Benchmark
    platform: k8s
```

===== END FILE =====


===== FILE: skills/scanning-containers-with-trivy-in-cicd/references/api-reference.md =====
# API Reference: Scanning Containers with Trivy in CI/CD

## Trivy CLI Commands

| Command | Description |
|---------|-------------|
| `trivy image <ref>` | Scan container image for vulnerabilities |
| `trivy config <path>` | Scan Dockerfiles/IaC for misconfigurations |
| `trivy fs <path>` | Scan filesystem for vulnerabilities |
| `trivy sbom <file>` | Scan existing SBOM for vulnerabilities |
| `trivy image --format sarif` | SARIF output for GitHub Security |
| `trivy image --format cyclonedx` | CycloneDX SBOM generation |
| `trivy image --exit-code 1` | Non-zero exit on findings |

## Scan Options

| Flag | Description |
|------|-------------|
| `--severity CRITICAL,HIGH` | Filter by severity level |
| `--ignore-unfixed` | Skip CVEs without patches |
| `--scanners vuln,misconfig,secret` | Select scanner types |
| `--format json/sarif/cyclonedx` | Output format |
| `--exit-code 1` | Fail pipeline on findings |
| `--skip-db-update` | Use cached vulnerability DB |
| `--cache-dir <path>` | Set database cache directory |

## CI/CD Integration

| Platform | Method |
|----------|--------|
| GitHub Actions | `aquasecurity/trivy-action@v0.28.0` |
| GitLab CI | `aquasec/trivy:latest` Docker image |
| Jenkins | Trivy CLI in pipeline script |
| Azure DevOps | Trivy CLI task |

## Quality Gate Severities

| Level | CVSS Range | Default Gate Action |
|-------|-----------|-------------------|
| CRITICAL | 9.0 - 10.0 | Block deployment |
| HIGH | 7.0 - 8.9 | Block deployment |
| MEDIUM | 4.0 - 6.9 | Warn |
| LOW | 0.1 - 3.9 | Allow |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `subprocess` | stdlib | Execute trivy CLI |
| `json` | stdlib | Parse scan results |
| `pathlib` | stdlib | Output file management |

## References

- Trivy Documentation: https://trivy.dev/docs/
- Trivy GitHub Action: https://github.com/aquasecurity/trivy-action
- Trivy GitHub: https://github.com/aquasecurity/trivy

===== END FILE =====


===== FILE: skills/scanning-containers-with-trivy-in-cicd/references/standards.md =====
# Standards Reference: Container Scanning with Trivy

## NIST SP 800-190 - Application Container Security Guide

### Image Vulnerabilities (Section 3.1)
- Scan all container images for known vulnerabilities before deployment
- Establish organizational policies for maximum acceptable vulnerability severity
- Monitor images in registries for newly discovered vulnerabilities
- Maintain an up-to-date vulnerability database for scanning tools

### Image Configuration Defects (Section 3.2)
- Verify images follow CIS Docker Benchmark configuration guidelines
- Ensure images run as non-root users unless operationally required
- Remove unnecessary packages, shells, and utilities from production images

## CIS Docker Benchmark v1.6.0

### Image Level Controls
- 4.1: Ensure a user for the container has been created (maps to Trivy DS002)
- 4.2: Ensure containers use trusted base images
- 4.3: Ensure unnecessary packages are not installed in the container
- 4.6: Ensure HEALTHCHECK instructions have been added (maps to Trivy DS026)
- 4.7: Ensure update instructions are not used alone in the Dockerfile
- 4.9: Ensure COPY is used instead of ADD in Dockerfiles (maps to Trivy DS005)

## OWASP Docker Security Cheat Sheet

### Vulnerability Management
- Scan images in the CI/CD pipeline before pushing to registries
- Use `--ignore-unfixed` to focus on actionable vulnerabilities
- Implement SBOM generation for full component visibility
- Re-scan images on a schedule to catch newly published CVEs

### Dockerfile Security
- Use minimal base images (distroless, Alpine, slim variants)
- Pin base image versions with digest for reproducibility
- Run containers as non-root with explicit USER instruction
- Use multi-stage builds to exclude build tools from production images

## NIST SSDF (SP 800-218)

### PW.4: Reuse Existing, Well-Secured Software
- PW.4.1: Use automated tools to check for known vulnerabilities in dependencies
- Map to Trivy's OS package and language dependency scanning capabilities

### PS.1: Protect All Forms of Code
- PS.1.1: Store all forms of code in a code repository protected by access controls
- Container images in registries should be scanned and signed before use

## SLSA Framework Alignment

### Source Level
- Trivy filesystem scanning validates source dependencies before build
- Git repository scanning detects secrets and vulnerable dependencies in source

### Build Level
- Trivy image scanning validates the build output for vulnerabilities
- SBOM generation creates a verifiable bill of materials for the built artifact

### Deployment Level
- Admission controllers can verify Trivy scan results before pod scheduling
- Harbor registry integration enforces scan-before-pull policies

===== END FILE =====


===== FILE: skills/scanning-containers-with-trivy-in-cicd/references/workflows.md =====
# Workflow Reference: Container Scanning with Trivy in CI/CD

## Container Security Scanning Pipeline

```
Source Code Push
       │
       ▼
┌──────────────────┐
│ Build Docker      │
│ Image             │
└──────┬───────────┘
       │
       ├──────────────────────┐
       ▼                      ▼
┌──────────────┐    ┌──────────────┐
│ Trivy Image  │    │ Trivy Config │
│ Vuln Scan    │    │ Misconfig    │
└──────┬───────┘    └──────┬───────┘
       │                    │
       ▼                    ▼
┌──────────────┐    ┌──────────────┐
│ SARIF/JSON   │    │ Table/JSON   │
│ Output       │    │ Output       │
└──────┬───────┘    └──────┬───────┘
       │                    │
       └──────────┬─────────┘
                  ▼
       ┌──────────────────┐
       │ Quality Gate     │
       │ Evaluation       │
       └──────┬───────────┘
              │
    ┌─────────┴──────────┐
    ▼                    ▼
 PASS: Push to         FAIL: Block
 Registry + Tag        + Alert Team
       │
       ▼
┌──────────────┐
│ Generate     │
│ SBOM + Sign  │
└──────────────┘
```

## Trivy Scan Types Reference

### Image Scanning
```bash
# Full scan (OS + language packages)
trivy image --severity CRITICAL,HIGH --exit-code 1 myimage:tag

# OS packages only
trivy image --vuln-type os myimage:tag

# Language-specific packages only
trivy image --vuln-type library myimage:tag

# From Docker archive
trivy image --input image.tar
```

### Filesystem Scanning
```bash
# Scan project directory for vulnerable dependencies
trivy fs --severity HIGH,CRITICAL /path/to/project

# Scan specific lockfile
trivy fs --severity HIGH,CRITICAL requirements.txt
```

### Repository Scanning
```bash
# Scan remote git repository
trivy repo https://github.com/org/repo

# Scan specific branch
trivy repo --branch develop https://github.com/org/repo
```

### Configuration Scanning
```bash
# Scan Dockerfile and Kubernetes manifests
trivy config .

# Scan Terraform files
trivy config --tf-vars terraform.tfvars ./terraform/

# Scan Helm charts
trivy config ./charts/myapp/
```

## Output Format Options

| Format | Use Case | Flag |
|--------|----------|------|
| table | Human-readable terminal output | `--format table` |
| json | Programmatic processing and storage | `--format json` |
| sarif | GitHub Security tab upload | `--format sarif` |
| cyclonedx | SBOM generation (CycloneDX) | `--format cyclonedx` |
| spdx-json | SBOM generation (SPDX) | `--format spdx-json` |
| template | Custom report format | `--format template --template @template.tpl` |
| cosign-vuln | Cosign attestation format | `--format cosign-vuln` |

## Severity Threshold Matrix

| Environment | Block On | Ignore Unfixed | Rationale |
|-------------|----------|----------------|-----------|
| Development | CRITICAL | Yes | Fast feedback, focus on worst issues |
| Staging | CRITICAL, HIGH | Yes | Catch more issues before production |
| Production | CRITICAL, HIGH | No | Full visibility even for unfixed CVEs |
| Compliance | ALL | No | Complete audit trail required |

## Database Management

### Database Update Strategy
```bash
# Download DB only (for caching)
trivy image --download-db-only --cache-dir /shared/trivy-cache

# Skip DB update (use cached)
trivy image --skip-db-update --cache-dir /shared/trivy-cache myimage:tag

# Java DB for JAR scanning
trivy image --download-java-db-only --cache-dir /shared/trivy-cache
```

### Cache Locations
- Default: `~/.cache/trivy/`
- CI override: `TRIVY_CACHE_DIR=/tmp/trivy-cache`
- GitHub Actions: Use `actions/cache` with key based on date

===== END FILE =====


===== FILE: skills/scanning-containers-with-trivy-in-cicd/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for scanning containers with Trivy in CI/CD pipelines.

Runs Trivy vulnerability and misconfiguration scans against
container images and Dockerfiles, enforces severity-based
quality gates, and generates CI/CD-compatible reports.
"""

import json
import subprocess
import sys
from pathlib import Path
from datetime import datetime


class TrivyCICDAgent:
    """Integrates Trivy scanning into CI/CD workflows."""

    def __init__(self, output_dir="./trivy_cicd"):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.scan_results = []

    def _run_trivy(self, subcmd, target, extra_args=None):
        """Execute trivy CLI and return parsed results."""
        cmd = ["trivy", subcmd, "--format", "json", "--quiet"]
        if extra_args:
            cmd.extend(extra_args)
        cmd.append(target)
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
            if result.stdout.strip():
                return json.loads(result.stdout)
            return {"error": result.stderr.strip() or "No output"}
        except FileNotFoundError:
            return {"error": "trivy not found. Install from https://trivy.dev"}
        except subprocess.TimeoutExpired:
            return {"error": "Trivy scan timed out after 600 seconds"}
        except json.JSONDecodeError:
            return {"error": "Failed to parse trivy JSON output"}

    def scan_image(self, image_ref, severity="CRITICAL,HIGH", ignore_unfixed=True):
        """Scan a container image for vulnerabilities."""
        args = ["--severity", severity]
        if ignore_unfixed:
            args.append("--ignore-unfixed")
        raw = self._run_trivy("image", image_ref, args)
        if "error" in raw:
            return raw

        vulns = []
        for result in raw.get("Results", []):
            for v in result.get("Vulnerabilities", []):
                vulns.append({
                    "id": v.get("VulnerabilityID", ""),
                    "severity": v.get("Severity", "UNKNOWN"),
                    "package": v.get("PkgName", ""),
                    "installed": v.get("InstalledVersion", ""),
                    "fixed": v.get("FixedVersion", ""),
                    "title": v.get("Title", ""),
                })

        summary = {}
        for v in vulns:
            summary[v["severity"]] = summary.get(v["severity"], 0) + 1

        scan = {
            "image": image_ref,
            "scan_type": "vulnerability",
            "scan_date": datetime.utcnow().isoformat(),
            "total": len(vulns),
            "summary": summary,
            "vulnerabilities": vulns,
        }
        self.scan_results.append(scan)
        return scan

    def scan_config(self, path, severity="CRITICAL,HIGH"):
        """Scan Dockerfiles or IaC for misconfigurations."""
        args = ["--severity", severity]
        raw = self._run_trivy("config", path, args)
        if "error" in raw:
            return raw

        misconfigs = []
        for result in raw.get("Results", []):
            for mc in result.get("Misconfigurations", []):
                misconfigs.append({
                    "id": mc.get("ID", ""),
                    "severity": mc.get("Severity", "UNKNOWN"),
                    "title": mc.get("Title", ""),
                    "message": mc.get("Message", ""),
                    "resolution": mc.get("Resolution", ""),
                })

        scan = {
            "path": path,
            "scan_type": "misconfiguration",
            "scan_date": datetime.utcnow().isoformat(),
            "total": len(misconfigs),
            "misconfigurations": misconfigs,
        }
        self.scan_results.append(scan)
        return scan

    def generate_sbom(self, image_ref, sbom_format="cyclonedx"):
        """Generate an SBOM for a container image."""
        out_file = self.output_dir / f"sbom.{sbom_format}.json"
        cmd = ["trivy", "image", "--format", sbom_format,
               "--output", str(out_file), image_ref]
        try:
            subprocess.run(cmd, capture_output=True, text=True, timeout=300)
            return {"sbom_path": str(out_file)}
        except (FileNotFoundError, subprocess.TimeoutExpired) as e:
            return {"error": str(e)}

    def enforce_quality_gate(self, severity_threshold="HIGH"):
        """Check if any scan result exceeds the severity threshold."""
        sev_order = ["CRITICAL", "HIGH", "MEDIUM", "LOW"]
        threshold_idx = sev_order.index(severity_threshold) if severity_threshold in sev_order else 0
        blocking_sevs = sev_order[:threshold_idx + 1]

        for scan in self.scan_results:
            summary = scan.get("summary", {})
            for sev in blocking_sevs:
                if summary.get(sev, 0) > 0:
                    return {"gate": "FAILED", "reason": f"{summary[sev]} {sev} findings in {scan.get('image', scan.get('path', ''))}"}
        return {"gate": "PASSED"}

    def generate_report(self):
        gate = self.enforce_quality_gate()
        report = {
            "report_date": datetime.utcnow().isoformat(),
            "scans": self.scan_results,
            "quality_gate": gate,
        }
        out = self.output_dir / "trivy_cicd_report.json"
        with open(out, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))
        return report


def main():
    if len(sys.argv) < 2:
        print("Usage: agent.py <image_ref> [--config <path>] [--severity CRITICAL,HIGH]")
        sys.exit(1)

    agent = TrivyCICDAgent()
    image = sys.argv[1]
    severity = "CRITICAL,HIGH"
    if "--severity" in sys.argv:
        idx = sys.argv.index("--severity")
        if idx + 1 < len(sys.argv):
            severity = sys.argv[idx + 1]

    agent.scan_image(image, severity=severity)

    if "--config" in sys.argv:
        idx = sys.argv.index("--config")
        if idx + 1 < len(sys.argv):
            agent.scan_config(sys.argv[idx + 1], severity=severity)

    report = agent.generate_report()
    if report["quality_gate"]["gate"] == "FAILED":
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/scanning-containers-with-trivy-in-cicd/scripts/process.py =====
#!/usr/bin/env python3
"""
Trivy Container Scanning Pipeline Script

Scans Docker images with Trivy, evaluates quality gates, generates reports,
and optionally uploads results. Supports both local scanning and CI/CD integration.

Usage:
    python process.py --image myapp:latest --severity-threshold high
    python process.py --image myapp:latest --output report.json --fail-on-findings
    python process.py --dockerfile ./Dockerfile --scan-config
"""

import argparse
import json
import os
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime, timezone
from typing import Optional


SEVERITY_ORDER = {"CRITICAL": 0, "HIGH": 1, "MEDIUM": 2, "LOW": 3, "UNKNOWN": 4}


@dataclass
class Vulnerability:
    vuln_id: str
    pkg_name: str
    installed_version: str
    fixed_version: str
    severity: str
    title: str
    description: str
    cvss_score: float = 0.0
    references: list = field(default_factory=list)


@dataclass
class Misconfiguration:
    misconfig_id: str
    title: str
    severity: str
    message: str
    resolution: str
    file_path: str = ""


@dataclass
class ScanResult:
    image: str
    vulnerabilities: list = field(default_factory=list)
    misconfigurations: list = field(default_factory=list)
    scan_duration: float = 0.0
    db_version: str = ""
    trivy_version: str = ""
    error: str = ""


def run_trivy_scan(image: str, severity: str = "CRITICAL,HIGH,MEDIUM,LOW",
                   ignore_unfixed: bool = True, scan_type: str = "image",
                   cache_dir: Optional[str] = None, skip_db_update: bool = False) -> dict:
    """Execute Trivy scan and return JSON results."""
    cmd = ["trivy", scan_type]

    if scan_type == "image":
        cmd.extend([
            "--format", "json",
            "--severity", severity,
        ])
        if ignore_unfixed:
            cmd.append("--ignore-unfixed")
        if cache_dir:
            cmd.extend(["--cache-dir", cache_dir])
        if skip_db_update:
            cmd.append("--skip-db-update")
        cmd.append(image)
    elif scan_type == "config":
        cmd.extend([
            "--format", "json",
            "--severity", severity,
            image
        ])

    try:
        proc = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            timeout=600
        )

        if proc.stdout:
            return json.loads(proc.stdout)
        else:
            return {"error": proc.stderr[:500]}

    except subprocess.TimeoutExpired:
        return {"error": "Trivy scan timed out after 600 seconds"}
    except FileNotFoundError:
        return {"error": "trivy binary not found. Install from https://trivy.dev"}
    except json.JSONDecodeError:
        return {"error": "Failed to parse Trivy JSON output"}


def parse_vulnerabilities(trivy_json: dict) -> list:
    """Extract vulnerability findings from Trivy JSON output."""
    vulns = []
    for result in trivy_json.get("Results", []):
        for vuln in result.get("Vulnerabilities", []):
            vulns.append(Vulnerability(
                vuln_id=vuln.get("VulnerabilityID", ""),
                pkg_name=vuln.get("PkgName", ""),
                installed_version=vuln.get("InstalledVersion", ""),
                fixed_version=vuln.get("FixedVersion", ""),
                severity=vuln.get("Severity", "UNKNOWN"),
                title=vuln.get("Title", ""),
                description=vuln.get("Description", "")[:300],
                cvss_score=vuln.get("CVSS", {}).get("nvd", {}).get("V3Score", 0.0),
                references=vuln.get("References", [])[:3]
            ))
    return vulns


def parse_misconfigurations(trivy_json: dict) -> list:
    """Extract misconfiguration findings from Trivy JSON output."""
    misconfigs = []
    for result in trivy_json.get("Results", []):
        for mc in result.get("Misconfigurations", []):
            if mc.get("Status") == "FAIL":
                misconfigs.append(Misconfiguration(
                    misconfig_id=mc.get("ID", ""),
                    title=mc.get("Title", ""),
                    severity=mc.get("Severity", "UNKNOWN"),
                    message=mc.get("Message", ""),
                    resolution=mc.get("Resolution", ""),
                    file_path=result.get("Target", "")
                ))
    return misconfigs


def generate_sarif(scan_result: ScanResult) -> dict:
    """Generate SARIF format output for GitHub Security tab integration."""
    rules = []
    results = []
    rule_ids_seen = set()

    for vuln in scan_result.vulnerabilities:
        if vuln.vuln_id not in rule_ids_seen:
            rule_ids_seen.add(vuln.vuln_id)
            severity_map = {"CRITICAL": "9.5", "HIGH": "7.5", "MEDIUM": "5.0", "LOW": "2.5"}
            rules.append({
                "id": vuln.vuln_id,
                "shortDescription": {"text": vuln.title or vuln.vuln_id},
                "fullDescription": {"text": vuln.description[:500]},
                "properties": {
                    "security-severity": str(vuln.cvss_score or severity_map.get(vuln.severity, "5.0")),
                    "tags": ["security", "vulnerability", vuln.severity.lower()]
                }
            })

        level_map = {"CRITICAL": "error", "HIGH": "error", "MEDIUM": "warning", "LOW": "note"}
        results.append({
            "ruleId": vuln.vuln_id,
            "level": level_map.get(vuln.severity, "warning"),
            "message": {
                "text": f"{vuln.pkg_name} {vuln.installed_version} -> {vuln.fixed_version}: {vuln.title}"
            },
            "locations": [{
                "physicalLocation": {
                    "artifactLocation": {"uri": "Dockerfile"},
                    "region": {"startLine": 1}
                }
            }]
        })

    return {
        "$schema": "https://raw.githubusercontent.com/oasis-tcs/sarif-spec/main/sarif-2.1/schema/sarif-schema-2.1.0.json",
        "version": "2.1.0",
        "runs": [{
            "tool": {
                "driver": {
                    "name": "Trivy",
                    "informationUri": "https://trivy.dev",
                    "rules": rules
                }
            },
            "results": results
        }]
    }


def evaluate_quality_gate(scan_result: ScanResult, threshold: str) -> dict:
    """Evaluate whether scan results pass the quality gate."""
    threshold_level = SEVERITY_ORDER.get(threshold.upper(), 1)

    blocking_vulns = [
        v for v in scan_result.vulnerabilities
        if SEVERITY_ORDER.get(v.severity, 4) <= threshold_level
    ]

    blocking_misconfigs = [
        m for m in scan_result.misconfigurations
        if SEVERITY_ORDER.get(m.severity, 4) <= threshold_level
    ]

    severity_counts = {}
    for v in scan_result.vulnerabilities:
        severity_counts[v.severity] = severity_counts.get(v.severity, 0) + 1

    return {
        "passed": len(blocking_vulns) == 0 and len(blocking_misconfigs) == 0,
        "threshold": threshold.upper(),
        "total_vulnerabilities": len(scan_result.vulnerabilities),
        "total_misconfigurations": len(scan_result.misconfigurations),
        "blocking_vulnerabilities": len(blocking_vulns),
        "blocking_misconfigurations": len(blocking_misconfigs),
        "severity_counts": severity_counts,
        "blocking_details": [
            {"id": v.vuln_id, "pkg": v.pkg_name, "severity": v.severity,
             "installed": v.installed_version, "fixed": v.fixed_version}
            for v in blocking_vulns[:20]
        ]
    }


def main():
    parser = argparse.ArgumentParser(description="Trivy Container Scanning Pipeline")
    parser.add_argument("--image", required=True, help="Docker image to scan (e.g., myapp:latest)")
    parser.add_argument("--output", default="trivy-report.json", help="Output report file path")
    parser.add_argument("--sarif-output", default=None, help="SARIF output file path")
    parser.add_argument("--severity-threshold", default="high",
                        choices=["critical", "high", "medium", "low"],
                        help="Minimum severity to block pipeline")
    parser.add_argument("--fail-on-findings", action="store_true",
                        help="Exit non-zero if quality gate fails")
    parser.add_argument("--ignore-unfixed", action="store_true", default=True,
                        help="Ignore vulnerabilities without fixes")
    parser.add_argument("--scan-config", action="store_true",
                        help="Also scan for misconfigurations")
    parser.add_argument("--dockerfile", default=None,
                        help="Path to Dockerfile for misconfiguration scanning")
    parser.add_argument("--cache-dir", default=None, help="Trivy cache directory")
    parser.add_argument("--skip-db-update", action="store_true", help="Skip DB update")
    args = parser.parse_args()

    scan_result = ScanResult(image=args.image)
    start_time = datetime.now(timezone.utc)

    print(f"[*] Scanning image: {args.image}")
    trivy_json = run_trivy_scan(
        args.image,
        ignore_unfixed=args.ignore_unfixed,
        cache_dir=args.cache_dir,
        skip_db_update=args.skip_db_update
    )

    if "error" in trivy_json:
        print(f"[ERROR] {trivy_json['error']}")
        sys.exit(2)

    scan_result.vulnerabilities = parse_vulnerabilities(trivy_json)
    scan_result.db_version = trivy_json.get("Metadata", {}).get("DB", {}).get("UpdatedAt", "unknown")

    if args.scan_config and args.dockerfile:
        print(f"[*] Scanning configuration: {args.dockerfile}")
        config_path = os.path.dirname(args.dockerfile) or "."
        config_json = run_trivy_scan(config_path, scan_type="config")
        if "error" not in config_json:
            scan_result.misconfigurations = parse_misconfigurations(config_json)

    scan_result.scan_duration = (datetime.now(timezone.utc) - start_time).total_seconds()

    quality_gate = evaluate_quality_gate(scan_result, args.severity_threshold)

    report = {
        "scan_metadata": {
            "image": args.image,
            "scan_date": datetime.now(timezone.utc).isoformat(),
            "duration_seconds": scan_result.scan_duration,
            "db_version": scan_result.db_version
        },
        "quality_gate": quality_gate,
        "vulnerabilities": [
            {
                "id": v.vuln_id, "package": v.pkg_name, "severity": v.severity,
                "installed": v.installed_version, "fixed": v.fixed_version,
                "title": v.title, "cvss": v.cvss_score
            }
            for v in sorted(scan_result.vulnerabilities,
                            key=lambda x: SEVERITY_ORDER.get(x.severity, 4))
        ],
        "misconfigurations": [
            {"id": m.misconfig_id, "title": m.title, "severity": m.severity,
             "message": m.message, "resolution": m.resolution}
            for m in scan_result.misconfigurations
        ]
    }

    with open(os.path.abspath(args.output), "w") as f:
        json.dump(report, f, indent=2)
    print(f"[*] Report: {os.path.abspath(args.output)}")

    if args.sarif_output:
        sarif = generate_sarif(scan_result)
        with open(os.path.abspath(args.sarif_output), "w") as f:
            json.dump(sarif, f, indent=2)
        print(f"[*] SARIF: {os.path.abspath(args.sarif_output)}")

    print(f"\n[*] Vulnerabilities: {len(scan_result.vulnerabilities)} "
          f"| Misconfigs: {len(scan_result.misconfigurations)}")

    if quality_gate["passed"]:
        print(f"[PASS] Quality gate passed.")
    else:
        print(f"[FAIL] Quality gate failed. {quality_gate['blocking_vulnerabilities']} blocking vulns.")
        for d in quality_gate["blocking_details"][:10]:
            print(f"  - [{d['severity']}] {d['id']} in {d['pkg']} ({d['installed']} -> {d['fixed']})")

    if args.fail_on_findings and not quality_gate["passed"]:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/scanning-iac-and-images-with-trivy/SKILL.md =====
---
name: scanning-iac-and-images-with-trivy
description: Scans container images, Infrastructure-as-Code (Terraform, CloudFormation,
  Kubernetes manifests, Dockerfile, Helm), filesystems, git repos, and SBOMs with Trivy's
  vuln, misconfig, secret, and license scanners. Use when building a CI/CD security
  gate that scans images before push, IaC before apply, or SBOMs for supply-chain
  weaknesses, and fails the build on policy violations.
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- trivy
- container-security
- vulnerability-scanning
- iac-security
- sbom
- ci-cd
- misconfiguration
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
mitre_attack:
- T1525
---
# Scanning IaC and Images with Trivy

## Overview

Trivy (by Aqua Security) is a comprehensive, open-source security scanner that finds vulnerabilities (CVEs), misconfigurations (IaC), secrets, software licenses, and software supply-chain weaknesses across a wide range of targets: container images, filesystems, Git repositories, virtual machine images, Kubernetes clusters, and SBOM documents. It is widely adopted as a "shift-left" gate in CI/CD pipelines because it is fast, runs as a single static binary, requires no agent, and supports machine-readable output formats (JSON, SARIF, CycloneDX, SPDX) for integration with code-scanning dashboards.

Trivy bundles four primary scanners that can be toggled with `--scanners`:

- **vuln** — OS package and language-dependency vulnerability detection (CVE matching against the Trivy vulnerability DB).
- **misconfig** — Infrastructure-as-Code and configuration misconfiguration detection (Terraform, CloudFormation, Kubernetes manifests, Dockerfile, Helm) using built-in and custom Rego policies.
- **secret** — Hard-coded secret/credential detection (API keys, tokens, private keys).
- **license** — Software license identification and policy enforcement.

This skill covers building a Trivy-based scanning workflow that gates a CI/CD pipeline: scanning images before push, scanning IaC before apply, generating and re-scanning SBOMs, and failing builds on policy violations. Detecting these weaknesses defends against the MITRE ATT&CK technique **T1525 (Implant Internal Image)**, where adversaries plant malicious or vulnerable images in a registry to be deployed across the environment.

## When to Use

- When integrating vulnerability and misconfiguration scanning into a CI/CD pipeline as a quality/security gate before images are pushed or infrastructure is applied.
- When auditing container images in a registry for known CVEs prior to deployment.
- When validating Terraform, CloudFormation, Kubernetes, Dockerfile, or Helm IaC for security misconfigurations.
- When generating an SBOM (CycloneDX/SPDX) for supply-chain transparency and later re-scanning that SBOM for newly disclosed CVEs.
- When scanning a running Kubernetes cluster for vulnerable workloads and misconfigured RBAC/resources.
- When enforcing license compliance policy on dependencies.

## Prerequisites

- A Linux/macOS/Windows host or CI runner with network access to download the Trivy vulnerability database.
- Docker (optional) if scanning local images by name or using the containerized Trivy.
- Install Trivy (Aqua Security official methods):

```bash
# Debian/Ubuntu (APT repository)
sudo apt-get install -y wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy

# RHEL/CentOS (YUM repository), macOS (Homebrew), and install script
brew install trivy
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin

# Containerized usage (no install)
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image python:3.10-alpine

# Verify
trivy --version
```

- For air-gapped or rate-limited environments, pre-download the DB with `trivy image --download-db-only` and the Java/policy bundles as needed.

## Objectives

- Scan a container image for OS and language vulnerabilities and fail the build above a severity threshold.
- Scan IaC (Terraform/Kubernetes/Dockerfile) for misconfigurations.
- Detect hard-coded secrets in a repository or filesystem.
- Generate a CycloneDX or SPDX SBOM and re-scan it for vulnerabilities.
- Emit SARIF for GitHub code scanning and JSON for programmatic gating.
- Wire Trivy into a CI/CD pipeline with `--exit-code` to enforce policy.

## MITRE ATT&CK Mapping

| Technique ID | Name | Tactic | Relevance |
|--------------|------|--------|-----------|
| T1525 | Implant Internal Image | Persistence | Trivy detects vulnerable or malicious images/layers and embedded secrets before they are implanted in a registry and propagated to running workloads. |

## Workflow

### 1. Scan a container image for vulnerabilities

Run a vulnerability-only scan of a registry image, restricting to high/critical findings and ignoring CVEs with no available fix:

```bash
trivy image \
  --scanners vuln \
  --severity HIGH,CRITICAL \
  --ignore-unfixed \
  --format table \
  python:3.10-alpine
```

Scan an image saved as a tarball (useful when the image is built but not yet pushed):

```bash
docker save myorg/app:1.4.0 -o app.tar
trivy image --input app.tar --severity CRITICAL --format json --output app-vulns.json
```

### 2. Enable multiple scanners on an image

Run vulnerability, misconfiguration, secret, and license scanners together:

```bash
trivy image \
  --scanners vuln,misconfig,secret,license \
  --severity MEDIUM,HIGH,CRITICAL \
  myorg/app:1.4.0
```

Scan the image's embedded config (Dockerfile-equivalent build history and history secrets):

```bash
trivy image --image-config-scanners misconfig,secret myorg/app:1.4.0
```

### 3. Scan Infrastructure-as-Code (misconfiguration)

Scan a directory of Terraform / Kubernetes / Dockerfile / Helm / CloudFormation for misconfigurations using the `config` target:

```bash
# Scan a Terraform / IaC directory
trivy config \
  --severity HIGH,CRITICAL \
  --format table \
  ./infra

# Scan with custom Rego policies and a specific policy namespace
trivy config \
  --config-policy ./policies \
  --policy-namespaces user \
  ./infra
```

Alternatively use the `fs` (filesystem) target with the misconfig scanner explicitly:

```bash
trivy fs --scanners misconfig,secret --severity HIGH,CRITICAL ./infra
```

### 4. Scan a repository and detect secrets

Scan a local working tree (or remote repo) for vulnerabilities in lockfiles and hard-coded secrets:

```bash
# Local filesystem (dependencies + secrets)
trivy fs --scanners vuln,secret --severity HIGH,CRITICAL .

# Remote Git repository
trivy repository --scanners vuln,secret https://github.com/myorg/myrepo
```

### 5. Generate and re-scan an SBOM

Produce a CycloneDX SBOM from an image, then scan the SBOM itself for vulnerabilities (so a stored SBOM can be re-evaluated as new CVEs are disclosed):

```bash
# Generate CycloneDX SBOM
trivy image --format cyclonedx --output sbom.cdx.json myorg/app:1.4.0

# Generate SPDX SBOM
trivy image --format spdx-json --output sbom.spdx.json myorg/app:1.4.0

# Re-scan the SBOM for vulnerabilities later
trivy sbom --severity HIGH,CRITICAL sbom.cdx.json
```

### 6. Emit SARIF for code-scanning dashboards

Produce SARIF for GitHub Advanced Security / code scanning ingestion:

```bash
trivy image \
  --format sarif \
  --output trivy-results.sarif \
  --severity HIGH,CRITICAL \
  myorg/app:1.4.0
```

### 7. Gate the CI/CD pipeline with exit codes

Use `--exit-code 1` so the pipeline step fails when findings at or above the chosen severity are present. Separate the "report everything" run (exit 0) from the "enforce" run (exit 1):

```bash
# 1) Informational report (never fails the build)
trivy image --severity LOW,MEDIUM,HIGH,CRITICAL --exit-code 0 --format table myorg/app:1.4.0

# 2) Enforcement gate (fails build on HIGH/CRITICAL with a fix available)
trivy image \
  --severity HIGH,CRITICAL \
  --ignore-unfixed \
  --exit-code 1 \
  --format json --output gate.json \
  myorg/app:1.4.0
```

Example GitHub Actions step using the official action:

```yaml
- name: Run Trivy image scan (gate)
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myorg/app:1.4.0'
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'HIGH,CRITICAL'
    ignore-unfixed: true
    exit-code: '1'
```

### 8. Manage false positives and the DB

Suppress accepted-risk findings with a `.trivyignore` file and keep the DB current:

```bash
# .trivyignore — one CVE/AVD/secret rule ID per line
echo "CVE-2023-12345" >> .trivyignore
echo "AVD-AWS-0089"   >> .trivyignore

# Refresh DBs explicitly (useful for caching layers in CI)
trivy image --download-db-only
trivy image --download-java-db-only

# Scan a Kubernetes cluster (summary report)
trivy k8s --report summary --severity HIGH,CRITICAL cluster
```

## Tools and Resources

| Tool / Resource | Purpose | Link |
|------------------|---------|------|
| Trivy | Core scanner CLI | https://github.com/aquasecurity/trivy |
| Trivy Documentation | Official docs (targets, scanners, flags) | https://trivy.dev/latest/docs/ |
| trivy-action | GitHub Actions integration | https://github.com/aquasecurity/trivy-action |
| Trivy Operator | In-cluster Kubernetes continuous scanning | https://github.com/aquasecurity/trivy-operator |
| Trivy vulnerability DB | OSS vulnerability data source | https://github.com/aquasecurity/trivy-db |
| CycloneDX | SBOM standard emitted by Trivy | https://cyclonedx.org/ |

## Validation Criteria

- [ ] Trivy installed and `trivy --version` reports a valid version.
- [ ] Container image scanned for vulnerabilities with severity filtering applied.
- [ ] Image scanned with multiple scanners (vuln, misconfig, secret, license).
- [ ] IaC directory scanned with `trivy config` and misconfigurations reviewed.
- [ ] Secret scanning run against the repository.
- [ ] CycloneDX/SPDX SBOM generated and successfully re-scanned with `trivy sbom`.
- [ ] SARIF output produced for the code-scanning dashboard.
- [ ] CI/CD gate fails the build on HIGH/CRITICAL findings via `--exit-code 1`.
- [ ] `.trivyignore` configured for accepted-risk findings with documented justification.

===== END FILE =====


===== FILE: skills/scanning-iac-and-images-with-trivy/LICENSE =====

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


===== FILE: skills/scanning-iac-and-images-with-trivy/references/api-reference.md =====
# Trivy — Command and Flag Reference

## Scan Targets (subcommands)

| Command | Target | Example |
|---------|--------|---------|
| `trivy image` | Container image (registry/tar) | `trivy image alpine:3.19` |
| `trivy fs` | Local filesystem / project dir | `trivy fs ./` |
| `trivy repository` (`repo`) | Git repository (local or remote URL) | `trivy repo https://github.com/org/repo` |
| `trivy config` | IaC / config misconfiguration | `trivy config ./infra` |
| `trivy sbom` | Existing SBOM (CycloneDX/SPDX) | `trivy sbom sbom.cdx.json` |
| `trivy kubernetes` (`k8s`) | Live Kubernetes cluster | `trivy k8s --report summary cluster` |
| `trivy vm` | VM image (AMI/EBS/VMDK) | `trivy vm ami:ami-0123` |
| `trivy rootfs` | Extracted root filesystem | `trivy rootfs /mnt/rootfs` |

## Key Flags

| Flag | Description |
|------|-------------|
| `--scanners vuln,misconfig,secret,license` | Select which scanners to run |
| `--severity LOW,MEDIUM,HIGH,CRITICAL` | Filter results by severity |
| `--exit-code <n>` | Exit code when matching results are found (gating) |
| `--ignore-unfixed` | Suppress vulnerabilities with no fixed version |
| `--format table\|json\|sarif\|cyclonedx\|spdx-json` | Output format |
| `--output <file>` | Write report to file |
| `--input <file>` | Scan an image tar instead of a registry ref |
| `--vuln-type os,library` | Limit vulnerability detection scope |
| `--image-config-scanners misconfig,secret` | Scan image build config/history |
| `--config-policy <dir>` | Custom Rego misconfig policy directory |
| `--policy-namespaces <ns>` | Rego policy namespaces to evaluate |
| `--download-db-only` | Pre-download vulnerability DB (caching/air-gap) |
| `--download-java-db-only` | Pre-download Java index DB |
| `--skip-dirs` / `--skip-files` | Exclude paths from scan |
| `--ignorefile <path>` | Path to `.trivyignore` (default `.trivyignore`) |

## Output Formats

| Format | Use |
|--------|-----|
| `table` | Human-readable console (default) |
| `json` | Programmatic gating / parsing |
| `sarif` | GitHub code scanning / IDE ingestion |
| `cyclonedx` | CycloneDX SBOM |
| `spdx-json` | SPDX SBOM (JSON) |
| `github` | GitHub dependency snapshot |

## `.trivyignore` Format

```
# One ID per line; supports CVE, AVD (misconfig), and secret rule IDs
CVE-2023-12345
AVD-AWS-0089
generic-api-key
```

## GitHub Actions (trivy-action)

```yaml
- uses: aquasecurity/trivy-action@master
  with:
    scan-type: 'image'      # image | fs | config | repo | sbom
    image-ref: 'myorg/app:1.4.0'
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'HIGH,CRITICAL'
    ignore-unfixed: true
    exit-code: '1'
```

## External References

- Trivy Docs: https://trivy.dev/latest/docs/
- Configuration reference: https://trivy.dev/latest/docs/configuration/
- Misconfiguration scanning: https://trivy.dev/latest/docs/scanner/misconfiguration/
- SBOM: https://trivy.dev/latest/docs/supply-chain/sbom/

===== END FILE =====


===== FILE: skills/scanning-iac-and-images-with-trivy/references/standards.md =====
# Standards and References — Scanning IaC and Images with Trivy

## NIST CSF 2.0

| ID | Name | Rationale |
|----|------|-----------|
| ID.RA-01 | Asset vulnerabilities are identified and recorded | Trivy enumerates CVEs, misconfigurations, and embedded secrets across images, IaC, and SBOMs, recording them for risk assessment and remediation tracking. |

## MITRE ATT&CK

| Technique ID | Name | Tactic | Rationale |
|--------------|------|--------|-----------|
| T1525 | Implant Internal Image | Persistence | Scanning images and SBOMs before they reach a registry detects vulnerable, secret-laden, or malicious layers an adversary could implant to persist across deployments. |

## Supporting Frameworks and Standards

- **CIS Benchmarks** — Trivy `misconfig` checks align with CIS Docker/Kubernetes hardening guidance.
- **CycloneDX / SPDX** — SBOM output formats supported for supply-chain transparency.
- **SARIF** — Static Analysis Results Interchange Format for code-scanning dashboards.
- **OWASP Top 10 (A06: Vulnerable and Outdated Components)** — Trivy directly addresses outdated/vulnerable dependency detection.

## Official Resources

- Trivy: https://github.com/aquasecurity/trivy
- Trivy Docs: https://trivy.dev/latest/docs/
- Trivy DB: https://github.com/aquasecurity/trivy-db
- trivy-action: https://github.com/aquasecurity/trivy-action
- Aqua Security: https://www.aquasec.com/products/trivy/

===== END FILE =====


===== FILE: skills/scanning-iac-and-images-with-trivy/scripts/agent.py =====
#!/usr/bin/env python3
"""Trivy scanning helper.

Wraps the Trivy CLI to scan container images, filesystems/IaC, and SBOMs,
parse the JSON results, summarise findings by severity, and exit non-zero
when findings at or above a chosen threshold are present (CI/CD gating).

Requires the `trivy` binary on PATH. See https://github.com/aquasecurity/trivy
"""

import argparse
import json
import shutil
import subprocess
import sys
from collections import Counter
from datetime import datetime, timezone

SEVERITY_ORDER = ["LOW", "MEDIUM", "HIGH", "CRITICAL"]


def ensure_trivy() -> str:
    """Return the trivy executable path or exit with guidance."""
    path = shutil.which("trivy")
    if not path:
        print("[!] trivy not found on PATH. Install: "
              "https://trivy.dev/latest/getting-started/installation/",
              file=sys.stderr)
        sys.exit(2)
    return path


def build_command(trivy: str, args: argparse.Namespace) -> list:
    """Construct the trivy command line for the requested target."""
    cmd = [trivy, args.target]
    cmd += ["--scanners", args.scanners]
    if args.severity:
        cmd += ["--severity", args.severity]
    if args.ignore_unfixed:
        cmd.append("--ignore-unfixed")
    # Always request JSON so this helper can parse + gate deterministically.
    cmd += ["--format", "json"]
    # exit-code 0 here; gating is enforced in Python after parsing.
    cmd += ["--exit-code", "0"]
    cmd.append(args.subject)
    return cmd


def run_scan(cmd: list, timeout: int) -> dict:
    """Execute trivy and return the parsed JSON report."""
    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
    except subprocess.TimeoutExpired:
        print(f"[!] Trivy scan timed out after {timeout}s", file=sys.stderr)
        sys.exit(3)
    if proc.returncode not in (0,):
        # Trivy writes scan results to stdout; real errors go to stderr.
        if not proc.stdout.strip():
            print(f"[!] Trivy failed (rc={proc.returncode}): {proc.stderr.strip()}",
                  file=sys.stderr)
            sys.exit(proc.returncode)
    try:
        return json.loads(proc.stdout)
    except json.JSONDecodeError:
        print("[!] Could not parse Trivy JSON output.", file=sys.stderr)
        print(proc.stderr.strip(), file=sys.stderr)
        sys.exit(4)


def summarise(report: dict) -> tuple:
    """Return (severity Counter, list of finding dicts) from a Trivy report."""
    counts = Counter()
    findings = []
    for result in report.get("Results", []) or []:
        target = result.get("Target", "")
        for vuln in result.get("Vulnerabilities", []) or []:
            sev = (vuln.get("Severity") or "UNKNOWN").upper()
            counts[sev] += 1
            findings.append({
                "type": "vuln",
                "target": target,
                "id": vuln.get("VulnerabilityID"),
                "pkg": vuln.get("PkgName"),
                "installed": vuln.get("InstalledVersion"),
                "fixed": vuln.get("FixedVersion"),
                "severity": sev,
            })
        for mis in result.get("Misconfigurations", []) or []:
            sev = (mis.get("Severity") or "UNKNOWN").upper()
            counts[sev] += 1
            findings.append({
                "type": "misconfig",
                "target": target,
                "id": mis.get("ID"),
                "title": mis.get("Title"),
                "severity": sev,
            })
        for sec in result.get("Secrets", []) or []:
            sev = (sec.get("Severity") or "UNKNOWN").upper()
            counts[sev] += 1
            findings.append({
                "type": "secret",
                "target": target,
                "id": sec.get("RuleID"),
                "title": sec.get("Title"),
                "severity": sev,
            })
    return counts, findings


def gate(counts: Counter, threshold: str) -> bool:
    """Return True if any finding is at or above threshold severity."""
    if threshold not in SEVERITY_ORDER:
        return False
    idx = SEVERITY_ORDER.index(threshold)
    blocking = SEVERITY_ORDER[idx:]
    return any(counts.get(s, 0) > 0 for s in blocking)


def main() -> None:
    parser = argparse.ArgumentParser(description="Trivy scan + CI/CD gate helper")
    parser.add_argument("subject", help="Image ref, directory, repo URL, or SBOM file")
    parser.add_argument("--target", default="image",
                        choices=["image", "fs", "config", "repo", "sbom"],
                        help="Trivy scan target subcommand")
    parser.add_argument("--scanners", default="vuln",
                        help="Comma list: vuln,misconfig,secret,license")
    parser.add_argument("--severity", default="HIGH,CRITICAL",
                        help="Severities to include")
    parser.add_argument("--ignore-unfixed", action="store_true",
                        help="Skip vulns with no fix available")
    parser.add_argument("--gate", default="", choices=["", *SEVERITY_ORDER],
                        help="Exit non-zero if findings >= this severity")
    parser.add_argument("--timeout", type=int, default=600, help="Scan timeout (s)")
    parser.add_argument("--output", help="Write JSON summary to file")
    args = parser.parse_args()

    trivy = ensure_trivy()
    cmd = build_command(trivy, args)
    print(f"[*] {datetime.now(timezone.utc).isoformat()} running: {' '.join(cmd)}")
    report = run_scan(cmd, args.timeout)
    counts, findings = summarise(report)

    print("\n=== TRIVY FINDINGS BY SEVERITY ===")
    for sev in reversed(SEVERITY_ORDER):
        print(f"  {sev:<9}: {counts.get(sev, 0)}")
    print(f"  TOTAL    : {sum(counts.values())}")

    if args.output:
        with open(args.output, "w", encoding="utf-8") as fh:
            json.dump({"counts": dict(counts), "findings": findings}, fh, indent=2)
        print(f"[+] Summary written to {args.output}")

    if args.gate and gate(counts, args.gate):
        print(f"[!] GATE FAILED: findings at or above {args.gate}", file=sys.stderr)
        sys.exit(1)
    print("[+] Scan complete.")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/securing-github-actions-workflows/SKILL.md =====
---
name: securing-github-actions-workflows
description: 'Hardens GitHub Actions workflows against supply chain attacks, credential
  theft, and privilege escalation: pinning actions to SHA digests, minimizing GITHUB_TOKEN
  permissions, protecting secrets, preventing script injection in workflow expressions,
  and requiring reviewers for workflow changes. Use when hardening GitHub Actions
  workflows that handle secrets, deploy to production, or run with elevated permissions.

  '
domain: cybersecurity
subdomain: devsecops
tags:
- devsecops
- cicd
- github-actions
- supply-chain
- workflow-security
- secure-sdlc
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.PS-01
- GV.SC-07
- ID.IM-04
- PR.PS-04
mitre_attack:
- T1195
- T1554
- T1059.004
- T1068
- T1548
---

# Securing GitHub Actions Workflows

## When to Use

- When GitHub Actions is the CI/CD platform and workflows need hardening against supply chain attacks
- When workflows handle secrets, deploy to production, or have elevated permissions
- When preventing script injection via untrusted PR titles, branch names, or commit messages
- When requiring audit trails and approval gates for workflow modifications
- When third-party actions pose supply chain risk through mutable version tags

**Do not use** for securing other CI/CD platforms (see platform-specific hardening guides), for application vulnerability scanning (use SAST/DAST), or for secret detection in code (use Gitleaks).

## Prerequisites

- GitHub repository with GitHub Actions enabled
- GitHub organization admin access for organization-level settings
- Understanding of GitHub Actions workflow syntax and events

## Workflow

### Step 1: Pin Actions to SHA Digests

```yaml
# INSECURE: Mutable tag can be overwritten by attacker
- uses: actions/checkout@v4

# SECURE: Pinned to immutable SHA digest
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11  # v4.1.1

# Use Dependabot to auto-update pinned SHAs
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    commit-message:
      prefix: "ci"
```

### Step 2: Minimize GITHUB_TOKEN Permissions

```yaml
# Set restrictive default permissions at workflow level
name: CI Pipeline
permissions: {}  # Start with no permissions

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read  # Only what's needed
    steps:
      - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11

  deploy:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      deployments: write
      id-token: write  # For OIDC-based cloud auth
    steps:
      - name: Deploy
        run: echo "deploying"
```

### Step 3: Prevent Script Injection

```yaml
# VULNERABLE: User-controlled input in run step
- run: echo "PR title is ${{ github.event.pull_request.title }}"

# SECURE: Use environment variable (properly escaped by shell)
- name: Process PR
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
    PR_BODY: ${{ github.event.pull_request.body }}
  run: |
    echo "PR title is ${PR_TITLE}"
    echo "PR body is ${PR_BODY}"

# SECURE: Use actions/github-script for complex operations
- uses: actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea
  with:
    script: |
      const title = context.payload.pull_request.title;
      console.log(`PR title: ${title}`);
```

### Step 4: Secure Fork Pull Request Handling

```yaml
# DANGEROUS: pull_request_target runs with base repo permissions
# on: pull_request_target  # AVOID unless absolutely necessary

# SAFE: pull_request runs in fork context with limited permissions
on:
  pull_request:
    branches: [main]

# If pull_request_target is required, never checkout PR code:
on:
  pull_request_target:
    types: [labeled]

jobs:
  safe-job:
    if: contains(github.event.pull_request.labels.*.name, 'safe-to-test')
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      # NEVER do: actions/checkout with ref: ${{ github.event.pull_request.head.sha }}
      - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11
        # This checks out the BASE branch, not the PR
```

### Step 5: Protect Secrets and Environment Variables

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # Requires approval
    steps:
      - name: Deploy with secret
        env:
          # Secrets are masked in logs automatically
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          # Never echo secrets
          # echo "$DEPLOY_KEY"  # BAD
          deploy-tool --key-file <(echo "$DEPLOY_KEY")

      - name: Audit secret access
        run: |
          # Log that secret was used without exposing it
          echo "::notice::Deploy key accessed for production deployment"
```

### Step 6: Implement Workflow Change Controls

```yaml
# Require CODEOWNERS approval for workflow changes
# .github/CODEOWNERS
.github/workflows/ @security-team @platform-team
.github/actions/ @security-team @platform-team

# Organization settings:
# 1. Settings > Actions > General > Fork PR policies
#    - Require approval for first-time contributors
#    - Require approval for all outside collaborators
# 2. Settings > Actions > General > Workflow permissions
#    - Read repository contents and packages permissions
#    - Do NOT allow GitHub Actions to create and approve PRs
```

## Key Concepts

| Term | Definition |
|------|------------|
| SHA Pinning | Referencing GitHub Actions by their immutable commit SHA instead of mutable version tags |
| Script Injection | Attack where untrusted input (PR title, branch name) is interpolated into shell commands |
| GITHUB_TOKEN | Automatically generated token with configurable permissions scoped to the current repository |
| pull_request_target | Dangerous event trigger that runs in the base repo context with full permissions on fork PRs |
| Environment Protection | GitHub feature requiring manual approval before jobs accessing an environment can run |
| CODEOWNERS | File defining required reviewers for specific paths including workflow files |
| OIDC Federation | Using GitHub's OIDC token to authenticate to cloud providers without storing long-lived credentials |

## Tools & Systems

- **Dependabot**: Automated dependency updater that keeps pinned action SHAs current
- **StepSecurity Harden Runner**: GitHub Action that monitors and restricts outbound network calls from workflows
- **actionlint**: Linter for GitHub Actions workflow files that detects security issues
- **allstar**: GitHub App by OpenSSF that enforces security policies on repositories
- **scorecard**: OpenSSF tool that evaluates supply chain security practices including CI/CD

## Common Scenarios

### Scenario: Preventing Supply Chain Attack via Compromised Third-Party Action

**Context**: A widely-used GitHub Action is compromised and its v3 tag is updated to include credential-stealing code. Repositories using `@v3` automatically pull the malicious version.

**Approach**:
1. Pin all actions to SHA digests immediately across all repositories
2. Configure Dependabot for github-actions ecosystem to manage SHA updates
3. Restrict GITHUB_TOKEN permissions so even compromised actions have minimal access
4. Add StepSecurity harden-runner to detect anomalous outbound network calls
5. Review all third-party actions and replace unnecessary ones with inline scripts
6. Require CODEOWNERS approval for any changes to .github/workflows/

**Pitfalls**: SHA pinning without Dependabot means missing legitimate security updates to actions. Overly restrictive permissions can break legitimate workflows. Using `pull_request_target` for label-based gating still exposes secrets if the workflow checks out PR code.

## Output Format

```
GitHub Actions Security Audit
================================
Repository: org/web-application
Date: 2026-02-23

WORKFLOW ANALYSIS:
  Total workflows: 8
  Total action references: 34

SHA PINNING:
  [FAIL] 12/34 actions use mutable tags instead of SHA digests
  - .github/workflows/ci.yml: actions/setup-node@v4
  - .github/workflows/deploy.yml: aws-actions/configure-aws-credentials@v4

PERMISSIONS:
  [FAIL] 3/8 workflows have no explicit permissions (inherit default)
  [WARN] 1/8 workflows request write-all permissions

SCRIPT INJECTION:
  [FAIL] 2 workflow steps interpolate user input directly
  - .github/workflows/pr-check.yml:23: ${{ github.event.pull_request.title }}

SECRETS:
  [PASS] No secrets exposed in workflow logs
  [PASS] All production deployments use environment protection

SCORE: 6/10 (Remediate 5 HIGH findings)
```

===== END FILE =====


===== FILE: skills/securing-github-actions-workflows/LICENSE =====

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


===== FILE: skills/securing-github-actions-workflows/assets/template.md =====
# GitHub Actions Security Templates

## Hardened Workflow Template

```yaml
name: Secure CI Pipeline
permissions: {}

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11  # v4.1.1
      - uses: step-security/harden-runner@17d0e2bd7d51742c71671bd19fa12bdc9d40a3d6  # v2.8.1
        with:
          egress-policy: audit
      - name: Build
        run: make build
      - name: Test
        run: make test
```

## Dependabot for Actions

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    commit-message:
      prefix: "ci"
```

## CODEOWNERS for Workflow Protection

```
# .github/CODEOWNERS
.github/workflows/ @org/security-team @org/platform-team
.github/actions/ @org/security-team
.github/dependabot.yml @org/platform-team
```

===== END FILE =====


===== FILE: skills/securing-github-actions-workflows/references/api-reference.md =====
# API Reference: Securing GitHub Actions Workflows

## Security Checks

| Check | Risk | Severity |
|-------|------|----------|
| Unpinned actions (mutable tags) | Supply chain attack via tag overwrite | Medium |
| Missing permissions block | Inherits overly broad defaults | Medium |
| write-all permissions | Excessive token scope | High |
| Script injection in run steps | Code execution via PR title/body | High |
| pull_request_target trigger | Fork code runs with base permissions | High |
| Secrets in workflow logs | Credential exposure | Critical |

## Dangerous Expression Contexts

| Context | Risk |
|---------|------|
| `github.event.pull_request.title` | Attacker-controlled PR title |
| `github.event.pull_request.body` | Attacker-controlled PR body |
| `github.event.issue.title` | Attacker-controlled issue title |
| `github.event.comment.body` | Attacker-controlled comment |
| `github.head_ref` | Attacker-controlled branch name |

## SHA Pinning Format

| Format | Security |
|--------|----------|
| `actions/checkout@v4` | Insecure - mutable tag |
| `actions/checkout@b4ffde65f...` | Secure - immutable SHA |

## Permission Scopes

| Scope | Values |
|-------|--------|
| contents | read, write |
| actions | read, write |
| deployments | read, write |
| id-token | write (for OIDC) |
| security-events | write |
| pull-requests | read, write |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `yaml` | PyYAML >=6.0 | Parse workflow YAML |
| `re` | stdlib | Pattern matching |
| `json` | stdlib | Report output |
| `pathlib` | stdlib | File discovery |

## References

- GitHub Actions Security Hardening: https://docs.github.com/en/actions/security-guides
- StepSecurity Harden Runner: https://github.com/step-security/harden-runner
- actionlint: https://github.com/rhysd/actionlint

===== END FILE =====


===== FILE: skills/securing-github-actions-workflows/references/standards.md =====
# Standards Reference: Securing GitHub Actions

## NIST SSDF (SP 800-218)

### PS.1: Protect All Forms of Code
- Workflows are code and must be reviewed and protected
- Pin action dependencies to SHA digests
- Minimize GITHUB_TOKEN permissions

## CIS Software Supply Chain Security

- BD-1: Define security requirements for build processes
- BD-2: Automate security validation of build configurations
- BD-3: Pin all external dependencies to immutable references

## OWASP CI/CD Top 10 Risks

| Risk | GitHub Actions Mitigation |
|------|--------------------------|
| CICD-SEC-1: Insufficient Flow Control | Environment protection rules, CODEOWNERS |
| CICD-SEC-3: Dependency Chain Abuse | SHA pinning of actions |
| CICD-SEC-4: Poisoned Pipeline Execution | Restrict pull_request_target, input sanitization |
| CICD-SEC-6: Credential Hygiene | OIDC federation, minimal GITHUB_TOKEN scope |
| CICD-SEC-9: Artifact Integrity | Sign artifacts in workflows |

## SLSA Framework

- Level 2: Hosted build service (GitHub Actions qualifies)
- Level 3: Hardened build platform with isolation guarantees
- Workflow hardening prevents provenance falsification

===== END FILE =====


===== FILE: skills/securing-github-actions-workflows/references/workflows.md =====
# Workflow Reference: Securing GitHub Actions

## Hardening Checklist

1. Pin all actions to SHA digests
2. Set restrictive default permissions
3. Sanitize all user-controlled inputs
4. Never use pull_request_target with PR checkout
5. Enable environment protection for production
6. Configure CODEOWNERS for workflow files
7. Enable Dependabot for github-actions
8. Audit third-party actions quarterly
9. Use OIDC instead of long-lived cloud credentials
10. Add harden-runner for network monitoring

## Permission Scoping Reference

| Permission | Use Case |
|-----------|----------|
| contents: read | Checkout code |
| contents: write | Create releases, push tags |
| security-events: write | Upload SARIF results |
| packages: write | Push container images |
| deployments: write | Create deployment status |
| id-token: write | OIDC cloud authentication |
| pull-requests: write | Comment on PRs |

## Script Injection Prevention

```yaml
# DANGEROUS patterns to avoid:
run: echo "${{ github.event.issue.title }}"
run: echo "${{ github.event.comment.body }}"
run: echo "${{ github.head_ref }}"

# SAFE alternatives:
env:
  TITLE: ${{ github.event.issue.title }}
run: echo "${TITLE}"
```

===== END FILE =====


===== FILE: skills/securing-github-actions-workflows/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for securing GitHub Actions workflows.

Audits GitHub Actions workflow files for security issues including
unpinned actions, excessive permissions, script injection risks,
dangerous triggers, and missing secret protections.
"""

import json
import re
import sys
from pathlib import Path
from datetime import datetime

try:
    import yaml
except ImportError:
    yaml = None


class GitHubActionsSecurityAgent:
    """Audits GitHub Actions workflows for security vulnerabilities."""

    def __init__(self, repo_path=".", output_dir="./gha_audit"):
        self.repo_path = Path(repo_path)
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.findings = []

    def _load_workflow(self, path):
        if yaml:
            with open(path) as f:
                return yaml.safe_load(f)
        with open(path) as f:
            return {"raw": f.read()}

    def find_workflows(self):
        """Discover all workflow files in the repository."""
        wf_dir = self.repo_path / ".github" / "workflows"
        if not wf_dir.exists():
            return []
        return sorted(wf_dir.glob("*.yml")) + sorted(wf_dir.glob("*.yaml"))

    def check_sha_pinning(self, workflow_path, content):
        """Check if actions are pinned to SHA digests."""
        unpinned = []
        raw = content.get("raw", "") if "raw" in content else ""
        if not raw:
            try:
                raw = Path(workflow_path).read_text()
            except Exception:
                return unpinned

        for line_num, line in enumerate(raw.splitlines(), 1):
            m = re.search(r'uses:\s+([^@\s]+)@([^\s#]+)', line)
            if m:
                action, ref = m.group(1), m.group(2)
                if not re.match(r'^[a-f0-9]{40}$', ref):
                    unpinned.append({
                        "action": action,
                        "ref": ref,
                        "line": line_num,
                        "file": str(workflow_path),
                    })
                    self.findings.append({
                        "severity": "medium",
                        "type": "Unpinned Action",
                        "detail": f"{action}@{ref} at line {line_num}",
                        "file": str(workflow_path),
                    })
        return unpinned

    def check_permissions(self, workflow_path, content):
        """Check for overly permissive GITHUB_TOKEN permissions."""
        issues = []
        if not isinstance(content, dict) or "raw" in content:
            return issues

        top_perms = content.get("permissions")
        if top_perms is None:
            issues.append({
                "issue": "No top-level permissions defined (inherits defaults)",
                "file": str(workflow_path),
            })
            self.findings.append({
                "severity": "medium",
                "type": "Missing Permissions",
                "detail": "Workflow has no permissions block",
                "file": str(workflow_path),
            })

        if top_perms == "write-all" or (isinstance(top_perms, dict) and
                top_perms.get("contents") == "write" and
                top_perms.get("actions") == "write"):
            issues.append({"issue": "Overly permissive write-all", "file": str(workflow_path)})
            self.findings.append({
                "severity": "high",
                "type": "Excessive Permissions",
                "detail": "write-all permissions granted",
                "file": str(workflow_path),
            })

        return issues

    def check_script_injection(self, workflow_path, content):
        """Check for user-controlled input in run steps (script injection)."""
        injections = []
        raw = content.get("raw", "") if "raw" in content else ""
        if not raw:
            try:
                raw = Path(workflow_path).read_text()
            except Exception:
                return injections

        dangerous_contexts = [
            "github.event.pull_request.title",
            "github.event.pull_request.body",
            "github.event.issue.title",
            "github.event.issue.body",
            "github.event.comment.body",
            "github.event.review.body",
            "github.head_ref",
        ]

        in_run = False
        for line_num, line in enumerate(raw.splitlines(), 1):
            stripped = line.strip()
            if stripped.startswith("run:") or stripped.startswith("run: |"):
                in_run = True
            elif in_run and not stripped.startswith("-") and not stripped.startswith("#"):
                for ctx in dangerous_contexts:
                    if f"${{{{ {ctx}" in line or f"${{{{{ctx}" in line:
                        injections.append({
                            "context": ctx,
                            "line": line_num,
                            "file": str(workflow_path),
                        })
                        self.findings.append({
                            "severity": "high",
                            "type": "Script Injection",
                            "detail": f"{ctx} in run step at line {line_num}",
                            "file": str(workflow_path),
                        })
            if stripped and not stripped.startswith("-") and not stripped.startswith("#") and ":" in stripped and not stripped.startswith("run"):
                in_run = False

        return injections

    def check_dangerous_triggers(self, workflow_path, content):
        """Check for dangerous event triggers."""
        issues = []
        if not isinstance(content, dict) or "raw" in content:
            raw = content.get("raw", "")
            if "pull_request_target" in raw:
                issues.append({"trigger": "pull_request_target", "file": str(workflow_path)})
                self.findings.append({
                    "severity": "high",
                    "type": "Dangerous Trigger",
                    "detail": "pull_request_target allows fork code to run with base permissions",
                    "file": str(workflow_path),
                })
            return issues

        on_block = content.get("on", content.get(True, {}))
        if isinstance(on_block, dict) and "pull_request_target" in on_block:
            issues.append({"trigger": "pull_request_target", "file": str(workflow_path)})
            self.findings.append({
                "severity": "high",
                "type": "Dangerous Trigger",
                "detail": "pull_request_target trigger used",
                "file": str(workflow_path),
            })
        return issues

    def audit_all(self):
        """Run all security checks on all workflow files."""
        workflows = self.find_workflows()
        results = []
        for wf in workflows:
            content = self._load_workflow(wf)
            unpinned = self.check_sha_pinning(wf, content)
            perms = self.check_permissions(wf, content)
            injections = self.check_script_injection(wf, content)
            triggers = self.check_dangerous_triggers(wf, content)
            results.append({
                "workflow": str(wf),
                "unpinned_actions": len(unpinned),
                "permission_issues": len(perms),
                "script_injections": len(injections),
                "dangerous_triggers": len(triggers),
            })
        return results

    def generate_report(self):
        audit = self.audit_all()
        report = {
            "report_date": datetime.utcnow().isoformat(),
            "repository": str(self.repo_path),
            "workflows_scanned": len(audit),
            "audit_summary": audit,
            "findings": self.findings,
            "total_findings": len(self.findings),
        }
        out = self.output_dir / "gha_security_report.json"
        with open(out, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))
        return report


def main():
    repo = sys.argv[1] if len(sys.argv) > 1 else "."
    agent = GitHubActionsSecurityAgent(repo)
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/securing-github-actions-workflows/scripts/process.py =====
#!/usr/bin/env python3
"""
GitHub Actions Workflow Security Audit Script

Analyzes workflow files for security issues including unpinned actions,
excessive permissions, script injection risks, and insecure patterns.

Usage:
    python process.py --workflows-dir .github/workflows/ --output audit-report.json
"""

import argparse
import json
import os
import re
import sys
from dataclasses import dataclass, field
from datetime import datetime, timezone
from pathlib import Path

import yaml


@dataclass
class SecurityFinding:
    file: str
    line: int
    check: str
    severity: str
    message: str
    remediation: str


SHA_PATTERN = re.compile(r"@[0-9a-f]{40}")
TAG_PATTERN = re.compile(r"@v?\d+(\.\d+)*$")
INJECTION_PATTERN = re.compile(r"\$\{\{\s*github\.event\.(issue|pull_request|comment|review)\.\w+")
DANGEROUS_CONTEXTS = [
    "github.event.issue.title",
    "github.event.issue.body",
    "github.event.pull_request.title",
    "github.event.pull_request.body",
    "github.event.comment.body",
    "github.event.review.body",
    "github.head_ref",
]


def load_workflow(filepath: str) -> dict:
    """Load a GitHub Actions workflow YAML file."""
    try:
        with open(filepath, "r") as f:
            return yaml.safe_load(f) or {}
    except (yaml.YAMLError, FileNotFoundError):
        return {}


def check_action_pinning(workflow: dict, filepath: str) -> list:
    """Check if actions are pinned to SHA digests."""
    findings = []
    filename = os.path.basename(filepath)

    for job_name, job in workflow.get("jobs", {}).items():
        for i, step in enumerate(job.get("steps", [])):
            uses = step.get("uses", "")
            if not uses or uses.startswith("./"):
                continue
            if not SHA_PATTERN.search(uses):
                findings.append(SecurityFinding(
                    file=filename, line=0,
                    check="ACTION_PINNING",
                    severity="HIGH",
                    message=f"Job '{job_name}' step {i}: '{uses}' not pinned to SHA digest",
                    remediation=f"Pin to SHA: {uses.split('@')[0]}@<commit-sha>"
                ))
    return findings


def check_permissions(workflow: dict, filepath: str) -> list:
    """Check for overly permissive GITHUB_TOKEN permissions."""
    findings = []
    filename = os.path.basename(filepath)

    top_perms = workflow.get("permissions")
    if top_perms is None:
        findings.append(SecurityFinding(
            file=filename, line=0,
            check="PERMISSIONS",
            severity="MEDIUM",
            message="No top-level permissions defined. Inherits default (may be write-all).",
            remediation="Add 'permissions: {}' at workflow level and grant per-job."
        ))
    elif top_perms == "write-all" or (isinstance(top_perms, dict) and
                                       all(v == "write" for v in top_perms.values())):
        findings.append(SecurityFinding(
            file=filename, line=0,
            check="PERMISSIONS",
            severity="HIGH",
            message="Workflow has write-all permissions.",
            remediation="Restrict to minimum required permissions per job."
        ))
    return findings


def check_script_injection(workflow: dict, filepath: str) -> list:
    """Check for script injection via user-controlled inputs."""
    findings = []
    filename = os.path.basename(filepath)

    for job_name, job in workflow.get("jobs", {}).items():
        for i, step in enumerate(job.get("steps", [])):
            run_cmd = step.get("run", "")
            if not run_cmd:
                continue
            for ctx in DANGEROUS_CONTEXTS:
                if f"${{{{ {ctx}" in run_cmd or f"${{{{{ctx}" in run_cmd:
                    findings.append(SecurityFinding(
                        file=filename, line=0,
                        check="SCRIPT_INJECTION",
                        severity="CRITICAL",
                        message=f"Job '{job_name}' step {i}: '{ctx}' interpolated in run step",
                        remediation="Use env variable: env: VAR: ${{ " + ctx + " }} then ${VAR}"
                    ))
    return findings


def check_pr_target(workflow: dict, filepath: str) -> list:
    """Check for dangerous pull_request_target usage."""
    findings = []
    filename = os.path.basename(filepath)

    triggers = workflow.get("on", {})
    if isinstance(triggers, dict) and "pull_request_target" in triggers:
        for job_name, job in workflow.get("jobs", {}).items():
            for step in job.get("steps", []):
                uses = step.get("uses", "")
                if "checkout" in uses:
                    with_ref = step.get("with", {}).get("ref", "")
                    if "pull_request" in with_ref or "head" in with_ref:
                        findings.append(SecurityFinding(
                            file=filename, line=0,
                            check="PR_TARGET_CHECKOUT",
                            severity="CRITICAL",
                            message=f"Job '{job_name}': pull_request_target with PR code checkout",
                            remediation="Never checkout PR code in pull_request_target workflows."
                        ))
    return findings


def main():
    parser = argparse.ArgumentParser(description="GitHub Actions Security Audit")
    parser.add_argument("--workflows-dir", required=True)
    parser.add_argument("--output", default="actions-security-report.json")
    parser.add_argument("--fail-on-findings", action="store_true")
    args = parser.parse_args()

    workflows_dir = os.path.abspath(args.workflows_dir)
    all_findings = []

    workflow_files = list(Path(workflows_dir).glob("*.yml")) + list(Path(workflows_dir).glob("*.yaml"))
    print(f"[*] Auditing {len(workflow_files)} workflow files in {workflows_dir}")

    for wf_path in workflow_files:
        workflow = load_workflow(str(wf_path))
        if not workflow:
            continue

        all_findings.extend(check_action_pinning(workflow, str(wf_path)))
        all_findings.extend(check_permissions(workflow, str(wf_path)))
        all_findings.extend(check_script_injection(workflow, str(wf_path)))
        all_findings.extend(check_pr_target(workflow, str(wf_path)))

    severity_counts = {}
    for f in all_findings:
        severity_counts[f.severity] = severity_counts.get(f.severity, 0) + 1

    report = {
        "metadata": {
            "directory": workflows_dir,
            "date": datetime.now(timezone.utc).isoformat(),
            "workflows_scanned": len(workflow_files)
        },
        "summary": {
            "total_findings": len(all_findings),
            "severity_counts": severity_counts
        },
        "findings": [
            {"file": f.file, "check": f.check, "severity": f.severity,
             "message": f.message, "remediation": f.remediation}
            for f in sorted(all_findings,
                            key=lambda x: {"CRITICAL": 0, "HIGH": 1, "MEDIUM": 2, "LOW": 3}.get(x.severity, 4))
        ]
    }

    output_path = os.path.abspath(args.output)
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[*] Report: {output_path}")

    for f in all_findings:
        print(f"  [{f.severity}] {f.file}: {f.message}")

    passed = len(all_findings) == 0
    print(f"\n[{'PASS' if passed else 'FAIL'}] {len(all_findings)} security findings")

    if args.fail_on_findings and not passed:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====
