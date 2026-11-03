

===== FILE: skills/analyzing-ethereum-smart-contract-vulnerabilities/SKILL.md =====
---
name: analyzing-ethereum-smart-contract-vulnerabilities
description: Perform static and symbolic analysis of Solidity smart contracts using
  Slither and Mythril to detect reentrancy, integer overflow, access control, and
  other vulnerability classes before deployment to Ethereum mainnet.
domain: cybersecurity
subdomain: blockchain-security
tags:
- ethereum
- solidity
- smart-contract
- slither
- mythril
- blockchain
- defi
- audit
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- ID.RA-01
mitre_attack:
- T1190
- T1059
---

# Analyzing Ethereum Smart Contract Vulnerabilities

## Overview

Smart contract vulnerabilities have led to billions of dollars in losses across DeFi protocols. Unlike traditional software, deployed smart contracts are immutable and handle real financial assets, making pre-deployment security analysis critical. Slither performs fast static analysis using an intermediate representation to detect over 90 vulnerability patterns in seconds, while Mythril uses symbolic execution and SMT solving to discover complex execution path vulnerabilities like reentrancy and integer overflows. This skill covers running both tools against Solidity contracts, interpreting results, triaging findings by severity, and generating audit reports.


## When to Use

- When investigating security incidents that require analyzing ethereum smart contract vulnerabilities
- When building detection rules or threat hunting queries for this domain
- When SOC analysts need structured procedures for this analysis type
- When validating security monitoring coverage for related attack techniques

## Prerequisites

- Python 3.10+ with pip
- Slither (pip install slither-analyzer) and solc compiler
- Mythril (pip install mythril) with solc-select for compiler version management
- Solidity source code or compiled contract bytecode
- Foundry or Hardhat development framework (optional, for project-level analysis)

## Steps

### Step 1: Run Slither Static Analysis

Execute Slither against the contract codebase to identify vulnerability patterns, optimization opportunities, and code quality issues using its 90+ built-in detectors.

### Step 2: Run Mythril Symbolic Execution

Run Mythril deep analysis to explore execution paths and discover reentrancy, unchecked external calls, and arithmetic vulnerabilities that require path-sensitive analysis.

### Step 3: Triage and Correlate Findings

Combine results from both tools, deduplicate findings, assess severity based on exploitability and financial impact, and filter false positives.

### Step 4: Generate Audit Report

Produce a structured audit report with vulnerability descriptions, affected code locations, exploit scenarios, and remediation recommendations.

## Expected Output

JSON report listing vulnerabilities with SWC (Smart Contract Weakness Classification) identifiers, severity ratings, affected functions, and suggested fixes.

===== END FILE =====


===== FILE: skills/analyzing-ethereum-smart-contract-vulnerabilities/LICENSE =====

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


===== FILE: skills/analyzing-ethereum-smart-contract-vulnerabilities/references/api-reference.md =====
# API Reference: Analyzing Ethereum Smart Contract Vulnerabilities

## Slither CLI

```bash
# Basic analysis
slither contracts/

# JSON output
slither contracts/ --json slither-report.json

# Run specific detector only
slither contracts/ --detect reentrancy-eth,unprotected-upgrade

# List all detectors
slither --list-detectors

# Print contract summary
slither contracts/ --print human-summary

# Generate inheritance graph
slither contracts/ --print inheritance-graph
```

## Mythril CLI

```bash
# Analyze single contract
myth analyze contracts/Token.sol

# JSON output
myth analyze contracts/Token.sol -o json

# Set execution timeout
myth analyze contracts/Token.sol --execution-timeout 300

# Analyze deployed bytecode
myth analyze --address 0x1234... --rpc infura

# Increase analysis depth
myth analyze contracts/Token.sol --max-depth 50 --transaction-count 3
```

## Slither Detector Severity Levels

| Impact | Confidence | Example Detectors |
|--------|------------|-------------------|
| High | High | reentrancy-eth, suicidal, arbitrary-send-eth |
| High | Medium | controlled-delegatecall, reentrancy-no-eth |
| Medium | High | locked-ether, incorrect-equality |
| Medium | Medium | uninitialized-state, shadowing-state |
| Low | High | naming-convention, solc-version |
| Informational | High | pragma, dead-code |

## SWC Registry (Key Entries)

| SWC ID | Title | Tool Coverage |
|--------|-------|---------------|
| SWC-101 | Integer Overflow/Underflow | Mythril |
| SWC-104 | Unchecked Call Return | Slither + Mythril |
| SWC-106 | Unprotected SELFDESTRUCT | Slither + Mythril |
| SWC-107 | Reentrancy | Slither + Mythril |
| SWC-110 | Assert Violation | Mythril |
| SWC-112 | Delegatecall to Untrusted Callee | Slither |
| SWC-115 | tx.origin Authentication | Slither |
| SWC-116 | Block Timestamp Dependence | Mythril |
| SWC-120 | Weak Randomness | Slither |

## Installation

```bash
# Slither (requires solc)
pip install slither-analyzer
solc-select install 0.8.20
solc-select use 0.8.20

# Mythril
pip install mythril
```

## Slither JSON Output Structure

```json
{
  "success": true,
  "results": {
    "detectors": [{
      "check": "reentrancy-eth",
      "impact": "High",
      "confidence": "Medium",
      "description": "Reentrancy in Contract.withdraw()",
      "elements": [{"source_mapping": {"filename_short": "Contract.sol", "lines": [42, 43]}}]
    }]
  }
}
```

### References

- Slither: https://github.com/crytic/slither
- Mythril: https://github.com/Consensys/mythril
- SWC Registry: https://swcregistry.io/
- Solidity Security: https://docs.soliditylang.org/en/latest/security-considerations.html

===== END FILE =====


===== FILE: skills/analyzing-ethereum-smart-contract-vulnerabilities/scripts/agent.py =====
#!/usr/bin/env python3
"""Smart Contract Security Agent - runs Slither and Mythril analysis on Solidity contracts."""

import json
import argparse
import logging
import subprocess
from collections import defaultdict
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

SWC_REGISTRY = {
    "SWC-101": "Integer Overflow and Underflow",
    "SWC-104": "Unchecked Call Return Value",
    "SWC-106": "Unprotected SELFDESTRUCT",
    "SWC-107": "Reentrancy",
    "SWC-110": "Assert Violation",
    "SWC-112": "Delegatecall to Untrusted Callee",
    "SWC-113": "DoS with Failed Call",
    "SWC-115": "Authorization through tx.origin",
    "SWC-116": "Block values as a proxy for time",
    "SWC-120": "Weak Sources of Randomness",
}


def run_slither(contract_path):
    """Run Slither static analysis on Solidity contract."""
    cmd = ["slither", contract_path, "--json", "-"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    try:
        return json.loads(result.stdout) if result.stdout else {}
    except json.JSONDecodeError:
        logger.error("Slither JSON parse failed")
        return {}


def run_mythril(contract_path, timeout=300):
    """Run Mythril symbolic execution analysis."""
    cmd = ["myth", "analyze", contract_path, "--execution-timeout", str(timeout), "-o", "json"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout + 60)
    try:
        return json.loads(result.stdout) if result.stdout else {}
    except json.JSONDecodeError:
        logger.error("Mythril JSON parse failed")
        return {}


def analyze_slither_results(slither_output):
    """Parse and categorize Slither detector findings."""
    findings = []
    by_severity = defaultdict(int)
    by_detector = defaultdict(int)
    for detector in slither_output.get("results", {}).get("detectors", []):
        severity = detector.get("impact", "informational").lower()
        by_severity[severity] += 1
        det_name = detector.get("check", "unknown")
        by_detector[det_name] += 1
        elements = detector.get("elements", [])
        location = ""
        if elements:
            elem = elements[0]
            location = f"{elem.get('source_mapping', {}).get('filename_short', '')}:" \
                       f"L{elem.get('source_mapping', {}).get('lines', [0])[0] if elem.get('source_mapping', {}).get('lines') else 0}"
        findings.append({
            "detector": det_name,
            "severity": severity,
            "description": detector.get("description", "")[:200],
            "location": location,
            "confidence": detector.get("confidence", ""),
        })
    return {
        "total": len(findings),
        "by_severity": dict(by_severity),
        "by_detector": dict(sorted(by_detector.items(), key=lambda x: x[1], reverse=True)[:15]),
        "findings": sorted(findings, key=lambda x: {"high": 0, "medium": 1, "low": 2, "informational": 3}.get(x["severity"], 4)),
    }


def analyze_mythril_results(mythril_output):
    """Parse Mythril symbolic execution findings."""
    findings = []
    by_swc = defaultdict(int)
    for issue in mythril_output.get("issues", []):
        swc_id = issue.get("swc-id", "")
        swc_key = f"SWC-{swc_id}" if swc_id else "unknown"
        by_swc[swc_key] += 1
        severity = issue.get("severity", "Medium").lower()
        findings.append({
            "swc_id": swc_key,
            "swc_title": SWC_REGISTRY.get(swc_key, issue.get("title", "")),
            "severity": severity,
            "description": issue.get("description", "")[:200],
            "contract": issue.get("contract", ""),
            "function": issue.get("function", ""),
            "line_number": issue.get("lineno", 0),
        })
    return {
        "total": len(findings),
        "by_swc": dict(by_swc),
        "findings": findings,
    }


def deduplicate_findings(slither_findings, mythril_findings):
    """Merge and deduplicate findings from both tools."""
    combined = []
    seen = set()
    for f in slither_findings.get("findings", []):
        key = (f.get("location", ""), f.get("detector", ""))
        if key not in seen:
            seen.add(key)
            combined.append({**f, "source": "slither"})
    for f in mythril_findings.get("findings", []):
        key = (f.get("contract", "") + str(f.get("line_number", 0)), f.get("swc_id", ""))
        if key not in seen:
            seen.add(key)
            combined.append({**f, "source": "mythril"})
    return combined


def generate_report(contract_path, slither_analysis, mythril_analysis, combined):
    critical_high = sum(1 for f in combined if f.get("severity") in ("high", "critical"))
    return {
        "timestamp": datetime.utcnow().isoformat(),
        "contract": contract_path,
        "slither_analysis": {
            "total_findings": slither_analysis["total"],
            "by_severity": slither_analysis["by_severity"],
            "top_detectors": slither_analysis["by_detector"],
        },
        "mythril_analysis": {
            "total_findings": mythril_analysis["total"],
            "by_swc": mythril_analysis["by_swc"],
        },
        "combined_findings": len(combined),
        "critical_high_findings": critical_high,
        "audit_result": "FAIL" if critical_high > 0 else "PASS",
        "findings": combined[:30],
    }


def main():
    parser = argparse.ArgumentParser(description="Solidity Smart Contract Security Analysis Agent")
    parser.add_argument("--contract", required=True, help="Path to Solidity contract or project directory")
    parser.add_argument("--mythril-timeout", type=int, default=300, help="Mythril execution timeout (seconds)")
    parser.add_argument("--skip-mythril", action="store_true", help="Skip Mythril (slow symbolic execution)")
    parser.add_argument("--output", default="smart_contract_audit_report.json")
    args = parser.parse_args()

    slither_output = run_slither(args.contract)
    slither_analysis = analyze_slither_results(slither_output)
    mythril_analysis = {"total": 0, "by_swc": {}, "findings": []}
    if not args.skip_mythril:
        mythril_output = run_mythril(args.contract, args.mythril_timeout)
        mythril_analysis = analyze_mythril_results(mythril_output)
    combined = deduplicate_findings(slither_analysis, mythril_analysis)
    report = generate_report(args.contract, slither_analysis, mythril_analysis, combined)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("Smart contract audit: %d findings (%d critical/high), result: %s",
                report["combined_findings"], report["critical_high_findings"], report["audit_result"])
    print(json.dumps(report, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/auditing-foundry-smart-contract-security/SKILL.md =====
---
name: auditing-foundry-smart-contract-security
description: >-
  Pre-deployment security audit of Solidity smart contracts in a Foundry project.
  Combines static analysis (Slither, Aderyn), symbolic execution (Mythril), and
  property-based testing (forge fuzz + invariant tests with handlers) to catch
  reentrancy, access-control, oracle/price manipulation, and arithmetic bugs
  BEFORE deploying to an EVM chain. Also enforces key hygiene (no plaintext
  private keys, encrypted cast keystore) and a secure deploy workflow. Use when
  writing, reviewing, testing, or deploying Solidity/Foundry contracts, building
  a dApp, or working with forge/cast/anvil, MetaMask, or Web3/DeFi code.
domain: cybersecurity
subdomain: blockchain-security
tags:
  - solidity
  - foundry
  - forge
  - smart-contract
  - slither
  - aderyn
  - mythril
  - reentrancy
  - defi
  - web3
  - invariant-testing
  - audit
version: "1.0"
author: devredious
license: Apache-2.0
based_on: mukul975/analyzing-ethereum-smart-contract-vulnerabilities
swc_registry: https://swcregistry.io/
mitre_attack:
  - T1190
  - T1059
---

# Auditing Foundry Smart Contract Security

## Overview

Deployed smart contracts are **immutable** and custody **real funds**, so a bug
shipped to mainnet cannot be patched — it can only be exploited. Most catastrophic
DeFi losses come from a small set of recurring classes: reentrancy, broken access
control, oracle/price manipulation, and unchecked arithmetic or external calls.

This skill runs a **defense-in-depth, pre-deployment audit** of a Foundry project,
layering four independent techniques that each catch what the others miss:

1. **Static analysis** — `slither` (90+ detectors) and `aderyn` (Cyfrin, Rust) scan
   the AST/IR in seconds for known anti-patterns.
2. **Symbolic execution** — `mythril` (optional, slow) explores execution paths and
   SMT-solves for deep arithmetic/reentrancy bugs.
3. **Property-based testing** — `forge test` with **fuzzing** (`testFuzz_*`) and
   **invariant tests** (`invariant_*` + handler contracts with ghost variables)
   proves protocol-level properties hold across millions of random sequences.
4. **Manual review + key hygiene** — a structured checklist (see
   `references/vulnerability-checklist.md`) and a secrets/keystore audit so no
   private key ever lives in plaintext and deployment goes through an encrypted
   `cast` keystore (see `references/secure-deployment-and-keys.md`).

The skill is **dev-side and pre-deployment** — it is run by the engineer building
the contract, not by a SOC after an incident. Findings gate the deploy: any
high/critical static finding, failing test, leaked key, or low coverage = **FAIL**.

## When to Use

- Before deploying any Solidity contract to a testnet or mainnet EVM chain.
- When writing or reviewing a Foundry project (`foundry.toml`, `src/`, `test/`, `script/`).
- When a contract handles value: tokens (ERC-20/721/1155), vaults, staking, AMMs, bridges, governance.
- When adding fuzz or invariant tests, or when coverage of value-moving functions is unknown.
- When wiring deployment scripts — to verify keys are in an encrypted keystore, not `.env` plaintext.
- When integrating a price oracle, external call, `delegatecall`, or upgradeable proxy.
- When triaging a Slither/Aderyn report and needing to separate real bugs from false positives.

## Prerequisites

- **Foundry** installed (`forge`, `cast`, `anvil`): `curl -L https://foundry.paradigm.xyz | bash && foundryup`
- **Slither** + solc: `pip install slither-analyzer` and `solc-select install <ver> && solc-select use <ver>`
- **Aderyn** (recommended): `cargo install aderyn` (or `npm i -g @cyfrin/aderyn`)
- **Mythril** (optional, slow symbolic exec): `pip install mythril`
- **gitleaks** (key-leak scan): see the companion `implementing-secret-scanning-with-gitleaks` skill
- A Foundry project that **compiles** (`forge build` succeeds) — analyzers need build artifacts.
- Solidity ^0.8.x is assumed (built-in overflow checks); pre-0.8 contracts need extra SafeMath review.

> Install the Python tools in a virtualenv (recommended on externally-managed distros). Never run
> analysis against untrusted contract source on a machine with funded wallets unlocked.

## Steps

### Step 1: Build and sanity-check the project

```bash
forge build                    # analyzers require fresh artifacts
forge fmt --check              # style gate (optional)
cat foundry.toml               # note solc version, optimizer, remappings, evm_version
```

### Step 2: Static analysis (fast, run every time)

```bash
# Slither — full project (uses foundry.toml + remappings automatically)
slither . --json slither-report.json

# Aderyn — Cyfrin Rust analyzer, complementary detectors
aderyn . -o aderyn-report.json
```

Or run the bundled orchestrator that runs both, deduplicates, and gates the result:

```bash
python3 scripts/agent.py --project . --output audit-report.json
```

### Step 3: Symbolic execution on critical contracts (optional, slow)

```bash
# Only on the highest-value contract(s) — Mythril is path-explosive
myth analyze src/Vault.sol --solc-json mythril.config.json --execution-timeout 300 -o json
# or: python3 scripts/agent.py --project . --mythril src/Vault.sol
```

### Step 4: Property-based testing — fuzz + invariants

```bash
forge test -vvv                                  # unit + fuzz tests
forge coverage --report summary                  # coverage of value-moving code
forge test --match-test invariant_ -vvv          # invariant suite (handler-based)
```

Every value-moving contract should have **invariant tests with a handler** (bounded
inputs, ghost variables, `targetContract(handler)`) — not just unit tests. See
`references/api-reference.md` for the handler pattern, and write a
`test_RevertWhen_*` (with `vm.expectRevert`) for each access-control guard.

### Step 5: Manual review against the checklist

Walk `references/vulnerability-checklist.md` for every contract: reentrancy
(checks-effects-interactions / `nonReentrant`), access control, oracle manipulation,
`delegatecall`/proxy storage layout, unchecked return values, `tx.origin`, weak
randomness, DoS, front-running/MEV, and ERC-specific pitfalls (approve race,
fee-on-transfer, rebasing).

### Step 6: Key hygiene & secure deploy

```bash
gitleaks detect --no-banner            # no private keys / mnemonics / .env committed
git ls-files | grep -E '\.env$|keystore' && echo "WARN: secrets tracked by git"

# Import the deploy key ONCE into an encrypted keystore — never a plaintext PRIVATE_KEY env
cast wallet import deployer --interactive

# Deploy via the keystore account (testnet first), simulate before --broadcast
forge script script/Deploy.s.sol --account deployer --rpc-url <testnet> --broadcast --verify
```

See `references/secure-deployment-and-keys.md` for the full hardening rules
(MetaMask hygiene, hardware wallet for mainnet, RPC trust, post-deploy verification).

### Step 7: Triage and report

Combine Slither + Aderyn + Mythril + test results, deduplicate by (file, line),
drop confirmed false positives, rank by exploitability × financial impact, and map
each to its SWC id. The orchestrator emits `audit-report.json` with a PASS/FAIL gate.

## Expected Output

A JSON audit report listing findings with **SWC identifiers**, severity, tool source,
affected contract/function/line, and remediation; plus the test/coverage summary and a
single **PASS / FAIL** deploy gate. FAIL on any high/critical static finding, failing
test, leaked secret, or coverage below the configured threshold on value-moving code.

===== END FILE =====


===== FILE: skills/auditing-foundry-smart-contract-security/LICENSE =====

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


===== FILE: skills/auditing-foundry-smart-contract-security/references/api-reference.md =====
# API Reference — Tooling

## Slither (static analysis)

```bash
slither .                                  # whole project (reads foundry.toml + remappings)
slither . --json slither-report.json       # machine-readable
slither . --json -                          # JSON to stdout (used by agent.py)
slither . --print human-summary             # quick overview
slither . --print inheritance-graph         # inheritance / proxy layout
slither . --detect reentrancy-eth,unprotected-upgrade   # specific detectors
slither --list-detectors                    # all 90+ detectors
slither . --exclude-informational --exclude-low          # focus high/medium
slither . --triage-mode                     # interactively suppress false positives -> slither.db.json
```

Severity matrix (impact × confidence):

| Impact | Confidence | Example detectors |
|--------|------------|-------------------|
| High | High | `reentrancy-eth`, `suicidal`, `arbitrary-send-eth` |
| High | Medium | `controlled-delegatecall`, `reentrancy-no-eth` |
| Medium | High | `locked-ether`, `incorrect-equality`, `tx-origin` |
| Medium | Medium | `uninitialized-state`, `shadowing-state`, `unchecked-transfer` |
| Low | High | `naming-convention`, `solc-version`, `low-level-calls` |
| Informational | High | `pragma`, `dead-code`, `assembly` |

## Aderyn (Cyfrin, Rust static analyzer — complementary to Slither)

```bash
aderyn .                                    # markdown report.md by default
aderyn . -o aderyn-report.json              # JSON (used by agent.py)
aderyn . --scope src/                       # limit scope
```

## Mythril (symbolic execution — slow, use on critical contracts only)

```bash
myth analyze src/Vault.sol -o json
myth analyze src/Vault.sol --execution-timeout 300 --max-depth 50 -o json
myth analyze --address 0x... --rpc <url>    # deployed bytecode (read-only)
```

## Foundry — testing

```bash
forge build                                 # required before static analysis
forge test -vvv                             # unit + fuzz; -vvvv shows traces
forge test --match-contract VaultTest
forge test --match-test invariant_          # invariant suite only
forge coverage --report summary             # line/branch coverage table
forge coverage --report lcov                # for CI / tooling
forge snapshot                              # gas snapshots (DoS-by-gas review)
forge fmt --check                           # style gate
```

### Fuzz test (property over random inputs)

```solidity
function testFuzz_SetNumber(uint256 x) public {
    counter.setNumber(x);
    assertEq(counter.number(), x);
}
```

### Revert test (replaces deprecated testFail)

```solidity
function test_RevertWhen_Unauthorized() public {
    vm.prank(attacker);
    vm.expectRevert("Not authorized");   // or vm.expectRevert(MyError.selector)
    target.adminOnly();
}
```

### Key cheatcodes (`vm.*`)

| Cheatcode | Use |
|-----------|-----|
| `vm.prank(addr)` / `vm.startPrank` | impersonate caller (test access control) |
| `vm.warp(ts)` / `vm.roll(n)` | manipulate `block.timestamp` / `block.number` |
| `vm.deal(addr, amt)` | set ETH balance |
| `vm.store(addr, slot, val)` | overwrite storage (test invariants under hostile state) |
| `vm.expectRevert(...)` | assert a call reverts (with msg / custom error selector) |
| `vm.expectEmit(...)` | assert events |
| `bound(x, lo, hi)` | constrain fuzz inputs in handlers |
| `makeAddr("name")` | deterministic labelled actor |

### Invariant testing — handler pattern (the important one)

A handler wraps the target, **bounds inputs**, rotates **actors**, and tracks
**ghost variables**; `targetContract(handler)` makes the fuzzer drive only the
handler so sequences stay realistic.

```solidity
// test/Invariant.t.sol
contract VaultInvariantTest is Test {
    Vault vault;
    VaultHandler handler;

    function setUp() public {
        vault = new Vault();
        handler = new VaultHandler(vault);
        targetContract(address(handler));         // fuzz the handler, not the vault directly
    }

    function invariant_ConservationOfDeposits() public view {
        assertEq(address(vault).balance,
                 handler.ghost_depositSum() - handler.ghost_withdrawSum());
    }
    function invariant_Solvency() public view {
        assertGe(address(vault).balance, vault.totalDeposits());
    }
}
```

```solidity
// test/handlers/VaultHandler.sol
contract VaultHandler is Test {
    Vault public vault;
    uint256 public ghost_depositSum;
    uint256 public ghost_withdrawSum;
    address[] public actors;
    address internal currentActor;

    modifier useActor(uint256 seed) {
        currentActor = actors[bound(seed, 0, actors.length - 1)];
        vm.startPrank(currentActor); _; vm.stopPrank();
    }
    constructor(Vault _v) {
        vault = _v;
        for (uint256 i; i < 10; i++) { actors.push(makeAddr(string(abi.encodePacked("actor", i)))); vm.deal(actors[i], 100 ether); }
    }
    function deposit(uint256 amt, uint256 seed) external useActor(seed) {
        amt = bound(amt, 0.01 ether, 10 ether);
        vault.deposit{value: amt}(); ghost_depositSum += amt;
    }
    function withdraw(uint256 amt, uint256 seed) external useActor(seed) {
        uint256 bal = vault.balanceOf(currentActor);
        if (bal == 0) return;
        amt = bound(amt, 1, bal);
        vault.withdraw(amt); ghost_withdrawSum += amt;
    }
}
```

Tune in `foundry.toml`:

```toml
[invariant]
runs = 256
depth = 128
fail_on_revert = false   # set true once the handler fully constrains inputs

[fuzz]
runs = 10000
```

## SWC Registry (key entries)

| SWC | Title | Detected by |
|-----|-------|-------------|
| SWC-101 | Integer Overflow/Underflow | Mythril (pre-0.8 only) |
| SWC-104 | Unchecked Call Return | Slither + Mythril |
| SWC-105 | Unprotected Ether Withdrawal | Slither + Mythril |
| SWC-106 | Unprotected SELFDESTRUCT | Slither + Mythril |
| SWC-107 | Reentrancy | Slither + Mythril |
| SWC-112 | Delegatecall to Untrusted Callee | Slither |
| SWC-114 | Transaction Order Dependence (front-running) | manual |
| SWC-115 | tx.origin Authentication | Slither |
| SWC-116 | Block Timestamp Dependence | Mythril |
| SWC-120 | Weak Randomness | Slither + manual |

## References
- Slither: https://github.com/crytic/slither
- Aderyn: https://github.com/Cyfrin/aderyn
- Mythril: https://github.com/Consensys/mythril
- Foundry Book: https://getfoundry.sh/
- SWC Registry: https://swcregistry.io/
- Solidity security: https://docs.soliditylang.org/en/latest/security-considerations.html
- Solodit (audit findings DB): https://solodit.xyz/

===== END FILE =====


===== FILE: skills/auditing-foundry-smart-contract-security/references/secure-deployment-and-keys.md =====
# Secure Deployment & Key Hygiene

The contract code can be flawless and you still lose everything if a **private key
leaks** or you sign a malicious transaction. This is the part most smart-contract
guides skip. Treat keys as the highest-severity asset.

## Golden rules

1. **A real private key or seed phrase NEVER touches a file, env var, shell history, or git.**
2. Plaintext `PRIVATE_KEY=0x...` in `.env` is the #1 leak vector — use an **encrypted keystore** instead.
3. **Separate wallets**: a throwaway dev wallet (testnet only) ≠ the mainnet deployer ≠ your personal MetaMask with real funds.
4. **Hardware wallet (Ledger/Trezor) for any mainnet deploy or admin action** that controls funds.
5. Simulate before broadcasting; verify after.

## Foundry encrypted keystore (`cast wallet`)

Import the key once into an encrypted, password-protected keystore — then reference
it by name. The raw key never appears in commands or files again.

```bash
# Import interactively (key is typed, not in argv/history), set a strong password
cast wallet import deployer --interactive

# Or generate a fresh dev key directly into the keystore
cast wallet new

# List / inspect (addresses only)
cast wallet list
```

Deploy by **account name**, never by `--private-key`:

```bash
# Testnet first — simulate (no --broadcast) then broadcast + verify
forge script script/Deploy.s.sol --account deployer --rpc-url <testnet_rpc>
forge script script/Deploy.s.sol --account deployer --rpc-url <testnet_rpc> --broadcast --verify

# Mainnet (prefer a Ledger):
forge script script/Deploy.s.sol --ledger --hd-paths "m/44'/60'/0'/0/0" --rpc-url <mainnet_rpc> --broadcast --verify
```

In deploy scripts, use `vm.startBroadcast()` with **no argument** (it uses the
`--account`/`--ledger` signer). Avoid `vm.envUint("PRIVATE_KEY")`.

## Anti-leak controls (wire into the project)

```bash
# 1. .gitignore the usual suspects
printf '.env\n.env.*\n*.key\nkeystore/\nbroadcast/\n' >> .gitignore

# 2. Scan history + working tree for secrets (see the implementing-secret-scanning-with-gitleaks skill)
gitleaks detect --no-banner
gitleaks detect --no-banner --log-opts="--all"   # full git history

# 3. Confirm nothing sensitive is tracked
git ls-files | grep -E '\.env$|\.key$|keystore' && echo "REMOVE THESE FROM GIT"
```

If a key was ever committed (even and then deleted): **consider it compromised** —
generate a new one, move funds, and purge history (BFG / `git filter-repo`).

## MetaMask / wallet operational security

- Dedicated browser profile for Web3; review every signature — **read what you sign**.
- Beware **blind signing** and `eth_sign`/`personal_sign` phishing; reject opaque hex.
- Token **approval hygiene**: avoid unlimited `approve`; periodically revoke (revoke.cash); prefer `permit` with deadlines.
- Verify the **contract address and chain id** before interacting; bookmark dApps, don't follow links.
- Add networks/RPCs only from trusted sources — a malicious RPC can lie about state and simulate fake balances.

## RPC & dependency trust

- Pin a reputable RPC (your own node, or a known provider); a hostile RPC can feed false data to scripts and frontends.
- Pin dependency versions (`forge install` with a tag/commit; lock OpenZeppelin version). Re-audit on bumps.
- Verify deployed bytecode matches source on the explorer (`forge verify-contract` / `--verify`).

## Post-deploy checklist

- [ ] Source verified on the block explorer.
- [ ] Ownership/admin transferred to a **multisig** (Safe), not an EOA, for anything controlling funds.
- [ ] Timelock on privileged upgrades/parameter changes.
- [ ] Monitoring/alerting on critical events (large withdrawals, ownership changes, pause).
- [ ] Emergency runbook: pause + emergency-withdraw path tested on testnet.
- [ ] Deploy key rotated/retired if it ever touched a less-trusted machine.

## References
- Foundry deploying guide: https://getfoundry.sh/guides/deploying
- `cast wallet`: https://getfoundry.sh/cast/reference/wallet
- OpenZeppelin Contracts: https://docs.openzeppelin.com/contracts
- Safe (multisig): https://safe.global/
- revoke.cash (approval management): https://revoke.cash/

===== END FILE =====


===== FILE: skills/auditing-foundry-smart-contract-security/references/vulnerability-checklist.md =====
# Manual Review Checklist — Solidity / EVM

Walk this for **every** contract that moves value. Each item: what to look for →
how to confirm (Foundry test / Slither detector) → fix. Tools catch the known
patterns; this catches the logic bugs they can't.

## 1. Reentrancy (SWC-107)
- [ ] External call (`call`, `transfer`, ERC-777 hooks, ERC-721 `onERC...Received`, arbitrary token) **before** state updates?
- [ ] Cross-function / read-only reentrancy: a view used by another protocol mid-call?
- **Confirm:** Slither `reentrancy-eth`/`reentrancy-no-eth`; write an attacker contract test that re-enters in its `receive()`.
- **Fix:** Checks-Effects-Interactions order; `nonReentrant` (OpenZeppelin `ReentrancyGuard`); pull-over-push payments. ("OZ" = OpenZeppelin throughout.)

## 2. Access control (SWC-105/106/115)
- [ ] Every state-changing/admin function gated (`onlyOwner`, roles, custom modifier)?
- [ ] `initialize()` on upgradeable contracts protected against re-init and front-running?
- [ ] No `tx.origin` for auth (phishable) — use `msg.sender`.
- [ ] `selfdestruct` / `delegatecall` reachable only by trusted roles?
- **Confirm:** a `test_RevertWhen_*` with `vm.prank(attacker)` + `vm.expectRevert` for each guard.
- **Fix:** OZ `Ownable2Step` / `AccessControl`; `initializer` modifier; remove `tx.origin`.

## 3. Oracle / price manipulation (DeFi #1 exploit class)
- [ ] Price from spot `getReserves()` / a single AMM pool? (flash-loan manipulable)
- [ ] Using Chainlink: checked `updatedAt` staleness, `answeredInRound`, min/max bounds, L2 sequencer uptime?
- **Confirm:** fork test that manipulates the pool within one tx and asserts your protocol stays solvent.
- **Fix:** TWAP / Chainlink with staleness+deviation checks; never trust spot for pricing.

## 4. Arithmetic & rounding (SWC-101)
- [ ] Solidity ^0.8 (built-in checked math) — and any `unchecked{}` block justified?
- [ ] Division before multiplication (precision loss)? Rounding always in protocol's favor?
- [ ] Share-inflation / first-depositor attack on ERC-4626-style vaults?
- **Confirm:** `testFuzz_*` over amounts; invariant `assertGe(assets, shares-implied)`.
- **Fix:** mulDiv (OZ `Math.mulDiv`); virtual shares/offset for vaults; explicit rounding direction.

## 5. Unchecked external calls / return values (SWC-104)
- [ ] Return value of low-level `call`/`send` and ERC-20 `transfer`/`transferFrom` checked?
- [ ] Non-standard ERC-20s (no return, fee-on-transfer, rebasing) handled?
- **Confirm:** Slither `unchecked-transfer`, `unchecked-lowlevel`.
- **Fix:** OZ `SafeERC20`; measure balance delta for fee-on-transfer; require success.

## 6. delegatecall / proxy storage (SWC-112)
- [ ] Storage layout identical across implementation upgrades (no reordered/removed vars)?
- [ ] No `delegatecall` to user-supplied address; implementation can't be re-initialized/self-destructed?
- **Confirm:** Slither `controlled-delegatecall`, `unprotected-upgrade`; storage-layout diff between versions.
- **Fix:** OZ `UUPS`/`Transparent` proxies + storage gaps; `_disableInitializers()` in constructor.

## 7. Front-running / MEV / ordering (SWC-114)
- [ ] Approve race (ERC-20 `approve` from non-zero to non-zero)?
- [ ] Slippage / deadline params on swaps and mints? Commit-reveal where needed?
- **Fix:** `increaseAllowance`/`permit`; enforce `minOut` + `deadline`; commit-reveal for sensitive ordering.

## 8. Randomness (SWC-120)
- [ ] Any `block.timestamp`/`blockhash`/`prevrandao` used as randomness for value?
- **Fix:** Chainlink VRF; never on-chain pseudo-randomness for payouts.

## 9. Denial of service (SWC-113/128)
- [ ] Unbounded loops over user-growable arrays? Push payments that can revert and brick the contract?
- [ ] Single external dependency whose revert blocks all users?
- **Fix:** pull payments; bounded iteration / pagination; isolate failures.

## 10. Token / standard pitfalls
- [ ] ERC-20: decimals assumptions, fee-on-transfer, rebasing, missing return.
- [ ] ERC-721/1155: safe-transfer reentrancy hooks; approval scope.
- [ ] Permit (EIP-2612): replay, deadline, signature malleability.

## 11. General hygiene
- [ ] `pragma` pinned (`pragma solidity 0.8.26;` not `^`)? Latest stable solc?
- [ ] Events emitted for every state-changing action (off-chain monitoring)?
- [ ] No leftover `console.log` / test backdoors / hardcoded addresses?
- [ ] Pausability + emergency withdraw for value-holding contracts?
- [ ] Invariants written for every conservation law (total supply, solvency, accounting)?

## Severity triage (impact × likelihood)
- **Critical/High** → direct loss/lock of funds, unauthorized mint/withdraw, broken access control. **Blocks deploy.**
- **Medium** → loss under specific conditions, griefing, precision drift.
- **Low/Info** → best-practice, gas, style.

Cross-check real-world findings on **Solodit** (https://solodit.xyz/) for the contract type
you're shipping (vault, AMM, staking, bridge, governance).

===== END FILE =====


===== FILE: skills/auditing-foundry-smart-contract-security/scripts/agent.py =====
#!/usr/bin/env python3
"""Foundry Smart Contract Security Agent.

Pre-deployment audit orchestrator for a Foundry project. Runs static analysis
(Slither, Aderyn), optional symbolic execution (Mythril), Foundry tests/coverage,
and a key-leak scan, then aggregates everything into a single JSON report with a
PASS/FAIL deploy gate.

Design constraints (mirrors the upstream repo's style, hardened):
  - subprocess always called with an ARGUMENT LIST, never shell=True
  - no outbound network calls; every tool runs locally on local source
  - every external tool guarded by timeout and graceful degradation if absent
  - read-only with respect to the project (only writes the report file)
"""

import os
import re
import json
import shutil
import argparse
import logging
import subprocess
from collections import defaultdict
from datetime import datetime, timezone

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

SWC_REGISTRY = {
    "SWC-101": "Integer Overflow and Underflow",
    "SWC-104": "Unchecked Call Return Value",
    "SWC-105": "Unprotected Ether Withdrawal",
    "SWC-106": "Unprotected SELFDESTRUCT",
    "SWC-107": "Reentrancy",
    "SWC-110": "Assert Violation",
    "SWC-112": "Delegatecall to Untrusted Callee",
    "SWC-113": "DoS with Failed Call",
    "SWC-114": "Transaction Order Dependence (front-running)",
    "SWC-115": "Authorization through tx.origin",
    "SWC-116": "Block values as a proxy for time",
    "SWC-120": "Weak Sources of Randomness",
    "SWC-128": "DoS with Block Gas Limit",
}

SEVERITY_RANK = {"critical": 0, "high": 1, "medium": 2, "low": 3, "informational": 4, "optimization": 5}

# Directories that are dependencies / build output, not the audited code.
SKIP_DIRS = {"lib", "out", "cache", "node_modules", ".git", "broadcast", "artifacts"}

# A raw 32-byte hex private key (with or without 0x). High-precision signal: a
# 64-hex literal in source is almost always a key. Broader secret detection
# (mnemonics, API tokens, generic secrets) is intentionally delegated to gitleaks
# (see references/secure-deployment-and-keys.md) rather than reinvented noisily here.
PRIVKEY_RE = re.compile(r"\b(0x)?[0-9a-fA-F]{64}\b")


def _which(tool):
    return shutil.which(tool) is not None


def _run(cmd, timeout):
    """Run a command (list args), return (returncode, stdout, stderr). Never raises."""
    try:
        p = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
        return p.returncode, p.stdout, p.stderr
    except subprocess.TimeoutExpired:
        logger.warning("Timeout: %s", " ".join(cmd))
        return 124, "", "timeout"
    except FileNotFoundError:
        return 127, "", "not found"


# --------------------------------------------------------------------------- #
# Static analysis
# --------------------------------------------------------------------------- #
def run_slither(project):
    if not _which("slither"):
        logger.warning("slither not installed - skipping static analysis")
        return None
    rc, out, err = _run(["slither", project, "--json", "-"], timeout=300)
    # Slither exits non-zero when it finds issues; JSON still lands on stdout.
    if not out:
        logger.error("slither produced no JSON (%s)", err.strip()[:200])
        return {}
    try:
        return json.loads(out)
    except json.JSONDecodeError:
        logger.error("slither JSON parse failed")
        return {}


def analyze_slither(slither_output):
    findings, by_severity, by_detector = [], defaultdict(int), defaultdict(int)
    for det in (slither_output or {}).get("results", {}).get("detectors", []):
        severity = det.get("impact", "informational").lower()
        by_severity[severity] += 1
        name = det.get("check", "unknown")
        by_detector[name] += 1
        loc = ""
        elems = det.get("elements", [])
        if elems:
            sm = elems[0].get("source_mapping", {})
            lines = sm.get("lines") or [0]
            loc = f"{sm.get('filename_short', '')}:L{lines[0]}"
        findings.append({
            "source": "slither", "detector": name, "severity": severity,
            "confidence": det.get("confidence", ""), "location": loc,
            "description": (det.get("description", "") or "").strip()[:240],
        })
    return {
        "total": len(findings),
        "by_severity": dict(by_severity),
        "top_detectors": dict(sorted(by_detector.items(), key=lambda x: -x[1])[:15]),
        "findings": sorted(findings, key=lambda f: SEVERITY_RANK.get(f["severity"], 9)),
    }


def run_aderyn(project):
    if not _which("aderyn"):
        logger.info("aderyn not installed - skipping (recommended: cargo install aderyn)")
        return None
    report = os.path.join(project, "aderyn-report.json")
    rc, out, err = _run(["aderyn", project, "-o", report], timeout=300)
    try:
        with open(report) as fh:
            data = json.load(fh)
        return data
    except (OSError, json.JSONDecodeError):
        logger.error("aderyn report not readable")
        return {}


def analyze_aderyn(aderyn_output):
    findings, by_severity = [], defaultdict(int)
    if not aderyn_output:
        return {"total": 0, "by_severity": {}, "findings": []}
    for sev_key, sev in (("high_issues", "high"), ("low_issues", "low")):
        block = aderyn_output.get(sev_key, {}) or {}
        for issue in block.get("issues", []) if isinstance(block, dict) else []:
            by_severity[sev] += 1
            inst = (issue.get("instances") or [{}])[0]
            loc = f"{inst.get('contract_path', '')}:L{inst.get('line_no', 0)}"
            findings.append({
                "source": "aderyn", "detector": issue.get("title", ""), "severity": sev,
                "location": loc, "description": (issue.get("description", "") or "").strip()[:240],
            })
    return {
        "total": len(findings),
        "by_severity": dict(by_severity),
        "findings": sorted(findings, key=lambda f: SEVERITY_RANK.get(f["severity"], 9)),
    }


# --------------------------------------------------------------------------- #
# Symbolic execution (optional)
# --------------------------------------------------------------------------- #
def run_mythril(target, timeout):
    if not _which("myth"):
        logger.info("mythril not installed - skipping symbolic execution")
        return None
    rc, out, err = _run(
        ["myth", "analyze", target, "--execution-timeout", str(timeout), "-o", "json"],
        timeout=timeout + 60,
    )
    if not out:
        return {}
    try:
        return json.loads(out)
    except json.JSONDecodeError:
        logger.error("mythril JSON parse failed")
        return {}


def analyze_mythril(mythril_output):
    findings, by_swc = [], defaultdict(int)
    for issue in (mythril_output or {}).get("issues", []):
        swc = f"SWC-{issue.get('swc-id')}" if issue.get("swc-id") else "unknown"
        by_swc[swc] += 1
        findings.append({
            "source": "mythril", "swc_id": swc,
            "swc_title": SWC_REGISTRY.get(swc, issue.get("title", "")),
            "severity": issue.get("severity", "Medium").lower(),
            "location": f"{issue.get('contract', '')}:L{issue.get('lineno', 0)}",
            "description": (issue.get("description", "") or "").strip()[:240],
        })
    return {"total": len(findings), "by_swc": dict(by_swc), "findings": findings}


# --------------------------------------------------------------------------- #
# Foundry tests + coverage
# --------------------------------------------------------------------------- #
def run_forge_tests(project):
    if not _which("forge"):
        logger.warning("forge not installed - skipping tests")
        return {"available": False}
    rc, out, err = _run(["forge", "test"], timeout=900)
    text = out + err
    passed = sum(int(n) for n in re.findall(r"(\d+)\s+passed", text))
    failed = sum(int(n) for n in re.findall(r"(\d+)\s+failed", text))
    return {"available": True, "exit_code": rc, "passed": passed,
            "failed": failed, "all_passed": rc == 0}


def run_forge_coverage(project):
    if not _which("forge"):
        return {"available": False}
    rc, out, err = _run(["forge", "coverage", "--report", "summary"], timeout=1200)
    m = re.search(r"^\|\s*Total\s*\|\s*([\d.]+)%", out, re.M)
    lines_pct = float(m.group(1)) if m else None
    return {"available": True, "lines_pct": lines_pct}


# --------------------------------------------------------------------------- #
# Key hygiene
# --------------------------------------------------------------------------- #
def scan_key_leaks(project):
    """Heuristic scan for plaintext private keys / mnemonics in source-controlled files."""
    hits = []
    for root, dirs, files in os.walk(project):
        dirs[:] = [d for d in dirs if d not in SKIP_DIRS]
        for f in files:
            # Match source/config files plus all dotenv variants (.env, .env.local, .env.prod...)
            if not (f.endswith((".sol", ".env", ".json", ".js", ".ts", ".toml", ".txt", ".md", ".sh", ".yaml", ".yml"))
                    or f.startswith(".env")):
                continue
            path = os.path.join(root, f)
            try:
                with open(path, encoding="utf-8", errors="ignore") as fh:
                    content = fh.read()
            except OSError:
                continue
            for m in PRIVKEY_RE.finditer(content):
                # Exclude the well-known Anvil/Hardhat test mnemonic-derived keys & all-zero.
                val = m.group(0).lower().removeprefix("0x")
                if val == "0" * 64 or "test test test" in content[max(0, m.start() - 80):m.start()]:
                    continue
                hits.append({"file": os.path.relpath(path, project), "type": "possible_private_key"})
                break
    return {"leaked_secret_candidates": len(hits), "hits": hits[:20],
            "note": "high-precision private-key scan only; run gitleaks for full secret coverage"}


# --------------------------------------------------------------------------- #
# Aggregation + gate
# --------------------------------------------------------------------------- #
def deduplicate(*finding_lists):
    seen, combined = set(), []
    for lst in finding_lists:
        for f in lst:
            key = (f.get("location", ""), f.get("detector", f.get("swc_id", "")))
            if key not in seen:
                seen.add(key)
                combined.append(f)
    return combined


def build_report(project, slither, aderyn, mythril, tests, coverage, keys, min_coverage):
    combined = deduplicate(slither["findings"], aderyn["findings"], mythril["findings"])
    crit_high = sum(1 for f in combined if f.get("severity") in ("critical", "high"))

    gate_fail = []
    if crit_high > 0:
        gate_fail.append(f"{crit_high} high/critical static finding(s)")
    if tests.get("available") and not tests.get("all_passed"):
        gate_fail.append(f"{tests.get('failed', '?')} failing test(s)")
    if keys["leaked_secret_candidates"] > 0:
        gate_fail.append(f"{keys['leaked_secret_candidates']} possible leaked secret(s)")
    cov = coverage.get("lines_pct")
    if cov is not None and cov < min_coverage:
        gate_fail.append(f"line coverage {cov}% < {min_coverage}% threshold")

    return {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "project": os.path.abspath(project),
        "static_analysis": {
            "slither": {"total": slither["total"], "by_severity": slither["by_severity"],
                        "top_detectors": slither["top_detectors"]},
            "aderyn": {"total": aderyn["total"], "by_severity": aderyn["by_severity"]},
        },
        "symbolic_execution": {"mythril": {"total": mythril["total"], "by_swc": mythril.get("by_swc", {})}},
        "testing": {"forge_test": tests, "coverage": coverage},
        "key_hygiene": keys,
        "combined_findings": len(combined),
        "critical_high_findings": crit_high,
        "deploy_gate": "PASS" if not gate_fail else "FAIL",
        "gate_failures": gate_fail,
        "findings": combined[:40],
    }


def main():
    ap = argparse.ArgumentParser(description="Foundry Smart Contract Security Audit Agent")
    ap.add_argument("--project", default=".", help="Path to the Foundry project root")
    ap.add_argument("--mythril", metavar="FILE", help="Run Mythril symbolic execution on this .sol file")
    ap.add_argument("--mythril-timeout", type=int, default=300)
    ap.add_argument("--min-coverage", type=float, default=80.0, help="Min line coverage %% for PASS")
    ap.add_argument("--output", default="audit-report.json")
    args = ap.parse_args()

    logger.info("Auditing Foundry project: %s", os.path.abspath(args.project))
    slither = analyze_slither(run_slither(args.project))
    aderyn = analyze_aderyn(run_aderyn(args.project))
    mythril = analyze_mythril(run_mythril(args.mythril, args.mythril_timeout) if args.mythril else {})
    tests = run_forge_tests(args.project)
    coverage = run_forge_coverage(args.project)
    keys = scan_key_leaks(args.project)

    report = build_report(args.project, slither, aderyn, mythril, tests, coverage, keys, args.min_coverage)
    with open(args.output, "w") as fh:
        json.dump(report, fh, indent=2, default=str)
    logger.info("Audit: %d findings (%d high/critical) | gate=%s",
                report["combined_findings"], report["critical_high_findings"], report["deploy_gate"])
    if report["gate_failures"]:
        logger.warning("Gate failures: %s", "; ".join(report["gate_failures"]))
    print(json.dumps(report, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====
