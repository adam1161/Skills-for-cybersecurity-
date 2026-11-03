

===== FILE: ./.claude-plugin/marketplace.json =====
{
  "name": "anthropic-cybersecurity-skills",
  "owner": {
    "name": "mukul975",
    "email": "mukuljangra5@gmail.com"
  },
  "metadata": {
    "description": "818 cybersecurity skills for AI agents mapped to 6 frameworks: MITRE ATT&CK, NIST CSF 2.0, MITRE ATLAS, D3FEND, NIST AI RMF, and the MITRE Fight Fraud Framework (F3).",
    "version": "1.3.0"
  },
  "plugins": [
    {
      "name": "cybersecurity-skills",
      "source": "./",
      "description": "818 cybersecurity skills covering web security, pentesting, DFIR, threat intelligence, cloud security, malware analysis, and more. Mapped to 6 frameworks.",
      "version": "1.3.0",
      "author": {
        "name": "mukul975"
      },
      "license": "Apache-2.0",
      "keywords": [
        "cybersecurity",
        "pentesting",
        "forensics",
        "threat-intelligence",
        "cloud-security",
        "malware-analysis",
        "incident-response",
        "zero-trust",
        "devsecops"
      ],
      "category": "security",
      "homepage": "https://github.com/mukul975/Anthropic-Cybersecurity-Skills",
      "repository": "https://github.com/mukul975/Anthropic-Cybersecurity-Skills"
    }
  ]
}

===== END FILE =====


===== FILE: ./.claude-plugin/plugin.json =====
{
  "name": "cybersecurity-skills",
  "description": "818 cybersecurity skills covering web security, pentesting, DFIR, threat intelligence, cloud security, malware analysis, and more.",
  "version": "1.3.0"
}

===== END FILE =====


===== FILE: ./.github/FUNDING.yml =====
github: mukul975
custom: ["https://paypal.me/mahipaljangra"]

===== END FILE =====


===== FILE: ./.github/ISSUE_TEMPLATE/bug-report.yml =====
name: Bug Report
description: Report a SKILL.md validation error, broken script, or incorrect content
title: "[Bug]: "
labels: ["bug", "needs-triage"]
body:
  - type: markdown
    attributes:
      value: |
        Thanks for taking the time to report a bug!
  - type: input
    id: skill-name
    attributes:
      label: Skill Name
      description: Which skill has the issue?
      placeholder: e.g., analyzing-disk-image-with-autopsy
    validations:
      required: true
  - type: dropdown
    id: bug-type
    attributes:
      label: Bug Type
      options:
        - SKILL.md validation error
        - Broken/incorrect script
        - Wrong instructions or commands
        - Missing required files
        - Incorrect metadata/frontmatter
        - Other
    validations:
      required: true
  - type: textarea
    id: description
    attributes:
      label: Description
      description: What is the issue?
    validations:
      required: true
  - type: textarea
    id: expected
    attributes:
      label: Expected Behavior
      description: What should happen instead?
    validations:
      required: true
  - type: input
    id: ai-agent
    attributes:
      label: AI Agent Used
      description: Which AI agent were you using?
      placeholder: e.g., Claude Code, GitHub Copilot, Codex CLI

===== END FILE =====


===== FILE: ./.github/ISSUE_TEMPLATE/config.yml =====
blank_issues_enabled: false
contact_links:
  - name: Security Vulnerability
    url: https://github.com/mukul975/Anthropic-Cybersecurity-Skills/security/advisories/new
    about: Report a security vulnerability in this repository
  - name: Discussion
    url: https://github.com/mukul975/Anthropic-Cybersecurity-Skills/discussions
    about: Ask questions or discuss ideas

===== END FILE =====


===== FILE: ./.github/ISSUE_TEMPLATE/improve-skill.md =====
---
name: Improve existing skill
about: Suggest improvements to an existing skill
title: '[IMPROVE] skill-name-here'
labels: 'enhancement'
assignees: ''
---

## Skill to improve
<!-- Folder name of the skill -->

## What needs improvement?
- [ ] agent.py has errors or placeholders
- [ ] api-reference.md is incomplete
- [ ] SKILL.md frontmatter is missing fields
- [ ] ATT&CK mapping is incorrect
- [ ] Other:

## Suggested improvement

===== END FILE =====


===== FILE: ./.github/ISSUE_TEMPLATE/new-skill-request.yml =====
name: New Skill Request
description: Request a new cybersecurity skill to be added to the database
title: "[Skill Request]: "
labels: ["enhancement", "new-skill", "help wanted"]
body:
  - type: markdown
    attributes:
      value: |
        Request a new cybersecurity skill. The more detail you provide, the faster we can add it!
  - type: input
    id: skill-name
    attributes:
      label: Proposed Skill Name
      description: Kebab-case gerund form (e.g., analyzing-memory-dump-with-volatility)
      placeholder: performing-task-name
    validations:
      required: true
  - type: dropdown
    id: category
    attributes:
      label: Category
      options:
        - Threat Detection
        - Incident Response
        - Penetration Testing
        - Digital Forensics
        - Compliance & Governance
        - Network Security
        - Cloud Security
        - Application Security
        - Malware Analysis
        - OSINT
        - Zero Trust Architecture
        - OT/ICS Security
        - DevSecOps
        - Ransomware Defense
        - Threat Intelligence
        - Other
    validations:
      required: true
  - type: textarea
    id: description
    attributes:
      label: Skill Description
      description: What should this skill teach an AI agent to do?
    validations:
      required: true
  - type: input
    id: mitre-attack
    attributes:
      label: MITRE ATT&CK Technique(s)
      description: Optional - relevant technique IDs
      placeholder: e.g., T1059, T1078
  - type: textarea
    id: tools
    attributes:
      label: Key Tools
      description: What tools/commands should this skill cover?

===== END FILE =====


===== FILE: ./.github/ISSUE_TEMPLATE/new-skill.md =====
---
name: Add new skill
about: Propose a new cybersecurity skill for the database
title: '[NEW SKILL] skill-name-here'
labels: 'new-skill, good first issue'
assignees: ''
---

## Skill name (kebab-case)
<!-- e.g., detecting-lateral-movement-in-azure -->

## Domain / Subdomain
<!-- e.g., cybersecurity / threat-hunting -->

## Description
<!-- One sentence describing what this skill does -->

## MITRE ATT&CK techniques
<!-- e.g., T1021.001, T1078.004 -->

## NIST CSF function
<!-- Identify, Protect, Detect, Respond, or Recover -->

## Why is this skill needed?
<!-- What problem does it solve for security practitioners? -->

===== END FILE =====


===== FILE: ./.github/ISSUE_TEMPLATE/skill-improvement.yml =====
name: Skill Improvement
description: Suggest improvements to an existing skill
title: "[Improvement]: "
labels: ["enhancement", "skill-improvement"]
body:
  - type: input
    id: skill-name
    attributes:
      label: Skill Name
      placeholder: e.g., analyzing-network-traffic-with-wireshark
    validations:
      required: true
  - type: dropdown
    id: improvement-type
    attributes:
      label: Type of Improvement
      options:
        - More accurate/updated instructions
        - Better workflow steps
        - Add missing tools or commands
        - Improve description for agent discovery
        - Add MITRE ATT&CK mapping
        - Add NIST CSF alignment
        - Improve scripts/assets
        - Fix outdated content
    validations:
      required: true
  - type: textarea
    id: current-issue
    attributes:
      label: Current Issue
      description: What is wrong or missing?
    validations:
      required: true
  - type: textarea
    id: suggested-improvement
    attributes:
      label: Suggested Improvement
      description: What should be changed or added?
    validations:
      required: true

===== END FILE =====


===== FILE: ./.github/copilot-instructions.md =====
# Copilot Instructions for Anthropic-Cybersecurity-Skills

**Anthropic-Cybersecurity-Skills** is the largest open-source cybersecurity skills library for AI agents, containing 817 production-grade skills mapped to 6 industry frameworks (MITRE ATT&CK, NIST CSF, MITRE ATLAS, MITRE D3FEND, NIST AI RMF, MITRE F3). This document guides Copilot agents contributing new skills and framework mappings.

## Quick Facts

- **Type**: Open-source cybersecurity skills library + framework mapping hub
- **Skills**: 817 across 29 security domains
- **Frameworks**: 6 (MITRE ATT&CK v14, NIST CSF 2.0, MITRE ATLAS, MITRE D3FEND, NIST AI RMF, MITRE F3)
- **Format**: agentskills.io standard (YAML frontmatter + Markdown)
- **License**: Apache 2.0 (ethical use required)
- **Community**: Independent, community-created (not affiliated with Anthropic)

## Repository Structure

```
Anthropic-Cybersecurity-Skills/
├── skills/                    # 817 skill directories (kebab-case)
│   ├── abusing-dpapi-for-credential-access/
│   │   ├── SKILL.md          # Frontmatter + detailed instructions
│   │   ├── LICENSE
│   │   ├── scripts/
│   │   │   └── process.py    # Optional helper scripts
│   │   └── references/
│   │       ├── api-reference.md
│   │       ├── standards.md
│   │       └── workflows.md
│   └── ... (816 more)
├── mappings/                  # Framework coverage & alignment
│   ├── mitre-attack/
│   │   ├── attack-navigator-layer.json
│   │   └── coverage-summary.md
│   ├── nist-csf/
│   ├── owasp/
│   └── README.md
├── docs/                      # Additional documentation
├── index.json                 # Central skill registry (auto-generated)
├── CONTRIBUTING.md            # Contribution guide
├── SECURITY.md               # Ethical use & dual-use policies
└── CODE_OF_CONDUCT.md        # Community guidelines
```

## Build & Development

### Prerequisites

- **Git** (for cloning and version control)
- **Python 3.8+** (optional, for scripts/metadata generation)
- **jq** (optional, for JSON processing; useful for index.json queries)

### Local Setup

```bash
# Clone the repository
git clone https://github.com/mukul975/Anthropic-Cybersecurity-Skills.git
cd Anthropic-Cybersecurity-Skills

# No installation needed—skills are static Markdown files
# View a skill directly:
cat skills/abusing-dpapi-for-credential-access/SKILL.md

# Search for skills by keyword:
grep -r "credential-access" skills/*/SKILL.md | head -10

# Query the central index:
jq '.skills[] | select(.description | contains("Active Directory"))' index.json
```

### Generating/Updating index.json

When adding new skills, the index must be regenerated:

```bash
# If a script exists (check repository):
python3 generate_index.py

# Otherwise, manually verify:
jq '.skills | length' index.json  # Should match skill directory count
```

## Architecture & Concepts

### Skill Structure

Each skill is a **self-contained directory** following the `agentskills.io` standard:

```
skills/skill-name/
├── SKILL.md              # The authoritative skill definition
├── LICENSE               # Apache 2.0 (usually)
├── scripts/
│   └── process.py        # Optional: helper scripts, agent implementations
└── references/
    ├── api-reference.md  # Technical API docs for the skill
    ├── standards.md      # CVE, NIST, MITRE refs
    └── workflows.md      # Deep technical procedures
```

### SKILL.md Format

Every skill follows this structure:

```markdown
---
name: skill-name-kebab-case
description: >-
  Clear, agent-discoverable description.
  Include keywords for search/filtering.
  This is what agents read to decide whether to use this skill.
domain: cybersecurity
subdomain: red-teaming  # e.g. digital-forensics, compliance-governance, etc.
tags:
  - tool-names (mimikatz, burp-suite, etc.)
  - frameworks (active-directory, cloud, kubernetes, etc.)
  - techniques (credential-access, privilege-escalation, etc.)
version: "1.0"
author: github-username
license: Apache-2.0
nist_csf:
  - DE.CM-01
  - PR.AC-01
mitre_attack:
  - T1555.004
  - T1078.002
mitre_atlas:
  - AML.P1.003
mitre_d3fend:
  - D3-CAA
  - D3-PCA
nist_ai_rmf:
  - GOV-1
mitre_f3:
  - FI-B-003
---

# Skill Title

> **Legal Notice:** This skill is for authorized [penetration testing/research/defense] only.
> Unauthorized use may violate computer fraud laws. Always operate within rules-of-engagement.

## Overview

Clear, concise explanation of what the skill does and why an agent needs it.
Include threat model context if applicable.

## When to Use

- Specific scenario 1
- Specific scenario 2
- Real-world contexts where this skill applies

## Prerequisites

- Required tools (with install commands if applicable)
- Required access/permissions
- Required knowledge/certifications
- System requirements

## Workflow

### Step 1: [Action]
Clear instructions with real commands.
```bash
tool-name --flag value
```

### Step 2: [Next Action]
Continue with detailed steps.

### Output Format

What success looks like:
```
Expected output or indicator
```

## Key Concepts

| Term | Definition |
|------|-----------|
| Concept1 | Explanation |
| Concept2 | Explanation |

## Tools & Systems

- **Tool A** — What it does, where to get it
- **Tool B** — What it does, where to get it

## Common Scenarios

### Scenario 1
When X, do Y.

### Scenario 2
When A, do B.

## References

- [MITRE ATT&CK: T1555.004](https://attack.mitre.org/techniques/T1555/004/)
- [NIST CSF: DE.CM-01](https://csrc.nist.gov/)
```

### Framework Mappings

Each skill can map to multiple frameworks. Key mappings:

| Framework | Scope | Example |
|-----------|-------|---------|
| **MITRE ATT&CK** | Adversarial tactics/techniques | T1555.004 (Credentials from Password Stores) |
| **NIST CSF 2.0** | Cybersecurity functions & categories | DE.CM (Detect - Monitor) |
| **MITRE ATLAS** | AI/ML system attacks | AML.P (AI Preparation) |
| **MITRE D3FEND** | Defensive techniques | D3-CAA (Capture Analysis Analytics) |
| **NIST AI RMF** | AI risk management | GOV (Governance) |
| **MITRE F3** | Fraud-specific techniques | FI-B (Fraud Impact) |

### Subdomains (Choose One)

- **web-application-security** — OWASP, API, web app testing
- **network-security** — Network tools, protocols, monitoring
- **penetration-testing** — General penetration testing methodology
- **red-teaming** — Simulating advanced attackers (C2, evasion, etc.)
- **digital-forensics** — Incident response, forensic analysis, disk imaging
- **malware-analysis** — Static/dynamic malware analysis, reverse engineering
- **threat-intelligence** — Gathering, analyzing, sharing threat data
- **cloud-security** — AWS/Azure/GCP-specific security
- **container-security** — Docker, Kubernetes, container runtime
- **identity-access-management** — Active Directory, IAM, authentication
- **cryptography** — Encryption, hashing, key management
- **vulnerability-management** — Scanning, assessment, remediation
- **compliance-governance** — CMMC, HIPAA, SOC2, auditing
- **zero-trust-architecture** — Zero-trust implementation patterns
- **ot-ics-security** — Operational technology, ICS/SCADA
- **devsecops** — Secure software development, CI/CD security

## Conventions & Patterns

### Naming

- **Directory/file**: kebab-case, lowercase with hyphens (e.g., `abusing-dpapi-for-credential-access`)
- **Skill name**: Same as directory (in YAML frontmatter)
- **GitHub usernames**: Use lowercase GitHub username as author

### Description Quality

Descriptions should be **agent-discoverable** — concise, keyword-rich, action-oriented:

```yaml
# ✗ Vague
description: How to abuse DPAPI

# ✓ Clear and searchable
description: >-
  Extract DPAPI-protected secrets such as credentials and browser data
  offline and online using SharpDPAPI, Mimikatz, or impacket. Ideal
  for post-exploitation Windows credential harvesting and offline analysis.
```

### Tags Strategy

Use 3-5 tags for discoverability:

```yaml
tags:
  - tool-names           # mimikatz, sharpdpapi, burp-suite
  - attack-frameworks    # active-directory, kerberos, oauth
  - techniques           # credential-access, privilege-escalation, lateral-movement
  - platforms            # windows, linux, macos, cloud
  - use-cases            # post-exploitation, threat-intel, forensics
```

### Framework ID Format

IDs are **case-sensitive** and **exact**:

```yaml
mitre_attack:
  - T1055                   # Parent technique
  - T1055.001               # Sub-technique
nist_csf:
  - DE.CM-01                # NIST Cybersecurity Framework 2.0
mitre_atlas:
  - AML.P1.003              # MITRE ATLAS for AI/ML
mitre_d3fend:
  - D3-CAA                  # MITRE D3FEND defensive technique ID
nist_ai_rmf:
  - GOV-1                   # NIST AI Risk Management Framework
mitre_f3:
  - FI-B-003                # MITRE Fight Fraud Framework
```

## Common Tasks

### Adding a New Skill

1. **Create skill directory** (kebab-case):
   ```bash
   mkdir -p skills/your-skill-name
   ```

2. **Create SKILL.md** with required frontmatter:
   ```bash
   cat > skills/your-skill-name/SKILL.md << 'EOF'
   ---
   name: your-skill-name
   description: >-
     Clear, discoverable description with keywords.
   domain: cybersecurity
   subdomain: red-teaming
   tags:
     - tool-name
     - technique
     - use-case
   version: "1.0"
   author: your-github-username
   license: Apache-2.0
   mitre_attack:
     - T1234.567
   nist_csf:
     - DE.CM-01
   mitre_atlas:
     - AML.P1.003
   mitre_d3fend:
     - D3-CAA
   nist_ai_rmf:
     - GOV-1
   mitre_f3:
     - FI-B-003
   ---
   
   # Skill Title
   
   > **Legal Notice:** Authorized use only. [...]
   
   ## Overview
   
   Clear explanation...
   EOF
   ```

3. **Write detailed sections** in Markdown:
   - When to Use (specific scenarios)
   - Prerequisites (tools, permissions, access)
   - Workflow (numbered steps with real commands)
   - Key Concepts (table for terminology)
   - Tools & Systems
   - Common Scenarios
   - References (framework links)

4. **Add optional supporting files**:
   ```
   scripts/process.py         # Helper script or agent implementation
   references/standards.md    # CVE, NIST, MITRE links
   references/workflows.md    # Deep technical procedures
   ```

5. **Add LICENSE**:
   ```bash
   cp LICENSE skills/your-skill-name/LICENSE
   # Or use a specific open-source license file
   ```

6. **Regenerate index** (if automation exists):
   ```bash
   python3 generate_index.py  # Updates index.json
   ```

7. **Submit PR**:
   ```bash
   git add skills/your-skill-name
   git commit -m "Add skill: your-skill-name"
   git push origin feature/add-skill-name
   # Create PR with title: "Add skill: your-skill-name"
   ```

### Updating Framework Mappings

If a skill maps to new frameworks or techniques change:

1. **Update SKILL.md frontmatter**:
   ```yaml
   mitre_attack:
     - T1555.004      # Add new technique IDs
   nist_csf:
     - DE.CM-01       # Add new control IDs
   ```

2. **Regenerate index** (if automation exists):
   ```bash
   python3 generate_index.py
   ```

3. **Verify mapping coverage**:
   ```bash
   # Check if all referenced IDs are valid:
   grep -r "T1[0-9]" skills/*/SKILL.md | grep -v "http"
   ```

### Searching Skills

**By keyword**:
```bash
grep -r "active-directory" skills/*/SKILL.md
```

**By framework**:
```bash
grep -r "T1055" skills/*/SKILL.md  # MITRE ATT&CK technique
```

**By subdomain**:
```bash
grep "subdomain: red-teaming" skills/*/SKILL.md
```

**Using jq (if index.json exists)**:
```bash
# Find skills by keyword
jq '.skills[] | select(.tags[] | contains("credential-access"))' index.json

# Count skills by subdomain
jq '[.skills[] | .subdomain] | group_by(.) | map({subdomain: .[0], count: length})' index.json
```

## Platform & Framework Notes

### Windows-Specific Skills

- Often leverage PowerShell, Windows APIs, Active Directory
- Reference MITRE ATT&CK Windows tactics: T1021 (Lateral Movement), T1078 (Valid Accounts)
- Include prerequisite (SYSTEM/Administrator access, domain join, etc.)

### Linux/macOS Skills

- Use standard Unix tools (bash, Python, curl, etc.)
- Note platform availability differences
- Cloud/container skills often multi-platform

### Cloud Security Skills

- Specify cloud provider (AWS, Azure, GCP, multi-cloud)
- Reference cloud-specific tools (awscli, az, gcloud)
- Map to cloud-specific MITRE ATLAS techniques

### AI/ML Attack Skills

- Use MITRE ATLAS techniques (AML.P1, AML.E1, etc.)
- Include model/system type (LLM, transformer, computer vision, etc.)
- Note NIST AI RMF alignment (GOV, MAP, MEASURE, MANAGE)

## Gotchas & Known Issues

### Legal & Ethical

- **Dual-use policy**: Skills for red-teaming and exploitation require legal notice
  - Must include "authorized use only" disclaimer
  - Reference SECURITY.md for policy
- **No credentials**: Never embed API keys, tokens, or credentials
- **Attribution**: Cite original tool authors and researchers

### Framework Maintenance

- **MITRE ATT&CK updates**: v14 is current; check attack.mitre.org for latest
- **NIST CSF 2.0**: Rolled out Feb 2024; use subcategory IDs as in this repo (e.g., `DE.CM-01`, `PR.PS-01`)
- **Technique changes**: Techniques may deprecate; verify via attack.mitre.org

### Subdomain Assignment

- **Common mistake**: Using wrong subdomain (e.g., "red-teaming" for defensive skill)
  - red-teaming = offensive/attacker perspective
  - compliance-governance = defensive/compliance perspective
  - Choose the **primary** subdomain if skill spans multiple

### index.json Generation

- If index doesn't auto-regenerate, manually verify:
  ```bash
  # Count skills in index vs directories
  jq '.skills | length' index.json
  ls -d skills/*/ | wc -l
  # Should match (or index may be stale)
  ```

## Testing & Quality

### Skill Quality Checklist

Before submitting a PR:

- [ ] **Name**: Kebab-case, 1-64 chars, descriptive
- [ ] **Description**: Clear, includes keywords, discoverable by agents
- [ ] **Instructions**: Actionable with real commands and tool names
- [ ] **Subdomain**: Correctly assigned (red-teaming vs defensive)
- [ ] **Tags**: 3-5 relevant tags (tools, techniques, platforms)
- [ ] **Framework IDs**: Valid MITRE ATT&CK, NIST CSF, MITRE ATLAS IDs
- [ ] **Legal notice**: Included if skill is offensive/dual-use
- [ ] **References**: Links to official framework docs
- [ ] **Formatting**: Proper Markdown, no typos, code blocks highlighted

### Manual Verification

```bash
# Validate skill frontmatter and conventions (repo validator):
python3 tools/validate-skill.py skills/my-skill/
# Or validate all skills:
python3 tools/validate-skill.py --all

# Check for framework ID patterns:
grep -E "^  - (T1[0-9]{3,4}(\.[0-9]{3})?|DE\.[A-Z]{2}-[0-9]{2}|AML\.)" skills/*/SKILL.md

# Verify all referenced skills have directories:
jq -r '.skills[].name' index.json | while read skill; do
  [ -d "skills/$skill" ] || echo "Missing: $skill"
done
```

## Contributing Notes

- **No build required** — Skills are static files; git pull = ready to use
- **Skill dependencies**: Skills are independent; if skill A requires skill B, note it in the workflow
- **Tool versions**: Mention tool versions in prerequisites (e.g., "Burp Suite 2024.1+")
- **Testing**: Test commands on actual systems before submitting
- **Code of Conduct**: See CODE_OF_CONDUCT.md — be respectful, follow ethical use policy

## Resources & Links

- **agentskills.io Standard**: https://agentskills.io (format specification)
- **MITRE ATT&CK**: https://attack.mitre.org (techniques, tactics)
- **NIST Cybersecurity Framework**: https://csrc.nist.gov/projects/cybersecurity-framework (controls)
- **MITRE ATLAS**: https://atlas.mitre.org (AI/ML attacks)
- **MITRE D3FEND**: https://d3fend.mitre.org (defensive techniques)
- **NIST AI RMF**: https://nvlabs.nist.gov/display/AIRFF (AI risk management)
- **Contributing Guide**: `CONTRIBUTING.md` in repo
- **Security Policy**: `SECURITY.md` — dual-use & ethical use
- **Code of Conduct**: `CODE_OF_CONDUCT.md`
- **Community Playground**: https://casky.ai (test skills in browser)

## Quick Reference

| Task | Command |
|------|---------|
| Add skill | `mkdir skills/name && cat > SKILL.md` |
| Search by technique | `grep -r "T1055" skills/` |
| Search by subdomain | `grep "subdomain: red-teaming" skills/*/SKILL.md` |
| Validate skill | `python3 tools/validate-skill.py skills/my-skill/` |
| Regenerate index | `python3 generate_index.py` (if exists) |
| View mapping coverage | Open `mappings/mitre-attack/attack-navigator-layer.json` in ATT&CK Navigator |

===== END FILE =====


===== FILE: ./.github/workflows/sync-marketplace-version.yml =====
name: Sync Marketplace Version on Release

on:
  release:
    types: [published]

jobs:
  sync-version:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          ref: main
          fetch-depth: 0

      - name: Extract version from tag
        id: version
        run: |
          VERSION=${GITHUB_REF_NAME#v}
          echo "version=$VERSION" >> $GITHUB_OUTPUT
          echo "tag=$GITHUB_REF_NAME" >> $GITHUB_OUTPUT

      - name: Update marketplace.json and plugin.json version
        env:
          VERSION: ${{ steps.version.outputs.version }}
        run: |
          jq --arg v "$VERSION" '.metadata.version = $v | .plugins[].version = $v' .claude-plugin/marketplace.json > tmp.json
          mv tmp.json .claude-plugin/marketplace.json
          jq --arg v "$VERSION" '.version = $v' .claude-plugin/plugin.json > tmp.json
          mv tmp.json .claude-plugin/plugin.json
          echo "Updated marketplace.json and plugin.json to version $VERSION"

      - name: Commit and push
        run: |
          git config user.name "mukul975"
          git config user.email "mukuljangra5@gmail.com"
          git add .claude-plugin/marketplace.json .claude-plugin/plugin.json
          git diff --staged --quiet || git commit -m "chore: bump plugin version to ${{ steps.version.outputs.tag }}"
          git push origin HEAD:main

===== END FILE =====


===== FILE: ./.github/workflows/update-contributors.yml =====
name: Update contributors

on:
  push:
    branches: [main]
  schedule:
    # GitHub's contributors API is cached and lags a merge by up to ~24h, so a
    # weekly pass catches anyone the push-triggered run was too early to see.
    - cron: '17 4 * * 1'
  workflow_dispatch:

# Only one run at a time; a burst of merges must not race on README.md.
concurrency:
  group: update-contributors
  cancel-in-progress: false

jobs:
  update:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Regenerate the contributor wall
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: python3 tools/update-contributors.py

      - name: Commit if it changed
        run: |
          git config user.name "mukul975"
          git config user.email "mukuljangra5@gmail.com"
          git add README.md
          if git diff --staged --quiet; then
            echo "No new contributors."
            exit 0
          fi
          git commit -m "chore: update contributor wall"
          # Another workflow (update-index) may have pushed while this ran.
          git pull --rebase --autostash origin main
          git push

===== END FILE =====


===== FILE: ./.github/workflows/update-index.yml =====
name: Update marketplace index

on:
  push:
    branches: [main]
    paths:
      - 'skills/**'
      - '.github/workflows/update-index.yml'
  workflow_dispatch:

jobs:
  update-index:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Install dependencies
        run: pip install pyyaml

      # Generation lives in tools/generate-index.py so it is testable outside CI
      # and shares one PyYAML-backed frontmatter parser with the validators.
      # The previous inline regex parser silently truncated 604/817 descriptions
      # to their first line for every YAML scalar style except '>'/'|'.
      - name: Regenerate index.json
        run: python3 tools/generate-index.py

      - name: Sync skill count into README and marketplace
        run: |
          python3 << 'EOF'
          import os, re, json

          # Authoritative count: directories under skills/ that contain a SKILL.md
          # and are not .bak backups (matches index.json generation above).
          count = 0
          for name in os.listdir("skills"):
              if name.endswith(".bak"):
                  continue
              if os.path.isfile(os.path.join("skills", name, "SKILL.md")):
                  count += 1
          print(f"Authoritative skill count: {count}")

          # README.md — replace the skills badge and every "<NNN> ... skills" phrase.
          with open("README.md", encoding="utf-8") as f:
              readme = f.read()
          readme = re.sub(r"(badge/skills-)\d+", rf"\g<1>{count}", readme)
          # "817 production-grade", "817 structured", "817 skills", "all 817 skills",
          # "Scans 817 skill", "contains **817 skills**", BibTeX "{817 structured"
          readme = re.sub(r"\b\d+(?=\s+production-grade cybersecurity skills)", str(count), readme)
          readme = re.sub(r"\b\d+(?=\s+structured cybersecurity skills)", str(count), readme)
          readme = re.sub(r"(all\s+)\d+(?=\s+skills)", rf"\g<1>{count}", readme)
          readme = re.sub(r"(Scans\s+)\d+(?=\s+skill\b)", rf"\g<1>{count}", readme)
          readme = re.sub(r"(contains\s+\*\*)\d+(?=\s+skills\*\*)", rf"\g<1>{count}", readme)
          readme = re.sub(r"(\{)\d+(?=\s+structured cybersecurity skills)", rf"\g<1>{count}", readme)
          with open("README.md", "w", encoding="utf-8") as f:
              f.write(readme)

          # marketplace.json + plugin.json — patch "<NNN> cybersecurity skills" in descriptions.
          for path in (".claude-plugin/marketplace.json", ".claude-plugin/plugin.json"):
              with open(path, encoding="utf-8") as f:
                  data = f.read()
              data = re.sub(r"\b\d+(?=\s+cybersecurity skills)", str(count), data)
              json.loads(data)  # fail loudly if the regex broke JSON
              with open(path, "w", encoding="utf-8") as f:
                  f.write(data)

          print("Synced skill count into README.md, marketplace.json, plugin.json")
          EOF

      - name: Commit updated index and skill count
        run: |
          git config user.name "mukul975"
          git config user.email "mukuljangra5@gmail.com"
          git add index.json README.md .claude-plugin/marketplace.json .claude-plugin/plugin.json
          git diff --staged --quiet || git commit -m "chore: auto-update index.json and skill count"
          git push

===== END FILE =====


===== FILE: ./.github/workflows/validate-skills.yml =====
name: Validate SKILL.md files

on:
  push:
    paths:
      - 'skills/**'
      - 'tools/**'
      - '.github/workflows/validate-skills.yml'
  pull_request:
    paths:
      - 'skills/**'
      - 'tools/**'
      - '.github/workflows/validate-skills.yml'
  workflow_dispatch:

jobs:
  validate:
    runs-on: ubuntu-latest
    name: Validate SKILL.md frontmatter
    steps:
      - uses: actions/checkout@v4

      - name: Install dependencies
        run: pip install pyyaml

      # All frontmatter is parsed by tools/skill_frontmatter.py (PyYAML). Any
      # reintroduced regex parser silently truncates multi-line descriptions --
      # that bug shipped 604/817 broken descriptions before it was caught.
      - name: Guard against hand-rolled YAML parsers
        run: |
          if grep -rnE '(re\.(search|match|compile)\([^)]*description|^\s*description:.*\(\.\*\))' \
               tools/ --include='*.py' ; then
            echo "::error::Regex-based frontmatter parsing detected. Use tools/skill_frontmatter.py."
            exit 1
          fi
          echo "OK: no regex frontmatter parsers"

      # Single source of truth: tools/validate-skill.py validates required
      # frontmatter fields, kebab-case name, description length, subdomain, and
      # tag count. (Previously this step duplicated a weaker inline parser.)
      - name: Validate SKILL.md frontmatter
        run: python3 tools/validate-skill.py --all

      # agentskills.io conformance: name==directory, 1..1024 description,
      # reserved-word ban, angle-bracket injection check.
      - name: Validate agentskills.io conformance
        run: python3 tools/validate-agentskills.py --strict

      # index.json is generated; a PR that changes a description must regenerate it.
      # Pull requests only: on a push to main, update-index.yml regenerates
      # index.json in parallel with this job, so checking here would race and go
      # red on every merge that adds a skill before self-healing seconds later.
      - name: Check index.json is current
        if: github.event_name == 'pull_request'
        run: python3 tools/generate-index.py --check

      # Description quality gate. Pre-existing failures are grandfathered in
      # tools/lint-baseline.json so this blocks NEW debt only; the baseline is
      # allowed to shrink and never to grow.
      - name: Lint descriptions
        run: python3 tools/lint-descriptions.py --all --stats

      # Ratchet: the number of unreviewed near-duplicate description pairs may
      # never increase. Lower this cap as disambiguation lands.
      - name: Detect skill collisions
        run: python3 tools/detect-collisions.py --max-unreviewed 55

      - name: Check for duplicate skill names
        run: |
          python3 << 'EOF'
          import os
          import re
          from collections import Counter

          names = []
          for root, dirs, files in os.walk('skills'):
              for file in files:
                  if file == 'SKILL.md':
                      path = os.path.join(root, file)
                      with open(path, 'r', encoding='utf-8') as f:
                          content = f.read()
                      fm_match = re.match(r'^---\n(.*?)\n---', content, re.DOTALL)
                      if fm_match:
                          name_match = re.search(r'^name:\s*(.+)$', fm_match.group(1), re.MULTILINE)
                          if name_match:
                              names.append(name_match.group(1).strip().strip('"'))

          duplicates = [name for name, count in Counter(names).items() if count > 1]
          if duplicates:
              print(f"❌ Duplicate skill names found: {duplicates}")
              exit(1)
          print(f"✅ No duplicate names in {len(names)} skills")
          EOF

      - name: Report skill counts
        if: always()
        run: |
          echo "## Skill Database Stats" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          python3 << 'EOF'
          import os
          import re
          from collections import Counter

          subdomain_counts = Counter()
          total = 0
          for root, dirs, files in os.walk('skills'):
              for file in files:
                  if file == 'SKILL.md':
                      total += 1
                      path = os.path.join(root, file)
                      with open(path, 'r', encoding='utf-8') as f:
                          content = f.read()
                      fm_match = re.match(r'^---\n(.*?)\n---', content, re.DOTALL)
                      if fm_match:
                          sd_match = re.search(r'^subdomain:\s*(.+)$', fm_match.group(1), re.MULTILINE)
                          if sd_match:
                              subdomain_counts[sd_match.group(1).strip()] += 1

          print(f"**Total Skills: {total}**")
          print("")
          print("| Subdomain | Count |")
          print("|-----------|-------|")
          for sd, count in sorted(subdomain_counts.items(), key=lambda x: -x[1]):
              print(f"| {sd} | {count} |")
          EOF

===== END FILE =====


===== FILE: ./.gitignore =====
.claude/
teams/
node_modules/
__pycache__/
*.py[cod]
*.bin
*.pt
*.safetensors
*.gguf
*.onnx
.DS_Store
Thumbs.db
*.swp
launch/
extract_attack.py
AUDIT_REPORT.md

===== END FILE =====


===== FILE: ./AGENTS.md =====
# AGENTS.md

Instructions for AI agents working in this repository.

## What this repository is

A library of 817 cybersecurity skills. Each skill is a directory under `skills/` containing a `SKILL.md` — YAML frontmatter plus a Markdown procedure — following the [agentskills.io](https://agentskills.io) standard.

The layout is flat: `skills/<skill-name>/SKILL.md`. Do not nest skills by domain; agents discover them by scanning `skills/*/SKILL.md`.

## Reading a skill

Only `name` and `description` load at discovery time. The body loads once the description matches the request; `references/`, `scripts/` and `assets/` load only when referenced.

Read the description first. If it carries a negative trigger — "Do not use for X — use `other-skill`" — honour it. Those exist because two skills would otherwise compete for the same request.

## Changing a skill

Frontmatter is parsed by `tools/skill_frontmatter.py`, which uses PyYAML. Do not write a regex frontmatter parser; CI fails the build if it detects one. Three hand-rolled parsers previously truncated 604 of 817 descriptions to their first line.

After changing any `SKILL.md`:

```bash
pip install pyyaml
python tools/validate-skill.py --all
python tools/validate-agentskills.py --strict
python tools/generate-index.py          # regenerate index.json
python tools/lint-descriptions.py --all
python tools/detect-collisions.py
```

All five run in CI. `index.json` is generated — never edit it by hand.

## Writing a description

The description is the only signal another agent sees when deciding whether to load the skill. It needs four things:

1. What it does, concretely.
2. `Use when …` — the phrasings a user would actually type.
3. `Keywords:` — tool names, event IDs, CVEs, API calls.
4. `Do not use for X — use other-skill.` — the negative trigger.

Keep it under 1024 characters. Keep the body under 500 lines; depth belongs in `references/`.

## Constraints

- `name` must equal the directory name, lowercase-kebab, ≤64 characters.
- `domain` is always `cybersecurity`. `subdomain` must be one the validator accepts — see CONTRIBUTING.md.
- Scripts must run. No placeholders, no invented API endpoints, no fabricated CVE numbers.
- Framework IDs must be real and current. A wrong mapping sends an investigation the wrong way; omit rather than guess.

## Scope

See [SCOPE.md](SCOPE.md). This repository holds skills. Runtimes, engines and applications belong elsewhere.

===== END FILE =====


===== FILE: ./ATTACK_COVERAGE.md =====
# MITRE ATT&CK Coverage Map

<p align="center">
  <a href="https://attack.mitre.org/"><img src="https://img.shields.io/badge/MITRE_ATT%26CK-v16-red?style=for-the-badge&logo=shield&logoColor=white" alt="MITRE ATT&CK" /></a>
  <img src="https://img.shields.io/badge/Techniques-291+-blueviolet?style=for-the-badge" alt="Techniques" />
  <img src="https://img.shields.io/badge/Tactics-14%2F14-green?style=for-the-badge" alt="Tactics" />
</p>

This document maps all **291 unique MITRE ATT&CK techniques** (across **149 parent techniques**) referenced in our **817 cybersecurity skills** to the 14 Enterprise ATT&CK tactics. Use this to identify coverage gaps, plan detection engineering priorities, or validate your security program against the ATT&CK framework.

> **How to read this:** Each technique links to its official ATT&CK page. Skills listed under each technique are the ones in this repository that teach detection, hunting, exploitation, or response for that technique.

---

## Coverage Summary

| Tactic | Techniques | Coverage |
|:-------|:---------:|:---------|
| 🔎 **Reconnaissance** | **12** | `████████████░░░░░░░░░░░░░░░░░░` |
| 🏗️ **Resource Development** | **7** | `███████░░░░░░░░░░░░░░░░░░░░░░░` |
| 🚪 **Initial Access** | **18** | `██████████████████░░░░░░░░░░░░` |
| ⚡ **Execution** | **18** | `██████████████████░░░░░░░░░░░░` |
| 🔩 **Persistence** | **36** | `██████████████████████████████` |
| ⬆️ **Privilege Escalation** | **11** | `███████████░░░░░░░░░░░░░░░░░░░` |
| 🥷 **Defense Evasion** | **48** | `██████████████████████████████` |
| 🔑 **Credential Access** | **27** | `███████████████████████████░░░` |
| 🗺️ **Discovery** | **20** | `████████████████████░░░░░░░░░░` |
| ↔️ **Lateral Movement** | **9** | `█████████░░░░░░░░░░░░░░░░░░░░░` |
| 📦 **Collection** | **13** | `█████████████░░░░░░░░░░░░░░░░░` |
| 📡 **Command and Control** | **20** | `████████████████████░░░░░░░░░░` |
| 📤 **Exfiltration** | **12** | `████████████░░░░░░░░░░░░░░░░░░` |
| 💥 **Impact** | **6** | `██████░░░░░░░░░░░░░░░░░░░░░░░░` |
| 🔧 **Other/Cross-tactic** | **34** | |
| | **291** | **Total unique techniques** |

---

## 🔎 Reconnaissance

**12 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1589](https://attack.mitre.org/techniques/T1589/) | `conducting-full-scope-red-team-engagement`, `conducting-social-engineering-pretext-call`, `performing-open-source-intelligence-gathering` |
| [T1590](https://attack.mitre.org/techniques/T1590/) | `performing-open-source-intelligence-gathering` |
| [T1591](https://attack.mitre.org/techniques/T1591/) | `collecting-open-source-intelligence`, `conducting-social-engineering-pretext-call`, `performing-open-source-intelligence-gathering` |
| [T1592](https://attack.mitre.org/techniques/T1592/) | `performing-open-source-intelligence-gathering` |
| [T1593](https://attack.mitre.org/techniques/T1593/) | `conducting-full-scope-red-team-engagement`, `performing-open-source-intelligence-gathering` |
| [T1594](https://attack.mitre.org/techniques/T1594/) | `performing-open-source-intelligence-gathering` |
| [T1595](https://attack.mitre.org/techniques/T1595/) | `executing-red-team-engagement-planning`, `triaging-security-incident` |
| [T1595.001](https://attack.mitre.org/techniques/T1595/001/) | `performing-open-source-intelligence-gathering` |
| [T1595.002](https://attack.mitre.org/techniques/T1595/002/) | `performing-open-source-intelligence-gathering` |
| [T1596](https://attack.mitre.org/techniques/T1596/) | `performing-open-source-intelligence-gathering` |
| [T1598](https://attack.mitre.org/techniques/T1598/) | `conducting-social-engineering-pretext-call` |
| [T1598.003](https://attack.mitre.org/techniques/T1598/003/) | `conducting-social-engineering-pretext-call`, `conducting-spearphishing-simulation-campaign` |

---

## 🏗️ Resource Development

**7 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1583.001](https://attack.mitre.org/techniques/T1583/001/) | `building-red-team-c2-infrastructure-with-havoc`, `conducting-full-scope-red-team-engagement`, `conducting-spearphishing-simulation-campaign`, `implementing-mitre-attack-coverage-mapping` |
| [T1583.003](https://attack.mitre.org/techniques/T1583/003/) | `building-red-team-c2-infrastructure-with-havoc` |
| [T1584.001](https://attack.mitre.org/techniques/T1584/001/) | `hunting-for-dns-based-persistence` |
| [T1585.002](https://attack.mitre.org/techniques/T1585/002/) | `conducting-spearphishing-simulation-campaign` |
| [T1587.001](https://attack.mitre.org/techniques/T1587/001/) | `building-red-team-c2-infrastructure-with-havoc`, `conducting-full-scope-red-team-engagement` |
| [T1608.001](https://attack.mitre.org/techniques/T1608/001/) | `conducting-spearphishing-simulation-campaign` |
| [T1608.005](https://attack.mitre.org/techniques/T1608/005/) | `conducting-spearphishing-simulation-campaign` |

---

## 🚪 Initial Access

**18 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1078](https://attack.mitre.org/techniques/T1078/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-powershell-script-block-logging`, `analyzing-windows-event-logs-in-splunk`, `building-threat-hunt-hypothesis-framework`, `conducting-full-scope-red-team-engagement` +13 more |
| [T1078.001](https://attack.mitre.org/techniques/T1078/001/) | `detecting-service-account-abuse` |
| [T1078.002](https://attack.mitre.org/techniques/T1078/002/) | `conducting-domain-persistence-with-dcsync`, `detecting-service-account-abuse`, `exploiting-active-directory-certificate-services-esc1`, `exploiting-constrained-delegation-abuse`, `exploiting-nopac-cve-2021-42278-42287` +1 more |
| [T1078.003](https://attack.mitre.org/techniques/T1078/003/) | `performing-privilege-escalation-assessment` |
| [T1078.004](https://attack.mitre.org/techniques/T1078/004/) | `detecting-azure-lateral-movement`, `detecting-azure-service-principal-abuse`, `implementing-mitre-attack-coverage-mapping`, `implementing-threat-modeling-with-mitre-attack` |
| [T1091](https://attack.mitre.org/techniques/T1091/) | `executing-red-team-engagement-planning`, `performing-physical-intrusion-assessment` |
| [T1133](https://attack.mitre.org/techniques/T1133/) | `executing-red-team-engagement-planning`, `performing-threat-landscape-assessment-for-sector` |
| [T1190](https://attack.mitre.org/techniques/T1190/) | `conducting-full-scope-red-team-engagement`, `executing-red-team-engagement-planning`, `exploiting-ms17-010-eternalblue-vulnerability`, `hunting-for-webshell-activity`, `performing-threat-landscape-assessment-for-sector` +1 more |
| [T1195](https://attack.mitre.org/techniques/T1195/) | `analyzing-supply-chain-malware-artifacts`, `performing-threat-landscape-assessment-for-sector` |
| [T1195.001](https://attack.mitre.org/techniques/T1195/001/) | `hunting-for-supply-chain-compromise` |
| [T1195.002](https://attack.mitre.org/techniques/T1195/002/) | `hunting-for-supply-chain-compromise` |
| [T1199](https://attack.mitre.org/techniques/T1199/) | `hunting-for-supply-chain-compromise`, `performing-physical-intrusion-assessment` |
| [T1200](https://attack.mitre.org/techniques/T1200/) | `executing-red-team-engagement-planning`, `performing-physical-intrusion-assessment` |
| [T1566](https://attack.mitre.org/techniques/T1566/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-threat-actor-ttps-with-mitre-attack`, `analyzing-threat-landscape-with-misp`, `building-attack-pattern-library-from-cti-reports`, `hunting-advanced-persistent-threats` +3 more |
| [T1566.001](https://attack.mitre.org/techniques/T1566/001/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-campaign-attribution-evidence`, `analyzing-macro-malware-in-office-documents`, `analyzing-threat-actor-ttps-with-mitre-navigator`, `building-attack-pattern-library-from-cti-reports` +13 more |
| [T1566.002](https://attack.mitre.org/techniques/T1566/002/) | `building-attack-pattern-library-from-cti-reports`, `conducting-spearphishing-simulation-campaign`, `hunting-for-spearphishing-indicators`, `implementing-continuous-security-validation-with-bas`, `implementing-mitre-attack-coverage-mapping` +1 more |
| [T1566.003](https://attack.mitre.org/techniques/T1566/003/) | `conducting-spearphishing-simulation-campaign`, `hunting-for-spearphishing-indicators`, `implementing-continuous-security-validation-with-bas` |
| [T1566.004](https://attack.mitre.org/techniques/T1566/004/) | `conducting-social-engineering-pretext-call` |

---

## ⚡ Execution

**18 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1047](https://attack.mitre.org/techniques/T1047/) | `conducting-full-scope-red-team-engagement`, `detecting-fileless-attacks-on-endpoints`, `detecting-lateral-movement-with-splunk`, `detecting-living-off-the-land-attacks`, `detecting-living-off-the-land-with-lolbas` +8 more |
| [T1053](https://attack.mitre.org/techniques/T1053/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-persistence-mechanisms-in-linux`, `hunting-advanced-persistent-threats`, `hunting-for-persistence-mechanisms-in-windows`, `implementing-mitre-attack-coverage-mapping` +4 more |
| [T1053.002](https://attack.mitre.org/techniques/T1053/002/) | `hunting-for-scheduled-task-persistence` |
| [T1053.003](https://attack.mitre.org/techniques/T1053/003/) | `analyzing-persistence-mechanisms-in-linux`, `hunting-for-scheduled-task-persistence`, `performing-privilege-escalation-assessment`, `performing-privilege-escalation-on-linux` |
| [T1053.005](https://attack.mitre.org/techniques/T1053/005/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-campaign-attribution-evidence`, `analyzing-windows-event-logs-in-splunk`, `building-attack-pattern-library-from-cti-reports`, `building-detection-rule-with-splunk-spl` +17 more |
| [T1059](https://attack.mitre.org/techniques/T1059/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-threat-actor-ttps-with-mitre-attack`, `analyzing-windows-event-logs-in-splunk`, `building-incident-timeline-with-timesketch`, `deobfuscating-powershell-obfuscated-malware` +7 more |
| [T1059.001](https://attack.mitre.org/techniques/T1059/001/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-campaign-attribution-evidence`, `analyzing-macro-malware-in-office-documents`, `analyzing-powershell-empire-artifacts`, `analyzing-powershell-script-block-logging` +29 more |
| [T1059.003](https://attack.mitre.org/techniques/T1059/003/) | `building-attack-pattern-library-from-cti-reports`, `building-detection-rule-with-splunk-spl`, `detecting-suspicious-powershell-execution`, `mapping-mitre-attack-techniques`, `performing-purple-team-atomic-testing` |
| [T1059.004](https://attack.mitre.org/techniques/T1059/004/) | `performing-purple-team-atomic-testing` |
| [T1059.005](https://attack.mitre.org/techniques/T1059/005/) | `analyzing-macro-malware-in-office-documents`, `detecting-living-off-the-land-attacks`, `executing-red-team-exercise`, `hunting-for-living-off-the-land-binaries`, `hunting-for-lolbins-execution-in-endpoint-logs` +2 more |
| [T1059.006](https://attack.mitre.org/techniques/T1059/006/) | `performing-purple-team-atomic-testing` |
| [T1059.007](https://attack.mitre.org/techniques/T1059/007/) | `performing-purple-team-atomic-testing` |
| [T1129](https://attack.mitre.org/techniques/T1129/) | `performing-purple-team-atomic-testing` |
| [T1203](https://attack.mitre.org/techniques/T1203/) | `performing-purple-team-atomic-testing` |
| [T1204.001](https://attack.mitre.org/techniques/T1204/001/) | `conducting-spearphishing-simulation-campaign` |
| [T1204.002](https://attack.mitre.org/techniques/T1204/002/) | `analyzing-macro-malware-in-office-documents`, `conducting-full-scope-red-team-engagement`, `conducting-spearphishing-simulation-campaign`, `detecting-living-off-the-land-attacks`, `executing-red-team-engagement-planning` +4 more |
| [T1569](https://attack.mitre.org/techniques/T1569/) | `performing-purple-team-atomic-testing` |
| [T1569.002](https://attack.mitre.org/techniques/T1569/002/) | `detecting-lateral-movement-in-network`, `detecting-lateral-movement-with-splunk`, `exploiting-ms17-010-eternalblue-vulnerability`, `performing-purple-team-atomic-testing` |

---

## 🔩 Persistence

**36 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1098](https://attack.mitre.org/techniques/T1098/) | `analyzing-windows-event-logs-in-splunk`, `conducting-domain-persistence-with-dcsync`, `hunting-for-t1098-account-manipulation`, `implementing-mitre-attack-coverage-mapping`, `implementing-siem-use-cases-for-detection` +1 more |
| [T1098.001](https://attack.mitre.org/techniques/T1098/001/) | `conducting-cloud-penetration-testing`, `detecting-azure-lateral-movement`, `detecting-azure-service-principal-abuse`, `hunting-for-t1098-account-manipulation`, `implementing-mitre-attack-coverage-mapping` |
| [T1098.002](https://attack.mitre.org/techniques/T1098/002/) | `detecting-azure-lateral-movement`, `detecting-email-forwarding-rules-attack` |
| [T1098.004](https://attack.mitre.org/techniques/T1098/004/) | `analyzing-persistence-mechanisms-in-linux`, `implementing-security-monitoring-with-datadog` |
| [T1136](https://attack.mitre.org/techniques/T1136/) | `detecting-privilege-escalation-in-kubernetes-pods`, `implementing-mitre-attack-coverage-mapping`, `performing-purple-team-atomic-testing` |
| [T1136.001](https://attack.mitre.org/techniques/T1136/001/) | `analyzing-windows-event-logs-in-splunk`, `performing-purple-team-atomic-testing` |
| [T1136.002](https://attack.mitre.org/techniques/T1136/002/) | `exploiting-nopac-cve-2021-42278-42287` |
| [T1197](https://attack.mitre.org/techniques/T1197/) | `detecting-living-off-the-land-attacks`, `detecting-living-off-the-land-with-lolbas`, `hunting-for-living-off-the-land-binaries`, `hunting-for-lolbins-execution-in-endpoint-logs`, `performing-purple-team-atomic-testing` |
| [T1505](https://attack.mitre.org/techniques/T1505/) | `performing-purple-team-atomic-testing` |
| [T1505.003](https://attack.mitre.org/techniques/T1505/003/) | `building-attack-pattern-library-from-cti-reports`, `hunting-for-webshell-activity`, `performing-purple-team-atomic-testing` |
| [T1542.001](https://attack.mitre.org/techniques/T1542/001/) | `analyzing-uefi-bootkit-persistence` |
| [T1542.003](https://attack.mitre.org/techniques/T1542/003/) | `analyzing-uefi-bootkit-persistence` |
| [T1543](https://attack.mitre.org/techniques/T1543/) | `analyzing-persistence-mechanisms-in-linux`, `hunting-for-persistence-mechanisms-in-windows`, `performing-purple-team-atomic-testing` |
| [T1543.002](https://attack.mitre.org/techniques/T1543/002/) | `analyzing-persistence-mechanisms-in-linux`, `performing-privilege-escalation-on-linux` |
| [T1543.003](https://attack.mitre.org/techniques/T1543/003/) | `detecting-lateral-movement-with-splunk`, `detecting-living-off-the-land-attacks`, `detecting-privilege-escalation-attempts`, `hunting-for-persistence-mechanisms-in-windows`, `hunting-for-unusual-service-installations` +2 more |
| [T1546](https://attack.mitre.org/techniques/T1546/) | `analyzing-persistence-mechanisms-in-linux`, `performing-purple-team-atomic-testing` |
| [T1546.001](https://attack.mitre.org/techniques/T1546/001/) | `performing-purple-team-atomic-testing` |
| [T1546.003](https://attack.mitre.org/techniques/T1546/003/) | `analyzing-windows-event-logs-in-splunk`, `detecting-fileless-attacks-on-endpoints`, `detecting-fileless-malware-techniques`, `detecting-wmi-persistence`, `hunting-for-lateral-movement-via-wmi` +3 more |
| [T1546.004](https://attack.mitre.org/techniques/T1546/004/) | `analyzing-persistence-mechanisms-in-linux` |
| [T1546.010](https://attack.mitre.org/techniques/T1546/010/) | `hunting-for-persistence-mechanisms-in-windows` |
| [T1546.012](https://attack.mitre.org/techniques/T1546/012/) | `hunting-for-persistence-mechanisms-in-windows`, `hunting-for-registry-persistence-mechanisms` |
| [T1546.015](https://attack.mitre.org/techniques/T1546/015/) | `hunting-for-persistence-mechanisms-in-windows`, `hunting-for-registry-persistence-mechanisms` |
| [T1547](https://attack.mitre.org/techniques/T1547/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-malware-persistence-with-autoruns`, `hunting-advanced-persistent-threats`, `hunting-for-persistence-mechanisms-in-windows`, `implementing-siem-use-cases-for-detection` +3 more |
| [T1547.001](https://attack.mitre.org/techniques/T1547/001/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-windows-event-logs-in-splunk`, `building-attack-pattern-library-from-cti-reports`, `conducting-full-scope-red-team-engagement`, `detecting-fileless-attacks-on-endpoints` +10 more |
| [T1547.004](https://attack.mitre.org/techniques/T1547/004/) | `hunting-for-persistence-mechanisms-in-windows`, `hunting-for-registry-persistence-mechanisms`, `performing-purple-team-atomic-testing` |
| [T1547.005](https://attack.mitre.org/techniques/T1547/005/) | `hunting-for-persistence-mechanisms-in-windows` |
| [T1547.009](https://attack.mitre.org/techniques/T1547/009/) | `performing-purple-team-atomic-testing` |
| [T1556](https://attack.mitre.org/techniques/T1556/) | `performing-initial-access-with-evilginx3` |
| [T1556.007](https://attack.mitre.org/techniques/T1556/007/) | `detecting-azure-lateral-movement` |
| [T1574](https://attack.mitre.org/techniques/T1574/) | `analyzing-persistence-mechanisms-in-linux`, `performing-purple-team-atomic-testing` |
| [T1574.001](https://attack.mitre.org/techniques/T1574/001/) | `detecting-dll-sideloading-attacks`, `hunting-for-persistence-mechanisms-in-windows`, `performing-purple-team-atomic-testing` |
| [T1574.002](https://attack.mitre.org/techniques/T1574/002/) | `analyzing-windows-event-logs-in-splunk`, `building-attack-pattern-library-from-cti-reports`, `detecting-dll-sideloading-attacks`, `implementing-siem-use-cases-for-detection`, `performing-purple-team-atomic-testing` |
| [T1574.006](https://attack.mitre.org/techniques/T1574/006/) | `analyzing-persistence-mechanisms-in-linux`, `detecting-dll-sideloading-attacks`, `performing-privilege-escalation-on-linux` |
| [T1574.008](https://attack.mitre.org/techniques/T1574/008/) | `detecting-dll-sideloading-attacks` |
| [T1574.009](https://attack.mitre.org/techniques/T1574/009/) | `detecting-privilege-escalation-attempts` |
| [T1574.011](https://attack.mitre.org/techniques/T1574/011/) | `detecting-privilege-escalation-attempts` |

---

## ⬆️ Privilege Escalation

**11 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1068](https://attack.mitre.org/techniques/T1068/) | `conducting-full-scope-red-team-engagement`, `detecting-container-escape-attempts`, `detecting-privilege-escalation-attempts`, `detecting-privilege-escalation-in-kubernetes-pods`, `executing-red-team-engagement-planning` +5 more |
| [T1134](https://attack.mitre.org/techniques/T1134/) | `analyzing-windows-event-logs-in-splunk`, `detecting-privilege-escalation-attempts` |
| [T1134.001](https://attack.mitre.org/techniques/T1134/001/) | `detecting-privilege-escalation-attempts`, `exploiting-constrained-delegation-abuse`, `performing-purple-team-atomic-testing` |
| [T1134.005](https://attack.mitre.org/techniques/T1134/005/) | `hunting-for-t1098-account-manipulation`, `performing-active-directory-compromise-investigation` |
| [T1484](https://attack.mitre.org/techniques/T1484/) | `exploiting-active-directory-certificate-services-esc1`, `performing-active-directory-vulnerability-assessment` |
| [T1484.001](https://attack.mitre.org/techniques/T1484/001/) | `deploying-active-directory-honeytokens`, `performing-active-directory-compromise-investigation` |
| [T1548](https://attack.mitre.org/techniques/T1548/) | `detecting-container-escape-attempts`, `detecting-privilege-escalation-in-kubernetes-pods`, `detecting-t1548-abuse-elevation-control-mechanism`, `performing-privilege-escalation-assessment` |
| [T1548.001](https://attack.mitre.org/techniques/T1548/001/) | `detecting-privilege-escalation-attempts`, `detecting-privilege-escalation-in-kubernetes-pods`, `detecting-t1548-abuse-elevation-control-mechanism`, `performing-privilege-escalation-assessment`, `performing-privilege-escalation-on-linux` |
| [T1548.002](https://attack.mitre.org/techniques/T1548/002/) | `conducting-full-scope-red-team-engagement`, `detecting-privilege-escalation-attempts`, `detecting-t1548-abuse-elevation-control-mechanism`, `performing-purple-team-atomic-testing` |
| [T1548.003](https://attack.mitre.org/techniques/T1548/003/) | `detecting-privilege-escalation-attempts`, `detecting-t1548-abuse-elevation-control-mechanism`, `performing-privilege-escalation-assessment`, `performing-privilege-escalation-on-linux` |
| [T1548.004](https://attack.mitre.org/techniques/T1548/004/) | `detecting-t1548-abuse-elevation-control-mechanism` |

---

## 🥷 Defense Evasion

**48 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1027](https://attack.mitre.org/techniques/T1027/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-powershell-empire-artifacts`, `analyzing-powershell-script-block-logging`, `building-attack-pattern-library-from-cti-reports`, `conducting-full-scope-red-team-engagement` +3 more |
| [T1036](https://attack.mitre.org/techniques/T1036/) | `detecting-evasion-techniques-in-endpoint-logs`, `implementing-mitre-attack-coverage-mapping`, `implementing-siem-use-cases-for-detection`, `performing-purple-team-atomic-testing` |
| [T1036.005](https://attack.mitre.org/techniques/T1036/005/) | `detecting-process-injection-techniques`, `performing-purple-team-atomic-testing` |
| [T1055](https://attack.mitre.org/techniques/T1055/) | `building-attack-pattern-library-from-cti-reports`, `building-red-team-c2-infrastructure-with-havoc`, `conducting-full-scope-red-team-engagement`, `detecting-evasion-techniques-in-endpoint-logs`, `detecting-fileless-attacks-on-endpoints` +13 more |
| [T1055.001](https://attack.mitre.org/techniques/T1055/001/) | `detecting-process-hollowing-technique`, `detecting-process-injection-techniques`, `detecting-t1055-process-injection-with-sysmon`, `hunting-for-process-injection-techniques`, `performing-purple-team-atomic-testing` +1 more |
| [T1055.002](https://attack.mitre.org/techniques/T1055/002/) | `detecting-process-injection-techniques`, `detecting-t1055-process-injection-with-sysmon` |
| [T1055.003](https://attack.mitre.org/techniques/T1055/003/) | `detecting-process-hollowing-technique`, `detecting-process-injection-techniques`, `detecting-t1055-process-injection-with-sysmon`, `performing-purple-team-atomic-testing` |
| [T1055.004](https://attack.mitre.org/techniques/T1055/004/) | `detecting-process-hollowing-technique`, `detecting-process-injection-techniques`, `detecting-t1055-process-injection-with-sysmon`, `hunting-for-process-injection-techniques` |
| [T1055.005](https://attack.mitre.org/techniques/T1055/005/) | `detecting-process-injection-techniques`, `detecting-t1055-process-injection-with-sysmon` |
| [T1055.008](https://attack.mitre.org/techniques/T1055/008/) | `detecting-process-injection-techniques` |
| [T1055.009](https://attack.mitre.org/techniques/T1055/009/) | `detecting-process-injection-techniques` |
| [T1055.011](https://attack.mitre.org/techniques/T1055/011/) | `detecting-process-injection-techniques` |
| [T1055.012](https://attack.mitre.org/techniques/T1055/012/) | `conducting-malware-incident-response`, `detecting-fileless-malware-techniques`, `detecting-process-hollowing-technique`, `detecting-process-injection-techniques`, `detecting-t1055-process-injection-with-sysmon` +2 more |
| [T1055.013](https://attack.mitre.org/techniques/T1055/013/) | `detecting-process-hollowing-technique`, `detecting-process-injection-techniques`, `detecting-t1055-process-injection-with-sysmon` |
| [T1055.014](https://attack.mitre.org/techniques/T1055/014/) | `detecting-process-injection-techniques` |
| [T1055.015](https://attack.mitre.org/techniques/T1055/015/) | `detecting-process-injection-techniques`, `detecting-t1055-process-injection-with-sysmon` |
| [T1070](https://attack.mitre.org/techniques/T1070/) | `detecting-evasion-techniques-in-endpoint-logs`, `implementing-siem-use-cases-for-detection`, `implementing-velociraptor-for-ir-collection`, `performing-purple-team-atomic-testing` |
| [T1070.001](https://attack.mitre.org/techniques/T1070/001/) | `detecting-evasion-techniques-in-endpoint-logs`, `implementing-mitre-attack-coverage-mapping`, `performing-purple-team-atomic-testing`, `performing-purple-team-exercise` |
| [T1070.004](https://attack.mitre.org/techniques/T1070/004/) | `implementing-threat-modeling-with-mitre-attack`, `performing-purple-team-atomic-testing` |
| [T1070.006](https://attack.mitre.org/techniques/T1070/006/) | `detecting-evasion-techniques-in-endpoint-logs`, `hunting-for-defense-evasion-via-timestomping` |
| [T1112](https://attack.mitre.org/techniques/T1112/) | `detecting-fileless-malware-techniques`, `performing-purple-team-atomic-testing` |
| [T1127](https://attack.mitre.org/techniques/T1127/) | `detecting-evasion-techniques-in-endpoint-logs`, `detecting-living-off-the-land-with-lolbas`, `hunting-for-lolbins-execution-in-endpoint-logs` |
| [T1127.001](https://attack.mitre.org/techniques/T1127/001/) | `detecting-living-off-the-land-attacks`, `detecting-living-off-the-land-with-lolbas`, `hunting-for-lolbins-execution-in-endpoint-logs` |
| [T1140](https://attack.mitre.org/techniques/T1140/) | `analyzing-powershell-script-block-logging`, `detecting-fileless-attacks-on-endpoints`, `detecting-living-off-the-land-with-lolbas`, `hunting-for-living-off-the-land-binaries`, `hunting-for-lolbins-execution-in-endpoint-logs` +1 more |
| [T1202](https://attack.mitre.org/techniques/T1202/) | `hunting-for-living-off-the-land-binaries`, `hunting-for-lolbins-execution-in-endpoint-logs` |
| [T1218](https://attack.mitre.org/techniques/T1218/) | `detecting-evasion-techniques-in-endpoint-logs`, `detecting-living-off-the-land-attacks`, `detecting-living-off-the-land-with-lolbas`, `hunting-advanced-persistent-threats`, `hunting-for-living-off-the-land-binaries` +3 more |
| [T1218.001](https://attack.mitre.org/techniques/T1218/001/) | `hunting-for-living-off-the-land-binaries`, `hunting-for-lolbins-execution-in-endpoint-logs`, `performing-purple-team-atomic-testing` |
| [T1218.002](https://attack.mitre.org/techniques/T1218/002/) | `hunting-for-living-off-the-land-binaries` |
| [T1218.003](https://attack.mitre.org/techniques/T1218/003/) | `detecting-living-off-the-land-attacks`, `detecting-living-off-the-land-with-lolbas`, `hunting-for-living-off-the-land-binaries`, `hunting-for-lolbins-execution-in-endpoint-logs`, `performing-purple-team-atomic-testing` |
| [T1218.004](https://attack.mitre.org/techniques/T1218/004/) | `detecting-living-off-the-land-attacks`, `hunting-for-lolbins-execution-in-endpoint-logs` |
| [T1218.005](https://attack.mitre.org/techniques/T1218/005/) | `detecting-fileless-malware-techniques`, `detecting-living-off-the-land-attacks`, `detecting-living-off-the-land-with-lolbas`, `hunting-for-living-off-the-land-binaries`, `hunting-for-lolbins-execution-in-endpoint-logs` +1 more |
| [T1218.007](https://attack.mitre.org/techniques/T1218/007/) | `hunting-for-living-off-the-land-binaries`, `hunting-for-lolbins-execution-in-endpoint-logs` |
| [T1218.010](https://attack.mitre.org/techniques/T1218/010/) | `detecting-living-off-the-land-attacks`, `detecting-living-off-the-land-with-lolbas`, `hunting-for-living-off-the-land-binaries`, `hunting-for-lolbins-execution-in-endpoint-logs`, `performing-purple-team-atomic-testing` |
| [T1218.011](https://attack.mitre.org/techniques/T1218/011/) | `detecting-living-off-the-land-attacks`, `detecting-living-off-the-land-with-lolbas`, `hunting-for-living-off-the-land-binaries`, `hunting-for-lolbins-execution-in-endpoint-logs`, `performing-dynamic-analysis-with-any-run` +1 more |
| [T1218.013](https://attack.mitre.org/techniques/T1218/013/) | `detecting-living-off-the-land-attacks` |
| [T1222.001](https://attack.mitre.org/techniques/T1222/001/) | `conducting-domain-persistence-with-dcsync` |
| [T1497](https://attack.mitre.org/techniques/T1497/) | `analyzing-malware-sandbox-evasion-techniques` |
| [T1497.001](https://attack.mitre.org/techniques/T1497/001/) | `analyzing-malware-sandbox-evasion-techniques` |
| [T1497.002](https://attack.mitre.org/techniques/T1497/002/) | `analyzing-malware-sandbox-evasion-techniques` |
| [T1497.003](https://attack.mitre.org/techniques/T1497/003/) | `analyzing-malware-sandbox-evasion-techniques` |
| [T1550](https://attack.mitre.org/techniques/T1550/) | `performing-lateral-movement-detection` |
| [T1550.001](https://attack.mitre.org/techniques/T1550/001/) | `detecting-azure-lateral-movement` |
| [T1550.002](https://attack.mitre.org/techniques/T1550/002/) | `analyzing-windows-event-logs-in-splunk`, `building-attack-pattern-library-from-cti-reports`, `conducting-full-scope-red-team-engagement`, `detecting-lateral-movement-in-network`, `detecting-lateral-movement-with-splunk` +6 more |
| [T1550.003](https://attack.mitre.org/techniques/T1550/003/) | `conducting-pass-the-ticket-attack`, `detecting-pass-the-hash-attacks`, `detecting-pass-the-ticket-attacks`, `exploiting-constrained-delegation-abuse` |
| [T1550.004](https://attack.mitre.org/techniques/T1550/004/) | `performing-initial-access-with-evilginx3` |
| [T1562](https://attack.mitre.org/techniques/T1562/) | `detecting-evasion-techniques-in-endpoint-logs`, `performing-purple-team-atomic-testing` |
| [T1562.001](https://attack.mitre.org/techniques/T1562/001/) | `analyzing-powershell-script-block-logging`, `building-attack-pattern-library-from-cti-reports`, `detecting-evasion-techniques-in-endpoint-logs`, `detecting-fileless-attacks-on-endpoints`, `detecting-suspicious-powershell-execution` +1 more |
| [T1610](https://attack.mitre.org/techniques/T1610/) | `detecting-container-escape-attempts`, `detecting-container-escape-with-falco-rules` |

---

## 🔑 Credential Access

**27 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1003](https://attack.mitre.org/techniques/T1003/) | `analyzing-powershell-script-block-logging`, `building-attack-pattern-library-from-cti-reports`, `building-detection-rules-with-sigma`, `detecting-container-escape-with-falco-rules`, `detecting-credential-dumping-techniques` +10 more |
| [T1003.001](https://attack.mitre.org/techniques/T1003/001/) | `analyzing-campaign-attribution-evidence`, `analyzing-powershell-script-block-logging`, `analyzing-windows-event-logs-in-splunk`, `building-attack-pattern-library-from-cti-reports`, `building-detection-rule-with-splunk-spl` +13 more |
| [T1003.002](https://attack.mitre.org/techniques/T1003/002/) | `detecting-credential-dumping-techniques`, `detecting-t1003-credential-dumping-with-edr`, `performing-purple-team-atomic-testing` |
| [T1003.003](https://attack.mitre.org/techniques/T1003/003/) | `detecting-credential-dumping-techniques`, `detecting-t1003-credential-dumping-with-edr`, `performing-purple-team-atomic-testing` |
| [T1003.004](https://attack.mitre.org/techniques/T1003/004/) | `detecting-t1003-credential-dumping-with-edr`, `performing-credential-access-with-lazagne`, `performing-purple-team-atomic-testing` |
| [T1003.005](https://attack.mitre.org/techniques/T1003/005/) | `detecting-t1003-credential-dumping-with-edr`, `performing-purple-team-atomic-testing` |
| [T1003.006](https://attack.mitre.org/techniques/T1003/006/) | `analyzing-windows-event-logs-in-splunk`, `conducting-domain-persistence-with-dcsync`, `conducting-full-scope-red-team-engagement`, `conducting-internal-network-penetration-test`, `detecting-dcsync-attack-in-active-directory` +8 more |
| [T1110](https://attack.mitre.org/techniques/T1110/) | `analyzing-windows-event-logs-in-splunk`, `building-detection-rule-with-splunk-spl`, `conducting-internal-network-penetration-test`, `implementing-mitre-attack-coverage-mapping`, `implementing-siem-use-cases-for-detection` +3 more |
| [T1110.001](https://attack.mitre.org/techniques/T1110/001/) | `analyzing-windows-event-logs-in-splunk`, `building-detection-rule-with-splunk-spl`, `implementing-siem-use-cases-for-detection`, `performing-false-positive-reduction-in-siem`, `performing-purple-team-atomic-testing` |
| [T1110.002](https://attack.mitre.org/techniques/T1110/002/) | `exploiting-kerberoasting-with-impacket` |
| [T1110.003](https://attack.mitre.org/techniques/T1110/003/) | `detecting-pass-the-ticket-attacks`, `implementing-siem-use-cases-for-detection`, `performing-purple-team-atomic-testing` |
| [T1187](https://attack.mitre.org/techniques/T1187/) | `detecting-ntlm-relay-with-event-correlation` |
| [T1528](https://attack.mitre.org/techniques/T1528/) | `detecting-azure-lateral-movement`, `detecting-azure-service-principal-abuse` |
| [T1539](https://attack.mitre.org/techniques/T1539/) | `performing-credential-access-with-lazagne`, `performing-initial-access-with-evilginx3` |
| [T1552](https://attack.mitre.org/techniques/T1552/) | `performing-cloud-incident-containment-procedures`, `performing-purple-team-atomic-testing` |
| [T1552.001](https://attack.mitre.org/techniques/T1552/001/) | `performing-credential-access-with-lazagne`, `performing-purple-team-atomic-testing` |
| [T1552.002](https://attack.mitre.org/techniques/T1552/002/) | `performing-credential-access-with-lazagne` |
| [T1552.005](https://attack.mitre.org/techniques/T1552/005/) | `conducting-cloud-penetration-testing` |
| [T1552.006](https://attack.mitre.org/techniques/T1552/006/) | `deploying-active-directory-honeytokens` |
| [T1557](https://attack.mitre.org/techniques/T1557/) | `performing-initial-access-with-evilginx3` |
| [T1557.001](https://attack.mitre.org/techniques/T1557/001/) | `conducting-internal-network-penetration-test`, `detecting-ntlm-relay-with-event-correlation`, `hunting-for-ntlm-relay-attacks` |
| [T1558](https://attack.mitre.org/techniques/T1558/) | `analyzing-windows-event-logs-in-splunk`, `conducting-pass-the-ticket-attack`, `exploiting-kerberoasting-with-impacket`, `exploiting-nopac-cve-2021-42278-42287`, `performing-lateral-movement-detection` +1 more |
| [T1558.001](https://attack.mitre.org/techniques/T1558/001/) | `analyzing-windows-event-logs-in-splunk`, `conducting-domain-persistence-with-dcsync`, `detecting-golden-ticket-attacks-in-kerberos-logs`, `detecting-golden-ticket-forgery`, `detecting-kerberoasting-attacks` +3 more |
| [T1558.002](https://attack.mitre.org/techniques/T1558/002/) | `performing-active-directory-compromise-investigation` |
| [T1558.003](https://attack.mitre.org/techniques/T1558/003/) | `analyzing-windows-event-logs-in-splunk`, `building-attack-pattern-library-from-cti-reports`, `conducting-full-scope-red-team-engagement`, `conducting-internal-network-penetration-test`, `deploying-active-directory-honeytokens` +12 more |
| [T1558.004](https://attack.mitre.org/techniques/T1558/004/) | `detecting-kerberoasting-attacks` |
| [T1649](https://attack.mitre.org/techniques/T1649/) | `exploiting-active-directory-certificate-services-esc1` |

---

## 🗺️ Discovery

**20 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1016](https://attack.mitre.org/techniques/T1016/) | `conducting-full-scope-red-team-engagement`, `conducting-internal-reconnaissance-with-bloodhound-ce`, `exploiting-active-directory-with-bloodhound`, `performing-purple-team-atomic-testing` |
| [T1018](https://attack.mitre.org/techniques/T1018/) | `conducting-full-scope-red-team-engagement`, `conducting-internal-reconnaissance-with-bloodhound-ce`, `detecting-network-scanning-with-ids-signatures`, `exploiting-active-directory-with-bloodhound`, `performing-active-directory-bloodhound-analysis` |
| [T1033](https://attack.mitre.org/techniques/T1033/) | `conducting-internal-reconnaissance-with-bloodhound-ce`, `detecting-privilege-escalation-attempts`, `exploiting-active-directory-with-bloodhound`, `performing-purple-team-atomic-testing` |
| [T1040](https://attack.mitre.org/techniques/T1040/) | `implementing-continuous-security-validation-with-bas` |
| [T1046](https://attack.mitre.org/techniques/T1046/) | `detecting-network-scanning-with-ids-signatures`, `detecting-privilege-escalation-attempts`, `performing-packet-injection-attack`, `triaging-security-incident` |
| [T1049](https://attack.mitre.org/techniques/T1049/) | `performing-purple-team-atomic-testing` |
| [T1057](https://attack.mitre.org/techniques/T1057/) | `performing-purple-team-atomic-testing` |
| [T1069](https://attack.mitre.org/techniques/T1069/) | `performing-purple-team-atomic-testing` |
| [T1069.001](https://attack.mitre.org/techniques/T1069/001/) | `performing-active-directory-bloodhound-analysis`, `performing-purple-team-atomic-testing` |
| [T1069.002](https://attack.mitre.org/techniques/T1069/002/) | `conducting-internal-reconnaissance-with-bloodhound-ce`, `exploiting-active-directory-with-bloodhound`, `performing-active-directory-bloodhound-analysis`, `performing-kerberoasting-attack`, `performing-purple-team-atomic-testing` |
| [T1082](https://attack.mitre.org/techniques/T1082/) | `conducting-full-scope-red-team-engagement`, `performing-purple-team-atomic-testing` |
| [T1083](https://attack.mitre.org/techniques/T1083/) | `implementing-canary-tokens-for-network-intrusion`, `performing-purple-team-atomic-testing` |
| [T1087](https://attack.mitre.org/techniques/T1087/) | `conducting-full-scope-red-team-engagement`, `executing-red-team-engagement-planning`, `implementing-continuous-security-validation-with-bas`, `performing-purple-team-atomic-testing` |
| [T1087.001](https://attack.mitre.org/techniques/T1087/001/) | `performing-purple-team-atomic-testing` |
| [T1087.002](https://attack.mitre.org/techniques/T1087/002/) | `conducting-internal-reconnaissance-with-bloodhound-ce`, `deploying-active-directory-honeytokens`, `exploiting-active-directory-certificate-services-esc1`, `exploiting-active-directory-with-bloodhound`, `exploiting-kerberoasting-with-impacket` +3 more |
| [T1087.004](https://attack.mitre.org/techniques/T1087/004/) | `detecting-azure-service-principal-abuse`, `implementing-mitre-attack-coverage-mapping` |
| [T1482](https://attack.mitre.org/techniques/T1482/) | `conducting-internal-reconnaissance-with-bloodhound-ce`, `exploiting-active-directory-with-bloodhound`, `performing-active-directory-bloodhound-analysis` |
| [T1518](https://attack.mitre.org/techniques/T1518/) | `performing-purple-team-atomic-testing` |
| [T1518.001](https://attack.mitre.org/techniques/T1518/001/) | `performing-purple-team-atomic-testing` |
| [T1580](https://attack.mitre.org/techniques/T1580/) | `implementing-mitre-attack-coverage-mapping` |

---

## ↔️ Lateral Movement

**9 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1021](https://attack.mitre.org/techniques/T1021/) | `detecting-lateral-movement-in-network`, `detecting-lateral-movement-with-splunk`, `detecting-service-account-abuse`, `executing-red-team-engagement-planning`, `exploiting-constrained-delegation-abuse` +10 more |
| [T1021.001](https://attack.mitre.org/techniques/T1021/001/) | `analyzing-campaign-attribution-evidence`, `analyzing-windows-event-logs-in-splunk`, `building-attack-pattern-library-from-cti-reports`, `building-detection-rule-with-splunk-spl`, `building-threat-hunt-hypothesis-framework` +8 more |
| [T1021.002](https://attack.mitre.org/techniques/T1021/002/) | `analyzing-windows-event-logs-in-splunk`, `building-attack-pattern-library-from-cti-reports`, `building-detection-rule-with-splunk-spl`, `conducting-full-scope-red-team-engagement`, `conducting-internal-network-penetration-test` +10 more |
| [T1021.003](https://attack.mitre.org/techniques/T1021/003/) | `detecting-lateral-movement-with-splunk`, `hunting-for-dcom-lateral-movement`, `performing-lateral-movement-detection`, `performing-lateral-movement-with-wmiexec`, `performing-purple-team-atomic-testing` |
| [T1021.004](https://attack.mitre.org/techniques/T1021/004/) | `detecting-lateral-movement-with-splunk`, `performing-purple-team-atomic-testing` |
| [T1021.006](https://attack.mitre.org/techniques/T1021/006/) | `building-attack-pattern-library-from-cti-reports`, `detecting-lateral-movement-with-splunk`, `performing-lateral-movement-detection`, `performing-purple-team-atomic-testing` |
| [T1210](https://attack.mitre.org/techniques/T1210/) | `exploiting-ms17-010-eternalblue-vulnerability`, `exploiting-zerologon-vulnerability-cve-2020-1472` |
| [T1534](https://attack.mitre.org/techniques/T1534/) | `implementing-mitre-attack-coverage-mapping` |
| [T1570](https://attack.mitre.org/techniques/T1570/) | `detecting-lateral-movement-in-network`, `detecting-lateral-movement-with-splunk`, `performing-lateral-movement-with-wmiexec`, `performing-purple-team-atomic-testing` |

---

## 📦 Collection

**13 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1005](https://attack.mitre.org/techniques/T1005/) | `conducting-malware-incident-response`, `detecting-container-escape-with-falco-rules`, `performing-purple-team-atomic-testing` |
| [T1039](https://attack.mitre.org/techniques/T1039/) | `performing-purple-team-atomic-testing` |
| [T1074](https://attack.mitre.org/techniques/T1074/) | `building-attack-pattern-library-from-cti-reports`, `executing-red-team-exercise`, `hunting-for-data-staging-before-exfiltration` |
| [T1074.001](https://attack.mitre.org/techniques/T1074/001/) | `hunting-for-data-staging-before-exfiltration`, `performing-purple-team-atomic-testing` |
| [T1074.002](https://attack.mitre.org/techniques/T1074/002/) | `hunting-for-data-staging-before-exfiltration` |
| [T1113](https://attack.mitre.org/techniques/T1113/) | `performing-purple-team-atomic-testing` |
| [T1114.002](https://attack.mitre.org/techniques/T1114/002/) | `detecting-email-forwarding-rules-attack` |
| [T1114.003](https://attack.mitre.org/techniques/T1114/003/) | `detecting-business-email-compromise`, `detecting-email-forwarding-rules-attack` |
| [T1115](https://attack.mitre.org/techniques/T1115/) | `performing-purple-team-atomic-testing` |
| [T1213](https://attack.mitre.org/techniques/T1213/) | `conducting-full-scope-red-team-engagement` |
| [T1530](https://attack.mitre.org/techniques/T1530/) | `detecting-insider-threat-behaviors`, `implementing-mitre-attack-coverage-mapping`, `performing-cloud-incident-containment-procedures` |
| [T1560](https://attack.mitre.org/techniques/T1560/) | `conducting-full-scope-red-team-engagement`, `hunting-for-data-staging-before-exfiltration` |
| [T1560.001](https://attack.mitre.org/techniques/T1560/001/) | `hunting-for-data-staging-before-exfiltration`, `performing-purple-team-atomic-testing` |

---

## 📡 Command and Control

**20 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1071](https://attack.mitre.org/techniques/T1071/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-network-covert-channels-in-malware`, `analyzing-ransomware-network-indicators`, `analyzing-threat-actor-ttps-with-mitre-attack`, `hunting-advanced-persistent-threats` +6 more |
| [T1071.001](https://attack.mitre.org/techniques/T1071/001/) | `analyzing-apt-group-with-mitre-navigator`, `analyzing-campaign-attribution-evidence`, `analyzing-powershell-empire-artifacts`, `analyzing-powershell-script-block-logging`, `building-attack-pattern-library-from-cti-reports` +13 more |
| [T1071.004](https://attack.mitre.org/techniques/T1071/004/) | `building-attack-pattern-library-from-cti-reports`, `building-c2-infrastructure-with-sliver-framework`, `hunting-for-beaconing-with-frequency-analysis`, `hunting-for-command-and-control-beaconing`, `hunting-for-dns-tunneling-with-zeek` +3 more |
| [T1090](https://attack.mitre.org/techniques/T1090/) | `implementing-mitre-attack-coverage-mapping`, `performing-purple-team-atomic-testing` |
| [T1090.001](https://attack.mitre.org/techniques/T1090/001/) | `performing-purple-team-atomic-testing` |
| [T1090.002](https://attack.mitre.org/techniques/T1090/002/) | `building-c2-infrastructure-with-sliver-framework`, `building-red-team-c2-infrastructure-with-havoc` |
| [T1090.004](https://attack.mitre.org/techniques/T1090/004/) | `hunting-for-domain-fronting-c2-traffic` |
| [T1095](https://attack.mitre.org/techniques/T1095/) | `hunting-for-command-and-control-beaconing`, `hunting-for-unusual-network-connections` |
| [T1102](https://attack.mitre.org/techniques/T1102/) | `hunting-for-living-off-the-cloud-techniques` |
| [T1105](https://attack.mitre.org/techniques/T1105/) | `analyzing-powershell-script-block-logging`, `building-attack-pattern-library-from-cti-reports`, `building-c2-infrastructure-with-sliver-framework`, `building-red-team-c2-infrastructure-with-havoc`, `detecting-fileless-attacks-on-endpoints` +7 more |
| [T1132](https://attack.mitre.org/techniques/T1132/) | `hunting-for-command-and-control-beaconing`, `performing-purple-team-atomic-testing` |
| [T1132.001](https://attack.mitre.org/techniques/T1132/001/) | `building-c2-infrastructure-with-sliver-framework`, `performing-purple-team-atomic-testing` |
| [T1219](https://attack.mitre.org/techniques/T1219/) | `performing-purple-team-atomic-testing` |
| [T1568](https://attack.mitre.org/techniques/T1568/) | `hunting-for-command-and-control-beaconing`, `implementing-mitre-attack-coverage-mapping` |
| [T1568.002](https://attack.mitre.org/techniques/T1568/002/) | `hunting-for-beaconing-with-frequency-analysis` |
| [T1571](https://attack.mitre.org/techniques/T1571/) | `hunting-for-unusual-network-connections`, `implementing-mitre-attack-coverage-mapping` |
| [T1572](https://attack.mitre.org/techniques/T1572/) | `building-c2-infrastructure-with-sliver-framework`, `hunting-for-command-and-control-beaconing`, `hunting-for-dns-tunneling-with-zeek`, `implementing-mitre-attack-coverage-mapping` |
| [T1573](https://attack.mitre.org/techniques/T1573/) | `analyzing-ransomware-network-indicators`, `hunting-for-beaconing-with-frequency-analysis`, `hunting-for-command-and-control-beaconing`, `implementing-mitre-attack-coverage-mapping`, `performing-purple-team-atomic-testing` |
| [T1573.001](https://attack.mitre.org/techniques/T1573/001/) | `performing-purple-team-atomic-testing` |
| [T1573.002](https://attack.mitre.org/techniques/T1573/002/) | `building-c2-infrastructure-with-sliver-framework`, `building-red-team-c2-infrastructure-with-havoc` |

---

## 📤 Exfiltration

**12 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1020](https://attack.mitre.org/techniques/T1020/) | `hunting-for-data-exfiltration-indicators` |
| [T1029](https://attack.mitre.org/techniques/T1029/) | `hunting-for-data-exfiltration-indicators` |
| [T1030](https://attack.mitre.org/techniques/T1030/) | `hunting-for-data-exfiltration-indicators` |
| [T1041](https://attack.mitre.org/techniques/T1041/) | `analyzing-campaign-attribution-evidence`, `analyzing-ransomware-network-indicators`, `building-attack-pattern-library-from-cti-reports`, `conducting-full-scope-red-team-engagement`, `conducting-malware-incident-response` +6 more |
| [T1048](https://attack.mitre.org/techniques/T1048/) | `building-attack-pattern-library-from-cti-reports`, `building-detection-rule-with-splunk-spl`, `conducting-full-scope-red-team-engagement`, `hunting-for-data-exfiltration-indicators`, `implementing-continuous-security-validation-with-bas` +2 more |
| [T1048.001](https://attack.mitre.org/techniques/T1048/001/) | `hunting-for-data-exfiltration-indicators` |
| [T1048.002](https://attack.mitre.org/techniques/T1048/002/) | `hunting-for-data-exfiltration-indicators` |
| [T1048.003](https://attack.mitre.org/techniques/T1048/003/) | `conducting-full-scope-red-team-engagement`, `hunting-for-data-exfiltration-indicators`, `hunting-for-dns-tunneling-with-zeek`, `implementing-continuous-security-validation-with-bas`, `implementing-mitre-attack-coverage-mapping` +2 more |
| [T1052](https://attack.mitre.org/techniques/T1052/) | `hunting-for-data-exfiltration-indicators` |
| [T1537](https://attack.mitre.org/techniques/T1537/) | `hunting-for-data-exfiltration-indicators`, `hunting-for-living-off-the-cloud-techniques`, `implementing-mitre-attack-coverage-mapping`, `implementing-threat-modeling-with-mitre-attack`, `performing-cloud-incident-containment-procedures` |
| [T1567](https://attack.mitre.org/techniques/T1567/) | `detecting-insider-threat-behaviors`, `hunting-for-data-exfiltration-indicators`, `hunting-for-living-off-the-cloud-techniques`, `implementing-continuous-security-validation-with-bas`, `performing-purple-team-atomic-testing` |
| [T1567.002](https://attack.mitre.org/techniques/T1567/002/) | `hunting-for-data-exfiltration-indicators`, `performing-purple-team-atomic-testing` |

---

## 💥 Impact

**6 techniques covered**

| Technique | Skills |
|:----------|:-------|
| [T1485](https://attack.mitre.org/techniques/T1485/) | `hunting-for-shadow-copy-deletion`, `performing-purple-team-atomic-testing` |
| [T1486](https://attack.mitre.org/techniques/T1486/) | `analyzing-ransomware-network-indicators`, `building-attack-pattern-library-from-cti-reports`, `building-threat-hunt-hypothesis-framework`, `conducting-full-scope-red-team-engagement`, `hunting-for-shadow-copy-deletion` +7 more |
| [T1489](https://attack.mitre.org/techniques/T1489/) | `conducting-full-scope-red-team-engagement`, `performing-purple-team-atomic-testing` |
| [T1490](https://attack.mitre.org/techniques/T1490/) | `building-soc-playbook-for-ransomware`, `hunting-for-shadow-copy-deletion`, `performing-purple-team-atomic-testing`, `performing-purple-team-exercise` |
| [T1491](https://attack.mitre.org/techniques/T1491/) | `performing-purple-team-atomic-testing` |
| [T1491.002](https://attack.mitre.org/techniques/T1491/002/) | `performing-purple-team-atomic-testing` |

---

## 🔧 Other / Cross-Tactic Techniques

| Technique | Skills |
|:----------|:-------|
| T0157 | `exploiting-kerberoasting-with-impacket` |
| T0200 | `building-vulnerability-scanning-workflow`, `performing-authenticated-scan-with-openvas` |
| T0802 | `detecting-attacks-on-historian-servers` |
| T0809 | `detecting-attacks-on-historian-servers` |
| T0814 | `detecting-modbus-command-injection-attacks` |
| T0816 | `detecting-dnp3-protocol-anomalies` |
| T0830 | `detecting-modbus-protocol-anomalies` |
| T0831 | `detecting-modbus-protocol-anomalies` |
| T0832 | `detecting-attacks-on-historian-servers` |
| T0833 | `detecting-stuxnet-style-attacks` |
| T0836 | `detecting-modbus-command-injection-attacks`, `detecting-modbus-protocol-anomalies`, `detecting-stuxnet-style-attacks` |
| T0839 | `detecting-dnp3-protocol-anomalies`, `detecting-stuxnet-style-attacks` |
| T0843 | `detecting-modbus-command-injection-attacks`, `performing-s7comm-protocol-security-analysis` |
| T0847 | `detecting-stuxnet-style-attacks` |
| T0855 | `detecting-dnp3-protocol-anomalies`, `detecting-modbus-command-injection-attacks`, `detecting-modbus-protocol-anomalies` |
| T0856 | `detecting-stuxnet-style-attacks` |
| T0862 | `detecting-stuxnet-style-attacks` |
| T0866 | `detecting-stuxnet-style-attacks` |
| T0869 | `detecting-dnp3-protocol-anomalies` |
| T0881 | `performing-s7comm-protocol-security-analysis` |
| T0886 | `detecting-modbus-protocol-anomalies` |
| T1404 | `analyzing-android-malware-with-apktool` |
| T1417 | `analyzing-android-malware-with-apktool` |
| T1418 | `analyzing-android-malware-with-apktool` |
| T1553.006 | `analyzing-uefi-bootkit-persistence` |
| T1555 | `performing-credential-access-with-lazagne`, `performing-purple-team-atomic-testing` |
| T1555.003 | `performing-credential-access-with-lazagne`, `performing-purple-team-atomic-testing` |
| T1555.004 | `performing-credential-access-with-lazagne` |
| T1578 | `performing-cloud-incident-containment-procedures` |
| T1582 | `analyzing-android-malware-with-apktool` |
| T1611 | `detecting-container-escape-attempts`, `detecting-container-escape-with-falco-rules` |
| T1615 | `conducting-internal-reconnaissance-with-bloodhound-ce`, `exploiting-active-directory-with-bloodhound`, `performing-active-directory-bloodhound-analysis` |
| T1620 | `detecting-fileless-attacks-on-endpoints` |
| T5577 | `performing-physical-intrusion-assessment` |

---

## How This Was Generated

This coverage map was automatically generated by scanning all 753+ SKILL.md and agent.py files for MITRE ATT&CK technique IDs (pattern: `T####` and `T####.###`). Each technique was mapped to its parent tactic using the [MITRE ATT&CK Enterprise Matrix v16](https://attack.mitre.org/matrices/enterprise/).

To regenerate: `python3 extract_attack.py`

---

## MITRE ATLAS Coverage (v5.5.0)

81 skills mapped to ATLAS adversarial ML techniques.

Key techniques applied:
- AML.T0051 — LLM Prompt Injection (Execution)
- AML.T0054 — LLM Jailbreak (Privilege Escalation)
- AML.T0088 — Generate Deepfakes (AI Attack Staging)
- AML.T0010 — AI Supply Chain Compromise (Initial Access)
- AML.T0020 — Poison Training Data (Resource Development)
- AML.T0070 — RAG Poisoning (Persistence)
- AML.T0080 — AI Agent Context Poisoning (Persistence)
- AML.T0056 — Extract LLM System Prompt (Exfiltration)

## MITRE D3FEND Coverage (v1.3)

11 skills mapped to D3FEND defensive countermeasures.

Countermeasures applied span D3FEND tactical categories:
Harden, Detect, Isolate, Deceive, Evict, Restore.
Each skill's d3fend_techniques field lists the top 5 most relevant
defensive countermeasures derived from the skill's ATT&CK technique tags.

## NIST AI RMF Coverage (AI 100-1)

85 skills mapped to NIST AI Risk Management Framework subcategories.

Core functions covered:
- GOVERN: Organizational accountability for AI risk (GOVERN-1.1, GOVERN-6.1, GOVERN-6.2)
- MAP: AI risk identification and context (MAP-5.1, MAP-5.2, MAP-1.6)
- MEASURE: AI risk analysis and evaluation (MEASURE-2.5, MEASURE-2.7, MEASURE-2.8, MEASURE-2.11)
- MANAGE: AI risk response and recovery (MANAGE-2.4, MANAGE-3.1)

GenAI-specific subcategories applied: GOVERN-6.1, GOVERN-6.2 (responsible deployment policies).

---

<p align="center">
  <sub>Part of <a href="https://github.com/mukul975/Anthropic-Cybersecurity-Skills">Anthropic Cybersecurity Skills</a> — 817 open-source cybersecurity skills for AI agents</sub>
</p>
===== END FILE =====


===== FILE: ./CITATION.cff =====
cff-version: 1.2.0
message: "If you use this repository in your research, tools, or publications, please cite it as below."
type: software
title: "Anthropic-Cybersecurity-Skills"
abstract: >
  A structured collection of 817 cybersecurity skills for AI agents, covering
  penetration testing, digital forensics, threat intelligence, incident response,
  cloud security, OT/SCADA security, AI security, and more. Each skill follows
  a standardized format with YAML frontmatter metadata, step-by-step procedures,
  tool commands, expected outputs, and MITRE ATT&CK mappings. Compatible with
  Claude Code, GitHub Copilot, Cursor, Windsurf, Gemini CLI, and 20+ AI agent
  platforms.
authors:
  - name: "Mahipal"
    email: mukuljangra5@gmail.com
    alias: mukul975
repository-code: "https://github.com/mukul975/Anthropic-Cybersecurity-Skills"
url: "https://github.com/mukul975/Anthropic-Cybersecurity-Skills"
license: Apache-2.0
version: "1.1.0"
date-released: "2026-03-21"
keywords:
  - cybersecurity
  - AI agents
  - skills
  - penetration testing
  - digital forensics
  - threat intelligence
  - incident response
  - MITRE ATT&CK
  - Claude Code
  - open source

===== END FILE =====


===== FILE: ./CODE_OF_CONDUCT.md =====
# Contributor Covenant Code of Conduct

## Our Pledge

We as members, contributors, and leaders pledge to make participation in our community a harassment-free experience for everyone, regardless of age, body size, visible or invisible disability, ethnicity, sex characteristics, gender identity and expression, level of experience, education, socio-economic status, nationality, personal appearance, race, caste, color, religion, or sexual identity and orientation.

We pledge to act and interact in ways that contribute to an open, welcoming, diverse, inclusive, and healthy community.

## Our Standards

Examples of behavior that contributes to a positive environment for our community include:

* Demonstrating empathy and kindness toward other people
* Being respectful of differing opinions, viewpoints, and experiences
* Giving and gracefully accepting constructive feedback
* Accepting responsibility and apologizing to those affected by our mistakes, and learning from the experience
* Focusing on what is best not just for us as individuals, but for the overall community

Examples of unacceptable behavior include:

* The use of sexualized language or imagery, and sexual attention or advances of any kind
* Trolling, insulting or derogatory comments, and personal or political attacks
* Public or private harassment
* Publishing others' private information, such as a physical or email address, without their explicit permission
* Other conduct which could reasonably be considered inappropriate in a professional setting

## Enforcement Responsibilities

Community leaders are responsible for clarifying and enforcing our standards of acceptable behavior and will take appropriate and fair corrective action in response to any behavior that they deem inappropriate, threatening, offensive, or harmful.

Community leaders have the right and responsibility to remove, edit, or reject comments, commits, code, wiki edits, issues, and other contributions that are not aligned to this Code of Conduct, and will communicate reasons for moderation decisions when appropriate.

## Scope

This Code of Conduct applies within all community spaces, and also applies when an individual is officially representing the community in public spaces. Examples of representing our community include using an official e-mail address, posting via an official social media account, or acting as an appointed representative at an online or offline event.

## Enforcement

Instances of abusive, harassing, or otherwise unacceptable behavior may be reported to the community leaders responsible for enforcement at mukuljangra5@gmail.com. All complaints will be reviewed and investigated promptly and fairly.

All community leaders are obligated to respect the privacy and security of the reporter of any incident.

## Enforcement Guidelines

Community leaders will follow these Community Impact Guidelines in determining the consequences for any action they deem in violation of this Code of Conduct:

### 1. Correction

**Community Impact**: Use of inappropriate language or other behavior deemed unprofessional or unwelcome in the community.

**Consequence**: A private, written warning from community leaders, providing clarity around the nature of the violation and an explanation of why the behavior was inappropriate. A public apology may be requested.

### 2. Warning

**Community Impact**: A violation through a single incident or series of actions.

**Consequence**: A warning with consequences for continued behavior. No interaction with the people involved, including unsolicited interaction with those enforcing the Code of Conduct, for a specified period of time. This includes avoiding interactions in community spaces as well as external channels like social media. Violating these terms may lead to a temporary or permanent ban.

### 3. Temporary Ban

**Community Impact**: A serious violation of community standards, including sustained inappropriate behavior.

**Consequence**: A temporary ban from any sort of interaction or public communication with the community for a specified period of time. No public or private interaction with the people involved, including unsolicited interaction with those enforcing the Code of Conduct, is allowed during this period. Violating these terms may lead to a permanent ban.

### 4. Permanent Ban

**Community Impact**: Demonstrating a pattern of violation of community standards, including sustained inappropriate behavior, harassment of an individual, or aggression toward or disparagement of classes of individuals.

**Consequence**: A permanent ban from any sort of public interaction within the community.

## Attribution

This Code of Conduct is adapted from the [Contributor Covenant][homepage], version 2.1, available at [https://www.contributor-covenant.org/version/2/1/code_of_conduct.html][v2.1].

Community Impact Guidelines were inspired by [Mozilla's code of conduct enforcement ladder][Mozilla CoC].

For answers to common questions about this code of conduct, see the FAQ at [https://www.contributor-covenant.org/faq][FAQ]. Translations are available at [https://www.contributor-covenant.org/translations][translations].

[homepage]: https://www.contributor-covenant.org
[v2.1]: https://www.contributor-covenant.org/version/2/1/code_of_conduct.html
[Mozilla CoC]: https://github.com/mozilla/diversity
[FAQ]: https://www.contributor-covenant.org/faq
[translations]: https://www.contributor-covenant.org/translations

===== END FILE =====


===== FILE: ./CONTRIBUTING.md =====
# Contributing to Anthropic-Cybersecurity-Skills

Before you start, read [SCOPE.md](SCOPE.md). It defines what belongs in this catalogue and what does not, and it will save you building something I have to decline.

Most of the policy below is new. Until now this file said nothing about scope, overlap, disclosure or pull-request size, and the absence of those rules is why several well-made contributions ended up somewhere I could not merge them. That is my failure to document, not anyone's failure to read. **These rules apply to pull requests opened from here on.** Anything already open will be reviewed as it was filed.

## How to add a new skill

1. Create a new directory: `skills/your-skill-name/`
2. Add a `SKILL.md` file with required YAML frontmatter:
   ```yaml
   ---
   name: your-skill-name
   description: >-
     What the skill does, in one sentence naming the real tools and artefacts.
     Use when THE TRIGGER CONDITION HOLDS. Keywords: tool, flag, artefact.
     Do not use for THE ADJACENT TASK - use the-other-skill-name.
   domain: cybersecurity
   subdomain: threat-hunting
   tags: [tag1, tag2, tag3]
   version: "1.0"
   author: your-github-username
   license: Apache-2.0
   ---
   ```
   `subdomain` is a single value, not a list — `subdomain: [threat-hunting]` is a YAML list and will fail validation. `name` must equal the directory name. Frontmatter values may not contain `<` or `>`.
3. Write clear, step-by-step instructions in the Markdown body using these sections:
   - ## When to Use
   - ## Prerequisites
   - ## Workflow (numbered steps with real commands)
   - ## Key Concepts (table)
   - ## Tools & Systems
   - ## Common Scenarios
   - ## Output Format
4. (Optional) Add supporting files:
   - `references/standards.md` — Real standard numbers, CVE refs, NIST/MITRE links
   - `references/workflows.md` — Deep technical procedure
   - `scripts/process.py` — Real working helper script
   - `assets/template.md` — Real filled-in checklist/template
5. Submit a PR with title: `Add skill: your-skill-name`

## Skill quality checklist
- [ ] Name is lowercase with hyphens (kebab-case), 1–64 characters
- [ ] Description is clear and includes agent-discovery keywords
- [ ] Instructions are actionable with real commands and tool names
- [ ] Domain and subdomain are set correctly
- [ ] Tags include relevant tools, frameworks, and techniques

## Writing the description

The description is the only text an agent sees when it decides whether to load your skill. Everything else in the file is invisible at that moment. Four things have to be in it:

1. **What it does** — the procedure, the real tools, the real artefacts.
2. **When to fire** — an explicit `Use when …` clause.
3. **Keywords** — the flags, event IDs, file names and tool names someone would actually search for.
4. **When *not* to fire** — a `Do not use for …` clause that names the nearest neighbouring skill by slug.

A worked example, the current description of `scanning-docker-images-with-trivy`:

> Scans a Docker image with Trivy for vulnerabilities in OS packages and language dependencies, misconfiguration, exposed secrets, and licence violations, emitting SARIF, CycloneDX, or SPDX output. Use when scanning or gating a specific image, wiring an image scan into CI/CD, or checking an image during an incident investigation. Keywords: Trivy, image scan, --severity, --exit-code, SARIF, ignore file, .trivyignore. Do not use for cluster-wide scanning or non-image targets - use performing-container-security-scanning-with-trivy; when the toolchain is Grype use scanning-container-images-with-grype.

The linter enforces the mechanical parts: 1024 characters maximum, terminal punctuation, a trigger clause, a negative trigger, and a 500-line cap on the file. Roughly 980 pre-existing failures are grandfathered in `tools/lint-baseline.json` so the gate blocks new debt only. That baseline may shrink and may never grow, which means a new skill has to meet the standard even though many old ones do not yet.

## Before you open a pull request

Run the validators locally. All five run in CI on every push and pull request that touches `skills/` or `tools/` — which any skill PR does — and all five have to be green before I can merge. A PR that changes only documentation does not trigger them at all, so an empty checks list there is not a pass.

```bash
pip install pyyaml   # the only external dependency

python tools/validate-skill.py skills/your-skill-name/      # frontmatter, this skill
python tools/lint-descriptions.py skills/your-skill-name/   # description quality, this skill
python tools/validate-agentskills.py --strict               # conformance, whole repo
python tools/detect-collisions.py                           # near-duplicates, whole repo
python tools/generate-index.py                              # regenerate index.json
```

Three things that catch people out:

- **Two of those commands are repo-wide.** `validate-agentskills.py` and `detect-collisions.py` have no single-skill mode, so a failure they report may belong to a skill you never touched. Check the slug in the output before assuming it is yours.
- **`index.json` is generated, and the PR must carry the regenerated file.** It is refreshed automatically on `main`, but not on your branch, and the freshness gate runs at PR time. If you touched a description and did not commit the regenerated `index.json`, the build fails.
- **The collision gate is a ratchet, currently sitting exactly at its ceiling.** One new near-duplicate pair fails the build. The cap CI enforces lives in `.github/workflows/validate-skills.yml` and gets lowered as disambiguation lands — read it from there rather than memorising a number.

## One skill per pull request

Open one pull request per skill.

Each skill is reviewed for technical accuracy — whether the flags exist, whether the output format is real, whether the procedure works on the version you claim. That review does not batch. In a ten-skill pull request, one wrong procedure holds up nine good ones, and the whole thing tends to stall.

A PR that adds more than one skill directory will be asked to split. It is not a judgement on the work; it is the only way I can land the good parts quickly. The exception is a mechanical repo-wide change — a lint sweep, a metadata fix across many files — which is fine in one PR as long as the description says plainly what the change is and that nothing else varies.

## Overlap with an existing skill

The catalogue currently has 55 unreviewed near-duplicate description pairs, involving 94 of the 817 skills. Overlapping descriptions are not a cosmetic problem: when two descriptions look alike, the agent picks the wrong one, and both skills get less useful.

Until that backlog is worked down, **a new skill that overlaps an existing one will usually be asked to extend the existing skill instead.** Adding depth to `performing-firmware-extraction-with-binwalk` is worth more to this catalogue right now than a second firmware-extraction skill beside it. That redirect is about where the work lands, not about its quality — an extension PR carries the same authorship and gets the same credit.

Before you write a new skill:

1. Search `index.json` for the tool, the technique and the artefact.
2. Run `python tools/detect-collisions.py` and see whether your intended description lands near anything.
3. If something close exists, open an issue proposing the extension, or send a PR against the existing skill.

If the overlap is real but the skills genuinely need to stay separate — different operating system, different tool, different stage of the same investigation — say so in the PR and make both descriptions name the other explicitly. That is how the split is recorded, and it is what `tools/collision-allowlist.json` is for.

## Review and response

I review every pull request myself, and the queue is currently longer than I would like. Small, focused PRs move fastest.

If a PR gets a review request and then goes quiet for 14 days, I may close it as stale. That is housekeeping, not rejection: your branch and your work are untouched, and a single comment reopens the conversation whenever you are ready to pick it up.

## Disclosure

### Self-promotion and vendor links

No undisclosed self-links. If a skill, a reference file, a README entry or a script links to a product, service, repository or domain you are involved with, say so in the PR. A link that exists to send traffic somewhere rather than to help the reader complete the procedure will be removed.

Many skills legitimately cover commercial tools, including ones with no free tier at all. That is fine — the procedure is the point. What is required of vendor-specific content is:

- **Honesty about cost.** If the procedure needs a paid licence, an enterprise appliance or a sales conversation, say so in `## Prerequisites` rather than letting a reader find out at step six.
- **Documentation links, not marketing links.** Link the vendor's docs, API reference or CLI manual. No pricing pages, no signup funnels, no referral or campaign-tagged URLs.
- **No cross-selling.** A general procedure should not route the reader toward one vendor's product when the task does not require it.

The `## Tools & Resources` lists are a special case. They carry standards, RFCs and vendor-neutral documentation. If you want to add a commercial product to one, open an issue first naming what it does that no entry already in that list does. "It also does this" is not enough: there are 87 of these lists in the repository and every vendor in a given space has an equal claim on them, so without that bar they turn into directories.

This applies to me as well. Where the README links to something of mine, it should be labelled as mine.

### Affiliation

If you are affiliated with a product, service, vendor, project or domain referenced in your PR — you work there, contract for it, founded it, are paid by it, or maintain it — say so in the PR description. One line is enough.

Disclosure is not disqualifying. People who build a tool often write the most accurate procedure for it, and I would rather have that procedure with a disclosure than a vaguer one without. It is the non-disclosure that damages trust, because it turns every later reader into someone who has to guess. If it is unclear to me, I will ask; a plain answer settles it.

If an undisclosed connection surfaces during review, that is the entire consequence: the PR goes on hold until it is stated, and is then reviewed on its merits like anything else. Adding the disclosure when asked carries no penalty, and nothing is closed over it.

### AI-assisted contributions

AI assistance is allowed. This is a repository of skills for AI agents; banning it would be absurd.

What is required is disclosure and human responsibility:

- **Say so in the PR description.** One line is enough.
- **A human must have run the commands.** Not read them, not sanity-checked them — run them, on a real system, and seen the output that is now in the skill.
- **A human takes responsibility.** The `author` frontmatter field names a person or a team account, not a tool, and that account should be able to answer review questions about the procedure.
- **It still has to pass the validators**, like everything else.

The failure mode to guard against is a generated procedure that was never executed: flags that look plausible but do not exist, options borrowed from a different major version, output formats that were invented rather than observed. That kind of content is worse than no skill at all, because an agent will follow it confidently. Generated and verified is welcome; generated and unverified is not.

## Subdomains

Choose the most appropriate subdomain for your skill. `tools/validate-skill.py` is the source of truth; these 34 are the canonical values. A handful of older aliases are still accepted for existing skills and are listed beside their canonical form — the validator prints a warning for them, and new skills should use the canonical value.

- `ai-security`
- `api-security`
- `blockchain-security`
- `cloud-security`
- `compliance-governance` — also accepts `governance-risk-compliance`
- `container-security`
- `cryptography`
- `data-protection`
- `deception-technology`
- `devsecops`
- `digital-forensics`
- `endpoint-security`
- `hardware-firmware-security` — also accepts `firmware-analysis`, `firmware-security`
- `identity-access-management` — also accepts `identity-and-access-management`, `identity-security`
- `incident-response`
- `malware-analysis`
- `mobile-security`
- `network-security`
- `ot-ics-security` — also accepts `ot-security`
- `penetration-testing` — also accepts `offensive-security`
- `phishing-defense` — also accepts `social-engineering-defense`
- `privacy-compliance`
- `purple-team`
- `ransomware-defense`
- `red-teaming` — also accepts `red-team`
- `soc-operations` — also accepts `security-operations`
- `supply-chain-security`
- `threat-detection`
- `threat-hunting`
- `threat-intelligence`
- `vulnerability-management`
- `web-application-security` — also accepts `application-security`
- `wireless-security`
- `zero-trust-architecture` — also accepts `zero-trust`

If none of these fits, open an issue before you submit. Adding a subdomain means changing the validator, and that is a separate conversation from adding a skill.

The thinnest subdomains are the ones most worth contributing to: `data-protection` and `purple-team` have one skill each, and `blockchain-security`, `wireless-security` and `privacy-compliance` have two.

## Code of Conduct
This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md). By participating, you agree to uphold this code.

## License
By contributing, you agree that your contributions will be licensed under Apache-2.0.

===== END FILE =====


===== FILE: ./LICENSE =====

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


===== FILE: ./README.fr.md =====
<p align="center">
  <img src="assets/banner.png" alt="Anthropic Cybersecurity Skills" width="100%">
</p>

<div align="center">

# Compétences Cybersecurity Anthropic

### La plus grande bibliothèque open source de compétences cybersécurité pour agents IA

[![GARS-2026 Survey](https://img.shields.io/badge/GARS--2026-Take%20the%20Survey-E8B84B?style=for-the-badge&logo=googleforms&logoColor=black)](https://mahipal.engineer/survey?utm_source=github_badge&utm_medium=readme&utm_campaign=gars2026)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg?style=flat-square)](LICENSE)
[![Skills](https://img.shields.io/badge/skills-817-brightgreen?style=flat-square)](#ce-quil-contient-29-domaines-de-securite)
[![Frameworks](https://img.shields.io/badge/frameworks-6-orange?style=flat-square)](#six-frameworks-une-seule-bibliotheque-de-competences)
[![MITRE F3](https://img.shields.io/badge/MITRE-F3_v1.1-blue?style=flat-square)](https://ctid.mitre.org/fraud/)
[![Domains](https://img.shields.io/badge/domains-29-9cf?style=flat-square)](#ce-quil-contient-29-domaines-de-securite)
[![Platforms](https://img.shields.io/badge/platforms-26%2B-blueviolet?style=flat-square)](#plateformes-compatibles)
[![Last Commit](https://img.shields.io/github/last-commit/mukul975/Anthropic-Cybersecurity-Skills?style=flat-square)](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/commits/main)
[![agentskills.io](https://img.shields.io/badge/standard-agentskills.io-ff6600?style=flat-square)](https://agentskills.io)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](CONTRIBUTING.md)
[![Playground](https://img.shields.io/badge/Playground-Casky.ai-blue)](https://casky.ai/?utm_source=github&utm_medium=readme&utm_campaign=cohort_launch#waitlist)
[![Hermes Agent](https://img.shields.io/badge/Hermes_Agent-compatible-blueviolet?style=flat)](https://github.com/NousResearch/hermes-agent)


**817 compétences cybersécurité de niveau production · 29 domaines de sécurité · 6 correspondances de frameworks · 26+ plateformes IA**

[Démarrage rapide](#demarrage-rapide) · [Contenu](#ce-quil-contient-29-domaines-de-securite) · [Frameworks](#six-frameworks-une-seule-bibliotheque-de-competences) · [Plateformes](#plateformes-compatibles) · [Contribuer](#contribuer)

</div>

---

> ⚠️ **Projet communautaire** — Il s'agit d'un projet indépendant créé par la communauté. Non affilié à Anthropic PBC.
>
> 🔐 **Usage autorisé et légal uniquement.** Cette bibliothèque inclut des techniques offensives et à double usage (ex. C2 red-team, simulation de phishing, exploitation) destinées à des **tests d'intrusion autorisés, à la recherche en sécurité, à la défense et à l'éducation**. Ne les utilisez que sur des systèmes que vous possédez ou pour lesquels vous avez une **autorisation écrite explicite** de tester, et respectez toutes les lois applicables ainsi que les règles d'engagement. Vous êtes seul responsable de la façon dont vous utilisez ces compétences. Voir SECURITY.md et CODE_OF_CONDUCT.md.

## Donnez à n'importe quel agent IA les compétences de sécurité d'un analyste senior

Un analyste junior sait quel plugin Volatility3 exécuter sur un dump mémoire suspect, quelles règles Sigma détectent un Kerberoasting, et comment analyser une compromission cloud sur trois fournisseurs. **Votre agent IA ne le sait pas — à moins que vous ne lui fournissiez ces compétences.**

Ce dépôt contient **817 compétences cybersécurité structurées** couvrant **29 domaines de sécurité**, chacune suivant le standard ouvert agentskills.io. Chaque compétence est mappée à **six frameworks de l'industrie** — MITRE ATT&CK, NIST CSF 2.0, MITRE ATLAS, MITRE D3FEND, NIST AI RMF, et le MITRE Fight Fraud Framework (F3) — ce qui en fait la seule bibliothèque open source de compétences offrant une couverture unifiée inter-framework. Clonez-la, pointez votre agent dessus, et votre prochaine investigation sécurité bénéficiera d'un guidage de niveau expert en quelques secondes.

## Six frameworks, une seule bibliothèque de compétences

Aucune autre bibliothèque open source de compétences ne mappe chaque compétence à l'ensemble de ces frameworks. Une compétence, six cases de conformité cochées.

|Framework|Version|Couverture dans ce dépôt|Ce qui est mappé|
|--|--|--|--|
|MITRE ATT&CK|v19.1|15 tactiques · 286 techniques|Comportements adverses et TTP|
|NIST CSF 2.0|2.0|6 fonctions · 22 catégories|Posture de sécurité organisationnelle|
|MITRE ATLAS|v5.4|16 tactiques · 84 techniques|Menaces adverses sur IA/ML|
|MITRE D3FEND|v1.3|7 catégories · 267 techniques|Contre-mesures défensives|
|NIST AI RMF|1.0|4 fonctions · 72 sous-catégories|Gestion des risques IA|
|MITRE F3 (Fight Fraud Framework)|v1.1 (2026-04-09)|8 tactiques · 123 techniques · 94 compétences liées à la fraude|TTP de fraude financière cybernétique|

**Exemple — une seule compétence mappée sur les six frameworks :**

|Compétence|ATT&CK|NIST CSF|ATLAS|D3FEND|AI RMF|F3|
|--|--|--|--|--|--|--|
|`analyzing-network-traffic-of-malware`|T1071|DE.CM|AML.T0047|D3-NTA|MEASURE-2.6|—|
|`detecting-business-email-compromise`|T1566|DE.AE|—|—|—|F1005.006 · monétisation|

### 🆕 MITRE Fight Fraud Framework (F3) — 94 compétences liées à la fraude

Le **MITRE Fight Fraud Framework (F3)** a été publié le **9 avril 2026** par le Center for Threat-Informed Defense (CTID) de MITRE, co-développé avec JPMorganChase, Citigroup, Lloyds Banking Group, Standard Chartered, CrowdStrike, Verizon Business, FS-ISAC, et d'autres. Il s'agit d'un catalogue TTP compatible ATT&CK pour la **fraude financière cybernétique** — comblant le vide laissé par ATT&CK après la compromission initiale.

F3 v1.1 ajoute **deux tactiques spécifiques à la fraude** qu'ATT&CK n'énumère pas :

- **Positioning** (`FA0001`) — actions menées après l'accès pour collecter/manipuler des données et préparer la fraude (ensemencement d'identité synthétique, mise en chauffe de compte, configuration de bénéficiaire, pré-positionnement pour SIM swap, hijack de session bancaire).
- **Monetization** (`FA0002`) — conversion d'actifs volés en fonds utilisables (empilement via money mule, fraude APP, off-ramping crypto, cash-out par carte, abus remboursement/chargeback).

Les techniques spécifiques à la fraude utilisent des IDs `F1XXX` (par ex. `F1005.003` Add Beneficiary, `F1025.003` Wire Transfer, `F1007` Adversary-in-the-Browser) ; les techniques ATT&CK réutilisées conservent leurs IDs `T1XXX`. Les mappings se trouvent dans chaque bloc de frontmatter `mitre_f3:` — les 123 IDs de techniques F3 v1.1 ont tous été vérifiés à partir du bundle STIX upstream. Voir `docs/mitre-f3-mapping.md` pour le schéma.

### MITRE ATT&CK v19.1 — 817/817 compétences mappées

Chaque compétence contient une liste `mitre_attack` en frontmatter validée contre **MITRE ATT&CK v19.1** (la dernière version) à l'aide de la bibliothèque officielle `mitreattack-python` — 286 techniques distinctes sur les 15 tactiques Enterprise, plus des techniques ICS et Mobile lorsque pertinent. Aucun ID révoqué ou obsolète. La restructuration de Defense Evasion dans v19.1 (désormais séparée en **Stealth** et **Defense Impairment**) est reflétée ci-dessous.

|Tactic|ID|Skills|
|--|--|--|
|Reconnaissance|TA0043|103|
|Resource Development|TA0042|22|
|Initial Access|TA0001|467|
|Execution|TA0002|350|
|Persistence|TA0003|444|
|Privilege Escalation|TA0004|464|
|Stealth|TA0005|442|
|Defense Impairment|TA0112|92|
|Credential Access|TA0006|202|
|Discovery|TA0007|237|
|Lateral Movement|TA0008|68|
|Collection|TA0009|172|
|Command and Control|TA0011|123|
|Exfiltration|TA0010|82|
|Impact|TA0040|50|

## Démarrage rapide

```bash
# Option 1 : npx (recommandé)
npx skills add mukul975/Anthropic-Cybersecurity-Skills

# Option 2 : clonage Git
git clone https://github.com/mukul975/Anthropic-Cybersecurity-Skills.git
cd Anthropic-Cybersecurity-Skills
```

Fonctionne immédiatement avec Claude Code, GitHub Copilot, OpenAI Codex CLI, Cursor, Gemini CLI, et toute plateforme compatible agentskills.io.

## 🌍 GARS-2026 — Global Agentic AI Readiness Survey

Je mène une étude académique mondiale mesurant à quel point les professionnels de la sécurité, les développeurs et les équipes d'entreprise sont réellement prêts pour l'IA agentique — serveurs MCP, tool calling, gouvernance, et workflows human-in-the-loop.

**Si vous utilisez ce dépôt, votre réponse serait une donnée réellement précieuse.**

📋 **Répondez à l'enquête (10 min) :** [Survey Link](https://mahipal.engineer/survey?utm_source=github_badge&utm_medium=readme&utm_campaign=gars2026)

- 60 questions · Anonyme · Supervisé par SRH Berlin
- Vous recevez **50 Casky Tokens** pour l'accès anticipé à casky.ai
- Résultats publiés en open access sous CC-BY 4.0

## 🚀 Essayez-le dans le Playground

Découvrez Casky.ai en pratique — aucune configuration requise.

**[→ Lancer le Playground sur Casky.ai](https://casky.ai/?utm_source=github&utm_medium=readme&utm_campaign=cohort_launch#waitlist)**

Le playground vous permet de :

- Exécuter des exercices de compétences cybersécurité en direct sur de vraies cibles
- Voir des agents IA exécuter des compétences structurées en temps réel
- Explorer de manière interactive des workflows mappés à MITRE ATT&CK
- Tester des scénarios de threat hunting, DFIR et pentest

Pas d'installation. Pas de configuration. Juste ouvrir et commencer.

## Pourquoi ce projet existe

La pénurie de main-d'œuvre en cybersécurité a atteint **4,8 millions de postes non pourvus** dans le monde en 2024 (ISC2). Les agents IA peuvent aider à combler cet écart — mais seulement s'ils disposent d'une base de connaissances métier structurée.

Les agents actuels savent écrire du code et chercher sur le web, mais ils n'ont pas les playbooks praticiens qui transforment un LLM générique en analyste sécurité compétent.

Les dépôts d'outils de sécurité existants fournissent des wordlists, des payloads ou du code d'exploitation. Aucun ne fournit à un agent IA le workflow de décision structuré qu'un analyste senior suit : quand utiliser chaque technique, quelles préconditions vérifier, comment exécuter étape par étape, et comment valider les résultats. C'est cet écart que ce projet comble.

**Anthropic Cybersecurity Skills** n'est pas une collection de scripts ou de checklists. C'est une **base de connaissances native IA** conçue dès l'origine pour le standard agentskills.io — frontmatter YAML pour la découverte en sous-seconde, Markdown structuré pour l'exécution pas à pas, et fichiers de référence pour le contexte technique approfondi. Chaque compétence encode de vrais workflows praticiens, pas des résumés générés.

## Ce qu'il contient — 29 domaines de sécurité

|Domaine|Compétences|Capacités clés|
|--|--|--|
|Cloud Security|66|AWS, Azure, GCP hardening · CSPM · émulation d'attaques cloud · forensics cloud|
|Threat Hunting|58|Chasse guidée par hypothèses · détection LOTL · chasse EVTX · chasse à l'échelle du parc|
|Threat Intelligence|52|STIX/TAXII · MISP · OpenCTI · intégration de flux · profilage d'acteurs|
|Network Security|43|IDS/IPS · règles de pare-feu · segmentation VLAN · analyse du trafic|
|Web Application Security|42|OWASP Top 10 · SQLi · XSS · SSRF · désérialisation|
|Digital Forensics|41|Imagerie disque · forensique mémoire · chronologies Hayabusa/KAPE/Plaso|
|Malware Analysis|39|Analyse statique/dynamique · rétro-ingénierie · sandboxing|
|Identity & Access Management|37|Entra ID/ROADtools · phishing par code d'appareil · PAM · identité zero trust|
|SOC Operations|35|Playbooks · workflows d'escalade · détection Graph-log · exercices tabletop|
|Red Teaming|33|ADCS/Certipy · BloodHound CE · C2 Sliver/Havoc · relais NTLM|
|Container Security|33|RBAC Kubernetes · scan d'images · Falco · évasion de conteneurs|
|Security Operations|28|Corrélation SIEM · analyse de logs · triage d'alertes|
|OT/ICS Security|28|Modbus · DNP3 · IEC 62443 · défense des historian · SCADA|
|API Security|28|GraphQL · REST · OWASP API Top 10 · contournement WAF|
|Incident Response|26|Confinement de compromission · réponse ransomware · playbooks IR|
|Vulnerability Management|25|Nessus · workflows de scan · priorisation des patchs · CVSS|
|Penetration Testing|21|Réseau · web · cloud · mobile · mouvement latéral NetExec|
|DevSecOps|18|Sécurité CI/CD · scan Trivy IaC/images · signature de code|
|Zero Trust Architecture|17|BeyondCorp · modèle de maturité CISA · microsegmentation|
|Endpoint Security|17|EDR · détection LOTL · malware fileless · chasse à la persistance|
|Cryptography|16|TLS · Ed25519 · migration post-quantique · gestion des clés|
|Phishing Defense|15|Authentification email · détection BEC · IR phishing|
|AI Security|14|Red-teaming LLM (garak/PyRIT) · injection de prompt · sécurité MCP/agentique · garde-fous|
|Mobile Security|13|Analyse Android/iOS · pentest mobile · forensics MDM|
|Ransomware Defense|13|Détection précurseur · réponse · récupération · analyse du chiffrement|
|Compliance & Governance|9|NIST 800-30/RMF · CMMC · HIPAA · TPRM · CIS benchmarks|
|Supply Chain Security|8|SBOMs · confusion de dépendances · triage de paquets malveillants · SLSA/Sigstore|
|Deception Technology|6|Honeytokens · canarytokens · détection de compromission|
|Hardware & Firmware Security|4|Audit CHIPSEC/UEFI · contournement Secure Boot · attestation TPM · chasse aux bootkits|

## Comment les agents IA utilisent ces compétences

Chaque compétence coûte **environ 30 tokens à scanner** (seulement le frontmatter) et **500 à 2 000 tokens à charger entièrement** (workflow complet). Cette architecture de divulgation progressive permet aux agents de parcourir les 817 compétences en une seule passe sans exploser les fenêtres de contexte.

```text
Prompt utilisateur : "Analyse ce dump mémoire pour des signes de vol d'identifiants"

Processus interne de l'agent :

  1. Scanne les frontmatters des 817 compétences (~30 tokens chacun)
     → identifie 12 compétences pertinentes en faisant correspondre tags, description, domaine

  2. Charge les 3 meilleurs résultats :
     -  performing-memory-forensics-with-volatility3
     -  hunting-for-credential-dumping-lsass
     -  analyzing-windows-event-logs-for-credential-access

  3. Exécute le workflow structuré étape par étape
     → lance les plugins Volatility3, vérifie les schémas d'accès à LSASS,
        corrèle avec les preuves des journaux d'événements

  4. Valide les résultats à l'aide de la section Verification
     → confirme les IOCs, mappe les constatations vers ATT&CK T1003 (Credential Dumping)
```

**Sans ces compétences**, l'agent devine les commandes d'outils et rate des étapes critiques. **Avec elles**, il suit le même playbook qu'un analyste DFIR senior.

## Anatomie d'une compétence

Chaque compétence suit une structure de répertoire cohérente :

```text
skills/performing-memory-forensics-with-volatility3/
├── SKILL.md ← Définition de la compétence (frontmatter YAML + corps Markdown)
├── references/
│ ├── standards.md ← Mappages MITRE ATT&CK, ATLAS, D3FEND, NIST
│ └── workflows.md ← Référence détaillée de procédure technique
├── scripts/
│ └── process.py ← Scripts d'assistance opérationnels
└── assets/
    └── template.md ← Checklists et modèles de rapport remplis
```

### Frontmatter YAML (exemple réel)

```yaml
---
name: performing-memory-forensics-with-volatility3
description: >-
  Analyser des dumps mémoire pour extraire les processus en cours,
  les connexions réseau, le code injecté et les artefacts malware à l'aide
  du framework Volatility3.
domain: cybersecurity
subdomain: digital-forensics
tags: [forensics, memory-analysis, volatility3, incident-response, dfir]
atlas_techniques: [AML.T0047]
d3fend_techniques: [D3-MA, D3-PSMD]
nist_ai_rmf: [MEASURE-2.6]
nist_csf: [DE.CM-01, RS.AN-03]
version: "1.2"
author: mukul975
license: Apache-2.0
---
```

### Sections du corps Markdown

```md
## When to Use
Conditions de déclenchement — quand un agent IA doit-il activer cette compétence ?

## Prerequisites
Outils requis, niveaux d'accès et configuration d'environnement.

## Workflow
Guide d'exécution pas à pas avec commandes spécifiques et points de décision.

## Verification
Comment confirmer que la compétence a été exécutée avec succès.
```

Les champs du frontmatter : `name` (kebab-case, 1–64 caractères), `description` (riche en mots-clés pour la découverte par agent), `domain`, `subdomain`, `tags`, `atlas_techniques` (IDs MITRE ATLAS), `d3fend_techniques` (IDs MITRE D3FEND), `nist_ai_rmf` (références NIST AI RMF), `nist_csf` (catégories NIST CSF 2.0). Les mappages MITRE ATT&CK sont documentés dans le fichier `references/standards.md` de chaque compétence et dans la couche ATT&CK Navigator incluse dans les releases.

**📊 Couverture MITRE ATT&CK Enterprise — les 15 tactiques**

|Tactic|ID|Coverage|Key skills|
|--|--|--|--|
|Reconnaissance|TA0043|Strong|OSINT, énumération de sous-domaines, reconnaissance DNS|
|Resource Development|TA0042|Moderate|Infrastructure de phishing, détection de mise en place C2|
|Initial Access|TA0001|Strong|Simulation de phishing, détection d'exploit, forced browsing|
|Execution|TA0002|Strong|Analyse PowerShell, malware fileless, script block logging|
|Persistence|TA0003|Strong|Tâches planifiées, registre, comptes de service, LOTL|
|Privilege Escalation|TA0004|Strong|Kerberoasting, attaques AD, élévation de privilèges cloud|
|Stealth|TA0005|Strong|Obfuscation, analyse rootkit, détection d'évasion|
|Defense Impairment|TA0112|Strong|Désactivation EDR, destruction de logs, contournement de défenses|
|Credential Access|TA0006|Strong|Détection Mimikatz, pass-the-hash, credential dumping|
|Discovery|TA0007|Moderate|BloodHound, énumération AD, scan réseau|
|Lateral Movement|TA0008|Strong|Exploits SMB, détection des déplacements latéraux avec Splunk|
|Collection|TA0009|Moderate|Forensics email, détection de data staging|
|Command and Control|TA0011|Strong|Beaconing C2, DNS tunneling, analyse Cobalt Strike|
|Exfiltration|TA0010|Strong|Exfiltration DNS, contrôles DLP, détection de perte de données|
|Impact|TA0040|Strong|Défense ransomware, analyse du chiffrement, récupération|

Un fichier de couche **ATT&CK Navigator** est inclus dans les assets de release v1.0.0 pour la visualisation de la couverture.

**📊 Alignement NIST CSF 2.0 — les 6 fonctions**

|Function|Skills|Examples|
|--|--|--|
|**Govern (GV)**|30+|Stratégie de risque, cadres de politique, rôles et responsabilités|
|**Identify (ID)**|120+|Découverte des actifs, évaluation du paysage de menaces, analyse du risque|
|**Protect (PR)**|150+|Durcissement IAM, règles WAF, zero trust, chiffrement|
|**Detect (DE)**|200+|Threat hunting, corrélation SIEM, détection d'anomalies|
|**Respond (RS)**|160+|Réponse à incident, forensics, confinement d'une compromission|
|**Recover (RC)**|40+|Récupération ransomware, BCP, reprise après sinistre|

NIST CSF 2.0 (février 2024) a ajouté la fonction **Govern** et étendu le périmètre des infrastructures critiques à toutes les organisations. Les mappages des compétences s'alignent sur les 22 catégories et font référence à 106 sous-catégories.

**📊 Analyse approfondie des frameworks — ATLAS, D3FEND, AI RMF**

### MITRE ATLAS v5.4 — Menaces adverses IA/ML

ATLAS mappe les tactiques, techniques et cas d'usage adverses spécifiques aux systèmes d'IA et de machine learning. La version 5.4 couvre **16 tactiques et 84 techniques**, y compris les vecteurs d'attaque agentiques ajoutés fin 2025 : empoisonnement du contexte d'un agent IA, abus d'invocation d'outils, compromission de serveurs MCP, et déploiement d'agents malveillants. Les compétences mappées à ATLAS aident les agents à identifier et contrer les menaces visant les pipelines ML, les poids de modèles, les API d'inférence et les workflows autonomes.

### MITRE D3FEND v1.3 — Contre-mesures défensives

D3FEND est un graphe de connaissances financé par la NSA contenant **267 techniques défensives** organisées en 7 catégories tactiques : Model, Harden, Detect, Isolate, Deceive, Evict et Restore. Construit sur l'ontologie OWL 2, il utilise une couche Digital Artifact partagée pour mapper bidirectionnellement les contre-mesures défensives aux techniques offensives ATT&CK. Les compétences taguées avec des identifiants D3FEND permettent aux agents de recommander des contre-mesures précises pour les menaces détectées.

### NIST AI RMF 1.0 + GenAI Profile (AI 600-1)

Le cadre AI Risk Management Framework définit 4 fonctions centrales — Govern, Map, Measure, Manage — avec **72 sous-catégories** pour un développement IA digne de confiance. Le GenAI Profile (AI 600-1, juillet 2024) ajoute **12 catégories de risque** spécifiques à l'IA générative, de la confabulation et de la confidentialité des données à l'injection de prompt et aux risques de chaîne d'approvisionnement. Le Colorado AI Act (entré en vigueur en février 2026) offre un **safe harbor légal** aux organisations conformes au NIST AI RMF, rendant ces mappages directement pertinents pour la conformité réglementaire.

## Plateformes compatibles

**Assistants de code IA**
Claude Code (Anthropic) · GitHub Copilot (Microsoft) · Cursor · Windsurf · Cline · Aider · Continue · Roo Code · Amazon Q Developer · Tabnine · Sourcegraph Cody · JetBrains AI

**Agents CLI**
OpenAI Codex CLI · Gemini CLI (Google)

**Agents autonomes**
Devin · Replit Agent · SWE-agent · OpenHands

**Frameworks & SDKs d'agents**
LangChain · CrewAI · AutoGen · Semantic Kernel · Haystack · Vercel AI SDK · Tout agent compatible MCP

Toutes les plateformes supportant le standard agentskills.io peuvent charger ces compétences sans configuration.

## Ce qu'ils en disent

> _"Une base de données de vraies compétences de sécurité organisées que n'importe quel agent IA peut brancher et utiliser. Pas des tutoriels. Pas des articles de blog."_
> — **Hasan Toor (@hasantoxr)**, créateur IA/tech

> _"Ce n'est pas une collection aléatoire de scripts de sécurité. C'est une base de connaissances opérationnelle structurée conçue pour les workflows de sécurité pilotés par IA."_
> — **fazal-sec**, Medium

## Mentionné dans

| Où | Type | Lien |
|--|--|--|
| **awesome-agent-skills** | Awesome List (index de 1000+ compétences) | VoltAgent/awesome-agent-skills |
| **awesome-ai-security** | Awesome List (outils de sécurité IA) | ottosulin/awesome-ai-security |
| **awesome-codex-cli** | Awesome List (ressources Codex CLI) | RoggeOhta/awesome-codex-cli |
| **SkillsLLM** | Annuaire & marketplace de compétences | skillsllm.com/skill/anthropic-cybersecurity-skills |
| **Openflows** | Analyse & suivi de signaux | openflows.org |
| **NeverSight skills_feed** | Index automatisé des compétences | NeverSight/skills_feed |

## Historique des étoiles

![Star History Chart](https://star-history.dera.page/svg?repos=mukul975/Anthropic-Cybersecurity-Skills&type=Date)

## Releases

| Version | Date | Points forts |
|--|--|--|
| v1.0.0 | 11 mars 2026 | 734 compétences · 26 domaines · Mappage MITRE ATT&CK + NIST CSF 2.0 · Couche ATT&CK Navigator |

Les compétences continuent de croître sur `main` depuis la v1.0.0 — la bibliothèque contient désormais **817 compétences** avec un **mappage sur 6 frameworks** (MITRE ATLAS, D3FEND, NIST AI RMF, et le MITRE Fight Fraud Framework ajoutés après la release). Consultez les Releases pour la dernière version taguée.

## Contribuer

Ce projet évolue grâce aux contributions de la communauté. Voici comment participer :

**Ajouter une nouvelle compétence** — Les domaines comme Deception Technology (6 compétences) et Hardware & Firmware Security (4 compétences) ont le plus besoin d'aide. Suivez le modèle dans CONTRIBUTING.md et soumettez une PR avec le titre `Add skill: nom-de-votre-competence`.

**Améliorer les compétences existantes** — Ajoutez des mappages de frameworks, corrigez les workflows, mettez à jour les références d'outils, ou contribuez avec des scripts et modèles.

**Signaler des problèmes** — Vous avez trouvé une procédure inexacte ou un script cassé ? Ouvrez une issue.

Chaque PR est revue pour sa précision technique et sa conformité au standard agentskills.io dans les 48 heures. Consultez les *good first issues* pour commencer.

Ce projet suit le Contributor Covenant. En participant, vous acceptez de respecter ce code.

## Communauté

💬 **Discussions** — Questions, idées et conversations sur la feuille de route
🐛 **Issues** — Rapports de bugs et demandes de fonctionnalités
🔒 **Politique de sécurité** — Processus de divulgation responsable (accusé de réception sous 48h)

## Citation

Si vous utilisez ce projet dans des recherches ou publications :

```bibtex
@software{anthropic_cybersecurity_skills,
  author       = {Jangra, Mahipal},
  title        = {Anthropic Cybersecurity Skills},
  year         = {2026},
  url          = {https://github.com/mukul975/Anthropic-Cybersecurity-Skills},
  license      = {Apache-2.0},
  note         = {817 compétences cybersécurité structurées pour agents IA,
                  mappées à MITRE ATT\&CK, NIST CSF 2.0, MITRE ATLAS,
                  MITRE D3FEND et NIST AI RMF}
}
```

## Licence

Ce projet est sous licence Apache License 2.0. Vous êtes libre d'utiliser, modifier et distribuer ces compétences dans des projets personnels comme commerciaux.

---

<div align="center">

**Si ce projet aide votre travail en sécurité, pensez à lui donner une ⭐**

> [⭐ Star](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/stargazers) · [🍴 Fork](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/fork) · [💬 Discuss](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/discussions) · [📝 Contribute](CONTRIBUTING.md~~~

*Projet communautaire par @mukul975. Non affilié à Anthropic PBC.*

</div>

===== END FILE =====


===== FILE: ./README.md =====
<p align="center">
  <img src="assets/banner.png" alt="Anthropic Cybersecurity Skills" width="100%">
</p>

<div align="center">

#  Anthropic Cybersecurity Skills

### The largest open-source cybersecurity skills library for AI agents

[![GARS-2026 Survey](https://img.shields.io/badge/GARS--2026-Take%20the%20Survey-E8B84B?style=for-the-badge&logo=googleforms&logoColor=black)](https://mahipal.engineer/survey?utm_source=github_badge&utm_medium=readme&utm_campaign=gars2026)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg?style=flat-square)](LICENSE)
[![Skills](https://img.shields.io/badge/skills-818-brightgreen?style=flat-square)](#whats-inside--34-security-domains)
[![Frameworks](https://img.shields.io/badge/frameworks-6-orange?style=flat-square)](#six-frameworks-one-skill-library)
[![MITRE F3](https://img.shields.io/badge/MITRE-F3_v1.1-blue?style=flat-square)](https://ctid.mitre.org/fraud/)
[![Domains](https://img.shields.io/badge/domains-34-9cf?style=flat-square)](#whats-inside--34-security-domains)
[![Platforms](https://img.shields.io/badge/platforms-26%2B-blueviolet?style=flat-square)](#compatible-platforms)
[![GitHub stars](https://img.shields.io/github/stars/mukul975/Anthropic-Cybersecurity-Skills?style=flat-square)](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/mukul975/Anthropic-Cybersecurity-Skills?style=flat-square)](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/network/members)
[![Last Commit](https://img.shields.io/github/last-commit/mukul975/Anthropic-Cybersecurity-Skills?style=flat-square)](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/commits/main)
[![agentskills.io](https://img.shields.io/badge/standard-agentskills.io-ff6600?style=flat-square)](https://agentskills.io)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](CONTRIBUTING.md)
[![Playground](https://img.shields.io/badge/Playground-Casky.ai-blue)](https://casky.ai/?utm_source=github&utm_medium=readme&utm_campaign=cohort_launch#waitlist)
[![Hermes Agent](https://img.shields.io/badge/Hermes_Agent-compatible-blueviolet?style=flat)](https://github.com/NousResearch/hermes-agent)


**818 production-grade cybersecurity skills · 34 security domains · 6 framework mappings · 26+ AI platforms**

[Get Started](#quick-start) · [What's Inside](#whats-inside--34-security-domains) · [Frameworks](#six-frameworks-one-skill-library) · [Platforms](#compatible-platforms) · [Contributing](#contributing)

</div>

---

> ⚠️ **Community Project** — This is an independent, community-created project. Not affiliated with Anthropic PBC.
>
> 🔐 **Authorized & lawful use only.** This library includes offensive and dual-use techniques (e.g. red-team C2, phishing simulation, exploitation) intended for **authorized penetration testing, security research, defense, and education**. Only use them against systems you own or have **explicit written permission** to test, and comply with all applicable laws and rules of engagement. You are solely responsible for how you use these skills. See [SECURITY.md](SECURITY.md) and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## Give any AI agent the security skills of a senior analyst

A junior analyst knows which Volatility3 plugin to run on a suspicious memory dump, which Sigma rules catch Kerberoasting, and how to scope a cloud breach across three providers. **Your AI agent doesn't — unless you give it these skills.**

This repo contains **818 structured cybersecurity skills** spanning **34 security domains**, each following the [agentskills.io](https://agentskills.io) open standard.  The library maps across **six industry frameworks** — MITRE ATT&CK, NIST CSF 2.0, MITRE ATLAS, MITRE D3FEND, NIST AI RMF, and the MITRE Fight Fraud Framework (F3) — with each skill mapped to the frameworks **relevant to its type** (a forensics skill carries ATT&CK + CSF; an AI-security skill adds ATLAS and AI RMF).  Clone it, point your agent at it, and your next security investigation gets expert-level guidance in seconds.

## Six frameworks, one skill library

Each skill maps to the frameworks that fit its subject — ATT&CK and NIST CSF are near-universal, while ATLAS, AI RMF, D3FEND, and F3 apply where they're relevant. **Framework coverage across the 817 skills:** MITRE ATT&CK **805** · NIST CSF 2.0 **804** · MITRE D3FEND **139** · NIST AI RMF **97** · MITRE F3 **94** · MITRE ATLAS **93**.

| Framework | Version | Framework scope | What it maps |
|---|---|---|---|
| [MITRE ATT&CK](https://attack.mitre.org) | v19.1 | 15 tactics · Enterprise/Mobile/ICS | Adversary behaviors and TTPs |
| [NIST CSF 2.0](https://www.nist.gov/cyberframework) | 2.0 | 6 functions · 22 categories · 106 subcategories | Organizational security posture |
| [MITRE ATLAS](https://atlas.mitre.org) | 2026.07 | 101 techniques · 77 sub-techniques | AI/ML adversarial threats |
| [MITRE D3FEND](https://d3fend.mitre.org) | v1.4.0 | 270 techniques | Defensive countermeasures |
| [NIST AI RMF](https://airc.nist.gov/AI_RMF) | 1.0 | 4 functions (Govern/Map/Measure/Manage) | AI risk management |
| [MITRE F3 (Fight Fraud Framework)](https://ctid.mitre.org/fraud/) | v1.1 (2026-04-09) | 8 tactics · 123 techniques · 94 fraud-relevant skills | Cyber-enabled financial fraud TTPs |

**Example — each skill maps only to the frameworks relevant to it (one may hit all six, another just a couple):**

| Skill | ATT&CK | NIST CSF | ATLAS | D3FEND | AI RMF | F3 |
|---|---|---|---|---|---|---|
| `analyzing-network-traffic-of-malware` | T1071 | DE.CM | AML.T0047 | D3-NTA | MEASURE-2.6 | — |
| `detecting-business-email-compromise` | T1566 | DE.AE | — | — | — | F1005.006 · monetization |

### 🆕 MITRE Fight Fraud Framework (F3) — 94 fraud-relevant skills

[![MITRE F3](https://img.shields.io/badge/MITRE-F3_v1.1-blue?style=flat-square)](https://ctid.mitre.org/fraud/)

The **[MITRE Fight Fraud Framework (F3)](https://ctid.mitre.org/fraud/)** was released **April 9, 2026** by MITRE's Center for Threat-Informed Defense (CTID), co-developed with JPMorganChase, Citigroup, Lloyds Banking Group, Standard Chartered, CrowdStrike, Verizon Business, FS-ISAC, and others. It is an ATT&CK-compatible TTP catalog for **cyber-enabled financial fraud** — filling the gap ATT&CK leaves after initial compromise.

F3 v1.1 adds **two fraud-specific tactics** that ATT&CK does not enumerate:
- **Positioning** (`FA0001`) — actions taken after access to collect/manipulate data and prepare the fraud (synthetic-identity seeding, account warming, beneficiary setup, SIM-swap pre-positioning, banking-session hijack).
- **Monetization** (`FA0002`) — converting stolen assets into usable funds (money-mule layering, APP fraud, crypto off-ramping, card cash-out, refund/chargeback abuse).

Fraud-specific techniques use `F1XXX` IDs (e.g. `F1005.003` Add Beneficiary, `F1025.003` Wire Transfer, `F1007` Adversary-in-the-Browser); reused ATT&CK techniques keep their `T1XXX` IDs. Mappings live in each skill's `mitre_f3:` frontmatter block — all 123 F3 v1.1 technique IDs were verified against the upstream STIX bundle. See [`docs/mitre-f3-mapping.md`](docs/mitre-f3-mapping.md) for the schema.

### MITRE ATT&CK v19.1 — 805/817 skills mapped

Every skill carries a `mitre_attack` frontmatter list validated against **MITRE ATT&CK v19.1** (the latest release) using the official `mitreattack-python` library — 290 distinct techniques and sub-techniques (146 base + 144 sub) across Enterprise, ICS, and Mobile. Zero revoked or deprecated IDs. v19.1's restructured Defense Evasion (now split into **Stealth** and **Defense Impairment**) is reflected below.

| Tactic | ID | Skills |
|--------|----|--------|
| Reconnaissance | TA0043 | 103 |
| Resource Development | TA0042 | 22 |
| Initial Access | TA0001 | 467 |
| Execution | TA0002 | 350 |
| Persistence | TA0003 | 444 |
| Privilege Escalation | TA0004 | 464 |
| Stealth | TA0005 | 442 |
| Defense Impairment | TA0112 | 92 |
| Credential Access | TA0006 | 202 |
| Discovery | TA0007 | 237 |
| Lateral Movement | TA0008 | 68 |
| Collection | TA0009 | 172 |
| Command and Control | TA0011 | 123 |
| Exfiltration | TA0010 | 82 |
| Impact | TA0040 | 50 |

## Quick start

```bash
# Option 1: npx (recommended)
npx skills add mukul975/Anthropic-Cybersecurity-Skills

# Option 2: Git clone
git clone https://github.com/mukul975/Anthropic-Cybersecurity-Skills.git
cd Anthropic-Cybersecurity-Skills
```

Works immediately with Claude Code, GitHub Copilot, OpenAI Codex CLI, Cursor, Gemini CLI, and any [agentskills.io](https://agentskills.io)-compatible platform. 

## 🌍 GARS-2026 — Global Agentic AI Readiness Survey

I'm running a global academic study measuring how ready security professionals,
developers, and enterprise teams actually are for agentic AI — MCP servers,
tool calling, governance, and human-in-the-loop workflows.

**If you use this repo, your response would be a genuinely valuable data point.**

📋 **Take the survey (10 min):**
[Survey Link](https://mahipal.engineer/survey?utm_source=github_repo&utm_medium=readme&utm_campaign=gars2026)

- 60 questions · Anonymous · Supervised by SRH Berlin
- You get **50 Casky Tokens** for early access to [casky.ai](https://casky.ai)
- Results published open access under CC-BY 4.0

## 🚀 Try it on the Playground

Experience Casky.ai hands-on — no setup required.

**[→ Launch Playground on Casky.ai](https://casky.ai/?utm_source=github&utm_medium=readme&utm_campaign=cohort_launch#waitlist)**

The playground lets you:
- Run live cybersecurity skill exercises against real targets
- See AI agents execute structured skills in real time
- Explore MITRE ATT&CK mapped workflows interactively
- Test threat hunting, DFIR, and penetration testing scenarios

No installation. No configuration. Just open and start.
## Why this exists

The cybersecurity workforce gap hit **4.8 million unfilled roles** globally in 2024 (ISC2). AI agents can help close that gap — but only if they have structured domain knowledge to work from. Today's agents can write code and search the web, but they lack the practitioner playbooks that turn a generic LLM into a capable security analyst.

Existing security tool repos give you wordlists, payloads, or exploit code. None of them give an AI agent the structured decision-making workflow a senior analyst follows: when to use each technique, what prerequisites to check, how to execute step-by-step, and how to verify results. That is the gap this project fills.

**Anthropic Cybersecurity Skills** is not a collection of scripts or checklists. It is an **AI-native knowledge base** built from the ground up for the agentskills.io standard  — YAML frontmatter for sub-second discovery, structured Markdown for step-by-step execution, and reference files for deep technical context.  Every skill encodes real practitioner workflows, not generated summaries. 

## What's inside — 34 security domains

| Domain | Skills | Key capabilities |
|---|---|---|
| Cloud Security | 66 | AWS, Azure, GCP hardening · CSPM · cloud attack emulation · cloud forensics |
| SOC Operations | 63 | Playbooks · escalation workflows · Graph-log detection · tabletop exercises |
| Threat Hunting | 58 | Hypothesis-driven hunts · LOTL detection · EVTX hunting · fleet hunting |
| Threat Intelligence | 52 | STIX/TAXII · MISP · OpenCTI · feed integration · actor profiling |
| Web Application Security | 46 | OWASP Top 10 · SQLi · XSS · SSRF · deserialization |
| Network Security | 43 | IDS/IPS · firewall rules · VLAN segmentation · traffic analysis |
| Digital Forensics | 41 | Disk imaging · memory forensics · Hayabusa/KAPE/Plaso timelines |
| Identity & Access Management | 40 | Entra ID/ROADtools · device-code phishing · PAM · zero trust identity |
| Malware Analysis | 39 | Static/dynamic analysis · reverse engineering · sandboxing |
| Red Teaming | 35 | ADCS/Certipy · BloodHound CE · Sliver/Havoc C2 · NTLM relay |
| Container Security | 33 | K8s RBAC · image scanning · Falco · container escape |
| OT/ICS Security | 29 | Modbus · DNP3 · IEC 62443 · historian defense · SCADA |
| API Security | 28 | GraphQL · REST · OWASP API Top 10 · WAF bypass |
| Incident Response | 26 | Breach containment · ransomware response · IR playbooks |
| Vulnerability Management | 25 | Nessus · scanning workflows · patch prioritization · CVSS |
| Penetration Testing | 23 | Network · web · cloud · mobile · NetExec lateral movement |
| DevSecOps | 18 | CI/CD security · Trivy IaC/image scanning · code signing |
| Zero Trust Architecture | 18 | BeyondCorp · CISA maturity model · microsegmentation |
| Endpoint Security | 17 | EDR · LOTL detection · fileless malware · persistence hunting |
| Phishing Defense | 16 | Email authentication · BEC detection · phishing IR |
| Cryptography | 16 | TLS · Ed25519 · post-quantum migration · key management |
| AI Security | 14 | LLM red-teaming (garak/PyRIT) · prompt injection · MCP/agentic security · guardrails |
| Mobile Security | 13 | Android/iOS analysis · mobile pentesting · MDM forensics |
| Ransomware Defense | 13 | Precursor detection · response · recovery · encryption analysis |
| Compliance & Governance | 10 | NIST 800-30/RMF · CMMC · HIPAA · TPRM · CIS benchmarks |
| Supply Chain Security | 8 | SBOMs · dependency confusion · malicious-package triage · SLSA/Sigstore |
| Threat Detection | 7 | Credential dumping · golden-ticket forgery · pass-the-ticket · LOLBAS · UEBA insider signals |
| Hardware & Firmware Security | 6 | CHIPSEC/UEFI audit · Secure Boot bypass · TPM attestation · bootkit hunting |
| Deception Technology | 6 | Honeytokens · canarytokens · breach detection |
| Blockchain Security | 2 | Ethereum smart-contract vulnerabilities · Foundry audit workflows |
| Wireless Security | 2 | Bluetooth Low Energy attack detection · BLE security assessment |
| Privacy Compliance | 2 | GDPR data-subject access requests · privacy impact assessments |
| Data Protection | 1 | Data loss prevention with Microsoft Purview |
| Purple Team | 1 | Atomic Red Team purple-team testing |

*817 skills across 34 domains. Counts come from the `subdomain` field in each skill's frontmatter.*

## How AI agents use these skills

Each skill costs **~30 tokens to scan** (frontmatter only)  and **500–2,000 tokens to fully load** (complete workflow). This progressive disclosure architecture lets agents search all 818 skills in a single pass without blowing context windows. 

```
User prompt: "Analyze this memory dump for signs of credential theft"

Agent's internal process:

  1. Scans 818 skill frontmatters (~30 tokens each)
     → identifies 12 relevant skills by matching tags, description, domain

  2. Loads top 3 matches:
     • performing-memory-forensics-with-volatility3
     • hunting-for-credential-dumping-lsass
     • analyzing-windows-event-logs-for-credential-access

  3. Executes the structured Workflow section step-by-step
     → runs Volatility3 plugins, checks LSASS access patterns,
        correlates with event log evidence

  4. Validates results using the Verification section
     → confirms IOCs, maps findings to ATT&CK T1003 (Credential Dumping)
```

**Without these skills**, the agent guesses at tool commands and misses critical steps. **With them**, it follows the same playbook a senior DFIR analyst would use. 

## Skill anatomy

Every skill follows a consistent directory structure:

```
skills/performing-memory-forensics-with-volatility3/
├── SKILL.md              ← Skill definition (YAML frontmatter + Markdown body)
├── references/
│   ├── standards.md      ← MITRE ATT&CK, ATLAS, D3FEND, NIST mappings
│   └── workflows.md      ← Deep technical procedure reference
├── scripts/
│   └── process.py        ← Working helper scripts
└── assets/
    └── template.md       ← Filled-in checklists and report templates
```


### YAML frontmatter (real example)

```yaml
---
name: performing-memory-forensics-with-volatility3
description: >-
  Analyze memory dumps to extract running processes, network connections,
  injected code, and malware artifacts using the Volatility3 framework.
domain: cybersecurity
subdomain: digital-forensics
tags: [forensics, memory-analysis, volatility3, incident-response, dfir]
atlas_techniques: [AML.T0047]
d3fend_techniques: [D3-MA, D3-PSMD]
nist_ai_rmf: [MEASURE-2.6]
nist_csf: [DE.CM-01, RS.AN-03]
version: "1.2"
author: mukul975
license: Apache-2.0
---
```


### Markdown body sections

```markdown
## When to Use
Trigger conditions — when should an AI agent activate this skill?

## Prerequisites
Required tools, access levels, and environment setup.

## Workflow
Step-by-step execution guide with specific commands and decision points.

## Verification
How to confirm the skill was executed successfully.
```

Frontmatter fields: `name` (kebab-case, 1–64 chars), `description` (keyword-rich for agent discovery), `domain`, `subdomain`, `tags`,  `atlas_techniques` (MITRE ATLAS IDs), `d3fend_techniques` (MITRE D3FEND IDs), `nist_ai_rmf` (NIST AI RMF references), `nist_csf` (NIST CSF 2.0 categories).  MITRE ATT&CK technique mappings are documented in each skill's `references/standards.md` file and in the ATT&CK Navigator layer included with releases. 

<details>
<summary><strong>📊 MITRE ATT&CK Enterprise coverage — all 15 tactics</strong></summary>

&nbsp;

| Tactic | ID | Coverage | Key skills |
|---|---|---|---|
| Reconnaissance | TA0043 | Strong | OSINT, subdomain enumeration, DNS recon |
| Resource Development | TA0042 | Moderate | Phishing infrastructure, C2 setup detection |
| Initial Access | TA0001 | Strong | Phishing simulation, exploit detection, forced browsing |
| Execution | TA0002 | Strong | PowerShell analysis, fileless malware, script block logging |
| Persistence | TA0003 | Strong | Scheduled tasks, registry, service accounts, LOTL |
| Privilege Escalation | TA0004 | Strong | Kerberoasting, AD attacks, cloud privilege escalation |
| Stealth | TA0005 | Strong | Obfuscation, rootkit analysis, evasion detection |
| Defense Impairment | TA0112 | Moderate | Impair Defenses (T1562), log/indicator removal, EDR tampering |
| Credential Access | TA0006 | Strong | Mimikatz detection, pass-the-hash, credential dumping |
| Discovery | TA0007 | Moderate | BloodHound, AD enumeration, network scanning |
| Lateral Movement | TA0008 | Strong | SMB exploits, lateral movement detection with Splunk |
| Collection | TA0009 | Moderate | Email forensics, data staging detection |
| Command and Control | TA0011 | Strong | C2 beaconing, DNS tunneling, Cobalt Strike analysis |
| Exfiltration | TA0010 | Strong | DNS exfiltration, DLP controls, data loss detection |
| Impact | TA0040 | Strong | Ransomware defense, encryption analysis, recovery |

An **ATT&CK Navigator layer file** is included in the [v1.0.0 release assets](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/releases/tag/v1.0.0) for visual coverage mapping. 

> **Note:** ATT&CK v19 lands April 28, 2026 — splitting Defense Evasion (TA0005) into two new tactics: *Stealth* and *Impair Defenses*.  Skill mappings will be updated in a forthcoming release.

</details>

<details>
<summary><strong>📊 NIST CSF 2.0 alignment — all 6 functions</strong></summary>

&nbsp;

| Function | Skills | Examples |
|---|---|---|
| **Govern (GV)** | 30+ | Risk strategy, policy frameworks, roles & responsibilities |
| **Identify (ID)** | 120+ | Asset discovery, threat landscape assessment, risk analysis |
| **Protect (PR)** | 150+ | IAM hardening, WAF rules, zero trust, encryption |
| **Detect (DE)** | 200+ | Threat hunting, SIEM correlation, anomaly detection |
| **Respond (RS)** | 160+ | Incident response, forensics, breach containment |
| **Recover (RC)** | 40+ | Ransomware recovery, BCP, disaster recovery |

NIST CSF 2.0 (February 2024) added the **Govern** function  and expanded scope from critical infrastructure to all organizations.  Skill mappings align to all 22 categories and reference 106 subcategories. 

</details>

<details>
<summary><strong>📊 Framework deep dive — ATLAS, D3FEND, AI RMF</strong></summary>

&nbsp;

### MITRE ATLAS 2026.07 — AI/ML adversarial threats
ATLAS maps adversarial tactics, techniques, and case studies specific to AI and machine learning systems. Release 2026.07 covers **101 techniques and 77 sub-techniques** including agentic AI attack vectors added in late 2025: AI agent context poisoning, tool invocation abuse, MCP server compromises, and malicious agent deployment.  Skills mapped to ATLAS help agents identify and defend against threats to ML pipelines, model weights, inference APIs, and autonomous workflows. 

### MITRE D3FEND v1.4.0 — Defensive countermeasures
D3FEND is an NSA-funded knowledge graph of **270 defensive techniques** organized across 7 tactical categories: Model, Harden, Detect, Isolate, Deceive, Evict, and Restore.  Built on OWL 2 ontology, it uses a shared Digital Artifact layer to bidirectionally map defensive countermeasures to ATT&CK offensive techniques.  Skills tagged with D3FEND identifiers let agents recommend specific countermeasures for detected threats.

### NIST AI RMF 1.0 + GenAI Profile (AI 600-1)
The AI Risk Management Framework defines 4 core functions — Govern, Map, Measure, Manage — with **72 subcategories** for trustworthy AI development.  The GenAI Profile (AI 600-1, July 2024) adds **12 risk categories** specific to generative AI, from confabulation and data privacy to prompt injection and supply chain risks.  Colorado's AI Act (effective February 2026) provides a **legal safe harbor** for organizations complying with NIST AI RMF, making these mappings directly relevant to regulatory compliance.

</details>

## Compatible platforms

**AI code assistants**
Claude Code (Anthropic) · GitHub Copilot (Microsoft) · Cursor · Windsurf · Cline · Aider · Continue · Roo Code · Amazon Q Developer · Tabnine · Sourcegraph Cody · JetBrains AI 

**CLI agents**
OpenAI Codex CLI · Gemini CLI (Google) 

**Autonomous agents**
Devin · Replit Agent · SWE-agent · OpenHands 

**Agent frameworks & SDKs**
LangChain · CrewAI · AutoGen · Semantic Kernel · Haystack · Vercel AI SDK · Any MCP-compatible agent 

All platforms that support the [agentskills.io](https://agentskills.io) standard can load these skills with zero configuration. 

## What people are saying

> *"A database of real, organized security skills that any AI agent can plug into and use. Not tutorials. Not blog posts."* 
> — **[Hasan Toor (@hasantoxr)](https://x.com/hasantoxr/status/2033193922349179249)**, AI/tech creator

> *"This is not a random collection of security scripts. It's a structured operational knowledge base designed for AI-driven security workflows."* 
> — **[fazal-sec](https://fazal-sec.medium.com/claude-skills-ai-powered-cybersecurity-the-complete-guide-to-building-intelligent-security-7bb7e9d14c8e)**,  Medium

## Featured in

| Where | Type | Link |
|---|---|---|
| **awesome-agent-skills** | Awesome List (1,000+ skills index) | [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) |
| **awesome-ai-security** | Awesome List (AI security tools) | [ottosulin/awesome-ai-security](https://github.com/ottosulin/awesome-ai-security) |
| **awesome-codex-cli** | Awesome List (Codex CLI resources) | [RoggeOhta/awesome-codex-cli](https://github.com/RoggeOhta/awesome-codex-cli) |
| **SkillsLLM** | Skills directory & marketplace | [skillsllm.com/skill/anthropic-cybersecurity-skills](https://skillsllm.com/skill/anthropic-cybersecurity-skills) |
| **Openflows** | Signal analysis & tracking | [openflows.org](https://openflows.org/currency/currents/anthropic-cybersecurity-skills/) |
| **NeverSight skills_feed** | Automated skills index | [NeverSight/skills_feed](https://github.com/NeverSight/skills_feed) |

## Star history

<a href="https://star-history.dera.page/#mukul975/Anthropic-Cybersecurity-Skills&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://star-history.dera.page/svg?repos=mukul975/Anthropic-Cybersecurity-Skills&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://star-history.dera.page/svg?repos=mukul975/Anthropic-Cybersecurity-Skills&type=Date" />
   <img alt="Star History Chart" src="https://star-history.dera.page/svg?repos=mukul975/Anthropic-Cybersecurity-Skills&type=Date" width="100%" />
 </picture>
</a>

## Releases

| Version | Date | Highlights |
|---|---|---|
| [v1.0.0](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/releases/tag/v1.0.0) | March 11, 2026 | 734 skills · 26 domains · MITRE ATT&CK + NIST CSF 2.0 mapping · ATT&CK Navigator layer |

Skills have continued to grow on `main` since v1.0.0 — the library now contains **818 skills** with **6-framework mapping**  (MITRE ATLAS, D3FEND, NIST AI RMF, and the MITRE Fight Fraud Framework added post-release).  Check [Releases](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/releases) for the latest tagged version.

## Contributing

This project grows through community contributions. Here is how to get involved:

**Add a new skill** — Read [SCOPE.md](SCOPE.md) first, then the template in [CONTRIBUTING.md](CONTRIBUTING.md). The thinnest domains are the ones most worth adding to: Data Protection and Purple Team have one skill each, and Blockchain Security, Wireless Security and Privacy Compliance have two. Submit one skill per PR, with the title `Add skill: your-skill-name`.

**Improve existing skills** — Add framework mappings, fix workflows, update tool references, or contribute scripts and templates. Deepening a skill that already exists is worth as much to this library as adding a new one.

**Report issues** — Found an inaccurate procedure or broken script? [Open an issue](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/issues).

Every PR is reviewed for technical accuracy and agentskills.io standard compliance. I do that review myself, and the queue is currently longer than I would like — some pull requests have been open for months. I am working through them, and small, focused PRs move fastest. Check [good first issues](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) for a starting point.

This project follows the [Contributor Covenant](https://www.contributor-covenant.org/). By participating, you agree to uphold this code.

## 🙏 Thanks to our contributors

This library is built by the community. Thank you to everyone who has contributed:

<!-- contributors:start -->
<p align="center">
<a href="https://github.com/mukul975" title="mukul975 — maintainer"><img src="https://github.com/mukul975.png?size=100" width="72" height="72" alt="@mukul975"></a>
<a href="https://github.com/valorisa" title="valorisa — 18 contributions"><img src="https://github.com/valorisa.png?size=100" width="72" height="72" alt="@valorisa"></a>
<a href="https://github.com/juliosuas" title="juliosuas — 13 contributions"><img src="https://github.com/juliosuas.png?size=100" width="72" height="72" alt="@juliosuas"></a>
<a href="https://github.com/Daytona39264" title="Daytona39264 — 3 contributions"><img src="https://github.com/Daytona39264.png?size=100" width="72" height="72" alt="@Daytona39264"></a>
<a href="https://github.com/kevglynn" title="kevglynn — 2 contributions"><img src="https://github.com/kevglynn.png?size=100" width="72" height="72" alt="@kevglynn"></a>
<a href="https://github.com/andrewibrah" title="andrewibrah — 2 contributions"><img src="https://github.com/andrewibrah.png?size=100" width="72" height="72" alt="@andrewibrah"></a>
<a href="https://github.com/Bortlesboat" title="Bortlesboat — 1 contribution"><img src="https://github.com/Bortlesboat.png?size=100" width="72" height="72" alt="@Bortlesboat"></a>
<a href="https://github.com/DevRedious" title="DevRedious — 1 contribution"><img src="https://github.com/DevRedious.png?size=100" width="72" height="72" alt="@DevRedious"></a>
<a href="https://github.com/ioxoi" title="ioxoi — 1 contribution"><img src="https://github.com/ioxoi.png?size=100" width="72" height="72" alt="@ioxoi"></a>
<a href="https://github.com/OctoBored" title="OctoBored — 1 contribution"><img src="https://github.com/OctoBored.png?size=100" width="72" height="72" alt="@OctoBored"></a>
<a href="https://github.com/shanujans" title="shanujans — 1 contribution"><img src="https://github.com/shanujans.png?size=100" width="72" height="72" alt="@shanujans"></a>
<a href="https://github.com/farhan6667" title="farhan6667 — 1 contribution"><img src="https://github.com/farhan6667.png?size=100" width="72" height="72" alt="@farhan6667"></a>
<a href="https://github.com/dakshverma23" title="dakshverma23 — 1 contribution"><img src="https://github.com/dakshverma23.png?size=100" width="72" height="72" alt="@dakshverma23"></a>
<a href="https://github.com/nyxst4ck" title="nyxst4ck — 1 contribution"><img src="https://github.com/nyxst4ck.png?size=100" width="72" height="72" alt="@nyxst4ck"></a>
</p>

<p align="center"><sub>14 contributors, ordered by contribution count · see the full <a href="https://github.com/mukul975/Anthropic-Cybersecurity-Skills/graphs/contributors">contributor graph</a></sub></p>
<!-- contributors:end -->

## Community

💬 [Discussions](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/discussions) — Questions, ideas, and roadmap conversations
🐛 [Issues](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/issues) — Bug reports and feature requests
🔒 [Security Policy](SECURITY.md) — Responsible disclosure process (48-hour acknowledgment) 

## Citation

If you use this project in research or publications:

```bibtex
@software{anthropic_cybersecurity_skills,
  author       = {Jangra, Mahipal},
  title        = {Anthropic Cybersecurity Skills},
  year         = {2026},
  url          = {https://github.com/mukul975/Anthropic-Cybersecurity-Skills},
  license      = {Apache-2.0},
  note         = {818 structured cybersecurity skills for AI agents,
                  mapped to MITRE ATT\&CK, NIST CSF 2.0, MITRE ATLAS,
                  MITRE D3FEND, and NIST AI RMF}
}
```

## License

This project is licensed under the [Apache License 2.0](LICENSE). You are free to use, modify, and distribute these skills in both personal and commercial projects. 

---

<div align="center">

**If this project helps your security work, consider giving it a ⭐**

[⭐ Star](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/stargazers) · [🍴 Fork](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/fork) · [💬 Discuss](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/discussions) · [📝 Contribute](CONTRIBUTING.md)

Community project by [@mukul975](https://github.com/mukul975). Not affiliated with Anthropic PBC.

</div>

===== END FILE =====


===== FILE: ./SCOPE.md =====
# Scope

This document defines what belongs in this repository and what does not.

It exists so the answer is available *before* the work is done rather than after it. Several of the pull requests I have had to decline were good work aimed at the wrong repository, and that is a documentation failure on my side, not a failure on the contributor's.

This repository is a catalogue. It holds cybersecurity procedures — 817 of them today — written so that an AI agent can read one and carry out a security task competently. Everything that ships here serves that one purpose. Work that does not serve it is out of scope regardless of how good it is, and is usually better off in its own repository, where it can be versioned, released and credited on its own terms.

## These rules apply going forward

This document is new. Until now the repository had no written scope, which is exactly why some contributors built things I then could not merge.

So: nothing here is applied retroactively as a reason to close a pull request that was already open when it landed. Every open PR will be worked through as it was filed, on its own merits, and where I decline one I will give the reason in the thread rather than pointing at a rule that did not exist when the work was done.

I should also say plainly that the review queue is longer than it should be — some pull requests have been waiting months. That is on me, not on the people who sent them, and raising the bar on new contributions does not excuse it. I am working through the backlog.

## What a skill is

A skill is one named security procedure that an agent can execute end to end.

On disk it is a single flat directory:

```
skills/your-skill-name/
├── SKILL.md          # YAML frontmatter + the procedure
├── LICENSE
├── references/       # api-reference.md, standards.md, workflows.md
├── scripts/          # agent.py, process.py, or another real helper
└── assets/           # optional: template.md, filled-in checklists
```

The namespace is flat and global. `skills/` is not nested by category; the category lives in the `subdomain` frontmatter field, which must be one of the values accepted by `tools/validate-skill.py`. All 817 skills today carry `SKILL.md`, `references/` and `scripts/`; 816 carry a `LICENSE` and 421 also carry `assets/`. The most common supporting files are `references/api-reference.md` (810 skills) and `scripts/agent.py` (809).

A submission is a skill when all of the following hold.

**It is a procedure, not a subject.** By convention the directory name is a gerund phrase naming the task — `analyzing-…`, `detecting-…`, `implementing-…`, `hunting-…`, `performing-…` — and 807 of the 817 current names follow it. CI enforces kebab-case, not the gerund, so treat this as the house style rather than a gate. Background material about a topic belongs in a skill's `references/`, not in a directory of its own.

**It is atomic.** One procedure per skill. A body carrying three unrelated workflows is either three skills, or one skill and two reference files.

**It is executable.** Real commands, real flags, real tool names, real paths. No placeholders, no `TODO`, no prose standing in for a command nobody ran.

**Its description routes.** The description is the only text an agent sees when deciding whether to load the skill. It has to say what the skill does, when to fire, which keywords match, and what it is *not* for — naming the nearest neighbouring skill. The 33 container-security skills are the current reference for that standard.

**It is distinct.** It does not restate a skill that already exists. See *Overlap* in [CONTRIBUTING.md](CONTRIBUTING.md).

**It passes CI.** Five gates run on every push and pull request that touches `skills/` or `tools/`: frontmatter validation, agentskills.io conformance, `index.json` freshness, description linting, and a near-duplicate ratchet. A documentation-only PR does not trigger them, so do not read an empty checks list as a pass. CONTRIBUTING.md lists the commands to run them locally.

## Offensive and dual-use content is in scope

This library deliberately covers red-team tradecraft, exploitation, C2, phishing simulation and adversary emulation alongside defence. A defender's agent that has never seen the offensive procedure detects it badly.

Offensive skills are in scope on exactly the same terms as everything else, subject to the authorised-use notice in [README.md](README.md). Nothing in this document is a reason to decline a skill for being offensive. If I decline one, the reason will be scope, overlap or accuracy, and I will say which.

## Out of scope

**Runtimes, engines, orchestrators and agent frameworks.** Code that loads, serves, routes or executes skills — MCP servers, agent harnesses, skill-browser web UIs, "foundation" or "platform" layers that turn the catalogue into a product.

**Applications built on the catalogue.** A tool that consumes these skills is a good thing to build and I will happily link it from the README. It is not a directory in this repository.

**A second toolchain.** `tools/` already holds a frontmatter validator, an agentskills.io conformance checker, an index generator, a description linter and a collision detector — all sharing one PyYAML-backed loader, all wired into CI. A new independent validator competes with those rather than improving them. Extend the existing tool and the existing gate; a PR that makes `tools/lint-descriptions.py` stricter is far more welcome than a new linter beside it.

**Documents about how the project is run.** Roadmaps, work cadences, routines, release plans, launch material, status trackers. How I schedule my own maintenance is not something the catalogue needs to carry, and a document describing a cadence I have not agreed to would be wrong the day it merged. Propose process changes in an issue or a discussion instead.

**Editor, IDE and per-contributor configuration.** `.vscode/` settings, local linter configs and vendor extension settings encode one contributor's environment on everyone else. Agent instruction files are the single exception and they are maintainer-owned — `.github/copilot-instructions.md` exists and is maintained in place. Changes to that class of file should start as an issue, not as a new parallel file.

**Product-shaped skills.** A skill may absolutely be tool-specific: 247 of the 817 names carry a `-with-<tool>` qualifier, and `scanning-docker-images-with-trivy` is exactly right, because the flags, the output formats and the failure modes genuinely differ by tool. What does not belong is a directory whose *subject* is a product rather than a task — a feature tour, an onboarding walkthrough, or an integration write-up for one vendor's service. The test: if the vendor vanished tomorrow and nothing of the procedure survived, it was a product page. When a skill does name a commercial tool, the vendor-link rules in CONTRIBUTING.md apply.

**Bulk imports.** A pull request adding many skills at once cannot be reviewed for technical accuracy at the depth this catalogue needs, and one wrong procedure blocks all the others. One skill per pull request; see CONTRIBUTING.md.

## Why there is no engine in this repository

The catalogue is engine-neutral by construction. `skills/` contains content only; `tools/` contains validators and the index generator and nothing that executes a skill; `.claude-plugin/` contains manifests that let an existing engine mount the catalogue rather than an engine of its own. The bug-report template asks which agent you were running, offering Claude Code, GitHub Copilot and Codex CLI as examples, and the README targets 26+ platforms.

That neutrality is the asset. The moment a runtime lives in this repository, the content starts being shaped by what that runtime supports, the release cadence of the content gets tied to the release cadence of the code, and every consumer on a different platform inherits a dependency they did not ask for. I want the content here to stay separate from the engine that reads it, and I intend to keep this repository on that side of the line.

## Grey areas

Some things sit on the boundary — a large rewrite of an existing skill, a new subdomain, a reference file that is really a small tool, a documentation page that is not README and not CONTRIBUTING. Open an issue and ask before you build it. I would much rather answer a two-line issue than decline a finished pull request, and if the answer is yes, the issue becomes the record of why.

## Changing this document

This is a working document, not a settlement. If a rule here is wrong, or is blocking something the catalogue would clearly be better for having, open an issue arguing the case. Scope decisions are mine to make, but they should be arguable in the open.

===== END FILE =====


===== FILE: ./SECURITY.md =====
# Security Policy

## Supported Versions

All skill content in this repository is covered by this security policy.

| Component | Supported |
|-----------|-----------|
| Skill definitions (SKILL.md files) | Yes |
| Scripts and automation | Yes |
| Documentation | Yes |

## Reporting a Vulnerability

If you discover a security issue with any skill's scripts, instructions, or content, please report it responsibly:

1. **Do not** open a public issue
2. Use GitHub's private security advisory: [Report a vulnerability](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/security/advisories/new)
3. Include in your report:
   - Affected skill name and file path
   - Nature of the vulnerability
   - Potential impact
   - Steps to reproduce (if applicable)
   - Suggested fix (if you have one)

## Response Timeline

- **Initial acknowledgment:** Within 48 hours
- **Assessment and triage:** Within 1 week
- **Fix or mitigation:** Based on severity, typically within 2 weeks

## Scope

The following are in scope for security reports:

- Skills that contain commands or scripts that could cause unintended harm
- Instructions that could lead to unauthorized access if followed incorrectly
- Sensitive data accidentally included in skill content
- Dependencies or external references that have become compromised

## Recognition

We credit responsible disclosures in our changelog. If you report a valid security issue, we will acknowledge your contribution unless you prefer to remain anonymous.

## Contact

For security matters that cannot be reported through GitHub's advisory system, reach out via the repository's discussion forum.

===== END FILE =====


===== FILE: ./SUPPORT.md =====
# Support

This project is maintained by one person alongside other work. That shapes what support looks like here, so it is worth being direct about it rather than leaving you to guess.

## Where to go

| I want to… | Use |
|---|---|
| Ask how something works | [Discussions](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/discussions) |
| Report a skill that is wrong, broken, or out of date | [Issues](https://github.com/mukul975/Anthropic-Cybersecurity-Skills/issues) |
| Propose a new skill | Issue first, then a PR — see [CONTRIBUTING.md](CONTRIBUTING.md) |
| Fix something yourself | Open a PR. This is the fastest path to change. |
| Report a security problem | See [SECURITY.md](SECURITY.md) |

Issues are for concrete defects: a command that does not work, a flag that does not exist, a framework ID that is wrong, a script that fails. Questions belong in Discussions, where other people can answer too and the answer stays findable.

## What to expect

There is no SLA. Response time varies with what else is happening; some pull requests have waited months, and that is a real cost I am working to reduce rather than a policy.

What moves fastest, in order:

1. A PR that fixes one thing and passes the validators.
2. An issue that names the skill, quotes the failing command, and shows the actual output.
3. Everything else.

What tends to stall: large multi-skill PRs, feature requests outside the skill-library format, and requests for bespoke integration help.

## Before you open an issue

```bash
pip install pyyaml
python tools/validate-skill.py skills/<skill-name>
```

If a skill fails validation, that output is the most useful thing you can paste. If you are reporting a procedure that does not work, say which version of the tool you ran and what it printed — "this does not work" is not actionable, and a wrong procedure in a security library is worth fixing properly.

## What this project is not

It is a reference library, not a product. It ships no service, holds no data, and makes no availability guarantee. Skills describe procedures for systems you own or are authorised to assess — see [SCOPE.md](SCOPE.md).

If you need something with a support contract behind it, this is not that, and I would rather say so plainly than have you discover it during an incident.

## Helping

The most useful contributions are unglamorous: correcting a procedure you actually ran, adding a framework mapping you verified, or telling me two skills are competing for the same request. All three make the library measurably better for everyone using it.

===== END FILE =====


===== FILE: ./assets/.gitkeep =====

===== END FILE =====


===== FILE: ./docs/mitre-f3-mapping.md =====
# MITRE Fight Fraud Framework (F3) — Mapping Schema

This repository maps fraud-relevant skills to the **MITRE Fight Fraud Framework (F3)**,
released April 9, 2026 by MITRE's Center for Threat-Informed Defense (CTID). F3 is an
ATT&CK-compatible TTP catalog for cyber-enabled financial fraud.

- Upstream project: <https://ctid.mitre.org/fraud/>
- Source repo: <https://github.com/center-for-threat-informed-defense/fight-fraud-framework>
- License: Apache-2.0
- Mapped version in this repo: **F3 v1.1**

## Why F3 in addition to ATT&CK

ATT&CK collapses post-compromise fraud into the single `T1657` (Financial Theft)
technique. F3 decomposes the "how a cyber intrusion becomes a financial loss" stages
into two dedicated tactics that ATT&CK does not have:

- **Positioning** (`FA0001`) — after access, collect/manipulate data and prepare the fraud.
- **Monetization** (`FA0002`) — convert stolen assets into usable funds.

So `mitre_attack` answers "how did the adversary get in / operate technically" and
`mitre_f3` answers "how did that turn into money." They are kept as **separate
frontmatter blocks** because F3 redefines several ATT&CK tactics for the fraud context.

## The 8 F3 v1.1 tactics

| Tactic slug | F3 ID | Origin |
|---|---|---|
| `reconnaissance` | TA0043 | ATT&CK (redefined) |
| `resource-development` | TA0042 | ATT&CK (redefined) |
| `initial-access` | TA0001 | ATT&CK (redefined) |
| `stealth` | TA0005 | ATT&CK (redefined) |
| `positioning` | **FA0001** | **F3-new** |
| `execution` | TA0002 | ATT&CK (redefined) |
| `monetization` | **FA0002** | **F3-new** |
| `defense-impairment` | TA0112 | ATT&CK (redefined) |

## Technique ID conventions

- **`F1XXX`** — fraud-specific techniques introduced by F3 (e.g. `F1005.003`
  Account Manipulation: Add Beneficiary, `F1025.003` Electronic Funds Transfer:
  Wire Transfer, `F1018` Convert to Cryptocurrency).
- **`T1XXX`** — ATT&CK techniques reused verbatim inside F3 (e.g. `T1566` Phishing,
  `T1586` Compromise Accounts, `T1557` Adversary-in-the-Middle).
- Sub-techniques use ATT&CK dot notation (`F1005.003`, `T1566.002`).

Every ID used in this repo is a real, active technique present in the F3 v1.1 STIX
bundle — there are no `TBD`/placeholder IDs.

## Frontmatter schema

The `mitre_f3` block sits alongside the existing `mitre_attack` block:

```yaml
mitre_f3:
  version: '1.1'
  tactics:
    - positioning
    - monetization
  techniques:
    - id: F1005.003
      name: 'Account Manipulation: Add Beneficiary'
      tactic: positioning
      source: f3          # F-prefixed = fraud-specific
    - id: T1586
      name: Compromise Accounts
      tactic: resource-development
      source: attack      # T-prefixed = reused ATT&CK
```

Rules:
1. `id` must be a real F3 v1.1 technique ID.
2. `name` must match the technique's official name in the F3 catalog.
3. `tactic` must be one the technique actually lists in the catalog.
4. `source` is `f3` for `F1XXX` IDs and `attack` for `T1XXX` IDs.

## Scope

F3 mappings are applied only to **fraud-relevant skills** — phishing/social
engineering, account takeover, banking malware/stealers, BEC, identity/KYC,
payment/card fraud, money-mule/cash-out, ransomware extortion, and the cross-cutting
DFIR and threat-intelligence skills. Skills with no fraud dimension do not carry an
`mitre_f3` block.

## Regenerating / verifying the catalog

```bash
git clone --depth 1 https://github.com/center-for-threat-informed-defense/fight-fraud-framework
# technique catalog is the STIX bundle:
#   fight-fraud-framework/public/f3-stix-v1.1.json
```

All `mitre_f3` IDs in this repo are validated against that bundle on every update.

===== END FILE =====


===== FILE: ./index.json =====
{"version":"1.1.0","generated_at":"2026-08-24T10:40:36Z","repository":"https://github.com/mukul975/Anthropic-Cybersecurity-Skills","domain":"cybersecurity","total_skills":818,"skills":[{"name":"abusing-dpapi-for-credential-access","description":"Extract and decrypt Windows DPAPI-protected secrets (Credential Manager, browser logins/cookies, Wi-Fi credentials, KeePass keys) online or offline using SharpDPAPI, SharpChrome, Mimikatz, or Impacket's dpapi.py, including domain-wide decryption via the DPAPI backup key. Use during authorized red-team credential-access engagements after gaining a foothold or when triaging DPAPI blobs pulled from a host.","domain":"cybersecurity","path":"skills/abusing-dpapi-for-credential-access"},{"name":"abusing-shadow-credentials-for-privesc","description":"Take over Active Directory accounts by writing attacker-controlled public keys to msDS-KeyCredentialLink (Shadow Credentials) with pyWhisker, Whisker, or Certipy, then authenticate via PKINIT to recover the target's NT hash without a password reset. Use when BloodHound shows GenericWrite/GenericAll/AddKeyCredentialLink over a target, as a stealthier alternative to ForceChangePassword, during authorized red-team engagements.","domain":"cybersecurity","path":"skills/abusing-shadow-credentials-for-privesc"},{"name":"achieving-cmmc-level-2-compliance","description":"Prepare a defense-contractor environment for CMMC Level 2 certification: scope CUI and FCI, implement the 110 NIST SP 800-171 Rev 2 security requirements across 14 families, compute the SPRS score with the DoD Assessment Methodology, manage a compliant POA&M, and ready the organization for a C3PAO assessment. Use when an organization handles Controlled Unclassified Information (CUI) under a DoD contract, when a contract carries DFARS clause 252.204-7012/7019/7020/7021, when preparing for or responding to a CMMC assessment, when computing or improving an SPRS score, when building a System Security Plan or POA&M for 800-171, or when scoping which systems are in the CUI boundary. Keywords: CMMC, CMMC Level 2, NIST 800-171, SP 800-171 Rev 2, CUI, FCI, SPRS, DFARS 7012, C3PAO, POA&M, System Security Plan, DoD Assessment Methodology, 110 controls, defense industrial base, DIB, FedRAMP equivalency.","domain":"cybersecurity","path":"skills/achieving-cmmc-level-2-compliance"},{"name":"acquiring-disk-image-with-dd-and-dcfldd","description":"Create forensically sound bit-for-bit disk images with dd or dcfldd on a Linux forensic workstation, preserving evidence integrity through hash verification (MD5/SHA) during acquisition. Use when imaging a suspect drive, USB device, or memory card for investigation, preserving volatile disk evidence during incident response, or producing a verified copy for legal or law-enforcement proceedings before any destructive analysis.","domain":"cybersecurity","path":"skills/acquiring-disk-image-with-dd-and-dcfldd"},{"name":"analyzing-active-directory-acl-abuse","description":"Detect dangerous ACL misconfigurations in Active Directory using ldap3 to identify GenericAll, WriteDACL, and WriteOwner abuse paths","domain":"cybersecurity","path":"skills/analyzing-active-directory-acl-abuse"},{"name":"analyzing-android-malware-with-apktool","description":"Perform static analysis of Android APK malware using apktool for resource decompilation, jadx for Java source recovery, and androguard for manifest inspection, dangerous permission-combination detection, and identification of obfuscated code, dynamic code loading, and reflection-based API calls. Use to statically triage a suspicious APK without executing it or to build mobile malware detection rules.","domain":"cybersecurity","path":"skills/analyzing-android-malware-with-apktool"},{"name":"analyzing-api-gateway-access-logs","description":"Parses API Gateway access logs (AWS API Gateway, Kong, Nginx) to detect BOLA/IDOR attacks, rate limit bypass, credential scanning, and injection attempts. Uses pandas for statistical analysis of request patterns and anomaly detection. Use when investigating API abuse or building API-specific threat detection rules.","domain":"cybersecurity","path":"skills/analyzing-api-gateway-access-logs"},{"name":"analyzing-apt-group-with-mitre-navigator","description":"Query ATT&CK data with attackcti, mitreattack-python, and stix2, then build MITRE ATT&CK Navigator layers and multi-layer heatmap overlays mapping one or more APT groups' TTPs for detection-gap analysis. Use to compare threat-actor technique coverage, find gaps in detection engineering, or produce Navigator visualizations for threat-intel reporting.","domain":"cybersecurity","path":"skills/analyzing-apt-group-with-mitre-navigator"},{"name":"analyzing-azure-activity-logs-for-threats","description":"Queries Azure Monitor activity logs and sign-in logs via azure-monitor-query to detect suspicious administrative operations, impossible travel, privilege escalation, and resource modifications. Builds KQL queries for threat hunting in Azure environments. Use when investigating suspicious Azure tenant activity or building cloud SIEM detections.","domain":"cybersecurity","path":"skills/analyzing-azure-activity-logs-for-threats"},{"name":"analyzing-bootkit-and-rootkit-samples","description":"Analyzes bootkit and advanced rootkit malware infecting the Master Boot Record (MBR), Volume Boot Record (VBR), or UEFI firmware for below-OS persistence, covering boot sector analysis, UEFI module inspection, and anti-rootkit detection. Use when compromise survives OS reinstallation or antivirus/EDR fails to detect malware despite clear infection signs.","domain":"cybersecurity","path":"skills/analyzing-bootkit-and-rootkit-samples"},{"name":"analyzing-browser-forensics-with-hindsight","description":"Parse Chromium-based browser databases with Hindsight to extract and correlate browsing history, downloads, cookies, cached content, autofill data, saved passwords, and extensions from Chrome, Edge, Brave, Opera, and Vivaldi into a unified timeline (XLSX, JSON, or SQLite output). Use during incident response, insider-threat investigations, or criminal cases when you need to reconstruct a user's web activity from a browser profile.","domain":"cybersecurity","path":"skills/analyzing-browser-forensics-with-hindsight"},{"name":"analyzing-campaign-attribution-evidence","description":"Systematically evaluate cyber-campaign evidence to attribute an operation to a threat actor, using the Diamond Model and Analysis of Competing Hypotheses (ACH) to weigh infrastructure overlaps, TTP consistency, malware code similarity, and timing/language artifacts into confidence-weighted attribution assessments. Use when an incident investigation needs a defensible attribution confidence level.","domain":"cybersecurity","path":"skills/analyzing-campaign-attribution-evidence"},{"name":"analyzing-certificate-transparency-for-phishing","description":"Monitor Certificate Transparency logs using crt.sh and Certstream to detect phishing domains, lookalike certificates, and unauthorized certificate issuance targeting your organization.","domain":"cybersecurity","path":"skills/analyzing-certificate-transparency-for-phishing"},{"name":"analyzing-cloud-storage-access-patterns","description":"Detect abnormal access in AWS S3, GCS, and Azure Blob Storage by analyzing CloudTrail Data Events, GCS audit logs, and Azure Storage Analytics for after-hours bulk downloads, new-IP access, and API-call spikes (e.g. GetObject) via statistical baselines and time-series anomaly detection. Use when investigating suspected cloud data exfiltration or building related detection rules.","domain":"cybersecurity","path":"skills/analyzing-cloud-storage-access-patterns"},{"name":"analyzing-cobalt-strike-beacon-configuration","description":"Extract and analyze Cobalt Strike beacon configuration from PE files and memory dumps to identify C2 infrastructure, malleable profiles, and operator tradecraft.","domain":"cybersecurity","path":"skills/analyzing-cobalt-strike-beacon-configuration"},{"name":"analyzing-cobaltstrike-malleable-c2-profiles","description":"Parse and analyze Cobalt Strike Malleable C2 profiles with dissect.cobaltstrike (profiles and beacon-payload configs) and pyMalleableC2 (AST parsing) to extract HTTP/DNS transforms, URIs, headers, sleep/jitter, and injection behavior, then generate network detection signatures. Use when reverse-engineering a captured malleable profile or building detections against Cobalt Strike Beacon traffic.","domain":"cybersecurity","path":"skills/analyzing-cobaltstrike-malleable-c2-profiles"},{"name":"analyzing-command-and-control-communication","description":"Analyzes malware C2 communication over HTTP, HTTPS, DNS, and custom protocols to reverse-engineer beacon patterns, command structures, data encoding, and infrastructure (primary servers, fallback domains, dead drops). Use after reverse engineering reveals network traffic needing protocol analysis or when building detection signatures for a framework like Cobalt Strike, Metasploit, or Sliver.","domain":"cybersecurity","path":"skills/analyzing-command-and-control-communication"},{"name":"analyzing-cyber-kill-chain","description":"Analyzes intrusion activity against the Lockheed Martin Cyber Kill Chain framework to identify which phases an adversary has completed, where defenses succeeded or failed, and what controls would have interrupted the attack at earlier phases. Use when conducting post-incident analysis, building prevention-focused security controls, or mapping detection gaps to kill chain phases. Activates for requests involving kill chain analysis, intrusion kill chain, attack phase mapping, or Lockheed Martin kill chain framework.","domain":"cybersecurity","path":"skills/analyzing-cyber-kill-chain"},{"name":"analyzing-disk-image-with-autopsy","description":"Perform comprehensive forensic analysis of raw (dd), E01, or AFF disk images with Autopsy and The Sleuth Kit, recovering deleted files, examining metadata and embedded artifacts, keyword searching, and building investigation timelines with visual reports. Use for structured analysis of a forensic disk image or when stakeholders need visual reports from evidence.","domain":"cybersecurity","path":"skills/analyzing-disk-image-with-autopsy"},{"name":"analyzing-dns-logs-for-exfiltration","description":"Analyzes DNS query logs to detect data exfiltration via DNS tunneling, DGA domain communication, and covert C2 channels using entropy analysis, query volume anomalies, and subdomain length detection in SIEM platforms. Use when SOC teams need to identify DNS-based threats that bypass traditional network security controls.","domain":"cybersecurity","path":"skills/analyzing-dns-logs-for-exfiltration"},{"name":"analyzing-docker-container-forensics","description":"Investigate compromised Docker containers by analyzing images, layers, volumes, logs, and runtime artifacts to identify malicious activity and evidence.","domain":"cybersecurity","path":"skills/analyzing-docker-container-forensics"},{"name":"analyzing-email-headers-for-phishing-investigation","description":"Parse and analyze email headers (Received chain, Return-Path, Message-ID) to trace the true origin of a phishing email and validate SPF, DKIM, and DMARC results to confirm or rule out sender spoofing. Use when triaging a suspicious or reported email, investigating a phishing incident, or verifying whether a message's sender domain was spoofed.","domain":"cybersecurity","path":"skills/analyzing-email-headers-for-phishing-investigation"},{"name":"analyzing-ethereum-smart-contract-vulnerabilities","description":"Perform static and symbolic analysis of Solidity smart contracts using Slither and Mythril to detect reentrancy, integer overflow, access control, and other vulnerability classes before deployment to Ethereum mainnet.","domain":"cybersecurity","path":"skills/analyzing-ethereum-smart-contract-vulnerabilities"},{"name":"analyzing-golang-malware-with-ghidra","description":"Reverse engineer Go-compiled malware in Ghidra by parsing Go buildinfo and pclntab structures, recovering stripped/obfuscated function names (e.g. via GoResolver), and extracting embedded module/dependency strings and types from Go binaries. Use when analyzing a Go-language malware sample, deobfuscating a garble-packed Go binary, or recovering function names and third-party dependencies from a stripped Go executable.","domain":"cybersecurity","path":"skills/analyzing-golang-malware-with-ghidra"},{"name":"analyzing-heap-spray-exploitation","description":"Detect and analyze heap spray attacks in memory dumps using Volatility3 plugins to identify NOP sled patterns, shellcode landing zones, and suspicious large allocations in process virtual address space.","domain":"cybersecurity","path":"skills/analyzing-heap-spray-exploitation"},{"name":"analyzing-indicators-of-compromise","description":"Analyzes indicators of compromise (IOCs) including IP addresses, domains, file hashes, URLs, and email artifacts to determine maliciousness confidence, campaign attribution, and blocking priority. Use when triaging IOCs from phishing emails, security alerts, or external threat feeds; enriching raw IOCs with multi-source intelligence; or making block/monitor/whitelist decisions. Activates for requests involving VirusTotal, AbuseIPDB, MalwareBazaar, MISP, or IOC enrichment pipelines.","domain":"cybersecurity","path":"skills/analyzing-indicators-of-compromise"},{"name":"analyzing-ios-app-security-with-objection","description":"Runtime iOS app security testing with Objection (Frida): inspect keychain and filesystem data, explore app internals at runtime, and validate/bypass client-side protections during authorized mobile assessments.","domain":"cybersecurity","path":"skills/analyzing-ios-app-security-with-objection"},{"name":"analyzing-kubernetes-audit-logs","description":"Parses Kubernetes API server audit logs (JSON lines) to detect exec-into-pod, secret access, RBAC modifications, privileged pod creation, and anonymous API access, and builds SIEM detection rules from the event patterns. Use when investigating a suspected cluster compromise, reconstructing what an attacker did through the API server, or writing Kubernetes-specific detection content. Keywords: audit policy, audit log, kube-apiserver, exec into pod, RBAC change, anonymous access, detection rules. Do not use for syscall-level detection inside a running container - use detecting-container-runtime-threats-with-falco. '","domain":"cybersecurity","path":"skills/analyzing-kubernetes-audit-logs"},{"name":"analyzing-linux-audit-logs-for-intrusion","description":"Uses the Linux Audit framework (auditd) with ausearch and aureport utilities to detect intrusion attempts, unauthorized access, privilege escalation, and suspicious system activity. Covers audit rule configuration, log querying, timeline reconstruction, and integration with SIEM platforms. Activates for requests involving auditd analysis, Linux audit log investigation, ausearch queries, aureport summaries, or host-based intrusion detection on Linux.","domain":"cybersecurity","path":"skills/analyzing-linux-audit-logs-for-intrusion"},{"name":"analyzing-linux-elf-malware","description":"Analyze malicious Linux ELF binaries \u2014 botnets, cryptominers, ransomware, and rootkits targeting Linux servers, containers, and cloud infrastructure \u2014 through static analysis, dynamic tracing, and reverse engineering of x86_64 and ARM samples. Use when investigating Linux malware, triaging a suspicious ELF binary, assessing a compromised Linux server, or analyzing container-targeted malware.","domain":"cybersecurity","path":"skills/analyzing-linux-elf-malware"},{"name":"analyzing-linux-kernel-rootkits","description":"Detect kernel-level rootkits in Linux memory dumps using Volatility3 linux plugins (check_syscall, lsmod, hidden_modules), rkhunter system scanning, and /proc vs /sys discrepancy analysis to identify hooked syscalls, hidden kernel modules, and tampered system structures.","domain":"cybersecurity","path":"skills/analyzing-linux-kernel-rootkits"},{"name":"analyzing-linux-system-artifacts","description":"Examine Linux system artifacts (auth logs, cron/systemd persistence, shell history, SSH keys, and system configuration) to uncover evidence of compromise, detect rootkits or backdoors, and reconstruct user/attacker activity. Use when investigating a compromised Linux server or workstation, hunting for persistence mechanisms, or scoping a Linux-based breach during incident response.","domain":"cybersecurity","path":"skills/analyzing-linux-system-artifacts"},{"name":"analyzing-lnk-file-and-jump-list-artifacts","description":"Analyze Windows LNK shortcut files and Jump List artifacts with LECmd, JLECmd, and manual Shell Link Binary Format parsing to establish evidence of file access, program execution, and user activity that persists even after the target file is deleted. Use when investigating Windows user activity, reconstructing file-access or program-execution timelines, or examining recent/frequently-used file evidence in a forensic exam.","domain":"cybersecurity","path":"skills/analyzing-lnk-file-and-jump-list-artifacts"},{"name":"analyzing-macro-malware-in-office-documents","description":"Analyzes malicious VBA macros embedded in Microsoft Office documents (Word, Excel, PowerPoint) to identify download cradles, payload execution, persistence mechanisms, and anti-analysis techniques. Uses olevba, oledump, and VBA deobfuscation to extract the attack chain. Activates for requests involving Office macro analysis, VBA malware investigation, maldoc analysis, or document-based threat examination.","domain":"cybersecurity","path":"skills/analyzing-macro-malware-in-office-documents"},{"name":"analyzing-malicious-pdf-with-peepdf","description":"Perform static analysis of malicious PDF documents using peepdf, pdfid, and pdf-parser to extract embedded JavaScript, shellcode, and suspicious objects. Use when triaging a suspicious PDF attachment from a phishing email, analyzing a PDF-based exploit document, or building detection signatures for weaponized PDF threats.","domain":"cybersecurity","path":"skills/analyzing-malicious-pdf-with-peepdf"},{"name":"analyzing-malicious-url-with-urlscan","description":"URLScan.io is a free service for scanning and analyzing suspicious URLs. It captures screenshots, DOM content, HTTP transactions, JavaScript behavior, and network connections of web pages in an isolat","domain":"cybersecurity","path":"skills/analyzing-malicious-url-with-urlscan"},{"name":"analyzing-malware-behavior-with-cuckoo-sandbox","description":"Detonate malware samples in Cuckoo Sandbox to observe runtime behavior \u2014 process creation, file system and registry changes, network communications, and API calls \u2014 and generate behavioral reports for classification and IOC extraction. Use when a sample has passed static triage and needs dynamic/behavioral analysis, when mapping a full infection chain, or when building YARA/behavioral signatures from observed sandbox activity.","domain":"cybersecurity","path":"skills/analyzing-malware-behavior-with-cuckoo-sandbox"},{"name":"analyzing-malware-family-relationships-with-malpedia","description":"Query the Malpedia API to look up malware family aliases and naming (platform.family_name), pull community/vendor YARA rules, link families to threat actors, and map family relationships such as loader-payload chains and shared authorship. Use when researching a malware family's aliases, lineage, or actor attribution, or when sourcing YARA rules for detection.","domain":"cybersecurity","path":"skills/analyzing-malware-family-relationships-with-malpedia"},{"name":"analyzing-malware-persistence-with-autoruns","description":"Use Sysinternals Autoruns to systematically enumerate and analyze malware persistence mechanisms across Windows registry run keys, scheduled tasks, services, drivers, and startup locations. Use when hunting for persistence during Windows incident response, triaging a compromised endpoint, or validating that malware autostart entries have been fully identified and removed.","domain":"cybersecurity","path":"skills/analyzing-malware-persistence-with-autoruns"},{"name":"analyzing-malware-sandbox-evasion-techniques","description":"Detect sandbox and VM evasion techniques in malware samples by analyzing timing checks, VM/hypervisor artifact queries, user-interaction checks, and sleep-inflation patterns from Cuckoo or AnyRun behavioral reports. Use when a sample shows no or minimal activity in a sandbox, when a behavioral report needs review for evasion indicators, or when building detections for anti-analysis techniques.","domain":"cybersecurity","path":"skills/analyzing-malware-sandbox-evasion-techniques"},{"name":"analyzing-memory-dumps-with-volatility","description":"Analyzes RAM memory dumps from compromised systems using the Volatility framework to identify malicious processes, injected code, network connections, loaded modules, and extracted credentials. Supports Windows, Linux, and macOS memory forensics. Activates for requests involving memory forensics, RAM analysis, volatile data examination, process injection detection, or memory-resident malware investigation.","domain":"cybersecurity","path":"skills/analyzing-memory-dumps-with-volatility"},{"name":"analyzing-memory-forensics-with-lime-and-volatility","description":"Performs Linux memory acquisition using LiME (Linux Memory Extractor) kernel module and analysis with Volatility 3 framework. Extracts process lists, network connections, bash history, loaded kernel modules, and injected code from Linux memory images. Use when performing incident response on compromised Linux systems.","domain":"cybersecurity","path":"skills/analyzing-memory-forensics-with-lime-and-volatility"},{"name":"analyzing-mft-for-deleted-file-recovery","description":"Analyze the NTFS Master File Table ($MFT) with MFTECmd, analyzeMFT, and X-Ways Forensics to recover metadata and content of deleted files by examining MFT record entries, $LogFile, $UsnJrnl, and MFT slack space. Use when recovering evidence of deleted files, reconstructing NTFS file-system timelines, or detecting anti-forensic timestomping during a Windows forensic examination.","domain":"cybersecurity","path":"skills/analyzing-mft-for-deleted-file-recovery"},{"name":"analyzing-network-covert-channels-in-malware","description":"Detect and analyze covert communication channels used by malware, including DNS tunneling, ICMP exfiltration, steganographic HTTP, and other protocol abuse used for C2 and data exfiltration. Use when investigating suspicious DNS/ICMP/HTTP traffic patterns, hunting for hidden C2 channels in network captures, or attributing exfiltration traffic to a known tunneling toolset.","domain":"cybersecurity","path":"skills/analyzing-network-covert-channels-in-malware"},{"name":"analyzing-network-flow-data-with-netflow","description":"Parse NetFlow v9 and IPFIX records to detect volumetric anomalies, port scanning, data exfiltration, and C2 beaconing patterns. Uses the Python netflow library to decode flow records, builds traffic baselines, and applies statistical analysis to identify flows with abnormal byte counts, connection durations, and periodic timing patterns.","domain":"cybersecurity","path":"skills/analyzing-network-flow-data-with-netflow"},{"name":"analyzing-network-packets-with-scapy","description":"Use Scapy to craft, send, sniff, and dissect TCP/UDP/ICMP/DNS packets, analyze pcap files, implement SYN scans, and detect anomalous traffic such as fragmented or malformed packets. Use when performing authorized network reconnaissance, protocol-level forensic analysis, or building traffic anomaly detection during security testing.","domain":"cybersecurity","path":"skills/analyzing-network-packets-with-scapy"},{"name":"analyzing-network-traffic-for-incidents","description":"Analyzes network traffic captures and flow data to identify adversary activity during security incidents, including command-and-control communications, lateral movement, data exfiltration, and exploitation attempts. Uses Wireshark, Zeek, and NetFlow analysis techniques. Activates for requests involving network traffic analysis, packet capture investigation, PCAP analysis, network forensics, C2 traffic detection, or exfiltration detection.","domain":"cybersecurity","path":"skills/analyzing-network-traffic-for-incidents"},{"name":"analyzing-network-traffic-of-malware","description":"Analyzes network traffic generated by malware during sandbox execution or live incident response to identify C2 protocols, data exfiltration channels, payload downloads, and lateral movement patterns using Wireshark, Zeek, and Suricata. Activates for requests involving malware network analysis, C2 traffic decoding, malware PCAP analysis, or network-based malware detection.","domain":"cybersecurity","path":"skills/analyzing-network-traffic-of-malware"},{"name":"analyzing-network-traffic-with-wireshark","description":"Captures and analyzes network packet data using Wireshark and tshark to identify malicious traffic patterns, diagnose protocol issues, extract artifacts, and support incident response investigations on authorized network segments.","domain":"cybersecurity","path":"skills/analyzing-network-traffic-with-wireshark"},{"name":"analyzing-office365-audit-logs-for-compromise","description":"Parse Office 365 Unified Audit Logs via Microsoft Graph API to detect email forwarding rule creation, inbox delegation, suspicious OAuth app grants, and other indicators of account compromise.","domain":"cybersecurity","path":"skills/analyzing-office365-audit-logs-for-compromise"},{"name":"analyzing-outlook-pst-for-email-forensics","description":"Parse Microsoft Outlook PST and OST files using libpff and pst-utils to extract message content, headers, attachments, deleted items, and MAPI metadata, including recovery of items from the Recoverable Items folder. Use when conducting email forensic investigations, legal e-discovery, or incident response that requires reconstructing communication patterns or tracing message routing from Outlook archives.","domain":"cybersecurity","path":"skills/analyzing-outlook-pst-for-email-forensics"},{"name":"analyzing-packed-malware-with-upx-unpacker","description":"Identifies and unpacks UPX-packed malware samples, including binaries with modified UPX magic bytes or headers that block automated decompression, to recover the original executable for static analysis. Use when a sample shows high entropy, minimal imports, or only LoadLibrary/GetProcAddress in its import table, or when preparing a packed binary for disassembly in Ghidra or IDA.","domain":"cybersecurity","path":"skills/analyzing-packed-malware-with-upx-unpacker"},{"name":"analyzing-pdf-malware-with-pdfid","description":"Analyzes malicious PDF files using PDFiD, pdf-parser, and peepdf to identify embedded JavaScript, shellcode, exploits, and suspicious objects without opening the document. Determines the attack vector and extracts embedded payloads for further analysis. Activates for requests involving PDF malware analysis, malicious document analysis, PDF exploit investigation, or suspicious attachment triage.","domain":"cybersecurity","path":"skills/analyzing-pdf-malware-with-pdfid"},{"name":"analyzing-persistence-mechanisms-in-linux","description":"Scan Linux systems for persistence mechanisms including crontab/systemd entries, LD_PRELOAD injection, shell profile modifications (.bashrc, .profile), and SSH authorized_keys backdoors, then correlate findings with auditd logs into an installation timeline. Use during incident response or threat hunting to detect or confirm how an adversary maintained access to a compromised Linux host.","domain":"cybersecurity","path":"skills/analyzing-persistence-mechanisms-in-linux"},{"name":"analyzing-powershell-empire-artifacts","description":"Detect PowerShell Empire post-exploitation framework artifacts in Windows Script Block Logging (Event ID 4104) and Module Logging (Event ID 4103), including the default launcher string, Base64-encoded WebClient/FromBase64String payloads, known module invocations (Invoke-Mimikatz, Invoke-Kerberoast), and staging URL patterns. Use when hunting for or confirming Empire C2 activity in Windows event logs.","domain":"cybersecurity","path":"skills/analyzing-powershell-empire-artifacts"},{"name":"analyzing-powershell-script-block-logging","description":"Parse Windows PowerShell Script Block Logs (Event ID 4104) from EVTX files to detect obfuscated commands, encoded payloads, and living-off-the-land techniques. Uses python-evtx to extract and reconstruct multi-block scripts, applies entropy analysis and pattern matching for Base64-encoded commands, Invoke-Expression abuse, download cradles, and AMSI bypass attempts.","domain":"cybersecurity","path":"skills/analyzing-powershell-script-block-logging"},{"name":"analyzing-prefetch-files-for-execution-history","description":"Parse Windows Prefetch files (versions 17, 23, 26, 30) with tools like PECmd, WinPrefetchView, or python-prefetch to determine program execution history, including run counts, execution timestamps, and referenced files/DLLs. Use when building a timeline of program execution on a Windows system, confirming whether a suspicious binary ran, or correlating execution evidence with other forensic artifacts during an investigation.","domain":"cybersecurity","path":"skills/analyzing-prefetch-files-for-execution-history"},{"name":"analyzing-ransomware-encryption-mechanisms","description":"Analyzes encryption algorithms, key management, and file encryption routines used by ransomware families to assess decryption feasibility, identify implementation weaknesses, and support recovery efforts. Covers AES, RSA, ChaCha20, and hybrid encryption schemes. Activates for requests involving ransomware cryptanalysis, encryption analysis, key recovery assessment, or ransomware decryption feasibility.","domain":"cybersecurity","path":"skills/analyzing-ransomware-encryption-mechanisms"},{"name":"analyzing-ransomware-leak-site-intelligence","description":"Safely monitor ransomware group Tor-hosted data leak sites (DLS) to collect and extract structured victim posting data, track group activity trends over time, and produce sector- and geography-specific ransomware risk assessments. Use when performing threat intelligence gathering on active ransomware groups or building proactive defense reporting from double-extortion leak-site activity.","domain":"cybersecurity","path":"skills/analyzing-ransomware-leak-site-intelligence"},{"name":"analyzing-ransomware-network-indicators","description":"Identify ransomware-related network indicators, including C2 beaconing patterns, TOR exit node connections, data exfiltration flows, and encryption key exchange, by analyzing Zeek conn.log and NetFlow data. Use when threat hunting for active ransomware network activity or investigating suspected pre-encryption exfiltration during incident response.","domain":"cybersecurity","path":"skills/analyzing-ransomware-network-indicators"},{"name":"analyzing-ransomware-payment-wallets","description":"Traces ransomware cryptocurrency payment flows using blockchain analysis tools such as Chainalysis Reactor, WalletExplorer, and blockchain.com APIs, identifying wallet clusters and tracking fund movement through mixers and exchanges to support law enforcement attribution. Use when tracing ransomware bitcoin payments, performing cryptocurrency wallet forensics, or gathering blockchain threat intelligence on extortion payments.","domain":"cybersecurity","path":"skills/analyzing-ransomware-payment-wallets"},{"name":"analyzing-sbom-for-supply-chain-vulnerabilities","description":"Parses Software Bill of Materials (SBOM) in CycloneDX and SPDX JSON formats to identify supply chain vulnerabilities by correlating components against the NVD CVE database via the NVD 2.0 API. Builds dependency graphs, calculates risk scores, identifies transitive vulnerability paths, and generates compliance reports. Activates for requests involving SBOM analysis, software composition analysis, supply chain security assessment, dependency vulnerability scanning, CycloneDX/SPDX parsing, or CVE correlation.","domain":"cybersecurity","path":"skills/analyzing-sbom-for-supply-chain-vulnerabilities"},{"name":"analyzing-security-logs-with-splunk","description":"Leverages Splunk Enterprise Security and SPL (Search Processing Language) to investigate security incidents through log correlation, timeline reconstruction, and anomaly detection. Covers Windows event logs, firewall logs, proxy logs, and authentication data analysis. Activates for requests involving Splunk investigation, SPL queries, SIEM log analysis, security event correlation, or log-based incident investigation.","domain":"cybersecurity","path":"skills/analyzing-security-logs-with-splunk"},{"name":"analyzing-slack-space-and-file-system-artifacts","description":"Examine NTFS slack space, MFT entries, the USN Change Journal, and Alternate Data Streams (ADS) to recover hidden or residual data, reconstruct deleted-file metadata, and reconstruct available file-system change activity from USN records. Use during deep forensic analysis of an NTFS image when standard file recovery is insufficient, such as hunting for data hidden in ADS.","domain":"cybersecurity","path":"skills/analyzing-slack-space-and-file-system-artifacts"},{"name":"analyzing-supply-chain-malware-artifacts","description":"Investigate supply chain attack artifacts including trojanized software updates, compromised build pipelines, and sideloaded dependencies to identify intrusion vectors and scope of compromise.","domain":"cybersecurity","path":"skills/analyzing-supply-chain-malware-artifacts"},{"name":"analyzing-threat-actor-ttps-with-mitre-attack","description":"Systematically map threat actor behavior and observed IOCs to the MITRE ATT&CK framework, build technique coverage heatmaps with the ATT&CK Navigator, identify detection gaps, and produce actionable threat intelligence reports across the Enterprise, Mobile, and ICS matrices. Use when analyzing threat actor TTPs, correlating IOCs to specific ATT&CK techniques, or assessing defensive detection coverage against adversary behavior.","domain":"cybersecurity","path":"skills/analyzing-threat-actor-ttps-with-mitre-attack"},{"name":"analyzing-threat-actor-ttps-with-mitre-navigator","description":"Map advanced persistent threat (APT) group TTPs to the MITRE ATT&CK framework using the attackcti Python library to query STIX/TAXII data for group-technique associations, then generate ATT&CK Navigator layer files to visualize and compare defensive coverage against adversary profiles. Use when profiling an APT group's techniques, building Navigator coverage heatmaps, or assessing technique coverage gaps against a specific threat actor.","domain":"cybersecurity","path":"skills/analyzing-threat-actor-ttps-with-mitre-navigator"},{"name":"analyzing-threat-intelligence-feeds","description":"Analyzes structured and unstructured threat intelligence feeds to extract actionable indicators, adversary tactics, and campaign context. Use when ingesting commercial or open-source CTI feeds, evaluating feed quality, normalizing data into STIX 2.1 format, or enriching existing IOCs with campaign attribution. Activates for requests involving ThreatConnect, Recorded Future, Mandiant Advantage, MISP, AlienVault OTX, or automated feed aggregation pipelines.","domain":"cybersecurity","path":"skills/analyzing-threat-intelligence-feeds"},{"name":"analyzing-threat-landscape-with-misp","description":"Query a MISP (Malware Information Sharing Platform) instance via PyMISP to compute event statistics, IOC type breakdowns, threat actor galaxy clusters, and tag trends, and generate threat landscape reports with temporal trends. Use when asked to analyze threat intelligence data, summarize top threat actors or malware families, or produce a CTI landscape report from MISP events.","domain":"cybersecurity","path":"skills/analyzing-threat-landscape-with-misp"},{"name":"analyzing-tls-certificate-transparency-logs","description":"Queries Certificate Transparency logs via crt.sh and pycrtsh to detect phishing domains, unauthorized certificate issuance, and shadow IT. Monitors newly issued certificates for typosquatting and brand impersonation using Levenshtein distance. Use for proactive phishing domain detection and certificate monitoring.","domain":"cybersecurity","path":"skills/analyzing-tls-certificate-transparency-logs"},{"name":"analyzing-typosquatting-domains-with-dnstwist","description":"Generate domain permutations with dnstwist and check DNS resolution to detect typosquatting, homograph phishing, and brand impersonation domains registered against your organization. Use when asked to monitor for lookalike domains, investigate a phishing domain, or assess brand-impersonation risk.","domain":"cybersecurity","path":"skills/analyzing-typosquatting-domains-with-dnstwist"},{"name":"analyzing-uefi-bootkit-persistence","description":"Analyzes UEFI bootkit persistence (SPI flash implants, ESP modifications, Secure Boot bypass, UEFI variable manipulation) using chipsec for firmware integrity verification, detecting known families like BlackLotus, LoJax, and MoonBounce. Use for UEFI malware analysis, firmware persistence investigation, or Secure Boot bypass detection.","domain":"cybersecurity","path":"skills/analyzing-uefi-bootkit-persistence"},{"name":"analyzing-usb-device-connection-history","description":"Correlate Windows registry keys (USBSTOR, MountedDevices), Event Logs, and setupapi.dev.log to reconstruct USB device connection history, first/last-plugged timestamps, and drive letter mappings. Use when investigating removable media usage, tracking device provenance, or building a timeline for suspected data exfiltration.","domain":"cybersecurity","path":"skills/analyzing-usb-device-connection-history"},{"name":"analyzing-web-server-logs-for-intrusion","description":"Parse Apache and Nginx access logs to detect SQL injection attempts, local file inclusion, directory traversal, web scanner fingerprints, and brute-force patterns. Uses regex-based pattern matching against OWASP attack signatures, GeoIP enrichment for source attribution, and statistical anomaly detection for request frequency and response size outliers.","domain":"cybersecurity","path":"skills/analyzing-web-server-logs-for-intrusion"},{"name":"analyzing-windows-amcache-artifacts","description":"Parses the Windows Amcache.hve registry hive with Eric Zimmerman's AmcacheParser and Timeline Explorer to extract evidence of program execution, application installation, and driver loading, including SHA-1 hash correlation with threat intel and timeline reconstruction. Use for Amcache forensics, program execution evidence gathering, or application compatibility cache investigations in DFIR work.","domain":"cybersecurity","path":"skills/analyzing-windows-amcache-artifacts"},{"name":"analyzing-windows-event-logs-in-splunk","description":"Analyzes Windows Security, System, and Sysmon event logs in Splunk to detect authentication attacks, privilege escalation, persistence mechanisms, and lateral movement using SPL queries mapped to MITRE ATT&CK techniques. Use when SOC analysts need to investigate Windows-based threats, build detection queries, or perform forensic timeline analysis of Windows endpoints and domain controllers.","domain":"cybersecurity","path":"skills/analyzing-windows-event-logs-in-splunk"},{"name":"analyzing-windows-lnk-files-for-artifacts","description":"Parse Windows LNK shortcut files to extract target paths, MAC timestamps, volume serial numbers, and machine identifiers for forensic timeline reconstruction. Use when investigating recently-accessed files, tracking removable media or network paths referenced by shortcuts, or building a DFIR timeline from LNK artifacts.","domain":"cybersecurity","path":"skills/analyzing-windows-lnk-files-for-artifacts"},{"name":"analyzing-windows-prefetch-with-python","description":"Parse Windows Prefetch (.pf) files with the windowsprefetch Python library to reconstruct application execution history, run counts, and accessed file/volume lists. Use when investigating renamed or masquerading binaries, verifying program execution timelines, or hunting for suspicious execution patterns in incident response.","domain":"cybersecurity","path":"skills/analyzing-windows-prefetch-with-python"},{"name":"analyzing-windows-registry-for-artifacts","description":"Extract and analyze Windows Registry hives with tools like RegRipper and Registry Explorer to uncover user activity, installed software, autostart/persistence entries, and evidence of system compromise. Use when investigating registry-based persistence, reconstructing user or system activity, or performing DFIR triage on a Windows image.","domain":"cybersecurity","path":"skills/analyzing-windows-registry-for-artifacts"},{"name":"analyzing-windows-shellbag-artifacts","description":"Analyze Windows Shellbag (BagMRU) registry artifacts with SBECmd and Shellbags Explorer to reconstruct folder browsing activity and prove user interaction with directories, including removable media and network shares, even after the folders are deleted. Use when reconstructing a user's folder access history or proving access to a since-removed directory in DFIR work.","domain":"cybersecurity","path":"skills/analyzing-windows-shellbag-artifacts"},{"name":"assessing-vector-and-embedding-weaknesses","description":"Test RAG vector stores (Pinecone, Qdrant, Weaviate, Chroma, pgvector, FAISS) for embedding inversion, cross-tenant data leakage, and data poisoning per OWASP LLM08:2025. Use when performing an authorized security assessment of a RAG pipeline's retrieval layer or auditing multi-tenant vector-store isolation.","domain":"cybersecurity","path":"skills/assessing-vector-and-embedding-weaknesses"},{"name":"attacking-entra-id-with-roadtools","description":"Enumerate Microsoft Entra ID (Azure AD) tenants with ROADrecon and acquire, exchange, and abuse tokens (including primary refresh tokens) with roadtx. Use for authorized red-team enumeration of a tenant's directory objects or for token-based identity attacks against Entra ID you are explicitly authorized to test.","domain":"cybersecurity","path":"skills/attacking-entra-id-with-roadtools"},{"name":"attacking-oauth-with-device-code-phishing","description":"Run OAuth 2.0 device-code and illicit-consent phishing attacks against Microsoft Entra ID, using TokenTactics-style tooling to steal access and refresh tokens, bypass MFA, and pivot across Microsoft 365 services. Use for authorized red-team engagements simulating device-code or consent-grant phishing against a tenant you have explicit written permission to test.","domain":"cybersecurity","path":"skills/attacking-oauth-with-device-code-phishing"},{"name":"auditing-aws-s3-bucket-permissions","description":"Systematically audit AWS S3 bucket permissions to identify publicly accessible buckets, overly permissive ACLs, misconfigured bucket policies, and missing encryption settings using AWS CLI, S3audit, and Prowler to enforce least-privilege data access controls.","domain":"cybersecurity","path":"skills/auditing-aws-s3-bucket-permissions"},{"name":"auditing-azure-active-directory-configuration","description":"Auditing Microsoft Entra ID (Azure Active Directory) configuration to identify risky authentication policies, overly permissive role assignments, stale accounts, conditional access gaps, and guest user risks using AzureAD PowerShell, Microsoft Graph API, and ScoutSuite.","domain":"cybersecurity","path":"skills/auditing-azure-active-directory-configuration"},{"name":"auditing-cloud-with-cis-benchmarks","description":"Audit AWS, Azure, and GCP environments against the CIS Foundations Benchmarks by running automated scans with tools like Prowler and ScoutSuite, interpreting failed controls, and tracking remediation for continuous compliance. Use when conducting a cloud security audit, validating CIS benchmark compliance (CIS v5 AWS, v4 Azure, v4 GCP), or setting up continuous cloud compliance monitoring.","domain":"cybersecurity","path":"skills/auditing-cloud-with-cis-benchmarks"},{"name":"auditing-entra-id-with-aadinternals","description":"Drive the AADInternals PowerShell toolkit to perform Microsoft Entra ID tenant reconnaissance, access-token acquisition across Microsoft APIs, and federation/AD FS backdoor testing (Golden SAML, T1606.002) for defensive validation. Use during an authorized Entra ID/Microsoft 365 red-team assessment to map external attack surface or verify AD FS signing certs resist Golden SAML.","domain":"cybersecurity","path":"skills/auditing-entra-id-with-aadinternals"},{"name":"auditing-foundry-smart-contract-security","description":"Pre-deployment security audit of Solidity smart contracts in a Foundry project. Combines static analysis (Slither, Aderyn), symbolic execution (Mythril), and property-based testing (forge fuzz + invariant tests with handlers) to catch reentrancy, access-control, oracle/price manipulation, and arithmetic bugs BEFORE deploying to an EVM chain. Also enforces key hygiene (no plaintext private keys, encrypted cast keystore) and a secure deploy workflow. Use when writing, reviewing, testing, or deploying Solidity/Foundry contracts, building a dApp, or working with forge/cast/anvil, MetaMask, or Web3/DeFi code.","domain":"cybersecurity","path":"skills/auditing-foundry-smart-contract-security"},{"name":"auditing-gcp-iam-permissions","description":"Auditing Google Cloud Platform IAM permissions to identify overly permissive bindings, primitive role usage, service account key proliferation, and cross-project access risks using gcloud CLI, Policy Analyzer, and IAM Recommender.","domain":"cybersecurity","path":"skills/auditing-gcp-iam-permissions"},{"name":"auditing-kubernetes-cluster-rbac","description":"Auditing Kubernetes cluster RBAC configurations to identify overly permissive roles, wildcard permissions, dangerous ClusterRoleBindings, service account abuse, and privilege escalation paths using kubectl, rbac-tool, KubiScan, and Kubeaudit.","domain":"cybersecurity","path":"skills/auditing-kubernetes-cluster-rbac"},{"name":"auditing-kubernetes-rbac-privilege-escalation","description":"Finds over-permissive RBAC roles and service-account token abuse paths in a Kubernetes cluster using kubectl auth can-i, rbac-police, kubectl-who-can, and rakkess, tracing which subjects can escalate toward cluster-admin. Use when reviewing who can escalate privileges in a cluster, hunting exploitable RoleBindings during an authorized review, or validating least privilege after an RBAC change. Keywords: RBAC, ClusterRoleBinding, service account token, auth can-i, rbac-police, escalate, bind, impersonate. Do not use for designing and applying hardened RBAC - use implementing-rbac-hardening-for-kubernetes.","domain":"cybersecurity","path":"skills/auditing-kubernetes-rbac-privilege-escalation"},{"name":"auditing-mcp-servers-for-tool-poisoning","description":"Audit MCP servers for tool poisoning, tool shadowing, rug pulls, SSRF, and unauthenticated exposure using Invariant Labs' mcp-scan for static/runtime scanning plus manual SSRF/auth checks and description pinning. Use before adding a new MCP server to an agent stack, when reviewing an internal MCP server, detecting rug pulls, or investigating an agent's unexpected tool-driven behavior.","domain":"cybersecurity","path":"skills/auditing-mcp-servers-for-tool-poisoning"},{"name":"auditing-terraform-infrastructure-for-security","description":"Auditing Terraform infrastructure-as-code for security misconfigurations using Checkov, tfsec, Terrascan, and OPA/Rego policies to detect overly permissive IAM policies, public resource exposure, missing encryption, and insecure defaults before cloud deployment.","domain":"cybersecurity","path":"skills/auditing-terraform-infrastructure-for-security"},{"name":"auditing-tls-certificate-transparency-logs","description":"Monitors Certificate Transparency (CT) logs to detect unauthorized certificate issuance, discover subdomains via CT data, and alert on suspicious certificate activity for owned domains. Uses the crt.sh API and direct CT log querying based on RFC 6962 to build continuous monitoring pipelines that catch rogue certificates, track CA behavior, and map the external attack surface. Activates for requests involving certificate transparency monitoring, CT log auditing, subdomain discovery via certificates, or certificate issuance alerting.","domain":"cybersecurity","path":"skills/auditing-tls-certificate-transparency-logs"},{"name":"auditing-uefi-firmware-with-chipsec","description":"Use Intel CHIPSEC to assess platform firmware configuration, SPI flash write protection, BIOS lock, SMM/SMRR, and Secure Boot variable state, dump SPI flash, and triage UEFI variables for firmware-level threats.","domain":"cybersecurity","path":"skills/auditing-uefi-firmware-with-chipsec"},{"name":"automating-ioc-enrichment","description":"Automates the enrichment of raw indicators of compromise with multi-source threat intelligence context using SOAR platforms, Python pipelines, or TIP playbooks to reduce analyst triage time and standardize enrichment outputs. Use when building automated enrichment workflows integrated with SIEM alerts, email submission pipelines, or bulk IOC processing from threat feeds. Activates for requests involving SOAR enrichment, Cortex XSOAR, Splunk SOAR, TheHive, Python enrichment pipelines, or automated IOC processing.","domain":"cybersecurity","path":"skills/automating-ioc-enrichment"},{"name":"benchmarking-kubernetes-with-kube-bench","description":"Installs and runs the kube-bench tool against a Kubernetes cluster as a Job, DaemonSet, or standalone binary, selecting the correct benchmark version and targets (control plane, etcd, kubelet, worker nodes) and emitting JSON or JUnit output for pipelines. Use when setting kube-bench up for the first time, choosing which benchmark version and node targets to run, wiring it into CI, or troubleshooting skipped or misdetected checks. Keywords: kube-bench, DaemonSet, --benchmark, --targets, JSON output, JUnit, CI integration. Do not use for interpreting the findings or producing an audit report - use performing-kubernetes-cis-benchmark-with-kube-bench.","domain":"cybersecurity","path":"skills/benchmarking-kubernetes-with-kube-bench"},{"name":"building-adversary-infrastructure-tracking-system","description":"Build an automated adversary infrastructure tracking system in Python (dnspython, python-whois, shodan, networkx) that pivots across passive DNS, certificate transparency logs, WHOIS records, and IP enrichment to map threat-actor C2 networks and flag newly registered domains matching known patterns. Use when pivoting from known indicators to discover related C2 infrastructure or maintaining a continuously updated map of a threat actor's network.","domain":"cybersecurity","path":"skills/building-adversary-infrastructure-tracking-system"},{"name":"building-attack-pattern-library-from-cti-reports","description":"Parse cyber threat intelligence reports (Mandiant, CrowdStrike, Talos, Microsoft) with stix2, mitreattack-python, and spaCy to extract adversary behaviors, map them to MITRE ATT&CK technique IDs, and build a searchable STIX 2.1 attack-pattern library with detection templates. Use when cataloging attack patterns from CTI reports for threat-informed detection engineering, or generating Sigma/YARA templates from documented behaviors.","domain":"cybersecurity","path":"skills/building-attack-pattern-library-from-cti-reports"},{"name":"building-automated-malware-submission-pipeline","description":"Builds an automated malware submission and analysis pipeline that collects suspicious files from endpoints and email gateways, submits them to sandbox environments and multi-engine scanners, and generates verdicts with IOCs for SIEM integration. Use when SOC teams need to scale malware analysis beyond manual sandbox submissions for high-volume alert triage.","domain":"cybersecurity","path":"skills/building-automated-malware-submission-pipeline"},{"name":"building-c2-infrastructure-with-sliver-framework","description":"Deploy and harden a Sliver C2 team server (BishopFox's Go-based adversary emulation framework) with multi-protocol listeners (mTLS, HTTP/S, DNS, WireGuard), redirectors, domain fronting, and multi-operator support for authorized red-team operations. Use when standing up resilient C2 for a red-team engagement or generating beacon/session implants that must survive blue-team detection.","domain":"cybersecurity","path":"skills/building-c2-infrastructure-with-sliver-framework"},{"name":"building-c2-redirector-infrastructure","description":"Build dumb-pipe and traffic-filtering C2 redirectors with nginx (proxy_pass) and Apache (mod_rewrite), deriving filter rules from a Malleable C2 profile, layering Let's Encrypt TLS, and applying OPSEC controls like domain fronting and UA/geo filtering. Use when standing up red-team C2 that must survive blue-team triage or ensuring only profile-matching implant traffic reaches the hidden team server.","domain":"cybersecurity","path":"skills/building-c2-redirector-infrastructure"},{"name":"building-cloud-siem-with-sentinel","description":"Deploy Microsoft Sentinel as a cloud-native SIEM/SOAR by configuring multi-cloud data connectors (AWS, Azure, GCP), writing KQL detection and hunting queries, and building automated Logic Apps response playbooks. Use when establishing a centralized SOC for multi-cloud environments, migrating from a legacy SIEM, or performing petabyte-scale threat hunting; not for AWS-only setups where Security Hub/GuardDuty suffice or for endpoint EDR needs.","domain":"cybersecurity","path":"skills/building-cloud-siem-with-sentinel"},{"name":"building-detection-rule-with-splunk-spl","description":"Build effective detection rules using Splunk Search Processing Language (SPL) correlation searches to identify security threats in SOC environments.","domain":"cybersecurity","path":"skills/building-detection-rule-with-splunk-spl"},{"name":"building-detection-rules-with-sigma","description":"Builds vendor-agnostic detection rules using the Sigma rule format for threat detection across SIEM platforms including Splunk, Elastic, and Microsoft Sentinel. Use when creating portable detection logic from threat intelligence, mapping rules to MITRE ATT&CK techniques, or converting community Sigma rules into platform-specific queries using sigmac or pySigma backends.","domain":"cybersecurity","path":"skills/building-detection-rules-with-sigma"},{"name":"building-devsecops-pipeline-with-gitlab-ci","description":"Configure a GitLab CI/CD pipeline that embeds SAST (Semgrep, SpotBugs, Gosec, Bandit, NodeJsScan), DAST, container scanning, dependency scanning, and secret detection via GitLab's managed security templates. Use when building a shift-left DevSecOps pipeline in GitLab, adding automated vulnerability scanning stages to .gitlab-ci.yml, or triaging scanner findings with GitLab Duo AI before deployment.","domain":"cybersecurity","path":"skills/building-devsecops-pipeline-with-gitlab-ci"},{"name":"building-identity-federation-with-saml-azure-ad","description":"Configure SAML 2.0 identity federation between on-premises Active Directory (via AD FS or a third-party IdP) and Microsoft Entra ID, covering federation models (AD FS, password hash sync, pass-through auth, third-party IdP) and the SAML authentication flow. Use when extending on-premises authentication authority to cloud resources or designing hybrid identity SSO architecture for Entra ID.","domain":"cybersecurity","path":"skills/building-identity-federation-with-saml-azure-ad"},{"name":"building-identity-governance-lifecycle-process","description":"Design identity governance and lifecycle (IGA) programs on platforms like SailPoint, Saviynt, or Entra ID Governance, covering joiner-mover-leaver (JML) automation, role mining, access requests, periodic recertification, and orphaned-account remediation sourced from an HR feed. Use when automating cross-system JML provisioning, remediating former-employee access, or building lifecycle processes for SOX, HIPAA, or GDPR compliance.","domain":"cybersecurity","path":"skills/building-identity-governance-lifecycle-process"},{"name":"building-incident-response-dashboard","description":"Builds real-time incident response dashboards in Splunk, Elastic, or Grafana to provide SOC analysts and leadership with situational awareness during active incidents, tracking affected systems, containment status, IOC spread, and response timeline. Use when IR teams need unified visibility during incident coordination and post-incident reporting.","domain":"cybersecurity","path":"skills/building-incident-response-dashboard"},{"name":"building-incident-response-playbook","description":"Designs and documents structured incident response playbooks with step-by-step procedures per incident type, decision trees, escalation criteria, RACI matrices, and SOAR platform integration, aligned to NIST SP 800-61r3 and SANS PICERL. Use when creating or maturing an IR program, documenting response runbooks for a new incident type, or designing SOAR playbooks.","domain":"cybersecurity","path":"skills/building-incident-response-playbook"},{"name":"building-incident-timeline-with-timesketch","description":"Build collaborative forensic incident timelines using Timesketch to ingest, normalize, and analyze multi-source event data (including Plaso output) for attack chain reconstruction and investigation documentation. Use when reconstructing the sequence of events during an incident investigation or when multiple analysts need to jointly tag, annotate, and search a shared DFIR timeline.","domain":"cybersecurity","path":"skills/building-incident-timeline-with-timesketch"},{"name":"building-ioc-defanging-and-sharing-pipeline","description":"Build an automated pipeline that ingests raw IOCs (URLs, IPs, domains, emails), normalizes and deduplicates them, then produces defanged renderings for safe human reading alongside canonical STIX 2.1 bundles distributed via TAXII servers, MISP, or email reports. Use when preparing indicators of compromise for safe analyst sharing or automating threat intel distribution to TAXII/MISP feeds.","domain":"cybersecurity","path":"skills/building-ioc-defanging-and-sharing-pipeline"},{"name":"building-ioc-enrichment-pipeline-with-opencti","description":"Build an automated IOC enrichment pipeline on OpenCTI (STIX 2.1 native threat intel platform) using its internal enrichment connectors to pull context from VirusTotal, Shodan, AbuseIPDB, and GreyNoise, correlate indicators with known actors/campaigns, and score them for analyst prioritization. Use when deploying OpenCTI or automating enrichment and confidence scoring of newly ingested indicators.","domain":"cybersecurity","path":"skills/building-ioc-enrichment-pipeline-with-opencti"},{"name":"building-malware-incident-communication-template","description":"Build structured communication templates for malware incidents (ransomware, wiper, trojan, worm), covering internal stakeholder notifications, executive briefings, technical advisories for IT teams, customer notifications, and regulatory disclosures, with severity-based escalation procedures. Use when drafting or standardizing incident communications and notification workflows for a malware outbreak.","domain":"cybersecurity","path":"skills/building-malware-incident-communication-template"},{"name":"building-patch-tuesday-response-process","description":"Establish a repeatable operational process for triaging, testing, and deploying Microsoft Patch Tuesday security updates (Windows, Office, Exchange, SQL Server, Azure) via WSUS/SCCM within risk-based remediation SLAs, from advisory review through validation. Use when building or improving a monthly patch management workflow or prioritizing which CVEs to remediate first.","domain":"cybersecurity","path":"skills/building-patch-tuesday-response-process"},{"name":"building-phishing-reporting-button-workflow","description":"Implement a phishing report button (Microsoft 365 built-in Report button or third-party like KnowBe4/Cofense) in email clients with a SOAR-driven automated triage workflow that classifies reported emails, extracts IOCs, takes remediation actions, and gives feedback to reporters. Use when deploying user-reported phishing intake or automating triage of the resulting reporting mailbox.","domain":"cybersecurity","path":"skills/building-phishing-reporting-button-workflow"},{"name":"building-ransomware-playbook-with-cisa-framework","description":"Builds a structured ransomware incident response playbook aligned with the CISA StopRansomware Guide and NIST Cybersecurity Framework, covering preparation, detection, containment, eradication, recovery, and post-incident phases with actionable checklists. Use when creating or updating a ransomware playbook, running a CISA-aligned readiness assessment, or validating response steps during a tabletop exercise.","domain":"cybersecurity","path":"skills/building-ransomware-playbook-with-cisa-framework"},{"name":"building-red-team-c2-infrastructure-with-havoc","description":"Deploy and configure the Havoc C2 framework (teamserver, HTTPS/HTTP/SMB listeners, Nginx redirectors, and Demon agents) with malleable traffic profiles and OPSEC-hardened infrastructure for authorized red team operations. Use when standing up or hardening Havoc C2 infrastructure for a written, authorized adversary emulation engagement.","domain":"cybersecurity","path":"skills/building-red-team-c2-infrastructure-with-havoc"},{"name":"building-role-mining-for-rbac-optimization","description":"Apply bottom-up and top-down role mining techniques, including clustering algorithms and formal concept analysis, to discover optimal RBAC roles from existing user-permission assignments, consolidating overlapping roles and enforcing least privilege. Use when an identity program needs to reduce role explosion or redesign its RBAC role set from access data.","domain":"cybersecurity","path":"skills/building-role-mining-for-rbac-optimization"},{"name":"building-soc-escalation-matrix","description":"Build a structured SOC escalation matrix defining severity tiers, response SLAs, tiered escalation paths, and notification procedures for security incidents, using context-driven criteria that combine business risk, asset criticality, and data sensitivity. Use when designing or revising how a SOC triages and escalates incidents across analyst tiers.","domain":"cybersecurity","path":"skills/building-soc-escalation-matrix"},{"name":"building-soc-metrics-and-kpi-tracking","description":"Builds SOC performance metrics and KPI tracking dashboards measuring Mean Time to Detect (MTTD), Mean Time to Respond (MTTR), alert quality ratios, analyst productivity, and detection coverage using SIEM data. Use when SOC leadership needs operational visibility, continuous improvement tracking, or executive-level reporting on security operations effectiveness.","domain":"cybersecurity","path":"skills/building-soc-metrics-and-kpi-tracking"},{"name":"building-soc-playbook-for-ransomware","description":"Builds a structured SOC incident response playbook for ransomware attacks covering detection, containment, eradication, and recovery phases with specific SIEM queries, isolation procedures, and decision trees. Use when SOC teams need formalized response procedures for ransomware incidents aligned to NIST SP 800-61 and MITRE ATT&CK ransomware techniques.","domain":"cybersecurity","path":"skills/building-soc-playbook-for-ransomware"},{"name":"building-super-timelines-with-plaso","description":"Generate forensic super-timelines with Plaso's log2timeline.py, pinfo.py, psort.py, and psteal.py CLI tools (fusing file-system MACB, registry, EVTX, browser history, prefetch, LNK, and more), then triage and filter the results in Timesketch. Use when reconstructing the full sequence of events on a compromised or forensically imaged host during a DFIR investigation.","domain":"cybersecurity","path":"skills/building-super-timelines-with-plaso"},{"name":"building-threat-actor-profile-from-osint","description":"Build threat actor profiles by collecting OSINT from vendor reports, paste sites, dark web forums, social media, and code repos, correlating indicators, mapping adversary infrastructure with tools like Maltego and SpiderFoot, and producing structured dossiers of motivations, capabilities, infrastructure, and TTPs. Use when performing attribution or building an adversary dossier from open-source intelligence.","domain":"cybersecurity","path":"skills/building-threat-actor-profile-from-osint"},{"name":"building-threat-feed-aggregation-with-misp","description":"Deploy MISP via Docker and configure feeds from sources like abuse.ch, AlienVault OTX, and CIRCL to aggregate, correlate, and distribute threat intelligence, including automated feed synchronization and STIX/TAXII-based integration with Splunk, Elasticsearch, and SOAR platforms. Use when standing up centralized IOC management or wiring multi-source threat feeds into a SIEM.","domain":"cybersecurity","path":"skills/building-threat-feed-aggregation-with-misp"},{"name":"building-threat-hunt-hypothesis-framework","description":"Build a systematic threat-hunt workflow that turns threat intelligence and ATT&CK gap analysis into testable hypotheses, then executes and validates them via EDR/SIEM queries (CrowdStrike, Defender, Splunk, Elastic, Sysmon, Velociraptor, Sigma) and documents findings in a standardized hunt report. Use when planning or running a proactive threat hunt or scoping compromise from an intel- or anomaly-driven lead.","domain":"cybersecurity","path":"skills/building-threat-hunt-hypothesis-framework"},{"name":"building-threat-intelligence-enrichment-in-splunk","description":"Build automated IOC enrichment pipelines in Splunk Enterprise Security by ingesting threat feeds into KV Store collections and correlating them against security events via lookup tables, modular inputs, and the Threat Intelligence Framework. Use when wiring threat intel into Splunk correlation searches to flag IOC matches and cut SOC triage time.","domain":"cybersecurity","path":"skills/building-threat-intelligence-enrichment-in-splunk"},{"name":"building-threat-intelligence-feed-integration","description":"Builds automated threat intelligence feed integration pipelines connecting STIX/TAXII feeds, open-source threat intel, and commercial TI platforms into SIEM and security tools for real-time IOC matching and alerting. Use when SOC teams need to operationalize threat intelligence by automating feed ingestion, normalization, scoring, and distribution to detection systems.","domain":"cybersecurity","path":"skills/building-threat-intelligence-feed-integration"},{"name":"building-threat-intelligence-platform","description":"Design and deploy a Threat Intelligence Platform (TIP) by integrating open-source CTI tools (MISP, OpenCTI, TheHive, Cortex) into a unified system with feed ingestion pipelines, enrichment workflows, STIX/TAXII interoperability, and analyst dashboards. Use when architecting or standing up a centralized CTI platform to collect, analyze, and disseminate threat intelligence across a security team.","domain":"cybersecurity","path":"skills/building-threat-intelligence-platform"},{"name":"building-vulnerability-aging-and-sla-tracking","description":"Implement a vulnerability aging dashboard and SLA tracking system that measures time-to-remediation against severity-based deadlines (e.g. 14 days critical, 30 days high, 60 days medium, 90 days low), with automated escalations and compliance metrics reporting. Use when designing SLA policies, building aging/remediation dashboards, or proving compliance with remediation timelines.","domain":"cybersecurity","path":"skills/building-vulnerability-aging-and-sla-tracking"},{"name":"building-vulnerability-dashboard-with-defectdojo","description":"Deploy DefectDojo as a centralized vulnerability management dashboard that ingests findings from 200+ security scanners, deduplicates results, tracks remediation metrics, and integrates with CI/CD, Jira ticketing, and Slack notifications via its REST API. Use when consolidating scanner output into one dashboard or automating vulnerability ticketing and executive reporting.","domain":"cybersecurity","path":"skills/building-vulnerability-dashboard-with-defectdojo"},{"name":"building-vulnerability-exception-tracking-system","description":"Build a vulnerability exception and risk acceptance tracking system covering approval workflows, compensating controls documentation, and automatic expiration for vulnerabilities that miss SLA remediation timelines. Use when standing up a governance process for risk acceptance and exception approvals to support PCI DSS, SOC 2, or NIST CSF compliance.","domain":"cybersecurity","path":"skills/building-vulnerability-exception-tracking-system"},{"name":"building-vulnerability-scanning-workflow","description":"Builds a structured vulnerability scanning workflow using tools like Nessus, Qualys, and OpenVAS to discover, prioritize, and track remediation of security vulnerabilities across infrastructure. Use when SOC teams need to establish recurring vulnerability assessment processes, integrate scan results with SIEM alerting, and build remediation tracking dashboards.","domain":"cybersecurity","path":"skills/building-vulnerability-scanning-workflow"},{"name":"bypassing-authentication-with-forced-browsing","description":"Discovering and accessing unprotected pages, APIs, and administrative interfaces by enumerating URLs and bypassing authentication controls during authorized security assessments.","domain":"cybersecurity","path":"skills/bypassing-authentication-with-forced-browsing"},{"name":"coercing-authentication-with-coercer-petitpotam","description":"Trigger machine account authentication with PetitPotam (MS-EFSR) and Coercer (MS-RPRN, MS-DFSNM, MS-FSRVP, MS-EVEN) via Coercer's scan/coerce/fuzz modes, feeding the coerced NTLM auth into a relay against AD CS Web Enrollment (ESC8), LDAP (RBCD), or SMB. Use in authorized engagements to complete a coercion-relay chain against a Domain Controller, or to validate coercion detections and signing/EPA mitigations.","domain":"cybersecurity","path":"skills/coercing-authentication-with-coercer-petitpotam"},{"name":"collecting-indicators-of-compromise","description":"Systematically collects, categorizes, and distributes indicators of compromise (IOCs) during and after security incidents to enable detection, blocking, and threat intelligence sharing. Covers network, host, email, and behavioral indicators using STIX/TAXII formats and threat intelligence platforms. Activates for requests involving IOC collection, indicator extraction, threat indicator sharing, compromise indicators, STIX export, or IOC enrichment.","domain":"cybersecurity","path":"skills/collecting-indicators-of-compromise"},{"name":"collecting-open-source-intelligence","description":"Collects and synthesizes open-source intelligence (OSINT) about threat actors, malicious infrastructure, and attack campaigns using publicly available data sources, passive reconnaissance tools, and dark web monitoring. Use when investigating external threat actor infrastructure, performing pre-engagement reconnaissance for authorized red team assessments, or enriching CTI reports with publicly available adversary context. Activates for requests involving Maltego, Shodan, OSINT framework, SpiderFoot, or infrastructure reconnaissance.","domain":"cybersecurity","path":"skills/collecting-open-source-intelligence"},{"name":"collecting-threat-intelligence-with-misp","description":"Deploy MISP, configure threat feeds (MISP community, freetext, TAXII, CSV), and use the PyMISP API to programmatically fetch, add, and search events and IOCs, building automated collection pipelines that aggregate indicators from community and commercial sources. Use when gathering, storing, or correlating IOCs and threat intelligence, or when scripting MISP ingestion via PyMISP.","domain":"cybersecurity","path":"skills/collecting-threat-intelligence-with-misp"},{"name":"collecting-volatile-evidence-from-compromised-host","description":"Collect volatile forensic evidence from a compromised host by following the order of volatility, preserving memory, network connections, running processes, and system state with documented chain of custody before they are lost. Use before isolating, shutting down, or remediating a compromised host, especially when fileless or memory-resident malware is suspected, root cause analysis is needed, or the evidence must hold up in legal proceedings.","domain":"cybersecurity","path":"skills/collecting-volatile-evidence-from-compromised-host"},{"name":"conducting-api-security-testing","description":"Conducts security testing of REST, GraphQL, and gRPC APIs to identify vulnerabilities in authentication, authorization, rate limiting, input validation, and business logic. The tester uses the OWASP API Security Top 10 as the testing framework, combining Burp Suite interception with Postman collections and custom scripts to test endpoint security at every privilege level. Activates for requests involving API security testing, REST API pentest, GraphQL security assessment, or API vulnerability testing.","domain":"cybersecurity","path":"skills/conducting-api-security-testing"},{"name":"conducting-cloud-incident-response","description":"Respond to security incidents in AWS, Azure, and GCP via identity-based containment, cloud-native log analysis (CloudTrail, Azure Activity Logs, GCP Audit Logs), resource isolation, and forensic evidence acquisition adapted for ephemeral cloud infrastructure. Use when CSPM alerts or audit logs show compromised cloud credentials, unauthorized IAM changes, or a breach spanning cloud services.","domain":"cybersecurity","path":"skills/conducting-cloud-incident-response"},{"name":"conducting-cloud-penetration-testing","description":"This skill outlines methodologies for performing authorized penetration testing against AWS, Azure, and GCP cloud environments. It covers understanding the shared responsibility model for testing scope, leveraging cloud-specific attack tools like Pacu and ScoutSuite, exploiting IAM misconfigurations, testing for SSRF to cloud metadata services, and reporting findings aligned to MITRE ATT&CK Cloud matrix.","domain":"cybersecurity","path":"skills/conducting-cloud-penetration-testing"},{"name":"conducting-cyber-risk-assessment-with-nist-800-30","description":"Conduct a defensible cybersecurity risk assessment using the NIST SP 800-30 Rev 1 methodology: prepare scope and a risk model, identify threat sources and threat events, identify vulnerabilities and predisposing conditions, determine likelihood and impact, compute risk, and communicate results as a prioritized risk register. Use when an organization needs an actual risk *assessment* (not a maturity score), when a control framework (CSF, ISO 27001, RMF, SOC 2, PCI) requires a documented risk analysis as input, when leadership asks \"what are our top risks and how bad are they\", when assessing risk for a new system or major change, or when building a risk register from scratch. This is the methodology that feeds framework selection, ATO packages, and treatment decisions. Keywords: risk assessment, NIST 800-30, threat modeling, likelihood and impact, risk register, risk analysis, threat sources, vulnerabilities, risk determination, qualitative risk, risk matrix, residual risk, risk treatment.","domain":"cybersecurity","path":"skills/conducting-cyber-risk-assessment-with-nist-800-30"},{"name":"conducting-domain-persistence-with-dcsync","description":"Perform DCSync attacks by abusing MS-DRSR replication rights (DS-Replication-Get-Changes/-All) to impersonate a Domain Controller and extract KRBTGT, Domain Admin, and service account hashes for Golden Ticket forging, typically with Mimikatz. Use in authorized engagements after finding principals with replication rights, to establish long-term domain persistence, or to validate detections for replication abuse.","domain":"cybersecurity","path":"skills/conducting-domain-persistence-with-dcsync"},{"name":"conducting-external-reconnaissance-with-osint","description":"Conduct external recon using OSINT techniques to map an organization's external attack surface without touching target systems, gathering DNS records, certificate transparency logs, search results, social media, code repositories, and breach databases into a target profile. Use for the passive info-gathering phase of a pentest, external footprinting, or collecting employee/email intel for a social engineering campaign.","domain":"cybersecurity","path":"skills/conducting-external-reconnaissance-with-osint"},{"name":"conducting-full-scope-red-team-engagement","description":"Plan and execute a comprehensive, MITRE ATT&CK-aligned red team engagement spanning threat modeling, reconnaissance, initial access, and post-exploitation to evaluate an organization's detection, prevention, and response against APT-style behavior. Use when scoping or running a full-scope, objective-based engagement, or purple-teaming against a specific threat actor's TTPs.","domain":"cybersecurity","path":"skills/conducting-full-scope-red-team-engagement"},{"name":"conducting-gdpr-compliance-assessment","description":"Conduct comprehensive GDPR compliance assessments by evaluating data processing activities against EU Regulation 2016/679, including Article 30 records of processing, lawful basis validation, data subject rights implementation, Data Protection Impact Assessments (DPIAs) under Article 35, breach notification procedures, international transfer safeguards (SCCs, adequacy decisions), and technical/organizational measures under Article 32. Use when processing personal data of EU residents, preparing for supervisory authority audits, implementing privacy-by-design for new systems, scoping compliance gaps for M&A due diligence, assessing third-party processors, or responding to data subject access requests at scale. Incorporates 2026 guidance from ICO, EDPB, and post-Data (Use and Access) Act 2025 UK-GDPR considerations. Do not use for implementing specific Article 32 controls \u2014 use implementing-gdpr-data-protection-controls; or for DSAR automation \u2014 use implementing-gdpr-data-subject-access-request.","domain":"cybersecurity","path":"skills/conducting-gdpr-compliance-assessment"},{"name":"conducting-internal-network-penetration-test","description":"Execute an internal network penetration test simulating an insider threat or post-breach attacker to identify lateral movement paths, privilege escalation vectors, and sensitive data exposure within the corporate network.","domain":"cybersecurity","path":"skills/conducting-internal-network-penetration-test"},{"name":"conducting-internal-reconnaissance-with-bloodhound-ce","description":"Conduct internal Active Directory reconnaissance using BloodHound Community Edition's graph database with the SharpHound (AD) and AzureHound (Entra ID) collectors, mapping ACLs, sessions, and group memberships into attack paths from a low-privileged foothold to Domain Admin. Use after an initial AD foothold to identify privilege escalation chains, or to validate that AD hardening closed known attack paths.","domain":"cybersecurity","path":"skills/conducting-internal-reconnaissance-with-bloodhound-ce"},{"name":"conducting-malware-incident-response","description":"Respond to malware infections across enterprise endpoints by identifying the malware family, determining infection vectors, assessing spread, and executing containment, analysis, eradication, and recovery procedures aligned to MITRE ATT&CK. Use when responding to a confirmed or suspected malware infection, including trojan/worm/ransomware outbreaks, malware triage, or infected endpoint remediation.","domain":"cybersecurity","path":"skills/conducting-malware-incident-response"},{"name":"conducting-man-in-the-middle-attack-simulation","description":"Simulates man-in-the-middle attacks using Ettercap, mitmproxy, and Bettercap in authorized environments to intercept, analyze, and modify network traffic for testing encryption enforcement, certificate validation, and detection capabilities.","domain":"cybersecurity","path":"skills/conducting-man-in-the-middle-attack-simulation"},{"name":"conducting-memory-forensics-with-volatility","description":"Performs memory forensics analysis using Volatility 3 to extract evidence of malware execution, process injection, network connections, and credential theft from RAM dumps captured during incident response. Covers memory acquisition, process analysis, DLL inspection, and malware detection. Activates for requests involving memory forensics, RAM analysis, Volatility framework, memory dump investigation, volatile evidence analysis, or live memory acquisition.","domain":"cybersecurity","path":"skills/conducting-memory-forensics-with-volatility"},{"name":"conducting-mobile-app-penetration-test","description":"Conducts penetration testing of iOS and Android mobile applications following the OWASP Mobile Application Security Testing Guide (MASTG) to identify vulnerabilities in data storage, network communication, authentication, cryptography, and platform-specific security controls. The tester performs static analysis of application binaries, dynamic analysis at runtime, and API security testing to evaluate the complete mobile attack surface. Activates for requests involving mobile app pentest, iOS security assessment, Android security testing, or OWASP MASTG assessment.","domain":"cybersecurity","path":"skills/conducting-mobile-app-penetration-test"},{"name":"conducting-network-penetration-test","description":"Conducts comprehensive network penetration tests against authorized target environments by performing host discovery, port scanning, service enumeration, vulnerability identification, and controlled exploitation to assess the security posture of network infrastructure. The tester follows PTES methodology from reconnaissance through post-exploitation and reporting. Activates for requests involving network pentest, infrastructure security assessment, internal network testing, or external perimeter testing.","domain":"cybersecurity","path":"skills/conducting-network-penetration-test"},{"name":"conducting-pass-the-ticket-attack","description":"Perform Pass-the-Ticket (PtT) lateral movement by extracting Kerberos TGT/TGS tickets from LSASS memory on a compromised host and injecting them into another session to impersonate the ticket owner without knowing their password. Use during authorized post-exploitation to move laterally or escalate access via stolen Kerberos tickets, or to validate detections for Kerberos ticket theft and reuse.","domain":"cybersecurity","path":"skills/conducting-pass-the-ticket-attack"},{"name":"conducting-phishing-incident-response","description":"Respond to phishing incidents by analyzing reported emails, extracting indicators, sandboxing URLs/attachments, assessing credential compromise, quarantining malicious messages organization-wide, and remediating affected accounts. Use when investigating a reported phishing or credential-phishing email, a suspected spearphishing incident, or when a mailbox-wide purge and account remediation is needed.","domain":"cybersecurity","path":"skills/conducting-phishing-incident-response"},{"name":"conducting-post-incident-lessons-learned","description":"Facilitate structured post-incident reviews to identify root causes, document what worked and failed, and produce actionable recommendations to improve future incident response.","domain":"cybersecurity","path":"skills/conducting-post-incident-lessons-learned"},{"name":"conducting-social-engineering-penetration-test","description":"Design and execute a social engineering penetration test combining OSINT-driven target profiling with phishing, vishing, smishing, and physical pretexting campaigns using tools like GoPhish, the Social Engineer Toolkit (SET), and Evilginx to measure human security resilience. Use when scoping or running an authorized human-attack-surface test and reporting results to identify security awareness training gaps.","domain":"cybersecurity","path":"skills/conducting-social-engineering-penetration-test"},{"name":"conducting-social-engineering-pretext-call","description":"Plan and execute authorized vishing (voice phishing) pretext calls to assess employee susceptibility to social engineering and evaluate security awareness controls.","domain":"cybersecurity","path":"skills/conducting-social-engineering-pretext-call"},{"name":"conducting-spearphishing-simulation-campaign","description":"Run a targeted spearphishing simulation for initial access by developing OSINT-derived pretexts, building payloads (HTML smuggling, macro docs, ISO/LNK, OneNote, QR codes), standing up look-alike-domain email infrastructure with SPF/DKIM/DMARC via GoPhish, and tracking results. Use in authorized engagements to gain initial access via personalized phishing against specific individuals, distinct from broad campaigns.","domain":"cybersecurity","path":"skills/conducting-spearphishing-simulation-campaign"},{"name":"conducting-wireless-network-penetration-test","description":"Conducts authorized wireless network penetration tests to assess the security of WiFi infrastructure by testing for weak encryption protocols, captive portal bypasses, evil twin attacks, WPA2/WPA3 handshake capture, rogue access point detection, and client-side attacks. The tester evaluates wireless authentication, network segmentation, and the effectiveness of wireless intrusion detection systems. Activates for requests involving wireless pentest, WiFi security assessment, WPA2/WPA3 testing, or rogue access point detection.","domain":"cybersecurity","path":"skills/conducting-wireless-network-penetration-test"},{"name":"configuring-active-directory-tiered-model","description":"Implement Microsoft's Enhanced Security Admin Environment (ESAE) tiered administration model for Active Directory, covering Tier 0/1/2 separation, privileged access workstations (PAWs), administrative forest design, and authentication policy silos. Use when designing or hardening AD privileged-access architecture, segmenting Domain/Enterprise Admin accounts into tiers, or containing lateral movement and credential theft (pass-the-hash, Kerberoasting, golden tickets).","domain":"cybersecurity","path":"skills/configuring-active-directory-tiered-model"},{"name":"configuring-aws-verified-access-for-ztna","description":"Configure AWS Verified Access to provide VPN-less zero trust network access to internal apps, combining identity trust providers (IAM Identity Center, Okta/OIDC), device posture providers (CrowdStrike, Jamf), Cedar policy authoring, and Terraform deployment. Use when replacing VPN access with ZTNA, writing Cedar access policies, or deploying Verified Access instances, groups, and endpoints across AWS accounts.","domain":"cybersecurity","path":"skills/configuring-aws-verified-access-for-ztna"},{"name":"configuring-certificate-authority-with-openssl","description":"Build a two-tier PKI Certificate Authority hierarchy (offline Root CA plus issuing Intermediate CA) using OpenSSL and the Python cryptography library, covering certificate extensions, CRL distribution points, OCSP responder configuration, and certificate policy management. Use when standing up an internal CA, issuing or revoking X.509 certificates, or designing PKI trust hierarchies for TLS, code-signing, or client-authentication use cases.","domain":"cybersecurity","path":"skills/configuring-certificate-authority-with-openssl"},{"name":"configuring-host-based-intrusion-detection","description":"Configures host-based intrusion detection systems (HIDS) to monitor endpoint file integrity, system calls, and configuration changes for security violations. Use when deploying OSSEC, Wazuh, or AIDE for endpoint monitoring, building file integrity monitoring (FIM) policies, or meeting compliance requirements for change detection. Activates for requests involving HIDS configuration, file integrity monitoring, OSSEC/Wazuh deployment, or host-based detection.","domain":"cybersecurity","path":"skills/configuring-host-based-intrusion-detection"},{"name":"configuring-hsm-for-key-storage","description":"Configures Hardware Security Modules for cryptographic key storage using the PKCS#11 standard interface, covering key generation, signing, encryption, and key management on physical HSMs and SoftHSM2 for development. Use when protecting cryptographic keys so they never leave a hardened device boundary, or when building and testing PKCS#11-based key management workflows.","domain":"cybersecurity","path":"skills/configuring-hsm-for-key-storage"},{"name":"configuring-identity-aware-proxy-with-google-iap","description":"Configures Google Cloud Identity-Aware Proxy (IAP) via gcloud to enforce per-request identity verification on Compute Engine, App Engine, Cloud Run, and GKE, including IAM bindings, Access Context Manager access levels, session/reauth settings, and service-account programmatic access. Use when replacing VPN access with identity-based access to GCP backends or configuring context-aware, zero-trust policies for Google Cloud services.","domain":"cybersecurity","path":"skills/configuring-identity-aware-proxy-with-google-iap"},{"name":"configuring-ldap-security-hardening","description":"Hardens LDAP directory services against credential harvesting, LDAP injection, anonymous binding, and channel-binding bypass by enforcing LDAPS, channel binding, and LDAP signing. Use when securing an LDAP or Active Directory environment against these attack classes or auditing directory service configurations for these vulnerabilities.","domain":"cybersecurity","path":"skills/configuring-ldap-security-hardening"},{"name":"configuring-microsegmentation-for-zero-trust","description":"Configures microsegmentation policies to enforce least-privilege workload-to-workload access using tools such as VMware NSX, Illumio, and Calico, preventing lateral movement in zero trust architectures. Use when designing or implementing network microsegmentation as part of a zero trust architecture aligned with NIST SP 800-207.","domain":"cybersecurity","path":"skills/configuring-microsegmentation-for-zero-trust"},{"name":"configuring-multi-factor-authentication-with-duo","description":"Deploys Cisco Duo multi-factor authentication across enterprise applications, VPN, RDP, and SSH access points, covering Duo Authentication Proxy setup, adaptive authentication policies, device trust assessment, and phishing-resistant WebAuthn/FIDO2 deployment aligned with NIST 800-63B AAL2/AAL3. Use when adding or hardening MFA for remote access, VPN, or privileged logins with Duo.","domain":"cybersecurity","path":"skills/configuring-multi-factor-authentication-with-duo"},{"name":"configuring-network-segmentation-with-vlans","description":"Designs and implements VLAN-based (802.1Q) network segmentation on managed switches to isolate zones such as corporate, servers, DMZ, guest, and IoT, and to limit lateral movement paths. Use when segmenting an enterprise network into isolated security zones, meeting compliance mandates (PCI-DSS, HIPAA, SOC 2) for network isolation, or reducing blast radius from a security incident.","domain":"cybersecurity","path":"skills/configuring-network-segmentation-with-vlans"},{"name":"configuring-oauth2-authorization-flow","description":"Configures secure OAuth 2.0 authorization flows, including Authorization Code with PKCE, Client Credentials, and Device Authorization Grant, covering flow selection, PKCE implementation, token lifecycle management, and scope design per OAuth 2.1. Use when implementing or hardening OAuth 2.0 authentication/authorization for web, mobile, SPA, or machine-to-machine clients.","domain":"cybersecurity","path":"skills/configuring-oauth2-authorization-flow"},{"name":"configuring-pfsense-firewall-rules","description":"Configures pfSense firewall rules, NAT policies, IPsec/OpenVPN tunnels, and traffic shaping to enforce network segmentation and control traffic between zones such as DMZ, internal, guest, and IoT. Use when deploying a pfSense perimeter or internal firewall, setting up port-forwarding NAT, configuring site-to-site or remote-access VPNs, or applying QoS/bandwidth policies.","domain":"cybersecurity","path":"skills/configuring-pfsense-firewall-rules"},{"name":"configuring-snort-ids-for-intrusion-detection","description":"Installs, configures, and tunes Snort 3 to monitor network traffic for malicious activity using custom and community rulesets, preprocessors, and alert output plugins. Use when deploying network-based intrusion detection at key boundaries, writing custom Snort rules, tuning rulesets to reduce false positives, or integrating Snort alerts with a SIEM.","domain":"cybersecurity","path":"skills/configuring-snort-ids-for-intrusion-detection"},{"name":"configuring-suricata-for-network-monitoring","description":"Deploys and configures Suricata IDS/IPS with Emerging Threats rulesets, EVE JSON logging, and custom rules for high-throughput, protocol-aware traffic inspection (HTTP, TLS, DNS, SMB) and SIEM integration. Use when running Suricata in IDS or inline IPS mode to detect or block malicious traffic, or when combining signature-based and protocol anomaly detection with file extraction.","domain":"cybersecurity","path":"skills/configuring-suricata-for-network-monitoring"},{"name":"configuring-tls-1-3-for-secure-communications","description":"Configures TLS 1.3 (RFC 8446) on servers, covering cipher suite and key-exchange group selection, and validates the resulting configuration with openssl s_client and testssl.sh. Use when deploying or hardening TLS 1.3 for secure communications, or when testing a server for common TLS misconfigurations and vulnerabilities.","domain":"cybersecurity","path":"skills/configuring-tls-1-3-for-secure-communications"},{"name":"configuring-windows-defender-advanced-settings","description":"Configures Microsoft Defender for Endpoint (MDE) advanced protection settings including attack surface reduction rules, controlled folder access, network protection, and exploit protection. Use when hardening Windows endpoints beyond default Defender settings, deploying enterprise-grade endpoint protection, or meeting compliance requirements for advanced malware defense. Activates for requests involving Windows Defender configuration, ASR rules, MDE tuning, or Microsoft endpoint security.","domain":"cybersecurity","path":"skills/configuring-windows-defender-advanced-settings"},{"name":"configuring-windows-event-logging-for-detection","description":"Configures Windows Event Logging with advanced audit policies to generate high-fidelity security events for threat detection and forensic investigation. Use when enabling audit policies for logon events, process creation, privilege use, and object access to feed SIEM detection rules. Activates for requests involving Windows audit policy, event log configuration, security logging, or detection-oriented logging.","domain":"cybersecurity","path":"skills/configuring-windows-event-logging-for-detection"},{"name":"configuring-zscaler-private-access-for-ztna","description":"Configures Zscaler Private Access (ZPA) to replace traditional VPN with zero trust network access by deploying App Connectors, defining application segments, configuring identity- and device-posture-based access policies, and integrating with IdPs. Use when replacing VPN concentrators with ZTNA or providing remote users secure access to internal applications without network-level connectivity.","domain":"cybersecurity","path":"skills/configuring-zscaler-private-access-for-ztna"},{"name":"containing-active-breach","description":"Executes containment strategies to stop active adversary operations and prevent lateral movement during a confirmed security breach. Implements short-term and long-term containment using network segmentation, endpoint isolation, credential revocation, and access control modifications. Activates for requests involving breach containment, lateral movement prevention, network isolation, active threat containment, or live incident response.","domain":"cybersecurity","path":"skills/containing-active-breach"},{"name":"continuous-llm-red-teaming-with-promptfoo","description":"Wires Promptfoo and DeepTeam into CI/CD for automated, repeatable red-teaming of LLM apps against OWASP LLM Top 10, OWASP Agentic, and MITRE ATLAS presets, failing the build when jailbreak or injection vulnerabilities regress. Use for continuous adversarial testing in CI/CD, a merge-blocking security gate, or comparing model/prompt versions for compliance reporting.","domain":"cybersecurity","path":"skills/continuous-llm-red-teaming-with-promptfoo"},{"name":"correlating-security-events-in-qradar","description":"Correlates security events in IBM QRadar SIEM using AQL (Ariel Query Language), custom rules, building blocks, and offense management to detect multi-stage attacks across network, endpoint, and application log sources. Use when SOC analysts need to investigate QRadar offenses, build correlation rules, or tune detection logic for reducing false positives.","domain":"cybersecurity","path":"skills/correlating-security-events-in-qradar"},{"name":"correlating-threat-campaigns","description":"Correlates disparate security incidents, IOCs, and adversary behaviors across time and organizations to identify unified threat campaigns, attribute them to common threat actors, and extract shared indicators for improved detection. Use when multiple incidents exhibit overlapping indicators, when sector-wide attack campaigns require cross-organizational analysis, or when building campaign-level intelligence products. Activates for requests involving campaign analysis, incident clustering, cross-organizational IOC correlation, or MISP correlation engine.","domain":"cybersecurity","path":"skills/correlating-threat-campaigns"},{"name":"defending-llms-with-guardrails","description":"Deploys Llama Guard 3 safety classification, NeMo Guardrails programmable dialogue rails, and LLM Guard input/output scanner pipelines as complementary runtime defenses that inspect and constrain LLM prompts and responses. Use when adding a production runtime safety layer to an LLM, RAG, or agent application to block jailbreaks, prompt injection (OWASP LLM01), toxic content, or sensitive-data leakage before it reaches or leaves the model.","domain":"cybersecurity","path":"skills/defending-llms-with-guardrails"},{"name":"deobfuscating-javascript-malware","description":"Deobfuscates malicious JavaScript found in phishing pages, web skimmers, and dropper scripts by reversing encoding layers, eval chains, string manipulation, and control-flow obfuscation to reveal the original malicious logic. Use when investigating a phishing page's obfuscated JavaScript, analyzing a Magecart-style web skimmer, or deobfuscating a JavaScript dropper that fetches second-stage malware.","domain":"cybersecurity","path":"skills/deobfuscating-javascript-malware"},{"name":"deobfuscating-powershell-obfuscated-malware","description":"Systematically deobfuscates multi-layer PowerShell malware using AST analysis, dynamic tracing, and tools like PSDecode and PowerDecode to reveal hidden payloads and C2 infrastructure. Use during incident response or malware analysis when a PowerShell script is obfuscated with encoding, string manipulation, or invocation tricks and you need to recover the underlying commands, dropped payloads, or C2 endpoints.","domain":"cybersecurity","path":"skills/deobfuscating-powershell-obfuscated-malware"},{"name":"deploying-active-directory-honeytokens","description":"Deploys deception-based honeytokens in Active Directory including fake privileged accounts with AdminCount=1, fake SPNs for Kerberoasting detection (honeyroasting), decoy GPOs with cpassword traps, and fake BloodHound paths. Monitors Windows Security Event IDs 4769, 4625, 4662, 5136 for honeytoken interaction. Use when implementing AD deception defenses for detecting lateral movement, credential theft, and reconnaissance.","domain":"cybersecurity","path":"skills/deploying-active-directory-honeytokens"},{"name":"deploying-cloud-deception-with-decoy-resources","description":"Deploy cloud-native deception across AWS, Azure, and GCP using decoy (honey) resources whose only purpose is to generate a high-fidelity alert the instant an attacker touches them: canary IAM access keys, permission-less decoy users/roles/service principals, honey object-storage buckets, and decoy secrets in Secrets Manager / Key Vault / Secret Manager. Wires detection through CloudTrail + EventBridge, Azure Sentinel honeytoken watchlists + Defender, and GCP Cloud Audit Logs, so any use of a decoy is routed to the SOC with near-zero false positives. Use when protecting cloud accounts and data stores, when an org has only on-prem honeypots and needs cloud coverage, when seeding fake AWS keys to catch credential theft and code-leak exposure, or when detecting cloud reconnaissance and lateral movement. Keywords: cloud deception, canary token AWS, honey S3 bucket, decoy IAM credentials, CloudTrail alert, GuardDuty, Sentinel honeytoken, decoy secret, honey service account, cloud honeypot, breach detection.","domain":"cybersecurity","path":"skills/deploying-cloud-deception-with-decoy-resources"},{"name":"deploying-cloudflare-access-for-zero-trust","description":"Deploys Cloudflare Access with Cloudflare Tunnel for zero trust access to self-hosted apps, configuring identity-aware policies, device posture checks, and WARP client enrollment as a VPN replacement. Use when replacing VPN with Cloudflare One, exposing internal apps without open inbound ports, or securing contractor/third-party access to specific applications.","domain":"cybersecurity","path":"skills/deploying-cloudflare-access-for-zero-trust"},{"name":"deploying-decoy-files-for-ransomware-detection","description":"Deploys canary files (honeytokens) across file systems to detect ransomware encryption activity in real time. Uses strategically placed decoy documents monitored via file integrity monitoring or OS-level watchdogs to trigger alerts when ransomware modifies or encrypts them. Activates for requests involving ransomware canary deployment, honeyfile setup, deception-based ransomware detection, or file integrity monitoring for encryption.","domain":"cybersecurity","path":"skills/deploying-decoy-files-for-ransomware-detection"},{"name":"deploying-edr-agent-with-crowdstrike","description":"Deploys and configures CrowdStrike Falcon EDR agents across enterprise endpoints to enable real-time threat detection, behavioral analysis, and automated response. Use when onboarding endpoints to EDR coverage, configuring detection policies, or integrating Falcon telemetry with SIEM platforms. Activates for requests involving CrowdStrike deployment, Falcon sensor installation, EDR policy configuration, or endpoint detection and response.","domain":"cybersecurity","path":"skills/deploying-edr-agent-with-crowdstrike"},{"name":"deploying-honeytokens-and-canarytokens","description":"Plants Canarytokens-based decoy artifacts (honey credentials, DNS tokens, web-bug URLs, AWS keys, documents, kubeconfigs) using Thinkst's open-source Canarytokens project and alerts via email or webhook when a token is touched. Use for high-fidelity intrusion detection in low-telemetry areas like file shares or credential stores, or to catch credential dumping and data-theft staging.","domain":"cybersecurity","path":"skills/deploying-honeytokens-and-canarytokens"},{"name":"deploying-osquery-for-endpoint-monitoring","description":"Deploys and configures osquery for real-time endpoint monitoring using SQL-based queries to inspect running processes, open ports, installed software, and system configuration. Use when building visibility into endpoint state, threat hunting across fleet, or implementing compliance monitoring. Activates for requests involving osquery deployment, endpoint visibility, fleet management, or SQL-based endpoint querying.","domain":"cybersecurity","path":"skills/deploying-osquery-for-endpoint-monitoring"},{"name":"deploying-palo-alto-prisma-access-zero-trust","description":"Deploys Palo Alto Networks Prisma Access for SASE-based zero trust network access, configuring GlobalProtect agents, ZTNA Connectors, security policy enforcement, and Strata Cloud Manager integration for unified management. Use when implementing enterprise-grade SASE with integrated ZTNA/SWG/CASB/FWaaS, replacing both VPN and branch firewalls with cloud-delivered security, or integrating ZTNA with an existing Palo Alto NGFW estate.","domain":"cybersecurity","path":"skills/deploying-palo-alto-prisma-access-zero-trust"},{"name":"deploying-ransomware-canary-files","description":"Deploys and monitors ransomware canary files using Python's watchdog library, placing decoy files mimicking high-value targets (financial records, credentials, database exports) where ransomware enumerates first, and alerting via email, Slack, or syslog on any read/modify/rename/delete. Use for early-warning ransomware detection on file servers, NAS, or endpoints, or to supplement EDR where agents can't be deployed.","domain":"cybersecurity","path":"skills/deploying-ransomware-canary-files"},{"name":"deploying-software-defined-perimeter","description":"Deploys a Software-Defined Perimeter per the CSA v2.0 specification, configuring Single Packet Authorization, mutual TLS, and SDP controller/gateway components to enforce zero trust network access. Use when building or hardening zero trust network architecture, implementing SPA-based \"invisible\" infrastructure that cloaks services from unauthenticated scanning, or meeting compliance requirements for zero trust network access.","domain":"cybersecurity","path":"skills/deploying-software-defined-perimeter"},{"name":"deploying-tailscale-for-zero-trust-vpn","description":"Deploys and configures Tailscale (or self-hosted Headscale) as a WireGuard-based zero trust mesh VPN, setting up identity-aware ACLs, exit nodes, subnet routers, and MagicDNS for encrypted peer-to-peer connectivity. Use when replacing traditional VPN servers with an identity-authenticated mesh network or enforcing granular per-device access control lists.","domain":"cybersecurity","path":"skills/deploying-tailscale-for-zero-trust-vpn"},{"name":"designing-adversary-engagement-with-mitre-engage","description":"Plan, run, and measure an adversary engagement operation using the MITRE Engage framework so that deployed deception is driven by strategy instead of deployed ad hoc. Covers the Engage Matrix (Prepare, Expose, Affect, Elicit, Understand), the 10-Step Operational Process, mapping engagement Activities to the ATT&CK techniques they expose, and defining measurable Goals and Operational Objectives. Use when a team has honeypots, honeytokens, or canary tokens but no coordinating strategy, when leadership asks \"should we engage attackers and how\", when building a deception/denial program, when writing an adversary engagement operation plan, or when deciding which deception Activities to deploy against a specific threat actor. Keywords: MITRE Engage, adversary engagement, cyber deception strategy, denial and deception, Engage Matrix, EAC, EGO, Expose Affect Elicit, deception program, honeypot strategy, engagement operation.","domain":"cybersecurity","path":"skills/designing-adversary-engagement-with-mitre-engage"},{"name":"detecting-ai-model-prompt-injection-attacks","description":"Detects prompt injection using regex signature matching, heuristic scoring for structural anomalies, and DeBERTa-based transformer classification, flagging direct injections (system-prompt overrides, role-play escapes) and indirect injections (encoded payloads, obfuscation) per OWASP LLM Top 10 (LLM01:2025). Use for input validation layers in chatbots/agents/RAG pipelines, or for retrospectively classifying injection attempts in logs or incident investigations.","domain":"cybersecurity","path":"skills/detecting-ai-model-prompt-injection-attacks"},{"name":"detecting-anomalies-in-industrial-control-systems","description":"Deploys anomaly detection for OT/ICS environments using machine learning on OT network baselines, physics-based process models, and Modbus/DNP3/OPC UA traffic analysis to flag deviations, rogue devices, and mismatches against historian data. Use for continuous OT monitoring, baselining deterministic SCADA polling, or investigating alerts from Nozomi Guardian/Dragos needing deeper protocol analysis.","domain":"cybersecurity","path":"skills/detecting-anomalies-in-industrial-control-systems"},{"name":"detecting-anomalous-authentication-patterns","description":"Detects anomalous authentication patterns using UEBA analytics, statistical baselines, and machine learning models to identify impossible travel, credential stuffing, brute force, password spraying, and compromised account behaviors across authentication logs. Activates for requests involving authentication anomaly detection, login behavior analysis, UEBA implementation, or suspicious sign-in investigation.","domain":"cybersecurity","path":"skills/detecting-anomalous-authentication-patterns"},{"name":"detecting-api-enumeration-attacks","description":"Detect API enumeration attacks (BOLA/IDOR, OWASP API1:2023) by writing SIEM detection rules that flag sequential or UUID identifier iteration, parameter tampering, and mixed 200/401/403 response patterns from API gateway and WAF logs. Use when investigating suspected object-level authorization abuse, building threat-hunting queries for API access-control bypass, or hardening API logging/rate-limiting against enumeration.","domain":"cybersecurity","path":"skills/detecting-api-enumeration-attacks"},{"name":"detecting-arp-poisoning-in-network-traffic","description":"Detect Layer 2 ARP poisoning/spoofing by deploying ARPWatch, Dynamic ARP Inspection (DAI), Wireshark packet analysis, and custom Python monitoring scripts that flag gratuitous ARP floods, IP-to-MAC mapping changes, and duplicate IP addresses. Use when investigating suspected man-in-the-middle interception or session hijacking on a local network segment, or when building layer-2 anomaly detection for a SOC.","domain":"cybersecurity","path":"skills/detecting-arp-poisoning-in-network-traffic"},{"name":"detecting-attacks-on-historian-servers","description":"Detect cyber attacks on OT historian servers (OSIsoft PI, Ignition, GE Proficy, Wonderware InSQL) using a Python detector that flags unauthorized queries, data manipulation, and lateral-movement indicators as historians pivot between IT and OT networks. Use when monitoring historians bridging IT/OT zones for compromise, investigating historian-specific CVE exploitation, or validating historian data integrity after a suspected OT incident.","domain":"cybersecurity","path":"skills/detecting-attacks-on-historian-servers"},{"name":"detecting-attacks-on-scada-systems","description":"This skill covers detecting cyber attacks targeting Supervisory Control and Data Acquisition (SCADA) systems including man-in-the-middle attacks on industrial protocols, unauthorized command injection into PLCs, HMI compromise, historian data manipulation, and denial-of-service against control system communications. It leverages OT-specific intrusion detection systems, industrial protocol anomaly detection, and process data analytics to identify attacks that traditional IT security tools miss.","domain":"cybersecurity","path":"skills/detecting-attacks-on-scada-systems"},{"name":"detecting-aws-cloudtrail-anomalies","description":"Detect unusual API call patterns in AWS CloudTrail logs using boto3, statistical baselining, and behavioral analysis to identify credential compromise, privilege escalation, and unauthorized resource access.","domain":"cybersecurity","path":"skills/detecting-aws-cloudtrail-anomalies"},{"name":"detecting-aws-credential-exposure-with-trufflehog","description":"Scan source code repositories, CI/CD pipelines, and configuration files for exposed AWS credentials using TruffleHog, git-secrets, and AWS-native detection. Use when integrating secrets scanning into CI/CD, auditing repositories (including git history) for historically committed AWS keys, responding to a GuardDuty alert about credential use from an unexpected location, or verifying credential rotation removed all exposed keys.","domain":"cybersecurity","path":"skills/detecting-aws-credential-exposure-with-trufflehog"},{"name":"detecting-aws-guardduty-findings-automation","description":"Build automated AWS GuardDuty finding response pipelines using EventBridge and Lambda to trigger real-time incident response, automatically quarantine compromised resources, and route security notifications. Use when designing automated remediation playbooks for GuardDuty findings across VPC Flow Logs, CloudTrail, DNS, EKS, or S3 data events, or when reducing mean time to respond to cloud threats.","domain":"cybersecurity","path":"skills/detecting-aws-guardduty-findings-automation"},{"name":"detecting-aws-iam-privilege-escalation","description":"Detect AWS IAM privilege escalation paths using boto3 and Cloudsplaining policy analysis to identify overly permissive policies, dangerous permission combinations, and least-privilege violations","domain":"cybersecurity","path":"skills/detecting-aws-iam-privilege-escalation"},{"name":"detecting-azure-lateral-movement","description":"Detect lateral movement in Azure AD/Entra ID environments using Microsoft Graph API audit logs, Azure Sentinel KQL hunting queries, and sign-in anomaly correlation to identify privilege escalation, token theft, and cross-tenant pivoting.","domain":"cybersecurity","path":"skills/detecting-azure-lateral-movement"},{"name":"detecting-azure-service-principal-abuse","description":"Detect Azure service principal abuse in Microsoft Entra ID using KQL detection queries (Sentinel/Splunk) against Azure AD Audit and Sign-in Logs, covering added credentials, privileged role assignment, admin consent bypass, and service principal enumeration. Use when investigating suspected privilege escalation or persistence via service principals, or building threat-hunting queries for Entra ID identity abuse.","domain":"cybersecurity","path":"skills/detecting-azure-service-principal-abuse"},{"name":"detecting-azure-storage-account-misconfigurations","description":"Audit Azure Blob and ADLS storage accounts for public access exposure, weak or long-lived SAS tokens, missing encryption at rest, disabled HTTPS-only traffic, and outdated TLS versions, using the azure-mgmt-storage Python SDK to generate a risk-scored report. Use when assessing an Azure subscription's storage accounts for misconfiguration, building cloud security posture checks, or investigating a suspected data exposure via public blob access.","domain":"cybersecurity","path":"skills/detecting-azure-storage-account-misconfigurations"},{"name":"detecting-beaconing-patterns-with-zeek","description":"Performs statistical analysis of Zeek conn.log connection intervals to detect C2 beaconing patterns. Uses the ZAT library to load Zeek logs into Pandas DataFrames, calculates inter-arrival time standard deviation, and flags periodic connections with low jitter. Use when hunting for command-and-control callbacks in network data.","domain":"cybersecurity","path":"skills/detecting-beaconing-patterns-with-zeek"},{"name":"detecting-bluetooth-low-energy-attacks","description":"Detects and analyzes Bluetooth Low Energy (BLE) security attacks including sniffing, replay attacks, GATT enumeration abuse, and Man-in-the-Middle interception. Uses Ubertooth One and nRF52840 sniffers for packet capture, the bleak Python library for GATT service enumeration, and crackle for BLE encryption cracking. Use when assessing IoT device BLE security, monitoring for BLE-based attacks on wireless infrastructure, or performing authorized BLE penetration testing. Activates for requests involving BLE security assessment, Ubertooth sniffing, GATT enumeration, or BLE replay detection.","domain":"cybersecurity","path":"skills/detecting-bluetooth-low-energy-attacks"},{"name":"detecting-broken-object-property-level-authorization","description":"Detect and test for OWASP API3:2023 Broken Object Property Level Authorization (BOPLA), covering excessive data exposure in API responses and mass assignment via injected request-body properties. Use when reviewing API responses/requests for over-exposed or over-writable object fields, or building detection rules and test cases for property-level authorization gaps that object-level checks miss.","domain":"cybersecurity","path":"skills/detecting-broken-object-property-level-authorization"},{"name":"detecting-business-email-compromise","description":"Detect Business Email Compromise (BEC) fraud, where attackers impersonate executives or vendors to trick employees into wire transfers or data disclosure without malicious links, using email gateway rules, behavioral analytics, and financial process controls. Use when investigating a suspected executive/vendor impersonation email or building detection rules for social-engineering fund-transfer fraud.","domain":"cybersecurity","path":"skills/detecting-business-email-compromise"},{"name":"detecting-business-email-compromise-with-ai","description":"Deploy AI and NLP-powered detection systems to identify business email compromise attacks by analyzing writing style, behavioral patterns, and contextual anomalies that evade traditional rule-based filters.","domain":"cybersecurity","path":"skills/detecting-business-email-compromise-with-ai"},{"name":"detecting-cloud-threats-with-guardduty","description":"Deploy and operationalize Amazon GuardDuty, covering protection plans for S3, EKS, EC2 runtime monitoring, and Lambda, interpreting finding severity, and building automated response with EventBridge and Lambda. Use when establishing threat detection for AWS accounts, investigating findings on compromised instances or credential abuse, or building automated incident-response playbooks.","domain":"cybersecurity","path":"skills/detecting-cloud-threats-with-guardduty"},{"name":"detecting-command-and-control-over-dns","description":"Detect command-and-control (C2) traffic tunneled over DNS from tools like Iodine, dnscat2, dns2tcp, and Cobalt Strike DNS beacon, using Shannon entropy analysis of query subdomains, ML-based DGA classification, passive DNS correlation, and Zeek/Suricata signatures. Use when investigating suspected DNS tunneling, classifying DGA domains, detecting DNS beaconing, or building DNS anomaly rules for a SOC/SIEM.","domain":"cybersecurity","path":"skills/detecting-command-and-control-over-dns"},{"name":"detecting-compromised-cloud-credentials","description":"Detect compromised cloud credentials across AWS, Azure, and GCP by analyzing anomalous API activity, impossible-travel patterns, and credential-stuffing indicators using GuardDuty, Microsoft Defender for Identity, and Google SCC Event Threat Detection. Use when investigating alerts about cloud API activity from unfamiliar locations, responding to an exposed-credential notification, or scoping a credential compromise.","domain":"cybersecurity","path":"skills/detecting-compromised-cloud-credentials"},{"name":"detecting-container-drift-at-runtime","description":"Detects unauthorized runtime drift in containers by monitoring binary execution, filesystem changes, and configuration deviation from the original immutable image, using Falco and Microsoft Defender for Containers. Use when validating immutable-infrastructure controls, hunting for unexpected package installs or binaries written inside a running container, or determining whether a container diverged from the image it was built from. Keywords: drift, immutable infrastructure, new binary executed, package install, image mismatch, Falco. Do not use for detecting breakout from the container to the host - use detecting-container-escape-attempts.","domain":"cybersecurity","path":"skills/detecting-container-drift-at-runtime"},{"name":"detecting-container-escape-attempts","description":"Detects container escape at runtime across tooling - namespace manipulation, capability abuse, kernel exploits, sensitive host mounts, and anomalous syscalls - and explains which signals matter regardless of whether Falco, Sysdig, auditd, or an EDR is doing the collection. Use when deciding what breakout behaviour to monitor, investigating a suspected Docker or Kubernetes breakout, or comparing escape coverage across runtime sensors. Keywords: container escape, breakout, namespaces, CAP_SYS_ADMIN, privileged, hostPath, kernel exploit, syscall. Do not use for Falco rule syntax itself - use detecting-container-escape-with-falco-rules; for a static configuration sweep use performing-container-escape-detection.","domain":"cybersecurity","path":"skills/detecting-container-escape-attempts"},{"name":"detecting-container-escape-with-falco-rules","description":"Writes and tunes Falco rule syntax for container escape detection - conditions, macros, lists, priorities, and output fields - covering host filesystem mounts, sensitive host path access, kernel module loading, and privileged capability abuse, including how to drive down false positives. Use when authoring or tuning a specific Falco rule for breakout behaviour, or triaging a noisy escape-related Falco alert. Keywords: Falco rule, macro, list, condition, priority, falco_rules.local.yaml, tuning, false positive. Do not use for deploying and operating Falco itself - use detecting-container-runtime-threats-with-falco; for tool-agnostic escape signals use detecting-container-escape-attempts.","domain":"cybersecurity","path":"skills/detecting-container-escape-with-falco-rules"},{"name":"detecting-container-runtime-threats-with-falco","description":"Deploys and operates Falco with the modern eBPF driver in Kubernetes and Docker, covering driver selection, Helm installation, output channels, and the built-in ruleset that detects container escape, namespace abuse, privileged mounts, and anomalous syscalls. Use when standing Falco up on a cluster, choosing between the eBPF and kernel-module drivers, routing Falco alerts into a SIEM or Falcosidekick, or upgrading an existing deployment. Keywords: Falco, modern_ebpf, kernel module, Helm, Falcosidekick, runtime security, syscall. Do not use for authoring individual escape rules - use detecting-container-escape-with-falco-rules.","domain":"cybersecurity","path":"skills/detecting-container-runtime-threats-with-falco"},{"name":"detecting-credential-dumping-techniques","description":"Detect LSASS credential dumping, SAM database extraction, and NTDS.dit theft (e.g. via Mimikatz) using Sysmon Event ID 10 process-access logging, Windows Security logs, and SIEM correlation rules. Use when hunting for credential-theft activity on Windows/Active Directory hosts or triaging EDR alerts on LSASS access.","domain":"cybersecurity","path":"skills/detecting-credential-dumping-techniques"},{"name":"detecting-cryptomining-in-cloud","description":"This skill teaches security teams how to detect and respond to unauthorized cryptocurrency mining operations in cloud environments. It covers identifying cryptomining indicators through compute usage anomalies, network traffic patterns to mining pools, GuardDuty CryptoCurrency findings, and runtime process monitoring on EC2, ECS, EKS, and Azure Automation workloads.","domain":"cybersecurity","path":"skills/detecting-cryptomining-in-cloud"},{"name":"detecting-data-and-model-poisoning","description":"Identify poisoned training data and backdoored ML models across the pipeline using IBM's Adversarial Robustness Toolbox (activation clustering, spectral signatures, trigger reconstruction), Cleanlab for label-quality issues, and supply-chain checks like weight-hash verification and safetensors enforcement. Use before training or deploying on third-party/user-contributed data or downloaded checkpoints, during ML supply-chain reviews, or when investigating model misbehavior tied to specific inputs (suspected backdoor trigger).","domain":"cybersecurity","path":"skills/detecting-data-and-model-poisoning"},{"name":"detecting-dcsync-attack-in-active-directory","description":"Detect DCSync attacks (MITRE T1003.006) where adversaries abuse Active Directory replication privileges to extract password hashes, by auditing Event ID 4662 for the DS-Replication-Get-Changes GUIDs and flagging non-domain-controller accounts issuing DsGetNCChanges RPC calls. Use when hunting for credential theft via Mimikatz lsadump::dcsync or Impacket secretsdump, investigating lateral movement with domain admin credentials, or auditing AD replication permissions.","domain":"cybersecurity","path":"skills/detecting-dcsync-attack-in-active-directory"},{"name":"detecting-deepfake-audio-in-vishing-attacks","description":"Detect AI-generated deepfake audio used in voice phishing (vishing) by extracting spectral features (MFCC, spectral centroid, spectral contrast, zero-crossing rate) and classifying samples with machine learning models, supporting batch audio analysis, confidence scoring, and forensic reporting. Use for deepfake voice detection, vishing investigations, AI-generated speech analysis, voice cloning detection, or audio authenticity verification.","domain":"cybersecurity","path":"skills/detecting-deepfake-audio-in-vishing-attacks"},{"name":"detecting-dependency-confusion","description":"Detect and prevent dependency confusion (public-over-private package name resolution) in npm, PyPI, and Maven by enumerating claimable internal package names with tools like `confused` and OWASP `dep-scan`, then enforcing source restrictions via `.npmrc`, `pip.conf`/`pyproject.toml`, and Maven `settings.xml`. Use when onboarding a repo to a supply-chain security program, auditing lockfiles/manifests for confusable dependencies, or after an incident that may have leaked internal package names.","domain":"cybersecurity","path":"skills/detecting-dependency-confusion"},{"name":"detecting-dll-sideloading-attacks","description":"Detect DLL side-loading and search-order hijacking (MITRE T1574) where adversaries plant malicious DLLs for legitimate signed applications to load, by analyzing Sysmon Event ID 7 DLL-load events, checking signatures/hashes against known-good versions, and flagging path anomalies with EDR tools like CrowdStrike, MDE, or SentinelOne. Use when investigating EDR alerts on unsigned DLLs, hunting for APT persistence via trojanized applications, or triaging incidents involving DLL hijacking.","domain":"cybersecurity","path":"skills/detecting-dll-sideloading-attacks"},{"name":"detecting-dnp3-protocol-anomalies","description":"Detect anomalies in DNP3 communications used in SCADA/ICS systems by monitoring unauthorized control commands, firmware update attempts, protocol violations, and deviations from baseline traffic using deep packet inspection and machine learning approaches. Use when securing energy-sector or other OT/ICS networks, investigating suspicious DNP3 master/outstation activity, or building an anomaly-based IDS for industrial control traffic.","domain":"cybersecurity","path":"skills/detecting-dnp3-protocol-anomalies"},{"name":"detecting-dns-exfiltration-with-dns-query-analysis","description":"Detect data exfiltration via DNS tunneling (tools like iodine, dnscat2, dns2tcp) by analyzing query entropy, subdomain length, query volume to single domains, TXT/CNAME/NULL record abuse, and oversized response payloads using passive DNS monitoring and statistical/ML methods. Use when hunting for covert DNS-based data exfiltration or building a passive DNS anomaly detection capability.","domain":"cybersecurity","path":"skills/detecting-dns-exfiltration-with-dns-query-analysis"},{"name":"detecting-email-account-compromise","description":"Detect compromised O365 and Google Workspace email accounts by analyzing Unified Audit Logs and Azure AD sign-in logs for impossible travel, inbox rule creation/deletion (Set-InboxRule, New-InboxRule), external mail forwarding rules, and unusual Microsoft Graph API access or OAuth token use. Use when investigating suspected business email compromise (BEC), account takeover, or mailbox persistence via malicious inbox rules.","domain":"cybersecurity","path":"skills/detecting-email-account-compromise"},{"name":"detecting-email-forwarding-rules-attack","description":"Detect malicious inbox/mail-flow forwarding rules that adversaries create to maintain persistent access to email communications for intelligence collection and business email compromise. Use when hunting for suspicious auto-forwarding, hidden mailbox rules, or T1114-style email collection persistence after a compromised account or BEC incident.","domain":"cybersecurity","path":"skills/detecting-email-forwarding-rules-attack"},{"name":"detecting-entra-offensive-tools-in-graph-logs","description":"Hunt AADGraphActivityLogs and MicrosoftGraphActivityLogs in Microsoft Sentinel/Log Analytics using KQL to fingerprint offensive Entra ID enumeration tools such as ROADtools, AADInternals, and AzureHound, including User-Agent signatures, roadrecon endpoint sweeps, and sign-in correlation. Use when investigating suspicious Microsoft Graph API activity, Entra ID reconnaissance, or building Sentinel analytics rules to detect these tools.","domain":"cybersecurity","path":"skills/detecting-entra-offensive-tools-in-graph-logs"},{"name":"detecting-evasion-techniques-in-endpoint-logs","description":"Detects defense evasion techniques used by adversaries in endpoint logs including log tampering, timestomping, process injection, and security tool disabling. Use when investigating suspicious endpoint behavior, building detection rules for evasion tactics, or conducting threat hunting for stealthy adversary activity. Activates for requests involving evasion detection, defense evasion analysis, log tampering detection, or MITRE ATT&CK TA0005.","domain":"cybersecurity","path":"skills/detecting-evasion-techniques-in-endpoint-logs"},{"name":"detecting-exfiltration-over-dns-with-zeek","description":"Detect DNS-based data exfiltration by analyzing Zeek dns.log for high-entropy subdomains, oversized TXT/NULL records, and anomalous query volume or patterns. Use when investigating suspected DNS tunneling, covert C2 over DNS, or data exfiltration hidden in DNS queries against network traffic captured by Zeek.","domain":"cybersecurity","path":"skills/detecting-exfiltration-over-dns-with-zeek"},{"name":"detecting-fileless-attacks-on-endpoints","description":"Detects fileless malware and in-memory attacks that execute entirely in RAM without writing persistent files to disk, evading traditional antivirus. Use when building detections for PowerShell-based attacks, reflective DLL injection, WMI persistence, and registry-resident malware. Activates for requests involving fileless malware detection, in-memory attacks, PowerShell exploitation, or living-off-the-land techniques.","domain":"cybersecurity","path":"skills/detecting-fileless-attacks-on-endpoints"},{"name":"detecting-fileless-malware-techniques","description":"Detects and analyzes fileless malware that operates entirely in memory using PowerShell, WMI, .NET reflection, registry-resident payloads, and living-off-the-land binaries (LOLBins) without writing traditional executable files to disk. Use for requests involving fileless threat detection, in-memory malware investigation, LOLBin abuse analysis, or WMI persistence examination.","domain":"cybersecurity","path":"skills/detecting-fileless-malware-techniques"},{"name":"detecting-golden-ticket-attacks-in-kerberos-logs","description":"Detect Golden Ticket attacks in Active Directory using Splunk and KQL queries against domain controller event logs, looking for Kerberos TGT anomalies such as mismatched encryption types, impossible ticket lifetimes, non-existent accounts, and forged PAC signatures. Use when hunting for Kerberos ticket forgery or krbtgt-based persistence (MITRE T1558.001) in AD environments.","domain":"cybersecurity","path":"skills/detecting-golden-ticket-attacks-in-kerberos-logs"},{"name":"detecting-golden-ticket-forgery","description":"Detect Kerberos Golden Ticket forgery (e.g. Mimikatz-forged tickets) by analyzing Windows Event ID 4769 for RC4 encryption downgrades (0x17), abnormal ticket lifetimes, and krbtgt account anomalies using Splunk and Elastic SIEM queries. Use when investigating suspected forged Kerberos tickets or krbtgt credential theft in Active Directory.","domain":"cybersecurity","path":"skills/detecting-golden-ticket-forgery"},{"name":"detecting-indirect-prompt-injection","description":"Detect and defend against indirect prompt injection hidden in web pages, documents, and images consumed by an agent, via content extraction (HTML/PDF/OCR), normalization, and scanning with LLM Guard's PromptInjection scanner or Hugging Face Prompt Guard 2. Use when an agent ingests untrusted external content and you need to screen it for injected instructions before the LLM processes it.","domain":"cybersecurity","path":"skills/detecting-indirect-prompt-injection"},{"name":"detecting-insider-data-exfiltration-via-dlp","description":"Detects insider data exfiltration by analyzing DLP policy violations, file access patterns, upload volume anomalies, and off-hours activity in endpoint and cloud logs. Uses pandas for behavioral analytics and statistical baselines. Use when investigating insider threats or building user behavior analytics for data loss prevention.","domain":"cybersecurity","path":"skills/detecting-insider-data-exfiltration-via-dlp"},{"name":"detecting-insider-threat-behaviors","description":"Detect insider threat behavioral indicators including unusual data access, off-hours activity, mass file downloads, privilege abuse, and resignation-correlated data theft. Use when proactively threat-hunting for malicious or negligent insider activity, or when investigating a departing or disgruntled employee for potential data theft.","domain":"cybersecurity","path":"skills/detecting-insider-threat-behaviors"},{"name":"detecting-insider-threat-with-ueba","description":"Implement User and Entity Behavior Analytics (UEBA) using Elasticsearch/OpenSearch to build behavioral baselines, calculate anomaly scores, perform peer group analysis, and alert on insider threat indicators such as data exfiltration, privilege abuse, and unauthorized access. Use when building or tuning a UEBA pipeline rather than a one-off manual hunt.","domain":"cybersecurity","path":"skills/detecting-insider-threat-with-ueba"},{"name":"detecting-kerberoasting-attacks","description":"Detect Kerberoasting attacks by monitoring for anomalous Kerberos TGS requests (Event ID 4769) targeting service accounts with SPNs, which attackers request offline to crack service account passwords. Use when hunting for MITRE T1558 credential access activity or investigating suspected service account password cracking attempts in Active Directory Kerberos logs.","domain":"cybersecurity","path":"skills/detecting-kerberoasting-attacks"},{"name":"detecting-lateral-movement-in-network","description":"Identifies lateral movement techniques in enterprise networks by analyzing authentication logs, network flows, SMB traffic, and RDP sessions using Zeek, Velociraptor, and SIEM correlation rules to detect attackers moving between systems.","domain":"cybersecurity","path":"skills/detecting-lateral-movement-in-network"},{"name":"detecting-lateral-movement-with-splunk","description":"Detect adversary lateral movement across networks using Splunk SPL queries against Windows authentication logs, SMB traffic, and remote service (WMI/PsExec/RDP) abuse. Use when hunting for MITRE ATT&CK TA0008 lateral movement activity or investigating suspected pivoting between hosts during an incident, with Splunk as the SIEM.","domain":"cybersecurity","path":"skills/detecting-lateral-movement-with-splunk"},{"name":"detecting-lateral-movement-with-zeek","description":"Detect lateral movement in network traffic using Zeek (formerly Bro) log analysis. Parses conn.log, smb_mapping.log, smb_files.log, dce_rpc.log, kerberos.log, and ntlm.log to identify SMB file transfers, NTLM account spray activity, remote service execution, and anomalous internal connections.","domain":"cybersecurity","path":"skills/detecting-lateral-movement-with-zeek"},{"name":"detecting-living-off-the-land-attacks","description":"Detect abuse of legitimate Windows binaries (LOLBins) used for living off the land attacks. Monitors process creation, command-line arguments, and parent-child relationships to identify suspicious LOLBin execution patterns.","domain":"cybersecurity","path":"skills/detecting-living-off-the-land-attacks"},{"name":"detecting-living-off-the-land-with-lolbas","description":"Detect Living Off the Land Binaries (LOLBins/LOLBAS) abuse including certutil, regsvr32, mshta, and rundll32 via process telemetry, Sigma rules, and parent-child process analysis with Sysmon endpoint data. Use when hunting for adversaries abusing built-in Windows binaries to download, execute, or proxy malicious code while evading traditional executable-based detection.","domain":"cybersecurity","path":"skills/detecting-living-off-the-land-with-lolbas"},{"name":"detecting-malicious-npm-packages","description":"Triage npm packages and lockfiles for install-script malware, credential exfiltration, and worming behavior using GuardDog, manual tarball inspection, and dynamic detonation with network/filesystem monitoring. Use when vetting a new dependency, reviewing a package.json/package-lock.json during code review, checking lockfiles against a supply-chain advisory's known-bad versions, or investigating a host suspected of installing a trojanized package.","domain":"cybersecurity","path":"skills/detecting-malicious-npm-packages"},{"name":"detecting-malicious-scheduled-tasks-with-sysmon","description":"Detect malicious scheduled task creation and modification using Sysmon Event IDs 1 (Process Create for schtasks.exe), 11 (File Create for task XML), and Windows Security Event 4698/4702. The analyst correlates task creation with suspicious parent processes, public directory paths, and encoded command arguments to identify persistence and lateral movement via scheduled tasks. Activates for requests involving scheduled task detection, Sysmon persistence hunting, or T1053.005 Scheduled Task/Job analysis.","domain":"cybersecurity","path":"skills/detecting-malicious-scheduled-tasks-with-sysmon"},{"name":"detecting-mimikatz-execution-patterns","description":"Detect Mimikatz credential-dumping activity via command-line pattern matching, LSASS access signatures, binary/hash indicators, and in-memory detection of known Mimikatz modules. Use when threat hunting for T1003 credential access, triaging EDR/SIEM alerts on LSASS access, scoping compromise during incident response, or validating detection coverage in a purple team exercise.","domain":"cybersecurity","path":"skills/detecting-mimikatz-execution-patterns"},{"name":"detecting-misconfigured-azure-storage","description":"Audit Azure Storage accounts for public blob containers, missing encryption, overly permissive SAS tokens, disabled logging, and network access violations using Azure CLI, PowerShell, and Microsoft Defender for Storage. Use for storage security audits across subscriptions, responding to Defender for Storage anonymous-access alerts, verifying compliance controls, or setting security baselines when onboarding a subscription.","domain":"cybersecurity","path":"skills/detecting-misconfigured-azure-storage"},{"name":"detecting-mobile-malware-behavior","description":"Detects and analyzes malicious behavior in mobile applications through behavioral analysis, permission abuse detection, network traffic monitoring, and dynamic instrumentation. Use when analyzing suspicious mobile applications for data exfiltration, command-and-control communication, credential stealing, SMS interception, or other malware indicators. Activates for requests involving mobile malware analysis, app behavior monitoring, trojan detection, or suspicious app investigation.","domain":"cybersecurity","path":"skills/detecting-mobile-malware-behavior"},{"name":"detecting-modbus-command-injection-attacks","description":"Detect command injection against Modbus TCP/RTU in ICS/SCADA environments by monitoring unauthorized writes, anomalous function codes, malformed frames, and deviations from communication baselines using ICS-aware IDS and deep packet inspection. Use when deploying IDS for Modbus OT networks, investigating unauthorized PLC register/coil changes, or responding to FrostyGoop-style Modbus attacks.","domain":"cybersecurity","path":"skills/detecting-modbus-command-injection-attacks"},{"name":"detecting-modbus-protocol-anomalies","description":"Detect anomalies in Modbus/TCP and Modbus RTU industrial traffic via function code monitoring, register range validation, timing analysis, and deep packet inspection, using Zeek's Modbus analyzer, Suricata IDS with OT rules, and Python Markov chain models of normal transaction sequences. Use for deploying Modbus IDS in OT environments, baselining polling patterns, investigating suspicious Modbus traffic, or building function code allowlists.","domain":"cybersecurity","path":"skills/detecting-modbus-protocol-anomalies"},{"name":"detecting-model-extraction-attacks","description":"Detect MITRE ATLAS AML.T0024 attacks (model stealing, inversion, membership inference) performed via inference-API abuse, by monitoring per-principal query volume/distribution, rate-limiting and perturbing outputs, and red-teaming your model's extractability. Use for a public or partner inference API needing cloning/inversion/membership-inference detection, or a pre-deployment red-team exercise to measure extraction risk.","domain":"cybersecurity","path":"skills/detecting-model-extraction-attacks"},{"name":"detecting-network-anomalies-with-zeek","description":"Deploy and configure Zeek (formerly Bro) to passively analyze network traffic, generate structured connection/DNS/HTTP/SSL/file logs, detect anomalous behavior, and write custom scripts for organization-specific threats. Use for passive monitoring at network choke points, feeding SIEM/threat hunting with protocol metadata, or retrospective log analysis during incident response; not a substitute for inline IDS/IPS or host agents.","domain":"cybersecurity","path":"skills/detecting-network-anomalies-with-zeek"},{"name":"detecting-network-scanning-with-ids-signatures","description":"Detect network reconnaissance and port scanning using Suricata and Snort IDS signatures, threshold-based detection rules, and traffic anomaly analysis to identify Nmap, Masscan, and custom scanning activity.","domain":"cybersecurity","path":"skills/detecting-network-scanning-with-ids-signatures"},{"name":"detecting-ntlm-relay-with-event-correlation","description":"Detect NTLM relay attacks (T1557.001) by correlating Windows Event 4624 LogonType 3 for IP-to-hostname mismatches, identifying Responder/LLMNR poisoning artifacts, auditing SMB/LDAP signing, and flagging NTLMv2-to-NTLMv1 downgrades. Use for hunting credential relay in NTLM-enabled AD, investigating auth-source anomalies, building SIEM correlation rules, or responding to PetitPotam/DFSCoerce/PrinterBug alerts.","domain":"cybersecurity","path":"skills/detecting-ntlm-relay-with-event-correlation"},{"name":"detecting-oauth-token-theft","description":"Detect and respond to OAuth token theft and replay in Microsoft Entra ID (Azure AD), covering access token theft, refresh token replay, Primary Refresh Token (PRT) abuse, pass-the-cookie attacks, and Token Protection conditional access policies. Use for impossible-travel or anomalous token-usage alerts, suspected session hijacking, sign-in log analysis, or configuring token-binding defenses in Azure/M365.","domain":"cybersecurity","path":"skills/detecting-oauth-token-theft"},{"name":"detecting-pass-the-hash-attacks","description":"Detect Pass-the-Hash (T1550.002) attacks by analyzing NTLM authentication patterns, flagging Type 3 logons using NTLM where Kerberos would be expected, and correlating with credential-dumping indicators. Use when threat hunting for lateral movement via stolen NTLM hashes, triaging EDR/SIEM alerts on suspicious NTLM logons, scoping compromise during incident response, or validating detection coverage in a purple team exercise.","domain":"cybersecurity","path":"skills/detecting-pass-the-hash-attacks"},{"name":"detecting-pass-the-ticket-attacks","description":"Detect Kerberos Pass-the-Ticket (PtT) attacks by analyzing Windows Event IDs 4768, 4769, and 4771 for anomalous ticket usage patterns, with detection queries for Splunk and Elastic SIEM. Use when investigating incidents involving stolen or replayed Kerberos tickets, building detection rules or threat hunting queries for ticket abuse, or validating SOC monitoring coverage for credential-theft attack techniques.","domain":"cybersecurity","path":"skills/detecting-pass-the-ticket-attacks"},{"name":"detecting-port-scanning-with-fail2ban","description":"Configures Fail2ban with custom filters and actions to detect port scanning activity, SSH brute force attempts, and network reconnaissance, automatically banning offending IP addresses and alerting security teams to suspicious network probing.","domain":"cybersecurity","path":"skills/detecting-port-scanning-with-fail2ban"},{"name":"detecting-privilege-escalation-attempts","description":"Detect privilege escalation attempts across Windows and Linux, including access token manipulation, UAC bypass, unquoted service path abuse, kernel exploits, and sudo/doas abuse. Use when threat hunting for T1068-style privilege escalation, triaging EDR/SIEM alerts on suspicious privilege changes, scoping compromise during incident response, or validating detection coverage in a purple team exercise.","domain":"cybersecurity","path":"skills/detecting-privilege-escalation-attempts"},{"name":"detecting-privilege-escalation-in-kubernetes-pods","description":"Detects and prevents privilege escalation inside Kubernetes pods by combining admission control (OPA policies), runtime monitoring (Falco), and audit log analysis of security contexts, Linux capabilities, and syscall patterns. Use when investigating a pod running as root or privileged, hardening workloads against in-pod escalation, or hunting for containers exceeding their intended scope. Keywords: allowPrivilegeEscalation, runAsRoot, capabilities, securityContext, OPA, Falco, audit log. Do not use for escalation through RBAC and service-account permissions - use auditing-kubernetes-rbac-privilege-escalation.","domain":"cybersecurity","path":"skills/detecting-privilege-escalation-in-kubernetes-pods"},{"name":"detecting-process-hollowing-technique","description":"Detect process hollowing (MITRE T1055.012) by analyzing memory-mapped sections, hollowed process indicators, and parent-child anomalies using EDR telemetry, Volatility's malfind plugin, pe-sieve, Hollows Hunter, and Sysmon Event ID 25. Use when investigating a legitimate-looking process (svchost.exe, explorer.exe, rundll32.exe) suspected of hosting injected code via NtUnmapViewOfSection.","domain":"cybersecurity","path":"skills/detecting-process-hollowing-technique"},{"name":"detecting-process-injection-techniques","description":"Detects and analyzes process injection techniques used by malware including classic DLL injection, process hollowing, APC injection, thread hijacking, and reflective loading. Uses memory forensics, API monitoring, and behavioral analysis to identify injection artifacts. Activates for requests involving process injection detection, code injection analysis, hollowed process investigation, or in-memory threat detection.","domain":"cybersecurity","path":"skills/detecting-process-injection-techniques"},{"name":"detecting-qr-code-phishing-with-email-security","description":"Detect and prevent QR code phishing (quishing) attacks that embed malicious URLs inside QR code images to bypass link-based email security, using image-based threat detection, OCR/QR decoding, and mobile-side scanning (Microsoft Defender for O365, Proofpoint TAP, Barracuda Multimodal AI). Use when configuring gateway rules against QR phishing or investigating suspicious emails containing QR codes.","domain":"cybersecurity","path":"skills/detecting-qr-code-phishing-with-email-security"},{"name":"detecting-ransomware-encryption-behavior","description":"Detects ransomware encryption activity in real time using entropy analysis, file system I/O monitoring (Sysmon, watchdog, psutil), and behavioral scoring to identify mass file modification, abnormal entropy spikes in written data, and suspicious process behavior characteristic of encryption routines. Use when building real-time ransomware detection, tuning entropy thresholds, or investigating suspected active encryption on an endpoint.","domain":"cybersecurity","path":"skills/detecting-ransomware-encryption-behavior"},{"name":"detecting-ransomware-precursors-in-network","description":"Detects early-stage ransomware indicators in network traffic before encryption begins, including initial access broker activity, command-and-control beaconing, credential harvesting, reconnaissance scanning, and staging behavior. Uses network detection tools (Zeek, Suricata, Arkime), SIEM correlation rules, and threat intelligence feeds to identify ransomware precursor patterns such as Cobalt Strike beacons, Mimikatz network signatures, and RDP brute-force attempts. Activates for requests involving pre-ransomware detection, network-based ransomware indicators, or early warning ransomware monitoring.","domain":"cybersecurity","path":"skills/detecting-ransomware-precursors-in-network"},{"name":"detecting-rdp-brute-force-attacks","description":"Detect RDP brute force attacks by parsing Windows Security Event Logs (EVTX files, via python-evtx) for failed logon patterns (Event ID 4625, Logon Type 10/3), correlating with successful logons (Event ID 4624), and analyzing NLA failures and source IP frequency. Use when investigating exposed RDP endpoints, building SIEM detection rules for credential guessing, or confirming whether a compromised account followed a brute-force pattern.","domain":"cybersecurity","path":"skills/detecting-rdp-brute-force-attacks"},{"name":"detecting-rootkit-activity","description":"Detects rootkit presence on compromised systems by identifying hidden processes, hooked system calls, modified kernel structures, and covert network connections using Volatility memory forensics, cross-view detection, and tools like GMER, rkhunter, chkrootkit, and RootkitRevealer. Use when standard tools (Task Manager, netstat, AV/EDR) show nothing abnormal but compromise is suspected.","domain":"cybersecurity","path":"skills/detecting-rootkit-activity"},{"name":"detecting-s3-data-exfiltration-attempts","description":"Detecting data exfiltration attempts from AWS S3 buckets by analyzing CloudTrail S3 data events, VPC Flow Logs, GuardDuty findings, Amazon Macie alerts, and S3 access patterns to identify unauthorized bulk downloads and cross-account data transfers.","domain":"cybersecurity","path":"skills/detecting-s3-data-exfiltration-attempts"},{"name":"detecting-secure-boot-bypass","description":"Detect UEFI Secure Boot bypasses and bootkits such as BlackLotus and Bootkitty by verifying Secure Boot state, checking dbx revocation currency, and hashing EFI boot binaries against known-bad sets using mokutil, efi-readvar/dbxtool, CHIPSEC, sbverify/pesign, and Windows Confirm-SecureBootUEFI. Use when auditing fleet dbx rollout after a bootkit advisory or hunting for pre-OS persistence on a suspected-compromised endpoint.","domain":"cybersecurity","path":"skills/detecting-secure-boot-bypass"},{"name":"detecting-serverless-function-injection","description":"Detects and prevents code injection attacks targeting serverless functions (AWS Lambda, Azure Functions, Google Cloud Functions) through event source poisoning, malicious layer injection, runtime command execution, and IAM privilege escalation via function modification. The analyst combines static analysis of function code, CloudTrail event correlation, runtime behavior monitoring, and IAM policy auditing to identify injection vectors across the expanded serverless attack surface including API Gateway, S3, SQS, DynamoDB Streams, and CloudWatch event triggers. Activates for requests involving Lambda security assessment, serverless injection detection, function event poisoning analysis, or serverless privilege escalation investigation.","domain":"cybersecurity","path":"skills/detecting-serverless-function-injection"},{"name":"detecting-service-account-abuse","description":"Detect abuse of service accounts by hunting for anomalous interactive logons, privilege escalation, and lateral movement using EDR/SIEM telemetry (CrowdStrike Falcon, Microsoft Defender, Splunk, Elastic Security, Sysmon, Velociraptor) and Sigma detection rules. Use when hunting for service-account misuse or investigating a service account performing unexpected interactive logons.","domain":"cybersecurity","path":"skills/detecting-service-account-abuse"},{"name":"detecting-shadow-api-endpoints","description":"Discover and inventory shadow API endpoints that operate outside documented OpenAPI/Swagger specs, using traffic analysis against API gateways (Kong, AWS API Gateway, Envoy), cloud configuration scanning, and source code repository mining for undocumented routes. Use when assessing API attack surface, auditing for forgotten test environments or deprecated API versions still running, or building an API registration governance policy.","domain":"cybersecurity","path":"skills/detecting-shadow-api-endpoints"},{"name":"detecting-shadow-it-cloud-usage","description":"Detect unauthorized SaaS and cloud service usage (shadow IT) by parsing proxy access logs, DNS query logs, and firewall/netflow data with Python pandas to aggregate traffic by domain, classify domains against known SaaS categories, and score risk by data volume and user count. Use when auditing an organization for unsanctioned cloud/SaaS usage or generating a shadow IT discovery report with remediation recommendations.","domain":"cybersecurity","path":"skills/detecting-shadow-it-cloud-usage"},{"name":"detecting-spearphishing-with-email-gateway","description":"Detect and block spearphishing emails that use personalized, researched content to evade generic spam filters, by configuring email security gateway (SEG) impersonation protection, URL rewriting, attachment sandboxing, and custom detection rules in Microsoft Defender for Office 365, Proofpoint, Mimecast, or Barracuda. Use when hardening a mail gateway against targeted phishing or investigating a suspected spearphishing campaign.","domain":"cybersecurity","path":"skills/detecting-spearphishing-with-email-gateway"},{"name":"detecting-sql-injection-via-waf-logs","description":"Analyze WAF (ModSecurity/AWS WAF/Cloudflare) logs to detect SQL injection attack campaigns. Parses ModSecurity audit logs and JSON WAF event logs to identify SQLi patterns (UNION SELECT, OR 1=1, SLEEP(), BENCHMARK()), tracks attack sources, correlates multi-stage injection attempts, and generates incident reports with OWASP classification.","domain":"cybersecurity","path":"skills/detecting-sql-injection-via-waf-logs"},{"name":"detecting-stuxnet-style-attacks","description":"Detects sophisticated cyber-physical attacks that follow the Stuxnet pattern of modifying PLC logic while spoofing sensor readings to hide the manipulation, using PLC logic integrity monitoring (Claroty xDome, Nozomi Guardian) and physics-based process anomaly detection. Use when hunting for IT-to-OT lateral movement or discrepancies between PLC program state and physical process behavior in ICS/SCADA environments.","domain":"cybersecurity","path":"skills/detecting-stuxnet-style-attacks"},{"name":"detecting-supply-chain-attacks-in-ci-cd","description":"Scans GitHub Actions workflows and CI/CD pipeline configurations for supply chain attack vectors including unpinned actions, script injection via expressions, dependency confusion, and secrets exposure. Uses PyGithub and YAML parsing for automated audit. Use when hardening CI/CD pipelines or investigating compromised build systems.","domain":"cybersecurity","path":"skills/detecting-supply-chain-attacks-in-ci-cd"},{"name":"detecting-suspicious-oauth-application-consent","description":"Detect risky OAuth application consent grants in Azure AD / Microsoft Entra ID using Microsoft Graph API, audit logs, and permission analysis to identify illicit consent grant attacks.","domain":"cybersecurity","path":"skills/detecting-suspicious-oauth-application-consent"},{"name":"detecting-suspicious-powershell-execution","description":"Hunt for suspicious PowerShell execution (T1059.001) such as encoded commands, download cradles, AMSI bypass, and constrained language mode evasion using EDR telemetry (CrowdStrike, Microsoft Defender for Endpoint), Sysmon, and SIEM queries (Splunk, Elastic). Use when proactively threat hunting, triaging EDR/SIEM alerts, or scoping an incident involving malicious PowerShell activity.","domain":"cybersecurity","path":"skills/detecting-suspicious-powershell-execution"},{"name":"detecting-t1003-credential-dumping-with-edr","description":"Detect OS credential dumping (MITRE T1003) targeting LSASS memory, the SAM database, NTDS.dit, and cached credentials by correlating EDR telemetry, Sysmon process-access events, and Windows security event logs. Use when hunting for Mimikatz-style credential theft, triaging an EDR alert on LSASS access, or scoping an incident after suspected credential dumping.","domain":"cybersecurity","path":"skills/detecting-t1003-credential-dumping-with-edr"},{"name":"detecting-t1055-process-injection-with-sysmon","description":"Detect process injection techniques (T1055) - including DLL injection, process hollowing, and APC injection - by analyzing Sysmon Event IDs 1, 7, 8, 10, and 25 for cross-process memory operations, remote thread creation, and anomalous DLL loads. Use when hunting defense-evasion activity that hides code inside legitimate processes, investigating an EDR alert on suspicious cross-process access, or validating Sysmon coverage for injection detection.","domain":"cybersecurity","path":"skills/detecting-t1055-process-injection-with-sysmon"},{"name":"detecting-t1548-abuse-elevation-control-mechanism","description":"Detect abuse of elevation control mechanisms (T1548), including Windows UAC bypass via auto-elevating binaries like fodhelper.exe and Linux sudo/setuid/setgid exploitation, by monitoring registry changes, integrity-level transitions, and parent-child process relationships via Sysmon and Windows Security events. Use when hunting privilege-escalation activity or validating elevation-abuse detection coverage.","domain":"cybersecurity","path":"skills/detecting-t1548-abuse-elevation-control-mechanism"},{"name":"detecting-typosquatting-packages","description":"Flag misspelled, brandjacked, and typosquatted package names across npm, PyPI, and crates.io before installation, using edit-distance, keyboard-proximity, and known-target corpus matching with typomania, Microsoft OSSGadget's oss-find-squats, and pypi-scan. Use before adding a new dependency, as a CI/CD gate on pull requests, when reviewing an AI-generated (\"slopsquatting\") dependency list, or when auditing a lockfile diff for a swapped package.","domain":"cybersecurity","path":"skills/detecting-typosquatting-packages"},{"name":"detecting-typosquatting-packages-in-npm-pypi","description":"Detects typosquatting attacks in npm and PyPI package registries by analyzing package name similarity using Levenshtein distance and other string metrics, examining publish date heuristics to identify recently created packages mimicking established ones, and flagging download count anomalies where suspicious packages have disproportionately low usage compared to their legitimate targets. The analyst queries the PyPI JSON API and npm registry API to gather package metadata for automated comparison. Activates for requests involving package typosquatting detection, dependency confusion analysis, malicious package identification, or software supply chain threat hunting in package registries.","domain":"cybersecurity","path":"skills/detecting-typosquatting-packages-in-npm-pypi"},{"name":"detecting-wmi-persistence","description":"Detect WMI event subscription persistence (MITRE T1546.003) by analyzing Sysmon Event IDs 19, 20, and 21 for malicious EventFilter, EventConsumer, and FilterToConsumerBinding creation, cross-referenced against root\\subscription namespace contents and Sysinternals Autoruns. Use when hunting WMI-based persistence, triaging a Sysmon alert on these event IDs, or during incident response and purple-team validation of WMI defenses.","domain":"cybersecurity","path":"skills/detecting-wmi-persistence"},{"name":"emulating-cloud-attacks-with-stratus-red-team","description":"Install and run Stratus Red Team to detonate granular, MITRE ATT&CK-mapped AWS, Azure, GCP, and Kubernetes attack techniques through their warmup-detonate-revert-cleanup lifecycle, then verify the telemetry they generate. Use to validate that cloud detections (CloudTrail, GuardDuty, Microsoft Sentinel, GCP SCC, Falco) actually fire, or to run a repeatable purple-team exercise generating realistic attacker telemetry.","domain":"cybersecurity","path":"skills/emulating-cloud-attacks-with-stratus-red-team"},{"name":"enumerating-cloud-with-cloudfox","description":"Run CloudFox's read-only Describe/List/Get enumeration (all-checks, role-trusts, secrets, endpoints, and permissions commands) to map AWS and Azure attack paths and surface exploitable misconfigurations. Use immediately after obtaining a cloud credential to build situational awareness, find exposed resources and secrets, or map sts:AssumeRole trust relationships for lateral-movement/privilege-escalation planning.","domain":"cybersecurity","path":"skills/enumerating-cloud-with-cloudfox"},{"name":"eradicating-malware-from-infected-systems","description":"Systematically map and remove malware, backdoors, and attacker persistence mechanisms (registry Run keys, scheduled tasks, WMI subscriptions, services, cron/init.d) from infected Windows and Linux systems using Autoruns, EDR/AV, and YARA, restoring a clean state while preventing re-infection. Use after containment and forensic analysis have identified all compromised systems and persistence mechanisms and you are ready to eradicate and recover.","domain":"cybersecurity","path":"skills/eradicating-malware-from-infected-systems"},{"name":"escaping-containers-to-host","description":"Exploits privileged pods, host mounts, runC CVEs, and exposed Docker sockets to break out of a container and reach the underlying host during an authorized container-security assessment. Use when executing an approved breakout test, demonstrating the real impact of a privileged or hostPath workload, or validating that escape mitigations actually hold. Keywords: container breakout, privileged, hostPath, docker.sock, runC CVE-2019-5736, CVE-2024-21626, release_agent, nsenter. Do not use for defensive detection of these techniques - use detecting-container-escape-attempts.","domain":"cybersecurity","path":"skills/escaping-containers-to-host"},{"name":"evaluating-threat-intelligence-platforms","description":"Evaluates and selects Threat Intelligence Platform (TIP) products based on organizational requirements including feed integration capability, STIX/TAXII support, workflow automation, analyst interface, and total cost of ownership. Use when conducting a TIP procurement, migrating between TIP solutions, or assessing whether the current TIP meets program maturity requirements. Activates for requests involving ThreatConnect, MISP, OpenCTI, Anomali, EclecticIQ, or TIP procurement decisions.","domain":"cybersecurity","path":"skills/evaluating-threat-intelligence-platforms"},{"name":"executing-active-directory-attack-simulation","description":"Executes authorized attack simulations against Active Directory environments to identify misconfigurations, weak credentials, dangerous privilege paths, and exploitable trust relationships that could lead to domain compromise. The tester uses BloodHound for attack path analysis, Mimikatz for credential extraction, and Impacket for protocol-level attacks including Kerberoasting, AS-REP Roasting, and delegation abuse. Activates for requests involving Active Directory pentest, AD attack simulation, domain compromise testing, or Kerberos attack assessment.","domain":"cybersecurity","path":"skills/executing-active-directory-attack-simulation"},{"name":"executing-nist-rmf-authorization-to-operate","description":"Drive a federal system through the NIST Risk Management Framework (SP 800-37 Rev 2) to an Authorization to Operate (ATO): Prepare, Categorize (FIPS 199), Select a control baseline (FIPS 200 / SP 800-53 Rev 5), Implement, Assess (SP 800-53A), Authorize, and Monitor continuously. Use when a system needs an ATO or a renewal, when working a FISMA/FedRAMP authorization package, when building or reviewing an SSP, SAR, or POA&M, when categorizing a system as Low/Moderate/High impact, when selecting or tailoring a control baseline, or when standing up continuous monitoring (ConMon) after authorization. Covers ATO, conditional ATO (cATO), and the artifacts assessors expect. Keywords: NIST RMF, 800-37, ATO, authorization to operate, FISMA, FedRAMP, SSP, SAR, POA&M, FIPS 199, FIPS 200, 800-53, 800-53A, control baseline, security categorization, continuous monitoring, authorizing official, system boundary, ongoing authorization.","domain":"cybersecurity","path":"skills/executing-nist-rmf-authorization-to-operate"},{"name":"executing-phishing-simulation-campaign","description":"Executes authorized phishing simulation campaigns to assess an organization's susceptibility to email-based social engineering attacks. The tester designs realistic phishing scenarios, builds credential harvesting infrastructure, sends targeted phishing emails, and tracks open rates, click-through rates, and credential submission rates to measure human security awareness. Activates for requests involving phishing simulation, social engineering assessment, email security testing, or security awareness measurement.","domain":"cybersecurity","path":"skills/executing-phishing-simulation-campaign"},{"name":"executing-red-team-engagement-planning","description":"Build the foundational red team engagement plan - scope definition, Rules of Engagement (restrictions, communication plan, emergency stop procedures, legal authorization), MITRE ATT&CK-aligned threat profile selection, and operational timelines - producing an engagement brief for stakeholder approval. Use before any offensive testing begins, when scoping a full-scope, assumed-breach, objective-based, or purple-team engagement.","domain":"cybersecurity","path":"skills/executing-red-team-engagement-planning"},{"name":"executing-red-team-exercise","description":"Run a stealthy, MITRE ATT&CK-mapped adversary emulation against an organization's people, processes, and technology, using C2 infrastructure (Cobalt Strike, Sliver, Brute Ratel, Mythic) through the full attack lifecycle to test SOC detection and response rather than just find vulnerabilities. Use when executing a full-scope red team exercise, adversary simulation/emulation, or a regulatory exercise (TIBER-EU, CBEST, AASE, iCAST) with executive authorization and a signed Rules of Engagement already in place.","domain":"cybersecurity","path":"skills/executing-red-team-exercise"},{"name":"exploiting-active-directory-certificate-services-esc1","description":"Exploit misconfigured Active Directory Certificate Services (AD CS) ESC1 vulnerability to request certificates as high-privileged users and escalate domain privileges during authorized red team assessments.","domain":"cybersecurity","path":"skills/exploiting-active-directory-certificate-services-esc1"},{"name":"exploiting-active-directory-with-bloodhound","description":"BloodHound is a graph-based Active Directory reconnaissance tool that uses graph theory to reveal hidden and unintended relationships within AD environments. Red teams use BloodHound to identify attac","domain":"cybersecurity","path":"skills/exploiting-active-directory-with-bloodhound"},{"name":"exploiting-adcs-with-certipy","description":"Use Certipy to enumerate AD CS certificate authorities and templates over LDAP/RPC, then exploit ESC1-ESC16 misconfigurations - SAN abuse, NTLM relay to web enrollment (ESC8), Shadow Credentials, golden certificate forgery, and PKINIT/Schannel auth. Use during authorized penetration tests to escalate a domain foothold to Domain Admin, or to validate that certificate template ACLs and CA hardening detect these attacks.","domain":"cybersecurity","path":"skills/exploiting-adcs-with-certipy"},{"name":"exploiting-api-injection-vulnerabilities","description":"Tests API parameters, headers, and request bodies for injection flaws \u2014 SQL injection, NoSQL injection, OS command injection, LDAP injection, and SSRF \u2014 by crafting payloads tailored to the target backend to extract data, execute commands, or reach internal services, mapped to OWASP API8:2023 and API7:2023 SSRF. Use when performing SQLi, NoSQL injection, command injection, or SSRF testing against APIs, or assessing API input validation.","domain":"cybersecurity","path":"skills/exploiting-api-injection-vulnerabilities"},{"name":"exploiting-aws-with-pacu","description":"Runs the Pacu AWS exploitation framework end-to-end \u2014 session and credential setup, IAM enumeration, automated privilege-escalation scanning via iam__privesc_scan, and persistence/backdooring modules like iam__backdoor_users_keys. Use for authorized AWS penetration tests or red-team engagements, assessing blast radius from a compromised IAM credential, or generating attacker telemetry for purple-team detection testing.","domain":"cybersecurity","path":"skills/exploiting-aws-with-pacu"},{"name":"exploiting-bgp-hijacking-vulnerabilities","description":"Analyzes and simulates BGP hijacking scenarios in authorized lab environments to assess route origin validation, RPKI deployment, and BGP monitoring defenses against prefix hijacking and route leak attacks on internet routing infrastructure.","domain":"cybersecurity","path":"skills/exploiting-bgp-hijacking-vulnerabilities"},{"name":"exploiting-broken-function-level-authorization","description":"Tests APIs for Broken Function Level Authorization (OWASP API5:2023) by identifying admin and privileged endpoints, then reaching them with regular-user credentials via HTTP method switching, URL path manipulation, and parameter tampering. Use when testing whether low-privilege users can invoke admin API functions or otherwise escalate privileges via function-level access control gaps.","domain":"cybersecurity","path":"skills/exploiting-broken-function-level-authorization"},{"name":"exploiting-broken-link-hijacking","description":"Discovers and exploits broken link hijacking by spidering a site (Burp Suite Spider, Scrapy, curl scraping), extracting referenced external scripts/domains, and checking DNS/CNAME records and domain registration status for expired or unclaimed resources an attacker could register. Use for subdomain takeover testing, supply-chain review of third-party scripts, or bug bounty hunting for hijackable external resources.","domain":"cybersecurity","path":"skills/exploiting-broken-link-hijacking"},{"name":"exploiting-constrained-delegation-abuse","description":"Exploits Kerberos Constrained Delegation misconfigurations in Active Directory using Impacket's findDelegation.py and getST.py (or Rubeus/Kekeo on Windows) to abuse S4U2Self and S4U2Proxy and impersonate privileged users. Use during authorized Active Directory penetration tests or red-team engagements for lateral movement and privilege escalation after finding an account trusted for constrained delegation.","domain":"cybersecurity","path":"skills/exploiting-constrained-delegation-abuse"},{"name":"exploiting-deeplink-vulnerabilities","description":"Tests and exploits deep link (URL scheme and App Link) vulnerabilities in Android and iOS mobile applications to identify unauthorized access, data injection, intent hijacking, and redirect manipulation. Use when assessing mobile app attack surface through custom URI schemes, Android App Links, iOS Universal Links, or intent-based navigation. Activates for requests involving deep link security testing, URL scheme exploitation, mobile intent abuse, or link hijacking.","domain":"cybersecurity","path":"skills/exploiting-deeplink-vulnerabilities"},{"name":"exploiting-excessive-data-exposure-in-api","description":"Tests APIs for excessive data exposure (OWASP API3:2023) by intercepting raw API responses and comparing them against what the UI actually renders, looking for leaked PII, internal identifiers, debug data, or business-sensitive fields the frontend filters but the API still transmits. Use when auditing REST or mobile-app APIs for over-fetching, response filtering bypass, or unintended data leakage in endpoint responses.","domain":"cybersecurity","path":"skills/exploiting-excessive-data-exposure-in-api"},{"name":"exploiting-http-request-smuggling","description":"Detects and exploits HTTP request smuggling caused by Content-Length/Transfer-Encoding parsing discrepancies between front-end and back-end servers, using Burp Suite Repeater (auto Content-Length disabled), the HTTP Request Smuggler extension, and smuggler.py. Use during authorized tests of multi-tier architectures behind a reverse proxy, load balancer, or CDN to find desync flaws and bypass front-end controls.","domain":"cybersecurity","path":"skills/exploiting-http-request-smuggling"},{"name":"exploiting-idor-vulnerabilities","description":"Identifies and exploits Insecure Direct Object Reference (IDOR) vulnerabilities by manipulating object identifiers (numeric IDs, UUIDs, slugs) in API requests and URLs, using Burp Suite proxy history, Intruder, and the Authorize extension to test object-level authorization across sessions. Use during authorized penetration tests or bug bounty work to validate that CRUD endpoints and multi-tenant applications enforce per-object access control.","domain":"cybersecurity","path":"skills/exploiting-idor-vulnerabilities"},{"name":"exploiting-insecure-data-storage-in-mobile","description":"Identifies and exploits insecure local data storage vulnerabilities in Android and iOS mobile applications including unencrypted databases, world-readable files, insecure SharedPreferences, plaintext credential storage, and improper keychain/keystore usage. Use when performing mobile penetration testing focused on OWASP M9 (Insecure Data Storage) or assessing compliance with MASVS-STORAGE requirements. Activates for requests involving mobile data storage security, local storage exploitation, SharedPreferences analysis, or mobile data leakage assessment.","domain":"cybersecurity","path":"skills/exploiting-insecure-data-storage-in-mobile"},{"name":"exploiting-insecure-deserialization","description":"Identifying and exploiting insecure deserialization vulnerabilities in Java, PHP, Python, and .NET applications to achieve remote code execution during authorized penetration tests.","domain":"cybersecurity","path":"skills/exploiting-insecure-deserialization"},{"name":"exploiting-ipv6-vulnerabilities","description":"Identifies and exploits IPv6-specific vulnerabilities including SLAAC spoofing, Router Advertisement flooding, and IPv6 tunneling during authorized assessments to test dual-stack security controls and IPv6-aware network defenses.","domain":"cybersecurity","path":"skills/exploiting-ipv6-vulnerabilities"},{"name":"exploiting-jwt-algorithm-confusion-attack","description":"Exploits JWT algorithm confusion where the server's verification library trusts the alg named in the token header, by switching RS256 to HS256 (signing with the RSA public key as HMAC secret), setting alg to none, or injecting kid/jku/x5u headers to supply an attacker-controlled key. Use when testing RS256 JWT auth for algorithm downgrade, alg:none bypass, or key-confusion signature forgery.","domain":"cybersecurity","path":"skills/exploiting-jwt-algorithm-confusion-attack"},{"name":"exploiting-kerberoasting-with-impacket","description":"Performs Kerberoasting (MITRE ATT&CK T1558.003) using Impacket's GetUserSPNs.py to request Kerberos TGS tickets for SPN-registered service accounts, then cracks the extracted RC4/AES-encrypted hashes offline to recover service account credentials. Use during authorized Active Directory penetration tests or red-team engagements for credential access against service accounts via Kerberos ticket-granting-service requests.","domain":"cybersecurity","path":"skills/exploiting-kerberoasting-with-impacket"},{"name":"exploiting-mass-assignment-in-rest-apis","description":"Discovers and exploits mass assignment (autobinding) in REST APIs by injecting unexpected or hidden parameters (e.g. role, isAdmin, plan) into create/update requests, using Burp Suite Intruder, Arjun, and param-miner to find bindable fields on ORM-backed endpoints (Rails, Django, Laravel, Spring). Use when testing REST APIs for privilege escalation or authorization bypass via unintended parameter binding.","domain":"cybersecurity","path":"skills/exploiting-mass-assignment-in-rest-apis"},{"name":"exploiting-ms17-010-eternalblue-vulnerability","description":"Detects and exploits MS17-010 (EternalBlue), a critical remote code execution flaw in Microsoft's SMBv1 implementation, using Nmap's ms-17-010 NSE script for detection and Metasploit's ms17_010_eternalblue/ms17_010_psexec modules for exploitation. Use during authorized red-team engagements or penetration tests against legacy Windows environments with unpatched SMBv1 to gain remote code execution.","domain":"cybersecurity","path":"skills/exploiting-ms17-010-eternalblue-vulnerability"},{"name":"exploiting-nopac-cve-2021-42278-42287","description":"Exploits the noPac Active Directory privilege-escalation chain (CVE-2021-42278 sAMAccountName spoofing plus CVE-2021-42287 KDC PAC confusion) using Impacket and secretsdump.py to escalate from a standard domain user to Domain Admin. Use when red-teaming or pentesting Active Directory to validate machine-account impersonation and DCSync-based credential dumping.","domain":"cybersecurity","path":"skills/exploiting-nopac-cve-2021-42278-42287"},{"name":"exploiting-nosql-injection-vulnerabilities","description":"Detects and exploits NoSQL injection vulnerabilities in MongoDB, CouchDB, and similar databases to demonstrate authentication bypass, data extraction, and unauthorized access via crafted query operators. Use when pentesting APIs or web applications backed by NoSQL databases to test input validation and injection defenses.","domain":"cybersecurity","path":"skills/exploiting-nosql-injection-vulnerabilities"},{"name":"exploiting-oauth-misconfiguration","description":"Identifying and exploiting OAuth 2.0 and OpenID Connect misconfigurations including redirect URI manipulation, token leakage, and authorization code theft during security assessments.","domain":"cybersecurity","path":"skills/exploiting-oauth-misconfiguration"},{"name":"exploiting-prototype-pollution-in-javascript","description":"Detects and exploits JavaScript prototype pollution vulnerabilities in client-side and server-side (Node.js) applications to achieve XSS, RCE, or authentication bypass through property injection into Object.prototype. Use when assessing a JavaScript/Node.js application for prototype pollution, especially where a merge, clone, or extend function accepts user-controlled keys.","domain":"cybersecurity","path":"skills/exploiting-prototype-pollution-in-javascript"},{"name":"exploiting-race-condition-vulnerabilities","description":"Detects and exploits race condition (TOCTOU) vulnerabilities in web applications using Burp Suite's Turbo Intruder extension and its single-packet attack technique to fire parallel requests that bypass rate limits, duplicate transactions, or overrun usage limits. Use when pentesting endpoints with balances, coupon redemption, or rate limiting that concurrent requests might manipulate.","domain":"cybersecurity","path":"skills/exploiting-race-condition-vulnerabilities"},{"name":"exploiting-server-side-request-forgery","description":"Identifying and exploiting SSRF vulnerabilities to access internal services, cloud metadata, and restricted network resources during authorized penetration tests.","domain":"cybersecurity","path":"skills/exploiting-server-side-request-forgery"},{"name":"exploiting-smb-vulnerabilities-with-metasploit","description":"Identifies and exploits SMB protocol vulnerabilities using Metasploit Framework during authorized penetration tests to demonstrate risks from unpatched Windows systems, misconfigured shares, and weak authentication in enterprise networks.","domain":"cybersecurity","path":"skills/exploiting-smb-vulnerabilities-with-metasploit"},{"name":"exploiting-sql-injection-vulnerabilities","description":"Identifies and exploits SQL injection vulnerabilities in web applications during authorized penetration tests using manual techniques and automated tools like sqlmap. The tester detects injection points through error-based, union-based, blind boolean, and time-based blind techniques across all major database engines (MySQL, PostgreSQL, MSSQL, Oracle) to demonstrate data extraction, authentication bypass, and potential remote code execution. Activates for requests involving SQL injection testing, SQLi exploitation, database security assessment, or injection vulnerability verification.","domain":"cybersecurity","path":"skills/exploiting-sql-injection-vulnerabilities"},{"name":"exploiting-sql-injection-with-sqlmap","description":"Detecting and exploiting SQL injection vulnerabilities using sqlmap to extract database contents during authorized penetration tests.","domain":"cybersecurity","path":"skills/exploiting-sql-injection-with-sqlmap"},{"name":"exploiting-template-injection-vulnerabilities","description":"Detects and exploits Server-Side Template Injection (SSTI) vulnerabilities across Jinja2, Twig, Freemarker, and other template engines to achieve remote code execution. Use when pentesting a web application that renders user input through a server-side template engine and you need to confirm and weaponize SSTI.","domain":"cybersecurity","path":"skills/exploiting-template-injection-vulnerabilities"},{"name":"exploiting-type-juggling-vulnerabilities","description":"Exploits PHP type juggling vulnerabilities caused by loose (==) comparison operators to bypass authentication, defeat hash verification via magic hashes, and manipulate application logic through type coercion. Use when pentesting PHP applications that compare user input with ==, in_array, or hash functions without strict type checks.","domain":"cybersecurity","path":"skills/exploiting-type-juggling-vulnerabilities"},{"name":"exploiting-vulnerabilities-with-metasploit-framework","description":"Uses the Metasploit Framework (msfconsole and its exploit, auxiliary, and post-exploitation modules) to validate that identified CVEs and vulnerabilities are actually exploitable, gather post-exploitation evidence, and confirm patch remediation. Use when performing vulnerability management validation, penetration testing, or post-patch verification and you need to prove real-world exploitability rather than rely on a scanner score alone.","domain":"cybersecurity","path":"skills/exploiting-vulnerabilities-with-metasploit-framework"},{"name":"exploiting-websocket-vulnerabilities","description":"Testing WebSocket implementations for authentication bypass, cross-site hijacking, injection attacks, and insecure message handling during authorized security assessments.","domain":"cybersecurity","path":"skills/exploiting-websocket-vulnerabilities"},{"name":"exploiting-zerologon-vulnerability-cve-2020-1472","description":"Exploits the Zerologon vulnerability (CVE-2020-1472) in the Netlogon Remote Protocol using Impacket to reset a domain controller's machine account password to empty, then runs DCSync via secretsdump.py to dump domain credentials. Use when red-teaming or validating unpatched Active Directory domain controllers for Zerologon, including restoring the machine account password afterward.","domain":"cybersecurity","path":"skills/exploiting-zerologon-vulnerability-cve-2020-1472"},{"name":"extracting-browser-history-artifacts","description":"Extracts and analyzes browser history, cookies, cache, downloads, and bookmarks from Chrome, Firefox, and Edge using sqlite3, DB Browser for SQLite, Hindsight, and NirSoft tools (BrowsingHistoryView, ChromeCacheView, MZCacheView). Use when performing digital forensics or incident response on a disk image or live system and you need timeline evidence of a user's web activity.","domain":"cybersecurity","path":"skills/extracting-browser-history-artifacts"},{"name":"extracting-config-from-agent-tesla-rat","description":"Extracts embedded configuration from Agent Tesla RAT samples, including SMTP/FTP/Telegram exfiltration credentials, keylogger settings, and C2 endpoints, via .NET decompilation and memory analysis. Use when analyzing a suspected or confirmed Agent Tesla sample and you need to recover its exfiltration channel and C2 configuration for threat intelligence or incident response.","domain":"cybersecurity","path":"skills/extracting-config-from-agent-tesla-rat"},{"name":"extracting-credentials-from-memory-dump","description":"Extracts cached credentials, password hashes, Kerberos tickets, and authentication tokens from Windows memory dumps using Volatility 3, Mimikatz, and pypykatz. Use when performing memory forensics or incident response on an LSASS or full memory dump and you need to recover credentials or Kerberos material for investigation.","domain":"cybersecurity","path":"skills/extracting-credentials-from-memory-dump"},{"name":"extracting-iocs-from-malware-samples","description":"Extracts indicators of compromise (IOCs) from malware samples, including file hashes, network indicators (IPs, domains, URLs, PCAP indicators), host artifacts (file paths, registry keys, mutexes), and behavioral patterns, using tools like CyberChef, then defangs and exports them in standard threat-intel formats. Use for IOC extraction, threat indicator harvesting, or building detection content from a sample.","domain":"cybersecurity","path":"skills/extracting-iocs-from-malware-samples"},{"name":"extracting-memory-artifacts-with-rekall","description":"Uses Rekall memory forensics framework to analyze memory dumps for process hollowing, injected code via VAD anomalies, hidden processes, and rootkit detection. Applies plugins like pslist, psscan, vadinfo, malfind, and dlllist to extract forensic artifacts from Windows memory images. Use during incident response memory analysis.","domain":"cybersecurity","path":"skills/extracting-memory-artifacts-with-rekall"},{"name":"extracting-windows-event-logs-artifacts","description":"Extract, parse, and analyze Windows Event Logs (EVTX) using Chainsaw, Hayabusa, and EvtxECmd to detect lateral movement, persistence, and privilege escalation.","domain":"cybersecurity","path":"skills/extracting-windows-event-logs-artifacts"},{"name":"fleet-hunting-with-velociraptor","description":"Deploy a Velociraptor server and agents, then author VQL (Velociraptor Query Language) artifacts and run them as fleet-wide hunts, on-demand forensic collections, or standalone offline collectors. Use when hunting a TTP across hundreds or thousands of endpoints, collecting forensic artifacts during incident response without re-imaging, or generating collectors for unmanaged/air-gapped hosts.","domain":"cybersecurity","path":"skills/fleet-hunting-with-velociraptor"},{"name":"generating-and-analyzing-sboms","description":"Generate CycloneDX and SPDX SBOMs from container images and filesystems with Syft, correlate them to CVEs with Grype, and sign/attest them with Cosign. Use when you need a machine-readable dependency inventory for supply-chain risk, want to scan images or SBOMs for known vulnerabilities, or are embedding SBOM generation and vulnerability gating into CI/CD.","domain":"cybersecurity","path":"skills/generating-and-analyzing-sboms"},{"name":"generating-forensic-timelines-with-hayabusa","description":"Run Hayabusa against collected Windows EVTX files to apply Sigma detection rules and produce a prioritized, chronological CSV/JSON timeline with severity levels, MITRE ATT&CK mappings, and per-host/per-Event-ID metrics. Use during DFIR triage to turn raw event logs into a fast, SIEM-free incident timeline, or to export results into Timesketch or Timeline Explorer for collaborative analysis.","domain":"cybersecurity","path":"skills/generating-forensic-timelines-with-hayabusa"},{"name":"generating-threat-intelligence-reports","description":"Generates structured cyber threat intelligence reports at strategic, operational, and tactical levels tailored to specific audiences including executives, security operations teams, and technical analysts. Use when producing finished intelligence products from raw collection data, creating sector threat briefings, or delivering post-incident intelligence assessments. Activates for requests involving CTI report writing, threat briefings, intelligence products, finished intelligence, or executive security reporting.","domain":"cybersecurity","path":"skills/generating-threat-intelligence-reports"},{"name":"hardening-docker-containers-for-production","description":"Hardens Dockerfiles, images, and per-container runtime settings against the CIS Docker Benchmark v1.8.0: non-root users, dropped capabilities, read-only root filesystem, seccomp and AppArmor profiles, and minimal multi-stage builds, validated with docker-bench-security, Hadolint, and Dockle. Use when preparing a container or Dockerfile for production, or auditing images and runtime flags against CIS Docker controls. Keywords: Dockerfile, USER, --cap-drop, read-only rootfs, seccomp, AppArmor, multi-stage, Hadolint, Dockle. Do not use for the Docker daemon's own configuration - use hardening-docker-daemon-configuration.","domain":"cybersecurity","path":"skills/hardening-docker-containers-for-production"},{"name":"hardening-docker-daemon-configuration","description":"Hardens the Docker daemon (dockerd) through /etc/docker/daemon.json with user namespace remapping, TLS client authentication, seccomp profiles, and CIS Docker Benchmark controls such as icc, no-new-privileges, and live-restore. Use when securing a Docker host's daemon to prevent privilege escalation, breakout, or lateral movement, or when auditing daemon settings against CIS requirements. Keywords: dockerd, daemon.json, userns-remap, no-new-privileges, icc, live-restore, TLS socket. Do not use for hardening images and per-container runtime flags - use hardening-docker-containers-for-production.","domain":"cybersecurity","path":"skills/hardening-docker-daemon-configuration"},{"name":"hardening-linux-endpoint-with-cis-benchmark","description":"Hardens Linux endpoints using CIS Benchmark recommendations for Ubuntu, RHEL, and CentOS to reduce attack surface, enforce security baselines, and meet compliance requirements. Use when deploying new Linux servers, remediating audit findings, or establishing security baselines for Linux infrastructure. Activates for requests involving Linux hardening, CIS benchmarks for Linux, server security baselines, or Linux configuration compliance.","domain":"cybersecurity","path":"skills/hardening-linux-endpoint-with-cis-benchmark"},{"name":"hardening-windows-endpoint-with-cis-benchmark","description":"Hardens Windows endpoints using CIS (Center for Internet Security) Benchmark recommendations to reduce attack surface, enforce security baselines, and meet compliance requirements. Use when deploying new Windows workstations or servers, remediating audit findings, or establishing organization-wide security baselines. Activates for requests involving Windows hardening, CIS benchmarks, GPO security baselines, or endpoint configuration compliance.","domain":"cybersecurity","path":"skills/hardening-windows-endpoint-with-cis-benchmark"},{"name":"hunting-advanced-persistent-threats","description":"Proactively hunts for Advanced Persistent Threat (APT) activity within enterprise environments using hypothesis-driven searches across endpoint telemetry, network logs, and memory artifacts. Use when conducting scheduled threat hunting cycles, investigating anomalous behavior flagged by UEBA, or validating that known APT TTPs are not present in the environment. Activates for requests involving MITRE ATT&CK, Velociraptor, osquery, Zeek, or threat hunting playbooks.","domain":"cybersecurity","path":"skills/hunting-advanced-persistent-threats"},{"name":"hunting-bootkits-in-efi-system-partition","description":"Baseline the EFI System Partition and hunt malicious EFI binaries such as ESPecter, BlackLotus, Bootkitty, and Glupteba by mounting the ESP, hashing and verifying boot loaders, scanning with YARA, and flagging anomalous non-EFI files or out-of-band bootloader changes. Use during proactive threat hunts for firmware/bootkit persistence (MITRE ATT&CK T1542.003) or when investigating suspected UEFI-level compromise that survives OS reinstall.","domain":"cybersecurity","path":"skills/hunting-bootkits-in-efi-system-partition"},{"name":"hunting-credential-stuffing-attacks","description":"Detects credential stuffing attacks by analyzing authentication logs for login velocity anomalies, ASN diversity, password spray patterns, and geographic distribution of failed logins. Uses statistical analysis on Splunk or raw log data. Use when investigating account takeover campaigns or building detection rules for auth abuse.","domain":"cybersecurity","path":"skills/hunting-credential-stuffing-attacks"},{"name":"hunting-evtx-with-chainsaw","description":"Run Chainsaw against collected Windows EVTX files to hunt with the SigmaHQ rule corpus, built-in detection rules, and high-speed keyword/regex search, plus analyze shimcache, SRUM, and event-log gaps, outputting colorized tables, CSV, or JSON. Use during first-response triage for offline, SIEM-free detection over Windows event logs, or to quickly confirm a hunt hypothesis and produce reporting output.","domain":"cybersecurity","path":"skills/hunting-evtx-with-chainsaw"},{"name":"hunting-for-anomalous-powershell-execution","description":"Hunt for malicious PowerShell activity by analyzing Script Block Logging (Event 4104), Module Logging (Event 4103), and process creation events. The analyst parses Windows Event Log EVTX files to detect obfuscated commands, AMSI bypass attempts, encoded payloads, credential dumping keywords, and suspicious download cradles. Activates for requests involving PowerShell threat hunting, script block analysis, encoded command detection, or AMSI bypass identification.","domain":"cybersecurity","path":"skills/hunting-for-anomalous-powershell-execution"},{"name":"hunting-for-beaconing-with-frequency-analysis","description":"Identify command-and-control beaconing patterns in network traffic by applying statistical frequency analysis, jitter calculation, and coefficient of variation scoring to detect periodic callbacks from compromised endpoints.","domain":"cybersecurity","path":"skills/hunting-for-beaconing-with-frequency-analysis"},{"name":"hunting-for-cobalt-strike-beacons","description":"Detect Cobalt Strike beacon command-and-control traffic using default TLS certificate signatures (serial 8BB00EE), JA3/JA3S/JARM fingerprints, HTTP malleable C2 profile pattern matching, and beacon jitter/interval analysis, built with Zeek network logs, Suricata IDS rules, and Python PCAP analysis. Use when hunting for Cobalt Strike beacon callbacks in network traffic or building detection rules for this C2 framework.","domain":"cybersecurity","path":"skills/hunting-for-cobalt-strike-beacons"},{"name":"hunting-for-command-and-control-beaconing","description":"Detect C2 beaconing patterns in network traffic using frequency analysis, jitter detection, and domain reputation to identify compromised endpoints communicating with adversary infrastructure.","domain":"cybersecurity","path":"skills/hunting-for-command-and-control-beaconing"},{"name":"hunting-for-data-exfiltration-indicators","description":"Hunt for data exfiltration by analyzing Zeek and Suricata network telemetry for unusual data flows, DNS tunneling via large/frequent TXT queries, uploads to personal cloud storage, and encrypted-channel abuse, correlated against threat intel on destination domains. Use when hunting for data theft in a compromised environment, investigating unusual outbound data volumes, or determining what data was stolen during incident response.","domain":"cybersecurity","path":"skills/hunting-for-data-exfiltration-indicators"},{"name":"hunting-for-data-staging-before-exfiltration","description":"Detect data-staging activity (MITRE ATT&CK T1074) by analyzing EDR/Sysmon process-creation and file-system telemetry (Event ID 4688, Sysmon 1/11) for 7-Zip/RAR/tar archive creation, unusual temp or hidden folder access, and anomalous consolidation of files from multiple directories. Use when hunting for pre-exfiltration staging behavior, building detection rules for archiver abuse, or validating monitoring coverage for T1074.","domain":"cybersecurity","path":"skills/hunting-for-data-staging-before-exfiltration"},{"name":"hunting-for-dcom-lateral-movement","description":"Hunt for DCOM-based lateral movement (MITRE ATT&CK T1021.003) by detecting abuse of MMC20.Application, ShellBrowserWindow, and ShellWindows COM objects via Sysmon Event ID 1/3 correlation, WMI event analysis, and RPC endpoint mapper traffic on port 135. Use when investigating suspicious mmc.exe/dllhost.exe child processes, building T1021.003 detections, or auditing DCOM exposure during purple-team exercises.","domain":"cybersecurity","path":"skills/hunting-for-dcom-lateral-movement"},{"name":"hunting-for-dcsync-attacks","description":"Detect DCSync attacks (MITRE ATT&CK T1003.006) by analyzing Windows Event ID 4662 (AccessMask 0x100) for DS-Replication-Get-Changes and DS-Replication-Get-Changes-All requests issued by non-domain-controller accounts. Use when hunting for DCSync credential theft, after detecting Mimikatz-class tooling, or during incident response and purple-team exercises involving Active Directory replication abuse.","domain":"cybersecurity","path":"skills/hunting-for-dcsync-attacks"},{"name":"hunting-for-defense-evasion-via-timestomping","description":"Detect NTFS timestamp manipulation (MITRE T1070.006) by comparing $STANDARD_INFORMATION vs $FILE_NAME timestamps in the MFT. Uses analyzeMFT and Python to identify files with anomalous temporal patterns indicating anti-forensic timestomping activity.","domain":"cybersecurity","path":"skills/hunting-for-defense-evasion-via-timestomping"},{"name":"hunting-for-dns-based-persistence","description":"Hunts for DNS-based persistence mechanisms such as DNS hijacking, dangling CNAME records enabling subdomain takeover, wildcard DNS abuse, and unauthorized zone or NS delegation changes, using passive DNS history (SecurityTrails API), Route53/Azure DNS/Cloudflare audit logs, and zone transfer analysis. Use when investigating suspected DNS hijacking or subdomain takeover, or when threat hunting for DNS record tampering that persists across credential rotations and endpoint reimaging.","domain":"cybersecurity","path":"skills/hunting-for-dns-based-persistence"},{"name":"hunting-for-dns-tunneling-with-zeek","description":"Detects DNS tunneling and covert-channel data exfiltration by analyzing Zeek dns.log for high-entropy subdomain queries, excessive query volume, abnormally long query lengths, and unusual DNS record types (TXT/NULL/CNAME). Use when hunting for DNS-based data exfiltration or C2 covert channels in network traffic, or when triaging suspicious DNS query volume/patterns surfaced by Zeek logs.","domain":"cybersecurity","path":"skills/hunting-for-dns-tunneling-with-zeek"},{"name":"hunting-for-domain-fronting-c2-traffic","description":"Detects domain fronting C2 traffic by analyzing SNI-vs-HTTP-Host-header mismatches in proxy logs and inspecting TLS certificate discrepancies with pyOpenSSL. Use when hunting for command-and-control traffic hidden behind legitimate CDN domains, or when investigating proxy/TLS logs for signs of domain fronting evasion.","domain":"cybersecurity","path":"skills/hunting-for-domain-fronting-c2-traffic"},{"name":"hunting-for-lateral-movement-via-wmi","description":"Detects WMI-based lateral movement (e.g. wmic process call create, Win32_Process.Create()) by analyzing Windows Event ID 4688 and Sysmon Event ID 1 for WmiPrvSE.exe spawning suspicious child processes like cmd.exe or powershell.exe, plus WMI-Activity/Operational events 5857/5860/5861 for subscription-based persistence. Use when hunting for remote code execution via WMI or investigating lateral movement across Windows hosts.","domain":"cybersecurity","path":"skills/hunting-for-lateral-movement-via-wmi"},{"name":"hunting-for-living-off-the-cloud-techniques","description":"Hunts for adversary abuse of legitimate cloud services (Azure, AWS, GCP, and SaaS platforms) for command-and-control, data staging, and exfiltration, i.e. \"living off the cloud\" tradecraft that blends in with normal cloud API and service activity. Use when threat hunting for cloud-native C2 channels, abnormal use of storage/SaaS services for data staging, or exfiltration hidden in legitimate cloud traffic.","domain":"cybersecurity","path":"skills/hunting-for-living-off-the-cloud-techniques"},{"name":"hunting-for-living-off-the-land-binaries","description":"Proactively hunts for adversary abuse of legitimate, signed system binaries (LOLBins) used to execute malicious payloads, download files, or proxy execution while evading application allowlisting and defense-evasion controls. Use when building LOLBins detection rules for EDR/SIEM or when threat hunting for defense-evasion activity involving trusted system binaries.","domain":"cybersecurity","path":"skills/hunting-for-living-off-the-land-binaries"},{"name":"hunting-for-lolbins-execution-in-endpoint-logs","description":"Hunts for LOLBins (Living Off the Land Binaries) abuse, mapped to MITRE T1218, by analyzing endpoint process-creation logs for suspicious execution patterns of legitimate Windows system binaries used for malicious purposes. Use when reviewing endpoint process telemetry for LOLBins-based defense evasion or building detections for signed-binary proxy execution.","domain":"cybersecurity","path":"skills/hunting-for-lolbins-execution-in-endpoint-logs"},{"name":"hunting-for-ntlm-relay-attacks","description":"Detects NTLM relay attacks (MITRE T1557.001) by analyzing Windows Event ID 4624 logon type 3 with NTLMSSP authentication, flagging IP-to-hostname mismatches, Responder/LLMNR poisoning signatures, SMB signing status, and anomalous cross-domain authentication patterns. Use when investigating credential-relay activity in Active Directory or building detections for NTLM relay and coercion-based attacks.","domain":"cybersecurity","path":"skills/hunting-for-ntlm-relay-attacks"},{"name":"hunting-for-persistence-mechanisms-in-windows","description":"Systematically hunts for adversary persistence mechanisms across Windows endpoints, covering registry Run/RunOnce keys, services, startup folders, scheduled tasks, and WMI event subscriptions. Use when performing a broad persistence sweep during incident response or building SIEM detections that cover the full range of Windows persistence techniques (MITRE T1547).","domain":"cybersecurity","path":"skills/hunting-for-persistence-mechanisms-in-windows"},{"name":"hunting-for-persistence-via-wmi-subscriptions","description":"Hunts for adversary persistence via WMI event subscriptions (MITRE T1546.003) by monitoring the creation of WMI event filters, consumers, and filter-to-consumer bindings that trigger malicious code execution on system events. Use when investigating fileless, trigger-based persistence on Windows hosts or auditing WMI repository contents for malicious event subscriptions.","domain":"cybersecurity","path":"skills/hunting-for-persistence-via-wmi-subscriptions"},{"name":"hunting-for-process-injection-techniques","description":"Detects process injection techniques (MITRE T1055) \u2014 including CreateRemoteThread injection, process hollowing, and DLL injection \u2014 by analyzing Sysmon Event IDs 8 (CreateRemoteThread) and 10 (ProcessAccess) alongside EDR process telemetry. Use when hunting for in-memory code injection or defense evasion via legitimate process abuse on Windows endpoints.","domain":"cybersecurity","path":"skills/hunting-for-process-injection-techniques"},{"name":"hunting-for-registry-persistence-mechanisms","description":"Hunts for registry-based persistence mechanisms (MITRE T1547) in Windows environments, including Run/RunOnce keys, Winlogon Shell/Userinit modifications, Image File Execution Options (IFEO) debugger injection, and COM hijacking via CLSID overrides. Use when auditing the registry for persistence artifacts or building detections for registry-based malware autostart techniques.","domain":"cybersecurity","path":"skills/hunting-for-registry-persistence-mechanisms"},{"name":"hunting-for-registry-run-key-persistence","description":"Detect MITRE ATT&CK T1547.001 registry Run key persistence by analyzing Sysmon Event ID 13 logs and registry queries to identify malicious auto-start entries.","domain":"cybersecurity","path":"skills/hunting-for-registry-run-key-persistence"},{"name":"hunting-for-scheduled-task-persistence","description":"Runs a hypothesis-driven threat hunt for Windows Scheduled Task persistence (T1053), guiding SIEM/EDR queries against task creation events (e.g. Event ID 4698), suspicious task actions, and unusual scheduling patterns. Use when hunting for scheduled-task persistence, after threat intel flags related campaigns, during incident response, or when alerts fire on schtasks/at.exe activity.","domain":"cybersecurity","path":"skills/hunting-for-scheduled-task-persistence"},{"name":"hunting-for-shadow-copy-deletion","description":"Runs a hypothesis-driven threat hunt for Volume Shadow Copy deletion (T1490) by querying SIEM/EDR telemetry for vssadmin, wmic shadowcopy, and PowerShell shadow-copy-deletion commands. Use when hunting for ransomware preparation or anti-forensics activity, after threat intel flags active campaigns, or when alerts trigger on shadow-copy deletion commands.","domain":"cybersecurity","path":"skills/hunting-for-shadow-copy-deletion"},{"name":"hunting-for-spearphishing-indicators","description":"Hunt for spearphishing campaign indicators across email logs, endpoint telemetry, and network data to detect targeted email attacks.","domain":"cybersecurity","path":"skills/hunting-for-spearphishing-indicators"},{"name":"hunting-for-startup-folder-persistence","description":"Detects T1547.001 startup folder persistence by monitoring Windows startup directories for suspicious file creation, cross-referencing Autoruns entries, and running a Python watchdog script for real-time filesystem monitoring. Use when hunting for malware or implants that survive reboot via startup-folder placement, or when validating autoruns/EDR findings against known-good startup baselines.","domain":"cybersecurity","path":"skills/hunting-for-startup-folder-persistence"},{"name":"hunting-for-supply-chain-compromise","description":"Runs a hypothesis-driven threat hunt for supply-chain compromise (T1195) by querying SIEM/EDR logs for trojanized software updates, compromised dependencies, unauthorized code modifications, and tampered build artifacts. Use when hunting after threat intel flags a compromised vendor/dependency, scoping a build-pipeline compromise, or reviewing update/build integrity.","domain":"cybersecurity","path":"skills/hunting-for-supply-chain-compromise"},{"name":"hunting-for-suspicious-scheduled-tasks","description":"Hunts for adversary persistence and execution via Windows scheduled tasks (T1053.005) by analyzing Security Event ID 4698 task-creation events, suspicious task properties, and unusual execution patterns from schtasks.exe/at.exe. Use after detecting schtasks or at.exe in process creation logs, during incident response to enumerate persistence on compromised hosts, or when Event ID 4698 fires for an unusual task.","domain":"cybersecurity","path":"skills/hunting-for-suspicious-scheduled-tasks"},{"name":"hunting-for-t1098-account-manipulation","description":"Hunts for MITRE ATT&CK T1098 account manipulation - shadow admin creation, SID history injection, group membership changes, and credential modifications - by analyzing Windows Security Event Log IDs 4738, 4728, 4732, 4756, 4670, and 5136. Use when investigating suspected privilege persistence in Active Directory, after detecting anomalous group/credential changes, or during incident response to trace account tampering.","domain":"cybersecurity","path":"skills/hunting-for-t1098-account-manipulation"},{"name":"hunting-for-unusual-network-connections","description":"Runs a hypothesis-driven threat hunt for command-and-control activity (T1071) by querying SIEM/EDR network telemetry for anomalous outbound traffic, rare destinations, non-standard ports, and unusual connection frequencies from endpoints. Use when hunting for beaconing/C2 traffic, after threat intel flags suspicious infrastructure, or when alerts fire on anomalous connections.","domain":"cybersecurity","path":"skills/hunting-for-unusual-network-connections"},{"name":"hunting-for-unusual-service-installations","description":"Detects suspicious Windows service installations (MITRE ATT&CK T1543.003) by parsing System event log Event ID 7045, analyzing service binary paths, and flagging indicators of persistence mechanisms via Sysmon/EDR telemetry. Use when hunting for new-service persistence after a suspected compromise, when Event ID 7045 fires for an unfamiliar service, or during incident response to enumerate service-based persistence on Windows hosts.","domain":"cybersecurity","path":"skills/hunting-for-unusual-service-installations"},{"name":"hunting-for-webshell-activity","description":"Runs a hypothesis-driven threat hunt for web shell deployment (T1505.003) on internet-facing servers by analyzing file creation in web directories, suspicious child-process spawning from web server processes, and anomalous HTTP request patterns. Use when hunting for web shells after a public-facing app compromise, when EDR/SIEM alerts fire on webserver process anomalies, or during incident response on internet-facing infrastructure.","domain":"cybersecurity","path":"skills/hunting-for-webshell-activity"},{"name":"hunting-saas-sso-token-abuse","description":"Hunts for stolen-session and OAuth/PRT token replay (T1550.001) by correlating Microsoft Entra ID SigninLogs SessionId/UniqueTokenIdentifier fields and Okta System Log sso/session events to spot impossible travel, refresh-token reuse, and token use from anomalous ASNs. Use when hunting MFA-bypass via stolen cookies/tokens, investigating impossible-travel alerts, or scoping SaaS lateral movement after phishing.","domain":"cybersecurity","path":"skills/hunting-saas-sso-token-abuse"},{"name":"implementing-aes-encryption-for-data-at-rest","description":"Guides implementing AES-256 encryption in GCM mode (FIPS 197) for files and data stores at rest, covering key derivation, IV/nonce management, and authenticated encryption. Use when deploying or configuring encryption for data at rest, establishing controls to meet compliance requirements, or reviewing an implementation during a security assessment.","domain":"cybersecurity","path":"skills/implementing-aes-encryption-for-data-at-rest"},{"name":"implementing-alert-fatigue-reduction","description":"Implements strategies to reduce SOC alert fatigue by tuning detection rules, consolidating duplicate alerts, implementing risk-based alerting, and measuring alert quality metrics to maintain analyst effectiveness and prevent critical alert dismissal. Use when SOC teams face overwhelming alert volumes, high false positive rates, or declining analyst performance.","domain":"cybersecurity","path":"skills/implementing-alert-fatigue-reduction"},{"name":"implementing-anti-phishing-training-program","description":"Guides designing, deploying, and measuring an anti-phishing security awareness program - baseline phishing simulations, interactive training modules, just-in-time learning, and metric tracking - using platforms like KnowBe4, Proofpoint Security Awareness, or Cofense. Use when building or maturing a phishing awareness program, establishing training controls for compliance, or measuring phishing susceptibility and reporting rates over time.","domain":"cybersecurity","path":"skills/implementing-anti-phishing-training-program"},{"name":"implementing-anti-ransomware-group-policy","description":"Configures Windows Group Policy Objects to block ransomware execution and lateral spread, covering AppLocker rules, Software Restriction Policies, Controlled Folder Access, attack surface reduction rules, and network protection settings. Use when hardening Windows endpoints against ransomware via GPO, configuring AppLocker or Controlled Folder Access, or building endpoint protection policies through Group Policy.","domain":"cybersecurity","path":"skills/implementing-anti-ransomware-group-policy"},{"name":"implementing-api-abuse-detection-with-rate-limiting","description":"Implements API abuse detection using token bucket, sliding window, and fixed window rate-limiting algorithms backed by Redis, including adaptive limits that tighten during detected attacks and relax during normal traffic. Use when defending APIs against DDoS, brute force login attempts, credential stuffing, or scraping abuse and you need to design or tune rate-limiting logic.","domain":"cybersecurity","path":"skills/implementing-api-abuse-detection-with-rate-limiting"},{"name":"implementing-api-gateway-security-controls","description":"Configures API gateways such as Kong, AWS API Gateway, Azure APIM, or Apigee as a centralized security enforcement point, covering authentication enforcement, rate limiting and throttling, request validation, IP allowlisting, TLS termination, and threat protection. Use when securing API traffic at the gateway layer, setting up gateway-level authentication and quota management, or centralizing API protection before requests reach backend services.","domain":"cybersecurity","path":"skills/implementing-api-gateway-security-controls"},{"name":"implementing-api-key-security-controls","description":"Implements secure API key generation with sufficient entropy, server-side hashing (SHA-256/bcrypt) instead of plaintext storage, per-key scoping to endpoints/IPs/rate limits, zero-downtime rotation, and automated leak monitoring across GitHub repos, logs, and client-side code. Use when designing API key formats, building key rotation or revocation workflows, or protecting server-to-server API credentials from leakage, brute force, and abuse.","domain":"cybersecurity","path":"skills/implementing-api-key-security-controls"},{"name":"implementing-api-rate-limiting-and-throttling","description":"Implements API rate limiting and throttling with token bucket, sliding window, and fixed window algorithms, configuring per-user, per-IP, and per-endpoint limits via Redis-backed counters, API gateway plugins, or middleware, and returning proper HTTP 429 responses with Retry-After headers. Use when setting up request quota management or preventing brute force, credential stuffing, and resource exhaustion attacks against APIs.","domain":"cybersecurity","path":"skills/implementing-api-rate-limiting-and-throttling"},{"name":"implementing-api-schema-validation-security","description":"Implements API schema validation using OpenAPI Specification and JSON Schema documents, enforced both at the API gateway (runtime) and during development (shift-left), to lock down request/response contracts and reject unknown properties. Use when preventing injection attacks (SQLi, XSS, XXE), blocking mass assignment, or stopping data leakage through unvalidated API responses.","domain":"cybersecurity","path":"skills/implementing-api-schema-validation-security"},{"name":"implementing-api-security-posture-management","description":"Implements API Security Posture Management (API-SPM) to continuously discover, classify, and risk-score APIs -- including internal, external, partner, and shadow endpoints -- while aggregating findings from DAST, SAST, SCA, and runtime monitoring into a unified risk view. Use when building continuous API inventory and risk-scoring, detecting configuration drift or policy violations, or unifying API risk visibility across an organization.","domain":"cybersecurity","path":"skills/implementing-api-security-posture-management"},{"name":"implementing-api-security-testing-with-42crunch","description":"Implements API security testing on the 42Crunch platform, combining API Audit for static analysis of OpenAPI definitions, API Conformance Scan for dynamic vulnerability testing, and API Protect for runtime threat prevention, integrated into CI/CD pipelines and IDEs. Use when shift-left testing APIs for OWASP API Security Top 10 vulnerabilities or setting up 42Crunch audit and conformance scanning.","domain":"cybersecurity","path":"skills/implementing-api-security-testing-with-42crunch"},{"name":"implementing-api-threat-protection-with-apigee","description":"Implements API threat protection using Google Apigee reverse-proxy policies, including JSON/XML threat protection, OAuth 2.0 enforcement, SpikeArrest rate limiting, regex-based threat detection, and Advanced API Security for detecting malicious clients. Use when shielding APIs proxied through Apigee against OWASP API Security Top 10 threats and malicious client abuse patterns.","domain":"cybersecurity","path":"skills/implementing-api-threat-protection-with-apigee"},{"name":"implementing-application-whitelisting-with-applocker","description":"Implements application whitelisting using Windows AppLocker to restrict unauthorized software execution on endpoints, reducing attack surface from malware, unauthorized tools, and shadow IT. Use when enforcing application control policies, meeting compliance requirements for software restriction, or preventing execution of unsigned or untrusted binaries. Activates for requests involving AppLocker, application whitelisting, software restriction, or executable control.","domain":"cybersecurity","path":"skills/implementing-application-whitelisting-with-applocker"},{"name":"implementing-aqua-security-for-container-scanning","description":"Deploy Aqua Security's Trivy scanner to detect vulnerabilities, misconfigurations, secrets, and license issues in container images across CI/CD pipelines and registries.","domain":"cybersecurity","path":"skills/implementing-aqua-security-for-container-scanning"},{"name":"implementing-attack-path-analysis-with-xm-cyber","description":"Deploys XM Cyber's continuous exposure management platform to build attack graphs that chain vulnerabilities, misconfigurations, identity risks, and credential weaknesses toward critical assets, identifying the small fraction of exposures sitting on converging \"choke points\". Use when mapping attack paths across an environment or prioritizing remediation within a continuous threat exposure management (CTEM) program.","domain":"cybersecurity","path":"skills/implementing-attack-path-analysis-with-xm-cyber"},{"name":"implementing-attack-surface-management","description":"Implements external attack surface management (EASM) using Shodan, Censys, and ProjectDiscovery tools (subfinder, httpx, nuclei) for asset discovery, subdomain enumeration, service fingerprinting, and exposure scoring. Includes a weighted risk scoring algorithm based on OWASP attack surface analysis methodology and the Relative Attack Surface Quotient (RSQ). Use when building continuous ASM programs or performing external reconnaissance for security assessments.","domain":"cybersecurity","path":"skills/implementing-attack-surface-management"},{"name":"implementing-aws-config-rules-for-compliance","description":"Implements AWS Config managed and custom rules for continuous compliance monitoring of AWS resources aligned to CIS and PCI DSS, configuring automatic remediation with SSM Automation and aggregating compliance data across accounts. Use when establishing continuous compliance monitoring, detecting configuration drift, or building a multi-account compliance dashboard via AWS Organizations.","domain":"cybersecurity","path":"skills/implementing-aws-config-rules-for-compliance"},{"name":"implementing-aws-iam-permission-boundaries","description":"Configures AWS IAM permission boundaries that cap the maximum permissions an identity-based policy can grant to a user or role, so effective permissions are the intersection of the identity policy and the boundary even if the policy grants AdministratorAccess. Use when letting security teams delegate IAM role and policy creation to developers for self-service while enforcing least-privilege ceilings and preventing privilege escalation.","domain":"cybersecurity","path":"skills/implementing-aws-iam-permission-boundaries"},{"name":"implementing-aws-macie-for-data-classification","description":"Enable and configure Amazon Macie via AWS CLI/Terraform to discover, classify, and protect sensitive data (PII, financial data, credentials) in S3 using ML and pattern matching, including discovery jobs, custom data identifiers, allow lists, and EventBridge-based remediation. Use when setting up S3 data classification, cloud DLP, or auditing S3 for unprotected sensitive data.","domain":"cybersecurity","path":"skills/implementing-aws-macie-for-data-classification"},{"name":"implementing-aws-nitro-enclave-security","description":"Build AWS Nitro Enclave confidential computing environments using nitro-cli to create enclave images, configure attestation-aware KMS policies with PCR condition keys, validate attestation documents against the Nitro PKI root, and set up vsock/kmstool-enclave-cli pipelines for processing PII, keys, and health records. Use for Nitro Enclave setup, attestation validation, or scoping KMS to an enclave image hash.","domain":"cybersecurity","path":"skills/implementing-aws-nitro-enclave-security"},{"name":"implementing-aws-security-hub","description":"Deploy AWS Security Hub as a centralized CSPM platform, backed by AWS Config, aggregating findings from GuardDuty, Inspector, Macie, and third-party tools; enable CIS Foundations, PCI-DSS, and NIST standards; automate remediation via EventBridge/Systems Manager; and produce Audit Manager evidence. Use for a centralized findings dashboard or compliance audit evidence; not for threat detection or container scanning.","domain":"cybersecurity","path":"skills/implementing-aws-security-hub"},{"name":"implementing-aws-security-hub-compliance","description":"Deploy AWS Security Hub, backed by AWS Config, to aggregate findings from GuardDuty, Inspector, Macie, Firewall Manager, and Prowler across multi-account AWS Organizations, enable standards like CIS AWS Foundations and PCI DSS, and automate remediation via EventBridge and Lambda. Use for centralizing AWS compliance monitoring or executive compliance dashboards; not for threat detection (GuardDuty) or data classification (Macie).","domain":"cybersecurity","path":"skills/implementing-aws-security-hub-compliance"},{"name":"implementing-azure-ad-privileged-identity-management","description":"Configure Microsoft Entra Privileged Identity Management (PIM) to convert standing privileged assignments into eligible, time-bound roles requiring justification, MFA, and approval, covering Entra roles, Azure resource roles, and PIM for Groups, plus access reviews. Use for role-assignment audits, just-in-time admin activation, or Zero Trust identity governance in Entra/Azure AD.","domain":"cybersecurity","path":"skills/implementing-azure-ad-privileged-identity-management"},{"name":"implementing-azure-defender-for-cloud","description":"Enable Microsoft Defender for Cloud (CSPM + CWPP) across VMs, containers, SQL, storage, and Key Vault, using Azure Policy for evaluation, Log Analytics for telemetry, Azure Arc for hybrid coverage, and Logic Apps for automated response. Use for Azure security monitoring, adaptive remediation, or regulatory compliance assessment; not for non-Azure-only protection or identity defense (Defender for Identity).","domain":"cybersecurity","path":"skills/implementing-azure-defender-for-cloud"},{"name":"implementing-beyondcorp-zero-trust-access-model","description":"Implement Google's BeyondCorp zero trust access model using Cloud IAP, Access Context Manager, Endpoint Verification, Chrome Enterprise Premium, and BeyondCorp Enterprise Connectors to enforce identity- and device-aware access for VPN-less application access. Use for replacing VPN, enforcing device posture checks, or securing remote/hybrid access to GCP-hosted or on-prem apps; not for raw network-level protocols.","domain":"cybersecurity","path":"skills/implementing-beyondcorp-zero-trust-access-model"},{"name":"implementing-bgp-security-with-rpki","description":"Implement RPKI-based BGP route origin validation by creating Route Origin Authorizations (ROAs) at RIRs (ARIN, RIPE, APNIC, AFRINIC, LACNIC), deploying validator software (Routinator, FORT, OctoRPKI), and configuring RPKI-to-Router protocol and ROV accept/reject policies on Cisco IOS-XE and Juniper Junos routers. Use when hardening BGP against route hijacking or leaks, or when configuring ROV filtering policy on production routers.","domain":"cybersecurity","path":"skills/implementing-bgp-security-with-rpki"},{"name":"implementing-browser-isolation-for-zero-trust","description":"Deploys remote browser isolation (RBI) as a core component of a Zero Trust architecture. Implements isolation policies with URL categorization and risk-based routing, content disarming and reconstruction (CDR) for file sanitization, data loss prevention controls within isolated sessions, and integration with Secure Web Gateway and ZTNA platforms. Based on Cloudflare Browser Isolation, Menlo Security, and Zscaler RBI approaches. Use when hardening web access against zero-day exploits, phishing, credential theft, and browser-based data exfiltration.","domain":"cybersecurity","path":"skills/implementing-browser-isolation-for-zero-trust"},{"name":"implementing-canary-tokens-for-network-intrusion","description":"Deploys DNS, HTTP, and AWS API key canary tokens across network infrastructure to detect unauthorized access and lateral movement. Integrates with webhook alerting (Slack, Teams, email, generic HTTP) for real-time intrusion notifications. Provides automated token generation, placement strategies, and monitoring for enterprise network environments. Use when building deception-based network intrusion detection with Canarytokens.org and Thinkst Canary platforms.","domain":"cybersecurity","path":"skills/implementing-canary-tokens-for-network-intrusion"},{"name":"implementing-cisa-zero-trust-maturity-model","description":"Assess, gap-analyze, and progressively implement the CISA Zero Trust Maturity Model v2.0 across five pillars (Identity, Devices, Networks, Applications & Workloads, Data) and three cross-cutting capabilities (Visibility/Analytics, Automation/Orchestration, Governance), from Traditional through Optimal maturity. Use for a federal/enterprise ZTMM assessment, phased roadmap, or mapping controls to NIST SP 800-207 and EO 14028.","domain":"cybersecurity","path":"skills/implementing-cisa-zero-trust-maturity-model"},{"name":"implementing-cloud-dlp-for-data-protection","description":"Implement cloud DLP using Amazon Macie, Google Cloud DLP API, Microsoft Purview, Azure Information Protection, and Nightfall AI to discover, classify, label, de-identify, and protect sensitive data (PII, PHI, financial data) across cloud storage, databases, and pipelines. Use for GDPR/HIPAA/PCI DSS data-discovery, cloud data governance, or CI/CD DLP scanning; not for endpoint, email, or network-level DLP.","domain":"cybersecurity","path":"skills/implementing-cloud-dlp-for-data-protection"},{"name":"implementing-cloud-security-posture-management","description":"Continuously monitor multi-cloud environments (AWS, Azure, GCP) for misconfigurations, compliance violations, and security risks using Prowler, ScoutSuite, AWS Security Hub, Microsoft Defender for Cloud, and GCP Security Command Center. Use for cross-cloud posture monitoring, CIS/SOC 2/PCI DSS compliance, or drift-detection workflows; not for runtime workload protection or application security testing.","domain":"cybersecurity","path":"skills/implementing-cloud-security-posture-management"},{"name":"implementing-cloud-trail-log-analysis","description":"Implementing AWS CloudTrail log analysis for security monitoring, threat detection, and forensic investigation using Athena, CloudWatch Logs Insights, and SIEM integration to identify unauthorized access, privilege escalation, and suspicious API activity.","domain":"cybersecurity","path":"skills/implementing-cloud-trail-log-analysis"},{"name":"implementing-cloud-vulnerability-posture-management","description":"Implement multi-cloud CSPM to detect cloud-native misconfigurations and vulnerabilities (IAM over-permissions, exposed storage, unencrypted data, missing network controls) using AWS Security Hub, Azure Defender for Cloud, and open-source Prowler and ScoutSuite scans, then aggregate results across clouds. Use when auditing multi-cloud environments for misconfiguration-driven vulnerabilities or building a consolidated cross-cloud posture report.","domain":"cybersecurity","path":"skills/implementing-cloud-vulnerability-posture-management"},{"name":"implementing-cloud-waf-rules","description":"Deploys and tunes Web Application Firewall rules on AWS WAF, Azure WAF, and Cloudflare, covering managed rule sets, custom business-logic rules, rate limiting, bot management, and false-positive reduction. Use when deploying new apps behind a cloud WAF, when pentests reveal injection/XSS flaws, when facing bot or credential-stuffing traffic, or when compliance (e.g. PCI-DSS) mandates a WAF.","domain":"cybersecurity","path":"skills/implementing-cloud-waf-rules"},{"name":"implementing-cloud-workload-protection","description":"Implements cloud workload protection using boto3 and google-cloud APIs for runtime security monitoring, process anomaly detection, and file integrity checking on EC2/GCE instances. Scans for cryptomining, reverse shells, and unauthorized binaries. Use when building runtime security controls for cloud compute workloads.","domain":"cybersecurity","path":"skills/implementing-cloud-workload-protection"},{"name":"implementing-code-signing-for-artifacts","description":"Implements code signing for build artifacts (binaries, packages, containers) using GPG, Sigstore, and platform-specific signing tools, establishing trust chains and verifying signatures in deployment pipelines. Use when establishing artifact integrity checks against supply-chain tampering, proving authenticity to customers, building zero-trust pipelines that reject unsigned artifacts, or meeting SLSA Level 2+ provenance requirements.","domain":"cybersecurity","path":"skills/implementing-code-signing-for-artifacts"},{"name":"implementing-conditional-access-policies-azure-ad","description":"Configures Microsoft Entra ID (Azure AD) Conditional Access policies for zero trust access control, covering signal-based policy design, device compliance requirements, risk-based authentication, named locations, and session controls aligned to NIST SP 1800-35. Use when deploying or hardening conditional access policies, building zero trust security architecture in Entra ID, or preparing for a security assessment of Azure AD access controls.","domain":"cybersecurity","path":"skills/implementing-conditional-access-policies-azure-ad"},{"name":"implementing-conduit-security-for-ot-remote-access","description":"Implements secure conduit architecture for OT remote access under the IEC 62443 zones-and-conduits model, deploying jump servers, MFA gateways, session recording, and approval-based workflows for vendor and engineer access to ICS. Use when replacing direct VPN access into OT networks, securing third-party vendor access to SCADA equipment, or remediating audit findings on uncontrolled OT remote access.","domain":"cybersecurity","path":"skills/implementing-conduit-security-for-ot-remote-access"},{"name":"implementing-container-image-minimal-base-with-distroless","description":"Reduces container attack surface by building application images on Google distroless base images that ship only the application runtime - no shell, package manager, or OS utilities - using multi-stage build patterns plus debugging and scanning techniques adapted to distroless. Use when hardening container images, cutting attack surface in a container architecture, or answering an assessment finding about bloated base images. Keywords: distroless, multi-stage build, no shell, nonroot tag, debug image, scratch, attack surface. Do not use for scanning an image for known CVEs - use scanning-docker-images-with-trivy.","domain":"cybersecurity","path":"skills/implementing-container-image-minimal-base-with-distroless"},{"name":"implementing-container-network-policies-with-calico","description":"Uses Calico's own policy CRDs beyond the upstream Kubernetes API - GlobalNetworkPolicy, HostEndpoint, NetworkSet, policy tiers, and DNS-based egress rules - applied and audited with calicoctl. Use when a policy must span namespaces or protect the host itself, when egress has to be expressed by domain name, or when ordering policies into tiers. Keywords: calicoctl, GlobalNetworkPolicy, HostEndpoint, NetworkSet, tier, DNS egress, order. Do not use for portable upstream NetworkPolicy - use implementing-network-policies-for-kubernetes; for installing Calico and writing standard policy with it use implementing-kubernetes-network-policy-with-calico.","domain":"cybersecurity","path":"skills/implementing-container-network-policies-with-calico"},{"name":"implementing-continuous-security-validation-with-bas","description":"Deploys Breach and Attack Simulation (BAS) platforms such as SafeBreach, AttackIQ, Picus, Cymulate, Pentera, or SCYTHE to continuously validate endpoint, network, email-gateway, SIEM, and incident-response effectiveness by safely emulating MITRE ATT&CK techniques. Use when moving beyond point-in-time pentesting to continuous control validation, or standing up automated adversary emulation.","domain":"cybersecurity","path":"skills/implementing-continuous-security-validation-with-bas"},{"name":"implementing-data-loss-prevention-with-microsoft-purview","description":"Implements DLP policies using Microsoft Purview PowerShell cmdlets and the Graph API to protect data across Exchange Online, SharePoint, OneDrive, Teams, endpoints, and Power BI, including sensitivity labels, custom sensitive information types with regex, endpoint DLP rules, and Activity Explorer monitoring. Use when stopping PII/PHI/PCI exfiltration, configuring sensitivity labels, or investigating DLP incidents for policy tuning.","domain":"cybersecurity","path":"skills/implementing-data-loss-prevention-with-microsoft-purview"},{"name":"implementing-ddos-mitigation-with-cloudflare","description":"Configure Cloudflare DDoS protection with managed rulesets, rate limiting, WAF rules, Bot Management, and origin protection to mitigate volumetric, protocol, and application-layer attacks.","domain":"cybersecurity","path":"skills/implementing-ddos-mitigation-with-cloudflare"},{"name":"implementing-deception-based-detection-with-canarytoken","description":"Deploys and monitors Canary Tokens via the Thinkst Canary REST API for deception-based breach detection, programmatically creating web bug, DNS, MS Word document, and AWS API key tokens and generating deception coverage reports from triggered alerts. Use when standing up honeytoken tripwires for early breach detection or building a deception-technology coverage report.","domain":"cybersecurity","path":"skills/implementing-deception-based-detection-with-canarytoken"},{"name":"implementing-delinea-secret-server-for-pam","description":"Implements Delinea Secret Server for privileged access management, covering secret vault configuration, role-based access policies, automated password rotation, session recording, and Active Directory/cloud integration. Use when centralizing privileged credential management, replacing spreadsheet-based secrets, automating password rotation, or meeting PAM compliance (SOX, PCI-DSS, HIPAA, NIST 800-53).","domain":"cybersecurity","path":"skills/implementing-delinea-secret-server-for-pam"},{"name":"implementing-device-posture-assessment-in-zero-trust","description":"Implements device posture assessment as a zero trust access control by integrating endpoint health signals from CrowdStrike ZTA, Microsoft Intune, and Jamf into conditional access policies that enforce compliance before granting access. Use when requiring device health checks before app access, wiring posture signals into conditional access, or implementing the CISA Zero Trust device pillar.","domain":"cybersecurity","path":"skills/implementing-device-posture-assessment-in-zero-trust"},{"name":"implementing-devsecops-security-scanning","description":"Integrates SAST, DAST, and SCA into CI/CD pipelines using Semgrep for SAST, Trivy for SCA and container scanning, OWASP ZAP for DAST, and Gitleaks for secrets detection. Use when setting up automated security scanning in CI/CD, shifting security left, meeting compliance mandates (SOC 2, PCI-DSS, ISO 27001), or gating deployments on critical vulnerabilities.","domain":"cybersecurity","path":"skills/implementing-devsecops-security-scanning"},{"name":"implementing-diamond-model-analysis","description":"The Diamond Model of Intrusion Analysis provides a structured framework for analyzing cyber intrusions by examining four core features - Adversary, Capability, Infrastructure, and Victim. This skill covers implementing the Diamond Model programmatically to classify and correlate intrusion events, build activity threads, and generate pivot-ready intelligence.","domain":"cybersecurity","path":"skills/implementing-diamond-model-analysis"},{"name":"implementing-digital-signatures-with-ed25519","description":"Implements digital signatures using the Ed25519 algorithm (Curve25519), covering key-pair generation, signing, signature verification, and security tradeoffs versus RSA and ECDSA. Use when adding message or artifact signing and authentication-integrity checks to a system, or when advising on Ed25519 key properties and validation criteria.","domain":"cybersecurity","path":"skills/implementing-digital-signatures-with-ed25519"},{"name":"implementing-disk-encryption-with-bitlocker","description":"Implements full disk encryption using Microsoft BitLocker on Windows endpoints to protect data at rest from unauthorized access in case of device loss or theft. Use when deploying encryption for compliance requirements, securing mobile workstations, or implementing data protection controls across the enterprise. Activates for requests involving BitLocker encryption, disk encryption, TPM configuration, or data-at-rest protection.","domain":"cybersecurity","path":"skills/implementing-disk-encryption-with-bitlocker"},{"name":"implementing-dmarc-dkim-spf-email-security","description":"Configures SPF, DKIM, and DMARC DNS TXT records to authenticate outbound email, prevent domain spoofing, and enforce a rejection/quarantine policy on unauthenticated mail, including auditing a domain's current DNS state. Use when hardening a domain's email security posture or defending against phishing and spoofing attacks.","domain":"cybersecurity","path":"skills/implementing-dmarc-dkim-spf-email-security"},{"name":"implementing-dragos-platform-for-ot-monitoring","description":"Deploys and configures Dragos Platform sensors and detection analytics for OT/ICS network monitoring, using industrial protocol parsers and threat-intel packs to detect groups like VOLTZITE, CHERNOVITE, and KAMACITE. Use when standing up OT-specific network detection and response or an OT SOC, or integrating OT monitoring into an enterprise SIEM; not for IT-only or Claroty/Nozomi environments.","domain":"cybersecurity","path":"skills/implementing-dragos-platform-for-ot-monitoring"},{"name":"implementing-ebpf-security-monitoring","description":"Implements eBPF-based security monitoring using Cilium Tetragon for real-time process execution tracking, network connection observability, file access auditing, and runtime enforcement. Covers TracingPolicy CRD authoring with kprobe/tracepoint hooks, in-kernel filtering via matchArgs/matchBinaries selectors, JSON event export, and integration with SIEM pipelines. Use when building kernel-level runtime security observability for Linux hosts or Kubernetes clusters.","domain":"cybersecurity","path":"skills/implementing-ebpf-security-monitoring"},{"name":"implementing-email-sandboxing-with-proofpoint","description":"Email sandboxing detonates suspicious attachments and URLs in isolated environments to detect zero-day malware and evasive phishing payloads. Proofpoint Targeted Attack Protection (TAP) is an industry","domain":"cybersecurity","path":"skills/implementing-email-sandboxing-with-proofpoint"},{"name":"implementing-end-to-end-encryption-for-messaging","description":"Implements a simplified Signal Protocol-style end-to-end encryption scheme for messaging, covering key exchange, forward secrecy, and the core cryptographic components so no server or intermediary can decrypt messages. Use when designing or building E2EE messaging, or evaluating forward-secrecy and key-management tradeoffs for a messaging system.","domain":"cybersecurity","path":"skills/implementing-end-to-end-encryption-for-messaging"},{"name":"implementing-endpoint-detection-with-wazuh","description":"Deploys and configures Wazuh SIEM/XDR for endpoint detection, covering agent authentication and management, custom decoder and rule XML creation, alert querying via the Wazuh REST API, rule testing with wazuh-logtest, and automated active-response actions. Use when setting up endpoint detection and response, writing or testing custom Wazuh rules, or querying and triaging Wazuh alerts.","domain":"cybersecurity","path":"skills/implementing-endpoint-detection-with-wazuh"},{"name":"implementing-endpoint-dlp-controls","description":"Implements endpoint Data Loss Prevention (DLP) controls to detect and prevent sensitive data exfiltration through email, USB, cloud storage, and printing. Use when deploying DLP agents, creating content inspection policies, or preventing unauthorized data movement from endpoints. Activates for requests involving DLP, data exfiltration prevention, content inspection, or sensitive data protection on endpoints.","domain":"cybersecurity","path":"skills/implementing-endpoint-dlp-controls"},{"name":"implementing-envelope-encryption-with-aws-kms","description":"Implements envelope encryption with AWS KMS, encrypting data locally with a data encryption key (DEK) and protecting that DEK with a KMS-managed key (KEK), covering the encrypt/decrypt flow, KMS key types, and security validation criteria. Use when designing key management for encrypting large or numerous data objects on AWS, or when reducing direct KMS API call volume.","domain":"cybersecurity","path":"skills/implementing-envelope-encryption-with-aws-kms"},{"name":"implementing-epss-score-for-vulnerability-prioritization","description":"Queries FIRST's Exploit Prediction Scoring System (EPSS) API to fetch exploitation-probability and percentile scores for CVEs, then uses those scores to prioritize vulnerability remediation. Use when triaging or ranking a vulnerability backlog by real-world 30-day exploitation likelihood rather than CVSS severity alone.","domain":"cybersecurity","path":"skills/implementing-epss-score-for-vulnerability-prioritization"},{"name":"implementing-file-integrity-monitoring-with-aide","description":"Configures AIDE (Advanced Intrusion Detection Environment) for file integrity monitoring on Linux, covering baseline database creation, scheduled integrity checks via cron, change detection, and alerting on unauthorized modifications. Use when setting up host-based file integrity monitoring, detecting unauthorized file changes, or meeting compliance requirements for FIM on Linux systems.","domain":"cybersecurity","path":"skills/implementing-file-integrity-monitoring-with-aide"},{"name":"implementing-fuzz-testing-in-cicd-with-aflplusplus","description":"Integrates AFL++ coverage-guided fuzzing into CI/CD pipelines, covering harness construction, AFL++/AddressSanitizer/CmpLog instrumentation builds, and persistent-mode fuzzing to discover memory-corruption and input-handling vulnerabilities in C/C++ code. Use when adding automated fuzz testing to a build pipeline or hunting for memory-safety bugs in native/compiled applications.","domain":"cybersecurity","path":"skills/implementing-fuzz-testing-in-cicd-with-aflplusplus"},{"name":"implementing-gcp-binary-authorization","description":"Implements GCP Binary Authorization end to end, including creating KMS-backed attestors, Container Analysis notes, deploy-time policies, and signing image attestations, so that only trusted, verified images deploy to GKE and Cloud Run. Use when enforcing container supply-chain integrity or deploy-time attestation checks on GCP.","domain":"cybersecurity","path":"skills/implementing-gcp-binary-authorization"},{"name":"implementing-gcp-organization-policy-constraints","description":"Implements GCP Organization Policy constraints via gcloud and Terraform, such as restricting external IPs, resource locations, default service accounts, and service account keys, plus dry-run testing of policy impact before enforcement. Use when enforcing security guardrails across an org's resource hierarchy, or hardening GCP config at the org, folder, or project level.","domain":"cybersecurity","path":"skills/implementing-gcp-organization-policy-constraints"},{"name":"implementing-gcp-vpc-firewall-rules","description":"Implements and audits GCP VPC firewall rules using gcloud, covering auditing overly permissive rules, creating restrictive ingress/egress rules, hierarchical firewall policies, and monitoring rule effectiveness with VPC Flow Logs. Use when deploying GCP workloads needing network access controls, auditing firewall configs, or responding to Security Command Center findings; not for Cloud Armor or DNS-based filtering.","domain":"cybersecurity","path":"skills/implementing-gcp-vpc-firewall-rules"},{"name":"implementing-gdpr-data-protection-controls","description":"Implements GDPR (EU 2016/679) technical and organizational measures \u2014 privacy by design/default, DPIAs, data subject rights management, 72-hour breach notification, and cross-border transfer mechanisms (SCCs, BCRs, adequacy). Use when designing or auditing GDPR controls, building a DPIA, handling data subject access/erasure requests, or assessing cross-border data transfers.","domain":"cybersecurity","path":"skills/implementing-gdpr-data-protection-controls"},{"name":"implementing-gdpr-data-subject-access-request","description":"Automates GDPR Data Subject Access Request (DSAR) workflows including identity verification, PII discovery across databases and files using regex and NER, data mapping, response templating per Article 15 requirements, deadline tracking, and audit logging. Covers ICO/EDPB guidance compliance, exemption handling, and scalable batch processing. Use when building or auditing DSAR response capabilities under GDPR/UK GDPR.","domain":"cybersecurity","path":"skills/implementing-gdpr-data-subject-access-request"},{"name":"implementing-github-advanced-security-for-code-scanning","description":"Configures GitHub Advanced Security (code scanning with CodeQL, secret scanning, dependency review, and Dependabot alerts) to perform automated static analysis and vulnerability detection across repositories at enterprise scale, including custom CodeQL queries and CI workflow integration. Use when setting up or tuning code scanning, rolling out CodeQL across an organization, or shifting SAST left into pull request workflows.","domain":"cybersecurity","path":"skills/implementing-github-advanced-security-for-code-scanning"},{"name":"implementing-google-workspace-admin-security","description":"Hardens a Google Workspace tenant via Admin Console configuration: phishing-resistant MFA enforcement, DLP policies, email authentication (SPF/DKIM/DMARC), OAuth third-party app control, and external sharing restrictions. Use when hardening a Google Workspace or G Suite tenant, enforcing MFA and OAuth app controls, or configuring cloud office security administration.","domain":"cybersecurity","path":"skills/implementing-google-workspace-admin-security"},{"name":"implementing-google-workspace-phishing-protection","description":"Configures Google Workspace advanced phishing and malware protection settings in the Admin Console \u2014 pre-delivery message scanning, attachment protection, spoofing/impersonation detection, and Enhanced Safe Browsing enforcement. Use when hardening Gmail against phishing, spoofing, and malware, or when tuning Workspace email security policies.","domain":"cybersecurity","path":"skills/implementing-google-workspace-phishing-protection"},{"name":"implementing-google-workspace-sso-configuration","description":"Configures SAML 2.0 single sign-on for Google Workspace against a third-party identity provider (Okta, Azure AD/Entra ID, ADFS), with Workspace as the Service Provider, to centralize authentication and enable immediate access revocation. Use when setting up or troubleshooting Google Workspace SSO/SAML federation or migrating from native Google passwords to an external IdP.","domain":"cybersecurity","path":"skills/implementing-google-workspace-sso-configuration"},{"name":"implementing-hardware-security-key-authentication","description":"Builds a FIDO2/WebAuthn relying party server with the python-fido2 library, covering registration and authentication ceremonies, YubiKey enrollment, resident key (discoverable credential/passkey) workflows, and user verification policies. Use when implementing phishing-resistant MFA with hardware security keys, building a WebAuthn relying party, enrolling YubiKeys for a workforce, or migrating password-based authentication to passkeys.","domain":"cybersecurity","path":"skills/implementing-hardware-security-key-authentication"},{"name":"implementing-hashicorp-vault-dynamic-secrets","description":"Configures HashiCorp Vault dynamic secrets engines for database credentials, AWS IAM keys, and PKI certificates, with automatic generation, lease management, and rotation to eliminate static secrets in application configuration. Use when eliminating long-lived static credentials, configuring a Vault secrets engine, generating ephemeral database or cloud credentials, or automating credential rotation for compliance (PCI-DSS, NIST 800-53 IA-5).","domain":"cybersecurity","path":"skills/implementing-hashicorp-vault-dynamic-secrets"},{"name":"implementing-hipaa-security-rule-safeguards","description":"Implement the HIPAA Security Rule (45 CFR Part 164 Subpart C) to protect electronic protected health information (ePHI): conduct the required risk analysis, deploy the administrative, physical, and technical safeguards, handle required vs addressable implementation specifications, execute Business Associate Agreements, and stand up breach-notification readiness. Use when an organization is a HIPAA covered entity or business associate, when protecting ePHI, when preparing for an OCR audit or responding to a breach, when performing a HIPAA Security Risk Analysis, when drafting or reviewing a BAA, or when mapping security controls to the \u00a7164.308/310/312/314/316 safeguards. Notes the 2025 NPRM proposed changes (not yet final). Keywords: HIPAA, HIPAA Security Rule, ePHI, PHI, 45 CFR 164, risk analysis, administrative safeguards, physical safeguards, technical safeguards, addressable, required, Business Associate Agreement, BAA, OCR, breach notification, HITECH, covered entity, business associate.","domain":"cybersecurity","path":"skills/implementing-hipaa-security-rule-safeguards"},{"name":"implementing-honeypot-for-ransomware-detection","description":"Deploys canary files, honeypot shares, and decoy systems to detect ransomware activity at the earliest possible stage. Configures canary tokens embedded in strategic file locations that trigger alerts when ransomware attempts encryption, uses honeypot network shares that mimic high-value targets, and deploys Thinkst Canary appliances for comprehensive deception-based detection. Activates for requests involving ransomware honeypots, canary files, deception technology for ransomware, or early ransomware alerting.","domain":"cybersecurity","path":"skills/implementing-honeypot-for-ransomware-detection"},{"name":"implementing-honeytokens-for-breach-detection","description":"Deploys canary tokens and honeytokens (fake AWS credentials, DNS canaries, document beacons, database records) that trigger alerts when accessed by attackers. Uses the Canarytokens API and custom webhook integrations for breach detection. Use when building deception-based early warning systems for intrusion detection.","domain":"cybersecurity","path":"skills/implementing-honeytokens-for-breach-detection"},{"name":"implementing-ics-firewall-with-tofino","description":"Deploys and configures Tofino industrial firewalls (Belden/Hirschmann) to protect SCADA systems and PLCs, using deep packet inspection of OT protocols (Modbus, EtherNet/IP, OPC, S7comm) to enforce access control between ICS zones. Use when deploying zone-level firewall protection in front of PLCs/RTUs, enforcing IEC 62443 zone/conduit boundaries, or adding compensating controls for unpatchable legacy PLCs.","domain":"cybersecurity","path":"skills/implementing-ics-firewall-with-tofino"},{"name":"implementing-identity-governance-with-sailpoint","description":"Deploys SailPoint IdentityNow or IdentityIQ for identity governance and administration, covering identity lifecycle management, access request workflows, certification campaigns, role mining, separation-of-duties (SOD) policy enforcement, and compliance reporting. Use when standing up or tuning an identity governance program, automating access certifications, or enforcing SOD policies across enterprise IAM.","domain":"cybersecurity","path":"skills/implementing-identity-governance-with-sailpoint"},{"name":"implementing-identity-verification-for-zero-trust","description":"Implements continuous, risk-adaptive identity verification for zero trust using phishing-resistant MFA (FIDO2/WebAuthn), risk-based conditional access, and identity governance aligned with NIST SP 800-207 and the CISA Zero Trust Maturity Model Identity Pillar. Use when designing zero trust identity controls, deploying phishing-resistant MFA, or building conditional access policies based on device posture, behavior, and location.","domain":"cybersecurity","path":"skills/implementing-identity-verification-for-zero-trust"},{"name":"implementing-iec-62443-security-zones","description":"Designs security zones and conduits for industrial control systems (IACS) per IEC 62443-3-2 \u2014 risk-based zone partitioning, Security Level target (SL-T) assignment, conduit controls, and firewall microsegmentation validated against the Purdue Reference Model. Use when designing OT network zones for a new facility, retrofitting zones into a flat OT network, or upgrading VLAN-only segmentation for IEC 62443-3-2 compliance.","domain":"cybersecurity","path":"skills/implementing-iec-62443-security-zones"},{"name":"implementing-image-provenance-verification-with-cosign","description":"Signs and verifies container image provenance with Sigstore Cosign, covering key-based and keyless OIDC signing (Fulcio, Rekor transparency log), SLSA attestations, and enforcing signature verification through Kubernetes admission control. Use when signing images for supply chain security, setting up keyless OIDC signing, attaching attestations, or enforcing a verified-images-only policy at admission. Keywords: Cosign, Sigstore, Fulcio, Rekor, keyless, attestation, cosign verify, admission policy. Do not use for in-toto layout-based pipeline attestation - use implementing-supply-chain-security-with-in-toto.","domain":"cybersecurity","path":"skills/implementing-image-provenance-verification-with-cosign"},{"name":"implementing-immutable-backup-with-restic","description":"Implements ransomware-resistant backups using restic with S3-compatible Object Lock (AWS S3, MinIO, Backblaze B2), automating backup creation, integrity checks via restic check --read-data, retention enforcement, and restore testing. Use when building immutable backup infrastructure, adding a WORM copy to a 3-2-1-1-0 strategy, or automating scheduled backup-verification workflows.","domain":"cybersecurity","path":"skills/implementing-immutable-backup-with-restic"},{"name":"implementing-infrastructure-as-code-security-scanning","description":"Implements automated security scanning for Infrastructure as Code using Checkov, tfsec, and KICS to detect misconfigurations in Terraform, CloudFormation, Kubernetes manifests, and Helm charts, plus policy-based governance and CI/CD integration. Use when validating cloud infrastructure before deployment or blocking insecure changes (public S3 buckets, open security groups) in pull requests.","domain":"cybersecurity","path":"skills/implementing-infrastructure-as-code-security-scanning"},{"name":"implementing-iso-27001-information-security-management","description":"Guides implementation of an ISO/IEC 27001:2022 Information Security Management System (ISMS) end to end: gap analysis and scoping, risk assessment methodology, Annex A control selection, Statement of Applicability (SoA) creation, and continuous improvement. Use when scoping a new ISMS, preparing for ISO 27001 certification or audit, or selecting and documenting Annex A controls for a compliance program.","domain":"cybersecurity","path":"skills/implementing-iso-27001-information-security-management"},{"name":"implementing-just-in-time-access-provisioning","description":"Implements Just-In-Time (JIT) access provisioning to eliminate standing privileges by granting temporary, time-bound access, covering approval workflows, automatic expiration/revocation, and PAM/IGA integration. Use when designing access approval workflows or replacing standing privileged accounts with time-bound, zero-trust-aligned grants.","domain":"cybersecurity","path":"skills/implementing-just-in-time-access-provisioning"},{"name":"implementing-jwt-signing-and-verification","description":"Implements secure JWT (RFC 7519) signing and verification using HMAC-SHA256, RSA-PSS, ES256, and EdDSA, including token expiration, claims validation, and defenses against algorithm-confusion, none-algorithm, and key-injection attacks. Use when adding or hardening JWT-based authentication/authorization, or when auditing token verification code for common JWT vulnerabilities.","domain":"cybersecurity","path":"skills/implementing-jwt-signing-and-verification"},{"name":"implementing-kubernetes-network-policy-with-calico","description":"Installs Calico as the cluster CNI and writes standard Kubernetes NetworkPolicy under it, covering default-deny baselines, policy ordering and precedence, service-account-based selectors, and verifying that policy is genuinely being enforced. Use when adopting Calico as the enforcement CNI, establishing a default-deny baseline, or debugging why a NetworkPolicy is not taking effect under Calico. Keywords: Calico CNI, NetworkPolicy, default deny, policy order, Felix, service account selector. Do not use for Calico-only CRDs such as GlobalNetworkPolicy or DNS egress - use implementing-container-network-policies-with-calico; for CNI-agnostic policy use implementing-network-policies-for-kubernetes.","domain":"cybersecurity","path":"skills/implementing-kubernetes-network-policy-with-calico"},{"name":"implementing-kubernetes-pod-security-standards","description":"Chooses and applies the correct Kubernetes Pod Security Standard (Privileged, Baseline, Restricted) for a workload: what each profile forbids, how to map existing workloads to a profile, which securityContext fields must change, and how to plan a PodSecurityPolicy-to-PSS migration without breaking running pods. Use when deciding which pod security profile a namespace or workload should run under, auditing which workloads would fail Restricted, planning a PSP migration, or mapping pod security posture to a compliance control. Keywords: Pod Security Standards, PSS, Privileged, Baseline, Restricted, securityContext, runAsNonRoot, drop ALL capabilities, seccomp RuntimeDefault, PSP migration. Do not use for configuring the admission controller that enforces these profiles - use implementing-pod-security-admission-controller.","domain":"cybersecurity","path":"skills/implementing-kubernetes-pod-security-standards"},{"name":"implementing-llm-guardrails-for-security","description":"Implements input/output validation guardrails for LLM applications using NVIDIA NeMo Guardrails (Colang), custom Python validators for PII detection, and the Guardrails AI framework, intercepting user inputs (prompt injection, PII, off-topic queries) and model outputs (hallucinations, toxic content, schema compliance). Use when adding safety controls to an LLM app/chatbot/RAG pipeline or validating outputs conform to expected schemas.","domain":"cybersecurity","path":"skills/implementing-llm-guardrails-for-security"},{"name":"implementing-log-forwarding-with-fluentd","description":"Configures Fluent Bit as an endpoint log forwarder and Fluentd as the central aggregator for centralized log collection, routing, filtering, and enrichment, covering input plugins for syslog/file-tailing/application logs and output routing to Elasticsearch, S3, and Splunk. Use when setting up centralized log aggregation across distributed infrastructure or generating Fluent Bit/Fluentd configuration files for a new log pipeline.","domain":"cybersecurity","path":"skills/implementing-log-forwarding-with-fluentd"},{"name":"implementing-log-integrity-with-blockchain","description":"Builds an append-only log integrity chain using SHA-256 hash chaining, where each entry incorporates the previous entry's hash so tampering invalidates all subsequent hashes; covers log ingestion (syslog/JSON/plain text), chain verification, pinpoint tamper detection, and checkpoint anchoring to external timestamping services. Use for tamper-evident log storage for compliance or forensics, or to verify whether log entries were altered.","domain":"cybersecurity","path":"skills/implementing-log-integrity-with-blockchain"},{"name":"implementing-memory-protection-with-dep-aslr","description":"Implements memory protection mechanisms including DEP (Data Execution Prevention), ASLR (Address Space Layout Randomization), CFG (Control Flow Guard), and other exploit mitigations to prevent memory corruption attacks. Use when hardening endpoints against buffer overflow exploits, ROP chains, and code injection. Activates for requests involving memory protection, exploit mitigation, DEP, ASLR, or CFG configuration.","domain":"cybersecurity","path":"skills/implementing-memory-protection-with-dep-aslr"},{"name":"implementing-microsegmentation-with-guardicore","description":"Implements microsegmentation with Akamai Guardicore Segmentation to map application dependencies, visualize east-west traffic flows, and create granular, least-privilege network policies across VMs, containers, bare metal, and cloud. Use when blocking lateral movement in a data center or when PCI DSS/HIPAA compliance requires validated network segmentation.","domain":"cybersecurity","path":"skills/implementing-microsegmentation-with-guardicore"},{"name":"implementing-mimecast-targeted-attack-protection","description":"Deploys and configures Mimecast Targeted Threat Protection (TTP) modules -- URL Protect (click-time URL rewriting/analysis), Attachment Protect (sandbox detonation), Impersonation Protect (BEC/whaling detection), and Internal Email Protect -- for Microsoft 365 or Google Workspace. Use when defending against phishing, spearphishing, or business email compromise, or configuring TTP policies in the Mimecast Administration Console.","domain":"cybersecurity","path":"skills/implementing-mimecast-targeted-attack-protection"},{"name":"implementing-mitre-attack-coverage-mapping","description":"Implement MITRE ATT&CK coverage mapping to identify detection gaps, prioritize rule development, and measure SOC detection maturity against adversary techniques.","domain":"cybersecurity","path":"skills/implementing-mitre-attack-coverage-mapping"},{"name":"implementing-mobile-application-management","description":"Implements Mobile Application Management (MAM) policies to protect enterprise data on managed and unmanaged mobile devices through app-level controls including data loss prevention, selective wipe, app configuration, and containerization. Use when securing corporate apps on BYOD devices, implementing Intune App Protection Policies, or enforcing data separation between personal and work apps. Activates for requests involving MAM deployment, app protection policies, mobile containerization, or BYOD security.","domain":"cybersecurity","path":"skills/implementing-mobile-application-management"},{"name":"implementing-mtls-for-zero-trust-services","description":"Configures mutual TLS (mTLS) authentication between microservices using Python cryptography library for certificate generation and ssl module for TLS verification. Validates certificate chains, checks expiration, and audits mTLS deployment status. Use when implementing zero-trust service-to-service authentication.","domain":"cybersecurity","path":"skills/implementing-mtls-for-zero-trust-services"},{"name":"implementing-nerc-cip-compliance-controls","description":"Implements NERC CIP controls for Bulk Electric System (BES) cyber systems: asset categorization (CIP-002), electronic security perimeters (CIP-005), system security management (CIP-007), configuration management (CIP-010), and supply chain risk (CIP-013), including 2025 MFA updates. Use when a registered entity must achieve or maintain NERC CIP compliance, prepare for a Regional Entity audit, or categorize newly commissioned BES cyber systems.","domain":"cybersecurity","path":"skills/implementing-nerc-cip-compliance-controls"},{"name":"implementing-network-access-control","description":"Implements 802.1X port-based network access control using RADIUS authentication, PacketFence NAC, and switch configuration to enforce identity-based access policies, posture assessment, and automatic VLAN assignment for authorized devices. Use when enforcing zero-trust access at the network edge, quarantining non-compliant endpoints, meeting PCI-DSS/HIPAA/SOC 2 access requirements, or onboarding BYOD devices with automated provisioning.","domain":"cybersecurity","path":"skills/implementing-network-access-control"},{"name":"implementing-network-access-control-with-cisco-ise","description":"Deploys Cisco Identity Services Engine (ISE) as a RADIUS policy server for 802.1X wired and wireless authentication, MAC Authentication Bypass, posture assessment, dynamic VLAN assignment, downloadable ACLs, and TrustSec Security Group Tags. Use when deploying enterprise NAC with ISE and Active Directory integration, enforcing endpoint posture compliance, or segmenting access with TrustSec instead of a generic 802.1X/PacketFence setup.","domain":"cybersecurity","path":"skills/implementing-network-access-control-with-cisco-ise"},{"name":"implementing-network-deception-with-honeypots","description":"Deploy and manage network honeypots using OpenCanary, T-Pot, or Cowrie to detect unauthorized access, lateral movement, and attacker reconnaissance.","domain":"cybersecurity","path":"skills/implementing-network-deception-with-honeypots"},{"name":"implementing-network-intrusion-prevention-with-suricata","description":"Deploys and configures Suricata as an inline network intrusion prevention system, covering IPS mode setup (NFQueue), custom rule writing, Emerging Threats ruleset management, performance tuning, and logging integration. Use when deploying real-time inline traffic inspection to actively block malicious traffic, or when tuning Suricata rules and performance for production IDS/IPS deployment.","domain":"cybersecurity","path":"skills/implementing-network-intrusion-prevention-with-suricata"},{"name":"implementing-network-policies-for-kubernetes","description":"Writes portable upstream Kubernetes NetworkPolicy YAML - default-deny-all, DNS egress, namespace and pod selector rules - that works on any conformant CNI such as Calico or Cilium. Use when segmentation must stay CNI-portable, introducing a default-deny posture, or restricting east-west traffic between pods and namespaces without depending on a vendor CRD. Keywords: NetworkPolicy, default deny, podSelector, namespaceSelector, ingress, egress, CNI portable. Do not use for Calico-specific resources - use implementing-kubernetes-network-policy-with-calico.","domain":"cybersecurity","path":"skills/implementing-network-policies-for-kubernetes"},{"name":"implementing-network-segmentation-for-ot","description":"Implements OT network segmentation using VLANs, OT-aware firewalls, data diodes, and IEC 62443 zone/conduit architecture, with a traffic-baseline-driven design tool for migrating flat Purdue-model networks without disrupting operations. Use when segmenting a flat OT network into Purdue levels, deploying an IT/OT DMZ, or isolating safety instrumented systems from basic process control systems.","domain":"cybersecurity","path":"skills/implementing-network-segmentation-for-ot"},{"name":"implementing-network-segmentation-with-firewall-zones","description":"Designs and implements network segmentation using firewall security zones, VLANs, inter-zone ACLs, and workload-level microsegmentation to restrict east-west lateral movement and enforce least-privilege access. Use when architecting security zones, writing inter-zone firewall policies, or meeting PCI DSS/HIPAA/NIST 800-53/zero-trust segmentation requirements for dynamic or traditional network environments.","domain":"cybersecurity","path":"skills/implementing-network-segmentation-with-firewall-zones"},{"name":"implementing-network-traffic-analysis-with-arkime","description":"Queries Arkime (formerly Moloch) full packet capture via its API to search sessions, download PCAPs, detect C2 beaconing through connection interval/jitter stats, spot DNS tunneling via query-length analysis, and flag known-bad TLS certificate issuers, using the bundled scripts/agent.py. Use when investigating suspicious network flows or doing full-packet-capture forensics against an Arkime deployment.","domain":"cybersecurity","path":"skills/implementing-network-traffic-analysis-with-arkime"},{"name":"implementing-network-traffic-baselining","description":"Builds network traffic baselines from NetFlow/IPFIX CSV or JSON exports using Python pandas, computing hourly/daily volume distributions, per-host and protocol/port statistics, and top-talker profiles, then flags outliers via z-score and IQR anomaly detection. Use when a SOC analyst needs to establish normal traffic patterns and surface deviations such as data exfiltration spikes, beaconing, or unusual port usage from historical flow data.","domain":"cybersecurity","path":"skills/implementing-network-traffic-baselining"},{"name":"implementing-next-generation-firewall-with-palo-alto","description":"Configures and deploys Palo Alto Networks next-generation firewalls end-to-end, covering App-ID application-aware policies, User-ID identity-based enforcement, zone-based security rules, SSL decryption for encrypted traffic visibility, and Content-ID threat prevention profiles. Use when moving an enterprise from port-based firewall rules to application- and identity-aware NGFW policy on Palo Alto hardware.","domain":"cybersecurity","path":"skills/implementing-next-generation-firewall-with-palo-alto"},{"name":"implementing-opa-gatekeeper-for-policy-enforcement","description":"Deploys OPA Gatekeeper via Helm as a Kubernetes admission controller and writes ConstraintTemplates with Rego plus instantiated Constraints to validate, mutate, or deny resource requests at admission time. Use when enforcing custom policy-as-code at admission on Kubernetes v1.24+, blocking non-compliant workloads before scheduling, or expressing a rule that built-in controls cannot. Keywords: Gatekeeper, ConstraintTemplate, Constraint, Rego, admission webhook, audit, mutation. Do not use for the standard pod security profiles that Pod Security Admission already covers - use implementing-pod-security-admission-controller.","domain":"cybersecurity","path":"skills/implementing-opa-gatekeeper-for-policy-enforcement"},{"name":"implementing-ot-incident-response-playbook","description":"Develops OT-specific incident response playbooks using a SANS PICERL-based Python engine that classifies incident severity (safety, process, access, recon) and coordinates IT SOC, OT engineering, and plant operations, aligned with IEC 62443 and NIST SP 800-82. Use when building ICS/SCADA incident response procedures for the first time, preparing for OT ransomware scenarios, or aligning IR with IEC 62443/NERC CIP reporting requirements.","domain":"cybersecurity","path":"skills/implementing-ot-incident-response-playbook"},{"name":"implementing-ot-network-traffic-analysis-with-nozomi","description":"Deploy Nozomi Networks Guardian sensors for passive OT network traffic analysis, providing asset visibility, behavioral anomaly detection, protocol-aware monitoring, and vulnerability assessment across industrial control systems without disrupting operations. Use when deploying OT/ICS network monitoring, configuring Guardian sensors, or building real-time threat detection for SCADA and industrial environments.","domain":"cybersecurity","path":"skills/implementing-ot-network-traffic-analysis-with-nozomi"},{"name":"implementing-pam-for-database-access","description":"Deploy privileged access management for database systems including Oracle, SQL Server, PostgreSQL, and MySQL, covering session proxy configuration, credential vaulting, query auditing, dynamic credential generation, and least-privilege database roles. Use when securing DBA access, implementing database PAM controls, or auditing privileged database sessions.","domain":"cybersecurity","path":"skills/implementing-pam-for-database-access"},{"name":"implementing-passwordless-auth-with-microsoft-entra","description":"Implements passwordless authentication using Microsoft Entra ID with FIDO2 security keys, Windows Hello for Business, Microsoft Authenticator passkeys, and certificate-based authentication to eliminate password-based attacks. Use when deploying passwordless sign-in, configuring FIDO2 passkeys, enforcing phishing-resistant MFA, or setting Microsoft Entra authentication method policies.","domain":"cybersecurity","path":"skills/implementing-passwordless-auth-with-microsoft-entra"},{"name":"implementing-passwordless-authentication-with-fido2","description":"Deploy FIDO2/WebAuthn passwordless authentication using security keys and platform authenticators, covering WebAuthn API integration, FIDO2 server configuration, passkey enrollment, biometric authentication, and migration from password-based systems aligned with NIST SP 800-63B AAL3. Use when implementing passkey login, configuring a FIDO2/WebAuthn server, or replacing passwords with phishing-resistant authentication.","domain":"cybersecurity","path":"skills/implementing-passwordless-authentication-with-fido2"},{"name":"implementing-patch-management-for-ot-systems","description":"Implements a structured patch management program for OT/ICS environments where IT-style patching can cause process disruption or safety hazards, covering vendor compatibility testing, risk-based prioritization, staged test deployment, maintenance window coordination, rollback procedures, and compensating controls. Use when planning or auditing patching for SCADA, PLCs, or other industrial control systems.","domain":"cybersecurity","path":"skills/implementing-patch-management-for-ot-systems"},{"name":"implementing-patch-management-workflow","description":"Patch management is the systematic process of identifying, testing, deploying, and verifying software updates to remediate vulnerabilities across an organization's IT infrastructure. An effective patc","domain":"cybersecurity","path":"skills/implementing-patch-management-workflow"},{"name":"implementing-pci-dss-compliance-controls","description":"Implements PCI DSS 4.0.1's 12 requirements across 6 control objectives for organizations that store, process, or transmit cardholder data, including the customized validation approach, enhanced authentication, and continuous monitoring controls introduced by the 51 requirements mandatory since March 2025. Use when scoping a cardholder data environment, building PCI DSS 4.0.1 compliance controls, or preparing for a PCI assessment.","domain":"cybersecurity","path":"skills/implementing-pci-dss-compliance-controls"},{"name":"implementing-pod-security-admission-controller","description":"Configures and operates the Kubernetes Pod Security Admission (PSA) controller that enforces Pod Security Standards: namespace enforce/audit/warn labels, cluster-wide defaults via AdmissionConfiguration, exemptions for usernames, runtime classes and namespaces, version pinning, and troubleshooting pods the controller rejected. Use when wiring PSA up on a cluster, setting cluster-wide default enforcement, exempting system namespaces, debugging why a pod was rejected or why enforcement is not firing, or reading PSA audit and warning output. Keywords: Pod Security Admission, PSA, admission controller, AdmissionConfiguration, pod-security.kubernetes.io labels, enforce audit warn, exemptions, kube-apiserver. Do not use for choosing which security profile a workload needs - use implementing-kubernetes-pod-security-standards.","domain":"cybersecurity","path":"skills/implementing-pod-security-admission-controller"},{"name":"implementing-policy-as-code-with-open-policy-agent","description":"Implements policy-as-code enforcement with Open Policy Agent (OPA) and Gatekeeper for Kubernetes and CI/CD pipelines, covering writing Rego policies, deploying OPA Gatekeeper as a Kubernetes admission controller, testing policies in development, and integrating policy evaluation into deployment pipelines. Use when writing Rego policies, deploying Gatekeeper admission control, or gating CI/CD pipelines with policy-as-code checks.","domain":"cybersecurity","path":"skills/implementing-policy-as-code-with-open-policy-agent"},{"name":"implementing-privileged-access-management-with-cyberark","description":"Deploy CyberArk Privileged Access Management to discover, vault, rotate, and monitor privileged credentials across enterprise infrastructure, covering vault architecture, session isolation, credential rotation policies, and integration with NIST 800-53 access control requirements. Use when standing up CyberArk PAM, vaulting privileged credentials, or designing credential rotation policies.","domain":"cybersecurity","path":"skills/implementing-privileged-access-management-with-cyberark"},{"name":"implementing-privileged-access-workstation","description":"Design and implement Privileged Access Workstations (PAWs) using the tiered administration model, with device hardening, device compliance enforcement via Microsoft Intune or Group Policy, just-in-time (JIT) access provisioning, and integration with PAM platforms like CyberArk or BeyondTrust. Use when hardening admin endpoints, designing a tiered administration model, or securing privileged administrative operations.","domain":"cybersecurity","path":"skills/implementing-privileged-access-workstation"},{"name":"implementing-privileged-session-monitoring","description":"Implements privileged session monitoring and recording using PAM solutions, focusing on CyberArk Privileged Session Manager (PSM) and open-source alternatives, covering session recording configuration, keystroke logging, real-time monitoring, risk-based session analysis, and compliance audit trail generation. Use when configuring CyberArk PSM, recording administrator sessions, or building compliance audit trails for privileged access.","domain":"cybersecurity","path":"skills/implementing-privileged-session-monitoring"},{"name":"implementing-proofpoint-email-security-gateway","description":"Deploy and configure Proofpoint Email Protection as a secure email gateway to detect and block phishing, malware, BEC, and spam before messages reach user inboxes.","domain":"cybersecurity","path":"skills/implementing-proofpoint-email-security-gateway"},{"name":"implementing-purdue-model-network-segmentation","description":"Implement network segmentation based on the Purdue Enterprise Reference Architecture (PERA) model, separating ICS networks into hierarchical security zones from Level 0 physical process through Level 5 enterprise and enforcing strict traffic control through IEC 62443-aligned DMZs between OT and IT domains. Use when designing ICS/SCADA network zones or segmenting OT from IT networks.","domain":"cybersecurity","path":"skills/implementing-purdue-model-network-segmentation"},{"name":"implementing-ransomware-backup-strategy","description":"Designs a ransomware-resilient backup strategy using the 3-2-1-1-0 methodology (3 copies, 2 media types, 1 offsite, 1 immutable/air-gapped, 0 restore errors), configuring RPO/RTO-aligned schedules, isolating backup credentials, and automating restore testing. Use when planning ransomware backup resilience or air-gapped/immutable backup architecture.","domain":"cybersecurity","path":"skills/implementing-ransomware-backup-strategy"},{"name":"implementing-ransomware-kill-switch-detection","description":"Analyzes ransomware kill switch mechanisms, including mutex-based execution guards, domain-based kill switches (e.g. WannaCry-style), and registry termination checks, then implements mutex vaccination and kill switch domain monitoring to stop ransomware before it runs. Use when analyzing a sample's execution guards or deploying vaccination/monitoring as a defensive control.","domain":"cybersecurity","path":"skills/implementing-ransomware-kill-switch-detection"},{"name":"implementing-rapid7-insightvm-for-scanning","description":"Deploy and configure Rapid7 InsightVM Security Console and Scan Engines, including scan templates, credentialed scanning, and Insight Agent integration, for authenticated and unauthenticated vulnerability scanning across enterprise environments. Use when standing up InsightVM infrastructure, configuring credentialed vulnerability scans, or integrating continuous asset assessment via the Insight Agent.","domain":"cybersecurity","path":"skills/implementing-rapid7-insightvm-for-scanning"},{"name":"implementing-rbac-hardening-for-kubernetes","description":"Hardens Kubernetes RBAC by designing least-privilege Roles and ClusterRoles, auditing RoleBindings, eliminating cluster-admin sprawl, separating service accounts, and integrating an external OIDC identity provider. Use when tightening cluster access control, removing excessive ClusterRoleBindings, or hardening service-account permissions against escalation and lateral movement. Keywords: RBAC, Role, ClusterRole, RoleBinding, least privilege, service account, OIDC, cluster-admin. Do not use for discovering existing escalation paths - use auditing-kubernetes-rbac-privilege-escalation.","domain":"cybersecurity","path":"skills/implementing-rbac-hardening-for-kubernetes"},{"name":"implementing-rsa-key-pair-management","description":"Generates, stores, rotates, and manages RSA key pairs following NIST SP 800-57 guidelines, covering serialization formats (PEM, DER, PKCS#8), passphrase protection, and key strength validation. Use when creating or rotating RSA keys for signatures, key exchange, or encryption, or when auditing existing keys for proper storage and NIST-compliant strength.","domain":"cybersecurity","path":"skills/implementing-rsa-key-pair-management"},{"name":"implementing-runtime-application-self-protection","description":"Deploy Runtime Application Self-Protection (RASP) agents to detect and block attacks from within application runtime, covering OpenRASP integration, attack pattern detection, and security policy configuration for Java and Python web applications.","domain":"cybersecurity","path":"skills/implementing-runtime-application-self-protection"},{"name":"implementing-runtime-security-with-tetragon","description":"Implements eBPF-based runtime observability and in-kernel enforcement in Kubernetes with Cilium Tetragon, monitoring process execution, file access, network connections, and syscalls, and blocking dangerous calls at the kernel level. Use when deploying Tetragon to detect or block syscalls such as ptrace, mount, and unshare, enforcing kernel-level policy, or adding low-overhead runtime detection to a cluster. Keywords: Tetragon, Cilium, eBPF, TracingPolicy, kprobe, enforcement, process lineage. Do not use for Falco-based detection - use detecting-container-runtime-threats-with-falco.","domain":"cybersecurity","path":"skills/implementing-runtime-security-with-tetragon"},{"name":"implementing-saml-sso-with-okta","description":"Implement SAML 2.0 Single Sign-On using Okta as the Identity Provider, covering SP-initiated and IdP-initiated flows, attribute mapping, certificate management, SHA-256 signature enforcement, and Single Logout. Use when configuring Okta SAML SSO for an application, hardening SAML certificate rotation and signing, or testing and troubleshooting SAML assertion flows with a tracer tool.","domain":"cybersecurity","path":"skills/implementing-saml-sso-with-okta"},{"name":"implementing-scim-provisioning-with-okta","description":"Implement automated user lifecycle provisioning and deprovisioning using the SCIM 2.0 protocol with Okta as the identity provider. Use when automating account creation, attribute sync, or deactivation across downstream applications through Okta SCIM integration, or when troubleshooting SCIM provisioning failures.","domain":"cybersecurity","path":"skills/implementing-scim-provisioning-with-okta"},{"name":"implementing-secret-scanning-with-gitleaks","description":"This skill covers implementing Gitleaks for detecting and preventing hardcoded secrets in git repositories. It addresses configuring pre-commit hooks, CI/CD pipeline integration, custom rule authoring for organization-specific secrets, baseline management for existing repositories, and remediation workflows for exposed credentials.","domain":"cybersecurity","path":"skills/implementing-secret-scanning-with-gitleaks"},{"name":"implementing-secrets-management-with-vault","description":"Deploy HashiCorp Vault for centralized secrets management, covering dynamic secret generation for databases and cloud providers, transit encryption, PKI certificate management, and Kubernetes integration. Use when eliminating hardcoded credentials from application code or CI/CD pipelines, migrating to short-lived auto-rotated secrets, or giving Kubernetes workloads secure access to database or cloud provider credentials.","domain":"cybersecurity","path":"skills/implementing-secrets-management-with-vault"},{"name":"implementing-secrets-scanning-in-ci-cd","description":"Integrate gitleaks and trufflehog into CI/CD pipelines to detect leaked secrets before deployment","domain":"cybersecurity","path":"skills/implementing-secrets-scanning-in-ci-cd"},{"name":"implementing-security-chaos-engineering","description":"Implements security chaos engineering experiments that deliberately disable or degrade security controls to verify detection and response capabilities. Tests WAF bypass, firewall rule removal, log pipeline disruption, and EDR disablement scenarios using boto3 and subprocess. Use when validating SOC detection coverage and resilience.","domain":"cybersecurity","path":"skills/implementing-security-chaos-engineering"},{"name":"implementing-security-information-sharing-with-stix2","description":"Create, validate, and share STIX 2.1 threat intelligence objects (indicators, malware, campaigns, relationships, bundles) using the stix2 Python library, and publish them over TAXII 2.1. Use when building or exchanging structured threat intelligence, modeling relationships between threat objects, or publishing/consuming a TAXII 2.1 feed.","domain":"cybersecurity","path":"skills/implementing-security-information-sharing-with-stix2"},{"name":"implementing-security-monitoring-with-datadog","description":"Implements security monitoring using Datadog Cloud SIEM, Cloud Security Management (CSM), and Workload Protection to detect threats, enforce compliance, and respond to security events across cloud and hybrid infrastructure. Covers Agent deployment, log source ingestion, detection rule creation, security dashboards, and automated notification workflows. Activates for requests involving Datadog security setup, Cloud SIEM configuration, CSM threat detection, or security monitoring dashboards.","domain":"cybersecurity","path":"skills/implementing-security-monitoring-with-datadog"},{"name":"implementing-semgrep-for-custom-sast-rules","description":"Write custom Semgrep SAST rules in YAML to detect application-specific vulnerabilities, enforce coding standards, and integrate into CI/CD pipelines.","domain":"cybersecurity","path":"skills/implementing-semgrep-for-custom-sast-rules"},{"name":"implementing-siem-correlation-rules-for-apt","description":"Write multi-event correlation rules in Splunk SPL and Sigma format that detect APT lateral movement by chaining Windows authentication events (4624, 4648), process execution (4688, Sysmon Event 1), and network connections (Sysmon Event 3) across hosts within sliding time windows. Use when building SIEM correlation searches to surface multi-stage attack sequences that single-event detections miss, such as pass-the-hash or lateral movement chains.","domain":"cybersecurity","path":"skills/implementing-siem-correlation-rules-for-apt"},{"name":"implementing-siem-use-case-tuning","description":"Tune SIEM detection rules in Splunk and Elastic to reduce false positives by analyzing alert volumes, creating context-aware exclusion lists, adjusting thresholds against environmental baselines, and measuring precision/recall efficacy metrics. Use when a SOC is drowning in noisy alerts and needs to tune correlation searches or detection rules, or when measuring and reporting alert-to-incident conversion rates.","domain":"cybersecurity","path":"skills/implementing-siem-use-case-tuning"},{"name":"implementing-siem-use-cases-for-detection","description":"Implements SIEM detection use cases by designing correlation rules, threshold alerts, and behavioral analytics mapped to MITRE ATT&CK techniques across Splunk, Elastic, and Sentinel. Use when SOC teams need to expand detection coverage, formalize use case lifecycle management, or build a detection library aligned to organizational threat profile.","domain":"cybersecurity","path":"skills/implementing-siem-use-cases-for-detection"},{"name":"implementing-sigstore-for-software-signing","description":"Implements Sigstore-based software signing and verification using Cosign keyless signing, Rekor transparency log verification, and Fulcio certificate authority integration to establish cryptographic provenance for container images, binaries, and software artifacts, including OIDC-based identity binding and CI/CD pipeline integration. Use when setting up keyless container/artifact signing, verifying signatures against the Rekor transparency log, or deploying Sigstore in a supply-chain security workflow.","domain":"cybersecurity","path":"skills/implementing-sigstore-for-software-signing"},{"name":"implementing-soar-automation-with-phantom","description":"Implements Security Orchestration, Automation, and Response (SOAR) workflows using Splunk SOAR (formerly Phantom) to automate alert triage, IOC enrichment, containment actions, and incident response playbooks. Use when SOC teams need to reduce manual analyst work, standardize response procedures, or integrate multiple security tools into automated workflows.","domain":"cybersecurity","path":"skills/implementing-soar-automation-with-phantom"},{"name":"implementing-soar-playbook-for-phishing","description":"Automates phishing incident response by calling the Splunk SOAR (Phantom) REST API to create containers, attach artifacts (emails, URLs, attachments), and trigger response playbooks. Use when building or wiring up a Splunk SOAR phishing playbook, ingesting a suspected phishing report into SOAR, or automating containment and triage for phishing incidents.","domain":"cybersecurity","path":"skills/implementing-soar-playbook-for-phishing"},{"name":"implementing-soar-playbook-with-palo-alto-xsoar","description":"Build automated incident response playbooks in Cortex XSOAR (Demisto) using its YAML playbook structure, integration commands, and task types to orchestrate phishing, malware, account-compromise, and DDoS response workflows across SOC tools. Use when authoring or wiring up an XSOAR playbook, adding custom XSOAR integration commands or Python automation scripts, or reducing manual SOC response time via orchestration.","domain":"cybersecurity","path":"skills/implementing-soar-playbook-with-palo-alto-xsoar"},{"name":"implementing-stix-taxii-feed-integration","description":"Implements a STIX 2.1/TAXII 2.1 threat-intelligence feed consumer and producer in Python, covering TAXII server discovery, collection polling, parsing STIX bundles with the stix2 library, and standing up a local TAXII server with Medallion. Use when integrating a STIX/TAXII CTI feed into a SIEM or TIP, writing a TAXII client to poll for new indicators, or setting up TAXII collections for indicator exchange.","domain":"cybersecurity","path":"skills/implementing-stix-taxii-feed-integration"},{"name":"implementing-supply-chain-security-with-in-toto","description":"Implements supply chain integrity verification for container builds with the in-toto framework: generating signing keys, defining a supply chain layout, recording pipeline steps as signed link metadata, verifying before deployment, enforcing at Kubernetes admission, and integrating with SLSA. Use when attesting CI/CD pipeline steps, proving an image followed the approved build process, or enforcing provenance at admission. Keywords: in-toto, layout, link metadata, step, inspection, SLSA, provenance, admission. Do not use for signing and verifying images with Cosign - use implementing-image-provenance-verification-with-cosign.","domain":"cybersecurity","path":"skills/implementing-supply-chain-security-with-in-toto"},{"name":"implementing-syslog-centralization-with-rsyslog","description":"Configure rsyslog for centralized log collection with TLS encryption, custom templates, and log rotation, generating server and client configuration files with GnuTLS stream drivers, x509 certificate authentication, per-host log segregation, and reliable queue settings. Use when building a centralized, encrypted syslog pipeline, hardening rsyslog client/server configs for high-availability log infrastructure, or troubleshooting TLS-based syslog forwarding.","domain":"cybersecurity","path":"skills/implementing-syslog-centralization-with-rsyslog"},{"name":"implementing-taxii-server-with-opentaxii","description":"Deploy and configure a TAXII 2.1 server (Medallion) with Docker, publish and consume STIX 2.1 bundles across collections, and integrate the feed with SIEM/SOAR platforms for automated indicator exchange between organizations. Use when standing up a TAXII server to share threat intel, configuring collections for CTI feeds, or automating STIX indicator ingestion into a SIEM/SOAR.","domain":"cybersecurity","path":"skills/implementing-taxii-server-with-opentaxii"},{"name":"implementing-threat-intelligence-lifecycle-management","description":"Build out a full CTI program around the six-phase threat intelligence lifecycle (direction, collection, processing, analysis, dissemination, feedback), including defining intelligence requirements, building a collection pipeline, normalizing data, and tracking dissemination feedback. Use when standing up or maturing a threat intelligence program, defining intelligence requirements, or designing collection-to-dissemination workflows for a CTI team.","domain":"cybersecurity","path":"skills/implementing-threat-intelligence-lifecycle-management"},{"name":"implementing-threat-modeling-with-mitre-attack","description":"Implements threat modeling using the MITRE ATT&CK framework to map adversary TTPs against organizational assets, assess detection coverage gaps, and prioritize defensive investments. Use when SOC teams need to align detection engineering with threat landscape, conduct threat assessments for new environments, or justify security tool procurement.","domain":"cybersecurity","path":"skills/implementing-threat-modeling-with-mitre-attack"},{"name":"implementing-ticketing-system-for-incidents","description":"Implements an integrated incident ticketing system connecting SIEM alerts to ServiceNow, Jira, or TheHive for structured incident tracking, SLA management, escalation workflows, and compliance documentation. Use when SOC teams need formalized incident lifecycle management with automated ticket creation, assignment routing, and resolution tracking.","domain":"cybersecurity","path":"skills/implementing-ticketing-system-for-incidents"},{"name":"implementing-usb-device-control-policy","description":"Implements USB device control policies to restrict unauthorized removable media access on endpoints, preventing data exfiltration and malware introduction via USB devices. Use when deploying device control via Group Policy, Intune, or EDR platforms to enforce USB restrictions. Activates for requests involving USB control, removable media policy, device control, or data loss prevention via USB.","domain":"cybersecurity","path":"skills/implementing-usb-device-control-policy"},{"name":"implementing-velociraptor-for-ir-collection","description":"Deploy and configure Velociraptor for scalable endpoint forensic artifact collection during incident response using VQL queries, hunts, and pre-built artifact packs across Windows, Linux, and macOS environments.","domain":"cybersecurity","path":"skills/implementing-velociraptor-for-ir-collection"},{"name":"implementing-vulnerability-management-with-greenbone","description":"Deploy and operate Greenbone/OpenVAS vulnerability management using the python-gvm library over the Greenbone Management Protocol (GMP) to connect via Unix socket or TLS, create scan targets and configs, execute scans, and parse the XML scan reports into actionable findings. Use when automating OpenVAS/GVM scan creation and execution, or programmatically retrieving and parsing vulnerability scan reports.","domain":"cybersecurity","path":"skills/implementing-vulnerability-management-with-greenbone"},{"name":"implementing-vulnerability-remediation-sla","description":"Design a vulnerability remediation SLA program covering asset tiering, a severity-based SLA matrix, exception processes, escalation chains, ticketing-system integration, and remediation KPIs/trending metrics. Use when defining mandatory patching timeframes by severity and asset criticality, building an SLA policy document, or setting up escalation and KPI tracking for vulnerability remediation.","domain":"cybersecurity","path":"skills/implementing-vulnerability-remediation-sla"},{"name":"implementing-vulnerability-sla-breach-alerting","description":"Build an automated SLA breach alerting system for vulnerability remediation, including a database schema for SLA tracking, breach detection logic, notification dispatch, a scheduled check runner, and a KPI/compliance metrics dashboard. Use when implementing severity-based SLA timelines (critical/high/medium/low), detecting and escalating SLA breaches, or building vulnerability remediation compliance reporting.","domain":"cybersecurity","path":"skills/implementing-vulnerability-sla-breach-alerting"},{"name":"implementing-web-application-logging-with-modsecurity","description":"Configure ModSecurity WAF with the OWASP Core Rule Set (CRS) for web application audit logging, tuning SecRuleEngine, SecAuditEngine, and CRS paranoia levels to reduce false positives, and writing custom SecRules for application-specific threats. Use when deploying or tuning a ModSecurity WAF, analyzing audit logs for attack detection, or reducing CRS false positives.","domain":"cybersecurity","path":"skills/implementing-web-application-logging-with-modsecurity"},{"name":"implementing-zero-knowledge-proof-for-authentication","description":"Implements the Schnorr identification protocol and a simplified Zero-Knowledge Password Proof (ZKPP) over the discrete logarithm problem, letting a prover authenticate by demonstrating knowledge of a secret without ever revealing it to the server. Use when designing or building password-less or password-secret-free authentication, or when a server must verify a user's credential without learning or storing the underlying secret.","domain":"cybersecurity","path":"skills/implementing-zero-knowledge-proof-for-authentication"},{"name":"implementing-zero-standing-privilege-with-cyberark","description":"Deploy CyberArk Secure Cloud Access (SCA) to eliminate standing privileges in AWS, Azure, and GCP by provisioning ephemeral, scoped roles on a just-in-time basis governed by the TEA framework (Time, Entitlements, Approvals). Use when designing or implementing zero standing privilege / just-in-time privileged access models with CyberArk, or when replacing persistent cloud admin roles with time-bound, approval-gated sessions.","domain":"cybersecurity","path":"skills/implementing-zero-standing-privilege-with-cyberark"},{"name":"implementing-zero-trust-dns-with-nextdns","description":"Configure NextDNS as an encrypted (DoH/DoT) zero trust DNS resolver that blocks malicious, phishing, and cryptojacking domains via real-time threat intelligence, detects DNS rebinding and CNAME cloaking, and enforces organizational DNS policy across endpoints. Use when deploying DNS-layer threat blocking and acceptable-use enforcement, or when extending zero trust controls (including Windows 11 Zero Trust DNS) to the DNS resolution path.","domain":"cybersecurity","path":"skills/implementing-zero-trust-dns-with-nextdns"},{"name":"implementing-zero-trust-for-saas-applications","description":"Secures SaaS apps (Microsoft 365, Google Workspace, Salesforce, Slack) via CASB/SSPM deployment, conditional access policies, OAuth app governance, and session-level DLP controls enforcing identity verification and device compliance. Use when adding MFA/device-compliance conditional access, discovering shadow IT, governing OAuth consent grants, or applying session controls to sensitive SaaS data.","domain":"cybersecurity","path":"skills/implementing-zero-trust-for-saas-applications"},{"name":"implementing-zero-trust-in-cloud","description":"Guides zero trust implementation across AWS, Azure, and GCP per NIST SP 800-207 and BeyondCorp principles, covering identity-centric access, micro-segmentation, continuous verification, device trust assessment, and Identity-Aware Proxy deployment. Use when migrating from perimeter security to identity-centric access, removing VPN dependency, or designing micro-segmentation for multi-cloud workloads.","domain":"cybersecurity","path":"skills/implementing-zero-trust-in-cloud"},{"name":"implementing-zero-trust-network-access","description":"Configures Zero Trust Network Access (ZTNA) in AWS, Azure, and GCP using identity-aware proxies, micro-segmentation, and continuous verification with conditional access policies, replacing VPN-based access with BeyondCorp-style architectures. Use when replacing VPN remote access with identity-based controls, limiting lateral movement via micro-segmentation, or exposing cloud workloads to authenticated users without public internet exposure.","domain":"cybersecurity","path":"skills/implementing-zero-trust-network-access"},{"name":"implementing-zero-trust-network-access-with-zscaler","description":"Configures Zero Trust Network Access using Zscaler Private Access (ZPA) to broker identity-based, context-aware connections between authenticated users and internal applications through the Zscaler Zero Trust Exchange, without placing users on the corporate network. Use when replacing traditional VPN architectures with ZTNA, or when brokering secure per-application access for remote and hybrid users via Zscaler.","domain":"cybersecurity","path":"skills/implementing-zero-trust-network-access-with-zscaler"},{"name":"implementing-zero-trust-with-beyondcorp","description":"Configures Google BeyondCorp Enterprise Identity-Aware Proxy (IAP) as the access enforcement point for web applications, defining Access Context Manager access levels from device trust and network attributes, and auditing the resulting policies for compliance. Use when eliminating perimeter/VPN trust for GCP resources or internal apps, or when setting up identity- and device-posture-based access controls on Google Cloud.","domain":"cybersecurity","path":"skills/implementing-zero-trust-with-beyondcorp"},{"name":"implementing-zero-trust-with-hashicorp-boundary","description":"Installs and configures HashiCorp Boundary as a default-deny, identity-aware proxy for infrastructure access, including controller/worker setup, Vault-backed credential brokering, session recording, and OIDC/LDAP auth across an org/project scope hierarchy. Use when replacing VPN or direct network access with just-in-time, credential-less Boundary sessions, or standing up Boundary controllers and workers.","domain":"cybersecurity","path":"skills/implementing-zero-trust-with-hashicorp-boundary"},{"name":"integrating-dast-with-owasp-zap-in-pipeline","description":"Integrates OWASP ZAP (Zed Attack Proxy) into GitHub Actions and GitLab CI pipelines, covering baseline, full, and API scan configuration against running applications, ZAP finding interpretation, scan policy tuning, and DAST quality gates. Use when testing running web apps or REST/GraphQL APIs for XSS, SQLi, CSRF, and auth/authz flaws, or when SAST alone is insufficient and runtime DAST is required for compliance or release gating.","domain":"cybersecurity","path":"skills/integrating-dast-with-owasp-zap-in-pipeline"},{"name":"integrating-sast-into-github-actions-pipeline","description":"Integrates CodeQL and Semgrep SAST scanning into GitHub Actions, covering scans on pull requests/pushes, rule tuning to cut false positives, SARIF upload to GitHub Advanced Security, and merge-blocking quality gates for high-severity findings. Use when adding automated code vulnerability detection to CI, enforcing consistent SAST org-wide, or producing SOC 2/PCI DSS/NIST SSDF compliance evidence.","domain":"cybersecurity","path":"skills/integrating-sast-into-github-actions-pipeline"},{"name":"intercepting-mobile-traffic-with-burpsuite","description":"Intercepts and analyzes HTTP/HTTPS traffic from mobile applications using Burp Suite proxy to identify insecure API communications, authentication flaws, data leakage, and server-side vulnerabilities. Use when performing mobile application penetration testing, assessing API security, or evaluating client-server communication patterns. Activates for requests involving mobile traffic interception, Burp Suite mobile proxy, API security testing, or mobile HTTPS analysis.","domain":"cybersecurity","path":"skills/intercepting-mobile-traffic-with-burpsuite"},{"name":"investigating-insider-threat-indicators","description":"Investigates insider threat indicators including data exfiltration attempts, unauthorized access patterns, policy violations, and pre-departure behaviors using SIEM analytics, DLP alerts, and HR data correlation. Use when SOC teams receive insider threat referrals from HR, detect anomalous data movement by employees, or need to build investigation timelines for potential insider threats.","domain":"cybersecurity","path":"skills/investigating-insider-threat-indicators"},{"name":"investigating-phishing-email-incident","description":"Investigates phishing email incidents from initial user report through header analysis, URL/attachment detonation, impacted user identification, and containment actions using SOC tools like Splunk, Microsoft Defender, and sandbox analysis platforms. Use when a reported phishing email requires full incident investigation to determine scope and impact.","domain":"cybersecurity","path":"skills/investigating-phishing-email-incident"},{"name":"investigating-ransomware-attack-artifacts","description":"Forensically preserve memory and disk, collect ransom notes and encrypted file samples, and identify the ransomware variant using tools such as ID Ransomware, Volatility, and Chainsaw/Hayabusa to determine the initial access vector and recovery options. Use immediately after discovering ransomware encryption, when scoping the incident forensically, or when documenting evidence for law enforcement and insurance claims.","domain":"cybersecurity","path":"skills/investigating-ransomware-attack-artifacts"},{"name":"managing-cloud-identity-with-okta","description":"Implement Okta as a centralized cloud identity provider: configure SSO with AWS, Azure, and GCP, deploy phishing-resistant MFA with Okta FastPass, automate user provisioning/deprovisioning, and enforce adaptive access policies on device posture and risk signals. Use when standing up Okta SSO, rolling out FastPass MFA, automating identity lifecycle, or building risk-based conditional access for cloud environments.","domain":"cybersecurity","path":"skills/managing-cloud-identity-with-okta"},{"name":"managing-intelligence-lifecycle","description":"Manages the end-to-end cyber threat intelligence lifecycle from planning and direction through collection, processing, analysis, dissemination, and feedback to ensure intelligence products meet stakeholder requirements and continuously improve. Use when establishing or maturing a CTI program, defining intelligence requirements with business stakeholders, or building feedback loops between intelligence consumers and producers. Activates for requests involving CTI program maturity, intelligence requirements, PIRs, or intelligence lifecycle management.","domain":"cybersecurity","path":"skills/managing-intelligence-lifecycle"},{"name":"managing-third-party-vendor-risk","description":"Build and run a third-party/vendor risk management (TPRM) program aligned to NIST SP 800-161 C-SCRM: inventory and tier vendors, issue SIG/CAIQ questionnaires, review SOC 2/ISO 27001 evidence, set contractual right-to-audit clauses, monitor vendors continuously, and offboard securely. Use when assessing a new vendor, standing up a vendor-risk program, tiering a portfolio, reviewing a SOC 2/CAIQ, or writing security terms into a contract.","domain":"cybersecurity","path":"skills/managing-third-party-vendor-risk"},{"name":"mapping-attack-paths-with-bloodhound-ce","description":"Collect Active Directory data with SharpHound and Entra ID data with AzureHound, ingest into BloodHound Community Edition, and analyze on-prem, cloud, and hybrid attack paths using built-in queries and custom Cypher. Use during authorized red-team or penetration-test engagements to map privilege-escalation chains toward domain/tenant compromise.","domain":"cybersecurity","path":"skills/mapping-attack-paths-with-bloodhound-ce"},{"name":"mapping-mitre-attack-techniques","description":"Maps observed adversary behaviors, security alerts, and detection rules to MITRE ATT&CK techniques and sub-techniques to quantify detection coverage and guide control prioritization. Use when building an ATT&CK-based coverage heatmap, tagging SIEM alerts with technique IDs, aligning security controls to adversary playbooks, or reporting threat exposure to executives. Activates for requests involving ATT&CK Navigator, Sigma rules, MITRE D3FEND, or coverage gap analysis.","domain":"cybersecurity","path":"skills/mapping-mitre-attack-techniques"},{"name":"migrating-to-post-quantum-cryptography","description":"Build a cryptographic inventory/CBOM with OpenSSL 3.5+, deploy hybrid post-quantum key exchange (X25519MLKEM768) on TLS/VPN/SSH endpoints, generate ML-KEM/ML-DSA keys and PQC/hybrid certificates, and prioritize migration by harvest-now-decrypt-later (HNDL) exposure per NIST SP 1800-38. Use when inventorying enterprise cryptography for quantum-readiness, enabling hybrid PQC key exchange, or issuing and verifying PQC/hybrid certificates.","domain":"cybersecurity","path":"skills/migrating-to-post-quantum-cryptography"},{"name":"modeling-threats-with-opencti","description":"Deploy OpenCTI (Filigran) via Docker Compose and use the pycti Python client to model threat actors, intrusion sets, campaigns, and indicators as a STIX 2.1 knowledge graph with relationships (uses, attributed-to, targets). Use when building a centralized threat-intel knowledge base, correlating IOCs from multiple feeds into one adversary graph, or producing STIX bundles for detection engineering.","domain":"cybersecurity","path":"skills/modeling-threats-with-opencti"},{"name":"monitoring-darkweb-sources","description":"Monitors dark web forums, marketplaces, paste sites, and ransomware leak sites for mentions of organizational assets, leaked credentials, threatened attacks, and threat actor communications to provide early warning intelligence. Use when establishing dark web monitoring coverage, investigating specific data breach claims, or enriching incident investigations with dark web context. Activates for requests involving dark web OSINT, leak site monitoring, credential exposure, Recorded Future dark web, or Tor hidden service intelligence.","domain":"cybersecurity","path":"skills/monitoring-darkweb-sources"},{"name":"monitoring-scada-modbus-traffic-anomalies","description":"Monitors Modbus TCP traffic on SCADA and ICS networks to detect anomalous function code usage, unauthorized register writes, and suspicious communication patterns. The analyst uses deep packet inspection with pymodbus, Scapy, and Zeek to baseline normal PLC/RTU communication behavior, then applies statistical and rule-based anomaly detection to identify reconnaissance, parameter manipulation, and denial-of-service attacks targeting Modbus devices on port 502. Activates for requests involving Modbus traffic analysis, SCADA network monitoring, ICS anomaly detection, PLC security monitoring, or OT network threat detection.","domain":"cybersecurity","path":"skills/monitoring-scada-modbus-traffic-anomalies"},{"name":"moving-laterally-with-netexec","description":"Use NetExec (nxc) to validate credentials, enumerate SMB shares/users/policy, password-spray safely across lockout thresholds, execute commands, and dump SAM/LSA/NTDS credentials across SMB, WinRM, LDAP, MSSQL, SSH, and other protocol modules. Use during authorized internal penetration tests to find lateral-movement paths after obtaining credentials or hashes, or to conduct controlled password spraying and post-exploitation on in-scope hosts.","domain":"cybersecurity","path":"skills/moving-laterally-with-netexec"},{"name":"operating-havoc-c2","description":"Deploy a Havoc C2 team server with Yaotl malleable profiles, generate evasive Demon agents using indirect syscalls and sleep obfuscation, and run post-exploitation and pivoting operations. Use during authorized, rules-of-engagement-bound red-team engagements for adversary emulation requiring a full C2 lifecycle (server setup, evasive implant generation, tasking, and lateral pivoting).","domain":"cybersecurity","path":"skills/operating-havoc-c2"},{"name":"operating-sliver-c2","description":"Stand up a Sliver C2 server and mTLS listeners, generate cross-platform implants and beacons, and run post-exploitation, pivoting, and BOF/.NET tooling via the Armory for adversary emulation. Use during authorized, rules-of-engagement-bound red-team engagements that need a full open-source C2 lifecycle spanning server setup, implant generation, and post-exploitation tradecraft.","domain":"cybersecurity","path":"skills/operating-sliver-c2"},{"name":"operationalizing-misp-threat-feeds","description":"Stand up MISP, enable and cache curated threat feeds (CIRCL, abuse.ch, Feodo Tracker), apply warninglists to suppress false positives, query indicators with PyMISP, and export attributes as auto-generated Suricata/Sigma/Wazuh detection rules. Use when maturing a MISP instance to actively drive detection, curating threat feeds with quality controls, or automating IOC-to-detection pipelines for the SIEM/IDS.","domain":"cybersecurity","path":"skills/operationalizing-misp-threat-feeds"},{"name":"orchestrating-llm-attacks-with-pyrit","description":"Build automated multi-turn adversarial attacks against conversational LLM targets using Microsoft PyRIT's RedTeamingOrchestrator, CrescendoOrchestrator (gradual escalation), and TreeOfAttacksWithPruningOrchestrator (adaptive branching), with scorer feedback loops and persisted conversation memory. Use when single-shot LLM scanning is insufficient and you need multi-turn, scorer-driven AI red-team campaigns against a chatbot or agent.","domain":"cybersecurity","path":"skills/orchestrating-llm-attacks-with-pyrit"},{"name":"parsing-artifacts-with-eric-zimmerman-tools","description":"Parse Windows forensic artifacts\u2014$MFT/$J (MFTECmd), Prefetch (PECmd), registry hives (RECmd), shellbags, and Amcache\u2014into normalized CSV/JSON with Eric Zimmerman's EZ Tools, then load results into Timeline Explorer for analysis. Use during DFIR/incident-response investigations, after triage collection (e.g. with KAPE), to establish program execution, file/folder access, and persistence evidence from acquired forensic images.","domain":"cybersecurity","path":"skills/parsing-artifacts-with-eric-zimmerman-tools"},{"name":"performing-access-recertification-with-saviynt","description":"Configure and execute access recertification campaigns in Saviynt Enterprise Identity Cloud to validate user entitlements, revoke excessive access, and maintain compliance with SOX, SOC 2, and HIPAA. Use when launching or managing certification campaigns for users/entitlements, remediating access review findings, or documenting identity governance evidence for an audit.","domain":"cybersecurity","path":"skills/performing-access-recertification-with-saviynt"},{"name":"performing-access-review-and-certification","description":"Designs and runs access review and certification campaigns-scoping, reviewer selection, risk-based prioritization, micro-certification, and remediation tracking-to verify user access matches job roles for SOX, HIPAA, and PCI DSS compliance. Use when planning or executing periodic access recertification or preparing evidence for an IAM compliance audit.","domain":"cybersecurity","path":"skills/performing-access-review-and-certification"},{"name":"performing-active-directory-bloodhound-analysis","description":"Use BloodHound and SharpHound (or AzureHound) to enumerate Active Directory relationships and graph attack paths from a compromised user to Domain Admin. Use when performing AD red-team reconnaissance, mapping privilege-escalation chains from group memberships, ACLs, and trusts, or auditing AD for exploitable misconfigurations.","domain":"cybersecurity","path":"skills/performing-active-directory-bloodhound-analysis"},{"name":"performing-active-directory-compromise-investigation","description":"Investigate Active Directory compromise by analyzing authentication logs, replication metadata, Group Policy changes, and Kerberos ticket anomalies to identify attacker persistence and lateral movement paths.","domain":"cybersecurity","path":"skills/performing-active-directory-compromise-investigation"},{"name":"performing-active-directory-forest-trust-attack","description":"Enumerate and audit Active Directory forest trust relationships using Impacket for SID filtering analysis, trust key extraction, cross-forest SID history abuse detection, and inter-realm Kerberos ticket assessment. Use when red-teaming multi-forest AD environments or auditing forest trusts for cross-forest privilege escalation and trust ticket forgery exposure.","domain":"cybersecurity","path":"skills/performing-active-directory-forest-trust-attack"},{"name":"performing-active-directory-penetration-test","description":"Conduct a focused Active Directory penetration test using BloodHound, Impacket, Certipy, Rubeus, and NetExec to enumerate domain objects, discover attack paths, exploit Kerberos weaknesses, escalate privileges via ADCS/DCSync, and demonstrate domain compromise. Use when running an authorized AD pentest from a standard domain user foothold toward Domain Admin or Enterprise Admin.","domain":"cybersecurity","path":"skills/performing-active-directory-penetration-test"},{"name":"performing-active-directory-vulnerability-assessment","description":"Assess Active Directory security posture using PingCastle, BloodHound, and Purple Knight to identify misconfigurations, privilege escalation paths, and attack vectors.","domain":"cybersecurity","path":"skills/performing-active-directory-vulnerability-assessment"},{"name":"performing-adversary-in-the-middle-phishing-detection","description":"Detect and respond to Adversary-in-the-Middle (AiTM) phishing attacks that use reverse proxy kits like EvilProxy, Evilginx, and Tycoon 2FA to bypass MFA and steal session tokens, correlating Azure AD/Entra sign-in logs, SIEM alerts, and EDR telemetry. Use when investigating suspected MFA-bypass phishing or session token theft, or building detection and response playbooks against reverse-proxy phishing kits.","domain":"cybersecurity","path":"skills/performing-adversary-in-the-middle-phishing-detection"},{"name":"performing-agentless-vulnerability-scanning","description":"Configure and execute agentless vulnerability scanning using network protocols, cloud snapshot analysis, and API-based discovery to assess systems without installing endpoint agents.","domain":"cybersecurity","path":"skills/performing-agentless-vulnerability-scanning"},{"name":"performing-ai-driven-osint-correlation","description":"Use AI/LLM-based reasoning with Sherlock, theHarvester, and SpiderFoot to correlate OSINT findings\u2014usernames, emails, social profiles, domain records, breach databases, and dark-web mentions\u2014into unified, confidence-scored intelligence profiles with link analysis. Use when raw OSINT data from multiple sources needs merging into one target profile or resolving identity linkage across platforms.","domain":"cybersecurity","path":"skills/performing-ai-driven-osint-correlation"},{"name":"performing-alert-triage-with-elastic-siem","description":"Perform systematic alert triage in Elastic Security SIEM\u2014classifying, prioritizing, and investigating alerts using Kibana, ES|QL queries, and ECS-normalized data\u2014to drive SOC analyst workflows. Use when triaging incoming Elastic Security detections, prioritizing an analyst's alert queue, or investigating alerts during SOC operations.","domain":"cybersecurity","path":"skills/performing-alert-triage-with-elastic-siem"},{"name":"performing-android-app-static-analysis-with-mobsf","description":"Performs automated static analysis of Android applications using Mobile Security Framework (MobSF) to identify hardcoded secrets, insecure permissions, vulnerable components, weak cryptography, and code-level security flaws without executing the application. Use when assessing Android APK/AAB files for security vulnerabilities before deployment, during penetration testing, or as part of CI/CD security gates. Activates for requests involving Android static analysis, MobSF scanning, APK security assessment, or mobile application code review.","domain":"cybersecurity","path":"skills/performing-android-app-static-analysis-with-mobsf"},{"name":"performing-api-fuzzing-with-restler","description":"Uses Microsoft RESTler to perform stateful REST API fuzzing: compiles an OpenAPI/Swagger spec into a RESTler grammar, configures authentication, and runs test/fuzz-lean/fuzz modes that generate request sequences exercising producer-consumer dependencies, then flags 500 errors, auth bypasses, resource leaks, and injection bugs. Use when fuzzing REST APIs for stateful bugs or running RESTler-based automated API security testing.","domain":"cybersecurity","path":"skills/performing-api-fuzzing-with-restler"},{"name":"performing-api-inventory-and-discovery","description":"Performs API inventory and discovery to identify all API endpoints in an organization's environment including documented, undocumented, shadow, zombie, and deprecated APIs. The tester uses passive traffic analysis, active scanning, DNS enumeration, JavaScript analysis, and cloud resource inventory to build a comprehensive API catalog. Maps to OWASP API9:2023 Improper Inventory Management. Activates for requests involving API discovery, shadow API detection, API inventory audit, or attack surface mapping.","domain":"cybersecurity","path":"skills/performing-api-inventory-and-discovery"},{"name":"performing-api-rate-limiting-bypass","description":"Tests API rate limiting for bypass vulnerabilities using Python (requests/aiohttp) and Burp Suite Turbo Intruder to manipulate headers (e.g. X-Forwarded-For spoofing), IPs, HTTP methods, API versions, and encodings, mapping findings to OWASP API4:2023 Unrestricted Resource Consumption. Use when assessing, under written authorization, whether rate limits can be bypassed to enable brute force or resource-exhaustion attacks.","domain":"cybersecurity","path":"skills/performing-api-rate-limiting-bypass"},{"name":"performing-api-security-testing-with-postman","description":"Uses Postman to build structured API security test collections covering the OWASP API Security Top 10\u2014authentication bypass, authorization flaws, injection, and data exposure\u2014with multi-role environments, automated test scripts, and OWASP ZAP/Newman integration for CI/CD. Use when building repeatable Postman-based API security regression tests or automating OWASP API Top 10 coverage in a pipeline.","domain":"cybersecurity","path":"skills/performing-api-security-testing-with-postman"},{"name":"performing-arp-spoofing-attack-simulation","description":"Simulates ARP spoofing/cache-poisoning attacks in authorized lab or pentest environments using arpspoof, Ettercap, and Scapy to demonstrate man-in-the-middle risk and validate Dynamic ARP Inspection, port security, and network monitoring detections. Use when testing whether switches, IDS/IPS, or a SIEM detect ARP spoofing under written authorization; do not use on production networks without explicit approval.","domain":"cybersecurity","path":"skills/performing-arp-spoofing-attack-simulation"},{"name":"performing-asset-criticality-scoring-for-vulns","description":"Build a multi-factor asset criticality scoring model\u2014incorporating data sensitivity, business function dependency, regulatory scope, network exposure, and recoverability\u2014to produce a 1-5 criticality tier that weights vulnerability prioritization and remediation SLAs. Use when prioritizing vulnerability remediation by business impact or aligning CMDB asset data with risk-based patching timelines.","domain":"cybersecurity","path":"skills/performing-asset-criticality-scoring-for-vulns"},{"name":"performing-authenticated-scan-with-openvas","description":"Configure and execute authenticated (credentialed) vulnerability scans using OpenVAS/Greenbone Vulnerability Management (GVM) with SSH, SMB, or ESXi credentials to detect local vulnerabilities, missing patches, and misconfigurations on target hosts. Use when a host-level scan needs to log into systems for deep inspection, since authenticated scans typically surface 10-50x more findings than unauthenticated scans.","domain":"cybersecurity","path":"skills/performing-authenticated-scan-with-openvas"},{"name":"performing-authenticated-vulnerability-scan","description":"Plan and run authenticated (credentialed) vulnerability scans with scanners such as Nessus, Qualys, OpenVAS, or Rapid7 InsightVM, using SSH, SMB, WinRM, or SNMPv3 credentials to inspect installed software, patches, and configurations on Linux, Windows, and network devices. Use when a scan must catch vulnerabilities unauthenticated scanning misses, or when choosing and managing credential types for a credentialed scan.","domain":"cybersecurity","path":"skills/performing-authenticated-vulnerability-scan"},{"name":"performing-automated-malware-analysis-with-cape","description":"Deploy and operate the CAPEv2 malware sandbox (a Cuckoo derivative) to run samples in a monitored Windows guest VM, capturing behavioral signatures, dropped files, PCAP network traffic, and family-specific configuration extraction (e.g. Emotet, TrickBot, Cobalt Strike) via cape-parsers. Use when a suspicious file or payload needs automated dynamic analysis, anti-evasion debugger tricks, or config/payload extraction.","domain":"cybersecurity","path":"skills/performing-automated-malware-analysis-with-cape"},{"name":"performing-aws-account-enumeration-with-scout-suite","description":"Run the agentless, open-source ScoutSuite tool (via pip install and the `scout` CLI) against an AWS account to enumerate resources across services, identify misconfigurations, and generate an interactive HTML security report. Use when assessing an AWS account's overall security posture with read-only IAM credentials, such as during a cloud security audit or compliance review.","domain":"cybersecurity","path":"skills/performing-aws-account-enumeration-with-scout-suite"},{"name":"performing-aws-privilege-escalation-assessment","description":"Performing authorized privilege escalation assessments in AWS environments to identify IAM misconfigurations that allow users or roles to elevate their permissions using Pacu, CloudFox, Principal Mapper, and manual IAM policy analysis techniques.","domain":"cybersecurity","path":"skills/performing-aws-privilege-escalation-assessment"},{"name":"performing-bandwidth-throttling-attack-simulation","description":"Simulate bandwidth throttling and network degradation attacks using tc, iperf3, and Scapy in authorized lab environments to test QoS controls, application resilience, and monitoring detection of traffic manipulation. Use when validating how VoIP, video, or other real-time applications and network monitoring tools respond to degraded bandwidth or slowloris-style throttling attacks.","domain":"cybersecurity","path":"skills/performing-bandwidth-throttling-attack-simulation"},{"name":"performing-binary-exploitation-analysis","description":"Analyze ELF binaries for memory-corruption vulnerabilities and build proof-of-concept exploits using pwntools, checksec, and ROPgadget for buffer overflows and ROP chains. Use when a penetration test or CTF challenge requires evaluating compiler mitigations (NX, ASLR, stack canaries, PIE, RELRO) or developing a working exploit to demonstrate impact.","domain":"cybersecurity","path":"skills/performing-binary-exploitation-analysis"},{"name":"performing-blind-ssrf-exploitation","description":"Detect and exploit blind Server-Side Request Forgery (SSRF) using out-of-band techniques such as Burp Collaborator DNS interactions and timing analysis, to reach internal services and cloud metadata endpoints even when server responses are not reflected. Use when testing URL/webhook parameters, PDF generators, image processors, or import/preview features where SSRF output cannot be observed directly.","domain":"cybersecurity","path":"skills/performing-blind-ssrf-exploitation"},{"name":"performing-bluetooth-security-assessment","description":"Assess Bluetooth Low Energy (BLE) device security using Python's bleak asyncio library to discover nearby devices, enumerate GATT services and characteristics, and flag unencrypted or unauthenticated read/write access to sensitive data. Use when auditing IoT, healthcare, fitness, or smart-home BLE devices for weak pairing configurations or known vulnerable device fingerprints.","domain":"cybersecurity","path":"skills/performing-bluetooth-security-assessment"},{"name":"performing-brand-monitoring-for-impersonation","description":"Monitor for brand impersonation attacks across domains, social media, mobile apps, and dark web channels to detect phishing campaigns, fake sites, and unauthorized brand usage targeting your organization.","domain":"cybersecurity","path":"skills/performing-brand-monitoring-for-impersonation"},{"name":"performing-clickjacking-attack-test","description":"Testing web applications for clickjacking vulnerabilities by assessing frame embedding controls and crafting proof-of-concept overlay attacks during authorized security assessments.","domain":"cybersecurity","path":"skills/performing-clickjacking-attack-test"},{"name":"performing-cloud-asset-inventory-with-cartography","description":"Run Cartography to sync AWS, GCP, or Azure resources into a Neo4j graph database, mapping relationships such as IAM permission chains, network paths, and cross-account trust. Use when building a cloud asset inventory, querying the graph to identify attack paths, or generating security reports across multi-cloud infrastructure.","domain":"cybersecurity","path":"skills/performing-cloud-asset-inventory-with-cartography"},{"name":"performing-cloud-forensics-investigation","description":"Collect and analyze cloud forensic evidence using AWS CLI, Azure CLI, or gcloud to snapshot volumes, capture instance metadata and security group configurations, and preserve cloud-native logs (CloudTrail, Activity Log, Audit Log). Use when investigating a suspected breach in AWS, Azure, or GCP, tracing unauthorized access through API logs, or analyzing a compromised VM, container, or serverless function.","domain":"cybersecurity","path":"skills/performing-cloud-forensics-investigation"},{"name":"performing-cloud-forensics-with-aws-cloudtrail","description":"Investigate AWS account compromise by querying CloudTrail with boto3's LookupEvents or AWS Athena SQL over S3-delivered logs, filtering on suspicious user agents, source IPs, and event names to reconstruct an attacker timeline. Use when tracing unauthorized API calls, S3 data exfiltration, IAM privilege escalation, or credential exposure, and building a forensic report of findings and remediation steps.","domain":"cybersecurity","path":"skills/performing-cloud-forensics-with-aws-cloudtrail"},{"name":"performing-cloud-incident-containment-procedures","description":"Execute cloud-native incident containment across AWS, Azure, and GCP using platform CLIs to revoke or disable compromised IAM credentials, isolate resources with security groups and network ACLs, and preserve forensic evidence via snapshots. Use when responding to a cloud security incident that requires stopping lateral movement while keeping evidence intact for later investigation.","domain":"cybersecurity","path":"skills/performing-cloud-incident-containment-procedures"},{"name":"performing-cloud-log-forensics-with-athena","description":"Uses AWS Athena to query CloudTrail, VPC Flow Logs, S3 access logs, and ALB logs for forensic investigation. Covers CREATE TABLE DDL with partition projection, forensic SQL queries for detecting unauthorized access, data exfiltration, lateral movement, and privilege escalation. Use when investigating AWS security incidents or building cloud-native forensic workflows at scale.","domain":"cybersecurity","path":"skills/performing-cloud-log-forensics-with-athena"},{"name":"performing-cloud-native-forensics-with-falco","description":"Uses Falco YAML rules for runtime threat detection in containers and Kubernetes, monitoring syscalls for shell spawns, file tampering, network anomalies, and privilege escalation. Manages Falco rules via the Falco gRPC API and parses Falco alert output. Use when building container runtime security or investigating k8s cluster compromises.","domain":"cybersecurity","path":"skills/performing-cloud-native-forensics-with-falco"},{"name":"performing-cloud-native-threat-hunting-with-aws-detective","description":"Investigate AWS security incidents using Amazon Detective's behavior graphs, built from CloudTrail, VPC Flow Logs, GuardDuty, and EKS audit logs, to trace entity timelines and profile IAM users, roles, EC2 instances, and IP addresses for lateral movement. Use when triaging GuardDuty findings, investigating a suspected AWS compromise, or reconstructing an attacker's activity timeline across AWS accounts.","domain":"cybersecurity","path":"skills/performing-cloud-native-threat-hunting-with-aws-detective"},{"name":"performing-cloud-penetration-testing-with-pacu","description":"Run authorized AWS penetration tests with Pacu, the open-source AWS exploitation framework, to enumerate IAM configuration, scan for privilege escalation paths, and test data access and lateral movement using compromised credentials. Use when conducting authorized red-team assessments of AWS environments, validating IAM policies and SCPs, or determining the blast radius of compromised AWS credentials.","domain":"cybersecurity","path":"skills/performing-cloud-penetration-testing-with-pacu"},{"name":"performing-cloud-storage-forensic-acquisition","description":"Perform forensic acquisition of cloud storage services including Google Drive, OneDrive, Dropbox, and Box by pulling API-based remote data such as revision history and audit logs, and collecting local sync-client artifacts including KAPE targets and OneDrive databases from endpoints. Use during incident response or e-discovery when evidence resides in cloud-synced storage and both cloud-side and endpoint-side artifacts must be preserved.","domain":"cybersecurity","path":"skills/performing-cloud-storage-forensic-acquisition"},{"name":"performing-container-escape-detection","description":"Audits container and pod configuration for escape-enabling misconfiguration using the Kubernetes Python client - privileged flags, dangerous capability grants, host path mounts, shared namespaces, and CVE-2022-0492 style cgroup abuse. Use when sweeping a cluster for workloads that could break out, producing a posture report, or checking configuration before enforcement is switched on. Keywords: privileged, hostPath, hostPID, capabilities, CVE-2022-0492, cgroup, kubernetes python client, posture audit. Do not use for runtime syscall-based detection - use detecting-container-escape-attempts. '","domain":"cybersecurity","path":"skills/performing-container-escape-detection"},{"name":"performing-container-image-hardening","description":"Harden container images by minimizing attack surface, stripping unnecessary packages, implementing multi-stage builds, configuring non-root users, and applying CIS Docker Benchmark recommendations to produce secure, production-ready images. Use when building production container images, when compliance requires CIS Docker Benchmark adherence, or when shrinking image size to reduce vulnerability exposure from unused packages.","domain":"cybersecurity","path":"skills/performing-container-image-hardening"},{"name":"performing-container-security-scanning-with-trivy","description":"Runs Trivy across every target type it supports - container images, filesystems, Git repositories, and Kubernetes clusters - for OS and dependency vulnerabilities, IaC misconfiguration, exposed secrets, and licences, generating CycloneDX or SPDX SBOMs. Use when integrating Trivy into CI/CD, deploying the Trivy Kubernetes operator, scanning non-image targets, or triaging results at scale. Keywords: Trivy, trivy k8s, operator, SBOM, CycloneDX, SPDX, misconfig, secret scanning. Do not use for a single Docker image scan - use scanning-docker-images-with-trivy.","domain":"cybersecurity","path":"skills/performing-container-security-scanning-with-trivy"},{"name":"performing-content-security-policy-bypass","description":"Analyze Content-Security-Policy headers and bypass them to achieve cross-site scripting by exploiting unsafe-inline/unsafe-eval, whitelisted JSONP endpoints, base-uri and form-action gaps, and nonce/hash weaknesses, then exfiltrate data even without script-src control. Use during web application security assessments or bug bounty hunting when XSS is found but blocked by CSP, or when auditing CSP header configuration for weaknesses.","domain":"cybersecurity","path":"skills/performing-content-security-policy-bypass"},{"name":"performing-credential-access-with-lazagne","description":"Extract stored credentials from compromised endpoints using the LaZagne post-exploitation tool to recover passwords from browsers, databases, system vaults, and applications during authorized red team operations.","domain":"cybersecurity","path":"skills/performing-credential-access-with-lazagne"},{"name":"performing-cryptographic-audit-of-application","description":"A cryptographic audit systematically reviews an application's use of cryptographic primitives, protocols, and key management to identify vulnerabilities such as weak algorithms, insecure modes, hardco","domain":"cybersecurity","path":"skills/performing-cryptographic-audit-of-application"},{"name":"performing-csrf-attack-simulation","description":"Testing web applications for Cross-Site Request Forgery vulnerabilities by crafting forged requests that exploit authenticated user sessions during authorized security assessments.","domain":"cybersecurity","path":"skills/performing-csrf-attack-simulation"},{"name":"performing-cve-prioritization-with-kev-catalog","description":"Fetch and parse the CISA Known Exploited Vulnerabilities (KEV) catalog, enrich it with EPSS scores and CVSS metrics, and build a multi-factor prioritization engine and report that ranks CVE remediation by real-world exploitation evidence and BOD 22-01 deadlines. Use when triaging a vulnerability backlog, deciding patch order across many CVEs, or building an automated KEV+EPSS prioritization workflow.","domain":"cybersecurity","path":"skills/performing-cve-prioritization-with-kev-catalog"},{"name":"performing-dark-web-monitoring-for-threats","description":"Dark web monitoring involves systematically scanning Tor hidden services, underground forums, paste sites, and dark web marketplaces to identify threats targeting an organization, including leaked cre","domain":"cybersecurity","path":"skills/performing-dark-web-monitoring-for-threats"},{"name":"performing-deception-technology-deployment","description":"Deploys deception technology including honeypots, honeytokens, and decoy systems to detect attackers who have bypassed perimeter defenses, providing high-fidelity alerts with near-zero false positive rates. Use when SOC teams need early warning of lateral movement, credential abuse, or internal reconnaissance by deploying convincing traps across the network.","domain":"cybersecurity","path":"skills/performing-deception-technology-deployment"},{"name":"performing-directory-traversal-testing","description":"Test web applications for path traversal and Local/Remote File Inclusion vulnerabilities by manipulating file path parameters, applying encoding and filter-bypass techniques, automating discovery with ffuf and dotdotpwn, and reading high-value files or achieving code execution. Use during authorized penetration tests of file download, view, or include functionality, or when assessing APIs that accept file names or file paths as parameters.","domain":"cybersecurity","path":"skills/performing-directory-traversal-testing"},{"name":"performing-disk-forensics-investigation","description":"Conduct disk forensics investigations using forensic imaging, file system analysis, and timeline reconstruction, with tools such as FTK Imager, Autopsy, and The Sleuth Kit, for evidence acquisition, deleted file recovery, and artifact examination. Use when a security incident requires forensic analysis of persistent storage or when evidence must be preserved for legal or HR proceedings.","domain":"cybersecurity","path":"skills/performing-disk-forensics-investigation"},{"name":"performing-dmarc-policy-enforcement-rollout","description":"Execute a phased DMARC rollout by inventorying sending sources, configuring SPF/DKIM alignment, and progressing DNS policy from p=none monitoring through p=quarantine to p=reject enforcement, ensuring all legitimate email sources authenticate before unauthorized senders are blocked. Use when deploying or advancing an organization's DMARC anti-spoofing posture, or when meeting bulk-sender authentication requirements from Google and Yahoo.","domain":"cybersecurity","path":"skills/performing-dmarc-policy-enforcement-rollout"},{"name":"performing-dns-enumeration-and-zone-transfer","description":"Enumerates DNS records, attempts zone transfers, brute-forces subdomains, and maps DNS infrastructure during authorized reconnaissance to identify attack surface, misconfigurations, and information disclosure in target domains.","domain":"cybersecurity","path":"skills/performing-dns-enumeration-and-zone-transfer"},{"name":"performing-dns-tunneling-detection","description":"Detects DNS tunneling by computing Shannon entropy of DNS query names, analyzing query length distributions, inspecting TXT record payloads, and identifying high subdomain cardinality. Uses scapy for packet capture analysis and statistical methods to distinguish legitimate DNS from covert channels. Use when hunting for data exfiltration.","domain":"cybersecurity","path":"skills/performing-dns-tunneling-detection"},{"name":"performing-docker-bench-security-assessment","description":"Runs Docker Bench for Security, the open-source CIS Docker Benchmark audit script, across host configuration, daemon settings, images, and runtime configuration, then interprets pass/fail/warn output and remediates the common failures. Use when auditing Docker hosts for CIS compliance, scheduling recurring container assessments, or validating runtime hardening controls after a change. Keywords: docker-bench-security, CIS Docker Benchmark, audit script, pass fail warn, host configuration, remediation. Do not use for applying the daemon hardening itself - use hardening-docker-daemon-configuration.","domain":"cybersecurity","path":"skills/performing-docker-bench-security-assessment"},{"name":"performing-dynamic-analysis-of-android-app","description":"Performs runtime dynamic analysis of Android applications using Frida, Objection, and Android Debug Bridge to observe application behavior during execution, intercept function calls, modify runtime values, and identify vulnerabilities that static analysis misses. Use when testing Android apps for runtime security flaws, hooking sensitive methods, bypassing client-side protections, or analyzing obfuscated applications. Activates for requests involving Android dynamic analysis, runtime hooking, Frida Android instrumentation, or live app behavior analysis.","domain":"cybersecurity","path":"skills/performing-dynamic-analysis-of-android-app"},{"name":"performing-dynamic-analysis-with-any-run","description":"Perform interactive dynamic malware analysis using the ANY.RUN cloud sandbox to detonate samples, observe real-time execution behavior, interact with malware prompts such as dialogs and CAPTCHAs, and capture process trees, network traffic, and system changes. Use when a suspicious file or URL needs live, interactive behavioral detonation in a cloud sandbox rather than static analysis alone.","domain":"cybersecurity","path":"skills/performing-dynamic-analysis-with-any-run"},{"name":"performing-endpoint-forensics-investigation","description":"Performs digital forensics investigation on compromised endpoints including memory acquisition, disk imaging, artifact analysis, and timeline reconstruction. Use when investigating security incidents, collecting evidence for legal proceedings, or analyzing endpoint compromise scope. Activates for requests involving endpoint forensics, memory analysis, disk forensics, or incident investigation.","domain":"cybersecurity","path":"skills/performing-endpoint-forensics-investigation"},{"name":"performing-endpoint-vulnerability-remediation","description":"Performs vulnerability remediation on endpoints by prioritizing CVEs based on risk scoring, deploying patches, applying configuration changes, and validating fixes. Use when remediating findings from vulnerability scans, responding to critical CVE advisories, or maintaining endpoint compliance with patch management SLAs. Activates for requests involving vulnerability remediation, CVE patching, endpoint vulnerability management, or security fix deployment.","domain":"cybersecurity","path":"skills/performing-endpoint-vulnerability-remediation"},{"name":"performing-entitlement-review-with-sailpoint-iiq","description":"Runs entitlement review and access certification campaigns in SailPoint IdentityIQ, covering manager certifications, targeted entitlement reviews, role-based access validation, segregation-of-duties violation remediation, and automated revocation workflows. Use when performing periodic user access recertification, auditing SailPoint IIQ access governance, or investigating SOD violations.","domain":"cybersecurity","path":"skills/performing-entitlement-review-with-sailpoint-iiq"},{"name":"performing-external-network-penetration-test","description":"Conduct a comprehensive external network penetration test to identify vulnerabilities in internet-facing infrastructure using PTES methodology, reconnaissance, scanning, exploitation, and reporting.","domain":"cybersecurity","path":"skills/performing-external-network-penetration-test"},{"name":"performing-false-positive-reduction-in-siem","description":"Reduces SIEM false positives through systematic rule tuning, threshold adjustment, correlation logic refinement, allowlisting, and threat intelligence enrichment. Use when SOC analysts are overwhelmed by alert noise, when tuning noisy detection rules, or during a quarterly SIEM rule review to cut alert fatigue.","domain":"cybersecurity","path":"skills/performing-false-positive-reduction-in-siem"},{"name":"performing-file-carving-with-foremost","description":"Recovers files from disk images and unallocated space using Foremost's header-footer signature carving, extracting evidence independent of the file system's state. Use during digital forensics investigations to carve deleted or fragmented files, such as documents, images, and archives, from raw disk images or unallocated space.","domain":"cybersecurity","path":"skills/performing-file-carving-with-foremost"},{"name":"performing-firmware-extraction-with-binwalk","description":"Performs firmware image extraction and analysis using binwalk to identify embedded filesystems, compressed archives, bootloaders, kernel images, and cryptographic material. Covers entropy analysis for detecting encrypted or compressed regions, recursive extraction of nested archives, SquashFS/CramFS/JFFS2 filesystem mounting, and string analysis for credential and configuration discovery. Activates for requests involving firmware reverse engineering, IoT device analysis, embedded system security assessment, or router/camera firmware extraction.","domain":"cybersecurity","path":"skills/performing-firmware-extraction-with-binwalk"},{"name":"performing-firmware-malware-analysis","description":"Analyzes firmware images for embedded malware, backdoors, and unauthorized modifications in routers, IoT devices, UEFI/BIOS, and embedded systems, covering firmware extraction, filesystem analysis, binary reverse engineering, and bootkit detection. Use for firmware security analysis, IoT malware investigation, UEFI rootkit detection, or embedded device compromise assessment.","domain":"cybersecurity","path":"skills/performing-firmware-malware-analysis"},{"name":"performing-fuzzing-with-aflplusplus","description":"Performs coverage-guided fuzzing of compiled binaries with AFL++, instrumenting targets via afl-cc/afl-clang-fast, minimizing corpora with afl-cmin and afl-tmin, running parallel campaigns with afl-fuzz, and triaging crashes with CASR or GDB scripts. Use for binary fuzzing, crash and memory-corruption discovery, coverage-guided testing, or running AFL++ fuzzing campaigns.","domain":"cybersecurity","path":"skills/performing-fuzzing-with-aflplusplus"},{"name":"performing-gcp-penetration-testing-with-gcpbucketbrute","description":"Performs authorized GCP security testing using GCPBucketBrute to enumerate publicly accessible storage buckets, combined with gcloud CLI IAM enumeration to find privilege escalation paths and audit service account permissions. Use when penetration testing a GCP project for exposed buckets, overly permissive IAM bindings, or service account key exposure.","domain":"cybersecurity","path":"skills/performing-gcp-penetration-testing-with-gcpbucketbrute"},{"name":"performing-gcp-security-assessment-with-forseti","description":"Performing comprehensive security assessments of Google Cloud Platform environments using Forseti Security, Security Command Center, and gcloud CLI to audit IAM policies, firewall rules, storage permissions, and compliance against CIS GCP Foundations Benchmark.","domain":"cybersecurity","path":"skills/performing-gcp-security-assessment-with-forseti"},{"name":"performing-graphql-depth-limit-attack","description":"Execute and test GraphQL depth limit attacks using deeply nested recursive queries to identify denial-of-service vulnerabilities in GraphQL APIs.","domain":"cybersecurity","path":"skills/performing-graphql-depth-limit-attack"},{"name":"performing-graphql-introspection-attack","description":"Performs GraphQL introspection attacks that extract the full API schema (types, queries, mutations, subscriptions, field definitions), map the attack surface, test query depth/complexity limits, and exploit GraphQL-specific weaknesses such as batching attacks, alias-based brute force, and nested query DoS. Use for GraphQL security testing, schema enumeration, or GraphQL API penetration testing.","domain":"cybersecurity","path":"skills/performing-graphql-introspection-attack"},{"name":"performing-graphql-security-assessment","description":"Assessing GraphQL API endpoints for introspection leaks, injection attacks, authorization flaws, and denial-of-service vulnerabilities during authorized security tests.","domain":"cybersecurity","path":"skills/performing-graphql-security-assessment"},{"name":"performing-hardware-security-module-integration","description":"Integrates Hardware Security Modules (HSMs) via the PKCS#11 interface using python-pkcs11, performing key generation, signing, encryption, verification, and token/slot queries against SoftHSM2, AWS CloudHSM, or YubiHSM2. Use when implementing HSM-backed key management or validating HSM configuration for FIPS 140-2/3 compliance.","domain":"cybersecurity","path":"skills/performing-hardware-security-module-integration"},{"name":"performing-hash-cracking-with-hashcat","description":"Cracks password hashes with Hashcat, covering hash-type identification, dictionary/brute-force/rule-based attack modes, custom rule creation, GPU benchmarking, and password-strength/compliance reporting. Use for authorized penetration testing or security audits that need to evaluate password strength or crack captured hashes.","domain":"cybersecurity","path":"skills/performing-hash-cracking-with-hashcat"},{"name":"performing-http-parameter-pollution-attack","description":"Executes HTTP Parameter Pollution attacks that inject duplicate request parameters to bypass input validation, WAF rules, and other security controls when front-end and back-end systems parse duplicate parameters differently. Use during web application penetration testing to test for parameter parsing inconsistencies or WAF/validation bypass.","domain":"cybersecurity","path":"skills/performing-http-parameter-pollution-attack"},{"name":"performing-ics-asset-discovery-with-claroty","description":"Performs ICS/OT asset discovery with Claroty xDome, combining passive monitoring and Claroty Edge active queries to inventory PLCs, RTUs, HMIs, and network infrastructure across Purdue Model levels. Use when gaining visibility into an undocumented OT environment, preparing an IEC 62443 asset inventory, or onboarding Claroty xDome; not for IT-only discovery.","domain":"cybersecurity","path":"skills/performing-ics-asset-discovery-with-claroty"},{"name":"performing-indicator-lifecycle-management","description":"Tracks IOCs through discovery, enrichment/validation (VirusTotal, Shodan, passive DNS), deployment to SIEM/IDS watchlists, hit-rate and false-positive monitoring, confidence-score decay, and automated expiration using MISP/OpenCTI and STIX. Use when building or maintaining a threat intelligence indicator lifecycle process, aging out stale IOCs, or reducing analyst fatigue from low-quality indicators.","domain":"cybersecurity","path":"skills/performing-indicator-lifecycle-management"},{"name":"performing-initial-access-with-evilginx3","description":"Perform authorized initial access using EvilGinx3 adversary-in-the-middle phishing framework to capture session tokens and bypass multi-factor authentication during red team engagements.","domain":"cybersecurity","path":"skills/performing-initial-access-with-evilginx3"},{"name":"performing-insider-threat-investigation","description":"Investigates insider threat incidents involving employees, contractors, or trusted partners who misuse authorized access to steal data, sabotage systems, or violate security policies, combining digital forensics, user behavior analytics, and HR/legal coordination to build an evidence-based case. Use when DLP alerts flag large data transfers to personal cloud storage or USB devices, when UBA detects anomalous access patterns for a user account, or when investigating employee data theft, privilege misuse, or internal threat detection requests.","domain":"cybersecurity","path":"skills/performing-insider-threat-investigation"},{"name":"performing-ioc-enrichment-automation","description":"Automates Indicator of Compromise (IOC) enrichment by orchestrating lookups across VirusTotal, AbuseIPDB, Shodan, MISP, and other intelligence sources to provide contextual scoring and disposition recommendations. Use when SOC analysts need rapid multi-source enrichment of IPs, domains, URLs, and file hashes during alert triage or incident investigation.","domain":"cybersecurity","path":"skills/performing-ioc-enrichment-automation"},{"name":"performing-ios-app-security-assessment","description":"Performs comprehensive iOS application security assessments using Frida for dynamic instrumentation, Objection for runtime exploration, SSL pinning bypass for traffic interception, keychain extraction for credential analysis, and IPA static analysis for binary-level review. Use when conducting authorized iOS penetration tests, evaluating mobile app security posture against OWASP MASTG, or assessing iOS app data protection and transport security controls. Activates for requests involving iOS app pentesting, Frida-based iOS instrumentation, mobile app SSL pinning bypass, or IPA reverse engineering.","domain":"cybersecurity","path":"skills/performing-ios-app-security-assessment"},{"name":"performing-iot-security-assessment","description":"Performs comprehensive security assessments of IoT devices and their ecosystems by testing hardware interfaces, firmware, network communications, cloud APIs, and companion mobile applications. The tester uses firmware extraction and analysis, hardware debugging via UART and JTAG, network protocol analysis, and runtime exploitation to identify vulnerabilities across all layers of the IoT stack. Activates for requests involving IoT security testing, embedded device assessment, firmware security analysis, or smart device penetration testing.","domain":"cybersecurity","path":"skills/performing-iot-security-assessment"},{"name":"performing-ip-reputation-analysis-with-shodan","description":"Analyze IP address reputation using the Shodan API to identify open ports, running services, known vulnerabilities, and hosting context for threat intelligence enrichment and incident triage.","domain":"cybersecurity","path":"skills/performing-ip-reputation-analysis-with-shodan"},{"name":"performing-jwt-none-algorithm-attack","description":"Execute and test the JWT none algorithm attack, crafting tokens with the alg header set to none using PyJWT and an intercepting proxy (Burp Suite/mitmproxy) to bypass signature verification and forge arbitrary claims. Use during authorized penetration tests or security assessments of applications that use JWT for authentication or authorization, to validate that the server rejects unsigned tokens.","domain":"cybersecurity","path":"skills/performing-jwt-none-algorithm-attack"},{"name":"performing-kerberoasting-attack","description":"Perform Kerberoasting, a post-exploitation technique that enumerates Active Directory service accounts with Service Principal Names (SPNs), requests their Kerberos TGS tickets, and cracks the NTLM-encrypted tickets offline to recover service account credentials. Use during authorized red team engagements or penetration tests to assess AD privilege escalation and credential access risk from weak service account passwords.","domain":"cybersecurity","path":"skills/performing-kerberoasting-attack"},{"name":"performing-kubernetes-cis-benchmark-with-kube-bench","description":"Turns kube-bench output into a finished CIS Kubernetes Benchmark audit: interpreting PASS/FAIL/WARN per control, judging which failures are genuine on a managed cluster, writing remediation, and packaging evidence for SOC 2 or PCI DSS. Use when conducting a scheduled compliance audit, triaging kube-bench results, deciding which controls are not applicable on EKS, GKE, or AKS, or producing hardening evidence for an auditor. Keywords: CIS Kubernetes Benchmark, control plane, remediation, compliance evidence, SOC 2, PCI DSS, managed cluster exception. Do not use for installing and running the tool - use benchmarking-kubernetes-with-kube-bench.","domain":"cybersecurity","path":"skills/performing-kubernetes-cis-benchmark-with-kube-bench"},{"name":"performing-kubernetes-etcd-security-assessment","description":"Assesses the security posture of the etcd cluster backing Kubernetes: encryption at rest, TLS peer and client transport, access control, backup encryption, and network isolation. Use when auditing or hardening a control plane, reviewing whether Secrets are encrypted at rest, or protecting etcd backups, since etcd stores Secrets, RBAC policy, and ConfigMaps in plaintext by default. Keywords: etcd, EncryptionConfiguration, encryption at rest, peer TLS, snapshot, backup, control plane. Do not use for broad cluster-wide CIS checks - use performing-kubernetes-cis-benchmark-with-kube-bench.","domain":"cybersecurity","path":"skills/performing-kubernetes-etcd-security-assessment"},{"name":"performing-kubernetes-penetration-testing","description":"Evaluates Kubernetes cluster security by actively simulating attacker techniques against the API server, kubelet, etcd, pods, RBAC, network policy, and secrets, using kube-hunter, Kubescape, peirates, and manual kubectl exploitation to find paths to cluster compromise. Use for an authorized penetration test or hands-on validation that controls actually stop an attacker. Keywords: kube-hunter, Kubescape, peirates, kubelet 10250, anonymous auth, token theft, lateral movement, cluster takeover. Do not use for a configuration-only compliance audit - use performing-kubernetes-cis-benchmark-with-kube-bench.","domain":"cybersecurity","path":"skills/performing-kubernetes-penetration-testing"},{"name":"performing-lateral-movement-detection","description":"Detects lateral movement techniques including Pass-the-Hash, PsExec, WMI execution, RDP pivoting, and SMB-based spreading by correlating Windows Security/Sysmon event logs, network flow data (NetFlow/Zeek), and endpoint telemetry in a SIEM, mapped to MITRE ATT&CK Lateral Movement (TA0008) techniques with sample SPL detection queries. Use when a SOC team needs to detect attackers pivoting between internal systems after initial compromise, trace an attacker's movement path during an incident investigation, or build detection engineering rules for TA0008; not for detecting initial access or external attacks.","domain":"cybersecurity","path":"skills/performing-lateral-movement-detection"},{"name":"performing-lateral-movement-with-wmiexec","description":"Perform lateral movement across Windows networks using WMI-based remote execution techniques including Impacket wmiexec.py, CrackMapExec, and native WMI commands for stealthy post-exploitation during red team engagements.","domain":"cybersecurity","path":"skills/performing-lateral-movement-with-wmiexec"},{"name":"performing-linux-log-forensics-investigation","description":"Perform forensic investigation of Linux system logs including syslog, auth.log, systemd journal (via journalctl), kern.log, auditd, and application logs to reconstruct user sessions, identify unauthorized access and privilege escalation, trace lateral movement, and establish event timelines. Use when investigating a suspected compromise of a Linux system and needing to analyze SSH, sudo, cron, or kernel-level activity from plain-text or systemd journal logs.","domain":"cybersecurity","path":"skills/performing-linux-log-forensics-investigation"},{"name":"performing-log-analysis-for-forensic-investigation","description":"Collect, parse, and correlate system, application, and security logs to reconstruct events and establish timelines during forensic investigations.","domain":"cybersecurity","path":"skills/performing-log-analysis-for-forensic-investigation"},{"name":"performing-log-source-onboarding-in-siem","description":"Perform structured log source onboarding into SIEM platforms (Splunk, Elastic, Sentinel, QRadar, or similar) by prioritizing sources with a tiered value framework, configuring collectors, building parsers, normalizing fields to a common schema (e.g. CIM), and validating data quality. Use when planning or executing the integration of new log sources \u2014 such as Active Directory, firewalls, EDR, VPN, DNS, or cloud audit logs \u2014 into a SIEM for security monitoring and detection.","domain":"cybersecurity","path":"skills/performing-log-source-onboarding-in-siem"},{"name":"performing-malware-hash-enrichment-with-virustotal","description":"Enrich malware file hashes (MD5, SHA-1, SHA-256) using the VirusTotal API v3 to retrieve multi-engine detection rates, sandbox behavioral analysis, YARA rule matches, related indicators, and community threat intelligence. Use during SOC triage, incident response, or threat intelligence workflows to validate whether a file hash is malicious and gather context for IOC enrichment.","domain":"cybersecurity","path":"skills/performing-malware-hash-enrichment-with-virustotal"},{"name":"performing-malware-ioc-extraction","description":"Malware IOC extraction is the process of analyzing malicious software to identify actionable indicators of compromise including file hashes, network indicators (C2 domains, IP addresses, URLs), regist","domain":"cybersecurity","path":"skills/performing-malware-ioc-extraction"},{"name":"performing-malware-persistence-investigation","description":"Systematically investigate all persistence mechanisms on Windows and Linux systems to identify how malware survives reboots and maintains access.","domain":"cybersecurity","path":"skills/performing-malware-persistence-investigation"},{"name":"performing-malware-triage-with-yara","description":"Performs rapid malware triage and classification using YARA rules that match file patterns, strings, byte sequences, and structural characteristics against known malware families and suspicious indicators, covering rule writing, scanning, and integration into analysis pipelines. Use when classifying a batch of malware samples against known family signatures, writing detection rules for a newly analyzed malware family, or performing signature-based malware triage.","domain":"cybersecurity","path":"skills/performing-malware-triage-with-yara"},{"name":"performing-memory-forensics-with-volatility3","description":"Analyze volatile memory (RAM) dumps using the Volatility 3 framework to extract running processes, network connections, loaded modules, credentials, and encryption keys, and to detect process hollowing, DLL injection, or hidden processes/rootkits. Use during incident response on a compromised or suspect system when disk-based forensics alone is insufficient and volatile evidence of malware or intrusion must be recovered from memory.","domain":"cybersecurity","path":"skills/performing-memory-forensics-with-volatility3"},{"name":"performing-memory-forensics-with-volatility3-plugins","description":"Analyze memory dumps using Volatility3 plugins to detect injected code, rootkits, credential theft, and malware artifacts in Windows, Linux, and macOS memory images.","domain":"cybersecurity","path":"skills/performing-memory-forensics-with-volatility3-plugins"},{"name":"performing-mobile-app-certificate-pinning-bypass","description":"Bypasses SSL/TLS certificate pinning implementations in Android and iOS applications to enable traffic interception during authorized security assessments. Covers OkHttp, TrustManager, NSURLSession, and third-party pinning library bypass techniques using Frida, Objection, and custom scripts. Activates for requests involving certificate pinning bypass, SSL pinning defeat, mobile TLS interception, or proxy-resistant app testing.","domain":"cybersecurity","path":"skills/performing-mobile-app-certificate-pinning-bypass"},{"name":"performing-mobile-device-forensics-with-cellebrite","description":"Acquire and analyze mobile device data using Cellebrite UFED Touch/4PC, UFED Physical Analyzer, and open-source alternatives (ALEAPP, iLEAPP, MEAT, libimobiledevice) to extract communications, call logs, location data, and application artifacts. Use when extracting or recovering deleted evidence from smartphones or tablets during criminal, corporate, or employee-misuse investigations.","domain":"cybersecurity","path":"skills/performing-mobile-device-forensics-with-cellebrite"},{"name":"performing-network-forensics-with-wireshark","description":"Capture and analyze network traffic using Wireshark and tshark to reconstruct network events from PCAP/PCAPNG files, extract transferred files and credentials, and identify command-and-control communications. Use when analyzing captured traffic from a security incident, reconstructing data exfiltration, or finding network indicators of compromise during malware analysis.","domain":"cybersecurity","path":"skills/performing-network-forensics-with-wireshark"},{"name":"performing-network-packet-capture-analysis","description":"Perform forensic analysis of network packet captures (PCAP/PCAPNG) using Wireshark, tshark, and tcpdump to reconstruct network communications, extract transferred files, identify malicious traffic, and establish evidence of data exfiltration or command-and-control activity. Use when a PCAP file from an incident needs to be examined to prove lateral movement, malware delivery, or unauthorized access.","domain":"cybersecurity","path":"skills/performing-network-packet-capture-analysis"},{"name":"performing-network-traffic-analysis-with-tshark","description":"Automate network traffic analysis using tshark (Wireshark CLI) and pyshark to compute protocol distribution statistics, detect suspicious flows such as port scans and beaconing, extract IOCs (IPs, domains, URLs), and identify DNS tunneling patterns from PCAP files. Use when scripted or repeatable analysis of packet captures is needed rather than interactive inspection.","domain":"cybersecurity","path":"skills/performing-network-traffic-analysis-with-tshark"},{"name":"performing-network-traffic-analysis-with-zeek","description":"Deploy Zeek (formerly Bro) as a passive network security monitor to generate structured logs of protocol metadata (HTTP, DNS, TLS, SSH, SMTP, FTP, and more), write custom detection scripts, and integrate outputs with SIEM platforms. Use when standing up continuous, high-fidelity network traffic monitoring for threat detection, anomaly identification, or forensic investigation beyond what raw PCAP analysis provides.","domain":"cybersecurity","path":"skills/performing-network-traffic-analysis-with-zeek"},{"name":"performing-nist-csf-maturity-assessment","description":"Conduct a NIST Cybersecurity Framework (CSF) 2.0 maturity assessment across the six core Functions (Govern, Identify, Protect, Detect, Respond, Recover), scoring organizational posture against the four Implementation Tiers (Partial, Risk-Informed, Repeatable, Adaptive) and producing an improvement roadmap. Use when benchmarking an organization's cybersecurity program maturity or preparing a CSF-based gap analysis and remediation plan.","domain":"cybersecurity","path":"skills/performing-nist-csf-maturity-assessment"},{"name":"performing-oauth-scope-minimization-review","description":"Performs OAuth 2.0 scope minimization review to identify over-permissioned third-party application integrations, excessive API scopes, unused token grants, and risky OAuth consent patterns across identity providers and SaaS platforms. Activates for requests involving OAuth scope audit, API permission review, third-party app risk assessment, or consent grant minimization.","domain":"cybersecurity","path":"skills/performing-oauth-scope-minimization-review"},{"name":"performing-oil-gas-cybersecurity-assessment","description":"Conduct cybersecurity assessments of upstream, midstream, and downstream oil and gas operations, covering pipeline SCADA, refinery DCS, safety instrumented systems, and remote wellhead RTUs, and evaluate compliance with API 1164, TSA Pipeline Security Directives, and IEC 62443. Use when assessing a refinery, pipeline, or production facility or preparing for TSA/API compliance audits; not for IT-only or purely physical-security assessments.","domain":"cybersecurity","path":"skills/performing-oil-gas-cybersecurity-assessment"},{"name":"performing-open-source-intelligence-gathering","description":"Open Source Intelligence (OSINT) gathering is the first active phase of a red team engagement, where operators collect publicly available information about the target organization to identify attack s","domain":"cybersecurity","path":"skills/performing-open-source-intelligence-gathering"},{"name":"performing-osint-with-spiderfoot","description":"Automate OSINT collection with the SpiderFoot REST API and CLI (sf.py/spiderfoot-cli) across 200+ modules, selecting scan modes (footprint, investigate, passive) and parsing results for domains, IPs, emails, leaked credentials, and DNS records into a target intelligence profile. Use when mapping an organization's attack surface or profiling a target for threat intelligence.","domain":"cybersecurity","path":"skills/performing-osint-with-spiderfoot"},{"name":"performing-ot-network-security-assessment","description":"This skill covers conducting comprehensive security assessments of Operational Technology (OT) networks including SCADA systems, DCS architectures, and industrial control system communication paths. It addresses the Purdue Reference Model layers, identifies IT/OT convergence risks, evaluates firewall rules between zones, and maps industrial protocol traffic (Modbus, DNP3, OPC UA, EtherNet/IP) to detect misconfigurations, unauthorized connections, and attack surfaces in critical infrastructure.","domain":"cybersecurity","path":"skills/performing-ot-network-security-assessment"},{"name":"performing-ot-vulnerability-assessment-with-claroty","description":"Perform OT vulnerability assessments using the Claroty xDome platform for asset discovery, risk scoring, and vulnerability correlation, combining passive traffic-based identification and active safe device querying with CVE/ICS-CERT advisory correlation for remediation prioritization. Use for scheduled IEC 62443 or NERC CIP OT vulnerability assessments, initial xDome deployment, or generating CIP-010-4 compliance evidence; not for active PLC scanning or penetration testing.","domain":"cybersecurity","path":"skills/performing-ot-vulnerability-assessment-with-claroty"},{"name":"performing-ot-vulnerability-scanning-safely","description":"Perform vulnerability scanning in OT/ICS environments safely using passive monitoring, native protocol queries, and carefully controlled active scanning with Tenable OT Security to identify vulnerabilities without disrupting industrial processes or crashing legacy controllers.","domain":"cybersecurity","path":"skills/performing-ot-vulnerability-scanning-safely"},{"name":"performing-packet-injection-attack","description":"Crafts and injects custom network packets using Scapy, hping3, and Nemesis during authorized security assessments to test firewall rules, IDS detection, protocol handling, and network stack resilience against malformed and spoofed traffic.","domain":"cybersecurity","path":"skills/performing-packet-injection-attack"},{"name":"performing-paste-site-monitoring-for-credentials","description":"Monitor paste sites like Pastebin and GitHub Gists for leaked credentials, API keys, and sensitive data dumps using automated scraping and keyword matching to detect breaches early.","domain":"cybersecurity","path":"skills/performing-paste-site-monitoring-for-credentials"},{"name":"performing-phishing-simulation-with-gophish","description":"Deploy and run authorized phishing awareness campaigns with GoPhish, covering admin panel setup, SMTP sending profiles, email template and landing page creation, target user groups, and campaign reporting to measure click and credential-submission rates. Use when planning or executing a phishing simulation for employee security-awareness testing or measuring susceptibility to social engineering.","domain":"cybersecurity","path":"skills/performing-phishing-simulation-with-gophish"},{"name":"performing-physical-intrusion-assessment","description":"Conduct authorized physical penetration testing against facilities, server rooms, and restricted areas using tailgating, RFID badge cloning, lock bypassing, rogue network device deployment, and security-guard procedure testing. Use as part of a full-scope red team engagement to evaluate physical security controls and their path to network access, always under signed client authorization.","domain":"cybersecurity","path":"skills/performing-physical-intrusion-assessment"},{"name":"performing-plc-firmware-security-analysis","description":"This skill covers analyzing Programmable Logic Controller (PLC) firmware for security vulnerabilities including hardcoded credentials, insecure update mechanisms, backdoor functions, memory corruption flaws, and undocumented debug interfaces. It addresses firmware extraction from common PLC platforms (Siemens S7, Allen-Bradley, Schneider Modicon), static analysis of firmware images, dynamic analysis in emulated environments, and comparison against known-good baselines to detect tampering.","domain":"cybersecurity","path":"skills/performing-plc-firmware-security-analysis"},{"name":"performing-post-quantum-cryptography-migration","description":"Assesses organizational readiness for post-quantum cryptography migration per NIST FIPS 203/204/205 standards. Performs cryptographic inventory scanning to identify quantum-vulnerable algorithms (RSA, ECDH, ECDSA), evaluates hybrid TLS configurations with X25519MLKEM768, and validates CRYSTALS-Kyber (ML-KEM) and CRYSTALS-Dilithium (ML-DSA) readiness. Implements crypto-agility assessment using oqs-provider for OpenSSL. Use when planning or executing the transition from classical to post-quantum cryptographic algorithms across enterprise infrastructure.","domain":"cybersecurity","path":"skills/performing-post-quantum-cryptography-migration"},{"name":"performing-power-grid-cybersecurity-assessment","description":"Conduct cybersecurity assessments of power grid infrastructure spanning generation, transmission substations, distribution, and EMS control centers, covering NERC CIP compliance verification, IEC 61850 (GOOSE/MMS) substation protocol analysis, and synchrophasor (PMU) network security against threats like Industroyer/CrashOverride. Use for periodic NERC CIP assessments, substation automation or EMS/SCADA security reviews, or regional entity compliance audits; not for non-BES systems or generic OT assessments without power grid specifics.","domain":"cybersecurity","path":"skills/performing-power-grid-cybersecurity-assessment"},{"name":"performing-privacy-impact-assessment","description":"Automates the Privacy Impact Assessment (PIA) workflow including data flow mapping, privacy risk scoring matrices, GDPR Article 35 DPIA and CCPA/CPRA alignment checks, data inventory cataloging, and remediation tracking. Implements the NIST Privacy Framework PRAM methodology and ICO DPIA guidance for systematic identification and mitigation of privacy risks across processing activities. Use when conducting privacy assessments for new systems, evaluating regulatory compliance posture, or building automated privacy governance programs.","domain":"cybersecurity","path":"skills/performing-privacy-impact-assessment"},{"name":"performing-privilege-escalation-assessment","description":"Performs privilege escalation assessments on compromised Linux and Windows systems to identify paths from low-privilege access to root or SYSTEM-level control. The tester enumerates misconfigurations, vulnerable services, kernel exploits, SUID binaries, unquoted service paths, and credential stores to demonstrate the full impact of an initial compromise. Activates for requests involving privilege escalation testing, local exploitation, post-compromise escalation, or OS-level security assessment.","domain":"cybersecurity","path":"skills/performing-privilege-escalation-assessment"},{"name":"performing-privilege-escalation-on-linux","description":"Guides manual enumeration and automated tooling to escalate from a low-privilege Linux user to root by exploiting misconfigurations, vulnerable services, kernel exploits, and weak permissions, mapped to MITRE ATT&CK. Use during authorized red team engagements or penetration tests once initial low-privilege access on a Linux host is obtained and root-level access is needed.","domain":"cybersecurity","path":"skills/performing-privilege-escalation-on-linux"},{"name":"performing-privileged-account-access-review","description":"Conducts systematic reviews of privileged accounts to validate access rights, identify excessive or stale permissions, and enforce least privilege across PAM infrastructure. Use when auditing privileged access for compliance, periodic access recertification, or investigating whether privileged entitlements are still justified.","domain":"cybersecurity","path":"skills/performing-privileged-account-access-review"},{"name":"performing-privileged-account-discovery","description":"Discovers and inventories privileged accounts across enterprise infrastructure, including domain admins, local admins, service accounts, database admins, cloud IAM roles, and application admin accounts, using automated scanning and risk classification. Use when building a privileged account inventory, onboarding accounts to a PAM solution, or scoping which accounts need privileged-access controls.","domain":"cybersecurity","path":"skills/performing-privileged-account-discovery"},{"name":"performing-purple-team-atomic-testing","description":"Executes Atomic Red Team tests mapped to MITRE ATT&CK via Invoke-AtomicRedTeam PowerShell, generates ATT&CK Navigator coverage heatmaps, correlates results against Sigma rules, and runs detection validation loops to measure blue team visibility. Use for purple team exercises, atomic test execution, ATT&CK coverage assessment, or detection engineering validation.","domain":"cybersecurity","path":"skills/performing-purple-team-atomic-testing"},{"name":"performing-purple-team-exercise","description":"Performs purple team exercises by coordinating red team adversary emulation with blue team detection validation using MITRE ATT&CK-mapped attack scenarios, real-time detection testing, and collaborative gap remediation. Use when SOC teams need to validate detection capabilities, improve analyst skills, and close detection gaps through structured offensive-defensive collaboration.","domain":"cybersecurity","path":"skills/performing-purple-team-exercise"},{"name":"performing-ransomware-response","description":"Executes a structured ransomware incident response from detection through containment, forensic analysis, decryption assessment, recovery, and post-incident hardening, covering ransom negotiation, backup integrity verification, and regulatory notification. Use when actively responding to a ransomware outbreak or crypto-ransomware infection, or deciding on ransom payment and containment steps.","domain":"cybersecurity","path":"skills/performing-ransomware-response"},{"name":"performing-ransomware-tabletop-exercise","description":"Plans and facilitates tabletop exercises simulating ransomware incidents, using realistic scenarios based on threat actors like LockBit and ALPHV/BlackCat with injects covering double extortion and backup destruction, then evaluates responses against NIST CSF and CISA guidelines. Use when planning or running a ransomware tabletop exercise or incident response readiness drill.","domain":"cybersecurity","path":"skills/performing-ransomware-tabletop-exercise"},{"name":"performing-red-team-phishing-with-gophish","description":"Automates GoPhish phishing simulation campaigns using the Python gophish library, creating email templates with tracking pixels, configuring SMTP sending profiles, building target groups from CSV, launching campaigns, and analyzing results such as open rates, click rates, and credential submission statistics. Use when running an authorized phishing simulation or security awareness assessment via GoPhish.","domain":"cybersecurity","path":"skills/performing-red-team-phishing-with-gophish"},{"name":"performing-red-team-with-covenant","description":"Conducts red team operations using the Covenant C2 framework for authorized adversary simulation, covering listener setup, grunt deployment, task execution, and lateral movement tracking. Use when standing up or operating a Covenant command-and-control server for an authorized red team engagement.","domain":"cybersecurity","path":"skills/performing-red-team-with-covenant"},{"name":"performing-s7comm-protocol-security-analysis","description":"Perform security analysis of Siemens S7comm and S7CommPlus protocols used by SIMATIC S7 PLCs to identify vulnerabilities including replay attacks, integrity bypass, unauthorized CPU stop commands, and program download manipulation exploiting weaknesses in S7-300, S7-400, S7-1200, and S7-1500 controllers.","domain":"cybersecurity","path":"skills/performing-s7comm-protocol-security-analysis"},{"name":"performing-sca-dependency-scanning-with-snyk","description":"This skill covers implementing Software Composition Analysis (SCA) using Snyk to detect vulnerable open-source dependencies in CI/CD pipelines. It addresses scanning package manifests and lockfiles, automated fix pull request generation, license compliance checking, continuous monitoring of deployed applications, and integration with GitHub, GitLab, and Jenkins pipelines.","domain":"cybersecurity","path":"skills/performing-sca-dependency-scanning-with-snyk"},{"name":"performing-scada-hmi-security-assessment","description":"Perform security assessments of SCADA Human-Machine Interface (HMI) systems to identify vulnerabilities in web-based HMIs, thin-client configurations, authentication mechanisms, and communication channels between HMI and PLCs, aligned with IEC 62443 and NIST SP 800-82 guidelines.","domain":"cybersecurity","path":"skills/performing-scada-hmi-security-assessment"},{"name":"performing-second-order-sql-injection","description":"Detect and exploit second-order SQL injection vulnerabilities where malicious input is stored in a database and later executed in an unsafe SQL query during a different application operation.","domain":"cybersecurity","path":"skills/performing-second-order-sql-injection"},{"name":"performing-security-headers-audit","description":"Auditing HTTP security headers including CSP, HSTS, X-Frame-Options, and cookie attributes to identify missing or misconfigured browser-level protections.","domain":"cybersecurity","path":"skills/performing-security-headers-audit"},{"name":"performing-serverless-function-security-review","description":"Performing security reviews of serverless functions across AWS Lambda, Azure Functions, and GCP Cloud Functions to identify overly permissive execution roles, insecure environment variables, injection vulnerabilities, and missing runtime protections.","domain":"cybersecurity","path":"skills/performing-serverless-function-security-review"},{"name":"performing-service-account-audit","description":"Audit service accounts across enterprise infrastructure to identify orphaned, over-privileged, and non-compliant accounts. This skill covers discovery of service accounts in Active Directory, cloud pl","domain":"cybersecurity","path":"skills/performing-service-account-audit"},{"name":"performing-service-account-credential-rotation","description":"Automates credential rotation for service accounts across Active Directory, cloud platforms, and application databases to eliminate stale secrets and reduce compromise risk. Use when rotating or automating rotation of service account passwords, API keys, or secrets stored in a vault.","domain":"cybersecurity","path":"skills/performing-service-account-credential-rotation"},{"name":"performing-soap-web-service-security-testing","description":"Performs security testing of SOAP web services by analyzing WSDL definitions and testing for XML injection, XXE, WS-Security bypass, SOAPAction spoofing, and XPath injection. Use when assessing a SOAP/WSDL-based API endpoint for XML-related vulnerabilities during a penetration test.","domain":"cybersecurity","path":"skills/performing-soap-web-service-security-testing"},{"name":"performing-soc-tabletop-exercise","description":"Performs tabletop exercises for SOC teams simulating security incidents through discussion-based scenarios to test incident response procedures, communication workflows, and decision-making under pressure without impacting production systems. Use when organizations need to validate IR playbooks, train analysts, or meet compliance requirements for incident response testing.","domain":"cybersecurity","path":"skills/performing-soc-tabletop-exercise"},{"name":"performing-soc2-type2-audit-preparation","description":"Automates SOC 2 Type II audit preparation including gap assessment against AICPA Trust Services Criteria (CC1-CC9), evidence collection from cloud providers and identity systems, control testing validation, remediation tracking, and continuous compliance monitoring. Covers all five TSC categories (Security, Availability, Processing Integrity, Confidentiality, Privacy) with automated evidence gathering from AWS, Azure, GCP, Okta, GitHub, and Jira. Use when preparing for or maintaining SOC 2 Type II certification.","domain":"cybersecurity","path":"skills/performing-soc2-type2-audit-preparation"},{"name":"performing-sqlite-database-forensics","description":"Performs forensic analysis of SQLite databases by examining B-tree page structures, recovering deleted records from freelist pages and Write-Ahead Log (WAL) files, decoding encoded timestamps, and extracting evidence from browser history, messaging apps, and mobile device databases. Use when recovering deleted or unallocated data from a SQLite database during digital forensics or mobile/browser evidence analysis.","domain":"cybersecurity","path":"skills/performing-sqlite-database-forensics"},{"name":"performing-ssl-certificate-lifecycle-management","description":"Automates the full SSL/TLS certificate lifecycle, including generating Certificate Signing Requests, issuing, deploying, monitoring, renewing, and revoking X.509 certificates, using Python and ACME protocol tools. Use when managing certificate issuance or renewal, preventing certificate-expiry outages, or building automated PKI/ACME workflows.","domain":"cybersecurity","path":"skills/performing-ssl-certificate-lifecycle-management"},{"name":"performing-ssl-stripping-attack","description":"Simulates SSL stripping / HTTPS downgrade attacks using sslstrip, Bettercap, and mitmproxy in authorized lab environments to test HSTS enforcement, certificate validation, and HTTPS upgrade mechanisms. Use when performing an authorized penetration test to validate HSTS preloading and TLS certificate handling, demonstrate downgrade-attack risk to stakeholders, or train SOC teams to detect SSL stripping indicators in network traffic.","domain":"cybersecurity","path":"skills/performing-ssl-stripping-attack"},{"name":"performing-ssl-tls-inspection-configuration","description":"Configure SSL/TLS break-and-inspect on next-generation firewalls and forward proxies to decrypt, inspect, and re-encrypt HTTPS traffic for malware and exfiltration detection, including deploying trusted CA certificates, managing exemptions for certificate-pinned apps, and privacy compliance. Use when setting up or auditing TLS inspection on network security devices to close the encrypted-traffic blind spot.","domain":"cybersecurity","path":"skills/performing-ssl-tls-inspection-configuration"},{"name":"performing-ssl-tls-security-assessment","description":"Assess SSL/TLS server configurations using the sslyze Python scanning library to evaluate supported protocol versions, cipher suite strength, certificate chain validation, HSTS enforcement, OCSP stapling, and known vulnerabilities such as Heartbleed and ROBOT. Use when conducting a security assessment of a server's TLS configuration or verifying remediation of cipher/certificate weaknesses.","domain":"cybersecurity","path":"skills/performing-ssl-tls-security-assessment"},{"name":"performing-ssrf-vulnerability-exploitation","description":"Tests web application URL parameters for Server-Side Request Forgery by probing cloud metadata endpoints (AWS/GCP/Azure at 169.254.169.254), internal network services, and protocol handlers (file://, gopher://, dict://) using a Python script, including IP-encoding bypass and DNS rebinding checks. Use during authorized penetration testing to confirm SSRF in a URL-fetching parameter and generate a vulnerability report.","domain":"cybersecurity","path":"skills/performing-ssrf-vulnerability-exploitation"},{"name":"performing-static-malware-analysis-with-pe-studio","description":"Performs static analysis of Windows PE malware samples using PEStudio to examine file headers, imports, strings, and resources without executing the binary, identifying packing, anti-analysis tricks, and malicious imports. Use for pre-execution triage of a suspicious Windows executable before sandbox detonation.","domain":"cybersecurity","path":"skills/performing-static-malware-analysis-with-pe-studio"},{"name":"performing-steganography-detection","description":"Detects and extracts hidden data embedded in images, audio, and other media files using steganalysis tools such as StegDetect, zsteg, stegsolve, binwalk, steghide, and OpenStego to uncover covert communication channels. Use when investigating suspected data hiding or exfiltration via media files, espionage/insider-threat cases, or anomalies in media file properties found during standard file analysis.","domain":"cybersecurity","path":"skills/performing-steganography-detection"},{"name":"performing-subdomain-enumeration-with-subfinder","description":"Enumerate subdomains of target domains using ProjectDiscovery's Subfinder passive reconnaissance tool to map the attack surface during security assessments.","domain":"cybersecurity","path":"skills/performing-subdomain-enumeration-with-subfinder"},{"name":"performing-supply-chain-attack-simulation","description":"Simulates and detects software supply chain attacks: typosquatting detection via Levenshtein distance against popular PyPI package names, dependency confusion testing against private registries, SHA-256 package hash verification, and known-CVE scanning with pip-audit. Use when auditing a project's dependencies for malicious or confused packages, or when assessing package-registry supply-chain risk.","domain":"cybersecurity","path":"skills/performing-supply-chain-attack-simulation"},{"name":"performing-thick-client-application-penetration-test","description":"Conduct a thick client application penetration test to identify insecure local storage, hardcoded credentials, DLL hijacking, memory manipulation, and insecure API communication in desktop applications using dnSpy, Procmon, and Burp Suite.","domain":"cybersecurity","path":"skills/performing-thick-client-application-penetration-test"},{"name":"performing-threat-emulation-with-atomic-red-team","description":"Executes Atomic Red Team tests for MITRE ATT&CK technique validation using the atomic-operator Python framework. Loads test definitions from YAML atomics, runs attack simulations, and validates detection coverage. Use when testing SIEM detection rules, validating EDR coverage, or conducting purple team exercises.","domain":"cybersecurity","path":"skills/performing-threat-emulation-with-atomic-red-team"},{"name":"performing-threat-hunting-with-elastic-siem","description":"Performs proactive threat hunting in Elastic Security SIEM using KQL/EQL queries, detection rules, and Timeline investigation to identify threats that evade automated detection. Use when SOC teams need to hunt for specific ATT&CK techniques, investigate anomalous behaviors, or validate detection coverage gaps using Elasticsearch and Kibana Security.","domain":"cybersecurity","path":"skills/performing-threat-hunting-with-elastic-siem"},{"name":"performing-threat-hunting-with-yara-rules","description":"Use YARA pattern-matching rules to hunt for malware, suspicious files, and indicators of compromise across filesystems and memory dumps. Covers rule authoring, yara-python scanning, and integration with threat intel feeds.","domain":"cybersecurity","path":"skills/performing-threat-hunting-with-yara-rules"},{"name":"performing-threat-intelligence-sharing-with-misp","description":"Uses PyMISP (the official MISP REST API library) to create events with structured IOCs (IPs, domains, hashes, URLs), enrich them with MITRE ATT&CK tags and galaxy clusters, manage sharing groups and distribution levels, search existing intelligence, and export in STIX 2.1 format. Use when creating, enriching, or sharing threat intelligence events on a MISP instance, or integrating IOC feeds with other platforms.","domain":"cybersecurity","path":"skills/performing-threat-intelligence-sharing-with-misp"},{"name":"performing-threat-landscape-assessment-for-sector","description":"Conducts a sector-specific threat landscape assessment (financial, healthcare, energy, government, etc.) by profiling targeting threat actors, mapping attack vectors and MITRE ATT&CK TTPs with the attackcti/pandas Python stack, and analyzing exploited CVEs and incident trends from ISAC and vendor reports. Use when producing CTI for risk management or board-level reporting on an industry's threat exposure.","domain":"cybersecurity","path":"skills/performing-threat-landscape-assessment-for-sector"},{"name":"performing-threat-modeling-with-owasp-threat-dragon","description":"Uses OWASP Threat Dragon (web or desktop) to build data flow diagrams, identify threats with STRIDE, LINDDUN, CIA, DIE, or PLOT4ai methodologies via its auto-generation rule engine, and produce PDF threat model reports. Use during secure design review of an application architecture to build a formal threat model and document mitigations for GRC compliance.","domain":"cybersecurity","path":"skills/performing-threat-modeling-with-owasp-threat-dragon"},{"name":"performing-timeline-reconstruction-with-plaso","description":"Builds comprehensive forensic super-timelines using Plaso (log2timeline and psort) to correlate events across file system metadata, event logs, browser history, and registry artifacts into a unified chronological view. Use during complex forensic investigations that need cross-source event correlation, or when standard log analysis is insufficient to establish the sequence of activities for reporting findings.","domain":"cybersecurity","path":"skills/performing-timeline-reconstruction-with-plaso"},{"name":"performing-user-behavior-analytics","description":"Performs User and Entity Behavior Analytics (UEBA) to detect anomalous user activities including impossible travel, unusual access patterns, privilege abuse, and insider threats using SIEM-based behavioral baselines and statistical analysis. Use when SOC teams need to identify compromised accounts or insider threats through deviation from established behavioral norms.","domain":"cybersecurity","path":"skills/performing-user-behavior-analytics"},{"name":"performing-vlan-hopping-attack","description":"Simulates VLAN hopping attacks using switch spoofing and 802.1Q double tagging techniques in authorized lab environments to test VLAN segmentation effectiveness and switch port security. Use during an authorized penetration test to validate trunk port hardening, confirm DTP is disabled on access ports, and demonstrate Layer 2 segmentation bypass risk to network teams.","domain":"cybersecurity","path":"skills/performing-vlan-hopping-attack"},{"name":"performing-vulnerability-scanning-with-nessus","description":"Performs authenticated and unauthenticated vulnerability scanning using Tenable Nessus to identify known vulnerabilities, misconfigurations, default credentials, and missing patches across network infrastructure, servers, and applications. The scanner correlates findings with CVE databases and CVSS scores to produce prioritized remediation guidance. Activates for requests involving vulnerability scanning, Nessus assessment, patch compliance checking, or automated vulnerability detection.","domain":"cybersecurity","path":"skills/performing-vulnerability-scanning-with-nessus"},{"name":"performing-web-application-firewall-bypass","description":"Bypasses Web Application Firewall protections using encoding tricks, HTTP method manipulation, parameter pollution, and payload obfuscation to smuggle SQL injection, XSS, and other exploit payloads past WAF detection rules. Use when a confirmed vulnerability is blocked by a WAF, during penetration tests or red team engagements requiring bypass of perimeter security controls, or when evaluating WAF rule effectiveness and bypass resistance.","domain":"cybersecurity","path":"skills/performing-web-application-firewall-bypass"},{"name":"performing-web-application-penetration-test","description":"Performs systematic security testing of web applications following the OWASP Web Security Testing Guide (WSTG) methodology to identify vulnerabilities in authentication, authorization, input validation, session management, and business logic. The tester uses Burp Suite as the primary interception proxy alongside manual testing techniques to find flaws that automated scanners miss. Activates for requests involving web app pentest, OWASP testing, application security assessment, or web vulnerability testing.","domain":"cybersecurity","path":"skills/performing-web-application-penetration-test"},{"name":"performing-web-application-scanning-with-nikto","description":"Runs Nikto, an open-source web server and web application scanner, to test over 7,000 potentially dangerous files/programs, check for outdated versions across 1,250+ servers, and identify XSS, SQL injection, misconfigurations, default credentials, and vulnerable CGI scripts. Use when performing web application vulnerability scanning, security assessments, scheduled security testing, or validating web server security controls.","domain":"cybersecurity","path":"skills/performing-web-application-scanning-with-nikto"},{"name":"performing-web-application-vulnerability-triage","description":"Triages web application vulnerability findings from DAST/SAST scanners such as Burp Suite and ZAP, using the OWASP Risk Rating Methodology to confirm true positives, dismiss false positives, and prioritize remediation. Use when reviewing scanner output to reduce alert fatigue and rank vulnerabilities for development teams to fix.","domain":"cybersecurity","path":"skills/performing-web-application-vulnerability-triage"},{"name":"performing-web-cache-deception-attack","description":"Executes web cache deception attacks by exploiting path normalization discrepancies between CDN/reverse-proxy caching layers (Cloudflare, Akamai, Varnish, Nginx) and origin servers to cache and retrieve sensitive authenticated content. Use when testing applications behind CDNs for cache-key misconfigurations, during bug bounty hunting on aggressively cached sites, or when assessing sensitive data exposure through cache layer misconfiguration.","domain":"cybersecurity","path":"skills/performing-web-cache-deception-attack"},{"name":"performing-web-cache-poisoning-attack","description":"Exploiting web cache mechanisms to serve malicious content to other users by poisoning cached responses through unkeyed headers and parameters during authorized security tests.","domain":"cybersecurity","path":"skills/performing-web-cache-poisoning-attack"},{"name":"performing-wifi-password-cracking-with-aircrack","description":"Captures WPA/WPA2 handshakes and performs offline password cracking using aircrack-ng, hashcat, and dictionary attacks during authorized wireless security assessments to evaluate passphrase strength and wireless network security posture.","domain":"cybersecurity","path":"skills/performing-wifi-password-cracking-with-aircrack"},{"name":"performing-windows-artifact-analysis-with-eric-zimmerman-tools","description":"Performs comprehensive Windows forensic artifact analysis using Eric Zimmerman's open-source EZ Tools suite (KAPE, MFTECmd, PECmd, LECmd, JLECmd, Timeline Explorer) to parse registry hives, prefetch files, event logs, and file system metadata. Use during DFIR investigations that need a timeline of program execution, file access, and persistence built from Windows artifacts.","domain":"cybersecurity","path":"skills/performing-windows-artifact-analysis-with-eric-zimmerman-tools"},{"name":"performing-wireless-network-penetration-test","description":"Execute a wireless network penetration test to assess WiFi security by capturing handshakes, cracking WPA2/WPA3 keys, detecting rogue access points, and testing wireless segmentation using Aircrack-ng and related tools.","domain":"cybersecurity","path":"skills/performing-wireless-network-penetration-test"},{"name":"performing-wireless-security-assessment-with-kismet","description":"Conduct wireless network security assessments using Kismet to detect rogue access points, hidden SSIDs, weak encryption, and unauthorized clients through passive RF monitoring.","domain":"cybersecurity","path":"skills/performing-wireless-security-assessment-with-kismet"},{"name":"performing-yara-rule-development-for-detection","description":"Develops precise YARA and YARA-X rules for malware detection by identifying unique strings, byte sequences, PE header traits, and behavioral indicators in unpacked malware artifacts while minimizing false positives. Use when building detection signatures for threat hunting, classifying malware families, or authoring rules from IOCs such as C2 URLs, mutex names, and encryption constants.","domain":"cybersecurity","path":"skills/performing-yara-rule-development-for-detection"},{"name":"post-exploiting-microsoft-graph-with-graphrunner","description":"Runs GraphRunner, a PowerShell post-exploitation toolset built on the Microsoft Graph API, to perform tenant recon, establish persistence (OAuth app injection, inbox rules), escalate privilege via group manipulation, and pillage mailboxes, SharePoint, and Teams data from a foothold Graph token. Use during authorized red-team engagements against Microsoft 365/Entra ID tenants once you hold a Graph token.","domain":"cybersecurity","path":"skills/post-exploiting-microsoft-graph-with-graphrunner"},{"name":"prioritizing-vulnerabilities-with-cvss-scoring","description":"The Common Vulnerability Scoring System (CVSS) is the industry standard framework maintained by FIRST (Forum of Incident Response and Security Teams) for assessing vulnerability severity. CVSS v4.0 (r","domain":"cybersecurity","path":"skills/prioritizing-vulnerabilities-with-cvss-scoring"},{"name":"processing-stix-taxii-feeds","description":"Processes STIX 2.1 threat intelligence bundles delivered via TAXII 2.1 servers, normalizing objects into platform-native schemas and routing them to appropriate consuming systems. Use when onboarding new TAXII collection endpoints, automating bi-directional intelligence sharing with ISACs, or building pipeline validation for malformed STIX bundles. Activates for requests involving OASIS STIX, TAXII server configuration, MISP TAXII, or Cortex XSOAR feed integrations.","domain":"cybersecurity","path":"skills/processing-stix-taxii-feeds"},{"name":"profiling-threat-actor-groups","description":"Develops comprehensive threat actor profiles for APT groups, criminal organizations, and hacktivist collectives by aggregating TTP documentation, historical campaign data, tooling fingerprints, and attribution indicators from multiple intelligence sources. Use when briefing executives on sector-specific threats, updating threat model assumptions, or prioritizing defensive controls against specific adversaries. Activates for requests involving MITRE ATT&CK Groups, Mandiant APT profiles, CrowdStrike adversary naming, or sector-specific threat briefings.","domain":"cybersecurity","path":"skills/profiling-threat-actor-groups"},{"name":"recovering-deleted-files-with-photorec","description":"Recovers deleted files from disk images and storage media using PhotoRec's file signature-based carving engine, which works regardless of file system damage or corruption. Use when recovering deleted or lost files from a forensic disk image, damaged storage device, or corrupted file system during evidence recovery.","domain":"cybersecurity","path":"skills/recovering-deleted-files-with-photorec"},{"name":"recovering-from-ransomware-attack","description":"Executes structured ransomware incident recovery following NIST/CISA frameworks: environment isolation, forensic evidence preservation, clean infrastructure rebuild, prioritized restoration from verified backups, credential reset, and Active Directory/database recovery in dependency order. Use when recovering from a ransomware attack, performing post-encryption restoration, or executing disaster recovery after ransomware encryption.","domain":"cybersecurity","path":"skills/recovering-from-ransomware-attack"},{"name":"red-teaming-llms-with-garak","description":"Runs NVIDIA garak probe suites (jailbreak, prompt injection, data leakage, toxicity, and more) against an LLM endpoint - Hugging Face models, OpenAI-compatible APIs, or Bedrock - then interprets the resulting hit-rate report for triage. Use when baselining LLM security before/after deployment, validating that a guardrail or fine-tune reduces jailbreak/injection success rates, or producing evidence for an AI risk assessment.","domain":"cybersecurity","path":"skills/red-teaming-llms-with-garak"},{"name":"relaying-ntlm-for-adcs-esc8","description":"Uses Impacket's ntlmrelayx.py with a coercion tool (PetitPotam, Coercer, printerbug) to relay NTLM authentication from a coerced domain controller into the AD CS HTTP web-enrollment endpoint (ESC8), obtaining a DC certificate usable via Certipy for PKINIT/DCSync. Use during internal AD penetration tests or purple-team exercises when AD CS Web Enrollment lacks Extended Protection for Authentication.","domain":"cybersecurity","path":"skills/relaying-ntlm-for-adcs-esc8"},{"name":"remediating-s3-bucket-misconfiguration","description":"Provides step-by-step procedures for remediating Amazon S3 bucket misconfigurations that expose sensitive data: enabling S3 Block Public Access, auditing bucket policies and ACLs, enforcing encryption, configuring access logging, and deploying automated remediation with AWS Config and Lambda. Use when AWS Config or Security Hub flags public or unencrypted S3 buckets, or preparing audit evidence for storage security controls.","domain":"cybersecurity","path":"skills/remediating-s3-bucket-misconfiguration"},{"name":"reverse-engineering-android-malware-with-jadx","description":"Reverse engineers malicious Android APK files using the JADX decompiler to read Java/Kotlin source, inspect manifest permissions, receivers, services, and native libraries, and identify data theft, C2 communication, privilege escalation, or overlay-attack behavior. Use when analyzing a suspected Android banking trojan, spyware, SMS stealer, or other flagged APK, or when investigating mobile malware or Android threats generally.","domain":"cybersecurity","path":"skills/reverse-engineering-android-malware-with-jadx"},{"name":"reverse-engineering-dotnet-malware-with-dnspy","description":"Reverse engineers .NET malware samples using the dnSpy decompiler and debugger to read C#/VB.NET source, deobfuscate code protected by tools like ConfuserEx or SmartAssembly, and extract hardcoded C2 configurations, keys, and credentials. Use when a sample is identified as a .NET assembly (e.g. AgentTesla, AsyncRAT, RedLine Stealer, Quasar RAT) and needs decompilation, deobfuscation, or config extraction.","domain":"cybersecurity","path":"skills/reverse-engineering-dotnet-malware-with-dnspy"},{"name":"reverse-engineering-ios-app-with-frida","description":"Reverse engineers iOS applications using Frida dynamic instrumentation to understand internal logic, extract encryption keys, bypass security controls, and discover hidden functionality without source code access. Use when performing authorized iOS penetration testing, analyzing proprietary protocols, understanding obfuscated logic, or extracting runtime secrets from iOS binaries. Activates for requests involving iOS reverse engineering, Frida iOS hooking, Objective-C/Swift method tracing, or iOS binary analysis.","domain":"cybersecurity","path":"skills/reverse-engineering-ios-app-with-frida"},{"name":"reverse-engineering-malware-with-ghidra","description":"Reverse engineers malware binaries using NSA's Ghidra disassembler and decompiler to study internal logic, cryptographic routines, C2 protocols, and evasion techniques at the assembly and pseudo-C level. Use when static or dynamic analysis flags suspicious functionality needing deeper code review, such as reversing C2 protocols, encryption algorithms, custom obfuscation, or a sample's exploit mechanism.","domain":"cybersecurity","path":"skills/reverse-engineering-malware-with-ghidra"},{"name":"reverse-engineering-ransomware-encryption-routine","description":"Reverse engineer ransomware encryption routines to identify cryptographic algorithms, key generation flaws, and potential decryption opportunities using static and dynamic analysis.","domain":"cybersecurity","path":"skills/reverse-engineering-ransomware-encryption-routine"},{"name":"reverse-engineering-rust-malware","description":"Reverse engineers Rust-compiled malware using IDA Pro and Ghidra, covering techniques for non-null-terminated fat-pointer strings, monomorphized/duplicated generic code, Result/Option unwrap chains, crate dependency extraction, and Rust-specific control flow and calling conventions. Use when analyzing Rust-based malware samples (e.g. BlackCat/ALPHV, Hive, Buer Loader) or attack artifacts in an authorized, controlled environment.","domain":"cybersecurity","path":"skills/reverse-engineering-rust-malware"},{"name":"scanning-container-images-with-grype","description":"Scans container images, filesystems, and SBOMs for known CVEs with Anchore Grype, matching Syft-generated SBOM packages against NVD, GitHub Advisories, and OS-specific feeds with configurable severity thresholds and failure gates. Use when Grype or Syft is the chosen toolchain, when scanning an existing SBOM rather than an image, or when gating a build on severity. Keywords: Grype, Syft, SBOM, NVD, GitHub Advisory, --fail-on, severity threshold. Do not use when the toolchain is Trivy - use scanning-docker-images-with-trivy.","domain":"cybersecurity","path":"skills/scanning-container-images-with-grype"},{"name":"scanning-containers-with-trivy-in-cicd","description":"Integrates Aqua Security's Trivy scanner into CI/CD pipelines to detect OS package and application dependency CVEs, Dockerfile misconfigurations, and issues in filesystems or git repositories, and to enforce severity-based quality gates that block vulnerable images from being deployed. Use when building Docker images in CI/CD and needing automated vulnerability scanning and pass/fail gates before registry push or production deployment.","domain":"cybersecurity","path":"skills/scanning-containers-with-trivy-in-cicd"},{"name":"scanning-docker-images-with-trivy","description":"Scans a Docker image with Trivy for vulnerabilities in OS packages and language dependencies, misconfiguration, exposed secrets, and licence violations, emitting SARIF, CycloneDX, or SPDX output. Use when scanning or gating a specific image, wiring an image scan into CI/CD, or checking an image during an incident investigation. Keywords: Trivy, image scan, --severity, --exit-code, SARIF, ignore file, .trivyignore. Do not use for cluster-wide scanning or non-image targets - use performing-container-security-scanning-with-trivy; when the toolchain is Grype use scanning-container-images-with-grype.","domain":"cybersecurity","path":"skills/scanning-docker-images-with-trivy"},{"name":"scanning-iac-and-images-with-trivy","description":"Scans container images, Infrastructure-as-Code (Terraform, CloudFormation, Kubernetes manifests, Dockerfile, Helm), filesystems, git repos, and SBOMs with Trivy's vuln, misconfig, secret, and license scanners. Use when building a CI/CD security gate that scans images before push, IaC before apply, or SBOMs for supply-chain weaknesses, and fails the build on policy violations.","domain":"cybersecurity","path":"skills/scanning-iac-and-images-with-trivy"},{"name":"scanning-infrastructure-with-nessus","description":"Tenable Nessus is the industry-leading vulnerability scanner used to identify security weaknesses across network infrastructure including servers, workstations, network devices, and operating systems.","domain":"cybersecurity","path":"skills/scanning-infrastructure-with-nessus"},{"name":"scanning-kubernetes-manifests-with-kubesec","description":"Scores Kubernetes resource manifests with Kubesec to flag misconfiguration and privilege-escalation risk before deployment, mapping each finding back to the securityContext change that fixes it. Use when gating manifests in CI, reviewing YAML or a rendered chart before it reaches a cluster, or explaining why a manifest scored negatively. Keywords: Kubesec, manifest score, securityContext, readOnlyRootFilesystem, runAsNonRoot, CI gate. Do not use for scanning built images for CVEs - use scanning-docker-images-with-trivy; for admission-time enforcement use implementing-opa-gatekeeper-for-policy-enforcement.","domain":"cybersecurity","path":"skills/scanning-kubernetes-manifests-with-kubesec"},{"name":"scanning-network-with-nmap-advanced","description":"Performs advanced network recon using Nmap's Scripting Engine (NSE), timing controls, firewall/IDS evasion, and structured output parsing to discover hosts, enumerate service versions, detect vulnerabilities, and fingerprint OSes. Use during authorized penetration tests or enterprise asset-discovery assessments needing scan evasion, NSE vulnerability checks, or output fed into a vulnerability management pipeline.","domain":"cybersecurity","path":"skills/scanning-network-with-nmap-advanced"},{"name":"securing-agentic-ai-tool-invocation","description":"Implements defense-in-depth controls at an AI agent's tool-invocation boundary using tool allowlisting, least-privilege identity binding, NeMo Guardrails policy enforcement, human-in-the-loop approval, and audit logging. Use when hardening an agent that calls tools with real side effects (email, payments, file writes, code execution), mapping OWASP Agentic AI Top 10 controls, or bounding prompt-injection blast radius.","domain":"cybersecurity","path":"skills/securing-agentic-ai-tool-invocation"},{"name":"securing-api-gateway-with-aws-waf","description":"Secures AWS API Gateway endpoints with AWS WAF by configuring managed rule groups for OWASP Top 10 protection, custom rate-limiting rules, bot control, IP reputation filtering, and WAF metric monitoring. Use when deploying API Gateway endpoints that need protection against common web attacks, or when implementing rate limiting and throttling to prevent API abuse and DDoS.","domain":"cybersecurity","path":"skills/securing-api-gateway-with-aws-waf"},{"name":"securing-aws-iam-permissions","description":"Hardens AWS IAM configurations to enforce least-privilege access, covering IAM policy scoping, permission boundaries, IAM Access Analyzer integration, and credential rotation strategies. Use when reducing the blast radius of compromised AWS identities, auditing overly permissive IAM policies, or setting up permission boundaries and Access Analyzer findings review across cloud accounts.","domain":"cybersecurity","path":"skills/securing-aws-iam-permissions"},{"name":"securing-aws-lambda-execution-roles","description":"Hardens AWS Lambda execution roles by writing least-privilege IAM policies, applying permission boundaries, restricting resource-based policies, validating permissions with IAM Access Analyzer, and enforcing role scoping through SCPs. Use when deploying new Lambda functions, remediating overly permissive Lambda roles found in an audit, or building reusable least-privilege IAM role templates for serverless teams.","domain":"cybersecurity","path":"skills/securing-aws-lambda-execution-roles"},{"name":"securing-azure-with-microsoft-defender","description":"Deploys and configures Microsoft Defender for Cloud as a CNAPP for Azure, multi-cloud, and hybrid environments: enabling Defender plans for servers, containers, storage, and databases, configuring recommendations, and managing Secure Score via the unified Defender portal. Use when onboarding workloads to Defender for Cloud or setting up cloud workload protection and threat monitoring.","domain":"cybersecurity","path":"skills/securing-azure-with-microsoft-defender"},{"name":"securing-container-registry-images","description":"Secures container registry images (ECR, ACR, GCR, Docker Hub) by scanning with Trivy and Grype, signing with Cosign and Sigstore, configuring registry access controls, and building CI/CD pipelines that block unscanned or unsigned images. Use when establishing registry security controls or enforcing scan/signature checks before image promotion.","domain":"cybersecurity","path":"skills/securing-container-registry-images"},{"name":"securing-container-registry-with-harbor","description":"Configures the security features of the Harbor open-source container registry - integrated Trivy scanning, Cosign and Notary content trust policies, project-level RBAC, immutable tag and retention rules, and OIDC authentication - to enforce provenance and block deployment of vulnerable images. Use when deploying or hardening Harbor, or when compliance requires that only signed and scanned images can be pulled. Keywords: Harbor, project policy, content trust, immutable tag, retention, robot account, OIDC, replication. Do not use for signing images with Cosign outside a registry - use implementing-image-provenance-verification-with-cosign.","domain":"cybersecurity","path":"skills/securing-container-registry-with-harbor"},{"name":"securing-github-actions-workflows","description":"Hardens GitHub Actions workflows against supply chain attacks, credential theft, and privilege escalation: pinning actions to SHA digests, minimizing GITHUB_TOKEN permissions, protecting secrets, preventing script injection in workflow expressions, and requiring reviewers for workflow changes. Use when hardening GitHub Actions workflows that handle secrets, deploy to production, or run with elevated permissions.","domain":"cybersecurity","path":"skills/securing-github-actions-workflows"},{"name":"securing-helm-chart-deployments","description":"Secures Helm chart deployments by verifying chart signatures and provenance, rendering and linting templates for misconfiguration, enforcing pod security contexts through values.yaml, moving secrets into an external store instead of Helm values, and scoping RBAC for Helm operations in CI/CD. Use when deploying charts to Kubernetes or reviewing chart provenance, templates, or release RBAC. Keywords: Helm, provenance file, helm verify, helm lint, values.yaml, Tiller-less, release RBAC, external secrets. Do not use for scanning the rendered manifests themselves - use scanning-kubernetes-manifests-with-kubesec.","domain":"cybersecurity","path":"skills/securing-helm-chart-deployments"},{"name":"securing-historian-server-in-ot-environment","description":"Audits and hardens process historian servers (OSIsoft PI, Honeywell PHD, GE Proficy, AVEVA Historian) in OT environments: Purdue-level network placement, interface access control, secure DMZ replication via data diodes or PI-to-PI connectors, SQL injection prevention, and process data integrity. Use when auditing or hardening a historian server, or designing secure OT-to-IT data replication through a DMZ.","domain":"cybersecurity","path":"skills/securing-historian-server-in-ot-environment"},{"name":"securing-kubernetes-on-cloud","description":"Hardens managed Kubernetes clusters on EKS, AKS, and GKE by implementing Pod Security Standards, network policies, workload identity (IRSA for EKS, Workload Identity for GKE, Managed Identities for AKS), RBAC scoping, image admission controls, and runtime security monitoring. Use when deploying a new managed Kubernetes cluster with security requirements or hardening an existing EKS, AKS, or GKE cluster after an audit or pentest finding.","domain":"cybersecurity","path":"skills/securing-kubernetes-on-cloud"},{"name":"securing-remote-access-to-ot-environment","description":"Designs and configures secure remote access to OT/ICS environments for operators, engineers, and vendors: jump server architecture, multi-factor authentication, session recording, privileged access management, vendor access controls, and IEC 62443 / NERC CIP-005 compliance. Use when designing OT remote access architecture or configuring a jump server with privileged access management.","domain":"cybersecurity","path":"skills/securing-remote-access-to-ot-environment"},{"name":"securing-serverless-functions","description":"Hardens serverless compute platforms (AWS Lambda, Azure Functions, Google Cloud Functions): least-privilege IAM roles, dependency vulnerability scanning, secrets management integration, input validation, function URL authentication, and runtime monitoring. Use when deploying serverless functions with sensitive access, auditing for overly permissive roles, or adding functions to a DevSecOps pipeline.","domain":"cybersecurity","path":"skills/securing-serverless-functions"},{"name":"testing-android-intents-for-vulnerabilities","description":"Tests Android inter-process communication (IPC) through intents for vulnerabilities including intent injection, unauthorized component access, broadcast sniffing, pending intent hijacking, and content provider data leakage. Use when assessing Android app attack surface through exported components, testing intent-based data flows, or evaluating IPC security. Activates for requests involving Android intent security, IPC testing, exported component analysis, or Drozer assessment.","domain":"cybersecurity","path":"skills/testing-android-intents-for-vulnerabilities"},{"name":"testing-api-authentication-weaknesses","description":"Tests API authentication mechanisms for weaknesses including broken token validation, missing authentication on endpoints, weak password policies, credential stuffing susceptibility, token leakage in URLs or logs, and session management flaws. The tester evaluates JWT implementation, API key handling, OAuth flows, and session token entropy to identify authentication bypasses. Maps to OWASP API2:2023 Broken Authentication. Activates for requests involving API authentication testing, token validation assessment, credential security testing, or API auth bypass.","domain":"cybersecurity","path":"skills/testing-api-authentication-weaknesses"},{"name":"testing-api-for-broken-object-level-authorization","description":"Tests REST and GraphQL APIs for Broken Object Level Authorization (BOLA/IDOR, OWASP API1:2023) by intercepting API calls, identifying object ID parameters (numeric IDs, UUIDs, slugs), and systematically substituting IDs belonging to other users to check whether the server enforces per-object authorization. Use when asked to test BOLA or IDOR in an API, verify object-level authorization, or assess an API for access control bypass.","domain":"cybersecurity","path":"skills/testing-api-for-broken-object-level-authorization"},{"name":"testing-api-for-mass-assignment-vulnerability","description":"Tests APIs for mass assignment (auto-binding), OWASP API3:2023, by identifying writable endpoints, adding undocumented fields to request bodies (role, isAdmin, price, balance), and checking whether the server binds them to the data model without filtering. Use when testing an API for mass assignment, parameter binding abuse, or over-posting on profile, registration, or object-creation endpoints.","domain":"cybersecurity","path":"skills/testing-api-for-mass-assignment-vulnerability"},{"name":"testing-api-security-with-owasp-top-10","description":"Systematically assesses REST, GraphQL, and gRPC API endpoints against the OWASP API Security Top 10 (2023) using Burp Suite and Postman for automated and manual testing. Use during authorized API penetration tests, before deploying new endpoints to production, or when validating API gateway controls and rate limiting.","domain":"cybersecurity","path":"skills/testing-api-security-with-owasp-top-10"},{"name":"testing-cors-misconfiguration","description":"Identifying and exploiting Cross-Origin Resource Sharing misconfigurations that allow unauthorized cross-domain data access and credential theft during security assessments.","domain":"cybersecurity","path":"skills/testing-cors-misconfiguration"},{"name":"testing-for-broken-access-control","description":"Systematically tests web applications and APIs for broken access control (OWASP A01:2021), including privilege escalation, missing function-level checks, insecure direct object references, and multi-tenant data leakage, using Burp Suite with the Authorize extension. Use during authorized penetration tests or RBAC/multi-tenant authorization audits.","domain":"cybersecurity","path":"skills/testing-for-broken-access-control"},{"name":"testing-for-business-logic-vulnerabilities","description":"Manually identifies flaws in application business logic - price manipulation, multi-step workflow bypass, and privilege escalation - by intercepting and modifying requests with Burp Suite, going beyond what automated vulnerability scanners detect. Use for e-commerce checkout/cart flows, voucher and rewards systems, or any assessment where scanners find little but business rules need scrutiny.","domain":"cybersecurity","path":"skills/testing-for-business-logic-vulnerabilities"},{"name":"testing-for-email-header-injection","description":"Tests web application email functionality (contact forms, password reset, newsletter subscriptions) for CRLF/SMTP header injection using Burp Suite and OWASP ZAP, checking whether attackers can inject headers, modify recipients, or abuse forms for spam relay. Use when testing any user-input-driven email-sending feature during a penetration test.","domain":"cybersecurity","path":"skills/testing-for-email-header-injection"},{"name":"testing-for-host-header-injection","description":"Test web applications for HTTP Host header injection vulnerabilities to identify password reset poisoning, web cache poisoning, SSRF, and virtual host routing manipulation risks.","domain":"cybersecurity","path":"skills/testing-for-host-header-injection"},{"name":"testing-for-json-web-token-vulnerabilities","description":"Tests JWT implementations for algorithm confusion, \"none\" algorithm bypass, kid/jku parameter injection, and weak secret exploitation using jwt_tool and Burp Suite's JWT Editor extension, aiming to achieve authentication bypass and privilege escalation. Use when assessing JWT-based auth/session management, OAuth2/OIDC token handling, or SSO systems during a security engagement.","domain":"cybersecurity","path":"skills/testing-for-json-web-token-vulnerabilities"},{"name":"testing-for-open-redirect-vulnerabilities","description":"Identifies and exploits open redirect vulnerabilities by analyzing URL redirection parameters (next, url, redirect, return, goto), applying bypass techniques, and chaining findings into phishing or token-theft exploits, using Burp Suite/OWASP ZAP and Burp Collaborator. Use when testing login/logout flows, OAuth redirect_uri handling, or SSO redirect validation.","domain":"cybersecurity","path":"skills/testing-for-open-redirect-vulnerabilities"},{"name":"testing-for-sensitive-data-exposure","description":"Identifying sensitive data exposure vulnerabilities including API key leakage, PII in responses, insecure storage, and unprotected data transmission during security assessments.","domain":"cybersecurity","path":"skills/testing-for-sensitive-data-exposure"},{"name":"testing-for-system-prompt-leakage","description":"Extracts LLM system prompts using direct requests, jailbreak/instruction-override framing, translation/encoding tricks, and few-shot replay, combining manual payloads with automated garak and Promptfoo scanners to surface embedded secrets, routing logic, and policy leakage (OWASP LLM07:2025). Use during LLM application red-team engagements or when validating that no credentials or authorization logic live in the system prompt.","domain":"cybersecurity","path":"skills/testing-for-system-prompt-leakage"},{"name":"testing-for-xml-injection-vulnerabilities","description":"Test web applications for XML injection vulnerabilities including XXE, XPath injection, and XML entity attacks to identify data exposure and server-side request forgery risks.","domain":"cybersecurity","path":"skills/testing-for-xml-injection-vulnerabilities"},{"name":"testing-for-xss-vulnerabilities","description":"Tests web applications for reflected, stored, and DOM-based Cross-Site Scripting by injecting JavaScript payloads with Burp Suite (XSS extensions, Active Scan++) and browser tools, then bypassing sanitization and CSP to demonstrate session hijacking and user impersonation. Use for OWASP WSTG client-side injection testing or when evaluating input sanitization and output encoding coverage.","domain":"cybersecurity","path":"skills/testing-for-xss-vulnerabilities"},{"name":"testing-for-xss-vulnerabilities-with-burpsuite","description":"Identifying and validating cross-site scripting vulnerabilities using Burp Suite's scanner, intruder, and repeater tools during authorized security assessments.","domain":"cybersecurity","path":"skills/testing-for-xss-vulnerabilities-with-burpsuite"},{"name":"testing-for-xxe-injection-vulnerabilities","description":"Discovering and exploiting XML External Entity injection vulnerabilities to read server files, perform SSRF, and exfiltrate data during authorized penetration tests.","domain":"cybersecurity","path":"skills/testing-for-xxe-injection-vulnerabilities"},{"name":"testing-jwt-token-security","description":"Assessing JSON Web Token implementations for cryptographic weaknesses, algorithm confusion attacks, and authorization bypass vulnerabilities during security engagements.","domain":"cybersecurity","path":"skills/testing-jwt-token-security"},{"name":"testing-mobile-api-authentication","description":"Tests authentication and authorization mechanisms in mobile application APIs to identify broken authentication, insecure token management, session fixation, privilege escalation, and IDOR vulnerabilities. Use when performing API security assessments against mobile app backends, testing JWT implementations, evaluating OAuth flows, or assessing session management. Activates for requests involving mobile API auth testing, token security assessment, OAuth mobile flow testing, or API authorization bypass.","domain":"cybersecurity","path":"skills/testing-mobile-api-authentication"},{"name":"testing-oauth2-implementation-flaws","description":"Tests OAuth 2.0 and OpenID Connect implementations for authorization code interception, redirect URI manipulation, CSRF in OAuth flows, token leakage, scope escalation, and PKCE bypass, using Burp Suite Professional and the EsPReSSO extension to probe the authorization server, client, and token handling. Use when assessing OAuth2/OIDC flows or SSO systems for misconfigurations enabling account takeover.","domain":"cybersecurity","path":"skills/testing-oauth2-implementation-flaws"},{"name":"testing-prompt-injection-in-rag-pipelines","description":"Probes Retrieval-Augmented Generation pipelines for indirect prompt injection via poisoned retrieved documents and embedding-space manipulation, using NVIDIA garak, Promptfoo red-team plugins, and Microsoft PyRIT against vector stores like FAISS, Chroma, Pinecone, or pgvector. Use when security-testing a RAG chatbot or document-Q&A system, validating retrieval guardrails, or gating CI/CD on prompt-template/retriever changes.","domain":"cybersecurity","path":"skills/testing-prompt-injection-in-rag-pipelines"},{"name":"testing-ransomware-recovery-procedures","description":"Tests and validates ransomware recovery procedures - backup restore operations (e.g. with Restic), RTO/RPO target verification, recovery sequencing, and clean-restore validation - to confirm organizational resilience against destructive ransomware attacks. Use when validating that recovery plans work under realistic conditions, measuring RTO/RPO against business requirements, or testing restore integrity after simulated encryption.","domain":"cybersecurity","path":"skills/testing-ransomware-recovery-procedures"},{"name":"testing-websocket-api-security","description":"Tests WebSocket API implementations for missing upgrade-handshake authentication, Cross-Site WebSocket Hijacking (CSWSH), message injection, insufficient input validation, message-flooding DoS, and information leakage, using Burp Suite's WebSocket interception and the wscat CLI to craft malicious payloads. Use for real-time API penetration testing or CSWSH/authorization-bypass assessments on WebSocket channels.","domain":"cybersecurity","path":"skills/testing-websocket-api-security"},{"name":"tracking-threat-actor-infrastructure","description":"Discovers and maps adversary-controlled infrastructure (C2 servers, phishing domains, exploit-kit hosts, bulletproof hosting) by pivoting across passive DNS, certificate transparency logs, Shodan/Censys scans, WHOIS records, and network fingerprints (JARM/JA3S). Use when tracking threat actor infrastructure, expanding a known IOC into related assets, or producing STIX-based threat intelligence during a CTI investigation.","domain":"cybersecurity","path":"skills/tracking-threat-actor-infrastructure"},{"name":"triaging-security-alerts-in-splunk","description":"Triages security alerts in Splunk Enterprise Security by classifying severity, investigating notable events, correlating related telemetry, and making escalation or closure decisions using SPL queries and the Incident Review dashboard. Use when SOC analysts face queued alerts from correlation searches, need to prioritize investigation order, or must document triage decisions for handoff to Tier 2/3 analysts.","domain":"cybersecurity","path":"skills/triaging-security-alerts-in-splunk"},{"name":"triaging-security-incident","description":"Performs initial triage of security incidents using the NIST SP 800-61r3 and SANS PICERL frameworks, classifying incident type, assigning priority by business impact, and routing to the appropriate response team. Use when a SIEM/EDR alert needs human classification, concurrent alerts must be prioritized, or a user report or threat-intel IOC match requires initial incident categorization.","domain":"cybersecurity","path":"skills/triaging-security-incident"},{"name":"triaging-security-incident-with-ir-playbook","description":"Classifies and prioritizes security incidents using structured IR playbooks and SIEM/case-management queries (Splunk, TheHive) to determine severity, assign response teams, and initiate the appropriate response procedures. Use when a new SOC alert needs triage, multiple concurrent incidents require prioritization, or automated triage rules need validation or tuning.","domain":"cybersecurity","path":"skills/triaging-security-incident-with-ir-playbook"},{"name":"triaging-vulnerabilities-with-ssvc-framework","description":"Triages and prioritizes vulnerabilities with CISA's Stakeholder-Specific Vulnerability Categorization (SSVC) decision tree, weighing exploitation status (via the CISA KEV catalog and FIRST EPSS API), technical impact, automatability, and mission prevalence to output Track/Track*/Attend/Act decisions. Use when prioritizing vulnerability scan results (OpenVAS, Nessus, Qualys) for remediation planning beyond raw CVSS scores.","domain":"cybersecurity","path":"skills/triaging-vulnerabilities-with-ssvc-framework"},{"name":"triaging-windows-with-kape","description":"Runs KAPE (Kroll Artifact Parser and Extractor) to collect targeted forensic artifacts (registry hives, $MFT, event logs, prefetch, browser data) via Targets and parse them with Modules wrapping Eric Zimmerman's EZ Tools (PECmd, MFTECmd, RECmd). Use during early incident containment/triage when full disk imaging is impractical but a defensible, parseable Windows artifact set is needed quickly, including at-scale remote collection.","domain":"cybersecurity","path":"skills/triaging-windows-with-kape"},{"name":"validating-backup-integrity-for-recovery","description":"Validates backup integrity through cryptographic hash verification, automated restore testing, corruption detection, and recoverability checks to confirm backups are reliable for disaster recovery and ransomware response. Use before relying on backups for recovery, when building post-backup validation pipelines, auditing backup infrastructure for compliance, or checking immutable/air-gapped backups for silent corruption or tampering.","domain":"cybersecurity","path":"skills/validating-backup-integrity-for-recovery"},{"name":"validating-tpm-measured-boot-attestation","description":"Verifies TPM 2.0 measured-boot integrity and remote attestation with tpm2-tools -- reading PCRs (tpm2_pcrread), replaying the boot event log, generating and checking signed quotes (tpm2_quote/tpm2_checkquote), and sealing secrets to a PCR policy. Use to confirm a system booted trusted firmware/kernel for Zero Trust device posture, detect boot-chain tampering via PCR drift, or build a golden-value baseline for fleet attestation.","domain":"cybersecurity","path":"skills/validating-tpm-measured-boot-attestation"},{"name":"verifying-build-provenance-with-slsa-sigstore","description":"Verifies artifact signatures and SLSA provenance using Sigstore's cosign (verify, verify-attestation, verify-blob-attestation) and slsa-verifier (verify-artifact), enforcing keyless OIDC builder identity and source repo against SLSA Build levels. Use in CI/CD before deploying artifacts, when consuming third-party attestations, establishing a SLSA Build L3 pipeline, or confirming provenance during incident response or admission control.","domain":"cybersecurity","path":"skills/verifying-build-provenance-with-slsa-sigstore"}]}
===== END FILE =====


===== FILE: ./mappings/README.md =====
# MITRE ATT&CK Navigator Layer - Anthropic Cybersecurity Skills

This directory contains a MITRE ATT&CK Navigator layer file that maps the coverage of the Anthropic Cybersecurity Skills repository against the ATT&CK Enterprise matrix.

## Files

| File | Description |
|------|-------------|
| `attack-navigator-layer.json` | ATT&CK Navigator layer (v4.5 format, Enterprise ATT&CK v14) |

## How to View

1. Open the [MITRE ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/)
2. Click **Open Existing Layer**
3. Select **Upload from local** and choose `attack-navigator-layer.json`
4. The matrix will display with blue-shaded techniques indicating coverage

Alternatively, paste the raw JSON URL into the Navigator's "Load from URL" option if this file is hosted publicly.

## Coverage Statistics

| Metric | Value |
|--------|-------|
| Total skills scanned | 817 |
| Unique ATT&CK techniques referenced | 218 |
| Parent techniques | 94 |
| Sub-techniques | 124 |
| Tactics with coverage | 14/14 |

## Coverage by Tactic

| Tactic | Techniques Covered |
|--------|-------------------|
| Defense Evasion | 36 |
| Credential Access | 33 |
| Persistence | 29 |
| Initial Access | 17 |
| Command and Control | 17 |
| Privilege Escalation | 13 |
| Discovery | 12 |
| Exfiltration | 12 |
| Reconnaissance | 11 |
| Collection | 10 |
| Lateral Movement | 9 |
| Execution | 8 |
| Resource Development | 6 |
| Impact | 5 |

## Color Scale

The layer uses a blue gradient to indicate coverage depth:

- **Light blue** (`#cfe2f3`): 1-2 skills reference this technique
- **Medium blue** (`#6fa8dc`): 3-5 skills reference this technique
- **Dark blue** (`#3d85c6`): 6-10 skills reference this technique
- **Deep blue** (`#1155cc`): 11+ skills reference this technique

## Top 10 Most Covered Techniques

| Technique | Name | Skills |
|-----------|------|--------|
| T1059.001 | PowerShell | 26 |
| T1055 | Process Injection | 17 |
| T1053.005 | Scheduled Task | 16 |
| T1566.001 | Spearphishing Attachment | 15 |
| T1558.003 | Kerberoasting | 14 |
| T1547.001 | Registry Run Keys / Startup Folder | 13 |
| T1078 | Valid Accounts | 13 |
| T1003.006 | DCSync | 13 |
| T1071.001 | Web Protocols | 12 |
| T1021.002 | SMB/Windows Admin Shares | 12 |

## Methodology

Techniques were extracted by scanning all `SKILL.md` files in the repository for ATT&CK technique ID patterns (`T1XXX` and `T1XXX.XXX`). Each technique's score is proportional to the number of distinct skills that reference it, normalized to a 1-100 scale.

## Layer Format

- **Format version**: 4.5
- **ATT&CK version**: 14 (Enterprise)
- **Navigator version**: 4.9.1
- **Domain**: enterprise-attack

## Related Links

- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/)
- [ATT&CK Navigator GitHub](https://github.com/mitre-attack/attack-navigator)

===== END FILE =====


===== FILE: ./mappings/attack-navigator-layer.json =====
{
  "name": "Anthropic Cybersecurity Skills - ATT&CK Coverage",
  "versions": {
    "attack": "14",
    "navigator": "4.9.1",
    "layer": "4.5"
  },
  "domain": "enterprise-attack",
  "description": "MITRE ATT&CK technique coverage map for the Anthropic Cybersecurity Skills repository. Each technique is scored by the number of skills that reference it. Higher scores (darker colors) indicate more comprehensive coverage across multiple training skills.",
  "filters": {
    "platforms": [
      "Linux",
      "macOS",
      "Windows",
      "Network",
      "PRE",
      "Containers",
      "Office 365",
      "SaaS",
      "Google Workspace",
      "IaaS",
      "Azure AD"
    ]
  },
  "sorting": 3,
  "layout": {
    "layout": "side",
    "showID": true,
    "showName": true,
    "showAggregateScores": false,
    "countUnscored": false,
    "aggregateFunction": "average",
    "expandedSubtechniques": "annotated"
  },
  "hideDisabled": false,
  "techniques": [
    {
      "techniqueID": "T1003",
      "score": 42,
      "comment": "OS Credential Dumping - Referenced in 11 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "11"
        },
        {
          "name": "skills",
          "value": "building-attack-pattern-library-from-cti-reports, building-detection-rules-with-sigma, detecting-container-escape-with-falco-rules, detecting-credential-dumping-techniques, detecting-credential-dumping-with-edr (+6 more)"
        }
      ]
    },
    {
      "techniqueID": "T1003.001",
      "score": 46,
      "comment": "LSASS Memory - Referenced in 12 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "12"
        },
        {
          "name": "skills",
          "value": "building-attack-pattern-library-from-cti-reports, building-detection-rule-with-splunk-spl, building-detection-rules-with-sigma, conducting-full-scope-red-team-engagement, conducting-internal-network-penetration-test (+7 more)"
        }
      ]
    },
    {
      "techniqueID": "T1003.002",
      "score": 8,
      "comment": "SAM - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-credential-dumping-with-edr, detecting-t1003-credential-dumping-with-edr"
        }
      ]
    },
    {
      "techniqueID": "T1003.003",
      "score": 8,
      "comment": "NTDS - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-credential-dumping-with-edr, detecting-t1003-credential-dumping-with-edr"
        }
      ]
    },
    {
      "techniqueID": "T1003.004",
      "score": 12,
      "comment": "LSA Secrets - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-credential-dumping-with-edr, detecting-t1003-credential-dumping-with-edr, performing-credential-access-with-lazagne"
        }
      ]
    },
    {
      "techniqueID": "T1003.005",
      "score": 8,
      "comment": "Cached Domain Credentials - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-credential-dumping-with-edr, detecting-t1003-credential-dumping-with-edr"
        }
      ]
    },
    {
      "techniqueID": "T1003.006",
      "score": 50,
      "comment": "DCSync - Referenced in 13 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "13"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, conducting-domain-persistence-with-dcsync, conducting-full-scope-red-team-engagement, conducting-internal-network-penetration-test, detecting-credential-dumping-with-edr (+8 more)"
        }
      ]
    },
    {
      "techniqueID": "T1005",
      "score": 8,
      "comment": "Data from Local System - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "conducting-malware-incident-response, detecting-container-escape-with-falco-rules"
        }
      ]
    },
    {
      "techniqueID": "T1016",
      "score": 12,
      "comment": "System Network Configuration Discovery - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, conducting-internal-reconnaissance-with-bloodhound-ce, exploiting-active-directory-with-bloodhound"
        }
      ]
    },
    {
      "techniqueID": "T1018",
      "score": 15,
      "comment": "Remote System Discovery - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, conducting-internal-reconnaissance-with-bloodhound-ce, exploiting-active-directory-with-bloodhound, performing-active-directory-bloodhound-analysis"
        }
      ]
    },
    {
      "techniqueID": "T1020",
      "score": 4,
      "comment": "Automated Exfiltration - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-data-exfiltration-indicators"
        }
      ]
    },
    {
      "techniqueID": "T1021",
      "score": 38,
      "comment": "Remote Services - Referenced in 10 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "10"
        },
        {
          "name": "skills",
          "value": "detecting-lateral-movement-in-network, detecting-lateral-movement-with-splunk, detecting-service-account-abuse, exploiting-constrained-delegation-abuse, implementing-continuous-security-validation-with-bas (+5 more)"
        }
      ]
    },
    {
      "techniqueID": "T1021.001",
      "score": 31,
      "comment": "Remote Desktop Protocol - Referenced in 8 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "8"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, building-attack-pattern-library-from-cti-reports, detecting-lateral-movement-with-splunk, executing-red-team-exercise, implementing-mitre-attack-coverage-mapping (+3 more)"
        }
      ]
    },
    {
      "techniqueID": "T1021.002",
      "score": 46,
      "comment": "SMB/Windows Admin Shares - Referenced in 12 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "12"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, building-attack-pattern-library-from-cti-reports, building-detection-rule-with-splunk-spl, conducting-full-scope-red-team-engagement, conducting-internal-network-penetration-test (+7 more)"
        }
      ]
    },
    {
      "techniqueID": "T1021.003",
      "score": 12,
      "comment": "DCOM - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-lateral-movement-with-splunk, performing-lateral-movement-detection, performing-lateral-movement-with-wmiexec"
        }
      ]
    },
    {
      "techniqueID": "T1021.004",
      "score": 4,
      "comment": "SSH - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-lateral-movement-with-splunk"
        }
      ]
    },
    {
      "techniqueID": "T1021.006",
      "score": 12,
      "comment": "Windows Remote Management - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "building-attack-pattern-library-from-cti-reports, detecting-lateral-movement-with-splunk, performing-lateral-movement-detection"
        }
      ]
    },
    {
      "techniqueID": "T1027",
      "score": 8,
      "comment": "Obfuscated Files or Information - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, conducting-full-scope-red-team-engagement"
        }
      ]
    },
    {
      "techniqueID": "T1029",
      "score": 4,
      "comment": "Scheduled Transfer - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-data-exfiltration-indicators"
        }
      ]
    },
    {
      "techniqueID": "T1030",
      "score": 4,
      "comment": "Data Transfer Size Limits - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-data-exfiltration-indicators"
        }
      ]
    },
    {
      "techniqueID": "T1033",
      "score": 8,
      "comment": "System Owner/User Discovery - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "conducting-internal-reconnaissance-with-bloodhound-ce, exploiting-active-directory-with-bloodhound"
        }
      ]
    },
    {
      "techniqueID": "T1036",
      "score": 12,
      "comment": "Masquerading - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-evasion-techniques-in-endpoint-logs, implementing-mitre-attack-coverage-mapping, implementing-siem-use-cases-for-detection"
        }
      ]
    },
    {
      "techniqueID": "T1036.005",
      "score": 4,
      "comment": "Match Legitimate Name or Location - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-process-injection-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1040",
      "score": 4,
      "comment": "Network Sniffing - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "implementing-continuous-security-validation-with-bas"
        }
      ]
    },
    {
      "techniqueID": "T1041",
      "score": 35,
      "comment": "Exfiltration Over C2 Channel - Referenced in 9 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "9"
        },
        {
          "name": "skills",
          "value": "analyzing-ransomware-network-indicators, building-attack-pattern-library-from-cti-reports, conducting-full-scope-red-team-engagement, conducting-malware-incident-response, executing-red-team-exercise (+4 more)"
        }
      ]
    },
    {
      "techniqueID": "T1047",
      "score": 19,
      "comment": "Windows Management Instrumentation - Referenced in 5 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "5"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, detecting-lateral-movement-with-splunk, performing-lateral-movement-detection, performing-lateral-movement-with-wmiexec, performing-purple-team-exercise"
        }
      ]
    },
    {
      "techniqueID": "T1048",
      "score": 19,
      "comment": "Exfiltration Over Alternative Protocol - Referenced in 5 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "5"
        },
        {
          "name": "skills",
          "value": "building-detection-rule-with-splunk-spl, conducting-full-scope-red-team-engagement, hunting-for-data-exfiltration-indicators, implementing-continuous-security-validation-with-bas, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1048.001",
      "score": 4,
      "comment": "Symmetric Encrypted Non-C2 - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-data-exfiltration-indicators"
        }
      ]
    },
    {
      "techniqueID": "T1048.002",
      "score": 4,
      "comment": "Asymmetric Encrypted Non-C2 - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-data-exfiltration-indicators"
        }
      ]
    },
    {
      "techniqueID": "T1048.003",
      "score": 19,
      "comment": "Unencrypted/Obfuscated Non-C2 - Referenced in 5 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "5"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, hunting-for-data-exfiltration-indicators, hunting-for-dns-tunneling-with-zeek, implementing-continuous-security-validation-with-bas, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1052",
      "score": 4,
      "comment": "Exfiltration Over Physical Medium - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-data-exfiltration-indicators"
        }
      ]
    },
    {
      "techniqueID": "T1053",
      "score": 23,
      "comment": "Scheduled Task/Job - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, analyzing-persistence-mechanisms-in-linux, hunting-for-persistence-mechanisms-in-windows, implementing-mitre-attack-coverage-mapping, implementing-siem-use-cases-for-detection (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1053.002",
      "score": 4,
      "comment": "At - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-scheduled-task-persistence"
        }
      ]
    },
    {
      "techniqueID": "T1053.003",
      "score": 8,
      "comment": "Cron - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-scheduled-task-persistence, performing-privilege-escalation-on-linux"
        }
      ]
    },
    {
      "techniqueID": "T1053.005",
      "score": 62,
      "comment": "Scheduled Task - Referenced in 16 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "16"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, analyzing-windows-event-logs-in-splunk, building-attack-pattern-library-from-cti-reports, building-detection-rule-with-splunk-spl, conducting-full-scope-red-team-engagement (+11 more)"
        }
      ]
    },
    {
      "techniqueID": "T1055",
      "score": 65,
      "comment": "Process Injection - Referenced in 17 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "17"
        },
        {
          "name": "skills",
          "value": "building-attack-pattern-library-from-cti-reports, building-red-team-c2-infrastructure-with-havoc, conducting-full-scope-red-team-engagement, detecting-evasion-techniques-in-endpoint-logs, detecting-process-hollowing-technique (+12 more)"
        }
      ]
    },
    {
      "techniqueID": "T1055.001",
      "score": 15,
      "comment": "DLL Injection - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "detecting-process-hollowing-technique, detecting-process-injection-techniques, detecting-t1055-process-injection-with-sysmon, hunting-for-process-injection-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1055.002",
      "score": 8,
      "comment": "Portable Executable Injection - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-process-injection-techniques, detecting-t1055-process-injection-with-sysmon"
        }
      ]
    },
    {
      "techniqueID": "T1055.003",
      "score": 12,
      "comment": "Thread Execution Hijacking - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-process-hollowing-technique, detecting-process-injection-techniques, detecting-t1055-process-injection-with-sysmon"
        }
      ]
    },
    {
      "techniqueID": "T1055.004",
      "score": 12,
      "comment": "APC Injection - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-process-hollowing-technique, detecting-process-injection-techniques, detecting-t1055-process-injection-with-sysmon"
        }
      ]
    },
    {
      "techniqueID": "T1055.005",
      "score": 8,
      "comment": "Thread Local Storage - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-process-injection-techniques, detecting-t1055-process-injection-with-sysmon"
        }
      ]
    },
    {
      "techniqueID": "T1055.008",
      "score": 4,
      "comment": "Ptrace System Calls - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-process-injection-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1055.009",
      "score": 4,
      "comment": "Proc Memory - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-process-injection-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1055.011",
      "score": 4,
      "comment": "Extra Window Memory Injection - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-process-injection-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1055.012",
      "score": 23,
      "comment": "Process Hollowing - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "conducting-malware-incident-response, detecting-fileless-malware-techniques, detecting-process-hollowing-technique, detecting-process-injection-techniques, detecting-t1055-process-injection-with-sysmon (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1055.013",
      "score": 12,
      "comment": "Process Doppelganging - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-process-hollowing-technique, detecting-process-injection-techniques, detecting-t1055-process-injection-with-sysmon"
        }
      ]
    },
    {
      "techniqueID": "T1055.014",
      "score": 4,
      "comment": "VDSO Hijacking - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-process-injection-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1055.015",
      "score": 8,
      "comment": "ListPlanting - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-process-injection-techniques, detecting-t1055-process-injection-with-sysmon"
        }
      ]
    },
    {
      "techniqueID": "T1059",
      "score": 38,
      "comment": "Command and Scripting Interpreter - Referenced in 10 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "10"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, analyzing-threat-actor-ttps-with-mitre-attack, analyzing-windows-event-logs-in-splunk, building-incident-timeline-with-timesketch, deobfuscating-powershell-obfuscated-malware (+5 more)"
        }
      ]
    },
    {
      "techniqueID": "T1059.001",
      "score": 100,
      "comment": "PowerShell - Referenced in 26 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "26"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, analyzing-macro-malware-in-office-documents, analyzing-powershell-empire-artifacts, analyzing-security-logs-with-splunk, analyzing-threat-actor-ttps-with-mitre-navigator (+21 more)"
        }
      ]
    },
    {
      "techniqueID": "T1059.003",
      "score": 12,
      "comment": "Windows Command Shell - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "building-attack-pattern-library-from-cti-reports, detecting-suspicious-powershell-execution, mapping-mitre-attack-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1059.005",
      "score": 15,
      "comment": "Visual Basic - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "analyzing-macro-malware-in-office-documents, executing-red-team-exercise, hunting-for-lolbins-execution-in-endpoint-logs, mapping-mitre-attack-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1068",
      "score": 31,
      "comment": "Exploitation for Privilege Escalation - Referenced in 8 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "8"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, detecting-container-escape-attempts, detecting-privilege-escalation-attempts, detecting-privilege-escalation-in-kubernetes-pods, exploiting-nopac-cve-2021-42278-42287 (+3 more)"
        }
      ]
    },
    {
      "techniqueID": "T1069.001",
      "score": 4,
      "comment": "Local Groups - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-active-directory-bloodhound-analysis"
        }
      ]
    },
    {
      "techniqueID": "T1069.002",
      "score": 15,
      "comment": "Domain Groups - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "conducting-internal-reconnaissance-with-bloodhound-ce, exploiting-active-directory-with-bloodhound, performing-active-directory-bloodhound-analysis, performing-kerberoasting-attack"
        }
      ]
    },
    {
      "techniqueID": "T1070",
      "score": 12,
      "comment": "Indicator Removal - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-evasion-techniques-in-endpoint-logs, implementing-siem-use-cases-for-detection, implementing-velociraptor-for-ir-collection"
        }
      ]
    },
    {
      "techniqueID": "T1070.001",
      "score": 12,
      "comment": "Clear Windows Event Logs - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-evasion-techniques-in-endpoint-logs, implementing-mitre-attack-coverage-mapping, performing-purple-team-exercise"
        }
      ]
    },
    {
      "techniqueID": "T1070.004",
      "score": 4,
      "comment": "File Deletion - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "implementing-threat-modeling-with-mitre-attack"
        }
      ]
    },
    {
      "techniqueID": "T1070.006",
      "score": 8,
      "comment": "Timestomping - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-evasion-techniques-in-endpoint-logs, hunting-for-defense-evasion-via-timestomping"
        }
      ]
    },
    {
      "techniqueID": "T1071",
      "score": 38,
      "comment": "Application Layer Protocol - Referenced in 10 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "10"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, analyzing-network-covert-channels-in-malware, analyzing-ransomware-network-indicators, analyzing-threat-actor-ttps-with-mitre-attack, hunting-advanced-persistent-threats (+5 more)"
        }
      ]
    },
    {
      "techniqueID": "T1071.001",
      "score": 46,
      "comment": "Web Protocols - Referenced in 12 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "12"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, building-c2-infrastructure-with-sliver-framework, building-red-team-c2-infrastructure-with-havoc, conducting-malware-incident-response, detecting-process-injection-techniques (+7 more)"
        }
      ]
    },
    {
      "techniqueID": "T1071.004",
      "score": 27,
      "comment": "DNS - Referenced in 7 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "7"
        },
        {
          "name": "skills",
          "value": "building-attack-pattern-library-from-cti-reports, building-c2-infrastructure-with-sliver-framework, hunting-for-beaconing-with-frequency-analysis, hunting-for-command-and-control-beaconing, hunting-for-dns-tunneling-with-zeek (+2 more)"
        }
      ]
    },
    {
      "techniqueID": "T1074",
      "score": 12,
      "comment": "Data Staged - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "building-attack-pattern-library-from-cti-reports, executing-red-team-exercise, hunting-for-data-staging-before-exfiltration"
        }
      ]
    },
    {
      "techniqueID": "T1074.001",
      "score": 4,
      "comment": "Local Data Staging - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-data-staging-before-exfiltration"
        }
      ]
    },
    {
      "techniqueID": "T1074.002",
      "score": 4,
      "comment": "Remote Data Staging - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-data-staging-before-exfiltration"
        }
      ]
    },
    {
      "techniqueID": "T1078",
      "score": 50,
      "comment": "Valid Accounts - Referenced in 13 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "13"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, analyzing-windows-event-logs-in-splunk, conducting-full-scope-red-team-engagement, conducting-internal-network-penetration-test, detecting-insider-threat-behaviors (+8 more)"
        }
      ]
    },
    {
      "techniqueID": "T1078.001",
      "score": 4,
      "comment": "Default Accounts - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-service-account-abuse"
        }
      ]
    },
    {
      "techniqueID": "T1078.002",
      "score": 23,
      "comment": "Domain Accounts - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "conducting-domain-persistence-with-dcsync, detecting-service-account-abuse, exploiting-active-directory-certificate-services-esc1, exploiting-constrained-delegation-abuse, exploiting-nopac-cve-2021-42278-42287 (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1078.004",
      "score": 12,
      "comment": "Cloud Accounts - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-azure-service-principal-abuse, implementing-mitre-attack-coverage-mapping, implementing-threat-modeling-with-mitre-attack"
        }
      ]
    },
    {
      "techniqueID": "T1082",
      "score": 4,
      "comment": "System Information Discovery - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement"
        }
      ]
    },
    {
      "techniqueID": "T1087",
      "score": 8,
      "comment": "Account Discovery - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, implementing-continuous-security-validation-with-bas"
        }
      ]
    },
    {
      "techniqueID": "T1087.002",
      "score": 23,
      "comment": "Domain Account - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "conducting-internal-reconnaissance-with-bloodhound-ce, exploiting-active-directory-certificate-services-esc1, exploiting-active-directory-with-bloodhound, exploiting-kerberoasting-with-impacket, performing-active-directory-bloodhound-analysis (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1087.004",
      "score": 8,
      "comment": "Cloud Account - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-azure-service-principal-abuse, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1090",
      "score": 4,
      "comment": "Proxy - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1090.002",
      "score": 8,
      "comment": "External Proxy - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "building-c2-infrastructure-with-sliver-framework, building-red-team-c2-infrastructure-with-havoc"
        }
      ]
    },
    {
      "techniqueID": "T1090.004",
      "score": 4,
      "comment": "Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-domain-fronting-c2-traffic"
        }
      ]
    },
    {
      "techniqueID": "T1091",
      "score": 4,
      "comment": "Replication Through Removable Media - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-physical-intrusion-assessment"
        }
      ]
    },
    {
      "techniqueID": "T1095",
      "score": 8,
      "comment": "Non-Application Layer Protocol - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-command-and-control-beaconing, hunting-for-unusual-network-connections"
        }
      ]
    },
    {
      "techniqueID": "T1098",
      "score": 19,
      "comment": "Account Manipulation - Referenced in 5 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "5"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, conducting-domain-persistence-with-dcsync, hunting-for-t1098-account-manipulation, implementing-mitre-attack-coverage-mapping, performing-active-directory-compromise-investigation"
        }
      ]
    },
    {
      "techniqueID": "T1098.001",
      "score": 12,
      "comment": "Additional Cloud Credentials - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "conducting-cloud-penetration-testing, detecting-azure-service-principal-abuse, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1098.002",
      "score": 4,
      "comment": "Additional Email Delegate Permissions - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-email-forwarding-rules-attack"
        }
      ]
    },
    {
      "techniqueID": "T1102",
      "score": 4,
      "comment": "Web Service - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-living-off-the-cloud-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1105",
      "score": 23,
      "comment": "Ingress Tool Transfer - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "building-c2-infrastructure-with-sliver-framework, building-red-team-c2-infrastructure-with-havoc, detecting-living-off-the-land-with-lolbas, implementing-mitre-attack-coverage-mapping, implementing-siem-use-cases-for-detection (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1110",
      "score": 15,
      "comment": "Brute Force - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, conducting-internal-network-penetration-test, implementing-mitre-attack-coverage-mapping, performing-alert-triage-with-elastic-siem"
        }
      ]
    },
    {
      "techniqueID": "T1110.001",
      "score": 15,
      "comment": "Password Guessing - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, building-detection-rule-with-splunk-spl, implementing-siem-use-cases-for-detection, performing-false-positive-reduction-in-siem"
        }
      ]
    },
    {
      "techniqueID": "T1110.002",
      "score": 4,
      "comment": "Password Cracking - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "exploiting-kerberoasting-with-impacket"
        }
      ]
    },
    {
      "techniqueID": "T1110.003",
      "score": 4,
      "comment": "Password Spraying - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "implementing-siem-use-cases-for-detection"
        }
      ]
    },
    {
      "techniqueID": "T1112",
      "score": 4,
      "comment": "Modify Registry - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-fileless-malware-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1114.002",
      "score": 4,
      "comment": "Remote Email Collection - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-email-forwarding-rules-attack"
        }
      ]
    },
    {
      "techniqueID": "T1114.003",
      "score": 8,
      "comment": "Email Forwarding Rule - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-business-email-compromise, detecting-email-forwarding-rules-attack"
        }
      ]
    },
    {
      "techniqueID": "T1127",
      "score": 8,
      "comment": "Trusted Developer Utilities Proxy Execution - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-evasion-techniques-in-endpoint-logs, detecting-living-off-the-land-with-lolbas"
        }
      ]
    },
    {
      "techniqueID": "T1127.001",
      "score": 4,
      "comment": "MSBuild - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-lolbins-execution-in-endpoint-logs"
        }
      ]
    },
    {
      "techniqueID": "T1132",
      "score": 4,
      "comment": "Data Encoding - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-command-and-control-beaconing"
        }
      ]
    },
    {
      "techniqueID": "T1132.001",
      "score": 4,
      "comment": "Standard Encoding - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "building-c2-infrastructure-with-sliver-framework"
        }
      ]
    },
    {
      "techniqueID": "T1133",
      "score": 4,
      "comment": "External Remote Services - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-threat-landscape-assessment-for-sector"
        }
      ]
    },
    {
      "techniqueID": "T1134",
      "score": 8,
      "comment": "Access Token Manipulation - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, detecting-privilege-escalation-attempts"
        }
      ]
    },
    {
      "techniqueID": "T1134.001",
      "score": 4,
      "comment": "Token Impersonation/Theft - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "exploiting-constrained-delegation-abuse"
        }
      ]
    },
    {
      "techniqueID": "T1134.005",
      "score": 4,
      "comment": "SID-History Injection - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-active-directory-compromise-investigation"
        }
      ]
    },
    {
      "techniqueID": "T1136",
      "score": 8,
      "comment": "Create Account - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-privilege-escalation-in-kubernetes-pods, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1136.001",
      "score": 4,
      "comment": "Local Account - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk"
        }
      ]
    },
    {
      "techniqueID": "T1136.002",
      "score": 4,
      "comment": "Domain Account - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "exploiting-nopac-cve-2021-42278-42287"
        }
      ]
    },
    {
      "techniqueID": "T1140",
      "score": 12,
      "comment": "Deobfuscate/Decode Files or Information - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-living-off-the-land-with-lolbas, hunting-for-living-off-the-land-binaries, hunting-for-lolbins-execution-in-endpoint-logs"
        }
      ]
    },
    {
      "techniqueID": "T1190",
      "score": 15,
      "comment": "Exploit Public-Facing Application - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, exploiting-ms17-010-eternalblue-vulnerability, hunting-for-webshell-activity, performing-threat-landscape-assessment-for-sector"
        }
      ]
    },
    {
      "techniqueID": "T1195",
      "score": 8,
      "comment": "Supply Chain Compromise - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "analyzing-supply-chain-malware-artifacts, performing-threat-landscape-assessment-for-sector"
        }
      ]
    },
    {
      "techniqueID": "T1195.001",
      "score": 4,
      "comment": "Compromise Software Dependencies - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-supply-chain-compromise"
        }
      ]
    },
    {
      "techniqueID": "T1195.002",
      "score": 4,
      "comment": "Compromise Software Supply Chain - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-supply-chain-compromise"
        }
      ]
    },
    {
      "techniqueID": "T1197",
      "score": 8,
      "comment": "BITS Jobs - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-living-off-the-land-binaries, hunting-for-lolbins-execution-in-endpoint-logs"
        }
      ]
    },
    {
      "techniqueID": "T1199",
      "score": 8,
      "comment": "Trusted Relationship - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-supply-chain-compromise, performing-physical-intrusion-assessment"
        }
      ]
    },
    {
      "techniqueID": "T1200",
      "score": 4,
      "comment": "Hardware Additions - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-physical-intrusion-assessment"
        }
      ]
    },
    {
      "techniqueID": "T1204.001",
      "score": 4,
      "comment": "Malicious Link - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-spearphishing-simulation-campaign"
        }
      ]
    },
    {
      "techniqueID": "T1204.002",
      "score": 23,
      "comment": "Malicious File - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "analyzing-macro-malware-in-office-documents, conducting-full-scope-red-team-engagement, conducting-spearphishing-simulation-campaign, implementing-siem-use-cases-for-detection, performing-dynamic-analysis-with-any-run (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1210",
      "score": 8,
      "comment": "Exploitation of Remote Services - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "exploiting-ms17-010-eternalblue-vulnerability, exploiting-zerologon-vulnerability-cve-2020-1472"
        }
      ]
    },
    {
      "techniqueID": "T1213",
      "score": 4,
      "comment": "Data from Information Repositories - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement"
        }
      ]
    },
    {
      "techniqueID": "T1218",
      "score": 23,
      "comment": "System Binary Proxy Execution - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "detecting-evasion-techniques-in-endpoint-logs, detecting-living-off-the-land-with-lolbas, hunting-advanced-persistent-threats, hunting-for-living-off-the-land-binaries, hunting-for-lolbins-execution-in-endpoint-logs (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1218.001",
      "score": 8,
      "comment": "Compiled HTML File - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-living-off-the-land-binaries, hunting-for-lolbins-execution-in-endpoint-logs"
        }
      ]
    },
    {
      "techniqueID": "T1218.002",
      "score": 4,
      "comment": "Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-living-off-the-land-binaries"
        }
      ]
    },
    {
      "techniqueID": "T1218.003",
      "score": 8,
      "comment": "CMSTP - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-living-off-the-land-binaries, hunting-for-lolbins-execution-in-endpoint-logs"
        }
      ]
    },
    {
      "techniqueID": "T1218.005",
      "score": 12,
      "comment": "Mshta - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-fileless-malware-techniques, hunting-for-living-off-the-land-binaries, hunting-for-lolbins-execution-in-endpoint-logs"
        }
      ]
    },
    {
      "techniqueID": "T1218.010",
      "score": 8,
      "comment": "Regsvr32 - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-living-off-the-land-binaries, hunting-for-lolbins-execution-in-endpoint-logs"
        }
      ]
    },
    {
      "techniqueID": "T1218.011",
      "score": 12,
      "comment": "Rundll32 - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "hunting-for-living-off-the-land-binaries, hunting-for-lolbins-execution-in-endpoint-logs, performing-dynamic-analysis-with-any-run"
        }
      ]
    },
    {
      "techniqueID": "T1222.001",
      "score": 4,
      "comment": "Windows - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-domain-persistence-with-dcsync"
        }
      ]
    },
    {
      "techniqueID": "T1482",
      "score": 12,
      "comment": "Domain Trust Discovery - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "conducting-internal-reconnaissance-with-bloodhound-ce, exploiting-active-directory-with-bloodhound, performing-active-directory-bloodhound-analysis"
        }
      ]
    },
    {
      "techniqueID": "T1484",
      "score": 8,
      "comment": "Domain Policy Modification - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "exploiting-active-directory-certificate-services-esc1, performing-active-directory-vulnerability-assessment"
        }
      ]
    },
    {
      "techniqueID": "T1484.001",
      "score": 4,
      "comment": "Group Policy Modification - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-active-directory-compromise-investigation"
        }
      ]
    },
    {
      "techniqueID": "T1485",
      "score": 4,
      "comment": "Data Destruction - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-shadow-copy-deletion"
        }
      ]
    },
    {
      "techniqueID": "T1486",
      "score": 23,
      "comment": "Data Encrypted for Impact - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, hunting-for-shadow-copy-deletion, implementing-honeypot-for-ransomware-detection, implementing-mitre-attack-coverage-mapping, performing-purple-team-exercise (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1489",
      "score": 4,
      "comment": "Service Stop - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement"
        }
      ]
    },
    {
      "techniqueID": "T1490",
      "score": 12,
      "comment": "Inhibit System Recovery - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "building-soc-playbook-for-ransomware, hunting-for-shadow-copy-deletion, performing-purple-team-exercise"
        }
      ]
    },
    {
      "techniqueID": "T1497",
      "score": 4,
      "comment": "Virtualization/Sandbox Evasion - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "analyzing-malware-sandbox-evasion-techniques"
        }
      ]
    },
    {
      "techniqueID": "T1505.003",
      "score": 8,
      "comment": "Web Shell - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "building-attack-pattern-library-from-cti-reports, hunting-for-webshell-activity"
        }
      ]
    },
    {
      "techniqueID": "T1528",
      "score": 4,
      "comment": "Steal Application Access Token - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-azure-service-principal-abuse"
        }
      ]
    },
    {
      "techniqueID": "T1530",
      "score": 12,
      "comment": "Data from Cloud Storage Object - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-insider-threat-behaviors, implementing-mitre-attack-coverage-mapping, performing-cloud-incident-containment-procedures"
        }
      ]
    },
    {
      "techniqueID": "T1534",
      "score": 4,
      "comment": "Internal Spearphishing - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1537",
      "score": 19,
      "comment": "Transfer Data to Cloud Account - Referenced in 5 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "5"
        },
        {
          "name": "skills",
          "value": "hunting-for-data-exfiltration-indicators, hunting-for-living-off-the-cloud-techniques, implementing-mitre-attack-coverage-mapping, implementing-threat-modeling-with-mitre-attack, performing-cloud-incident-containment-procedures"
        }
      ]
    },
    {
      "techniqueID": "T1539",
      "score": 8,
      "comment": "Steal Web Session Cookie - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "performing-credential-access-with-lazagne, performing-initial-access-with-evilginx3"
        }
      ]
    },
    {
      "techniqueID": "T1543",
      "score": 8,
      "comment": "Create or Modify System Process - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "analyzing-persistence-mechanisms-in-linux, hunting-for-persistence-mechanisms-in-windows"
        }
      ]
    },
    {
      "techniqueID": "T1543.002",
      "score": 4,
      "comment": "Systemd Service - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-privilege-escalation-on-linux"
        }
      ]
    },
    {
      "techniqueID": "T1543.003",
      "score": 12,
      "comment": "Windows Service - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "hunting-for-persistence-mechanisms-in-windows, hunting-for-unusual-service-installations, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1546",
      "score": 4,
      "comment": "Event Triggered Execution - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "analyzing-persistence-mechanisms-in-linux"
        }
      ]
    },
    {
      "techniqueID": "T1546.003",
      "score": 19,
      "comment": "WMI Event Subscription - Referenced in 5 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "5"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, detecting-fileless-malware-techniques, detecting-wmi-persistence, hunting-for-persistence-mechanisms-in-windows, hunting-for-persistence-via-wmi-subscriptions"
        }
      ]
    },
    {
      "techniqueID": "T1546.010",
      "score": 4,
      "comment": "AppInit DLLs - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-persistence-mechanisms-in-windows"
        }
      ]
    },
    {
      "techniqueID": "T1546.012",
      "score": 8,
      "comment": "IFEO Injection - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-persistence-mechanisms-in-windows, hunting-for-registry-persistence-mechanisms"
        }
      ]
    },
    {
      "techniqueID": "T1546.015",
      "score": 8,
      "comment": "COM Hijacking - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-persistence-mechanisms-in-windows, hunting-for-registry-persistence-mechanisms"
        }
      ]
    },
    {
      "techniqueID": "T1547",
      "score": 23,
      "comment": "Boot or Logon Autostart Execution - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, analyzing-malware-persistence-with-autoruns, hunting-advanced-persistent-threats, hunting-for-persistence-mechanisms-in-windows, implementing-siem-use-cases-for-detection (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1547.001",
      "score": 50,
      "comment": "Registry Run Keys / Startup Folder - Referenced in 13 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "13"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, analyzing-windows-event-logs-in-splunk, building-attack-pattern-library-from-cti-reports, conducting-full-scope-red-team-engagement, hunting-for-persistence-mechanisms-in-windows (+8 more)"
        }
      ]
    },
    {
      "techniqueID": "T1547.004",
      "score": 8,
      "comment": "Winlogon Helper DLL - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-persistence-mechanisms-in-windows, hunting-for-registry-persistence-mechanisms"
        }
      ]
    },
    {
      "techniqueID": "T1547.005",
      "score": 4,
      "comment": "Security Support Provider - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-persistence-mechanisms-in-windows"
        }
      ]
    },
    {
      "techniqueID": "T1548",
      "score": 15,
      "comment": "Abuse Elevation Control Mechanism - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "detecting-container-escape-attempts, detecting-privilege-escalation-in-kubernetes-pods, detecting-t1548-abuse-elevation-control-mechanism, performing-privilege-escalation-assessment"
        }
      ]
    },
    {
      "techniqueID": "T1548.001",
      "score": 12,
      "comment": "Setuid and Setgid - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-privilege-escalation-in-kubernetes-pods, detecting-t1548-abuse-elevation-control-mechanism, performing-privilege-escalation-on-linux"
        }
      ]
    },
    {
      "techniqueID": "T1548.002",
      "score": 12,
      "comment": "Bypass User Account Control - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, detecting-privilege-escalation-attempts, detecting-t1548-abuse-elevation-control-mechanism"
        }
      ]
    },
    {
      "techniqueID": "T1548.003",
      "score": 12,
      "comment": "Sudo and Sudo Caching - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-t1548-abuse-elevation-control-mechanism, performing-privilege-escalation-assessment, performing-privilege-escalation-on-linux"
        }
      ]
    },
    {
      "techniqueID": "T1548.004",
      "score": 4,
      "comment": "Elevated Execution with Prompt - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-t1548-abuse-elevation-control-mechanism"
        }
      ]
    },
    {
      "techniqueID": "T1550",
      "score": 4,
      "comment": "Use Alternate Authentication Material - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-lateral-movement-detection"
        }
      ]
    },
    {
      "techniqueID": "T1550.002",
      "score": 35,
      "comment": "Pass the Hash - Referenced in 9 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "9"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, building-attack-pattern-library-from-cti-reports, conducting-full-scope-red-team-engagement, detecting-lateral-movement-in-network, detecting-pass-the-hash-attacks (+4 more)"
        }
      ]
    },
    {
      "techniqueID": "T1550.003",
      "score": 15,
      "comment": "Pass the Ticket - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "conducting-pass-the-ticket-attack, detecting-pass-the-hash-attacks, detecting-pass-the-ticket-attacks, exploiting-constrained-delegation-abuse"
        }
      ]
    },
    {
      "techniqueID": "T1550.004",
      "score": 4,
      "comment": "Web Session Cookie - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-initial-access-with-evilginx3"
        }
      ]
    },
    {
      "techniqueID": "T1552",
      "score": 4,
      "comment": "Unsecured Credentials - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-cloud-incident-containment-procedures"
        }
      ]
    },
    {
      "techniqueID": "T1552.001",
      "score": 4,
      "comment": "Credentials In Files - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-credential-access-with-lazagne"
        }
      ]
    },
    {
      "techniqueID": "T1552.002",
      "score": 4,
      "comment": "Credentials in Registry - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-credential-access-with-lazagne"
        }
      ]
    },
    {
      "techniqueID": "T1552.005",
      "score": 4,
      "comment": "Cloud Instance Metadata API - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-cloud-penetration-testing"
        }
      ]
    },
    {
      "techniqueID": "T1555",
      "score": 4,
      "comment": "Credentials from Password Stores - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-credential-access-with-lazagne"
        }
      ]
    },
    {
      "techniqueID": "T1555.003",
      "score": 4,
      "comment": "Web Browsers - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-credential-access-with-lazagne"
        }
      ]
    },
    {
      "techniqueID": "T1555.004",
      "score": 4,
      "comment": "Windows Credential Manager - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-credential-access-with-lazagne"
        }
      ]
    },
    {
      "techniqueID": "T1556",
      "score": 4,
      "comment": "Modify Authentication Process - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-initial-access-with-evilginx3"
        }
      ]
    },
    {
      "techniqueID": "T1557",
      "score": 4,
      "comment": "Adversary-in-the-Middle - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-initial-access-with-evilginx3"
        }
      ]
    },
    {
      "techniqueID": "T1557.001",
      "score": 8,
      "comment": "LLMNR/NBT-NS Poisoning - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "conducting-internal-network-penetration-test, hunting-for-ntlm-relay-attacks"
        }
      ]
    },
    {
      "techniqueID": "T1558",
      "score": 19,
      "comment": "Steal or Forge Kerberos Tickets - Referenced in 5 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "5"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, conducting-pass-the-ticket-attack, exploiting-kerberoasting-with-impacket, exploiting-nopac-cve-2021-42278-42287, performing-lateral-movement-detection"
        }
      ]
    },
    {
      "techniqueID": "T1558.001",
      "score": 27,
      "comment": "Golden Ticket - Referenced in 7 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "7"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, conducting-domain-persistence-with-dcsync, detecting-golden-ticket-forgery, detecting-kerberoasting-attacks, detecting-mimikatz-execution-patterns (+2 more)"
        }
      ]
    },
    {
      "techniqueID": "T1558.002",
      "score": 4,
      "comment": "Silver Ticket - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-active-directory-compromise-investigation"
        }
      ]
    },
    {
      "techniqueID": "T1558.003",
      "score": 54,
      "comment": "Kerberoasting - Referenced in 14 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "14"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, building-attack-pattern-library-from-cti-reports, conducting-full-scope-red-team-engagement, conducting-internal-network-penetration-test, detecting-kerberoasting-attacks (+9 more)"
        }
      ]
    },
    {
      "techniqueID": "T1558.004",
      "score": 4,
      "comment": "AS-REP Roasting - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-kerberoasting-attacks"
        }
      ]
    },
    {
      "techniqueID": "T1560",
      "score": 8,
      "comment": "Archive Collected Data - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, hunting-for-data-staging-before-exfiltration"
        }
      ]
    },
    {
      "techniqueID": "T1562",
      "score": 4,
      "comment": "Impair Defenses - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-evasion-techniques-in-endpoint-logs"
        }
      ]
    },
    {
      "techniqueID": "T1562.001",
      "score": 4,
      "comment": "Disable or Modify Tools - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-suspicious-powershell-execution"
        }
      ]
    },
    {
      "techniqueID": "T1566",
      "score": 23,
      "comment": "Phishing - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, analyzing-threat-actor-ttps-with-mitre-attack, analyzing-threat-landscape-with-misp, building-attack-pattern-library-from-cti-reports, implementing-mitre-attack-coverage-mapping (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1566.001",
      "score": 58,
      "comment": "Spearphishing Attachment - Referenced in 15 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "15"
        },
        {
          "name": "skills",
          "value": "analyzing-apt-group-with-mitre-navigator, analyzing-macro-malware-in-office-documents, analyzing-threat-actor-ttps-with-mitre-navigator, building-attack-pattern-library-from-cti-reports, conducting-full-scope-red-team-engagement (+10 more)"
        }
      ]
    },
    {
      "techniqueID": "T1566.002",
      "score": 23,
      "comment": "Spearphishing Link - Referenced in 6 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "6"
        },
        {
          "name": "skills",
          "value": "building-attack-pattern-library-from-cti-reports, conducting-spearphishing-simulation-campaign, hunting-for-spearphishing-indicators, implementing-continuous-security-validation-with-bas, implementing-mitre-attack-coverage-mapping (+1 more)"
        }
      ]
    },
    {
      "techniqueID": "T1566.003",
      "score": 12,
      "comment": "Spearphishing via Service - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "conducting-spearphishing-simulation-campaign, hunting-for-spearphishing-indicators, implementing-continuous-security-validation-with-bas"
        }
      ]
    },
    {
      "techniqueID": "T1566.004",
      "score": 4,
      "comment": "Spearphishing Voice - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-social-engineering-pretext-call"
        }
      ]
    },
    {
      "techniqueID": "T1567",
      "score": 15,
      "comment": "Exfiltration Over Web Service - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "detecting-insider-threat-behaviors, hunting-for-data-exfiltration-indicators, hunting-for-living-off-the-cloud-techniques, implementing-continuous-security-validation-with-bas"
        }
      ]
    },
    {
      "techniqueID": "T1567.002",
      "score": 4,
      "comment": "Exfiltration to Cloud Storage - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-data-exfiltration-indicators"
        }
      ]
    },
    {
      "techniqueID": "T1568",
      "score": 8,
      "comment": "Dynamic Resolution - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-command-and-control-beaconing, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1568.002",
      "score": 4,
      "comment": "Domain Generation Algorithms - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "hunting-for-beaconing-with-frequency-analysis"
        }
      ]
    },
    {
      "techniqueID": "T1569.002",
      "score": 12,
      "comment": "Service Execution - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-lateral-movement-in-network, detecting-lateral-movement-with-splunk, exploiting-ms17-010-eternalblue-vulnerability"
        }
      ]
    },
    {
      "techniqueID": "T1570",
      "score": 12,
      "comment": "Lateral Tool Transfer - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "detecting-lateral-movement-in-network, detecting-lateral-movement-with-splunk, performing-lateral-movement-with-wmiexec"
        }
      ]
    },
    {
      "techniqueID": "T1571",
      "score": 8,
      "comment": "Non-Standard Port - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "hunting-for-unusual-network-connections, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1572",
      "score": 15,
      "comment": "Protocol Tunneling - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "building-c2-infrastructure-with-sliver-framework, hunting-for-command-and-control-beaconing, hunting-for-dns-tunneling-with-zeek, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1573",
      "score": 15,
      "comment": "Encrypted Channel - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "analyzing-ransomware-network-indicators, hunting-for-beaconing-with-frequency-analysis, hunting-for-command-and-control-beaconing, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1573.002",
      "score": 8,
      "comment": "Asymmetric Cryptography - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "building-c2-infrastructure-with-sliver-framework, building-red-team-c2-infrastructure-with-havoc"
        }
      ]
    },
    {
      "techniqueID": "T1574",
      "score": 4,
      "comment": "Hijack Execution Flow - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "analyzing-persistence-mechanisms-in-linux"
        }
      ]
    },
    {
      "techniqueID": "T1574.001",
      "score": 8,
      "comment": "DLL Search Order Hijacking - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-dll-sideloading-attacks, hunting-for-persistence-mechanisms-in-windows"
        }
      ]
    },
    {
      "techniqueID": "T1574.002",
      "score": 15,
      "comment": "DLL Side-Loading - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "analyzing-windows-event-logs-in-splunk, building-attack-pattern-library-from-cti-reports, detecting-dll-sideloading-attacks, implementing-siem-use-cases-for-detection"
        }
      ]
    },
    {
      "techniqueID": "T1574.006",
      "score": 8,
      "comment": "Dynamic Linker Hijacking - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-dll-sideloading-attacks, performing-privilege-escalation-on-linux"
        }
      ]
    },
    {
      "techniqueID": "T1574.008",
      "score": 4,
      "comment": "Path Interception by Search Order Hijacking - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-dll-sideloading-attacks"
        }
      ]
    },
    {
      "techniqueID": "T1574.009",
      "score": 4,
      "comment": "Unquoted Service Path - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "detecting-privilege-escalation-attempts"
        }
      ]
    },
    {
      "techniqueID": "T1578",
      "score": 4,
      "comment": "Modify Cloud Compute Infrastructure - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-cloud-incident-containment-procedures"
        }
      ]
    },
    {
      "techniqueID": "T1580",
      "score": 4,
      "comment": "Cloud Infrastructure Discovery - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1583.001",
      "score": 15,
      "comment": "Domains - Referenced in 4 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "4"
        },
        {
          "name": "skills",
          "value": "building-red-team-c2-infrastructure-with-havoc, conducting-full-scope-red-team-engagement, conducting-spearphishing-simulation-campaign, implementing-mitre-attack-coverage-mapping"
        }
      ]
    },
    {
      "techniqueID": "T1583.003",
      "score": 4,
      "comment": "Virtual Private Server - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "building-red-team-c2-infrastructure-with-havoc"
        }
      ]
    },
    {
      "techniqueID": "T1585.002",
      "score": 4,
      "comment": "Email Accounts - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-spearphishing-simulation-campaign"
        }
      ]
    },
    {
      "techniqueID": "T1587.001",
      "score": 8,
      "comment": "Malware - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "building-red-team-c2-infrastructure-with-havoc, conducting-full-scope-red-team-engagement"
        }
      ]
    },
    {
      "techniqueID": "T1589",
      "score": 12,
      "comment": "Gather Victim Identity Information - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, conducting-social-engineering-pretext-call, performing-open-source-intelligence-gathering"
        }
      ]
    },
    {
      "techniqueID": "T1590",
      "score": 4,
      "comment": "Gather Victim Network Information - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-open-source-intelligence-gathering"
        }
      ]
    },
    {
      "techniqueID": "T1591",
      "score": 12,
      "comment": "Gather Victim Org Information - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "collecting-open-source-intelligence, conducting-social-engineering-pretext-call, performing-open-source-intelligence-gathering"
        }
      ]
    },
    {
      "techniqueID": "T1592",
      "score": 4,
      "comment": "Gather Victim Host Information - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-open-source-intelligence-gathering"
        }
      ]
    },
    {
      "techniqueID": "T1593",
      "score": 8,
      "comment": "Search Open Websites/Domains - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "conducting-full-scope-red-team-engagement, performing-open-source-intelligence-gathering"
        }
      ]
    },
    {
      "techniqueID": "T1594",
      "score": 4,
      "comment": "Search Victim-Owned Websites - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-open-source-intelligence-gathering"
        }
      ]
    },
    {
      "techniqueID": "T1595.001",
      "score": 4,
      "comment": "Scanning IP Blocks - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-open-source-intelligence-gathering"
        }
      ]
    },
    {
      "techniqueID": "T1595.002",
      "score": 4,
      "comment": "Vulnerability Scanning - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-open-source-intelligence-gathering"
        }
      ]
    },
    {
      "techniqueID": "T1596",
      "score": 4,
      "comment": "Search Open Technical Databases - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "performing-open-source-intelligence-gathering"
        }
      ]
    },
    {
      "techniqueID": "T1598",
      "score": 4,
      "comment": "Phishing for Information - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-social-engineering-pretext-call"
        }
      ]
    },
    {
      "techniqueID": "T1598.003",
      "score": 8,
      "comment": "Spearphishing Link/Voice - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "conducting-social-engineering-pretext-call, conducting-spearphishing-simulation-campaign"
        }
      ]
    },
    {
      "techniqueID": "T1608.001",
      "score": 4,
      "comment": "Upload Malware - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-spearphishing-simulation-campaign"
        }
      ]
    },
    {
      "techniqueID": "T1608.005",
      "score": 4,
      "comment": "Link Target - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "conducting-spearphishing-simulation-campaign"
        }
      ]
    },
    {
      "techniqueID": "T1610",
      "score": 8,
      "comment": "Deploy Container - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-container-escape-attempts, detecting-container-escape-with-falco-rules"
        }
      ]
    },
    {
      "techniqueID": "T1611",
      "score": 8,
      "comment": "Escape to Host - Referenced in 2 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "2"
        },
        {
          "name": "skills",
          "value": "detecting-container-escape-attempts, detecting-container-escape-with-falco-rules"
        }
      ]
    },
    {
      "techniqueID": "T1615",
      "score": 12,
      "comment": "Group Policy Discovery - Referenced in 3 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "3"
        },
        {
          "name": "skills",
          "value": "conducting-internal-reconnaissance-with-bloodhound-ce, exploiting-active-directory-with-bloodhound, performing-active-directory-bloodhound-analysis"
        }
      ]
    },
    {
      "techniqueID": "T1649",
      "score": 4,
      "comment": "Steal or Forge Authentication Certificates - Referenced in 1 skill(s)",
      "enabled": true,
      "metadata": [
        {
          "name": "skill_count",
          "value": "1"
        },
        {
          "name": "skills",
          "value": "exploiting-active-directory-certificate-services-esc1"
        }
      ]
    }
  ],
  "gradient": {
    "colors": [
      "#cfe2f3",
      "#6fa8dc",
      "#1155cc"
    ],
    "minValue": 1,
    "maxValue": 100
  },
  "legendItems": [
    {
      "label": "1-2 skills (Low coverage)",
      "color": "#cfe2f3"
    },
    {
      "label": "3-5 skills (Moderate coverage)",
      "color": "#6fa8dc"
    },
    {
      "label": "6-10 skills (Good coverage)",
      "color": "#3d85c6"
    },
    {
      "label": "11+ skills (Strong coverage)",
      "color": "#1155cc"
    }
  ],
  "showTacticRowBackground": true,
  "tacticRowBackground": "#205080",
  "selectTechniquesAcrossTactics": true,
  "selectSubtechniquesWithParent": true,
  "selectVisibleTechniques": false,
  "metadata": [
    {
      "name": "repository",
      "value": "Anthropic-Cybersecurity-Skills"
    },
    {
      "name": "total_techniques",
      "value": "218"
    },
    {
      "name": "total_skills_scanned",
      "value": "742"
    },
    {
      "name": "generated_date",
      "value": "2026-03-11"
    },
    {
      "name": "attack_version",
      "value": "14"
    },
    {
      "name": "description",
      "value": "Auto-generated from skill SKILL.md files referencing ATT&CK technique IDs"
    }
  ],
  "links": [
    {
      "label": "Repository",
      "url": "https://github.com/anthropics/cybersecurity-skills"
    },
    {
      "label": "ATT&CK Navigator",
      "url": "https://mitre-attack.github.io/attack-navigator/"
    }
  ]
}
===== END FILE =====


===== FILE: ./mappings/mitre-attack/README.md =====
# MITRE ATT&CK Mapping

This directory maps the cybersecurity skills in this repository to the [MITRE ATT&CK](https://attack.mitre.org/) framework (Enterprise v15).

## Overview

MITRE ATT&CK is a curated knowledge base and model for cyber adversary behavior, reflecting the various phases of an adversary's lifecycle and the platforms they target. This mapping connects our hands-on skills to ATT&CK tactics and techniques, enabling:

- **Threat-informed defense** -- prioritize skill development based on real adversary behavior
- **Gap analysis** -- identify ATT&CK techniques not yet covered by available skills
- **Purple team exercises** -- pair offensive (red team) and defensive (blue team) skills for each technique
- **Agent-driven discovery** -- AI agents can query skills by ATT&CK ID for automated security workflows

## Mapping Methodology

### Tactic Mapping (TA00xx)

Each of the 14 ATT&CK Enterprise tactics represents a distinct adversary objective. Skills are mapped to tactics based on which adversary goal they help achieve (offensive) or defend against (defensive):

| Tactic | ID | Offensive Skills | Defensive Skills |
|--------|-----|-----------------|------------------|
| Reconnaissance | TA0043 | penetration-testing, red-teaming | threat-intelligence, phishing-defense |
| Resource Development | TA0042 | red-teaming | threat-intelligence |
| Initial Access | TA0001 | web-application-security, penetration-testing | phishing-defense, endpoint-security |
| Execution | TA0002 | penetration-testing, red-teaming | malware-analysis, endpoint-security, soc-operations |
| Persistence | TA0003 | red-teaming, penetration-testing | threat-hunting, digital-forensics, endpoint-security |
| Privilege Escalation | TA0004 | penetration-testing, red-teaming | endpoint-security, identity-access-management |
| Defense Evasion | TA0005 | red-teaming | malware-analysis, endpoint-security, threat-hunting |
| Credential Access | TA0006 | penetration-testing, red-teaming | identity-access-management, soc-operations |
| Discovery | TA0007 | penetration-testing, red-teaming | threat-hunting, network-security |
| Lateral Movement | TA0008 | red-teaming, penetration-testing | network-security, threat-hunting, soc-operations |
| Collection | TA0009 | red-teaming | digital-forensics, threat-hunting |
| Command and Control | TA0011 | red-teaming | threat-intelligence, network-security, soc-operations |
| Exfiltration | TA0010 | red-teaming | threat-hunting, digital-forensics, network-security |
| Impact | TA0040 | red-teaming | ransomware-defense, incident-response |

### Technique Mapping (T1xxx)

Skills are mapped to specific techniques based on their content. Examples:

| Technique | ID | Example Skills |
|-----------|-----|---------------|
| Phishing | T1566 | analyzing-phishing-email-headers, analyzing-certificate-transparency-for-phishing |
| Exploit Public-Facing Application | T1190 | web-application-security skills (SQL injection, XSS, SSRF) |
| OS Credential Dumping | T1003 | penetration-testing credential harvesting skills |
| PowerShell | T1059.001 | analyzing-windows-event-logs-in-splunk, malware-analysis skills |
| Remote Services | T1021 | network-security lateral movement skills |
| Data Encrypted for Impact | T1486 | analyzing-ransomware-encryption-mechanisms |
| Command and Scripting Interpreter | T1059 | malware-analysis script deobfuscation skills |
| Scheduled Task/Job | T1053 | analyzing-malware-persistence-with-autoruns |
| Registry Run Keys | T1547.001 | analyzing-windows-registry-for-artifacts |
| DLL Side-Loading | T1574.002 | analyzing-bootkit-and-rootkit-samples |

### Sub-technique Mapping (T1xxx.xxx)

Where applicable, skills are mapped to sub-techniques for precision. For example:

- `T1566.001` (Spearphishing Attachment) -- analyzing-email-headers-for-phishing-investigation
- `T1566.002` (Spearphishing Link) -- analyzing-certificate-transparency-for-phishing
- `T1003.001` (LSASS Memory) -- analyzing-memory-dumps-with-volatility

## ATT&CK Navigator Integration

You can visualize our skill coverage using the [ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/). To generate a Navigator layer:

1. Use the coverage summary in [`coverage-summary.md`](coverage-summary.md) to identify covered tactics
2. Import the tactic/technique IDs into a Navigator layer JSON
3. Color-code by coverage depth (number of skills per technique)

### Suggested Color Scale

| Coverage | Color | Meaning |
|----------|-------|---------|
| 0 skills | White | No coverage -- gap |
| 1-2 skills | Light blue | Basic coverage |
| 3-5 skills | Medium blue | Moderate coverage |
| 6+ skills | Dark blue | Strong coverage |

## Skill Tag Convention

Skills relevant to ATT&CK carry these tags in their YAML frontmatter:

- `mitre-attack` -- general ATT&CK relevance (56 skills currently tagged)
- Technique-specific tags like `privilege-escalation`, `lateral-movement`, `persistence`
- Tool-specific tags that map to ATT&CK software entries (e.g., `cobalt-strike`, `mimikatz`)

## How to Contribute Mappings

1. **Identify the skill** -- Read the skill's SKILL.md to understand what it teaches
2. **Find the ATT&CK technique** -- Search [attack.mitre.org](https://attack.mitre.org/) for the matching technique
3. **Determine offensive vs. defensive** -- Is the skill about performing or detecting/preventing the technique?
4. **Update the mapping** -- Add the technique ID to the appropriate table in this directory
5. **Update skill tags** -- Add `mitre-attack` and technique-specific tags to the skill's frontmatter
6. **Submit a PR** -- Include the ATT&CK technique URL as justification

## References

- [MITRE ATT&CK Enterprise Matrix](https://attack.mitre.org/matrices/enterprise/)
- [MITRE ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/)
- [ATT&CK v15 Release Notes](https://attack.mitre.org/resources/updates/)
- [MITRE ATT&CK for ICS](https://attack.mitre.org/matrices/ics/) -- relevant for ot-ics-security skills
- [MITRE ATT&CK for Mobile](https://attack.mitre.org/matrices/mobile/) -- relevant for mobile-security skills

===== END FILE =====


===== FILE: ./mappings/mitre-attack/coverage-summary.md =====
# ATT&CK Coverage Summary

Coverage analysis of the 805 cybersecurity skills mapped to MITRE ATT&CK Enterprise tactics.

## Tactic Coverage Matrix

| ATT&CK Tactic | ID | Relevant Subdomains | Skills Count |
|---------------|-----|---------------------|--------------|
| Reconnaissance | TA0043 | threat-intelligence, penetration-testing, red-teaming | ~48 |
| Resource Development | TA0042 | threat-intelligence, red-teaming | ~30 |
| Initial Access | TA0001 | web-application-security, phishing-defense, api-security | ~45 |
| Execution | TA0002 | malware-analysis, endpoint-security, soc-operations | ~32 |
| Persistence | TA0003 | threat-hunting, digital-forensics, endpoint-security | ~28 |
| Privilege Escalation | TA0004 | penetration-testing, red-teaming, identity-access-management | ~40 |
| Defense Evasion | TA0005 | malware-analysis, endpoint-security, threat-hunting | ~25 |
| Credential Access | TA0006 | identity-access-management, penetration-testing | ~30 |
| Discovery | TA0007 | penetration-testing, threat-hunting, network-security | ~35 |
| Lateral Movement | TA0008 | red-teaming, network-security, soc-operations | ~28 |
| Collection | TA0009 | digital-forensics, threat-hunting | ~22 |
| Command and Control | TA0011 | threat-intelligence, network-security, soc-operations | ~30 |
| Exfiltration | TA0010 | threat-hunting, digital-forensics, network-security | ~20 |
| Impact | TA0040 | ransomware-defense, incident-response, ot-ics-security | ~35 |

## Subdomain-to-Tactic Heat Map

Shows which subdomains contribute skills to each ATT&CK tactic. Intensity indicates relevance (H = High, M = Medium, L = Low).

| Subdomain (skills) | Recon | Res Dev | Init Access | Exec | Persist | Priv Esc | Def Evasion | Cred Access | Disc | Lat Mov | Collect | C2 | Exfil | Impact |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| web-application-security (41) | L | - | **H** | M | L | M | L | M | L | - | - | - | - | M |
| threat-intelligence (43) | **H** | **H** | M | L | L | - | L | - | M | - | - | **H** | L | L |
| threat-hunting (35) | L | - | M | M | **H** | M | **H** | M | **H** | M | **H** | M | **H** | M |
| digital-forensics (34) | - | - | L | M | **H** | L | M | L | L | M | **H** | L | M | M |
| malware-analysis (34) | - | L | M | **H** | **H** | M | **H** | L | L | L | M | **H** | L | M |
| identity-access-management (33) | - | - | M | L | M | **H** | L | **H** | L | M | - | - | - | - |
| network-security (33) | M | - | M | L | L | L | L | L | M | **H** | L | **H** | **H** | L |
| soc-operations (33) | L | - | M | **H** | M | M | M | M | M | M | M | M | M | M |
| cloud-security (48) | M | M | **H** | M | M | **H** | M | **H** | **H** | M | M | L | M | M |
| api-security (28) | L | - | **H** | M | L | M | L | **H** | L | - | M | - | M | L |
| ot-ics-security (28) | M | L | M | M | M | L | L | M | **H** | M | **H** | M | L | **H** |
| container-security (26) | L | L | M | **H** | M | **H** | **H** | M | M | L | L | L | M | M |
| incident-response (24) | - | - | M | M | M | M | M | M | L | M | M | M | M | **H** |
| vulnerability-management (24) | M | - | **H** | M | L | M | L | L | **H** | L | - | - | - | M |
| penetration-testing (23) | **H** | M | **H** | **H** | M | **H** | M | **H** | **H** | M | M | M | M | L |
| red-teaming (24) | **H** | **H** | **H** | **H** | **H** | **H** | **H** | **H** | **H** | **H** | **H** | **H** | **H** | **H** |
| devsecops (16) | L | L | M | M | L | M | L | M | L | - | - | - | - | L |
| endpoint-security (16) | - | - | M | **H** | **H** | **H** | **H** | M | M | M | M | M | L | M |
| phishing-defense (16) | M | M | **H** | M | - | - | M | **H** | - | - | M | L | L | L |
| cryptography (13) | - | - | L | - | - | - | M | **H** | - | - | M | M | **H** | L |
| zero-trust-architecture (13) | - | - | M | L | L | **H** | L | **H** | L | **H** | L | L | M | - |
| mobile-security (12) | M | L | **H** | M | M | M | M | M | M | L | M | M | M | L |
| compliance-governance (5) | L | L | L | - | - | L | - | L | L | - | - | - | - | L |
| ransomware-defense (5) | - | - | M | M | M | L | M | - | - | - | M | M | L | **H** |

## Key Technique Coverage

High-confidence technique-to-skill mappings based on skill content analysis.

### Initial Access (TA0001) -- 45 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Phishing | T1566 | analyzing-phishing-email-headers, analyzing-certificate-transparency-for-phishing, 14 phishing-defense skills |
| Exploit Public-Facing Application | T1190 | 41 web-application-security skills, 28 api-security skills |
| External Remote Services | T1133 | network-security VPN/remote access skills |
| Valid Accounts | T1078 | identity-access-management credential skills |
| Supply Chain Compromise | T1195 | analyzing-supply-chain-malware-artifacts, devsecops dependency scanning |

### Execution (TA0002) -- 32 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Command and Scripting Interpreter | T1059 | malware-analysis script analysis skills |
| Exploitation for Client Execution | T1203 | web-application-security exploit skills |
| User Execution | T1204 | phishing-defense awareness skills |
| Container Administration Command | T1609 | container-security skills |

### Persistence (TA0003) -- 28 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Boot or Logon Autostart Execution | T1547 | analyzing-malware-persistence-with-autoruns, analyzing-windows-registry-for-artifacts |
| Scheduled Task/Job | T1053 | endpoint-security scheduled task skills |
| Create Account | T1136 | identity-access-management monitoring skills |
| Implant Internal Image | T1525 | container-security image scanning skills |

### Privilege Escalation (TA0004) -- 40 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Exploitation for Privilege Escalation | T1068 | penetration-testing privilege escalation skills |
| Access Token Manipulation | T1134 | identity-access-management token skills |
| Container Escape | T1611 | container-security escape detection skills |
| Domain Policy Modification | T1484 | identity-access-management AD skills |

### Defense Evasion (TA0005) -- 25 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Obfuscated Files or Information | T1027 | analyzing-packed-malware-with-upx-unpacker, malware deobfuscation skills |
| Masquerading | T1036 | threat-hunting detection skills |
| Rootkit | T1014 | analyzing-bootkit-and-rootkit-samples |
| Indicator Removal | T1070 | digital-forensics anti-forensics skills |

### Credential Access (TA0006) -- 30 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| OS Credential Dumping | T1003 | analyzing-memory-dumps-with-volatility, penetration-testing credential skills |
| Brute Force | T1110 | identity-access-management authentication skills |
| Steal Web Session Cookie | T1539 | web-application-security session skills |
| Unsecured Credentials | T1552 | cloud-security secrets management skills |

### Discovery (TA0007) -- 35 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Network Service Discovery | T1046 | network-security scanning skills, penetration-testing recon |
| System Information Discovery | T1082 | threat-hunting system enumeration skills |
| Cloud Infrastructure Discovery | T1580 | cloud-security asset discovery skills |
| Account Discovery | T1087 | identity-access-management enumeration skills |

### Lateral Movement (TA0008) -- 28 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Remote Services | T1021 | network-security remote access skills |
| Lateral Tool Transfer | T1570 | threat-hunting lateral movement detection skills |
| Use Alternate Authentication Material | T1550 | identity-access-management pass-the-hash skills |
| Exploitation of Remote Services | T1210 | penetration-testing exploitation skills |

### Collection (TA0009) -- 22 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Data from Local System | T1005 | digital-forensics disk/file analysis skills |
| Data from Network Shared Drive | T1039 | threat-hunting data access monitoring skills |
| Email Collection | T1114 | analyzing-outlook-pst-for-email-forensics |
| Screen Capture | T1113 | malware-analysis behavior analysis skills |

### Command and Control (TA0011) -- 30 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Application Layer Protocol | T1071 | analyzing-command-and-control-communication, network-security C2 detection |
| Encrypted Channel | T1573 | analyzing-network-covert-channels-in-malware |
| Ingress Tool Transfer | T1105 | analyzing-cobalt-strike-beacon-configuration |
| Proxy | T1090 | network-security proxy analysis skills |

### Exfiltration (TA0010) -- 20 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Exfiltration Over C2 Channel | T1041 | analyzing-dns-logs-for-exfiltration |
| Exfiltration Over Alternative Protocol | T1048 | network-security protocol analysis skills |
| Exfiltration Over Web Service | T1567 | cloud-security data loss prevention skills |

### Impact (TA0040) -- 35 skills

| Technique | ID | Primary Skills |
|-----------|----|---------------|
| Data Encrypted for Impact | T1486 | analyzing-ransomware-encryption-mechanisms, 5 ransomware-defense skills |
| Service Stop | T1489 | incident-response service restoration skills |
| Inhibit System Recovery | T1490 | ransomware-defense recovery skills |
| Manipulation of Control | T0831 | ot-ics-security control system skills |

## Coverage Gaps

Areas where additional skills would improve ATT&CK coverage:

| Gap Area | ATT&CK Techniques | Recommendation |
|----------|-------------------|----------------|
| Firmware attacks | T1542 (Pre-OS Boot) | Add UEFI/firmware analysis skills |
| Audio/video capture | T1123, T1125 | Add surveillance detection skills |
| Cloud-specific lateral movement | T1550.001 (Web Session Cookie in cloud) | Expand cloud-security lateral movement |
| Hardware additions | T1200 | Add physical security assessment skills |
| Traffic signaling | T1205 | Add network covert channel detection skills |

===== END FILE =====


===== FILE: ./mappings/nist-csf/README.md =====
# NIST Cybersecurity Framework 2.0 Mapping

This directory maps the cybersecurity skills in this repository to the [NIST Cybersecurity Framework (CSF) 2.0](https://www.nist.gov/cyberframework), published February 2024.

## Overview

NIST CSF 2.0 organizes cybersecurity activities into 6 core functions that represent the full lifecycle of managing cybersecurity risk. This mapping enables organizations to:

- **Align skill development** to their CSF implementation tier
- **Identify training gaps** across the CSF functions
- **Build role-based learning paths** using CSF categories
- **Automate compliance mapping** through AI agent queries

## CSF 2.0 Functions and Skill Alignment

### Govern (GV) -- Cybersecurity Risk Management Strategy

Establishing and monitoring the organization's cybersecurity risk management strategy, expectations, and policy.

| Category | ID | Mapped Subdomains | Skills |
|----------|-----|-------------------|--------|
| Organizational Context | GV.OC | compliance-governance | 5 |
| Risk Management Strategy | GV.RM | compliance-governance, vulnerability-management | 29 |
| Roles, Responsibilities, and Authorities | GV.RR | compliance-governance, identity-access-management | 38 |
| Policy | GV.PO | compliance-governance, zero-trust-architecture | 18 |
| Oversight | GV.OV | compliance-governance, soc-operations | 38 |
| Cybersecurity Supply Chain Risk Management | GV.SC | devsecops, container-security | 42 |

**Primary subdomains:** compliance-governance (5), identity-access-management (33), devsecops (16)

### Identify (ID) -- Understanding Organizational Cybersecurity Risk

Understanding the organization's current cybersecurity risks.

| Category | ID | Mapped Subdomains | Skills |
|----------|-----|-------------------|--------|
| Asset Management | ID.AM | cloud-security, container-security, network-security | 107 |
| Risk Assessment | ID.RA | vulnerability-management, threat-intelligence | 67 |
| Improvement | ID.IM | soc-operations, compliance-governance | 38 |

**Primary subdomains:** vulnerability-management (24), threat-intelligence (43), cloud-security (48)

### Protect (PR) -- Safeguarding Assets

Using safeguards to prevent or reduce cybersecurity risk.

| Category | ID | Mapped Subdomains | Skills |
|----------|-----|-------------------|--------|
| Identity Management, Authentication, and Access Control | PR.AA | identity-access-management, zero-trust-architecture | 46 |
| Awareness and Training | PR.AT | phishing-defense, compliance-governance | 21 |
| Data Security | PR.DS | cryptography, cloud-security, api-security | 89 |
| Platform Security | PR.PS | endpoint-security, container-security, devsecops | 58 |
| Technology Infrastructure Resilience | PR.IR | network-security, zero-trust-architecture | 46 |

**Primary subdomains:** zero-trust-architecture (13), devsecops (16), identity-access-management (33), cryptography (13)

### Detect (DE) -- Finding and Analyzing Cybersecurity Events

Finding and analyzing possible cybersecurity compromises and anomalies.

| Category | ID | Mapped Subdomains | Skills |
|----------|-----|-------------------|--------|
| Continuous Monitoring | DE.CM | soc-operations, threat-hunting, network-security | 101 |
| Adverse Event Analysis | DE.AE | threat-hunting, malware-analysis, soc-operations | 102 |

**Primary subdomains:** threat-hunting (35), soc-operations (33), malware-analysis (34)

### Respond (RS) -- Taking Action Regarding Detected Incidents

Managing and responding to detected cybersecurity incidents.

| Category | ID | Mapped Subdomains | Skills |
|----------|-----|-------------------|--------|
| Incident Management | RS.MA | incident-response, soc-operations | 57 |
| Incident Analysis | RS.AN | digital-forensics, malware-analysis, threat-intelligence | 111 |
| Incident Response Reporting and Communication | RS.CO | incident-response, compliance-governance | 29 |
| Incident Mitigation | RS.MI | incident-response, endpoint-security, network-security | 73 |

**Primary subdomains:** incident-response (24), digital-forensics (34), malware-analysis (34)

### Recover (RC) -- Restoring Capabilities After an Incident

Restoring assets and operations affected by a cybersecurity incident.

| Category | ID | Mapped Subdomains | Skills |
|----------|-----|-------------------|--------|
| Incident Recovery Plan Execution | RC.RP | incident-response, ransomware-defense | 29 |
| Incident Recovery Communication | RC.CO | incident-response, compliance-governance | 29 |

**Primary subdomains:** incident-response (24), ransomware-defense (5)

## Function Coverage Distribution

```
Govern   (GV): ████████████░░░░░░░░  ~54 skills (compliance, IAM, devsecops)
Identify (ID): ██████████████████░░  ~115 skills (vuln-mgmt, threat-intel, cloud)
Protect  (PR): ████████████████████  ~160 skills (IAM, ZTA, devsecops, crypto)
Detect   (DE): ████████████████░░░░  ~102 skills (threat-hunting, SOC, malware)
Respond  (RS): ██████████████████░░  ~111 skills (IR, forensics, malware)
Recover  (RC): ████░░░░░░░░░░░░░░░░  ~29 skills (IR recovery, ransomware)
```

## How to Use This Mapping

### For Organizations

1. Determine your target CSF implementation tier (Partial, Risk Informed, Repeatable, Adaptive)
2. Identify your CSF function priorities
3. Use the category tables above to find relevant skill subdomains
4. Deploy skills from those subdomains to your team's training plan

### For AI Agents

Query skills by CSF function using subdomain filters:

```
# Find all Detect (DE) function skills
Filter: subdomain IN (threat-hunting, soc-operations, malware-analysis)

# Find all Protect (PR) function skills
Filter: subdomain IN (identity-access-management, zero-trust-architecture, devsecops, cryptography)
```

### For Security Teams

Use the alignment table in [`csf-alignment.md`](csf-alignment.md) for a complete subdomain-to-category cross-reference.

## References

- [NIST CSF 2.0 (February 2024)](https://www.nist.gov/cyberframework)
- [NIST SP 800-53 Rev. 5 Control Mapping](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
- [CSF 2.0 Quick Start Guides](https://www.nist.gov/cyberframework/getting-started)
- [CSF 2.0 Reference Tool](https://csrc.nist.gov/Projects/Cybersecurity-Framework/Filters)

===== END FILE =====


===== FILE: ./mappings/nist-csf/csf-alignment.md =====
# NIST CSF 2.0 Alignment Table

Complete mapping of each skill subdomain to NIST CSF 2.0 functions and categories.

## Subdomain-to-CSF Alignment

| Subdomain | Skills | GV | ID | PR | PR | DE | RS | RC |
|-----------|--------|-----|-----|-----|-----|-----|-----|-----|
| | | Govern | Identify | Protect | Protect | Detect | Respond | Recover |

### Detailed Alignment

| Subdomain (Skills) | Primary CSF Function | CSF Categories | Alignment Rationale |
|---------------------|---------------------|----------------|---------------------|
| api-security (28) | Protect (PR) | PR.DS, PR.PS | API hardening, authentication, input validation |
| cloud-security (48) | Identify (ID), Protect (PR) | ID.AM, PR.DS, PR.PS, PR.IR | Cloud asset management, data protection, infrastructure resilience |
| compliance-governance (5) | Govern (GV) | GV.OC, GV.RM, GV.RR, GV.PO, GV.OV | Risk strategy, policy, organizational oversight |
| container-security (26) | Protect (PR) | PR.PS, GV.SC | Platform security, supply chain risk management |
| cryptography (13) | Protect (PR) | PR.DS | Data confidentiality and integrity at rest and in transit |
| devsecops (16) | Protect (PR), Govern (GV) | PR.PS, GV.SC | Secure development lifecycle, supply chain security |
| digital-forensics (34) | Respond (RS) | RS.AN, RS.MA | Incident analysis, evidence collection and examination |
| endpoint-security (16) | Protect (PR), Detect (DE) | PR.PS, DE.CM, DE.AE | Endpoint hardening, continuous monitoring, threat detection |
| identity-access-management (33) | Protect (PR), Govern (GV) | PR.AA, GV.RR | Identity lifecycle, authentication, authorization, access governance |
| incident-response (24) | Respond (RS), Recover (RC) | RS.MA, RS.AN, RS.MI, RS.CO, RC.RP, RC.CO | Full incident lifecycle from detection through recovery |
| malware-analysis (34) | Detect (DE), Respond (RS) | DE.AE, RS.AN | Adverse event analysis, reverse engineering, threat characterization |
| mobile-security (12) | Protect (PR) | PR.PS, PR.DS | Mobile platform security, application data protection |
| network-security (33) | Protect (PR), Detect (DE) | PR.IR, DE.CM | Network infrastructure resilience, traffic monitoring |
| ot-ics-security (28) | Protect (PR), Detect (DE) | PR.PS, PR.IR, DE.CM | Industrial control system protection and monitoring |
| penetration-testing (23) | Identify (ID) | ID.RA | Risk assessment through offensive security testing |
| phishing-defense (16) | Protect (PR), Detect (DE) | PR.AT, DE.CM, DE.AE | Security awareness training, phishing detection |
| ransomware-defense (5) | Respond (RS), Recover (RC) | RS.MI, RC.RP | Ransomware mitigation and recovery planning |
| red-teaming (24) | Identify (ID) | ID.RA, ID.IM | Adversary simulation for risk assessment and program improvement |
| soc-operations (33) | Detect (DE), Respond (RS) | DE.CM, DE.AE, RS.MA | Continuous monitoring, alert triage, incident management |
| threat-hunting (35) | Detect (DE) | DE.CM, DE.AE | Proactive threat detection, hypothesis-driven analysis |
| threat-intelligence (43) | Identify (ID), Detect (DE) | ID.RA, DE.AE | Threat landscape understanding, intelligence-driven detection |
| vulnerability-management (24) | Identify (ID) | ID.RA, GV.RM | Vulnerability identification, risk assessment, remediation prioritization |
| web-application-security (41) | Protect (PR), Identify (ID) | PR.DS, PR.PS, ID.RA | Application security testing and hardening |
| zero-trust-architecture (13) | Protect (PR) | PR.AA, PR.IR | Zero trust access control and network segmentation |

## CSF Category Coverage Summary

### Govern (GV)

| Category | ID | Description | Subdomain Coverage |
|----------|-----|------------|-------------------|
| Organizational Context | GV.OC | Understanding the organizational mission and stakeholder expectations | compliance-governance |
| Risk Management Strategy | GV.RM | Risk management priorities, constraints, and appetite | compliance-governance, vulnerability-management |
| Roles, Responsibilities, and Authorities | GV.RR | Cybersecurity roles and authorities are established | compliance-governance, identity-access-management |
| Policy | GV.PO | Organizational cybersecurity policy is established | compliance-governance, zero-trust-architecture |
| Oversight | GV.OV | Results of cybersecurity activities are reviewed | compliance-governance, soc-operations |
| Cybersecurity Supply Chain Risk Management | GV.SC | Supply chain risks are managed | devsecops, container-security |

### Identify (ID)

| Category | ID | Description | Subdomain Coverage |
|----------|-----|------------|-------------------|
| Asset Management | ID.AM | Assets that enable the organization are identified and managed | cloud-security, container-security, network-security |
| Risk Assessment | ID.RA | The cybersecurity risk to the organization is understood | vulnerability-management, threat-intelligence, penetration-testing, red-teaming |
| Improvement | ID.IM | Improvements to organizational cybersecurity are identified | soc-operations, red-teaming, compliance-governance |

### Protect (PR)

| Category | ID | Description | Subdomain Coverage |
|----------|-----|------------|-------------------|
| Identity Management, Authentication, and Access Control | PR.AA | Access is limited to authorized users, services, and hardware | identity-access-management, zero-trust-architecture |
| Awareness and Training | PR.AT | Personnel are provided cybersecurity awareness and training | phishing-defense, compliance-governance |
| Data Security | PR.DS | Data are managed consistent with the organization's risk strategy | cryptography, cloud-security, api-security |
| Platform Security | PR.PS | Hardware, software, and services are managed consistent with risk strategy | endpoint-security, container-security, devsecops, ot-ics-security |
| Technology Infrastructure Resilience | PR.IR | Security architectures are managed to protect asset confidentiality, integrity, and availability | network-security, zero-trust-architecture, ot-ics-security |

### Detect (DE)

| Category | ID | Description | Subdomain Coverage |
|----------|-----|------------|-------------------|
| Continuous Monitoring | DE.CM | Assets are monitored to find anomalies and indicators of compromise | soc-operations, threat-hunting, network-security, endpoint-security |
| Adverse Event Analysis | DE.AE | Anomalies and potential adverse events are analyzed | threat-hunting, malware-analysis, soc-operations, threat-intelligence |

### Respond (RS)

| Category | ID | Description | Subdomain Coverage |
|----------|-----|------------|-------------------|
| Incident Management | RS.MA | Responses to detected incidents are managed | incident-response, soc-operations |
| Incident Analysis | RS.AN | Investigations are conducted to understand the incident | digital-forensics, malware-analysis, threat-intelligence |
| Incident Response Reporting and Communication | RS.CO | Response activities are coordinated with internal and external stakeholders | incident-response, compliance-governance |
| Incident Mitigation | RS.MI | Activities are performed to prevent expansion and mitigate effects | incident-response, endpoint-security, network-security |

### Recover (RC)

| Category | ID | Description | Subdomain Coverage |
|----------|-----|------------|-------------------|
| Incident Recovery Plan Execution | RC.RP | Restoration activities are performed to ensure operational availability | incident-response, ransomware-defense |
| Incident Recovery Communication | RC.CO | Restoration activities are coordinated with internal and external parties | incident-response, compliance-governance |

## Gap Analysis

| CSF Category | Current Coverage | Gap |
|-------------|-----------------|-----|
| GV.OC | Low (5 skills) | Need more organizational security context and mission alignment skills |
| GV.PO | Low | Need dedicated policy development and management skills |
| PR.AT | Moderate (16 skills) | Could expand security awareness training beyond phishing |
| RC.RP | Low (29 skills) | Need more disaster recovery and business continuity skills |
| RC.CO | Low | Need dedicated incident communication and stakeholder management skills |

===== END FILE =====


===== FILE: ./mappings/owasp/README.md =====
# OWASP Top 10 (2025) Mapping

This directory maps the cybersecurity skills in this repository to the [OWASP Top 10](https://owasp.org/www-project-top-ten/) categories for web application security risks.

## Overview

The OWASP Top 10 represents the most critical security risks to web applications. This mapping connects hands-on skills to each risk category, enabling teams to build targeted training programs for secure development and application security testing.

## OWASP Top 10 2025 Skill Mapping

### A01:2025 -- Broken Access Control

Restrictions on what authenticated users are allowed to do are not properly enforced.

| Relevant Subdomains | Skills | Key Topics |
|---------------------|--------|------------|
| web-application-security | 41 | IDOR, privilege escalation, path traversal, CORS misconfiguration |
| identity-access-management | 33 | RBAC, ABAC, session management, OAuth/OIDC flaws |
| api-security | 28 | Broken object level authorization (BOLA), function level authorization |
| zero-trust-architecture | 13 | Least privilege enforcement, microsegmentation |

**Example skills:** Implementing RBAC, testing for IDOR vulnerabilities, configuring OAuth 2.0 securely, enforcing API authorization policies.

### A02:2025 -- Cryptographic Failures

Failures related to cryptography that lead to exposure of sensitive data.

| Relevant Subdomains | Skills | Key Topics |
|---------------------|--------|------------|
| cryptography | 13 | TLS configuration, key management, hashing, encryption at rest |
| web-application-security | 41 | HTTPS enforcement, cookie security flags, certificate validation |
| cloud-security | 48 | KMS configuration, secrets management, encryption in transit |
| api-security | 28 | API transport security, token encryption |

**Example skills:** Configuring TLS 1.3, implementing envelope encryption with KMS, securing JWT tokens, certificate pinning.

### A03:2025 -- Injection

User-supplied data is sent to an interpreter as part of a command or query without proper validation.

| Relevant Subdomains | Skills | Key Topics |
|---------------------|--------|------------|
| web-application-security | 41 | SQL injection, XSS, command injection, LDAP injection |
| api-security | 28 | GraphQL injection, NoSQL injection, header injection |
| devsecops | 16 | SAST/DAST scanning, input validation, parameterized queries |
| penetration-testing | 23 | Injection testing, payload crafting, WAF bypass |

**Example skills:** Exploiting and remediating SQL injection, testing for stored/reflected XSS, configuring parameterized queries, SAST pipeline integration.

### A04:2025 -- Insecure Design

Risks related to design and architectural flaws, calling for more use of threat modeling and secure design patterns.

| Relevant Subdomains | Skills | Key Topics |
|---------------------|--------|------------|
| devsecops | 16 | Threat modeling, secure SDLC, security requirements |
| zero-trust-architecture | 13 | Zero trust design principles, defense in depth |
| compliance-governance | 5 | Security architecture review, risk assessment frameworks |
| web-application-security | 41 | Business logic flaws, trust boundary definition |

**Example skills:** Conducting threat modeling with STRIDE, implementing secure design patterns, defining trust boundaries, security architecture review.

### A05:2025 -- Security Misconfiguration

Missing or incorrect security hardening across the application stack.

| Relevant Subdomains | Skills | Key Topics |
|---------------------|--------|------------|
| cloud-security | 48 | Cloud service misconfiguration, IAM policy errors, S3 bucket exposure |
| container-security | 26 | Container hardening, Kubernetes RBAC, pod security policies |
| network-security | 33 | Firewall rules, segmentation errors, default credentials |
| endpoint-security | 16 | OS hardening, unnecessary services, default configurations |

**Example skills:** Auditing AWS S3 bucket permissions, hardening Kubernetes clusters, configuring security headers, CIS benchmark compliance.

### A06:2025 -- Vulnerable and Outdated Components

Using components with known vulnerabilities or that are no longer maintained.

| Relevant Subdomains | Skills | Key Topics |
|---------------------|--------|------------|
| vulnerability-management | 24 | CVE tracking, vulnerability scanning, patch management |
| devsecops | 16 | SCA scanning, dependency management, SBOM generation |
| container-security | 26 | Image scanning, base image updates, registry security |
| web-application-security | 41 | Third-party library vulnerabilities, framework updates |

**Example skills:** Running Trivy container scans, implementing SCA in CI/CD, generating and analyzing SBOMs, CVE prioritization with CVSS/EPSS.

### A07:2025 -- Identification and Authentication Failures

Weaknesses in authentication and session management.

| Relevant Subdomains | Skills | Key Topics |
|---------------------|--------|------------|
| identity-access-management | 33 | MFA implementation, password policies, session fixation |
| web-application-security | 41 | Credential stuffing defense, brute force protection |
| api-security | 28 | API key management, OAuth token handling, JWT validation |
| phishing-defense | 16 | Credential phishing prevention, anti-phishing controls |

**Example skills:** Implementing FIDO2/WebAuthn, configuring adaptive MFA, securing API authentication, detecting credential stuffing attacks.

### A08:2025 -- Software and Data Integrity Failures

Failures related to code and infrastructure that do not protect against integrity violations.

| Relevant Subdomains | Skills | Key Topics |
|---------------------|--------|------------|
| devsecops | 16 | CI/CD pipeline security, code signing, artifact integrity |
| container-security | 26 | Image signing, admission control, supply chain verification |
| cryptography | 13 | Digital signatures, integrity hashing, code signing certificates |
| vulnerability-management | 24 | Supply chain risk, dependency integrity verification |

**Example skills:** Implementing Sigstore for container signing, securing CI/CD pipelines, verifying software supply chain integrity, content trust enforcement.

### A09:2025 -- Security Logging and Monitoring Failures

Insufficient logging, detection, monitoring, and active response.

| Relevant Subdomains | Skills | Key Topics |
|---------------------|--------|------------|
| soc-operations | 33 | SIEM configuration, log aggregation, alert tuning |
| threat-hunting | 35 | Log analysis, detection engineering, hypothesis-driven hunting |
| incident-response | 24 | Incident detection, log-based investigation, response automation |
| network-security | 33 | Network monitoring, flow analysis, IDS/IPS tuning |

**Example skills:** Analyzing security logs with Splunk, writing Sigma detection rules, configuring SIEM correlation rules, implementing centralized logging.

### A10:2025 -- Server-Side Request Forgery (SSRF)

Fetching a remote resource without validating the user-supplied URL.

| Relevant Subdomains | Skills | Key Topics |
|---------------------|--------|------------|
| web-application-security | 41 | SSRF exploitation, URL validation, allowlisting |
| cloud-security | 48 | IMDS exploitation, cloud metadata access, VPC endpoint security |
| api-security | 28 | API-to-API SSRF, webhook validation |
| penetration-testing | 23 | SSRF detection and exploitation techniques |

**Example skills:** Testing for SSRF vulnerabilities, securing cloud metadata endpoints (IMDSv2), implementing URL validation and allowlisting, detecting SSRF in API integrations.

## Cross-Reference: OWASP to ATT&CK

| OWASP Category | Related ATT&CK Techniques |
|---------------|--------------------------|
| A01: Broken Access Control | T1078 (Valid Accounts), T1548 (Abuse Elevation Control) |
| A02: Cryptographic Failures | T1557 (Adversary-in-the-Middle), T1040 (Network Sniffing) |
| A03: Injection | T1190 (Exploit Public-Facing App), T1059 (Command and Scripting) |
| A04: Insecure Design | T1195 (Supply Chain Compromise), cross-cutting |
| A05: Security Misconfiguration | T1574 (Hijack Execution Flow), T1190 |
| A06: Vulnerable Components | T1190 (Exploit Public-Facing App), T1195 |
| A07: Authentication Failures | T1110 (Brute Force), T1539 (Steal Web Session Cookie) |
| A08: Integrity Failures | T1195 (Supply Chain Compromise), T1554 (Compromise Client Software) |
| A09: Logging Failures | T1070 (Indicator Removal), T1562 (Impair Defenses) |
| A10: SSRF | T1190 (Exploit Public-Facing App) |

## Cross-Reference: OWASP to NIST CSF 2.0

| OWASP Category | NIST CSF Functions | CSF Categories |
|---------------|-------------------|----------------|
| A01: Broken Access Control | Protect | PR.AA |
| A02: Cryptographic Failures | Protect | PR.DS |
| A03: Injection | Protect, Detect | PR.DS, DE.AE |
| A04: Insecure Design | Govern, Protect | GV.RM, PR.PS |
| A05: Security Misconfiguration | Protect | PR.PS, PR.IR |
| A06: Vulnerable Components | Identify, Govern | ID.RA, GV.SC |
| A07: Authentication Failures | Protect | PR.AA |
| A08: Integrity Failures | Protect, Govern | PR.DS, GV.SC |
| A09: Logging Failures | Detect | DE.CM, DE.AE |
| A10: SSRF | Protect, Detect | PR.DS, DE.AE |

## References

- [OWASP Top 10 Project](https://owasp.org/www-project-top-ten/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/) -- relevant for api-security subdomain
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/) -- relevant for mobile-security subdomain
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/) -- Application Security Verification Standard

===== END FILE =====


===== FILE: ./tools/README.md =====
# Skill Validation Tools

## validate-skill.py

Validate SKILL.md metadata before submitting a PR.

### Usage

```bash
# Validate a single skill
python tools/validate-skill.py skills/my-new-skill/

# Validate all skills
python tools/validate-skill.py --all
```

### What it checks

- SKILL.md exists in the skill directory
- Valid YAML frontmatter (between `---` markers)
- Required fields present: `name`, `description`, `domain`, `subdomain`, `tags`, `version`, `author`, `license`
- Name is kebab-case, 1–64 characters
- Description is at least 50 characters (agentskills.io caps it at 1024; `tools/validate-agentskills.py` enforces that)
- Domain is `cybersecurity`
- Subdomain is from the allowed list
- Tags is a list with at least 2 items

### Requirements

Python 3.8+ and PyYAML (`pip install pyyaml`).

All frontmatter is parsed by `skill_frontmatter.py`, the single PyYAML-backed loader. It replaced three hand-rolled regex parsers that silently truncated multi-line descriptions to their first line — that bug shipped 604 of 817 descriptions broken in `index.json`. Do not reintroduce regex frontmatter parsing; CI fails the build if it detects any.

===== END FILE =====


===== FILE: ./tools/agentskills-skill.schema.json =====
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://agentskills.io/schema/skill-frontmatter.json",
  "title": "agentskills.io SKILL.md frontmatter",
  "description": "JSON Schema for the YAML frontmatter of a SKILL.md file per the agentskills.io open standard (Anthropic, 2025-12-18). Only `name` and `description` are required; `license`, `compatibility`, `metadata`, and `allowed-tools` are recognized optional fields. Additional top-level keys are PERMITTED as extra metadata (this repo uses domain/subdomain/tags/version/author and framework-mapping keys), so `additionalProperties` is true. Constraints JSON Schema cannot express (name must equal the parent directory; no angle-bracket content in frontmatter) are enforced by tools/validate-agentskills.py.",
  "type": "object",
  "additionalProperties": true,
  "required": ["name", "description"],
  "properties": {
    "name": {
      "type": "string",
      "minLength": 1,
      "maxLength": 64,
      "pattern": "^[a-z0-9]+(-[a-z0-9]+)*$",
      "description": "1-64 chars, lowercase alphanumeric and single hyphens only; no leading/trailing/consecutive hyphens; must match the parent directory name; must not be a reserved word."
    },
    "description": {
      "type": "string",
      "minLength": 1,
      "maxLength": 1024,
      "description": "1-1024 chars. Must state BOTH what the skill does and when to use it."
    },
    "license": {
      "type": "string",
      "minLength": 1,
      "description": "SPDX license identifier or license text."
    },
    "compatibility": {
      "type": "string",
      "maxLength": 500,
      "description": "Free-text compatibility notes (<=500 chars)."
    },
    "metadata": {
      "type": "object",
      "description": "Arbitrary key/value map for non-standard fields. This is where domain, subdomain, tags, version, author, and framework mappings (mitre_attack, nist_csf, atlas_techniques, d3fend_techniques, nist_ai_rmf, mitre_f3) belong under strict compliance.",
      "additionalProperties": true
    },
    "allowed-tools": {
      "description": "EXPERIMENTAL. Tools the skill is permitted to use; string or array of strings.",
      "oneOf": [
        { "type": "string" },
        { "type": "array", "items": { "type": "string" } }
      ]
    }
  }
}

===== END FILE =====


===== FILE: ./tools/collision-allowlist.json =====
{
  "_comment": [
    "Skill pairs that score as near-duplicates but are legitimately distinct.",
    "",
    "Add a pair here ONLY after reading both descriptions and confirming a real",
    "difference in scope, platform, or offense-vs-defense posture. The 'reason'",
    "is not bookkeeping: it is the wording each skill's negative trigger should",
    "use, e.g. reason 'Linux vs Windows' becomes 'Do not use for Windows hosts",
    "- use hardening-windows-endpoint-with-cis-benchmark.'",
    "",
    "A pair that is genuinely ambiguous does NOT belong here. Disambiguate the",
    "two descriptions instead.",
    "",
    "Regenerate the candidate list with: python tools/detect-collisions.py"
  ],
  "reviewed_distinct": [
    {
      "pair": [
        "hardening-linux-endpoint-with-cis-benchmark",
        "hardening-windows-endpoint-with-cis-benchmark"
      ],
      "reason": "Same CIS Benchmark methodology, different operating systems. The controls, tooling and audit commands do not overlap, so both must exist; each description must name the OS explicitly and point at the other."
    }
  ]
}

===== END FILE =====


===== FILE: ./tools/detect-collisions.py =====
#!/usr/bin/env python3
"""Find skill pairs whose descriptions compete for the same user request.

An agent picks a skill from its description alone. When two descriptions are
near-duplicates, the router cannot reliably choose between them and misroutes
-- "skill collision". This scores every pair by TF-IDF cosine similarity over
description + slug and reports the ones close enough to collide.

A high score is NOT automatically a defect. Some near-twins are legitimately
distinct (Linux vs Windows CIS hardening; red-team DCSync vs blue-team DCSync
detection). Those belong in tools/collision-allowlist.json with a reason, which
is also a record of WHY they differ -- exactly the wording their negative
triggers need.

Pure stdlib; no numpy/sklearn required.

Usage:
    python tools/detect-collisions.py
    python tools/detect-collisions.py --threshold 0.5
    python tools/detect-collisions.py --json
    python tools/detect-collisions.py --max-unreviewed 60   # CI gate
"""
from __future__ import annotations

import argparse
import itertools
import json
import math
import os
import re
import sys
from collections import Counter

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))

from skill_frontmatter import description_of, iter_skill_dirs, load_frontmatter, FrontmatterError

ALLOWLIST_PATH = os.path.join(os.path.dirname(os.path.abspath(__file__)),
                              "collision-allowlist.json")
DEFAULT_THRESHOLD = 0.45

# Words too common in this corpus to carry signal.
STOPWORDS = set("""
the a an and or of to for in on with using use uses used when this that is are be by
from as it its into via at not skill security detect detects detecting analyze analyzes
analyzing perform performs performing implement implements implementing
""".split())

# A term appearing in more than this many skills is treated as a domain-wide
# background word and skipped when pairing (keeps the comparison O(usable pairs)).
MAX_DOCS_PER_TERM = 60


# A negative trigger names the sibling skill on purpose ("Do not use for X -
# use other-skill."). Scoring that text would inject the sibling's own slug
# tokens into this skill's vector, so correctly disambiguating a pair would
# RAISE its similarity score -- the metric would punish the fix. Strip the
# disambiguation scaffolding and score only the descriptive part.
DISAMBIGUATION_RE = re.compile(r"\b(?:do\s+not\s+use|don'?t\s+use|avoid\s+(?:this\s+)?(?:skill\s+)?for)\b.*",
                               re.IGNORECASE | re.DOTALL)
KEYWORDS_LABEL_RE = re.compile(r"\bkeywords\s*:", re.IGNORECASE)


def strip_disambiguation(text: str) -> str:
    """Drop negative-trigger sentences and the literal 'Keywords:' label.

    The keyword TERMS stay -- they are real content. Only the label is removed,
    since it would otherwise be a shared token across every fixed skill.
    """
    text = DISAMBIGUATION_RE.sub("", text)
    return KEYWORDS_LABEL_RE.sub(" ", text)


def tokenize(text: str) -> Counter:
    return Counter(w for w in re.findall(r"[a-z0-9]+", text.lower())
                   if w not in STOPWORDS and len(w) > 2)


def build_vectors(skills_dir: str) -> dict[str, dict[str, float]]:
    """L2-normalized TF-IDF vectors keyed by slug."""
    docs: dict[str, Counter] = {}
    for slug, skill_dir in iter_skill_dirs(skills_dir):
        try:
            frontmatter = load_frontmatter(os.path.join(skill_dir, "SKILL.md"))
        except FrontmatterError:
            continue
        described = strip_disambiguation(description_of(frontmatter))
        docs[slug] = tokenize(f"{described} {slug.replace('-', ' ')}")

    doc_freq: Counter = Counter()
    for counts in docs.values():
        doc_freq.update(counts.keys())

    total = len(docs)
    vectors: dict[str, dict[str, float]] = {}
    for slug, counts in docs.items():
        weights = {term: (1 + math.log(freq)) * math.log(total / doc_freq[term])
                   for term, freq in counts.items()}
        norm = math.sqrt(sum(w * w for w in weights.values())) or 1.0
        vectors[slug] = {term: w / norm for term, w in weights.items()}
    return vectors


def score_pairs(vectors: dict[str, dict[str, float]], threshold: float):
    """Cosine similarity for every pair sharing at least one discriminating term."""
    postings: dict[str, list[str]] = {}
    for slug, vector in vectors.items():
        for term in vector:
            postings.setdefault(term, []).append(slug)

    sims: dict[tuple[str, str], float] = {}
    for term, slugs in postings.items():
        if len(slugs) > MAX_DOCS_PER_TERM:
            continue
        weight_by_slug = {s: vectors[s][term] for s in slugs}
        for a, b in itertools.combinations(sorted(slugs), 2):
            sims[(a, b)] = sims.get((a, b), 0.0) + weight_by_slug[a] * weight_by_slug[b]

    return sorted(((s, p) for p, s in sims.items() if s >= threshold), reverse=True)


def load_allowlist() -> dict[tuple[str, str], str]:
    if not os.path.isfile(ALLOWLIST_PATH):
        return {}
    with open(ALLOWLIST_PATH, encoding="utf-8") as handle:
        data = json.load(handle)
    out: dict[tuple[str, str], str] = {}
    for entry in data.get("reviewed_distinct", []):
        pair = tuple(sorted(entry["pair"]))
        out[pair] = entry.get("reason", "")
    return out


def main() -> int:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("--threshold", type=float, default=DEFAULT_THRESHOLD)
    parser.add_argument("--json", action="store_true", dest="as_json")
    parser.add_argument("--max-unreviewed", type=int, default=None,
                        help="exit 1 if unreviewed colliding pairs exceed this")
    parser.add_argument("--skills-dir", default="skills")
    args = parser.parse_args()

    if not os.path.isdir(args.skills_dir):
        print(f"ERROR: '{args.skills_dir}' not found. Run from the repository root.")
        return 1

    vectors = build_vectors(args.skills_dir)
    pairs = score_pairs(vectors, args.threshold)
    allowlist = load_allowlist()

    unreviewed = [(s, p) for s, p in pairs if p not in allowlist]
    reviewed = [(s, p) for s, p in pairs if p in allowlist]
    involved = {slug for _, pair in unreviewed for slug in pair}

    if args.as_json:
        print(json.dumps({
            "threshold": args.threshold,
            "total_pairs": len(pairs),
            "unreviewed": [{"score": round(s, 3), "pair": list(p)} for s, p in unreviewed],
            "reviewed_distinct": [{"score": round(s, 3), "pair": list(p),
                                   "reason": allowlist[p]} for s, p in reviewed],
            "skills_involved": sorted(involved),
        }, indent=2))
    else:
        print(f"Colliding pairs at cosine >= {args.threshold}: {len(pairs)} "
              f"({len(reviewed)} reviewed-distinct, {len(unreviewed)} unreviewed)")
        print(f"Skills involved in an unreviewed collision: {len(involved)} "
              f"of {len(vectors)}\n")
        for score, (a, b) in unreviewed:
            print(f"  {score:.2f}  {a}\n        {b}")
        if reviewed:
            print(f"\nReviewed as legitimately distinct ({len(reviewed)}):")
            for score, (a, b) in reviewed:
                print(f"  {score:.2f}  {a} || {b}\n        reason: {allowlist[(a, b)]}")

    if args.max_unreviewed is not None and len(unreviewed) > args.max_unreviewed:
        print(f"\nERROR: {len(unreviewed)} unreviewed colliding pairs "
              f"exceeds --max-unreviewed {args.max_unreviewed}.")
        print("Either disambiguate the descriptions or record the pair in "
              f"{os.path.relpath(ALLOWLIST_PATH)} with a reason.")
        return 1
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: ./tools/generate-index.py =====
#!/usr/bin/env python3
"""Generate index.json from the SKILL.md files under skills/.

Previously this logic lived as an inline heredoc inside
.github/workflows/update-index.yml with a hand-rolled regex YAML parser that
truncated 604 of 817 descriptions. It lives here now so it is testable outside
CI and shares one PyYAML-backed parser with every other tool.

Usage:
    python tools/generate-index.py            # write index.json
    python tools/generate-index.py --check    # verify index.json is current (CI)
"""
from __future__ import annotations

import argparse
import json
import os
import sys
from datetime import datetime, timezone

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))

from skill_frontmatter import description_of, iter_skill_dirs, load_frontmatter, FrontmatterError

INDEX_VERSION = "1.1.0"
REPOSITORY = "https://github.com/mukul975/Anthropic-Cybersecurity-Skills"
DEFAULT_DOMAIN = "cybersecurity"


def build_index(skills_dir: str = "skills") -> tuple[dict, list[str]]:
    """Build the index payload. Returns (index, errors)."""
    skills = []
    errors = []

    for slug, skill_dir in iter_skill_dirs(skills_dir):
        try:
            frontmatter = load_frontmatter(os.path.join(skill_dir, "SKILL.md"))
        except FrontmatterError as exc:
            errors.append(f"{slug}: {exc}")
            continue

        description = description_of(frontmatter)
        if not description:
            errors.append(f"{slug}: empty description")

        skills.append({
            "name": slug,
            "description": description,
            "domain": frontmatter.get("domain") or DEFAULT_DOMAIN,
            "path": f"{skills_dir}/{slug}",
        })

    index = {
        "version": INDEX_VERSION,
        "generated_at": datetime.now(timezone.utc).strftime("%Y-%m-%dT%H:%M:%SZ"),
        "repository": REPOSITORY,
        "domain": DEFAULT_DOMAIN,
        "total_skills": len(skills),
        "skills": skills,
    }
    return index, errors


def _comparable(index: dict) -> str:
    """Serialize an index ignoring generated_at, so --check tolerates a re-run."""
    return json.dumps({k: v for k, v in index.items() if k != "generated_at"}, sort_keys=True)


def main() -> int:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("--check", action="store_true",
                        help="verify index.json matches the skills tree; do not write")
    parser.add_argument("--skills-dir", default="skills")
    parser.add_argument("--out", default="index.json")
    args = parser.parse_args()

    if not os.path.isdir(args.skills_dir):
        print(f"ERROR: '{args.skills_dir}' not found. Run from the repository root.")
        return 1

    index, errors = build_index(args.skills_dir)

    for error in errors:
        print(f"ERROR {error}")
    if errors:
        print(f"\n{len(errors)} skill(s) could not be indexed.")
        return 1

    if args.check:
        if not os.path.isfile(args.out):
            print(f"ERROR: {args.out} is missing. Run: python tools/generate-index.py")
            return 1
        with open(args.out, encoding="utf-8") as handle:
            current = json.load(handle)
        if _comparable(current) != _comparable(index):
            print(f"ERROR: {args.out} is out of date. Run: python tools/generate-index.py")
            return 1
        print(f"OK: {args.out} is up to date ({index['total_skills']} skills)")
        return 0

    with open(args.out, "w", encoding="utf-8") as handle:
        json.dump(index, handle, separators=(",", ":"))

    print(f"Updated {args.out}: {index['total_skills']} skills")
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: ./tools/lint-baseline.json =====
{
  "_comment": "Pre-existing lint failures, grandfathered so CI can gate new work today. This file may only shrink. Never add a slug by hand -- fix the skill, then run: python tools/lint-descriptions.py --update-baseline",
  "_total_grandfathered": 980,
  "desc-ends-punctuation": [
    "analyzing-active-directory-acl-abuse",
    "analyzing-malicious-url-with-urlscan",
    "detecting-aws-iam-privilege-escalation",
    "exploiting-active-directory-with-bloodhound",
    "implementing-email-sandboxing-with-proofpoint",
    "implementing-patch-management-workflow",
    "implementing-secrets-scanning-in-ci-cd",
    "performing-cryptographic-audit-of-application",
    "performing-dark-web-monitoring-for-threats",
    "performing-malware-ioc-extraction",
    "performing-open-source-intelligence-gathering",
    "performing-service-account-audit",
    "prioritizing-vulnerabilities-with-cvss-scoring"
  ],
  "desc-has-use-when": [
    "analyzing-active-directory-acl-abuse",
    "analyzing-android-malware-with-apktool",
    "analyzing-apt-group-with-mitre-navigator",
    "analyzing-certificate-transparency-for-phishing",
    "analyzing-cobalt-strike-beacon-configuration",
    "analyzing-docker-container-forensics",
    "analyzing-ethereum-smart-contract-vulnerabilities",
    "analyzing-heap-spray-exploitation",
    "analyzing-ios-app-security-with-objection",
    "analyzing-linux-audit-logs-for-intrusion",
    "analyzing-linux-kernel-rootkits",
    "analyzing-macro-malware-in-office-documents",
    "analyzing-malicious-url-with-urlscan",
    "analyzing-memory-dumps-with-volatility",
    "analyzing-network-flow-data-with-netflow",
    "analyzing-network-traffic-for-incidents",
    "analyzing-network-traffic-of-malware",
    "analyzing-network-traffic-with-wireshark",
    "analyzing-office365-audit-logs-for-compromise",
    "analyzing-pdf-malware-with-pdfid",
    "analyzing-powershell-script-block-logging",
    "analyzing-ransomware-encryption-mechanisms",
    "analyzing-sbom-for-supply-chain-vulnerabilities",
    "analyzing-security-logs-with-splunk",
    "analyzing-supply-chain-malware-artifacts",
    "analyzing-web-server-logs-for-intrusion",
    "auditing-aws-s3-bucket-permissions",
    "auditing-azure-active-directory-configuration",
    "auditing-gcp-iam-permissions",
    "auditing-kubernetes-cluster-rbac",
    "auditing-terraform-infrastructure-for-security",
    "auditing-tls-certificate-transparency-logs",
    "auditing-uefi-firmware-with-chipsec",
    "building-detection-rule-with-splunk-spl",
    "bypassing-authentication-with-forced-browsing",
    "coercing-authentication-with-coercer-petitpotam",
    "collecting-indicators-of-compromise",
    "conducting-api-security-testing",
    "conducting-cloud-penetration-testing",
    "conducting-domain-persistence-with-dcsync",
    "conducting-internal-network-penetration-test",
    "conducting-man-in-the-middle-attack-simulation",
    "conducting-memory-forensics-with-volatility",
    "conducting-mobile-app-penetration-test",
    "conducting-network-penetration-test",
    "conducting-post-incident-lessons-learned",
    "conducting-social-engineering-pretext-call",
    "conducting-spearphishing-simulation-campaign",
    "conducting-wireless-network-penetration-test",
    "containing-active-breach",
    "deploying-decoy-files-for-ransomware-detection",
    "detecting-anomalous-authentication-patterns",
    "detecting-attacks-on-scada-systems",
    "detecting-aws-cloudtrail-anomalies",
    "detecting-aws-iam-privilege-escalation",
    "detecting-azure-lateral-movement",
    "detecting-business-email-compromise-with-ai",
    "detecting-cryptomining-in-cloud",
    "detecting-lateral-movement-in-network",
    "detecting-lateral-movement-with-zeek",
    "detecting-living-off-the-land-attacks",
    "detecting-malicious-scheduled-tasks-with-sysmon",
    "detecting-network-scanning-with-ids-signatures",
    "detecting-port-scanning-with-fail2ban",
    "detecting-process-injection-techniques",
    "detecting-ransomware-precursors-in-network",
    "detecting-s3-data-exfiltration-attempts",
    "detecting-serverless-function-injection",
    "detecting-sql-injection-via-waf-logs",
    "detecting-suspicious-oauth-application-consent",
    "detecting-typosquatting-packages-in-npm-pypi",
    "emulating-cloud-attacks-with-stratus-red-team",
    "enumerating-cloud-with-cloudfox",
    "executing-active-directory-attack-simulation",
    "executing-phishing-simulation-campaign",
    "exploiting-active-directory-certificate-services-esc1",
    "exploiting-active-directory-with-bloodhound",
    "exploiting-bgp-hijacking-vulnerabilities",
    "exploiting-insecure-deserialization",
    "exploiting-ipv6-vulnerabilities",
    "exploiting-oauth-misconfiguration",
    "exploiting-server-side-request-forgery",
    "exploiting-smb-vulnerabilities-with-metasploit",
    "exploiting-sql-injection-vulnerabilities",
    "exploiting-sql-injection-with-sqlmap",
    "exploiting-websocket-vulnerabilities",
    "extracting-windows-event-logs-artifacts",
    "hunting-for-anomalous-powershell-execution",
    "hunting-for-beaconing-with-frequency-analysis",
    "hunting-for-command-and-control-beaconing",
    "hunting-for-defense-evasion-via-timestomping",
    "hunting-for-registry-run-key-persistence",
    "hunting-for-spearphishing-indicators",
    "implementing-aqua-security-for-container-scanning",
    "implementing-cloud-trail-log-analysis",
    "implementing-ddos-mitigation-with-cloudflare",
    "implementing-diamond-model-analysis",
    "implementing-email-sandboxing-with-proofpoint",
    "implementing-honeypot-for-ransomware-detection",
    "implementing-mitre-attack-coverage-mapping",
    "implementing-network-deception-with-honeypots",
    "implementing-patch-management-workflow",
    "implementing-proofpoint-email-security-gateway",
    "implementing-runtime-application-self-protection",
    "implementing-secret-scanning-with-gitleaks",
    "implementing-secrets-scanning-in-ci-cd",
    "implementing-security-monitoring-with-datadog",
    "implementing-semgrep-for-custom-sast-rules",
    "implementing-velociraptor-for-ir-collection",
    "investigating-ransomware-attack-artifacts",
    "monitoring-scada-modbus-traffic-anomalies",
    "performing-active-directory-compromise-investigation",
    "performing-active-directory-vulnerability-assessment",
    "performing-agentless-vulnerability-scanning",
    "performing-api-inventory-and-discovery",
    "performing-aws-privilege-escalation-assessment",
    "performing-brand-monitoring-for-impersonation",
    "performing-clickjacking-attack-test",
    "performing-credential-access-with-lazagne",
    "performing-cryptographic-audit-of-application",
    "performing-csrf-attack-simulation",
    "performing-dark-web-monitoring-for-threats",
    "performing-dns-enumeration-and-zone-transfer",
    "performing-external-network-penetration-test",
    "performing-firmware-extraction-with-binwalk",
    "performing-gcp-security-assessment-with-forseti",
    "performing-graphql-depth-limit-attack",
    "performing-graphql-security-assessment",
    "performing-initial-access-with-evilginx3",
    "performing-iot-security-assessment",
    "performing-ip-reputation-analysis-with-shodan",
    "performing-lateral-movement-with-wmiexec",
    "performing-log-analysis-for-forensic-investigation",
    "performing-malware-ioc-extraction",
    "performing-malware-persistence-investigation",
    "performing-memory-forensics-with-volatility3-plugins",
    "performing-mobile-app-certificate-pinning-bypass",
    "performing-oauth-scope-minimization-review",
    "performing-open-source-intelligence-gathering",
    "performing-ot-network-security-assessment",
    "performing-ot-vulnerability-scanning-safely",
    "performing-packet-injection-attack",
    "performing-paste-site-monitoring-for-credentials",
    "performing-physical-intrusion-assessment",
    "performing-plc-firmware-security-analysis",
    "performing-privilege-escalation-assessment",
    "performing-s7comm-protocol-security-analysis",
    "performing-sca-dependency-scanning-with-snyk",
    "performing-scada-hmi-security-assessment",
    "performing-second-order-sql-injection",
    "performing-security-headers-audit",
    "performing-serverless-function-security-review",
    "performing-service-account-audit",
    "performing-subdomain-enumeration-with-subfinder",
    "performing-thick-client-application-penetration-test",
    "performing-threat-hunting-with-yara-rules",
    "performing-vulnerability-scanning-with-nessus",
    "performing-web-application-penetration-test",
    "performing-web-cache-poisoning-attack",
    "performing-wifi-password-cracking-with-aircrack",
    "performing-wireless-network-penetration-test",
    "performing-wireless-security-assessment-with-kismet",
    "prioritizing-vulnerabilities-with-cvss-scoring",
    "reverse-engineering-ransomware-encryption-routine",
    "scanning-infrastructure-with-nessus",
    "testing-api-authentication-weaknesses",
    "testing-cors-misconfiguration",
    "testing-for-host-header-injection",
    "testing-for-sensitive-data-exposure",
    "testing-for-xml-injection-vulnerabilities",
    "testing-for-xss-vulnerabilities-with-burpsuite",
    "testing-for-xxe-injection-vulnerabilities",
    "testing-jwt-token-security",
    "validating-tpm-measured-boot-attestation",
    "verifying-build-provenance-with-slsa-sigstore"
  ],
  "desc-has-negative-trigger": [
    "abusing-dpapi-for-credential-access",
    "abusing-shadow-credentials-for-privesc",
    "achieving-cmmc-level-2-compliance",
    "acquiring-disk-image-with-dd-and-dcfldd",
    "analyzing-active-directory-acl-abuse",
    "analyzing-android-malware-with-apktool",
    "analyzing-api-gateway-access-logs",
    "analyzing-apt-group-with-mitre-navigator",
    "analyzing-azure-activity-logs-for-threats",
    "analyzing-bootkit-and-rootkit-samples",
    "analyzing-browser-forensics-with-hindsight",
    "analyzing-campaign-attribution-evidence",
    "analyzing-certificate-transparency-for-phishing",
    "analyzing-cloud-storage-access-patterns",
    "analyzing-cobalt-strike-beacon-configuration",
    "analyzing-cobaltstrike-malleable-c2-profiles",
    "analyzing-command-and-control-communication",
    "analyzing-cyber-kill-chain",
    "analyzing-disk-image-with-autopsy",
    "analyzing-dns-logs-for-exfiltration",
    "analyzing-docker-container-forensics",
    "analyzing-email-headers-for-phishing-investigation",
    "analyzing-ethereum-smart-contract-vulnerabilities",
    "analyzing-golang-malware-with-ghidra",
    "analyzing-heap-spray-exploitation",
    "analyzing-indicators-of-compromise",
    "analyzing-ios-app-security-with-objection",
    "analyzing-linux-audit-logs-for-intrusion",
    "analyzing-linux-elf-malware",
    "analyzing-linux-kernel-rootkits",
    "analyzing-linux-system-artifacts",
    "analyzing-lnk-file-and-jump-list-artifacts",
    "analyzing-macro-malware-in-office-documents",
    "analyzing-malicious-pdf-with-peepdf",
    "analyzing-malicious-url-with-urlscan",
    "analyzing-malware-behavior-with-cuckoo-sandbox",
    "analyzing-malware-family-relationships-with-malpedia",
    "analyzing-malware-persistence-with-autoruns",
    "analyzing-malware-sandbox-evasion-techniques",
    "analyzing-memory-dumps-with-volatility",
    "analyzing-memory-forensics-with-lime-and-volatility",
    "analyzing-mft-for-deleted-file-recovery",
    "analyzing-network-covert-channels-in-malware",
    "analyzing-network-flow-data-with-netflow",
    "analyzing-network-packets-with-scapy",
    "analyzing-network-traffic-for-incidents",
    "analyzing-network-traffic-of-malware",
    "analyzing-network-traffic-with-wireshark",
    "analyzing-office365-audit-logs-for-compromise",
    "analyzing-outlook-pst-for-email-forensics",
    "analyzing-packed-malware-with-upx-unpacker",
    "analyzing-pdf-malware-with-pdfid",
    "analyzing-persistence-mechanisms-in-linux",
    "analyzing-powershell-empire-artifacts",
    "analyzing-powershell-script-block-logging",
    "analyzing-prefetch-files-for-execution-history",
    "analyzing-ransomware-encryption-mechanisms",
    "analyzing-ransomware-leak-site-intelligence",
    "analyzing-ransomware-network-indicators",
    "analyzing-ransomware-payment-wallets",
    "analyzing-sbom-for-supply-chain-vulnerabilities",
    "analyzing-security-logs-with-splunk",
    "analyzing-slack-space-and-file-system-artifacts",
    "analyzing-supply-chain-malware-artifacts",
    "analyzing-threat-actor-ttps-with-mitre-attack",
    "analyzing-threat-actor-ttps-with-mitre-navigator",
    "analyzing-threat-intelligence-feeds",
    "analyzing-threat-landscape-with-misp",
    "analyzing-tls-certificate-transparency-logs",
    "analyzing-typosquatting-domains-with-dnstwist",
    "analyzing-uefi-bootkit-persistence",
    "analyzing-usb-device-connection-history",
    "analyzing-web-server-logs-for-intrusion",
    "analyzing-windows-amcache-artifacts",
    "analyzing-windows-event-logs-in-splunk",
    "analyzing-windows-lnk-files-for-artifacts",
    "analyzing-windows-prefetch-with-python",
    "analyzing-windows-registry-for-artifacts",
    "analyzing-windows-shellbag-artifacts",
    "assessing-vector-and-embedding-weaknesses",
    "attacking-entra-id-with-roadtools",
    "attacking-oauth-with-device-code-phishing",
    "auditing-aws-s3-bucket-permissions",
    "auditing-azure-active-directory-configuration",
    "auditing-cloud-with-cis-benchmarks",
    "auditing-entra-id-with-aadinternals",
    "auditing-foundry-smart-contract-security",
    "auditing-gcp-iam-permissions",
    "auditing-kubernetes-cluster-rbac",
    "auditing-mcp-servers-for-tool-poisoning",
    "auditing-terraform-infrastructure-for-security",
    "auditing-tls-certificate-transparency-logs",
    "auditing-uefi-firmware-with-chipsec",
    "automating-ioc-enrichment",
    "building-adversary-infrastructure-tracking-system",
    "building-attack-pattern-library-from-cti-reports",
    "building-automated-malware-submission-pipeline",
    "building-c2-infrastructure-with-sliver-framework",
    "building-c2-redirector-infrastructure",
    "building-detection-rule-with-splunk-spl",
    "building-detection-rules-with-sigma",
    "building-devsecops-pipeline-with-gitlab-ci",
    "building-identity-federation-with-saml-azure-ad",
    "building-identity-governance-lifecycle-process",
    "building-incident-response-dashboard",
    "building-incident-response-playbook",
    "building-incident-timeline-with-timesketch",
    "building-ioc-defanging-and-sharing-pipeline",
    "building-ioc-enrichment-pipeline-with-opencti",
    "building-malware-incident-communication-template",
    "building-patch-tuesday-response-process",
    "building-phishing-reporting-button-workflow",
    "building-ransomware-playbook-with-cisa-framework",
    "building-red-team-c2-infrastructure-with-havoc",
    "building-role-mining-for-rbac-optimization",
    "building-soc-escalation-matrix",
    "building-soc-metrics-and-kpi-tracking",
    "building-soc-playbook-for-ransomware",
    "building-super-timelines-with-plaso",
    "building-threat-actor-profile-from-osint",
    "building-threat-feed-aggregation-with-misp",
    "building-threat-hunt-hypothesis-framework",
    "building-threat-intelligence-enrichment-in-splunk",
    "building-threat-intelligence-feed-integration",
    "building-threat-intelligence-platform",
    "building-vulnerability-aging-and-sla-tracking",
    "building-vulnerability-dashboard-with-defectdojo",
    "building-vulnerability-exception-tracking-system",
    "building-vulnerability-scanning-workflow",
    "bypassing-authentication-with-forced-browsing",
    "coercing-authentication-with-coercer-petitpotam",
    "collecting-indicators-of-compromise",
    "collecting-open-source-intelligence",
    "collecting-threat-intelligence-with-misp",
    "collecting-volatile-evidence-from-compromised-host",
    "conducting-api-security-testing",
    "conducting-cloud-incident-response",
    "conducting-cloud-penetration-testing",
    "conducting-cyber-risk-assessment-with-nist-800-30",
    "conducting-domain-persistence-with-dcsync",
    "conducting-external-reconnaissance-with-osint",
    "conducting-full-scope-red-team-engagement",
    "conducting-internal-network-penetration-test",
    "conducting-internal-reconnaissance-with-bloodhound-ce",
    "conducting-malware-incident-response",
    "conducting-man-in-the-middle-attack-simulation",
    "conducting-memory-forensics-with-volatility",
    "conducting-mobile-app-penetration-test",
    "conducting-network-penetration-test",
    "conducting-pass-the-ticket-attack",
    "conducting-phishing-incident-response",
    "conducting-post-incident-lessons-learned",
    "conducting-social-engineering-penetration-test",
    "conducting-social-engineering-pretext-call",
    "conducting-spearphishing-simulation-campaign",
    "conducting-wireless-network-penetration-test",
    "configuring-active-directory-tiered-model",
    "configuring-aws-verified-access-for-ztna",
    "configuring-certificate-authority-with-openssl",
    "configuring-host-based-intrusion-detection",
    "configuring-hsm-for-key-storage",
    "configuring-identity-aware-proxy-with-google-iap",
    "configuring-ldap-security-hardening",
    "configuring-microsegmentation-for-zero-trust",
    "configuring-multi-factor-authentication-with-duo",
    "configuring-network-segmentation-with-vlans",
    "configuring-oauth2-authorization-flow",
    "configuring-pfsense-firewall-rules",
    "configuring-snort-ids-for-intrusion-detection",
    "configuring-suricata-for-network-monitoring",
    "configuring-tls-1-3-for-secure-communications",
    "configuring-windows-defender-advanced-settings",
    "configuring-windows-event-logging-for-detection",
    "configuring-zscaler-private-access-for-ztna",
    "containing-active-breach",
    "continuous-llm-red-teaming-with-promptfoo",
    "correlating-security-events-in-qradar",
    "correlating-threat-campaigns",
    "defending-llms-with-guardrails",
    "deobfuscating-javascript-malware",
    "deobfuscating-powershell-obfuscated-malware",
    "deploying-active-directory-honeytokens",
    "deploying-cloud-deception-with-decoy-resources",
    "deploying-cloudflare-access-for-zero-trust",
    "deploying-decoy-files-for-ransomware-detection",
    "deploying-edr-agent-with-crowdstrike",
    "deploying-honeytokens-and-canarytokens",
    "deploying-osquery-for-endpoint-monitoring",
    "deploying-palo-alto-prisma-access-zero-trust",
    "deploying-ransomware-canary-files",
    "deploying-software-defined-perimeter",
    "deploying-tailscale-for-zero-trust-vpn",
    "designing-adversary-engagement-with-mitre-engage",
    "detecting-ai-model-prompt-injection-attacks",
    "detecting-anomalies-in-industrial-control-systems",
    "detecting-anomalous-authentication-patterns",
    "detecting-api-enumeration-attacks",
    "detecting-arp-poisoning-in-network-traffic",
    "detecting-attacks-on-historian-servers",
    "detecting-attacks-on-scada-systems",
    "detecting-aws-cloudtrail-anomalies",
    "detecting-aws-credential-exposure-with-trufflehog",
    "detecting-aws-guardduty-findings-automation",
    "detecting-aws-iam-privilege-escalation",
    "detecting-azure-lateral-movement",
    "detecting-azure-service-principal-abuse",
    "detecting-azure-storage-account-misconfigurations",
    "detecting-beaconing-patterns-with-zeek",
    "detecting-bluetooth-low-energy-attacks",
    "detecting-broken-object-property-level-authorization",
    "detecting-business-email-compromise",
    "detecting-business-email-compromise-with-ai",
    "detecting-cloud-threats-with-guardduty",
    "detecting-command-and-control-over-dns",
    "detecting-compromised-cloud-credentials",
    "detecting-credential-dumping-techniques",
    "detecting-cryptomining-in-cloud",
    "detecting-data-and-model-poisoning",
    "detecting-dcsync-attack-in-active-directory",
    "detecting-deepfake-audio-in-vishing-attacks",
    "detecting-dependency-confusion",
    "detecting-dll-sideloading-attacks",
    "detecting-dnp3-protocol-anomalies",
    "detecting-dns-exfiltration-with-dns-query-analysis",
    "detecting-email-account-compromise",
    "detecting-email-forwarding-rules-attack",
    "detecting-entra-offensive-tools-in-graph-logs",
    "detecting-evasion-techniques-in-endpoint-logs",
    "detecting-exfiltration-over-dns-with-zeek",
    "detecting-fileless-attacks-on-endpoints",
    "detecting-fileless-malware-techniques",
    "detecting-golden-ticket-attacks-in-kerberos-logs",
    "detecting-golden-ticket-forgery",
    "detecting-indirect-prompt-injection",
    "detecting-insider-data-exfiltration-via-dlp",
    "detecting-insider-threat-behaviors",
    "detecting-insider-threat-with-ueba",
    "detecting-kerberoasting-attacks",
    "detecting-lateral-movement-in-network",
    "detecting-lateral-movement-with-splunk",
    "detecting-lateral-movement-with-zeek",
    "detecting-living-off-the-land-attacks",
    "detecting-living-off-the-land-with-lolbas",
    "detecting-malicious-npm-packages",
    "detecting-malicious-scheduled-tasks-with-sysmon",
    "detecting-mimikatz-execution-patterns",
    "detecting-misconfigured-azure-storage",
    "detecting-mobile-malware-behavior",
    "detecting-modbus-command-injection-attacks",
    "detecting-modbus-protocol-anomalies",
    "detecting-model-extraction-attacks",
    "detecting-network-anomalies-with-zeek",
    "detecting-network-scanning-with-ids-signatures",
    "detecting-ntlm-relay-with-event-correlation",
    "detecting-oauth-token-theft",
    "detecting-pass-the-hash-attacks",
    "detecting-pass-the-ticket-attacks",
    "detecting-port-scanning-with-fail2ban",
    "detecting-privilege-escalation-attempts",
    "detecting-process-hollowing-technique",
    "detecting-process-injection-techniques",
    "detecting-qr-code-phishing-with-email-security",
    "detecting-ransomware-encryption-behavior",
    "detecting-ransomware-precursors-in-network",
    "detecting-rdp-brute-force-attacks",
    "detecting-rootkit-activity",
    "detecting-s3-data-exfiltration-attempts",
    "detecting-secure-boot-bypass",
    "detecting-serverless-function-injection",
    "detecting-service-account-abuse",
    "detecting-shadow-api-endpoints",
    "detecting-shadow-it-cloud-usage",
    "detecting-spearphishing-with-email-gateway",
    "detecting-sql-injection-via-waf-logs",
    "detecting-stuxnet-style-attacks",
    "detecting-supply-chain-attacks-in-ci-cd",
    "detecting-suspicious-oauth-application-consent",
    "detecting-suspicious-powershell-execution",
    "detecting-t1003-credential-dumping-with-edr",
    "detecting-t1055-process-injection-with-sysmon",
    "detecting-t1548-abuse-elevation-control-mechanism",
    "detecting-typosquatting-packages",
    "detecting-typosquatting-packages-in-npm-pypi",
    "detecting-wmi-persistence",
    "emulating-cloud-attacks-with-stratus-red-team",
    "enumerating-cloud-with-cloudfox",
    "eradicating-malware-from-infected-systems",
    "evaluating-threat-intelligence-platforms",
    "executing-active-directory-attack-simulation",
    "executing-nist-rmf-authorization-to-operate",
    "executing-phishing-simulation-campaign",
    "executing-red-team-engagement-planning",
    "executing-red-team-exercise",
    "exploiting-active-directory-certificate-services-esc1",
    "exploiting-active-directory-with-bloodhound",
    "exploiting-adcs-with-certipy",
    "exploiting-api-injection-vulnerabilities",
    "exploiting-aws-with-pacu",
    "exploiting-bgp-hijacking-vulnerabilities",
    "exploiting-broken-function-level-authorization",
    "exploiting-broken-link-hijacking",
    "exploiting-constrained-delegation-abuse",
    "exploiting-deeplink-vulnerabilities",
    "exploiting-excessive-data-exposure-in-api",
    "exploiting-http-request-smuggling",
    "exploiting-idor-vulnerabilities",
    "exploiting-insecure-data-storage-in-mobile",
    "exploiting-insecure-deserialization",
    "exploiting-ipv6-vulnerabilities",
    "exploiting-jwt-algorithm-confusion-attack",
    "exploiting-kerberoasting-with-impacket",
    "exploiting-mass-assignment-in-rest-apis",
    "exploiting-ms17-010-eternalblue-vulnerability",
    "exploiting-nopac-cve-2021-42278-42287",
    "exploiting-nosql-injection-vulnerabilities",
    "exploiting-oauth-misconfiguration",
    "exploiting-prototype-pollution-in-javascript",
    "exploiting-race-condition-vulnerabilities",
    "exploiting-server-side-request-forgery",
    "exploiting-smb-vulnerabilities-with-metasploit",
    "exploiting-sql-injection-vulnerabilities",
    "exploiting-sql-injection-with-sqlmap",
    "exploiting-template-injection-vulnerabilities",
    "exploiting-type-juggling-vulnerabilities",
    "exploiting-vulnerabilities-with-metasploit-framework",
    "exploiting-websocket-vulnerabilities",
    "exploiting-zerologon-vulnerability-cve-2020-1472",
    "extracting-browser-history-artifacts",
    "extracting-config-from-agent-tesla-rat",
    "extracting-credentials-from-memory-dump",
    "extracting-iocs-from-malware-samples",
    "extracting-memory-artifacts-with-rekall",
    "extracting-windows-event-logs-artifacts",
    "fleet-hunting-with-velociraptor",
    "generating-and-analyzing-sboms",
    "generating-forensic-timelines-with-hayabusa",
    "generating-threat-intelligence-reports",
    "hardening-linux-endpoint-with-cis-benchmark",
    "hardening-windows-endpoint-with-cis-benchmark",
    "hunting-advanced-persistent-threats",
    "hunting-bootkits-in-efi-system-partition",
    "hunting-credential-stuffing-attacks",
    "hunting-evtx-with-chainsaw",
    "hunting-for-anomalous-powershell-execution",
    "hunting-for-beaconing-with-frequency-analysis",
    "hunting-for-cobalt-strike-beacons",
    "hunting-for-command-and-control-beaconing",
    "hunting-for-data-exfiltration-indicators",
    "hunting-for-data-staging-before-exfiltration",
    "hunting-for-dcom-lateral-movement",
    "hunting-for-dcsync-attacks",
    "hunting-for-defense-evasion-via-timestomping",
    "hunting-for-dns-based-persistence",
    "hunting-for-dns-tunneling-with-zeek",
    "hunting-for-domain-fronting-c2-traffic",
    "hunting-for-lateral-movement-via-wmi",
    "hunting-for-living-off-the-cloud-techniques",
    "hunting-for-living-off-the-land-binaries",
    "hunting-for-lolbins-execution-in-endpoint-logs",
    "hunting-for-ntlm-relay-attacks",
    "hunting-for-persistence-mechanisms-in-windows",
    "hunting-for-persistence-via-wmi-subscriptions",
    "hunting-for-process-injection-techniques",
    "hunting-for-registry-persistence-mechanisms",
    "hunting-for-registry-run-key-persistence",
    "hunting-for-scheduled-task-persistence",
    "hunting-for-shadow-copy-deletion",
    "hunting-for-spearphishing-indicators",
    "hunting-for-startup-folder-persistence",
    "hunting-for-supply-chain-compromise",
    "hunting-for-suspicious-scheduled-tasks",
    "hunting-for-t1098-account-manipulation",
    "hunting-for-unusual-network-connections",
    "hunting-for-unusual-service-installations",
    "hunting-for-webshell-activity",
    "hunting-saas-sso-token-abuse",
    "implementing-aes-encryption-for-data-at-rest",
    "implementing-alert-fatigue-reduction",
    "implementing-anti-phishing-training-program",
    "implementing-anti-ransomware-group-policy",
    "implementing-api-abuse-detection-with-rate-limiting",
    "implementing-api-gateway-security-controls",
    "implementing-api-key-security-controls",
    "implementing-api-rate-limiting-and-throttling",
    "implementing-api-schema-validation-security",
    "implementing-api-security-posture-management",
    "implementing-api-security-testing-with-42crunch",
    "implementing-api-threat-protection-with-apigee",
    "implementing-application-whitelisting-with-applocker",
    "implementing-aqua-security-for-container-scanning",
    "implementing-attack-path-analysis-with-xm-cyber",
    "implementing-attack-surface-management",
    "implementing-aws-config-rules-for-compliance",
    "implementing-aws-iam-permission-boundaries",
    "implementing-aws-macie-for-data-classification",
    "implementing-aws-nitro-enclave-security",
    "implementing-azure-ad-privileged-identity-management",
    "implementing-bgp-security-with-rpki",
    "implementing-browser-isolation-for-zero-trust",
    "implementing-canary-tokens-for-network-intrusion",
    "implementing-cisa-zero-trust-maturity-model",
    "implementing-cloud-trail-log-analysis",
    "implementing-cloud-vulnerability-posture-management",
    "implementing-cloud-waf-rules",
    "implementing-cloud-workload-protection",
    "implementing-code-signing-for-artifacts",
    "implementing-conditional-access-policies-azure-ad",
    "implementing-conduit-security-for-ot-remote-access",
    "implementing-continuous-security-validation-with-bas",
    "implementing-data-loss-prevention-with-microsoft-purview",
    "implementing-ddos-mitigation-with-cloudflare",
    "implementing-deception-based-detection-with-canarytoken",
    "implementing-delinea-secret-server-for-pam",
    "implementing-device-posture-assessment-in-zero-trust",
    "implementing-devsecops-security-scanning",
    "implementing-diamond-model-analysis",
    "implementing-digital-signatures-with-ed25519",
    "implementing-disk-encryption-with-bitlocker",
    "implementing-dmarc-dkim-spf-email-security",
    "implementing-ebpf-security-monitoring",
    "implementing-email-sandboxing-with-proofpoint",
    "implementing-end-to-end-encryption-for-messaging",
    "implementing-endpoint-detection-with-wazuh",
    "implementing-endpoint-dlp-controls",
    "implementing-envelope-encryption-with-aws-kms",
    "implementing-epss-score-for-vulnerability-prioritization",
    "implementing-file-integrity-monitoring-with-aide",
    "implementing-fuzz-testing-in-cicd-with-aflplusplus",
    "implementing-gcp-binary-authorization",
    "implementing-gcp-organization-policy-constraints",
    "implementing-gdpr-data-protection-controls",
    "implementing-gdpr-data-subject-access-request",
    "implementing-github-advanced-security-for-code-scanning",
    "implementing-google-workspace-admin-security",
    "implementing-google-workspace-phishing-protection",
    "implementing-google-workspace-sso-configuration",
    "implementing-hardware-security-key-authentication",
    "implementing-hashicorp-vault-dynamic-secrets",
    "implementing-hipaa-security-rule-safeguards",
    "implementing-honeypot-for-ransomware-detection",
    "implementing-honeytokens-for-breach-detection",
    "implementing-ics-firewall-with-tofino",
    "implementing-identity-governance-with-sailpoint",
    "implementing-identity-verification-for-zero-trust",
    "implementing-iec-62443-security-zones",
    "implementing-immutable-backup-with-restic",
    "implementing-infrastructure-as-code-security-scanning",
    "implementing-iso-27001-information-security-management",
    "implementing-just-in-time-access-provisioning",
    "implementing-jwt-signing-and-verification",
    "implementing-llm-guardrails-for-security",
    "implementing-log-forwarding-with-fluentd",
    "implementing-log-integrity-with-blockchain",
    "implementing-memory-protection-with-dep-aslr",
    "implementing-microsegmentation-with-guardicore",
    "implementing-mimecast-targeted-attack-protection",
    "implementing-mitre-attack-coverage-mapping",
    "implementing-mobile-application-management",
    "implementing-mtls-for-zero-trust-services",
    "implementing-nerc-cip-compliance-controls",
    "implementing-network-access-control",
    "implementing-network-access-control-with-cisco-ise",
    "implementing-network-deception-with-honeypots",
    "implementing-network-intrusion-prevention-with-suricata",
    "implementing-network-segmentation-for-ot",
    "implementing-network-segmentation-with-firewall-zones",
    "implementing-network-traffic-analysis-with-arkime",
    "implementing-network-traffic-baselining",
    "implementing-next-generation-firewall-with-palo-alto",
    "implementing-ot-incident-response-playbook",
    "implementing-ot-network-traffic-analysis-with-nozomi",
    "implementing-pam-for-database-access",
    "implementing-passwordless-auth-with-microsoft-entra",
    "implementing-passwordless-authentication-with-fido2",
    "implementing-patch-management-for-ot-systems",
    "implementing-patch-management-workflow",
    "implementing-pci-dss-compliance-controls",
    "implementing-policy-as-code-with-open-policy-agent",
    "implementing-privileged-access-management-with-cyberark",
    "implementing-privileged-access-workstation",
    "implementing-privileged-session-monitoring",
    "implementing-proofpoint-email-security-gateway",
    "implementing-purdue-model-network-segmentation",
    "implementing-ransomware-backup-strategy",
    "implementing-ransomware-kill-switch-detection",
    "implementing-rapid7-insightvm-for-scanning",
    "implementing-rsa-key-pair-management",
    "implementing-runtime-application-self-protection",
    "implementing-saml-sso-with-okta",
    "implementing-scim-provisioning-with-okta",
    "implementing-secret-scanning-with-gitleaks",
    "implementing-secrets-management-with-vault",
    "implementing-secrets-scanning-in-ci-cd",
    "implementing-security-chaos-engineering",
    "implementing-security-information-sharing-with-stix2",
    "implementing-security-monitoring-with-datadog",
    "implementing-semgrep-for-custom-sast-rules",
    "implementing-siem-correlation-rules-for-apt",
    "implementing-siem-use-case-tuning",
    "implementing-siem-use-cases-for-detection",
    "implementing-sigstore-for-software-signing",
    "implementing-soar-automation-with-phantom",
    "implementing-soar-playbook-for-phishing",
    "implementing-soar-playbook-with-palo-alto-xsoar",
    "implementing-stix-taxii-feed-integration",
    "implementing-syslog-centralization-with-rsyslog",
    "implementing-taxii-server-with-opentaxii",
    "implementing-threat-intelligence-lifecycle-management",
    "implementing-threat-modeling-with-mitre-attack",
    "implementing-ticketing-system-for-incidents",
    "implementing-usb-device-control-policy",
    "implementing-velociraptor-for-ir-collection",
    "implementing-vulnerability-management-with-greenbone",
    "implementing-vulnerability-remediation-sla",
    "implementing-vulnerability-sla-breach-alerting",
    "implementing-web-application-logging-with-modsecurity",
    "implementing-zero-knowledge-proof-for-authentication",
    "implementing-zero-standing-privilege-with-cyberark",
    "implementing-zero-trust-dns-with-nextdns",
    "implementing-zero-trust-for-saas-applications",
    "implementing-zero-trust-in-cloud",
    "implementing-zero-trust-network-access",
    "implementing-zero-trust-network-access-with-zscaler",
    "implementing-zero-trust-with-beyondcorp",
    "implementing-zero-trust-with-hashicorp-boundary",
    "integrating-dast-with-owasp-zap-in-pipeline",
    "integrating-sast-into-github-actions-pipeline",
    "intercepting-mobile-traffic-with-burpsuite",
    "investigating-insider-threat-indicators",
    "investigating-phishing-email-incident",
    "investigating-ransomware-attack-artifacts",
    "managing-cloud-identity-with-okta",
    "managing-intelligence-lifecycle",
    "managing-third-party-vendor-risk",
    "mapping-attack-paths-with-bloodhound-ce",
    "mapping-mitre-attack-techniques",
    "migrating-to-post-quantum-cryptography",
    "modeling-threats-with-opencti",
    "monitoring-darkweb-sources",
    "monitoring-scada-modbus-traffic-anomalies",
    "moving-laterally-with-netexec",
    "operating-havoc-c2",
    "operating-sliver-c2",
    "operationalizing-misp-threat-feeds",
    "orchestrating-llm-attacks-with-pyrit",
    "parsing-artifacts-with-eric-zimmerman-tools",
    "performing-access-recertification-with-saviynt",
    "performing-access-review-and-certification",
    "performing-active-directory-bloodhound-analysis",
    "performing-active-directory-compromise-investigation",
    "performing-active-directory-forest-trust-attack",
    "performing-active-directory-penetration-test",
    "performing-active-directory-vulnerability-assessment",
    "performing-adversary-in-the-middle-phishing-detection",
    "performing-agentless-vulnerability-scanning",
    "performing-ai-driven-osint-correlation",
    "performing-alert-triage-with-elastic-siem",
    "performing-android-app-static-analysis-with-mobsf",
    "performing-api-fuzzing-with-restler",
    "performing-api-inventory-and-discovery",
    "performing-api-rate-limiting-bypass",
    "performing-api-security-testing-with-postman",
    "performing-asset-criticality-scoring-for-vulns",
    "performing-authenticated-scan-with-openvas",
    "performing-authenticated-vulnerability-scan",
    "performing-automated-malware-analysis-with-cape",
    "performing-aws-account-enumeration-with-scout-suite",
    "performing-aws-privilege-escalation-assessment",
    "performing-bandwidth-throttling-attack-simulation",
    "performing-binary-exploitation-analysis",
    "performing-blind-ssrf-exploitation",
    "performing-bluetooth-security-assessment",
    "performing-brand-monitoring-for-impersonation",
    "performing-clickjacking-attack-test",
    "performing-cloud-asset-inventory-with-cartography",
    "performing-cloud-forensics-investigation",
    "performing-cloud-forensics-with-aws-cloudtrail",
    "performing-cloud-incident-containment-procedures",
    "performing-cloud-log-forensics-with-athena",
    "performing-cloud-native-forensics-with-falco",
    "performing-cloud-native-threat-hunting-with-aws-detective",
    "performing-cloud-penetration-testing-with-pacu",
    "performing-cloud-storage-forensic-acquisition",
    "performing-container-image-hardening",
    "performing-content-security-policy-bypass",
    "performing-credential-access-with-lazagne",
    "performing-cryptographic-audit-of-application",
    "performing-csrf-attack-simulation",
    "performing-cve-prioritization-with-kev-catalog",
    "performing-dark-web-monitoring-for-threats",
    "performing-deception-technology-deployment",
    "performing-directory-traversal-testing",
    "performing-disk-forensics-investigation",
    "performing-dmarc-policy-enforcement-rollout",
    "performing-dns-enumeration-and-zone-transfer",
    "performing-dns-tunneling-detection",
    "performing-dynamic-analysis-of-android-app",
    "performing-dynamic-analysis-with-any-run",
    "performing-endpoint-forensics-investigation",
    "performing-endpoint-vulnerability-remediation",
    "performing-entitlement-review-with-sailpoint-iiq",
    "performing-external-network-penetration-test",
    "performing-false-positive-reduction-in-siem",
    "performing-file-carving-with-foremost",
    "performing-firmware-extraction-with-binwalk",
    "performing-firmware-malware-analysis",
    "performing-fuzzing-with-aflplusplus",
    "performing-gcp-penetration-testing-with-gcpbucketbrute",
    "performing-gcp-security-assessment-with-forseti",
    "performing-graphql-depth-limit-attack",
    "performing-graphql-introspection-attack",
    "performing-graphql-security-assessment",
    "performing-hardware-security-module-integration",
    "performing-hash-cracking-with-hashcat",
    "performing-http-parameter-pollution-attack",
    "performing-indicator-lifecycle-management",
    "performing-initial-access-with-evilginx3",
    "performing-insider-threat-investigation",
    "performing-ioc-enrichment-automation",
    "performing-ios-app-security-assessment",
    "performing-iot-security-assessment",
    "performing-ip-reputation-analysis-with-shodan",
    "performing-jwt-none-algorithm-attack",
    "performing-kerberoasting-attack",
    "performing-lateral-movement-with-wmiexec",
    "performing-linux-log-forensics-investigation",
    "performing-log-analysis-for-forensic-investigation",
    "performing-log-source-onboarding-in-siem",
    "performing-malware-hash-enrichment-with-virustotal",
    "performing-malware-ioc-extraction",
    "performing-malware-persistence-investigation",
    "performing-malware-triage-with-yara",
    "performing-memory-forensics-with-volatility3",
    "performing-memory-forensics-with-volatility3-plugins",
    "performing-mobile-app-certificate-pinning-bypass",
    "performing-mobile-device-forensics-with-cellebrite",
    "performing-network-forensics-with-wireshark",
    "performing-network-packet-capture-analysis",
    "performing-network-traffic-analysis-with-tshark",
    "performing-network-traffic-analysis-with-zeek",
    "performing-nist-csf-maturity-assessment",
    "performing-oauth-scope-minimization-review",
    "performing-open-source-intelligence-gathering",
    "performing-osint-with-spiderfoot",
    "performing-ot-network-security-assessment",
    "performing-ot-vulnerability-scanning-safely",
    "performing-packet-injection-attack",
    "performing-paste-site-monitoring-for-credentials",
    "performing-phishing-simulation-with-gophish",
    "performing-physical-intrusion-assessment",
    "performing-plc-firmware-security-analysis",
    "performing-post-quantum-cryptography-migration",
    "performing-privacy-impact-assessment",
    "performing-privilege-escalation-assessment",
    "performing-privilege-escalation-on-linux",
    "performing-privileged-account-access-review",
    "performing-privileged-account-discovery",
    "performing-purple-team-atomic-testing",
    "performing-purple-team-exercise",
    "performing-ransomware-response",
    "performing-ransomware-tabletop-exercise",
    "performing-red-team-phishing-with-gophish",
    "performing-red-team-with-covenant",
    "performing-s7comm-protocol-security-analysis",
    "performing-sca-dependency-scanning-with-snyk",
    "performing-scada-hmi-security-assessment",
    "performing-second-order-sql-injection",
    "performing-security-headers-audit",
    "performing-serverless-function-security-review",
    "performing-service-account-audit",
    "performing-service-account-credential-rotation",
    "performing-soap-web-service-security-testing",
    "performing-soc-tabletop-exercise",
    "performing-soc2-type2-audit-preparation",
    "performing-sqlite-database-forensics",
    "performing-ssl-certificate-lifecycle-management",
    "performing-ssl-stripping-attack",
    "performing-ssl-tls-inspection-configuration",
    "performing-ssl-tls-security-assessment",
    "performing-ssrf-vulnerability-exploitation",
    "performing-static-malware-analysis-with-pe-studio",
    "performing-steganography-detection",
    "performing-subdomain-enumeration-with-subfinder",
    "performing-supply-chain-attack-simulation",
    "performing-thick-client-application-penetration-test",
    "performing-threat-emulation-with-atomic-red-team",
    "performing-threat-hunting-with-elastic-siem",
    "performing-threat-hunting-with-yara-rules",
    "performing-threat-intelligence-sharing-with-misp",
    "performing-threat-landscape-assessment-for-sector",
    "performing-threat-modeling-with-owasp-threat-dragon",
    "performing-timeline-reconstruction-with-plaso",
    "performing-user-behavior-analytics",
    "performing-vlan-hopping-attack",
    "performing-vulnerability-scanning-with-nessus",
    "performing-web-application-firewall-bypass",
    "performing-web-application-penetration-test",
    "performing-web-application-scanning-with-nikto",
    "performing-web-application-vulnerability-triage",
    "performing-web-cache-deception-attack",
    "performing-web-cache-poisoning-attack",
    "performing-wifi-password-cracking-with-aircrack",
    "performing-windows-artifact-analysis-with-eric-zimmerman-tools",
    "performing-wireless-network-penetration-test",
    "performing-wireless-security-assessment-with-kismet",
    "performing-yara-rule-development-for-detection",
    "post-exploiting-microsoft-graph-with-graphrunner",
    "prioritizing-vulnerabilities-with-cvss-scoring",
    "processing-stix-taxii-feeds",
    "profiling-threat-actor-groups",
    "recovering-deleted-files-with-photorec",
    "recovering-from-ransomware-attack",
    "red-teaming-llms-with-garak",
    "relaying-ntlm-for-adcs-esc8",
    "remediating-s3-bucket-misconfiguration",
    "reverse-engineering-android-malware-with-jadx",
    "reverse-engineering-dotnet-malware-with-dnspy",
    "reverse-engineering-ios-app-with-frida",
    "reverse-engineering-malware-with-ghidra",
    "reverse-engineering-ransomware-encryption-routine",
    "reverse-engineering-rust-malware",
    "scanning-containers-with-trivy-in-cicd",
    "scanning-iac-and-images-with-trivy",
    "scanning-infrastructure-with-nessus",
    "scanning-network-with-nmap-advanced",
    "securing-agentic-ai-tool-invocation",
    "securing-api-gateway-with-aws-waf",
    "securing-aws-iam-permissions",
    "securing-aws-lambda-execution-roles",
    "securing-azure-with-microsoft-defender",
    "securing-container-registry-images",
    "securing-github-actions-workflows",
    "securing-historian-server-in-ot-environment",
    "securing-kubernetes-on-cloud",
    "securing-remote-access-to-ot-environment",
    "securing-serverless-functions",
    "testing-android-intents-for-vulnerabilities",
    "testing-api-authentication-weaknesses",
    "testing-api-for-broken-object-level-authorization",
    "testing-api-for-mass-assignment-vulnerability",
    "testing-api-security-with-owasp-top-10",
    "testing-cors-misconfiguration",
    "testing-for-broken-access-control",
    "testing-for-business-logic-vulnerabilities",
    "testing-for-email-header-injection",
    "testing-for-host-header-injection",
    "testing-for-json-web-token-vulnerabilities",
    "testing-for-open-redirect-vulnerabilities",
    "testing-for-sensitive-data-exposure",
    "testing-for-system-prompt-leakage",
    "testing-for-xml-injection-vulnerabilities",
    "testing-for-xss-vulnerabilities",
    "testing-for-xss-vulnerabilities-with-burpsuite",
    "testing-for-xxe-injection-vulnerabilities",
    "testing-jwt-token-security",
    "testing-mobile-api-authentication",
    "testing-oauth2-implementation-flaws",
    "testing-prompt-injection-in-rag-pipelines",
    "testing-ransomware-recovery-procedures",
    "testing-websocket-api-security",
    "tracking-threat-actor-infrastructure",
    "triaging-security-alerts-in-splunk",
    "triaging-security-incident",
    "triaging-security-incident-with-ir-playbook",
    "triaging-vulnerabilities-with-ssvc-framework",
    "triaging-windows-with-kape",
    "validating-backup-integrity-for-recovery",
    "validating-tpm-measured-boot-attestation",
    "verifying-build-provenance-with-slsa-sigstore"
  ],
  "body-max-lines": [
    "building-automated-malware-submission-pipeline",
    "building-identity-governance-lifecycle-process",
    "detecting-anomalous-authentication-patterns",
    "detecting-attacks-on-scada-systems",
    "detecting-command-and-control-over-dns",
    "detecting-living-off-the-land-attacks",
    "detecting-modbus-command-injection-attacks",
    "detecting-ntlm-relay-with-event-correlation",
    "detecting-serverless-function-injection",
    "hunting-for-dcom-lateral-movement",
    "implementing-data-loss-prevention-with-microsoft-purview",
    "implementing-google-workspace-admin-security",
    "implementing-hashicorp-vault-dynamic-secrets",
    "implementing-iec-62443-security-zones",
    "implementing-passwordless-auth-with-microsoft-entra",
    "implementing-zero-trust-with-hashicorp-boundary",
    "performing-cloud-log-forensics-with-athena",
    "performing-graphql-introspection-attack",
    "performing-ics-asset-discovery-with-claroty",
    "performing-oauth-scope-minimization-review",
    "performing-ot-network-security-assessment",
    "performing-plc-firmware-security-analysis",
    "performing-purple-team-atomic-testing"
  ]
}

===== END FILE =====


===== FILE: ./tools/lint-descriptions.py =====
#!/usr/bin/env python3
"""Lint SKILL.md descriptions for the qualities that drive agent routing.

The description is the ONLY signal an agent sees at discovery time, so it must
say what the skill does, when to fire, and -- critically -- when NOT to fire.
Skills whose descriptions overlap without a distinguishing negative trigger get
misrouted ("skill collision").

Rules
-----
  name-matches-folder        frontmatter name == directory name
  desc-max-length            description <= 1024 chars (agentskills.io limit)
  desc-ends-punctuation      ends in . ! ? ) " '  -- a truncation canary
  desc-has-use-when          carries an explicit trigger clause
  desc-has-negative-trigger  says what it is NOT for
  body-max-lines             SKILL.md <= 500 lines (Anthropic guidance)

Grandfathering
--------------
Known pre-existing failures live in tools/lint-baseline.json so CI can go green
today while the debt is paid down. A baselined skill that starts passing is
reported as a stale entry -- run --update-baseline to shrink the file. The
baseline can only ever shrink in review, never silently grow: any NEW failure
is a hard error.

Usage:
    python tools/lint-descriptions.py --all
    python tools/lint-descriptions.py --all --stats
    python tools/lint-descriptions.py --update-baseline
    python tools/lint-descriptions.py skills/<slug>
"""
from __future__ import annotations

import argparse
import json
import os
import re
import sys

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))

from skill_frontmatter import description_of, iter_skill_dirs, load_frontmatter, FrontmatterError

BASELINE_PATH = os.path.join(os.path.dirname(os.path.abspath(__file__)), "lint-baseline.json")

DESCRIPTION_MAX_CHARS = 1024
BODY_MAX_LINES = 500
SENTENCE_ENDINGS = ".!?)\"'"

# "Use when ...", "Use this skill when ...", "Use for ...", "Use during ..."
USE_WHEN_RE = re.compile(
    r"\buse\s+(?:this\s+)?(?:skill\s+)?(?:when|whenever|for|during|after|before)\b",
    re.IGNORECASE,
)

# "Do not use for X", "Don't use when Y", "Not for Z"
NEGATIVE_TRIGGER_RE = re.compile(
    r"\b(?:do\s+not\s+use|don'?t\s+use|not\s+for\b|avoid\s+(?:this\s+)?(?:skill\s+)?for)\b",
    re.IGNORECASE,
)

RULES = (
    "name-matches-folder",
    "desc-max-length",
    "desc-ends-punctuation",
    "desc-has-use-when",
    "desc-has-negative-trigger",
    "body-max-lines",
)

RED, GREEN, YELLOW, DIM, RESET = "\033[91m", "\033[92m", "\033[93m", "\033[2m", "\033[0m"


def check_skill(slug: str, skill_dir: str) -> dict[str, str]:
    """Return {rule_id: human message} for every rule this skill violates."""
    skill_md = os.path.join(skill_dir, "SKILL.md")
    violations: dict[str, str] = {}

    try:
        frontmatter = load_frontmatter(skill_md)
    except FrontmatterError as exc:
        return {"name-matches-folder": f"unparseable frontmatter: {exc}"}

    name = str(frontmatter.get("name", "") or "")
    if name != slug:
        violations["name-matches-folder"] = f"name {name!r} != folder {slug!r}"

    description = description_of(frontmatter)

    if len(description) > DESCRIPTION_MAX_CHARS:
        violations["desc-max-length"] = (
            f"{len(description)} chars, max {DESCRIPTION_MAX_CHARS}")

    if description and description[-1] not in SENTENCE_ENDINGS:
        violations["desc-ends-punctuation"] = f"ends with {description[-40:]!r}"

    if not USE_WHEN_RE.search(description):
        violations["desc-has-use-when"] = "no trigger clause (add 'Use when ...')"

    if not NEGATIVE_TRIGGER_RE.search(description):
        violations["desc-has-negative-trigger"] = (
            "no negative trigger (add 'Do not use for X - use <other-skill>.')")

    with open(skill_md, encoding="utf-8") as handle:
        line_count = sum(1 for _ in handle)
    if line_count > BODY_MAX_LINES:
        violations["body-max-lines"] = f"{line_count} lines, max {BODY_MAX_LINES}"

    return violations


def collect(skills_dir: str = "skills") -> dict[str, dict[str, str]]:
    return {slug: v for slug, d in iter_skill_dirs(skills_dir)
            if (v := check_skill(slug, d))}


def load_baseline() -> dict[str, list[str]]:
    if not os.path.isfile(BASELINE_PATH):
        return {}
    with open(BASELINE_PATH, encoding="utf-8") as handle:
        return {k: v for k, v in json.load(handle).items() if not k.startswith("_")}


def write_baseline(violations: dict[str, dict[str, str]]) -> dict[str, list[str]]:
    baseline = {rule: sorted(s for s, v in violations.items() if rule in v)
                for rule in RULES}
    baseline = {rule: slugs for rule, slugs in baseline.items() if slugs}
    payload = {
        "_comment": (
            "Pre-existing lint failures, grandfathered so CI can gate new work today. "
            "This file may only shrink. Never add a slug by hand -- fix the skill, then "
            "run: python tools/lint-descriptions.py --update-baseline"
        ),
        "_total_grandfathered": sum(len(s) for s in baseline.values()),
        **baseline,
    }
    with open(BASELINE_PATH, "w", encoding="utf-8") as handle:
        json.dump(payload, handle, indent=2)
        handle.write("\n")
    return baseline


def main() -> int:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("target", nargs="?", help="a single skills/<slug> directory")
    parser.add_argument("--all", action="store_true", help="lint every skill")
    parser.add_argument("--stats", action="store_true", help="show per-rule totals")
    parser.add_argument("--update-baseline", action="store_true",
                        help="rewrite lint-baseline.json from current state")
    parser.add_argument("--skills-dir", default="skills")
    args = parser.parse_args()

    if not os.path.isdir(args.skills_dir):
        print(f"ERROR: '{args.skills_dir}' not found. Run from the repository root.")
        return 1

    single_skill = bool(args.target) and not (args.all or args.update_baseline)
    if single_skill:
        slug = os.path.basename(args.target.rstrip("/\\"))
        violations = {slug: v} if (v := check_skill(slug, args.target.rstrip("/\\"))) else {}
    else:
        violations = collect(args.skills_dir)

    if args.update_baseline:
        baseline = write_baseline(violations)
        total = sum(len(s) for s in baseline.values())
        print(f"Baseline written: {total} grandfathered violation(s) across "
              f"{len(baseline)} rule(s) -> {os.path.relpath(BASELINE_PATH)}")
        for rule in RULES:
            if baseline.get(rule):
                print(f"  {rule:26s} {len(baseline[rule])}")
        return 0

    baseline = load_baseline()

    new_failures: list[tuple[str, str, str]] = []
    grandfathered = 0
    for slug, rule_map in sorted(violations.items()):
        for rule, message in sorted(rule_map.items()):
            if slug in baseline.get(rule, []):
                grandfathered += 1
            else:
                new_failures.append((slug, rule, message))

    # Only meaningful over the whole tree: linting one skill says nothing about
    # whether the other 816 baselined violations still stand.
    stale = [] if single_skill else [
        (rule, slug) for rule, slugs in baseline.items() for slug in slugs
        if rule not in violations.get(slug, {})
    ]

    if args.stats:
        print("Violations by rule (grandfathered + new):")
        for rule in RULES:
            count = sum(1 for v in violations.values() if rule in v)
            remaining = len(baseline.get(rule, []))
            print(f"  {rule:26s} {count:4d}   baselined {remaining:4d}")
        print()

    for slug, rule, message in new_failures:
        print(f"{RED}FAIL{RESET} {slug}: {YELLOW}{rule}{RESET} - {message}")

    if stale:
        print(f"\n{GREEN}{len(stale)} baselined violation(s) now pass.{RESET} "
              f"Shrink the baseline: python tools/lint-descriptions.py --update-baseline")
        for rule, slug in stale[:10]:
            print(f"  {DIM}fixed{RESET} {slug}: {rule}")

    print(f"\n{'=' * 60}")
    checked = 1 if single_skill else sum(1 for _ in iter_skill_dirs(args.skills_dir))
    print(f"Skills: {checked}   "
          f"{RED}New failures: {len(new_failures)}{RESET}   "
          f"{YELLOW}Grandfathered: {grandfathered}{RESET}")

    return 1 if new_failures else 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: ./tools/skill_frontmatter.py =====
#!/usr/bin/env python3
"""Single source of truth for reading SKILL.md YAML frontmatter.

Every tool in this repository MUST read frontmatter through this module.

Why this exists
---------------
This repo previously carried three independent hand-rolled "YAML-ish" parsers
(the index generator, validate-skill.py, validate-agentskills.py). Each handled
a different subset of YAML scalar styles, and all of them silently truncated
multi-line descriptions to their first line.

A census of the 817 skills shows why that was fatal:

    block scalar   (description: >-)    43
    single-quoted multiline            278
    plain unquoted multiline           496
    single-line                          0

Only the 43 block-scalar files parsed correctly; 774 (94.7%) used a style the
hand-rolled parsers mishandled, and 604 descriptions shipped truncated in
index.json with no error and no warning.

PyYAML handles every scalar style, quoting form and escape correctly. Do not
reintroduce a regex-based frontmatter parser -- CI greps for that.
"""
from __future__ import annotations

import os
import re
from typing import Dict, Iterator, Tuple

import yaml

# Frontmatter is the block between the opening '---' and the next '---' that
# sits alone on its own line. Tolerates CRLF and a leading UTF-8 BOM.
_FRONTMATTER_RE = re.compile(r"\A﻿?---[ \t]*\r?\n(.*?)\r?\n---[ \t]*(?:\r?\n|\Z)", re.DOTALL)

BACKUP_SUFFIX = ".bak"


class FrontmatterError(ValueError):
    """Raised when a SKILL.md has missing or unparseable frontmatter."""


def extract_block(text: str) -> str:
    """Return the raw YAML frontmatter block from a SKILL.md's text."""
    match = _FRONTMATTER_RE.match(text)
    if not match:
        raise FrontmatterError("no YAML frontmatter block (file must start with '---')")
    return match.group(1)


def parse(text: str) -> dict:
    """Parse a SKILL.md's full text into a frontmatter dict."""
    try:
        data = yaml.safe_load(extract_block(text))
    except yaml.YAMLError as exc:
        raise FrontmatterError(f"invalid YAML in frontmatter: {exc}") from exc

    if data is None:
        return {}
    if not isinstance(data, dict):
        raise FrontmatterError(f"frontmatter must be a mapping, got {type(data).__name__}")
    return data


def load_frontmatter(skill_md_path: str) -> dict:
    """Read one SKILL.md and return its frontmatter as a dict."""
    try:
        with open(skill_md_path, encoding="utf-8") as handle:
            text = handle.read()
    except UnicodeDecodeError as exc:
        raise FrontmatterError(f"not valid UTF-8: {exc}") from exc
    return parse(text)


def description_of(frontmatter: dict) -> str:
    """Return the description as a single normalized line.

    YAML preserves the newlines of a literal ('|') scalar and folds a folded
    ('>') one; collapsing whitespace here gives every style the same shape,
    which is what index.json and the linters want to compare.
    """
    return " ".join(str(frontmatter.get("description", "")).split())


def iter_skill_dirs(skills_dir: str = "skills") -> Iterator[Tuple[str, str]]:
    """Yield (slug, skill_dir) for every real skill, in sorted order.

    Skips '*.bak' backup directories and any directory lacking a SKILL.md.
    """
    for slug in sorted(os.listdir(skills_dir)):
        if slug.endswith(BACKUP_SUFFIX):
            continue
        skill_dir = os.path.join(skills_dir, slug)
        if not os.path.isdir(skill_dir):
            continue
        if not os.path.isfile(os.path.join(skill_dir, "SKILL.md")):
            continue
        yield slug, skill_dir


def load_all(skills_dir: str = "skills") -> Tuple[Dict[str, dict], Dict[str, str]]:
    """Load frontmatter for every skill.

    Returns (frontmatter_by_slug, errors_by_slug). Callers decide whether a
    parse failure is fatal; nothing is silently dropped.
    """
    loaded: Dict[str, dict] = {}
    errors: Dict[str, str] = {}

    for slug, skill_dir in iter_skill_dirs(skills_dir):
        try:
            loaded[slug] = load_frontmatter(os.path.join(skill_dir, "SKILL.md"))
        except FrontmatterError as exc:
            errors[slug] = str(exc)

    return loaded, errors

===== END FILE =====


===== FILE: ./tools/update-contributors.py =====
#!/usr/bin/env python3
"""Regenerate the contributor avatar wall in README.md from the GitHub API.

Writes between the markers:

    <!-- contributors:start -->
    ...generated...
    <!-- contributors:end -->

Avatars are served from github.com/<login>.png rather than a third-party
contributor-image service. That is deliberate: a README image is fetched on
every page view, so an external host would be an uncontrolled dependency in the
most-viewed file in the repository. GitHub's own avatar endpoint has neither
problem, and GitHub proxies it through camo like any other image.

Usage:
    python tools/update-contributors.py            # rewrite README.md
    python tools/update-contributors.py --check    # exit 1 if out of date
"""
from __future__ import annotations

import argparse
import json
import os
import sys
import urllib.request

REPO = os.environ.get("GITHUB_REPOSITORY", "mukul975/Anthropic-Cybersecurity-Skills")
MAINTAINER = REPO.split("/")[0]
README = "README.md"

START = "<!-- contributors:start -->"
END = "<!-- contributors:end -->"

# Accounts that are bots or automation, excluded from the wall.
EXCLUDE_SUFFIXES = ("[bot]",)
EXCLUDE_LOGINS = {"github-actions", "dependabot", "pull"}

AVATAR_PX = 72


def _get(path: str):
    req = urllib.request.Request(f"https://api.github.com/repos/{REPO}/{path}", headers={
        "Accept": "application/vnd.github+json",
        "User-Agent": "update-contributors",
    })
    token = os.environ.get("GITHUB_TOKEN")
    if token:
        req.add_header("Authorization", f"Bearer {token}")
    with urllib.request.urlopen(req, timeout=30) as resp:
        return json.load(resp)


def _keep(login: str) -> bool:
    return bool(login) and not login.endswith(EXCLUDE_SUFFIXES) and login not in EXCLUDE_LOGINS


def fetch_contributors() -> list[dict]:
    """Every non-bot contributor, most contributions first.

    Two endpoints, because one of them lies. /contributors carries the
    authoritative contribution counts but is heavily cached — a merge can take
    up to a day to show up there. /commits is live. So anyone whose commit is
    already linked to their account but has not yet surfaced in /contributors
    gets picked up from the commit list instead of waiting a day to be thanked.

    Commits authored with an unlinkable email (a machine hostname such as
    user@HOST.localdomain) have no `author` object and are skipped by both
    paths. They never appear in GitHub's own contributor graph either, so the
    wall matches what GitHub itself shows.
    """
    people: list[dict] = []
    page = 1
    while True:
        batch = _get(f"contributors?per_page=100&page={page}")
        if not batch:
            break
        people.extend(batch)
        if len(batch) < 100:
            break
        page += 1

    known = {p.get("login") for p in people}
    ranked = [p for p in people if p.get("type") != "Bot" and _keep(p.get("login", ""))]

    # Catch anyone the cached endpoint has not caught up with yet.
    recent: dict[str, int] = {}
    for page in (1, 2):
        for commit in _get(f"commits?per_page=100&page={page}") or []:
            author = commit.get("author")
            if not author or author.get("type") == "Bot":
                continue
            login = author.get("login", "")
            if login in known or not _keep(login):
                continue
            recent[login] = recent.get(login, 0) + 1

    for login, count in sorted(recent.items(), key=lambda kv: (-kv[1], kv[0])):
        ranked.append({"login": login, "contributions": count})

    return ranked


def render(people: list[dict]) -> str:
    lines = ['<p align="center">']
    for person in people:
        login = person["login"]
        count = person.get("contributions", 0)
        plural = "" if count == 1 else "s"
        title = f"{login} — maintainer" if login == MAINTAINER else f"{login} — {count} contribution{plural}"
        lines.append(
            f'<a href="https://github.com/{login}" title="{title}">'
            f'<img src="https://github.com/{login}.png?size=100" '
            f'width="{AVATAR_PX}" height="{AVATAR_PX}" alt="@{login}"></a>'
        )
    lines.append("</p>")
    lines.append("")
    lines.append(
        f'<p align="center"><sub>{len(people)} contributors, ordered by contribution count · '
        f'see the full <a href="https://github.com/{REPO}/graphs/contributors">contributor graph</a>'
        "</sub></p>"
    )
    return "\n".join(lines)


def splice(readme: str, block: str) -> str:
    if START not in readme or END not in readme:
        raise SystemExit(
            f"ERROR: {README} is missing the {START} / {END} markers. "
            "Add them around the contributor wall."
        )
    head, rest = readme.split(START, 1)
    _, tail = rest.split(END, 1)
    return f"{head}{START}\n{block}\n{END}{tail}"


def main() -> int:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("--check", action="store_true",
                        help="exit 1 if README.md is out of date; do not write")
    args = parser.parse_args()

    if not os.path.isfile(README):
        print(f"ERROR: {README} not found. Run from the repository root.")
        return 1

    people = fetch_contributors()
    if not people:
        print("ERROR: the API returned no contributors; refusing to blank the section.")
        return 1

    current = open(README, encoding="utf-8").read()
    updated = splice(current, render(people))

    if updated == current:
        print(f"OK: contributor wall is current ({len(people)} contributors)")
        return 0

    if args.check:
        print(f"ERROR: contributor wall is out of date ({len(people)} contributors). "
              "Run: python tools/update-contributors.py")
        return 1

    with open(README, "w", encoding="utf-8", newline="") as handle:
        handle.write(updated)
    print(f"Updated contributor wall: {len(people)} contributors")
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: ./tools/validate-agentskills.py =====
#!/usr/bin/env python3
"""Validate SKILL.md frontmatter against the strict agentskills.io standard.

Reports, per skill, any deviation from tools/agentskills-skill.schema.json plus
the two constraints JSON Schema can't express (name == parent dir; no angle
brackets in frontmatter). READ-ONLY; never edits files.

Usage:
  python3 tools/validate-agentskills.py            # summary + report
  python3 tools/validate-agentskills.py --json      # machine-readable JSON
  python3 tools/validate-agentskills.py --strict    # exit 1 if any non-compliant
"""
import os, re, sys, json, glob
from collections import Counter

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))

from skill_frontmatter import description_of, load_frontmatter, FrontmatterError

REPO = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
ALLOWED = {"name", "description", "license", "compatibility", "metadata", "allowed-tools"}
NAME_RE = re.compile(r"^[a-z0-9]+(-[a-z0-9]+)*$")

# The agentskills.io standard forbids reserved vendor words in a skill name.
# tools/agentskills-skill.schema.json names this script as the enforcement
# point, but the check was never actually implemented until now.
RESERVED_NAME_WORDS = ("anthropic", "claude")


def validate(path):
    slug = os.path.basename(os.path.dirname(path))
    problems = []

    try:
        fm = load_frontmatter(path)
    except FrontmatterError as exc:
        return slug, [str(exc)], []

    keys = list(fm.keys())

    if "name" not in keys:
        problems.append("missing required key: name")
    if "description" not in keys:
        problems.append("missing required key: description")

    name = str(fm.get("name", "") or "")
    if name:
        if not NAME_RE.match(name):
            problems.append(f"name not lowercase-kebab-case: {name!r}")
        if not (1 <= len(name) <= 64):
            problems.append(f"name length {len(name)} out of 1..64")
        if name != slug:
            problems.append(f"name {name!r} != directory {slug!r}")
        for reserved in RESERVED_NAME_WORDS:
            if reserved in name.lower():
                problems.append(f"name contains reserved word {reserved!r}: {name!r}")

    desc = description_of(fm)
    if desc:
        if not (1 <= len(desc) <= 1024):
            problems.append(f"description length {len(desc)} out of 1..1024")
    elif "description" in keys:
        problems.append("description empty")

    # Angle brackets are an injection risk. Checking the PARSED values (rather
    # than the raw text) means YAML block-scalar indicators like `>-` are never
    # mistaken for content, so no indicator-stripping hack is needed.
    for key, value in fm.items():
        if isinstance(value, str) and ("<" in value or ">" in value):
            problems.append(f"frontmatter value for {key!r} contains angle brackets "
                            "(injection risk / not allowed)")
            break

    # Additional top-level keys are PERMITTED by the standard (name+description
    # are the only required fields). They are reported for information, not
    # counted as compliance failures.
    nonstd = [k for k in keys if k not in ALLOWED]
    return slug, problems, nonstd

def main():
    as_json = "--json" in sys.argv
    strict = "--strict" in sys.argv
    skills = sorted(glob.glob(os.path.join(REPO, "skills", "*", "SKILL.md")))
    results = []
    nonstd_hist = Counter()
    compliant = 0
    for p in skills:
        slug, problems, nonstd = validate(p)
        nonstd_hist.update(nonstd)
        if not problems:
            compliant += 1
        results.append({"skill": slug, "compliant": not problems, "problems": problems})
    noncompliant = [r for r in results if not r["compliant"]]
    summary = {
        "total": len(skills),
        "compliant": compliant,
        "noncompliant": len(noncompliant),
        "nonstandard_key_frequency": dict(nonstd_hist.most_common()),
    }
    if as_json:
        print(json.dumps({"summary": summary, "results": results}, indent=1))
    else:
        print(f"agentskills.io compliance: {compliant}/{len(skills)} compliant, "
              f"{len(noncompliant)} non-compliant")
        print("\nNon-standard top-level keys (count of skills carrying each):")
        for k, n in nonstd_hist.most_common():
            print(f"  {k:20s} {n}")
        # distinct problem types (excluding the per-key nonstd noise)
        other = Counter()
        for r in noncompliant:
            for pr in r["problems"]:
                if not pr.startswith("non-standard top-level key:"):
                    other[re.sub(r':.*$', '', pr)] += 1
        if other:
            print("\nOther (non-key) issues:")
            for k, n in other.most_common():
                print(f"  {k}: {n}")
    if strict and noncompliant:
        sys.exit(1)

if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: ./tools/validate-skill.py =====
#!/usr/bin/env python3
"""Validate SKILL.md metadata for the Anthropic-Cybersecurity-Skills repository.

Usage:
    python tools/validate-skill.py skills/my-skill/
    python tools/validate-skill.py --all
"""
import os
import re
import sys
import glob

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))

from skill_frontmatter import load_frontmatter, FrontmatterError

# Kept in sync with the CI workflow (.github/workflows/validate-skills.yml),
# which now delegates to this script so there is a single source of truth.
REQUIRED_FIELDS = ["name", "description", "domain", "subdomain", "tags",
                   "version", "author", "license"]

# Canonical subdomain → set of accepted aliases (including canonical itself).
# When a skill uses an alias, the validator accepts it but the canonical form
# is the first entry in each group below.  New skills should use the canonical.
_SUBDOMAIN_ALIASES = {
    # identity
    "identity-access-management": {"identity-access-management", "identity-and-access-management", "identity-security"},
    # zero-trust
    "zero-trust-architecture": {"zero-trust-architecture", "zero-trust"},
    # OT/ICS
    "ot-ics-security": {"ot-ics-security", "ot-security"},
    # SOC / security ops
    "soc-operations": {"soc-operations", "security-operations"},
    # red team
    "red-teaming": {"red-teaming", "red-team"},
    # standalone (no aliases)
    "web-application-security": {"web-application-security", "application-security"},
    "network-security": {"network-security"},
    "penetration-testing": {"penetration-testing", "offensive-security"},
    "digital-forensics": {"digital-forensics"},
    "malware-analysis": {"malware-analysis"},
    "threat-intelligence": {"threat-intelligence"},
    "cloud-security": {"cloud-security"},
    "container-security": {"container-security"},
    "cryptography": {"cryptography"},
    "vulnerability-management": {"vulnerability-management"},
    "compliance-governance": {"compliance-governance", "governance-risk-compliance"},
    "devsecops": {"devsecops"},
    "threat-hunting": {"threat-hunting"},
    "incident-response": {"incident-response"},
    "endpoint-security": {"endpoint-security"},
    "phishing-defense": {"phishing-defense", "social-engineering-defense"},
    "api-security": {"api-security"},
    "mobile-security": {"mobile-security"},
    "ransomware-defense": {"ransomware-defense"},
    "threat-detection": {"threat-detection"},
    "blockchain-security": {"blockchain-security"},
    "data-protection": {"data-protection"},
    "deception-technology": {"deception-technology"},
    "hardware-firmware-security": {"hardware-firmware-security", "firmware-analysis", "firmware-security"},
    "privacy-compliance": {"privacy-compliance"},
    "purple-team": {"purple-team"},
    "supply-chain-security": {"supply-chain-security"},
    "wireless-security": {"wireless-security"},
    "ai-security": {"ai-security"},
}

# Flat set of all accepted subdomain values (canonical + aliases).
ALLOWED_SUBDOMAINS: set = {v for group in _SUBDOMAIN_ALIASES.values() for v in group}

# Reverse map: alias → canonical (for warning messages).
_ALIAS_TO_CANONICAL: dict = {}
for canonical, aliases in _SUBDOMAIN_ALIASES.items():
    for alias in aliases:
        _ALIAS_TO_CANONICAL[alias] = canonical

KEBAB_RE = re.compile(r"^[a-z0-9]+(-[a-z0-9]+)*$")

# Minimum description length.  Other repo tooling uses 50 chars; align here.
DESCRIPTION_MIN_CHARS = 50

RED = "\033[91m"
GREEN = "\033[92m"
YELLOW = "\033[93m"
RESET = "\033[0m"


def validate_skill(skill_dir):
    """Validate a single skill directory. Returns list of error strings."""
    errors = []
    skill_md = os.path.join(skill_dir, "SKILL.md")

    if not os.path.isfile(skill_md):
        return [f"SKILL.md not found in {skill_dir}"]

    try:
        fm = load_frontmatter(skill_md)
    except FrontmatterError as e:
        return [str(e)]
    except IOError as e:
        return [f"Could not read SKILL.md: {e}"]

    # Check required fields.
    for field in REQUIRED_FIELDS:
        if field not in fm:
            errors.append(f"Missing required field: {field}")

    # Validate name.
    name = fm.get("name", "")
    if name:
        if not KEBAB_RE.match(name):
            errors.append(
                f"Name '{name}' is not valid kebab-case (lowercase letters, digits, hyphens only)"
            )
        if len(name) > 64:
            errors.append(f"Name too long ({len(name)} chars, max 64)")

    # Validate description.
    desc = fm.get("description", "")
    if isinstance(desc, list):
        errors.append("Description must be a string value, not a list")
    elif isinstance(desc, str):
        if len(desc) < DESCRIPTION_MIN_CHARS:
            errors.append(
                f"Description too short ({len(desc)} chars, min {DESCRIPTION_MIN_CHARS})"
            )
        # No hard upper-limit enforced; multi-line folded scalars (>-) produce
        # long strings that are valid and common in this repo.

    # Validate domain.
    domain = fm.get("domain", "")
    if domain and domain != "cybersecurity":
        errors.append(f"Domain must be 'cybersecurity', got '{domain}'")

    # Validate subdomain.
    subdomain = fm.get("subdomain", "")
    if subdomain:
        if subdomain not in ALLOWED_SUBDOMAINS:
            errors.append(
                f"Unknown subdomain '{subdomain}'. Allowed: {', '.join(sorted(ALLOWED_SUBDOMAINS))}"
            )
        else:
            canonical = _ALIAS_TO_CANONICAL.get(subdomain, subdomain)
            if subdomain != canonical:
                # Warn (non-blocking) — alias is accepted but canonical is preferred
                print(
                    f"{YELLOW}WARN{RESET} subdomain '{subdomain}' is an alias;"
                    f" canonical form is '{canonical}'"
                )

    # Validate tags.
    tags = fm.get("tags", [])
    if isinstance(tags, str):
        tags = [tags]
    if len(tags) < 2:
        errors.append(f"Need at least 2 tags, got {len(tags)}")

    return errors


def main():
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <skill-dir> | --all")
        sys.exit(1)

    if sys.argv[1] == "--all":
        # Skip .bak backup directories — they are stale copies without a SKILL.md.
        # glob may return OS-native separators, so normalize before checking.
        skill_dirs = sorted(
            d for d in glob.glob("skills/*/")
            if not d.rstrip("/\\").endswith(".bak")
        )
        if not skill_dirs:
            print("ERROR: No skill directories found. Run from the repository root.")
            sys.exit(1)
    else:
        skill_dirs = [sys.argv[1].rstrip("/") + "/"]

    total = 0
    passed = 0
    failed = 0

    for skill_dir in skill_dirs:
        if not os.path.isdir(skill_dir.rstrip("/")):
            print(f"{RED}SKIP{RESET} {skill_dir} — not a directory")
            continue

        total += 1
        errors = validate_skill(skill_dir.rstrip("/"))

        name = os.path.basename(skill_dir.rstrip("/"))
        if errors:
            failed += 1
            print(f"{RED}FAIL{RESET} {name}")
            for e in errors:
                print(f"      {YELLOW}→ {e}{RESET}")
        else:
            passed += 1
            print(f"{GREEN}PASS{RESET} {name}")

    print(f"\n{'='*50}")
    print(f"Total: {total}  {GREEN}Passed: {passed}{RESET}  {RED}Failed: {failed}{RESET}")

    sys.exit(0 if failed == 0 else 1)


if __name__ == "__main__":
    main()

===== END FILE =====
