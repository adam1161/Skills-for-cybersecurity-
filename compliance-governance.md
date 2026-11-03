

===== FILE: skills/achieving-cmmc-level-2-compliance/SKILL.md =====
---
name: achieving-cmmc-level-2-compliance
description: >-
  Prepare a defense-contractor environment for CMMC Level 2 certification: scope CUI
  and FCI, implement the 110 NIST SP 800-171 Rev 2 security requirements across 14
  families, compute the SPRS score with the DoD Assessment Methodology, manage a
  compliant POA&M, and ready the organization for a C3PAO assessment. Use when an
  organization handles Controlled Unclassified Information (CUI) under a DoD contract,
  when a contract carries DFARS clause 252.204-7012/7019/7020/7021, when preparing for
  or responding to a CMMC assessment, when computing or improving an SPRS score, when
  building a System Security Plan or POA&M for 800-171, or when scoping which systems
  are in the CUI boundary. Keywords: CMMC, CMMC Level 2, NIST 800-171, SP 800-171 Rev 2,
  CUI, FCI, SPRS, DFARS 7012, C3PAO, POA&M, System Security Plan, DoD Assessment
  Methodology, 110 controls, defense industrial base, DIB, FedRAMP equivalency.
domain: cybersecurity
subdomain: compliance-governance
tags:
- cmmc
- nist-800-171
- cui
- sprs
- dfars
- c3pao
- poam
- compliance
- governance
- defense-industrial-base
version: "1.0"
author: andrewibrah
license: Apache-2.0
nist_csf:
- GV.OC-03
- GV.SC-01
- ID.AM-08
- ID.RA-05
- PR.AA-01
- PR.DS-01
mitre_attack:
- T1078
- T1190
- T1041
- T1048
- T1567
---

# Achieving CMMC Level 2 Compliance

## When to Use

- When an organization in the **Defense Industrial Base (DIB)** stores, processes, or transmits **Controlled Unclassified Information (CUI)** under a DoD contract.
- When a contract includes **DFARS 252.204-7012** (safeguarding/incident reporting), **-7019/-7020** (NIST 800-171 self-assessment + SPRS), or the new **-7021** (CMMC requirement).
- When preparing for a **C3PAO** third-party assessment or a DoD-led assessment.
- When you must **compute, post, or improve an SPRS score** based on the NIST SP 800-171 DoD Assessment Methodology.
- When authoring or remediating a **System Security Plan (SSP)** and **POA&M** for the 110 requirements.
- When **scoping** which assets fall inside the CUI/FCI boundary (CUI assets, security-protection assets, contractor risk-managed assets, out-of-scope).

## Prerequisites

- Knowledge of **which contracts carry CUI** and the CUI categories involved (check the contract and the DoD CUI Registry).
- An asset inventory and network diagram so you can define the **CMMC assessment scope** before assessing controls.
- The **NIST SP 800-171 Rev 2** requirements and the **DoD Assessment Methodology** scoring weights.
- A documented **SSP** (its absence is itself a failed requirement — 3.12.4).
- Identification of any **External Service Providers (ESPs)** / cloud services touching CUI, and whether they meet **FedRAMP Moderate (or equivalency)**.

## Workflow

### 1. Determine applicability and CUI categories
Confirm the contract requires CMMC Level 2 (CUI present, not just FCI). FCI-only contracts are **Level 1** (the 15 FAR 52.204-21 requirements). Identify CUI categories from the contract and the DoD CUI Registry.

### 2. Scope the environment
Classify every asset into one of the CMMC scoping categories:
- **CUI Assets** — process/store/transmit CUI (in scope, assessed against all applicable controls).
- **Security Protection Assets** — provide security to the CUI environment (in scope).
- **Contractor Risk Managed Assets** — could but are not intended to handle CUI; managed by policy.
- **Specialized Assets** (IoT/OT, GFE, test equipment) — documented, limited assessment.
- **Out-of-Scope** — physically/logically isolated from CUI.

Minimize scope deliberately — a smaller, well-segmented CUI enclave is far cheaper to certify than a flat network.

### 3. Implement the 110 requirements (NIST SP 800-171 Rev 2)
Work the **14 families** (3.1–3.14). For each requirement, implement, then write the **how** in the SSP. High-leverage early wins: MFA (3.5.3), FIPS-validated cryptography (3.13.11), audit logging (3.3.x), access control + least privilege (3.1.x), and incident response (3.6.x).

### 4. Score with the DoD Assessment Methodology (SPRS)
Start at **110** and subtract the weighted value (**1, 3, or 5 points**) of each **unmet** requirement; partial credit applies to a small number of controls (e.g., MFA, FIPS crypto). The result is the **SPRS score** (maximum 110; the methodology floor is −203). Post the score, the SSP date, and the assessment scope to **SPRS** (or eMASS for higher assessments).

### 5. Build a compliant POA&M
Document every unmet requirement with owner, remediation, and milestone. **Constraints under the CMMC rule:** a **Conditional** status requires a score of at least **80%** (≥ 88 of 110), only **POA&M-eligible** requirements may be deferred (the highest-weighted security requirements must be fully met — verify eligibility against 32 CFR Part 170), and all POA&M items must be **closed within 180 days** to convert Conditional → **Final**.

### 6. Assess (self or C3PAO)
- **Level 1** and a subset of Level 2 = annual **self-assessment** with an affirmation in SPRS.
- **Level 2 (most CUI contracts)** = triennial **C3PAO** certification assessment.
- **Level 3** = DoD (DIBCAC) assessment on top of Level 2, adding SP 800-172 enhanced requirements.
Assessors evaluate each objective as **MET / NOT MET / N/A** with evidence (examine/interview/test). A senior official files the **annual affirmation** of continued compliance.

### 7. Maintain certification
Certification is valid **three years** with **annual affirmations**. Maintain the SSP, re-score on change, keep evidence current, and feed significant changes back into the assessment.

## Key Concepts

| Concept | Definition |
|---|---|
| FCI | Federal Contract Information — Level 1 protects it (FAR 52.204-21). |
| CUI | Controlled Unclassified Information — Level 2 protects it (NIST 800-171). |
| 110 requirements | The SP 800-171 Rev 2 security requirements across 14 families. |
| SPRS | Supplier Performance Risk System — where the 800-171 score is posted. |
| DoD Assessment Methodology | The 1/3/5-point weighting used to compute the score from 110. |
| C3PAO | CMMC Third-Party Assessment Organization — performs Level 2 certification. |
| POA&M | Plan of Action & Milestones — limited, must close in 180 days for Final status. |
| Conditional vs Final | Conditional = open POA&M (score ≥ 80%); Final = all controls met. |
| ESP | External Service Provider — must meet FedRAMP Moderate / equivalency for CUI. |
| Scoping categories | CUI / Security Protection / Contractor Risk Managed / Specialized / Out-of-Scope. |

## Tools & Systems

- **NIST SP 800-171 Rev 2** — the 110 requirements (and 800-171A for assessment objectives).
- **DoD NIST SP 800-171 Assessment Methodology** — the scoring weights.
- **32 CFR Part 170** (CMMC Program rule) and **48 CFR / DFARS 252.204-7021** (acquisition rule).
- **SPRS** — score posting; **SAM.gov** for registration.
- **SP 800-172 / 800-172A** — enhanced requirements for Level 3.
- **GRC / compliance tooling** — to manage the SSP, POA&M, and evidence (e.g., Xacta, RegScale, FutureFeed-style trackers).

## Common Scenarios

- **Prime flows CUI to a sub.** The sub needs its own Level 2 scope, SSP, SPRS score, and (most likely) C3PAO certification.
- **Score is below 88.** Prioritize the highest-weighted unmet requirements (5-point, then 3-point) to clear the conditional threshold and shrink the POA&M.
- **Cloud holds CUI.** Confirm the service is FedRAMP Moderate authorized or meets equivalency; document the responsibility split.
- **Flat network.** Re-scope into a segmented CUI enclave to cut the assessment surface before spending on controls.
- **Annual affirmation due.** A senior official affirms continued compliance in SPRS; let it lapse and you risk contract eligibility.

## Output Format

Produce a **CMMC Level 2 Readiness Report** using `assets/template.md`, containing:

1. **Applicability & CUI categories** — why Level 2 applies.
2. **Scope** — assets by scoping category and the CUI boundary diagram reference.
3. **Control status by family** — met / not met / N/A across the 14 families.
4. **SPRS score** — computed score, deductions, and the gap to 110 and to the 88 threshold.
5. **POA&M** — unmet requirements, eligibility check, owners, 180-day milestones.
6. **Assessment path** — self vs C3PAO, target date, affirmation owner.
7. **Remediation roadmap** — sequenced by point value and effort.

Use `scripts/process.py` to compute the SPRS score from a control-status JSON, flag POA&M-eligibility concerns, and report the gap to the conditional-certification threshold.

===== END FILE =====


===== FILE: skills/achieving-cmmc-level-2-compliance/LICENSE =====

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


===== FILE: skills/achieving-cmmc-level-2-compliance/assets/template.md =====
# CMMC Level 2 Readiness Report — Worked Example

> Filled example for a small DIB manufacturer handling CUI on a segmented enclave.
> Replace bracketed content for your own organization.

## 1. Applicability & CUI Categories
- **Contract drivers:** Prime subcontract with DFARS **252.204-7012** and **-7021**; CUI present → **CMMC Level 2** required.
- **CUI categories (from contract + DoD CUI Registry):** Controlled Technical Information (CTI), Export Controlled (EAR).
- **Target assessment path:** Triennial **C3PAO** certification (Phase 2 applies from Nov 10, 2026).

## 2. Scope (CMMC Level 2 Scoping Guide)
| Category | Examples in this environment |
|---|---|
| CUI Assets | Engineering workstations, CUI file share, the segmented "Enclave-1" VLAN |
| Security Protection Assets | EDR console, SIEM, firewall, IdP/MFA, jump host |
| Contractor Risk Managed | General corporate laptops (policy-blocked from CUI) |
| Specialized Assets | CNC machine controllers (documented, isolated) |
| Out-of-Scope | Guest Wi-Fi, marketing SaaS |

**Boundary note:** CUI is confined to Enclave-1 behind segmentation and MFA. Deliberately minimized to shrink the assessment surface. See network diagram `CUI-boundary-v3`.

## 3. Control Status by Family (NIST SP 800-171 Rev 2)
*(summary; full per-requirement status lives in the SSP)*

| Family | Met | Partial | Not Met | N/A |
|---|---|---|---|---|
| 3.1 Access Control | 22 | 0 | 0 | 0 |
| 3.3 Audit & Accountability | 8 | 0 | 1 | 0 |
| 3.5 Identification & Auth | 10 | 1 | 0 | 0 |
| 3.8 Media Protection | 8 | 0 | 1 | 0 |
| 3.13 System & Comms Protection | 15 | 0 | 1 | 0 |
| 3.14 System & Info Integrity | 6 | 0 | 1 | 0 |
| *(others)* | all met | — | — | — |

## 4. SPRS Score
*(computed by `scripts/process.py` from the control-status JSON)*

- **Score: 97 / 110** (started at 110; deducted 13).
- **Gap to perfect:** 13 points across 4 not-met + 1 partial requirement.
- **Conditional threshold (≥ 88):** **MET** (margin 9) — eligible for Conditional status *if* the remaining items are POA&M-eligible.
- **Posted to SPRS:** score, SSP date, and assessment scope.

## 5. POA&M (eligibility-checked)
| ID | Requirement | Points | Eligibility | Remediation | Owner | Milestone (≤180d) |
|---|---|---|---|---|---|---|
| 3.3.1 | Audit log generation/coverage | 5 | **Verify** — high weight; confirm against 32 CFR 170 | Enable full audit policy + ship to SIEM | SecOps | 2026-07-30 |
| 3.13.11 | FIPS-validated cryptography | 3 | **Verify** eligibility | Replace non-validated module with FIPS 140-validated | Infra | 2026-08-15 |
| 3.5.3 | MFA (partial) | 3 | Partial-credit control | Extend MFA to remaining admin paths | IAM | 2026-07-20 |
| 3.8.9 | Backup CUI protection | 1 | Eligible | Encrypt + access-control backup store | Infra | 2026-08-31 |
| 3.14.1 | Flaw remediation | 1 | Eligible | Formalize patch SLA + tracking | IT | 2026-08-31 |

> The two 3-point and one 5-point items must clear eligibility review; the highest-weighted security requirements generally cannot remain on a POA&M. All items close within **180 days** to convert Conditional → **Final**.

## 6. Assessment Path
- **Type:** C3PAO certification assessment.
- **Target window:** Q4 2026, after POA&M closure of the high-weight items.
- **Affirmation owner:** [senior official] files the annual affirmation in SPRS.

## 7. Remediation Roadmap (sequenced by point value, then effort)
1. **3.3.1 audit logging (5 pts)** — biggest score lever and likely POA&M-ineligible → do first.
2. **3.13.11 FIPS crypto (3 pts)** and **3.5.3 MFA gap (3 pts)** — close to remove eligibility risk.
3. **3.8.9, 3.14.1 (1 pt each)** — low-effort cleanups before the C3PAO date.
4. Re-run the SPRS calculator after each closure; goal is **110** before assessment.

===== END FILE =====


===== FILE: skills/achieving-cmmc-level-2-compliance/references/standards.md =====
# CMMC Level 2 — Standards & Reference

## Governing rules
| Rule | Citation | Status / effective date |
|---|---|---|
| CMMC Program rule | 32 CFR Part 170 | Effective **December 16, 2024** |
| CMMC acquisition rule (DFARS) | 48 CFR; DFARS clause **252.204-7021** (and 204.7503) | Published Sept 10, 2025; effective **November 10, 2025** |
| Safeguarding CUI / incident reporting | DFARS **252.204-7012** | In effect |
| NIST 800-171 self-assessment + SPRS posting | DFARS **252.204-7019 / -7020** | In effect |

> Always confirm current status at the source — acquisition rules and phase dates have moved before. Authoritative: https://dodcio.defense.gov/CMMC/ and the eCFR for 32 CFR Part 170.

## Phased rollout (per the acquisition rule)
| Phase | Begins | What applies |
|---|---|---|
| Phase 1 | **Nov 10, 2025** | Level 1 and some Level 2 **self-assessment** required in solicitations |
| Phase 2 | **Nov 10, 2026** | Level 2 **C3PAO certification** required for applicable contracts |
| Phase 3 | **Nov 10, 2027** | Level 2 C3PAO + Level 3 **DIBCAC** assessment phased in |
| Phase 4 | **Nov 10, 2028** | Full implementation across applicable DoD contracts |

## The three CMMC levels
| Level | Protects | Requirements | Assessment |
|---|---|---|---|
| Level 1 | FCI | 15 requirements (FAR 52.204-21) | Annual self-assessment + affirmation |
| Level 2 | CUI | **110 requirements (NIST SP 800-171 Rev 2)** | Self **or** triennial C3PAO certification |
| Level 3 | CUI (high priority) | 110 + selected **SP 800-172** enhanced | DoD (DIBCAC) assessment |

Certification validity: **3 years**, with **annual affirmation** by a senior official in SPRS.

## NIST SP 800-171 Rev 2 — the 14 families (110 requirements)
| § | Family | # reqs |
|---|---|---|
| 3.1 | Access Control | 22 |
| 3.2 | Awareness and Training | 3 |
| 3.3 | Audit and Accountability | 9 |
| 3.4 | Configuration Management | 9 |
| 3.5 | Identification and Authentication | 11 |
| 3.6 | Incident Response | 3 |
| 3.7 | Maintenance | 6 |
| 3.8 | Media Protection | 9 |
| 3.9 | Personnel Security | 2 |
| 3.10 | Physical Protection | 6 |
| 3.11 | Risk Assessment | 3 |
| 3.12 | Security Assessment | 4 |
| 3.13 | System and Communications Protection | 16 |
| 3.14 | System and Information Integrity | 7 |
| | **Total** | **110** |

(Assessment objectives for each requirement are in **NIST SP 800-171A**.)

## DoD Assessment Methodology — SPRS scoring
- Start at **110**. Subtract the weighted value of each **NOT MET** requirement.
- Weights: **1, 3, or 5 points**. The most security-significant requirements are weighted 3 or 5.
- **Partial credit** applies to a small number of requirements (notably MFA at 3.5.3 and FIPS-validated cryptography at 3.13.11) where partial implementation reduces the deduction.
- Maximum score **110**; the methodology floor is **−203** (more is deducted than the 110 starting points because of the weighting).
- The complete per-requirement point assignment is published in the **DoD NIST SP 800-171 Assessment Methodology** — use that document for the authoritative weight of each control rather than estimating.

## POA&M rules under the CMMC rule (32 CFR Part 170)
- A **Conditional** Level 2 status is allowed only if the assessment score is **at least 80% (≥ 88 of 110)**.
- Only **POA&M-eligible** requirements may be deferred. The highest-weighted security requirements generally **must be fully met** and **cannot** sit on a POA&M — verify each item's eligibility against the rule.
- All POA&M items must be **closed within 180 days**; a closeout assessment then converts **Conditional → Final**.

## Scoping categories (CMMC Level 2 Scoping Guide)
| Category | Treatment |
|---|---|
| CUI Assets | Process/store/transmit CUI — assessed against applicable requirements. |
| Security Protection Assets | Provide security to the CUI environment — in scope. |
| Contractor Risk Managed Assets | Capable of handling CUI but not intended to — managed by policy/config. |
| Specialized Assets | IoT/OT, GFE, test equipment — documented, limited assessment. |
| Out-of-Scope Assets | Isolated from CUI — not assessed. |

## External Service Providers / cloud
- Cloud services that store/process/transmit CUI must be **FedRAMP Moderate authorized or meet FedRAMP Moderate equivalency**.
- Document the customer/provider responsibility split (CRM) and inherited controls in the SSP.

## NIST CSF 2.0 alignment
| CSF 2.0 ID | Relevance |
|---|---|
| GV.OC-03 | Legal/regulatory (DFARS/CMMC) requirements understood. |
| GV.SC-01 | Supply-chain risk management — flowdown to subs / ESPs. |
| ID.AM-08 | Assets managed across the lifecycle (scoping). |
| ID.RA-05 | Risk informs prioritization of unmet requirements. |
| PR.AA-01 | Identity and access (3.1 / 3.5 families). |
| PR.DS-01 | Data-at-rest protection (FIPS crypto, media protection). |

===== END FILE =====


===== FILE: skills/achieving-cmmc-level-2-compliance/scripts/process.py =====
#!/usr/bin/env python3
"""
CMMC Level 2 / NIST SP 800-171 Rev 2 SPRS score calculator.

Implements the DoD Assessment Methodology arithmetic: start at 110 and subtract
the weighted value (1, 3, or 5) of each NOT MET requirement, with partial credit
for the small set of requirements that allow it. Reports the SPRS score, the gap
to a perfect 110 and to the 88-point (80%) conditional-certification threshold,
and flags higher-weighted unmet requirements whose POA&M eligibility must be
verified against 32 CFR Part 170.

NOTE: per-requirement point weights are defined by the DoD NIST SP 800-171
Assessment Methodology. Supply each requirement's official weight in the input
(this tool does not invent weights). Use status 'partial' with 'partial_deduction'
only for requirements the methodology allows partial credit on (e.g., 3.5.3 MFA,
3.13.11 FIPS crypto).

Input JSON shape:
{
  "org": {"name": "Acme Defense LLC", "scope": "CUI enclave"},
  "requirements": [
    {"id": "3.1.1",  "family": "3.1", "status": "met",      "weight": 5},
    {"id": "3.5.3",  "family": "3.5", "status": "partial",  "weight": 5, "partial_deduction": 3},
    {"id": "3.3.1",  "family": "3.3", "status": "not_met",  "weight": 5},
    {"id": "3.8.9",  "family": "3.8", "status": "not_met",  "weight": 1},
    {"id": "3.2.1",  "family": "3.2", "status": "na",       "weight": 1}
  ]
}

status: met | not_met | partial | na

Usage:
  python process.py --input controls.json [--output readiness.md]
  python process.py --input controls.json --require-conditional   # exit 1 if score < 88
"""

import argparse
import json
import sys

START_SCORE = 110
CONDITIONAL_THRESHOLD = 88   # 80% of 110
VALID_STATUS = {"met", "not_met", "partial", "na"}
VALID_WEIGHTS = {1, 3, 5}


def compute(data):
    reqs = data.get("requirements", [])
    if not reqs:
        raise ValueError("requirements list is required")

    deductions = 0
    counts = {"met": 0, "not_met": 0, "partial": 0, "na": 0}
    poam_flags = []   # higher-weight unmet -> verify POA&M eligibility
    by_family = {}    # family -> {met,not_met,partial,na}
    detail = []

    for r in reqs:
        rid = r.get("id", "?")
        status = r.get("status")
        weight = r.get("weight")
        if status not in VALID_STATUS:
            raise ValueError(f"{rid}: status '{status}' invalid (met|not_met|partial|na)")
        if status in ("not_met", "partial", "met") and weight not in VALID_WEIGHTS:
            raise ValueError(f"{rid}: weight '{weight}' invalid (must be 1, 3, or 5)")

        fam = r.get("family", rid.rsplit(".", 1)[0])
        fam_rec = by_family.setdefault(fam, {"met": 0, "not_met": 0, "partial": 0, "na": 0})
        fam_rec[status] += 1
        counts[status] += 1

        ded = 0
        if status == "not_met":
            ded = weight
            if weight > 1:
                poam_flags.append((rid, weight))
        elif status == "partial":
            ded = r.get("partial_deduction")
            if ded is None:
                raise ValueError(f"{rid}: status 'partial' requires 'partial_deduction'")
            if ded < 0 or ded > weight:
                raise ValueError(f"{rid}: partial_deduction {ded} out of range (0..{weight})")
            if ded > 1:
                poam_flags.append((rid, ded))
        deductions += ded
        detail.append((rid, fam, status, weight, ded))

    score = START_SCORE - deductions
    return {
        "score": score,
        "deductions": deductions,
        "counts": counts,
        "by_family": by_family,
        "poam_flags": poam_flags,
        "detail": detail,
    }


def render(data, res):
    org = data.get("org", {})
    lines = []
    lines.append(f"# CMMC Level 2 Readiness - {org.get('name','Organization')}")
    lines.append("")
    if org.get("scope"):
        lines.append(f"- **Scope:** {org['scope']}")
    lines.append("")

    score = res["score"]
    lines.append("## SPRS Score (DoD Assessment Methodology)")
    lines.append("")
    lines.append(f"- **Score:** **{score}** / 110  (started at 110, deducted {res['deductions']})")
    lines.append(f"- **Gap to perfect (110):** {110 - score}")
    if score >= CONDITIONAL_THRESHOLD:
        lines.append(f"- **Conditional threshold (>= {CONDITIONAL_THRESHOLD}):** MET "
                     f"(margin {score - CONDITIONAL_THRESHOLD}) - eligible for Conditional status "
                     "if remaining items are POA&M-eligible.")
    else:
        lines.append(f"- **Conditional threshold (>= {CONDITIONAL_THRESHOLD}):** NOT MET "
                     f"(short by {CONDITIONAL_THRESHOLD - score}) - not eligible for Conditional "
                     "certification until the score reaches 88.")
    c = res["counts"]
    lines.append(f"- **Status tally:** met {c['met']}, partial {c['partial']}, "
                 f"not met {c['not_met']}, N/A {c['na']}")
    lines.append("")

    # by family
    lines.append("## Status by family")
    lines.append("")
    lines.append("| Family | Met | Partial | Not Met | N/A |")
    lines.append("|---|---|---|---|---|")
    for fam in sorted(res["by_family"]):
        f = res["by_family"][fam]
        lines.append(f"| {fam} | {f['met']} | {f['partial']} | {f['not_met']} | {f['na']} |")
    lines.append("")

    # POA&M eligibility flags
    lines.append("## POA&M eligibility check")
    lines.append("")
    if not res["poam_flags"]:
        lines.append("No unmet requirement carries more than 1 point of deduction. "
                     "Remaining gaps are most likely POA&M-eligible (still verify against 32 CFR Part 170).")
    else:
        lines.append("The following unmet/partial requirements carry **> 1 point**. The highest-weighted "
                     "security requirements generally **cannot** sit on a POA&M - verify each against "
                     "32 CFR Part 170 before relying on Conditional status:")
        lines.append("")
        lines.append("| Requirement | Points lost |")
        lines.append("|---|---|")
        for rid, w in sorted(res["poam_flags"], key=lambda x: -x[1]):
            lines.append(f"| {rid} | {w} |")
    lines.append("")
    lines.append("> All POA&M items must be closed within **180 days** to convert Conditional -> Final.")

    return "\n".join(lines)


def main():
    ap = argparse.ArgumentParser(description="CMMC L2 / NIST 800-171 SPRS score calculator")
    ap.add_argument("--input", "-i", required=True, help="Path to control-status JSON")
    ap.add_argument("--output", "-o", help="Write Markdown readiness report to this path")
    ap.add_argument("--require-conditional", action="store_true",
                    help="Exit non-zero if SPRS score < 88 (conditional threshold)")
    args = ap.parse_args()

    try:
        with open(args.input) as f:
            data = json.load(f)
    except (OSError, json.JSONDecodeError) as e:
        print(f"ERROR: could not read input JSON: {e}", file=sys.stderr)
        return 2

    try:
        res = compute(data)
        md = render(data, res)
    except ValueError as e:
        print(f"ERROR: {e}", file=sys.stderr)
        return 2

    if args.output:
        with open(args.output, "w") as f:
            f.write(md + "\n")
        print(f"Readiness report written to {args.output}", file=sys.stderr)
    else:
        print(md)

    print(f"SPRS score {res['score']}/110 (deductions {res['deductions']}; "
          f"not met {res['counts']['not_met']}, partial {res['counts']['partial']}).",
          file=sys.stderr)

    if args.require_conditional and res["score"] < CONDITIONAL_THRESHOLD:
        print(f"FAIL: score {res['score']} < {CONDITIONAL_THRESHOLD} conditional threshold.",
              file=sys.stderr)
        return 1
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/conducting-cyber-risk-assessment-with-nist-800-30/SKILL.md =====
---
name: conducting-cyber-risk-assessment-with-nist-800-30
description: >-
  Conduct a defensible cybersecurity risk assessment using the NIST SP 800-30 Rev 1
  methodology: prepare scope and a risk model, identify threat sources and threat events,
  identify vulnerabilities and predisposing conditions, determine likelihood and impact,
  compute risk, and communicate results as a prioritized risk register. Use when an
  organization needs an actual risk *assessment* (not a maturity score), when a control
  framework (CSF, ISO 27001, RMF, SOC 2, PCI) requires a documented risk analysis as input,
  when leadership asks "what are our top risks and how bad are they", when assessing risk
  for a new system or major change, or when building a risk register from scratch. This is
  the methodology that feeds framework selection, ATO packages, and treatment decisions.
  Keywords: risk assessment, NIST 800-30, threat modeling, likelihood and impact, risk
  register, risk analysis, threat sources, vulnerabilities, risk determination, qualitative
  risk, risk matrix, residual risk, risk treatment.
domain: cybersecurity
subdomain: compliance-governance
tags:
- risk-assessment
- nist-800-30
- risk-management
- threat-modeling
- risk-register
- governance
- nist-800-39
- compliance
version: "1.0"
author: andrewibrah
license: Apache-2.0
nist_csf:
- GV.RM-01
- ID.RA-01
- ID.RA-03
- ID.RA-04
- ID.RA-05
mitre_attack:
- T1566
- T1078
- T1190
- T1486
- T1021
---

# Conducting a Cyber Risk Assessment with NIST SP 800-30

## When to Use

- When the organization needs a real risk *assessment* — an analysis of specific threats, likelihoods, and impacts — rather than a maturity score against a framework. (Maturity tells you how mature your practices are; a risk assessment tells you what could hurt you and how badly.)
- When another framework requires a documented risk analysis as a mandatory input: NIST CSF (ID.RA), ISO 27001 (Clause 6.1.2), NIST RMF / 800-37 (the Prepare and Select steps), SOC 2 (CC3), PCI DSS, or HIPAA (§164.308(a)(1)(ii)(A)).
- When standing up or significantly changing a system and you must understand its risk before authorization or go-live.
- When leadership asks for the organization's top risks, ranked, with a rationale they can defend to a board or regulator.
- When building or refreshing an enterprise risk register.

## Prerequisites

- An inventory of in-scope assets, systems, and the information types they handle (system boundary defined).
- Access to threat intelligence (internal incident history, sector ISAC feeds, MITRE ATT&CK) to ground threat-event likelihood in observed behavior.
- Vulnerability data (scan results, pen-test findings, configuration/architecture review) for the in-scope systems.
- Business context: which missions/processes the systems support, and what impact to confidentiality, integrity, or availability would mean in business terms.
- Agreement on the **risk model and scales** before scoring, so results are comparable and repeatable (see `references/standards.md`).
- Familiarity with the three-tier risk-management context from NIST SP 800-39 (organization, mission/business process, information system).

## Workflow

NIST SP 800-30 Rev 1 defines four steps. Steps 1 and 4 bookend the assessment; Step 2 is the analytic core.

### 1. Prepare for the assessment
Define and document:
- **Purpose** (e.g., support an authorization decision, inform control selection, satisfy ISO 6.1.2).
- **Scope** — organizational tier (Tier 1/2/3), systems, and time horizon.
- **Assumptions and constraints** (e.g., assume an external adversarial threat with moderate capability).
- **Information sources** — threat, vulnerability, and impact inputs.
- **Risk model and analytic approach** — the factors (threat source, threat event, vulnerability, likelihood, impact) and the scales (qualitative Very Low–Very High, or semi-quantitative 0–10). Lock these now.

### 2. Conduct the assessment
Work through the analytic tasks in order. The 800-30 appendices provide the reference taxonomies (D–I).

**2a. Identify threat sources (Appendix D).** Classify by type: **Adversarial** (individuals, groups, nation-states — characterize capability, intent, targeting), **Accidental** (user error), **Structural** (equipment/software failure), **Environmental** (natural disasters, infrastructure outages).

**2b. Identify threat events (Appendix E).** The specific actions a source could take (e.g., "adversary exfiltrates credentials via phishing then moves laterally"). Map adversarial events to MITRE ATT&CK techniques for traceability.

**2c. Identify vulnerabilities and predisposing conditions (Appendix F).** Weaknesses (missing MFA, unpatched service) and conditions that make exploitation more or less likely (internet exposure, flat network, lack of segmentation).

**2d. Determine likelihood (Appendix G).** Assess the likelihood that a threat event is *initiated* (adversarial) or *occurs* (non-adversarial), and the likelihood it results in adverse impact given the vulnerabilities. Combine into an overall likelihood on the agreed scale.

**2e. Determine impact (Appendix H).** Magnitude of harm if the event succeeds — to operations, assets, individuals, other organizations, or the nation. Express against the agreed scale and in business terms.

**2f. Determine risk (Appendix I).** Risk is a function of likelihood and impact. Plot each threat event on the risk matrix (e.g., likelihood × impact → Very Low … Very High). Record the risk level, the contributing factors, and uncertainty/assumptions.

### 3. Communicate and share results
Produce the **risk register** and an executive briefing. For each risk: the threat event, affected assets, likelihood, impact, risk level, key contributing vulnerabilities, and a recommended treatment. Rank by risk level so decision-makers see the top risks first.

### 4. Maintain the assessment
Risk is not static. Define a refresh cadence and the triggers that force re-assessment (new system, major architecture change, significant incident, new threat intel). Track risk-acceptance decisions and treatment progress over time.

### 5. Drive treatment decisions
Hand the ranked register to risk owners. For each risk choose a treatment — **mitigate** (add/strengthen controls), **transfer** (insurance, contractual), **avoid** (stop the activity), or **accept** (document residual risk with an authorizing signature). Re-score residual risk after planned controls to show the post-treatment position.

## Key Concepts

| Concept | Definition |
|---|---|
| Threat source | The cause of a threat event: adversarial, accidental, structural, or environmental. |
| Threat event | A specific action or occurrence that could cause harm (mapped to ATT&CK for adversarial cases). |
| Vulnerability | A weakness that a threat event can exploit. |
| Predisposing condition | A condition that increases or decreases the likelihood of adverse impact (e.g., internet exposure). |
| Likelihood | The chance a threat event initiates/occurs *and* results in adverse impact. |
| Impact | The magnitude of harm if the event succeeds. |
| Risk | A function of likelihood and impact; the expected harm to the organization. |
| Inherent vs residual risk | Risk before vs after planned/implemented controls. |
| Risk tolerance / appetite | The level of risk leadership is willing to accept. |
| Risk register | The prioritized record of risks, scores, owners, and treatments. |

## Tools & Systems

- **NIST SP 800-30 Rev 1** — the methodology and Appendices D–I taxonomies (threat sources, events, vulnerabilities, likelihood, impact, risk).
- **NIST SP 800-39** — enterprise risk-management context (three tiers).
- **MITRE ATT&CK** — to enumerate and ground adversarial threat events in observed TTPs.
- **Vulnerability scanners / pen-test reports** — empirical vulnerability input.
- **Threat intel sources** — sector ISACs, vendor feeds, internal incident history for likelihood grounding.
- **GRC / risk-register tooling** — spreadsheet, or platforms such as ServiceNow GRC, Archer, OneTrust, to store and track the register.
- **FAIR** (optional) — a quantitative model if leadership wants risk expressed in dollar ranges rather than qualitative bands.

## Common Scenarios

- **CSF/ISO/SOC 2 needs a risk analysis.** Run 800-30 to produce the documented assessment those frameworks require as input, then map results to their control sets.
- **New system before go-live.** Assess threat events against the system's architecture and feed the result into the authorization (RMF Select/Authorize) decision.
- **Board wants the top risks.** Deliver a ranked register with impacts in business terms and clear treatment recommendations.
- **Post-incident.** Re-assess the affected systems; the incident updates likelihood evidence and may surface new threat events.
- **Annual refresh.** Re-score against current threat intel and control changes; show movement in residual risk year over year.

## Output Format

Produce a **Risk Assessment Report** using `assets/template.md`, containing:

1. **Purpose, scope, and tier** — what was assessed and why.
2. **Assumptions, constraints, and risk model** — the factors and scales used (so results are reproducible).
3. **Threat sources** — by type, with adversarial characterization.
4. **Threat events** — each with affected assets and ATT&CK mapping where adversarial.
5. **Vulnerabilities and predisposing conditions** — tied to threat events.
6. **Risk register** — table: ID, threat event, asset, likelihood, impact, **risk level**, contributing vulnerabilities, recommended treatment, owner, residual risk.
7. **Top risks summary** — ranked, in business terms, for leadership.
8. **Maintenance plan** — refresh cadence and re-assessment triggers.

Use `scripts/process.py` to score the register from a risk-input JSON (likelihood × impact → risk level on a configurable matrix), rank risks, and emit the register table.

===== END FILE =====


===== FILE: skills/conducting-cyber-risk-assessment-with-nist-800-30/LICENSE =====

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


===== FILE: skills/conducting-cyber-risk-assessment-with-nist-800-30/assets/template.md =====
# Risk Assessment Report (NIST SP 800-30 Rev 1) — Worked Example

> Filled example for a mid-size SaaS company assessing its internet-facing
> application tier. Replace bracketed content for your own assessment.

## 1. Purpose, Scope, and Tier
- **Purpose:** Inform control-investment decisions for FY26 and satisfy the documented risk-analysis input required by the company's SOC 2 (CC3) program and ISO 27001 Clause 6.1.2.
- **Scope:** Internet-facing application tier — customer web portal, its identity provider (IdP), the API gateway, and the data lake behind them.
- **Risk-management tier (SP 800-39):** Tier 3 (information system), rolling up to Tier 2 (the "Customer Platform" mission).
- **Assessment window:** FY26 Q2; valid 12 months or until a triggering change.

## 2. Assumptions, Constraints, and Risk Model
- **Assumptions:** A financially motivated external adversary of moderate-to-high capability is actively targeting the sector; internal users are trusted but fallible.
- **Constraints:** Assessment is architecture- and scan-driven; no red-team engagement was run this cycle.
- **Risk model:** Factors = threat source, threat event, vulnerability/predisposing condition, likelihood, impact. Adversarial threat events are mapped to MITRE ATT&CK.
- **Scales:** Five-level qualitative (Very Low → Very High). Risk computed via the 800-30 Rev 1 reference 5×5 matrix (see `references/standards.md`). This matrix was agreed during Prepare and is locked for the cycle.

## 3. Threat Sources (Appendix D)
| Source | Type | Characterization |
|---|---|---|
| Organized cybercrime group | Adversarial | High capability, financial intent, opportunistic + targeted |
| Authorized employee | Accidental | Misconfiguration / human error |
| Hardware/software failure | Structural | Storage, service, or dependency failure |
| Regional power / cloud-region outage | Environmental | Low frequency, availability impact |

## 4. Threat Events (Appendix E)
| ID | Threat event | Source | ATT&CK |
|---|---|---|---|
| R-01 | Adversary phishes employee credentials, then moves laterally to the IdP | Adversarial | T1566, T1021 |
| R-02 | Adversary exploits an unpatched edge VPN to deploy ransomware | Adversarial | T1190 |
| R-04 | Employee misconfigures a data-lake bucket to public | Accidental | — |
| R-03 | SIEM storage disk fails, corrupting un-backed-up logs | Structural | — |

## 5. Vulnerabilities and Predisposing Conditions (Appendix F)
| Threat event | Vulnerabilities / conditions |
|---|---|
| R-01 | No phishing-resistant MFA; flat internal network (no segmentation) |
| R-02 | VPN appliance missing a critical patch; internet-exposed management plane |
| R-04 | No preventive SCP/guardrail; broad write permissions on storage |
| R-03 | No RAID; no offsite/immutable backup of SIEM data |

## 6. Risk Register
*(generated by `scripts/process.py`; likelihood × impact → risk on the 800-30 5×5 matrix, ranked highest-first)*

| ID | Threat event | Source | Asset | ATT&CK | Likelihood | Impact | Risk | Key vulnerabilities | Treatment | Residual | Owner |
|---|---|---|---|---|---|---|---|---|---|---|---|
| R-01 | Phish → lateral movement to IdP | Adversarial | Identity provider | T1566, T1021 | High | Very High | **Very High** | No phishing-resistant MFA; flat network | Mitigate | Moderate | IAM Lead |
| R-02 | Ransomware via unpatched edge VPN | Adversarial | VPN concentrator | T1190 | Moderate | Very High | **High** | Missing critical patch | Mitigate | Low | Infra |
| R-04 | Public S3 misconfiguration | Accidental | Data lake | — | Moderate | High | **Moderate** | No SCP guardrail | Mitigate | Low | Cloud |
| R-03 | Disk failure corrupts logs | Structural | SIEM storage | — | Low | Moderate | **Low** | No RAID; no offsite backup | Accept | Low | SOC |

## 7. Top Risks Summary (for leadership)
1. **R-01 — Very High.** A successful phish into our identity provider is the single most damaging plausible event: it grants broad access and, on a flat network, fast lateral movement. **Recommendation:** deploy phishing-resistant MFA (FIDO2) and segment the IdP — this is the highest-leverage control this fiscal year.
2. **R-02 — High.** Edge-VPN ransomware is a sector-common entry path. **Recommendation:** emergency-patch the appliance, restrict the management plane, and validate offline backups.
3. **R-04 — Moderate.** Cloud misconfiguration is likely but bounded; a preventive SCP guardrail collapses the likelihood cheaply.

## 8. Maintenance Plan
- **Cadence:** Full re-assessment annually; register reviewed quarterly.
- **Re-assessment triggers:** new internet-facing system, major architecture change, any incident affecting in-scope assets, or significant new threat intel (e.g., active exploitation of a component we run).
- **Tracking:** Treatment progress and risk-acceptance signatures recorded in the GRC tool; residual risk re-scored after each control lands to show year-over-year movement.

===== END FILE =====


===== FILE: skills/conducting-cyber-risk-assessment-with-nist-800-30/references/standards.md =====
# NIST SP 800-30 — Standards & Reference

## Primary standard
### NIST SP 800-30 Revision 1 — Guide for Conducting Risk Assessments
- **Publisher**: NIST
- **Published**: September 2012
- **Scope**: The risk-assessment component of the broader risk-management process.
- **URL**: https://csrc.nist.gov/pubs/sp/800/30/r1/final

## Companion standards
| Document | Role |
|---|---|
| NIST SP 800-39 | Managing Information Security Risk — three-tier context (Tier 1 organization, Tier 2 mission/business process, Tier 3 information system). |
| NIST SP 800-37 Rev 2 | Risk Management Framework — risk assessment feeds the Prepare, Select, and Authorize steps. |
| NIST SP 800-53 Rev 5 | Control catalog — source of mitigating controls chosen in risk treatment. |
| FIPS 199 / FIPS 200 | Security categorization (L/M/H per C/I/A) and minimum requirements. |
| MITRE ATT&CK | Adversarial threat-event enumeration and traceability. |
| FAIR (Open Group) | Optional quantitative risk model (dollar-range loss exposure). |

## The four-step process (800-30 Rev 1)
1. **Prepare** — purpose, scope, assumptions/constraints, sources, risk model and scales.
2. **Conduct** — tasks 2a–2f below.
3. **Communicate** — risk register + briefing.
4. **Maintain** — monitoring and refresh.

### Conduct tasks and their reference appendices
| Task | Appendix | Output |
|---|---|---|
| Identify threat sources | D | Adversarial / Accidental / Structural / Environmental sources |
| Identify threat events | E | Specific events (map adversarial to ATT&CK) |
| Identify vulnerabilities & predisposing conditions | F | Weaknesses + conditions affecting impact likelihood |
| Determine likelihood | G | Likelihood of initiation/occurrence and of adverse impact |
| Determine impact | H | Magnitude of harm |
| Determine risk | I | Risk level = f(likelihood, impact) |

## Threat source types (Appendix D)
- **Adversarial** — individuals, groups, organizations, nation-states. Characterize by capability, intent, and targeting.
- **Accidental** — erroneous actions by authorized users.
- **Structural** — failures of equipment, software, or environmental controls.
- **Environmental** — natural or man-made disasters, infrastructure outages.

## Assessment scales (qualitative / semi-quantitative)
800-30 uses five-level scales. A common qualitative mapping:

| Level | Semi-quantitative (0–10) |
|---|---|
| Very Low | 0–4 |
| Low | 5–20 |
| Moderate | 21–79 |
| High | 80–95 |
| Very High | 96–100 |

### Reference 5×5 risk matrix (likelihood × impact → risk)
| Likelihood ↓ / Impact → | Very Low | Low | Moderate | High | Very High |
|---|---|---|---|---|---|
| Very High | Very Low | Low | Moderate | High | Very High |
| High | Very Low | Low | Moderate | High | Very High |
| Moderate | Very Low | Low | Moderate | Moderate | High |
| Low | Very Low | Low | Low | Low | Moderate |
| Very Low | Very Low | Very Low | Very Low | Low | Low |

> Document whichever matrix the organization adopts during Prepare; the engine in `scripts/process.py` defaults to the table above and is configurable.

## NIST CSF 2.0 alignment
| CSF 2.0 ID | Relevance |
|---|---|
| GV.RM-01 | Risk management objectives established |
| ID.RA-01 | Vulnerabilities in assets identified |
| ID.RA-03 | Internal and external threats identified |
| ID.RA-04 | Potential impacts and likelihoods identified |
| ID.RA-05 | Threats, vulnerabilities, likelihoods, and impacts used to understand inherent risk and prioritize response |

## Risk treatment options
- **Mitigate** — implement/strengthen controls (re-score residual risk).
- **Transfer** — insurance or contractual shift.
- **Avoid** — discontinue the risk-generating activity.
- **Accept** — document residual risk with an authorizing signature within risk tolerance.

===== END FILE =====


===== FILE: skills/conducting-cyber-risk-assessment-with-nist-800-30/scripts/process.py =====
#!/usr/bin/env python3
"""
NIST SP 800-30 Rev 1 risk-register scoring engine.

Reads a risk-input JSON file describing threat events with their assessed
likelihood and impact, computes a risk level for each using the 800-30 5x5
reference matrix (configurable), ranks the register highest-risk-first, and
emits a Markdown risk register plus a top-risks summary.

Input JSON shape:
{
  "assessment": {
    "name": "Q2 FY26 Enterprise Risk Assessment",
    "scope": "Internet-facing application tier",
    "tier": 3
  },
  "matrix": {                      # OPTIONAL - overrides the default 800-30 matrix
    "High": {"Very High": "Very High", ...}
  },
  "risks": [
    {
      "id": "R-01",
      "threat_event": "Adversary phishes credentials and moves laterally",
      "threat_source": "Adversarial",
      "asset": "Customer portal / identity provider",
      "attack_techniques": ["T1566", "T1021"],
      "likelihood": "High",
      "impact": "Very High",
      "vulnerabilities": ["No phishing-resistant MFA", "Flat internal network"],
      "treatment": "Mitigate",
      "owner": "IAM Lead",
      "residual_risk": "Moderate"
    }
  ]
}

Likelihood / impact values must be one of:
  Very Low | Low | Moderate | High | Very High

Usage:
  python process.py --input risks.json [--output register.md] [--fail-on High]
  python process.py --print-matrix
"""

import argparse
import json
import sys

LEVELS = ["Very Low", "Low", "Moderate", "High", "Very High"]
RANK = {lvl: i for i, lvl in enumerate(LEVELS)}

# 800-30 Rev 1 reference 5x5 matrix: MATRIX[likelihood][impact] -> risk level
MATRIX = {
    "Very High": {"Very Low": "Very Low", "Low": "Low", "Moderate": "Moderate", "High": "High", "Very High": "Very High"},
    "High":      {"Very Low": "Very Low", "Low": "Low", "Moderate": "Moderate", "High": "High", "Very High": "Very High"},
    "Moderate":  {"Very Low": "Very Low", "Low": "Low", "Moderate": "Moderate", "High": "Moderate", "Very High": "High"},
    "Low":       {"Very Low": "Very Low", "Low": "Low", "Moderate": "Low", "High": "Low", "Very High": "Moderate"},
    "Very Low":  {"Very Low": "Very Low", "Low": "Very Low", "Moderate": "Very Low", "High": "Low", "Very High": "Low"},
}


def validate_level(value, field, rid):
    if value not in LEVELS:
        raise ValueError(
            f"risk {rid}: {field} '{value}' is not a valid 800-30 level "
            f"(expected one of {LEVELS})"
        )
    return value


def score_risk(risk, matrix):
    rid = risk.get("id", "?")
    likelihood = validate_level(risk.get("likelihood"), "likelihood", rid)
    impact = validate_level(risk.get("impact"), "impact", rid)
    level = matrix[likelihood][impact]
    return level


def render_markdown(data, matrix):
    meta = data.get("assessment", {})
    risks = data.get("risks", [])

    scored = []
    for r in risks:
        level = score_risk(r, matrix)
        scored.append((r, level))

    # rank highest risk first; tie-break by impact then likelihood
    scored.sort(
        key=lambda rl: (
            RANK[rl[1]],
            RANK[rl[0].get("impact", "Very Low")],
            RANK[rl[0].get("likelihood", "Very Low")],
        ),
        reverse=True,
    )

    lines = []
    title = meta.get("name", "Risk Assessment")
    lines.append(f"# Risk Register - {title}")
    lines.append("")
    if meta.get("scope"):
        lines.append(f"- **Scope:** {meta['scope']}")
    if meta.get("tier"):
        lines.append(f"- **Risk-management tier (SP 800-39):** Tier {meta['tier']}")
    lines.append(f"- **Risks assessed:** {len(scored)}")
    lines.append("")
    lines.append("Risk level computed from likelihood x impact on the NIST SP 800-30 Rev 1 5x5 matrix.")
    lines.append("")

    # main register
    header = (
        "| ID | Threat event | Source | Asset | ATT&CK | Likelihood | Impact "
        "| Risk | Key vulnerabilities | Treatment | Residual | Owner |"
    )
    sep = "|" + "---|" * 12
    lines.append(header)
    lines.append(sep)
    for r, level in scored:
        attck = ", ".join(r.get("attack_techniques", [])) or "-"
        vulns = "; ".join(r.get("vulnerabilities", [])) or "-"
        lines.append(
            "| {id} | {event} | {src} | {asset} | {attck} | {like} | {imp} "
            "| **{risk}** | {vulns} | {treat} | {res} | {owner} |".format(
                id=r.get("id", "-"),
                event=r.get("threat_event", "-"),
                src=r.get("threat_source", "-"),
                asset=r.get("asset", "-"),
                attck=attck,
                like=r.get("likelihood", "-"),
                imp=r.get("impact", "-"),
                risk=level,
                vulns=vulns,
                treat=r.get("treatment", "-"),
                res=r.get("residual_risk", "-"),
                owner=r.get("owner", "-"),
            )
        )

    # top risks summary
    top = [rl for rl in scored if RANK[rl[1]] >= RANK["High"]]
    lines.append("")
    lines.append("## Top risks (High and above)")
    lines.append("")
    if not top:
        lines.append("_No risks scored High or above._")
    else:
        for r, level in top:
            lines.append(f"- **{level}** - {r.get('id','-')}: {r.get('threat_event','-')} "
                         f"(impact {r.get('impact','-')}, likelihood {r.get('likelihood','-')})")

    # distribution
    dist = {lvl: 0 for lvl in LEVELS}
    for _, level in scored:
        dist[level] += 1
    lines.append("")
    lines.append("## Risk distribution")
    lines.append("")
    lines.append("| Risk level | Count |")
    lines.append("|---|---|")
    for lvl in reversed(LEVELS):
        lines.append(f"| {lvl} | {dist[lvl]} |")

    return "\n".join(lines), scored, dist


def main():
    ap = argparse.ArgumentParser(description="NIST SP 800-30 risk-register scoring engine")
    ap.add_argument("--input", "-i", help="Path to risk-input JSON")
    ap.add_argument("--output", "-o", help="Write Markdown register to this path")
    ap.add_argument("--fail-on", choices=LEVELS,
                    help="Exit non-zero if any risk scores at or above this level (for pipelines)")
    ap.add_argument("--print-matrix", action="store_true", help="Print the active risk matrix and exit")
    args = ap.parse_args()

    if args.print_matrix:
        print("NIST SP 800-30 Rev 1 reference 5x5 matrix (likelihood x impact -> risk):\n")
        hdr = "Likelihood \\ Impact | " + " | ".join(LEVELS)
        print(hdr)
        print("-" * len(hdr))
        for like in reversed(LEVELS):
            row = " | ".join(MATRIX[like][imp] for imp in LEVELS)
            print(f"{like} | {row}")
        return 0

    if not args.input:
        ap.error("--input is required unless --print-matrix is used")

    try:
        with open(args.input) as f:
            data = json.load(f)
    except (OSError, json.JSONDecodeError) as e:
        print(f"ERROR: could not read input JSON: {e}", file=sys.stderr)
        return 2

    matrix = data.get("matrix", MATRIX)

    try:
        md, scored, dist = render_markdown(data, matrix)
    except ValueError as e:
        print(f"ERROR: {e}", file=sys.stderr)
        return 2

    if args.output:
        with open(args.output, "w") as f:
            f.write(md + "\n")
        print(f"Risk register written to {args.output}", file=sys.stderr)
    else:
        print(md)

    # summary to stderr
    summary = ", ".join(f"{lvl}:{dist[lvl]}" for lvl in reversed(LEVELS) if dist[lvl])
    print(f"Scored {len(scored)} risks ({summary}).", file=sys.stderr)

    if args.fail_on:
        threshold = RANK[args.fail_on]
        breaches = [(r.get("id", "?"), lvl) for r, lvl in scored if RANK[lvl] >= threshold]
        if breaches:
            ids = ", ".join(f"{rid}={lvl}" for rid, lvl in breaches)
            print(f"FAIL: {len(breaches)} risk(s) at or above {args.fail_on}: {ids}", file=sys.stderr)
            return 1

    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/conducting-gdpr-compliance-assessment/SKILL.md =====
---
name: conducting-gdpr-compliance-assessment
description: >-
  Conduct comprehensive GDPR compliance assessments by evaluating data processing
  activities against EU Regulation 2016/679, including Article 30 records of processing,
  lawful basis validation, data subject rights implementation, Data Protection Impact
  Assessments (DPIAs) under Article 35, breach notification procedures, international
  transfer safeguards (SCCs, adequacy decisions), and technical/organizational measures
  under Article 32. Use when processing personal data of EU residents, preparing for
  supervisory authority audits, implementing privacy-by-design for new systems, scoping
  compliance gaps for M&A due diligence, assessing third-party processors, or responding
  to data subject access requests at scale. Incorporates 2026 guidance from ICO, EDPB,
  and post-Data (Use and Access) Act 2025 UK-GDPR considerations. Do not use for implementing
  specific Article 32 controls — use implementing-gdpr-data-protection-controls; or for DSAR
  automation — use implementing-gdpr-data-subject-access-request.
domain: cybersecurity
subdomain: compliance-governance
tags:
- gdpr
- data-protection
- privacy
- compliance
- dpia
- data-subject-rights
- article-30
- controller
- processor
- eu-regulation
- ico
- supervisory-authority
version: "1.0"
author: dakshverma23
license: Apache-2.0
nist_csf:
- GV.OC-02
- GV.PO-01
- GV.RM-04
- PR.DS-01
- PR.DS-02
- ID.AM-05
mitre_attack:
- T1530
- T1567
---
# Conducting GDPR Compliance Assessment

> **Effective Date**: August 2026  
> **Legal Basis**: EU Regulation 2016/679 (GDPR), UK GDPR as amended by Data Protection Act 2018 and Data (Use and Access) Act 2025 (ukpga/2025/18)  
> **Pending Changes**: Digital Omnibus proposal (COM(2025) 837) would change Article 30(5) threshold from 250 to 750 employees and Article 33 breach notification from 72h to 96h. Still in proposal stage; current requirements remain in force.

## When to Use

- When an organization **processes personal data of EU residents** (Article 3 territorial scope applies)
- When preparing for a **supervisory authority audit** (ICO, CNIL, BfDI) or responding to formal inquiry
- When implementing **privacy-by-design** requirements (Article 25) for new systems or data flows
- When **scoping compliance gaps** before M&A due diligence or contract negotiations with EU entities
- When responding to **data subject access requests (DSARs)** and discovering gaps in data inventory
- When assessing **third-party processors** for GDPR compliance before signing Data Processing Agreements (DPAs)
- After **data breach incidents** to verify notification procedures meet 72-hour requirement (Article 33)

**Do not use** for:
- **Technical implementation** of specific GDPR controls (encryption, pseudonymization, access controls) — use **implementing-gdpr-data-protection-controls** for Article 32 technical/organizational measures
- **Automated DSAR processing workflows** (identity verification, PII discovery, redaction, delivery) — use **implementing-gdpr-data-subject-access-request** for DSAR automation
- Non-EU privacy frameworks alone (CCPA, PIPEDA, LGPD); those require separate assessments with jurisdiction-specific criteria
- This skill is for **comprehensive compliance assessment** across all GDPR articles; use the specialized skills for focused implementation tasks

## Prerequisites

- Understanding of GDPR Articles 5-32 and key definitions
- Access to Article 30 records of processing activities
- Data Processing Agreements with third-party processors
- Privacy policies, consent forms, cookie notices
- Knowledge of lawful bases (Article 6)
- Data breach response plan and incident register
- List of international data transfers with safeguards

## Workflow

**For detailed procedures, templates, and examples, see `references/detailed-workflow.md`**

### Phase 1: Determine Territorial Applicability (Article 3)

GDPR applies if:
1. Organization has establishment in EU
2. Offers goods/services to EU residents
3. Monitors behavior of EU residents

**Check**: EU office? EU website targeting? Behavioral tracking?

### Phase 2: Inventory Data Processing Activities (Article 30)

Document for EACH activity:
- Controller/processor details
- Processing purposes (specific)
- Data categories and special categories (Art. 9)
- Recipients and international transfers
- Retention periods
- Security measures

**Tools**: Use `scripts/article30_parser.py`, `article30_validator.py`, `generate_ropa_report.py`

**Common gaps**: Missing retention periods (68%), vague purposes, undocumented transfers

### Phase 3: Validate Lawful Basis (Article 6)

| Basis | Use Case | Key Requirement |
|-------|----------|-----------------|
| **Consent** (6(1)(a)) | Marketing, profiling | Freely given, specific, withdrawable |
| **Contract** (6(1)(b)) | Order fulfillment | Strictly necessary only |
| **Legal Obligation** (6(1)(c)) | Tax records | Cite specific law |
| **Legitimate Interest** (6(1)(f)) | Fraud prevention, analytics | Three-part test + balancing |

**Action**: Map each Article 30 activity to one lawful basis. Document legitimate interest assessments.

### Phase 4: Assess Data Subject Rights (Articles 12-23)

Verify capability for:
- **Access** (15): Provide copy in machine-readable format within 1 month
- **Rectification** (16): Correct inaccurate data
- **Erasure** (17): "Right to be forgotten" (with exceptions)
- **Portability** (20): Transfer data in structured format
- **Objection** (21): Opt-out of legitimate interest processing
- **Automated Decision-Making** (22): Human review of algorithmic decisions

**Test**: Process sample DSAR through full workflow. Use `scripts/` for automation.

### Phase 5: Review DPIAs (Article 35)

DPIA **mandatory** for:
- Large-scale profiling with automated decisions
- Large-scale special categories processing
- Systematic monitoring of public areas (facial recognition)

**Template**: See `references/detailed-workflow.md` for complete DPIA structure

**Content**: Description, necessity, risks, mitigation, consultation (DPO, supervisory authority if novel high-risk)

### Phase 6: Audit Breach Notification (Articles 33-34)

**72-hour rule**: Notify supervisory authority within 72 hours of becoming aware of breach likely to risk rights.

**Decision tree**:
- Unencrypted SSNs stolen? → NOTIFY + notify data subjects
- Encrypted backup stolen (key secure)? → Document only
- Temporary exposure (2 hours, no financial data)? → NOTIFY authority, assess data subject notification

**Content**: Nature, categories/numbers, DPO contact, consequences, mitigation

### Phase 7: Verify International Transfers (Chapter V)

**Mechanisms**:
- Adequacy decisions (UK, Japan, etc.)
- Standard Contractual Clauses (SCCs) 2021 + Transfer Impact Assessment
- Binding Corporate Rules (BCRs)
- Derogations (Article 49 - limited)

**Post-Schrems II**: Assess destination country surveillance laws, implement supplementary measures (encryption with EU-held keys)

### Phase 8: Assess Security Measures (Article 32)

"Security appropriate to the risk":
- **Low risk**: TLS 1.2+, password hashing, access logs, patching
- **Medium risk**: AES-256 encryption, MFA, RBAC, penetration testing, SOC 2
- **High risk**: HSMs, key rotation, SIEM, bug bounty, ISO 27001

**Pseudonymization** vs. **Anonymization**: Pseudo = reversible (still personal data); Anon = irreversible (no longer GDPR)

### Phase 9: Compile Findings and Remediation Roadmap

Generate compliance report:
- Executive summary (overall status, high-priority gaps)
- Article-by-article findings
- Risk-prioritized remediation plan (Critical/High/Medium/Low)
- Cost estimates and timelines
- Responsible parties (DPO, IT, Legal, Business)

**Format**: See Output Format section below

## Key Concepts

| Term | Definition |
|------|------------|
| **Controller** | Determines purposes and means of processing (Article 4(7)) |
| **Processor** | Processes on behalf of controller (Article 4(8); requires DPA per Article 28) |
| **Personal Data** | Any information relating to identified/identifiable natural person (Article 4(1)) |
| **Special Categories** | Health, biometric, genetic, racial, political, religious, trade union, sex life data (Article 9; heightened protection) |
| **Consent** | Freely given, specific, informed, unambiguous indication of wishes (Article 4(11)) |
| **Legitimate Interest** | Lawful basis requiring three-part test: purpose, necessity, balancing (Recital 47) |
| **DPIA** | Data Protection Impact Assessment for high-risk processing (Article 35) |
| **DPO** | Data Protection Officer (Article 37; mandatory for public authorities, large-scale monitoring/special categories) |
| **SCCs** | Standard Contractual Clauses for international transfers (Commission Implementing Decision 2021/914) |
| **Supervisory Authority** | National data protection regulator (ICO for UK, CNIL for France, BfDI for Germany) |

## Tools & Systems

- **ICO Self-Assessment**: https://ico.org.uk/for-organisations/sme-web-hub/checklists/gdpr-check-list/
- **EDPB Guidelines**: https://edpb.europa.eu/our-work-tools/general-guidance_en
- **OneTrust / TrustArc**: Commercial GRC platforms with DPIA, Article 30, cookie consent modules
- **Article 30 Scripts**: `article30_parser.py`, `article30_validator.py` (included)
- **SCCs (2021)**: https://commission.europa.eu/law/law-topic/data-protection/international-dimension-data-protection/standard-contractual-clauses-scc_en
- **DPO Certification**: IAPP CIPP/E (Certified Information Privacy Professional/Europe)

## Common Scenarios

### Scenario: M&A Due Diligence

**Context**: Acquiring SaaS company with 50K EU customers. Need compliance assessment within 2 weeks.

**Approach**:
1. Request Article 30 records + DPAs with processors (AWS, Stripe, Mailchimp)
2. Validate lawful basis: Consent for marketing, Contract for service delivery
3. Check breach notification procedures (Article 33): No procedures found → HIGH RISK
4. Review international transfers: AWS US-East-1 without SCCs → BLOCKER
5. Deliverable: Gap analysis with remediation costs ($120K for SCCs + DPO hire + breach procedures)

### Scenario: Supervisory Authority Audit

**Context**: ICO formal inquiry after consumer complaint about unsubscribe not working.

**Response**:
1. Produce Article 30 records within 7 days
2. Demonstrate consent records (timestamp, version, scope)
3. Show withdrawal mechanism (unsubscribe link functional, processed within 48h)
4. Provide audit logs of DSAR/erasure requests
5. Outcome: Warning + 3-month corrective order (no fine due to cooperation)

## Output Format

```
GDPR COMPLIANCE ASSESSMENT REPORT
===================================
Organization: XYZ Corp | Assessment Date: 2026-08-24
Assessor: Jane Smith, CIPP/E | DPO: dpo@xyzcorp.com

EXECUTIVE SUMMARY
━━━━━━━━━━━━━━━━━
Overall Status: PARTIAL COMPLIANCE (67/100)
Critical Gaps: 3 | High: 5 | Medium: 8 | Low: 12

CRITICAL FINDINGS
━━━━━━━━━━━━━━━━━
1. Article 33: No breach notification procedures (72-hour deadline unmet)
2. Chapter V: International transfers to US without SCCs (Schrems II violation)
3. Article 30: Records incomplete (retention periods missing for 40% of activities)

ARTICLE-BY-ARTICLE STATUS
━━━━━━━━━━━━━━━━━━━━━━━━
✅ Article 3: Applicability confirmed (EU establishment)
⚠️  Article 6: Lawful basis documented but 3 activities use invalid bundled consent
✅ Article 15-23: DSAR procedures operational (18-day avg response time)
❌ Article 28: 40% of processors lack signed DPAs
⚠️  Article 32: Encryption at rest implemented but no MFA on admin accounts
❌ Article 33/34: No breach notification procedures
⚠️  Article 35: DPIA completed for profiling but not reviewed in 18 months
❌ Chapter V: US transfers without SCCs

REMEDIATION ROADMAP
━━━━━━━━━━━━━━━━━━━
Priority 1 (0-30 days, $50K):
  - Implement breach notification procedures + incident register
  - Execute SCCs with AWS, Stripe (Module 2)
  - Complete Article 30 records (retention periods, security measures)

Priority 2 (1-3 months, $80K):
  - Execute DPAs with remaining 8 processors
  - Deploy MFA on all admin accounts
  - Conduct legitimate interest assessments for analytics

Priority 3 (3-6 months, $40K):
  - Review and update DPIA
  - Automated DSAR response workflow
  - Annual GDPR training for staff

COMPLIANCE SCORE: 67/100 → Target 90/100 (6 months post-remediation)
```

## Verification Checklist

- [ ] Article 3 applicability determination documented
- [ ] Article 30 records complete for all activities (controller + processor roles)
- [ ] Lawful basis identified and documented for each activity
- [ ] Legitimate interest assessments documented with balancing test
- [ ] Consent mechanism is granular, withdrawable, and logged
- [ ] Data subject rights procedures operational (1-month response time)
- [ ] DPIA completed for high-risk processing (profiling, special categories, monitoring)
- [ ] Breach notification procedures documented (72-hour timeline)
- [ ] DPAs executed with all processors (Article 28 requirements)
- [ ] International transfers use SCCs 2021 + Transfer Impact Assessment
- [ ] Security measures appropriate to risk (encryption, MFA, logging, testing)
- [ ] Retention periods defined and automated deletion implemented
- [ ] Privacy policy published and updated within 12 months
- [ ] DPO designated if required (Article 37 criteria met)
- [ ] Staff trained on GDPR principles and data subject rights


===== END FILE =====


===== FILE: skills/conducting-gdpr-compliance-assessment/LICENSE =====
                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   Copyright 2026 dakshverma23

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


===== FILE: skills/conducting-gdpr-compliance-assessment/assets/compliance-scorecard.md =====
# GDPR Compliance Scorecard Template

> Use this template to assess and track GDPR compliance across all key areas.
> Replace [ORGANIZATION NAME] and fill in assessment results.

## Organization Information

- **Organization Name**: [ORGANIZATION NAME]
- **Assessment Date**: [YYYY-MM-DD]
- **Assessor**: [Name, Role]
- **Scope**: [All EU operations / Specific business unit / Specific system]
- **Next Review Date**: [YYYY-MM-DD] (recommended: annual)

---

## Executive Summary

| Metric | Score | Target | Status |
|--------|-------|--------|--------|
| Overall Compliance | __/100 | 100 | 🔴 / 🟡 / 🟢 |
| Critical Issues | __ | 0 | 🔴 / 🟡 / 🟢 |
| High Priority Gaps | __ | 0 | 🔴 / 🟡 / 🟢 |
| Medium Priority Gaps | __ | - | 🔴 / 🟡 / 🟢 |

**Compliance Status Legend**:
- 🟢 **Compliant** (≥90%): Minor gaps only
- 🟡 **Partially Compliant** (70-89%): Action required
- 🔴 **Non-Compliant** (<70%): Immediate remediation required

---

## 1. Territorial Applicability (Article 3)

| Check | Status | Evidence |
|-------|--------|----------|
| EU establishment identified? | ☐ Yes ☐ No ☐ N/A | [Location/entity] |
| Targeting EU data subjects? | ☐ Yes ☐ No ☐ N/A | [Website/marketing evidence] |
| Monitoring EU data subjects? | ☐ Yes ☐ No ☐ N/A | [Tracking/profiling activities] |
| Article 27 representative (if non-EU)? | ☐ Yes ☐ No ☐ N/A | [Representative contact] |

**Assessment**: ☐ Applies ☐ Does not apply  
**Score**: __/4

---

## 2. Article 30 Records of Processing (RoPA)

| Requirement | Status | Score | Notes |
|-------------|--------|-------|-------|
| Written RoPA exists | ☐ Yes ☐ Partial ☐ No | __/10 | [Last updated: date] |
| All processing activities documented | ☐ Yes ☐ Partial ☐ No | __/10 | [X of Y activities] |
| Purposes specified | ☐ Yes ☐ Partial ☐ No | __/5 | |
| Data subjects categorized | ☐ Yes ☐ Partial ☐ No | __/5 | |
| Personal data categories listed | ☐ Yes ☐ Partial ☐ No | __/5 | |
| Recipients documented | ☐ Yes ☐ Partial ☐ No | __/5 | |
| Retention periods specified | ☐ Yes ☐ Partial ☐ No | __/10 | [Gap: __% missing] |
| International transfers documented | ☐ Yes ☐ Partial ☐ No ☐ N/A | __/5 | |
| Security measures described | ☐ Yes ☐ Partial ☐ No | __/5 | |

**Assessment Notes**: [Key gaps identified]  
**Score**: __/55 → __% compliant

---

## 3. Lawful Basis (Article 6)

| Check | Status | Evidence |
|-------|--------|----------|
| Lawful basis identified for all processing | ☐ Yes ☐ Partial ☐ No | [X of Y activities] |
| Consent mechanisms valid (if used) | ☐ Yes ☐ Partial ☐ No ☐ N/A | [Consent tool: name] |
| Legitimate Interest Assessments (LIAs) conducted | ☐ Yes ☐ Partial ☐ No ☐ N/A | [X LIAs on file] |
| Special category data legal basis (Article 9) | ☐ Yes ☐ Partial ☐ No ☐ N/A | [Additional condition documented] |

**Common Issues Found**:
- [ ] Consent not freely given (bundled)
- [ ] "Legitimate interest" claimed without LIA
- [ ] Contract claimed for non-essential processing
- [ ] Special category data without explicit consent

**Score**: __/10

---

## 4. Data Subject Rights (Articles 12-23)

| Right | Capability | Response Time | Status |
|-------|------------|---------------|--------|
| **Right to be Informed** (Art 13-14) | Privacy notices at collection | - | ☐ ✓ ☐ ✗ |
| **Right of Access** (Art 15) | DSAR process documented | __ days (≤30 required) | ☐ ✓ ☐ ✗ |
| **Right to Rectification** (Art 16) | Correction mechanism | __ days | ☐ ✓ ☐ ✗ |
| **Right to Erasure** (Art 17) | Deletion across all systems | __ days | ☐ ✓ ☐ ✗ |
| **Right to Restrict Processing** (Art 18) | Processing pause capability | __ days | ☐ ✓ ☐ ✗ |
| **Right to Data Portability** (Art 20) | CSV/JSON export | __ days | ☐ ✓ ☐ ✗ |
| **Right to Object** (Art 21) | Marketing opt-out | Immediate | ☐ ✓ ☐ ✗ |
| **Automated Decision-Making** (Art 22) | Human review process | ☐ Yes ☐ No ☐ N/A | ☐ ✓ ☐ ✗ |

**DSAR Volume (last 12 months)**: __ requests  
**Average Response Time**: __ days  
**Score**: __/8 → __% implemented

---

## 5. Data Protection Impact Assessments (Article 35)

| Check | Status | Notes |
|-------|--------|-------|
| High-risk processing identified | ☐ Yes ☐ No | [List activities requiring DPIA] |
| DPIAs conducted for mandatory cases | ☐ Yes ☐ Partial ☐ No | [X of Y required DPIAs completed] |
| DPIAs include necessity/proportionality | ☐ Yes ☐ Partial ☐ No | |
| Risks to data subjects assessed | ☐ Yes ☐ Partial ☐ No | |
| Mitigation measures documented | ☐ Yes ☐ Partial ☐ No | |
| DPO consulted (if designated) | ☐ Yes ☐ No ☐ N/A | |
| Supervisory authority consulted (if high residual risk) | ☐ Yes ☐ No ☐ N/A | |

**Mandatory DPIA Triggers**:
- [ ] Large-scale processing of special category data
- [ ] Systematic monitoring of public areas
- [ ] Systematic extensive profiling
- [ ] Large-scale processing of biometric/genetic data

**Score**: __/7

---

## 6. Data Breach Procedures (Articles 33-34)

| Requirement | Status | Metric |
|-------------|--------|--------|
| Breach detection capability | ☐ Yes ☐ Partial ☐ No | Detection time: __ hours |
| Incident response plan documented | ☐ Yes ☐ No | [Last updated: date] |
| 72-hour notification process | ☐ Yes ☐ No | Current capability: __ hours |
| Breach register maintained (Art 33(5)) | ☐ Yes ☐ No | [X breaches in last 12 months] |
| Data subject notification process | ☐ Yes ☐ No | |
| Breach simulation/tabletop exercise | ☐ Yes ☐ No | [Last conducted: date] |

**Last Breach**: [Date or "None"]  
**Reported to Supervisory Authority**: ☐ Yes ☐ No ☐ N/A  
**Within 72 Hours**: ☐ Yes ☐ No ☐ N/A

**Score**: __/6

---

## 7. International Data Transfers (Chapter V)

| Transfer | Destination | Safeguard | Status |
|----------|-------------|-----------|--------|
| [Service/System 1] | [Country] | ☐ Adequacy ☐ SCCs ☐ BCRs ☐ None | ☐ ✓ ☐ ✗ |
| [Service/System 2] | [Country] | ☐ Adequacy ☐ SCCs ☐ BCRs ☐ None | ☐ ✓ ☐ ✗ |
| [Service/System 3] | [Country] | ☐ Adequacy ☐ SCCs ☐ BCRs ☐ None | ☐ ✓ ☐ ✗ |

**Common Transfer Destinations**:
- USA: ☐ Adequacy (Data Privacy Framework) ☐ SCCs ☐ None
- UK: ☐ Adequacy ☐ SCCs
- Other: [List countries]

**SCCs in Use**: ☐ 2021 version ☐ 2010 version (must update)  
**Transfer Impact Assessment (TIA) Conducted**: ☐ Yes ☐ No (required for high-risk countries)

**Score**: __/10

---

## 8. Technical & Organizational Measures (Article 32)

| Security Control | Implemented | Evidence |
|------------------|-------------|----------|
| Encryption at rest | ☐ Yes ☐ Partial ☐ No | [Algorithm: AES-256 / other] |
| Encryption in transit | ☐ Yes ☐ Partial ☐ No | [TLS 1.2+ / other] |
| Access control (least privilege) | ☐ Yes ☐ Partial ☐ No | [IAM tool] |
| Multi-factor authentication | ☐ Yes ☐ Partial ☐ No | [X% of users] |
| Audit logging | ☐ Yes ☐ Partial ☐ No | [Retention: X months] |
| Pseudonymization | ☐ Yes ☐ Partial ☐ No ☐ N/A | [Where implemented] |
| Backup and recovery | ☐ Yes ☐ Partial ☐ No | [RPO: __ / RTO: __] |
| Vulnerability scanning | ☐ Yes ☐ Partial ☐ No | [Frequency: quarterly / monthly] |
| Penetration testing | ☐ Yes ☐ No | [Last conducted: date] |
| Security awareness training | ☐ Yes ☐ Partial ☐ No | [X% of staff trained] |

**Score**: __/10

---

## 9. Processor Management (Article 28)

| Requirement | Status | Notes |
|-------------|--------|-------|
| List of all processors maintained | ☐ Yes ☐ Partial ☐ No | [X processors identified] |
| Data Processing Agreements (DPAs) signed | ☐ Yes ☐ Partial ☐ No | [__% coverage] |
| DPAs contain all Article 28(3) requirements | ☐ Yes ☐ Partial ☐ No | |
| Sub-processor list disclosed | ☐ Yes ☐ Partial ☐ No | |
| Sub-processor approval mechanism | ☐ Yes ☐ No | |
| Processor audits conducted | ☐ Yes ☐ No | [Last audit: date] |

**Score**: __/6

---

## 10. Data Protection Officer (Articles 37-39)

| Check | Status | Notes |
|-------|--------|-------|
| DPO designation required? | ☐ Yes ☐ No | [Public authority / large-scale / special categories] |
| DPO designated | ☐ Yes ☐ No ☐ N/A | [Name: / Contact:] |
| DPO contact published | ☐ Yes ☐ No ☐ N/A | [Privacy policy / website] |
| DPO independence ensured | ☐ Yes ☐ No ☐ N/A | [No conflict of interest] |
| DPO involved in compliance matters | ☐ Yes ☐ No ☐ N/A | |

**Score**: __/5 (or N/A if not required)

---

## Overall Compliance Score

| Category | Weight | Score | Weighted Score |
|----------|--------|-------|----------------|
| Article 30 RoPA | 20% | __% | __ |
| Lawful Basis | 15% | __% | __ |
| Data Subject Rights | 15% | __% | __ |
| DPIAs | 10% | __% | __ |
| Breach Procedures | 10% | __% | __ |
| International Transfers | 10% | __% | __ |
| Security Measures (Art 32) | 10% | __% | __ |
| Processor Management | 5% | __% | __ |
| DPO (if required) | 5% | __% | __ |
| **TOTAL** | **100%** | - | **__/100** |

**Overall Assessment**: 🔴 / 🟡 / 🟢

---

## Priority Action Items

### 🔴 Critical (Immediate - 0-30 days)
1. [Item]
2. [Item]

### 🟡 High Priority (1-3 months)
1. [Item]
2. [Item]

### 🟠 Medium Priority (3-6 months)
1. [Item]
2. [Item]

---

## Recommendations

1. **Short-term** (0-3 months):
   - [Recommendation]

2. **Medium-term** (3-6 months):
   - [Recommendation]

3. **Long-term** (6-12 months):
   - [Recommendation]

---

## Sign-off

**Assessed by**: ________________________ Date: __________

**Reviewed by (DPO)**: ________________________ Date: __________

**Approved by (Senior Management)**: ________________________ Date: __________

---

*This scorecard provides a snapshot of GDPR compliance status. It should be reviewed and updated at least annually or when significant changes occur in data processing activities.*

===== END FILE =====


===== FILE: skills/conducting-gdpr-compliance-assessment/references/api-reference.md =====
# API Reference: GDPR Compliance Tools and Commands

## Python GDPR Compliance Helper Commands

### Article 30 Parser
```bash
# Parse DPAs and privacy policies for Article 30 fields
python scripts/article30_parser.py --input contracts/processors/ --output ropa.json

# Validate completeness against mandatory fields
python scripts/article30_validator.py --ropa ropa.json --check-retention --check-transfers

# Generate Article 30 register in supervisory authority format
python scripts/generate_ropa_report.py --input ropa.json --output Article30_Register.pdf
```

## Data Subject Access Request (DSAR) Automation

### Open Data Rights (ODR)
```bash
# Initialize ODR project
npm install @opendatarights/odr-core

# Submit DSAR programmatically
node scripts/submit_dsar.js --email user@example.com --company "Acme Corp"

# Check DSAR status
node scripts/check_dsar_status.js --request-id abc123
```

## GDPR Compliance Scanning Tools

### BigID Data Discovery
```bash
# Scan for personal data across data sources
bigid-cli scan --source "s3://company-bucket" --classification PII

# Generate data inventory report
bigid-cli report --format json --output data_inventory.json
```

### OneTrust Cookie Consent API
```bash
# Verify cookie consent compliance
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.onetrust.com/v1/consent/categories" | jq

# Check GDPR consent records
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.onetrust.com/v1/consent/receipts?user=john@example.com"
```

## Standard Contractual Clauses (SCCs) Templates

| SCC Module | Use Case | Download |
|------------|----------|----------|
| Module One | Controller to Controller | https://commission.europa.eu/law/law-topic/data-protection/international-dimension-data-protection/standard-contractual-clauses-scc_en |
| Module Two | Controller to Processor | Same link (2021 version) |
| Module Three | Processor to Processor | Same link |
| Module Four | Processor to Controller | Same link |

## Data Processing Agreement (DPA) Checklist

| Article 28 Requirement | DPA Clause Reference |
|------------------------|----------------------|
| Subject matter and duration | § 1.1 |
| Nature and purpose | § 1.2 |
| Type of personal data | § 1.3, Annex A |
| Categories of data subjects | § 1.4, Annex A |
| Controller obligations and rights | § 2 |
| Processor obligations | § 3-8 |
| Sub-processor approval | § 9 |
| Data subject rights assistance | § 10 |
| Security measures | § 11, Annex B |
| Breach notification | § 12 |
| Deletion/return of data | § 13 |
| Audits | § 14 |
| Liability and indemnity | § 15-16 |

## DPIA (Data Protection Impact Assessment) Template Structure

```yaml
# DPIA for [Processing Activity Name]
date: 2026-03-15
version: 1.0
assessor: [Name, Role]

1. Description of Processing:
  - purpose:
  - categories_of_data:
  - categories_of_subjects:
  - retention_period:
  - recipients:

2. Necessity and Proportionality:
  - necessity_justification:
  - proportionality_assessment:

3. Risks to Data Subject Rights:
  - risk_1:
      description:
      likelihood: [low/medium/high]
      severity: [low/medium/high]
      risk_level: [likelihood × severity]
  - risk_2: ...

4. Mitigation Measures:
  - measure_1:
      description:
      effectiveness: [reduces risk to ...]
      responsibility: [who implements]
  - measure_2: ...

5. Residual Risk:
  - residual_risk_assessment:
  - supervisory_authority_consultation: [yes/no, if yes why]

6. Sign-off:
  - dpo_review_date:
  - dpo_approval:
  - controller_approval:
```

## Breach Notification API (Supervisory Authority)

### ICO Data Security Incident Reporting Tool
```bash
# UK: Report breach to ICO
# https://ico.org.uk/for-organisations/report-a-breach/

# EU Member State: Check your supervisory authority
# https://edpb.europa.eu/about-edpb/about-edpb/members_en
```

### Breach Register Template
```json
{
  "breach_id": "BR-2026-001",
  "discovered_date": "2026-03-10T14:30:00Z",
  "notification_date_authority": "2026-03-12T10:00:00Z",
  "72_hour_deadline": "2026-03-13T14:30:00Z",
  "within_deadline": true,
  "affected_data_subjects": 1500,
  "categories_of_data": ["email", "name", "payment_card_last4"],
  "likely_consequences": "Risk of phishing targeting affected users",
  "measures_taken": "Mandatory password reset, notification sent to all users, MFA enforced",
  "supervisory_authority": "ICO",
  "notification_reference": "ICO-BR-2026-12345"
}
```

## GDPR-Compliant Logging Commands

### Audit Log Retention Check
```bash
# Verify retention periods align with Article 30 documented periods
grep -r "retention" Article30_Register.json | jq

# Check if logs contain personal data (must be protected under Article 32)
grep -iE "(email|name|ip_address)" /var/log/app/*.log
```

### Pseudonymization Example (Python)
```python
import hashlib
import hmac

def pseudonymize(email, secret_key):
    """Pseudonymize email using HMAC-SHA256"""
    return hmac.new(
        secret_key.encode(),
        email.encode(),
        hashlib.sha256
    ).hexdigest()

# Usage
secret = "your-secret-key-store-securely"
pseudonym = pseudonymize("user@example.com", secret)
# Output: 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8
```

## Data Subject Rights Verification

| Right | Verification Check | Command/Tool |
|-------|-------------------|--------------|
| Right to Access (Art 15) | Can org deliver all data within 1 month? | Test DSAR workflow end-to-end |
| Right to Erasure (Art 17) | Can org delete across all systems? | `python scripts/test_deletion.py --user-id 12345` |
| Right to Portability (Art 20) | Can org export in machine-readable format? | Verify CSV/JSON export functionality |
| Right to Object (Art 21) | Can org stop direct marketing? | Test unsubscribe mechanism |

## International Transfer Checklist

```bash
# Identify all non-EU/EEA data flows
grep -r "transfer_destination" Article30_Register.json | grep -v "EU\|EEA"

# Verify safeguards for each transfer
for dest in $(jq -r '.transfers[].destination_country' Article30_Register.json); do
  echo "Transfer to: $dest"
  echo "Safeguard: $(jq -r ".transfers[] | select(.destination_country==\"$dest\") | .safeguard" Article30_Register.json)"
done
```

## References

- **ICO Self-Assessment Tool**: https://ico.org.uk/for-organisations/sme-web-hub/checklists/self-assessment/
- **EDPB Interactive Tool**: https://ec.europa.eu/info/law/law-topic/data-protection/reform/rights-citizens/how-my-personal-data-protected_en
- **GDPR.eu Compliance Checklist**: https://gdpr.eu/checklist/
- **NIST Privacy Framework**: https://www.nist.gov/privacy-framework


===== END FILE =====


===== FILE: skills/conducting-gdpr-compliance-assessment/references/detailed-workflow.md =====
# Detailed GDPR Compliance Assessment Workflow

## Phase 2: Article 30 Records Template (Extended)

### Small Company Exemption Analysis

Companies <250 employees are exempt from Article 30 ONLY if:
- Processing is occasional (NOT true for regular customer/employee data)
- Processing is not high-risk
- Processing excludes special categories (health, biometric, genetic, etc.)

**In practice**: This exempts almost no SaaS, e-commerce, HR, or B2C operators.

### Article 30 Fields (Complete)

```csv
Activity ID,Purpose,Legal Basis,Data Categories,Data Subjects,Recipients,Retention,Security Measures,DPO Contact
PROC-001,Customer relationship management,Contract (Art 6(1)(b)),Name|Email|Phone|Company|Job Title,B2B customers,Salesforce (processor)|Marketing team,5 years after contract end,TLS 1.3|AES-256|MFA|Access logs,dpo@company.com
PROC-002,Employee payroll processing,Legal obligation (Art 6(1)(c)),Name|SSN|Bank details|Tax info|Salary,Employees,ADP (processor)|Tax authority|Bank,7 years (legal requirement),AES-256|RBAC|Audit logs,dpo@company.com
PROC-003,Website analytics,Legitimate interest (Art 6(1)(f)),IP address|Browser|Pages viewed|Session duration,Website visitors,Google Analytics (processor),26 months,Pseudonymization|Cookies,dpo@company.com
```

### Common Article 30 Gaps

1. **Missing fields**: 47% of controllers omit "data retention period" (Article 30(1)(f))
2. **Vague purposes**: "Business operations" does not meet specificity requirement
3. **Processor confusion**: Article 28 DPAs not cross-referenced in Article 30 records
4. **International transfers**: No documentation of safeguards for non-EU processors

## Phase 3: Lawful Basis Decision Tree (Detailed)

### Consent (Article 6(1)(a))

**When to use**:
- Processing is optional (not required for service delivery)
- Can demonstrate freely given, specific, informed, unambiguous affirmative action
- Withdrawal mechanism is as easy as giving consent

**Red flags**:
- "By continuing to use our service" ≠ valid consent
- Pre-ticked boxes ≠ valid consent
- Bundled consent (accept all or lose service) ≠ freely given

**Verification checklist**:
```
[ ] Consent request uses plain language (Article 7(2))
[ ] Consent is granular (separate boxes for marketing, analytics, third-party sharing)
[ ] Withdrawal button/link is visible on same page as original consent
[ ] Consent records include: timestamp, version, scope, withdrawal mechanism
[ ] Children <16 require parental consent for online services (Article 8)
```

### Contract (Article 6(1)(b))

**When to use**:
- Processing is strictly necessary to fulfill a contract with the data subject
- Examples: Processing shipping address to deliver purchased goods, processing payment details for subscription service

**Red flags**:
- "Necessary for contract" claimed for ancillary marketing activities (not true)
- Contract basis for cookies/analytics (usually legitimate interest, not contract)

### Legitimate Interest (Article 6(1)(f))

**Three-part test** (WP29 Opinion 06/2014):
1. **Purpose test**: Is the interest real and present (not speculative)?
2. **Necessity test**: Is processing necessary, or could you use less intrusive means?
3. **Balancing test**: Do data subject rights override your legitimate interest?

**Common legitimate interests**:
- Fraud prevention and network security (Recital 49)
- Direct marketing to existing customers (Recital 47, but with objection right)
- Intra-group administrative transfers (Recital 48)

**Documentation required**:
```markdown
## Legitimate Interest Assessment: Website Analytics

**Purpose**: Improve website user experience and detect technical errors

**Necessity**: 
- Analytics data (pages viewed, session duration) necessary to identify UX friction
- Pseudonymized IP addresses sufficient (full IPs not retained)
- Alternative (user surveys) would not provide real-time technical error detection

**Balancing**:
- Data subjects: Website visitors, minimal expectation of privacy for navigational data
- Impact: Low (no profiling, no automated decisions, no special categories)
- Safeguards: IP pseudonymization, 26-month retention, opt-out via cookie banner

**Conclusion**: Legitimate interest established, objection right provided in privacy policy
```

## Phase 4: Data Subject Rights Response Times

### Article 12(3) Timeline

**Default**: Respond within **one month** of receipt

**Extension**: May extend by **two further months** if complex or numerous requests

**Requirements**:
- Must inform data subject of extension within original one-month period
- Must explain reason for extension

### 2025 ICO Statistics

- **Average DSAR response time**: 18 days
- **Compliance rate**: 73% responded within one month
- **Most common delays**: Distributed data across multiple systems (42%), identity verification disputes (28%), unclear request scope (19%)

### Technical Implementation Requirements

1. **Identity verification** (Article 12(6))
   - Request additional information if doubt about identity
   - But: Cannot ask for excessive ID documents
   - Passport/driving license photocopy is standard practice

2. **Response format** (Article 15(3))
   - "Commonly used electronic format" if requested electronically
   - CSV, PDF, or API access (JSON) are acceptable
   - Must be machine-readable (scanned paper ≠ compliant)

3. **Fee exceptions** (Article 12(5))
   - First request: Free
   - Subsequent requests: May charge "reasonable fee based on administrative costs" if manifestly unfounded or excessive
   - ICO threshold: £10-50 for excessive requests (not first requests)

4. **Logging requirements** (Article 30(2) + accountability)
   - Log all DSARs received (date, scope, response deadline)
   - Log verification method used
   - Log data delivered (systems queried, records provided)
   - Retention: 6 years (evidence of compliance)

## Phase 5: DPIA Templates (Detailed)

### Article 35(3) Mandatory DPIA Triggers

A DPIA is **mandatory** when processing:

1. **Systematic and extensive profiling** with automated decision-making producing legal/similarly significant effects
   - Example: Credit scoring determining loan eligibility
   - Example: AI-driven employee performance evaluation affecting termination decisions

2. **Large-scale processing of special categories** (Article 9) or criminal conviction data (Article 10)
   - "Large-scale" factors: number of data subjects, volume of data, duration, geographical extent
   - Example: Health insurer processing 500,000+ patient records
   - Example: National background check service processing criminal records

3. **Systematic monitoring of publicly accessible areas on a large scale**
   - Example: CCTV with facial recognition in shopping malls
   - NOT required: Single-location CCTV without biometric processing

### DPIA Template Sections

```markdown
## 1. Description of Processing Operation

**Name**: AI-Powered Recruitment Screening System
**Controller**: XYZ Corp
**DPO Contact**: dpo@xyzcorp.com
**Processing purpose**: Automatically screen CVs, rank candidates, flag high-potential applicants
**Legal basis**: Legitimate interest (efficient recruitment)
**Data categories**: Name, CV content (education, employment history, skills), LinkedIn profile (if provided)
**Special categories**: None processed (no health, ethnicity, political views extracted)
**Data subjects**: Job applicants
**Recipients**: HR team, hiring managers
**Retention**: 6 months after recruitment process ends (unless consent obtained for future roles)
**International transfers**: None (all processing within EU)

## 2. Necessity and Proportionality

**Is processing necessary for the stated purpose?**
Yes — automated screening of 2,000+ applications per role is impractical manually; human review of flagged candidates follows AI screening.

**Could you achieve the purpose with less intrusive means?**
Considered: Keyword-only matching (less effective, misses semantic skills). Could reduce data retention to 3 months (implemented).

**Is data minimization applied?**
Yes — System does NOT process: age, photo, address (only city for location-based roles), gender, marital status.

## 3. Risks to Rights and Freedoms

| Risk | Likelihood | Severity | Impact |
|------|-----------|----------|---------|
| Algorithmic bias (gender, ethnicity) leading to discriminatory screening | Medium | High | Discrimination in employment decisions violates GDPR Article 22 and Equality Act 2010 |
| Data breach exposing applicant CVs | Low | Medium | Reputational harm, identity theft if combined with contact details |
| Lack of transparency in AI decision logic | High | Medium | Applicants cannot effectively challenge decisions (Article 22(3)) |

## 4. Measures to Address Risks

**Risk 1 mitigation**:
- Bias audit every 6 months (test for demographic parity, equal opportunity)
- Training data balanced across protected characteristics
- Human review of all "rejected" candidates (AI only flags "proceed to interview")

**Risk 2 mitigation**:
- AES-256 encryption at rest, TLS 1.3 in transit
- Access control: HR team only (RBAC with audit logs)
- Penetration test annually

**Risk 3 mitigation**:
- Privacy policy explains AI screening in plain language
- Applicants can request human review of rejection (Article 22(3))
- Provide explanation of key factors in AI decision upon request

## 5. Consultation

**Consulted parties**:
- DPO: Reviewed and approved DPIA (Article 35(2))
- ICO: Not required (no novel high-risk processing; standard AI recruitment within EDPB guidelines)
- Data subjects: Privacy policy updated to describe AI screening; feedback channel provided

**Date completed**: 2026-03-15
**Next review**: 2027-03-15 or upon material change to processing
```

## Phase 6: Breach Notification Decision Tree

### "Likely to Result in Risk" Assessment (Article 33)

**Notify supervisory authority within 72 hours** if breach is likely to result in risk to rights.

**Risk factors** (WP29 Guidelines WP250rev.01):
- Type of breach: Confidentiality (unauthorized access), availability (data loss/destruction), integrity (unauthorized modification)
- Nature of personal data: Special categories (Article 9) = high risk
- Ease of identification: Encrypted data breached = lower risk if key not compromised
- Severity of consequences: Financial loss, discrimination, reputational damage, identity theft
- Special characteristics: Children, vulnerable individuals = higher risk
- Number of affected individuals: 10 individuals with special categories = higher risk than 1,000 individuals with generic email addresses

### Example: Notify or Not?

**Scenario 1**: Laptop stolen containing 500 employee records (names, salaries, SSNs)
- **Decision**: **NOTIFY** (72 hours)
- **Rationale**: Unencrypted SSNs = high risk of identity theft
- **Article 34**: Also notify data subjects without undue delay

**Scenario 2**: Marketing database backup (encrypted AES-256) stolen, but encryption key NOT compromised
- **Decision**: **Do NOT notify** (unless key later compromised)
- **Rationale**: Encrypted data without key poses minimal risk
- **Documentation**: Internal breach log maintained (Article 33(5))

**Scenario 3**: Technical error causes customer order history (product names, dates, no financials) to be visible to other customers for 2 hours before fix
- **Decision**: **NOTIFY** (72 hours)
- **Rationale**: Confidentiality breach affecting 1,200 customers; low individual risk but large scale triggers reporting
- **Article 34**: Low individual risk = notification not required to data subjects

### Breach Notification Content (Article 33(3))

**Minimum required fields**:
1. Nature of breach (confidentiality, availability, integrity)
2. Categories and approximate number of data subjects
3. Categories and approximate number of records
4. Name and contact of DPO or other contact point
5. Likely consequences of the breach
6. Measures taken or proposed to address the breach and mitigate adverse effects

**Template**:
```
To: ICO Breach Notification (via online form)
Date: 2026-08-24 14:30 UTC

Breach Reference: BREACH-2026-0824
Reporting Organization: XYZ Corp (ICO Registration: Z1234567)
DPO Contact: dpo@xyzcorp.com, +44 20 1234 5678

Nature of Breach: Confidentiality - Unauthorized access via compromised admin credential

Data Subjects Affected: Approximately 1,200 customers
Records Compromised: Names, email addresses, order history (product names, dates, amounts), shipping addresses

Breach Discovery: 2026-08-24 12:00 UTC (monitoring alert for unusual admin database queries)
Breach Occurrence: Estimated 2026-08-23 22:00 UTC to 2026-08-24 11:30 UTC (13.5 hours)

Likely Consequences: Low individual risk - no financial details or passwords compromised; moderate risk of targeted phishing using order history

Measures Taken:
- Compromised credential revoked 2026-08-24 11:30 UTC
- All admin credentials force-reset
- MFA enforcement deployed
- Forensic analysis initiated (external consultant engaged)

Measures Proposed:
- Customer notification (Article 34) by email within 48 hours describing breach, advising phishing vigilance
- SOC 2 Type 2 audit brought forward to Q3 2026
- Penetration test scheduled for 2026-09-15
```

## Phase 7: International Transfer Mechanisms (Extended)

### Standard Contractual Clauses (SCCs) 2021

**When to use**:
- Transferring personal data to non-EU country without adequacy decision
- Most common mechanism post-Schrems II

**Implementation**:
1. Download SCCs from European Commission website
2. Choose appropriate module:
   - Module 1: Controller to Controller
   - Module 2: Controller to Processor
   - Module 3: Processor to Processor
   - Module 4: Processor to Controller

3. Complete Annex I (parties, data subjects, data categories, special categories, processing purpose, retention)
4. Complete Annex II (technical/organizational measures — reference Article 32 controls)
5. Complete Annex III (sub-processors list)

6. **Schrems II compliance**:
   - Assess laws of destination country (do government surveillance laws permit access to personal data?)
   - Document assessment in Annex (new requirement)
   - Implement supplementary measures if country laws undermine SCCs (e.g., additional encryption)

**Red flags**:
- Using pre-2021 SCCs (invalid as of December 27, 2022)
- Failing to assess destination country laws (Schrems II requirement)
- Generic Annex II measures not tailored to actual data transferred

### Adequacy Decisions (Current 2026)

**Countries with adequacy**:
- Andorra, Argentina, Canada (commercial), Faroe Islands, Guernsey, Israel, Isle of Man, Japan, Jersey, New Zealand, South Korea, Switzerland, United Kingdom, Uruguay

**United States**: No blanket adequacy (Schrems I invalidated Safe Harbor, Schrems II invalidated Privacy Shield)
- Use SCCs for US transfers
- Exception: Data Privacy Framework (DPF) certification (launched 2023) — US companies can self-certify for adequacy

### Transfer Impact Assessment (TIA)

**EDPB Recommendations 01/2020** require Transfer Impact Assessment before relying on SCCs:

```markdown
## Transfer Impact Assessment: AWS US-East-1 Processing

**Destination country**: United States
**Recipient**: Amazon Web Services Inc. (DPF-certified)

**Step 1: Map Data Transfer**
- Personal data: Customer names, emails, encrypted payment tokens
- Transfer purpose: Cloud hosting of SaaS application
- Transfer mechanism: SCCs Module 2 (Controller to Processor)

**Step 2: Assess Destination Country Laws**
- FISA Section 702: Permits NSA access to communications of non-US persons
- CLOUD Act: Permits US law enforcement to compel data disclosure
- Risk: Payment tokens are encrypted with EU-held keys (inaccessible to US authorities)

**Step 3: Evaluate Supplementary Measures**
- Encryption: AES-256 with keys held in EU (AWS KMS EU region)
- Pseudonymization: Customer IDs are pseudonymized UUIDs
- Contractual: AWS DPF certification + SCCs
- Organizational: AWS resists overbroad government requests (transparency report published)

**Step 4: Conclusion**
Transfer may proceed. Encryption keys held exclusively in EU provide supplementary measure ensuring US government access would yield only encrypted data. No FISA 702 precedent for compelling decryption keys held outside US jurisdiction.

**Review date**: 2027-08-24 or upon change to US surveillance laws
```

## Phase 8: Article 32 Security Measures (Extended)

### Security Appropriate to the Risk

**Low Risk** (generic non-sensitive data, small scale):
- TLS 1.2+ for data in transit
- Password hashing (bcrypt, Argon2)
- Access logging
- Regular patching

**Medium Risk** (larger scale, potential for harm):
- AES-256 encryption at rest
- MFA for admin accounts
- RBAC with least privilege
- Annual penetration testing
- SOC 2 Type 2 audit

**High Risk** (special categories, large scale, children, automated decision-making):
- Hardware security modules (HSMs) for key management
- Encryption key rotation every 90 days
- Continuous security monitoring (SIEM)
- Bug bounty program
- ISO 27001 certification
- Regular DPIA reviews

### Pseudonymization vs. Anonymization

**Pseudonymization** (Article 4(5)):
- Replaces identifiers with pseudonyms (e.g., hash, UUID)
- Re-identification **possible** with additional information (kept separately)
- Still considered personal data under GDPR
- Recommended security measure (Article 32(1)(a))

**Anonymization**:
- Removes all identifiers such that re-identification is **not possible**
- No longer personal data (Recital 26)
- GDPR no longer applies
- Hard to achieve in practice (linkage attacks, de-anonymization research)

**Example**:
- Original: John Doe, john@example.com, IP 192.168.1.1
- Pseudonymized: User-abc123, hashed-email-xyz, IP *.*.1.1 (retain last octet for geolocation)
- Anonymized: Aggregated count "500 users from London accessed feature X" (no individual records)

### Data Retention Limits (Article 5(1)(e))

**No fixed retention periods** in GDPR (unlike CCPA's 12 months for sale opt-outs). Controller must define and justify retention.

**Common justifications**:
- **Contract fulfillment**: Retain customer data for duration of contract + reasonable period for warranty claims (e.g., 2 years)
- **Legal obligation**: Tax records 7 years, employment records per national labor law
- **Legitimate interest**: Marketing data retained while interest remains (typically 2-3 years of no interaction = delete)

**Best practice**:
- Document retention schedule in Article 30 records
- Implement automated deletion workflows (not "we'll delete if requested")
- "Retention period review" in annual GDPR compliance audit

### Automated Deletion Script Example

```python
# automated_deletion.py - GDPR Article 5(1)(e) Retention Limits
import psycopg2
from datetime import datetime, timedelta

# Configuration: Retention policies (days)
RETENTION_POLICIES = {
    "marketing_leads": 730,      # 2 years (legitimate interest)
    "customer_orders": 2555,     # 7 years (legal obligation - tax)
    "support_tickets": 1095,     # 3 years (contract fulfillment)
    "website_analytics": 780     # 26 months (legitimate interest)
}

def delete_expired_data(table, retention_days):
    cutoff_date = datetime.now() - timedelta(days=retention_days)
    
    conn = psycopg2.connect("dbname=gdpr_app user=app_user password=secret")
    cursor = conn.cursor()
    
    # Log deletion for Article 30 compliance
    cursor.execute(f"""
        INSERT INTO gdpr_deletion_log (table_name, deletion_date, records_deleted)
        SELECT '{table}', NOW(), COUNT(*)
        FROM {table}
        WHERE created_at < %s
    """, (cutoff_date,))
    
    # Perform deletion
    cursor.execute(f"DELETE FROM {table} WHERE created_at < %s", (cutoff_date,))
    
    conn.commit()
    cursor.close()
    conn.close()
    
    print(f"[{datetime.now()}] Deleted records from {table} older than {cutoff_date}")

if __name__ == "__main__":
    for table, retention in RETENTION_POLICIES.items():
        delete_expired_data(table, retention)
```

**Schedule**: Run daily via cron (0 2 * * * /usr/bin/python3 /opt/gdpr/automated_deletion.py)

===== END FILE =====


===== FILE: skills/conducting-gdpr-compliance-assessment/references/standards.md =====
# Standards and References — Conducting GDPR Compliance Assessment

## NIST Cybersecurity Framework 2.0

| ID | Category | Rationale |
|----|----------|-----------|
| GV.OC-02 | Organizational Context: Legal, regulatory, and contractual requirements regarding cybersecurity are understood and managed | GDPR is a primary legal/regulatory requirement for organizations processing EU resident data; understanding its applicability (Article 3) and requirements is foundational to governance. |
| GV.PO-01 | Policy: Organizational cybersecurity policy is established, communicated, and enforced | GDPR Article 5 accountability principle and Article 24 controller responsibilities require documented data protection policies aligned with GDPR principles. |
| GV.RM-04 | Risk Management Strategy: Strategic direction that describes appropriate risk response options is established and communicated | DPIAs (Article 35) and controller accountability (Article 24) mandate risk-based approaches to data protection; risk tolerance must be documented and communicated. |
| PR.DS-01 | Data Security: Data-at-rest is protected | Article 32 requires encryption and pseudonymization as technical measures appropriate to the risk; data-at-rest encryption directly implements this. |
| PR.DS-02 | Data Security: Data-in-transit is protected | Article 32 confidentiality/integrity; TLS 1.2+ for all personal data transmission implements this security baseline. |
| ID.AM-05 | Asset Management: Resources are prioritized based on classification, criticality, and business value | Article 30 records of processing categorize personal data by sensitivity (special category under Article 9 vs. general); this informs security prioritization. |

## MITRE ATT&CK

| Technique ID | Name | Tactic | Rationale |
|--------------|------|--------|-----------|
| T1530 | Data from Cloud Storage Object | Collection | Unauthorized access to cloud-stored personal data represents a data breach under GDPR Article 4(12); assessing cloud security controls (access logs, IAM) validates Article 32 compliance. |
| T1567 | Exfiltration Over Web Service | Exfiltration | Data exfiltration scenarios (T1567.002 to cloud, T1567.004 over encrypted channel) constitute personal data breaches; GDPR Article 33 breach notification and Article 32 exfiltration prevention controls directly address this. |

## GDPR Articles Mapped

| Article | Title | Assessment Focus |
|---------|-------|------------------|
| Article 3 | Territorial Scope | Determine applicability: establishment, offering goods/services, monitoring |
| Article 4 | Definitions | Controller, processor, personal data, special category data, consent |
| Article 5 | Principles | Lawfulness, fairness, transparency, purpose limitation, data minimization, accuracy, storage limitation, integrity/confidentiality, accountability |
| Article 6 | Lawfulness of Processing | Identify lawful basis for each processing activity (consent, contract, legal obligation, vital interests, public task, legitimate interest) |
| Article 7 | Conditions for Consent | Verify consent is freely given, specific, informed, unambiguous; withdrawal mechanism |
| Article 9 | Special Categories | Additional conditions for processing health, biometric, genetic, racial, political, religious, trade union, sex life data |
| Article 12-23 | Data Subject Rights | Access (15), rectification (16), erasure (17), restriction (18), portability (20), objection (21), automated decision-making (22) |
| Article 24 | Controller Responsibility | Implement technical/organizational measures demonstrating compliance; accountability |
| Article 25 | Privacy by Design | Data protection by design and by default |
| Article 28 | Processor Obligations | Data Processing Agreement requirements, sub-processor rules |
| Article 30 | Records of Processing | Written register of all processing activities (mandatory for orgs 250+ employees or high-risk) |
| Article 32 | Security of Processing | Encryption, pseudonymization, confidentiality, integrity, availability, resilience, regular testing |
| Article 33 | Breach Notification (Authority) | Notify supervisory authority within 72 hours unless unlikely to risk rights |
| Article 34 | Breach Notification (Data Subject) | Notify individuals without undue delay if high risk to rights |
| Article 35 | Data Protection Impact Assessment | Mandatory for systematic large-scale processing, special categories, monitoring public areas |
| Article 37-39 | Data Protection Officer | Designation criteria, tasks, independence |
| Articles 44-50 | International Transfers | Chapter V: adequacy decisions, SCCs, BCRs, derogations |

## Supporting Standards

- **ISO/IEC 27701:2019** — Privacy Information Management System (PIMS); extension to ISO 27001 for GDPR-style privacy
- **ISO/IEC 29134:2017** — Privacy Impact Assessment (PIA) methodology; aligns with GDPR Article 35 DPIA
- **NIST Privacy Framework 1.0** — Risk-based approach to privacy; complements GDPR compliance programs
- **EDPB Guidelines** — European Data Protection Board authoritative guidance on GDPR interpretation

## Official Resources

- **GDPR Full Text**: https://gdpr.eu/tag/gdpr/
- **ICO GDPR Guidance** (UK Supervisory Authority): https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/
- **EDPB Guidelines**: https://edpb.europa.eu/our-work-tools/general-guidance_en
- **Standard Contractual Clauses (2021)**: https://commission.europa.eu/law/law-topic/data-protection/international-dimension-data-protection/standard-contractual-clauses-scc_en
- **Article 29 Working Party Opinions** (superseded by EDPB but still referenced): https://ec.europa.eu/justice/article-29/documentation/opinion-recommendation/index_en.htm
- **Data (Use and Access) Act 2025** (UK, Royal Assent 19 June 2025): https://www.legislation.gov.uk/ukpga/2025/18

## Enforcement Statistics (2024-2026)

- **Total fines issued**: €4.8 billion (2024-2025 period)
- **Largest fine**: €1.2 billion (Meta Ireland, data transfer violations)
- **Most common violations**: Insufficient legal basis (28%), inadequate security (Article 32, 22%), failure to implement data subject rights (18%)
- **Average DSAR response time**: 18 days (requirement: within 1 month, Article 15)
- **Breach notification compliance**: 41% of controllers notified ICO within 72 hours (2025 ICO report)


===== END FILE =====


===== FILE: skills/conducting-gdpr-compliance-assessment/scripts/article30_parser.py =====
#!/usr/bin/env python3
"""
GDPR Article 30 Records of Processing Activities (RoPA) Parser.

Parses Data Processing Agreements (DPAs), privacy policies, and contracts to
extract Article 30 mandatory fields and generate a structured RoPA (Register
of Processing Activities) in JSON format.

Article 30 requires controllers to maintain written records containing:
- Name and contact details of controller (and DPO if designated)
- Purposes of processing
- Categories of data subjects and personal data
- Categories of recipients
- International transfers (destination countries + safeguards)
- Retention periods (or criteria)
- Security measures description

Usage:
  python article30_parser.py --input contracts/processors/ --output ropa.json
  python article30_parser.py --input privacy_policy.md --output ropa.json --mode single

This is a helper tool; manual review and completion is required. The parser
uses keyword extraction and NLP patterns to identify Article 30 fields but
cannot guarantee 100% accuracy.
"""

import argparse
import json
import os
import re
import sys
from pathlib import Path
from datetime import datetime, timezone

# Keywords for field extraction (naive pattern matching; production would use NLP)
KEYWORDS = {
    "purposes": ["purpose", "why we process", "reason for processing", "use of data"],
    "data_subjects": ["customer", "employee", "user", "visitor", "subscriber", "data subject"],
    "personal_data": ["name", "email", "address", "phone", "ip address", "device id", "location", "biometric"],
    "recipients": ["processor", "vendor", "third party", "recipient", "share with", "disclose to"],
    "retention": ["retention period", "keep for", "store for", "delete after", "retain until"],
    "transfers": ["transfer to", "country", "outside EU", "outside EEA", "international transfer"],
    "security": ["encryption", "access control", "security measure", "pseudonymization", "tls", "mfa"]
}

def extract_text(file_path):
    """Extract text from markdown, txt, or JSON files."""
    ext = Path(file_path).suffix.lower()
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            if ext == '.json':
                data = json.load(f)
                # Flatten JSON to text
                return json.dumps(data, indent=2)
            else:
                return f.read()
    except Exception as e:
        print(f"[!] Could not read {file_path}: {e}", file=sys.stderr)
        return ""

def extract_fields(text, file_name):
    """Extract Article 30 fields using keyword patterns."""
    text_lower = text.lower()
    extracted = {
        "source_file": file_name,
        "purposes": [],
        "data_subjects": [],
        "personal_data_categories": [],
        "recipients": [],
        "retention_period": None,
        "international_transfers": [],
        "security_measures": []
    }
    
    # Extract sentences containing keywords
    sentences = re.split(r'[.!?\n]', text)
    
    for sent in sentences:
        sent_lower = sent.lower().strip()
        if not sent_lower:
            continue
            
        # Purposes
        if any(kw in sent_lower for kw in KEYWORDS["purposes"]):
            if len(sent) < 200:  # Avoid very long sentences
                extracted["purposes"].append(sent.strip())
        
        # Data subjects
        for subj in KEYWORDS["data_subjects"]:
            if subj in sent_lower:
                extracted["data_subjects"].append(subj)
        
        # Personal data categories
        for cat in KEYWORDS["personal_data"]:
            if cat in sent_lower:
                extracted["personal_data_categories"].append(cat)
        
        # Recipients
        if any(kw in sent_lower for kw in KEYWORDS["recipients"]):
            if len(sent) < 200:
                extracted["recipients"].append(sent.strip())
        
        # Retention
        if any(kw in sent_lower for kw in KEYWORDS["retention"]):
            if not extracted["retention_period"]:
                extracted["retention_period"] = sent.strip()
        
        # International transfers
        if any(kw in sent_lower for kw in KEYWORDS["transfers"]):
            extracted["international_transfers"].append(sent.strip())
        
        # Security measures
        if any(kw in sent_lower for kw in KEYWORDS["security"]):
            if len(sent) < 200:
                extracted["security_measures"].append(sent.strip())
    
    # Deduplicate lists
    extracted["data_subjects"] = list(set(extracted["data_subjects"]))
    extracted["personal_data_categories"] = list(set(extracted["personal_data_categories"]))
    extracted["purposes"] = list(set(extracted["purposes"]))[:5]  # Limit to top 5
    extracted["recipients"] = list(set(extracted["recipients"]))[:5]
    extracted["security_measures"] = list(set(extracted["security_measures"]))[:5]
    
    return extracted

def parse_directory(input_dir):
    """Parse all documents in a directory."""
    ropa_entries = []
    
    for root, dirs, files in os.walk(input_dir):
        for file in files:
            if file.startswith('.'):
                continue
            file_path = os.path.join(root, file)
            print(f"[*] Parsing: {file_path}", file=sys.stderr)
            text = extract_text(file_path)
            if text:
                entry = extract_fields(text, file)
                ropa_entries.append(entry)
    
    return ropa_entries

def main():
    parser = argparse.ArgumentParser(description="Parse documents for GDPR Article 30 fields")
    parser.add_argument("--input", "-i", required=True, help="Input file or directory")
    parser.add_argument("--output", "-o", default="ropa.json", help="Output JSON file")
    parser.add_argument("--mode", choices=["single", "directory"], default="directory",
                        help="Parse single file or directory")
    args = parser.parse_args()
    
    print(f"[*] Article 30 RoPA Parser - {datetime.now(timezone.utc).isoformat()}", file=sys.stderr)
    print(f"[*] Input: {args.input}", file=sys.stderr)
    print(f"[*] Mode: {args.mode}", file=sys.stderr)
    
    if args.mode == "single":
        if not os.path.isfile(args.input):
            print(f"[!] File not found: {args.input}", file=sys.stderr)
            return 2
        text = extract_text(args.input)
        ropa_entries = [extract_fields(text, os.path.basename(args.input))]
    else:
        if not os.path.isdir(args.input):
            print(f"[!] Directory not found: {args.input}", file=sys.stderr)
            return 2
        ropa_entries = parse_directory(args.input)
    
    # Build output structure
    output = {
        "organization": {
            "name": "[TO BE COMPLETED]",
            "controller_contact": "[TO BE COMPLETED]",
            "dpo_contact": "[IF REQUIRED]",
            "generated_date": datetime.now(timezone.utc).isoformat()
        },
        "processing_activities": ropa_entries,
        "completion_notes": [
            "This is a DRAFT generated by automated parsing.",
            "Manual review required for accuracy and completeness.",
            "Fill in [TO BE COMPLETED] placeholders.",
            "Verify all extracted fields against source documents.",
            "Add missing Article 30 mandatory fields.",
            "Consult legal counsel for final RoPA approval."
        ]
    }
    
    with open(args.output, 'w') as f:
        json.dump(output, f, indent=2)
    
    print(f"\n[✓] Parsed {len(ropa_entries)} document(s)", file=sys.stderr)
    print(f"[✓] RoPA draft written to: {args.output}", file=sys.stderr)
    print(f"[!] Manual review required - this is a DRAFT only", file=sys.stderr)
    
    return 0

if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/conducting-gdpr-compliance-assessment/scripts/article30_validator.py =====
#!/usr/bin/env python3
"""
GDPR Article 30 RoPA Validator.

Validates a Records of Processing Activities (RoPA) JSON file against Article 30
mandatory fields and flags incomplete or missing entries.

Checks:
- All mandatory Article 30 fields present
- Retention periods specified (or criteria documented)
- International transfers have documented safeguards
- Special category data properly flagged
- Security measures documented

Usage:
  python article30_validator.py --ropa ropa.json
  python article30_validator.py --ropa ropa.json --check-retention --check-transfers --strict
"""

import argparse
import json
import sys
from datetime import datetime, timezone

MANDATORY_FIELDS = [
    "purposes",
    "data_subjects",
    "personal_data_categories",
    "recipients"
]

SPECIAL_CATEGORY_KEYWORDS = [
    "health", "medical", "biometric", "genetic", "racial", "ethnic",
    "political", "religious", "trade union", "sex life", "sexual orientation"
]

def validate_entry(entry, index, args):
    """Validate a single processing activity entry."""
    issues = []
    entry_id = entry.get("source_file", f"Entry {index}")
    
    # Check mandatory fields
    for field in MANDATORY_FIELDS:
        if field not in entry or not entry[field]:
            issues.append({
                "severity": "ERROR",
                "field": field,
                "message": f"Missing mandatory field: {field}"
            })
        elif isinstance(entry[field], list) and len(entry[field]) == 0:
            issues.append({
                "severity": "ERROR",
                "field": field,
                "message": f"Empty list for mandatory field: {field}"
            })
    
    # Check retention period
    if args.check_retention:
        if "retention_period" not in entry or not entry["retention_period"]:
            issues.append({
                "severity": "WARNING",
                "field": "retention_period",
                "message": "Retention period not specified (Article 30(1)(f))"
            })
        elif "[TO BE COMPLETED]" in str(entry.get("retention_period", "")):
            issues.append({
                "severity": "WARNING",
                "field": "retention_period",
                "message": "Retention period placeholder not completed"
            })
    
    # Check international transfers
    if args.check_transfers:
        transfers = entry.get("international_transfers", [])
        if len(transfers) > 0:
            # Check if safeguards are documented
            safeguards_mentioned = False
            for transfer in transfers:
                transfer_lower = str(transfer).lower()
                if any(word in transfer_lower for word in ["scc", "standard contractual clause", "adequacy", "bcr", "binding corporate rule"]):
                    safeguards_mentioned = True
                    break
            
            if not safeguards_mentioned:
                issues.append({
                    "severity": "ERROR",
                    "field": "international_transfers",
                    "message": "International transfers identified but no safeguards documented (Chapter V)"
                })
    
    # Check for special category data without additional legal basis
    personal_data_str = " ".join(entry.get("personal_data_categories", [])).lower()
    if any(keyword in personal_data_str for keyword in SPECIAL_CATEGORY_KEYWORDS):
        if "special_category_legal_basis" not in entry:
            issues.append({
                "severity": "WARNING",
                "field": "special_category_legal_basis",
                "message": "Possible special category data (Article 9) but no additional legal basis documented"
            })
    
    # Check security measures
    if not entry.get("security_measures"):
        issues.append({
            "severity": "WARNING",
            "field": "security_measures",
            "message": "No security measures documented (Article 32)"
        })
    
    return entry_id, issues

def generate_report(results, args):
    """Generate validation report."""
    total_errors = sum(1 for _, issues in results for issue in issues if issue["severity"] == "ERROR")
    total_warnings = sum(1 for _, issues in results for issue in issues if issue["severity"] == "WARNING")
    
    print(f"\n{'='*70}")
    print(f"GDPR Article 30 RoPA Validation Report")
    print(f"Generated: {datetime.now(timezone.utc).isoformat()}")
    print(f"{'='*70}\n")
    
    print(f"Total Entries: {len(results)}")
    print(f"Total Errors: {total_errors}")
    print(f"Total Warnings: {total_warnings}")
    print()
    
    if total_errors == 0 and total_warnings == 0:
        print("✓ All entries passed validation")
        return 0
    
    for entry_id, issues in results:
        if not issues:
            continue
        
        print(f"\n{entry_id}")
        print(f"{'-'*70}")
        for issue in issues:
            icon = "✗" if issue["severity"] == "ERROR" else "⚠"
            print(f"  {icon} [{issue['severity']}] {issue['field']}: {issue['message']}")
    
    print(f"\n{'='*70}")
    if total_errors > 0:
        print(f"VALIDATION FAILED: {total_errors} error(s) found")
        print("Fix errors before using this RoPA for compliance purposes.")
        return 1
    else:
        print(f"VALIDATION PASSED with {total_warnings} warning(s)")
        print("Review warnings and complete missing optional fields.")
        return 0

def main():
    parser = argparse.ArgumentParser(description="Validate GDPR Article 30 RoPA")
    parser.add_argument("--ropa", required=True, help="Path to RoPA JSON file")
    parser.add_argument("--check-retention", action="store_true",
                        help="Validate retention periods are specified")
    parser.add_argument("--check-transfers", action="store_true",
                        help="Validate international transfer safeguards")
    parser.add_argument("--strict", action="store_true",
                        help="Treat warnings as errors")
    args = parser.parse_args()
    
    try:
        with open(args.ropa, 'r') as f:
            ropa = json.load(f)
    except (OSError, json.JSONDecodeError) as e:
        print(f"ERROR: Could not read RoPA file: {e}", file=sys.stderr)
        return 2
    
    processing_activities = ropa.get("processing_activities", [])
    if not processing_activities:
        print("ERROR: No processing activities found in RoPA", file=sys.stderr)
        return 2
    
    results = []
    for i, entry in enumerate(processing_activities):
        entry_id, issues = validate_entry(entry, i, args)
        if args.strict:
            # Promote warnings to errors in strict mode
            for issue in issues:
                if issue["severity"] == "WARNING":
                    issue["severity"] = "ERROR"
        results.append((entry_id, issues))
    
    return generate_report(results, args)

if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/conducting-gdpr-compliance-assessment/scripts/generate_ropa_report.py =====
#!/usr/bin/env python3
"""
GDPR Article 30 RoPA Report Generator.

Generates a formatted Article 30 Records of Processing Activities report
in Markdown format suitable for supervisory authority submission.

Usage:
  python generate_ropa_report.py --input ropa.json --output Article30_Register.md
  python generate_ropa_report.py --input ropa.json --output report.pdf --format pdf
"""

import argparse
import json
import sys
from datetime import datetime, timezone

def format_list(items):
    """Format a list of items as bullet points."""
    if not items:
        return "*[None specified]*"
    if isinstance(items, str):
        return items
    return "\n".join(f"- {item}" for item in items)

def generate_markdown(ropa):
    """Generate Markdown report from RoPA JSON."""
    org = ropa.get("organization", {})
    activities = ropa.get("processing_activities", [])
    
    lines = []
    lines.append("# Article 30 Records of Processing Activities")
    lines.append("")
    lines.append(f"**Organization**: {org.get('name', '[Organization Name]')}")
    lines.append(f"**Controller Contact**: {org.get('controller_contact', '[Contact Details]')}")
    if org.get('dpo_contact'):
        lines.append(f"**Data Protection Officer**: {org.get('dpo_contact')}")
    lines.append(f"**Generated**: {org.get('generated_date', datetime.now(timezone.utc).isoformat())}")
    lines.append(f"**Total Processing Activities**: {len(activities)}")
    lines.append("")
    lines.append("---")
    lines.append("")
    
    for idx, activity in enumerate(activities, 1):
        lines.append(f"## Processing Activity {idx}: {activity.get('source_file', 'Unnamed Activity')}")
        lines.append("")
        
        # Purposes
        lines.append("### Purposes of Processing")
        lines.append(format_list(activity.get('purposes', [])))
        lines.append("")
        
        # Data Subjects
        lines.append("### Categories of Data Subjects")
        lines.append(format_list(activity.get('data_subjects', [])))
        lines.append("")
        
        # Personal Data
        lines.append("### Categories of Personal Data")
        lines.append(format_list(activity.get('personal_data_categories', [])))
        lines.append("")
        
        # Recipients
        lines.append("### Categories of Recipients")
        lines.append(format_list(activity.get('recipients', [])))
        lines.append("")
        
        # Retention
        lines.append("### Retention Period")
        retention = activity.get('retention_period', '*[Not specified]*')
        lines.append(f"{retention}")
        lines.append("")
        
        # International Transfers
        lines.append("### International Transfers")
        transfers = activity.get('international_transfers', [])
        if transfers:
            lines.append(format_list(transfers))
        else:
            lines.append("*No international transfers*")
        lines.append("")
        
        # Security Measures
        lines.append("### Technical and Organizational Security Measures")
        lines.append(format_list(activity.get('security_measures', [])))
        lines.append("")
        
        lines.append("---")
        lines.append("")
    
    # Completion Notes
    if ropa.get('completion_notes'):
        lines.append("## Completion Notes")
        lines.append("")
        for note in ropa['completion_notes']:
            lines.append(f"- {note}")
        lines.append("")
    
    # Footer
    lines.append("---")
    lines.append("*This document fulfills the requirements of GDPR Article 30 (Records of Processing Activities).*")
    lines.append("*Controllers must maintain this register and make it available to supervisory authorities upon request.*")
    
    return "\n".join(lines)

def generate_html(markdown_content):
    """Convert Markdown to basic HTML."""
    html_header = """<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Article 30 RoPA</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 900px; margin: 40px auto; padding: 0 20px; }
        h1 { color: #1a5490; border-bottom: 3px solid #1a5490; }
        h2 { color: #2563eb; margin-top: 30px; }
        h3 { color: #4b5563; margin-top: 20px; }
        hr { margin: 30px 0; border: none; border-top: 1px solid #ddd; }
        ul { line-height: 1.6; }
        em { color: #6b7280; }
    </style>
</head>
<body>
"""
    html_footer = """
</body>
</html>
"""
    
    # Simple Markdown to HTML conversion
    html_body = markdown_content
    html_body = html_body.replace("# ", "<h1>").replace("\n\n", "</h1>\n\n")
    html_body = html_body.replace("## ", "<h2>").replace("\n\n", "</h2>\n\n")
    html_body = html_body.replace("### ", "<h3>").replace("\n\n", "</h3>\n\n")
    html_body = html_body.replace("**", "<strong>").replace("**", "</strong>")
    html_body = html_body.replace("*[", "<em>[").replace("]*", "]</em>")
    html_body = html_body.replace("---\n", "<hr>\n")
    html_body = html_body.replace("- ", "<li>").replace("\n", "</li>\n")
    
    return html_header + html_body + html_footer

def main():
    parser = argparse.ArgumentParser(description="Generate Article 30 RoPA Report")
    parser.add_argument("--input", "-i", required=True, help="Input RoPA JSON file")
    parser.add_argument("--output", "-o", required=True, help="Output file path")
    parser.add_argument("--format", choices=["markdown", "html", "pdf"], default="markdown",
                        help="Output format (default: markdown)")
    args = parser.parse_args()
    
    try:
        with open(args.input, 'r') as f:
            ropa = json.load(f)
    except (OSError, json.JSONDecodeError) as e:
        print(f"ERROR: Could not read RoPA file: {e}", file=sys.stderr)
        return 2
    
    markdown_content = generate_markdown(ropa)
    
    if args.format == "markdown":
        with open(args.output, 'w') as f:
            f.write(markdown_content)
        print(f"[✓] Markdown report written to: {args.output}", file=sys.stderr)
    
    elif args.format == "html":
        html_content = generate_html(markdown_content)
        with open(args.output, 'w') as f:
            f.write(html_content)
        print(f"[✓] HTML report written to: {args.output}", file=sys.stderr)
    
    elif args.format == "pdf":
        try:
            # Try to use markdown2pdf if available
            import subprocess
            md_temp = args.output.replace('.pdf', '.md')
            with open(md_temp, 'w') as f:
                f.write(markdown_content)
            
            # Try pandoc first, fall back to instructions
            result = subprocess.run(['pandoc', md_temp, '-o', args.output], 
                                    capture_output=True)
            if result.returncode == 0:
                print(f"[✓] PDF report written to: {args.output}", file=sys.stderr)
                import os
                os.remove(md_temp)
            else:
                print(f"[!] pandoc not found. Markdown saved to: {md_temp}", file=sys.stderr)
                print(f"[!] Install pandoc to generate PDF: apt install pandoc", file=sys.stderr)
        except Exception as e:
            print(f"[!] PDF generation requires pandoc: {e}", file=sys.stderr)
            print(f"[!] Generating Markdown instead: {args.output.replace('.pdf', '.md')}", file=sys.stderr)
            with open(args.output.replace('.pdf', '.md'), 'w') as f:
                f.write(markdown_content)
    
    return 0

if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/executing-nist-rmf-authorization-to-operate/SKILL.md =====
---
name: executing-nist-rmf-authorization-to-operate
description: >-
  Drive a federal system through the NIST Risk Management Framework (SP 800-37 Rev 2)
  to an Authorization to Operate (ATO): Prepare, Categorize (FIPS 199), Select a control
  baseline (FIPS 200 / SP 800-53 Rev 5), Implement, Assess (SP 800-53A), Authorize, and
  Monitor continuously. Use when a system needs an ATO or a renewal, when working a
  FISMA/FedRAMP authorization package, when building or reviewing an SSP, SAR, or POA&M,
  when categorizing a system as Low/Moderate/High impact, when selecting or tailoring a
  control baseline, or when standing up continuous monitoring (ConMon) after authorization.
  Covers ATO, conditional ATO (cATO), and the artifacts assessors expect. Keywords: NIST
  RMF, 800-37, ATO, authorization to operate, FISMA, FedRAMP, SSP, SAR, POA&M, FIPS 199,
  FIPS 200, 800-53, 800-53A, control baseline, security categorization, continuous
  monitoring, authorizing official, system boundary, ongoing authorization.
domain: cybersecurity
subdomain: compliance-governance
tags:
- nist-rmf
- nist-800-37
- ato
- fisma
- fedramp
- nist-800-53
- fips-199
- ssp
- poam
- continuous-monitoring
- governance
version: "1.0"
author: andrewibrah
license: Apache-2.0
nist_csf:
- GV.OC-03
- GV.RM-01
- ID.AM-08
- ID.RA-05
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1068
- T1210
- T1486
---

# Executing the NIST RMF to an Authorization to Operate (ATO)

## When to Use

- When a federal or federally-aligned system (or a FedRAMP cloud service) needs an **Authorization to Operate**, a re-authorization, or has fallen out of authorization.
- When you must produce or review the core authorization artifacts: **System Security Plan (SSP)**, **Security Assessment Report (SAR)**, and **Plan of Action & Milestones (POA&M)**.
- When categorizing a system's impact level (Low / Moderate / High) under **FIPS 199**.
- When selecting, tailoring, or implementing a **NIST SP 800-53 Rev 5** control baseline.
- When standing up **continuous monitoring (ConMon)** or pursuing **ongoing authorization / cATO** after an initial ATO.

## Prerequisites

- A defined **system** and **authorization boundary** (what's in, what's inherited, what's a leveraged service).
- An identified **Authorizing Official (AO)**, **System Owner**, and **ISSO**.
- The information types the system handles (use **SP 800-60** to map them to impact levels).
- For cloud: the provider's **Customer Responsibility Matrix (CRM)** and any inherited/leveraged ATO.
- Access to assessment evidence sources (config, scans, policies) for the Assess step.

## Workflow

NIST SP 800-37 Rev 2 defines **seven steps**. Prepare is the foundation; the rest run in order and then loop through Monitor.

### 0/1. Prepare (organization and system level)
Establish context: roles (AO, SO, ISSO, assessor), risk-management strategy and tolerance (ties to SP 800-39), a control baseline strategy, common controls available for inheritance, and the system's mission/business context. Define the **authorization boundary** precisely — scope creep here inflates the whole package.

### 2. Categorize (FIPS 199 + SP 800-60)
Determine the impact level for **confidentiality, integrity, and availability** for each information type, then take the **high-water mark** across the three to set the overall system categorization: **Low**, **Moderate**, or **High**. Document in the SSP. This single decision drives the entire control baseline.

### 3. Select (FIPS 200 + SP 800-53 Rev 5 + SP 800-53B)
Start from the SP 800-53B baseline matching the categorization (Low/Moderate/High). Then **tailor**: apply scoping guidance, select compensating controls where needed, and assign values to organization-defined parameters. Add overlays (e.g., privacy, FedRAMP). Record the tailored set and the rationale in the SSP. Identify which controls are **common (inherited)**, **system-specific**, or **hybrid**.

### 4. Implement
Deploy the selected controls and **document how each is implemented** in the SSP — the implementation statement, not just "yes." This is the artifact assessors read first; vague statements generate findings.

### 5. Assess (SP 800-53A Rev 5)
An independent assessor evaluates controls using the **examine / interview / test** methods against assessment objectives. Findings of "other than satisfied" become weaknesses. Output is the **Security Assessment Report (SAR)**. Remediate what you can before authorization; the rest flows to the POA&M.

### 6. Authorize
Assemble the **authorization package**: SSP + SAR + POA&M (plus supporting artifacts). The AO reviews **residual risk** and renders a decision:
- **ATO** — authorized, typically with a defined term and a ConMon expectation.
- **Conditional / cATO** — authorized subject to conditions or operating under an approved ongoing-authorization model.
- **Denial / DATO** — risk too high; system may not operate.

The decision and its rationale are captured in the **authorization decision document**.

### 7. Monitor (continuous monitoring)
Authorization is not a one-time gate. Maintain an ongoing posture: track control effectiveness, ingest scan/config drift, update the SSP on change, work the POA&M to closure, report per the ConMon plan, and feed significant changes back into reassessment. Mature programs move from periodic re-ATO to **ongoing authorization**.

## Key Concepts

| Concept | Definition |
|---|---|
| Authorization boundary | The set of components, data flows, and inherited services covered by the authorization. |
| FIPS 199 categorization | Low/Moderate/High per C/I/A; overall = high-water mark across the three. |
| Control baseline | The SP 800-53B starting control set for the categorization, before tailoring. |
| Tailoring | Adjusting the baseline via scoping, compensating controls, and parameter values. |
| Common / inherited control | A control provided by another entity (e.g., the platform) and inherited by the system. |
| SSP | System Security Plan — describes the system, boundary, and how each control is implemented. |
| SAR | Security Assessment Report — the assessor's findings on control effectiveness. |
| POA&M | Plan of Action & Milestones — tracked weaknesses with owners and remediation dates. |
| ATO / cATO / DATO | Authorize / conditional (ongoing) / denial of authorization to operate. |
| Authorizing Official (AO) | The senior official who accepts residual risk and signs the authorization. |
| ConMon | Continuous monitoring — ongoing control-effectiveness and risk tracking post-ATO. |

## Tools & Systems

- **NIST SP 800-37 Rev 2** — the RMF process (7 steps).
- **FIPS 199 / FIPS 200 / SP 800-60** — categorization and minimum requirements.
- **NIST SP 800-53 Rev 5 / 800-53B** — control catalog and baselines.
- **NIST SP 800-53A Rev 5** — assessment procedures (examine/interview/test).
- **OSCAL** — machine-readable SSP/SAR/POA&M (NIST's authorization-document format).
- **eMASS** (DoD) / **FedRAMP** templates — package management and submission.
- **GRC platforms** — Xacta, ServiceNow, RegScale, etc., to manage the package and ConMon.
- **NIST CSF 2.0** — cross-walks to communicate RMF posture in framework terms.

## Common Scenarios

- **New system pre-launch.** Run Categorize → Authorize before go-live; ATO is the gate to production.
- **Cloud service (FedRAMP).** Inherit the platform's controls, document the CRM split, and authorize the customer-responsible delta.
- **Re-authorization.** Triggered by term expiry or significant change; refresh SSP/SAR/POA&M and re-decide.
- **cATO / ongoing authorization.** Replace periodic re-ATO with continuous evidence and an approved ConMon model.
- **POA&M review.** Triage open weaknesses by risk, assign owners and dates, and report closure trend to the AO.

## Output Format

Produce an **Authorization Package summary** using `assets/template.md`, containing:

1. **System & boundary** — description, components, data flows, inherited services.
2. **Categorization** — FIPS 199 C/I/A and overall impact, with information-type rationale.
3. **Control baseline & tailoring** — baseline selected, tailoring decisions, common vs system-specific.
4. **Implementation status** — per-family implementation summary (from the SSP).
5. **Assessment results (SAR)** — findings by severity; what's satisfied vs other-than-satisfied.
6. **POA&M** — open weaknesses, risk, owner, milestone dates.
7. **Authorization decision** — ATO/cATO/DATO, term, conditions, residual-risk statement, AO.
8. **ConMon plan** — what's monitored, how often, reporting cadence, reassessment triggers.

Use `scripts/process.py` to select the right SP 800-53B baseline from a FIPS 199 categorization, summarize control-implementation status, and generate a POA&M table from a findings JSON.

===== END FILE =====


===== FILE: skills/executing-nist-rmf-authorization-to-operate/LICENSE =====

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


===== FILE: skills/executing-nist-rmf-authorization-to-operate/assets/template.md =====
# Authorization Package Summary (NIST RMF / SP 800-37 Rev 2) — Worked Example

> Filled example for a Moderate-impact federal web application seeking an initial ATO.
> Replace bracketed content for your own system.

## 1. System & Authorization Boundary
- **System name:** Citizen Services Portal (CSP)
- **System owner / ISSO / AO:** [SO name] / [ISSO name] / [AO name]
- **Description:** Public-facing web portal for benefit applications; React frontend, containerized API, managed Postgres, all in an authorized cloud (FedRAMP Moderate) tenant.
- **Boundary:** The application containers, API gateway, database, and CI/CD pipeline within the project's cloud account. **Inherited:** physical, environmental, and hypervisor controls from the FedRAMP-authorized platform (documented in the CRM).
- **Leveraged authorization:** Platform IaaS at FedRAMP Moderate.

## 2. FIPS 199 Categorization
*(generated by `scripts/process.py` from the information-type table; overall = high-water mark)*

| Objective | High-water mark |
|---|---|
| Confidentiality | Moderate |
| Integrity | Moderate |
| Availability | Moderate |
| **Overall system impact** | **Moderate** |

**Information types:** PII (C:Mod / I:Mod / A:Low), Eligibility records (C:Mod / I:Mod / A:Mod), Public content (C:Low / I:Mod / A:Mod).
**Selected SP 800-53B baseline:** Moderate. Privacy overlay applied (PII present).

## 3. Control Baseline & Tailoring
- **Baseline:** SP 800-53B Moderate + privacy controls (PT family).
- **Tailoring decisions:** PE family largely **inherited** from the platform (common controls). Organization-defined parameters set for AC-7 (lockout threshold = 5), AU-11 (log retention = 1 year), IA-5 (password/authenticator policy).
- **Allocation:** Common (inherited) — PE, parts of SC/CP; System-specific — AC, AU, SI, application-layer SC; Hybrid — IR, CM (platform + app split per CRM).

## 4. Control Implementation Status (from the SSP)
| Family | Implemented | Total | % |
|---|---|---|---|
| AC | 22 | 25 | 88% |
| AU | 14 | 16 | 88% |
| SC | 28 | 30 | 93% |
| SI | 12 | 14 | 86% |
| **Total** | **76** | **85** | **89%** |

*(Each implemented control carries an implementation statement in the SSP — not a bare "yes." Open items map to the POA&M below.)*

## 5. Assessment Results (SAR)
Independent assessment per SP 800-53A Rev 5 (examine / interview / test). Result: **3 controls "Other Than Satisfied"**, all with remediation plans. No Critical findings. Detailed evidence in the full SAR.

## 6. Plan of Action & Milestones (POA&M)
*(generated by `scripts/process.py`, sorted by severity)*

| ID | Control | Weakness | Severity | Status | Remediation | Owner | Milestone |
|---|---|---|---|---|---|---|---|
| F-001 | AC-7 | No account lockout on the portal login | High | Other Than Satisfied | Configure lockout after 5 failed attempts | App team | 2026-07-15 |
| F-002 | AU-6 | Audit logs not reviewed on a defined cadence | Moderate | Other Than Satisfied | Stand up weekly SIEM review + alerting | SOC | 2026-08-01 |
| F-003 | SI-2 | Two medium CVEs unpatched in a dependency | Moderate | Other Than Satisfied | Patch in next sprint; add Dependabot gate | App team | 2026-07-30 |

_Open High/Critical findings: 1 (F-001) — track to closure before/within ATO conditions._

## 7. Authorization Decision
- **Decision:** **ATO with conditions** (effectively a cATO posture).
- **Term:** 3 years, contingent on continuous monitoring and POA&M adherence.
- **Conditions:** Close F-001 (High) within 30 days of authorization; F-002 and F-003 per their milestones.
- **Residual-risk statement:** Residual risk is **Moderate and acceptable** given the compensating monitoring and the committed remediation timeline.
- **Authorizing Official:** [AO name], [date].

## 8. Continuous Monitoring (ConMon) Plan
- **Monitored continuously:** vulnerability scans (weekly), configuration drift (CM), POA&M status (monthly to the AO), control effectiveness sampling (quarterly).
- **Reporting cadence:** Monthly ConMon report; immediate notification of any High/Critical finding or significant change.
- **Reassessment triggers:** new external interface, change of categorization, major architecture change, or a significant incident.
- **Maturity goal:** Move from periodic re-ATO toward ongoing authorization as ConMon evidence stabilizes.

===== END FILE =====


===== FILE: skills/executing-nist-rmf-authorization-to-operate/references/standards.md =====
# NIST RMF / ATO — Standards & Reference

## Primary standard
### NIST SP 800-37 Revision 2 — Risk Management Framework for Information Systems and Organizations
- **Publisher**: NIST
- **Published**: December 2018
- **Scope**: A 7-step lifecycle for managing security and privacy risk and authorizing systems to operate.
- **URL**: https://csrc.nist.gov/pubs/sp/800/37/r2/final

## The seven RMF steps
| # | Step | Core question | Key inputs |
|---|---|---|---|
| 1 | Prepare | Are roles, strategy, and boundary set? | SP 800-39 risk strategy, common controls |
| 2 | Categorize | How bad is a loss of C/I/A? | FIPS 199, SP 800-60 |
| 3 | Select | Which controls apply? | FIPS 200, SP 800-53B baselines, SP 800-53 Rev 5 |
| 4 | Implement | How is each control built? | SSP implementation statements |
| 5 | Assess | Do the controls work? | SP 800-53A Rev 5; produces the SAR |
| 6 | Authorize | Is residual risk acceptable? | Package (SSP+SAR+POA&M); AO decision |
| 7 | Monitor | Is it still effective? | ConMon plan, scans, change management |

## Companion standards
| Document | Role |
|---|---|
| FIPS 199 | Security categorization — Low/Moderate/High per confidentiality, integrity, availability. |
| FIPS 200 | Minimum security requirements for federal information and systems. |
| NIST SP 800-60 Vol 1 & 2 | Maps information types to impact levels (input to FIPS 199). |
| NIST SP 800-53 Rev 5 | Control catalog — 20 control families. |
| NIST SP 800-53B | Control baselines (Low / Moderate / High) and the privacy baseline. |
| NIST SP 800-53A Rev 5 | Assessment procedures (examine / interview / test). |
| NIST SP 800-39 | Organization-wide risk management context (three tiers). |
| NIST SP 800-137 | Information Security Continuous Monitoring (ISCM) — the Monitor step. |
| OSCAL | Open Security Controls Assessment Language — machine-readable SSP/SAP/SAR/POA&M. |

## FIPS 199 categorization
For each information type, rate the impact of a loss of:
- **Confidentiality** — unauthorized disclosure
- **Integrity** — unauthorized modification/destruction
- **Availability** — disruption of access/use

Each at **Low / Moderate / High**. The **overall system impact level = the high-water mark** (highest single value) across all information types and all three objectives. That overall level selects the SP 800-53B baseline.

## SP 800-53 Rev 5 control families (20)
AC (Access Control), AT (Awareness & Training), AU (Audit & Accountability), CA (Assessment, Authorization & Monitoring), CM (Configuration Management), CP (Contingency Planning), IA (Identification & Authentication), IR (Incident Response), MA (Maintenance), MP (Media Protection), PE (Physical & Environmental Protection), PL (Planning), PM (Program Management), PS (Personnel Security), PT (PII Processing & Transparency), RA (Risk Assessment), SA (System & Services Acquisition), SC (System & Communications Protection), SI (System & Information Integrity), SR (Supply Chain Risk Management).

## Control allocation
- **Common (inherited)** — provided by another provider/platform; the system inherits the implementation and the evidence.
- **System-specific** — implemented and owned by this system.
- **Hybrid** — partly inherited, partly system-specific (responsibility split documented, e.g., in a FedRAMP CRM).

## Core authorization artifacts
| Artifact | Produced in step | Contents |
|---|---|---|
| SSP — System Security Plan | Select/Implement | System description, boundary, categorization, control implementation statements. |
| SAR — Security Assessment Report | Assess | Assessor findings: satisfied / other-than-satisfied, with evidence. |
| POA&M — Plan of Action & Milestones | Assess → Authorize | Open weaknesses, risk, remediation owner, milestone dates. |
| Authorization Decision Document | Authorize | ATO/cATO/DATO, term, conditions, residual-risk acceptance, AO signature. |
| ConMon Plan | Monitor | What's monitored, frequency, reporting cadence, reassessment triggers. |

## Authorization outcomes
- **ATO** — Authorization to Operate (often time-bound, e.g., up to 3 years, with ConMon).
- **cATO** — Conditional / ongoing authorization under an approved continuous model (increasingly preferred in DoD).
- **DATO** — Denial of Authorization to Operate.

## NIST CSF 2.0 alignment
| CSF 2.0 ID | Relevance |
|---|---|
| GV.OC-03 | Legal/regulatory requirements (FISMA) understood and managed. |
| GV.RM-01 | Risk-management objectives established and agreed. |
| ID.AM-08 | Systems managed across the lifecycle (authorization boundary). |
| ID.RA-05 | Risk used to inform prioritization and the authorization decision. |
| PR.IR-01 | Protective technology / controls implemented per the baseline. |

===== END FILE =====


===== FILE: skills/executing-nist-rmf-authorization-to-operate/scripts/process.py =====
#!/usr/bin/env python3
"""
NIST RMF helper: FIPS 199 categorization -> SP 800-53B baseline selection,
control-implementation status summary, and POA&M generation from findings.

Input JSON shape:
{
  "system": {"name": "Customer Portal", "ao": "Jane Roe"},
  "information_types": [
    {"name": "PII", "confidentiality": "Moderate", "integrity": "Moderate", "availability": "Low"},
    {"name": "Authentication data", "confidentiality": "High", "integrity": "High", "availability": "Moderate"}
  ],
  "implementation": {                 # OPTIONAL: per-family implemented/total counts
    "AC": {"implemented": 20, "total": 25},
    "AU": {"implemented": 10, "total": 16}
  },
  "findings": [                       # OPTIONAL: assessor findings -> POA&M
    {
      "id": "F-001", "control": "AC-7", "weakness": "No account lockout on portal",
      "severity": "High", "status": "Other Than Satisfied",
      "remediation": "Configure lockout after 5 failed attempts",
      "owner": "App team", "milestone": "2026-07-15"
    }
  ]
}

Categorization rule (FIPS 199): overall impact = high-water mark across all
information types and all three objectives (C, I, A).

Usage:
  python process.py --input system.json [--output package.md]
  python process.py --input system.json --fail-open-high   # exit 1 if any High finding open
"""

import argparse
import json
import sys

IMPACT = ["Low", "Moderate", "High"]
IMPACT_RANK = {v: i for i, v in enumerate(IMPACT)}
SEVERITY_RANK = {"Low": 0, "Moderate": 1, "Medium": 1, "High": 2, "Critical": 3}

# Indicative SP 800-53B baseline control counts by impact level.
# (Approximate; the authoritative counts live in SP 800-53B. Used here to
#  communicate relative baseline size, not as a substitute for the catalog.)
BASELINE_SIZE = {"Low": "~150 controls", "Moderate": "~260 controls", "High": "~340 controls"}


def categorize(info_types):
    """Return (overall, per_objective_high) using the FIPS 199 high-water mark."""
    highs = {"confidentiality": "Low", "integrity": "Low", "availability": "Low"}
    for it in info_types:
        for obj in highs:
            val = it.get(obj, "Low")
            if val not in IMPACT_RANK:
                raise ValueError(f"information type '{it.get('name','?')}' has invalid {obj} '{val}'")
            if IMPACT_RANK[val] > IMPACT_RANK[highs[obj]]:
                highs[obj] = val
    overall = max(highs.values(), key=lambda v: IMPACT_RANK[v])
    return overall, highs


def render(data):
    sysmeta = data.get("system", {})
    info_types = data.get("information_types", [])
    if not info_types:
        raise ValueError("information_types is required to categorize the system")

    overall, highs = categorize(info_types)

    lines = []
    lines.append(f"# Authorization Package Summary - {sysmeta.get('name','System')}")
    lines.append("")
    if sysmeta.get("ao"):
        lines.append(f"- **Authorizing Official:** {sysmeta['ao']}")
    lines.append("")

    # categorization
    lines.append("## FIPS 199 Categorization")
    lines.append("")
    lines.append("| Objective | High-water mark |")
    lines.append("|---|---|")
    lines.append(f"| Confidentiality | {highs['confidentiality']} |")
    lines.append(f"| Integrity | {highs['integrity']} |")
    lines.append(f"| Availability | {highs['availability']} |")
    lines.append(f"| **Overall system impact** | **{overall}** |")
    lines.append("")
    lines.append(f"**Selected SP 800-53B baseline:** {overall} ({BASELINE_SIZE[overall]}). "
                 "Tailor from this baseline; document scoping, compensating controls, and "
                 "organization-defined parameter values in the SSP.")
    lines.append("")

    lines.append("### Information types")
    lines.append("")
    lines.append("| Information type | C | I | A |")
    lines.append("|---|---|---|---|")
    for it in info_types:
        lines.append(f"| {it.get('name','-')} | {it.get('confidentiality','-')} "
                     f"| {it.get('integrity','-')} | {it.get('availability','-')} |")
    lines.append("")

    # implementation status
    impl = data.get("implementation")
    if impl:
        lines.append("## Control Implementation Status (by family)")
        lines.append("")
        lines.append("| Family | Implemented | Total | % |")
        lines.append("|---|---|---|---|")
        tot_i = tot_t = 0
        for fam, c in sorted(impl.items()):
            i, t = c.get("implemented", 0), c.get("total", 0)
            tot_i += i
            tot_t += t
            pct = f"{(100*i/t):.0f}%" if t else "-"
            lines.append(f"| {fam} | {i} | {t} | {pct} |")
        overall_pct = f"{(100*tot_i/tot_t):.0f}%" if tot_t else "-"
        lines.append(f"| **Total** | **{tot_i}** | **{tot_t}** | **{overall_pct}** |")
        lines.append("")

    # POA&M
    findings = data.get("findings", [])
    open_high = []
    if findings:
        findings_sorted = sorted(
            findings,
            key=lambda f: SEVERITY_RANK.get(f.get("severity", "Low"), 0),
            reverse=True,
        )
        lines.append("## Plan of Action & Milestones (POA&M)")
        lines.append("")
        lines.append("| ID | Control | Weakness | Severity | Status | Remediation | Owner | Milestone |")
        lines.append("|---|---|---|---|---|---|---|---|")
        for f in findings_sorted:
            sev = f.get("severity", "-")
            status = f.get("status", "-")
            is_open = status.strip().lower() != "satisfied"
            is_high = SEVERITY_RANK.get(sev, 0) >= SEVERITY_RANK["High"]
            if is_open and is_high:
                open_high.append(f)
            lines.append("| {id} | {ctl} | {wk} | {sev} | {st} | {rem} | {own} | {ms} |".format(
                id=f.get("id", "-"), ctl=f.get("control", "-"), wk=f.get("weakness", "-"),
                sev=sev, st=status, rem=f.get("remediation", "-"),
                own=f.get("owner", "-"), ms=f.get("milestone", "-"),
            ))
        lines.append("")
        lines.append(f"_Open High/Critical findings: {len(open_high)}_")
        lines.append("")

    # decision scaffold
    lines.append("## Authorization Decision (to be completed by the AO)")
    lines.append("")
    lines.append("- **Decision:** [ ATO | cATO | DATO ]")
    lines.append("- **Term / conditions:** ____")
    lines.append("- **Residual-risk statement:** ____")
    lines.append(f"- **Authorizing Official:** {sysmeta.get('ao','____')}")

    return "\n".join(lines), overall, open_high


def main():
    ap = argparse.ArgumentParser(description="NIST RMF baseline selector + POA&M generator")
    ap.add_argument("--input", "-i", required=True, help="Path to system JSON")
    ap.add_argument("--output", "-o", help="Write Markdown package summary to this path")
    ap.add_argument("--fail-open-high", action="store_true",
                    help="Exit non-zero if any High/Critical finding remains open")
    args = ap.parse_args()

    try:
        with open(args.input) as f:
            data = json.load(f)
    except (OSError, json.JSONDecodeError) as e:
        print(f"ERROR: could not read input JSON: {e}", file=sys.stderr)
        return 2

    try:
        md, overall, open_high = render(data)
    except ValueError as e:
        print(f"ERROR: {e}", file=sys.stderr)
        return 2

    if args.output:
        with open(args.output, "w") as f:
            f.write(md + "\n")
        print(f"Package summary written to {args.output}", file=sys.stderr)
    else:
        print(md)

    print(f"Overall categorization: {overall}. Open High/Critical findings: {len(open_high)}.",
          file=sys.stderr)

    if args.fail_open_high and open_high:
        ids = ", ".join(f.get("id", "?") for f in open_high)
        print(f"FAIL: {len(open_high)} open High/Critical finding(s): {ids}", file=sys.stderr)
        return 1
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/implementing-gdpr-data-protection-controls/SKILL.md =====
---
name: implementing-gdpr-data-protection-controls
description: Implements GDPR (EU 2016/679) technical and organizational measures — privacy by design/default, DPIAs, data subject rights management, 72-hour breach notification, and cross-border transfer mechanisms (SCCs, BCRs, adequacy). Use when designing or auditing GDPR controls, building a DPIA, handling data subject access/erasure requests, or assessing cross-border data transfers.
domain: cybersecurity
subdomain: compliance-governance
tags:
- compliance
- governance
- gdpr
- privacy
- data-protection
- eu-regulation
nist_csf:
- GV.OC-02
- GV.PO-01
- PR.DS-01
- PR.AA-01
- ID.AM-02
version: '1.0'
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- MEASURE-2.7
- MAP-5.1
- MANAGE-2.4
- MEASURE-2.8
- MEASURE-2.9
atlas_techniques:
- AML.T0070
- AML.T0066
- AML.T0082
mitre_attack:
- T1078
- T1530
- T1685.002
---
# Implementing GDPR Data Protection Controls

## Overview
The General Data Protection Regulation (EU) 2016/679 (GDPR) is the EU's comprehensive data protection law governing the collection, processing, storage, and transfer of personal data. This skill covers implementing the technical and organizational measures required by GDPR, including data protection by design and by default, Data Protection Impact Assessments (DPIAs), data subject rights management, breach notification procedures, and cross-border data transfer mechanisms.


## When to Use

- When deploying or configuring implementing gdpr data protection controls capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites
- Understanding of EU data protection law and its territorial scope
- Knowledge of personal data processing activities within the organization
- Familiarity with data architecture, databases, and application systems
- Understanding of data flows including cross-border transfers

## Core Concepts

### Key GDPR Articles for Technical Controls

| Article | Requirement |
|---------|-------------|
| Art. 5 | Principles: lawfulness, purpose limitation, data minimization, accuracy, storage limitation, integrity and confidentiality, accountability |
| Art. 6 | Lawful basis for processing (consent, contract, legal obligation, vital interests, public task, legitimate interest) |
| Art. 25 | Data protection by design and by default |
| Art. 28 | Processor obligations and contractual requirements |
| Art. 30 | Records of processing activities (ROPA) |
| Art. 32 | Security of processing (technical and organizational measures) |
| Art. 33 | Breach notification to supervisory authority (72 hours) |
| Art. 34 | Communication of breach to data subjects |
| Art. 35 | Data Protection Impact Assessment (DPIA) |
| Art. 37-39 | Data Protection Officer (DPO) appointment and role |
| Art. 44-49 | Cross-border data transfers (adequacy, SCCs, BCRs) |

### Article 32 Security Measures
The regulation requires organizations to implement measures appropriate to the risk:
- **Pseudonymization** and encryption of personal data
- **Confidentiality, integrity, availability, and resilience** of processing systems
- **Ability to restore** availability and access to personal data in a timely manner
- **Regular testing** and evaluation of technical and organizational measures

### Data Subject Rights (Articles 12-22)
| Right | Article | Description |
|-------|---------|-------------|
| Right to be informed | 13-14 | Transparent information about processing |
| Right of access | 15 | Obtain copy of personal data |
| Right to rectification | 16 | Correct inaccurate data |
| Right to erasure | 17 | "Right to be forgotten" |
| Right to restrict processing | 18 | Limit processing of data |
| Right to data portability | 20 | Receive data in machine-readable format |
| Right to object | 21 | Object to processing (especially direct marketing) |
| Automated decision-making | 22 | Not be subject to solely automated decisions |

## Workflow

### Phase 1: Data Mapping and Assessment (Weeks 1-6)
1. Create comprehensive data inventory:
   - What personal data is collected
   - From whom (data subjects)
   - Why (purposes and lawful bases)
   - Where it's stored (systems, locations, countries)
   - Who has access (internal and external)
   - How long it's retained
   - What security measures protect it
2. Document Records of Processing Activities (ROPA) per Article 30
3. Identify lawful basis for each processing activity
4. Map cross-border data transfers and transfer mechanisms
5. Identify processing activities requiring DPIA

### Phase 2: Gap Analysis and Risk Assessment (Weeks 7-10)
1. Assess current state against GDPR requirements
2. Perform DPIAs for high-risk processing activities
3. Identify security gaps in Article 32 compliance
4. Evaluate data retention compliance
5. Assess data subject rights request handling capabilities

### Phase 3: Technical Controls Implementation (Weeks 11-24)
1. **Encryption**:
   - Data at rest: AES-256 for databases, file systems, backups
   - Data in transit: TLS 1.2+ for all personal data transfers
   - Key management: secure key storage and rotation procedures
2. **Pseudonymization**:
   - Implement tokenization for sensitive identifiers
   - Separate pseudonymization keys from data stores
3. **Access Controls**:
   - Role-based access control (RBAC) for personal data
   - Principle of least privilege
   - MFA for systems processing personal data
   - Regular access reviews
4. **Data Minimization**:
   - Implement data collection limits at application layer
   - Default privacy settings (data protection by default)
   - Automated data retention enforcement
5. **Erasure and Portability**:
   - Build data deletion workflows across all systems
   - Implement data export in machine-readable formats (JSON, CSV)
   - Cascade deletion to backups and archives
6. **Consent Management**:
   - Implement granular consent collection mechanisms
   - Consent withdrawal functionality
   - Consent audit trail and versioning
7. **Breach Detection**:
   - SIEM for personal data access monitoring
   - Data loss prevention (DLP) controls
   - Anomalous access detection

### Phase 4: Organizational Controls (Weeks 11-24)
1. Appoint Data Protection Officer (DPO) if required
2. Develop data protection policies and procedures
3. Create breach notification procedures (72-hour timeline)
4. Establish data subject request (DSR) handling procedures
5. Implement vendor management with Data Processing Agreements (DPAs)
6. Deploy privacy awareness training for all staff
7. Create data protection by design guidance for development teams

### Phase 5: Documentation and Compliance Evidence (Weeks 25-30)
1. Finalize ROPA documentation
2. Document all DPIAs and outcomes
3. Create data protection policies
4. Document technical and organizational measures
5. Establish privacy notice and consent records
6. Create international transfer documentation (SCCs, TIAs)

### Phase 6: Ongoing Compliance (Continuous)
1. Regular DPIA reviews for new processing activities
2. Annual data mapping refresh
3. Periodic security measure testing (Art. 32 requirement)
4. Data subject request tracking and SLA monitoring
5. Breach response readiness testing
6. Training refresh and awareness campaigns

## Key Artifacts
- Records of Processing Activities (ROPA)
- Data Protection Impact Assessments (DPIAs)
- Data Processing Agreements (DPAs)
- Privacy Notices and Consent Records
- Breach Response Procedures and Register
- Data Subject Request Handling Procedures
- International Data Transfer Mechanisms (SCCs, BCRs)
- Technical and Organizational Measures Documentation

## Common Pitfalls
- Treating GDPR as only a legal/compliance exercise without technical implementation
- Incomplete data mapping missing shadow IT or legacy systems
- Failing to maintain consent audit trails
- Not testing 72-hour breach notification capability
- Ignoring cross-border transfer requirements for cloud services
- Over-reliance on consent as lawful basis when legitimate interest applies

## References
- GDPR Official Text: https://gdpr-info.eu/
- European Data Protection Board (EDPB) Guidelines
- ICO (UK) GDPR Guidance: https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/
- CNIL (France) GDPR Compliance Toolkit
- Article 29 Working Party Guidelines on DPIAs

===== END FILE =====


===== FILE: skills/implementing-gdpr-data-protection-controls/LICENSE =====

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


===== FILE: skills/implementing-gdpr-data-protection-controls/assets/template.md =====
# GDPR Compliance Audit Checklist

## Organization Information
| Field | Value |
|-------|-------|
| Organization Name | |
| Role | Controller / Processor / Joint Controller |
| DPO Name and Contact | |
| Lead Supervisory Authority | |
| Assessment Date | |
| Assessor | |

---

## Article 5: Data Processing Principles
- [ ] Lawfulness: All processing has documented lawful basis (Art. 6)
- [ ] Fairness: Processing is fair and does not cause unjustified adverse effects
- [ ] Transparency: Privacy notices provided at point of collection (Art. 13/14)
- [ ] Purpose Limitation: Data collected for specified, explicit, and legitimate purposes
- [ ] Data Minimization: Only data necessary for the purpose is collected
- [ ] Accuracy: Processes exist to keep personal data accurate and up to date
- [ ] Storage Limitation: Retention periods defined and enforced for all data categories
- [ ] Integrity and Confidentiality: Technical and organizational security measures in place
- [ ] Accountability: Ability to demonstrate compliance with all principles

## Article 6: Lawful Basis
- [ ] Lawful basis identified for each processing activity
- [ ] Consent is freely given, specific, informed, and unambiguous where used
- [ ] Consent withdrawal mechanism available and easy to use
- [ ] Legitimate interest assessments documented where Art. 6(1)(f) relied upon
- [ ] Legal bases recorded in ROPA

## Articles 13-14: Transparency
- [ ] Privacy notice provided at time of data collection (Art. 13)
- [ ] Privacy notice provided when data obtained indirectly (Art. 14)
- [ ] Notices include: controller identity, purposes, lawful basis, recipients, retention, rights, DPO contact
- [ ] Notices are concise, transparent, intelligible, and in plain language
- [ ] Notices available in appropriate languages

## Articles 15-22: Data Subject Rights
- [ ] Process for receiving and handling DSRs documented
- [ ] Identity verification procedure before fulfilling requests
- [ ] Response within one month (extendable by two months for complex requests)
- [ ] Right of access (Art. 15): can provide copy of personal data
- [ ] Right to rectification (Art. 16): can correct inaccurate data
- [ ] Right to erasure (Art. 17): can delete data across all systems including backups
- [ ] Right to restriction (Art. 18): can restrict processing when contested
- [ ] Right to portability (Art. 20): can export data in machine-readable format
- [ ] Right to object (Art. 21): can cease processing when objected to
- [ ] Automated decision-making (Art. 22): safeguards for solely automated decisions

## Article 25: Data Protection by Design and Default
- [ ] Privacy considerations integrated into system design processes
- [ ] Default settings are most privacy-protective
- [ ] Only personal data necessary for each purpose is processed by default
- [ ] Data protection integrated into development lifecycle

## Article 28: Processors
- [ ] All processors identified and documented
- [ ] Data Processing Agreements (DPAs) in place with all processors
- [ ] DPAs include required Art. 28 provisions
- [ ] Processor security measures verified
- [ ] Sub-processor notification process in place

## Article 30: Records of Processing Activities (ROPA)
- [ ] ROPA maintained and up to date
- [ ] All processing activities documented
- [ ] Controller details, purposes, data categories, recipients, transfers, retention, security measures recorded
- [ ] Available for supervisory authority on request

## Article 32: Security of Processing
- [ ] Risk-appropriate technical measures:
  - [ ] Encryption of personal data (at rest and in transit)
  - [ ] Pseudonymization implemented where appropriate
  - [ ] Access controls and authentication
  - [ ] Logging and monitoring of access to personal data
  - [ ] Data loss prevention controls
- [ ] Risk-appropriate organizational measures:
  - [ ] Information security policies
  - [ ] Staff training on data protection
  - [ ] Confidentiality agreements
  - [ ] Access review processes
- [ ] Ability to restore availability and access after incident
- [ ] Regular testing and evaluation of security measures

## Articles 33-34: Breach Notification
- [ ] Breach detection and assessment procedures documented
- [ ] 72-hour notification to supervisory authority process in place
- [ ] Data subject notification process for high-risk breaches
- [ ] Breach register maintained
- [ ] Breach response plan tested within last 12 months

## Article 35: Data Protection Impact Assessment
- [ ] DPIA criteria documented (when DPIA is required)
- [ ] DPIA process documented
- [ ] DPIAs conducted for all high-risk processing
- [ ] DPO consulted on DPIAs
- [ ] DPIAs reviewed when processing changes

## Articles 44-49: International Transfers
- [ ] All international transfers identified and documented
- [ ] Transfer mechanisms in place (adequacy, SCCs, BCRs)
- [ ] Transfer Impact Assessments conducted for non-adequate countries
- [ ] Supplementary measures implemented where required
- [ ] Standard Contractual Clauses (new 2021 modular version) executed

## Articles 37-39: Data Protection Officer
- [ ] DPO appointed (if required: public authority, core activity large-scale monitoring, core activity special categories)
- [ ] DPO has expert knowledge of data protection law
- [ ] DPO involved in all data protection matters
- [ ] DPO reports to highest management level
- [ ] DPO contact details published and communicated to supervisory authority

---

## Summary

| GDPR Area | Items | Compliant | Non-Compliant | N/A |
|-----------|-------|-----------|---------------|-----|
| Principles (Art. 5) | | | | |
| Lawful Basis (Art. 6) | | | | |
| Transparency (Art. 13-14) | | | | |
| Data Subject Rights (Art. 15-22) | | | | |
| Privacy by Design (Art. 25) | | | | |
| Processors (Art. 28) | | | | |
| ROPA (Art. 30) | | | | |
| Security (Art. 32) | | | | |
| Breach Notification (Art. 33-34) | | | | |
| DPIA (Art. 35) | | | | |
| International Transfers (Art. 44-49) | | | | |
| DPO (Art. 37-39) | | | | |
| **Total** | | | | |

## Sign-off
| Role | Name | Signature | Date |
|------|------|-----------|------|
| DPO | | | |
| CISO | | | |
| Legal Counsel | | | |
| Senior Management | | | |

===== END FILE =====


===== FILE: skills/implementing-gdpr-data-protection-controls/references/api-reference.md =====
# API Reference: Implementing GDPR Data Protection Controls

## Key GDPR Articles

| Article | Requirement | Technical Control |
|---------|-------------|-------------------|
| Art 5 | Processing principles | Data minimization, retention policies |
| Art 25 | Privacy by design | Default privacy settings |
| Art 30 | Records of processing | ROPA documentation system |
| Art 32 | Security of processing | Encryption, access controls, testing |
| Art 33 | Breach notification | 72-hour DPA notification |
| Art 35 | DPIA | Impact assessment for high-risk processing |

## Data Subject Rights (Art 12-22)

| Right | Article | SLA |
|-------|---------|-----|
| Access | Art 15 | 1 month |
| Rectification | Art 16 | 1 month |
| Erasure | Art 17 | 1 month |
| Portability | Art 20 | 1 month |
| Object | Art 21 | Without undue delay |

## PII Detection Patterns

```python
import re
patterns = {
    "email": r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b",
    "iban": r"\b[A-Z]{2}\d{2}[A-Z0-9]{11,30}\b",
    "ip_address": r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b",
}
```

## ROPA Required Fields (Art 30)

| Field | Description |
|-------|-------------|
| controller_name | Data controller identity |
| purposes | Processing purposes |
| data_categories | Types of personal data |
| data_subjects | Categories of data subjects |
| recipients | Data recipients |
| transfers | Cross-border transfers |
| retention_periods | Data retention schedules |
| security_measures | Art 32 controls |

## Cross-Border Transfer Mechanisms (Art 44-49)

| Mechanism | Use Case |
|-----------|----------|
| Adequacy Decision | Transfer to adequate countries (Art 45) |
| Standard Contractual Clauses (SCCs) | Most common mechanism (Art 46) |
| Binding Corporate Rules (BCRs) | Intra-group transfers (Art 47) |
| Derogations | Consent, contract necessity (Art 49) |

### References

- GDPR Official Text: https://gdpr-info.eu/
- EDPB Guidelines: https://edpb.europa.eu/our-work-tools/general-guidance/guidelines-recommendations-best-practices_en
- ICO GDPR Guide: https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/

===== END FILE =====


===== FILE: skills/implementing-gdpr-data-protection-controls/references/standards.md =====
# GDPR Standards Reference

## Primary Legislation

### Regulation (EU) 2016/679 - General Data Protection Regulation
- **Adopted**: April 14, 2016
- **Effective**: May 25, 2018
- **Scope**: Applies to any organization processing personal data of EU/EEA residents
- **Chapters**: 11 chapters, 99 articles, 173 recitals
- **Enforcement**: Supervisory authorities in each EU member state
- **Penalties**: Up to EUR 20 million or 4% of annual global turnover (whichever is greater)

## Key Articles Reference

### Chapter II - Principles (Articles 5-11)
- **Art. 5**: Core processing principles (lawfulness, fairness, transparency, purpose limitation, data minimization, accuracy, storage limitation, integrity and confidentiality, accountability)
- **Art. 6**: Six lawful bases for processing
- **Art. 7**: Conditions for consent
- **Art. 8**: Child's consent (minimum age varies by member state, 13-16)
- **Art. 9**: Special categories of data (health, biometric, genetic, racial/ethnic, political, religious, trade union, sexual orientation)
- **Art. 10**: Criminal conviction data

### Chapter III - Rights of the Data Subject (Articles 12-23)
- **Art. 12**: Transparent communication (one month response deadline)
- **Art. 13**: Information for direct collection
- **Art. 14**: Information for indirect collection
- **Art. 15**: Right of access (copy of data, processing purposes, recipients, retention periods, safeguards for transfers)
- **Art. 16**: Right to rectification
- **Art. 17**: Right to erasure (applies when: consent withdrawn, purpose fulfilled, unlawful processing, legal obligation)
- **Art. 18**: Right to restriction of processing
- **Art. 20**: Right to data portability (structured, commonly used, machine-readable format)
- **Art. 21**: Right to object (especially direct marketing - absolute right)
- **Art. 22**: Automated individual decision-making including profiling

### Chapter IV - Controller and Processor (Articles 24-43)
- **Art. 24**: Responsibility of the controller
- **Art. 25**: Data protection by design and by default
- **Art. 26**: Joint controllers
- **Art. 28**: Processor (DPA requirements: subject-matter, duration, nature/purpose, personal data types, data subject categories, controller obligations/rights)
- **Art. 30**: Records of processing activities
- **Art. 32**: Security of processing
- **Art. 33**: Notification to supervisory authority (72 hours)
- **Art. 34**: Communication to data subject (when high risk to rights and freedoms)
- **Art. 35**: Data Protection Impact Assessment (DPIA)
- **Art. 36**: Prior consultation with supervisory authority
- **Art. 37-39**: Data Protection Officer

### Chapter V - International Transfers (Articles 44-49)
- **Art. 45**: Adequacy decision (EU Commission determines adequate countries)
- **Art. 46**: Appropriate safeguards (SCCs, BCRs, codes of conduct, certification)
- **Art. 47**: Binding Corporate Rules
- **Art. 49**: Derogations (explicit consent, contract, public interest)

## Supporting Standards and Guidance

### ISO/IEC 27701:2019
- Privacy Information Management System (PIMS) extension to ISO 27001
- Maps GDPR requirements to ISO management system controls
- Provides controller and processor-specific guidance

### EDPB Guidelines
- Guidelines on Data Protection Impact Assessment (WP 248)
- Guidelines on Data Breach Notification (WP 250)
- Guidelines on Consent (updated 2020)
- Guidelines on International Data Transfers (post-Schrems II)
- Guidelines on Data Protection by Design and Default (04/2019)

### Transfer Mechanisms Post-Schrems II
- **Standard Contractual Clauses (SCCs)**: New modular SCCs adopted June 2021
  - Module 1: Controller to Controller
  - Module 2: Controller to Processor
  - Module 3: Processor to Processor
  - Module 4: Processor to Controller
- **Transfer Impact Assessment (TIA)**: Required to supplement SCCs
- **Supplementary Measures**: Technical (encryption, pseudonymization), contractual, organizational
- **EU-US Data Privacy Framework**: Adequacy decision adopted July 2023

### DPIA Criteria (Article 35(3) and EDPB)
DPIA required when processing involves:
1. Systematic and extensive evaluation of personal aspects (profiling)
2. Large-scale processing of special categories or criminal data
3. Systematic monitoring of publicly accessible areas
4. New technologies with potential high risk
5. Large-scale data processing
6. Matching or combining datasets
7. Data concerning vulnerable subjects
8. Innovative use of biometric data
9. Data transfers outside EU without adequacy
10. Processing that prevents data subjects from exercising rights

## Supervisory Authorities
| Country | Authority | Website |
|---------|-----------|---------|
| EU-wide | European Data Protection Board (EDPB) | edpb.europa.eu |
| France | CNIL | cnil.fr |
| Germany | BfDI (Federal), State DPAs | bfdi.bund.de |
| Ireland | DPC | dataprotection.ie |
| Netherlands | Autoriteit Persoonsgegevens | autoriteitpersoonsgegevens.nl |
| Spain | AEPD | aepd.es |
| Italy | Garante | garanteprivacy.it |
| UK | ICO (UK GDPR post-Brexit) | ico.org.uk |

## Key Enforcement Decisions (Benchmark)
- Meta (Ireland DPC, 2023): EUR 1.2 billion - Transfers to US without adequate safeguards
- Amazon (Luxembourg CNPD, 2021): EUR 746 million - Advertising targeting
- WhatsApp (Ireland DPC, 2021): EUR 225 million - Transparency failures
- Google (CNIL, 2022): EUR 150 million - Cookie consent
- H&M (Hamburg DPA, 2020): EUR 35.3 million - Employee surveillance

===== END FILE =====


===== FILE: skills/implementing-gdpr-data-protection-controls/references/workflows.md =====
# GDPR Data Protection Control Workflows

## Workflow 1: Data Subject Request (DSR) Handling

```
Start
  |
  v
[Receive DSR from Data Subject]
  - Via email, web form, phone, in-person
  - Record receipt timestamp (30-day clock starts)
  |
  v
[Verify Identity of Requestor]
  - Request additional identification if needed
  - Clock pauses until identity verified
  |
  v
[Classify Request Type]
  |
  +--> Access Request (Art. 15) --> Locate all personal data
  +--> Rectification (Art. 16) --> Identify incorrect data
  +--> Erasure (Art. 17) --> Verify grounds for erasure
  +--> Restriction (Art. 18) --> Flag data for restriction
  +--> Portability (Art. 20) --> Export in machine-readable format
  +--> Objection (Art. 21) --> Assess processing basis
  |
  v
[Check for Exemptions]
  - Legal obligation to retain
  - Freedom of expression
  - Public health
  - Archiving in public interest
  - Legal claims
  |
  v
[Execute Request Across All Systems]
  - Production databases
  - Backups and archives
  - Third-party processors
  - Cloud services
  - Analytics platforms
  |
  v
[Document Action Taken]
  |
  v
[Respond to Data Subject within 30 days]
  - Extension to 60 additional days if complex (notify subject)
  |
  v
End
```

## Workflow 2: Data Breach Notification (Art. 33-34)

```
Start
  |
  v
[Breach Detected or Reported]
  - Technical detection (SIEM, DLP, IDS)
  - Employee report
  - External notification
  - Third-party processor notification
  |
  v
[72-hour Clock Starts]
  |
  v
[Assess Breach Severity]
  - Number of data subjects affected
  - Types of personal data compromised
  - Special categories involved?
  - Risk to rights and freedoms
  |
  v
[Determine Notification Requirements]
  |
  +--> [Risk to Rights and Freedoms?]
        |
        +--> No Risk --> Document in breach register only
        |
        +--> Risk exists --> Notify Supervisory Authority (72 hours)
        |                    |
        |                    v
        +--> High Risk --> Notify Supervisory Authority (72 hours)
                           AND notify affected data subjects
  |
  v
[Prepare Supervisory Authority Notification]
  - Nature of the breach
  - Categories and approximate number of data subjects
  - Categories and approximate number of records
  - Name and contact details of DPO
  - Likely consequences of the breach
  - Measures taken or proposed to address the breach
  |
  v
[Submit Notification to Lead Supervisory Authority]
  |
  v
[If High Risk: Notify Data Subjects]
  - Clear and plain language
  - Description of breach
  - DPO contact information
  - Likely consequences
  - Measures taken to mitigate
  |
  v
[Conduct Post-Breach Review]
  - Root cause analysis
  - Control improvements
  - Update breach register
  |
  v
End
```

## Workflow 3: Data Protection Impact Assessment (DPIA)

```
Start
  |
  v
[Identify Processing Activity]
  |
  v
[Screen Against DPIA Criteria]
  - Profiling or automated decision-making?
  - Large-scale special category data?
  - Systematic monitoring of public areas?
  - New technology application?
  - Cross-border data transfer?
  - Vulnerable data subjects?
  |
  +--> [No DPIA triggers] --> Document screening decision --> End
  |
  +--> [DPIA required]
        |
        v
[Describe Processing Operation]
  - Purpose and scope
  - Data elements collected
  - Data subjects categories
  - Recipients and transfers
  - Retention period
  - Technology used
  |
  v
[Assess Necessity and Proportionality]
  - Is processing necessary for the purpose?
  - Could purpose be achieved with less data?
  - Is lawful basis appropriate?
  - Are data subject rights supported?
  |
  v
[Identify and Assess Risks]
  - Risks to confidentiality
  - Risks to integrity
  - Risks to availability
  - Risks to rights and freedoms
  - Likelihood and severity of each risk
  |
  v
[Identify Mitigation Measures]
  - Technical measures (encryption, pseudonymization, access controls)
  - Organizational measures (policies, training, DPO oversight)
  - Contractual measures (DPAs, SCCs)
  |
  v
[Determine Residual Risk]
  |
  +--> [Residual Risk Acceptable] --> Approve and proceed
  |
  +--> [Residual Risk High] --> Consult DPO
  |                             |
  |                             +--> [Can mitigate further] --> Add measures
  |                             |
  |                             +--> [Cannot mitigate] --> Prior consultation
  |                                    with Supervisory Authority (Art. 36)
  |
  v
[Document DPIA]
  - Keep under review
  - Reassess when processing changes
  |
  v
End
```

## Workflow 4: International Data Transfer Assessment

```
Start
  |
  v
[Identify Cross-Border Transfer]
  - Data flowing outside EEA
  - Cloud services in non-EEA regions
  - Group company data sharing
  - Vendor/processor locations
  |
  v
[Check Adequacy Decision]
  - Is destination country on EU adequacy list?
  - (Andorra, Argentina, Canada, Faroe Islands, Guernsey,
     Israel, Isle of Man, Japan, Jersey, New Zealand, Republic
     of Korea, Switzerland, UK, Uruguay, US under DPF)
  |
  +--> [Adequate] --> Document and proceed
  |
  +--> [Not Adequate]
        |
        v
      [Select Transfer Mechanism]
        |
        +--> Standard Contractual Clauses (SCCs)
        +--> Binding Corporate Rules (BCRs)
        +--> Approved Code of Conduct
        +--> Certification Mechanism
        +--> Derogations (Art. 49) - limited circumstances
        |
        v
      [Conduct Transfer Impact Assessment (TIA)]
        - Laws of destination country
        - Government surveillance powers
        - Data protection standards
        - Access by public authorities
        |
        v
      [Identify Supplementary Measures]
        - Technical: encryption with EEA-held keys
        - Contractual: additional processor obligations
        - Organizational: policies limiting access
        |
        v
      [Document Transfer Mechanism]
        - Signed SCCs with correct module
        - TIA findings and conclusions
        - Supplementary measures implemented
  |
  v
End
```

## Workflow 5: Records of Processing Activities (ROPA)

```
Start
  |
  v
[Identify All Processing Activities]
  - Interview business units
  - Review system inventory
  - Analyze data flows
  - Check vendor agreements
  |
  v
[For Each Processing Activity Document:]
  |
  v
[Controller Record (Art. 30(1))]
  - Name and contact details of controller (and DPO)
  - Purposes of processing
  - Categories of data subjects
  - Categories of personal data
  - Categories of recipients
  - Transfers to third countries (safeguards)
  - Retention periods
  - Technical and organizational security measures
  |
  v
[Processor Record (Art. 30(2))]
  - Name and contact details of processor and controller
  - Categories of processing carried out
  - Transfers to third countries (safeguards)
  - Technical and organizational security measures
  |
  v
[Maintain and Update ROPA]
  - Review quarterly or when processing changes
  - Update for new systems, vendors, purposes
  - Make available to supervisory authority on request
  |
  v
End
```

===== END FILE =====


===== FILE: skills/implementing-gdpr-data-protection-controls/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for assessing and managing GDPR data protection compliance."""

import json
import argparse
import re
from datetime import datetime
from collections import Counter


GDPR_ARTICLES = {
    "Art5": "Principles of processing",
    "Art6": "Lawful basis",
    "Art13": "Information to data subject (direct collection)",
    "Art14": "Information to data subject (indirect collection)",
    "Art15": "Right of access",
    "Art17": "Right to erasure",
    "Art20": "Right to data portability",
    "Art25": "Data protection by design and by default",
    "Art30": "Records of processing activities",
    "Art32": "Security of processing",
    "Art33": "Breach notification (72h to DPA)",
    "Art34": "Breach communication to data subjects",
    "Art35": "Data Protection Impact Assessment",
    "Art44": "Cross-border transfer principles",
}

PII_PATTERNS = {
    "email": r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b",
    "phone_eu": r"\b\+?[0-9]{1,3}[\s.-]?[0-9]{6,14}\b",
    "iban": r"\b[A-Z]{2}\d{2}[A-Z0-9]{11,30}\b",
    "ip_address": r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b",
    "date_of_birth": r"\b\d{2}[/.-]\d{2}[/.-]\d{4}\b",
    "national_id": r"\b\d{3}-\d{2}-\d{4}\b",
    "passport": r"\b[A-Z]{1,2}\d{6,9}\b",
}


def scan_for_pii(file_path, max_bytes=1024 * 1024):
    """Scan a file for GDPR-relevant personal data patterns."""
    try:
        with open(file_path, "r", encoding="utf-8", errors="ignore") as f:
            content = f.read(max_bytes)
    except (OSError, PermissionError):
        return None
    matches = {}
    for pii_type, pattern in PII_PATTERNS.items():
        found = re.findall(pattern, content)
        if found:
            matches[pii_type] = len(found)
    if not matches:
        return None
    return {"file": str(file_path), "pii_types": matches,
            "total_matches": sum(matches.values()),
            "category": classify_data_category(matches)}


def classify_data_category(matches):
    """Classify data into GDPR special categories."""
    if any(k in matches for k in ["national_id", "passport"]):
        return "special_category"
    if "iban" in matches:
        return "financial"
    return "standard_personal_data"


def assess_ropa_completeness(ropa_json):
    """Assess Records of Processing Activities (ROPA) completeness per Art 30."""
    with open(ropa_json) as f:
        ropa = json.load(f)
    required_fields = [
        "controller_name", "purposes", "data_categories", "data_subjects",
        "recipients", "transfers", "retention_periods", "security_measures",
    ]
    findings = []
    activities = ropa if isinstance(ropa, list) else ropa.get("activities", [])
    for activity in activities:
        missing = [f for f in required_fields if not activity.get(f)]
        findings.append({
            "activity": activity.get("name", activity.get("purpose", "unknown")),
            "complete": len(missing) == 0,
            "missing_fields": missing,
            "compliance": "COMPLIANT" if not missing else "NON_COMPLIANT",
        })
    total = len(findings)
    compliant = sum(1 for f in findings if f["compliance"] == "COMPLIANT")
    return {
        "activities_assessed": total,
        "compliant": compliant,
        "compliance_rate": round(compliant / total * 100, 1) if total else 0,
        "details": findings,
    }


def assess_dsr_handling(dsr_log_path):
    """Assess Data Subject Request handling compliance."""
    with open(dsr_log_path) as f:
        requests_log = json.load(f)
    dsrs = requests_log if isinstance(requests_log, list) else requests_log.get("requests", [])
    findings = []
    for dsr in dsrs:
        received = datetime.fromisoformat(dsr.get("received_date", "2024-01-01"))
        completed = dsr.get("completed_date")
        if completed:
            completed_dt = datetime.fromisoformat(completed)
            days = (completed_dt - received).days
        else:
            days = (datetime.utcnow() - received).days
        overdue = days > 30  # GDPR requires response within one month
        findings.append({
            "request_id": dsr.get("id", ""),
            "type": dsr.get("type", dsr.get("right", "")),
            "status": dsr.get("status", "pending"),
            "days_elapsed": days,
            "overdue": overdue,
            "severity": "HIGH" if overdue else "INFO",
        })
    overdue_count = sum(1 for f in findings if f["overdue"])
    return {
        "total_requests": len(findings),
        "overdue": overdue_count,
        "by_type": dict(Counter(f["type"] for f in findings)),
        "details": findings,
    }


def assess_breach_notification_readiness(breach_log_path):
    """Assess breach notification compliance (Art 33/34)."""
    with open(breach_log_path) as f:
        breaches = json.load(f)
    breach_list = breaches if isinstance(breaches, list) else breaches.get("breaches", [])
    findings = []
    for breach in breach_list:
        detected = datetime.fromisoformat(breach.get("detected", "2024-01-01"))
        notified = breach.get("dpa_notified")
        if notified:
            notified_dt = datetime.fromisoformat(notified)
            hours = (notified_dt - detected).total_seconds() / 3600
        else:
            hours = None
        compliant = hours is not None and hours <= 72
        findings.append({
            "breach_id": breach.get("id", ""),
            "detected": str(detected),
            "notification_hours": round(hours, 1) if hours else None,
            "art33_compliant": compliant,
            "data_subjects_affected": breach.get("subjects_affected", 0),
            "severity": breach.get("severity", "HIGH"),
        })
    return {"total_breaches": len(findings), "details": findings,
            "art33_compliance_rate": round(
                sum(1 for f in findings if f["art33_compliant"]) / len(findings) * 100, 1
            ) if findings else 0}


def generate_art32_checklist():
    """Generate Article 32 security measures compliance checklist."""
    return {
        "encryption": {
            "data_at_rest": {"required": True, "standard": "AES-256"},
            "data_in_transit": {"required": True, "standard": "TLS 1.2+"},
            "key_management": {"required": True, "standard": "HSM or KMS"},
        },
        "pseudonymization": {
            "tokenization": {"required": True},
            "key_separation": {"required": True},
        },
        "access_controls": {
            "rbac": {"required": True},
            "mfa": {"required": True},
            "least_privilege": {"required": True},
            "access_reviews": {"required": True, "frequency": "quarterly"},
        },
        "resilience": {
            "backup_strategy": {"required": True},
            "disaster_recovery": {"required": True, "rto": "4h", "rpo": "1h"},
            "business_continuity": {"required": True},
        },
        "testing": {
            "penetration_testing": {"required": True, "frequency": "annual"},
            "vulnerability_scanning": {"required": True, "frequency": "monthly"},
            "security_awareness": {"required": True, "frequency": "annual"},
        },
    }


def main():
    parser = argparse.ArgumentParser(description="GDPR Data Protection Agent")
    parser.add_argument("--scan-pii", help="File or directory to scan for PII")
    parser.add_argument("--ropa", help="ROPA JSON file for completeness check")
    parser.add_argument("--dsr-log", help="Data Subject Request log JSON")
    parser.add_argument("--breach-log", help="Breach notification log JSON")
    parser.add_argument("--action", choices=["scan", "ropa", "dsr", "breach",
                                              "art32", "full"], default="full")
    parser.add_argument("--output", default="gdpr_compliance_report.json")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "results": {}}

    if args.action in ("scan", "full") and args.scan_pii:
        result = scan_for_pii(args.scan_pii)
        report["results"]["pii_scan"] = result
        if result:
            print(f"[+] PII found: {result['total_matches']} matches ({result['category']})")
        else:
            print("[+] No PII detected")

    if args.action in ("ropa", "full") and args.ropa:
        result = assess_ropa_completeness(args.ropa)
        report["results"]["ropa"] = result
        print(f"[+] ROPA compliance: {result['compliance_rate']}%")

    if args.action in ("dsr", "full") and args.dsr_log:
        result = assess_dsr_handling(args.dsr_log)
        report["results"]["dsr"] = result
        print(f"[+] DSRs: {result['total_requests']} total, {result['overdue']} overdue")

    if args.action in ("breach", "full") and args.breach_log:
        result = assess_breach_notification_readiness(args.breach_log)
        report["results"]["breach"] = result
        print(f"[+] Art 33 compliance rate: {result['art33_compliance_rate']}%")

    if args.action in ("art32", "full"):
        checklist = generate_art32_checklist()
        report["results"]["art32_checklist"] = checklist
        print("[+] Article 32 security checklist generated")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-gdpr-data-protection-controls/scripts/process.py =====
#!/usr/bin/env python3
"""
GDPR Data Protection Compliance Automation

Automates data mapping, ROPA creation, DPIA assessment, data subject
request tracking, breach notification management, and compliance reporting.
"""

import json
import csv
import os
from datetime import datetime, timedelta
from pathlib import Path
from dataclasses import dataclass, field, asdict
from enum import Enum
from typing import Optional


class LawfulBasis(Enum):
    CONSENT = "Consent (Art. 6(1)(a))"
    CONTRACT = "Contract (Art. 6(1)(b))"
    LEGAL_OBLIGATION = "Legal Obligation (Art. 6(1)(c))"
    VITAL_INTERESTS = "Vital Interests (Art. 6(1)(d))"
    PUBLIC_TASK = "Public Task (Art. 6(1)(e))"
    LEGITIMATE_INTEREST = "Legitimate Interest (Art. 6(1)(f))"


class DSRType(Enum):
    ACCESS = "Right of Access (Art. 15)"
    RECTIFICATION = "Right to Rectification (Art. 16)"
    ERASURE = "Right to Erasure (Art. 17)"
    RESTRICTION = "Right to Restriction (Art. 18)"
    PORTABILITY = "Right to Data Portability (Art. 20)"
    OBJECTION = "Right to Object (Art. 21)"
    AUTOMATED = "Automated Decision-Making (Art. 22)"


class BreachSeverity(Enum):
    NO_RISK = "No Risk"
    RISK = "Risk to Rights and Freedoms"
    HIGH_RISK = "High Risk to Rights and Freedoms"


class TransferMechanism(Enum):
    ADEQUACY = "Adequacy Decision (Art. 45)"
    SCCS = "Standard Contractual Clauses (Art. 46(2)(c))"
    BCRS = "Binding Corporate Rules (Art. 47)"
    CONSENT = "Explicit Consent (Art. 49(1)(a))"
    CONTRACT = "Necessary for Contract (Art. 49(1)(b))"
    DEROGATION = "Other Derogation (Art. 49)"


ADEQUATE_COUNTRIES = [
    "Andorra", "Argentina", "Canada", "Faroe Islands", "Guernsey",
    "Israel", "Isle of Man", "Japan", "Jersey", "New Zealand",
    "Republic of Korea", "Switzerland", "United Kingdom", "Uruguay",
    "United States (Data Privacy Framework participants)"
]


@dataclass
class ProcessingActivity:
    activity_id: str
    name: str
    purpose: str
    lawful_basis: str
    data_subjects: list
    personal_data_categories: list
    special_categories: bool = False
    recipients: list = field(default_factory=list)
    international_transfers: list = field(default_factory=list)
    retention_period: str = ""
    systems: list = field(default_factory=list)
    security_measures: list = field(default_factory=list)
    dpia_required: bool = False
    controller_name: str = ""
    processor_name: str = ""


@dataclass
class DataSubjectRequest:
    dsr_id: str
    request_type: str
    data_subject_name: str
    data_subject_email: str
    received_date: str
    identity_verified: bool = False
    identity_verified_date: str = ""
    deadline: str = ""
    status: str = "Received"
    systems_searched: list = field(default_factory=list)
    response_date: str = ""
    notes: str = ""

    def __post_init__(self):
        if not self.deadline and self.received_date:
            received = datetime.strptime(self.received_date, "%Y-%m-%d")
            self.deadline = (received + timedelta(days=30)).strftime("%Y-%m-%d")


@dataclass
class BreachRecord:
    breach_id: str
    detected_date: str
    detected_time: str
    description: str
    data_categories: list
    subjects_affected: int
    severity: str
    notification_deadline: str = ""
    authority_notified: bool = False
    authority_notification_date: str = ""
    subjects_notified: bool = False
    subjects_notification_date: str = ""
    root_cause: str = ""
    remediation: str = ""
    status: str = "Investigating"

    def __post_init__(self):
        if not self.notification_deadline and self.detected_date:
            detected = datetime.strptime(self.detected_date, "%Y-%m-%d")
            self.notification_deadline = (detected + timedelta(hours=72)).strftime("%Y-%m-%d %H:%M")


@dataclass
class DPIARecord:
    dpia_id: str
    processing_activity: str
    description: str
    necessity_assessment: str
    risks_identified: list
    mitigation_measures: list
    residual_risk: str
    dpo_consultation: bool = False
    authority_consultation: bool = False
    approved: bool = False
    approval_date: str = ""
    reviewer: str = ""


class GDPRComplianceManager:
    """Manages GDPR compliance controls and tracking."""

    def __init__(self, output_dir: str = "./gdpr_output"):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.processing_activities: list[ProcessingActivity] = []
        self.dsr_log: list[DataSubjectRequest] = []
        self.breach_register: list[BreachRecord] = []
        self.dpia_register: list[DPIARecord] = []

    def create_ropa(self, activities: list[dict]) -> list[ProcessingActivity]:
        """Create Records of Processing Activities (Art. 30)."""
        print("\n" + "=" * 70)
        print("RECORDS OF PROCESSING ACTIVITIES (ROPA)")
        print("=" * 70)

        for act_data in activities:
            activity = ProcessingActivity(**act_data)
            self.processing_activities.append(activity)
            print(f"\n  [{activity.activity_id}] {activity.name}")
            print(f"    Purpose: {activity.purpose}")
            print(f"    Lawful Basis: {activity.lawful_basis}")
            print(f"    Data Subjects: {', '.join(activity.data_subjects)}")
            print(f"    Special Categories: {'Yes' if activity.special_categories else 'No'}")
            print(f"    International Transfers: {'Yes' if activity.international_transfers else 'No'}")
            print(f"    DPIA Required: {'Yes' if activity.dpia_required else 'No'}")

        # Summary statistics
        total = len(self.processing_activities)
        special = sum(1 for a in self.processing_activities if a.special_categories)
        transfers = sum(1 for a in self.processing_activities if a.international_transfers)
        dpia_needed = sum(1 for a in self.processing_activities if a.dpia_required)

        print(f"\n  ROPA Summary:")
        print(f"    Total Processing Activities: {total}")
        print(f"    Special Category Processing: {special}")
        print(f"    International Transfers: {transfers}")
        print(f"    DPIA Required: {dpia_needed}")

        # Lawful basis breakdown
        basis_counts = {}
        for a in self.processing_activities:
            basis_counts.setdefault(a.lawful_basis, 0)
            basis_counts[a.lawful_basis] += 1
        print(f"\n  Lawful Basis Breakdown:")
        for basis, count in basis_counts.items():
            print(f"    {basis}: {count}")

        # Save ROPA
        ropa_path = self.output_dir / "ropa.json"
        with open(ropa_path, "w") as f:
            json.dump([asdict(a) for a in self.processing_activities], f, indent=2)

        csv_path = self.output_dir / "ropa.csv"
        with open(csv_path, "w", newline="", encoding="utf-8") as f:
            writer = csv.writer(f)
            writer.writerow([
                "Activity ID", "Name", "Purpose", "Lawful Basis",
                "Data Subjects", "Personal Data Categories", "Special Categories",
                "Recipients", "International Transfers", "Retention Period",
                "Systems", "Security Measures", "DPIA Required"
            ])
            for a in self.processing_activities:
                writer.writerow([
                    a.activity_id, a.name, a.purpose, a.lawful_basis,
                    "; ".join(a.data_subjects), "; ".join(a.personal_data_categories),
                    "Yes" if a.special_categories else "No",
                    "; ".join(a.recipients), "; ".join(a.international_transfers),
                    a.retention_period, "; ".join(a.systems),
                    "; ".join(a.security_measures), "Yes" if a.dpia_required else "No"
                ])

        print(f"\n  ROPA saved to: {ropa_path}")
        return self.processing_activities

    def process_dsr(self, requests: list[dict]) -> dict:
        """Track and manage data subject requests."""
        print("\n" + "=" * 70)
        print("DATA SUBJECT REQUEST TRACKER")
        print("=" * 70)

        for req_data in requests:
            dsr = DataSubjectRequest(**req_data)
            self.dsr_log.append(dsr)
            print(f"\n  [{dsr.dsr_id}] {dsr.request_type}")
            print(f"    Subject: {dsr.data_subject_name}")
            print(f"    Received: {dsr.received_date}")
            print(f"    Deadline: {dsr.deadline}")
            print(f"    Status: {dsr.status}")
            print(f"    Identity Verified: {'Yes' if dsr.identity_verified else 'No'}")

        # Overdue check
        today = datetime.now().strftime("%Y-%m-%d")
        overdue = [d for d in self.dsr_log if d.deadline < today and d.status != "Completed"]
        if overdue:
            print(f"\n  ALERT: {len(overdue)} overdue DSRs!")
            for d in overdue:
                print(f"    [{d.dsr_id}] {d.request_type} - Deadline: {d.deadline}")

        # Summary
        type_counts = {}
        for d in self.dsr_log:
            type_counts.setdefault(d.request_type, 0)
            type_counts[d.request_type] += 1

        print(f"\n  DSR Summary:")
        for rtype, count in type_counts.items():
            print(f"    {rtype}: {count}")
        print(f"    Total: {len(self.dsr_log)}")
        print(f"    Overdue: {len(overdue)}")

        dsr_path = self.output_dir / "dsr_log.json"
        with open(dsr_path, "w") as f:
            json.dump([asdict(d) for d in self.dsr_log], f, indent=2)

        print(f"\n  DSR Log saved to: {dsr_path}")
        return {"total": len(self.dsr_log), "overdue": len(overdue), "by_type": type_counts}

    def manage_breach(self, breaches: list[dict]) -> list[BreachRecord]:
        """Manage breach register and notification tracking."""
        print("\n" + "=" * 70)
        print("BREACH REGISTER AND NOTIFICATION TRACKER")
        print("=" * 70)

        for breach_data in breaches:
            breach = BreachRecord(**breach_data)
            self.breach_register.append(breach)
            print(f"\n  [{breach.breach_id}] {breach.description[:60]}...")
            print(f"    Detected: {breach.detected_date} {breach.detected_time}")
            print(f"    Severity: {breach.severity}")
            print(f"    Subjects Affected: {breach.subjects_affected}")
            print(f"    Notification Deadline: {breach.notification_deadline}")
            print(f"    Authority Notified: {'Yes' if breach.authority_notified else 'No'}")
            if breach.severity == BreachSeverity.HIGH_RISK.value:
                print(f"    Subjects Notified: {'Yes' if breach.subjects_notified else 'No'}")
            print(f"    Status: {breach.status}")

        breach_path = self.output_dir / "breach_register.json"
        with open(breach_path, "w") as f:
            json.dump([asdict(b) for b in self.breach_register], f, indent=2)

        print(f"\n  Breach Register saved to: {breach_path}")
        return self.breach_register

    def conduct_dpia(self, dpias: list[dict]) -> list[DPIARecord]:
        """Conduct and record Data Protection Impact Assessments."""
        print("\n" + "=" * 70)
        print("DATA PROTECTION IMPACT ASSESSMENTS (DPIAs)")
        print("=" * 70)

        for dpia_data in dpias:
            dpia = DPIARecord(**dpia_data)
            self.dpia_register.append(dpia)
            print(f"\n  [{dpia.dpia_id}] {dpia.processing_activity}")
            print(f"    Description: {dpia.description[:60]}...")
            print(f"    Risks Identified: {len(dpia.risks_identified)}")
            print(f"    Mitigation Measures: {len(dpia.mitigation_measures)}")
            print(f"    Residual Risk: {dpia.residual_risk}")
            print(f"    DPO Consulted: {'Yes' if dpia.dpo_consultation else 'No'}")
            print(f"    Approved: {'Yes' if dpia.approved else 'No'}")

        dpia_path = self.output_dir / "dpia_register.json"
        with open(dpia_path, "w") as f:
            json.dump([asdict(d) for d in self.dpia_register], f, indent=2)

        print(f"\n  DPIA Register saved to: {dpia_path}")
        return self.dpia_register

    def assess_compliance(self) -> dict:
        """Generate overall GDPR compliance assessment."""
        print("\n" + "=" * 70)
        print("GDPR COMPLIANCE ASSESSMENT")
        print("=" * 70)

        checks = {
            "Article 5 - Principles": {
                "All processing has documented lawful basis": bool(self.processing_activities),
                "Purpose limitation documented for each activity": False,
                "Data minimization assessed": False,
                "Accuracy procedures in place": False,
                "Retention periods defined": any(a.retention_period for a in self.processing_activities),
                "Security measures documented": any(a.security_measures for a in self.processing_activities),
            },
            "Article 25 - Privacy by Design": {
                "Development processes include privacy reviews": False,
                "Default privacy settings implemented": False,
                "Data minimization built into systems": False,
            },
            "Article 30 - ROPA": {
                "Records of processing maintained": bool(self.processing_activities),
                "Controller details documented": any(a.controller_name for a in self.processing_activities),
                "All processing activities captured": len(self.processing_activities) > 0,
            },
            "Article 32 - Security Measures": {
                "Encryption implemented for personal data": False,
                "Pseudonymization implemented where appropriate": False,
                "Access controls for personal data systems": False,
                "Regular security testing performed": False,
            },
            "Articles 33-34 - Breach Notification": {
                "Breach detection capabilities in place": False,
                "72-hour notification process documented": False,
                "Breach register maintained": bool(self.breach_register),
                "Breach response tested": False,
            },
            "Article 35 - DPIA": {
                "DPIA criteria documented": False,
                "DPIAs conducted for high-risk processing": bool(self.dpia_register),
                "DPO consulted on DPIAs": any(d.dpo_consultation for d in self.dpia_register),
            },
            "Articles 12-22 - Data Subject Rights": {
                "DSR handling process documented": False,
                "30-day response capability": bool(self.dsr_log),
                "Identity verification process": any(d.identity_verified for d in self.dsr_log),
                "Erasure capability across systems": False,
                "Portability export capability": False,
            },
            "Articles 44-49 - International Transfers": {
                "Cross-border transfers mapped": any(a.international_transfers for a in self.processing_activities),
                "Transfer mechanisms in place (SCCs/BCRs)": False,
                "Transfer impact assessments conducted": False,
            },
        }

        total = sum(len(v) for v in checks.values())
        passed = sum(sum(1 for x in v.values() if x) for v in checks.values())
        pct = (passed / total * 100) if total > 0 else 0

        print(f"\n  Compliance Score: {passed}/{total} ({pct:.1f}%)")

        for article, items in checks.items():
            cat_passed = sum(1 for x in items.values() if x)
            cat_total = len(items)
            print(f"\n  {article} ({cat_passed}/{cat_total}):")
            for item, status in items.items():
                icon = "[PASS]" if status else "[FAIL]"
                print(f"    {icon} {item}")

        report = {
            "date": datetime.now().isoformat(),
            "compliance_percentage": pct,
            "checks": checks,
            "ropa_count": len(self.processing_activities),
            "dsr_count": len(self.dsr_log),
            "breach_count": len(self.breach_register),
            "dpia_count": len(self.dpia_register),
        }

        report_path = self.output_dir / "gdpr_compliance_report.json"
        with open(report_path, "w") as f:
            json.dump(report, f, indent=2)

        print(f"\n  Compliance Report saved to: {report_path}")
        return report


def main():
    """Run GDPR compliance assessment."""
    manager = GDPRComplianceManager()

    # ROPA
    sample_activities = [
        {
            "activity_id": "PROC-001",
            "name": "Customer Account Management",
            "purpose": "Managing customer accounts and providing services",
            "lawful_basis": LawfulBasis.CONTRACT.value,
            "data_subjects": ["Customers"],
            "personal_data_categories": ["Name", "Email", "Phone", "Address", "Payment details"],
            "recipients": ["Payment processor", "CRM provider"],
            "international_transfers": ["US (Payment processor - SCCs)"],
            "retention_period": "Duration of contract + 6 years",
            "systems": ["CRM", "Payment gateway", "Database"],
            "security_measures": ["Encryption at rest", "TLS in transit", "RBAC", "MFA"],
            "controller_name": "Example Corp Ltd",
        },
        {
            "activity_id": "PROC-002",
            "name": "Employee HR Processing",
            "purpose": "Employment administration, payroll, benefits",
            "lawful_basis": LawfulBasis.CONTRACT.value,
            "data_subjects": ["Employees", "Job applicants"],
            "personal_data_categories": ["Name", "Address", "DOB", "NI number", "Bank details", "Health data"],
            "special_categories": True,
            "recipients": ["Payroll provider", "Pension provider", "HMRC"],
            "retention_period": "Employment + 7 years",
            "systems": ["HRIS", "Payroll system"],
            "security_measures": ["Encryption", "Access controls", "Audit logging"],
            "dpia_required": True,
            "controller_name": "Example Corp Ltd",
        },
        {
            "activity_id": "PROC-003",
            "name": "Website Analytics",
            "purpose": "Analyzing website usage to improve user experience",
            "lawful_basis": LawfulBasis.CONSENT.value,
            "data_subjects": ["Website visitors"],
            "personal_data_categories": ["IP address", "Cookie identifiers", "Browsing behavior"],
            "recipients": ["Analytics provider"],
            "international_transfers": ["US (Analytics provider - EU-US DPF)"],
            "retention_period": "26 months",
            "systems": ["Website", "Analytics platform"],
            "security_measures": ["IP anonymization", "Cookie consent", "Data pseudonymization"],
            "controller_name": "Example Corp Ltd",
        },
    ]
    manager.create_ropa(sample_activities)

    # DSRs
    sample_dsrs = [
        {
            "dsr_id": "DSR-001",
            "request_type": DSRType.ACCESS.value,
            "data_subject_name": "John Smith",
            "data_subject_email": "john.smith@email.com",
            "received_date": "2024-12-01",
            "identity_verified": True,
            "identity_verified_date": "2024-12-02",
            "status": "In Progress",
            "systems_searched": ["CRM", "Email", "Analytics"],
        },
        {
            "dsr_id": "DSR-002",
            "request_type": DSRType.ERASURE.value,
            "data_subject_name": "Jane Doe",
            "data_subject_email": "jane.doe@email.com",
            "received_date": "2024-12-15",
            "identity_verified": True,
            "identity_verified_date": "2024-12-16",
            "status": "Completed",
            "response_date": "2024-12-28",
        },
    ]
    manager.process_dsr(sample_dsrs)

    # Breaches
    sample_breaches = [
        {
            "breach_id": "BREACH-001",
            "detected_date": "2024-11-15",
            "detected_time": "14:30",
            "description": "Unauthorized access to customer database via compromised admin credentials",
            "data_categories": ["Name", "Email", "Phone"],
            "subjects_affected": 2500,
            "severity": BreachSeverity.HIGH_RISK.value,
            "authority_notified": True,
            "authority_notification_date": "2024-11-16",
            "subjects_notified": True,
            "subjects_notification_date": "2024-11-18",
            "root_cause": "Credential compromise via phishing",
            "remediation": "MFA enforced, passwords reset, phishing training deployed",
            "status": "Closed",
        },
    ]
    manager.manage_breach(sample_breaches)

    # DPIAs
    sample_dpias = [
        {
            "dpia_id": "DPIA-001",
            "processing_activity": "Employee HR Processing",
            "description": "Processing of employee health data for occupational health and benefits administration",
            "necessity_assessment": "Processing is necessary for employment contract and legal obligations",
            "risks_identified": [
                "Unauthorized access to health data",
                "Data breach exposing sensitive employee information",
                "Excessive data collection beyond what is necessary",
            ],
            "mitigation_measures": [
                "Encryption of all health data at rest and in transit",
                "Strict RBAC limiting access to HR and OH staff only",
                "Audit logging of all access to health records",
                "Annual access reviews",
                "Staff training on handling special category data",
            ],
            "residual_risk": "Low",
            "dpo_consultation": True,
            "approved": True,
            "approval_date": "2024-06-15",
            "reviewer": "DPO",
        },
    ]
    manager.conduct_dpia(sample_dpias)

    # Overall compliance assessment
    manager.assess_compliance()

    print("\n" + "=" * 70)
    print("GDPR COMPLIANCE ASSESSMENT COMPLETE")
    print("=" * 70)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-hipaa-security-rule-safeguards/SKILL.md =====
---
name: implementing-hipaa-security-rule-safeguards
description: >-
  Implement the HIPAA Security Rule (45 CFR Part 164 Subpart C) to protect electronic
  protected health information (ePHI): conduct the required risk analysis, deploy the
  administrative, physical, and technical safeguards, handle required vs addressable
  implementation specifications, execute Business Associate Agreements, and stand up
  breach-notification readiness. Use when an organization is a HIPAA covered entity or
  business associate, when protecting ePHI, when preparing for an OCR audit or responding
  to a breach, when performing a HIPAA Security Risk Analysis, when drafting or reviewing a
  BAA, or when mapping security controls to the §164.308/310/312/314/316 safeguards. Notes
  the 2025 NPRM proposed changes (not yet final). Keywords: HIPAA, HIPAA Security Rule,
  ePHI, PHI, 45 CFR 164, risk analysis, administrative safeguards, physical safeguards,
  technical safeguards, addressable, required, Business Associate Agreement, BAA, OCR,
  breach notification, HITECH, covered entity, business associate.
domain: cybersecurity
subdomain: compliance-governance
tags:
- hipaa
- hipaa-security-rule
- ephi
- phi
- 45-cfr-164
- risk-analysis
- baa
- breach-notification
- ocr
- compliance
- governance
version: "1.0"
author: andrewibrah
license: Apache-2.0
nist_csf:
- GV.OC-03
- GV.RM-01
- ID.RA-01
- ID.RA-05
- PR.DS-01
- PR.AA-01
- DE.CM-01
mitre_attack:
- T1078
- T1566
- T1486
- T1530
- T1048
---

# Implementing HIPAA Security Rule Safeguards

## When to Use

- When an organization is a **covered entity** (health plan, clearinghouse, or provider transmitting electronic transactions) or a **business associate** handling **ePHI** on their behalf.
- When standing up or maturing controls to protect **electronic protected health information**.
- When performing the mandatory **HIPAA Security Risk Analysis** (§164.308(a)(1)(ii)(A)) — the single most-cited gap in OCR enforcement.
- When preparing for an **OCR audit/investigation** or responding to a suspected **breach**.
- When drafting, reviewing, or remediating a **Business Associate Agreement (BAA)**.
- When mapping existing security controls to the HIPAA safeguard standards and implementation specifications.

> Scope note: this skill covers the **Security Rule** (ePHI). The **Privacy Rule** (uses/disclosures of all PHI) and the **Breach Notification Rule** are related but distinct; this skill touches breach readiness and BAAs where they intersect security.

## Prerequisites

- A clear determination of the organization's **role** (covered entity vs business associate) and where ePHI lives, flows, and is stored (an ePHI data map).
- An **asset inventory** of systems that create, receive, maintain, or transmit ePHI.
- Knowledge of the current rule's structure (45 CFR §§164.302–318) and the **required vs addressable** distinction.
- Awareness that a **2025 NPRM** proposes significant changes (see Workflow step 7 and `references/standards.md`) — track but do not assume them as in force.

## Workflow

### 1. Conduct the Security Risk Analysis (§164.308(a)(1)(ii)(A))
This is **required** and foundational. Inventory ePHI and systems, identify threats and vulnerabilities, assess current controls, determine likelihood and impact, and assign risk levels. (Pair with the NIST 800-30 methodology and HHS's SRA Tool.) Output is a documented, dated risk analysis — the artifact OCR asks for first.

### 2. Implement Administrative Safeguards (§164.308)
The largest section. Includes the **Security Management Process** (risk analysis, risk management, sanction policy, information-system activity review), assigned **security responsibility** (a named Security Official), **workforce security**, **information access management**, **security awareness and training**, **security incident procedures**, **contingency planning** (data backup, disaster recovery, emergency-mode operation), **evaluation**, and **BAAs** with business associates.

### 3. Implement Physical Safeguards (§164.310)
**Facility access controls**, **workstation use** and **workstation security**, and **device and media controls** (disposal, media re-use, accountability, data backup and storage).

### 4. Implement Technical Safeguards (§164.312)
**Access control** (unique user ID, emergency access, automatic logoff, encryption/decryption), **audit controls**, **integrity** (mechanisms to authenticate ePHI), **person/entity authentication**, and **transmission security** (integrity controls + encryption).

### 5. Resolve "Required" vs "Addressable" specifications
Under the current rule, each implementation specification is **Required** (must implement) or **Addressable** (assess whether reasonable and appropriate; if so implement, if not document why and implement an equivalent alternative). **Addressable does not mean optional** — it means make and document a risk-based decision.

### 6. Execute Business Associate Agreements (§164.314 / §164.308(b))
Every business associate that touches ePHI needs a BAA binding it to safeguard ePHI, report incidents, and flow requirements to subcontractors. Maintain the BAA inventory.

### 7. Track the 2025 NPRM proposed changes (NOT yet final)
HHS OCR published an NPRM (Jan 6, 2025) proposing to **remove the required/addressable distinction** (make nearly all specifications required), and to mandate **MFA**, **encryption of ePHI at rest and in transit**, **asset inventory and network maps**, **vulnerability scans every 6 months**, **annual penetration testing**, **72-hour restoration of certain systems/data**, and **annual risk-analysis updates**. **These are proposals** — the current rule remains in force until a final rule is published and effective. Plan toward them, but comply with what is current.

### 8. Stand up breach-notification readiness (45 CFR §§164.400–414)
Define how you detect, assess (the four-factor risk assessment), and report breaches of unsecured PHI: to **individuals** and **HHS** (and **media** for breaches affecting 500+ in a state/jurisdiction), within the required timelines. Encryption to NIST standards renders PHI "secured" and is a safe harbor from breach notification.

### 9. Document everything (§164.316)
Maintain policies, procedures, and records of actions/decisions in writing, **retain for six years**, review periodically, and update in response to environmental or operational change.

## Key Concepts

| Concept | Definition |
|---|---|
| ePHI | Electronic protected health information — the Security Rule's scope. |
| Covered entity | Health plan, clearinghouse, or provider doing electronic transactions. |
| Business associate | A vendor that handles ePHI for a covered entity; bound by a BAA. |
| Risk analysis | Required, documented assessment of risks to ePHI (§164.308(a)(1)(ii)(A)). |
| Required vs addressable | Must-implement vs risk-based-decision implementation specifications. |
| Administrative / Physical / Technical safeguards | §164.308 / §164.310 / §164.312. |
| BAA | Business Associate Agreement — contractually binds vendors to safeguard ePHI. |
| Breach (unsecured PHI) | Triggers notification under §§164.400–414; encryption is a safe harbor. |
| OCR | HHS Office for Civil Rights — enforces HIPAA. |
| Six-year retention | Documentation retention requirement (§164.316). |

## Tools & Systems

- **45 CFR Part 164 Subpart C** — the Security Rule text (and Subpart D, Breach Notification).
- **HHS Security Risk Assessment (SRA) Tool** — free guided risk analysis.
- **NIST SP 800-66 Rev 2** — implementing the HIPAA Security Rule (NIST guidance, maps to 800-53).
- **NIST SP 800-30** — risk-assessment methodology to ground the SRA.
- **GRC / compliance platforms** — to manage policies, the BAA inventory, and evidence.
- **Encryption / MFA / SIEM / audit-logging tooling** — to satisfy technical safeguards and the proposed mandates.

## Common Scenarios

- **OCR investigation after a breach.** First request is almost always the current, dated **risk analysis** and the **risk-management plan** — have them ready.
- **New SaaS handling ePHI.** Sign a **BAA** before any ePHI flows; confirm the vendor's safeguards.
- **Addressable spec you won't implement as written.** Document the risk-based rationale and the **equivalent alternative** you implemented instead.
- **Preparing for the proposed rule.** Pre-position MFA, at-rest/in-transit encryption, asset inventory, scanning, and pen-testing so a final rule is a small step, not a scramble.
- **Lost/stolen device.** If ePHI was encrypted to NIST standards, the safe harbor applies; if not, run the four-factor breach assessment and notify as required.

## Output Format

Produce a **HIPAA Security Rule Gap Assessment** using `assets/template.md`, containing:

1. **Role & ePHI scope** — covered entity vs BA; ePHI data map and systems.
2. **Risk analysis summary** — top risks to ePHI with likelihood/impact (feeds risk management).
3. **Safeguard status** — Administrative / Physical / Technical, each specification marked **Implemented / Partial / Gap** with required-vs-addressable noted.
4. **BAA inventory** — business associates and BAA status.
5. **Breach-notification readiness** — detection, four-factor assessment, notification workflow.
6. **2025 NPRM gap view** — readiness against the proposed mandates (clearly labeled proposed).
7. **Remediation plan** — prioritized, with owners and dates; required specs and risk-analysis gaps first.

Use `scripts/process.py` to score a safeguard-status JSON across the §164.308/310/312 standards, weight required gaps above addressable ones, and emit the gap table plus a remediation-priority list.

===== END FILE =====


===== FILE: skills/implementing-hipaa-security-rule-safeguards/LICENSE =====

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


===== FILE: skills/implementing-hipaa-security-rule-safeguards/assets/template.md =====
# HIPAA Security Rule Gap Assessment — Worked Example

> Filled example for a small outpatient clinic (covered entity).
> Replace bracketed content for your own organization.

## 1. Role & ePHI Scope
- **Role:** Covered Entity (outpatient provider doing electronic transactions).
- **ePHI locations:** EHR SaaS, the clinic's clearing workstation, encrypted backups, and a billing vendor (business associate).
- **ePHI data map:** Patient intake → EHR (SaaS, BAA in place) → claims to billing BA → encrypted backup. Reference diagram `ephi-flow-v2`.

## 2. Risk Analysis Summary (feeds Risk Management)
| Top risk to ePHI | Likelihood | Impact | Note |
|---|---|---|---|
| Phishing → EHR credential theft | High | High | No MFA on remote EHR access |
| Lost/stolen laptop with cached ePHI | Moderate | High | Disk encryption not enforced fleet-wide |
| Billing BA mishandles ePHI | Low | High | BAA present; vendor safeguards unverified |

> The risk analysis is **dated and documented** — this is the first artifact OCR requests.

## 3. Safeguard Status
*(scored by `scripts/process.py`; weighted readiness ≈ 62% in this example)*

| Specification | Section | Requirement | Status | Alt. documented |
|---|---|---|---|---|
| 164.308(a)(1)(ii)(A) Risk Analysis | Administrative | required | **gap** | — |
| 164.308(a)(1)(ii)(B) Risk Management | Administrative | required | partial | — |
| 164.308(a)(2) Assigned Security Responsibility | Administrative | required | implemented | — |
| 164.310(d)(2)(i) Disposal | Physical | required | implemented | — |
| 164.312(a)(1) Unique User ID | Technical | required | implemented | — |
| 164.312(a)(2)(iv) Encryption/Decryption | Technical | addressable | partial | no |
| 164.312(a)(2)(iii) Automatic Logoff | Technical | addressable | gap | **yes** (equivalent timeout via MDM documented) |

**OCR-priority flag:** Risk Analysis is a gap → remediate first.

## 4. BAA Inventory
| Business associate | Service | BAA status |
|---|---|---|
| EHR SaaS vendor | Records system | Signed, current |
| Billing vendor | Claims processing | Signed; request SOC 2 / safeguards attestation |
| Backup provider | Encrypted offsite backup | Signed, current |

## 5. Breach-Notification Readiness (§§164.400–414)
- **Detection:** EHR + endpoint alerts route to the Security Official.
- **Assessment:** Documented **four-factor** procedure to decide whether a breach of unsecured PHI occurred.
- **Notification workflow:** Individuals within 60 days; HHS (annual log for <500, prompt for 500+); media for 500+ in-jurisdiction.
- **Safe harbor:** Enforce NIST-standard encryption so lost/stolen encrypted devices are "secured" and exempt from notification.

## 6. 2025 NPRM Gap View (PROPOSED — not yet final)
| Proposed mandate | Current state | Pre-position action |
|---|---|---|
| Mandatory MFA | Not enforced | Roll out MFA on all ePHI access |
| Encryption at rest & in transit | Partial | Enforce full-disk + TLS everywhere |
| Asset inventory + network map | Informal | Formalize and keep current |
| Vuln scans every 6 mo / annual pentest | Ad hoc | Schedule recurring scans + annual test |
| 72-hour restoration | Untested | Add RTO target + restore drills |
| Annual risk-analysis update | Irregular | Calendar annual refresh |

> These are **proposals**; comply with the current rule today and treat this column as a readiness runway.

## 7. Remediation Plan (prioritized)
1. **Risk Analysis (164.308(a)(1)(ii)(A)) — required gap, OCR-priority.** Complete and date a full SRA (use HHS SRA Tool + NIST 800-30). **Owner:** Security Official. **Due:** 30 days.
2. **Risk Management (164.308(a)(1)(ii)(B)) — required, partial.** Build the remediation plan that flows from the SRA. **Due:** 45 days.
3. **Encryption (164.312(a)(2)(iv)) — addressable, partial.** Enforce full-disk + in-transit encryption (also clears NPRM + breach safe harbor). **Due:** 60 days.
4. Verify the billing BA's safeguards (request SOC 2). **Due:** 60 days.
5. Re-run the scorer after each fix; target ≥ 95% weighted readiness with zero required gaps.

===== END FILE =====


===== FILE: skills/implementing-hipaa-security-rule-safeguards/references/standards.md =====
# HIPAA Security Rule — Standards & Reference

## Primary regulation
### HIPAA Security Rule — 45 CFR Part 164, Subpart C
- **Regulator**: U.S. Department of Health and Human Services, **Office for Civil Rights (OCR)**
- **Scope**: Security of **electronic** protected health information (ePHI) held or transmitted by covered entities and business associates.
- **Statutory basis**: HIPAA (1996) as amended by **HITECH** (2009).
- **eCFR**: https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C

## Rule structure (key sections)
| Section | Title |
|---|---|
| §164.302 | Applicability |
| §164.304 | Definitions |
| §164.306 | Security standards: general rules (flexibility, scalability) |
| §164.308 | **Administrative safeguards** |
| §164.310 | **Physical safeguards** |
| §164.312 | **Technical safeguards** |
| §164.314 | Organizational requirements (BAAs, group health plans) |
| §164.316 | Policies, procedures, and documentation (6-year retention) |

Breach Notification Rule: **45 CFR §§164.400–414** (Subpart D).

## Administrative safeguards (§164.308) — standards
- Security Management Process — **Risk Analysis (R)**, **Risk Management (R)**, Sanction Policy (R), Information System Activity Review (R)
- Assigned Security Responsibility (named Security Official)
- Workforce Security (authorization/supervision, clearance, termination — Addressable)
- Information Access Management (isolating clearinghouse functions (R); access authorization/establishment/modification — Addressable)
- Security Awareness and Training (reminders, malware protection, log-in monitoring, password management — Addressable)
- Security Incident Procedures — Response and Reporting (R)
- Contingency Plan — **Data Backup Plan (R)**, **Disaster Recovery Plan (R)**, **Emergency Mode Operation Plan (R)**, Testing/Revision (A), Applications/Data Criticality Analysis (A)
- Evaluation (periodic)
- Business Associate Contracts (§164.308(b))

## Physical safeguards (§164.310) — standards
- Facility Access Controls (contingency operations, facility security plan, access control/validation, maintenance records — all Addressable)
- Workstation Use (R)
- Workstation Security (R)
- Device and Media Controls — **Disposal (R)**, **Media Re-use (R)**, Accountability (A), Data Backup and Storage (A)

## Technical safeguards (§164.312) — standards
- Access Control — **Unique User Identification (R)**, **Emergency Access Procedure (R)**, Automatic Logoff (A), Encryption and Decryption (A)
- Audit Controls (R)
- Integrity — Mechanism to Authenticate ePHI (A)
- Person or Entity Authentication (R)
- Transmission Security — Integrity Controls (A), Encryption (A)

> (R) = Required, (A) = Addressable, under the **current** rule.

## Required vs Addressable
- **Required**: must be implemented as specified.
- **Addressable**: assess whether the specification is reasonable and appropriate; if yes, implement; if no, **document why** and implement an **equivalent alternative measure** if reasonable. Addressable is **not** optional.

## Breach Notification (45 CFR §§164.400–414)
- Applies to breaches of **unsecured PHI**.
- **Four-factor risk assessment** to determine whether a breach occurred (nature/extent of PHI, who used/received it, whether it was actually acquired/viewed, mitigation).
- Notify **individuals without unreasonable delay and within 60 days**; notify **HHS** (annually for <500; without unreasonable delay and within 60 days for 500+); notify **media** for breaches affecting **500+** residents of a state/jurisdiction.
- **Safe harbor**: PHI encrypted to HHS/NIST-specified standards is "secured" and not subject to breach notification.

## 2025 NPRM — PROPOSED changes (NOT yet final)
- **Citation**: 90 FR 800, RIN 0945-AA22, published **January 6, 2025**; comment period closed March 7, 2025.
- **Status**: **Proposed only.** The current Security Rule remains in force until a final rule is published and becomes effective. Track at https://www.federalregister.gov.
- **Notable proposals**:
  - Remove the **required/addressable** distinction — make (nearly) all implementation specifications **required**.
  - Mandatory **multi-factor authentication**.
  - Mandatory **encryption of ePHI at rest and in transit**.
  - **Asset inventory** and **network maps**, updated regularly.
  - **Vulnerability scans at least every 6 months** and **penetration testing at least annually**.
  - **72-hour restoration** of certain systems/data after an incident.
  - **Annual** risk-analysis updates and written documentation of compliance reviews.
- **Typical effective/compliance timing if finalized**: effective ~60 days after publication; compliance ~180 days after effective (subject to the final rule).

## Supporting NIST guidance
| Document | Role |
|---|---|
| NIST SP 800-66 Rev 2 (2024) | Implementing the HIPAA Security Rule; maps safeguards to SP 800-53 controls. |
| NIST SP 800-30 Rev 1 | Risk-assessment methodology underpinning the required risk analysis. |
| HHS SRA Tool | Free guided Security Risk Assessment for smaller organizations. |

## NIST CSF 2.0 alignment
| CSF 2.0 ID | Relevance |
|---|---|
| GV.OC-03 | Legal/regulatory (HIPAA) requirements understood. |
| GV.RM-01 | Risk-management objectives established. |
| ID.RA-01 / ID.RA-05 | Vulnerabilities identified; risk informs prioritization (the risk analysis). |
| PR.DS-01 | Data-at-rest protection (encryption of ePHI). |
| PR.AA-01 | Identity/authentication (unique IDs, MFA). |
| DE.CM-01 | Monitoring (audit controls, activity review). |

===== END FILE =====


===== FILE: skills/implementing-hipaa-security-rule-safeguards/scripts/process.py =====
#!/usr/bin/env python3
"""
HIPAA Security Rule safeguard gap-assessment scorer.

Scores a safeguard-status inventory across the Administrative (164.308),
Physical (164.310), and Technical (164.312) safeguards. Required gaps are
weighted above addressable gaps, and any gap in the Risk Analysis or Risk
Management specifications is escalated (these are the most-cited OCR findings).
Emits a gap table, a weighted readiness score, and a prioritized remediation list.

Input JSON shape:
{
  "org": {"name": "Northstar Clinic", "role": "Covered Entity"},
  "safeguards": [
    {
      "id": "164.308(a)(1)(ii)(A)", "section": "308",
      "name": "Risk Analysis", "requirement": "required",
      "status": "gap"
    },
    {
      "id": "164.312(a)(2)(iv)", "section": "312",
      "name": "Encryption and Decryption", "requirement": "addressable",
      "status": "partial",
      "alternative_documented": false
    }
  ]
}

requirement: required | addressable
status: implemented | partial | gap

Usage:
  python process.py --input safeguards.json [--output gap.md]
  python process.py --input safeguards.json --fail-on-required-gap
"""

import argparse
import json
import sys

SECTION_NAMES = {
    "308": "Administrative (§164.308)",
    "310": "Physical (§164.310)",
    "312": "Technical (§164.312)",
}
VALID_REQ = {"required", "addressable"}
VALID_STATUS = {"implemented", "partial", "gap"}

# status -> credit fraction toward "implemented"
CREDIT = {"implemented": 1.0, "partial": 0.5, "gap": 0.0}
# weighting: required safeguards count more toward the score
WEIGHT = {"required": 2, "addressable": 1}

# Specifications whose absence OCR cites most often -> escalate
HIGH_PRIORITY_KEYS = ("risk analysis", "risk management")


def score(data):
    sgs = data.get("safeguards", [])
    if not sgs:
        raise ValueError("safeguards list is required")

    total_weight = 0.0
    earned_weight = 0.0
    required_gaps = []
    addressable_gaps = []
    escalated = []
    by_section = {}
    rows = []

    for s in sgs:
        sid = s.get("id", "?")
        req = s.get("requirement")
        status = s.get("status")
        if req not in VALID_REQ:
            raise ValueError(f"{sid}: requirement '{req}' invalid (required|addressable)")
        if status not in VALID_STATUS:
            raise ValueError(f"{sid}: status '{status}' invalid (implemented|partial|gap)")

        section = str(s.get("section", "")).replace("164.", "").strip()
        sec_rec = by_section.setdefault(section, {"implemented": 0, "partial": 0, "gap": 0})
        sec_rec[status] += 1

        w = WEIGHT[req]
        total_weight += w
        earned_weight += w * CREDIT[status]

        name = s.get("name", "")
        is_high = any(k in name.lower() for k in HIGH_PRIORITY_KEYS)

        # an addressable item with a documented equivalent alternative is acceptable
        addressable_ok = (
            req == "addressable"
            and status != "implemented"
            and s.get("alternative_documented") is True
        )

        if status in ("gap", "partial") and not addressable_ok:
            if req == "required":
                required_gaps.append(s)
            else:
                addressable_gaps.append(s)
            if is_high:
                escalated.append(s)

        rows.append((sid, section, name, req, status, s.get("alternative_documented", None), is_high))

    readiness = (100.0 * earned_weight / total_weight) if total_weight else 0.0
    return {
        "readiness": readiness,
        "required_gaps": required_gaps,
        "addressable_gaps": addressable_gaps,
        "escalated": escalated,
        "by_section": by_section,
        "rows": rows,
    }


def render(data, res):
    org = data.get("org", {})
    lines = []
    lines.append(f"# HIPAA Security Rule Gap Assessment - {org.get('name','Organization')}")
    lines.append("")
    if org.get("role"):
        lines.append(f"- **Role:** {org['role']}")
    lines.append(f"- **Weighted readiness:** **{res['readiness']:.0f}%** "
                 "(required specifications weighted 2x addressable)")
    lines.append(f"- **Required gaps:** {len(res['required_gaps'])} | "
                 f"Addressable gaps (no documented alternative): {len(res['addressable_gaps'])}")
    lines.append("")

    if res["escalated"]:
        lines.append("> **OCR-priority gap detected:** Risk Analysis / Risk Management is "
                     "incomplete. This is the most-cited HIPAA finding - remediate first.")
        lines.append("")

    # status by section
    lines.append("## Status by safeguard section")
    lines.append("")
    lines.append("| Section | Implemented | Partial | Gap |")
    lines.append("|---|---|---|---|")
    for sec in sorted(res["by_section"]):
        r = res["by_section"][sec]
        label = SECTION_NAMES.get(sec, sec)
        lines.append(f"| {label} | {r['implemented']} | {r['partial']} | {r['gap']} |")
    lines.append("")

    # full table
    lines.append("## Safeguard detail")
    lines.append("")
    lines.append("| Specification | Section | Requirement | Status | Alt. documented |")
    lines.append("|---|---|---|---|---|")
    for sid, sec, name, req, status, alt, _ in res["rows"]:
        altdisp = "-" if alt is None else ("yes" if alt else "no")
        secdisp = SECTION_NAMES.get(sec, sec)
        lines.append(f"| {sid} {name} | {secdisp} | {req} | {status} | {altdisp} |")
    lines.append("")

    # remediation priority
    lines.append("## Remediation priority")
    lines.append("")
    order = []
    order += [(s, "ESCALATED (Risk Analysis/Mgmt)") for s in res["escalated"]]
    order += [(s, "Required gap") for s in res["required_gaps"] if s not in res["escalated"]]
    order += [(s, "Addressable - implement or document alternative") for s in res["addressable_gaps"]]
    if not order:
        lines.append("No outstanding gaps. Maintain documentation and re-evaluate on change.")
    else:
        seen = set()
        i = 1
        for s, why in order:
            key = s.get("id")
            if key in seen:
                continue
            seen.add(key)
            lines.append(f"{i}. **{s.get('id')}** {s.get('name','')} - {why} "
                         f"(currently {s.get('status')}).")
            i += 1

    return "\n".join(lines)


def main():
    ap = argparse.ArgumentParser(description="HIPAA Security Rule safeguard gap-assessment scorer")
    ap.add_argument("--input", "-i", required=True, help="Path to safeguard-status JSON")
    ap.add_argument("--output", "-o", help="Write Markdown gap assessment to this path")
    ap.add_argument("--fail-on-required-gap", action="store_true",
                    help="Exit non-zero if any required specification is partial/gap")
    args = ap.parse_args()

    try:
        with open(args.input) as f:
            data = json.load(f)
    except (OSError, json.JSONDecodeError) as e:
        print(f"ERROR: could not read input JSON: {e}", file=sys.stderr)
        return 2

    try:
        res = score(data)
        md = render(data, res)
    except ValueError as e:
        print(f"ERROR: {e}", file=sys.stderr)
        return 2

    if args.output:
        with open(args.output, "w") as f:
            f.write(md + "\n")
        print(f"Gap assessment written to {args.output}", file=sys.stderr)
    else:
        print(md)

    print(f"Readiness {res['readiness']:.0f}%; required gaps {len(res['required_gaps'])}; "
          f"escalated {len(res['escalated'])}.", file=sys.stderr)

    if args.fail_on_required_gap and res["required_gaps"]:
        ids = ", ".join(s.get("id", "?") for s in res["required_gaps"])
        print(f"FAIL: {len(res['required_gaps'])} required specification gap(s): {ids}", file=sys.stderr)
        return 1
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/implementing-iso-27001-information-security-management/SKILL.md =====
---
name: implementing-iso-27001-information-security-management
description: >-
  Guides implementation of an ISO/IEC 27001:2022 Information Security Management
  System (ISMS) end to end: gap analysis and scoping, risk assessment methodology,
  Annex A control selection, Statement of Applicability (SoA) creation, and
  continuous improvement. Use when scoping a new ISMS, preparing for ISO 27001
  certification or audit, or selecting and documenting Annex A controls for a
  compliance program.
domain: cybersecurity
subdomain: compliance-governance
tags:
- compliance
- governance
- iso27001
- isms
- risk-management
- certification
nist_csf:
- GV.OC-01
- GV.RM-01
- GV.PO-01
- ID.RA-01
- PR.DS-01
version: '1.0'
author: mahipal
license: Apache-2.0
mitre_attack:
- T1078
- T1530
- T1685.002
---
# Implementing ISO 27001 Information Security Management

## Overview
ISO/IEC 27001:2022 is the international standard for establishing, implementing, maintaining, and continually improving an Information Security Management System (ISMS). This skill covers the complete lifecycle from scoping through certification, including Annex A control selection, risk assessment methodology, Statement of Applicability (SoA) creation, and continuous improvement processes.


## When to Use

- When deploying or configuring implementing iso 27001 information security management capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites
- Understanding of information security principles and risk management concepts
- Familiarity with organizational governance structures and business processes
- Knowledge of IT infrastructure, network architecture, and data flows
- Access to ISO/IEC 27001:2022 and ISO/IEC 27002:2022 standards documents

## Core Concepts

### ISMS Clauses (4-10)
The management system requirements define **what** must be done:
- **Clause 4 - Context of the Organization**: Define scope, interested parties, and internal/external issues
- **Clause 5 - Leadership**: Top management commitment, information security policy, roles and responsibilities
- **Clause 6 - Planning**: Risk assessment process, risk treatment plan, information security objectives
- **Clause 7 - Support**: Resources, competence, awareness, communication, documented information
- **Clause 8 - Operation**: Operational planning, risk assessment execution, risk treatment implementation
- **Clause 9 - Performance Evaluation**: Monitoring, measurement, internal audit, management review
- **Clause 10 - Improvement**: Nonconformities, corrective actions, continual improvement

### Annex A Controls (2022 Edition)
The 2022 revision restructured 93 controls into four categories:

| Category | Controls | Examples |
|----------|----------|----------|
| Organizational (A.5) | 37 controls | Policies, roles, threat intelligence, cloud security |
| People (A.6) | 8 controls | Screening, awareness, remote working, reporting |
| Physical (A.7) | 14 controls | Perimeters, entry controls, equipment security |
| Technological (A.8) | 34 controls | Access control, cryptography, logging, secure development |

### New Controls in 2022 Edition
11 new controls were added:
1. A.5.7 - Threat Intelligence
2. A.5.23 - Information Security for Cloud Services
3. A.5.30 - ICT Readiness for Business Continuity
4. A.7.4 - Physical Security Monitoring
5. A.8.9 - Configuration Management
6. A.8.10 - Information Deletion
7. A.8.11 - Data Masking
8. A.8.12 - Data Leakage Prevention
9. A.8.16 - Monitoring Activities
10. A.8.23 - Web Filtering
11. A.8.28 - Secure Coding

## Workflow

### Phase 1: Gap Analysis and Scoping (Weeks 1-4)
1. Define ISMS scope boundaries (locations, business units, systems)
2. Identify interested parties and their requirements
3. Perform gap analysis against ISO 27001:2022 requirements
4. Document internal and external context (PESTLE, SWOT)
5. Obtain top management commitment and allocate budget

### Phase 2: Risk Assessment (Weeks 5-10)
1. Define risk assessment methodology (asset-based, scenario-based, or hybrid)
2. Create asset inventory covering information, people, processes, technology
3. Identify threats and vulnerabilities for each asset
4. Assess risk likelihood and impact using defined criteria
5. Calculate risk levels and determine risk treatment options (mitigate, accept, transfer, avoid)
6. Develop Risk Treatment Plan (RTP)

### Phase 3: Control Selection and SoA (Weeks 11-14)
1. Map risk treatments to Annex A controls
2. Create Statement of Applicability (SoA) documenting:
   - Which controls are applicable and justification
   - Which controls are excluded and justification
   - Implementation status of each control
3. Design control implementation plans with owners and timelines

### Phase 4: Implementation (Weeks 15-30)
1. Develop and approve information security policy
2. Implement selected Annex A controls
3. Create mandatory documented procedures:
   - Information Security Policy (A.5.1)
   - Risk Assessment Process (Clause 6.1.2)
   - Risk Treatment Process (Clause 6.1.3)
   - Internal Audit Programme (Clause 9.2)
   - Management Review Process (Clause 9.3)
   - Corrective Action Procedure (Clause 10.1)
4. Deploy technical controls and security tooling
5. Conduct security awareness training for all personnel

### Phase 5: Internal Audit and Management Review (Weeks 31-36)
1. Plan and execute internal audit programme covering all clauses and applicable controls
2. Document audit findings and nonconformities
3. Implement corrective actions with root cause analysis
4. Conduct management review covering:
   - Status of previous actions
   - Changes in internal/external issues
   - Information security performance metrics
   - Audit results and risk assessment outcomes
   - Opportunities for improvement

### Phase 6: Certification Audit (Weeks 37-42)
1. **Stage 1 Audit**: Documentation review, readiness assessment
2. Address Stage 1 findings
3. **Stage 2 Audit**: On-site assessment of ISMS effectiveness
4. Resolve any nonconformities (major NCRs require re-audit)
5. Receive ISO 27001 certification (valid for 3 years)

### Phase 7: Continual Improvement (Ongoing)
1. Annual surveillance audits (Years 1 and 2)
2. Recertification audit (Year 3)
3. Regular risk reassessment and control effectiveness reviews
4. Incident-driven improvements and lessons learned integration

## Key Artifacts
- ISMS Scope Document
- Information Security Policy
- Risk Assessment Methodology
- Risk Register and Risk Treatment Plan
- Statement of Applicability (SoA)
- Internal Audit Reports
- Management Review Minutes
- Corrective Action Register
- Metrics and KPI Dashboard

## Common Pitfalls
- Scope too broad or too narrow, leading to audit complications
- Treating ISO 27001 as a checkbox exercise rather than embedding into business processes
- Insufficient top management involvement and commitment
- Failing to maintain documented evidence of control operation
- Not performing regular risk reassessments as the threat landscape changes
- Ignoring the 11 new controls in the 2022 edition during transition

## Integration Points
- **ISO 27002:2022**: Detailed implementation guidance for Annex A controls
- **ISO 27005**: Information security risk management methodology
- **ISO 27017**: Cloud security controls
- **ISO 27018**: Protection of PII in cloud services
- **ISO 27701**: Privacy Information Management System (PIMS) extension
- **NIST CSF 2.0**: Cross-mapping for dual compliance
- **SOC 2**: Overlapping trust service criteria

## References
- ISO/IEC 27001:2022 Information Security Management Systems
- ISO/IEC 27002:2022 Information Security Controls
- ISO/IEC 27005:2022 Information Security Risk Management
- ISMS.online ISO 27001 Annex A Guide: https://www.isms.online/iso-27001/annex-a-2022/
- IT Governance ISO 27001 Controls Guide: https://www.itgovernance.co.uk/blog/iso-27001-the-14-control-sets-of-annex-a-explained

===== END FILE =====


===== FILE: skills/implementing-iso-27001-information-security-management/LICENSE =====

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


===== FILE: skills/implementing-iso-27001-information-security-management/assets/template.md =====
# ISO 27001:2022 Implementation Audit Checklist

## Organization Information
| Field | Value |
|-------|-------|
| Organization Name | |
| ISMS Scope | |
| Assessment Date | |
| Assessor Name | |
| Assessment Type | Gap Analysis / Internal Audit / Stage 1 / Stage 2 |

---

## Clause 4: Context of the Organization

### 4.1 Understanding the Organization and its Context
- [ ] Internal issues identified and documented
- [ ] External issues identified and documented
- [ ] PESTLE analysis completed
- [ ] Issues regularly reviewed and updated
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed
- **Notes**: _______________

### 4.2 Understanding the Needs and Expectations of Interested Parties
- [ ] Interested parties identified
- [ ] Requirements of interested parties documented
- [ ] Legal, regulatory, and contractual requirements identified
- [ ] Requirements reviewed periodically
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 4.3 Determining the Scope of the ISMS
- [ ] ISMS scope defined and documented
- [ ] Scope considers internal and external issues (4.1)
- [ ] Scope considers interested party requirements (4.2)
- [ ] Scope considers interfaces and dependencies
- [ ] Scope available as documented information
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 4.4 Information Security Management System
- [ ] ISMS processes established
- [ ] ISMS implemented and maintained
- [ ] Continual improvement processes in place
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

---

## Clause 5: Leadership

### 5.1 Leadership and Commitment
- [ ] Top management demonstrates commitment to the ISMS
- [ ] Information security policy and objectives aligned with strategic direction
- [ ] ISMS requirements integrated into business processes
- [ ] Adequate resources provided
- [ ] Importance of information security communicated
- [ ] ISMS achieves its intended outcomes
- [ ] Persons directed and supported to contribute to ISMS effectiveness
- [ ] Continual improvement promoted
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 5.2 Information Security Policy
- [ ] Policy appropriate to the purpose of the organization
- [ ] Policy includes information security objectives or framework for setting objectives
- [ ] Policy includes commitment to satisfy applicable requirements
- [ ] Policy includes commitment to continual improvement
- [ ] Policy available as documented information
- [ ] Policy communicated within the organization
- [ ] Policy available to interested parties as appropriate
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 5.3 Organizational Roles, Responsibilities and Authorities
- [ ] Information security roles and responsibilities assigned and communicated
- [ ] Responsibility for ISMS conformance assigned
- [ ] Responsibility for reporting ISMS performance assigned
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

---

## Clause 6: Planning

### 6.1.1 General (Actions to Address Risks and Opportunities)
- [ ] Risks and opportunities considered (referencing 4.1 and 4.2)
- [ ] Actions planned to address risks and opportunities
- [ ] Plans for integrating actions into ISMS processes
- [ ] Plans for evaluating effectiveness of actions
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 6.1.2 Information Security Risk Assessment
- [ ] Risk assessment process defined and documented
- [ ] Risk acceptance criteria established
- [ ] Criteria for performing risk assessments defined
- [ ] Risk assessment process produces consistent, valid, and comparable results
- [ ] Information security risks identified
- [ ] Risk owners identified
- [ ] Risk likelihood and impact assessed
- [ ] Risk levels determined
- [ ] Risk assessment results documented
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 6.1.3 Information Security Risk Treatment
- [ ] Risk treatment options selected (mitigate, accept, avoid, transfer)
- [ ] Controls determined for risk treatment
- [ ] Controls compared with Annex A
- [ ] Statement of Applicability produced
- [ ] Risk treatment plan formulated
- [ ] Risk owners approve risk treatment plan and residual risks
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 6.2 Information Security Objectives and Planning to Achieve Them
- [ ] Objectives established at relevant functions and levels
- [ ] Objectives consistent with information security policy
- [ ] Objectives measurable (where practicable)
- [ ] Objectives consider applicable requirements and risk assessment results
- [ ] Objectives communicated
- [ ] Objectives updated as appropriate
- [ ] Plans: what will be done, resources, responsibilities, timelines, evaluation
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 6.3 Planning of Changes
- [ ] Changes to ISMS carried out in a planned manner
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

---

## Clause 7: Support

### 7.1 Resources
- [ ] Resources needed for ISMS determined and provided
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 7.2 Competence
- [ ] Competence requirements determined for ISMS roles
- [ ] Competence ensured through education, training, or experience
- [ ] Actions taken to acquire competence where needed
- [ ] Evidence of competence retained
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 7.3 Awareness
- [ ] Persons aware of information security policy
- [ ] Persons aware of their contribution to ISMS effectiveness
- [ ] Persons aware of implications of not conforming to ISMS requirements
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 7.4 Communication
- [ ] Internal and external communication needs determined
- [ ] What to communicate defined
- [ ] When to communicate defined
- [ ] With whom to communicate defined
- [ ] How to communicate defined
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 7.5 Documented Information
- [ ] Documented information required by ISO 27001 maintained
- [ ] Additional documented information determined by organization maintained
- [ ] Appropriate identification and description
- [ ] Appropriate format (language, software version, graphics)
- [ ] Appropriate review and approval
- [ ] Documented information available and suitable for use
- [ ] Adequate protection (loss of confidentiality, improper use, integrity loss)
- [ ] Distribution, access, retrieval, and use controlled
- [ ] Storage and preservation controlled
- [ ] Control of changes maintained
- [ ] Retention and disposition determined
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

---

## Clause 8: Operation

### 8.1 Operational Planning and Control
- [ ] Processes needed to meet ISMS requirements planned, implemented, and controlled
- [ ] Criteria for processes established
- [ ] Control of processes in accordance with criteria implemented
- [ ] Documented information retained to demonstrate processes carried out as planned
- [ ] Planned changes controlled
- [ ] Unintended changes reviewed and actions taken to mitigate adverse effects
- [ ] Outsourced processes controlled
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 8.2 Information Security Risk Assessment
- [ ] Risk assessments performed at planned intervals
- [ ] Risk assessments performed when significant changes occur
- [ ] Results of risk assessments documented
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 8.3 Information Security Risk Treatment
- [ ] Risk treatment plan implemented
- [ ] Results of risk treatment documented
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

---

## Clause 9: Performance Evaluation

### 9.1 Monitoring, Measurement, Analysis and Evaluation
- [ ] What needs to be monitored and measured determined
- [ ] Monitoring and measurement methods determined
- [ ] When monitoring and measuring shall be performed determined
- [ ] Who shall monitor and measure determined
- [ ] When results shall be analysed and evaluated determined
- [ ] Who shall analyse and evaluate results determined
- [ ] Results of monitoring and measurement documented
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 9.2 Internal Audit
- [ ] Internal audit programme planned (frequency, methods, responsibilities)
- [ ] Audit programme considers importance of processes and previous audit results
- [ ] Audit criteria and scope defined for each audit
- [ ] Auditors selected to ensure objectivity and impartiality
- [ ] Audit results reported to relevant management
- [ ] Documented information retained as evidence of audit programme and results
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 9.3 Management Review
- [ ] Management review conducted at planned intervals
- [ ] Review considers status of actions from previous reviews
- [ ] Review considers changes in external and internal issues
- [ ] Review considers feedback on information security performance
- [ ] Review considers feedback from interested parties
- [ ] Review considers risk assessment results and treatment plan status
- [ ] Review considers opportunities for continual improvement
- [ ] Decisions on improvement opportunities documented
- [ ] Decisions on changes needed to ISMS documented
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

---

## Clause 10: Improvement

### 10.1 Continual Improvement
- [ ] Suitability, adequacy, and effectiveness of ISMS continually improved
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

### 10.2 Nonconformity and Corrective Action
- [ ] Nonconformities identified and reacted to
- [ ] Actions taken to control and correct nonconformities
- [ ] Consequences dealt with
- [ ] Need for action to eliminate root cause evaluated
- [ ] Corrective actions implemented
- [ ] Effectiveness of corrective actions reviewed
- [ ] Changes to ISMS made if necessary
- [ ] Documented information retained on nature of nonconformities, actions taken, and results
- **Evidence**: _______________
- **Status**: Conforming / Minor NC / Major NC / Not Assessed

---

## Summary

| Category | Total Items | Conforming | Minor NC | Major NC | Not Assessed |
|----------|-------------|------------|----------|----------|--------------|
| Clause 4 | | | | | |
| Clause 5 | | | | | |
| Clause 6 | | | | | |
| Clause 7 | | | | | |
| Clause 8 | | | | | |
| Clause 9 | | | | | |
| Clause 10 | | | | | |
| **Total** | | | | | |

## Annex A Control Assessment (attach separately)
- Total Applicable Controls: _____ / 93
- Fully Implemented: _____
- Partially Implemented: _____
- Not Implemented: _____
- Not Applicable: _____

## Recommendations
1. _______________
2. _______________
3. _______________

## Sign-off
| Role | Name | Signature | Date |
|------|------|-----------|------|
| Assessor | | | |
| ISMS Manager | | | |
| Top Management | | | |

===== END FILE =====


===== FILE: skills/implementing-iso-27001-information-security-management/references/api-reference.md =====
# API Reference: Implementing ISO 27001 Information Security Management

## ISO 27001:2022 Clause Structure

| Clause | Title | Key Deliverable |
|--------|-------|----------------|
| 4 | Context of the Organization | ISMS Scope Document |
| 5 | Leadership | Information Security Policy |
| 6 | Planning | SoA, Risk Treatment Plan |
| 7 | Support | Competence records, Awareness |
| 8 | Operation | Risk assessment/treatment results |
| 9 | Performance Evaluation | Audit reports, Management review |
| 10 | Improvement | Corrective action records |

## Annex A Control Categories (2022)

| Category | Name | Controls |
|----------|------|----------|
| A.5 | Organizational | 37 controls |
| A.6 | People | 8 controls |
| A.7 | Physical | 14 controls |
| A.8 | Technological | 34 controls |

## Required Documented Information

| Document | Clause |
|----------|--------|
| ISMS Scope | 4.3 |
| Information Security Policy | 5.2 |
| Risk Assessment Methodology | 6.1.2 |
| Statement of Applicability | 6.1.3d |
| Risk Treatment Plan | 6.1.3 |
| Security Objectives | 6.2 |
| Internal Audit Program | 9.2 |
| Management Review Minutes | 9.3 |

## Risk Assessment Formula

```
Risk Level = Likelihood x Impact
- Likelihood: 1 (Rare) to 5 (Almost Certain)
- Impact: 1 (Negligible) to 5 (Catastrophic)
- Risk Rating: Low (1-6), Medium (7-12), High (13-19), Critical (20-25)
```

### References

- ISO 27001:2022: https://www.iso.org/standard/27001
- ISO 27002:2022: https://www.iso.org/standard/75652.html
- ISO 27005 Risk Management: https://www.iso.org/standard/80585.html

===== END FILE =====


===== FILE: skills/implementing-iso-27001-information-security-management/references/standards.md =====
# ISO 27001 Standards Reference

## Primary Standards

### ISO/IEC 27001:2022
- **Title**: Information Security, Cybersecurity and Privacy Protection - Information Security Management Systems - Requirements
- **Published**: October 2022
- **Scope**: Specifies requirements for establishing, implementing, maintaining, and continually improving an ISMS
- **Certification**: Organizations can be certified against this standard by accredited certification bodies
- **Key Changes from 2013**:
  - Annex A restructured from 14 domains to 4 themes (Organizational, People, Physical, Technological)
  - 93 controls (down from 114)
  - 11 new controls added
  - Title expanded to include "Cybersecurity and Privacy Protection"
  - Clause 6.3 added for planning changes to the ISMS

### ISO/IEC 27002:2022
- **Title**: Information Security, Cybersecurity and Privacy Protection - Information Security Controls
- **Role**: Implementation guidance for Annex A controls
- **Key Addition**: Each control now includes attribute taxonomy:
  - Control type: Preventive, Detective, Corrective
  - Information security properties: Confidentiality, Integrity, Availability
  - Cybersecurity concepts: Identify, Protect, Detect, Respond, Recover
  - Operational capabilities: 15 categories (Governance, Asset Management, etc.)
  - Security domains: Governance and Ecosystem, Protection, Defence, Resilience

## Supporting Standards in the ISO 27000 Family

### ISO/IEC 27000:2018
- Overview and vocabulary for ISMS

### ISO/IEC 27003:2017
- Guidance on ISMS implementation

### ISO/IEC 27004:2016
- Information security management monitoring, measurement, analysis, and evaluation

### ISO/IEC 27005:2022
- Guidance on managing information security risks
- Aligns with ISO 31000 risk management framework
- Provides asset-based and event-based risk assessment approaches

### ISO/IEC 27006-1:2024
- Requirements for bodies providing audit and certification of ISMS

### ISO/IEC 27007:2020
- Guidelines for ISMS auditing

### ISO/IEC 27017:2015
- Code of practice for cloud services information security controls

### ISO/IEC 27018:2019
- Code of practice for protection of PII in public clouds

### ISO/IEC 27701:2019
- Extension to ISO 27001 for Privacy Information Management System (PIMS)
- Supports GDPR compliance

## Annex A Control Categories Detail

### A.5 Organizational Controls (37 controls)
| Control | Title |
|---------|-------|
| A.5.1 | Policies for information security |
| A.5.2 | Information security roles and responsibilities |
| A.5.3 | Segregation of duties |
| A.5.4 | Management responsibilities |
| A.5.5 | Contact with authorities |
| A.5.6 | Contact with special interest groups |
| A.5.7 | Threat intelligence (NEW) |
| A.5.8 | Information security in project management |
| A.5.9 | Inventory of information and other associated assets |
| A.5.10 | Acceptable use of information and other associated assets |
| A.5.11 | Return of assets |
| A.5.12 | Classification of information |
| A.5.13 | Labelling of information |
| A.5.14 | Information transfer |
| A.5.15 | Access control |
| A.5.16 | Identity management |
| A.5.17 | Authentication information |
| A.5.18 | Access rights |
| A.5.19 | Information security in supplier relationships |
| A.5.20 | Addressing information security within supplier agreements |
| A.5.21 | Managing information security in the ICT supply chain |
| A.5.22 | Monitoring, review and change management of supplier services |
| A.5.23 | Information security for use of cloud services (NEW) |
| A.5.24 | Information security incident management planning and preparation |
| A.5.25 | Assessment and decision on information security events |
| A.5.26 | Response to information security incidents |
| A.5.27 | Learning from information security incidents |
| A.5.28 | Collection of evidence |
| A.5.29 | Information security during disruption |
| A.5.30 | ICT readiness for business continuity (NEW) |
| A.5.31 | Legal, statutory, regulatory and contractual requirements |
| A.5.32 | Intellectual property rights |
| A.5.33 | Protection of records |
| A.5.34 | Privacy and protection of PII |
| A.5.35 | Independent review of information security |
| A.5.36 | Compliance with policies, rules and standards for information security |
| A.5.37 | Documented operating procedures |

### A.6 People Controls (8 controls)
| Control | Title |
|---------|-------|
| A.6.1 | Screening |
| A.6.2 | Terms and conditions of employment |
| A.6.3 | Information security awareness, education and training |
| A.6.4 | Disciplinary process |
| A.6.5 | Responsibilities after termination or change of employment |
| A.6.6 | Confidentiality or non-disclosure agreements |
| A.6.7 | Remote working |
| A.6.8 | Information security event reporting |

### A.7 Physical Controls (14 controls)
| Control | Title |
|---------|-------|
| A.7.1 | Physical security perimeters |
| A.7.2 | Physical entry |
| A.7.3 | Securing offices, rooms and facilities |
| A.7.4 | Physical security monitoring (NEW) |
| A.7.5 | Protecting against physical and environmental threats |
| A.7.6 | Working in secure areas |
| A.7.7 | Clear desk and clear screen |
| A.7.8 | Equipment siting and protection |
| A.7.9 | Security of assets off-premises |
| A.7.10 | Storage media |
| A.7.11 | Supporting utilities |
| A.7.12 | Cabling security |
| A.7.13 | Equipment maintenance |
| A.7.14 | Secure disposal or re-use of equipment |

### A.8 Technological Controls (34 controls)
| Control | Title |
|---------|-------|
| A.8.1 | User endpoint devices |
| A.8.2 | Privileged access rights |
| A.8.3 | Information access restriction |
| A.8.4 | Access to source code |
| A.8.5 | Secure authentication |
| A.8.6 | Capacity management |
| A.8.7 | Protection against malware |
| A.8.8 | Management of technical vulnerabilities |
| A.8.9 | Configuration management (NEW) |
| A.8.10 | Information deletion (NEW) |
| A.8.11 | Data masking (NEW) |
| A.8.12 | Data leakage prevention (NEW) |
| A.8.13 | Information backup |
| A.8.14 | Redundancy of information processing facilities |
| A.8.15 | Logging |
| A.8.16 | Monitoring activities (NEW) |
| A.8.17 | Clock synchronization |
| A.8.18 | Use of privileged utility programs |
| A.8.19 | Installation of software on operational systems |
| A.8.20 | Networks security |
| A.8.21 | Security of network services |
| A.8.22 | Segregation of networks |
| A.8.23 | Web filtering (NEW) |
| A.8.24 | Use of cryptography |
| A.8.25 | Secure development life cycle |
| A.8.26 | Application security requirements |
| A.8.27 | Secure system architecture and engineering principles |
| A.8.28 | Secure coding (NEW) |
| A.8.29 | Security testing in development and acceptance |
| A.8.30 | Outsourced development |
| A.8.31 | Separation of development, test and production environments |
| A.8.32 | Change management |
| A.8.33 | Test information |
| A.8.34 | Protection of information systems during audit testing |

## Certification Process
1. **Stage 1 Audit** (Document Review): Auditor reviews ISMS documentation, scope, SoA, risk assessment methodology
2. **Stage 2 Audit** (Certification Audit): On-site assessment of ISMS implementation and effectiveness
3. **Surveillance Audits**: Annual audits in Years 1 and 2 to verify continued compliance
4. **Recertification Audit**: Full re-assessment in Year 3 before certificate expiry

## Accreditation Bodies
- UKAS (United Kingdom)
- ANAB (United States)
- DAkkS (Germany)
- JAS-ANZ (Australia/New Zealand)
- COFRAC (France)

===== END FILE =====


===== FILE: skills/implementing-iso-27001-information-security-management/references/workflows.md =====
# ISO 27001 Implementation Workflows

## Workflow 1: ISMS Scoping and Context Analysis

```
Start
  |
  v
[Identify Internal Context]
  - Organization structure
  - Existing policies and processes
  - IT infrastructure and systems
  - Culture and capabilities
  |
  v
[Identify External Context]
  - Legal and regulatory requirements
  - Industry standards and obligations
  - Customer and partner requirements
  - Threat landscape and geopolitical factors
  |
  v
[Identify Interested Parties]
  - Customers and clients
  - Regulators and authorities
  - Employees and contractors
  - Shareholders and board members
  - Suppliers and partners
  |
  v
[Define ISMS Scope]
  - Business units in scope
  - Physical locations
  - Information systems and networks
  - Third-party services
  - Exclusions with justification
  |
  v
[Document ISMS Scope Statement]
  |
  v
[Obtain Top Management Approval]
  |
  v
End
```

## Workflow 2: Risk Assessment Process

```
Start
  |
  v
[Define Risk Criteria]
  - Risk acceptance criteria
  - Likelihood scale (1-5)
  - Impact scale (1-5)
  - Risk matrix thresholds
  |
  v
[Create Asset Inventory]
  - Information assets
  - Software assets
  - Hardware assets
  - People (roles)
  - Services (cloud, third-party)
  - Physical locations
  |
  v
[Identify Threats]
  - Natural threats (fire, flood)
  - Human threats (insider, external attacker)
  - Technical threats (malware, system failure)
  - Supply chain threats
  |
  v
[Identify Vulnerabilities]
  - Technical vulnerabilities
  - Process weaknesses
  - People-related gaps
  - Physical security gaps
  |
  v
[Assess Existing Controls]
  - Document current controls
  - Evaluate control effectiveness
  |
  v
[Calculate Risk Level]
  Risk = Likelihood x Impact
  - Consider existing controls
  - Use defined risk criteria
  |
  v
[Compare Against Risk Acceptance]
  |
  +--> [Risk Acceptable] --> Document and Monitor
  |
  +--> [Risk Not Acceptable]
        |
        v
      [Select Risk Treatment]
        - Mitigate (apply controls)
        - Transfer (insurance, outsource)
        - Avoid (stop activity)
        - Accept (with justification)
        |
        v
      [Map to Annex A Controls]
        |
        v
      [Document in Risk Treatment Plan]
        |
        v
      [Update Statement of Applicability]
        |
        v
      End
```

## Workflow 3: Statement of Applicability (SoA) Creation

```
Start
  |
  v
[List All 93 Annex A Controls]
  |
  v
[For Each Control]
  |
  v
[Is Control Required by Risk Treatment?]
  |
  +--> Yes --> Mark as Applicable
  |             - Link to risk(s)
  |             - Document implementation status
  |             - Assign control owner
  |
  +--> No --> [Is Control Required by Law/Contract?]
               |
               +--> Yes --> Mark as Applicable
               |             - Document legal/contractual basis
               |
               +--> No --> [Is Control Best Practice?]
                            |
                            +--> Yes --> Mark as Applicable
                            |             - Document business justification
                            |
                            +--> No --> Mark as Not Applicable
                                        - Document exclusion justification
  |
  v
[Review SoA Completeness]
  - All 93 controls addressed
  - Every exclusion justified
  - Implementation status documented
  |
  v
[Approve SoA]
  |
  v
End
```

## Workflow 4: Internal Audit Programme

```
Start
  |
  v
[Plan Audit Programme]
  - Define audit scope (clauses and controls)
  - Schedule audits across the year
  - Assign qualified auditors (independent of audited area)
  - Prepare audit criteria and checklists
  |
  v
[Conduct Audit]
  - Opening meeting with auditees
  - Review documented information
  - Interview key personnel
  - Observe processes in action
  - Collect objective evidence
  - Closing meeting with preliminary findings
  |
  v
[Document Findings]
  - Major Nonconformities (systemic failure, absence of control)
  - Minor Nonconformities (isolated failure, partial implementation)
  - Observations (potential for improvement)
  - Opportunities for Improvement (OFIs)
  |
  v
[Issue Audit Report]
  |
  v
[Corrective Action Process]
  - Containment: immediate action to limit impact
  - Root Cause Analysis: identify underlying cause
  - Corrective Action: implement fix to prevent recurrence
  - Verification: confirm effectiveness of correction
  |
  v
[Track to Closure]
  |
  v
[Feed into Management Review]
  |
  v
End
```

## Workflow 5: Management Review

```
Start
  |
  v
[Prepare Review Inputs]
  - Status of actions from previous reviews
  - Changes in external/internal issues
  - Changes in interested party needs
  - Information security performance:
    * Nonconformities and corrective actions
    * Monitoring and measurement results
    * Audit results
    * Fulfilment of objectives
  - Feedback from interested parties
  - Results of risk assessment and treatment plan
  - Opportunities for continual improvement
  |
  v
[Conduct Management Review Meeting]
  - Present ISMS performance data
  - Discuss risk landscape changes
  - Review incident trends
  - Evaluate resource adequacy
  |
  v
[Document Review Outputs]
  - Decisions on continual improvement opportunities
  - Changes needed to the ISMS
  - Resource allocation decisions
  - Updated risk acceptance criteria (if needed)
  |
  v
[Assign Actions with Owners and Deadlines]
  |
  v
[Track Implementation]
  |
  v
End
```

## Workflow 6: Certification Audit Preparation

```
Start
  |
  v
[Pre-Audit Readiness Check]
  - All mandatory documents in place
  - SoA current and approved
  - Risk assessment completed
  - Internal audit completed
  - Management review conducted
  - Corrective actions closed
  |
  v
[Select Certification Body]
  - Verify UKAS/ANAB accreditation
  - Compare audit team experience
  - Review commercial terms
  |
  v
[Stage 1 Audit (Documentation Review)]
  - Auditor reviews ISMS documentation
  - Assesses readiness for Stage 2
  - Identifies any significant gaps
  |
  v
[Address Stage 1 Findings]
  - Resolve documentation gaps
  - Complete any missing processes
  |
  v
[Stage 2 Audit (Certification Audit)]
  - On-site assessment (typically 3-5 days)
  - Evidence-based verification
  - Interviews across the organization
  - Technical control testing
  |
  v
[Audit Outcome]
  |
  +--> [No Major NCRs] --> Certificate Issued
  |
  +--> [Major NCRs Found]
        |
        v
      [Resolve NCRs within 90 days]
        |
        v
      [Follow-up Audit]
        |
        v
      [Certificate Issued]
  |
  v
End
```

===== END FILE =====


===== FILE: skills/implementing-iso-27001-information-security-management/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for assessing ISO 27001:2022 ISMS compliance."""

import json
import argparse
from datetime import datetime
from collections import Counter

ANNEX_A_CATEGORIES = {
    "A.5": {"name": "Organizational controls", "count": 37},
    "A.6": {"name": "People controls", "count": 8},
    "A.7": {"name": "Physical controls", "count": 14},
    "A.8": {"name": "Technological controls", "count": 34},
}

REQUIRED_DOCUMENTS = [
    "ISMS Scope (4.3)", "Information Security Policy (5.2)",
    "Risk Assessment Methodology (6.1.2)", "Risk Treatment Plan (6.1.3)",
    "Statement of Applicability (6.1.3d)", "Information Security Objectives (6.2)",
    "Evidence of Competence (7.2)", "Documented Operating Procedures (8.1)",
    "Risk Assessment Results (8.2)", "Risk Treatment Results (8.3)",
    "Monitoring and Measurement Results (9.1)", "Internal Audit Program (9.2)",
    "Management Review Results (9.3)", "Nonconformities and Corrective Actions (10.1)",
]


def assess_soa_completeness(soa_path):
    """Assess Statement of Applicability completeness."""
    with open(soa_path) as f:
        soa = json.load(f)
    controls = soa if isinstance(soa, list) else soa.get("controls", [])
    total = len(controls)
    implemented = sum(1 for c in controls if c.get("status", "").lower() == "implemented")
    partially = sum(1 for c in controls if c.get("status", "").lower() == "partial")
    not_implemented = sum(1 for c in controls if c.get("status", "").lower() == "not_implemented")
    excluded = sum(1 for c in controls if c.get("status", "").lower() == "excluded")

    missing_justification = [c for c in controls
                             if c.get("status", "").lower() == "excluded"
                             and not c.get("justification")]
    findings = []
    if missing_justification:
        findings.append({
            "issue": f"{len(missing_justification)} excluded controls without justification",
            "severity": "HIGH",
            "controls": [c.get("id", "") for c in missing_justification],
        })

    no_evidence = [c for c in controls
                   if c.get("status", "").lower() == "implemented"
                   and not c.get("evidence")]
    if no_evidence:
        findings.append({
            "issue": f"{len(no_evidence)} implemented controls without evidence",
            "severity": "MEDIUM",
        })

    return {
        "total_controls": total,
        "implemented": implemented,
        "partial": partially,
        "not_implemented": not_implemented,
        "excluded": excluded,
        "implementation_rate": round(implemented / total * 100, 1) if total else 0,
        "findings": findings,
    }


def assess_documentation(docs_inventory_path):
    """Check required ISMS documentation against inventory."""
    with open(docs_inventory_path) as f:
        inventory = json.load(f)
    docs = inventory if isinstance(inventory, list) else inventory.get("documents", [])
    doc_names = [d.get("name", d.get("title", "")).lower() for d in docs]

    missing = []
    for req in REQUIRED_DOCUMENTS:
        found = any(req.lower().split("(")[0].strip() in name for name in doc_names)
        if not found:
            missing.append(req)

    outdated = [d for d in docs
                if d.get("last_review") and
                (datetime.utcnow() - datetime.fromisoformat(
                    d["last_review"].replace("Z", ""))).days > 365]

    return {
        "required": len(REQUIRED_DOCUMENTS),
        "present": len(REQUIRED_DOCUMENTS) - len(missing),
        "missing": missing,
        "outdated_documents": len(outdated),
        "compliance_rate": round(
            (len(REQUIRED_DOCUMENTS) - len(missing)) / len(REQUIRED_DOCUMENTS) * 100, 1),
    }


def assess_risk_register(risk_register_path):
    """Assess risk register for completeness and currency."""
    with open(risk_register_path) as f:
        register = json.load(f)
    risks = register if isinstance(register, list) else register.get("risks", [])
    findings = []
    by_level = Counter()

    for risk in risks:
        level = risk.get("risk_level", risk.get("rating", "unknown")).lower()
        by_level[level] += 1
        if not risk.get("treatment", risk.get("mitigation")):
            findings.append({
                "risk": risk.get("id", risk.get("name", "")),
                "issue": "No treatment plan defined",
                "severity": "HIGH",
            })
        if not risk.get("owner"):
            findings.append({
                "risk": risk.get("id", ""),
                "issue": "No risk owner assigned",
                "severity": "MEDIUM",
            })
        if risk.get("treatment", "").lower() == "accept" and not risk.get("acceptance_authority"):
            findings.append({
                "risk": risk.get("id", ""),
                "issue": "Risk accepted without management approval",
                "severity": "HIGH",
            })

    return {
        "total_risks": len(risks),
        "by_level": dict(by_level),
        "findings": findings,
        "untreated": sum(1 for r in risks if not r.get("treatment")),
    }


def generate_gap_analysis(current_state):
    """Generate ISO 27001 gap analysis from current state assessment."""
    clauses = {
        "4_context": ["scope_defined", "interested_parties", "isms_scope_document"],
        "5_leadership": ["security_policy", "roles_responsibilities", "management_commitment"],
        "6_planning": ["risk_methodology", "risk_assessment", "soa", "security_objectives"],
        "7_support": ["resources", "competence", "awareness", "communication", "documented_info"],
        "8_operation": ["operational_planning", "risk_assessment_results", "risk_treatment"],
        "9_evaluation": ["monitoring", "internal_audit", "management_review"],
        "10_improvement": ["nonconformity_handling", "corrective_actions", "continual_improvement"],
    }
    gaps = {}
    for clause, requirements in clauses.items():
        clause_state = current_state.get(clause, {})
        met = sum(1 for r in requirements if clause_state.get(r, False))
        gaps[clause] = {
            "total": len(requirements),
            "met": met,
            "gaps": [r for r in requirements if not clause_state.get(r, False)],
            "compliance": round(met / len(requirements) * 100, 1),
        }
    return gaps


def main():
    parser = argparse.ArgumentParser(description="ISO 27001 ISMS Compliance Agent")
    parser.add_argument("--soa", help="Statement of Applicability JSON")
    parser.add_argument("--docs", help="Documentation inventory JSON")
    parser.add_argument("--risks", help="Risk register JSON")
    parser.add_argument("--state", help="Current state assessment JSON for gap analysis")
    parser.add_argument("--action", choices=["soa", "docs", "risks", "gaps", "full"],
                        default="full")
    parser.add_argument("--output", default="iso27001_report.json")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "results": {}}

    if args.action in ("soa", "full") and args.soa:
        result = assess_soa_completeness(args.soa)
        report["results"]["soa"] = result
        print(f"[+] SoA: {result['implementation_rate']}% implemented")

    if args.action in ("docs", "full") and args.docs:
        result = assess_documentation(args.docs)
        report["results"]["documentation"] = result
        print(f"[+] Documentation: {result['compliance_rate']}%, {len(result['missing'])} missing")

    if args.action in ("risks", "full") and args.risks:
        result = assess_risk_register(args.risks)
        report["results"]["risks"] = result
        print(f"[+] Risk register: {result['total_risks']} risks, {result['untreated']} untreated")

    if args.action in ("gaps", "full") and args.state:
        with open(args.state) as f:
            state = json.load(f)
        gaps = generate_gap_analysis(state)
        report["results"]["gap_analysis"] = gaps
        avg = sum(g["compliance"] for g in gaps.values()) / len(gaps)
        print(f"[+] Gap analysis: {avg:.1f}% average compliance")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-iso-27001-information-security-management/scripts/process.py =====
#!/usr/bin/env python3
"""
ISO 27001 ISMS Compliance Check Automation

Automates gap analysis, risk assessment tracking, Statement of Applicability
management, and audit readiness checks for ISO/IEC 27001:2022 implementation.
"""

import json
import csv
import os
import sys
from datetime import datetime, timedelta
from typing import Optional
from pathlib import Path
from dataclasses import dataclass, field, asdict
from enum import Enum


class ControlCategory(Enum):
    ORGANIZATIONAL = "A.5"
    PEOPLE = "A.6"
    PHYSICAL = "A.7"
    TECHNOLOGICAL = "A.8"


class ControlStatus(Enum):
    NOT_IMPLEMENTED = "Not Implemented"
    PARTIALLY_IMPLEMENTED = "Partially Implemented"
    FULLY_IMPLEMENTED = "Fully Implemented"
    NOT_APPLICABLE = "Not Applicable"


class RiskLevel(Enum):
    CRITICAL = "Critical"
    HIGH = "High"
    MEDIUM = "Medium"
    LOW = "Low"
    NEGLIGIBLE = "Negligible"


class RiskTreatment(Enum):
    MITIGATE = "Mitigate"
    TRANSFER = "Transfer"
    AVOID = "Avoid"
    ACCEPT = "Accept"


# Full Annex A control catalog for ISO 27001:2022
ANNEX_A_CONTROLS = {
    "A.5.1": {"title": "Policies for information security", "category": "Organizational"},
    "A.5.2": {"title": "Information security roles and responsibilities", "category": "Organizational"},
    "A.5.3": {"title": "Segregation of duties", "category": "Organizational"},
    "A.5.4": {"title": "Management responsibilities", "category": "Organizational"},
    "A.5.5": {"title": "Contact with authorities", "category": "Organizational"},
    "A.5.6": {"title": "Contact with special interest groups", "category": "Organizational"},
    "A.5.7": {"title": "Threat intelligence", "category": "Organizational", "new": True},
    "A.5.8": {"title": "Information security in project management", "category": "Organizational"},
    "A.5.9": {"title": "Inventory of information and other associated assets", "category": "Organizational"},
    "A.5.10": {"title": "Acceptable use of information and other associated assets", "category": "Organizational"},
    "A.5.11": {"title": "Return of assets", "category": "Organizational"},
    "A.5.12": {"title": "Classification of information", "category": "Organizational"},
    "A.5.13": {"title": "Labelling of information", "category": "Organizational"},
    "A.5.14": {"title": "Information transfer", "category": "Organizational"},
    "A.5.15": {"title": "Access control", "category": "Organizational"},
    "A.5.16": {"title": "Identity management", "category": "Organizational"},
    "A.5.17": {"title": "Authentication information", "category": "Organizational"},
    "A.5.18": {"title": "Access rights", "category": "Organizational"},
    "A.5.19": {"title": "Information security in supplier relationships", "category": "Organizational"},
    "A.5.20": {"title": "Addressing information security within supplier agreements", "category": "Organizational"},
    "A.5.21": {"title": "Managing information security in the ICT supply chain", "category": "Organizational"},
    "A.5.22": {"title": "Monitoring, review and change management of supplier services", "category": "Organizational"},
    "A.5.23": {"title": "Information security for use of cloud services", "category": "Organizational", "new": True},
    "A.5.24": {"title": "Information security incident management planning and preparation", "category": "Organizational"},
    "A.5.25": {"title": "Assessment and decision on information security events", "category": "Organizational"},
    "A.5.26": {"title": "Response to information security incidents", "category": "Organizational"},
    "A.5.27": {"title": "Learning from information security incidents", "category": "Organizational"},
    "A.5.28": {"title": "Collection of evidence", "category": "Organizational"},
    "A.5.29": {"title": "Information security during disruption", "category": "Organizational"},
    "A.5.30": {"title": "ICT readiness for business continuity", "category": "Organizational", "new": True},
    "A.5.31": {"title": "Legal, statutory, regulatory and contractual requirements", "category": "Organizational"},
    "A.5.32": {"title": "Intellectual property rights", "category": "Organizational"},
    "A.5.33": {"title": "Protection of records", "category": "Organizational"},
    "A.5.34": {"title": "Privacy and protection of PII", "category": "Organizational"},
    "A.5.35": {"title": "Independent review of information security", "category": "Organizational"},
    "A.5.36": {"title": "Compliance with policies, rules and standards", "category": "Organizational"},
    "A.5.37": {"title": "Documented operating procedures", "category": "Organizational"},
    "A.6.1": {"title": "Screening", "category": "People"},
    "A.6.2": {"title": "Terms and conditions of employment", "category": "People"},
    "A.6.3": {"title": "Information security awareness, education and training", "category": "People"},
    "A.6.4": {"title": "Disciplinary process", "category": "People"},
    "A.6.5": {"title": "Responsibilities after termination or change of employment", "category": "People"},
    "A.6.6": {"title": "Confidentiality or non-disclosure agreements", "category": "People"},
    "A.6.7": {"title": "Remote working", "category": "People"},
    "A.6.8": {"title": "Information security event reporting", "category": "People"},
    "A.7.1": {"title": "Physical security perimeters", "category": "Physical"},
    "A.7.2": {"title": "Physical entry", "category": "Physical"},
    "A.7.3": {"title": "Securing offices, rooms and facilities", "category": "Physical"},
    "A.7.4": {"title": "Physical security monitoring", "category": "Physical", "new": True},
    "A.7.5": {"title": "Protecting against physical and environmental threats", "category": "Physical"},
    "A.7.6": {"title": "Working in secure areas", "category": "Physical"},
    "A.7.7": {"title": "Clear desk and clear screen", "category": "Physical"},
    "A.7.8": {"title": "Equipment siting and protection", "category": "Physical"},
    "A.7.9": {"title": "Security of assets off-premises", "category": "Physical"},
    "A.7.10": {"title": "Storage media", "category": "Physical"},
    "A.7.11": {"title": "Supporting utilities", "category": "Physical"},
    "A.7.12": {"title": "Cabling security", "category": "Physical"},
    "A.7.13": {"title": "Equipment maintenance", "category": "Physical"},
    "A.7.14": {"title": "Secure disposal or re-use of equipment", "category": "Physical"},
    "A.8.1": {"title": "User endpoint devices", "category": "Technological"},
    "A.8.2": {"title": "Privileged access rights", "category": "Technological"},
    "A.8.3": {"title": "Information access restriction", "category": "Technological"},
    "A.8.4": {"title": "Access to source code", "category": "Technological"},
    "A.8.5": {"title": "Secure authentication", "category": "Technological"},
    "A.8.6": {"title": "Capacity management", "category": "Technological"},
    "A.8.7": {"title": "Protection against malware", "category": "Technological"},
    "A.8.8": {"title": "Management of technical vulnerabilities", "category": "Technological"},
    "A.8.9": {"title": "Configuration management", "category": "Technological", "new": True},
    "A.8.10": {"title": "Information deletion", "category": "Technological", "new": True},
    "A.8.11": {"title": "Data masking", "category": "Technological", "new": True},
    "A.8.12": {"title": "Data leakage prevention", "category": "Technological", "new": True},
    "A.8.13": {"title": "Information backup", "category": "Technological"},
    "A.8.14": {"title": "Redundancy of information processing facilities", "category": "Technological"},
    "A.8.15": {"title": "Logging", "category": "Technological"},
    "A.8.16": {"title": "Monitoring activities", "category": "Technological", "new": True},
    "A.8.17": {"title": "Clock synchronization", "category": "Technological"},
    "A.8.18": {"title": "Use of privileged utility programs", "category": "Technological"},
    "A.8.19": {"title": "Installation of software on operational systems", "category": "Technological"},
    "A.8.20": {"title": "Networks security", "category": "Technological"},
    "A.8.21": {"title": "Security of network services", "category": "Technological"},
    "A.8.22": {"title": "Segregation of networks", "category": "Technological"},
    "A.8.23": {"title": "Web filtering", "category": "Technological", "new": True},
    "A.8.24": {"title": "Use of cryptography", "category": "Technological"},
    "A.8.25": {"title": "Secure development life cycle", "category": "Technological"},
    "A.8.26": {"title": "Application security requirements", "category": "Technological"},
    "A.8.27": {"title": "Secure system architecture and engineering principles", "category": "Technological"},
    "A.8.28": {"title": "Secure coding", "category": "Technological", "new": True},
    "A.8.29": {"title": "Security testing in development and acceptance", "category": "Technological"},
    "A.8.30": {"title": "Outsourced development", "category": "Technological"},
    "A.8.31": {"title": "Separation of development, test and production environments", "category": "Technological"},
    "A.8.32": {"title": "Change management", "category": "Technological"},
    "A.8.33": {"title": "Test information", "category": "Technological"},
    "A.8.34": {"title": "Protection of information systems during audit testing", "category": "Technological"},
}


@dataclass
class RiskEntry:
    risk_id: str
    asset: str
    threat: str
    vulnerability: str
    likelihood: int  # 1-5
    impact: int      # 1-5
    existing_controls: list = field(default_factory=list)
    risk_level: str = ""
    treatment: str = ""
    treatment_controls: list = field(default_factory=list)
    owner: str = ""
    status: str = "Open"
    due_date: str = ""

    def __post_init__(self):
        score = self.likelihood * self.impact
        if score >= 20:
            self.risk_level = RiskLevel.CRITICAL.value
        elif score >= 15:
            self.risk_level = RiskLevel.HIGH.value
        elif score >= 8:
            self.risk_level = RiskLevel.MEDIUM.value
        elif score >= 4:
            self.risk_level = RiskLevel.LOW.value
        else:
            self.risk_level = RiskLevel.NEGLIGIBLE.value


@dataclass
class SoAEntry:
    control_id: str
    control_title: str
    applicable: bool
    justification: str
    implementation_status: str
    control_owner: str = ""
    linked_risks: list = field(default_factory=list)
    evidence_reference: str = ""


@dataclass
class AuditFinding:
    finding_id: str
    clause_or_control: str
    finding_type: str  # Major NCR, Minor NCR, Observation, OFI
    description: str
    evidence: str
    root_cause: str = ""
    corrective_action: str = ""
    responsible_person: str = ""
    due_date: str = ""
    status: str = "Open"
    closure_date: str = ""


class ISO27001ComplianceManager:
    """Manages ISO 27001 ISMS compliance checks and tracking."""

    def __init__(self, output_dir: str = "./iso27001_output"):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.risk_register: list[RiskEntry] = []
        self.soa_entries: list[SoAEntry] = []
        self.audit_findings: list[AuditFinding] = []

    def perform_gap_analysis(self) -> dict:
        """Perform gap analysis against all 93 Annex A controls."""
        print("\n" + "=" * 70)
        print("ISO 27001:2022 GAP ANALYSIS")
        print("=" * 70)

        results = {
            "date": datetime.now().isoformat(),
            "total_controls": len(ANNEX_A_CONTROLS),
            "categories": {},
            "controls": {},
            "summary": {}
        }

        status_counts = {s.value: 0 for s in ControlStatus}

        for control_id, control_info in ANNEX_A_CONTROLS.items():
            category = control_info["category"]
            if category not in results["categories"]:
                results["categories"][category] = {
                    "total": 0,
                    "implemented": 0,
                    "partial": 0,
                    "not_implemented": 0,
                    "not_applicable": 0,
                }

            # Default assessment - mark all as not implemented for gap analysis
            status = ControlStatus.NOT_IMPLEMENTED.value
            results["controls"][control_id] = {
                "title": control_info["title"],
                "category": category,
                "is_new_2022": control_info.get("new", False),
                "status": status,
                "gap_notes": "",
                "priority": "High" if control_info.get("new", False) else "Medium",
            }

            results["categories"][category]["total"] += 1
            results["categories"][category]["not_implemented"] += 1
            status_counts[status] += 1

        results["summary"] = {
            "compliance_percentage": 0.0,
            "controls_needing_implementation": status_counts[ControlStatus.NOT_IMPLEMENTED.value],
            "new_2022_controls": sum(
                1 for c in ANNEX_A_CONTROLS.values() if c.get("new", False)
            ),
            "status_breakdown": status_counts,
        }

        # Save gap analysis report
        report_path = self.output_dir / "gap_analysis_report.json"
        with open(report_path, "w") as f:
            json.dump(results, f, indent=2)

        print(f"\nTotal Controls Assessed: {results['total_controls']}")
        print(f"\nCategory Breakdown:")
        for cat, data in results["categories"].items():
            print(f"  {cat}: {data['total']} controls")
        print(f"\nNew 2022 Controls: {results['summary']['new_2022_controls']}")
        print(f"Gap Analysis Report saved to: {report_path}")

        return results

    def create_risk_register(self, risks: list[dict]) -> list[RiskEntry]:
        """Create and populate risk register with assessed risks."""
        print("\n" + "=" * 70)
        print("RISK REGISTER CREATION")
        print("=" * 70)

        for risk_data in risks:
            entry = RiskEntry(**risk_data)
            self.risk_register.append(entry)
            print(f"\n  [{entry.risk_id}] {entry.asset}")
            print(f"    Threat: {entry.threat}")
            print(f"    Vulnerability: {entry.vulnerability}")
            print(f"    Score: {entry.likelihood} x {entry.impact} = {entry.likelihood * entry.impact}")
            print(f"    Risk Level: {entry.risk_level}")
            print(f"    Treatment: {entry.treatment}")

        # Save risk register
        register_path = self.output_dir / "risk_register.json"
        with open(register_path, "w") as f:
            json.dump([asdict(r) for r in self.risk_register], f, indent=2)

        # Generate risk summary
        risk_summary = {}
        for entry in self.risk_register:
            risk_summary.setdefault(entry.risk_level, 0)
            risk_summary[entry.risk_level] += 1

        print(f"\n  Risk Summary:")
        for level, count in sorted(risk_summary.items()):
            print(f"    {level}: {count}")
        print(f"\n  Risk Register saved to: {register_path}")

        return self.risk_register

    def generate_soa(self, control_assessments: Optional[dict] = None) -> list[SoAEntry]:
        """Generate Statement of Applicability for all 93 controls."""
        print("\n" + "=" * 70)
        print("STATEMENT OF APPLICABILITY (SoA)")
        print("=" * 70)

        if control_assessments is None:
            control_assessments = {}

        for control_id, control_info in ANNEX_A_CONTROLS.items():
            assessment = control_assessments.get(control_id, {})
            entry = SoAEntry(
                control_id=control_id,
                control_title=control_info["title"],
                applicable=assessment.get("applicable", True),
                justification=assessment.get("justification", "Required by risk assessment"),
                implementation_status=assessment.get("status", ControlStatus.NOT_IMPLEMENTED.value),
                control_owner=assessment.get("owner", ""),
                linked_risks=assessment.get("linked_risks", []),
                evidence_reference=assessment.get("evidence", ""),
            )
            self.soa_entries.append(entry)

        # Calculate SoA statistics
        applicable_count = sum(1 for e in self.soa_entries if e.applicable)
        implemented_count = sum(
            1 for e in self.soa_entries
            if e.applicable and e.implementation_status == ControlStatus.FULLY_IMPLEMENTED.value
        )
        partial_count = sum(
            1 for e in self.soa_entries
            if e.applicable and e.implementation_status == ControlStatus.PARTIALLY_IMPLEMENTED.value
        )
        not_applicable_count = sum(1 for e in self.soa_entries if not e.applicable)

        compliance_pct = (
            (implemented_count / applicable_count * 100) if applicable_count > 0 else 0
        )

        print(f"\n  Total Controls: {len(self.soa_entries)}")
        print(f"  Applicable: {applicable_count}")
        print(f"  Not Applicable: {not_applicable_count}")
        print(f"  Fully Implemented: {implemented_count}")
        print(f"  Partially Implemented: {partial_count}")
        print(f"  Compliance Rate: {compliance_pct:.1f}%")

        # Save SoA
        soa_path = self.output_dir / "statement_of_applicability.json"
        with open(soa_path, "w") as f:
            json.dump([asdict(e) for e in self.soa_entries], f, indent=2)

        # Generate CSV version for auditors
        csv_path = self.output_dir / "soa_report.csv"
        with open(csv_path, "w", newline="") as f:
            writer = csv.writer(f)
            writer.writerow([
                "Control ID", "Title", "Category", "Applicable",
                "Justification", "Implementation Status", "Owner",
                "Linked Risks", "Evidence Reference", "New in 2022"
            ])
            for entry in self.soa_entries:
                control_info = ANNEX_A_CONTROLS.get(entry.control_id, {})
                writer.writerow([
                    entry.control_id,
                    entry.control_title,
                    control_info.get("category", ""),
                    "Yes" if entry.applicable else "No",
                    entry.justification,
                    entry.implementation_status,
                    entry.control_owner,
                    "; ".join(entry.linked_risks),
                    entry.evidence_reference,
                    "Yes" if control_info.get("new", False) else "No",
                ])

        print(f"  SoA saved to: {soa_path}")
        print(f"  SoA CSV saved to: {csv_path}")

        return self.soa_entries

    def check_audit_readiness(self) -> dict:
        """Check readiness for Stage 1 and Stage 2 certification audits."""
        print("\n" + "=" * 70)
        print("CERTIFICATION AUDIT READINESS CHECK")
        print("=" * 70)

        checks = {
            "mandatory_documents": {
                "Information Security Policy (A.5.1)": False,
                "ISMS Scope Document (Clause 4.3)": False,
                "Risk Assessment Methodology (Clause 6.1.2)": False,
                "Risk Assessment Report (Clause 8.2)": False,
                "Risk Treatment Plan (Clause 6.1.3)": False,
                "Statement of Applicability (Clause 6.1.3)": False,
                "Information Security Objectives (Clause 6.2)": False,
                "Competence Evidence (Clause 7.2)": False,
                "Operational Planning Documents (Clause 8.1)": False,
                "Internal Audit Programme and Reports (Clause 9.2)": False,
                "Management Review Minutes (Clause 9.3)": False,
                "Corrective Action Records (Clause 10.1)": False,
            },
            "process_checks": {
                "Risk assessment completed within last 12 months": False,
                "Internal audit covers all clauses and applicable controls": False,
                "Management review conducted": False,
                "All major nonconformities closed": False,
                "Security awareness training delivered": False,
                "Incident response process tested": False,
                "Business continuity plans tested": False,
                "Supplier security reviews conducted": False,
            },
            "control_implementation": {
                "All applicable Annex A controls implemented": False,
                "Evidence of control operation available": False,
                "Control effectiveness measured": False,
                "New 2022 controls addressed": False,
            },
        }

        # Auto-check based on existing data
        if self.soa_entries:
            checks["mandatory_documents"]["Statement of Applicability (Clause 6.1.3)"] = True

        if self.risk_register:
            checks["mandatory_documents"]["Risk Assessment Report (Clause 8.2)"] = True
            checks["process_checks"]["Risk assessment completed within last 12 months"] = True

        total_checks = sum(len(v) for v in checks.values())
        passed_checks = sum(
            sum(1 for passed in v.values() if passed) for v in checks.values()
        )
        readiness_pct = (passed_checks / total_checks * 100) if total_checks > 0 else 0

        print(f"\n  Readiness Score: {passed_checks}/{total_checks} ({readiness_pct:.1f}%)")

        for section, items in checks.items():
            print(f"\n  {section.replace('_', ' ').title()}:")
            for item, status in items.items():
                icon = "[PASS]" if status else "[FAIL]"
                print(f"    {icon} {item}")

        if readiness_pct < 80:
            print(f"\n  WARNING: Readiness below 80%. Address gaps before scheduling Stage 1 audit.")
        elif readiness_pct < 100:
            print(f"\n  NOTICE: Some items pending. Complete before Stage 2 audit.")
        else:
            print(f"\n  READY: All checks passed. Proceed with certification audit.")

        # Save readiness report
        report = {
            "date": datetime.now().isoformat(),
            "readiness_percentage": readiness_pct,
            "passed": passed_checks,
            "total": total_checks,
            "checks": checks,
        }
        report_path = self.output_dir / "audit_readiness_report.json"
        with open(report_path, "w") as f:
            json.dump(report, f, indent=2)

        print(f"\n  Readiness Report saved to: {report_path}")
        return report

    def track_audit_findings(self, findings: list[dict]) -> list[AuditFinding]:
        """Track internal or external audit findings and corrective actions."""
        print("\n" + "=" * 70)
        print("AUDIT FINDINGS TRACKER")
        print("=" * 70)

        for finding_data in findings:
            finding = AuditFinding(**finding_data)
            self.audit_findings.append(finding)
            print(f"\n  [{finding.finding_id}] {finding.finding_type}")
            print(f"    Clause/Control: {finding.clause_or_control}")
            print(f"    Description: {finding.description}")
            print(f"    Status: {finding.status}")

        # Summary
        type_counts = {}
        for f in self.audit_findings:
            type_counts.setdefault(f.finding_type, 0)
            type_counts[f.finding_type] += 1

        print(f"\n  Finding Summary:")
        for ftype, count in type_counts.items():
            print(f"    {ftype}: {count}")

        open_count = sum(1 for f in self.audit_findings if f.status == "Open")
        print(f"    Open Findings: {open_count}")

        # Save findings
        findings_path = self.output_dir / "audit_findings.json"
        with open(findings_path, "w") as f:
            json.dump([asdict(af) for af in self.audit_findings], f, indent=2)

        print(f"\n  Findings saved to: {findings_path}")
        return self.audit_findings

    def generate_compliance_dashboard(self) -> dict:
        """Generate overall ISMS compliance dashboard metrics."""
        print("\n" + "=" * 70)
        print("ISMS COMPLIANCE DASHBOARD")
        print("=" * 70)

        dashboard = {
            "generated": datetime.now().isoformat(),
            "risk_register": {
                "total_risks": len(self.risk_register),
                "by_level": {},
                "by_treatment": {},
                "open_risks": sum(1 for r in self.risk_register if r.status == "Open"),
            },
            "soa": {
                "total_controls": len(ANNEX_A_CONTROLS),
                "applicable": sum(1 for e in self.soa_entries if e.applicable),
                "fully_implemented": sum(
                    1 for e in self.soa_entries
                    if e.applicable and e.implementation_status == ControlStatus.FULLY_IMPLEMENTED.value
                ),
                "partially_implemented": sum(
                    1 for e in self.soa_entries
                    if e.applicable and e.implementation_status == ControlStatus.PARTIALLY_IMPLEMENTED.value
                ),
                "not_implemented": sum(
                    1 for e in self.soa_entries
                    if e.applicable and e.implementation_status == ControlStatus.NOT_IMPLEMENTED.value
                ),
            },
            "audit_findings": {
                "total": len(self.audit_findings),
                "open": sum(1 for f in self.audit_findings if f.status == "Open"),
                "major_ncrs": sum(1 for f in self.audit_findings if f.finding_type == "Major NCR"),
                "minor_ncrs": sum(1 for f in self.audit_findings if f.finding_type == "Minor NCR"),
            },
        }

        # Risk breakdown
        for r in self.risk_register:
            dashboard["risk_register"]["by_level"].setdefault(r.risk_level, 0)
            dashboard["risk_register"]["by_level"][r.risk_level] += 1
            if r.treatment:
                dashboard["risk_register"]["by_treatment"].setdefault(r.treatment, 0)
                dashboard["risk_register"]["by_treatment"][r.treatment] += 1

        # Compliance rate
        applicable = dashboard["soa"]["applicable"]
        implemented = dashboard["soa"]["fully_implemented"]
        dashboard["soa"]["compliance_rate"] = (
            f"{(implemented / applicable * 100):.1f}%" if applicable > 0 else "N/A"
        )

        print(f"\n  Risk Register:")
        print(f"    Total Risks: {dashboard['risk_register']['total_risks']}")
        print(f"    Open Risks: {dashboard['risk_register']['open_risks']}")
        for level, count in dashboard["risk_register"]["by_level"].items():
            print(f"    {level}: {count}")

        print(f"\n  Statement of Applicability:")
        print(f"    Total Controls: {dashboard['soa']['total_controls']}")
        print(f"    Applicable: {dashboard['soa']['applicable']}")
        print(f"    Fully Implemented: {dashboard['soa']['fully_implemented']}")
        print(f"    Compliance Rate: {dashboard['soa']['compliance_rate']}")

        print(f"\n  Audit Findings:")
        print(f"    Total: {dashboard['audit_findings']['total']}")
        print(f"    Open: {dashboard['audit_findings']['open']}")
        print(f"    Major NCRs: {dashboard['audit_findings']['major_ncrs']}")

        # Save dashboard
        dashboard_path = self.output_dir / "compliance_dashboard.json"
        with open(dashboard_path, "w") as f:
            json.dump(dashboard, f, indent=2)

        print(f"\n  Dashboard saved to: {dashboard_path}")
        return dashboard


def main():
    """Run ISO 27001 compliance assessment."""
    manager = ISO27001ComplianceManager()

    # Phase 1: Gap Analysis
    gap_results = manager.perform_gap_analysis()

    # Phase 2: Risk Register with sample risks
    sample_risks = [
        {
            "risk_id": "RISK-001",
            "asset": "Customer Database",
            "threat": "SQL Injection Attack",
            "vulnerability": "Unparameterized database queries",
            "likelihood": 4,
            "impact": 5,
            "existing_controls": ["WAF", "Input validation"],
            "treatment": RiskTreatment.MITIGATE.value,
            "treatment_controls": ["A.8.26", "A.8.28", "A.8.29"],
            "owner": "CTO",
            "due_date": (datetime.now() + timedelta(days=30)).strftime("%Y-%m-%d"),
        },
        {
            "risk_id": "RISK-002",
            "asset": "Employee Laptops",
            "threat": "Ransomware Infection",
            "vulnerability": "Outdated endpoint protection",
            "likelihood": 4,
            "impact": 4,
            "existing_controls": ["Antivirus"],
            "treatment": RiskTreatment.MITIGATE.value,
            "treatment_controls": ["A.8.1", "A.8.7", "A.8.8"],
            "owner": "IT Manager",
            "due_date": (datetime.now() + timedelta(days=14)).strftime("%Y-%m-%d"),
        },
        {
            "risk_id": "RISK-003",
            "asset": "Cloud Infrastructure (AWS)",
            "threat": "Unauthorized Access via Misconfigured IAM",
            "vulnerability": "Overly permissive IAM policies",
            "likelihood": 3,
            "impact": 5,
            "existing_controls": ["MFA for root account"],
            "treatment": RiskTreatment.MITIGATE.value,
            "treatment_controls": ["A.5.15", "A.5.23", "A.8.2", "A.8.3"],
            "owner": "Cloud Architect",
            "due_date": (datetime.now() + timedelta(days=21)).strftime("%Y-%m-%d"),
        },
        {
            "risk_id": "RISK-004",
            "asset": "Physical Server Room",
            "threat": "Unauthorized Physical Access",
            "vulnerability": "No CCTV monitoring, single-factor entry",
            "likelihood": 2,
            "impact": 4,
            "existing_controls": ["Key card access"],
            "treatment": RiskTreatment.MITIGATE.value,
            "treatment_controls": ["A.7.1", "A.7.2", "A.7.4"],
            "owner": "Facilities Manager",
            "due_date": (datetime.now() + timedelta(days=45)).strftime("%Y-%m-%d"),
        },
        {
            "risk_id": "RISK-005",
            "asset": "Business Operations",
            "threat": "Key Person Dependency",
            "vulnerability": "Single administrator with no backup",
            "likelihood": 3,
            "impact": 3,
            "existing_controls": [],
            "treatment": RiskTreatment.MITIGATE.value,
            "treatment_controls": ["A.5.2", "A.5.3", "A.6.2"],
            "owner": "HR Director",
            "due_date": (datetime.now() + timedelta(days=60)).strftime("%Y-%m-%d"),
        },
    ]

    risk_register = manager.create_risk_register(sample_risks)

    # Phase 3: Statement of Applicability
    soa = manager.generate_soa()

    # Phase 4: Audit Readiness Check
    readiness = manager.check_audit_readiness()

    # Phase 5: Sample Audit Findings
    sample_findings = [
        {
            "finding_id": "FIND-001",
            "clause_or_control": "A.5.7",
            "finding_type": "Minor NCR",
            "description": "Threat intelligence process not formally documented",
            "evidence": "No threat intelligence procedure or subscription found",
            "root_cause": "New 2022 control not yet addressed",
            "corrective_action": "Establish threat intelligence feeds and document process",
            "responsible_person": "Security Analyst",
            "due_date": (datetime.now() + timedelta(days=30)).strftime("%Y-%m-%d"),
        },
        {
            "finding_id": "FIND-002",
            "clause_or_control": "Clause 9.2",
            "finding_type": "Minor NCR",
            "description": "Internal audit programme does not cover all Annex A controls",
            "evidence": "Audit plan only covers 60 of 93 applicable controls",
            "root_cause": "Audit programme not updated for 2022 edition",
            "corrective_action": "Revise audit programme to cover all applicable controls",
            "responsible_person": "Internal Auditor",
            "due_date": (datetime.now() + timedelta(days=14)).strftime("%Y-%m-%d"),
        },
    ]

    findings = manager.track_audit_findings(sample_findings)

    # Phase 6: Compliance Dashboard
    dashboard = manager.generate_compliance_dashboard()

    print("\n" + "=" * 70)
    print("ISO 27001 COMPLIANCE ASSESSMENT COMPLETE")
    print("=" * 70)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-pci-dss-compliance-controls/SKILL.md =====
---
name: implementing-pci-dss-compliance-controls
description: Implements PCI DSS 4.0.1's 12 requirements across 6 control objectives
  for organizations that store, process, or transmit cardholder data, including the
  customized validation approach, enhanced authentication, and continuous monitoring
  controls introduced by the 51 requirements mandatory since March 2025. Use when
  scoping a cardholder data environment, building PCI DSS 4.0.1 compliance controls,
  or preparing for a PCI assessment.
domain: cybersecurity
subdomain: compliance-governance
tags:
- compliance
- governance
- pci-dss
- payment-security
- cardholder-data
nist_csf:
- GV.PO-01
- PR.DS-01
- PR.AA-01
- DE.CM-01
- ID.RA-01
version: '1.0'
author: mahipal
license: Apache-2.0
mitre_attack:
- T1078
- T1530
- T1685.002
---
# Implementing PCI DSS Compliance Controls

## Overview
PCI DSS 4.0.1 establishes 12 requirements across 6 control objectives for organizations that store, process, or transmit cardholder data. With PCI DSS 3.2.1 retiring April 2024 and 51 new requirements becoming mandatory March 31, 2025, this skill covers implementing all requirements including the new customized validation approach, enhanced authentication, and continuous monitoring controls.


## When to Use

- When deploying or configuring implementing pci dss compliance controls capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites
- Understanding of payment card processing flows and cardholder data environment (CDE)
- Knowledge of network segmentation and security architecture
- Access to cardholder data environment for scoping
- Understanding of PCI compliance validation levels (merchant levels 1-4, service provider levels 1-2)

## Core Concepts

### 12 PCI DSS Requirements by Control Objective

**Build and Maintain a Secure Network and Systems**
1. Install and maintain network security controls (firewalls, NSCs)
2. Apply secure configurations to all system components

**Protect Account Data**
3. Protect stored account data (encryption, tokenization, truncation)
4. Protect cardholder data with strong cryptography during transmission

**Maintain a Vulnerability Management Program**
5. Protect all systems and networks from malicious software
6. Develop and maintain secure systems and software

**Implement Strong Access Control Measures**
7. Restrict access to system components and cardholder data by business need to know
8. Identify users and authenticate access to system components
9. Restrict physical access to cardholder data

**Regularly Monitor and Test Networks**
10. Log and monitor all access to system components and cardholder data
11. Test security of systems and networks regularly

**Maintain an Information Security Policy**
12. Support information security with organizational policies and programs

### Key PCI DSS 4.0 Changes
- **Customized Approach**: Alternative to defined approach, allowing custom control design with objective-based validation
- **MFA for all CDE access**: Extended beyond admin to all access to cardholder data (Req 8.4.2)
- **Targeted Risk Analysis**: Organizations perform their own risk analysis for flexible requirements
- **Authenticated Vulnerability Scanning**: Internal scans must use authenticated scanning (Req 11.3.1.1)
- **Anti-phishing mechanisms**: Technical controls to detect and protect against phishing (Req 5.4.1)
- **Automated log review**: Automated mechanisms for review of audit logs (Req 10.4.1.1)

## Workflow

### Phase 1: Scoping and Assessment (Weeks 1-4)
1. Identify all cardholder data flows (card present, card not present, storage)
2. Define Cardholder Data Environment (CDE) boundaries
3. Validate network segmentation effectiveness
4. Determine compliance validation level
5. Conduct PCI DSS gap assessment against all 12 requirements

### Phase 2: Network and System Security (Weeks 5-12)
1. Deploy and configure network security controls (Req 1)
2. Implement network segmentation to minimize CDE scope
3. Harden system configurations using CIS Benchmarks (Req 2)
4. Implement WAF for public-facing web applications (Req 6.4.1)
5. Deploy anti-malware on all in-scope systems (Req 5)

### Phase 3: Data Protection (Weeks 13-20)
1. Implement encryption for stored cardholder data (Req 3)
2. Deploy tokenization where possible to reduce scope
3. Enforce TLS 1.2+ for all cardholder data transmission (Req 4)
4. Implement key management procedures
5. Deploy data discovery tools to locate unencrypted cardholder data

### Phase 4: Access Controls (Weeks 21-28)
1. Implement RBAC based on business need to know (Req 7)
2. Deploy MFA for all access to CDE (Req 8)
3. Implement unique user IDs for all users
4. Enforce password policies meeting PCI DSS 4.0 requirements
5. Implement physical access controls for CDE facilities (Req 9)

### Phase 5: Monitoring and Testing (Weeks 29-36)
1. Deploy centralized logging for all CDE components (Req 10)
2. Implement automated log review mechanisms
3. Conduct internal and external vulnerability scans (Req 11)
4. Perform penetration testing (internal and external)
5. Implement file integrity monitoring (FIM) for critical files

### Phase 6: Policy and Governance (Weeks 37-42)
1. Develop comprehensive information security policy (Req 12)
2. Implement security awareness training including anti-phishing
3. Establish incident response plan specific to cardholder data
4. Conduct targeted risk analyses for flexible requirements
5. Document and validate all controls for assessment

## Key Artifacts
- CDE Scope Documentation and Network Diagrams
- Self-Assessment Questionnaire (SAQ) or Report on Compliance (ROC)
- Attestation of Compliance (AOC)
- Quarterly ASV Scan Reports
- Annual Penetration Test Report
- Risk Assessment Documentation
- Security Policies and Procedures

## Common Pitfalls
- Scope creep due to inadequate network segmentation
- Storing prohibited data (CVV, full track data) after authorization
- Missing the March 2025 deadline for new mandatory requirements
- Treating PCI DSS as annual compliance rather than continuous security
- Not including cloud and container environments in CDE scope

## References
- PCI DSS v4.0.1: https://www.pcisecuritystandards.org/document_library/
- PCI SSC Quick Reference Guide
- PCI DSS 4.0 Summary of Changes
- UpGuard PCI DSS 4.0 Guide: https://www.upguard.com/blog/pci-compliance

===== END FILE =====


===== FILE: skills/implementing-pci-dss-compliance-controls/LICENSE =====

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


===== FILE: skills/implementing-pci-dss-compliance-controls/assets/template.md =====
# Implementing PCI DSS Compliance Controls - Audit Checklist

## Organization Information
| Field | Value |
|-------|-------|
| Organization Name | |
| Assessment Date | |
| Assessor | |
| Standard | PCI DSS v4.0.1 |

---

## Compliance Checklist

### 1. Requirement 1: Install and maintain network security controls
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 2. Requirement 2: Apply secure configurations to all system components
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 3. Requirement 3: Protect stored account data
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 4. Requirement 4: Protect cardholder data with strong cryptography during transmission
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 5. Requirement 5: Protect all systems and networks from malicious software
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 6. Requirement 6: Develop and maintain secure systems and software
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 7. Requirement 7: Restrict access by business need to know
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 8. Requirement 8: Identify users and authenticate access
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 9. Requirement 9: Restrict physical access to cardholder data
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 10. Requirement 10: Log and monitor all access to system components
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 11. Requirement 11: Test security of systems and networks regularly
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

### 12. Requirement 12: Support information security with policies and programs
- [ ] Policy/procedure documented
- [ ] Controls implemented
- [ ] Evidence available
- [ ] Tested and verified
- **Status**: Compliant / Partially Compliant / Non-Compliant / N/A
- **Evidence**: _______________
- **Notes**: _______________

---

## Summary

| Status | Count |
|--------|-------|
| Compliant | |
| Partially Compliant | |
| Non-Compliant | |
| N/A | |

## Sign-off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Assessor | | | |
| Security Manager | | | |
| Senior Management | | | |

===== END FILE =====


===== FILE: skills/implementing-pci-dss-compliance-controls/references/api-reference.md =====
# API Reference: PCI DSS Compliance Control Audit

## Libraries Used

| Library | Purpose |
|---------|---------|
| `requests` | API calls to scan engines and cloud services |
| `jinja2` | Generate compliance assessment reports |
| `json` | Parse control status and evidence data |
| `subprocess` | Run network segmentation and encryption checks |
| `csv` | Export compliance matrices |

## Installation

```bash
pip install requests jinja2
```

## PCI DSS v4.0 Requirements Map

| Requirement | Title | Automated Checks |
|-------------|-------|-----------------|
| 1 | Install and maintain network security controls | Firewall rules, segmentation testing |
| 2 | Apply secure configurations | Default credential scan, hardening baselines |
| 3 | Protect stored account data | Encryption at rest, key management |
| 4 | Protect cardholder data with strong cryptography during transmission | TLS version, cipher suites |
| 5 | Protect all systems against malware | AV status, EDR coverage |
| 6 | Develop and maintain secure systems | Vulnerability scans, SAST/DAST |
| 7 | Restrict access by business need to know | RBAC review, access logs |
| 8 | Identify users and authenticate access | MFA status, password policy |
| 9 | Restrict physical access to cardholder data | Physical access logs |
| 10 | Log and monitor all access | Log aggregation, SIEM alerts |
| 11 | Test security of systems regularly | Penetration tests, IDS/IPS |
| 12 | Support information security with policies | Policy review dates |

## Core Compliance Checks

### Requirement 2: Default Credentials Check
```python
import requests

DEFAULT_CREDS = [
    ("admin", "admin"), ("admin", "password"), ("root", "root"),
    ("admin", ""), ("user", "user"), ("test", "test"),
]

def check_default_credentials(target_url):
    findings = []
    for username, password in DEFAULT_CREDS:
        try:
            resp = requests.post(
                f"{target_url}/login",
                data={"username": username, "password": password},
                timeout=10,
                allow_redirects=False,
            )
            if resp.status_code in (200, 302):
                findings.append({
                    "target": target_url,
                    "username": username,
                    "requirement": "2.2.2",
                    "severity": "critical",
                })
        except requests.RequestException:
            pass
    return findings
```

### Requirement 4: TLS Configuration Check
```python
import ssl
import socket

def check_tls_config(hostname, port=443):
    findings = []
    context = ssl.create_default_context()
    with socket.create_connection((hostname, port), timeout=10) as sock:
        with context.wrap_socket(sock, server_hostname=hostname) as ssock:
            protocol = ssock.version()
            cipher = ssock.cipher()
            cert = ssock.getpeercert()

            # Check TLS version (must be 1.2+)
            if protocol in ("TLSv1", "TLSv1.1"):
                findings.append({
                    "check": "tls_version",
                    "requirement": "4.2.1",
                    "severity": "high",
                    "detail": f"Weak TLS version: {protocol}",
                })

            # Check cipher strength
            if cipher and cipher[2] < 128:
                findings.append({
                    "check": "cipher_strength",
                    "requirement": "4.2.1",
                    "severity": "high",
                    "detail": f"Weak cipher: {cipher[0]} ({cipher[2]} bits)",
                })

    return {"protocol": protocol, "cipher": cipher[0], "findings": findings}
```

### Requirement 8: MFA and Password Policy
```python
def check_password_policy(identity_provider_url, headers):
    resp = requests.get(
        f"{identity_provider_url}/api/v1/policies/password",
        headers=headers,
        timeout=30,
    )
    policy = resp.json()

    findings = []
    if policy.get("minLength", 0) < 12:
        findings.append({
            "check": "password_length",
            "requirement": "8.3.6",
            "severity": "medium",
            "detail": f"Min password length {policy['minLength']} < 12",
        })
    if not policy.get("requireUppercase"):
        findings.append({
            "check": "password_complexity",
            "requirement": "8.3.6",
            "severity": "low",
            "detail": "Uppercase not required",
        })
    return findings
```

### Requirement 10: Log Monitoring Check
```python
def check_logging_coverage(siem_url, siem_headers):
    """Verify all CDE systems forward logs to SIEM."""
    resp = requests.get(
        f"{siem_url}/api/sources",
        headers=siem_headers,
        timeout=30,
    )
    active_sources = resp.json().get("sources", [])
    return {
        "total_sources": len(active_sources),
        "requirement": "10.2",
        "active": [s for s in active_sources if s.get("status") == "active"],
        "inactive": [s for s in active_sources if s.get("status") != "active"],
    }
```

### Generate Compliance Report
```python
from jinja2 import Template

REPORT_TEMPLATE = """
# PCI DSS v4.0 Compliance Assessment

Generated: {{ timestamp }}
Scope: {{ scope }}

## Summary
- Total Controls: {{ total }}
- Compliant: {{ compliant }}
- Non-Compliant: {{ non_compliant }}
- Compliance Rate: {{ rate }}%

## Findings
{% for finding in findings %}
### {{ finding.requirement }} — {{ finding.check }}
- **Severity**: {{ finding.severity }}
- **Detail**: {{ finding.detail }}
{% endfor %}
"""

def generate_report(findings, scope, timestamp):
    compliant = sum(1 for f in findings if not f.get("findings"))
    template = Template(REPORT_TEMPLATE)
    return template.render(
        timestamp=timestamp,
        scope=scope,
        total=len(findings),
        compliant=compliant,
        non_compliant=len(findings) - compliant,
        rate=round(compliant / len(findings) * 100, 1),
        findings=[f for f in findings if f.get("findings")],
    )
```

## Output Format

```json
{
  "assessment_date": "2025-01-15",
  "pci_dss_version": "4.0",
  "scope": "Cardholder Data Environment",
  "total_requirements": 12,
  "compliant": 9,
  "non_compliant": 3,
  "findings": [
    {
      "requirement": "4.2.1",
      "check": "tls_version",
      "severity": "high",
      "detail": "Payment gateway using TLSv1.1",
      "remediation": "Upgrade to TLS 1.2 or higher"
    }
  ]
}
```

===== END FILE =====


===== FILE: skills/implementing-pci-dss-compliance-controls/references/standards.md =====
# Implementing PCI DSS Compliance Controls - Standards Reference

## Primary Standard
- **Standard**: PCI DSS v4.0.1
- **Governing Body**: PCI Security Standards Council

## Key Requirements
- Requirement 1: Install and maintain network security controls
- Requirement 2: Apply secure configurations to all system components
- Requirement 3: Protect stored account data
- Requirement 4: Protect cardholder data with strong cryptography during transmission
- Requirement 5: Protect all systems and networks from malicious software
- Requirement 6: Develop and maintain secure systems and software
- Requirement 7: Restrict access by business need to know
- Requirement 8: Identify users and authenticate access
- Requirement 9: Restrict physical access to cardholder data
- Requirement 10: Log and monitor all access to system components
- Requirement 11: Test security of systems and networks regularly
- Requirement 12: Support information security with policies and programs

## Cross-References
- ISO/IEC 27001:2022
- NIST Cybersecurity Framework 2.0
- CIS Controls v8.1
- COBIT 2019

===== END FILE =====


===== FILE: skills/implementing-pci-dss-compliance-controls/references/workflows.md =====
# Implementing PCI DSS Compliance Controls - Workflows

## Workflow 1: Assessment and Planning

```
Start
  |
  v
[Scope Definition]
  - Define boundaries and objectives
  - Identify stakeholders
  - Gather existing documentation
  |
  v
[Current State Assessment]
  - Review existing controls
  - Identify gaps against requirements
  - Document findings
  |
  v
[Gap Analysis]
  - Compare current vs required state
  - Prioritize gaps by risk
  - Estimate remediation effort
  |
  v
[Remediation Planning]
  - Define action items with owners
  - Set timelines and milestones
  - Allocate resources and budget
  |
  v
End
```

## Workflow 2: Implementation

```
Start
  |
  v
[Policy and Procedure Development]
  - Draft policies aligned to standard
  - Review with stakeholders
  - Obtain management approval
  |
  v
[Technical Control Deployment]
  - Implement technical controls
  - Configure monitoring and alerting
  - Validate control effectiveness
  |
  v
[Training and Awareness]
  - Train relevant personnel
  - Communicate policy changes
  - Document training completion
  |
  v
[Verification and Testing]
  - Test controls against requirements
  - Document evidence of operation
  - Address deficiencies
  |
  v
End
```

## Workflow 3: Ongoing Compliance

```
Start
  |
  v
[Continuous Monitoring]
  - Monitor control effectiveness
  - Track compliance metrics
  - Report to management
  |
  v
[Periodic Review]
  - Annual reassessment
  - Update for regulatory changes
  - Incorporate lessons learned
  |
  v
[Audit and Certification]
  - Internal audit programme
  - External audit/assessment
  - Address findings
  |
  v
End
```

===== END FILE =====


===== FILE: skills/implementing-pci-dss-compliance-controls/scripts/agent.py =====
#!/usr/bin/env python3
"""PCI DSS compliance control audit agent.

Audits systems and configurations against PCI DSS v4.0 requirements
including network segmentation, encryption, access controls, logging,
vulnerability management, and secure configuration checks.
"""
import argparse
import json
import os
import re
import socket
import ssl
import subprocess
import sys
from datetime import datetime, timezone


def check_tls_configuration(host, port=443):
    """PCI DSS Req 4.2.1 - Strong cryptography for transmission."""
    findings = []
    print(f"[*] Req 4.2.1: Checking TLS on {host}:{port}")
    try:
        context = ssl.create_default_context()
        with socket.create_connection((host, port), timeout=10) as sock:
            with context.wrap_socket(sock, server_hostname=host) as ssock:
                protocol = ssock.version()
                cipher = ssock.cipher()
                if protocol in ("TLSv1.0", "TLSv1.1", "SSLv3", "SSLv2"):
                    findings.append({
                        "requirement": "4.2.1", "check": "TLS Protocol Version",
                        "status": "FAIL", "severity": "CRITICAL",
                        "detail": f"Deprecated protocol: {protocol}",
                    })
                else:
                    findings.append({
                        "requirement": "4.2.1", "check": "TLS Protocol Version",
                        "status": "PASS", "severity": "INFO",
                        "detail": f"Protocol: {protocol}",
                    })
                if cipher:
                    weak_ciphers = ["RC4", "DES", "3DES", "NULL", "EXPORT", "MD5"]
                    if any(w in cipher[0] for w in weak_ciphers):
                        findings.append({
                            "requirement": "4.2.1", "check": "Cipher Strength",
                            "status": "FAIL", "severity": "HIGH",
                            "detail": f"Weak cipher: {cipher[0]}",
                        })
                    else:
                        findings.append({
                            "requirement": "4.2.1", "check": "Cipher Strength",
                            "status": "PASS", "severity": "INFO",
                            "detail": f"Cipher: {cipher[0]} ({cipher[2]} bits)",
                        })
    except Exception as e:
        findings.append({
            "requirement": "4.2.1", "check": "TLS Connection",
            "status": "ERROR", "severity": "HIGH", "detail": str(e)[:100],
        })
    return findings


def check_password_policy():
    """PCI DSS Req 8.3.6 - Password complexity requirements."""
    findings = []
    print("[*] Req 8.3.6: Checking password policy")

    if sys.platform != "win32":
        # Check PAM password quality
        pam_files = ["/etc/pam.d/common-password", "/etc/pam.d/system-auth",
                     "/etc/security/pwquality.conf"]
        for pam_file in pam_files:
            if os.path.isfile(pam_file):
                with open(pam_file, "r") as f:
                    content = f.read()
                if "minlen" in content:
                    match = re.search(r'minlen\s*=\s*(\d+)', content)
                    if match and int(match.group(1)) >= 12:
                        findings.append({
                            "requirement": "8.3.6", "check": "Min password length",
                            "status": "PASS", "severity": "INFO",
                            "detail": f"minlen={match.group(1)} in {pam_file}",
                        })
                    else:
                        findings.append({
                            "requirement": "8.3.6", "check": "Min password length",
                            "status": "FAIL", "severity": "HIGH",
                            "detail": f"Password minlen < 12 in {pam_file}",
                        })
                break
        else:
            findings.append({
                "requirement": "8.3.6", "check": "Password policy config",
                "status": "WARN", "severity": "MEDIUM",
                "detail": "Could not find PAM password config",
            })
    return findings


def check_audit_logging():
    """PCI DSS Req 10.2 - Audit logging configuration."""
    findings = []
    print("[*] Req 10.2: Checking audit logging")

    if sys.platform != "win32":
        # Check auditd
        result = subprocess.run(
            ["systemctl", "is-active", "auditd"],
            capture_output=True, text=True, timeout=10,
        )
        if result.stdout.strip() == "active":
            findings.append({
                "requirement": "10.2", "check": "Audit daemon running",
                "status": "PASS", "severity": "INFO",
            })
        else:
            findings.append({
                "requirement": "10.2", "check": "Audit daemon running",
                "status": "FAIL", "severity": "CRITICAL",
                "detail": "auditd is not running",
            })

        # Check syslog
        for syslog in ["rsyslog", "syslog-ng"]:
            result = subprocess.run(
                ["systemctl", "is-active", syslog],
                capture_output=True, text=True, timeout=10,
            )
            if result.stdout.strip() == "active":
                findings.append({
                    "requirement": "10.2", "check": f"{syslog} running",
                    "status": "PASS", "severity": "INFO",
                })
                break
    return findings


def check_file_integrity():
    """PCI DSS Req 11.5.2 - File integrity monitoring."""
    findings = []
    print("[*] Req 11.5.2: Checking file integrity monitoring")

    fim_tools = {
        "aide": ["/usr/bin/aide", "/usr/sbin/aide"],
        "ossec": ["/var/ossec/bin/ossec-syscheckd"],
        "tripwire": ["/usr/sbin/tripwire"],
        "samhain": ["/usr/local/sbin/samhain"],
    }

    found_fim = False
    for tool_name, paths in fim_tools.items():
        for path in paths:
            if os.path.isfile(path):
                findings.append({
                    "requirement": "11.5.2", "check": f"FIM tool: {tool_name}",
                    "status": "PASS", "severity": "INFO",
                    "detail": f"Found at {path}",
                })
                found_fim = True
                break

    if not found_fim:
        findings.append({
            "requirement": "11.5.2", "check": "File integrity monitoring",
            "status": "FAIL", "severity": "HIGH",
            "detail": "No FIM tool detected (AIDE, OSSEC, Tripwire, Samhain)",
        })

    return findings


def check_default_credentials():
    """PCI DSS Req 2.2.2 - Change vendor defaults."""
    findings = []
    print("[*] Req 2.2.2: Checking for default credentials")

    # Check for common default accounts
    if os.path.isfile("/etc/passwd"):
        with open("/etc/passwd", "r") as f:
            for line in f:
                parts = line.strip().split(":")
                if len(parts) >= 7:
                    username = parts[0]
                    shell = parts[6]
                    if username in ("guest", "test", "demo", "admin") and shell not in ("/usr/sbin/nologin", "/bin/false"):
                        findings.append({
                            "requirement": "2.2.2", "check": f"Default account: {username}",
                            "status": "FAIL", "severity": "HIGH",
                            "detail": f"Account '{username}' has login shell: {shell}",
                        })

    return findings


def check_network_segmentation(target_ip, ports=None):
    """PCI DSS Req 1.3 - Network segmentation check."""
    findings = []
    if not ports:
        ports = [22, 80, 443, 3306, 5432, 1433, 6379, 9200, 27017]
    print(f"[*] Req 1.3: Checking network segmentation to {target_ip}")

    for port in ports:
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(3)
            result = sock.connect_ex((target_ip, port))
            sock.close()
            if result == 0:
                findings.append({
                    "requirement": "1.3", "check": f"Port {port} reachable",
                    "status": "WARN", "severity": "MEDIUM",
                    "detail": f"{target_ip}:{port} is open from this network segment",
                })
        except Exception:
            pass

    if not findings:
        findings.append({
            "requirement": "1.3", "check": "Network segmentation",
            "status": "PASS", "severity": "INFO",
            "detail": f"No tested ports reachable on {target_ip}",
        })

    return findings


def format_summary(all_findings):
    """Print PCI DSS audit summary."""
    print(f"\n{'='*60}")
    print(f"  PCI DSS v4.0 Compliance Audit Report")
    print(f"{'='*60}")

    pass_count = sum(1 for f in all_findings if f["status"] == "PASS")
    fail_count = sum(1 for f in all_findings if f["status"] == "FAIL")
    warn_count = sum(1 for f in all_findings if f["status"] == "WARN")

    print(f"  Total Checks : {len(all_findings)}")
    print(f"  Passed       : {pass_count}")
    print(f"  Failed       : {fail_count}")
    print(f"  Warnings     : {warn_count}")

    by_req = {}
    for f in all_findings:
        req = f.get("requirement", "unknown")
        by_req.setdefault(req, []).append(f)

    print(f"\n  Results by Requirement:")
    for req in sorted(by_req.keys()):
        items = by_req[req]
        failed = sum(1 for i in items if i["status"] == "FAIL")
        passed = sum(1 for i in items if i["status"] == "PASS")
        status = "FAIL" if failed > 0 else "PASS"
        print(f"    Req {req:8s}: [{status:4s}] {passed} passed, {failed} failed")

    if fail_count > 0:
        print(f"\n  Failed Checks:")
        for f in all_findings:
            if f["status"] == "FAIL":
                print(f"    [{f['severity']:8s}] Req {f['requirement']}: {f['check']} - {f.get('detail', '')}")

    severity_counts = {}
    for f in all_findings:
        if f["status"] == "FAIL":
            sev = f.get("severity", "MEDIUM")
            severity_counts[sev] = severity_counts.get(sev, 0) + 1
    return severity_counts


def main():
    parser = argparse.ArgumentParser(description="PCI DSS compliance control audit agent")
    parser.add_argument("--tls-host", help="Host to check TLS configuration")
    parser.add_argument("--tls-port", type=int, default=443)
    parser.add_argument("--segment-target", help="IP to check network segmentation")
    parser.add_argument("--skip-password", action="store_true")
    parser.add_argument("--skip-logging", action="store_true")
    parser.add_argument("--skip-fim", action="store_true")
    parser.add_argument("--output", "-o", help="Output JSON report")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    all_findings = []

    if args.tls_host:
        all_findings.extend(check_tls_configuration(args.tls_host, args.tls_port))
    if not args.skip_password:
        all_findings.extend(check_password_policy())
    if not args.skip_logging:
        all_findings.extend(check_audit_logging())
    if not args.skip_fim:
        all_findings.extend(check_file_integrity())
    all_findings.extend(check_default_credentials())
    if args.segment_target:
        all_findings.extend(check_network_segmentation(args.segment_target))

    if not all_findings:
        print("[!] No checks performed. Use --tls-host or other options.", file=sys.stderr)
        sys.exit(1)

    severity_counts = format_summary(all_findings)

    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "PCI DSS Audit",
        "standard": "PCI DSS v4.0",
        "findings": all_findings,
        "severity_counts": severity_counts,
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


===== FILE: skills/managing-third-party-vendor-risk/SKILL.md =====
---
name: managing-third-party-vendor-risk
description: >-
  Build and run a third-party/vendor risk management (TPRM) program aligned to NIST SP
  800-161 C-SCRM: inventory and tier vendors, issue SIG/CAIQ questionnaires, review
  SOC 2/ISO 27001 evidence, set contractual right-to-audit clauses, monitor vendors
  continuously, and offboard securely. Use when assessing a new vendor, standing up a
  vendor-risk program, tiering a portfolio, reviewing a SOC 2/CAIQ, or writing security
  terms into a contract.
domain: cybersecurity
subdomain: compliance-governance
tags:
- third-party-risk
- vendor-risk-management
- tprm
- supply-chain-risk
- c-scrm
- nist-800-161
- soc2
- caiq
- continuous-monitoring
- governance
version: "1.0"
author: andrewibrah
license: Apache-2.0
nist_csf:
- GV.SC-01
- GV.SC-04
- GV.SC-06
- GV.SC-07
- ID.RA-05
- GV.OC-03
mitre_attack:
- T1199
- T1195
- T1078
- T1190
- T1567
---

# Managing Third-Party Vendor Risk

## When to Use

- When assessing a **new vendor** before onboarding, especially one that will handle sensitive data, connect to your network, or be embedded in a critical process.
- When **standing up or maturing** a third-party risk management (TPRM) program and you need a repeatable tiering + assessment workflow.
- When **tiering an existing vendor portfolio** so effort matches risk.
- When **reviewing vendor evidence** — a SOC 2 Type II report, ISO 27001 certificate, CAIQ, or pen-test summary — and you need to know what to look for.
- When writing **security and privacy requirements into a contract / DPA**, including breach-notification SLAs and right-to-audit.
- When a vendor (or their subcontractor) suffers a **breach** and you must assess exposure.
- When managing **software supply-chain** and **Nth-party** (fourth-party and beyond) risk.

## Prerequisites

- A **vendor inventory** (who you use, for what, and what data/access each has).
- A defined **risk-tiering model** (criteria and thresholds) agreed with the business.
- Access to standardized **questionnaires** (Shared Assessments **SIG**, CSA **CAIQ**) and a way to collect evidence.
- Clarity on your own **regulatory obligations** that flow down to vendors (e.g., HIPAA BAAs, CMMC flowdown, GDPR processor terms, PCI).
- Stakeholders identified: procurement, legal, security, data owner, and the business sponsor.

## Workflow

### 1. Inventory and classify vendors
Catalog every third party and capture: data sensitivity handled, type of access (network, physical, none), business criticality, and regulatory scope. You cannot manage what you have not inventoried — shadow vendors are a common blind spot.

### 2. Tier by inherent risk
Score each vendor on inherent-risk factors (data sensitivity, access, criticality, regulatory scope, spend/concentration) and assign a **tier** (e.g., Critical / High / Moderate / Low). The tier drives **how deep** the assessment goes and **how often** you reassess. A payroll processor with PII and system access is not the same risk as a stock-photo subscription.

### 3. Run tier-appropriate due diligence
- **Critical/High:** full **SIG** (or SIG Core), request **SOC 2 Type II** and/or **ISO 27001**, recent **pen-test** summary, and evidence of an incident-response capability. Consider an assessor call.
- **Moderate:** **SIG Lite** or **CAIQ**, plus key attestations.
- **Low:** lightweight questionnaire / self-attestation.

### 4. Review evidence critically
Don't just collect — **read**:
- **SOC 2 Type II:** check scope, the Trust Services Criteria covered, the audit **period** (not just the date), and especially the **exceptions/deviations** and any qualified opinion. A clean cover page can hide noted exceptions.
- **ISO 27001:** confirm the **scope statement** and the Statement of Applicability actually cover the service you're buying.
- **CAIQ:** look for "no" answers and CCM domains left blank.
- **Pen-test:** age, scope, and whether highs/criticals were remediated.

### 5. Identify gaps and decide
Compare findings against your control requirements. For each gap: accept, require remediation (with a date), add a compensating control on your side, or walk away. Record the **residual risk** and a risk-owner decision.

### 6. Codify in the contract / DPA
Bake requirements into the agreement: security control obligations, **breach-notification timeline**, data-handling and return/destruction terms, **right-to-audit / right to assessment evidence**, subcontractor (Nth-party) flowdown, and liability/insurance. Contracts are where TPRM gets teeth.

### 7. Monitor continuously
Tiering is not a one-time gate. For higher tiers: periodic reassessment, **security-ratings** feeds, breach/news monitoring, certificate-expiry tracking, and watching for material changes (acquisition, region change, new subprocessors). Re-tier on change.

### 8. Manage Nth-party and concentration risk
Map critical **fourth parties** (your vendor's key subprocessors) and watch for **concentration** (many vendors riding on the same upstream provider) — a single upstream outage or breach can hit your whole portfolio at once.

### 9. Offboard securely
On termination: revoke access and credentials, confirm **data return or certified destruction**, remove integrations/API keys, and update the inventory. Un-offboarded vendors are standing risk.

## Key Concepts

| Concept | Definition |
|---|---|
| Inherent risk | Risk a vendor poses before controls — drives tiering. |
| Residual risk | Risk remaining after the vendor's (and your) controls. |
| Vendor tier | Risk band (Critical/High/Moderate/Low) setting assessment depth and cadence. |
| SIG | Shared Assessments Standardized Information Gathering questionnaire (full / Lite / Core). |
| CAIQ | CSA Consensus Assessments Initiative Questionnaire (maps to the Cloud Controls Matrix). |
| SOC 2 Type II | Attestation on control design **and** operating effectiveness over a period. |
| Right to audit | Contractual right to assess the vendor or obtain assessment evidence. |
| Nth-party / fourth-party | Your vendor's vendors (and beyond) — indirect supply-chain risk. |
| Concentration risk | Many vendors depending on the same upstream provider. |
| C-SCRM | Cybersecurity Supply Chain Risk Management (NIST SP 800-161). |

## Tools & Systems

- **NIST SP 800-161 Rev 1** — Cybersecurity Supply Chain Risk Management practices.
- **NIST CSF 2.0 — GV.SC** — the supply-chain risk-management category (program backbone).
- **Shared Assessments SIG** and **CSA CAIQ / STAR registry** — standardized questionnaires.
- **SOC 2 / ISO 27001 / PCI AOC / pen-test reports** — vendor evidence.
- **Security-ratings services** (e.g., BitSight/SecurityScorecard-style) — continuous external signal.
- **TPRM platforms** — OneTrust, ProcessUnity, Prevalent, ServiceNow VRM, etc., to manage the workflow and inventory.
- **GDPR DPA / HIPAA BAA / CMMC flowdown** — regulatory contract instruments.

## Common Scenarios

- **New SaaS onboarding.** Tier it, send the right questionnaire, read the SOC 2 exceptions, set contract terms, then approve with documented residual risk.
- **Portfolio has 400 vendors, no tiers.** Tier first; concentrate assessment effort on the Critical/High tail rather than spreading thin.
- **Vendor breach in the news.** Pull the vendor record, assess data/access exposure, invoke the breach-notification clause, and require a post-incident report.
- **Auditor asks for your TPRM program.** Show the tiering model, the assessment cadence, and evidence of continuous monitoring mapped to GV.SC.
- **Critical fourth party identified.** Document the dependency and the concentration risk; build a contingency for that upstream provider.

## Output Format

Produce a **Vendor Risk Assessment** using `assets/template.md`, containing:

1. **Vendor profile** — service, data handled, access type, business criticality, regulatory scope.
2. **Inherent-risk tier** — score and resulting tier, with rationale.
3. **Due-diligence performed** — questionnaire used and evidence collected (SOC 2 period, ISO scope, pen-test age).
4. **Findings** — gaps with severity, including notable SOC 2 exceptions.
5. **Decision & residual risk** — approve/conditional/reject, with risk-owner sign-off.
6. **Contractual requirements** — security terms, breach SLA, right-to-audit, subprocessor flowdown.
7. **Monitoring & reassessment plan** — cadence, signals watched, re-tier triggers.
8. **Nth-party notes** — critical subprocessors and concentration risk.

Use `scripts/process.py` to compute a vendor's inherent-risk tier from a profile JSON, set the assessment depth and reassessment cadence, and flag missing evidence for the assigned tier.

===== END FILE =====


===== FILE: skills/managing-third-party-vendor-risk/LICENSE =====

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


===== FILE: skills/managing-third-party-vendor-risk/assets/template.md =====
# Vendor Risk Assessment — Worked Example

> Filled example for a payroll-processing vendor (regulated PII, deep integration).
> Replace bracketed content for your own vendor.

## 1. Vendor Profile
- **Vendor:** PayWorks
- **Service:** Payroll processing (SaaS)
- **Data handled:** Employee PII, bank details (regulated).
- **Access type:** System access (API + SSO into HRIS).
- **Business criticality:** High — a multi-day outage would block payroll.
- **Regulatory scope:** PII / state payroll requirements.

## 2. Inherent-Risk Tier
*(scored by `scripts/process.py`)*

| Factor | Value | Points |
|---|---|---|
| Data sensitivity | regulated | 4 |
| Access | system | 4 |
| Criticality | high | 4 |
| Integration | deep | 2 |
| Regulated scope | PII | 2 |
| Concentration | single payroll source | 1 |
| **Total** | | **17 → Tier: Critical** |

**Rationale:** regulated data + system access + high criticality place this in the top tier; assess deeply and monitor continuously.

## 3. Due Diligence Performed
- **Questionnaire:** Full SIG requested.
- **SOC 2:** Type II, **12-month** period obtained.
- **ISO 27001:** Certificate obtained — scope statement confirmed to cover the payroll service.
- **Pen-test:** Summary from 5 months ago; highs/criticals remediated.

## 4. Findings
| Finding | Severity | Note |
|---|---|---|
| SOC 2 exception: one quarter of incomplete access reviews | Moderate | Vendor provided remediation evidence; accept with monitoring |
| No customer-managed encryption keys | Low | Within risk tolerance for this data set |
| Two critical fourth parties (cloud + email) | Info | Concentration noted (see §8) |

> The SOC 2 cover page was clean — the exception was found in the body. Always read the deviations and CUECs.

## 5. Decision & Residual Risk
- **Decision:** **Approve — Conditional.**
- **Condition:** Vendor confirms completion of the access-review remediation within 60 days.
- **Residual risk:** **Moderate, accepted** by [data owner / risk owner], [date].

## 6. Contractual Requirements
- Security obligations mapped to our baseline (encryption, access control, logging).
- **Breach notification within 48 hours** of discovery.
- Data return / **certified destruction** within 30 days of termination.
- **Right to audit** or to receive a current SOC 2 annually.
- **Subprocessor flowdown** + prior notice of new subprocessors.
- Cyber-insurance minimum and liability terms.
- **DPA** executed (PII processing).

## 7. Monitoring & Reassessment Plan
- **Cadence:** Full reassessment **annually** (Critical tier).
- **Continuous signals:** security-ratings feed, breach/news monitoring, SOC 2 / ISO expiry tracking.
- **Re-tier triggers:** ownership change, new region/subprocessor, material breach, scope expansion.

## 8. Nth-Party / Concentration Notes
- **Critical fourth parties:** cloud IaaS provider and transactional email provider (from the SOC 2 subservice list).
- **Concentration risk:** our HRIS and PayWorks both ride the same cloud region — a single regional outage hits payroll and HR together. Contingency: documented manual-payroll fallback for one cycle.

===== END FILE =====


===== FILE: skills/managing-third-party-vendor-risk/references/standards.md =====
# Third-Party / Vendor Risk Management — Standards & Reference

## Primary standards & frameworks
| Source | Role |
|---|---|
| **NIST SP 800-161 Rev 1** (May 2022) | Cybersecurity Supply Chain Risk Management (C-SCRM) practices for systems and organizations. URL: https://csrc.nist.gov/pubs/sp/800/161/r1/final |
| **NIST CSF 2.0 — GV.SC** | The Cybersecurity Supply Chain Risk Management category; the governance backbone for a TPRM program. |
| **NIST SP 800-37 / 800-53 (SR family)** | Supply Chain Risk Management controls (SR-x) within the broader control catalog. |
| **ISO/IEC 27036** | Information security for supplier relationships. |
| **Shared Assessments** | SIG questionnaire + Third Party Risk Management framework. |
| **CSA CAIQ / Cloud Controls Matrix (CCM) / STAR** | Cloud-vendor self-assessment and registry. |

## NIST CSF 2.0 — GV.SC subcategories (selected)
| ID | Outcome |
|---|---|
| GV.SC-01 | A cyber supply-chain risk-management program/strategy is established and agreed. |
| GV.SC-03 | Supply-chain risk management is integrated into cybersecurity and ERM. |
| GV.SC-04 | Suppliers are known and prioritized by criticality. |
| GV.SC-05 | Requirements to address supply-chain risk are established in contracts. |
| GV.SC-06 | Due diligence is performed to reduce risk before entering relationships. |
| GV.SC-07 | Supplier risks are understood, monitored, and managed over the relationship. |
| GV.SC-08 | Suppliers are included in incident planning, response, and recovery. |
| GV.SC-10 | Supply-chain risk is managed through to relationship termination. |

## Vendor tiering — typical inherent-risk factors
- **Data sensitivity** handled (regulated PII/PHI/CHD, IP, none).
- **Access type** (network/system access, physical access, none).
- **Business criticality** (would an outage stop operations?).
- **Regulatory scope** (HIPAA, PCI, GDPR, CMMC flowdown).
- **Integration depth** (API/identity federation vs standalone).
- **Concentration / spend** (single-source, large dependency).

Tiers commonly: **Critical / High / Moderate / Low** — each mapped to an assessment depth and a reassessment cadence.

## Due-diligence instruments
| Instrument | What it is |
|---|---|
| SIG (Full / Core / Lite) | Shared Assessments standardized questionnaire; depth scales with tier. |
| CAIQ | CSA questionnaire mapped to the Cloud Controls Matrix. |
| SOC 2 Type II | AICPA attestation on control **design and operating effectiveness over a period** (Trust Services Criteria: Security required; Availability, Confidentiality, Processing Integrity, Privacy optional). |
| SOC 2 Type I | Design only, at a point in time (weaker assurance than Type II). |
| ISO/IEC 27001 certificate + SoA | Certified ISMS; check the **scope statement** covers the purchased service. |
| Penetration-test summary | Independent testing; check age, scope, and remediation of highs/criticals. |
| PCI AOC | Attestation of Compliance for card-data handlers. |

## Reading a SOC 2 critically
- Confirm the **report type** (II > I) and the **audit period** length.
- Check the **scope / system description** matches the service you buy.
- Read the **exceptions / deviations** and the auditor's opinion (unqualified vs qualified).
- Review **complementary user-entity controls (CUECs)** — what the vendor expects **you** to do.
- Note the **subservice organizations** (their critical fourth parties).

## Contractual security terms to require
- Security control obligations (map to your baseline).
- **Breach-notification timeline** (e.g., notify within X hours of discovery).
- Data handling, location, and **return/certified destruction** on exit.
- **Right to audit** or to receive current assessment evidence.
- **Subcontractor (Nth-party) flowdown** and prior-approval of new subprocessors.
- Liability, indemnity, and cyber-insurance requirements.
- Regulatory instruments: **DPA** (GDPR), **BAA** (HIPAA), CMMC flowdown.

## Continuous monitoring signals
Security-ratings feeds, breach/news monitoring, certificate/attestation expiry, new subprocessor notices, ownership/region changes, and periodic re-questionnaire on cadence by tier.

## Nth-party & concentration risk
- **Fourth-party** = your vendor's vendors; map the critical ones.
- **Concentration risk** = many vendors depending on the same upstream (e.g., one cloud region or one auth provider) — a single upstream failure can be systemic.

===== END FILE =====


===== FILE: skills/managing-third-party-vendor-risk/scripts/process.py =====
#!/usr/bin/env python3
"""
Third-party vendor inherent-risk tiering and evidence-gap checker.

Scores a vendor's inherent risk from a profile, assigns a tier
(Critical/High/Moderate/Low), sets the assessment depth and reassessment
cadence for that tier, and flags evidence that is missing or stale for the
assigned tier.

Input JSON shape:
{
  "vendor": {"name": "PayWorks", "service": "Payroll processing"},
  "profile": {
    "data_sensitivity": "regulated",   # regulated | confidential | internal | public
    "access": "system",                # system | network | physical | none
    "criticality": "high",             # high | medium | low
    "regulated_scope": ["PII"],        # list; non-empty raises risk
    "integration": "deep",             # deep | moderate | none
    "concentration": true              # single-source / large dependency
  },
  "evidence": {                        # OPTIONAL; what you have on file
    "sig": "core",                     # full | core | lite | caiq | none
    "soc2_type": "II",                 # II | I | none
    "soc2_period_months": 12,
    "iso27001": true,
    "pentest_age_months": 8
  }
}

Usage:
  python process.py --input vendor.json [--output assessment.md]
  python process.py --input vendor.json --fail-on-evidence-gap
"""

import argparse
import json
import sys

# inherent-risk points per factor
DATA_POINTS = {"regulated": 4, "confidential": 3, "internal": 1, "public": 0}
ACCESS_POINTS = {"system": 4, "network": 3, "physical": 2, "none": 0}
CRIT_POINTS = {"high": 4, "medium": 2, "low": 1}
INTEG_POINTS = {"deep": 2, "moderate": 1, "none": 0}

# tier thresholds on total inherent score (max ~17)
def tier_for(score):
    if score >= 13:
        return "Critical"
    if score >= 9:
        return "High"
    if score >= 5:
        return "Moderate"
    return "Low"

# per-tier expectations
TIER_PLAYBOOK = {
    "Critical": {
        "depth": "Full SIG + SOC 2 Type II (12-month period) + ISO 27001 + recent pen-test + assessor call",
        "cadence": "Reassess annually; continuous security-ratings monitoring",
        "require": {"sig": ("full", "core"), "soc2_type": ("II",), "iso27001": True, "pentest_max_months": 12},
    },
    "High": {
        "depth": "SIG Core + SOC 2 Type II + pen-test summary",
        "cadence": "Reassess annually",
        "require": {"sig": ("full", "core"), "soc2_type": ("II",), "iso27001": False, "pentest_max_months": 18},
    },
    "Moderate": {
        "depth": "SIG Lite or CAIQ + key attestations",
        "cadence": "Reassess every 2 years",
        "require": {"sig": ("full", "core", "lite", "caiq"), "soc2_type": ("II", "I"), "iso27001": False, "pentest_max_months": None},
    },
    "Low": {
        "depth": "Lightweight questionnaire / self-attestation",
        "cadence": "Reassess every 3 years or on change",
        "require": {"sig": ("full", "core", "lite", "caiq", "none"), "soc2_type": ("II", "I", "none"), "iso27001": False, "pentest_max_months": None},
    },
}


def score_inherent(p):
    breakdown = {}
    breakdown["data_sensitivity"] = DATA_POINTS.get(p.get("data_sensitivity", "internal"), 1)
    breakdown["access"] = ACCESS_POINTS.get(p.get("access", "none"), 0)
    breakdown["criticality"] = CRIT_POINTS.get(p.get("criticality", "low"), 1)
    breakdown["integration"] = INTEG_POINTS.get(p.get("integration", "none"), 0)
    breakdown["regulated_scope"] = 2 if p.get("regulated_scope") else 0
    breakdown["concentration"] = 1 if p.get("concentration") else 0
    total = sum(breakdown.values())
    return total, breakdown


def check_evidence(tier, ev):
    req = TIER_PLAYBOOK[tier]["require"]
    gaps = []
    sig = (ev.get("sig") or "none").lower()
    if sig not in req["sig"]:
        gaps.append(f"Questionnaire '{sig}' insufficient for {tier} (need one of {', '.join(req['sig'])})")
    soc = (ev.get("soc2_type") or "none")
    if soc not in req["soc2_type"]:
        gaps.append(f"SOC 2 type '{soc}' insufficient for {tier} (need {', '.join(req['soc2_type'])})")
    if soc == "II" and ev.get("soc2_period_months", 0) < 6:
        gaps.append("SOC 2 Type II period under 6 months - limited operating-effectiveness assurance")
    if req["iso27001"] and not ev.get("iso27001"):
        gaps.append(f"ISO 27001 certificate expected for {tier}")
    pmax = req["pentest_max_months"]
    if pmax is not None:
        age = ev.get("pentest_age_months")
        if age is None:
            gaps.append(f"No pen-test on file (expected within {pmax} months for {tier})")
        elif age > pmax:
            gaps.append(f"Pen-test is {age} months old (>{pmax} for {tier}) - request a current test")
    return gaps


def render(data):
    vendor = data.get("vendor", {})
    profile = data.get("profile", {})
    evidence = data.get("evidence", {})
    if not profile:
        raise ValueError("profile is required to tier the vendor")

    total, breakdown = score_inherent(profile)
    tier = tier_for(total)
    play = TIER_PLAYBOOK[tier]
    gaps = check_evidence(tier, evidence) if evidence else ["No evidence provided - collect tier-appropriate evidence"]

    lines = []
    lines.append(f"# Vendor Risk Assessment - {vendor.get('name','Vendor')}")
    lines.append("")
    if vendor.get("service"):
        lines.append(f"- **Service:** {vendor['service']}")
    lines.append(f"- **Inherent-risk score:** {total} -> **Tier: {tier}**")
    lines.append("")

    lines.append("## Inherent-risk breakdown")
    lines.append("")
    lines.append("| Factor | Value | Points |")
    lines.append("|---|---|---|")
    lines.append(f"| Data sensitivity | {profile.get('data_sensitivity','-')} | {breakdown['data_sensitivity']} |")
    lines.append(f"| Access | {profile.get('access','-')} | {breakdown['access']} |")
    lines.append(f"| Criticality | {profile.get('criticality','-')} | {breakdown['criticality']} |")
    lines.append(f"| Integration | {profile.get('integration','-')} | {breakdown['integration']} |")
    lines.append(f"| Regulated scope | {', '.join(profile.get('regulated_scope', [])) or 'none'} | {breakdown['regulated_scope']} |")
    lines.append(f"| Concentration | {profile.get('concentration', False)} | {breakdown['concentration']} |")
    lines.append(f"| **Total** | | **{total}** |")
    lines.append("")

    lines.append(f"## {tier}-tier playbook")
    lines.append("")
    lines.append(f"- **Assessment depth:** {play['depth']}")
    lines.append(f"- **Reassessment cadence:** {play['cadence']}")
    lines.append("")

    lines.append("## Evidence gaps")
    lines.append("")
    if not gaps:
        lines.append(f"Evidence on file meets the {tier}-tier bar. Proceed to findings review and contracting.")
    else:
        for g in gaps:
            lines.append(f"- {g}")
    lines.append("")

    lines.append("## Next steps")
    lines.append("")
    lines.append("1. Close the evidence gaps above (or document risk-accepted exceptions).")
    lines.append("2. Review collected evidence critically (SOC 2 exceptions, ISO scope, CAIQ 'no' answers).")
    lines.append("3. Record findings, residual risk, and a risk-owner decision.")
    lines.append("4. Codify security terms, breach SLA, right-to-audit, and subprocessor flowdown in the contract.")
    lines.append("5. Enroll in continuous monitoring per the cadence above.")

    return "\n".join(lines), tier, gaps


def main():
    ap = argparse.ArgumentParser(description="Vendor inherent-risk tiering + evidence-gap checker")
    ap.add_argument("--input", "-i", required=True, help="Path to vendor profile JSON")
    ap.add_argument("--output", "-o", help="Write Markdown assessment to this path")
    ap.add_argument("--fail-on-evidence-gap", action="store_true",
                    help="Exit non-zero if any evidence gap remains for the tier")
    args = ap.parse_args()

    try:
        with open(args.input) as f:
            data = json.load(f)
    except (OSError, json.JSONDecodeError) as e:
        print(f"ERROR: could not read input JSON: {e}", file=sys.stderr)
        return 2

    try:
        md, tier, gaps = render(data)
    except ValueError as e:
        print(f"ERROR: {e}", file=sys.stderr)
        return 2

    if args.output:
        with open(args.output, "w") as f:
            f.write(md + "\n")
        print(f"Assessment written to {args.output}", file=sys.stderr)
    else:
        print(md)

    print(f"Tier: {tier}; evidence gaps: {len(gaps)}.", file=sys.stderr)

    if args.fail_on_evidence_gap and gaps:
        print(f"FAIL: {len(gaps)} evidence gap(s) for {tier} tier.", file=sys.stderr)
        return 1
    return 0


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/performing-nist-csf-maturity-assessment/SKILL.md =====
---
name: performing-nist-csf-maturity-assessment
description: Conduct a NIST Cybersecurity Framework (CSF) 2.0 maturity assessment across the six core Functions (Govern, Identify, Protect, Detect, Respond, Recover), scoring organizational posture against the four Implementation Tiers (Partial, Risk-Informed, Repeatable, Adaptive) and producing an improvement roadmap. Use when benchmarking an organization's cybersecurity program maturity or preparing a CSF-based gap analysis and remediation plan.
domain: cybersecurity
subdomain: compliance-governance
tags:
- compliance
- governance
- nist
- csf
- maturity-assessment
- risk-management
nist_csf:
- GV.OC-01
- GV.RM-01
- GV.PO-01
- ID.RA-01
- GV.OV-01
version: '1.0'
author: mahipal
license: Apache-2.0
mitre_attack:
- T1078
- T1530
- T1685.002
---
# Performing NIST CSF Maturity Assessment

## Overview
The NIST Cybersecurity Framework (CSF) 2.0, released in February 2024, provides a comprehensive taxonomy for managing cybersecurity risk through six core Functions: Govern, Identify, Protect, Detect, Respond, and Recover. This skill covers conducting a maturity assessment against the CSF, using the four Implementation Tiers (Partial, Risk-Informed, Repeatable, Adaptive) to measure organizational cybersecurity posture and create improvement roadmaps.


## When to Use

- When conducting security assessments that involve performing nist csf maturity assessment
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites
- Understanding of cybersecurity risk management principles
- Access to NIST CSF 2.0 documentation and reference tool
- Knowledge of organizational IT/OT environment and security controls
- Stakeholder access across business units for assessment interviews

## Core Concepts

### CSF 2.0 Functions (6 Functions, 22 Categories)

| Function | Code | Categories | Purpose |
|----------|------|-----------|---------|
| **Govern** | GV | 6 | Establish and monitor cybersecurity risk management strategy |
| **Identify** | ID | 3 | Determine current cybersecurity risk to the organization |
| **Protect** | PR | 5 | Implement safeguards to prevent or reduce risk |
| **Detect** | DE | 2 | Find and analyze possible cybersecurity attacks |
| **Respond** | RS | 4 | Take action regarding detected cybersecurity incidents |
| **Recover** | RC | 2 | Restore capabilities impaired by cybersecurity incidents |

### Govern Function (New in CSF 2.0)
- GV.OC: Organizational Context
- GV.RM: Risk Management Strategy
- GV.RR: Roles, Responsibilities, and Authorities
- GV.PO: Policy
- GV.OV: Oversight
- GV.SC: Cybersecurity Supply Chain Risk Management

### Implementation Tiers
| Tier | Name | Description |
|------|------|-------------|
| Tier 1 | Partial | Ad hoc, reactive; limited awareness of cybersecurity risk |
| Tier 2 | Risk-Informed | Risk-aware but not organization-wide; approved but may not be policy |
| Tier 3 | Repeatable | Formal policies; consistently implemented; regularly updated |
| Tier 4 | Adaptive | Continuous improvement; real-time risk response; lessons learned integrated |

## Workflow

### Phase 1: Scoping and Preparation (Weeks 1-2)
1. Define assessment scope (enterprise-wide vs. business unit)
2. Identify stakeholders and schedule interviews
3. Gather existing documentation (policies, procedures, architecture diagrams)
4. Customize CSF Profile for organizational context
5. Select assessment methodology (self-assessment, facilitated, third-party)

### Phase 2: Current State Assessment (Weeks 3-6)
1. Assess each CSF Category and Subcategory against Implementation Tiers
2. For each subcategory, evaluate:
   - Policy/documentation maturity
   - Implementation completeness
   - Automation level
   - Measurement and metrics
   - Continuous improvement evidence
3. Score using tier criteria (1-4 scale)
4. Document evidence supporting each tier rating
5. Identify strengths, gaps, and improvement areas

### Phase 3: Target State Definition (Weeks 7-8)
1. Define target tier for each Function based on:
   - Risk appetite and tolerance
   - Industry requirements and benchmarks
   - Regulatory obligations
   - Available resources and budget
2. Create Target Profile documenting desired maturity state
3. Validate target state with executive leadership

### Phase 4: Gap Analysis and Roadmap (Weeks 9-12)
1. Compare Current Profile to Target Profile
2. Prioritize gaps based on risk reduction potential
3. Develop improvement roadmap with:
   - Short-term quick wins (0-3 months)
   - Medium-term improvements (3-12 months)
   - Long-term strategic initiatives (12-24 months)
4. Estimate resource requirements for each initiative
5. Assign ownership and timelines

### Phase 5: Implementation and Reassessment (Ongoing)
1. Execute improvement roadmap initiatives
2. Track progress against milestones
3. Conduct periodic reassessments (annually recommended)
4. Report maturity progress to leadership
5. Adjust roadmap based on evolving threats and business changes

## Key Artifacts
- CSF Current Profile (by Function/Category/Subcategory)
- CSF Target Profile
- Gap Analysis Report
- Maturity Assessment Scorecard
- Improvement Roadmap with Priorities
- Executive Summary and Dashboard

## Common Pitfalls
- Assessing technology only without evaluating governance and people
- Setting unrealistic target tiers without resource commitment
- Treating assessment as one-time rather than continuous process
- Ignoring the new Govern function in CSF 2.0
- Not aligning CSF assessment with existing compliance requirements (ISO 27001, SOC 2)

## References
- NIST CSF 2.0: https://csf.tools/reference/nist-cybersecurity-framework/v2-0/
- NIST SP 800-53 Rev 5 (control catalog that maps to CSF)
- NIST CSF 2.0 Quick Start Guides
- CSF 2.0 Reference Tool: https://csrc.nist.gov/projects/cybersecurity-framework

===== END FILE =====


===== FILE: skills/performing-nist-csf-maturity-assessment/LICENSE =====

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


===== FILE: skills/performing-nist-csf-maturity-assessment/assets/template.md =====
# NIST CSF 2.0 Maturity Assessment Scorecard

## Organization: ________________
## Assessment Date: ________________
## Assessor: ________________

---

## Scoring Guide
| Tier | Name | Criteria |
|------|------|----------|
| 1 | Partial | Ad hoc; limited awareness; no formal process |
| 2 | Risk-Informed | Management-approved; inconsistent application |
| 3 | Repeatable | Formal policies; consistent implementation; metrics |
| 4 | Adaptive | Continuous improvement; real-time; advanced automation |

---

## Function: GOVERN (GV)

| Category | Current (1-4) | Target (1-4) | Gap | Evidence | Notes |
|----------|:---:|:---:|:---:|----------|-------|
| GV.OC - Organizational Context | | | | | |
| GV.RM - Risk Management Strategy | | | | | |
| GV.RR - Roles and Responsibilities | | | | | |
| GV.PO - Policy | | | | | |
| GV.OV - Oversight | | | | | |
| GV.SC - Supply Chain Risk Mgmt | | | | | |
| **Function Average** | | | | | |

## Function: IDENTIFY (ID)

| Category | Current (1-4) | Target (1-4) | Gap | Evidence | Notes |
|----------|:---:|:---:|:---:|----------|-------|
| ID.AM - Asset Management | | | | | |
| ID.RA - Risk Assessment | | | | | |
| ID.IM - Improvement | | | | | |
| **Function Average** | | | | | |

## Function: PROTECT (PR)

| Category | Current (1-4) | Target (1-4) | Gap | Evidence | Notes |
|----------|:---:|:---:|:---:|----------|-------|
| PR.AA - Identity & Access Control | | | | | |
| PR.AT - Awareness and Training | | | | | |
| PR.DS - Data Security | | | | | |
| PR.PS - Platform Security | | | | | |
| PR.IR - Infrastructure Resilience | | | | | |
| **Function Average** | | | | | |

## Function: DETECT (DE)

| Category | Current (1-4) | Target (1-4) | Gap | Evidence | Notes |
|----------|:---:|:---:|:---:|----------|-------|
| DE.CM - Continuous Monitoring | | | | | |
| DE.AE - Adverse Event Analysis | | | | | |
| **Function Average** | | | | | |

## Function: RESPOND (RS)

| Category | Current (1-4) | Target (1-4) | Gap | Evidence | Notes |
|----------|:---:|:---:|:---:|----------|-------|
| RS.MA - Incident Management | | | | | |
| RS.AN - Incident Analysis | | | | | |
| RS.CO - Response Communication | | | | | |
| RS.MI - Incident Mitigation | | | | | |
| **Function Average** | | | | | |

## Function: RECOVER (RC)

| Category | Current (1-4) | Target (1-4) | Gap | Evidence | Notes |
|----------|:---:|:---:|:---:|----------|-------|
| RC.RP - Recovery Plan Execution | | | | | |
| **Function Average** | | | | | |

---

## Overall Summary

| Function | Current Avg | Target Avg | Gap |
|----------|:---:|:---:|:---:|
| Govern | | | |
| Identify | | | |
| Protect | | | |
| Detect | | | |
| Respond | | | |
| Recover | | | |
| **Overall** | | | |

## Top Priority Gaps

| # | Category | Current | Target | Recommended Action | Owner | Timeline |
|---|----------|:---:|:---:|-----------|-------|----------|
| 1 | | | | | | |
| 2 | | | | | | |
| 3 | | | | | | |
| 4 | | | | | | |
| 5 | | | | | | |

## Sign-off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| CISO | | | |
| CTO/CIO | | | |
| Risk Officer | | | |

===== END FILE =====


===== FILE: skills/performing-nist-csf-maturity-assessment/references/api-reference.md =====
# API Reference — Performing NIST CSF Maturity Assessment

## Libraries Used
- **csv**: Parse and generate assessment CSV files
- **pathlib**: File operations

## CLI Interface
```
python agent.py assess --csv assessment_responses.csv
python agent.py gaps --csv assessment_responses.csv
python agent.py template [--output template.csv]
python agent.py executive --csv assessment_responses.csv
```

## Core Functions

### `assess_from_csv(assessment_file)` — Calculate maturity scores
Scores each NIST CSF function (Identify, Protect, Detect, Respond, Recover).
Calculates overall maturity level (1-4 scale) and gap-to-target.

### `generate_gap_analysis(assessment_file)` — Prioritized gap report
Classifies gaps: HIGH (>=2 gap), MEDIUM (>=1), LOW (<1).

### `create_assessment_template(output_file)` — Generate blank assessment CSV
Produces CSV with all 23 CSF categories, score/target/evidence columns.

### `generate_executive_summary(assessment_file)` — Board-level report

## NIST CSF Functions & Categories (23 total)
| Function | Categories |
|----------|-----------|
| IDENTIFY | ID.AM, ID.BE, ID.GV, ID.RA, ID.RM, ID.SC |
| PROTECT | PR.AC, PR.AT, PR.DS, PR.IP, PR.MA, PR.PT |
| DETECT | DE.AE, DE.CM, DE.DP |
| RESPOND | RS.RP, RS.CO, RS.AN, RS.MI, RS.IM |
| RECOVER | RC.RP, RC.IM, RC.CO |

## Maturity Levels
| Level | Name | Description |
|-------|------|-------------|
| 1 | Partial | Not formalized |
| 2 | Risk Informed | Approved but not org-wide |
| 3 | Repeatable | Formally expressed as policy |
| 4 | Adaptive | Continuous improvement |

## Dependencies
No external packages — Python standard library only.

===== END FILE =====


===== FILE: skills/performing-nist-csf-maturity-assessment/references/standards.md =====
# NIST CSF 2.0 Standards Reference

## Primary Standard
### NIST Cybersecurity Framework (CSF) Version 2.0
- **Published**: February 26, 2024
- **Publisher**: National Institute of Standards and Technology (NIST)
- **Document**: NIST CSWP 29
- **Scope**: Applicable to all organizations regardless of size, sector, or maturity
- **Key Change**: Added Govern function as central pillar; expanded from 5 to 6 functions

## CSF 2.0 Core Structure

### GV - Govern
- GV.OC-01 to GV.OC-05: Organizational context understood
- GV.RM-01 to GV.RM-07: Risk management strategy established
- GV.RR-01 to GV.RR-04: Roles, responsibilities, and authorities
- GV.PO-01 to GV.PO-02: Cybersecurity policy established
- GV.OV-01 to GV.OV-03: Cybersecurity strategy oversight
- GV.SC-01 to GV.SC-10: Supply chain risk management

### ID - Identify
- ID.AM-01 to ID.AM-08: Asset management
- ID.RA-01 to ID.RA-10: Risk assessment
- ID.IM-01 to ID.IM-04: Improvement
- ID.BE (removed in 2.0, moved to GV)

### PR - Protect
- PR.AA-01 to PR.AA-06: Identity management, authentication, access control
- PR.AT-01 to PR.AT-02: Awareness and training
- PR.DS-01 to PR.DS-10: Data security
- PR.PS-01 to PR.PS-06: Platform security
- PR.IR-01 to PR.IR-02: Technology infrastructure resilience

### DE - Detect
- DE.CM-01 to DE.CM-09: Continuous monitoring
- DE.AE-01 to DE.AE-08: Adverse event analysis

### RS - Respond
- RS.MA-01 to RS.MA-05: Incident management
- RS.AN-01 to RS.AN-08: Incident analysis
- RS.CO-01 to RS.CO-03: Incident response reporting and communication
- RS.MI-01 to RS.MI-02: Incident mitigation

### RC - Recover
- RC.RP-01 to RC.RP-06: Incident recovery plan execution

## Implementation Tiers Detail

### Tier 1: Partial
- **Risk Management Process**: Ad hoc; not formalized
- **Integrated Risk Management**: Limited awareness; irregular risk practices
- **External Participation**: No formal collaboration with external entities
- **Governance**: Cybersecurity not integrated into enterprise risk management

### Tier 2: Risk-Informed
- **Risk Management Process**: Approved by management but may not be policy
- **Integrated Risk Management**: Awareness at organizational level; some sharing
- **External Participation**: Aware of ecosystem role; limited collaboration
- **Governance**: Some integration of cybersecurity into enterprise risk

### Tier 3: Repeatable
- **Risk Management Process**: Formally approved policies; regularly updated
- **Integrated Risk Management**: Organization-wide approach; consistent implementation
- **External Participation**: Regular collaboration with partners and ecosystem
- **Governance**: Cybersecurity fully integrated into enterprise risk management

### Tier 4: Adaptive
- **Risk Management Process**: Continuous improvement using advanced techniques
- **Integrated Risk Management**: Real-time shared awareness; dynamic risk response
- **External Participation**: Active leadership in ecosystem risk management
- **Governance**: Agile cybersecurity governance adapting to changes

## Related NIST Publications
- **SP 800-53 Rev 5**: Security and Privacy Controls (detailed control catalog)
- **SP 800-37 Rev 2**: Risk Management Framework (RMF)
- **SP 800-30 Rev 1**: Guide for Conducting Risk Assessments
- **SP 800-171 Rev 3**: Protecting CUI in Nonfederal Systems
- **SP 800-207**: Zero Trust Architecture
- **SP 800-218**: Secure Software Development Framework (SSDF)

## CSF 2.0 Informative References
The CSF references specific controls from:
- NIST SP 800-53 Rev 5
- ISO/IEC 27001:2022
- CIS Controls v8
- COBIT 2019
- ISA/IEC 62443 (Industrial Control Systems)

===== END FILE =====


===== FILE: skills/performing-nist-csf-maturity-assessment/references/workflows.md =====
# NIST CSF Maturity Assessment Workflows

## Workflow 1: Assessment Planning

```
Start
  |
  v
[Define Assessment Scope]
  - Enterprise-wide or business unit
  - Include/exclude OT systems
  - Include/exclude third parties
  |
  v
[Identify Stakeholders]
  - CISO and security team
  - IT leadership
  - Business unit leaders
  - Risk management
  - Legal/compliance
  - Executive sponsors
  |
  v
[Select Assessment Approach]
  +--> Self-Assessment (internal team)
  +--> Facilitated (consultant-guided)
  +--> Third-Party (independent assessment)
  |
  v
[Gather Documentation]
  - Security policies and procedures
  - Risk assessments and registers
  - Architecture diagrams
  - Previous audit results
  - Incident reports
  - Training records
  |
  v
[Schedule Assessment Activities]
  |
  v
End
```

## Workflow 2: Current State Scoring

```
Start
  |
  v
[For Each CSF Function (GV, ID, PR, DE, RS, RC)]
  |
  v
  [For Each Category in Function]
    |
    v
    [For Each Subcategory]
      |
      v
      [Evaluate Against Tier Criteria]
        |
        +--> Tier 1 (Partial)?
        |     - No formal process
        |     - Ad hoc practices
        |     - Limited documentation
        |
        +--> Tier 2 (Risk-Informed)?
        |     - Approved by management
        |     - Inconsistent application
        |     - Some documentation
        |
        +--> Tier 3 (Repeatable)?
        |     - Formal policies
        |     - Consistent implementation
        |     - Regular updates
        |     - Metrics captured
        |
        +--> Tier 4 (Adaptive)?
              - Continuous improvement
              - Real-time adaptation
              - Advanced automation
              - Lessons learned integrated
      |
      v
      [Document Score and Evidence]
      |
      v
      [Record Strengths and Gaps]
    |
    v
  [Calculate Category Average Score]
  |
  v
[Calculate Function Average Score]
  |
  v
[Generate Current Profile Heatmap]
  |
  v
End
```

## Workflow 3: Gap Analysis

```
Start
  |
  v
[Define Target Profile]
  - Executive input on risk appetite
  - Industry benchmark comparison
  - Regulatory requirements
  - Available resources
  |
  v
[Compare Current vs Target for Each Subcategory]
  Gap = Target Tier - Current Tier
  |
  v
[Classify Gaps]
  |
  +--> Critical (Gap >= 2 tiers, high-risk area)
  +--> Significant (Gap = 1 tier, high-risk area)
  +--> Moderate (Gap = 1 tier, medium-risk area)
  +--> Minor (Gap = 1 tier, low-risk area)
  +--> None (current meets or exceeds target)
  |
  v
[Prioritize Based On]
  - Risk reduction impact
  - Regulatory requirements
  - Implementation effort
  - Cost and resource availability
  - Dependencies on other improvements
  |
  v
[Generate Prioritized Gap Report]
  |
  v
End
```

## Workflow 4: Improvement Roadmap

```
Start
  |
  v
[Quick Wins (0-3 months)]
  - Low effort, high impact
  - Policy updates and documentation
  - Enable existing but unused capabilities
  - Awareness training refresh
  |
  v
[Medium-Term (3-12 months)]
  - Tool deployment and configuration
  - Process formalization
  - Staff training and certification
  - Vendor security programme establishment
  |
  v
[Long-Term (12-24 months)]
  - Architecture redesign
  - Advanced automation (SOAR, AI-driven)
  - Cultural transformation
  - Advanced threat detection capabilities
  |
  v
[Assign Ownership and Budget]
  - Initiative owner for each item
  - Resource allocation
  - Budget approval
  - Success metrics
  |
  v
[Track Progress Quarterly]
  - Milestone reviews
  - Reassess maturity scores
  - Adjust roadmap as needed
  |
  v
End
```

===== END FILE =====


===== FILE: skills/performing-nist-csf-maturity-assessment/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing NIST Cybersecurity Framework (CSF) maturity assessment."""

import json
import argparse
import csv
from datetime import datetime


NIST_CSF_FUNCTIONS = {
    "IDENTIFY": {
        "categories": ["ID.AM", "ID.BE", "ID.GV", "ID.RA", "ID.RM", "ID.SC"],
        "descriptions": {
            "ID.AM": "Asset Management",
            "ID.BE": "Business Environment",
            "ID.GV": "Governance",
            "ID.RA": "Risk Assessment",
            "ID.RM": "Risk Management Strategy",
            "ID.SC": "Supply Chain Risk Management",
        },
    },
    "PROTECT": {
        "categories": ["PR.AC", "PR.AT", "PR.DS", "PR.IP", "PR.MA", "PR.PT"],
        "descriptions": {
            "PR.AC": "Identity Management & Access Control",
            "PR.AT": "Awareness and Training",
            "PR.DS": "Data Security",
            "PR.IP": "Information Protection Processes",
            "PR.MA": "Maintenance",
            "PR.PT": "Protective Technology",
        },
    },
    "DETECT": {
        "categories": ["DE.AE", "DE.CM", "DE.DP"],
        "descriptions": {
            "DE.AE": "Anomalies and Events",
            "DE.CM": "Security Continuous Monitoring",
            "DE.DP": "Detection Processes",
        },
    },
    "RESPOND": {
        "categories": ["RS.RP", "RS.CO", "RS.AN", "RS.MI", "RS.IM"],
        "descriptions": {
            "RS.RP": "Response Planning",
            "RS.CO": "Communications",
            "RS.AN": "Analysis",
            "RS.MI": "Mitigation",
            "RS.IM": "Improvements",
        },
    },
    "RECOVER": {
        "categories": ["RC.RP", "RC.IM", "RC.CO"],
        "descriptions": {
            "RC.RP": "Recovery Planning",
            "RC.IM": "Improvements",
            "RC.CO": "Communications",
        },
    },
}

MATURITY_LEVELS = {
    1: "Partial — Risk management practices not formalized",
    2: "Risk Informed — Risk management approved but not org-wide",
    3: "Repeatable — Policies and practices formally approved and expressed as policy",
    4: "Adaptive — Organization adapts based on lessons learned and predictive indicators",
}


def assess_from_csv(assessment_file):
    """Load assessment responses from CSV and calculate maturity scores."""
    with open(assessment_file, "r", encoding="utf-8", errors="replace") as f:
        reader = csv.DictReader(f)
        rows = list(reader)
    scores = {}
    for row in rows:
        category = row.get("category", row.get("Category", row.get("subcategory", "")))
        score = int(row.get("score", row.get("maturity_level", row.get("Score", 0))))
        target = int(row.get("target", row.get("Target", 3)))
        function_name = ""
        for fn, data in NIST_CSF_FUNCTIONS.items():
            if any(category.startswith(c) for c in data["categories"]):
                function_name = fn
                break
        scores.setdefault(function_name or "UNKNOWN", []).append({
            "category": category, "score": score, "target": target, "gap": target - score,
        })
    function_scores = {}
    for fn, items in scores.items():
        avg = sum(i["score"] for i in items) / len(items) if items else 0
        avg_target = sum(i["target"] for i in items) / len(items) if items else 0
        function_scores[fn] = {
            "average_maturity": round(avg, 1),
            "target_maturity": round(avg_target, 1),
            "gap": round(avg_target - avg, 1),
            "categories_assessed": len(items),
            "below_target": sum(1 for i in items if i["gap"] > 0),
        }
    overall = sum(fs["average_maturity"] for fs in function_scores.values()) / max(len(function_scores), 1)
    return {
        "assessment_file": assessment_file,
        "overall_maturity": round(overall, 1),
        "overall_level": MATURITY_LEVELS.get(round(overall), "Unknown"),
        "function_scores": function_scores,
        "total_categories": sum(fs["categories_assessed"] for fs in function_scores.values()),
        "categories_below_target": sum(fs["below_target"] for fs in function_scores.values()),
    }


def generate_gap_analysis(assessment_file):
    """Generate detailed gap analysis with prioritized recommendations."""
    assessment = assess_from_csv(assessment_file)
    gaps = []
    for fn, data in assessment["function_scores"].items():
        if data["gap"] > 0:
            gaps.append({
                "function": fn, "current": data["average_maturity"],
                "target": data["target_maturity"], "gap": data["gap"],
                "priority": "HIGH" if data["gap"] >= 2 else "MEDIUM" if data["gap"] >= 1 else "LOW",
            })
    gaps.sort(key=lambda x: x["gap"], reverse=True)
    return {
        "generated": datetime.utcnow().isoformat(),
        "overall_maturity": assessment["overall_maturity"],
        "gaps": gaps,
        "high_priority_gaps": [g for g in gaps if g["priority"] == "HIGH"],
    }


def create_assessment_template(output_file=None):
    """Create a blank NIST CSF assessment CSV template."""
    rows = [["category", "description", "score", "target", "evidence", "notes"]]
    for fn, data in NIST_CSF_FUNCTIONS.items():
        for cat in data["categories"]:
            rows.append([cat, data["descriptions"].get(cat, ""), "", "3", "", ""])
    if output_file:
        with open(output_file, "w", newline="", encoding="utf-8") as f:
            writer = csv.writer(f)
            writer.writerows(rows)
    return {"template_rows": len(rows) - 1, "functions": list(NIST_CSF_FUNCTIONS.keys()),
            "output": output_file, "categories": [r[0] for r in rows[1:]]}


def generate_executive_summary(assessment_file):
    """Generate executive-level maturity summary."""
    assessment = assess_from_csv(assessment_file)
    gap = generate_gap_analysis(assessment_file)
    return {
        "generated": datetime.utcnow().isoformat(),
        "framework": "NIST CSF 2.0",
        "overall_maturity_score": assessment["overall_maturity"],
        "maturity_level": assessment["overall_level"],
        "total_categories_assessed": assessment["total_categories"],
        "categories_meeting_target": assessment["total_categories"] - assessment["categories_below_target"],
        "categories_below_target": assessment["categories_below_target"],
        "function_summary": {fn: {"score": d["average_maturity"], "target": d["target_maturity"]}
                            for fn, d in assessment["function_scores"].items()},
        "top_gaps": gap["high_priority_gaps"][:5],
    }


def main():
    parser = argparse.ArgumentParser(description="NIST CSF Maturity Assessment Agent")
    sub = parser.add_subparsers(dest="command")
    a = sub.add_parser("assess", help="Run maturity assessment from CSV")
    a.add_argument("--csv", required=True)
    g = sub.add_parser("gaps", help="Generate gap analysis")
    g.add_argument("--csv", required=True)
    t = sub.add_parser("template", help="Create assessment template")
    t.add_argument("--output", help="Output CSV file path")
    e = sub.add_parser("executive", help="Executive summary")
    e.add_argument("--csv", required=True)
    args = parser.parse_args()
    if args.command == "assess":
        result = assess_from_csv(args.csv)
    elif args.command == "gaps":
        result = generate_gap_analysis(args.csv)
    elif args.command == "template":
        result = create_assessment_template(args.output)
    elif args.command == "executive":
        result = generate_executive_summary(args.csv)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-nist-csf-maturity-assessment/scripts/process.py =====
#!/usr/bin/env python3
"""
NIST CSF 2.0 Maturity Assessment Automation

Automates maturity scoring across all 6 CSF functions, gap analysis
between current and target profiles, and improvement roadmap generation.
"""

import json
import csv
from datetime import datetime
from pathlib import Path
from dataclasses import dataclass, field, asdict
from typing import Optional

# CSF 2.0 Category structure
CSF_CATEGORIES = {
    "GV": {
        "name": "Govern",
        "categories": {
            "GV.OC": "Organizational Context",
            "GV.RM": "Risk Management Strategy",
            "GV.RR": "Roles, Responsibilities, and Authorities",
            "GV.PO": "Policy",
            "GV.OV": "Oversight",
            "GV.SC": "Cybersecurity Supply Chain Risk Management",
        }
    },
    "ID": {
        "name": "Identify",
        "categories": {
            "ID.AM": "Asset Management",
            "ID.RA": "Risk Assessment",
            "ID.IM": "Improvement",
        }
    },
    "PR": {
        "name": "Protect",
        "categories": {
            "PR.AA": "Identity Management, Authentication, and Access Control",
            "PR.AT": "Awareness and Training",
            "PR.DS": "Data Security",
            "PR.PS": "Platform Security",
            "PR.IR": "Technology Infrastructure Resilience",
        }
    },
    "DE": {
        "name": "Detect",
        "categories": {
            "DE.CM": "Continuous Monitoring",
            "DE.AE": "Adverse Event Analysis",
        }
    },
    "RS": {
        "name": "Respond",
        "categories": {
            "RS.MA": "Incident Management",
            "RS.AN": "Incident Analysis",
            "RS.CO": "Incident Response Reporting and Communication",
            "RS.MI": "Incident Mitigation",
        }
    },
    "RC": {
        "name": "Recover",
        "categories": {
            "RC.RP": "Incident Recovery Plan Execution",
        }
    },
}

TIER_DESCRIPTIONS = {
    1: "Partial - Ad hoc, reactive, limited awareness",
    2: "Risk-Informed - Approved practices, inconsistent application",
    3: "Repeatable - Formal policies, consistent implementation",
    4: "Adaptive - Continuous improvement, real-time adaptation",
}


@dataclass
class CategoryAssessment:
    category_id: str
    category_name: str
    function_id: str
    current_tier: int
    target_tier: int
    evidence: str = ""
    strengths: list = field(default_factory=list)
    gaps: list = field(default_factory=list)
    recommendations: list = field(default_factory=list)

    @property
    def gap_size(self) -> int:
        return max(0, self.target_tier - self.current_tier)

    @property
    def gap_priority(self) -> str:
        gap = self.gap_size
        if gap >= 2:
            return "Critical"
        elif gap == 1 and self.target_tier >= 3:
            return "High"
        elif gap == 1:
            return "Medium"
        else:
            return "On Target"


@dataclass
class ImprovementInitiative:
    initiative_id: str
    category_id: str
    title: str
    description: str
    current_tier: int
    target_tier: int
    timeframe: str  # Quick Win, Medium-Term, Long-Term
    effort: str     # Low, Medium, High
    impact: str     # Low, Medium, High
    owner: str = ""
    estimated_cost: str = ""
    status: str = "Planned"


class NISTCSFAssessment:
    """Conducts NIST CSF 2.0 maturity assessment."""

    def __init__(self, output_dir: str = "./nist_csf_output",
                 organization: str = "Organization"):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.organization = organization
        self.assessments: list[CategoryAssessment] = []
        self.initiatives: list[ImprovementInitiative] = []

    def assess_maturity(self, scores: list[dict]) -> list[CategoryAssessment]:
        """Score each CSF category against implementation tiers."""
        print("\n" + "=" * 70)
        print(f"NIST CSF 2.0 MATURITY ASSESSMENT - {self.organization}")
        print("=" * 70)

        for score_data in scores:
            assessment = CategoryAssessment(**score_data)
            self.assessments.append(assessment)

        # Display by function
        for func_id, func_info in CSF_CATEGORIES.items():
            func_assessments = [a for a in self.assessments if a.function_id == func_id]
            if not func_assessments:
                continue

            avg_current = sum(a.current_tier for a in func_assessments) / len(func_assessments)
            avg_target = sum(a.target_tier for a in func_assessments) / len(func_assessments)

            print(f"\n  {func_info['name']} ({func_id}) - Avg Current: {avg_current:.1f} | Avg Target: {avg_target:.1f}")
            print(f"  {'Category':<50} {'Current':>8} {'Target':>8} {'Gap':>5} {'Priority':>10}")
            print(f"  {'-'*85}")

            for a in func_assessments:
                gap_indicator = f"+{a.gap_size}" if a.gap_size > 0 else "0"
                print(f"  {a.category_name:<50} {a.current_tier:>5}/4  {a.target_tier:>5}/4  {gap_indicator:>5} {a.gap_priority:>10}")

        # Overall summary
        total_avg_current = sum(a.current_tier for a in self.assessments) / len(self.assessments)
        total_avg_target = sum(a.target_tier for a in self.assessments) / len(self.assessments)

        print(f"\n  {'='*85}")
        print(f"  Overall Average Current Tier: {total_avg_current:.2f}")
        print(f"  Overall Average Target Tier:  {total_avg_target:.2f}")
        print(f"  Overall Gap:                  {total_avg_target - total_avg_current:.2f}")

        # Tier distribution
        tier_dist = {1: 0, 2: 0, 3: 0, 4: 0}
        for a in self.assessments:
            tier_dist[a.current_tier] += 1

        print(f"\n  Current Tier Distribution:")
        for tier, count in tier_dist.items():
            bar = "#" * (count * 3)
            print(f"    Tier {tier}: {count:>3} categories {bar}")

        # Save assessment
        report_path = self.output_dir / "maturity_assessment.json"
        with open(report_path, "w") as f:
            json.dump({
                "organization": self.organization,
                "date": datetime.now().isoformat(),
                "overall_current": total_avg_current,
                "overall_target": total_avg_target,
                "assessments": [asdict(a) for a in self.assessments],
            }, f, indent=2)

        print(f"\n  Assessment saved to: {report_path}")
        return self.assessments

    def generate_gap_analysis(self) -> dict:
        """Generate gap analysis between current and target profiles."""
        print("\n" + "=" * 70)
        print("GAP ANALYSIS")
        print("=" * 70)

        gaps = {
            "Critical": [],
            "High": [],
            "Medium": [],
            "On Target": [],
        }

        for a in self.assessments:
            gaps[a.gap_priority].append({
                "category": a.category_id,
                "name": a.category_name,
                "current": a.current_tier,
                "target": a.target_tier,
                "gap": a.gap_size,
                "gaps_detail": a.gaps,
                "recommendations": a.recommendations,
            })

        for priority in ["Critical", "High", "Medium", "On Target"]:
            items = gaps[priority]
            print(f"\n  {priority} ({len(items)} categories):")
            for item in items:
                if priority != "On Target":
                    print(f"    {item['category']}: {item['name']} (Tier {item['current']} -> {item['target']})")
                    for gap in item.get("gaps_detail", []):
                        print(f"      - {gap}")

        gap_path = self.output_dir / "gap_analysis.json"
        with open(gap_path, "w") as f:
            json.dump(gaps, f, indent=2)

        print(f"\n  Gap Analysis saved to: {gap_path}")
        return gaps

    def create_roadmap(self, initiatives: list[dict]) -> list[ImprovementInitiative]:
        """Create improvement roadmap with prioritized initiatives."""
        print("\n" + "=" * 70)
        print("IMPROVEMENT ROADMAP")
        print("=" * 70)

        for init_data in initiatives:
            initiative = ImprovementInitiative(**init_data)
            self.initiatives.append(initiative)

        timeframes = {"Quick Win (0-3 months)": [], "Medium-Term (3-12 months)": [], "Long-Term (12-24 months)": []}
        for init in self.initiatives:
            timeframes.setdefault(init.timeframe, [])
            timeframes[init.timeframe].append(init)

        for tf, items in timeframes.items():
            print(f"\n  {tf}:")
            for item in items:
                print(f"    [{item.initiative_id}] {item.title}")
                print(f"      Category: {item.category_id} | Effort: {item.effort} | Impact: {item.impact}")
                if item.owner:
                    print(f"      Owner: {item.owner}")

        roadmap_path = self.output_dir / "improvement_roadmap.json"
        with open(roadmap_path, "w") as f:
            json.dump([asdict(i) for i in self.initiatives], f, indent=2)

        print(f"\n  Roadmap saved to: {roadmap_path}")
        return self.initiatives

    def generate_executive_summary(self) -> dict:
        """Generate executive-level maturity summary."""
        print("\n" + "=" * 70)
        print("EXECUTIVE SUMMARY")
        print("=" * 70)

        func_scores = {}
        for func_id, func_info in CSF_CATEGORIES.items():
            func_assessments = [a for a in self.assessments if a.function_id == func_id]
            if func_assessments:
                avg = sum(a.current_tier for a in func_assessments) / len(func_assessments)
                target_avg = sum(a.target_tier for a in func_assessments) / len(func_assessments)
                func_scores[func_info["name"]] = {"current": round(avg, 2), "target": round(target_avg, 2)}

        summary = {
            "organization": self.organization,
            "date": datetime.now().isoformat(),
            "function_scores": func_scores,
            "total_categories": len(self.assessments),
            "critical_gaps": sum(1 for a in self.assessments if a.gap_priority == "Critical"),
            "high_gaps": sum(1 for a in self.assessments if a.gap_priority == "High"),
            "improvement_initiatives": len(self.initiatives),
        }

        print(f"\n  Organization: {self.organization}")
        print(f"\n  Function Maturity Scores:")
        for func, scores in func_scores.items():
            bar_current = "|" * int(scores["current"] * 5)
            bar_target = "." * int((scores["target"] - scores["current"]) * 5)
            print(f"    {func:<15} Current: {scores['current']:.1f}/4  Target: {scores['target']:.1f}/4  {bar_current}{bar_target}")

        print(f"\n  Critical Gaps: {summary['critical_gaps']}")
        print(f"  High Gaps: {summary['high_gaps']}")
        print(f"  Planned Initiatives: {summary['improvement_initiatives']}")

        summary_path = self.output_dir / "executive_summary.json"
        with open(summary_path, "w") as f:
            json.dump(summary, f, indent=2)

        print(f"\n  Executive Summary saved to: {summary_path}")
        return summary


def main():
    """Run NIST CSF 2.0 maturity assessment."""
    assessment = NISTCSFAssessment(organization="Example Corporation")

    sample_scores = [
        {"category_id": "GV.OC", "category_name": "Organizational Context", "function_id": "GV", "current_tier": 2, "target_tier": 3, "gaps": ["Risk appetite not formally documented"], "recommendations": ["Document and approve risk appetite statement"]},
        {"category_id": "GV.RM", "category_name": "Risk Management Strategy", "function_id": "GV", "current_tier": 2, "target_tier": 3, "gaps": ["Strategy not updated annually"], "recommendations": ["Establish annual review cycle"]},
        {"category_id": "GV.RR", "category_name": "Roles and Responsibilities", "function_id": "GV", "current_tier": 3, "target_tier": 3},
        {"category_id": "GV.PO", "category_name": "Policy", "function_id": "GV", "current_tier": 2, "target_tier": 3, "gaps": ["Policies not reviewed in 18+ months"], "recommendations": ["Implement annual policy review cycle"]},
        {"category_id": "GV.OV", "category_name": "Oversight", "function_id": "GV", "current_tier": 1, "target_tier": 3, "gaps": ["No board-level security reporting", "No metrics dashboard"], "recommendations": ["Establish quarterly board reporting", "Deploy security metrics dashboard"]},
        {"category_id": "GV.SC", "category_name": "Supply Chain Risk Management", "function_id": "GV", "current_tier": 1, "target_tier": 3, "gaps": ["No supply chain risk programme", "Vendor assessments ad hoc"], "recommendations": ["Establish vendor risk management programme"]},
        {"category_id": "ID.AM", "category_name": "Asset Management", "function_id": "ID", "current_tier": 2, "target_tier": 3, "gaps": ["Asset inventory incomplete for cloud"], "recommendations": ["Extend CMDB to cloud assets"]},
        {"category_id": "ID.RA", "category_name": "Risk Assessment", "function_id": "ID", "current_tier": 2, "target_tier": 3, "gaps": ["Risk assessments not quantitative"], "recommendations": ["Adopt FAIR methodology for quantification"]},
        {"category_id": "ID.IM", "category_name": "Improvement", "function_id": "ID", "current_tier": 2, "target_tier": 3},
        {"category_id": "PR.AA", "category_name": "Identity and Access Control", "function_id": "PR", "current_tier": 3, "target_tier": 3},
        {"category_id": "PR.AT", "category_name": "Awareness and Training", "function_id": "PR", "current_tier": 2, "target_tier": 3, "gaps": ["No role-based training"], "recommendations": ["Implement role-based security training"]},
        {"category_id": "PR.DS", "category_name": "Data Security", "function_id": "PR", "current_tier": 2, "target_tier": 3, "gaps": ["DLP not fully deployed"], "recommendations": ["Complete DLP deployment"]},
        {"category_id": "PR.PS", "category_name": "Platform Security", "function_id": "PR", "current_tier": 3, "target_tier": 3},
        {"category_id": "PR.IR", "category_name": "Infrastructure Resilience", "function_id": "PR", "current_tier": 2, "target_tier": 3},
        {"category_id": "DE.CM", "category_name": "Continuous Monitoring", "function_id": "DE", "current_tier": 2, "target_tier": 3, "gaps": ["Limited cloud monitoring", "No OT monitoring"], "recommendations": ["Extend SIEM to cloud and OT"]},
        {"category_id": "DE.AE", "category_name": "Adverse Event Analysis", "function_id": "DE", "current_tier": 2, "target_tier": 3, "gaps": ["Manual correlation only"], "recommendations": ["Implement automated correlation"]},
        {"category_id": "RS.MA", "category_name": "Incident Management", "function_id": "RS", "current_tier": 3, "target_tier": 3},
        {"category_id": "RS.AN", "category_name": "Incident Analysis", "function_id": "RS", "current_tier": 2, "target_tier": 3},
        {"category_id": "RS.CO", "category_name": "Response Communication", "function_id": "RS", "current_tier": 2, "target_tier": 3},
        {"category_id": "RS.MI", "category_name": "Incident Mitigation", "function_id": "RS", "current_tier": 2, "target_tier": 3},
        {"category_id": "RC.RP", "category_name": "Recovery Plan Execution", "function_id": "RC", "current_tier": 2, "target_tier": 3, "gaps": ["DR plan not tested in 2+ years"], "recommendations": ["Conduct DR tabletop and technical test"]},
    ]

    assessment.assess_maturity(sample_scores)
    assessment.generate_gap_analysis()

    sample_initiatives = [
        {"initiative_id": "INIT-001", "category_id": "GV.OV", "title": "Establish Board Security Reporting", "description": "Create quarterly security report template and present to board", "current_tier": 1, "target_tier": 3, "timeframe": "Quick Win (0-3 months)", "effort": "Low", "impact": "High", "owner": "CISO"},
        {"initiative_id": "INIT-002", "category_id": "GV.SC", "title": "Vendor Risk Management Programme", "description": "Establish formal vendor assessment and monitoring programme", "current_tier": 1, "target_tier": 3, "timeframe": "Medium-Term (3-12 months)", "effort": "Medium", "impact": "High", "owner": "Security Manager"},
        {"initiative_id": "INIT-003", "category_id": "DE.CM", "title": "Cloud Security Monitoring", "description": "Extend SIEM to cover AWS/Azure/GCP workloads", "current_tier": 2, "target_tier": 3, "timeframe": "Medium-Term (3-12 months)", "effort": "High", "impact": "High", "owner": "SOC Manager"},
        {"initiative_id": "INIT-004", "category_id": "RC.RP", "title": "DR Testing Programme", "description": "Annual DR tabletop and semi-annual technical recovery tests", "current_tier": 2, "target_tier": 3, "timeframe": "Quick Win (0-3 months)", "effort": "Medium", "impact": "High", "owner": "IT Director"},
    ]

    assessment.create_roadmap(sample_initiatives)
    assessment.generate_executive_summary()

    print("\n" + "=" * 70)
    print("NIST CSF 2.0 MATURITY ASSESSMENT COMPLETE")
    print("=" * 70)


if __name__ == "__main__":
    main()

===== END FILE =====
