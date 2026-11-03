

===== FILE: skills/implementing-gdpr-data-subject-access-request/SKILL.md =====
---
name: implementing-gdpr-data-subject-access-request
description: 'Automates GDPR Data Subject Access Request (DSAR) workflows including
  identity verification, PII discovery across databases and files using regex and
  NER, data mapping, response templating per Article 15 requirements, deadline tracking,
  and audit logging. Covers ICO/EDPB guidance compliance, exemption handling, and
  scalable batch processing. Use when building or auditing DSAR response capabilities
  under GDPR/UK GDPR.

  '
domain: cybersecurity
subdomain: privacy-compliance
tags:
- gdpr
- dsar
- privacy
- pii-discovery
- data-subject-rights
- compliance
- article-15
version: '1.0'
author: mukul975
license: Apache-2.0
nist_csf:
- GV.PO-01
- PR.DS-01
- GV.OC-05
mitre_attack:
- T1078
- T1190
- T1059
---

# Implementing GDPR Data Subject Access Request (DSAR) Workflow

## When to Use

- When building automated DSAR processing pipelines for GDPR/UK GDPR compliance
- When implementing PII discovery across structured and unstructured data sources
- When creating response templates that satisfy Article 15 disclosure requirements
- When auditing existing DSAR handling for regulatory compliance gaps
- When scaling DSAR processing from manual to automated workflows

## Prerequisites

- Python 3.8+ with required dependencies (spacy, presidio-analyzer, jinja2)
- Access to data sources where personal data resides (databases, file shares, logs)
- Understanding of GDPR Article 15 requirements and ICO/EDPB guidance
- Appropriate authorization and data protection officer (DPO) approval
- Test environment with synthetic or anonymized data for validation

## Background

### GDPR Article 15 - Right of Access

Under GDPR Article 15, data subjects have the right to obtain from the controller:

1. **Confirmation** that their personal data is being processed
2. **A copy** of all personal data held about them
3. **Supplementary information** including:
   - Purposes of processing
   - Categories of personal data
   - Recipients or categories of recipients
   - Retention periods or criteria to determine them
   - Right to rectification, erasure, restriction, or objection
   - Right to lodge a complaint with a supervisory authority
   - Source of the data (if not collected directly from the subject)
   - Existence of automated decision-making, including profiling

### Timeline Requirements

- **Standard deadline**: 1 calendar month from receipt of valid request
- **Complex extension**: Up to 2 additional months (must notify within first month)
- **Clock pause**: Permitted when identity verification or clarification is needed
- **Format**: Electronic form if request made electronically (unless otherwise requested)
- **Cost**: Free of charge (unless manifestly unfounded/excessive)

### ICO/EDPB Guidance Key Points

- No formal format required for DSARs - verbal, written, social media all valid
- Request need not mention "subject access request" or cite Article 15
- Identity verification must be proportionate to the risk
- Exemptions exist for legal privilege, third-party data, trade secrets
- EDPB coordinated enforcement actions cover right of access compliance

## Instructions

### Step 1: DSAR Intake and Verification

Implement a request intake system that captures the request through any channel,
verifies the requester's identity, and starts the compliance clock.

```python
from agent import DSARWorkflowEngine

engine = DSARWorkflowEngine(config_path="dsar_config.json")

# Register a new DSAR
request = engine.register_dsar(
    requester_name="Jane Smith",
    requester_email="jane.smith@example.com",
    request_channel="email",
    request_text="I would like a copy of all personal data you hold about me.",
    identity_docs=["passport_verified"],
)
print(f"DSAR ID: {request['dsar_id']}, Deadline: {request['deadline']}")
```

### Step 2: PII Discovery Across Data Sources

Scan databases, files, and logs using regex patterns and NER to find all
personal data associated with the data subject.

```python
from agent import PIIDiscoveryEngine

pii_engine = PIIDiscoveryEngine()

# Scan structured data (database)
db_results = pii_engine.scan_database(
    connection_string="postgresql://user:pass@localhost/appdb",
    search_identifiers={"email": "jane.smith@example.com", "name": "Jane Smith"},
)

# Scan unstructured data (files, logs)
file_results = pii_engine.scan_files(
    directories=["/var/log/app", "/data/exports", "/data/documents"],
    search_identifiers={"email": "jane.smith@example.com", "name": "Jane Smith"},
)

# Scan with NER for contextual PII detection
ner_results = pii_engine.scan_with_ner(
    text_corpus=file_results["raw_text_matches"],
    entity_types=["PERSON", "EMAIL", "PHONE_NUMBER", "LOCATION", "DATE_OF_BIRTH"],
)

all_pii = pii_engine.consolidate_results(db_results, file_results, ner_results)
print(f"Found {all_pii['total_records']} PII records across {all_pii['source_count']} sources")
```

### Step 3: Data Mapping and Classification

Map discovered PII to processing purposes, legal bases, and retention periods
as required by Article 15.

```python
from agent import DataMapper

mapper = DataMapper(data_inventory_path="data_inventory.json")

# Map PII to Article 15 categories
mapped_data = mapper.map_to_article15(
    pii_records=all_pii,
    data_subject_id="jane.smith@example.com",
)

# Output includes processing purposes, recipients, retention for each data category
for category in mapped_data["categories"]:
    print(f"Category: {category['name']}")
    print(f"  Purpose: {category['processing_purpose']}")
    print(f"  Legal basis: {category['legal_basis']}")
    print(f"  Retention: {category['retention_period']}")
    print(f"  Recipients: {', '.join(category['recipients'])}")
```

### Step 4: Exemption Review

Apply exemptions where lawful (third-party data, legal privilege, trade secrets)
before compiling the response.

```python
from agent import ExemptionReviewer

reviewer = ExemptionReviewer()

# Check for applicable exemptions
review_result = reviewer.review_exemptions(
    mapped_data=mapped_data,
    exemption_checks=[
        "third_party_data",
        "legal_professional_privilege",
        "trade_secrets",
        "crime_prevention",
        "management_forecasting",
    ],
)

# Apply redactions where exemptions apply
redacted_data = reviewer.apply_redactions(mapped_data, review_result["exemptions"])
print(f"Applied {review_result['exemption_count']} exemptions")
```

### Step 5: Response Generation

Generate a compliant DSAR response package with cover letter, data export,
and supplementary information document.

```python
from agent import DSARResponseGenerator

generator = DSARResponseGenerator(template_dir="templates/")

# Generate complete response package
response = generator.generate_response(
    dsar_id=request["dsar_id"],
    data_subject="Jane Smith",
    mapped_data=redacted_data,
    format="pdf",  # or "json", "csv"
)

# Package includes: cover letter, data export, supplementary info, audit log
for doc in response["documents"]:
    print(f"Generated: {doc['filename']} ({doc['type']})")
```

### Step 6: Audit Trail and Compliance Logging

Maintain complete audit trail of the DSAR lifecycle for accountability.

```python
from agent import DSARAuditLogger

logger = DSARAuditLogger(log_path="dsar_audit_logs/")

# Log complete DSAR lifecycle
logger.log_event(request["dsar_id"], "request_received", {
    "channel": "email",
    "identity_verified": True,
})
logger.log_event(request["dsar_id"], "pii_discovery_complete", {
    "records_found": all_pii["total_records"],
    "sources_scanned": all_pii["source_count"],
})
logger.log_event(request["dsar_id"], "response_sent", {
    "format": "pdf",
    "documents_count": len(response["documents"]),
    "exemptions_applied": review_result["exemption_count"],
})

# Generate compliance report
compliance_report = logger.generate_compliance_report(request["dsar_id"])
```

## Examples

### Complete DSAR Processing Pipeline

```python
from agent import DSARWorkflowEngine, PIIDiscoveryEngine, DSARResponseGenerator

# Full automated pipeline
engine = DSARWorkflowEngine(config_path="dsar_config.json")
pii = PIIDiscoveryEngine()
gen = DSARResponseGenerator(template_dir="templates/")

# 1. Intake
req = engine.register_dsar(
    requester_name="John Doe",
    requester_email="john.doe@example.com",
    request_channel="web_form",
    request_text="Please provide all my data under GDPR Article 15.",
    identity_docs=["email_verified", "account_match"],
)

# 2. Discover
results = pii.full_scan(
    search_identifiers={"email": "john.doe@example.com"},
    sources=["database", "files", "logs"],
)

# 3. Generate response
response = gen.generate_response(
    dsar_id=req["dsar_id"],
    data_subject="John Doe",
    mapped_data=results,
)

# 4. Track deadline
engine.update_status(req["dsar_id"], "response_sent")
print(f"DSAR {req['dsar_id']} completed, {engine.days_remaining(req['dsar_id'])} days remaining")
```

### PII Regex Pattern Testing

```python
from agent import PIIPatternMatcher

matcher = PIIPatternMatcher()

# Test individual patterns
test_text = "Contact jane.smith@example.com or call +44 20 7946 0958. SSN: 123-45-6789"
matches = matcher.scan_text(test_text)
for m in matches:
    print(f"  [{m['type']}] '{m['value']}' (confidence: {m['confidence']})")
```

## References

- GDPR Article 15: https://gdpr-info.eu/art-15-gdpr/
- ICO Subject Access Request Guidance: https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/subject-access-requests/
- EDPB Guidelines 01/2022 on Right of Access: https://www.edpb.europa.eu/system/files/2023-04/edpb_guidelines_202201_data_subject_rights_access_v2_en.pdf
- GDPR Article 12 (DSAR Modalities): https://gdpr-info.eu/art-12-gdpr/
- Regulation (EU) 2025/2518 (Procedural Rules): Cross-border GDPR enforcement procedural rules

===== END FILE =====


===== FILE: skills/implementing-gdpr-data-subject-access-request/LICENSE =====

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


===== FILE: skills/implementing-gdpr-data-subject-access-request/references/api-reference.md =====
# API Reference: GDPR DSAR Workflow Automation

## PIIPatternMatcher

Scans text for PII using compiled regex patterns with confidence scoring and contextual boosting.

### Constructor
```python
PIIPatternMatcher(custom_patterns=None)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `custom_patterns` | `dict` or `None` | Additional regex patterns to include in scanning |

### Methods

#### `scan_text(text, min_confidence=0.5)`
Scan a string for PII matches.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | required | Text to scan for PII |
| `min_confidence` | `float` | `0.5` | Minimum confidence threshold (0.0-1.0) |

**Returns:** `list[dict]` -- Each match contains `type`, `value`, `description`, `confidence`, `gdpr_category`, `position`.

#### `scan_file(file_path, min_confidence=0.5)`
Scan a file on disk for PII matches.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `file_path` | `str` | required | Absolute path to the file |
| `min_confidence` | `float` | `0.5` | Minimum confidence threshold |

**Returns:** `dict` with `file`, `size_bytes`, `matches`, `match_count`, `pii_types_found`.

### Built-in PII Patterns

| Pattern Name | Description | Confidence | GDPR Category |
|-------------|-------------|------------|---------------|
| `email` | Email address | 0.95 | contact_information |
| `phone_international` | International phone number | 0.70 | contact_information |
| `uk_phone` | UK phone number | 0.80 | contact_information |
| `ssn_us` | US Social Security Number | 0.85 | government_id |
| `nino_uk` | UK National Insurance Number | 0.90 | government_id |
| `credit_card` | Credit/debit card number | 0.85 | financial_data |
| `iban` | International Bank Account Number | 0.80 | financial_data |
| `ipv4` | IPv4 address | 0.60 | online_identifier |
| `date_of_birth` | Date of birth (DD/MM/YYYY) | 0.65 | demographic_data |
| `uk_postcode` | UK postcode | 0.75 | location_data |
| `passport_uk` | UK passport number (9 digits) | 0.40 | government_id |
| `eu_vat` | EU VAT number | 0.50 | financial_data |

---

## PIIDiscoveryEngine

Discovers PII across structured (database) and unstructured (files) data sources.

### Constructor
```python
PIIDiscoveryEngine(custom_patterns=None)
```

### Methods

#### `scan_database(connection_string, search_identifiers, tables=None)`
Generate parameterized SQL queries for PII discovery in databases.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `connection_string` | `str` | required | Database connection string (redacted in output) |
| `search_identifiers` | `dict` | required | Key-value pairs to search for (e.g., `{"email": "user@example.com"}`) |
| `tables` | `list[str]` or `None` | auto | Tables to scan; defaults to common tables |

**Returns:** `dict` with `source_type`, `connection`, `tables_scanned`, `queries_generated`, `queries`.

#### `scan_files(directories, search_identifiers, file_extensions=None, max_file_size_mb=50)`
Scan files in directories for PII matching identifiers.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `directories` | `list[str]` | required | Directory paths to scan |
| `search_identifiers` | `dict` | required | Identifiers to search for |
| `file_extensions` | `list[str]` or `None` | common types | File extensions to include |
| `max_file_size_mb` | `int` | `50` | Skip files larger than this |

**Returns:** `dict` with `files_scanned`, `files_with_matches`, `matches`, `raw_text_matches`.

#### `scan_with_ner(text_corpus, entity_types=None, confidence_threshold=0.7)`
Scan text using Named Entity Recognition (spaCy NER with regex fallback).

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `text_corpus` | `list[str]` | required | List of file paths to scan |
| `entity_types` | `list[str]` or `None` | common types | NER entity types to detect |
| `confidence_threshold` | `float` | `0.7` | Minimum confidence for results |

**Supported Entity Types:** `PERSON`, `EMAIL`, `PHONE_NUMBER`, `LOCATION`, `DATE_OF_BIRTH`, `ORG`, `GPE`

**Returns:** `dict` with `files_processed`, `total_entities`, `results`, `model_used`.

#### `consolidate_results(*result_sets)`
Merge results from database, file, and NER scans into a unified record set.

**Returns:** `dict` with `total_records`, `source_count`, `sources`, `records`.

#### `full_scan(search_identifiers, sources=None, db_connection="", directories=None)`
Run a complete PII discovery scan across all source types.

**Returns:** Consolidated `dict` from all scans.

---

## DataMapper

Maps discovered PII to GDPR Article 15 disclosure categories.

### Constructor
```python
DataMapper(data_inventory_path=None)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `data_inventory_path` | `str` or `None` | Path to JSON data inventory for overrides |

### Methods

#### `map_to_article15(pii_records, data_subject_id)`
Map PII records to Article 15 required categories including processing purposes, legal basis, retention periods, and recipients.

**Returns:** `dict` with `categories`, `supplementary_info`, `article_15_reference`.

### Article 15 Categories Mapped

| Category | Article Reference | Contents |
|----------|-------------------|----------|
| Processing Purposes | Art. 15(1)(a) | Why data is processed |
| Data Categories | Art. 15(1)(b) | Types of personal data |
| Recipients | Art. 15(1)(c) | Who receives the data |
| Retention Period | Art. 15(1)(d) | How long data is kept |
| Data Subject Rights | Art. 15(1)(e-f) | Rights to rectify, erase, restrict, object |
| Data Source | Art. 15(1)(g) | Where data was collected from |
| Automated Decisions | Art. 15(1)(h) | Profiling and automated decision-making |
| International Transfers | Art. 15(2) | Safeguards for cross-border transfers |

---

## ExemptionReviewer

Reviews DSAR data against applicable GDPR/UK GDPR exemptions.

### Methods

#### `review_exemptions(mapped_data, exemption_checks=None)`
Flag applicable exemptions for DPO review.

**Returns:** `dict` with `exemption_count`, `exemptions`, `review_status`.

#### `apply_redactions(mapped_data, approved_exemptions)`
Apply approved exemption redactions to the mapped data.

**Returns:** Redacted `dict` with `redaction_log`.

### Supported Exemption Types

| Type | Legal Basis | Action |
|------|-------------|--------|
| `third_party_data` | Art. 15(4) / DPA 2018 Sch. 2 Para 16 | redact |
| `legal_professional_privilege` | DPA 2018 Sch. 2 Para 19 | withhold |
| `trade_secrets` | Recital 63 GDPR | redact |
| `crime_prevention` | DPA 2018 Sch. 2 Para 2 | withhold |
| `management_forecasting` | DPA 2018 Sch. 2 Para 22 | withhold |
| `negotiations` | DPA 2018 Sch. 2 Para 24 | withhold |
| `regulatory_function` | DPA 2018 Sch. 2 Para 20 | withhold |

---

## DSARResponseGenerator

Generates compliant DSAR response packages per GDPR Article 15.

### Constructor
```python
DSARResponseGenerator(template_dir=None, organization_name="Organization",
                      dpo_email="dpo@organization.com", controller_name="Data Protection Officer")
```

### Methods

#### `generate_response(dsar_id, data_subject, mapped_data, format="json", request_date=None)`
Generate a complete response package with cover letter, data export, supplementary info, and audit metadata.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `dsar_id` | `str` | required | DSAR reference ID |
| `data_subject` | `str` | required | Name of the data subject |
| `mapped_data` | `dict` | required | Output from DataMapper/ExemptionReviewer |
| `format` | `str` | `"json"` | Export format: `json` or `csv` |
| `request_date` | `str` or `None` | today | Date the request was received |

**Returns:** `dict` with `documents` list containing filename, type, and content for each document.

#### `save_response_package(response, output_dir)`
Save all response documents to disk.

**Returns:** `list[str]` of saved file paths.

---

## DSARWorkflowEngine

Manages the complete DSAR lifecycle: intake, tracking, deadlines, and compliance.

### Constructor
```python
DSARWorkflowEngine(config_path=None)
```

### Methods

#### `register_dsar(requester_name, requester_email, request_channel, request_text, identity_docs=None)`
Register a new DSAR and start the 30-day compliance clock.

**Returns:** `dict` with `dsar_id`, `deadline`, `status`, `identity_verified`.

#### `update_status(dsar_id, new_status, notes="")`
Update DSAR processing status.

**Valid Statuses:** `received`, `identity_verification`, `verification_failed`, `in_progress`, `pii_discovery`, `exemption_review`, `dpo_review`, `response_generation`, `response_sent`, `closed`, `refused`.

#### `apply_extension(dsar_id, reason)`
Apply a 2-month extension for complex requests per Art. 12(3).

#### `pause_clock(dsar_id, reason)`
Pause the response clock (e.g., awaiting identity verification).

#### `days_remaining(dsar_id)`
Calculate remaining days until DSAR deadline. **Returns:** `int`.

#### `get_overdue_dsars()`
Get all DSARs past their deadline. **Returns:** `list[dict]`.

#### `generate_dashboard()`
Generate a DSAR processing dashboard summary. **Returns:** `dict` with status breakdown and overdue info.

---

## DSARAuditLogger

Maintains JSONL audit trails for DSAR processing lifecycle.

### Constructor
```python
DSARAuditLogger(log_path="dsar_audit_logs")
```

### Methods

#### `log_event(dsar_id, event_type, details=None)`
Log a DSAR processing event to the JSONL audit file.

#### `get_audit_trail(dsar_id)`
Retrieve the complete audit trail. **Returns:** `list[dict]`.

#### `generate_compliance_report(dsar_id)`
Generate a compliance report with pass/fail checks for all processing steps.

**Returns:** `dict` with `compliance_checks`, `timeline`, `overall_compliance` (`COMPLIANT` or `REVIEW_REQUIRED`).

---

## CLI Usage

```bash
# Full automated pipeline
python agent.py --action full_pipeline \
    --requester-name "Jane Smith" \
    --requester-email "jane.smith@example.com" \
    --scan-dirs /var/log/app /data/exports \
    --db-connection "postgresql://user:pass@localhost/appdb" \
    --output-dir dsar_output \
    --format json

# Scan text for PII
python agent.py --action scan_pii \
    --scan-text "Contact jane@example.com or call +44 20 7946 0958"

# Scan files only
python agent.py --action scan_files \
    --scan-dirs /data/exports /var/log \
    --requester-email "jane@example.com"

# Generate dashboard
python agent.py --action dashboard
```

### CLI Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `--action` | `full_pipeline` | Action to perform |
| `--requester-name` | `Test Subject` | Data subject name |
| `--requester-email` | `test@example.com` | Data subject email |
| `--request-channel` | `email` | Request channel |
| `--scan-dirs` | `[]` | Directories to scan |
| `--db-connection` | `""` | Database connection string |
| `--output-dir` | `dsar_output` | Output directory |
| `--config` | `dsar_config.json` | Configuration file path |
| `--format` | `json` | Output format (`json` or `csv`) |
| `--min-confidence` | `0.5` | Minimum PII confidence threshold |
| `--scan-text` | `""` | Direct text to scan for PII |

===== END FILE =====


===== FILE: skills/implementing-gdpr-data-subject-access-request/scripts/agent.py =====
#!/usr/bin/env python3
"""
GDPR Data Subject Access Request (DSAR) Workflow Automation Agent.

Implements end-to-end DSAR processing: intake, identity verification, PII discovery
using regex and NER, data mapping to Article 15 categories, exemption review,
response generation, deadline tracking, and audit logging.

References:
    - GDPR Article 15: https://gdpr-info.eu/art-15-gdpr/
    - ICO DSAR Guidance: https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/subject-access-requests/
    - EDPB Guidelines 01/2022 on Right of Access
"""

import os
import re
import json
import uuid
import hashlib
import argparse
import csv
import io
from datetime import datetime, timedelta
from pathlib import Path
from typing import Any

# ---------------------------------------------------------------------------
# PII Regex Patterns -- sourced from Netwrix, PII Crawler, and Varonis
# guidance for EU/UK personal data discovery
# ---------------------------------------------------------------------------

PII_PATTERNS = {
    "email": {
        "pattern": r"\b[A-Za-z0-9._%+\-]+@[A-Za-z0-9.\-]+\.[A-Za-z]{2,}\b",
        "description": "Email address",
        "confidence": 0.95,
        "gdpr_category": "contact_information",
    },
    "phone_international": {
        "pattern": r"(?:\+\d{1,3}[\s\-]?)?\(?\d{2,4}\)?[\s\-]?\d{3,4}[\s\-]?\d{3,4}",
        "description": "Phone number (international format)",
        "confidence": 0.70,
        "gdpr_category": "contact_information",
    },
    "uk_phone": {
        "pattern": r"\b(?:0|\+44[\s\-]?)(?:\d[\s\-]?){9,10}\b",
        "description": "UK phone number",
        "confidence": 0.80,
        "gdpr_category": "contact_information",
    },
    "ssn_us": {
        "pattern": r"\b(?!000|666|9\d{2})\d{3}[\-\s]?(?!00)\d{2}[\-\s]?(?!0000)\d{4}\b",
        "description": "US Social Security Number",
        "confidence": 0.85,
        "gdpr_category": "government_id",
    },
    "nino_uk": {
        "pattern": r"\b[A-CEGHJ-PR-TW-Z]{2}\s?\d{2}\s?\d{2}\s?\d{2}\s?[A-D]\b",
        "description": "UK National Insurance Number",
        "confidence": 0.90,
        "gdpr_category": "government_id",
    },
    "credit_card": {
        "pattern": r"\b(?:4\d{3}|5[1-5]\d{2}|3[47]\d{2}|6(?:011|5\d{2}))"
                   r"[\-\s]?\d{4}[\-\s]?\d{4}[\-\s]?\d{1,4}\b",
        "description": "Credit/debit card number",
        "confidence": 0.85,
        "gdpr_category": "financial_data",
    },
    "iban": {
        "pattern": r"\b[A-Z]{2}\d{2}\s?(?:\d{4}\s?){2,7}\d{1,4}\b",
        "description": "IBAN (International Bank Account Number)",
        "confidence": 0.80,
        "gdpr_category": "financial_data",
    },
    "ipv4": {
        "pattern": r"\b(?:(?:25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)\.){3}"
                   r"(?:25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)\b",
        "description": "IPv4 address",
        "confidence": 0.60,
        "gdpr_category": "online_identifier",
    },
    "date_of_birth": {
        "pattern": r"\b(?:0[1-9]|[12]\d|3[01])[/\-.](?:0[1-9]|1[0-2])[/\-.]"
                   r"(?:19|20)\d{2}\b",
        "description": "Date of birth (DD/MM/YYYY or DD-MM-YYYY)",
        "confidence": 0.65,
        "gdpr_category": "demographic_data",
    },
    "uk_postcode": {
        "pattern": r"\b[A-Z]{1,2}\d[A-Z\d]?\s?\d[A-Z]{2}\b",
        "description": "UK postcode",
        "confidence": 0.75,
        "gdpr_category": "location_data",
    },
    "passport_uk": {
        "pattern": r"\b\d{9}\b",
        "description": "UK passport number (9 digits)",
        "confidence": 0.40,
        "gdpr_category": "government_id",
    },
    "eu_vat": {
        "pattern": r"\b[A-Z]{2}\d{8,12}\b",
        "description": "EU VAT number",
        "confidence": 0.50,
        "gdpr_category": "financial_data",
    },
}

# Compiled patterns for performance
COMPILED_PATTERNS = {
    name: re.compile(info["pattern"], re.IGNORECASE if name in ("email",) else 0)
    for name, info in PII_PATTERNS.items()
}

# ---------------------------------------------------------------------------
# Article 15 response categories -- information that MUST be provided
# ---------------------------------------------------------------------------

ARTICLE_15_CATEGORIES = {
    "processing_purposes": {
        "label": "Purposes of Processing",
        "article_ref": "Art. 15(1)(a)",
        "description": "The purposes for which the personal data are being processed",
    },
    "data_categories": {
        "label": "Categories of Personal Data",
        "article_ref": "Art. 15(1)(b)",
        "description": "The categories of personal data concerned",
    },
    "recipients": {
        "label": "Recipients or Categories of Recipients",
        "article_ref": "Art. 15(1)(c)",
        "description": "Recipients to whom personal data have been or will be disclosed",
    },
    "retention_period": {
        "label": "Retention Period",
        "article_ref": "Art. 15(1)(d)",
        "description": "Envisaged retention period or criteria used to determine it",
    },
    "data_subject_rights": {
        "label": "Data Subject Rights",
        "article_ref": "Art. 15(1)(e-f)",
        "description": "Right to rectification, erasure, restriction, objection, and complaint",
    },
    "data_source": {
        "label": "Source of Data",
        "article_ref": "Art. 15(1)(g)",
        "description": "Where data was not collected from the subject, available source info",
    },
    "automated_decisions": {
        "label": "Automated Decision-Making",
        "article_ref": "Art. 15(1)(h)",
        "description": "Existence of automated decision-making including profiling",
    },
    "international_transfers": {
        "label": "International Transfers",
        "article_ref": "Art. 15(2)",
        "description": "Appropriate safeguards for transfers to third countries",
    },
}

# ---------------------------------------------------------------------------
# DSAR exemption types per GDPR/UK GDPR
# ---------------------------------------------------------------------------

EXEMPTION_TYPES = {
    "third_party_data": {
        "label": "Third-Party Personal Data",
        "description": "Data relating to another identifiable individual",
        "legal_basis": "Art. 15(4) / DPA 2018 Sch. 2 Para 16",
        "action": "redact",
    },
    "legal_professional_privilege": {
        "label": "Legal Professional Privilege",
        "description": "Communications subject to legal privilege",
        "legal_basis": "DPA 2018 Sch. 2 Para 19",
        "action": "withhold",
    },
    "trade_secrets": {
        "label": "Trade Secrets / Confidential Info",
        "description": "Trade secrets or intellectual property",
        "legal_basis": "Recital 63 GDPR",
        "action": "redact",
    },
    "crime_prevention": {
        "label": "Crime Prevention / Detection",
        "description": "Data processed for crime prevention purposes",
        "legal_basis": "DPA 2018 Sch. 2 Para 2",
        "action": "withhold",
    },
    "management_forecasting": {
        "label": "Management Forecasting / Planning",
        "description": "Data processed for management planning that would prejudice business",
        "legal_basis": "DPA 2018 Sch. 2 Para 22",
        "action": "withhold",
    },
    "negotiations": {
        "label": "Negotiations",
        "description": "Data that would prejudice negotiations with the data subject",
        "legal_basis": "DPA 2018 Sch. 2 Para 24",
        "action": "withhold",
    },
    "regulatory_function": {
        "label": "Regulatory Functions",
        "description": "Data processed for regulatory purposes",
        "legal_basis": "DPA 2018 Sch. 2 Para 20",
        "action": "withhold",
    },
}


# ===========================================================================
# PII Pattern Matcher
# ===========================================================================

class PIIPatternMatcher:
    """Scans text for PII using compiled regex patterns with confidence scoring."""

    def __init__(self, custom_patterns=None):
        self.patterns = dict(COMPILED_PATTERNS)
        self.pattern_info = dict(PII_PATTERNS)
        if custom_patterns:
            for name, spec in custom_patterns.items():
                self.patterns[name] = re.compile(spec["pattern"])
                self.pattern_info[name] = spec

    def scan_text(self, text: str, min_confidence: float = 0.5) -> list[dict]:
        """Scan text for PII matches with confidence scoring."""
        matches = []
        for name, compiled in self.patterns.items():
            info = self.pattern_info[name]
            if info.get("confidence", 1.0) < min_confidence:
                continue
            for m in compiled.finditer(text):
                value = m.group().strip()
                if len(value) < 3:
                    continue
                confidence = info.get("confidence", 0.5)
                # Boost confidence if contextual keywords are nearby
                context_start = max(0, m.start() - 50)
                context_end = min(len(text), m.end() + 50)
                context = text[context_start:context_end].lower()
                context_keywords = {
                    "email": ["email", "e-mail", "contact", "address"],
                    "phone_international": ["phone", "tel", "mobile", "call"],
                    "uk_phone": ["phone", "tel", "mobile", "call"],
                    "ssn_us": ["ssn", "social security", "tax id"],
                    "nino_uk": ["nino", "national insurance", "ni number"],
                    "credit_card": ["card", "visa", "mastercard", "payment"],
                    "iban": ["iban", "bank", "account"],
                    "date_of_birth": ["dob", "birth", "born", "age"],
                    "uk_postcode": ["postcode", "post code", "address", "zip"],
                }
                if name in context_keywords:
                    for kw in context_keywords[name]:
                        if kw in context:
                            confidence = min(1.0, confidence + 0.15)
                            break

                matches.append({
                    "type": name,
                    "value": value,
                    "description": info["description"],
                    "confidence": round(confidence, 2),
                    "gdpr_category": info.get("gdpr_category", "unknown"),
                    "position": {"start": m.start(), "end": m.end()},
                })
        return matches

    def scan_file(self, file_path: str, min_confidence: float = 0.5) -> dict:
        """Scan a file for PII matches."""
        path = Path(file_path)
        if not path.exists():
            return {"file": file_path, "error": "File not found", "matches": []}
        try:
            text = path.read_text(encoding="utf-8", errors="replace")
        except Exception as e:
            return {"file": file_path, "error": str(e), "matches": []}
        matches = self.scan_text(text, min_confidence)
        return {
            "file": file_path,
            "size_bytes": path.stat().st_size,
            "matches": matches,
            "match_count": len(matches),
            "pii_types_found": list({m["type"] for m in matches}),
        }


# ===========================================================================
# PII Discovery Engine
# ===========================================================================

class PIIDiscoveryEngine:
    """Discovers PII across structured (database) and unstructured (files) data sources."""

    def __init__(self, custom_patterns=None):
        self.matcher = PIIPatternMatcher(custom_patterns)
        self.results = []

    def scan_database(self, connection_string: str,
                      search_identifiers: dict,
                      tables: list[str] | None = None) -> dict:
        """
        Scan a database for records matching search identifiers.

        In production, this connects via SQLAlchemy/psycopg2. This implementation
        generates the parameterized queries needed for discovery.
        """
        queries = []
        if not tables:
            tables = [
                "users", "customers", "orders", "contacts", "employees",
                "audit_log", "login_history", "consent_records",
                "communication_preferences", "support_tickets",
            ]

        safe_table_re = re.compile(r"^[a-zA-Z_][a-zA-Z0-9_.]*$")

        for table in tables:
            if not safe_table_re.match(table):
                continue
            for field, value in search_identifiers.items():
                if not safe_table_re.match(field):
                    continue
                queries.append({
                    "table": table,
                    "query": f"SELECT * FROM [{table}] WHERE [{field}] = ?",
                    "params": [value],
                    "search_field": field,
                    "search_value": value,
                })

        # Full-text search query for unstructured columns
        for table in tables:
            if not safe_table_re.match(table):
                continue
            for identifier_value in search_identifiers.values():
                queries.append({
                    "table": table,
                    "query": f"SELECT * FROM [{table}] WHERE CAST(* AS TEXT) LIKE ?",
                    "params": [f"%{identifier_value}%"],
                    "search_type": "full_text",
                })

        result = {
            "source_type": "database",
            "connection": _redact_connection_string(connection_string),
            "tables_scanned": len(tables),
            "queries_generated": len(queries),
            "queries": queries,
            "scan_timestamp": datetime.utcnow().isoformat(),
        }
        self.results.append(result)
        return result

    def scan_files(self, directories: list[str],
                   search_identifiers: dict,
                   file_extensions: list[str] | None = None,
                   max_file_size_mb: int = 50) -> dict:
        """Scan files in directories for PII matching search identifiers."""
        if not file_extensions:
            file_extensions = [
                ".txt", ".csv", ".json", ".xml", ".log", ".html",
                ".md", ".yaml", ".yml", ".ini", ".conf", ".cfg",
            ]

        scanned_files = []
        matches_found = []
        errors = []
        max_bytes = max_file_size_mb * 1024 * 1024

        for directory in directories:
            dir_path = Path(directory)
            if not dir_path.exists():
                errors.append({"directory": directory, "error": "Directory not found"})
                continue
            for ext in file_extensions:
                for file_path in dir_path.rglob(f"*{ext}"):
                    if file_path.stat().st_size > max_bytes:
                        continue
                    try:
                        text = file_path.read_text(encoding="utf-8", errors="replace")
                    except Exception as e:
                        errors.append({"file": str(file_path), "error": str(e)})
                        continue

                    scanned_files.append(str(file_path))

                    # Check for identifier matches
                    for id_type, id_value in search_identifiers.items():
                        if id_value.lower() in text.lower():
                            # Run full PII scan on matching files
                            pii_matches = self.matcher.scan_text(text)
                            matches_found.append({
                                "file": str(file_path),
                                "matched_identifier": id_type,
                                "pii_matches": pii_matches,
                            })
                            break

        result = {
            "source_type": "files",
            "directories_scanned": len(directories),
            "files_scanned": len(scanned_files),
            "files_with_matches": len(matches_found),
            "matches": matches_found,
            "errors": errors,
            "raw_text_matches": [m["file"] for m in matches_found],
            "scan_timestamp": datetime.utcnow().isoformat(),
        }
        self.results.append(result)
        return result

    def scan_with_ner(self, text_corpus: list[str],
                      entity_types: list[str] | None = None,
                      confidence_threshold: float = 0.7) -> dict:
        """
        Scan text using Named Entity Recognition for contextual PII detection.

        Uses spaCy NER model when available, falls back to regex+context heuristics.
        Entity types: PERSON, EMAIL, PHONE_NUMBER, LOCATION, DATE_OF_BIRTH,
                      ORG, GPE, NORP, CARDINAL
        """
        if not entity_types:
            entity_types = [
                "PERSON", "EMAIL", "PHONE_NUMBER", "LOCATION",
                "DATE_OF_BIRTH", "ORG", "GPE",
            ]

        ner_results = []
        nlp = None

        # Attempt to load spaCy model
        try:
            import spacy
            try:
                nlp = spacy.load("en_core_web_lg")
            except OSError:
                try:
                    nlp = spacy.load("en_core_web_sm")
                except OSError:
                    nlp = None
        except ImportError:
            nlp = None

        for file_path in text_corpus:
            path = Path(file_path)
            if not path.exists():
                continue
            try:
                text = path.read_text(encoding="utf-8", errors="replace")
            except Exception:
                continue

            entities_found = []

            if nlp is not None:
                # Use spaCy NER
                doc = nlp(text[:100000])  # Limit to 100k chars for performance
                for ent in doc.ents:
                    if ent.label_ in entity_types:
                        entities_found.append({
                            "text": ent.text,
                            "label": ent.label_,
                            "start": ent.start_char,
                            "end": ent.end_char,
                            "confidence": round(0.7 + (0.3 if ent.label_ in ("PERSON", "ORG") else 0.1), 2),
                            "method": "spacy_ner",
                        })
            else:
                # Fallback: regex + context heuristics
                regex_matches = self.matcher.scan_text(text, min_confidence=confidence_threshold)
                for m in regex_matches:
                    ner_label = _map_pii_type_to_ner(m["type"])
                    if ner_label in entity_types:
                        entities_found.append({
                            "text": m["value"],
                            "label": ner_label,
                            "start": m["position"]["start"],
                            "end": m["position"]["end"],
                            "confidence": m["confidence"],
                            "method": "regex_heuristic",
                        })

                # Name detection heuristic (Title Case sequences near person-keywords)
                if "PERSON" in entity_types:
                    name_pattern = re.compile(
                        r"(?:(?:name|customer|employee|patient|client|user|requester|subject)"
                        r"[\s:=]+)([A-Z][a-z]+(?:\s+[A-Z][a-z]+){1,3})",
                        re.MULTILINE,
                    )
                    for m in name_pattern.finditer(text):
                        entities_found.append({
                            "text": m.group(1),
                            "label": "PERSON",
                            "start": m.start(1),
                            "end": m.end(1),
                            "confidence": 0.75,
                            "method": "context_heuristic",
                        })

            ner_results.append({
                "file": str(file_path),
                "entities": entities_found,
                "entity_count": len(entities_found),
            })

        return {
            "source_type": "ner",
            "files_processed": len(ner_results),
            "total_entities": sum(r["entity_count"] for r in ner_results),
            "results": ner_results,
            "model_used": "spacy" if nlp else "regex_heuristic",
            "entity_types_requested": entity_types,
            "scan_timestamp": datetime.utcnow().isoformat(),
        }

    def consolidate_results(self, *result_sets) -> dict:
        """Consolidate PII discovery results from multiple sources."""
        all_records = []
        sources = set()

        for result in result_sets:
            if not result:
                continue
            source_type = result.get("source_type", "unknown")
            sources.add(source_type)

            if source_type == "database":
                for query in result.get("queries", []):
                    all_records.append({
                        "source": f"database:{query['table']}",
                        "type": "structured",
                        "details": query,
                    })

            elif source_type == "files":
                for match in result.get("matches", []):
                    for pii in match.get("pii_matches", []):
                        all_records.append({
                            "source": f"file:{match['file']}",
                            "type": "unstructured",
                            "pii_type": pii["type"],
                            "value_hash": hashlib.sha256(
                                pii["value"].encode()
                            ).hexdigest()[:16],
                            "confidence": pii["confidence"],
                            "gdpr_category": pii["gdpr_category"],
                        })

            elif source_type == "ner":
                for file_result in result.get("results", []):
                    for entity in file_result.get("entities", []):
                        all_records.append({
                            "source": f"ner:{file_result['file']}",
                            "type": "ner_entity",
                            "entity_label": entity["label"],
                            "value_hash": hashlib.sha256(
                                entity["text"].encode()
                            ).hexdigest()[:16],
                            "confidence": entity["confidence"],
                        })

        return {
            "total_records": len(all_records),
            "source_count": len(sources),
            "sources": list(sources),
            "records": all_records,
            "consolidated_at": datetime.utcnow().isoformat(),
        }

    def full_scan(self, search_identifiers: dict,
                  sources: list[str] | None = None,
                  db_connection: str = "",
                  directories: list[str] | None = None) -> dict:
        """Run a complete PII discovery scan across all source types."""
        if sources is None:
            sources = ["database", "files"]
        if directories is None:
            directories = []

        results = []

        if "database" in sources and db_connection:
            results.append(self.scan_database(db_connection, search_identifiers))

        if "files" in sources and directories:
            results.append(self.scan_files(directories, search_identifiers))

        if "ner" in sources:
            # Gather text files from file scan
            text_files = []
            for r in results:
                text_files.extend(r.get("raw_text_matches", []))
            if text_files:
                results.append(self.scan_with_ner(text_files))

        return self.consolidate_results(*results)


# ===========================================================================
# Data Mapper -- maps PII to Article 15 categories
# ===========================================================================

class DataMapper:
    """Maps discovered PII to GDPR Article 15 disclosure categories."""

    def __init__(self, data_inventory_path: str | None = None):
        self.inventory = {}
        if data_inventory_path and Path(data_inventory_path).exists():
            with open(data_inventory_path) as f:
                self.inventory = json.load(f)

    def map_to_article15(self, pii_records: dict,
                         data_subject_id: str) -> dict:
        """Map PII records to Article 15 required categories."""
        categories = []
        gdpr_categories_found = set()

        for record in pii_records.get("records", []):
            cat = record.get("gdpr_category") or record.get("entity_label", "unknown")
            gdpr_categories_found.add(cat)

        # Build category mappings
        category_mapping = {
            "contact_information": {
                "name": "Contact Information",
                "processing_purpose": "Account management, communication, service delivery",
                "legal_basis": "Art. 6(1)(b) - Contract performance",
                "retention_period": "Duration of account + 6 years post-closure",
                "recipients": ["Internal customer service", "Email service provider"],
                "data_types": ["Email address", "Phone number", "Postal address"],
            },
            "government_id": {
                "name": "Government-Issued Identification",
                "processing_purpose": "Identity verification, regulatory compliance (KYC/AML)",
                "legal_basis": "Art. 6(1)(c) - Legal obligation",
                "retention_period": "5 years after last verification event",
                "recipients": ["Compliance team", "Identity verification provider"],
                "data_types": ["National Insurance Number", "Passport number", "SSN"],
            },
            "financial_data": {
                "name": "Financial Information",
                "processing_purpose": "Payment processing, billing, fraud prevention",
                "legal_basis": "Art. 6(1)(b) - Contract performance",
                "retention_period": "7 years for tax compliance",
                "recipients": ["Payment processor", "Finance department", "Tax authority"],
                "data_types": ["Credit card number (tokenized)", "IBAN", "Transaction records"],
            },
            "online_identifier": {
                "name": "Online Identifiers",
                "processing_purpose": "Security monitoring, service analytics",
                "legal_basis": "Art. 6(1)(f) - Legitimate interest (security)",
                "retention_period": "90 days for logs, 2 years for analytics",
                "recipients": ["IT security team", "Analytics platform"],
                "data_types": ["IP address", "Cookie ID", "Device fingerprint"],
            },
            "demographic_data": {
                "name": "Demographic Data",
                "processing_purpose": "Service personalization, age verification",
                "legal_basis": "Art. 6(1)(a) - Consent / Art. 6(1)(b) - Contract",
                "retention_period": "Duration of account relationship",
                "recipients": ["Marketing team (with consent)", "Analytics"],
                "data_types": ["Date of birth", "Gender", "Language preference"],
            },
            "location_data": {
                "name": "Location Data",
                "processing_purpose": "Service delivery, address verification",
                "legal_basis": "Art. 6(1)(b) - Contract performance",
                "retention_period": "Duration of account + 2 years",
                "recipients": ["Delivery partner", "Address verification service"],
                "data_types": ["Postal code", "City", "Country"],
            },
        }

        # Override with data inventory if available
        if self.inventory:
            for cat_key, inv_data in self.inventory.items():
                if cat_key in category_mapping:
                    category_mapping[cat_key].update(inv_data)

        for cat in gdpr_categories_found:
            if cat in category_mapping:
                mapping = category_mapping[cat]
                categories.append(mapping)
            else:
                categories.append({
                    "name": cat.replace("_", " ").title(),
                    "processing_purpose": "See data processing register for details",
                    "legal_basis": "Determined per processing activity",
                    "retention_period": "Per retention schedule",
                    "recipients": ["See recipient register"],
                    "data_types": [cat],
                })

        # Add standard Article 15 supplementary information
        supplementary = {
            "data_subject_rights": {
                "right_to_rectification": "Art. 16 - Right to rectification of inaccurate data",
                "right_to_erasure": "Art. 17 - Right to erasure ('right to be forgotten')",
                "right_to_restriction": "Art. 18 - Right to restriction of processing",
                "right_to_data_portability": "Art. 20 - Right to data portability",
                "right_to_object": "Art. 21 - Right to object to processing",
                "right_to_complaint": "Right to lodge a complaint with the ICO (ico.org.uk) "
                                     "or relevant supervisory authority",
            },
            "automated_decision_making": {
                "exists": False,
                "description": "No automated decision-making or profiling with legal/significant effect",
                "note": "Update based on actual processing activities",
            },
            "international_transfers": {
                "transfers_exist": False,
                "safeguards": "Standard Contractual Clauses (SCCs) where applicable",
                "countries": [],
            },
        }

        return {
            "data_subject": data_subject_id,
            "categories": categories,
            "supplementary_info": supplementary,
            "article_15_reference": ARTICLE_15_CATEGORIES,
            "mapped_at": datetime.utcnow().isoformat(),
        }


# ===========================================================================
# Exemption Reviewer
# ===========================================================================

class ExemptionReviewer:
    """Reviews DSAR data against applicable GDPR/UK GDPR exemptions."""

    def __init__(self):
        self.exemption_types = EXEMPTION_TYPES

    def review_exemptions(self, mapped_data: dict,
                          exemption_checks: list[str] | None = None) -> dict:
        """Review mapped data for applicable exemptions."""
        if not exemption_checks:
            exemption_checks = list(self.exemption_types.keys())

        applicable_exemptions = []

        for check in exemption_checks:
            if check not in self.exemption_types:
                continue

            exemption_info = self.exemption_types[check]
            # Each exemption requires manual DPO review; we flag candidates
            applicable_exemptions.append({
                "exemption_type": check,
                "label": exemption_info["label"],
                "legal_basis": exemption_info["legal_basis"],
                "action": exemption_info["action"],
                "status": "pending_review",
                "dpo_review_required": True,
                "notes": f"Flagged for DPO review: {exemption_info['description']}",
            })

        return {
            "exemption_count": len(applicable_exemptions),
            "exemptions": applicable_exemptions,
            "review_status": "pending_dpo_approval",
            "reviewed_at": datetime.utcnow().isoformat(),
        }

    def apply_redactions(self, mapped_data: dict,
                         approved_exemptions: list[dict]) -> dict:
        """Apply approved exemption redactions to mapped data."""
        redacted = json.loads(json.dumps(mapped_data))

        redaction_log = []
        for exemption in approved_exemptions:
            if exemption.get("status") != "approved":
                continue
            action = exemption.get("action", "redact")
            redaction_log.append({
                "exemption_type": exemption["exemption_type"],
                "action_taken": action,
                "legal_basis": exemption["legal_basis"],
                "applied_at": datetime.utcnow().isoformat(),
            })

        redacted["redaction_log"] = redaction_log
        redacted["redactions_applied"] = len(redaction_log)
        return redacted


# ===========================================================================
# DSAR Response Generator
# ===========================================================================

class DSARResponseGenerator:
    """Generates compliant DSAR response packages per GDPR Article 15."""

    COVER_LETTER_TEMPLATE = """
DATA SUBJECT ACCESS REQUEST RESPONSE
=====================================

Date: {response_date}
DSAR Reference: {dsar_id}

Dear {data_subject},

Thank you for your data subject access request received on {request_date}.

In accordance with Article 15 of the General Data Protection Regulation (GDPR),
we are writing to confirm that we do process your personal data. Please find
enclosed:

1. A copy of all personal data we hold about you
2. Supplementary information as required under Article 15(1)

SUPPLEMENTARY INFORMATION
--------------------------

Purposes of Processing:
{processing_purposes}

Categories of Personal Data:
{data_categories}

Recipients:
{recipients}

Retention Periods:
{retention_periods}

Data Source:
{data_source}

Your Rights:
You have the right to:
- Request rectification of inaccurate personal data (Art. 16)
- Request erasure of your personal data (Art. 17)
- Request restriction of processing (Art. 18)
- Receive your data in a portable format (Art. 20)
- Object to processing based on legitimate interest (Art. 21)
- Lodge a complaint with the Information Commissioner's Office (ico.org.uk)

Automated Decision-Making:
{automated_decisions}

International Transfers:
{international_transfers}

If you have any questions about this response, please contact our Data
Protection Officer at {dpo_email}.

Yours sincerely,
{controller_name}
Data Protection Officer
{organization_name}
"""

    def __init__(self, template_dir: str | None = None,
                 organization_name: str = "Organization",
                 dpo_email: str = "dpo@organization.com",
                 controller_name: str = "Data Protection Officer"):
        self.template_dir = template_dir
        self.organization_name = organization_name
        self.dpo_email = dpo_email
        self.controller_name = controller_name

    def generate_response(self, dsar_id: str, data_subject: str,
                          mapped_data: dict, format: str = "json",
                          request_date: str | None = None) -> dict:
        """Generate a complete DSAR response package."""
        if not request_date:
            request_date = datetime.utcnow().strftime("%Y-%m-%d")

        documents = []

        # 1. Cover letter with supplementary information
        cover_letter = self._generate_cover_letter(
            dsar_id, data_subject, mapped_data, request_date
        )
        documents.append({
            "filename": f"DSAR_{dsar_id}_cover_letter.txt",
            "type": "cover_letter",
            "content": cover_letter,
        })

        # 2. Personal data export
        data_export = self._generate_data_export(dsar_id, mapped_data, format)
        ext = "json" if format == "json" else "csv"
        documents.append({
            "filename": f"DSAR_{dsar_id}_personal_data.{ext}",
            "type": "data_export",
            "content": data_export,
        })

        # 3. Supplementary information document
        supp_doc = self._generate_supplementary_doc(dsar_id, mapped_data)
        documents.append({
            "filename": f"DSAR_{dsar_id}_supplementary_info.json",
            "type": "supplementary_information",
            "content": supp_doc,
        })

        # 4. Audit metadata
        audit_meta = {
            "dsar_id": dsar_id,
            "data_subject": data_subject,
            "response_generated_at": datetime.utcnow().isoformat(),
            "documents_generated": len(documents),
            "format": format,
            "exemptions_applied": mapped_data.get("redactions_applied", 0),
        }
        documents.append({
            "filename": f"DSAR_{dsar_id}_audit_metadata.json",
            "type": "audit_metadata",
            "content": json.dumps(audit_meta, indent=2),
        })

        return {
            "dsar_id": dsar_id,
            "documents": documents,
            "generated_at": datetime.utcnow().isoformat(),
        }

    def _generate_cover_letter(self, dsar_id: str, data_subject: str,
                               mapped_data: dict, request_date: str) -> str:
        """Generate the DSAR cover letter."""
        categories = mapped_data.get("categories", [])
        supplementary = mapped_data.get("supplementary_info", {})

        processing_purposes = "\n".join(
            f"  - {cat['name']}: {cat['processing_purpose']}"
            for cat in categories
        ) or "  No personal data processing identified."

        data_categories_text = "\n".join(
            f"  - {cat['name']}: {', '.join(cat.get('data_types', []))}"
            for cat in categories
        ) or "  No categories identified."

        recipients_text = "\n".join(
            f"  - {cat['name']}: {', '.join(cat.get('recipients', []))}"
            for cat in categories
        ) or "  No third-party recipients."

        retention_text = "\n".join(
            f"  - {cat['name']}: {cat.get('retention_period', 'Per retention schedule')}"
            for cat in categories
        ) or "  Per organizational retention schedule."

        auto_decisions = supplementary.get("automated_decision_making", {})
        auto_text = auto_decisions.get(
            "description",
            "No automated decision-making or profiling applies."
        )

        transfers = supplementary.get("international_transfers", {})
        transfer_text = (
            f"Transfers to: {', '.join(transfers['countries'])}. "
            f"Safeguards: {transfers.get('safeguards', 'N/A')}"
            if transfers.get("transfers_exist")
            else "No international transfers of your personal data."
        )

        return self.COVER_LETTER_TEMPLATE.format(
            response_date=datetime.utcnow().strftime("%d %B %Y"),
            dsar_id=dsar_id,
            data_subject=data_subject,
            request_date=request_date,
            processing_purposes=processing_purposes,
            data_categories=data_categories_text,
            recipients=recipients_text,
            retention_periods=retention_text,
            data_source="Data collected directly from you unless otherwise stated.",
            automated_decisions=auto_text,
            international_transfers=transfer_text,
            dpo_email=self.dpo_email,
            controller_name=self.controller_name,
            organization_name=self.organization_name,
        )

    def _generate_data_export(self, dsar_id: str, mapped_data: dict,
                              format: str) -> str:
        """Generate the personal data export in requested format."""
        export_data = {
            "dsar_reference": dsar_id,
            "export_date": datetime.utcnow().isoformat(),
            "categories": [],
        }

        for cat in mapped_data.get("categories", []):
            export_data["categories"].append({
                "category": cat["name"],
                "data_types": cat.get("data_types", []),
                "processing_purpose": cat["processing_purpose"],
                "legal_basis": cat.get("legal_basis", ""),
            })

        if format == "csv":
            output = io.StringIO()
            writer = csv.writer(output)
            writer.writerow([
                "Category", "Data Types", "Processing Purpose", "Legal Basis",
            ])
            for cat in export_data["categories"]:
                writer.writerow([
                    cat["category"],
                    "; ".join(cat["data_types"]),
                    cat["processing_purpose"],
                    cat["legal_basis"],
                ])
            return output.getvalue()

        return json.dumps(export_data, indent=2)

    def _generate_supplementary_doc(self, dsar_id: str,
                                    mapped_data: dict) -> str:
        """Generate the Article 15 supplementary information document."""
        doc = {
            "dsar_reference": dsar_id,
            "article_15_compliance": {},
        }

        for key, cat_info in ARTICLE_15_CATEGORIES.items():
            doc["article_15_compliance"][key] = {
                "article_reference": cat_info["article_ref"],
                "label": cat_info["label"],
                "description": cat_info["description"],
                "provided": True,
            }

        doc["supplementary_info"] = mapped_data.get("supplementary_info", {})
        doc["redaction_log"] = mapped_data.get("redaction_log", [])

        return json.dumps(doc, indent=2)

    def save_response_package(self, response: dict, output_dir: str) -> list[str]:
        """Save all response documents to disk."""
        out_path = Path(output_dir)
        out_path.mkdir(parents=True, exist_ok=True)
        saved = []
        for doc in response.get("documents", []):
            file_path = out_path / doc["filename"]
            file_path.write_text(doc["content"], encoding="utf-8")
            saved.append(str(file_path))
        return saved


# ===========================================================================
# DSAR Workflow Engine -- orchestrates the full lifecycle
# ===========================================================================

class DSARWorkflowEngine:
    """Manages the complete DSAR lifecycle: intake, tracking, and compliance."""

    VALID_STATUSES = [
        "received", "identity_verification", "verification_failed",
        "in_progress", "pii_discovery", "exemption_review",
        "dpo_review", "response_generation", "response_sent",
        "closed", "refused",
    ]

    def __init__(self, config_path: str | None = None):
        self.config = {}
        if config_path and Path(config_path).exists():
            with open(config_path) as f:
                self.config = json.load(f)
        self.dsars: dict[str, dict] = {}

    def register_dsar(self, requester_name: str, requester_email: str,
                      request_channel: str, request_text: str,
                      identity_docs: list[str] | None = None) -> dict:
        """Register a new DSAR and start the compliance clock."""
        dsar_id = f"DSAR-{datetime.utcnow().strftime('%Y%m%d')}-{uuid.uuid4().hex[:8].upper()}"
        received_at = datetime.utcnow()
        deadline = received_at + timedelta(days=30)

        identity_verified = bool(identity_docs and len(identity_docs) > 0)

        dsar = {
            "dsar_id": dsar_id,
            "requester_name": requester_name,
            "requester_email": requester_email,
            "request_channel": request_channel,
            "request_text": request_text,
            "received_at": received_at.isoformat(),
            "deadline": deadline.isoformat(),
            "deadline_date": deadline.strftime("%Y-%m-%d"),
            "identity_verified": identity_verified,
            "identity_docs": identity_docs or [],
            "status": "received" if identity_verified else "identity_verification",
            "status_history": [
                {
                    "status": "received",
                    "timestamp": received_at.isoformat(),
                    "notes": f"Request received via {request_channel}",
                }
            ],
            "clock_paused": False,
            "extension_applied": False,
        }

        self.dsars[dsar_id] = dsar
        return dsar

    def update_status(self, dsar_id: str, new_status: str,
                      notes: str = "") -> dict:
        """Update DSAR processing status."""
        if dsar_id not in self.dsars:
            raise ValueError(f"DSAR not found: {dsar_id}")
        if new_status not in self.VALID_STATUSES:
            raise ValueError(f"Invalid status: {new_status}")

        dsar = self.dsars[dsar_id]
        dsar["status"] = new_status
        dsar["status_history"].append({
            "status": new_status,
            "timestamp": datetime.utcnow().isoformat(),
            "notes": notes,
        })
        return dsar

    def apply_extension(self, dsar_id: str, reason: str) -> dict:
        """Apply a 2-month extension for complex requests (Art. 12(3))."""
        if dsar_id not in self.dsars:
            raise ValueError(f"DSAR not found: {dsar_id}")

        dsar = self.dsars[dsar_id]
        if dsar["extension_applied"]:
            raise ValueError("Extension already applied to this DSAR")

        original_deadline = datetime.fromisoformat(dsar["deadline"])
        new_deadline = original_deadline + timedelta(days=60)

        dsar["deadline"] = new_deadline.isoformat()
        dsar["deadline_date"] = new_deadline.strftime("%Y-%m-%d")
        dsar["extension_applied"] = True
        dsar["extension_reason"] = reason
        dsar["status_history"].append({
            "status": "extension_applied",
            "timestamp": datetime.utcnow().isoformat(),
            "notes": f"2-month extension: {reason}",
        })
        return dsar

    def pause_clock(self, dsar_id: str, reason: str) -> dict:
        """Pause the response clock (e.g., awaiting identity verification)."""
        if dsar_id not in self.dsars:
            raise ValueError(f"DSAR not found: {dsar_id}")

        dsar = self.dsars[dsar_id]
        dsar["clock_paused"] = True
        dsar["clock_paused_at"] = datetime.utcnow().isoformat()
        dsar["clock_pause_reason"] = reason
        dsar["status_history"].append({
            "status": "clock_paused",
            "timestamp": datetime.utcnow().isoformat(),
            "notes": f"Clock paused: {reason}",
        })
        return dsar

    def days_remaining(self, dsar_id: str) -> int:
        """Calculate remaining days until DSAR deadline."""
        if dsar_id not in self.dsars:
            raise ValueError(f"DSAR not found: {dsar_id}")

        dsar = self.dsars[dsar_id]
        deadline = datetime.fromisoformat(dsar["deadline"])
        remaining = (deadline - datetime.utcnow()).days
        return max(0, remaining)

    def get_overdue_dsars(self) -> list[dict]:
        """Get all DSARs that are past their deadline."""
        overdue = []
        now = datetime.utcnow()
        for dsar in self.dsars.values():
            if dsar["status"] in ("closed", "refused", "response_sent"):
                continue
            deadline = datetime.fromisoformat(dsar["deadline"])
            if now > deadline:
                overdue.append({
                    "dsar_id": dsar["dsar_id"],
                    "requester": dsar["requester_name"],
                    "deadline": dsar["deadline_date"],
                    "days_overdue": (now - deadline).days,
                    "status": dsar["status"],
                })
        return overdue

    def generate_dashboard(self) -> dict:
        """Generate a DSAR processing dashboard summary."""
        total = len(self.dsars)
        statuses = {}
        for dsar in self.dsars.values():
            status = dsar["status"]
            statuses[status] = statuses.get(status, 0) + 1

        overdue = self.get_overdue_dsars()

        return {
            "total_dsars": total,
            "status_breakdown": statuses,
            "overdue_count": len(overdue),
            "overdue_dsars": overdue,
            "generated_at": datetime.utcnow().isoformat(),
        }


# ===========================================================================
# DSAR Audit Logger
# ===========================================================================

class DSARAuditLogger:
    """Maintains audit trails for DSAR processing lifecycle."""

    def __init__(self, log_path: str = "dsar_audit_logs"):
        self.log_path = Path(log_path)
        self.log_path.mkdir(parents=True, exist_ok=True)

    def log_event(self, dsar_id: str, event_type: str,
                  details: dict | None = None) -> dict:
        """Log a DSAR processing event."""
        event = {
            "dsar_id": dsar_id,
            "event_type": event_type,
            "timestamp": datetime.utcnow().isoformat(),
            "details": details or {},
            "event_id": uuid.uuid4().hex[:12],
        }

        log_file = self.log_path / f"{dsar_id}.jsonl"
        with open(log_file, "a") as f:
            f.write(json.dumps(event) + "\n")

        return event

    def get_audit_trail(self, dsar_id: str) -> list[dict]:
        """Retrieve the complete audit trail for a DSAR."""
        log_file = self.log_path / f"{dsar_id}.jsonl"
        if not log_file.exists():
            return []
        events = []
        with open(log_file) as f:
            for line in f:
                line = line.strip()
                if line:
                    events.append(json.loads(line))
        return events

    def generate_compliance_report(self, dsar_id: str) -> dict:
        """Generate a compliance report for a DSAR showing all processing steps."""
        events = self.get_audit_trail(dsar_id)

        report = {
            "dsar_id": dsar_id,
            "report_generated_at": datetime.utcnow().isoformat(),
            "total_events": len(events),
            "event_types": list({e["event_type"] for e in events}),
            "timeline": [],
            "compliance_checks": {
                "request_acknowledged": False,
                "identity_verified": False,
                "pii_discovery_complete": False,
                "exemption_review_complete": False,
                "response_generated": False,
                "response_sent": False,
                "within_deadline": False,
            },
        }

        for event in events:
            report["timeline"].append({
                "timestamp": event["timestamp"],
                "event": event["event_type"],
                "details": event.get("details", {}),
            })

            etype = event["event_type"]
            if etype == "request_received":
                report["compliance_checks"]["request_acknowledged"] = True
            elif etype == "identity_verified":
                report["compliance_checks"]["identity_verified"] = True
            elif etype == "pii_discovery_complete":
                report["compliance_checks"]["pii_discovery_complete"] = True
            elif etype == "exemption_review_complete":
                report["compliance_checks"]["exemption_review_complete"] = True
            elif etype == "response_generated":
                report["compliance_checks"]["response_generated"] = True
            elif etype == "response_sent":
                report["compliance_checks"]["response_sent"] = True
                report["compliance_checks"]["within_deadline"] = True

        all_passed = all(report["compliance_checks"].values())
        report["overall_compliance"] = "COMPLIANT" if all_passed else "REVIEW_REQUIRED"

        return report


# ===========================================================================
# Utility functions
# ===========================================================================

def _redact_connection_string(conn_str: str) -> str:
    """Redact passwords from connection strings for logging."""
    return re.sub(r"://([^:]+):([^@]+)@", r"://\1:****@", conn_str)


def _map_pii_type_to_ner(pii_type: str) -> str:
    """Map PII regex type names to NER entity labels."""
    mapping = {
        "email": "EMAIL",
        "phone_international": "PHONE_NUMBER",
        "uk_phone": "PHONE_NUMBER",
        "ssn_us": "GOVERNMENT_ID",
        "nino_uk": "GOVERNMENT_ID",
        "credit_card": "FINANCIAL",
        "iban": "FINANCIAL",
        "ipv4": "ONLINE_ID",
        "date_of_birth": "DATE_OF_BIRTH",
        "uk_postcode": "LOCATION",
        "passport_uk": "GOVERNMENT_ID",
        "eu_vat": "FINANCIAL",
    }
    return mapping.get(pii_type, "UNKNOWN")


# ===========================================================================
# CLI Entry Point
# ===========================================================================

def main():
    parser = argparse.ArgumentParser(
        description="GDPR DSAR Workflow Automation Agent"
    )
    parser.add_argument(
        "--action",
        choices=[
            "register", "scan_pii", "scan_files", "map_data",
            "generate_response", "full_pipeline", "dashboard",
        ],
        default="full_pipeline",
        help="Action to perform",
    )
    parser.add_argument("--requester-name", default="Test Subject")
    parser.add_argument("--requester-email", default="test@example.com")
    parser.add_argument("--request-channel", default="email")
    parser.add_argument("--scan-dirs", nargs="*", default=[])
    parser.add_argument("--db-connection", default="")
    parser.add_argument("--output-dir", default="dsar_output")
    parser.add_argument("--config", default="dsar_config.json")
    parser.add_argument("--format", choices=["json", "csv"], default="json")
    parser.add_argument("--min-confidence", type=float, default=0.5)
    parser.add_argument(
        "--scan-text",
        help="Direct text to scan for PII",
        default="",
    )
    args = parser.parse_args()

    print("=" * 60)
    print("GDPR DSAR Workflow Automation Agent")
    print("=" * 60)

    if args.action == "scan_pii" and args.scan_text:
        matcher = PIIPatternMatcher()
        matches = matcher.scan_text(args.scan_text, args.min_confidence)
        print(f"\n[+] PII Scan Results ({len(matches)} matches):")
        for m in matches:
            print(f"  [{m['type']}] '{m['value']}' "
                  f"(confidence: {m['confidence']}, category: {m['gdpr_category']})")
        return

    if args.action == "scan_files" and args.scan_dirs:
        pii = PIIDiscoveryEngine()
        results = pii.scan_files(
            args.scan_dirs,
            {"email": args.requester_email, "name": args.requester_name},
        )
        print(f"\n[+] File Scan: {results['files_scanned']} files scanned, "
              f"{results['files_with_matches']} with matches")
        output_file = Path(args.output_dir) / "file_scan_results.json"
        output_file.parent.mkdir(parents=True, exist_ok=True)
        output_file.write_text(json.dumps(results, indent=2))
        print(f"[+] Results saved to {output_file}")
        return

    # Full pipeline
    engine = DSARWorkflowEngine(config_path=args.config)
    pii_engine = PIIDiscoveryEngine()
    mapper = DataMapper()
    reviewer = ExemptionReviewer()
    generator = DSARResponseGenerator(
        organization_name=engine.config.get("organization_name", "Organization"),
        dpo_email=engine.config.get("dpo_email", "dpo@organization.com"),
    )
    audit_logger = DSARAuditLogger(log_path=f"{args.output_dir}/audit_logs")

    # Step 1: Register DSAR
    print("\n[Step 1] Registering DSAR...")
    request = engine.register_dsar(
        requester_name=args.requester_name,
        requester_email=args.requester_email,
        request_channel=args.request_channel,
        request_text="Request for all personal data under GDPR Article 15.",
        identity_docs=["email_verified"],
    )
    print(f"  DSAR ID: {request['dsar_id']}")
    print(f"  Deadline: {request['deadline_date']}")
    print(f"  Status: {request['status']}")

    audit_logger.log_event(request["dsar_id"], "request_received", {
        "channel": args.request_channel,
        "requester": args.requester_name,
    })

    # Step 2: PII Discovery
    print("\n[Step 2] Running PII Discovery...")
    engine.update_status(request["dsar_id"], "pii_discovery")

    search_ids = {"email": args.requester_email, "name": args.requester_name}
    all_results = []

    if args.db_connection:
        db_results = pii_engine.scan_database(args.db_connection, search_ids)
        all_results.append(db_results)
        print(f"  Database: {db_results['queries_generated']} queries generated")

    if args.scan_dirs:
        file_results = pii_engine.scan_files(args.scan_dirs, search_ids)
        all_results.append(file_results)
        print(f"  Files: {file_results['files_scanned']} scanned, "
              f"{file_results['files_with_matches']} matches")

    consolidated = pii_engine.consolidate_results(*all_results)
    print(f"  Total PII records: {consolidated['total_records']}")

    audit_logger.log_event(request["dsar_id"], "pii_discovery_complete", {
        "records_found": consolidated["total_records"],
        "sources": consolidated["sources"],
    })

    # Step 3: Data Mapping
    print("\n[Step 3] Mapping to Article 15 categories...")
    mapped = mapper.map_to_article15(consolidated, args.requester_email)
    print(f"  Categories mapped: {len(mapped['categories'])}")

    # Step 4: Exemption Review
    print("\n[Step 4] Reviewing exemptions...")
    engine.update_status(request["dsar_id"], "exemption_review")
    review = reviewer.review_exemptions(mapped)
    redacted = reviewer.apply_redactions(mapped, review["exemptions"])
    print(f"  Exemptions flagged for DPO review: {review['exemption_count']}")

    audit_logger.log_event(request["dsar_id"], "exemption_review_complete", {
        "exemptions_flagged": review["exemption_count"],
    })

    # Step 5: Response Generation
    print("\n[Step 5] Generating response package...")
    engine.update_status(request["dsar_id"], "response_generation")
    response = generator.generate_response(
        dsar_id=request["dsar_id"],
        data_subject=args.requester_name,
        mapped_data=redacted,
        format=args.format,
        request_date=datetime.utcnow().strftime("%Y-%m-%d"),
    )
    saved_files = generator.save_response_package(response, args.output_dir)
    for f in saved_files:
        print(f"  Saved: {f}")

    audit_logger.log_event(request["dsar_id"], "response_generated", {
        "documents": len(response["documents"]),
        "format": args.format,
    })

    # Step 6: Mark complete
    engine.update_status(request["dsar_id"], "response_sent",
                         "Response package generated and ready for delivery")
    audit_logger.log_event(request["dsar_id"], "response_sent", {
        "delivery_method": "manual",
    })

    # Compliance report
    print("\n[Step 6] Generating compliance report...")
    compliance = audit_logger.generate_compliance_report(request["dsar_id"])
    compliance_file = Path(args.output_dir) / f"compliance_report_{request['dsar_id']}.json"
    compliance_file.write_text(json.dumps(compliance, indent=2))
    print(f"  Compliance status: {compliance['overall_compliance']}")
    print(f"  Report saved: {compliance_file}")

    # Dashboard
    print("\n" + "=" * 60)
    dashboard = engine.generate_dashboard()
    print(f"Dashboard: {dashboard['total_dsars']} DSARs, "
          f"{dashboard['overdue_count']} overdue")
    print(f"Days remaining: {engine.days_remaining(request['dsar_id'])}")
    print("=" * 60)
    print("\n[+] DSAR processing complete.")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-privacy-impact-assessment/SKILL.md =====
---
name: performing-privacy-impact-assessment
description: 'Automates the Privacy Impact Assessment (PIA) workflow including data
  flow mapping, privacy risk scoring matrices, GDPR Article 35 DPIA and CCPA/CPRA
  alignment checks, data inventory cataloging, and remediation tracking. Implements
  the NIST Privacy Framework PRAM methodology and ICO DPIA guidance for systematic
  identification and mitigation of privacy risks across processing activities. Use
  when conducting privacy assessments for new systems, evaluating regulatory compliance
  posture, or building automated privacy governance programs.

  '
domain: cybersecurity
subdomain: privacy-compliance
tags:
- privacy
- impact-assessment
- GDPR
- CCPA
- NIST
- DPIA
- data-flow-mapping
- risk-scoring
version: '1.0'
author: mukul975
license: Apache-2.0
nist_csf:
- GV.PO-01
- PR.DS-01
- GV.OC-05
mitre_attack:
- T1078
- T1190
- T1059
---

# Performing Privacy Impact Assessment

## When to Use

- When launching a new system, product, or processing activity that handles personal data
- When conducting GDPR Article 35 Data Protection Impact Assessments (DPIAs)
- When evaluating CCPA/CPRA compliance for data processing operations
- When performing privacy risk assessments aligned to the NIST Privacy Framework
- When mapping data flows across organizational boundaries and third-party processors
- When building automated privacy governance and assessment pipelines
- When preparing for regulatory audits or demonstrating accountability obligations

## Prerequisites

- Familiarity with GDPR, CCPA/CPRA, and NIST Privacy Framework concepts
- Access to data processing inventories and system architecture documentation
- Python 3.8+ with required dependencies installed
- Appropriate authorization from the Data Protection Officer (DPO) or privacy team
- Knowledge of organizational data flows and third-party processor relationships

## Instructions

### Phase 1: Data Inventory and Processing Activity Catalog

Build a complete inventory of personal data processing activities. Each record of
processing activity (ROPA) entry must capture the data categories, legal basis,
retention periods, and data subjects involved.

```python
from agent import PrivacyImpactAssessmentEngine

engine = PrivacyImpactAssessmentEngine()

# Register a processing activity for assessment
activity = engine.register_processing_activity(
    name="Customer Analytics Platform",
    description="Collects browsing behavior and purchase history for personalization",
    data_controller="Acme Corp",
    data_processor="CloudAnalytics Inc",
    data_categories=["browsing_history", "purchase_records", "ip_address", "device_id"],
    data_subjects=["customers", "website_visitors"],
    legal_basis="consent",
    retention_period_days=730,
    cross_border_transfer=True,
    transfer_destinations=["US", "IN"],
    automated_decision_making=True,
)
print(f"Registered activity: {activity['activity_id']}")
```

### Phase 2: Data Flow Mapping

Map all data flows from collection to deletion, identifying every touchpoint,
transformation, and storage location. This reveals hidden privacy risks in data
movement across systems.

```python
# Build the data flow map
flow_map = engine.map_data_flows(
    activity_id=activity["activity_id"],
    flows=[
        {
            "stage": "collection",
            "source": "Web browser cookie + form submission",
            "destination": "CDN edge server",
            "data_elements": ["ip_address", "device_id", "browsing_history"],
            "encryption_in_transit": True,
            "protocol": "TLS 1.3",
        },
        {
            "stage": "processing",
            "source": "CDN edge server",
            "destination": "Analytics data warehouse (US-East)",
            "data_elements": ["browsing_history", "purchase_records", "device_id"],
            "encryption_in_transit": True,
            "encryption_at_rest": True,
            "protocol": "mTLS",
        },
        {
            "stage": "storage",
            "source": "Analytics data warehouse",
            "destination": "S3 encrypted bucket",
            "data_elements": ["browsing_history", "purchase_records"],
            "encryption_at_rest": True,
            "retention_days": 730,
            "access_controls": "IAM role-based, MFA required",
        },
        {
            "stage": "sharing",
            "source": "Analytics data warehouse",
            "destination": "Third-party ML provider (IN)",
            "data_elements": ["browsing_history", "purchase_records"],
            "encryption_in_transit": True,
            "data_processing_agreement": True,
            "cross_border": True,
        },
        {
            "stage": "deletion",
            "source": "S3 bucket + data warehouse",
            "destination": "Secure erasure",
            "method": "Cryptographic erasure + lifecycle policy",
            "verification": "Automated deletion audit log",
        },
    ],
)
engine.render_data_flow_diagram(flow_map)
```

### Phase 3: Privacy Risk Assessment with Scoring Matrix

Apply a structured risk scoring methodology evaluating likelihood and impact
across multiple privacy risk dimensions. The matrix aligns with both the
NIST PRAM and ICO DPIA risk assessment approaches.

```python
# Run the risk assessment
risk_report = engine.assess_privacy_risks(
    activity_id=activity["activity_id"],
    assessment_type="full_dpia",
)

# Display risk matrix results
for risk in risk_report["risks"]:
    print(f"[{risk['severity']}] {risk['category']}: {risk['description']}")
    print(f"  Likelihood: {risk['likelihood']}/5 | Impact: {risk['impact']}/5 | Score: {risk['risk_score']}/25")
    print(f"  Mitigation: {risk['recommended_mitigation']}")
```

Risk categories evaluated include:
1. **Data Minimization** -- Excessive collection beyond stated purpose
2. **Purpose Limitation** -- Secondary use without legal basis
3. **Cross-Border Transfer** -- Transfers without adequate safeguards (SCCs, BCRs)
4. **Automated Decision Making** -- Profiling without human oversight or appeal
5. **Data Subject Rights** -- Inability to fulfill access/erasure/portability requests
6. **Third-Party Risk** -- Processor compliance gaps, subprocessor chains
7. **Security Controls** -- Encryption, access control, breach response gaps
8. **Retention** -- Storing data beyond necessity or legal requirement
9. **Consent Management** -- Invalid or ambiguous consent mechanisms
10. **Breach Notification** -- Inability to detect and notify within 72 hours (GDPR)

### Phase 4: GDPR and CCPA/CPRA Alignment Checks

Run automated compliance checks against specific regulatory requirements.
The engine maps each processing activity against article-level GDPR obligations
and CCPA/CPRA consumer rights requirements.

```python
# GDPR compliance check
gdpr_report = engine.check_gdpr_compliance(activity_id=activity["activity_id"])
print(f"GDPR Score: {gdpr_report['compliance_score']}/100")
for finding in gdpr_report["findings"]:
    print(f"  [{finding['status']}] Art.{finding['article']}: {finding['description']}")

# CCPA/CPRA compliance check
ccpa_report = engine.check_ccpa_compliance(activity_id=activity["activity_id"])
print(f"CCPA Score: {ccpa_report['compliance_score']}/100")
for finding in ccpa_report["findings"]:
    print(f"  [{finding['status']}] Sec.{finding['section']}: {finding['description']}")
```

### Phase 5: Remediation Plan and Report Generation

Generate a prioritized remediation plan with specific action items, responsible
parties, deadlines, and generate the formal PIA/DPIA report document.

```python
# Generate remediation plan
remediation = engine.generate_remediation_plan(
    activity_id=activity["activity_id"],
    risk_report=risk_report,
    gdpr_report=gdpr_report,
    ccpa_report=ccpa_report,
)

for item in remediation["action_items"]:
    print(f"[{item['priority']}] {item['action']}")
    print(f"  Owner: {item['owner']} | Deadline: {item['deadline']}")
    print(f"  Addresses: {', '.join(item['addresses_risks'])}")

# Generate formal DPIA report
engine.generate_dpia_report(
    activity_id=activity["activity_id"],
    output_path="dpia_report_customer_analytics.json",
    format="json",
)
print("[+] DPIA report generated")
```

## Examples

### Quick Screening Assessment

Determine whether a full DPIA is required using the ICO screening checklist:

```python
engine = PrivacyImpactAssessmentEngine()

screening = engine.run_screening_checklist(
    uses_special_category_data=False,
    large_scale_processing=True,
    systematic_monitoring=True,
    automated_decision_making=True,
    cross_border_transfer=True,
    vulnerable_data_subjects=False,
    innovative_technology=True,
    denial_of_service_or_rights=False,
)
print(f"DPIA Required: {screening['dpia_required']}")
print(f"Triggers: {screening['triggers']}")
# Output: DPIA Required: True
# Triggers: ['large_scale_processing', 'systematic_monitoring',
#            'automated_decision_making', 'cross_border_transfer',
#            'innovative_technology']
```

### Batch Assessment of Multiple Processing Activities

```python
engine = PrivacyImpactAssessmentEngine()

activities = [
    {"name": "Email Marketing", "data_categories": ["email", "name"],
     "legal_basis": "consent", "cross_border_transfer": False},
    {"name": "HR Analytics", "data_categories": ["employee_id", "performance_scores",
     "health_data"], "legal_basis": "legitimate_interest", "cross_border_transfer": True},
    {"name": "Fraud Detection", "data_categories": ["transaction_data", "ip_address",
     "device_fingerprint"], "legal_basis": "legitimate_interest",
     "automated_decision_making": True, "cross_border_transfer": False},
]

for act_def in activities:
    activity = engine.register_processing_activity(**act_def)
    risk = engine.assess_privacy_risks(activity_id=activity["activity_id"])
    print(f"{act_def['name']}: Overall Risk={risk['overall_risk_level']} "
          f"({risk['risk_count_by_severity']})")
```

### NIST Privacy Framework Profile Mapping

```python
engine = PrivacyImpactAssessmentEngine()

profile = engine.generate_nist_privacy_profile(
    activity_id=activity["activity_id"],
    target_tier="tier_3",  # Repeatable
)

for function_id, outcomes in profile["functions"].items():
    print(f"\n{function_id}:")
    for outcome in outcomes:
        status = "PASS" if outcome["implemented"] else "GAP"
        print(f"  [{status}] {outcome['subcategory']}: {outcome['description']}")
```

===== END FILE =====


===== FILE: skills/performing-privacy-impact-assessment/LICENSE =====

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


===== FILE: skills/performing-privacy-impact-assessment/references/api-reference.md =====
# API Reference: Performing Privacy Impact Assessment

## PrivacyImpactAssessmentEngine

Core engine for automated PIA/DPIA workflows.

### Initialization

```python
from agent import PrivacyImpactAssessmentEngine

engine = PrivacyImpactAssessmentEngine(
    organization_name="Acme Corp",
    dpo_email="dpo@acme.com",
)
```

### register_processing_activity()

Register a processing activity for assessment.

```python
activity = engine.register_processing_activity(
    name="Customer Analytics",                    # Required
    description="Behavioral analytics pipeline",  # Required
    data_controller="Acme Corp",                  # Controller name
    data_processor="CloudAnalytics Inc",          # Processor name
    data_categories=["email", "ip_address"],      # List of data types
    data_subjects=["customers"],                  # Affected individuals
    legal_basis="consent",                        # consent|contract|legal_obligation|
                                                  # vital_interests|public_task|legitimate_interest
    retention_period_days=365,                    # Days before deletion
    cross_border_transfer=True,                   # International transfer
    transfer_destinations=["US", "IN"],           # ISO country codes
    automated_decision_making=False,              # Profiling/auto-decisions
)
# Returns: dict with activity_id, sensitivity_profile, etc.
```

**Supported data_categories:**

| Category | Sensitivity | Weight |
|----------|------------|--------|
| health_data, biometric_data, genetic_data | special_category | 5 |
| ssn, financial_account, credit_card, login_credentials | high | 4 |
| email, phone_number, ip_address, geolocation | medium | 3 |
| name, job_title, browsing_history, device_id | low | 2 |
| cookie_id, public_profile | low | 1 |

### map_data_flows()

Map data flows through the processing lifecycle.

```python
flow_map = engine.map_data_flows(
    activity_id="PA-XXXXXXXX",
    flows=[
        {
            "stage": "collection",       # collection|processing|storage|sharing|deletion
            "source": "Web form",
            "destination": "API server",
            "data_elements": ["email", "name"],
            "encryption_in_transit": True,
            "encryption_at_rest": False,
            "protocol": "TLS 1.3",
            "cross_border": False,
            "data_processing_agreement": False,
        },
    ],
)
```

### assess_privacy_risks()

Run risk assessment with scoring matrix.

```python
risk_report = engine.assess_privacy_risks(
    activity_id="PA-XXXXXXXX",
    assessment_type="full_dpia",  # full_dpia|screening|targeted
)
```

**Risk Scoring Matrix:**

| Score | Severity |
|-------|----------|
| 20-25 | CRITICAL |
| 15-19 | HIGH |
| 10-14 | MEDIUM |
| 5-9 | LOW |
| 1-4 | INFORMATIONAL |

Score = Likelihood (1-5) x Impact (1-5)

**Risk Categories Evaluated:**

| ID | Category | Description |
|----|----------|-------------|
| RISK-001 | Data Minimization | Excessive collection beyond purpose |
| RISK-002 | Purpose Limitation | Undefined or exceeded purposes |
| RISK-003 | Cross-Border Transfer | Transfer without safeguards |
| RISK-004 | Automated Decision Making | No human oversight |
| RISK-005 | Data Subject Rights | Missing DSR mechanisms |
| RISK-006 | Third-Party Risk | Processor compliance gaps |
| RISK-007 | Security Controls | Encryption/access gaps |
| RISK-008 | Retention | Over-retention or no policy |
| RISK-009 | Consent Management | Ambiguous consent |
| RISK-010 | Breach Notification | No 72-hour capability |
| RISK-011 | Special Category Data | Missing Art. 9 basis |
| RISK-012 | Transparency | Incomplete privacy notice |
| RISK-013 | Vulnerable Data Subjects | Missing extra safeguards |
| RISK-014 | Data Quality | No accuracy measures |

### run_screening_checklist()

ICO DPIA screening to determine if full DPIA is required.

```python
result = engine.run_screening_checklist(
    uses_special_category_data=False,
    large_scale_processing=True,
    systematic_monitoring=True,
    automated_decision_making=False,
    cross_border_transfer=True,
    vulnerable_data_subjects=False,
    innovative_technology=False,
    denial_of_service_or_rights=False,
    evaluation_or_scoring=False,
    matching_or_combining_datasets=False,
)
# Returns: {"dpia_required": True, "triggers": [...], ...}
```

### check_gdpr_compliance()

Article-level GDPR compliance checks.

```python
gdpr_report = engine.check_gdpr_compliance(activity_id="PA-XXXXXXXX")
# Returns: compliance_score (0-100), findings per article
```

**GDPR Articles Checked:**

| Article | Title |
|---------|-------|
| Art. 5 | Principles (lawfulness, minimization, retention, etc.) |
| Art. 6 | Lawfulness of processing |
| Art. 7 | Conditions for consent |
| Art. 13 | Information at collection |
| Art. 22 | Automated decision-making |
| Art. 25 | Data protection by design |
| Art. 28 | Processor obligations |
| Art. 30 | Records of processing |
| Art. 32 | Security of processing |
| Art. 33 | Breach notification |
| Art. 35 | DPIA requirements |
| Art. 44 | Transfer safeguards |

### check_ccpa_compliance()

CCPA/CPRA section-level compliance checks.

```python
ccpa_report = engine.check_ccpa_compliance(activity_id="PA-XXXXXXXX")
# Returns: compliance_score (0-100), findings per section
```

**CCPA Sections Checked:**

| Section | Title |
|---------|-------|
| 1798.100 | Right to know |
| 1798.105 | Right to delete |
| 1798.106 | Right to correct |
| 1798.110 | Right to specific PI |
| 1798.115 | Right to know about selling/sharing |
| 1798.120 | Right to opt-out |
| 1798.121 | Limit use of sensitive PI |
| 1798.125 | Non-discrimination |
| 1798.130 | Notice and request handling |
| 1798.135 | Do Not Sell link |
| 1798.185 | CPRA risk assessment |

### generate_nist_privacy_profile()

Map activity against NIST Privacy Framework functions.

```python
profile = engine.generate_nist_privacy_profile(
    activity_id="PA-XXXXXXXX",
    target_tier="tier_3",  # tier_1|tier_2|tier_3|tier_4
)
# Returns: coverage per function (ID-P, GV-P, CT-P, CM-P, PR-P)
```

### generate_remediation_plan()

Prioritized remediation action items.

```python
plan = engine.generate_remediation_plan(
    activity_id="PA-XXXXXXXX",
    risk_report=risk_report,
    gdpr_report=gdpr_report,
    ccpa_report=ccpa_report,
)
```

**Priority Levels:**

| Priority | Severity | Deadline |
|----------|----------|----------|
| P1 | CRITICAL | 14 days |
| P2 | HIGH | 30 days |
| P3 | MEDIUM | 60 days |
| P4 | LOW | 90 days |

### generate_dpia_report()

Generate formal DPIA report document.

```python
engine.generate_dpia_report(
    activity_id="PA-XXXXXXXX",
    output_path="dpia_report.json",
    format="json",
)
```

## CLI Usage

```bash
# Run demonstration workflow
python agent.py --action demo --org "Acme Corp" --output report.json

# Run screening checklist
python agent.py --action screening

# Specify DPO email
python agent.py --action demo --dpo-email dpo@acme.com --output dpia.json
```

## References

- ICO DPIA Guidance: https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/accountability-and-governance/data-protection-impact-assessments-dpias/
- NIST Privacy Framework: https://www.nist.gov/privacy-framework
- NIST PRAM: https://www.nist.gov/itl/applied-cybersecurity/privacy-engineering/collaboration-space/privacy-risk-assessment
- GDPR Full Text: https://gdpr-info.eu/
- CCPA Full Text: https://oag.ca.gov/privacy/ccpa
- IAPP PIA Template: https://iapp.org/resources/article/private-sector-privacy-impact-assessment-template/

===== END FILE =====


===== FILE: skills/performing-privacy-impact-assessment/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing automated Privacy Impact Assessments (PIA/DPIA).

Implements the NIST Privacy Framework PRAM methodology and ICO DPIA guidance
for systematic identification and mitigation of privacy risks. Supports
GDPR Article 35 and CCPA/CPRA compliance checks with risk scoring matrices.
"""

import os
import json
import uuid
import argparse
from datetime import datetime, timedelta
from copy import deepcopy


# ---------------------------------------------------------------------------
# Data sensitivity classification
# ---------------------------------------------------------------------------
DATA_SENSITIVITY = {
    # Special category / sensitive (GDPR Art. 9)
    "health_data": {"sensitivity": "special_category", "weight": 5},
    "biometric_data": {"sensitivity": "special_category", "weight": 5},
    "genetic_data": {"sensitivity": "special_category", "weight": 5},
    "racial_ethnic_origin": {"sensitivity": "special_category", "weight": 5},
    "political_opinions": {"sensitivity": "special_category", "weight": 5},
    "religious_beliefs": {"sensitivity": "special_category", "weight": 5},
    "sexual_orientation": {"sensitivity": "special_category", "weight": 5},
    "trade_union_membership": {"sensitivity": "special_category", "weight": 5},
    # High sensitivity PII
    "ssn": {"sensitivity": "high", "weight": 4},
    "financial_account": {"sensitivity": "high", "weight": 4},
    "credit_card": {"sensitivity": "high", "weight": 4},
    "passport_number": {"sensitivity": "high", "weight": 4},
    "drivers_license": {"sensitivity": "high", "weight": 4},
    "login_credentials": {"sensitivity": "high", "weight": 4},
    "performance_scores": {"sensitivity": "high", "weight": 4},
    "salary_data": {"sensitivity": "high", "weight": 4},
    # Medium sensitivity
    "email": {"sensitivity": "medium", "weight": 3},
    "phone_number": {"sensitivity": "medium", "weight": 3},
    "date_of_birth": {"sensitivity": "medium", "weight": 3},
    "home_address": {"sensitivity": "medium", "weight": 3},
    "ip_address": {"sensitivity": "medium", "weight": 3},
    "geolocation": {"sensitivity": "medium", "weight": 3},
    "employee_id": {"sensitivity": "medium", "weight": 3},
    "purchase_records": {"sensitivity": "medium", "weight": 3},
    "transaction_data": {"sensitivity": "medium", "weight": 3},
    # Low sensitivity
    "name": {"sensitivity": "low", "weight": 2},
    "job_title": {"sensitivity": "low", "weight": 2},
    "company_name": {"sensitivity": "low", "weight": 2},
    "browsing_history": {"sensitivity": "low", "weight": 2},
    "device_id": {"sensitivity": "low", "weight": 2},
    "device_fingerprint": {"sensitivity": "low", "weight": 2},
    "cookie_id": {"sensitivity": "low", "weight": 1},
    "public_profile": {"sensitivity": "low", "weight": 1},
}

# Countries with GDPR adequacy decisions (as of 2026)
GDPR_ADEQUATE_COUNTRIES = {
    "AD", "AR", "CA", "FO", "GG", "IL", "IM", "JP", "JE", "NZ",
    "KR", "CH", "GB", "UY", "US",  # US under EU-US Data Privacy Framework
    # All EU/EEA member states
    "AT", "BE", "BG", "HR", "CY", "CZ", "DK", "EE", "FI", "FR",
    "DE", "GR", "HU", "IE", "IT", "LV", "LT", "LU", "MT", "NL",
    "PL", "PT", "RO", "SK", "SI", "ES", "SE", "IS", "LI", "NO",
}

# GDPR articles relevant to DPIA
GDPR_DPIA_ARTICLES = {
    "5": {
        "title": "Principles relating to processing of personal data",
        "checks": [
            {"id": "5.1a", "desc": "Lawfulness, fairness and transparency",
             "field": "legal_basis", "required": True},
            {"id": "5.1b", "desc": "Purpose limitation",
             "field": "purpose_specified", "required": True},
            {"id": "5.1c", "desc": "Data minimization",
             "field": "data_minimization_justified", "required": True},
            {"id": "5.1d", "desc": "Accuracy of data",
             "field": "accuracy_measures", "required": True},
            {"id": "5.1e", "desc": "Storage limitation",
             "field": "retention_period_days", "required": True},
            {"id": "5.1f", "desc": "Integrity and confidentiality",
             "field": "security_measures", "required": True},
        ],
    },
    "6": {
        "title": "Lawfulness of processing",
        "checks": [
            {"id": "6.1", "desc": "Valid legal basis identified",
             "field": "legal_basis", "valid_values": [
                 "consent", "contract", "legal_obligation",
                 "vital_interests", "public_task", "legitimate_interest"]},
        ],
    },
    "7": {
        "title": "Conditions for consent",
        "checks": [
            {"id": "7.1", "desc": "Demonstrable consent obtained",
             "field": "consent_mechanism", "condition_field": "legal_basis",
             "condition_value": "consent"},
            {"id": "7.3", "desc": "Right to withdraw consent",
             "field": "consent_withdrawal_mechanism", "condition_field": "legal_basis",
             "condition_value": "consent"},
        ],
    },
    "13": {
        "title": "Information to be provided at collection",
        "checks": [
            {"id": "13.1", "desc": "Privacy notice provided at collection",
             "field": "privacy_notice", "required": True},
        ],
    },
    "22": {
        "title": "Automated individual decision-making",
        "checks": [
            {"id": "22.1", "desc": "Right not to be subject to automated decision",
             "field": "human_oversight", "condition_field": "automated_decision_making",
             "condition_value": True},
            {"id": "22.3", "desc": "Right to contest automated decision",
             "field": "appeal_mechanism", "condition_field": "automated_decision_making",
             "condition_value": True},
        ],
    },
    "25": {
        "title": "Data protection by design and by default",
        "checks": [
            {"id": "25.1", "desc": "Privacy by design measures implemented",
             "field": "privacy_by_design", "required": True},
            {"id": "25.2", "desc": "Data protection by default",
             "field": "data_protection_defaults", "required": True},
        ],
    },
    "28": {
        "title": "Processor obligations",
        "checks": [
            {"id": "28.3", "desc": "Data processing agreement with processor",
             "field": "data_processing_agreement", "condition_field": "data_processor",
             "condition_value": "_nonempty"},
        ],
    },
    "30": {
        "title": "Records of processing activities",
        "checks": [
            {"id": "30.1", "desc": "ROPA maintained",
             "field": "ropa_entry", "required": True},
        ],
    },
    "32": {
        "title": "Security of processing",
        "checks": [
            {"id": "32.1", "desc": "Appropriate technical and organizational measures",
             "field": "security_measures", "required": True},
        ],
    },
    "33": {
        "title": "Notification of breach to supervisory authority",
        "checks": [
            {"id": "33.1", "desc": "Breach notification within 72 hours capability",
             "field": "breach_notification_plan", "required": True},
        ],
    },
    "35": {
        "title": "Data protection impact assessment",
        "checks": [
            {"id": "35.7a", "desc": "Systematic description of processing",
             "field": "processing_description", "required": True},
            {"id": "35.7b", "desc": "Necessity and proportionality assessment",
             "field": "necessity_assessment", "required": True},
            {"id": "35.7c", "desc": "Risks to rights and freedoms assessment",
             "field": "risk_assessment_completed", "required": True},
            {"id": "35.7d", "desc": "Measures to address risks",
             "field": "risk_mitigation_measures", "required": True},
        ],
    },
    "44": {
        "title": "General principle for transfers",
        "checks": [
            {"id": "44.1", "desc": "Transfer safeguards for cross-border data",
             "field": "transfer_safeguards", "condition_field": "cross_border_transfer",
             "condition_value": True},
        ],
    },
}

# CCPA/CPRA sections for compliance checks
CCPA_SECTIONS = {
    "1798.100": {
        "title": "Consumer right to know",
        "checks": [
            {"id": "100.a", "desc": "Right to know what PI is collected",
             "field": "data_inventory_available", "required": True},
            {"id": "100.b", "desc": "Disclosure of categories of PI collected",
             "field": "categories_disclosed", "required": True},
        ],
    },
    "1798.105": {
        "title": "Consumer right to delete",
        "checks": [
            {"id": "105.a", "desc": "Right to request deletion of PI",
             "field": "deletion_mechanism", "required": True},
        ],
    },
    "1798.106": {
        "title": "Consumer right to correct",
        "checks": [
            {"id": "106.a", "desc": "Right to correct inaccurate PI",
             "field": "correction_mechanism", "required": True},
        ],
    },
    "1798.110": {
        "title": "Right to know specific pieces of PI",
        "checks": [
            {"id": "110.a", "desc": "Disclose categories, sources, purposes, third parties",
             "field": "data_inventory_available", "required": True},
        ],
    },
    "1798.115": {
        "title": "Right to know about selling/sharing",
        "checks": [
            {"id": "115.a", "desc": "Disclosure of selling or sharing of PI",
             "field": "selling_sharing_disclosed", "required": True},
        ],
    },
    "1798.120": {
        "title": "Consumer right to opt-out of sale/sharing",
        "checks": [
            {"id": "120.a", "desc": "Do Not Sell or Share opt-out mechanism",
             "field": "opt_out_mechanism", "condition_field": "sells_or_shares_data",
             "condition_value": True},
        ],
    },
    "1798.121": {
        "title": "Right to limit use of sensitive PI",
        "checks": [
            {"id": "121.a", "desc": "Limit use and disclosure of sensitive PI",
             "field": "sensitive_data_controls", "condition_field": "_has_sensitive_data",
             "condition_value": True},
        ],
    },
    "1798.125": {
        "title": "Non-discrimination",
        "checks": [
            {"id": "125.a", "desc": "No discrimination for exercising rights",
             "field": "non_discrimination_policy", "required": True},
        ],
    },
    "1798.130": {
        "title": "Notice and request handling",
        "checks": [
            {"id": "130.a", "desc": "Methods for submitting consumer requests",
             "field": "request_submission_methods", "required": True},
            {"id": "130.a.2", "desc": "Respond to requests within 45 days",
             "field": "response_sla_days", "required": True, "max_value": 45},
        ],
    },
    "1798.135": {
        "title": "Do Not Sell or Share link",
        "checks": [
            {"id": "135.a", "desc": "Clear and conspicuous opt-out link on homepage",
             "field": "opt_out_link_on_homepage", "condition_field": "sells_or_shares_data",
             "condition_value": True},
        ],
    },
    "1798.185": {
        "title": "CPRA risk assessment requirement",
        "checks": [
            {"id": "185.a.15", "desc": "Risk assessment for significant processing",
             "field": "risk_assessment_completed", "required": True},
        ],
    },
}

# NIST Privacy Framework functions and categories
NIST_PRIVACY_FUNCTIONS = {
    "ID-P": {
        "name": "Identify-P",
        "description": "Develop organizational understanding to manage privacy risk",
        "categories": {
            "ID.IM-P": {
                "name": "Inventory and Mapping",
                "subcategories": [
                    {"id": "ID.IM-P1", "desc": "Systems and assets that process data are inventoried",
                     "check": "data_inventory_available"},
                    {"id": "ID.IM-P2", "desc": "Data actions mapped to system operations",
                     "check": "data_flow_mapped"},
                    {"id": "ID.IM-P3", "desc": "Data elements mapped within data actions",
                     "check": "data_elements_mapped"},
                    {"id": "ID.IM-P4", "desc": "Data actions of service providers identified",
                     "check": "processor_actions_identified"},
                    {"id": "ID.IM-P5", "desc": "Affected individuals identified",
                     "check": "data_subjects_identified"},
                    {"id": "ID.IM-P6", "desc": "Data processing purposes identified",
                     "check": "purpose_specified"},
                    {"id": "ID.IM-P7", "desc": "Legal authorities for data actions identified",
                     "check": "legal_basis"},
                    {"id": "ID.IM-P8", "desc": "Problematic data actions assessed",
                     "check": "risk_assessment_completed"},
                ],
            },
            "ID.BE-P": {
                "name": "Business Environment",
                "subcategories": [
                    {"id": "ID.BE-P1", "desc": "Privacy values and policies established",
                     "check": "privacy_policy_established"},
                    {"id": "ID.BE-P2", "desc": "Privacy role in enterprise risk management",
                     "check": "privacy_in_erm"},
                ],
            },
            "ID.RA-P": {
                "name": "Risk Assessment",
                "subcategories": [
                    {"id": "ID.RA-P1", "desc": "Contextual factors related to data processing identified",
                     "check": "contextual_factors_documented"},
                    {"id": "ID.RA-P2", "desc": "Data analytic inputs and outputs identified",
                     "check": "analytics_io_documented"},
                    {"id": "ID.RA-P3", "desc": "Potential problematic data actions assessed",
                     "check": "risk_assessment_completed"},
                    {"id": "ID.RA-P4", "desc": "Problematic data actions prioritized",
                     "check": "risks_prioritized"},
                    {"id": "ID.RA-P5", "desc": "Risk responses identified and prioritized",
                     "check": "risk_responses_identified"},
                ],
            },
        },
    },
    "GV-P": {
        "name": "Govern-P",
        "description": "Develop and implement organizational governance structure",
        "categories": {
            "GV.PO-P": {
                "name": "Governance Policies, Processes, and Procedures",
                "subcategories": [
                    {"id": "GV.PO-P1", "desc": "Organizational privacy policy established",
                     "check": "privacy_policy_established"},
                    {"id": "GV.PO-P2", "desc": "Privacy risk management processes established",
                     "check": "risk_management_process"},
                    {"id": "GV.PO-P3", "desc": "Roles and responsibilities defined",
                     "check": "privacy_roles_defined"},
                    {"id": "GV.PO-P4", "desc": "Governance and risk management reviewed",
                     "check": "governance_reviewed"},
                    {"id": "GV.PO-P5", "desc": "Legal requirements identified",
                     "check": "legal_requirements_identified"},
                    {"id": "GV.PO-P6", "desc": "Governance structure includes privacy",
                     "check": "privacy_in_governance"},
                ],
            },
            "GV.AT-P": {
                "name": "Awareness and Training",
                "subcategories": [
                    {"id": "GV.AT-P1", "desc": "Workforce informed of privacy policies",
                     "check": "privacy_training"},
                    {"id": "GV.AT-P2", "desc": "Senior executives understand privacy risk",
                     "check": "executive_awareness"},
                    {"id": "GV.AT-P3", "desc": "Privacy personnel understand responsibilities",
                     "check": "privacy_team_trained"},
                    {"id": "GV.AT-P4", "desc": "Third parties understand privacy obligations",
                     "check": "third_party_aware"},
                ],
            },
            "GV.MT-P": {
                "name": "Monitoring and Review",
                "subcategories": [
                    {"id": "GV.MT-P1", "desc": "Privacy risk re-assessed on ongoing basis",
                     "check": "ongoing_reassessment"},
                    {"id": "GV.MT-P2", "desc": "Privacy values reflected in practices",
                     "check": "privacy_values_in_practice"},
                    {"id": "GV.MT-P3", "desc": "Policies updated to reflect changes",
                     "check": "policy_update_process"},
                ],
            },
        },
    },
    "CT-P": {
        "name": "Control-P",
        "description": "Develop and implement appropriate activities for data management",
        "categories": {
            "CT.DM-P": {
                "name": "Data Management",
                "subcategories": [
                    {"id": "CT.DM-P1", "desc": "Data elements managed consistent with risk strategy",
                     "check": "data_management_aligned"},
                    {"id": "CT.DM-P2", "desc": "Data retention and disposal policy",
                     "check": "retention_policy"},
                    {"id": "CT.DM-P3", "desc": "Data processing accuracy maintained",
                     "check": "accuracy_measures"},
                    {"id": "CT.DM-P4", "desc": "Data disclosed only as authorized",
                     "check": "disclosure_controls"},
                    {"id": "CT.DM-P5", "desc": "Data from processing manipulations managed",
                     "check": "derived_data_managed"},
                    {"id": "CT.DM-P6", "desc": "Data in transit managed",
                     "check": "transit_controls"},
                    {"id": "CT.DM-P7", "desc": "Mechanisms for transmitting consent preferences",
                     "check": "consent_preference_mechanism"},
                    {"id": "CT.DM-P8", "desc": "Audit/log records for data processing",
                     "check": "processing_audit_log"},
                    {"id": "CT.DM-P9", "desc": "Technical measures for consent management",
                     "check": "consent_management_system"},
                    {"id": "CT.DM-P10", "desc": "Data quality maintained through lifecycle",
                     "check": "data_quality_lifecycle"},
                ],
            },
            "CT.PO-P": {
                "name": "Data Processing Policies",
                "subcategories": [
                    {"id": "CT.PO-P1", "desc": "De-identification policies and processes",
                     "check": "deidentification_policy"},
                    {"id": "CT.PO-P2", "desc": "Data tagging for purpose and provenance",
                     "check": "data_tagging"},
                    {"id": "CT.PO-P3", "desc": "Administrative access safeguards",
                     "check": "admin_access_controls"},
                    {"id": "CT.PO-P4", "desc": "System monitoring for privacy compliance",
                     "check": "privacy_monitoring"},
                ],
            },
        },
    },
    "CM-P": {
        "name": "Communicate-P",
        "description": "Develop and implement communication activities",
        "categories": {
            "CM.AW-P": {
                "name": "Communication Policies",
                "subcategories": [
                    {"id": "CM.AW-P1", "desc": "Privacy practices communicated to individuals",
                     "check": "privacy_notice"},
                    {"id": "CM.AW-P2", "desc": "Mechanisms for individual complaints",
                     "check": "complaint_mechanism"},
                    {"id": "CM.AW-P3", "desc": "System changes communicated",
                     "check": "change_notification"},
                    {"id": "CM.AW-P4", "desc": "Data processing awareness program",
                     "check": "awareness_program"},
                ],
            },
        },
    },
    "PR-P": {
        "name": "Protect-P",
        "description": "Develop and implement safeguards for data processing",
        "categories": {
            "PR.AC-P": {
                "name": "Identity Management, Authentication and Access Control",
                "subcategories": [
                    {"id": "PR.AC-P1", "desc": "Identities and credentials managed",
                     "check": "identity_management"},
                    {"id": "PR.AC-P2", "desc": "Physical access managed",
                     "check": "physical_access_controls"},
                    {"id": "PR.AC-P3", "desc": "Remote access managed",
                     "check": "remote_access_controls"},
                    {"id": "PR.AC-P4", "desc": "Access permissions managed",
                     "check": "access_permissions"},
                    {"id": "PR.AC-P5", "desc": "Network integrity protected",
                     "check": "network_integrity"},
                    {"id": "PR.AC-P6", "desc": "Individuals authenticated",
                     "check": "authentication_controls"},
                ],
            },
            "PR.DS-P": {
                "name": "Data Security",
                "subcategories": [
                    {"id": "PR.DS-P1", "desc": "Data at rest protected",
                     "check": "encryption_at_rest"},
                    {"id": "PR.DS-P2", "desc": "Data in transit protected",
                     "check": "encryption_in_transit"},
                    {"id": "PR.DS-P3", "desc": "Systems and data formally managed",
                     "check": "asset_management"},
                    {"id": "PR.DS-P4", "desc": "Adequate capacity maintained",
                     "check": "capacity_management"},
                    {"id": "PR.DS-P5", "desc": "Protections against data leaks implemented",
                     "check": "dlp_controls"},
                    {"id": "PR.DS-P6", "desc": "Integrity checking mechanisms used",
                     "check": "integrity_checking"},
                    {"id": "PR.DS-P7", "desc": "Development and testing use non-production data",
                     "check": "non_production_data"},
                    {"id": "PR.DS-P8", "desc": "Hardware integrity verified",
                     "check": "hardware_integrity"},
                ],
            },
            "PR.PO-P": {
                "name": "Protective Policies",
                "subcategories": [
                    {"id": "PR.PO-P1", "desc": "Baseline configuration maintained",
                     "check": "baseline_config"},
                    {"id": "PR.PO-P2", "desc": "Configuration change control in place",
                     "check": "change_control"},
                    {"id": "PR.PO-P3", "desc": "Backups maintained and tested",
                     "check": "backup_tested"},
                    {"id": "PR.PO-P4", "desc": "Response and recovery plans tested",
                     "check": "incident_response_tested"},
                    {"id": "PR.PO-P5", "desc": "Improvements implemented from testing",
                     "check": "continuous_improvement"},
                    {"id": "PR.PO-P6", "desc": "Data disposal performed",
                     "check": "data_disposal"},
                    {"id": "PR.PO-P7", "desc": "Protection processes improved",
                     "check": "protection_improvement"},
                    {"id": "PR.PO-P8", "desc": "Protection technology shared",
                     "check": "protection_shared"},
                    {"id": "PR.PO-P9", "desc": "Response plans managed",
                     "check": "response_plans"},
                    {"id": "PR.PO-P10", "desc": "Protection updated with testing results",
                     "check": "protection_updated"},
                ],
            },
        },
    },
}

# Risk assessment rules
PRIVACY_RISK_RULES = [
    {
        "id": "RISK-001",
        "category": "Data Minimization",
        "description": "Excessive data collection beyond stated processing purpose",
        "check": lambda a: len(a.get("data_categories", [])) > 5 and not a.get("data_minimization_justified"),
        "likelihood_base": 4,
        "impact_base": 3,
        "mitigation": "Conduct data minimization review; remove data elements not required for stated purpose; document justification for each element collected",
    },
    {
        "id": "RISK-002",
        "category": "Purpose Limitation",
        "description": "Processing purpose not clearly specified or data used beyond original purpose",
        "check": lambda a: not a.get("purpose_specified") or a.get("secondary_purposes"),
        "likelihood_base": 3,
        "impact_base": 4,
        "mitigation": "Document explicit processing purposes; obtain separate legal basis for any secondary use; implement purpose limitation controls in data pipeline",
    },
    {
        "id": "RISK-003",
        "category": "Cross-Border Transfer",
        "description": "Personal data transferred to jurisdiction without adequate protection",
        "check": lambda a: a.get("cross_border_transfer") and any(
            d not in GDPR_ADEQUATE_COUNTRIES for d in a.get("transfer_destinations", [])),
        "likelihood_base": 3,
        "impact_base": 5,
        "mitigation": "Implement Standard Contractual Clauses (SCCs) or Binding Corporate Rules (BCRs); conduct Transfer Impact Assessment; consider data localization",
    },
    {
        "id": "RISK-004",
        "category": "Automated Decision Making",
        "description": "Automated profiling or decision-making without human oversight or appeal",
        "check": lambda a: a.get("automated_decision_making") and not a.get("human_oversight"),
        "likelihood_base": 4,
        "impact_base": 5,
        "mitigation": "Implement meaningful human review for significant automated decisions; provide appeal mechanism; document algorithm logic and bias testing results",
    },
    {
        "id": "RISK-005",
        "category": "Data Subject Rights",
        "description": "Insufficient mechanisms to fulfill data subject access, erasure, or portability requests",
        "check": lambda a: not a.get("dsr_mechanism"),
        "likelihood_base": 3,
        "impact_base": 4,
        "mitigation": "Implement automated DSR fulfillment workflows; test access, rectification, erasure, portability, and objection request handling; ensure response within regulatory timeframes",
    },
    {
        "id": "RISK-006",
        "category": "Third-Party Risk",
        "description": "Data processor lacks adequate privacy controls or valid DPA",
        "check": lambda a: a.get("data_processor") and not a.get("data_processing_agreement"),
        "likelihood_base": 3,
        "impact_base": 4,
        "mitigation": "Execute Data Processing Agreement with all processors; conduct processor privacy audits; implement subprocessor approval workflows; require breach notification clauses",
    },
    {
        "id": "RISK-007",
        "category": "Security Controls",
        "description": "Inadequate encryption, access controls, or breach detection capabilities",
        "check": lambda a: not a.get("encryption_at_rest") or not a.get("encryption_in_transit"),
        "likelihood_base": 4,
        "impact_base": 5,
        "mitigation": "Implement AES-256 encryption at rest and TLS 1.3 in transit; enforce MFA and least-privilege access; deploy intrusion detection and breach monitoring",
    },
    {
        "id": "RISK-008",
        "category": "Retention",
        "description": "Data retained beyond necessary period or no retention policy defined",
        "check": lambda a: not a.get("retention_period_days") or a.get("retention_period_days", 0) > 1095,
        "likelihood_base": 3,
        "impact_base": 3,
        "mitigation": "Define and enforce retention schedules per data category; implement automated deletion workflows; conduct periodic retention audits",
    },
    {
        "id": "RISK-009",
        "category": "Consent Management",
        "description": "Consent mechanisms are ambiguous, pre-checked, or lack granularity",
        "check": lambda a: a.get("legal_basis") == "consent" and not a.get("consent_mechanism"),
        "likelihood_base": 4,
        "impact_base": 4,
        "mitigation": "Implement granular consent management with clear opt-in; record consent evidence with timestamp and scope; provide easy withdrawal mechanism; avoid dark patterns",
    },
    {
        "id": "RISK-010",
        "category": "Breach Notification",
        "description": "No documented breach response plan or inability to notify within 72 hours",
        "check": lambda a: not a.get("breach_notification_plan"),
        "likelihood_base": 3,
        "impact_base": 5,
        "mitigation": "Document and test incident response procedures; implement automated breach detection; establish 72-hour notification workflow for supervisory authority and affected individuals",
    },
    {
        "id": "RISK-011",
        "category": "Special Category Data",
        "description": "Processing of special category data without explicit consent or Art. 9 exemption",
        "check": lambda a: any(
            DATA_SENSITIVITY.get(cat, {}).get("sensitivity") == "special_category"
            for cat in a.get("data_categories", [])
        ) and a.get("legal_basis") != "explicit_consent" and not a.get("art9_exemption"),
        "likelihood_base": 3,
        "impact_base": 5,
        "mitigation": "Obtain explicit consent for special category data; document Article 9 exemption basis; implement additional safeguards including encryption and access logging",
    },
    {
        "id": "RISK-012",
        "category": "Transparency",
        "description": "Privacy notice is incomplete, inaccessible, or not provided at point of collection",
        "check": lambda a: not a.get("privacy_notice"),
        "likelihood_base": 3,
        "impact_base": 3,
        "mitigation": "Provide clear layered privacy notice at point of collection; include all GDPR Art. 13/14 required information; make notice accessible in plain language",
    },
    {
        "id": "RISK-013",
        "category": "Vulnerable Data Subjects",
        "description": "Processing data of children, elderly, or other vulnerable individuals without additional safeguards",
        "check": lambda a: any(s in a.get("data_subjects", []) for s in [
            "children", "minors", "patients", "elderly", "disabled", "employees"]),
        "likelihood_base": 3,
        "impact_base": 4,
        "mitigation": "Implement age verification for children's data; apply heightened protections; conduct specific impact assessment for vulnerable populations; ensure guardian consent where required",
    },
    {
        "id": "RISK-014",
        "category": "Data Quality",
        "description": "No mechanisms to ensure accuracy, completeness, and currency of personal data",
        "check": lambda a: not a.get("accuracy_measures"),
        "likelihood_base": 2,
        "impact_base": 3,
        "mitigation": "Implement data validation at collection; provide self-service data correction; establish periodic data quality reviews; document accuracy requirements per data element",
    },
]


def _risk_severity(score):
    """Map a numeric risk score (1-25) to a severity label."""
    if score >= 20:
        return "CRITICAL"
    if score >= 15:
        return "HIGH"
    if score >= 10:
        return "MEDIUM"
    if score >= 5:
        return "LOW"
    return "INFORMATIONAL"


class PrivacyImpactAssessmentEngine:
    """Engine for conducting automated Privacy Impact Assessments."""

    def __init__(self, organization_name="", dpo_email=""):
        self.organization_name = organization_name
        self.dpo_email = dpo_email
        self.activities = {}
        self.flow_maps = {}
        self.risk_reports = {}
        self.gdpr_reports = {}
        self.ccpa_reports = {}

    # ------------------------------------------------------------------
    # Processing activity registration
    # ------------------------------------------------------------------
    def register_processing_activity(self, name, description="",
                                     data_controller="", data_processor="",
                                     data_categories=None, data_subjects=None,
                                     legal_basis="", retention_period_days=None,
                                     cross_border_transfer=False,
                                     transfer_destinations=None,
                                     automated_decision_making=False,
                                     **kwargs):
        """Register a processing activity for assessment."""
        activity_id = f"PA-{uuid.uuid4().hex[:8].upper()}"
        activity = {
            "activity_id": activity_id,
            "name": name,
            "description": description,
            "data_controller": data_controller,
            "data_processor": data_processor,
            "data_categories": data_categories or [],
            "data_subjects": data_subjects or [],
            "legal_basis": legal_basis,
            "retention_period_days": retention_period_days,
            "cross_border_transfer": cross_border_transfer,
            "transfer_destinations": transfer_destinations or [],
            "automated_decision_making": automated_decision_making,
            "registered_at": datetime.utcnow().isoformat(),
            "ropa_entry": True,
            "purpose_specified": bool(description),
            "processing_description": description,
        }
        activity.update(kwargs)

        # Calculate data sensitivity profile
        max_sensitivity = "low"
        total_weight = 0
        for cat in activity["data_categories"]:
            info = DATA_SENSITIVITY.get(cat, {"sensitivity": "unknown", "weight": 2})
            total_weight += info["weight"]
            if info["sensitivity"] == "special_category":
                max_sensitivity = "special_category"
            elif info["sensitivity"] == "high" and max_sensitivity not in ("special_category",):
                max_sensitivity = "high"
            elif info["sensitivity"] == "medium" and max_sensitivity in ("low",):
                max_sensitivity = "medium"

        activity["_sensitivity_profile"] = {
            "max_sensitivity": max_sensitivity,
            "total_weight": total_weight,
            "category_count": len(activity["data_categories"]),
        }

        self.activities[activity_id] = activity
        return activity

    # ------------------------------------------------------------------
    # Data flow mapping
    # ------------------------------------------------------------------
    def map_data_flows(self, activity_id, flows):
        """Map data flows for a processing activity."""
        if activity_id not in self.activities:
            raise ValueError(f"Activity {activity_id} not found")

        flow_map = {
            "activity_id": activity_id,
            "activity_name": self.activities[activity_id]["name"],
            "mapped_at": datetime.utcnow().isoformat(),
            "flows": [],
            "flow_summary": {
                "total_stages": 0,
                "has_cross_border": False,
                "encryption_gaps": [],
                "third_parties": [],
            },
        }

        for i, flow in enumerate(flows):
            flow_entry = {
                "flow_id": f"FL-{i+1:03d}",
                "stage": flow.get("stage", "unknown"),
                "source": flow.get("source", ""),
                "destination": flow.get("destination", ""),
                "data_elements": flow.get("data_elements", []),
                "encryption_in_transit": flow.get("encryption_in_transit", False),
                "encryption_at_rest": flow.get("encryption_at_rest", False),
                "protocol": flow.get("protocol", ""),
                "cross_border": flow.get("cross_border", False),
                "data_processing_agreement": flow.get("data_processing_agreement", False),
                "retention_days": flow.get("retention_days"),
                "access_controls": flow.get("access_controls", ""),
                "method": flow.get("method", ""),
                "verification": flow.get("verification", ""),
            }
            flow_map["flows"].append(flow_entry)
            flow_map["flow_summary"]["total_stages"] += 1

            if flow_entry["cross_border"]:
                flow_map["flow_summary"]["has_cross_border"] = True

            if flow_entry["stage"] in ("processing", "storage", "sharing"):
                if not flow_entry["encryption_in_transit"] and flow_entry["stage"] != "storage":
                    flow_map["flow_summary"]["encryption_gaps"].append({
                        "flow_id": flow_entry["flow_id"],
                        "gap": "No encryption in transit",
                        "stage": flow_entry["stage"],
                    })
                if not flow_entry["encryption_at_rest"] and flow_entry["stage"] == "storage":
                    flow_map["flow_summary"]["encryption_gaps"].append({
                        "flow_id": flow_entry["flow_id"],
                        "gap": "No encryption at rest",
                        "stage": flow_entry["stage"],
                    })

        self.flow_maps[activity_id] = flow_map

        # Update activity with flow-derived flags
        activity = self.activities[activity_id]
        activity["data_flow_mapped"] = True
        activity["data_elements_mapped"] = True
        if flow_map["flow_summary"]["has_cross_border"]:
            activity["cross_border_transfer"] = True
        if not flow_map["flow_summary"]["encryption_gaps"]:
            activity["encryption_in_transit"] = True
            activity["encryption_at_rest"] = True

        return flow_map

    def render_data_flow_diagram(self, flow_map):
        """Render an ASCII data flow diagram to stdout."""
        print(f"\n{'='*70}")
        print(f"DATA FLOW DIAGRAM: {flow_map['activity_name']}")
        print(f"{'='*70}")

        for flow in flow_map["flows"]:
            stage_label = flow["stage"].upper()
            enc_icon = "[ENC]" if flow["encryption_in_transit"] else "[!!!]"
            border_icon = "[XBORDER]" if flow["cross_border"] else ""
            elements = ", ".join(flow["data_elements"][:3])
            if len(flow["data_elements"]) > 3:
                elements += f" (+{len(flow['data_elements'])-3} more)"

            print(f"\n  [{stage_label}] {flow['source']}")
            print(f"    |")
            print(f"    | {enc_icon} {border_icon} ({elements})")
            print(f"    v")
            print(f"  [{stage_label}] {flow['destination']}")

        if flow_map["flow_summary"]["encryption_gaps"]:
            print(f"\n  WARNING: {len(flow_map['flow_summary']['encryption_gaps'])} encryption gap(s) detected")
            for gap in flow_map["flow_summary"]["encryption_gaps"]:
                print(f"    - {gap['flow_id']}: {gap['gap']} at {gap['stage']} stage")

        print(f"\n{'='*70}\n")

    # ------------------------------------------------------------------
    # Risk assessment
    # ------------------------------------------------------------------
    def assess_privacy_risks(self, activity_id, assessment_type="full_dpia"):
        """Assess privacy risks for a processing activity."""
        if activity_id not in self.activities:
            raise ValueError(f"Activity {activity_id} not found")

        activity = self.activities[activity_id]
        sensitivity = activity.get("_sensitivity_profile", {})

        risks = []
        for rule in PRIVACY_RISK_RULES:
            try:
                triggered = rule["check"](activity)
            except Exception:
                triggered = False

            if triggered:
                # Adjust scores based on data sensitivity
                likelihood_adj = 0
                impact_adj = 0
                if sensitivity.get("max_sensitivity") == "special_category":
                    impact_adj = 1
                elif sensitivity.get("max_sensitivity") == "high":
                    impact_adj = 0.5
                if sensitivity.get("category_count", 0) > 8:
                    likelihood_adj = 1

                likelihood = min(5, rule["likelihood_base"] + likelihood_adj)
                impact = min(5, rule["impact_base"] + impact_adj)
                risk_score = round(likelihood * impact, 1)

                risks.append({
                    "risk_id": rule["id"],
                    "category": rule["category"],
                    "description": rule["description"],
                    "likelihood": likelihood,
                    "impact": impact,
                    "risk_score": risk_score,
                    "severity": _risk_severity(risk_score),
                    "recommended_mitigation": rule["mitigation"],
                })

        # Sort by risk score descending
        risks.sort(key=lambda r: r["risk_score"], reverse=True)

        # Calculate summary
        severity_counts = {}
        for r in risks:
            severity_counts[r["severity"]] = severity_counts.get(r["severity"], 0) + 1

        overall = "LOW"
        if severity_counts.get("CRITICAL", 0) > 0:
            overall = "CRITICAL"
        elif severity_counts.get("HIGH", 0) > 0:
            overall = "HIGH"
        elif severity_counts.get("MEDIUM", 0) > 0:
            overall = "MEDIUM"

        report = {
            "activity_id": activity_id,
            "activity_name": activity["name"],
            "assessment_type": assessment_type,
            "assessed_at": datetime.utcnow().isoformat(),
            "overall_risk_level": overall,
            "total_risks": len(risks),
            "risk_count_by_severity": severity_counts,
            "risks": risks,
            "data_sensitivity_profile": sensitivity,
        }

        self.risk_reports[activity_id] = report
        activity["risk_assessment_completed"] = True
        activity["risks_prioritized"] = True
        activity["risk_responses_identified"] = True

        return report

    # ------------------------------------------------------------------
    # ICO DPIA screening checklist
    # ------------------------------------------------------------------
    def run_screening_checklist(self, uses_special_category_data=False,
                                large_scale_processing=False,
                                systematic_monitoring=False,
                                automated_decision_making=False,
                                cross_border_transfer=False,
                                vulnerable_data_subjects=False,
                                innovative_technology=False,
                                denial_of_service_or_rights=False,
                                evaluation_or_scoring=False,
                                matching_or_combining_datasets=False):
        """Run ICO DPIA screening checklist to determine if full DPIA is required.

        A DPIA is required if at least two criteria are met, or if any single
        high-impact criterion is triggered.
        """
        criteria = {
            "special_category_data": uses_special_category_data,
            "large_scale_processing": large_scale_processing,
            "systematic_monitoring": systematic_monitoring,
            "automated_decision_making": automated_decision_making,
            "cross_border_transfer": cross_border_transfer,
            "vulnerable_data_subjects": vulnerable_data_subjects,
            "innovative_technology": innovative_technology,
            "denial_of_service_or_rights": denial_of_service_or_rights,
            "evaluation_or_scoring": evaluation_or_scoring,
            "matching_or_combining_datasets": matching_or_combining_datasets,
        }

        triggers = [name for name, value in criteria.items() if value]
        high_impact_single = {"special_category_data", "large_scale_processing",
                              "denial_of_service_or_rights"}
        dpia_required = (len(triggers) >= 2 or
                         bool(set(triggers) & high_impact_single))

        return {
            "dpia_required": dpia_required,
            "triggers": triggers,
            "trigger_count": len(triggers),
            "criteria_evaluated": len(criteria),
            "recommendation": (
                "Full DPIA must be conducted before processing begins"
                if dpia_required
                else "Full DPIA not mandatory but recommended as good practice"
            ),
            "ico_reference": "https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/accountability-and-governance/data-protection-impact-assessments-dpias/",
        }

    # ------------------------------------------------------------------
    # GDPR compliance check
    # ------------------------------------------------------------------
    def check_gdpr_compliance(self, activity_id):
        """Run GDPR article-level compliance checks against a processing activity."""
        if activity_id not in self.activities:
            raise ValueError(f"Activity {activity_id} not found")

        activity = self.activities[activity_id]
        findings = []
        total_checks = 0
        passed = 0

        for article_num, article_def in GDPR_DPIA_ARTICLES.items():
            for check in article_def["checks"]:
                total_checks += 1
                check_id = check["id"]
                desc = check["desc"]

                # Determine if check is applicable
                applicable = True
                if "condition_field" in check:
                    cond_field = check["condition_field"]
                    cond_value = check["condition_value"]
                    actual = activity.get(cond_field)
                    if cond_value == "_nonempty":
                        applicable = bool(actual)
                    else:
                        applicable = actual == cond_value

                if not applicable:
                    findings.append({
                        "article": check_id,
                        "description": desc,
                        "status": "NOT_APPLICABLE",
                        "details": f"Condition not met: {check.get('condition_field', '')}",
                    })
                    passed += 1
                    continue

                # Check field value
                field = check.get("field", "")
                value = activity.get(field)

                if "valid_values" in check:
                    if value in check["valid_values"]:
                        status = "COMPLIANT"
                        passed += 1
                    else:
                        status = "NON_COMPLIANT"
                elif check.get("required"):
                    if value:
                        status = "COMPLIANT"
                        passed += 1
                    else:
                        status = "NON_COMPLIANT"
                else:
                    if value:
                        status = "COMPLIANT"
                        passed += 1
                    else:
                        status = "NON_COMPLIANT"

                findings.append({
                    "article": check_id,
                    "description": desc,
                    "status": status,
                    "field": field,
                    "value_present": bool(value),
                })

        score = round((passed / total_checks) * 100, 1) if total_checks else 0

        report = {
            "activity_id": activity_id,
            "regulation": "GDPR",
            "checked_at": datetime.utcnow().isoformat(),
            "compliance_score": score,
            "total_checks": total_checks,
            "passed": passed,
            "failed": total_checks - passed,
            "findings": findings,
        }

        self.gdpr_reports[activity_id] = report
        return report

    # ------------------------------------------------------------------
    # CCPA/CPRA compliance check
    # ------------------------------------------------------------------
    def check_ccpa_compliance(self, activity_id):
        """Run CCPA/CPRA section-level compliance checks."""
        if activity_id not in self.activities:
            raise ValueError(f"Activity {activity_id} not found")

        activity = self.activities[activity_id]

        # Derive helper flags
        has_sensitive = any(
            DATA_SENSITIVITY.get(cat, {}).get("sensitivity") in ("special_category", "high")
            for cat in activity.get("data_categories", [])
        )
        activity["_has_sensitive_data"] = has_sensitive

        findings = []
        total_checks = 0
        passed = 0

        for section_num, section_def in CCPA_SECTIONS.items():
            for check in section_def["checks"]:
                total_checks += 1
                check_id = check["id"]
                desc = check["desc"]

                # Applicability
                applicable = True
                if "condition_field" in check:
                    cond_field = check["condition_field"]
                    cond_value = check["condition_value"]
                    actual = activity.get(cond_field)
                    applicable = actual == cond_value

                if not applicable:
                    findings.append({
                        "section": check_id,
                        "description": desc,
                        "status": "NOT_APPLICABLE",
                    })
                    passed += 1
                    continue

                field = check.get("field", "")
                value = activity.get(field)

                if "max_value" in check:
                    if value and isinstance(value, (int, float)) and value <= check["max_value"]:
                        status = "COMPLIANT"
                        passed += 1
                    elif not value:
                        status = "NON_COMPLIANT"
                    else:
                        status = "NON_COMPLIANT"
                elif check.get("required"):
                    if value:
                        status = "COMPLIANT"
                        passed += 1
                    else:
                        status = "NON_COMPLIANT"
                else:
                    if value:
                        status = "COMPLIANT"
                        passed += 1
                    else:
                        status = "NON_COMPLIANT"

                findings.append({
                    "section": check_id,
                    "description": desc,
                    "status": status,
                    "field": field,
                    "value_present": bool(value),
                })

        score = round((passed / total_checks) * 100, 1) if total_checks else 0

        report = {
            "activity_id": activity_id,
            "regulation": "CCPA/CPRA",
            "checked_at": datetime.utcnow().isoformat(),
            "compliance_score": score,
            "total_checks": total_checks,
            "passed": passed,
            "failed": total_checks - passed,
            "findings": findings,
        }

        self.ccpa_reports[activity_id] = report
        return report

    # ------------------------------------------------------------------
    # NIST Privacy Framework profile
    # ------------------------------------------------------------------
    def generate_nist_privacy_profile(self, activity_id, target_tier="tier_2"):
        """Generate NIST Privacy Framework profile for an activity."""
        if activity_id not in self.activities:
            raise ValueError(f"Activity {activity_id} not found")

        activity = self.activities[activity_id]
        profile = {
            "activity_id": activity_id,
            "target_tier": target_tier,
            "generated_at": datetime.utcnow().isoformat(),
            "functions": {},
        }

        total_subcats = 0
        implemented = 0

        for func_id, func_def in NIST_PRIVACY_FUNCTIONS.items():
            func_outcomes = []
            for cat_id, cat_def in func_def["categories"].items():
                for subcat in cat_def["subcategories"]:
                    total_subcats += 1
                    check_field = subcat["check"]
                    is_implemented = bool(activity.get(check_field))
                    if is_implemented:
                        implemented += 1

                    func_outcomes.append({
                        "subcategory": subcat["id"],
                        "description": subcat["desc"],
                        "category": cat_def["name"],
                        "implemented": is_implemented,
                        "check_field": check_field,
                    })

            profile["functions"][func_id] = func_outcomes

        profile["coverage"] = {
            "total_subcategories": total_subcats,
            "implemented": implemented,
            "gaps": total_subcats - implemented,
            "coverage_percentage": round((implemented / total_subcats) * 100, 1) if total_subcats else 0,
        }

        return profile

    # ------------------------------------------------------------------
    # Remediation plan
    # ------------------------------------------------------------------
    def generate_remediation_plan(self, activity_id, risk_report=None,
                                   gdpr_report=None, ccpa_report=None):
        """Generate a prioritized remediation plan from assessment results."""
        action_items = []
        now = datetime.utcnow()

        # From risk report
        if risk_report:
            for risk in risk_report.get("risks", []):
                priority_map = {"CRITICAL": "P1", "HIGH": "P2", "MEDIUM": "P3", "LOW": "P4"}
                deadline_days = {"CRITICAL": 14, "HIGH": 30, "MEDIUM": 60, "LOW": 90}
                sev = risk["severity"]
                action_items.append({
                    "source": "risk_assessment",
                    "priority": priority_map.get(sev, "P4"),
                    "action": risk["recommended_mitigation"],
                    "addresses_risks": [risk["risk_id"]],
                    "risk_category": risk["category"],
                    "risk_score": risk["risk_score"],
                    "owner": "Privacy Team",
                    "deadline": (now + timedelta(days=deadline_days.get(sev, 90))).strftime("%Y-%m-%d"),
                    "status": "open",
                })

        # From GDPR findings
        if gdpr_report:
            for finding in gdpr_report.get("findings", []):
                if finding["status"] == "NON_COMPLIANT":
                    action_items.append({
                        "source": "gdpr_compliance",
                        "priority": "P2",
                        "action": f"Address GDPR Art. {finding['article']} non-compliance: {finding['description']}",
                        "addresses_risks": [f"GDPR-{finding['article']}"],
                        "risk_category": "Regulatory Compliance",
                        "owner": "DPO / Legal",
                        "deadline": (now + timedelta(days=30)).strftime("%Y-%m-%d"),
                        "status": "open",
                    })

        # From CCPA findings
        if ccpa_report:
            for finding in ccpa_report.get("findings", []):
                if finding["status"] == "NON_COMPLIANT":
                    action_items.append({
                        "source": "ccpa_compliance",
                        "priority": "P2",
                        "action": f"Address CCPA Sec. {finding['section']} non-compliance: {finding['description']}",
                        "addresses_risks": [f"CCPA-{finding['section']}"],
                        "risk_category": "Regulatory Compliance",
                        "owner": "Privacy Team / Legal",
                        "deadline": (now + timedelta(days=30)).strftime("%Y-%m-%d"),
                        "status": "open",
                    })

        # Sort by priority
        priority_order = {"P1": 0, "P2": 1, "P3": 2, "P4": 3}
        action_items.sort(key=lambda x: priority_order.get(x["priority"], 9))

        return {
            "activity_id": activity_id,
            "generated_at": now.isoformat(),
            "total_actions": len(action_items),
            "by_priority": {
                p: sum(1 for a in action_items if a["priority"] == p)
                for p in ["P1", "P2", "P3", "P4"]
            },
            "action_items": action_items,
        }

    # ------------------------------------------------------------------
    # Report generation
    # ------------------------------------------------------------------
    def generate_dpia_report(self, activity_id, output_path="dpia_report.json",
                              format="json"):
        """Generate the formal DPIA report document."""
        if activity_id not in self.activities:
            raise ValueError(f"Activity {activity_id} not found")

        activity = self.activities[activity_id]
        report = {
            "report_type": "Data Protection Impact Assessment (DPIA)",
            "report_id": f"DPIA-{uuid.uuid4().hex[:8].upper()}",
            "generated_at": datetime.utcnow().isoformat(),
            "organization": self.organization_name,
            "dpo_contact": self.dpo_email,
            "processing_activity": {
                k: v for k, v in activity.items() if not k.startswith("_")
            },
            "data_flow_map": self.flow_maps.get(activity_id),
            "risk_assessment": self.risk_reports.get(activity_id),
            "gdpr_compliance": self.gdpr_reports.get(activity_id),
            "ccpa_compliance": self.ccpa_reports.get(activity_id),
            "dpo_sign_off": {
                "status": "pending",
                "dpo_name": "",
                "date": "",
                "comments": "",
            },
            "review_schedule": {
                "next_review": (datetime.utcnow() + timedelta(days=365)).strftime("%Y-%m-%d"),
                "review_frequency": "annual",
                "triggers_for_early_review": [
                    "Change in processing purpose",
                    "New data categories collected",
                    "Change in processor or subprocessor",
                    "Security incident or breach",
                    "Regulatory change",
                    "Complaints from data subjects",
                ],
            },
        }

        with open(output_path, "w") as f:
            json.dump(report, f, indent=2, default=str)

        return report


def main():
    parser = argparse.ArgumentParser(
        description="Privacy Impact Assessment (PIA/DPIA) Automation Agent"
    )
    parser.add_argument("--org", default="", help="Organization name")
    parser.add_argument("--dpo-email", default="", help="DPO email address")
    parser.add_argument("--output", default="pia_report.json", help="Output report path")
    parser.add_argument("--action", choices=[
        "screening", "full_assessment", "gdpr_check", "ccpa_check",
        "nist_profile", "demo",
    ], default="demo", help="Action to perform")
    args = parser.parse_args()

    engine = PrivacyImpactAssessmentEngine(
        organization_name=args.org,
        dpo_email=args.dpo_email,
    )

    if args.action == "screening":
        result = engine.run_screening_checklist(
            large_scale_processing=True,
            systematic_monitoring=True,
            automated_decision_making=True,
        )
        print(f"[+] DPIA Required: {result['dpia_required']}")
        print(f"    Triggers: {result['triggers']}")

    elif args.action == "demo":
        print("[*] Running demonstration PIA workflow...\n")

        # Register activity
        activity = engine.register_processing_activity(
            name="Customer Analytics Platform",
            description="Collects browsing behavior and purchase history for personalization",
            data_controller="Demo Corp",
            data_processor="CloudAnalytics Inc",
            data_categories=["browsing_history", "purchase_records", "ip_address",
                             "device_id", "email", "name"],
            data_subjects=["customers", "website_visitors"],
            legal_basis="consent",
            retention_period_days=730,
            cross_border_transfer=True,
            transfer_destinations=["US", "IN"],
            automated_decision_making=True,
        )
        print(f"[+] Registered activity: {activity['activity_id']}")

        # Map data flows
        flow_map = engine.map_data_flows(
            activity_id=activity["activity_id"],
            flows=[
                {"stage": "collection", "source": "Web browser",
                 "destination": "CDN edge", "data_elements": ["ip_address", "device_id"],
                 "encryption_in_transit": True, "protocol": "TLS 1.3"},
                {"stage": "processing", "source": "CDN edge",
                 "destination": "Data warehouse (US)", "data_elements": ["browsing_history", "purchase_records"],
                 "encryption_in_transit": True, "encryption_at_rest": True, "protocol": "mTLS"},
                {"stage": "sharing", "source": "Data warehouse",
                 "destination": "ML Provider (IN)", "data_elements": ["browsing_history"],
                 "encryption_in_transit": True, "cross_border": True,
                 "data_processing_agreement": True},
                {"stage": "deletion", "source": "All stores",
                 "destination": "Secure erasure", "method": "Cryptographic erasure"},
            ],
        )
        engine.render_data_flow_diagram(flow_map)

        # Risk assessment
        risk_report = engine.assess_privacy_risks(activity_id=activity["activity_id"])
        print(f"[+] Risk Assessment: {risk_report['overall_risk_level']}")
        print(f"    Total Risks: {risk_report['total_risks']}")
        for risk in risk_report["risks"][:5]:
            print(f"    [{risk['severity']}] {risk['category']}: Score {risk['risk_score']}/25")

        # GDPR check
        gdpr = engine.check_gdpr_compliance(activity_id=activity["activity_id"])
        print(f"\n[+] GDPR Compliance Score: {gdpr['compliance_score']}/100")
        for f in gdpr["findings"]:
            if f["status"] == "NON_COMPLIANT":
                print(f"    [FAIL] Art.{f['article']}: {f['description']}")

        # CCPA check
        ccpa = engine.check_ccpa_compliance(activity_id=activity["activity_id"])
        print(f"\n[+] CCPA Compliance Score: {ccpa['compliance_score']}/100")
        for f in ccpa["findings"]:
            if f["status"] == "NON_COMPLIANT":
                print(f"    [FAIL] Sec.{f['section']}: {f['description']}")

        # Remediation
        remediation = engine.generate_remediation_plan(
            activity_id=activity["activity_id"],
            risk_report=risk_report,
            gdpr_report=gdpr,
            ccpa_report=ccpa,
        )
        print(f"\n[+] Remediation Plan: {remediation['total_actions']} action items")
        print(f"    By Priority: {remediation['by_priority']}")

        # Generate report
        engine.generate_dpia_report(
            activity_id=activity["activity_id"],
            output_path=args.output,
        )
        print(f"\n[+] DPIA report saved to {args.output}")

    else:
        print(f"[!] Action '{args.action}' requires an activity to be registered first.")
        print("    Use --action demo for a full demonstration workflow.")


if __name__ == "__main__":
    main()

===== END FILE =====
