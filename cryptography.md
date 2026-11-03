

===== FILE: skills/configuring-certificate-authority-with-openssl/SKILL.md =====
---
name: configuring-certificate-authority-with-openssl
description: Build a two-tier PKI Certificate Authority hierarchy (offline Root CA
  plus issuing Intermediate CA) using OpenSSL and the Python cryptography library,
  covering certificate extensions, CRL distribution points, OCSP responder
  configuration, and certificate policy management. Use when standing up an internal
  CA, issuing or revoking X.509 certificates, or designing PKI trust hierarchies for
  TLS, code-signing, or client-authentication use cases.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- pki
- certificate-authority
- openssl
- x509
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1649
- T1553.004
- T1557
- T1587.003
---
# Configuring Certificate Authority with OpenSSL

## Overview

A Certificate Authority (CA) is the trust anchor in a PKI hierarchy, responsible for issuing, signing, and revoking digital certificates. This skill covers building a two-tier CA hierarchy (Root CA + Intermediate CA) using OpenSSL and the Python cryptography library, including CRL distribution, OCSP responder configuration, and certificate policy management.


## When to Use

- When deploying or configuring configuring certificate authority with openssl capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Create a Root CA with self-signed certificate
- Create an Intermediate CA signed by the Root CA
- Issue server and client certificates from the Intermediate CA
- Configure Certificate Revocation Lists (CRLs)
- Implement certificate policies and constraints
- Build a complete PKI hierarchy programmatically

## Key Concepts

### CA Hierarchy

```
Root CA (offline, air-gapped)
  |
  +-- Intermediate CA (online, operational)
        |
        +-- Server Certificates
        +-- Client Certificates
        +-- Code Signing Certificates
```

### Certificate Extensions

| Extension | Purpose | Critical |
|-----------|---------|----------|
| basicConstraints | CA:TRUE/FALSE, pathLenConstraint | Yes |
| keyUsage | keyCertSign, cRLSign, digitalSignature | Yes |
| extendedKeyUsage | serverAuth, clientAuth, codeSigning | No |
| subjectKeyIdentifier | Hash of public key | No |
| authorityKeyIdentifier | Issuer's key identifier | No |
| crlDistributionPoints | URL to CRL | No |
| authorityInfoAccess | OCSP responder URL | No |

## Security Considerations

- Root CA private key must be stored offline (air-gapped HSM)
- Use minimum 4096-bit RSA or P-384 ECDSA for CA keys
- Set path length constraints on intermediate CAs
- Implement certificate policies (OIDs)
- Enable CRL and OCSP for revocation checking
- Audit all certificate issuance operations

## Validation Criteria

- [ ] Root CA self-signed certificate is valid
- [ ] Intermediate CA certificate chains to Root CA
- [ ] Issued certificates chain to Intermediate -> Root
- [ ] Path length constraints are enforced
- [ ] CRL is generated and accessible
- [ ] Revoked certificates appear in CRL
- [ ] Certificate policies are correctly embedded

===== END FILE =====


===== FILE: skills/configuring-certificate-authority-with-openssl/LICENSE =====

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


===== FILE: skills/configuring-certificate-authority-with-openssl/assets/template.md =====
# Certificate Authority Configuration Template

## CA Directory Structure

```
pki/
  root-ca/
    private/root-ca.key
    certs/root-ca.crt
    serial.json
    index.json
  intermediate-ca/
    private/intermediate-ca.key
    certs/intermediate-ca.crt
    certs/ca-chain.crt
    certs/issued/
    crl/intermediate.crl
    serial.json
    index.json
```

## OpenSSL Configuration Template (openssl.cnf)

```ini
[ca]
default_ca = CA_default

[CA_default]
dir               = ./ca
certs             = $dir/certs
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
private_key       = $dir/private/ca.key
certificate       = $dir/certs/ca.crt
default_md        = sha256
default_days      = 365
policy            = policy_strict

[policy_strict]
countryName       = match
organizationName  = match
commonName        = supplied

[v3_ca]
basicConstraints = critical, CA:true
keyUsage = critical, keyCertSign, cRLSign
subjectKeyIdentifier = hash

[v3_intermediate_ca]
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, keyCertSign, cRLSign
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always

[server_cert]
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always
```

## Certificate Issuance Checklist

- [ ] Verify CSR subject and SAN entries
- [ ] Validate key strength (minimum 2048-bit RSA or P-256 ECDSA)
- [ ] Check domain ownership or authorization
- [ ] Set appropriate validity period
- [ ] Include correct extensions (EKU, constraints)
- [ ] Sign with intermediate CA (never root)
- [ ] Record in certificate database
- [ ] Provide full chain to requester

===== END FILE =====


===== FILE: skills/configuring-certificate-authority-with-openssl/references/api-reference.md =====
# Certificate Authority with OpenSSL — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| cryptography | `pip install cryptography` | X.509 certificate generation, parsing, and validation |
| pyOpenSSL | `pip install pyOpenSSL` | OpenSSL wrapper for certificate operations |

## Key cryptography Methods

| Method | Description |
|--------|-------------|
| `x509.CertificateBuilder()` | Build X.509 certificates |
| `rsa.generate_private_key(65537, key_size)` | Generate RSA private key |
| `x509.load_pem_x509_certificate(data)` | Parse PEM certificate |
| `cert.subject.rfc4514_string()` | Get subject as RFC 4514 string |
| `x509.random_serial_number()` | Generate unique serial number |

## OpenSSL CLI Commands

| Command | Purpose |
|---------|---------|
| `openssl req -x509 -newkey rsa:4096 -sha256 -days 3650` | Create self-signed CA |
| `openssl req -new -key server.key -out server.csr` | Generate CSR |
| `openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key` | Sign certificate |
| `openssl verify -CAfile ca.crt server.crt` | Verify certificate chain |
| `openssl x509 -in cert.pem -text -noout` | Display certificate details |

## Certificate Best Practices

| Parameter | Recommended Value |
|-----------|-------------------|
| Root CA Key Size | RSA 4096 or EC P-384 |
| Server Key Size | RSA 2048+ or EC P-256 |
| Signature Algorithm | SHA-256 or SHA-384 |
| Root CA Validity | 10-20 years |
| Server Cert Validity | 1 year (398 days max for public) |

## External References

- [cryptography.io X.509 Docs](https://cryptography.io/en/latest/x509/)
- [OpenSSL Cookbook](https://www.feistyduck.com/library/openssl-cookbook/)
- [RFC 5280 X.509 PKI](https://datatracker.ietf.org/doc/html/rfc5280)

===== END FILE =====


===== FILE: skills/configuring-certificate-authority-with-openssl/references/standards.md =====
# Standards and References - Certificate Authority with OpenSSL

## Primary Standards

### RFC 5280 - Internet X.509 PKI Certificate and CRL Profile
- **URL**: https://www.rfc-editor.org/rfc/rfc5280
- **Description**: Core X.509v3 certificate and CRL format specification

### RFC 6960 - X.509 OCSP
- **URL**: https://www.rfc-editor.org/rfc/rfc6960
- **Description**: Online Certificate Status Protocol

### RFC 3647 - Internet X.509 PKI Certificate Policy and Certification Practices Framework
- **URL**: https://www.rfc-editor.org/rfc/rfc3647
- **Description**: Framework for CP and CPS documents

### NIST SP 800-57 Part 1 Rev. 5
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final
- **Description**: Key management recommendations

### CA/Browser Forum Baseline Requirements
- **URL**: https://cabforum.org/baseline-requirements/
- **Description**: Requirements for publicly trusted CAs

## Tools

### OpenSSL
- **URL**: https://www.openssl.org/
- **Docs**: https://www.openssl.org/docs/man3.0/man1/

### Python cryptography library
- **URL**: https://cryptography.io/en/latest/x509/
- **Description**: X.509 certificate generation and parsing

===== END FILE =====


===== FILE: skills/configuring-certificate-authority-with-openssl/references/workflows.md =====
# Workflows - Certificate Authority with OpenSSL

## Workflow 1: Build Two-Tier CA Hierarchy

```
[Generate Root CA Key] (RSA 4096 / ECDSA P-384)
      |
[Create Root CA Self-Signed Certificate]
(validity: 20 years, basicConstraints: CA:TRUE)
      |
[Store Root CA Key Offline]
      |
[Generate Intermediate CA Key]
      |
[Create Intermediate CA CSR]
      |
[Sign Intermediate CSR with Root CA]
(pathLenConstraint: 0, keyUsage: keyCertSign, cRLSign)
      |
[Create CA Chain Bundle]
(intermediate.crt + root.crt)
```

## Workflow 2: Issue End-Entity Certificate

```
[Applicant Generates Key + CSR]
      |
[Submit CSR to Intermediate CA]
      |
[Validate CSR]
(check subject, SAN, key strength)
      |
[Sign with Intermediate CA Key]
(basicConstraints: CA:FALSE)
(extendedKeyUsage: serverAuth / clientAuth)
      |
[Issue Certificate]
      |
[Record in Certificate Database]
```

## Workflow 3: Certificate Revocation

```
[Revocation Request]
      |
[Verify Authorization]
      |
[Revoke Certificate]
(record serial number + reason + date)
      |
[Generate Updated CRL]
(sign with CA key, set nextUpdate)
      |
[Publish CRL to Distribution Point]
      |
[Update OCSP Responder Database]
```

## Workflow 4: OpenSSL CA Commands

```bash
# 1. Create CA directory structure
mkdir -p ca/{certs,crl,newcerts,private}
touch ca/index.txt
echo 1000 > ca/serial
echo 1000 > ca/crlnumber

# 2. Generate Root CA
openssl genrsa -aes256 -out ca/private/ca.key 4096
openssl req -config ca/openssl.cnf -key ca/private/ca.key \
    -new -x509 -days 7300 -sha256 -extensions v3_ca -out ca/certs/ca.crt

# 3. Generate Intermediate CA
openssl genrsa -aes256 -out intermediate/private/intermediate.key 4096
openssl req -config intermediate/openssl.cnf \
    -key intermediate/private/intermediate.key -new -sha256 -out intermediate/csr/intermediate.csr
openssl ca -config ca/openssl.cnf -extensions v3_intermediate_ca \
    -days 3650 -notext -md sha256 -in intermediate/csr/intermediate.csr \
    -out intermediate/certs/intermediate.crt

# 4. Issue server certificate
openssl req -config intermediate/openssl.cnf \
    -key server.key -new -sha256 -out server.csr
openssl ca -config intermediate/openssl.cnf -extensions server_cert \
    -days 365 -notext -md sha256 -in server.csr -out server.crt
```

===== END FILE =====


===== FILE: skills/configuring-certificate-authority-with-openssl/scripts/agent.py =====
#!/usr/bin/env python3
"""Certificate Authority management agent using OpenSSL and cryptography library."""

import json
import sys
import argparse
from datetime import datetime

try:
    from cryptography import x509
    from cryptography.x509.oid import NameOID
    from cryptography.hazmat.primitives import hashes
    from cryptography.hazmat.primitives.asymmetric import rsa
    from datetime import timedelta
except ImportError:
    print("Install: pip install cryptography")
    sys.exit(1)


def generate_ca_certificate(cn="Internal Root CA", org="Organization", days=3650):
    """Generate a self-signed root CA certificate."""
    key = rsa.generate_private_key(public_exponent=65537, key_size=4096)
    subject = issuer = x509.Name([
        x509.NameAttribute(NameOID.COMMON_NAME, cn),
        x509.NameAttribute(NameOID.ORGANIZATION_NAME, org),
    ])
    cert = (x509.CertificateBuilder()
            .subject_name(subject)
            .issuer_name(issuer)
            .public_key(key.public_key())
            .serial_number(x509.random_serial_number())
            .not_valid_before(datetime.utcnow())
            .not_valid_after(datetime.utcnow() + timedelta(days=days))
            .add_extension(x509.BasicConstraints(ca=True, path_length=None), critical=True)
            .add_extension(x509.KeyUsage(digital_signature=True, key_cert_sign=True,
                                          crl_sign=True, content_commitment=False,
                                          key_encipherment=False, data_encipherment=False,
                                          key_agreement=False, encipher_only=False,
                                          decipher_only=False), critical=True)
            .sign(key, hashes.SHA256()))
    return cert, key


def generate_server_cert(ca_cert, ca_key, cn, san_names, days=365):
    """Generate a server certificate signed by the CA."""
    key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
    subject = x509.Name([x509.NameAttribute(NameOID.COMMON_NAME, cn)])
    san = x509.SubjectAlternativeName([x509.DNSName(n) for n in san_names])
    cert = (x509.CertificateBuilder()
            .subject_name(subject)
            .issuer_name(ca_cert.subject)
            .public_key(key.public_key())
            .serial_number(x509.random_serial_number())
            .not_valid_before(datetime.utcnow())
            .not_valid_after(datetime.utcnow() + timedelta(days=days))
            .add_extension(san, critical=False)
            .add_extension(x509.BasicConstraints(ca=False, path_length=None), critical=True)
            .sign(ca_key, hashes.SHA256()))
    return cert, key


def audit_certificate(cert_path):
    """Audit an existing certificate for security issues."""
    with open(cert_path, "rb") as f:
        cert_data = f.read()
    cert = x509.load_pem_x509_certificate(cert_data)
    issues = []
    if cert.public_key().key_size < 2048:
        issues.append({"issue": "Weak key size", "severity": "HIGH",
                        "detail": f"Key size {cert.public_key().key_size} < 2048"})
    if cert.not_valid_after_utc.replace(tzinfo=None) < datetime.utcnow():
        issues.append({"issue": "Certificate expired", "severity": "CRITICAL",
                        "detail": f"Expired on {cert.not_valid_after_utc}"})
    days_remaining = (cert.not_valid_after_utc.replace(tzinfo=None) - datetime.utcnow()).days
    if 0 < days_remaining < 30:
        issues.append({"issue": "Certificate expiring soon", "severity": "HIGH",
                        "detail": f"{days_remaining} days remaining"})
    sig_algo = cert.signature_hash_algorithm
    if sig_algo and sig_algo.name in ("sha1", "md5"):
        issues.append({"issue": f"Weak signature algorithm: {sig_algo.name}", "severity": "HIGH"})
    return {
        "subject": cert.subject.rfc4514_string(),
        "issuer": cert.issuer.rfc4514_string(),
        "not_before": str(cert.not_valid_before_utc),
        "not_after": str(cert.not_valid_after_utc),
        "key_size": cert.public_key().key_size,
        "serial": cert.serial_number,
        "issues": issues,
    }


def run_audit(cert_path=None, generate=False, cn=None, org=None):
    """Execute CA operations."""
    print(f"\n{'='*60}")
    print(f"  CERTIFICATE AUTHORITY AGENT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    if cert_path:
        info = audit_certificate(cert_path)
        print(f"--- CERTIFICATE AUDIT ---")
        print(f"  Subject: {info['subject']}")
        print(f"  Issuer: {info['issuer']}")
        print(f"  Valid: {info['not_before']} to {info['not_after']}")
        print(f"  Key Size: {info['key_size']}")
        print(f"  Issues: {len(info['issues'])}")
        for i in info["issues"]:
            print(f"    [{i['severity']}] {i['issue']}: {i.get('detail', '')}")
        return info

    if generate:
        cert, key = generate_ca_certificate(cn=cn or "Internal Root CA", org=org or "Org")
        print(f"--- CA CERTIFICATE GENERATED ---")
        print(f"  Subject: {cert.subject.rfc4514_string()}")
        print(f"  Serial: {cert.serial_number}")
        return {"status": "generated", "subject": cert.subject.rfc4514_string()}

    return {}


def main():
    parser = argparse.ArgumentParser(description="Certificate Authority Agent")
    parser.add_argument("--audit", help="Path to PEM certificate to audit")
    parser.add_argument("--generate", action="store_true", help="Generate new CA")
    parser.add_argument("--cn", help="Common name for CA cert")
    parser.add_argument("--org", help="Organization name")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_audit(args.audit, args.generate, args.cn, args.org)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-certificate-authority-with-openssl/scripts/process.py =====
#!/usr/bin/env python3
"""
Certificate Authority Builder using Python cryptography library.

Builds a complete two-tier CA hierarchy (Root CA + Intermediate CA)
and issues server/client certificates programmatically.

Requirements:
    pip install cryptography

Usage:
    python process.py build-ca --output ./pki --org "My Organization"
    python process.py issue-cert --ca-dir ./pki --domain server.example.com --type server
    python process.py revoke --ca-dir ./pki --serial 1001
    python process.py generate-crl --ca-dir ./pki
"""

import os
import sys
import json
import argparse
import logging
import datetime
from pathlib import Path
from typing import Dict, Optional, List

from cryptography import x509
from cryptography.x509.oid import NameOID, ExtensionOID
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import rsa, ec
from cryptography.hazmat.backends import default_backend

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def generate_key(key_type: str = "rsa", key_size: int = 4096):
    """Generate a private key."""
    if key_type == "ecdsa":
        return ec.generate_private_key(ec.SECP384R1(), default_backend())
    return rsa.generate_private_key(
        public_exponent=65537, key_size=key_size, backend=default_backend()
    )


def save_key(key, path: Path, passphrase: Optional[str] = None):
    """Save a private key to PEM file."""
    if passphrase:
        enc = serialization.BestAvailableEncryption(passphrase.encode())
    else:
        enc = serialization.NoEncryption()
    path.write_bytes(key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=enc,
    ))


def save_cert(cert, path: Path):
    """Save a certificate to PEM file."""
    path.write_bytes(cert.public_bytes(serialization.Encoding.PEM))


def build_root_ca(
    output_dir: Path,
    organization: str,
    country: str = "US",
    validity_years: int = 20,
) -> Dict:
    """Build a Root CA with self-signed certificate."""
    ca_dir = output_dir / "root-ca"
    ca_dir.mkdir(parents=True, exist_ok=True)
    (ca_dir / "certs").mkdir(exist_ok=True)
    (ca_dir / "private").mkdir(exist_ok=True)

    key = generate_key("rsa", 4096)
    save_key(key, ca_dir / "private" / "root-ca.key")

    subject = x509.Name([
        x509.NameAttribute(NameOID.COUNTRY_NAME, country),
        x509.NameAttribute(NameOID.ORGANIZATION_NAME, organization),
        x509.NameAttribute(NameOID.COMMON_NAME, f"{organization} Root CA"),
    ])

    now = datetime.datetime.utcnow()
    cert = (
        x509.CertificateBuilder()
        .subject_name(subject)
        .issuer_name(subject)
        .public_key(key.public_key())
        .serial_number(x509.random_serial_number())
        .not_valid_before(now)
        .not_valid_after(now + datetime.timedelta(days=validity_years * 365))
        .add_extension(
            x509.BasicConstraints(ca=True, path_length=1), critical=True
        )
        .add_extension(
            x509.KeyUsage(
                digital_signature=True, key_cert_sign=True, crl_sign=True,
                content_commitment=False, key_encipherment=False,
                data_encipherment=False, key_agreement=False,
                encipher_only=False, decipher_only=False,
            ),
            critical=True,
        )
        .add_extension(
            x509.SubjectKeyIdentifier.from_public_key(key.public_key()),
            critical=False,
        )
        .sign(key, hashes.SHA384(), default_backend())
    )

    save_cert(cert, ca_dir / "certs" / "root-ca.crt")

    # Initialize serial number tracker
    (ca_dir / "serial.json").write_text(json.dumps({"next_serial": 1001}))
    (ca_dir / "index.json").write_text(json.dumps({"certificates": []}))

    logger.info(f"Root CA created: {ca_dir}")
    return {
        "type": "root-ca",
        "subject": subject.rfc4514_string(),
        "key_path": str(ca_dir / "private" / "root-ca.key"),
        "cert_path": str(ca_dir / "certs" / "root-ca.crt"),
        "serial_number": hex(cert.serial_number),
        "valid_until": cert.not_valid_after_utc.isoformat(),
    }


def build_intermediate_ca(
    output_dir: Path,
    organization: str,
    country: str = "US",
    validity_years: int = 10,
) -> Dict:
    """Build an Intermediate CA signed by the Root CA."""
    root_dir = output_dir / "root-ca"
    int_dir = output_dir / "intermediate-ca"
    int_dir.mkdir(parents=True, exist_ok=True)
    (int_dir / "certs").mkdir(exist_ok=True)
    (int_dir / "private").mkdir(exist_ok=True)

    root_key_data = (root_dir / "private" / "root-ca.key").read_bytes()
    root_key = serialization.load_pem_private_key(root_key_data, password=None, backend=default_backend())

    root_cert_data = (root_dir / "certs" / "root-ca.crt").read_bytes()
    root_cert = x509.load_pem_x509_certificate(root_cert_data, default_backend())

    int_key = generate_key("rsa", 4096)
    save_key(int_key, int_dir / "private" / "intermediate-ca.key")

    subject = x509.Name([
        x509.NameAttribute(NameOID.COUNTRY_NAME, country),
        x509.NameAttribute(NameOID.ORGANIZATION_NAME, organization),
        x509.NameAttribute(NameOID.COMMON_NAME, f"{organization} Intermediate CA"),
    ])

    now = datetime.datetime.utcnow()
    cert = (
        x509.CertificateBuilder()
        .subject_name(subject)
        .issuer_name(root_cert.subject)
        .public_key(int_key.public_key())
        .serial_number(x509.random_serial_number())
        .not_valid_before(now)
        .not_valid_after(now + datetime.timedelta(days=validity_years * 365))
        .add_extension(
            x509.BasicConstraints(ca=True, path_length=0), critical=True
        )
        .add_extension(
            x509.KeyUsage(
                digital_signature=True, key_cert_sign=True, crl_sign=True,
                content_commitment=False, key_encipherment=False,
                data_encipherment=False, key_agreement=False,
                encipher_only=False, decipher_only=False,
            ),
            critical=True,
        )
        .add_extension(
            x509.SubjectKeyIdentifier.from_public_key(int_key.public_key()),
            critical=False,
        )
        .add_extension(
            x509.AuthorityKeyIdentifier.from_issuer_public_key(root_key.public_key()),
            critical=False,
        )
        .sign(root_key, hashes.SHA384(), default_backend())
    )

    save_cert(cert, int_dir / "certs" / "intermediate-ca.crt")

    # Create chain file
    chain = cert.public_bytes(serialization.Encoding.PEM) + root_cert.public_bytes(serialization.Encoding.PEM)
    (int_dir / "certs" / "ca-chain.crt").write_bytes(chain)

    # Initialize serial and index
    (int_dir / "serial.json").write_text(json.dumps({"next_serial": 2001}))
    (int_dir / "index.json").write_text(json.dumps({"certificates": [], "revoked": []}))

    logger.info(f"Intermediate CA created: {int_dir}")
    return {
        "type": "intermediate-ca",
        "subject": subject.rfc4514_string(),
        "key_path": str(int_dir / "private" / "intermediate-ca.key"),
        "cert_path": str(int_dir / "certs" / "intermediate-ca.crt"),
        "chain_path": str(int_dir / "certs" / "ca-chain.crt"),
        "serial_number": hex(cert.serial_number),
        "valid_until": cert.not_valid_after_utc.isoformat(),
    }


def issue_certificate(
    ca_dir: Path,
    domain: str,
    cert_type: str = "server",
    validity_days: int = 365,
    san_domains: Optional[List[str]] = None,
) -> Dict:
    """Issue a certificate from the Intermediate CA."""
    int_dir = ca_dir / "intermediate-ca"

    int_key_data = (int_dir / "private" / "intermediate-ca.key").read_bytes()
    int_key = serialization.load_pem_private_key(int_key_data, password=None, backend=default_backend())

    int_cert_data = (int_dir / "certs" / "intermediate-ca.crt").read_bytes()
    int_cert = x509.load_pem_x509_certificate(int_cert_data, default_backend())

    # Read and update serial
    serial_data = json.loads((int_dir / "serial.json").read_text())
    serial_num = serial_data["next_serial"]
    serial_data["next_serial"] = serial_num + 1
    (int_dir / "serial.json").write_text(json.dumps(serial_data))

    # Generate end-entity key
    ee_key = generate_key("ecdsa")
    subject = x509.Name([
        x509.NameAttribute(NameOID.COMMON_NAME, domain),
    ])

    san_list = [x509.DNSName(domain)]
    if san_domains:
        for d in san_domains:
            san_list.append(x509.DNSName(d))

    if cert_type == "server":
        eku = x509.ExtendedKeyUsage([x509.oid.ExtendedKeyUsageOID.SERVER_AUTH])
    elif cert_type == "client":
        eku = x509.ExtendedKeyUsage([x509.oid.ExtendedKeyUsageOID.CLIENT_AUTH])
    else:
        eku = x509.ExtendedKeyUsage([
            x509.oid.ExtendedKeyUsageOID.SERVER_AUTH,
            x509.oid.ExtendedKeyUsageOID.CLIENT_AUTH,
        ])

    now = datetime.datetime.utcnow()
    cert = (
        x509.CertificateBuilder()
        .subject_name(subject)
        .issuer_name(int_cert.subject)
        .public_key(ee_key.public_key())
        .serial_number(serial_num)
        .not_valid_before(now)
        .not_valid_after(now + datetime.timedelta(days=validity_days))
        .add_extension(x509.BasicConstraints(ca=False, path_length=None), critical=True)
        .add_extension(
            x509.KeyUsage(
                digital_signature=True, key_encipherment=True,
                key_cert_sign=False, crl_sign=False,
                content_commitment=False, data_encipherment=False,
                key_agreement=False, encipher_only=False, decipher_only=False,
            ),
            critical=True,
        )
        .add_extension(eku, critical=False)
        .add_extension(x509.SubjectAlternativeName(san_list), critical=False)
        .add_extension(
            x509.AuthorityKeyIdentifier.from_issuer_public_key(int_key.public_key()),
            critical=False,
        )
        .sign(int_key, hashes.SHA256(), default_backend())
    )

    certs_dir = int_dir / "certs" / "issued"
    certs_dir.mkdir(exist_ok=True)

    save_key(ee_key, certs_dir / f"{domain}.key")
    save_cert(cert, certs_dir / f"{domain}.crt")

    # Update index
    index_data = json.loads((int_dir / "index.json").read_text())
    index_data["certificates"].append({
        "serial": serial_num,
        "domain": domain,
        "type": cert_type,
        "issued": now.isoformat(),
        "expires": (now + datetime.timedelta(days=validity_days)).isoformat(),
        "status": "valid",
    })
    (int_dir / "index.json").write_text(json.dumps(index_data, indent=2))

    logger.info(f"Issued {cert_type} certificate for {domain} (serial: {serial_num})")
    return {
        "domain": domain,
        "serial": serial_num,
        "type": cert_type,
        "key_path": str(certs_dir / f"{domain}.key"),
        "cert_path": str(certs_dir / f"{domain}.crt"),
        "chain_path": str(int_dir / "certs" / "ca-chain.crt"),
        "valid_until": cert.not_valid_after_utc.isoformat(),
    }


def revoke_certificate(ca_dir: Path, serial: int, reason: str = "unspecified") -> Dict:
    """Revoke a certificate by serial number."""
    int_dir = ca_dir / "intermediate-ca"
    index_data = json.loads((int_dir / "index.json").read_text())

    reason_map = {
        "unspecified": x509.ReasonFlags.unspecified,
        "key_compromise": x509.ReasonFlags.key_compromise,
        "ca_compromise": x509.ReasonFlags.ca_compromise,
        "affiliation_changed": x509.ReasonFlags.affiliation_changed,
        "superseded": x509.ReasonFlags.superseded,
        "cessation_of_operation": x509.ReasonFlags.cessation_of_operation,
    }

    found = False
    for cert_entry in index_data["certificates"]:
        if cert_entry["serial"] == serial:
            cert_entry["status"] = "revoked"
            cert_entry["revoked_at"] = datetime.datetime.utcnow().isoformat()
            cert_entry["revocation_reason"] = reason
            found = True
            break

    if not found:
        raise ValueError(f"Certificate with serial {serial} not found")

    if "revoked" not in index_data:
        index_data["revoked"] = []

    index_data["revoked"].append({
        "serial": serial,
        "revoked_at": datetime.datetime.utcnow().isoformat(),
        "reason": reason,
    })

    (int_dir / "index.json").write_text(json.dumps(index_data, indent=2))
    logger.info(f"Revoked certificate serial {serial} (reason: {reason})")

    return {"serial": serial, "status": "revoked", "reason": reason}


def generate_crl(ca_dir: Path, validity_days: int = 30) -> Dict:
    """Generate a Certificate Revocation List."""
    int_dir = ca_dir / "intermediate-ca"

    int_key_data = (int_dir / "private" / "intermediate-ca.key").read_bytes()
    int_key = serialization.load_pem_private_key(int_key_data, password=None, backend=default_backend())

    int_cert_data = (int_dir / "certs" / "intermediate-ca.crt").read_bytes()
    int_cert = x509.load_pem_x509_certificate(int_cert_data, default_backend())

    index_data = json.loads((int_dir / "index.json").read_text())

    now = datetime.datetime.utcnow()
    builder = x509.CertificateRevocationListBuilder()
    builder = builder.issuer_name(int_cert.subject)
    builder = builder.last_update(now)
    builder = builder.next_update(now + datetime.timedelta(days=validity_days))

    reason_map = {
        "unspecified": x509.ReasonFlags.unspecified,
        "key_compromise": x509.ReasonFlags.key_compromise,
        "ca_compromise": x509.ReasonFlags.ca_compromise,
        "superseded": x509.ReasonFlags.superseded,
        "cessation_of_operation": x509.ReasonFlags.cessation_of_operation,
    }

    for entry in index_data.get("revoked", []):
        revoked_cert = (
            x509.RevokedCertificateBuilder()
            .serial_number(entry["serial"])
            .revocation_date(datetime.datetime.fromisoformat(entry["revoked_at"]))
            .add_extension(
                x509.CRLReason(reason_map.get(entry.get("reason", "unspecified"), x509.ReasonFlags.unspecified)),
                critical=False,
            )
            .build()
        )
        builder = builder.add_revoked_certificate(revoked_cert)

    crl = builder.sign(int_key, hashes.SHA256(), default_backend())

    crl_path = int_dir / "crl" / "intermediate.crl"
    crl_path.parent.mkdir(exist_ok=True)
    crl_path.write_bytes(crl.public_bytes(serialization.Encoding.PEM))

    logger.info(f"Generated CRL with {len(index_data.get('revoked', []))} revoked certificates")
    return {
        "crl_path": str(crl_path),
        "revoked_count": len(index_data.get("revoked", [])),
        "next_update": (now + datetime.timedelta(days=validity_days)).isoformat(),
    }


def main():
    parser = argparse.ArgumentParser(description="Certificate Authority Builder")
    subparsers = parser.add_subparsers(dest="command")

    build = subparsers.add_parser("build-ca", help="Build complete CA hierarchy")
    build.add_argument("--output", "-o", default="./pki", help="Output directory")
    build.add_argument("--org", required=True, help="Organization name")
    build.add_argument("--country", default="US", help="Country code")

    issue = subparsers.add_parser("issue-cert", help="Issue a certificate")
    issue.add_argument("--ca-dir", required=True, help="CA directory")
    issue.add_argument("--domain", required=True, help="Domain name")
    issue.add_argument("--type", choices=["server", "client", "both"], default="server")
    issue.add_argument("--days", type=int, default=365, help="Validity days")
    issue.add_argument("--san", nargs="*", help="Additional SAN domains")

    rev = subparsers.add_parser("revoke", help="Revoke a certificate")
    rev.add_argument("--ca-dir", required=True, help="CA directory")
    rev.add_argument("--serial", type=int, required=True, help="Serial number")
    rev.add_argument("--reason", default="unspecified", help="Revocation reason")

    crl = subparsers.add_parser("generate-crl", help="Generate CRL")
    crl.add_argument("--ca-dir", required=True, help="CA directory")
    crl.add_argument("--days", type=int, default=30, help="CRL validity days")

    args = parser.parse_args()

    if args.command == "build-ca":
        output = Path(args.output)
        root = build_root_ca(output, args.org, args.country)
        intermediate = build_intermediate_ca(output, args.org, args.country)
        print(json.dumps({"root_ca": root, "intermediate_ca": intermediate}, indent=2))
    elif args.command == "issue-cert":
        result = issue_certificate(
            Path(args.ca_dir), args.domain, args.type, args.days, args.san
        )
        print(json.dumps(result, indent=2))
    elif args.command == "revoke":
        result = revoke_certificate(Path(args.ca_dir), args.serial, args.reason)
        print(json.dumps(result, indent=2))
    elif args.command == "generate-crl":
        result = generate_crl(Path(args.ca_dir), args.days)
        print(json.dumps(result, indent=2))
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-hsm-for-key-storage/SKILL.md =====
---
name: configuring-hsm-for-key-storage
description: Configures Hardware Security Modules for cryptographic key storage
  using the PKCS#11 standard interface, covering key generation, signing, encryption,
  and key management on physical HSMs and SoftHSM2 for development. Use when protecting
  cryptographic keys so they never leave a hardened device boundary, or when building
  and testing PKCS#11-based key management workflows.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- hsm
- key-management
- pkcs11
- hardware-security
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
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1552.004
- T1555
- T1078
---
# Configuring HSM for Key Storage

## Overview

Hardware Security Modules (HSMs) are tamper-resistant physical devices that safeguard cryptographic keys and perform cryptographic operations in a hardened environment. Keys stored in an HSM never leave the device boundary, providing the highest level of key protection. This skill covers configuring HSMs using the PKCS#11 standard interface, including key generation, signing, encryption, and key management using both physical HSMs and SoftHSM2 for development.


## When to Use

- When deploying or configuring configuring hsm for key storage capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Configure SoftHSM2 as a development PKCS#11 provider
- Generate and manage keys inside the HSM via PKCS#11
- Perform cryptographic operations (sign, verify, encrypt, decrypt) using HSM-resident keys
- Implement HSM-backed certificate authority operations
- Configure key access policies and user authentication
- Interface with cloud HSM services (AWS CloudHSM, Azure)

## Key Concepts

### HSM Compliance Levels

| FIPS Level | Protection | Use Case |
|-----------|-----------|----------|
| FIPS 140-2 Level 1 | Software only | Development |
| FIPS 140-2 Level 2 | Tamper-evident, role-based auth | General production |
| FIPS 140-2 Level 3 | Tamper-resistant, identity-based auth | Financial, government |
| FIPS 140-2 Level 4 | Physical tamper response | Military, classified |

### PKCS#11 Architecture

```
Application --> PKCS#11 API --> HSM Provider --> Hardware HSM
                                    |
                              (SoftHSM2 for dev)
```

### Key Objects in PKCS#11

| Object Type | Description | Operations |
|-------------|-------------|-----------|
| CKO_SECRET_KEY | Symmetric keys (AES) | Encrypt, Decrypt, Wrap |
| CKO_PUBLIC_KEY | Public keys (RSA, EC) | Verify, Encrypt, Wrap |
| CKO_PRIVATE_KEY | Private keys (RSA, EC) | Sign, Decrypt, Unwrap |
| CKO_CERTIFICATE | X.509 certificates | Storage, retrieval |

## Security Considerations

- Never export private keys from HSM (use CKA_EXTRACTABLE=False)
- Use separate slots/partitions for different applications
- Implement multi-person key ceremony for CA root keys
- Enable audit logging for all HSM operations
- Implement HSM backup and disaster recovery
- Use strong PINs and enable SO (Security Officer) PIN

## Validation Criteria

- [ ] SoftHSM2 initializes with token and user PIN
- [ ] AES key generates inside HSM
- [ ] RSA key pair generates inside HSM
- [ ] Encryption/decryption uses HSM-resident keys
- [ ] Signing/verification uses HSM-resident keys
- [ ] Keys cannot be exported (non-extractable)
- [ ] Key listing shows all HSM-stored objects

===== END FILE =====


===== FILE: skills/configuring-hsm-for-key-storage/LICENSE =====

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


===== FILE: skills/configuring-hsm-for-key-storage/assets/template.md =====
# HSM Key Storage Configuration Template

## HSM Selection Matrix

| HSM | FIPS Level | Cloud | On-Premise | Cost |
|-----|-----------|-------|-----------|------|
| SoftHSM2 | N/A (dev) | N/A | Yes | Free |
| AWS CloudHSM | 140-2 L3 | Yes | No | ~$1.60/hr |
| Azure Dedicated HSM | 140-2 L3 | Yes | No | ~$5,500/mo |
| Thales Luna | 140-2 L3 | Both | Yes | License |
| YubiHSM 2 | 140-2 L3 | No | Yes | ~$650 |

## PKCS#11 Key Attributes

```
CKA_TOKEN = True          # Persistent storage
CKA_PRIVATE = True        # Requires login
CKA_SENSITIVE = True      # Cannot be revealed in clear
CKA_EXTRACTABLE = False   # Cannot be exported
CKA_MODIFIABLE = False    # Cannot change attributes
CKA_LABEL = "my-key"      # Human-readable label
CKA_ID = <byte_string>    # Unique identifier
```

## Key Ceremony Checklist

- [ ] Prepare air-gapped workstation with HSM
- [ ] Assemble M-of-N key custodians (quorum)
- [ ] Initialize HSM and set SO/User PINs
- [ ] Generate root CA key in HSM (non-extractable)
- [ ] Generate and sign root CA certificate
- [ ] Export root CA certificate (public only)
- [ ] Verify certificate independently
- [ ] Secure HSM in physical vault
- [ ] Document ceremony in audit log
- [ ] Distribute key custodian tokens/smart cards

===== END FILE =====


===== FILE: skills/configuring-hsm-for-key-storage/references/api-reference.md =====
# HSM Key Storage — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| boto3 | `pip install boto3` | AWS CloudHSM and KMS API |
| python-pkcs11 | `pip install python-pkcs11` | PKCS#11 interface for HSM operations |

## Key boto3 CloudHSMv2 Methods

| Method | Description |
|--------|-------------|
| `describe_clusters()` | List CloudHSM clusters |
| `describe_backups()` | List cluster backups |
| `create_cluster(HsmType, SubnetIds)` | Create new cluster |
| `create_hsm(ClusterId, AvailabilityZone)` | Add HSM to cluster |
| `initialize_cluster(ClusterId, SignedCert, TrustAnchor)` | Initialize cluster |

## Key boto3 KMS Methods (Custom Key Store)

| Method | Description |
|--------|-------------|
| `create_custom_key_store()` | Create KMS custom key store backed by CloudHSM |
| `describe_key(KeyId)` | Get key metadata including CustomKeyStoreId |
| `create_key(Origin="AWS_CLOUDHSM", CustomKeyStoreId=)` | Create key in HSM |

## PKCS#11 Operations

| Function | Description |
|----------|-------------|
| `C_Initialize` | Initialize PKCS#11 library |
| `C_OpenSession` | Open session with HSM |
| `C_Login` | Authenticate with HSM PIN |
| `C_GenerateKeyPair` | Generate asymmetric key pair |
| `C_Sign / C_Verify` | Cryptographic signing operations |

## HSM Types

| Type | Use Case |
|------|----------|
| AWS CloudHSM | Cloud-native FIPS 140-2 Level 3 |
| Thales Luna | On-premises enterprise HSM |
| nCipher nShield | High-assurance code signing |

## External References

- [AWS CloudHSM Docs](https://docs.aws.amazon.com/cloudhsm/)
- [boto3 CloudHSMv2](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/cloudhsmv2.html)
- [PKCS#11 Standard](https://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/pkcs11-base-v2.40.html)

===== END FILE =====


===== FILE: skills/configuring-hsm-for-key-storage/references/standards.md =====
# Standards and References - HSM for Key Storage

## Primary Standards

### PKCS#11 v3.0 (Cryptoki)
- **URL**: https://docs.oasis-open.org/pkcs11/pkcs11-base/v3.0/pkcs11-base-v3.0.html
- **Description**: Standard API for cryptographic token interface

### FIPS 140-2 / FIPS 140-3
- **URL**: https://csrc.nist.gov/publications/detail/fips/140/3/final
- **Description**: Security requirements for cryptographic modules
- **CMVP**: https://csrc.nist.gov/projects/cryptographic-module-validation-program

### NIST SP 800-57 Part 1 Rev. 5
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final
- **Description**: Key management recommendations (HSM storage for high-value keys)

## HSM Products

### SoftHSM2 (Development/Testing)
- **URL**: https://www.opendnssec.org/softhsm/
- **GitHub**: https://github.com/opendnssec/SoftHSMv2
- **Description**: Software-only PKCS#11 implementation for testing

### AWS CloudHSM
- **URL**: https://docs.aws.amazon.com/cloudhsm/
- **FIPS**: 140-2 Level 3
- **PKCS#11**: https://docs.aws.amazon.com/cloudhsm/latest/userguide/pkcs11-library.html

### Azure Dedicated HSM
- **URL**: https://docs.microsoft.com/en-us/azure/dedicated-hsm/
- **FIPS**: 140-2 Level 3 (Thales Luna)

### Thales Luna HSM
- **URL**: https://cpl.thalesgroup.com/encryption/hardware-security-modules
- **FIPS**: 140-2 Level 3

## Python Libraries

### python-pkcs11
- **URL**: https://python-pkcs11.readthedocs.io/
- **PyPI**: https://pypi.org/project/python-pkcs11/

### PyKCS11
- **URL**: https://github.com/LudovicRousseau/PyKCS11
- **PyPI**: https://pypi.org/project/PyKCS11/

===== END FILE =====


===== FILE: skills/configuring-hsm-for-key-storage/references/workflows.md =====
# Workflows - HSM for Key Storage

## Workflow 1: SoftHSM2 Initialization

```bash
# Install SoftHSM2
# Ubuntu: apt install softhsm2
# macOS: brew install softhsm

# Initialize a token
softhsm2-util --init-token --slot 0 --label "MyToken" --pin 1234 --so-pin 5678

# List tokens
softhsm2-util --show-slots
```

## Workflow 2: Key Generation via PKCS#11

```
[Connect to HSM]
(open session, login with PIN)
      |
[Generate Key]:
  Symmetric: AES-256 (CKM_AES_KEY_GEN)
  Asymmetric: RSA-4096 (CKM_RSA_PKCS_KEY_PAIR_GEN)
  Asymmetric: EC P-256 (CKM_EC_KEY_PAIR_GEN)
      |
[Set Key Attributes]:
  CKA_EXTRACTABLE = False
  CKA_SENSITIVE = True
  CKA_TOKEN = True (persistent)
  CKA_LABEL = "my-key-001"
      |
[Key Stored in HSM]
(returns handle, not key material)
```

## Workflow 3: Cryptographic Operations

```
[Application Request]
      |
[Open PKCS#11 Session]
      |
[Find Key by Label/ID]
      |
[Perform Operation on HSM]:
  Sign:    C_SignInit + C_Sign
  Verify:  C_VerifyInit + C_Verify
  Encrypt: C_EncryptInit + C_Encrypt
  Decrypt: C_DecryptInit + C_Decrypt
      |
[Return Result to Application]
(key never leaves HSM)
      |
[Close Session]
```

## Workflow 4: HSM Key Ceremony (Root CA)

```
[Prepare Air-Gapped HSM Station]
      |
[Multi-Person Authentication]
(M-of-N key custodians present)
      |
[Generate Root CA Key in HSM]
(CKA_EXTRACTABLE=False)
      |
[Sign Root CA Certificate]
(self-signed, 20-year validity)
      |
[Export Root CA Certificate]
(public certificate only)
      |
[Secure HSM in Safe/Vault]
(offline until next signing ceremony)
```

===== END FILE =====


===== FILE: skills/configuring-hsm-for-key-storage/scripts/agent.py =====
#!/usr/bin/env python3
"""HSM key storage management agent using PKCS#11 and AWS CloudHSM."""

import json
import sys
import argparse
from datetime import datetime

try:
    import boto3
    from botocore.exceptions import ClientError
except ImportError:
    print("Install: pip install boto3")
    sys.exit(1)


def list_cloudhsm_clusters(session):
    """List AWS CloudHSM clusters."""
    client = session.client("cloudhsmv2")
    clusters = []
    response = client.describe_clusters()
    for cluster in response.get("Clusters", []):
        clusters.append({
            "id": cluster["ClusterId"],
            "state": cluster["State"],
            "hsm_type": cluster["HsmType"],
            "vpc_id": cluster.get("VpcId", ""),
            "hsms": len(cluster.get("Hsms", [])),
            "security_group": cluster.get("SecurityGroup", ""),
        })
    return clusters


def list_hsm_instances(session, cluster_id):
    """List HSM instances in a cluster."""
    client = session.client("cloudhsmv2")
    response = client.describe_clusters(Filters={"clusterIds": [cluster_id]})
    hsms = []
    for cluster in response.get("Clusters", []):
        for hsm in cluster.get("Hsms", []):
            hsms.append({
                "hsm_id": hsm["HsmId"],
                "az": hsm.get("AvailabilityZone", ""),
                "ip": hsm.get("EniIp", ""),
                "state": hsm.get("State", ""),
            })
    return hsms


def audit_kms_keys(session):
    """Audit KMS keys backed by CloudHSM custom key store."""
    kms = session.client("kms")
    custom_store_keys = []
    paginator = kms.get_paginator("list_keys")
    for page in paginator.paginate():
        for key in page["Keys"]:
            try:
                desc = kms.describe_key(KeyId=key["KeyId"])
                meta = desc["KeyMetadata"]
                if meta.get("CustomKeyStoreId"):
                    custom_store_keys.append({
                        "key_id": meta["KeyId"],
                        "description": meta.get("Description", ""),
                        "key_state": meta["KeyState"],
                        "key_spec": meta.get("KeySpec", ""),
                        "custom_store_id": meta["CustomKeyStoreId"],
                        "origin": meta.get("Origin", ""),
                    })
            except ClientError:
                pass
    return custom_store_keys


def check_cloudhsm_backup(session):
    """Check CloudHSM backup status."""
    client = session.client("cloudhsmv2")
    response = client.describe_backups()
    backups = []
    for backup in response.get("Backups", []):
        backups.append({
            "backup_id": backup["BackupId"],
            "state": backup["BackupState"],
            "cluster_id": backup.get("ClusterId", ""),
            "create_time": str(backup.get("CreateTimestamp", "")),
        })
    return sorted(backups, key=lambda x: x["create_time"], reverse=True)


def run_audit(profile=None, region="us-east-1"):
    """Execute HSM key storage audit."""
    session = boto3.Session(profile_name=profile, region_name=region)
    print(f"\n{'='*60}")
    print(f"  HSM KEY STORAGE AUDIT")
    print(f"  Region: {region}")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    clusters = list_cloudhsm_clusters(session)
    print(f"--- CLOUDHSM CLUSTERS ({len(clusters)}) ---")
    for c in clusters:
        print(f"  {c['id']}: {c['state']} ({c['hsm_type']}, {c['hsms']} HSMs)")

    for cluster in clusters:
        hsms = list_hsm_instances(session, cluster["id"])
        print(f"\n--- HSMs in {cluster['id']} ({len(hsms)}) ---")
        for h in hsms:
            print(f"  {h['hsm_id']}: {h['state']} ({h['az']}, {h['ip']})")

    keys = audit_kms_keys(session)
    print(f"\n--- CUSTOM KEY STORE KEYS ({len(keys)}) ---")
    for k in keys[:10]:
        print(f"  {k['key_id']}: {k['key_state']} ({k['key_spec']})")

    backups = check_cloudhsm_backup(session)
    print(f"\n--- BACKUPS ({len(backups)}) ---")
    for b in backups[:5]:
        print(f"  {b['backup_id']}: {b['state']} ({b['create_time']})")

    return {"clusters": clusters, "keys": keys, "backups": backups}


def main():
    parser = argparse.ArgumentParser(description="HSM Key Storage Agent")
    parser.add_argument("--profile", help="AWS CLI profile")
    parser.add_argument("--region", default="us-east-1", help="AWS region")
    parser.add_argument("--audit", action="store_true", help="Run full audit")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    if args.audit:
        report = run_audit(args.profile, args.region)
        if args.output:
            with open(args.output, "w") as f:
                json.dump(report, f, indent=2, default=str)
            print(f"\n[+] Report saved to {args.output}")
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-hsm-for-key-storage/scripts/process.py =====
#!/usr/bin/env python3
"""
HSM Key Storage Configuration Tool (PKCS#11)

Demonstrates HSM key management using PKCS#11 interface with SoftHSM2
for development and testing. Covers key generation, signing, encryption,
and key management operations.

Requirements:
    pip install python-pkcs11 asn1crypto
    # Also requires SoftHSM2 installed:
    # Ubuntu: apt install softhsm2
    # macOS: brew install softhsm

Usage:
    python process.py init-token --label MyToken --pin 1234 --so-pin 5678
    python process.py generate-aes --token MyToken --pin 1234 --label my-aes-key
    python process.py generate-rsa --token MyToken --pin 1234 --label my-rsa-key
    python process.py list-keys --token MyToken --pin 1234
    python process.py sign --token MyToken --pin 1234 --key-label my-rsa-key --input data.txt
    python process.py encrypt --token MyToken --pin 1234 --key-label my-aes-key --input data.txt
"""

import os
import sys
import json
import argparse
import logging
import subprocess
import platform
from pathlib import Path
from typing import Dict, List, Optional

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

# Common SoftHSM2 library paths
SOFTHSM_PATHS = {
    "linux": [
        "/usr/lib/softhsm/libsofthsm2.so",
        "/usr/lib/x86_64-linux-gnu/softhsm/libsofthsm2.so",
        "/usr/local/lib/softhsm/libsofthsm2.so",
    ],
    "darwin": [
        "/usr/local/lib/softhsm/libsofthsm2.so",
        "/opt/homebrew/lib/softhsm/libsofthsm2.so",
    ],
    "win32": [
        r"C:\SoftHSM2\lib\softhsm2-x64.dll",
        r"C:\Program Files\SoftHSM2\lib\softhsm2-x64.dll",
    ],
}


def find_softhsm_lib() -> Optional[str]:
    """Find the SoftHSM2 library path."""
    system = platform.system().lower()
    if system == "windows":
        paths = SOFTHSM_PATHS.get("win32", [])
    elif system == "darwin":
        paths = SOFTHSM_PATHS.get("darwin", [])
    else:
        paths = SOFTHSM_PATHS.get("linux", [])

    for path in paths:
        if Path(path).exists():
            return path

    env_path = os.environ.get("SOFTHSM2_LIB")
    if env_path and Path(env_path).exists():
        return env_path

    return None


def init_token_via_cli(label: str, pin: str, so_pin: str) -> Dict:
    """Initialize a SoftHSM2 token using command-line tool."""
    try:
        result = subprocess.run(
            ["softhsm2-util", "--init-token", "--free",
             "--label", label, "--pin", pin, "--so-pin", so_pin],
            capture_output=True, text=True, timeout=30,
        )
        if result.returncode == 0:
            logger.info(f"Token '{label}' initialized successfully")
            slot_info = result.stdout.strip()
            return {
                "status": "success",
                "label": label,
                "message": slot_info or "Token initialized",
            }
        else:
            return {"status": "error", "message": result.stderr.strip()}
    except FileNotFoundError:
        return {
            "status": "error",
            "message": "softhsm2-util not found. Install SoftHSM2 first.",
        }
    except Exception as e:
        return {"status": "error", "message": str(e)}


def pkcs11_operations_demo(token_label: str, pin: str, lib_path: str) -> Dict:
    """
    Demonstrate PKCS#11 operations using python-pkcs11.
    This requires the python-pkcs11 package and SoftHSM2.
    """
    try:
        import pkcs11
        from pkcs11 import KeyType, Attribute, ObjectClass, Mechanism
        from pkcs11.util.rsa import encode_rsa_public_key
        from pkcs11.util.ec import encode_named_curve_parameters
    except ImportError:
        return {
            "status": "error",
            "message": "python-pkcs11 not installed. Run: pip install python-pkcs11",
        }

    try:
        lib = pkcs11.lib(lib_path)
        token = lib.get_token(token_label=token_label)
    except Exception as e:
        return {"status": "error", "message": f"Cannot access token: {e}"}

    results = {"operations": []}

    with token.open(user_pin=pin, rw=True) as session:
        # Generate AES-256 key
        try:
            aes_key = session.generate_key(
                KeyType.AES, 256,
                label="demo-aes-256",
                store=True,
                capabilities=pkcs11.defaults.DEFAULT_KEY_CAPABILITIES[KeyType.AES],
            )
            results["operations"].append({
                "operation": "generate_aes_key",
                "status": "success",
                "label": "demo-aes-256",
                "key_type": "AES-256",
            })
        except Exception as e:
            results["operations"].append({
                "operation": "generate_aes_key",
                "status": "error",
                "message": str(e),
            })

        # Generate RSA-2048 key pair
        try:
            pub_key, priv_key = session.generate_keypair(
                KeyType.RSA, 2048,
                label="demo-rsa-2048",
                store=True,
            )
            results["operations"].append({
                "operation": "generate_rsa_keypair",
                "status": "success",
                "label": "demo-rsa-2048",
                "key_type": "RSA-2048",
            })
        except Exception as e:
            results["operations"].append({
                "operation": "generate_rsa_keypair",
                "status": "error",
                "message": str(e),
            })

        # AES Encryption/Decryption
        try:
            for key in session.get_objects({
                Attribute.CLASS: ObjectClass.SECRET_KEY,
                Attribute.LABEL: "demo-aes-256",
            }):
                iv = os.urandom(16)
                plaintext = b"HSM encryption test data for PKCS#11"
                padded = plaintext + (b"\x10" * (16 - len(plaintext) % 16))
                ciphertext = key.encrypt(padded, mechanism=Mechanism.AES_CBC_PAD, mechanism_param=iv)
                decrypted = key.decrypt(ciphertext, mechanism=Mechanism.AES_CBC_PAD, mechanism_param=iv)
                results["operations"].append({
                    "operation": "aes_encrypt_decrypt",
                    "status": "success",
                    "match": decrypted.rstrip(b"\x10")[:len(plaintext)] == plaintext,
                })
                break
        except Exception as e:
            results["operations"].append({
                "operation": "aes_encrypt_decrypt",
                "status": "error",
                "message": str(e),
            })

        # RSA Sign/Verify
        try:
            for key in session.get_objects({
                Attribute.CLASS: ObjectClass.PRIVATE_KEY,
                Attribute.LABEL: "demo-rsa-2048",
            }):
                data = b"Data to sign with HSM-resident RSA key"
                signature = key.sign(data, mechanism=Mechanism.SHA256_RSA_PKCS)
                results["operations"].append({
                    "operation": "rsa_sign",
                    "status": "success",
                    "signature_length": len(signature),
                })
                break

            for key in session.get_objects({
                Attribute.CLASS: ObjectClass.PUBLIC_KEY,
                Attribute.LABEL: "demo-rsa-2048",
            }):
                valid = key.verify(data, signature, mechanism=Mechanism.SHA256_RSA_PKCS)
                results["operations"].append({
                    "operation": "rsa_verify",
                    "status": "success",
                    "valid": True,
                })
                break
        except Exception as e:
            results["operations"].append({
                "operation": "rsa_sign_verify",
                "status": "error",
                "message": str(e),
            })

        # List all objects
        try:
            objects = []
            for obj in session.get_objects():
                obj_info = {
                    "label": str(obj.get(Attribute.LABEL, "N/A")),
                    "class": str(obj.object_class),
                }
                objects.append(obj_info)
            results["stored_objects"] = objects
        except Exception as e:
            results["stored_objects_error"] = str(e)

    results["status"] = "success"
    return results


def list_tokens(lib_path: str) -> Dict:
    """List all available PKCS#11 tokens."""
    try:
        import pkcs11
        lib = pkcs11.lib(lib_path)
        tokens = []
        for slot in lib.get_slots(token_present=True):
            token = slot.get_token()
            tokens.append({
                "slot_id": slot.slot_id,
                "label": token.label.strip(),
                "manufacturer": token.manufacturer_id.strip() if hasattr(token, 'manufacturer_id') else "N/A",
            })
        return {"status": "success", "tokens": tokens}
    except ImportError:
        return {"status": "error", "message": "python-pkcs11 not installed"}
    except Exception as e:
        return {"status": "error", "message": str(e)}


def generate_hsm_config() -> Dict:
    """Generate HSM configuration templates for different providers."""
    configs = {
        "softhsm2": {
            "description": "SoftHSM2 (Development/Testing)",
            "install": {
                "ubuntu": "apt install softhsm2",
                "macos": "brew install softhsm",
                "windows": "Download from https://github.com/disig/SoftHSM2-for-Windows",
            },
            "config_file": "softhsm2.conf",
            "config_content": "directories.tokendir = /var/lib/softhsm/tokens/\nobjectstore.backend = file\nlog.level = INFO\n",
            "init_command": "softhsm2-util --init-token --free --label MyToken --pin 1234 --so-pin 5678",
        },
        "aws_cloudhsm": {
            "description": "AWS CloudHSM",
            "setup_steps": [
                "Create CloudHSM cluster in AWS Console",
                "Initialize the cluster",
                "Install CloudHSM client on EC2 instance",
                "Activate the cluster with crypto officer credentials",
                "Install PKCS#11 library from AWS",
            ],
            "pkcs11_lib": "/opt/cloudhsm/lib/libcloudhsm_pkcs11.so",
            "config_file": "/opt/cloudhsm/etc/cloudhsm_client.cfg",
        },
        "azure_dedicated_hsm": {
            "description": "Azure Dedicated HSM (Thales Luna)",
            "setup_steps": [
                "Provision Dedicated HSM via Azure Portal",
                "Configure network connectivity (VNet peering)",
                "Install Luna client on application server",
                "Register client with HSM",
                "Create partition for application",
            ],
            "pkcs11_lib": "/usr/safenet/lunaclient/lib/libCryptoki2_64.so",
        },
    }
    return configs


def main():
    parser = argparse.ArgumentParser(description="HSM Key Storage Configuration Tool")
    subparsers = parser.add_subparsers(dest="command")

    init = subparsers.add_parser("init-token", help="Initialize HSM token")
    init.add_argument("--label", required=True, help="Token label")
    init.add_argument("--pin", required=True, help="User PIN")
    init.add_argument("--so-pin", required=True, help="Security Officer PIN")

    demo = subparsers.add_parser("demo", help="Run PKCS#11 operations demo")
    demo.add_argument("--token", required=True, help="Token label")
    demo.add_argument("--pin", required=True, help="User PIN")
    demo.add_argument("--lib", help="PKCS#11 library path")

    lst = subparsers.add_parser("list-tokens", help="List PKCS#11 tokens")
    lst.add_argument("--lib", help="PKCS#11 library path")

    subparsers.add_parser("config-templates", help="Show HSM configuration templates")

    args = parser.parse_args()

    if args.command == "init-token":
        result = init_token_via_cli(args.label, args.pin, args.so_pin)
        print(json.dumps(result, indent=2))
    elif args.command == "demo":
        lib_path = args.lib or find_softhsm_lib()
        if not lib_path:
            print(json.dumps({
                "status": "error",
                "message": "SoftHSM2 library not found. Set SOFTHSM2_LIB env var or use --lib.",
            }, indent=2))
            sys.exit(1)
        result = pkcs11_operations_demo(args.token, args.pin, lib_path)
        print(json.dumps(result, indent=2))
    elif args.command == "list-tokens":
        lib_path = args.lib or find_softhsm_lib()
        if not lib_path:
            print(json.dumps({"status": "error", "message": "SoftHSM2 library not found."}, indent=2))
            sys.exit(1)
        result = list_tokens(lib_path)
        print(json.dumps(result, indent=2))
    elif args.command == "config-templates":
        configs = generate_hsm_config()
        print(json.dumps(configs, indent=2))
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-tls-1-3-for-secure-communications/SKILL.md =====
---
name: configuring-tls-1-3-for-secure-communications
description: Configures TLS 1.3 (RFC 8446) on servers, covering cipher suite and
  key-exchange group selection, and validates the resulting configuration with openssl
  s_client and testssl.sh. Use when deploying or hardening TLS 1.3 for secure communications,
  or when testing a server for common TLS misconfigurations and vulnerabilities.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- tls
- ssl
- transport-security
- network-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1557
- T1040
- T1573.002
- T1539
- T1556.004
---
# Configuring TLS 1.3 for Secure Communications

## Overview

TLS 1.3 (RFC 8446) is the latest version of the Transport Layer Security protocol, providing significant improvements over TLS 1.2 in both security and performance. It reduces handshake latency to 1-RTT (and 0-RTT for resumed sessions), removes obsolete cipher suites, and mandates perfect forward secrecy. This skill covers configuring TLS 1.3 on servers, validating configurations, and testing for common misconfigurations.


## When to Use

- When deploying or configuring configuring tls 1 3 for secure communications capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Configure TLS 1.3 on nginx and Apache web servers
- Implement TLS 1.3 in Python applications using the ssl module
- Validate TLS configurations with openssl and testssl.sh
- Understand TLS 1.3 cipher suites and key exchange mechanisms
- Configure 0-RTT early data with appropriate protections
- Disable legacy TLS versions (1.0, 1.1) and weak cipher suites

## Key Concepts

### TLS 1.3 Cipher Suites

| Cipher Suite | Key Exchange | Authentication | Encryption | Hash |
|-------------|-------------|----------------|------------|------|
| TLS_AES_256_GCM_SHA384 | ECDHE/DHE | Certificate | AES-256-GCM | SHA-384 |
| TLS_AES_128_GCM_SHA256 | ECDHE/DHE | Certificate | AES-128-GCM | SHA-256 |
| TLS_CHACHA20_POLY1305_SHA256 | ECDHE/DHE | Certificate | ChaCha20-Poly1305 | SHA-256 |

### TLS 1.3 vs 1.2 Improvements

- **1-RTT Handshake**: Full handshake completes in one round trip (vs 2 in TLS 1.2)
- **0-RTT Resumption**: Resumed connections can send data immediately
- **No RSA Key Exchange**: Only ephemeral Diffie-Hellman (mandatory PFS)
- **Simplified Cipher Suites**: Removed CBC, RC4, 3DES, static RSA, SHA-1
- **Encrypted Handshake**: Server certificate is encrypted after ServerHello

### Key Exchange Groups

- **x25519**: Curve25519 ECDH (preferred, fast)
- **secp256r1**: NIST P-256 ECDH (widely supported)
- **secp384r1**: NIST P-384 ECDH (higher security margin)
- **x448**: Curve448 ECDH (highest security)

## Workflow

1. Verify OpenSSL version supports TLS 1.3 (1.1.1+)
2. Generate or obtain TLS certificate and private key
3. Configure server to use TLS 1.3 cipher suites
4. Disable TLS 1.0 and 1.1 (optionally keep 1.2 for compatibility)
5. Set preferred key exchange groups
6. Enable OCSP stapling for certificate validation
7. Test configuration with openssl s_client and testssl.sh
8. Configure HSTS header for HTTP Strict Transport Security

## Security Considerations

- 0-RTT data is vulnerable to replay attacks; limit to idempotent requests
- Always include TLS 1.2 fallback if legacy client support is required
- Use ECDSA certificates for better performance (vs RSA)
- Enable OCSP stapling to improve client certificate validation
- Set HSTS header with long max-age and includeSubDomains
- Monitor for certificate transparency logs

## Validation Criteria

- [ ] TLS 1.3 handshake completes successfully
- [ ] Only approved cipher suites are offered
- [ ] Perfect forward secrecy is enforced
- [ ] TLS 1.0 and 1.1 are rejected
- [ ] OCSP stapling is functional
- [ ] Certificate chain is valid and complete
- [ ] testssl.sh reports no vulnerabilities

===== END FILE =====


===== FILE: skills/configuring-tls-1-3-for-secure-communications/LICENSE =====

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


===== FILE: skills/configuring-tls-1-3-for-secure-communications/assets/template.md =====
# TLS 1.3 Configuration Template

## Pre-Configuration Checklist

- [ ] Verify OpenSSL version >= 1.1.1 (`openssl version`)
- [ ] Obtain valid TLS certificate from trusted CA
- [ ] Identify all server endpoints requiring TLS
- [ ] Determine minimum TLS version (1.2 or 1.3 only)
- [ ] Plan certificate renewal automation (Let's Encrypt / ACME)
- [ ] Review compliance requirements (PCI-DSS, HIPAA)

## nginx Configuration Template

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
ssl_prefer_server_ciphers off;
ssl_ecdh_curve X25519:secp256r1:secp384r1;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

## Python TLS 1.3 Client Template

```python
import ssl
import socket

context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
context.minimum_version = ssl.TLSVersion.TLSv1_3
context.load_default_certs()

with socket.create_connection(("example.com", 443)) as sock:
    with context.wrap_socket(sock, server_hostname="example.com") as tls:
        print(f"Protocol: {tls.version()}")
        print(f"Cipher: {tls.cipher()}")
```

## Validation Commands

```bash
# Test TLS 1.3 support
openssl s_client -connect example.com:443 -tls1_3

# Show full certificate chain
openssl s_client -connect example.com:443 -showcerts

# List supported cipher suites
openssl s_client -connect example.com:443 -cipher 'ALL' -tls1_3

# Test with testssl.sh
./testssl.sh --protocols --ciphers --headers example.com
```

## Security Headers Checklist

| Header | Value | Purpose |
|--------|-------|---------|
| Strict-Transport-Security | max-age=63072000; includeSubDomains; preload | Force HTTPS |
| X-Content-Type-Options | nosniff | Prevent MIME sniffing |
| X-Frame-Options | DENY | Prevent clickjacking |
| Content-Security-Policy | default-src 'self' | Prevent XSS |
| Referrer-Policy | strict-origin-when-cross-origin | Limit referrer leakage |

===== END FILE =====


===== FILE: skills/configuring-tls-1-3-for-secure-communications/references/api-reference.md =====
# TLS 1.3 Configuration — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| cryptography | `pip install cryptography` | X.509 certificate parsing |
| ssl | stdlib | TLS connection testing |
| sslyze | `pip install sslyze` | Comprehensive TLS/SSL scanner |

## Python ssl Module Methods

| Method | Description |
|--------|-------------|
| `ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)` | Create TLS client context |
| `ctx.minimum_version = ssl.TLSVersion.TLSv1_3` | Set minimum TLS version |
| `ctx.wrap_socket(sock, server_hostname=)` | Wrap socket with TLS |
| `ssock.cipher()` | Get negotiated cipher tuple |
| `ssock.getpeercert(binary_form=True)` | Get server certificate DER bytes |

## TLS 1.3 Cipher Suites

| Cipher Suite | Security |
|-------------|----------|
| TLS_AES_256_GCM_SHA384 | Recommended |
| TLS_AES_128_GCM_SHA256 | Recommended |
| TLS_CHACHA20_POLY1305_SHA256 | Recommended (mobile) |

## Deprecated Versions

| Version | Status | Risk |
|---------|--------|------|
| SSL 3.0 | Deprecated (RFC 7568) | POODLE attack |
| TLS 1.0 | Deprecated (RFC 8996) | BEAST, CRIME |
| TLS 1.1 | Deprecated (RFC 8996) | Weak ciphers |

## External References

- [RFC 8446 TLS 1.3](https://datatracker.ietf.org/doc/html/rfc8446)
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)
- [sslyze Documentation](https://nabla-c0d3.github.io/sslyze/documentation/)
- [cryptography.io Docs](https://cryptography.io/)

===== END FILE =====


===== FILE: skills/configuring-tls-1-3-for-secure-communications/references/standards.md =====
# Standards and References - TLS 1.3 Configuration

## Primary Standards

### RFC 8446 - The Transport Layer Security (TLS) Protocol Version 1.3
- **URL**: https://www.rfc-editor.org/rfc/rfc8446
- **Description**: The core TLS 1.3 specification
- **Key changes**: 1-RTT handshake, mandatory PFS, removed RSA key transport, encrypted handshake messages

### RFC 8447 - IANA Registry Updates for TLS and DTLS
- **URL**: https://www.rfc-editor.org/rfc/rfc8447
- **Description**: Updates IANA registries for TLS cipher suites and extensions

### RFC 8449 - Record Size Limit Extension for TLS
- **URL**: https://www.rfc-editor.org/rfc/rfc8449
- **Description**: Allows endpoints to negotiate maximum record size

### RFC 8470 - Using Early Data in HTTP (0-RTT)
- **URL**: https://www.rfc-editor.org/rfc/rfc8470
- **Description**: Defines how 0-RTT early data works with HTTP, including replay protections

### RFC 6961 - TLS Multiple Certificate Status Extension (OCSP Stapling)
- **URL**: https://www.rfc-editor.org/rfc/rfc6961
- **Description**: Allows servers to provide OCSP responses during handshake

### RFC 6797 - HTTP Strict Transport Security (HSTS)
- **URL**: https://www.rfc-editor.org/rfc/rfc6797
- **Description**: Forces browsers to use HTTPS for all connections

## NIST Guidelines

### NIST SP 800-52 Rev. 2 - Guidelines for TLS Implementations
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-52/rev-2/final
- **Description**: Federal guidelines for TLS deployment
- **TLS 1.3**: Recommended for all new deployments
- **TLS 1.2**: Acceptable with approved cipher suites
- **TLS 1.0/1.1**: Prohibited

### NIST SP 800-57 Part 3 Rev. 1 - Application-Specific Key Management
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-57-part-3/rev-1/final
- **Description**: Key management guidance for TLS

## Testing Tools

### testssl.sh
- **URL**: https://testssl.sh/
- **GitHub**: https://github.com/drwetter/testssl.sh
- **Description**: Command-line tool for checking TLS/SSL configurations

### SSL Labs Server Test
- **URL**: https://www.ssllabs.com/ssltest/
- **Description**: Online TLS configuration analyzer (Qualys)

### Mozilla SSL Configuration Generator
- **URL**: https://ssl-config.mozilla.org/
- **Description**: Generate recommended TLS configurations for various servers

## Compliance

### PCI DSS v4.0
- TLS 1.0 and early TLS prohibited since June 2018
- TLS 1.2+ required; TLS 1.3 recommended
- Strong cipher suites must be configured

### HIPAA
- Encryption in transit required for ePHI
- TLS 1.2+ satisfies the requirement

===== END FILE =====


===== FILE: skills/configuring-tls-1-3-for-secure-communications/references/workflows.md =====
# Workflows - Configuring TLS 1.3

## Workflow 1: TLS 1.3 Handshake (1-RTT)

```
Client                              Server
  |                                    |
  |--- ClientHello ------------------>|
  |    (supported_versions: TLS 1.3)  |
  |    (key_share: x25519)            |
  |    (signature_algorithms)         |
  |    (cipher_suites)                |
  |                                    |
  |<-- ServerHello -------------------|
  |    (selected cipher suite)        |
  |    (key_share: x25519)            |
  |<-- {EncryptedExtensions} ---------|
  |<-- {Certificate} -----------------|
  |<-- {CertificateVerify} -----------|
  |<-- {Finished} --------------------|
  |                                    |
  |--- {Finished} ------------------->|
  |                                    |
  |<== Application Data ==============>|
```

## Workflow 2: nginx TLS 1.3 Configuration

```
1. Check OpenSSL version (>= 1.1.1)
   $ openssl version

2. Generate ECDSA certificate
   $ openssl ecparam -genkey -name prime256v1 -out server.key
   $ openssl req -new -x509 -key server.key -out server.crt -days 365

3. Configure nginx
   Edit /etc/nginx/nginx.conf

4. Test configuration
   $ nginx -t

5. Reload nginx
   $ systemctl reload nginx

6. Verify TLS 1.3
   $ openssl s_client -connect localhost:443 -tls1_3
```

## Workflow 3: TLS Configuration Validation

```
[Server] --> [openssl s_client test]
                  |
          [Check protocol version]
          [Check cipher suite]
          [Check certificate chain]
                  |
          [testssl.sh full scan]
                  |
          [Check for vulnerabilities]
          - BEAST, POODLE, Heartbleed
          - ROBOT, DROWN, FREAK
          - Weak ciphers, expired certs
                  |
          [SSL Labs grade assessment]
          Target: A+ rating
```

## Workflow 4: Certificate Lifecycle

```
[Generate Key Pair]
      |
[Create CSR] --> [Submit to CA]
                       |
               [CA Issues Certificate]
                       |
               [Install Certificate]
                       |
               [Configure OCSP Stapling]
                       |
               [Set Up Auto-Renewal]
               (certbot / ACME)
                       |
               [Monitor Expiration]
```

===== END FILE =====


===== FILE: skills/configuring-tls-1-3-for-secure-communications/scripts/agent.py =====
#!/usr/bin/env python3
"""TLS 1.3 configuration audit agent using ssl and cryptography libraries."""

import json
import sys
import argparse
import ssl
import socket
from datetime import datetime

try:
    from cryptography import x509
except ImportError:
    print("Install: pip install cryptography")
    sys.exit(1)


def check_tls_versions(host, port=443):
    """Check supported TLS versions on a server."""
    results = {}
    protocols = {
        "TLSv1.0": ssl.TLSVersion.TLSv1 if hasattr(ssl.TLSVersion, 'TLSv1') else None,
        "TLSv1.1": ssl.TLSVersion.TLSv1_1 if hasattr(ssl.TLSVersion, 'TLSv1_1') else None,
        "TLSv1.2": ssl.TLSVersion.TLSv1_2,
        "TLSv1.3": ssl.TLSVersion.TLSv1_3,
    }
    for version_name, version in protocols.items():
        if version is None:
            results[version_name] = {"supported": False, "note": "Not available in this Python build"}
            continue
        try:
            ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
            ctx.check_hostname = False
            ctx.verify_mode = ssl.CERT_NONE
            ctx.minimum_version = version
            ctx.maximum_version = version
            with socket.create_connection((host, port), timeout=5) as sock:
                with ctx.wrap_socket(sock, server_hostname=host) as ssock:
                    results[version_name] = {
                        "supported": True,
                        "cipher": ssock.cipher()[0],
                        "severity": "CRITICAL" if "1.0" in version_name or "1.1" in version_name else "INFO",
                    }
        except (ssl.SSLError, ConnectionRefusedError, socket.timeout, OSError):
            results[version_name] = {"supported": False}
    return results


def get_certificate_info(host, port=443):
    """Retrieve and analyze server certificate."""
    ctx = ssl.create_default_context()
    ctx.check_hostname = False
    ctx.verify_mode = ssl.CERT_NONE
    try:
        with socket.create_connection((host, port), timeout=5) as sock:
            with ctx.wrap_socket(sock, server_hostname=host) as ssock:
                der_cert = ssock.getpeercert(binary_form=True)
                cert = x509.load_der_x509_certificate(der_cert)
                days_remaining = (cert.not_valid_after_utc.replace(tzinfo=None) - datetime.utcnow()).days
                return {
                    "subject": cert.subject.rfc4514_string(),
                    "issuer": cert.issuer.rfc4514_string(),
                    "not_after": str(cert.not_valid_after_utc),
                    "days_remaining": days_remaining,
                    "key_size": cert.public_key().key_size,
                    "signature_algorithm": cert.signature_hash_algorithm.name if cert.signature_hash_algorithm else "unknown",
                    "issues": [],
                }
    except Exception as e:
        return {"error": str(e)}


def check_cipher_suites(host, port=443):
    """Check negotiated cipher suites."""
    ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
    ctx.check_hostname = False
    ctx.verify_mode = ssl.CERT_NONE
    try:
        with socket.create_connection((host, port), timeout=5) as sock:
            with ctx.wrap_socket(sock, server_hostname=host) as ssock:
                cipher = ssock.cipher()
                return {
                    "negotiated_cipher": cipher[0],
                    "protocol": cipher[1],
                    "bits": cipher[2],
                    "tls13_ciphers": ["TLS_AES_256_GCM_SHA384", "TLS_AES_128_GCM_SHA256",
                                       "TLS_CHACHA20_POLY1305_SHA256"],
                }
    except Exception as e:
        return {"error": str(e)}


def run_audit(host, port=443):
    """Execute TLS 1.3 configuration audit."""
    print(f"\n{'='*60}")
    print(f"  TLS 1.3 CONFIGURATION AUDIT")
    print(f"  Target: {host}:{port}")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    versions = check_tls_versions(host, port)
    print(f"--- TLS VERSION SUPPORT ---")
    for ver, info in versions.items():
        status = "SUPPORTED" if info.get("supported") else "NOT SUPPORTED"
        sev = info.get("severity", "")
        print(f"  {ver}: {status} {f'[{sev}]' if sev else ''}")

    cert = get_certificate_info(host, port)
    print(f"\n--- CERTIFICATE ---")
    if "error" not in cert:
        print(f"  Subject: {cert['subject']}")
        print(f"  Issuer: {cert['issuer']}")
        print(f"  Expires: {cert['not_after']} ({cert['days_remaining']} days)")
        print(f"  Key size: {cert['key_size']}")

    cipher = check_cipher_suites(host, port)
    print(f"\n--- CIPHER SUITE ---")
    if "error" not in cipher:
        print(f"  Negotiated: {cipher['negotiated_cipher']}")
        print(f"  Protocol: {cipher['protocol']}")

    return {"versions": versions, "certificate": cert, "cipher": cipher}


def main():
    parser = argparse.ArgumentParser(description="TLS 1.3 Audit Agent")
    parser.add_argument("--host", required=True, help="Target hostname")
    parser.add_argument("--port", type=int, default=443, help="Target port")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_audit(args.host, args.port)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-tls-1-3-for-secure-communications/scripts/process.py =====
#!/usr/bin/env python3
"""
TLS 1.3 Configuration and Validation Tool

Implements TLS 1.3 server/client testing, configuration generation,
and vulnerability scanning using Python's ssl module and OpenSSL.

Requirements:
    pip install cryptography

Usage:
    python process.py test-server --host example.com --port 443
    python process.py generate-nginx --domain example.com --cert-path /etc/ssl
    python process.py generate-cert --domain example.com --output ./certs
    python process.py check-ciphers --host example.com
"""

import os
import ssl
import sys
import json
import socket
import argparse
import logging
import subprocess
import datetime
from pathlib import Path
from typing import Optional, Dict, List

from cryptography import x509
from cryptography.x509.oid import NameOID, ExtensionOID
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import ec, rsa

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

TLS_13_CIPHERS = [
    "TLS_AES_256_GCM_SHA384",
    "TLS_AES_128_GCM_SHA256",
    "TLS_CHACHA20_POLY1305_SHA256",
]

RECOMMENDED_TLS_12_CIPHERS = [
    "ECDHE-ECDSA-AES256-GCM-SHA384",
    "ECDHE-RSA-AES256-GCM-SHA384",
    "ECDHE-ECDSA-AES128-GCM-SHA256",
    "ECDHE-RSA-AES128-GCM-SHA256",
    "ECDHE-ECDSA-CHACHA20-POLY1305",
    "ECDHE-RSA-CHACHA20-POLY1305",
]


def test_tls_connection(host: str, port: int = 443, timeout: int = 10) -> Dict:
    """Test TLS connection to a server and report protocol details."""
    results = {
        "host": host,
        "port": port,
        "tls_13_supported": False,
        "tls_12_supported": False,
        "protocol_version": None,
        "cipher_suite": None,
        "certificate": None,
        "issues": [],
    }

    # Test TLS 1.3
    try:
        ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
        ctx.minimum_version = ssl.TLSVersion.TLSv1_3
        ctx.maximum_version = ssl.TLSVersion.TLSv1_3

        with socket.create_connection((host, port), timeout=timeout) as sock:
            with ctx.wrap_socket(sock, server_hostname=host) as ssock:
                results["tls_13_supported"] = True
                results["protocol_version"] = ssock.version()
                cipher = ssock.cipher()
                results["cipher_suite"] = {
                    "name": cipher[0],
                    "protocol": cipher[1],
                    "bits": cipher[2],
                }
                cert = ssock.getpeercert()
                results["certificate"] = {
                    "subject": dict(x[0] for x in cert.get("subject", ())),
                    "issuer": dict(x[0] for x in cert.get("issuer", ())),
                    "notBefore": cert.get("notBefore"),
                    "notAfter": cert.get("notAfter"),
                    "serialNumber": cert.get("serialNumber"),
                    "version": cert.get("version"),
                }
                logger.info(f"TLS 1.3 connection successful: {cipher[0]}")
    except ssl.SSLError as e:
        results["issues"].append(f"TLS 1.3 not supported: {e}")
        logger.warning(f"TLS 1.3 not supported on {host}:{port}")
    except Exception as e:
        results["issues"].append(f"Connection error: {e}")

    # Test TLS 1.2
    try:
        ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
        ctx.minimum_version = ssl.TLSVersion.TLSv1_2
        ctx.maximum_version = ssl.TLSVersion.TLSv1_2

        with socket.create_connection((host, port), timeout=timeout) as sock:
            with ctx.wrap_socket(sock, server_hostname=host) as ssock:
                results["tls_12_supported"] = True
                if not results["tls_13_supported"]:
                    results["protocol_version"] = ssock.version()
                    cipher = ssock.cipher()
                    results["cipher_suite"] = {
                        "name": cipher[0],
                        "protocol": cipher[1],
                        "bits": cipher[2],
                    }
    except (ssl.SSLError, Exception):
        pass

    # Test deprecated protocols
    for version_name, version_enum in [
        ("TLS 1.1", ssl.TLSVersion.TLSv1_1),
        ("TLS 1.0", ssl.TLSVersion.TLSv1),
    ]:
        try:
            ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
            ctx.minimum_version = version_enum
            ctx.maximum_version = version_enum
            ctx.check_hostname = False
            ctx.verify_mode = ssl.CERT_NONE
            with socket.create_connection((host, port), timeout=timeout) as sock:
                with ctx.wrap_socket(sock, server_hostname=host) as ssock:
                    results["issues"].append(
                        f"VULNERABLE: {version_name} is still enabled (should be disabled)"
                    )
                    logger.warning(f"{version_name} is enabled on {host}:{port}")
        except (ssl.SSLError, OSError):
            pass

    return results


def check_cipher_suites(host: str, port: int = 443, timeout: int = 10) -> Dict:
    """Check which cipher suites a server supports."""
    results = {"host": host, "port": port, "supported_ciphers": [], "weak_ciphers": []}

    weak_patterns = ["RC4", "DES", "3DES", "MD5", "NULL", "EXPORT", "anon"]

    ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
    ctx.check_hostname = False
    ctx.verify_mode = ssl.CERT_NONE

    try:
        with socket.create_connection((host, port), timeout=timeout) as sock:
            with ctx.wrap_socket(sock, server_hostname=host) as ssock:
                cipher = ssock.cipher()
                results["negotiated_cipher"] = {
                    "name": cipher[0],
                    "protocol": cipher[1],
                    "bits": cipher[2],
                }
                shared = ssock.shared_ciphers()
                if shared:
                    for c in shared:
                        cipher_info = {"name": c[0], "protocol": c[1], "bits": c[2]}
                        results["supported_ciphers"].append(cipher_info)
                        if any(weak in c[0] for weak in weak_patterns):
                            results["weak_ciphers"].append(cipher_info)
    except Exception as e:
        results["error"] = str(e)

    return results


def generate_self_signed_cert(
    domain: str, output_dir: str, key_type: str = "ecdsa"
) -> Dict:
    """Generate a self-signed TLS certificate for testing."""
    output_path = Path(output_dir)
    output_path.mkdir(parents=True, exist_ok=True)

    if key_type == "ecdsa":
        private_key = ec.generate_private_key(ec.SECP256R1())
        key_filename = "server-ecdsa.key"
        cert_filename = "server-ecdsa.crt"
    else:
        private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
        key_filename = "server-rsa.key"
        cert_filename = "server-rsa.crt"

    subject = issuer = x509.Name([
        x509.NameAttribute(NameOID.COUNTRY_NAME, "US"),
        x509.NameAttribute(NameOID.STATE_OR_PROVINCE_NAME, "California"),
        x509.NameAttribute(NameOID.ORGANIZATION_NAME, "Test Organization"),
        x509.NameAttribute(NameOID.COMMON_NAME, domain),
    ])

    cert = (
        x509.CertificateBuilder()
        .subject_name(subject)
        .issuer_name(issuer)
        .public_key(private_key.public_key())
        .serial_number(x509.random_serial_number())
        .not_valid_before(datetime.datetime.utcnow())
        .not_valid_after(datetime.datetime.utcnow() + datetime.timedelta(days=365))
        .add_extension(
            x509.SubjectAlternativeName([
                x509.DNSName(domain),
                x509.DNSName(f"*.{domain}"),
            ]),
            critical=False,
        )
        .add_extension(
            x509.BasicConstraints(ca=False, path_length=None),
            critical=True,
        )
        .sign(private_key, hashes.SHA256())
    )

    key_path = output_path / key_filename
    cert_path = output_path / cert_filename

    key_path.write_bytes(
        private_key.private_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PrivateFormat.TraditionalOpenSSL,
            encryption_algorithm=serialization.NoEncryption(),
        )
    )

    cert_path.write_bytes(cert.public_bytes(serialization.Encoding.PEM))

    logger.info(f"Generated {key_type.upper()} certificate for {domain}")
    logger.info(f"  Key:  {key_path}")
    logger.info(f"  Cert: {cert_path}")

    return {
        "domain": domain,
        "key_type": key_type,
        "key_path": str(key_path),
        "cert_path": str(cert_path),
        "valid_days": 365,
    }


def generate_nginx_config(
    domain: str, cert_path: str, key_path: str, enable_tls12: bool = True
) -> str:
    """Generate nginx TLS 1.3 configuration."""
    tls_protocols = "TLSv1.3"
    if enable_tls12:
        tls_protocols = "TLSv1.2 TLSv1.3"

    tls12_ciphers = ":".join(RECOMMENDED_TLS_12_CIPHERS)

    config = f"""# TLS 1.3 Optimized nginx Configuration
# Generated for: {domain}
# Mozilla SSL Configuration Generator: Modern profile

server {{
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name {domain};

    # Certificate paths
    ssl_certificate {cert_path};
    ssl_certificate_key {key_path};

    # Protocol versions
    ssl_protocols {tls_protocols};

    # TLS 1.2 cipher suites (TLS 1.3 ciphers are configured automatically)
    ssl_ciphers '{tls12_ciphers}';
    ssl_prefer_server_ciphers off;

    # ECDH curve
    ssl_ecdh_curve X25519:secp256r1:secp384r1;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 1.1.1.1 8.8.8.8 valid=300s;
    resolver_timeout 5s;

    # Session configuration
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;

    # Security headers
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-XSS-Protection "0" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Root and index
    root /var/www/{domain}/html;
    index index.html;

    location / {{
        try_files $uri $uri/ =404;
    }}
}}

# HTTP to HTTPS redirect
server {{
    listen 80;
    listen [::]:80;
    server_name {domain};
    return 301 https://$server_name$request_uri;
}}
"""
    return config


def generate_apache_config(
    domain: str, cert_path: str, key_path: str, enable_tls12: bool = True
) -> str:
    """Generate Apache TLS 1.3 configuration."""
    tls_protocols = "TLSv1.3"
    if enable_tls12:
        tls_protocols = "TLSv1.2 TLSv1.3"

    tls12_ciphers = ":".join(RECOMMENDED_TLS_12_CIPHERS)

    config = f"""# TLS 1.3 Optimized Apache Configuration
# Generated for: {domain}

<VirtualHost *:443>
    ServerName {domain}
    DocumentRoot /var/www/{domain}/html

    SSLEngine on
    SSLCertificateFile {cert_path}
    SSLCertificateKeyFile {key_path}

    # Protocol versions
    SSLProtocol all -{" -".join(["SSLv3", "TLSv1", "TLSv1.1"])}
    {"" if enable_tls12 else "SSLProtocol TLSv1.3"}

    # Cipher suites
    SSLCipherSuite {tls12_ciphers}
    SSLHonorCipherOrder off

    # OCSP Stapling
    SSLUseStapling on
    SSLStaplingCache shmcb:/var/run/ocsp(128000)

    # Security headers
    Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "DENY"
</VirtualHost>

# HTTP to HTTPS redirect
<VirtualHost *:80>
    ServerName {domain}
    Redirect permanent / https://{domain}/
</VirtualHost>
"""
    return config


def main():
    parser = argparse.ArgumentParser(description="TLS 1.3 Configuration Tool")
    subparsers = parser.add_subparsers(dest="command")

    # Test server
    test = subparsers.add_parser("test-server", help="Test TLS configuration of a server")
    test.add_argument("--host", required=True, help="Server hostname")
    test.add_argument("--port", type=int, default=443, help="Server port")

    # Check ciphers
    ciphers = subparsers.add_parser("check-ciphers", help="Check supported cipher suites")
    ciphers.add_argument("--host", required=True, help="Server hostname")
    ciphers.add_argument("--port", type=int, default=443, help="Server port")

    # Generate certificate
    cert = subparsers.add_parser("generate-cert", help="Generate self-signed certificate")
    cert.add_argument("--domain", required=True, help="Domain name")
    cert.add_argument("--output", default="./certs", help="Output directory")
    cert.add_argument("--key-type", choices=["ecdsa", "rsa"], default="ecdsa")

    # Generate nginx config
    nginx = subparsers.add_parser("generate-nginx", help="Generate nginx TLS config")
    nginx.add_argument("--domain", required=True, help="Domain name")
    nginx.add_argument("--cert-path", required=True, help="Certificate file path")
    nginx.add_argument("--key-path", required=True, help="Private key file path")
    nginx.add_argument("--tls12", action="store_true", default=True, help="Enable TLS 1.2")

    # Generate Apache config
    apache = subparsers.add_parser("generate-apache", help="Generate Apache TLS config")
    apache.add_argument("--domain", required=True, help="Domain name")
    apache.add_argument("--cert-path", required=True, help="Certificate file path")
    apache.add_argument("--key-path", required=True, help="Private key file path")

    args = parser.parse_args()

    if args.command == "test-server":
        result = test_tls_connection(args.host, args.port)
        print(json.dumps(result, indent=2, default=str))
    elif args.command == "check-ciphers":
        result = check_cipher_suites(args.host, args.port)
        print(json.dumps(result, indent=2))
    elif args.command == "generate-cert":
        result = generate_self_signed_cert(args.domain, args.output, args.key_type)
        print(json.dumps(result, indent=2))
    elif args.command == "generate-nginx":
        config = generate_nginx_config(args.domain, args.cert_path, args.key_path, args.tls12)
        print(config)
    elif args.command == "generate-apache":
        config = generate_apache_config(args.domain, args.cert_path, args.key_path)
        print(config)
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-aes-encryption-for-data-at-rest/SKILL.md =====
---
name: implementing-aes-encryption-for-data-at-rest
description: Guides implementing AES-256 encryption in GCM mode (FIPS 197) for files and data stores at rest, covering key derivation, IV/nonce management, and authenticated encryption. Use when deploying or configuring encryption for data at rest, establishing controls to meet compliance requirements, or reviewing an implementation during a security assessment.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- encryption
- aes
- data-at-rest
- symmetric-encryption
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
- T1486
---
# Implementing AES Encryption for Data at Rest

## Overview

AES (Advanced Encryption Standard) is a symmetric block cipher standardized by NIST (FIPS 197) used to protect classified and sensitive data. This skill covers implementing AES-256 encryption in GCM mode for encrypting files and data stores at rest, including proper key derivation, IV/nonce management, and authenticated encryption.


## When to Use

- When deploying or configuring implementing aes encryption for data at rest capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Implement AES-256-GCM encryption and decryption for files
- Derive encryption keys from passwords using PBKDF2 and Argon2
- Manage initialization vectors (IVs) and nonces securely
- Encrypt and decrypt entire directory trees
- Implement authenticated encryption to detect tampering
- Handle large files with streaming encryption

## Key Concepts

### AES Modes of Operation

| Mode | Authentication | Parallelizable | Use Case |
|------|---------------|----------------|----------|
| GCM  | Yes (AEAD)    | Yes            | Network data, file encryption |
| CBC  | No            | Decrypt only   | Legacy systems, disk encryption |
| CTR  | No            | Yes            | Streaming encryption |
| CCM  | Yes (AEAD)    | No             | IoT, constrained environments |

### Key Derivation

Never use raw passwords as encryption keys. Always derive keys using:
- **PBKDF2**: NIST-approved, widely supported (minimum 600,000 iterations as of 2024)
- **Argon2id**: Winner of Password Hashing Competition, memory-hard
- **scrypt**: Memory-hard, good alternative to Argon2

### Nonce/IV Management

- GCM requires a 96-bit (12-byte) nonce that must NEVER be reused with the same key
- Generate nonces using `os.urandom()` (CSPRNG)
- Store nonce alongside ciphertext (it is not secret)

## Workflow

1. Install the `cryptography` library: `pip install cryptography`
2. Generate or derive an encryption key
3. Create a random nonce for each encryption operation
4. Encrypt data using AES-256-GCM with the key and nonce
5. Store nonce + ciphertext + authentication tag together
6. For decryption, extract nonce, verify tag, and decrypt

## Encrypted File Format

```
[salt: 16 bytes][nonce: 12 bytes][ciphertext: variable][tag: 16 bytes]
```

## Security Considerations

- Always use authenticated encryption (GCM, CCM) to prevent tampering
- Never reuse a nonce with the same key (catastrophic in GCM)
- Use at least 256-bit keys for long-term data protection
- Securely wipe keys from memory after use when possible
- Rotate encryption keys periodically per organizational policy
- For disk-level encryption, consider XTS mode (AES-XTS)

## Validation Criteria

- [ ] AES-256-GCM encryption produces valid ciphertext
- [ ] Decryption recovers original plaintext exactly
- [ ] Authentication tag detects any ciphertext modification
- [ ] Key derivation uses sufficient iterations/parameters
- [ ] Nonces are never reused for the same key
- [ ] Large files (>1GB) can be processed via streaming
- [ ] Encrypted file format includes all necessary metadata

===== END FILE =====


===== FILE: skills/implementing-aes-encryption-for-data-at-rest/LICENSE =====

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


===== FILE: skills/implementing-aes-encryption-for-data-at-rest/assets/template.md =====
# AES Encryption Implementation Template

## Pre-Implementation Checklist

- [ ] Identify data classification level and regulatory requirements
- [ ] Determine key management strategy (local, HSM, KMS)
- [ ] Select AES mode (GCM recommended for authenticated encryption)
- [ ] Define key derivation parameters (algorithm, iterations)
- [ ] Plan nonce/IV generation strategy
- [ ] Determine encrypted file format and metadata storage
- [ ] Review compliance requirements (PCI-DSS, HIPAA, GDPR)

## Configuration Parameters

```yaml
encryption:
  algorithm: AES-256-GCM
  key_length: 256
  nonce_length: 96  # bits
  tag_length: 128   # bits

key_derivation:
  algorithm: PBKDF2-SHA256
  iterations: 600000
  salt_length: 128  # bits

file_format:
  magic_bytes: "AES256GCM"
  version: 1
  header: "magic || version || salt || nonce"
  body: "ciphertext || tag"
```

## Integration Code Template

```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes
import os

def encrypt_data(plaintext: bytes, password: str) -> bytes:
    """Encrypt data with AES-256-GCM."""
    salt = os.urandom(16)
    kdf = PBKDF2HMAC(
        algorithm=hashes.SHA256(),
        length=32,
        salt=salt,
        iterations=600_000,
    )
    key = kdf.derive(password.encode())
    nonce = os.urandom(12)
    aesgcm = AESGCM(key)
    ciphertext = aesgcm.encrypt(nonce, plaintext, None)
    return salt + nonce + ciphertext

def decrypt_data(data: bytes, password: str) -> bytes:
    """Decrypt AES-256-GCM encrypted data."""
    salt = data[:16]
    nonce = data[16:28]
    ciphertext = data[28:]
    kdf = PBKDF2HMAC(
        algorithm=hashes.SHA256(),
        length=32,
        salt=salt,
        iterations=600_000,
    )
    key = kdf.derive(password.encode())
    aesgcm = AESGCM(key)
    return aesgcm.decrypt(nonce, ciphertext, None)
```

## Testing Checklist

- [ ] Encrypt and decrypt a small text file
- [ ] Encrypt and decrypt a large binary file (>100MB)
- [ ] Verify wrong password raises authentication error
- [ ] Verify tampered ciphertext raises authentication error
- [ ] Verify nonce uniqueness across multiple encryptions
- [ ] Measure encryption throughput (MB/s)
- [ ] Test with empty files and edge cases

## Common Pitfalls

| Pitfall | Impact | Mitigation |
|---------|--------|------------|
| Nonce reuse with same key | Complete loss of confidentiality in GCM | Always generate random nonce per encryption |
| Low PBKDF2 iterations | Brute-force password attacks | Use minimum 600,000 iterations |
| ECB mode usage | Pattern leakage in ciphertext | Always use GCM or CBC (never ECB) |
| No authentication | Undetected ciphertext modification | Use AEAD modes (GCM, CCM) |
| Hardcoded keys | Key compromise | Use KMS, HSM, or environment variables |
| No key rotation | Extended exposure window | Implement periodic key rotation policy |

===== END FILE =====


===== FILE: skills/implementing-aes-encryption-for-data-at-rest/references/api-reference.md =====
# API Reference: Implementing AES Encryption for Data at Rest

## cryptography Library - AESGCM

```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

key = AESGCM.generate_key(bit_length=256)
aesgcm = AESGCM(key)
nonce = os.urandom(12)  # 96-bit nonce, NEVER reuse

ciphertext = aesgcm.encrypt(nonce, plaintext, associated_data)
plaintext = aesgcm.decrypt(nonce, ciphertext, associated_data)
```

## Key Derivation - PBKDF2

```python
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes

kdf = PBKDF2HMAC(
    algorithm=hashes.SHA256(),
    length=32,               # 256-bit key
    salt=os.urandom(16),
    iterations=600_000,      # NIST 2024 recommendation
)
key = kdf.derive(password.encode())
```

## Encrypted File Format

```
[salt: 16 bytes][nonce: 12 bytes][ciphertext + tag: variable]
```

| Field | Size | Purpose |
|-------|------|---------|
| Salt | 16 bytes | PBKDF2 salt (random per file) |
| Nonce | 12 bytes | GCM nonce (random per encryption) |
| Ciphertext | Variable | Encrypted data + 16-byte auth tag |

## AES Modes Comparison

| Mode | AEAD | Nonce Size | Use Case |
|------|------|------------|----------|
| GCM | Yes | 12 bytes | File/network encryption |
| CBC | No | 16 bytes | Legacy, disk encryption |
| CTR | No | 16 bytes | Streaming |
| XTS | No | 16 bytes | Full disk encryption |

## Fernet (High-Level API)

```python
from cryptography.fernet import Fernet
key = Fernet.generate_key()
f = Fernet(key)
token = f.encrypt(b"data")
plaintext = f.decrypt(token)
```

### References

- cryptography AESGCM: https://cryptography.io/en/latest/hazmat/primitives/aead/
- NIST SP 800-38D (GCM): https://csrc.nist.gov/publications/detail/sp/800-38d/final
- NIST FIPS 197 (AES): https://csrc.nist.gov/publications/detail/fips/197/final

===== END FILE =====


===== FILE: skills/implementing-aes-encryption-for-data-at-rest/references/standards.md =====
# Standards and References - AES Encryption for Data at Rest

## Primary Standards

### NIST FIPS 197 - Advanced Encryption Standard (AES)
- **URL**: https://csrc.nist.gov/publications/detail/fips/197/final
- **Description**: Defines the AES algorithm (Rijndael) with key sizes of 128, 192, and 256 bits
- **Block size**: 128 bits (16 bytes)
- **Key sizes**: 128, 192, or 256 bits
- **Rounds**: 10 (128-bit), 12 (192-bit), 14 (256-bit)

### NIST SP 800-38D - Recommendation for Block Cipher Modes: GCM and GMAC
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-38d/final
- **Description**: Specifies Galois/Counter Mode (GCM) for authenticated encryption
- **IV length**: 96 bits recommended for GCM
- **Tag length**: 128 bits recommended (minimum 96 bits)
- **Max plaintext**: 2^39 - 256 bits per invocation

### NIST SP 800-132 - Recommendation for Password-Based Key Derivation
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-132/final
- **Description**: Covers PBKDF2 for deriving cryptographic keys from passwords
- **Minimum iterations**: 600,000 (OWASP 2024 recommendation for PBKDF2-SHA256)
- **Salt length**: Minimum 128 bits (16 bytes)

### NIST SP 800-38A - Recommendation for Block Cipher Modes of Operation
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-38a/final
- **Description**: Defines ECB, CBC, CFB, OFB, and CTR modes

### NIST SP 800-57 Part 1 Rev. 5 - Key Management
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final
- **Description**: Recommendations for cryptographic key lengths and algorithms
- **AES-256 security strength**: 256 bits
- **Recommended until**: Beyond 2031

## RFC Standards

### RFC 5116 - An Interface and Algorithms for Authenticated Encryption
- **URL**: https://www.rfc-editor.org/rfc/rfc5116
- **Description**: Defines AEAD interface including AES-GCM

### RFC 5869 - HMAC-based Extract-and-Expand Key Derivation Function (HKDF)
- **URL**: https://www.rfc-editor.org/rfc/rfc5869
- **Description**: Key derivation from existing key material (not passwords)

### RFC 9106 - Argon2 Memory-Hard Function
- **URL**: https://www.rfc-editor.org/rfc/rfc9106
- **Description**: Argon2 password hashing / key derivation specification
- **Recommended variant**: Argon2id (hybrid of Argon2i and Argon2d)

## Compliance Frameworks

### PCI DSS v4.0 - Requirement 3
- Encrypt stored cardholder data with strong cryptography
- AES-256 meets the strong cryptography requirement
- Key management procedures required

### HIPAA Security Rule - 45 CFR 164.312(a)(2)(iv)
- Encryption of ePHI at rest is an addressable implementation specification
- AES-256 is an acceptable encryption method

### GDPR Article 32 - Security of Processing
- Encryption is listed as an appropriate technical measure
- AES-256 satisfies encryption requirements for personal data protection

## Python Library References

### cryptography (pyca/cryptography)
- **URL**: https://cryptography.io/en/latest/
- **PyPI**: https://pypi.org/project/cryptography/
- **AES-GCM**: `cryptography.hazmat.primitives.ciphers.aead.AESGCM`
- **PBKDF2**: `cryptography.hazmat.primitives.kdf.pbkdf2.PBKDF2HMAC`

### PyCryptodome
- **URL**: https://pycryptodome.readthedocs.io/
- **PyPI**: https://pypi.org/project/pycryptodome/
- **AES-GCM**: `Crypto.Cipher.AES` with `MODE_GCM`

===== END FILE =====


===== FILE: skills/implementing-aes-encryption-for-data-at-rest/references/workflows.md =====
# Workflows - AES Encryption for Data at Rest

## Workflow 1: Single File Encryption

```
[Input File] --> [Read File Bytes]
                      |
              [Derive Key from Password]
              (PBKDF2 / Argon2id + random salt)
                      |
              [Generate Random Nonce]
              (12 bytes from CSPRNG)
                      |
              [AES-256-GCM Encrypt]
              (key + nonce + plaintext --> ciphertext + tag)
                      |
              [Write Encrypted File]
              (salt || nonce || ciphertext || tag)
```

## Workflow 2: Single File Decryption

```
[Encrypted File] --> [Parse Header]
                     (extract salt, nonce)
                          |
                  [Derive Key from Password]
                  (same PBKDF2 / Argon2id params + extracted salt)
                          |
                  [AES-256-GCM Decrypt]
                  (key + nonce + ciphertext + tag)
                          |
                  [Verify Authentication Tag]
                  (reject if tag invalid)
                          |
                  [Write Decrypted File]
```

## Workflow 3: Streaming Encryption for Large Files

```
[Large Input File]
      |
[Read in Chunks] (e.g., 64KB chunks)
      |
[For Each Chunk]:
  - [Encrypt chunk with AES-256-CTR]
  - [Update HMAC with ciphertext chunk]
  - [Write encrypted chunk to output]
      |
[Finalize HMAC]
[Append HMAC tag to output]
```

## Workflow 4: Directory Tree Encryption

```
[Source Directory]
      |
[Walk Directory Tree]
      |
[For Each File]:
  - [Derive unique file key from master key + file path]
  - [Generate random nonce]
  - [AES-256-GCM encrypt file]
  - [Write encrypted file preserving directory structure]
      |
[Create Manifest File]
(maps original paths to encrypted paths with metadata)
```

## Workflow 5: Key Derivation Pipeline

```
[User Password]
      |
[Generate Random Salt] (16 bytes)
      |
[PBKDF2-SHA256]
  - iterations: 600,000+
  - dkLen: 32 bytes (256 bits)
      |
[Derived Key (256-bit)]
      |
[Optional: HKDF Expand]
  - Derive multiple subkeys from single derived key
  - info="encryption" --> encryption key
  - info="authentication" --> HMAC key
```

## Workflow 6: Envelope Encryption Pattern

```
[Master Key] (stored in HSM/KMS)
      |
[Generate Random Data Encryption Key (DEK)]
(32 bytes from CSPRNG)
      |
[Encrypt DEK with Master Key] --> [Encrypted DEK]
      |
[Encrypt Data with DEK] --> [Ciphertext]
      |
[Store: Encrypted DEK + Ciphertext]
[Securely Wipe DEK from Memory]
```

## Error Handling Workflow

```
[Decryption Attempt]
      |
  [Parse Header] --FAIL--> [Return: Corrupt/invalid file format]
      |
  [Derive Key] --FAIL--> [Return: KDF parameter error]
      |
  [Decrypt + Verify Tag]
      |
  [Tag Valid?]
    YES --> [Return plaintext]
    NO  --> [Return: Authentication failed - data tampered]
            [DO NOT return partial plaintext]
```

===== END FILE =====


===== FILE: skills/implementing-aes-encryption-for-data-at-rest/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing AES-256-GCM encryption for data at rest."""

import os
import json
import argparse
from datetime import datetime
from pathlib import Path

from cryptography.hazmat.primitives.ciphers.aead import AESGCM
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes


SALT_SIZE = 16
NONCE_SIZE = 12
KEY_SIZE = 32  # 256 bits
TAG_SIZE = 16
PBKDF2_ITERATIONS = 600_000


def derive_key(password, salt=None):
    """Derive AES-256 key from password using PBKDF2-HMAC-SHA256."""
    if salt is None:
        salt = os.urandom(SALT_SIZE)
    kdf = PBKDF2HMAC(
        algorithm=hashes.SHA256(),
        length=KEY_SIZE,
        salt=salt,
        iterations=PBKDF2_ITERATIONS,
    )
    key = kdf.derive(password.encode("utf-8"))
    return key, salt


def encrypt_file(input_path, output_path, password):
    """Encrypt a file using AES-256-GCM with PBKDF2 key derivation."""
    key, salt = derive_key(password)
    nonce = os.urandom(NONCE_SIZE)
    aesgcm = AESGCM(key)

    with open(input_path, "rb") as f:
        plaintext = f.read()

    ciphertext = aesgcm.encrypt(nonce, plaintext, None)

    with open(output_path, "wb") as f:
        f.write(salt)
        f.write(nonce)
        f.write(ciphertext)

    return {
        "input": str(input_path),
        "output": str(output_path),
        "original_size": len(plaintext),
        "encrypted_size": SALT_SIZE + NONCE_SIZE + len(ciphertext),
        "algorithm": "AES-256-GCM",
        "kdf": f"PBKDF2-HMAC-SHA256 ({PBKDF2_ITERATIONS} iterations)",
    }


def decrypt_file(input_path, output_path, password):
    """Decrypt an AES-256-GCM encrypted file."""
    with open(input_path, "rb") as f:
        salt = f.read(SALT_SIZE)
        nonce = f.read(NONCE_SIZE)
        ciphertext = f.read()

    key, _ = derive_key(password, salt)
    aesgcm = AESGCM(key)
    plaintext = aesgcm.decrypt(nonce, ciphertext, None)

    with open(output_path, "wb") as f:
        f.write(plaintext)

    return {
        "input": str(input_path),
        "output": str(output_path),
        "decrypted_size": len(plaintext),
    }


def encrypt_directory(dir_path, output_dir, password):
    """Encrypt all files in a directory tree."""
    src = Path(dir_path)
    dst = Path(output_dir)
    dst.mkdir(parents=True, exist_ok=True)
    results = []
    for filepath in src.rglob("*"):
        if filepath.is_file():
            rel = filepath.relative_to(src)
            out = dst / (str(rel) + ".enc")
            out.parent.mkdir(parents=True, exist_ok=True)
            result = encrypt_file(str(filepath), str(out), password)
            results.append(result)
    return results


def generate_random_key():
    """Generate a random AES-256 key."""
    key = os.urandom(KEY_SIZE)
    return {
        "key_hex": key.hex(),
        "key_size_bits": KEY_SIZE * 8,
        "algorithm": "AES-256",
    }


def verify_encryption(original_path, encrypted_path, password):
    """Verify encryption by decrypting and comparing."""
    with open(original_path, "rb") as f:
        original = f.read()

    with open(encrypted_path, "rb") as f:
        salt = f.read(SALT_SIZE)
        nonce = f.read(NONCE_SIZE)
        ciphertext = f.read()

    key, _ = derive_key(password, salt)
    aesgcm = AESGCM(key)
    try:
        decrypted = aesgcm.decrypt(nonce, ciphertext, None)
        match = original == decrypted
        return {"status": "PASS" if match else "FAIL", "content_match": match}
    except Exception as e:
        return {"status": "FAIL", "error": str(e)}


def main():
    parser = argparse.ArgumentParser(description="AES-256-GCM Encryption Agent")
    parser.add_argument("--action", required=True,
                        choices=["encrypt", "decrypt", "encrypt_dir", "genkey", "verify"])
    parser.add_argument("--input", help="Input file or directory")
    parser.add_argument("--output", help="Output file or directory")
    parser.add_argument("--password", help="Encryption password")
    parser.add_argument("--report", default="aes_encryption_report.json")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "action": args.action}

    if args.action == "encrypt":
        result = encrypt_file(args.input, args.output, args.password)
        report["result"] = result
        print(f"[+] Encrypted: {args.input} -> {args.output}")

    elif args.action == "decrypt":
        result = decrypt_file(args.input, args.output, args.password)
        report["result"] = result
        print(f"[+] Decrypted: {args.input} -> {args.output}")

    elif args.action == "encrypt_dir":
        results = encrypt_directory(args.input, args.output, args.password)
        report["results"] = results
        print(f"[+] Encrypted {len(results)} files")

    elif args.action == "genkey":
        result = generate_random_key()
        report["result"] = result
        print(f"[+] Key: {result['key_hex']}")

    elif args.action == "verify":
        result = verify_encryption(args.input, args.output, args.password)
        report["result"] = result
        print(f"[+] Verification: {result['status']}")

    with open(args.report, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.report}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-aes-encryption-for-data-at-rest/scripts/process.py =====
#!/usr/bin/env python3
"""
AES-256-GCM Encryption for Data at Rest

Implements file and directory encryption using AES-256-GCM with
PBKDF2 key derivation. Supports single file, streaming large file,
and directory tree encryption.

Requirements:
    pip install cryptography

Usage:
    python process.py encrypt --input secret.pdf --output secret.pdf.enc --password "MySecurePass"
    python process.py decrypt --input secret.pdf.enc --output secret.pdf --password "MySecurePass"
    python process.py encrypt-dir --input ./sensitive/ --output ./encrypted/ --password "MySecurePass"
"""

import os
import sys
import json
import struct
import hashlib
import argparse
import logging
from pathlib import Path
from typing import Optional, Tuple

from cryptography.hazmat.primitives.ciphers.aead import AESGCM
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.backends import default_backend

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

# Constants
SALT_LENGTH = 16          # 128-bit salt
NONCE_LENGTH = 12         # 96-bit nonce (recommended for GCM)
TAG_LENGTH = 16           # 128-bit authentication tag
KEY_LENGTH = 32           # 256-bit key
PBKDF2_ITERATIONS = 600_000  # OWASP 2024 recommendation
CHUNK_SIZE = 64 * 1024    # 64KB chunks for streaming
MAGIC_BYTES = b"AES256GCM"  # File format identifier
VERSION = 1


def derive_key(password: str, salt: bytes, iterations: int = PBKDF2_ITERATIONS) -> bytes:
    """Derive a 256-bit encryption key from a password using PBKDF2-SHA256."""
    kdf = PBKDF2HMAC(
        algorithm=hashes.SHA256(),
        length=KEY_LENGTH,
        salt=salt,
        iterations=iterations,
        backend=default_backend(),
    )
    return kdf.derive(password.encode("utf-8"))


def encrypt_bytes(plaintext: bytes, password: str) -> bytes:
    """
    Encrypt plaintext bytes using AES-256-GCM with PBKDF2 key derivation.

    Output format:
        MAGIC (9 bytes) || VERSION (1 byte) || SALT (16 bytes) || NONCE (12 bytes) || CIPHERTEXT+TAG (variable)

    The authentication tag is appended to ciphertext by AESGCM.
    """
    salt = os.urandom(SALT_LENGTH)
    nonce = os.urandom(NONCE_LENGTH)
    key = derive_key(password, salt)

    aesgcm = AESGCM(key)
    ciphertext = aesgcm.encrypt(nonce, plaintext, associated_data=None)

    header = MAGIC_BYTES + struct.pack("B", VERSION)
    return header + salt + nonce + ciphertext


def decrypt_bytes(data: bytes, password: str) -> bytes:
    """
    Decrypt AES-256-GCM encrypted data.

    Raises:
        ValueError: If file format is invalid or authentication fails.
    """
    magic_len = len(MAGIC_BYTES)
    min_length = magic_len + 1 + SALT_LENGTH + NONCE_LENGTH + TAG_LENGTH

    if len(data) < min_length:
        raise ValueError("Data too short to be a valid encrypted file")

    magic = data[:magic_len]
    if magic != MAGIC_BYTES:
        raise ValueError(f"Invalid file format: expected magic bytes {MAGIC_BYTES!r}, got {magic!r}")

    version = struct.unpack("B", data[magic_len : magic_len + 1])[0]
    if version != VERSION:
        raise ValueError(f"Unsupported version: {version}")

    offset = magic_len + 1
    salt = data[offset : offset + SALT_LENGTH]
    offset += SALT_LENGTH
    nonce = data[offset : offset + NONCE_LENGTH]
    offset += NONCE_LENGTH
    ciphertext = data[offset:]

    key = derive_key(password, salt)
    aesgcm = AESGCM(key)

    try:
        plaintext = aesgcm.decrypt(nonce, ciphertext, associated_data=None)
    except Exception as e:
        raise ValueError(
            "Decryption failed: authentication tag verification failed. "
            "Either the password is wrong or the data has been tampered with."
        ) from e

    return plaintext


def encrypt_file(input_path: str, output_path: str, password: str) -> dict:
    """Encrypt a single file."""
    input_file = Path(input_path)
    if not input_file.exists():
        raise FileNotFoundError(f"Input file not found: {input_path}")

    plaintext = input_file.read_bytes()
    original_size = len(plaintext)
    original_hash = hashlib.sha256(plaintext).hexdigest()

    ciphertext = encrypt_bytes(plaintext, password)

    output_file = Path(output_path)
    output_file.parent.mkdir(parents=True, exist_ok=True)
    output_file.write_bytes(ciphertext)

    encrypted_size = len(ciphertext)
    logger.info(f"Encrypted {input_path} -> {output_path} ({original_size} -> {encrypted_size} bytes)")

    return {
        "input": str(input_path),
        "output": str(output_path),
        "original_size": original_size,
        "encrypted_size": encrypted_size,
        "original_sha256": original_hash,
        "algorithm": "AES-256-GCM",
        "kdf": "PBKDF2-SHA256",
        "kdf_iterations": PBKDF2_ITERATIONS,
    }


def decrypt_file(input_path: str, output_path: str, password: str) -> dict:
    """Decrypt a single file."""
    input_file = Path(input_path)
    if not input_file.exists():
        raise FileNotFoundError(f"Input file not found: {input_path}")

    data = input_file.read_bytes()
    plaintext = decrypt_bytes(data, password)

    output_file = Path(output_path)
    output_file.parent.mkdir(parents=True, exist_ok=True)
    output_file.write_bytes(plaintext)

    recovered_hash = hashlib.sha256(plaintext).hexdigest()
    logger.info(f"Decrypted {input_path} -> {output_path} ({len(plaintext)} bytes)")

    return {
        "input": str(input_path),
        "output": str(output_path),
        "decrypted_size": len(plaintext),
        "recovered_sha256": recovered_hash,
    }


def encrypt_directory(input_dir: str, output_dir: str, password: str) -> dict:
    """Encrypt all files in a directory tree, preserving structure."""
    input_path = Path(input_dir)
    output_path = Path(output_dir)

    if not input_path.is_dir():
        raise NotADirectoryError(f"Input is not a directory: {input_dir}")

    output_path.mkdir(parents=True, exist_ok=True)

    manifest = {
        "algorithm": "AES-256-GCM",
        "kdf": "PBKDF2-SHA256",
        "kdf_iterations": PBKDF2_ITERATIONS,
        "files": [],
    }

    file_count = 0
    total_original = 0
    total_encrypted = 0

    for file in sorted(input_path.rglob("*")):
        if file.is_file():
            relative = file.relative_to(input_path)
            encrypted_name = str(relative) + ".enc"
            dest = output_path / encrypted_name

            result = encrypt_file(str(file), str(dest), password)
            manifest["files"].append({
                "original_path": str(relative),
                "encrypted_path": encrypted_name,
                "original_sha256": result["original_sha256"],
                "original_size": result["original_size"],
            })

            file_count += 1
            total_original += result["original_size"]
            total_encrypted += result["encrypted_size"]

    manifest_path = output_path / "manifest.json"
    manifest_encrypted = encrypt_bytes(json.dumps(manifest, indent=2).encode(), password)
    (output_path / "manifest.json.enc").write_bytes(manifest_encrypted)

    logger.info(
        f"Encrypted {file_count} files from {input_dir} -> {output_dir} "
        f"({total_original} -> {total_encrypted} bytes)"
    )

    return {
        "files_encrypted": file_count,
        "total_original_bytes": total_original,
        "total_encrypted_bytes": total_encrypted,
        "output_directory": str(output_path),
    }


def decrypt_directory(input_dir: str, output_dir: str, password: str) -> dict:
    """Decrypt all .enc files in a directory tree."""
    input_path = Path(input_dir)
    output_path = Path(output_dir)

    if not input_path.is_dir():
        raise NotADirectoryError(f"Input is not a directory: {input_dir}")

    output_path.mkdir(parents=True, exist_ok=True)

    file_count = 0
    total_decrypted = 0

    for file in sorted(input_path.rglob("*.enc")):
        if file.name == "manifest.json.enc":
            continue
        if file.is_file():
            relative = file.relative_to(input_path)
            decrypted_name = str(relative).removesuffix(".enc")
            dest = output_path / decrypted_name

            result = decrypt_file(str(file), str(dest), password)
            file_count += 1
            total_decrypted += result["decrypted_size"]

    logger.info(f"Decrypted {file_count} files from {input_dir} -> {output_dir}")

    return {
        "files_decrypted": file_count,
        "total_decrypted_bytes": total_decrypted,
        "output_directory": str(output_path),
    }


def verify_roundtrip(test_data: bytes, password: str) -> bool:
    """Verify encryption/decryption roundtrip integrity."""
    encrypted = encrypt_bytes(test_data, password)
    decrypted = decrypt_bytes(encrypted, password)
    return decrypted == test_data


def main():
    parser = argparse.ArgumentParser(description="AES-256-GCM File Encryption Tool")
    subparsers = parser.add_subparsers(dest="command", help="Command to execute")

    # Encrypt command
    enc = subparsers.add_parser("encrypt", help="Encrypt a file")
    enc.add_argument("--input", "-i", required=True, help="Input file path")
    enc.add_argument("--output", "-o", required=True, help="Output file path")
    enc.add_argument("--password", "-p", required=True, help="Encryption password")

    # Decrypt command
    dec = subparsers.add_parser("decrypt", help="Decrypt a file")
    dec.add_argument("--input", "-i", required=True, help="Input file path")
    dec.add_argument("--output", "-o", required=True, help="Output file path")
    dec.add_argument("--password", "-p", required=True, help="Decryption password")

    # Encrypt directory command
    encdir = subparsers.add_parser("encrypt-dir", help="Encrypt a directory")
    encdir.add_argument("--input", "-i", required=True, help="Input directory path")
    encdir.add_argument("--output", "-o", required=True, help="Output directory path")
    encdir.add_argument("--password", "-p", required=True, help="Encryption password")

    # Decrypt directory command
    decdir = subparsers.add_parser("decrypt-dir", help="Decrypt a directory")
    decdir.add_argument("--input", "-i", required=True, help="Input directory path")
    decdir.add_argument("--output", "-o", required=True, help="Output directory path")
    decdir.add_argument("--password", "-p", required=True, help="Decryption password")

    # Verify command
    subparsers.add_parser("verify", help="Run roundtrip verification test")

    args = parser.parse_args()

    if args.command == "encrypt":
        result = encrypt_file(args.input, args.output, args.password)
        print(json.dumps(result, indent=2))
    elif args.command == "decrypt":
        result = decrypt_file(args.input, args.output, args.password)
        print(json.dumps(result, indent=2))
    elif args.command == "encrypt-dir":
        result = encrypt_directory(args.input, args.output, args.password)
        print(json.dumps(result, indent=2))
    elif args.command == "decrypt-dir":
        result = decrypt_directory(args.input, args.output, args.password)
        print(json.dumps(result, indent=2))
    elif args.command == "verify":
        test_data = b"The quick brown fox jumps over the lazy dog. " * 100
        password = "test_password_123!"
        success = verify_roundtrip(test_data, password)
        print(f"Roundtrip verification: {'PASSED' if success else 'FAILED'}")
        if not success:
            sys.exit(1)
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-digital-signatures-with-ed25519/SKILL.md =====
---
name: implementing-digital-signatures-with-ed25519
description: Implements digital signatures using the Ed25519 algorithm (Curve25519), covering key-pair generation, signing, signature verification, and security tradeoffs versus RSA and ECDSA. Use when adding message or artifact signing and authentication-integrity checks to a system, or when advising on Ed25519 key properties and validation criteria.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- digital-signatures
- ed25519
- authentication
- integrity
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
---
# Implementing Digital Signatures with Ed25519

## Overview

Ed25519 is a high-performance digital signature algorithm using the Edwards curve Curve25519. It provides 128-bit security with 64-byte signatures and 32-byte keys, offering significant advantages over RSA and ECDSA including deterministic signatures (no random nonce needed), resistance to side-channel attacks, and fast verification. This skill covers implementing Ed25519 for document signing, code signing, and API authentication.


## When to Use

- When deploying or configuring implementing digital signatures with ed25519 capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Generate Ed25519 key pairs for signing
- Sign messages and files with Ed25519
- Verify signatures against public keys
- Implement multi-signature verification
- Build a simple code signing system
- Compare Ed25519 performance with RSA and ECDSA

## Key Concepts

### Ed25519 vs RSA vs ECDSA

| Property | Ed25519 | RSA-3072 | ECDSA P-256 |
|----------|---------|----------|-------------|
| Security | 128-bit | 128-bit | 128-bit |
| Public key size | 32 bytes | 384 bytes | 64 bytes |
| Signature size | 64 bytes | 384 bytes | 64 bytes |
| Key generation | ~50 us | ~100 ms | ~1 ms |
| Sign | ~70 us | ~5 ms | ~200 us |
| Verify | ~200 us | ~200 us | ~500 us |
| Deterministic | Yes | No (PSS) | No (unless RFC 6979) |

### Key Properties

- **Deterministic**: Same message + key always produces same signature
- **Collision-resistant**: No separate hash function needed
- **Side-channel resistant**: Constant-time implementation
- **Small keys**: 32 bytes each (public and private)

## Security Considerations

- Ed25519 does not support key recovery from signatures
- Verify the full message, not a hash (Ed25519 hashes internally)
- Public keys must be validated before use (check for low-order points)
- Private keys should be stored encrypted at rest
- Ed25519 is not yet approved for all NIST use cases (Ed448 is preferred for federal)

## Validation Criteria

- [ ] Key pair generation produces valid Ed25519 keys
- [ ] Signature verification succeeds for valid message
- [ ] Signature verification fails for tampered message
- [ ] Signature verification fails for wrong public key
- [ ] Deterministic: same input produces same signature
- [ ] File signing and verification works correctly
- [ ] Performance meets or exceeds RSA-3072

===== END FILE =====


===== FILE: skills/implementing-digital-signatures-with-ed25519/LICENSE =====

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


===== FILE: skills/implementing-digital-signatures-with-ed25519/assets/template.md =====
# Ed25519 Digital Signatures Template

## Quick Reference

```python
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey

# Generate
private_key = Ed25519PrivateKey.generate()
public_key = private_key.public_key()

# Sign
signature = private_key.sign(b"message data")

# Verify
public_key.verify(signature, b"message data")  # raises InvalidSignature on failure
```

## Key Formats

| Format | Private Key Size | Public Key Size | Signature Size |
|--------|-----------------|-----------------|----------------|
| Raw | 32 bytes | 32 bytes | 64 bytes |
| PEM (PKCS#8) | ~119 bytes | ~90 bytes | N/A |
| SSH | ~83 bytes | ~51 bytes | ~83 bytes |

## Use Cases

- API request authentication (sign request body)
- Software/code signing
- Document signing
- Git commit signing (ssh-ed25519)
- JWT signing (EdDSA algorithm)
- Certificate signing (X.509 with Ed25519)

===== END FILE =====


===== FILE: skills/implementing-digital-signatures-with-ed25519/references/api-reference.md =====
# API Reference: Ed25519 Digital Signature Agent

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| cryptography | >=41.0 | Ed25519 key generation, signing, verification |

## CLI Usage

```bash
# Generate keypair
python scripts/agent.py --generate-keys --output-dir /keys/

# Sign a file
python scripts/agent.py --sign release.tar.gz --private-key /keys/ed25519_private.pem

# Verify files
python scripts/agent.py --verify release.tar.gz --public-key /keys/ed25519_public.pem
```

## Functions

### `generate_keypair(output_dir, key_name) -> dict`
`Ed25519PrivateKey.generate()`, serializes with `private_bytes(PEM, PKCS8, NoEncryption)` and `public_bytes(PEM, SubjectPublicKeyInfo)`.

### `sign_message(private_key_path, message) -> dict`
Loads key via `load_pem_private_key()`, calls `key.sign(message)`. Returns base64 and hex signature.

### `sign_file(private_key_path, file_path) -> dict`
Signs file contents, writes `.ed25519.sig` JSON containing signature, hash, timestamp.

### `verify_message(public_key_path, message, signature_b64) -> dict`
Calls `key.verify(signature, message)`. Catches `InvalidSignature`.

### `verify_file(public_key_path, file_path, sig_path) -> dict`
Verifies file against `.ed25519.sig` JSON, checks hash match.

## cryptography API

| Method | Purpose |
|--------|---------|
| `Ed25519PrivateKey.generate()` | Generate 32-byte private key |
| `private_key.sign(data)` | Create 64-byte signature |
| `public_key.verify(signature, data)` | Verify signature |
| `load_pem_private_key(data, password)` | Load PEM key |

## Output Schema

```json
{
  "verifications": [{"file": "release.tar.gz", "valid": true}],
  "valid": 3, "invalid": 0
}
```

===== END FILE =====


===== FILE: skills/implementing-digital-signatures-with-ed25519/references/standards.md =====
# Standards and References - Digital Signatures with Ed25519

## Primary Standards

### RFC 8032 - Edwards-Curve Digital Signature Algorithm (EdDSA)
- **URL**: https://www.rfc-editor.org/rfc/rfc8032
- **Description**: Defines Ed25519 and Ed448 signature algorithms

### RFC 8709 - Ed25519 and Ed448 Public Key Algorithms for SSH
- **URL**: https://www.rfc-editor.org/rfc/rfc8709
- **Description**: SSH key format for Ed25519

### NIST FIPS 186-5 - Digital Signature Standard
- **URL**: https://csrc.nist.gov/publications/detail/fips/186/5/final
- **Description**: Includes EdDSA as approved signature algorithm

### RFC 7748 - Elliptic Curves for Security
- **URL**: https://www.rfc-editor.org/rfc/rfc7748
- **Description**: Defines Curve25519 and Curve448

## Python Libraries

### cryptography (pyca/cryptography)
- **Ed25519**: `cryptography.hazmat.primitives.asymmetric.ed25519`
- **Docs**: https://cryptography.io/en/latest/hazmat/primitives/asymmetric/ed25519/

### PyNaCl (libsodium)
- **URL**: https://pynacl.readthedocs.io/
- **Ed25519**: `nacl.signing`
- **Docs**: https://pynacl.readthedocs.io/en/latest/signing/

## Related

### Daniel J. Bernstein et al. - High-speed high-security signatures
- **URL**: https://ed25519.cr.yp.to/
- **Description**: Original Ed25519 paper and reference implementation

===== END FILE =====


===== FILE: skills/implementing-digital-signatures-with-ed25519/references/workflows.md =====
# Workflows - Digital Signatures with Ed25519

## Workflow 1: Key Generation and Storage

```
[Generate Ed25519 Key Pair]
(32-byte private seed -> 32-byte public key)
      |
[Serialize Private Key (PKCS#8 PEM)]
[Serialize Public Key (SubjectPublicKeyInfo PEM)]
      |
[Encrypt Private Key with Passphrase]
      |
[Store with Metadata]
(key_id, fingerprint, creation_date)
```

## Workflow 2: Sign Document

```
[Document to Sign]
      |
[Load Private Key (decrypt passphrase)]
      |
[Ed25519 Sign]
(deterministic: SHA-512 internal hash)
      |
[Output: 64-byte Signature]
      |
[Create Signature File]
(signature + public key reference + metadata)
```

## Workflow 3: Verify Signature

```
[Document + Signature + Public Key]
      |
[Load Public Key]
      |
[Ed25519 Verify]
      |
[Valid?]
  YES -> Accept document as authentic
  NO  -> Reject (tampering detected)
```

## Workflow 4: Code Signing System

```
[Build Artifact] (binary, package, container)
      |
[Hash Artifact] (SHA-256)
      |
[Create Signing Manifest]
(artifact_name, hash, timestamp, signer_id)
      |
[Sign Manifest with Ed25519]
      |
[Distribute: Artifact + Manifest + Signature + Public Key]
      |
[Recipient Verifies]:
  1. Verify signature on manifest
  2. Hash artifact and compare to manifest
  3. Check signer identity against trust store
```

===== END FILE =====


===== FILE: skills/implementing-digital-signatures-with-ed25519/scripts/agent.py =====
#!/usr/bin/env python3
"""Ed25519 digital signature agent using the cryptography library."""

import argparse
import base64
import hashlib
import json
import logging
import os
import sys
from datetime import datetime
from typing import List

try:
    from cryptography.hazmat.primitives.asymmetric.ed25519 import (
        Ed25519PrivateKey, Ed25519PublicKey)
    from cryptography.hazmat.primitives import serialization
    from cryptography.exceptions import InvalidSignature
except ImportError:
    sys.exit("cryptography required: pip install cryptography")

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def generate_keypair(output_dir: str, key_name: str = "ed25519") -> dict:
    """Generate Ed25519 keypair and save to PEM files."""
    private_key = Ed25519PrivateKey.generate()
    priv_pem = private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=serialization.NoEncryption())
    pub_pem = private_key.public_key().public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo)
    priv_path = os.path.join(output_dir, f"{key_name}_private.pem")
    pub_path = os.path.join(output_dir, f"{key_name}_public.pem")
    with open(priv_path, "wb") as f:
        f.write(priv_pem)
    with open(pub_path, "wb") as f:
        f.write(pub_pem)
    logger.info("Keypair saved: %s, %s", priv_path, pub_path)
    return {"private_key_path": priv_path, "public_key_path": pub_path}


def load_private_key(path: str) -> Ed25519PrivateKey:
    """Load Ed25519 private key from PEM file."""
    with open(path, "rb") as f:
        return serialization.load_pem_private_key(f.read(), password=None)


def load_public_key(path: str) -> Ed25519PublicKey:
    """Load Ed25519 public key from PEM file."""
    with open(path, "rb") as f:
        return serialization.load_pem_public_key(f.read())


def sign_message(private_key_path: str, message: bytes) -> dict:
    """Sign a message with Ed25519 private key."""
    key = load_private_key(private_key_path)
    signature = key.sign(message)
    return {
        "signature_b64": base64.b64encode(signature).decode(),
        "signature_hex": signature.hex(),
        "message_hash": hashlib.sha256(message).hexdigest(),
        "signature_bytes": len(signature),
    }


def sign_file(private_key_path: str, file_path: str) -> dict:
    """Sign a file with Ed25519 and write signature to .sig file."""
    with open(file_path, "rb") as f:
        data = f.read()
    result = sign_message(private_key_path, data)
    sig_path = file_path + ".ed25519.sig"
    with open(sig_path, "w") as f:
        json.dump({"signature": result["signature_b64"],
                    "file_hash": result["message_hash"],
                    "algorithm": "Ed25519",
                    "signed_at": datetime.utcnow().isoformat()}, f, indent=2)
    result["signature_file"] = sig_path
    return result


def verify_message(public_key_path: str, message: bytes, signature_b64: str) -> dict:
    """Verify an Ed25519 signature on a message."""
    key = load_public_key(public_key_path)
    signature = base64.b64decode(signature_b64)
    try:
        key.verify(signature, message)
        return {"valid": True, "algorithm": "Ed25519"}
    except InvalidSignature:
        return {"valid": False, "error": "Signature verification failed"}


def verify_file(public_key_path: str, file_path: str, sig_path: str) -> dict:
    """Verify a file's Ed25519 signature."""
    with open(file_path, "rb") as f:
        data = f.read()
    with open(sig_path) as f:
        sig_data = json.load(f)
    result = verify_message(public_key_path, data, sig_data["signature"])
    result["file"] = file_path
    result["file_hash"] = hashlib.sha256(data).hexdigest()
    result["hash_matches"] = result["file_hash"] == sig_data.get("file_hash", "")
    return result


def batch_verify(public_key_path: str, files: List[str]) -> List[dict]:
    """Verify signatures for multiple files."""
    results = []
    for file_path in files:
        sig_path = file_path + ".ed25519.sig"
        if os.path.isfile(sig_path):
            results.append(verify_file(public_key_path, file_path, sig_path))
        else:
            results.append({"file": file_path, "valid": False, "error": "No signature file"})
    return results


def main():
    parser = argparse.ArgumentParser(description="Ed25519 Digital Signature Agent")
    parser.add_argument("--generate-keys", action="store_true")
    parser.add_argument("--sign", help="File to sign")
    parser.add_argument("--verify", nargs="+", help="Files to verify")
    parser.add_argument("--private-key", help="Private key PEM path")
    parser.add_argument("--public-key", help="Public key PEM path")
    parser.add_argument("--output-dir", default=".")
    parser.add_argument("--output", default="signature_report.json")
    args = parser.parse_args()

    os.makedirs(args.output_dir, exist_ok=True)

    if args.generate_keys:
        result = generate_keypair(args.output_dir)
        print(json.dumps(result, indent=2))
    elif args.sign and args.private_key:
        result = sign_file(args.private_key, args.sign)
        print(json.dumps(result, indent=2))
    elif args.verify and args.public_key:
        results = batch_verify(args.public_key, args.verify)
        report = {"verifications": results,
                  "valid": sum(1 for r in results if r.get("valid")),
                  "invalid": sum(1 for r in results if not r.get("valid"))}
        out_path = os.path.join(args.output_dir, args.output)
        with open(out_path, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-digital-signatures-with-ed25519/scripts/process.py =====
#!/usr/bin/env python3
"""
Ed25519 Digital Signature Tool

Implements Ed25519 key generation, signing, verification, and a
simple code signing system.

Requirements:
    pip install cryptography

Usage:
    python process.py generate --output ./keys
    python process.py sign --key ./keys/private.pem --input document.pdf
    python process.py verify --key ./keys/public.pem --input document.pdf --signature document.pdf.sig
    python process.py code-sign --key ./keys/private.pem --artifact ./build/app.zip
    python process.py benchmark
"""

import os
import sys
import json
import time
import hashlib
import argparse
import logging
import datetime
import base64
from pathlib import Path
from typing import Dict, Optional, Tuple

from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey, Ed25519PublicKey
from cryptography.hazmat.primitives import serialization
from cryptography.exceptions import InvalidSignature

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def generate_ed25519_keypair(
    output_dir: str, passphrase: Optional[str] = None
) -> Dict:
    """Generate an Ed25519 key pair."""
    private_key = Ed25519PrivateKey.generate()
    public_key = private_key.public_key()

    output_path = Path(output_dir)
    output_path.mkdir(parents=True, exist_ok=True)

    if passphrase:
        enc = serialization.BestAvailableEncryption(passphrase.encode())
    else:
        enc = serialization.NoEncryption()

    private_pem = private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=enc,
    )
    (output_path / "private.pem").write_bytes(private_pem)

    public_pem = public_key.public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo,
    )
    (output_path / "public.pem").write_bytes(public_pem)

    # Compute fingerprint
    public_raw = public_key.public_bytes(
        encoding=serialization.Encoding.Raw,
        format=serialization.PublicFormat.Raw,
    )
    fingerprint = hashlib.sha256(public_raw).hexdigest()

    metadata = {
        "algorithm": "Ed25519",
        "public_key_hex": public_raw.hex(),
        "fingerprint_sha256": fingerprint,
        "created_at": datetime.datetime.utcnow().isoformat() + "Z",
        "private_key_path": str(output_path / "private.pem"),
        "public_key_path": str(output_path / "public.pem"),
    }
    (output_path / "key_metadata.json").write_text(json.dumps(metadata, indent=2))

    logger.info(f"Ed25519 key pair generated in {output_dir}")
    logger.info(f"Fingerprint: {fingerprint}")
    return metadata


def load_private_key(path: str, passphrase: Optional[str] = None) -> Ed25519PrivateKey:
    """Load Ed25519 private key from PEM file."""
    data = Path(path).read_bytes()
    pwd = passphrase.encode() if passphrase else None
    key = serialization.load_pem_private_key(data, password=pwd)
    if not isinstance(key, Ed25519PrivateKey):
        raise TypeError("Key is not Ed25519")
    return key


def load_public_key(path: str) -> Ed25519PublicKey:
    """Load Ed25519 public key from PEM file."""
    data = Path(path).read_bytes()
    key = serialization.load_pem_public_key(data)
    if not isinstance(key, Ed25519PublicKey):
        raise TypeError("Key is not Ed25519")
    return key


def sign_data(data: bytes, private_key: Ed25519PrivateKey) -> bytes:
    """Sign data with Ed25519."""
    return private_key.sign(data)


def verify_data(data: bytes, signature: bytes, public_key: Ed25519PublicKey) -> bool:
    """Verify Ed25519 signature."""
    try:
        public_key.verify(signature, data)
        return True
    except InvalidSignature:
        return False


def sign_file(key_path: str, input_path: str, passphrase: Optional[str] = None) -> Dict:
    """Sign a file and save the signature."""
    private_key = load_private_key(key_path, passphrase)
    data = Path(input_path).read_bytes()

    signature = sign_data(data, private_key)
    sig_path = input_path + ".sig"
    Path(sig_path).write_bytes(signature)

    # Also save base64 signature for text-friendly contexts
    sig_b64_path = input_path + ".sig.b64"
    Path(sig_b64_path).write_text(base64.b64encode(signature).decode())

    file_hash = hashlib.sha256(data).hexdigest()

    logger.info(f"Signed {input_path} ({len(data)} bytes)")
    return {
        "file": input_path,
        "signature_file": sig_path,
        "signature_b64_file": sig_b64_path,
        "signature_hex": signature.hex(),
        "file_sha256": file_hash,
        "algorithm": "Ed25519",
    }


def verify_file(key_path: str, input_path: str, sig_path: str) -> Dict:
    """Verify a file's Ed25519 signature."""
    public_key = load_public_key(key_path)
    data = Path(input_path).read_bytes()
    signature = Path(sig_path).read_bytes()

    # Handle base64 encoded signatures
    if len(signature) != 64:
        try:
            signature = base64.b64decode(signature)
        except Exception:
            pass

    valid = verify_data(data, signature, public_key)
    logger.info(f"Verification: {'VALID' if valid else 'INVALID'}")

    return {
        "file": input_path,
        "valid": valid,
        "file_sha256": hashlib.sha256(data).hexdigest(),
        "algorithm": "Ed25519",
    }


def code_sign(key_path: str, artifact_path: str, passphrase: Optional[str] = None) -> Dict:
    """Create a code signing manifest for an artifact."""
    private_key = load_private_key(key_path, passphrase)
    data = Path(artifact_path).read_bytes()

    public_raw = private_key.public_key().public_bytes(
        encoding=serialization.Encoding.Raw,
        format=serialization.PublicFormat.Raw,
    )

    manifest = {
        "artifact": Path(artifact_path).name,
        "size": len(data),
        "sha256": hashlib.sha256(data).hexdigest(),
        "sha512": hashlib.sha512(data).hexdigest(),
        "signer_public_key": public_raw.hex(),
        "signer_fingerprint": hashlib.sha256(public_raw).hexdigest(),
        "signed_at": datetime.datetime.utcnow().isoformat() + "Z",
        "algorithm": "Ed25519",
    }

    manifest_json = json.dumps(manifest, indent=2, sort_keys=True).encode()
    signature = sign_data(manifest_json, private_key)

    signed_manifest = {
        **manifest,
        "signature": base64.b64encode(signature).decode(),
    }

    manifest_path = artifact_path + ".manifest.json"
    Path(manifest_path).write_text(json.dumps(signed_manifest, indent=2))

    logger.info(f"Code signed: {artifact_path}")
    return signed_manifest


def verify_code_signature(manifest_path: str, artifact_path: str) -> Dict:
    """Verify a code signing manifest."""
    signed_manifest = json.loads(Path(manifest_path).read_text())
    signature = base64.b64decode(signed_manifest["signature"])

    public_raw = bytes.fromhex(signed_manifest["signer_public_key"])
    public_key = Ed25519PublicKey.from_public_bytes(public_raw)

    manifest_copy = {k: v for k, v in signed_manifest.items() if k != "signature"}
    manifest_json = json.dumps(manifest_copy, indent=2, sort_keys=True).encode()

    sig_valid = verify_data(manifest_json, signature, public_key)

    data = Path(artifact_path).read_bytes()
    hash_valid = hashlib.sha256(data).hexdigest() == signed_manifest["sha256"]

    return {
        "artifact": artifact_path,
        "signature_valid": sig_valid,
        "hash_valid": hash_valid,
        "overall_valid": sig_valid and hash_valid,
        "signer_fingerprint": signed_manifest["signer_fingerprint"],
    }


def benchmark():
    """Benchmark Ed25519 operations."""
    print("=== Ed25519 Benchmark ===\n")

    # Key generation
    count = 1000
    start = time.time()
    for _ in range(count):
        Ed25519PrivateKey.generate()
    elapsed = time.time() - start
    print(f"Key generation: {count / elapsed:.0f} keys/s ({elapsed / count * 1e6:.1f} us/key)")

    # Signing
    key = Ed25519PrivateKey.generate()
    message = b"Benchmark message for Ed25519 signing performance test." * 10
    count = 5000
    start = time.time()
    for _ in range(count):
        key.sign(message)
    elapsed = time.time() - start
    print(f"Signing:        {count / elapsed:.0f} sigs/s ({elapsed / count * 1e6:.1f} us/sig)")

    # Verification
    public_key = key.public_key()
    signature = key.sign(message)
    count = 2000
    start = time.time()
    for _ in range(count):
        public_key.verify(signature, message)
    elapsed = time.time() - start
    print(f"Verification:   {count / elapsed:.0f} verifs/s ({elapsed / count * 1e6:.1f} us/verify)")


def main():
    parser = argparse.ArgumentParser(description="Ed25519 Digital Signature Tool")
    subparsers = parser.add_subparsers(dest="command")

    gen = subparsers.add_parser("generate", help="Generate Ed25519 key pair")
    gen.add_argument("--output", "-o", default="./keys", help="Output directory")
    gen.add_argument("--passphrase", "-p", help="Passphrase for private key")

    sig = subparsers.add_parser("sign", help="Sign a file")
    sig.add_argument("--key", required=True, help="Private key path")
    sig.add_argument("--input", "-i", required=True, help="File to sign")
    sig.add_argument("--passphrase", "-p", help="Key passphrase")

    ver = subparsers.add_parser("verify", help="Verify signature")
    ver.add_argument("--key", required=True, help="Public key path")
    ver.add_argument("--input", "-i", required=True, help="File to verify")
    ver.add_argument("--signature", "-s", required=True, help="Signature file")

    cs = subparsers.add_parser("code-sign", help="Code sign an artifact")
    cs.add_argument("--key", required=True, help="Private key path")
    cs.add_argument("--artifact", required=True, help="Artifact to sign")
    cs.add_argument("--passphrase", "-p", help="Key passphrase")

    csv = subparsers.add_parser("code-verify", help="Verify code signature")
    csv.add_argument("--manifest", required=True, help="Manifest file path")
    csv.add_argument("--artifact", required=True, help="Artifact file path")

    subparsers.add_parser("benchmark", help="Benchmark Ed25519 performance")

    args = parser.parse_args()

    if args.command == "generate":
        result = generate_ed25519_keypair(args.output, args.passphrase)
        print(json.dumps(result, indent=2))
    elif args.command == "sign":
        result = sign_file(args.key, args.input, args.passphrase)
        print(json.dumps(result, indent=2))
    elif args.command == "verify":
        result = verify_file(args.key, args.input, args.signature)
        print(json.dumps(result, indent=2))
    elif args.command == "code-sign":
        result = code_sign(args.key, args.artifact, args.passphrase)
        print(json.dumps(result, indent=2))
    elif args.command == "code-verify":
        result = verify_code_signature(args.manifest, args.artifact)
        print(json.dumps(result, indent=2))
    elif args.command == "benchmark":
        benchmark()
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-end-to-end-encryption-for-messaging/SKILL.md =====
---
name: implementing-end-to-end-encryption-for-messaging
description: Implements a simplified Signal Protocol-style end-to-end encryption scheme for messaging, covering key exchange, forward secrecy, and the core cryptographic components so no server or intermediary can decrypt messages. Use when designing or building E2EE messaging, or evaluating forward-secrecy and key-management tradeoffs for a messaging system.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- encryption
- e2e
- messaging
- signal-protocol
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
- T1486
---
# Implementing End-to-End Encryption for Messaging

## Overview

End-to-end encryption (E2EE) ensures that only the communicating parties can read messages, with no intermediary (including the server) able to decrypt them. This skill implements a simplified version of the Signal Protocol's Double Ratchet algorithm, using X25519 for key exchange, HKDF for key derivation, and AES-256-GCM for message encryption.


## When to Use

- When deploying or configuring implementing end to end encryption for messaging capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Implement X25519 Diffie-Hellman key exchange for session establishment
- Build the Double Ratchet key management algorithm
- Encrypt and decrypt messages with per-message keys
- Implement forward secrecy (compromise of current key does not reveal past messages)
- Handle out-of-order message delivery
- Implement key agreement using X3DH (Extended Triple Diffie-Hellman)

## Key Concepts

### Signal Protocol Components

| Component | Purpose | Algorithm |
|-----------|---------|-----------|
| X3DH | Initial key agreement | X25519 |
| Double Ratchet | Ongoing key management | X25519 + HKDF + AES-GCM |
| Sending Chain | Per-message encryption keys | HMAC-SHA256 chain |
| Receiving Chain | Per-message decryption keys | HMAC-SHA256 chain |
| Root Chain | Derives new chain keys on DH ratchet | HKDF |

### Forward Secrecy

Each message uses a unique encryption key derived from a ratcheting chain. After a key is used, it is deleted, ensuring that compromise of the current state does not reveal previously sent/received messages.

## Security Considerations

- Delete message keys immediately after decryption
- Implement message ordering and replay protection
- Use authenticated encryption (AES-GCM) for all messages
- Protect identity keys with device-level security
- Verify identity keys out-of-band (safety numbers)

## Validation Criteria

- [ ] X25519 key exchange produces shared secret
- [ ] Messages encrypt and decrypt correctly between two parties
- [ ] Different messages produce different ciphertexts
- [ ] Forward secrecy: old keys cannot decrypt new messages
- [ ] Out-of-order messages can be decrypted
- [ ] Tampered messages are rejected by authentication

===== END FILE =====


===== FILE: skills/implementing-end-to-end-encryption-for-messaging/LICENSE =====

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


===== FILE: skills/implementing-end-to-end-encryption-for-messaging/assets/template.md =====
# E2E Encryption for Messaging Template

## Protocol Summary

| Phase | Algorithm | Purpose |
|-------|-----------|---------|
| Key Exchange | X3DH (X25519) | Initial shared secret |
| Key Ratchet | Double Ratchet | Per-message key derivation |
| Encryption | AES-256-GCM | Message confidentiality + integrity |
| Key Derivation | HKDF-SHA256 | Derive keys from DH outputs |
| Chain KDF | HMAC-SHA256 | Advance symmetric ratchet |

## Security Properties Checklist

- [ ] Forward secrecy: Past messages safe if current keys compromised
- [ ] Post-compromise security: Recovery after temporary key compromise
- [ ] Deniability: No cryptographic proof of message authorship
- [ ] Authenticated encryption: Tampered messages detected and rejected
- [ ] Replay protection: Message counters prevent replay attacks
- [ ] Out-of-order delivery: Skipped keys stored for late messages

## Message Format

```
[Header (40 bytes)]
  - DH Public Key: 32 bytes
  - Previous Chain Length: 4 bytes (big-endian)
  - Message Number: 4 bytes (big-endian)

[Encrypted Payload]
  - Nonce: 12 bytes
  - Ciphertext + Tag: variable
```

## Integration Notes

- Identity keys should be stored in secure device storage (Keychain, TEE)
- Implement safety number verification for identity key comparison
- Handle device changes by re-running X3DH with new identity keys
- Store skipped message keys with a maximum limit (e.g., 1000)
- Delete message keys immediately after successful decryption

===== END FILE =====


===== FILE: skills/implementing-end-to-end-encryption-for-messaging/references/api-reference.md =====
# API Reference — Implementing End-to-End Encryption for Messaging

## Libraries Used
- **cryptography**: X25519 key exchange, HKDF key derivation, AES-256-GCM encryption

## CLI Interface

```
python agent.py keygen                                    # Generate X25519 key pair
python agent.py exchange                                  # Simulate key exchange
python agent.py demo                                      # Full E2EE demo flow
python agent.py encrypt --message <text> --key <hex>      # Encrypt message
python agent.py decrypt --nonce <hex> --ciphertext <hex> --key <hex>
```

## Core Functions

### `generate_keypair()`
Generates X25519 key pair for Diffie-Hellman key exchange.
- `X25519PrivateKey.generate()` -> private key
- `private_key.public_key()` -> public key
- Returns hex-encoded private and public keys.

### `derive_shared_secret(my_private_hex, their_public_hex)`
Performs X25519 ECDH key exchange and derives symmetric key via HKDF-SHA256.
- `my_private.exchange(their_public)` -> 32-byte raw shared secret
- `HKDF(algorithm=SHA256(), length=32, info=b"e2ee-messaging-v1").derive(shared)`

### `encrypt_message(message, shared_key_hex)`
Encrypts plaintext using AES-256-GCM with random 12-byte nonce.
- `AESGCM(key).encrypt(nonce, plaintext, None)` -> ciphertext with GCM tag

### `decrypt_message(nonce_hex, ciphertext_hex, shared_key_hex)`
Decrypts and authenticates ciphertext. Raises `InvalidTag` if tampered.

## Cryptography API Calls

| Class | Module | Purpose |
|-------|--------|---------|
| `X25519PrivateKey` | `cryptography.hazmat.primitives.asymmetric.x25519` | ECDH private key |
| `X25519PublicKey` | same | ECDH public key |
| `AESGCM` | `cryptography.hazmat.primitives.ciphers.aead` | Authenticated encryption |
| `HKDF` | `cryptography.hazmat.primitives.kdf.hkdf` | Key derivation |

## Dependencies
```
pip install cryptography>=41.0
```

===== END FILE =====


===== FILE: skills/implementing-end-to-end-encryption-for-messaging/references/standards.md =====
# Standards and References - End-to-End Encryption for Messaging

## Signal Protocol Specifications

### The Double Ratchet Algorithm
- **URL**: https://signal.org/docs/specifications/doubleratchet/
- **Description**: Core key management algorithm for E2EE messaging

### The X3DH Key Agreement Protocol
- **URL**: https://signal.org/docs/specifications/x3dh/
- **Description**: Initial key exchange using Extended Triple Diffie-Hellman

### The Sesame Algorithm
- **URL**: https://signal.org/docs/specifications/sesame/
- **Description**: Multi-device session management

## Cryptographic Standards

### RFC 7748 - Elliptic Curves for Security (X25519)
- **URL**: https://www.rfc-editor.org/rfc/rfc7748
- **Description**: X25519 Diffie-Hellman key exchange

### RFC 5869 - HKDF (HMAC-based Key Derivation Function)
- **URL**: https://www.rfc-editor.org/rfc/rfc5869
- **Description**: Key derivation for chain key updates

### RFC 8032 - Edwards-Curve Digital Signature Algorithm (Ed25519)
- **URL**: https://www.rfc-editor.org/rfc/rfc8032
- **Description**: Identity key signatures

### NIST SP 800-38D - AES-GCM
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-38d/final
- **Description**: Authenticated encryption for messages

## Python Libraries

### cryptography
- **X25519**: `cryptography.hazmat.primitives.asymmetric.x25519`
- **HKDF**: `cryptography.hazmat.primitives.kdf.hkdf`
- **AES-GCM**: `cryptography.hazmat.primitives.ciphers.aead.AESGCM`

===== END FILE =====


===== FILE: skills/implementing-end-to-end-encryption-for-messaging/references/workflows.md =====
# Workflows - End-to-End Encryption for Messaging

## Workflow 1: X3DH Key Agreement

```
Alice (initiator)                 Server                  Bob (responder)
  |                                 |                         |
  |                                 |<-- Register:            |
  |                                 |    Identity Key (IK_B)  |
  |                                 |    Signed PreKey (SPK_B)|
  |                                 |    One-Time PreKeys     |
  |                                 |                         |
  |-- Fetch Bob's Keys ----------->|                         |
  |<-- IK_B, SPK_B, OPK_B --------|                         |
  |                                 |                         |
  [Compute shared secret]:                                    |
  DH1 = DH(IK_A, SPK_B)                                     |
  DH2 = DH(EK_A, IK_B)                                      |
  DH3 = DH(EK_A, SPK_B)                                     |
  DH4 = DH(EK_A, OPK_B)                                     |
  SK = HKDF(DH1 || DH2 || DH3 || DH4)                      |
  |                                 |                         |
  |-- Send Initial Message ------->|-- Forward to Bob ------>|
  |   (IK_A, EK_A, OPK_id, msg)   |                         |
  |                                 |   [Bob computes same SK]|
```

## Workflow 2: Double Ratchet (Sending)

```
[Message to Send]
      |
[Check: Do we have recipient's new DH public key?]
  YES --> [DH Ratchet Step]
          - Generate new DH key pair
          - Compute DH shared secret
          - Derive new root key + sending chain key via HKDF
  NO  --> [Continue with current sending chain]
      |
[Symmetric Ratchet: Derive message key from sending chain]
(chain_key, message_key) = HMAC(chain_key, constants)
      |
[Encrypt message with AES-256-GCM using message_key]
      |
[Include header: DH public key, previous chain length, message number]
      |
[Delete message_key from memory]
```

## Workflow 3: Double Ratchet (Receiving)

```
[Received Encrypted Message + Header]
      |
[Check DH public key in header]
  [New key?]
    YES --> [DH Ratchet Step]
            - Compute DH shared secret
            - Derive new root key + receiving chain key
    NO  --> [Use current receiving chain]
      |
[Symmetric Ratchet: Derive message key]
      |
[Decrypt message with AES-256-GCM]
      |
[Verify authentication tag]
  FAIL --> Reject message
  PASS --> Return plaintext
      |
[Delete message_key from memory]
```

## Workflow 4: Session Lifecycle

```
[Initial Contact] --> [X3DH Key Exchange]
                            |
                      [Initialize Double Ratchet]
                            |
                      [Exchange Messages]
                      (DH ratchet + symmetric ratchet)
                            |
                      [Periodic DH Ratchet]
                      (every N messages or on reply)
                            |
                      [Session End / Archive]
```

===== END FILE =====


===== FILE: skills/implementing-end-to-end-encryption-for-messaging/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing end-to-end encryption (E2EE) for messaging using X25519 + AES-GCM."""

import json
import argparse
import os

try:
    from cryptography.hazmat.primitives.asymmetric.x25519 import X25519PrivateKey, X25519PublicKey
    from cryptography.hazmat.primitives.ciphers.aead import AESGCM
    from cryptography.hazmat.primitives.kdf.hkdf import HKDF
    from cryptography.hazmat.primitives import hashes, serialization
    HAS_CRYPTO = True
except ImportError:
    HAS_CRYPTO = False

NONCE_SIZE = 12
KEY_SIZE = 32
HKDF_INFO = b"e2ee-messaging-v1"


def generate_keypair():
    """Generate X25519 key pair for Diffie-Hellman key exchange."""
    private_key = X25519PrivateKey.generate()
    public_key = private_key.public_key()
    priv_bytes = private_key.private_bytes(
        serialization.Encoding.Raw, serialization.PrivateFormat.Raw, serialization.NoEncryption()
    )
    pub_bytes = public_key.public_bytes(serialization.Encoding.Raw, serialization.PublicFormat.Raw)
    return {
        "private_key_hex": priv_bytes.hex(),
        "public_key_hex": pub_bytes.hex(),
        "algorithm": "X25519",
    }


def derive_shared_secret(my_private_hex, their_public_hex):
    """Derive shared secret using X25519 ECDH + HKDF-SHA256."""
    my_private = X25519PrivateKey.from_private_bytes(bytes.fromhex(my_private_hex))
    their_public = X25519PublicKey.from_public_bytes(bytes.fromhex(their_public_hex))
    shared_key = my_private.exchange(their_public)
    derived_key = HKDF(
        algorithm=hashes.SHA256(), length=KEY_SIZE, salt=None, info=HKDF_INFO
    ).derive(shared_key)
    return derived_key


def encrypt_message(message, shared_key_hex):
    """Encrypt a message using AES-256-GCM with a shared key."""
    key = bytes.fromhex(shared_key_hex)
    nonce = os.urandom(NONCE_SIZE)
    aesgcm = AESGCM(key)
    ciphertext = aesgcm.encrypt(nonce, message.encode("utf-8"), None)
    return {
        "nonce_hex": nonce.hex(),
        "ciphertext_hex": ciphertext.hex(),
        "algorithm": "AES-256-GCM",
    }


def decrypt_message(nonce_hex, ciphertext_hex, shared_key_hex):
    """Decrypt a message using AES-256-GCM."""
    key = bytes.fromhex(shared_key_hex)
    nonce = bytes.fromhex(nonce_hex)
    ciphertext = bytes.fromhex(ciphertext_hex)
    aesgcm = AESGCM(key)
    plaintext = aesgcm.decrypt(nonce, ciphertext, None)
    return {"plaintext": plaintext.decode("utf-8")}


def simulate_key_exchange(alice_name="Alice", bob_name="Bob"):
    """Simulate a complete key exchange between two parties."""
    alice_kp = generate_keypair()
    bob_kp = generate_keypair()

    alice_shared = derive_shared_secret(alice_kp["private_key_hex"], bob_kp["public_key_hex"])
    bob_shared = derive_shared_secret(bob_kp["private_key_hex"], alice_kp["public_key_hex"])

    keys_match = alice_shared == bob_shared
    return {
        "alice_public_key": alice_kp["public_key_hex"],
        "bob_public_key": bob_kp["public_key_hex"],
        "shared_secret_match": keys_match,
        "shared_key_hex": alice_shared.hex() if keys_match else None,
        "key_exchange": "X25519 ECDH",
        "kdf": "HKDF-SHA256",
        "encryption": "AES-256-GCM",
    }


def demo_full_flow():
    """Demonstrate complete E2EE message flow."""
    kx = simulate_key_exchange()
    if not kx["shared_secret_match"]:
        return {"error": "Key exchange failed"}
    shared_key = kx["shared_key_hex"]
    test_message = "Hello, this is an end-to-end encrypted message."
    encrypted = encrypt_message(test_message, shared_key)
    decrypted = decrypt_message(encrypted["nonce_hex"], encrypted["ciphertext_hex"], shared_key)
    return {
        "key_exchange": kx,
        "original_message": test_message,
        "encrypted": encrypted,
        "decrypted": decrypted,
        "integrity_check": decrypted["plaintext"] == test_message,
    }


def main():
    if not HAS_CRYPTO:
        print(json.dumps({"error": "cryptography library not installed"}))
        return
    parser = argparse.ArgumentParser(description="E2EE Messaging Agent (X25519 + AES-256-GCM)")
    sub = parser.add_subparsers(dest="command")
    sub.add_parser("keygen", help="Generate X25519 key pair")
    sub.add_parser("exchange", help="Simulate key exchange")
    sub.add_parser("demo", help="Full E2EE demo flow")
    e = sub.add_parser("encrypt", help="Encrypt message")
    e.add_argument("--message", required=True)
    e.add_argument("--key", required=True, help="Shared key hex")
    d = sub.add_parser("decrypt", help="Decrypt message")
    d.add_argument("--nonce", required=True)
    d.add_argument("--ciphertext", required=True)
    d.add_argument("--key", required=True, help="Shared key hex")
    args = parser.parse_args()
    if args.command == "keygen":
        result = generate_keypair()
    elif args.command == "exchange":
        result = simulate_key_exchange()
    elif args.command == "demo":
        result = demo_full_flow()
    elif args.command == "encrypt":
        result = encrypt_message(args.message, args.key)
    elif args.command == "decrypt":
        result = decrypt_message(args.nonce, args.ciphertext, args.key)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-end-to-end-encryption-for-messaging/scripts/process.py =====
#!/usr/bin/env python3
"""
End-to-End Encryption for Messaging (Simplified Double Ratchet)

Implements a simplified version of the Signal Protocol's Double Ratchet
algorithm using X25519 key exchange, HKDF key derivation, and AES-256-GCM.

Requirements:
    pip install cryptography

Usage:
    python process.py demo
    python process.py benchmark --messages 1000
"""

import os
import sys
import json
import time
import struct
import hashlib
import argparse
import logging
from dataclasses import dataclass, field
from typing import Dict, Optional, Tuple, List

from cryptography.hazmat.primitives.asymmetric.x25519 import X25519PrivateKey, X25519PublicKey
from cryptography.hazmat.primitives.kdf.hkdf import HKDF
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
from cryptography.hazmat.primitives import hashes, hmac, serialization
from cryptography.hazmat.backends import default_backend

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

INFO_ROOT_KEY = b"DoubleRatchetRootKey"
INFO_CHAIN_KEY = b"DoubleRatchetChainKey"
CHAIN_KEY_CONSTANT = b"\x01"
MESSAGE_KEY_CONSTANT = b"\x02"


def generate_x25519_keypair() -> Tuple[X25519PrivateKey, bytes]:
    """Generate an X25519 key pair, returning (private_key, public_key_bytes)."""
    private_key = X25519PrivateKey.generate()
    public_bytes = private_key.public_key().public_bytes(
        serialization.Encoding.Raw, serialization.PublicFormat.Raw
    )
    return private_key, public_bytes


def dh(private_key: X25519PrivateKey, public_key_bytes: bytes) -> bytes:
    """Perform X25519 Diffie-Hellman key exchange."""
    public_key = X25519PublicKey.from_public_bytes(public_key_bytes)
    return private_key.exchange(public_key)


def hkdf_derive(input_key: bytes, info: bytes, length: int = 64) -> bytes:
    """Derive key material using HKDF-SHA256."""
    derived = HKDF(
        algorithm=hashes.SHA256(),
        length=length,
        salt=b"\x00" * 32,
        info=info,
        backend=default_backend(),
    ).derive(input_key)
    return derived


def hmac_derive(key: bytes, constant: bytes) -> bytes:
    """Derive a key using HMAC-SHA256."""
    h = hmac.HMAC(key, hashes.SHA256(), backend=default_backend())
    h.update(constant)
    return h.finalize()


def kdf_rk(root_key: bytes, dh_output: bytes) -> Tuple[bytes, bytes]:
    """Root key KDF: derive new root key and chain key from DH output."""
    derived = hkdf_derive(dh_output + root_key, INFO_ROOT_KEY, 64)
    new_root_key = derived[:32]
    new_chain_key = derived[32:]
    return new_root_key, new_chain_key


def kdf_ck(chain_key: bytes) -> Tuple[bytes, bytes]:
    """Chain key KDF: derive next chain key and message key."""
    new_chain_key = hmac_derive(chain_key, CHAIN_KEY_CONSTANT)
    message_key = hmac_derive(chain_key, MESSAGE_KEY_CONSTANT)
    return new_chain_key, message_key


def encrypt_message(message_key: bytes, plaintext: bytes, associated_data: bytes = b"") -> bytes:
    """Encrypt a message using AES-256-GCM."""
    nonce = os.urandom(12)
    aesgcm = AESGCM(message_key)
    ciphertext = aesgcm.encrypt(nonce, plaintext, associated_data)
    return nonce + ciphertext


def decrypt_message(message_key: bytes, data: bytes, associated_data: bytes = b"") -> bytes:
    """Decrypt a message using AES-256-GCM."""
    nonce = data[:12]
    ciphertext = data[12:]
    aesgcm = AESGCM(message_key)
    return aesgcm.decrypt(nonce, ciphertext, associated_data)


@dataclass
class MessageHeader:
    """Header included with each encrypted message."""
    dh_public_key: bytes
    previous_chain_length: int
    message_number: int

    def serialize(self) -> bytes:
        return (
            self.dh_public_key
            + struct.pack(">II", self.previous_chain_length, self.message_number)
        )

    @classmethod
    def deserialize(cls, data: bytes) -> "MessageHeader":
        dh_public_key = data[:32]
        prev_chain_len, msg_num = struct.unpack(">II", data[32:40])
        return cls(dh_public_key, prev_chain_len, msg_num)


@dataclass
class DoubleRatchetState:
    """State for one side of the Double Ratchet."""
    dh_self_private: Optional[X25519PrivateKey] = None
    dh_self_public: bytes = b""
    dh_remote_public: bytes = b""
    root_key: bytes = b""
    sending_chain_key: Optional[bytes] = None
    receiving_chain_key: Optional[bytes] = None
    send_count: int = 0
    recv_count: int = 0
    previous_send_count: int = 0
    skipped_keys: Dict[Tuple[bytes, int], bytes] = field(default_factory=dict)
    max_skip: int = 100


def initialize_alice(shared_secret: bytes, bob_dh_public: bytes) -> DoubleRatchetState:
    """Initialize the ratchet for Alice (initiator)."""
    state = DoubleRatchetState()
    state.dh_remote_public = bob_dh_public

    state.dh_self_private, state.dh_self_public = generate_x25519_keypair()

    dh_output = dh(state.dh_self_private, bob_dh_public)
    state.root_key, state.sending_chain_key = kdf_rk(shared_secret, dh_output)
    state.receiving_chain_key = None
    state.send_count = 0
    state.recv_count = 0
    state.previous_send_count = 0

    return state


def initialize_bob(shared_secret: bytes, bob_dh_keypair: Tuple[X25519PrivateKey, bytes]) -> DoubleRatchetState:
    """Initialize the ratchet for Bob (responder)."""
    state = DoubleRatchetState()
    state.dh_self_private = bob_dh_keypair[0]
    state.dh_self_public = bob_dh_keypair[1]
    state.root_key = shared_secret
    state.sending_chain_key = None
    state.receiving_chain_key = None
    state.send_count = 0
    state.recv_count = 0
    state.previous_send_count = 0

    return state


def ratchet_encrypt(state: DoubleRatchetState, plaintext: bytes) -> Tuple[MessageHeader, bytes]:
    """Encrypt a message using the Double Ratchet."""
    state.sending_chain_key, message_key = kdf_ck(state.sending_chain_key)

    header = MessageHeader(
        dh_public_key=state.dh_self_public,
        previous_chain_length=state.previous_send_count,
        message_number=state.send_count,
    )

    ciphertext = encrypt_message(message_key, plaintext, header.serialize())
    state.send_count += 1

    return header, ciphertext


def dh_ratchet_step(state: DoubleRatchetState, header: MessageHeader):
    """Perform a DH ratchet step when receiving a new public key."""
    state.previous_send_count = state.send_count
    state.send_count = 0
    state.recv_count = 0
    state.dh_remote_public = header.dh_public_key

    dh_recv = dh(state.dh_self_private, state.dh_remote_public)
    state.root_key, state.receiving_chain_key = kdf_rk(state.root_key, dh_recv)

    state.dh_self_private, state.dh_self_public = generate_x25519_keypair()

    dh_send = dh(state.dh_self_private, state.dh_remote_public)
    state.root_key, state.sending_chain_key = kdf_rk(state.root_key, dh_send)


def skip_message_keys(state: DoubleRatchetState, until: int):
    """Skip and store message keys for out-of-order messages."""
    if state.receiving_chain_key is None:
        return
    while state.recv_count < until:
        state.receiving_chain_key, mk = kdf_ck(state.receiving_chain_key)
        state.skipped_keys[(state.dh_remote_public, state.recv_count)] = mk
        state.recv_count += 1
        if len(state.skipped_keys) > state.max_skip:
            oldest = next(iter(state.skipped_keys))
            del state.skipped_keys[oldest]


def ratchet_decrypt(state: DoubleRatchetState, header: MessageHeader, ciphertext: bytes) -> bytes:
    """Decrypt a message using the Double Ratchet."""
    # Check skipped keys
    skip_key = (header.dh_public_key, header.message_number)
    if skip_key in state.skipped_keys:
        mk = state.skipped_keys.pop(skip_key)
        return decrypt_message(mk, ciphertext, header.serialize())

    # DH ratchet step if new public key
    if header.dh_public_key != state.dh_remote_public:
        if state.receiving_chain_key is not None:
            skip_message_keys(state, header.previous_chain_length)
        dh_ratchet_step(state, header)

    skip_message_keys(state, header.message_number)
    state.receiving_chain_key, message_key = kdf_ck(state.receiving_chain_key)
    state.recv_count += 1

    return decrypt_message(message_key, ciphertext, header.serialize())


def demo_conversation():
    """Demonstrate a complete E2EE conversation."""
    print("=== End-to-End Encryption Demo ===\n")

    # Simulate X3DH: both parties derive the same shared secret
    alice_ik_private, alice_ik_public = generate_x25519_keypair()
    bob_ik_private, bob_ik_public = generate_x25519_keypair()
    bob_spk_private, bob_spk_public = generate_x25519_keypair()
    alice_ek_private, alice_ek_public = generate_x25519_keypair()

    # Alice computes shared secret
    dh1 = dh(alice_ik_private, bob_spk_public)
    dh2 = dh(alice_ek_private, bob_ik_public)
    dh3 = dh(alice_ek_private, bob_spk_public)
    alice_shared = hkdf_derive(dh1 + dh2 + dh3, b"X3DH", 32)

    # Bob computes same shared secret
    bob_ik_pub_obj = X25519PublicKey.from_public_bytes(alice_ik_public)
    bob_ek_pub_obj = X25519PublicKey.from_public_bytes(alice_ek_public)
    dh1_b = bob_spk_private.exchange(bob_ik_pub_obj)
    dh2_b = bob_ik_private.exchange(bob_ek_pub_obj)
    dh3_b = bob_spk_private.exchange(bob_ek_pub_obj)
    bob_shared = hkdf_derive(dh1_b + dh2_b + dh3_b, b"X3DH", 32)

    assert alice_shared == bob_shared, "X3DH shared secret mismatch!"
    print("[OK] X3DH key agreement: shared secrets match\n")

    # Initialize Double Ratchet
    bob_dh_private, bob_dh_public = generate_x25519_keypair()
    alice_state = initialize_alice(alice_shared, bob_dh_public)
    bob_state = initialize_bob(bob_shared, (bob_dh_private, bob_dh_public))

    # Alice sends messages to Bob
    messages = [
        b"Hello Bob! This is an encrypted message.",
        b"Can you read this?",
        b"This uses the Double Ratchet algorithm.",
    ]

    print("--- Alice sends to Bob ---")
    for msg in messages:
        header, ct = ratchet_encrypt(alice_state, msg)
        pt = ratchet_decrypt(bob_state, header, ct)
        print(f"  Alice -> Bob: {pt.decode()}")
        assert pt == msg

    # Bob replies to Alice
    replies = [
        b"Hi Alice! Yes, I can read your messages.",
        b"The DH ratchet just advanced!",
    ]

    print("\n--- Bob sends to Alice ---")
    for msg in replies:
        header, ct = ratchet_encrypt(bob_state, msg)
        pt = ratchet_decrypt(alice_state, header, ct)
        print(f"  Bob -> Alice: {pt.decode()}")
        assert pt == msg

    # Alice sends again (another DH ratchet)
    print("\n--- Alice sends again (new DH ratchet) ---")
    msg = b"This message uses a new DH ratchet step."
    header, ct = ratchet_encrypt(alice_state, msg)
    pt = ratchet_decrypt(bob_state, header, ct)
    print(f"  Alice -> Bob: {pt.decode()}")
    assert pt == msg

    print("\n[OK] All messages encrypted and decrypted successfully")
    print("[OK] Forward secrecy: each message uses a unique key")
    print("[OK] DH ratchet advanced on direction change")


def benchmark(num_messages: int = 1000):
    """Benchmark encryption/decryption throughput."""
    alice_ik_private, _ = generate_x25519_keypair()
    bob_spk_private, bob_spk_public = generate_x25519_keypair()
    shared = dh(alice_ik_private, bob_spk_public)
    shared_key = hkdf_derive(shared, b"benchmark", 32)

    bob_dh_private, bob_dh_public = generate_x25519_keypair()
    alice_state = initialize_alice(shared_key, bob_dh_public)
    bob_state = initialize_bob(shared_key, (bob_dh_private, bob_dh_public))

    message = b"Benchmark message for throughput testing. " * 10

    start = time.time()
    for _ in range(num_messages):
        header, ct = ratchet_encrypt(alice_state, message)
        pt = ratchet_decrypt(bob_state, header, ct)
    elapsed = time.time() - start

    print(f"Messages: {num_messages}")
    print(f"Time: {elapsed:.3f}s")
    print(f"Throughput: {num_messages / elapsed:.0f} msg/s")
    print(f"Latency: {elapsed / num_messages * 1000:.2f} ms/msg")


def main():
    parser = argparse.ArgumentParser(description="E2EE Messaging Demo")
    subparsers = parser.add_subparsers(dest="command")

    subparsers.add_parser("demo", help="Run E2EE conversation demo")

    bench = subparsers.add_parser("benchmark", help="Benchmark throughput")
    bench.add_argument("--messages", type=int, default=1000, help="Number of messages")

    args = parser.parse_args()

    if args.command == "demo":
        demo_conversation()
    elif args.command == "benchmark":
        benchmark(args.messages)
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-envelope-encryption-with-aws-kms/SKILL.md =====
---
name: implementing-envelope-encryption-with-aws-kms
description: Implements envelope encryption with AWS KMS, encrypting data locally with a data encryption key (DEK) and protecting that DEK with a KMS-managed key (KEK), covering the encrypt/decrypt flow, KMS key types, and security validation criteria. Use when designing key management for encrypting large or numerous data objects on AWS, or when reducing direct KMS API call volume.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- encryption
- aws
- kms
- envelope-encryption
- key-management
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
- T1078.004
- T1530
---
# Implementing Envelope Encryption with AWS KMS

## Overview

Envelope encryption is a strategy where data is encrypted with a data encryption key (DEK), and the DEK itself is encrypted with a master key (KEK) managed by AWS KMS. This approach allows encrypting large volumes of data locally while keeping the master key secure in a hardware security module (HSM) managed by AWS. This skill covers implementing envelope encryption using AWS KMS GenerateDataKey API.


## When to Use

- When deploying or configuring implementing envelope encryption with aws kms capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Understand the envelope encryption pattern and its advantages
- Generate data encryption keys using AWS KMS GenerateDataKey
- Encrypt/decrypt data locally using DEKs
- Store encrypted DEK alongside ciphertext
- Implement key caching to reduce KMS API calls
- Handle key rotation with automatic re-encryption
- Implement multi-region encryption for disaster recovery

## Key Concepts

### Envelope Encryption Flow

1. Call `kms:GenerateDataKey` to get plaintext DEK + encrypted DEK
2. Use plaintext DEK to encrypt data locally (AES-256-GCM)
3. Store encrypted DEK alongside ciphertext
4. Discard plaintext DEK from memory
5. For decryption: call `kms:Decrypt` on encrypted DEK, then decrypt data

### Advantages Over Direct KMS Encryption

| Aspect | Direct KMS | Envelope Encryption |
|--------|-----------|-------------------|
| Max data size | 4 KB | Unlimited |
| Latency | Network round-trip per operation | Local encryption |
| Cost | $0.03/10,000 requests | Fewer KMS requests |
| Offline | Not possible | Yes (with cached DEKs) |

### KMS Key Types

- **AWS Managed**: AWS creates and manages (`aws/s3`, `aws/ebs`)
- **Customer Managed**: You create and manage policies
- **Custom Key Store**: Backed by CloudHSM cluster

## Security Considerations

- Never store plaintext DEK; only keep encrypted DEK
- Use key policies to restrict who can call GenerateDataKey and Decrypt
- Enable AWS CloudTrail logging for all KMS API calls
- Implement key rotation (automatic annual rotation for CMKs)
- Use encryption context for authenticated encryption metadata
- Handle KMS throttling with exponential backoff

## Validation Criteria

- [ ] GenerateDataKey returns plaintext and encrypted DEK
- [ ] Data encrypts correctly with plaintext DEK using AES-256-GCM
- [ ] Encrypted DEK can be decrypted via KMS Decrypt API
- [ ] Decrypted DEK recovers the original data
- [ ] Plaintext DEK is wiped from memory after use
- [ ] Encryption context is validated during decryption
- [ ] Key rotation re-encrypts DEKs with new master key

===== END FILE =====


===== FILE: skills/implementing-envelope-encryption-with-aws-kms/LICENSE =====

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


===== FILE: skills/implementing-envelope-encryption-with-aws-kms/assets/template.md =====
# Envelope Encryption with AWS KMS Template

## Prerequisites Checklist

- [ ] AWS account with KMS access
- [ ] IAM policy allows kms:GenerateDataKey, kms:Decrypt, kms:ReEncrypt
- [ ] KMS Customer Managed Key (CMK) created
- [ ] CloudTrail logging enabled for KMS events
- [ ] boto3 and cryptography Python libraries installed

## IAM Policy Template

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kms:GenerateDataKey",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:DescribeKey"
      ],
      "Resource": "arn:aws:kms:us-east-1:123456789012:key/your-key-id"
    }
  ]
}
```

## KMS Key Policy Template

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowKeyAdministration",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:role/KeyAdmin"},
      "Action": [
        "kms:Create*", "kms:Describe*", "kms:Enable*", "kms:List*",
        "kms:Put*", "kms:Update*", "kms:Revoke*", "kms:Disable*",
        "kms:Get*", "kms:Delete*", "kms:ScheduleKeyDeletion",
        "kms:CancelKeyDeletion"
      ],
      "Resource": "*"
    },
    {
      "Sid": "AllowKeyUsage",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:role/AppRole"},
      "Action": ["kms:Decrypt", "kms:GenerateDataKey", "kms:ReEncrypt*"],
      "Resource": "*"
    }
  ]
}
```

## Quick Reference

```python
import boto3
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

kms = boto3.client('kms')

# Encrypt
resp = kms.generate_data_key(KeyId='alias/my-key', KeySpec='AES_256')
plaintext_key = resp['Plaintext']
encrypted_key = resp['CiphertextBlob']

nonce = os.urandom(12)
ciphertext = AESGCM(plaintext_key).encrypt(nonce, data, None)
# Store: encrypted_key + nonce + ciphertext

# Decrypt
resp = kms.decrypt(CiphertextBlob=encrypted_key)
plaintext_key = resp['Plaintext']
data = AESGCM(plaintext_key).decrypt(nonce, ciphertext, None)
```

## Cost Estimation

| Operation | Price | Notes |
|-----------|-------|-------|
| KMS API requests | $0.03 per 10,000 | GenerateDataKey, Decrypt |
| CMK storage | $1.00 per month | Per customer managed key |
| Key rotation | Free | Automatic annual rotation |

===== END FILE =====


===== FILE: skills/implementing-envelope-encryption-with-aws-kms/references/api-reference.md =====
# API Reference — Implementing Envelope Encryption with AWS KMS

## Libraries Used
- **boto3**: AWS SDK for KMS key management and data key generation
- **cryptography**: AES-256-GCM for local data encryption with generated data keys

## CLI Interface

```
python agent.py --region us-east-1 encrypt --input <file> --output <out> --key-id <kms_key>
python agent.py --region us-east-1 decrypt --input <encrypted> --output <out>
python agent.py --region us-east-1 list-keys
python agent.py --region us-east-1 audit --key-id <kms_key>
```

## Core Functions

### `generate_data_key(kms_key_id, region)`
Generates a data encryption key (DEK) using AWS KMS.
- `kms.generate_data_key(KeyId=key_id, KeySpec="AES_256")`
- Returns plaintext key (for local encryption) and encrypted key (for storage).

### `encrypt_data(plaintext_bytes, kms_key_id, region)`
Performs envelope encryption: generates DEK via KMS, encrypts data locally with AES-256-GCM, stores encrypted DEK alongside ciphertext.

### `decrypt_data(envelope, region)`
Decrypts envelope: calls `kms.decrypt(CiphertextBlob=encrypted_key)` to recover DEK, then decrypts data locally.

### `list_kms_keys(region)`
Lists KMS keys with metadata using `kms.list_keys()` and `kms.describe_key()`.

### `audit_key_policy(key_id, region)`
Audits KMS key policy for overly permissive principals (`Principal: "*"`).
- `kms.get_key_policy(KeyId=key_id, PolicyName="default")`

## boto3 KMS API Calls

| Method | Purpose |
|--------|---------|
| `kms.generate_data_key(KeyId, KeySpec)` | Generate plaintext + encrypted DEK |
| `kms.decrypt(CiphertextBlob)` | Decrypt encrypted DEK back to plaintext |
| `kms.list_keys()` | List all KMS keys in the account |
| `kms.describe_key(KeyId)` | Get key metadata (state, usage, origin) |
| `kms.get_key_policy(KeyId, PolicyName)` | Get key resource policy JSON |

## Dependencies
```
pip install boto3>=1.28 cryptography>=41.0
```

===== END FILE =====


===== FILE: skills/implementing-envelope-encryption-with-aws-kms/references/standards.md =====
# Standards and References - Envelope Encryption with AWS KMS

## AWS Documentation

### AWS KMS Developer Guide
- **URL**: https://docs.aws.amazon.com/kms/latest/developerguide/
- **Envelope Encryption**: https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#enveloping

### AWS KMS API Reference
- **GenerateDataKey**: https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html
- **Decrypt**: https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html
- **ReEncrypt**: https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html

### AWS Encryption SDK
- **URL**: https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/
- **Python**: https://aws-encryption-sdk-python.readthedocs.io/

## Cryptographic Standards

### NIST SP 800-57 Part 1 - Key Management
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final
- **Relevance**: Key hierarchy and key wrapping concepts

### NIST SP 800-38F - Key Wrap
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-38f/final
- **Description**: AES Key Wrap specification used by KMS internally

### FIPS 140-2 Level 2 (KMS HSMs)
- **Description**: KMS HSMs are validated at FIPS 140-2 Level 2 (Level 3 for CloudHSM)

## Compliance Frameworks

### PCI DSS v4.0 Requirement 3
- Key management with separation of DEK and KEK
- KMS satisfies key management requirements

### SOC 2 Type II
- AWS KMS is SOC 2 compliant
- Encryption controls map to CC6.1 (logical access controls)

### HIPAA
- KMS encryption satisfies encryption requirements for ePHI
- BAA required with AWS

## Python Libraries

### boto3 (AWS SDK for Python)
- **URL**: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html
- **PyPI**: https://pypi.org/project/boto3/

### aws-encryption-sdk
- **URL**: https://pypi.org/project/aws-encryption-sdk/
- **Description**: High-level envelope encryption with caching

===== END FILE =====


===== FILE: skills/implementing-envelope-encryption-with-aws-kms/references/workflows.md =====
# Workflows - Envelope Encryption with AWS KMS

## Workflow 1: Encrypt Data with Envelope Encryption

```
[Application]
      |
[Call KMS GenerateDataKey]
(KeyId=CMK ARN, KeySpec=AES_256)
      |
[KMS Returns]:
  - Plaintext DEK (32 bytes)
  - Encrypted DEK (ciphertext blob)
      |
[Encrypt Data Locally]
(AES-256-GCM with plaintext DEK)
      |
[Store]:
  - Encrypted DEK (ciphertext blob)
  - Encrypted data (nonce + ciphertext + tag)
  - Encryption context metadata
      |
[Wipe Plaintext DEK from Memory]
```

## Workflow 2: Decrypt Data

```
[Read Stored Data]
  - Encrypted DEK
  - Encrypted data
  - Encryption context
      |
[Call KMS Decrypt]
(CiphertextBlob=Encrypted DEK, EncryptionContext)
      |
[KMS Returns Plaintext DEK]
      |
[Decrypt Data Locally]
(AES-256-GCM with plaintext DEK)
      |
[Return Plaintext Data]
      |
[Wipe Plaintext DEK from Memory]
```

## Workflow 3: Key Rotation

```
[Enable Automatic Key Rotation on CMK]
(KMS rotates backing key annually)
      |
[New GenerateDataKey calls use new backing key]
      |
[Old encrypted DEKs still decrypt]
(KMS tracks all backing key versions)
      |
[Optional: Re-encrypt old DEKs]
[Call KMS ReEncrypt to update DEK encryption]
```

## Workflow 4: Multi-Region Encryption

```
[Primary Region (us-east-1)]
  |
  [Create Multi-Region CMK]
  [Replicate to us-west-2, eu-west-1]
  |
  [Encrypt with Regional Endpoint]
  |
[Secondary Region (us-west-2)]
  |
  [Same Key ID works for Decrypt]
  [No cross-region API calls needed]
```

===== END FILE =====


===== FILE: skills/implementing-envelope-encryption-with-aws-kms/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing envelope encryption using AWS KMS."""

import json
import argparse
import os
import base64

try:
    import boto3
    from botocore.exceptions import ClientError
except ImportError:
    boto3 = None

try:
    from cryptography.hazmat.primitives.ciphers.aead import AESGCM
except ImportError:
    AESGCM = None

NONCE_SIZE = 12


def generate_data_key(kms_key_id, region="us-east-1"):
    """Generate a data encryption key using AWS KMS."""
    kms = boto3.client("kms", region_name=region)
    resp = kms.generate_data_key(KeyId=kms_key_id, KeySpec="AES_256")
    return {
        "plaintext_key": resp["Plaintext"],
        "encrypted_key": resp["CiphertextBlob"],
        "key_id": resp["KeyId"],
    }


def encrypt_data(plaintext_bytes, kms_key_id, region="us-east-1"):
    """Encrypt data using envelope encryption with AWS KMS."""
    key_data = generate_data_key(kms_key_id, region)
    nonce = os.urandom(NONCE_SIZE)
    aesgcm = AESGCM(key_data["plaintext_key"])
    ciphertext = aesgcm.encrypt(nonce, plaintext_bytes, None)

    # Zero out plaintext key from memory
    key_data["plaintext_key"] = b"\x00" * 32

    envelope = {
        "encrypted_data_key": base64.b64encode(key_data["encrypted_key"]).decode(),
        "nonce": base64.b64encode(nonce).decode(),
        "ciphertext": base64.b64encode(ciphertext).decode(),
        "kms_key_id": key_data["key_id"],
        "algorithm": "AES-256-GCM",
        "envelope_version": 1,
    }
    return envelope


def decrypt_data(envelope, region="us-east-1"):
    """Decrypt envelope-encrypted data using AWS KMS."""
    kms = boto3.client("kms", region_name=region)
    encrypted_key = base64.b64decode(envelope["encrypted_data_key"])
    resp = kms.decrypt(CiphertextBlob=encrypted_key)
    plaintext_key = resp["Plaintext"]

    nonce = base64.b64decode(envelope["nonce"])
    ciphertext = base64.b64decode(envelope["ciphertext"])
    aesgcm = AESGCM(plaintext_key)
    plaintext = aesgcm.decrypt(nonce, ciphertext, None)

    # Zero out plaintext key
    plaintext_key = b"\x00" * 32
    return plaintext


def encrypt_file(input_path, output_path, kms_key_id, region="us-east-1"):
    """Encrypt a file using envelope encryption."""
    with open(input_path, "rb") as f:
        plaintext = f.read()
    envelope = encrypt_data(plaintext, kms_key_id, region)
    with open(output_path, "w") as f:
        json.dump(envelope, f, indent=2)
    return {
        "input": str(input_path),
        "output": str(output_path),
        "original_size": len(plaintext),
        "kms_key_id": envelope["kms_key_id"],
    }


def decrypt_file(input_path, output_path, region="us-east-1"):
    """Decrypt an envelope-encrypted file."""
    with open(input_path, "r") as f:
        envelope = json.load(f)
    plaintext = decrypt_data(envelope, region)
    with open(output_path, "wb") as f:
        f.write(plaintext)
    return {"input": str(input_path), "output": str(output_path), "decrypted_size": len(plaintext)}


def list_kms_keys(region="us-east-1"):
    """List available KMS keys."""
    kms = boto3.client("kms", region_name=region)
    paginator = kms.get_paginator("list_keys")
    keys = []
    for page in paginator.paginate():
        for key in page["Keys"]:
            try:
                desc = kms.describe_key(KeyId=key["KeyId"])
                meta = desc["KeyMetadata"]
                keys.append({
                    "key_id": meta["KeyId"],
                    "arn": meta["Arn"],
                    "description": meta.get("Description", ""),
                    "state": meta["KeyState"],
                    "key_usage": meta["KeyUsage"],
                    "origin": meta["Origin"],
                })
            except ClientError:
                keys.append({"key_id": key["KeyId"], "error": "access denied"})
    return {"keys": keys, "total": len(keys)}


def audit_key_policy(key_id, region="us-east-1"):
    """Audit a KMS key's policy for overly permissive access."""
    kms = boto3.client("kms", region_name=region)
    policy = json.loads(kms.get_key_policy(KeyId=key_id, PolicyName="default")["Policy"])
    findings = []
    for stmt in policy.get("Statement", []):
        principal = stmt.get("Principal", {})
        if principal == "*" or principal.get("AWS") == "*":
            findings.append({
                "severity": "HIGH",
                "finding": "Key policy allows access to all AWS principals",
                "statement_id": stmt.get("Sid", "unknown"),
            })
    return {"key_id": key_id, "policy": policy, "findings": findings}


def main():
    if not boto3 or not AESGCM:
        print(json.dumps({"error": "boto3 and cryptography required"}))
        return
    parser = argparse.ArgumentParser(description="AWS KMS Envelope Encryption Agent")
    parser.add_argument("--region", default="us-east-1")
    sub = parser.add_subparsers(dest="command")
    e = sub.add_parser("encrypt", help="Encrypt file with envelope encryption")
    e.add_argument("--input", required=True)
    e.add_argument("--output", required=True)
    e.add_argument("--key-id", required=True, help="KMS key ID or ARN")
    d = sub.add_parser("decrypt", help="Decrypt envelope-encrypted file")
    d.add_argument("--input", required=True)
    d.add_argument("--output", required=True)
    sub.add_parser("list-keys", help="List KMS keys")
    a = sub.add_parser("audit", help="Audit KMS key policy")
    a.add_argument("--key-id", required=True)
    args = parser.parse_args()
    if args.command == "encrypt":
        result = encrypt_file(args.input, args.output, args.key_id, args.region)
    elif args.command == "decrypt":
        result = decrypt_file(args.input, args.output, args.region)
    elif args.command == "list-keys":
        result = list_kms_keys(args.region)
    elif args.command == "audit":
        result = audit_key_policy(args.key_id, args.region)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-envelope-encryption-with-aws-kms/scripts/process.py =====
#!/usr/bin/env python3
"""
Envelope Encryption with AWS KMS

Implements envelope encryption pattern using AWS KMS for master key management
and local AES-256-GCM for data encryption.

Requirements:
    pip install boto3 cryptography

Usage:
    python process.py encrypt --key-id alias/my-key --input data.json --output data.json.enc
    python process.py decrypt --input data.json.enc --output data.json
    python process.py encrypt-file --key-id alias/my-key --input largefile.zip --output largefile.zip.enc
    python process.py re-encrypt --key-id alias/new-key --input data.json.enc --output data.json.reenc

Environment:
    AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION
    or ~/.aws/credentials
"""

import os
import sys
import json
import struct
import argparse
import logging
import base64
import ctypes
from pathlib import Path
from typing import Dict, Optional, Tuple

import boto3
from botocore.exceptions import ClientError
from cryptography.hazmat.primitives.ciphers.aead import AESGCM

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

NONCE_LENGTH = 12
MAGIC_BYTES = b"ENVKMS01"


def secure_wipe(data: bytearray):
    """Attempt to securely wipe sensitive data from memory."""
    if isinstance(data, (bytearray, memoryview)):
        for i in range(len(data)):
            data[i] = 0


def generate_data_key(
    kms_client,
    key_id: str,
    encryption_context: Optional[Dict[str, str]] = None,
) -> Tuple[bytes, bytes]:
    """
    Generate a data encryption key (DEK) using AWS KMS.

    Returns:
        Tuple of (plaintext_key, encrypted_key)
    """
    params = {
        "KeyId": key_id,
        "KeySpec": "AES_256",
    }
    if encryption_context:
        params["EncryptionContext"] = encryption_context

    try:
        response = kms_client.generate_data_key(**params)
        plaintext_key = response["Plaintext"]
        encrypted_key = response["CiphertextBlob"]
        logger.info(f"Generated data key using KMS key: {key_id}")
        return plaintext_key, encrypted_key
    except ClientError as e:
        logger.error(f"KMS GenerateDataKey failed: {e}")
        raise


def decrypt_data_key(
    kms_client,
    encrypted_key: bytes,
    encryption_context: Optional[Dict[str, str]] = None,
) -> bytes:
    """Decrypt an encrypted data key using AWS KMS."""
    params = {"CiphertextBlob": encrypted_key}
    if encryption_context:
        params["EncryptionContext"] = encryption_context

    try:
        response = kms_client.decrypt(**params)
        return response["Plaintext"]
    except ClientError as e:
        logger.error(f"KMS Decrypt failed: {e}")
        raise


def re_encrypt_data_key(
    kms_client,
    encrypted_key: bytes,
    new_key_id: str,
    source_context: Optional[Dict[str, str]] = None,
    dest_context: Optional[Dict[str, str]] = None,
) -> bytes:
    """Re-encrypt a data key with a new master key without exposing plaintext."""
    params = {
        "CiphertextBlob": encrypted_key,
        "DestinationKeyId": new_key_id,
    }
    if source_context:
        params["SourceEncryptionContext"] = source_context
    if dest_context:
        params["DestinationEncryptionContext"] = dest_context

    try:
        response = kms_client.re_encrypt(**params)
        return response["CiphertextBlob"]
    except ClientError as e:
        logger.error(f"KMS ReEncrypt failed: {e}")
        raise


def envelope_encrypt(
    data: bytes,
    kms_client,
    key_id: str,
    encryption_context: Optional[Dict[str, str]] = None,
) -> bytes:
    """
    Encrypt data using envelope encryption.

    Output format:
        MAGIC (8 bytes) || encrypted_key_len (4 bytes, big-endian) ||
        encrypted_key (variable) || context_len (4 bytes) || context_json (variable) ||
        nonce (12 bytes) || ciphertext+tag (variable)
    """
    plaintext_key, encrypted_key = generate_data_key(kms_client, key_id, encryption_context)

    try:
        nonce = os.urandom(NONCE_LENGTH)
        aesgcm = AESGCM(plaintext_key)
        ciphertext = aesgcm.encrypt(nonce, data, associated_data=None)
    finally:
        # Wipe plaintext key
        if isinstance(plaintext_key, bytes):
            plaintext_key = bytearray(plaintext_key)
        secure_wipe(plaintext_key)

    context_json = json.dumps(encryption_context or {}).encode()

    output = bytearray()
    output.extend(MAGIC_BYTES)
    output.extend(struct.pack(">I", len(encrypted_key)))
    output.extend(encrypted_key)
    output.extend(struct.pack(">I", len(context_json)))
    output.extend(context_json)
    output.extend(nonce)
    output.extend(ciphertext)

    return bytes(output)


def envelope_decrypt(
    data: bytes,
    kms_client,
) -> Tuple[bytes, Dict]:
    """
    Decrypt envelope-encrypted data.

    Returns:
        Tuple of (plaintext, encryption_context)
    """
    if not data.startswith(MAGIC_BYTES):
        raise ValueError("Invalid envelope encryption format")

    offset = len(MAGIC_BYTES)

    enc_key_len = struct.unpack(">I", data[offset : offset + 4])[0]
    offset += 4
    encrypted_key = data[offset : offset + enc_key_len]
    offset += enc_key_len

    ctx_len = struct.unpack(">I", data[offset : offset + 4])[0]
    offset += 4
    context_json = data[offset : offset + ctx_len]
    offset += ctx_len
    encryption_context = json.loads(context_json.decode())

    nonce = data[offset : offset + NONCE_LENGTH]
    offset += NONCE_LENGTH
    ciphertext = data[offset:]

    plaintext_key = decrypt_data_key(
        kms_client,
        encrypted_key,
        encryption_context if encryption_context else None,
    )

    try:
        aesgcm = AESGCM(plaintext_key)
        plaintext = aesgcm.decrypt(nonce, ciphertext, associated_data=None)
    finally:
        if isinstance(plaintext_key, bytes):
            plaintext_key = bytearray(plaintext_key)
        secure_wipe(plaintext_key)

    return plaintext, encryption_context


def envelope_re_encrypt(
    data: bytes,
    kms_client,
    new_key_id: str,
) -> bytes:
    """Re-encrypt an envelope-encrypted file with a new KMS key."""
    if not data.startswith(MAGIC_BYTES):
        raise ValueError("Invalid envelope encryption format")

    offset = len(MAGIC_BYTES)
    enc_key_len = struct.unpack(">I", data[offset : offset + 4])[0]
    offset += 4
    encrypted_key = data[offset : offset + enc_key_len]
    offset += enc_key_len

    ctx_len = struct.unpack(">I", data[offset : offset + 4])[0]
    ctx_start = offset + 4
    context_json = data[ctx_start : ctx_start + ctx_len]
    encryption_context = json.loads(context_json.decode())

    remainder = data[ctx_start + ctx_len :]

    new_encrypted_key = re_encrypt_data_key(
        kms_client,
        encrypted_key,
        new_key_id,
        source_context=encryption_context if encryption_context else None,
        dest_context=encryption_context if encryption_context else None,
    )

    output = bytearray()
    output.extend(MAGIC_BYTES)
    output.extend(struct.pack(">I", len(new_encrypted_key)))
    output.extend(new_encrypted_key)
    output.extend(struct.pack(">I", ctx_len))
    output.extend(context_json)
    output.extend(remainder)

    logger.info(f"Re-encrypted data key with new KMS key: {new_key_id}")
    return bytes(output)


def encrypt_file(
    input_path: str,
    output_path: str,
    kms_client,
    key_id: str,
    encryption_context: Optional[Dict[str, str]] = None,
) -> Dict:
    """Encrypt a file using envelope encryption."""
    plaintext = Path(input_path).read_bytes()

    if encryption_context is None:
        encryption_context = {"filename": Path(input_path).name}

    encrypted = envelope_encrypt(plaintext, kms_client, key_id, encryption_context)

    Path(output_path).write_bytes(encrypted)
    logger.info(f"Encrypted {input_path} -> {output_path}")

    return {
        "input": input_path,
        "output": output_path,
        "original_size": len(plaintext),
        "encrypted_size": len(encrypted),
        "kms_key_id": key_id,
        "encryption_context": encryption_context,
    }


def decrypt_file(input_path: str, output_path: str, kms_client) -> Dict:
    """Decrypt an envelope-encrypted file."""
    data = Path(input_path).read_bytes()
    plaintext, context = envelope_decrypt(data, kms_client)

    Path(output_path).write_bytes(plaintext)
    logger.info(f"Decrypted {input_path} -> {output_path}")

    return {
        "input": input_path,
        "output": output_path,
        "decrypted_size": len(plaintext),
        "encryption_context": context,
    }


def main():
    parser = argparse.ArgumentParser(description="Envelope Encryption with AWS KMS")
    parser.add_argument("--region", default=None, help="AWS region")
    parser.add_argument("--profile", default=None, help="AWS profile")
    subparsers = parser.add_subparsers(dest="command")

    enc = subparsers.add_parser("encrypt", help="Encrypt a file")
    enc.add_argument("--key-id", required=True, help="KMS key ID or alias")
    enc.add_argument("--input", "-i", required=True, help="Input file")
    enc.add_argument("--output", "-o", required=True, help="Output file")
    enc.add_argument("--context", help="Encryption context as JSON string")

    dec = subparsers.add_parser("decrypt", help="Decrypt a file")
    dec.add_argument("--input", "-i", required=True, help="Encrypted input file")
    dec.add_argument("--output", "-o", required=True, help="Output file")

    reenc = subparsers.add_parser("re-encrypt", help="Re-encrypt with new key")
    reenc.add_argument("--key-id", required=True, help="New KMS key ID or alias")
    reenc.add_argument("--input", "-i", required=True, help="Encrypted input file")
    reenc.add_argument("--output", "-o", required=True, help="Output file")

    args = parser.parse_args()

    session_kwargs = {}
    if args.region:
        session_kwargs["region_name"] = args.region
    if args.profile:
        session_kwargs["profile_name"] = args.profile

    session = boto3.Session(**session_kwargs)
    kms_client = session.client("kms")

    if args.command == "encrypt":
        context = json.loads(args.context) if args.context else None
        result = encrypt_file(args.input, args.output, kms_client, args.key_id, context)
        print(json.dumps(result, indent=2))
    elif args.command == "decrypt":
        result = decrypt_file(args.input, args.output, kms_client)
        print(json.dumps(result, indent=2))
    elif args.command == "re-encrypt":
        data = Path(args.input).read_bytes()
        result = envelope_re_encrypt(data, kms_client, args.key_id)
        Path(args.output).write_bytes(result)
        print(json.dumps({"input": args.input, "output": args.output, "new_key_id": args.key_id}))
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-jwt-signing-and-verification/SKILL.md =====
---
name: implementing-jwt-signing-and-verification
description: >-
  Implements secure JWT (RFC 7519) signing and verification using HMAC-SHA256,
  RSA-PSS, ES256, and EdDSA, including token expiration, claims validation, and
  defenses against algorithm-confusion, none-algorithm, and key-injection
  attacks. Use when adding or hardening JWT-based authentication/authorization,
  or when auditing token verification code for common JWT vulnerabilities.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- jwt
- authentication
- token-security
- digital-signatures
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
---
# Implementing JWT Signing and Verification

## Overview

JSON Web Tokens (JWT) defined in RFC 7519 are compact, URL-safe tokens used for authentication and authorization in web applications. This skill covers implementing secure JWT signing with HMAC-SHA256, RSA-PSS, and EdDSA algorithms, along with verification, token expiration, claims validation, and defense against common JWT attacks (algorithm confusion, none algorithm, key injection).


## When to Use

- When deploying or configuring implementing jwt signing and verification capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Implement JWT signing with HS256, RS256, ES256, and EdDSA
- Verify JWT signatures and validate standard claims
- Implement token expiration, not-before, and audience validation
- Defend against algorithm confusion and none algorithm attacks
- Implement JWT key rotation with JWK Sets
- Build a complete authentication middleware

## Key Concepts

### JWT Algorithms

| Algorithm | Type | Key | Security Level |
|-----------|------|-----|---------------|
| HS256 | Symmetric (HMAC) | Shared secret | 128-bit |
| RS256 | Asymmetric (RSA) | RSA key pair | 112-bit |
| ES256 | Asymmetric (ECDSA) | P-256 key pair | 128-bit |
| EdDSA | Asymmetric (Ed25519) | Ed25519 pair | 128-bit |

### Common JWT Attacks

- **Algorithm confusion**: Switching from RS256 to HS256, using public key as HMAC secret
- **None algorithm**: Setting alg=none to bypass signature verification
- **Key injection**: Embedding key in JWK header
- **Weak secrets**: Brute-forcing short HMAC secrets
- **Token replay**: Reusing valid tokens without expiration

## Security Considerations

- Always validate the algorithm header against an allowlist
- Never accept alg=none in production
- Use asymmetric algorithms (RS256, ES256) for distributed systems
- Set short expiration times (15 min for access tokens)
- Implement token refresh mechanism
- Store secrets securely (not in source code)

## Validation Criteria

- [ ] JWT signing produces valid tokens for all algorithms
- [ ] Signature verification rejects tampered tokens
- [ ] Expired tokens are rejected
- [ ] Algorithm confusion attack is prevented
- [ ] None algorithm is rejected
- [ ] JWK key rotation works correctly
- [ ] Claims validation enforces all required claims

===== END FILE =====


===== FILE: skills/implementing-jwt-signing-and-verification/LICENSE =====

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


===== FILE: skills/implementing-jwt-signing-and-verification/assets/template.md =====
# JWT Implementation Template

## Algorithm Selection Guide

| Use Case | Recommended Algorithm | Reason |
|----------|----------------------|--------|
| Single server | HS256 | Simple, fast, shared secret |
| Microservices | RS256 / ES256 | Asymmetric, verify without secret |
| Mobile/IoT | ES256 | Small key/signature size |
| High performance | EdDSA | Fastest asymmetric signing |

## JWT Claims Checklist

- [ ] `sub` - Subject (user ID)
- [ ] `iss` - Issuer (your app identifier)
- [ ] `aud` - Audience (intended recipient)
- [ ] `exp` - Expiration (short-lived: 15 min access, 7 day refresh)
- [ ] `nbf` - Not before (prevents premature use)
- [ ] `iat` - Issued at (token creation time)
- [ ] `jti` - JWT ID (unique, for revocation)

## Security Checklist

- [ ] Algorithm allowlist enforced (never accept unknown alg)
- [ ] `alg: none` explicitly rejected
- [ ] Short expiration (access: 15 min, refresh: 7 days)
- [ ] Issuer and audience validation enabled
- [ ] Secrets >= 256 bits for HMAC algorithms
- [ ] RSA keys >= 2048 bits
- [ ] Tokens stored securely on client (httpOnly cookies preferred)
- [ ] Token refresh mechanism implemented
- [ ] Token revocation mechanism available (blacklist/JTI check)

===== END FILE =====


===== FILE: skills/implementing-jwt-signing-and-verification/references/api-reference.md =====
# API Reference: Implementing JWT Signing and Verification

## PyJWT Library

```python
import jwt
# Sign with HS256
token = jwt.encode({"sub": "user1", "exp": time.time() + 3600}, "secret", algorithm="HS256")
# Verify
payload = jwt.decode(token, "secret", algorithms=["HS256"])
# Sign with RS256
token = jwt.encode(payload, private_key, algorithm="RS256")
payload = jwt.decode(token, public_key, algorithms=["RS256"])
```

## JWT Algorithms

| Algorithm | Type | Key Size | Use Case |
|-----------|------|----------|----------|
| HS256 | HMAC | 256-bit secret | Internal services |
| RS256 | RSA | 2048+ bit | Public verification |
| ES256 | ECDSA | P-256 curve | Compact tokens |
| EdDSA | Ed25519 | 256-bit | High performance |
| none | - | - | NEVER use in production |

## Standard JWT Claims (RFC 7519)

| Claim | Type | Description |
|-------|------|-------------|
| `iss` | String | Issuer |
| `sub` | String | Subject |
| `aud` | String/Array | Audience |
| `exp` | NumericDate | Expiration time |
| `nbf` | NumericDate | Not before |
| `iat` | NumericDate | Issued at |
| `jti` | String | JWT ID (unique) |

## Common JWT Attacks

| Attack | Description | Mitigation |
|--------|-------------|-----------|
| Algorithm confusion | Switch RS256 to HS256 | Explicit algorithm allowlist |
| none algorithm | Remove signature | Reject alg=none |
| JKU/JWK injection | Inject attacker key | Ignore JKU/JWK headers |
| Token replay | Reuse valid token | Use jti + short exp |

### References

- RFC 7519 (JWT): https://datatracker.ietf.org/doc/html/rfc7519
- PyJWT: https://pyjwt.readthedocs.io/
- jose (JavaScript): https://github.com/panva/jose
- JWT.io Debugger: https://jwt.io/

===== END FILE =====


===== FILE: skills/implementing-jwt-signing-and-verification/references/standards.md =====
# Standards and References - JWT Signing and Verification

## Primary Standards

### RFC 7519 - JSON Web Token (JWT)
- **URL**: https://www.rfc-editor.org/rfc/rfc7519
- **Description**: JWT format and claims specification

### RFC 7515 - JSON Web Signature (JWS)
- **URL**: https://www.rfc-editor.org/rfc/rfc7515
- **Description**: Signature mechanism for JWTs

### RFC 7517 - JSON Web Key (JWK)
- **URL**: https://www.rfc-editor.org/rfc/rfc7517
- **Description**: Key format for JWT signing keys

### RFC 7518 - JSON Web Algorithms (JWA)
- **URL**: https://www.rfc-editor.org/rfc/rfc7518
- **Description**: Cryptographic algorithms for JWS/JWE

### RFC 8725 - JWT Best Current Practices
- **URL**: https://www.rfc-editor.org/rfc/rfc8725
- **Description**: Security best practices for JWT implementations

## OWASP References

### OWASP JWT Cheat Sheet
- **URL**: https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html

## Python Libraries

### PyJWT
- **URL**: https://pyjwt.readthedocs.io/
- **PyPI**: https://pypi.org/project/PyJWT/

### python-jose
- **URL**: https://python-jose.readthedocs.io/
- **PyPI**: https://pypi.org/project/python-jose/

===== END FILE =====


===== FILE: skills/implementing-jwt-signing-and-verification/references/workflows.md =====
# Workflows - JWT Signing and Verification

## Workflow 1: Token Issuance

```
[Authentication Request] (username + password)
      |
[Validate Credentials]
      |
[Build JWT Claims]:
  - sub: user ID
  - iss: issuer URL
  - aud: audience
  - exp: expiration (now + 15 min)
  - iat: issued at
  - jti: unique token ID
      |
[Sign with Private Key / Secret]
(RS256 / ES256 / HS256)
      |
[Return: access_token + refresh_token]
```

## Workflow 2: Token Verification

```
[Incoming Request with Bearer Token]
      |
[Extract Token from Authorization Header]
      |
[Decode Header (without verification)]
[Check alg against allowlist]
      |
[Verify Signature]
(using public key / shared secret)
      |
[Validate Claims]:
  - exp: not expired
  - nbf: not before current time
  - iss: expected issuer
  - aud: expected audience
      |
[Accept / Reject Request]
```

## Workflow 3: Key Rotation

```
[Generate New Signing Key]
      |
[Add to JWK Set with unique kid]
      |
[Update /.well-known/jwks.json]
(new key + old key)
      |
[New tokens signed with new key]
[Old tokens still verify with old key]
      |
[After grace period: remove old key]
```

===== END FILE =====


===== FILE: skills/implementing-jwt-signing-and-verification/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for JWT signing, verification, and security auditing."""

import json
import argparse
import base64
import hmac
import hashlib
import time
from datetime import datetime

try:
    from cryptography.hazmat.primitives.asymmetric import rsa
    from cryptography.hazmat.primitives import serialization
    from cryptography.hazmat.primitives.asymmetric import padding
    HAS_CRYPTO = True
except ImportError:
    HAS_CRYPTO = False


def b64url_encode(data):
    """Base64url encode without padding."""
    if isinstance(data, str):
        data = data.encode()
    return base64.urlsafe_b64encode(data).rstrip(b"=").decode()


def b64url_decode(data):
    """Base64url decode with padding restoration."""
    padding_needed = 4 - len(data) % 4
    if padding_needed != 4:
        data += "=" * padding_needed
    return base64.urlsafe_b64decode(data)


def create_jwt_hs256(payload, secret):
    """Create a JWT signed with HMAC-SHA256."""
    header = {"alg": "HS256", "typ": "JWT"}
    header_b64 = b64url_encode(json.dumps(header))
    payload_b64 = b64url_encode(json.dumps(payload))
    signing_input = f"{header_b64}.{payload_b64}"
    signature = hmac.new(secret.encode(), signing_input.encode(), hashlib.sha256).digest()
    sig_b64 = b64url_encode(signature)
    return f"{signing_input}.{sig_b64}"


def verify_jwt_hs256(token, secret):
    """Verify an HS256 JWT and return claims."""
    parts = token.split(".")
    if len(parts) != 3:
        return {"valid": False, "error": "Invalid token format"}
    header_b64, payload_b64, sig_b64 = parts
    signing_input = f"{header_b64}.{payload_b64}"
    expected_sig = hmac.new(secret.encode(), signing_input.encode(), hashlib.sha256).digest()
    actual_sig = b64url_decode(sig_b64)
    if not hmac.compare_digest(expected_sig, actual_sig):
        return {"valid": False, "error": "Signature verification failed"}
    header = json.loads(b64url_decode(header_b64))
    payload = json.loads(b64url_decode(payload_b64))
    now = int(time.time())
    if payload.get("exp") and payload["exp"] < now:
        return {"valid": False, "error": "Token expired", "claims": payload}
    if payload.get("nbf") and payload["nbf"] > now:
        return {"valid": False, "error": "Token not yet valid", "claims": payload}
    return {"valid": True, "header": header, "claims": payload}


def decode_jwt_unsafe(token):
    """Decode a JWT without verification (for inspection only)."""
    parts = token.split(".")
    if len(parts) != 3:
        return {"error": "Invalid token format"}
    header = json.loads(b64url_decode(parts[0]))
    payload = json.loads(b64url_decode(parts[1]))
    return {"header": header, "payload": payload, "signature_present": len(parts[2]) > 0}


def audit_jwt_security(token):
    """Audit a JWT for common security vulnerabilities."""
    findings = []
    decoded = decode_jwt_unsafe(token)
    if "error" in decoded:
        return [{"issue": decoded["error"], "severity": "HIGH"}]
    header = decoded["header"]
    payload = decoded["payload"]

    alg = header.get("alg", "")
    if alg == "none":
        findings.append({"issue": "Algorithm 'none' - unsigned token",
                         "severity": "CRITICAL"})
    if alg == "HS256" and header.get("jwk"):
        findings.append({"issue": "JWK in header with symmetric algorithm - key injection risk",
                         "severity": "CRITICAL"})
    if alg in ("HS256", "HS384", "HS512"):
        findings.append({"issue": f"Symmetric algorithm {alg} - shared secret risk",
                         "severity": "MEDIUM",
                         "recommendation": "Use RS256 or ES256 for multi-party verification"})

    if not payload.get("exp"):
        findings.append({"issue": "No expiration claim (exp)", "severity": "HIGH"})
    else:
        exp = payload["exp"]
        now = int(time.time())
        if exp - now > 86400:
            findings.append({"issue": f"Long expiration: {(exp - now) / 3600:.0f} hours",
                             "severity": "MEDIUM"})
    if not payload.get("iss"):
        findings.append({"issue": "No issuer claim (iss)", "severity": "MEDIUM"})
    if not payload.get("aud"):
        findings.append({"issue": "No audience claim (aud)", "severity": "MEDIUM"})
    if not payload.get("iat"):
        findings.append({"issue": "No issued-at claim (iat)", "severity": "LOW"})
    if not payload.get("jti"):
        findings.append({"issue": "No JWT ID (jti) - replay attack risk", "severity": "MEDIUM"})

    sensitive_keys = ["password", "secret", "ssn", "credit_card", "api_key"]
    for key in payload:
        if any(s in key.lower() for s in sensitive_keys):
            findings.append({"issue": f"Sensitive data in claim: {key}",
                             "severity": "HIGH"})

    return findings


def generate_rsa_keypair():
    """Generate RSA key pair for RS256 JWT signing."""
    if not HAS_CRYPTO:
        return {"error": "cryptography library not available"}
    private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
    priv_pem = private_key.private_bytes(
        serialization.Encoding.PEM, serialization.PrivateFormat.PKCS8,
        serialization.NoEncryption()).decode()
    pub_pem = private_key.public_key().public_bytes(
        serialization.Encoding.PEM, serialization.PublicFormat.SubjectPublicKeyInfo).decode()
    return {"private_key": priv_pem, "public_key": pub_pem, "algorithm": "RS256"}


def main():
    parser = argparse.ArgumentParser(description="JWT Security Agent")
    parser.add_argument("--create", help="JSON payload to sign as JWT")
    parser.add_argument("--verify", help="JWT token to verify")
    parser.add_argument("--audit", help="JWT token to audit for vulnerabilities")
    parser.add_argument("--decode", help="JWT token to decode (no verification)")
    parser.add_argument("--secret", default="change-me-secret", help="HMAC secret")
    parser.add_argument("--gen-keys", action="store_true", help="Generate RSA key pair")
    parser.add_argument("--output", default="jwt_report.json")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "results": {}}

    if args.create:
        payload = json.loads(args.create)
        payload.setdefault("iat", int(time.time()))
        payload.setdefault("exp", int(time.time()) + 3600)
        token = create_jwt_hs256(payload, args.secret)
        report["results"]["token"] = token
        print(f"[+] JWT: {token[:50]}...")

    if args.verify:
        result = verify_jwt_hs256(args.verify, args.secret)
        report["results"]["verification"] = result
        print(f"[+] Valid: {result['valid']}")

    if args.audit:
        findings = audit_jwt_security(args.audit)
        report["results"]["audit"] = findings
        critical = sum(1 for f in findings if f.get("severity") == "CRITICAL")
        print(f"[+] Audit: {len(findings)} findings, {critical} critical")

    if args.decode:
        decoded = decode_jwt_unsafe(args.decode)
        report["results"]["decoded"] = decoded
        print(f"[+] Algorithm: {decoded.get('header', {}).get('alg', 'unknown')}")

    if args.gen_keys:
        keys = generate_rsa_keypair()
        report["results"]["keys"] = keys
        print("[+] RSA-2048 key pair generated")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-jwt-signing-and-verification/scripts/process.py =====
#!/usr/bin/env python3
"""
JWT Signing and Verification Tool

Implements secure JWT creation and verification with multiple algorithms,
including defense against common JWT attacks.

Requirements:
    pip install PyJWT cryptography

Usage:
    python process.py create --alg RS256 --subject user123 --issuer myapp --expiry 900
    python process.py verify --token <jwt> --key ./public.pem --issuer myapp
    python process.py generate-keys --alg RS256 --output ./jwt-keys
    python process.py attack-demo  # Demonstrates and defends common attacks
"""

import os
import sys
import json
import time
import hmac
import hashlib
import base64
import argparse
import logging
import datetime
from pathlib import Path
from typing import Dict, Optional, List

import jwt
from cryptography.hazmat.primitives.asymmetric import rsa, ec, ed25519
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.backends import default_backend

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

ALLOWED_ALGORITHMS = ["HS256", "HS384", "HS512", "RS256", "RS384", "RS512",
                       "ES256", "ES384", "ES512", "EdDSA"]


def generate_signing_keys(algorithm: str, output_dir: str) -> Dict:
    """Generate signing keys for a JWT algorithm."""
    output_path = Path(output_dir)
    output_path.mkdir(parents=True, exist_ok=True)

    if algorithm.startswith("HS"):
        key_size = {"HS256": 32, "HS384": 48, "HS512": 64}.get(algorithm, 32)
        secret = os.urandom(key_size)
        secret_hex = secret.hex()
        (output_path / "secret.key").write_text(secret_hex)
        return {"algorithm": algorithm, "key_type": "symmetric", "key_file": str(output_path / "secret.key")}

    if algorithm.startswith("RS"):
        key_size = {"RS256": 2048, "RS384": 3072, "RS512": 4096}.get(algorithm, 2048)
        private_key = rsa.generate_private_key(public_exponent=65537, key_size=key_size, backend=default_backend())
    elif algorithm.startswith("ES"):
        curve = {"ES256": ec.SECP256R1(), "ES384": ec.SECP384R1(), "ES512": ec.SECP521R1()}.get(algorithm, ec.SECP256R1())
        private_key = ec.generate_private_key(curve, default_backend())
    elif algorithm == "EdDSA":
        private_key = ed25519.Ed25519PrivateKey.generate()
    else:
        raise ValueError(f"Unsupported algorithm: {algorithm}")

    priv_pem = private_key.private_bytes(
        serialization.Encoding.PEM, serialization.PrivateFormat.PKCS8, serialization.NoEncryption()
    )
    pub_pem = private_key.public_key().public_bytes(
        serialization.Encoding.PEM, serialization.PublicFormat.SubjectPublicKeyInfo
    )

    (output_path / "private.pem").write_bytes(priv_pem)
    (output_path / "public.pem").write_bytes(pub_pem)

    return {
        "algorithm": algorithm,
        "key_type": "asymmetric",
        "private_key": str(output_path / "private.pem"),
        "public_key": str(output_path / "public.pem"),
    }


def create_jwt(
    algorithm: str,
    signing_key,
    subject: str,
    issuer: str,
    audience: Optional[str] = None,
    expiry_seconds: int = 900,
    extra_claims: Optional[Dict] = None,
) -> str:
    """Create a signed JWT."""
    if algorithm not in ALLOWED_ALGORITHMS:
        raise ValueError(f"Algorithm {algorithm} not in allowlist: {ALLOWED_ALGORITHMS}")

    now = datetime.datetime.utcnow()
    payload = {
        "sub": subject,
        "iss": issuer,
        "iat": now,
        "exp": now + datetime.timedelta(seconds=expiry_seconds),
        "nbf": now,
        "jti": os.urandom(16).hex(),
    }
    if audience:
        payload["aud"] = audience
    if extra_claims:
        payload.update(extra_claims)

    token = jwt.encode(payload, signing_key, algorithm=algorithm)
    return token


def verify_jwt(
    token: str,
    verification_key,
    algorithms: List[str],
    issuer: Optional[str] = None,
    audience: Optional[str] = None,
) -> Dict:
    """
    Securely verify a JWT with algorithm allowlist.
    Defends against algorithm confusion by requiring explicit algorithm list.
    """
    # Reject 'none' algorithm
    safe_algorithms = [a for a in algorithms if a.lower() != "none"]
    if not safe_algorithms:
        raise ValueError("No valid algorithms specified")

    try:
        options = {}
        kwargs = {"algorithms": safe_algorithms}
        if issuer:
            kwargs["issuer"] = issuer
        if audience:
            kwargs["audience"] = audience

        payload = jwt.decode(token, verification_key, **kwargs)
        return {"valid": True, "payload": payload}
    except jwt.ExpiredSignatureError:
        return {"valid": False, "error": "Token has expired"}
    except jwt.InvalidIssuerError:
        return {"valid": False, "error": "Invalid issuer"}
    except jwt.InvalidAudienceError:
        return {"valid": False, "error": "Invalid audience"}
    except jwt.InvalidAlgorithmError:
        return {"valid": False, "error": "Algorithm not allowed"}
    except jwt.InvalidSignatureError:
        return {"valid": False, "error": "Invalid signature"}
    except jwt.DecodeError as e:
        return {"valid": False, "error": f"Decode error: {e}"}
    except Exception as e:
        return {"valid": False, "error": str(e)}


def decode_jwt_unverified(token: str) -> Dict:
    """Decode JWT header and payload without verification (for inspection only)."""
    parts = token.split(".")
    if len(parts) != 3:
        return {"error": "Invalid JWT format"}

    def decode_part(part):
        padding = 4 - len(part) % 4
        part += "=" * padding
        return json.loads(base64.urlsafe_b64decode(part))

    header = decode_part(parts[0])
    payload = decode_part(parts[1])
    return {"header": header, "payload": payload}


def attack_demo():
    """Demonstrate common JWT attacks and defenses."""
    print("=== JWT Security Attack Demonstrations ===\n")

    # Setup
    private_key = rsa.generate_private_key(65537, 2048, default_backend())
    public_key = private_key.public_key()
    priv_pem = private_key.private_bytes(
        serialization.Encoding.PEM, serialization.PrivateFormat.PKCS8, serialization.NoEncryption()
    )
    pub_pem = public_key.public_bytes(
        serialization.Encoding.PEM, serialization.PublicFormat.SubjectPublicKeyInfo
    )

    # Create legitimate token
    token = create_jwt("RS256", priv_pem, "user123", "myapp", expiry_seconds=3600)
    print(f"[1] Legitimate RS256 token created")

    # Verify legitimate token
    result = verify_jwt(token, pub_pem, ["RS256"], issuer="myapp")
    print(f"    Verification: {result['valid']}")

    # Attack 1: Algorithm Confusion (RS256 -> HS256)
    print(f"\n[2] Attack: Algorithm Confusion (RS256 -> HS256)")
    try:
        malicious_token = jwt.encode(
            {"sub": "admin", "iss": "myapp", "exp": datetime.datetime.utcnow() + datetime.timedelta(hours=1)},
            pub_pem, algorithm="HS256"
        )
        result = verify_jwt(malicious_token, pub_pem, ["RS256"])  # Only allow RS256
        print(f"    Defense: Algorithm restricted to RS256 only -> {result}")
    except Exception as e:
        print(f"    Defense: Attack blocked -> {e}")

    # Attack 2: None Algorithm
    print(f"\n[3] Attack: None Algorithm")
    header = base64.urlsafe_b64encode(json.dumps({"alg": "none", "typ": "JWT"}).encode()).rstrip(b"=").decode()
    payload = base64.urlsafe_b64encode(json.dumps({"sub": "admin", "iss": "myapp"}).encode()).rstrip(b"=").decode()
    none_token = f"{header}.{payload}."
    result = verify_jwt(none_token, pub_pem, ["RS256"])
    print(f"    Defense: None algorithm rejected -> {result}")

    # Attack 3: Expired Token
    print(f"\n[4] Attack: Expired Token Replay")
    expired_token = create_jwt("RS256", priv_pem, "user123", "myapp", expiry_seconds=-10)
    result = verify_jwt(expired_token, pub_pem, ["RS256"], issuer="myapp")
    print(f"    Defense: Expired token rejected -> {result}")

    # Attack 4: Wrong Issuer
    print(f"\n[5] Attack: Wrong Issuer")
    wrong_issuer_token = create_jwt("RS256", priv_pem, "user123", "evil-app")
    result = verify_jwt(wrong_issuer_token, pub_pem, ["RS256"], issuer="myapp")
    print(f"    Defense: Wrong issuer rejected -> {result}")

    print(f"\n[OK] All attacks successfully defended")


def main():
    parser = argparse.ArgumentParser(description="JWT Signing and Verification Tool")
    subparsers = parser.add_subparsers(dest="command")

    gen = subparsers.add_parser("generate-keys", help="Generate signing keys")
    gen.add_argument("--alg", required=True, choices=ALLOWED_ALGORITHMS, help="Algorithm")
    gen.add_argument("--output", "-o", default="./jwt-keys", help="Output directory")

    create = subparsers.add_parser("create", help="Create a JWT")
    create.add_argument("--alg", required=True, choices=ALLOWED_ALGORITHMS)
    create.add_argument("--key", required=True, help="Signing key file")
    create.add_argument("--subject", required=True, help="Subject claim")
    create.add_argument("--issuer", required=True, help="Issuer claim")
    create.add_argument("--audience", help="Audience claim")
    create.add_argument("--expiry", type=int, default=900, help="Expiry in seconds")

    verify = subparsers.add_parser("verify", help="Verify a JWT")
    verify.add_argument("--token", required=True, help="JWT token")
    verify.add_argument("--key", required=True, help="Verification key file")
    verify.add_argument("--alg", nargs="+", default=["RS256"], help="Allowed algorithms")
    verify.add_argument("--issuer", help="Expected issuer")
    verify.add_argument("--audience", help="Expected audience")

    inspect = subparsers.add_parser("inspect", help="Inspect JWT without verification")
    inspect.add_argument("--token", required=True, help="JWT token")

    subparsers.add_parser("attack-demo", help="Demonstrate JWT attacks and defenses")

    args = parser.parse_args()

    if args.command == "generate-keys":
        result = generate_signing_keys(args.alg, args.output)
        print(json.dumps(result, indent=2))
    elif args.command == "create":
        key_data = Path(args.key).read_text().strip()
        if args.alg.startswith("HS"):
            key_data = bytes.fromhex(key_data)
        token = create_jwt(args.alg, key_data, args.subject, args.issuer, args.audience, args.expiry)
        print(token)
    elif args.command == "verify":
        key_data = Path(args.key).read_text().strip()
        result = verify_jwt(args.token, key_data, args.alg, args.issuer, args.audience)
        print(json.dumps(result, indent=2, default=str))
    elif args.command == "inspect":
        result = decode_jwt_unverified(args.token)
        print(json.dumps(result, indent=2, default=str))
    elif args.command == "attack-demo":
        attack_demo()
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-rsa-key-pair-management/SKILL.md =====
---
name: implementing-rsa-key-pair-management
description: Generates, stores, rotates, and manages RSA key pairs following NIST
  SP 800-57 guidelines, covering serialization formats (PEM, DER, PKCS#8), passphrase
  protection, and key strength validation. Use when creating or rotating RSA keys
  for signatures, key exchange, or encryption, or when auditing existing keys for
  proper storage and NIST-compliant strength.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- rsa
- key-management
- pki
- asymmetric-encryption
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
- T1486
---
# Implementing RSA Key Pair Management

## Overview

RSA (Rivest-Shamir-Adleman) is the most widely deployed asymmetric cryptographic algorithm, used for digital signatures, key exchange, and encryption. This skill covers generating, storing, rotating, and managing RSA key pairs following NIST SP 800-57 key management guidelines, including key serialization formats (PEM, DER, PKCS#8), passphrase protection, and key strength validation.


## When to Use

- When deploying or configuring implementing rsa key pair management capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Generate RSA key pairs with appropriate key sizes (2048, 3072, 4096 bits)
- Serialize keys in PEM and DER formats with PKCS#8
- Protect private keys with strong passphrase encryption
- Implement key rotation with versioning
- Extract public key components and fingerprints
- Validate key strength and detect weak keys
- Sign and verify data using RSA-PSS

## Key Concepts

### RSA Key Sizes and Security Strength

| Key Size (bits) | Security Strength (bits) | Recommended Until |
|-----------------|-------------------------|-------------------|
| 2048            | 112                     | 2030              |
| 3072            | 128                     | Beyond 2030       |
| 4096            | ~140                    | Beyond 2030       |

### RSA Padding Schemes

| Scheme | Use Case | Standard |
|--------|----------|----------|
| OAEP   | Encryption | PKCS#1 v2.2 (RFC 8017) |
| PSS    | Signatures | PKCS#1 v2.2 (RFC 8017) |
| PKCS#1 v1.5 | Legacy only | Deprecated for new systems |

### Key Storage Formats

- **PEM**: Base64-encoded with headers, human-readable
- **DER**: Binary ASN.1 encoding, compact
- **PKCS#8**: Standard for private key encapsulation
- **PKCS#12/PFX**: Bundled key + certificate, password-protected

## Security Considerations

- Minimum 3072-bit keys for new deployments (NIST recommendation)
- Always protect private keys with AES-256-CBC passphrase encryption
- Use RSA-PSS for signatures (not PKCS#1 v1.5)
- Use RSA-OAEP for encryption (not PKCS#1 v1.5)
- Store private keys with restrictive file permissions (0600)
- Implement key rotation at least annually

## Validation Criteria

- [ ] Key generation produces valid RSA key pair
- [ ] Public key can be extracted from private key
- [ ] Private key is protected with passphrase
- [ ] RSA-PSS signature verification succeeds
- [ ] Tampered signature verification fails
- [ ] Key fingerprint is computed correctly
- [ ] Key rotation maintains old key access for verification

===== END FILE =====


===== FILE: skills/implementing-rsa-key-pair-management/LICENSE =====

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


===== FILE: skills/implementing-rsa-key-pair-management/assets/template.md =====
# RSA Key Pair Management Template

## Key Generation Checklist

- [ ] Select key size (minimum 3072 bits for new deployments)
- [ ] Generate key pair using secure random number generator
- [ ] Protect private key with strong passphrase (AES-256)
- [ ] Compute and record key fingerprint (SHA-256)
- [ ] Set restrictive file permissions on private key
- [ ] Store public key in accessible location
- [ ] Document key metadata (size, algorithm, creation date)

## Key Metadata Template

```json
{
  "key_id": "rsa-prod-001",
  "algorithm": "RSA",
  "key_size": 4096,
  "public_exponent": 65537,
  "fingerprint_sha256": "<hex-digest>",
  "created_at": "2024-01-01T00:00:00Z",
  "expires_at": "2025-01-01T00:00:00Z",
  "usage": ["sign", "verify"],
  "owner": "security-team",
  "version": 1
}
```

## Key Rotation Schedule

| Environment | Rotation Frequency | Grace Period |
|------------|-------------------|--------------|
| Production | 12 months | 30 days |
| Staging    | 6 months  | 14 days |
| Development| 3 months  | 7 days  |

## Quick Reference

```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes, serialization

# Generate
key = rsa.generate_private_key(public_exponent=65537, key_size=4096)

# Sign (RSA-PSS)
signature = key.sign(data, padding.PSS(
    mgf=padding.MGF1(hashes.SHA256()),
    salt_length=padding.PSS.MAX_LENGTH), hashes.SHA256())

# Verify
key.public_key().verify(signature, data, padding.PSS(
    mgf=padding.MGF1(hashes.SHA256()),
    salt_length=padding.PSS.MAX_LENGTH), hashes.SHA256())
```

===== END FILE =====


===== FILE: skills/implementing-rsa-key-pair-management/references/api-reference.md =====
# API Reference: RSA Key Pair Lifecycle Management

## Libraries Used

| Library | Purpose |
|---------|---------|
| `cryptography` | RSA key generation, signing, verification, serialization |
| `os` | Secure random bytes, file permissions |
| `datetime` | Certificate validity periods and key rotation schedules |
| `json` | Export key metadata and audit reports |

## Installation

```bash
pip install cryptography
```

## Key Generation

### Generate RSA Key Pair
```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes, serialization

def generate_rsa_keypair(key_size=4096):
    """Generate an RSA key pair. Use 2048 minimum, 4096 recommended."""
    private_key = rsa.generate_private_key(
        public_exponent=65537,
        key_size=key_size,
    )
    return private_key
```

### Serialize Private Key (PEM, encrypted)
```python
def save_private_key(private_key, filepath, passphrase):
    pem = private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=serialization.BestAvailableEncryption(
            passphrase.encode()
        ),
    )
    with open(filepath, "wb") as f:
        f.write(pem)
    os.chmod(filepath, 0o600)  # Restrict permissions
```

### Serialize Public Key
```python
def save_public_key(private_key, filepath):
    public_key = private_key.public_key()
    pem = public_key.public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo,
    )
    with open(filepath, "wb") as f:
        f.write(pem)
```

### Load Existing Key
```python
def load_private_key(filepath, passphrase=None):
    with open(filepath, "rb") as f:
        private_key = serialization.load_pem_private_key(
            f.read(),
            password=passphrase.encode() if passphrase else None,
        )
    return private_key

def load_public_key(filepath):
    with open(filepath, "rb") as f:
        public_key = serialization.load_pem_public_key(f.read())
    return public_key
```

## Signing and Verification

### Sign Data
```python
def sign_data(private_key, data):
    signature = private_key.sign(
        data,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH,
        ),
        hashes.SHA256(),
    )
    return signature
```

### Verify Signature
```python
from cryptography.exceptions import InvalidSignature

def verify_signature(public_key, data, signature):
    try:
        public_key.verify(
            signature,
            data,
            padding.PSS(
                mgf=padding.MGF1(hashes.SHA256()),
                salt_length=padding.PSS.MAX_LENGTH,
            ),
            hashes.SHA256(),
        )
        return True
    except InvalidSignature:
        return False
```

## Encryption and Decryption

### Encrypt with RSA-OAEP
```python
def encrypt_data(public_key, plaintext):
    ciphertext = public_key.encrypt(
        plaintext,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None,
        ),
    )
    return ciphertext
```

### Decrypt with RSA-OAEP
```python
def decrypt_data(private_key, ciphertext):
    plaintext = private_key.decrypt(
        ciphertext,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None,
        ),
    )
    return plaintext
```

## Key Audit and Rotation

### Inspect Key Properties
```python
def audit_key(filepath, passphrase=None):
    key = load_private_key(filepath, passphrase)
    pub = key.public_key()
    numbers = pub.public_numbers()
    return {
        "key_size": key.key_size,
        "compliant": key.key_size >= 2048,
        "recommended": key.key_size >= 4096,
        "public_exponent": numbers.e,
        "modulus_bits": numbers.n.bit_length(),
        "format": "PKCS8-PEM",
        "encrypted": passphrase is not None,
    }
```

### Check Key Strength
```python
def check_key_strength(key_path, passphrase=None):
    key = load_private_key(key_path, passphrase)
    findings = []
    if key.key_size < 2048:
        findings.append({
            "issue": f"Key size {key.key_size} bits is below minimum (2048)",
            "severity": "critical",
        })
    elif key.key_size < 4096:
        findings.append({
            "issue": f"Key size {key.key_size} bits — 4096 recommended",
            "severity": "low",
        })
    return {"key_size": key.key_size, "findings": findings}
```

## Self-Signed Certificate Generation

```python
from cryptography import x509
from cryptography.x509.oid import NameOID
from datetime import datetime, timedelta, timezone

def create_self_signed_cert(private_key, common_name, days_valid=365):
    subject = issuer = x509.Name([
        x509.NameAttribute(NameOID.COMMON_NAME, common_name),
        x509.NameAttribute(NameOID.ORGANIZATION_NAME, "Security Audit"),
    ])
    cert = (
        x509.CertificateBuilder()
        .subject_name(subject)
        .issuer_name(issuer)
        .public_key(private_key.public_key())
        .serial_number(x509.random_serial_number())
        .not_valid_before(datetime.now(timezone.utc))
        .not_valid_after(datetime.now(timezone.utc) + timedelta(days=days_valid))
        .sign(private_key, hashes.SHA256())
    )
    return cert
```

## Output Format

```json
{
  "key_path": "/etc/pki/private/server.key",
  "key_size": 4096,
  "public_exponent": 65537,
  "compliant": true,
  "encrypted": true,
  "certificate": {
    "common_name": "server.example.com",
    "not_before": "2025-01-15T00:00:00Z",
    "not_after": "2026-01-15T00:00:00Z",
    "serial_number": "ABC123..."
  },
  "findings": []
}
```

===== END FILE =====


===== FILE: skills/implementing-rsa-key-pair-management/references/standards.md =====
# Standards and References - RSA Key Pair Management

## Primary Standards

### NIST FIPS 186-5 - Digital Signature Standard (DSS)
- **URL**: https://csrc.nist.gov/publications/detail/fips/186/5/final
- **Description**: Specifies RSA digital signature generation and verification
- **RSA minimum**: 2048-bit keys

### RFC 8017 - PKCS #1: RSA Cryptography Specifications Version 2.2
- **URL**: https://www.rfc-editor.org/rfc/rfc8017
- **Description**: Defines RSA key formats, OAEP encryption, and PSS signatures
- **Key operations**: RSAEP, RSADP, RSASP1, RSAVP1

### RFC 5958 - Asymmetric Key Packages (PKCS#8 v2)
- **URL**: https://www.rfc-editor.org/rfc/rfc5958
- **Description**: Private key information syntax for storage

### RFC 7468 - Textual Encodings of PKIX, PKCS, and CMS Structures
- **URL**: https://www.rfc-editor.org/rfc/rfc7468
- **Description**: PEM encoding format specification

### NIST SP 800-57 Part 1 Rev. 5 - Key Management
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final
- **Description**: Key length recommendations and lifecycle management
- **RSA 2048**: Acceptable through 2030
- **RSA 3072+**: Recommended for beyond 2030

### NIST SP 800-131A Rev. 2 - Transitioning Cryptographic Algorithms
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-131a/rev-2/final
- **Description**: Transition guidance for algorithm selection
- **PKCS#1 v1.5 signatures**: Legacy use only
- **RSA-PSS**: Recommended for all new applications

## Python Library References

### cryptography (pyca/cryptography)
- **RSA Key Generation**: `cryptography.hazmat.primitives.asymmetric.rsa`
- **Serialization**: `cryptography.hazmat.primitives.serialization`
- **Signatures**: `cryptography.hazmat.primitives.asymmetric.padding`
- **Documentation**: https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa/

===== END FILE =====


===== FILE: skills/implementing-rsa-key-pair-management/references/workflows.md =====
# Workflows - RSA Key Pair Management

## Workflow 1: Key Pair Generation

```
[Select Key Size] (3072 or 4096 bits)
      |
[Generate RSA Key Pair]
(public_exponent=65537)
      |
[Serialize Private Key]
(PEM/PKCS#8 with AES-256-CBC passphrase)
      |
[Extract and Serialize Public Key]
(PEM/SubjectPublicKeyInfo)
      |
[Compute Key Fingerprint]
(SHA-256 of DER-encoded public key)
      |
[Store Keys with Metadata]
(key_id, creation_date, algorithm, size)
```

## Workflow 2: Digital Signature (RSA-PSS)

```
[Document/Data to Sign]
      |
[Hash Data] (SHA-256)
      |
[Load Private Key] (decrypt with passphrase)
      |
[RSA-PSS Sign]
(padding=PSS, mgf=MGF1(SHA256), salt_length=PSS.MAX_LENGTH)
      |
[Output Signature] (DER or Base64)
```

## Workflow 3: Signature Verification

```
[Document + Signature + Public Key]
      |
[Load Public Key]
      |
[RSA-PSS Verify]
(same padding parameters as signing)
      |
[Valid?]
  YES --> Accept
  NO  --> Reject (data or signature tampered)
```

## Workflow 4: Key Rotation

```
[Current Key Pair (version N)]
      |
[Generate New Key Pair (version N+1)]
      |
[Update Active Key Reference]
      |
[Archive Old Key Pair]
(mark as "decrypt/verify only")
      |
[After Grace Period: Destroy Old Private Key]
(keep public key for verification)
```

## Workflow 5: RSA Encryption (OAEP)

```
[Plaintext] (max size depends on key and padding)
      |
[Load Recipient's Public Key]
      |
[RSA-OAEP Encrypt]
(padding=OAEP, mgf=MGF1(SHA256), algorithm=SHA256)
      |
[Ciphertext]
```

===== END FILE =====


===== FILE: skills/implementing-rsa-key-pair-management/scripts/agent.py =====
#!/usr/bin/env python3
"""RSA key pair lifecycle management agent.

Generates, audits, rotates, and manages RSA key pairs using the
cryptography library. Supports key generation with configurable sizes,
PEM export with encryption, public key extraction, key strength
auditing, and expiration tracking.
"""
import argparse
import json
import os
import sys
from datetime import datetime, timezone

try:
    from cryptography.hazmat.primitives import hashes, serialization
    from cryptography.hazmat.primitives.asymmetric import rsa, padding
    from cryptography.hazmat.backends import default_backend
    from cryptography.x509 import load_pem_x509_certificate
    HAS_CRYPTO = True
except ImportError:
    HAS_CRYPTO = False


def generate_key_pair(key_size=4096, passphrase=None):
    """Generate an RSA key pair."""
    if not HAS_CRYPTO:
        print("[!] 'cryptography' required: pip install cryptography", file=sys.stderr)
        sys.exit(1)
    print(f"[*] Generating {key_size}-bit RSA key pair...")
    private_key = rsa.generate_private_key(
        public_exponent=65537,
        key_size=key_size,
        backend=default_backend(),
    )
    if passphrase:
        encryption = serialization.BestAvailableEncryption(passphrase.encode())
    else:
        encryption = serialization.NoEncryption()
    private_pem = private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=encryption,
    )
    public_key = private_key.public_key()
    public_pem = public_key.public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo,
    )
    public_ssh = public_key.public_bytes(
        encoding=serialization.Encoding.OpenSSH,
        format=serialization.PublicFormat.OpenSSH,
    )
    print(f"[+] Key pair generated ({key_size} bits)")
    return {
        "private_pem": private_pem.decode(),
        "public_pem": public_pem.decode(),
        "public_ssh": public_ssh.decode(),
        "key_size": key_size,
        "encrypted": passphrase is not None,
    }


def save_key_pair(key_data, private_path, public_path):
    """Save key pair to files with secure permissions."""
    with open(private_path, "w") as f:
        f.write(key_data["private_pem"])
    os.chmod(private_path, 0o600)
    print(f"[+] Private key saved to {private_path} (mode 0600)")
    with open(public_path, "w") as f:
        f.write(key_data["public_pem"])
    os.chmod(public_path, 0o644)
    print(f"[+] Public key saved to {public_path}")


def audit_key_file(key_path):
    """Audit an existing RSA key file for security issues."""
    if not HAS_CRYPTO:
        print("[!] 'cryptography' required", file=sys.stderr)
        sys.exit(1)
    findings = []
    if not os.path.isfile(key_path):
        findings.append({"check": "File exists", "status": "FAIL", "severity": "CRITICAL"})
        return findings
    stat = os.stat(key_path)
    mode = oct(stat.st_mode)[-3:]
    if mode not in ("600", "400"):
        findings.append({
            "check": "File permissions",
            "status": f"FAIL (mode {mode})",
            "severity": "HIGH",
            "recommendation": "Set permissions to 600: chmod 600 " + key_path,
        })
    else:
        findings.append({"check": "File permissions", "status": f"PASS (mode {mode})", "severity": "INFO"})
    with open(key_path, "rb") as f:
        pem_data = f.read()
    is_encrypted = b"ENCRYPTED" in pem_data
    try:
        if b"PRIVATE" in pem_data:
            if is_encrypted:
                findings.append({"check": "Key encryption", "status": "PASS (encrypted)", "severity": "INFO"})
                findings.append({"check": "Key type", "status": "Private key (encrypted)", "severity": "INFO"})
                return findings
            private_key = serialization.load_pem_private_key(pem_data, password=None, backend=default_backend())
            key_size = private_key.key_size
            findings.append({"check": "Key encryption", "status": "FAIL (unencrypted)", "severity": "HIGH",
                            "recommendation": "Encrypt private key with a passphrase"})
        else:
            from cryptography.hazmat.primitives.serialization import load_pem_public_key
            public_key = load_pem_public_key(pem_data, backend=default_backend())
            key_size = public_key.key_size
            findings.append({"check": "Key type", "status": "Public key", "severity": "INFO"})

        if key_size < 2048:
            findings.append({"check": "Key strength", "status": f"FAIL ({key_size} bits)", "severity": "CRITICAL",
                            "recommendation": "Minimum 2048-bit; recommend 4096-bit for new keys"})
        elif key_size < 4096:
            findings.append({"check": "Key strength", "status": f"WARN ({key_size} bits)", "severity": "MEDIUM",
                            "recommendation": "Consider upgrading to 4096-bit"})
        else:
            findings.append({"check": "Key strength", "status": f"PASS ({key_size} bits)", "severity": "INFO"})
    except Exception as e:
        findings.append({"check": "Key parsing", "status": f"FAIL: {e}", "severity": "HIGH"})
    return findings


def scan_directory_for_keys(directory, recursive=True):
    """Scan a directory for key files and audit each one."""
    key_files = []
    key_extensions = (".pem", ".key", ".pub", ".rsa", ".der")
    key_markers = (b"BEGIN RSA PRIVATE", b"BEGIN PRIVATE", b"BEGIN PUBLIC", b"BEGIN OPENSSH")
    for root, dirs, files in os.walk(directory):
        for fname in files:
            full_path = os.path.join(root, fname)
            is_key = False
            if any(fname.endswith(ext) for ext in key_extensions):
                is_key = True
            else:
                try:
                    with open(full_path, "rb") as f:
                        header = f.read(64)
                    if any(marker in header for marker in key_markers):
                        is_key = True
                except (IOError, PermissionError):
                    pass
            if is_key:
                findings = audit_key_file(full_path)
                key_files.append({"path": full_path, "findings": findings})
        if not recursive:
            break
    return key_files


def format_summary(results, action):
    """Print a human-readable summary."""
    print(f"\n{'='*60}")
    print(f"  RSA Key Management Report")
    print(f"{'='*60}")
    print(f"  Action    : {action}")
    if action == "audit" and isinstance(results, list):
        total_keys = len(results)
        critical = sum(1 for r in results for f in r.get("findings", []) if f.get("severity") == "CRITICAL")
        high = sum(1 for r in results for f in r.get("findings", []) if f.get("severity") == "HIGH")
        print(f"  Keys Found: {total_keys}")
        print(f"  Critical  : {critical}")
        print(f"  High      : {high}")
        for r in results:
            print(f"\n    Key: {r['path']}")
            for f in r.get("findings", []):
                print(f"      [{f['severity']:8s}] {f['check']}: {f['status']}")


def main():
    parser = argparse.ArgumentParser(description="RSA key pair lifecycle management agent")
    sub = parser.add_subparsers(dest="command")

    p_gen = sub.add_parser("generate", help="Generate new RSA key pair")
    p_gen.add_argument("--key-size", type=int, default=4096, choices=[2048, 3072, 4096],
                       help="RSA key size in bits (default: 4096)")
    p_gen.add_argument("--passphrase", help="Passphrase to encrypt private key")
    p_gen.add_argument("--private-key", default="id_rsa", help="Private key output path")
    p_gen.add_argument("--public-key", default="id_rsa.pub", help="Public key output path")

    p_audit = sub.add_parser("audit", help="Audit existing key files")
    p_audit.add_argument("--path", required=True, help="Key file or directory to audit")
    p_audit.add_argument("--recursive", action="store_true", help="Scan directory recursively")

    parser.add_argument("--output", "-o", help="Output JSON report path")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    if not args.command:
        parser.print_help()
        sys.exit(1)

    if args.command == "generate":
        key_data = generate_key_pair(args.key_size, args.passphrase)
        save_key_pair(key_data, args.private_key, args.public_key)
        result = {"action": "generate", "key_size": args.key_size,
                  "private_key": args.private_key, "public_key": args.public_key,
                  "encrypted": key_data["encrypted"]}
    elif args.command == "audit":
        if os.path.isdir(args.path):
            results = scan_directory_for_keys(args.path, args.recursive)
        else:
            findings = audit_key_file(args.path)
            results = [{"path": args.path, "findings": findings}]
        format_summary(results, "audit")
        result = {"action": "audit", "keys_audited": results}

    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "RSA Key Manager",
        "result": result,
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


===== FILE: skills/implementing-rsa-key-pair-management/scripts/process.py =====
#!/usr/bin/env python3
"""
RSA Key Pair Management Tool

Implements RSA key generation, serialization, signing, verification,
encryption, and key rotation using the cryptography library.

Requirements:
    pip install cryptography

Usage:
    python process.py generate --size 4096 --output ./keys --passphrase "MyKeyPass"
    python process.py sign --key ./keys/private.pem --input document.pdf --passphrase "MyKeyPass"
    python process.py verify --key ./keys/public.pem --input document.pdf --signature document.pdf.sig
    python process.py info --key ./keys/public.pem
    python process.py rotate --keystore ./keys --passphrase "MyKeyPass"
"""

import os
import sys
import json
import hashlib
import argparse
import logging
import datetime
from pathlib import Path
from typing import Dict, Optional, Tuple

from cryptography.hazmat.primitives.asymmetric import rsa, padding, utils
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.backends import default_backend
from cryptography.exceptions import InvalidSignature

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

RECOMMENDED_KEY_SIZE = 4096
PUBLIC_EXPONENT = 65537


def generate_rsa_keypair(
    key_size: int = RECOMMENDED_KEY_SIZE,
    passphrase: Optional[str] = None,
) -> Tuple[bytes, bytes, Dict]:
    """
    Generate an RSA key pair.

    Returns:
        Tuple of (private_key_pem, public_key_pem, metadata)
    """
    if key_size < 2048:
        raise ValueError("Key size must be at least 2048 bits (3072+ recommended)")

    private_key = rsa.generate_private_key(
        public_exponent=PUBLIC_EXPONENT,
        key_size=key_size,
        backend=default_backend(),
    )

    if passphrase:
        encryption = serialization.BestAvailableEncryption(passphrase.encode())
    else:
        encryption = serialization.NoEncryption()

    private_pem = private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=encryption,
    )

    public_key = private_key.public_key()
    public_pem = public_key.public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo,
    )

    # Compute fingerprint (SHA-256 of DER-encoded public key)
    public_der = public_key.public_bytes(
        encoding=serialization.Encoding.DER,
        format=serialization.PublicFormat.SubjectPublicKeyInfo,
    )
    fingerprint = hashlib.sha256(public_der).hexdigest()

    metadata = {
        "algorithm": "RSA",
        "key_size": key_size,
        "public_exponent": PUBLIC_EXPONENT,
        "fingerprint_sha256": fingerprint,
        "created_at": datetime.datetime.utcnow().isoformat() + "Z",
        "passphrase_protected": passphrase is not None,
        "format": "PKCS#8 PEM",
    }

    return private_pem, public_pem, metadata


def save_keypair(
    output_dir: str,
    private_pem: bytes,
    public_pem: bytes,
    metadata: Dict,
    version: int = 1,
) -> Dict:
    """Save key pair to files with metadata."""
    output_path = Path(output_dir)
    output_path.mkdir(parents=True, exist_ok=True)

    prefix = f"v{version}_" if version > 1 else ""

    private_path = output_path / f"{prefix}private.pem"
    public_path = output_path / f"{prefix}public.pem"
    meta_path = output_path / f"{prefix}key_metadata.json"

    private_path.write_bytes(private_pem)
    public_path.write_bytes(public_pem)

    metadata["version"] = version
    metadata["private_key_path"] = str(private_path)
    metadata["public_key_path"] = str(public_path)

    meta_path.write_text(json.dumps(metadata, indent=2))

    logger.info(f"Key pair saved to {output_dir} (version {version})")

    return metadata


def load_private_key(key_path: str, passphrase: Optional[str] = None):
    """Load an RSA private key from PEM file."""
    key_data = Path(key_path).read_bytes()
    pwd = passphrase.encode() if passphrase else None
    return serialization.load_pem_private_key(key_data, password=pwd, backend=default_backend())


def load_public_key(key_path: str):
    """Load an RSA public key from PEM file."""
    key_data = Path(key_path).read_bytes()
    return serialization.load_pem_public_key(key_data, backend=default_backend())


def sign_data(data: bytes, private_key) -> bytes:
    """Sign data using RSA-PSS with SHA-256."""
    signature = private_key.sign(
        data,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH,
        ),
        hashes.SHA256(),
    )
    return signature


def verify_signature(data: bytes, signature: bytes, public_key) -> bool:
    """Verify RSA-PSS signature."""
    try:
        public_key.verify(
            signature,
            data,
            padding.PSS(
                mgf=padding.MGF1(hashes.SHA256()),
                salt_length=padding.PSS.MAX_LENGTH,
            ),
            hashes.SHA256(),
        )
        return True
    except InvalidSignature:
        return False


def encrypt_data(plaintext: bytes, public_key) -> bytes:
    """Encrypt data using RSA-OAEP."""
    max_size = (public_key.key_size // 8) - 2 * 32 - 2  # OAEP with SHA-256
    if len(plaintext) > max_size:
        raise ValueError(
            f"Plaintext too large for RSA-OAEP ({len(plaintext)} bytes, max {max_size}). "
            "Use envelope encryption for large data."
        )
    return public_key.encrypt(
        plaintext,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None,
        ),
    )


def decrypt_data(ciphertext: bytes, private_key) -> bytes:
    """Decrypt RSA-OAEP encrypted data."""
    return private_key.decrypt(
        ciphertext,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None,
        ),
    )


def get_key_info(key_path: str, passphrase: Optional[str] = None) -> Dict:
    """Get information about an RSA key."""
    key_data = Path(key_path).read_bytes()

    try:
        pwd = passphrase.encode() if passphrase else None
        key = serialization.load_pem_private_key(key_data, password=pwd, backend=default_backend())
        key_type = "private"
        public_key = key.public_key()
    except (ValueError, TypeError):
        key = serialization.load_pem_public_key(key_data, backend=default_backend())
        key_type = "public"
        public_key = key

    public_der = public_key.public_bytes(
        encoding=serialization.Encoding.DER,
        format=serialization.PublicFormat.SubjectPublicKeyInfo,
    )
    fingerprint = hashlib.sha256(public_der).hexdigest()

    numbers = public_key.public_numbers()

    info = {
        "key_type": key_type,
        "algorithm": "RSA",
        "key_size": public_key.key_size,
        "public_exponent": numbers.e,
        "fingerprint_sha256": fingerprint,
        "modulus_hex_prefix": hex(numbers.n)[:32] + "...",
    }

    if public_key.key_size < 2048:
        info["warning"] = "Key size below 2048 bits is considered insecure"
    elif public_key.key_size < 3072:
        info["note"] = "Key size below 3072 bits; consider upgrading for post-2030 use"

    return info


def rotate_keys(keystore_dir: str, passphrase: Optional[str] = None) -> Dict:
    """Rotate RSA key pair, archiving the old one."""
    keystore = Path(keystore_dir)

    # Find current version
    version = 1
    meta_files = sorted(keystore.glob("*key_metadata.json"))
    if meta_files:
        for mf in meta_files:
            meta = json.loads(mf.read_text())
            v = meta.get("version", 1)
            if v >= version:
                version = v + 1

    # Generate new key pair
    private_pem, public_pem, metadata = generate_rsa_keypair(
        key_size=RECOMMENDED_KEY_SIZE, passphrase=passphrase
    )

    result = save_keypair(keystore_dir, private_pem, public_pem, metadata, version=version)

    # Update current key symlink info
    current_meta = {
        "current_version": version,
        "current_fingerprint": metadata["fingerprint_sha256"],
        "rotated_at": datetime.datetime.utcnow().isoformat() + "Z",
        "all_versions": list(range(1, version + 1)),
    }
    (keystore / "current.json").write_text(json.dumps(current_meta, indent=2))

    logger.info(f"Key rotated to version {version}")
    return result


def sign_file(key_path: str, input_path: str, passphrase: Optional[str] = None) -> str:
    """Sign a file and save the signature."""
    private_key = load_private_key(key_path, passphrase)
    data = Path(input_path).read_bytes()
    signature = sign_data(data, private_key)

    sig_path = input_path + ".sig"
    Path(sig_path).write_bytes(signature)
    logger.info(f"Signature saved to {sig_path}")
    return sig_path


def verify_file(key_path: str, input_path: str, sig_path: str) -> bool:
    """Verify a file's signature."""
    public_key = load_public_key(key_path)
    data = Path(input_path).read_bytes()
    signature = Path(sig_path).read_bytes()
    valid = verify_signature(data, signature, public_key)
    logger.info(f"Signature verification: {'VALID' if valid else 'INVALID'}")
    return valid


def main():
    parser = argparse.ArgumentParser(description="RSA Key Pair Management Tool")
    subparsers = parser.add_subparsers(dest="command")

    gen = subparsers.add_parser("generate", help="Generate RSA key pair")
    gen.add_argument("--size", type=int, default=4096, help="Key size in bits")
    gen.add_argument("--output", "-o", default="./keys", help="Output directory")
    gen.add_argument("--passphrase", "-p", help="Passphrase for private key")

    sig = subparsers.add_parser("sign", help="Sign a file")
    sig.add_argument("--key", required=True, help="Private key path")
    sig.add_argument("--input", "-i", required=True, help="File to sign")
    sig.add_argument("--passphrase", "-p", help="Key passphrase")

    ver = subparsers.add_parser("verify", help="Verify a signature")
    ver.add_argument("--key", required=True, help="Public key path")
    ver.add_argument("--input", "-i", required=True, help="Original file")
    ver.add_argument("--signature", "-s", required=True, help="Signature file")

    info = subparsers.add_parser("info", help="Show key information")
    info.add_argument("--key", required=True, help="Key file path")
    info.add_argument("--passphrase", "-p", help="Key passphrase")

    rot = subparsers.add_parser("rotate", help="Rotate key pair")
    rot.add_argument("--keystore", required=True, help="Keystore directory")
    rot.add_argument("--passphrase", "-p", help="Passphrase for new key")

    args = parser.parse_args()

    if args.command == "generate":
        priv, pub, meta = generate_rsa_keypair(args.size, args.passphrase)
        result = save_keypair(args.output, priv, pub, meta)
        print(json.dumps(result, indent=2))
    elif args.command == "sign":
        sig_path = sign_file(args.key, args.input, args.passphrase)
        print(json.dumps({"signature_file": sig_path}))
    elif args.command == "verify":
        valid = verify_file(args.key, args.input, args.signature)
        print(json.dumps({"valid": valid}))
        if not valid:
            sys.exit(1)
    elif args.command == "info":
        result = get_key_info(args.key, args.passphrase)
        print(json.dumps(result, indent=2))
    elif args.command == "rotate":
        result = rotate_keys(args.keystore, args.passphrase)
        print(json.dumps(result, indent=2))
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-zero-knowledge-proof-for-authentication/SKILL.md =====
---
name: implementing-zero-knowledge-proof-for-authentication
description: Implements the Schnorr identification protocol and a simplified Zero-Knowledge Password Proof (ZKPP) over the discrete logarithm problem, letting a prover authenticate by demonstrating knowledge of a secret without ever revealing it to the server. Use when designing or building password-less or password-secret-free authentication, or when a server must verify a user's credential without learning or storing the underlying secret.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- zero-knowledge-proof
- authentication
- privacy
- zkp
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
---
# Implementing Zero-Knowledge Proof for Authentication

## Overview

Zero-Knowledge Proofs (ZKPs) allow a prover to demonstrate knowledge of a secret (such as a password or private key) without revealing the secret itself. This skill implements the Schnorr identification protocol and a simplified ZKPP (Zero-Knowledge Password Proof) using the discrete logarithm problem, enabling authentication where the server never learns the user's password.


## When to Use

- When deploying or configuring implementing zero knowledge proof for authentication capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Implement Schnorr's identification protocol for ZKP authentication
- Build a non-interactive ZKP using Fiat-Shamir heuristic
- Implement zero-knowledge password proof (ZKPP)
- Demonstrate completeness, soundness, and zero-knowledge properties
- Compare ZKP authentication with traditional password verification

## Key Concepts

### ZKP Properties

| Property | Description |
|----------|------------|
| Completeness | Honest prover always convinces honest verifier |
| Soundness | Dishonest prover cannot convince verifier (except negligible probability) |
| Zero-Knowledge | Verifier learns nothing beyond the statement's truth |

### Schnorr Protocol

1. **Setup**: Public generator g, prime p, q (order of g)
2. **Registration**: Prover computes y = g^x mod p (public key from secret x)
3. **Commitment**: Prover sends t = g^r mod p (random r)
4. **Challenge**: Verifier sends random c
5. **Response**: Prover sends s = r + c*x mod q
6. **Verify**: Check g^s == t * y^c mod p

## Security Considerations

- Use cryptographically secure random number generators
- Challenge must be unpredictable (from verifier's perspective)
- For non-interactive proofs, use Fiat-Shamir with collision-resistant hash
- ZKP alone does not provide forward secrecy; combine with TLS

## Validation Criteria

- [ ] Honest prover always verifies successfully (completeness)
- [ ] Random response without secret does not verify (soundness)
- [ ] Server never receives the secret value
- [ ] Non-interactive proof is verifiable offline
- [ ] Multiple authentications produce different transcripts
- [ ] Protocol resists replay attacks

===== END FILE =====


===== FILE: skills/implementing-zero-knowledge-proof-for-authentication/LICENSE =====

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


===== FILE: skills/implementing-zero-knowledge-proof-for-authentication/references/api-reference.md =====
# API Reference: Zero-Knowledge Proof Authentication

## hashlib (Python Standard Library)

### PBKDF2 Key Derivation
```python
import hashlib
key = hashlib.pbkdf2_hmac("sha256", password.encode(), salt.encode(), iterations)
```

### SHA-256 Hashing (Fiat-Shamir Heuristic)
```python
challenge = int(hashlib.sha256(data.encode()).hexdigest(), 16) % prime
```

## secrets (Python Standard Library)

| Function | Description |
|----------|-------------|
| `secrets.randbelow(n)` | Cryptographically secure random int in [0, n) |
| `secrets.token_hex(n)` | Random hex string of n bytes |
| `secrets.token_bytes(n)` | Random bytes of length n |

## Schnorr Protocol Steps

| Step | Prover | Verifier |
|------|--------|----------|
| Setup | Private key x, public key y=g^x mod p | Knows g, p, y |
| Commit | Pick random k, send r=g^k mod p | Receive r |
| Challenge | - | Send random c |
| Response | Send s = k - c*x mod (p-1) | Check g^s * y^c == r mod p |

## Fiat-Shamir Heuristic (Non-Interactive)
```
c = H(g || r || y)   # Challenge derived from hash
s = k - c * x mod (p-1)
```

## ZKP Properties
| Property | Guarantee |
|----------|-----------|
| Completeness | Honest prover always convinces verifier |
| Soundness | Dishonest prover fails with high probability |
| Zero-Knowledge | Verifier learns nothing beyond validity |

## References
- Schnorr Protocol: https://en.wikipedia.org/wiki/Schnorr_identification
- RFC 8235 (Schnorr NIZK): https://www.rfc-editor.org/rfc/rfc8235
- hashlib docs: https://docs.python.org/3/library/hashlib.html
- secrets docs: https://docs.python.org/3/library/secrets.html

===== END FILE =====


===== FILE: skills/implementing-zero-knowledge-proof-for-authentication/references/standards.md =====
# Standards and References - Zero-Knowledge Proof for Authentication

## Academic References

### Schnorr Identification Protocol
- **Paper**: "Efficient Signature Generation by Smart Cards" (Claus-Peter Schnorr, 1989)
- **Standard**: ISO/IEC 9798-5 (Entity authentication using zero-knowledge techniques)

### Fiat-Shamir Heuristic
- **Paper**: "How To Prove Yourself" (Fiat, Shamir, 1986)
- **Description**: Converts interactive ZKP to non-interactive using hash function

### RFC 8235 - Schnorr Non-Interactive Zero-Knowledge Proof
- **URL**: https://www.rfc-editor.org/rfc/rfc8235
- **Description**: Standardized Schnorr NIZKP

### RFC 5054 - SRP (Secure Remote Password)
- **URL**: https://www.rfc-editor.org/rfc/rfc5054
- **Description**: Zero-knowledge password authentication protocol

## Python Libraries

### py-ecc
- **URL**: https://github.com/ethereum/py_ecc
- **Description**: Elliptic curve operations for ZKPs

### cryptography
- **URL**: https://cryptography.io/
- **Description**: Hash functions, modular arithmetic support

===== END FILE =====


===== FILE: skills/implementing-zero-knowledge-proof-for-authentication/references/workflows.md =====
# Workflows - Zero-Knowledge Proof for Authentication

## Workflow 1: Schnorr Interactive ZKP

```
Prover (knows secret x)              Verifier (knows y = g^x mod p)
      |                                      |
      |-- Commitment: t = g^r mod p -------->|
      |                                      |
      |<-- Challenge: c (random) ------------|
      |                                      |
      |-- Response: s = (r + c*x) mod q ---->|
      |                                      |
      |                              [Verify: g^s == t * y^c mod p]
      |                              [Accept or Reject]
```

## Workflow 2: Non-Interactive ZKP (Fiat-Shamir)

```
Prover:
  1. Choose random r
  2. Compute t = g^r mod p
  3. Compute c = H(g || y || t)  (Fiat-Shamir)
  4. Compute s = (r + c*x) mod q
  5. Send proof (t, s) to verifier

Verifier:
  1. Compute c = H(g || y || t)
  2. Check g^s == t * y^c mod p
```

## Workflow 3: Registration and Authentication

```
[Registration]:
  User --> [Choose password/secret x]
       --> [Compute y = g^x mod p]
       --> [Send y to server]
  Server --> [Store y (public key only)]

[Authentication]:
  User <--> Server: [Run Schnorr protocol]
  Server: [Verifies proof without learning x]
  Server: [Grants session token on success]
```

===== END FILE =====


===== FILE: skills/implementing-zero-knowledge-proof-for-authentication/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for implementing zero-knowledge proof authentication using Schnorr protocol."""

import hashlib
import secrets
import json
import argparse
from datetime import datetime


# Safe prime and generator for discrete log ZKP
SAFE_PRIME = 0xFFFFFFFFFFFFFFFFC90FDAA22168C234C4C6628B80DC1CD129024E088A67CC74020BBEA63B139B22514A08798E3404DDEF9519B3CD3A431B302B0A6DF25F14374FE1356D6D51C245E485B576625E7EC6F44C42E9A637ED6B0BFF5CB6F406B7EDEE386BFB5A899FA5AE9F24117C4B1FE649286651ECE45B3DC2007CB8A163BF0598DA48361C55D39A69163FA8FD24CF5F83655D23DCA3AD961C62F356208552BB9ED529077096966D670C354E4ABC9804F1746C08CA18217C32905E462E36CE3BE39E772C180E86039B2783A2EC07A28FB5C55DF06F4C52C9DE2BCBF6955817183995497CEA956AE515D2261898FA051015728E5A8AACAA68FFFFFFFFFFFFFFFF
GENERATOR = 2


def generate_keypair():
    """Generate a ZKP key pair (private key x, public key y = g^x mod p)."""
    x = secrets.randbelow(SAFE_PRIME - 2) + 1
    y = pow(GENERATOR, x, SAFE_PRIME)
    print(f"[*] Generated key pair")
    print(f"  Public key (y): {hex(y)[:40]}...")
    return x, y


def schnorr_prove(private_key):
    """Generate a Schnorr ZKP proof (commitment, challenge, response)."""
    k = secrets.randbelow(SAFE_PRIME - 2) + 1
    r = pow(GENERATOR, k, SAFE_PRIME)
    # Fiat-Shamir heuristic: non-interactive challenge
    c_input = f"{GENERATOR}{r}{pow(GENERATOR, private_key, SAFE_PRIME)}"
    c = int(hashlib.sha256(c_input.encode()).hexdigest(), 16) % SAFE_PRIME
    s = (k - c * private_key) % (SAFE_PRIME - 1)
    return {"commitment": r, "challenge": c, "response": s}


def schnorr_verify(public_key, proof):
    """Verify a Schnorr ZKP proof without learning the private key."""
    r, c, s = proof["commitment"], proof["challenge"], proof["response"]
    lhs = pow(GENERATOR, s, SAFE_PRIME) * pow(public_key, c, SAFE_PRIME) % SAFE_PRIME
    valid = lhs == r
    print(f"  [{'+'if valid else '!'}] Verification: {'PASSED' if valid else 'FAILED'}")
    return valid


def zkp_password_register(password):
    """Register a password using ZKP (server stores only public key)."""
    salt = secrets.token_hex(16)
    pwd_hash = hashlib.pbkdf2_hmac("sha256", password.encode(), salt.encode(), 100000)
    x = int.from_bytes(pwd_hash, "big") % (SAFE_PRIME - 2) + 1
    y = pow(GENERATOR, x, SAFE_PRIME)
    print(f"[*] Registered user (server stores salt + public key, never the password)")
    return {"salt": salt, "public_key": y, "private_key": x}


def zkp_password_authenticate(password, registration):
    """Authenticate using ZKP (prove password knowledge without revealing it)."""
    salt = registration["salt"]
    pwd_hash = hashlib.pbkdf2_hmac("sha256", password.encode(), salt.encode(), 100000)
    x = int.from_bytes(pwd_hash, "big") % (SAFE_PRIME - 2) + 1
    proof = schnorr_prove(x)
    valid = schnorr_verify(registration["public_key"], proof)
    return valid


def run_protocol_demo(rounds=5):
    """Demonstrate ZKP authentication protocol with multiple rounds."""
    print("[*] ZKP Schnorr Protocol Demo\n")
    x, y = generate_keypair()
    successes = 0
    for i in range(rounds):
        print(f"\n[*] Round {i+1}/{rounds}")
        proof = schnorr_prove(x)
        if schnorr_verify(y, proof):
            successes += 1
    print(f"\n[*] Protocol: {successes}/{rounds} rounds passed")
    print(f"[*] Completeness: {'VERIFIED' if successes == rounds else 'FAILED'}")
    # Soundness test: wrong key should fail
    wrong_x = secrets.randbelow(SAFE_PRIME - 2) + 1
    wrong_proof = schnorr_prove(wrong_x)
    forgery = schnorr_verify(y, wrong_proof)
    print(f"[*] Soundness (wrong key rejected): {'VERIFIED' if not forgery else 'FAILED'}")
    return successes == rounds and not forgery


def run_password_demo(password="SecureP@ss123"):
    """Demonstrate ZKP password authentication."""
    print("\n[*] ZKP Password Authentication Demo\n")
    reg = zkp_password_register(password)
    print("\n[*] Authenticating with correct password...")
    ok = zkp_password_authenticate(password, reg)
    print(f"  Result: {'Authenticated' if ok else 'Rejected'}")
    print("\n[*] Authenticating with wrong password...")
    bad = zkp_password_authenticate("WrongPassword", reg)
    print(f"  Result: {'Authenticated' if bad else 'Rejected'}")
    return ok and not bad


def main():
    parser = argparse.ArgumentParser(description="Zero-Knowledge Proof Authentication Agent")
    parser.add_argument("action", choices=["demo-protocol", "demo-password", "keygen", "full-test"])
    parser.add_argument("--rounds", type=int, default=5, help="Protocol verification rounds")
    parser.add_argument("--password", default="SecureP@ss123", help="Password for ZKP demo")
    parser.add_argument("-o", "--output", default="zkp_report.json")
    args = parser.parse_args()

    report = {"date": datetime.now().isoformat(), "action": args.action}
    if args.action == "keygen":
        x, y = generate_keypair()
        report["public_key"] = hex(y)
    elif args.action == "demo-protocol":
        report["protocol_valid"] = run_protocol_demo(args.rounds)
    elif args.action == "demo-password":
        report["password_auth_valid"] = run_password_demo(args.password)
    elif args.action == "full-test":
        report["protocol_valid"] = run_protocol_demo(args.rounds)
        report["password_auth_valid"] = run_password_demo(args.password)

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\n[*] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/migrating-to-post-quantum-cryptography/SKILL.md =====
---
name: migrating-to-post-quantum-cryptography
description: Build a cryptographic inventory/CBOM with OpenSSL 3.5+, deploy hybrid post-quantum key exchange (X25519MLKEM768) on TLS/VPN/SSH endpoints, generate ML-KEM/ML-DSA keys and PQC/hybrid certificates, and prioritize migration by harvest-now-decrypt-later (HNDL) exposure per NIST SP 1800-38. Use when inventorying enterprise cryptography for quantum-readiness, enabling hybrid PQC key exchange, or issuing and verifying PQC/hybrid certificates.
domain: cybersecurity
subdomain: cryptography
tags:
- post-quantum
- cryptography
- ml-kem
- ml-dsa
- crypto-agility
- cbom
- tls
- quantum-readiness
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-02
mitre_attack:
- T1573
---
# Migrating to Post-Quantum Cryptography

> **Scope and Authorization:** This skill describes defensive cryptographic-migration engineering on systems you own or operate. Cryptographic discovery scanning can touch sensitive key material and production traffic — run inventory tooling only with authorization and in line with your organization's change-management and data-handling policies.

## Overview

A cryptographically relevant quantum computer (CRQC) running Shor's algorithm will break the public-key cryptography that secures almost all of today's communications and signatures: RSA, finite-field and elliptic-curve Diffie-Hellman (DH/ECDH), and ECDSA. Symmetric primitives (AES) and hashes (SHA-2/3) are only weakened (Grover gives a quadratic speedup, mitigated by larger key/output sizes), but asymmetric algorithms are catastrophically broken. The most urgent threat is **harvest-now, decrypt-later (HNDL)**: adversaries capturing encrypted traffic today to decrypt once a CRQC exists, which puts long-lived secrets (health records, state secrets, intellectual property, root-of-trust keys) at risk *now*.

On 13 August 2024 NIST finalized the first post-quantum standards: **FIPS 203 (ML-KEM**, Module-Lattice KEM, formerly CRYSTALS-Kyber) for key establishment, **FIPS 204 (ML-DSA**, Module-Lattice digital signatures, formerly CRYSTALS-Dilithium), and **FIPS 205 (SLH-DSA**, the stateless hash-based signature scheme SPHINCS+). The migration playbook (NIST SP 1800-38, *Migration to Post-Quantum Cryptography*) is: (1) build a **cryptographic inventory / CBOM**, (2) prioritize by HNDL exposure and crypto-agility, (3) deploy **hybrid** schemes (a classical algorithm AND a PQC algorithm combined, e.g. `X25519MLKEM768`) so a break in either leg does not compromise the session, and (4) re-key and rotate.

This skill maps to ATT&CK **T1573 – Encrypted Channel**: the same cryptographic channels adversaries abuse for stealthy C2 are the channels defenders must make quantum-resistant; understanding the algorithms in use is foundational to both attack detection and defensive migration. The NIST CSF outcome is **PR.DS-02 (data-in-transit protection)** — and by extension data-at-rest for HNDL-sensitive stores.

## When to Use

- When building an enterprise cryptographic inventory / Cryptography Bill of Materials (CBOM) for quantum-readiness.
- When prioritizing which systems must migrate first based on data lifetime and HNDL exposure.
- When enabling hybrid post-quantum key exchange (`X25519MLKEM768`) on TLS endpoints, VPNs, or SSH.
- When issuing PQC or hybrid certificates and testing PQC signature verification.
- When evaluating crypto-agility — the ability to swap algorithms without re-architecting applications.

## Prerequisites

- OpenSSL **3.5.0 or later**, which ships native ML-KEM, ML-DSA, and SLH-DSA support:
  ```bash
  openssl version            # expect 3.5.0+
  openssl list -kem-algorithms | grep -i mlkem
  openssl list -signature-algorithms | grep -i mldsa
  ```
- For OpenSSL 3.0–3.4, the Open Quantum Safe **oqs-provider** plus **liboqs**:
  ```bash
  git clone https://github.com/open-quantum-safe/liboqs && \
    cmake -S liboqs -B liboqs/build && cmake --build liboqs/build && \
    sudo cmake --install liboqs/build
  git clone https://github.com/open-quantum-safe/oqs-provider && \
    cmake -S oqs-provider -B oqs-provider/_build && \
    cmake --build oqs-provider/_build && \
    sudo cmake --install oqs-provider/_build
  ```
- Python 3.8+ for the inventory helper:
  ```bash
  python3 -m pip install cryptography
  ```
- (Optional) A CBOM generator: CycloneDX `cdxgen`, or `cbomkit-theia` for container/directory crypto discovery.

## Objectives

- Produce a cryptographic inventory (CBOM) of algorithms, key sizes, certificates, and protocols in use.
- Classify assets by quantum vulnerability and HNDL exposure and prioritize migration.
- Stand up and verify hybrid `X25519MLKEM768` key exchange on a TLS endpoint.
- Generate ML-KEM and ML-DSA keys and a PQC/hybrid certificate, and verify signatures.
- Establish a crypto-agility baseline and a re-keying / rotation plan.

## MITRE ATT&CK Mapping

| ID | Official Technique Name | Relevance |
|----|------------------------|-----------|
| T1573 | Encrypted Channel | Migration secures the encrypted channels (TLS/VPN/SSH) that protect data in transit; cryptographic inventory of these channels also underpins detection of adversary-controlled encrypted C2. |
| T1573.002 | Encrypted Channel: Asymmetric Cryptography | RSA/ECDH key exchange — the exact asymmetric primitives broken by a CRQC and replaced by ML-KEM hybrids. |
| T1573.001 | Encrypted Channel: Symmetric Cryptography | AES and other symmetric ciphers; quantum-weakened by Grover, mitigated by 256-bit keys rather than replacement. |

## Workflow

### 1. Confirm PQC algorithm availability
```bash
openssl version
# List quantum-safe KEMs and signatures available in this OpenSSL build
openssl list -kem-algorithms | grep -Ei 'mlkem|kyber'
openssl list -signature-algorithms | grep -Ei 'mldsa|dilithium|slhdsa|sphincs'
openssl list -tls-groups 2>/dev/null | grep -Ei 'mlkem'
```
If using oqs-provider on OpenSSL 3.0–3.4, activate it in `openssl.cnf`:
```ini
[provider_sect]
default = default_sect
oqsprovider = oqsprovider_sect
[default_sect]
activate = 1
[oqsprovider_sect]
activate = 1
```

### 2. Build a cryptographic inventory (CBOM)
Generate a CycloneDX CBOM from a code repo or container with `cbomkit-theia` / `cdxgen`:
```bash
# Directory / container image crypto discovery
cbomkit-theia dir ./myapp --output cbom.json
# or with cdxgen (Java keystores, certs, source-level algorithms)
cdxgen -t java --include-crypto -o cbom.json ./myapp
```
Enumerate TLS algorithms and certificate signature schemes across live endpoints with the helper `agent.py scan` (below), and the public-key strength of any certificate:
```bash
openssl x509 -in server.crt -noout -text | grep -E 'Signature Algorithm|Public Key'
```

### 3. Classify and prioritize by HNDL exposure
For each inventoried asset, record: algorithm, key size, where the key lives, data sensitivity, and **data lifetime**. Prioritize migration where `data_lifetime_years + migration_time > years_until_CRQC` (Mosca's inequality). Long-lived confidential data over public networks ranks highest; ephemeral internal traffic ranks lower. Hash-based signature roots-of-trust (firmware signing) are also high priority because they protect long-lived trust anchors.

### 4. Generate ML-KEM and ML-DSA key material
```bash
# ML-KEM-768 (key establishment) keypair
openssl genpkey -algorithm ML-KEM-768 -out mlkem768.key
# OpenSSL 3.0-3.4 + oqs-provider uses lowercase 'mlkem768'
# openssl genpkey -algorithm mlkem768 -out mlkem768.key

# ML-DSA-65 (signature) keypair
openssl genpkey -algorithm ML-DSA-65 -out mldsa65.key
openssl pkey -in mldsa65.key -pubout -out mldsa65.pub
```

### 5. Issue a PQC (ML-DSA) certificate
```bash
# Self-signed ML-DSA-65 certificate for testing
openssl req -new -x509 -key mldsa65.key -out mldsa65.crt -days 365 \
  -subj "/CN=pqc-test.example.com"
openssl x509 -in mldsa65.crt -noout -text | grep -A1 'Signature Algorithm'
```

### 6. Sign and verify with ML-DSA
```bash
echo "firmware-image-v2.bin" > artifact.txt
openssl dgst -sign mldsa65.key -out artifact.sig artifact.txt
openssl dgst -verify mldsa65.pub -signature artifact.sig artifact.txt
# -> "Verified OK"
```

### 7. Deploy and test hybrid TLS key exchange
Run a TLS 1.3 server and force the hybrid group `X25519MLKEM768` (classical X25519 + ML-KEM-768):
```bash
# Server (use a classical or ML-DSA cert/key)
openssl s_server -accept 4433 -www -tls1_3 \
  -cert mldsa65.crt -key mldsa65.key -groups X25519MLKEM768

# Client — negotiate the hybrid group and confirm it was used
openssl s_client -connect localhost:4433 -tls1_3 -groups X25519MLKEM768 \
  </dev/null 2>/dev/null | grep -E 'Negotiated|Server Temp Key|Cipher'
```
For external endpoints, confirm support against a public PQC test server:
```bash
openssl s_client -groups X25519MLKEM768 -tls1_3 -connect pq.cloudflareresearch.com:443 </dev/null
```

### 8. Enable hybrid PQC on production TLS terminators
Configure the web server / load balancer to offer the hybrid group while keeping classical fallback for old clients. NGINX with OpenSSL 3.5+:
```nginx
server {
    listen 443 ssl;
    ssl_protocols TLSv1.3;
    ssl_ecdh_curve X25519MLKEM768:X25519:secp256r1;   # hybrid first, classical fallback
    ssl_certificate     /etc/nginx/certs/server.crt;
    ssl_certificate_key /etc/nginx/certs/server.key;
}
```
Reload and verify with the s_client command from step 7 against the live host.

### 9. Establish crypto-agility and a rotation plan
Centralize algorithm selection (config, not code), record key/cert expiry, and schedule re-keying. Re-run the inventory (step 2) on a cadence to confirm no quantum-vulnerable-only algorithms remain on prioritized assets, and track residual RSA/ECDH usage to zero on high-HNDL paths.

## Tools and Resources

| Tool / Resource | Purpose | Link |
|-----------------|---------|------|
| FIPS 203 (ML-KEM) | KEM standard | https://csrc.nist.gov/pubs/fips/203/final |
| FIPS 204 (ML-DSA) | Signature standard | https://csrc.nist.gov/pubs/fips/204/final |
| FIPS 205 (SLH-DSA) | Hash-based signature standard | https://csrc.nist.gov/pubs/fips/205/final |
| NIST SP 1800-38 | Migration practice guide / crypto discovery | https://www.nccoe.nist.gov/crypto-agility-considerations-migrating-post-quantum-cryptographic-algorithms |
| OpenSSL 3.5 | Native ML-KEM/ML-DSA/SLH-DSA + hybrid groups | https://www.openssl.org |
| oqs-provider / liboqs | PQC for OpenSSL 3.0–3.4 | https://github.com/open-quantum-safe/oqs-provider |
| CycloneDX CBOM | Cryptography Bill of Materials spec | https://cyclonedx.org/capabilities/cbom/ |
| CBOMkit / cbomkit-theia | Crypto discovery & CBOM generation | https://github.com/cbomkit/cbomkit-theia |

## Algorithm Reference

| Classical (broken/weakened) | Quantum-safe replacement | Standard | Use |
|-----------------------------|--------------------------|----------|-----|
| RSA / ECDH / DH key exchange | ML-KEM-512/768/1024 (hybrid: X25519MLKEM768) | FIPS 203 | Key establishment |
| RSA / ECDSA / EdDSA signatures | ML-DSA-44/65/87 | FIPS 204 | General signatures |
| (backup signature) | SLH-DSA (SPHINCS+) | FIPS 205 | Conservative/firmware signing |
| AES-128 | AES-256 | FIPS 197 | Symmetric (Grover-hardened) |
| SHA-256 | SHA-384/512, SHA-3 | FIPS 180-4/202 | Hashing |

## Validation Criteria

- [ ] OpenSSL 3.5+ (or 3.x + oqs-provider) confirmed exposing ML-KEM and ML-DSA.
- [ ] Cryptographic inventory / CBOM produced covering algorithms, keys, certs, and protocols.
- [ ] Assets classified and prioritized by HNDL exposure (Mosca's inequality applied).
- [ ] ML-KEM-768 and ML-DSA-65 keypairs generated successfully.
- [ ] PQC (ML-DSA) certificate issued and its signature algorithm verified.
- [ ] Sign/verify round trip with ML-DSA returns "Verified OK".
- [ ] Hybrid `X25519MLKEM768` key exchange negotiated and confirmed on a test endpoint.
- [ ] Production TLS terminator offers the hybrid group with classical fallback.
- [ ] Crypto-agility/rotation plan documented and inventory re-run scheduled.

===== END FILE =====


===== FILE: skills/migrating-to-post-quantum-cryptography/LICENSE =====

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


===== FILE: skills/migrating-to-post-quantum-cryptography/references/api-reference.md =====
# OpenSSL PQC Command Reference

## Discovery

| Task | Command |
|------|---------|
| OpenSSL version (need 3.5+) | `openssl version` |
| List quantum-safe KEMs | `openssl list -kem-algorithms \| grep -i mlkem` |
| List quantum-safe signatures | `openssl list -signature-algorithms \| grep -Ei 'mldsa\|slhdsa'` |
| List TLS groups | `openssl list -tls-groups \| grep -i mlkem` |
| Inspect cert algorithm | `openssl x509 -in server.crt -noout -text \| grep -E 'Signature Algorithm\|Public Key'` |

## Key generation

| Task | Command (OpenSSL 3.5+) | oqs-provider (3.0–3.4) |
|------|------------------------|------------------------|
| ML-KEM-768 keypair | `openssl genpkey -algorithm ML-KEM-768 -out mlkem768.key` | `-algorithm mlkem768` |
| ML-DSA-65 keypair | `openssl genpkey -algorithm ML-DSA-65 -out mldsa65.key` | `-algorithm mldsa65` |
| Extract public key | `openssl pkey -in mldsa65.key -pubout -out mldsa65.pub` | same |

## Certificates

| Task | Command |
|------|---------|
| Self-signed ML-DSA cert | `openssl req -new -x509 -key mldsa65.key -out mldsa65.crt -days 365 -subj "/CN=pqc.example.com"` |
| Inspect signature alg | `openssl x509 -in mldsa65.crt -noout -text \| grep -A1 'Signature Algorithm'` |

## Sign / verify

| Task | Command |
|------|---------|
| Sign | `openssl dgst -sign mldsa65.key -out artifact.sig artifact.txt` |
| Verify | `openssl dgst -verify mldsa65.pub -signature artifact.sig artifact.txt` |

## Hybrid TLS key exchange

| Task | Command |
|------|---------|
| TLS server (hybrid group) | `openssl s_server -accept 4433 -www -tls1_3 -cert mldsa65.crt -key mldsa65.key -groups X25519MLKEM768` |
| TLS client (hybrid group) | `openssl s_client -connect localhost:4433 -tls1_3 -groups X25519MLKEM768` |
| Test public PQC endpoint | `openssl s_client -groups X25519MLKEM768 -tls1_3 -connect pq.cloudflareresearch.com:443` |

### Standardized hybrid TLS groups

| Group | Classical leg | PQC leg |
|-------|---------------|---------|
| X25519MLKEM768 | X25519 | ML-KEM-768 |
| SecP256r1MLKEM768 | NIST P-256 | ML-KEM-768 |
| SecP384r1MLKEM1024 | NIST P-384 | ML-KEM-1024 |

## NGINX hybrid config (OpenSSL 3.5+)

```nginx
ssl_protocols TLSv1.3;
ssl_ecdh_curve X25519MLKEM768:X25519:secp256r1;   # hybrid first, classical fallback
```

## oqs-provider activation (openssl.cnf)

```ini
[provider_sect]
default = default_sect
oqsprovider = oqsprovider_sect
[default_sect]
activate = 1
[oqsprovider_sect]
activate = 1
```

## CBOM generation

| Tool | Command |
|------|---------|
| cbomkit-theia (dir) | `cbomkit-theia dir ./myapp --output cbom.json` |
| cdxgen (Java + crypto) | `cdxgen -t java --include-crypto -o cbom.json ./myapp` |

===== END FILE =====


===== FILE: skills/migrating-to-post-quantum-cryptography/references/standards.md =====
# Standards and Framework Mapping

## NIST Cybersecurity Framework 2.0

| ID | Name | Rationale |
|----|------|-----------|
| PR.DS-02 | The confidentiality, integrity, and availability of data-in-transit are protected | Hybrid PQC key exchange (X25519MLKEM768) protects data in transit against harvest-now-decrypt-later attacks by a future CRQC. |

## MITRE ATT&CK

| ID | Name | Rationale |
|----|------|-----------|
| T1573 | Encrypted Channel | Migration hardens the encrypted channels protecting data in transit; cryptographic inventory of these channels also underpins detection of adversary encrypted C2. |
| T1573.001 | Encrypted Channel: Symmetric Cryptography | AES/symmetric ciphers — quantum-weakened by Grover and hardened via 256-bit keys. |
| T1573.002 | Encrypted Channel: Asymmetric Cryptography | RSA/ECDH — the asymmetric primitives broken by Shor's algorithm and replaced by ML-KEM. |

## NIST Post-Quantum Standards (finalized 13 Aug 2024)

| Standard | Algorithm | Former name | Purpose |
|----------|-----------|-------------|---------|
| FIPS 203 | ML-KEM (Module-Lattice KEM) | CRYSTALS-Kyber | Key encapsulation / establishment |
| FIPS 204 | ML-DSA (Module-Lattice DSA) | CRYSTALS-Dilithium | Primary digital signatures |
| FIPS 205 | SLH-DSA (Stateless Hash-based DSA) | SPHINCS+ | Conservative backup signatures |

## Migration Guidance

| Reference | Rationale |
|-----------|-----------|
| NIST SP 1800-38 (NCCoE, Migration to Post-Quantum Cryptography) | Crypto-discovery test plan, CBOM-driven inventory, and migration architecture across CI/CD, operational systems, and network services. |
| Mosca's inequality | Prioritization rule: migrate when data_lifetime + migration_time > time_to_CRQC. |
| CycloneDX 1.6 CBOM | Cryptography Bill of Materials object model for inventory and dependency tracking. |

===== END FILE =====


===== FILE: skills/migrating-to-post-quantum-cryptography/scripts/agent.py =====
#!/usr/bin/env python3
"""
pqc_agent.py — Post-quantum cryptography migration helper.

Three defensive functions for quantum-readiness work:

  scan      Inventory the public-key crypto of a remote TLS endpoint and a set
            of local X.509 certificates, flagging quantum-vulnerable algorithms
            (RSA/EC/DSA/DH) vs. quantum-safe (ML-KEM / ML-DSA / SLH-DSA).
  prioritize  Apply Mosca's inequality to a CSV of assets to rank migration order.
  hybrid-test  Use the local OpenSSL CLI to negotiate the hybrid X25519MLKEM768
            TLS group against a host and report whether it succeeded.

Run only against systems you are authorized to assess. The scan opens TLS
sockets and reads certificates; it does not transmit or store key material.

Examples:
  python3 pqc_agent.py scan --host example.com --port 443 --certs ./certs/*.pem
  python3 pqc_agent.py prioritize --csv assets.csv --crqc-years 8
  python3 pqc_agent.py hybrid-test --host pq.cloudflareresearch.com --port 443
"""
import argparse
import csv
import glob
import socket
import ssl
import subprocess
import sys

try:
    from cryptography import x509
    from cryptography.hazmat.primitives.asymmetric import rsa, ec, dsa, ed25519, ed448
    HAVE_CRYPTO = True
except ImportError:
    HAVE_CRYPTO = False

QUANTUM_VULNERABLE = ("rsa", "ec", "ecdsa", "dsa", "dh", "ecdh", "ed25519", "ed448")
QUANTUM_SAFE = ("ml-kem", "mlkem", "ml-dsa", "mldsa", "slh-dsa", "slhdsa")


def _classify_public_key(pubkey):
    """Return (algorithm_label, vulnerable_bool, key_size_or_curve)."""
    if isinstance(pubkey, rsa.RSAPublicKey):
        return "RSA", True, pubkey.key_size
    if isinstance(pubkey, ec.EllipticCurvePublicKey):
        return "EC", True, pubkey.curve.name
    if isinstance(pubkey, dsa.DSAPublicKey):
        return "DSA", True, pubkey.key_size
    if isinstance(pubkey, (ed25519.Ed25519PublicKey, ed448.Ed448PublicKey)):
        return type(pubkey).__name__, True, "edwards"
    return pubkey.__class__.__name__, False, "?"


def _fetch_peer_cert_pem(host, port, timeout):
    ctx = ssl.create_default_context()
    ctx.check_hostname = False
    ctx.verify_mode = ssl.CERT_NONE
    with socket.create_connection((host, port), timeout=timeout) as sock:
        with ctx.wrap_socket(sock, server_hostname=host) as ssock:
            der = ssock.getpeercert(binary_form=True)
            version = ssock.version()
            cipher = ssock.cipher()
    return ssl.DER_cert_to_PEM_cert(der), version, cipher


def _inspect_cert_pem(pem_text, label):
    if not HAVE_CRYPTO:
        sys.stderr.write("ERROR: install dependency: python3 -m pip install cryptography\n")
        sys.exit(2)
    cert = x509.load_pem_x509_certificate(pem_text.encode())
    alg, vuln, size = _classify_public_key(cert.public_key())
    sig = cert.signature_algorithm_oid._name
    sig_vuln = any(v in sig.lower() for v in QUANTUM_VULNERABLE) and \
        not any(s in sig.lower() for s in QUANTUM_SAFE)
    flag = "VULNERABLE" if (vuln or sig_vuln) else "quantum-safe"
    print(f"[{flag:>12}] {label}")
    print(f"               subject : {cert.subject.rfc4514_string()}")
    print(f"               pubkey  : {alg} ({size})")
    print(f"               sig alg : {sig}")
    return vuln or sig_vuln


def cmd_scan(args):
    findings = 0
    if args.host:
        try:
            pem, version, cipher = _fetch_peer_cert_pem(args.host, args.port, args.timeout)
            print(f"== Remote endpoint {args.host}:{args.port} ({version}, {cipher[0]}) ==")
            if _inspect_cert_pem(pem, f"{args.host}:{args.port} leaf cert"):
                findings += 1
        except (socket.error, ssl.SSLError, OSError) as exc:
            sys.stderr.write(f"ERROR: could not connect to {args.host}:{args.port}: {exc}\n")
    cert_paths = []
    for pattern in args.certs:
        cert_paths.extend(glob.glob(pattern))
    if cert_paths:
        print("\n== Local certificates ==")
    for path in cert_paths:
        try:
            with open(path, "r", encoding="utf-8") as fh:
                if _inspect_cert_pem(fh.read(), path):
                    findings += 1
        except (OSError, ValueError) as exc:
            sys.stderr.write(f"WARN: skipping {path}: {exc}\n")
    print(f"\nQuantum-vulnerable assets found: {findings}")
    return 1 if findings else 0


def cmd_prioritize(args):
    """Mosca's inequality: migrate if data_lifetime + migration_time > crqc_years."""
    try:
        with open(args.csv, newline="", encoding="utf-8") as fh:
            rows = list(csv.DictReader(fh))
    except OSError as exc:
        sys.stderr.write(f"ERROR: cannot read {args.csv}: {exc}\n")
        sys.exit(1)
    scored = []
    for r in rows:
        try:
            life = float(r.get("data_lifetime_years", 0))
            mig = float(r.get("migration_time_years", 1))
        except ValueError:
            life, mig = 0.0, 1.0
        urgent = (life + mig) > args.crqc_years
        margin = (life + mig) - args.crqc_years
        scored.append((margin, urgent, r.get("asset", "?"), life, mig))
    scored.sort(reverse=True)
    print(f"{'PRIORITY':<10}{'MARGIN':>8}  ASSET (data_life + migration vs CRQC={args.crqc_years}y)")
    print("-" * 70)
    for margin, urgent, asset, life, mig in scored:
        tag = "MIGRATE" if urgent else "monitor"
        print(f"{tag:<10}{margin:>+8.1f}  {asset}  (life={life}, mig={mig})")
    return 0


def cmd_hybrid_test(args):
    cmd = ["openssl", "s_client", "-tls1_3", "-groups", "X25519MLKEM768",
           "-connect", f"{args.host}:{args.port}"]
    try:
        proc = subprocess.run(cmd, input=b"", capture_output=True, timeout=args.timeout)
    except FileNotFoundError:
        sys.stderr.write("ERROR: openssl not found on PATH (need 3.5+ or oqs-provider).\n")
        sys.exit(2)
    except subprocess.TimeoutExpired:
        sys.stderr.write("ERROR: openssl s_client timed out.\n")
        sys.exit(1)
    out = (proc.stdout + proc.stderr).decode(errors="replace")
    ok = "Server Temp Key" in out or "Negotiated TLS1.3 group: X25519MLKEM768" in out
    if proc.returncode == 0 and ok:
        for line in out.splitlines():
            if "Temp Key" in line or "Negotiated" in line or "Cipher" in line:
                print(line.strip())
        print(f"\n[OK] {args.host}:{args.port} negotiated hybrid X25519MLKEM768")
        return 0
    sys.stderr.write(f"[FAIL] {args.host}:{args.port} did not negotiate X25519MLKEM768\n")
    sys.stderr.write(out[-800:] + "\n")
    return 1


def build_parser():
    p = argparse.ArgumentParser(description="Post-quantum cryptography migration helper.")
    sub = p.add_subparsers(dest="cmd", required=True)

    s = sub.add_parser("scan", help="Inventory TLS endpoint + local certs for quantum-vulnerable crypto")
    s.add_argument("--host", help="Remote host to inspect")
    s.add_argument("--port", type=int, default=443)
    s.add_argument("--certs", nargs="*", default=[], help="Glob(s) of local PEM certs")
    s.add_argument("--timeout", type=float, default=10)
    s.set_defaults(func=cmd_scan)

    pr = sub.add_parser("prioritize", help="Rank assets via Mosca's inequality from a CSV")
    pr.add_argument("--csv", required=True,
                    help="CSV with columns: asset,data_lifetime_years,migration_time_years")
    pr.add_argument("--crqc-years", type=float, default=10,
                    help="Estimated years until a cryptographically relevant quantum computer")
    pr.set_defaults(func=cmd_prioritize)

    h = sub.add_parser("hybrid-test", help="Test hybrid X25519MLKEM768 negotiation via openssl")
    h.add_argument("--host", required=True)
    h.add_argument("--port", type=int, default=443)
    h.add_argument("--timeout", type=float, default=15)
    h.set_defaults(func=cmd_hybrid_test)
    return p


def main():
    args = build_parser().parse_args()
    return args.func(args)


if __name__ == "__main__":
    sys.exit(main())

===== END FILE =====


===== FILE: skills/performing-cryptographic-audit-of-application/SKILL.md =====
---
name: performing-cryptographic-audit-of-application
description: A cryptographic audit systematically reviews an application's use of
  cryptographic primitives, protocols, and key management to identify vulnerabilities
  such as weak algorithms, insecure modes, hardco
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- audit
- security-review
- compliance
- vulnerability-assessment
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - initial-access
  - positioning
  techniques:
  - id: T1557
    name: Adversary-in-the-Middle
    tactic: positioning
    source: attack
  - id: T1555
    name: Credentials from Password Stores
    tactic: reconnaissance
    source: attack
  - id: F1006.001
    name: 'Account Takeover: Exposed API Key'
    tactic: initial-access
    source: f3
  - id: F1004
    name: Access with Stolen Session Cookie
    tactic: initial-access
    source: f3
---
# Performing Cryptographic Audit of Application

## Overview

A cryptographic audit systematically reviews an application's use of cryptographic primitives, protocols, and key management to identify vulnerabilities such as weak algorithms, insecure modes, hardcoded keys, insufficient entropy, and protocol misconfigurations. This skill covers building an automated crypto audit tool that scans Python and configuration files for common cryptographic weaknesses.


## When to Use

- When conducting security assessments that involve performing cryptographic audit of application
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Detect usage of deprecated algorithms (MD5, SHA-1, DES, RC4)
- Identify insecure cipher modes (ECB) and padding schemes
- Find hardcoded keys, passwords, and secrets in source code
- Verify TLS/SSL configuration strength
- Check key derivation function parameters
- Validate random number generator usage
- Produce a structured audit report with findings and remediation

## Key Concepts

### Cryptographic Weakness Categories

| Category | Examples | Risk Level |
|----------|----------|------------|
| Weak Hashing | MD5, SHA-1 for integrity/signatures | High |
| Insecure Encryption | DES, 3DES, RC4, Blowfish | High |
| Bad Cipher Mode | ECB mode for any block cipher | High |
| Insufficient Key Size | RSA < 2048, AES-128 for long-term | Medium |
| Hardcoded Secrets | Keys/passwords in source code | Critical |
| Weak KDF | Low iteration PBKDF2, plain MD5 | High |
| Poor Entropy | time-based seeds, predictable IVs | High |
| Deprecated Protocols | SSLv3, TLS 1.0, TLS 1.1 | High |

## Security Considerations

- Review both application code and configuration files
- Check third-party dependencies for known crypto vulnerabilities
- Verify certificates and TLS configurations on deployed servers
- Ensure secrets are loaded from environment variables or vaults
- Review key storage and rotation practices

## Validation Criteria

- [ ] Scanner detects all injected test weaknesses
- [ ] MD5/SHA-1 usage for security purposes is flagged
- [ ] ECB mode usage is flagged
- [ ] Hardcoded keys/passwords are detected
- [ ] Weak KDF parameters are identified
- [ ] Report includes severity, location, and remediation
- [ ] False positive rate is below 10%

===== END FILE =====


===== FILE: skills/performing-cryptographic-audit-of-application/LICENSE =====

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


===== FILE: skills/performing-cryptographic-audit-of-application/assets/template.md =====
# Cryptographic Audit Report Template

## Executive Summary

| Metric | Value |
|--------|-------|
| Target | [Application Name] |
| Scan Date | [Date] |
| Overall Risk | [CRITICAL/HIGH/MEDIUM/LOW] |
| Total Findings | [Count] |
| Critical | [Count] |
| High | [Count] |
| Medium | [Count] |
| Low | [Count] |

## Audit Scope

- [ ] Source code review (Python, JavaScript, Java, Go)
- [ ] Configuration file review (YAML, JSON, INI)
- [ ] TLS/SSL configuration assessment
- [ ] Key management practices review
- [ ] Dependency vulnerability check
- [ ] Deployed infrastructure scan

## Approved Algorithms

| Purpose | Approved | Deprecated |
|---------|----------|-----------|
| Hashing (integrity) | SHA-256, SHA-3 | MD5, SHA-1 |
| Password hashing | Argon2id, bcrypt, scrypt | MD5, SHA-1, plain SHA-256 |
| Symmetric encryption | AES-256-GCM, ChaCha20-Poly1305 | DES, 3DES, RC4, Blowfish |
| Asymmetric encryption | RSA-OAEP (3072+), ECIES | RSA-PKCS1v15 |
| Digital signatures | Ed25519, RSA-PSS (3072+), ECDSA (P-256+) | RSA-PKCS1v15 |
| Key exchange | X25519, ECDH (P-256+), DH (3072+) | DH (1024) |
| TLS | TLS 1.2 (approved ciphers), TLS 1.3 | SSLv3, TLS 1.0, TLS 1.1 |

## Finding Template

```
### Finding F-001: [Title]

**Severity**: CRITICAL / HIGH / MEDIUM / LOW
**Category**: [Category]
**CWE**: [CWE-XXX]
**File**: [file_path:line_number]

**Description**:
[Description of the vulnerability]

**Code**:
[Code snippet]

**Remediation**:
[Steps to fix]

**References**:
- [NIST/OWASP reference]
```

## Remediation Priority

1. CRITICAL: Hardcoded secrets, broken encryption (fix immediately)
2. HIGH: Weak algorithms, insecure modes (fix in current sprint)
3. MEDIUM: Suboptimal parameters, deprecated protocols (next release)
4. LOW: Informational, best practice improvements (backlog)

===== END FILE =====


===== FILE: skills/performing-cryptographic-audit-of-application/references/api-reference.md =====
# API Reference: Application Cryptographic Audit

## Libraries Used

| Library | Purpose |
|---------|---------|
| `ssl` | TLS connection inspection and cipher suite enumeration |
| `socket` | TCP connections for TLS handshake testing |
| `cryptography` | Certificate parsing, key strength analysis |
| `json` | Structure audit findings |
| `datetime` | Check certificate validity periods |

## Installation

```bash
pip install cryptography
```

## TLS Configuration Audit

### Check TLS Version and Cipher Suite
```python
import ssl
import socket

def check_tls_config(hostname, port=443):
    context = ssl.create_default_context()
    with socket.create_connection((hostname, port), timeout=10) as sock:
        with context.wrap_socket(sock, server_hostname=hostname) as ssock:
            return {
                "hostname": hostname,
                "protocol": ssock.version(),
                "cipher": ssock.cipher()[0],
                "cipher_bits": ssock.cipher()[2],
                "compression": ssock.compression(),
            }
```

### Test for Weak TLS Versions
```python
WEAK_PROTOCOLS = {
    ssl.PROTOCOL_TLSv1: "TLSv1.0",
    ssl.PROTOCOL_TLSv1_1: "TLSv1.1",
}

def test_weak_tls(hostname, port=443):
    findings = []
    for protocol_const, name in WEAK_PROTOCOLS.items():
        try:
            ctx = ssl.SSLContext(protocol_const)
            ctx.check_hostname = False
            ctx.verify_mode = ssl.CERT_NONE
            with socket.create_connection((hostname, port), timeout=5) as sock:
                with ctx.wrap_socket(sock) as ssock:
                    findings.append({
                        "protocol": name,
                        "supported": True,
                        "severity": "high",
                        "issue": f"{name} is supported — deprecated and insecure",
                    })
        except (ssl.SSLError, ConnectionRefusedError, OSError):
            findings.append({"protocol": name, "supported": False})
    return findings
```

### Enumerate Supported Cipher Suites
```python
WEAK_CIPHERS = {
    "RC4", "DES", "3DES", "MD5", "NULL", "EXPORT", "anon", "CBC",
}

def check_cipher_suites(hostname, port=443):
    context = ssl.create_default_context()
    context.set_ciphers("ALL:COMPLEMENTOFALL")
    findings = []
    try:
        with socket.create_connection((hostname, port), timeout=10) as sock:
            with context.wrap_socket(sock, server_hostname=hostname) as ssock:
                cipher_name, protocol, bits = ssock.cipher()
                is_weak = any(w in cipher_name for w in WEAK_CIPHERS)
                findings.append({
                    "cipher": cipher_name,
                    "bits": bits,
                    "weak": is_weak,
                    "severity": "high" if is_weak else "pass",
                })
    except ssl.SSLError as e:
        findings.append({"error": str(e)})
    return findings
```

## Certificate Analysis

### Parse and Audit Certificate
```python
from cryptography import x509
from cryptography.hazmat.primitives.asymmetric import rsa, ec
from datetime import datetime, timezone

def audit_certificate(hostname, port=443):
    context = ssl.create_default_context()
    with socket.create_connection((hostname, port), timeout=10) as sock:
        with context.wrap_socket(sock, server_hostname=hostname) as ssock:
            der_cert = ssock.getpeercert(binary_form=True)

    cert = x509.load_der_x509_certificate(der_cert)
    pub_key = cert.public_key()

    findings = []
    # Key strength check
    if isinstance(pub_key, rsa.RSAPublicKey):
        key_size = pub_key.key_size
        if key_size < 2048:
            findings.append({
                "check": "key_size",
                "severity": "critical",
                "detail": f"RSA key {key_size} bits — minimum 2048 required",
            })
    elif isinstance(pub_key, ec.EllipticCurvePublicKey):
        key_size = pub_key.curve.key_size
        if key_size < 256:
            findings.append({
                "check": "key_size",
                "severity": "high",
                "detail": f"EC key {key_size} bits — minimum 256 required",
            })

    # Signature algorithm
    sig_algo = cert.signature_algorithm_oid._name
    if "sha1" in sig_algo.lower():
        findings.append({
            "check": "signature_algorithm",
            "severity": "high",
            "detail": f"SHA-1 signature ({sig_algo}) — deprecated",
        })

    # Validity period
    now = datetime.now(timezone.utc)
    days_remaining = (cert.not_valid_after_utc - now).days
    if days_remaining < 0:
        findings.append({"check": "expiry", "severity": "critical", "detail": "Certificate expired"})
    elif days_remaining < 30:
        findings.append({"check": "expiry", "severity": "warning", "detail": f"Expires in {days_remaining} days"})

    return {
        "subject": cert.subject.rfc4514_string(),
        "issuer": cert.issuer.rfc4514_string(),
        "not_before": cert.not_valid_before_utc.isoformat(),
        "not_after": cert.not_valid_after_utc.isoformat(),
        "days_remaining": days_remaining,
        "serial_number": hex(cert.serial_number),
        "signature_algorithm": sig_algo,
        "key_type": "RSA" if isinstance(pub_key, rsa.RSAPublicKey) else "EC",
        "key_size": key_size,
        "findings": findings,
    }
```

### Check HSTS Header
```python
import requests

def check_hsts(url):
    resp = requests.get(url, timeout=10, allow_redirects=True)
    hsts = resp.headers.get("Strict-Transport-Security", "")
    findings = []
    if not hsts:
        findings.append({"check": "hsts", "severity": "medium", "detail": "HSTS header missing"})
    else:
        if "includeSubDomains" not in hsts:
            findings.append({"check": "hsts", "severity": "low", "detail": "HSTS missing includeSubDomains"})
        max_age = 0
        for part in hsts.split(";"):
            if "max-age" in part:
                max_age = int(part.split("=")[1].strip())
        if max_age < 31536000:
            findings.append({"check": "hsts_max_age", "severity": "low", "detail": f"max-age {max_age} < 1 year"})

    return {"hsts_header": hsts, "findings": findings}
```

## Output Format

```json
{
  "hostname": "example.com",
  "tls_version": "TLSv1.3",
  "cipher": "TLS_AES_256_GCM_SHA384",
  "certificate": {
    "subject": "CN=example.com",
    "issuer": "CN=R3,O=Let's Encrypt",
    "days_remaining": 62,
    "key_type": "EC",
    "key_size": 256
  },
  "weak_tls_supported": ["TLSv1.0"],
  "hsts_enabled": true,
  "findings": [
    {
      "check": "weak_tls",
      "severity": "high",
      "detail": "TLSv1.0 is supported — deprecated and insecure"
    }
  ]
}
```

===== END FILE =====


===== FILE: skills/performing-cryptographic-audit-of-application/references/standards.md =====
# Standards and References - Cryptographic Audit

## NIST Guidelines

### NIST SP 800-131A Rev. 2 - Transitioning Cryptographic Algorithms
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-131a/rev-2/final
- **Description**: Defines deprecated, legacy, and acceptable algorithms

### NIST SP 800-57 Part 1 Rev. 5 - Key Management
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final
- **Description**: Minimum key sizes and algorithm recommendations

## OWASP References

### OWASP Cryptographic Failures
- **URL**: https://owasp.org/Top10/A02_2021-Cryptographic_Failures/
- **Description**: OWASP Top 10 #2 - Cryptographic Failures

### OWASP Cheat Sheet - Cryptographic Storage
- **URL**: https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
- **Description**: Guidelines for secure cryptographic storage

### OWASP Cheat Sheet - Password Storage
- **URL**: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- **Description**: Password hashing best practices

## Tools

### Bandit
- **URL**: https://bandit.readthedocs.io/
- **Description**: Python security linter with crypto checks

### Semgrep
- **URL**: https://semgrep.dev/
- **Crypto rules**: https://semgrep.dev/r?q=crypto
- **Description**: Static analysis with crypto-specific rules

### CryptoGuard
- **URL**: https://github.com/AkshayaVS/CryptoGuard
- **Description**: Specialized cryptographic misuse detector

===== END FILE =====


===== FILE: skills/performing-cryptographic-audit-of-application/references/workflows.md =====
# Workflows - Cryptographic Audit

## Workflow 1: Automated Source Code Scan

```
[Target Application Source]
      |
[Scan for Crypto Patterns]:
  - Deprecated algorithms (MD5, SHA-1, DES, RC4)
  - Insecure modes (ECB)
  - Hardcoded secrets (keys, passwords, tokens)
  - Weak KDF parameters
  - Insecure random number generation
      |
[Scan Configuration Files]:
  - TLS/SSL settings
  - Cipher suite configurations
  - Certificate paths and validity
      |
[Generate Findings with Severity]
      |
[Produce Audit Report]
```

## Workflow 2: Manual Crypto Review

```
[Identify All Crypto Touchpoints]:
  - Encryption/decryption operations
  - Hashing operations
  - Key generation and storage
  - TLS/SSL connections
  - Token generation (JWT, API keys)
  - Password handling
      |
[For Each Touchpoint]:
  [Verify algorithm choice]
  [Verify mode/padding]
  [Verify key management]
  [Verify entropy sources]
  [Verify error handling]
      |
[Document Findings]
```

## Workflow 3: Remediation Prioritization

```
[All Findings]
      |
[Classify by Severity]:
  CRITICAL: Hardcoded keys, broken encryption
  HIGH: Weak algorithms, ECB mode, weak KDF
  MEDIUM: Short key sizes, deprecated protocols
  LOW: Missing best practices, informational
      |
[Prioritize by Risk]:
  1. CRITICAL findings (immediate fix)
  2. HIGH findings (fix in current sprint)
  3. MEDIUM findings (plan for next release)
  4. LOW findings (backlog)
```

===== END FILE =====


===== FILE: skills/performing-cryptographic-audit-of-application/scripts/agent.py =====
#!/usr/bin/env python3
"""Application cryptographic audit agent.

Audits application code and configurations for cryptographic weaknesses
including weak algorithms, insecure key sizes, hardcoded keys, deprecated
protocols, and missing certificate validation. Scans source files, config
files, and TLS endpoints.
"""
import argparse
import json
import os
import re
import ssl
import socket
import sys
from datetime import datetime, timezone

try:
    from cryptography import x509
    from cryptography.hazmat.backends import default_backend
    HAS_CRYPTO = True
except ImportError:
    HAS_CRYPTO = False


WEAK_PATTERNS = {
    "weak_hash": {
        "pattern": r'\b(MD5|md5|SHA1|sha1|SHA-1)\b(?!.*hmac)',
        "severity": "HIGH",
        "description": "Weak hash algorithm detected (MD5/SHA1)",
        "recommendation": "Use SHA-256 or SHA-3",
    },
    "weak_cipher": {
        "pattern": r'\b(DES|3DES|RC4|RC2|Blowfish|IDEA)\b',
        "severity": "HIGH",
        "description": "Weak cipher algorithm detected",
        "recommendation": "Use AES-256-GCM or ChaCha20-Poly1305",
    },
    "ecb_mode": {
        "pattern": r'\b(ECB|MODE_ECB|ecb)\b',
        "severity": "CRITICAL",
        "description": "ECB mode detected - does not provide semantic security",
        "recommendation": "Use GCM, CBC with HMAC, or CTR mode",
    },
    "hardcoded_key": {
        "pattern": r'(?:key|secret|password|token)\s*[=:]\s*["\'][A-Za-z0-9+/=]{16,}["\']',
        "severity": "CRITICAL",
        "description": "Possible hardcoded cryptographic key or secret",
        "recommendation": "Use environment variables or a secrets manager",
    },
    "small_rsa_key": {
        "pattern": r'(?:key_?size|bits)\s*[=:]\s*(?:512|768|1024)\b',
        "severity": "CRITICAL",
        "description": "RSA key size too small (<2048 bits)",
        "recommendation": "Use minimum 2048-bit, preferably 4096-bit RSA keys",
    },
    "weak_random": {
        "pattern": r'\b(?:random\.random|math\.random|Math\.random|rand\(\)|srand)\b',
        "severity": "HIGH",
        "description": "Non-cryptographic random number generator used",
        "recommendation": "Use secrets module, os.urandom, or crypto.getRandomValues",
    },
    "ssl_no_verify": {
        "pattern": r'(?:verify\s*=\s*False|CERT_NONE|SSL_VERIFY_NONE|InsecureRequestWarning|verify_ssl\s*=\s*False)',
        "severity": "HIGH",
        "description": "TLS certificate verification disabled",
        "recommendation": "Enable certificate verification in all TLS connections",
    },
    "tls_v1": {
        "pattern": r'(?:TLSv1[^._23]|SSLv[23]|PROTOCOL_TLS(?!v1_[23])|TLS_1_0|TLS_1_1)',
        "severity": "HIGH",
        "description": "Deprecated TLS/SSL protocol version detected",
        "recommendation": "Use TLS 1.2 or TLS 1.3 minimum",
    },
    "padding_oracle": {
        "pattern": r'(?:PKCS1v15|pkcs1_v1_5|PKCS5Padding)(?!.*OAEP)',
        "severity": "MEDIUM",
        "description": "PKCS#1 v1.5 padding used (vulnerable to padding oracle attacks)",
        "recommendation": "Use OAEP padding for RSA, or switch to AEAD ciphers",
    },
    "static_iv": {
        "pattern": r'(?:iv|IV|nonce)\s*[=:]\s*(?:b?["\'][^"\']{1,32}["\']|bytes\([^)]*\))',
        "severity": "HIGH",
        "description": "Possible static/hardcoded IV or nonce",
        "recommendation": "Generate random IV/nonce for each encryption operation",
    },
}

SCAN_EXTENSIONS = {
    ".py", ".js", ".ts", ".java", ".go", ".rb", ".php", ".cs", ".c", ".cpp",
    ".h", ".yaml", ".yml", ".json", ".xml", ".conf", ".cfg", ".ini", ".env",
    ".toml", ".properties",
}


def scan_file(file_path):
    """Scan a single file for cryptographic weaknesses."""
    findings = []
    try:
        with open(file_path, "r", encoding="utf-8", errors="ignore") as f:
            lines = f.readlines()
    except (IOError, PermissionError):
        return findings

    for line_num, line in enumerate(lines, 1):
        for rule_id, rule in WEAK_PATTERNS.items():
            if re.search(rule["pattern"], line):
                findings.append({
                    "rule": rule_id,
                    "file": file_path,
                    "line": line_num,
                    "content": line.strip()[:120],
                    "severity": rule["severity"],
                    "description": rule["description"],
                    "recommendation": rule["recommendation"],
                })
    return findings


def scan_directory(directory, extensions=None, exclude_dirs=None):
    """Recursively scan a directory for cryptographic weaknesses."""
    if extensions is None:
        extensions = SCAN_EXTENSIONS
    if exclude_dirs is None:
        exclude_dirs = {"node_modules", ".git", "__pycache__", "venv", ".venv",
                        "vendor", "dist", "build", ".tox"}

    all_findings = []
    files_scanned = 0

    for root, dirs, files in os.walk(directory):
        dirs[:] = [d for d in dirs if d not in exclude_dirs]
        for fname in files:
            ext = os.path.splitext(fname)[1].lower()
            if ext not in extensions:
                continue
            full_path = os.path.join(root, fname)
            findings = scan_file(full_path)
            all_findings.extend(findings)
            files_scanned += 1

    print(f"[+] Scanned {files_scanned} files, found {len(all_findings)} issues")
    return all_findings, files_scanned


def audit_tls_endpoint(host, port=443):
    """Audit TLS configuration of a remote endpoint."""
    findings = []
    print(f"[*] Auditing TLS: {host}:{port}")

    try:
        context = ssl.create_default_context()
        with socket.create_connection((host, port), timeout=10) as sock:
            with context.wrap_socket(sock, server_hostname=host) as ssock:
                cert_der = ssock.getpeercert(binary_form=True)
                cipher = ssock.cipher()
                protocol = ssock.version()

                findings.append({
                    "check": "TLS Protocol",
                    "value": protocol,
                    "severity": "CRITICAL" if "TLSv1.0" in protocol or "TLSv1.1" in protocol
                               else "INFO",
                    "description": f"Negotiated protocol: {protocol}",
                })

                if cipher:
                    cipher_name, tls_ver, key_bits = cipher
                    findings.append({
                        "check": "Cipher Suite",
                        "value": cipher_name,
                        "severity": "HIGH" if any(w in cipher_name for w in ["RC4", "DES", "NULL", "EXPORT"])
                                   else "INFO",
                        "description": f"Cipher: {cipher_name} ({key_bits} bits)",
                    })

                if HAS_CRYPTO and cert_der:
                    cert = x509.load_der_x509_certificate(cert_der, default_backend())
                    not_after = cert.not_valid_after_utc if hasattr(cert, 'not_valid_after_utc') else cert.not_valid_after
                    days_remaining = (not_after - datetime.now(timezone.utc)).days

                    pub_key = cert.public_key()
                    key_size = getattr(pub_key, 'key_size', 0)
                    sig_algo = cert.signature_algorithm_oid._name if hasattr(cert.signature_algorithm_oid, '_name') else str(cert.signature_hash_algorithm)

                    findings.append({
                        "check": "Certificate Expiry",
                        "value": f"{days_remaining} days",
                        "severity": "CRITICAL" if days_remaining < 0
                                   else "HIGH" if days_remaining < 30
                                   else "MEDIUM" if days_remaining < 90
                                   else "INFO",
                        "description": f"Expires: {not_after.isoformat()} ({days_remaining} days)",
                    })
                    findings.append({
                        "check": "Key Size",
                        "value": f"{key_size} bits",
                        "severity": "CRITICAL" if key_size < 2048
                                   else "MEDIUM" if key_size < 4096
                                   else "INFO",
                    })
                    findings.append({
                        "check": "Signature Algorithm",
                        "value": str(sig_algo),
                        "severity": "HIGH" if "sha1" in str(sig_algo).lower() else "INFO",
                    })

    except ssl.SSLError as e:
        findings.append({"check": "TLS Connection", "severity": "CRITICAL",
                         "description": f"SSL error: {e}"})
    except socket.timeout:
        findings.append({"check": "TLS Connection", "severity": "HIGH",
                         "description": "Connection timed out"})
    except Exception as e:
        findings.append({"check": "TLS Connection", "severity": "HIGH",
                         "description": f"Error: {e}"})

    return findings


def format_summary(code_findings, tls_findings, files_scanned, target):
    """Print audit summary."""
    all_findings = code_findings + tls_findings
    print(f"\n{'='*60}")
    print(f"  Cryptographic Audit Report")
    print(f"{'='*60}")
    print(f"  Target         : {target}")
    print(f"  Files Scanned  : {files_scanned}")
    print(f"  Code Findings  : {len(code_findings)}")
    print(f"  TLS Findings   : {len(tls_findings)}")

    severity_counts = {}
    for f in all_findings:
        sev = f.get("severity", "INFO")
        severity_counts[sev] = severity_counts.get(sev, 0) + 1

    print(f"\n  By Severity:")
    for sev in ["CRITICAL", "HIGH", "MEDIUM", "LOW", "INFO"]:
        count = severity_counts.get(sev, 0)
        if count > 0:
            print(f"    {sev:10s}: {count}")

    if code_findings:
        by_rule = {}
        for f in code_findings:
            by_rule.setdefault(f["rule"], []).append(f)
        print(f"\n  Code Issues by Rule:")
        for rule, items in sorted(by_rule.items(), key=lambda x: -len(x[1])):
            print(f"    {rule:25s}: {len(items)} ({items[0]['severity']})")

        print(f"\n  Top Code Findings:")
        for f in code_findings[:15]:
            if f["severity"] in ("CRITICAL", "HIGH"):
                print(f"    [{f['severity']:8s}] {f['file']}:{f['line']} - {f['description']}")

    if tls_findings:
        print(f"\n  TLS Audit Results:")
        for f in tls_findings:
            print(f"    [{f['severity']:8s}] {f.get('check', '')}: "
                  f"{f.get('value', f.get('description', ''))}")

    return severity_counts


def main():
    parser = argparse.ArgumentParser(
        description="Application cryptographic audit agent"
    )
    parser.add_argument("--target", required=True,
                        help="Source directory to scan or TLS host to audit")
    parser.add_argument("--tls-port", type=int, default=443,
                        help="TLS port for endpoint audit (default: 443)")
    parser.add_argument("--tls-only", action="store_true",
                        help="Only audit TLS endpoint, skip code scan")
    parser.add_argument("--code-only", action="store_true",
                        help="Only scan code, skip TLS audit")
    parser.add_argument("--exclude-dirs", nargs="+",
                        help="Additional directories to exclude from scan")
    parser.add_argument("--output", "-o", help="Output JSON report path")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    code_findings = []
    tls_findings = []
    files_scanned = 0

    if not args.tls_only and os.path.isdir(args.target):
        exclude = {"node_modules", ".git", "__pycache__", "venv", ".venv", "vendor"}
        if args.exclude_dirs:
            exclude.update(args.exclude_dirs)
        code_findings, files_scanned = scan_directory(args.target, exclude_dirs=exclude)

    if not args.code_only and not os.path.isdir(args.target):
        tls_findings = audit_tls_endpoint(args.target, args.tls_port)
    elif not args.code_only and os.path.isdir(args.target):
        pass  # Can't audit TLS on a directory

    severity_counts = format_summary(code_findings, tls_findings, files_scanned, args.target)

    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "Crypto Audit",
        "target": args.target,
        "files_scanned": files_scanned,
        "severity_counts": severity_counts,
        "code_findings": code_findings,
        "tls_findings": tls_findings,
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


===== FILE: skills/performing-cryptographic-audit-of-application/scripts/process.py =====
#!/usr/bin/env python3
"""
Cryptographic Audit Scanner

Scans Python source files and configuration files for cryptographic
weaknesses including deprecated algorithms, insecure modes, hardcoded
secrets, and weak key derivation parameters.

Requirements:
    pip install cryptography

Usage:
    python process.py scan --target ./myapp --output audit_report.json
    python process.py scan --target ./myapp/crypto.py --output report.json
    python process.py test-samples  # Run against built-in test samples
"""

import os
import re
import sys
import json
import argparse
import logging
from pathlib import Path
from typing import Dict, List, Optional
from dataclasses import dataclass, asdict

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


@dataclass
class Finding:
    """A cryptographic audit finding."""
    severity: str  # CRITICAL, HIGH, MEDIUM, LOW, INFO
    category: str
    title: str
    description: str
    file_path: str
    line_number: int
    code_snippet: str
    remediation: str
    cwe_id: Optional[str] = None


WEAK_HASH_PATTERNS = [
    (r"\bmd5\b", "MD5", "CWE-328"),
    (r"\bsha1\b", "SHA-1", "CWE-328"),
    (r"\bSHA1\b", "SHA-1", "CWE-328"),
    (r"\bMD5\b", "MD5", "CWE-328"),
    (r"hashlib\.md5", "MD5 via hashlib", "CWE-328"),
    (r"hashlib\.sha1", "SHA-1 via hashlib", "CWE-328"),
    (r"hashes\.MD5", "MD5 via cryptography", "CWE-328"),
    (r"hashes\.SHA1", "SHA-1 via cryptography", "CWE-328"),
]

WEAK_CIPHER_PATTERNS = [
    (r"\bDES\b(?!3)", "DES", "CWE-327"),
    (r"\bRC4\b", "RC4", "CWE-327"),
    (r"\bRC2\b", "RC2", "CWE-327"),
    (r"\bBlowfish\b", "Blowfish", "CWE-327"),
    (r"algorithms\.TripleDES", "3DES", "CWE-327"),
    (r"MODE_ECB", "ECB mode", "CWE-327"),
    (r"AES\.MODE_ECB", "AES-ECB mode", "CWE-327"),
]

HARDCODED_SECRET_PATTERNS = [
    (r'(?:password|passwd|pwd)\s*=\s*["\'][^"\']{4,}["\']', "Hardcoded password", "CWE-798"),
    (r'(?:secret|api_?key|token)\s*=\s*["\'][^"\']{8,}["\']', "Hardcoded secret/key", "CWE-798"),
    (r'(?:private_?key|priv_?key)\s*=\s*["\'][^"\']+["\']', "Hardcoded private key", "CWE-798"),
    (r'["\']-----BEGIN (?:RSA |EC )?PRIVATE KEY-----', "Embedded private key", "CWE-798"),
    (r'AKIA[0-9A-Z]{16}', "AWS Access Key", "CWE-798"),
]

WEAK_KDF_PATTERNS = [
    (r'iterations\s*=\s*(\d+)', "KDF iterations check", "CWE-916"),
    (r'PBKDF2.*iterations.*?(\d+)', "PBKDF2 iterations", "CWE-916"),
]

INSECURE_RANDOM_PATTERNS = [
    (r'\brandom\.random\b', "Insecure random (use secrets/os.urandom)", "CWE-338"),
    (r'\brandom\.randint\b', "Insecure random for crypto", "CWE-338"),
    (r'\brandom\.choice\b(?!.*secrets)', "Insecure random choice", "CWE-338"),
    (r'\brandom\.seed\b', "Seeded random (predictable)", "CWE-338"),
]

DEPRECATED_TLS_PATTERNS = [
    (r'SSLv2', "SSLv2 protocol", "CWE-326"),
    (r'SSLv3', "SSLv3 protocol (POODLE)", "CWE-326"),
    (r'TLSv1[^._23]', "TLS 1.0 protocol", "CWE-326"),
    (r'TLSv1_1\b', "TLS 1.1 protocol", "CWE-326"),
    (r'PROTOCOL_SSLv23', "Legacy SSL context", "CWE-326"),
    (r'ssl\.PROTOCOL_TLS(?!_CLIENT)', "Permissive TLS context", "CWE-326"),
    (r'verify_mode\s*=\s*ssl\.CERT_NONE', "TLS certificate verification disabled", "CWE-295"),
    (r'check_hostname\s*=\s*False', "TLS hostname checking disabled", "CWE-297"),
]


def scan_file(file_path: str) -> List[Finding]:
    """Scan a single file for cryptographic weaknesses."""
    findings = []

    try:
        content = Path(file_path).read_text(encoding="utf-8", errors="ignore")
    except Exception as e:
        logger.warning(f"Cannot read {file_path}: {e}")
        return findings

    lines = content.split("\n")

    for line_num, line in enumerate(lines, 1):
        stripped = line.strip()
        if stripped.startswith("#") or stripped.startswith("//"):
            continue

        # Weak hash algorithms
        for pattern, algo_name, cwe in WEAK_HASH_PATTERNS:
            if re.search(pattern, line, re.IGNORECASE):
                findings.append(Finding(
                    severity="HIGH",
                    category="Weak Hashing",
                    title=f"Use of weak hash algorithm: {algo_name}",
                    description=f"{algo_name} is cryptographically broken and should not be used for security purposes (signatures, integrity, password hashing).",
                    file_path=file_path,
                    line_number=line_num,
                    code_snippet=stripped[:200],
                    remediation=f"Replace {algo_name} with SHA-256 or SHA-3. For password hashing, use Argon2id or bcrypt.",
                    cwe_id=cwe,
                ))

        # Weak ciphers and modes
        for pattern, algo_name, cwe in WEAK_CIPHER_PATTERNS:
            if re.search(pattern, line):
                findings.append(Finding(
                    severity="HIGH",
                    category="Weak Encryption",
                    title=f"Use of insecure cipher or mode: {algo_name}",
                    description=f"{algo_name} is deprecated or insecure. ECB mode leaks patterns in ciphertext.",
                    file_path=file_path,
                    line_number=line_num,
                    code_snippet=stripped[:200],
                    remediation="Use AES-256-GCM for authenticated encryption. Never use ECB mode.",
                    cwe_id=cwe,
                ))

        # Hardcoded secrets
        for pattern, desc, cwe in HARDCODED_SECRET_PATTERNS:
            if re.search(pattern, line, re.IGNORECASE):
                sanitized = re.sub(r'["\'][^"\']+["\']', '"***REDACTED***"', stripped)
                findings.append(Finding(
                    severity="CRITICAL",
                    category="Hardcoded Secret",
                    title=f"Potential hardcoded secret: {desc}",
                    description="Secrets should never be hardcoded in source code. They can be extracted from version control history.",
                    file_path=file_path,
                    line_number=line_num,
                    code_snippet=sanitized[:200],
                    remediation="Store secrets in environment variables, AWS Secrets Manager, HashiCorp Vault, or similar.",
                    cwe_id=cwe,
                ))

        # Weak KDF parameters
        for pattern, desc, cwe in WEAK_KDF_PATTERNS:
            match = re.search(pattern, line)
            if match:
                try:
                    iterations = int(match.group(1))
                    if iterations < 100_000:
                        findings.append(Finding(
                            severity="HIGH",
                            category="Weak Key Derivation",
                            title=f"Insufficient KDF iterations: {iterations}",
                            description=f"PBKDF2 with {iterations} iterations is too low. OWASP recommends minimum 600,000 for PBKDF2-SHA256.",
                            file_path=file_path,
                            line_number=line_num,
                            code_snippet=stripped[:200],
                            remediation="Increase PBKDF2 iterations to 600,000+ or switch to Argon2id.",
                            cwe_id=cwe,
                        ))
                except (ValueError, IndexError):
                    pass

        # Insecure random
        for pattern, desc, cwe in INSECURE_RANDOM_PATTERNS:
            if re.search(pattern, line):
                if "import" not in stripped:
                    findings.append(Finding(
                        severity="HIGH",
                        category="Insecure Randomness",
                        title=f"Insecure random number generation: {desc}",
                        description="Python's random module is not cryptographically secure. Use os.urandom() or secrets module.",
                        file_path=file_path,
                        line_number=line_num,
                        code_snippet=stripped[:200],
                        remediation="Use os.urandom(), secrets.token_bytes(), or secrets.token_hex() for cryptographic operations.",
                        cwe_id=cwe,
                    ))

        # Deprecated TLS
        for pattern, desc, cwe in DEPRECATED_TLS_PATTERNS:
            if re.search(pattern, line):
                findings.append(Finding(
                    severity="HIGH",
                    category="Deprecated Protocol",
                    title=f"Insecure TLS/SSL configuration: {desc}",
                    description=f"Deprecated or insecure protocol/configuration detected. {desc} is vulnerable to known attacks.",
                    file_path=file_path,
                    line_number=line_num,
                    code_snippet=stripped[:200],
                    remediation="Use TLS 1.2+ with strong cipher suites. Enable certificate verification and hostname checking.",
                    cwe_id=cwe,
                ))

    return findings


def scan_directory(target_dir: str, extensions: Optional[List[str]] = None) -> List[Finding]:
    """Scan all matching files in a directory."""
    if extensions is None:
        extensions = [".py", ".js", ".ts", ".java", ".go", ".yml", ".yaml", ".json", ".conf", ".cfg", ".ini", ".env"]

    all_findings = []
    target_path = Path(target_dir)

    if target_path.is_file():
        return scan_file(str(target_path))

    for ext in extensions:
        for file in target_path.rglob(f"*{ext}"):
            if ".git" in str(file) or "node_modules" in str(file) or "__pycache__" in str(file):
                continue
            findings = scan_file(str(file))
            all_findings.extend(findings)

    return all_findings


def generate_report(findings: List[Finding], target: str) -> Dict:
    """Generate a structured audit report."""
    severity_counts = {"CRITICAL": 0, "HIGH": 0, "MEDIUM": 0, "LOW": 0, "INFO": 0}
    category_counts = {}

    for f in findings:
        severity_counts[f.severity] = severity_counts.get(f.severity, 0) + 1
        category_counts[f.category] = category_counts.get(f.category, 0) + 1

    overall_risk = "LOW"
    if severity_counts["CRITICAL"] > 0:
        overall_risk = "CRITICAL"
    elif severity_counts["HIGH"] > 0:
        overall_risk = "HIGH"
    elif severity_counts["MEDIUM"] > 0:
        overall_risk = "MEDIUM"

    return {
        "audit_target": target,
        "total_findings": len(findings),
        "overall_risk": overall_risk,
        "severity_summary": severity_counts,
        "category_summary": category_counts,
        "findings": [asdict(f) for f in findings],
        "recommendations": [
            "Replace all deprecated hash algorithms (MD5, SHA-1) with SHA-256 or SHA-3",
            "Replace DES/3DES/RC4 with AES-256-GCM",
            "Never use ECB mode; use GCM for authenticated encryption",
            "Move all hardcoded secrets to a secrets manager",
            "Use PBKDF2 with 600,000+ iterations or switch to Argon2id",
            "Use os.urandom() or secrets module for cryptographic randomness",
            "Enforce TLS 1.2+ and disable all legacy protocols",
            "Enable certificate verification and hostname checking",
        ],
    }


def test_with_samples():
    """Test the scanner with known-bad samples."""
    samples = '''
import hashlib
import random

password = "SuperSecret123!"
api_key = "sk-abcdef1234567890abcdef"

def hash_password(pw):
    return hashlib.md5(pw.encode()).hexdigest()

def generate_token():
    return random.randint(100000, 999999)

from Crypto.Cipher import AES
cipher = AES.new(key, AES.MODE_ECB)

from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
kdf = PBKDF2HMAC(algorithm=hashes.SHA256(), length=32, salt=salt, iterations=1000)

import ssl
ctx = ssl.SSLContext(ssl.PROTOCOL_SSLv23)
ctx.verify_mode = ssl.CERT_NONE
ctx.check_hostname = False
'''
    # Write temp sample file
    sample_path = Path("__crypto_audit_test_sample.py")
    sample_path.write_text(samples)

    findings = scan_file(str(sample_path))
    report = generate_report(findings, str(sample_path))

    print(json.dumps(report, indent=2))
    print(f"\nTotal findings: {report['total_findings']}")
    print(f"Overall risk: {report['overall_risk']}")

    sample_path.unlink()
    return report


def main():
    parser = argparse.ArgumentParser(description="Cryptographic Audit Scanner")
    subparsers = parser.add_subparsers(dest="command")

    scan = subparsers.add_parser("scan", help="Scan target for crypto weaknesses")
    scan.add_argument("--target", "-t", required=True, help="File or directory to scan")
    scan.add_argument("--output", "-o", help="Output report file (JSON)")

    subparsers.add_parser("test-samples", help="Test with built-in weak samples")

    args = parser.parse_args()

    if args.command == "scan":
        findings = scan_directory(args.target)
        report = generate_report(findings, args.target)
        if args.output:
            Path(args.output).write_text(json.dumps(report, indent=2))
            logger.info(f"Report saved to {args.output}")
        print(json.dumps(report, indent=2))
    elif args.command == "test-samples":
        test_with_samples()
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-hardware-security-module-integration/SKILL.md =====
---
name: performing-hardware-security-module-integration
description: Integrates Hardware Security Modules (HSMs) via the PKCS#11 interface
  using python-pkcs11, performing key generation, signing, encryption, verification,
  and token/slot queries against SoftHSM2, AWS CloudHSM, or YubiHSM2. Use when implementing
  HSM-backed key management or validating HSM configuration for FIPS 140-2/3 compliance.
domain: cybersecurity
subdomain: cryptography
tags:
- HSM
- PKCS11
- CloudHSM
- YubiHSM2
- key-management
- cryptographic-operations
- hardware-security
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
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
- T1078.004
- T1530
---

# Performing Hardware Security Module Integration

## Overview

Hardware Security Modules (HSMs) provide tamper-resistant cryptographic key storage and operations. This skill covers integrating with HSMs via the PKCS#11 standard interface using python-pkcs11, performing key generation, signing, encryption, and verification operations, querying token and slot information, and validating HSM configuration for compliance with FIPS 140-2/3 requirements.


## When to Use

- When conducting security assessments that involve performing hardware security module integration
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- HSM device or software HSM (SoftHSM2 for testing)
- PKCS#11 shared library (.so/.dll) for the HSM vendor
- Python 3.9+ with `python-pkcs11`
- Token initialized with SO PIN and user PIN
- For AWS CloudHSM: `cloudhsm-pkcs11` provider configured

## Steps

1. Load PKCS#11 library and enumerate available slots and tokens
2. Open session and authenticate with user PIN
3. Generate RSA 2048-bit or EC P-256 key pairs on the HSM
4. Perform signing and verification using on-device keys
5. List all objects (keys, certificates) stored on the token
6. Query mechanism list to verify supported algorithms
7. Generate compliance report with key inventory and algorithm audit

## Expected Output

- JSON report listing HSM slots, tokens, stored keys, supported mechanisms, and compliance status
- Signing test results with key metadata and algorithm details

===== END FILE =====


===== FILE: skills/performing-hardware-security-module-integration/LICENSE =====

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


===== FILE: skills/performing-hardware-security-module-integration/references/api-reference.md =====
# API Reference — Performing Hardware Security Module Integration

## Libraries Used
- **python-pkcs11**: Python PKCS#11 wrapper for HSM cryptographic operations
- **json**: JSON serialization for audit reports

## CLI Interface
```
python agent.py --lib /usr/lib/softhsm/libsofthsm2.so --token MyToken --pin 1234 slots
python agent.py --lib /usr/lib/softhsm/libsofthsm2.so --token MyToken --pin 1234 objects
python agent.py --lib /usr/lib/softhsm/libsofthsm2.so --token MyToken --pin 1234 gen-rsa --label mykey --bits 2048
python agent.py --lib /usr/lib/softhsm/libsofthsm2.so --token MyToken --pin 1234 gen-ec --label myec
python agent.py --lib /usr/lib/softhsm/libsofthsm2.so --token MyToken --pin 1234 sign-verify --key-label mykey
python agent.py --lib /usr/lib/softhsm/libsofthsm2.so --token MyToken --pin 1234 mechanisms
python agent.py --lib /usr/lib/softhsm/libsofthsm2.so --token MyToken --pin 1234 full
```

## Core Functions

### `load_library(lib_path)` — Load PKCS#11 shared library
Calls `pkcs11.lib(lib_path)` to initialize the PKCS#11 provider.

### `enumerate_slots(lib)` — List slots and token info
Iterates `lib.get_slots(token_present=True)`. Returns token label, manufacturer,
model, serial, initialization status, and supported mechanism list.

### `list_objects(lib, token_label, pin)` — Inventory stored keys
Opens authenticated session, calls `session.get_objects()`. Returns object class,
label, key type, key length, and object ID.

### `generate_rsa_keypair(lib, token_label, pin, key_label, bits)` — RSA key generation
Calls `session.generate_keypair(KeyType.RSA, bits, store=True, label=key_label)`.

### `generate_ec_keypair(lib, token_label, pin, key_label)` — EC P-256 key generation
Creates domain parameters for secp256r1 via `encode_named_curve_parameters`,
then calls `ecparams.generate_keypair()`.

### `sign_and_verify(lib, token_label, pin, key_label)` — Signing test
Signs with `priv.sign(data, mechanism=Mechanism.SHA256_RSA_PKCS)`.
Verifies with `pub.verify(data, signature, mechanism=Mechanism.SHA256_RSA_PKCS)`.

### `query_mechanisms(lib, token_label)` — Algorithm support audit
Enumerates all mechanisms with min/max key sizes from the slot.

### `full_audit(lib, token_label, pin)` — Comprehensive compliance report

## PKCS#11 Object Classes
| Class | Description |
|-------|-------------|
| PUBLIC_KEY | RSA/EC public keys |
| PRIVATE_KEY | RSA/EC private keys (non-extractable) |
| SECRET_KEY | Symmetric keys (AES, DES3) |
| CERTIFICATE | X.509 certificates |

## FIPS 140-2 Required Mechanisms
RSA_PKCS, SHA256_RSA_PKCS, SHA384_RSA_PKCS, SHA512_RSA_PKCS,
ECDSA, ECDSA_SHA256, AES_CBC, AES_GCM, SHA256, SHA384, SHA512

## Common PKCS#11 Libraries
| HSM | Library Path |
|-----|-------------|
| SoftHSM2 | `/usr/lib/softhsm/libsofthsm2.so` |
| AWS CloudHSM | `/opt/cloudhsm/lib/libcloudhsm_pkcs11.so` |
| YubiHSM2 | `/usr/lib/x86_64-linux-gnu/pkcs11/yubihsm_pkcs11.so` |
| Thales Luna | `/usr/safenet/lunaclient/lib/libCryptoki2_64.so` |

## Dependencies
- `python-pkcs11` >= 0.7.0
- PKCS#11 shared library for target HSM
- Initialized token with user PIN

===== END FILE =====


===== FILE: skills/performing-hardware-security-module-integration/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for Hardware Security Module integration via PKCS#11 interface."""

import json
import argparse
from datetime import datetime

try:
    import pkcs11
    from pkcs11 import KeyType, ObjectClass, Mechanism
except ImportError:
    pkcs11 = None


def load_library(lib_path):
    """Load PKCS#11 shared library."""
    if not pkcs11:
        raise RuntimeError("python-pkcs11 not installed: pip install python-pkcs11")
    return pkcs11.lib(lib_path)


def enumerate_slots(lib):
    """List all available PKCS#11 slots and token info."""
    slots = []
    for slot in lib.get_slots(token_present=True):
        token = slot.get_token()
        mechs = slot.get_mechanisms()
        slots.append({
            "slot_id": slot.slot_id,
            "slot_description": slot.slot_description.strip() if hasattr(slot, 'slot_description') else str(slot),
            "token_label": token.label.strip(),
            "token_manufacturer": token.manufacturer_id.strip(),
            "token_model": token.model.strip(),
            "token_serial": token.serial.strip(),
            "token_initialized": token.flags & pkcs11.TokenFlag.TOKEN_INITIALIZED != 0,
            "mechanism_count": len(list(mechs)),
            "supported_mechanisms": sorted([m.name for m in mechs])[:30],
        })
    return slots


def list_objects(lib, token_label, pin):
    """List all cryptographic objects stored on the HSM token."""
    token = lib.get_token(token_label=token_label)
    with token.open(user_pin=pin) as session:
        objects = []
        for obj in session.get_objects():
            obj_info = {
                "object_class": obj.object_class.name if hasattr(obj.object_class, 'name') else str(obj.object_class),
                "label": getattr(obj, 'label', 'N/A'),
            }
            if hasattr(obj, 'key_type'):
                obj_info["key_type"] = obj.key_type.name if hasattr(obj.key_type, 'name') else str(obj.key_type)
            if hasattr(obj, 'key_length'):
                obj_info["key_length"] = obj.key_length
            if hasattr(obj, 'id'):
                obj_info["id"] = obj.id.hex() if isinstance(obj.id, bytes) else str(obj.id)
            objects.append(obj_info)
        return objects


def generate_rsa_keypair(lib, token_label, pin, key_label="agent-rsa-2048", bits=2048):
    """Generate an RSA key pair on the HSM."""
    token = lib.get_token(token_label=token_label)
    with token.open(rw=True, user_pin=pin) as session:
        pub, priv = session.generate_keypair(
            KeyType.RSA, bits,
            store=True,
            label=key_label,
        )
        return {
            "action": "generate_rsa_keypair",
            "key_label": key_label,
            "key_size": bits,
            "public_key_class": pub.object_class.name,
            "private_key_class": priv.object_class.name,
            "status": "SUCCESS",
        }


def generate_ec_keypair(lib, token_label, pin, key_label="agent-ec-p256"):
    """Generate an EC P-256 key pair on the HSM."""
    token = lib.get_token(token_label=token_label)
    with token.open(rw=True, user_pin=pin) as session:
        ecparams = session.create_domain_parameters(
            KeyType.EC,
            {pkcs11.Attribute.EC_PARAMS: pkcs11.util.ec.encode_named_curve_parameters("secp256r1")},
            local=True,
        )
        pub, priv = ecparams.generate_keypair(store=True, label=key_label)
        return {
            "action": "generate_ec_keypair",
            "key_label": key_label,
            "curve": "secp256r1 (P-256)",
            "public_key_class": pub.object_class.name,
            "private_key_class": priv.object_class.name,
            "status": "SUCCESS",
        }


def sign_and_verify(lib, token_label, pin, key_label, data=b"HSM test message"):
    """Sign data with an RSA private key and verify with the public key."""
    token = lib.get_token(token_label=token_label)
    with token.open(user_pin=pin) as session:
        priv_keys = list(session.get_objects({
            pkcs11.Attribute.CLASS: ObjectClass.PRIVATE_KEY,
            pkcs11.Attribute.LABEL: key_label,
        }))
        if not priv_keys:
            return {"error": f"Private key '{key_label}' not found"}
        priv = priv_keys[0]
        signature = priv.sign(data, mechanism=Mechanism.SHA256_RSA_PKCS)

        pub_keys = list(session.get_objects({
            pkcs11.Attribute.CLASS: ObjectClass.PUBLIC_KEY,
            pkcs11.Attribute.LABEL: key_label,
        }))
        if not pub_keys:
            return {"error": f"Public key '{key_label}' not found"}
        pub = pub_keys[0]
        try:
            pub.verify(data, signature, mechanism=Mechanism.SHA256_RSA_PKCS)
            verified = True
        except Exception:
            verified = False

        return {
            "action": "sign_and_verify",
            "key_label": key_label,
            "data_length": len(data),
            "signature_length": len(signature),
            "signature_hex": signature[:32].hex() + "...",
            "mechanism": "SHA256_RSA_PKCS",
            "verified": verified,
        }


def query_mechanisms(lib, token_label):
    """List all supported mechanisms for the token's slot."""
    token = lib.get_token(token_label=token_label)
    slot = token.slot
    mechs = []
    for m in slot.get_mechanisms():
        info = slot.get_mechanism_info(m)
        mechs.append({
            "mechanism": m.name,
            "min_key_size": info.min_key_size if hasattr(info, 'min_key_size') else None,
            "max_key_size": info.max_key_size if hasattr(info, 'max_key_size') else None,
        })
    return mechs


def full_audit(lib, token_label, pin):
    """Run comprehensive HSM compliance audit."""
    slots = enumerate_slots(lib)
    objects = list_objects(lib, token_label, pin)
    mechanisms = query_mechanisms(lib, token_label)
    rsa_keys = [o for o in objects if o.get("key_type") == "RSA"]
    ec_keys = [o for o in objects if o.get("key_type") == "EC"]
    weak_keys = [o for o in objects if o.get("key_type") == "RSA" and (o.get("key_length") or 2048) < 2048]
    fips_mechs = {"RSA_PKCS", "SHA256_RSA_PKCS", "SHA384_RSA_PKCS", "SHA512_RSA_PKCS",
                  "ECDSA", "ECDSA_SHA256", "AES_CBC", "AES_GCM", "SHA256", "SHA384", "SHA512"}
    supported_names = {m["mechanism"] for m in mechanisms}
    fips_coverage = len(fips_mechs & supported_names)
    return {
        "audit_type": "HSM PKCS#11 Compliance Audit",
        "timestamp": datetime.utcnow().isoformat(),
        "slots": slots,
        "stored_objects": len(objects),
        "objects": objects[:30],
        "rsa_key_count": len(rsa_keys),
        "ec_key_count": len(ec_keys),
        "weak_rsa_keys": len(weak_keys),
        "total_mechanisms": len(mechanisms),
        "fips_mechanism_coverage": f"{fips_coverage}/{len(fips_mechs)}",
        "compliance": "PASS" if not weak_keys and fips_coverage >= 6 else "REVIEW",
    }


def main():
    parser = argparse.ArgumentParser(description="HSM PKCS#11 Integration Agent")
    parser.add_argument("--lib", required=True, help="Path to PKCS#11 shared library")
    parser.add_argument("--token", required=True, help="Token label")
    parser.add_argument("--pin", required=True, help="User PIN")
    sub = parser.add_subparsers(dest="command")
    sub.add_parser("slots", help="Enumerate PKCS#11 slots and tokens")
    sub.add_parser("objects", help="List stored cryptographic objects")
    p_rsa = sub.add_parser("gen-rsa", help="Generate RSA key pair")
    p_rsa.add_argument("--label", default="agent-rsa-2048")
    p_rsa.add_argument("--bits", type=int, default=2048)
    p_ec = sub.add_parser("gen-ec", help="Generate EC P-256 key pair")
    p_ec.add_argument("--label", default="agent-ec-p256")
    p_sign = sub.add_parser("sign-verify", help="Sign and verify test data")
    p_sign.add_argument("--key-label", required=True)
    sub.add_parser("mechanisms", help="List supported mechanisms")
    sub.add_parser("full", help="Full HSM compliance audit")
    args = parser.parse_args()

    lib = load_library(args.lib)

    if args.command == "slots":
        result = enumerate_slots(lib)
    elif args.command == "objects":
        result = list_objects(lib, args.token, args.pin)
    elif args.command == "gen-rsa":
        result = generate_rsa_keypair(lib, args.token, args.pin, args.label, args.bits)
    elif args.command == "gen-ec":
        result = generate_ec_keypair(lib, args.token, args.pin, args.label)
    elif args.command == "sign-verify":
        result = sign_and_verify(lib, args.token, args.pin, args.key_label)
    elif args.command == "mechanisms":
        result = query_mechanisms(lib, args.token)
    elif args.command == "full":
        result = full_audit(lib, args.token, args.pin)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-hash-cracking-with-hashcat/SKILL.md =====
---
name: performing-hash-cracking-with-hashcat
description: Cracks password hashes with Hashcat, covering hash-type identification,
  dictionary/brute-force/rule-based attack modes, custom rule creation, GPU benchmarking,
  and password-strength/compliance reporting. Use for authorized penetration testing
  or security audits that need to evaluate password strength or crack captured hashes.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- hash-cracking
- password-security
- hashcat
- penetration-testing
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
---
# Performing Hash Cracking with Hashcat

## Overview

Hash cracking is an essential skill for penetration testers and security auditors to evaluate password strength. Hashcat is the world's fastest password recovery tool, supporting over 300 hash types with GPU acceleration. This skill covers using hashcat for authorized password auditing, understanding attack modes, creating effective rule sets, and generating hash analysis reports. This is strictly for authorized penetration testing and password policy assessment.


## When to Use

- When conducting security assessments that involve performing hash cracking with hashcat
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Identify hash types from captured hashes
- Execute dictionary, brute-force, and rule-based attacks
- Create custom hashcat rules for targeted cracking
- Analyze password strength from cracking results
- Generate compliance reports on password policy effectiveness
- Benchmark GPU performance for hash cracking

## Key Concepts

### Hashcat Attack Modes

| Mode | Flag | Description | Use Case |
|------|------|-------------|----------|
| Dictionary | -a 0 | Wordlist attack | Known password patterns |
| Combination | -a 1 | Combine two wordlists | Compound passwords |
| Brute-force | -a 3 | Mask-based enumeration | Short passwords |
| Rule-based | -a 0 -r | Dictionary + transformation rules | Complex variations |
| Hybrid | -a 6/7 | Wordlist + mask | Passwords with appended numbers |

### Common Hash Types

| Hash Mode | Type | Example Use |
|-----------|------|-------------|
| 0 | MD5 | Legacy web apps |
| 100 | SHA-1 | Legacy systems |
| 1000 | NTLM | Windows credentials |
| 1800 | sha512crypt | Linux /etc/shadow |
| 3200 | bcrypt | Modern web apps |
| 13100 | Kerberos TGS-REP | Active Directory |

## Security Considerations

- Only perform hash cracking with explicit written authorization
- Secure all captured hash data in transit and at rest
- Report all cracked passwords immediately to asset owners
- Use results to improve password policies, not exploit users
- Destroy cracked password data after engagement concludes
- Follow rules of engagement for penetration test scope

## Validation Criteria

- [ ] Hash type identification is correct
- [ ] Dictionary attack cracks weak passwords
- [ ] Rule-based attack cracks policy-compliant passwords
- [ ] Mask attack cracks short passwords
- [ ] Results report shows password strength distribution
- [ ] All operations performed within authorized scope

===== END FILE =====


===== FILE: skills/performing-hash-cracking-with-hashcat/LICENSE =====

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


===== FILE: skills/performing-hash-cracking-with-hashcat/assets/template.md =====
# Hash Cracking Assessment Template

## Authorization Verification

- [ ] Written authorization obtained from asset owner
- [ ] Scope of engagement defines which hashes may be cracked
- [ ] Data handling requirements documented
- [ ] Results reporting chain defined
- [ ] Data destruction timeline agreed

## Hashcat Quick Reference

```bash
# Identify hash type
hashcat --identify hash.txt

# MD5 dictionary
hashcat -m 0 -a 0 hashes.txt wordlist.txt

# NTLM with rules
hashcat -m 1000 -a 0 hashes.txt wordlist.txt -r rules/best64.rule

# bcrypt dictionary (slow)
hashcat -m 3200 -a 0 hashes.txt wordlist.txt

# Benchmark
hashcat -b
```

## Mask Charsets

| Charset | Characters | Symbol |
|---------|-----------|--------|
| ?l | a-z | lowercase |
| ?u | A-Z | uppercase |
| ?d | 0-9 | digits |
| ?s | special chars | symbols |
| ?a | ?l?u?d?s | all printable |

## Reporting Template

| Metric | Value |
|--------|-------|
| Total hashes | [count] |
| Cracked | [count] ([%]) |
| < 1 minute | [count] (CRITICAL) |
| < 1 hour | [count] (HIGH) |
| < 1 day | [count] (MEDIUM) |
| Not cracked | [count] (OK) |

===== END FILE =====


===== FILE: skills/performing-hash-cracking-with-hashcat/references/api-reference.md =====
# API Reference — Performing Hash Cracking with Hashcat

## Libraries Used
- **subprocess**: Execute hashcat with various attack modes
- **hashlib**: Generate test hashes (MD5, SHA1, SHA256, SHA512)
- **re**: Pattern matching for hash type identification
- **pathlib**: Read hash files and potfiles

## CLI Interface
```
python agent.py identify --hash <hash_string>
python agent.py identify --file hashes.txt
python agent.py crack --hash-file hashes.txt --mode 0 --attack dictionary --wordlist rockyou.txt [--rules best64.rule]
python agent.py crack --hash-file hashes.txt --mode 1000 --attack brute --mask "?u?l?l?l?d?d?d?s"
python agent.py parse --potfile hashcat.potfile
python agent.py gen --text "password123" --algo sha256
```

## Core Functions

### `identify_hash(hash_string)` — Detect hash type and hashcat mode
Matches against 12 patterns: MD5 (0), SHA1 (100), SHA256 (1400), SHA512 (1700),
NTLM (1000), bcrypt (3200), sha512crypt (1800), md5crypt (500), NetNTLMv2 (5600),
Kerberos TGS (13100), Kerberos AS-REP (18200).

### `run_hashcat(hash_file, mode, attack, wordlist, rules, mask)` — Execute hashcat
Attack modes: `dictionary` (-a 0), `brute` (-a 3), `combinator` (-a 1).

### `parse_hashcat_status(potfile)` — Analyze cracked passwords
Returns length distribution, charset analysis, top passwords from potfile.

### `generate_hash(plaintext, algorithm)` — Create test hashes
Supported: md5, sha1, sha256, sha512.

## Hashcat Mask Charsets
- `?l` lowercase, `?u` uppercase, `?d` digits, `?s` special, `?a` all

## Dependencies
System: hashcat (GPU-accelerated hash cracking)
No Python packages required — standard library only.

===== END FILE =====


===== FILE: skills/performing-hash-cracking-with-hashcat/references/standards.md =====
# Standards and References - Hash Cracking with Hashcat

## Tools

### Hashcat
- **URL**: https://hashcat.net/hashcat/
- **GitHub**: https://github.com/hashcat/hashcat
- **Wiki**: https://hashcat.net/wiki/
- **Hash modes**: https://hashcat.net/wiki/doku.php?id=hashcat

### Wordlists
- **SecLists**: https://github.com/danielmiessler/SecLists
- **RockYou**: Classic breach wordlist (14 million passwords)
- **CrackStation**: https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm

## Standards

### NIST SP 800-63B - Digital Identity Guidelines (Authentication)
- **URL**: https://pages.nist.gov/800-63-3/sp800-63b.html
- **Description**: Password requirements and memorized secret guidelines

### OWASP Password Storage Cheat Sheet
- **URL**: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html

## Legal Framework

- Computer Fraud and Abuse Act (CFAA) - US
- Computer Misuse Act 1990 - UK
- Always obtain written authorization before testing
- Follow penetration testing rules of engagement

===== END FILE =====


===== FILE: skills/performing-hash-cracking-with-hashcat/references/workflows.md =====
# Workflows - Hash Cracking with Hashcat

## Workflow 1: Hash Identification and Cracking

```
[Captured Hashes]
      |
[Identify Hash Type]
(hashcat --identify, hashid, haiti)
      |
[Select Attack Strategy]:
  1. Dictionary attack (rockyou.txt)
  2. Dictionary + rules (best64.rule)
  3. Mask attack (brute-force)
  4. Hybrid (wordlist + mask)
      |
[Execute Hashcat]
      |
[Analyze Results]
(cracked count, time, patterns)
      |
[Generate Password Strength Report]
```

## Workflow 2: Hashcat Command Examples

```bash
# Dictionary attack
hashcat -m 0 -a 0 hashes.txt rockyou.txt

# Dictionary + rules
hashcat -m 0 -a 0 hashes.txt rockyou.txt -r best64.rule

# Mask (brute-force 8-char lowercase)
hashcat -m 0 -a 3 hashes.txt ?l?l?l?l?l?l?l?l

# Hybrid (wordlist + 4 digits)
hashcat -m 0 -a 6 hashes.txt rockyou.txt ?d?d?d?d

# Show cracked passwords
hashcat -m 0 hashes.txt --show
```

## Workflow 3: Password Audit Report

```
[Cracking Results]
      |
[Categorize]:
  - Cracked in < 1 min: CRITICAL
  - Cracked in < 1 hour: HIGH
  - Cracked in < 1 day: MEDIUM
  - Not cracked: Acceptable
      |
[Pattern Analysis]:
  - Common base words
  - Character set distribution
  - Length distribution
  - Policy compliance
      |
[Generate Report with Recommendations]
```

===== END FILE =====


===== FILE: skills/performing-hash-cracking-with-hashcat/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing hash cracking with hashcat — hash identification, attack management, and result analysis."""

import json
import argparse
import subprocess
import hashlib
import re
from collections import Counter
from pathlib import Path


HASH_PATTERNS = {
    "MD5": (r"^[a-f0-9]{32}$", 0),
    "SHA1": (r"^[a-f0-9]{40}$", 100),
    "SHA256": (r"^[a-f0-9]{64}$", 1400),
    "SHA512": (r"^[a-f0-9]{128}$", 1700),
    "NTLM": (r"^[a-f0-9]{32}$", 1000),
    "bcrypt": (r"^\$2[aby]?\$\d+\$.{53}$", 3200),
    "sha512crypt": (r"^\$6\$[^\$]+\$[a-zA-Z0-9./]{86}$", 1800),
    "sha256crypt": (r"^\$5\$[^\$]+\$[a-zA-Z0-9./]{43}$", 7400),
    "md5crypt": (r"^\$1\$[^\$]+\$[a-zA-Z0-9./]{22}$", 500),
    "NetNTLMv2": (r"^[^:]+::\S+:[a-f0-9]{16}:[a-f0-9]{32}:[a-f0-9]+$", 5600),
    "Kerberos_TGS": (r"^\$krb5tgs\$", 13100),
    "Kerberos_AS": (r"^\$krb5asrep\$", 18200),
}


def identify_hash(hash_string):
    """Identify hash type and return hashcat mode."""
    candidates = []
    for name, (pattern, mode) in HASH_PATTERNS.items():
        if re.match(pattern, hash_string.strip(), re.IGNORECASE):
            candidates.append({"type": name, "hashcat_mode": mode})
    return {"hash": hash_string[:40] + "..." if len(hash_string) > 40 else hash_string, "candidates": candidates}


def identify_hashes_file(hash_file):
    """Identify hash types from a file of hashes."""
    hashes = Path(hash_file).read_text(encoding="utf-8", errors="replace").strip().splitlines()
    results = []
    for h in hashes[:100]:
        h = h.strip()
        if h:
            results.append(identify_hash(h))
    types = {}
    for r in results:
        for c in r["candidates"]:
            types[c["type"]] = types.get(c["type"], 0) + 1
    return {"total_hashes": len(results), "type_distribution": types, "samples": results[:5]}


def run_hashcat(hash_file, mode, attack="dictionary", wordlist=None, rules=None, mask=None):
    """Execute hashcat with specified attack mode."""
    cmd = ["hashcat", "-m", str(mode), "--quiet", "--potfile-disable", "-o", "/tmp/hashcat_out.txt"]
    if attack == "dictionary":
        if not wordlist:
            return {"error": "wordlist required for dictionary attack"}
        cmd += ["-a", "0", hash_file, wordlist]
        if rules:
            cmd += ["-r", rules]
    elif attack == "brute":
        m = mask or "?a?a?a?a?a?a?a?a"
        cmd += ["-a", "3", hash_file, m]
    elif attack == "combinator":
        if not wordlist:
            return {"error": "two wordlists required (comma-separated)"}
        wl = wordlist.split(",")
        if len(wl) != 2:
            return {"error": "combinator needs two wordlists: list1.txt,list2.txt"}
        cmd += ["-a", "1", hash_file, wl[0], wl[1]]
    else:
        return {"error": f"Unknown attack type: {attack}"}
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=3600)
        cracked = []
        out_file = Path("/tmp/hashcat_out.txt")
        if out_file.exists():
            for line in out_file.read_text().strip().splitlines():
                parts = line.split(":", 1)
                if len(parts) == 2:
                    cracked.append({"hash": parts[0][:30] + "...", "plain": parts[1]})
        return {
            "attack": attack, "mode": mode, "cracked_count": len(cracked),
            "cracked": cracked[:50], "return_code": result.returncode,
            "stderr_snippet": result.stderr[:300] if result.stderr else "",
        }
    except FileNotFoundError:
        return {"error": "hashcat not found in PATH"}
    except subprocess.TimeoutExpired:
        return {"error": "hashcat timed out after 3600s"}


def parse_hashcat_status(potfile):
    """Parse hashcat potfile for cracked results."""
    cracked = []
    try:
        for line in Path(potfile).read_text(encoding="utf-8", errors="replace").strip().splitlines():
            parts = line.split(":", 1)
            if len(parts) == 2:
                cracked.append({"hash": parts[0], "plain": parts[1]})
    except FileNotFoundError:
        return {"error": f"Potfile not found: {potfile}"}
    passwords = [c["plain"] for c in cracked]
    length_dist = {}
    for p in passwords:
        l = len(p)
        bucket = f"{l}" if l <= 8 else "9-12" if l <= 12 else "13+"
        length_dist[bucket] = length_dist.get(bucket, 0) + 1
    charset = {"lowercase_only": 0, "uppercase_mixed": 0, "with_digits": 0, "with_special": 0}
    for p in passwords:
        if re.match(r"^[a-z]+$", p):
            charset["lowercase_only"] += 1
        elif re.search(r"\d", p):
            charset["with_digits"] += 1
        elif re.search(r"[!@#$%^&*()_+=\-\[\]{};:'\",.<>?/\\|`~]", p):
            charset["with_special"] += 1
        else:
            charset["uppercase_mixed"] += 1
    return {
        "total_cracked": len(cracked),
        "length_distribution": length_dist,
        "charset_analysis": charset,
        "top_passwords": [p for p, _ in Counter(passwords).most_common(10)],
    }


def generate_hash(plaintext, algorithm="sha256"):
    """Generate hash of a plaintext string for testing."""
    algos = {"md5": hashlib.md5, "sha1": hashlib.sha1, "sha256": hashlib.sha256, "sha512": hashlib.sha512}
    if algorithm not in algos:
        return {"error": f"Unsupported: {algorithm}. Use: {list(algos.keys())}"}
    h = algos[algorithm](plaintext.encode()).hexdigest()
    return {"plaintext": plaintext, "algorithm": algorithm, "hash": h}


def main():
    parser = argparse.ArgumentParser(description="Hashcat Hash Cracking Agent")
    sub = parser.add_subparsers(dest="command")
    i = sub.add_parser("identify", help="Identify hash type")
    i.add_argument("--hash", help="Single hash string")
    i.add_argument("--file", help="File containing hashes")
    c = sub.add_parser("crack", help="Run hashcat")
    c.add_argument("--hash-file", required=True)
    c.add_argument("--mode", type=int, required=True, help="Hashcat mode number")
    c.add_argument("--attack", default="dictionary", choices=["dictionary", "brute", "combinator"])
    c.add_argument("--wordlist", help="Wordlist path (or two comma-separated for combinator)")
    c.add_argument("--rules", help="Hashcat rules file")
    c.add_argument("--mask", help="Brute force mask")
    p = sub.add_parser("parse", help="Parse potfile results")
    p.add_argument("--potfile", required=True)
    g = sub.add_parser("gen", help="Generate test hash")
    g.add_argument("--text", required=True)
    g.add_argument("--algo", default="sha256")
    args = parser.parse_args()
    if args.command == "identify":
        result = identify_hashes_file(args.file) if args.file else identify_hash(args.hash) if args.hash else {"error": "provide --hash or --file"}
    elif args.command == "crack":
        result = run_hashcat(args.hash_file, args.mode, args.attack, args.wordlist, args.rules, args.mask)
    elif args.command == "parse":
        result = parse_hashcat_status(args.potfile)
    elif args.command == "gen":
        result = generate_hash(args.text, args.algo)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-hash-cracking-with-hashcat/scripts/process.py =====
#!/usr/bin/env python3
"""
Hash Cracking Analysis and Hashcat Automation Tool

Provides hash identification, hashcat command generation, result analysis,
and password strength reporting for authorized security assessments.

Requirements:
    pip install passlib argon2-cffi

Usage:
    python process.py identify --hash "5f4dcc3b5aa765d61d8327deb882cf99"
    python process.py generate-cmd --hash-file hashes.txt --mode 0 --attack dictionary
    python process.py analyze-results --potfile hashcat.potfile --hash-file hashes.txt
    python process.py create-test-hashes --output test_hashes.txt
"""

import os
import re
import sys
import json
import hashlib
import argparse
import logging
from pathlib import Path
from typing import Dict, List, Optional, Tuple
from collections import Counter

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

HASH_PATTERNS = {
    r"^[a-f0-9]{32}$": [("MD5", 0), ("NTLM", 1000)],
    r"^[a-f0-9]{40}$": [("SHA-1", 100)],
    r"^[a-f0-9]{64}$": [("SHA-256", 1400)],
    r"^[a-f0-9]{128}$": [("SHA-512", 1700)],
    r"^\$2[aby]\$\d{2}\$.{53}$": [("bcrypt", 3200)],
    r"^\$6\$": [("sha512crypt", 1800)],
    r"^\$5\$": [("sha256crypt", 7400)],
    r"^\$argon2(i|d|id)\$": [("Argon2", 32600)],
    r"^\$1\$": [("md5crypt", 500)],
    r"^\$apr1\$": [("Apache APR1", 1600)],
    r"^[a-f0-9]{32}:[a-f0-9]+$": [("MD5 salted", 10)],
    r"^\$krb5tgs\$": [("Kerberos TGS-REP", 13100)],
    r"^[a-f0-9]{32}:[a-f0-9]{32}$": [("NetNTLMv1", 5500)],
}


def identify_hash(hash_value: str) -> List[Dict]:
    """Identify the type of a hash value."""
    hash_value = hash_value.strip()
    matches = []

    for pattern, hash_types in HASH_PATTERNS.items():
        if re.match(pattern, hash_value, re.IGNORECASE):
            for name, mode in hash_types:
                matches.append({
                    "hash_type": name,
                    "hashcat_mode": mode,
                    "confidence": "high" if len(hash_types) == 1 else "medium",
                })

    if not matches:
        matches.append({
            "hash_type": "Unknown",
            "hashcat_mode": None,
            "confidence": "none",
        })

    return matches


def generate_hashcat_command(
    hash_file: str,
    mode: int,
    attack: str = "dictionary",
    wordlist: str = "rockyou.txt",
    rules: Optional[str] = None,
    mask: Optional[str] = None,
    output_file: Optional[str] = None,
) -> str:
    """Generate a hashcat command string."""
    cmd_parts = ["hashcat"]
    cmd_parts.extend(["-m", str(mode)])

    if attack == "dictionary":
        cmd_parts.extend(["-a", "0"])
        cmd_parts.append(hash_file)
        cmd_parts.append(wordlist)
        if rules:
            cmd_parts.extend(["-r", rules])
    elif attack == "bruteforce":
        cmd_parts.extend(["-a", "3"])
        cmd_parts.append(hash_file)
        cmd_parts.append(mask or "?a?a?a?a?a?a?a?a")
    elif attack == "hybrid_wm":
        cmd_parts.extend(["-a", "6"])
        cmd_parts.append(hash_file)
        cmd_parts.append(wordlist)
        cmd_parts.append(mask or "?d?d?d?d")
    elif attack == "hybrid_mw":
        cmd_parts.extend(["-a", "7"])
        cmd_parts.append(hash_file)
        cmd_parts.append(mask or "?d?d?d?d")
        cmd_parts.append(wordlist)

    if output_file:
        cmd_parts.extend(["-o", output_file])

    cmd_parts.extend(["--force", "--status", "--status-timer=10"])
    return " ".join(cmd_parts)


def create_test_hashes(output_path: str) -> Dict:
    """Create a set of test password hashes for practice."""
    test_passwords = [
        "password", "123456", "admin", "letmein", "welcome",
        "monkey", "dragon", "master", "qwerty", "login",
        "P@ssw0rd", "Summer2024!", "company123", "test1234",
        "hunter2", "trustno1", "batman", "shadow", "sunshine",
        "iloveyou",
    ]

    hashes = {"md5": [], "sha1": [], "sha256": []}
    for pwd in test_passwords:
        pwd_bytes = pwd.encode("utf-8")
        hashes["md5"].append(hashlib.md5(pwd_bytes).hexdigest())
        hashes["sha1"].append(hashlib.sha1(pwd_bytes).hexdigest())
        hashes["sha256"].append(hashlib.sha256(pwd_bytes).hexdigest())

    output = Path(output_path)
    for hash_type, hash_list in hashes.items():
        path = output.parent / f"{output.stem}_{hash_type}{output.suffix}"
        path.write_text("\n".join(hash_list) + "\n")
        logger.info(f"Created {path} with {len(hash_list)} hashes")

    return {
        "test_passwords_count": len(test_passwords),
        "hash_types": list(hashes.keys()),
        "output_directory": str(output.parent),
        "note": "These are intentionally weak passwords for authorized testing only",
    }


def analyze_cracked_results(potfile_path: str, hash_file_path: str) -> Dict:
    """Analyze hashcat results from potfile."""
    potfile = Path(potfile_path)
    hash_file = Path(hash_file_path)

    if not potfile.exists():
        return {"error": f"Potfile not found: {potfile_path}"}

    total_hashes = 0
    if hash_file.exists():
        total_hashes = sum(1 for line in hash_file.read_text().strip().split("\n") if line.strip())

    cracked_passwords = []
    for line in potfile.read_text().strip().split("\n"):
        if ":" in line:
            parts = line.rsplit(":", 1)
            if len(parts) == 2:
                cracked_passwords.append(parts[1])

    cracked_count = len(cracked_passwords)
    crack_rate = (cracked_count / total_hashes * 100) if total_hashes > 0 else 0

    # Password analysis
    lengths = [len(p) for p in cracked_passwords]
    charset_analysis = {"lowercase_only": 0, "with_uppercase": 0, "with_digits": 0, "with_special": 0}

    for pwd in cracked_passwords:
        has_upper = any(c.isupper() for c in pwd)
        has_digit = any(c.isdigit() for c in pwd)
        has_special = any(not c.isalnum() for c in pwd)
        if has_special:
            charset_analysis["with_special"] += 1
        elif has_digit and has_upper:
            charset_analysis["with_digits"] += 1
        elif has_upper:
            charset_analysis["with_uppercase"] += 1
        else:
            charset_analysis["lowercase_only"] += 1

    # Strength categories
    strength = {"critical": 0, "weak": 0, "moderate": 0}
    for pwd in cracked_passwords:
        if len(pwd) < 8:
            strength["critical"] += 1
        elif len(pwd) < 12 or not any(not c.isalnum() for c in pwd):
            strength["weak"] += 1
        else:
            strength["moderate"] += 1

    common_words = Counter()
    for pwd in cracked_passwords:
        base = re.sub(r"[^a-zA-Z]", "", pwd).lower()
        if len(base) >= 3:
            common_words[base] += 1

    return {
        "total_hashes": total_hashes,
        "cracked_count": cracked_count,
        "crack_rate_percent": round(crack_rate, 1),
        "uncracked_count": total_hashes - cracked_count,
        "password_length": {
            "min": min(lengths) if lengths else 0,
            "max": max(lengths) if lengths else 0,
            "avg": round(sum(lengths) / len(lengths), 1) if lengths else 0,
        },
        "charset_distribution": charset_analysis,
        "strength_distribution": strength,
        "top_base_words": dict(common_words.most_common(10)),
        "recommendations": [
            "Enforce minimum 12-character passwords",
            "Require mixed character sets (upper, lower, digit, special)",
            "Block common dictionary words and patterns",
            "Implement password breach checking (Have I Been Pwned API)",
            "Use Argon2id or bcrypt for password hashing (not MD5/SHA)",
            "Consider passphrase-based policies over complexity rules",
        ],
    }


def main():
    parser = argparse.ArgumentParser(description="Hash Cracking Analysis Tool")
    subparsers = parser.add_subparsers(dest="command")

    ident = subparsers.add_parser("identify", help="Identify hash type")
    ident.add_argument("--hash", required=True, help="Hash value to identify")

    gen = subparsers.add_parser("generate-cmd", help="Generate hashcat command")
    gen.add_argument("--hash-file", required=True, help="Hash file path")
    gen.add_argument("--mode", type=int, required=True, help="Hashcat hash mode")
    gen.add_argument("--attack", choices=["dictionary", "bruteforce", "hybrid_wm", "hybrid_mw"], default="dictionary")
    gen.add_argument("--wordlist", default="rockyou.txt")
    gen.add_argument("--rules", help="Rules file")
    gen.add_argument("--mask", help="Mask pattern")

    create = subparsers.add_parser("create-test-hashes", help="Create test hashes")
    create.add_argument("--output", "-o", default="test_hashes.txt")

    analyze = subparsers.add_parser("analyze-results", help="Analyze cracking results")
    analyze.add_argument("--potfile", required=True, help="Hashcat potfile")
    analyze.add_argument("--hash-file", required=True, help="Original hash file")

    args = parser.parse_args()

    if args.command == "identify":
        result = identify_hash(args.hash)
        print(json.dumps(result, indent=2))
    elif args.command == "generate-cmd":
        cmd = generate_hashcat_command(args.hash_file, args.mode, args.attack, args.wordlist, args.rules, args.mask)
        print(cmd)
    elif args.command == "create-test-hashes":
        result = create_test_hashes(args.output)
        print(json.dumps(result, indent=2))
    elif args.command == "analyze-results":
        result = analyze_cracked_results(args.potfile, args.hash_file)
        print(json.dumps(result, indent=2))
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-post-quantum-cryptography-migration/SKILL.md =====
---
name: performing-post-quantum-cryptography-migration
description: 'Assesses organizational readiness for post-quantum cryptography migration
  per NIST FIPS 203/204/205 standards. Performs cryptographic inventory scanning to
  identify quantum-vulnerable algorithms (RSA, ECDH, ECDSA), evaluates hybrid TLS
  configurations with X25519MLKEM768, and validates CRYSTALS-Kyber (ML-KEM) and CRYSTALS-Dilithium
  (ML-DSA) readiness. Implements crypto-agility assessment using oqs-provider for
  OpenSSL. Use when planning or executing the transition from classical to post-quantum
  cryptographic algorithms across enterprise infrastructure.

  '
domain: cybersecurity
subdomain: cryptography
tags:
- post-quantum
- PQC
- CRYSTALS-Kyber
- ML-KEM
- ML-DSA
- FIPS-203
- FIPS-204
- hybrid-TLS
- crypto-agility
version: '1.0'
author: mukul975
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
- T1040
---

# Performing Post-Quantum Cryptography Migration

## When to Use

- When assessing organizational readiness for the NIST post-quantum cryptography transition
- When building a cryptographic inventory to identify quantum-vulnerable algorithms across infrastructure
- When evaluating hybrid TLS 1.3 configurations using X25519MLKEM768 key exchange
- When testing CRYSTALS-Kyber (ML-KEM) and CRYSTALS-Dilithium (ML-DSA) algorithm support
- When implementing crypto-agility to support both classical and post-quantum algorithms
- When preparing migration roadmaps aligned with NIST IR 8547 deprecation timelines
- When configuring oqs-provider with OpenSSL 3.x for post-quantum algorithm support

## Prerequisites

- Python 3.8+ with `cryptography`, `requests`, `pyOpenSSL` libraries
- OpenSSL 3.0+ (3.5+ recommended for native ML-KEM/ML-DSA support)
- oqs-provider for OpenSSL (for hybrid TLS testing with older OpenSSL)
- Network access to target servers for TLS assessment
- Administrative access for infrastructure scanning
- Familiarity with PKI, TLS, and cryptographic protocols

## Core Concepts

### NIST Post-Quantum Cryptography Standards

NIST published three finalized PQC standards on August 13, 2024:

| Standard | Algorithm | Renamed To | Purpose | Based On |
|----------|-----------|------------|---------|----------|
| FIPS 203 | CRYSTALS-Kyber | ML-KEM | Key Encapsulation Mechanism | Module lattice |
| FIPS 204 | CRYSTALS-Dilithium | ML-DSA | Digital Signatures | Module lattice |
| FIPS 205 | SPHINCS+ | SLH-DSA | Digital Signatures (backup) | Stateless hash |

**ML-KEM (FIPS 203)** -- Primary standard for key exchange and encryption. Replaces
RSA and ECDH for key establishment. Three security levels: ML-KEM-512, ML-KEM-768,
ML-KEM-1024.

**ML-DSA (FIPS 204)** -- Primary standard for digital signatures. Replaces RSA and
ECDSA for signing. Three security levels: ML-DSA-44, ML-DSA-65, ML-DSA-87.

**SLH-DSA (FIPS 205)** -- Backup signature standard using hash-based approach. Intended
as fallback if lattice-based ML-DSA is found vulnerable. Larger signatures but
conservative security assumptions.

### Quantum-Vulnerable Algorithms

These classical algorithms are vulnerable to quantum attack via Shor's algorithm:

| Algorithm | Usage | Quantum Threat | Migration Priority |
|-----------|-------|---------------|-------------------|
| RSA-2048/4096 | Key exchange, signatures, encryption | Shor's algorithm breaks factoring | Critical |
| ECDH (P-256, P-384) | TLS key exchange | Shor's algorithm breaks ECDLP | Critical |
| ECDSA | Code signing, TLS certificates | Shor's algorithm breaks ECDLP | Critical |
| DSA | Legacy signatures | Shor's algorithm breaks DLP | Critical |
| DH (Diffie-Hellman) | Key exchange | Shor's algorithm breaks DLP | Critical |
| AES-128 | Symmetric encryption | Grover's halves key strength | Medium (upgrade to AES-256) |
| SHA-256 | Hashing | Grover's reduces to 128-bit | Low (still adequate) |

### NIST Migration Timeline (IR 8547)

- **2024**: Standards published, migration planning should begin
- **2030**: Deprecation of quantum-vulnerable algorithms for most federal systems
- **2035**: Complete removal of quantum-vulnerable algorithms from NIST standards
- **Now**: "Harvest now, decrypt later" attacks make early migration essential for
  long-lived secrets and data requiring long-term confidentiality

### Hybrid TLS Key Exchange

During the transition period, hybrid key exchange combines a classical algorithm with
a post-quantum algorithm. If either algorithm is secure, the connection remains protected.

```
Hybrid Key Exchange: X25519MLKEM768
  = X25519 (classical ECDH) + ML-KEM-768 (post-quantum)

Client Hello:
  supported_groups: X25519MLKEM768, X25519, secp256r1
  key_share: X25519MLKEM768

Server Hello:
  selected_group: X25519MLKEM768
  key_share: X25519MLKEM768

Shared Secret = KDF(X25519_shared || MLKEM768_shared)
```

## Instructions

### Phase 1: Cryptographic Inventory Scanning

The first step in PQC migration is discovering all cryptographic algorithm usage
across the enterprise. This includes TLS configurations, certificates, code libraries,
key stores, and protocol configurations.

```python
# Scan TLS endpoints for quantum-vulnerable algorithms
python scripts/agent.py --action scan_tls \
    --targets targets.txt \
    --output tls_inventory.json
```

The scanner identifies:
- TLS protocol versions in use
- Key exchange algorithms (RSA, ECDH, DH -- all quantum-vulnerable)
- Certificate signature algorithms (RSA, ECDSA)
- Cipher suite configurations
- Certificate key sizes and expiration dates

### Phase 2: Crypto-Agility Assessment

Evaluate the organization's ability to swap cryptographic algorithms without
major infrastructure changes:

```python
# Assess crypto-agility readiness
python scripts/agent.py --action assess_agility \
    --scan-results tls_inventory.json \
    --output agility_report.json
```

Key assessment areas:
1. **Protocol flexibility**: Can TLS configurations be updated without downtime?
2. **Library versions**: Do deployed crypto libraries support PQC algorithms?
3. **Certificate infrastructure**: Can CA issue PQC certificates?
4. **Key management**: Can KMS handle larger PQC key sizes?
5. **Hardware constraints**: Can HSMs support PQC operations?

### Phase 3: Hybrid TLS Readiness Testing

Test whether infrastructure supports hybrid key exchange with X25519MLKEM768:

```python
# Test hybrid TLS support on target servers
python scripts/agent.py --action test_hybrid_tls \
    --target server.example.com:443 \
    --output hybrid_tls_report.json
```

**OpenSSL 3.5+ (native ML-KEM support):**
```bash
# Test with native PQC support
openssl s_client -connect server.example.com:443 \
    -groups X25519MLKEM768
```

**OpenSSL 3.0-3.4 with oqs-provider:**
```bash
# Configure oqs-provider
# /etc/ssl/openssl-oqs.cnf
[openssl_init]
providers = provider_sect

[provider_sect]
default = default_sect
oqsprovider = oqsprovider_sect

[default_sect]
activate = 1

[oqsprovider_sect]
activate = 1
module = /usr/lib/oqs-provider/oqsprovider.so

# Test hybrid TLS
OPENSSL_CONF=/etc/ssl/openssl-oqs.cnf \
openssl s_client -connect server.example.com:443 \
    -groups x25519_mlkem768
```

**Web Server Configuration for Hybrid TLS:**

Apache httpd:
```apache
SSLEngine on
SSLCertificateFile /etc/ssl/certs/server.crt
SSLCertificateKeyFile /etc/ssl/private/server.key
SSLOpenSSLConfCmd Curves X25519MLKEM768:X25519:prime256v1
SSLProtocol -all +TLSv1.2 +TLSv1.3
```

NGINX:
```nginx
ssl_ecdh_curve X25519MLKEM768:X25519:prime256v1;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
```

### Phase 4: ML-KEM Key Encapsulation Validation

Validate that ML-KEM (CRYSTALS-Kyber) key encapsulation works correctly in your
environment:

```python
# Test ML-KEM key encapsulation at all security levels
python scripts/agent.py --action test_mlkem \
    --output mlkem_validation.json
```

ML-KEM parameter comparison:

| Parameter | ML-KEM-512 | ML-KEM-768 | ML-KEM-1024 |
|-----------|-----------|-----------|------------|
| Security Level | NIST Level 1 | NIST Level 3 | NIST Level 5 |
| Public Key Size | 800 bytes | 1,184 bytes | 1,568 bytes |
| Ciphertext Size | 768 bytes | 1,088 bytes | 1,568 bytes |
| Shared Secret | 32 bytes | 32 bytes | 32 bytes |
| Comparable To | AES-128 | AES-192 | AES-256 |

### Phase 5: ML-DSA Digital Signature Validation

Validate ML-DSA (CRYSTALS-Dilithium) signature operations:

```python
# Test ML-DSA digital signatures
python scripts/agent.py --action test_mldsa \
    --output mldsa_validation.json
```

ML-DSA parameter comparison:

| Parameter | ML-DSA-44 | ML-DSA-65 | ML-DSA-87 |
|-----------|----------|----------|----------|
| Security Level | NIST Level 2 | NIST Level 3 | NIST Level 5 |
| Public Key Size | 1,312 bytes | 1,952 bytes | 2,592 bytes |
| Signature Size | 2,420 bytes | 3,293 bytes | 4,595 bytes |
| Secret Key Size | 2,560 bytes | 4,032 bytes | 4,896 bytes |

### Phase 6: Migration Roadmap Generation

Generate a prioritized migration roadmap based on inventory and assessment results:

```python
# Generate complete migration roadmap
python scripts/agent.py --action roadmap \
    --scan-results tls_inventory.json \
    --agility-results agility_report.json \
    --output migration_roadmap.json
```

The roadmap prioritizes systems by:
1. **Data sensitivity**: Systems handling long-lived secrets migrate first
2. **Exposure level**: Internet-facing services before internal
3. **Crypto-agility**: Systems that can easily swap algorithms first
4. **Compliance requirements**: Federal/regulated systems per NIST IR 8547 timeline
5. **Dependency chains**: Libraries and frameworks before applications

## Examples

### Full Assessment Pipeline

```bash
# Step 1: Scan all TLS endpoints
python scripts/agent.py --action scan_tls --targets hosts.txt --output scan.json

# Step 2: Assess crypto-agility
python scripts/agent.py --action assess_agility --scan-results scan.json --output agility.json

# Step 3: Test hybrid TLS on critical servers
python scripts/agent.py --action test_hybrid_tls --target critical.example.com:443

# Step 4: Validate ML-KEM support
python scripts/agent.py --action test_mlkem --output mlkem.json

# Step 5: Validate ML-DSA support
python scripts/agent.py --action test_mldsa --output mldsa.json

# Step 6: Generate migration roadmap
python scripts/agent.py --action roadmap --scan-results scan.json --agility-results agility.json --output roadmap.json
```

### Quick Server Assessment

```bash
# Single server PQC readiness check
python scripts/agent.py --action scan_tls --target server.example.com:443
```

## Validation Checklist

- [ ] Cryptographic inventory covers all TLS endpoints, certificates, and key stores
- [ ] All quantum-vulnerable algorithms (RSA, ECDH, ECDSA, DH, DSA) are identified
- [ ] Crypto-agility assessment documents library versions and upgrade paths
- [ ] Hybrid TLS (X25519MLKEM768) tested on representative server configurations
- [ ] ML-KEM key encapsulation validated at target security level (768 recommended)
- [ ] ML-DSA signature verification validated for certificate chain use
- [ ] SLH-DSA (FIPS 205) evaluated as backup signature algorithm
- [ ] Migration roadmap prioritizes by data sensitivity and compliance timeline
- [ ] OpenSSL version and oqs-provider compatibility confirmed
- [ ] Key size increases accounted for in network and storage capacity planning
- [ ] HSM/KMS compatibility with PQC algorithms verified
- [ ] Performance impact of PQC algorithms benchmarked under production load
- [ ] "Harvest now, decrypt later" risk assessed for sensitive data channels
- [ ] Certificate Authority PQC readiness confirmed for certificate issuance

## References

- NIST PQC Standards: https://csrc.nist.gov/projects/post-quantum-cryptography
- FIPS 203 (ML-KEM): https://csrc.nist.gov/pubs/fips/203/final
- FIPS 204 (ML-DSA): https://csrc.nist.gov/pubs/fips/204/final
- FIPS 205 (SLH-DSA): https://csrc.nist.gov/pubs/fips/205/final
- NIST SP 1800-38 Migration Guide: https://www.nccoe.nist.gov/crypto-agility-considerations-migrating-post-quantum-cryptographic-algorithms
- NIST IR 8547 Transition Timeline: https://csrc.nist.gov/pubs/ir/8547/ipd
- Open Quantum Safe Project: https://openquantumsafe.org/
- oqs-provider for OpenSSL: https://github.com/open-quantum-safe/oqs-provider
- OQS TLS Integration: https://openquantumsafe.org/applications/tls.html
- CISA PQC Migration Strategy: https://www.cisa.gov/sites/default/files/2024-09/Strategy-for-Migrating-to-Automated-PQC-Discovery-and-Inventory-Tools.pdf
- IETF Hybrid Key Exchange Draft: https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/
- CycloneDX Crypto BOM: https://cyclonedx.org/use-cases/cryptographic-key/

===== END FILE =====


===== FILE: skills/performing-post-quantum-cryptography-migration/LICENSE =====

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


===== FILE: skills/performing-post-quantum-cryptography-migration/references/api-reference.md =====
# API Reference: Post-Quantum Cryptography Migration

## NIST PQC Standards Summary

### FIPS 203 -- ML-KEM (Module-Lattice-Based Key-Encapsulation Mechanism)

Formerly CRYSTALS-Kyber. Primary standard for key exchange and encryption.

**Security Levels:**

| Parameter Set | NIST Level | Public Key | Ciphertext | Shared Secret |
|---------------|-----------|------------|------------|---------------|
| ML-KEM-512 | Level 1 | 800 B | 768 B | 32 B |
| ML-KEM-768 | Level 3 | 1,184 B | 1,088 B | 32 B |
| ML-KEM-1024 | Level 5 | 1,568 B | 1,568 B | 32 B |

**Operations:**
- `KeyGen() -> (ek, dk)` -- Generate encapsulation/decapsulation key pair
- `Encaps(ek) -> (K, c)` -- Encapsulate: produce shared secret K and ciphertext c
- `Decaps(dk, c) -> K` -- Decapsulate: recover shared secret K from ciphertext

**Python (mlkem library):**
```python
from mlkem.ml_kem import ML_KEM

ml_kem = ML_KEM(768)  # ML-KEM-768
ek, dk = ml_kem.key_gen()
shared_secret, ciphertext = ml_kem.encaps(ek)
recovered_secret = ml_kem.decaps(dk, ciphertext)
assert shared_secret == recovered_secret
```

**OpenSSL 3.5+ (native):**
```bash
# Generate ML-KEM-768 key pair
openssl genpkey -algorithm mlkem768 -out mlkem768_key.pem

# Display key details
openssl pkey -in mlkem768_key.pem -text -noout

# Extract public key
openssl pkey -in mlkem768_key.pem -pubout -out mlkem768_pub.pem
```

### FIPS 204 -- ML-DSA (Module-Lattice-Based Digital Signature Algorithm)

Formerly CRYSTALS-Dilithium. Primary standard for digital signatures.

**Security Levels:**

| Parameter Set | NIST Level | Public Key | Secret Key | Signature |
|---------------|-----------|------------|------------|-----------|
| ML-DSA-44 | Level 2 | 1,312 B | 2,560 B | 2,420 B |
| ML-DSA-65 | Level 3 | 1,952 B | 4,032 B | 3,293 B |
| ML-DSA-87 | Level 5 | 2,592 B | 4,896 B | 4,595 B |

**Operations:**
- `KeyGen() -> (pk, sk)` -- Generate signing/verification key pair
- `Sign(sk, M) -> sigma` -- Sign message M with secret key
- `Verify(pk, M, sigma) -> bool` -- Verify signature on message

**OpenSSL 3.5+ (native):**
```bash
# Generate ML-DSA-65 key pair
openssl genpkey -algorithm mldsa65 -out mldsa65_key.pem

# Extract public key
openssl pkey -in mldsa65_key.pem -pubout -out mldsa65_pub.pem

# Sign a file
openssl dgst -sign mldsa65_key.pem -out signature.bin message.txt

# Verify signature
openssl dgst -verify mldsa65_pub.pem -signature signature.bin message.txt
```

### FIPS 205 -- SLH-DSA (Stateless Hash-Based Digital Signature Algorithm)

Formerly SPHINCS+. Backup signature standard using conservative hash-based approach.

**Parameter Sets (SHA2 variants):**

| Parameter Set | NIST Level | Public Key | Signature (fast) | Signature (small) |
|---------------|-----------|------------|------------------|-------------------|
| SLH-DSA-128 | Level 1 | 32 B | 17,088 B | 7,856 B |
| SLH-DSA-192 | Level 3 | 48 B | 35,664 B | 16,224 B |
| SLH-DSA-256 | Level 5 | 64 B | 49,856 B | 29,792 B |

**Variants:** Each level has fast (f) and small (s) variants with SHA2 or SHAKE hash.

## Hybrid TLS Configuration

### X25519MLKEM768 Key Exchange

The hybrid key exchange combines classical X25519 ECDH with ML-KEM-768 post-quantum
KEM. Both must be broken for the handshake to be compromised.

**Apache httpd:**
```apache
# httpd.conf or ssl.conf
SSLEngine on
SSLProtocol -all +TLSv1.2 +TLSv1.3
SSLOpenSSLConfCmd Curves X25519MLKEM768:X25519:prime256v1
SSLCertificateFile /etc/ssl/certs/server.crt
SSLCertificateKeyFile /etc/ssl/private/server.key
```

**NGINX:**
```nginx
server {
    listen 443 ssl;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ecdh_curve X25519MLKEM768:X25519:prime256v1;
    ssl_prefer_server_ciphers on;
    ssl_certificate /etc/ssl/certs/server.crt;
    ssl_certificate_key /etc/ssl/private/server.key;
}
```

**Verification:**
```bash
# Test hybrid TLS connection
openssl s_client -connect server.example.com:443 -groups X25519MLKEM768

# Verify negotiated group
# Look for "Server Temp Key: X25519MLKEM768" in output
```

## oqs-provider for OpenSSL 3.0+

### Installation

```bash
# Clone and build oqs-provider
git clone https://github.com/open-quantum-safe/oqs-provider.git
cd oqs-provider
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..
make -j$(nproc)
sudo make install
```

### Configuration

```ini
# /etc/ssl/openssl-oqs.cnf
openssl_conf = openssl_init

[openssl_init]
providers = provider_sect
ssl_conf = ssl_sect

[provider_sect]
default = default_sect
oqsprovider = oqsprovider_sect

[default_sect]
activate = 1

[oqsprovider_sect]
activate = 1
module = /usr/lib/oqs-provider/oqsprovider.so

[ssl_sect]
system_default = system_default_sect

[system_default_sect]
Groups = x25519_mlkem768:X25519:P-256:P-384
MinProtocol = TLSv1.2
```

### Usage

```bash
# Set environment variable
export OPENSSL_CONF=/etc/ssl/openssl-oqs.cnf

# List available PQC algorithms
openssl list -kem-algorithms | grep -i ml
openssl list -signature-algorithms | grep -i ml

# Generate PQC key pair
openssl genpkey -algorithm mlkem768 -out key.pem

# Test hybrid TLS
openssl s_client -connect server:443 -groups x25519_mlkem768
```

## Cryptographic Inventory Scanning

### NIST SP 1800-38 Discovery Architecture

```
+------------------+     +------------------+     +------------------+
| Source Code Scan | --> |                  | --> | Risk Assessment  |
+------------------+    | Central Analysis |     +------------------+
+------------------+    |     Engine       |            |
| Binary Analysis  | -->|  (Normalization  |     +------------------+
+------------------+    |  & Correlation)  |     | Migration        |
+------------------+    |                  |     | Prioritization   |
| Network Traffic  | -->|                  |     +------------------+
+------------------+    +------------------+
+------------------+
| Certificate Scan | -->
+------------------+
```

### Discovery Domains

| Domain | What to Scan | Tools |
|--------|-------------|-------|
| CI/CD Pipeline | Source code, build configs, dependencies | SCA tools, Semgrep |
| Operational Systems | Running services, installed libraries, key stores | NIST SP 1800-38B tools |
| Network Services | TLS endpoints, VPN configs, IPsec tunnels | This agent, sslyze, testssl |
| Certificates | CA chains, code signing certs, TLS certificates | cert-manager, openssl |

## Quantum-Vulnerable Algorithm Reference

| Algorithm | NIST Status (IR 8547) | Quantum Threat | Replacement |
|-----------|-----------------------|----------------|-------------|
| RSA (all sizes) | Deprecated 2030, removed 2035 | Shor's algorithm | ML-KEM (encryption), ML-DSA (signing) |
| ECDH / ECDHE | Deprecated 2030, removed 2035 | Shor's algorithm | ML-KEM / X25519MLKEM768 hybrid |
| ECDSA | Deprecated 2030, removed 2035 | Shor's algorithm | ML-DSA |
| DSA | Already deprecated | Shor's algorithm | ML-DSA |
| DH / DHE | Deprecated 2030, removed 2035 | Shor's algorithm | ML-KEM |
| AES-128 | Acceptable with caveat | Grover's halves to 64-bit | AES-256 |
| AES-256 | Quantum-safe | Grover's reduces to 128-bit | No change needed |
| SHA-256 | Quantum-safe | Grover's reduces to 128-bit | No change needed |
| SHA-3 | Quantum-safe | Grover's reduces to 128-bit | No change needed |

## MITRE ATT&CK Relevance

| Technique | ID | PQC Relevance |
|-----------|----|---------------|
| Adversary-in-the-Middle | T1557 | Quantum computers can break key exchange in recorded sessions |
| Encrypted Channel | T1573 | Harvest-now-decrypt-later targets encrypted C2 traffic |
| Steal Application Access Token | T1528 | Quantum computers can forge digital signatures |
| Forge Web Credentials | T1606 | Quantum computers can break certificate private keys |

## References

- NIST PQC Project: https://csrc.nist.gov/projects/post-quantum-cryptography
- FIPS 203 Final: https://csrc.nist.gov/pubs/fips/203/final
- FIPS 204 Final: https://csrc.nist.gov/pubs/fips/204/final
- FIPS 205 Final: https://csrc.nist.gov/pubs/fips/205/final
- NIST IR 8547 (Transition Timeline): https://csrc.nist.gov/pubs/ir/8547/ipd
- NIST SP 1800-38 (Migration Guide): https://www.nccoe.nist.gov/crypto-agility-considerations-migrating-post-quantum-cryptographic-algorithms
- CISA PQC Strategy: https://www.cisa.gov/sites/default/files/2024-09/Strategy-for-Migrating-to-Automated-PQC-Discovery-and-Inventory-Tools.pdf
- Open Quantum Safe: https://openquantumsafe.org/
- oqs-provider GitHub: https://github.com/open-quantum-safe/oqs-provider
- OQS TLS Applications: https://openquantumsafe.org/applications/tls.html
- IETF Hybrid Design Draft: https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/
- kyber-py (Python ML-KEM): https://github.com/GiacomoPope/kyber-py
- ml-kem (Python FIPS 203): https://github.com/AntonKueltz/ml-kem
- CycloneDX Crypto BOM: https://cyclonedx.org/use-cases/cryptographic-key/

===== END FILE =====


===== FILE: skills/performing-post-quantum-cryptography-migration/scripts/agent.py =====
#!/usr/bin/env python3
"""
Agent for performing post-quantum cryptography migration assessment.

Scans TLS endpoints for quantum-vulnerable algorithms, assesses crypto-agility
readiness, tests hybrid TLS (X25519MLKEM768) support, validates ML-KEM and
ML-DSA algorithm functionality, and generates prioritized migration roadmaps
per NIST FIPS 203/204/205 standards.
"""

import os
import sys
import json
import ssl
import socket
import struct
import argparse
import logging
import subprocess
import hashlib
import re
from datetime import datetime, timezone
from pathlib import Path
from collections import defaultdict

import requests

# ---------------------------------------------------------------------------
# Logging
# ---------------------------------------------------------------------------
LOG_FORMAT = "%(asctime)s [%(levelname)s] %(message)s"
logging.basicConfig(level=logging.INFO, format=LOG_FORMAT)
logger = logging.getLogger("pqc-migration-agent")

# ---------------------------------------------------------------------------
# Constants: Quantum-vulnerable algorithm classification
# ---------------------------------------------------------------------------

QUANTUM_VULNERABLE_KEY_EXCHANGE = {
    "RSA",
    "DH",
    "DHE",
    "ECDH",
    "ECDHE",
}

QUANTUM_VULNERABLE_SIGNATURE = {
    "RSA",
    "ECDSA",
    "DSA",
    "Ed25519",
    "Ed448",
}

QUANTUM_SAFE_KEY_EXCHANGE = {
    "X25519MLKEM768",
    "X25519_MLKEM768",
    "SecP256r1MLKEM768",
    "MLKEM512",
    "MLKEM768",
    "MLKEM1024",
    "X448MLKEM1024",
}

PQC_SIGNATURE_ALGORITHMS = {
    "MLDSA44",
    "MLDSA65",
    "MLDSA87",
    "SLHDSA_SHA2_128S",
    "SLHDSA_SHA2_128F",
    "SLHDSA_SHA2_192S",
    "SLHDSA_SHA2_192F",
    "SLHDSA_SHA2_256S",
    "SLHDSA_SHA2_256F",
    "SLHDSA_SHAKE_128S",
    "SLHDSA_SHAKE_128F",
    "SLHDSA_SHAKE_192S",
    "SLHDSA_SHAKE_192F",
    "SLHDSA_SHAKE_256S",
    "SLHDSA_SHAKE_256F",
}

# Minimum key sizes that provide adequate classical security
MIN_SECURE_KEY_SIZES = {
    "RSA": 2048,
    "EC": 256,
    "AES": 256,  # For post-quantum, AES-256 recommended (Grover's halves effective strength)
}

NIST_MIGRATION_DEADLINES = {
    "deprecation": "2030",
    "disallowed": "2035",
    "description": "NIST IR 8547: quantum-vulnerable algorithms deprecated by 2030, "
                   "disallowed by 2035 for federal systems",
}

# ML-KEM parameters per FIPS 203
MLKEM_PARAMS = {
    "ML-KEM-512": {
        "security_level": 1,
        "pk_bytes": 800,
        "sk_bytes": 1632,
        "ct_bytes": 768,
        "ss_bytes": 32,
        "comparable_to": "AES-128",
    },
    "ML-KEM-768": {
        "security_level": 3,
        "pk_bytes": 1184,
        "sk_bytes": 2400,
        "ct_bytes": 1088,
        "ss_bytes": 32,
        "comparable_to": "AES-192",
    },
    "ML-KEM-1024": {
        "security_level": 5,
        "pk_bytes": 1568,
        "sk_bytes": 3168,
        "ct_bytes": 1568,
        "ss_bytes": 32,
        "comparable_to": "AES-256",
    },
}

# ML-DSA parameters per FIPS 204
MLDSA_PARAMS = {
    "ML-DSA-44": {
        "security_level": 2,
        "pk_bytes": 1312,
        "sk_bytes": 2560,
        "sig_bytes": 2420,
        "comparable_to": "NIST Level 2 (~AES-128+)",
    },
    "ML-DSA-65": {
        "security_level": 3,
        "pk_bytes": 1952,
        "sk_bytes": 4032,
        "sig_bytes": 3293,
        "comparable_to": "NIST Level 3 (~AES-192)",
    },
    "ML-DSA-87": {
        "security_level": 5,
        "pk_bytes": 2592,
        "sk_bytes": 4896,
        "sig_bytes": 4595,
        "comparable_to": "NIST Level 5 (~AES-256)",
    },
}

# SLH-DSA parameters per FIPS 205
SLHDSA_PARAMS = {
    "SLH-DSA-SHA2-128s": {"security_level": 1, "pk_bytes": 32, "sig_bytes": 7856},
    "SLH-DSA-SHA2-128f": {"security_level": 1, "pk_bytes": 32, "sig_bytes": 17088},
    "SLH-DSA-SHA2-192s": {"security_level": 3, "pk_bytes": 48, "sig_bytes": 16224},
    "SLH-DSA-SHA2-192f": {"security_level": 3, "pk_bytes": 48, "sig_bytes": 35664},
    "SLH-DSA-SHA2-256s": {"security_level": 5, "pk_bytes": 64, "sig_bytes": 29792},
    "SLH-DSA-SHA2-256f": {"security_level": 5, "pk_bytes": 64, "sig_bytes": 49856},
    "SLH-DSA-SHAKE-128s": {"security_level": 1, "pk_bytes": 32, "sig_bytes": 7856},
    "SLH-DSA-SHAKE-128f": {"security_level": 1, "pk_bytes": 32, "sig_bytes": 17088},
    "SLH-DSA-SHAKE-192s": {"security_level": 3, "pk_bytes": 48, "sig_bytes": 16224},
    "SLH-DSA-SHAKE-192f": {"security_level": 3, "pk_bytes": 48, "sig_bytes": 35664},
    "SLH-DSA-SHAKE-256s": {"security_level": 5, "pk_bytes": 64, "sig_bytes": 29792},
    "SLH-DSA-SHAKE-256f": {"security_level": 5, "pk_bytes": 64, "sig_bytes": 49856},
}


# ---------------------------------------------------------------------------
# TLS Endpoint Scanning
# ---------------------------------------------------------------------------

def scan_tls_endpoint(host, port=443, timeout=10):
    """
    Scan a TLS endpoint to extract cryptographic algorithm details.

    Connects to the target, performs TLS handshake, and extracts:
    - Protocol version
    - Cipher suite (key exchange, encryption, MAC)
    - Certificate details (algorithm, key size, validity)
    - Supported groups / curves

    Args:
        host: Target hostname or IP
        port: Target port (default 443)
        timeout: Connection timeout in seconds

    Returns:
        Dict with comprehensive TLS cryptographic inventory
    """
    result = {
        "host": host,
        "port": port,
        "scan_time": datetime.now(timezone.utc).isoformat(),
        "tls_version": None,
        "cipher_suite": None,
        "key_exchange": None,
        "certificate": {},
        "quantum_vulnerable": True,
        "vulnerabilities": [],
        "recommendations": [],
    }

    try:
        context = ssl.create_default_context()
        context.check_hostname = True
        context.verify_mode = ssl.CERT_REQUIRED

        with socket.create_connection((host, port), timeout=timeout) as sock:
            with context.wrap_socket(sock, server_hostname=host) as tls_sock:
                # Extract TLS session details
                cipher = tls_sock.cipher()
                result["tls_version"] = tls_sock.version()
                result["cipher_suite"] = cipher[0] if cipher else None
                result["cipher_protocol"] = cipher[1] if cipher and len(cipher) > 1 else None
                result["cipher_bits"] = cipher[2] if cipher and len(cipher) > 2 else None

                # Parse key exchange from cipher suite name
                cipher_name = cipher[0] if cipher else ""
                result["key_exchange"] = _extract_key_exchange(cipher_name)

                # Extract certificate details
                cert = tls_sock.getpeercert()
                cert_der = tls_sock.getpeercert(binary_form=True)
                result["certificate"] = _parse_certificate(cert, cert_der)

                # Assess quantum vulnerability
                _assess_quantum_vulnerability(result)

        logger.info("Scanned %s:%d -- %s [%s]", host, port,
                     result["cipher_suite"], result["tls_version"])

    except ssl.SSLError as e:
        result["error"] = f"SSL error: {e}"
        logger.warning("SSL error scanning %s:%d: %s", host, port, e)
    except socket.timeout:
        result["error"] = "Connection timed out"
        logger.warning("Timeout scanning %s:%d", host, port)
    except socket.gaierror as e:
        result["error"] = f"DNS resolution failed: {e}"
        logger.warning("DNS error for %s: %s", host, e)
    except ConnectionRefusedError:
        result["error"] = "Connection refused"
        logger.warning("Connection refused: %s:%d", host, port)
    except Exception as e:
        result["error"] = f"Unexpected error: {e}"
        logger.error("Error scanning %s:%d: %s", host, port, e)

    return result


def _extract_key_exchange(cipher_name):
    """Extract key exchange algorithm from cipher suite name."""
    cipher_upper = cipher_name.upper()
    if "ECDHE" in cipher_upper:
        return "ECDHE"
    elif "DHE" in cipher_upper or "EDH" in cipher_upper:
        return "DHE"
    elif "ECDH" in cipher_upper:
        return "ECDH"
    elif "DH" in cipher_upper:
        return "DH"
    elif "RSA" in cipher_upper:
        return "RSA"
    return "Unknown"


def _parse_certificate(cert, cert_der=None):
    """Parse certificate details from Python ssl cert dict."""
    cert_info = {
        "subject": "",
        "issuer": "",
        "not_before": "",
        "not_after": "",
        "serial_number": "",
        "signature_algorithm": "Unknown",
        "public_key_algorithm": "Unknown",
        "public_key_bits": 0,
        "san": [],
    }

    if not cert:
        return cert_info

    # Extract subject
    subject = cert.get("subject", ())
    for rdn in subject:
        for attr_type, attr_value in rdn:
            if attr_type == "commonName":
                cert_info["subject"] = attr_value

    # Extract issuer
    issuer = cert.get("issuer", ())
    for rdn in issuer:
        for attr_type, attr_value in rdn:
            if attr_type == "organizationName":
                cert_info["issuer"] = attr_value

    cert_info["not_before"] = cert.get("notBefore", "")
    cert_info["not_after"] = cert.get("notAfter", "")
    cert_info["serial_number"] = cert.get("serialNumber", "")

    # Extract SANs
    sans = cert.get("subjectAltName", ())
    cert_info["san"] = [value for san_type, value in sans if san_type == "DNS"]

    # Try to get detailed cert info via openssl
    if cert_der:
        try:
            cert_details = _openssl_parse_cert_der(cert_der)
            cert_info.update(cert_details)
        except Exception:
            pass

    return cert_info


def _openssl_parse_cert_der(cert_der):
    """Use openssl CLI to parse DER certificate for algorithm details."""
    details = {}
    try:
        proc = subprocess.run(
            ["openssl", "x509", "-inform", "DER", "-noout", "-text"],
            input=cert_der,
            capture_output=True,
            timeout=10,
        )
        output = proc.stdout.decode("utf-8", errors="replace")

        # Extract signature algorithm
        sig_match = re.search(r"Signature Algorithm:\s+(.+)", output)
        if sig_match:
            details["signature_algorithm"] = sig_match.group(1).strip()

        # Extract public key algorithm and size
        pk_match = re.search(r"Public Key Algorithm:\s+(.+)", output)
        if pk_match:
            details["public_key_algorithm"] = pk_match.group(1).strip()

        bits_match = re.search(r"(?:RSA Public-Key|Public-Key):\s+\((\d+) bit\)", output)
        if bits_match:
            details["public_key_bits"] = int(bits_match.group(1))

        ec_match = re.search(r"ASN1 OID:\s+(.+)", output)
        if ec_match:
            details["ec_curve"] = ec_match.group(1).strip()

    except (subprocess.TimeoutExpired, FileNotFoundError):
        pass

    return details


def _assess_quantum_vulnerability(result):
    """Assess whether the TLS connection uses quantum-vulnerable cryptography."""
    vulnerabilities = []
    recommendations = []
    is_vulnerable = False

    # Check key exchange
    kx = result.get("key_exchange", "").upper()
    kx_base = kx.replace("_", "")
    if any(v in kx_base for v in ["RSA", "ECDH", "ECDHE", "DHE", "DH"]):
        if not any(pq in kx_base for pq in ["MLKEM", "KYBER"]):
            is_vulnerable = True
            vulnerabilities.append({
                "component": "key_exchange",
                "algorithm": kx,
                "threat": "Shor's algorithm can break this key exchange",
                "severity": "critical",
            })
            recommendations.append(
                "Migrate to hybrid key exchange X25519MLKEM768 for TLS 1.3"
            )

    # Check certificate signature algorithm
    sig_algo = result.get("certificate", {}).get("signature_algorithm", "").lower()
    if any(v in sig_algo for v in ["rsa", "ecdsa", "dsa"]):
        if not any(pq in sig_algo for pq in ["mldsa", "dilithium", "slhdsa", "sphincs"]):
            is_vulnerable = True
            vulnerabilities.append({
                "component": "certificate_signature",
                "algorithm": sig_algo,
                "threat": "Shor's algorithm can forge signatures",
                "severity": "high",
            })
            recommendations.append(
                "Plan migration to ML-DSA (FIPS 204) for certificate signatures"
            )

    # Check public key algorithm
    pk_algo = result.get("certificate", {}).get("public_key_algorithm", "").lower()
    pk_bits = result.get("certificate", {}).get("public_key_bits", 0)

    if "rsa" in pk_algo:
        is_vulnerable = True
        if pk_bits < 2048:
            vulnerabilities.append({
                "component": "certificate_public_key",
                "algorithm": f"RSA-{pk_bits}",
                "threat": "Below minimum key size AND quantum-vulnerable",
                "severity": "critical",
            })
        else:
            vulnerabilities.append({
                "component": "certificate_public_key",
                "algorithm": f"RSA-{pk_bits}",
                "threat": "Quantum-vulnerable (adequate classically)",
                "severity": "high",
            })

    if "ec" in pk_algo or "ecdsa" in pk_algo:
        is_vulnerable = True
        vulnerabilities.append({
            "component": "certificate_public_key",
            "algorithm": pk_algo,
            "threat": "Shor's algorithm breaks elliptic curve discrete log",
            "severity": "high",
        })

    # Check TLS version
    tls_version = result.get("tls_version", "")
    if tls_version and "1.3" not in tls_version:
        recommendations.append(
            f"Upgrade from {tls_version} to TLS 1.3 (required for hybrid PQC key exchange)"
        )

    result["quantum_vulnerable"] = is_vulnerable
    result["vulnerabilities"] = vulnerabilities
    result["recommendations"] = recommendations


def scan_multiple_endpoints(targets_file, port=443):
    """
    Scan multiple TLS endpoints from a targets file.

    Args:
        targets_file: Path to file with one host[:port] per line
        port: Default port if not specified per target

    Returns:
        List of scan results for all targets
    """
    targets_path = Path(targets_file)
    if not targets_path.exists():
        logger.error("Targets file not found: %s", targets_file)
        return []

    results = []
    with open(targets_path, encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if not line or line.startswith("#"):
                continue

            if ":" in line:
                host, target_port = line.rsplit(":", 1)
                try:
                    target_port = int(target_port)
                except ValueError:
                    target_port = port
            else:
                host = line
                target_port = port

            result = scan_tls_endpoint(host, target_port)
            results.append(result)

    return results


# ---------------------------------------------------------------------------
# Crypto-Agility Assessment
# ---------------------------------------------------------------------------

def assess_crypto_agility(scan_results):
    """
    Assess organizational crypto-agility based on TLS scan results.

    Evaluates the ability to migrate from quantum-vulnerable algorithms
    to post-quantum alternatives without major infrastructure changes.

    Args:
        scan_results: List of TLS scan result dicts

    Returns:
        Crypto-agility assessment report
    """
    assessment = {
        "assessment_time": datetime.now(timezone.utc).isoformat(),
        "total_endpoints": len(scan_results),
        "quantum_vulnerable_endpoints": 0,
        "tls13_ready": 0,
        "algorithm_inventory": defaultdict(int),
        "certificate_algorithms": defaultdict(int),
        "key_exchange_algorithms": defaultdict(int),
        "risk_summary": {},
        "agility_score": 0,
        "findings": [],
        "recommendations": [],
    }

    for result in scan_results:
        if result.get("error"):
            continue

        if result.get("quantum_vulnerable"):
            assessment["quantum_vulnerable_endpoints"] += 1

        tls_ver = result.get("tls_version", "")
        if "1.3" in tls_ver:
            assessment["tls13_ready"] += 1

        cipher = result.get("cipher_suite", "Unknown")
        assessment["algorithm_inventory"][cipher] += 1

        kx = result.get("key_exchange", "Unknown")
        assessment["key_exchange_algorithms"][kx] += 1

        sig_algo = result.get("certificate", {}).get("signature_algorithm", "Unknown")
        assessment["certificate_algorithms"][sig_algo] += 1

    total = assessment["total_endpoints"]
    if total == 0:
        return assessment

    vuln_pct = (assessment["quantum_vulnerable_endpoints"] / total) * 100
    tls13_pct = (assessment["tls13_ready"] / total) * 100

    # Calculate agility score (0-100)
    score = 0
    score += min(40, tls13_pct * 0.4)  # TLS 1.3 readiness (up to 40 points)
    score += max(0, 30 - (vuln_pct * 0.3))  # Fewer vulnerabilities (up to 30 points)

    # Bonus for algorithm diversity (indicates flexibility)
    unique_ciphers = len(assessment["algorithm_inventory"])
    score += min(15, unique_ciphers * 3)  # Up to 15 points for diversity

    # Bonus for modern configurations
    modern_kx = sum(
        v for k, v in assessment["key_exchange_algorithms"].items()
        if k in ("ECDHE", "DHE")
    )
    if total > 0:
        score += min(15, (modern_kx / total) * 15)  # Up to 15 points for PFS

    assessment["agility_score"] = round(score, 1)

    # Risk summary
    assessment["risk_summary"] = {
        "quantum_vulnerable_percentage": round(vuln_pct, 1),
        "tls13_percentage": round(tls13_pct, 1),
        "unique_cipher_suites": unique_ciphers,
        "risk_level": (
            "critical" if vuln_pct > 90 else
            "high" if vuln_pct > 70 else
            "medium" if vuln_pct > 40 else
            "low"
        ),
    }

    # Findings
    if vuln_pct > 0:
        assessment["findings"].append({
            "finding": f"{assessment['quantum_vulnerable_endpoints']}/{total} "
                       f"endpoints ({vuln_pct:.0f}%) use quantum-vulnerable algorithms",
            "severity": "critical" if vuln_pct > 70 else "high",
            "category": "quantum_vulnerability",
        })

    if tls13_pct < 100:
        not_tls13 = total - assessment["tls13_ready"]
        assessment["findings"].append({
            "finding": f"{not_tls13} endpoints do not support TLS 1.3 "
                       f"(required for hybrid PQC key exchange)",
            "severity": "high",
            "category": "protocol_version",
        })

    # Recommendations
    if tls13_pct < 100:
        assessment["recommendations"].append({
            "priority": 1,
            "action": "Upgrade all TLS endpoints to TLS 1.3",
            "rationale": "Hybrid PQC key exchange (X25519MLKEM768) requires TLS 1.3",
            "effort": "medium",
        })

    assessment["recommendations"].append({
        "priority": 2,
        "action": "Deploy hybrid key exchange X25519MLKEM768 on TLS 1.3 endpoints",
        "rationale": "Provides quantum-resistant key exchange while maintaining "
                     "classical security as fallback",
        "effort": "low" if tls13_pct > 80 else "medium",
    })

    assessment["recommendations"].append({
        "priority": 3,
        "action": "Update OpenSSL to 3.5+ or install oqs-provider for PQC support",
        "rationale": "OpenSSL 3.5 provides native ML-KEM/ML-DSA support; "
                     "oqs-provider adds PQC to OpenSSL 3.0-3.4",
        "effort": "medium",
    })

    assessment["recommendations"].append({
        "priority": 4,
        "action": "Plan certificate migration to ML-DSA (FIPS 204) signatures",
        "rationale": "Certificate signatures need PQC before the NIST 2030 "
                     "deprecation deadline",
        "effort": "high",
    })

    # Convert defaultdicts to regular dicts for JSON serialization
    assessment["algorithm_inventory"] = dict(assessment["algorithm_inventory"])
    assessment["certificate_algorithms"] = dict(assessment["certificate_algorithms"])
    assessment["key_exchange_algorithms"] = dict(assessment["key_exchange_algorithms"])

    return assessment


# ---------------------------------------------------------------------------
# Hybrid TLS Testing
# ---------------------------------------------------------------------------

def test_hybrid_tls_support(host, port=443):
    """
    Test whether a server supports hybrid post-quantum TLS key exchange.

    Attempts connection using X25519MLKEM768 and other hybrid groups.
    Requires OpenSSL 3.5+ or oqs-provider.

    Args:
        host: Target hostname
        port: Target port

    Returns:
        Dict with hybrid TLS support test results
    """
    result = {
        "host": host,
        "port": port,
        "test_time": datetime.now(timezone.utc).isoformat(),
        "openssl_version": "",
        "hybrid_groups_tested": [],
        "pqc_supported": False,
        "details": {},
    }

    # Check OpenSSL version
    try:
        proc = subprocess.run(
            ["openssl", "version"],
            capture_output=True, timeout=5,
        )
        result["openssl_version"] = proc.stdout.decode().strip()
    except (FileNotFoundError, subprocess.TimeoutExpired):
        result["openssl_version"] = "openssl not found"

    # Test hybrid key exchange groups
    hybrid_groups = [
        "X25519MLKEM768",
        "x25519_mlkem768",  # oqs-provider naming
    ]

    for group in hybrid_groups:
        test_result = _test_tls_group(host, port, group)
        result["hybrid_groups_tested"].append(test_result)
        if test_result.get("supported"):
            result["pqc_supported"] = True

    # Also test classical groups for comparison
    classical_groups = ["X25519", "P-256", "P-384"]
    for group in classical_groups:
        test_result = _test_tls_group(host, port, group)
        result["hybrid_groups_tested"].append(test_result)

    return result


def _test_tls_group(host, port, group):
    """Test a specific TLS key exchange group against a server."""
    test = {
        "group": group,
        "supported": False,
        "error": None,
    }

    try:
        cmd = [
            "openssl", "s_client",
            "-connect", f"{host}:{port}",
            "-groups", group,
            "-brief",
        ]
        proc = subprocess.run(
            cmd,
            input=b"",
            capture_output=True,
            timeout=15,
        )
        output = proc.stdout.decode("utf-8", errors="replace")
        stderr = proc.stderr.decode("utf-8", errors="replace")

        # Check if connection succeeded with the specified group
        if "Protocol" in output or "Verification" in output:
            test["supported"] = True
            # Extract negotiated protocol and cipher
            proto_match = re.search(r"Protocol version:\s+(\S+)", output)
            cipher_match = re.search(r"Ciphersuite:\s+(\S+)", output)
            if proto_match:
                test["protocol"] = proto_match.group(1)
            if cipher_match:
                test["cipher"] = cipher_match.group(1)
        elif "no protocols available" in stderr.lower():
            test["error"] = "Group not supported by server"
        elif "unknown group" in stderr.lower():
            test["error"] = "Group not supported by local OpenSSL"
        else:
            test["error"] = "Connection failed"
            if stderr:
                # Take first line of error
                test["error_detail"] = stderr.split("\n")[0][:200]

    except subprocess.TimeoutExpired:
        test["error"] = "Connection timed out"
    except FileNotFoundError:
        test["error"] = "openssl binary not found"

    return test


# ---------------------------------------------------------------------------
# ML-KEM (FIPS 203) Validation
# ---------------------------------------------------------------------------

def test_mlkem_support():
    """
    Test ML-KEM (CRYSTALS-Kyber / FIPS 203) key encapsulation support.

    Tests keygen, encapsulation, and decapsulation at all three security levels
    using either the mlkem Python library or OpenSSL with oqs-provider.

    Returns:
        Dict with ML-KEM validation results for each security level
    """
    results = {
        "test_time": datetime.now(timezone.utc).isoformat(),
        "library": None,
        "levels": {},
    }

    # Try Python mlkem library first
    try:
        from mlkem.ml_kem import ML_KEM

        results["library"] = "mlkem (Python)"

        for level_name, params in MLKEM_PARAMS.items():
            level_result = _test_mlkem_python(level_name, params)
            results["levels"][level_name] = level_result

        logger.info("ML-KEM tested via Python mlkem library")
        return results

    except ImportError:
        logger.info("mlkem Python library not available, trying OpenSSL")

    # Fallback to OpenSSL CLI
    try:
        for level_name, params in MLKEM_PARAMS.items():
            level_result = _test_mlkem_openssl(level_name, params)
            results["levels"][level_name] = level_result

        results["library"] = "OpenSSL"
        logger.info("ML-KEM tested via OpenSSL")
        return results

    except Exception as e:
        logger.warning("ML-KEM testing failed: %s", e)
        results["error"] = str(e)

    return results


def _test_mlkem_python(level_name, params):
    """Test ML-KEM at a specific level using the Python mlkem library."""
    result = {
        "level": level_name,
        "security_level": params["security_level"],
        "supported": False,
        "keygen": False,
        "encaps": False,
        "decaps": False,
        "shared_secret_match": False,
        "performance": {},
    }

    try:
        from mlkem.ml_kem import ML_KEM
        import time

        # Map level name to ML_KEM parameter
        param_map = {
            "ML-KEM-512": 512,
            "ML-KEM-768": 768,
            "ML-KEM-1024": 1024,
        }
        k = param_map.get(level_name)
        if k is None:
            result["error"] = f"Unknown level: {level_name}"
            return result

        ml_kem = ML_KEM(k)

        # Key generation
        t0 = time.perf_counter()
        ek, dk = ml_kem.key_gen()
        keygen_ms = (time.perf_counter() - t0) * 1000
        result["keygen"] = True
        result["performance"]["keygen_ms"] = round(keygen_ms, 2)

        # Verify key sizes
        result["pk_bytes"] = len(ek)
        result["sk_bytes"] = len(dk)

        # Encapsulation
        t0 = time.perf_counter()
        shared_secret, ciphertext = ml_kem.encaps(ek)
        encaps_ms = (time.perf_counter() - t0) * 1000
        result["encaps"] = True
        result["performance"]["encaps_ms"] = round(encaps_ms, 2)
        result["ct_bytes"] = len(ciphertext)

        # Decapsulation
        t0 = time.perf_counter()
        shared_secret_dec = ml_kem.decaps(dk, ciphertext)
        decaps_ms = (time.perf_counter() - t0) * 1000
        result["decaps"] = True
        result["performance"]["decaps_ms"] = round(decaps_ms, 2)

        # Verify shared secrets match
        result["shared_secret_match"] = (shared_secret == shared_secret_dec)
        result["ss_bytes"] = len(shared_secret)
        result["supported"] = result["shared_secret_match"]

    except Exception as e:
        result["error"] = str(e)

    return result


def _test_mlkem_openssl(level_name, params):
    """Test ML-KEM support via OpenSSL CLI (requires OpenSSL 3.5+ or oqs-provider)."""
    result = {
        "level": level_name,
        "security_level": params["security_level"],
        "supported": False,
        "error": None,
    }

    algo_map = {
        "ML-KEM-512": "mlkem512",
        "ML-KEM-768": "mlkem768",
        "ML-KEM-1024": "mlkem1024",
    }
    algo = algo_map.get(level_name, "mlkem768")

    try:
        # Test key generation with openssl
        proc = subprocess.run(
            ["openssl", "pkey", "-algorithm", algo, "-text", "-noout"],
            input=b"",
            capture_output=True,
            timeout=15,
        )

        # Also try via genpkey
        proc2 = subprocess.run(
            ["openssl", "genpkey", "-algorithm", algo, "-outform", "PEM"],
            capture_output=True,
            timeout=15,
        )

        if proc2.returncode == 0:
            result["supported"] = True
            result["keygen"] = True
            logger.info("OpenSSL supports %s (%s)", level_name, algo)
        else:
            stderr = proc2.stderr.decode("utf-8", errors="replace")
            result["error"] = stderr.split("\n")[0][:200] if stderr else "keygen failed"

    except subprocess.TimeoutExpired:
        result["error"] = "OpenSSL command timed out"
    except FileNotFoundError:
        result["error"] = "openssl binary not found"

    return result


# ---------------------------------------------------------------------------
# ML-DSA (FIPS 204) Validation
# ---------------------------------------------------------------------------

def test_mldsa_support():
    """
    Test ML-DSA (CRYSTALS-Dilithium / FIPS 204) digital signature support.

    Tests key generation, signing, and verification at all three security levels
    using OpenSSL with native support or oqs-provider.

    Returns:
        Dict with ML-DSA validation results
    """
    results = {
        "test_time": datetime.now(timezone.utc).isoformat(),
        "library": None,
        "levels": {},
    }

    algo_map = {
        "ML-DSA-44": "mldsa44",
        "ML-DSA-65": "mldsa65",
        "ML-DSA-87": "mldsa87",
    }

    for level_name, params in MLDSA_PARAMS.items():
        algo = algo_map.get(level_name, "mldsa65")
        level_result = _test_mldsa_openssl(level_name, algo, params)
        results["levels"][level_name] = level_result

    results["library"] = "OpenSSL"
    return results


def _test_mldsa_openssl(level_name, algo, params):
    """Test ML-DSA at a specific level via OpenSSL CLI."""
    result = {
        "level": level_name,
        "security_level": params["security_level"],
        "supported": False,
        "keygen": False,
        "sign": False,
        "verify": False,
        "error": None,
    }

    import tempfile

    try:
        with tempfile.TemporaryDirectory() as tmpdir:
            key_path = os.path.join(tmpdir, "key.pem")
            pub_path = os.path.join(tmpdir, "pub.pem")
            msg_path = os.path.join(tmpdir, "message.txt")
            sig_path = os.path.join(tmpdir, "signature.bin")

            # Generate key pair
            proc = subprocess.run(
                ["openssl", "genpkey", "-algorithm", algo, "-out", key_path],
                capture_output=True, timeout=30,
            )
            if proc.returncode != 0:
                stderr = proc.stderr.decode("utf-8", errors="replace")
                result["error"] = f"keygen failed: {stderr.split(chr(10))[0][:200]}"
                return result
            result["keygen"] = True

            # Extract public key
            proc = subprocess.run(
                ["openssl", "pkey", "-in", key_path, "-pubout", "-out", pub_path],
                capture_output=True, timeout=15,
            )
            if proc.returncode != 0:
                result["error"] = "public key extraction failed"
                return result

            # Create test message
            with open(msg_path, "w") as f:
                f.write("Post-quantum cryptography migration test message")

            # Sign
            proc = subprocess.run(
                ["openssl", "pkeyutl", "-sign",
                 "-inkey", key_path,
                 "-in", msg_path,
                 "-out", sig_path],
                capture_output=True, timeout=30,
            )
            if proc.returncode != 0:
                # Try dgst approach
                proc = subprocess.run(
                    ["openssl", "dgst", "-sign", key_path,
                     "-out", sig_path, msg_path],
                    capture_output=True, timeout=30,
                )
            if proc.returncode == 0:
                result["sign"] = True
            else:
                result["error"] = "signing failed"
                return result

            # Verify
            proc = subprocess.run(
                ["openssl", "pkeyutl", "-verify",
                 "-pubin", "-inkey", pub_path,
                 "-in", msg_path,
                 "-sigfile", sig_path],
                capture_output=True, timeout=30,
            )
            if proc.returncode != 0:
                proc = subprocess.run(
                    ["openssl", "dgst", "-verify", pub_path,
                     "-signature", sig_path, msg_path],
                    capture_output=True, timeout=30,
                )
            if proc.returncode == 0:
                result["verify"] = True
                result["supported"] = True
            else:
                result["error"] = "verification failed"

    except subprocess.TimeoutExpired:
        result["error"] = "OpenSSL command timed out"
    except FileNotFoundError:
        result["error"] = "openssl binary not found"

    return result


# ---------------------------------------------------------------------------
# Migration Roadmap Generation
# ---------------------------------------------------------------------------

def generate_migration_roadmap(scan_results, agility_assessment=None):
    """
    Generate a prioritized PQC migration roadmap.

    Prioritizes systems based on data sensitivity, exposure, crypto-agility,
    compliance requirements, and dependency chains.

    Args:
        scan_results: List of TLS scan results
        agility_assessment: Optional crypto-agility assessment

    Returns:
        Migration roadmap with phased recommendations
    """
    roadmap = {
        "generated_at": datetime.now(timezone.utc).isoformat(),
        "nist_timeline": NIST_MIGRATION_DEADLINES,
        "executive_summary": "",
        "phases": [],
        "risk_register": [],
        "quick_wins": [],
    }

    total = len(scan_results)
    vuln_count = sum(1 for r in scan_results if r.get("quantum_vulnerable"))
    tls13_count = sum(1 for r in scan_results
                      if "1.3" in r.get("tls_version", ""))

    roadmap["executive_summary"] = (
        f"Scanned {total} TLS endpoints: {vuln_count} ({vuln_count/total*100:.0f}%) "
        f"use quantum-vulnerable algorithms. {tls13_count} ({tls13_count/total*100:.0f}%) "
        f"support TLS 1.3 (prerequisite for hybrid PQC). "
        f"NIST mandates deprecation of quantum-vulnerable algorithms by 2030 and "
        f"complete removal by 2035."
    ) if total > 0 else "No endpoints scanned."

    # Phase 1: Immediate (0-6 months)
    phase1_actions = [
        {
            "action": "Complete cryptographic inventory across all systems",
            "priority": "P0",
            "effort": "medium",
            "description": "Extend scanning beyond TLS to include code libraries, "
                           "key stores, HSMs, certificates, VPN configurations, "
                           "and embedded systems.",
        },
        {
            "action": "Upgrade OpenSSL to 3.5+ on development and staging systems",
            "priority": "P0",
            "effort": "low",
            "description": "OpenSSL 3.5 provides native ML-KEM, ML-DSA, SLH-DSA support. "
                           "For OpenSSL 3.0-3.4, install oqs-provider as interim solution.",
        },
        {
            "action": "Enable X25519MLKEM768 hybrid key exchange on TLS 1.3 endpoints",
            "priority": "P1",
            "effort": "low",
            "description": "Add X25519MLKEM768 to supported_groups in TLS configuration. "
                           "This is a drop-in change for servers already on TLS 1.3 with "
                           "OpenSSL 3.5+ or oqs-provider.",
        },
    ]

    # Phase 2: Short-term (6-18 months)
    phase2_actions = [
        {
            "action": "Upgrade all endpoints to TLS 1.3",
            "priority": "P1",
            "effort": "medium",
            "description": f"{total - tls13_count} endpoints need TLS 1.3 upgrade. "
                           "Hybrid PQC key exchange is only available in TLS 1.3.",
        },
        {
            "action": "Deploy hybrid key exchange across production infrastructure",
            "priority": "P1",
            "effort": "medium",
            "description": "Configure X25519MLKEM768 as preferred key exchange group "
                           "on all production TLS endpoints.",
        },
        {
            "action": "Test ML-DSA certificate chains in staging environments",
            "priority": "P2",
            "effort": "high",
            "description": "Issue test certificates with ML-DSA signatures from internal CA. "
                           "Validate certificate chain verification across all clients.",
        },
        {
            "action": "Assess HSM and KMS PQC compatibility",
            "priority": "P2",
            "effort": "medium",
            "description": "Verify that hardware security modules and key management "
                           "systems support PQC key sizes and algorithms.",
        },
    ]

    # Phase 3: Medium-term (18-36 months)
    phase3_actions = [
        {
            "action": "Migrate certificate infrastructure to hybrid or PQC signatures",
            "priority": "P2",
            "effort": "high",
            "description": "Deploy hybrid certificates (classical + ML-DSA) for backward "
                           "compatibility, then transition to pure ML-DSA.",
        },
        {
            "action": "Update code signing and software supply chain to PQC",
            "priority": "P2",
            "effort": "high",
            "description": "Migrate code signing certificates, package signatures, "
                           "and firmware signing to ML-DSA or SLH-DSA.",
        },
        {
            "action": "Replace quantum-vulnerable VPN and IPsec configurations",
            "priority": "P2",
            "effort": "medium",
            "description": "Upgrade VPN concentrators and IPsec configurations to "
                           "support PQC key exchange.",
        },
    ]

    # Phase 4: Long-term (36-60 months, by 2030)
    phase4_actions = [
        {
            "action": "Complete deprecation of all quantum-vulnerable algorithms",
            "priority": "P3",
            "effort": "high",
            "description": "Remove RSA, ECDH, ECDSA, DH, DSA from all systems. "
                           "Ensure 100% PQC coverage before NIST 2030 deadline.",
        },
        {
            "action": "Validate SLH-DSA (FIPS 205) as backup signature standard",
            "priority": "P3",
            "effort": "low",
            "description": "Maintain tested SLH-DSA deployment capability as fallback "
                           "in case ML-DSA is found vulnerable.",
        },
    ]

    roadmap["phases"] = [
        {"name": "Phase 1: Discovery and Quick Wins", "timeline": "0-6 months",
         "actions": phase1_actions},
        {"name": "Phase 2: Hybrid Deployment", "timeline": "6-18 months",
         "actions": phase2_actions},
        {"name": "Phase 3: Full PQC Migration", "timeline": "18-36 months",
         "actions": phase3_actions},
        {"name": "Phase 4: Algorithm Deprecation", "timeline": "36-60 months",
         "actions": phase4_actions},
    ]

    # Quick wins (can be done immediately with minimal effort)
    if tls13_count > 0:
        roadmap["quick_wins"].append({
            "action": f"Enable X25519MLKEM768 on {tls13_count} TLS 1.3 endpoints",
            "effort": "configuration change only",
            "impact": "Immediate quantum-resistant key exchange for existing TLS 1.3 servers",
        })

    roadmap["quick_wins"].append({
        "action": "Increase AES key sizes to 256-bit where currently using 128-bit",
        "effort": "configuration change",
        "impact": "Grover's algorithm halves effective symmetric key strength; "
                  "AES-256 provides 128-bit post-quantum security",
    })

    # Risk register
    roadmap["risk_register"] = [
        {
            "risk": "Harvest Now, Decrypt Later (HNDL) attacks",
            "description": "Adversaries record encrypted traffic today to decrypt when "
                           "quantum computers become available",
            "likelihood": "high",
            "impact": "critical for long-lived secrets (government, healthcare, finance)",
            "mitigation": "Priority migration of systems handling data with >10yr confidentiality",
        },
        {
            "risk": "Algorithm implementation vulnerabilities",
            "description": "Side-channel attacks or implementation bugs in new PQC libraries",
            "likelihood": "medium",
            "impact": "high",
            "mitigation": "Use NIST-validated implementations, conduct security audits, "
                          "deploy hybrid schemes for defense-in-depth",
        },
        {
            "risk": "Performance degradation",
            "description": "PQC algorithms have larger key/signature sizes and may be slower",
            "likelihood": "medium",
            "impact": "medium",
            "mitigation": "Benchmark PQC under production load, optimize TLS handshake "
                          "configurations, consider ML-KEM-768 (balanced performance)",
        },
        {
            "risk": "Compatibility issues",
            "description": "Older clients/devices may not support PQC algorithms",
            "likelihood": "high",
            "impact": "medium",
            "mitigation": "Hybrid schemes ensure backward compatibility; maintain classical "
                          "fallback during transition",
        },
    ]

    return roadmap


# ---------------------------------------------------------------------------
# OpenSSL and oqs-provider Configuration
# ---------------------------------------------------------------------------

def check_openssl_pqc_support():
    """
    Check the local OpenSSL installation for PQC algorithm support.

    Returns:
        Dict with OpenSSL version, provider status, and PQC algorithm availability
    """
    result = {
        "check_time": datetime.now(timezone.utc).isoformat(),
        "openssl_version": "",
        "providers": [],
        "pqc_kem_algorithms": [],
        "pqc_signature_algorithms": [],
        "hybrid_groups": [],
        "pqc_ready": False,
    }

    # Get OpenSSL version
    try:
        proc = subprocess.run(["openssl", "version", "-a"],
                              capture_output=True, timeout=5)
        result["openssl_version"] = proc.stdout.decode().strip()
    except (FileNotFoundError, subprocess.TimeoutExpired):
        result["error"] = "openssl not found"
        return result

    # List providers
    try:
        proc = subprocess.run(["openssl", "list", "-providers"],
                              capture_output=True, timeout=5)
        output = proc.stdout.decode()
        result["providers"] = [
            line.strip() for line in output.split("\n")
            if line.strip() and "name:" in line.lower()
        ]
    except (FileNotFoundError, subprocess.TimeoutExpired):
        pass

    # Check for PQC KEM algorithms
    try:
        proc = subprocess.run(["openssl", "list", "-kem-algorithms"],
                              capture_output=True, timeout=5)
        output = proc.stdout.decode()
        for line in output.split("\n"):
            line = line.strip().lower()
            if any(pqc in line for pqc in ["mlkem", "kyber", "bike", "hqc", "frodo"]):
                result["pqc_kem_algorithms"].append(line)
    except (FileNotFoundError, subprocess.TimeoutExpired):
        pass

    # Check for PQC signature algorithms
    try:
        proc = subprocess.run(["openssl", "list", "-signature-algorithms"],
                              capture_output=True, timeout=5)
        output = proc.stdout.decode()
        for line in output.split("\n"):
            line = line.strip().lower()
            if any(pqc in line for pqc in ["mldsa", "dilithium", "slhdsa", "sphincs", "falcon"]):
                result["pqc_signature_algorithms"].append(line)
    except (FileNotFoundError, subprocess.TimeoutExpired):
        pass

    # Check for hybrid groups
    try:
        proc = subprocess.run(["openssl", "list", "-tls1-3-groups"],
                              capture_output=True, timeout=5)
        output = proc.stdout.decode()
        for line in output.split("\n"):
            line_stripped = line.strip()
            if any(pqc in line_stripped.lower() for pqc in ["mlkem", "kyber"]):
                result["hybrid_groups"].append(line_stripped)
    except (FileNotFoundError, subprocess.TimeoutExpired):
        pass

    result["pqc_ready"] = bool(
        result["pqc_kem_algorithms"] or result["pqc_signature_algorithms"]
    )

    return result


def generate_oqs_provider_config():
    """
    Generate an OpenSSL configuration file for oqs-provider.

    Returns the configuration text for enabling PQC algorithms via
    the Open Quantum Safe provider in OpenSSL 3.x.
    """
    config = """# OpenSSL configuration for oqs-provider (Post-Quantum Cryptography)
# Place at /etc/ssl/openssl-oqs.cnf
# Set OPENSSL_CONF=/etc/ssl/openssl-oqs.cnf before running OpenSSL/nginx/Apache

openssl_conf = openssl_init

[openssl_init]
providers = provider_sect
ssl_conf = ssl_sect

[provider_sect]
default = default_sect
oqsprovider = oqsprovider_sect

[default_sect]
activate = 1

[oqsprovider_sect]
activate = 1
# Adjust path to match your oqs-provider installation
module = /usr/lib/oqs-provider/oqsprovider.so

[ssl_sect]
system_default = system_default_sect

[system_default_sect]
# Hybrid PQC groups: prefer X25519MLKEM768 with classical fallbacks
Groups = x25519_mlkem768:X25519:P-256:P-384

# Minimum TLS version (1.3 required for PQC key exchange)
MinProtocol = TLSv1.2
"""
    return config


# ---------------------------------------------------------------------------
# Main CLI
# ---------------------------------------------------------------------------

def main():
    parser = argparse.ArgumentParser(
        description="Post-Quantum Cryptography Migration Assessment Agent",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Actions:
  scan_tls        Scan TLS endpoints for quantum-vulnerable algorithms
  assess_agility  Assess crypto-agility from scan results
  test_hybrid_tls Test hybrid PQC TLS key exchange support
  test_mlkem      Test ML-KEM (FIPS 203) key encapsulation
  test_mldsa      Test ML-DSA (FIPS 204) digital signatures
  check_openssl   Check local OpenSSL PQC algorithm support
  roadmap         Generate prioritized PQC migration roadmap
  full_assessment Run complete assessment pipeline

Examples:
  python agent.py --action scan_tls --targets hosts.txt --output scan.json
  python agent.py --action scan_tls --target server.example.com:443
  python agent.py --action test_hybrid_tls --target server.example.com:443
  python agent.py --action test_mlkem --output mlkem.json
  python agent.py --action check_openssl
  python agent.py --action roadmap --scan-results scan.json --output roadmap.json
  python agent.py --action full_assessment --targets hosts.txt --output full_report.json
        """,
    )
    parser.add_argument("--action", required=True, choices=[
        "scan_tls", "assess_agility", "test_hybrid_tls",
        "test_mlkem", "test_mldsa", "check_openssl",
        "roadmap", "full_assessment",
    ])
    parser.add_argument("--target", default=None,
                        help="Single target host[:port] for scanning")
    parser.add_argument("--targets", default=None,
                        help="File with one host[:port] per line")
    parser.add_argument("--port", type=int, default=443,
                        help="Default port for TLS scanning")
    parser.add_argument("--scan-results", default=None,
                        help="Path to previous scan results JSON (for assess/roadmap)")
    parser.add_argument("--agility-results", default=None,
                        help="Path to agility assessment JSON (for roadmap)")
    parser.add_argument("--output", default="pqc_report.json",
                        help="Output file for results")
    args = parser.parse_args()

    report = {
        "agent": "pqc-migration-assessment",
        "generated_at": datetime.now(timezone.utc).isoformat(),
        "action": args.action,
        "nist_standards": {
            "FIPS_203": "ML-KEM (CRYSTALS-Kyber) -- Key Encapsulation",
            "FIPS_204": "ML-DSA (CRYSTALS-Dilithium) -- Digital Signatures",
            "FIPS_205": "SLH-DSA (SPHINCS+) -- Digital Signatures (backup)",
        },
    }

    # --- TLS Scanning ---
    scan_results = []
    if args.action in ("scan_tls", "full_assessment"):
        if args.targets:
            scan_results = scan_multiple_endpoints(args.targets, args.port)
        elif args.target:
            host_port = args.target.split(":")
            host = host_port[0]
            port = int(host_port[1]) if len(host_port) > 1 else args.port
            scan_results = [scan_tls_endpoint(host, port)]
        else:
            print("[!] Provide --target or --targets for TLS scanning")
            sys.exit(1)

        report["tls_scan"] = scan_results
        vuln = sum(1 for r in scan_results if r.get("quantum_vulnerable"))
        print(f"[+] Scanned {len(scan_results)} endpoints: "
              f"{vuln} quantum-vulnerable")
        for r in scan_results:
            status = "VULNERABLE" if r.get("quantum_vulnerable") else "OK"
            err = r.get("error", "")
            if err:
                print(f"    {r['host']}:{r['port']} -- ERROR: {err}")
            else:
                print(f"    {r['host']}:{r['port']} -- [{status}] "
                      f"{r.get('cipher_suite', 'N/A')} ({r.get('tls_version', 'N/A')})")

    # --- Crypto-Agility Assessment ---
    if args.action in ("assess_agility", "full_assessment"):
        if not scan_results and args.scan_results:
            with open(args.scan_results, encoding="utf-8") as f:
                data = json.load(f)
                scan_results = data.get("tls_scan", data) if isinstance(data, dict) else data

        if scan_results:
            agility = assess_crypto_agility(scan_results)
            report["agility_assessment"] = agility
            print(f"[+] Crypto-agility score: {agility['agility_score']}/100")
            print(f"    Risk level: {agility['risk_summary'].get('risk_level', 'unknown')}")
            print(f"    TLS 1.3 ready: {agility['risk_summary'].get('tls13_percentage', 0)}%")
        else:
            print("[!] No scan results available for agility assessment")

    # --- Hybrid TLS Testing ---
    if args.action in ("test_hybrid_tls", "full_assessment"):
        target = args.target
        if not target and scan_results:
            target = f"{scan_results[0]['host']}:{scan_results[0]['port']}"
        if target:
            host_port = target.split(":")
            host = host_port[0]
            port = int(host_port[1]) if len(host_port) > 1 else 443
            hybrid_result = test_hybrid_tls_support(host, port)
            report["hybrid_tls"] = hybrid_result
            pqc_status = "SUPPORTED" if hybrid_result["pqc_supported"] else "NOT SUPPORTED"
            print(f"[+] Hybrid TLS (X25519MLKEM768): {pqc_status}")
            print(f"    OpenSSL: {hybrid_result.get('openssl_version', 'unknown')}")
            for group_test in hybrid_result.get("hybrid_groups_tested", []):
                status = "OK" if group_test.get("supported") else "FAIL"
                print(f"    {group_test['group']}: [{status}] "
                      f"{group_test.get('error', '')}")

    # --- ML-KEM Testing ---
    if args.action in ("test_mlkem", "full_assessment"):
        mlkem_result = test_mlkem_support()
        report["mlkem_validation"] = mlkem_result
        print(f"[+] ML-KEM (FIPS 203) validation via {mlkem_result.get('library', 'N/A')}:")
        for level, result in mlkem_result.get("levels", {}).items():
            status = "PASS" if result.get("supported") else "FAIL"
            perf = result.get("performance", {})
            perf_str = ""
            if perf:
                perf_str = (f" (keygen={perf.get('keygen_ms', '?')}ms, "
                            f"encaps={perf.get('encaps_ms', '?')}ms, "
                            f"decaps={perf.get('decaps_ms', '?')}ms)")
            print(f"    {level}: [{status}]{perf_str}")

    # --- ML-DSA Testing ---
    if args.action in ("test_mldsa", "full_assessment"):
        mldsa_result = test_mldsa_support()
        report["mldsa_validation"] = mldsa_result
        print(f"[+] ML-DSA (FIPS 204) validation:")
        for level, result in mldsa_result.get("levels", {}).items():
            status = "PASS" if result.get("supported") else "FAIL"
            err = result.get("error", "")
            print(f"    {level}: [{status}] {err}")

    # --- OpenSSL PQC Check ---
    if args.action in ("check_openssl", "full_assessment"):
        ossl = check_openssl_pqc_support()
        report["openssl_pqc"] = ossl
        print(f"[+] OpenSSL PQC support check:")
        print(f"    Version: {ossl.get('openssl_version', 'unknown')}")
        print(f"    PQC ready: {ossl.get('pqc_ready', False)}")
        if ossl.get("pqc_kem_algorithms"):
            print(f"    KEM algorithms: {', '.join(ossl['pqc_kem_algorithms'][:5])}")
        if ossl.get("pqc_signature_algorithms"):
            print(f"    Signature algorithms: {', '.join(ossl['pqc_signature_algorithms'][:5])}")
        if ossl.get("hybrid_groups"):
            print(f"    Hybrid groups: {', '.join(ossl['hybrid_groups'][:5])}")

        if not ossl.get("pqc_ready"):
            print("\n[*] To enable PQC, either:")
            print("    1. Upgrade to OpenSSL 3.5+ (native ML-KEM/ML-DSA)")
            print("    2. Install oqs-provider for OpenSSL 3.0+:")
            print("       https://github.com/open-quantum-safe/oqs-provider")
            config = generate_oqs_provider_config()
            report["oqs_provider_config"] = config

    # --- Migration Roadmap ---
    if args.action in ("roadmap", "full_assessment"):
        if not scan_results and args.scan_results:
            with open(args.scan_results, encoding="utf-8") as f:
                data = json.load(f)
                scan_results = data.get("tls_scan", data) if isinstance(data, dict) else data

        agility = None
        if args.agility_results:
            with open(args.agility_results, encoding="utf-8") as f:
                agility = json.load(f)

        if scan_results:
            roadmap = generate_migration_roadmap(scan_results, agility)
            report["migration_roadmap"] = roadmap
            print(f"\n[+] Migration Roadmap")
            print(f"    {roadmap['executive_summary']}")
            print(f"\n    NIST Timeline: deprecation by {NIST_MIGRATION_DEADLINES['deprecation']}, "
                  f"removal by {NIST_MIGRATION_DEADLINES['disallowed']}")
            for phase in roadmap["phases"]:
                print(f"\n    {phase['name']} ({phase['timeline']}):")
                for action in phase["actions"]:
                    print(f"      [{action['priority']}] {action['action']}")
            if roadmap["quick_wins"]:
                print(f"\n    Quick Wins:")
                for qw in roadmap["quick_wins"]:
                    print(f"      - {qw['action']}")
        else:
            print("[!] No scan results available for roadmap generation")

    # --- Write report ---
    output_path = Path(args.output)
    output_path.parent.mkdir(parents=True, exist_ok=True)
    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-ssl-certificate-lifecycle-management/SKILL.md =====
---
name: performing-ssl-certificate-lifecycle-management
description: Automates the full SSL/TLS certificate lifecycle, including generating
  Certificate Signing Requests, issuing, deploying, monitoring, renewing, and revoking
  X.509 certificates, using Python and ACME protocol tools. Use when managing certificate
  issuance or renewal, preventing certificate-expiry outages, or building automated
  PKI/ACME workflows.
domain: cybersecurity
subdomain: cryptography
tags:
- cryptography
- ssl
- certificates
- pki
- tls
- key-management
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
mitre_attack:
- T1600
- T1573
- T1553
- T1040
---
# Performing SSL Certificate Lifecycle Management

## Overview

SSL/TLS certificate lifecycle management encompasses the full process of requesting, issuing, deploying, monitoring, renewing, and revoking X.509 certificates. Poor certificate management is a leading cause of outages and security incidents. This skill covers automating the entire certificate lifecycle using Python and ACME protocol tools.


## When to Use

- When conducting security assessments that involve performing ssl certificate lifecycle management
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Familiarity with cryptography concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Objectives

- Generate Certificate Signing Requests (CSRs) programmatically
- Parse and validate X.509 certificates
- Monitor certificate expiration across infrastructure
- Automate renewal using ACME protocol (Let's Encrypt)
- Implement certificate revocation checking (CRL and OCSP)
- Track certificate inventory across multiple domains

## Key Concepts

### Certificate Lifecycle Stages

1. **Request**: Generate key pair and CSR
2. **Issuance**: CA validates and issues certificate
3. **Deployment**: Install certificate on servers
4. **Monitoring**: Track expiration and health
5. **Renewal**: Request new certificate before expiry
6. **Revocation**: Invalidate compromised certificates

### Certificate Types

| Type | Validation | Use Case |
|------|-----------|----------|
| DV (Domain Validation) | Domain ownership | Websites, APIs |
| OV (Organization Validation) | Domain + org identity | Business sites |
| EV (Extended Validation) | Full legal verification | E-commerce, banking |
| Wildcard | *.domain.com | Multi-subdomain |
| SAN/UCC | Multiple domains | Multi-domain hosting |

## Security Considerations

- Set up automated monitoring for all certificates
- Use ECDSA (P-256) certificates for better performance over RSA
- Enable OCSP stapling on all servers
- Implement Certificate Transparency log monitoring
- Maintain inventory of all certificates and their locations
- Plan for CA compromise scenarios (key pinning, backup CAs)

## Validation Criteria

- [ ] CSR generation produces valid PKCS#10 request
- [ ] Certificate parsing extracts all relevant fields
- [ ] Expiration monitoring detects certificates within threshold
- [ ] Certificate chain validation verifies trust path
- [ ] OCSP checking detects revoked certificates
- [ ] Certificate inventory tracks all deployed certificates

===== END FILE =====


===== FILE: skills/performing-ssl-certificate-lifecycle-management/LICENSE =====

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


===== FILE: skills/performing-ssl-certificate-lifecycle-management/assets/template.md =====
# SSL Certificate Lifecycle Management Template

## Certificate Inventory Template

| Domain | Type | CA | Issued | Expires | Days Left | Status |
|--------|------|-----|--------|---------|-----------|--------|
| example.com | DV | Let's Encrypt | 2024-01-01 | 2024-04-01 | 90 | OK |
| api.example.com | DV | DigiCert | 2024-01-01 | 2025-01-01 | 365 | OK |

## Monitoring Thresholds

```yaml
monitoring:
  ok_threshold: 30        # days
  warning_threshold: 15   # days
  critical_threshold: 7   # days
  check_interval: 86400   # seconds (daily)
  notification:
    email: security@example.com
    slack: "#cert-alerts"
```

## CSR Generation Command

```bash
# ECDSA (recommended)
openssl ecparam -genkey -name prime256v1 -out server.key
openssl req -new -key server.key -out server.csr -subj "/CN=example.com"

# RSA 4096
openssl genrsa -out server.key 4096
openssl req -new -key server.key -out server.csr -subj "/CN=example.com"
```

## Renewal Automation (certbot)

```bash
# Initial issuance
certbot certonly --nginx -d example.com -d www.example.com

# Auto-renewal (cron)
0 0 * * * certbot renew --quiet --deploy-hook "systemctl reload nginx"
```

## Revocation Checklist

- [ ] Identify affected certificate(s)
- [ ] Contact CA to initiate revocation
- [ ] Provide revocation reason (key compromise, cessation, etc.)
- [ ] Verify revocation in CRL/OCSP
- [ ] Issue replacement certificate
- [ ] Deploy replacement to all affected servers
- [ ] Update certificate inventory
- [ ] Document incident

===== END FILE =====


===== FILE: skills/performing-ssl-certificate-lifecycle-management/references/api-reference.md =====
# API Reference: SSL Certificate Lifecycle Management

## cryptography Library - CSR Generation

| Class / Method | Description |
|----------------|-------------|
| `ec.generate_private_key(ec.SECP256R1())` | Generate ECDSA P-256 private key |
| `rsa.generate_private_key(65537, 2048)` | Generate RSA 2048-bit private key |
| `x509.CertificateSigningRequestBuilder()` | Build a PKCS#10 CSR |
| `.subject_name(x509.Name([...]))` | Set CSR subject |
| `.add_extension(SubjectAlternativeName(...))` | Add SAN extension |
| `.sign(private_key, hashes.SHA256())` | Sign CSR with private key |

## cryptography Library - Certificate Parsing

| Method | Description |
|--------|-------------|
| `x509.load_pem_x509_certificate(data)` | Parse PEM certificate |
| `x509.load_der_x509_certificate(data)` | Parse DER certificate |
| `cert.subject` | Get subject Distinguished Name |
| `cert.issuer` | Get issuer Distinguished Name |
| `cert.not_valid_after_utc` | Expiration datetime |
| `cert.serial_number` | Certificate serial number |
| `cert.extensions.get_extension_for_oid(OID)` | Get specific extension |

## Python ssl Module

| Function | Description |
|----------|-------------|
| `ssl.create_default_context()` | Create SSL context with system CAs |
| `ctx.wrap_socket(sock, server_hostname=host)` | TLS handshake |
| `s.getpeercert(binary_form=True)` | Get DER-encoded server certificate |
| `s.getpeercert()` | Get parsed certificate dict |

## Certificate Types

| Type | Validation | Typical Use |
|------|-----------|-------------|
| DV | Domain ownership | Websites, APIs |
| OV | Organization verified | Business applications |
| EV | Full legal verification | E-commerce, banking |
| Wildcard | `*.domain.com` | Multi-subdomain |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `cryptography` | >=41.0 | CSR generation, certificate parsing |
| `ssl` | stdlib | TLS handshake, remote cert fetch |
| `socket` | stdlib | TCP connections |

## References

- cryptography docs: https://cryptography.io/en/latest/x509/
- Let's Encrypt ACME: https://letsencrypt.org/docs/
- OCSP Stapling: https://datatracker.ietf.org/doc/html/rfc6960
- Certificate Transparency: https://certificate.transparency.dev/

===== END FILE =====


===== FILE: skills/performing-ssl-certificate-lifecycle-management/references/standards.md =====
# Standards and References - SSL Certificate Lifecycle Management

## Primary Standards

### RFC 5280 - Internet X.509 PKI Certificate and CRL Profile
- **URL**: https://www.rfc-editor.org/rfc/rfc5280
- **Description**: Core X.509 certificate format and Certificate Revocation List (CRL)

### RFC 6960 - X.509 Online Certificate Status Protocol (OCSP)
- **URL**: https://www.rfc-editor.org/rfc/rfc6960
- **Description**: Real-time certificate revocation checking

### RFC 8555 - Automatic Certificate Management Environment (ACME)
- **URL**: https://www.rfc-editor.org/rfc/rfc8555
- **Description**: Protocol for automating certificate issuance (Let's Encrypt)

### RFC 6962 - Certificate Transparency
- **URL**: https://www.rfc-editor.org/rfc/rfc6962
- **Description**: Public logging framework for TLS certificates

### RFC 2986 - PKCS #10: Certification Request Syntax
- **URL**: https://www.rfc-editor.org/rfc/rfc2986
- **Description**: CSR format specification

### NIST SP 800-57 Part 3 - Application-Specific Key Management
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-57-part-3/rev-1/final
- **Description**: Key management guidance for TLS certificates

## Tools

### Let's Encrypt / Certbot
- **URL**: https://letsencrypt.org/
- **Certbot**: https://certbot.eff.org/
- **Rate limits**: 50 certificates per domain per week

### Certificate Transparency Logs
- **Google**: https://ct.googleapis.com/logs
- **crt.sh**: https://crt.sh/ (certificate search)

### Mozilla Observatory
- **URL**: https://observatory.mozilla.org/
- **Description**: Web security scanning including TLS configuration

===== END FILE =====


===== FILE: skills/performing-ssl-certificate-lifecycle-management/references/workflows.md =====
# Workflows - SSL Certificate Lifecycle Management

## Workflow 1: Certificate Request and Issuance

```
[Generate Private Key] (ECDSA P-256 or RSA 4096)
      |
[Create CSR] (PKCS#10)
(CN, SAN, Organization, etc.)
      |
[Submit CSR to CA]
      |
[CA Validates Domain/Org]
(DNS, HTTP, or Email challenge)
      |
[CA Issues Certificate]
      |
[Download Certificate + Chain]
      |
[Verify Certificate Chain]
      |
[Deploy to Server]
```

## Workflow 2: Expiration Monitoring

```
[Certificate Inventory] (list of all domains/endpoints)
      |
[For Each Endpoint]:
  [Connect and retrieve certificate]
  [Parse notAfter field]
  [Calculate days remaining]
      |
[Apply Threshold Rules]:
  > 30 days: OK
  15-30 days: WARNING
  < 15 days: CRITICAL
  Expired: ALERT
      |
[Generate Report / Send Alerts]
```

## Workflow 3: Automated Renewal (ACME)

```
[Cron Job / Scheduler]
      |
[Check Certificate Expiry]
      |
[< 30 days remaining?]
  NO  --> Sleep
  YES --> [Initiate ACME Renewal]
              |
          [Complete Challenge]
          (HTTP-01, DNS-01, TLS-ALPN-01)
              |
          [Receive New Certificate]
              |
          [Deploy and Reload Server]
              |
          [Verify New Certificate Works]
```

## Workflow 4: Certificate Revocation

```
[Security Incident Detected]
(key compromise, CA breach, etc.)
      |
[Revoke Certificate with CA]
(provide reason code)
      |
[Verify in CRL / OCSP]
      |
[Issue Replacement Certificate]
      |
[Deploy Replacement]
      |
[Update Certificate Inventory]
```

===== END FILE =====


===== FILE: skills/performing-ssl-certificate-lifecycle-management/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for SSL/TLS certificate lifecycle management.

Generates CSRs, parses X.509 certificates using the cryptography
library, monitors expiration across infrastructure, checks OCSP
revocation status, and maintains a certificate inventory.
"""

import json
import sys
import ssl
import socket
from datetime import datetime
from pathlib import Path

try:
    from cryptography import x509
    from cryptography.x509.oid import NameOID
    from cryptography.hazmat.primitives import hashes, serialization
    from cryptography.hazmat.primitives.asymmetric import ec, rsa
    HAS_CRYPTO = True
except ImportError:
    HAS_CRYPTO = False


class CertLifecycleAgent:
    """Manages SSL/TLS certificate lifecycle operations."""

    def __init__(self, output_dir="./cert_inventory"):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.inventory = []

    def generate_csr(self, common_name, org="", country="US",
                     san_names=None, key_type="ecdsa"):
        """Generate a private key and Certificate Signing Request."""
        if not HAS_CRYPTO:
            return {"error": "cryptography library required"}

        if key_type == "ecdsa":
            private_key = ec.generate_private_key(ec.SECP256R1())
        else:
            private_key = rsa.generate_private_key(
                public_exponent=65537, key_size=2048)

        subject = x509.Name([
            x509.NameAttribute(NameOID.COUNTRY_NAME, country),
            x509.NameAttribute(NameOID.ORGANIZATION_NAME, org or common_name),
            x509.NameAttribute(NameOID.COMMON_NAME, common_name),
        ])

        builder = x509.CertificateSigningRequestBuilder().subject_name(subject)

        if san_names:
            sans = [x509.DNSName(n) for n in san_names]
            builder = builder.add_extension(
                x509.SubjectAlternativeName(sans), critical=False)

        csr = builder.sign(private_key, hashes.SHA256())

        key_path = self.output_dir / f"{common_name}.key"
        csr_path = self.output_dir / f"{common_name}.csr"

        key_path.write_bytes(private_key.private_bytes(
            serialization.Encoding.PEM,
            serialization.PrivateFormat.PKCS8,
            serialization.NoEncryption()))
        csr_path.write_bytes(csr.public_bytes(serialization.Encoding.PEM))

        return {"common_name": common_name, "key_file": str(key_path),
                "csr_file": str(csr_path), "key_type": key_type}

    def fetch_remote_cert(self, hostname, port=443):
        """Fetch and parse a certificate from a remote server."""
        try:
            ctx = ssl.create_default_context()
            with ctx.wrap_socket(socket.socket(), server_hostname=hostname) as s:
                s.settimeout(10)
                s.connect((hostname, port))
                der = s.getpeercert(binary_form=True)
                pem_info = s.getpeercert()

            not_after = datetime.strptime(
                pem_info["notAfter"], "%b %d %H:%M:%S %Y %Z")
            not_before = datetime.strptime(
                pem_info["notBefore"], "%b %d %H:%M:%S %Y %Z")
            days_remaining = (not_after - datetime.utcnow()).days

            subject = dict(x[0] for x in pem_info.get("subject", ()))
            issuer = dict(x[0] for x in pem_info.get("issuer", ()))
            sans = [entry[1] for entry in pem_info.get("subjectAltName", ())]

            entry = {
                "hostname": hostname, "port": port,
                "subject_cn": subject.get("commonName", ""),
                "issuer_cn": issuer.get("commonName", ""),
                "issuer_org": issuer.get("organizationName", ""),
                "not_before": not_before.isoformat(),
                "not_after": not_after.isoformat(),
                "days_remaining": days_remaining,
                "san": sans[:20],
                "serial": pem_info.get("serialNumber", ""),
                "version": pem_info.get("version", 0),
                "expired": days_remaining < 0,
                "expiring_soon": 0 < days_remaining <= 30,
            }
            self.inventory.append(entry)
            return entry

        except (socket.error, ssl.SSLError, OSError) as exc:
            return {"hostname": hostname, "error": str(exc)}

    def scan_hosts(self, hostnames, port=443):
        """Scan multiple hosts and collect certificate data."""
        results = []
        for host in hostnames:
            result = self.fetch_remote_cert(host, port)
            results.append(result)
        return results

    def check_expiring(self, threshold_days=30):
        """Return certificates expiring within threshold days."""
        return [c for c in self.inventory
                if c.get("days_remaining", 999) <= threshold_days
                and "error" not in c]

    def generate_report(self):
        """Generate certificate inventory report."""
        expiring = self.check_expiring(30)
        expired = [c for c in self.inventory if c.get("expired")]

        report = {
            "report_date": datetime.utcnow().isoformat(),
            "total_certificates": len(self.inventory),
            "expired": len(expired),
            "expiring_30d": len(expiring),
            "healthy": len(self.inventory) - len(expired) - len(expiring),
            "certificates": self.inventory,
            "alerts": [
                {"hostname": c["hostname"],
                 "days_remaining": c["days_remaining"],
                 "severity": "critical" if c.get("expired") else "warning"}
                for c in expired + expiring
            ],
        }

        report_path = self.output_dir / "cert_inventory_report.json"
        with open(report_path, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(json.dumps(report, indent=2, default=str))
        return report


def main():
    hosts = sys.argv[1:] if len(sys.argv) > 1 else [
        "google.com", "github.com", "expired.badssl.com"]
    agent = CertLifecycleAgent()
    agent.scan_hosts(hosts)
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-ssl-certificate-lifecycle-management/scripts/process.py =====
#!/usr/bin/env python3
"""
SSL Certificate Lifecycle Management Tool

Implements certificate generation, parsing, monitoring, chain validation,
and OCSP checking for managing TLS certificate lifecycles.

Requirements:
    pip install cryptography requests

Usage:
    python process.py generate-csr --domain example.com --output ./certs
    python process.py check-expiry --host example.com
    python process.py parse-cert --cert ./server.crt
    python process.py monitor --domains domains.txt --threshold 30
    python process.py verify-chain --cert ./server.crt --ca-bundle ./ca-bundle.crt
"""

import os
import ssl
import sys
import json
import socket
import argparse
import logging
import datetime
from pathlib import Path
from typing import Dict, List, Optional

from cryptography import x509
from cryptography.x509.oid import NameOID, ExtensionOID
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import ec, rsa
from cryptography.hazmat.backends import default_backend

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

EXPIRY_WARNING_DAYS = 30
EXPIRY_CRITICAL_DAYS = 15


def generate_csr(
    domain: str,
    output_dir: str,
    key_type: str = "ecdsa",
    san_domains: Optional[List[str]] = None,
    organization: Optional[str] = None,
) -> Dict:
    """Generate a private key and CSR for a domain."""
    output_path = Path(output_dir)
    output_path.mkdir(parents=True, exist_ok=True)

    if key_type == "ecdsa":
        private_key = ec.generate_private_key(ec.SECP256R1(), default_backend())
    else:
        private_key = rsa.generate_private_key(
            public_exponent=65537, key_size=4096, backend=default_backend()
        )

    subject_attrs = [
        x509.NameAttribute(NameOID.COMMON_NAME, domain),
    ]
    if organization:
        subject_attrs.insert(0, x509.NameAttribute(NameOID.ORGANIZATION_NAME, organization))

    subject = x509.Name(subject_attrs)

    san_list = [x509.DNSName(domain)]
    if san_domains:
        for d in san_domains:
            san_list.append(x509.DNSName(d))

    csr = (
        x509.CertificateSigningRequestBuilder()
        .subject_name(subject)
        .add_extension(x509.SubjectAlternativeName(san_list), critical=False)
        .sign(private_key, hashes.SHA256(), default_backend())
    )

    key_path = output_path / f"{domain}.key"
    csr_path = output_path / f"{domain}.csr"

    key_path.write_bytes(
        private_key.private_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PrivateFormat.PKCS8,
            encryption_algorithm=serialization.NoEncryption(),
        )
    )

    csr_path.write_bytes(csr.public_bytes(serialization.Encoding.PEM))

    logger.info(f"Generated CSR for {domain}")

    return {
        "domain": domain,
        "key_type": key_type,
        "key_path": str(key_path),
        "csr_path": str(csr_path),
        "san_domains": [d.value for d in san_list],
    }


def parse_certificate(cert_path: str) -> Dict:
    """Parse an X.509 certificate and extract key information."""
    cert_data = Path(cert_path).read_bytes()

    if b"-----BEGIN CERTIFICATE-----" in cert_data:
        cert = x509.load_pem_x509_certificate(cert_data, default_backend())
    else:
        cert = x509.load_der_x509_certificate(cert_data, default_backend())

    subject_attrs = {}
    for attr in cert.subject:
        subject_attrs[attr.oid._name] = attr.value

    issuer_attrs = {}
    for attr in cert.issuer:
        issuer_attrs[attr.oid._name] = attr.value

    san_names = []
    try:
        san_ext = cert.extensions.get_extension_for_oid(ExtensionOID.SUBJECT_ALTERNATIVE_NAME)
        san_names = [name.value for name in san_ext.value.get_values_for_type(x509.DNSName)]
    except x509.ExtensionNotFound:
        pass

    now = datetime.datetime.utcnow()
    not_after = cert.not_valid_after_utc.replace(tzinfo=None)
    days_remaining = (not_after - now).days

    pub_key = cert.public_key()
    if isinstance(pub_key, rsa.RSAPublicKey):
        key_info = {"type": "RSA", "size": pub_key.key_size}
    elif isinstance(pub_key, ec.EllipticCurvePublicKey):
        key_info = {"type": "ECDSA", "curve": pub_key.curve.name, "size": pub_key.key_size}
    else:
        key_info = {"type": "Unknown"}

    return {
        "subject": subject_attrs,
        "issuer": issuer_attrs,
        "serial_number": hex(cert.serial_number),
        "not_valid_before": cert.not_valid_before_utc.isoformat(),
        "not_valid_after": cert.not_valid_after_utc.isoformat(),
        "days_remaining": days_remaining,
        "san_domains": san_names,
        "signature_algorithm": cert.signature_algorithm_oid._name,
        "public_key": key_info,
        "version": cert.version.value,
        "is_expired": days_remaining < 0,
        "fingerprint_sha256": cert.fingerprint(hashes.SHA256()).hex(),
    }


def check_remote_certificate(host: str, port: int = 443, timeout: int = 10) -> Dict:
    """Check the TLS certificate of a remote host."""
    result = {
        "host": host,
        "port": port,
        "status": "unknown",
        "days_remaining": None,
        "certificate": None,
        "errors": [],
    }

    try:
        ctx = ssl.create_default_context()
        with socket.create_connection((host, port), timeout=timeout) as sock:
            with ctx.wrap_socket(sock, server_hostname=host) as ssock:
                cert_der = ssock.getpeercert(binary_form=True)
                cert = x509.load_der_x509_certificate(cert_der, default_backend())

                now = datetime.datetime.utcnow()
                not_after = cert.not_valid_after_utc.replace(tzinfo=None)
                days_remaining = (not_after - now).days

                result["days_remaining"] = days_remaining
                result["not_after"] = not_after.isoformat()
                result["protocol"] = ssock.version()
                result["cipher"] = ssock.cipher()[0]

                subject_cn = None
                for attr in cert.subject:
                    if attr.oid == NameOID.COMMON_NAME:
                        subject_cn = attr.value
                        break

                result["common_name"] = subject_cn
                result["fingerprint_sha256"] = cert.fingerprint(hashes.SHA256()).hex()

                if days_remaining < 0:
                    result["status"] = "EXPIRED"
                elif days_remaining < EXPIRY_CRITICAL_DAYS:
                    result["status"] = "CRITICAL"
                elif days_remaining < EXPIRY_WARNING_DAYS:
                    result["status"] = "WARNING"
                else:
                    result["status"] = "OK"

    except ssl.SSLCertVerificationError as e:
        result["status"] = "INVALID"
        result["errors"].append(f"Certificate verification failed: {e}")
    except socket.timeout:
        result["status"] = "TIMEOUT"
        result["errors"].append("Connection timed out")
    except Exception as e:
        result["status"] = "ERROR"
        result["errors"].append(str(e))

    return result


def monitor_domains(domains: List[str], threshold_days: int = 30) -> Dict:
    """Monitor certificate expiration for multiple domains."""
    results = {
        "scan_time": datetime.datetime.utcnow().isoformat() + "Z",
        "threshold_days": threshold_days,
        "total_domains": len(domains),
        "ok": 0,
        "warning": 0,
        "critical": 0,
        "expired": 0,
        "errors": 0,
        "domains": [],
    }

    for domain in domains:
        domain = domain.strip()
        if not domain or domain.startswith("#"):
            continue

        host = domain.split(":")[0]
        port = int(domain.split(":")[1]) if ":" in domain else 443

        logger.info(f"Checking {host}:{port}...")
        check = check_remote_certificate(host, port)
        results["domains"].append(check)

        status = check["status"]
        if status == "OK":
            results["ok"] += 1
        elif status == "WARNING":
            results["warning"] += 1
        elif status == "CRITICAL":
            results["critical"] += 1
        elif status == "EXPIRED":
            results["expired"] += 1
        else:
            results["errors"] += 1

    return results


def verify_certificate_chain(cert_path: str, ca_bundle_path: str) -> Dict:
    """Verify a certificate chain against a CA bundle."""
    cert_data = Path(cert_path).read_bytes()
    ca_data = Path(ca_bundle_path).read_bytes()

    cert = x509.load_pem_x509_certificate(cert_data, default_backend())

    ca_certs = []
    pem_blocks = ca_data.split(b"-----END CERTIFICATE-----")
    for block in pem_blocks:
        block = block.strip()
        if block and b"-----BEGIN CERTIFICATE-----" in block:
            pem = block + b"\n-----END CERTIFICATE-----\n"
            ca_certs.append(x509.load_pem_x509_certificate(pem, default_backend()))

    chain = []
    current = cert
    chain.append({
        "subject": current.subject.rfc4514_string(),
        "issuer": current.issuer.rfc4514_string(),
    })

    for ca in ca_certs:
        if current.issuer == ca.subject:
            chain.append({
                "subject": ca.subject.rfc4514_string(),
                "issuer": ca.issuer.rfc4514_string(),
            })
            current = ca
            if ca.issuer == ca.subject:
                break

    is_self_signed = cert.issuer == cert.subject
    chain_complete = len(chain) > 1 or is_self_signed

    return {
        "certificate": cert.subject.rfc4514_string(),
        "chain_length": len(chain),
        "chain": chain,
        "chain_complete": chain_complete,
        "is_self_signed": is_self_signed,
    }


def main():
    parser = argparse.ArgumentParser(description="SSL Certificate Lifecycle Tool")
    subparsers = parser.add_subparsers(dest="command")

    csr = subparsers.add_parser("generate-csr", help="Generate CSR")
    csr.add_argument("--domain", required=True, help="Primary domain")
    csr.add_argument("--output", default="./certs", help="Output directory")
    csr.add_argument("--key-type", choices=["ecdsa", "rsa"], default="ecdsa")
    csr.add_argument("--san", nargs="*", help="Additional SAN domains")
    csr.add_argument("--org", help="Organization name")

    parse = subparsers.add_parser("parse-cert", help="Parse certificate")
    parse.add_argument("--cert", required=True, help="Certificate file path")

    check = subparsers.add_parser("check-expiry", help="Check remote cert expiry")
    check.add_argument("--host", required=True, help="Hostname")
    check.add_argument("--port", type=int, default=443, help="Port")

    mon = subparsers.add_parser("monitor", help="Monitor multiple domains")
    mon.add_argument("--domains", required=True, help="File with domain list")
    mon.add_argument("--threshold", type=int, default=30, help="Warning threshold (days)")

    chain = subparsers.add_parser("verify-chain", help="Verify certificate chain")
    chain.add_argument("--cert", required=True, help="Certificate file")
    chain.add_argument("--ca-bundle", required=True, help="CA bundle file")

    args = parser.parse_args()

    if args.command == "generate-csr":
        result = generate_csr(args.domain, args.output, args.key_type, args.san, args.org)
        print(json.dumps(result, indent=2))
    elif args.command == "parse-cert":
        result = parse_certificate(args.cert)
        print(json.dumps(result, indent=2, default=str))
    elif args.command == "check-expiry":
        result = check_remote_certificate(args.host, args.port)
        print(json.dumps(result, indent=2, default=str))
    elif args.command == "monitor":
        domains = Path(args.domains).read_text().strip().split("\n")
        result = monitor_domains(domains, args.threshold)
        print(json.dumps(result, indent=2, default=str))
    elif args.command == "verify-chain":
        result = verify_certificate_chain(args.cert, args.ca_bundle)
        print(json.dumps(result, indent=2))
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====
