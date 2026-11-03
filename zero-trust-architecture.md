

===== FILE: skills/configuring-aws-verified-access-for-ztna/SKILL.md =====
---
name: configuring-aws-verified-access-for-ztna
description: Configure AWS Verified Access to provide VPN-less zero trust network
  access to internal apps, combining identity trust providers (IAM Identity Center,
  Okta/OIDC), device posture providers (CrowdStrike, Jamf), Cedar policy authoring, and
  Terraform deployment. Use when replacing VPN access with ZTNA, writing Cedar access
  policies, or deploying Verified Access instances, groups, and endpoints across AWS
  accounts.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zero-trust
- aws
- verified-access
- ztna
- cedar-policy
- vpn-less
- identity-verification
- device-posture
- aws-ram
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078.004
- T1133
- T1021.007
---

# Configuring AWS Verified Access for ZTNA

## Overview

AWS Verified Access is a Zero Trust Network Access (ZTNA) service that provides secure, VPN-less access to corporate applications hosted in AWS. It evaluates each access request in real-time against granular conditional access policies written in the Cedar policy language, ensuring access is granted per-application only when specific security requirements such as user identity and device security posture are met and maintained. Verified Access integrates with AWS IAM Identity Center, third-party identity providers (Okta, CrowdStrike, JumpCloud, Jamf), and device management solutions. For multi-account deployments, AWS Resource Access Manager (RAM) enables sharing Verified Access groups across organizational units.


## When to Use

- When deploying or configuring configuring aws verified access for ztna capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- AWS account with appropriate IAM permissions
- Identity provider (AWS IAM Identity Center, Okta, or OIDC-compatible)
- Device trust provider (CrowdStrike, Jamf, JumpCloud, or AWS Verified Access native)
- Internal Application Load Balancer (ALB) or network interface endpoint
- Understanding of Cedar policy language
- VPC with application workloads to protect

## Architecture

```
    End User (Browser)
         |
         | HTTPS
         v
  +------+--------+
  | Verified      |
  | Access        |
  | Endpoint      |
  | (Public DNS)  |
  +------+--------+
         |
  +------+--------+
  | Verified      |  <-- Cedar Access Policies
  | Access        |  <-- Identity Provider Signals
  | Instance      |  <-- Device Trust Signals
  | (Policy       |
  |  Evaluation)  |
  +------+--------+
         |
  +------+--------+
  | Verified      |
  | Access Group  |
  | (App Group)   |
  +------+--------+
         |
  +------+--------+
  | Internal ALB  |
  | or ENI Target |
  +------+--------+
         |
  +------+--------+
  | Application   |
  | (Private VPC) |
  +--------------+
```

## Core Components

### Verified Access Instance

The regional entity that evaluates access requests against policies.

```bash
# Create Verified Access Instance via AWS CLI
aws ec2 create-verified-access-instance \
  --description "Production Zero Trust Instance" \
  --tag-specifications 'ResourceType=verified-access-instance,Tags=[{Key=Environment,Value=production}]'
```

### Trust Providers

#### Identity Trust Provider (AWS IAM Identity Center)

```bash
# Create identity trust provider
aws ec2 create-verified-access-trust-provider \
  --trust-provider-type user \
  --user-trust-provider-type iam-identity-center \
  --policy-reference-name "idc" \
  --description "IAM Identity Center trust provider" \
  --tag-specifications 'ResourceType=verified-access-trust-provider,Tags=[{Key=Type,Value=identity}]'
```

#### Identity Trust Provider (OIDC - Okta)

```bash
aws ec2 create-verified-access-trust-provider \
  --trust-provider-type user \
  --user-trust-provider-type oidc \
  --oidc-options '{
    "Issuer": "https://company.okta.com/oauth2/default",
    "AuthorizationEndpoint": "https://company.okta.com/oauth2/default/v1/authorize",
    "TokenEndpoint": "https://company.okta.com/oauth2/default/v1/token",
    "UserInfoEndpoint": "https://company.okta.com/oauth2/default/v1/userinfo",
    "ClientId": "0oa1234567890",
    "ClientSecret": "client-secret-here",
    "Scope": "openid profile groups"
  }' \
  --policy-reference-name "okta" \
  --description "Okta OIDC trust provider"
```

#### Device Trust Provider (CrowdStrike)

```bash
aws ec2 create-verified-access-trust-provider \
  --trust-provider-type device \
  --device-trust-provider-type crowdstrike \
  --device-options '{
    "TenantId": "crowdstrike-tenant-id",
    "PublicSigningKeyUrl": "https://api.crowdstrike.com/zero-trust/v2/certificates"
  }' \
  --policy-reference-name "crowdstrike" \
  --description "CrowdStrike device trust provider"
```

### Attach Trust Providers to Instance

```bash
# Attach identity provider
aws ec2 attach-verified-access-trust-provider \
  --verified-access-instance-id vai-0123456789abcdef \
  --verified-access-trust-provider-id vatp-0123456789abcdef

# Attach device provider
aws ec2 attach-verified-access-trust-provider \
  --verified-access-instance-id vai-0123456789abcdef \
  --verified-access-trust-provider-id vatp-device123456
```

### Verified Access Groups

```bash
# Create a group for web applications
aws ec2 create-verified-access-group \
  --verified-access-instance-id vai-0123456789abcdef \
  --description "Production Web Applications" \
  --policy-document 'permit(principal, action, resource)
    when {
      context.okta.groups.contains("production-access") &&
      context.crowdstrike.assessment.overall > 50
    };' \
  --tag-specifications 'ResourceType=verified-access-group,Tags=[{Key=Tier,Value=web}]'
```

### Verified Access Endpoints

```bash
# Create endpoint for ALB-backed application
aws ec2 create-verified-access-endpoint \
  --verified-access-group-id vag-0123456789abcdef \
  --endpoint-type load-balancer \
  --attachment-type vpc \
  --domain-certificate-arn arn:aws:acm:us-east-1:123456789012:certificate/xxxx \
  --application-domain app.internal.company.com \
  --endpoint-domain-prefix myapp \
  --load-balancer-options '{
    "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/internal-alb/xxxx",
    "Port": 443,
    "Protocol": "https",
    "SubnetIds": ["subnet-abc123", "subnet-def456"]
  }' \
  --security-group-ids sg-0123456789abcdef \
  --description "Internal HR Application"
```

## Cedar Policy Language

### Policy Basics

```cedar
// Allow access for users in the engineering group with compliant devices
permit(principal, action, resource)
when {
    context.okta.groups.contains("engineering") &&
    context.crowdstrike.assessment.overall > 70 &&
    context.crowdstrike.assessment.sensor_config.status == "active"
};

// Deny access from unmanaged devices
forbid(principal, action, resource)
when {
    !context.crowdstrike.assessment.sensor_config.status == "active"
};
```

### Advanced Policy Examples

```cedar
// Time-based access - only during business hours (UTC)
permit(principal, action, resource)
when {
    context.okta.groups.contains("contractors") &&
    context.http_request.http_method == "GET" &&
    context.crowdstrike.assessment.overall > 80
};

// Restrict admin access to specific user group with high device trust
permit(principal, action, resource)
when {
    context.idc.groups.contains("admins") &&
    context.crowdstrike.assessment.overall > 90 &&
    context.crowdstrike.assessment.os_version.startswith("Windows 11") ||
    context.crowdstrike.assessment.os_version.startswith("macOS 14")
};

// Allow read-only access for lower trust levels
permit(principal, action, resource)
when {
    context.okta.groups.contains("read-only") &&
    context.crowdstrike.assessment.overall > 30 &&
    context.http_request.http_method == "GET"
};
```

### Group-Level vs Endpoint-Level Policies

```cedar
// Group-level policy (applies to all endpoints in the group)
// Set on the Verified Access Group
permit(principal, action, resource)
when {
    context.okta.groups.contains("employees") &&
    context.crowdstrike.assessment.overall > 50
};

// Endpoint-level policy (additional restrictions for specific app)
// Set on the Verified Access Endpoint
permit(principal, action, resource)
when {
    context.okta.groups.contains("hr-team") &&
    context.okta.email.endsWith("@company.com")
};
```

## Terraform Configuration

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Verified Access Instance
resource "aws_verifiedaccess_instance" "main" {
  description = "Production Zero Trust Access"
  tags = {
    Environment = "production"
  }
}

# Identity Trust Provider (OIDC)
resource "aws_verifiedaccess_trust_provider" "okta" {
  policy_reference_name    = "okta"
  trust_provider_type      = "user"
  user_trust_provider_type = "oidc"
  description              = "Okta identity provider"

  oidc_options {
    authorization_endpoint = "https://company.okta.com/oauth2/default/v1/authorize"
    client_id              = var.okta_client_id
    client_secret          = var.okta_client_secret
    issuer                 = "https://company.okta.com/oauth2/default"
    scope                  = "openid profile groups"
    token_endpoint         = "https://company.okta.com/oauth2/default/v1/token"
    user_info_endpoint     = "https://company.okta.com/oauth2/default/v1/userinfo"
  }
}

# Device Trust Provider (CrowdStrike)
resource "aws_verifiedaccess_trust_provider" "crowdstrike" {
  policy_reference_name     = "crowdstrike"
  trust_provider_type       = "device"
  device_trust_provider_type = "crowdstrike"
  description               = "CrowdStrike device trust"

  device_options {
    tenant_id = var.crowdstrike_tenant_id
  }
}

# Attach providers to instance
resource "aws_verifiedaccess_instance_trust_provider_attachment" "okta" {
  verifiedaccess_instance_id       = aws_verifiedaccess_instance.main.id
  verifiedaccess_trust_provider_id = aws_verifiedaccess_trust_provider.okta.id
}

resource "aws_verifiedaccess_instance_trust_provider_attachment" "crowdstrike" {
  verifiedaccess_instance_id       = aws_verifiedaccess_instance.main.id
  verifiedaccess_trust_provider_id = aws_verifiedaccess_trust_provider.crowdstrike.id
}

# Verified Access Group
resource "aws_verifiedaccess_group" "web_apps" {
  verifiedaccess_instance_id = aws_verifiedaccess_instance.main.id
  description                = "Production Web Applications"

  policy_document = <<-CEDAR
    permit(principal, action, resource)
    when {
      context.okta.groups.contains("production-access") &&
      context.crowdstrike.assessment.overall > 50
    };
  CEDAR

  tags = {
    Tier = "web"
  }
}

# Verified Access Endpoint
resource "aws_verifiedaccess_endpoint" "internal_app" {
  verified_access_group_id = aws_verifiedaccess_group.web_apps.id
  endpoint_type            = "load-balancer"
  attachment_type          = "vpc"
  domain_certificate_arn   = aws_acm_certificate.app.arn
  application_domain       = "app.internal.company.com"
  endpoint_domain_prefix   = "myapp"
  description              = "Internal Application"

  load_balancer_options {
    load_balancer_arn = aws_lb.internal.arn
    port              = 443
    protocol          = "https"
    subnet_ids        = var.private_subnet_ids
  }

  security_group_ids = [aws_security_group.verified_access.id]

  policy_document = <<-CEDAR
    permit(principal, action, resource)
    when {
      context.okta.groups.contains("app-users")
    };
  CEDAR
}

# Logging configuration
resource "aws_verifiedaccess_instance_logging_configuration" "main" {
  verifiedaccess_instance_id = aws_verifiedaccess_instance.main.id

  access_logs {
    cloudwatch_logs {
      enabled   = true
      log_group = aws_cloudwatch_log_group.verified_access.name
    }
    s3 {
      enabled     = true
      bucket_name = aws_s3_bucket.access_logs.id
      prefix      = "verified-access/"
    }
  }
}

resource "aws_cloudwatch_log_group" "verified_access" {
  name              = "/aws/verified-access/production"
  retention_in_days = 90
}
```

## Multi-Account Deployment with AWS RAM

```hcl
# Share Verified Access Group across accounts via RAM
resource "aws_ram_resource_share" "verified_access" {
  name                      = "verified-access-share"
  allow_external_principals = false
}

resource "aws_ram_resource_association" "group_share" {
  resource_arn       = aws_verifiedaccess_group.web_apps.verified_access_group_arn
  resource_share_arn = aws_ram_resource_share.verified_access.arn
}

resource "aws_ram_principal_association" "workload_ou" {
  principal          = "arn:aws:organizations::123456789012:ou/o-xxxx/ou-xxxx-xxxxxxxx"
  resource_share_arn = aws_ram_resource_share.verified_access.arn
}
```

## Monitoring and Logging

```bash
# Query access logs in CloudWatch
aws logs filter-log-events \
  --log-group-name /aws/verified-access/production \
  --filter-pattern '{ $.status_code = "403" }' \
  --start-time $(date -d '1 hour ago' +%s000)

# CloudWatch alarm for access denials
aws cloudwatch put-metric-alarm \
  --alarm-name "VerifiedAccess-HighDenialRate" \
  --metric-name "AccessDenied" \
  --namespace "AWS/VerifiedAccess" \
  --statistic Sum \
  --period 300 \
  --threshold 100 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:security-alerts
```

## Security Best Practices

1. **Layer policies**: Use group-level policies for broad controls and endpoint-level for app-specific restrictions
2. **Require device trust**: Always include device posture checks in Cedar policies
3. **Enable access logging**: Send to both CloudWatch and S3 for real-time monitoring and long-term retention
4. **Use RAM for multi-account**: Share groups across OUs instead of duplicating configuration
5. **Rotate OIDC secrets**: Automate client secret rotation via Secrets Manager
6. **Test policies in non-production**: Validate Cedar policies before production deployment
7. **Set high device trust thresholds**: Require overall score above 70 for production access
8. **Monitor for policy drift**: Use AWS Config rules to detect unauthorized changes

## References

- [AWS Verified Access Documentation](https://docs.aws.amazon.com/verified-access/)
- [Cedar Policy Language](https://www.cedarpolicy.com/)
- [Building Zero Trust Across Multi-Account AWS Environments](https://aws.amazon.com/blogs/networking-and-content-delivery/building-zero-trust-access-across-multi-account-aws-environments/)
- [Visual Guide to AWS Verified Access Setup](https://medium.com/@chaim_sanders/a-visual-guide-to-setting-up-aws-verified-access-1333466f7222)
- [NIST SP 800-207 Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final)

===== END FILE =====


===== FILE: skills/configuring-aws-verified-access-for-ztna/LICENSE =====

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


===== FILE: skills/configuring-aws-verified-access-for-ztna/assets/template.md =====
# AWS Verified Access Deployment Template

## Deployment Configuration
- **AWS Region**: _______________
- **Account Type**: [ ] Single-account [ ] Multi-account (RAM)
- **Identity Provider**: [ ] IAM Identity Center [ ] Okta [ ] Other OIDC
- **Device Trust Provider**: [ ] CrowdStrike [ ] Jamf [ ] JumpCloud

## Trust Providers

| Provider | Type | Reference Name | Status |
|---|---|---|---|
| ___ | user (identity) | ___ | [ ] Configured |
| ___ | device | ___ | [ ] Configured |

## Access Groups

| Group Name | Policy Summary | Endpoint Count | Min Device Score |
|---|---|---|---|
| ___ | ___ | ___ | ___ |

## Application Endpoints

| Application | Domain | Port | Group | Policy Level |
|---|---|---|---|---|
| ___ | ___ | 443 | ___ | group/endpoint |

## Cedar Policy Checklist
- [ ] Group-level policies defined for all groups
- [ ] Endpoint-level policies added for sensitive applications
- [ ] Device trust score thresholds set appropriately
- [ ] Forbid policies block unmanaged devices
- [ ] Admin access requires high device trust (>90)
- [ ] Policies tested in non-production first

## Monitoring Setup
- [ ] Access logs sent to CloudWatch Logs
- [ ] Access logs archived to S3
- [ ] CloudWatch alarms for high denial rates
- [ ] Dashboard created for access metrics

===== END FILE =====


===== FILE: skills/configuring-aws-verified-access-for-ztna/references/api-reference.md =====
# AWS Verified Access ZTNA — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| boto3 | `pip install boto3` | AWS SDK — EC2 Verified Access API |

## Key boto3 EC2 Methods

| Method | Description |
|--------|-------------|
| `describe_verified_access_instances()` | List VA instances |
| `describe_verified_access_groups()` | List VA groups with policies |
| `describe_verified_access_endpoints()` | List VA application endpoints |
| `describe_verified_access_trust_providers()` | List identity/device trust providers |
| `create_verified_access_instance()` | Create new VA instance |
| `create_verified_access_group(VerifiedAccessInstanceId=)` | Create group under instance |
| `create_verified_access_endpoint()` | Expose application via VA |
| `modify_verified_access_group_policy()` | Update Cedar policy document |

## Cedar Policy Language

```cedar
permit(principal, action, resource)
when {
    context.identity.groups has "engineering" &&
    context.identity.email.address like "*@company.com" &&
    context.device.status == "compliant"
};
```

## Trust Provider Types

| Type | Description |
|------|-------------|
| user / iam-identity-center | AWS IAM Identity Center OIDC |
| user / oidc | Third-party OIDC provider |
| device / jamf | Jamf device trust |
| device / crowdstrike | CrowdStrike device posture |

## External References

- [AWS Verified Access Docs](https://docs.aws.amazon.com/verified-access/)
- [Cedar Policy Language](https://www.cedarpolicy.com/en)
- [boto3 EC2 Verified Access](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2/client/describe_verified_access_instances.html)

===== END FILE =====


===== FILE: skills/configuring-aws-verified-access-for-ztna/references/standards.md =====
# Standards Reference: AWS Verified Access ZTNA

## AWS Verified Access Standards

### Cedar Policy Language
- Developed by AWS for fine-grained access control
- Supports permit and forbid policies with conditions
- Evaluates context attributes (identity, device posture, request metadata)
- Open-source specification at cedarpolicy.com

### AWS Well-Architected Framework - Security Pillar
- SEC03-BP01: Define access requirements
- SEC03-BP02: Grant least privilege access
- SEC03-BP05: Define permission guardrails for organization
- SEC05-BP03: Automate network protection

## Zero Trust Standards

### NIST SP 800-207
- Verified Access implements enhanced identity governance (Section 3.1)
- Per-request policy evaluation aligns with continuous verification
- Device posture assessment satisfies device trust requirements
- Integration with identity providers meets identity pillar requirements

### CISA Zero Trust Maturity Model
- Identity Pillar: OIDC integration with MFA-enabled providers
- Device Pillar: CrowdStrike/Jamf device posture assessment
- Network Pillar: VPN-less access eliminates implicit network trust
- Application Pillar: Per-application access policies

## Compliance Mappings

### SOC 2
- CC6.1: Logical access security through Cedar policies
- CC6.2: Identity verification via trust providers
- CC6.3: Role-based access through group policies
- CC7.2: System monitoring through access logging

### FedRAMP
- AC-2: Account Management via identity provider integration
- AC-3: Access Enforcement through Cedar policies
- AC-6: Least Privilege via per-application policies
- AU-2: Auditable Events through CloudWatch logging
- SC-7: Boundary Protection via Verified Access endpoints

===== END FILE =====


===== FILE: skills/configuring-aws-verified-access-for-ztna/references/workflows.md =====
# Workflows: AWS Verified Access ZTNA Configuration

## Workflow 1: Initial Setup

```
Step 1: Create Verified Access Instance
  - Deploy instance in target AWS region
  - Tag with environment and ownership information

Step 2: Configure Trust Providers
  - Set up identity trust provider (IAM Identity Center or OIDC)
  - Set up device trust provider (CrowdStrike, Jamf, JumpCloud)
  - Attach both providers to the Verified Access instance

Step 3: Create Access Groups
  - Define groups based on application tiers or teams
  - Write group-level Cedar policies
  - Set baseline identity and device requirements

Step 4: Create Endpoints
  - Map each application to a Verified Access endpoint
  - Configure ALB or ENI attachment
  - Set application domain and certificate
  - Write endpoint-specific Cedar policies

Step 5: Configure DNS and Certificates
  - Create ACM certificate for application domain
  - Configure Route 53 CNAME to endpoint domain
  - Validate certificate and DNS resolution
```

## Workflow 2: Cedar Policy Development

```
Step 1: Define Access Requirements
  - Map user groups to application access needs
  - Define device compliance requirements
  - Identify time-based or context-based restrictions

Step 2: Write and Test Policies
  - Start with permit policies for known good access
  - Add forbid policies for explicit denials
  - Test with various identity and device contexts
  - Validate in non-production environment

Step 3: Deploy Policies
  - Apply group-level policies first
  - Add endpoint-specific policies for sensitive apps
  - Monitor access logs for false denials
  - Iterate based on user feedback

Step 4: Ongoing Maintenance
  - Review policies quarterly
  - Update device trust thresholds
  - Add new groups as teams change
  - Remove deprecated application endpoints
```

## Workflow 3: Multi-Account Deployment

```
Step 1: Design Architecture
  - Deploy Verified Access in dedicated networking account
  - Plan group sharing via AWS RAM
  - Define organizational unit boundaries

Step 2: Share Resources
  - Create RAM resource shares
  - Associate Verified Access groups
  - Share with target OUs or accounts

Step 3: Create Endpoints in Workload Accounts
  - Accept RAM shares in workload accounts
  - Create endpoints using shared groups
  - Configure application-specific settings

Step 4: Centralized Monitoring
  - Aggregate access logs to central S3 bucket
  - Configure cross-account CloudWatch dashboards
  - Set up centralized alerting for policy violations
```

===== END FILE =====


===== FILE: skills/configuring-aws-verified-access-for-ztna/scripts/agent.py =====
#!/usr/bin/env python3
"""AWS Verified Access ZTNA configuration agent using boto3."""

import json
import sys
import argparse
from datetime import datetime

try:
    import boto3
except ImportError:
    print("Install: pip install boto3")
    sys.exit(1)


def list_verified_access_instances(session):
    """List all Verified Access instances."""
    ec2 = session.client("ec2")
    response = ec2.describe_verified_access_instances()
    instances = []
    for inst in response.get("VerifiedAccessInstances", []):
        instances.append({
            "id": inst["VerifiedAccessInstanceId"],
            "description": inst.get("Description", ""),
            "creation_time": str(inst.get("CreationTime", "")),
            "trust_providers": [tp["VerifiedAccessTrustProviderId"]
                                for tp in inst.get("VerifiedAccessTrustProviders", [])],
        })
    return instances


def list_verified_access_groups(session):
    """List Verified Access groups and their policies."""
    ec2 = session.client("ec2")
    response = ec2.describe_verified_access_groups()
    groups = []
    for grp in response.get("VerifiedAccessGroups", []):
        groups.append({
            "id": grp["VerifiedAccessGroupId"],
            "instance_id": grp.get("VerifiedAccessInstanceId", ""),
            "description": grp.get("Description", ""),
            "policy_enabled": grp.get("PolicyEnabled", False),
            "policy_document": grp.get("PolicyDocument", ""),
        })
    return groups


def list_verified_access_endpoints(session):
    """List Verified Access endpoints."""
    ec2 = session.client("ec2")
    response = ec2.describe_verified_access_endpoints()
    endpoints = []
    for ep in response.get("VerifiedAccessEndpoints", []):
        endpoints.append({
            "id": ep["VerifiedAccessEndpointId"],
            "group_id": ep.get("VerifiedAccessGroupId", ""),
            "type": ep.get("EndpointType", ""),
            "domain": ep.get("DomainCertificateArn", ""),
            "status": ep.get("Status", {}).get("Code", ""),
            "application_domain": ep.get("ApplicationDomain", ""),
        })
    return endpoints


def audit_trust_providers(session):
    """Audit Verified Access trust providers."""
    ec2 = session.client("ec2")
    response = ec2.describe_verified_access_trust_providers()
    providers = []
    for tp in response.get("VerifiedAccessTrustProviders", []):
        providers.append({
            "id": tp["VerifiedAccessTrustProviderId"],
            "type": tp.get("TrustProviderType", ""),
            "user_trust_type": tp.get("UserTrustProviderType", ""),
            "device_trust_type": tp.get("DeviceTrustProviderType", ""),
            "policy_reference": tp.get("PolicyReferenceName", ""),
        })
    return providers


def run_audit(profile=None, region="us-east-1"):
    """Execute AWS Verified Access audit."""
    session = boto3.Session(profile_name=profile, region_name=region)
    print(f"\n{'='*60}")
    print(f"  AWS VERIFIED ACCESS ZTNA AUDIT")
    print(f"  Region: {region}")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    instances = list_verified_access_instances(session)
    print(f"--- INSTANCES ({len(instances)}) ---")
    for i in instances:
        print(f"  {i['id']}: {i['description']} (providers: {len(i['trust_providers'])})")

    providers = audit_trust_providers(session)
    print(f"\n--- TRUST PROVIDERS ({len(providers)}) ---")
    for p in providers:
        print(f"  {p['id']}: type={p['type']} user={p['user_trust_type']} device={p['device_trust_type']}")

    groups = list_verified_access_groups(session)
    print(f"\n--- GROUPS ({len(groups)}) ---")
    for g in groups:
        status = "ENABLED" if g["policy_enabled"] else "DISABLED"
        print(f"  {g['id']}: policy={status}")

    endpoints = list_verified_access_endpoints(session)
    print(f"\n--- ENDPOINTS ({len(endpoints)}) ---")
    for e in endpoints:
        print(f"  {e['id']}: {e['application_domain']} ({e['status']})")

    return {"instances": instances, "providers": providers, "groups": groups, "endpoints": endpoints}


def main():
    parser = argparse.ArgumentParser(description="AWS Verified Access ZTNA Agent")
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


===== FILE: skills/configuring-aws-verified-access-for-ztna/scripts/process.py =====
#!/usr/bin/env python3
"""
AWS Verified Access ZTNA Configuration and Policy Management.

Generates Cedar access policies, validates configurations,
and monitors Verified Access deployments.
"""

import json
import datetime
from dataclasses import dataclass, field
from pathlib import Path


@dataclass
class TrustProvider:
    name: str
    provider_type: str  # "user" or "device"
    reference_name: str
    config: dict = field(default_factory=dict)


@dataclass
class AccessGroup:
    name: str
    description: str
    policy: str
    endpoints: list = field(default_factory=list)


@dataclass
class AccessEndpoint:
    name: str
    application_domain: str
    endpoint_type: str  # "load-balancer" or "network-interface"
    port: int = 443
    policy: str = ""
    alb_arn: str = ""


class CedarPolicyGenerator:
    """Generate Cedar access policies for AWS Verified Access."""

    def __init__(self, identity_ref: str = "okta", device_ref: str = "crowdstrike"):
        self.identity_ref = identity_ref
        self.device_ref = device_ref

    def permit_group_with_device_trust(self, group: str, min_score: int = 50) -> str:
        return f'''permit(principal, action, resource)
when {{
    context.{self.identity_ref}.groups.contains("{group}") &&
    context.{self.device_ref}.assessment.overall > {min_score}
}};'''

    def permit_group_read_only(self, group: str, min_score: int = 30) -> str:
        return f'''permit(principal, action, resource)
when {{
    context.{self.identity_ref}.groups.contains("{group}") &&
    context.{self.device_ref}.assessment.overall > {min_score} &&
    context.http_request.http_method == "GET"
}};'''

    def forbid_unmanaged_devices(self) -> str:
        return f'''forbid(principal, action, resource)
when {{
    !context.{self.device_ref}.assessment.sensor_config.status == "active"
}};'''

    def permit_admin_high_trust(self, admin_group: str, min_score: int = 90) -> str:
        return f'''permit(principal, action, resource)
when {{
    context.{self.identity_ref}.groups.contains("{admin_group}") &&
    context.{self.device_ref}.assessment.overall > {min_score} &&
    context.{self.identity_ref}.email.endsWith("@company.com")
}};'''

    def combine_policies(self, policies: list) -> str:
        return "\n\n".join(policies)


class VerifiedAccessConfigGenerator:
    """Generate Terraform configuration for AWS Verified Access."""

    def __init__(self):
        self.trust_providers: list[TrustProvider] = []
        self.groups: list[AccessGroup] = []
        self.endpoints: list[AccessEndpoint] = []

    def add_trust_provider(self, name: str, provider_type: str,
                           reference_name: str, config: dict = None):
        self.trust_providers.append(TrustProvider(
            name=name, provider_type=provider_type,
            reference_name=reference_name, config=config or {}
        ))

    def add_group(self, name: str, description: str, policy: str):
        self.groups.append(AccessGroup(name=name, description=description, policy=policy))

    def add_endpoint(self, group_name: str, name: str, domain: str,
                     port: int = 443, policy: str = "", alb_arn: str = ""):
        endpoint = AccessEndpoint(
            name=name, application_domain=domain,
            endpoint_type="load-balancer", port=port,
            policy=policy, alb_arn=alb_arn
        )
        for group in self.groups:
            if group.name == group_name:
                group.endpoints.append(endpoint)
        self.endpoints.append(endpoint)

    def generate_terraform(self) -> str:
        sections = [self._provider_block()]

        sections.append('\n# Verified Access Instance')
        sections.append('''resource "aws_verifiedaccess_instance" "main" {
  description = "Production Zero Trust Access"
  tags = {
    Environment = "production"
    ManagedBy   = "terraform"
  }
}''')

        for tp in self.trust_providers:
            sections.append(self._trust_provider_block(tp))

        for group in self.groups:
            sections.append(self._group_block(group))

        for endpoint in self.endpoints:
            sections.append(self._endpoint_block(endpoint))

        return "\n\n".join(sections)

    def _provider_block(self) -> str:
        return '''terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}'''

    def _trust_provider_block(self, tp: TrustProvider) -> str:
        resource_name = tp.name.replace("-", "_")
        return f'''resource "aws_verifiedaccess_trust_provider" "{resource_name}" {{
  policy_reference_name = "{tp.reference_name}"
  trust_provider_type   = "{tp.provider_type}"
  description           = "{tp.name} trust provider"
}}

resource "aws_verifiedaccess_instance_trust_provider_attachment" "{resource_name}" {{
  verifiedaccess_instance_id       = aws_verifiedaccess_instance.main.id
  verifiedaccess_trust_provider_id = aws_verifiedaccess_trust_provider.{resource_name}.id
}}'''

    def _group_block(self, group: AccessGroup) -> str:
        resource_name = group.name.replace("-", "_")
        return f'''resource "aws_verifiedaccess_group" "{resource_name}" {{
  verifiedaccess_instance_id = aws_verifiedaccess_instance.main.id
  description                = "{group.description}"

  policy_document = <<-CEDAR
    {group.policy}
  CEDAR
}}'''

    def _endpoint_block(self, endpoint: AccessEndpoint) -> str:
        resource_name = endpoint.name.replace("-", "_").replace(".", "_")
        policy_block = ""
        if endpoint.policy:
            policy_block = f'''

  policy_document = <<-CEDAR
    {endpoint.policy}
  CEDAR'''

        return f'''resource "aws_verifiedaccess_endpoint" "{resource_name}" {{
  verified_access_group_id = aws_verifiedaccess_group.web_apps.id
  endpoint_type            = "load-balancer"
  attachment_type          = "vpc"
  application_domain       = "{endpoint.application_domain}"
  description              = "{endpoint.name}"{policy_block}
}}'''

    def export_terraform(self, output_path: str):
        config = self.generate_terraform()
        path = Path(output_path)
        path.parent.mkdir(parents=True, exist_ok=True)
        with open(path, "w") as f:
            f.write(config)
        return config


def main():
    """Generate example AWS Verified Access configuration."""
    # Generate Cedar policies
    cedar = CedarPolicyGenerator(identity_ref="okta", device_ref="crowdstrike")

    group_policy = cedar.combine_policies([
        cedar.permit_group_with_device_trust("production-access", 50),
        cedar.forbid_unmanaged_devices(),
    ])

    admin_policy = cedar.permit_admin_high_trust("platform-admins", 90)
    readonly_policy = cedar.permit_group_read_only("read-only-users", 30)

    print("Generated Cedar Policies:")
    print("=" * 60)
    print("\nGroup Policy (production access):")
    print(group_policy)
    print(f"\nAdmin Policy:")
    print(admin_policy)
    print(f"\nRead-Only Policy:")
    print(readonly_policy)

    # Generate Terraform config
    gen = VerifiedAccessConfigGenerator()
    gen.add_trust_provider("okta-identity", "user", "okta")
    gen.add_trust_provider("crowdstrike-device", "device", "crowdstrike")
    gen.add_group("web-apps", "Production Web Applications", group_policy)
    gen.add_endpoint("web-apps", "internal-app", "app.internal.company.com", 443)
    gen.add_endpoint("web-apps", "admin-portal", "admin.internal.company.com", 443,
                     policy=admin_policy)

    config = gen.export_terraform("verified_access.tf")
    print("\n" + "=" * 60)
    print("Generated Terraform Configuration:")
    print("=" * 60)
    print(config[:3000])


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-identity-aware-proxy-with-google-iap/SKILL.md =====
---
name: configuring-identity-aware-proxy-with-google-iap
description: 'Configures Google Cloud Identity-Aware Proxy (IAP) via gcloud to enforce
  per-request identity verification on Compute Engine, App Engine, Cloud Run, and
  GKE, including IAM bindings, Access Context Manager access levels, session/reauth
  settings, and service-account programmatic access. Use when replacing VPN access
  with identity-based access to GCP backends or configuring context-aware, zero-trust
  policies for Google Cloud services.

  '
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- google-iap
- identity-aware-proxy
- gcp
- zero-trust
- access-context-manager
- cloud-run
- app-engine
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078.004
- T1133
- T1021.007
mitre_f3:
  version: '1.1'
  tactics:
  - initial-access
  - positioning
  techniques:
  - id: F1006
    name: Account Takeover
    tactic: initial-access
    source: f3
  - id: F1004
    name: Access with Stolen Session Cookie
    tactic: initial-access
    source: f3
  - id: T1550.001
    name: 'Use Alternate Authentication Material: Application Access Token'
    tactic: initial-access
    source: attack
  - id: T1539
    name: Steal Web Session Cookie
    tactic: positioning
    source: attack
---

# Configuring Identity-Aware Proxy with Google IAP

## When to Use

- When protecting Google Cloud applications (App Engine, Cloud Run, GKE, Compute Engine) with identity-based access
- When implementing context-aware access requiring device posture and location verification
- When providing secure access to internal tools without VPN or public IP exposure
- When needing per-request authentication and authorization for web applications and TCP services
- When configuring programmatic access to IAP-protected resources using service accounts

**Do not use** for non-HTTP applications that cannot be placed behind an HTTPS load balancer, for public-facing applications that need unauthenticated access, or when applications handle their own authentication and IAP would conflict with existing auth flows.

## Prerequisites

- Google Cloud project with billing enabled
- IAP API enabled (`gcloud services enable iap.googleapis.com`)
- Application deployed behind HTTPS Load Balancer, App Engine, or Cloud Run
- Cloud Identity or Google Workspace for user management
- Access Context Manager API enabled for access levels
- OAuth consent screen configured for the project

## Workflow

### Step 1: Enable IAP on Backend Services

Configure IAP for different GCP compute platforms.

```bash
# Enable required APIs
gcloud services enable iap.googleapis.com
gcloud services enable accesscontextmanager.googleapis.com

# Create OAuth consent screen
gcloud iap oauth-brands create \
  --application_title="Internal Applications" \
  --support_email=security@company.com

# Create OAuth client
gcloud iap oauth-clients create \
  projects/PROJECT_ID/brands/BRAND_ID \
  --display_name="IAP Web Client"

# === Enable IAP on Compute Engine Backend Service ===
gcloud compute backend-services update my-backend-service \
  --iap=enabled,oauth2-client-id=CLIENT_ID,oauth2-client-secret=CLIENT_SECRET \
  --global

# === Enable IAP on App Engine ===
gcloud iap web enable \
  --resource-type=app-engine \
  --oauth2-client-id=CLIENT_ID \
  --oauth2-client-secret=CLIENT_SECRET

# === Enable IAP on Cloud Run ===
# First grant IAP service account the Cloud Run Invoker role
gcloud run services add-iam-policy-binding my-service \
  --member="serviceAccount:service-PROJECT_NUM@gcp-sa-iap.iam.gserviceaccount.com" \
  --role="roles/run.invoker" \
  --region=us-central1

# Enable IAP on the Cloud Run backend service
gcloud compute backend-services update my-cloud-run-backend \
  --iap=enabled,oauth2-client-id=CLIENT_ID,oauth2-client-secret=CLIENT_SECRET \
  --global

# === Enable IAP TCP Forwarding for SSH/RDP ===
# No load balancer needed - uses IAP tunnel
gcloud compute instances add-iam-policy-binding my-vm \
  --member="group:developers@company.com" \
  --role="roles/iap.tunnelResourceAccessor" \
  --zone=us-central1-a

# SSH through IAP tunnel
gcloud compute ssh my-vm --zone=us-central1-a --tunnel-through-iap

# RDP through IAP tunnel
gcloud compute start-iap-tunnel my-windows-vm 3389 \
  --local-host-port=localhost:3390 \
  --zone=us-central1-a
```

### Step 2: Configure IAM Bindings for Access Control

Grant access to specific users and groups with optional access level conditions.

```bash
# Grant basic access to a group
gcloud iap web add-iam-policy-binding \
  --resource-type=backend-services \
  --service=my-backend-service \
  --member="group:engineering@company.com" \
  --role="roles/iap.httpsResourceAccessor"

# Grant access with access level condition
gcloud iap web add-iam-policy-binding \
  --resource-type=backend-services \
  --service=finance-app \
  --member="group:finance@company.com" \
  --role="roles/iap.httpsResourceAccessor" \
  --condition='expression=request.auth.access_levels.exists(x, x == "accessPolicies/POLICY_ID/accessLevels/corporate-device"),title=RequireCorporateDevice,description=Requires managed corporate device'

# Grant access only during business hours
gcloud iap web add-iam-policy-binding \
  --resource-type=backend-services \
  --service=admin-console \
  --member="group:admins@company.com" \
  --role="roles/iap.httpsResourceAccessor" \
  --condition='expression=request.time.getHours("America/New_York") >= 8 && request.time.getHours("America/New_York") <= 18 && request.time.getDayOfWeek("America/New_York") >= 1 && request.time.getDayOfWeek("America/New_York") <= 5,title=BusinessHoursOnly'

# Grant access to a specific URL path
gcloud iap web add-iam-policy-binding \
  --resource-type=backend-services \
  --service=internal-api \
  --member="group:api-consumers@company.com" \
  --role="roles/iap.httpsResourceAccessor" \
  --condition='expression=request.path.startsWith("/api/v2/"),title=APIv2Access'
```

### Step 3: Create Access Levels with Access Context Manager

Define context-based access requirements using device attributes and network conditions.

```bash
# Create access level requiring encrypted corporate device
cat > managed-device.yaml << 'EOF'
- devicePolicy:
    allowedEncryptionStatuses:
      - ENCRYPTED
    osConstraints:
      - osType: DESKTOP_WINDOWS
        minimumVersion: "10.0.19045"
      - osType: DESKTOP_MAC
        minimumVersion: "14.0"
      - osType: DESKTOP_CHROME_OS
    requireScreenlock: true
    requireAdminApproval: true
    allowedDeviceManagementLevels:
      - ADVANCED
EOF

gcloud access-context-manager levels create managed-device \
  --policy=POLICY_ID \
  --title="Managed Device" \
  --basic-level-spec=managed-device.yaml

# Create access level for corporate network
cat > corp-network.yaml << 'EOF'
- ipSubnetworks:
    - "203.0.113.0/24"
    - "198.51.100.0/24"
  regions:
    - US
    - GB
EOF

gcloud access-context-manager levels create corp-network \
  --policy=POLICY_ID \
  --title="Corporate Network" \
  --basic-level-spec=corp-network.yaml

# Create custom access level using CEL for complex logic
cat > high-trust.yaml << 'EOF'
expression: >
  device.encryption_status == DeviceEncryptionStatus.ENCRYPTED &&
  device.is_admin_approved_device == true &&
  (
    origin.ip in ["203.0.113.0/24"] ||
    device.os_type == OsType.DESKTOP_CHROME_OS
  ) &&
  request.auth.claims.hd == "company.com"
EOF

gcloud access-context-manager levels create high-trust \
  --policy=POLICY_ID \
  --title="High Trust" \
  --custom-level-spec=high-trust.yaml
```

### Step 4: Configure Session Settings and Re-authentication

Set session duration and re-authentication policies per application.

```bash
# Configure re-authentication for a backend service
# Requires login every 4 hours for sensitive apps
gcloud iap settings set \
  --project=PROJECT_ID \
  --resource-type=compute \
  --service=finance-app \
  reauthSettings.method=LOGIN \
  reauthSettings.maxAge=14400s \
  reauthSettings.policyType=MINIMUM

# Configure session settings for App Engine
gcloud iap settings set \
  --project=PROJECT_ID \
  --resource-type=app-engine \
  reauthSettings.method=SECURE_KEY \
  reauthSettings.maxAge=3600s \
  reauthSettings.policyType=MINIMUM

# View current IAP settings
gcloud iap settings get \
  --project=PROJECT_ID \
  --resource-type=compute \
  --service=finance-app
```

### Step 5: Configure Programmatic Access for Service Accounts

Enable service-to-service communication through IAP-protected endpoints.

```python
#!/usr/bin/env python3
"""Access IAP-protected resource using service account credentials."""

import google.auth
import google.auth.transport.requests
from google.auth import impersonated_credentials
import requests as req

IAP_CLIENT_ID = "YOUR_IAP_OAUTH_CLIENT_ID.apps.googleusercontent.com"
IAP_URL = "https://my-app.company.com/api/data"

def access_iap_resource():
    # Get default credentials (works with service account key or workload identity)
    credentials, project = google.auth.default()

    # Create IAP-authenticated request
    authed_session = google.auth.transport.requests.AuthorizedSession(
        credentials,
        target_audience=IAP_CLIENT_ID
    )

    # Make request to IAP-protected resource
    response = authed_session.get(IAP_URL)
    print(f"Status: {response.status_code}")
    print(f"Response: {response.text[:500]}")

    return response

if __name__ == "__main__":
    access_iap_resource()
```

### Step 6: Set Up Audit Logging and Monitoring

Configure logging for all IAP access decisions.

```bash
# Enable data access audit logs for IAP
gcloud projects get-iam-policy PROJECT_ID --format=json > policy.json

# Add IAP audit config to policy.json:
# {
#   "service": "iap.googleapis.com",
#   "auditLogConfigs": [
#     {"logType": "ADMIN_READ"},
#     {"logType": "DATA_READ"},
#     {"logType": "DATA_WRITE"}
#   ]
# }

gcloud projects set-iam-policy PROJECT_ID policy.json

# Create log-based metric for denied access
gcloud logging metrics create iap-denied-access \
  --description="Count of IAP access denials" \
  --log-filter='resource.type="gce_backend_service" AND protoPayload.status.code=16'

# Create alerting policy for high denial rates
gcloud alpha monitoring policies create \
  --display-name="IAP High Denial Rate" \
  --condition-display-name="Denied access > 50 in 5 min" \
  --condition-filter='metric.type="logging.googleapis.com/user/iap-denied-access"' \
  --condition-threshold-value=50 \
  --condition-threshold-duration=300s \
  --notification-channels=projects/PROJECT_ID/notificationChannels/CHANNEL_ID

# Query IAP access logs
gcloud logging read '
  resource.type="gce_backend_service"
  protoPayload.serviceName="iap.googleapis.com"
  timestamp >= "2026-02-22T00:00:00Z"
' --project=PROJECT_ID --format='table(timestamp,protoPayload.authenticationInfo.principalEmail,protoPayload.status.code,resource.labels.backend_service_name)' --limit=50
```

## Key Concepts

| Term | Definition |
|------|------------|
| Identity-Aware Proxy | GCP service that intercepts web requests and TCP connections, authenticating users and evaluating access policies before proxying to backend services |
| Backend Service | GCP load balancer component that IAP protects; can serve Compute Engine instances, GKE pods, Cloud Run services, or App Engine |
| IAP Tunnel | Secure TCP tunnel through IAP allowing SSH, RDP, and other TCP access to VMs without public IPs or VPN |
| OAuth Consent Screen | GCP configuration specifying the application name and support email shown to users during IAP authentication |
| Access Level | Named condition in Access Context Manager evaluated during IAP authorization (device posture, IP, geography) |
| Re-authentication | IAP feature requiring users to prove their identity again after a configurable session duration |

## Tools & Systems

- **Google Cloud IAP**: Identity-aware reverse proxy for GCP applications and TCP services
- **Access Context Manager**: Defines access levels based on device, network, and geographic attributes
- **gcloud CLI**: Command-line tool for configuring IAP, access levels, and IAM bindings
- **IAP TCP Forwarding**: Tunnel-based access to VMs for SSH/RDP without public IPs
- **Cloud Audit Logs**: Immutable records of all IAP access decisions for compliance
- **Endpoint Verification**: Chrome extension collecting device attributes for access level evaluation

## Common Scenarios

### Scenario: Securing 15 Internal GCP Services with IAP

**Context**: An e-commerce company runs 15 internal services on GKE and Cloud Run (admin dashboards, internal APIs, monitoring tools). Currently, these services are protected only by VPN and firewall rules, creating excessive network-level access.

**Approach**:
1. Deploy all services behind an HTTPS Load Balancer with managed SSL certificates
2. Enable IAP on each backend service with per-service OAuth clients
3. Create IAM bindings mapping Google Groups to specific services (admin group -> admin dashboard, engineering -> monitoring)
4. Define access levels: managed-device (encryption + screen lock), corp-network (office IP ranges)
5. Apply managed-device access level to admin dashboard and financial tools
6. Configure IAP TCP tunneling for SSH access to GKE nodes (replacing SSH bastion host)
7. Set re-authentication to 4 hours for admin tools, 8 hours for monitoring
8. Configure Cloud Audit Logs and create alerting for repeated denials

**Pitfalls**: IAP adds 10-50ms latency per request; test application performance. WebSocket connections through IAP require specific backend service configuration. Service-to-service calls within GKE should bypass IAP using internal service mesh, not external IAP endpoints. Break-glass access should use a separate IAM binding without access level conditions.

## Output Format

```
Google Cloud IAP Configuration Report
==================================================
Project: ecommerce-internal
Report Date: 2026-02-23

IAP-PROTECTED SERVICES:
  Backend Services:     12
  App Engine:            1
  Cloud Run:             2
  IAP TCP Tunnels:       4 (SSH access)
  Total:                19

ACCESS CONTROL:
  IAM Bindings:         34
  With Access Levels:   18 (52.9%)
  Access Levels:         3 (managed-device, corp-network, high-trust)

SESSION POLICIES:
  Admin tools:          4h re-auth (SECURE_KEY)
  Sensitive apps:       4h re-auth (LOGIN)
  General tools:        8h re-auth (LOGIN)

ACCESS LOGS (last 24h):
  Total requests:       23,456
  Authenticated:        23,289 (99.3%)
  Denied by IAM:           112
  Denied by access level:   55
  Unique users:            134
```

===== END FILE =====


===== FILE: skills/configuring-identity-aware-proxy-with-google-iap/LICENSE =====

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


===== FILE: skills/configuring-identity-aware-proxy-with-google-iap/assets/template.md =====
# Google Cloud IAP - Configuration Checklist

## Project Information
| Field | Value |
|-------|-------|
| GCP Project | ecommerce-internal-prod |
| Organization | E-Commerce Corp |
| IAP OAuth Brand | Corporate Applications |
| Lead | Security Engineering |

## IAP Backend Service Configuration

| Service | Platform | IAP Enabled | Access Level | Re-auth | Groups |
|---------|----------|-------------|-------------|---------|--------|
| admin-dashboard | GKE | Yes | managed-device | 1h / SECURE_KEY | admins@ |
| internal-api | Cloud Run | Yes | corp-network | 8h / LOGIN | engineering@ |
| monitoring | GKE | Yes | None | 8h / LOGIN | sre@, engineering@ |
| hr-portal | Compute Engine | Yes | high-trust | 4h / LOGIN | hr@ |
| finance-app | Compute Engine | Yes | high-trust | 1h / SECURE_KEY | finance@ |
| wiki | App Engine | Yes | None | 8h / LOGIN | all-staff@ |

## Access Levels

| Level Name | Type | Device Policy | Encryption | IP Restriction | Region |
|-----------|------|--------------|------------|---------------|--------|
| managed-device | Basic | Admin-approved, Screen lock | ENCRYPTED | None | US, GB |
| corp-network | Basic | None | None | 203.0.113.0/24 | US |
| high-trust | Custom (CEL) | Admin-approved, Encrypted | ENCRYPTED | Corp network OR ChromeOS | US |

## IAP TCP Tunnel Access

| VM | Zone | IAP Tunnel | Groups | External IP Removed |
|----|------|-----------|--------|-------------------|
| bastion-1 | us-central1-a | SSH | sre@ | Yes |
| db-admin | us-central1-b | SSH | dba@ | Yes |
| windows-admin | us-east1-b | RDP | admins@ | Yes |

## Validation Checklist
- [x] IAP enabled on all internal backend services
- [x] Direct access blocked (no public IPs, firewall rules restrict to LB + IAP)
- [x] Access levels applied to sensitive services
- [x] Re-authentication configured per sensitivity tier
- [x] Break-glass IAM binding created without access level conditions
- [ ] Service account programmatic access tested
- [x] Audit logs enabled and flowing to BigQuery
- [x] Alert policies created for access denials

## Sign-Off
| Role | Name | Date | Approved |
|------|------|------|----------|
| Security Architect | _________________ | __________ | [ ] |
| Cloud Platform Lead | _________________ | __________ | [ ] |

===== END FILE =====


===== FILE: skills/configuring-identity-aware-proxy-with-google-iap/references/api-reference.md =====
# Google Identity-Aware Proxy (IAP) — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| google-cloud-iap | `pip install google-cloud-iap` | IAP admin and settings management |
| google-cloud-resource-manager | `pip install google-cloud-resource-manager` | GCP project enumeration |

## Key IAP Client Methods

| Method | Description |
|--------|-------------|
| `IdentityAwareProxyAdminServiceClient()` | Create IAP admin client |
| `get_iap_settings(name=)` | Get IAP configuration for a resource |
| `update_iap_settings(iap_settings=, update_mask=)` | Update IAP settings |
| `get_iam_policy(resource=)` | Get IAP IAM bindings |
| `set_iam_policy(resource=, policy=)` | Set IAP IAM bindings |
| `list_tunnel_dest_groups(parent=)` | List TCP forwarding tunnel groups |

## IAP IAM Roles

| Role | Description |
|------|-------------|
| `roles/iap.httpsResourceAccessor` | Access IAP-protected web resources |
| `roles/iap.tunnelResourceAccessor` | Access IAP TCP forwarding tunnels |
| `roles/iap.admin` | Full IAP administration |

## gcloud CLI Commands

```bash
gcloud iap web enable --resource-type=app-engine
gcloud iap tcp enable --resource-type=compute --dest-group=GROUP
gcloud iap web get-iam-policy --project=PROJECT
gcloud compute ssh INSTANCE --tunnel-through-iap
```

## External References

- [Google IAP Documentation](https://cloud.google.com/iap/docs)
- [google-cloud-iap Python Reference](https://cloud.google.com/python/docs/reference/iap/latest)
- [IAP Programmatic Auth](https://cloud.google.com/iap/docs/authentication-howto)

===== END FILE =====


===== FILE: skills/configuring-identity-aware-proxy-with-google-iap/references/standards.md =====
# Google Cloud IAP - Standards & References

## NIST SP 800-207: Zero Trust Architecture
- **Section 3.1**: Policy Engine - IAP evaluates identity and context per request
- **Section 3.2**: Trust Algorithm - Access Levels compute trust score
- **Section 4.2**: SDP Gateway Model - IAP acts as application-layer gateway
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-207/final

## CISA Zero Trust Maturity Model v2.0
- **Identity Pillar**: Per-request identity verification via IAP
- **Application Pillar**: Application-level access controls
- **URL**: https://www.cisa.gov/zero-trust-maturity-model

## Google Cloud Documentation
- **IAP Overview**: https://cloud.google.com/iap/docs/concepts-overview
- **Enabling IAP for Compute Engine**: https://cloud.google.com/iap/docs/enabling-compute-howto
- **Enabling IAP for App Engine**: https://cloud.google.com/iap/docs/app-engine-quickstart
- **Enabling IAP for Cloud Run**: https://cloud.google.com/iap/docs/enabling-cloud-run
- **IAP TCP Forwarding**: https://cloud.google.com/iap/docs/using-tcp-forwarding
- **Context-Aware Access**: https://cloud.google.com/iap/docs/cloud-iap-context-aware-access-howto
- **Managing Access**: https://cloud.google.com/iap/docs/managing-access
- **Access Context Manager**: https://cloud.google.com/access-context-manager/docs
- **Programmatic Authentication**: https://cloud.google.com/iap/docs/authentication-howto

## Google BeyondCorp Papers
- **BeyondCorp: A New Approach to Enterprise Security** (2014): https://research.google/pubs/pub43231/
- **BeyondCorp: The Access Proxy** (2017): https://research.google/pubs/pub45728/

## FedRAMP
- Google Cloud IAP operates within FedRAMP High boundary
- **URL**: https://cloud.google.com/security/compliance/fedramp

===== END FILE =====


===== FILE: skills/configuring-identity-aware-proxy-with-google-iap/references/workflows.md =====
# Google IAP Configuration Workflow

## Phase 1: Prerequisites (Day 1)
1. Enable IAP API: `gcloud services enable iap.googleapis.com`
2. Enable Access Context Manager API
3. Configure OAuth consent screen with organization branding
4. Create OAuth client credentials for IAP
5. Verify applications are behind HTTPS Load Balancer or Cloud Run/App Engine

## Phase 2: IAP Enablement (Day 2-3)

### Compute Engine / GKE Backend Services
1. Enable IAP on each backend service with OAuth credentials
2. Configure health checks to work through IAP
3. Verify backend service firewall rules allow only load balancer and IAP ranges
4. Block direct access to backend instances (remove external IPs, restrict firewall)

### App Engine
1. Enable IAP on App Engine with OAuth credentials
2. Verify no App Engine firewall rules bypass IAP
3. Test authentication flow with pilot users

### Cloud Run
1. Grant IAP service account Cloud Run Invoker role
2. Configure Cloud Run service with `--no-allow-unauthenticated`
3. Enable IAP on the backend service fronting Cloud Run
4. Test end-to-end request flow

### TCP Forwarding (SSH/RDP)
1. Grant IAP Tunnel Resource Accessor role to user groups
2. Remove public IP addresses from VMs
3. Configure firewall rules to allow only IAP tunnel IP ranges (35.235.240.0/20)
4. Test SSH/RDP access through IAP tunnel

## Phase 3: Access Control (Day 4-5)
1. Create IAM bindings mapping Google Groups to backend services
2. Add access level conditions for sensitive applications
3. Configure time-based conditions for admin access
4. Set up path-based conditions for API access
5. Test each binding with authorized and unauthorized users

## Phase 4: Access Levels (Day 6-7)
1. Create basic access levels for device posture (encryption, OS, screen lock)
2. Create IP-based access levels for corporate network
3. Create custom access levels with CEL for complex conditions
4. Apply access levels as conditions on IAM bindings
5. Validate with compliant and non-compliant devices

## Phase 5: Session and Re-auth (Day 8)
1. Configure session duration per application tier
2. Set re-authentication method (LOGIN or SECURE_KEY)
3. Test session expiry and re-authentication flow
4. Document expected user experience

## Phase 6: Audit and Monitoring (Day 9-10)
1. Enable data access audit logs for IAP
2. Create log-based metrics for access denials
3. Set up alerting for anomalous patterns
4. Build dashboard for IAP access analytics
5. Test break-glass access procedures

===== END FILE =====


===== FILE: skills/configuring-identity-aware-proxy-with-google-iap/scripts/agent.py =====
#!/usr/bin/env python3
"""Google Identity-Aware Proxy (IAP) configuration agent using google-cloud-iap."""

import json
import sys
import argparse
from datetime import datetime

try:
    from google.cloud import iap_v1
except ImportError:
    print("Install: pip install google-cloud-iap google-cloud-resource-manager")
    sys.exit(1)


def list_iap_tunnels(project_id):
    """List IAP TCP forwarding tunnels."""
    client = iap_v1.IdentityAwareProxyAdminServiceClient()
    parent = f"projects/{project_id}"
    tunnels = []
    try:
        request = iap_v1.ListTunnelDestGroupsRequest(parent=f"{parent}/iap_tunnel/locations/-")
        for group in client.list_tunnel_dest_groups(request=request):
            tunnels.append({
                "name": group.name,
                "cidrs": list(group.cidrs),
                "fqdns": list(group.fqdns),
            })
    except Exception as e:
        tunnels.append({"error": str(e)})
    return tunnels


def get_iap_settings(project_id, resource_type="web"):
    """Get IAP settings for web resources."""
    client = iap_v1.IdentityAwareProxyAdminServiceClient()
    resource_name = f"projects/{project_id}/iap_web"
    try:
        request = iap_v1.GetIapSettingsRequest(name=resource_name)
        settings = client.get_iap_settings(request=request)
        return {
            "name": settings.name,
            "access_settings": {
                "cors_settings": str(settings.access_settings.cors_settings) if settings.access_settings else "",
            },
        }
    except Exception as e:
        return {"error": str(e)}


def audit_iap_iam_policy(project_id):
    """Audit IAM bindings for IAP-secured resources."""
    client = iap_v1.IdentityAwareProxyAdminServiceClient()
    resource = f"projects/{project_id}/iap_web"
    try:
        policy = client.get_iam_policy(request={"resource": resource})
        bindings = []
        for binding in policy.bindings:
            bindings.append({
                "role": binding.role,
                "members": list(binding.members),
                "condition": str(binding.condition) if binding.condition else None,
            })
        return bindings
    except Exception as e:
        return [{"error": str(e)}]


def check_oauth_consent(project_id):
    """Verify OAuth consent screen configuration."""
    return {
        "check": "OAuth consent screen",
        "project": project_id,
        "requirements": [
            "Application type: Internal (for organization apps)",
            "Support email: Valid group email",
            "Authorized domains: Company domains only",
            "Scopes: Minimal required (email, profile)",
        ],
        "verification_url": f"https://console.cloud.google.com/apis/credentials/consent?project={project_id}",
    }


def run_audit(project_id):
    """Execute IAP configuration audit."""
    print(f"\n{'='*60}")
    print(f"  GOOGLE IAP CONFIGURATION AUDIT")
    print(f"  Project: {project_id}")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    settings = get_iap_settings(project_id)
    print(f"--- IAP SETTINGS ---")
    print(f"  {json.dumps(settings, indent=2)}")

    bindings = audit_iap_iam_policy(project_id)
    print(f"\n--- IAM BINDINGS ({len(bindings)}) ---")
    for b in bindings:
        if "error" not in b:
            print(f"  {b['role']}: {', '.join(b['members'][:3])}")

    tunnels = list_iap_tunnels(project_id)
    print(f"\n--- TCP TUNNELS ({len(tunnels)}) ---")
    for t in tunnels:
        if "error" not in t:
            print(f"  {t['name']}: CIDRs={t['cidrs']}")

    consent = check_oauth_consent(project_id)
    print(f"\n--- OAUTH CONSENT ---")
    for req in consent["requirements"]:
        print(f"  - {req}")

    return {"settings": settings, "bindings": bindings, "tunnels": tunnels, "consent": consent}


def main():
    parser = argparse.ArgumentParser(description="Google IAP Audit Agent")
    parser.add_argument("--project", required=True, help="GCP project ID")
    parser.add_argument("--audit", action="store_true", help="Run full audit")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    if args.audit:
        report = run_audit(args.project)
        if args.output:
            with open(args.output, "w") as f:
                json.dump(report, f, indent=2, default=str)
            print(f"\n[+] Report saved to {args.output}")
    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-identity-aware-proxy-with-google-iap/scripts/process.py =====
#!/usr/bin/env python3
"""
Google Cloud IAP - Configuration Audit Tool

Audits IAP-enabled backend services, IAM bindings, access levels,
session settings, and access logs for compliance validation.

Requirements:
    pip install google-cloud-compute google-auth requests
"""

import json
import subprocess
import sys
from datetime import datetime, timedelta, timezone
from typing import Any


def run_gcloud(args: list[str]) -> Any:
    """Execute gcloud command and return JSON output."""
    cmd = ["gcloud"] + args + ["--format=json"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
    if result.returncode != 0:
        return []
    try:
        return json.loads(result.stdout) if result.stdout.strip() else []
    except json.JSONDecodeError:
        return []


def audit_iap_backends(project_id: str) -> list[dict]:
    """Audit all backend services for IAP configuration."""
    print("\n[1/5] Auditing IAP-enabled backend services...")
    backends = run_gcloud(["compute", "backend-services", "list", "--project", project_id])
    if not isinstance(backends, list):
        return []

    results = []
    for backend in backends:
        name = backend.get("name", "unknown")
        iap = backend.get("iap", {})
        enabled = iap.get("enabled", False)

        info = {"name": name, "iap_enabled": enabled, "bindings": [], "has_conditions": False}

        if enabled:
            policy = run_gcloud([
                "iap", "web", "get-iam-policy",
                "--resource-type=backend-services",
                "--service", name, "--project", project_id
            ])
            bindings = policy.get("bindings", []) if isinstance(policy, dict) else []
            info["bindings"] = bindings
            info["bindings_count"] = len(bindings)
            info["has_conditions"] = any(b.get("condition") for b in bindings)
            print(f"  [IAP ON]  {name}: {len(bindings)} bindings, conditions: {info['has_conditions']}")
        else:
            print(f"  [IAP OFF] {name}")

        results.append(info)
    return results


def audit_access_levels(policy_id: str) -> list[dict]:
    """Audit Access Context Manager levels."""
    print("\n[2/5] Auditing access levels...")
    if not policy_id:
        print("  Skipped - no policy ID provided")
        return []

    levels = run_gcloud(["access-context-manager", "levels", "list", "--policy", policy_id])
    if not isinstance(levels, list):
        return []

    results = []
    for level in levels:
        name = level.get("name", "").split("/")[-1]
        title = level.get("title", "")
        basic = level.get("basic", {})
        has_device = any(
            c.get("devicePolicy") for c in basic.get("conditions", [])
        )
        has_ip = any(
            c.get("ipSubnetworks") for c in basic.get("conditions", [])
        )
        results.append({"name": name, "title": title, "has_device": has_device, "has_ip": has_ip})
        print(f"  {title}: device_policy={has_device}, ip_restriction={has_ip}")

    return results


def audit_iap_tunnel_access(project_id: str) -> dict:
    """Audit IAP TCP tunnel permissions."""
    print("\n[3/5] Auditing IAP tunnel access...")
    instances = run_gcloud(["compute", "instances", "list", "--project", project_id])
    if not isinstance(instances, list):
        return {"total_vms": 0}

    stats = {"total_vms": len(instances), "with_external_ip": 0, "tunnel_accessible": 0}
    for vm in instances:
        name = vm.get("name", "unknown")
        interfaces = vm.get("networkInterfaces", [])
        has_ext_ip = any(
            iface.get("accessConfigs") for iface in interfaces
        )
        if has_ext_ip:
            stats["with_external_ip"] += 1

    print(f"  Total VMs: {stats['total_vms']}, With external IP: {stats['with_external_ip']}")
    if stats["with_external_ip"] > 0:
        print(f"  [WARN] {stats['with_external_ip']} VMs have external IPs - consider removing for IAP-only access")
    return stats


def audit_iap_logs(project_id: str) -> dict:
    """Analyze recent IAP access logs."""
    print("\n[4/5] Analyzing IAP access logs (24h)...")
    start = (datetime.now(timezone.utc) - timedelta(hours=24)).strftime("%Y-%m-%dT%H:%M:%SZ")
    logs = run_gcloud([
        "logging", "read",
        f'resource.type="gce_backend_service" AND protoPayload.serviceName="iap.googleapis.com" AND timestamp>="{start}"',
        "--project", project_id, "--limit=500"
    ])
    if not isinstance(logs, list):
        return {"total": 0, "allowed": 0, "denied": 0}

    stats = {"total": len(logs), "allowed": 0, "denied": 0, "users": set()}
    for entry in logs:
        payload = entry.get("protoPayload", {})
        status = payload.get("status", {}).get("code", 0)
        user = payload.get("authenticationInfo", {}).get("principalEmail", "")
        if status == 0:
            stats["allowed"] += 1
        else:
            stats["denied"] += 1
        if user:
            stats["users"].add(user)

    stats["unique_users"] = len(stats["users"])
    del stats["users"]
    print(f"  Requests: {stats['total']}, Allowed: {stats['allowed']}, Denied: {stats['denied']}")
    return stats


def generate_report(project_id: str, backends: list, levels: list,
                    tunnels: dict, logs: dict) -> str:
    """Generate IAP audit report."""
    print("\n[5/5] Generating report...")
    now = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M UTC")
    iap_on = [b for b in backends if b["iap_enabled"]]
    iap_off = [b for b in backends if not b["iap_enabled"]]
    with_conditions = [b for b in iap_on if b.get("has_conditions")]

    report = f"""
Google Cloud IAP Audit Report
{'=' * 55}
Project: {project_id}
Generated: {now}

1. BACKEND SERVICES
   Total backend services:       {len(backends)}
   IAP enabled:                  {len(iap_on)}
   IAP disabled:                 {len(iap_off)}
   With access level conditions: {len(with_conditions)} / {len(iap_on)}

2. ACCESS LEVELS
   Total defined:                {len(levels)}
   With device policy:           {sum(1 for l in levels if l['has_device'])}
   With IP restrictions:         {sum(1 for l in levels if l['has_ip'])}

3. VM ACCESS
   Total VMs:                    {tunnels['total_vms']}
   With external IP:             {tunnels['with_external_ip']}

4. ACCESS LOGS (24h)
   Total requests:               {logs['total']}
   Allowed:                      {logs['allowed']}
   Denied:                       {logs['denied']}
   Unique users:                 {logs.get('unique_users', 0)}

5. RECOMMENDATIONS
"""
    recs = []
    if iap_off:
        recs.append(f"   - Enable IAP on {len(iap_off)} unprotected backend service(s)")
    if len(with_conditions) < len(iap_on):
        recs.append(f"   - Add access level conditions to {len(iap_on) - len(with_conditions)} IAP service(s)")
    if tunnels["with_external_ip"] > 0:
        recs.append(f"   - Remove external IPs from {tunnels['with_external_ip']} VM(s) for IAP-only access")
    if not recs:
        recs.append("   - Configuration meets best practices")
    report += "\n".join(recs)
    return report


def main():
    if len(sys.argv) < 2:
        print("Usage: python process.py <project-id> [access-policy-id]")
        sys.exit(1)

    project_id = sys.argv[1]
    policy_id = sys.argv[2] if len(sys.argv) > 2 else None

    backends = audit_iap_backends(project_id)
    levels = audit_access_levels(policy_id)
    tunnels = audit_iap_tunnel_access(project_id)
    logs = audit_iap_logs(project_id)
    report = generate_report(project_id, backends, levels, tunnels, logs)
    print(report)

    filename = f"iap_audit_{project_id}_{datetime.now().strftime('%Y%m%d')}.txt"
    with open(filename, "w") as f:
        f.write(report)
    print(f"\nReport saved to: {filename}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-microsegmentation-for-zero-trust/SKILL.md =====
---
name: configuring-microsegmentation-for-zero-trust
description: Configures microsegmentation policies to enforce least-privilege workload-to-workload
  access using tools such as VMware NSX, Illumio, and Calico, preventing lateral
  movement in zero trust architectures. Use when designing or implementing network
  microsegmentation as part of a zero trust architecture aligned with NIST SP 800-207.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zero-trust
- microsegmentation
- network-access
- lateral-movement
- network-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1021
- T1210
- T1570
- T1046
- T1018
---

# Configuring Microsegmentation for Zero Trust

## Prerequisites

- Understanding of zero trust principles (NIST SP 800-207)
- Knowledge of network segmentation concepts
- Familiarity with firewall and SDN technologies
- Experience with VMware NSX, Illumio, Guardicore, or Cisco ACI

## Overview

Microsegmentation divides a network into granular security zones, enforcing least-privilege access between workloads at the application layer rather than relying on traditional VLAN-based segmentation. In a zero trust architecture, microsegmentation eliminates implicit trust between workloads within the same network segment, preventing lateral movement even after an attacker gains initial access.

This skill covers designing microsegmentation policies using workload identity, implementing host-based and network-based enforcement, and validating segmentation effectiveness with tools like Illumio Core and VMware NSX.


## When to Use

- When deploying or configuring configuring microsegmentation for zero trust capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with zero trust architecture concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Architecture

### Microsegmentation Models

1. **Network-Based (VMware NSX, Cisco ACI)**: Distributed firewall rules enforced at the hypervisor or network fabric level
2. **Host-Based (Illumio, Guardicore)**: Agent-based enforcement at the OS level using iptables/WFP rules
3. **Container-Based (Calico, Cilium)**: Network policies enforced at the pod/container level in Kubernetes
4. **Application-Based (Zscaler Workload Segmentation)**: Identity-based segmentation based on software identity rather than IP addresses

### Enforcement Points

```
Traditional Segmentation        Microsegmentation
┌─────────────────┐            ┌──────────────────────┐
│  VLAN 10        │            │  Workload A ←policy→ │
│  ┌───┐ ┌───┐   │            │  Workload B ←policy→ │
│  │ A │ │ B │   │            │  Workload C ←policy→ │
│  └───┘ └───┘   │            │  Workload D ←policy→ │
│  (trust each    │            │  (zero trust between  │
│   other)        │            │   every pair)         │
└─────────────────┘            └──────────────────────┘
```

## Key Concepts

### Application Dependency Mapping
Before creating segmentation policies, discover actual communication flows between workloads using traffic telemetry. Tools like Illumio, Guardicore, and AppDynamics provide application dependency maps showing which workloads communicate, over which ports, and how frequently.

### Policy Modeling
Draft policies in monitor/visibility mode before enforcement. This allows validation that proposed rules will not break legitimate traffic while identifying unnecessary or risky communication paths.

### Label-Based Policy
Modern microsegmentation uses labels (role, application, environment, location) instead of IP-based rules. Label-based policies are portable across environments and survive IP changes during migrations.

### Ring-Fencing
Isolate critical applications (PCI cardholder data environment, SWIFT financial systems, healthcare PHI) with strict allow-list policies that deny all traffic not explicitly permitted.

## Workflow

### Phase 1: Discovery and Mapping

1. **Deploy Visibility Agents**
   - Install lightweight agents on all workloads (servers, VMs, containers)
   - Configure agents to report real-time traffic telemetry to the management console
   - Allow 2-4 weeks of traffic collection to build a comprehensive flow map

2. **Build Application Dependency Map**
   - Review auto-discovered communication flows in the management console
   - Identify application tiers: web servers, app servers, databases, middleware
   - Map legitimate communication paths and flag unexpected connections
   - Document data flows for compliance scope (PCI, HIPAA)

3. **Assign Labels**
   - Create a labeling taxonomy: Role (web, app, db), Application (ERP, CRM), Environment (prod, dev, staging), Location (dc1, aws-east)
   - Apply labels to all workloads via the management console or API
   - Validate label accuracy against CMDB and application owner input

### Phase 2: Policy Design

4. **Define Segmentation Zones**
   - Environment isolation: Production cannot communicate with Development
   - Tier isolation: Database tier only accepts connections from application tier
   - Application ring-fencing: PCI applications isolated from non-PCI workloads
   - Administrative access: Jump servers are the only management path

5. **Create Allow-List Policies**
   - For each application, define explicit allow rules for required communication
   - Use label-based rules rather than IP-based where possible
   - Include process-level restrictions where supported (e.g., only httpd on port 443)
   - Set default-deny for all unlisted communication

6. **Model Policies in Test Mode**
   - Enable policies in visibility/test mode (do not enforce)
   - Monitor for would-be blocked legitimate traffic
   - Refine policies based on test results over 1-2 weeks
   - Get application owner sign-off before enforcement

### Phase 3: Enforcement

7. **Enforce Incrementally**
   - Start with the most isolated, lowest-risk application
   - Switch policy from test mode to enforce mode
   - Monitor for application issues in the first 24-48 hours
   - Proceed to next application after validation

8. **Validate Segmentation**
   - Run penetration tests attempting lateral movement between segments
   - Verify that blocked traffic generates alerts in the management console
   - Test emergency override procedures (break-glass)
   - Document enforcement status for each application zone

### Phase 4: Operational Maintenance

9. **Ongoing Policy Management**
   - Integrate with CI/CD: auto-label new workloads from deployment pipelines
   - Review policy violations weekly and investigate anomalies
   - Update policies when applications change or new services deploy
   - Perform quarterly segmentation effectiveness reviews

## Validation Checklist

- [ ] Agents deployed on all in-scope workloads
- [ ] Application dependency map reviewed and approved by app owners
- [ ] Labels assigned and validated against CMDB
- [ ] Policies modeled in test mode with no false positives for 2+ weeks
- [ ] Policies enforced incrementally with monitoring
- [ ] Default-deny active for all segmented zones
- [ ] Lateral movement tests confirm blocked unauthorized traffic
- [ ] Alerting configured for policy violations
- [ ] Break-glass procedure documented and tested
- [ ] Compliance auditor sign-off for regulated environments

## References

- NIST SP 800-207: Zero Trust Architecture
- CISA Zero Trust Maturity Model v2.0 - Network Pillar
- Illumio Core Administration Guide
- VMware NSX Distributed Firewall Configuration Guide
- Forrester Zero Trust eXtended (ZTX) Framework

===== END FILE =====


===== FILE: skills/configuring-microsegmentation-for-zero-trust/LICENSE =====

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


===== FILE: skills/configuring-microsegmentation-for-zero-trust/assets/template.md =====
# Microsegmentation Implementation Plan Template

## Project Information

| Field | Value |
|---|---|
| Project Name | |
| Organization | |
| Project Lead | |
| Start Date | |
| Segmentation Tool | [Illumio / VMware NSX / Guardicore / Cisco ACI] |

## Workload Inventory

| Workload | IP Address | OS | Role | Application | Environment | Location |
|---|---|---|---|---|---|---|
| | | | web | | prod | |
| | | | app | | prod | |
| | | | db | | prod | |

## Segmentation Zone Design

### Zone Definitions

| Zone Name | Description | Workloads | Default Policy |
|---|---|---|---|
| PCI-CDE | Cardholder data environment | [list] | Deny-all |
| HR-Systems | HR applications | [list] | Deny-all |
| DMZ | Internet-facing services | [list] | Deny-all |
| Management | Admin/monitoring | [list] | Restricted |

### Inter-Zone Communication Matrix

| Source Zone | Destination Zone | Ports/Protocols | Justification |
|---|---|---|---|
| DMZ | App-Tier | 8080/tcp | Web application traffic |
| App-Tier | DB-Tier | 3306/tcp | Database queries |
| Management | All Zones | 22/tcp, 9090/tcp | SSH and monitoring |

## Policy Rules

### Allow Rules

| Rule ID | Source | Destination | Port | Protocol | Process | Justification |
|---|---|---|---|---|---|---|
| 1 | | | | tcp | | |
| 2 | | | | tcp | | |

### Default Deny
- All traffic not explicitly allowed is denied
- Deny rule logged and alerted

## Enforcement Schedule

| Week | Activity | Applications | Risk Level |
|---|---|---|---|
| 1-2 | Agent deployment and discovery | All | Low |
| 3-4 | Label assignment and validation | All | Low |
| 5-6 | Policy design and test mode | All | Low |
| 7 | Enforce: Dev/Test environments | Dev apps | Low |
| 8 | Enforce: Low-risk production | Non-critical | Medium |
| 9-10 | Enforce: Business-critical apps | ERP, CRM | High |
| 11-12 | Enforce: Regulated environments | PCI, HIPAA | High |

## Validation Tests

- [ ] Legitimate traffic flows uninterrupted after enforcement
- [ ] Unauthorized cross-zone traffic is blocked
- [ ] Lateral movement from compromised workload is contained
- [ ] Policy violation alerts appear in SIEM
- [ ] Break-glass procedure works for emergency access
- [ ] Application dependency map matches actual flows

## Sign-Off

| Stakeholder | Role | Approval | Date |
|---|---|---|---|
| | Security Architecture | | |
| | Network Operations | | |
| | Application Owners | | |
| | Compliance/Audit | | |

===== END FILE =====


===== FILE: skills/configuring-microsegmentation-for-zero-trust/references/api-reference.md =====
# Microsegmentation for Zero Trust — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| boto3 | `pip install boto3` | AWS security group audit |
| requests | `pip install requests` | Illumio / Guardicore API client |

## Key boto3 EC2 Methods

| Method | Description |
|--------|-------------|
| `describe_security_groups()` | List SGs with inbound/outbound rules |
| `authorize_security_group_ingress()` | Add inbound rule |
| `revoke_security_group_ingress()` | Remove inbound rule |

## Illumio PCE API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v2/orgs/{id}/workloads` | List managed workloads |
| GET | `/api/v2/orgs/{id}/sec_policy/draft/rule_sets` | List rule sets |
| PUT | `/api/v2/orgs/{id}/workloads/{id}` | Update workload enforcement mode |

## Segmentation Enforcement Modes

| Mode | Description |
|------|-------------|
| Visibility Only | Monitor traffic without blocking |
| Selective | Block specific flows, allow rest |
| Full | Deny all, allow by policy (zero trust) |

## External References

- [Illumio API Guide](https://docs.illumio.com/core/23.5/API-Reference/index.html)
- [NIST SP 800-207 Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final)
- [AWS Security Groups Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html)

===== END FILE =====


===== FILE: skills/configuring-microsegmentation-for-zero-trust/references/standards.md =====
# Standards and Frameworks Reference

## NIST SP 800-207: Zero Trust Architecture

### Microsegmentation as ZTA Deployment Model
NIST SP 800-207 identifies microsegmentation as one of three primary deployment approaches for zero trust:
- Places individual or groups of resources on a unique network segment protected by a gateway security component
- The enterprise places infrastructure devices such as intelligent switches, next-generation firewalls, or special-purpose gateway devices to act as PEPs protecting each resource or group of resources
- This approach can be implemented using software-defined networking (SDN) or hypervisor-level enforcement

### Applicable Controls
- **AC-4 (Information Flow Enforcement)**: Microsegmentation enforces approved information flows between workloads
- **SC-7 (Boundary Protection)**: Each microsegment boundary acts as a security boundary
- **SI-4 (Information System Monitoring)**: Microsegmentation tools provide flow telemetry for monitoring

## CISA Zero Trust Maturity Model v2.0

### Network Pillar - Microsegmentation Maturity

| Level | Network Segmentation | Microsegmentation | Traffic Management |
|---|---|---|---|
| Traditional | Large, macro-segmented perimeters | None | Static ACLs |
| Initial | Defined architecture with some isolation | Initial workload isolation | Basic flow visibility |
| Advanced | Ingress/egress micro-perimeters | Workload-level microsegmentation | Identity-based traffic rules |
| Optimal | Full microsegmentation, dynamically defined | Automated, adaptive policies | ML-driven anomaly detection |

### Cross-Cutting: Visibility and Analytics
- Flow telemetry from microsegmentation agents feeds into SIEM/SOAR
- Application dependency maps provide baseline for anomaly detection
- Policy violation alerts enable real-time incident detection

## PCI DSS v4.0

### Microsegmentation for Scope Reduction
- **Requirement 1.3**: Network controls restrict access to and from the cardholder data environment (CDE)
- **Requirement 1.4**: Network connections between trusted and untrusted networks are controlled
- Microsegmentation can reduce PCI scope by isolating CDE workloads from non-CDE systems
- Compensating control: host-based microsegmentation validated by QSA as equivalent to network segmentation

## Forrester Zero Trust eXtended (ZTX) Framework

### Workload Security Pillar
- Microsegmentation is a core capability for securing workloads
- Policies should be based on workload identity and context, not network location
- Continuous monitoring of east-west traffic for anomaly detection
- Integration with DevOps pipelines for automated policy management

## VMware NSX Distributed Firewall

### Architecture
- Stateful Layer 4-7 firewall embedded in the hypervisor kernel
- Policies evaluated at the vNIC level before traffic reaches the physical network
- Context-aware rules using Active Directory groups, VM tags, and application identification
- No network topology changes required for deployment

## Illumio Core Platform

### Architecture
- Virtual Enforcement Node (VEN) agents installed on workloads
- Policy Compute Engine (PCE) centralizes policy management and visualization
- Enforcement via native OS firewall (iptables on Linux, WFP on Windows)
- Label-based policy model: Role, Application, Environment, Location

## Guardicore (Akamai)

### Architecture
- Lightweight agents provide process-level visibility and enforcement
- Reveal module builds application dependency maps
- Centra management platform for policy creation and monitoring
- Supports bare-metal, VM, container, and cloud workloads

===== END FILE =====


===== FILE: skills/configuring-microsegmentation-for-zero-trust/references/workflows.md =====
# Microsegmentation Implementation Workflows

## Workflow 1: Microsegmentation Deployment Lifecycle

```
┌──────────────────────┐
│ 1. Discovery          │
│ - Deploy agents       │
│ - Collect traffic     │
│   telemetry (2-4 wks)│
│ - Build flow map      │
└──────────┬───────────┘
           v
┌──────────────────────┐
│ 2. Classification     │
│ - Assign workload     │
│   labels (role, app,  │
│   env, location)      │
│ - Validate with CMDB  │
│ - Group by app tier   │
└──────────┬───────────┘
           v
┌──────────────────────┐
│ 3. Policy Design      │
│ - Define zones        │
│ - Create allow-list   │
│   rules per app       │
│ - Set default-deny    │
│ - Document exceptions │
└──────────┬───────────┘
           v
┌──────────────────────┐
│ 4. Test Mode          │
│ - Enable policies in  │
│   visibility mode     │
│ - Monitor would-block │
│   events (1-2 weeks)  │
│ - Refine rules        │
└──────────┬───────────┘
           v
┌──────────────────────┐
│ 5. Enforcement        │
│ - Enforce per-app,    │
│   starting low-risk   │
│ - Monitor 24-48 hrs   │
│ - Proceed to next app │
└──────────┬───────────┘
           v
┌──────────────────────┐
│ 6. Continuous Ops     │
│ - Weekly violation    │
│   review              │
│ - Quarterly audits    │
│ - CI/CD integration   │
│ - Incident response   │
└──────────────────────┘
```

## Workflow 2: Policy Creation Flow

```
Identify Application
    │
    v
┌─────────────────────┐
│ Map Dependencies     │
│ - Inbound sources    │
│ - Outbound targets   │
│ - Ports/protocols    │
│ - Process names      │
└──────────┬──────────┘
           v
┌─────────────────────┐
│ Define Labels        │
│ Role: web/app/db     │
│ App: erp/crm/hr      │
│ Env: prod/dev/stg    │
│ Loc: dc1/aws/azure   │
└──────────┬──────────┘
           v
┌─────────────────────┐
│ Create Rules         │
│ Allow: web→app:8080  │
│ Allow: app→db:3306   │
│ Allow: mon→all:9090  │
│ Deny: all other      │
└──────────┬──────────┘
           v
┌─────────────────────┐
│ Test and Validate    │
│ - Simulate in test   │
│ - Check flow map     │
│ - App owner sign-off │
└──────────┬──────────┘
           v
┌─────────────────────┐
│ Enforce and Monitor  │
│ - Switch to enforce  │
│ - Alert on violations│
│ - Log to SIEM        │
└─────────────────────┘
```

## Workflow 3: Ring-Fencing Critical Assets

```
Identify Critical Asset (e.g., PCI CDE Database)
    │
    v
┌──────────────────────────────────────┐
│ 1. Baseline Traffic                   │
│ - Observe all inbound/outbound flows │
│ - Document legitimate connections    │
│ - Identify unnecessary connections   │
└──────────────┬───────────────────────┘
               v
┌──────────────────────────────────────┐
│ 2. Define Ring-Fence Rules            │
│ - Allow: app-server → db:5432        │
│ - Allow: backup-agent → db:5432      │
│ - Allow: monitoring → db:9100        │
│ - Deny: ALL other inbound            │
│ - Deny: ALL outbound (except DNS,NTP)│
└──────────────┬───────────────────────┘
               v
┌──────────────────────────────────────┐
│ 3. Test with Production Traffic       │
│ - Enable in test mode                 │
│ - Verify zero false positives         │
│ - Validate backup and monitoring work │
└──────────────┬───────────────────────┘
               v
┌──────────────────────────────────────┐
│ 4. Enforce and Lock Down              │
│ - Switch to enforcement               │
│ - Enable alerting on any violation    │
│ - Review violations daily             │
│ - QSA validation for PCI scope       │
└──────────────────────────────────────┘
```

## Workflow 4: Incident Response with Microsegmentation

```
Alert: Unusual East-West Traffic Detected
    │
    v
┌─────────────────────────┐
│ 1. Investigate           │
│ - Review flow in console │
│ - Check source workload  │
│ - Identify destination   │
│ - Cross-ref with SIEM    │
└──────────┬──────────────┘
           v
┌──────────────────────────────┐
│ 2. Contain                    │
│ - Apply quarantine policy     │
│   (deny all except forensic) │
│ - Isolate compromised         │
│   workload instantly          │
└──────────┬───────────────────┘
           v
┌─────────────────────────┐
│ 3. Assess Impact         │
│ - Check if lateral move  │
│   was blocked by policy  │
│ - Review adjacent zones  │
│ - Determine blast radius │
└──────────┬──────────────┘
           v
┌─────────────────────────┐
│ 4. Remediate             │
│ - Patch/reimagevworkload │
│ - Strengthen policies    │
│ - Remove quarantine      │
│ - Post-incident review   │
└─────────────────────────┘
```

===== END FILE =====


===== FILE: skills/configuring-microsegmentation-for-zero-trust/scripts/agent.py =====
#!/usr/bin/env python3
"""Microsegmentation audit agent for zero trust network enforcement."""

import json
import os
import sys
import argparse
from datetime import datetime

try:
    import requests
    requests.packages.urllib3.disable_warnings()
except ImportError:
    print("Install: pip install requests")
    sys.exit(1)


def audit_aws_security_groups(session):
    """Audit AWS security groups for microsegmentation compliance."""
    import boto3
    ec2 = session.client("ec2")
    findings = []
    for sg in ec2.describe_security_groups()["SecurityGroups"]:
        for rule in sg.get("IpPermissions", []):
            for ip_range in rule.get("IpRanges", []):
                cidr = ip_range.get("CidrIp", "")
                if cidr == "0.0.0.0/0":
                    findings.append({
                        "sg_id": sg["GroupId"],
                        "sg_name": sg.get("GroupName", ""),
                        "port": rule.get("FromPort", "all"),
                        "cidr": cidr,
                        "severity": "HIGH",
                        "recommendation": "Restrict to specific CIDR blocks",
                    })
                elif "/" in cidr:
                    prefix = int(cidr.split("/")[1])
                    if prefix < 24:
                        findings.append({
                            "sg_id": sg["GroupId"],
                            "sg_name": sg.get("GroupName", ""),
                            "port": rule.get("FromPort", "all"),
                            "cidr": cidr,
                            "severity": "MEDIUM",
                            "recommendation": f"Narrow CIDR from /{prefix} to /32 or workload-specific range",
                        })
    return findings


def check_illumio_workloads(base_url, api_key, org_id):
    """Check Illumio workload segmentation status."""
    headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}
    try:
        resp = requests.get(f"{base_url}/api/v2/orgs/{org_id}/workloads",
                            headers=headers, verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true", timeout=30)  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        resp.raise_for_status()
        workloads = resp.json()
        return [{
            "hostname": w.get("hostname", ""),
            "enforcement_mode": w.get("enforcement_mode", ""),
            "visibility_level": w.get("visibility_level", ""),
            "online": w.get("online", False),
        } for w in workloads[:20]]
    except Exception as e:
        return [{"error": str(e)}]


def generate_segmentation_policy(app_tiers):
    """Generate microsegmentation policy recommendations."""
    policies = []
    for tier in app_tiers:
        policies.append({
            "tier": tier["name"],
            "allowed_inbound": tier.get("inbound_from", []),
            "allowed_ports": tier.get("ports", []),
            "deny_default": True,
            "enforcement": "block",
        })
    return {
        "principle": "Zero Trust — deny all, allow by exception",
        "policies": policies,
        "example_tiers": [
            {"name": "web", "inbound_from": ["load-balancer"], "ports": [443]},
            {"name": "app", "inbound_from": ["web"], "ports": [8080]},
            {"name": "db", "inbound_from": ["app"], "ports": [5432]},
        ],
    }


def run_audit(profile=None, region="us-east-1"):
    """Execute microsegmentation audit."""
    import boto3
    session = boto3.Session(profile_name=profile, region_name=region)
    print(f"\n{'='*60}")
    print(f"  MICROSEGMENTATION ZERO TRUST AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    sg_findings = audit_aws_security_groups(session)
    print(f"--- SECURITY GROUP FINDINGS ({len(sg_findings)}) ---")
    for f in sg_findings[:15]:
        print(f"  [{f['severity']}] {f['sg_id']} ({f['sg_name']}): port {f['port']} from {f['cidr']}")

    policy = generate_segmentation_policy([])
    print(f"\n--- RECOMMENDED SEGMENTATION MODEL ---")
    print(f"  Principle: {policy['principle']}")
    for tier in policy["example_tiers"]:
        print(f"  {tier['name']}: allow from {tier['inbound_from']} on ports {tier['ports']}")

    return {"sg_findings": sg_findings, "policy": policy}


def main():
    parser = argparse.ArgumentParser(description="Microsegmentation Audit Agent")
    parser.add_argument("--profile", help="AWS CLI profile")
    parser.add_argument("--region", default="us-east-1", help="AWS region")
    parser.add_argument("--audit", action="store_true", help="Run audit")
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


===== FILE: skills/configuring-microsegmentation-for-zero-trust/scripts/process.py =====
#!/usr/bin/env python3
"""
Microsegmentation Policy Analyzer and Flow Validator

Analyzes network flow data to identify microsegmentation opportunities,
validates policies against observed traffic, and generates segmentation reports.
"""

import json
import csv
import sys
from collections import defaultdict
from datetime import datetime
from pathlib import Path
from typing import Optional


def parse_flow_data(flow_file: str) -> list:
    """Parse network flow data from CSV or JSON format."""
    flows = []
    path = Path(flow_file)
    if path.suffix == ".csv":
        with open(flow_file) as f:
            reader = csv.DictReader(f)
            for row in reader:
                flows.append({
                    "src_ip": row.get("src_ip", ""),
                    "dst_ip": row.get("dst_ip", ""),
                    "src_port": int(row.get("src_port", 0)),
                    "dst_port": int(row.get("dst_port", 0)),
                    "protocol": row.get("protocol", "tcp").lower(),
                    "bytes": int(row.get("bytes", 0)),
                    "packets": int(row.get("packets", 0)),
                    "timestamp": row.get("timestamp", ""),
                    "src_label": row.get("src_label", ""),
                    "dst_label": row.get("dst_label", ""),
                })
    elif path.suffix == ".json":
        with open(flow_file) as f:
            flows = json.load(f)
    return flows


def build_dependency_map(flows: list) -> dict:
    """Build an application dependency map from flow data."""
    dep_map = defaultdict(lambda: defaultdict(lambda: {
        "ports": set(), "protocols": set(), "total_bytes": 0,
        "total_packets": 0, "flow_count": 0
    }))

    for flow in flows:
        src = flow.get("src_label") or flow["src_ip"]
        dst = flow.get("dst_label") or flow["dst_ip"]
        port = flow["dst_port"]
        proto = flow["protocol"]

        dep_map[src][dst]["ports"].add(port)
        dep_map[src][dst]["protocols"].add(proto)
        dep_map[src][dst]["total_bytes"] += flow.get("bytes", 0)
        dep_map[src][dst]["total_packets"] += flow.get("packets", 0)
        dep_map[src][dst]["flow_count"] += 1

    serializable = {}
    for src, destinations in dep_map.items():
        serializable[src] = {}
        for dst, stats in destinations.items():
            serializable[src][dst] = {
                "ports": sorted(list(stats["ports"])),
                "protocols": sorted(list(stats["protocols"])),
                "total_bytes": stats["total_bytes"],
                "total_packets": stats["total_packets"],
                "flow_count": stats["flow_count"],
            }
    return serializable


def generate_segmentation_rules(dep_map: dict, default_action: str = "deny") -> list:
    """Generate microsegmentation rules from dependency map."""
    rules = []
    rule_id = 1

    for src, destinations in dep_map.items():
        for dst, stats in destinations.items():
            for port in stats["ports"]:
                for proto in stats["protocols"]:
                    rules.append({
                        "id": rule_id,
                        "action": "allow",
                        "src": src,
                        "dst": dst,
                        "port": port,
                        "protocol": proto,
                        "justification": f"Observed {stats['flow_count']} flows, {stats['total_bytes']} bytes",
                        "status": "proposed",
                    })
                    rule_id += 1

    rules.append({
        "id": rule_id,
        "action": default_action,
        "src": "any",
        "dst": "any",
        "port": "any",
        "protocol": "any",
        "justification": "Default deny - zero trust baseline",
        "status": "proposed",
    })
    return rules


def validate_policy_against_flows(policy_rules: list, flows: list) -> dict:
    """Validate segmentation policy against observed flows. Identify blocks and gaps."""
    results = {
        "allowed_flows": 0,
        "blocked_flows": 0,
        "unmatched_flows": 0,
        "blocked_details": [],
        "unmatched_details": [],
    }

    allow_rules = [r for r in policy_rules if r["action"] == "allow"]
    has_default_deny = any(
        r["action"] == "deny" and r.get("src") == "any" and r.get("dst") == "any"
        for r in policy_rules
    )

    for flow in flows:
        src = flow.get("src_label") or flow["src_ip"]
        dst = flow.get("dst_label") or flow["dst_ip"]
        port = flow["dst_port"]
        proto = flow["protocol"]

        matched = False
        for rule in allow_rules:
            src_match = rule["src"] in (src, "any")
            dst_match = rule["dst"] in (dst, "any")
            port_match = rule["port"] in (port, "any")
            proto_match = rule["protocol"] in (proto, "any")

            if src_match and dst_match and port_match and proto_match:
                results["allowed_flows"] += 1
                matched = True
                break

        if not matched:
            if has_default_deny:
                results["blocked_flows"] += 1
                results["blocked_details"].append({
                    "src": src, "dst": dst, "port": port, "protocol": proto,
                    "timestamp": flow.get("timestamp", ""),
                })
            else:
                results["unmatched_flows"] += 1
                results["unmatched_details"].append({
                    "src": src, "dst": dst, "port": port, "protocol": proto,
                })

    return results


def identify_segmentation_zones(flows: list, workload_labels: dict) -> dict:
    """Identify natural segmentation zones from flow patterns and labels."""
    zones = defaultdict(lambda: {"workloads": set(), "internal_flows": 0, "external_flows": 0})

    for flow in flows:
        src_ip = flow["src_ip"]
        dst_ip = flow["dst_ip"]

        src_zone = workload_labels.get(src_ip, {}).get("application", "unknown")
        dst_zone = workload_labels.get(dst_ip, {}).get("application", "unknown")

        zones[src_zone]["workloads"].add(src_ip)
        zones[dst_zone]["workloads"].add(dst_ip)

        if src_zone == dst_zone:
            zones[src_zone]["internal_flows"] += 1
        else:
            zones[src_zone]["external_flows"] += 1
            zones[dst_zone]["external_flows"] += 1

    serializable = {}
    for zone_name, stats in zones.items():
        serializable[zone_name] = {
            "workloads": sorted(list(stats["workloads"])),
            "workload_count": len(stats["workloads"]),
            "internal_flows": stats["internal_flows"],
            "external_flows": stats["external_flows"],
            "isolation_ratio": round(
                stats["internal_flows"] / max(stats["internal_flows"] + stats["external_flows"], 1),
                2
            ),
        }
    return serializable


def detect_anomalous_flows(flows: list, policy_rules: list) -> list:
    """Detect flows that violate segmentation policy and may indicate lateral movement."""
    anomalies = []
    allow_set = set()

    for rule in policy_rules:
        if rule["action"] == "allow":
            allow_set.add((rule["src"], rule["dst"], rule["port"], rule["protocol"]))

    for flow in flows:
        src = flow.get("src_label") or flow["src_ip"]
        dst = flow.get("dst_label") or flow["dst_ip"]
        port = flow["dst_port"]
        proto = flow["protocol"]

        is_allowed = (
            (src, dst, port, proto) in allow_set
            or ("any", dst, port, proto) in allow_set
            or (src, "any", port, proto) in allow_set
            or (src, dst, "any", proto) in allow_set
        )

        if not is_allowed:
            anomalies.append({
                "src": src,
                "dst": dst,
                "port": port,
                "protocol": proto,
                "bytes": flow.get("bytes", 0),
                "timestamp": flow.get("timestamp", ""),
                "severity": "high" if port in [22, 3389, 445, 135] else "medium",
                "reason": "Flow not covered by any allow rule",
            })

    return sorted(anomalies, key=lambda x: x["severity"])


def generate_segmentation_report(flows: list, policy_rules: list, workload_labels: dict) -> dict:
    """Generate comprehensive microsegmentation report."""
    dep_map = build_dependency_map(flows)
    zones = identify_segmentation_zones(flows, workload_labels)
    validation = validate_policy_against_flows(policy_rules, flows)
    anomalies = detect_anomalous_flows(flows, policy_rules)

    unique_sources = set(f.get("src_label") or f["src_ip"] for f in flows)
    unique_dests = set(f.get("dst_label") or f["dst_ip"] for f in flows)
    unique_ports = set(f["dst_port"] for f in flows)

    return {
        "generated": datetime.now().isoformat(),
        "summary": {
            "total_flows": len(flows),
            "unique_sources": len(unique_sources),
            "unique_destinations": len(unique_dests),
            "unique_ports": len(unique_ports),
            "segmentation_zones": len(zones),
            "policy_rules": len(policy_rules),
            "allowed_flows": validation["allowed_flows"],
            "blocked_flows": validation["blocked_flows"],
            "anomalies_detected": len(anomalies),
        },
        "zones": zones,
        "validation": validation,
        "anomalies": anomalies[:50],
        "dependency_map_summary": {
            src: len(dsts) for src, dsts in dep_map.items()
        },
    }


def main():
    import argparse

    parser = argparse.ArgumentParser(description="Microsegmentation Policy Analyzer")
    parser.add_argument("--flows", type=str, help="Path to flow data (CSV or JSON)")
    parser.add_argument("--policy", type=str, help="Path to policy rules JSON")
    parser.add_argument("--labels", type=str, help="Path to workload labels JSON")
    parser.add_argument("--action", choices=["map", "rules", "validate", "report", "anomalies"],
                        default="report", help="Action to perform")
    parser.add_argument("--output", type=str, default="segmentation_report.json", help="Output file")
    args = parser.parse_args()

    if not args.flows:
        parser.print_help()
        return

    flows = parse_flow_data(args.flows)
    print(f"Loaded {len(flows)} flows")

    policy_rules = []
    if args.policy:
        with open(args.policy) as f:
            policy_rules = json.load(f)

    workload_labels = {}
    if args.labels:
        with open(args.labels) as f:
            workload_labels = json.load(f)

    if args.action == "map":
        dep_map = build_dependency_map(flows)
        with open(args.output, "w") as f:
            json.dump(dep_map, f, indent=2)
        print(f"Dependency map: {len(dep_map)} sources mapped")

    elif args.action == "rules":
        dep_map = build_dependency_map(flows)
        rules = generate_segmentation_rules(dep_map)
        with open(args.output, "w") as f:
            json.dump(rules, f, indent=2)
        print(f"Generated {len(rules)} proposed rules")

    elif args.action == "validate":
        if not policy_rules:
            print("Error: --policy required for validation")
            return
        results = validate_policy_against_flows(policy_rules, flows)
        with open(args.output, "w") as f:
            json.dump(results, f, indent=2)
        print(f"Allowed: {results['allowed_flows']}, Blocked: {results['blocked_flows']}")

    elif args.action == "anomalies":
        if not policy_rules:
            print("Error: --policy required for anomaly detection")
            return
        anomalies = detect_anomalous_flows(flows, policy_rules)
        with open(args.output, "w") as f:
            json.dump(anomalies, f, indent=2)
        print(f"Detected {len(anomalies)} anomalous flows")

    elif args.action == "report":
        report = generate_segmentation_report(flows, policy_rules, workload_labels)
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"Report generated: {args.output}")
        print(f"  Zones: {report['summary']['segmentation_zones']}")
        print(f"  Anomalies: {report['summary']['anomalies_detected']}")

    print(f"Output saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-zscaler-private-access-for-ztna/SKILL.md =====
---
name: configuring-zscaler-private-access-for-ztna
description: 'Configures Zscaler Private Access (ZPA) to replace traditional VPN
  with zero trust network access by deploying App Connectors, defining application
  segments, configuring identity- and device-posture-based access policies, and
  integrating with IdPs. Use when replacing VPN concentrators with ZTNA or providing
  remote users secure access to internal applications without network-level connectivity.

  '
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zscaler
- zpa
- ztna
- zero-trust
- app-connector
- access-policy
- sase
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1133
- T1078
- T1021
- T1219
- T1190
---

# Configuring Zscaler Private Access for ZTNA

## When to Use

- When replacing traditional VPN concentrators with application-level zero trust access
- When providing remote users secure access to internal applications without network-level connectivity
- When implementing least-privilege access where users only see authorized applications
- When needing to make internal applications invisible to unauthorized users and the internet
- When integrating ZTNA with existing SASE architecture using Zscaler Internet Access (ZIA)

**Do not use** for applications requiring raw UDP access (ZPA primarily supports TCP), for providing full network-level access equivalent to site-to-site VPN (use ZPA AppProtection or branch connector instead), or when the organization requires on-premises-only access control without cloud dependency.

## Prerequisites

- Zscaler Private Access subscription (Business or Transformation edition)
- Identity provider configured: Okta, Microsoft Entra ID, Ping Identity, or SAML 2.0 IdP
- App Connector VM requirements: Linux VM (CentOS 7/8, RHEL 7/8, Ubuntu 18.04+, Amazon Linux 2) with 2 vCPU, 4GB RAM minimum
- Outbound connectivity from App Connector to ZPA cloud on port 443 (no inbound ports required)
- DNS resolution from App Connector to internal application FQDNs
- Zscaler Client Connector deployed on user endpoints

## Workflow

### Step 1: Deploy App Connectors in Application Network

App Connectors establish outbound-only tunnels to the ZPA cloud, providing access to internal applications.

```bash
# Download and install App Connector on Linux VM
# Obtain provisioning key from ZPA Admin Portal > Administration > App Connectors

# For RHEL/CentOS
sudo yum install -y https://yum.private.zscaler.com/yum/el7/zpa-connector-latest.rpm

# For Ubuntu/Debian
curl -sS https://dist.private.zscaler.com/apt/pubkey.gpg | sudo apt-key add -
echo "deb https://dist.private.zscaler.com/apt stable main" | sudo tee /etc/apt/sources.list.d/zpa.list
sudo apt update && sudo apt install -y zpa-connector

# Configure the connector with provisioning key
sudo /opt/zscaler/bin/zpa-connector configure \
  --provision-key "PROVISIONING_KEY_FROM_PORTAL"

# Start the connector service
sudo systemctl enable zpa-connector
sudo systemctl start zpa-connector

# Verify connector status
sudo systemctl status zpa-connector
sudo /opt/zscaler/bin/zpa-connector status

# Deploy second connector for HA (minimum 2 per site)
# Repeat on second VM with same App Connector Group provisioning key
```

### Step 2: Define Server Groups and Application Segments

Map internal applications to server groups and create application segments.

```text
ZPA Admin Portal Configuration:

1. Server Groups:
   Navigate to: Administration > App Connectors > Server Groups
   - Name: "DC-East-Servers"
   - App Connector Group: "DC-East-Connectors"
   - Servers:
     - hr-portal.internal.corp (10.1.1.50, TCP 443)
     - finance-app.internal.corp (10.1.1.51, TCP 443)
     - git.internal.corp (10.1.2.10, TCP 22, 443)

2. Application Segments:
   Navigate to: Resources > Application Segments > Add Application Segment
   - Name: "HR Applications"
   - Domain/URL: hr-portal.internal.corp
   - TCP Ports: 443
   - Server Group: DC-East-Servers
   - Health Reporting: Continuous
   - Bypass Type: Never (force all traffic through ZPA)

   - Name: "Engineering Tools"
   - Domain/URL: git.internal.corp, ci.internal.corp, wiki.internal.corp
   - TCP Ports: 22, 80, 443
   - Server Group: DC-East-Servers
   - Segment Group: "Engineering Segment Group"
```

### Step 3: Configure Access Policies

Define who can access which application segments based on identity and device posture.

```text
ZPA Admin Portal > Policies > Access Policy:

Rule 1: HR Team Access
  - Name: "HR Portal Access"
  - Action: ALLOW
  - Criteria:
    - User Groups: "HR-Department" (from IdP)
    - Application Segment: "HR Applications"
    - Device Posture Profile: "Corporate Managed Device"
    - Client Type: Zscaler Client Connector
  - Conditions:
    - SAML Attribute: department = "Human Resources"
    - Device Trust Level: "HIGH" (CrowdStrike ZTA score > 70)

Rule 2: Engineering Access
  - Name: "Engineering Tools Access"
  - Action: ALLOW
  - Criteria:
    - User Groups: "Engineering-Team", "DevOps-Team"
    - Application Segment: "Engineering Tools"
    - Device Posture Profile: "Developer Workstation"
  - Conditions:
    - Machine Group: "Engineering Laptops"

Rule 3: Contractor Limited Access
  - Name: "Contractor Wiki Access"
  - Action: ALLOW
  - Criteria:
    - User Groups: "External-Contractors"
    - Application Segment: "Wiki Only"
    - Client Type: Zscaler Client Connector OR Browser Access
  - Conditions:
    - Time Window: Mon-Fri 08:00-18:00 EST

Rule 4: Default Deny
  - Name: "Block All Other Access"
  - Action: DENY
  - Criteria: All Users, All Applications
  - Log: Enabled
```

### Step 4: Configure Device Posture Profiles

Integrate device posture signals from endpoint security tools.

```text
ZPA Admin Portal > Administration > Device Posture:

Profile 1: Corporate Managed Device
  - CrowdStrike Falcon: Running, ZTA Score >= 60
  - OS: Windows 10 21H2+, macOS 13+, Ubuntu 22.04+
  - Disk Encryption: Enabled (BitLocker/FileVault)
  - Firewall: Enabled
  - Screen Lock: Enabled

Profile 2: Developer Workstation
  - Inherits: Corporate Managed Device
  - CrowdStrike Falcon: ZTA Score >= 70
  - Patch Level: Within 30 days of latest
  - Certificate: Valid corporate certificate present

Profile 3: BYOD Device
  - OS: Latest minus 1 version
  - Browser: Chrome 120+ or Edge 120+
  - Antivirus: Any recognized AV running
```

### Step 5: Enable Browser Access for Clientless ZTNA

Configure Browser Access for users without Zscaler Client Connector installed.

```text
ZPA Admin Portal > Resources > Application Segments:

For "HR Applications" segment:
  - Enable Browser Access: Yes
  - Browser Access Type: HTTPS
  - Custom Domain: hr.access.company.com
  - Certificate: Upload TLS certificate for custom domain
  - Authentication: SAML via corporate IdP
  - Session Timeout: 4 hours
  - Clipboard Control: Disabled for sensitive apps
  - File Upload/Download: Restricted

For Browser Access Portal:
  - Portal URL: access.company.com
  - IdP: Microsoft Entra ID (SAML 2.0)
  - MFA: Required
  - Applications shown: Only authorized per user group
```

### Step 6: Configure Logging and Monitoring

Set up log streaming for SIEM integration and continuous monitoring.

```text
ZPA Admin Portal > Administration > Log Streaming Service:

Log Receiver Configuration:
  - Name: "Splunk-SIEM"
  - Type: Splunk (HEC)
  - Destination: https://splunk-hec.company.com:8088
  - HEC Token: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  - Log Types:
    - User Activity: Enabled
    - App Connector Status: Enabled
    - Audit Logs: Enabled
    - Browser Access: Enabled

# Splunk search for ZPA access anomalies
index=zscaler_zpa sourcetype=zpa:useractivity
| where action="denied"
| stats count by user, application, policy_name
| where count > 10
| sort -count
```

## Key Concepts

| Term | Definition |
|------|------------|
| App Connector | Lightweight Linux service that creates outbound-only encrypted tunnels from internal networks to ZPA cloud, providing access to applications without inbound ports |
| Application Segment | Logical grouping of internal applications defined by FQDN/IP and ports, mapped to server groups for access policy enforcement |
| Server Group | Collection of application servers associated with App Connector groups that can serve requests for application segments |
| Access Policy | Rules defining which users/groups can access which application segments under what conditions (device posture, time, location) |
| Zscaler Client Connector | Endpoint agent installed on user devices that routes traffic to ZPA cloud for policy enforcement and application access |
| Browser Access | Clientless ZTNA option allowing application access through a web browser without requiring Zscaler Client Connector installation |

## Tools & Systems

- **Zscaler Private Access (ZPA)**: Cloud-native ZTNA platform replacing VPN with identity-based application access
- **Zscaler Client Connector**: Cross-platform endpoint agent routing traffic through ZPA for policy enforcement
- **ZPA App Connector**: Outbound-only tunnel endpoint deployed in application networks
- **ZPA Admin Portal**: Web-based management console for policy, segment, and connector configuration
- **ZPA Log Streaming Service (LSS)**: Real-time log export to SIEM platforms (Splunk, Sentinel, QRadar)
- **CrowdStrike ZTA Integration**: Device posture scoring for conditional access policy enforcement

## Common Scenarios

### Scenario: Migrating 500-User Organization from Cisco AnyConnect VPN to ZPA

**Context**: A financial services firm with 500 employees uses Cisco AnyConnect for remote access. VPN split-tunnel configuration creates security gaps, and full-tunnel mode causes performance issues. The firm needs application-level access control for SOX compliance.

**Approach**:
1. Deploy 4 App Connectors (2 per data center) with HA configuration
2. Define application segments for 20 internal applications grouped by business function
3. Configure access policies mapping AD groups to application segments with device posture requirements
4. Integrate CrowdStrike ZTA scores as device posture input (minimum score 60 for standard, 80 for financial apps)
5. Enable Browser Access for contractors accessing the vendor portal
6. Configure LSS to stream access logs to Splunk for SOX audit trail
7. Run parallel operation for 3 weeks: VPN and ZPA side by side
8. Phase out VPN connections after validating all application access through ZPA

**Pitfalls**: App Connector DNS must resolve all internal FQDNs used in application segments. Wildcard domain segments can cause performance issues if too broad. Browser Access does not support all web application frameworks (WebSocket-heavy apps may require Client Connector). CrowdStrike ZTA integration requires Falcon sensor deployment on all endpoints before enforcing posture policies.

## Output Format

```
ZPA ZTNA Deployment Report
==================================================
Organization: FinanceCorp
Deployment Date: 2026-02-23

INFRASTRUCTURE:
  App Connectors: 4 (2x DC-East, 2x DC-West)
  Connector Status: All healthy
  Connector Version: 24.1.2

APPLICATION COVERAGE:
  Application Segments: 20
  Total Applications: 45
  Server Groups: 4
  Segment Groups: 6

ACCESS POLICIES:
  Total Rules: 12
  Allow Rules: 11
  Deny Rules: 1 (default deny)
  Device Posture Profiles: 3

USER ACCESS (last 30 days):
  Active Users: 487 / 500
  Total Sessions: 124,567
  Allowed Sessions: 123,890 (99.5%)
  Denied Sessions: 677 (0.5%)
  Browser Access Sessions: 2,341

VPN MIGRATION:
  Users migrated to ZPA: 487 / 500
  VPN decommission date: 2026-03-15
```

===== END FILE =====


===== FILE: skills/configuring-zscaler-private-access-for-ztna/LICENSE =====

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


===== FILE: skills/configuring-zscaler-private-access-for-ztna/assets/template.md =====
# ZPA ZTNA Deployment Checklist

## Project Information
| Field | Value |
|-------|-------|
| Organization | Acme Financial Corp |
| ZPA Customer ID | 12345678 |
| ZPA Cloud | zscaler.net (US) |
| Project Lead | Security Architecture Team |
| Start Date | 2026-01-20 |

## Pre-Deployment

### Identity Provider
- [x] IdP configured: Microsoft Entra ID (SAML 2.0)
- [x] SCIM provisioning enabled for user/group sync
- [x] MFA enforced for all users
- [x] User groups mapped: Engineering, HR, Finance, Contractors

### Network Requirements
- [x] Outbound HTTPS (443) to *.private.zscaler.com allowed
- [x] Outbound HTTPS (443) to *.zpath.net allowed
- [x] DNS resolution from App Connector VMs to all internal FQDNs verified
- [ ] Split-horizon DNS configured for hybrid access

## App Connector Deployment

| Connector Name | Group | Location | VM Specs | Status |
|---------------|-------|----------|----------|--------|
| ac-dc-east-01 | DC-East | AWS us-east-1 | t3.medium | Healthy |
| ac-dc-east-02 | DC-East | AWS us-east-1 | t3.medium | Healthy |
| ac-dc-west-01 | DC-West | AWS us-west-2 | t3.medium | Healthy |
| ac-dc-west-02 | DC-West | AWS us-west-2 | t3.medium | Healthy |

## Application Segments

| Segment Name | Applications | Ports | Server Group | Bypass |
|-------------|-------------|-------|-------------|--------|
| HR Applications | hr-portal.internal.corp | 443 | DC-East | Never |
| Finance Tools | finance.internal.corp, reports.internal.corp | 443 | DC-East | Never |
| Engineering | git.internal.corp, ci.internal.corp, wiki.internal.corp | 22, 443 | DC-East + DC-West | Never |
| Monitoring | grafana.internal.corp, prometheus.internal.corp | 443, 9090 | DC-West | Never |
| Contractor Portal | vendor.internal.corp | 443 | DC-East | Never |

## Access Policies

| Rule | Action | Users/Groups | Segments | Posture | Priority |
|------|--------|-------------|----------|---------|----------|
| HR Access | ALLOW | HR-Department | HR Applications | Corporate Managed | 1 |
| Finance Access | ALLOW | Finance-Team | Finance Tools | Corporate Managed | 2 |
| Engineering Access | ALLOW | Engineering, DevOps | Engineering | Developer Workstation | 3 |
| Monitoring Access | ALLOW | SRE-Team, Engineering | Monitoring | Corporate Managed | 4 |
| Contractor Access | ALLOW | Contractors | Contractor Portal | BYOD | 5 |
| Default Deny | DENY | All Users | All Segments | N/A | 99 |

## Device Posture Profiles

| Profile | CrowdStrike ZTA | OS Version | Encryption | Firewall |
|---------|----------------|------------|------------|----------|
| Corporate Managed | >= 60 | Win 10 21H2+ / macOS 13+ | Required | Required |
| Developer Workstation | >= 70 | Win 10 21H2+ / macOS 13+ | Required | Required |
| BYOD | N/A | Latest -1 | Recommended | N/A |

## Migration Tracker

| Phase | Users | Applications | Start | Status |
|-------|-------|-------------|-------|--------|
| Phase 1: IT/Security | 50 | All | 2026-02-01 | Complete |
| Phase 2: Engineering | 200 | Engineering, Monitoring | 2026-02-10 | Complete |
| Phase 3: Business | 250 | HR, Finance, Contractor | 2026-02-20 | In Progress |
| VPN Decommission | N/A | N/A | 2026-03-15 | Pending |

## Sign-Off

| Role | Name | Date | Approved |
|------|------|------|----------|
| CISO | _________________ | __________ | [ ] |
| Network Architect | _________________ | __________ | [ ] |
| IT Operations | _________________ | __________ | [ ] |

===== END FILE =====


===== FILE: skills/configuring-zscaler-private-access-for-ztna/references/api-reference.md =====
# Zscaler Private Access ZTNA — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| requests | `pip install requests` | ZPA REST API client |
| zscaler-sdk-python | `pip install zscaler-sdk-python` | Official Zscaler Python SDK |

## ZPA API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/signin` | Authenticate and get bearer token |
| GET | `/mgmtconfig/v1/admin/customers/{id}/application` | List app segments |
| GET | `/mgmtconfig/v1/admin/customers/{id}/connector` | List app connectors |
| GET | `/mgmtconfig/v1/admin/customers/{id}/serverGroup` | List server groups |
| GET | `/mgmtconfig/v1/admin/customers/{id}/policySet/rules` | List access policies |
| GET | `/mgmtconfig/v1/admin/customers/{id}/segmentGroup` | List segment groups |

## Key ZPA Concepts

| Component | Description |
|-----------|-------------|
| App Segment | Application definition with domain/IP and port ranges |
| App Connector | On-premise agent that brokers connections to apps |
| Server Group | Group of application servers behind connectors |
| Access Policy | Rules defining who can access which app segments |
| Segment Group | Logical grouping of app segments |

## External References

- [ZPA API Documentation](https://help.zscaler.com/zpa/api)
- [Zscaler SDK Python](https://github.com/zscaler/zscaler-sdk-python)
- [ZPA Admin Guide](https://help.zscaler.com/zpa)

===== END FILE =====


===== FILE: skills/configuring-zscaler-private-access-for-ztna/references/standards.md =====
# Zscaler Private Access ZTNA - Standards & References

## NIST SP 800-207: Zero Trust Architecture
- **Section 2.1**: Zero Trust Tenets - ZPA implements "never trust, always verify" with per-session application access
- **Section 3.1**: Policy Engine / Policy Administrator - maps to ZPA access policy engine
- **Section 3.3**: Agent/Gateway Model - maps to Zscaler Client Connector + App Connector architecture
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-207/final

## CISA Zero Trust Maturity Model v2.0
- **Network Pillar**: Application micro-segmentation - ZPA application segments eliminate network-level access
- **Identity Pillar**: Continuous validation - ZPA evaluates identity and posture per session
- **URL**: https://www.cisa.gov/zero-trust-maturity-model

## Zscaler Documentation
- **ZPA Admin Guide**: https://help.zscaler.com/zpa
- **Step-by-Step Configuration**: https://help.zscaler.com/zpa/step-step-configuration-guide-zpa
- **App Connector Deployment**: https://help.zscaler.com/zpa/about-app-connectors
- **Access Policy Configuration**: https://help.zscaler.com/zpa/configuring-access-policies
- **Browser Access**: https://help.zscaler.com/zpa/about-browser-access
- **Log Streaming Service**: https://help.zscaler.com/zpa/about-log-streaming-service
- **ZPA Reference Architecture (AWS)**: https://help.zscaler.com/downloads/zpa/reference-architecture

## NIST SP 800-63-3: Digital Identity Guidelines
- **Section 4-6**: Identity assurance levels relevant to ZPA IdP integration
- **URL**: https://pages.nist.gov/800-63-3/

## SOX Compliance (Sarbanes-Oxley)
- **Section 302/404**: Access controls for financial systems - ZPA provides auditable per-application access
- Application-level access logging satisfies audit trail requirements

## Gartner ZTNA Market Guide
- ZPA classified as agent-based ZTNA with service-initiated architecture
- Reference: Gartner Market Guide for Zero Trust Network Access (2024)

===== END FILE =====


===== FILE: skills/configuring-zscaler-private-access-for-ztna/references/workflows.md =====
# ZPA ZTNA Configuration Workflow

## Phase 1: Planning and Prerequisites (Week 1)

### 1.1 Architecture Design
1. Map current VPN access patterns: which users access which applications
2. Identify all internal applications by FQDN, IP, and port
3. Group applications into logical segments by business function and sensitivity
4. Design App Connector placement: minimum 2 connectors per data center for HA
5. Plan DNS resolution: App Connectors must resolve all application FQDNs

### 1.2 Identity Provider Integration
1. Configure SAML 2.0 or OIDC integration with corporate IdP (Okta, Entra ID, Ping)
2. Map IdP user groups to ZPA access policy groups
3. Configure SCIM provisioning for automated user/group sync
4. Test SSO authentication flow end-to-end

## Phase 2: Infrastructure Deployment (Week 2-3)

### 2.1 App Connector Deployment
1. Provision Linux VMs (CentOS/RHEL/Ubuntu) meeting minimum specs (2 vCPU, 4GB RAM)
2. Ensure outbound HTTPS (443) to ZPA cloud domains (*.private.zscaler.com, *.zpath.net)
3. Generate provisioning keys in ZPA Admin Portal (one per App Connector Group)
4. Install and configure App Connector packages
5. Verify connector enrollment and health status in portal
6. Deploy second connector per group for high availability

### 2.2 Server Group and Application Segment Configuration
1. Create server groups mapping to App Connector groups
2. Add application servers with FQDNs and ports to server groups
3. Create application segments grouping related applications
4. Configure health monitoring for each segment
5. Test application reachability from App Connector

## Phase 3: Policy Configuration (Week 3-4)

### 3.1 Access Policy Design
1. Create allow rules ordered from most specific to least specific
2. Map IdP groups to application segments in each rule
3. Add device posture profile requirements to rules
4. Configure time-based restrictions for contractor access
5. Create default deny rule as the last policy rule
6. Test each policy rule with representative users

### 3.2 Device Posture Configuration
1. Define posture profiles for each device category (managed, developer, BYOD)
2. Integrate CrowdStrike ZTA scores for real-time posture
3. Configure OS version, encryption, and firewall requirements
4. Test posture evaluation with compliant and non-compliant devices

## Phase 4: User Migration (Week 4-6)

### 4.1 Client Connector Deployment
1. Deploy Zscaler Client Connector via MDM (Intune, Jamf, SCCM)
2. Configure auto-enrollment with IdP authentication
3. Test client connectivity to ZPA cloud
4. Validate application access through Client Connector

### 4.2 Phased Rollout
1. Phase 1: IT/Security team (50 users) - validate all segments
2. Phase 2: Engineering (200 users) - validate developer tools
3. Phase 3: Business users (remaining) - validate general applications
4. Monitor access logs for denials and policy gaps at each phase

## Phase 5: Browser Access Configuration (Week 5)

1. Enable Browser Access for applications requiring clientless access
2. Configure custom domains and TLS certificates
3. Set session timeout and clipboard/file transfer policies
4. Test with contractor accounts

## Phase 6: VPN Decommission (Week 7-8)

1. Run VPN and ZPA in parallel for 2-3 weeks
2. Monitor VPN usage to identify remaining dependencies
3. Migrate remaining applications and users
4. Disable VPN and redirect users to ZPA
5. Decommission VPN infrastructure

===== END FILE =====


===== FILE: skills/configuring-zscaler-private-access-for-ztna/scripts/agent.py =====
#!/usr/bin/env python3
"""Zscaler Private Access (ZPA) ZTNA audit agent using ZPA API."""

import json
import sys
import argparse
from datetime import datetime

try:
    import requests
except ImportError:
    print("Install: pip install requests")
    sys.exit(1)


class ZPAClient:
    """Zscaler Private Access API client."""

    def __init__(self, client_id, client_secret, customer_id):
        self.base_url = "https://config.private.zscaler.com"
        self.customer_id = customer_id
        self.token = self._authenticate(client_id, client_secret)

    def _authenticate(self, client_id, client_secret):
        resp = requests.post(f"{self.base_url}/signin", json={
            "client_id": client_id, "client_secret": client_secret,
        }, timeout=30)
        resp.raise_for_status()
        return resp.json()["token"]

    def _get(self, endpoint):
        resp = requests.get(f"{self.base_url}/mgmtconfig/v1/admin/customers/{self.customer_id}/{endpoint}",
                            headers={"Authorization": f"Bearer {self.token}"}, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def list_app_segments(self):
        return self._get("application")

    def list_server_groups(self):
        return self._get("serverGroup")

    def list_app_connectors(self):
        return self._get("connector")

    def list_access_policies(self):
        return self._get("policySet/rules")

    def list_segment_groups(self):
        return self._get("segmentGroup")


def audit_zpa_config(client):
    """Audit ZPA configuration for security posture."""
    findings = []
    apps = client.list_app_segments()
    for app in apps.get("list", []):
        if not app.get("enabled"):
            findings.append({
                "type": "disabled_app_segment",
                "name": app.get("name", ""),
                "severity": "LOW",
            })
        if app.get("bypassType") == "ALWAYS":
            findings.append({
                "type": "bypass_enabled",
                "name": app.get("name", ""),
                "severity": "HIGH",
                "recommendation": "Remove bypass to enforce ZPA inspection",
            })
    connectors = client.list_app_connectors()
    for conn in connectors.get("list", []):
        if conn.get("runtimeStatus") != "running":
            findings.append({
                "type": "connector_down",
                "name": conn.get("name", ""),
                "status": conn.get("runtimeStatus", ""),
                "severity": "CRITICAL",
            })
    return findings


def run_audit(client_id, client_secret, customer_id):
    """Execute ZPA ZTNA audit."""
    client = ZPAClient(client_id, client_secret, customer_id)
    print(f"\n{'='*60}")
    print(f"  ZSCALER PRIVATE ACCESS ZTNA AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    apps = client.list_app_segments()
    app_list = apps.get("list", [])
    print(f"--- APPLICATION SEGMENTS ({len(app_list)}) ---")
    for a in app_list[:10]:
        print(f"  {a.get('name', '')}: enabled={a.get('enabled', '')} bypass={a.get('bypassType', '')}")

    connectors = client.list_app_connectors()
    conn_list = connectors.get("list", [])
    print(f"\n--- APP CONNECTORS ({len(conn_list)}) ---")
    for c in conn_list[:10]:
        print(f"  {c.get('name', '')}: {c.get('runtimeStatus', '')}")

    findings = audit_zpa_config(client)
    print(f"\n--- AUDIT FINDINGS ({len(findings)}) ---")
    for f in findings:
        print(f"  [{f['severity']}] {f['type']}: {f.get('name', '')}")

    return {"apps": len(app_list), "connectors": len(conn_list), "findings": findings}


def main():
    parser = argparse.ArgumentParser(description="ZPA ZTNA Audit Agent")
    parser.add_argument("--client-id", required=True, help="ZPA API client ID")
    parser.add_argument("--client-secret", required=True, help="ZPA API client secret")
    parser.add_argument("--customer-id", required=True, help="ZPA customer ID")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_audit(args.client_id, args.client_secret, args.customer_id)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/configuring-zscaler-private-access-for-ztna/scripts/process.py =====
#!/usr/bin/env python3
"""
Zscaler Private Access (ZPA) - Deployment Audit and Compliance Checker

Queries ZPA Admin API to audit App Connector health, application segment
coverage, access policy configuration, and user activity for ZTNA compliance.

Requirements:
    pip install requests
"""

import json
import sys
import time
from datetime import datetime, timezone
from typing import Any

import requests

ZPA_BASE_URL = "https://config.private.zscaler.com"


class ZPAAuditor:
    """Audit Zscaler Private Access deployment configuration."""

    def __init__(self, client_id: str, client_secret: str, customer_id: str):
        self.client_id = client_id
        self.client_secret = client_secret
        self.customer_id = customer_id
        self.token = None
        self._authenticate()

    def _authenticate(self):
        """Authenticate with ZPA API."""
        resp = requests.post(
            f"{ZPA_BASE_URL}/signin",
            json={
                "apiKey": self.client_id,
                "username": self.client_secret,
                "password": self.customer_id
            },
            timeout=30
        )
        resp.raise_for_status()
        self.token = resp.json().get("token")
        print("[AUTH] Successfully authenticated with ZPA API")

    def _get(self, endpoint: str, params: dict = None) -> dict:
        """Make authenticated GET request to ZPA API."""
        headers = {
            "Authorization": f"Bearer {self.token}",
            "Content-Type": "application/json"
        }
        resp = requests.get(
            f"{ZPA_BASE_URL}/mgmtconfig/v1/admin/customers/{self.customer_id}/{endpoint}",
            headers=headers,
            params=params or {},
            timeout=30
        )
        resp.raise_for_status()
        return resp.json()

    def audit_app_connectors(self) -> dict[str, Any]:
        """Audit App Connector deployment and health."""
        print("\n[1/5] Auditing App Connectors...")
        data = self._get("connector")
        connectors = data.get("list", [])

        stats = {
            "total": len(connectors),
            "healthy": 0,
            "unhealthy": 0,
            "groups": {},
            "version_distribution": {},
            "unhealthy_list": []
        }

        for conn in connectors:
            name = conn.get("name", "unknown")
            enabled = conn.get("enabled", False)
            runtime_status = conn.get("runtimeStatus", "UNKNOWN")
            version = conn.get("currentVersion", "unknown")
            group = conn.get("appConnectorGroupName", "ungrouped")

            stats["groups"][group] = stats["groups"].get(group, 0) + 1
            stats["version_distribution"][version] = stats["version_distribution"].get(version, 0) + 1

            if enabled and runtime_status == "ACTIVE":
                stats["healthy"] += 1
            else:
                stats["unhealthy"] += 1
                stats["unhealthy_list"].append({
                    "name": name, "status": runtime_status, "group": group
                })
                print(f"  [WARN] Unhealthy connector: {name} (status: {runtime_status})")

        print(f"  Total: {stats['total']}, Healthy: {stats['healthy']}, Unhealthy: {stats['unhealthy']}")
        print(f"  Groups: {stats['groups']}")

        # Check HA: each group should have >= 2 connectors
        for group, count in stats["groups"].items():
            if count < 2:
                print(f"  [CRITICAL] Group '{group}' has only {count} connector(s) - no HA!")

        return stats

    def audit_application_segments(self) -> dict[str, Any]:
        """Audit application segment configuration."""
        print("\n[2/5] Auditing Application Segments...")
        data = self._get("application")
        segments = data.get("list", [])

        stats = {
            "total": len(segments),
            "enabled": 0,
            "disabled": 0,
            "bypass_enabled": 0,
            "health_reporting": {"continuous": 0, "on_access": 0, "none": 0},
            "segments": []
        }

        for seg in segments:
            name = seg.get("name", "unknown")
            enabled = seg.get("enabled", False)
            bypass_type = seg.get("bypassType", "NEVER")
            health = seg.get("healthReporting", "NONE")
            domains = seg.get("domainNames", [])
            tcp_ports = seg.get("tcpPortRanges", [])

            if enabled:
                stats["enabled"] += 1
            else:
                stats["disabled"] += 1
                print(f"  [WARN] Disabled segment: {name}")

            if bypass_type != "NEVER":
                stats["bypass_enabled"] += 1
                print(f"  [WARN] Bypass enabled on segment: {name} (type: {bypass_type})")

            health_key = health.lower() if health.lower() in stats["health_reporting"] else "none"
            stats["health_reporting"][health_key] += 1

            stats["segments"].append({
                "name": name,
                "enabled": enabled,
                "domains": domains[:5],
                "ports": tcp_ports[:5],
                "bypass": bypass_type
            })

        print(f"  Total: {stats['total']}, Enabled: {stats['enabled']}, Disabled: {stats['disabled']}")
        print(f"  Bypass enabled: {stats['bypass_enabled']}")
        print(f"  Health reporting: {stats['health_reporting']}")
        return stats

    def audit_access_policies(self) -> dict[str, Any]:
        """Audit access policy configuration."""
        print("\n[3/5] Auditing Access Policies...")
        data = self._get("policySet/rules?policyType=ACCESS_POLICY")
        rules = data.get("list", [])

        stats = {
            "total": len(rules),
            "allow_rules": 0,
            "deny_rules": 0,
            "has_default_deny": False,
            "rules_without_conditions": 0,
            "rules_with_posture": 0,
            "rules": []
        }

        for rule in rules:
            name = rule.get("name", "unknown")
            action = rule.get("action", "UNKNOWN")
            conditions = rule.get("conditions", [])
            order = rule.get("order", 0)

            if action == "ALLOW":
                stats["allow_rules"] += 1
            elif action == "DENY":
                stats["deny_rules"] += 1

            if not conditions:
                stats["rules_without_conditions"] += 1
                print(f"  [WARN] Rule '{name}' has no conditions - overly permissive")

            # Check for device posture conditions
            has_posture = any(
                c.get("operands", [{}])[0].get("objectType") == "POSTURE_PROFILE"
                for c in conditions if c.get("operands")
            )
            if has_posture:
                stats["rules_with_posture"] += 1

            stats["rules"].append({
                "name": name, "action": action, "order": order,
                "conditions_count": len(conditions), "has_posture": has_posture
            })

        # Check for default deny rule (should be last rule with DENY action)
        if rules and rules[-1].get("action") == "DENY":
            stats["has_default_deny"] = True
        else:
            print("  [CRITICAL] No default deny rule found! All unlisted access may be permitted.")

        print(f"  Total rules: {stats['total']}")
        print(f"  Allow: {stats['allow_rules']}, Deny: {stats['deny_rules']}")
        print(f"  Rules with device posture: {stats['rules_with_posture']}")
        print(f"  Default deny: {'Yes' if stats['has_default_deny'] else 'NO - MISSING!'}")
        return stats

    def audit_server_groups(self) -> dict[str, Any]:
        """Audit server group configuration."""
        print("\n[4/5] Auditing Server Groups...")
        data = self._get("serverGroup")
        groups = data.get("list", [])

        stats = {
            "total": len(groups),
            "enabled": 0,
            "servers_total": 0,
            "groups": []
        }

        for group in groups:
            name = group.get("name", "unknown")
            enabled = group.get("enabled", False)
            servers = group.get("servers", [])

            if enabled:
                stats["enabled"] += 1
            stats["servers_total"] += len(servers)
            stats["groups"].append({
                "name": name, "enabled": enabled, "server_count": len(servers)
            })

        print(f"  Total groups: {stats['total']}, Enabled: {stats['enabled']}")
        print(f"  Total servers: {stats['servers_total']}")
        return stats

    def generate_report(self, connectors, segments, policies, server_groups) -> str:
        """Generate comprehensive ZPA audit report."""
        print("\n[5/5] Generating audit report...")
        now = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M UTC")

        report = f"""
ZPA ZTNA Deployment Audit Report
{'=' * 55}
Customer ID: {self.customer_id}
Generated: {now}

1. APP CONNECTORS
   Total deployed:              {connectors['total']}
   Healthy:                     {connectors['healthy']}
   Unhealthy:                   {connectors['unhealthy']}
   Connector groups:            {len(connectors['groups'])}
   HA compliance (2+ per group): {'PASS' if all(v >= 2 for v in connectors['groups'].values()) else 'FAIL'}

2. APPLICATION SEGMENTS
   Total segments:              {segments['total']}
   Enabled:                     {segments['enabled']}
   Disabled:                    {segments['disabled']}
   With bypass enabled:         {segments['bypass_enabled']}
   Health reporting:            {segments['health_reporting']}

3. ACCESS POLICIES
   Total rules:                 {policies['total']}
   Allow rules:                 {policies['allow_rules']}
   Deny rules:                  {policies['deny_rules']}
   Default deny rule:           {'YES' if policies['has_default_deny'] else 'MISSING - CRITICAL'}
   Rules with device posture:   {policies['rules_with_posture']} / {policies['total']}

4. SERVER GROUPS
   Total groups:                {server_groups['total']}
   Total servers:               {server_groups['servers_total']}

5. RECOMMENDATIONS
"""
        recommendations = []
        if connectors['unhealthy'] > 0:
            recommendations.append(f"   - Fix {connectors['unhealthy']} unhealthy App Connector(s)")
        if not policies['has_default_deny']:
            recommendations.append("   - ADD DEFAULT DENY RULE immediately")
        if policies['rules_with_posture'] < policies['allow_rules']:
            gap = policies['allow_rules'] - policies['rules_with_posture']
            recommendations.append(f"   - Add device posture to {gap} allow rule(s)")
        if segments['bypass_enabled'] > 0:
            recommendations.append(f"   - Review {segments['bypass_enabled']} segment(s) with bypass enabled")
        if not recommendations:
            recommendations.append("   - No critical issues found")

        report += "\n".join(recommendations)
        return report


def main():
    if len(sys.argv) < 4:
        print("Usage: python process.py <client_id> <client_secret> <customer_id>")
        print("\nAudits ZPA deployment for connector health, segment coverage, and policy compliance.")
        sys.exit(1)

    auditor = ZPAAuditor(sys.argv[1], sys.argv[2], sys.argv[3])

    connectors = auditor.audit_app_connectors()
    segments = auditor.audit_application_segments()
    policies = auditor.audit_access_policies()
    server_groups = auditor.audit_server_groups()

    report = auditor.generate_report(connectors, segments, policies, server_groups)
    print(report)

    filename = f"zpa_audit_{datetime.now().strftime('%Y%m%d')}.txt"
    with open(filename, "w") as f:
        f.write(report)
    print(f"\nReport saved to: {filename}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/deploying-cloudflare-access-for-zero-trust/SKILL.md =====
---
name: deploying-cloudflare-access-for-zero-trust
description: Deploys Cloudflare Access with Cloudflare Tunnel for zero trust access to self-hosted apps, configuring identity-aware policies, device posture checks, and WARP client enrollment as a VPN replacement. Use when replacing VPN with Cloudflare One, exposing internal apps without open inbound ports, or securing contractor/third-party access to specific applications.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- cloudflare
- cloudflare-access
- zero-trust
- cloudflare-tunnel
- warp
- ztna
- cloudflare-one
version: '1.0'
author: mahipal
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
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1133
- T1078
- T1190
- T1021
---

# Deploying Cloudflare Access for Zero Trust

## When to Use

- When replacing VPN infrastructure with identity-aware application access using Cloudflare One
- When exposing self-hosted internal applications through Cloudflare Tunnel without opening inbound ports
- When implementing ZTNA for a distributed workforce accessing web applications, SSH, and RDP services
- When needing a cost-effective zero trust solution with integrated DLP, CASB, and SWG capabilities
- When securing contractor and third-party access to specific applications without full network access

**Do not use** for applications requiring persistent UDP connections not supported by Cloudflare Tunnel, for environments requiring air-gapped or fully on-premises access control, or when regulatory requirements prohibit routing traffic through third-party cloud infrastructure.

## Prerequisites

- Cloudflare account with Zero Trust subscription (Free for up to 50 users, paid plans for larger teams)
- Domain name managed by Cloudflare DNS (or ability to add CNAME records)
- Linux, Windows, or macOS server to run `cloudflared` tunnel daemon
- Identity provider: Okta, Microsoft Entra ID, Google Workspace, GitHub, or any SAML/OIDC provider
- Cloudflare WARP client for device-level enrollment (optional but recommended)

## Workflow

### Step 1: Create a Cloudflare Tunnel to Internal Applications

Install `cloudflared` and create a persistent tunnel to expose internal services.

```bash
# Install cloudflared on Ubuntu/Debian
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb \
  -o cloudflared.deb
sudo dpkg -i cloudflared.deb

# Authenticate cloudflared with your Cloudflare account
cloudflared tunnel login

# Create a named tunnel
cloudflared tunnel create internal-apps
# Output: Created tunnel internal-apps with id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

# Configure tunnel routes to internal applications
cat > ~/.cloudflared/config.yml << 'EOF'
tunnel: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
credentials-file: /home/admin/.cloudflared/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json

ingress:
  - hostname: wiki.company.com
    service: http://localhost:8080
  - hostname: git.company.com
    service: http://10.1.1.50:3000
  - hostname: grafana.company.com
    service: http://10.1.1.60:3000
  - hostname: ssh.company.com
    service: ssh://localhost:22
  - hostname: rdp.company.com
    service: rdp://10.1.1.100:3389
  # Catch-all rule (required)
  - service: http_status:404
EOF

# Route DNS to the tunnel
cloudflared tunnel route dns internal-apps wiki.company.com
cloudflared tunnel route dns internal-apps git.company.com
cloudflared tunnel route dns internal-apps grafana.company.com

# Run tunnel as a systemd service
sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared

# Verify tunnel status
cloudflared tunnel info internal-apps
```

### Step 2: Configure Identity Provider Integration

Set up authentication with your organization's identity provider.

```bash
# Using Cloudflare API to configure Okta as IdP
curl -X PUT "https://api.cloudflare.com/client/v4/accounts/{account_id}/access/identity_providers" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Corporate Okta",
    "type": "okta",
    "config": {
      "client_id": "OKTA_CLIENT_ID",
      "client_secret": "OKTA_CLIENT_SECRET",
      "okta_account": "company.okta.com",
      "api_token": "OKTA_API_TOKEN",
      "claims": ["email", "groups", "name"],
      "email_claim_name": "email"
    }
  }'

# Configure Microsoft Entra ID as additional IdP
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/access/identity_providers" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Microsoft Entra ID",
    "type": "azureAD",
    "config": {
      "client_id": "AZURE_APP_CLIENT_ID",
      "client_secret": "AZURE_APP_CLIENT_SECRET",
      "directory_id": "AZURE_TENANT_ID",
      "support_groups": true,
      "claims": ["email", "groups", "name"]
    }
  }'
```

### Step 3: Create Access Applications and Policies

Define Access applications with identity-aware policies for each internal service.

```bash
# Create Access application for internal wiki
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/access/apps" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Internal Wiki",
    "domain": "wiki.company.com",
    "type": "self_hosted",
    "session_duration": "8h",
    "auto_redirect_to_identity": true,
    "http_only_cookie_attribute": true,
    "same_site_cookie_attribute": "lax",
    "logo_url": "https://company.com/wiki-logo.png",
    "allowed_idps": ["OKTA_IDP_ID", "AZURE_IDP_ID"]
  }'

# Create Allow policy for the wiki application
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/access/apps/{app_id}/policies" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Allow Engineering Team",
    "decision": "allow",
    "precedence": 1,
    "include": [
      {"group": {"id": "ENGINEERING_GROUP_ID"}},
      {"okta": {"name": "Engineering", "identity_provider_id": "OKTA_IDP_ID"}}
    ],
    "require": [
      {"device_posture": {"integration_uid": "CROWDSTRIKE_INTEGRATION_ID"}}
    ]
  }'

# Create Access application for SSH access
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/access/apps" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "SSH Access",
    "domain": "ssh.company.com",
    "type": "ssh",
    "session_duration": "4h",
    "auto_redirect_to_identity": true
  }'
```

### Step 4: Deploy WARP Client for Device Enrollment

Enroll corporate devices using Cloudflare WARP for private network access and device posture.

```bash
# Create device enrollment rule
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/devices/policy" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Corporate Device Enrollment",
    "match": "identity.email matches \".*@company\\.com$\"",
    "precedence": 100,
    "enabled": true,
    "gateway_unique_id": "GATEWAY_ID",
    "support_url": "https://helpdesk.company.com/warp-help"
  }'

# Install WARP on macOS via MDM (Jamf/Intune)
# Download: https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/download-warp/
# Deploy with MDM configuration profile:
cat > warp_mdm_config.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>organization</key>
    <string>company</string>
    <key>auto_connect</key>
    <integer>1</integer>
    <key>switch_locked</key>
    <true/>
    <key>onboarding</key>
    <false/>
</dict>
</plist>
EOF

# Install Cloudflare root certificate for TLS inspection
# Download from: https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/user-side-certificates/
sudo cp cloudflare-root-ca.pem /usr/local/share/ca-certificates/cloudflare-root-ca.crt
sudo update-ca-certificates

# Configure split tunnel to route private network through WARP
curl -X PUT "https://api.cloudflare.com/client/v4/accounts/{account_id}/devices/policy/{policy_id}/fallback_domains" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '[
    {"suffix": "internal.corp", "description": "Internal corporate domain"},
    {"suffix": "10.0.0.0/8", "description": "Private network range"}
  ]'
```

### Step 5: Configure Device Posture Checks

Integrate endpoint security signals into Access policies.

```bash
# Add CrowdStrike device posture integration
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/devices/posture/integration" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "CrowdStrike Falcon",
    "type": "crowdstrike_s2s",
    "config": {
      "api_url": "https://api.crowdstrike.com",
      "client_id": "CS_API_CLIENT_ID",
      "client_secret": "CS_API_CLIENT_SECRET",
      "customer_id": "CS_CUSTOMER_ID"
    },
    "interval": "10m"
  }'

# Create device posture rule for disk encryption
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/devices/posture" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Disk Encryption Required",
    "type": "disk_encryption",
    "match": [{"platform": "windows"}, {"platform": "mac"}],
    "input": {"requireAll": true}
  }'

# Create device posture rule for OS version
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/devices/posture" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Minimum OS Version",
    "type": "os_version",
    "match": [{"platform": "windows"}],
    "input": {"version": "10.0.19045", "operator": ">="}
  }'
```

### Step 6: Set Up Audit Logging and Analytics

Configure logging for access decisions and tunnel health monitoring.

```bash
# Enable Logpush for Access audit logs to S3
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/logpush/jobs" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "access-audit-logs",
    "output_options": {
      "field_names": ["RayID","Action","Allowed","AppDomain","AppUUID","Connection","Country","CreatedAt","Email","IPAddress","PurposeJustificationPrompt","PurposeJustificationResponse","TemporaryAccessDuration","UserUID"],
      "timestamp_format": "rfc3339"
    },
    "destination_conf": "s3://security-logs-bucket/cloudflare-access/?region=us-east-1&access-key-id=AKID&secret-access-key=SECRET",
    "dataset": "access_requests",
    "enabled": true
  }'

# Query access logs via GraphQL Analytics API
curl -X POST "https://api.cloudflare.com/client/v4/graphql" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "query": "{ viewer { accounts(filter: {accountTag: \"ACCOUNT_ID\"}) { accessLoginRequestsAdaptiveGroups(filter: {datetime_gt: \"2026-02-22T00:00:00Z\"}, limit: 100, orderBy: [count_DESC]) { dimensions { action appName userEmail country } count } } } }"
  }'
```

## Key Concepts

| Term | Definition |
|------|------------|
| Cloudflare Tunnel | Encrypted outbound-only connection from your infrastructure to Cloudflare's network, exposing internal services without opening inbound firewall ports |
| Cloudflare Access | Identity-aware reverse proxy evaluating every request against access policies before granting access to protected applications |
| WARP Client | Cloudflare's endpoint agent that routes device traffic through Cloudflare's network for policy enforcement and private network access |
| Access Application | Configuration object defining a protected resource (self-hosted, SaaS, or infrastructure) with associated access policies |
| Device Posture | Endpoint health signals (OS version, disk encryption, EDR status) evaluated as conditions in Access policies |
| Cloudflare One | Unified SASE platform combining ZTNA (Access), SWG (Gateway), CASB, DLP, and RBI |

## Tools & Systems

- **Cloudflare Access**: Identity-aware application proxy providing per-request authorization
- **Cloudflare Tunnel (cloudflared)**: Daemon creating encrypted tunnels from internal networks to Cloudflare edge
- **WARP Client**: Cross-platform endpoint agent for device enrollment, DNS filtering, and private network routing
- **Cloudflare Gateway**: Secure Web Gateway providing DNS/HTTP filtering and DLP inspection
- **Cloudflare Logpush**: Real-time log streaming to external SIEM and storage destinations
- **Access for Infrastructure**: SSH and RDP access with short-lived certificates and session recording

## Common Scenarios

### Scenario: Startup with 200 Employees Deploying Zero Trust from Scratch

**Context**: A SaaS startup with 200 employees and no existing VPN wants to provide secure access to internal tools (Grafana, internal APIs, staging environments) running on AWS. Budget is limited, and the team has no dedicated security staff.

**Approach**:
1. Start with Cloudflare Zero Trust free tier (up to 50 users) for proof of concept
2. Deploy one `cloudflared` tunnel on an EC2 instance in the production VPC
3. Expose Grafana, internal wiki, and staging apps through tunnel with DNS routing
4. Configure Google Workspace as IdP for SSO authentication
5. Create Access policies requiring @company.com email domain for all applications
6. Add device posture checks for disk encryption and OS version
7. Upgrade to paid plan and deploy WARP client to all employee laptops via MDM
8. Enable Gateway DNS filtering and HTTP inspection for malware protection
9. Configure Logpush to send access logs to Datadog for monitoring

**Pitfalls**: Cloudflare root certificate must be installed on all devices for TLS inspection to work; some applications may break with TLS interception. Tunnel failover requires running multiple `cloudflared` instances or using Cloudflare's replicas feature. Access policies should always include a default deny rule. WebSocket applications may require specific tunnel configuration.

## Output Format

```
Cloudflare Zero Trust Deployment Report
==================================================
Organization: StartupCorp
Team Name: startupcorp
Deployment Date: 2026-02-23

TUNNEL INFRASTRUCTURE:
  Active Tunnels: 2 (primary + failover)
  Tunnel Status: Healthy
  Connected Edge: Washington DC, Ashburn
  Ingress Routes: 8

ACCESS APPLICATIONS:
  Self-Hosted Apps: 6
  SaaS Apps: 3
  SSH/Infrastructure: 2
  Total Policies: 15

DEVICE ENROLLMENT:
  Enrolled Devices: 187 / 200
  WARP Connected: 182 / 187 (97.3%)
  Posture Compliant: 175 / 187 (93.6%)

ACCESS METRICS (last 30 days):
  Total Requests: 89,432
  Allowed: 88,756 (99.2%)
  Blocked: 676 (0.8%)
  Unique Users: 195
  Countries: 12
  Avg Session Duration: 6.2 hours
```

===== END FILE =====


===== FILE: skills/deploying-cloudflare-access-for-zero-trust/LICENSE =====

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


===== FILE: skills/deploying-cloudflare-access-for-zero-trust/assets/template.md =====
# Cloudflare Access Zero Trust - Deployment Checklist

## Project Information
| Field | Value |
|-------|-------|
| Organization | TechStartup Inc |
| Cloudflare Account | CF-XXXXXXX |
| Team Name | techstartup |
| Plan | Zero Trust Teams (50 seats) |
| Start Date | 2026-01-25 |

## Identity Provider Configuration
- [x] Primary IdP: Google Workspace (OIDC)
- [x] MFA enforced at IdP level
- [ ] Secondary IdP for contractors: GitHub OAuth
- [x] Email domain restriction: @techstartup.com

## Tunnel Deployment

| Tunnel Name | Server | Network | Routes | Status |
|-------------|--------|---------|--------|--------|
| prod-tunnel | prod-bastion (10.1.0.5) | Production VPC | wiki, grafana, api | Healthy |
| staging-tunnel | staging-bastion (10.2.0.5) | Staging VPC | staging-* | Healthy |

## Access Applications

| Application | Type | Domain | Session | Policies | Status |
|-------------|------|--------|---------|----------|--------|
| Internal Wiki | self_hosted | wiki.techstartup.com | 8h | 2 (Allow Eng, Deny All) | Active |
| Grafana | self_hosted | grafana.techstartup.com | 8h | 2 (Allow SRE, Deny All) | Active |
| Internal API | self_hosted | api.techstartup.com | 4h | 3 (Allow Backend, Service Token, Deny All) | Active |
| Staging Apps | self_hosted | staging.techstartup.com | 4h | 2 (Allow Eng, Deny All) | Active |
| SSH Jump | ssh | ssh.techstartup.com | 2h | 1 (Allow SRE) | Active |

## Device Posture Rules
- [x] Disk encryption: Required (Windows BitLocker, macOS FileVault)
- [x] OS version: Windows >= 10.0.19045, macOS >= 14.0
- [x] Firewall: Enabled
- [ ] CrowdStrike integration: Pending deployment

## WARP Enrollment

| Platform | Enrolled | Total | Coverage |
|----------|----------|-------|----------|
| macOS | 32 | 35 | 91.4% |
| Windows | 12 | 13 | 92.3% |
| Linux | 2 | 2 | 100% |
| Total | 46 | 50 | 92.0% |

## Sign-Off

| Role | Name | Date | Approved |
|------|------|------|----------|
| CTO | _________________ | __________ | [ ] |
| Security Lead | _________________ | __________ | [ ] |

===== END FILE =====


===== FILE: skills/deploying-cloudflare-access-for-zero-trust/references/api-reference.md =====
# Cloudflare Access Zero Trust — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| requests | `pip install requests` | Cloudflare API v4 client |

## Cloudflare Access API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/accounts/{id}/access/apps` | List Access applications |
| GET | `/accounts/{id}/access/apps/{id}/policies` | List app policies |
| GET | `/accounts/{id}/access/groups` | List Access groups |
| GET | `/accounts/{id}/access/identity_providers` | List IdP configs |
| GET | `/accounts/{id}/access/service_tokens` | List service tokens |
| POST | `/accounts/{id}/access/apps` | Create application |
| PUT | `/accounts/{id}/access/apps/{id}` | Update application |

## Authentication

```python
headers = {
    "Authorization": "Bearer <api_token>",
    "Content-Type": "application/json"
}
```

## Access Policy Rule Types

| Rule | Description |
|------|-------------|
| `include` | Must match (OR within group) |
| `exclude` | Must not match |
| `require` | Must match (AND) |

## External References

- [Cloudflare Access API](https://developers.cloudflare.com/api/operations/access-applications-list-access-applications)
- [Cloudflare Zero Trust Docs](https://developers.cloudflare.com/cloudflare-one/)
- [cloudflare Python SDK](https://github.com/cloudflare/cloudflare-python)

===== END FILE =====


===== FILE: skills/deploying-cloudflare-access-for-zero-trust/references/standards.md =====
# Cloudflare Access Zero Trust - Standards & References

## NIST SP 800-207: Zero Trust Architecture
- **Section 2**: ZTA Tenets - Cloudflare Access implements per-request identity verification
- **Section 3.2**: Enhanced Identity Governance - Access policies enforce continuous authorization
- **Section 4.2**: Cloud-Based SDP - Cloudflare Tunnel maps to software-defined perimeter architecture
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-207/final

## CISA Zero Trust Maturity Model v2.0
- **Identity**: SSO with MFA via integrated IdP support
- **Device**: WARP client posture checks for OS, encryption, EDR
- **Network**: Cloudflare Tunnel eliminates inbound firewall rules
- **Application**: Per-application Access policies with session controls
- **URL**: https://www.cisa.gov/zero-trust-maturity-model

## Cloudflare Documentation
- **Cloudflare One Overview**: https://developers.cloudflare.com/cloudflare-one/
- **Cloudflare Access**: https://developers.cloudflare.com/cloudflare-one/policies/access/
- **Cloudflare Tunnel**: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/
- **WARP Deployment**: https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/
- **Device Posture**: https://developers.cloudflare.com/cloudflare-one/identity/devices/
- **Logpush**: https://developers.cloudflare.com/logs/about/
- **Zero Trust Reference Architecture**: https://developers.cloudflare.com/reference-architecture/implementation-guides/zero-trust/

## SOC 2 Type II
- Cloudflare maintains SOC 2 Type II certification
- Access audit logs provide evidence for CC6.1 (Logical Access), CC6.3 (Role-Based Access)
- **URL**: https://www.cloudflare.com/trust-hub/compliance-resources/

## GDPR
- Cloudflare processes data per EU GDPR requirements
- Data localization options for EU-based traffic
- **URL**: https://www.cloudflare.com/trust-hub/gdpr/

===== END FILE =====


===== FILE: skills/deploying-cloudflare-access-for-zero-trust/references/workflows.md =====
# Cloudflare Access Zero Trust Deployment Workflow

## Phase 1: Account Setup (Day 1)

1. Create Cloudflare account and navigate to Zero Trust dashboard
2. Select team name (organization identifier for WARP enrollment)
3. Choose subscription plan based on user count
4. Configure authentication: add primary IdP (Okta, Entra ID, Google Workspace)
5. Add secondary IdP for contractors or partners if needed
6. Enable MFA requirements at the IdP level

## Phase 2: Tunnel Deployment (Day 2-3)

### 2.1 Install cloudflared
1. Install `cloudflared` on a server within the private network
2. Authenticate with `cloudflared tunnel login`
3. Create named tunnel: `cloudflared tunnel create <name>`
4. Configure ingress rules in `config.yml` mapping hostnames to internal services
5. Route DNS: `cloudflared tunnel route dns <tunnel> <hostname>`

### 2.2 High Availability
1. Deploy multiple `cloudflared` instances for redundancy
2. Use `cloudflared tunnel run --protocol quic` for better performance
3. Configure systemd service for automatic restart
4. Monitor tunnel health via Cloudflare dashboard

### 2.3 Private Network Routing
1. Add private network routes: `cloudflared tunnel route ip add 10.0.0.0/8 <tunnel-id>`
2. Configure split tunnel in WARP device settings
3. Set up DNS fallback domains for private DNS resolution

## Phase 3: Access Application Configuration (Day 4-5)

1. Create Access applications for each internal service
2. Define access policies per application:
   - Include rules: email domains, IdP groups, service tokens
   - Require rules: device posture, country restrictions
   - Exclude rules: specific users or IPs
3. Configure session duration per application sensitivity
4. Enable purpose justification for sensitive applications
5. Test access with pilot users

## Phase 4: WARP Client Deployment (Week 2)

1. Create device enrollment policies with email domain restrictions
2. Deploy WARP client via MDM (Intune, Jamf, SCCM)
3. Install Cloudflare root certificate for TLS inspection
4. Configure split tunnel settings for private network access
5. Enable device posture checks: OS version, disk encryption, firewall

## Phase 5: Gateway and DLP Configuration (Week 3)

1. Enable DNS filtering with block categories (malware, phishing)
2. Configure HTTP inspection policies
3. Set up DLP profiles for sensitive data detection
4. Enable browser isolation for high-risk web categories
5. Configure CASB for SaaS application monitoring

## Phase 6: Monitoring and Optimization (Ongoing)

1. Enable Logpush to SIEM (S3, Splunk, Datadog)
2. Monitor Access audit logs for denied requests
3. Review tunnel health metrics
4. Optimize split tunnel configuration
5. Conduct quarterly access policy reviews

===== END FILE =====


===== FILE: skills/deploying-cloudflare-access-for-zero-trust/scripts/agent.py =====
#!/usr/bin/env python3
"""Cloudflare Access zero trust audit agent using Cloudflare API."""

import json
import sys
import argparse
from datetime import datetime

try:
    import requests
except ImportError:
    print("Install: pip install requests")
    sys.exit(1)


class CloudflareAccessClient:
    """Cloudflare Access API client."""

    def __init__(self, api_token, account_id):
        self.base = f"https://api.cloudflare.com/client/v4/accounts/{account_id}/access"
        self.headers = {"Authorization": f"Bearer {api_token}", "Content-Type": "application/json"}

    def _get(self, endpoint):
        resp = requests.get(f"{self.base}/{endpoint}", headers=self.headers, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def list_applications(self):
        return self._get("apps")

    def list_policies(self, app_id):
        return self._get(f"apps/{app_id}/policies")

    def list_groups(self):
        return self._get("groups")

    def list_identity_providers(self):
        return self._get("identity_providers")

    def list_service_tokens(self):
        return self._get("service_tokens")


def audit_access_config(client):
    """Audit Cloudflare Access configuration."""
    findings = []
    apps = client.list_applications()
    for app in apps.get("result", []):
        if not app.get("session_duration"):
            findings.append({
                "type": "no_session_timeout",
                "app": app.get("name", ""),
                "severity": "MEDIUM",
            })
    tokens = client.list_service_tokens()
    for token in tokens.get("result", []):
        if token.get("expires_at"):
            expiry = datetime.fromisoformat(token["expires_at"].replace("Z", "+00:00"))
            if expiry.replace(tzinfo=None) < datetime.utcnow():
                findings.append({
                    "type": "expired_service_token",
                    "token_name": token.get("name", ""),
                    "severity": "HIGH",
                })
    return findings


def run_audit(api_token, account_id):
    """Execute Cloudflare Access audit."""
    client = CloudflareAccessClient(api_token, account_id)
    print(f"\n{'='*60}")
    print(f"  CLOUDFLARE ACCESS ZERO TRUST AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    apps = client.list_applications()
    app_list = apps.get("result", [])
    print(f"--- APPLICATIONS ({len(app_list)}) ---")
    for a in app_list[:10]:
        print(f"  {a.get('name', '')}: domain={a.get('domain', '')} type={a.get('type', '')}")

    idps = client.list_identity_providers()
    idp_list = idps.get("result", [])
    print(f"\n--- IDENTITY PROVIDERS ({len(idp_list)}) ---")
    for idp in idp_list:
        print(f"  {idp.get('name', '')}: type={idp.get('type', '')}")

    findings = audit_access_config(client)
    print(f"\n--- FINDINGS ({len(findings)}) ---")
    for f in findings:
        print(f"  [{f['severity']}] {f['type']}: {f.get('app', f.get('token_name', ''))}")

    return {"apps": len(app_list), "idps": len(idp_list), "findings": findings}


def main():
    parser = argparse.ArgumentParser(description="Cloudflare Access Audit Agent")
    parser.add_argument("--api-token", required=True, help="Cloudflare API token")
    parser.add_argument("--account-id", required=True, help="Cloudflare account ID")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_audit(args.api_token, args.account_id)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/deploying-cloudflare-access-for-zero-trust/scripts/process.py =====
#!/usr/bin/env python3
"""
Cloudflare Access Zero Trust - Deployment Audit Tool

Queries Cloudflare API to audit Access applications, policies, tunnel
health, and device enrollment for zero trust compliance validation.

Requirements:
    pip install requests
"""

import json
import sys
from datetime import datetime, timezone
from typing import Any

import requests

CF_API_BASE = "https://api.cloudflare.com/client/v4"


class CloudflareAccessAuditor:
    """Audit Cloudflare Zero Trust Access deployment."""

    def __init__(self, api_token: str, account_id: str):
        self.api_token = api_token
        self.account_id = account_id
        self.headers = {
            "Authorization": f"Bearer {api_token}",
            "Content-Type": "application/json"
        }

    def _get(self, endpoint: str, params: dict = None) -> dict:
        """Make authenticated GET request."""
        url = f"{CF_API_BASE}/accounts/{self.account_id}/{endpoint}"
        resp = requests.get(url, headers=self.headers, params=params or {}, timeout=30)
        resp.raise_for_status()
        return resp.json()

    def audit_access_applications(self) -> dict[str, Any]:
        """Audit all Access applications and their configurations."""
        print("\n[1/5] Auditing Access Applications...")
        data = self._get("access/apps")
        apps = data.get("result", [])

        stats = {
            "total": len(apps),
            "self_hosted": 0,
            "saas": 0,
            "ssh": 0,
            "vnc": 0,
            "without_policies": 0,
            "session_durations": {},
            "apps": []
        }

        for app in apps:
            app_type = app.get("type", "unknown")
            name = app.get("name", "unknown")
            domain = app.get("domain", "N/A")
            session = app.get("session_duration", "24h")
            policies_count = len(app.get("policies", []))

            if app_type == "self_hosted":
                stats["self_hosted"] += 1
            elif app_type == "saas":
                stats["saas"] += 1
            elif app_type == "ssh":
                stats["ssh"] += 1
            elif app_type == "vnc":
                stats["vnc"] += 1

            if policies_count == 0:
                stats["without_policies"] += 1
                print(f"  [WARN] App '{name}' has no access policies!")

            stats["session_durations"][session] = stats["session_durations"].get(session, 0) + 1
            stats["apps"].append({
                "name": name, "type": app_type, "domain": domain,
                "session": session, "policies": policies_count
            })
            print(f"  [{app_type.upper()}] {name} ({domain}) - {policies_count} policies, session: {session}")

        return stats

    def audit_tunnels(self) -> dict[str, Any]:
        """Audit Cloudflare Tunnel health and configuration."""
        print("\n[2/5] Auditing Cloudflare Tunnels...")
        data = self._get("cfd_tunnel", params={"is_deleted": "false"})
        tunnels = data.get("result", [])

        stats = {
            "total": len(tunnels),
            "healthy": 0,
            "degraded": 0,
            "inactive": 0,
            "tunnels": []
        }

        for tunnel in tunnels:
            name = tunnel.get("name", "unknown")
            status = tunnel.get("status", "unknown")
            connections = tunnel.get("connections", [])
            created = tunnel.get("created_at", "")

            if status == "healthy":
                stats["healthy"] += 1
            elif status == "degraded":
                stats["degraded"] += 1
                print(f"  [WARN] Tunnel '{name}' is degraded")
            else:
                stats["inactive"] += 1
                print(f"  [WARN] Tunnel '{name}' is inactive")

            stats["tunnels"].append({
                "name": name, "status": status,
                "connections": len(connections), "created": created
            })

        print(f"  Total: {stats['total']}, Healthy: {stats['healthy']}, "
              f"Degraded: {stats['degraded']}, Inactive: {stats['inactive']}")
        return stats

    def audit_device_posture(self) -> dict[str, Any]:
        """Audit device posture rules configuration."""
        print("\n[3/5] Auditing Device Posture Rules...")
        data = self._get("devices/posture")
        rules = data.get("result", [])

        stats = {
            "total": len(rules),
            "types": {},
            "rules": []
        }

        for rule in rules:
            name = rule.get("name", "unknown")
            rule_type = rule.get("type", "unknown")
            stats["types"][rule_type] = stats["types"].get(rule_type, 0) + 1
            stats["rules"].append({"name": name, "type": rule_type})
            print(f"  [{rule_type}] {name}")

        required_types = {"disk_encryption", "os_version", "firewall"}
        missing = required_types - set(stats["types"].keys())
        if missing:
            print(f"  [WARN] Missing recommended posture types: {missing}")

        return stats

    def audit_device_enrollment(self) -> dict[str, Any]:
        """Audit enrolled devices."""
        print("\n[4/5] Auditing Device Enrollment...")
        data = self._get("devices")
        devices = data.get("result", [])

        stats = {
            "total": len(devices),
            "os_distribution": {},
            "active": 0,
            "revoked": 0
        }

        for device in devices:
            os_type = device.get("os_version", "unknown").split(" ")[0] if device.get("os_version") else "unknown"
            stats["os_distribution"][os_type] = stats["os_distribution"].get(os_type, 0) + 1
            if device.get("revoked_at"):
                stats["revoked"] += 1
            else:
                stats["active"] += 1

        print(f"  Total: {stats['total']}, Active: {stats['active']}, Revoked: {stats['revoked']}")
        print(f"  OS Distribution: {stats['os_distribution']}")
        return stats

    def generate_report(self, apps, tunnels, posture, devices) -> str:
        """Generate comprehensive audit report."""
        print("\n[5/5] Generating report...")
        now = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M UTC")

        report = f"""
Cloudflare Zero Trust Access Audit Report
{'=' * 55}
Account: {self.account_id}
Generated: {now}

1. ACCESS APPLICATIONS
   Total applications:         {apps['total']}
   Self-hosted:                {apps['self_hosted']}
   SaaS:                       {apps['saas']}
   SSH/Infrastructure:         {apps['ssh']}
   Without policies:           {apps['without_policies']}
   Session durations:          {apps['session_durations']}

2. TUNNEL INFRASTRUCTURE
   Total tunnels:              {tunnels['total']}
   Healthy:                    {tunnels['healthy']}
   Degraded:                   {tunnels['degraded']}
   Inactive:                   {tunnels['inactive']}

3. DEVICE POSTURE
   Posture rules defined:      {posture['total']}
   Rule types:                 {posture['types']}

4. DEVICE ENROLLMENT
   Total devices:              {devices['total']}
   Active:                     {devices['active']}
   Revoked:                    {devices['revoked']}

5. RECOMMENDATIONS
"""
        recs = []
        if apps['without_policies'] > 0:
            recs.append(f"   - {apps['without_policies']} app(s) without policies - add access rules immediately")
        if tunnels['degraded'] > 0 or tunnels['inactive'] > 0:
            recs.append(f"   - {tunnels['degraded'] + tunnels['inactive']} tunnel(s) need attention")
        if "disk_encryption" not in posture.get("types", {}):
            recs.append("   - Add disk encryption posture rule")
        if "os_version" not in posture.get("types", {}):
            recs.append("   - Add OS version posture rule")
        if not recs:
            recs.append("   - No critical issues found")
        report += "\n".join(recs)
        return report


def main():
    if len(sys.argv) < 3:
        print("Usage: python process.py <cf_api_token> <account_id>")
        sys.exit(1)

    auditor = CloudflareAccessAuditor(sys.argv[1], sys.argv[2])
    apps = auditor.audit_access_applications()
    tunnels = auditor.audit_tunnels()
    posture = auditor.audit_device_posture()
    devices = auditor.audit_device_enrollment()
    report = auditor.generate_report(apps, tunnels, posture, devices)
    print(report)

    filename = f"cloudflare_zt_audit_{datetime.now().strftime('%Y%m%d')}.txt"
    with open(filename, "w") as f:
        f.write(report)
    print(f"\nReport saved to: {filename}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/deploying-palo-alto-prisma-access-zero-trust/SKILL.md =====
---
name: deploying-palo-alto-prisma-access-zero-trust
description: Deploys Palo Alto Networks Prisma Access for SASE-based zero trust network access, configuring GlobalProtect agents, ZTNA Connectors, security policy enforcement, and Strata Cloud Manager integration for unified management. Use when implementing enterprise-grade SASE with integrated ZTNA/SWG/CASB/FWaaS, replacing both VPN and branch firewalls with cloud-delivered security, or integrating ZTNA with an existing Palo Alto NGFW estate.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- prisma-access
- palo-alto
- ztna
- sase
- globalprotect
- strata-cloud-manager
- zero-trust
version: '1.0'
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- GOVERN-1.1
- MEASURE-2.7
- MANAGE-3.1
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1133
- T1078
- T1071.001
- T1572
---

# Deploying Palo Alto Prisma Access Zero Trust

## When to Use

- When implementing enterprise-grade SASE with integrated ZTNA, SWG, CASB, and FWaaS
- When replacing both VPN and branch office firewalls with cloud-delivered security
- When needing advanced threat prevention (WildFire, DNS Security) for remote access traffic
- When deploying zero trust for both mobile users and remote network (branch) connections
- When integrating ZTNA with existing Palo Alto NGFW infrastructure via Strata Cloud Manager

**Do not use** for small organizations (< 200 users) where simpler ZTNA solutions suffice, for environments requiring only web application access without full network security, or when budget constraints preclude enterprise SASE licensing.

## Prerequisites

- Prisma Access license (Business Premium or equivalent)
- Strata Cloud Manager (SCM) tenant configured
- GlobalProtect agent for endpoint deployment
- ZTNA Connector VM: 4 vCPU, 8GB RAM, 128GB disk (VMware, AWS, Azure, or GCP)
- Identity provider: Okta, Entra ID, Ping Identity (SAML 2.0)
- Palo Alto Cortex Data Lake for log storage

## Workflow

### Step 1: Configure Prisma Access Infrastructure in Strata Cloud Manager

Set up the cloud infrastructure for mobile user and remote network connections.

```text
Strata Cloud Manager > Prisma Access > Infrastructure Settings:

Mobile Users Configuration:
  - Service Connection: Auto-selected based on user location
  - DNS Servers: 10.1.1.10, 10.1.1.11 (corporate DNS)
  - IP Pool for Mobile Users: 10.100.0.0/16
  - Authentication: SAML with Okta (Primary), Entra ID (Secondary)
  - GlobalProtect Portal: portal.company.com
  - GlobalProtect Gateway: Auto (nearest Prisma Access location)

Infrastructure Subnet:
  - Range: 172.16.0.0/16
  - Allocation: /24 per Prisma Access location
```

### Step 2: Deploy ZTNA Connectors for Private Application Access

Install ZTNA Connectors to provide secure access to internal applications.

```bash
# Deploy ZTNA Connector on VMware (OVA)
# Download OVA from Strata Cloud Manager > Prisma Access > ZTNA Connectors

# AWS deployment via CloudFormation
aws cloudformation create-stack \
  --stack-name prisma-ztna-connector \
  --template-url https://prisma-access-connector-templates.s3.amazonaws.com/ztna-connector-aws.yaml \
  --parameters \
    ParameterKey=VpcId,ParameterValue=vpc-PROD \
    ParameterKey=SubnetId,ParameterValue=subnet-PRIVATE \
    ParameterKey=InstanceType,ParameterValue=m5.xlarge \
    ParameterKey=TenantServiceGroup,ParameterValue=TSG_ID \
    ParameterKey=ConnectorName,ParameterValue=dc-east-connector-01

# Verify connector registration
# Strata Cloud Manager > Prisma Access > ZTNA Connectors
# Status should show "Connected" with nearest Prisma Access location

# Deploy second connector for HA
# ZTNA Connector auto-discovers nearest Prisma Access location
# IPSec tunnel uses: ecp384/aes256/sha512 for IKE and ESP
# Bandwidth: up to 2 Gbps per connector
```

### Step 3: Define Application Definitions and Access Policies

Create application definitions pointing to internal applications via ZTNA Connectors.

```text
Strata Cloud Manager > Prisma Access > Applications:

Application 1: Internal Wiki
  - FQDN: wiki.internal.corp
  - Port: TCP 443
  - ZTNA Connector: dc-east-connector-01
  - Protocol: HTTPS
  - Health Check: Enabled (HTTP GET /health)

Application 2: Source Code Repository
  - FQDN: git.internal.corp
  - Ports: TCP 22, 443
  - ZTNA Connector: dc-east-connector-01, dc-east-connector-02
  - Protocol: HTTPS, SSH

Application 3: Finance ERP
  - FQDN: erp.internal.corp
  - Port: TCP 443
  - ZTNA Connector: dc-east-connector-01
  - Protocol: HTTPS
  - User Authentication: Required (re-auth every 2h)

Strata Cloud Manager > Policies > Security Policy:

Rule 1: Engineering Access to Dev Tools
  Source: User Group "Engineering" (from Okta SAML)
  Destination: Application "Source Code Repository", "Internal Wiki"
  HIP Profile: "Managed Device with CrowdStrike"
  Action: Allow
  Logging: Enabled
  Threat Prevention: Best Practice profile

Rule 2: Finance Access to ERP
  Source: User Group "Finance"
  Destination: Application "Finance ERP"
  HIP Profile: "Compliant Device - High Security"
  Action: Allow
  SSL Decryption: Forward Proxy
  DLP Profile: "Financial Data Protection"

Rule 3: Default Deny Private Apps
  Source: Any
  Destination: Any Private App
  Action: Deny
  Logging: Enabled
```

### Step 4: Configure Host Information Profile (HIP) for Device Posture

Define device posture requirements using HIP checks.

```text
Strata Cloud Manager > Objects > GlobalProtect > HIP Objects:

HIP Object: "CrowdStrike Running"
  - Vendor: CrowdStrike
  - Product: Falcon Sensor
  - Is Running: Yes
  - Minimum Version: 7.10

HIP Object: "Disk Encryption Enabled"
  - Windows: BitLocker = Encrypted
  - macOS: FileVault = Encrypted

HIP Object: "OS Patch Level"
  - Windows: >= 10.0.22631
  - macOS: >= 14.0

HIP Profile: "Managed Device with CrowdStrike"
  - Match: "CrowdStrike Running" AND "Disk Encryption Enabled"

HIP Profile: "Compliant Device - High Security"
  - Match: "CrowdStrike Running" AND "Disk Encryption Enabled" AND "OS Patch Level"
```

### Step 5: Deploy GlobalProtect Agent to Endpoints

Roll out the GlobalProtect agent for secure connectivity.

```bash
# Deploy GlobalProtect via Intune (Windows)
# MSI download from Strata Cloud Manager > GlobalProtect > Agent Downloads

# GlobalProtect pre-deployment configuration
# pre-deploy.xml for automated portal connection:
cat > pre-deploy.xml << 'EOF'
<GlobalProtect>
  <Settings>
    <portal>portal.company.com</portal>
    <connect-method>pre-logon</connect-method>
    <authentication-override>
      <generate-cookie>yes</generate-cookie>
      <cookie-lifetime>24</cookie-lifetime>
    </authentication-override>
  </Settings>
</GlobalProtect>
EOF

# Verify GlobalProtect connection status
# GlobalProtect system tray > Settings > Connection Details
# Should show: Connected to nearest Prisma Access gateway
# IPSec tunnel established with full threat prevention
```

### Step 6: Configure Logging and Monitoring

Set up Cortex Data Lake integration and monitoring dashboards.

```text
Strata Cloud Manager > Prisma Access > Monitoring:

Log Forwarding:
  - Cortex Data Lake: Enabled (all log types)
  - SIEM Forwarding: Splunk HEC (https://splunk-hec.company.com:8088)
  - Log Types: Traffic, Threat, URL, WildFire, GlobalProtect, HIP Match

Dashboard Monitoring:
  - Mobile Users: Active connections, locations, bandwidth
  - ZTNA Connectors: Health, latency, tunnel status
  - Security Events: Threats blocked, DLP violations, HIP failures
  - Application Usage: Top apps, top users, denied access attempts

Alerting:
  - ZTNA Connector down: Email + PagerDuty
  - HIP failure rate > 10%: Email to IT
  - Threat detected on mobile user: SOC alert
```

## Key Concepts

| Term | Definition |
|------|------------|
| Prisma Access | Palo Alto's cloud-delivered SASE platform providing FWaaS, SWG, CASB, DLP, and ZTNA from a single architecture |
| ZTNA Connector | VM-based connector establishing IPSec tunnels from internal networks to Prisma Access for private application access |
| GlobalProtect | Endpoint agent providing secure connectivity to Prisma Access with HIP checks and always-on VPN |
| Host Information Profile (HIP) | Device posture checks evaluating endpoint security state (EDR, encryption, patches) before granting access |
| Strata Cloud Manager | Unified management console for Prisma Access, NGFW, and Prisma Cloud security policy |
| Cortex Data Lake | Cloud-based log storage and analytics platform for Palo Alto security telemetry |

## Tools & Systems

- **Prisma Access**: Cloud-delivered SASE with integrated ZTNA, SWG, CASB, DLP, FWaaS
- **Strata Cloud Manager (SCM)**: Unified policy management across Palo Alto security products
- **GlobalProtect Agent**: Endpoint connectivity agent with HIP data collection
- **ZTNA Connector**: Outbound-only tunnel connector for internal application access
- **Cortex Data Lake**: Centralized log storage with analytics and threat detection
- **WildFire**: Cloud-based malware analysis and prevention integrated with Prisma Access

## Common Scenarios

### Scenario: Enterprise SASE Migration for 5,000-User Organization

**Context**: A manufacturing company with 5,000 users across 15 offices is consolidating VPN, SWG, and branch firewalls into Prisma Access SASE. Users access 50+ internal applications and need consistent security regardless of location.

**Approach**:
1. Deploy ZTNA Connectors at 3 data centers (2 per DC for HA) for internal application access
2. Configure GlobalProtect with pre-logon connection for always-on security
3. Define 50+ application definitions in SCM with FQDN and port mappings
4. Create HIP profiles: Standard (encryption + AV), Enhanced (+ CrowdStrike + patches)
5. Build security policies mapping user groups to applications with HIP requirements
6. Enable threat prevention profiles (Anti-Spyware, Anti-Virus, WildFire, URL Filtering)
7. Deploy GlobalProtect agent via SCCM to all 5,000 endpoints in phases
8. Configure Cortex Data Lake forwarding to Splunk for SOC monitoring
9. Decommission VPN concentrators and branch firewall appliances

**Pitfalls**: ZTNA Connector requires minimum 4 vCPU and 8GB RAM; under-provisioning causes latency. GlobalProtect pre-logon requires machine certificates for authentication before user login. HIP check intervals should be 60 seconds minimum to avoid performance impact. Plan for a 4-6 week pilot before full deployment.

## Output Format

```
Prisma Access ZTNA Deployment Report
==================================================
Organization: ManufactureCorp
Deployment Date: 2026-02-23

INFRASTRUCTURE:
  ZTNA Connectors: 6 (2x DC-East, 2x DC-West, 2x DC-EU)
  Prisma Access Locations: 8 (auto-selected)
  GlobalProtect Portal: portal.manufacturecorp.com

APPLICATION ACCESS:
  Defined Applications: 52
  Active ZTNA Connections: 3,247
  Average Latency: 12ms

ENDPOINT DEPLOYMENT:
  GlobalProtect Deployed: 4,812 / 5,000 (96.2%)
  HIP Compliant: 4,567 / 4,812 (94.9%)
  HIP Failures: 245 (top: missing patches 120, encryption 85)

SECURITY (last 30 days):
  Threats Blocked: 1,234
  DLP Violations: 89
  URL Blocked: 45,678
  WildFire Submissions: 2,345
```

===== END FILE =====


===== FILE: skills/deploying-palo-alto-prisma-access-zero-trust/LICENSE =====

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


===== FILE: skills/deploying-palo-alto-prisma-access-zero-trust/references/api-reference.md =====
# Palo Alto Prisma Access Zero Trust — API Reference

## Authentication

| Parameter | Value |
|-----------|-------|
| Token URL | `https://auth.apps.paloaltonetworks.com/oauth2/access_token` |
| Grant Type | `client_credentials` |
| Scope | `tsg_id:<tenant_service_group_id>` |

## SASE Configuration API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/sse/config/v1/remote-networks` | List remote network connections |
| GET | `/sse/config/v1/service-connections` | List service connections |
| GET | `/sse/config/v1/ike-gateways` | List IKE gateway configurations |
| GET | `/sse/config/v1/security-rules` | List security policy rules |
| GET | `/sse/config/v1/hip-profiles` | List Host Information Profiles |
| GET | `/sse/config/v1/mobile-agent/global-settings` | GlobalProtect mobile user config |
| POST | `/sse/config/v1/security-rules` | Create security rule |
| PUT | `/sse/config/v1/security-rules/{id}` | Update security rule |

## Base URL

```
https://api.sase.paloaltonetworks.com
```

## Security Rule Actions

| Action | Description |
|--------|-------------|
| `allow` | Permit traffic matching rule |
| `deny` | Block traffic matching rule |
| `drop` | Silently drop traffic |
| `reset-client` | Send TCP RST to client |

## HIP Match Criteria

| Field | Description |
|-------|-------------|
| `disk-encryption` | Require disk encryption enabled |
| `firewall` | Require host firewall active |
| `patch-management` | Require OS patches current |
| `anti-malware` | Require AV/EDR running |

## External References

- [Prisma Access SASE API](https://pan.dev/sase/api/)
- [Prisma Access Configuration Guide](https://docs.paloaltonetworks.com/prisma-access)
- [SASE Authentication](https://pan.dev/sase/docs/getstarted/)

===== END FILE =====


===== FILE: skills/deploying-palo-alto-prisma-access-zero-trust/scripts/agent.py =====
#!/usr/bin/env python3
"""Palo Alto Prisma Access zero trust deployment audit agent."""

import json
import sys
import argparse
from datetime import datetime

try:
    import requests
except ImportError:
    print("Install: pip install requests")
    sys.exit(1)


class PrismaAccessClient:
    """Client for Prisma Access Cloud Management API."""

    def __init__(self, tsg_id, client_id, client_secret):
        self.base_url = "https://api.sase.paloaltonetworks.com"
        self.tsg_id = tsg_id
        self.token = self._authenticate(client_id, client_secret)

    def _authenticate(self, client_id, client_secret):
        resp = requests.post(
            "https://auth.apps.paloaltonetworks.com/oauth2/access_token",
            data={
                "grant_type": "client_credentials",
                "scope": f"tsg_id:{self.tsg_id}",
            },
            auth=(client_id, client_secret),
            timeout=15,
        )
        resp.raise_for_status()
        return resp.json()["access_token"]

    def _get(self, path, params=None):
        headers = {"Authorization": f"Bearer {self.token}"}
        resp = requests.get(f"{self.base_url}{path}", headers=headers,
                            params=params, timeout=15)
        resp.raise_for_status()
        return resp.json()

    def list_remote_networks(self):
        return self._get("/sse/config/v1/remote-networks").get("data", [])

    def list_service_connections(self):
        return self._get("/sse/config/v1/service-connections").get("data", [])

    def list_ike_gateways(self):
        return self._get("/sse/config/v1/ike-gateways").get("data", [])

    def list_security_rules(self):
        return self._get("/sse/config/v1/security-rules").get("data", [])

    def list_hip_profiles(self):
        return self._get("/sse/config/v1/hip-profiles").get("data", [])

    def get_mobile_users_config(self):
        return self._get("/sse/config/v1/mobile-agent/global-settings").get("data", {})


def audit_security_rules(rules):
    """Check for overly permissive security rules."""
    findings = []
    for rule in rules:
        if rule.get("action") == "allow":
            src = rule.get("source", [])
            dst = rule.get("destination", [])
            if "any" in src and "any" in dst:
                findings.append({
                    "rule": rule.get("name", ""),
                    "issue": "Allow-any-to-any rule detected",
                    "severity": "HIGH",
                })
        if not rule.get("log_end", False):
            findings.append({
                "rule": rule.get("name", ""),
                "issue": "Logging not enabled on rule",
                "severity": "MEDIUM",
            })
    return findings


def run_audit(tsg_id, client_id, client_secret):
    """Execute Prisma Access zero trust audit."""
    print(f"\n{'='*60}")
    print(f"  PRISMA ACCESS ZERO TRUST AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    client = PrismaAccessClient(tsg_id, client_id, client_secret)
    report = {}

    networks = client.list_remote_networks()
    report["remote_networks"] = len(networks)
    print(f"--- REMOTE NETWORKS ({len(networks)}) ---")
    for n in networks[:10]:
        print(f"  {n.get('name', '')}: region={n.get('region', '')}")

    connections = client.list_service_connections()
    report["service_connections"] = len(connections)
    print(f"\n--- SERVICE CONNECTIONS ({len(connections)}) ---")
    for c in connections[:10]:
        print(f"  {c.get('name', '')}: {c.get('ipsec_tunnel', '')}")

    rules = client.list_security_rules()
    findings = audit_security_rules(rules)
    report["security_rules"] = len(rules)
    report["findings"] = findings
    print(f"\n--- SECURITY RULES ({len(rules)}) ---")
    print(f"  Findings: {len(findings)}")
    for f in findings[:10]:
        print(f"  [{f['severity']}] {f['rule']}: {f['issue']}")

    hip = client.list_hip_profiles()
    report["hip_profiles"] = len(hip)
    print(f"\n--- HIP PROFILES ({len(hip)}) ---")
    for h in hip[:5]:
        print(f"  {h.get('name', '')}")

    return report


def main():
    parser = argparse.ArgumentParser(description="Prisma Access Zero Trust Audit")
    parser.add_argument("--tsg-id", required=True, help="Tenant Service Group ID")
    parser.add_argument("--client-id", required=True, help="OAuth client ID")
    parser.add_argument("--client-secret", required=True, help="OAuth client secret")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_audit(args.tsg_id, args.client_id, args.client_secret)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/deploying-software-defined-perimeter/SKILL.md =====
---
name: deploying-software-defined-perimeter
description: Deploys a Software-Defined Perimeter per the CSA v2.0 specification, configuring Single Packet Authorization, mutual TLS, and SDP controller/gateway components to enforce zero trust network access. Use when building or hardening zero trust network architecture, implementing SPA-based "invisible" infrastructure that cloaks services from unauthenticated scanning, or meeting compliance requirements for zero trust network access.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zero-trust
- sdp
- software-defined-perimeter
- network-access
- ztna
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1133
- T1078
- T1021
- T1046
- T1190
---

# Deploying Software-Defined Perimeter

## Prerequisites

- Understanding of zero trust principles (NIST SP 800-207)
- Knowledge of CSA Software-Defined Perimeter specification
- Familiarity with PKI and mutual TLS authentication
- Experience with network security architecture

## Overview

A Software-Defined Perimeter (SDP) implements zero trust by creating a dynamically provisioned, identity-centric perimeter around individual resources. Defined by the Cloud Security Alliance (CSA), SDP makes application infrastructure invisible to unauthorized users through a "dark cloud" approach where services are hidden until authenticated and authorized. Unlike traditional VPN, SDP establishes one-to-one encrypted connections between verified users and specific applications.

This skill covers deploying SDP using the CSA v2.0 specification, implementing Single Packet Authorization (SPA), configuring the SDP controller and gateway, and validating the deployment against NIST SP 800-207 requirements.


## When to Use

- When deploying or configuring deploying software defined perimeter capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with zero trust architecture concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Architecture

### SDP Components (CSA Specification)

```
┌─────────────────────┐
│ SDP Controller       │
│ - Authentication     │
│ - Authorization      │
│ - Policy management  │
│ - Key management     │
└──────────┬──────────┘
           │
    ┌──────┴──────┐
    │             │
    v             v
┌────────┐  ┌────────────┐
│ IH     │  │ AH         │
│(Client)│  │(Gateway)   │
│        │  │            │
│ SPA    │──│ Protected  │
│ mTLS   │  │ Resources  │
└────────┘  └────────────┘

IH = Initiating Host (User Device)
AH = Accepting Host (Application Gateway)
SPA = Single Packet Authorization
```

### SDP Deployment Models
1. **Client-to-Gateway**: User device connects through SDP gateway to backend applications
2. **Client-to-Server**: Direct connection between user and application server
3. **Server-to-Server**: Workload-to-workload communication through SDP
4. **Gateway-to-Gateway**: Site-to-site connectivity replacing traditional VPN tunnels

## Key Concepts

### Single Packet Authorization (SPA)
SPA is a network security mechanism where the SDP gateway drops all TCP/UDP packets by default. A cryptographically signed single packet must be sent before any connection is established. The gateway validates the SPA packet, and only then opens a temporary port for the authenticated session. This makes the gateway invisible to port scanners.

### Mutual TLS (mTLS)
After SPA validation, both the client and server authenticate each other using X.509 certificates. This bidirectional authentication prevents man-in-the-middle attacks and ensures both endpoints are verified.

### Dynamic Provisioning
SDP connections are provisioned on-demand based on real-time policy evaluation. No persistent network tunnels exist; each session is individually authorized and encrypted.

## Workflow

### Phase 1: SDP Controller Deployment

1. **Deploy SDP Controller**
   - Install SDP controller on hardened, redundant infrastructure
   - Configure PKI integration for certificate issuance
   - Set up authentication backend (LDAP, SAML, OIDC)
   - Configure policy database with application definitions
   - Enable audit logging for all controller decisions

2. **Configure Authentication**
   - Integrate with enterprise IdP via SAML 2.0 or OIDC
   - Configure device certificate enrollment (SCEP/EST)
   - Enable multi-factor authentication requirements
   - Set up certificate revocation checking (OCSP/CRL)

3. **Define Access Policies**
   - Map users/groups to authorized applications
   - Define device posture requirements per application
   - Configure contextual conditions (location, time, risk level)
   - Set session duration and re-authentication intervals

### Phase 2: SDP Gateway Deployment

4. **Deploy Accepting Hosts (Gateways)**
   - Install SDP gateway instances in front of protected applications
   - Configure default-drop firewall rules (deny all inbound)
   - Enable SPA listener on designated ports
   - Configure mTLS with controller-issued certificates
   - Set up health monitoring and failover

5. **Configure Application Definitions**
   - Register each protected application with the controller
   - Define backend server IPs, ports, and protocols
   - Configure load balancing for multi-instance applications
   - Set up application health checks

### Phase 3: Client Deployment

6. **Deploy Initiating Hosts (Clients)**
   - Install SDP client software on user endpoints
   - Enroll device certificates through automated provisioning
   - Configure SPA key material distribution
   - Test authentication flow: SPA → mTLS → application access

7. **Validate End-to-End Flow**
   - Verify SPA packets are accepted by gateway
   - Confirm mTLS handshake succeeds with valid certificates
   - Test application access through the SDP tunnel
   - Verify unauthorized access is blocked (no SPA = invisible gateway)

### Phase 4: Operational Validation

8. **Security Testing**
   - Port scan the SDP gateway to confirm invisibility (all ports show filtered/closed)
   - Attempt connection without valid SPA (must fail silently)
   - Test with revoked client certificate (must be denied)
   - Attempt lateral movement from one authorized app to another unauthorized app
   - Validate audit trail completeness

9. **Monitoring and Maintenance**
   - Configure SIEM integration for SDP controller and gateway logs
   - Set up alerting for failed SPA attempts and certificate errors
   - Establish certificate rotation schedule
   - Document incident response procedures for SDP events

## Validation Checklist

- [ ] SDP Controller deployed with HA and audit logging
- [ ] IdP integration tested with SAML/OIDC and MFA
- [ ] SDP Gateways deployed with default-drop firewall
- [ ] SPA mechanism validated (gateway invisible to port scans)
- [ ] mTLS established between clients and gateways
- [ ] Access policies enforce least-privilege per user/app
- [ ] Device certificate enrollment automated
- [ ] Unauthorized access attempts blocked silently
- [ ] Lateral movement between apps prevented
- [ ] Logs streaming to SIEM with alerting configured
- [ ] Certificate rotation and revocation procedures tested

## References

- CSA Software-Defined Perimeter Architecture Guide v3
- CSA SDP Specification v2.0
- NIST SP 800-207: Zero Trust Architecture
- CISA Zero Trust Maturity Model v2.0
- fwknop: Single Packet Authorization implementation

===== END FILE =====


===== FILE: skills/deploying-software-defined-perimeter/LICENSE =====

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


===== FILE: skills/deploying-software-defined-perimeter/assets/template.md =====
# SDP Deployment Plan Template

## Project Information

| Field | Value |
|---|---|
| Project Name | |
| SDP Solution | [Appgate SDP / Zscaler / Open-source / Other] |
| Project Lead | |
| Start Date | |

## Application Inventory

| Application | FQDN/IP | Port | Protocol | Criticality | Gateway Assignment |
|---|---|---|---|---|---|
| | | | | | |

## SDP Controller Configuration

| Parameter | Value |
|---|---|
| HA Mode | [Active-Active / Active-Passive] |
| IdP Integration | [SAML / OIDC] |
| IdP Provider | [Azure AD / Okta / Ping] |
| PKI Backend | [Internal CA / HashiCorp Vault / EJBCA] |
| Client Cert Lifetime | [24h / 48h / 72h] |
| Audit Log Destination | [SIEM / Syslog / Cloud storage] |

## Gateway Deployment

| Gateway Name | Location | Protected Apps | SPA Enabled | mTLS Enabled | Default-Drop |
|---|---|---|---|---|---|
| | | | Yes | Yes | Yes |

## Access Policy Matrix

| User Group | Application | Conditions | Action |
|---|---|---|---|
| | | Device posture + MFA | Allow |
| Default | All | None | Deny |

## Security Validation

- [ ] Port scan confirms gateway invisibility
- [ ] SPA validation working correctly
- [ ] mTLS handshake succeeds with valid certs
- [ ] Invalid SPA packets dropped silently
- [ ] Revoked certificates denied access
- [ ] Lateral movement between apps blocked
- [ ] Logs captured in SIEM

## Sign-Off

| Stakeholder | Role | Approval | Date |
|---|---|---|---|
| | Security Architecture | | |
| | Network Engineering | | |
| | Application Owners | | |

===== END FILE =====


===== FILE: skills/deploying-software-defined-perimeter/references/api-reference.md =====
# Software-Defined Perimeter — API Reference

## Core SDP Concepts

| Component | Description |
|-----------|-------------|
| SDP Controller | Central policy engine managing authentication and authorization |
| SDP Gateway | Enforces access policies, terminates encrypted tunnels |
| SDP Client | End-user agent performing Single Packet Authorization (SPA) |
| SPA (Single Packet Authorization) | Cryptographic knock before TCP connection allowed |

## Appgate SDP Admin API

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/admin/login` | Authenticate and get Bearer token |
| GET | `/admin/sites` | List configured sites |
| GET | `/admin/policies` | List access policies |
| GET | `/admin/entitlements` | List entitlements (resource access rules) |
| GET | `/admin/appliances` | List SDP gateways and controllers |
| GET | `/admin/identity-providers` | List identity providers |
| POST | `/admin/entitlements` | Create new entitlement |

## Dark Port Scanning

```python
import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.settimeout(5)
result = sock.connect_ex((host, port))  # Non-zero = port dark (SDP enforced)
```

## Mutual TLS Verification

```python
import ssl
ctx = ssl.create_default_context()
ctx.load_cert_chain("client.crt", "client.key")
with ctx.wrap_socket(socket.socket(), server_hostname=host) as s:
    s.connect((host, 443))
```

## SPA Packet Structure

| Field | Description |
|-------|-------------|
| Random Data | 16 bytes of random padding |
| Username | SDP client identity |
| Timestamp | Prevents replay attacks |
| HMAC | SHA-256 authentication code |

## External References

- [Appgate SDP API Docs](https://sdphelp.appgate.com/adminguide/rest-api-guide.html)
- [CSA SDP Specification](https://cloudsecurityalliance.org/research/sdp/)
- [fwknop SPA Tool](https://www.cipherdyne.org/fwknop/)

===== END FILE =====


===== FILE: skills/deploying-software-defined-perimeter/references/standards.md =====
# Standards and Frameworks Reference

## CSA Software-Defined Perimeter Specification v2.0

### Core Architecture
- **SDP Controller**: Central policy and authentication authority
- **Initiating Host (IH)**: Client device requesting access
- **Accepting Host (AH)**: Gateway protecting backend resources
- **Single Packet Authorization (SPA)**: Pre-authentication mechanism making services invisible

### SDP Workflow
1. IH authenticates to SDP Controller
2. Controller validates identity, device posture, and policy
3. Controller instructs AH to accept connection from specific IH
4. IH sends SPA packet to AH
5. AH validates SPA and opens temporary port
6. mTLS tunnel established between IH and AH
7. Application traffic flows through encrypted tunnel

### Deployment Models
| Model | Use Case | Architecture |
|---|---|---|
| Client-to-Gateway | Remote user access | IH → AH Gateway → Backend servers |
| Client-to-Server | Direct application access | IH → AH (application server) |
| Server-to-Server | Workload communication | IH (server) → AH (server) |
| Gateway-to-Gateway | Site-to-site connectivity | AH₁ → Controller → AH₂ |

## NIST SP 800-207: SDP as Zero Trust Deployment

### SDP Mapping to NIST ZTA Components
| NIST Component | SDP Equivalent |
|---|---|
| Policy Engine (PE) | SDP Controller policy evaluation |
| Policy Administrator (PA) | SDP Controller session management |
| Policy Enforcement Point (PEP) | SDP Gateway (Accepting Host) |

### NIST ZTA Tenets Addressed by SDP
- All communication secured regardless of network location (mTLS tunnels)
- Per-session access grants (dynamic SDP connections)
- Dynamic policy evaluation (controller real-time decisions)
- Asset integrity monitoring (device posture checks)

## CISA Zero Trust Maturity Model v2.0

### Network Pillar - SDP Alignment
| Maturity | SDP Capability |
|---|---|
| Traditional | No SDP, perimeter-based VPN |
| Initial | SDP for remote access, basic SPA |
| Advanced | Full SDP with device posture, context-aware |
| Optimal | Dynamic SDP with continuous verification, ML-driven |

## Single Packet Authorization (SPA) Technical Details

### SPA Packet Structure
- Encrypted with shared key or asymmetric cryptography
- Contains: source IP, timestamp, HMAC, requested service
- Single UDP packet (no TCP handshake visible)
- Anti-replay protection via timestamp and sequence number

### fwknop Implementation
- Open-source SPA implementation
- Supports AES-256 and GnuPG encryption
- Integrates with iptables/nftables for firewall rule insertion
- Temporary rule created for authenticated session only

## mTLS Configuration Standards

### Certificate Requirements
- Minimum RSA 2048-bit or ECDSA P-256 keys
- Short-lived certificates (24-72 hours) preferred
- OCSP stapling for real-time revocation checking
- Certificate pinning for additional security

===== END FILE =====


===== FILE: skills/deploying-software-defined-perimeter/references/workflows.md =====
# SDP Deployment Workflows

## Workflow 1: SDP Connection Establishment

```
┌────────────┐     ┌──────────────┐     ┌────────────┐
│ IH (Client) │     │ SDP Controller│     │ AH (Gateway)│
└──────┬─────┘     └──────┬───────┘     └──────┬─────┘
       │                   │                     │
       │ 1. Authenticate   │                     │
       │──────────────────>│                     │
       │                   │                     │
       │ 2. Validate ID,   │                     │
       │    device, policy │                     │
       │                   │                     │
       │ 3. Auth response  │                     │
       │<──────────────────│                     │
       │  (SPA key, AH IP) │                     │
       │                   │ 4. Notify AH to     │
       │                   │    expect IH        │
       │                   │────────────────────>│
       │                   │                     │
       │ 5. Send SPA packet│                     │
       │─────────────────────────────────────────>│
       │                   │                     │
       │                   │  6. Validate SPA    │
       │                   │     Open port       │
       │                   │                     │
       │ 7. mTLS handshake │                     │
       │<════════════════════════════════════════>│
       │                   │                     │
       │ 8. Application    │                     │
       │    traffic flows  │                     │
       │<═══════════════════════════════════════=>│
```

## Workflow 2: SDP Deployment Lifecycle

```
Phase 1: Planning (Weeks 1-2)
├── Inventory protected applications
├── Map user-to-application access requirements
├── Design PKI infrastructure for mTLS
├── Select SDP solution (open-source or commercial)
└── Plan network architecture changes

Phase 2: Controller Setup (Weeks 3-4)
├── Deploy SDP controller with HA
├── Integrate with IdP (SAML/OIDC)
├── Configure PKI and certificate templates
├── Define application catalog and policies
└── Test controller authentication flow

Phase 3: Gateway Deployment (Weeks 5-6)
├── Deploy gateways in each app environment
├── Configure default-drop firewall rules
├── Enable SPA listeners
├── Register applications with controller
└── Verify gateway invisibility (port scan test)

Phase 4: Client Rollout (Weeks 7-10)
├── Package SDP client with certificates
├── Deploy to pilot user group
├── Validate end-to-end connectivity
├── Expand to all user groups
└── Decommission legacy VPN access

Phase 5: Operations (Ongoing)
├── Monitor SDP controller and gateway health
├── Rotate certificates on schedule
├── Review and update access policies
├── Conduct quarterly penetration tests
└── Update SDP components for security patches
```

## Workflow 3: SPA Validation

```
Incoming Packet to Gateway
    │
    v
┌─────────────────────┐
│ Is it a SPA packet?  │
│ (Check magic bytes)  │
└───┬──────────┬──────┘
    │          │
   YES        NO
    │          │
    v          v
┌──────────┐  ┌──────────┐
│ Decrypt  │  │ DROP     │
│ SPA data │  │ silently │
└────┬─────┘  └──────────┘
     v
┌─────────────────────┐
│ Validate timestamp   │
│ (within 60s window)  │
└───┬──────────┬──────┘
   VALID    EXPIRED
    │          │
    v          v
┌──────────┐  ┌──────────┐
│ Check    │  │ DROP +   │
│ HMAC     │  │ Log      │
└────┬─────┘  └──────────┘
     v
┌─────────────────────┐
│ Verify replay        │
│ (check sequence DB)  │
└───┬──────────┬──────┘
   NEW      REPLAY
    │          │
    v          v
┌──────────┐  ┌──────────┐
│ Open port │  │ DROP +   │
│ for src IP│  │ Alert    │
│ (30s TTL) │  └──────────┘
└──────────┘
```

===== END FILE =====


===== FILE: skills/deploying-software-defined-perimeter/scripts/agent.py =====
#!/usr/bin/env python3
"""Software-Defined Perimeter (SDP) deployment audit agent."""

import json
import sys
import argparse
import socket
import ssl
from datetime import datetime

try:
    import requests
except ImportError:
    print("Install: pip install requests")
    sys.exit(1)


def check_spa_port(host, port, timeout=5):
    """Check if a port responds to standard TCP connect (should be dark in SDP)."""
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout)
        result = sock.connect_ex((host, port))
        sock.close()
        return {"host": host, "port": port, "open": result == 0}
    except socket.error as e:
        return {"host": host, "port": port, "open": False, "error": str(e)}


def audit_dark_ports(host, ports):
    """Verify SDP ports are invisible to unauthorized scanners."""
    results = []
    for port in ports:
        scan = check_spa_port(host, port)
        if scan.get("open"):
            scan["finding"] = "Port visible without SPA — SDP not enforced"
            scan["severity"] = "HIGH"
        else:
            scan["finding"] = "Port dark — SDP enforced"
            scan["severity"] = "INFO"
        results.append(scan)
    return results


def check_tls_mutual_auth(host, port, client_cert=None, client_key=None):
    """Verify mutual TLS authentication on SDP controller."""
    result = {"host": host, "port": port}
    try:
        ctx = ssl.create_default_context()
        if client_cert and client_key:
            ctx.load_cert_chain(client_cert, client_key)
        with ctx.wrap_socket(socket.socket(), server_hostname=host) as s:
            s.settimeout(10)
            s.connect((host, port))
            cert = s.getpeercert()
            result["tls_version"] = s.version()
            result["cipher"] = s.cipher()[0]
            result["server_cn"] = dict(x[0] for x in cert.get("subject", ()))
            result["mtls_enforced"] = client_cert is not None
    except ssl.SSLError as e:
        result["error"] = str(e)
        if "CERTIFICATE_REQUIRED" in str(e):
            result["mtls_enforced"] = True
            result["finding"] = "Server requires client certificate — mTLS enforced"
    except Exception as e:
        result["error"] = str(e)
    return result


class SDPControllerClient:
    """Client for SDP controller REST API (e.g., Appgate SDP)."""

    def __init__(self, controller_url, username, password):
        self.base_url = controller_url.rstrip("/")
        self.session = requests.Session()
        self._authenticate(username, password)

    def _authenticate(self, username, password):
        resp = self.session.post(f"{self.base_url}/admin/login", json={
            "providerName": "local",
            "username": username,
            "password": password,
        }, timeout=15, verify=True)
        resp.raise_for_status()
        token = resp.json().get("token", "")
        self.session.headers.update({"Authorization": f"Bearer {token}"})

    def list_sites(self):
        resp = self.session.get(f"{self.base_url}/admin/sites", timeout=15)
        resp.raise_for_status()
        return resp.json().get("data", [])

    def list_policies(self):
        resp = self.session.get(f"{self.base_url}/admin/policies", timeout=15)
        resp.raise_for_status()
        return resp.json().get("data", [])

    def list_entitlements(self):
        resp = self.session.get(f"{self.base_url}/admin/entitlements", timeout=15)
        resp.raise_for_status()
        return resp.json().get("data", [])

    def list_appliances(self):
        resp = self.session.get(f"{self.base_url}/admin/appliances", timeout=15)
        resp.raise_for_status()
        return resp.json().get("data", [])


def run_audit(args):
    """Execute SDP deployment audit."""
    print(f"\n{'='*60}")
    print(f"  SOFTWARE-DEFINED PERIMETER AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    report = {}

    if args.scan_host:
        ports = [int(p) for p in args.ports.split(",")] if args.ports else [22, 443, 8443, 3389]
        dark_results = audit_dark_ports(args.scan_host, ports)
        report["dark_port_scan"] = dark_results
        print(f"--- DARK PORT SCAN ({args.scan_host}) ---")
        for r in dark_results:
            status = "OPEN" if r["open"] else "DARK"
            print(f"  Port {r['port']}: {status} — {r['finding']}")

    if args.mtls_host:
        mtls = check_tls_mutual_auth(args.mtls_host, args.mtls_port or 443,
                                      args.client_cert, args.client_key)
        report["mtls_check"] = mtls
        print(f"\n--- MUTUAL TLS CHECK ---")
        print(f"  Host: {mtls['host']}:{mtls['port']}")
        print(f"  mTLS Enforced: {mtls.get('mtls_enforced', 'unknown')}")
        if mtls.get("tls_version"):
            print(f"  TLS Version: {mtls['tls_version']}")

    if args.controller_url and args.username and args.password:
        client = SDPControllerClient(args.controller_url, args.username, args.password)
        appliances = client.list_appliances()
        report["appliances"] = len(appliances)
        print(f"\n--- SDP APPLIANCES ({len(appliances)}) ---")
        for a in appliances[:10]:
            print(f"  {a.get('name', '')}: {a.get('function', '')} ({a.get('state', '')})")

        entitlements = client.list_entitlements()
        report["entitlements"] = len(entitlements)
        print(f"\n--- ENTITLEMENTS ({len(entitlements)}) ---")
        for e in entitlements[:10]:
            print(f"  {e.get('name', '')}: {e.get('site', '')} -> {e.get('actions', [])}")

    return report


def main():
    parser = argparse.ArgumentParser(description="SDP Deployment Audit")
    parser.add_argument("--scan-host", help="Host to scan for dark ports")
    parser.add_argument("--ports", help="Comma-separated ports to scan (default: 22,443,8443,3389)")
    parser.add_argument("--mtls-host", help="Host to check mutual TLS")
    parser.add_argument("--mtls-port", type=int, default=443, help="mTLS port")
    parser.add_argument("--client-cert", help="Client certificate for mTLS test")
    parser.add_argument("--client-key", help="Client key for mTLS test")
    parser.add_argument("--controller-url", help="SDP controller URL (Appgate)")
    parser.add_argument("--username", help="Controller admin username")
    parser.add_argument("--password", help="Controller admin password")
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


===== FILE: skills/deploying-software-defined-perimeter/scripts/process.py =====
#!/usr/bin/env python3
"""
Software-Defined Perimeter Deployment Validator

Validates SDP deployment readiness, tests SPA mechanisms,
verifies gateway invisibility, and generates deployment reports.
"""

import json
import socket
import hashlib
import hmac
import struct
import time
import ssl
import subprocess
import sys
from datetime import datetime
from pathlib import Path
from typing import Optional


def check_gateway_invisibility(host: str, port_range: tuple = (1, 1024), timeout: float = 0.5) -> dict:
    """Scan gateway ports to verify SDP invisibility (all ports should appear closed/filtered)."""
    result = {
        "host": host,
        "scanned_range": f"{port_range[0]}-{port_range[1]}",
        "open_ports": [],
        "closed_ports": 0,
        "filtered_ports": 0,
        "invisible": True,
        "timestamp": datetime.now().isoformat(),
    }

    for port in range(port_range[0], port_range[1] + 1):
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(timeout)
            conn_result = sock.connect_ex((host, port))
            if conn_result == 0:
                result["open_ports"].append(port)
                result["invisible"] = False
            else:
                result["filtered_ports"] += 1
            sock.close()
        except socket.timeout:
            result["filtered_ports"] += 1
        except OSError:
            result["closed_ports"] += 1

    return result


def generate_spa_packet(
    source_ip: str,
    destination_service: str,
    shared_key: str,
    timestamp: Optional[float] = None
) -> bytes:
    """Generate a Single Packet Authorization payload (demonstration)."""
    if timestamp is None:
        timestamp = time.time()

    payload = json.dumps({
        "version": 2,
        "source_ip": source_ip,
        "service": destination_service,
        "timestamp": timestamp,
        "nonce": hashlib.sha256(f"{time.time()}{source_ip}".encode()).hexdigest()[:16],
    }).encode()

    mac = hmac.new(shared_key.encode(), payload, hashlib.sha256).digest()
    packet = struct.pack("!I", len(payload)) + payload + mac

    return packet


def validate_spa_packet(packet: bytes, shared_key: str, max_age_seconds: int = 60) -> dict:
    """Validate a received SPA packet."""
    result = {"valid": False, "errors": [], "payload": None}

    try:
        payload_len = struct.unpack("!I", packet[:4])[0]
        payload = packet[4:4 + payload_len]
        received_mac = packet[4 + payload_len:]

        expected_mac = hmac.new(shared_key.encode(), payload, hashlib.sha256).digest()
        if not hmac.compare_digest(received_mac, expected_mac):
            result["errors"].append("HMAC verification failed")
            return result

        data = json.loads(payload.decode())
        result["payload"] = data

        age = time.time() - data.get("timestamp", 0)
        if age > max_age_seconds:
            result["errors"].append(f"Packet expired ({age:.0f}s old, max {max_age_seconds}s)")
            return result

        if age < -5:
            result["errors"].append("Packet timestamp is in the future")
            return result

        result["valid"] = True

    except (struct.error, json.JSONDecodeError, KeyError) as e:
        result["errors"].append(f"Packet parse error: {str(e)}")

    return result


def validate_mtls_certificate(host: str, port: int, ca_cert_path: Optional[str] = None) -> dict:
    """Validate mTLS certificate configuration on SDP gateway."""
    result = {
        "host": host,
        "port": port,
        "tls_configured": False,
        "certificate": None,
        "errors": [],
    }

    try:
        context = ssl.create_default_context()
        if ca_cert_path:
            context.load_verify_locations(ca_cert_path)

        with socket.create_connection((host, port), timeout=10) as sock:
            with context.wrap_socket(sock, server_hostname=host) as ssock:
                cert = ssock.getpeercert()
                result["tls_configured"] = True
                result["certificate"] = {
                    "subject": dict(x[0] for x in cert.get("subject", [])),
                    "issuer": dict(x[0] for x in cert.get("issuer", [])),
                    "version": cert.get("version"),
                    "not_before": cert.get("notBefore"),
                    "not_after": cert.get("notAfter"),
                    "serial": cert.get("serialNumber"),
                }

                not_after = cert.get("notAfter", "")
                if not_after:
                    expiry = datetime.strptime(not_after, "%b %d %H:%M:%S %Y %Z")
                    days_remaining = (expiry - datetime.utcnow()).days
                    result["certificate"]["days_remaining"] = days_remaining
                    if days_remaining < 30:
                        result["errors"].append(f"Certificate expires in {days_remaining} days")

    except ssl.SSLError as e:
        result["errors"].append(f"SSL error: {str(e)}")
    except Exception as e:
        result["errors"].append(f"Connection error: {str(e)}")

    return result


def validate_sdp_config(config: dict) -> dict:
    """Validate SDP deployment configuration."""
    findings = []
    score = 100

    controller = config.get("controller", {})
    if not controller.get("ha_enabled"):
        findings.append({"severity": "high", "finding": "Controller HA not enabled"})
        score -= 15

    if not controller.get("idp_integration"):
        findings.append({"severity": "critical", "finding": "No IdP integration configured"})
        score -= 25

    if not controller.get("audit_logging"):
        findings.append({"severity": "high", "finding": "Audit logging not enabled on controller"})
        score -= 10

    gateways = config.get("gateways", [])
    if not gateways:
        findings.append({"severity": "critical", "finding": "No SDP gateways deployed"})
        score -= 25
    for gw in gateways:
        if not gw.get("default_drop"):
            findings.append({"severity": "critical", "finding": f"Gateway {gw.get('name')}: default-drop not enabled"})
            score -= 20
        if not gw.get("spa_enabled"):
            findings.append({"severity": "critical", "finding": f"Gateway {gw.get('name')}: SPA not enabled"})
            score -= 15
        if not gw.get("mtls_enabled"):
            findings.append({"severity": "high", "finding": f"Gateway {gw.get('name')}: mTLS not configured"})
            score -= 10

    pki = config.get("pki", {})
    cert_lifetime_hours = pki.get("client_cert_lifetime_hours", 8760)
    if cert_lifetime_hours > 72:
        findings.append({
            "severity": "warning",
            "finding": f"Client certificate lifetime is {cert_lifetime_hours}h (recommend <=72h for zero trust)"
        })
        score -= 5

    if not pki.get("ocsp_enabled") and not pki.get("crl_enabled"):
        findings.append({"severity": "high", "finding": "No certificate revocation checking enabled"})
        score -= 10

    monitoring = config.get("monitoring", {})
    if not monitoring.get("siem_integration"):
        findings.append({"severity": "warning", "finding": "No SIEM integration for SDP events"})
        score -= 5

    return {
        "score": max(score, 0),
        "findings": findings,
        "status": "ready" if score >= 80 else "needs_work" if score >= 50 else "not_ready",
        "timestamp": datetime.now().isoformat(),
    }


def generate_sdp_deployment_report(config: dict) -> dict:
    """Generate comprehensive SDP deployment report."""
    validation = validate_sdp_config(config)

    applications = config.get("applications", [])
    users = config.get("authorized_users", [])

    return {
        "generated": datetime.now().isoformat(),
        "deployment_status": validation["status"],
        "security_score": validation["score"],
        "findings": validation["findings"],
        "summary": {
            "controller_ha": config.get("controller", {}).get("ha_enabled", False),
            "gateways_deployed": len(config.get("gateways", [])),
            "applications_protected": len(applications),
            "authorized_users": len(users),
            "spa_enabled": all(g.get("spa_enabled") for g in config.get("gateways", [])),
            "mtls_enabled": all(g.get("mtls_enabled") for g in config.get("gateways", [])),
        },
        "recommendations": [f["finding"] for f in validation["findings"] if f["severity"] in ("critical", "high")],
    }


def main():
    import argparse
    parser = argparse.ArgumentParser(description="SDP Deployment Validator")
    parser.add_argument("--config", type=str, help="Path to SDP configuration JSON")
    parser.add_argument("--scan", type=str, help="Gateway host to scan for invisibility")
    parser.add_argument("--scan-ports", type=str, default="1-1024", help="Port range to scan")
    parser.add_argument("--check-tls", type=str, help="Host:port to check TLS certificate")
    parser.add_argument("--output", type=str, default="sdp_report.json")
    args = parser.parse_args()

    if args.config:
        with open(args.config) as f:
            config = json.load(f)
        report = generate_sdp_deployment_report(config)
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"SDP Status: {report['deployment_status']} (Score: {report['security_score']})")
        for r in report["recommendations"]:
            print(f"  - {r}")

    elif args.scan:
        start, end = args.scan_ports.split("-")
        result = check_gateway_invisibility(args.scan, (int(start), int(end)))
        with open(args.output, "w") as f:
            json.dump(result, f, indent=2)
        status = "INVISIBLE" if result["invisible"] else "EXPOSED"
        print(f"Gateway {args.scan}: {status}")
        if result["open_ports"]:
            print(f"  Open ports: {result['open_ports']}")

    elif args.check_tls:
        parts = args.check_tls.split(":")
        host = parts[0]
        port = int(parts[1]) if len(parts) > 1 else 443
        result = validate_mtls_certificate(host, port)
        with open(args.output, "w") as f:
            json.dump(result, f, indent=2)
        print(f"TLS on {host}:{port}: {'configured' if result['tls_configured'] else 'not configured'}")

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/deploying-tailscale-for-zero-trust-vpn/SKILL.md =====
---
name: deploying-tailscale-for-zero-trust-vpn
description: Deploys and configures Tailscale (or self-hosted Headscale) as a WireGuard-based zero trust mesh VPN, setting up identity-aware ACLs, exit nodes, subnet routers, and MagicDNS for encrypted peer-to-peer connectivity. Use when replacing traditional VPN servers with an identity-authenticated mesh network or enforcing granular per-device access control lists.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zero-trust
- tailscale
- wireguard
- mesh-vpn
- ztna
- peer-to-peer
- acl
- identity-aware
- headscale
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1133
- T1078
- T1021
- T1572
---

# Deploying Tailscale for Zero Trust VPN

## Overview

Tailscale is a zero trust mesh VPN built on WireGuard that creates encrypted peer-to-peer connections between devices without requiring traditional VPN servers or complex network configuration. Every connection in a Tailscale network (tailnet) is end-to-end encrypted using WireGuard's Noise protocol framework with Curve25519 key exchange. Tailscale implements zero trust networking by authenticating every connection request through identity providers, enforcing granular Access Control Lists (ACLs), and supporting features like exit nodes, subnet routers, MagicDNS, and Tailscale SSH. For organizations preferring self-hosted infrastructure, Headscale provides an open-source implementation of the Tailscale control server.


## When to Use

- When deploying or configuring deploying tailscale for zero trust vpn capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Identity provider (Okta, Azure AD, Google Workspace, GitHub, or OIDC-compatible)
- Devices running supported OS (Linux, Windows, macOS, iOS, Android, FreeBSD)
- Administrative access to configure DNS and firewall rules
- Understanding of WireGuard protocol fundamentals
- Network planning documentation for subnet routing requirements

## Architecture

```
                    Tailscale Coordination Server
                    (or self-hosted Headscale)
                           |
                    Key Distribution
                    & NAT Traversal
                           |
         +-----------------+-----------------+
         |                 |                 |
    +----+----+      +----+----+      +----+----+
    | Node A  |<---->| Node B  |<---->| Node C  |
    | (Linux) |      | (macOS) |      |(Windows)|
    +---------+      +---------+      +---------+
    WireGuard         WireGuard        WireGuard
    Encrypted         Encrypted        Encrypted
    P2P Tunnel        P2P Tunnel       P2P Tunnel

    Each node connects directly to every other node.
    DERP relay servers used only when direct P2P fails.
```

## Installation and Setup

### Linux Installation

```bash
# Add Tailscale repository and install
curl -fsSL https://tailscale.com/install.sh | sh

# Start Tailscale and authenticate
sudo tailscale up

# Check connection status
tailscale status

# View assigned IP address
tailscale ip -4
tailscale ip -6
```

### Windows / macOS Installation

```bash
# Windows: Download from https://tailscale.com/download/windows
# macOS: Install via Homebrew
brew install --cask tailscale

# Or download from https://tailscale.com/download/mac
```

### Docker Deployment

```yaml
# docker-compose.yml for Tailscale sidecar
version: '3.8'
services:
  tailscale:
    image: tailscale/tailscale:latest
    container_name: tailscale
    hostname: my-service
    environment:
      - TS_AUTHKEY=tskey-auth-xxxxx  # Pre-auth key
      - TS_STATE_DIR=/var/lib/tailscale
      - TS_EXTRA_ARGS=--advertise-tags=tag:container
    volumes:
      - tailscale-state:/var/lib/tailscale
      - /dev/net/tun:/dev/net/tun
    cap_add:
      - net_admin
      - sys_module
    restart: unless-stopped

volumes:
  tailscale-state:
```

### Kubernetes Deployment

```yaml
# Tailscale operator for Kubernetes
apiVersion: v1
kind: Secret
metadata:
  name: tailscale-auth
  namespace: tailscale
type: Opaque
stringData:
  TS_AUTHKEY: "tskey-auth-xxxxx"
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: tailscale
  namespace: tailscale
spec:
  selector:
    matchLabels:
      app: tailscale
  template:
    metadata:
      labels:
        app: tailscale
    spec:
      containers:
      - name: tailscale
        image: tailscale/tailscale:latest
        env:
        - name: TS_AUTHKEY
          valueFrom:
            secretKeyRef:
              name: tailscale-auth
              key: TS_AUTHKEY
        - name: TS_KUBE_SECRET
          value: tailscale-state
        - name: TS_USERSPACE
          value: "true"
        securityContext:
          capabilities:
            add: ["NET_ADMIN"]
```

## Access Control Lists (ACLs)

Tailscale ACLs define who can access what within your tailnet using a declarative JSON format. The default policy is deny-all, making it zero trust by design.

```json
{
  "acls": [
    // Engineering team can access development servers
    {
      "action": "accept",
      "src": ["group:engineering"],
      "dst": ["tag:dev-server:*"]
    },
    // SRE team can access production infrastructure
    {
      "action": "accept",
      "src": ["group:sre"],
      "dst": ["tag:production:22,443,8080"]
    },
    // Database access restricted to backend services
    {
      "action": "accept",
      "src": ["tag:backend"],
      "dst": ["tag:database:5432,3306,27017"]
    },
    // All employees can access internal tools
    {
      "action": "accept",
      "src": ["group:employees"],
      "dst": ["tag:internal-tools:443"]
    }
  ],

  "groups": {
    "group:engineering": ["user@company.com", "dev@company.com"],
    "group:sre": ["sre@company.com", "oncall@company.com"],
    "group:employees": ["autogroup:members"]
  },

  "tagOwners": {
    "tag:dev-server": ["group:engineering"],
    "tag:production": ["group:sre"],
    "tag:backend": ["group:sre"],
    "tag:database": ["group:sre"],
    "tag:internal-tools": ["group:sre"],
    "tag:container": ["group:sre"]
  },

  "ssh": [
    {
      "action": "check",
      "src": ["group:sre"],
      "dst": ["tag:production"],
      "users": ["root", "admin"]
    },
    {
      "action": "accept",
      "src": ["group:engineering"],
      "dst": ["tag:dev-server"],
      "users": ["autogroup:nonroot"]
    }
  ],

  "nodeAttrs": [
    {
      "target": ["autogroup:members"],
      "attr": ["funnel:deny"]
    }
  ]
}
```

## Exit Nodes and Subnet Routing

### Configure Exit Node

```bash
# On the exit node machine
sudo tailscale up --advertise-exit-node

# On the client machine, use the exit node
sudo tailscale up --exit-node=<exit-node-ip>

# Verify exit node routing
curl ifconfig.me  # Should show exit node's public IP
```

### Subnet Router Configuration

```bash
# Advertise local subnets through Tailscale
sudo tailscale up --advertise-routes=10.0.0.0/24,192.168.1.0/24

# Enable IP forwarding on Linux
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Accept routes on client
sudo tailscale up --accept-routes
```

## Tailscale SSH (Zero Trust SSH)

Tailscale SSH replaces traditional SSH key management with identity-based access.

```bash
# Enable Tailscale SSH on a server
sudo tailscale up --ssh

# Connect using Tailscale SSH (no SSH keys needed)
ssh user@hostname  # Authenticates via Tailscale identity

# Session recording (audit logging)
# Configure in ACL policy:
# "ssh": [{"action": "check", "src": [...], "dst": [...], "users": [...]}]
# "check" action requires re-authentication and records sessions
```

## MagicDNS Configuration

```bash
# MagicDNS is enabled by default in new tailnets
# Access devices by hostname instead of IP
ping my-server  # Resolves via MagicDNS

# Custom DNS configuration via admin console
# Split DNS: route specific domains to internal DNS servers
# Global nameservers: override default DNS resolution
```

## Self-Hosted with Headscale

```bash
# Install Headscale (open-source Tailscale control server)
wget https://github.com/juanfont/headscale/releases/latest/download/headscale_linux_amd64
chmod +x headscale_linux_amd64
sudo mv headscale_linux_amd64 /usr/local/bin/headscale

# Create configuration
sudo mkdir -p /etc/headscale
sudo headscale generate config > /etc/headscale/config.yaml

# Edit config for your environment
# Key settings:
#   server_url: https://headscale.example.com
#   listen_addr: 0.0.0.0:8080
#   private_key_path: /etc/headscale/private.key
#   db_type: sqlite3
#   db_path: /var/lib/headscale/db.sqlite

# Start Headscale
sudo headscale serve

# Create user and pre-auth key
headscale users create myorg
headscale preauthkeys create --user myorg --reusable --expiration 24h

# Connect Tailscale client to Headscale
tailscale up --login-server https://headscale.example.com
```

## Security Hardening

### Key Expiry and Rotation

```bash
# Set key expiry in admin console (default: 180 days)
# Force re-authentication periodically

# Disable key expiry for servers (use auth keys instead)
sudo tailscale up --authkey=tskey-auth-xxxxx

# Pre-auth keys for automated deployment
# Create ephemeral, single-use keys for CI/CD
```

### Device Authorization

```json
{
  "nodeAttrs": [
    {
      "target": ["autogroup:members"],
      "attr": [
        "mullvad:deny",
        "funnel:deny"
      ]
    }
  ],
  "autoApprovers": {
    "routes": {
      "10.0.0.0/24": ["group:sre"],
      "192.168.0.0/16": ["group:sre"]
    },
    "exitNode": ["group:sre"]
  }
}
```

### Network Lock (Tailnet Lock)

```bash
# Initialize network lock with signing keys
tailscale lock init

# Add trusted signing keys
tailscale lock add nodekey:xxxxx

# All new nodes require signing before joining
# Prevents unauthorized nodes from joining the tailnet
```

## Monitoring and Observability

```bash
# View network status
tailscale status --json | jq '.Peer | to_entries[] | {name: .value.HostName, online: .value.Online, os: .value.OS}'

# Check connection quality
tailscale ping <peer-ip>

# View network map
tailscale netcheck

# Audit logs available in Tailscale admin console
# Integration with SIEM via webhook or API
```

## Integration Patterns

### Service Mesh Integration

```bash
# Tailscale as sidecar for service-to-service communication
# Each service gets a Tailscale identity
# ACLs enforce service-to-service access policies

# Example: API service can only reach database service
# ACL: tag:api -> tag:database:5432
```

### CI/CD Pipeline Integration

```bash
# Use ephemeral auth keys in CI/CD
export TS_AUTHKEY=tskey-auth-xxxxx-ephemeral
tailscale up --authkey=$TS_AUTHKEY --hostname=ci-runner-$CI_JOB_ID

# Access internal resources during build/deploy
# Node automatically removed when container stops
```

## References

- [Tailscale Documentation](https://tailscale.com/kb/)
- [How Tailscale Works](https://tailscale.com/blog/how-tailscale-works)
- [Tailscale ACL Documentation](https://tailscale.com/kb/1018/acls/)
- [Headscale - Open Source Control Server](https://github.com/juanfont/headscale)
- [WireGuard Protocol](https://www.wireguard.com/protocol/)
- [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh/)

===== END FILE =====


===== FILE: skills/deploying-tailscale-for-zero-trust-vpn/LICENSE =====

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


===== FILE: skills/deploying-tailscale-for-zero-trust-vpn/assets/template.md =====
# Tailscale Deployment Planning Template

## Network Architecture

- **Organization**: _______________
- **Tailnet Name**: _______________
- **Identity Provider**: _______________
- **Key Expiry Policy**: _______________
- **Self-hosted (Headscale)**: [ ] Yes [ ] No

## User Groups

| Group Name | Description | Members Count | Access Level |
|---|---|---|---|
| group:engineering | Development team | ___ | Development, Staging |
| group:sre | SRE/DevOps team | ___ | All environments |
| group:security | Security team | ___ | Monitoring, Audit |
| group:management | Leadership | ___ | Dashboards only |

## Infrastructure Tags

| Tag | Description | Owner Group | Environment |
|---|---|---|---|
| tag:production | Production servers | group:sre | Production |
| tag:staging | Staging servers | group:engineering | Staging |
| tag:development | Dev servers | group:engineering | Development |
| tag:database | Database servers | group:sre | All |
| tag:monitoring | Monitoring stack | group:sre | All |

## Subnet Routes

| CIDR | Description | Router Node | Auto-Approved |
|---|---|---|---|
| 10.0.0.0/16 | Corporate network | ___ | [ ] Yes |
| 192.168.0.0/24 | Lab network | ___ | [ ] Yes |

## Exit Nodes

| Hostname | Location | Purpose | Auto-Approved |
|---|---|---|---|
| ___ | ___ | Internet routing | [ ] Yes |
| ___ | ___ | Geo-specific access | [ ] Yes |

## Security Checklist

- [ ] Identity provider configured with MFA
- [ ] Key expiry enabled (recommended: 90 days)
- [ ] ACLs configured with deny-all default
- [ ] Network Lock enabled
- [ ] SSH access requires re-authentication for privileged users
- [ ] Audit logging enabled
- [ ] Subnet routes approved only for authorized nodes
- [ ] Exit nodes approved only for authorized nodes
- [ ] Untagged node policy defined
- [ ] Ephemeral keys used for CI/CD and temporary workloads

## Rollout Plan

### Phase 1: Infrastructure
- [ ] Deploy to servers and critical infrastructure
- [ ] Configure subnet routers
- [ ] Set up exit nodes
- [ ] Test ACL enforcement

### Phase 2: User Onboarding
- [ ] Pilot group deployment
- [ ] Full organization rollout
- [ ] VPN migration (decommission legacy VPN)
- [ ] User training and documentation

### Phase 3: Hardening
- [ ] Enable Network Lock
- [ ] Enable Tailscale SSH with session recording
- [ ] Configure auto-approvers
- [ ] Set up monitoring and alerting

===== END FILE =====


===== FILE: skills/deploying-tailscale-for-zero-trust-vpn/references/api-reference.md =====
# Tailscale Zero Trust VPN — API Reference

## Libraries

| Library | Install | Purpose |
|---------|---------|---------|
| requests | `pip install requests` | Tailscale API v2 client |

## Tailscale API v2 Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v2/tailnet/{tailnet}/devices` | List all devices in tailnet |
| GET | `/api/v2/tailnet/{tailnet}/acl` | Get ACL policy |
| PUT | `/api/v2/tailnet/{tailnet}/acl` | Update ACL policy |
| GET | `/api/v2/tailnet/{tailnet}/dns/nameservers` | Get DNS nameservers |
| GET | `/api/v2/tailnet/{tailnet}/keys` | List auth keys |
| GET | `/api/v2/device/{deviceid}` | Get device details |
| DELETE | `/api/v2/device/{deviceid}` | Remove device from tailnet |
| GET | `/api/v2/tailnet/{tailnet}/webhooks` | List webhooks |

## Base URL & Authentication

```
Base: https://api.tailscale.com
Header: Authorization: Bearer <api-key>
```

## ACL Policy Structure

| Field | Description |
|-------|-------------|
| `acls` | Access control rules (src, dst, action) |
| `groups` | Named groups of users |
| `tagOwners` | Tag-based device ownership |
| `ssh` | Tailscale SSH access rules |
| `autoApprovers` | Auto-approve routes and exit nodes |
| `tests` | ACL policy unit tests |

## Device Fields

| Field | Description |
|-------|-------------|
| `hostname` | Device hostname |
| `os` | Operating system |
| `clientVersion` | Tailscale client version |
| `keyExpiryDisabled` | Whether key expiry is disabled |
| `online` | Current online status |
| `lastSeen` | Last seen timestamp |
| `addresses` | Tailscale IP addresses |

## External References

- [Tailscale API Docs](https://tailscale.com/api)
- [Tailscale ACL Policy](https://tailscale.com/kb/1018/acls)
- [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh)

===== END FILE =====


===== FILE: skills/deploying-tailscale-for-zero-trust-vpn/references/standards.md =====
# Standards Reference: Tailscale Zero Trust VPN

## Protocol Standards

### WireGuard Protocol
- **Encryption**: ChaCha20 for symmetric encryption
- **Key Exchange**: Curve25519 for Diffie-Hellman
- **MAC**: Poly1305 for message authentication
- **Hashing**: BLAKE2s for hashing
- **Framework**: Noise Protocol Framework for key negotiation

### NIST SP 800-207: Zero Trust Architecture
- Tailscale implements identity-aware proxying (Section 3.2.2)
- End-to-end encryption satisfies data-in-transit requirements
- ACL-based access control implements least privilege access
- Device identity via WireGuard keys maps to device trust

### NIST SP 800-77: Guide to IPsec VPNs
- WireGuard provides alternative to IPsec with reduced complexity
- Tailscale automates key distribution and NAT traversal
- Mesh topology eliminates single point of failure

## Tailscale Security Model

### Identity Layer
- Authentication via OIDC-compatible identity providers
- SSO integration with Okta, Azure AD, Google Workspace, GitHub
- MFA enforcement through identity provider policies
- Key expiry forces periodic re-authentication

### Network Layer
- Default deny ACL policy (zero trust)
- Per-connection authorization based on identity and tags
- No implicit trust based on network location
- All traffic encrypted with WireGuard (256-bit keys)

### Device Layer
- Unique WireGuard key pair per device
- Device authorization required before network access
- Network Lock prevents unauthorized node addition
- Ephemeral nodes for temporary workloads

## Compliance Considerations

### SOC 2
- End-to-end encryption for data in transit
- ACL-based access control for authorization
- Audit logging for all connection events
- Key management through coordination server

### GDPR
- Data minimization: Tailscale only routes traffic, does not inspect
- Encryption: All traffic encrypted end-to-end
- Self-hosted option (Headscale) for data sovereignty
- Log retention configurable per organization policy

===== END FILE =====


===== FILE: skills/deploying-tailscale-for-zero-trust-vpn/references/workflows.md =====
# Workflows: Deploying Tailscale for Zero Trust VPN

## Workflow 1: Initial Tailnet Deployment

```
Step 1: Plan Network Architecture
  - Identify all devices and services requiring connectivity
  - Map existing network topology and access requirements
  - Define user groups and access policies
  - Plan subnet routing for legacy network integration
  - Determine exit node placement for internet routing

Step 2: Configure Identity Provider
  - Enable SSO with organizational identity provider
  - Configure MFA enforcement policies
  - Map identity provider groups to Tailscale groups
  - Set key expiry policy (recommended: 90 days)

Step 3: Deploy Tailscale Nodes
  - Install on critical infrastructure first (servers, databases)
  - Deploy to user endpoints (laptops, mobile devices)
  - Configure subnet routers for non-Tailscale networks
  - Set up exit nodes for secure internet access
  - Enable MagicDNS for hostname resolution

Step 4: Configure ACLs
  - Start with deny-all baseline
  - Define groups matching organizational structure
  - Create tag-based policies for infrastructure
  - Test ACLs in audit mode before enforcement
  - Document all ACL rules and their business justification

Step 5: Validate and Monitor
  - Test connectivity between all required paths
  - Verify ACL enforcement blocks unauthorized access
  - Enable audit logging
  - Configure alerts for connection anomalies
```

## Workflow 2: ACL Policy Development

```
Step 1: Inventory Access Requirements
  - List all user roles and their resource needs
  - Map application dependencies (service-to-service)
  - Identify privileged access paths
  - Document temporary/exception access needs

Step 2: Design Policy Structure
  - Define groups (users, teams, roles)
  - Define tags (environments, service types, sensitivity)
  - Map access rules: group/tag -> destination:ports
  - Plan SSH access policies with session recording

Step 3: Implement and Test
  - Write ACL JSON configuration
  - Deploy in test/staging tailnet first
  - Validate each rule with test connections
  - Verify deny rules block unauthorized access
  - Review with security team before production deployment

Step 4: Maintain and Audit
  - Review ACLs quarterly for stale rules
  - Audit access logs for policy violations
  - Update groups when team membership changes
  - Remove deprecated rules and tags
```

## Workflow 3: Headscale Self-Hosted Deployment

```
Step 1: Prepare Infrastructure
  - Provision server with public IP and domain
  - Configure TLS certificate (Let's Encrypt)
  - Set up PostgreSQL or SQLite database
  - Configure firewall rules (port 443, DERP relay ports)

Step 2: Install and Configure Headscale
  - Download latest Headscale binary
  - Generate configuration file
  - Configure OIDC provider integration
  - Set up DNS records for coordination server
  - Configure DERP relay servers

Step 3: Onboard Users and Devices
  - Create users/namespaces in Headscale
  - Generate pre-auth keys for automated deployment
  - Connect client devices to Headscale server
  - Configure ACLs via Headscale policy file

Step 4: Operational Maintenance
  - Monitor Headscale server health
  - Rotate pre-auth keys regularly
  - Backup database and configuration
  - Update Headscale and client versions
  - Review and rotate DERP relay configuration
```

===== END FILE =====


===== FILE: skills/deploying-tailscale-for-zero-trust-vpn/scripts/agent.py =====
#!/usr/bin/env python3
"""Tailscale zero trust VPN deployment audit agent."""

import json
import sys
import argparse
from datetime import datetime

try:
    import requests
except ImportError:
    print("Install: pip install requests")
    sys.exit(1)


class TailscaleClient:
    """Client for Tailscale API v2."""

    def __init__(self, api_key, tailnet="-"):
        self.base_url = "https://api.tailscale.com/api/v2"
        self.tailnet = tailnet
        self.headers = {"Authorization": f"Bearer {api_key}"}

    def _get(self, path):
        resp = requests.get(f"{self.base_url}{path}",
                            headers=self.headers, timeout=15)
        resp.raise_for_status()
        return resp.json()

    def list_devices(self):
        return self._get(f"/tailnet/{self.tailnet}/devices").get("devices", [])

    def get_acl(self):
        return self._get(f"/tailnet/{self.tailnet}/acl")

    def list_dns(self):
        return self._get(f"/tailnet/{self.tailnet}/dns/nameservers")

    def get_key_expiry_disabled(self):
        return self._get(f"/tailnet/{self.tailnet}/keys")

    def list_webhooks(self):
        return self._get(f"/tailnet/{self.tailnet}/webhooks").get("webhooks", [])


def audit_devices(devices):
    """Audit device compliance: OS versions, key expiry, last seen."""
    findings = []
    now = datetime.utcnow()
    for dev in devices:
        hostname = dev.get("hostname", "unknown")
        if dev.get("keyExpiryDisabled", False):
            findings.append({
                "device": hostname,
                "issue": "Key expiry disabled — device never requires re-authentication",
                "severity": "HIGH",
            })
        if not dev.get("updateAvailable", False) is False and dev.get("updateAvailable"):
            findings.append({
                "device": hostname,
                "issue": "Tailscale update available but not installed",
                "severity": "MEDIUM",
            })
        os_name = dev.get("os", "")
        if dev.get("blocksIncomingConnections", False):
            findings.append({
                "device": hostname,
                "issue": "Device blocks incoming connections (shields up mode)",
                "severity": "INFO",
            })
    return findings


def audit_acl(acl_data):
    """Check ACL policy for overly permissive rules."""
    findings = []
    acls = acl_data.get("acls", []) if isinstance(acl_data, dict) else []
    for i, rule in enumerate(acls):
        src = rule.get("src", [])
        dst = rule.get("dst", [])
        if "*" in src and any("*:*" in d for d in dst):
            findings.append({
                "rule_index": i,
                "issue": "Allow-all rule: src=* dst=*:* — no zero trust segmentation",
                "severity": "CRITICAL",
            })
    ssh_rules = acl_data.get("ssh", []) if isinstance(acl_data, dict) else []
    for rule in ssh_rules:
        if rule.get("action") == "accept" and "*" in rule.get("src", []):
            findings.append({
                "rule": "SSH",
                "issue": "SSH access allowed from all users",
                "severity": "HIGH",
            })
    return findings


def run_audit(api_key, tailnet):
    """Execute Tailscale zero trust audit."""
    print(f"\n{'='*60}")
    print(f"  TAILSCALE ZERO TRUST VPN AUDIT")
    print(f"  Generated: {datetime.utcnow().isoformat()} UTC")
    print(f"{'='*60}\n")

    client = TailscaleClient(api_key, tailnet)
    report = {}

    devices = client.list_devices()
    report["total_devices"] = len(devices)
    print(f"--- DEVICES ({len(devices)}) ---")
    for d in devices[:15]:
        print(f"  {d.get('hostname','')}: {d.get('os','')}/{d.get('clientVersion','')} "
              f"({'online' if d.get('online') else 'offline'})")

    dev_findings = audit_devices(devices)
    report["device_findings"] = dev_findings
    print(f"\n--- DEVICE FINDINGS ({len(dev_findings)}) ---")
    for f in dev_findings[:10]:
        print(f"  [{f['severity']}] {f['device']}: {f['issue']}")

    acl_data = client.get_acl()
    acl_findings = audit_acl(acl_data)
    report["acl_findings"] = acl_findings
    print(f"\n--- ACL POLICY FINDINGS ({len(acl_findings)}) ---")
    for f in acl_findings[:10]:
        print(f"  [{f['severity']}] {f['issue']}")

    dns = client.list_dns()
    report["dns_config"] = dns
    print(f"\n--- DNS CONFIG ---")
    for ns in dns.get("dns", []):
        print(f"  Nameserver: {ns}")

    return report


def main():
    parser = argparse.ArgumentParser(description="Tailscale Zero Trust Audit")
    parser.add_argument("--api-key", required=True, help="Tailscale API key")
    parser.add_argument("--tailnet", default="-", help="Tailnet name (default: current)")
    parser.add_argument("--output", help="Save report to JSON file")
    args = parser.parse_args()

    report = run_audit(args.api_key, args.tailnet)
    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"\n[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/deploying-tailscale-for-zero-trust-vpn/scripts/process.py =====
#!/usr/bin/env python3
"""
Tailscale Zero Trust VPN Management and Monitoring.

Manages Tailscale deployment, ACL generation, network health monitoring,
and compliance reporting for zero trust mesh VPN infrastructure.
"""

import json
import subprocess
import datetime
from dataclasses import dataclass, field
from pathlib import Path
from typing import Optional


@dataclass
class TailscaleNode:
    hostname: str
    ip4: str
    ip6: str = ""
    os: str = ""
    online: bool = False
    tags: list = field(default_factory=list)
    key_expiry: str = ""
    last_seen: str = ""
    exit_node: bool = False
    subnet_routes: list = field(default_factory=list)


@dataclass
class ACLRule:
    action: str  # "accept" or "deny"
    src: list
    dst: list
    description: str = ""


@dataclass
class ACLGroup:
    name: str
    members: list


class TailscaleACLGenerator:
    """Generate and validate Tailscale ACL configurations."""

    def __init__(self):
        self.groups: dict[str, list] = {}
        self.tag_owners: dict[str, list] = {}
        self.acls: list[dict] = []
        self.ssh_rules: list[dict] = []
        self.auto_approvers: dict = {"routes": {}, "exitNode": []}
        self.node_attrs: list[dict] = []

    def add_group(self, name: str, members: list):
        if not name.startswith("group:"):
            name = f"group:{name}"
        self.groups[name] = members

    def add_tag(self, tag: str, owners: list):
        if not tag.startswith("tag:"):
            tag = f"tag:{tag}"
        self.tag_owners[tag] = owners

    def add_acl_rule(self, src: list, dst: list, action: str = "accept"):
        self.acls.append({"action": action, "src": src, "dst": dst})

    def add_ssh_rule(self, src: list, dst: list, users: list, action: str = "check"):
        self.ssh_rules.append({
            "action": action,
            "src": src,
            "dst": dst,
            "users": users,
        })

    def add_auto_approver_route(self, cidr: str, approvers: list):
        self.auto_approvers["routes"][cidr] = approvers

    def add_auto_approver_exit_node(self, approvers: list):
        self.auto_approvers["exitNode"] = approvers

    def generate_policy(self) -> dict:
        policy = {}
        if self.groups:
            policy["groups"] = self.groups
        if self.tag_owners:
            policy["tagOwners"] = self.tag_owners
        if self.acls:
            policy["acls"] = self.acls
        if self.ssh_rules:
            policy["ssh"] = self.ssh_rules
        if self.auto_approvers["routes"] or self.auto_approvers["exitNode"]:
            policy["autoApprovers"] = self.auto_approvers
        if self.node_attrs:
            policy["nodeAttrs"] = self.node_attrs
        return policy

    def export_policy(self, output_path: str):
        policy = self.generate_policy()
        path = Path(output_path)
        path.parent.mkdir(parents=True, exist_ok=True)
        with open(path, "w") as f:
            json.dump(policy, f, indent=2)
        return policy

    def validate_policy(self) -> list:
        """Validate ACL policy for common issues."""
        issues = []
        policy = self.generate_policy()

        # Check for empty ACLs
        if not policy.get("acls"):
            issues.append("WARNING: No ACL rules defined - all traffic will be denied")

        # Check for overly permissive rules
        for i, rule in enumerate(self.acls):
            if "*" in rule.get("src", []) and "*" in rule.get("dst", []):
                issues.append(f"CRITICAL: Rule {i} allows all-to-all access")
            for dst in rule.get("dst", []):
                if dst.endswith(":*"):
                    issues.append(f"WARNING: Rule {i} allows all ports to {dst}")

        # Check groups reference valid members
        for group_name, members in self.groups.items():
            if not members:
                issues.append(f"WARNING: Group {group_name} has no members")

        # Check tag owners exist
        for tag, owners in self.tag_owners.items():
            for owner in owners:
                if owner.startswith("group:") and owner not in self.groups:
                    issues.append(f"ERROR: Tag {tag} references undefined group {owner}")

        # Check SSH rules
        for i, rule in enumerate(self.ssh_rules):
            if "root" in rule.get("users", []) and rule.get("action") != "check":
                issues.append(f"WARNING: SSH rule {i} allows root access without re-auth check")

        return issues


class TailscaleMonitor:
    """Monitor Tailscale network health and compliance."""

    def __init__(self):
        self.nodes: list[TailscaleNode] = []

    def get_status(self) -> dict:
        """Get current Tailscale status via CLI."""
        try:
            result = subprocess.run(
                ["tailscale", "status", "--json"],
                capture_output=True, text=True, timeout=10
            )
            if result.returncode == 0:
                return json.loads(result.stdout)
        except (subprocess.TimeoutExpired, FileNotFoundError, json.JSONDecodeError):
            pass
        return {}

    def parse_nodes(self, status: dict) -> list[TailscaleNode]:
        """Parse Tailscale status into node objects."""
        self.nodes = []
        peers = status.get("Peer", {})
        for peer_id, peer_data in peers.items():
            node = TailscaleNode(
                hostname=peer_data.get("HostName", "unknown"),
                ip4=peer_data.get("TailscaleIPs", [""])[0] if peer_data.get("TailscaleIPs") else "",
                ip6=peer_data.get("TailscaleIPs", ["", ""])[1] if len(peer_data.get("TailscaleIPs", [])) > 1 else "",
                os=peer_data.get("OS", ""),
                online=peer_data.get("Online", False),
                tags=peer_data.get("Tags", []),
                key_expiry=peer_data.get("KeyExpiry", ""),
                last_seen=peer_data.get("LastSeen", ""),
                exit_node=peer_data.get("ExitNode", False),
            )
            self.nodes.append(node)
        return self.nodes

    def check_health(self) -> dict:
        """Run health checks on the tailnet."""
        report = {
            "timestamp": datetime.datetime.now().isoformat(),
            "total_nodes": len(self.nodes),
            "online_nodes": sum(1 for n in self.nodes if n.online),
            "offline_nodes": sum(1 for n in self.nodes if not n.online),
            "expiring_keys": [],
            "untagged_nodes": [],
            "exit_nodes": [],
            "issues": [],
        }

        for node in self.nodes:
            # Check for expiring keys
            if node.key_expiry:
                try:
                    expiry = datetime.datetime.fromisoformat(node.key_expiry.replace("Z", "+00:00"))
                    days_until = (expiry - datetime.datetime.now(datetime.timezone.utc)).days
                    if days_until < 30:
                        report["expiring_keys"].append({
                            "hostname": node.hostname,
                            "expires_in_days": days_until,
                        })
                except (ValueError, TypeError):
                    pass

            # Check for untagged nodes
            if not node.tags:
                report["untagged_nodes"].append(node.hostname)

            # Track exit nodes
            if node.exit_node:
                report["exit_nodes"].append(node.hostname)

        # Generate issues
        if report["offline_nodes"] > 0:
            report["issues"].append(
                f"{report['offline_nodes']} nodes offline"
            )
        if report["expiring_keys"]:
            report["issues"].append(
                f"{len(report['expiring_keys'])} nodes with keys expiring within 30 days"
            )
        if report["untagged_nodes"]:
            report["issues"].append(
                f"{len(report['untagged_nodes'])} nodes without tags (ungoverned by ACLs)"
            )

        return report

    def generate_compliance_report(self) -> dict:
        """Generate zero trust compliance report for the tailnet."""
        report = {
            "report_date": datetime.datetime.now().isoformat(),
            "zero_trust_checks": {
                "encryption": {
                    "status": "PASS",
                    "detail": "All connections use WireGuard end-to-end encryption",
                },
                "identity_based_access": {
                    "status": "PASS" if all(n.tags for n in self.nodes) else "FAIL",
                    "detail": "All nodes should have identity tags for ACL enforcement",
                },
                "least_privilege": {
                    "status": "REVIEW",
                    "detail": "ACL policy review required - validate minimum necessary access",
                },
                "continuous_verification": {
                    "status": "PASS" if not any(
                        n.key_expiry == "" for n in self.nodes
                    ) else "WARNING",
                    "detail": "Key expiry should be enabled for all non-server nodes",
                },
                "device_trust": {
                    "status": "REVIEW",
                    "detail": "Verify device authorization and Network Lock status",
                },
            },
        }
        return report


def generate_example_policy():
    """Generate an example zero trust ACL policy for Tailscale."""
    gen = TailscaleACLGenerator()

    # Define groups
    gen.add_group("engineering", ["eng1@company.com", "eng2@company.com"])
    gen.add_group("sre", ["sre1@company.com", "sre2@company.com"])
    gen.add_group("security", ["sec1@company.com"])
    gen.add_group("management", ["mgr1@company.com"])

    # Define tags
    gen.add_tag("production", ["group:sre"])
    gen.add_tag("staging", ["group:engineering", "group:sre"])
    gen.add_tag("development", ["group:engineering"])
    gen.add_tag("database", ["group:sre"])
    gen.add_tag("monitoring", ["group:sre", "group:security"])
    gen.add_tag("ci-runner", ["group:sre"])

    # ACL rules - zero trust, least privilege
    gen.add_acl_rule(
        src=["group:engineering"],
        dst=["tag:development:*", "tag:staging:443,8080"]
    )
    gen.add_acl_rule(
        src=["group:sre"],
        dst=["tag:production:22,443,8080", "tag:staging:*", "tag:database:5432,3306"]
    )
    gen.add_acl_rule(
        src=["group:security"],
        dst=["tag:monitoring:443,9090", "tag:production:443"]
    )
    gen.add_acl_rule(
        src=["tag:ci-runner"],
        dst=["tag:staging:443,8080", "tag:production:443"]
    )

    # SSH rules with re-authentication
    gen.add_ssh_rule(
        src=["group:sre"],
        dst=["tag:production"],
        users=["admin"],
        action="check"  # Requires re-auth, records session
    )
    gen.add_ssh_rule(
        src=["group:engineering"],
        dst=["tag:development"],
        users=["autogroup:nonroot"],
        action="accept"
    )

    # Auto-approvers for subnet routes
    gen.add_auto_approver_route("10.0.0.0/16", ["group:sre"])
    gen.add_auto_approver_exit_node(["group:sre"])

    # Validate
    issues = gen.validate_policy()
    if issues:
        print("Policy Validation Issues:")
        for issue in issues:
            print(f"  - {issue}")
    else:
        print("Policy validation: PASSED")

    # Export
    policy = gen.export_policy("tailscale_acl_policy.json")
    print(f"\nGenerated ACL policy with:")
    print(f"  Groups: {len(gen.groups)}")
    print(f"  Tags: {len(gen.tag_owners)}")
    print(f"  ACL Rules: {len(gen.acls)}")
    print(f"  SSH Rules: {len(gen.ssh_rules)}")
    print(f"\nPolicy saved to: tailscale_acl_policy.json")
    print(f"\nPolicy preview:")
    print(json.dumps(policy, indent=2))


if __name__ == "__main__":
    generate_example_policy()

===== END FILE =====


===== FILE: skills/implementing-beyondcorp-zero-trust-access-model/SKILL.md =====
---
name: implementing-beyondcorp-zero-trust-access-model
description: 'Implement Google''s BeyondCorp zero trust access model using Cloud
  IAP, Access Context Manager, Endpoint Verification, Chrome Enterprise Premium, and
  BeyondCorp Enterprise Connectors to enforce identity- and device-aware access for
  VPN-less application access. Use for replacing VPN, enforcing device posture checks,
  or securing remote/hybrid access to GCP-hosted or on-prem apps; not for raw network-level
  protocols.

  '
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- beyondcorp
- zero-trust
- google-cloud
- iap
- identity-aware-proxy
- ztna
- access-context-manager
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078
- T1190
- T1059
- T1078.004
- T1530
---

# Implementing BeyondCorp Zero Trust Access Model

## When to Use

- When replacing traditional VPN infrastructure with identity-based application access
- When migrating to Google Cloud and requiring zero trust access for internal applications
- When implementing device trust verification as a prerequisite for resource access
- When needing context-aware access policies based on user identity, device posture, and location
- When securing access for remote and hybrid workforce without network-level trust

**Do not use** when applications require raw network-level access (e.g., UDP-based protocols not supported by IAP), for consumer-facing public applications, or when the organization lacks an identity provider with MFA capabilities.

## Prerequisites

- Google Cloud organization with Cloud Identity or Google Workspace
- Identity-Aware Proxy (IAP) API enabled on the GCP project
- Chrome Enterprise Premium license for endpoint verification
- Applications deployed behind a Google Cloud Load Balancer or on App Engine/Cloud Run
- Endpoint Verification extension deployed on all corporate devices
- Access Context Manager API enabled

## Workflow

### Step 1: Configure Access Context Manager with Access Levels

Define access levels that represent trust tiers based on device and user attributes.

```bash
# Enable required APIs
gcloud services enable iap.googleapis.com
gcloud services enable accesscontextmanager.googleapis.com
gcloud services enable beyondcorp.googleapis.com

# Create an access policy (organization level)
gcloud access-context-manager policies create \
  --organization=ORG_ID \
  --title="BeyondCorp Enterprise Policy"

# Create a basic access level for corporate managed devices
cat > corporate-device-level.yaml << 'EOF'
- devicePolicy:
    allowedEncryptionStatuses:
      - ENCRYPTED
    osConstraints:
      - osType: DESKTOP_CHROME_OS
        minimumVersion: "13816.0.0"
      - osType: DESKTOP_WINDOWS
        minimumVersion: "10.0.19045"
      - osType: DESKTOP_MAC
        minimumVersion: "13.0.0"
    requireScreenlock: true
    requireAdminApproval: true
  regions:
    - US
    - GB
    - DE
EOF

gcloud access-context-manager levels create corporate-managed \
  --policy=POLICY_ID \
  --title="Corporate Managed Device" \
  --basic-level-spec=corporate-device-level.yaml

# Create a custom access level using CEL expressions
gcloud access-context-manager levels create high-trust \
  --policy=POLICY_ID \
  --title="High Trust Level" \
  --custom-level-spec=high-trust-cel.yaml
```

### Step 2: Deploy Identity-Aware Proxy on Applications

Enable IAP on backend services to enforce identity verification before granting access.

```bash
# Create OAuth consent screen
gcloud iap oauth-brands create \
  --application_title="Corporate Applications" \
  --support_email=security@company.com

# Create OAuth client for IAP
gcloud iap oauth-clients create BRAND_NAME \
  --display_name="BeyondCorp IAP Client"

# Enable IAP on a backend service (GCE/GKE behind HTTPS LB)
gcloud compute backend-services update internal-app-backend \
  --iap=enabled,oauth2-client-id=CLIENT_ID,oauth2-client-secret=CLIENT_SECRET \
  --global

# Enable IAP on App Engine
gcloud iap web enable \
  --resource-type=app-engine \
  --oauth2-client-id=CLIENT_ID \
  --oauth2-client-secret=CLIENT_SECRET

# Enable IAP on Cloud Run service
gcloud run services add-iam-policy-binding internal-api \
  --member="serviceAccount:service-PROJECT_NUM@gcp-sa-iap.iam.gserviceaccount.com" \
  --role="roles/run.invoker" \
  --region=us-central1
```

### Step 3: Configure IAM Bindings with Access Level Conditions

Bind IAP access to specific groups with access level requirements.

```bash
# Grant access to engineering group with corporate device requirement
gcloud iap web add-iam-policy-binding \
  --resource-type=backend-services \
  --service=internal-app-backend \
  --member="group:engineering@company.com" \
  --role="roles/iap.httpsResourceAccessor" \
  --condition="expression=accessPolicies/POLICY_ID/accessLevels/corporate-managed,title=Require Corporate Device"

# Grant access to contractors with high-trust requirement
gcloud iap web add-iam-policy-binding \
  --resource-type=backend-services \
  --service=internal-app-backend \
  --member="group:contractors@company.com" \
  --role="roles/iap.httpsResourceAccessor" \
  --condition="expression=accessPolicies/POLICY_ID/accessLevels/high-trust,title=Require High Trust"

# Configure re-authentication settings (session duration)
gcloud iap settings set --project=PROJECT_ID \
  --resource-type=compute \
  --service=internal-app-backend \
  --reauth-method=LOGIN \
  --max-session-duration=3600s
```

### Step 4: Deploy Endpoint Verification on Corporate Devices

Roll out Chrome Enterprise Endpoint Verification for device posture collection.

```bash
# Deploy Endpoint Verification via Chrome policy (managed browsers)
# In Google Admin Console > Devices > Chrome > Apps & extensions
# Force-install: Endpoint Verification extension ID: callobklhcbilhphinckomhgkigmfocg

# Verify device inventory in Admin SDK
gcloud endpoint-verification list-endpoints \
  --filter="deviceType=CHROME_BROWSER" \
  --format="table(deviceId, osVersion, isCompliant, encryptionStatus)"

# Create device trust connector for third-party EDR signals
gcloud beyondcorp app connections create crowdstrike-connector \
  --project=PROJECT_ID \
  --location=global \
  --application-endpoint=host=crowdstrike-api.internal:443,port=443 \
  --type=TCP_PROXY_TUNNEL \
  --connectors=projects/PROJECT_ID/locations/us-central1/connectors/connector-1

# List enrolled devices and their compliance status
gcloud alpha devices list --format="table(name,deviceType,complianceState)"
```

### Step 5: Implement BeyondCorp Enterprise Threat Protection

Enable URL filtering, malware scanning, and DLP for Chrome Enterprise users.

```bash
# Configure Chrome Enterprise Premium threat protection rules
# In Google Admin Console > Security > Chrome Enterprise Premium

# Create a BeyondCorp Enterprise connector for on-prem apps
gcloud beyondcorp app connectors create onprem-connector \
  --project=PROJECT_ID \
  --location=us-central1 \
  --display-name="On-Premises App Connector"

gcloud beyondcorp app connections create hr-portal \
  --project=PROJECT_ID \
  --location=us-central1 \
  --application-endpoint=host=hr.internal.company.com,port=443 \
  --type=TCP_PROXY_TUNNEL \
  --connectors=projects/PROJECT_ID/locations/us-central1/connectors/onprem-connector

# Enable security investigation tool for access anomaly detection
gcloud logging read '
  resource.type="iap_tunnel"
  jsonPayload.decision="DENY"
  timestamp >= "2026-02-22T00:00:00Z"
' --project=PROJECT_ID --format=json --limit=100
```

### Step 6: Monitor and Audit BeyondCorp Access Decisions

Set up comprehensive logging and alerting for zero trust policy enforcement.

```bash
# Create a log sink for IAP access decisions
gcloud logging sinks create iap-access-audit \
  --destination=bigquery.googleapis.com/projects/PROJECT_ID/datasets/beyondcorp_audit \
  --log-filter='resource.type="iap_tunnel" OR resource.type="gce_backend_service"'

# Query BigQuery for access pattern analysis
bq query --use_legacy_sql=false '
SELECT
  protopayload_auditlog.authenticationInfo.principalEmail AS user,
  resource.labels.backend_service_name AS application,
  JSON_EXTRACT_SCALAR(protopayload_auditlog.requestMetadata.callerSuppliedUserAgent, "$") AS device,
  protopayload_auditlog.status.code AS decision_code,
  COUNT(*) AS request_count
FROM `PROJECT_ID.beyondcorp_audit.cloudaudit_googleapis_com_data_access`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
GROUP BY user, application, device, decision_code
ORDER BY request_count DESC
LIMIT 50
'

# Create an alert policy for repeated access denials
gcloud alpha monitoring policies create \
  --display-name="BeyondCorp Repeated Access Denials" \
  --condition-display-name="High denial rate" \
  --condition-filter='resource.type="iap_tunnel" AND jsonPayload.decision="DENY"' \
  --condition-threshold-value=10 \
  --condition-threshold-duration=300s \
  --notification-channels=projects/PROJECT_ID/notificationChannels/CHANNEL_ID
```

## Key Concepts

| Term | Definition |
|------|------------|
| BeyondCorp | Google's zero trust security framework that shifts access controls from network perimeter to per-request identity and device verification |
| Identity-Aware Proxy (IAP) | Google Cloud service that intercepts HTTP requests and verifies user identity and device context before forwarding to backend applications |
| Access Context Manager | GCP service that defines fine-grained attribute-based access control policies using access levels and service perimeters |
| Endpoint Verification | Chrome Enterprise extension that collects device attributes (OS version, encryption, screen lock) for access level evaluation |
| Access Levels | Named conditions in Access Context Manager that define minimum requirements (device posture, IP range, geography) for resource access |
| Chrome Enterprise Premium | Google's commercial BeyondCorp offering providing threat protection, URL filtering, DLP, and continuous access evaluation |

## Tools & Systems

- **Google Cloud IAP**: Identity-aware reverse proxy enforcing per-request authentication and authorization for GCP-hosted applications
- **Access Context Manager**: Policy engine defining access levels based on device attributes, IP ranges, and geographic locations
- **Chrome Enterprise Premium**: Extended BeyondCorp capabilities including real-time threat protection and data loss prevention
- **Endpoint Verification**: Device posture collection agent deployed as Chrome extension to all corporate endpoints
- **BeyondCorp Enterprise Connectors**: Secure tunnel connectors enabling IAP protection for on-premises applications
- **Cloud Audit Logs**: Immutable log records of all IAP access decisions for compliance and forensic analysis

## Common Scenarios

### Scenario: Migrating 50+ Internal Applications from VPN to BeyondCorp

**Context**: A technology company with 3,000 employees uses Cisco AnyConnect VPN for accessing internal applications. The VPN introduces latency, creates a single point of failure, and grants excessive network access after authentication.

**Approach**:
1. Inventory all 50+ applications and categorize by hosting (GCP, on-prem, SaaS) and protocol (HTTPS, TCP, SSH)
2. Deploy Endpoint Verification to all corporate devices and establish baseline device posture data over 2 weeks
3. Create access levels in Access Context Manager: corporate-managed, contractor-device, high-trust
4. Enable IAP on GCP-hosted HTTPS applications first (App Engine, Cloud Run, GKE services)
5. Deploy BeyondCorp Enterprise connectors for on-premises applications
6. Migrate users in 3 phases: IT/Engineering (week 1-2), General staff (week 3-4), Executives/Finance (week 5-6)
7. Configure re-authentication policies: 8 hours for general apps, 1 hour for financial systems
8. Set up BigQuery audit pipeline for continuous monitoring and anomaly detection
9. Decommission VPN after 30-day parallel operation period

**Pitfalls**: Some legacy applications may not support HTTPS proxying and require TCP tunnel mode. Device enrollment takes time; plan a 2-week onboarding period before enforcing device posture requirements. Break-glass accounts with bypassed access levels must be created and tested for identity provider outages.

## Output Format

```
BeyondCorp Zero Trust Implementation Report
==================================================
Organization: TechCorp Inc.
Implementation Date: 2026-02-23
Migration Phase: Phase 2 of 3

ACCESS ARCHITECTURE:
  Identity Provider: Google Workspace
  Access Proxy: Google Cloud IAP
  Device Management: Chrome Enterprise + Endpoint Verification
  Threat Protection: Chrome Enterprise Premium
  On-Prem Connector: BeyondCorp Enterprise Connector (3 instances)

ACCESS LEVEL COVERAGE:
  Access Level: corporate-managed
    Devices enrolled:              2,847 / 3,000 (94.9%)
    Compliant devices:             2,712 / 2,847 (95.3%)
  Access Level: high-trust
    Devices enrolled:              312 / 350 (89.1%)
    Compliant devices:             298 / 312 (95.5%)

APPLICATION MIGRATION:
  GCP HTTPS apps (IAP-protected):  32 / 35 (91.4%)
  On-prem apps (via connector):    12 / 15 (80.0%)
  SaaS apps (via SAML/OIDC):       8 / 8 (100%)
  Total migrated:                  52 / 58 (89.7%)

SECURITY METRICS (last 30 days):
  Total access requests:           1,247,832
  Denied by IAP policy:            3,412 (0.27%)
  Denied by access level:          1,208 (0.10%)
  Re-authentication triggered:     45,219
  Anomalous access patterns:       12 (investigated)
  VPN-related incidents (before):  8/month
  BeyondCorp incidents (after):    1/month

VPN DECOMMISSION STATUS:
  Parallel operation remaining:    14 days
  Users still on VPN:              148 (5%)
  Planned decommission:            2026-03-15
```

===== END FILE =====


===== FILE: skills/implementing-beyondcorp-zero-trust-access-model/LICENSE =====

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


===== FILE: skills/implementing-beyondcorp-zero-trust-access-model/assets/template.md =====
# BeyondCorp Zero Trust Access - Migration Checklist

## Project Information
| Field | Value |
|-------|-------|
| Organization | Acme Corporation |
| Project ID | acme-prod-beyondcorp |
| Lead Engineer | J. Smith, Security Architecture |
| Start Date | 2026-01-15 |
| Target Completion | 2026-04-15 |

## Pre-Migration Checklist

### Identity Provider Configuration
- [x] Google Workspace or Cloud Identity configured as primary IdP
- [x] MFA enforced for all users (FIDO2 security keys for privileged accounts)
- [x] User groups defined and synchronized (engineering, finance, contractors, executives)
- [x] Service accounts inventoried and mapped to applications
- [ ] Break-glass accounts created with documented access procedures

### Google Cloud Infrastructure
- [x] IAP API enabled on all production projects
- [x] Access Context Manager API enabled at organization level
- [x] BeyondCorp Enterprise API enabled
- [x] Cloud Audit Logs enabled for IAP data access
- [x] OAuth consent screen configured
- [ ] IAP OAuth clients created per application tier

### Endpoint Verification
- [x] Endpoint Verification extension deployed to Chrome managed browsers
- [x] Device inventory populated (2,847 devices enrolled)
- [ ] Device compliance baseline established (target: 95%)
- [ ] Non-compliant device remediation plan documented
- [ ] BYOD enrollment policy defined

## Access Level Design

| Access Level | Device Policy | Encryption | Screen Lock | Geo Restriction | Applications |
|-------------|---------------|------------|-------------|-----------------|-------------|
| basic-access | Any enrolled | Not required | Not required | None | Public wiki, cafeteria menu |
| standard-access | Enrolled + managed | Required | Required | US, GB, DE | Email, calendar, chat |
| enhanced-access | Managed + EDR | Required | Required | US, GB | Internal tools, CI/CD |
| high-trust | Managed + EDR + patched | Required | Required | US only | Finance, HR, admin panels |

## Application Migration Tracker

| Application | Hosting | Protocol | Current Access | IAP Status | Access Level | Migration Date |
|-------------|---------|----------|---------------|------------|-------------|---------------|
| Internal Wiki | App Engine | HTTPS | VPN | Enabled | basic-access | 2026-02-01 |
| CI/CD Dashboard | GKE | HTTPS | VPN | Enabled | enhanced-access | 2026-02-08 |
| HR Portal | On-prem | HTTPS | VPN | Connector deployed | high-trust | 2026-02-15 |
| Finance System | Compute Engine | HTTPS | VPN | Enabled | high-trust | 2026-02-22 |
| Git Repository | GKE | SSH+HTTPS | VPN | Enabled | enhanced-access | 2026-02-08 |
| Monitoring | Cloud Run | HTTPS | VPN | Enabled | standard-access | 2026-02-01 |
| Admin Console | On-prem | HTTPS | VPN | Connector pending | high-trust | 2026-03-01 |

## Session Policy Configuration

| Application Tier | Session Duration | Re-auth Method | Re-auth Trigger |
|-----------------|-----------------|----------------|-----------------|
| General (Tier 1) | 8 hours | LOGIN | Session expiry |
| Sensitive (Tier 2) | 4 hours | LOGIN | Session expiry, device change |
| Critical (Tier 3) | 1 hour | SECURE_KEY (FIDO2) | Session expiry, IP change |
| Admin (Tier 4) | 30 minutes | SECURE_KEY (FIDO2) | Any context change |

## Post-Migration Validation

### Functional Testing
- [ ] All migrated applications accessible through IAP without VPN
- [ ] Access denied for users not in authorized groups
- [ ] Access denied for non-compliant devices
- [ ] Re-authentication triggers working per policy
- [ ] Break-glass access procedure tested successfully
- [ ] On-premises connector failover tested

### Security Testing
- [ ] Direct application access blocked (only through IAP)
- [ ] IAP bypass attempts detected and logged
- [ ] Session hijacking mitigations verified
- [ ] Cross-tenant access properly denied
- [ ] Audit logs capturing all access decisions

### Monitoring
- [ ] BigQuery audit pipeline operational
- [ ] Alert policies configured for repeated denials
- [ ] Dashboard showing real-time access metrics
- [ ] Monthly access review process documented

## VPN Decommission Timeline

| Phase | Date | Action | Status |
|-------|------|--------|--------|
| Parallel Start | 2026-03-01 | VPN and BeyondCorp running side by side | Pending |
| VPN Monitoring | 2026-03-01 to 2026-03-15 | Track remaining VPN usage | Pending |
| VPN Block New | 2026-03-15 | Block new VPN connections | Pending |
| VPN Shutdown | 2026-04-01 | Decommission VPN infrastructure | Pending |
| Post-Mortem | 2026-04-15 | Migration lessons learned review | Pending |

## Sign-Off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| CISO | _________________ | __________ | __________ |
| Security Architect | _________________ | __________ | __________ |
| IT Operations Lead | _________________ | __________ | __________ |
| Application Owner | _________________ | __________ | __________ |

===== END FILE =====


===== FILE: skills/implementing-beyondcorp-zero-trust-access-model/references/api-reference.md =====
# API Reference: BeyondCorp Zero Trust Assessment Agent

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| requests | >=2.28 | HTTP client for Google Cloud IAP and Access Context Manager APIs |

## CLI Usage

```bash
python scripts/agent.py \
  --project my-gcp-project \
  --output-dir /reports/ \
  --output beyondcorp_report.json
```

## Functions

### `get_gcloud_token() -> str`
Runs `gcloud auth print-access-token` to obtain Bearer token.

### `list_iap_resources(project_id, token) -> list`
GET IAP tunnel destination groups for the project.

### `get_iap_settings(project_id, resource, token) -> dict`
GET IAP settings for a specific compute service resource.

### `list_access_levels(org_id, policy_name, token) -> list`
GET `/accessPolicies/{name}/accessLevels` from Access Context Manager.

### `audit_iap_bindings(project_id, token) -> list`
POST `getIamPolicy` and filters for IAP-related role bindings.

### `assess_zero_trust_posture(project_id, token) -> dict`
Evaluates IAP coverage, binding security, checks for allUsers exposure.

### `generate_report(project_id, token) -> dict`
Computes zero trust score (0-100) based on findings.

## Google Cloud APIs Used

| API | Endpoint |
|-----|----------|
| IAP | `iap.googleapis.com/v1/projects/{id}/iap_tunnel/...` |
| Access Context Manager | `accesscontextmanager.googleapis.com/v1/accessPolicies/...` |
| Resource Manager | `cloudresourcemanager.googleapis.com/v1/projects/{id}:getIamPolicy` |

## Output Schema

```json
{
  "project": "my-project",
  "posture": {"iap_resources": 5, "findings": []},
  "zero_trust_score": 85
}
```

===== END FILE =====


===== FILE: skills/implementing-beyondcorp-zero-trust-access-model/references/standards.md =====
# BeyondCorp Zero Trust Standards & References

## NIST SP 800-207: Zero Trust Architecture
- **Section 2**: Zero Trust Tenets - defines the core principles BeyondCorp implements
- **Section 3.1**: Policy Engine (PE) and Policy Administrator (PA) - maps to IAP and Access Context Manager
- **Section 3.2**: Trust Algorithm - corresponds to Access Levels evaluation
- **Section 4.1**: Device Agent/Gateway-Based Deployment - matches BeyondCorp connector model
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-207/final

## CISA Zero Trust Maturity Model v2.0 (April 2023)
- **Identity Pillar**: MFA enforcement, continuous validation - maps to IAP re-authentication
- **Device Pillar**: Device health monitoring, compliance enforcement - maps to Endpoint Verification
- **Network Pillar**: Micro-segmentation, encrypted traffic - maps to IAP tunnel encryption
- **Application Pillar**: Application access authorization - maps to per-service IAP policies
- **Data Pillar**: Data access governance, DLP - maps to Chrome Enterprise Premium DLP
- **URL**: https://www.cisa.gov/zero-trust-maturity-model

## Google BeyondCorp Papers
- **BeyondCorp: A New Approach to Enterprise Security** (2014)
  - Describes the original BeyondCorp architecture eliminating the privileged intranet
  - URL: https://research.google/pubs/pub43231/
- **BeyondCorp: Design to Deployment at Google** (2016)
  - Details the migration strategy from VPN to BeyondCorp
  - URL: https://research.google/pubs/pub44860/
- **BeyondCorp: The Access Proxy** (2017)
  - Describes the access proxy component that became IAP
  - URL: https://research.google/pubs/pub45728/
- **Migrating to BeyondCorp** (2018)
  - Covers the phased migration approach and lessons learned
  - URL: https://research.google/pubs/pub46134/

## Google Cloud IAP Documentation
- **IAP Overview**: https://cloud.google.com/iap/docs/concepts-overview
- **IAP for Compute Engine**: https://cloud.google.com/iap/docs/enabling-compute-howto
- **IAP for App Engine**: https://cloud.google.com/iap/docs/app-engine-quickstart
- **Access Context Manager**: https://cloud.google.com/access-context-manager/docs
- **Endpoint Verification**: https://cloud.google.com/endpoint-verification/docs
- **BeyondCorp Enterprise**: https://cloud.google.com/beyondcorp-enterprise/docs

## NIST SP 800-63-3: Digital Identity Guidelines
- **Section 4**: Defines identity assurance levels (IAL1-3) relevant to access level design
- **URL**: https://pages.nist.gov/800-63-3/

## DoD Zero Trust Reference Architecture v2.0
- **Section 3.4**: Identity, Credential, and Access Management pillar
- **Section 3.5**: Device pillar - endpoint compliance requirements
- **URL**: https://dodcio.defense.gov/Portals/0/Documents/Library/ZTRAv2.0.pdf

===== END FILE =====


===== FILE: skills/implementing-beyondcorp-zero-trust-access-model/references/workflows.md =====
# BeyondCorp Zero Trust Implementation Workflow

## Phase 1: Discovery and Planning (Weeks 1-2)

### 1.1 Application Inventory
1. Enumerate all internal applications accessed via VPN or corporate network
2. Classify each application by:
   - Hosting environment: GCP (App Engine, GKE, Compute Engine, Cloud Run), on-premises, SaaS
   - Protocol: HTTPS, TCP, SSH, RDP
   - Authentication method: SAML, OIDC, Kerberos, LDAP, custom
   - Sensitivity: Public, Internal, Confidential, Restricted
3. Document current access patterns: which groups access which applications
4. Identify applications that cannot be proxied (raw UDP, custom protocols)

### 1.2 Device Inventory
1. Enumerate all corporate-managed and BYOD devices
2. Document OS distribution: Windows, macOS, ChromeOS, Linux, iOS, Android
3. Verify device management coverage: Intune, Jamf, Chrome Enterprise
4. Identify gaps in device management enrollment

### 1.3 Access Level Design
1. Define trust tiers based on organizational risk appetite:
   - **Tier 1 (Basic)**: Any authenticated user from any device
   - **Tier 2 (Standard)**: Authenticated user from enrolled device with screen lock
   - **Tier 3 (Enhanced)**: Authenticated user from compliant device with disk encryption
   - **Tier 4 (High)**: Authenticated user from managed device with EDR, specific geography
2. Map applications to required trust tiers
3. Define exception process for access level overrides

## Phase 2: Infrastructure Setup (Weeks 3-4)

### 2.1 Google Cloud Configuration
1. Enable required APIs: IAP, Access Context Manager, BeyondCorp Enterprise, Cloud Audit Logs
2. Configure OAuth consent screen and IAP OAuth clients
3. Set up IAP service accounts with minimal permissions
4. Configure Cloud DNS for IAP-protected applications

### 2.2 Access Context Manager Setup
1. Create access policy at the organization level
2. Define access levels using basic conditions (device policy, IP ranges, regions)
3. Define custom access levels using CEL expressions for complex conditions
4. Test access levels with a pilot group before broad deployment

### 2.3 Endpoint Verification Deployment
1. Deploy Endpoint Verification Chrome extension via Google Admin Console policy
2. Configure extension settings: data collection scope, reporting frequency
3. Allow 1-2 weeks for device inventory population
4. Validate device attribute collection against access level requirements

## Phase 3: Application Migration (Weeks 5-10)

### 3.1 GCP-Hosted HTTPS Applications
1. Ensure applications are behind an HTTPS Load Balancer
2. Enable IAP on each backend service
3. Configure IAM bindings with access level conditions
4. Test access with pilot users before expanding
5. Monitor IAP access logs for false denials

### 3.2 On-Premises Applications
1. Deploy BeyondCorp Enterprise connectors in on-premises DMZ
2. Create app connections mapping external DNS to internal endpoints
3. Configure IAP tunnels for TCP-based applications
4. Validate network connectivity from connector to internal applications
5. Test end-to-end access through IAP connector

### 3.3 SaaS Applications
1. Configure SAML/OIDC federation from Google Workspace to SaaS apps
2. Apply conditional access policies at the IdP level
3. Enable session controls and re-authentication requirements

## Phase 4: Policy Enforcement (Weeks 11-12)

### 4.1 Gradual Enforcement
1. Start with audit-only mode: log but do not block non-compliant access
2. Review audit logs to identify users/devices that would be blocked
3. Communicate requirements and provide remediation guidance
4. Enable enforcement in stages: Tier 2 first, then Tier 3, then Tier 4

### 4.2 Re-authentication Configuration
1. Set session duration per application based on sensitivity:
   - General applications: 8-hour session
   - Sensitive applications: 4-hour session
   - Critical applications: 1-hour session
2. Configure re-authentication method: LOGIN (full re-auth) or SECURE_KEY (FIDO2 touch)

## Phase 5: VPN Decommission (Weeks 13-16)

### 5.1 Parallel Operation
1. Run VPN and BeyondCorp in parallel for 30 days
2. Monitor VPN usage to identify remaining dependencies
3. Migrate stragglers and address edge cases
4. Document break-glass procedures for BeyondCorp failure scenarios

### 5.2 VPN Retirement
1. Disable new VPN connections
2. Notify all users of VPN decommission date
3. Remove VPN client from managed devices
4. Decommission VPN infrastructure
5. Redirect VPN DNS entries to BeyondCorp access portal

## Phase 6: Continuous Monitoring (Ongoing)

### 6.1 Access Analytics
1. Build BigQuery dashboards for access pattern analysis
2. Configure alerting for anomalous access patterns:
   - Access from new geographies
   - Access outside business hours
   - Repeated authentication failures
   - Device compliance changes
3. Perform monthly access reviews of IAP bindings

### 6.2 Policy Optimization
1. Review access level effectiveness quarterly
2. Adjust device posture requirements based on threat landscape
3. Update session duration policies based on incident trends
4. Validate break-glass procedures monthly

===== END FILE =====


===== FILE: skills/implementing-beyondcorp-zero-trust-access-model/scripts/agent.py =====
#!/usr/bin/env python3
"""BeyondCorp zero trust access assessment agent using Google Cloud IAP API via requests."""

import argparse
import json
import logging
import os
import subprocess
import sys
from datetime import datetime
from typing import List

try:
    import requests
except ImportError:
    sys.exit("requests required: pip install requests")

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def get_gcloud_token() -> str:
    """Get access token from gcloud CLI."""
    try:
        result = subprocess.run(
            ["gcloud", "auth", "print-access-token"], capture_output=True, text=True, timeout=10)
        return result.stdout.strip()
    except FileNotFoundError:
        return ""


def list_iap_resources(project_id: str, token: str) -> List[dict]:
    """List IAP-protected resources in a GCP project."""
    url = f"https://iap.googleapis.com/v1/projects/{project_id}/iap_tunnel/locations/-/destGroups"
    resp = requests.get(url, headers={"Authorization": f"Bearer {token}"}, timeout=30)
    if resp.status_code == 200:
        return resp.json().get("destGroups", [])
    return []


def get_iap_settings(project_id: str, resource: str, token: str) -> dict:
    """Get IAP settings for a specific resource."""
    url = f"https://iap.googleapis.com/v1/projects/{project_id}/iap_web/compute/services/{resource}:iapSettings"
    resp = requests.get(url, headers={"Authorization": f"Bearer {token}"}, timeout=30)
    if resp.status_code == 200:
        return resp.json()
    return {"error": resp.status_code}


def list_access_levels(org_id: str, policy_name: str, token: str) -> List[dict]:
    """List Access Context Manager access levels."""
    url = f"https://accesscontextmanager.googleapis.com/v1/accessPolicies/{policy_name}/accessLevels"
    resp = requests.get(url, headers={"Authorization": f"Bearer {token}"}, timeout=30)
    if resp.status_code == 200:
        return resp.json().get("accessLevels", [])
    return []


def audit_iap_bindings(project_id: str, token: str) -> List[dict]:
    """Audit IAM policy bindings for IAP-secured resources."""
    url = f"https://cloudresourcemanager.googleapis.com/v1/projects/{project_id}:getIamPolicy"
    resp = requests.post(url, headers={"Authorization": f"Bearer {token}"},
                         json={}, timeout=30)
    if resp.status_code != 200:
        return []
    bindings = resp.json().get("bindings", [])
    iap_bindings = [b for b in bindings if "iap" in b.get("role", "").lower()]
    return iap_bindings


def assess_zero_trust_posture(project_id: str, token: str) -> dict:
    """Assess BeyondCorp zero trust posture for a project."""
    iap_resources = list_iap_resources(project_id, token)
    iap_bindings = audit_iap_bindings(project_id, token)
    findings = []
    if not iap_resources:
        findings.append({"severity": "HIGH", "finding": "No IAP-protected resources found"})
    if not iap_bindings:
        findings.append({"severity": "HIGH", "finding": "No IAP IAM bindings configured"})
    allUsers = any("allUsers" in str(b.get("members", [])) for b in iap_bindings)
    if allUsers:
        findings.append({"severity": "CRITICAL", "finding": "IAP binding includes allUsers"})
    return {
        "iap_resources": len(iap_resources),
        "iap_bindings": len(iap_bindings),
        "findings": findings,
    }


def generate_report(project_id: str, token: str) -> dict:
    """Generate BeyondCorp zero trust assessment report."""
    report = {
        "analysis_date": datetime.utcnow().isoformat(),
        "project": project_id,
        "posture": assess_zero_trust_posture(project_id, token),
    }
    score = 100
    for f in report["posture"]["findings"]:
        if f["severity"] == "CRITICAL":
            score -= 30
        elif f["severity"] == "HIGH":
            score -= 15
    report["zero_trust_score"] = max(0, score)
    return report


def main():
    parser = argparse.ArgumentParser(description="BeyondCorp Zero Trust Assessment Agent")
    parser.add_argument("--project", required=True, help="GCP project ID")
    parser.add_argument("--token", default="", help="Access token (or uses gcloud)")
    parser.add_argument("--output-dir", default=".")
    parser.add_argument("--output", default="beyondcorp_report.json")
    args = parser.parse_args()

    token = args.token or get_gcloud_token()
    if not token:
        logger.error("No access token. Run: gcloud auth print-access-token")
        sys.exit(1)

    os.makedirs(args.output_dir, exist_ok=True)
    report = generate_report(args.project, token)
    out_path = os.path.join(args.output_dir, args.output)
    with open(out_path, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", out_path)
    print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-beyondcorp-zero-trust-access-model/scripts/process.py =====
#!/usr/bin/env python3
"""
BeyondCorp Zero Trust Access Model - Assessment and Audit Tool

Audits GCP IAP configuration, Access Context Manager access levels,
and Endpoint Verification device compliance to validate BeyondCorp
deployment readiness and ongoing compliance.

Requirements:
    pip install google-cloud-iap google-cloud-access-context-manager
    pip install google-cloud-logging google-auth pandas
"""

import json
import subprocess
import sys
from datetime import datetime, timedelta, timezone
from typing import Any


def run_gcloud(args: list[str]) -> dict | list | str:
    """Execute a gcloud command and return parsed JSON output."""
    cmd = ["gcloud"] + args + ["--format=json"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
    if result.returncode != 0:
        print(f"[ERROR] gcloud {' '.join(args[:3])}...: {result.stderr.strip()}")
        return {}
    try:
        return json.loads(result.stdout) if result.stdout.strip() else {}
    except json.JSONDecodeError:
        return result.stdout.strip()


def audit_iap_enabled_services(project_id: str) -> list[dict[str, Any]]:
    """List all backend services with IAP enabled and their policy bindings."""
    print("\n[1/6] Auditing IAP-enabled backend services...")

    backends = run_gcloud([
        "compute", "backend-services", "list",
        "--project", project_id
    ])
    if not isinstance(backends, list):
        print("  No backend services found or unable to list.")
        return []

    iap_services = []
    for backend in backends:
        name = backend.get("name", "unknown")
        iap_config = backend.get("iap", {})
        iap_enabled = iap_config.get("enabled", False)

        if iap_enabled:
            # Get IAM policy for this backend service
            policy = run_gcloud([
                "iap", "web", "get-iam-policy",
                "--resource-type=backend-services",
                "--service", name,
                "--project", project_id
            ])
            bindings = policy.get("bindings", []) if isinstance(policy, dict) else []

            iap_services.append({
                "service_name": name,
                "iap_enabled": True,
                "oauth2_client_id": iap_config.get("oauth2ClientId", "N/A"),
                "bindings_count": len(bindings),
                "bindings": bindings,
                "has_access_level_conditions": any(
                    b.get("condition") for b in bindings
                )
            })
            status = "WITH" if iap_services[-1]["has_access_level_conditions"] else "WITHOUT"
            print(f"  [IAP ON]  {name} - {len(bindings)} bindings {status} access level conditions")
        else:
            print(f"  [IAP OFF] {name}")

    return iap_services


def audit_access_levels(policy_id: str) -> list[dict[str, Any]]:
    """Enumerate and validate Access Context Manager access levels."""
    print("\n[2/6] Auditing Access Context Manager access levels...")

    levels = run_gcloud([
        "access-context-manager", "levels", "list",
        "--policy", policy_id
    ])
    if not isinstance(levels, list):
        print("  No access levels found or unable to list.")
        return []

    access_levels = []
    for level in levels:
        name = level.get("name", "").split("/")[-1]
        title = level.get("title", "Untitled")
        basic = level.get("basic", {})
        custom = level.get("custom", {})

        level_info = {
            "name": name,
            "title": title,
            "type": "basic" if basic else "custom",
            "combining_function": basic.get("combiningFunction", "AND"),
            "conditions_count": len(basic.get("conditions", [])),
            "has_device_policy": False,
            "has_ip_restriction": False,
            "has_region_restriction": False,
            "requires_encryption": False,
            "requires_screenlock": False,
        }

        for condition in basic.get("conditions", []):
            device_policy = condition.get("devicePolicy", {})
            if device_policy:
                level_info["has_device_policy"] = True
                enc_statuses = device_policy.get("allowedEncryptionStatuses", [])
                if "ENCRYPTED" in enc_statuses:
                    level_info["requires_encryption"] = True
                if device_policy.get("requireScreenlock"):
                    level_info["requires_screenlock"] = True
            if condition.get("ipSubnetworks"):
                level_info["has_ip_restriction"] = True
            if condition.get("regions"):
                level_info["has_region_restriction"] = True
                level_info["allowed_regions"] = condition["regions"]

        access_levels.append(level_info)
        print(f"  [{level_info['type'].upper()}] {title} ({name})")
        print(f"    Device policy: {level_info['has_device_policy']}, "
              f"Encryption: {level_info['requires_encryption']}, "
              f"Screen lock: {level_info['requires_screenlock']}")

    return access_levels


def audit_endpoint_verification(project_id: str) -> dict[str, Any]:
    """Check Endpoint Verification device enrollment and compliance."""
    print("\n[3/6] Auditing Endpoint Verification device compliance...")

    devices = run_gcloud([
        "alpha", "devices", "list",
        "--format=json"
    ])
    if not isinstance(devices, list):
        print("  Unable to retrieve device inventory. Ensure Endpoint Verification is deployed.")
        return {"total": 0, "compliant": 0, "non_compliant": 0}

    stats = {
        "total": len(devices),
        "compliant": 0,
        "non_compliant": 0,
        "os_distribution": {},
        "encryption_status": {"encrypted": 0, "unencrypted": 0, "unknown": 0},
        "non_compliant_devices": []
    }

    for device in devices:
        os_type = device.get("osType", "UNKNOWN")
        stats["os_distribution"][os_type] = stats["os_distribution"].get(os_type, 0) + 1

        compliance = device.get("complianceState", "UNKNOWN")
        if compliance == "COMPLIANT":
            stats["compliant"] += 1
        else:
            stats["non_compliant"] += 1
            stats["non_compliant_devices"].append({
                "device_id": device.get("name", "unknown"),
                "os": os_type,
                "compliance_state": compliance
            })

    compliance_rate = (stats["compliant"] / stats["total"] * 100) if stats["total"] > 0 else 0
    print(f"  Total devices: {stats['total']}")
    print(f"  Compliant: {stats['compliant']} ({compliance_rate:.1f}%)")
    print(f"  Non-compliant: {stats['non_compliant']}")
    print(f"  OS distribution: {stats['os_distribution']}")

    return stats


def audit_iap_access_logs(project_id: str, hours: int = 24) -> dict[str, Any]:
    """Analyze IAP access logs for denials and anomalies."""
    print(f"\n[4/6] Analyzing IAP access logs (last {hours} hours)...")

    start_time = (datetime.now(timezone.utc) - timedelta(hours=hours)).isoformat()

    log_filter = (
        f'resource.type="iap_tunnel" OR resource.type="gce_backend_service" '
        f'AND timestamp >= "{start_time}"'
    )

    logs = run_gcloud([
        "logging", "read", log_filter,
        "--project", project_id,
        "--limit=1000"
    ])
    if not isinstance(logs, list):
        print("  No IAP access logs found or unable to query.")
        return {"total": 0, "allowed": 0, "denied": 0}

    stats = {
        "total": len(logs),
        "allowed": 0,
        "denied": 0,
        "denied_users": {},
        "denied_applications": {},
        "top_accessed_apps": {}
    }

    for entry in logs:
        payload = entry.get("jsonPayload", {})
        decision = payload.get("decision", "ALLOW")
        user = payload.get("authenticationInfo", {}).get("principalEmail", "unknown")
        resource = entry.get("resource", {}).get("labels", {}).get("backend_service_name", "unknown")

        stats["top_accessed_apps"][resource] = stats["top_accessed_apps"].get(resource, 0) + 1

        if decision == "DENY":
            stats["denied"] += 1
            stats["denied_users"][user] = stats["denied_users"].get(user, 0) + 1
            stats["denied_applications"][resource] = stats["denied_applications"].get(resource, 0) + 1
        else:
            stats["allowed"] += 1

    denial_rate = (stats["denied"] / stats["total"] * 100) if stats["total"] > 0 else 0
    print(f"  Total requests: {stats['total']}")
    print(f"  Allowed: {stats['allowed']}")
    print(f"  Denied: {stats['denied']} ({denial_rate:.1f}%)")
    if stats["denied_users"]:
        top_denied = sorted(stats["denied_users"].items(), key=lambda x: x[1], reverse=True)[:5]
        print(f"  Top denied users: {top_denied}")

    return stats


def audit_session_policies(project_id: str) -> list[dict[str, Any]]:
    """Check IAP session duration and re-authentication settings."""
    print("\n[5/6] Auditing IAP session and re-authentication policies...")

    backends = run_gcloud([
        "compute", "backend-services", "list",
        "--project", project_id
    ])
    if not isinstance(backends, list):
        return []

    session_policies = []
    for backend in backends:
        iap = backend.get("iap", {})
        if not iap.get("enabled"):
            continue

        name = backend.get("name", "unknown")
        settings = run_gcloud([
            "iap", "settings", "get",
            "--project", project_id,
            "--resource-type=compute",
            "--service", name
        ])

        if isinstance(settings, dict):
            access_settings = settings.get("accessSettings", {})
            reauth = access_settings.get("reauthSettings", {})
            session_policies.append({
                "service": name,
                "max_session_duration": reauth.get("maxAge", "Not configured"),
                "reauth_method": reauth.get("method", "Not configured"),
                "policy_type": reauth.get("policyType", "DEFAULT")
            })
            print(f"  {name}: session={reauth.get('maxAge', 'default')}, "
                  f"method={reauth.get('method', 'default')}")

    return session_policies


def generate_compliance_report(
    project_id: str,
    iap_services: list,
    access_levels: list,
    device_stats: dict,
    log_stats: dict,
    session_policies: list
) -> str:
    """Generate a comprehensive BeyondCorp compliance report."""
    print("\n[6/6] Generating BeyondCorp compliance report...")

    now = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M UTC")
    total_services = len(iap_services)
    with_conditions = sum(1 for s in iap_services if s["has_access_level_conditions"])
    device_compliance = (
        (device_stats["compliant"] / device_stats["total"] * 100)
        if device_stats["total"] > 0 else 0
    )

    report = f"""
BeyondCorp Zero Trust Compliance Audit Report
{'=' * 55}
Project: {project_id}
Generated: {now}
Audit Period: Last 24 hours

1. IAP COVERAGE
   IAP-enabled services:            {total_services}
   Services with access levels:     {with_conditions} / {total_services}
   Coverage rate:                   {(with_conditions / total_services * 100) if total_services else 0:.1f}%

2. ACCESS LEVELS
   Total access levels defined:     {len(access_levels)}
   With device policy:              {sum(1 for a in access_levels if a['has_device_policy'])}
   Requiring encryption:            {sum(1 for a in access_levels if a['requires_encryption'])}
   Requiring screen lock:           {sum(1 for a in access_levels if a['requires_screenlock'])}
   With IP restrictions:            {sum(1 for a in access_levels if a['has_ip_restriction'])}
   With region restrictions:        {sum(1 for a in access_levels if a['has_region_restriction'])}

3. DEVICE COMPLIANCE
   Total enrolled devices:          {device_stats['total']}
   Compliant devices:               {device_stats['compliant']} ({device_compliance:.1f}%)
   Non-compliant devices:           {device_stats['non_compliant']}

4. ACCESS LOG ANALYSIS (24h)
   Total access requests:           {log_stats['total']}
   Allowed:                         {log_stats['allowed']}
   Denied:                          {log_stats['denied']}
   Denial rate:                     {(log_stats['denied'] / log_stats['total'] * 100) if log_stats['total'] else 0:.1f}%

5. SESSION POLICIES
   Services with re-auth policy:    {len(session_policies)}

6. RECOMMENDATIONS
"""
    recommendations = []
    if with_conditions < total_services:
        recommendations.append(
            f"   - {total_services - with_conditions} IAP services lack access level conditions. "
            "Add device posture requirements."
        )
    if device_compliance < 95:
        recommendations.append(
            f"   - Device compliance is {device_compliance:.1f}% (target: 95%). "
            "Remediate non-compliant devices."
        )
    if not any(a["requires_encryption"] for a in access_levels):
        recommendations.append(
            "   - No access levels require disk encryption. Add encryption requirement to sensitive app tiers."
        )
    if len(session_policies) < total_services:
        recommendations.append(
            f"   - {total_services - len(session_policies)} services lack explicit session policies. "
            "Configure re-authentication."
        )
    if not recommendations:
        recommendations.append("   - No critical issues found. Continue monitoring.")

    report += "\n".join(recommendations)

    return report


def main():
    if len(sys.argv) < 2:
        print("Usage: python process.py <project-id> [access-policy-id]")
        print("\nExample:")
        print("  python process.py my-gcp-project 123456789")
        sys.exit(1)

    project_id = sys.argv[1]
    policy_id = sys.argv[2] if len(sys.argv) > 2 else None

    print(f"BeyondCorp Zero Trust Audit - Project: {project_id}")
    print("=" * 55)

    iap_services = audit_iap_enabled_services(project_id)
    access_levels = audit_access_levels(policy_id) if policy_id else []
    device_stats = audit_endpoint_verification(project_id)
    log_stats = audit_iap_access_logs(project_id)
    session_policies = audit_session_policies(project_id)

    report = generate_compliance_report(
        project_id, iap_services, access_levels,
        device_stats, log_stats, session_policies
    )
    print(report)

    report_path = f"beyondcorp_audit_{project_id}_{datetime.now().strftime('%Y%m%d')}.txt"
    with open(report_path, "w") as f:
        f.write(report)
    print(f"\nReport saved to: {report_path}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-cisa-zero-trust-maturity-model/SKILL.md =====
---
name: implementing-cisa-zero-trust-maturity-model
description: Assess, gap-analyze, and progressively implement the CISA Zero Trust
  Maturity Model v2.0 across five pillars (Identity, Devices, Networks, Applications
  & Workloads, Data) and three cross-cutting capabilities (Visibility/Analytics, Automation/Orchestration,
  Governance), from Traditional through Optimal maturity. Use for a federal/enterprise
  ZTMM assessment, phased roadmap, or mapping controls to NIST SP 800-207 and EO 14028.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zero-trust
- cisa
- maturity-model
- federal-compliance
- governance
- nist-800-207
- identity
- devices
- networks
- applications
- data-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- GOVERN-1.1
- GOVERN-1.7
- MAP-1.1
- GOVERN-4.2
- MAP-2.3
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078
- T1190
- T1059
---

# Implementing CISA Zero Trust Maturity Model

## Overview

The CISA Zero Trust Maturity Model (ZTMM) Version 2.0, released in April 2023, provides federal agencies and organizations with a structured roadmap for adopting zero trust architecture. The model defines five core pillars -- Identity, Devices, Networks, Applications & Workloads, and Data -- each progressing through four maturity stages: Traditional, Initial, Advanced, and Optimal. Three cross-cutting capabilities (Visibility and Analytics, Automation and Orchestration, and Governance) span all pillars. This skill covers assessment, gap analysis, and progressive implementation across all pillars and maturity levels.


## When to Use

- When deploying or configuring implementing cisa zero trust maturity model capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with NIST SP 800-207 Zero Trust Architecture
- Understanding of federal cybersecurity mandates (EO 14028, OMB M-22-09)
- Access to organizational IT asset inventory and network architecture documentation
- Knowledge of identity and access management (IAM) fundamentals
- Understanding of network segmentation and microsegmentation concepts

## CISA ZTMM Five Pillars

### Pillar 1: Identity

Identity refers to attributes that uniquely describe an agency user or entity, including non-person entities (NPEs) such as service accounts and machine identities.

**Traditional Stage:**
- Password-based authentication
- Limited identity validation
- Manual provisioning and deprovisioning

**Initial Stage:**
- MFA deployed for privileged users
- Identity governance initiated
- Basic identity lifecycle management

**Advanced Stage:**
- Phishing-resistant MFA for all users (FIDO2/WebAuthn)
- Continuous identity validation
- Automated provisioning tied to HR systems
- Identity threat detection and response (ITDR)

**Optimal Stage:**
- Continuous, real-time identity verification
- Passwordless authentication across all systems
- AI-driven anomaly detection for identity behaviors
- Full integration of identity signals into access decisions

### Pillar 2: Devices

Devices include any hardware, software, or firmware asset that connects to a network -- servers, laptops, mobile phones, IoT devices, and network equipment.

**Traditional Stage:**
- Limited device inventory
- Basic endpoint protection (antivirus)
- No device compliance checks

**Initial Stage:**
- Comprehensive device inventory
- Endpoint Detection and Response (EDR) deployment
- Basic device health checks before network access

**Advanced Stage:**
- Real-time device posture assessment
- Automated compliance enforcement
- Device certificates for machine identity
- Vulnerability scanning integrated into access decisions

**Optimal Stage:**
- Continuous device trust scoring
- Automated remediation of non-compliant devices
- Full device lifecycle management integrated with zero trust policies
- Firmware integrity verification

### Pillar 3: Networks

Networks encompass all communications media including internal networks, wireless, and the internet.

**Traditional Stage:**
- Perimeter-based security (firewalls, VPNs)
- Flat internal networks
- Minimal east-west traffic inspection

**Initial Stage:**
- Initial network segmentation
- Encrypted DNS and internal traffic
- Basic network monitoring and logging

**Advanced Stage:**
- Microsegmentation of critical assets
- Software-defined networking (SDN) for dynamic policy enforcement
- Full TLS encryption for all internal communications
- Network Detection and Response (NDR)

**Optimal Stage:**
- Fully software-defined, policy-driven network
- Zero implicit trust zones
- AI-driven network anomaly detection
- Automated threat response integrated with network controls

### Pillar 4: Applications and Workloads

Applications and workloads include agency systems, programs, and services running on-premises, on mobile devices, and in cloud environments.

**Traditional Stage:**
- Perimeter-protected applications
- Manual vulnerability patching
- Limited application-level logging

**Initial Stage:**
- Application-level access controls
- Web Application Firewalls (WAF)
- Regular vulnerability scanning
- Application inventory established

**Advanced Stage:**
- Continuous integration of security testing (SAST/DAST)
- Application-aware microsegmentation
- API security gateways
- Immutable infrastructure patterns

**Optimal Stage:**
- Runtime application self-protection (RASP)
- Automated application security orchestration
- Full DevSecOps pipeline integration
- Zero-standing privileges for application access

### Pillar 5: Data

Data encompasses all structured and unstructured information, at rest, in transit, and in use.

**Traditional Stage:**
- Basic encryption for data at rest
- Limited data classification
- No data loss prevention

**Initial Stage:**
- Data classification scheme implemented
- DLP policies for sensitive data
- Encryption for data in transit (TLS 1.2+)
- Basic data inventory

**Advanced Stage:**
- Automated data classification
- Fine-grained data access controls
- Data activity monitoring
- Rights management for sensitive documents

**Optimal Stage:**
- Real-time data flow analytics
- AI-driven data classification and protection
- Automated response to data exfiltration attempts
- Full data lifecycle governance with zero trust principles

## Cross-Cutting Capabilities

### Visibility and Analytics

```
Maturity Progression:
Traditional -> Manual log review, limited SIEM
Initial     -> Centralized logging, basic SIEM correlation
Advanced    -> UEBA, automated threat detection, data lake analytics
Optimal     -> AI/ML-driven continuous monitoring, predictive analytics
```

### Automation and Orchestration

```
Maturity Progression:
Traditional -> Manual incident response, ad-hoc scripts
Initial     -> Basic SOAR playbooks, automated alerting
Advanced    -> Integrated SOAR with multi-pillar orchestration
Optimal     -> Fully autonomous response, self-healing infrastructure
```

### Governance

```
Maturity Progression:
Traditional -> Ad-hoc policies, manual compliance checks
Initial     -> Documented zero trust strategy, basic policy framework
Advanced    -> Policy-as-code, continuous compliance monitoring
Optimal     -> Dynamic policy engine, real-time governance decisions
```

## Implementation Process

### Phase 1: Assessment and Baseline

1. **Inventory all assets** across the five pillars
2. **Map current capabilities** to ZTMM maturity stages
3. **Conduct gap analysis** between current and target states
4. **Identify quick wins** that move from Traditional to Initial stage
5. **Document dependencies** between pillars

```python
# Example: CISA ZTMM Maturity Assessment Scoring
class ZTMMAssessment:
    PILLARS = ['Identity', 'Devices', 'Networks', 'Applications', 'Data']
    STAGES = ['Traditional', 'Initial', 'Advanced', 'Optimal']
    CROSS_CUTTING = ['Visibility_Analytics', 'Automation_Orchestration', 'Governance']

    def __init__(self):
        self.scores = {}

    def assess_pillar(self, pillar, capabilities):
        """
        Assess a pillar against ZTMM criteria.
        capabilities: dict of capability_name -> maturity_stage
        """
        stage_values = {stage: i for i, stage in enumerate(self.STAGES)}
        scores = [stage_values.get(stage, 0) for stage in capabilities.values()]
        avg_score = sum(scores) / len(scores) if scores else 0

        overall_stage = self.STAGES[int(avg_score)]
        self.scores[pillar] = {
            'capabilities': capabilities,
            'average_score': avg_score,
            'overall_stage': overall_stage
        }
        return self.scores[pillar]

    def generate_roadmap(self):
        """Generate prioritized improvement roadmap."""
        roadmap = []
        for pillar, data in self.scores.items():
            for capability, stage in data['capabilities'].items():
                stage_idx = self.STAGES.index(stage)
                if stage_idx < 3:  # Not yet Optimal
                    next_stage = self.STAGES[stage_idx + 1]
                    roadmap.append({
                        'pillar': pillar,
                        'capability': capability,
                        'current': stage,
                        'target': next_stage,
                        'priority': 3 - stage_idx  # Higher priority for lower maturity
                    })
        return sorted(roadmap, key=lambda x: x['priority'], reverse=True)
```

### Phase 2: Identity Foundation

1. Deploy phishing-resistant MFA (FIDO2/WebAuthn)
2. Implement identity governance and administration (IGA)
3. Establish continuous identity verification
4. Integrate identity providers with all applications
5. Deploy identity threat detection and response

### Phase 3: Device Trust

1. Complete asset inventory with automated discovery
2. Deploy EDR across all endpoints
3. Implement device compliance checking
4. Establish device certificate infrastructure
5. Create device trust scoring mechanism

### Phase 4: Network Transformation

1. Implement network segmentation strategy
2. Deploy microsegmentation for critical assets
3. Enable encrypted DNS (DoH/DoT)
4. Enforce TLS 1.3 for all internal communications
5. Deploy NDR capabilities

### Phase 5: Application Security

1. Implement application-level access controls
2. Deploy WAF and API security gateways
3. Integrate security testing into CI/CD pipelines
4. Establish application inventory and classification
5. Implement runtime protection

### Phase 6: Data Protection

1. Implement data classification framework
2. Deploy DLP across endpoints and network
3. Enable data activity monitoring
4. Implement rights management
5. Establish data lifecycle governance

## Compliance Mapping

| CISA ZTMM Pillar | OMB M-22-09 Requirement | NIST 800-207 Section |
|---|---|---|
| Identity | MFA for agency staff | 3.1.1 |
| Devices | EDR for federal endpoints | 3.1.2 |
| Networks | Encrypt DNS traffic | 3.1.3 |
| Applications | Application security testing | 3.1.4 |
| Data | Data categorization | 3.1.5 |

## Metrics and KPIs

- **Identity Pillar**: Percentage of users with phishing-resistant MFA
- **Device Pillar**: Percentage of devices with real-time posture assessment
- **Network Pillar**: Percentage of network segments microsegmented
- **Application Pillar**: Percentage of applications with zero trust access controls
- **Data Pillar**: Percentage of sensitive data classified and protected
- **Overall**: ZTMM stage achieved per pillar (target: Advanced minimum)

## References

- [CISA Zero Trust Maturity Model v2.0](https://www.cisa.gov/zero-trust-maturity-model)
- [CISA ZTMM v2.0 PDF](https://www.cisa.gov/sites/default/files/2023-04/zero_trust_maturity_model_v2_508.pdf)
- [NIST SP 800-207 Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final)
- [OMB Memorandum M-22-09](https://www.whitehouse.gov/wp-content/uploads/2022/01/M-22-09.pdf)
- [NSA Zero Trust Pillars Guidance](https://media.defense.gov/2024/Apr/09/2003434442/-1/-1/0/CSI_DATA_PILLAR_ZT.PDF)
- [Microsoft Guidance for CISA ZTMM](https://www.microsoft.com/en-us/security/blog/2024/12/19/new-microsoft-guidance-for-the-cisa-zero-trust-maturity-model/)

===== END FILE =====


===== FILE: skills/implementing-cisa-zero-trust-maturity-model/LICENSE =====

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


===== FILE: skills/implementing-cisa-zero-trust-maturity-model/assets/template.md =====
# CISA ZTMM Assessment Template

## Organization Information
- **Organization Name**: _______________
- **Assessment Date**: _______________
- **Assessment Lead**: _______________
- **Pillar Owners**:
  - Identity: _______________
  - Devices: _______________
  - Networks: _______________
  - Applications: _______________
  - Data: _______________

## Pillar Assessment Worksheet

### Identity Pillar

| Function | Traditional | Initial | Advanced | Optimal | Current | Evidence |
|---|---|---|---|---|---|---|
| Authentication | [ ] | [ ] | [ ] | [ ] | ___ | |
| Identity Stores | [ ] | [ ] | [ ] | [ ] | ___ | |
| Risk Assessment | [ ] | [ ] | [ ] | [ ] | ___ | |
| Access Management | [ ] | [ ] | [ ] | [ ] | ___ | |
| Identity Lifecycle | [ ] | [ ] | [ ] | [ ] | ___ | |
| Visibility & Analytics | [ ] | [ ] | [ ] | [ ] | ___ | |
| Automation & Orchestration | [ ] | [ ] | [ ] | [ ] | ___ | |
| Governance | [ ] | [ ] | [ ] | [ ] | ___ | |

### Devices Pillar

| Function | Traditional | Initial | Advanced | Optimal | Current | Evidence |
|---|---|---|---|---|---|---|
| Policy Enforcement | [ ] | [ ] | [ ] | [ ] | ___ | |
| Asset Management | [ ] | [ ] | [ ] | [ ] | ___ | |
| Device Compliance | [ ] | [ ] | [ ] | [ ] | ___ | |
| Device Threat Protection | [ ] | [ ] | [ ] | [ ] | ___ | |
| Visibility & Analytics | [ ] | [ ] | [ ] | [ ] | ___ | |
| Automation & Orchestration | [ ] | [ ] | [ ] | [ ] | ___ | |
| Governance | [ ] | [ ] | [ ] | [ ] | ___ | |

### Networks Pillar

| Function | Traditional | Initial | Advanced | Optimal | Current | Evidence |
|---|---|---|---|---|---|---|
| Network Segmentation | [ ] | [ ] | [ ] | [ ] | ___ | |
| Threat Protection | [ ] | [ ] | [ ] | [ ] | ___ | |
| Encryption | [ ] | [ ] | [ ] | [ ] | ___ | |
| Network Resilience | [ ] | [ ] | [ ] | [ ] | ___ | |
| Visibility & Analytics | [ ] | [ ] | [ ] | [ ] | ___ | |
| Automation & Orchestration | [ ] | [ ] | [ ] | [ ] | ___ | |
| Governance | [ ] | [ ] | [ ] | [ ] | ___ | |

### Applications & Workloads Pillar

| Function | Traditional | Initial | Advanced | Optimal | Current | Evidence |
|---|---|---|---|---|---|---|
| Access Authorization | [ ] | [ ] | [ ] | [ ] | ___ | |
| Threat Protection | [ ] | [ ] | [ ] | [ ] | ___ | |
| Accessibility | [ ] | [ ] | [ ] | [ ] | ___ | |
| Application Security | [ ] | [ ] | [ ] | [ ] | ___ | |
| Visibility & Analytics | [ ] | [ ] | [ ] | [ ] | ___ | |
| Automation & Orchestration | [ ] | [ ] | [ ] | [ ] | ___ | |
| Governance | [ ] | [ ] | [ ] | [ ] | ___ | |

### Data Pillar

| Function | Traditional | Initial | Advanced | Optimal | Current | Evidence |
|---|---|---|---|---|---|---|
| Data Inventory | [ ] | [ ] | [ ] | [ ] | ___ | |
| Data Categorization | [ ] | [ ] | [ ] | [ ] | ___ | |
| Data Availability | [ ] | [ ] | [ ] | [ ] | ___ | |
| Data Access | [ ] | [ ] | [ ] | [ ] | ___ | |
| Data Encryption | [ ] | [ ] | [ ] | [ ] | ___ | |
| Visibility & Analytics | [ ] | [ ] | [ ] | [ ] | ___ | |
| Automation & Orchestration | [ ] | [ ] | [ ] | [ ] | ___ | |
| Governance | [ ] | [ ] | [ ] | [ ] | ___ | |

## Gap Analysis Summary

| Pillar | Current Stage | Target Stage | Gap | Priority |
|---|---|---|---|---|
| Identity | ___ | Advanced | ___ | ___ |
| Devices | ___ | Advanced | ___ | ___ |
| Networks | ___ | Advanced | ___ | ___ |
| Applications | ___ | Advanced | ___ | ___ |
| Data | ___ | Advanced | ___ | ___ |

## OMB M-22-09 Compliance Checklist

- [ ] Phishing-resistant MFA deployed for all agency staff
- [ ] Complete device inventory with EDR coverage
- [ ] DNS and HTTP traffic encrypted
- [ ] Applications treated as internet-connected with regular testing
- [ ] Data categorization and automated discovery implemented

## Roadmap Priorities

### Quick Wins (0-3 months)
1. _______________
2. _______________
3. _______________

### Short-term (3-6 months)
1. _______________
2. _______________
3. _______________

### Medium-term (6-12 months)
1. _______________
2. _______________
3. _______________

### Long-term (12-24 months)
1. _______________
2. _______________
3. _______________

===== END FILE =====


===== FILE: skills/implementing-cisa-zero-trust-maturity-model/references/api-reference.md =====
# API Reference: CISA Zero Trust Maturity Model Assessment Agent

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| (stdlib only) | Python 3.8+ | JSON processing, assessment logic |

## CLI Usage

```bash
python scripts/agent.py \
  --data /assessments/zt_responses.json \
  --output-dir /reports/ \
  --output ztmm_report.json
```

## Functions

### `assess_control(control, implemented, maturity) -> dict`
Scores a single control: 0 (Traditional) to 3 (Optimal).

### `assess_pillar(pillar, responses) -> dict`
Evaluates all controls within a CISA ZT pillar. Returns score, percentage, and maturity level.

### `compute_overall_maturity(pillar_results) -> dict`
Aggregates pillar scores into overall maturity: Traditional/Initial/Advanced/Optimal.

### `generate_recommendations(pillar_results) -> list`
Identifies unimplemented controls, prioritizes by pillar weakness.

### `generate_report(data_path) -> dict`
Full assessment pipeline: load data, assess 5 pillars, compute maturity, generate recommendations.

## CISA ZT Pillars

| Pillar | Controls Assessed |
|--------|-------------------|
| Identity | MFA, phishing-resistant MFA, JIT access, PAM |
| Devices | Inventory, EDR, health attestation, posture |
| Networks | Microsegmentation, encrypted DNS, SDP |
| Applications | Inventory, access controls, API security |
| Data | Classification, DLP, encryption at rest |

## Input Data Format

```json
{
  "Identity": {
    "MFA enforced for all users": {"implemented": true, "maturity": "Advanced"},
    "Phishing-resistant MFA (FIDO2/PIV)": {"implemented": false, "maturity": "Traditional"}
  }
}
```

## Output Schema

```json
{
  "overall_maturity": {"percentage": 52.3, "maturity_level": "Advanced"},
  "pillars": [{"pillar": "Identity", "percentage": 66.7, "maturity_level": "Advanced"}],
  "recommendations": [{"pillar": "Devices", "control": "EDR deployed", "priority": "HIGH"}]
}
```

===== END FILE =====


===== FILE: skills/implementing-cisa-zero-trust-maturity-model/references/standards.md =====
# Standards Reference: CISA Zero Trust Maturity Model

## Primary Standards

### CISA Zero Trust Maturity Model v2.0 (April 2023)
- **Source**: Cybersecurity and Infrastructure Security Agency
- **Scope**: Federal agencies and organizations implementing zero trust
- **Five Pillars**: Identity, Devices, Networks, Applications & Workloads, Data
- **Four Maturity Stages**: Traditional, Initial, Advanced, Optimal
- **Cross-Cutting**: Visibility & Analytics, Automation & Orchestration, Governance

### NIST SP 800-207: Zero Trust Architecture
- **Published**: August 2020
- **Tenets**: Never trust, always verify; assume breach; least privilege access
- **Deployment Models**: Device agent/gateway, enclave, resource portal
- **Key Requirement**: Policy decision point (PDP) and policy enforcement point (PEP)

### Executive Order 14028: Improving the Nation's Cybersecurity
- **Signed**: May 12, 2021
- **Mandate**: Federal agencies must adopt zero trust architecture
- **Timeline**: Agencies required to develop zero trust implementation plans

### OMB Memorandum M-22-09: Federal Zero Trust Strategy
- **Published**: January 2022
- **Requirements per pillar**:
  - Identity: Phishing-resistant MFA for all staff
  - Devices: EDR deployed across federal endpoints
  - Networks: DNS traffic encrypted, HTTP traffic encrypted
  - Applications: Application security testing in CI/CD
  - Data: Data categorization and automated classification

## Supporting Standards

### NSA Zero Trust Pillar Guidance Series (2024)
- User Pillar (February 2024)
- Device Pillar (March 2024)
- Data Pillar (April 2024)
- Application & Workload Pillar (April 2024)
- Network & Environment Pillar (May 2024)
- Visibility & Analytics Pillar (May 2024)
- Automation & Orchestration Pillar (June 2024)

### DISA Zero Trust Reference Architecture
- Department of Defense specific implementation
- Aligns with NIST 800-207 and CISA ZTMM
- Covers DoD-specific compliance requirements

### FedRAMP Zero Trust Requirements
- Cloud service providers must support zero trust
- Continuous monitoring requirements
- Identity federation standards

===== END FILE =====


===== FILE: skills/implementing-cisa-zero-trust-maturity-model/references/workflows.md =====
# Workflows: CISA Zero Trust Maturity Model Implementation

## Workflow 1: Initial Maturity Assessment

```
Step 1: Establish Assessment Team
  - Identify stakeholders from IT, security, compliance, and business units
  - Assign pillar owners for each of the five ZTMM pillars
  - Define assessment timeline and reporting cadence

Step 2: Inventory Current Capabilities
  - Identity: Catalog authentication methods, identity providers, MFA coverage
  - Devices: Enumerate all endpoints, document endpoint security tools
  - Networks: Map network architecture, segmentation, encryption status
  - Applications: List all applications, classify access controls
  - Data: Identify data repositories, classification, DLP status

Step 3: Map to ZTMM Stages
  - For each pillar, evaluate each function against the four maturity stages
  - Document evidence for current stage determination
  - Identify gaps between current and target maturity
  - Rate cross-cutting capabilities (visibility, automation, governance)

Step 4: Produce Assessment Report
  - Pillar-by-pillar maturity scores
  - Gap analysis with prioritized recommendations
  - Quick wins vs. long-term transformation items
  - Resource requirements and estimated timelines
```

## Workflow 2: Identity Pillar Advancement (Traditional to Advanced)

```
Phase A: MFA Deployment
  1. Inventory all user accounts (privileged, standard, service)
  2. Select phishing-resistant MFA solution (FIDO2/WebAuthn)
  3. Deploy MFA for privileged accounts first
  4. Extend MFA to all user accounts
  5. Implement MFA for service accounts and APIs
  6. Configure conditional access policies

Phase B: Identity Governance
  1. Implement identity lifecycle management
  2. Connect IAM to HR system for automated provisioning
  3. Establish access certification reviews
  4. Deploy identity threat detection
  5. Implement just-in-time access for elevated privileges

Phase C: Continuous Verification
  1. Integrate identity signals into access decisions
  2. Deploy risk-based authentication
  3. Implement session-level re-authentication for sensitive actions
  4. Enable behavioral analytics for identity anomalies
```

## Workflow 3: Cross-Pillar Integration

```
Step 1: Establish Unified Policy Engine
  - Define access policies that incorporate all five pillars
  - Implement Policy Decision Point (PDP) per NIST 800-207
  - Deploy Policy Enforcement Points (PEP) at all access boundaries

Step 2: Integrate Signal Sources
  - Identity signals -> trust score component
  - Device posture -> trust score component
  - Network context -> trust score component
  - Application risk -> trust score component
  - Data sensitivity -> access control component

Step 3: Implement Continuous Evaluation
  - Real-time trust scoring engine
  - Dynamic policy adjustment based on risk
  - Automated access revocation on policy violation
  - Audit logging for all access decisions

Step 4: Measure and Report
  - Track maturity progression per pillar quarterly
  - Report to leadership with ZTMM scorecard
  - Adjust roadmap based on threat landscape changes
  - Document lessons learned for continuous improvement
```

## Workflow 4: Governance and Compliance Reporting

```
Step 1: Establish Zero Trust Governance Board
  - Executive sponsor, CISO, pillar owners, compliance
  - Monthly review of zero trust maturity progress
  - Annual strategic review and roadmap adjustment

Step 2: Continuous Compliance Monitoring
  - Map ZTMM controls to OMB M-22-09 requirements
  - Automate evidence collection for each pillar
  - Generate compliance dashboards
  - Prepare for FISMA and other audit requirements

Step 3: Reporting to CISA
  - Submit agency zero trust implementation plan
  - Provide quarterly progress updates
  - Document deviations and remediation plans
```

===== END FILE =====


===== FILE: skills/implementing-cisa-zero-trust-maturity-model/scripts/agent.py =====
#!/usr/bin/env python3
"""CISA Zero Trust Maturity Model assessment agent for organizational ZT posture evaluation."""

import argparse
import json
import logging
import os
from datetime import datetime
from typing import Dict, List

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

PILLARS = ["Identity", "Devices", "Networks", "Applications", "Data"]
MATURITY_LEVELS = ["Traditional", "Initial", "Advanced", "Optimal"]
CROSS_CUTTING = ["Visibility & Analytics", "Automation & Orchestration", "Governance"]

PILLAR_CONTROLS = {
    "Identity": [
        "MFA enforced for all users",
        "Phishing-resistant MFA (FIDO2/PIV)",
        "Continuous identity validation",
        "Identity lifecycle management",
        "Privileged access management",
        "Just-in-time access provisioning",
    ],
    "Devices": [
        "Device inventory and compliance",
        "EDR deployed on all endpoints",
        "Device health attestation",
        "Real-time posture assessment",
        "Automated remediation for non-compliant devices",
    ],
    "Networks": [
        "Microsegmentation implemented",
        "Encrypted DNS (DoH/DoT)",
        "Network traffic encrypted in transit",
        "Software-defined perimeter",
        "Network access based on identity",
    ],
    "Applications": [
        "Application inventory maintained",
        "Application-level access controls",
        "Continuous application security testing",
        "Secure API gateway",
        "Application isolation and sandboxing",
    ],
    "Data": [
        "Data classification implemented",
        "Data loss prevention controls",
        "Encryption at rest for sensitive data",
        "Data access logging and monitoring",
        "Automated data lifecycle management",
    ],
}


def assess_control(control: str, implemented: bool, maturity: str) -> dict:
    """Assess a single control's implementation status and maturity."""
    level_scores = {"Traditional": 0, "Initial": 1, "Advanced": 2, "Optimal": 3}
    return {
        "control": control,
        "implemented": implemented,
        "maturity_level": maturity,
        "score": level_scores.get(maturity, 0) if implemented else 0,
    }


def assess_pillar(pillar: str, responses: Dict[str, dict]) -> dict:
    """Assess a single CISA ZT pillar based on control responses."""
    controls = PILLAR_CONTROLS.get(pillar, [])
    assessed = []
    for control in controls:
        resp = responses.get(control, {"implemented": False, "maturity": "Traditional"})
        assessed.append(assess_control(control, resp["implemented"], resp["maturity"]))
    max_score = len(controls) * 3
    actual_score = sum(c["score"] for c in assessed)
    pct = (actual_score / max_score * 100) if max_score else 0
    implemented_count = sum(1 for c in assessed if c["implemented"])
    if pct >= 75:
        level = "Optimal"
    elif pct >= 50:
        level = "Advanced"
    elif pct >= 25:
        level = "Initial"
    else:
        level = "Traditional"
    return {
        "pillar": pillar,
        "controls_assessed": len(assessed),
        "controls_implemented": implemented_count,
        "score": actual_score,
        "max_score": max_score,
        "percentage": round(pct, 1),
        "maturity_level": level,
        "controls": assessed,
    }


def load_assessment_data(data_path: str) -> dict:
    """Load assessment responses from JSON file."""
    with open(data_path, "r") as f:
        return json.load(f)


def compute_overall_maturity(pillar_results: List[dict]) -> dict:
    """Compute overall zero trust maturity from pillar assessments."""
    total_score = sum(p["score"] for p in pillar_results)
    total_max = sum(p["max_score"] for p in pillar_results)
    pct = (total_score / total_max * 100) if total_max else 0
    if pct >= 75:
        level = "Optimal"
    elif pct >= 50:
        level = "Advanced"
    elif pct >= 25:
        level = "Initial"
    else:
        level = "Traditional"
    return {"overall_score": total_score, "max_score": total_max,
            "percentage": round(pct, 1), "maturity_level": level}


def generate_recommendations(pillar_results: List[dict]) -> List[dict]:
    """Generate prioritized recommendations based on assessment gaps."""
    recs = []
    for pillar in pillar_results:
        for control in pillar["controls"]:
            if not control["implemented"]:
                recs.append({
                    "pillar": pillar["pillar"],
                    "control": control["control"],
                    "priority": "HIGH" if pillar["percentage"] < 50 else "MEDIUM",
                    "action": f"Implement: {control['control']}",
                })
    recs.sort(key=lambda r: 0 if r["priority"] == "HIGH" else 1)
    return recs


def generate_report(data_path: str) -> dict:
    """Generate CISA Zero Trust Maturity assessment report."""
    data = load_assessment_data(data_path)
    pillar_results = []
    for pillar in PILLARS:
        responses = data.get(pillar, {})
        pillar_results.append(assess_pillar(pillar, responses))
    overall = compute_overall_maturity(pillar_results)
    recs = generate_recommendations(pillar_results)
    return {
        "analysis_date": datetime.utcnow().isoformat(),
        "framework": "CISA Zero Trust Maturity Model v2.0",
        "overall_maturity": overall,
        "pillars": pillar_results,
        "recommendations": recs[:20],
        "recommendation_count": len(recs),
    }


def main():
    parser = argparse.ArgumentParser(description="CISA Zero Trust Maturity Model Assessment")
    parser.add_argument("--data", required=True, help="Path to assessment data JSON")
    parser.add_argument("--output-dir", default=".")
    parser.add_argument("--output", default="ztmm_report.json")
    args = parser.parse_args()

    os.makedirs(args.output_dir, exist_ok=True)
    report = generate_report(args.data)
    out_path = os.path.join(args.output_dir, args.output)
    with open(out_path, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", out_path)
    print(json.dumps(report["overall_maturity"], indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-cisa-zero-trust-maturity-model/scripts/process.py =====
#!/usr/bin/env python3
"""
CISA Zero Trust Maturity Model Assessment and Roadmap Generator.

Evaluates organizational zero trust maturity across the five CISA ZTMM pillars
(Identity, Devices, Networks, Applications, Data) and three cross-cutting
capabilities (Visibility & Analytics, Automation & Orchestration, Governance).
Generates gap analysis, prioritized roadmap, and compliance mapping.
"""

import json
import csv
import datetime
from dataclasses import dataclass, field
from enum import IntEnum
from typing import Optional
from pathlib import Path


class MaturityStage(IntEnum):
    TRADITIONAL = 0
    INITIAL = 1
    ADVANCED = 2
    OPTIMAL = 3


PILLAR_FUNCTIONS = {
    "Identity": [
        "authentication",
        "identity_stores",
        "risk_assessment",
        "access_management",
        "identity_lifecycle",
        "visibility_analytics",
        "automation_orchestration",
        "governance",
    ],
    "Devices": [
        "policy_enforcement",
        "asset_management",
        "device_compliance",
        "device_threat_protection",
        "visibility_analytics",
        "automation_orchestration",
        "governance",
    ],
    "Networks": [
        "network_segmentation",
        "threat_protection",
        "encryption",
        "network_resilience",
        "visibility_analytics",
        "automation_orchestration",
        "governance",
    ],
    "Applications": [
        "access_authorization",
        "threat_protection",
        "accessibility",
        "application_security",
        "visibility_analytics",
        "automation_orchestration",
        "governance",
    ],
    "Data": [
        "data_inventory",
        "data_categorization",
        "data_availability",
        "data_access",
        "data_encryption",
        "visibility_analytics",
        "automation_orchestration",
        "governance",
    ],
}

OMB_M2209_REQUIREMENTS = {
    "Identity": {
        "requirement": "Agency staff use enterprise-managed identities with phishing-resistant MFA",
        "target_stage": MaturityStage.ADVANCED,
        "key_actions": [
            "Deploy FIDO2/WebAuthn for all users",
            "Integrate identity provider with all applications",
            "Implement authorization based on user attributes",
        ],
    },
    "Devices": {
        "requirement": "Federal government has a complete inventory of authorized devices and can prevent/detect/respond to incidents",
        "target_stage": MaturityStage.ADVANCED,
        "key_actions": [
            "Deploy EDR on all endpoints",
            "Maintain real-time asset inventory",
            "Enforce device compliance before access",
        ],
    },
    "Networks": {
        "requirement": "Agencies encrypt all DNS and HTTP traffic within their environment",
        "target_stage": MaturityStage.ADVANCED,
        "key_actions": [
            "Encrypt all DNS traffic (DoH/DoT)",
            "Enforce HTTPS for all web traffic",
            "Implement network microsegmentation",
        ],
    },
    "Applications": {
        "requirement": "Agencies treat all applications as internet-connected and routinely test them",
        "target_stage": MaturityStage.ADVANCED,
        "key_actions": [
            "Integrate SAST/DAST into CI/CD",
            "Remove application access from VPN dependencies",
            "Implement application-level access policies",
        ],
    },
    "Data": {
        "requirement": "Agencies have thorough data categorization and employ automated tools",
        "target_stage": MaturityStage.ADVANCED,
        "key_actions": [
            "Implement data classification scheme",
            "Deploy automated data tagging",
            "Enable DLP across all data channels",
        ],
    },
}


@dataclass
class FunctionAssessment:
    function_name: str
    current_stage: MaturityStage
    target_stage: MaturityStage
    evidence: str = ""
    gaps: list = field(default_factory=list)
    recommendations: list = field(default_factory=list)


@dataclass
class PillarAssessment:
    pillar_name: str
    functions: list = field(default_factory=list)
    overall_stage: MaturityStage = MaturityStage.TRADITIONAL
    compliance_gap: bool = False

    def calculate_overall(self):
        if not self.functions:
            return
        scores = [f.current_stage.value for f in self.functions]
        avg = sum(scores) / len(scores)
        self.overall_stage = MaturityStage(int(avg))

    def check_compliance(self):
        req = OMB_M2209_REQUIREMENTS.get(self.pillar_name)
        if req:
            self.compliance_gap = self.overall_stage < req["target_stage"]


@dataclass
class RoadmapItem:
    pillar: str
    function_name: str
    current_stage: str
    target_stage: str
    priority: int  # 1=highest, 4=lowest
    effort: str  # low, medium, high
    recommendations: list = field(default_factory=list)
    dependencies: list = field(default_factory=list)


class ZTMMAssessment:
    """Full CISA Zero Trust Maturity Model assessment engine."""

    def __init__(self, organization_name: str):
        self.organization = organization_name
        self.assessment_date = datetime.datetime.now().isoformat()
        self.pillar_assessments: dict[str, PillarAssessment] = {}
        self.roadmap: list[RoadmapItem] = []

    def assess_function(
        self,
        pillar: str,
        function_name: str,
        current_stage: int,
        target_stage: int = 3,
        evidence: str = "",
        gaps: Optional[list] = None,
    ) -> FunctionAssessment:
        current = MaturityStage(current_stage)
        target = MaturityStage(target_stage)
        fa = FunctionAssessment(
            function_name=function_name,
            current_stage=current,
            target_stage=target,
            evidence=evidence,
            gaps=gaps or [],
        )
        if pillar not in self.pillar_assessments:
            self.pillar_assessments[pillar] = PillarAssessment(pillar_name=pillar)
        self.pillar_assessments[pillar].functions.append(fa)
        return fa

    def calculate_maturity(self):
        for pa in self.pillar_assessments.values():
            pa.calculate_overall()
            pa.check_compliance()

    def generate_roadmap(self) -> list[RoadmapItem]:
        self.roadmap = []
        effort_map = {0: "high", 1: "medium", 2: "low", 3: "low"}

        for pillar_name, pa in self.pillar_assessments.items():
            for func in pa.functions:
                if func.current_stage < func.target_stage:
                    gap = func.target_stage.value - func.current_stage.value
                    next_stage = MaturityStage(func.current_stage.value + 1)
                    item = RoadmapItem(
                        pillar=pillar_name,
                        function_name=func.function_name,
                        current_stage=func.current_stage.name,
                        target_stage=next_stage.name,
                        priority=max(1, 4 - gap),
                        effort=effort_map.get(func.current_stage.value, "high"),
                        recommendations=func.recommendations,
                    )
                    self.roadmap.append(item)

        self.roadmap.sort(key=lambda x: (x.priority, x.effort != "low"))
        return self.roadmap

    def get_compliance_status(self) -> dict:
        status = {}
        for pillar_name, pa in self.pillar_assessments.items():
            req = OMB_M2209_REQUIREMENTS.get(pillar_name, {})
            status[pillar_name] = {
                "current_stage": pa.overall_stage.name,
                "required_stage": req.get("target_stage", MaturityStage.ADVANCED).name,
                "compliant": not pa.compliance_gap,
                "requirement": req.get("requirement", "N/A"),
                "key_actions": req.get("key_actions", []),
            }
        return status

    def get_summary(self) -> dict:
        summary = {
            "organization": self.organization,
            "assessment_date": self.assessment_date,
            "pillars": {},
            "overall_maturity": "TRADITIONAL",
        }
        stages = []
        for name, pa in self.pillar_assessments.items():
            summary["pillars"][name] = {
                "stage": pa.overall_stage.name,
                "score": pa.overall_stage.value,
                "functions_assessed": len(pa.functions),
                "compliance_gap": pa.compliance_gap,
            }
            stages.append(pa.overall_stage.value)

        if stages:
            avg = sum(stages) / len(stages)
            summary["overall_maturity"] = MaturityStage(int(avg)).name
            summary["overall_score"] = round(avg, 2)

        return summary

    def export_report(self, output_path: str):
        report = {
            "summary": self.get_summary(),
            "compliance_status": self.get_compliance_status(),
            "roadmap": [
                {
                    "pillar": r.pillar,
                    "function": r.function_name,
                    "current": r.current_stage,
                    "target": r.target_stage,
                    "priority": r.priority,
                    "effort": r.effort,
                }
                for r in self.roadmap
            ],
            "detailed_assessments": {},
        }

        for name, pa in self.pillar_assessments.items():
            report["detailed_assessments"][name] = {
                "overall_stage": pa.overall_stage.name,
                "functions": [
                    {
                        "name": f.function_name,
                        "current": f.current_stage.name,
                        "target": f.target_stage.name,
                        "evidence": f.evidence,
                        "gaps": f.gaps,
                    }
                    for f in pa.functions
                ],
            }

        path = Path(output_path)
        path.parent.mkdir(parents=True, exist_ok=True)
        with open(path, "w") as f:
            json.dump(report, f, indent=2)
        return report

    def export_csv(self, output_path: str):
        path = Path(output_path)
        path.parent.mkdir(parents=True, exist_ok=True)
        with open(path, "w", newline="") as f:
            writer = csv.writer(f)
            writer.writerow([
                "Pillar", "Function", "Current Stage", "Target Stage",
                "Gap", "Priority", "Effort", "Evidence"
            ])
            for name, pa in self.pillar_assessments.items():
                for func in pa.functions:
                    gap = func.target_stage.value - func.current_stage.value
                    writer.writerow([
                        name, func.function_name,
                        func.current_stage.name, func.target_stage.name,
                        gap, max(1, 4 - gap),
                        "high" if func.current_stage == MaturityStage.TRADITIONAL else "medium",
                        func.evidence,
                    ])


def run_sample_assessment():
    """Run a sample assessment demonstrating the ZTMM assessment process."""
    assessment = ZTMMAssessment("Example Federal Agency")

    # Identity Pillar Assessment
    assessment.assess_function("Identity", "authentication", 1, 3,
                               evidence="MFA deployed for 60% of users, not phishing-resistant",
                               gaps=["No FIDO2/WebAuthn deployment", "Service accounts lack MFA"])
    assessment.assess_function("Identity", "identity_stores", 1, 3,
                               evidence="Centralized AD, partial cloud identity integration")
    assessment.assess_function("Identity", "risk_assessment", 0, 3,
                               evidence="No risk-based authentication in place")
    assessment.assess_function("Identity", "access_management", 1, 3,
                               evidence="Basic RBAC, no attribute-based access control")
    assessment.assess_function("Identity", "identity_lifecycle", 1, 3,
                               evidence="Manual provisioning, no HR integration")

    # Devices Pillar Assessment
    assessment.assess_function("Devices", "policy_enforcement", 1, 3,
                               evidence="MDM deployed, basic compliance checks")
    assessment.assess_function("Devices", "asset_management", 1, 3,
                               evidence="Partial inventory, no IoT coverage")
    assessment.assess_function("Devices", "device_compliance", 0, 3,
                               evidence="No real-time compliance checking")
    assessment.assess_function("Devices", "device_threat_protection", 1, 3,
                               evidence="EDR on 70% of endpoints")

    # Networks Pillar Assessment
    assessment.assess_function("Networks", "network_segmentation", 0, 3,
                               evidence="Flat network, basic VLAN segmentation only")
    assessment.assess_function("Networks", "threat_protection", 1, 3,
                               evidence="Perimeter firewall, no NDR")
    assessment.assess_function("Networks", "encryption", 1, 3,
                               evidence="TLS for external, plaintext internal")
    assessment.assess_function("Networks", "network_resilience", 1, 3,
                               evidence="Basic redundancy, no SD-WAN")

    # Applications Pillar Assessment
    assessment.assess_function("Applications", "access_authorization", 1, 3,
                               evidence="VPN-based access for internal apps")
    assessment.assess_function("Applications", "threat_protection", 1, 3,
                               evidence="WAF for public apps only")
    assessment.assess_function("Applications", "application_security", 0, 3,
                               evidence="Annual pen tests, no CI/CD security integration")

    # Data Pillar Assessment
    assessment.assess_function("Data", "data_inventory", 0, 3,
                               evidence="No comprehensive data inventory")
    assessment.assess_function("Data", "data_categorization", 1, 3,
                               evidence="Basic classification, manual process")
    assessment.assess_function("Data", "data_access", 1, 3,
                               evidence="Role-based access, no fine-grained controls")
    assessment.assess_function("Data", "data_encryption", 1, 3,
                               evidence="Encryption at rest for databases, not all storage")

    # Calculate and report
    assessment.calculate_maturity()
    roadmap = assessment.generate_roadmap()

    print("=" * 70)
    print(f"CISA ZTMM Assessment: {assessment.organization}")
    print(f"Date: {assessment.assessment_date}")
    print("=" * 70)

    summary = assessment.get_summary()
    print(f"\nOverall Maturity: {summary['overall_maturity']} (Score: {summary.get('overall_score', 'N/A')})")
    print("\nPillar Maturity Scores:")
    for pillar, data in summary["pillars"].items():
        compliance = " [COMPLIANCE GAP]" if data["compliance_gap"] else " [COMPLIANT]"
        print(f"  {pillar:20s}: {data['stage']:12s} (Score: {data['score']}){compliance}")

    print("\nOMB M-22-09 Compliance Status:")
    compliance = assessment.get_compliance_status()
    for pillar, status in compliance.items():
        icon = "PASS" if status["compliant"] else "FAIL"
        print(f"  [{icon}] {pillar}: {status['current_stage']} / Required: {status['required_stage']}")

    print(f"\nPrioritized Roadmap ({len(roadmap)} items):")
    for i, item in enumerate(roadmap[:10], 1):
        print(f"  {i}. [{item.pillar}] {item.function_name}: "
              f"{item.current_stage} -> {item.target_stage} "
              f"(Priority: {item.priority}, Effort: {item.effort})")

    # Export reports
    assessment.export_report("ztmm_assessment_report.json")
    assessment.export_csv("ztmm_assessment_report.csv")
    print("\nReports exported: ztmm_assessment_report.json, ztmm_assessment_report.csv")


if __name__ == "__main__":
    run_sample_assessment()

===== END FILE =====


===== FILE: skills/implementing-device-posture-assessment-in-zero-trust/SKILL.md =====
---
name: implementing-device-posture-assessment-in-zero-trust
description: 'Implements device posture assessment as a zero trust access control
  by integrating endpoint health signals from CrowdStrike ZTA, Microsoft Intune, and
  Jamf into conditional access policies that enforce compliance before granting access.
  Use when requiring device health checks before app access, wiring posture signals
  into conditional access, or implementing the CISA Zero Trust device pillar.

  '
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- device-posture
- zero-trust
- endpoint-compliance
- crowdstrike-zta
- intune
- conditional-access
- jamf
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078
- T1190
- T1059
---

# Implementing Device Posture Assessment in Zero Trust

## When to Use

- When enforcing device health as a prerequisite for accessing corporate applications
- When integrating CrowdStrike ZTA scores, Intune compliance, or Jamf device status into access decisions
- When implementing CISA Zero Trust Maturity Model device pillar requirements
- When building conditional access policies that adapt based on real-time endpoint security posture
- When detecting and blocking access from compromised, unmanaged, or non-compliant devices

**Do not use** for IoT or headless devices that cannot run posture agents, as a standalone security control without identity verification, or when real-time posture data is unavailable and stale compliance data would create false trust.

## Prerequisites

- Endpoint Detection and Response (EDR): CrowdStrike Falcon with ZTA module, or Microsoft Defender for Endpoint
- Mobile Device Management (MDM): Microsoft Intune, Jamf Pro, or VMware Workspace ONE
- Identity Provider: Microsoft Entra ID, Okta, or Ping Identity with conditional access capability
- ZTNA Platform: Zscaler ZPA, Cloudflare Access, Palo Alto Prisma Access, or cloud-native IAP
- API access to EDR/MDM platforms for posture signal ingestion

## Workflow

### Step 1: Define Device Compliance Baselines

Establish minimum security requirements for each device category.

```powershell
# Microsoft Intune: Create device compliance policy via Graph API
Connect-MgGraph -Scopes "DeviceManagementConfiguration.ReadWrite.All"

# Windows 10/11 Compliance Policy
$compliancePolicy = @{
    "@odata.type" = "#microsoft.graph.windows10CompliancePolicy"
    displayName = "Zero Trust - Windows Compliance"
    description = "Minimum device requirements for zero trust access"
    osMinimumVersion = "10.0.19045"
    bitLockerEnabled = $true
    secureBootEnabled = $true
    codeIntegrityEnabled = $true
    tpmRequired = $true
    antivirusRequired = $true
    antiSpywareRequired = $true
    defenderEnabled = $true
    firewallEnabled = $true
    passwordRequired = $true
    passwordMinimumLength = 12
    passwordRequiredType = "alphanumeric"
    storageRequireEncryption = $true
    scheduledActionsForRule = @(
        @{
            ruleName = "PasswordRequired"
            scheduledActionConfigurations = @(
                @{
                    actionType = "block"
                    gracePeriodHours = 24
                    notificationTemplateId = ""
                    notificationMessageCCList = @()
                }
            )
        }
    )
}

New-MgDeviceManagementDeviceCompliancePolicy -BodyParameter $compliancePolicy

# macOS Compliance Policy via Jamf Pro API
curl -X POST "https://jamf.company.com/api/v1/compliance-policies" \
  -H "Authorization: Bearer ${JAMF_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Zero Trust - macOS Compliance",
    "rules": [
      {"type": "os_version", "operator": ">=", "value": "14.0"},
      {"type": "filevault_enabled", "value": true},
      {"type": "firewall_enabled", "value": true},
      {"type": "gatekeeper_enabled", "value": true},
      {"type": "sip_enabled", "value": true},
      {"type": "auto_update_enabled", "value": true},
      {"type": "screen_lock_timeout", "operator": "<=", "value": 300},
      {"type": "falcon_sensor_running", "value": true}
    ]
  }'
```

### Step 2: Configure CrowdStrike Zero Trust Assessment

Enable ZTA scoring and configure score thresholds for access tiers.

```bash
# CrowdStrike Falcon API: Query ZTA scores for all endpoints
curl -X GET "https://api.crowdstrike.com/zero-trust-assessment/entities/assessments/v1?ids=${DEVICE_AID}" \
  -H "Authorization: Bearer ${CS_TOKEN}" \
  -H "Content-Type: application/json"

# Response includes:
# {
#   "aid": "device-agent-id",
#   "assessment": {
#     "overall": 82,
#     "os": 90,
#     "sensor_config": 85,
#     "version": "7.14.16703"
#   },
#   "assessment_items": {
#     "os_signals": [
#       {"signal_id": "firmware_protection", "meets_criteria": "yes"},
#       {"signal_id": "disk_encryption", "meets_criteria": "yes"},
#       {"signal_id": "kernel_protection", "meets_criteria": "yes"}
#     ],
#     "sensor_signals": [
#       {"signal_id": "sensor_version", "meets_criteria": "yes"},
#       {"signal_id": "prevention_policies", "meets_criteria": "yes"}
#     ]
#   }
# }

# Define ZTA score thresholds for access tiers
# Tier 1 (Basic Access):      ZTA >= 50
# Tier 2 (Standard Access):   ZTA >= 65
# Tier 3 (Sensitive Access):  ZTA >= 80
# Tier 4 (Critical Access):   ZTA >= 90

# Query devices below minimum threshold
curl -X GET "https://api.crowdstrike.com/zero-trust-assessment/queries/assessments/v1?filter=assessment.overall:<50" \
  -H "Authorization: Bearer ${CS_TOKEN}"

# CrowdStrike ZTA signals evaluated:
# - OS patch level and version
# - Disk encryption (BitLocker/FileVault)
# - Sensor version and configuration
# - Prevention policy enforcement
# - Firmware protection (Secure Boot)
# - Kernel protection (SIP, Code Integrity)
# - Firewall status
```

### Step 3: Integrate Device Posture with Entra ID Conditional Access

Create conditional access policies that require compliant devices.

```powershell
# Create Conditional Access policy requiring compliant device
Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess"

$caPolicy = @{
    displayName = "Zero Trust - Require Compliant Device"
    state = "enabled"
    conditions = @{
        applications = @{
            includeApplications = @("All")
        }
        users = @{
            includeUsers = @("All")
            excludeGroups = @("BreakGlass-Admins-Group-ID")
        }
        platforms = @{
            includePlatforms = @("all")
        }
        clientAppTypes = @("browser", "mobileAppsAndDesktopClients")
    }
    grantControls = @{
        operator = "AND"
        builtInControls = @("mfa", "compliantDevice")
    }
    sessionControls = @{
        signInFrequency = @{
            value = 4
            type = "hours"
            isEnabled = $true
            authenticationType = "primaryAndSecondaryAuthentication"
            frequencyInterval = "timeBased"
        }
        persistentBrowser = @{
            mode = "never"
            isEnabled = $true
        }
    }
}

New-MgIdentityConditionalAccessPolicy -BodyParameter $caPolicy

# Create risk-based policy using device compliance + sign-in risk
$riskPolicy = @{
    displayName = "Zero Trust - Block High Risk Sign-Ins on Non-Compliant Devices"
    state = "enabled"
    conditions = @{
        applications = @{ includeApplications = @("All") }
        users = @{ includeUsers = @("All") }
        signInRiskLevels = @("high", "medium")
        devices = @{
            deviceFilter = @{
                mode = "include"
                rule = "device.isCompliant -ne True"
            }
        }
    }
    grantControls = @{
        operator = "OR"
        builtInControls = @("block")
    }
}

New-MgIdentityConditionalAccessPolicy -BodyParameter $riskPolicy
```

### Step 4: Configure Okta Device Trust with CrowdStrike Integration

Set up Okta device trust policies using CrowdStrike posture signals.

```bash
# Okta: Configure CrowdStrike device trust integration
# Admin Console > Security > Device Integrations > Add Integration

# Okta API: Create device assurance policy
curl -X POST "https://company.okta.com/api/v1/device-assurances" \
  -H "Authorization: SSWS ${OKTA_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Corporate Device Assurance",
    "platform": "WINDOWS",
    "osVersion": {
      "minimum": "10.0.19045"
    },
    "diskEncryptionType": {
      "include": ["ALL_INTERNAL_VOLUMES"]
    },
    "screenLockType": {
      "include": ["BIOMETRIC", "PASSCODE"]
    },
    "secureHardwarePresent": true,
    "thirdPartySignalProviders": {
      "dtc": {
        "browserVersion": {
          "minimum": "120.0"
        },
        "builtInDnsClientEnabled": true,
        "chromeRemoteDesktopAppBlocked": true,
        "crowdStrikeCustomerId": "CS_CUSTOMER_ID",
        "crowdStrikeAgentId": "REQUIRED",
        "crowdStrikeVerifiedState": {
          "include": ["RUNNING"]
        }
      }
    }
  }'

# Create Okta authentication policy with device assurance
curl -X POST "https://company.okta.com/api/v1/policies" \
  -H "Authorization: SSWS ${OKTA_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Zero Trust Application Policy",
    "type": "ACCESS_POLICY",
    "conditions": null,
    "rules": [
      {
        "name": "Managed Device Access",
        "conditions": {
          "device": {
            "assurance": {
              "include": ["DEVICE_ASSURANCE_POLICY_ID"]
            },
            "managed": true,
            "registered": true
          },
          "people": {
            "groups": {"include": ["EMPLOYEES_GROUP_ID"]}
          }
        },
        "actions": {
          "appSignOn": {
            "access": "ALLOW",
            "verificationMethod": {
              "factorMode": "1FA",
              "type": "ASSURANCE"
            }
          }
        }
      },
      {
        "name": "Unmanaged Device - Block",
        "conditions": {
          "device": { "managed": false }
        },
        "actions": {
          "appSignOn": { "access": "DENY" }
        }
      }
    ]
  }'
```

### Step 5: Implement Continuous Posture Monitoring

Set up real-time monitoring of device compliance state changes.

```python
#!/usr/bin/env python3
"""Monitor device posture compliance drift in real-time."""

import requests
import time
import json
from datetime import datetime, timezone

CROWDSTRIKE_BASE = "https://api.crowdstrike.com"
INTUNE_BASE = "https://graph.microsoft.com/v1.0"

def get_cs_token(client_id: str, client_secret: str) -> str:
    resp = requests.post(f"{CROWDSTRIKE_BASE}/oauth2/token", data={
        "client_id": client_id,
        "client_secret": client_secret
    })
    return resp.json()["access_token"]

def get_low_zta_devices(token: str, threshold: int = 50) -> list:
    resp = requests.get(
        f"{CROWDSTRIKE_BASE}/zero-trust-assessment/queries/assessments/v1",
        headers={"Authorization": f"Bearer {token}"},
        params={"filter": f"assessment.overall:<{threshold}", "limit": 100}
    )
    return resp.json().get("resources", [])

def get_intune_noncompliant(token: str) -> list:
    resp = requests.get(
        f"{INTUNE_BASE}/deviceManagement/managedDevices",
        headers={"Authorization": f"Bearer {token}"},
        params={
            "$filter": "complianceState eq 'noncompliant'",
            "$select": "id,deviceName,userPrincipalName,complianceState,lastSyncDateTime,operatingSystem"
        }
    )
    return resp.json().get("value", [])

def check_posture_drift(cs_token: str, intune_token: str):
    print(f"\n[{datetime.now(timezone.utc).isoformat()}] Device Posture Check")
    print("=" * 60)

    low_zta = get_low_zta_devices(cs_token, threshold=50)
    print(f"CrowdStrike ZTA < 50: {len(low_zta)} devices")

    noncompliant = get_intune_noncompliant(intune_token)
    print(f"Intune Non-Compliant: {len(noncompliant)} devices")

    for device in noncompliant[:10]:
        print(f"  - {device['deviceName']} ({device['userPrincipalName']}): "
              f"{device['complianceState']} | Last sync: {device['lastSyncDateTime']}")

    return {"low_zta_count": len(low_zta), "noncompliant_count": len(noncompliant)}
```

## Key Concepts

| Term | Definition |
|------|------------|
| Device Posture | Collection of endpoint security attributes (OS version, encryption, EDR status, patch level) evaluated before granting access |
| CrowdStrike ZTA Score | Numerical score (1-100) calculated by CrowdStrike Falcon assessing endpoint security posture based on OS signals and sensor configuration |
| Device Compliance Policy | MDM-defined rules specifying minimum security requirements (encryption, PIN, OS version) that devices must meet |
| Conditional Access | Policy engine (Entra ID, Okta) that evaluates user identity, device compliance, location, and risk before allowing access |
| Device Trust | Verification that an endpoint is managed, enrolled, and meets security baselines before treating it as trusted |
| Posture Drift | Degradation of device security posture over time (expired patches, disabled encryption) that should trigger access revocation |

## Tools & Systems

- **CrowdStrike Falcon ZTA**: Real-time endpoint posture scoring based on OS and sensor security signals
- **Microsoft Intune**: MDM platform enforcing device compliance policies and reporting to Entra ID Conditional Access
- **Jamf Pro**: Apple device management with compliance rules for macOS and iOS endpoints
- **Microsoft Entra ID Conditional Access**: Policy engine consuming Intune compliance and risk signals for access decisions
- **Okta Device Trust**: Device assurance policies integrating with CrowdStrike, Chrome Enterprise, and MDM platforms
- **Cloudflare Device Posture**: WARP client-based posture checks for disk encryption, OS version, and third-party EDR

## Common Scenarios

### Scenario: Enforcing Device Compliance for 2,000 Endpoints Across Windows and macOS

**Context**: A healthcare company with 2,000 endpoints (70% Windows, 30% macOS) must enforce HIPAA-compliant device posture before allowing access to patient data systems. Devices are managed by Intune (Windows) and Jamf (macOS) with CrowdStrike Falcon deployed on all endpoints.

**Approach**:
1. Define Windows compliance policy in Intune: BitLocker, Secure Boot, TPM, Defender enabled, OS >= 10.0.19045
2. Define macOS compliance policy in Jamf: FileVault, Gatekeeper, SIP, Firewall, OS >= 14.0
3. Configure CrowdStrike ZTA thresholds: >= 70 for general apps, >= 85 for patient data systems
4. Create Entra ID Conditional Access policies requiring compliant device + MFA for all cloud apps
5. Configure 24-hour grace period for newly non-compliant devices before blocking
6. Set up weekly compliance report for IT showing non-compliant devices and remediation actions
7. Implement automated remediation via Intune: push BitLocker enablement, deploy pending patches

**Pitfalls**: Grace periods must be long enough for IT to remediate but short enough to limit risk exposure. CrowdStrike ZTA scores can fluctuate with sensor updates; avoid setting thresholds too aggressively initially. BYOD devices may lack MDM enrollment; provide a separate Browser Access path with reduced functionality for unmanaged devices.

## Output Format

```
Device Posture Assessment Report
==================================================
Organization: HealthCorp
Report Date: 2026-02-23
Total Managed Devices: 2,000

COMPLIANCE BY PLATFORM:
  Windows (1,400 devices):
    Compliant:              1,302 (93.0%)
    Non-compliant:            98 (7.0%)
    Top Issue: Missing patches (45), BitLocker disabled (23)

  macOS (600 devices):
    Compliant:                567 (94.5%)
    Non-compliant:             33 (5.5%)
    Top Issue: OS outdated (18), FileVault disabled (8)

CROWDSTRIKE ZTA SCORES:
  Average Score:              78.4
  Devices >= 85 (Critical):  1,456 (72.8%)
  Devices >= 70 (Standard):  1,812 (90.6%)
  Devices < 50 (Blocked):       34 (1.7%)

CONDITIONAL ACCESS IMPACT (last 7 days):
  Total sign-in attempts:    45,678
  Blocked by posture:           312 (0.7%)
  Remediated within 24h:        289 (92.6%)
  Still non-compliant:           23

POSTURE DRIFT ALERTS:
  Encryption disabled:            5
  EDR sensor stopped:             3
  OS downgraded:                  1
```

===== END FILE =====


===== FILE: skills/implementing-device-posture-assessment-in-zero-trust/LICENSE =====

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


===== FILE: skills/implementing-device-posture-assessment-in-zero-trust/assets/template.md =====
# Device Posture Assessment - Compliance Requirements Template

## Organization Information
| Field | Value |
|-------|-------|
| Organization | HealthCorp Medical |
| Total Endpoints | 2,000 |
| MDM Platform | Microsoft Intune (Windows), Jamf Pro (macOS) |
| EDR Platform | CrowdStrike Falcon |
| IdP | Microsoft Entra ID |

## Device Posture Tiers

### Tier 1: Basic Access (General Applications)
| Requirement | Windows | macOS | iOS/Android |
|------------|---------|-------|-------------|
| OS Version | >= 10.0.19045 | >= 14.0 | Latest -2 |
| Screen Lock | Enabled | Enabled | Enabled (6-digit PIN) |
| Encryption | Recommended | Recommended | Required |
| Firewall | Enabled | Enabled | N/A |
| CrowdStrike ZTA | >= 50 | >= 50 | N/A |
| Applications | Email, Wiki, Chat | Email, Wiki, Chat | Email, Chat |

### Tier 2: Standard Access (Business Applications)
| Requirement | Windows | macOS |
|------------|---------|-------|
| OS Version | >= 10.0.22621 | >= 14.0 |
| Screen Lock | Enabled, 5min timeout | Enabled, 5min timeout |
| Disk Encryption | BitLocker Required | FileVault Required |
| Firewall | Enabled | Enabled |
| Antivirus | Defender Active | XProtect Active |
| CrowdStrike ZTA | >= 65 | >= 65 |
| Secure Boot | Required | N/A |
| Applications | CRM, Internal Tools, Intranet | CRM, Internal Tools, Intranet |

### Tier 3: Enhanced Access (Sensitive Data)
| Requirement | Windows | macOS |
|------------|---------|-------|
| OS Version | >= 10.0.22631 (latest) | >= 15.0 (latest) |
| Disk Encryption | BitLocker + TPM | FileVault + Secure Enclave |
| CrowdStrike ZTA | >= 80 | >= 80 |
| Patch Level | Within 14 days of release | Within 14 days of release |
| Admin Approval | Device admin-approved | Device admin-approved |
| MDM Managed | Required | Required |
| Applications | Financial Systems, HR Data, Source Code | Financial Systems, HR Data |

### Tier 4: Critical Access (Regulated Data)
| Requirement | Windows | macOS |
|------------|---------|-------|
| CrowdStrike ZTA | >= 90 | >= 90 |
| Patch Level | Within 7 days | Within 7 days |
| Code Integrity | HVCI Enabled | SIP Enabled |
| Certificate | Corporate certificate present | Corporate certificate present |
| Geo Restriction | US only | US only |
| Applications | Patient Records (HIPAA), PCI Data | Patient Records (HIPAA) |

## Compliance Policy Assignments

| Policy | Device Group | Grace Period | Non-Compliance Action |
|--------|-------------|-------------|----------------------|
| Tier 1 Baseline | All Managed Devices | 72 hours | Email notification |
| Tier 2 Standard | Corporate Workstations | 48 hours | Mark non-compliant |
| Tier 3 Enhanced | Sensitive Data Users | 24 hours | Block access |
| Tier 4 Critical | HIPAA/PCI Users | 4 hours | Block access + alert SOC |

## CrowdStrike ZTA Score Mapping

| ZTA Score Range | Access Tier | Conditional Access Action |
|----------------|-------------|--------------------------|
| 90-100 | Tier 4 (Critical) | Allow all applications |
| 80-89 | Tier 3 (Enhanced) | Allow up to sensitive apps |
| 65-79 | Tier 2 (Standard) | Allow business apps only |
| 50-64 | Tier 1 (Basic) | Allow general apps only |
| < 50 | BLOCKED | Block all access, notify IT |

## Current Compliance Dashboard

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| Overall Compliance Rate | >= 95% | 93.2% | At Risk |
| Encryption Coverage | 100% | 97.8% | On Track |
| EDR Coverage | 100% | 99.1% | On Track |
| Average ZTA Score | >= 75 | 78.4 | Meeting |
| Devices Below ZTA 50 | 0 | 34 | Action Needed |
| Stale Devices (>30d) | < 1% | 0.8% | Meeting |
| Patch Compliance (14d) | >= 90% | 87.3% | At Risk |

## Remediation Actions

| Issue | Automated Remediation | Manual Escalation |
|-------|----------------------|-------------------|
| BitLocker Disabled | Intune policy pushes enablement | IT ticket after 24h |
| FileVault Disabled | Jamf policy enables FileVault | IT ticket after 24h |
| OS Outdated | Intune/Jamf pushes update | IT ticket after 48h |
| CrowdStrike Stopped | Auto-restart via watchdog | SOC alert after 30min |
| Firewall Disabled | GPO re-enables | IT ticket after 4h |

## Sign-Off

| Role | Name | Date | Approved |
|------|------|------|----------|
| CISO | _________________ | __________ | [ ] |
| IT Operations Director | _________________ | __________ | [ ] |
| Compliance Officer | _________________ | __________ | [ ] |

===== END FILE =====


===== FILE: skills/implementing-device-posture-assessment-in-zero-trust/references/api-reference.md =====
# API Reference: Device Posture Assessment Agent

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| (stdlib only) | Python 3.8+ | Platform detection, subprocess for OS security checks |

## CLI Usage

```bash
python scripts/agent.py --output-dir /reports/ --output device_posture.json
```

## Functions

### `check_os_version() -> dict`
Uses `platform.system()`, `platform.version()` for OS identification.

### `check_disk_encryption() -> dict`
Windows: `manage-bde -status C:` (BitLocker). macOS: `fdesetup status` (FileVault). Linux: `lsblk` for LUKS.

### `check_firewall_status() -> dict`
Windows: `netsh advfirewall show allprofiles state`. Linux: `ufw status`.

### `check_antivirus() -> dict`
Windows: PowerShell `Get-MpComputerStatus` for Defender real-time protection.

### `check_screen_lock() -> dict`
Windows: Registry `InactivityTimeoutSecs` check.

### `compute_posture_score(checks) -> dict`
Weighted scoring: encryption (25), firewall (20), AV (25), screen lock (15), OS (15). Returns COMPLIANT/PARTIAL/NON_COMPLIANT.

## Posture Checks

| Check | Weight | Tool |
|-------|--------|------|
| Disk Encryption | 25 | BitLocker/FileVault/LUKS |
| Firewall | 20 | Windows Firewall/UFW |
| Antivirus/EDR | 25 | Defender/endpoint agent |
| Screen Lock | 15 | OS policy |
| OS Supported | 15 | Platform detection |

## Output Schema

```json
{
  "hostname": "WORKSTATION-01",
  "posture": {"score": 85, "compliance": "COMPLIANT"},
  "recommendations": ["Enable disk encryption"]
}
```

===== END FILE =====


===== FILE: skills/implementing-device-posture-assessment-in-zero-trust/references/standards.md =====
# Device Posture Assessment - Standards & References

## NIST SP 800-207: Zero Trust Architecture
- **Section 2, Tenet 5**: "The enterprise monitors and measures the integrity and security posture of all owned and associated assets"
- **Section 3.3**: Agent/Gateway Model includes device health as access input
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-207/final

## CISA Zero Trust Maturity Model v2.0 - Device Pillar
- **Traditional**: Limited visibility into device health
- **Initial**: Compliance enforcement via MDM
- **Advanced**: Continuous monitoring with automated remediation
- **Optimal**: Real-time posture integrated into every access decision
- **URL**: https://www.cisa.gov/zero-trust-maturity-model

## NIST SP 800-124r2: Guidelines for Managing Mobile Device Security
- **Section 3.2**: Device compliance checking requirements
- **Section 4.1**: MDM security capabilities for posture enforcement
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-124/rev-2/final

## CrowdStrike ZTA Documentation
- **ZTA Overview**: https://www.crowdstrike.com/products/zero-trust-protection/
- **ZTA API**: https://falcon.crowdstrike.com/documentation/156/zero-trust-assessment-apis
- **ZTA Scoring Methodology**: OS signals + sensor configuration signals

## Microsoft Intune Compliance
- **Compliance Policies**: https://learn.microsoft.com/en-us/mem/intune/protect/device-compliance-get-started
- **Conditional Access Integration**: https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-grant
- **Device Health Attestation**: https://learn.microsoft.com/en-us/windows/security/operating-system-security/system-security/protect-high-value-assets-by-controlling-the-health-of-windows-10-based-devices

## Jamf Pro Compliance
- **Smart Groups**: https://learn.jamf.com/en-US/bundle/jamf-pro-documentation-current/page/Smart_Groups.html
- **Compliance Reporter**: https://learn.jamf.com/en-US/bundle/jamf-compliance-editor-documentation/page/Jamf_Compliance_Editor.html

## HIPAA Security Rule (45 CFR 164.312)
- **(a)(1)**: Access control - device posture as access control mechanism
- **(d)**: Device and media controls - encryption and integrity requirements

===== END FILE =====


===== FILE: skills/implementing-device-posture-assessment-in-zero-trust/references/workflows.md =====
# Device Posture Assessment Implementation Workflow

## Phase 1: Baseline Assessment (Week 1)

### 1.1 Inventory Current State
1. Export all managed devices from Intune/Jamf/SCCM
2. Identify unmanaged devices accessing corporate resources
3. Document OS distribution, patch levels, and encryption status
4. Measure current compliance rate before enforcement

### 1.2 Define Posture Requirements
1. Establish minimum requirements per device tier:
   - **Tier 1 (Basic)**: OS updated within 90 days, screen lock enabled
   - **Tier 2 (Standard)**: Disk encryption, firewall, antivirus, OS within 60 days
   - **Tier 3 (Enhanced)**: EDR running, ZTA score >= 70, OS within 30 days, TPM/Secure Boot
   - **Tier 4 (Critical)**: ZTA score >= 90, fully managed, patched within 7 days
2. Map application sensitivity to required posture tier
3. Define grace periods for remediation (24h standard, 4h for critical)

## Phase 2: MDM Policy Configuration (Week 2-3)

### 2.1 Intune Compliance Policies
1. Create Windows compliance policy: BitLocker, Secure Boot, TPM, Defender, OS version
2. Create macOS compliance policy: FileVault, Gatekeeper, SIP, Firewall
3. Create iOS/Android compliance policy: Encryption, PIN, jailbreak detection
4. Configure non-compliance actions: email notification, mark non-compliant, block after grace
5. Assign policies to device groups

### 2.2 Jamf Pro Configuration
1. Create smart groups for compliant/non-compliant macOS devices
2. Configure compliance criteria: FileVault, SIP, Gatekeeper, OS version
3. Set up automated remediation scripts for common issues
4. Configure compliance reporting to Jamf Protect or SIEM

## Phase 3: EDR Integration (Week 3-4)

### 3.1 CrowdStrike ZTA Setup
1. Enable Zero Trust Assessment module in Falcon console
2. Configure ZTA score thresholds per access tier
3. Set up API integration for ZTNA platform (Zscaler, Cloudflare, Okta)
4. Create host groups for ZTA monitoring
5. Build dashboard for ZTA score distribution

### 3.2 Microsoft Defender for Endpoint
1. Enable device risk assessment in Defender Security Center
2. Configure risk levels: Low, Medium, High, Critical
3. Integrate with Intune compliance via Defender connector
4. Set up conditional access policy consuming device risk signal

## Phase 4: Conditional Access Configuration (Week 4-5)

### 4.1 Entra ID Conditional Access
1. Create policy: Require compliant device for all cloud apps
2. Create policy: Block high-risk devices from sensitive apps
3. Create policy: Require MFA + compliant device for admin portals
4. Configure break-glass exclusions for emergency access
5. Start in report-only mode, then switch to enforcement

### 4.2 Okta Device Trust
1. Configure device trust integration with MDM platforms
2. Create device assurance policies with CrowdStrike integration
3. Set up authentication policies requiring device trust
4. Test with enrolled and non-enrolled devices

## Phase 5: Monitoring and Remediation (Ongoing)

1. Build compliance dashboard showing real-time posture across fleet
2. Configure alerts for posture drift (encryption disabled, EDR stopped)
3. Automate remediation: push encryption enablement, deploy patches
4. Generate weekly compliance reports for security leadership
5. Conduct monthly review of posture requirements vs. threat landscape

===== END FILE =====


===== FILE: skills/implementing-device-posture-assessment-in-zero-trust/scripts/agent.py =====
#!/usr/bin/env python3
"""Device posture assessment agent for zero trust endpoint compliance evaluation."""

import argparse
import json
import logging
import os
import platform
import subprocess
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def check_os_version() -> dict:
    """Check OS version and patch level."""
    return {
        "os": platform.system(),
        "version": platform.version(),
        "release": platform.release(),
        "machine": platform.machine(),
    }


def check_disk_encryption() -> dict:
    """Check if disk encryption is enabled."""
    system = platform.system()
    if system == "Windows":
        try:
            result = subprocess.run(
                ["manage-bde", "-status", "C:"], capture_output=True, text=True, timeout=10)
            encrypted = "Fully Encrypted" in result.stdout or "100%" in result.stdout
            return {"enabled": encrypted, "tool": "BitLocker", "output": result.stdout[:200]}
        except FileNotFoundError:
            return {"enabled": False, "tool": "BitLocker", "error": "manage-bde not found"}
    elif system == "Darwin":
        try:
            result = subprocess.run(
                ["fdesetup", "status"], capture_output=True, text=True, timeout=10)
            return {"enabled": "On" in result.stdout, "tool": "FileVault"}
        except FileNotFoundError:
            return {"enabled": False, "error": "fdesetup not found"}
    elif system == "Linux":
        try:
            result = subprocess.run(
                ["lsblk", "-o", "NAME,TYPE,FSTYPE"], capture_output=True, text=True, timeout=10)
            encrypted = "crypto_LUKS" in result.stdout or "crypt" in result.stdout
            return {"enabled": encrypted, "tool": "LUKS"}
        except FileNotFoundError:
            return {"enabled": False, "error": "lsblk not found"}
    return {"enabled": False, "error": "Unsupported OS"}


def check_firewall_status() -> dict:
    """Check if host firewall is enabled."""
    system = platform.system()
    if system == "Windows":
        try:
            result = subprocess.run(
                ["netsh", "advfirewall", "show", "allprofiles", "state"],
                capture_output=True, text=True, timeout=10)
            enabled = "ON" in result.stdout.upper()
            return {"enabled": enabled, "tool": "Windows Firewall"}
        except FileNotFoundError:
            return {"enabled": False, "error": "netsh not found"}
    elif system == "Linux":
        try:
            result = subprocess.run(
                ["ufw", "status"], capture_output=True, text=True, timeout=10)
            return {"enabled": "active" in result.stdout.lower(), "tool": "UFW"}
        except FileNotFoundError:
            return {"enabled": False, "error": "ufw not found"}
    return {"enabled": False, "error": "Unsupported OS"}


def check_antivirus() -> dict:
    """Check if antivirus/EDR is running."""
    system = platform.system()
    if system == "Windows":
        try:
            result = subprocess.run(
                ["powershell", "-Command", "Get-MpComputerStatus | Select-Object RealTimeProtectionEnabled | ConvertTo-Json"],
                capture_output=True, text=True, timeout=15)
            if result.stdout:
                data = json.loads(result.stdout)
                return {"enabled": data.get("RealTimeProtectionEnabled", False),
                        "tool": "Windows Defender"}
        except (FileNotFoundError, json.JSONDecodeError):
            pass
    return {"enabled": False, "tool": "unknown"}


def check_screen_lock() -> dict:
    """Check if screen lock is configured with timeout."""
    system = platform.system()
    if system == "Windows":
        try:
            result = subprocess.run(
                ["powershell", "-Command",
                 "(Get-ItemProperty 'HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Policies\\System').InactivityTimeoutSecs"],
                capture_output=True, text=True, timeout=10)
            timeout = int(result.stdout.strip()) if result.stdout.strip() else 0
            return {"configured": timeout > 0, "timeout_seconds": timeout}
        except (FileNotFoundError, ValueError):
            pass
    return {"configured": False, "timeout_seconds": 0}


def compute_posture_score(checks: dict) -> dict:
    """Compute device posture compliance score."""
    weights = {"disk_encryption": 25, "firewall": 20, "antivirus": 25,
               "screen_lock": 15, "os_supported": 15}
    score = 0
    if checks.get("disk_encryption", {}).get("enabled"):
        score += weights["disk_encryption"]
    if checks.get("firewall", {}).get("enabled"):
        score += weights["firewall"]
    if checks.get("antivirus", {}).get("enabled"):
        score += weights["antivirus"]
    if checks.get("screen_lock", {}).get("configured"):
        score += weights["screen_lock"]
    score += weights["os_supported"]
    if score >= 80:
        compliance = "COMPLIANT"
    elif score >= 50:
        compliance = "PARTIAL"
    else:
        compliance = "NON_COMPLIANT"
    return {"score": score, "max_score": 100, "compliance": compliance}


def generate_report() -> dict:
    """Generate device posture assessment report."""
    checks = {
        "os_info": check_os_version(),
        "disk_encryption": check_disk_encryption(),
        "firewall": check_firewall_status(),
        "antivirus": check_antivirus(),
        "screen_lock": check_screen_lock(),
    }
    posture = compute_posture_score(checks)
    recommendations = []
    if not checks["disk_encryption"].get("enabled"):
        recommendations.append("Enable disk encryption (BitLocker/FileVault/LUKS)")
    if not checks["firewall"].get("enabled"):
        recommendations.append("Enable host firewall")
    if not checks["antivirus"].get("enabled"):
        recommendations.append("Enable antivirus/EDR with real-time protection")
    return {
        "analysis_date": datetime.utcnow().isoformat(),
        "hostname": platform.node(),
        "checks": checks,
        "posture": posture,
        "recommendations": recommendations,
    }


def main():
    parser = argparse.ArgumentParser(description="Device Posture Assessment Agent")
    parser.add_argument("--output-dir", default=".")
    parser.add_argument("--output", default="device_posture_report.json")
    args = parser.parse_args()

    os.makedirs(args.output_dir, exist_ok=True)
    report = generate_report()
    out_path = os.path.join(args.output_dir, args.output)
    with open(out_path, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s", out_path)
    print(json.dumps(report["posture"], indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-device-posture-assessment-in-zero-trust/scripts/process.py =====
#!/usr/bin/env python3
"""
Device Posture Assessment - Compliance Audit Tool

Queries CrowdStrike ZTA scores, Microsoft Intune compliance status,
and generates a consolidated device posture compliance report.

Requirements:
    pip install requests msal pandas
"""

import json
import sys
from datetime import datetime, timezone
from typing import Any

import requests


class DevicePostureAuditor:
    """Audit device posture compliance across EDR and MDM platforms."""

    def __init__(self, cs_client_id: str, cs_client_secret: str,
                 azure_tenant_id: str, azure_client_id: str, azure_client_secret: str):
        self.cs_client_id = cs_client_id
        self.cs_client_secret = cs_client_secret
        self.azure_tenant_id = azure_tenant_id
        self.azure_client_id = azure_client_id
        self.azure_client_secret = azure_client_secret
        self.cs_token = None
        self.azure_token = None

    def authenticate_crowdstrike(self):
        """Get CrowdStrike API bearer token."""
        resp = requests.post(
            "https://api.crowdstrike.com/oauth2/token",
            data={
                "client_id": self.cs_client_id,
                "client_secret": self.cs_client_secret
            },
            timeout=30
        )
        resp.raise_for_status()
        self.cs_token = resp.json()["access_token"]
        print("[AUTH] CrowdStrike authenticated")

    def authenticate_azure(self):
        """Get Microsoft Graph API token."""
        resp = requests.post(
            f"https://login.microsoftonline.com/{self.azure_tenant_id}/oauth2/v2.0/token",
            data={
                "client_id": self.azure_client_id,
                "client_secret": self.azure_client_secret,
                "scope": "https://graph.microsoft.com/.default",
                "grant_type": "client_credentials"
            },
            timeout=30
        )
        resp.raise_for_status()
        self.azure_token = resp.json()["access_token"]
        print("[AUTH] Microsoft Graph authenticated")

    def get_zta_scores(self) -> dict[str, Any]:
        """Get CrowdStrike ZTA score distribution."""
        print("\n[1/4] Querying CrowdStrike ZTA scores...")
        headers = {"Authorization": f"Bearer {self.cs_token}"}

        # Get all device AIDs with ZTA assessments
        resp = requests.get(
            "https://api.crowdstrike.com/zero-trust-assessment/queries/assessments/v1",
            headers=headers,
            params={"limit": 5000},
            timeout=60
        )
        resp.raise_for_status()
        device_ids = resp.json().get("resources", [])

        if not device_ids:
            print("  No ZTA assessments found")
            return {"total": 0, "distribution": {}}

        # Get detailed assessments in batches of 100
        scores = []
        for i in range(0, len(device_ids), 100):
            batch = device_ids[i:i+100]
            resp = requests.get(
                "https://api.crowdstrike.com/zero-trust-assessment/entities/assessments/v1",
                headers=headers,
                params={"ids": batch},
                timeout=60
            )
            if resp.status_code == 200:
                for resource in resp.json().get("resources", []):
                    assessment = resource.get("assessment", {})
                    scores.append({
                        "aid": resource.get("aid"),
                        "overall": assessment.get("overall", 0),
                        "os_score": assessment.get("os", 0),
                        "sensor_score": assessment.get("sensor_config", 0)
                    })

        # Calculate distribution
        distribution = {
            "critical_90_100": sum(1 for s in scores if s["overall"] >= 90),
            "high_80_89": sum(1 for s in scores if 80 <= s["overall"] < 90),
            "medium_65_79": sum(1 for s in scores if 65 <= s["overall"] < 80),
            "low_50_64": sum(1 for s in scores if 50 <= s["overall"] < 65),
            "blocked_below_50": sum(1 for s in scores if s["overall"] < 50),
        }
        avg_score = sum(s["overall"] for s in scores) / len(scores) if scores else 0

        print(f"  Total devices: {len(scores)}")
        print(f"  Average ZTA score: {avg_score:.1f}")
        print(f"  Distribution: {distribution}")

        return {
            "total": len(scores),
            "average_score": round(avg_score, 1),
            "distribution": distribution,
            "below_threshold_50": distribution["blocked_below_50"]
        }

    def get_intune_compliance(self) -> dict[str, Any]:
        """Get Microsoft Intune device compliance status."""
        print("\n[2/4] Querying Intune compliance status...")
        headers = {"Authorization": f"Bearer {self.azure_token}"}

        resp = requests.get(
            "https://graph.microsoft.com/v1.0/deviceManagement/managedDevices",
            headers=headers,
            params={
                "$select": "id,deviceName,userPrincipalName,complianceState,"
                           "operatingSystem,osVersion,isEncrypted,lastSyncDateTime,"
                           "managementAgent",
                "$top": 999
            },
            timeout=60
        )
        resp.raise_for_status()
        devices = resp.json().get("value", [])

        stats = {
            "total": len(devices),
            "compliant": 0,
            "noncompliant": 0,
            "in_grace_period": 0,
            "unknown": 0,
            "os_distribution": {},
            "encryption_status": {"encrypted": 0, "not_encrypted": 0},
            "stale_devices": 0,
            "noncompliant_details": []
        }

        now = datetime.now(timezone.utc)
        for device in devices:
            compliance = device.get("complianceState", "unknown")
            os_name = device.get("operatingSystem", "unknown")
            encrypted = device.get("isEncrypted", False)

            stats["os_distribution"][os_name] = stats["os_distribution"].get(os_name, 0) + 1

            if encrypted:
                stats["encryption_status"]["encrypted"] += 1
            else:
                stats["encryption_status"]["not_encrypted"] += 1

            if compliance == "compliant":
                stats["compliant"] += 1
            elif compliance == "noncompliant":
                stats["noncompliant"] += 1
                stats["noncompliant_details"].append({
                    "device": device.get("deviceName"),
                    "user": device.get("userPrincipalName"),
                    "os": f"{os_name} {device.get('osVersion', '')}",
                    "encrypted": encrypted
                })
            elif compliance == "inGracePeriod":
                stats["in_grace_period"] += 1
            else:
                stats["unknown"] += 1

            # Check for stale devices (no sync in 30 days)
            last_sync = device.get("lastSyncDateTime")
            if last_sync:
                try:
                    sync_dt = datetime.fromisoformat(last_sync.replace("Z", "+00:00"))
                    if (now - sync_dt).days > 30:
                        stats["stale_devices"] += 1
                except (ValueError, TypeError):
                    pass

        compliance_rate = (stats["compliant"] / stats["total"] * 100) if stats["total"] else 0
        print(f"  Total: {stats['total']}, Compliant: {stats['compliant']} ({compliance_rate:.1f}%)")
        print(f"  Non-compliant: {stats['noncompliant']}, Grace period: {stats['in_grace_period']}")
        print(f"  Encryption: {stats['encryption_status']}")
        print(f"  Stale devices (>30d no sync): {stats['stale_devices']}")

        return stats

    def correlate_posture(self, zta: dict, intune: dict) -> dict[str, Any]:
        """Correlate ZTA and MDM compliance for overall posture score."""
        print("\n[3/4] Correlating posture signals...")

        total_devices = max(zta["total"], intune["total"])
        zta_passing = zta["total"] - zta.get("below_threshold_50", 0)
        intune_passing = intune["compliant"] + intune["in_grace_period"]

        overall_compliance = min(
            (zta_passing / zta["total"] * 100) if zta["total"] else 0,
            (intune_passing / intune["total"] * 100) if intune["total"] else 0
        )

        summary = {
            "estimated_total_devices": total_devices,
            "zta_passing_rate": round((zta_passing / zta["total"] * 100) if zta["total"] else 0, 1),
            "intune_passing_rate": round((intune_passing / intune["total"] * 100) if intune["total"] else 0, 1),
            "overall_compliance_rate": round(overall_compliance, 1),
            "risk_level": "LOW" if overall_compliance >= 90 else "MEDIUM" if overall_compliance >= 75 else "HIGH"
        }

        print(f"  ZTA passing: {summary['zta_passing_rate']}%")
        print(f"  Intune passing: {summary['intune_passing_rate']}%")
        print(f"  Overall compliance: {summary['overall_compliance_rate']}%")
        print(f"  Risk level: {summary['risk_level']}")

        return summary

    def generate_report(self, zta: dict, intune: dict, correlated: dict) -> str:
        """Generate consolidated posture report."""
        print("\n[4/4] Generating report...")
        now = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M UTC")

        report = f"""
Device Posture Compliance Report
{'=' * 55}
Generated: {now}

1. CROWDSTRIKE ZTA SCORES
   Total devices assessed:      {zta['total']}
   Average ZTA score:           {zta['average_score']}
   Score >= 90 (Critical OK):   {zta['distribution'].get('critical_90_100', 0)}
   Score 80-89 (High OK):       {zta['distribution'].get('high_80_89', 0)}
   Score 65-79 (Medium OK):     {zta['distribution'].get('medium_65_79', 0)}
   Score 50-64 (Low):           {zta['distribution'].get('low_50_64', 0)}
   Score < 50 (BLOCKED):        {zta['distribution'].get('blocked_below_50', 0)}

2. INTUNE COMPLIANCE
   Total managed devices:       {intune['total']}
   Compliant:                   {intune['compliant']}
   Non-compliant:               {intune['noncompliant']}
   In grace period:             {intune['in_grace_period']}
   Stale (>30d no sync):        {intune['stale_devices']}
   Encrypted:                   {intune['encryption_status']['encrypted']}
   Not encrypted:               {intune['encryption_status']['not_encrypted']}

3. OVERALL POSTURE
   ZTA passing rate:            {correlated['zta_passing_rate']}%
   Intune passing rate:         {correlated['intune_passing_rate']}%
   Combined compliance:         {correlated['overall_compliance_rate']}%
   Risk level:                  {correlated['risk_level']}

4. RECOMMENDATIONS
"""
        recs = []
        if zta["distribution"].get("blocked_below_50", 0) > 0:
            recs.append(f"   - {zta['distribution']['blocked_below_50']} devices below ZTA 50 - investigate immediately")
        if intune["encryption_status"]["not_encrypted"] > 0:
            recs.append(f"   - {intune['encryption_status']['not_encrypted']} devices lack encryption - enforce BitLocker/FileVault")
        if intune["stale_devices"] > 0:
            recs.append(f"   - {intune['stale_devices']} stale devices - verify active use or remove")
        if correlated["overall_compliance_rate"] < 95:
            recs.append(f"   - Overall compliance {correlated['overall_compliance_rate']}% below 95% target")
        if not recs:
            recs.append("   - All devices meet compliance requirements")
        report += "\n".join(recs)
        return report


def main():
    if len(sys.argv) < 6:
        print("Usage: python process.py <cs_client_id> <cs_client_secret> "
              "<azure_tenant_id> <azure_client_id> <azure_client_secret>")
        sys.exit(1)

    auditor = DevicePostureAuditor(*sys.argv[1:6])
    auditor.authenticate_crowdstrike()
    auditor.authenticate_azure()

    zta = auditor.get_zta_scores()
    intune = auditor.get_intune_compliance()
    correlated = auditor.correlate_posture(zta, intune)
    report = auditor.generate_report(zta, intune, correlated)
    print(report)

    filename = f"device_posture_report_{datetime.now().strftime('%Y%m%d')}.txt"
    with open(filename, "w") as f:
        f.write(report)
    print(f"\nReport saved to: {filename}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-identity-verification-for-zero-trust/SKILL.md =====
---
name: implementing-identity-verification-for-zero-trust
description: Implements continuous, risk-adaptive identity verification for zero trust
  using phishing-resistant MFA (FIDO2/WebAuthn), risk-based conditional access, and
  identity governance aligned with NIST SP 800-207 and the CISA Zero Trust Maturity
  Model Identity Pillar. Use when designing zero trust identity controls, deploying
  phishing-resistant MFA, or building conditional access policies based on device
  posture, behavior, and location.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zero-trust
- identity
- authentication
- mfa
- identity-verification
version: '1.0'
author: mahipal
license: Apache-2.0
atlas_techniques:
- AML.T0052
nist_ai_rmf:
- GOVERN-1.1
- GOVERN-1.7
- MAP-1.1
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078
- T1190
- T1059
- T1566
- T1598
mitre_f3:
  version: '1.1'
  tactics:
  - initial-access
  - positioning
  - reconnaissance
  techniques:
  - id: F1006
    name: Account Takeover
    tactic: initial-access
    source: f3
  - id: T1110.004
    name: 'Brute Force:  Credential Stuffing'
    tactic: initial-access
    source: attack
  - id: T1111
    name: Multi-Factor Authentication Interception
    tactic: initial-access
    source: attack
  - id: T1557
    name: Adversary-in-the-Middle
    tactic: initial-access
    source: attack
  - id: T1539
    name: Steal Web Session Cookie
    tactic: positioning
    source: attack
  - id: T1598
    name: Phishing for Information
    tactic: reconnaissance
    source: attack
---

# Implementing Identity Verification for Zero Trust

## Prerequisites

- Understanding of zero trust principles (NIST SP 800-207)
- Familiarity with identity providers (Azure AD, Okta, Ping Identity)
- Knowledge of authentication protocols (SAML 2.0, OIDC, FIDO2)
- Understanding of MFA and passwordless authentication

## Overview

Identity is the foundational pillar of zero trust architecture. NIST SP 800-207 mandates that all resource authentication and authorization are dynamic and strictly enforced before access is allowed. Identity verification in zero trust goes beyond traditional username/password by implementing continuous, risk-adaptive authentication using multiple signals including device posture, behavioral biometrics, location, and network context.

This skill covers implementing phishing-resistant MFA, continuous identity verification, risk-based conditional access, and identity governance aligned with the CISA Zero Trust Maturity Model Identity Pillar.


## When to Use

- When deploying or configuring implementing identity verification for zero trust capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with zero trust architecture concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Architecture

### Identity Verification Flow

```
User Access Request
    │
    v
┌───────────────────────┐
│ Primary Authentication │
│ - FIDO2/WebAuthn key  │
│ - Certificate-based    │
│ - Passwordless         │
└──────────┬────────────┘
           v
┌───────────────────────┐
│ Contextual Assessment  │
│ - Device posture       │
│ - Network location     │
│ - Geo-velocity check   │
│ - Time of access       │
│ - Behavioral baseline  │
└──────────┬────────────┘
           v
┌───────────────────────┐
│ Risk Scoring Engine    │
│ - Aggregate signals    │
│ - Calculate risk score │
│ - Compare to threshold │
└───┬──────────┬────────┘
    │          │
 Low Risk   High Risk
    │          │
    v          v
┌────────┐  ┌──────────────┐
│ Grant  │  │ Step-up Auth  │
│ Access │  │ - Hardware key│
│        │  │ - Biometric   │
│        │  │ - Manager OK  │
└────────┘  └──────────────┘
```

### Identity Provider Architecture

1. **Primary IdP**: Azure AD / Okta / Ping Identity for centralized identity management
2. **FIDO2 Authenticators**: Hardware security keys (YubiKey) or platform authenticators (Windows Hello, Touch ID)
3. **Risk Engine**: Adaptive access using identity threat detection (Microsoft Entra ID Protection, Okta ThreatInsight)
4. **Identity Governance**: Lifecycle management, access reviews, just-in-time provisioning
5. **Privileged Identity**: Separate verification for elevated access (CyberArk, BeyondTrust)

## Key Concepts

### Phishing-Resistant MFA
FIDO2/WebAuthn eliminates phishable credentials by binding authentication to the origin domain. Hardware security keys and platform authenticators provide cryptographic proof of identity without transmitting secrets.

### Continuous Identity Verification
Rather than authenticating once at session start, zero trust requires ongoing verification through session token evaluation, behavioral analytics, and periodic re-authentication challenges based on risk signals.

### Risk-Based Conditional Access
Conditional access policies evaluate multiple signals (user risk level, sign-in risk, device compliance, location) to dynamically adjust authentication requirements and access grants.

### Identity Threat Detection
AI-driven analytics detect compromised identities through impossible travel detection, anomalous sign-in patterns, credential stuffing detection, and token replay attacks.

## Workflow

### Phase 1: Identity Infrastructure

1. **Consolidate Identity Providers**
   - Audit all identity sources across the organization
   - Federate to a single authoritative IdP using SAML 2.0 or OIDC
   - Configure SCIM for automated provisioning and deprovisioning
   - Eliminate local accounts and shared credentials

2. **Deploy Phishing-Resistant MFA**
   - Enroll all users in FIDO2/WebAuthn with hardware security keys
   - Configure platform authenticators (Windows Hello for Business, macOS Touch ID)
   - Disable SMS and voice call as MFA methods (phishable)
   - Create conditional access policy requiring phishing-resistant methods for all sign-ins

3. **Configure Conditional Access Policies**
   - Require compliant device for access to sensitive applications
   - Block legacy authentication protocols (basic auth, IMAP, POP3)
   - Require MFA for all users from untrusted locations
   - Enforce session time limits with re-authentication
   - Block or require additional verification for high-risk sign-ins

### Phase 2: Risk-Based Authentication

4. **Enable Identity Threat Detection**
   - Activate Microsoft Entra ID Protection or Okta ThreatInsight
   - Configure risk levels: low (allow), medium (require MFA), high (block and investigate)
   - Enable impossible travel detection and anomalous token alerts
   - Integrate identity risk signals with SIEM/SOAR

5. **Implement Step-Up Authentication**
   - For sensitive operations (privilege elevation, financial transactions), require additional verification
   - Configure step-up policies: re-authenticate with hardware key
   - Integrate with PAM for privileged session approval workflows
   - Log all step-up events for audit trail

### Phase 3: Continuous Verification

6. **Deploy Continuous Access Evaluation (CAE)**
   - Enable Continuous Access Evaluation Protocol (CAEP) for real-time token revocation
   - Configure critical event triggers: user disabled, password changed, location change
   - Test that token revocation occurs within minutes (not hours) of security event
   - Monitor CAE event logs for operational health

7. **Implement Session Controls**
   - Configure session duration limits based on application sensitivity
   - Enable sign-in frequency controls (re-authenticate every N hours)
   - Implement persistent browser session controls
   - Configure app-enforced restrictions for unmanaged devices

### Phase 4: Identity Governance

8. **Automate Identity Lifecycle**
   - Configure joiner-mover-leaver workflows with HR system integration
   - Automate access provisioning based on role and department
   - Enable just-in-time access for temporary elevated permissions
   - Configure automatic access expiration for contractors and guests

9. **Implement Access Reviews**
   - Schedule quarterly access certification campaigns
   - Configure automated reminders and escalation
   - Require manager approval for continued access
   - Auto-revoke access for unreviewed certifications

## Validation Checklist

- [ ] Single authoritative IdP with all applications federated
- [ ] FIDO2/WebAuthn enrolled for all users
- [ ] SMS and voice MFA methods disabled
- [ ] Legacy authentication protocols blocked
- [ ] Conditional access policies enforced for all applications
- [ ] Identity threat detection active with risk-based policies
- [ ] Continuous Access Evaluation enabled and tested
- [ ] Step-up authentication configured for sensitive operations
- [ ] Identity lifecycle automated with HR integration
- [ ] Quarterly access reviews scheduled and operational
- [ ] Identity events streaming to SIEM

## References

- NIST SP 800-207: Zero Trust Architecture
- NIST SP 800-63B: Digital Identity Guidelines - Authentication
- CISA Zero Trust Maturity Model v2.0 - Identity Pillar
- FIDO Alliance WebAuthn Specification
- Microsoft Entra Conditional Access Documentation

===== END FILE =====


===== FILE: skills/implementing-identity-verification-for-zero-trust/LICENSE =====

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


===== FILE: skills/implementing-identity-verification-for-zero-trust/assets/template.md =====
# Identity Verification Implementation Plan Template

## Project Information

| Field | Value |
|---|---|
| Project Name | |
| Organization | |
| Identity Provider | [Azure AD / Okta / Ping Identity] |
| User Population | |
| Target Completion | |

## Current State Assessment

### Identity Infrastructure
- **Primary IdP**: |
- **Secondary IdPs**: |
- **Local Accounts**: [Count and locations] |
- **Shared Accounts**: [Count - target for elimination] |

### Current MFA State
| Method | Enabled | Users Enrolled | Phishing-Resistant |
|---|---|---|---|
| SMS OTP | | | No |
| Voice Call | | | No |
| TOTP App | | | No |
| Push Notification | | | No |
| FIDO2 Security Key | | | Yes |
| Windows Hello | | | Yes |
| Certificate/PIV | | | Yes |

### MFA Enrollment Target
- Current enrollment rate: ___%
- Target enrollment rate: 100%
- Phishing-resistant target: 100%

## Phishing-Resistant MFA Rollout Plan

### Hardware Key Distribution

| User Group | Key Type | Quantity | Distribution Method | Timeline |
|---|---|---|---|---|
| Executive Leadership | YubiKey 5 NFC | | In-person | Week 1 |
| IT Administrators | YubiKey 5C | | IT distribution | Week 2 |
| All Employees | YubiKey Security Key | | Self-service + mail | Weeks 3-8 |

### Enrollment Campaign
- [ ] Communication sent to all users
- [ ] Self-service portal configured
- [ ] Help desk trained on enrollment support
- [ ] Enrollment deadline set: ____
- [ ] Escalation path for non-compliant users

## Conditional Access Policies

| Policy Name | Users | Apps | Conditions | Grant Controls | Session Controls |
|---|---|---|---|---|---|
| Block Legacy Auth | All | All | Legacy clients | Block | N/A |
| Require MFA | All | All | Any | Require MFA | Sign-in freq: 8hr |
| Require Compliant Device | All | Sensitive Apps | Any | Compliant device | App enforced |
| Block Risky Sign-In | All | All | High sign-in risk | Block | N/A |
| Require FIDO2 for Admins | Admin roles | Admin portals | Any | FIDO2 only | 1hr frequency |

## Risk-Based Policies

| Risk Level | User Risk Response | Sign-In Risk Response |
|---|---|---|
| Low | Allow | Allow |
| Medium | Require MFA step-up | Require MFA step-up |
| High | Block + alert SOC | Block + alert SOC |

## Identity Governance

### Lifecycle Automation
- [ ] HR system integrated for joiner/mover/leaver
- [ ] Automatic provisioning on hire
- [ ] Role change triggers access review
- [ ] Automatic deprovisioning on termination
- [ ] Contractor access expiration configured

### Access Reviews
- Frequency: Quarterly
- Scope: All application assignments
- Reviewers: Direct managers
- Auto-action on non-response: Revoke access

## Monitoring and Detection

| Capability | Tool | Status |
|---|---|---|
| Sign-in log analysis | SIEM (Splunk/Sentinel) | |
| Identity threat detection | Entra ID Protection / ThreatInsight | |
| Impossible travel detection | IdP + UEBA | |
| Continuous Access Evaluation | CAE/CAEP | |
| Behavioral analytics | UEBA platform | |

## Sign-Off

| Stakeholder | Role | Approval | Date |
|---|---|---|---|
| | CISO | | |
| | Identity Team Lead | | |
| | Help Desk Manager | | |
| | HR Systems | | |

===== END FILE =====


===== FILE: skills/implementing-identity-verification-for-zero-trust/references/api-reference.md =====
# API Reference: Implementing Identity Verification for Zero Trust

## CISA Zero Trust Maturity Model - Identity Pillar

| Level | Description | Requirements |
|-------|-------------|-------------|
| Traditional | Password-based, static policies | Basic auth |
| Initial | MFA deployed, basic conditional access | MFA for all users |
| Advanced | Phishing-resistant MFA, risk-based | FIDO2, risk signals |
| Optimal | Continuous verification, passwordless | Behavioral analytics |

## Azure AD Conditional Access API

```python
import requests
headers = {"Authorization": "Bearer <token>"}
policies = requests.get(
    "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies",
    headers=headers).json()
```

## FIDO2/WebAuthn Registration

```javascript
const credential = await navigator.credentials.create({
    publicKey: {
        rp: { name: "Example Corp" },
        user: { id: userId, name: email, displayName: name },
        challenge: serverChallenge,
        pubKeyCredParams: [{ type: "public-key", alg: -7 }],
        authenticatorSelection: { residentKey: "required" },
    }
});
```

## Conditional Access Signals

| Signal | Source | Zero Trust Level |
|--------|--------|-----------------|
| Device compliance | MDM/Intune | Initial |
| Location/IP | Network context | Initial |
| User risk | Identity Protection | Advanced |
| Sign-in risk | Real-time analysis | Advanced |
| Session behavior | UEBA | Optimal |

## Okta Authentication Policies API

```bash
curl -X GET "https://DOMAIN.okta.com/api/v1/policies?type=ACCESS_POLICY" \
  -H "Authorization: SSWS <token>"
```

### References

- CISA Zero Trust Maturity Model: https://www.cisa.gov/zero-trust-maturity-model
- NIST SP 800-207: https://csrc.nist.gov/pubs/sp/800/207/final
- FIDO Alliance: https://fidoalliance.org/fido2/

===== END FILE =====


===== FILE: skills/implementing-identity-verification-for-zero-trust/references/standards.md =====
# Standards and Frameworks Reference

## NIST SP 800-207: Zero Trust Architecture - Identity

### Core Identity Tenets
- All resource authentication and authorization are dynamic and strictly enforced before access is allowed
- The enterprise collects as much information as possible about the current state of assets, network infrastructure, and communications and uses it to improve its security posture
- Access decisions should consider behavioral attributes, environmental attributes, and the identity of the requester

### Policy Engine Identity Inputs
The Policy Engine (PE) uses the following identity-related inputs:
- User identity and attributes from the IdP
- Device identity and posture from the endpoint management system
- Behavioral attributes from user and entity behavior analytics (UEBA)
- Environmental attributes (location, time, network)

## NIST SP 800-63B: Digital Identity Guidelines

### Authentication Assurance Levels (AAL)
| AAL | Description | Methods | Zero Trust Mapping |
|---|---|---|---|
| AAL1 | Some assurance | Single-factor (password) | Insufficient for ZT |
| AAL2 | High confidence | Multi-factor (push notification, OTP) | Minimum for ZT |
| AAL3 | Very high confidence | Hardware-based (FIDO2, PIV card) | Target for ZT |

### Phishing-Resistant Authenticators
- FIDO2/WebAuthn: Cryptographic authentication bound to origin domain
- PIV/CAC smart cards: Certificate-based authentication
- Not phishing-resistant: SMS OTP, voice calls, push notifications, TOTP

## CISA Zero Trust Maturity Model v2.0 - Identity Pillar

| Maturity Level | Authentication | Identity Store | Risk Assessment | Visibility |
|---|---|---|---|---|
| Traditional | Password + basic MFA | Multiple disconnected stores | None | Basic audit logs |
| Initial | MFA for all users | Federated IdP | Static risk rules | Centralized auth logs |
| Advanced | Phishing-resistant MFA | Single authoritative IdP with SCIM | Risk-based conditional access | Identity analytics |
| Optimal | Continuous verification | Automated lifecycle governance | AI-driven threat detection | Real-time UEBA |

## FIDO Alliance Standards

### FIDO2 / WebAuthn
- W3C Web Authentication specification for passwordless authentication
- Public-key cryptography: private key never leaves the authenticator
- Origin-bound: authentication is cryptographically tied to the service domain
- Resistant to phishing, replay, and man-in-the-middle attacks

### Passkeys
- Evolution of FIDO2 for consumer and enterprise use
- Synced across devices via platform credential managers (iCloud Keychain, Google Password Manager)
- Discoverable credentials eliminate need to remember usernames

## Microsoft Entra (Azure AD) Identity Protection

### Risk Detection Categories
| Risk Type | Detection | Response |
|---|---|---|
| Anonymous IP | Sign-in from anonymous proxy/VPN | Require MFA |
| Atypical travel | Impossible travel between sign-in locations | Block + investigate |
| Malware-linked IP | Sign-in from known malicious IP | Block |
| Unfamiliar sign-in | Unusual sign-in properties | Step-up auth |
| Leaked credentials | Credentials found in dark web dumps | Force password reset |
| Token anomaly | Unusual token characteristics | Revoke session |

### Continuous Access Evaluation Protocol (CAEP)
- Real-time token revocation on security events
- Critical events: user disabled, password changed, high risk detected
- Reduces token lifetime gap from hours to near real-time
- Supported by Microsoft 365, Exchange Online, SharePoint Online

## Okta Identity Security

### Okta ThreatInsight
- Pre-authentication threat detection using IP reputation
- Credential stuffing protection
- Bot detection and rate limiting
- Anomalous location and device detection

### Okta FastPass
- Passwordless, phishing-resistant authentication
- Device-bound biometric verification
- Continuous device trust assessment
- No shared secrets transmitted over network

===== END FILE =====


===== FILE: skills/implementing-identity-verification-for-zero-trust/references/workflows.md =====
# Identity Verification Workflows

## Workflow 1: Zero Trust Authentication Flow

```
User Initiates Access
    │
    v
┌─────────────────────────┐
│ 1. Pre-Authentication    │
│ - Check IP reputation    │
│ - Rate limit evaluation  │
│ - Bot detection          │
│ - Geo-blocking check     │
└──────────┬──────────────┘
           v
┌─────────────────────────┐
│ 2. Primary Authentication│
│ - FIDO2 key challenge    │
│ - Biometric verification │
│ - Certificate validation │
│ - Passwordless flow      │
└──────────┬──────────────┘
           v
┌─────────────────────────┐
│ 3. Context Assessment    │
│ - Device compliance      │
│ - Network location       │
│ - Time of access         │
│ - Behavioral baseline    │
│ - Previous session state │
└──────────┬──────────────┘
           v
┌─────────────────────────┐
│ 4. Risk Calculation      │
│ - User risk level        │
│ - Sign-in risk level     │
│ - Aggregate score        │
└───┬──────────┬──────┬───┘
    │          │      │
  LOW        MED    HIGH
    │          │      │
    v          v      v
┌──────┐ ┌────────┐ ┌────────┐
│Grant │ │Step-Up │ │Block + │
│Token │ │ Auth   │ │Alert   │
└──────┘ └────────┘ └────────┘
```

## Workflow 2: Continuous Access Evaluation

```
Active Session
    │
    v
┌──────────────────────────────┐
│ Continuous Monitoring Loop    │
│                               │
│  ┌─── Check every N minutes ──┐
│  │                             │
│  │  ┌─────────────────────┐   │
│  │  │ Signal Collection    │   │
│  │  │ - Device compliance  │   │
│  │  │ - User risk change   │   │
│  │  │ - Location shift     │   │
│  │  │ - Behavior anomaly   │   │
│  │  └──────────┬──────────┘   │
│  │             v               │
│  │  ┌─────────────────────┐   │
│  │  │ Critical Events      │   │
│  │  │ - Account disabled   │   │
│  │  │ - Password changed   │   │
│  │  │ - MFA registration   │   │
│  │  │ - Admin revocation   │   │
│  │  └──────────┬──────────┘   │
│  │             v               │
│  │  ┌─────────────────────┐   │
│  │  │ Re-Evaluate Access   │   │
│  │  │ - Recalculate risk   │   │
│  │  │ - Apply policy       │   │
│  │  └───┬─────────┬───────┘   │
│  │      │         │           │
│  │   Continue   Revoke        │
│  │   Session    Token         │
│  └──────┘         │           │
│                   v           │
│           ┌──────────────┐    │
│           │ Force Re-Auth│    │
│           │ or Terminate │    │
│           └──────────────┘    │
└──────────────────────────────┘
```

## Workflow 3: FIDO2 Enrollment

```
Admin Initiates Enrollment Campaign
    │
    v
┌──────────────────────────┐
│ 1. User Notification      │
│ - Email with instructions │
│ - Self-service portal URL │
│ - Deadline for enrollment │
└──────────┬───────────────┘
           v
┌──────────────────────────┐
│ 2. User Self-Service      │
│ - Authenticate with       │
│   existing credentials    │
│ - Register security key   │
│   (YubiKey, Titan key)    │
│ - Register platform auth  │
│   (Windows Hello, TouchID)│
│ - Register backup method  │
└──────────┬───────────────┘
           v
┌──────────────────────────┐
│ 3. Verification           │
│ - Test sign-in with FIDO2 │
│ - Confirm backup works    │
│ - Record key serial/ID    │
└──────────┬───────────────┘
           v
┌──────────────────────────┐
│ 4. Policy Enforcement     │
│ - Enable phishing-resist  │
│   conditional access      │
│ - Disable legacy MFA      │
│ - Monitor compliance rate │
└──────────────────────────┘
```

## Workflow 4: Compromised Identity Response

```
Identity Threat Detected
    │
    v
┌──────────────────────────┐
│ 1. Detection Signal       │
│ - Impossible travel       │
│ - Leaked credentials      │
│ - Token anomaly           │
│ - Behavioral deviation    │
└──────────┬───────────────┘
           v
┌──────────────────────────┐
│ 2. Automated Response     │
│ - Revoke all sessions     │
│ - Disable account         │
│ - Trigger SOAR playbook   │
│ - Notify SOC analyst      │
└──────────┬───────────────┘
           v
┌──────────────────────────┐
│ 3. Investigation          │
│ - Review sign-in logs     │
│ - Check accessed resources│
│ - Correlate with EDR data │
│ - Interview user          │
└──────────┬───────────────┘
           v
┌──────────────────────────┐
│ 4. Remediation            │
│ - Reset all credentials   │
│ - Re-enroll FIDO2 keys    │
│ - Review and restrict     │
│   access permissions      │
│ - Re-enable account       │
│ - Update detection rules  │
└──────────────────────────┘
```

===== END FILE =====


===== FILE: skills/implementing-identity-verification-for-zero-trust/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for assessing identity verification controls in zero trust architecture."""

import json
import argparse
from datetime import datetime
from collections import Counter


CISA_ZT_IDENTITY_LEVELS = {
    "traditional": {
        "description": "Password-based auth, static policies",
        "score": 1,
    },
    "initial": {
        "description": "MFA deployed, basic conditional access",
        "score": 2,
    },
    "advanced": {
        "description": "Phishing-resistant MFA, risk-based access",
        "score": 3,
    },
    "optimal": {
        "description": "Continuous verification, passwordless, behavioral analytics",
        "score": 4,
    },
}


def assess_authentication_methods(auth_config):
    """Assess authentication method strength against zero trust requirements."""
    findings = []
    methods = auth_config.get("methods", auth_config.get("authentication", {}))

    if isinstance(methods, dict):
        mfa_enabled = methods.get("mfa_enabled", False)
        phishing_resistant = methods.get("phishing_resistant_mfa", False)
        passwordless = methods.get("passwordless", False)
        fido2 = methods.get("fido2_enabled", False)
        sso = methods.get("sso_enabled", False)

        if not mfa_enabled:
            findings.append({"control": "MFA", "status": "MISSING",
                             "severity": "CRITICAL"})
        elif not phishing_resistant:
            findings.append({"control": "Phishing-resistant MFA", "status": "MISSING",
                             "severity": "HIGH",
                             "recommendation": "Deploy FIDO2 or certificate-based auth"})

        if not fido2:
            findings.append({"control": "FIDO2/WebAuthn", "status": "NOT_DEPLOYED",
                             "severity": "MEDIUM"})
        if not passwordless:
            findings.append({"control": "Passwordless authentication",
                             "status": "NOT_DEPLOYED", "severity": "MEDIUM"})
        if not sso:
            findings.append({"control": "SSO", "status": "NOT_DEPLOYED",
                             "severity": "HIGH"})
    return findings


def assess_conditional_access(policies_path):
    """Assess conditional access policies for zero trust alignment."""
    with open(policies_path) as f:
        policies = json.load(f)
    items = policies if isinstance(policies, list) else policies.get("policies", [])
    findings = []
    required_signals = ["device_compliance", "location", "risk_level",
                        "application", "user_group"]
    covered_signals = set()

    for policy in items:
        conditions = policy.get("conditions", {})
        for signal in required_signals:
            if conditions.get(signal) or signal in str(conditions):
                covered_signals.add(signal)
        if policy.get("grant_controls", {}).get("operator") == "OR":
            findings.append({
                "policy": policy.get("name", ""),
                "issue": "Grant controls use OR (should be AND)",
                "severity": "HIGH",
            })
        if not policy.get("state", "").lower() in ("enabled", "on"):
            findings.append({
                "policy": policy.get("name", ""),
                "issue": "Policy not enabled",
                "severity": "MEDIUM",
            })

    missing = set(required_signals) - covered_signals
    for signal in missing:
        findings.append({
            "control": f"Conditional access signal: {signal}",
            "status": "NOT_COVERED",
            "severity": "HIGH",
        })
    return {"findings": findings, "covered_signals": list(covered_signals),
            "missing_signals": list(missing)}


def assess_identity_maturity(config):
    """Assess identity pillar maturity against CISA Zero Trust Maturity Model."""
    scores = {}
    categories = {
        "authentication": {
            "checks": ["mfa_enforced", "phishing_resistant_mfa", "passwordless",
                        "continuous_auth"],
        },
        "identity_stores": {
            "checks": ["centralized_idp", "cloud_identity", "directory_sync",
                        "identity_federation"],
        },
        "risk_assessment": {
            "checks": ["risk_based_access", "behavioral_analytics",
                        "impossible_travel_detection", "session_risk_scoring"],
        },
        "visibility": {
            "checks": ["identity_audit_logging", "real_time_monitoring",
                        "identity_analytics_dashboard", "automated_anomaly_detection"],
        },
    }

    for category, info in categories.items():
        implemented = sum(1 for check in info["checks"]
                          if config.get(category, {}).get(check, False))
        total = len(info["checks"])
        ratio = implemented / total if total else 0
        if ratio >= 0.9:
            level = "optimal"
        elif ratio >= 0.6:
            level = "advanced"
        elif ratio >= 0.3:
            level = "initial"
        else:
            level = "traditional"
        scores[category] = {
            "implemented": implemented,
            "total": total,
            "ratio": round(ratio, 2),
            "maturity_level": level,
            "score": CISA_ZT_IDENTITY_LEVELS[level]["score"],
        }

    avg_score = sum(s["score"] for s in scores.values()) / len(scores) if scores else 0
    for level_name, level_info in CISA_ZT_IDENTITY_LEVELS.items():
        if level_info["score"] >= avg_score:
            overall_level = level_name
            break
    else:
        overall_level = "traditional"

    return {"categories": scores, "overall_score": round(avg_score, 1),
            "overall_level": overall_level}


def analyze_auth_events(events_path):
    """Analyze authentication events for zero trust insights."""
    with open(events_path) as f:
        events = json.load(f)
    items = events if isinstance(events, list) else events.get("events", [])

    by_method = Counter(e.get("auth_method", "unknown") for e in items)
    by_result = Counter(e.get("result", "unknown") for e in items)
    risky = [e for e in items if e.get("risk_level", "").lower() in ("high", "critical")]
    mfa_bypassed = [e for e in items if e.get("mfa_bypassed", False)]

    return {
        "total_events": len(items),
        "by_method": dict(by_method),
        "by_result": dict(by_result),
        "high_risk_events": len(risky),
        "mfa_bypass_events": len(mfa_bypassed),
        "password_only_rate": round(
            by_method.get("password", 0) / len(items) * 100, 1) if items else 0,
    }


def main():
    parser = argparse.ArgumentParser(description="Zero Trust Identity Verification Agent")
    parser.add_argument("--auth-config", help="Authentication config JSON")
    parser.add_argument("--policies", help="Conditional access policies JSON")
    parser.add_argument("--maturity-config", help="Identity maturity assessment config JSON")
    parser.add_argument("--auth-events", help="Authentication events log JSON")
    parser.add_argument("--action", choices=["auth", "policies", "maturity", "events", "full"],
                        default="full")
    parser.add_argument("--output", default="zt_identity_report.json")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "results": {}}

    if args.action in ("auth", "full") and args.auth_config:
        with open(args.auth_config) as f:
            config = json.load(f)
        findings = assess_authentication_methods(config)
        report["results"]["auth_assessment"] = findings
        critical = sum(1 for f in findings if f.get("severity") == "CRITICAL")
        print(f"[+] Auth findings: {len(findings)}, {critical} critical")

    if args.action in ("policies", "full") and args.policies:
        result = assess_conditional_access(args.policies)
        report["results"]["conditional_access"] = result
        print(f"[+] Missing signals: {result['missing_signals']}")

    if args.action in ("maturity", "full") and args.maturity_config:
        with open(args.maturity_config) as f:
            config = json.load(f)
        result = assess_identity_maturity(config)
        report["results"]["maturity"] = result
        print(f"[+] Identity maturity: {result['overall_level']} (score: {result['overall_score']})")

    if args.action in ("events", "full") and args.auth_events:
        result = analyze_auth_events(args.auth_events)
        report["results"]["events"] = result
        print(f"[+] Auth events: {result['total_events']}, password-only: {result['password_only_rate']}%")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-identity-verification-for-zero-trust/scripts/process.py =====
#!/usr/bin/env python3
"""
Identity Verification Assessment Tool for Zero Trust

Analyzes identity configurations, evaluates MFA strength, assesses
conditional access policies, and generates identity maturity reports.
"""

import json
import csv
import sys
from datetime import datetime, timedelta
from pathlib import Path
from collections import defaultdict
from typing import Optional


def assess_mfa_strength(mfa_config: dict) -> dict:
    """Evaluate MFA configuration against zero trust requirements."""
    findings = []
    score = 100

    methods = mfa_config.get("enabled_methods", [])
    phishing_resistant = {"fido2", "webauthn", "certificate", "windows_hello", "platform_authenticator"}
    phishable = {"sms", "voice", "email_otp"}
    moderate = {"totp", "push_notification", "authenticator_app"}

    enabled_phishing_resistant = set(methods) & phishing_resistant
    enabled_phishable = set(methods) & phishable

    if not enabled_phishing_resistant:
        findings.append({
            "severity": "critical",
            "finding": "No phishing-resistant MFA methods enabled",
            "recommendation": "Deploy FIDO2 security keys or platform authenticators",
            "reference": "NIST SP 800-63B AAL3, CISA ZT Identity Pillar (Advanced)"
        })
        score -= 40

    if enabled_phishable:
        findings.append({
            "severity": "high",
            "finding": f"Phishable MFA methods still enabled: {', '.join(enabled_phishable)}",
            "recommendation": "Disable SMS, voice, and email OTP methods",
            "reference": "CISA Phishing-Resistant MFA Guidance"
        })
        score -= 20

    if not mfa_config.get("enforced_for_all_users"):
        findings.append({
            "severity": "critical",
            "finding": "MFA not enforced for all users",
            "recommendation": "Enable MFA requirement for all user accounts",
            "reference": "CISA ZT Identity Pillar (Initial)"
        })
        score -= 30

    if not mfa_config.get("number_matching_enabled") and "push_notification" in methods:
        findings.append({
            "severity": "high",
            "finding": "Push notification MFA without number matching",
            "recommendation": "Enable number matching to prevent MFA fatigue attacks",
            "reference": "Microsoft MFA fatigue defense guidance"
        })
        score -= 10

    enrollment_rate = mfa_config.get("enrollment_rate_percent", 0)
    if enrollment_rate < 95:
        findings.append({
            "severity": "warning",
            "finding": f"MFA enrollment rate is {enrollment_rate}% (target: 95%+)",
            "recommendation": "Launch enrollment campaign for remaining users",
            "reference": "CISA ZT Identity Pillar (Advanced)"
        })
        score -= 10

    return {
        "mfa_score": max(score, 0),
        "phishing_resistant_methods": list(enabled_phishing_resistant),
        "phishable_methods": list(enabled_phishable),
        "findings": findings,
        "maturity_level": (
            "optimal" if score >= 90 else
            "advanced" if score >= 70 else
            "initial" if score >= 50 else
            "traditional"
        )
    }


def assess_conditional_access(policies: list) -> dict:
    """Evaluate conditional access policies for zero trust alignment."""
    findings = []
    coverage = {
        "legacy_auth_blocked": False,
        "mfa_required_all": False,
        "device_compliance_required": False,
        "location_restrictions": False,
        "session_controls": False,
        "risk_based_policies": False,
        "privileged_access_secured": False,
    }

    for policy in policies:
        conditions = policy.get("conditions", {})
        grant_controls = policy.get("grant_controls", {})
        session_controls = policy.get("session_controls", {})

        if policy.get("blocks_legacy_auth"):
            coverage["legacy_auth_blocked"] = True

        if grant_controls.get("require_mfa") and conditions.get("users") == "all":
            coverage["mfa_required_all"] = True

        if grant_controls.get("require_compliant_device"):
            coverage["device_compliance_required"] = True

        if conditions.get("locations") and conditions["locations"].get("excluded_trusted"):
            coverage["location_restrictions"] = True

        if session_controls.get("sign_in_frequency") or session_controls.get("persistent_browser"):
            coverage["session_controls"] = True

        if conditions.get("user_risk") or conditions.get("sign_in_risk"):
            coverage["risk_based_policies"] = True

        if conditions.get("roles") and "privileged" in str(conditions.get("roles", [])).lower():
            coverage["privileged_access_secured"] = True

    if not coverage["legacy_auth_blocked"]:
        findings.append({
            "severity": "critical",
            "finding": "Legacy authentication protocols not blocked",
            "recommendation": "Create policy blocking basic auth, IMAP, POP3, SMTP AUTH",
        })

    if not coverage["mfa_required_all"]:
        findings.append({
            "severity": "critical",
            "finding": "MFA not required for all users",
            "recommendation": "Create policy requiring MFA for all user sign-ins",
        })

    if not coverage["device_compliance_required"]:
        findings.append({
            "severity": "high",
            "finding": "Device compliance not required for access",
            "recommendation": "Require managed and compliant devices for sensitive apps",
        })

    if not coverage["risk_based_policies"]:
        findings.append({
            "severity": "high",
            "finding": "No risk-based conditional access policies",
            "recommendation": "Enable user risk and sign-in risk based policies",
        })

    if not coverage["session_controls"]:
        findings.append({
            "severity": "warning",
            "finding": "No session lifetime controls configured",
            "recommendation": "Configure sign-in frequency and browser session persistence",
        })

    covered = sum(1 for v in coverage.values() if v)
    total = len(coverage)

    return {
        "coverage": coverage,
        "coverage_score": round((covered / total) * 100),
        "policies_evaluated": len(policies),
        "findings": findings,
    }


def analyze_sign_in_logs(logs: list) -> dict:
    """Analyze sign-in logs for identity security insights."""
    stats = {
        "total_sign_ins": len(logs),
        "successful": 0,
        "failed": 0,
        "mfa_challenged": 0,
        "risk_events": [],
        "locations": defaultdict(int),
        "devices": defaultdict(int),
        "applications": defaultdict(int),
        "auth_methods": defaultdict(int),
        "risky_sign_ins": [],
    }

    for log in logs:
        if log.get("status") == "success":
            stats["successful"] += 1
        else:
            stats["failed"] += 1

        if log.get("mfa_required"):
            stats["mfa_challenged"] += 1

        location = log.get("location", {}).get("country", "unknown")
        stats["locations"][location] += 1

        device = log.get("device", {}).get("os", "unknown")
        stats["devices"][device] += 1

        app = log.get("application", "unknown")
        stats["applications"][app] += 1

        method = log.get("auth_method", "password")
        stats["auth_methods"][method] += 1

        risk_level = log.get("risk_level", "none")
        if risk_level in ("medium", "high"):
            stats["risky_sign_ins"].append({
                "user": log.get("user", "unknown"),
                "risk_level": risk_level,
                "risk_detail": log.get("risk_detail", ""),
                "location": location,
                "timestamp": log.get("timestamp", ""),
                "application": app,
            })

    stats["locations"] = dict(stats["locations"])
    stats["devices"] = dict(stats["devices"])
    stats["applications"] = dict(stats["applications"])
    stats["auth_methods"] = dict(stats["auth_methods"])
    stats["mfa_coverage_percent"] = round(
        (stats["mfa_challenged"] / max(stats["total_sign_ins"], 1)) * 100, 1
    )

    return stats


def detect_impossible_travel(sign_ins: list, max_speed_kmh: int = 900) -> list:
    """Detect impossible travel scenarios from sign-in logs."""
    alerts = []
    user_logins = defaultdict(list)

    for si in sign_ins:
        user = si.get("user", "")
        if user and si.get("location", {}).get("latitude"):
            user_logins[user].append(si)

    for user, logins in user_logins.items():
        sorted_logins = sorted(logins, key=lambda x: x.get("timestamp", ""))

        for i in range(1, len(sorted_logins)):
            prev = sorted_logins[i - 1]
            curr = sorted_logins[i]

            lat1 = prev["location"].get("latitude", 0)
            lon1 = prev["location"].get("longitude", 0)
            lat2 = curr["location"].get("latitude", 0)
            lon2 = curr["location"].get("longitude", 0)

            import math
            R = 6371
            dlat = math.radians(lat2 - lat1)
            dlon = math.radians(lon2 - lon1)
            a = (math.sin(dlat / 2) ** 2 +
                 math.cos(math.radians(lat1)) * math.cos(math.radians(lat2)) *
                 math.sin(dlon / 2) ** 2)
            c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
            distance_km = R * c

            try:
                t1 = datetime.fromisoformat(prev.get("timestamp", "").replace("Z", "+00:00"))
                t2 = datetime.fromisoformat(curr.get("timestamp", "").replace("Z", "+00:00"))
                time_diff_hours = (t2 - t1).total_seconds() / 3600
            except (ValueError, TypeError):
                continue

            if time_diff_hours > 0:
                required_speed = distance_km / time_diff_hours
                if required_speed > max_speed_kmh and distance_km > 100:
                    alerts.append({
                        "user": user,
                        "severity": "high",
                        "type": "impossible_travel",
                        "from_location": prev["location"],
                        "to_location": curr["location"],
                        "distance_km": round(distance_km),
                        "time_diff_hours": round(time_diff_hours, 2),
                        "required_speed_kmh": round(required_speed),
                        "from_time": prev.get("timestamp"),
                        "to_time": curr.get("timestamp"),
                    })

    return alerts


def generate_identity_maturity_report(config: dict) -> dict:
    """Generate a comprehensive identity maturity assessment."""
    report = {
        "generated": datetime.now().isoformat(),
        "overall_maturity": "traditional",
        "pillars": {},
    }

    mfa_assessment = assess_mfa_strength(config.get("mfa", {}))
    report["pillars"]["mfa"] = mfa_assessment

    ca_assessment = assess_conditional_access(config.get("conditional_access_policies", []))
    report["pillars"]["conditional_access"] = ca_assessment

    governance = config.get("governance", {})
    gov_score = 0
    if governance.get("automated_provisioning"): gov_score += 25
    if governance.get("access_reviews_enabled"): gov_score += 25
    if governance.get("jit_access_enabled"): gov_score += 25
    if governance.get("lifecycle_automation"): gov_score += 25
    report["pillars"]["governance"] = {"score": gov_score}

    monitoring = config.get("monitoring", {})
    mon_score = 0
    if monitoring.get("siem_integration"): mon_score += 25
    if monitoring.get("ueba_enabled"): mon_score += 25
    if monitoring.get("identity_threat_detection"): mon_score += 25
    if monitoring.get("cae_enabled"): mon_score += 25
    report["pillars"]["monitoring"] = {"score": mon_score}

    avg_score = (
        mfa_assessment["mfa_score"] +
        ca_assessment["coverage_score"] +
        gov_score +
        mon_score
    ) / 4

    report["overall_score"] = round(avg_score)
    report["overall_maturity"] = (
        "optimal" if avg_score >= 90 else
        "advanced" if avg_score >= 70 else
        "initial" if avg_score >= 50 else
        "traditional"
    )

    return report


def main():
    import argparse
    parser = argparse.ArgumentParser(description="Identity Verification Assessment Tool")
    parser.add_argument("--config", type=str, help="Path to identity configuration JSON")
    parser.add_argument("--logs", type=str, help="Path to sign-in logs JSON")
    parser.add_argument("--action", choices=["assess", "analyze-logs", "detect-travel", "report"],
                        default="report")
    parser.add_argument("--output", type=str, default="identity_report.json")
    args = parser.parse_args()

    if args.action == "assess" and args.config:
        with open(args.config) as f:
            config = json.load(f)
        mfa_result = assess_mfa_strength(config.get("mfa", {}))
        ca_result = assess_conditional_access(config.get("conditional_access_policies", []))
        result = {"mfa": mfa_result, "conditional_access": ca_result}
        with open(args.output, "w") as f:
            json.dump(result, f, indent=2)
        print(f"MFA Score: {mfa_result['mfa_score']}, CA Coverage: {ca_result['coverage_score']}%")

    elif args.action == "analyze-logs" and args.logs:
        with open(args.logs) as f:
            logs = json.load(f)
        stats = analyze_sign_in_logs(logs)
        with open(args.output, "w") as f:
            json.dump(stats, f, indent=2)
        print(f"Analyzed {stats['total_sign_ins']} sign-ins, {len(stats['risky_sign_ins'])} risky")

    elif args.action == "detect-travel" and args.logs:
        with open(args.logs) as f:
            logs = json.load(f)
        alerts = detect_impossible_travel(logs)
        with open(args.output, "w") as f:
            json.dump(alerts, f, indent=2)
        print(f"Detected {len(alerts)} impossible travel events")

    elif args.action == "report" and args.config:
        with open(args.config) as f:
            config = json.load(f)
        report = generate_identity_maturity_report(config)
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"Identity Maturity: {report['overall_maturity']} (Score: {report['overall_score']})")

    else:
        parser.print_help()

    if args.output and Path(args.output).exists():
        print(f"Output saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-microsegmentation-with-guardicore/SKILL.md =====
---
name: implementing-microsegmentation-with-guardicore
description: 'Implements microsegmentation with Akamai Guardicore Segmentation to map
  application dependencies, visualize east-west traffic flows, and create granular,
  least-privilege network policies across VMs, containers, bare metal, and cloud.
  Use when blocking lateral movement in a data center or when PCI DSS/HIPAA
  compliance requires validated network segmentation.

  '
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- microsegmentation
- guardicore
- akamai
- zero-trust
- east-west-traffic
- network-segmentation
- lateral-movement
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078
- T1190
- T1059
- T1021
- T1550
---

# Implementing Microsegmentation with Guardicore

## When to Use

- When implementing east-west traffic controls to prevent lateral movement within data centers
- When needing application-level visibility into network communication patterns before writing segmentation policies
- When segmenting workloads across heterogeneous environments (VMs, containers, bare metal, cloud)
- When compliance frameworks (PCI DSS, HIPAA) require network segmentation validation
- When deploying zero trust at the network layer with process-level granularity

**Do not use** for perimeter-only security (use traditional firewalls), for environments with fewer than 50 workloads where VLANs/security groups suffice, or when network team lacks capacity for ongoing policy management.

## Prerequisites

- Akamai Guardicore Segmentation license (Enterprise or Premium)
- Guardicore Management Server deployed (on-prem or SaaS)
- Agent deployment access to target workloads (Linux, Windows, Kubernetes)
- Network visibility: SPAN/TAP ports or VPC flow logs for agentless collection
- Application owner engagement for dependency validation

## Workflow

### Step 1: Deploy Guardicore Agents on Workloads

Install agents to collect process-level network communication data.

```bash
# Linux agent installation
curl -sSL https://management.guardicore.com/api/v3.0/agents/download/linux \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -o gc-agent-installer.sh
chmod +x gc-agent-installer.sh
sudo ./gc-agent-installer.sh \
  --management-url=https://management.guardicore.com \
  --site-id=datacenter-east \
  --label="web-tier"

# Windows agent installation (PowerShell)
# Invoke-WebRequest -Uri "https://management.guardicore.com/api/v3.0/agents/download/windows" `
#   -Headers @{"Authorization"="Bearer $GC_API_TOKEN"} `
#   -OutFile gc-agent-installer.exe
# Start-Process -FilePath .\gc-agent-installer.exe `
#   -ArgumentList "--management-url=https://management.guardicore.com","--site-id=datacenter-east" `
#   -Wait

# Kubernetes DaemonSet deployment
cat > gc-daemonset.yaml << 'EOF'
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: guardicore-agent
  namespace: guardicore
spec:
  selector:
    matchLabels:
      app: gc-agent
  template:
    metadata:
      labels:
        app: gc-agent
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: gc-agent
        image: guardicore/agent:latest
        securityContext:
          privileged: true
        env:
        - name: GC_MANAGEMENT_URL
          value: "https://management.guardicore.com"
        - name: GC_API_KEY
          valueFrom:
            secretKeyRef:
              name: gc-credentials
              key: api-key
        volumeMounts:
        - mountPath: /host
          name: host-root
      volumes:
      - name: host-root
        hostPath:
          path: /
EOF
kubectl apply -f gc-daemonset.yaml

# Verify agent enrollment
curl -s "https://management.guardicore.com/api/v3.0/agents?status=active" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" | python3 -m json.tool
```

### Step 2: Map Application Dependencies with Reveal

Use Guardicore Reveal to discover and visualize application communication patterns.

```bash
# Query discovered application flows via API
curl -s "https://management.guardicore.com/api/v3.0/connections" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -d '{
    "time_range": {"from": "2026-02-17T00:00:00Z", "to": "2026-02-24T00:00:00Z"},
    "filter": {
      "source_label": "web-tier",
      "destination_label": "app-tier"
    },
    "aggregation": "process",
    "limit": 1000
  }' | python3 -m json.tool

# Export application dependency map
curl -s "https://management.guardicore.com/api/v3.0/maps/export" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -d '{
    "format": "json",
    "labels": ["web-tier", "app-tier", "db-tier"],
    "time_range": "7d"
  }' -o app-dependency-map.json

# Typical discovery findings:
# web-tier -> app-tier: TCP 8080, 8443 (expected)
# app-tier -> db-tier: TCP 5432, 3306 (expected)
# web-tier -> db-tier: TCP 5432 (UNEXPECTED - should be blocked)
# app-tier -> internet: TCP 443 (verify if needed)
```

### Step 3: Create Segmentation Labels and Policies

Define labels and create ring-fence policies around applications.

```bash
# Create labels for application tiers
curl -X POST "https://management.guardicore.com/api/v3.0/labels" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "PCI-CDE",
    "description": "Cardholder Data Environment workloads",
    "criteria": {"ip_ranges": ["10.10.0.0/16"]},
    "color": "#FF0000"
  }'

# Create segmentation policy: Allow web-to-app communication
curl -X POST "https://management.guardicore.com/api/v3.0/policies" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Web-to-App Allowed",
    "action": "ALLOW",
    "priority": 100,
    "source": {"labels": ["web-tier"]},
    "destination": {"labels": ["app-tier"]},
    "services": [
      {"protocol": "TCP", "port": 8080},
      {"protocol": "TCP", "port": 8443}
    ],
    "log": true,
    "enabled": true,
    "section": "application-segmentation"
  }'

# Create deny policy: Block web-to-database direct access
curl -X POST "https://management.guardicore.com/api/v3.0/policies" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Block Web-to-DB Direct",
    "action": "DENY",
    "priority": 200,
    "source": {"labels": ["web-tier"]},
    "destination": {"labels": ["db-tier"]},
    "services": [{"protocol": "TCP", "port_range": "1-65535"}],
    "log": true,
    "alert": true,
    "enabled": true
  }'

# Create ring-fence policy for PCI CDE
curl -X POST "https://management.guardicore.com/api/v3.0/policies" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "PCI CDE Ring Fence",
    "action": "DENY",
    "priority": 50,
    "source": {"labels": ["!PCI-CDE"]},
    "destination": {"labels": ["PCI-CDE"]},
    "services": [{"protocol": "TCP", "port_range": "1-65535"}],
    "log": true,
    "alert": true,
    "enabled": true
  }'
```

### Step 4: Test Policies in Reveal Mode Before Enforcement

Simulate policy enforcement without blocking traffic.

```bash
# Enable reveal mode (log-only) for new policies
curl -X PATCH "https://management.guardicore.com/api/v3.0/policies/POLICY_ID" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -d '{"enforcement_mode": "REVEAL"}'

# Check what would be blocked in reveal mode
curl -s "https://management.guardicore.com/api/v3.0/violations" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -d '{
    "time_range": "24h",
    "policy_id": "POLICY_ID",
    "limit": 100
  }' | python3 -c "
import json, sys
data = json.load(sys.stdin)
for v in data.get('violations', []):
    print(f\"{v['source_ip']}:{v['source_process']} -> {v['dest_ip']}:{v['dest_port']} [{v['action']}]\")
"

# After validation, switch to enforcement
curl -X PATCH "https://management.guardicore.com/api/v3.0/policies/POLICY_ID" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -d '{"enforcement_mode": "ENFORCE"}'
```

### Step 5: Monitor and Respond to Policy Violations

Set up alerting and continuous monitoring for segmentation violations.

```bash
# Configure SIEM integration for policy violations
curl -X POST "https://management.guardicore.com/api/v3.0/integrations/syslog" \
  -H "Authorization: Bearer ${GC_API_TOKEN}" \
  -d '{
    "name": "Splunk SIEM",
    "host": "splunk-syslog.company.com",
    "port": 514,
    "protocol": "TCP",
    "format": "CEF",
    "events": ["policy_violation", "agent_status", "deception_alert"]
  }'

# Splunk query for microsegmentation violations
# index=guardicore sourcetype=guardicore:policy
# | where action="DENY" AND enforcement_mode="ENFORCE"
# | stats count by src_ip, dst_ip, dst_port, policy_name
# | sort -count
```

## Key Concepts

| Term | Definition |
|------|------------|
| Microsegmentation | Network security technique creating granular security zones around individual workloads or applications to control east-west traffic |
| Reveal Mode | Guardicore's simulation mode that logs policy decisions without enforcing them, allowing validation before blocking |
| Ring-Fence Policy | Isolation policy that restricts all traffic into or out of a defined group of assets (e.g., PCI CDE) |
| Application Dependency Map | Visual representation of discovered network communication patterns between workloads showing processes, ports, and protocols |
| East-West Traffic | Network traffic flowing laterally between workloads within a data center, as opposed to north-south traffic crossing the perimeter |
| Process-Level Visibility | Guardicore's ability to identify which process on a workload initiated or received a network connection |

## Tools & Systems

- **Akamai Guardicore Segmentation**: Agent-based microsegmentation platform with application visualization and policy enforcement
- **Guardicore Reveal**: Network visualization engine mapping application dependencies across hybrid environments
- **Guardicore Centra**: Management console for policy creation, monitoring, and incident investigation
- **Guardicore Agents**: Lightweight agents deployed on workloads collecting process-level network telemetry
- **Guardicore Insight**: Analytics engine for compliance reporting and segmentation effectiveness measurement

## Common Scenarios

### Scenario: PCI DSS Microsegmentation for E-Commerce Platform

**Context**: An e-commerce company must isolate its Cardholder Data Environment (CDE) from the rest of the corporate network for PCI DSS compliance. The CDE spans 200 servers across on-prem and AWS.

**Approach**:
1. Deploy Guardicore agents on all 200 CDE servers and 300 non-CDE servers
2. Run Reveal for 2 weeks to map all communication patterns into and out of the CDE
3. Identify and remediate unexpected flows (e.g., dev servers connecting to production CDE)
4. Create ring-fence policy blocking all non-CDE to CDE traffic by default
5. Create explicit allow policies for validated CDE communication paths
6. Test in Reveal mode for 1 week, validate no legitimate traffic blocked
7. Switch to enforcement mode and monitor for violations
8. Generate PCI DSS segmentation validation report showing enforced controls

**Pitfalls**: Agent deployment on legacy systems (Windows Server 2012) may require manual installation. Ring-fence policies must account for management traffic (monitoring, patching, backup). Start with broad allow rules and progressively tighten. Application owners must validate dependency maps before enforcement.

## Output Format

```
Microsegmentation Deployment Report
==================================================
Organization: E-Commerce Corp
Report Date: 2026-02-23

AGENT DEPLOYMENT:
  Total workloads:            500
  Agents installed:           487 (97.4%)
  Agents active:              482 (98.9%)
  Agentless (flow logs):       13

POLICY COVERAGE:
  Total policies:              45
  Allow rules:                 38
  Deny rules:                   7
  Reveal mode:                  3
  Enforced:                    42

TRAFFIC ANALYSIS (7 days):
  Total flows observed:        2,456,789
  Flows matching allow:        2,441,234 (99.4%)
  Flows matching deny:            15,555 (0.6%)
  Unclassified flows:                 0

PCI CDE ISOLATION:
  CDE workloads:               200
  Ring-fence violations:         0 (last 30 days)
  Authorized CDE entry points:  4
  Lateral movement paths blocked: 95%
```

===== END FILE =====


===== FILE: skills/implementing-microsegmentation-with-guardicore/LICENSE =====

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


===== FILE: skills/implementing-microsegmentation-with-guardicore/references/api-reference.md =====
# API Reference: Implementing Microsegmentation with Guardicore

## Akamai Guardicore API

```python
import requests
headers = {"Authorization": "Bearer <token>"}
base = "https://guardicore.example.com/api/v3.0"
# Get assets
assets = requests.get(f"{base}/assets", headers=headers).json()
# Get policies
policies = requests.get(f"{base}/policies", headers=headers).json()
# Get traffic map
traffic = requests.get(f"{base}/connections", headers=headers,
                       params={"from_time": "2024-01-01"}).json()
```

## Policy Modes

| Mode | Description |
|------|-------------|
| Reveal | Monitor only, log violations |
| Enforce | Block unauthorized traffic |
| Override | Temporary exception |

## Ringfencing Pattern

| Rule | Source | Destination | Action |
|------|--------|-------------|--------|
| 1 | Frontend | Backend:8443 | Allow |
| 2 | Backend | Database:5432 | Allow |
| 3 | Any | Backend:* | Deny |
| 4 | Backend | Any | Deny |

## Segmentation Metrics

| Metric | Target |
|--------|--------|
| Coverage rate | > 80% of flows |
| Enforced policies | > 90% |
| Cross-zone flows controlled | 100% |
| Default deny coverage | All zones |

## Traffic Analysis Fields

| Field | Description |
|-------|-------------|
| `src_ip` | Source IP address |
| `dst_ip` | Destination IP |
| `dst_port` | Destination port |
| `src_label` | Source workload label |
| `dst_label` | Destination workload label |
| `count` | Flow count |

### References

- Akamai Guardicore: https://www.akamai.com/products/akamai-segmentation
- Zero Trust Microsegmentation: https://www.nist.gov/publications/zero-trust-architecture
- NIST SP 800-207: https://csrc.nist.gov/pubs/sp/800/207/final

===== END FILE =====


===== FILE: skills/implementing-microsegmentation-with-guardicore/scripts/agent.py =====
#!/usr/bin/env python3
"""Guardicore Microsegmentation Agent - audits segmentation policies and network flow visibility."""

import json
import argparse
import logging
import os
import subprocess
from collections import defaultdict
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

GC_API = os.environ.get("GUARDICORE_API_URL", "https://gc-centra.example.com/api/v3.0")


def gc_request(api_url, token, endpoint, method="GET", data=None):
    cmd = ["curl", "-s", "-k", "-X", method,
           "-H", f"Authorization: Bearer {token}",
           "-H", "Content-Type: application/json",
           f"{api_url}{endpoint}"]
    if data:
        cmd.extend(["-d", json.dumps(data)])
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    return json.loads(result.stdout) if result.stdout else {}


def authenticate(api_url, username, password):
    """Authenticate to Guardicore Centra and get access token."""
    data = {"username": username, "password": password}
    result = gc_request(api_url, "", "/authenticate", "POST", data)
    return result.get("access_token", "")


def get_segmentation_policies(api_url, token):
    """Retrieve all segmentation policies."""
    return gc_request(api_url, token, "/policies")


def get_network_flows(api_url, token, hours_back=24):
    """Get recent network flow data for analysis."""
    params = f"?time_range={hours_back}h&limit=1000"
    return gc_request(api_url, token, f"/connections{params}")


def get_labels(api_url, token):
    """Get all asset labels/tags."""
    return gc_request(api_url, token, "/labels")


def get_agents(api_url, token):
    """Get deployed agent status."""
    return gc_request(api_url, token, "/agents")


def analyze_policy_coverage(policies, flows):
    """Analyze how well policies cover observed traffic."""
    policy_rules = set()
    for policy in policies:
        for rule in policy.get("rules", []):
            src = rule.get("source", {}).get("label", "any")
            dst = rule.get("destination", {}).get("label", "any")
            port = rule.get("port", "any")
            policy_rules.add((src, dst, str(port)))
    covered = 0
    uncovered_flows = []
    for flow in flows:
        src_label = flow.get("source_label", "unknown")
        dst_label = flow.get("destination_label", "unknown")
        port = str(flow.get("destination_port", ""))
        if (src_label, dst_label, port) in policy_rules or ("any", "any", "any") in policy_rules:
            covered += 1
        else:
            uncovered_flows.append({
                "source": flow.get("source_ip", ""),
                "destination": flow.get("destination_ip", ""),
                "port": port,
                "protocol": flow.get("protocol", ""),
                "bytes": flow.get("bytes_total", 0),
            })
    total = len(flows)
    return {
        "total_flows": total,
        "covered_by_policy": covered,
        "uncovered": len(uncovered_flows),
        "coverage_percent": round(covered / max(total, 1) * 100, 1),
        "top_uncovered_flows": sorted(uncovered_flows, key=lambda x: x["bytes"], reverse=True)[:20],
    }


def detect_lateral_movement_risk(flows):
    """Identify potential lateral movement patterns in east-west traffic."""
    source_targets = defaultdict(set)
    for flow in flows:
        src = flow.get("source_ip", "")
        dst = flow.get("destination_ip", "")
        if src and dst and src != dst:
            source_targets[src].add(dst)
    risks = []
    for src, targets in source_targets.items():
        if len(targets) > 10:
            risks.append({"source_ip": src, "unique_targets": len(targets), "risk": "high"})
    return sorted(risks, key=lambda x: x["unique_targets"], reverse=True)


def audit_agent_health(agents):
    """Audit deployment agent health status."""
    healthy = sum(1 for a in agents if a.get("status") == "online")
    offline = sum(1 for a in agents if a.get("status") == "offline")
    return {"total": len(agents), "online": healthy, "offline": offline,
            "health_percent": round(healthy / max(len(agents), 1) * 100, 1)}


def generate_report(policies, flows, agents, api_url):
    coverage = analyze_policy_coverage(policies, flows)
    lateral = detect_lateral_movement_risk(flows)
    health = audit_agent_health(agents)
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "guardicore_url": api_url,
        "total_policies": len(policies),
        "policy_coverage": coverage,
        "lateral_movement_risks": lateral[:10],
        "agent_health": health,
    }
    return report


def main():
    parser = argparse.ArgumentParser(description="Guardicore Microsegmentation Audit Agent")
    parser.add_argument("--api-url", default=GC_API, help="Guardicore Centra API URL")
    parser.add_argument("--username", required=True, help="API username")
    parser.add_argument("--password", required=True, help="API password")
    parser.add_argument("--hours-back", type=int, default=24, help="Flow analysis window (hours)")
    parser.add_argument("--output", default="microseg_report.json")
    args = parser.parse_args()

    token = authenticate(args.api_url, args.username, args.password)
    policies = get_segmentation_policies(args.api_url, token)
    flows = get_network_flows(args.api_url, token, args.hours_back)
    agents = get_agents(args.api_url, token)
    report = generate_report(policies, flows, agents, args.api_url)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("Coverage: %.1f%%, Agent health: %.1f%%",
                report["policy_coverage"]["coverage_percent"], report["agent_health"]["health_percent"])
    print(json.dumps(report, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-zero-trust-dns-with-nextdns/SKILL.md =====
---
name: implementing-zero-trust-dns-with-nextdns
description: Configure NextDNS as an encrypted (DoH/DoT) zero trust DNS resolver that blocks malicious, phishing, and cryptojacking domains via real-time threat intelligence, detects DNS rebinding and CNAME cloaking, and enforces organizational DNS policy across endpoints. Use when deploying DNS-layer threat blocking and acceptable-use enforcement, or when extending zero trust controls (including Windows 11 Zero Trust DNS) to the DNS resolution path.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zero-trust
- dns
- nextdns
- dns-over-https
- dns-over-tls
- threat-blocking
- dns-filtering
- privacy
- encrypted-dns
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078
- T1190
- T1059
- T1573
- T1486
---

# Implementing Zero Trust DNS with NextDNS

## Overview

NextDNS is a cloud-based DNS resolver that provides encrypted DNS resolution (DNS-over-HTTPS and DNS-over-TLS), real-time threat intelligence blocking, ad and tracker filtering, and granular DNS policy enforcement. In a zero trust architecture, DNS is a critical control point -- every network connection begins with a DNS query, making DNS filtering an effective layer for blocking malicious domains, preventing data exfiltration via DNS tunneling, enforcing acceptable use policies, and gaining visibility into all network communications. NextDNS processes queries using threat intelligence feeds containing millions of malicious domains updated in real-time, blocks cryptojacking and phishing domains, detects DNS rebinding attacks, and supports CNAME cloaking protection. For enterprise environments, Microsoft's Zero Trust DNS (ZTDNS) feature on Windows 11 extends this concept by enforcing that endpoints can only resolve domains through approved protected DNS servers.


## When to Use

- When deploying or configuring implementing zero trust dns with nextdns capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- NextDNS account (free tier: 300,000 queries/month; Pro: unlimited)
- Network devices supporting DoH or DoT configuration
- Administrative access to endpoint DNS settings
- Understanding of DNS protocol and resolution chain
- Familiarity with organizational acceptable use policies

## Architecture

```
    Endpoint Device
         |
    DNS Query (Encrypted)
         |
    +----+----+
    |  DoH/DoT |  (DNS-over-HTTPS or DNS-over-TLS)
    |  Tunnel  |
    +----+----+
         |
    +----+----+
    | NextDNS  |
    | Resolver |
    +----+----+
         |
    +----+----+------+--------+
    |         |       |        |
  Threat   Ad/Tracker Privacy  Parental
  Intel    Blocklists Controls Controls
  Check    Check      Check    Check
    |         |       |        |
    +----+----+------+--------+
         |
    ALLOW or BLOCK
         |
    Response to Endpoint
```

## Configuration Setup

### NextDNS Profile Configuration

```
Dashboard: https://my.nextdns.io

Configuration ID: abc123 (unique per profile)

Endpoints:
  DNS-over-HTTPS: https://dns.nextdns.io/abc123
  DNS-over-TLS:   abc123.dns.nextdns.io
  DNS-over-QUIC:  quic://abc123.dns.nextdns.io
  IPv4:           45.90.28.x, 45.90.30.x (linked to config)
  IPv6:           2a07:a8c0::xx, 2a07:a8c1::xx
```

### Security Settings

```
Security Tab Configuration:
  [x] Threat Intelligence Feeds - Block domains from curated threat feeds
  [x] AI-Driven Threat Detection - Machine learning-based detection
  [x] Google Safe Browsing - Cross-reference with Google's threat database
  [x] Cryptojacking Protection - Block crypto mining domains
  [x] DNS Rebinding Protection - Prevent DNS rebinding attacks
  [x] IDN Homograph Attacks - Block internationalized domain name attacks
  [x] Typosquatting Protection - Block common typosquatting domains
  [x] DGA Protection - Block domain generation algorithm domains
  [x] NRD (Newly Registered Domains) - Block domains < 30 days old
  [x] DDNS (Dynamic DNS) - Block dynamic DNS services
  [x] Parked Domains - Block parked/unused domains
  [x] CSAM - Block child sexual abuse material domains
```

### Privacy Settings

```
Privacy Tab Configuration:
  Blocklists:
    [x] NextDNS Ads & Trackers Blocklist
    [x] OISD (Full)
    [x] EasyPrivacy
    [x] AdGuard DNS Filter

  Native Tracking Protection:
    [x] Block Windows telemetry
    [x] Block Apple telemetry
    [x] Block Samsung telemetry
    [x] Block Xiaomi telemetry
    [x] Block Huawei telemetry
    [x] Block Roku telemetry
    [x] Block Sonos telemetry

  [x] Block Disguised Third-Party Trackers (CNAME cloaking)
  [x] Allow Affiliate & Tracking Links (optional, for business)
```

### Allowlist and Denylist

```
Allowlist (domains that bypass all blocking):
  - login.microsoftonline.com
  - graph.microsoft.com
  - *.company.com

Denylist (always blocked regardless of other settings):
  - known-malicious-domain.com
  - unauthorized-cloud-storage.com
  - personal-email-provider.com  (if policy requires)
```

## Endpoint Deployment

### Linux (systemd-resolved)

```bash
# Configure DNS-over-TLS with systemd-resolved
sudo tee /etc/systemd/resolved.conf << 'EOF'
[Resolve]
DNS=45.90.28.x#abc123.dns.nextdns.io
DNS=45.90.30.x#abc123.dns.nextdns.io
DNS=2a07:a8c0::xx#abc123.dns.nextdns.io
DNS=2a07:a8c1::xx#abc123.dns.nextdns.io
DNSOverTLS=yes
Domains=~.
EOF

sudo systemctl restart systemd-resolved

# Verify
resolvectl status
resolvectl query example.com
```

### Linux (NextDNS CLI)

```bash
# Install NextDNS CLI
sh -c 'sh -e $(curl -sL https://nextdns.io/install)'

# Configure with your profile
sudo nextdns install \
  -config abc123 \
  -report-client-info \
  -auto-activate

# Verify
nextdns status
nextdns log
```

### macOS

```bash
# Install via Homebrew
brew install nextdns/tap/nextdns

# Configure
sudo nextdns install \
  -config abc123 \
  -report-client-info

# Or configure via System Settings > Network > DNS
# Add DNS-over-HTTPS: https://dns.nextdns.io/abc123
```

### Windows

```powershell
# Install NextDNS CLI for Windows
# Download from: https://nextdns.io/download/windows

# Or configure DoH natively (Windows 11)
# Settings > Network & Internet > Ethernet/Wi-Fi > DNS
# Preferred DNS: 45.90.28.x
# DNS over HTTPS: On (Manual template)
# DoH Template: https://dns.nextdns.io/abc123

# PowerShell: Configure DoH
Set-DnsClientDohServerAddress -ServerAddress "45.90.28.x" `
  -DohTemplate "https://dns.nextdns.io/abc123" `
  -AllowFallbackToUdp $false `
  -AutoUpgrade $true
```

### Router-Level Configuration

```
# Most routers support custom DNS servers
# For DoH/DoT-capable routers (pfSense, OPNsense, OpenWrt):

# pfSense DNS Resolver (Unbound):
# Services > DNS Resolver > Custom Options:
server:
  forward-zone:
    name: "."
    forward-tls-upstream: yes
    forward-addr: 45.90.28.x@853#abc123.dns.nextdns.io
    forward-addr: 45.90.30.x@853#abc123.dns.nextdns.io

# OpenWrt (using https-dns-proxy):
opkg update && opkg install https-dns-proxy
uci set https-dns-proxy.default.resolver_url='https://dns.nextdns.io/abc123'
uci commit https-dns-proxy
/etc/init.d/https-dns-proxy restart
```

### Mobile Devices

```
iOS:
  Install NextDNS app from App Store
  Or: Settings > General > VPN & Device Management
  Install NextDNS configuration profile

Android:
  Settings > Network > Private DNS
  DNS Provider: abc123.dns.nextdns.io

  Or: Install NextDNS app from Play Store
```

## Microsoft Zero Trust DNS (Windows 11)

For enterprise Windows environments, Microsoft's ZTDNS enforces that endpoints can only communicate with domains resolved through approved DNS servers.

```powershell
# Enable ZTDNS (Windows 11 23H2+)
# Requires Windows Defender Firewall in enforcing mode

# Configure Protected DNS Servers via Group Policy:
# Computer Configuration > Administrative Templates > Network > DNS Client
# > Configure DNS over HTTPS (DoH) name resolution
# > Protected DNS servers: https://dns.nextdns.io/abc123

# Windows Firewall blocks all traffic to domains not resolved
# through the protected DNS server
```

## Monitoring and Analytics

### Log Analysis

```
NextDNS Analytics Dashboard provides:
  - Total queries over time
  - Blocked queries by category
  - Top domains (allowed and blocked)
  - Top blocked reasons (threat, ad, tracker)
  - Device-level breakdown
  - Geographic query distribution

Log Settings:
  Retention: 1 hour / 6 hours / 1 day / 1 week / 1 month / 3 months / 1 year / 2 years
  Storage Location: US / EU / UK / Switzerland
  Logging: [ ] Enable / [ ] Disable
```

### API Integration

```bash
# NextDNS API for automated monitoring
# Get analytics data
curl -H "X-Api-Key: your-api-key" \
  "https://api.nextdns.io/profiles/abc123/analytics/domains?from=-24h"

# Get blocked domains
curl -H "X-Api-Key: your-api-key" \
  "https://api.nextdns.io/profiles/abc123/analytics/domains?from=-24h&status=blocked"

# Export logs for SIEM integration
curl -H "X-Api-Key: your-api-key" \
  "https://api.nextdns.io/profiles/abc123/logs?from=-1h" \
  | jq '.data[] | select(.status == "blocked")'
```

## Zero Trust DNS Policy Framework

### Policy Tiers

```
Tier 1 - Security (Mandatory for all):
  - Threat intelligence blocking
  - Cryptojacking protection
  - DNS rebinding protection
  - DGA detection
  - NRD blocking (< 30 days)

Tier 2 - Privacy (Recommended):
  - Tracker blocking
  - Native telemetry blocking
  - CNAME cloaking protection

Tier 3 - Compliance (Organization-specific):
  - Category-based blocking
  - Custom allowlists/denylists
  - Time-based access policies
  - Log retention per regulatory requirements
```

## Security Best Practices

1. **Enforce encrypted DNS**: Block plaintext DNS (port 53 UDP/TCP) at the firewall
2. **Use NextDNS CLI on endpoints**: Ensures per-device identification and logging
3. **Enable NRD blocking**: Newly registered domains are overwhelmingly malicious
4. **Block DNS-over-HTTPS bypass**: Ensure browsers use system DNS, not built-in DoH
5. **Review blocklists quarterly**: Remove false positives, add organizational blocks
6. **Enable CNAME cloaking protection**: Prevents tracker evasion via CNAME records
7. **Set appropriate log retention**: Balance privacy with forensic needs (90 days recommended)
8. **Monitor for DNS tunneling**: Watch for unusual query patterns and high entropy domains
9. **Deploy at router level**: Catches all devices including IoT and unmanaged endpoints
10. **Combine with endpoint DNS**: Defense in depth with both router and per-device filtering

## References

- [NextDNS Documentation](https://nextdns.io/)
- [NextDNS Configuration Guide (GitHub)](https://github.com/yokoffing/NextDNS-Config)
- [Microsoft Zero Trust DNS](https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/zero-trust-dns/)
- [NIST SP 800-81-2: Secure DNS Deployment Guide](https://csrc.nist.gov/publications/detail/sp/800-81/2/final)
- [RFC 8484: DNS Queries over HTTPS (DoH)](https://tools.ietf.org/html/rfc8484)
- [RFC 7858: DNS over TLS (DoT)](https://tools.ietf.org/html/rfc7858)

===== END FILE =====


===== FILE: skills/implementing-zero-trust-dns-with-nextdns/LICENSE =====

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


===== FILE: skills/implementing-zero-trust-dns-with-nextdns/assets/template.md =====
# NextDNS Zero Trust DNS Deployment Template

## Configuration
- **NextDNS Config ID**: _______________
- **Plan**: [ ] Free [ ] Pro [ ] Business [ ] Enterprise
- **Log Retention**: _____ days
- **Log Storage Region**: [ ] US [ ] EU [ ] UK [ ] Switzerland

## Security Settings Checklist
- [ ] Threat Intelligence Feeds enabled
- [ ] AI-Driven Threat Detection enabled
- [ ] Google Safe Browsing enabled
- [ ] Cryptojacking Protection enabled
- [ ] DNS Rebinding Protection enabled
- [ ] IDN Homograph Attack Protection enabled
- [ ] Typosquatting Protection enabled
- [ ] DGA Protection enabled
- [ ] NRD Protection (< 30 days) enabled
- [ ] DDNS Blocking enabled
- [ ] Parked Domains blocking enabled

## Privacy Settings Checklist
- [ ] Blocklists configured (list: ___)
- [ ] Native Tracking Protection enabled
- [ ] CNAME Cloaking Protection enabled

## Deployment Targets

| Target | Method | Status | Config Verified |
|---|---|---|---|
| Router | DoT/DoH | [ ] Complete | [ ] Verified |
| Windows endpoints | NextDNS CLI | [ ] Complete | [ ] Verified |
| macOS endpoints | NextDNS CLI | [ ] Complete | [ ] Verified |
| Linux servers | systemd-resolved | [ ] Complete | [ ] Verified |
| iOS devices | MDM profile | [ ] Complete | [ ] Verified |
| Android devices | Private DNS | [ ] Complete | [ ] Verified |

## Network Controls
- [ ] Plaintext DNS (port 53) blocked at firewall
- [ ] Unauthorized DoH endpoints blocked
- [ ] Split DNS configured for internal domains
- [ ] Browser DoH disabled in favor of system DNS

===== END FILE =====


===== FILE: skills/implementing-zero-trust-dns-with-nextdns/references/api-reference.md =====
# API Reference: Zero Trust DNS with NextDNS

## NextDNS REST API

### Authentication
```
Header: X-Api-Key: <your-api-key>
Base URL: https://api.nextdns.io
```

### Profile Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/profiles/{id}` | Get profile configuration |
| GET | `/profiles/{id}/security` | Get security feature settings |
| GET | `/profiles/{id}/denylist` | Get blocked domains list |
| GET | `/profiles/{id}/allowlist` | Get allowed domains list |
| GET | `/profiles/{id}/logs` | Get DNS query logs |
| GET | `/profiles/{id}/analytics/status` | Query volume analytics |
| GET | `/profiles/{id}/analytics/domains` | Top queried domains |
| GET | `/profiles/{id}/analytics/blockedReasons` | Block reason breakdown |

### Security Feature Keys
| Key | Feature |
|-----|---------|
| `threatIntelligenceFeeds` | Real-time threat intel blocking |
| `aiDetection` | AI-based threat detection |
| `googleSafeBrowsing` | Google Safe Browsing integration |
| `cryptojacking` | Cryptomining domain blocking |
| `dnsRebinding` | DNS rebinding attack protection |
| `idnHomographs` | IDN homograph attack protection |
| `typosquatting` | Typosquatting domain detection |
| `dga` | Domain generation algorithm blocking |
| `nrd` | Newly registered domain blocking |

### Log Entry Fields
| Field | Description |
|-------|-------------|
| `domain` | Queried domain name |
| `status` | `allowed`, `blocked`, or `default` |
| `reasons` | Array of block reasons |
| `clientIp` | Requesting client IP |
| `timestamp` | Query timestamp (ISO 8601) |

## References
- NextDNS API: https://nextdns.github.io/api/
- NextDNS Security: https://nextdns.io/security

===== END FILE =====


===== FILE: skills/implementing-zero-trust-dns-with-nextdns/references/standards.md =====
# Standards Reference: Zero Trust DNS with NextDNS

## DNS Security Standards

### RFC 8484: DNS Queries over HTTPS (DoH)
- Encrypts DNS queries over HTTPS on port 443
- Prevents DNS eavesdropping and manipulation
- Browser and OS-level support

### RFC 7858: DNS over Transport Layer Security (DoT)
- Encrypts DNS queries over TLS on port 853
- Dedicated port enables network-level policy control
- Supported by Android 9+, systemd-resolved, Unbound

### RFC 9250: DNS over Dedicated QUIC Connections (DoQ)
- Lower latency than DoH/DoT
- Multiplexed queries without head-of-line blocking
- Supported by NextDNS and Adguard

### NIST SP 800-81-2: Secure Domain Name System Deployment Guide
- DNS infrastructure security requirements
- DNSSEC validation recommendations
- DNS monitoring and logging guidance

## Zero Trust DNS Standards

### Microsoft ZTDNS (Windows 11)
- Enforces that endpoints only use approved DNS resolvers
- Windows Firewall blocks traffic to domains not resolved through protected DNS
- Integration with Windows Defender for endpoint protection

### CISA ZTMM - Network Pillar
- DNS encryption requirement for zero trust compliance
- DNS monitoring for visibility and analytics
- DNS filtering as network security control

## Compliance Mapping

### NIST SP 800-53
- SC-20: Secure Name/Address Resolution Service (Authoritative Source)
- SC-21: Secure Name/Address Resolution Service (Recursive or Caching Resolver)
- SC-22: Architecture and Provisioning for Name/Address Resolution Service
- SI-4: Information System Monitoring (DNS logging)

### CIS Controls v8
- Control 9.2: Use DNS Filtering Services
- Control 9.3: Maintain and Enforce Network-Based URL Filters
- Control 13.3: Deploy Network Intrusion Detection (DNS monitoring)

===== END FILE =====


===== FILE: skills/implementing-zero-trust-dns-with-nextdns/references/workflows.md =====
# Workflows: Zero Trust DNS with NextDNS

## Workflow 1: Initial NextDNS Deployment

```
Step 1: Create NextDNS Configuration Profile
  - Sign up at nextdns.io
  - Create configuration profile with unique ID
  - Configure security settings (all threat protection enabled)
  - Configure privacy settings (blocklists, native tracking)
  - Set log retention policy

Step 2: Deploy to Network Infrastructure
  - Configure router-level DNS (DoH/DoT)
  - Block plaintext DNS (port 53) at firewall for bypass prevention
  - Configure split DNS for internal domains
  - Test resolution of allowed and blocked domains

Step 3: Deploy to Endpoints
  - Install NextDNS CLI on managed endpoints
  - Configure mobile devices via app or Private DNS
  - Deploy MDM profile for iOS devices
  - Verify per-device identification in NextDNS dashboard

Step 4: Monitor and Tune
  - Review blocked domains for false positives
  - Add necessary allowlist entries
  - Monitor query patterns for anomalies
  - Adjust blocklists based on organizational needs
```

## Workflow 2: DNS Security Policy Enforcement

```
Step 1: Define DNS Security Policy
  - Mandatory security protections (threat intel, DGA, NRD)
  - Privacy protections (tracker blocking, telemetry)
  - Compliance-specific blocking categories
  - Exception handling process

Step 2: Block Plaintext DNS Bypass
  - Firewall rule: Block outbound port 53 UDP/TCP
  - Exception: Only NextDNS CLI or approved resolvers
  - Block known DoH endpoints not managed by organization
  - Disable browser-level DoH in favor of system DNS

Step 3: Implement Monitoring
  - Set up API integration for log export
  - Forward DNS logs to SIEM
  - Create alerts for suspicious DNS patterns
  - Monitor for DNS tunneling indicators
```

## Workflow 3: Incident Response with DNS Logs

```
Step 1: Detect Suspicious Activity
  - Alert on high-frequency queries to single domain
  - Alert on queries to known C2 domains (auto-blocked)
  - Alert on DGA-like domain patterns
  - Alert on DNS tunneling indicators (high entropy, long subdomains)

Step 2: Investigate
  - Query NextDNS API for device-level DNS logs
  - Correlate blocked domains with threat intelligence
  - Identify affected devices and users
  - Determine scope of potential compromise

Step 3: Respond
  - Add malicious domains to denylist for immediate blocking
  - Isolate affected endpoints
  - Update firewall rules as needed
  - Document findings for incident report
```

===== END FILE =====


===== FILE: skills/implementing-zero-trust-dns-with-nextdns/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for configuring and auditing NextDNS zero trust DNS filtering via API."""

import requests
import json
import argparse
from datetime import datetime, timezone

NEXTDNS_API = "https://api.nextdns.io"


def get_profile(api_key, profile_id):
    """Retrieve NextDNS profile configuration."""
    headers = {"X-Api-Key": api_key}
    resp = requests.get(f"{NEXTDNS_API}/profiles/{profile_id}", headers=headers, timeout=15)
    resp.raise_for_status()
    profile = resp.json()
    print(f"[*] Profile: {profile.get('name', profile_id)}")
    print(f"  Security: {json.dumps(profile.get('security', {}), indent=2)[:200]}")
    return profile


def audit_security_settings(api_key, profile_id):
    """Audit security features enabled on a NextDNS profile."""
    headers = {"X-Api-Key": api_key}
    resp = requests.get(f"{NEXTDNS_API}/profiles/{profile_id}/security", headers=headers, timeout=15)
    resp.raise_for_status()
    security = resp.json()
    findings = []
    checks = {
        "threatIntelligenceFeeds": "Threat intelligence feeds",
        "aiDetection": "AI-driven threat detection",
        "googleSafeBrowsing": "Google Safe Browsing",
        "cryptojacking": "Cryptojacking protection",
        "dnsRebinding": "DNS rebinding protection",
        "idnHomographs": "IDN homograph protection",
        "typosquatting": "Typosquatting protection",
        "dga": "DGA domain protection",
        "nrd": "Newly registered domains blocking",
        "ddns": "Dynamic DNS blocking",
        "csam": "CSAM blocking",
    }
    for key, label in checks.items():
        enabled = security.get(key, False)
        status = "ENABLED" if enabled else "DISABLED"
        if not enabled:
            findings.append({"feature": label, "key": key, "severity": "MEDIUM"})
        print(f"  [{'+' if enabled else '!'}] {label}: {status}")
    print(f"\n[*] {len(findings)} security features disabled")
    return findings


def get_query_logs(api_key, profile_id, limit=100):
    """Retrieve recent DNS query logs for analysis."""
    headers = {"X-Api-Key": api_key}
    params = {"limit": limit}
    resp = requests.get(f"{NEXTDNS_API}/profiles/{profile_id}/logs",
                        headers=headers, params=params, timeout=15)
    resp.raise_for_status()
    logs = resp.json().get("data", [])
    blocked = [l for l in logs if l.get("status") == "blocked"]
    print(f"[*] Query logs: {len(logs)} total, {len(blocked)} blocked")
    for entry in blocked[:10]:
        print(f"  [BLOCKED] {entry.get('domain')} - reason: {entry.get('reasons', ['?'])[0]}")
    return logs


def get_analytics(api_key, profile_id, period="last30d"):
    """Retrieve DNS analytics and threat statistics."""
    headers = {"X-Api-Key": api_key}
    endpoints = {
        "queries": f"/profiles/{profile_id}/analytics/status",
        "domains": f"/profiles/{profile_id}/analytics/domains",
        "blocked_reasons": f"/profiles/{profile_id}/analytics/blockedReasons",
    }
    analytics = {}
    for name, path in endpoints.items():
        resp = requests.get(f"{NEXTDNS_API}{path}", headers=headers,
                            params={"from": f"-{period}"}, timeout=15)
        if resp.status_code == 200:
            analytics[name] = resp.json()
    if "queries" in analytics:
        data = analytics["queries"].get("data", [])
        total = sum(d.get("queries", 0) for d in data)
        blocked = sum(d.get("blockedQueries", 0) for d in data)
        print(f"[*] Analytics ({period}): {total} queries, {blocked} blocked "
              f"({blocked/total*100:.1f}%)" if total else "[*] No query data")
    return analytics


def check_denylist(api_key, profile_id):
    """Check configured denylists and custom blocked domains."""
    headers = {"X-Api-Key": api_key}
    resp = requests.get(f"{NEXTDNS_API}/profiles/{profile_id}/denylist",
                        headers=headers, timeout=15)
    resp.raise_for_status()
    denylist = resp.json()
    entries = denylist.get("data", [])
    print(f"[*] Denylist entries: {len(entries)}")
    for e in entries[:20]:
        print(f"  {e.get('id', 'unknown')}: active={e.get('active', True)}")
    return entries


def generate_report(profile, findings, logs, analytics, output_path):
    """Generate NextDNS audit report."""
    report = {
        "audit_date": datetime.now(timezone.utc).isoformat(),
        "profile": profile.get("name", "unknown"),
        "security_findings": findings,
        "blocked_queries_sample": [l for l in logs if l.get("status") == "blocked"][:20],
        "analytics_summary": analytics,
    }
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\n[*] Report saved to {output_path}")


def main():
    parser = argparse.ArgumentParser(description="NextDNS Zero Trust DNS Audit Agent")
    parser.add_argument("action", choices=["audit", "logs", "analytics", "denylist", "full-audit"])
    parser.add_argument("--api-key", required=True, help="NextDNS API key")
    parser.add_argument("--profile", required=True, help="NextDNS profile ID")
    parser.add_argument("-o", "--output", default="nextdns_audit.json")
    args = parser.parse_args()

    if args.action == "audit":
        get_profile(args.api_key, args.profile)
        audit_security_settings(args.api_key, args.profile)
    elif args.action == "logs":
        get_query_logs(args.api_key, args.profile)
    elif args.action == "analytics":
        get_analytics(args.api_key, args.profile)
    elif args.action == "denylist":
        check_denylist(args.api_key, args.profile)
    elif args.action == "full-audit":
        prof = get_profile(args.api_key, args.profile)
        findings = audit_security_settings(args.api_key, args.profile)
        logs = get_query_logs(args.api_key, args.profile)
        analytics = get_analytics(args.api_key, args.profile)
        generate_report(prof, findings, logs, analytics, args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-zero-trust-dns-with-nextdns/scripts/process.py =====
#!/usr/bin/env python3
"""
NextDNS Zero Trust DNS Configuration and Monitoring.

Manages NextDNS profiles, analyzes DNS logs for threats,
and generates compliance reports for DNS security posture.
"""

import json
import datetime
from dataclasses import dataclass, field
from pathlib import Path
from typing import Optional
import math


@dataclass
class DNSPolicy:
    name: str
    security_features: list = field(default_factory=list)
    privacy_blocklists: list = field(default_factory=list)
    native_tracking: list = field(default_factory=list)
    allowlist: list = field(default_factory=list)
    denylist: list = field(default_factory=list)
    log_retention_days: int = 90


@dataclass
class DNSQueryLog:
    timestamp: str
    domain: str
    query_type: str
    status: str  # "allowed", "blocked"
    blocked_reason: str = ""
    device: str = ""
    client_ip: str = ""


class NextDNSPolicyGenerator:
    """Generate NextDNS configuration profiles."""

    SECURITY_FEATURES = [
        "threat_intelligence_feeds",
        "ai_driven_threat_detection",
        "google_safe_browsing",
        "cryptojacking_protection",
        "dns_rebinding_protection",
        "idn_homograph_attacks",
        "typosquatting_protection",
        "dga_protection",
        "nrd_protection",
        "ddns_blocking",
        "parked_domains",
        "csam_blocking",
    ]

    PRIVACY_BLOCKLISTS = [
        "nextdns_recommended",
        "oisd_full",
        "easyprivacy",
        "adguard_dns_filter",
        "hagezi_pro",
        "steven_black_unified",
    ]

    NATIVE_TRACKING = [
        "windows", "apple", "samsung", "xiaomi",
        "huawei", "roku", "sonos", "alexa",
    ]

    def create_enterprise_policy(self, name: str) -> DNSPolicy:
        return DNSPolicy(
            name=name,
            security_features=self.SECURITY_FEATURES.copy(),
            privacy_blocklists=["nextdns_recommended", "oisd_full", "easyprivacy"],
            native_tracking=["windows", "apple"],
            log_retention_days=90,
        )

    def create_strict_policy(self, name: str) -> DNSPolicy:
        return DNSPolicy(
            name=name,
            security_features=self.SECURITY_FEATURES.copy(),
            privacy_blocklists=self.PRIVACY_BLOCKLISTS.copy(),
            native_tracking=self.NATIVE_TRACKING.copy(),
            log_retention_days=365,
        )

    def create_minimal_policy(self, name: str) -> DNSPolicy:
        return DNSPolicy(
            name=name,
            security_features=[
                "threat_intelligence_feeds",
                "cryptojacking_protection",
                "dns_rebinding_protection",
                "dga_protection",
                "nrd_protection",
            ],
            privacy_blocklists=["nextdns_recommended"],
            native_tracking=[],
            log_retention_days=30,
        )

    def export_policy(self, policy: DNSPolicy, output_path: str):
        config = {
            "name": policy.name,
            "security": {
                "features_enabled": policy.security_features,
            },
            "privacy": {
                "blocklists": policy.privacy_blocklists,
                "native_tracking_protection": policy.native_tracking,
                "block_cname_cloaking": True,
            },
            "allowlist": policy.allowlist,
            "denylist": policy.denylist,
            "logging": {
                "enabled": True,
                "retention_days": policy.log_retention_days,
                "storage_location": "US",
            },
        }
        path = Path(output_path)
        path.parent.mkdir(parents=True, exist_ok=True)
        with open(path, "w") as f:
            json.dump(config, f, indent=2)
        return config


class DNSLogAnalyzer:
    """Analyze DNS query logs for security threats."""

    def __init__(self):
        self.logs: list[DNSQueryLog] = []
        self.findings: list[dict] = []

    def add_log(self, log: DNSQueryLog):
        self.logs.append(log)

    def calculate_entropy(self, domain: str) -> float:
        """Calculate Shannon entropy of a domain name (DNS tunneling indicator)."""
        label = domain.split(".")[0]
        if not label:
            return 0.0
        freq = {}
        for c in label:
            freq[c] = freq.get(c, 0) + 1
        entropy = 0.0
        for count in freq.values():
            p = count / len(label)
            if p > 0:
                entropy -= p * math.log2(p)
        return entropy

    def detect_dns_tunneling(self, threshold: float = 4.0) -> list[dict]:
        """Detect potential DNS tunneling based on subdomain entropy and length."""
        suspects = []
        for log in self.logs:
            if log.status == "blocked":
                continue
            subdomain = log.domain.split(".")[0]
            entropy = self.calculate_entropy(log.domain)
            if entropy > threshold and len(subdomain) > 20:
                suspects.append({
                    "domain": log.domain,
                    "entropy": round(entropy, 2),
                    "subdomain_length": len(subdomain),
                    "device": log.device,
                    "timestamp": log.timestamp,
                })
        if suspects:
            self.findings.append({
                "type": "dns_tunneling_suspect",
                "severity": "HIGH",
                "count": len(suspects),
                "details": suspects[:20],
            })
        return suspects

    def detect_dga_domains(self) -> list[dict]:
        """Detect domain generation algorithm patterns."""
        suspects = []
        for log in self.logs:
            parts = log.domain.split(".")
            if len(parts) >= 2:
                sld = parts[-2]
                entropy = self.calculate_entropy(sld + "." + parts[-1])
                if entropy > 3.5 and len(sld) > 8 and not any(
                    c in sld for c in ["-", "_"]
                ):
                    digits = sum(1 for c in sld if c.isdigit())
                    if digits / len(sld) > 0.3:
                        suspects.append({
                            "domain": log.domain,
                            "entropy": round(entropy, 2),
                            "digit_ratio": round(digits / len(sld), 2),
                        })
        if suspects:
            self.findings.append({
                "type": "dga_domain_suspect",
                "severity": "HIGH",
                "count": len(suspects),
                "details": suspects[:20],
            })
        return suspects

    def get_blocked_summary(self) -> dict:
        """Summarize blocked queries by reason."""
        blocked = [l for l in self.logs if l.status == "blocked"]
        by_reason = {}
        for log in blocked:
            reason = log.blocked_reason or "unknown"
            by_reason[reason] = by_reason.get(reason, 0) + 1

        by_domain = {}
        for log in blocked:
            by_domain[log.domain] = by_domain.get(log.domain, 0) + 1

        top_blocked = sorted(by_domain.items(), key=lambda x: x[1], reverse=True)[:20]

        return {
            "total_blocked": len(blocked),
            "total_allowed": len(self.logs) - len(blocked),
            "block_rate": round(len(blocked) / len(self.logs) * 100, 2) if self.logs else 0,
            "by_reason": by_reason,
            "top_blocked_domains": dict(top_blocked),
        }

    def generate_report(self) -> dict:
        return {
            "report_date": datetime.datetime.now().isoformat(),
            "total_queries": len(self.logs),
            "blocked_summary": self.get_blocked_summary(),
            "security_findings": self.findings,
            "unique_devices": len(set(l.device for l in self.logs if l.device)),
            "unique_domains": len(set(l.domain for l in self.logs)),
        }


def main():
    """Demonstrate NextDNS policy generation and DNS log analysis."""
    # Generate policies
    gen = NextDNSPolicyGenerator()
    enterprise = gen.create_enterprise_policy("Enterprise Standard")
    enterprise.allowlist = ["login.microsoftonline.com", "graph.microsoft.com", "*.company.com"]
    enterprise.denylist = ["malware-c2.example.com", "data-exfil.example.com"]

    config = gen.export_policy(enterprise, "nextdns_enterprise_policy.json")
    print("Enterprise DNS Policy:")
    print(json.dumps(config, indent=2))

    # Analyze sample DNS logs
    analyzer = DNSLogAnalyzer()
    sample_logs = [
        DNSQueryLog("2024-01-15T10:00:00Z", "google.com", "A", "allowed", device="laptop-1"),
        DNSQueryLog("2024-01-15T10:00:01Z", "malware.example.com", "A", "blocked",
                    "threat_intelligence", "laptop-2"),
        DNSQueryLog("2024-01-15T10:00:02Z", "aHR0cHM6Ly9leGFtcGxlLmNvbQ.tunnel.evil.com",
                    "TXT", "allowed", device="server-1"),
        DNSQueryLog("2024-01-15T10:00:03Z", "x8k3m9p2q5.botnet.com", "A", "blocked",
                    "dga_protection", "laptop-3"),
        DNSQueryLog("2024-01-15T10:00:04Z", "office.com", "A", "allowed", device="laptop-1"),
        DNSQueryLog("2024-01-15T10:00:05Z", "ads.tracker.com", "A", "blocked",
                    "ad_blocker", "laptop-1"),
    ]

    for log in sample_logs:
        analyzer.add_log(log)

    analyzer.detect_dns_tunneling()
    analyzer.detect_dga_domains()

    report = analyzer.generate_report()
    print("\n" + "=" * 60)
    print("DNS Security Analysis Report")
    print("=" * 60)
    print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-zero-trust-for-saas-applications/SKILL.md =====
---
name: implementing-zero-trust-for-saas-applications
description: Secures SaaS apps (Microsoft 365, Google Workspace, Salesforce, Slack) via CASB/SSPM deployment, conditional access policies, OAuth app governance, and session-level DLP controls enforcing identity verification and device compliance. Use when adding MFA/device-compliance conditional access, discovering shadow IT, governing OAuth consent grants, or applying session controls to sensitive SaaS data.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zero-trust
- saas-security
- casb
- sspm
- conditional-access
- oauth-governance
- session-controls
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078
- T1190
- T1059
- T1078.004
- T1530
---

# Implementing Zero Trust for SaaS Applications

## When to Use

- When securing access to SaaS applications (Microsoft 365, Google Workspace, Salesforce, Slack)
- When implementing conditional access policies requiring MFA and device compliance for SaaS
- When deploying CASB for shadow IT discovery and unsanctioned app blocking
- When enforcing session-level controls (DLP, download restrictions) for sensitive SaaS data
- When governing OAuth application permissions and detecting excessive consent grants

**Do not use** as a replacement for SaaS-native security controls (configure those first), for applications with no SAML/OIDC support, or when SaaS vendor does not support API integration for CASB/SSPM.

## Prerequisites

- Identity provider with conditional access: Microsoft Entra ID P1/P2, Okta
- CASB solution: Microsoft Defender for Cloud Apps, Netskope, or Zscaler CASB
- SaaS applications configured with SSO via SAML 2.0 or OIDC
- MDM enrollment for device compliance signals (Intune, Jamf)
- DLP policies defined for sensitive data categories

## Workflow

### Step 1: Federate SaaS Authentication Through Identity Provider

Centralize authentication for all SaaS applications through a single IdP.

```powershell
# Configure SAML SSO for Salesforce via Entra ID
Connect-MgGraph -Scopes "Application.ReadWrite.All"

# Create enterprise application for Salesforce
$app = New-MgServicePrincipal -AppId "SALESFORCE_APP_ID" -DisplayName "Salesforce"

# Configure SAML SSO settings
$samlSettings = @{
    preferredSingleSignOnMode = "saml"
    samlSingleSignOnSettings = @{
        relayState = ""
    }
}
Update-MgServicePrincipal -ServicePrincipalId $app.Id -BodyParameter $samlSettings

# Assign user groups to the application
New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $app.Id -BodyParameter @{
    principalId = "SALES_GROUP_ID"
    resourceId = $app.Id
    appRoleId = "DEFAULT_ROLE_ID"
}
```

### Step 2: Create Conditional Access Policies for SaaS Applications

Enforce identity and device requirements before granting SaaS access.

```powershell
# Block access from non-compliant devices to sensitive SaaS apps
$policy = @{
    displayName = "ZT - Require Compliant Device for SaaS"
    state = "enabled"
    conditions = @{
        applications = @{
            includeApplications = @("SALESFORCE_APP_ID", "M365_APP_ID", "SLACK_APP_ID")
        }
        users = @{
            includeUsers = @("All")
            excludeGroups = @("BREAK_GLASS_GROUP")
        }
        clientAppTypes = @("browser", "mobileAppsAndDesktopClients")
    }
    grantControls = @{
        operator = "AND"
        builtInControls = @("mfa", "compliantDevice")
    }
    sessionControls = @{
        cloudAppSecurity = @{
            isEnabled = $true
            cloudAppSecurityType = "mcasConfigured"
        }
        signInFrequency = @{
            value = 8
            type = "hours"
            isEnabled = $true
        }
    }
}
New-MgIdentityConditionalAccessPolicy -BodyParameter $policy

# Block downloads on unmanaged devices
$downloadPolicy = @{
    displayName = "ZT - Block Downloads on Unmanaged Devices"
    state = "enabled"
    conditions = @{
        applications = @{ includeApplications = @("SHAREPOINT_APP_ID") }
        users = @{ includeUsers = @("All") }
        devices = @{
            deviceFilter = @{
                mode = "include"
                rule = "device.isCompliant -ne True -or device.trustType -ne 'ServerAD'"
            }
        }
    }
    sessionControls = @{
        cloudAppSecurity = @{
            isEnabled = $true
            cloudAppSecurityType = "mcasConfigured"
        }
    }
}
New-MgIdentityConditionalAccessPolicy -BodyParameter $downloadPolicy
```

### Step 3: Deploy CASB for Shadow IT Discovery and App Governance

Configure Microsoft Defender for Cloud Apps to discover and control SaaS usage.

```bash
# Query discovered cloud apps via Defender for Cloud Apps API
curl -X GET "https://api.cloudappsecurity.com/api/v1/discovery/" \
  -H "Authorization: Token ${MDCA_API_TOKEN}" \
  -H "Content-Type: application/json"

# Get list of unsanctioned apps
curl -X GET "https://api.cloudappsecurity.com/api/v1/discovery/discovered_apps/" \
  -H "Authorization: Token ${MDCA_API_TOKEN}" \
  -d '{
    "filters": {
      "appTag": {"eq": "unsanctioned"},
      "traffic": {"gte": 1000}
    },
    "sortField": "traffic",
    "sortDirection": "desc"
  }'

# Create session policy for DLP enforcement
curl -X POST "https://api.cloudappsecurity.com/api/v1/policies/" \
  -H "Authorization: Token ${MDCA_API_TOKEN}" \
  -d '{
    "name": "Block PII Upload to SaaS",
    "policyType": "SESSION",
    "severity": "HIGH",
    "enabled": true,
    "sessionPolicyType": "CONTROL_UPLOAD",
    "filters": {
      "fileType": {"eq": ["DOCUMENT", "SPREADSHEET"]},
      "contentInspection": {
        "dataType": ["CREDIT_CARD", "SSN", "PASSPORT"]
      }
    },
    "actions": {
      "block": true,
      "notify": {
        "emailRecipients": ["security-team@company.com"]
      }
    }
  }'
```

### Step 4: Configure OAuth App Governance

Review and restrict OAuth application permissions to prevent excessive consent.

```powershell
# Query OAuth apps with high-privilege permissions
$oauthApps = Invoke-MgGraphRequest -Method GET `
  "https://graph.microsoft.com/v1.0/servicePrincipals?\$filter=tags/any(t:t eq 'WindowsAzureActiveDirectoryIntegratedApp')&\$select=displayName,appId,oauth2PermissionScopes"

# Review consent grants
$grants = Get-MgOauth2PermissionGrant -All
$highRisk = $grants | Where-Object {
    $_.Scope -match "Mail.ReadWrite|Files.ReadWrite.All|Directory.ReadWrite.All"
}

Write-Host "High-risk OAuth grants: $($highRisk.Count)"
$highRisk | ForEach-Object {
    $sp = Get-MgServicePrincipal -ServicePrincipalId $_.ClientId
    Write-Host "  App: $($sp.DisplayName) | Scope: $($_.Scope) | Type: $($_.ConsentType)"
}

# Configure app consent policy to require admin approval
$consentPolicy = @{
    displayName = "Require Admin Approval for High-Risk Permissions"
    conditions = @{
        clientApplications = @{ includeAllClientApplications = $true }
        permissions = @{
            permissionClassification = "high"
            permissions = @(
                @{ permissionValue = "Mail.ReadWrite"; permissionType = "delegated" }
                @{ permissionValue = "Files.ReadWrite.All"; permissionType = "delegated" }
            )
        }
    }
}
```

### Step 5: Implement SaaS Security Posture Management (SSPM)

Audit and remediate SaaS security configuration drift.

```bash
# Query SaaS security posture via CASB API
curl -X GET "https://api.cloudappsecurity.com/api/v1/security_config/" \
  -H "Authorization: Token ${MDCA_API_TOKEN}" \
  -d '{"app": "Microsoft 365"}'

# Common SSPM checks:
# - MFA enforcement for all admin accounts
# - External sharing restrictions in SharePoint/OneDrive
# - Email forwarding rules to external domains blocked
# - Idle session timeout configured (< 8 hours)
# - Legacy authentication protocols disabled
# - Admin consent workflow enabled
# - Conditional access policies active
# - Audit logging enabled for all services
```

## Key Concepts

| Term | Definition |
|------|------------|
| CASB | Cloud Access Security Broker - intermediary enforcing security policies between users and SaaS applications |
| SSPM | SaaS Security Posture Management - continuous monitoring of SaaS application security configurations |
| OAuth Governance | Review and control of third-party application permissions granted through OAuth consent flows |
| Session Controls | Real-time access restrictions (block downloads, DLP inspection, watermarking) applied during active SaaS sessions |
| Shadow IT | Unauthorized SaaS applications used by employees without IT approval or security review |
| Conditional Access | Policy engine evaluating identity, device, location, and risk signals before granting SaaS access |

## Tools & Systems

- **Microsoft Defender for Cloud Apps**: CASB providing shadow IT discovery, session controls, DLP, and SSPM
- **Microsoft Entra ID Conditional Access**: Policy engine for identity-based access control to SaaS applications
- **Netskope CASB**: Cloud-native CASB with inline and API-based SaaS security controls
- **Okta Identity Governance**: OAuth app governance and access certification for SaaS applications
- **SSPM Tools**: AppOmni, Adaptive Shield, Valence Security for SaaS configuration monitoring

## Common Scenarios

### Scenario: Securing Microsoft 365 and Salesforce for 1,000-User Organization

**Context**: A professional services firm with 1,000 users uses Microsoft 365, Salesforce, Slack, and 20+ other SaaS apps. Several data breaches in the industry drive a zero trust initiative for all SaaS access.

**Approach**:
1. Federate all SaaS authentication through Entra ID with SAML SSO
2. Create conditional access policies requiring MFA + compliant device for all SaaS apps
3. Deploy Defender for Cloud Apps for shadow IT discovery (identify 150+ unauthorized apps)
4. Mark unauthorized apps as unsanctioned and block via SWG/proxy
5. Configure session controls: block downloads on unmanaged devices, DLP for file uploads
6. Review OAuth app permissions: revoke 45 high-risk consent grants, enable admin approval workflow
7. Enable SSPM monitoring for Microsoft 365 and Salesforce configurations
8. Set up weekly automated posture reports for security leadership

**Pitfalls**: Conditional access policies need break-glass exclusions. Some legacy SaaS apps may not support modern authentication. Session controls require proxy-based CASB which can impact performance. OAuth app revocation may break integrations; coordinate with app owners first.

## Output Format

```
Zero Trust SaaS Security Report
==================================================
Organization: ProServices Corp
Report Date: 2026-02-23

SAAS INVENTORY:
  Sanctioned Apps: 25
  Unsanctioned (blocked): 127
  Shadow IT Users: 342 (discovered in last 30 days)

CONDITIONAL ACCESS:
  Policies active: 8
  Sign-ins evaluated: 456,789
  Blocked by policy: 2,345 (0.5%)
  MFA enforced: 100% of sign-ins

DEVICE COMPLIANCE:
  Compliant device required: All 25 sanctioned apps
  Sign-ins from compliant: 448,123 (98.1%)
  Sign-ins blocked (non-compliant): 8,666

CASB / DLP:
  DLP violations detected: 89
  Files blocked from upload: 34
  Downloads blocked (unmanaged): 1,234

OAUTH GOVERNANCE:
  Total OAuth apps: 312
  High-risk permissions: 12 (reviewed)
  Revoked consents: 45
  Pending admin approval: 8

SSPM FINDINGS:
  Critical misconfigurations: 3
  High: 7
  Medium: 15
  Remediated this month: 18
```

===== END FILE =====


===== FILE: skills/implementing-zero-trust-for-saas-applications/LICENSE =====

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


===== FILE: skills/implementing-zero-trust-for-saas-applications/references/api-reference.md =====
# API Reference: Zero Trust for SaaS Applications

## Microsoft Graph API v1.0

### Authentication
```python
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Body: grant_type=client_credentials&client_id=X&client_secret=Y&scope=https://graph.microsoft.com/.default
```

### Conditional Access
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/identity/conditionalAccess/policies` | List CA policies |
| GET | `/identity/conditionalAccess/policies/{id}` | Get policy details |

### Enterprise Applications
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/servicePrincipals` | List service principals |
| GET | `/oauth2PermissionGrants` | List OAuth consent grants |
| GET | `/appRoleAssignments` | List app role assignments |

### Identity Protection
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/identityProtection/riskyUsers` | List at-risk users |
| GET | `/identityProtection/riskDetections` | Risk detection events |

### CA Policy Grant Controls
| Control | Description |
|---------|-------------|
| `mfa` | Require multi-factor authentication |
| `compliantDevice` | Require Intune-compliant device |
| `domainJoinedDevice` | Require hybrid Azure AD join |
| `passwordChange` | Force password change |

### Risky OAuth Scopes
| Scope | Risk |
|-------|------|
| `Mail.ReadWrite` | Full mailbox access |
| `Files.ReadWrite.All` | All OneDrive/SharePoint files |
| `Directory.ReadWrite.All` | Full directory modification |

## References
- Graph API: https://learn.microsoft.com/en-us/graph/api/overview
- Conditional Access: https://learn.microsoft.com/en-us/entra/identity/conditional-access/

===== END FILE =====


===== FILE: skills/implementing-zero-trust-for-saas-applications/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for auditing zero trust controls on SaaS applications via Microsoft Graph API."""

import requests
import json
import argparse
from datetime import datetime, timezone

GRAPH_API = "https://graph.microsoft.com/v1.0"


def get_token(tenant_id, client_id, client_secret):
    """Acquire OAuth2 token for Microsoft Graph API."""
    url = f"https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token"
    data = {"grant_type": "client_credentials", "client_id": client_id,
            "client_secret": client_secret, "scope": "https://graph.microsoft.com/.default"}
    resp = requests.post(url, data=data, timeout=30)
    resp.raise_for_status()
    token = resp.json()["access_token"]
    print("[*] Authenticated to Microsoft Graph API")
    return {"Authorization": f"Bearer {token}"}


def list_conditional_access_policies(headers):
    """List Entra ID conditional access policies for SaaS apps."""
    url = f"{GRAPH_API}/identity/conditionalAccess/policies"
    resp = requests.get(url, headers=headers, timeout=30)
    resp.raise_for_status()
    policies = resp.json().get("value", [])
    findings = []
    print(f"\n[*] Conditional Access Policies: {len(policies)}")
    for p in policies:
        state = p.get("state", "disabled")
        conditions = p.get("conditions", {})
        grant_controls = p.get("grantControls", {})
        mfa_required = "mfa" in str(grant_controls.get("builtInControls", [])).lower()
        print(f"  [{'+' if state == 'enabled' else '-'}] {p['displayName']} "
              f"(state={state}, MFA={'Yes' if mfa_required else 'No'})")
        if state == "enabled" and not mfa_required:
            findings.append({"policy": p["displayName"], "issue": "No MFA required",
                             "severity": "HIGH"})
    return policies, findings


def list_enterprise_apps(headers):
    """List enterprise applications (service principals) for shadow IT discovery."""
    url = f"{GRAPH_API}/servicePrincipals"
    params = {"$top": 100, "$select": "displayName,appId,accountEnabled,signInAudience"}
    resp = requests.get(url, headers=headers, params=params, timeout=30)
    resp.raise_for_status()
    apps = resp.json().get("value", [])
    print(f"\n[*] Enterprise Applications: {len(apps)}")
    third_party = [a for a in apps if a.get("signInAudience") != "AzureADMyOrg"]
    print(f"  Third-party apps: {len(third_party)}")
    for a in third_party[:10]:
        print(f"  {a['displayName']} (enabled={a.get('accountEnabled', '?')})")
    return apps


def check_oauth_app_consents(headers):
    """Audit OAuth2 permission grants for overprivileged consents."""
    url = f"{GRAPH_API}/oauth2PermissionGrants"
    resp = requests.get(url, headers=headers, timeout=30)
    resp.raise_for_status()
    grants = resp.json().get("value", [])
    findings = []
    for g in grants:
        scope = g.get("scope", "")
        if any(perm in scope for perm in ["Mail.ReadWrite", "Files.ReadWrite.All",
                                           "Directory.ReadWrite.All", "User.ReadWrite.All"]):
            findings.append({
                "clientId": g.get("clientId"), "scope": scope,
                "consentType": g.get("consentType"), "severity": "HIGH",
            })
    print(f"\n[*] OAuth grants: {len(grants)} total, {len(findings)} overprivileged")
    for f in findings[:5]:
        print(f"  [!] {f['clientId']}: {f['scope'][:80]}")
    return findings


def check_sign_in_risk(headers, days=7):
    """Check risky sign-ins to SaaS applications."""
    url = f"{GRAPH_API}/identityProtection/riskyUsers"
    params = {"$filter": "riskState eq 'atRisk'", "$top": 50}
    resp = requests.get(url, headers=headers, params=params, timeout=30)
    if resp.status_code == 200:
        users = resp.json().get("value", [])
        print(f"\n[*] Risky users: {len(users)}")
        for u in users[:10]:
            print(f"  [!] {u.get('userDisplayName', 'N/A')} - risk: {u.get('riskLevel')}")
        return users
    return []


def generate_report(policies, ca_findings, oauth_findings, risky_users, output_path):
    """Generate SaaS zero trust audit report."""
    report = {
        "audit_date": datetime.now(timezone.utc).isoformat(),
        "summary": {
            "conditional_access_policies": len(policies),
            "ca_findings": len(ca_findings),
            "overprivileged_oauth": len(oauth_findings),
            "risky_users": len(risky_users),
        },
        "ca_findings": ca_findings,
        "oauth_findings": oauth_findings[:20],
        "risky_users": risky_users[:20],
    }
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\n[*] Report saved to {output_path}")


def main():
    parser = argparse.ArgumentParser(description="SaaS Zero Trust Audit Agent")
    parser.add_argument("action", choices=["ca-policies", "apps", "oauth", "risk", "full-audit"])
    parser.add_argument("--tenant-id", required=True)
    parser.add_argument("--client-id", required=True)
    parser.add_argument("--client-secret", required=True)
    parser.add_argument("-o", "--output", default="saas_zt_audit.json")
    args = parser.parse_args()

    headers = get_token(args.tenant_id, args.client_id, args.client_secret)
    if args.action == "ca-policies":
        list_conditional_access_policies(headers)
    elif args.action == "apps":
        list_enterprise_apps(headers)
    elif args.action == "oauth":
        check_oauth_app_consents(headers)
    elif args.action == "risk":
        check_sign_in_risk(headers)
    elif args.action == "full-audit":
        policies, ca_f = list_conditional_access_policies(headers)
        list_enterprise_apps(headers)
        oauth_f = check_oauth_app_consents(headers)
        risky = check_sign_in_risk(headers)
        generate_report(policies, ca_f, oauth_f, risky, args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-zero-trust-network-access-with-zscaler/SKILL.md =====
---
name: implementing-zero-trust-network-access-with-zscaler
description: Configures Zero Trust Network Access using Zscaler Private Access (ZPA) to broker identity-based, context-aware connections between authenticated users and internal applications through the Zscaler Zero Trust Exchange, without placing users on the corporate network. Use when replacing traditional VPN architectures with ZTNA, or when brokering secure per-application access for remote and hybrid users via Zscaler.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zero-trust
- ztna
- zscaler
- network-access
- vpn-replacement
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078
- T1190
- T1059
---

# Implementing Zero Trust Network Access with Zscaler

## Prerequisites

- Understanding of zero trust principles (NIST SP 800-207)
- Familiarity with identity providers (Okta, Azure AD, Ping Identity)
- Knowledge of network security fundamentals
- Access to Zscaler Private Access (ZPA) tenant

## Overview

Zero Trust Network Access (ZTNA) replaces traditional VPN architectures by enforcing identity-based, context-aware access to private applications without placing users on the corporate network. Zscaler Private Access (ZPA) is a leading ZTNA solution that brokers secure connections between authenticated users and internal applications through the Zscaler Zero Trust Exchange cloud platform.

This skill covers end-to-end deployment of ZPA including connector setup, application segmentation, policy configuration, and integration with identity providers for continuous verification.


## When to Use

- When deploying or configuring implementing zero trust network access with zscaler capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- Familiarity with zero trust architecture concepts and tools
- Access to a test or lab environment for safe execution
- Python 3.8+ with required dependencies installed
- Appropriate authorization for any testing activities

## Architecture

### Zscaler Private Access Components

1. **Client Connector**: Lightweight agent on user endpoints that establishes outbound TLS tunnels to the nearest ZPA Service Edge
2. **ZPA Service Edge**: Cloud-hosted broker (or Private Service Edge on-premises) that stitches user-to-app connections after policy evaluation
3. **App Connector**: Lightweight VM deployed in the application environment that creates outbound tunnels to the Service Edge
4. **ZPA Admin Portal**: Centralized management console for defining applications, segments, and access policies

### Connection Flow

```
User Device (Client Connector)
    |
    v [Outbound TLS tunnel]
ZPA Service Edge (Policy Evaluation + IdP Auth)
    |
    v [Outbound TLS tunnel]
App Connector --> Internal Application
```

Key principle: No inbound connections are required. Both the Client Connector and App Connector initiate outbound-only connections, eliminating the attack surface of traditional VPNs.

## Key Concepts

### Application Segments
Define specific applications or groups of applications by IP address, FQDN, port, and protocol. Segments enable granular microsegmentation rather than broad network access.

### Access Policies
Policies combine user identity, group membership, device posture, and contextual signals (location, time) to grant or deny access to application segments.

### Server Groups
Logical groupings of App Connectors that serve specific application segments, enabling high availability and geographic distribution.

### Browser Access
ZPA supports clientless browser-based access for web applications, enabling ZTNA for unmanaged devices and third-party users without requiring the Client Connector.

## Workflow

### Phase 1: Foundation Setup

1. **Configure Identity Provider Integration**
   - Navigate to Administration > IdP Configuration in ZPA Admin Portal
   - Add SAML 2.0 or OIDC integration with your IdP (Azure AD, Okta, Ping)
   - Configure SCIM provisioning for automatic user/group synchronization
   - Test SSO authentication flow

2. **Deploy App Connectors**
   - Provision App Connector VMs in each application environment (data center, AWS VPC, Azure VNet)
   - Download the provisioning key from ZPA Admin Portal
   - Install and enroll the App Connector using the provisioning key
   - Verify connector status shows "Healthy" in the admin portal
   - Deploy at least two connectors per environment for high availability

3. **Create Server Groups**
   - Group App Connectors by geographic location or application tier
   - Configure health check intervals and failover behavior

### Phase 2: Application Segmentation

4. **Define Application Segments**
   - Create segments for each application or logical group
   - Specify domains/IPs, ports, and protocols
   - Associate segments with appropriate server groups
   - Enable or disable browser access as needed

5. **Create Segment Groups**
   - Organize application segments into logical groups (e.g., HR apps, Finance apps)
   - Use segment groups to simplify policy management

### Phase 3: Policy Configuration

6. **Configure Access Policies**
   - Define rules matching user groups to application segments
   - Apply conditions: device posture, client type, SAML attributes
   - Order rules by priority (most restrictive first)
   - Create deny rules for blocked access scenarios

7. **Enable Device Posture Checks**
   - Configure posture profiles requiring OS patch level, disk encryption, antivirus status
   - Integrate with endpoint management (CrowdStrike, Microsoft Intune, Carbon Black)
   - Associate posture profiles with access policies

### Phase 4: Client Deployment

8. **Deploy Client Connector**
   - Package the Zscaler Client Connector with enrollment token
   - Deploy via MDM (Intune, Jamf, SCCM) or manual installation
   - Configure forwarding profile to route private app traffic through ZPA
   - Test user authentication and application access

### Phase 5: Monitoring and Optimization

9. **Enable Logging and Monitoring**
   - Configure log streaming to SIEM (Splunk, Sentinel, QRadar)
   - Set up alerts for policy violations, connector health, and authentication failures
   - Review ZPA Insights dashboard for usage analytics

10. **Iterative Refinement**
    - Analyze access logs to identify shadow IT and unauthorized access attempts
    - Refine application segments based on actual traffic patterns
    - Expand coverage from pilot applications to full enterprise deployment

## Validation Checklist

- [ ] Identity provider integration tested with SSO and SCIM sync
- [ ] App Connectors deployed and showing healthy status in all environments
- [ ] Application segments defined with correct IPs/FQDNs, ports, protocols
- [ ] Access policies enforce least-privilege per user group
- [ ] Device posture checks block non-compliant endpoints
- [ ] Client Connector deployed to all managed endpoints
- [ ] Log streaming to SIEM confirmed with test events
- [ ] Failover tested by disabling one App Connector per server group
- [ ] Browser Access configured for web apps requiring third-party access
- [ ] VPN decommission plan documented with rollback procedures

## References

- NIST SP 800-207: Zero Trust Architecture
- CISA Zero Trust Maturity Model v2.0 - Network Pillar
- Zscaler Private Access Architecture Guide
- CSA Software-Defined Perimeter and Zero Trust Specification v2.0

===== END FILE =====


===== FILE: skills/implementing-zero-trust-network-access-with-zscaler/LICENSE =====

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


===== FILE: skills/implementing-zero-trust-network-access-with-zscaler/assets/template.md =====
# ZTNA Deployment Plan Template

## Project Information

| Field | Value |
|---|---|
| Project Name | |
| Organization | |
| Project Lead | |
| Start Date | |
| Target Completion | |
| ZPA Tenant ID | |

## Current State Assessment

### Existing Remote Access
- **VPN Solution**: [e.g., Cisco AnyConnect, Palo Alto GlobalProtect]
- **Concurrent VPN Users**: [number]
- **VPN Concentrator Locations**: [list]
- **Known VPN Issues**: [latency, split-tunnel risks, capacity]

### Application Inventory

| App Name | FQDN/IP | Port | Protocol | User Groups | Criticality | Migration Wave |
|---|---|---|---|---|---|---|
| | | | | | | |
| | | | | | | |

### Identity Provider
- **Primary IdP**: [Azure AD / Okta / Ping Identity / Other]
- **MFA Enabled**: [Yes/No - Method]
- **SCIM Supported**: [Yes/No]
- **Federation Protocol**: [SAML 2.0 / OIDC]

## ZPA Architecture Design

### App Connector Placement

| Environment | Location | VM Count | Server Group |
|---|---|---|---|
| Primary DC | | 2 | |
| DR Site | | 2 | |
| AWS VPC | | 2 | |
| Azure VNet | | 2 | |

### Application Segments

| Segment Name | Domains/IPs | TCP Ports | UDP Ports | Server Group | Browser Access |
|---|---|---|---|---|---|
| | | | | | |
| | | | | | |

### Access Policy Matrix

| Policy Name | User Groups | App Segments | Posture Profile | Conditions | Action |
|---|---|---|---|---|---|
| | | | | | Allow |
| | | | | | Allow |
| Block Default | All | All | None | None | Deny |

### Device Posture Profiles

| Profile Name | OS | Min Version | Disk Encryption | AV/EDR Required | Domain Joined |
|---|---|---|---|---|---|
| Corporate Managed | Windows | 10 22H2+ | BitLocker | CrowdStrike | Yes |
| Corporate Managed | macOS | 14.0+ | FileVault | CrowdStrike | No |
| BYOD | Any | Current-1 | Any | Any | No |

## Migration Plan

### Wave 1: Low-Risk Web Applications (Weeks 1-2)
- [ ] Applications: [list]
- [ ] User groups: [list]
- [ ] Success criteria: [define]
- [ ] Rollback plan: [define]

### Wave 2: Business-Critical Web Apps (Weeks 3-5)
- [ ] Applications: [list]
- [ ] User groups: [list]
- [ ] Success criteria: [define]
- [ ] Rollback plan: [define]

### Wave 3: Non-Web TCP/UDP Applications (Weeks 6-8)
- [ ] Applications: [list]
- [ ] User groups: [list]
- [ ] Success criteria: [define]
- [ ] Rollback plan: [define]

### Wave 4: Legacy Applications (Weeks 9-12)
- [ ] Applications: [list]
- [ ] User groups: [list]
- [ ] Success criteria: [define]
- [ ] Rollback plan: [define]

## SIEM Integration

| Field | Value |
|---|---|
| SIEM Platform | [Splunk / Sentinel / QRadar / Other] |
| Log Streaming Method | [HTTPS / Syslog / Cloud NSS] |
| Log Types | [User Activity / App Connector Status / Policy Violations] |
| Retention Period | |
| Alert Rules | |

## Testing Plan

### Functional Tests
- [ ] User authentication via IdP
- [ ] Application access per policy
- [ ] Device posture enforcement
- [ ] Browser Access for web apps
- [ ] Failover between App Connectors
- [ ] Client Connector auto-update

### Security Tests
- [ ] Unauthorized user access denied
- [ ] Non-compliant device blocked or restricted
- [ ] No lateral movement between segments
- [ ] Log events captured in SIEM
- [ ] Incident response playbook tested

### Performance Tests
- [ ] Application latency within SLA
- [ ] Throughput meets requirements
- [ ] Service Edge selection optimal for user locations

## Sign-Off

| Stakeholder | Role | Approval | Date |
|---|---|---|---|
| | Network Security | | |
| | Identity Team | | |
| | Application Owners | | |
| | CISO / Security Lead | | |
| | IT Operations | | |

===== END FILE =====


===== FILE: skills/implementing-zero-trust-network-access-with-zscaler/references/api-reference.md =====
# API Reference: Zscaler Private Access (ZPA)

## ZPA Management API

### Authentication
```
POST https://config.private.zscaler.com/signin
Body: client_id=X&client_secret=Y
Returns: {"access_token": "...", "token_type": "Bearer"}
```

### Application Segments
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/mgmtconfig/v1/admin/customers/{id}/application` | List app segments |
| POST | `/mgmtconfig/v1/admin/customers/{id}/application` | Create app segment |

### Server Groups
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/mgmtconfig/v1/admin/customers/{id}/serverGroup` | List server groups |

### Access Policies
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/mgmtconfig/v1/admin/customers/{id}/policySet/rules` | List policy rules |

### Connectors
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/mgmtconfig/v1/admin/customers/{id}/connector` | List connectors |

### App Segment Fields
| Field | Description |
|-------|-------------|
| `name` | Application segment name |
| `enabled` | Whether segment is active |
| `bypassType` | `NEVER`, `ALWAYS`, or `ON_NET` |
| `domainNames` | FQDN list for the segment |
| `tcpPortRanges` | Allowed TCP port ranges |

### Bypass Types
| Value | Security Implication |
|-------|---------------------|
| `NEVER` | Always enforce ZPA (recommended) |
| `ALWAYS` | Bypass ZPA entirely (high risk) |
| `ON_NET` | Bypass when on corporate network |

## References
- ZPA API: https://help.zscaler.com/zpa/about-zpa-api
- ZPA Admin Guide: https://help.zscaler.com/zpa

===== END FILE =====


===== FILE: skills/implementing-zero-trust-network-access-with-zscaler/references/standards.md =====
# Standards and Frameworks Reference

## NIST SP 800-207: Zero Trust Architecture

### Core Tenets Applicable to ZTNA
1. **All data sources and computing services are considered resources** - Every application behind ZPA is treated as a discrete resource requiring explicit access grants
2. **All communication is secured regardless of network location** - ZPA encrypts all tunnels end-to-end regardless of whether users are on-premises or remote
3. **Access to individual enterprise resources is granted on a per-session basis** - ZPA evaluates policy for each connection request rather than granting persistent network access
4. **Access to resources is determined by dynamic policy** - ZPA policies incorporate identity, device posture, location, and behavioral signals
5. **The enterprise monitors and measures the integrity and security posture of all owned and associated assets** - Device posture checks validate endpoint compliance before granting access
6. **All resource authentication and authorization are dynamic and strictly enforced before access is allowed** - ZPA requires authentication through IdP and authorization through policy engine for every session

### NIST ZTA Deployment Models
- **Enhanced Identity Governance**: ZPA implements this model by using identity as the primary decision factor combined with device trust signals
- **Micro-Segmentation**: ZPA application segments function as software-defined microsegments at the application layer
- **Software Defined Perimeters**: ZPA directly implements the SDP model with its broker-based architecture

### NIST SP 800-207A: Zero Trust Architecture Model for Cloud-Native Applications
- Extends zero trust principles to multi-cloud environments
- ZPA App Connectors can be deployed across AWS, Azure, GCP, and on-premises
- Supports workload-to-workload zero trust with ZPA for workloads

## CISA Zero Trust Maturity Model v2.0

### Network Pillar
| Maturity Level | Capability | ZPA Implementation |
|---|---|---|
| Traditional | Macro-segmentation with static rules | Legacy VPN replaced by ZPA |
| Initial | Define network architecture with isolation | App Connectors isolate segments |
| Advanced | Micro-perimeters with identity-based access | Per-app segments with IdP integration |
| Optimal | Dynamic microsegmentation with continuous verification | Real-time posture + behavioral analytics |

### Identity Pillar
| Maturity Level | Capability | ZPA Implementation |
|---|---|---|
| Traditional | Password-based, agency-managed | Basic IdP integration |
| Initial | MFA for privileged users, federated identity | SAML/OIDC with IdP, SCIM provisioning |
| Advanced | MFA for all users, phishing-resistant | Conditional access with posture checks |
| Optimal | Continuous validation, risk-based authentication | ZPA + CrowdStrike/UEBA integration |

### Devices Pillar
| Maturity Level | Capability | ZPA Implementation |
|---|---|---|
| Traditional | Limited device visibility | Manual device inventory |
| Initial | Compliance enforcement for some devices | Basic posture profiles |
| Advanced | Real-time device analytics | CrowdStrike ZTA score integration |
| Optimal | Continuous diagnostics and mitigation | EDR-driven dynamic access decisions |

## CSA Software-Defined Perimeter Specification v2.0

### SDP Architecture Mapping to ZPA
| SDP Component | ZPA Equivalent |
|---|---|
| SDP Controller | ZPA Service Edge + Policy Engine |
| Initiating Host (IH) | Client Connector |
| Accepting Host (AH) | App Connector |
| SDP Gateway | ZPA Service Edge |

### SDP Deployment Models
- **Client-to-Gateway**: Standard ZPA deployment (user to application via Service Edge)
- **Client-to-Server**: ZPA Browser Access (direct browser connection through Service Edge)
- **Server-to-Server**: ZPA Workload-to-Workload (App Connector to App Connector)
- **Client-to-Server-to-Client**: Not directly supported in ZPA

## DoD Zero Trust Reference Architecture v2.0

### Pillar Alignment
- ZPA maps to the **Network & Environment** pillar through application-layer microsegmentation
- ZPA maps to the **User** pillar through IdP integration and continuous authentication
- ZPA maps to the **Device** pillar through endpoint posture assessment
- ZPA maps to the **Application & Workload** pillar through per-application access control
- Visibility & Analytics pillar addressed through ZPA log streaming and analytics dashboards

## Compliance Mapping

| Regulation | Requirement | ZPA Capability |
|---|---|---|
| NIST 800-53 AC-4 | Information flow enforcement | Application segment policies |
| NIST 800-53 AC-17 | Remote access | ZTNA replaces VPN |
| PCI DSS 4.0 Req 1 | Network security controls | Microsegmentation per cardholder segment |
| HIPAA 164.312(e) | Transmission security | End-to-end encrypted tunnels |
| SOX Section 404 | Access controls over financial systems | Auditable per-session access logs |
| FedRAMP | Continuous monitoring | ZPA FedRAMP Moderate authorized |

===== END FILE =====


===== FILE: skills/implementing-zero-trust-network-access-with-zscaler/references/workflows.md =====
# ZTNA Implementation Workflows

## Workflow 1: Initial ZPA Deployment

```
┌─────────────────┐
│ Pre-Assessment   │
│ - Inventory apps │
│ - Map user groups│
│ - Classify data  │
└───────┬─────────┘
        v
┌─────────────────────┐
│ IdP Integration      │
│ - SAML/OIDC config   │
│ - SCIM provisioning  │
│ - MFA enrollment     │
│ - Test SSO flow      │
└───────┬─────────────┘
        v
┌─────────────────────┐
│ App Connector Deploy │
│ - Provision VMs      │
│ - Generate enroll key│
│ - Install + enroll   │
│ - Health validation  │
└───────┬─────────────┘
        v
┌─────────────────────┐
│ Application Segments │
│ - Define apps by     │
│   FQDN/IP + ports   │
│ - Create seg groups  │
│ - Map to server grps │
└───────┬─────────────┘
        v
┌─────────────────────┐
│ Access Policies      │
│ - User->App mapping  │
│ - Posture conditions │
│ - Deny rules         │
│ - Priority ordering  │
└───────┬─────────────┘
        v
┌─────────────────────┐
│ Client Deployment    │
│ - Package connector  │
│ - MDM distribution   │
│ - Forwarding profile │
│ - User acceptance    │
└───────┬─────────────┘
        v
┌─────────────────────┐
│ Validation & Monitor │
│ - Access testing     │
│ - SIEM integration   │
│ - Dashboard setup    │
│ - Incident playbooks │
└─────────────────────┘
```

## Workflow 2: Access Request Evaluation (Runtime)

```
User Request
    │
    v
┌──────────────────┐    ┌──────────────────┐
│ Client Connector  │───>│ ZPA Service Edge  │
│ - Capture request │    │ - Receive tunnel  │
│ - Forward to edge │    │ - Identify user   │
└──────────────────┘    └────────┬─────────┘
                                 │
                    ┌────────────v────────────┐
                    │ Authentication          │
                    │ - Redirect to IdP       │
                    │ - Validate SAML/OIDC    │
                    │ - Check MFA completion  │
                    └────────────┬────────────┘
                                 │
                    ┌────────────v────────────┐
                    │ Authorization            │
                    │ - Match access policies  │
                    │ - Evaluate posture       │
                    │ - Check context signals  │
                    │ - Apply least privilege  │
                    └────────────┬────────────┘
                                 │
                    ┌────YES─────┴─────NO────┐
                    v                         v
           ┌──────────────┐         ┌──────────────┐
           │ Grant Access  │         │ Deny Access   │
           │ - Select App  │         │ - Log denial  │
           │   Connector   │         │ - Alert SIEM  │
           │ - Stitch tunnel│        │ - User notify │
           │ - Monitor     │         └──────────────┘
           └──────────────┘
```

## Workflow 3: VPN-to-ZTNA Migration

```
Phase 1: Assessment (Weeks 1-2)
├── Catalog all VPN-accessed applications
├── Map user groups to applications
├── Identify application dependencies
├── Baseline VPN performance metrics
└── Document compliance requirements

Phase 2: Parallel Deployment (Weeks 3-6)
├── Deploy ZPA alongside existing VPN
├── Configure App Connectors for pilot apps
├── Create policies mirroring VPN ACLs
├── Deploy Client Connector to pilot users
└── Validate access and performance

Phase 3: Migration Waves (Weeks 7-16)
├── Wave 1: Low-risk web applications
├── Wave 2: Business-critical web apps
├── Wave 3: Non-web TCP/UDP applications
├── Wave 4: Legacy applications
└── Each wave: test → validate → migrate → monitor

Phase 4: VPN Decommission (Weeks 17-20)
├── Verify all applications accessible via ZPA
├── Disable VPN for migrated user groups
├── Monitor for access issues (2-week soak)
├── Decommission VPN concentrators
└── Update disaster recovery documentation
```

## Workflow 4: Device Posture Enforcement

```
┌───────────────────┐
│ Device Connects    │
└───────┬───────────┘
        v
┌───────────────────┐
│ Posture Assessment │
│ - OS version       │
│ - Patch level      │
│ - Disk encryption  │
│ - AV/EDR status    │
│ - Firewall enabled │
│ - Domain joined    │
└───────┬───────────┘
        v
┌───────────────────┐
│ Posture Evaluation │
│ Compare against    │
│ posture profiles   │
└───┬──────────┬────┘
    │          │
  PASS       FAIL
    │          │
    v          v
┌────────┐ ┌──────────────────┐
│ Full   │ │ Restricted Access │
│ Access │ │ - Browser only    │
└────────┘ │ - Limited apps    │
           │ - Remediation msg │
           └──────────────────┘
```

## Workflow 5: Incident Response with ZPA

```
Alert Triggered (SIEM/SOAR)
    │
    v
┌──────────────────┐
│ 1. Triage         │
│ - Review ZPA logs │
│ - Identify user   │
│ - Identify app    │
│ - Classify event  │
└───────┬──────────┘
        v
┌──────────────────┐
│ 2. Containment    │
│ - Revoke user     │
│   access in ZPA   │
│ - Isolate app     │
│   segment         │
│ - Block device    │
│   via posture     │
└───────┬──────────┘
        v
┌──────────────────┐
│ 3. Investigation  │
│ - Pull session    │
│   logs from ZPA   │
│ - Correlate with  │
│   IdP/EDR/SIEM    │
│ - Map lateral     │
│   movement        │
└───────┬──────────┘
        v
┌──────────────────┐
│ 4. Recovery       │
│ - Update policies │
│ - Re-enable access│
│ - Post-incident   │
│   review          │
└──────────────────┘
```

===== END FILE =====


===== FILE: skills/implementing-zero-trust-network-access-with-zscaler/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for auditing Zscaler Private Access (ZPA) zero trust configuration via API."""

import argparse
import json
import os
import requests
from datetime import datetime, timezone

ZPA_BASE = os.environ.get("ZPA_BASE_URL", "https://config.private.zscaler.com")


def authenticate(client_id, client_secret, customer_id):
    """Authenticate to Zscaler ZPA API."""
    url = f"{ZPA_BASE}/signin"
    payload = {"client_id": client_id, "client_secret": client_secret}
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    resp = requests.post(url, data=payload, headers=headers, timeout=30)
    resp.raise_for_status()
    token = resp.json()["access_token"]
    print("[*] Authenticated to ZPA API")
    return {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}


def list_app_segments(headers, customer_id):
    """List ZPA application segments."""
    url = f"{ZPA_BASE}/mgmtconfig/v1/admin/customers/{customer_id}/application"
    resp = requests.get(url, headers=headers, timeout=30)
    resp.raise_for_status()
    apps = resp.json().get("list", [])
    print(f"\n[*] Application Segments: {len(apps)}")
    findings = []
    for app in apps:
        bypass = app.get("bypassType", "NEVER")
        if bypass != "NEVER":
            findings.append({"name": app["name"], "bypass": bypass, "severity": "HIGH"})
        print(f"  {app['name']} - bypass={bypass}, enabled={app.get('enabled', False)}")
    return apps, findings


def list_server_groups(headers, customer_id):
    """List server groups and their connectors."""
    url = f"{ZPA_BASE}/mgmtconfig/v1/admin/customers/{customer_id}/serverGroup"
    resp = requests.get(url, headers=headers, timeout=30)
    resp.raise_for_status()
    groups = resp.json().get("list", [])
    print(f"\n[*] Server Groups: {len(groups)}")
    for g in groups:
        connectors = g.get("connectors", [])
        print(f"  {g['name']} - {len(connectors)} connectors, enabled={g.get('enabled')}")
    return groups


def list_access_policies(headers, customer_id):
    """List ZPA access policies to verify least-privilege enforcement."""
    url = f"{ZPA_BASE}/mgmtconfig/v1/admin/customers/{customer_id}/policySet/rules"
    resp = requests.get(url, headers=headers, timeout=30)
    resp.raise_for_status()
    rules = resp.json().get("list", [])
    findings = []
    print(f"\n[*] Access Policy Rules: {len(rules)}")
    for r in rules:
        action = r.get("action", "")
        conditions = r.get("conditions", [])
        if action == "ALLOW" and not conditions:
            findings.append({"rule": r.get("name"), "issue": "ALLOW with no conditions",
                             "severity": "CRITICAL"})
            print(f"  [!] {r.get('name')}: ALLOW without conditions")
        else:
            print(f"  {r.get('name')}: action={action}, conditions={len(conditions)}")
    return rules, findings


def check_connector_health(headers, customer_id):
    """Check connector enrollment and health status."""
    url = f"{ZPA_BASE}/mgmtconfig/v1/admin/customers/{customer_id}/connector"
    resp = requests.get(url, headers=headers, timeout=30)
    resp.raise_for_status()
    connectors = resp.json().get("list", [])
    issues = []
    for c in connectors:
        status = c.get("currentVersion", "unknown")
        enabled = c.get("enabled", False)
        if not enabled:
            issues.append({"connector": c.get("name"), "issue": "disabled", "severity": "MEDIUM"})
        print(f"  {c.get('name')}: enabled={enabled}, version={status}")
    print(f"[*] Connectors: {len(connectors)} total, {len(issues)} disabled")
    return connectors, issues


def generate_report(apps, app_findings, policy_findings, connector_issues, output_path):
    """Generate ZPA audit report."""
    report = {
        "audit_date": datetime.now(timezone.utc).isoformat(),
        "summary": {"app_segments": len(apps), "bypass_findings": len(app_findings),
                     "policy_findings": len(policy_findings),
                     "connector_issues": len(connector_issues)},
        "bypass_findings": app_findings, "policy_findings": policy_findings,
        "connector_issues": connector_issues,
    }
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\n[*] Report saved to {output_path}")


def main():
    parser = argparse.ArgumentParser(description="Zscaler ZPA Zero Trust Audit Agent")
    parser.add_argument("action", choices=["apps", "servers", "policies", "connectors", "full-audit"])
    parser.add_argument("--client-id", required=True)
    parser.add_argument("--client-secret", required=True)
    parser.add_argument("--customer-id", required=True)
    parser.add_argument("-o", "--output", default="zpa_audit.json")
    args = parser.parse_args()

    headers = authenticate(args.client_id, args.client_secret, args.customer_id)
    if args.action == "apps":
        list_app_segments(headers, args.customer_id)
    elif args.action == "servers":
        list_server_groups(headers, args.customer_id)
    elif args.action == "policies":
        list_access_policies(headers, args.customer_id)
    elif args.action == "connectors":
        check_connector_health(headers, args.customer_id)
    elif args.action == "full-audit":
        apps, af = list_app_segments(headers, args.customer_id)
        list_server_groups(headers, args.customer_id)
        _, pf = list_access_policies(headers, args.customer_id)
        _, ci = check_connector_health(headers, args.customer_id)
        generate_report(apps, af, pf, ci, args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-zero-trust-network-access-with-zscaler/scripts/process.py =====
#!/usr/bin/env python3
"""
ZTNA Deployment Readiness Assessment and ZPA Configuration Validator

This script performs pre-deployment checks for Zscaler Private Access (ZPA)
implementation, validates existing configurations, and generates deployment
readiness reports.
"""

import json
import csv
import socket
import ssl
import subprocess
import sys
import ipaddress
from datetime import datetime
from pathlib import Path
from typing import Optional


def check_dns_resolution(fqdn: str) -> dict:
    """Validate DNS resolution for application FQDNs."""
    result = {"fqdn": fqdn, "resolved": False, "addresses": [], "error": None}
    try:
        addresses = socket.getaddrinfo(fqdn, None)
        result["resolved"] = True
        result["addresses"] = list(set(addr[4][0] for addr in addresses))
    except socket.gaierror as e:
        result["error"] = str(e)
    return result


def check_port_connectivity(host: str, port: int, timeout: int = 5) -> dict:
    """Test TCP connectivity to application endpoints."""
    result = {
        "host": host,
        "port": port,
        "reachable": False,
        "latency_ms": None,
        "error": None,
    }
    start = datetime.now()
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout)
        sock.connect((host, port))
        elapsed = (datetime.now() - start).total_seconds() * 1000
        result["reachable"] = True
        result["latency_ms"] = round(elapsed, 2)
        sock.close()
    except (socket.timeout, ConnectionRefusedError, OSError) as e:
        result["error"] = str(e)
    return result


def check_tls_certificate(host: str, port: int = 443) -> dict:
    """Validate TLS certificate for HTTPS applications."""
    result = {
        "host": host,
        "port": port,
        "valid": False,
        "issuer": None,
        "subject": None,
        "expiry": None,
        "days_remaining": None,
        "error": None,
    }
    try:
        context = ssl.create_default_context()
        with socket.create_connection((host, port), timeout=10) as sock:
            with context.wrap_socket(sock, server_hostname=host) as ssock:
                cert = ssock.getpeercert()
                result["valid"] = True
                result["issuer"] = dict(x[0] for x in cert.get("issuer", []))
                result["subject"] = dict(x[0] for x in cert.get("subject", []))
                expiry_str = cert.get("notAfter", "")
                if expiry_str:
                    expiry = datetime.strptime(expiry_str, "%b %d %H:%M:%S %Y %Z")
                    result["expiry"] = expiry.isoformat()
                    result["days_remaining"] = (expiry - datetime.utcnow()).days
    except Exception as e:
        result["error"] = str(e)
    return result


def validate_app_segment_config(config: dict) -> list:
    """Validate application segment configuration against best practices."""
    issues = []

    if not config.get("name"):
        issues.append({"severity": "critical", "message": "Application segment name is missing"})

    domains = config.get("domains", [])
    ips = config.get("ip_addresses", [])
    if not domains and not ips:
        issues.append({
            "severity": "critical",
            "message": "No domains or IP addresses defined for segment"
        })

    ports = config.get("tcp_ports", []) + config.get("udp_ports", [])
    if not ports:
        issues.append({"severity": "critical", "message": "No ports defined for segment"})

    for port_range in config.get("tcp_ports", []):
        if "-" in str(port_range):
            start, end = port_range.split("-")
            if int(end) - int(start) > 100:
                issues.append({
                    "severity": "warning",
                    "message": f"Wide port range {port_range} violates least-privilege. Consider narrowing."
                })

    if not config.get("server_group"):
        issues.append({
            "severity": "warning",
            "message": "No server group assigned. High availability not configured."
        })

    for ip in ips:
        try:
            network = ipaddress.ip_network(ip, strict=False)
            if network.prefixlen < 24:
                issues.append({
                    "severity": "warning",
                    "message": f"Broad IP range {ip} (/{network.prefixlen}). Consider narrowing to specific hosts."
                })
        except ValueError:
            pass

    if config.get("bypass_type") == "always":
        issues.append({
            "severity": "critical",
            "message": "Bypass is set to 'always'. Traffic will not be inspected."
        })

    return issues


def validate_access_policy(policy: dict) -> list:
    """Validate access policy configuration."""
    issues = []

    if not policy.get("name"):
        issues.append({"severity": "critical", "message": "Policy name is missing"})

    if not policy.get("conditions"):
        issues.append({
            "severity": "critical",
            "message": "No conditions defined. Policy grants unrestricted access."
        })

    conditions = policy.get("conditions", {})
    if not conditions.get("user_groups") and not conditions.get("users"):
        issues.append({
            "severity": "warning",
            "message": "No user or group conditions. Consider restricting by group."
        })

    if not conditions.get("posture_profiles"):
        issues.append({
            "severity": "warning",
            "message": "No device posture profile attached. Unmanaged devices may access."
        })

    if policy.get("action") == "allow" and not conditions.get("saml_attributes"):
        issues.append({
            "severity": "info",
            "message": "Consider adding SAML attribute conditions for finer-grained access."
        })

    app_segments = policy.get("app_segments", [])
    if len(app_segments) > 20:
        issues.append({
            "severity": "warning",
            "message": f"Policy covers {len(app_segments)} segments. Consider splitting for manageability."
        })

    return issues


def generate_app_inventory_csv(apps: list, output_path: str) -> str:
    """Generate a CSV inventory of applications for ZPA migration planning."""
    fieldnames = [
        "app_name", "fqdn", "ip_address", "port", "protocol",
        "user_groups", "criticality", "current_access_method",
        "migration_wave", "zpa_segment_name", "status"
    ]
    with open(output_path, "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        for app in apps:
            writer.writerow({field: app.get(field, "") for field in fieldnames})
    return output_path


def assess_deployment_readiness(config: dict) -> dict:
    """Perform comprehensive deployment readiness assessment."""
    report = {
        "timestamp": datetime.now().isoformat(),
        "overall_status": "ready",
        "checks": [],
        "summary": {"critical": 0, "warning": 0, "info": 0, "passed": 0},
    }

    # Check IdP configuration
    idp = config.get("identity_provider", {})
    if idp.get("type") in ["saml", "oidc"]:
        report["checks"].append({
            "category": "Identity Provider",
            "check": "IdP type configured",
            "status": "passed",
            "details": f"Type: {idp['type']}"
        })
        report["summary"]["passed"] += 1
    else:
        report["checks"].append({
            "category": "Identity Provider",
            "check": "IdP type configured",
            "status": "critical",
            "details": "No IdP integration configured"
        })
        report["summary"]["critical"] += 1

    if idp.get("scim_enabled"):
        report["checks"].append({
            "category": "Identity Provider",
            "check": "SCIM provisioning enabled",
            "status": "passed",
            "details": "Automated user/group sync active"
        })
        report["summary"]["passed"] += 1
    else:
        report["checks"].append({
            "category": "Identity Provider",
            "check": "SCIM provisioning enabled",
            "status": "warning",
            "details": "Manual user management required without SCIM"
        })
        report["summary"]["warning"] += 1

    # Check App Connectors
    connectors = config.get("app_connectors", [])
    if len(connectors) >= 2:
        report["checks"].append({
            "category": "App Connectors",
            "check": "High availability",
            "status": "passed",
            "details": f"{len(connectors)} connectors deployed"
        })
        report["summary"]["passed"] += 1
    elif len(connectors) == 1:
        report["checks"].append({
            "category": "App Connectors",
            "check": "High availability",
            "status": "warning",
            "details": "Single connector. Deploy at least 2 for HA."
        })
        report["summary"]["warning"] += 1
    else:
        report["checks"].append({
            "category": "App Connectors",
            "check": "High availability",
            "status": "critical",
            "details": "No App Connectors configured"
        })
        report["summary"]["critical"] += 1

    # Check application segments
    segments = config.get("app_segments", [])
    for seg in segments:
        seg_issues = validate_app_segment_config(seg)
        for issue in seg_issues:
            report["checks"].append({
                "category": f"App Segment: {seg.get('name', 'unknown')}",
                "check": issue["message"],
                "status": issue["severity"],
                "details": ""
            })
            report["summary"][issue["severity"]] += 1
        if not seg_issues:
            report["summary"]["passed"] += 1

    # Check access policies
    policies = config.get("access_policies", [])
    if not policies:
        report["checks"].append({
            "category": "Access Policies",
            "check": "Policies defined",
            "status": "critical",
            "details": "No access policies configured"
        })
        report["summary"]["critical"] += 1
    for pol in policies:
        pol_issues = validate_access_policy(pol)
        for issue in pol_issues:
            report["checks"].append({
                "category": f"Policy: {pol.get('name', 'unknown')}",
                "check": issue["message"],
                "status": issue["severity"],
                "details": ""
            })
            report["summary"][issue["severity"]] += 1
        if not pol_issues:
            report["summary"]["passed"] += 1

    # Check SIEM integration
    siem = config.get("siem_integration", {})
    if siem.get("enabled"):
        report["checks"].append({
            "category": "Monitoring",
            "check": "SIEM integration",
            "status": "passed",
            "details": f"Streaming to {siem.get('type', 'unknown')}"
        })
        report["summary"]["passed"] += 1
    else:
        report["checks"].append({
            "category": "Monitoring",
            "check": "SIEM integration",
            "status": "warning",
            "details": "No SIEM integration. Access events not centrally monitored."
        })
        report["summary"]["warning"] += 1

    if report["summary"]["critical"] > 0:
        report["overall_status"] = "not_ready"
    elif report["summary"]["warning"] > 3:
        report["overall_status"] = "ready_with_warnings"

    return report


def connectivity_scan(targets: list) -> list:
    """Scan application endpoints for connectivity from App Connector perspective."""
    results = []
    for target in targets:
        host = target.get("host", "")
        ports = target.get("ports", [])

        dns_result = check_dns_resolution(host) if not host.replace(".", "").isdigit() else None

        for port in ports:
            conn_result = check_port_connectivity(host, port)
            tls_result = None
            if port == 443:
                tls_result = check_tls_certificate(host, port)

            results.append({
                "host": host,
                "port": port,
                "dns": dns_result,
                "connectivity": conn_result,
                "tls": tls_result,
            })
    return results


def generate_migration_plan(apps: list) -> dict:
    """Generate a phased VPN-to-ZTNA migration plan."""
    waves = {
        "wave_1": {"name": "Low-risk Web Apps", "apps": [], "duration_weeks": 2},
        "wave_2": {"name": "Business-critical Web Apps", "apps": [], "duration_weeks": 3},
        "wave_3": {"name": "Non-web TCP/UDP Apps", "apps": [], "duration_weeks": 3},
        "wave_4": {"name": "Legacy Applications", "apps": [], "duration_weeks": 4},
    }

    for app in apps:
        criticality = app.get("criticality", "medium").lower()
        protocol = app.get("protocol", "https").lower()
        legacy = app.get("is_legacy", False)

        if legacy:
            waves["wave_4"]["apps"].append(app)
        elif protocol not in ["http", "https"]:
            waves["wave_3"]["apps"].append(app)
        elif criticality in ["high", "critical"]:
            waves["wave_2"]["apps"].append(app)
        else:
            waves["wave_1"]["apps"].append(app)

    total_weeks = sum(w["duration_weeks"] for w in waves.values())
    return {
        "total_duration_weeks": total_weeks,
        "waves": waves,
        "generated": datetime.now().isoformat(),
    }


def main():
    """Run the ZTNA deployment readiness assessment."""
    import argparse

    parser = argparse.ArgumentParser(
        description="ZPA Deployment Readiness Assessment Tool"
    )
    parser.add_argument(
        "--config", type=str, help="Path to ZPA configuration JSON file"
    )
    parser.add_argument(
        "--scan", type=str, help="Path to targets JSON for connectivity scan"
    )
    parser.add_argument(
        "--inventory", type=str, help="Path to app inventory JSON for CSV generation"
    )
    parser.add_argument(
        "--migrate", type=str, help="Path to app list JSON for migration planning"
    )
    parser.add_argument(
        "--output", type=str, default="report.json", help="Output file path"
    )
    args = parser.parse_args()

    if args.config:
        with open(args.config) as f:
            config = json.load(f)
        report = assess_deployment_readiness(config)
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"Readiness report: {report['overall_status']}")
        print(f"  Critical: {report['summary']['critical']}")
        print(f"  Warnings: {report['summary']['warning']}")
        print(f"  Passed:   {report['summary']['passed']}")
        print(f"Report saved to {args.output}")

    elif args.scan:
        with open(args.scan) as f:
            targets = json.load(f)
        results = connectivity_scan(targets)
        with open(args.output, "w") as f:
            json.dump(results, f, indent=2)
        reachable = sum(1 for r in results if r["connectivity"]["reachable"])
        print(f"Scan complete: {reachable}/{len(results)} endpoints reachable")
        print(f"Results saved to {args.output}")

    elif args.inventory:
        with open(args.inventory) as f:
            apps = json.load(f)
        csv_path = args.output.replace(".json", ".csv")
        generate_app_inventory_csv(apps, csv_path)
        print(f"Inventory CSV saved to {csv_path}")

    elif args.migrate:
        with open(args.migrate) as f:
            apps = json.load(f)
        plan = generate_migration_plan(apps)
        with open(args.output, "w") as f:
            json.dump(plan, f, indent=2)
        for wave_id, wave in plan["waves"].items():
            print(f"  {wave_id}: {wave['name']} - {len(wave['apps'])} apps ({wave['duration_weeks']} weeks)")
        print(f"Migration plan saved to {args.output}")

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-zero-trust-with-hashicorp-boundary/SKILL.md =====
---
name: implementing-zero-trust-with-hashicorp-boundary
description: Installs and configures HashiCorp Boundary as a default-deny, identity-aware proxy for infrastructure access, including controller/worker setup, Vault-backed credential brokering, session recording, and OIDC/LDAP auth across an org/project scope hierarchy. Use when replacing VPN or direct network access with just-in-time, credential-less Boundary sessions, or standing up Boundary controllers and workers.
domain: cybersecurity
subdomain: zero-trust-architecture
tags:
- zero-trust
- hashicorp
- boundary
- privileged-access
- vault
- identity-aware-proxy
- session-recording
- just-in-time-access
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.AA-01
- PR.AA-05
- PR.IR-01
- GV.PO-01
mitre_attack:
- T1078
- T1190
- T1059
- T1003
- T1110
mitre_f3:
  version: '1.1'
  tactics:
  - initial-access
  - positioning
  - resource-development
  techniques:
  - id: T1586
    name: Compromise Accounts
    tactic: resource-development
    source: attack
  - id: T1110
    name: Brute Force
    tactic: initial-access
    source: attack
  - id: T1110.004
    name: 'Brute Force:  Credential Stuffing'
    tactic: initial-access
    source: attack
  - id: T1219
    name: Remote Access Tools
    tactic: positioning
    source: attack
  - id: F1033
    name: Insider Access Abuse
    tactic: initial-access
    source: f3
---

# Implementing Zero Trust with HashiCorp Boundary

## Overview

HashiCorp Boundary is an identity-aware proxy that provides secure, zero trust access to infrastructure resources without traditional VPNs or direct network access. Boundary operates on a default-deny model -- users start with no access and must be explicitly granted permissions for specific resources. When integrated with HashiCorp Vault, Boundary can dynamically broker credentials, ensuring users never see or manage underlying secrets. This eliminates credential sprawl and enables just-in-time access with automatic credential revocation when sessions end. Boundary supports session recording for audit compliance, OIDC/LDAP authentication, and manages access through a hierarchical scope model of organizations and projects.


## When to Use

- When deploying or configuring implementing zero trust with hashicorp boundary capabilities in your environment
- When establishing security controls aligned to compliance requirements
- When building or improving security architecture for this domain
- When conducting security assessments that require this implementation

## Prerequisites

- HashiCorp Boundary server (self-hosted or HCP Boundary)
- HashiCorp Vault (for credential brokering)
- Identity provider supporting OIDC (Okta, Azure AD, Auth0)
- PostgreSQL database for Boundary's backend
- TLS certificates for secure communication
- Understanding of PKI and X.509 certificate management

## Architecture

```
                    Identity Provider (OIDC)
                           |
                    Authentication
                           |
                  +--------+--------+
                  |   Boundary      |
                  |   Controller    |
                  |  (Control Plane)|
                  +--------+--------+
                           |
              +------------+------------+
              |                         |
     +--------+--------+      +--------+--------+
     | Boundary Worker |      | Boundary Worker |
     | (Data Plane)    |      | (Data Plane)    |
     +--------+--------+      +--------+--------+
              |                         |
     +--------+--------+      +--------+--------+
     |  Target Hosts   |      |  Target Hosts   |
     |  (SSH, RDP,     |      |  (Databases,    |
     |   K8s, HTTP)    |      |   APIs)         |
     +-----------------+      +-----------------+

     Vault (Credential Brokering)
     - Dynamic database credentials
     - SSH certificate signing
     - Credential libraries
```

## Installation and Configuration

### Boundary Server Setup

```bash
# Install Boundary
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install boundary

# Initialize the database
boundary database init \
  -config=/etc/boundary/controller.hcl

# Start the controller
boundary server -config=/etc/boundary/controller.hcl
```

### Controller Configuration

```hcl
# /etc/boundary/controller.hcl
controller {
  name = "boundary-controller-1"
  description = "Primary Boundary Controller"
  database {
    url = "postgresql://boundary:password@localhost:5432/boundary?sslmode=require"
  }
  public_cluster_addr = "boundary.example.com"
}

listener "tcp" {
  address = "0.0.0.0:9200"
  purpose = "api"
  tls_cert_file = "/etc/boundary/tls/cert.pem"
  tls_key_file  = "/etc/boundary/tls/key.pem"
}

listener "tcp" {
  address = "0.0.0.0:9201"
  purpose = "cluster"
  tls_cert_file = "/etc/boundary/tls/cert.pem"
  tls_key_file  = "/etc/boundary/tls/key.pem"
}

kms "aead" {
  purpose = "root"
  aead_type = "aes-gcm"
  key = "sP1fnF5Xz85RrXM..."  # Use Vault Transit in production
  key_id = "global_root"
}

kms "aead" {
  purpose = "worker-auth"
  aead_type = "aes-gcm"
  key = "8fZBjCUfN0TzjEG..."
  key_id = "global_worker-auth"
}

kms "aead" {
  purpose = "recovery"
  aead_type = "aes-gcm"
  key = "8fZBjCUfN0TzjEG..."
  key_id = "global_recovery"
}
```

### Worker Configuration

```hcl
# /etc/boundary/worker.hcl
worker {
  name = "boundary-worker-1"
  description = "Worker in production VPC"
  public_addr = "worker1.example.com"

  controllers = [
    "boundary.example.com:9201"
  ]

  tags {
    type = ["production"]
    region = ["us-east-1"]
  }
}

listener "tcp" {
  address = "0.0.0.0:9202"
  purpose = "proxy"
}

kms "aead" {
  purpose = "worker-auth"
  aead_type = "aes-gcm"
  key = "8fZBjCUfN0TzjEG..."
  key_id = "global_worker-auth"
}
```

## Terraform Configuration

### Scope and Auth Configuration

```hcl
# main.tf - Boundary resources via Terraform
terraform {
  required_providers {
    boundary = {
      source  = "hashicorp/boundary"
      version = "~> 1.1"
    }
  }
}

provider "boundary" {
  addr             = "https://boundary.example.com:9200"
  recovery_kms_hcl = file("recovery_kms.hcl")
}

# Organization scope
resource "boundary_scope" "org" {
  scope_id                 = "global"
  name                     = "production-org"
  description              = "Production organization scope"
  auto_create_admin_role   = true
  auto_create_default_role = true
}

# Project scope
resource "boundary_scope" "production" {
  name                     = "production"
  description              = "Production infrastructure project"
  scope_id                 = boundary_scope.org.id
  auto_create_admin_role   = true
  auto_create_default_role = true
}

# OIDC Auth Method (Okta example)
resource "boundary_auth_method_oidc" "okta" {
  scope_id               = boundary_scope.org.id
  name                   = "okta"
  description            = "Okta OIDC authentication"
  issuer                 = "https://company.okta.com/oauth2/default"
  client_id              = var.okta_client_id
  client_secret          = var.okta_client_secret
  signing_algorithms     = ["RS256"]
  api_url_prefix         = "https://boundary.example.com:9200"
  claims_scopes          = ["groups"]
  account_claim_maps     = ["oid=sub"]
  is_primary_for_scope   = true
}

# Managed group for auto-assignment
resource "boundary_managed_group" "sre_team" {
  auth_method_id = boundary_auth_method_oidc.okta.id
  name           = "sre-team"
  description    = "SRE team members from Okta"
  filter         = "\"sre-team\" in \"/token/groups\""
}

resource "boundary_managed_group" "dev_team" {
  auth_method_id = boundary_auth_method_oidc.okta.id
  name           = "dev-team"
  description    = "Development team from Okta"
  filter         = "\"dev-team\" in \"/token/groups\""
}
```

### Host Catalogs and Targets

```hcl
# Static host catalog for known infrastructure
resource "boundary_host_catalog_static" "production_servers" {
  name     = "production-servers"
  scope_id = boundary_scope.production.id
}

resource "boundary_host_static" "web_server" {
  name            = "web-server-1"
  host_catalog_id = boundary_host_catalog_static.production_servers.id
  address         = "10.0.1.10"
}

resource "boundary_host_static" "db_server" {
  name            = "db-server-1"
  host_catalog_id = boundary_host_catalog_static.production_servers.id
  address         = "10.0.2.20"
}

# Host set grouping
resource "boundary_host_set_static" "web_servers" {
  name            = "web-servers"
  host_catalog_id = boundary_host_catalog_static.production_servers.id
  host_ids        = [boundary_host_static.web_server.id]
}

resource "boundary_host_set_static" "db_servers" {
  name            = "database-servers"
  host_catalog_id = boundary_host_catalog_static.production_servers.id
  host_ids        = [boundary_host_static.db_server.id]
}

# SSH target
resource "boundary_target" "ssh_production" {
  name         = "ssh-production-servers"
  description  = "SSH access to production servers"
  type         = "ssh"
  scope_id     = boundary_scope.production.id
  default_port = 22

  host_source_ids = [
    boundary_host_set_static.web_servers.id
  ]

  session_max_seconds          = 3600  # 1 hour max session
  session_connection_limit     = 1
  enable_session_recording     = true
  storage_bucket_id            = boundary_storage_bucket.sessions.id

  injected_application_credential_source_ids = [
    boundary_credential_library_vault_ssh_certificate.ssh_cert.id
  ]
}

# Database target with Vault credential brokering
resource "boundary_target" "postgres_production" {
  name         = "postgres-production"
  description  = "PostgreSQL production database"
  type         = "tcp"
  scope_id     = boundary_scope.production.id
  default_port = 5432

  host_source_ids = [
    boundary_host_set_static.db_servers.id
  ]

  session_max_seconds      = 1800  # 30 min max
  session_connection_limit = 5

  brokered_credential_source_ids = [
    boundary_credential_library_vault.postgres_creds.id
  ]
}
```

### Vault Integration for Credential Brokering

```hcl
# Vault credential store
resource "boundary_credential_store_vault" "vault" {
  name        = "vault-store"
  scope_id    = boundary_scope.production.id
  address     = "https://vault.example.com:8200"
  token       = var.vault_token
  namespace   = "production"
}

# Dynamic database credentials from Vault
resource "boundary_credential_library_vault" "postgres_creds" {
  name                = "postgres-dynamic-creds"
  credential_store_id = boundary_credential_store_vault.vault.id
  path                = "database/creds/readonly"
  http_method         = "GET"
  credential_type     = "username_password"
}

# SSH certificate signing via Vault
resource "boundary_credential_library_vault_ssh_certificate" "ssh_cert" {
  name                = "ssh-certificate"
  credential_store_id = boundary_credential_store_vault.vault.id
  path                = "ssh-client-signer/sign/production"
  username            = "admin"
  key_type            = "ed25519"
  key_bits            = 256
  extensions = {
    "permit-pty" = ""
  }
}

# Session recording storage
resource "boundary_storage_bucket" "sessions" {
  name        = "session-recordings"
  scope_id    = "global"
  plugin_name = "aws"
  bucket_name = "boundary-session-recordings"
  attributes_json = jsonencode({
    "region"                      = "us-east-1"
    "disable_credential_rotation" = true
  })
  secrets_json = jsonencode({
    "access_key_id"     = var.aws_access_key
    "secret_access_key" = var.aws_secret_key
  })
}
```

### Role-Based Access Control

```hcl
# SRE team role - full production access
resource "boundary_role" "sre_production" {
  name          = "sre-production-access"
  scope_id      = boundary_scope.production.id
  grant_strings = [
    "ids=*;type=target;actions=list,read,authorize-session",
    "ids=*;type=session;actions=list,read,cancel",
    "ids=*;type=host;actions=list,read",
  ]
  principal_ids = [
    boundary_managed_group.sre_team.id
  ]
}

# Dev team role - limited access
resource "boundary_role" "dev_staging" {
  name          = "dev-staging-access"
  scope_id      = boundary_scope.production.id
  grant_strings = [
    "ids=${boundary_target.ssh_production.id};type=target;actions=read,authorize-session",
  ]
  principal_ids = [
    boundary_managed_group.dev_team.id
  ]
}
```

## Connecting to Targets

```bash
# Authenticate via OIDC
boundary authenticate oidc \
  -auth-method-id amoidc_xxxxx

# List available targets
boundary targets list -scope-id p_xxxxx

# Connect to SSH target (credentials injected by Vault)
boundary connect ssh \
  -target-id ttcp_xxxxx

# Connect to database (credentials brokered by Vault)
boundary connect postgres \
  -target-id ttcp_xxxxx \
  -dbname production

# Use Boundary Desktop client for GUI access
# Download from: https://developer.hashicorp.com/boundary/install
```

## Session Recording and Auditing

```bash
# List session recordings
boundary session-recordings list \
  -scope-id p_xxxxx

# Download session recording for review
boundary session-recordings download \
  -id sr_xxxxx \
  -output recording.cast

# Play back with asciinema
asciinema play recording.cast
```

## Dynamic Host Catalogs

```hcl
# AWS dynamic host catalog - auto-discovers EC2 instances
resource "boundary_host_catalog_plugin" "aws_catalog" {
  scope_id    = boundary_scope.production.id
  name        = "aws-production"
  plugin_name = "aws"

  attributes_json = jsonencode({
    "region"                      = "us-east-1"
    "disable_credential_rotation" = true
  })

  secrets_json = jsonencode({
    "access_key_id"     = var.aws_access_key
    "secret_access_key" = var.aws_secret_key
  })
}

resource "boundary_host_set_plugin" "web_tier" {
  host_catalog_id = boundary_host_catalog_plugin.aws_catalog.id
  name            = "web-tier"
  attributes_json = jsonencode({
    "filters" = [
      "tag:Environment=production",
      "tag:Tier=web"
    ]
  })
}
```

## Security Best Practices

1. **Use Vault KMS for key management** instead of static AEAD keys in production
2. **Enable session recording** for all privileged access targets
3. **Set session time limits** appropriate to the resource sensitivity
4. **Use OIDC managed groups** for automatic role assignment from IdP
5. **Deploy multi-hop workers** for accessing resources across network boundaries
6. **Rotate Vault tokens** used by credential stores regularly
7. **Enable audit logging** on both controllers and workers
8. **Use credential injection** (SSH certificates) over brokering when possible
9. **Implement least-privilege grants** -- avoid wildcard permissions
10. **Review session recordings** regularly for compliance and incident response

## References

- [HashiCorp Boundary Documentation](https://developer.hashicorp.com/boundary/docs)
- [Boundary and Vault Integration](https://developer.hashicorp.com/boundary/docs/concepts/credential-management)
- [From Zero to Hero with Boundary](https://www.hashicorp.com/en/blog/from-zero-to-hero-hashicorp-boundary)
- [Boundary Terraform Provider](https://registry.terraform.io/providers/hashicorp/boundary/latest/docs)
- [Zero Trust with Vault, Consul, and Boundary](https://www.hashicorp.com/en/resources/zero-trust-security-with-hashicorp-vault-consul-and-boundary)

===== END FILE =====


===== FILE: skills/implementing-zero-trust-with-hashicorp-boundary/LICENSE =====

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


===== FILE: skills/implementing-zero-trust-with-hashicorp-boundary/assets/template.md =====
# HashiCorp Boundary Deployment Template

## Deployment Information
- **Organization**: _______________
- **Deployment Type**: [ ] Self-hosted [ ] HCP Boundary
- **Identity Provider**: _______________
- **Vault Integration**: [ ] Yes [ ] No

## Scope Hierarchy

| Scope Type | Name | Description | Owner |
|---|---|---|---|
| Organization | ___ | ___ | ___ |
| Project | ___ | ___ | ___ |
| Project | ___ | ___ | ___ |

## Targets Inventory

| Target Name | Type | Port | Hosts | Session Max | Recording | Credentials |
|---|---|---|---|---|---|---|
| ___ | ssh | 22 | ___ | 3600s | [ ] Yes | injected |
| ___ | tcp | 5432 | ___ | 1800s | [ ] Yes | brokered |
| ___ | tcp | 443 | ___ | 3600s | [ ] Yes | none |

## Role Assignments

| Role | Scope | Grants | Groups |
|---|---|---|---|
| ___ | ___ | ___ | ___ |

## Security Checklist

- [ ] OIDC authentication configured with MFA-enabled IdP
- [ ] Managed groups auto-assign roles from IdP claims
- [ ] Vault credential brokering enabled for database targets
- [ ] SSH certificate injection via Vault SSH engine
- [ ] Session recording enabled for privileged access
- [ ] Session duration limits configured per target
- [ ] KMS configured with Vault Transit (not static AEAD)
- [ ] Workers deployed in each network zone
- [ ] Audit logging enabled on controllers and workers
- [ ] Break-glass recovery KMS configured and secured

===== END FILE =====


===== FILE: skills/implementing-zero-trust-with-hashicorp-boundary/references/api-reference.md =====
# API Reference: HashiCorp Boundary

## Boundary CLI (JSON output)

### Core Commands
```bash
boundary scopes list -scope-id=global -format=json
boundary targets list -scope-id=<id> -format=json
boundary host-catalogs list -scope-id=<id> -format=json
boundary credential-stores list -scope-id=<id> -format=json
boundary sessions list -scope-id=<id> -format=json
boundary auth-methods list -scope-id=global -format=json
```

### Environment Variables
| Variable | Description |
|----------|-------------|
| `BOUNDARY_ADDR` | Controller address (e.g., `http://127.0.0.1:9200`) |
| `BOUNDARY_TOKEN` | Authentication token |

### Target Fields
| Field | Description |
|-------|-------------|
| `name` | Target display name |
| `type` | `tcp` or `ssh` |
| `session_max_seconds` | Maximum session duration |
| `session_connection_limit` | Max concurrent connections (-1 = unlimited) |

### Credential Store Types
| Type | Description |
|------|-------------|
| `vault` | Vault-brokered dynamic credentials (recommended) |
| `static` | Static credentials stored in Boundary |

### Auth Method Types
| Type | Zero Trust Suitability |
|------|----------------------|
| `oidc` | Recommended (SSO, MFA support) |
| `ldap` | Acceptable with MFA |
| `password` | Not recommended for zero trust |

### Session Recording
```bash
boundary targets update tcp -id=<id> -enable-session-recording=true \
  -storage-bucket-id=<bucket-id>
```

## References
- Boundary docs: https://developer.hashicorp.com/boundary/docs
- Boundary CLI: https://developer.hashicorp.com/boundary/docs/commands
- Boundary API: https://developer.hashicorp.com/boundary/api-docs

===== END FILE =====


===== FILE: skills/implementing-zero-trust-with-hashicorp-boundary/references/standards.md =====
# Standards Reference: HashiCorp Boundary Zero Trust

## Core Standards

### NIST SP 800-207: Zero Trust Architecture
- Boundary implements identity-aware proxy architecture (Section 3.2.2)
- Default-deny access model aligns with ZTA core principle
- Dynamic credential brokering eliminates standing privileges
- Session-level authorization satisfies continuous verification

### NIST SP 800-53: Security Controls
- AC-2: Account Management (identity lifecycle via OIDC)
- AC-3: Access Enforcement (role-based grants)
- AC-6: Least Privilege (fine-grained permissions)
- AU-2: Audit Events (session recording and logging)
- AU-3: Content of Audit Records (session playback)
- IA-2: Identification and Authentication (OIDC/LDAP)
- SC-7: Boundary Protection (identity-aware proxy)

### SOC 2 Compliance
- CC6.1: Logical and Physical Access Controls
- CC6.2: Prior to Access, Identity is Established
- CC6.3: Role-Based Access Controls
- CC7.2: Monitoring of System Components
- CC8.1: Change Management

## Boundary Security Model

### Identity-Aware Access
- Authentication via OIDC, LDAP, or password auth methods
- Managed groups for automatic role assignment from IdP claims
- Session tokens with configurable expiry
- No direct network access to targets without Boundary session

### Credential Management
- Brokered credentials: Vault generates and returns credentials to user
- Injected credentials: Boundary injects credentials directly (user never sees them)
- SSH certificate signing: Vault CA issues short-lived SSH certificates
- Dynamic database credentials: just-in-time access with automatic revocation

### Session Controls
- Maximum session duration enforcement
- Connection limits per session
- Session recording for privileged access audit
- Automatic credential revocation on session end

## Integration Standards

### HashiCorp Vault Integration
- Transit KMS for Boundary encryption keys
- Database secrets engine for dynamic credentials
- SSH secrets engine for certificate-based access
- PKI secrets engine for TLS certificate management

### Terraform Infrastructure as Code
- Boundary Terraform provider for declarative configuration
- Version-controlled access policies
- Automated deployment and updates
- Drift detection for access control changes

===== END FILE =====


===== FILE: skills/implementing-zero-trust-with-hashicorp-boundary/references/workflows.md =====
# Workflows: HashiCorp Boundary Zero Trust Implementation

## Workflow 1: Initial Boundary Deployment

```
Step 1: Infrastructure Preparation
  - Provision PostgreSQL database for Boundary backend
  - Generate TLS certificates for controller and workers
  - Configure KMS (Vault Transit or AEAD for dev)
  - Set up network connectivity between components

Step 2: Controller Deployment
  - Install Boundary binary on controller hosts
  - Configure controller with database, KMS, and listeners
  - Initialize database schema
  - Verify controller health and API accessibility

Step 3: Worker Deployment
  - Install Boundary on worker hosts in each network zone
  - Configure worker with controller address and KMS
  - Register workers with tags for routing decisions
  - Verify worker registration and health

Step 4: Identity Provider Integration
  - Configure OIDC auth method with organizational IdP
  - Map IdP groups to Boundary managed groups
  - Test authentication flow end-to-end
  - Configure token and session expiry policies
```

## Workflow 2: Target Onboarding

```
Step 1: Create Scope Hierarchy
  - Define organization scope for each business unit
  - Create project scopes for environment isolation
  - Assign admin roles to scope owners

Step 2: Configure Host Catalogs
  - Static catalogs for fixed infrastructure
  - Dynamic catalogs for cloud resources (AWS, Azure, GCP)
  - Plugin-based catalogs for auto-discovery

Step 3: Define Targets
  - Map each target to host sets
  - Configure default ports and session limits
  - Enable session recording for privileged targets
  - Link credential sources (Vault libraries)

Step 4: Create Access Policies
  - Define roles with minimum necessary grants
  - Assign roles to managed groups from IdP
  - Test access with each role
  - Document access patterns and justifications
```

## Workflow 3: Vault Credential Integration

```
Step 1: Configure Vault Secrets Engines
  - Enable database secrets engine for dynamic credentials
  - Configure SSH secrets engine for certificate signing
  - Set up PKI engine for TLS certificates
  - Define roles with appropriate TTL and permissions

Step 2: Create Boundary Credential Stores
  - Create Vault credential store in Boundary
  - Provide Vault token with appropriate policies
  - Configure namespace if using Vault Enterprise

Step 3: Create Credential Libraries
  - Map Vault paths to Boundary credential libraries
  - Configure credential type (username_password, ssh_certificate)
  - Link libraries to targets as brokered or injected sources

Step 4: Test and Validate
  - Connect to target with dynamic credentials
  - Verify credentials are revoked after session end
  - Confirm session recording captures access
  - Validate audit logs contain credential events
```

## Workflow 4: Access Review and Audit

```
Step 1: Regular Access Review
  - Export role assignments and grant strings
  - Review with resource owners quarterly
  - Remove stale or unnecessary access
  - Update managed group filters if IdP groups change

Step 2: Session Recording Review
  - Review session recordings for privileged targets
  - Investigate anomalous session patterns
  - Export recordings for compliance evidence
  - Archive recordings per retention policy

Step 3: Compliance Reporting
  - Generate access control matrix from Boundary
  - Map controls to compliance framework requirements
  - Document exceptions and compensating controls
  - Present findings to audit and compliance teams
```

===== END FILE =====


===== FILE: skills/implementing-zero-trust-with-hashicorp-boundary/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for auditing HashiCorp Boundary zero trust access configuration."""

import os
import subprocess
import json
import argparse
from datetime import datetime, timezone


def run_boundary_cmd(args_list, addr, token):
    """Execute a boundary CLI command and return parsed JSON."""
    env_vars = {"BOUNDARY_ADDR": addr, "BOUNDARY_TOKEN": token}
    cmd = ["boundary"] + args_list + ["-format=json"]
    result = subprocess.run(cmd, capture_output=True, text=True, env={**dict(os.environ), **env_vars}, timeout=30)
    if result.returncode != 0:
        print(f"  [-] Error: {result.stderr.strip()[:200]}")
        return {}
    return json.loads(result.stdout) if result.stdout else {}


def list_scopes(addr, token):
    """List organization and project scopes."""
    data = run_boundary_cmd(["scopes", "list", "-scope-id=global"], addr, token)
    scopes = data.get("items", [])
    print(f"[*] Scopes: {len(scopes)}")
    for s in scopes:
        print(f"  {s.get('id')}: {s.get('name', 'unnamed')} (type={s.get('type')})")
    return scopes


def list_targets(addr, token, scope_id):
    """List targets (resources users can connect to) in a scope."""
    data = run_boundary_cmd(["targets", "list", f"-scope-id={scope_id}"], addr, token)
    targets = data.get("items", [])
    findings = []
    print(f"\n[*] Targets in scope {scope_id}: {len(targets)}")
    for t in targets:
        session_max = t.get("session_max_seconds", 0)
        conn_limit = t.get("session_connection_limit", -1)
        print(f"  {t.get('name')}: type={t.get('type')}, "
              f"max_sec={session_max}, conn_limit={conn_limit}")
        if conn_limit == -1:
            findings.append({"target": t.get("name"), "issue": "Unlimited connections",
                             "severity": "MEDIUM"})
        if session_max == 0 or session_max > 28800:
            findings.append({"target": t.get("name"),
                             "issue": f"Long session timeout ({session_max}s)", "severity": "HIGH"})
    return targets, findings


def list_host_catalogs(addr, token, scope_id):
    """List host catalogs (static or dynamic)."""
    data = run_boundary_cmd(["host-catalogs", "list", f"-scope-id={scope_id}"], addr, token)
    catalogs = data.get("items", [])
    print(f"\n[*] Host Catalogs in scope {scope_id}: {len(catalogs)}")
    for c in catalogs:
        print(f"  {c.get('id')}: {c.get('name', 'unnamed')} (type={c.get('type')})")
    return catalogs


def list_credential_stores(addr, token, scope_id):
    """List credential stores (Vault integration check)."""
    data = run_boundary_cmd(["credential-stores", "list", f"-scope-id={scope_id}"], addr, token)
    stores = data.get("items", [])
    print(f"\n[*] Credential Stores in scope {scope_id}: {len(stores)}")
    vault_stores = [s for s in stores if s.get("type") == "vault"]
    static_stores = [s for s in stores if s.get("type") == "static"]
    print(f"  Vault-backed: {len(vault_stores)}, Static: {len(static_stores)}")
    findings = []
    if static_stores:
        for s in static_stores:
            findings.append({"store": s.get("name"), "type": "static",
                             "issue": "Static credentials (not Vault-brokered)", "severity": "MEDIUM"})
    return stores, findings


def list_sessions(addr, token, scope_id):
    """List active sessions for audit."""
    data = run_boundary_cmd(["sessions", "list", f"-scope-id={scope_id}"], addr, token)
    sessions = data.get("items", [])
    print(f"\n[*] Active Sessions in scope {scope_id}: {len(sessions)}")
    for s in sessions[:10]:
        print(f"  {s.get('id')[:12]}... user={s.get('user_id', 'N/A')} "
              f"target={s.get('target_id', 'N/A')} status={s.get('status')}")
    return sessions


def check_auth_methods(addr, token, scope_id="global"):
    """Audit configured authentication methods."""
    data = run_boundary_cmd(["auth-methods", "list", f"-scope-id={scope_id}"], addr, token)
    methods = data.get("items", [])
    findings = []
    print(f"\n[*] Auth Methods: {len(methods)}")
    for m in methods:
        mtype = m.get("type", "unknown")
        print(f"  {m.get('name', 'unnamed')}: type={mtype}")
        if mtype == "password":
            findings.append({"method": m.get("name"), "type": mtype,
                             "issue": "Password-only auth (use OIDC for zero trust)", "severity": "HIGH"})
    return methods, findings


def generate_report(target_findings, cred_findings, auth_findings, output_path):
    """Generate Boundary audit report."""
    all_findings = target_findings + cred_findings + auth_findings
    report = {
        "audit_date": datetime.now(timezone.utc).isoformat(),
        "total_findings": len(all_findings),
        "target_findings": target_findings,
        "credential_findings": cred_findings,
        "auth_findings": auth_findings,
    }
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\n[*] Report saved to {output_path}")


def main():
    parser = argparse.ArgumentParser(description="HashiCorp Boundary Zero Trust Audit Agent")
    parser.add_argument("action", choices=["scopes", "targets", "hosts", "creds",
                                           "sessions", "auth", "full-audit"])
    parser.add_argument("--addr", required=True, help="Boundary controller address")
    parser.add_argument("--token", required=True, help="Boundary auth token")
    parser.add_argument("--scope-id", default="global", help="Scope ID to audit")
    parser.add_argument("-o", "--output", default="boundary_audit.json")
    args = parser.parse_args()

    if args.action == "scopes":
        list_scopes(args.addr, args.token)
    elif args.action == "targets":
        list_targets(args.addr, args.token, args.scope_id)
    elif args.action == "hosts":
        list_host_catalogs(args.addr, args.token, args.scope_id)
    elif args.action == "creds":
        list_credential_stores(args.addr, args.token, args.scope_id)
    elif args.action == "sessions":
        list_sessions(args.addr, args.token, args.scope_id)
    elif args.action == "auth":
        check_auth_methods(args.addr, args.token)
    elif args.action == "full-audit":
        scopes = list_scopes(args.addr, args.token)
        all_tf, all_cf, all_af = [], [], []
        _, af = check_auth_methods(args.addr, args.token)
        all_af.extend(af)
        for s in scopes:
            sid = s.get("id")
            _, tf = list_targets(args.addr, args.token, sid)
            all_tf.extend(tf)
            _, cf = list_credential_stores(args.addr, args.token, sid)
            all_cf.extend(cf)
            list_sessions(args.addr, args.token, sid)
        generate_report(all_tf, all_cf, all_af, args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-zero-trust-with-hashicorp-boundary/scripts/process.py =====
#!/usr/bin/env python3
"""
HashiCorp Boundary Zero Trust Access Management.

Generates Boundary Terraform configurations, validates access policies,
and monitors session activity for compliance reporting.
"""

import json
import datetime
from dataclasses import dataclass, field
from pathlib import Path
from typing import Optional


@dataclass
class BoundaryScope:
    name: str
    description: str
    scope_type: str  # "org" or "project"
    parent_scope_id: str = "global"


@dataclass
class BoundaryTarget:
    name: str
    description: str
    target_type: str  # "ssh" or "tcp"
    default_port: int
    host_addresses: list = field(default_factory=list)
    session_max_seconds: int = 3600
    session_connection_limit: int = -1
    enable_recording: bool = False
    credential_type: str = "none"  # "none", "brokered", "injected"
    vault_path: str = ""


@dataclass
class BoundaryRole:
    name: str
    description: str
    scope_id: str
    grant_strings: list = field(default_factory=list)
    principal_groups: list = field(default_factory=list)


class BoundaryTerraformGenerator:
    """Generate Terraform configuration for Boundary resources."""

    def __init__(self, boundary_addr: str, vault_addr: str = ""):
        self.boundary_addr = boundary_addr
        self.vault_addr = vault_addr
        self.scopes: list[BoundaryScope] = []
        self.targets: list[BoundaryTarget] = []
        self.roles: list[BoundaryRole] = []
        self.oidc_config: dict = {}

    def add_scope(self, name: str, description: str, scope_type: str = "project",
                  parent: str = "global") -> BoundaryScope:
        scope = BoundaryScope(name=name, description=description,
                              scope_type=scope_type, parent_scope_id=parent)
        self.scopes.append(scope)
        return scope

    def add_target(self, name: str, description: str, target_type: str,
                   port: int, hosts: list, **kwargs) -> BoundaryTarget:
        target = BoundaryTarget(
            name=name, description=description, target_type=target_type,
            default_port=port, host_addresses=hosts, **kwargs
        )
        self.targets.append(target)
        return target

    def add_role(self, name: str, description: str, scope_id: str,
                 grants: list, groups: list) -> BoundaryRole:
        role = BoundaryRole(
            name=name, description=description, scope_id=scope_id,
            grant_strings=grants, principal_groups=groups
        )
        self.roles.append(role)
        return role

    def configure_oidc(self, issuer: str, client_id: str, scopes: list = None):
        self.oidc_config = {
            "issuer": issuer,
            "client_id": client_id,
            "scopes": scopes or ["openid", "profile", "groups"],
        }

    def generate_provider_block(self) -> str:
        return f'''terraform {{
  required_providers {{
    boundary = {{
      source  = "hashicorp/boundary"
      version = "~> 1.1"
    }}
  }}
}}

provider "boundary" {{
  addr             = "{self.boundary_addr}"
  recovery_kms_hcl = file("recovery_kms.hcl")
}}
'''

    def generate_scopes(self) -> str:
        blocks = []
        for scope in self.scopes:
            resource_name = scope.name.replace("-", "_")
            if scope.scope_type == "org":
                blocks.append(f'''
resource "boundary_scope" "{resource_name}" {{
  scope_id                 = "{scope.parent_scope_id}"
  name                     = "{scope.name}"
  description              = "{scope.description}"
  auto_create_admin_role   = true
  auto_create_default_role = true
}}
''')
            else:
                parent_ref = scope.parent_scope_id.replace("-", "_")
                blocks.append(f'''
resource "boundary_scope" "{resource_name}" {{
  name                     = "{scope.name}"
  description              = "{scope.description}"
  scope_id                 = boundary_scope.{parent_ref}.id
  auto_create_admin_role   = true
  auto_create_default_role = true
}}
''')
        return "\n".join(blocks)

    def generate_targets(self) -> str:
        blocks = []
        for target in self.targets:
            resource_name = target.name.replace("-", "_")
            recording_block = ""
            if target.enable_recording:
                recording_block = """
  enable_session_recording = true
  storage_bucket_id        = boundary_storage_bucket.sessions.id"""

            credential_block = ""
            if target.credential_type == "brokered" and target.vault_path:
                credential_block = f"""
  brokered_credential_source_ids = [
    boundary_credential_library_vault.{resource_name}_creds.id
  ]"""
            elif target.credential_type == "injected" and target.vault_path:
                credential_block = f"""
  injected_application_credential_source_ids = [
    boundary_credential_library_vault_ssh_certificate.{resource_name}_cert.id
  ]"""

            blocks.append(f'''
resource "boundary_target" "{resource_name}" {{
  name         = "{target.name}"
  description  = "{target.description}"
  type         = "{target.target_type}"
  scope_id     = boundary_scope.production.id
  default_port = {target.default_port}

  session_max_seconds      = {target.session_max_seconds}
  session_connection_limit = {target.session_connection_limit}{recording_block}{credential_block}
}}
''')
        return "\n".join(blocks)

    def generate_roles(self) -> str:
        blocks = []
        for role in self.roles:
            resource_name = role.name.replace("-", "_")
            grants = ",\n    ".join(f'"{g}"' for g in role.grant_strings)
            principals = ",\n    ".join(
                f'boundary_managed_group.{g.replace("-", "_")}.id'
                for g in role.principal_groups
            )
            blocks.append(f'''
resource "boundary_role" "{resource_name}" {{
  name          = "{role.name}"
  description   = "{role.description}"
  scope_id      = {role.scope_id}
  grant_strings = [
    {grants}
  ]
  principal_ids = [
    {principals}
  ]
}}
''')
        return "\n".join(blocks)

    def generate_full_config(self) -> str:
        sections = [
            self.generate_provider_block(),
            "# Scopes",
            self.generate_scopes(),
            "# Targets",
            self.generate_targets(),
            "# Roles",
            self.generate_roles(),
        ]
        return "\n".join(sections)

    def export_config(self, output_path: str):
        config = self.generate_full_config()
        path = Path(output_path)
        path.parent.mkdir(parents=True, exist_ok=True)
        with open(path, "w") as f:
            f.write(config)
        return config


class BoundaryAccessAuditor:
    """Audit and validate Boundary access configurations."""

    def __init__(self):
        self.findings: list[dict] = []

    def check_session_limits(self, targets: list[BoundaryTarget]) -> list[dict]:
        for target in targets:
            if target.session_max_seconds > 7200:
                self.findings.append({
                    "severity": "WARNING",
                    "target": target.name,
                    "finding": f"Session max exceeds 2 hours ({target.session_max_seconds}s)",
                    "recommendation": "Reduce session duration for privileged targets",
                })
            if target.session_connection_limit == -1:
                self.findings.append({
                    "severity": "INFO",
                    "target": target.name,
                    "finding": "Unlimited connections per session",
                    "recommendation": "Consider setting connection limits for sensitive targets",
                })
            if target.default_port == 22 and not target.enable_recording:
                self.findings.append({
                    "severity": "HIGH",
                    "target": target.name,
                    "finding": "SSH target without session recording",
                    "recommendation": "Enable session recording for SSH access",
                })
            if target.credential_type == "none":
                self.findings.append({
                    "severity": "MEDIUM",
                    "target": target.name,
                    "finding": "No credential management configured",
                    "recommendation": "Use Vault credential brokering or injection",
                })
        return self.findings

    def check_role_grants(self, roles: list[BoundaryRole]) -> list[dict]:
        for role in roles:
            for grant in role.grant_strings:
                if "ids=*" in grant and "actions=*" in grant:
                    self.findings.append({
                        "severity": "CRITICAL",
                        "role": role.name,
                        "finding": "Wildcard IDs and actions grant (admin-level access)",
                        "recommendation": "Restrict to specific resource types and actions",
                    })
                if "type=target" in grant and "authorize-session" in grant and "ids=*" in grant:
                    self.findings.append({
                        "severity": "HIGH",
                        "role": role.name,
                        "finding": "Role can authorize sessions to all targets",
                        "recommendation": "Restrict to specific target IDs or use target-level grants",
                    })
        return self.findings

    def generate_report(self) -> dict:
        report = {
            "audit_date": datetime.datetime.now().isoformat(),
            "total_findings": len(self.findings),
            "by_severity": {},
            "findings": self.findings,
        }
        for finding in self.findings:
            sev = finding["severity"]
            report["by_severity"][sev] = report["by_severity"].get(sev, 0) + 1
        return report


def main():
    """Generate example Boundary Terraform configuration."""
    gen = BoundaryTerraformGenerator(
        boundary_addr="https://boundary.example.com:9200",
        vault_addr="https://vault.example.com:8200"
    )

    # Create scopes
    gen.add_scope("production-org", "Production Organization", "org")
    gen.add_scope("production", "Production Infrastructure", "project", "production-org")

    # Create targets
    gen.add_target(
        "ssh-web-servers", "SSH to production web servers", "ssh", 22,
        ["10.0.1.10", "10.0.1.11"],
        session_max_seconds=3600, enable_recording=True,
        credential_type="injected", vault_path="ssh-signer/sign/web"
    )
    gen.add_target(
        "postgres-production", "Production PostgreSQL", "tcp", 5432,
        ["10.0.2.20"],
        session_max_seconds=1800,
        credential_type="brokered", vault_path="database/creds/readonly"
    )
    gen.add_target(
        "redis-cache", "Production Redis cache", "tcp", 6379,
        ["10.0.3.30"],
        session_max_seconds=900
    )

    # Create roles
    gen.add_role(
        "sre-full-access", "SRE team full production access",
        "boundary_scope.production.id",
        [
            "ids=*;type=target;actions=list,read,authorize-session",
            "ids=*;type=session;actions=list,read,cancel",
        ],
        ["sre-team"]
    )
    gen.add_role(
        "dev-readonly", "Dev team read-only access",
        "boundary_scope.production.id",
        [
            "ids=*;type=target;actions=list,read",
        ],
        ["dev-team"]
    )

    # Generate and export
    config = gen.export_config("boundary_config.tf")
    print("Generated Terraform configuration:")
    print(config[:2000])

    # Run audit
    auditor = BoundaryAccessAuditor()
    auditor.check_session_limits(gen.targets)
    auditor.check_role_grants(gen.roles)
    report = auditor.generate_report()

    print("\n" + "=" * 60)
    print("Access Audit Report")
    print("=" * 60)
    print(f"Total findings: {report['total_findings']}")
    for sev, count in report["by_severity"].items():
        print(f"  {sev}: {count}")
    for finding in report["findings"]:
        target_or_role = finding.get("target", finding.get("role", "N/A"))
        print(f"\n  [{finding['severity']}] {target_or_role}")
        print(f"    Finding: {finding['finding']}")
        print(f"    Recommendation: {finding['recommendation']}")


if __name__ == "__main__":
    main()

===== END FILE =====
