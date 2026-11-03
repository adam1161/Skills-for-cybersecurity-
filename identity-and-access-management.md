

===== FILE: skills/implementing-hardware-security-key-authentication/SKILL.md =====
---
name: implementing-hardware-security-key-authentication
description: 'Builds a FIDO2/WebAuthn relying party server with the python-fido2
  library, covering registration and authentication ceremonies, YubiKey enrollment,
  resident key (discoverable credential/passkey) workflows, and user verification
  policies. Use when implementing phishing-resistant MFA with hardware security keys,
  building a WebAuthn relying party, enrolling YubiKeys for a workforce, or migrating
  password-based authentication to passkeys.

  '
domain: cybersecurity
subdomain: identity-and-access-management
tags:
- FIDO2
- WebAuthn
- hardware-security-key
- YubiKey
- passkeys
- passwordless-authentication
- CTAP2
version: 1.0.0
author: mukul975
license: Apache-2.0
atlas_techniques:
- AML.T0051
- AML.T0054
- AML.T0056
nist_ai_rmf:
- MEASURE-2.7
- MEASURE-2.5
- GOVERN-6.1
- MAP-5.1
nist_csf:
- PR.AA-01
- PR.AA-02
- PR.AA-05
mitre_attack:
- T1078
- T1190
- T1059
- T1003
- T1110
---
# Implementing Hardware Security Key Authentication

## When to Use

- Deploying phishing-resistant multi-factor authentication (MFA) using FIDO2 hardware security keys for high-value accounts (administrators, developers, privileged users)
- Building a WebAuthn relying party server that supports both roaming authenticators (USB/NFC security keys) and platform authenticators (Windows Hello, Touch ID, Android biometrics)
- Migrating an existing password-based authentication system to support passkeys (discoverable credentials) as a primary or secondary authentication factor
- Enrolling YubiKey devices for an organization's workforce, including PIN setup, credential registration, and backup key provisioning
- Implementing passwordless authentication flows that comply with NIST SP 800-63B AAL3 (authenticator assurance level 3) requirements

**Do not use** without HTTPS in production (WebAuthn requires a secure origin), for systems where users cannot physically access a USB/NFC port, or as the sole authentication factor without a recovery mechanism for lost keys.

## Prerequisites

- Python 3.10+ with `fido2` (python-fido2 >= 2.0.0), `flask`, and `cryptography` libraries installed
- HTTPS-enabled web server (WebAuthn API requires secure context; localhost is exempt for development)
- FIDO2-compatible hardware security key (YubiKey 5 Series, SoloKeys, Titan Security Key) or platform authenticator
- Modern web browser supporting the WebAuthn API (Chrome 67+, Firefox 60+, Safari 14+, Edge 79+)
- Understanding of public key cryptography, challenge-response protocols, and HTTP session management

## Workflow

### Step 1: Relying Party Server Configuration

Configure the WebAuthn relying party (RP) identity and server:

- **Define RP identity**: Create a `PublicKeyCredentialRpEntity` with the relying party name (display name shown to users) and RP ID (the effective domain of the application). The RP ID must be a registrable domain suffix of the origin -- for example, `example.com` is valid for `https://auth.example.com` but `other.com` is not.
- **Initialize Fido2Server**: Instantiate the `Fido2Server` class from the python-fido2 library with the RP entity. The server handles challenge generation, attestation verification, and assertion validation.
- **Configure attestation preference**: Set the attestation conveyance preference to control whether the server requests proof of the authenticator's identity:
  - `none`: No attestation requested (simplest, recommended for most deployments)
  - `indirect`: Attestation may be provided but CA may anonymize it
  - `direct`: Full attestation chain from the authenticator's manufacturer
  - `enterprise`: Device-identifying attestation for managed environments
- **Session management**: Configure server-side sessions to store WebAuthn state between the begin and complete phases of registration/authentication ceremonies. Use secure, httponly cookies with SameSite=Strict.
- **Credential storage**: Design the database schema to store credential records: `credential_id` (binary), `public_key` (COSE key), `sign_count` (uint32 for clone detection), `user_id`, `created_at`, `last_used`, `display_name`, and `transports` (USB, NFC, BLE, internal).

### Step 2: Registration Ceremony (Credential Creation)

Implement the WebAuthn registration flow to create new credentials:

- **Begin registration**: Call `server.register_begin()` with the user entity (`PublicKeyCredentialUserEntity` containing user ID, username, and display name), the list of existing credentials for the user (to prevent duplicate registration), and options for `user_verification` and `authenticator_attachment`.
- **Authenticator selection criteria**:
  - `authenticator_attachment: cross-platform` restricts to roaming authenticators (USB/NFC keys)
  - `authenticator_attachment: platform` restricts to built-in authenticators (Touch ID, Windows Hello)
  - Omitting this field allows both types
  - `resident_key: required` forces creation of a discoverable credential (passkey) stored on the authenticator
  - `user_verification: required` enforces PIN or biometric verification on the authenticator
- **Client-side ceremony**: The browser calls `navigator.credentials.create()` with the options from the server. The authenticator generates a new key pair, stores the private key in its secure element, and returns the public key, credential ID, attestation object, and client data JSON.
- **Complete registration**: Call `server.register_complete()` with the saved state and the client response. The server verifies the attestation signature, extracts the credential public key and ID, and returns `AuthenticatorData` containing the credential data to store.
- **Store credential**: Persist the `credential_data` (contains `credential_id`, `public_key` as COSE key, and `sign_count`) to the database associated with the user account.

### Step 3: Authentication Ceremony (Assertion)

Implement the WebAuthn authentication flow to verify credentials:

- **Begin authentication**: Call `server.authenticate_begin()` with the list of registered credentials for the user (or omit for discoverable credential flows where the authenticator identifies the user). Set `user_verification` based on the assurance level required.
- **Client-side assertion**: The browser calls `navigator.credentials.get()` with the server options. The authenticator locates the matching credential, performs user verification if required, increments the signature counter, and signs the challenge with the private key.
- **Complete authentication**: Call `server.authenticate_complete()` with the saved state, registered credentials, and the client response. The server verifies the assertion signature against the stored public key and validates the signature counter has incremented (clone detection).
- **Update sign count**: After successful authentication, update the stored `sign_count` for the credential. If the new sign count is not greater than the stored value, the key may have been cloned -- flag this as a security event.
- **Discoverable credential flow**: For passwordless authentication, the user does not need to enter a username first. The authenticator presents all discoverable credentials for the RP ID, and the selected credential's `userHandle` identifies the user.

### Step 4: YubiKey Enrollment and Management

Implement organizational YubiKey provisioning workflows:

- **PIN initialization**: Before first use, a YubiKey requires a FIDO2 PIN (minimum 4 characters, 8 retries before lockout). Guide users through PIN setup using the Yubico Authenticator application or programmatically via the CTAP2 `clientPin` command.
- **Primary key enrollment**: Register the user's primary YubiKey with their account. Store the credential with a user-friendly label (e.g., "USB-A YubiKey - Office") and the authenticator's AAGUID for device identification.
- **Backup key enrollment**: Require users to register at least two security keys. The backup key should be stored separately (home, safety deposit box). Both keys must be registered to the same account so either can authenticate.
- **Key attestation verification**: For enterprise deployments, verify the attestation certificate chain to confirm the key is a genuine YubiKey from Yubico. Compare the AAGUID against Yubico's published values to identify the exact model.
- **Key lifecycle management**: Implement administrative functions to list a user's registered keys, revoke compromised keys, force re-enrollment, and audit key usage patterns (last authentication time, total authentications).

### Step 5: Passkey Migration Strategy

Plan and execute migration from passwords to passkeys:

- **Phased rollout**: Begin with voluntary passkey enrollment alongside existing password authentication. Track adoption metrics (percentage of users with passkeys, percentage of logins using passkeys vs. passwords).
- **Credential upgrade flow**: When a user authenticates with a password, prompt them to register a passkey. Present the WebAuthn registration dialog immediately after successful password login to minimize friction.
- **Cross-device passkeys**: Support synced passkeys (passkeys stored in platform credential managers like iCloud Keychain, Google Password Manager, or 1Password) for users who do not have hardware security keys. These provide phishing resistance without requiring dedicated hardware.
- **Account recovery**: Design recovery flows for users who lose all their security keys:
  - Recovery codes generated at enrollment time (printed, stored in password manager)
  - Supervised re-enrollment by an administrator after identity verification
  - Temporary time-limited password login with mandatory key re-enrollment
  - Never allow recovery via email or SMS alone, as these defeat the phishing resistance
- **Password deprecation timeline**: After passkey adoption exceeds the target threshold, enforce passkey-only authentication for high-privilege accounts first, then expand to all accounts. Maintain password as a fallback during the transition window.
- **Monitoring and metrics**: Track registration success rates, authentication failure rates (wrong key, timeout, cancelled), mean time to authenticate, and the ratio of passkey to password logins.

## Key Concepts

| Term | Definition |
|------|------------|
| **FIDO2** | An umbrella term for the combination of the W3C WebAuthn API and the FIDO Alliance CTAP2 protocol, enabling passwordless and phishing-resistant authentication using public key cryptography |
| **WebAuthn** | The W3C Web Authentication API that allows web applications to create and use public key credentials via `navigator.credentials.create()` (registration) and `navigator.credentials.get()` (authentication) |
| **CTAP2** | Client to Authenticator Protocol version 2; the protocol used by the browser (client) to communicate with external authenticators over USB, NFC, or BLE |
| **Relying Party (RP)** | The web application or service that requests authentication; identified by its RP ID (a domain) and RP name (display string) |
| **Discoverable Credential (Passkey)** | A credential stored on the authenticator that can be enumerated without the RP providing a credential ID, enabling username-less authentication flows |
| **Attestation** | Cryptographic proof from the authenticator about its identity and properties; used by the RP to verify the authenticator model and manufacturer |
| **AAGUID** | Authenticator Attestation Globally Unique Identifier; a 128-bit value identifying the authenticator model (e.g., all YubiKey 5 NFC devices share the same AAGUID) |
| **Sign Count** | A monotonically increasing counter maintained by the authenticator and included in each assertion; used by the RP to detect cloned authenticators |
| **User Verification (UV)** | Local authentication on the authenticator itself (PIN, fingerprint, face recognition) that proves the person holding the authenticator is the legitimate owner |

## Tools & Systems

- **python-fido2**: Yubico's official Python library (v2.0+) providing `Fido2Server` for relying party implementation and `CtapHidDevice`/`Fido2Client` for direct authenticator communication over USB
- **YubiKey 5 Series**: Yubico hardware security keys supporting FIDO2/CTAP2, U2F, PIV, OpenPGP, and OTP; available in USB-A, USB-C, NFC, and Nano form factors
- **py_webauthn**: Duo Labs' Python WebAuthn library providing `generate_registration_options()`, `verify_registration_response()`, `generate_authentication_options()`, and `verify_authentication_response()` functions
- **Yubico Authenticator**: Desktop and mobile application for managing YubiKey FIDO2 credentials, setting PINs, and viewing registered accounts
- **WebAuthn.io / demo.yubico.com**: Online testing tools for verifying WebAuthn registration and authentication flows against real authenticators

## Common Scenarios

### Scenario: Deploying FIDO2 MFA for a Development Team

**Context**: A software company wants to replace TOTP-based MFA with hardware security keys for its 50-person development team. Developers have root access to production infrastructure and are high-value targets for phishing attacks. The company has standardized on YubiKey 5 NFC.

**Approach**:
1. Provision YubiKey 5 NFC keys (2 per developer: primary + backup) and distribute in tamper-evident packaging with initial PIN setup instructions
2. Deploy the WebAuthn relying party server integrated with the company's SSO (OAuth 2.0 / OpenID Connect) provider, configured with `authenticator_attachment: cross-platform` and `user_verification: required`
3. Run enrollment sessions where each developer registers both keys to their account, with attestation verification confirming genuine YubiKey 5 NFC AAGUIDs
4. Configure the SSO provider to require FIDO2 as the second factor for all developer accounts, with a 30-day grace period where TOTP remains available
5. Implement a self-service portal for key management: view registered keys, register replacement keys, and report lost/stolen keys (which triggers immediate credential revocation and re-enrollment)
6. After the grace period, disable TOTP for developer accounts. Monitor authentication logs for any fallback attempts and provide 1:1 support for remaining holdouts
7. Achieve 100% FIDO2 adoption for the development team, reducing phishing risk to near-zero for production infrastructure access

**Pitfalls**:
- Not requiring backup key enrollment, leading to account lockouts when a single key is lost
- Setting `user_verification: discouraged` which allows anyone who physically possesses the key to authenticate without a PIN
- Forgetting to validate the sign counter, missing cloned key attacks
- Not supporting NFC for developers who primarily work from tablets or phones
- Allowing TOTP as a permanent fallback, which undermines the phishing resistance of the FIDO2 deployment

### Scenario: Implementing Passwordless Login for a Customer-Facing Application

**Context**: An e-commerce platform wants to offer passkey-based passwordless login to its 2 million users as an alternative to passwords, reducing account takeover from credential stuffing and phishing.

**Approach**:
1. Implement WebAuthn with `resident_key: required` to create discoverable credentials that enable username-less login
2. Support both platform authenticators (Touch ID, Windows Hello, Android biometrics) and roaming authenticators (security keys) by omitting `authenticator_attachment`
3. Add a "Sign in with a passkey" button to the login page that triggers `navigator.credentials.get()` with an empty `allowCredentials` list, prompting the authenticator to present available passkeys
4. After successful passkey creation, prompt users to create a passkey on a second device for redundancy
5. Maintain password login as a fallback during the rollout period, with a persistent prompt encouraging passkey setup after each password login
6. Track metrics: passkey registration rate (target 30% in first quarter), passkey vs. password login ratio, authentication failure rates, and account takeover incidents
7. After 6 months, offer incentives (extended session duration, reduced CAPTCHA) for users who switch to passkey-only authentication

**Pitfalls**:
- Not handling the case where a user's platform authenticator (e.g., laptop Touch ID) is unavailable and they need cross-device authentication via QR code
- Assuming all users have biometric-capable devices; some will need to fall back to PIN-based verification
- Not implementing proper account recovery for users who lose access to all registered passkeys
- Ignoring browser compatibility gaps, particularly in older Safari versions on iOS

## Output Format

```
## FIDO2 Deployment Report

**Application**: auth.example.com
**RP ID**: example.com
**Date**: 2026-03-19

### Enrollment Summary
- **Total Users**: 50
- **Users with Primary Key**: 50 (100%)
- **Users with Backup Key**: 47 (94%)
- **Authenticator Models**: YubiKey 5 NFC (48), YubiKey 5C NFC (2)

### Authentication Metrics (Last 30 Days)
- **Total Authentications**: 12,847
- **FIDO2 Authentications**: 12,203 (95.0%)
- **TOTP Fallback**: 644 (5.0%) -- grace period active
- **Mean Authentication Time**: 2.3 seconds
- **Authentication Failures**: 127 (0.99%)
  - User cancelled: 89
  - Timeout: 23
  - Invalid signature: 12
  - Sign count regression (possible clone): 3

### Security Events
- **Lost Key Reports**: 2
  - User A: primary key lost 2026-03-12, revoked, backup promoted, new backup enrolled
  - User B: backup key damaged 2026-03-15, revoked, replacement enrolled

### Credential Details
| User | Key Label | AAGUID | Registered | Last Used | Sign Count |
|------|-----------|--------|------------|-----------|------------|
| alice | YubiKey Primary | 2fc0579f... | 2026-02-15 | 2026-03-19 | 847 |
| alice | YubiKey Backup | 2fc0579f... | 2026-02-15 | 2026-03-01 | 12 |
| bob | YubiKey Primary | 2fc0579f... | 2026-02-16 | 2026-03-19 | 631 |
```

===== END FILE =====


===== FILE: skills/implementing-hardware-security-key-authentication/LICENSE =====

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


===== FILE: skills/implementing-hardware-security-key-authentication/references/api-reference.md =====
# API Reference: FIDO2/WebAuthn Hardware Security Key Authentication Server

## Overview

Implements a complete WebAuthn relying party server with registration/authentication ceremonies, YubiKey enrollment, credential management, recovery codes, and audit logging. Built on python-fido2 and Flask, supporting both roaming authenticators (USB/NFC security keys) and platform authenticators (Windows Hello, Touch ID).

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| fido2 | >=2.0.0 | Yubico's python-fido2 library for WebAuthn relying party operations |
| flask | >=2.3 | HTTP server framework for API endpoints and session management |
| cryptography | >=41.0 | Cryptographic primitives used by python-fido2 |

Install with: `pip install fido2 flask cryptography`

## CLI Usage

```bash
# Development server on localhost (no TLS needed)
python agent.py --rp-id localhost --rp-name "My App" --port 5000

# Production with strict security settings
python agent.py --rp-id auth.example.com --rp-name "Example Corp" \
    --user-verification required --attestation direct --db prod_keys.db

# Verbose logging for debugging
python agent.py --rp-id localhost --rp-name "Test" -v
```

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--rp-id` | No | Relying Party ID -- must match the application domain (default: localhost) |
| `--rp-name` | No | Relying Party display name shown in authenticator prompts (default: FIDO2 Demo) |
| `--host` | No | Server bind address (default: localhost) |
| `--port` | No | Server port (default: 5000) |
| `--db` | No | SQLite database path for credentials and users (default: webauthn.db) |
| `--attestation` | No | Attestation preference: none, indirect, direct, enterprise (default: none) |
| `--user-verification` | No | UV requirement: required, preferred, discouraged (default: preferred) |
| `-v, --verbose` | No | Enable debug logging |

## API Endpoints

### Registration

#### `POST /api/register/begin`
Start the WebAuthn registration ceremony.

**Request body:**
```json
{
  "username": "alice",
  "display_name": "Alice Smith",
  "resident_key": true
}
```

**Response:** PublicKeyCredentialCreationOptions (JSON-serialized)

#### `POST /api/register/complete`
Complete registration with the authenticator's response.

**Request body:** Serialized PublicKeyCredential from `navigator.credentials.create()`

**Response:**
```json
{
  "status": "OK",
  "recovery_codes": ["A1B2C3D4", "E5F6G7H8", ...],
  "message": "Save these recovery codes securely."
}
```

### Authentication

#### `POST /api/authenticate/begin`
Start the WebAuthn authentication ceremony.

**Request body:**
```json
{
  "username": "alice"
}
```
Omit `username` for discoverable credential (passwordless) flow.

**Response:** PublicKeyCredentialRequestOptions (JSON-serialized)

#### `POST /api/authenticate/complete`
Complete authentication with the authenticator's assertion.

**Request body:** Serialized PublicKeyCredential from `navigator.credentials.get()`

**Response:**
```json
{
  "status": "OK",
  "username": "alice"
}
```

### Key Management

#### `GET /api/keys`
List all registered security keys for the authenticated user.

#### `POST /api/keys/<id>/revoke`
Revoke a security key. Requires at least one remaining active credential.

#### `PUT /api/keys/<id>/label`
Update a key's display label.

### Recovery

#### `POST /api/recover`
Authenticate using a recovery code when all keys are unavailable.

**Request body:**
```json
{
  "username": "alice",
  "recovery_code": "A1B2C3D4"
}
```

### Admin

#### `GET /api/admin/stats`
Deployment statistics: total users, credentials, backup key adoption, authentication metrics.

#### `GET /api/admin/audit-log?limit=100`
Authentication event audit trail with timestamps, usernames, event types, and IP addresses.

## Key Functions

### User Management

#### `create_user(conn, username, display_name)`
Creates a user with a cryptographically random 32-byte user handle.

#### `get_user_by_username(conn, username)` / `get_user_by_handle(conn, user_handle)`
User lookups by username (standard flow) or user handle (discoverable credential flow).

### Credential Management

#### `store_credential(conn, user_id, credential_id, public_key, sign_count, ...)`
Persists a WebAuthn credential with AAGUID, label, transport hints, and discoverable flag.

#### `get_user_credentials(conn, user_id)`
Retrieves all active (non-revoked) credentials for a user.

#### `revoke_credential(conn, credential_db_id, user_id)`
Soft-revokes a credential. Prevents revocation of the last remaining credential.

#### `update_sign_count(conn, credential_id, new_count)`
Updates sign count and last-used timestamp after successful authentication. Sign count regression is logged as a security event (possible cloned key).

### Recovery

#### `generate_recovery_codes(conn, user_id, count=8)`
Generates 8 one-time recovery codes (stored as SHA-256 hashes). Previous codes are invalidated.

#### `verify_recovery_code(conn, user_id, code)`
Verifies and consumes a recovery code (single-use).

### Helpers

#### `build_credential_descriptors(creds)`
Converts stored credentials to `PublicKeyCredentialDescriptor` list for WebAuthn ceremonies.

#### `reconstruct_credential_data(creds)`
Rebuilds `AttestedCredentialData` objects from database records for python-fido2 verification.

## Database Schema

| Table | Purpose |
|-------|---------|
| `users` | User accounts with random user handles, usernames, and passkey-only flag |
| `credentials` | WebAuthn credentials with public keys, sign counts, AAGUIDs, and revocation status |
| `auth_events` | Audit log of all registration, authentication, revocation, and recovery events |
| `recovery_codes` | Hashed one-time recovery codes with usage tracking |

## Security Features

| Feature | Implementation |
|---------|---------------|
| Phishing resistance | Origin binding via WebAuthn RP ID verification |
| Clone detection | Sign count validation with regression warnings |
| Recovery codes | SHA-256 hashed, single-use, auto-invalidated on regeneration |
| Session security | Secure, HttpOnly, SameSite=Strict cookies |
| Key revocation | Soft delete with minimum-one-key enforcement |
| Audit trail | All auth events logged with IP, user agent, and timestamp |
| Attestation verification | Configurable attestation preference for enterprise key verification |

===== END FILE =====


===== FILE: skills/implementing-hardware-security-key-authentication/scripts/agent.py =====
#!/usr/bin/env python3
"""FIDO2/WebAuthn Hardware Security Key Authentication Server.

Implements a complete WebAuthn relying party with registration ceremonies,
authentication flows, YubiKey enrollment management, and passkey support
using the python-fido2 library.

For authorized deployment and security testing only.
"""

import argparse
import hashlib
import json
import logging
import os
import secrets
import sqlite3
import sys
import time
from base64 import urlsafe_b64decode, urlsafe_b64encode
from datetime import datetime, timezone
from pathlib import Path

from flask import Flask, abort, jsonify, redirect, request, session, render_template_string

from fido2.server import Fido2Server
from fido2.webauthn import (
    AttestationConveyancePreference,
    AuthenticatorAttachment,
    AuthenticatorSelectionCriteria,
    PublicKeyCredentialDescriptor,
    PublicKeyCredentialRpEntity,
    PublicKeyCredentialUserEntity,
    ResidentKeyRequirement,
    UserVerificationRequirement,
)

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[logging.StreamHandler(sys.stdout)],
)
logger = logging.getLogger(__name__)


# ---------------------------------------------------------------------------
# Database layer
# ---------------------------------------------------------------------------

def init_database(db_path: str) -> sqlite3.Connection:
    """Initialize SQLite database for credential and user storage."""
    conn = sqlite3.connect(db_path, check_same_thread=False)
    conn.execute("PRAGMA journal_mode=WAL")
    conn.execute("PRAGMA foreign_keys=ON")
    conn.executescript("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_handle BLOB UNIQUE NOT NULL,
            username TEXT UNIQUE NOT NULL,
            display_name TEXT NOT NULL,
            created_at TEXT NOT NULL DEFAULT (datetime('now')),
            passkey_only INTEGER DEFAULT 0
        );

        CREATE TABLE IF NOT EXISTS credentials (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER NOT NULL,
            credential_id BLOB UNIQUE NOT NULL,
            public_key BLOB NOT NULL,
            sign_count INTEGER NOT NULL DEFAULT 0,
            aaguid TEXT,
            label TEXT,
            transports TEXT,
            created_at TEXT NOT NULL DEFAULT (datetime('now')),
            last_used TEXT,
            is_discoverable INTEGER DEFAULT 0,
            is_revoked INTEGER DEFAULT 0,
            FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
        );

        CREATE TABLE IF NOT EXISTS auth_events (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER NOT NULL,
            credential_id BLOB,
            event_type TEXT NOT NULL,
            success INTEGER NOT NULL,
            ip_address TEXT,
            user_agent TEXT,
            details TEXT,
            created_at TEXT NOT NULL DEFAULT (datetime('now')),
            FOREIGN KEY (user_id) REFERENCES users(id)
        );

        CREATE TABLE IF NOT EXISTS recovery_codes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER NOT NULL,
            code_hash TEXT UNIQUE NOT NULL,
            used INTEGER DEFAULT 0,
            created_at TEXT NOT NULL DEFAULT (datetime('now')),
            FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
        );

        CREATE INDEX IF NOT EXISTS idx_creds_user ON credentials(user_id);
        CREATE INDEX IF NOT EXISTS idx_creds_cred_id ON credentials(credential_id);
        CREATE INDEX IF NOT EXISTS idx_events_user ON auth_events(user_id);
    """)
    conn.commit()
    return conn


# ---------------------------------------------------------------------------
# User management
# ---------------------------------------------------------------------------

def create_user(conn: sqlite3.Connection, username: str, display_name: str) -> dict:
    """Create a new user with a random user handle."""
    user_handle = secrets.token_bytes(32)
    try:
        conn.execute(
            "INSERT INTO users (user_handle, username, display_name) VALUES (?, ?, ?)",
            (user_handle, username, display_name),
        )
        conn.commit()
        user_id = conn.execute(
            "SELECT id FROM users WHERE username = ?", (username,)
        ).fetchone()[0]
        logger.info("Created user: %s (ID: %d)", username, user_id)
        return {
            "id": user_id,
            "user_handle": user_handle,
            "username": username,
            "display_name": display_name,
        }
    except sqlite3.IntegrityError:
        logger.warning("User already exists: %s", username)
        return get_user_by_username(conn, username)


def get_user_by_username(conn: sqlite3.Connection, username: str) -> dict | None:
    """Retrieve user by username."""
    row = conn.execute(
        "SELECT id, user_handle, username, display_name, passkey_only FROM users WHERE username = ?",
        (username,),
    ).fetchone()
    if not row:
        return None
    return {
        "id": row[0],
        "user_handle": row[1],
        "username": row[2],
        "display_name": row[3],
        "passkey_only": bool(row[4]),
    }


def get_user_by_handle(conn: sqlite3.Connection, user_handle: bytes) -> dict | None:
    """Retrieve user by user handle (for discoverable credential flows)."""
    row = conn.execute(
        "SELECT id, user_handle, username, display_name, passkey_only FROM users WHERE user_handle = ?",
        (user_handle,),
    ).fetchone()
    if not row:
        return None
    return {
        "id": row[0],
        "user_handle": row[1],
        "username": row[2],
        "display_name": row[3],
        "passkey_only": bool(row[4]),
    }


# ---------------------------------------------------------------------------
# Credential management
# ---------------------------------------------------------------------------

def store_credential(
    conn: sqlite3.Connection,
    user_id: int,
    credential_id: bytes,
    public_key: bytes,
    sign_count: int,
    aaguid: str = None,
    label: str = None,
    transports: list[str] = None,
    is_discoverable: bool = False,
) -> int:
    """Store a new WebAuthn credential in the database."""
    cursor = conn.execute(
        """INSERT INTO credentials
           (user_id, credential_id, public_key, sign_count, aaguid, label,
            transports, is_discoverable)
           VALUES (?, ?, ?, ?, ?, ?, ?, ?)""",
        (
            user_id, credential_id, public_key, sign_count,
            aaguid, label,
            json.dumps(transports) if transports else None,
            1 if is_discoverable else 0,
        ),
    )
    conn.commit()
    cred_id = cursor.lastrowid
    logger.info(
        "Stored credential for user %d: %s (label: %s)",
        user_id, urlsafe_b64encode(credential_id).decode(), label,
    )
    return cred_id


def get_user_credentials(conn: sqlite3.Connection, user_id: int) -> list[dict]:
    """Get all active credentials for a user."""
    rows = conn.execute(
        """SELECT id, credential_id, public_key, sign_count, aaguid, label,
                  transports, created_at, last_used, is_discoverable
           FROM credentials
           WHERE user_id = ? AND is_revoked = 0""",
        (user_id,),
    ).fetchall()
    creds = []
    for row in rows:
        creds.append({
            "db_id": row[0],
            "credential_id": row[1],
            "public_key": row[2],
            "sign_count": row[3],
            "aaguid": row[4],
            "label": row[5],
            "transports": json.loads(row[6]) if row[6] else [],
            "created_at": row[7],
            "last_used": row[8],
            "is_discoverable": bool(row[9]),
        })
    return creds


def get_all_credentials(conn: sqlite3.Connection) -> list[dict]:
    """Get all active credentials across all users (for discoverable flows)."""
    rows = conn.execute(
        """SELECT c.credential_id, c.public_key, c.sign_count, u.user_handle
           FROM credentials c JOIN users u ON c.user_id = u.id
           WHERE c.is_revoked = 0"""
    ).fetchall()
    return [
        {
            "credential_id": r[0],
            "public_key": r[1],
            "sign_count": r[2],
            "user_handle": r[3],
        }
        for r in rows
    ]


def revoke_credential(conn: sqlite3.Connection, credential_db_id: int, user_id: int) -> bool:
    """Revoke a credential (soft delete)."""
    cursor = conn.execute(
        "UPDATE credentials SET is_revoked = 1 WHERE id = ? AND user_id = ?",
        (credential_db_id, user_id),
    )
    conn.commit()
    return cursor.rowcount > 0


def update_sign_count(conn: sqlite3.Connection, credential_id: bytes, new_count: int):
    """Update the sign count and last-used timestamp after authentication."""
    conn.execute(
        "UPDATE credentials SET sign_count = ?, last_used = datetime('now') WHERE credential_id = ?",
        (new_count, credential_id),
    )
    conn.commit()


# ---------------------------------------------------------------------------
# Recovery codes
# ---------------------------------------------------------------------------

def generate_recovery_codes(conn: sqlite3.Connection, user_id: int, count: int = 8) -> list[str]:
    """Generate one-time recovery codes for account recovery."""
    # Invalidate existing codes
    conn.execute("DELETE FROM recovery_codes WHERE user_id = ?", (user_id,))
    codes = []
    for _ in range(count):
        code = secrets.token_hex(4).upper()  # 8-char hex code
        code_hash = hashlib.sha256(code.encode()).hexdigest()
        conn.execute(
            "INSERT INTO recovery_codes (user_id, code_hash) VALUES (?, ?)",
            (user_id, code_hash),
        )
        codes.append(code)
    conn.commit()
    logger.info("Generated %d recovery codes for user %d", count, user_id)
    return codes


def verify_recovery_code(conn: sqlite3.Connection, user_id: int, code: str) -> bool:
    """Verify and consume a recovery code."""
    code_hash = hashlib.sha256(code.strip().upper().encode()).hexdigest()
    row = conn.execute(
        "SELECT id FROM recovery_codes WHERE user_id = ? AND code_hash = ? AND used = 0",
        (user_id, code_hash),
    ).fetchone()
    if not row:
        return False
    conn.execute("UPDATE recovery_codes SET used = 1 WHERE id = ?", (row[0],))
    conn.commit()
    return True


# ---------------------------------------------------------------------------
# Auth event logging
# ---------------------------------------------------------------------------

def log_auth_event(
    conn: sqlite3.Connection,
    user_id: int,
    event_type: str,
    success: bool,
    credential_id: bytes = None,
    ip_address: str = None,
    user_agent: str = None,
    details: str = None,
):
    """Log an authentication event for auditing."""
    conn.execute(
        """INSERT INTO auth_events
           (user_id, credential_id, event_type, success, ip_address, user_agent, details)
           VALUES (?, ?, ?, ?, ?, ?, ?)""",
        (user_id, credential_id, event_type, 1 if success else 0,
         ip_address, user_agent, details),
    )
    conn.commit()


# ---------------------------------------------------------------------------
# Credential data helpers for python-fido2
# ---------------------------------------------------------------------------

def build_credential_descriptors(creds: list[dict]) -> list:
    """Build PublicKeyCredentialDescriptor list from stored credentials."""
    descriptors = []
    for c in creds:
        desc = PublicKeyCredentialDescriptor(
            type="public-key",
            id=c["credential_id"],
        )
        descriptors.append(desc)
    return descriptors


def reconstruct_credential_data(creds: list[dict]):
    """Reconstruct AttestedCredentialData objects from stored credentials.

    The python-fido2 library's Fido2Server.authenticate_complete expects
    credential data objects that contain credential_id and public_key.
    We rebuild them from our database records.
    """
    from fido2.webauthn import AttestedCredentialData
    result = []
    for c in creds:
        cred_data = AttestedCredentialData.create(
            aaguid=bytes.fromhex(c["aaguid"]) if c.get("aaguid") else b"\x00" * 16,
            credential_id=c["credential_id"],
            public_key=c["public_key"],
        )
        result.append(cred_data)
    return result


# ---------------------------------------------------------------------------
# Flask application factory
# ---------------------------------------------------------------------------

INDEX_HTML = """<!DOCTYPE html>
<html>
<head>
    <title>FIDO2 WebAuthn Demo</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
               max-width: 700px; margin: 40px auto; padding: 0 20px; }
        h1 { color: #333; }
        .section { background: #f8f9fa; border: 1px solid #dee2e6; border-radius: 8px;
                   padding: 20px; margin: 20px 0; }
        input { padding: 8px 12px; margin: 5px; border: 1px solid #ced4da; border-radius: 4px; }
        button { padding: 8px 16px; margin: 5px; border: none; border-radius: 4px;
                 cursor: pointer; font-weight: 500; }
        .btn-primary { background: #0d6efd; color: white; }
        .btn-success { background: #198754; color: white; }
        .btn-danger { background: #dc3545; color: white; }
        #status { margin-top: 20px; padding: 10px; border-radius: 4px; display: none; }
        .success { background: #d1e7dd; color: #0f5132; display: block !important; }
        .error { background: #f8d7da; color: #842029; display: block !important; }
        .info { background: #cff4fc; color: #055160; display: block !important; }
    </style>
</head>
<body>
    <h1>FIDO2 / WebAuthn Authentication</h1>

    <div class="section">
        <h2>Register</h2>
        <input type="text" id="reg-username" placeholder="Username" />
        <input type="text" id="reg-display" placeholder="Display Name" />
        <br/>
        <label><input type="checkbox" id="reg-resident" /> Discoverable credential (passkey)</label>
        <br/>
        <button class="btn-primary" onclick="doRegister()">Register Security Key</button>
    </div>

    <div class="section">
        <h2>Authenticate</h2>
        <input type="text" id="auth-username" placeholder="Username (optional for passkeys)" />
        <button class="btn-success" onclick="doAuthenticate()">Authenticate</button>
    </div>

    <div id="status"></div>

    <script>
        function b64encode(buf) {
            return btoa(String.fromCharCode(...new Uint8Array(buf)))
                .replace(/\\+/g, '-').replace(/\\//g, '_').replace(/=+$/, '');
        }
        function b64decode(str) {
            str = str.replace(/-/g, '+').replace(/_/g, '/');
            while (str.length % 4) str += '=';
            return Uint8Array.from(atob(str), c => c.charCodeAt(0)).buffer;
        }
        function showStatus(msg, type) {
            const el = document.getElementById('status');
            el.textContent = msg;
            el.className = type;
        }

        async function doRegister() {
            const username = document.getElementById('reg-username').value;
            const displayName = document.getElementById('reg-display').value || username;
            const resident = document.getElementById('reg-resident').checked;
            if (!username) { showStatus('Username required', 'error'); return; }

            try {
                showStatus('Starting registration...', 'info');
                const beginResp = await fetch('/api/register/begin', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({username, display_name: displayName, resident_key: resident})
                });
                if (!beginResp.ok) throw new Error(await beginResp.text());
                const options = await beginResp.json();

                // Decode binary fields
                options.publicKey.challenge = b64decode(options.publicKey.challenge);
                options.publicKey.user.id = b64decode(options.publicKey.user.id);
                if (options.publicKey.excludeCredentials) {
                    options.publicKey.excludeCredentials.forEach(c => { c.id = b64decode(c.id); });
                }

                showStatus('Touch your security key...', 'info');
                const credential = await navigator.credentials.create(options);

                const completeResp = await fetch('/api/register/complete', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({
                        id: credential.id,
                        rawId: b64encode(credential.rawId),
                        type: credential.type,
                        response: {
                            attestationObject: b64encode(credential.response.attestationObject),
                            clientDataJSON: b64encode(credential.response.clientDataJSON),
                        }
                    })
                });
                if (!completeResp.ok) throw new Error(await completeResp.text());
                const result = await completeResp.json();
                showStatus('Registration successful! ' + (result.recovery_codes ?
                    'Recovery codes: ' + result.recovery_codes.join(', ') : ''), 'success');
            } catch (err) {
                showStatus('Registration failed: ' + err.message, 'error');
            }
        }

        async function doAuthenticate() {
            const username = document.getElementById('auth-username').value;
            try {
                showStatus('Starting authentication...', 'info');
                const beginResp = await fetch('/api/authenticate/begin', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({username: username || null})
                });
                if (!beginResp.ok) throw new Error(await beginResp.text());
                const options = await beginResp.json();

                options.publicKey.challenge = b64decode(options.publicKey.challenge);
                if (options.publicKey.allowCredentials) {
                    options.publicKey.allowCredentials.forEach(c => { c.id = b64decode(c.id); });
                }

                showStatus('Touch your security key...', 'info');
                const assertion = await navigator.credentials.get(options);

                const completeResp = await fetch('/api/authenticate/complete', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({
                        id: assertion.id,
                        rawId: b64encode(assertion.rawId),
                        type: assertion.type,
                        response: {
                            authenticatorData: b64encode(assertion.response.authenticatorData),
                            clientDataJSON: b64encode(assertion.response.clientDataJSON),
                            signature: b64encode(assertion.response.signature),
                            userHandle: assertion.response.userHandle ?
                                b64encode(assertion.response.userHandle) : null,
                        }
                    })
                });
                if (!completeResp.ok) throw new Error(await completeResp.text());
                const result = await completeResp.json();
                showStatus('Authentication successful! Welcome, ' + result.username, 'success');
            } catch (err) {
                showStatus('Authentication failed: ' + err.message, 'error');
            }
        }
    </script>
</body>
</html>"""


def create_app(
    rp_id: str,
    rp_name: str,
    db_path: str,
    attestation: str = "none",
    user_verification: str = "preferred",
) -> Flask:
    """Create and configure the Flask application with WebAuthn endpoints."""
    app = Flask(__name__)
    app.secret_key = os.urandom(32)
    app.config["SESSION_COOKIE_SECURE"] = rp_id != "localhost"
    app.config["SESSION_COOKIE_HTTPONLY"] = True
    app.config["SESSION_COOKIE_SAMESITE"] = "Strict"

    rp = PublicKeyCredentialRpEntity(name=rp_name, id=rp_id)
    server = Fido2Server(rp)
    conn = init_database(db_path)

    attestation_pref = {
        "none": AttestationConveyancePreference.NONE,
        "indirect": AttestationConveyancePreference.INDIRECT,
        "direct": AttestationConveyancePreference.DIRECT,
        "enterprise": AttestationConveyancePreference.ENTERPRISE,
    }.get(attestation, AttestationConveyancePreference.NONE)

    uv_pref = {
        "required": UserVerificationRequirement.REQUIRED,
        "preferred": UserVerificationRequirement.PREFERRED,
        "discouraged": UserVerificationRequirement.DISCOURAGED,
    }.get(user_verification, UserVerificationRequirement.PREFERRED)

    @app.route("/")
    def index():
        return render_template_string(INDEX_HTML)

    # ------ Registration endpoints ------

    @app.route("/api/register/begin", methods=["POST"])
    def register_begin():
        data = request.get_json()
        if not data or not data.get("username"):
            abort(400, "username required")
        username = data["username"]
        display_name = data.get("display_name", username)
        resident_key = data.get("resident_key", False)

        user = get_user_by_username(conn, username)
        if not user:
            user = create_user(conn, username, display_name)

        # Get existing credentials to exclude
        existing_creds = get_user_credentials(conn, user["id"])
        exclude_list = build_credential_descriptors(existing_creds)

        resident_req = (
            ResidentKeyRequirement.REQUIRED if resident_key
            else ResidentKeyRequirement.DISCOURAGED
        )

        registration_data, state = server.register_begin(
            PublicKeyCredentialUserEntity(
                id=user["user_handle"],
                name=user["username"],
                display_name=user["display_name"],
            ),
            credentials=reconstruct_credential_data(existing_creds) if existing_creds else [],
            user_verification=uv_pref,
            authenticator_attachment=None,
            resident_key_requirement=resident_req,
        )
        session["reg_state"] = state
        session["reg_user_id"] = user["id"]
        session["reg_resident"] = resident_key

        # Serialize for JSON response
        options = dict(registration_data)
        return jsonify(options)

    @app.route("/api/register/complete", methods=["POST"])
    def register_complete():
        data = request.get_json()
        if not data:
            abort(400, "No credential response")

        state = session.pop("reg_state", None)
        user_id = session.pop("reg_user_id", None)
        is_resident = session.pop("reg_resident", False)
        if not state or not user_id:
            abort(400, "No pending registration")

        try:
            auth_data = server.register_complete(state, data)
        except Exception as exc:
            logger.warning("Registration verification failed: %s", exc)
            log_auth_event(
                conn, user_id, "registration", False,
                ip_address=request.remote_addr,
                user_agent=request.headers.get("User-Agent", ""),
                details=str(exc),
            )
            abort(400, f"Registration failed: {exc}")

        cred_data = auth_data.credential_data
        aaguid_hex = cred_data.aaguid.hex() if hasattr(cred_data, "aaguid") else None

        store_credential(
            conn,
            user_id=user_id,
            credential_id=cred_data.credential_id,
            public_key=cred_data.public_key,
            sign_count=auth_data.sign_count if hasattr(auth_data, "sign_count") else 0,
            aaguid=aaguid_hex,
            label=data.get("label", f"Key registered {datetime.now(timezone.utc).strftime('%Y-%m-%d')}"),
            is_discoverable=is_resident,
        )

        log_auth_event(
            conn, user_id, "registration", True,
            credential_id=cred_data.credential_id,
            ip_address=request.remote_addr,
            user_agent=request.headers.get("User-Agent", ""),
        )

        # Generate recovery codes on first credential registration
        user_creds = get_user_credentials(conn, user_id)
        response_data = {"status": "OK"}
        if len(user_creds) == 1:
            codes = generate_recovery_codes(conn, user_id)
            response_data["recovery_codes"] = codes
            response_data["message"] = "Save these recovery codes securely. They will not be shown again."

        return jsonify(response_data)

    # ------ Authentication endpoints ------

    @app.route("/api/authenticate/begin", methods=["POST"])
    def authenticate_begin():
        data = request.get_json() or {}
        username = data.get("username")

        if username:
            user = get_user_by_username(conn, username)
            if not user:
                abort(404, "User not found")
            existing_creds = get_user_credentials(conn, user["id"])
            if not existing_creds:
                abort(404, "No credentials registered")
            cred_data = reconstruct_credential_data(existing_creds)
            session["auth_user_id"] = user["id"]
        else:
            # Discoverable credential flow (passwordless)
            cred_data = []
            session["auth_user_id"] = None

        auth_data, state = server.authenticate_begin(
            credentials=cred_data if cred_data else None,
            user_verification=uv_pref,
        )
        session["auth_state"] = state

        options = dict(auth_data)
        return jsonify(options)

    @app.route("/api/authenticate/complete", methods=["POST"])
    def authenticate_complete():
        data = request.get_json()
        if not data:
            abort(400, "No assertion response")

        state = session.pop("auth_state", None)
        expected_user_id = session.pop("auth_user_id", None)
        if not state:
            abort(400, "No pending authentication")

        # Gather all credentials that could match
        if expected_user_id:
            user_creds = get_user_credentials(conn, expected_user_id)
            cred_data = reconstruct_credential_data(user_creds)
        else:
            # For discoverable credentials, gather all credentials
            all_creds = get_all_credentials(conn)
            user_creds = all_creds
            cred_data = reconstruct_credential_data(all_creds)

        try:
            auth_result = server.authenticate_complete(
                state,
                credentials=cred_data,
                response=data,
            )
        except Exception as exc:
            logger.warning("Authentication verification failed: %s", exc)
            if expected_user_id:
                log_auth_event(
                    conn, expected_user_id, "authentication", False,
                    ip_address=request.remote_addr,
                    user_agent=request.headers.get("User-Agent", ""),
                    details=str(exc),
                )
            abort(401, f"Authentication failed: {exc}")

        # Find the credential that was used
        used_cred_id = auth_result.credential_id
        new_sign_count = auth_result.new_sign_count

        # Detect sign count regression (possible cloned key)
        for c in user_creds:
            if c["credential_id"] == used_cred_id:
                if new_sign_count <= c["sign_count"] and new_sign_count != 0:
                    logger.warning(
                        "SECURITY: Sign count regression for credential %s "
                        "(stored: %d, received: %d) -- possible cloned key!",
                        urlsafe_b64encode(used_cred_id).decode(),
                        c["sign_count"], new_sign_count,
                    )
                break

        update_sign_count(conn, used_cred_id, new_sign_count)

        # Determine user from credential for discoverable flows
        if expected_user_id:
            user_id = expected_user_id
        else:
            # Look up user by credential
            row = conn.execute(
                "SELECT user_id FROM credentials WHERE credential_id = ?",
                (used_cred_id,),
            ).fetchone()
            user_id = row[0] if row else None

        if user_id:
            user = conn.execute(
                "SELECT username, display_name FROM users WHERE id = ?", (user_id,)
            ).fetchone()
            username = user[0] if user else "unknown"
            log_auth_event(
                conn, user_id, "authentication", True,
                credential_id=used_cred_id,
                ip_address=request.remote_addr,
                user_agent=request.headers.get("User-Agent", ""),
            )
        else:
            username = "unknown"

        session["authenticated_user"] = username
        return jsonify({"status": "OK", "username": username})

    # ------ Key management endpoints ------

    @app.route("/api/keys", methods=["GET"])
    def list_keys():
        username = session.get("authenticated_user")
        if not username:
            abort(401, "Not authenticated")
        user = get_user_by_username(conn, username)
        if not user:
            abort(404, "User not found")
        creds = get_user_credentials(conn, user["id"])
        return jsonify([
            {
                "id": c["db_id"],
                "label": c["label"],
                "aaguid": c["aaguid"],
                "created_at": c["created_at"],
                "last_used": c["last_used"],
                "sign_count": c["sign_count"],
                "is_discoverable": c["is_discoverable"],
                "credential_id_b64": urlsafe_b64encode(c["credential_id"]).decode(),
            }
            for c in creds
        ])

    @app.route("/api/keys/<int:key_id>/revoke", methods=["POST"])
    def revoke_key(key_id):
        username = session.get("authenticated_user")
        if not username:
            abort(401, "Not authenticated")
        user = get_user_by_username(conn, username)
        if not user:
            abort(404, "User not found")

        # Ensure at least one credential remains
        creds = get_user_credentials(conn, user["id"])
        if len(creds) <= 1:
            abort(400, "Cannot revoke last credential")

        if revoke_credential(conn, key_id, user["id"]):
            log_auth_event(
                conn, user["id"], "key_revocation", True,
                ip_address=request.remote_addr,
                details=f"Revoked key ID {key_id}",
            )
            return jsonify({"status": "OK", "message": f"Key {key_id} revoked"})
        abort(404, "Key not found")

    @app.route("/api/keys/<int:key_id>/label", methods=["PUT"])
    def update_key_label(key_id):
        username = session.get("authenticated_user")
        if not username:
            abort(401, "Not authenticated")
        user = get_user_by_username(conn, username)
        if not user:
            abort(404)
        data = request.get_json() or {}
        label = data.get("label", "")
        if not label:
            abort(400, "label required")
        conn.execute(
            "UPDATE credentials SET label = ? WHERE id = ? AND user_id = ?",
            (label, key_id, user["id"]),
        )
        conn.commit()
        return jsonify({"status": "OK"})

    # ------ Recovery endpoint ------

    @app.route("/api/recover", methods=["POST"])
    def recover_account():
        data = request.get_json() or {}
        username = data.get("username")
        code = data.get("recovery_code")
        if not username or not code:
            abort(400, "username and recovery_code required")
        user = get_user_by_username(conn, username)
        if not user:
            abort(404, "User not found")
        if verify_recovery_code(conn, user["id"], code):
            session["authenticated_user"] = username
            session["recovery_mode"] = True
            log_auth_event(
                conn, user["id"], "recovery", True,
                ip_address=request.remote_addr,
                details="Authenticated via recovery code",
            )
            return jsonify({
                "status": "OK",
                "message": "Recovery successful. Please register a new security key immediately.",
            })
        log_auth_event(
            conn, user["id"], "recovery", False,
            ip_address=request.remote_addr,
            details="Invalid recovery code",
        )
        abort(401, "Invalid recovery code")

    # ------ Admin/reporting endpoints ------

    @app.route("/api/admin/stats", methods=["GET"])
    def admin_stats():
        total_users = conn.execute("SELECT COUNT(*) FROM users").fetchone()[0]
        total_creds = conn.execute(
            "SELECT COUNT(*) FROM credentials WHERE is_revoked = 0"
        ).fetchone()[0]
        users_with_backup = conn.execute(
            """SELECT COUNT(DISTINCT user_id) FROM credentials
               WHERE is_revoked = 0
               GROUP BY user_id HAVING COUNT(*) >= 2"""
        ).fetchall()
        recent_auths = conn.execute(
            """SELECT COUNT(*) FROM auth_events
               WHERE event_type = 'authentication' AND success = 1
               AND created_at >= datetime('now', '-30 days')"""
        ).fetchone()[0]
        auth_failures = conn.execute(
            """SELECT COUNT(*) FROM auth_events
               WHERE event_type = 'authentication' AND success = 0
               AND created_at >= datetime('now', '-30 days')"""
        ).fetchone()[0]
        sign_count_warnings = conn.execute(
            """SELECT COUNT(*) FROM auth_events
               WHERE details LIKE '%sign count regression%'
               AND created_at >= datetime('now', '-30 days')"""
        ).fetchone()[0]

        # AAGUID distribution (authenticator model breakdown)
        aaguid_dist = conn.execute(
            """SELECT aaguid, COUNT(*) as cnt
               FROM credentials WHERE is_revoked = 0 AND aaguid IS NOT NULL
               GROUP BY aaguid ORDER BY cnt DESC"""
        ).fetchall()

        return jsonify({
            "total_users": total_users,
            "total_active_credentials": total_creds,
            "users_with_backup_key": len(users_with_backup),
            "auth_last_30_days": recent_auths,
            "auth_failures_last_30_days": auth_failures,
            "sign_count_regressions_30d": sign_count_warnings,
            "authenticator_models": [
                {"aaguid": r[0], "count": r[1]} for r in aaguid_dist
            ],
        })

    @app.route("/api/admin/audit-log", methods=["GET"])
    def audit_log():
        limit = request.args.get("limit", 100, type=int)
        rows = conn.execute(
            """SELECT ae.created_at, u.username, ae.event_type, ae.success,
                      ae.ip_address, ae.details
               FROM auth_events ae JOIN users u ON ae.user_id = u.id
               ORDER BY ae.created_at DESC LIMIT ?""",
            (min(limit, 1000),),
        ).fetchall()
        return jsonify([
            {
                "timestamp": r[0], "username": r[1], "event": r[2],
                "success": bool(r[3]), "ip": r[4], "details": r[5],
            }
            for r in rows
        ])

    return app


# ---------------------------------------------------------------------------
# CLI entry point
# ---------------------------------------------------------------------------

def main():
    parser = argparse.ArgumentParser(
        description="FIDO2/WebAuthn Hardware Security Key Authentication Server",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  # Start development server on localhost
  python agent.py --rp-id localhost --rp-name "My App" --port 5000

  # Production mode with strict user verification
  python agent.py --rp-id auth.example.com --rp-name "Example Corp" \\
      --user-verification required --attestation direct --db prod_keys.db

  # Require discoverable credentials (passkeys)
  python agent.py --rp-id example.com --rp-name "Example" --port 8443
        """,
    )
    parser.add_argument("--rp-id", default="localhost", help="Relying Party ID (domain, default: localhost)")
    parser.add_argument("--rp-name", default="FIDO2 Demo", help="Relying Party display name")
    parser.add_argument("--host", default="localhost", help="Server bind address (default: localhost)")
    parser.add_argument("--port", type=int, default=5000, help="Server port (default: 5000)")
    parser.add_argument("--db", default="webauthn.db", help="SQLite database path (default: webauthn.db)")
    parser.add_argument(
        "--attestation", choices=["none", "indirect", "direct", "enterprise"],
        default="none", help="Attestation conveyance preference (default: none)",
    )
    parser.add_argument(
        "--user-verification", choices=["required", "preferred", "discouraged"],
        default="preferred", help="User verification requirement (default: preferred)",
    )
    parser.add_argument("-v", "--verbose", action="store_true", help="Enable debug logging")

    args = parser.parse_args()

    if args.verbose:
        logging.getLogger().setLevel(logging.DEBUG)

    logger.info("Starting FIDO2 WebAuthn server")
    logger.info("  RP ID:    %s", args.rp_id)
    logger.info("  RP Name:  %s", args.rp_name)
    logger.info("  Host:     %s:%d", args.host, args.port)
    logger.info("  Database: %s", args.db)
    logger.info("  Attestation: %s", args.attestation)
    logger.info("  User Verification: %s", args.user_verification)

    app = create_app(
        rp_id=args.rp_id,
        rp_name=args.rp_name,
        db_path=args.db,
        attestation=args.attestation,
        user_verification=args.user_verification,
    )

    if args.rp_id != "localhost":
        logger.warning(
            "Running without TLS. In production, place behind a TLS-terminating "
            "reverse proxy (nginx, Caddy) -- WebAuthn requires HTTPS."
        )

    app.run(host=args.host, port=args.port, debug=args.verbose)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-privileged-access-workstation/SKILL.md =====
---
name: implementing-privileged-access-workstation
description: Design and implement Privileged Access Workstations (PAWs) using the
  tiered administration model, with device hardening, device compliance enforcement
  via Microsoft Intune or Group Policy, just-in-time (JIT) access provisioning, and
  integration with PAM platforms like CyberArk or BeyondTrust. Use when hardening
  admin endpoints, designing a tiered administration model, or securing privileged
  administrative operations.
domain: cybersecurity
subdomain: identity-and-access-management
tags:
- privileged-access
- PAW
- zero-trust
- device-hardening
- CyberArk
- BeyondTrust
- just-in-time-access
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-02
- PR.AA-05
mitre_attack:
- T1078
- T1190
- T1059
---

# Implementing Privileged Access Workstation

## Overview

A Privileged Access Workstation (PAW) is a hardened device dedicated to performing sensitive administrative tasks. This skill covers PAW design using the tiered administration model, device compliance enforcement via Microsoft Intune or Group Policy, just-in-time (JIT) access provisioning, and integration with privileged access management (PAM) platforms like CyberArk and BeyondTrust.


## When to Use

- When deploying or configuring implementing privileged access workstation capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Windows 10/11 Enterprise with Virtualization Based Security (VBS)
- Microsoft Intune or Active Directory Group Policy
- CyberArk Privileged Access Security or BeyondTrust Password Safe (optional)
- Python 3.9+ with `requests`, `subprocess`, `json`
- Administrative access to target endpoints

## Steps

1. Audit current privileged access patterns and identify Tier 0/1/2 assets
2. Configure device hardening baselines (AppLocker, Credential Guard, Device Guard)
3. Enforce compliance policies via Intune or GPO
4. Implement just-in-time access with time-limited admin group membership
5. Integrate with CyberArk/BeyondTrust for credential vaulting
6. Validate PAW configuration against CIS and Microsoft PAW guidance
7. Monitor privileged sessions and generate compliance reports

## Expected Output

- JSON report listing device compliance status, hardening checks, JIT access windows, and PAM integration verification
- Risk scoring per workstation with remediation recommendations

===== END FILE =====


===== FILE: skills/implementing-privileged-access-workstation/LICENSE =====

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


===== FILE: skills/implementing-privileged-access-workstation/references/api-reference.md =====
# API Reference — Implementing Privileged Access Workstation

## Libraries Used
- **subprocess**: Execute PowerShell cmdlets for device hardening, group membership, software inventory
- **json**: Parse PowerShell ConvertTo-Json output

## CLI Interface
```
python agent.py harden
python agent.py admins
python agent.py software
python agent.py network
python agent.py full
```

## Core Functions

### `check_device_hardening()` — Audit 7 PAW hardening controls
Checks: Credential Guard, VBS status, Secure Boot, BitLocker, AppLocker,
Windows Firewall profiles, UAC level via registry.

### `check_local_admin_group()` — JIT access audit
Enumerates local Administrators group via `Get-LocalGroupMember`.
Flags unexpected members not matching known admin accounts.

### `check_installed_software()` — Software allowlist enforcement
Queries installed software from registry. Checks against blocked list:
browsers (Chrome, Firefox), personal apps (Spotify, Steam, Slack, Zoom, Dropbox).

### `check_network_restrictions()` — Network isolation verification
Counts outbound firewall block rules. Tests general internet connectivity.
PAW Tier 0 should block internet — only management endpoints allowed.

### `full_paw_audit()` — Comprehensive compliance report

## PAW Hardening Checks
| Check | PowerShell Source | Pass Criteria |
|-------|------------------|---------------|
| Credential Guard | Win32_DeviceGuard | SecurityServicesRunning > 0 |
| VBS | Win32_DeviceGuard | VirtualizationBasedSecurityStatus = 2 |
| Secure Boot | Confirm-SecureBootUEFI | Returns True |
| BitLocker | Get-BitLockerVolume | ProtectionStatus = On |
| AppLocker | Get-AppLockerPolicy | RuleCollection count > 0 |
| Firewall | Get-NetFirewallProfile | All profiles enabled |
| UAC | Registry query | ConsentPromptBehaviorAdmin >= 2 |

## Blocked Software Patterns
chrome, firefox, spotify, steam, vlc, zoom, slack, dropbox, itunes, whatsapp, telegram

## Dependencies
No external packages — Python standard library only.
Requires: Windows 10/11 Enterprise with PowerShell 5.1+

===== END FILE =====


===== FILE: skills/implementing-privileged-access-workstation/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing and auditing Privileged Access Workstation (PAW) configurations."""

import json
import argparse
import subprocess
from datetime import datetime


def check_device_hardening():
    """Audit local device hardening controls for PAW compliance."""
    checks = {}
    hardening_cmds = {
        "credential_guard": ["powershell", "-Command",
            "(Get-CimInstance -ClassName Win32_DeviceGuard -Namespace root\\Microsoft\\Windows\\DeviceGuard).SecurityServicesRunning"],
        "vbs_status": ["powershell", "-Command",
            "(Get-CimInstance -ClassName Win32_DeviceGuard -Namespace root\\Microsoft\\Windows\\DeviceGuard).VirtualizationBasedSecurityStatus"],
        "secure_boot": ["powershell", "-Command", "Confirm-SecureBootUEFI"],
        "bitlocker": ["powershell", "-Command",
            "(Get-BitLockerVolume -MountPoint C:).ProtectionStatus"],
        "applocker_status": ["powershell", "-Command",
            "(Get-AppLockerPolicy -Effective -Xml | Select-String 'RuleCollection').Count"],
        "firewall_enabled": ["powershell", "-Command",
            "(Get-NetFirewallProfile | Where-Object {$_.Enabled -eq $true}).Name -join ','"],
        "uac_level": ["reg", "query",
            "HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Policies\\System",
            "/v", "ConsentPromptBehaviorAdmin"],
    }
    for name, cmd in hardening_cmds.items():
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=15)
            output = result.stdout.strip()
            checks[name] = {"value": output, "status": "PASS" if output and output not in ("0", "False", "") else "FAIL"}
        except Exception as e:
            checks[name] = {"value": str(e), "status": "ERROR"}
    passed = sum(1 for c in checks.values() if c["status"] == "PASS")
    return {
        "timestamp": datetime.utcnow().isoformat(),
        "checks": checks,
        "passed": passed,
        "total": len(checks),
        "compliance_pct": round(passed / len(checks) * 100, 1),
        "risk": "LOW" if passed >= 6 else "MEDIUM" if passed >= 4 else "HIGH",
    }


def check_local_admin_group():
    """Enumerate local Administrators group membership for JIT audit."""
    try:
        result = subprocess.run(
            ["powershell", "-Command",
             "Get-LocalGroupMember -Group Administrators | Select-Object Name,ObjectClass,PrincipalSource | ConvertTo-Json"],
            capture_output=True, text=True, timeout=15)
        members = json.loads(result.stdout) if result.stdout.strip() else []
        if isinstance(members, dict):
            members = [members]
        expected_admins = ["Administrator", "Domain Admins"]
        unexpected = [m for m in members if not any(e in m.get("Name", "") for e in expected_admins)]
        return {
            "total_members": len(members),
            "members": members,
            "unexpected_admins": unexpected,
            "jit_finding": "UNEXPECTED_ADMINS" if unexpected else "CLEAN",
            "severity": "HIGH" if unexpected else "INFO",
        }
    except Exception as e:
        return {"error": str(e)}


def check_installed_software():
    """Audit installed software against PAW allowlist."""
    try:
        result = subprocess.run(
            ["powershell", "-Command",
             "Get-ItemProperty HKLM:\\Software\\Microsoft\\Windows\\CurrentVersion\\Uninstall\\* | "
             "Select-Object DisplayName,Publisher,InstallDate | Where-Object {$_.DisplayName -ne $null} | "
             "ConvertTo-Json -Depth 2"],
            capture_output=True, text=True, timeout=30)
        software = json.loads(result.stdout) if result.stdout.strip() else []
        if isinstance(software, dict):
            software = [software]
        blocked_patterns = [
            "chrome", "firefox", "spotify", "steam", "vlc", "zoom", "slack",
            "dropbox", "onedrive personal", "itunes", "whatsapp", "telegram",
        ]
        violations = []
        for sw in software:
            name = (sw.get("DisplayName") or "").lower()
            if any(p in name for p in blocked_patterns):
                violations.append({"name": sw.get("DisplayName"), "publisher": sw.get("Publisher")})
        return {
            "total_installed": len(software),
            "paw_violations": len(violations),
            "violations": violations[:20],
            "finding": "BLOCKED_SOFTWARE_FOUND" if violations else "COMPLIANT",
        }
    except Exception as e:
        return {"error": str(e)}


def check_network_restrictions():
    """Verify PAW network isolation and restrictions."""
    checks = {}
    try:
        result = subprocess.run(
            ["powershell", "-Command",
             "Get-NetFirewallRule | Where-Object {$_.Enabled -eq 'True' -and $_.Direction -eq 'Outbound' -and $_.Action -eq 'Block'} | "
             "Measure-Object | Select-Object -ExpandProperty Count"],
            capture_output=True, text=True, timeout=15)
        checks["outbound_block_rules"] = int(result.stdout.strip() or 0)
    except Exception:
        checks["outbound_block_rules"] = 0
    try:
        result = subprocess.run(["powershell", "-Command",
            "Test-NetConnection -ComputerName google.com -Port 80 -WarningAction SilentlyContinue | Select-Object TcpTestSucceeded | ConvertTo-Json"],
            capture_output=True, text=True, timeout=15)
        data = json.loads(result.stdout) if result.stdout.strip() else {}
        checks["internet_access"] = data.get("TcpTestSucceeded", True)
    except Exception:
        checks["internet_access"] = "unknown"
    paw_should_block_internet = checks.get("internet_access") is False
    return {
        "network_checks": checks,
        "internet_blocked": paw_should_block_internet,
        "finding": "INTERNET_BLOCKED" if paw_should_block_internet else "INTERNET_ALLOWED",
        "severity": "INFO" if paw_should_block_internet else "MEDIUM",
        "recommendation": "PAW Tier 0 should block general internet — restrict to management endpoints only",
    }


def full_paw_audit():
    """Run comprehensive PAW compliance audit."""
    return {
        "audit_type": "Privileged Access Workstation",
        "timestamp": datetime.utcnow().isoformat(),
        "device_hardening": check_device_hardening(),
        "admin_group": check_local_admin_group(),
        "software_compliance": check_installed_software(),
        "network_isolation": check_network_restrictions(),
    }


def main():
    parser = argparse.ArgumentParser(description="Privileged Access Workstation Audit Agent")
    sub = parser.add_subparsers(dest="command")
    sub.add_parser("harden", help="Check device hardening controls")
    sub.add_parser("admins", help="Audit local admin group membership")
    sub.add_parser("software", help="Check installed software against allowlist")
    sub.add_parser("network", help="Verify network isolation")
    sub.add_parser("full", help="Full PAW compliance audit")
    args = parser.parse_args()
    if args.command == "harden":
        result = check_device_hardening()
    elif args.command == "admins":
        result = check_local_admin_group()
    elif args.command == "software":
        result = check_installed_software()
    elif args.command == "network":
        result = check_network_restrictions()
    elif args.command == "full":
        result = full_paw_audit()
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====
