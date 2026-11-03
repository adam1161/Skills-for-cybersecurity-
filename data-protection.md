

===== FILE: skills/implementing-data-loss-prevention-with-microsoft-purview/SKILL.md =====
---
name: implementing-data-loss-prevention-with-microsoft-purview
description: 'Implements DLP policies using Microsoft Purview PowerShell cmdlets and
  the Graph API to protect data across Exchange Online, SharePoint, OneDrive, Teams,
  endpoints, and Power BI, including sensitivity labels, custom sensitive information
  types with regex, endpoint DLP rules, and Activity Explorer monitoring. Use when
  stopping PII/PHI/PCI exfiltration, configuring sensitivity labels, or investigating
  DLP incidents for policy tuning.

  '
domain: cybersecurity
subdomain: data-protection
tags:
- DLP
- Microsoft-Purview
- sensitivity-labels
- endpoint-DLP
- data-classification
- compliance
version: 1.0.0
author: mukul975
license: Apache-2.0
nist_csf:
- PR.DS-01
- PR.DS-02
- PR.DS-10
- GV.PO-01
mitre_attack:
- T1486
- T1530
- T1537
- T1048
- T1573
---
# Implementing Data Loss Prevention with Microsoft Purview

## When to Use

- Deploying DLP policies to prevent sensitive data (PII, PHI, PCI, intellectual property) from leaving the organization through email, cloud storage, chat, or endpoint file operations
- Configuring sensitivity labels with encryption, content marking, and auto-labeling to classify documents and emails by confidentiality level
- Creating custom sensitive information types with regex patterns to detect organization-specific data formats (employee IDs, project codes, internal account numbers)
- Deploying endpoint DLP to control copy-to-USB, print, upload-to-cloud, and copy-to-clipboard actions for labeled or sensitive content on managed devices
- Investigating DLP incidents through Activity Explorer to analyze policy match events, user activity patterns, and false positive rates for policy tuning

**Do not use** without appropriate Microsoft 365 E5, E5 Compliance, or E5 Information Protection licensing. Do not deploy DLP policies directly to production enforcement mode without a simulation period. Do not configure endpoint DLP without coordinating with the endpoint management team responsible for device onboarding.

## Prerequisites

- Microsoft 365 E5 or E5 Compliance / E5 Information Protection add-on license assigned to target users
- Global Administrator, Compliance Administrator, or Compliance Data Administrator role in the Microsoft Purview portal
- Exchange Online PowerShell module (ExchangeOnlineManagement v3.x) and Security & Compliance PowerShell for policy automation
- Devices onboarded to Microsoft Purview endpoint DLP through Microsoft Intune or Configuration Manager (Windows 10/11 21H2+, macOS 12+)
- Data classification scan completed or content explorer populated to understand existing sensitive data distribution
- Stakeholder agreement on sensitivity label taxonomy (classification levels, encryption requirements, scope)

## Workflow

### Step 1: Design the Sensitivity Label Taxonomy

Define the classification hierarchy that maps to organizational data handling requirements:

- **Establish label tiers**: Create a label hierarchy reflecting data sensitivity levels. A standard enterprise taxonomy includes:
  ```
  Public           -> No protection, external sharing allowed
  General          -> No encryption, internal watermark "GENERAL"
  Confidential     -> Encryption (all employees), header/footer marking
    ├─ Confidential - All Employees
    ├─ Confidential - Finance
    └─ Confidential - HR
  Highly Confidential -> Encryption (specific users/groups), watermark, no forwarding
    ├─ Highly Confidential - Project X
    └─ Highly Confidential - Board Only
  ```
- **Define protection settings per label**: For each label, configure encryption scope (all employees, specific groups, or custom permissions), content marking (headers, footers, watermarks), and auto-labeling conditions:
  ```powershell
  # Connect to Security & Compliance PowerShell
  Connect-IPPSSession -UserPrincipalName admin@contoso.com

  # Create parent label
  New-Label -DisplayName "Confidential" `
    -Name "Confidential" `
    -Tooltip "Business data that could cause damage if disclosed to unauthorized parties" `
    -Comment "Apply to internal business documents, financial reports, and customer data"

  # Create sub-label with encryption
  New-Label -DisplayName "Confidential - Finance" `
    -Name "Confidential-Finance" `
    -ParentId (Get-Label -Identity "Confidential").Guid `
    -Tooltip "Financial data restricted to Finance department" `
    -EncryptionEnabled $true `
    -EncryptionProtectionType "Template" `
    -EncryptionRightsDefinitions "finance-group@contoso.com:VIEW,VIEWRIGHTSDATA,DOCEDIT,EDIT,PRINT,EXTRACT,OBJMODEL" `
    -ContentType "File, Email"
  ```
- **Configure content marking**: Apply visual indicators that persist with the document:
  ```powershell
  Set-Label -Identity "Confidential-Finance" `
    -HeaderEnabled $true `
    -HeaderText "CONFIDENTIAL - FINANCE" `
    -HeaderFontSize 10 `
    -HeaderFontColor "#FF0000" `
    -HeaderAlignment "Center" `
    -FooterEnabled $true `
    -FooterText "This document contains confidential financial information" `
    -WatermarkEnabled $true `
    -WatermarkText "CONFIDENTIAL" `
    -WatermarkFontSize 36
  ```
- **Publish labels via label policy**: Labels must be published to users through a label policy that defines which users see the labels and whether a default label or mandatory labeling is enforced:
  ```powershell
  New-LabelPolicy -Name "Corporate Label Policy" `
    -Labels "Public","General","Confidential","Confidential-Finance",
            "Confidential-HR","HighlyConfidential","HighlyConfidential-ProjectX" `
    -ExchangeLocation "All" `
    -ModernGroupLocation "All" `
    -Comment "Standard corporate sensitivity labels"

  # Require justification for label downgrade
  Set-LabelPolicy -Identity "Corporate Label Policy" `
    -AdvancedSettings @{RequireDowngradeJustification="True";
                        DefaultLabelId="General"}
  ```

### Step 2: Create DLP Policies with Sensitive Information Types

Configure DLP policies that detect and protect sensitive content across Microsoft 365 workloads:

- **Create a DLP policy using built-in sensitive information types**: Microsoft Purview includes 300+ built-in SITs for credit card numbers, Social Security numbers, passport numbers, and health records. Create a policy targeting financial data:
  ```powershell
  # Create DLP policy scoped to Exchange, SharePoint, OneDrive
  New-DlpCompliancePolicy -Name "Financial Data Protection" `
    -ExchangeLocation "All" `
    -SharePointLocation "All" `
    -OneDriveLocation "All" `
    -TeamsLocation "All" `
    -Mode "TestWithNotifications" `
    -Comment "Protects credit card numbers, bank account numbers, and financial identifiers"

  # Create rule for high-volume credit card detection
  New-DlpComplianceRule -Name "Block Bulk Credit Card Sharing" `
    -Policy "Financial Data Protection" `
    -ContentContainsSensitiveInformation @{
      Name = "Credit Card Number";
      MinCount = 5;
      MinConfidence = 85
    } `
    -BlockAccess $true `
    -BlockAccessScope "All" `
    -NotifyUser "SiteAdmin","LastModifier" `
    -NotifyUserType "NotSet" `
    -GenerateIncidentReport "SiteAdmin" `
    -IncidentReportContent "All" `
    -ReportSeverityLevel "High"

  # Create rule for low-volume with user override
  New-DlpComplianceRule -Name "Warn on Credit Card Sharing" `
    -Policy "Financial Data Protection" `
    -ContentContainsSensitiveInformation @{
      Name = "Credit Card Number";
      MinCount = 1;
      MaxCount = 4;
      MinConfidence = 75
    } `
    -NotifyUser "LastModifier" `
    -NotifyUserType "NotSet" `
    -GenerateAlert "Low" `
    -NotifyOverride "WithJustification"
  ```
- **Create custom sensitive information types with regex**: Define organization-specific patterns for data that built-in SITs do not cover:
  ```powershell
  # Create custom SIT for employee ID format (EMP-XXXXXX)
  $rulePackXml = @"
  <RulePackage xmlns="http://schemas.microsoft.com/office/2011/mce">
    <RulePack id="$(New-Guid)">
      <Version major="1" minor="0" build="0" revision="0"/>
      <Publisher id="$(New-Guid)"/>
    </RulePack>
    <Rules>
      <Entity id="$(New-Guid)" patternsProximity="300"
              recommendedConfidence="85">
        <Pattern confidenceLevel="85">
          <IdMatch idRef="EmployeeId_Regex"/>
        </Pattern>
        <Pattern confidenceLevel="95">
          <IdMatch idRef="EmployeeId_Regex"/>
          <Match idRef="EmployeeId_Keyword"/>
        </Pattern>
      </Entity>
      <Regex id="EmployeeId_Regex">EMP-[0-9]{6}</Regex>
      <Keyword id="EmployeeId_Keyword">
        <Group matchStyle="word">
          <Term>employee</Term>
          <Term>employee id</Term>
          <Term>emp id</Term>
          <Term>staff number</Term>
        </Group>
      </Keyword>
      <LocalizedStrings>
        <Resource idRef="EmployeeId_Regex">
          <Name default="true" langcode="en-us">Contoso Employee ID</Name>
          <Description default="true" langcode="en-us">
            Detects Contoso employee IDs in format EMP-XXXXXX
          </Description>
        </Resource>
      </LocalizedStrings>
    </Rules>
  </RulePackage>
  "@

  # Save and import the rule package
  $rulePackXml | Out-File -FilePath "EmployeeID_SIT.xml" -Encoding utf8
  New-DlpSensitiveInformationTypeRulePackage -FileData (
    [System.IO.File]::ReadAllBytes("EmployeeID_SIT.xml")
  )
  ```
- **Use sensitivity labels as DLP conditions**: Create policies that apply different restrictions based on the label applied to the content:
  ```powershell
  New-DlpCompliancePolicy -Name "Highly Confidential Sharing Control" `
    -ExchangeLocation "All" `
    -SharePointLocation "All" `
    -OneDriveLocation "All" `
    -Mode "Enable"

  New-DlpComplianceRule -Name "Block External Sharing of HC Content" `
    -Policy "Highly Confidential Sharing Control" `
    -ContentContainsSensitiveInformation $null `
    -ContentPropertyContainsWords "MSIP_Label_$(
      (Get-Label -Identity 'HighlyConfidential').Guid
    )_Enabled=True" `
    -BlockAccess $true `
    -BlockAccessScope "NotInOrganization" `
    -NotifyUser "LastModifier" `
    -GenerateIncidentReport "SiteAdmin" `
    -ReportSeverityLevel "High"
  ```

### Step 3: Deploy Endpoint DLP Rules

Extend DLP protection to managed Windows and macOS endpoints to control file operations:

- **Verify device onboarding**: Confirm devices are onboarded to Microsoft Purview endpoint DLP through Microsoft Intune or the local onboarding script:
  ```powershell
  # Check onboarding status via Intune Graph API
  # GET https://graph.microsoft.com/beta/deviceManagement/managedDevices
  # Filter for complianceState and dlpOnboardingStatus

  # Local verification on Windows endpoint
  # Check registry key:
  # HKLM\SOFTWARE\Microsoft\Windows Advanced Threat Protection\Status
  # OnboardingState should be 1
  ```
- **Configure endpoint DLP settings**: Define global settings that control which applications and file types endpoint DLP monitors:
  ```powershell
  # Configure unallowed apps (browsers, cloud sync clients)
  Set-PolicyConfig -EndpointDlpGlobalSettings `
    -UnallowedApps @(
      @{Name="Chrome"; Executable="chrome.exe"},
      @{Name="Firefox"; Executable="firefox.exe"},
      @{Name="PersonalDropbox"; Executable="Dropbox.exe"}
    )

  # Configure unallowed Bluetooth apps
  Set-PolicyConfig -EndpointDlpGlobalSettings `
    -UnallowedBluetoothApps @(
      @{Name="BluetoothFileTransfer"; Executable="fsquirt.exe"}
    )

  # Configure network share groups
  Set-PolicyConfig -EndpointDlpGlobalSettings `
    -NetworkShareGroups @(
      @{
        Name = "Authorized Shares";
        NetworkPaths = @("\\server01\approved$", "\\server02\secure$")
      }
    )

  # Configure sensitive service domains (allowed cloud destinations)
  Set-PolicyConfig -EndpointDlpGlobalSettings `
    -SensitiveServiceDomains @(
      @{
        Name = "Approved Cloud Storage";
        Domains = @("sharepoint.com", "onedrive.com")
        MatchType = "Allow"
      },
      @{
        Name = "Blocked Cloud Storage";
        Domains = @("dropbox.com", "box.com", "drive.google.com")
        MatchType = "Block"
      }
    )
  ```
- **Create endpoint-specific DLP rules**: Define rules that control copy-to-USB, print, upload, and clipboard operations for sensitive content:
  ```powershell
  # Add endpoint location to existing policy
  Set-DlpCompliancePolicy -Identity "Financial Data Protection" `
    -EndpointDlpLocation "All"

  # Create endpoint-specific rule
  New-DlpComplianceRule -Name "Block USB Copy of Financial Data" `
    -Policy "Financial Data Protection" `
    -ContentContainsSensitiveInformation @{
      Name = "Credit Card Number";
      MinCount = 1;
      MinConfidence = 85
    } `
    -EndpointDlpRestrictions @(
      @{Setting="CopyToRemovableMedia"; Value="Block"},
      @{Setting="CopyToNetworkShare"; Value="Audit"},
      @{Setting="CopyToClipboard"; Value="Block"},
      @{Setting="Print"; Value="Warn"},
      @{Setting="UploadToCloudService"; Value="Block"},
      @{Setting="UnallowedBluetoothApp"; Value="Block"}
    ) `
    -NotifyUser "LastModifier" `
    -GenerateIncidentReport "SiteAdmin"
  ```
- **Configure printer groups and USB device exceptions**: Allow specific printers and approved USB devices while blocking unauthorized removable media:
  ```powershell
  # Define authorized USB devices by vendor/product ID
  Set-PolicyConfig -EndpointDlpGlobalSettings `
    -RemovableMediaGroups @(
      @{
        Name = "Approved Encrypted USBs";
        Devices = @(
          @{VendorId="0781"; ProductId="5583"; SerialNumber="*"}  # SanDisk Extreme
        )
      }
    )

  # Define authorized printers
  Set-PolicyConfig -EndpointDlpGlobalSettings `
    -PrinterGroups @(
      @{
        Name = "Corporate Printers";
        Printers = @(
          @{PrinterName="*Corporate*"; PrinterType="Corporate"},
          @{PrinterName="PDF Printer"; PrinterType="Print to PDF"}
        )
      }
    )
  ```

### Step 4: Configure Auto-Labeling Policies

Deploy service-side auto-labeling to automatically classify content at rest and in transit:

- **Create auto-labeling policy for email**: Automatically label inbound and outbound emails containing sensitive information:
  ```powershell
  New-AutoSensitivityLabelPolicy -Name "Auto-Label Financial Emails" `
    -ExchangeLocation "All" `
    -Mode "TestWithNotifications" `
    -Comment "Automatically labels emails containing financial data as Confidential-Finance"

  New-AutoSensitivityLabelRule -Name "Financial SIT Match" `
    -Policy "Auto-Label Financial Emails" `
    -SensitiveInformationType @{
      Name = "Credit Card Number";
      MinCount = 1;
      MinConfidence = 85
    },@{
      Name = "U.S. Bank Account Number";
      MinCount = 1;
      MinConfidence = 85
    } `
    -WorkloadDomain "Exchange" `
    -ApplySensitivityLabel "Confidential-Finance"
  ```
- **Create auto-labeling policy for SharePoint and OneDrive**: Label existing files at rest that match sensitive information patterns:
  ```powershell
  New-AutoSensitivityLabelPolicy -Name "Auto-Label SP Financial Docs" `
    -SharePointLocation "https://contoso.sharepoint.com/sites/finance" `
    -OneDriveLocation "All" `
    -Mode "TestWithNotifications"

  New-AutoSensitivityLabelRule -Name "Financial Docs SIT Match" `
    -Policy "Auto-Label SP Financial Docs" `
    -SensitiveInformationType @{
      Name = "Credit Card Number"; MinCount = 1; MinConfidence = 85
    } `
    -WorkloadDomain "SharePoint" `
    -ApplySensitivityLabel "Confidential-Finance"
  ```
- **Simulate before enforcing**: Always run auto-labeling in simulation mode first. Review the simulation results in the Microsoft Purview portal under Information Protection > Auto-labeling. The simulation shows estimated matches per location and sample content matches for validation. Only switch to enforcement mode after confirming accuracy:
  ```powershell
  # Check simulation results
  Get-AutoSensitivityLabelPolicy -Identity "Auto-Label Financial Emails" |
    Select-Object Name, Mode, WhenCreated, DistributionStatus

  # Switch to enforcement after validation
  Set-AutoSensitivityLabelPolicy -Identity "Auto-Label Financial Emails" `
    -Mode "Enable"
  ```

### Step 5: Monitor with Activity Explorer and Manage DLP Alerts

Use Activity Explorer and the DLP alerts dashboard to monitor policy effectiveness and investigate incidents:

- **Access Activity Explorer**: Navigate to Microsoft Purview portal > Data Classification > Activity Explorer. Filter by activity type "DLPRuleMatch" to see all DLP policy matches. Key columns include:
  - Activity timestamp and user principal name
  - Sensitive information type matched and confidence level
  - Policy and rule name that triggered
  - Action taken (Audit, Block, Warn with Override)
  - Location (Exchange, SharePoint, OneDrive, Endpoint)
  - File name and site URL
- **Analyze false positive rates**: Export Activity Explorer data filtered by "Override" actions with justification text to identify rules that users frequently override. A high override rate (>20%) indicates the rule may be too aggressive or matching non-sensitive content:
  ```
  Activity Explorer filter:
    Activity type = DLPRuleMatch
    Action = Override
    Date range = Last 30 days
    Policy name = Financial Data Protection

  Export to CSV for analysis of override justifications and
  affected file types to refine SIT confidence thresholds.
  ```
- **Configure DLP alerts**: Set up alert policies in Microsoft Purview > Data Loss Prevention > Alerts to receive notifications for high-severity matches:
  ```powershell
  # DLP alerts are configured within the DLP rule itself
  # Adjust alert volume thresholds on high-traffic rules
  Set-DlpComplianceRule -Identity "Block Bulk Credit Card Sharing" `
    -GenerateAlert "High" `
    -AlertProperties @{
      AggregationType = "SimpleAggregation";
      Threshold = 1;
      TimeWindow = "00:05:00"
    }
  ```
- **Query DLP events via Microsoft Graph API**: Programmatically retrieve DLP alerts and policy match details for integration with SIEM or custom dashboards:
  ```python
  import requests

  # Authenticate with Microsoft Graph (client credentials flow)
  token_url = "https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token"
  token_response = requests.post(token_url, data={
      "client_id": client_id,
      "client_secret": client_secret,
      "scope": "https://graph.microsoft.com/.default",
      "grant_type": "client_credentials"
  })
  access_token = token_response.json()["access_token"]

  headers = {"Authorization": f"Bearer {access_token}"}

  # Retrieve DLP alerts
  alerts_url = "https://graph.microsoft.com/v1.0/security/alerts_v2"
  params = {
      "$filter": "serviceSource eq 'microsoftDataLossPrevention'",
      "$top": 50,
      "$orderby": "createdDateTime desc"
  }
  response = requests.get(alerts_url, headers=headers, params=params)
  alerts = response.json().get("value", [])

  for alert in alerts:
      print(f"Alert: {alert['title']}")
      print(f"  Severity: {alert['severity']}")
      print(f"  Status: {alert['status']}")
      print(f"  Created: {alert['createdDateTime']}")
      print(f"  User: {alert.get('userStates', [{}])[0].get('userPrincipalName', 'N/A')}")
  ```
- **Retrieve DLP policy match details for compliance reporting**: Use the unified audit log to extract granular DLP match data including the matched content, SIT type, and confidence level:
  ```powershell
  # Search unified audit log for DLP policy matches
  Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) `
    -EndDate (Get-Date) `
    -RecordType "DLP" `
    -ResultSize 1000 |
    Select-Object CreationDate, UserIds, Operations,
      @{N='PolicyName';E={($_.AuditData | ConvertFrom-Json).PolicyDetails.PolicyName}},
      @{N='RuleName';E={($_.AuditData | ConvertFrom-Json).PolicyDetails.Rules.RuleName}},
      @{N='SITMatched';E={($_.AuditData | ConvertFrom-Json).SensitiveInfoDetections.SensitiveType}} |
    Export-Csv -Path "DLP_Audit_Report.csv" -NoTypeInformation
  ```

## Key Concepts

| Term | Definition |
|------|------------|
| **Sensitivity Label** | A classification tag applied to documents and emails that can enforce encryption, content marking (headers/footers/watermarks), and access restrictions. Labels persist with the content and travel with files when shared externally. |
| **Sensitive Information Type (SIT)** | A pattern-based classifier that detects specific data patterns (credit card numbers, SSNs, custom regex) in content. Each SIT has a confidence level (low/medium/high) determined by primary pattern match plus corroborating evidence (keywords, proximity). |
| **DLP Policy** | A set of rules that detect sensitive information in Microsoft 365 locations (Exchange, SharePoint, OneDrive, Teams, Endpoints) and apply protective actions (audit, warn with override, block) based on the sensitivity of matched content and the sharing context. |
| **Endpoint DLP** | Extension of DLP protection to managed Windows and macOS devices that monitors and controls file operations including copy-to-USB, print, upload-to-cloud, copy-to-clipboard, and access by unallowed applications for files containing sensitive information. |
| **Activity Explorer** | A monitoring dashboard in Microsoft Purview that displays a historical view (up to 30 days) of labeled content activities, DLP policy matches, and user interactions with classified data across all monitored locations. |
| **Auto-Labeling** | Service-side automatic classification that applies sensitivity labels to documents and emails matching specified SIT patterns without requiring user interaction. Runs in simulation mode first to preview matches before enforcement. |
| **Content Marking** | Visual indicators (headers, footers, watermarks) applied by sensitivity labels to documents and emails. Markings persist in the file and are visible when printed or shared, serving as a visual classification reminder. |
| **DLP Alert** | A notification generated when a DLP rule match meets the configured severity threshold. Alerts appear in the Microsoft Purview DLP alerts dashboard and can be routed to Microsoft Sentinel or other SIEM platforms. |

## Tools & Systems

- **Microsoft Purview Compliance Portal**: Web-based administration interface for creating and managing sensitivity labels, DLP policies, auto-labeling rules, and reviewing Activity Explorer data and DLP alerts.
- **Security & Compliance PowerShell**: PowerShell module (Connect-IPPSSession) providing cmdlets for programmatic management of labels (New-Label, Set-Label), label policies (New-LabelPolicy), DLP policies (New-DlpCompliancePolicy, New-DlpComplianceRule), and sensitive information types.
- **Microsoft Graph Security API**: REST API providing programmatic access to DLP alerts (security/alerts_v2), data classification insights, and protection scope evaluation for integrating Purview DLP with custom applications and SIEM platforms.
- **Microsoft Intune**: Endpoint management platform used to onboard Windows and macOS devices to endpoint DLP, deploy configuration profiles, and manage device compliance states.
- **Microsoft Sentinel**: Cloud-native SIEM that ingests DLP alerts and audit logs from Microsoft Purview via the Microsoft 365 Defender data connector for correlation with other security events and automated incident response.
- **Unified Audit Log**: Microsoft 365 audit service recording all DLP policy match events (RecordType "DLP") with detailed match metadata for compliance reporting and forensic investigation.

## Common Scenarios

### Scenario: Protecting Financial Data Across a Multinational Organization

**Context**: A financial services company with 15,000 users across 12 countries needs to prevent credit card numbers, bank account details, and financial statements from being shared externally through email, Teams, SharePoint, and endpoint file operations. The company must comply with PCI-DSS and GDPR.

**Approach**:
1. Design a four-tier sensitivity label taxonomy: Public, Internal, Confidential (with sub-labels for Finance, Legal, HR), and Highly Confidential. Publish labels to all users with "Internal" as the default label and mandatory labeling enabled for email.
2. Create a DLP policy "PCI-DSS Financial Data Protection" scoped to all Exchange, SharePoint, OneDrive, Teams, and Endpoint locations. Configure two rules: a warning rule for 1-4 credit card numbers (notify user, allow override with justification) and a blocking rule for 5+ credit card numbers (block external sharing, generate incident report to compliance team).
3. Deploy endpoint DLP with rules blocking copy-to-USB and upload-to-unapproved-cloud for any file containing credit card numbers or labeled "Confidential - Finance". Allow printing with audit logging. Configure approved USB device exceptions for encrypted corporate drives by vendor/product ID.
4. Create auto-labeling policies that scan existing SharePoint finance sites and OneDrive locations, automatically applying "Confidential - Finance" to documents matching credit card number or bank account number SITs with confidence level 85+.
5. Run all policies in simulation mode for 14 days. Review Activity Explorer for false positive rates, override patterns, and unprotected sensitive content locations. Tune SIT confidence thresholds from 75 to 85 on the credit card SIT after identifying false positives from partial number sequences in meeting notes.
6. Switch to enforcement mode after stakeholder sign-off. Configure DLP alerts with Microsoft Sentinel integration for real-time incident correlation. Schedule monthly Activity Explorer reviews to track policy effectiveness metrics.

**Pitfalls**:
- Deploying DLP policies in enforcement mode without simulation, causing mass blocking of legitimate business communications and user productivity disruption
- Using low confidence thresholds (65) for SITs, generating excessive false positives that erode user trust and lead to policy override fatigue
- Not configuring endpoint DLP exceptions for approved encrypted USB devices, blocking legitimate data transfers to authorized external parties
- Forgetting to publish sensitivity labels via a label policy after creation, resulting in labels being invisible to end users in Office applications
- Not coordinating auto-labeling deployment with document library owners, leading to unexpected label changes on existing content that alter access permissions

### Scenario: Implementing Custom DLP for Intellectual Property Protection

**Context**: A pharmaceutical company needs to prevent research data identified by internal project codes (format: RX-YYYY-NNNN) and compound identifiers from being shared outside the research department. The data appears in lab reports, research presentations, and email communications.

**Approach**:
1. Create a custom sensitive information type using regex `RX-20[2-3][0-9]-[0-9]{4}` with corroborating keywords ("compound", "trial", "formulation", "assay", "efficacy") within 300-character proximity. Set primary pattern at 85% confidence and keyword-corroborated pattern at 95%.
2. Create a second custom SIT for compound identifiers using regex `CPD-[A-Z]{3}-[0-9]{5}` with keywords ("molecule", "synthesis", "pharmacokinetics") for higher confidence matching.
3. Deploy a DLP policy scoped to the Research department's Exchange distribution list, SharePoint research site collection, and research team OneDrive accounts. Block external sharing, block forwarding to non-research internal users, and generate alerts for the research compliance officer.
4. Configure endpoint DLP to prevent copy-to-USB and screen capture of documents containing the custom SITs on research department laptops. Allow printing only to approved secure printers in the research facility.
5. Create a sensitivity label "Highly Confidential - Research" with encryption restricting access to the Research security group. Configure auto-labeling to apply this label to documents matching either custom SIT.
6. Monitor Activity Explorer weekly for 30 days post-deployment. The compliance team identifies that the RX-YYYY-NNNN regex matches historical project codes in archived documents. Refine the regex to `RX-202[4-6]-[0-9]{4}` to target only active project codes and reduce false positives by 60%.

**Pitfalls**:
- Using positional regex anchors (^ and $) in custom SITs, which do not work as expected in Microsoft Purview regex evaluation and cause pattern match failures
- Setting MinCount too low (1) for the project code SIT without keyword corroboration, matching isolated instances in general business correspondence that happen to follow the same format
- Not testing the custom SIT against a representative sample corpus before deploying the DLP policy, missing edge cases in the regex pattern
- Scoping the policy too broadly (entire organization) instead of targeting the research department, causing alerts on legitimate references to project codes in executive summaries

## Output Format

```
## DLP Policy Deployment Report

**Policy Name**: PCI-DSS Financial Data Protection
**Deployment Date**: 2026-03-19
**Current Mode**: Simulation (TestWithNotifications)
**Locations**: Exchange Online, SharePoint Online, OneDrive, Teams, Endpoints

---

### Simulation Results (14-Day Period)

**Total Policy Matches**: 4,287
**Unique Users Affected**: 892
**Unique Files/Messages**: 3,641

| Rule | Matches | Action | Override Rate |
|------|---------|--------|---------------|
| Block Bulk Credit Card Sharing (5+) | 47 | Block | N/A |
| Warn on Credit Card Sharing (1-4) | 4,240 | Warn | 12.3% |

### Sensitive Information Type Breakdown

| SIT | Matches | Avg Confidence | False Positive Est. |
|-----|---------|----------------|---------------------|
| Credit Card Number | 3,891 | 87% | 8.2% |
| U.S. Bank Account Number | 312 | 82% | 15.1% |
| ABA Routing Number | 84 | 79% | 22.6% |

### Recommendations

1. **Enable enforcement** for "Block Bulk Credit Card Sharing" rule -
   47 matches are all true positives involving bulk credit card data in
   spreadsheet attachments.

2. **Increase confidence threshold** for ABA Routing Number from 75 to 85 -
   22.6% false positive rate driven by 9-digit numbers in invoice references
   matching the routing number pattern.

3. **Add file type exception** for password-protected ZIP attachments that
   trigger false positives when the credit card SIT matches encrypted content
   metadata.

4. **Deploy endpoint DLP** in audit mode for 7 additional days before
   enabling block actions on USB copy and cloud upload.

---

### DLP Alert Summary (Last 7 Days)

| Severity | Count | Top Policy | Top User |
|----------|-------|------------|----------|
| High | 12 | Financial Data Protection | j.smith@contoso.com |
| Medium | 89 | IP Protection - Research | r.chen@contoso.com |
| Low | 234 | General PII Protection | (distributed) |

### Activity Explorer Insights

- Peak DLP match activity: Monday 09:00-11:00 UTC (weekly report distribution)
- Top matched location: Finance SharePoint site (62% of all matches)
- Most overridden rule: "Warn on Credit Card Sharing" (523 overrides, 12.3%)
- Override justification analysis: 78% "Business requirement", 15% "False positive",
  7% "Other"
```

===== END FILE =====


===== FILE: skills/implementing-data-loss-prevention-with-microsoft-purview/LICENSE =====

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


===== FILE: skills/implementing-data-loss-prevention-with-microsoft-purview/references/api-reference.md =====
# API Reference: Microsoft Purview DLP Management Agent

## Overview

Automates Microsoft Purview DLP monitoring and compliance reporting through the Microsoft Graph Security API. Retrieves DLP alerts, sensitivity label configurations, and generates policy health assessments and compliance reports. Requires Azure AD app registration with Security.Read.All and InformationProtectionPolicy.Read.All permissions.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| requests | >=2.28 | HTTP requests to Microsoft Graph API |

## CLI Usage

```bash
# Retrieve DLP alerts from last 7 days
python agent.py --tenant-id <tenant-id> --client-id <client-id> \
  --client-secret <secret> --action alerts --days 7

# Filter high-severity alerts
python agent.py --tenant-id <tenant-id> --client-id <client-id> \
  --client-secret <secret> --action alerts --severity high --days 30

# List sensitivity labels
python agent.py --tenant-id <tenant-id> --client-id <client-id> \
  --client-secret <secret> --action labels

# Check DLP policy health
python agent.py --tenant-id <tenant-id> --client-id <client-id> \
  --client-secret <secret> --action health --days 14

# Generate full compliance report
python agent.py --tenant-id <tenant-id> --client-id <client-id> \
  --client-secret <secret> --action report --days 30 --output-dir ./reports
```

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--tenant-id` | Yes | Azure AD tenant ID for the Microsoft 365 organization |
| `--client-id` | Yes | Azure AD app registration client (application) ID |
| `--client-secret` | Yes | Azure AD app registration client secret |
| `--action` | Yes | Action to perform: `alerts`, `labels`, `health`, or `report` |
| `--days` | No | Number of days to look back for alerts (default: 7) |
| `--severity` | No | Filter alerts by severity: high, medium, low, informational |
| `--output-dir` | No | Directory for output files (default: current directory) |
| `--output` | No | Specific output file path (overrides default naming) |

## Azure AD App Registration Requirements

The app registration requires the following Microsoft Graph API permissions (Application type):

| Permission | Type | Purpose |
|-----------|------|---------|
| `Security.Read.All` | Application | Read DLP alerts from security/alerts_v2 |
| `InformationProtectionPolicy.Read.All` | Application | Read sensitivity labels and DLP policies |
| `User.Read.All` | Application | Resolve user principal names in alert data |

## Key Classes

### `PurviewAuthClient`
Handles OAuth2 client credentials flow authentication with automatic token caching and renewal.

**Methods:**
- `get_token()` - Obtains or returns cached access token. Refreshes 5 minutes before expiry.
- `headers()` - Returns authorization headers dictionary for Graph API requests.

## Key Functions

### `get_dlp_alerts(auth_client, days_back, severity, top)`
Retrieves DLP alerts from Microsoft Graph Security API (`/security/alerts_v2`). Filters by service source `microsoftDataLossPrevention`, date range, and optional severity. Returns list of alert objects.

### `get_sensitivity_labels(auth_client)`
Retrieves all sensitivity labels configured in the tenant from the beta endpoint (`/security/informationProtection/sensitivityLabels`). Returns list of label objects with ID, name, protection settings, and hierarchy.

### `generate_alert_summary(alerts)`
Computes summary statistics from alert list: severity breakdown, status breakdown, top 10 triggered policies, and top 10 affected users.

### `generate_label_report(labels)`
Transforms raw label data into a sorted report with configuration details including protection status, parent relationships, and content format support.

### `check_policy_health(alerts, threshold_high, threshold_override_pct)`
Analyzes alert patterns to identify policy health issues:
- `HIGH_ALERT_VOLUME`: More than threshold high-severity alerts
- `NOISY_POLICY`: Single policy generating 100+ alerts
- `UNRESOLVED_ALERT_BACKLOG`: 50+ alerts in "new" status
- `HEALTHY`: No anomalies detected

### `export_alerts_csv(alerts, output_path)`
Exports alerts to CSV format with columns: id, title, severity, status, createdDateTime, user, description, category. Suitable for compliance reporting and spreadsheet analysis.

### `generate_compliance_report(auth_client, days_back, output_dir)`
Generates comprehensive DLP compliance report combining alert summary, policy health assessment, sensitivity label configuration, and detailed alert data. Outputs JSON report and CSV export.

## Output Files

| Action | Default Output | Format |
|--------|---------------|--------|
| `alerts` | `dlp_alerts.json` | JSON with summary and alert details |
| `labels` | `sensitivity_labels.json` | JSON array of label configurations |
| `health` | `dlp_health.json` | JSON array of health findings |
| `report` | `dlp_compliance_report.json` + `dlp_alerts_export.csv` | JSON report + CSV export |

## Health Finding Types

| Finding | Severity | Trigger |
|---------|----------|---------|
| `HIGH_ALERT_VOLUME` | WARNING | More than 10 high-severity alerts in analysis period |
| `NOISY_POLICY` | INFO | Single policy generating 100+ alerts |
| `UNRESOLVED_ALERT_BACKLOG` | WARNING | 50+ alerts in "new" status |
| `HEALTHY` | INFO | All health checks passed |

===== END FILE =====


===== FILE: skills/implementing-data-loss-prevention-with-microsoft-purview/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized Microsoft 365 compliance administration only
"""Microsoft Purview DLP Management Agent - Automates DLP policy deployment and monitoring via Graph API."""

import json
import logging
import argparse
import csv
from datetime import datetime, timezone, timedelta
from pathlib import Path

import requests

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

GRAPH_BASE = "https://graph.microsoft.com/v1.0"
GRAPH_BETA = "https://graph.microsoft.com/beta"


class PurviewAuthClient:
    """Handles OAuth2 client credentials authentication for Microsoft Graph."""

    def __init__(self, tenant_id, client_id, client_secret):
        self.tenant_id = tenant_id
        self.client_id = client_id
        self.client_secret = client_secret
        self.access_token = None
        self.token_expiry = None

    def get_token(self):
        if self.access_token and self.token_expiry and datetime.now(timezone.utc) < self.token_expiry:
            return self.access_token
        token_url = f"https://login.microsoftonline.com/{self.tenant_id}/oauth2/v2.0/token"
        response = requests.post(token_url, data={
            "client_id": self.client_id,
            "client_secret": self.client_secret,
            "scope": "https://graph.microsoft.com/.default",
            "grant_type": "client_credentials",
        }, timeout=30)
        response.raise_for_status()
        token_data = response.json()
        self.access_token = token_data["access_token"]
        self.token_expiry = datetime.now(timezone.utc) + timedelta(
            seconds=token_data.get("expires_in", 3600) - 300
        )
        logger.info("Obtained Graph API access token (expires in %d seconds)",
                     token_data.get("expires_in", 3600))
        return self.access_token

    def headers(self):
        return {
            "Authorization": f"Bearer {self.get_token()}",
            "Content-Type": "application/json",
        }


def get_dlp_alerts(auth_client, days_back=7, severity=None, top=50):
    """Retrieve DLP alerts from Microsoft Graph Security API."""
    url = f"{GRAPH_BASE}/security/alerts_v2"
    start_date = (datetime.now(timezone.utc) - timedelta(days=days_back)).strftime(
        "%Y-%m-%dT%H:%M:%SZ"
    )
    filter_parts = [
        "serviceSource eq 'microsoftDataLossPrevention'",
        f"createdDateTime ge {start_date}",
    ]
    if severity:
        filter_parts.append(f"severity eq '{severity}'")
    params = {
        "$filter": " and ".join(filter_parts),
        "$top": top,
        "$orderby": "createdDateTime desc",
    }
    response = requests.get(url, headers=auth_client.headers(), params=params, timeout=60)
    response.raise_for_status()
    alerts = response.json().get("value", [])
    logger.info("Retrieved %d DLP alerts from last %d days", len(alerts), days_back)
    return alerts


def get_sensitivity_labels(auth_client):
    """Retrieve all sensitivity labels from the tenant."""
    url = f"{GRAPH_BETA}/security/informationProtection/sensitivityLabels"
    response = requests.get(url, headers=auth_client.headers(), timeout=30)
    response.raise_for_status()
    labels = response.json().get("value", [])
    logger.info("Retrieved %d sensitivity labels", len(labels))
    return labels


def evaluate_dlp_protection_scope(auth_client, user_id):
    """Evaluate DLP protection scope for a specific user."""
    url = f"{GRAPH_BETA}/users/{user_id}/security/informationProtection/policy/evaluateApplication"
    payload = {
        "contentInfo": {
            "@odata.type": "#microsoft.graph.security.contentInfo",
            "format@odata.type": "#microsoft.graph.security.contentFormat",
            "format": "default",
        }
    }
    response = requests.post(url, headers=auth_client.headers(), json=payload, timeout=30)
    if response.status_code == 200:
        return response.json()
    logger.warning("DLP evaluation for user %s returned status %d", user_id, response.status_code)
    return None


def generate_alert_summary(alerts):
    """Generate summary statistics from DLP alerts."""
    severity_counts = {"high": 0, "medium": 0, "low": 0, "informational": 0}
    policy_counts = {}
    user_counts = {}
    status_counts = {"new": 0, "inProgress": 0, "resolved": 0}

    for alert in alerts:
        sev = alert.get("severity", "informational").lower()
        severity_counts[sev] = severity_counts.get(sev, 0) + 1

        title = alert.get("title", "Unknown Policy")
        policy_counts[title] = policy_counts.get(title, 0) + 1

        status = alert.get("status", "new")
        status_counts[status] = status_counts.get(status, 0) + 1

        user_states = alert.get("userStates", [])
        for user_state in user_states:
            upn = user_state.get("userPrincipalName", "Unknown")
            user_counts[upn] = user_counts.get(upn, 0) + 1

    top_policies = sorted(policy_counts.items(), key=lambda x: x[1], reverse=True)[:10]
    top_users = sorted(user_counts.items(), key=lambda x: x[1], reverse=True)[:10]

    return {
        "total_alerts": len(alerts),
        "severity_breakdown": severity_counts,
        "status_breakdown": status_counts,
        "top_policies": [{"policy": p, "count": c} for p, c in top_policies],
        "top_users": [{"user": u, "count": c} for u, c in top_users],
    }


def generate_label_report(labels):
    """Generate a report of sensitivity label configuration."""
    report = []
    for label in labels:
        entry = {
            "id": label.get("id"),
            "name": label.get("name"),
            "description": label.get("description", ""),
            "color": label.get("color", ""),
            "sensitivity": label.get("sensitivity", 0),
            "is_active": label.get("isActive", False),
            "parent_id": label.get("parent", {}).get("id") if label.get("parent") else None,
            "content_formats": label.get("contentFormats", []),
            "has_protection": bool(label.get("protectionEnabled")),
        }
        report.append(entry)
    return sorted(report, key=lambda x: x.get("sensitivity", 0))


def check_policy_health(alerts, threshold_high=10, threshold_override_pct=20.0):
    """Analyze DLP policy health based on alert patterns."""
    findings = []

    high_severity = [a for a in alerts if a.get("severity", "").lower() == "high"]
    if len(high_severity) > threshold_high:
        findings.append({
            "finding": "HIGH_ALERT_VOLUME",
            "severity": "WARNING",
            "detail": f"{len(high_severity)} high-severity DLP alerts in the analysis period. "
                      f"Threshold: {threshold_high}. Investigate for data exfiltration patterns.",
            "recommendation": "Review top-triggered policies and affected users. Check for "
                              "compromised accounts or policy misconfiguration.",
        })

    policy_alert_counts = {}
    for alert in alerts:
        title = alert.get("title", "Unknown")
        policy_alert_counts[title] = policy_alert_counts.get(title, 0) + 1

    for policy, count in policy_alert_counts.items():
        if count > 100:
            findings.append({
                "finding": "NOISY_POLICY",
                "severity": "INFO",
                "detail": f"Policy '{policy}' generated {count} alerts. May indicate "
                          f"false positive issues or overly broad matching rules.",
                "recommendation": "Review SIT confidence thresholds and policy conditions. "
                                  "Consider increasing MinConfidence or adding exclusions.",
            })

    unresolved = [a for a in alerts if a.get("status") == "new"]
    if len(unresolved) > 50:
        findings.append({
            "finding": "UNRESOLVED_ALERT_BACKLOG",
            "severity": "WARNING",
            "detail": f"{len(unresolved)} DLP alerts in 'new' status. Alert fatigue risk.",
            "recommendation": "Assign alerts to compliance analysts. Configure auto-resolution "
                              "for low-severity informational alerts. Implement alert triage SOP.",
        })

    if not findings:
        findings.append({
            "finding": "HEALTHY",
            "severity": "INFO",
            "detail": "DLP policy health checks passed. No anomalies detected.",
            "recommendation": "Continue regular monitoring. Schedule quarterly policy review.",
        })

    return findings


def export_alerts_csv(alerts, output_path):
    """Export DLP alerts to CSV for compliance reporting."""
    fieldnames = [
        "id", "title", "severity", "status", "createdDateTime",
        "user", "description", "category",
    ]
    with open(output_path, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        for alert in alerts:
            user_states = alert.get("userStates", [])
            upn = user_states[0].get("userPrincipalName", "N/A") if user_states else "N/A"
            writer.writerow({
                "id": alert.get("id", ""),
                "title": alert.get("title", ""),
                "severity": alert.get("severity", ""),
                "status": alert.get("status", ""),
                "createdDateTime": alert.get("createdDateTime", ""),
                "user": upn,
                "description": alert.get("description", ""),
                "category": alert.get("category", ""),
            })
    logger.info("Exported %d alerts to %s", len(alerts), output_path)


def generate_compliance_report(auth_client, days_back=30, output_dir="."):
    """Generate comprehensive DLP compliance report."""
    output_dir = Path(output_dir)
    output_dir.mkdir(parents=True, exist_ok=True)

    logger.info("Generating DLP compliance report for last %d days", days_back)

    alerts = get_dlp_alerts(auth_client, days_back=days_back, top=500)
    alert_summary = generate_alert_summary(alerts)
    health_findings = check_policy_health(alerts)

    labels = get_sensitivity_labels(auth_client)
    label_report = generate_label_report(labels)

    report = {
        "report_generated": datetime.now(timezone.utc).isoformat(),
        "analysis_period_days": days_back,
        "alert_summary": alert_summary,
        "policy_health": health_findings,
        "sensitivity_labels": label_report,
        "alert_details": alerts[:100],
    }

    report_path = output_dir / "dlp_compliance_report.json"
    report_path.write_text(json.dumps(report, indent=2, default=str))
    logger.info("Compliance report saved to %s", report_path)

    csv_path = output_dir / "dlp_alerts_export.csv"
    export_alerts_csv(alerts, csv_path)

    print("\n" + "=" * 70)
    print("DLP COMPLIANCE REPORT SUMMARY")
    print("=" * 70)
    print(f"Report Period: Last {days_back} days")
    print(f"Total Alerts: {alert_summary['total_alerts']}")
    print(f"Severity: High={alert_summary['severity_breakdown']['high']}, "
          f"Medium={alert_summary['severity_breakdown']['medium']}, "
          f"Low={alert_summary['severity_breakdown']['low']}")
    print(f"Sensitivity Labels Configured: {len(label_report)}")
    print(f"\nPolicy Health Findings: {len(health_findings)}")
    for finding in health_findings:
        print(f"  [{finding['severity']}] {finding['finding']}: {finding['detail']}")
    print(f"\nTop Triggered Policies:")
    for entry in alert_summary.get("top_policies", [])[:5]:
        print(f"  - {entry['policy']}: {entry['count']} alerts")
    print(f"\nTop Affected Users:")
    for entry in alert_summary.get("top_users", [])[:5]:
        print(f"  - {entry['user']}: {entry['count']} alerts")
    print("=" * 70)
    print(f"Full report: {report_path}")
    print(f"Alert export: {csv_path}")

    return report


def main():
    parser = argparse.ArgumentParser(
        description="Microsoft Purview DLP Management Agent - Monitor and report on DLP policies"
    )
    parser.add_argument("--tenant-id", required=True, help="Azure AD tenant ID")
    parser.add_argument("--client-id", required=True, help="App registration client ID")
    parser.add_argument("--client-secret", required=True,
                        help="App registration client secret")
    parser.add_argument("--action", required=True,
                        choices=["alerts", "labels", "health", "report"],
                        help="Action to perform")
    parser.add_argument("--days", type=int, default=7,
                        help="Number of days to look back for alerts (default: 7)")
    parser.add_argument("--severity", choices=["high", "medium", "low", "informational"],
                        help="Filter alerts by severity")
    parser.add_argument("--output-dir", default=".",
                        help="Directory for output files (default: current directory)")
    parser.add_argument("--output", help="Output file path (overrides default naming)")
    args = parser.parse_args()

    auth_client = PurviewAuthClient(args.tenant_id, args.client_id, args.client_secret)

    if args.action == "alerts":
        alerts = get_dlp_alerts(auth_client, days_back=args.days, severity=args.severity)
        summary = generate_alert_summary(alerts)
        output = {"summary": summary, "alerts": alerts}
        out_path = args.output or Path(args.output_dir) / "dlp_alerts.json"
        Path(out_path).write_text(json.dumps(output, indent=2, default=str))
        logger.info("Alert report saved to %s (%d alerts)", out_path, len(alerts))

    elif args.action == "labels":
        labels = get_sensitivity_labels(auth_client)
        label_report = generate_label_report(labels)
        out_path = args.output or Path(args.output_dir) / "sensitivity_labels.json"
        Path(out_path).write_text(json.dumps(label_report, indent=2, default=str))
        logger.info("Label report saved to %s (%d labels)", out_path, len(labels))

    elif args.action == "health":
        alerts = get_dlp_alerts(auth_client, days_back=args.days, top=500)
        findings = check_policy_health(alerts)
        out_path = args.output or Path(args.output_dir) / "dlp_health.json"
        Path(out_path).write_text(json.dumps(findings, indent=2, default=str))
        logger.info("Health report saved to %s (%d findings)", out_path, len(findings))
        for finding in findings:
            level = logging.WARNING if finding["severity"] == "WARNING" else logging.INFO
            logger.log(level, "[%s] %s: %s", finding["severity"], finding["finding"],
                       finding["detail"])

    elif args.action == "report":
        generate_compliance_report(auth_client, days_back=args.days, output_dir=args.output_dir)


if __name__ == "__main__":
    main()

===== END FILE =====
